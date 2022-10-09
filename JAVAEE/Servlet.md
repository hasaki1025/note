# Servlet

-  ###Servlet生命周期

  - 对于非服务器创建的servlet对象并不受服务器管理，即创建和销毁都是由自己实现，以下的生命周期均为服务器（web容器）下的生命周期
  
  - （1）服务器启动时不会实例化Servlet对象，但是可以在标签中声明使其实例化
  
    ```xml
    <servlet>
        <load-on-startup>0</load-on-startup>
    </servlet>
    ```
  
    - 中间的数字任意正数
  
  - （2）服务器获取响应时创建servlet对象（底层：服务器创建一个类似于HashMap的集合，key存访问路径，value存servlet对象）
  
    - （1）servlet调用无参构造方法，只会调用无参构造，若只有有参构造会报错http500，建议不修改无参构造方法
  
    - （2）servlet调用init方法，一般用建立数据库连接池等，GenericServlet含有一个带参数的和一个无参数的
  
      ```java
      void init(ServletConfig config)//其中的ServletConfig由Tomcat初始化servlet时传递参数
      ```
  
      - 两个init方法优先执行带参数的init方法
  
    - （3）servlet调用service方法，每次浏览器刷新都会发送一个请求，并且servlet响应一次，这表明servlet有且仅一个实例，但这并不代表servlet是单例模式，真正的单例模式的构造方法是私有化的，servlet的假单例现象是由Tomcat创建servlet方式决定的（多线程共享机制），init和无参构造只执行一次
  
  - （3）服务器关闭，destroy方法调用（仅一次），一般用于保存资源，之后服务器销毁servlet内存，

- servlet的五个方法中最重要的方法为service方法，所以可以采用适配器设计，对于不常用的方法在一个虚类实例，对于常用的service方法设置为虚函数，每次需要编写servlet类时直接继承对应的适配器（类），而javaEE中配置了GenericServlet这个类作为适配器，该类对于除了service方法外其他方法均实现。

- ###ServletConfig接口

  - 实现

    - Web容器实现，不同Web容器实现不同
    - 对于不同的servlet，ServletConfig不同，在Servlet创建时同时创建
    - GenericServlet继承了这个接口并实现了方法，在GenericServlet可以直接使用ServletConfig中的方法
  
  - 作用
  
    - 将xml文件中<servlet></servlet>中的内容包装，如servlet名称
  
      - 主要方法
  
         - | Modifier and Type     | Method and Description                                       |
            | --------------------- | ------------------------------------------------------------ |
            | `String`              | `getInitParameter(String name)`  获取指定名称的初始化参数的值 |
            | `Enumeration<String>` | `getInitParameterNames()`  获取所有初始化参数的名称          |
            | `ServletContext`      | `getServletContext()`  Returns a reference to the [`ServletContext`](../../javax/servlet/ServletContext.html)  in which the caller is executing.  译: 返回对调用者正在执行的 [`ServletContext`](../../javax/servlet/ServletContext.html)的引用。 |
            | `String`              | `getServletName()` 返回此servlet实例的名称。                 |
         
      - servlet初始化参数信息配置
      
         ```xml
         <servlet>
               <servlet-name>hello</servlet-name>
               <servlet-class>Myservlet</servlet-class>
                 <init-param>
                     <param-name>driver</param-name>
                     <param-value>com.mysql.cj.jdbc.Driver</param-value>
                 </init-param>
         
                 <init-param>
                     <param-name>url</param-name>
                     <param-value>jdbc:mysql://localhost:3306/wdnmd</param-value>
                 </init-param>
         
                 <init-param>
                     <param-name>password</param-name>
                     <param-value>root123</param-value>
                 </init-param>
             </servlet>
         ```
      
         - 这些配置信息可以通过getInitParameterNames（）和getInitParameter(String name)方法得到，配置信息所记录的值都是String
      
  
