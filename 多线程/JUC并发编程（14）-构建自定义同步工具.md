# JUC并发编程（14）-构建自定义同步工具

- ## 状态依赖性管理

  - 通常对某个对象进行操作的时候需要判断条件是否满足操作的执行（如连接池非空才能获取连接），如果条件不成立则操作执行失败，在并发的情况下这种执行不好，可以一直堵塞直到条件满足

    ```java
    //获取锁..
    while(条件如果不合适)
    {
        //释放锁
        //等待条件成熟
        //等待超时或者中断
        //再次请求锁
    }
    //执行操作
    //操作结束释放锁
    ```

  - 有界缓存基类案例

    ```java
    @ThreadSafe
    public abstract class BaseBoundedBuffer <V> {
        @GuardedBy("this") private final V[] buf;
        @GuardedBy("this") private int tail;
        @GuardedBy("this") private int head;
        @GuardedBy("this") private int count;
    
        protected BaseBoundedBuffer(int capacity) {
            this.buf = (V[]) new Object[capacity];
        }
    
        protected synchronized final void doPut(V v) {
            buf[tail] = v;
            if (++tail == buf.length)
                tail = 0;
            ++count;
        }
    
        protected synchronized final V doTake() {
            V v = buf[head];
            buf[head] = null;
            if (++head == buf.length)
                head = 0;
            --count;
            return v;
        }
    
        public synchronized final boolean isFull() {
            return count == buf.length;
        }
    
        public synchronized final boolean isEmpty() {
            return count == 0;
        }
    }
    ```

  - 示例1：条件不满足传递异常给调用者

    ```java
    public class GrumpyBoundedBuffer<V> extends BaseBoundedBuffer<V>{
        protected GrumpyBoundedBuffer(int capacity) {
            super(capacity);
        }
        public synchronized V take()
        {
            if (isEmpty())
            {
                throw new RuntimeException();
            }
            return super.doTake();
        }
        
        public synchronized void put(V v)
        {
            if (isFull())
            {
                throw new RuntimeException();
            }
            doPut(v);
        }
    }
    ```

    - 错误不应该仍然让调用者解决

  - 示例2：通过轮询和休眠来实现简单的阻塞

    ```java
    
    public class SleepyBoundedBuffer<V> extends BaseBoundedBuffer<V>{
        protected SleepyBoundedBuffer(int capacity) {
            super(capacity);
        }
    
        public  V take() throws InterruptedException {
            while (true)
            {
                synchronized (this)
                {
                    if (!isEmpty()) {
                        return doTake();
                    }
                }
                sleep(500);
            }
        }
    
        public  void put(V v) throws InterruptedException {
            while (true)
            {
                synchronized (this)
                {
                    if (!isFull()) {
                        doPut(v);
                        return;
                    }
                }
                sleep(500);
            }
        }
    }
    ```

  - 条件队列

    - Object.wait会自动释放锁，并请求操作系统挂起当前线程，从而使其他线程能够获得这个锁并修改对象的状态。当被挂起的线程醒来时，将会在返回之前重新获取锁，

    - 示例：使用wait和notifyAll来实现有界缓存

      ```java
      public class BounderBuffer<V> extends BaseBoundedBuffer<V>{
          protected BounderBuffer(int capacity) {
              super(capacity);
          }
      
      
          public synchronized V take() throws InterruptedException {
              while (isEmpty())
              {
                  //自动释放锁
                  wait();
              }
              V v = doTake();
              //有元素删除唤醒所有等待this的线程
              notifyAll();
              return v;
          }
      
          public synchronized void put(V v) throws InterruptedException {
              while (isFull())
              {
                  //自动释放锁
                  wait();
              }
              doPut(v);
              //有新元素加入唤醒所有等待this的线程
              notifyAll();
          }
      }
      ```

