# JUC并发编程（11）-性能和可伸缩性

- ## 性能和可伸缩性

  - 性能可以通过多个指标衡量，可伸缩性指的是当计算资源增加时（如CPU核心增加）程序的吞吐量或者处理能力也会提高
  - 可伸缩性的提高通常会造成性能的损失
  - 评估各种性能权衡因素
    - 以测试为基准，不要猜测

- ## Amdahl定律

  - 加速比

    - 加速比=同一个任务在单处理器系统和并行处理器系统中运行消耗的时间的比率

  - Amdahl定律

    - f为只能串行执行的程序部分,N为CPU逻辑处理器数量

    - 加速比<=1/(f+(1-f)/N)
    - 在所有并发程序中都包含了串行部分

- ## 线程引入的开销

  - 上下文切换

    - 上下文切换可能会导致缓存的缺失（重新从主存中读取）
    - 阻塞策略（当一个线程堵塞时JVM采取的措施）
      - 通常为线程挂起

  - 内存同步

    - 现代的JVM能够通过优化去掉一些不需要竞争的锁

      - 局部变量锁

        ```java
        synchronized (new Object())
        {
        
        }
        ```

      - 一些更加完备的JVM能够通过逸出分析找出不会被发布到堆中的本地对象引用，从而优化这些对象相关的锁

        ```java
        void test()
        {
            Vector<Integer> vector = new Vector<>();
            vector.add(1);
            vector.add(2);
            vector.add(3);
        }
        ```

        - 以上代码vector的add方法携带的锁是无意义的可以优化掉

      - 锁粒度粗化

        ```java
        void test()
        {
            Vector<Integer> vector = new Vector<>();
            //synchronized (vector)
            //{
                vector.add(1);
                vector.add(2);
                vector.add(3);
            //}
        }
        ```

        - 相同的锁重复获取也可以被简化为只获取一次

  - 阻塞

    - JVM对于阻塞的线程还可以采用自旋的方式（不断循环获取锁直到成功为止），对于频繁但是时间较短的持有锁的程序采用这种方式比直接线程挂起更好，JVM也可以通过历史等待时间来决定使用哪种方式，但是大部分JVM仍然采用线程挂起方式

- ## 减少锁的竞争

  - 在并发程序中可伸缩性的最主要威胁就是独占式的资源锁

  - 降低锁的竞争程度方式

    - 减少锁的持有时间
    - 降低请求频率
    - 使用带有协调机制的独占锁，这些机制允许更高的并发性

  - 缩小锁的范围

    - 只在有必要对共享变量加锁的时候加锁

  - 减小锁的粒度

    - 通过锁分解或者锁分段（ConcurrentHashMap）的形式，然而使用的锁越多，导致死锁的概率也越高

      ```java
      //改造前
      public class LockTest {
      
          private final Set<Integer> set=new HashSet<>();
          private final Set<Integer> set2=new HashSet<>();
      
          public synchronized void addSet(Integer a)
          {
              set.add(a);
          }
      
          public synchronized void addSet2(Integer a)
          {
              set2.add(a);
          }
      }
      //改造后
      public class LockTest {
      
          private final Set<Integer> set=new HashSet<>();
          private final Set<Integer> set2=new HashSet<>();
      
         public  void addSet(Integer a)
         {
             synchronized (set)
             {
                 set.add(a);
             }
             
         }
      
         public synchronized void addSet2(Integer a)
         {
             synchronized (set2)
             {
                 set2.add(a);
             }
         }
      }
      ```

    - 锁分段

      - 在ConcurrentMap中采用了16个锁来保护Map,一个锁负责N/16个散列桶，第N个散列通由N%16来保护，这技术也能让ConcurrentHashMap支持16个线程的并发
      - 劣势
        - 获取多个锁要比获取一个独占锁的要求更高，通常执行一个操作只需要一个锁，但在某种情况下需要加锁整个容器就需要获取所有锁

  - 避免热点域

    - HashMap中调用size方法，size方法实现可以是每次调用时枚举所有Map中的值，也可以维护一个计数器，在put方法和remove方法调用时增加或者减少计数器，但是在ConcurrentHashMap中如果采用这种方式每次增加或者减少计数器都需要更新这个共享的计数器，计数器也就变成了热点域，每次增加或者删除都会访问该计数器，所以ConcurrentMap采用每个段都使用一个计数器计数

  - 一些替代独占锁的方法

    - 放弃使用独占锁，如使用并发容器或者读写锁（见15章）、不可变对象、原子变量

  - 监测CPU的利用率

    - 如果CPU利用率不均匀表示大多数计算都是由同一组线程完成，需要找出程序的并行性
    - 原因
      - 负载不充足
        - 测试负载不够，上强度
      - IO密集
        - 通过iostat或者perfmon判断某个程序是否是IO密集
      - 外部限制
        - 可能由于Web服务或者数据库的性能不够
      - 锁竞争
        - 通过线程转储分析

  - 拒绝对象池

    - 通常对象分配操作比同步（对象池获取需要同步）的开销更小

  - 比较Map的性能

    - 单线程情况下，ConcurrentHashMap只比HashMap好一点，但是并发条件下，ConcurrentHashMap更优，HashMap只有一个锁（全体锁），ConcurrentHashMap在读情况下不加锁，写情况下采用分段锁

  - 减少上下文切换开销

    - 

