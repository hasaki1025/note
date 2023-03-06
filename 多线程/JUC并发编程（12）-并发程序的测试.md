# JUC并发编程（12）-并发程序的测试

​	并发测试大致分为两类，安全测试和活跃性测试，进行安全测试时通常采用测试不变性条件测试，即判断某个类是否与规范保持一致。

​	活跃性测试也存在很大问题，很难验证一个方法是堵塞还是运行缓慢，同样如何测试一个算法不会发生死锁，要等待多久算是故障

​	性能指标：

​	**吞吐量**：一组并发任务中已完成的任务所占的比例

​	**响应性**：从请求发出到完成之间的时间

​	**可伸缩性**：增加更多资源的情况下吞吐量的提升情况

------------

- ## 正确性测试

  - 找出不变性条件和后验条件

    - 有界缓存类

      ```java
      public class BoundedBuffer <V> extends BaseBoundedBuffer<V> {
          // CONDITION PREDICATE: not-full (!isFull())
          // CONDITION PREDICATE: not-empty (!isEmpty())
          public BoundedBuffer() {
              this(100);
          }
      
          public BoundedBuffer(int size) {
              super(size);
          }
      
          // BLOCKS-UNTIL: not-full
          public synchronized void put(V v) throws InterruptedException {
              while (isFull())
                  wait();
              doPut(v);
              notifyAll();
          }
      
          // BLOCKS-UNTIL: not-empty
          public synchronized V take() throws InterruptedException {
              while (isEmpty())
                  wait();
              V v = doTake();
              notifyAll();
              return v;
          }
      
          // BLOCKS-UNTIL: not-full
          // Alternate form of put() using conditional notification
          public synchronized void alternatePut(V v) throws InterruptedException {
              while (isFull())
                  wait();
              boolean wasEmpty = isEmpty();
              doPut(v);
              if (wasEmpty)
                  notifyAll();
          }
      }
      
      public abstract class BaseBoundedBuffer <V> {
          private final V[] buf;
          private int tail;
          private int head;
          private int count;
      
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

  - 基本测试

    - 测试以上有界缓存类的各个方法的正确性

  - 阻塞操作的测试

    - 测试可以阻塞的方法是否可以正常阻塞且阻塞后是否能够被中断

      ```java
      @Test
      void testTakeMethod()
      {
          BoundedBuffer<Integer> buffer = new BoundedBuffer<>();
          Thread thread = new Thread(() -> {
              try {
                  buffer.take();
                  fail();
              } catch (InterruptedException e) {
      
              }
          });
      
      
          try {
              thread.start();
              sleep(3000);
              thread.interrupt();
              thread.join();
              assertFalse(thread.isAlive());
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          }
      
      }
      
      void fail()
      {
          System.out.println("测试失败");
      }
      ```

  - 