# JDK源码（2）数据结构

- ## ArrayList

  - Collection接口

    - 继承了迭代器接口，定义了大量集合操作相关类

  - ArrayList本身继承了AbstractList

  - 基本变量

    ```java
    //默认容量
    private static final int DEFAULT_CAPACITY = 10;
    //空实例的数组
    
        private static final Object[] EMPTY_ELEMENTDATA = {};
    //默认大小的实例数组
    
        private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    //数组缓冲区（不可序列化）
        transient Object[] elementData; // non-private to simplify nested class access
    
        private int size;
    ```

    

  - 构造方法

    ```java
        public ArrayList(int initialCapacity) {
            if (initialCapacity > 0) {
                this.elementData = new Object[initialCapacity];
            } else if (initialCapacity == 0) {
                this.elementData = EMPTY_ELEMENTDATA;
            } else {
                throw new IllegalArgumentException("Illegal Capacity: "+
                                                   initialCapacity);
            }
        }
    
        /**
         * Constructs an empty list with an initial capacity of ten.
         */
        public ArrayList() {
            //默认创建采用空实例的数组
            this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
        }
    //通过现有集合创建
        public ArrayList(Collection<? extends E> c) {
            Object[] a = c.toArray();
            if ((size = a.length) != 0) {
                if (c.getClass() == ArrayList.class) {
                    elementData = a;
                } else {
                    //Arrays.copyOf底层采用本地方法复制数组
                    elementData = Arrays.copyOf(a, size, Object[].class);
                }
            } else {
                // replace with empty array.
                elementData = EMPTY_ELEMENTDATA;
            }
        }
    ```

  - trimToSize方法（将当前ArrayList数组的容量调整位当前列表的大小）

    ```java
    //用于记录列表在结构上被修改的次数
    protected transient int modCount = 0;
    public void trimToSize() {
        //
        modCount++;
        if (size < elementData.length) {
            //如果当前列表中有元素则创建新的数组并复制
            elementData = (size == 0)
                ? EMPTY_ELEMENTDATA
                : Arrays.copyOf(elementData, size);
        }
    }
    ```

  - ensureCapacity（容量增加）

    ```java
    //minCapacity为所需最小容量
    public void ensureCapacity(int minCapacity) {
        //当前最小容量获取（如果不是采用默认构造方法构造则最小容量为0，默认的最小容量是默认容量）
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            ? 0
            : DEFAULT_CAPACITY;
    
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
    
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
    //如果当前list采用默认构造方法构造则返回默认容量和指定的最小容量的最大值
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        //容量不足需要扩充
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    ```
    
    - grow方法
    
      ```java
      //数组最大容量（不一定数组最大容量为该值最大仍然可能取Integer.MAX_VALUE）
      private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
      
      private void grow(int minCapacity) {
          // overflow-conscious code
          int oldCapacity = elementData.length;
          //新容量为当前容量*3/2
          int newCapacity = oldCapacity + (oldCapacity >> 1);
          //如果小于最小指定容量则新容量为最小容量
          if (newCapacity - minCapacity < 0)
              newCapacity = minCapacity;
          if (newCapacity - MAX_ARRAY_SIZE > 0)
              newCapacity = hugeCapacity(minCapacity);
          // minCapacity is usually close to size, so this is a win:
          elementData = Arrays.copyOf(elementData, newCapacity);
      }
      
      private static int hugeCapacity(int minCapacity) {
          if (minCapacity < 0) // 容量溢出
              throw new OutOfMemoryError();
          return (minCapacity > MAX_ARRAY_SIZE) ?
              Integer.MAX_VALUE :
          MAX_ARRAY_SIZE;
      }
      ```
    
  - contains方法和indexOf方法
  
    ```java
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
    //找到第一个指定元素并返回索引值，找不到返回-1，lastIndexOf可以找到最后一个
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
    ```
  
  - clone方法
  
    ```java
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
    }
    ```
  
    - 本质采用深拷贝（数组内的引用不同，但是数组内的引用依旧相同）
  
  - toArray方法
  
    ```java
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
           //根据当前list大小复制数组
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        //直接复制到a中，大小仍然是list的当前大小
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
    ```
  
  - add方法
  
    ```java
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    public void add(int index, E element) {
        rangeCheckForAdd(index);
    
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
    ```
  
    - 如果采用默认构造则第一次添加元素采用默认容量（如果所需容量大于默认容量则使用指定容量）
    - 每次添加都会增加modCount
      - modCount的作用是检测迭代器迭代过程中的修改
  
  - remove方法
  
    ```java
    public E remove(int index) {
        rangeCheck(index);
    
        modCount++;
        E oldValue = elementData(index);
    //移动次数
        int numMoved = size - index - 1;
        //如果需要移动则通过拷贝后半部分
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // 设置为null便于GC回收
    
        return oldValue;
    }
    ```
  
    - 每次删除都会增加modCount
  