- ## 使用条件队列

  - 尽量使用堵塞队列，Latch，Semaphore和Future代替条件队列

  - 使用条件队列

    - 确定条件谓词（队列不为空）
    - 重要的三元关系：加锁，wait,条件谓词(执行notify的时机)

  - 过早唤醒

    - wait方法返回不一定意味着等待条件谓词为真（被其他线程抢先了），所以醒来的时候必须再次测试条件谓词

  - 丢失的信号

    - 如果wait前没有检查，信号可能丢失，那么线程只能等待下一次唤醒

  - 通知

    - notify唤醒等待线程队列中的任意（随机）一个，notifyAll唤醒所有

    - 唤醒后必须快速释放锁，这样刚刚被唤醒的线程才能尽快拿到锁

    - 由于多个线程可以基于不同的条件谓词在同一个条件队列上等待（等待的线程之前都持有过同一个锁，但是条件不一样），使用notify可能会传达错误的信号导致信号丢失（比如队列为空和队列为满的条件同时锁在this上，put方法将会唤醒所有等待this的线程，包括等待队列为满的线程），但notifyAll会唤醒所有线程共同抢占锁

    - 条件谓词和锁一一对应（见上）

    - 采用notifyAll的处理方法同样会造成大量开销（最坏的情况下将导致O(n^2)次唤醒）

    - 条件唤醒

      ```java
      public synchronized void put(V v) throws InterruptedException {
          while (isFull())
          {
              wait();
          }
          boolean wasEmpty = isEmpty();
          doPut(v);
          if (wasEmpty)
          {
              notifyAll();
          }
      }
      ```

      - 仅当状态从非空到空或者空到**非空才**唤醒（减少唤醒次数，也就是没有线程等待就不需要唤醒了）

  - 示例：阀门类
  
    - 闭锁可以实现类似于阀门的效果，但是闭锁只能打开一次，通过条件队列可以设计一个可多次打开的阀门
  
      ```java
      public class ThreadGate {
          
          private boolean isOpen;
          private int version=1;
          
          public synchronized void close()
          {
              isOpen=false;
          }
          
          public synchronized void open()
          {
              version++;
              isOpen=true;
              notifyAll();
          }
          
          public synchronized void await() throws InterruptedException {
              int ownerVersion = version;
              while (!isOpen && ownerVersion==version)
              {
                  wait();
              } 
          }
      }
      
      ```
  
      - 当一个线程等待阀门开启时会先获得当前阀门的版本（或者说当前是第几次关闭），如果阀门打开（Isopen为true），此时阀门版本变化，第二条件不满足，即便当isOpen设为true后又快速设置为false，在该版本阀门前等待的线程也能顺利执行
  
  - 入口协议和出口协议
  
    - 入口协议
      - 该操作的条件谓词
    - 出口协议
      - 检查条件谓词有关的变量并通知（如果条件为true）
  
  - 显示的Condition对象
  
    - Condition是一种广义上的内置条件队列
  
      ```java
      public interface Condition {
          //和wait差不多
          void await() throws InterruptedException;
      //不接受中断的wait
          void awaitUninterruptibly();
          //wait超时版本但是时间单位是纳秒
          long awaitNanos(long nanosTimeout) throws InterruptedException;
      	//超时版本
          boolean await(long time, TimeUnit unit) throws InterruptedException;
      	//指定结束时间
          boolean awaitUntil(Date deadline) throws InterruptedException;
      //类似于notify
          void signal();
      //类似于notifyAll
          void signalAll();
      }
      ```
  
    - 使用
  
      ```java
      void test() throws InterruptedException {
          lock.lockInterruptibly();
          Condition condition = lock.newCondition();
          try {
              condition.await();
          }finally {
              lock.unlock(); 
          }
      }
      ```
  
    - 内置锁的缺陷，一个锁只能有一个条件队列，但是一个条件队列中的线程各自等待的条件谓词可能不同，而Condition类不同，Condition拥有更加丰富的方法，并且，一个Lock可以对于多个Condition（一个Condition只能有一个lock）
  
    - 示例：通过Condition实现有界缓存
  
      ```java
      package org.chapter14.test03;
      
      import java.util.concurrent.locks.Condition;
      import java.util.concurrent.locks.ReentrantLock;
      
      public class ReentrantLockBoundedBuffer<V> {
      
      
          protected ReentrantLockBoundedBuffer(int capacity) {
              this.buf = (V[]) new Object[capacity];
          }
          
          private final ReentrantLock lock=new ReentrantLock();
          private final Condition notFull=lock.newCondition();
          private final Condition notEmpty=lock.newCondition();
      
      
          private final V[] buf;
          private int tail;
          private int head;
          private int count;
      
      
      
          protected  final void doPut(V v) {
              buf[tail] = v;
              if (++tail == buf.length)
                  tail = 0;
              ++count;
          }
      
          protected  final V doTake() {
              V v = buf[head];
              buf[head] = null;
              if (++head == buf.length)
                  head = 0;
              --count;
              return v;
          }
      
          public  final boolean isFull() {
              return count == buf.length;
          }
      
          public  final boolean isEmpty() {
              return count == 0;
          }
          
          public V take() throws InterruptedException {
              lock.lockInterruptibly();
              try {
                while (!isEmpty())
                {
                    notEmpty.await();
                }
                  V v = doTake();
                notFull.signal();
                  return v;
              }
              finally {
                  lock.unlock();
              }
          }
          
          
          public void put(V v) throws InterruptedException {
              lock.lockInterruptibly();
              try {
                  while (!isFull())
                  {
                      notFull.await();
                  }
                  doPut(v);
                  notEmpty.signal();
              }finally {
                  lock.unlock();
              }
          }
      }
      
      ```
  
