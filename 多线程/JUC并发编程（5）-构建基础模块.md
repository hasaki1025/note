# JUC并发编程（5）-构建基础模块

- ## 同步容器类

  - 同步容器类的问题

    - 错误示例

      ```java
      public static Object getLast(Vector list)
      {
          return list.get(list.size()-1);
      }
      
      public static Object deleteLast(Vector list)
      {
          return list.remove(list.size()-1);
      }
      /**
      同时启动两个线程，一个获取最后的元素，一个删除最后的元素，在并发的情况下可能会出现
      get方法先获取到最后一个元素的索引值然后另一个线程删除了最后一个元素，此时再进行获取就会报错（索引值已失效）
      **/
      ```

      - 对于多个同步容器调用的方法组合属于复合操作，需要加锁保护

  - 迭代器和ConcurrentModificationException

    - 当使用迭代器遍历一个集合的同时删除或者添加元素，则会抛出ConcurrentModificationException。

    - 隐藏迭代器

      ````java
      public class HiddenIterator {
          @GuardedBy("this") private final Set<Integer> set = new HashSet<Integer>();
      
          public synchronized void add(Integer i) {
              set.add(i);
          }
      
          public synchronized void remove(Integer i) {
              set.remove(i);
          }
      
          public void addTenThings() {
              Random r = new Random();
              for (int i = 0; i < 10; i++)
                  add(r.nextInt());
              //再这里直接输出toString方法会自动遍历set集合
              System.out.println("DEBUG: added ten elements to " + set);
          }
      }
      ````

      - 类似的方法由hashCode\equals或者containsAll、removeAll、retainAll等方法