- ## LinkedList

  - 内部变量

    ```java
    transient int size = 0;
    //链表头指针
    transient Node<E> first;
    //链表尾指针
    transient Node<E> last;
    ```

    - 内部类Node

    ```java
    private static class Node<E> {
        E item;
        //下一个
        Node<E> next;
        //上一个
        Node<E> prev;
    
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    ```

    

  - addFirst方法

    ```java
    public void addFirst(E e) {
        linkFirst(e);
    }
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
    ```

  - addLast

    ```java
    public void addLast(E e) {
        linkLast(e);
    }
    
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
    ```

  - removeFirst

    ```java
    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
    
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        //已经没有元素了，设置last指针
        if (next == null)
            last = null;
        else
            //设置前指针
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
    
    ```

  - removeLast基本类似removeFirst

  - Spliterator

    - 一个可分流的迭代器，通常分类器只能顺序访问，Spliterator可以分为多个迭代器，方便执行并行操作

    - 主要方法

      ```java
      public interface Spliterator<T> {
      	//如果有下一个元素则对其进行action操作
          boolean tryAdvance(Consumer<? super T> action);
      	//对Spliterator之后的所有元素进行操作
          default void forEachRemaining(Consumer<? super T> action) {
              do { } while (tryAdvance(action));
          }
      	//尝试分为两个Spliterator（一般分为两半）
          Spliterator<T> trySplit();
      	//预估该Spliterator之后的元素数量
          long estimateSize();
          //如果能确定元素数量则返回size否则返回-1
          default long getExactSizeIfKnown() {
              return (characteristics() & SIZED) == 0 ? -1L : estimateSize();
          }
      }
      ```

    - Spliterator特征

      ```java
      //有序的
      public static final int ORDERED    = 0x00000010;
      //无重复
      public static final int DISTINCT   = 0x00000001;
      //可排序
      public static final int SORTED     = 0x00000004;
      //可确定大小
      public static final int SIZED      = 0x00000040;
      //无空元素
      public static final int NONNULL    = 0x00000100;
      //因此在遍历期间不能发生此类更改。
      public static final int IMMUTABLE  = 0x00000400;
      ```

      - 特征相关方法

        ```java
        int characteristics();//多个特征做或运算
        
        default boolean hasCharacteristics(int characteristics) {
            return (characteristics() & characteristics) == characteristics;
        }
        ```

  - ArrayListSpliterator

    ```java
    static final class ArrayListSpliterator<E> implements Spliterator<E> {
        private final ArrayList<E> list;
        private int index; // 当前值
        private int fence; // 最后一个元素的索引值，初始化值为-1
        private int expectedModCount; //修改次数，防并发修改
    
    
        ArrayListSpliterator(ArrayList<E> list, int origin, int fence,
                             int expectedModCount) {
            this.list = list; // OK if null unless traversed
            this.index = origin;
            this.fence = fence;
            this.expectedModCount = expectedModCount;
        }
    
        private int getFence() { // initialize fence to size on first use
            int hi; // (a specialized variant appears in method forEach)
            ArrayList<E> lst;
            if ((hi = fence) < 0) {//未初始化则初始化
                if ((lst = list) == null)//list为空则初始化为0
                    hi = fence = 0;
                else {//修改次数设置
                    expectedModCount = lst.modCount;
                    hi = fence = lst.size;//设置fence为当前list大小
                }
            }
            return hi;
        }
    
        public ArrayListSpliterator<E> trySplit() {
            int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
            return (lo >= mid) ? null : // divide range in half unless too small
            new ArrayListSpliterator<E>(list, lo, index = mid,
                                        expectedModCount);//如果可分则分一半
        }
    	//如果有下一个元素则进行操作
        public boolean tryAdvance(Consumer<? super E> action) {
            if (action == null)
                throw new NullPointerException();
            int hi = getFence(), i = index;
            if (i < hi) {
                index = i + 1;
                @SuppressWarnings("unchecked") E e = (E)list.elementData[i];
                action.accept(e);
                if (list.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                return true;
            }
            //没有下一个元素了
            return false;
        }
    
        public void forEachRemaining(Consumer<? super E> action) {
            int i, hi, mc; // hoist accesses and checks from loop
            ArrayList<E> lst; Object[] a;
            if (action == null)
                throw new NullPointerException();
            if ((lst = list) != null && (a = lst.elementData) != null) {
                if ((hi = fence) < 0) {
                    mc = lst.modCount;
                    hi = lst.size;
                }
                else
                    mc = expectedModCount;
                //逐个遍历操作
                if ((i = index) >= 0 && (index = hi) <= a.length) {
                    for (; i < hi; ++i) {
                        @SuppressWarnings("unchecked") E e = (E) a[i];
                        action.accept(e);
                    }
                    if (lst.modCount == mc)
                        return;
                }
            }
            throw new ConcurrentModificationException();
        }
    
        public long estimateSize() {
            return (long) (getFence() - index);
        }
        //有序，可确定大小，子Spliterator都可确定大小
        public int characteristics() {
            return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
        }
    }
    ```

