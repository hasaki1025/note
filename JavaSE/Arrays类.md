# Arrays类

- ## 构造方法

  ```java
  int []a=new int[10];
  int [][]a=new int[3][2];
  int[][] a=new int[3][];
  a[0]=new int[3];
  a[1]=new int[2];
  ....
  int[][] a=new int[][]{{1},{2},{3}}
  
  ```
  
- ## 常用方法
  
  - 二分搜索法
  
    | `static int`        | `binarySearch(Object[] a,  int fromIndex, int toIndex, Object key)`        使用二分搜索法来搜索指定数组的范围，以获得指定对象。 |
    | ------------------- | ------------------------------------------------------------ |
    | `static int`        | `binarySearch(Object[] a, Object key)`        使用二分搜索法来搜索指定数组，以获得指定对象。 |
    | `static   <T>  int` | `binarySearch(T[] a,  int fromIndex, int toIndex, T key, Comparator<? super T> c)`        使用二分搜索法来搜索指定数组的范围，以获得指定对象。 |
    | `static   <T>  int` | `binarySearch(T[] a,  T key, Comparator<? super T> c)`        使用二分搜索法来搜索指定数组，以获得指定对象。 |
  
    - 调用静态方法中的泛型：
  
      ```java
      Arrays.<Integer>sort();
      ```
  
  - 数组截取或填充
  
    | `static int[]`      | `copyOf(int[] original,  int newLength)`       复制指定的数组，截取或用 0 填充（如有必要），以使副本具有指定的长度。（其他类型也一样） |
    | ------------------- | ------------------------------------------------------------ |
    | `static   <T>  T[]` | `copyOf(T[] original,  int newLength)`       复制指定的数组，截取或用 null 填充（如有必要），以使副本具有指定的长度。 |
  
  | `static int[]`      | `copyOfRange(int[] original,  int from, int to)`       将指定数组的指定范围复制到一个新数组。 |
  | ------------------- | ------------------------------------------------------------ |
  | `static   <T>  T[]` | `copyOfRange(T[] original,  int from, int to)`       将指定数组的指定范围复制到一个新数组。 |
  
  - 判断数组是否相同
  
    | `static boolean` | `equals(Object[] a, Object[] a2)`        如果两个指定的 Objects 数组彼此*相等*，则返回 `true`。 |
    | ---------------- | ------------------------------------------------------------ |
  
  - 用特定值填充数组
  
    | `static void` | `fill(Object[] a,  int fromIndex, int toIndex, Object val)`       将指定的  Object 引用分配给指定 Object 数组指定范围中的每个元素。 |
    | ------------- | ------------------------------------------------------------ |
    | `static void` | `fill(Object[] a, Object val)`        将指定的 Object 引用分配给指定 Object 数组的每个元素。 |
  
  - 数组排序（数组元素小于47用插入，大于用快排）
  
    | `static void`        | `sort(Object[] a)`        根据元素的[自然顺序](../../java/lang/Comparable.html)对指定对象数组按升序进行排序。 |
    | -------------------- | ------------------------------------------------------------ |
    | `static void`        | `sort(Object[] a,  int fromIndex, int toIndex)`       根据元素的[自然顺序](../../java/lang/Comparable.html)对指定对象数组的指定范围按升序进行排序。 |
    | `static   <T>  void` | `sort(T[] a,  Comparator<? super T> c)`        根据指定比较器产生的顺序对指定对象数组进行排序。 |
  
  - 打印数组
  
  - | `static String` | `toString(Object[] a)`        返回指定数组内容的字符串表示形式。 |
    | --------------- | ------------------------------------------------------------ |