- ## synchronizer剖析

  - AbstractQueuedSynchronizer

    - 如果一个类想成为状态依赖的类，必须拥有一些状态，可以通过getState、setState、compareAndSetstate等方法来操作状态，AQS中保存了一个整数型变量state来表示状态，比如在可重入锁中该状态的意思是所有者线程已重复获取该锁的次数，信号量用来表示剩余的许可，FutureTask用来表示任务的状态，同步器类中还可以自己管理一些额外的变量，如可重入锁用一个变量来表示当前占用该锁的线程（可重入实现）

    - AQS的获取操作可以是独占的也可以是非独占的,获取操作可以是阻塞的也可以是非阻塞的,以下是AQS标准的获取操作和释放操作

      ```java
      boolean acquire throw InterruptedException{
          while(当前不允许执行操作)
          {
              if(需要阻塞)
              {
                  阻塞队列中没有该线程则加入
                  开始阻塞
              }
              else
              {
                  return false
              }
          }
          
          可能更新同步器状态
          如果线程位于队列中则将其移出队列
          return true;
      }
      
      void release()
      {
          更新状态
          if(新的状态产生该状态是否需要唤醒某些等待线程)
          {
              解除符合条件的线程的阻塞状态
          }
      }
      ```

    - 除了基本的获取操作和释放操作(这些方法无法重写)，如果获取操作有独占需求需要实现tryAcquire等方法，采用非独占方式需要实现带有tryAcquireShared等方法

    - 示例：使用AQS实现一个简单的二元闭锁

      ```java
      public class OneShotLatch {
      
      
          private final Sync sync=new Sync();
      
      
          public void await() throws InterruptedException {
              sync.acquireSharedInterruptibly(0);
          }
      
          public void release()
          {
              sync.releaseShared(0);
          }
      
          private class Sync extends AbstractQueuedSynchronizer{
              @Override
              protected int tryAcquireShared(int arg) {
                  //如果成功返回正数，失败返回负数
                  return getState()==1 ? 1:-1;
              }
      
              @Override
              protected boolean tryReleaseShared(int arg) {
                  setState(1);
                  return true;
              }
          }
      }
      ```

      - AQS中的方法原型

        ```java
        public final void acquireShared(int arg) {
            if (tryAcquireShared(arg) < 0)
                doAcquireShared(arg);
        }
        //需要行实现
        protected int tryAcquireShared(int arg) {
            throw new UnsupportedOperationException();
        }
        
        public final void acquireInterruptibly(int arg)
            throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            if (!tryAcquire(arg))
                doAcquireInterruptibly(arg);
        }
        //需要实现
        protected boolean tryAcquire(int arg) {
            throw new UnsupportedOperationException();
        }
        ```

      - OneShotLatch同样也可以通过扩展AQS来实现但是这样会破坏OneShotLatch代码的整洁性，JUC中的所有同步类都没有直接扩展AQS而是采用委托的方式