- ## 并发容器

  - ConcurrentHashMap
    - ConcurrentHashMap并不是在每一个方法上都在同一个锁上同步，而是采用一种粒度更细的加速机制来实现更大程度的共享，这种机制称为分段锁，在这种同步机制下任意数量的读取线程可以并发的访问Map，读线程和写线程都可以并发的访问Map
    - ConcurrentHashMap的迭代器不会抛出ConcurrentModificationException
    - 额外的原子操作
      - ConcurrentHashMap能被加锁执行独占访问，所以无法使用客户端加锁来创建新的原子操作，但是大部分复合操作在ConcurrentHashMap都有实现
    
  - CopyOnWriteArrayList
  
    - 它相当于线程安全的ArrayList。和ArrayList一样，它是个可变数组；但是和ArrayList不同的时，它具有以下特性：
      1. 它最适合于具有以下特征的应用程序：List 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
      2. 它是线程安全的。
      3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
      4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等操作。
      5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。
  
    -  CopyOnWriteArrayList包含了成员lock。每一个CopyOnWriteArrayList都和一个监视器锁lock绑定，通过lock，实现了对CopyOnWriteArrayList的互斥访问。
  
      ```java
      final transient Object lock = new Object();
      ```
  
    - CopyOnWriteArrayList的“动态数组”机制 -- 它内部有个“volatile数组”(array)来保持数据。在“添加/修改/删除”数据时，都会新建一个数组，并将更新后的数据拷贝到新建的数组中，最后再将该数组赋值给“volatile数组”。
  
      ```java
      private transient volatile Object[] array;
      
          /**
           * Gets the array.  Non-private so as to also be accessible
           * from CopyOnWriteArraySet class.
           */
          final Object[] getArray() {
              return array;
          }
      
      //add方法
      public void add(int index, E element) {
              synchronized (lock) {
                  Object[] es = getArray();
                  int len = es.length;
                  if (index > len || index < 0)
                      throw new IndexOutOfBoundsException(outOfBounds(index, len));
                  Object[] newElements;
                  int numMoved = len - index;
                  //插入到末尾，直接复制整个数组
                  if (numMoved == 0)
                      newElements = Arrays.copyOf(es, len + 1);
                  else {
                      //分段复制
                      newElements = new Object[len + 1];
                      //前一段复制
                      System.arraycopy(es, 0, newElements, 0, index);
                      //后一段复制
                      System.arraycopy(es, index, newElements, index + 1,
                                       numMoved);
                  }
                  //插入元素赋值
                  newElements[index] = element;
                  //写回到数组
                  setArray(newElements);
              }
          }
      ```
  
  - 阻塞队列和生产者消费者模式
  
    - 堵塞队列提供了可堵塞的put和take方法，当队列中无元素时，take方法会堵塞，队列满时，put方法会堵塞，同时堵塞队列支持定时的offer和poll方法
  
      ![image-20230303143528144](C:\Users\YX\AppData\Roaming\Typora\typora-user-images\image-20230303143528144.png)
  
    - 堵塞队列可以有界也可以无界
  
    - put方法源码
  
      ```java
      public void put(E e) throws InterruptedException {
              Objects.requireNonNull(e);
          //可重入锁
              final ReentrantLock lock = this.lock;
              lock.lockInterruptibly();
              try {
                  while (count == items.length)
                      //持续等待
                      notFull.await();
                  //元素添加
                  enqueue(e);
              } finally {
                  lock.unlock();
              }
          }
      ```
  
      - 可重入锁
  
        ```java
        Java并发包下的ReentrantLock(可重入锁)，实现了高并发下的加锁机制，原理是通过AQS(
        AbstractQueuedSynchronizer，抽象的队列同步器)中循环调用CAS(compareAndSet,原子性操作)操作来实现加锁，它的性能比较好是因为尽最大努力的避免了使线程进入内核态的阻塞状态，Java并发包下的很多锁功能，本质都是使用了AQS(AbstractQueuedSynchronizer，抽象的队列同步器)，该实现的作者是Doug Lea，ThreadPoolExecutor线程池也是这位外国大佬的杰作。
        ————————————————
        版权声明：本文为CSDN博主「头顶假发」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
        原文链接：https://blog.csdn.net/lt_xiaodou/article/details/126506187
        ```
  
      - Lock类
  
        ```java
        二、Lock和syncronized的区别
        synchronized是Java语言的关键字。Lock是一个接口。
        synchronized不需要用户去手动释放锁，发生异常或者线程结束时自动释放锁;Lock则必须要用户去手动释放锁，如果没有主动释放锁，就有可能导致出现死锁现象。
        lock可以配置公平策略,实现线程按照先后顺序获取锁。
        提供了trylock方法 可以试图获取锁，获取到或获取不到时，返回不同的返回值 让程序可以灵活处理。
        lock()和unlock()可以在不同的方法中执行,可以实现同一个线程在上一个方法中lock()在后续的其他方法中unlock(),比syncronized灵活的多。
        ————————————————
        版权声明：本文为CSDN博主「向上的狼」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
        原文链接：https://blog.csdn.net/m0_50370837/article/details/124471888
        ```
  
        - lock方法
  
          ```java
              /**
          获取锁。 如果锁不可用，则当前线程出于线程调度的目的将被禁用，并处于休眠状态，直到获得锁。 实现注意事项 Lock实现可以检测到锁的错误使用，例如会导致死锁的调用，并且在这种情况下可能会抛出(未检查的)异常。该Lock实现必须记录环境和异常类型。
               */
              void lock();
          
          //使用
          Lock lock = ...;
          lock.lock();
          try{
              //处理任务
          }catch(Exception ex){
               
          }finally{
              lock.unlock();   //释放锁
          }
          ```
  
        - tryLock
  
          ```java
              /**
           只有当锁在调用时是空闲的，才会获取锁。 如果锁可用，则获取锁，并立即返回值为true。如果锁不可用，则此方法将立即返回值为false。 这个方法的典型用法是:
               * Lock lock = ...;
               * if (lock.tryLock()) {
               *   try {
               *     // manipulate protected state
               *   } finally {
               *     lock.unlock();
               *   }
               * } else {
               *   // perform alternative actions
               * }}
               这种用法确保锁在被获取时被解锁，并且在没有被获取时不会尝试解锁。 返回值: 如果获得了锁，则为True，否则为false
               */
              boolean tryLock();
          
          ```
  
          - 除此之外trylock同时提供了一个定时方法
  
            ```java
             boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
            
            ```
  
        - unLock方法（释放锁）
  
    - 双端队列和工作密取
      - java6增加了两种容量类型，Deque(双端队列)和BlockingDeque，可以从两端进行入队和出队
      - 工作密取
        - 每个消费者都拥有自己的工作队列，当自己的工作队列中没有任务会从其他的工作队列的末尾获取任务
        - 降低各个线程之间的竞争
  
  - 阻塞方法和中断方法
  
    - BlockingQueue的put方法和take方法等其他堵塞方法（如sleep）会抛出受检查异常InterruptedException，如果一个方法会抛出InterruptedException则代表这是一个阻塞方法，如果这个方法被中断，那么它将会努力终止堵塞状态
  
    - 中断只是个协作机制，一个线程不能强制其他线程中止当前的操作只能设置一个中断位，如果被中断的线程检查到了这个中断位置才会采取措施（可能会在某个位置停止也可能会屏蔽该中断），一般措施有两种
  
      - 传递InterruptedException
  
        - 传递给方法的调用者（如果达到线程顶部任选择抛出则抛出到JVM，jvm一般会选择终止程序）
  
      - 恢复中断
  
        - catch该异常并处理(恢复中断状态)
  
          ```java
          public class InterrWorker implements Runnable {
              @Override
              public void run() {
                      try {
                          while (true) {
                              sleep(1000);
                              System.out.println("running");
                          }
                      } catch (InterruptedException e) {
                          //catch后中断取消
                          System.out.println("interrupt...");
                          //false
           System.out.println(Thread.currentThread().isInterrupted());
                          //再次恢复中断
                          Thread.currentThread().interrupt();
                          //true
           System.out.println(Thread.currentThread().isInterrupted());
                      }
          
              }
          }
          ```
  
  - 同步工具类
  
    - 闭锁
  
      - 闭锁相当于一扇门，在闭锁到达结束状态（通过程序设定）前，闭锁一直时关闭的，所有调用闭锁的await方法的线程都会被阻塞直到闭锁打开，打开后闭锁将不会再关上
  
        - 常用场景
  
          - 等待所有资源都被初始化后才继续执行
          - 确保每个服务依赖的所有服务开启后才启动
  
        - 使用方法
  
          - 闭锁中含有一个计数器，计数器初始化为一个正数，表示需要等待的时间数量，countDown方法可以使计数器减一，表示一个事件的完成，，await方法表示阻塞直到计数器归0
  
          ```java
          public class TestCountDownLatch {
              public static void main(String[] args) {
                  CountDownLatch startLatch = new CountDownLatch(1);
                  for (int i = 0; i < 10; i++) {
                      new Thread(()->{
                          try {
                              System.out.println("waiting startLatch...");
                              startLatch.await();
                              System.out.println("running...");
                          } catch (InterruptedException e) {
                              throw new RuntimeException(e);
                          }
                      }).start();
                  }
          
                  new Thread(()->{
                      try {
                          System.out.println("3秒后开锁");
                          sleep(3000);
                          startLatch.countDown();
                      } catch (InterruptedException e) {
                          throw new RuntimeException(e);
                      }
                  }).start();
              }
          }
          
          ```
  
    - FutureTask
  
      - FutureTask实现了Runnable接口和Future接口，Future接口可以保存线程执行后返回的结果，Runnable表示任务，FutureTask相当于一个类身兼两职
  
      - 可以将需要长时间计算的任务通过FutureTask执行，到需要的时候才提取
  
        ```java
        FutureTask<Integer> task = new FutureTask<>(() -> {
        
            for (int i = 0; i < 5; i++) {
                sleep(1000);
                System.out.println("running...");
            }
            return 2;
        });
        Thread thread = new Thread(task);
        thread.start();
        System.out.println("waiting futureTask Result...");
        try {
            System.out.println("Result: "+task.get());
            //无论任务中抛出了什么异常最终都会封装到ExecutionException中并在get方法中重新抛出
        } catch (InterruptedException | ExecutionException e) {
            throw new RuntimeException(e);
        }
        /**
        waiting futureTask Result...
        running...
        running...
        running...
        running...
        running...
        Result: 2
        **/
        ```
  
    - 信号量
  
      - semaphore中管理着一组虚拟的许可，许可的初始数量通过构造方法传递，线程执行操作前需要先获取到一个许可，并在使用后释放，如果没有许可将会一直阻塞。
  
      - semaphore可以实现有限资源池，可以将任何一种容器包装成有界堵塞容器
  
        ```java
        public class TestSem {
            private final Set<Integer> set;
            private final Semaphore semaphore;
        
            public TestSem(int bound) {
                this.set = new CopyOnWriteArraySet<>();
                this.semaphore = new Semaphore(bound);
            }
            
            public boolean add(Integer e) throws InterruptedException {
                semaphore.acquire();
                boolean isAdded=false;
                try{
                    isAdded = set.add(e);
                    return isAdded;
                }finally {
                    if (isAdded)
                    {
                        semaphore.release();
                    }
                }
            }
        }
        
        ```
  
    - 栅栏
  
      - 类似于闭锁，能够阻塞一组线程直到所有线程调用await方法，所有线程必须都到达栅栏才能开门，CyclicBarrier是栅栏的一种，不同于闭锁，CyclicBarrier可以反复开启和关闭（闭锁只能开一次之后就是一直打开的状态，CyclicBarrier可以打开后再次关闭）
  
      - 如果CyclicBarrier调用的await超时或者阻塞的线程被中断，栅栏则被视为被打破，所有被阻塞的await都将抛出BrokenBarrierException
  
      - 如果所有线程通过栅栏则await方法会返回一个唯一的到达索引值，也就是当前线程的到达索引，其中阻塞线程数- 1表示第一个到达，0表示最后一个到达
  
      - 同时CyclicBarrier的构造方法上可以传递一个Runnable对象，在栅栏打开时将调用该Runnable
  
        ```java
        CyclicBarrier barrier = new CyclicBarrier(10, () -> {
            System.out.println("start barrier..");
        });
        
        for (int i = 0; i < 10; i++) {
            Worker worker = new Worker(10 - i) {
                @Override
                public void work() {
                    super.work();
                    try {
                        int await = barrier.await();
                        System.out.println(Thread.currentThread().getName()+"到达索引值:"+await);
                    } catch (InterruptedException | BrokenBarrierException e) {
                        throw new RuntimeException(e);
                    }
                }
            };
            Thread thread = new Thread(worker);
            thread.setName("thread"+i);
            thread.start();
        }
        
        //worker实现
        public class Worker implements Runnable{
        
            private final int workTime;
        
            public Worker(int workTime) {
                this.workTime = workTime;
            }
        
            @Override
            public void run() {
                try {
                    sleep(workTime* 1000L);
                    work();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            public void work()
            {
                //NOOP
            }
        
        }
        
        最终结果
        start barrier..
        thread1到达索引值:1
        thread5到达索引值:5
        thread3到达索引值:3
        thread2到达索引值:2
        thread0到达索引值:0
        thread4到达索引值:4
        thread6到达索引值:6
        thread8到达索引值:8
        thread9到达索引值:9
        thread7到达索引值:7
        ```
  
  - 构建高效且可伸缩的结果缓存
  
    - 假设现在有一个计算需要耗费很长时间才能得到结果，需要使用一种缓存用来保存之前计算的结果，减少平均响应的时间
  
    - 第一种方案：采用HashMap保持计算结果
  
      ```java
      public class Memoizer1 {
          private final HashMap<Integer,Integer> cache=new HashMap<>();
      
      
          public synchronized int compute(int arg) throws InterruptedException {
              if (cache.containsKey(arg))
              {
                  return cache.get(arg);
              }
              cache.put(arg,computeValue(arg));
              return cache.get(arg);
          }
      
          private int computeValue(int arg) throws InterruptedException {
              System.out.println("computeing...");
              sleep(5000);
              return arg+1;
          }
      }
      
      ```
  
      - 虽然保证了线程安全性但是同一时间只有一个线程访问HashMap效率较低
  
    - 第二种方案：采用ConcurrentHashMap代替HashMap
  
      ```java
      public class Memoizer2 {
          private final ConcurrentHashMap<Integer,Integer> cache=new ConcurrentHashMap<>();
      
      
          public  int compute(int arg) throws InterruptedException {
              if (cache.containsKey(arg))
              {
                  return cache.get(arg);
              }
              cache.put(arg,computeValue(arg));
              return cache.get(arg);
          }
      
          private int computeValue(int arg) throws InterruptedException {
              System.out.println("computeing...");
              sleep(5000);
              return arg+1;
          }
      }
      
      ```
  
      - 虽然可以并发的访问ConcurrentHashMap，但是存在重复计算的可能，比如一个线程刚刚判断Map中没有想要的值后开始计算，但立刻有线程已经计算出了值并放入到Map中，之后线程计算的值虽然无误但是时没必要的计算
      - 也即是说其他线程需要知道当前关于该arg的计算有没有开始而不是是否已经结束，如果该arg的计算已经开始则没必要计算只需要等待结果即可
  
    - 第三种方案：采用Future作为value
  
      ```java
      public class Memoizer3 {
          private final ConcurrentHashMap<Integer, Future<Integer>> cache=new ConcurrentHashMap<>();
      
      
          public  int compute(int arg,ExecutorService executorService) throws InterruptedException, ExecutionException {
              if (cache.containsKey(arg))
              {
                  return cache.get(arg).get();
              }
              Future<Integer> future = executorService.submit(() -> {
                  return computeValue(arg);
              });
              cache.put(arg,future);
              return cache.get(arg).get();
          }
      
          private int computeValue(int arg) throws InterruptedException {
              System.out.println("computeing...");
              sleep(5000);
              return arg+1;
          }
      }
      
      ```
  
      - future能够很好的表示计算的任务这一个概念，如果map中存在则代表任务已经开始
      - 但是该方案仍然存在一些并发问题，也就是先判断后写入的操作并不是原子操作，中间仍然存在空隙，也就是说仍然存在重复计算的可能。
  
    - 方案4：使用ConcurrentHashMap提供的原子操作
  
      ```java
      public class Memoizer4 {
          private final ConcurrentHashMap<Integer, Future<Integer>> cache=new ConcurrentHashMap<>();
      
      
          public  int compute(int arg, ExecutorService executorService) throws InterruptedException, ExecutionException {
              return cache.computeIfAbsent(arg,
                      k -> {
                          return executorService.submit(() -> {
                              return computeValue(arg);
                          });
                      }
              ).get();
          }
      
          private int computeValue(int arg) throws InterruptedException {
              System.out.println("computeing...");
              sleep(5000);
              return arg+1;
          }
      }
      
      
      
      ```
  
      - 到此为止关于Map的实现已经几乎完美但是仍存在问题
        - 缓存污染问题
          - Map中存储的是future，如果future在执行过程中出现错误，map则不能保持该值并且需要告诉调用者该计算已失败
          - 如果计算被取消同样需要在map中移除该值
          - 如果计算的结果具有时效性或者说长期不使用的缓存需要清理等问题，需要Map定时扫描是否存在逾期的缓存
  

