# Java Script

- 脚本语言

  - 特点：
    - 最终运行程序是以普通文本保存的，而java是先编写java源文件编译后形成class文件
    - 基于事件驱动型语言
      - click事件：鼠标单击
      - dblclick事件：鼠标双击事件
      - focus事件：获取焦点事件
      - blur事件：失去焦点事件
    - 如何设置这些事件？
      - 设置事件句柄属性，事件句柄属性的名称通常为on+事件名称，如：onclick

- HTML嵌入js

  - 第一方式

    ```html
    <input type="button" value="hello" onclick="window.alert('123')"/>
    ```

    - 在js语言中字符串双引号和单引号都可以，在以上代码中window可以省略，实现效果同样是鼠标单击后发出弹窗显示123.

  - 第二种方式

    - 脚本块

      ```html
      <script type="text/javascript">
      			alert(123);
      </script>
      ```

      - js注释与java相同
      - 这种方式的js代码无需事件触发，自相而下的执行

  - 第三种方式

    - 外部引入js文件

      ```html
      <script src="./js/new_file.js" type="text/javascript"></script>
      ```

- js基本语法

  ```javascript
  var a=1;
  var a;
  a=1;
  var a,b,c=10;//abc中只有c是10，其他的是undefined
  ```

  - 如果不初始化则初始值为undefined（这是一个具体的值）
  - 对于js而言没必要声明变量类型
  - 控制台输出
    - console.log(123);
  - 所以的js变量都是对象，可以通过typeof查看对象类型
  - 如果变量在函数内没有声明（没有使用 var 关键字），该变量为全局变量。
  - 你可以使用索引位置来访问字符串中的每个字符，你也可以在字符串添加转义字符来使用引号
  - 可以使用内置属性 **length** 来计算字符串的长度，如var a='aaa'.length
  - === 为绝对相等，即数据类型与值都必须相等。
  - 两个数字相加，返回数字相加的和，如果数字与字符串相加，返回字符串;

  - for in循环

    ```js
    ```

    

- js函数

  - js函数定义

    ```js
    function gg(a,b)
    {
        console.log(a+":"+b);
    }
    
    gg=function(a,b){
         console.log(a+":"+b);
    }
    ```

    - 函数的调用可以在调用之前

  - 函数通过事件使用

    ```html
    <input type="button" value="sum" onclick="gg(1,1)"/>
    ```

  - 函数调用

    ```js
    gg();//输出undefined:undefined
    gg(1);//1:undefined
    gg(1,2);//1:2
    gg(1,2,3);//1:2
    ```

    - 参数赋值会从左到右逐个赋值，多的参数会被丢弃

- js代码块

  ```js
  
  list:
  {
      //js语句
      break list;
  }
  ```

  - 通过标签名可以对js代码块进行命名，同时break将可以用于非循环语句

- **constructor**属性

  - 可以通过调用constructor属性返回构造函数的描述

    ```js
    var a=string.constructor;
    ```

- js声明提升

  - 变量声明会被提升到代码块的最上端，但变量初始化不会

    ```js
    var x=5
    alert(x+" "+y)
    var y=1
    ```

    - 最后输出5 undefined
  
- js严格模式

  ```js
  "use strict";//在js代码的最前段声明
  ```

  - 使用了严格模式的js代码块对于变量的使用必须先声明后使用，同时严格模式也可用于函数中且作用域仅限于被声明的函数


-  js事件

  - blur事件
    - 文本输入框光标消失=失去焦点=触发blur
  - change事件
    - 常用于下拉列表，下拉列表改变=触发change
  - focus获取焦点
    - 文本框光标出现=focus事件
  - load事件
    - 页面加载完毕触发load事件

- js DOM

  - js内置对象

    - window：浏览器窗口

    - document：浏览器网页（文档）

      - getElementsBynames(var name)
        - 获取所有name的标签以数组返回
      
      - 在一个网页上有许多标签，也可以成为结点，可以根据结点的id来获取结点的信息
      
        ```js
        document.getElementById(id名称)
        ```
      
        - 这个方法会根据id获取标签并返回
      
        - innerHTML属性和innerText
      
          - 在js中每个标签都有一个innerHTML属性，这个属性可以设置也可以获取
      
            ```html
            <!DOCTYPE html>
            <html>
            	<head>
            		<meta charset="utf-8" />
            		<title></title>
            			
            	</head>
            	<body>
            		<div id="ttt">
            			jjj
            		</div>
            		<script>
            		function get()
            		{
            			var ele=document.getElementById("ttt");
            			ele.innerHTML='sb';
            		}
            		</script>
            		<input type="button" value="yy" onclick="get()" />
            	</body>
            </html>
            
            ```
      
            - 当点击按钮后jjj会变成sb
            - innerHTML和innerText区别
              - innerHTML的设置会将其中的内容当做html代码来解释，但innerText为纯文本解释
      
      - 除了innerHTML和innerText之外，标签中的所有属性都可以获取和赋值
      
        ```html
        <!DOCTYPE html>
        <html>
        	<head>
        		<meta charset="utf-8" />
        		<title></title>
        			
        	</head>
        	<body>
        		<div id="ttt">
        			jjj
        		</div>
        		<script>
        		function get()
        		{
        			var ele=document.getElementById("ttt");
        			ele.value='tt';
        		}
        		</script>
        		<input type="button" value="yy" onclick="get()" id="button"/>
        	</body>
        </html>
        
        ```
      
        
      
        
    

