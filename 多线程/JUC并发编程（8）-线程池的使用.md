# JUC并发编程（8）-线程池的使用

- ## 在任务和执行策略之间的隐性耦合

  - 虽然Executor框架将任务的执行和任务的提交解耦合，但是并不是所有任务都适用于所有执行策略（也就是不太适用于放在线程池中运行），如以下几种

    - 依赖性任务
      - 一个任务的执行依赖于另一个任务的执行，虽然大部分任务的执行的都是独立的，但仍存在这种依赖性问题
    - 使用线程封闭性的任务
      - 对于某些任务在单线程情况（或者单线程线程池中）下可以正常的运行，但是放在并发的线程池中可能会出现并发问题
    - 对响应时间敏感的任务
      - 如果任务的执行时间过长或者线程池的数量过小，很容易出现响应性不敏感的问题
    - 使用ThreadLocal的任务
      - 当一个线程使用了ThreadLocal并结束执行后可能会再次被线程池利用，但其中的ThreadLocal仍然采用之前的值，只有当ThreadLoacl的生命周期和任务的生命周期相同时才不会有问题

  - 只有当任务都是同一类任务并且相互独立时线程池的性能才能达到最佳，时间较长和时间较短的任务都放在同一个线程池中，除非线程池够大否则都可能会造成阻塞，如果提交的任务依赖于其他任务，除非线程池无限大否则都可能造成死锁

  - 线程饥饿死锁

    - 在线程池中，如果一个任务在执行过程中提交了另一任务到同一个线程池并且等待该任务的执行完成，这种情况极有可能发生死锁

      - 第二个任务在队列中排在第一个任务的后面等待第一个任务的执行完成，而第一个任务等待第二个任务的执行完成

        ```java
        public class test01 {
            private final static ExecutorService service= Executors.newSingleThreadExecutor();
        
            public static void main(String[] args) {
                service.submit(()->{
        
                    System.out.println("任务一执行");
                    Future<?> future = service.submit(
                        () -> {
                            System.out.println("任务2执行");
                        });
        
                    System.out.println("等待任务2的执行完成");
                    try {
                        future.get();
                    } catch (InterruptedException | ExecutionException e) {
                        throw new RuntimeException(e);
                    }
                });
            }
        }
        
        ```

      - 在更大的线程池中如果所以正在执行的任务线程都由于等待其他仍处于工作队列中的任务而堵塞那么会发生同样的问题

  - 运行时间较长的任务

    - 执行时间过长即使不出现死锁，响应性也会很低
    - 通过限时的堵塞方法能够缓解该问题，如果等待超时可以将该任务标记为失败或者重新放回队列中以便之后执行。
    - 如果线程池中总是充满了阻塞的任务也可能是线程池的规模太小

- ## 设置线程池大小

  - 线程池大小决定因素
    - CPU数量、内存大小、任务是IO密集型还是CPU密集型还是二者皆可、是否需要JDBC连接这种稀缺资源
  - 常用判断
    - CPU密集型
      - CPU个数为N时，线程池大小可以设置为N+1
    - IO密集和CPU+IO
      - N为CPU核心数量,U为CPU目标利用率，W/C=等待时间/CPU计算时间
      - 线程数量=U**N*(1+W/C)
    - 含有特殊资源的任务（JDBC连接池等）
      - 线程数量=总资源数量/每个任务所需的资源
    - 可以通过Runtime.getRuntime().availableProcessors()获取到CPU逻辑处理器数量

