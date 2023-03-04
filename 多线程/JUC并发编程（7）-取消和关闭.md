# JUC并发编程（7）-取消和关闭

- ## 任务的取消

  - 取消的场景

    - 用户请求取消（通过远程方法调用等）
    - 时间限制（超时取消）
    - 应用程序事件（多个线程同时寻找一个问题的解，如果其中一个找到了，其他的线程就可以取消当前的任务）
    - 错误
    - 关闭（当一个服务或者程序停止运行，线程池关闭等），在java中没有一种安全的抢占式方法来停止线程，也就是说没有安全的抢占式方法来中止任务，只能通过一种协作式的机制设置某个请求取消的标记位

  - 取消策略

    - 其他代码如何取消任务
    - 何时检查任务需要取消
    - 取消时应该执行什么操作

  - 中断

    - 错误的取消策略

      ```java
      public class CancelPolicy1 extends Thread{
          private final BlockingQueue<Integer> queue=new LinkedBlockingQueue<>(10);
          private volatile boolean cancelled=false;
          @Override
          public void run() {
              int i=1;
              while (!cancelled)
              {
                  try {
                      queue.put(i++);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
          
          public void cancel()
          {
              cancelled=true;
          }
      }
      ```

      - 阻塞队列提供的阻塞方法可能会导致线程永远无法检查到cancelled的变化

    - 在第五章提到过一些特殊的阻塞方法支持检测中断信号，如sleep，wait等方法可以在检测到中断后立刻清除中断状态并抛出异常，JVM并不能保证检测到中断的速度但是实际上这种响应还是非常快的

    - **在非堵塞情况下**发生中断，中断状态将被设置，然后根据被取消的操作来检查中断状态以判断发生了中断，通过这样的方法，只要不触发InterruptedException那么中断状态将一直保持直到明确的清除中断状态

    - 中断策略

      - 推迟处理中断
        - 尽快退出，在必要时进行清理，通知某个所有者该线程已退出。

    - 响应中断

      - 传递异常

        - 向上层代码传递异常

      - 恢复中断

        - 对于可以取消的任务可以在捕获到异常时提前结束，对于不可以取消的任务可以在先保留中断的状态（通过一个bool变量保存）并再次尝试操作，在最后再恢复中断状态

          ```java
          public class InterruptedTask implements Runnable {
          
              private volatile boolean interrupted;
              @Override
              public void run() {
                  try{
                      while (true)
                      {
                          //执行操作
                          throw new InterruptedException();
                      }
                  }catch (InterruptedException e)
                  {
                      interrupted=true;
                      //再次尝试
                  }
                  finally {
                      if (interrupted)
                      {
                          //恢复中断
                          Thread.currentThread().interrupt();
                      }
                  }
              }
          }
          
          ```

  - 通过Future实现取消

    - future的cancel方法

      ```java
          /**
      	试图取消此任务的执行。如果任务已经完成或取消，或者由于其他原因无法取消，则此方法无效。否则，如果在调用cancel时此任务尚未启动，则该任务将永远不会运行。如果任务已经启动，则mayInterruptIfRunning参数确定执行此任务的线程(当实现知道时)是否被中断以试图停止任务。 
      	此方法的返回值不一定指示任务现在是否取消;使用isCancelled。 
      	形参: mayInterruptIfRunning -如果执行此任务的线程应该被中断(如果该线程为实现所知)则为true;否则，允许完成正在进行的任务 
      返回值: 如果任务不能取消，通常是因为任务已经完成，则为False;真正的否则。如果两个或多个线程导致一个任务被取消，那么至少有一个线程返回true。实现可以提供更强的保证。
           */
          boolean cancel(boolean mayInterruptIfRunning);
      ```

    - 具体使用

      ```java
      try {
          future.get(10, TimeUnit.SECONDS);
      } catch (InterruptedException e) {
          //中断异常，重新抛出
          throw new RuntimeException(e);
      } catch (ExecutionException e) {
          //其他异常，重新抛出
          throw new RuntimeException(e);
      } catch (TimeoutException e) {
          //超时异常
          throw new RuntimeException(e);
      }
      finally {
          future.cancel(true);
      }
      ```

  - 处理不可中断的堵塞

    - 许多可堵塞方法都支持堵塞期间检查中断位置，但是某些堵塞方法不支持,这些方法只能设置中断位置

      - SocketIO

        - 虽然read和Write方法不会响应中断，但是可以通过关闭套接字来抛出SocketException

        - Socket中断响应实例：

          ```java
          public class SocketInerruptHandler extends Thread {
          
              private final Socket socket;
          
              public SocketInerruptHandler(Socket socket) {
                  this.socket = socket;
              }
          
              @Override
              public void interrupt() {
                  try {
                      socket.close();
                  } catch (IOException e) {
                      throw new RuntimeException(e);
                  }
                  finally {
                      super.interrupt();
                  }
              }
          
              @Override
              public void run() {
                  try{
                      InputStream stream = socket.getInputStream();
                      //IO操作
                  }catch (IOException e)
                  {
                      //重写后的中断方法将会关闭Socket的IO流，之后对IO流的操作必然会抛出IO异常，捕获IO异常后允许线程退出
                  }
              }
          }
          
          ```

      - 同步IO 

      - Selector的异步IO

        - 调用close方法或者wakeup方法可以使调用select的线程抛出ClosedSelectorException

      - 内置锁堵塞

        - 一个线程获取内置锁时堵塞将无法响应中断，因为该线程认为自己肯定能获得锁，但在Lock类种提供了lockInterruptibly方法允许在等待一个锁的同时仍能响应中断（13章）

  - 采用newTaskFor方法来封装非标准的取消

    - 在上面的Socket中断策略中直接对线程进行方法重写，但是在实际应用中一般是通过Future进行取消，但是如果要重写Future的cancel方法则需要重写submit等方法，过于复杂，在JAVA6之后可以通过newTaskFor方法将Callable封装为RunnableFuture（实际采用FutureTask实现），RunnableFuture继承了Runnable和Future，通过newTaskFor，我们只需要自定义一个线程池重写newTaskFor方法返回自定义的FutureTask即可。

      - 自定义Callable实现

      ```java
      public class SockCallable<T> implements Callable<T> {
      
          private final Socket socket;
      
          public void cancel()
          {
              if (socket!=null)
              {
                  try {
                      socket.close();
                  } catch (IOException e) {
                      //可忽视
                  }
              }
          }
      
          public SockCallable(Socket socket) {
              this.socket = socket;
          }
      
          @Override
          public T call() throws Exception {
              try
              {
                  //执行Socket连接
                  InputStream inputStream = socket.getInputStream();
                  return null;
              }
              catch (IOException e)
              {
                  //执行线程退出
              }
              return null;
          }
      
          public RunnableFuture<T> newTask()
          {
              return new FutureTask<>(this)
              {
                  @Override
                  public boolean cancel(boolean mayInterruptIfRunning) {
                      SockCallable.this.cancel();
                      return super.cancel(mayInterruptIfRunning);
                  }
              };
          }
      }
      
      ```

      - 自定义线程池

        ```java
        public class SocketThreadPool extends ThreadPoolExecutor {
           //忽略构造方法
        
            @Override
            protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
                if (callable instanceof SockCallable)
                {
                    return ((SockCallable) callable).newTask();
                }
                else {
                    return super.newTaskFor(callable);
                }
        
            }
        }
        
        ```

