#JSP (Java Serve Pages)

##基于java语言实现的服务器端的页面

- 底层原理
  - web容器（服务器）将jsp文件翻译为java文件，再将java文件编译为class文件
    - 例
      - 假设编写了一个index.jsp，在Tomcat服务器中运行
      - Tomcat创建了一个java类名为index_jsp并继承一个类HttpJspBase并实现了一系列接口
        - HttpJspBase为HttpServlet的子类
    
  - 综上，jsp被web容器翻译后的java文件是一个Servlet，所以jsp的生命周期和Servlet相同，所以jsp有一个缺点，第一次访问较慢，因为需要Tomcat需要翻译和编译生成java文件
  
  - 在jsp中直接编写的文字都会以Servlet中service方法中的out.prinln()的方式直接输出
  
    - 解决中文乱码问题
  
      ```jsp
      <%@
      page contentType="text/html;charset=UTF-8"
      %>
      ```
  
- ##JSP基础语法

  - <%java代码%>

    - <%%>中的java代码会被直接翻译到service中

  - jsp注释

    ```jsp
    <%--
        注释内容
        --%>
    ```

  - <%!  java代码%>

    - 在这个标签中的java代码会被翻译在service方法外类的内部
    - 但这种标签使用较少
      - 在service外建立的实例变量和静态变量在多线程环境下的web容器中访问存在线程安全问题

  - jsp表达式

    - <%= java代码%>：java代码会被转换为字符串直接输出，末尾不能有分号

    - 例：

      ```java
      //jsp代码
      <%= (new java.util.Date()).toLocaleString()%>
      //翻译后的java代码
      out.print( (new java.util.Date()).toLocaleString());
      ```

  - jsp指令

    - <%@指令标签 %>
      - page
      - include
      - taglib

