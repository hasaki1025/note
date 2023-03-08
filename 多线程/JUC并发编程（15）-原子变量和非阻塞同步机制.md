# JUC并发编程（15）-原子变量和非阻塞同步机制

- ## 锁的劣势

  - 锁的同步带来的大量的性能损失（上下文切换，单线程的执行），volatile虽然带来了轻量级的同步机制但是无法保证原子性
  - 一个线程在等待锁的时候只能采用阻塞的方法，这可能会导致高优先级线程持续等待低优先级线程释放锁（优先级反转）

- ## 硬件对并发的支持

  - 独占锁也称为悲观锁，乐观锁类似于CAS

  - 现代大部分处理器都包含了某种形式的读-改-写的指令（原子操作）,JDK5之后JVM技能直接使用这些指令

  - 比较并交换指令（CAS）

    - 三个参数
      - V需要读写的内存位置中的值
      - A进行比较的值（程序之前读取到的值）
      - B需要写入的新值
    - 操作过程
      - 比较V和A，如果相同则修改否则什么都不做，最终都会返回V值

    ```java
    boolean compareAndSet()
    boolean compareAndSwap()
    ```

    - 多个线程进行操作时只有一个线程能成功，失败的线程能够选择重新尝试或者其他操作（交给程序实现）

  - 非阻塞的计数器

    ```java
    public class CasCounter {
        private SimulatedCAS value;
    
        public int getValue() {
            return value.get();
        }
    
        public int increment() {
            int v;
            do {
                v = value.get();
            } while (v != value.compareAndSwap(v, v + 1));
            return v + 1;
        }
    }
    ```

    - 通常反复重试是一种合理的策略但是在一些竞争激烈的情况下，最好的方法是等待一段时间或者回退，从而避免造成活锁
    - 竞争程度不大的情况下CAS性能更好，CAS的虽然看起来复杂但是在JVM和操作系统的代码调用实际上比较短，CAS主要缺点是调用对于竞争问题的处理(重试、回退、放弃)

  - JVM对CAS的支持

    - JDK5后提供底层支持，在此之前将使用自旋锁
    - 原子变量类采用的就是CAS

- ## 原子变量

  - 提供set和get方法以及原子的CAS操作

  - 4种原子变量

    - 标量类

      - AtomicInteger等

    - 更新器类

      ```java
      
      //外部变量
      public volatile int date;
      
      void change()
      {
          //可原子操作的更新非原子类变量，只需要指定该变量所属class和变量名称
          AtomicIntegerFieldUpdater<test19> updater =  AtomicIntegerFieldUpdater.newUpdater(test19.class,"date");
          updater.set(this,2);
      }
      ```

      - 除此之外还支持引用的更新

    - 数组

      - 只支持Integer、Long、Reference

        ```java
        AtomicIntegerArray array = new AtomicIntegerArray(10);
        AtomicIntegerArray array2 = new AtomicIntegerArray(new int[]{2,3,4});
        ```

    - 复合变量

  - 原子变量一种更好的volatile

    - 之前有个一个类含有两个方法用来获取和设置数字大小范围的下界和上界,可以通过原子类实现该类

      ```java
      public class CASNumberRange {
          private static class IntPair{
             final int upper;
             final int down;
      
              private IntPair(int upper, int down) {
                  this.upper = upper;
                  this.down = down;
              }
      
              public int getUpper() {
                  return upper;
              }
      
              public int getDown() {
                  return down;
              }
          }
      
          AtomicReference<IntPair> reference=new AtomicReference<>(new IntPair(0,0));
      
          public boolean setUpper(final int upper)
          {
              while (true)
              {
      
                  IntPair pair = reference.get();
                  if (upper<pair.getDown())
                  {
                      throw new RuntimeException();
                  }
      
                  if (reference.compareAndSet(pair,new IntPair(upper,pair.getDown())))
                  {
                      return true;
                  }
              }
          }
      
      }
      ```

      - 使用一个原子引用，每次更新upper和down时采用直接创建一个新的对象，并直接对对象引用更新

  - 性能比较

    ![image-20230307215235644](D:\note\多线程\JUC并发编程.assets\image-20230307215235644.png)

    - 在高竞争的情况下，原子类只会加剧竞争
  
