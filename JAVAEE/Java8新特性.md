# Java8新特性

- Java 8 允许我们通过 `default` 关键字对接口中定义的抽象方法提供一个默认的实现。

  请看下面示例代码：

  ```
  // 定义一个公式接口
  interface Formula {
      // 计算
      double calculate(int a);
  
      // 求平方根
      default double sqrt(int a) {
          return Math.sqrt(a);
      }
  }
  ```

- Lambda表达式

  - 只支持函数式接口Functional Interface

    - Functional Interface

      - 接口中只有一个方法

        ```java
        @FunctionalInterface
        interface Converter<F, T> {
            T convert(F from);
        }
        ```

      - @FunctionalInterface的作用类似于@Override，不写也行，写了后如果在改接口中添加第二个方法就会报错

- 使用：：调用方法

  ```java
  class Something {
      String startsWith(String s) {
          return String.valueOf(s.charAt(0));
      }
  }
  ```

  ```java
  String abc=Something::startsWith("123");
  ```

  