- ###ServletContext接口（Servlet上下文）

  - 创建

    - Tomcat服务器（web容器）

    - Web容器启动时创建
    - 对于一个webapp而言，Servlet只有一个

    - Web容器关闭时销毁

  - 作用

    - 管理多个Servlet对象
    - 保存多个Servlet对象的配置信息
    - 可以理解为，Tomcat中包含多个ServletContext，一个ServletContext又包含多个Servlet对象，同时一个Servlet对象又对应着一个ServletConfig对象

  - 设置Context初始化参数

    ```xml
    <context-param>
            <param-name>name</param-name>
            <param-value>fuck you</param-value>
            
        </context-param>
        
        <context-param>
            <param-name>ttt</param-name>
            <param-value>fuck you</param-value>
        </context-param>
    ```
  
    - 一个context-param标签中只能有一个param-name以及值
    - 对于所有servlet而言该初始化参数是共享的
  
  - 获取Context初始化参数
  
     - | `String`              | `getInitParameter(String name)`  Returns a `String` containing  the value of the named context-wide initialization parameter, or  `null` if the parameter does not exist.  译: 返回  `String`包含命名上下文范围的初始化参数的值，或 `null`如果该参数不存在。 |
        | --------------------- | ------------------------------------------------------------ |
        | `Enumeration<String>` | `getInitParameterNames()`  Returns the names of the context's  initialization parameters as an `Enumeration` of `String`  objects, or an empty `Enumeration` if the context has no  initialization parameters.  译: 返回上下文的初始化参数的名称为 `Enumeration`的  `String`的对象，或空 `Enumeration`如果上下文没有初始化参数。 |
        
     - 使用方法和ServletConfig的两种方法相同
     
  - 与ServletConfig的两种方法的区别，
  
     - 可以理解为ServletContext设置的为全局变量而ServletConfig是局部变量
  
  - 获取项目根目录
  
     ```java
     ServletContext context =this.getServletContext();
     String path=context.getContextPath();
     PrintWriter writer=servletResponse.getWriter();
     writer.println(path)
     ```
  
     - 注意：如需使用getServletContext方法，则必须初始化ServletConfig（通过带参数的init方法）
  
  - 获取文件绝对路径
  
     - ```java
       String real_path=context.getRealPath("路径名称");
       //例：
       String real_path=context.getRealPath("/index.html");
       String real_path=context.getRealPath("index.html");
       //路径名称为相对于项目根目录的路径
       ```
  
  - 日志记录功能
  
     - ```java
       ServletContext context =this.getServletContext();
       context.log("我是Tom猫");
       ```
  
       - 如果不通过IDEA创建该项目，则Tomcat会将log中的内容记录到Tomcat根目录下的logs文件夹中，但如果使用IDEA工具则，IDEA会在部署服务器时为服务器创建一个副本存放在IDEA工具下，路径会在启动服务器时弹出
  
         ```txt
         Using CATALINA_BASE:   "C:\Users\房间内红茶香气依旧\AppData\Local\JetBrains\IntelliJIdea2021.3\tomcat\5ae61c2f-3053-4f7c-9e92-522df932d460"
         
         ```
  
       - 在该目录下可以找到某个文件中含有
  
         ```txt
         30-Mar-2022 16:52:03.434 信息 [http-nio-8080-exec-9] org.apache.catalina.core.ApplicationContext.log 我是Tom猫
         ```
  
     - 同时log还有记录异常的功能
  
       ```java
        context.log("age smaller than 18",new RuntimeException("FBI OPEN THE DOOR!!!"));
       ```
  
       - 日志记录
  
         ```txt
         30-Mar-2022 17:06:01.708 严重 [http-nio-8080-exec-4] org.apache.catalina.core.ApplicationContext.log age smaller than 18
         	java.lang.RuntimeException: FBI OPEN THE DOOR!!!
         ```
  
  - ServletContext共享数据
  
     - ServletContext的另一个名称：应用域
  
     - ServletContext中存放一部分对于所有servlet共享的数据，这部分数据的数据量小，只读，利于提高执行效率，这部分数据以Map集合的方式存放
  
     - 常用方法
  
        - | `Object`              | `(String name)`  Returns the servlet container attribute  with the given name, or `null` if there is no attribute by that name.   译: 返回具有给定名称的servlet容器属性，如果该名称没有属性，则返回 `null` 。 |
           | --------------------- | ------------------------------------------------------------ |
           | `Enumeration<String>` | `getAttributeNames()`  Returns an `Enumeration`  containing the attribute names available within this ServletContext.  译:  返回包含此ServletContext中可用的属性名称的 `Enumeration` 。 |
  
           - | `void` | `removeAttribute(String name)`  Removes the attribute with the given name  from this ServletContext.  译: 从此ServletContext中删除具有给定名称的属性。 |
              | ------ | ------------------------------------------------------------ |
              | `void` | `setAttribute(String name,  Object object)`  Binds an object to a given attribute name  in this ServletContext.  译: 将对象绑定到此ServletContext中的给定属性名称。 |
              
              ```java
              ServletContext context =this.getServletContext();
              User ttt = new User("ttt", "123");//增
              User my_user = (User)context.getAttribute("MY_USER");//取
              context.removeAttribute("MY_USER");//删
              ```
        
     
  - 以上就是关于GenericServlet中有关类的解析，但是在开发中我们并不常使用继承GenericServlet，而是采用GenericServlet的子类HttpServlet，在HttpServlet中实现了两个service方法，一个由GenericServlet继承而来，在该方法中调用另一个service方法，也就是HttpServlet自己的方法，HttpServlet的service方法中会根据http请求类型分别调用不同的方法，如doGet（）方法和doPost（）方法。
  
  -----------------------

