# JUC并发编程（10）-避免活跃性问题

- ## 死锁

  - 锁顺序死锁

    - 在代码中A先获取L锁后获取R锁，B先获取R锁后获取L锁，这种执行顺序极容易造成死锁，如果A获取锁的顺序和B相同就不会产生死锁

  - 动态的锁顺序死锁

    - 固定的锁顺序死锁的基础上，两个锁的位置不再固定，比如执行银行的转账，需要锁定两个账户（A和B），A和B是不确定的，但仍然可能会发生死锁

    - 对与这种情况，必须定义锁的顺序，比如银行账户的锁必须先获取hashcode更小的那个（获取锁的顺序并不影响业务逻辑），使用System.identityHashCode

      ```java
      public void TransferMoney(final Object fromAct,final Object toAct,final int amout)
          {
              int fromHash = System.identityHashCode(fromAct);
              int toHash = System.identityHashCode(toAct);
              
              if (fromHash<toHash)
              {
                  synchronized (fromAct)
                  {
                      synchronized (toAct)
                      {
                          //业务逻辑
                      }
                  }
              }
              else {
                  synchronized (toAct)
                  {
                      synchronized (fromAct)
                      {
                          //业务逻辑
                      }
                  }
              }
          }
      ```

    - 在极少数的情况下两者的hashCode是相同的,相同情况下可以采用加时赛锁，在获取这两个锁之前先获取一个加时赛对象的锁，确保每次只有一个线程以未知的顺序获取这两个锁

    - 如果两个账户（或者两个对象之间）包含一个唯一的、不变的键值则就更加容易确定顺序

  - 协作对象之间发生的死锁

    - 某些获取多个锁并不像以上的代码如此明显，获取两个锁的动作不一定在同一个方法中被获取

      ```java
      class A{
          synchronized void dosomething (B b)
          {
              b.dosomething();
          }
          
          synchronized void print()
          {
              System.out.println("nihao");
          }
      }
      
      class B{
          synchronized void dosomething()
          {
              //未知的代码
          }
          
          synchronized void dosomething2(A a)
          {
             a.print();
          }
      }
      
      ```

      - 假设有两个线程并发执行A中的doSomething和B的dosomething2方法，也会产生死锁的可能
      - 这种情况下只能警惕在一个带有锁的方法中获取另一个锁（调用另一个方法）

  - 开放调用

    - 开发调用指的是调用一个方法本身不需要持有锁，仅在需要同步数据的情况下获取锁，开发调用并不能保证一定不发生死锁，但是开发调用的方法对于获取锁的顺序更加容易分析
    - 开发调用可能会使原本为原子操作的操作不再同步，但是只要不是必要的原子操作都可以舍弃，只需要保证必要操作的原子性

  - 资源死锁

    - 除了内置锁带来的死锁，资源同样也能带来死锁，如果用两个信号量表示两种资源

      ```java
      class A{
          void getResource() throws InterruptedException {
              test09.sem1.acquire();
              test09.sem2.acquire();
          }
      }
      
      class B{
          void getResource() throws InterruptedException {
              test09.sem2.acquire();
              test09.sem1.acquire();
          }
      }
      
      public class test09 {
      
          public static final Semaphore sem1=new Semaphore(1);
          public static final Semaphore sem2=new Semaphore(1);
      }
      ```

      - 以上情况同样可能带来死锁

    - 线程饥饿死锁同样属于资源死锁（A线程提交任务B到与自己同源的线程池中并堵塞等待B的执行完成，恰好此时线程池无空闲线程，B任务进入堵塞队列等待A线程的执行完成）

- ## 死锁的避免和诊断

  - 支持定时的锁
    - 显式锁Lock类中的tryLock机制代替内置锁机制
  - 通过线程转储信息来分析死锁
    - 线程转储其实就是显示当前所有线程的快照，通过快照描述可看到线程的状态以及持有的锁和请求的锁
  
- ## 其他活跃性危险

  - 饥饿
    - 线程优先级
      - Java设置的线程优先级在不同的操作系统下可能有不同的映射，可能在java中设置的不同线程优先级在一个操作系统下优先级相同
      - 优先级起到的作用有限，可能提高了优先级但是仍然没有什么明显的作用，通常不建议修改优先级，修改优先级可能会提高饥饿问题的风险
  - 糟糕的响应性
    - 不良的锁设计会导致糟糕的响应性，
  - 活锁
    - 与死锁不同的是，死锁导致线程堵塞，活锁导致线程不断重复无意义的工作（无限循环等）而且总会失败
    - 要解决活锁问题，需要给问题添加一点随机性，比如以太网协议中，两个相同频率的波发送并碰撞后，以太网规定之后的发送将会延迟一个随机时间防止再次碰撞