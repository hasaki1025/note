# SpringMVC源码（一）

## （一）内置Tomcat配置与DispatchServlet的初始化

- 内置Tomcat创建

  ```java
  public MainServer(){
      Tomcat tomcat = new Tomcat();//创建tomcat对象
      tomcat.setPort(8088); //设置端口
      tomcat.getConnector();
  
      //创建web容器上下文
      Context context = tomcat.addContext("", null);
      try {
          //注册前端控制器
          DispatcherServlet dispatcherServlet = new DispatcherServlet(
              this.createApplicationContext(context.getServletContext()));
          Wrapper servlet = tomcat.addServlet(context, "dispatcherServlet", dispatcherServlet);
          servlet.setLoadOnStartup(1);
          servlet.addMapping("/*");
  
          tomcat.start();
      } catch (Exception e) {
          e.printStackTrace();
      }
  }
  ```

  - Spring容器创建

    ```java
    public WebApplicationContext createApplicationContext(
        ServletContext servletContext) {
        AnnotationConfigWebApplicationContext ctx
            = new AnnotationConfigWebApplicationContext();
        ctx.register(WebApplicationConfig.class); //加载配置类
    
        ctx.setServletContext(servletContext);
        ctx.refresh();//就是调用AbstractApplicationContext中的refresh方法
        ctx.registerShutdownHook();
        return ctx;
    }
    ```

    - AnnotationConfigWebApplicationContext注册

      ```java
      public void register(Class<?>... componentClasses) {
          Assert.notEmpty(componentClasses, "At least one component class must be specified");
          Collections.addAll(this.componentClasses, componentClasses);//直接添加到组件集合中
      }
      ```

    - 设置ServletContext（单纯设置为成员变量）

    - 刷新（调用父类AbstractApplicationContext中刷新方法，和普通的Spring容器结构相同，但是具体实现不同）

      - 创建后置处理器

      ```java
      // 允许在上下文子类中对 bean 工厂进行后处理。（暂时这里什么都没有,但是如果是Web类的Context会设置后置处理器）
      postProcessBeanFactory(beanFactory);
      ```

      ```java
      //来自AbstractRefreshableWebApplicationContext，也就是Web容器的父类
      
      protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
          beanFactory.addBeanPostProcessor(new ServletContextAwareProcessor(this.servletContext, this.servletConfig));//添加ServletAware后置处理器，该处理器用于帮助初始化Servlet，
          beanFactory.ignoreDependencyInterface(ServletContextAware.class);
          beanFactory.ignoreDependencyInterface(ServletConfigAware.class);
      
          WebApplicationContextUtils.registerWebApplicationScopes(beanFactory, this.servletContext);
          WebApplicationContextUtils.registerEnvironmentBeans(beanFactory, this.servletContext, this.servletConfig);
      }
      ```

      - ServletContextAwareProcessor的构造方法包装了成员变量ServletContext和ServletConfig（这里暂时为null），该类实现了BeanPostProcessor,postProcessBeforeInitialization方法通过内部的ServletContext和ServletConfig创建ServletContextAware并设置属性，postProcessAfterInitialization方法直接返回Bean

      - registerWebApplicationScopes方法

        ```java
        public static void registerWebApplicationScopes(ConfigurableListableBeanFactory beanFactory,
        			@Nullable ServletContext sc) {
        
        		beanFactory.registerScope(WebApplicationContext.SCOPE_REQUEST, new RequestScope());//注册request生命周期(放入BeanFactory的Map集合中)
        		beanFactory.registerScope(WebApplicationContext.SCOPE_SESSION, new SessionScope());//注册Session生命周期
        		if (sc != null) {
        			ServletContextScope appScope = new ServletContextScope(sc);
        			beanFactory.registerScope(WebApplicationContext.SCOPE_APPLICATION, appScope);//如果ServletContext不为空则注册application作用域
        			// Register as ServletContext attribute, for ContextCleanupListener to detect it.
        			sc.setAttribute(ServletContextScope.class.getName(), appScope);//并将作用域的值放入ServletContext中
        		}
        
        		beanFactory.registerResolvableDependency(ServletRequest.class, new RequestObjectFactory());//注册接口实现类和实例对象
        		beanFactory.registerResolvableDependency(ServletResponse.class, new ResponseObjectFactory());
        		beanFactory.registerResolvableDependency(HttpSession.class, new SessionObjectFactory());
        		beanFactory.registerResolvableDependency(WebRequest.class, new WebRequestObjectFactory());
        		if (jsfPresent) {
        			FacesDependencyRegistrar.registerFacesDependencies(beanFactory);
        		}
        	}
        ```

        - registerResolvableDependency方法（来自DefaultListableBeanFactory）

          ```java
          @Override
          public void registerResolvableDependency(Class<?> dependencyType, @Nullable Object autowiredValue) {
              Assert.notNull(dependencyType, "Dependency type must not be null");
              if (autowiredValue != null) {
                  if (!(autowiredValue instanceof ObjectFactory || dependencyType.isInstance(autowiredValue))) {
                      throw new IllegalArgumentException("Value [" + autowiredValue +
                                                         "] does not implement specified dependency type [" + dependencyType.getName() + "]");
                  }//resolvableDependencies是Map<Class,Object>集合
                  this.resolvableDependencies.put(dependencyType, autowiredValue);
              }
          }
          ```

          - ConfigurableListableBeanFactory中提供了registerResolvableDependency方法，该方法提供了一个接口让用户将接口类型和实例直接关联，告诉spring指定接口注入时的需要的实例对象

      - registerEnvironmentBeans方法

        ```java
        public static void registerEnvironmentBeans(ConfigurableListableBeanFactory bf,
                                                    @Nullable ServletContext servletContext, @Nullable ServletConfig servletConfig) {
        
            if (servletContext != null && !bf.containsBean(WebApplicationContext.SERVLET_CONTEXT_BEAN_NAME)) {//如果不存在名称为servletContext的Bean则注册为单例
                bf.registerSingleton(WebApplicationContext.SERVLET_CONTEXT_BEAN_NAME, servletContext);//注册单例（直接实例化）
            }
        
            if (servletConfig != null && !bf.containsBean(ConfigurableWebApplicationContext.SERVLET_CONFIG_BEAN_NAME)) {
                bf.registerSingleton(ConfigurableWebApplicationContext.SERVLET_CONFIG_BEAN_NAME, servletConfig);
            }
        
            if (!bf.containsBean(WebApplicationContext.CONTEXT_PARAMETERS_BEAN_NAME)) {//名称为contextParameters的Bean
                Map<String, String> parameterMap = new HashMap<>();
                if (servletContext != null) {//servletContext参数包装
                    Enumeration<?> paramNameEnum = servletContext.getInitParameterNames();//获取ervletContext中的initParamer列表，这些属性配置作用域全局Servlet
                    while (paramNameEnum.hasMoreElements()) {//遍历参数名称并将参数列表放入parameterMap中
                        String paramName = (String) paramNameEnum.nextElement();
                        parameterMap.put(paramName, servletContext.getInitParameter(paramName));
                    }
                }
                if (servletConfig != null) {//ServletConfig参数包装
                    Enumeration<?> paramNameEnum = servletConfig.getInitParameterNames();//ServletConfig中的配置作用域单个Servlet
                    while (paramNameEnum.hasMoreElements()) {
                        String paramName = (String) paramNameEnum.nextElement();
                        parameterMap.put(paramName, servletConfig.getInitParameter(paramName));
                    }
                }
                bf.registerSingleton(WebApplicationContext.CONTEXT_PARAMETERS_BEAN_NAME,
                                     Collections.unmodifiableMap(parameterMap));//注册单例
            }
        
            if (!bf.containsBean(WebApplicationContext.CONTEXT_ATTRIBUTES_BEAN_NAME)) {//contextAttributes作为Bean加载到容器中
                Map<String, Object> attributeMap = new HashMap<>();
                if (servletContext != null) {
                    Enumeration<?> attrNameEnum = servletContext.getAttributeNames();//获取Attribute列表，attribute参数可以在后续请求中设置
                    while (attrNameEnum.hasMoreElements()) {
                        String attrName = (String) attrNameEnum.nextElement();
                        attributeMap.put(attrName, servletContext.getAttribute(attrName));
                    }
                }
                bf.registerSingleton(WebApplicationContext.CONTEXT_ATTRIBUTES_BEAN_NAME,
                                     Collections.unmodifiableMap(attributeMap));
            }
        }
        ```

        - 将ServletContext中一些参数包装为Bean注入到容器中，将ServletContext本身作为Bean注册到容器中
  
  - 

