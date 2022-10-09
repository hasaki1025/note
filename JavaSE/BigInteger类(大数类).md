#常用类

-----------------------

#	BigInteger类(大数类)

- ## 构造方法

  | `BigInteger(byte[] val)`       将包含  BigInteger  的二进制补码表示形式的 byte 数组转换为 BigInteger。 |
  | ------------------------------------------------------------ |
  | `BigInteger(String val)`        将 BigInteger 的十进制字符串表示形式转换为 BigInteger。 |

- ##对大数类的一些操作

  | ` BigInteger` | `abs()`        返回其值是此 BigInteger 的绝对值的 BigInteger。 |
  | ------------- | ------------------------------------------------------------ |
  | ` BigInteger` | `add(BigInteger val)`       返回其值为 `(this +  val)` 的 BigInteger。 |
  | `BigInteger`  | `divide(BigInteger val)`       返回其值为 `(this /  val)` 的 BigInteger。 |
  | `int`         | `compareTo(BigInteger val)`       将此 BigInteger 与指定的 BigInteger 进行比较。 |
  | `BigInteger`  | `remainder(BigInteger val)`       返回其值为 `(this %  val)` 的 BigInteger。 |
  | `BigInteger`  | `multiply(BigInteger val)`       返回其值为 `(this *  val)` 的 BigInteger。 |
  | `BigInteger`  | `subtract(BigInteger val)`       返回其值为 `(this -  val)` 的 BigInteger。 |
  | `BigInteger`  | `pow(int exponent)`        返回其值为 `(thisexponent)` 的 BigInteger。 |
  | ` String`     | `toString()`        返回此 BigInteger 的十进制字符串表示形式。 |

--------------

# Random(随机类)

- 一般直接使用无参构造函数

- ##常用方法

  | `int`  | `nextInt()`        返回下一个伪随机数，它是此随机数生成器的序列中均匀分布的 `int` 值。 |
  | ------ | ------------------------------------------------------------ |
  | ` int` | `nextInt(int n)`        返回一个伪随机数，它是取自此随机数生成器序列的、在 0（包括）和指定值（不包括）之间均匀分布的 `int`  值。 |

  - boolean,float和double和int一样但不可以指定范围

------------

# Math类

- 经常使用其静态方法

- 常用方法(都是静态方法)

- | `static double`       | `pow(double a,  double b)`       返回第一个参数的第二个参数次幂的值。 |
  | :-------------------- | ------------------------------------------------------------ |
  | int,double,float,long | min () 和max()函数                                           |
  | int,double,float,long | abs()绝对值函数                                              |
  | `static double`       | `sqrt(double a)`        返回正确舍入的 `double` 值的正平方根。 |