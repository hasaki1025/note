# JUC并发编程（2）线程安全性

- ## 什么是线程安全性

  - 什么是线程安全类

    - 当多个线程访问某个类时,不管运行时环境再用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或者协同，这个类都能表现出正确的行为，那么这个类就是线程安全的。

  - 示例一：

    - 一个无状态的Servlet

      ```java
      public class MyServlet implements Servlet {
      
      
          @Override
          public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
              BigInteger integer= extractFromRequest(servletRequest);
              BigInteger[] integers=factor(integer);
              encodeIntoResponse(integers);
          }
      }
      ```

      - 这个Servlet不包含任何域也不包含任何对其他类中域的引用，所用到的所有变量都是局部变量，都封闭在一个线程的栈中，不会影响其他线程访问。
      - 无状态对象一定是线程安全的

  - 原子性

    - 以上示例上如果添加一个long类型的域，则就不是无状态对象，无状态可以理解为之前的执行不会影响之后的执行

      ```java
      public class MyServlet implements Servlet {
      
      private long count=0;
          @Override
          public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
              BigInteger integer= extractFromRequest(servletRequest);
              BigInteger[] integers=factor(integer);
              count++;
              encodeIntoResponse(integers);
          }
      }
      ```

  - 竟态条件

    - 最常见的竟态条件是先检查后执行操作，当一个计算的结果取决于多个线程的交替执行时序时，就会发生竟态条件

    - 示例：延迟初始化

      ```java
      @NotThreadSafe
      public calss LazyInitRace{
          private ExpensiveObject instance = null;
          public ExpensiveObject getInstance()
          {
              if(instance == null)
              {
                  instance = new ExpensiveObject();
              }
              return instance;
          }
      }
      ```

  - 原子类

    - AtomicLong等类
      - 这些类的递增等方法都是原子操作，可以保证线程安全性，但是多个原子操作组成的复合操作并不能保证线程安全
      - 实现原理
        - CAS循环（Compare And swap）
          - 三个参数，内存中的值V，程序之前读取到的值A（期望值），将要写入的值B
          - CAS（unsafe.compareAndSwapInt()方法）
            - 写入新值（B）前检查V和A是否相等
              - 如果相等则写入内存中
              - 如果不相等则说明A已过期（与其他线程冲突），则返回false
          - CAS循环
            - 执行CAS，如果返回false则重新读取新值并再次进行CAS判断
  
  - 加锁机制
  
    - 内置锁
  
      - synchronized指定的锁同时只能由一个线程持有，加在变量上是变量锁，在方法上是对方法调用者的锁，加载类上表示该类的所有非静态方法都含有锁，加在class对象上表示所有静态方法上的锁
      - synchronized加到静态方法上或者synchronized(class)代码块是给Class类上锁。Class锁对类的所有对象都其作用
  
      - 重入
        - 如果某个线程是试图获得一个已经由它自己持有的锁，那么这个请求就会成功