- ## 非阻塞算法

  - 非阻塞的栈（使用原子变量实现）

    ```java
    public class CASStack {
    
        private  AtomicReference<Node<Integer>> stack= new AtomicReference<>();
    
        @Data
        private class Node<T> {
    
            T val;
            Node<T> next;
        }
    
    
        public void push(Integer i)
        {
            Node<Integer> head = new Node<>();
            head.val=i;
            Node<Integer> node;
            do {
                node = stack.get();
                head.next= node;
            }while (!stack.compareAndSet(node,head));
        }
    
        public Integer pop()
        {
            Node<Integer> node;
            Node<Integer> nextHead;
    
            do {
                node = stack.get();
                nextHead = node.getNext();
            }while (!stack.compareAndSet(node,nextHead));
            return node.getVal();
        }
    
    }
    ```

  - 非阻塞的队列

    - 队列的入队操作一共有四步

      ```java
      1-获取tail
      2-获取tail的next
      3-修改tail的next
      4-修改tail
      ```

      - 当执行修改tail的next时存在三种情况
        - tail还是之前获取的tail，一切正常,可以继续执行
        - tail不再是之前的tail,代表已经有线程执行完第四步了,此时的尾部是错误的
          - 需要重新获取tail
        - tail的next不再是null，代表已经有线程执行完第三步了
          - 两种措施
            - 直接进入第四步帮助另一个线程执行（ConcurrentLinkedQueue采用方式）
            - 操作失败

    - 示例

      ```java
      public class CASQueue<T> {
      
          private static class Node<T> {
              T val;
              AtomicReference<Node<T>> next=new AtomicReference<>();
              private Node(T val) {
                  this.val = val;
              }
          }
      
          private AtomicReference<Node<T>> head=new AtomicReference<>();
          private AtomicReference<Node<T>> tail=new AtomicReference<>();
      
      
          public CASQueue() {
              head.set(null);
              tail.set(null);
          }
      
          public void put(T t){
      
              Node<T> node = new Node<>(t);
      
      
              if (tail.get()==null)
              {
                  tail.set(node);
                  head.set(node);
                  return;
              }
              do {
                  Node<T> oldTail=tail.get();
                  Node<T> oldNext=oldTail.next.get();
                  //保证tail暂时没变,如果tail变了直接进入下一次重新获取
                  if (oldTail==tail.get())
                  {
                      //tail的next不为null,代表已经有线程更新了tail的next
                         if (oldTail.next.get()!=null)
                         {
                             //保证oldTail依旧没变
                             tail.compareAndSet(oldTail,oldNext);
                         }
                         //一切正常
                         else {
                             //CAS更新
                             if (oldTail.next.compareAndSet(null,node)) {
                                 //如果有其他线程帮忙推进了tail节点导致该操作失败也无需重试
                                 tail.compareAndSet(oldTail,node);
                                 break;
                             }
                         }
                  }
              }while (true);
          }
      
      
          public void print()
          {
              Node<T> node = head.get();
              while (node!=null)
              {
      
                  System.out.println(node.val+" ");
                  node=node.next.get();
              }
      
              System.out.println();
          }
      }
      
      ```

  - 原子的域的更新器

    - 更新器提供的原子性更弱一点
    - 在ConcurrentLinkedQueue中采用了更新器来更新next域，虽然有点繁琐但是提高了性能（去除了 AtomicReference<Node<T>> next的创建）

  - ABA问题

    - 线程A查看V的值为A之后线程B将值修改为B后其他线程由将其修改为A，A此时再来查看会发现值没有变化，但是其实中间已经经历了A->B->A的过程

    - 严重后果

      ```java
      小牛取款，由于机器不太好使，多点了几次取款操作。后台threadA和threadB工作，
      此时threadA操作成功(100->50)，threadB阻塞。正好牛妈打款50元给小牛(50->100)，
      threadC执行成功，之后threadB运行了，又改为(100->50)。
      牛气冲天，lz钱哪去了？？？
      ————————————————
      版权声明：本文为CSDN博主「oldRose啊」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
      原文链接：https://blog.csdn.net/m0_37587418/article/details/120802358
      ```

      - 解决方法
        - 每次不只是更新引用同时更新一个版本号（修改的版本），使用AtomicStampedReference可以同时更新版本号和引用,除此之外还有AtomicMarkableReference用于同时一个bool值和引用

  - 

