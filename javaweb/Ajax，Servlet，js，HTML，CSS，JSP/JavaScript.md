# JavaScript

- javaScript特点

  - javaScript是弱类型语言
  - 交互性
  - 安全性（不访问本地硬盘）
  - 跨平台性（有浏览器就能用）

- javascript与html联合

  - 第一种方式

  ```html
  <head>
      <meta charset="UTF-8"><!--表示字符的编码 -->
      <title>Title</title><!-- 表示标题-->
      <script type="text/javascript" >
  
          //javascript提供的一个警告框可以接受任何参数
          alert("hello world!")
          //在打开网页的时候加载
      </script>
  </head>
  ```

  - 第二种方式（引用javascript文件）

    ```html
    <script type="text/javascript" src="firstjavascript.js">
        </script>
    src下既可以是相对路径也可以是绝对路径
    ```

    - 两种方式二选一（同时存在选第一个）
    - 记得在以 “(“、”[“ 、”/“、”+”、”-“ 开头的js语句前面都加上一个分号

- ##变量

  - 数值类型

  - 字符串类型

  - 对象类型

  - 布尔类型

  - 函数类型

  - 某些特殊值

    - undifned:未初始化的默认值
    - null：空
    - NAN：非数字

  - typeof函数(以字符串返回变量类型)

  - 在javascirpt中0，null，undefined，空字符串都可以表示为布尔类型的false

  - NAN的出现

    ```javascript
    var a=12;
    var b="12"
    alert(a*b);
    //结果为NAN
    ```

  - 布尔运算

    - 且运算
      - 当表达式为真时返回最后一个表达式的值
      - 为假时返回第一个为假的表达式的值
    - 或运算
      - 当表达式为假时返回最后一个表达式的值
      - 为真时返回第一个为真的表达式的值
    - javascript布尔运算存在短路现象

  - 数组

    - 一个数组可以存不同类型的变量
    - 只要对数组下表赋值（无论存不存在），就会自动给数组扩容。
    - 可以使用var a=[1,2,3]这种方式为数组初始化

  - 函数

    - 第一种定义方式

      ```javascript
      function fun()
      {
          alert("sss");
      }
      
      function fun(a,b)
      {
      alert(a);
      alert(b);
      return b;
      }
      ```

    - 第二种

      ```javascript
      var fun=function (a,b){alert("sss");}
      ```

    - javascript不允许函数重载，再次定义会覆盖之前的值。

    - 隐形参数arguments：在函数中不定义也可以获取所有参数，类似与java可变长参数

      ```javascript
      function fun()
      {
          alert(arguments[0]);
          alert(arguments[1]);
          alert(arguments[2]);
          alert(arguments.length);
      }
      
      fun(1,2,3,4);
      
      
      //输出：1 2 3 4
      ```

      - arguments是所有参数的一个数组。

        ```javascript
        function fun(a)
        {
            alert(a);
            alert(arguments[0]);
            alert(arguments.length);
        }
        
        fun(1,2,3,4);
        //输出1 1 4
        ```

  - 自定义对象

    - 第一种方式自定义对象

      ```javascript
      var aa=new Object();
      aa.fun = function (){alert("s");};
      aa.gg=[1,2,3];
      aa.tt="aasaa";
      ```

    - 第二种方式自定义对象

      ```javascript
      var bb={
          name:"hhhh",
          age:21,
          hobby:function(){
              alert("asfa");
          },
          gg:function () {
              alert(444);
          }
      }
      ```

  
  - javascript事件
  
    - onload 加载完成事件 页面初始化操作
  
      - 静态注册
  
        ```javascript
        <script type="text/javascript" >
                function onload()
                {
                    alert("Loading...这是静态注册onload方法");
                }
            </script>
        ```
  
        - 使用
  
          ```javascript
          <body onload="onload();">
          ```
  
      - 动态注册
  
        ```javascript
        <script type="text/javascript" >
                window.onload=new function () {
                    alert("这是动态注册onload方法");
                }
            </script>
        ```
  
        - 使用与静态注册相同
  
    - onclick 单击事件  按钮点击操作
  
    - onblur 失去焦点事件 输入框验证输入内容是否合法
  
    - oncharge 内容改变事件 下拉列表和输入框内容改变后事件
  
    - onsubmit 表单提交事件 表单提交前，验证表单项是否合法

