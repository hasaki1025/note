# JUC并发编程（13）-显式锁

​	**jdk5之后提供了一种新的机制--可重入锁ReentrantLock，ReentrantLock并不是一种代替内置锁的方法而是当内置锁机制不适用时作为一种可选择的高级功能**

- ## Lock与ReentrantLock

  - Lock接口中定义了一组抽象操作的加锁操作，Lock提供了无条件的、可轮询的、定时的、以及可中断的锁的获取操作，所有加锁和解锁的方法都是显式的

    ```java
    public interface Lock {
    //获取锁。 如果锁不可用，则当前线程出于线程调度的目的将被禁用，并处于休眠状态，直到获得锁。 实现注意事项 Lock实现可以检测到锁的错误使用，例如会导致死锁的调用，并且在这种情况下可能会抛出(未检查的)异常。该Lock实现必须记录环境和异常类型
        void lock();
    //获取锁，除非当前线程被中断。
    //如果锁可用，则获取锁并立即返回。 如果锁不可用，那么当前线程出于线程调度的目的被禁用，并处于休眠状态，直到发生以下两种情况之一: 锁由当前线程获取;或 其他线程中断当前线程，并且支持锁获取的中断。 如果当前线程: 在进入此方法时设置其中断状态;或 在获取锁时被中断，并且支持锁获取的中断， 然后抛出InterruptedException，并清除当前线程的中断状态。 实现注意事项 在某些实现中，中断锁获取的能力可能是不可能的，如果可能的话，可能是一个昂贵的操作。程序员应该意识到可能会出现这种情况。在这种情况下，实现应该记录。 实现更倾向于响应中断而不是正常的方法返回。 Lock实现可以检测到锁的错误使用，例如会导致死锁的调用，并且在这种情况下可能会抛出(未检查的)异常。该Lock实现必须记录环境和异常类型。
        
        void lockInterruptibly() throws InterruptedException;
       
        
        //只有当锁在调用时是空闲的，才会获取锁。 如果锁可用，则获取锁，并立即返回值为true。如果锁不可用，则此方法将立即返回值为false。 这个方法的典型用法是: 
        /**
        Lock lock = ...;
         * if (lock.tryLock()) {
         *   try {
         *     // manipulate protected state
         *   } finally {
         *     lock.unlock();
         *   }
         * } else {
         *   // perform alternative actions
         * }}
        **/
        
     //   这种用法确保锁在被获取时被解锁，并且在没有被获取时不会尝试解锁。 返回值: 如果获得了锁，则为True，否则为false
        boolean tryLock();
    //在指定时间内阻塞超时直接返回
        boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    //释放锁
        void unlock();
    //返回一个绑定到此Lock实例的新Condition实例。 在等待条件之前，锁必须被当前线程持有。调用Condition.await()将在等待之前自动释放锁，并在等待返回之前重新获取锁。
        Condition newCondition();
    }
    
    ```

    - 可重入锁实现了Lock接口并提供了和内置锁相同的互斥性和内存可见性，但是内置锁存在局限性，如无法中断等待锁的线程，内置锁必须在获取锁的代码块中释放

    - Lock的标准使用方式

      ```java
      ReentrantLock lock = new ReentrantLock();
      lock.lock();
      try {
      
      }finally {
          lock.unlock();
      }
      ```

      - 如果没有使用finally来释放lock,那么相当于启动了一个定时炸弹，炸弹爆炸时很难追踪到最初发生错误的为止，这也是重入锁无法完全替代内置锁的原因，它更加危险

  - 轮询锁和定时锁

    - 可定时和可轮询的锁获取模式是通过tryLock方法实现的，对于内置锁，死锁是无法解决的只有通过重启。可定时锁和可轮询锁可以避免死锁的发生。

    - 使用tryLock代替内置锁完成银行转账

      ```java
      boolean transfer(Account from, Account to, int money, long timeOut) throws InterruptedException {
      
              long startTime = System.currentTimeMillis();
              while (true)
              {
                  if (from.lock.tryLock())
                  {
                      try {
      
                          if (to.lock.tryLock())
                          {
                              try {
      
                                  if (from.money>money)
                                  {
                                      from.money-=money;
                                      to.money+=money;
                                      return true;
                                  }
                                  else {
                                      throw new RuntimeException();
                                  }
                              }finally {
                                  to.lock.unlock();
                              }
                          }
                      }finally {
                          from.lock.unlock();
                      }
                  }
      
                  if (System.currentTimeMillis()-startTime>timeOut)
                  {
                      return false;
                  }
                  //避免活锁的发生
                  sleep(20);
              }
      
          }
      ```

      - 在指定时间内不断尝试获取两个锁，期间使用休眠方法避免活锁(防止程序空转)
      - 将以上的tryLock替换为定时版本同样可行且可以去掉while循环

  - 可中断的锁获取操作

    - 使用基本和重入锁基本相同，可中断锁会抛出中断异常，可以根据需求自行处理异常

  - 非块结构加锁

    - 采用Lock类锁可以在降低锁的粒度实现锁分段,比如每个链表节点使用一个锁，遍历链表时只有持有上一个锁才能持有下一个节点的锁并释放上一个节点的锁

