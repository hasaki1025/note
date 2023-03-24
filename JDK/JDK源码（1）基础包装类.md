# JDK源码（1）基础包装类

- ## Object

  - 简介

    - 所有引用对象都继承了Object类，也就是除了基础类型以外包括对象数组和基本类型数组都是Object的子类

      ```java
      int[] ints = new int[2];
      //输出是true
      System.out.println(ints instanceof Object);
      ```

      

  - getClass方法

    ```java
    public final native Class<?> getClass();
    ```

    - 本身是一个静态方法，返回的Class对象无需强制转换（如下）

      ```java
      Number n = 0;  
      Class<? extends Number> c = n.getClass(); 
      ```

  - hashCode方法

    ```java
    public native int hashCode();
    ```

    - 通过System.identityHashCode方法和Objects.hashcode()方法都可以得到相同的值

    - 默认情况下JDK采用内存地址（或者说是引用地址）作为hashCode。

      ```java
      Object a = new Object();
      Object b = a;
      //两者相同
      System.out.println(a.hashCode());
      System.out.println(b.hashCode());
      ```

  - equals方法

    ```java
    public boolean equals(Object obj) {
            return (this == obj);
        }
    ```

    - 默认情况下比较引用地址，一般来说equals方法和hashCode方法需要同时重写，如果hashCode相同一般代表equals为true，反之亦然
    - java语言规范要求重写后的equals方法必须要有以下三种性质
      - 自反性：x.equals(x)==true
      - 对称性：x.equals(y)==y.equals(x)
      - 传递性：如果x.equals(y)==true，y.equals(z)==true，则x.equals(z)==true

  - clone方法

    ```java
    protected native Object clone() throws CloneNotSupportedException;
    ```

    - clone为protected方法，正常情况下无法使用，如果对象的类不支持克隆接口。覆盖clone方法的子类也可以抛出此异常，以指示不能克隆实例。想要使用clone方法需要实现Cloneable方法
    - 默认情况下所有数组类型都实现了Cloneable接口。
    - 默认情况下super.clone方法使用的是浅拷贝（成员变量只传递引用，并不会新建）

  - toString方法

    ```java
    public String toString() {
            return getClass().getName() + "@" + Integer.toHexString(hashCode());
        }
    ```

    - 默认情况下使用class的name+@+16进制的hashCode

  - notify方法

    ```java
    
    //唤醒正在此对象的监视器上等待的单个线程。如果有任何线程在此对象上等待，则选择其中一个线程进行唤醒。这种选择是任意的，由实现决定。
    //线程通过调用一个wait方法在对象的监视器上等待。 在当前线程放弃此对象上的锁之前，被唤醒的线程将无法继续。被唤醒的线程将以通常的方式与任何其他可能积极竞争同步该对象的线程进行竞争;例如，被唤醒的线程在成为下一个锁定该对象的线程方面没有可靠的特权或劣势。
    //此方法只能由该对象的监视器的所有者的线程调用。线程通过以下三种方式之一成为对象监视器的所有者:
    //通过执行该对象的同步实例方法。
    //通过执行同步语句的主体来同步对象。 
    //对于Class类型的对象，通过执行该类的同步静态方法。 一次只能有一个线程拥有对象的监视器
    //IllegalMonitorStateException -如果当前线程不是该对象的监视器的所有者
    public final native void notify();
    ```
    
    - 异常抛出(此方法只能由该对象的监视器的所有者的线程调用。通俗的说就是持有a对象的锁的线程才能调用该方法)
    
      ```java
      Object a = new Object();
      a.wait();
      
      new Thread(()->{
          try {
              sleep(2000);
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          }
          a.notify();
      }).start();
      ```
    
  - wait方法
  
    ```java
    public final native void wait(long timeout) throws InterruptedException;
    
    //nanos单位是纳秒，范围0-999999
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
    
        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                "nanosecond timeout value out of range");
        }
    
        if (nanos > 0) {
            timeout++;
        }
    
        wait(timeout);
    }
    
    public final void wait() throws InterruptedException {
        wait(0);
    }
    ```
  
    - 可响应中断
    - 可定时操作
    - 必须先持有锁才能调用wait
    - 如果timeout设置为0则持续等待直到唤醒
  
- ## Integer

  - 基本类图

  ![image-20230321203348374](C:\Users\pcdn\AppData\Roaming\Typora\typora-user-images\image-20230321203348374.png)

  - Number

    ```java
    public abstract class Number implements java.io.Serializable {
    
        public abstract int intValue();
    
        public abstract long longValue();
    
        public abstract float floatValue();
    
        public abstract double doubleValue();
    
        public byte byteValue() {
            return (byte)intValue();
        }
    
        public short shortValue() {
            return (short)intValue();
        }
    
        private static final long serialVersionUID = -8742448824652078965L;
    }
    ```

  - 除此之外Integer内部采用缓存机制用来缓存Integer对应的int

    ```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    ```

    - IntegerCache

      ```java
      private static class IntegerCache {
          static final int low = -128;
          static final int high;
          static final Integer cache[];
      
          static {
              // high value may be configured by property
              int h = 127;
              //从系统属性中读取缓存最大值（至少127）
              String integerCacheHighPropValue =
                  sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
              if (integerCacheHighPropValue != null) {
                  try {
                      int i = parseInt(integerCacheHighPropValue);
                      //系统属性和127之间取最大值
                      i = Math.max(i, 127);
                      // Maximum array size is Integer.MAX_VALUE
                      //最大Integer.MAX_VALUE - (-low) -1
                      h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                  } catch( NumberFormatException nfe) {
                      // If the property cannot be parsed into an int, ignore it.
                  }
              }
              high = h;
      
              cache = new Integer[(high - low) + 1];
              int j = low;
              //常量池创建
              for(int k = 0; k < cache.length; k++)
                  cache[k] = new Integer(j++);
      
              //至少要缓存-128到127
              assert IntegerCache.high >= 127;
          }
      
          private IntegerCache() {}
      }
      ```

      

  - 不可变类型的Integer

    ```java
    private final int value;
    ```

    - 内部值采用final修饰这也代表了Integer本身状态不可修改，不会存在Integer为1但是int可以修改为2的情况

  - hashcode

    ```java
    public static int hashCode(int value) {
        return value;
    }
    ```

    

- ## String类型

  - 内部变量

    ```java
    private final int value;public final class String
        implements java.io.Serializable, Comparable<String>, CharSequence {
    
        //采用char数组记录String变量
        private final char value[];
    //hash值的缓存
        private int hash;
    ```

  - hashcode方法

    ```java
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;
    
            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
    ```

    - 将字符串看成31进制的数字计算其值

  - 本地方法字符串常量池

    ```java
    //返回字符串对象的规范表示形式。 字符串池最初为空，由类String私有地维护。 
    //调用intern方法时，如果由equals(object)方法确定的池中已经包含与此string对象相等的字符串，则返回池中的字符串。否则，将此String对象添加到池中，并返回对该String对象的引用。 
    //由此可见，对于任意两个字符串s和t，当且仅当s.equals(t)为真时，s.t iintern () == t.t iintern()为真。 
    //所有字面值的字符串和字符串值的常量表达式都被存储。字符串字面量在Java™语言规范的3.10.5节中定义。 返回值: 一个与此字符串具有相同内容的字符串，但保证来自唯一字符串池
    public native String intern();
    ```

    

