# CSS

- CSS基本样式

  ```CSS
  选择器
  {
      属性名:属性值;
      属性名:属性值
  }
  ```

- CSS注释和java中的块注释一样

- CSS与html的结合

  - 第一种：在标签的style属性中设置属性和属性值

  ```html
  修改边框的宽度，实虚，颜色
  <span style="border: 1px solid red">fuckyou</span>
  <span style="border: 2px solid green">fuckyou</span>
  <span style="border: 10px solid gray">fuckyou</span>
  <span style="border: 19px solid blue">fuckyou</span>
  缺点：
  属性多代码多
  可读性差
  代码复用性差
  ```

  - 第二种方式（在head中定义style标签）

    ```html
    <style type="text/css">
            /*在这里的代码都是CSS代码*/
            spad{
                border: 4px soild red;
            }
            div{
                border: 4px soild red;
            }
            /*这样写的代码复用性高，但只能在一个页面中复用，维护起来也不方便*/
        </style>
    ```

  - 第三种方式（将CSS文件单写成一个CSS文件）

    - 编写CSS文件

      ```CSS
      span{
          border:2px solid green;
      }
      div{
          border:3px solid red;
      }
      ```

    - 引用CSS文件

      ```html
      引用CSS文件
      <link rel="stylesheet" type="text/css" href="csstext.css">
      ```

- CSS选择器

  - 标签选择器（以上使用的就是标签选择器）

  - id选择器

    - 格式

      ```CSS
      #id编号
      {
          属性:属性值;
      }
      ```

      - 使用格式

        ```html
        <标签 id="id编号">
        ```

    - 案例

      ```CSS
      #id001
      {
          border:10px solid green;
          color:gold;
          front-size:7
      }
      #id002
      {
          border:6px dashed gold;
          color:gray;
          front-size:2
      }
      ```

      ```html
      <span id="id001">gggg</span>
      <div id="id002">yyyyyyyyyyy</div>
      ```

  - 类选择器

    - 定义方法

      ```CSS
      .类名
      {
          属性名:属性值;
      }
      这种定义方式可以理解为对所有的标签都是这种格式
      标签名.类名
      {
          属性名:属性值;
      }
      第二种定义方式可以使在不同标签中使用相同的类时有不同的效果（类似于重载）
      ```

      - 使用方法

        ```html
        <标签名 class="类名">
        ```

    - 案例

      ```CSS
      div.class01 {
          border:1px solid gold
      }
      
      span.class01 {
          border:10px solid gold;
      }
      .class02 {
          color:green;
          front-size:7
      }
      ```

      ```html
      <span class="class01"  >gggg</span>
      <div class="class01">yyyyyyyyyyy</div>
      ```

  - 组合选择器

    - 定义方式

      ```CSS
      选择器01,选择器02...
      {
          属性:属性值;
      }
      ```

      - 组合选择器可以理解为两种选择器采用同一种渲染方案

      - 定义案例

      ```CSS
      #id001,div.class01
      {
          border:10px solid green;
          color:gold;
          front-size:7
      }
      ```

      

  