- ###Http协议

  - 由W3c（万维网联盟，制定了html，xml，CSS等 ）制定的一种超文本传输协议
    - 超文本协议：可以传输图片，声音，视频等
    - http协议支持：普通文本，图片，声音，视频等
    
  - B和S之间的通信遵守这种协议，使得B不依赖S,S不依赖B
  
  - http包含
  
    - 请求协议
  
      - 请求行
  
        ```xml
        GET /untitled28_war/ HTTP/1.1		请求行
        ```
  
        - 如果是GET请求方式，则表单内容（比如密码和用户名等）会在请求行中表现出，但POST方式则会在请求体中提出
        - 包含
          - 请求方式
            - GET
              - 在浏览器中输入URL
              - 超链接
              - form不指定method默认为GET方式
              - 请求格式：URI?name=value&name=value，在请求行上发送数据
              - 只能发送普通字符串，长度有限
              - 适合从服务器中获取数据，在这种情况下所以GET请求较为安全
              - 支持缓存，也就是通过GET请求得到的资源会缓存在浏览器中，类似于Cache
            - POST
              - 仅当使用form表单并method属性设置为POST
              - 发送的数据在请求体中，请求格式：name=value&name=value
              - 可发送任何类型，理论上无长度限制
              - 适合向服务器传送数据，在这种情况下所以POST请求比较危险
              - 不支持缓存
            - form表单中的input标签中的name属性就是以上两种方式的name
            - 大部分form表单提交使用POST
          - URI
            - 与URL的区别
              - URL：统一资源定位符，通过URL可以定位到一个固定的资源，URL包含URI
              - URI：统一资源标识符，代表某个资源的名字，但通过URI是无法直接找到资源
          - 协议版本号
    
      - 请求头
    
        ```xml
        Connection: keep-alive
        Cache-Control: max-age=0
        sec-ch-ua: " Not A;Brand";v="99", "Chromium";v="99", "Google Chrome";v="99"
        sec-ch-ua-mobile: ?0
        sec-ch-ua-platform: "Windows"
        Upgrade-Insecure-Requests: 1
        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36
        Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
        Sec-Fetch-Site: none
        Sec-Fetch-Mode: navigate
        Sec-Fetch-User: ?1
        Sec-Fetch-Dest: document
        Accept-Encoding: gzip, deflate, br
        Accept-Language: zh-CN,zh;q=0.9
        Cookie: Idea-d8402bb6=c67e72d8-6b9e-4a87-bdbc-efb00293fec9
        If-None-Match: W/"52-1648464044000"
        If-Modified-Since: Mon, 28 Mar 2022 10:40:44 GMT
        ```
    
        - 包含请求主机
        - 主机端口
        - 浏览器信息
        - 平台信息
        - Cookie信息
    
      - 空白行
    
      - 请求体
    
    - 响应协议
    
      - 状态行
    
        ```xml
        HTTP/1.1 304
        ```
    
        - 协议版本号：HTTP/1.1
        - 状态码：304https://www.runoob.com/http/http-status-codes.html
          - 常用状态码
            - 404：访问资源不存在，路径写错了或者服务器资源未启动成功，404为前端错误
            - 405：前端请求方式和后端处理方式不同，比如前端使用GET而后端使用POST方式
            - 200：访问成功
            - 500：后端错误
    
      - 响应头
    
        ```xml
        
        ETag: W/"52-1648464044000"
        Date: Wed, 30 Mar 2022 12:37:47 GMT
        Keep-Alive: timeout=20
        Connection: keep-alive
        ```
    
        - 包含响应内容类型，长度，时间
    
      - 空白行
    
      - 响应体
    
        ```xml
        <html>
        <body>
        <h2>Hello World!</h2>
        </body>
        </html>
        ```
    



