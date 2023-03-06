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

      - 仅当状态从非空到空或者空到非空才唤醒（减少唤醒次数，也就是没有线程等待就不需要唤醒了）

  - 

