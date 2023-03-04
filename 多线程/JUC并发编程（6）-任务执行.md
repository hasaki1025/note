# JUC并发编程（6）-任务执行

- ## Executor框架

  - Executor基于生产者-消费者模式，Executor任务作为生产者，执行任务的线程作为消费者

    ```java
    public interface Executor {
        void execute(Runnable command);
    }
    
    ```

    - 这种框架模式将任务的执行和提交解耦合

  - 线程池创建

    ```java
    private final ExecutorService executorService=
                new ThreadPoolExecutor(10,100,100, TimeUnit.SECONDS,new LinkedBlockingQueue<>(),new MyThreadFactory(),new handler());
    ```

    - 第一个参数：核心线程数

    - 第二个参数：最大线程数

    - 第三个参数：当当前线程数大于核心线程数时多余的线程的最大存活时间

    - 第四个参数：存活时间单位

    - 第五个参数：任务队列

    - 第六个参数：线程工厂（可选）

    - 第七个参数：任务执行拒绝策略(可选)

      ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210116004240116.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=,size_16,color_FFFFFF,t_70)

  - 任务的执行

    - 在什么线程中执行
    - 按照什么顺序执行（FIFO、优先级等）
    - 多少个线程能够并发执行
    - 有多少个任务等待执行
    - 如果因为负载问题拒绝一个任务的执行应该选择哪个任务
    - 在执行一个任务之前或者之后应该进行哪些动作

  - 线程池种类

    - 固定长度线程池

      ```java
      private final ExecutorService cachedThread1 = Executors.newFixedThreadPool(10);
      //实现
      public static ExecutorService newFixedThreadPool(int nThreads) {
              return new ThreadPoolExecutor(nThreads, nThreads,
                                            0L, TimeUnit.MILLISECONDS,
                                            new LinkedBlockingQueue<Runnable>());
          }
      ```

      - 每提交一个任务就创建一个线程，直到达到线程池的最大规模，之后不会再创建线程

    - 可缓存的线程池

      ```java
      private static final ExecutorService cachedThread2 = Executors.newCachedThreadPool();
      
      public static ExecutorService newCachedThreadPool() {
              return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                            60L, TimeUnit.SECONDS,
                                            new SynchronousQueue<Runnable>());
          }
      ```

      - 如果线程池当前规模超过任务需求则回收空闲的线程，需求增加时则可以创建新的线程，线程池规模不存在限制（Int的最大值）

    - 单线程线程池

      ```java
      private static final ExecutorService cachedThread3 = Executors.newSingleThreadExecutor();
      
      public static ExecutorService newSingleThreadExecutor() {
              return new FinalizableDelegatedExecutorService
                  (new ThreadPoolExecutor(1, 1,
                                          0L, TimeUnit.MILLISECONDS,
                                          new LinkedBlockingQueue<Runnable>()));
          }
      ```

      - 只有一个线程串行执行

    - 固定长度的定时线程池

      ```java
      private static final ExecutorService cachedThread4 = Executors.newScheduledThreadPool(10);
      
      
      public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
          return new ScheduledThreadPoolExecutor(corePoolSize);
      }
      
      public ScheduledThreadPoolExecutor(int corePoolSize) {
          super(corePoolSize, Integer.MAX_VALUE,
                DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
                new DelayedWorkQueue());
      }
      
      public ThreadPoolExecutor(int corePoolSize,
                                int maximumPoolSize,
                                long keepAliveTime,
                                TimeUnit unit,
                                BlockingQueue<Runnable> workQueue) {
          this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
               Executors.defaultThreadFactory(), defaultHandler);
      }
      ```

      - 固定长度但是可以执行定时或者延时任务，无规模限制，默认存活时间为10ms，采用延迟工作队列

  - Executor的生命周期管理方法

    - JVM在所有线程（非守护）终止后才会结束，如果不关闭线程池则JVM无法关闭

    - ExecutorService继承了Executor接口，其中包含了许多生命周期方法

    - 生命周期方法

      ```java
      public interface ExecutorService extends Executor {
          //启动有序关闭，执行之前提交的任务，但不接受新任务。如果已关闭调用，则没有其他影响。 此方法不会等待先前提交的任务完成执行。使用awaitterminate来做到这一点。
          void shutdown();
          
      	//尝试停止所有正在执行的任务，停止等待任务的处理，并返回等待执行的任务列表。 此方法不会等待正在执行的任务终止。使用awaitterminate来做到这一点。 除了尽最大努力尝试停止处理正在执行的任务之外，没有任何保证。例如，典型的实现将通过Thread.interrupt取消，因此任何未能响应中断的任务可能永远不会终止。
         // 返回值: 从未开始执行的任务列表
          List<Runnable> shutdownNow();
          //如果该执行程序已关闭，则返回true。 返回值: 如果此执行程序已关闭，则为
      
          boolean isShutdown();
      //如果关闭后所有任务都已完成，则返回true。注意，除非先调用shutdown或shutdownNow, terminate永远不会为真。 返回值: 如果关闭后所有任务都已完成，则为
          boolean isTerminated();
      //阻塞，直到所有任务在关闭请求后完成执行，或超时发生，或当前线程被中断，以先发生者为准。 形参: Timeout -最大等待时间单位- Timeout参数的时间单位 返回值: 如果该执行程序终止，则为True;如果在终止前超时，则为false 
          boolean awaitTermination(long timeout, TimeUnit unit)
              throws InterruptedException;
          //....
      ```

      - 线程池关闭后提交的任务将交给拒绝策略处理，一般会丢弃任务或者抛出一个RejectedExecutionException

        ```java
        //默认拒绝策略实现
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
        ```