- ### HttpServlet

  - http包下的类

    - HttpServlet
    - httpServletRequest
      - 封装了http请求协议
    - httpServletResponse
      - 封装了http响应协议

  - HttpServlet生命周期

    - 无参构造

    - init方法：

      - 调用GenericServlet中的init方法（有参），有参init方法为ServletConfig赋值后执行Servlet接口中的init方法

    - service方法

      - service方法有两个，一个是GenericServlet继承得来的也是servlet原生的，一个是httpservlet自己的

        - GenericServlet继承的service原码，首先调用这个方法

          ```java
          public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
                  if (req instanceof HttpServletRequest && res instanceof HttpServletResponse) {
                      HttpServletRequest request = (HttpServletRequest)req;
                      HttpServletResponse response = (HttpServletResponse)res;
                      this.service(request, response);
                  } else {
                      throw new ServletException("non-HTTP request or response");
                  }
              }
          ```

          - 对参数进行强制下转型，后调用httpservlet自带的service方法

        - httpservlet自带的service方法

          ```java
          protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                  String method = req.getMethod();
                  long lastModified;
                  if (method.equals("GET")) {
                      lastModified = this.getLastModified(req);
                      if (lastModified == -1L) {
                          this.doGet(req, resp);
                      } else {
                          long ifModifiedSince = req.getDateHeader("If-Modified-Since");
                          if (ifModifiedSince < lastModified) {
                              this.maybeSetLastModified(resp, lastModified);
                              this.doGet(req, resp);
                          } else {
                              resp.setStatus(304);
                          }
                      }
                  } else if (method.equals("HEAD")) {
                      lastModified = this.getLastModified(req);
                      this.maybeSetLastModified(resp, lastModified);
                      this.doHead(req, resp);
                  } else if (method.equals("POST")) {
                      this.doPost(req, resp);
                  } else if (method.equals("PUT")) {
                      this.doPut(req, resp);
                  } else if (method.equals("DELETE")) {
                      this.doDelete(req, resp);
                  } else if (method.equals("OPTIONS")) {
                      this.doOptions(req, resp);
                  } else if (method.equals("TRACE")) {
                      this.doTrace(req, resp);
                  } else {
                      String errMsg = lStrings.getString("http.method_not_implemented");
                      Object[] errArgs = new Object[]{method};
                      errMsg = MessageFormat.format(errMsg, errArgs);
                      resp.sendError(501, errMsg);
                  }
          
              }
          ```

          - 获取请求方式后根据请求方式调用不同的方法，如GET请求方式调用doGet（方法）

        - doGET方法

          ```java
          protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                  String protocol = req.getProtocol();
                  String msg = lStrings.getString("http.method_get_not_supported");
                  if (protocol.endsWith("1.1")) {
                      resp.sendError(405, msg);
                  } else {
                      resp.sendError(400, msg);
                  }
          
              }
          ```

          - httpservlet中doget方法原型是发送405错误（dopost也一样），也就是说必须重写httpservlet中的doget方法和dopost方法（如果有get和post请求发出时），同时这样也可以限制网页的请求方式，如果需要重写doget和dopost方法可以直接service方法（继承父类的）

        - 欢迎界面的配置

          ```xml
          <welcome-file-list>
              <welcome-file>fuckyou.html</welcome-file>
          </welcome-file-list>
          ```

          - welcome-file中默认从webapp文件夹下开始查找

          - 这个标签也是为什么index开头的文件总是默认打开的文件的原因，因为在Tomcat中的web.xml文件中设置了

            ```xml
             <welcome-file-list>
                    <welcome-file>index.html</welcome-file>
                    <welcome-file>index.htm</welcome-file>
                    <welcome-file>index.jsp</welcome-file>
            </welcome-file-list>
            ```

            - 在浏览器启动时会默认先查找项目的web.xml文件中是否有 <welcome-file-list>标签，如果有找到第一个 <welcome-file>（也就是从上至下优先级减少），如果项目web.xml文件中没有设置则会在Tomcat中的xml文件中查找。

        - 关于WEB-INF文件夹

          - WEB-INF文件夹下的文件是受保护的，无法通过url访问，所以html等文件需要设置在该文件夹外部

  - ####HttpServletRequest接口

    - HttpServletRequest的实现类
    
      - HttpServletRequest是ServletRequest的子接口，并未实现ServletRequest的方法，真正实现HttpServletRequest接口的是Tomcat服务器中的某个类（并不重要，可以通过HttpServletRequest的toString方法得知）。
      
      - HttpServletRequest中包含了http请求协议的数据和内容，由web容器解析http请求协议中的内容然后封装给HttpServletRequest，程序员通过service方法获得这些数据
      
      - Request和Response生命周期
        - 一次请求和一次响应分别对应一个Request和Response，请求结束和响应结束便是Request和Response的结束
    
      - 获取用户提交的数据方法
      
          - | `String`               | `getParameter(String name)`  通过名称获取指定数据， 如果参数不存在，则返回请求参数的值 `String`或 `null` 。 |
             | ---------------------- | ------------------------------------------------------------ |
             | `Map<String,String[]>` | `getParameterMap()`   返回此请求的参数的java.util.Map。      |
             | `Enumeration<String>`  | `getParameterNames()`  返回所有参数的名称                    |
             | `String[]`             | `getParameterValues(String name)`  如果一个参数有多个数据则可以使用这个方法 |
             
             - form表单提交的数据采用map集合方式存放
          
      - Servlet资源跳转方法
      
          -  | `RequestDispatcher` | `getRequestDispatcher(String path)`    返回一个 [`RequestDispatcher`](../../javax/servlet/RequestDispatcher.html)对象，该对象充当位于给定路径的资源的包装器。 |
              | ------------------- | ------------------------------------------------------------ |
          
          - 例
          
            ```java
            RequestDispatcher dispatcher = request.getRequestDispatcher("/hello");
            //获取资源转发器
            dispatcher.forward(request,response);
            //调用资源转发器forward方法实现跳转
            ```
          
            - /hello表示跳转的servlet对象访问路径，底层实现为将指定路径的servlet对象包装到转发器中（由Tomcat实现），再将请求和响应参数传入转发器实现转发，这种方法一般用于Servlet之间的信息传输，除此以外html也可以作为接受方的终点。
          
      - 获取客户端IP地址
      
          -  | `String` | `getRemoteAddr()`   返回发送请求的客户端或最后一个代理的Internet协议（IP）地址。 |
              | -------- | ------------------------------------------------------------ |
          
      - 设置请求体的编码
      
          -  | `void` | `setCharacterEncoding(String env)`  Overrides the name of the character  encoding used in the body of this request.  译: 覆盖此请求正文中使用的字符编码的名称。 |
              | ------ | ------------------------------------------------------------ |
              
              - 仅限于Post请求，get请求方法通过URI提交数据，在Tomcat9之前URI默认编码为ISO-8858-1，Tomcat9之后为UTF-8。
          
      - 获取项目根目录
      
          -  | `String` | `getContextPath()`返回请求URI的一部分，指示请求的上下文。 |
              | -------- | --------------------------------------------------------- |
          
      - 获取请求的URI
      
          -  | `String` | `getRequestURI()`  将此请求的URL部分从协议名称返回到HTTP请求第一行中的查询字符串。 |
              | -------- | ------------------------------------------------------------ |
          
      - 获取Servlet路径
      
           - | `String` | `getServletPath()`  Returns the part of this request's URL that  calls the servlet.  译: 返回此请求调用servlet的URL的一部分。 |
              | -------- | ------------------------------------------------------------ |
              
              - 三种获取路径比较
              
                ```java
                response.setContentType("text/html;charset=utf-8");
                PrintWriter writer = response.getWriter();
                writer.println("项目路径： "+request.getContextPath()+"<br>");
                writer.println("URI: "+request.getRequestURI()+"<br>");
                writer.println("Servlet请求路径："+request.getServletPath()+"<br>");
                
                输出
                项目路径： /untitled28_war
                URI: /untitled28_war/a
                Servlet请求路径：/a
                ```
           
      - 关于响应中文乱码问题
      
           - ```java
             response.setContentType("text/html;charset=utf-8");
             ```
      
             - 必须在获取流前执行
           
      - Servlet重定向
      
           -  | `void` | `sendRedirect(String location)`   使用指定的重定向位置URL向客户端发送临时重定向响应并清除缓冲区。 |
               | ------ | ------------------------------------------------------------ |
               
               - 使用效果和资源转发类似，但在路径上需提供项目路径
               
                 ```java
                 response.sendRedirect("/untitled28_war/hello");
                 ```
               
               - 两者的区别：
               
                 - 重定向会改变URL，而资源转发不会
               
                 - 重定向由浏览器完成，转发资源由Tomcat服务器完成
               
                 - 转发刷新页面可能会多次提交表单
               
                 - 使用场景
               
                   - 资源共享方面使用转发
                   - 其余情况使用重定向（重定向同样可以重定向到一个html页面）
               
                 - 转发资源（一次请求）
               
                   ![image-20220405112146398](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220405112146398.png)
               
                 - 重定向(两次请求)
               
                   ![image-20220405112331154](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220405112331154.png)
    
    -----------------------
    
  - ## Servlet的注解开发
  
    - 使用场景
  
      - 对于不经常改变的配置可以通过注解标注，xml文件依旧可以使用
  
      - 开发中常使用注解+xml文件
    - 常用注解
    
      - @WebServlet
    
        - 属性
    
          - String name：类似于<servlet-name>
          - String[] urlPatterns:类似于url-pattern标签，String数组类型，也就是一个servlet对应多个url
          - String[] value:与urlPatterns相同，只不过value在注解中初始化时可以省略名称。
          - int loadOnStartup： 类似用 <load-on-startup></load-on-startup>，使Tomcat在启动时加载Servlet（正常情况下Tomcat会在需要响应Servlet时初始化）
          - WebInitParam[] initParams():初始化参数
            - @WebInitParam
              - String name
              - String  value
              - String  description
    
    ------------------
    
  - ### Session机制（会话机制）
  
    - 会话定义
  
      - 用户打开浏览器到关闭浏览器的过程（同一个网页或者说项目）
    
      - 作用
        - 保存会话状态
          - 为什么要保存会话状态
            - http协议是无状态协议，当请求结束后会自动与服务器断开连接
        
      - Session生命周期
        - 诞生：浏览器打开，web容器分配一个新的Session对象
        
        - 终结：浏览器关闭，web容器销毁Session
        
        - 相比于Request和context，Request的生命太短，context的生命周期从服务器启动开始到服务器关闭结束，采用context过于浪费。
        
        - 底层原理
        
          - 当浏览器第一次发送请求会获得一个Session，浏览器为这个Session编上号，并在之后的请求之前都会先寻找这个Session，而服务器在浏览器关闭前都不会销毁这个Session。
        
            - 浏览器会存储一个以Cookie的方式的SessionID
        
          - 当浏览器关闭时Session经过一段时间后被销毁
        
            - 这个的实现是通过Session超时机制，当某段时间内Session未被使用时，Tomcat会销毁该Session对象，采取这个机制也是因为http为无状态协议，http无法告诉Tomcat浏览器是否关闭，该功能可以在xml文件中设置超时时间
        
              ```xml
              
              <session-config>
                      <session-timeout>30</session-timeout>
                  </session-config>
              ```
        
              - 30分钟内无访问则会销毁该session，30min也是默认的
              - 这种机制会产生两个现象
                - 浏览器关了，Session短时间内还未被销毁
                - 浏览器没关，Session销毁了（无用户请求）
        
          - 浏览器关闭时本地浏览器关于session的编号也会被销毁
        
          - 关于session对象存储在底层为map集合，这个集合称为session列表
        
      - Cookie禁用
    
        - 可以在浏览器中设置不接受任何Cookie，每次访问服务器都会发送一个Cookie作为SessionID给浏览器，但浏览器拒收，则每次浏览器访问都会产生一个新的Session且在Session超时销毁前之前的Session都会一直存在。
    
        - Cookie后是否真的无法采用Session机制？
    
          - 可以通过URL重写机制
    
            - （1）记录第一次的Jsessionid：6CA1D354538046BDD4177A125C85F161
    
              ![image-20220405210351066](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220405210351066.png)
    
            - （2）在url上重写为：原先的URL+：+jsessionid=6CA1D354538046BDD4177A125C85F161
    
            - 这样也可以实现Session机制，只不过开发成本较高
    
    - 获取Session对象
    
      -  | `HttpSession` | `getSession()`    返回与此请求关联的当前会话，或者如果请求没有会话，则创建一个会话。 |
          | ------------- | ------------------------------------------------------------ |
      
    - HttpSession
    
      - 常用方法
    
        - 对数据操作
    
          -  | `Object`              | `getAttribute(String name)`   返回在此会话中使用指定名称绑定的对象，如果名称下没有绑定对象，则 `null` 。 |
              | --------------------- | ------------------------------------------------------------ |
              | `Enumeration<String>` | `getAttributeNames()`     返回 `Enumeration`个  `String`对象，其中包含绑定到此会话的所有对象的名称。 |
              | `void`                | `setAttribute(String name,  Object value)` 添加值            |
              | `void`                | `removeAttribute(String name)`   删除值                      |
              | `HttpSession`         | `getSession(boolean create)`  返回与此请求关联的当前 `HttpSession` ，如果没有当前会话且 `create`为true，则返回新会话。  如果为false且没有当前会话返回null |
    
    -------------------
    
    ## Cookie
    
    - Session的保存方式就是通过Cookie保存的
    
      - | JSESSIONID | BCCB87F86A18475FBD4D86B64A841545 |
        | ---------- | -------------------------------- |
    
        - 这种Cookie保存在浏览器的运行内存中，只要浏览器不关闭，这类资源就会保存
    
    - Cookie保存位置
    
      - 保存在浏览器客户端上
        - 运行内存中（浏览器关闭消失）
        - 硬盘文件中（永久保存）
    
    - Cookie作用
    
      - http协议无状态协议，Cookie和session作用差不多，但session保存在服务器上
      - 例：
        - bilibili在第一次登录后浏览器将登录的信息保存在浏览器的内存中也就是硬盘中，下一次浏览器访问时会直接寻找本地的登录信息，京东的购物车同理，而这种操作Session做不到
        - 邮箱10天内免登录
      - Cookie在http中规定：任何一个Cookie都是由name和value组成
    
    - Cookie类
    
      - 构造方法
    
        | `Cookie(String name,  String value)` 构造具有指定名称和值的cookie。 |
        | ------------------------------------------------------------ |
    
      - 底层原理
    
        - 1）服务器创建cookie对象，把会话数据存储到cookie对象中。
    
            Cookie   cookie  new Cookie("name","value");
    
        - 2）  服务器发送cookie信息到浏览器
    
                response.addCookie(cookie);
    
        - 3）浏览器得到服务器发送的cookie，然后保存在浏览器端。
    
        - 4）浏览器在下次访问服务器时，会带着cookie信息，通过url携带
    
               举例： cookie: name=eric (隐藏带着一个叫cookie名称的请求头)
    
        - 5）服务器接收到浏览器带来的cookie信息
    
               request.getCookies();
    
      - 有关方法
    
        - Cookie路径设置和获取
    
           - | `void`   | `setPath(String uri)`   指定客户端应返回cookie的cookie的路径。 |
              | -------- | ------------------------------------------------------------ |
              | `String` | `getPath()`   返回浏览器返回此cookie的服务器上的路径         |
              
              - 如果不设置路径则默认为项目根路径（http://localhost:8080/untitled29_war/），如需设置路径需要添加项目根路径
           
        - Cookie保存时间设置
        
           -  | `int` | `getMaxAge()`   获取此Cookie的最大年龄（以秒为单位）。 |
               | ----- | ------------------------------------------------------ |
               | `int` | `getMaxAge()`  获取此Cookie的最大年龄（以秒为单位）。  |
               
               - 若不设置有效时间，则Cookie保存在浏览器运行内存中
               - 若设置有效时间为0则表示该Cookie被删除，主要应用于删除浏览器上同名的Cookie
               - 如果设置为负数则表示该Cookie不会被存储在硬盘中和未设置效果相同
           
        - Cookie获取value和设置
        
           -  | `void`   | `setValue(String newValue)`  为此Cookie指定一个新值。 |
               | -------- | ----------------------------------------------------- |
               | `String` | `getValue()` 获取此Cookie的当前值。                   |
           
        - 通过Request获取Cookie
        
           - | `Cookie[]` | `getCookies()`    返回一个数组，其中包含客户端使用此请求发送的所有 `Cookie`对象。 |
               | ---------- | ------------------------------------------------------------ |
           
               - 如果没有Cookie则返回null
    
    
    
    -----------------------------
    
  - ## Filter过滤器接口
  
    - 作用提高代码复用
  
    - Filter需要实现的方法
  
      - ```java
        void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
        ```
        
        - 用户发送一次请求则调用一次
  
    - Filter方法
  
      - init方法
    
      ```java
      void init(FilterConfig filterConfig)
      ```
  
      - destroy方法
    
       ```java
        default void destroy() 
       ```
      
      - 以上两种方法和servlet中的init和destroy方法执行基本类似
      
    - Filter的使用
    
      - （1）编写一个类实现Filter接口
    
      - （2）在xml文件中配置filter属性或者通过注解配置
    
        - xml文件
    
          ```xml
          <filter>
              <filter-name>filter1</filter-name>
              <filter-class>filter</filter-class>
          </filter>
          
          <filter-mapping>
              <filter-name>filter1</filter-name>
              <url-pattern>/*</url-pattern>
          </filter-mapping>
          ```
    
        - 注解方式
    
          ```java
          @WebFilter(urlPatterns = "/*", filterName="filter2")
          ```
    
    - Filter生命周期
    
      - 在服务器启动时创建，且为单例
      - 启动时先执行无参构造后执行init方法
      - 服务器关闭时先执行destroy方法后销毁
    
    - Filter的doFilter方法
    
      - 可以吧Filter视为一个特殊的Servlet，doFilter方法为该servlet的service方法
    
      - 执行doFilter方法后通过参数 filterChain执行下一个过滤器或者是Servlet
    
        ```java
        System.out.println("doFilter");
        filterChain.doFilter(servletRequest,servletResponse);
        System.out.println("doFilter over");
        ```
    
        - filterChain.doFilter(servletRequest,servletResponse)会跳转到访问该filter的路径相匹配的下一个Filter或Servlet，在此之后又会重新回到doFilter方法中并向下执行，比如以上代码的输出doFilter over
    
    - Filter的执行顺序
    
      - 如果通过注解配置
    
      - 按照Filter名称的字典顺序执行
    
        - 如
    
          - Filter1优先执行于Filter2
      - FilterA优先执行于FilterB
      - 如果通过xml文件配置
        - Filter-mapping越靠上优先级越高
        - 这种优先级设置方便修改filter执行顺序，所以开发中常采用这种方式
      
    - Filter中的设计模式
    
      - 责任链设计模式。
        - 过滤器最大的优点：
          - 在程序编译阶段不会确定调用顺序。因为Filter的调用顺序是配置到web.xml文件中的，只要修改web.xml配置文件中filter-mapping的顺序就可以调整Filter的执行顺序。显然Filter的执行顺序是在程序运行阶段动态组合的。那么这种设计模式被称为责任链设计模式。
        - 责任链设计模式最大的核心思想：
          - 在程序运行阶段，动态的组合程序的调用顺序。