- ## CopyOnWriteArrayList

  - 基本描述

    - 线程安全的List
    - 所有元素修改操作都通过拷贝一个副本来实现
    - 当遍历操作次数远多于修改操作次数时该集合效率较高
    - 迭代器将不会抛出ConcurrentModificationException异常，不支持迭代器本身的元素更改操作(删除、设置和添加)。这些方法会抛出UnsupportedOperationException异常。

    - 允许NULL

  - 成员变量

    ```java
    //可重入锁
    final transient ReentrantLock lock = new ReentrantLock();
    
    //内置数组，只能通过getArray/setArray访问。
    //使用volatile修饰数组引用
    private transient volatile Object[] array;
    ```

  - 构造方法

    ```java
    public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }
    
    public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            elements = c.toArray();
            //ArrayList底层的toArray()直接采用Arrays.copyOf返回数组的拷贝副本
            if (c.getClass() != ArrayList.class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
    }
    //深拷贝复制
    public CopyOnWriteArrayList(E[] toCopyIn) {
        setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
    }
    ```

  - 基本方法

    ```java
    public int size() {
        //采用getArray方法获取到最新的内置数组引用并返回其长度
        return getArray().length;
    }
    
    public boolean isEmpty() {
        return size() == 0;
    }
    ```

    - clone方法

      ```java
      public Object clone() {
          try {
              @SuppressWarnings("unchecked")
              CopyOnWriteArrayList<E> clone =
                  (CopyOnWriteArrayList<E>) super.clone();
              //重置锁（重新创建一个锁）
              clone.resetLock();
              return clone;
          } catch (CloneNotSupportedException e) {
              // this shouldn't happen, since we are Cloneable
              throw new InternalError();
          }
      }
      ```

      - 重置锁方法

        ```java
        private void resetLock() {
            UNSAFE.putObjectVolatile(this, lockOffset, new ReentrantLock());//直接操作内存（在lockOffset位置放一个ReentrantLock）
        }
        private static final sun.misc.Unsafe UNSAFE;
        private static final long lockOffset;
        static {
            try {
                UNSAFE = sun.misc.Unsafe.getUnsafe();//初始化Unsafe
                Class<?> k = CopyOnWriteArrayList.class;
                lockOffset = UNSAFE.objectFieldOffset//获取到lock该成员变量在内存中的偏移地址
                    (k.getDeclaredField("lock"));
            } catch (Exception e) {
                throw new Error(e);
            }
        }
        ```
    
        - Unsafe含有大量本地方法可以直接访问内存和class文件并提供一个底层的方法（CAS、内存屏障等）
        - Unsafe详解：https://blog.csdn.net/weixin_42073629/article/details/104489155
    
    - set方法
    
      ```java
      public E set(int index, E element) {
          final ReentrantLock lock = this.lock;
          lock.lock();
          try {
              Object[] elements = getArray();
              E oldValue = get(elements, index);
      		//如果设置的元素和之前的元素不同则采用数组复制
              if (oldValue != element) {
                  int len = elements.length;
                  Object[] newElements = Arrays.copyOf(elements, len);
                  newElements[index] = element;
                  setArray(newElements);//设置数组引用
              } else {
                  // Not quite a no-op; ensures volatile write semantics
                  setArray(elements);//否则没必要设置该元素
              }
              return oldValue;
          } finally {//解锁
              lock.unlock();
          }
      }
      ```
    
    - add方法
    
      ```java
      public boolean add(E e) {
          final ReentrantLock lock = this.lock;
          lock.lock();
          try {
              Object[] elements = getArray();
              int len = elements.length;//同样采用复制的方式
              Object[] newElements = Arrays.copyOf(elements, len + 1);
              newElements[len] = e;
              setArray(newElements);
              return true;
          } finally {
              lock.unlock();
          }
      }
      ```
    
    - remove操作
    
      ```java
      public E remove(int index) {
          final ReentrantLock lock = this.lock;
          lock.lock();
          try {
              Object[] elements = getArray();
              int len = elements.length;
              E oldValue = get(elements, index);//获取被删除的元素
              int numMoved = len - index - 1;//计算移动次数
              if (numMoved == 0)//不需要移动则直接复制
                  setArray(Arrays.copyOf(elements, len - 1));
              else {
                  Object[] newElements = new Object[len - 1];//两次复制
                  System.arraycopy(elements, 0, newElements, 0, index);
                  System.arraycopy(elements, index + 1, newElements, index,
                                   numMoved);
                  setArray(newElements);
              }
              return oldValue;
          } finally {
              lock.unlock();
          }
      }
      ```
    
    - 移除指定元素
    
      ```java
      public boolean remove(Object o) {
          Object[] snapshot = getArray();
          int index = indexOf(o, snapshot, 0, snapshot.length);
          return (index < 0) ? false : remove(o, snapshot, index);
      }
      
      /**
               * A version of remove(Object) using the strong hint that given
               * recent snapshot contains o at the given index.
               */
      private boolean remove(Object o, Object[] snapshot, int index) {
          final ReentrantLock lock = this.lock;
          lock.lock();
          try {
              Object[] current = getArray();
              int len = current.length;
              //如果数组已修改则
              if (snapshot != current) findIndex: {
                  int prefix = Math.min(index, len);//当前数组长度和index取最小值
                  //循环比对index之前所有值（）
                  for (int i = 0; i < prefix; i++) {
                      //之前的值已被修改但是需要被删除的元素还在
                      if (current[i] != snapshot[i] && eq(o, current[i])) {
                          index = i;
                          //找到被删除的元素的新的索引并退出
                          break findIndex;
                      }
                  }
                  //该元素已被删除
                  if (index >= len)
                      return false;
                  //如果需要被删除的元素的索引没有变化则跳出
                  if (current[index] == o)
                      break findIndex;
                  //否则重新获取索引
                  index = indexOf(o, current, index, len);
                  //找不到返回false
                  if (index < 0)
                      return false;
              }
             	//执行删除
              Object[] newElements = new Object[len - 1];
              System.arraycopy(current, 0, newElements, 0, index);
              System.arraycopy(current, index + 1,
                               newElements, index,
                               len - index - 1);
              setArray(newElements);
              return true;
          } finally {
              lock.unlock();
          }
      }
      ```
    
      - 标签代码块
    
        ```java
        //普通代码块(可以放在方法中)
        {
            int a=1;
        }
        
        a:{
        	int a=2;
        }
        b:{
        	int a=2;
        }
        ```
    
        - 带有标记的代码块可以在循环中使用
    
          ```java
          find:
          for (int i = 0; i < 10; i++){
              for (int j = 0; j < 10; j++) {
                  //直接跳出当i==1时的外层循环执行i==2时的外层循环
                  if (i==1 && j==5)
                  {
                      continue find;
                  }
              }
              System.out.println(i);
          }
          ```
    
    - addIfAbsent方法
    
      ```java
      public boolean addIfAbsent(E e) {
          Object[] snapshot = getArray();
          return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
          addIfAbsent(e, snapshot);
      }
      
      private boolean addIfAbsent(E e, Object[] snapshot) {
          final ReentrantLock lock = this.lock;
          lock.lock();
          try {
              Object[] current = getArray();
              int len = current.length;
              //数组已被修改,开始尝试找到需要被添加的元素
              if (snapshot != current) {
                  // Optimize for lost race to another addXXX operation
                  int common = Math.min(snapshot.length, len);
                  for (int i = 0; i < common; i++)
                  //找到了已被添加了，无需添加直接返回
                      if (current[i] != snapshot[i] && eq(e, current[i]))
                          return false;
                  //indexOf找到了已被添加了，无需添加直接返回
                  if (indexOf(e, current, common, len) >= 0)
                      return false;
              }
              //执行添加
              Object[] newElements = Arrays.copyOf(current, len + 1);
              newElements[len] = e;
              setArray(newElements);
              return true;
          } finally {
              lock.unlock();
          }
      }
      ```
    
    - forEach方法
    
      ```java
      public void forEach(Consumer<? super E> action) {
          if (action == null) throw new NullPointerException();
          Object[] elements = getArray();
          int len = elements.length;
          for (int i = 0; i < len; ++i) {
              @SuppressWarnings("unchecked") E e = (E) elements[i];
              action.accept(e);
          }
      }
      ```

      - 与ArrayList不同的是，CopyOnWriteArrayList允许遍历时修改

        ```java
        @Override
        //ArrayList版本
        public void forEach(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            final int expectedModCount = modCount;
            @SuppressWarnings("unchecked")
            final E[] elementData = (E[]) this.elementData;
            final int size = this.size;
            for (int i=0; modCount == expectedModCount && i < size; i++) {
                action.accept(elementData[i]);
            }
            //产生了修改则会抛出异常
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
        }
        ```
    
      - 原因也是因为CopyOnWriteArrayList的所有修改方法都含有锁,是线程安全的
    
    - 序列化方法
    
      ```java
      private void writeObject(java.io.ObjectOutputStream s)
          throws java.io.IOException {
      
          s.defaultWriteObject();
      
          Object[] elements = getArray();
          // Write out array length
          //第一个int是长度
          s.writeInt(elements.length);
      
          // Write out all elements in the proper order.
          for (Object element : elements)
              s.writeObject(element);
      }
      private void readObject(java.io.ObjectInputStream s)
          throws java.io.IOException, ClassNotFoundException {
      
          s.defaultReadObject();
      
          // 绑定一个新锁
          resetLock();
      
          //读取数组长度
          int len = s.readInt();
          SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, len);
          Object[] elements = new Object[len];
      
          //读取所有元素并设置
          for (int i = 0; i < len; i++)
              elements[i] = s.readObject();
          setArray(elements);
      }
      ```
    
      - [ ] 不会有线程安全问题吗?
    
    - 迭代器
    
      - COWIterator
    
        - 构造方法
    
          ```java
          private final Object[] snapshot;
          //指向下一个可被访问的元素
          private int cursor;
          
          
          private COWIterator(Object[] elements, int initialCursor) {
              cursor = initialCursor;
              snapshot = elements;
          }
          ```
    
        - next方法
    
          ```java
          public E next() {
              if (! hasNext())
                  throw new NoSuchElementException();
              return (E) snapshot[cursor++];
          }
          ```
    
          - 直接返回cursor指向的元素
    
        - hasNext方法
    
          ```java
          public boolean hasNext() {
              return cursor < snapshot.length;
          }
          ```
    
        - 修改元素的方法
    
          ```java
          public void remove() {
              throw new UnsupportedOperationException();
          }
          
          /**
                   * Not supported. Always throws UnsupportedOperationException.
                   * @throws UnsupportedOperationException always; {@code set}
                   *         is not supported by this iterator.
                   */
          public void set(E e) {
              throw new UnsupportedOperationException();
          }
          
          /**
                   * Not supported. Always throws UnsupportedOperationException.
                   * @throws UnsupportedOperationException always; {@code add}
                   *         is not supported by this iterator.
                   */
          public void add(E e) {
              throw new UnsupportedOperationException();
          }
          ```
    
          - 不支持修改元素，直接抛出UnsupportedOperationException异常
    
        - forEachRemaining方法
    
          ```java
          public void forEachRemaining(Consumer<? super E> action) {
              Objects.requireNonNull(action);
              Object[] elements = snapshot;
              final int size = elements.length;
              for (int i = cursor; i < size; i++) {
                  @SuppressWarnings("unchecked") E e = (E) elements[i];
                  action.accept(e);
              }
              cursor = size;
          }
          ```
    
          - 从当前cursor开始逐个访问
    
    - COWSubList
    
      - COW的子列表
    
      - 构造方法
    
        ```java
        private final CopyOnWriteArrayList<E> l;
        private final int offset;
        private int size;
        private Object[] expectedArray;//直接使用原数组的引用，但是通过offset和size限制访问
        
        // 只在持有COW的锁时才能创建
        COWSubList(CopyOnWriteArrayList<E> list,
                   int fromIndex, int toIndex) {
            l = list;
            expectedArray = l.getArray();
            offset = fromIndex;
            size = toIndex - fromIndex;
        }
        ```
    
      - 通过COW创建SubList
    
        ```java
        public List<E> subList(int fromIndex, int toIndex) {
            final ReentrantLock lock = l.lock;
            lock.lock();
            try {
                checkForComodification();
                if (fromIndex < 0 || toIndex > size || fromIndex > toIndex)
                    throw new IndexOutOfBoundsException();
                return new COWSubList<E>(l, fromIndex + offset,
                                         toIndex + offset);//可以看出来只有持有可重入锁时才能创建
            } finally {
                lock.unlock();
            }
        }
        ```
    
      - 检查并发修改
    
        ```java
        // 只在持有COW的锁时才能创建，且只监视数组引用是否有变化，但其实似乎很难发生
        private void checkForComodification() {
            if (l.getArray() != expectedArray)
                throw new ConcurrentModificationException();
        }
        ```
    
      - get和set方法
    
        ```java
        public E set(int index, E element) {
            final ReentrantLock lock = l.lock;
            lock.lock();
            try {
                rangeCheck(index);
                checkForComodification();//每次执行前判断是否已修改如果已修改则失败
                E x = l.set(index+offset, element);//COW执行set
                expectedArray = l.getArray();//重新获取数组引用
                return x;
            } finally {
                lock.unlock();
            }
        }
        
        public E get(int index) {
            final ReentrantLock lock = l.lock;
            lock.lock();
            try {
                rangeCheck(index);
                checkForComodification();//会检查并发修改
                return l.get(index+offset);
            } finally {
                lock.unlock();
            }
        }
        ```
    
        - 持有COW锁实现同步
    
      - size和add方法
    
        ```java
        public int size() {
            final ReentrantLock lock = l.lock;
            lock.lock();
            try {
                checkForComodification();
                return size;
            } finally {
                lock.unlock();
            }
        }
        //和set方法基本类似
        public void add(int index, E element) {
            final ReentrantLock lock = l.lock;
            lock.lock();
            try {
                checkForComodification();
                if (index < 0 || index > size)
                    throw new IndexOutOfBoundsException();
                l.add(index+offset, element);
                expectedArray = l.getArray();
                size++;
            } finally {
                lock.unlock();
            }
        }
        ```
    
      - forEach方法
    
        ```java
        public void forEach(Consumer<? super E> action) {
            if (action == null) throw new NullPointerException();
            int lo = offset;
            int hi = offset + size;
            Object[] a = expectedArray;
            if (l.getArray() != a)//检查数组引用
                throw new ConcurrentModificationException();
            if (lo < 0 || hi > a.length)
                throw new IndexOutOfBoundsException();
            for (int i = lo; i < hi; ++i) {
                @SuppressWarnings("unchecked") E e = (E) a[i];
                action.accept(e);
            }
        }
        ```
    
      - COWSublist迭代器
    
        - 创建迭代器
    
          ```java
          public Iterator<E> iterator() {
              final ReentrantLock lock = l.lock;
              lock.lock();
              try {
                  checkForComodification();
                  return new COWSubListIterator<E>(l, 0, offset, size);
              } finally {
                  lock.unlock();
              }
          }
          ```
    
        - 该迭代器不支持所有修改操作
    
          ```java
          public void remove() {
              throw new UnsupportedOperationException();
          }
          
          public void set(E e) {
              throw new UnsupportedOperationException();
          }
          
          public void add(E e) {
              throw new UnsupportedOperationException();
          }
          ```
    
        - 构造方法
    
          ```java
          private final ListIterator<E> it;
          private final int offset;
          private final int size;
          
          COWSubListIterator(List<E> l, int index, int offset, int size) {
              this.offset = offset;
              this.size = size;
              it = l.listIterator(index+offset);//偏移量设置
          }
          ```
    
        - hasNext和Next
    
          ```java
          public boolean hasNext() {
              return nextIndex() < size;//通过size限制访问
          }
          
          public E next() {
              if (hasNext())
                  return it.next();
              else
                  throw new NoSuchElementException();
          }
          ```
    
          
    
    
  
  