- ## JUC中的AQS

  - ReentrantLock中AQS的实现

    ```java
    
    abstract static class Sync extends AbstractQueuedSynchronizer {
            private static final long serialVersionUID = -5179523762034025860L;
    
            /**
             * Performs non-fair tryLock.  tryAcquire is implemented in
             * subclasses, but both need nonfair try for trylock method.
             */
            @ReservedStackAccess
            final boolean nonfairTryAcquire(int acquires) {
                final Thread current = Thread.currentThread();
                int c = getState();
                //未被独占
                if (c == 0) {
                    //CAS尝试设置值为acquires（尝试独占）
                    if (compareAndSetState(0, acquires)) {
                        //设置独占者线程
                        setExclusiveOwnerThread(current);
                        return true;
                    }
                }
                //可重入判断（当前占有的线程是否是当前线程）
                else if (current == getExclusiveOwnerThread()) {
                    int nextc = c + acquires;
                    //整型溢出则抛出异常(重入的次数太多了)
                    if (nextc < 0) // overflow
                        throw new Error("Maximum lock count exceeded");
                    //设置状态
                    setState(nextc);
                    //重入成功
                    return true;
                }
                //被其他线程独占了，无法获取
                return false;
            }
    
            @ReservedStackAccess
            protected final boolean tryRelease(int releases) {
                //重入次数-释放的个数
                int c = getState() - releases;
                //当前线程不是该锁的独占者抛出异常(或者说当前线程未持有该锁)
                if (Thread.currentThread() != getExclusiveOwnerThread())
                    throw new IllegalMonitorStateException();
                //锁的占用情况（true表示还在占用，false表示已全部释放）
                boolean free = false;
                //已全部释放
                if (c == 0) {
                    free = true;
                    //设置独占者线程为null
                    setExclusiveOwnerThread(null);
                }
                //状态更新
                setState(c);
                //返回当前锁状态
                return free;
            }
    		//该锁是否被当前线程独占
            protected final boolean isHeldExclusively() {
                // While we must in general read state before owner,
                // we don't need to do so to check if current thread is owner
                return getExclusiveOwnerThread() == Thread.currentThread();
            }
    
            final ConditionObject newCondition() {
                //ConditionObject是AQS中的一个内部类
                return new ConditionObject();
            }
    
            // Methods relayed from outer class
    		//获取独占者线程
            final Thread getOwner() {
                return getState() == 0 ? null : getExclusiveOwnerThread();
            }
    		//当前线程拥有该锁的数量（重入次数）
            final int getHoldCount() {
                return isHeldExclusively() ? getState() : 0;
            }
    		//是否已被占用
            final boolean isLocked() {
                return getState() != 0;
            }
    
            /**
             * 反序列化
             */
            private void readObject(java.io.ObjectInputStream s)
                throws java.io.IOException, ClassNotFoundException {
                //从此流中读取当前类的非静态和非瞬态字段。这只能从被反序列化的类的readObject方法调用。如果以其他方式调用它
                s.defaultReadObject();
                setState(0); // reset to unlocked state
            }
        }
    
    
    //非公平同步实现（默认采用）
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
    
    //以下是ReentrantLock中重要方法的实现
    public void lock() {
        sync.acquire(1);
    }
    
    public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }
    
    public void unlock() {
        sync.release(1);
    }
    
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
    ```

  - Semaphore实现

    ```java
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1192457210091910933L;
    
        Sync(int permits) {
            setState(permits);
        }
    
        final int getPermits() {
            return getState();
        }
    
        //循环CAS
        final int nonfairTryAcquireShared(int acquires) {
            for (;;) {
                //可用许可数量
                int available = getState();
                //请求后剩余许可数量
                int remaining = available - acquires;
                //如果剩余许可数量小于0则直接返回（不进行设置），如果剩余数量大于0则尝试CAS设置剩余数量，如果成功则返回失败则再次进入循环再次尝试CAS设置
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                //整型溢出
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                //循环CAS
                if (compareAndSetState(current, next))
                    return true;
            }
        }
    //减少可用许可
        final void reducePermits(int reductions) {
            for (;;) {
                int current = getState();
                int next = current - reductions;
                //后验条件不成立
                if (next > current) // underflow
                    throw new Error("Permit count underflow");
                //CAS循环
                if (compareAndSetState(current, next))
                    return;
            }
        }
    //许可归零
        final int drainPermits() {
            for (;;) {
                int current = getState();
                //当前许可为0直接推出否则CAS循环设置许可为0
                if (current == 0 || compareAndSetState(current, 0))
                    return current;
            }
        }
    }
    
    //非公平实现
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = -2694183684443567898L;
    
        NonfairSync(int permits) {
            super(permits);
        }
    
        protected int tryAcquireShared(int acquires) {
            return nonfairTryAcquireShared(acquires);
        }
    }
    
    //信号量实现
    public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
    
    public boolean tryAcquire() {
        return sync.nonfairTryAcquireShared(1) >= 0;
    }
    
    public void release() {
        sync.releaseShared(1);
    }
    ```

  - FutureTask实现

    - FutureTask并没有直接显式的含有一个AQS实现类而是直接内部实现了一个AQS，除此以外FutureTask内部含有一个WaitNode类用于保存该future对应的Thread

      ```java
      static final class WaitNode {
          volatile Thread thread;
          volatile WaitNode next;
          WaitNode() { thread = Thread.currentThread(); }
      }
      ```

  - 读写锁实现

    - 读写锁的AQS实现由单个AQS管理读锁和写锁

    - AQS内部维护一个等待线程队列，其中同时记录了等待线程的请求类型（独占还是共享），当锁可用时，如果位于队列头部的线程执行写入则会得到该锁，如果执行的是读操作则队列中第一个写入线程之前的所有线程都会得到这个锁

      ```java
      //写线程请求
      protected final boolean tryAcquire(int acquires) {
          Thread current = Thread.currentThread();
          //获取当前持有读锁和写锁的总数
          int c = getState();
          //获取写锁数量
          int w = exclusiveCount(c);
          
          if (c != 0) {
              // //如果总锁！=0且写锁数量为0-->多个读线程-->写线程互斥
              //如果总锁！=0且写线程！=0且当前请求的线程不是写锁的独占线程-->有一个写线程而且不是请求的线程-->写写互斥
              if (w == 0 || current != getExclusiveOwnerThread())
                  return false;
              //溢出
              if (w + exclusiveCount(acquires) > MAX_COUNT)
                  throw new Error("Maximum lock count exceeded");
              // 重入
              setState(c + acquires);
              return true;
          }
          //没有写线程和读线程的情况
         //当前线程需要阻塞或者CAS设置失败
          if (writerShouldBlock() ||
              !compareAndSetState(c, c + acquires))
              return false;
          //设置独占
          setExclusiveOwnerThread(current);
          return true;
      }
      
      //读线程请求
      @ReservedStackAccess
      protected final int tryAcquireShared(int unused) {
          Thread current = Thread.currentThread();
          int c = getState();
          //有写线程且不是当前线程-->请求失败
          if (exclusiveCount(c) != 0 &&
              getExclusiveOwnerThread() != current)
              return -1;
          //读线程数量
          int r = sharedCount(c);
          //该线程不需要阻塞，读线程不没有达到最大，CAS尝试增加读线程成功
          if (!readerShouldBlock() &&
              r < MAX_COUNT &&
              compareAndSetState(c, c + SHARED_UNIT)) {
              //第一个读线程设置
              if (r == 0) {
                  firstReader = current;
                  firstReaderHoldCount = 1;
                  //第一个读线程就是我
              } else if (firstReader == current) {
                  firstReaderHoldCount++;
              } else {
                  //最后一个读线程的计数器
                  HoldCounter rh = cachedHoldCounter;
                  //计数器为null或者计数器所属线程不是当前线程
                  if (rh == null ||
                      rh.tid != LockSupport.getThreadId(current))
                      //计数设置
                      cachedHoldCounter = rh = readHolds.get();
                  else if (rh.count == 0)
                      readHolds.set(rh);
                  rh.count++;
              }
              return 1;
          }
          return fullTryAcquireShared(current);
      }
      
      //公平锁中writerShouldBlock实现
      final boolean writerShouldBlock() {
          //查询是否有线程一直在等待比当前线程更长的时间
          return hasQueuedPredecessors();
      }
      
      //非公平锁实现
      final boolean writerShouldBlock() {
          return false; // writers can always barge
      }
      
      
      
      final boolean readerShouldBlock() {
      //作为避免无限期写入器饥饿的启发式方法，如果暂时出现在队列头(如果存在)的线程是等待写入器，则阻塞。这只是一种概率效应，因为如果在其他尚未从队列中耗尽的启用的读取器后面有一个等待的写入器，那么新的读取器将不会阻塞
          return apparentlyFirstQueuedIsExclusive();
      }
      
      final boolean readerShouldBlock() {
          ///查询是否有线程一直在等待比当前线程更长的时间
          return hasQueuedPredecessors();
      }
      ```

      

    