- ## 性能考虑因素

  - JDK6之后采用了改进后的算法来管理内置锁，该算法与可重入锁类似，有效的提高了可伸缩性

    - 性能测试下JDK5中可重入锁有更高的吞吐量，但在JDK6中两者相似

      ![image-20230306180630854](D:\note\多线程\JUC并发编程.assets\JUC并发编程（2）线程安全性.md)

- ## 公平性

  - ReentrantLock在构造方法上提供了两种公平性选择，创建一个非公平锁（默认）或者一个公平锁，公平锁表示线程将按照它们发出请求的顺序来获取锁，非公平允许插队：当一个线程请求非公平锁时，如果发出请求的同时该锁的状态变为可用那么这个线程将跳过队列中所有等待线程并获得这个锁（semaphore同样可以选择采用公平或者非公平的方式），非公平不提倡插队但无法防止，公平锁用一个队列来保存请求的线程。

  - 公平性由于在挂起线程和恢复线程时存在开销而极大地降低性能，大多数情况下，非公平的性能要高于公平

    ![image-20230306181333982](D:\note\多线程\JUC并发编程.assets\image-20230306181333982.png)

    - 即使对于公平锁而言，可轮询的tryLock仍然会插队

- ## 内置锁和可重入锁孰优孰劣

  - 不使用可重入锁完全代替内置锁的原因
    - 老系统的维护困难，两者混用容易发生错误，可重入锁很危险
    - 线程转储时能识别内置锁但识别不了可重入锁
    - 内置锁才是正黄旗，未来可能会提升性能，JVM对内置锁的优化不可忽视

- ## 读-写锁

  - 无论是内置锁还是可重入锁都是互斥锁，互斥是一种保守的机制，实现了读写之间的互斥的同时同样让读读也无法并发，如果放宽读读操作可以带来更高的并发性

  - ReadWriteLock接口

    ```java
    public interface ReadWriteLock {
    
        Lock readLock();
    
        Lock writeLock();
    }
    ```

    - 允许多个读操作，但是每次只能有一个写操作，在多读的场景下读写锁能提高性能但是其他情况下比独占锁要差一点（读写锁更复杂）

    - 可选实现方式考量

      - 释放优先
        - 读线程和写线程哪个优先释放
      - 读进程插队
        - 是否允许读线程优先（允许读进程插队？），如果写线程先可能会导致读线程饥饿
      - 重入性
        - 是否可重入（ReentrantReadWriteLock实现了）
      - 降级
        - 如果一个线程持有写入锁，能否在不释放写入锁的情况下获得读取锁，这可能会导致写入锁降级为读锁，同时不允许其他写线程修改资源
      - 升级
        - 读线程是否能优先于其他等待的读线程和写线程升级为读取锁，大多数都不支持升级，因为这容易造成死锁（两个读锁同时想升级，两者都不会释放读锁）

    - ReentrantReadWriteLock

      - 公平模式（默认非公平）
        - 等待时间最长的线程优先获取锁
        - 如果读取锁被A占用，B占用了写锁，A用完后释放了读锁，B开始写，在B未释放前，其他读线程都不能获取读锁
      - 非公平
        - 获取访问许可的顺序是不确定的，写线程可以降级为读线程，读不可以升级

    - 案例：使用读写锁包装HashMap

      ```java
      public class ReadWriteMap<K,V> {
      
          private final HashMap<K,V> map=new HashMap<>();
          private final ReadWriteLock lock=new ReentrantReadWriteLock(true);
      
          public V put(K key,V value) throws InterruptedException {
              lock.writeLock().lockInterruptibly();
              try {
                  return map.put(key, value);
              }finally {
                  lock.writeLock().unlock();
              }
          }
      
      
          public V get(K key) throws InterruptedException {
              lock.readLock().lockInterruptibly();
              try {
                  return map.get(key);
              }finally {
                  lock.readLock().unlock();
              }
          }
      
      }
      
      ```

      

​	