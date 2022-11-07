# SpringMVC容器（二）

 DispatcherServlet的创建

```java
 DispatcherServlet dispatcherServlet = new DispatcherServlet(
                    this.createApplicationContext(context.getServletContext()));

public DispatcherServlet(WebApplicationContext webApplicationContext) {
		super(webApplicationContext);//设置Application容器
		setDispatchOptionsRequest(true);//设置是否可以让servlet接受option类型请求
	}
```

向Tomcat中注册DispatcherServlet

```java
 Wrapper servlet = Tomcat.addServlet(context, "dispatcherServlet", dispatcherServlet);
```

内置Tomcat的创建和启动

- 创建Tomcat对象

  ```java
  Tomcat tomcat = new Tomcat();//创建tomcat对象
  tomcat.setPort(8088); //设置端口（成员变量）
  tomcat.getConnector();//
  //创建web容器上下文
  Context context = tomcat.addContext("", null);
  ```

  - 静态代码块(读取conf/logging.properties文件，用于日志的设置)

    ```java
    static {
        // Graal native images don't load any configuration except the VM default
        if (JreCompat.isGraalAvailable()) {
            try (InputStream is = new FileInputStream(new File(System.getProperty("java.util.logging.config.file", "conf/logging.properties")))) {
                LogManager.getLogManager().readConfiguration(is);
            } catch (SecurityException | IOException e) {
                // Ignore, the VM default will be used
            }
        }
    }
    ```

- addContext方法，添加上下文编程模式，不使用默认的web.xml，

  ```java
  //host—将部署上下文的主机。contextPath—要使用的上下文映射，“”用于根上下文。contextName—上下文名称dir—用于静态文件的上下文的基本目录。必须存在，相对于服务器主页
  
  public Context addContext(Host host, String contextPath, String contextName,
                            String dir) {
      silence(host, contextName);//设置日志
      Context ctx = createContext(host, contextPath);//创建上下文
      ctx.setName(contextName);//设置上下文名称
      ctx.setPath(contextPath);//设置上下文映射路径
      ctx.setDocBase(dir);//设置静态资源基本目录
      ctx.addLifecycleListener(new FixContextListener());//创建监听器
  
      if (host == null) {
          getHost().addChild(ctx);
      } else {
          host.addChild(ctx);//Host类继承了Container接口，Container接口类似与分层的容器。
      }
      return ctx;
  }
  
  ```

  Tomcat添加DispatchServlet

  ```java
  public static Wrapper addServlet(Context ctx,
                                   String servletName,
                                   Servlet servlet) {
      // will do class for name and set init params
      Wrapper sw = new ExistingStandardWrapper(servlet);//使用容器包装Servlet
      sw.setName(servletName);
      ctx.addChild(sw);//Context将Servlet添加为子容器
  
      return sw;
  }
  ```

  Tomcat启动(start方法)

  ```java
  public void start() throws LifecycleException {
          getServer();
          server.start();
      }
  ```

  - getServer

    ```java
    public Server getServer() {
    
        if (server != null) {
            return server;
        }
    
        System.setProperty("catalina.useNaming", "false");
    
        server = new StandardServer();
    
        initBaseDir();
    
        // Set configuration source
        ConfigFileLoader.setSource(new CatalinaBaseConfigurationSource(new File(basedir), null));
    
        server.setPort( -1 );
    
        Service service = new StandardService();
        service.setName("Tomcat");
        server.addService(service);
        return server;
    }
    ```

  - server的start方法

    ```java
    public final synchronized void start() throws LifecycleException {
    
        if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||
            LifecycleState.STARTED.equals(state)) {//当前阶段为准备阶段则打印日志并返回
    
            if (log.isDebugEnabled()) {
                Exception e = new LifecycleException();
                log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
            } else if (log.isInfoEnabled()) {
                log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
            }
    
            return;
        }
    
        if (state.equals(LifecycleState.NEW)) {//新建阶段	
            init();
        } else if (state.equals(LifecycleState.FAILED)) {
            stop();
        } else if (!state.equals(LifecycleState.INITIALIZED) &&
                   !state.equals(LifecycleState.STOPPED)) {
            invalidTransition(Lifecycle.BEFORE_START_EVENT);
        }
    
        try {
            setStateInternal(LifecycleState.STARTING_PREP, null, false);
            startInternal();
            if (state.equals(LifecycleState.FAILED)) {
                // This is a 'controlled' failure. The component put itself into the
                // FAILED state so call stop() to complete the clean-up.
                stop();
            } else if (!state.equals(LifecycleState.STARTING)) {
                // Shouldn't be necessary but acts as a check that sub-classes are
                // doing what they are supposed to.
                invalidTransition(Lifecycle.AFTER_START_EVENT);
            } else {
                setStateInternal(LifecycleState.STARTED, null, false);
            }
        } catch (Throwable t) {
            // This is an 'uncontrolled' failure so put the component into the
            // FAILED state and throw an exception.
            handleSubClassException(t, "lifecycleBase.startFail", toString());
        }
    }
    ```

    - init方法

      ```java
      public final synchronized void init() throws LifecycleException {
              if (!state.equals(LifecycleState.NEW)) {
                  invalidTransition(Lifecycle.BEFORE_INIT_EVENT);//不是new阶段直接抛出异常
              }
      
              try {
                  setStateInternal(LifecycleState.INITIALIZING, null, false);//阶段设置（初始化中）
                  initInternal();//初始化
                  setStateInternal(LifecycleState.INITIALIZED, null, false);//初始化结束
              } catch (Throwable t) {
                  handleSubClassException(t, "lifecycleBase.initFail", toString());
              }
          }
      ```

      - initInternal方法

        ```java
        ```

        