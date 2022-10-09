# Data类（日期类）

- ##构造方法

  | `Date(long date)`        使用给定毫秒时间值构造一个 `Date` 对象。 |
  | ------------------------------------------------------------ |
  | Date( )                              使用当前日期和时间来初始化对象 |

  - 当使用无参构造方法初始化Date时返回的创建的是现在的时间

- ## 设置时间

  | ` void` | `setTime(long date)`        使用给定毫秒时间值设置现有 `Date` 对象。 |
  | ------- | ------------------------------------------------------------ |

  - 分配 `Date` 对象并初始化此对象，以表示自从标准基准时间（称为“历元（epoch）”，即 1970 年 1  月 1 日 00:00:00 GMT）以来的指定毫秒数。

- ## 时间格式

  | ` String`     | `toString()`        格式化日期转义形式 yyyy-mm-dd 的日期。   |
  | ------------- | ------------------------------------------------------------ |
  | `static Date` | `valueOf(String s)`        将 JDBC 日期转义形式的字符串转换成 `Date` 值。 |

  一个包装了毫秒值的瘦包装器 (thin wrapper)，它允许 JDBC 将毫秒值标识为 SQL `DATE` 值。毫秒值表示自 1970  年 1 月 1 日 00:00:00 GMT 以来经过的毫秒数。 

  为了与 SQL `DATE` 的定义一致，由 `java.sql.Date`  实例包装的毫秒值必须通过将小时、分钟、秒和毫秒设置为与该实例相关的特定时区中的零来“规范化”。 

- ## 比较时间

  | ` boolean` | `after(Date when)`        测试此日期是否在指定日期之后。  |
  | ---------- | --------------------------------------------------------- |
  | ` boolean` | `before(Date when)`        测试此日期是否在指定日期之前。 |
  | ` int`     | `compareTo(Date anotherDate)`        比较两个日期的顺序。 |
  | ` boolean` | `equals(Object obj)`        比较两个日期的相等性。        |



#Calendar类

- ##构造方法(通常通过静态方法构造)

  | `static Calendar` | `getInstance()`        使用默认时区和语言环境获得一个日历。  |
  | ----------------- | ------------------------------------------------------------ |
  | `static Calendar` | `getInstance(TimeZone zone, Locale aLocale)`        使用指定时区和语言环境获得一个日历。 |

- ## 设置时间

  | `void`  | `set(int year,  int month, int date)`       设置日历字段  `YEAR`、`MONTH` 和 `DAY_OF_MONTH` 的值。 |
  | ------- | ------------------------------------------------------------ |
  | ` void` | `set(int year,  int month, int date, int hourOfDay, int minute)`       设置日历字段  `YEAR`、`MONTH`、`DAY_OF_MONTH`、`HOUR_OF_DAY`  和 `MINUTE` 的值。 |
  | ` void` | `set(int year,  int month, int date, int hourOfDay, int minute, int second)`        设置字段  `YEAR`、`MONTH`、`DAY_OF_MONTH`、`HOUR`、`MINUTE`  和 `SECOND` 的值。 |
  | ` void` | `setTime(Date date)`        使用给定的 `Date` 设置此 Calendar 的时间。 |
  | ` void` | `setTimeInMillis(long millis)`        用给定的 long 值设置此 Calendar 的当前时间值。 |

- ##获取时间

  | ` long` | `getTimeInMillis()`        返回此 Calendar 的时间值，以毫秒为单位。 |
  | ------- | ------------------------------------------------------------ |
  | `int`   | `get(int field)`        返回给定日历字段的值。               |

  - 关于get方法
    - 例如：calendar.get(Calendar.MONTH); 返回一个整数，如果该整数是0表示当前日历是在一月，该整数是1表示当前日历是在二月等。
    - 例如：calendar.get(Calendar.DAY_OF_WEEK);返回一个整数，如果该整数是1表示星期日，如果是2表示是星期一，依次类推，如果是7表示是星期六。 

- ## 日期格式化

  - String 类中的format()静态方法

    | `static String` | `format(String format, Object... args)`        使用指定的格式字符串和参数返回一个格式化字符串。 |
    | --------------- | ------------------------------------------------------------ |

    ```java
    String s = String.format("%tY年--%tm月--%td日",new Date(),new Date(),new Date());
    System.out.println(s);
    String s3 = String.format("%tY年%<tm月%<td日",new Date());
    //<表示与前面的格式相同
    System.out.println(s3);
    ```

    - String format()函数可以用于数字格式化
    
      ```java
      String s = format("从左向右：%d,%.3f,%d",x,y,100);
      ```
    
      - 如果想要改变赋值顺序:
    
        ```java
        String s=String.format(“不是从左向右：%2$.3f,%3$d,%1$d”,x,y,100);
        //通过数字加$可以设置顺序
        ```
    
    - 数据的宽度就是format方法返回的字符串的长度。规定数据宽度的一般格式为“%md”,其效果是在数字的左面增加空格;或"%-md“,其效果是在数字的右面增加空格,
      	例如，将数字59格式化为宽度为8的字符串：
    
                      ```java
                      String s=String.format("%8d",59);	          
                      字符串s就是：“      59"，其长度（s.length()）为8，即s在59左面添加了6个空格字符。
                      ```
    
    - 在宽度的前面增加前缀0,表示用数字0（不用空格）来填充宽度左面的富裕部分，例如：
    
      ```java
      String s=String.format(“%011f”,59.88); 字符串s就是："0059.880000"，其长度（s.length()）为11。
      ```
    
      
    
        