- ## 配置ThreadPoolExecutor

  - 线程的创建和销毁

    - 见第六章笔记

  - 管理队列任务

    - 基本的任务排队顺序一般是：无界队列、有界队列、同步移交

      - 无界队列
        - newFixedThreadPool和newSingleThreadExecutor在默认情况下采用无界的阻塞队列，队列可以无限增长，这有可能会导致内存耗尽，常用为LinkedBlockingQueue
      - 有界队列
        - 常用的有两类，一类是遵循FIFO原则的队列如ArrayBlockingQueue与有界的LinkedBlockingQueue，另一类是优先级队列如PriorityBlockingQueue。PriorityBlockingQueue中的优先级由任务的Comparator（任务实现Comparator接口）决定。 
          **使用有界队列时队列大小需和线程池大小互相配合**，线程池较小有界队列较大时可减少内存消耗，降低cpu使用率和上下文切换，但是可能会限制系统吞吐量。

      - 同步移交

        ```java
        如果不希望任务在队列中等待而是希望将任务直接移交给工作线程，可使用SynchronousQueue作为等待队列。SynchronousQueue不是一个真正的队列，而是一种线程之间移交的机制。要将一个元素放入SynchronousQueue中，必须有另一个线程正在等待接收这个元素。只有在使用无界线程池或者有饱和策略时才建议使用该队列。
        ————————————————
        版权声明：本文为CSDN博主「BoltBear」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
        原文链接：https://blog.csdn.net/woaini886353/article/details/124380929
        ```

        - newCacheThreadPool采用的就是这种机制，当提交一个任务时
          - 有空余线程，直接移交执行
          - 没有空余线程，但是当前线程数未达到最大值，创建新线程并移交
          - 已达到最大值，根据饱和策略拒绝该任务

  - 饱和策略

    - 饱和策略可以通过setRejectedExecutionHandler方法设置，JDK提供了4种预设的拒绝策略

      ```java
      public interface RejectedExecutionHandler {
          void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
      }
      ```

      - AbortPolicy(直接抛出异常拒绝)

        ```java
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
        ```

      - CallerRunsPolicy

        ```java
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                r.run();
            }
        }
        ```

        - 如果线程池没有关闭，则在任务提交的线程中再次执行该任务

      - DiscardOldestPolicy

        ```java
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            if (!e.isShutdown()) {
                e.getQueue().poll();
                e.execute(r);
            }
        }
        ```

        - 如果线程池没有关闭，抛弃线程池队列中的头部并在提交任务的线程中执行该任务

      - DiscardPolicy

        ```java
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
        	//NOOP
        }
        ```

        - 什么也不做

  - 通过信号量控制任务提交速率

    ```java
    public class BoundExecutor {
    
        //没有显式设置饱和策略则默认使用AbortPolicy
        private final ExecutorService service=
            new ThreadPoolExecutor(13,100,1, TimeUnit.SECONDS,new LinkedBlockingQueue<>());
        private final Semaphore semaphore;
    
        public BoundExecutor(int bound) {
            this.semaphore = new Semaphore(bound);
        }
    
        public void execute(Runnable r) throws InterruptedException {
            semaphore.acquire();
            try {
                service.execute(()->{
                    try {
                        r.run();
                    }
                    finally {
                        semaphore.release();
                    }
                });
            } catch (RejectedExecutionException e) {
                //发生饱和策略
                semaphore.release();
            }
    
        }
    }
    
    ```

  - 线程工厂

    - 线程池可以自定义线程工厂，只需要实现ThreadFactory
    - 如果在应用程序中需要利用安全策略来控制对某些特殊代码库的访问权限，可以通过Executors中的privilegedThreadFactory来定制自己的线程工厂(该方法已在java17弃用)

  - 调用构造方法后的线程池

    - 通过构造方法构造完成后可以通过set方法对线程池重新配置

    - 如果不希望构造完成的线程池配置得到修改可以通过Executors的unconfigurableExecutorService方法包装并返回一个无法改变配置的线程池对象

      ```java
      private final static ExecutorService service= 
          Executors.unconfigurableExecutorService(
          new ThreadPoolExecutor(13,100,1, TimeUnit.SECONDS,new LinkedBlockingQueue<>()));
      ```

      - unconfigurableExecutorService方法

        ```java
        public static ExecutorService unconfigurableExecutorService(ExecutorService executor) {
            if (executor == null)
                throw new NullPointerException();
            return new DelegatedExecutorService(executor);
        }
        
        //DelegatedExecutorService实际上就是外部传入的线程池的简单包装，但是采用private修饰导致无法实例化并且无法只能访问提供的public方法
        private static class DelegatedExecutorService
                    implements ExecutorService {
                private final ExecutorService e;
        ```

- ## 扩展ThreadPoolExecutor

  - ThreadPoolExecutor提供了几个可以继承扩展的方法

    - beforeExecute

      ```java
      /**
      在给定线程中执行给定Runnable之前调用的方法。此方法由将执行任务r的线程t调用，可用于重新初始化ThreadLocals或执行日志记录。 这个实现什么也不做，但是可以在子类中定制。注意:要正确嵌套多个重写，子类通常应该调用super。beforeExecute在此方法的末尾。
      形参: T -将运行任务r的线程r -将被执行的任务
      **/
      protected void beforeExecute(Thread t, Runnable r) { }
      ```

    - afterExecutor

      ```java
      
      /**
      无论是正常终止还是通过抛出异常终止都会进入该方法（Error不会）
      ，如果beforeExecute抛出一个RuntimeException异常，则任务不会执行，afterExecute也不会执行
      **/
      protected void afterExecute(Runnable r, Throwable t) { }
      ```

    - terminated

      ```java
      //当执行程序终止时调用的方法。默认实现什么都不做。注意:要正确嵌套多个重写，子类通常应该调用super。在此方法中终止。可以用来释放资源
      protected void terminated() { }
      ```

  - 示例：给线程记录时间

    - 每次任务执行开始前通过ThreadLocal记录开始时间，结束时计算总共时间并添加在sumTime上，nums用于记录执行次数

    ```java
    @Slf4j
    public class MyExecutor extends ThreadPoolExecutor {
    
    
        private final ThreadLocal<Long> startTime=new ThreadLocal<>();
        private final AtomicLong nums=new AtomicLong(0L);
        private final AtomicLong sumTime=new AtomicLong(0L);
    
    
    
        @Override
        protected void beforeExecute(Thread t, Runnable r) {
            startTime.set(System.currentTimeMillis());
        }
        @Override
        protected void afterExecute(Runnable r, Throwable t) {
            long timeSpan=System.currentTimeMillis()-startTime.get();
            long l = nums.incrementAndGet();
            sumTime.addAndGet(timeSpan);
            log.info("第{}次执行为Thread[{}]: 执行时间{},开始时间:{}",l,Thread.currentThread().getName(),timeSpan,startTime.get());
        }
        @Override
        protected void terminated() {
            super.terminated();
        }
    
    }
    
    ```

- ## 递归算法的并行性

  - 