# JUC并发编程（16）-Java内存模型

- ## 什么是内存模型，为什么需要它

  - Java语言规范要求JVM在线程中维护一种类似于串行的语义：只要程序的最终结果与在严格串行环境中执行的结果相同，那么上述的所有操作都是允许的（处理器可以采用乱序或者并行的方式来执行指令）

    - 不断提升的并行性
      - 采用流水线的超标量执行单元
      - 动态指令调度
      - 猜测执行
      - 多级缓存
    - 编译器的改进
      - 指令重排实现优化执行
      - 全局寄存器分配算法

  - 平台的内存模型

    - 串行一致性
      - 程序执行一种简单的假设，程序中只存在唯一的操作执行顺序（单线程），不考虑在何种处理器上执行，并且每次读取变量时，都能获得在执行序列中（任何处理器）最近一次写入该变量的值

  - 重排序

    - 如果程序中没有包含足够的同步可能会产生奇怪的结果

      - 内存级的重排序会使程序的行为变得不可预测

        ```java
        private static int a=0,b=0;
        private static int x=0,y=0;
        
        public static void main(String[] args) {
            new Thread(()->{
                x=a;
                y=b;
            }).start();
        
            new Thread(()->{
                a=1;
                b=2;
            }).start();
        }
        ```

        - 可能a先赋值为1可能b先赋值为2，在JVM看来这两种顺序带来的结果是一样，但是在并发的情况下，x得到的值却有可能是不一样的

  - java内存模型简介

    - JMM为程序种所有操作定义了一个偏序关系--Happens-Before，要想执行操作B的线程看到操作A的结果（无论AB是否在同一个线程中执行），那么AB之间必须满足Happens-Before，如果两者之间不存在Happens-Before，则JVM可以对它们进行任意的重排序
      - 程序顺序规则
        - 如果A在B前，那么线程中A将在B之前执行
      - 监视器锁规则
        - 解锁操作必须加锁之前执行（同一个锁）
      - volatile规则
        - volatile的写入必须在读之前
      - 线程启动规则
        - 在线程上对于Thread.Start的调用必须在该线程中执行任何才做之前执行
      - 线程结果规则
        - 任何操作必须在其他线程检测到该线程已经结束之前执行或者从join方法中返回或者在调用isAlive方法时返回false
      - 中断规则
        - 当一个线程在另一个线程调用中断时，必须在被中断的线程检测到interrupt调用之前执行（中断操作必须在检测操作之前执行，检测操作如抛出异常或者调用isInterrupted和interrupted）
      - 终结器规则
        - 对象的构造函数必须在终结器之前执行完成
      - 传递性
        - 如果A在B之前执行，且B在C之前执行，那么A必须在C之前
    - 锁上的Happended-Before规则
      - 如果线程A获取到锁M，线程B之后获取到锁M，那么在线程A获取锁到释放锁的所有操作必须在B获取锁到释放锁的所有操作之前

  - 不安全发布

    ```java
    private Resource resource;
    
    public Resource getInstance()
    {
        if (resource==null)
        {
            resource=new Resource();
        }
    
        return resource;
    }
    ```

    - 问题一：检查并new并不是原子操作，这可能会导致对象多次初始化

    - 问题二：new并不是原子操作,一共有一下三个步骤

      ```java
      1、分配内存空间
      2、初始化对象
      3、将内存空间的地址赋值给对应的引用
      ```

      - 但是可能存在重排序，导致3在2之前发生，这可能会导致resource已经获取了引用但是并未完全构造完成，此时其他线程进入会获取到不完整的对象

  - 安全初始化

    ```java
    //提前初始化
    private Resource resource=new Resource();
    
    //或者（延迟初始化）
    public synchronized Resource resourceInit()
    {
        if (resource==null)
        {
            resource=new Resource();
        }
        return resource;
    }
    
    ```

    - 也可以通过占位类的方式延迟初始化

      ```java
      private Resource resource=new Resource();
      
      
      private static class ResourceInit{
          public static Resource resource=new Resource();
      }
      ```

      - 静态内部类的加载是在程序中调用静态内部类的时候加载的，和外部类的加载没有必然关系

  - 双重检查加锁（DCL）

    - 一种糟糕的技巧

      ```java
      private Resource resource;
      
      public Resource getResource()
      {
          //检查是否已初始化，如果已经初始化则无需同步直接返回
          if (resource==null)
          {
              //如果未初始化则第一个获取到锁的线程会初始化
              synchronized (this)
              {
                  //如果是第一个获取到锁的线程则初始化，之后的线程会直接返回
                  if (resource==null)
                  {
                      resource=new Resource();
                  }
              }
          }
          //问题所在
          return resource;
      }
      ```

      - DCL想要实现的是在常见代码路径上的延迟初始化不存在同步开销。但是DCL的问题和不安全初始化的问题一样，都有可能将部分构造的对象发布，在JDK5后如果将resource设置为volatile类型则可以使用DCL(volatile禁止重排序)，即使如此DCL已经被广泛的抛弃了，促使该模式出现的驱动力（无竞争同步的执行速度慢，以及JVM启动时慢）已经不复存在了

- ## 初始化过程的安全性

  - 如果能够保证初始化过程的安全性，则被正确构造的不可变对象可以在多线程中安全的访问

    - 对于含有final域的对象可以方式对对象的初始引用被重排序到构造过程之前（倒不如说final域的对象必须强制在构造方法中初始化或者提前初始化），所以以下构造是完全可以的

      ```java
      final Resource resource;
      
      public test22() {
          resource=new Resource();
      }
      ```

      