- ## 找出可利用的并行性

  - 携带结果的任务和Future

    - Callable和Future

      - 通过submit提交具有返回值的任务，submit会返回一个Future，通过Future可以取消任务，查看任务的状态，获取任务结果

    - FutureTask

      - java6后线程池可以实现AbstractExecutorService中的newTaskFor方法为Callable、Runnable创建一个FutureTask

        ```java
        //AbstractExecutorService默认实现
        protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
            return new FutureTask<T>(callable);
        }
        ```

        - 该方法是protected方法所以需要自己创建一个AbstractExecutorService的子类才能访问该方法
      
    - CompletionService
    
      - 线程池中提交了一组任务后希望能够计算完成后获得结果，可以将Future保持下来并反复调用get方法，CompletionService提供了类似的工作机制，CompletionService选择将结果放入BlockingQueue中
    
      - ExecutorCompletionService
    
        - ExecutorCompletionService的实现很简单，只是在构造方法中创建一个BlockingQueue（也可以自定义）用于保存计算任务的Future。
    
        - 当计算完成时，调用Future-task的done方法（该方法是在任务完成后的回调方法）。
    
        - 当提交一个任务时将Future封装为QueueingFuture,这是一个FutureTask的子类，在改写子类的done方法将结果放入BlockingQueue中
    
          ```java
          public Future<V> submit(Callable<V> task) {
              if (task == null) throw new NullPointerException();
              //返回一个FutureTask
              RunnableFuture<V> f = newTaskFor(task);
              //封装为QueueingFuture并通过线程池执行
              executor.execute(new QueueingFuture<V>(f, completionQueue));
              return f;
          }
          
          //queueFuture实现
          private static class QueueingFuture<V> extends FutureTask<Void> {
              QueueingFuture(RunnableFuture<V> task,
                             BlockingQueue<Future<V>> completionQueue) {
                  super(task, null);
                  this.task = task;
                  this.completionQueue = completionQueue;
              }
              private final Future<V> task;
              private final BlockingQueue<Future<V>> completionQueue;
              //重写done方法，Future放入队列中
              //以下是FutureTask中done方法的注释
              //当此任务转换到状态isDone时调用的受保护方法(无论是正常还是通过取消)。默认实现什么也不做。子类可以重写此方法来调用完成回调或执行记帐。注意，您可以在此方法的实现中查询状态，以确定该任务是否已取消
              protected void done() { completionQueue.add(task); }
          }
          ```
    
        - 使用
    
          ```java
          public static void main(String[] args) {
              CompletionService<Integer> completionService = new ExecutorCompletionService<>(service);
              //同样可以自己获取future自己查看计算结果
              Future<Integer> submit = completionService.submit(new Worker());
              //也可以从completionService获取结果
              try {
                  //此时返回的Future是已经计算完成的future，如果没有会堵塞
                  Future<Integer> take = completionService.take();
              } catch (InterruptedException e) {
                  throw new RuntimeException(e);
              }
          }
          ```
    
  - 为任务设置时限
  
    - 设置时限具体做法
  
      ```java
      Future<Integer> f1 = service.submit(new Worker());
      try {
          f1.get(10,TimeUnit.SECONDS);
      } catch (InterruptedException | ExecutionException e) {
          e.printStackTrace();
      } catch (TimeoutException e) {
          e.printStackTrace();
          f1.cancel(true);
      }
      ```
  
    - invokeAll
  
      - 线程池提供了invokeAll方法用于执行一组任务，同时返回一组Future（结构和提交时的任务组相同），每个任务最后只有完成和取消两种状态