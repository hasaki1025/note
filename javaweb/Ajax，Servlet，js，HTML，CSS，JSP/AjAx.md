# AjAx

- 特点：

  - 异步刷新：在不刷新的情况下获得http请求并响应
  - 可根据用户事件来更新页面
- 缺点
  - 无浏览历史，不能回退
  - 跨域问题：不同服务不能交互？
  - 对于爬虫（SEO）不友好：部分内容是动态加载的，源代码中没有
- 全局刷新和局部刷新
  - 全局刷新（大部分情况下）
  - 局部刷新（AJAX）

 ![image-20220415145149935](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220415145149935.png)

  - 使用Ajax步骤

    - 创建XMLHttpRequest对象

      ```js
      var http=new XMLHttpRequest();
      ```

    - 绑定事件

      - onreadystatechange函数:当异步对象的readystate发生变化时都会调用这个函数

        ```js
        var http=new XMLHttpRequest();
        		http.onreadystatechange= function(){
        			//处理请求状态变化
        			if(http.readyState ==4 &&http.status==200)
        			{
        				//可处理数据
        			}
        		}
        ```
      
      - 异步对象属性readystate
      
        - 0:请求未初始化 ，var http=new XMLHttpRequest();
        - 1：初始化异步请求对象，调用open(请求方式，请求地址，true)
        - 2：异步对象发送请求send()
        - 3：异步对象接收应答数据从服务端返回数据，XMLHttpRequest做内部处理
        - 4：异步对象已将数据解析完毕，可以读取数据
      
      - 异步对象的属性：status，用于表示网络请求状态,200表示成功，404表示找不到资源
      
    - 初始化异步请求对象
    
      - 调用open方法
      - open(请求方式，请求地址，true(用于表示该请求是异步还是同步，异步为true))
    
    - 使用异步对象发送请求
    
      - send()，发送请求
      - responseText:用于获取服务器端返回的数据
    
      ```jsp
      <%@ page contentType="text/html;charset=UTF-8" language="java" %>
      <html>
      <head>
          <title>Title</title>
          <script type="text/javascript">
              function doajax()
              {
                  var http=new XMLHttpRequest();
                  http.onreadystatechange=function () {
                      if(http.readyState==4 && http.status==200)
                      {
                      }
                  }
                  http.open("get","test/some.action",true);
                  http.send();
              }
      
          </script>
      </head>
      <body>
      <input type="button" name="button" value="ajax" onclick="doajax()"/>
      </body>
      </html>
      ```
    
      - AJAX大致执行流程图

![AJax执行流程](D:\java框架\javaweb\AJax执行流程.png)

- ### JSON

  - 类似于xml文件的数据传递文件

  - 常用的json解析器

    - Gjson：google
    - Fastjson：最快，但不规范
    - jackson：性能好
    - json-lib：不怎么用

  - 在java中创建json

    ```java
    void test() throws JsonProcessingException {
        User user = new User("zhangsan", "123", 21);
        ObjectMapper mapper = new ObjectMapper();
        //通过jackson工具类将对象转换为json
        System.out.println(mapper.writeValueAsString(user));
        System.out.println(user);
    }
    ```

    

----------------------------------

#jQuery

- 引入jQuery包

  - 通过maven添加依赖

    ```xml
    <!-- https://mvnrepository.com/artifact/org.webjars/jquery -->
    <dependency>
      <groupId>org.webjars</groupId>
      <artifactId>jquery</artifactId>
      <version>3.5.1</version>
    </dependency>
    ```

  - 在head标签中指定路径

    ```xml
    <script type="text/javascript" src="webjars/jquery/3.5.1/jquery.min.js"></script>
    ```

- 基本语法

  - 基础语法是：*$(selector).action()*

    - 美元符号定义 jQuery
    - 选择符（selector）“查询”和“查找” HTML 元素
    - jQuery 的 action() 执行对元素的操作

  - 案例

    <script type="text/javascript" src="/jquery/jquery.js"></script>
    <script type="text/javascript">
    $(document).ready(function(){
      $("button").click(function(){
      $(this).hide();
    });
    });
    </script>

  - ```js
    $(document).ready(function(){
    
      // jQuery functions go here
    
    });
    ```

    这是为了防止文档在完全加载（就绪）之前运行 jQuery 代码。

    如果在文档没有完全加载之前就运行函数，操作可能失败。下面是两个具体的例子：

    - 试图隐藏一个不存在的元素
    - 获得未完全加载的图像的大小

  - jQuery 元素选择器

    - jQuery 使用 CSS 选择器来选取 HTML 元素。

    - $("p") 选取 <p> 元素。

    - $("p.intro") 选取所有 class="intro" 的 <p> 元素。

    - $("p#demo") 选取所有 id="demo" 的 <p> 元素。

  - jQuery 属性选择器

    - jQuery 使用 XPath 表达式来选择带有给定属性的元素。

    - $("[href]") 选取所有带有 href 属性的元素。

    - $("[href='#']") 选取所有带有 href 值等于 "#" 的元素。

    - $("[href!='#']") 选取所有带有 href 值不等于 "#" 的元素。

    - $("[href$='.jpg']") 选取所有 href 值以 ".jpg" 结尾的元素。

- Ajax与JQuery

  - load方法

    - ```js
      $(selector).load(URL,data,callback);
      $("#div1").load("demo_test.txt #p1");
      ```

      - 指定在div标签中发生load事件
        - load事件：从服务器中加载数据并将返回的数据放入被选元素中
        - url后的#p1代表指定的id为p1的div标签
      - URL：请求路径
      - data：请求数据，一般用json
      - callback：回调函数
        - 可以有三个参数
          - *responseTxt* - 包含调用成功时的结果内容
          - *statusTXT* - 包含调用的状态
          - *xhr* - 包含 XMLHttpRequest 对象

  - get方法

    - ```js
      $.get(URL,callback);
      ```

      - 例

        ```js
        $("button").click(function(){
          $.get("demo_test.asp",function(data,status){
            alert("Data: " + data + "\nStatus: " + status);
          });
        });
        ```

  - post方法

    - ```js
      $.post(URL,data,callback);
      ```

      - 例

        ```js
        $("button").click(function(){
          $.post("demo_test_post.asp",
          {
            name:"Donald Duck",
            city:"Duckburg"
          },
          function(data,status){
            alert("Data: " + data + "\nStatus: " + status);
          });
        });
        ```