- ## 停止基于线程的服务

  - 示例日志服务

    - 日志服务基于多生产者-单一消费者模式，生产者线程将需要写入的日志信息放入到BlockingQueue中，消费者线程读取并写出

      - 不支持关闭的日志服务

        ```java
        public class LogWriter {
            private final BlockingQueue<String> queue;
            private final LoggerThread logger;
            private static final int CAPACITY = 1000;
        
            public LogWriter(Writer writer) {
                this.queue = new LinkedBlockingQueue<String>(CAPACITY);
                this.logger = new LoggerThread(writer);
            }
        
            public void start() {
                logger.start();
            }
        
            public void log(String msg) throws InterruptedException {
                queue.put(msg);
            }
        
            private class LoggerThread extends Thread {
                private final PrintWriter writer;
        
                public LoggerThread(Writer writer) {
                    this.writer = new PrintWriter(writer, true); // autoflush
                }
        
                public void run() {
                    try {
                        while (true)
                            writer.println(queue.take());
                    } catch (InterruptedException ignored) {
                    } finally {
                        writer.close();
                    }
                }
            }
        }
        
        ```

        - 要想以上日志服务支持关闭较为简单，因为堵塞队列的take方法支持响应中断，只需要传递中断信号就能中止线程，如果捕捉到InterruptExcetion就直接推出，这样的做法是不正确的，直接退出线程会导致未写出的日志信息丢失，不仅如此，如果堵塞队列在外部含有引用，其他线程在调用log服务时可能会因为队列已满导致长时间堵塞
        - 正确做法时关闭后停止接受日志请求，并且消化完队列中未能写出的日志，解除因为队列而堵塞的线程

      - 设置log服务关闭的标志的日志服务

        ```java
        public void log(String msg)
        {
            if(!isShutdown)
            {
                queue.put(msg);
            }
            else
            {
                throw new RuntimeException();
            }
        }
        ```

        - 先判断后操作的机制并不是原子操作仍然存在一些线程在日志服务关闭后再次写入日志

      - 合理的取消操作的日志服务

        ```java
        public class LogService {
            private final BlockingQueue<String> queue;
            private final LoggerThread loggerThread;
            private final PrintWriter writer;
            @GuardedBy("this") private boolean isShutdown;
            //记录log服务剩余请求数量
            @GuardedBy("this") private int reservations;
        
            public LogService(Writer writer) {
                this.queue = new LinkedBlockingQueue<String>();
                this.loggerThread = new LoggerThread();
                this.writer = new PrintWriter(writer);
            }
        
            public void start() {
                loggerThread.start();
            }
        
            public void stop() {
                synchronized (this) {
                    isShutdown = true;
                }
                //通过中断取消
                loggerThread.interrupt();
            }
        
            public void log(String msg) throws InterruptedException {
                synchronized (this) {
                    if (isShutdown)
                        throw new IllegalStateException(/*...*/);
                    //如果没有关闭则增加请求
                    ++reservations;
                }
                queue.put(msg);
            }
        
            private class LoggerThread extends Thread {
                public void run() {
                    try {
                        while (true) {
                            try {
                                synchronized (LogService.this) {
                                    //如果已经关闭，且无剩余请求数量则退出
                                    if (isShutdown && reservations == 0)
                                        break;
                                }
                                //还有剩余请求继续获取
                                String msg = queue.take();
                                synchronized (LogService.this) {
                                    //完成请求减少剩余次数
                                    --reservations;
                                }
                                writer.println(msg);
                            } catch (InterruptedException e) {
                                //继续循环直到无剩余请求
                                /* retry */
                            }
                        }
                    } finally {
                        //关闭IO
                        writer.close();
                    }
                }
            }
        }
        
        ```

  - 关闭ExecutorService

    - 通过线程池封装上面的日志服务

      ```java
      public class LogService {
      
          private final ExecutorService executorService= Executors.newSingleThreadExecutor();
      
          private volatile boolean isShutdowned=false;
      
          public void shutdown() throws InterruptedException {
              if (!isShutdowned)
              {
                  isShutdowned=true;
                  executorService.shutdown();
                  if (!executorService.awaitTermination(1, TimeUnit.SECONDS)) {
                      List<Runnable> runnables = executorService.shutdownNow();
                      System.out.println("剩余任务数:"+ runnables.size());
                  }
                  System.out.println("shutdown.....");
              }
          }
      
          public void log(String s)  {
              if (!isShutdowned)
              {
                 executorService.execute(new logRunnable(s));
              }
          }
          private class logRunnable implements Runnable
          {
      
              private final String message;
      
      
              public logRunnable(String message) {
      
                  this.message = message;
              }
      
              @Override
              public void run() {
                  try {
                      sleep(1000);
                      System.out.println("log:"+message);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
      
          }
      }
      
      ```
      
      - 使用单例线程池自带的shutdown方法和shutdownNow方法
        - shutdown只会等待线程池中所有任务执行完成
        - shutdownNow会给没有终止的任务设置中断信号
        - awaitTermination会堵塞直到超时或者线程池关闭，如果超时则会返回false，此时可以通过shutdownNow关闭线程池
    
  - 毒丸对象
  
    - 在生产者消费者模型上，如果需要停止线程可以通过安排一个特殊的毒丸任务，执行到毒丸任务时关闭线程池，通过毒丸对象可以停止无限循环的线程
  
      ```java
      public class IndexService {
      
          private volatile boolean isShutdown=false;
          private final String POISON ="我是毒丸";
          private final BlockingQueue<Object> blockingQueue=new LinkedBlockingQueue<>();
      
          private volatile boolean isInit=false;
      
          private final Producer producer=new Producer();
          private final Consumer consumer=new Consumer();
      
          public void init()
          {
              if (!isInit)
              {
                  isInit=true;
                  System.out.println("init.....");
                  producer.start();
                  consumer.start();
              }
          }
      
          public void shutdown() {
              if (!isShutdown)
              {
                  isShutdown=true;
                  System.out.println("shutdown");
                  producer.interrupt();
              }
          }
      
          public void awaitShutdown() throws InterruptedException {
              producer.join();
          }
      
      
          public class Producer extends Thread{
      
              @Override
              public void run() {
                  try {
                      crawl();
                  } catch (InterruptedException e) {
                      //无需响应直接进入finally
                      e.printStackTrace();
                  }
                  finally {
                      while (true)
                      {
                          try {
                              blockingQueue.put(POISON);
                              break;
                          } catch (InterruptedException e) {
                             //无视，该异常是由于put方法带来的，我们需要再次尝试放入毒丸对象直到成功为止
                              e.printStackTrace();
                          }
                      }
                  }
              }
          }
      
          private void crawl() throws InterruptedException
          {
              while (true)
              {
                  //执行生产者任务
                  sleep(1000);
                  blockingQueue.put("nihao");
              }
          }
      
          public class Consumer extends Thread{
              @Override
              public void run() {
                      try {
                          while (true) {
                              Object take = blockingQueue.take();
                              if (take == POISON) {
                                  System.out.println("找到毒丸了");
                                  break;
                              }
                              sleep(2000);
                              System.out.println("执行任务..."+take);
                          }
                      } catch (InterruptedException e) {
                          //如果能通过中断停止那最好
                          e.printStackTrace();
                      }
              }
          }
      
      }
      
      ```
  
  - shutdownNow的局限性
  
    - shutdownNow将会返回所有未开始的任务，但是对于已经开始尚未结束的任务的状态我们是无法获取的
  
