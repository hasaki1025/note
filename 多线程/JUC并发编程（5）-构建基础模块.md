# JUC并发编程（5）-构建基础模块

- ## 同步容器类

  - 同步容器类的问题

    - 错误示例

      ```java
      public static Object getLast(Vector list)
      {
          return list.get(list.size()-1);
      }
      
      public static Object deleteLast(Vector list)
      {
          return list.remove(list.size()-1);
      }
      /**
      同时启动两个线程，一个获取最后的元素，一个删除最后的元素，在并发的情况下可能会出现
      get方法先获取到最后一个元素的索引值然后另一个线程删除了最后一个元素，此时再进行获取就会报错（索引值已失效）
      **/
      ```

      - 对于多个同步容器调用的方法组合属于复合操作，需要加锁保护

  - 迭代器和ConcurrentModificationException

    - 当使用迭代器遍历一个集合的同时删除或者添加元素，则会抛出ConcurrentModificationException。

    - 隐藏迭代器

      ````java
      public class HiddenIterator {
          @GuardedBy("this") private final Set<Integer> set = new HashSet<Integer>();
      
          public synchronized void add(Integer i) {
              set.add(i);
          }
      
          public synchronized void remove(Integer i) {
              set.remove(i);
          }
      
          public void addTenThings() {
              Random r = new Random();
              for (int i = 0; i < 10; i++)
                  add(r.nextInt());
              //再这里直接输出toString方法会自动遍历set集合
              System.out.println("DEBUG: added ten elements to " + set);
          }
      }
      ````

      - 类似的方法由hashCode\equals或者containsAll、removeAll、retainAll等方法

- ## 并发容器

  - ConcurrentHashMap
    - ConcurrentHashMap并不是在每一个方法上都在同一个锁上同步，而是采用一种粒度更细的加速机制来实现更大程度的共享，这种机制称为分段锁，在这种同步机制下任意数量的读取线程可以并发的访问Map，读线程和写线程都可以并发的访问Map
    - ConcurrentHashMap的迭代器不会抛出ConcurrentModificationException
    - 额外的原子操作
      - ConcurrentHashMap能被加锁执行独占访问，所以无法使用客户端加锁来创建新的原子操作，但是大部分复合操作在ConcurrentHashMap都有实现

  