------------------------------------------

- ### Listener监听器

  - 作用

    - 在特殊时机执行特殊代码

  - Servlet提供的Listener监听器接口

    - ServletContextListener
    - ServletContextAttributeListener
    - ServletRequestListener
    - ServletRequestAttributrListener
    - HttpSessionListener
    - HttpSessionAttributeListener
    - HttpSessionBindingListener
    - HttpSessionIdListener
      - 用于监听Sessionid的变化
    - HttpSessionActicationListener
      - 用于监听Session的钝化和活化
      - 钝化：Session对象从浏览器运行内存到硬盘中
      - 活化：Session对象从硬盘到浏览器运行内存中

  - Listener使用（以ServletContextListener为例）

    - 编写一个类并实现ServletContextListener接口
  
      - 实现方法
  
         - | `void` | `contextDestroyed(ServletContextEvent sce)`   收到ServletContext即将关闭的通知。 |
            | ------ | ------------------------------------------------------------ |
            | `void` | `contextInitialized(ServletContextEvent sce)`  接收Web应用程序初始化过程正在启动的通知。 |
      
    - 在xml文件中配置Listener或者使用注解配置
    
      - 该步骤与Servlet基本相同
    
      - xml文件方式
    
        ```java
        <listener>
          <listener-class>My_Listener</listener-class>
        </listener>
        ```
    
      - 注解方式
    
        ```java
        @WebListener
        ```
    
    - Listener中的方法不需要java程序员调用，当程序中发生某个特殊事件时服务器自动调用
    
      - 调用的方法和时刻
        - contextDestroyed(ServletContextEvent sce)
          - 当ServletContext创建时调用
        - contextInitialized(ServletContextEvent sce)
          - 当ServletContext被销毁的时候调用
      - 其他的监听器效果同理，比如ServletContextListener是关于ServletContext的创建于销毁时需要的操作，ServletRequestListener是关于request对象创建于销毁时需要的操作，HttpSessionListener是关于Session创建与销毁时进行操作
    
  - 参数有关的Listener（以ServletContextAttributeListener为例）
  
    - ServletContextAttributeListener重写的方法
  
      -  | `void` | `attributeAdded(ServletContextAttributeEvent event)`  接收已将属性添加到ServletContext的通知。 |
          | ------ | ------------------------------------------------------------ |
          | `void` | `attributeRemoved(ServletContextAttributeEvent event)`   收到已从ServletContext中删除属性的通知。 |
          | `void` | `attributeReplaced(ServletContextAttributeEvent event)`      |
          
      - 当Context中添加属性和移除属性时分别调用attributeAdded和attributeRemoved方法,覆盖值时调用attributeReplaced方法
      
          - 例
      
            ```java
            ServletContext context = this.getServletContext();
            context.setAttribute("ttt","aaa");
            context.setAttribute("ttt","kkk");
            context.removeAttribute("ttt");
            ```
      
            ```html
            输出：
            add
            Replaced
            remove
            ```
  
  - HttpSessionBindingListener
  
    - 类似于特殊的HttpSessionAttributeListener
  
      - 对于只有实现了HttpSessionBindingListener的类才会触发特殊事件
  
      - 例
  
        - 一个普通的类但是实现了HttpSessionBindingListener接口（并没有注册监听）
  
          ```java
          public class good_class implements HttpSessionBindingListener {
              @Override
              public void valueBound(HttpSessionBindingEvent event) {
                  System.out.println("绑定数据");
              }
          
              @Override
              public void valueUnbound(HttpSessionBindingEvent event) {
                  System.out.println("解绑数据");
              }
          
              String gg;
          
              public good_class(String gg) {
                  this.gg = gg;
              }
          }
          ```
  
        - Servlet
  
          ```java
          public class contextListener extends HttpServlet {
              @Override
              protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                  HttpSession session = req.getSession();
                  session.setAttribute("good_class",new good_class("gg"));
                  session.removeAttribute("good_class");
              }
          }
          ```
  
        - 控制台输出
  
          ```html
          绑定数据
          解绑数据
          ```
  
      - 对于某些特殊的类可以实现这个接口，当将这个类绑定到Session上时会触发监听器
  
      - 这种监听器可以实现实时查看网站登录人数
  
  - 