- ## 处理非正常的线程终止

  - UncaughtExceptionHandler

    - UncaughtExceptionHandler能够检测并捕获导致线程意外终止的异常

      ```java
      public interface UncaughtExceptionHandler {
              void uncaughtException(Thread t, Throwable e);
          }
      
      //使用
      @Slf4j
      public class MyUncaughtExceptionHandler implements Thread.UncaughtExceptionHandler {
          @Override
          public void uncaughtException(Thread t, Throwable e) {
              log.info(t.getName()+"get exception: "+e.getMessage());
              e.printStackTrace();
          }
      }
      //测试
      public class test12 {
          public static void main(String[] args) {
              Thread thread = new Thread(() -> {
                  try {
                      sleep(2000);
                      throw new RuntimeException("我是一个错误");
                  } catch (InterruptedException e) {
                      throw new RuntimeException(e);
                  }
              });
              thread.setUncaughtExceptionHandler(new MyUncaughtExceptionHandler());
      
              thread.start();
          }
      }
      
      ```

      - 如果想要为线程池中的每一个线程都设置一个UncaughtExceptionHandler，可以使用线程工厂
      - 如果你希望任务由于异常失败时获取通知并执行一定的恢复操作，可以将任务封装到能捕获异常的Callable或者Runnable中，或者可以重写ThreadPollExecutor的afterExecute方法
      - 在线程池中，只有通过execute执行的任务才会传递给UncaughtExceptionHandler，而submit提交的任务将会通过Future封装，通过get方法抛出ExecutionException

