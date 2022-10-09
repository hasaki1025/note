# String类

- ## 构造方法

  - 基本方法

  ```java
  String a="hello";
  ```

  - 通过字节数组和int[]初始化

  | `String(byte[] bytes, int offset, int length)`        通过使用平台的默认字符集解码指定的 byte 子数组，构造一个新的 `String`。 |
  | ------------------------------------------------------------ |
  | `String(byte[] bytes)`        通过使用平台的默认字符集解码指定的 byte 数组，构造一个新的 `String`。 |
  | `String(int[] codePoints, int offset,  int count)`       分配一个新的 `String`，它包含 Unicode  代码点数组参数一个子数组的字符。 |

  - 通过char[]初始化

    | `String(char[] value)`       分配一个新的  `String`，使其表示字符数组参数中当前包含的字符序列。 |
    | ------------------------------------------------------------ |
    | `String(char[] value, int offset, int count)`        分配一个新的 `String`，它包含取自字符数组参数一个子数组的字符。 |

  - 通过String和StringBuffer和StringBuilder初始化

    | `String(String original)`       初始化一个新创建的  `String`  对象，使其表示一个与参数相同的字符序列；换句话说，新创建的字符串是该参数字符串的副本。 |
    | ------------------------------------------------------------ |
    | `String(StringBuffer buffer)`        分配一个新的字符串，它包含字符串缓冲区参数中当前包含的字符序列。 |
    | `String(StringBuilder builder)`        分配一个新的字符串，它包含字符串生成器参数中当前包含的字符序列 |

- ## 常用方法

  - 索引相关方法

    | `int`   | `indexOf(int ch)`        返回指定字符在此字符串中第一次出现处的索引。 |
    | ------- | ------------------------------------------------------------ |
    | ` int`  | `indexOf(int ch,  int fromIndex)`       返回在此字符串中第一次出现指定字符处的索引，从指定的索引开始搜索。 |
    | ` int`  | `indexOf(String str)`        返回指定子字符串在此字符串中第一次出现处的索引。 |
    | ` int`  | `indexOf(String str,  int fromIndex)`       返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始。 |
    | ` int`  | `lastIndexOf(int ch)`        返回指定字符在此字符串中最后一次出现处的索引。 |
    | ` int`  | `lastIndexOf(int ch,  int fromIndex)`        返回指定字符在此字符串中最后一次出现处的索引，从指定的索引处开始进行反向搜索。 |
    | ` int`  | `lastIndexOf(String str)`        返回指定子字符串在此字符串中最右边出现处的索引。 |
    | ` int`  | `lastIndexOf(String str,  int fromIndex)`       返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索。 |
    | ` char` | `charAt(int index)`        返回指定索引处的 `char` 值。      |

  - 字符串比较

    | ` int` | `compareTo(String anotherString)`        按字典顺序比较两个字符串（String实现了Comparable接口） |
    | ------ | ------------------------------------------------------------ |
    | ` int` | `compareToIgnoreCase(String str)`        按字典顺序比较两个字符串，不考虑大小写。 |

  - 字符串拼接和改动

    | ` String` | `concat(String str)`        将指定字符串连接到此字符串的结尾。 |
    | --------- | ------------------------------------------------------------ |
    | `String`  | `replace(char oldChar,  char newChar)`       返回一个新的字符串，它是通过用 `newChar`  替换此字符串中出现的所有 `oldChar` 得到的。 |
    | ` String` | `substring(int beginIndex)`        返回一个新的字符串，它是此字符串的一个子字符串。begininex前的舍弃 |
    | ` String` | `substring(int beginIndex,  int endIndex)`       返回一个新字符串，它是此字符串的一个子字符串。 |
    | `String`  | `trim()`        返回字符串的副本，忽略前导空白和尾部空白。   |
    | `String`  | `toUpperCase()`        使用默认语言环境的规则将此 `String` 中的所有字符都转换为大写。 |
    | `String`  | `toLowerCase()`        使用默认语言环境的规则将此 `String` 中的所有字符都转换为小写。 |
    
  - 判断类型（是否包含）
  
    | ` boolean` | `contains(CharSequence s)`        当且仅当此字符串包含指定的 char 值序列时，返回 true。 |
    | ---------- | ------------------------------------------------------------ |
    | `boolean`  | `endsWith(String suffix)`        测试此字符串是否以指定的后缀结束。 |
    | ` boolean` | `equals(Object anObject)`        将此字符串与指定的对象比较。 |
    | ` boolean` | `equalsIgnoreCase(String anotherString)`        将此 `String` 与另一个 `String` 比较，不考虑大小写。 |
  
  - 类型转换方法
  
    | `byte[]`        | `getBytes()`        使用平台的默认字符集将此 `String` 编码为 byte 序列，并将结果存储到一个新的 byte 数组中。 |
    | --------------- | :----------------------------------------------------------- |
    | `byte[]`        | `getBytes(String charsetName)`        使用指定的字符集将此 `String` 编码为 byte 序列，并将结果存储到一个新的 byte 数组中。 |
    | `static String` | `valueOf(char[] data,  int offset, int count)`       返回 `char`  数组参数的特定子数组的字符串表示形式。 |
    | `static String` | `valueOf(Object obj)`        返回 `Object` 参数的字符串表示形式，返回toString（）的值 |

--------

# StringBuffer和StringBuilder

- ##构造方法（两个都差不多）

  | `StringBuilder()`        构造一个其中不带字符的字符串生成器，初始容量为 16 个字符。 |
  | ------------------------------------------------------------ |
  | `StringBuilder(CharSequence seq)`        构造一个字符串生成器，包含与指定的 `CharSequence` 相同的字符。 |
  | `StringBuilder(int capacity)`        构造一个其中不带字符的字符串生成器，初始容量由 `capacity` 参数指定。 |
  | `StringBuilder(String str)`        构造一个字符串生成器，并初始化为指定的字符串内容。 |

- ##常用方法

  | `StringBuilder`  | `append(Object obj)`        追加字符串（任意类型）           |
  | ---------------- | ------------------------------------------------------------ |
  | `char`           | `charAt(int index)`        返回索引的字符                    |
  | ` StringBuilder` | `delete(int start,  int end)`       Removes the characters in a substring of this  sequence.（删除指定范围内的字符串并返回删除后的字符串） |
  | `StringBuilder`  | `deleteCharAt(int index)`        Removes the `char` at the specified position in this  sequence. |
  | `StringBuilder`  | `insert(int offset,  Object obj)`       Inserts  the string representation of the `Object` argument into this  character sequence.(从offset开始插入一个Object) |
  | `String`         | `substring(int start,  int end)`       Returns a new `String` that contains a  subsequence of characters currently contained in this sequence. |
  | `StringBuilder`  | `reverse()`        Causes this character sequence to be replaced by the reverse of  the sequence.翻转字符串 |

  - 大部分String支持的方法这两个都支持

