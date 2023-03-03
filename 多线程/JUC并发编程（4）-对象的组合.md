# JUC并发编程（4）-对象的组合

- ## 设计线程安全类

  - 三要素
    - 找出所有构成该对象状态的所有变量
    - 找出约束状态变量的不变性条件（比如int类型的取值范围等，比如日期的值一定大于0）
    - 建立对象状态的并发访问管理策略
      - 在不违背不变性条件或者后验条件（某个域的下一个状态依赖于前一个状态）的情况下对其状态的访问操作进行协同

- ## 实例封装

  - 当数据被封装到一个对象中时，访问数据的路径都被限制在对象的方法上

  - 对象可以被封装到一个类的实例中（例如作为私有成员）或者封闭在一个域中（作为局部变量）或者线程中

  - 监视器模式

    - 将对象所有可变状态全部封装起来并由对象自己的内置锁来保护

      ```java
      //通过私有锁实现监视器模式
      public class PrivateLock {
      
          private final Object lock=new Object();
          Object data;
      
          void someMethod()
          {
              synchronized (lock)
              {
                  //执行一些对data的操作
              }
          }
      }
      
      ```

    - 使用内置锁同样是一种监视器模式（this变量加锁）

- ## 线程安全性委托

  - 将可变对象封装到线程安全类中，这种方式可能是线程安全的也可能不是（视情况而定）

  - 单个委托

    - 比如将get方法和put委托给ConcurrentHashMap

      ```java
      // 将线程安全委托给 ConcurrentHashMap
      
      @ThreadSafe
      public class DelegatingVehicleTracker {
      	private final ConcurrentMap<String, Point> locations;
      	private final Map<String, Point> unmodifiableMap;
      	
      	public DelegatingVehucleTracker (Map<String, Point> points) {
      		locations = new ConcurrentHashMap<String, Point>(points);
      		unmodifiableMap = Collections.unmodifiableMap(locations);	
      	}
      	
      	public Map<String, Point> getLocations () {
      		return unmodifiableMap;
      	}
      	
      	public Point getLocation (String id) {
      		return locations.get(id);
      	}
      	
      	public void setLocation (String id, int x, int y) {
      		if (locations.replace(id, new Point(x, y)) == null) {
      			throw new IllegalArgumentException ("invalid vehicle name：" + id);
      		}
      	}
      }
      
      ```

      

  - 独立的状态变量（委托给多个状态变量，要求这些变量时彼此独立的）

    ```java
    // 将线程安全性委托给多个状态变量
    
    public class VisualComponent {
    	private final List<KeyListener> keyListeners 
    		= new CopyOnWriteArrayList<KeyListener>();
    	private final List<MouseListener> mouseListeners 
    		= new CopyOnWriteArrayList<MouseListener>();
    
    	public void addKeyListener (KeyListener listener) {
    		keyListeners.add(listener);
    	}
    	
    	public void addMouseListener (MouseListener listener) {
    		mouseListeners.add(listener);
    	}
    	
    	public void removeKeyListener (KeyListener listener) {
    		keyListeners.remove(listener);
    	}
    	
    	public void removeMouseListener (MouseListener listener) {
    		mouseListeners.remove(listener);
    	}	
    }
    
    ```

  - 委托失效

    ```java
    // NumberRange 类并不足以保护它的不变性条件
    
    public class NumberRange {
        // 不变性条件：lower <= upper
    
        private final AtomicInteger lower = new AtomicInteger(0);
        private final AtomicInteger upper = new AtomicInteger(0);
    
        public void setLower (int i) {
            // 注意 --- 不安全的“先检查后执行”
            if (i > upper.get()) {
                if (i > upper.get()) {
                    throw new IllegalArgumentException("can't set lower to " + i + " > upper");
                }
                lower.set(i);
            }
        }
    
        public void setUpper (int i) {
            // 注意 --- 不安全的“先检查后执行”
            if (i < lower.get()) {
                throw new IllegalArgumentException("can't set upper to " + i + " < lower");
            }
            upper.set(i);
        }
    
        public boolean isInRange (int i) {
            return(i >= lower.get() && i <= upper.get());
        }
    }
    
    ```

    - 在setLower方法中调用了upper.get方法先进行检查后调用lower.set方法放入数据，这两个操作无法组成原子操作。

  - 发布底层的状态变量

    - 如果状态变量没有后验条件和不变性条件（也就是对这个变量可以无限制的修改），则可以安全的发布这个变量

    ```java
    // 线程安全且可变的Point类
    
    @ThreadSafe
    public class SafePoint {
        @GuardedBy("this") private int x, y;
    
        private SafePoint (int[] a) { this(a[0], a[1]); }
    
        public SafePoint (SafePoint p) { this(p.get()); }
    
        public synchronized int[] get () {
            return new int[] { x, y };
        }
    
        public synchronized void set (int x, int y) {
            this.x = x;
            this.y = y;
        }
    }
    
    ```

  - 在现有线程安全类中添加功能

    - 采用直接继承的方式扩展线程安全类并实现相应的线程安全措施

    ```java
    // 扩展Vector并增加一个“若没有则添加”方法
    
    @ThreadSafe
    public class BetterVector<E> extends Vector<E> {
    	public synchrnized boolean putIfAbsent (E x) {
    		boolean absent = !contains(x);
    		if (absent) {
    			add(x)
    		}
    		return absent;
    	}
    }
    
    ```

    

  - 客户端加锁机制

    - 如果无法对线程安全类进行继承扩展可以采用一个辅助类包装线程安全类并添加相应的线程安全方法
    - 非线程安全的“若没有则添加”

    ```java
    //这里假设 ListHelper的其他方法上没有synchronized(如get等)
    //
    @NotThreadSafe
    public class ListHelper<E> {
    	public List<E> list = 
    		Collections.synchronizedList(new ArrayList<E>());
    	
    	public synchronized boolean putIfAbsent (E x) {
    		boolean absent = !list.contains(x);
    		if (absent) {
    			list.add(x);
    		}
    		return absent;
    	}
    }
    
    /**
    add和contains中已经带有一个对list的锁，但是外部添加的锁是对于ListHelper.this加锁，这意味着 putlfAbsent 相对于 List 的其他操作来说并不是原子的，因此就无法确保当putlfAbsent执行时另一个线程不会修改链表。
    **/
    ```

    - 通过客户端加锁来实现“若没有则添加”

      ```java
      // 这里假设 ListHelper的其他方法上没有synchronized(如get等)
      //synchronized保证同时只有一个线程对list进行操作
      @ThreadSafe
      public class ListHelper<E> {
      	public List<E> list = 
      		Collections.synchronizedList(new ArrayList<E>());
      	
      	pubic boolean putIfAbsent (E x) {
      		synchronized (list) {
      			boolean absent = !list.contains(x);
      			if (absent) {
      				list.add(x);
      			}
      			return absent;
      		}
      	}
      }
      
      ```

  - 组合

    - 将对于list对象的操作交给其他List类型实现

    ```java
    // 通过组合实现“若没有则添加”
    //其实不实现List接口也可以，实现接口只是强制ImproveList实现List中的方法
    //需要保证list的引用只存在于ImproveList中（防止外部并发）
    @ThreadSafe
    public class ImproveList<T> implements List<T> {
    	private final List<T> list;
    	
    	public ImprovedList (List<T> list) { this.list = list; }
    	
    	public synchronized boolean putIfAbsent(T x) {
    		boolean contains = list.contains(x);
    		if (contains) {
    			list.add(x);
    		}
    		return !contains;
    	}
    	
    	public synchronized void clear () {
    		list.clear();
    	}
    	// ...按照类似的方式委托List的其他方法
    }
    
    ```

    - ImproveList并不关心传递的list是不是线程安全的，ImproveList所有操作都将采用自己的线程安全策略（对this加锁）

