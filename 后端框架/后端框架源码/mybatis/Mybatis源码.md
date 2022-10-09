# Mybatis源码

- ## 类加载器

  - 具体作用

    - 将class文件加载到jvm虚拟机中，但是jvm不会加载所有的class文件而是采用动态加载

  - 类别

    - Bootstrap ClassLoader

      - 加载核心类，%JRE_HOME%\lib，C语言实现

    - Extension ClassLoader

      - 加载可扩展的类加载器，%JRE_HOME%\lib\ext

    - App  ClassLoader

      - classpath下的所有类，比如MAVEN依赖下的

    - Custon ClassLoader

      - 自定义类加载器

    - 四种加载器自上而下依次尝试加载

    - 双亲委派机制https://blog.csdn.net/weixin_45962068/article/details/124087845

      - 执行以下代码

        ```java
        public class Test {
        
            public static void main(String[] args) {
                System.out.println(Test.class.getClassLoader());
                System.out.println(Test.class.getClassLoader().getParent());
                System.out.println(String.class.getClassLoader());
            }
        }
        
        /*
        sun.misc.Launcher$AppClassLoader@18b4aac2
        sun.misc.Launcher$ExtClassLoader@1b6d3586
        null
        */
        ```

      - String.class.getClassLoader()输出为空是由于Bootstrap ClassLoader底层由C实现

      - sun.misc.Launcher$AppClassLoader@18b4aac2 表示该ClassLoader是sun.misc.Launcher下的一个内部类

      - jdk 一般在rt.jar包下ClassLoader

- ## 源码阅读

  - ### 案例

    ```java
    //设置mybatis主配置文件路径
    String config="mybatis-config.xml";
    //读取mybatis主配置文件
    //InputStream in = Resources.getResourceAsStream(config);
    InputStream in = Thread.currentThread().getContextClassLoader().getResourceAsStream(config);
    //创建SqlSessionFactoryBuilder
    SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
    //通过SqlSessionFactoryBuilder指定主配置文件读取流创建SqlSessionFactory
    SqlSessionFactory factory=builder.build(in);
    //（重要）创建SQL执行对象
    SqlSession session = factory.openSession();
    //指定需要执行的sql语句的id，id=namespace+.+sql语句的id
    String sqlid="DAO.UserDao.selectAll";
    //获取查询结果
    List<User> list = session.selectList(sqlid);
    //输出查询结果
    list.forEach(System.out::println);
    //关闭sqlsession
    session.close();
    ```

  - ### Mybatis获取ClassLoader并读入配置文件

    ```java
    InputStream in = Resources.getResourceAsStream(config);//(在该代码打上断点)
    ```

    - 很容易可以了解到，Resources导入配置文件位置，通过ClassLoader中的静态方法加载文件(以下是jdk源码)

      ```java
      public InputStream getResourceAsStream(String name) {
              URL url = getResource(name);
              try {
                  return url != null ? url.openStream() : null;
              } catch (IOException e) {
                  return null;
              }
          }
      ```

    - 但是一开始ClassLoader为空，所以Mybatis先创建ClassLoaderWrapper，ClassLoaderWrapper调用ClassLoader（jdk自带）getSystemClassLoader获取到SystemClassLoader，之后再调用以下方法获取到所有环境下可能存在的ClassLoader

      ```java
       //不同环境的Classloader不同，这里将所有能考虑到的环境的ClassLoader全部获取
        ClassLoader[] getClassLoaders(ClassLoader classLoader) {
          return new ClassLoader[]{
              classLoader,
              defaultClassLoader,
              Thread.currentThread().getContextClassLoader(),//Web环境
              getClass().getClassLoader(),
              systemClassLoader};
        }
      ```

    - 之后Mybatis通过遍历上面得到的ClassLoader数组查找可用的ClassLoader并通过该ClassLoader的静态方法加载配置文件

    - #### 结论

      - 通过本次运行环境获取可用的ClassLoader以加载mybatis配置文件，所以该代码可以自己编写

        ```java
        InputStream in = Resources.getResourceAsStream(config);//可替代为以下代码
         InputStream in = Thread.currentThread().getContextClassLoader().getResourceAsStream(config);//只要获取到ClassLoader就行
        ```

  - ### Mybatis创建SqlSessionFactory

    - 为以下代码打上断点

      ```java
      //创建SqlSessionFactoryBuilder，不重要，只是单纯的创建对象
      SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
      //通过SqlSessionFactoryBuilder指定主配置文件读取流创建SqlSessionFactory
      SqlSessionFactory factory=builder.build(in);
      ```

      - 执行流程

        - builder.build(in)方法得到SqlSessionFactory

          ```java
          //新建XMLConfigBuilder对象（解析器），其中environment，properties的值都是空
          XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
          //解析XML文件
          return build(parser.parse());
          ```
        
          - XMLConfigBuilder的创建

            ```java
            //创建XPathParser并将其作为参数执行XMLConfigBuilder的构造方法
            this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);
            //其中XMLMapperEntityResolver是用于加载本地DTD文件的，也即是mybatis配置文件规范文件
            ```
        
            - XPathParser的创建

              ```java
              public XPathParser(InputStream inputStream, boolean validation, Properties variables, EntityResolver entityResolver) {
                  //为XPathParser中的一些成员变量赋值，其中validation为true，variables为空，entityResolver在上一步中创建了
                  commonConstructor(validation, variables, entityResolver);
                  this.document = createDocument(new InputSource(inputStream));
              }
              ```
        
              - commonConstructor方法
        
                ```java
                this.validation = validation;
                this.entityResolver = entityResolver;//EntityResolver赋值
                this.variables = variables;//Properties赋值
                XPathFactory factory = XPathFactory.newInstance();//执行XPathFactory类加载器，XPathFactory是jdk中的类，主要用于创建XPath类
                this.xpath = factory.newXPath();//通过XPathFactory创建XPath类
                ```
        
              - createDocument方法
        
                ```java
                  private Document createDocument(InputSource inputSource) {
                    // important: this must only be called AFTER common constructor
                    try {
                        //创建DocumentBuilderFactordy对象
                      DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
                     //此处省略许多为DocumentBuilderFactory赋值的操作
                      DocumentBuilder builder = factory.newDocumentBuilder();
                        //DocumentBuilder设置本地DTD解析类
                      builder.setEntityResolver(entityResolver);
                        //DocumentBuilder设置错误处理器
                      builder.setErrorHandler(new ErrorHandler() {
                        //省略
                      });
                        //将XML封装为一个Document并返回
                      return builder.parse(inputSource);
                    } catch (Exception e) {
                      throw new BuilderException("Error creating document instance.  Cause: " + e, e);
                    }
                  }
                ```
        
            - 执行XMLConfigBuilder构造方法
        
              ```java
              //调用父类方法初始化Configuration对象
              super(new Configuration());
              //配置当前线程的错误上下文，每一个线程有且仅有一个，通过ThreadLocal实现
              ErrorContext.instance().resource("SQL Mapper Configuration");
              this.configuration.setVariables(props);//props为空
              this.parsed = false;//设置XML文件为未解析
              this.environment = environment;//environment为空
              this.parser = parser;//将XPathParser赋值，XPathParser包括了Document对象
              ```
        
              - Configuration对象的创建
        
                ```java
                 public Configuration() {
                     //设置别名，比如在mybatis中JDBC代表JdbcTransactionFactory类
                     //typeAliasRegistry是Configuration的成员变量，本质是一个Map集合
                    typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
                //以下省略类似typeAliasRegistry.registerAlias的代码
                    languageRegistry.setDefaultDriverClass(XMLLanguageDriver.class);
                    languageRegistry.register(RawLanguageDriver.class);
                  }
                ```
        
                - Configuration对象是对mybatis配置文件中标签的封装，其中包括了环境等配置信息
        
                - registerAlias方法
        
                  ```java
                    public void registerAlias(String alias, Class<?> value) {
                      if (alias == null) {//检查该别名是否为空
                        throw new TypeException("The parameter alias cannot be null");
                      }
                      // issue #748
                      String key = alias.toLowerCase(Locale.ENGLISH);//将别名转化为小写
                      if (typeAliases.containsKey(key) && typeAliases.get(key) != null && !typeAliases.get(key).equals(value)) {//判断是否解析过
                        throw new TypeException("The alias '" + alias + "' is already mapped to the value '" + typeAliases.get(key).getName() + "'.");
                      }
                      typeAliases.put(key, value);//将该别名和对应的值放入typeAliases中
                    }
                  ```
        
          - build(parser.parse())执行XML文件解析
        
            ```java
            public Configuration parse() {
                if (parsed) {//检查是否解析过该XML文件，该值在上面已经赋值
                    throw new BuilderException("Each XMLConfigBuilder can only be used once.");
                }
                parsed = true;//设置该XML已解析过
                //开始解析,/configuration代表根标签
                parseConfiguration(parser.evalNode("/configuration"));
                //返回configuration对象
                return configuration;
            }
            ```
        
            - parser中含有Document和XPath对象，通过这两个对象对XML文件进行解析
        
              ```java
              public XNode evalNode(String expression) {
                  return evalNode(document, expression);
              }
              
              public XNode evalNode(Object root, String expression) {//Object root是Document对象
                  Node node = (Node) evaluate(expression, root, XPathConstants.NODE);//解析XML文件获取Node节点对象
                  if (node == null) {
                      return null;
                  }
                  return new XNode(this, node, variables);//封装节点并返回，XNode包括了解析器对象和node对象
              }
              ```
        
              - 假设mybatis配置文件是一个树，而configuration标签就是root节点，通过root节点可以获取到子节点也就是configuration中的其他标签，而Node对象就是用来描述configuration标签的类
        
              - evaluate方法
        
                ```java
                private Object evaluate(String expression, Object root, QName returnType) {//QName是命名空间
                    try {
                        return xpath.evaluate(expression, root, returnType);//调用xpath中的解析方法并返回Node对象
                    } catch (Exception e) {
                        throw new BuilderException("Error evaluating XPath.  Cause: " + e, e);
                    }
                }
                ```
        
            - XMLConfigBuild调用parseConfiguration方法读入标签信息并将信息存储到Configuration中
        
              ```java
              private void parseConfiguration(XNode root) {
                  try {
                      //issue #117 read properties first
                      propertiesElement(root.evalNode("properties"));//获取properties标签信息
                      Properties settings = settingsAsProperties(root.evalNode("settings"));//获取settings标签信息
                      environmentsElement(root.evalNode("environments"));//获取环境信息
                      mapperElement(root.evalNode("mappers"));//添加mapper
                      //省略类似代码
                  } catch (Exception e) {
                      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
                  }
              }
              ```
              
              - 以environmentsElement为例查看如何获取XML文件信息
              
                ```java
                private void environmentsElement(XNode context) throws Exception {
                    if (context != null) {
                      if (environment == null) {//检查环境设置是否未空，如果为空则设置为默认环境，有时我们会配置不同环境的配置信息，如生产环境、测试环境、开发环境
                        environment = context.getStringAttribute("default");
                      }
                      for (XNode child : context.getChildren()) {//依次遍历子节点
                        String id = child.getStringAttribute("id");
                        if (isSpecifiedEnvironment(id)) {
                            //初始化事务工厂
                          TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
                            //初始化数据源工厂
                          DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
                            //初始化数据源
                          DataSource dataSource = dsFactory.getDataSource();
                          Environment.Builder environmentBuilder = new Environment.Builder(id)
                              .transactionFactory(txFactory)
                              .dataSource(dataSource);
                            //为configuration设置环境
                          configuration.setEnvironment(environmentBuilder.build());
                        }
                      }
                    }
                  }
                ```
          
          - 最后通过build方法返回一个SqlSession
          
            ```java
            return new DefaultSqlSessionFactory(config);//通过Configuration对象创建DefaultSqlSessionFactory，其构造方法也只是对configuration进行赋值
            ```
        
      - ### 总结
      
        - （1）先创建XMLConfigBuilder对象，该类中包括了XML解析器（XPathParser）、Configuration、环境名称。
        - （2）XPathParser的创建，通过本地DTD文件创建Document对象
        - （3）执行XMLConfigBuilder构造方法，初始化Configruation对象
        - （4）开始解析XML文件并将文件，获得Node对象，之后将信息存储至Configruation中并返回Configruation
        - （5）通过Configruation创建DefaultSqlSessionFactory并返回
    
  - ## 创建SqlSession执行对象
  
    - 入口
  
      ```java
      SqlSession session = factory.openSession();
      ```
  
    - 执行openSession方法
  
      ```java
      return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
      //三个参数分别是configuration中的执行器对象，事务的隔离级别，是否自动提交
      ```
  
      - configuration.getDefaultExecutorType(获取执行器对象)
  
        ```java
        public ExecutorType getDefaultExecutorType() {
            return defaultExecutorType;
          }
        //获取单一的执行器对象
        protected ExecutorType defaultExecutorType = ExecutorType.SIMPLE
        //    ExecutorType类，分别是单一语句执行、复用语句执行、批量语句执行
        public enum ExecutorType {
          SIMPLE, REUSE, BATCH
        }
        ```
  
      - openSessionFromDataSource
  
        ```java
        private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
          Transaction tx = null;
          try {
              //获取环境信息
            final Environment environment = configuration.getEnvironment();
              //通过环境信息创建事务工厂
            final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
              //通过事务工厂和事务级别和是否自动提交获取事务
            tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
              //通过执行对象类型创建执行器
            final Executor executor = configuration.newExecutor(tx, execType);
              //通过configuration、执行器、是否自动提交创建一个DefaultSqlSession并返回
            return new DefaultSqlSession(configuration, executor, autoCommit);
          } catch (Exception e) {
            closeTransaction(tx); // may have fetched a connection so lets call close()
            throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
          } finally {
            ErrorContext.instance().reset();
          }
        }
        ```
  
        - getTransactionFactoryFromEnvironment方法
  
        ```java
        private TransactionFactory getTransactionFactoryFromEnvironment(Environment environment) {
            if (environment == null || environment.getTransactionFactory() == null) {//如果环境为空或者环境没有设置事务工厂则创建ManagedTransactionFactory
              return new ManagedTransactionFactory();
            }
          
            return environment.getTransactionFactory();
          }
        ```
  
        - newTransaction方法（通过事务工厂创建事务）
  
          ```java
          public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {
              return new JdbcTransaction(ds, level, autoCommit);//底层采用JDBC创建事务，JdbcTransaction的构造方法也只是将JdbcTransaction中的数据源，事务隔离级别，是否自动提交进行赋值
            }
          ```
  
        - newExecutor方法（Configuration创建执行器对象）
  
          ```java
          public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
              executorType = executorType == null ? defaultExecutorType : executorType;//若executorType为空则采用默认的执行器
              executorType = executorType == null ? ExecutorType.SIMPLE : executorType;//若executorType为空则采用默认的执行器类型
              Executor executor;
              if (ExecutorType.BATCH == executorType) {//根据不同的ExecutorType创建执行器
                  executor = new BatchExecutor(this, transaction);
              } else if (ExecutorType.REUSE == executorType) {
                  executor = new ReuseExecutor(this, transaction);
              } else {
                  executor = new SimpleExecutor(this, transaction);
              }
              if (cacheEnabled) {//如果开启缓存，则创建一个缓存执行器，缓存执行器中包含一个普通执行器
                  executor = new CachingExecutor(executor);
              }
              //将该执行器加入执行链中
              executor = (Executor) interceptorChain.pluginAll(executor);
              return executor;
          }
          ```
  
          - 创建缓存执行器(new CachingExecutor)
  
            ```java
            public CachingExecutor(Executor delegate) {
                this.delegate = delegate;
                delegate.setExecutorWrapper(this);
            }
            ```
  
          - 将执行器加入至执行链中（pluginAll）
  
            ```java
            //按序存放拦截器对象，拦截器对象通过Mybatis配置文件中可以配置（通过plugin标签）
            private final List<Interceptor> interceptors = new ArrayList<>();
            
            public Object pluginAll(Object target) {
                //如果没有配置拦截器插件则不会执行For，直接返回执行器对象，如果存在拦截器对象，则for将所有拦截器通过动态代理添加到执行器对象之前
                for (Interceptor interceptor : interceptors) {
                    target = interceptor.plugin(target);
                }
                return target;
            }
            ```
  
            - plugin方法以及之后执行的方法
  
              ```java
               default Object plugin(Object target) {
                  return Plugin.wrap(target, this);
                }
              
              //wrap方法以及成员变量
              
                private final Object target;
                private final Interceptor interceptor;
                private final Map<Class<?>, Set<Method>> signatureMap;//封装的需要代理的类和需要被加强的方法
              
              
              public static Object wrap(Object target, Interceptor interceptor) {
                  Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);//将拦截器中的方法封装为signatureMap
                  Class<?> type = target.getClass();
                  Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
                  if (interfaces.length > 0) {//如果继承的接口数量>0则进行动态代理
                    return Proxy.newProxyInstance(
                        type.getClassLoader(),
                        interfaces,
                        new Plugin(target, interceptor, signatureMap));
                  }
                  return target;
                }
              ```
  
              - getSignatureMap方法
  
                ```java
                private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
                    //获取拦截器类上的注解
                    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
                    // issue #251
                    if (interceptsAnnotation == null) {//判断是否为拦截器
                      throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());
                    }
                    Signature[] sigs = interceptsAnnotation.value();//获取拦截器中的方法签名数组，Signature包含返回类型，参数列表，方法类
                    Map<Class<?>, Set<Method>> signatureMap = new HashMap<>();
                    for (Signature sig : sigs) {
                        //如果指定的键尚未与值相关联（或映射到 null ），则尝试使用给定的映射函数计算其值，并将其输入到此映射中
                      Set<Method> methods = signatureMap.computeIfAbsent(sig.type(), k -> new HashSet<>());
                      try {
                          //获取方法类
                        Method method = sig.type().getMethod(sig.method(), sig.args());
                        methods.add(method);
                      } catch (NoSuchMethodException e) {
                        throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + e, e);
                      }
                    }
                    return signatureMap;
                  }
                ```
  
      - 最终返回DefaultSqlSession
  
    - ### 总结
  
      - 根据执行器类型和事务隔离级别以及是否自动提交创建执行器对象最后创建DefaultSqlSession并返回
  
  - ## 调用SqlSession获取Mapper类
  
    - 入口
  
      ```java
      UserDao mapper = session.getMapper(UserDao.class);//获取到的UserDao是个动态代理对象（JDK）
      ```
  
      - 调用configuration的getMapper方法
  
        ```java
        public <T> T getMapper(Class<T> type) {//第一步调用，来自sqlsession
            return configuration.getMapper(type, this);
        }
        public <T> T getMapper(Class<T> type, SqlSession sqlSession) {//第二步调用，来自Configuration
            return mapperRegistry.getMapper(type, sqlSession);
          }
        ```
  
        - getMapper方法（来自mapperRegistry）
  
        ```java
        //mapperRegistry成员变量
          private final Configuration config;
          private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();//已知的Mapper
        
        public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
            final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
            if (mapperProxyFactory == null) {
              throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
            }
            try {
              return mapperProxyFactory.newInstance(sqlSession);
            } catch (Exception e) {
              throw new BindingException("Error getting mapper instance. Cause: " + e, e);
            }
          }
        ```
  
        - knownMappers的初始化
  
          ```java
          public void addMappers(String packageName) {
              addMappers(packageName, Object.class);
          }
          
          public void addMappers(String packageName, Class<?> superType) {
              ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
              resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
              Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
              for (Class<?> mapperClass : mapperSet) {
                  addMapper(mapperClass);
              }
          }
          
          public <T> void addMapper(Class<T> type) {
              if (type.isInterface()) {
                if (hasMapper(type)) {
                  throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
                }
                boolean loadCompleted = false;
                try {
                  knownMappers.put(type, new MapperProxyFactory<>(type));
                  MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
                  parser.parse();
                  loadCompleted = true;
                } finally {
                  if (!loadCompleted) {
                    knownMappers.remove(type);
                  }
                }
              }
            }
          ```
  
          - addMapper中的MapperAnnotationBuilder创建
  
            ```java
            String resource = type.getName().replace('.', '/') + ".java (best guess)";//将class的引用名称转换为路径名称
            this.assistant = new MapperBuilderAssistant(configuration, resource);
            this.configuration = configuration;
            this.type = type;
            ```
  
            - MapperBuilderAssistant（Mapper创建助手）的创建
  
              ```java
              
              public MapperBuilderAssistant(Configuration configuration, String resource) {
                  super(configuration);
                  //以下是父类的构造方法
                  //   public BaseBuilder(Configuration configuration) {
                  //   this.configuration = configuration;
                  //   this.typeAliasRegistry = this.configuration.getTypeAliasRegistry();
                  //   this.typeHandlerRegistry = this.configuration.getTypeHandlerRegistry();
                  // }
                  ErrorContext.instance().resource(resource);
                  this.resource = resource;
              }
              ```
  
          - MapperAnnotationBuilder的parse方法
  
            ```java
              protected final Set<String> loadedResources = new HashSet<>(); 
            protected final Map<String, XNode> sqlFragments = new StrictMap<>("XML fragments parsed from previous mappers");
            
            public void parse() {
                String resource = type.toString();//type是mapper的class类，resource是包名
                if (!configuration.isResourceLoaded(resource)) {//判断loadedResources中是否含有该resource
                  loadXmlResource();//加载配置文件
                  configuration.addLoadedResource(resource);//loadedResources添加该mapper包名（单纯 Set集合添加）
                    //调用用mapper创建助手设置命名空间，mapper创建助手中包含成员变量resource（mapper包名），命名空间，该方法会检查mapper创建助手中的命名空间是否和传入的命名空间相同，不同则抛出异常，相同则进行赋值
                  assistant.setCurrentNamespace(type.getName());
                  parseCache();
                  parseCacheRef();
                  Method[] methods = type.getMethods();
                  for (Method method : methods) {
                    try {
                      // issue #237
                      if (!method.isBridge()) {//检查该方法是否是桥接方法，桥接方法具体定义可查看以下链接
                          //https://blog.csdn.net/qq_40272978/article/details/107370187
                        parseStatement(method);//使用JDBC预编译SQL语句
                      }
                    } catch (IncompleteElementException e) {
                      configuration.addIncompleteMethod(new MethodResolver(this, method));
                    }
                  }
                }
                parsePendingMethods();
              }
            ```
  
            - loadXmlResource方法
  
            ```java
             if (!configuration.isResourceLoaded("namespace:" + type.getName())) {
                  String xmlResource = type.getName().replace('.', '/') + ".xml";//将包名替换为路径名
                  // #1347
                  InputStream inputStream = type.getResourceAsStream("/" + xmlResource);//获取输入流
                  if (inputStream == null) {//如果输入流为空则使用Resources重新读取
                    // Search XML mapper that is not in the module but in the classpath.
                    try {
                        //这一步也就是整个程序运行的第一步
                      inputStream = Resources.getResourceAsStream(type.getClassLoader(), xmlResource);
                    } catch (IOException e2) {
                      // ignore, resource is not required
                    }
                  }
                  if (inputStream != null) {//如果输入流不未空则使用XML读取，重复整个程序运行的第二步
                    XMLMapperBuilder xmlParser = new XMLMapperBuilder(inputStream, assistant.getConfiguration(), xmlResource, configuration.getSqlFragments(), type.getName());
                    xmlParser.parse();
                  }
                }
            ```
  
            - parseStatement方法
  
              ```java
              void parseStatement(Method method) {
                  //获取Mapper接口方法上的参数类型，如果有多个参数则返回一个HashMap
                  Class<?> parameterTypeClass = getParameterType(method);
                  //创建语言驱动，主要作用是生成SqlSource
                  LanguageDriver languageDriver = getLanguageDriver(method);
                  //根据注解创建SqlSource
                  SqlSource sqlSource = getSqlSourceFromAnnotations(method, parameterTypeClass, languageDriver);
                  if (sqlSource != null) {//如果采用注解生成SQL语句则执行以下语句
                      //MyBatis的@Options注解能够设置缓存时间，能够为对象生成自增的主键值
                    Options options = method.getAnnotation(Options.class);
                      //sql语句ID
                    final String mappedStatementId = type.getName() + "." + method.getName();
                    Integer fetchSize = null;
                    Integer timeout = null;
                      //设置JDBC采用提前编译SQL
                    StatementType statementType = StatementType.PREPARED;
                    ResultSetType resultSetType = configuration.getDefaultResultSetType();
                      //获取SQL语句类型（select、update等）
                    SqlCommandType sqlCommandType = getSqlCommandType(method);
                    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
                      //如果是查询语句则不刷新缓存
                    boolean flushCache = !isSelect;
                      //查询语句则采用缓存
                    boolean useCache = isSelect;
              //sql语句执行前和执行后的操作类
                    KeyGenerator keyGenerator;
                    String keyProperty = null;
                    String keyColumn = null;
                    if (SqlCommandType.INSERT.equals(sqlCommandType) || SqlCommandType.UPDATE.equals(sqlCommandType)) {
                      // first check for SelectKey annotation - that overrides everything else
                        //获取selectKey注解，selectKey用于在已被指定SQL语句的方法前或者后执行的SQL语句，具体用法参考下面说明
                      SelectKey selectKey = method.getAnnotation(SelectKey.class);
                        //https://blog.csdn.net/Schaffer_W/article/details/115536494
                      if (selectKey != null) {
                          //执行keyGenerator前置方法
                        keyGenerator = handleSelectKeyAnnotation(selectKey, mappedStatementId, getParameterType(method), languageDriver);
                        keyProperty = selectKey.keyProperty();
                      } else if (options == null) {
                        keyGenerator = configuration.isUseGeneratedKeys() ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
                      } else {
                        keyGenerator = options.useGeneratedKeys() ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
                        keyProperty = options.keyProperty();
                        keyColumn = options.keyColumn();
                      }
                    } else {
                      keyGenerator = NoKeyGenerator.INSTANCE;
                    }
              
                    if (options != null) {
                      if (FlushCachePolicy.TRUE.equals(options.flushCache())) {
                        flushCache = true;
                      } else if (FlushCachePolicy.FALSE.equals(options.flushCache())) {
                        flushCache = false;
                      }
                      useCache = options.useCache();
                      fetchSize = options.fetchSize() > -1 || options.fetchSize() == Integer.MIN_VALUE ? options.fetchSize() : null; //issue #348
                      timeout = options.timeout() > -1 ? options.timeout() : null;
                      statementType = options.statementType();
                      if (options.resultSetType() != ResultSetType.DEFAULT) {
                        resultSetType = options.resultSetType();
                      }
                    }
              
                    String resultMapId = null;
                    ResultMap resultMapAnnotation = method.getAnnotation(ResultMap.class);
                    if (resultMapAnnotation != null) {
                      resultMapId = String.join(",", resultMapAnnotation.value());
                    } else if (isSelect) {
                      resultMapId = parseResultMap(method);
                    }
              
                    assistant.addMappedStatement(
                        mappedStatementId,
                        sqlSource,
                        statementType,
                        sqlCommandType,
                        fetchSize,
                        timeout,
                        // ParameterMapID
                        null,
                        parameterTypeClass,
                        resultMapId,
                        getReturnType(method),
                        resultSetType,
                        flushCache,
                        useCache,
                        // TODO gcode issue #577
                        false,
                        keyGenerator,
                        keyProperty,
                        keyColumn,
                        // DatabaseID
                        null,
                        languageDriver,
                        // ResultSets
                        options != null ? nullOrEmpty(options.resultSets()) : null);
                  }
                }
              ```
        
          - 在上文中的创建Configurarion的方法中XMLConfigBuild调用parseConfiguration方法读入标签信息并将信息存储到Configuration中，也就是parseConfiguration方法中
        
            ```java
            mapperElement(root.evalNode("mappers"));//添加mapper,随后XMLConfigBuild调用了mapperElement方法
            ```
        
            - mapperElement方法
        
              ```java
              private void mapperElement(XNode parent) throws Exception {
                  if (parent != null) {
                      for (XNode child : parent.getChildren()) {
                          if ("package".equals(child.getName())) {
                              String mapperPackage = child.getStringAttribute("name");//获取mapper包名
                              configuration.addMappers(mapperPackage);//调用configuration将该mapper添加至knownMappers中
                          } //后面的代码都是解析XML文件，省略
              ```
        
        - mapperProxyFactory的类初始化方法
        
          ```java
          public T newInstance(SqlSession sqlSession) {
              //创建mapperProxy
              final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
              //使用MapperProxy动态代理
              return newInstance(mapperProxy);
          }
          
          
          protected T newInstance(MapperProxy<T> mapperProxy) {
              //JDK动态代理，返回经过代理后的Mapper类，mapperInterface.getClassLoader()是需要被代理的类的类加载器，第二个参数是被代理类所继承的接口，第三个类是完成代理的类，也就是实现了InvocationHandler接口的类
              return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
            }
          ```
  
  - ## 通过Mapper执行SQL语句
  
    - CGlib动态代理
  
      - JDK动态代理需要接口才能代理而CGlib对于普通类也能动态代理，底层采用Java字节码操作框架ASM，最终效果是通过继承实现
      - CGlib无法对final方法和final类进行动态代理
      - 大致原理
        - 动态生成需要动态代理的类的子类，在子类中采用方法拦截的技术拦截对父类方法的调用
  
      - 案例
  
        - 编写一个需要被代理的类
  
          ```java
          public class Talk {
              public void takeTalk()
              {
                  System.out.println("this a talk");
              }
          }
          ```
  
        - 编写一个被代理的子类
  
          ```java
          
          public class TalkImpl extends  Talk{
              @Override
              public void takeTalk() {
                  super.takeTalk();
                  System.out.println("this is TalkImpl");
              }
          }
          ```
  
        - 编写代理类
  
          ```java
          public class CGlibProxy implements MethodInterceptor {
          
              //需要被代理的类
              Object object;
          
          
              public<T> T getProxy(Class<T> clazz)
              {
                  Enhancer enhancer = new Enhancer();
                  // MethodInterceptor接口继承了Callback接口
                  //如果被代理的类是一个接口则可以使用以下代码
                  //enhancer.setInterfaces(clazz);
                  enhancer.setSuperclass(clazz);
                  enhancer.setCallback(this);
                  return (T)enhancer.create();
              }
          
              @Override//类似于JDK的invoke方法,Object是需要被代理的类，objects是方法参数，methodProxy是代理类对象
              public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                  System.out.println("this is cglib proxy start.......");
                  methodProxy.invokeSuper(o,objects);//如果被代理类是一个接口的话，这里就需要先实现接口的方法才能调用
                  System.out.println("this is cglib proxy end.......");
                  return null;
              }
          }
          ```
  
        - 测试
  
          ```java
           public static void main(String[] args) {
                  CGlibProxy cGlibProxy = new CGlibProxy();
                  Talk proxy = cGlibProxy.getProxy(TalkImpl.class);//如果代理类是接口的话这里需要替换为以下代码
               //Talk proxy = cGlibProxy.getProxy(Talk.class);
                  proxy.takeTalk();
              }
          //结果
          //this is cglib proxy start.......
          //this a talk
          //this is TalkImpl
          //this is cglib proxy end......
          ```
    
    - ### SQL语句执行
    
      - 入口
    
        ```java
        User user= mapper.selectById(1);
        ```
    
        - 进入动态代理类的invoke方法（由于本身mapper接口并没有实现，所以在源码中mybatis并没有调用接口实现类的方法而是自己实现了mapper接口）
    
          ```java
          public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
              try {
                if (Object.class.equals(method.getDeclaringClass())) {//检查proxy类是否是object类型，如果是object类型则不拦截，原因是OBject中的方法既是tostring等方法无需拦截
                  return method.invoke(this, args);
                } else {//cachedInvoker方法是用于获取MapperMethod对象，且将MapperMethod对象放入Cache（Map）中，第二次及其之后的调用都会直接从缓存中获取
                  return cachedInvoker(method).invoke(proxy, method, args, sqlSession);//cachedInvoker主要是负责ResultMap，cache，SQL语句预编译等工作
                }
              } catch (Throwable t) {
                throw ExceptionUtil.unwrapThrowable(t);
              }
            }
          ```
    
          - MapperMethod的invoke方法
          
            ```java
            public Object invoke(Object proxy, Method method, Object[] args, SqlSession sqlSession) throws Throwable {
                  return mapperMethod.execute(sqlSession, args);//这里的mapperMethod是cachedInvoker中创建的MapperMethod
                }
            //mapperMethod的execute方法
            public Object execute(SqlSession sqlSession, Object[] args) {
                Object result;
                switch (command.getType()) {//根据SQL类型调用方法
                  case INSERT: {
                    Object param = method.convertArgsToSqlCommandParam(args);
                    result = rowCountResult(sqlSession.insert(command.getName(), param));
                    break;
                  }
                  case UPDATE: {
                    Object param = method.convertArgsToSqlCommandParam(args);
                    result = rowCountResult(sqlSession.update(command.getName(), param));
                    break;
                  }
                  case DELETE: {
                    Object param = method.convertArgsToSqlCommandParam(args);
                    result = rowCountResult(sqlSession.delete(command.getName(), param));
                    break;
                  }
                  case SELECT:
                    if (method.returnsVoid() && method.hasResultHandler()) {//是否返回空且含有结果处理器
                      executeWithResultHandler(sqlSession, args);//通过结果处理器处理SQL
                      result = null;
                    } else if (method.returnsMany()) {
                      result = executeForMany(sqlSession, args);//是否返回多条结果
                    } else if (method.returnsMap()) {//是否返回Map
                      result = executeForMap(sqlSession, args);
                    } else if (method.returnsCursor()) {
                      result = executeForCursor(sqlSession, args);//是否返回游标
                    } else {//单条记录查询
                      Object param = method.convertArgsToSqlCommandParam(args);//通过类型转换器将参数转换
                      result = sqlSession.selectOne(command.getName(), param);
                      if (method.returnsOptional()
                          && (result == null || !method.getReturnType().equals(result.getClass()))) {
                        result = Optional.ofNullable(result);
                      }
                    }
                    break;
                  case FLUSH:
                    result = sqlSession.flushStatements();//Flush类型
                    break;
                  default:
                    throw new BindingException("Unknown execution method for: " + command.getName());
                }
                if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {//isPrimitive方法用于判断类型是否是基础类型
                  throw new BindingException("Mapper method '" + command.getName()
                      + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
                }
                return result;
              }
            
            ```
          
            - convertArgsToSqlCommandParam方法
          
              ```java
               private final ParamNameResolver paramNameResolver;
              
              public Object convertArgsToSqlCommandParam(Object[] args) {
                    return paramNameResolver.getNamedParams(args);//paramNameResolver是参数名处理器，在MethodSignure中
                  }
              
              //ParamNameResolver的getNamedParams方法
              //以下是ParamNameResolver成员变量和方法
                public static final String GENERIC_NAME_PREFIX = "param";
              
                private final SortedMap<Integer, String> names;
              
                private boolean hasParamAnnotation;
              
              
              public Object getNamedParams(Object[] args) {
                  final int paramCount = names.size();//参数数量
                  if (args == null || paramCount == 0) {//没有参数的情况
                    return null;
                  } else if (!hasParamAnnotation && paramCount == 1) {//只有一个参数的情况
                    return args[names.firstKey()];
                  } else {//复杂类型的 返回通过Map返回
                    final Map<String, Object> param = new ParamMap<>();//只是个HashMap
                    int i = 0;
                    for (Map.Entry<Integer, String> entry : names.entrySet()) {
                      param.put(entry.getValue(), args[entry.getKey()]);//将参数对应位置放入Map集合
                      // 添加通用参数名称（param1，param2，...）
                      final String genericParamName = GENERIC_NAME_PREFIX + (i + 1);
                      // 确保不覆盖以@Param 命名的参数
                      if (!names.containsValue(genericParamName)) {
                        param.put(genericParamName, args[entry.getKey()]);
                      }
                      i++;
                    }
                    return param;
                  }
                }
              ```
          
            - sqlSession的selectOne方法
          
              ```java
               public <T> T selectOne(String statement, Object parameter) {//statement为方法全名，parameter为参数
                  // Popular vote was to return null on 0 results and throw exception on too many.
                  List<T> list = this.selectList(statement, parameter);
                  if (list.size() == 1) {
                    return list.get(0);
                  } else if (list.size() > 1) {
                    throw new TooManyResultsException("Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
                  } else {
                    return null;
                  }
                }
              ```
          
              - selectList方法
          
              ```java
              public <E> List<E> selectList(String statement, Object parameter) {
                return this.selectList(statement, parameter, RowBounds.DEFAULT);
              }
              
                public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
                  try {
                    MappedStatement ms = configuration.getMappedStatement(statement);//获取MappedStatement，MappedStatement是Mapper.xml文件的包装类
                    return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
                  } catch (Exception e) {
                    throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
                  } finally {
                    ErrorContext.instance().reset();
                  }
                }
              ```
          
              - wrapCollection方法
          
                ```java
                private Object wrapCollection(final Object object) {//对Collection和数组类的参数进行包装
                    if (object instanceof Collection) {
                      StrictMap<Object> map = new StrictMap<>();
                      map.put("collection", object);
                      if (object instanceof List) {
                        map.put("list", object);
                      }
                      return map;
                    } else if (object != null && object.getClass().isArray()) {
                      StrictMap<Object> map = new StrictMap<>();
                      map.put("array", object);
                      return map;
                    }
                    return object;//普通参数直接返回Object即可
                  }
                ```
          
                
          
              - 执行器的query
          
                ```java
                public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
                    BoundSql boundSql = ms.getBoundSql(parameterObject);//获取SQL语句
                    CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);//获取缓存KEY，Mybatis中含有两级缓存，一级缓存基于Session，二级查询基于查询，二级缓存默认关闭，如果需要使用缓存可以在Mapper文件中的SQL语句上添加属性useCache="true"
                    return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                } 
                ```
          
                - query方法
          
                  ```java
                  public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
                        throws SQLException {
                      Cache cache = ms.getCache();
                      if (cache != null) {
                        flushCacheIfRequired(ms);//刷新所有缓存如果在SQL标签上有指定的话
                        if (ms.isUseCache() && resultHandler == null) {
                          ensureNoOutParams(ms, boundSql);//确保没有输出参数
                          @SuppressWarnings("unchecked")
                            //tcm是事务缓存管理器（TransactionalCacheManager，在执行器的内部），以下代码是为了获取缓存中的SQL语句以及查询结果
                          List<E> list = (List<E>) tcm.getObject(cache, key);
                          if (list == null) {//在缓存中找不到的情况下
                              //delegate是一个简单的执行器对象（SimpleExecutor），执行SQL语句并将结果存入list当中
                            list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                              //将该语句的结果和Key放入缓存（TransactionalCacheManager）中
                            tcm.putObject(cache, key, list); // issue #578 and #116
                          }
                          return list;//返回查询结果
                        }
                      }
                      //没有缓存 的情况下执行查询
                      return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
                    }
                  ```
                  
                  - 简单执行器的query方法（实际上采用的是BaseExecutor的query方法）
                  
                    ```java
                    public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
                        ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
                        if (closed) {//closed是执行器中的成员变量，用于标识该执行器是否关闭
                          throw new ExecutorException("Executor was closed.");
                        }
                        if (queryStack == 0 && ms.isFlushCacheRequired()) {//判断当前查询的嵌套层数（根据查询次数）是否为0和是否需要清楚缓存
                          clearLocalCache();//清除缓存
                        }
                        List<E> list;
                        try {
                          queryStack++;//增加查询次数
                            //判断是否采用缓存，如果不采用缓存则list为空，采用缓存则使用缓存中的resultHandler
                          list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
                          if (list != null) {//这一步暂时不知道干嘛
                            handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
                          } else {//调用JDBC查询
                            list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
                          }
                        } finally {
                          queryStack--;
                        }
                        if (queryStack == 0) {
                          for (DeferredLoad deferredLoad : deferredLoads) {
                            deferredLoad.load();
                          }
                          // issue #601
                          deferredLoads.clear();
                          if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
                            // issue #482
                            clearLocalCache();
                          }
                        }
                        return list;
                      }
                    ```
                  
                    - queryFromDatabase方法
                  
                      ```java
                        private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
                          List<E> list;
                          localCache.putObject(key, EXECUTION_PLACEHOLDER);//将key放入缓存中
                          try {
                            list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);//执行查询语句,这里的doquery调用的是SimpleExecutor的方法
                          } finally {
                            localCache.removeObject(key);//从缓存中删除key
                          }
                          localCache.putObject(key, list);
                          if (ms.getStatementType() == StatementType.CALLABLE) {
                            localOutputParameterCache.putObject(key, parameter);
                          }
                          return list;
                        }
                      ```
                  
                      - doquery方法（封装JDBC）
                  
                        ```java
                        public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
                            Statement stmt = null;
                            try {
                              Configuration configuration = ms.getConfiguration();
                              StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
                              stmt = prepareStatement(handler, ms.getStatementLog());
                              return handler.query(stmt, resultHandler);
                            } finally {
                              closeStatement(stmt);
                            }
                          }
                        ```
                  
                        - newStatementHandler方法（创建StatementHandler）
                  
                          ```java
                          public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
                              StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);//创建一个RoutingStatementHandler类
                              statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);//将StatementHandler放入拦截器链中
                              return statementHandler;
                          }
                          ```
                  
                          - RoutingStatementHandler创建
                  
                            ```java
                            public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {//策略模式，根据不同的情形采用不同的处理方式
                            
                                switch (ms.getStatementType()) {
                                    case STATEMENT:
                                        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                                        break;
                                    case PREPARED:
                                        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                                        break;
                                    case CALLABLE://Callable是存储过程对象
                                        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                                        break;
                                    default:
                                        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
                                }
                            
                            }
                            ```
                  
                        - prepareStatement方法
                  
                          ```java
                          private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
                              Statement stmt;
                              Connection connection = getConnection(statementLog);//获取Connection，这里采用的是动态代理
                              stmt = handler.prepare(connection, transaction.getTimeout());//获取Statement
                              handler.parameterize(stmt);//设置参数
                              return stmt;
                          }
                          ```
                  
                          - getConnection方法
                  
                            ```java
                            protected Connection getConnection(Log statementLog) throws SQLException {
                                Connection connection = transaction.getConnection();//从事务中获取连接,事务是从openSession方法时创建
                                if (statementLog.isDebugEnabled()) {
                                    return ConnectionLogger.newInstance(connection, statementLog, queryStack);
                                } else {
                                    return connection;
                                }
                            }
                            ```
                  
                            - 事务的getConnection方法
                  
                            ```java
                             public Connection getConnection() throws SQLException {
                                if (connection == null) {
                                  openConnection();
                                }
                                return connection;
                              }
                            
                            
                            //openConnection方法
                            protected void openConnection() throws SQLException {
                                if (log.isDebugEnabled()) {//是否开启debug
                                    log.debug("Opening JDBC Connection");
                                }
                                connection = dataSource.getConnection();//从数据源中获取Connection
                                if (level != null) {
                                    connection.setTransactionIsolation(level.getLevel());//通过Connection获取事务类型
                                }
                                setDesiredAutoCommit(autoCommit);//是否启用自动提交
                            }
                            
                            //从dataSource中获取连接
                            public Connection getConnection() throws SQLException {
                                //这个连接池时Mybatis中自带的，Connection最后采用的还是代理连接
                                //popConnection实现了InvokeHandler（代理类需要实现的接口）
                                return popConnection(dataSource.getUsername(), dataSource.getPassword()).getProxyConnection();
                            }
                            
                            //popConnection的构造方法
                            //重点语句（其余部分省略）
                            //popConnection本身还是JDBCConnection的包装类同时也是一个动态代理，其初始化方法时对连接的一些基本属性进行赋值并通过动态代理得到代理连接，下面有该类的invoke方法
                            conn = new PooledConnection(dataSource.getConnection(), this);
                            //dataSource的获取连接方法
                            public Connection getConnection() throws SQLException {
                                return doGetConnection(username, password);
                            }
                            //doGetConnection方法
                            private Connection doGetConnection(String username, String password) throws SQLException {
                                Properties props = new Properties();//使用Properties封装配置消息
                                if (driverProperties != null) {
                                    props.putAll(driverProperties);
                                }
                                if (username != null) {
                                    props.setProperty("user", username);//设置用户名并放入Properties中
                                }
                                if (password != null) {
                                    props.setProperty("password", password);//设置密码并放入Properties中
                                }
                                return doGetConnection(props);//通过Properties构造连接
                                
                            }
                            
                            //通过Properties构造连接的doGetConnection方法
                            private Connection doGetConnection(Properties properties) throws SQLException {
                                initializeDriver();//初始化驱动，这里其实就是JDBC中的第一步，启动驱动的类加载器和注册驱动
                                Connection connection = DriverManager.getConnection(url, properties);//通过驱动管理获取连接
                                configureConnection(connection);//设置的事务级别，是否自动提交，设置连接超时时间
                                return connection;
                              }
                            ```
                  
                            - PooledConnection的invoke方法
                  
                              ```java
                              public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                                  String methodName = method.getName();//获取方法名
                                  if (CLOSE.equals(methodName)) {
                                      //如果是Close方法则将连接归还至连接池中
                                      //在连接池中Connection存储在list集合中，有两个list集合，一个用于放置空闲连接池，一个用于放置活跃连接
                                      dataSource.pushConnection(this);
                                      return null;
                                  }
                                  try {
                                      if (!Object.class.equals(method.getDeclaringClass())) {
                                          // issue #579 toString() should never fail
                                          // throw an SQLException instead of a Runtime
                                          checkConnection();
                                      }
                                      return method.invoke(realConnection, args);
                                  } catch (Throwable t) {
                                      throw ExceptionUtil.unwrapThrowable(t);
                                  }
                              
                              }
                              ```
                  
                        - StatementHandler的prepare方法
                        
                          ```java
                          public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
                              return delegate.prepare(connection, transactionTimeout);
                            }
                          
                          //delegate的prepare方法
                          public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
                              ErrorContext.instance().sql(boundSql.getSql());
                              Statement statement = null;
                              try {
                                  statement = instantiateStatement(connection);//获取实例化语句
                                  setStatementTimeout(statement, transactionTimeout);//初始化事务超时时间
                                  setFetchSize(statement);
                                  return statement;
                              } catch (SQLException e) {
                                  closeStatement(statement);
                                  throw e;
                              } catch (Exception e) {
                                  closeStatement(statement);
                                  throw new ExecutorException("Error preparing statement.  Cause: " + e, e);
                              }
                          }
                          ```
                        
                          - instantiateStatement
                        
                            ```java
                            protected Statement instantiateStatement(Connection connection) throws SQLException {
                                String sql = boundSql.getSql();
                                //是否有KeyGenerator在SQL语句前或者后执行
                                if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
                                    String[] keyColumnNames = mappedStatement.getKeyColumns();
                                    if (keyColumnNames == null) {
                                        return connection.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);
                                    } else {
                                        return connection.prepareStatement(sql, keyColumnNames);
                                    }
                                } else if (mappedStatement.getResultSetType() == ResultSetType.DEFAULT) {
                                    //采用默认的结果集预编译JDBC的SQL语句
                                    return connection.prepareStatement(sql);
                                } else {
                                    return connection.prepareStatement(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
                                }
                            }
                            ```
                        
                        - parameterize方法
                        
                          ```java
                          public void parameterize(Statement statement) throws SQLException {//RoutingStatementHandler的parameterize方法
                              delegate.parameterize(statement);
                            }
                          
                          public void parameterize(Statement statement) throws SQLException {//PreparedStatementHandler的parameterize方法
                              parameterHandler.setParameters((PreparedStatement) statement);
                            }
                          
                           @Override
                            public void setParameters(PreparedStatement ps) {//DefaultParameterHandler的setParameters方法
                              ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
                                //获取参数映射
                              List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
                              if (parameterMappings != null) {
                                for (int i = 0; i < parameterMappings.size(); i++) {
                                    //获取对应位置的参数
                                  ParameterMapping parameterMapping = parameterMappings.get(i);
                                  if (parameterMapping.getMode() != ParameterMode.OUT) {//如果该参数不是输出参数
                                    Object value;
                                    String propertyName = parameterMapping.getProperty();
                                      //判断参数名称是否符合标准（不是很明白）
                                    if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
                                      value = boundSql.getAdditionalParameter(propertyName);
                                    } else if (parameterObject == null) {
                                      value = null;
                                       //判断是否有该类型的参数处理器，这里的parameterObject是我们自己传入的参数
                                    } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                                      value = parameterObject;
                                    } else {
                                        //MetaObject是一个包装类其中包装了某个类及其类的构造工厂、反射工厂、包装器、包装器工厂
                                      MetaObject metaObject = configuration.newMetaObject(parameterObject);
                                        //获取需要的参数
                                      value = metaObject.getValue(propertyName);
                                    }
                                    TypeHandler typeHandler = parameterMapping.getTypeHandler();
                                    JdbcType jdbcType = parameterMapping.getJdbcType();
                                    if (value == null && jdbcType == null) {//如果没有参数则设置JDBCType为null类型
                                      jdbcType = configuration.getJdbcTypeForNull();
                                    }
                                    try {
                                        //设置statement参数
                                      typeHandler.setParameter(ps, i + 1, value, jdbcType);
                                    } catch (TypeException | SQLException e) {
                                      throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
                                    }
                                  }
                                }
                              }
                            
                          ```
                        
                          - setParameter参数
                        
                            ```java
                            @Override
                            public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
                                if (parameter == null) {
                                    if (jdbcType == null) {//两者都为空则抛出异常，如果没有参数则JDBCtype应该为NULL
                                        throw new TypeException("JDBC requires that the JdbcType must be specified for all nullable parameters.");
                                    }
                                    try {
                                        ps.setNull(i, jdbcType.TYPE_CODE);//将参数设置为空
                                    } catch (SQLException e) {
                                        throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . "
                                                                + "Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. "
                                                                + "Cause: " + e, e);
                                    }
                                } else {
                                    try {
                                        setNonNullParameter(ps, i, parameter, jdbcType);//设置非空参数
                                    } catch (Exception e) {
                                        throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType + " . "
                                                                + "Try setting a different JdbcType for this parameter or a different configuration property. "
                                                                + "Cause: " + e, e);
                                    }
                                }
                            }
                            
                            //setNonNullParameter方法
                            public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType)
                                  throws SQLException {
                                //获取参数类型解析器
                                TypeHandler handler = resolveTypeHandler(parameter, jdbcType);
                                //设置参数，源码根据参数是否为空而设置参数，如果不为空则调用TypeHandler中的方法设置参数
                                handler.setParameter(ps, i, parameter, jdbcType);
                              }
                            
                            ```
                        
                      - statementHandler的query方法
                      
                        ```java
                        //先调用RoutingStatementHandler的query方法后调用PreparedStatementHandler中的query方法
                        public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
                            return delegate.query(statement, resultHandler);
                          }
                        
                        public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
                            PreparedStatement ps = (PreparedStatement) statement;
                            ps.execute();
                            return resultSetHandler.handleResultSets(ps);
                          }
                        ```
                      
                        - DefaultResultSetHandler的handleResultSets方法
                      
                          ```java
                          public List<Object> handleResultSets(Statement stmt) throws SQLException {
                              ErrorContext.instance().activity("handling results").object(mappedStatement.getId());
                          
                              final List<Object> multipleResults = new ArrayList<>();
                          
                              int resultSetCount = 0;
                              ResultSetWrapper rsw = getFirstResultSet(stmt);
                          
                              List<ResultMap> resultMaps = mappedStatement.getResultMaps();//获取Mapper.xml文件中的所有ResultMap
                              int resultMapCount = resultMaps.size();//这里的size指的是ResultMap的数量
                              validateResultMapsCount(rsw, resultMapCount);//校验ResultMap的数量至少为1（在指定了ResultMap的情况下）
                              while (rsw != null && resultMapCount > resultSetCount) {
                                  ResultMap resultMap = resultMaps.get(resultSetCount);//获取ResultMap
                                  handleResultSet(rsw, resultMap, multipleResults, null);//调用重载方法 
                                  rsw = getNextResultSet(stmt);//获取下一结果集
                                  cleanUpAfterHandlingResultSet();//清楚已被获取的结果集
                                  resultSetCount++;
                              }
                          
                              String[] resultSets = mappedStatement.getResultSets();//如果mappedStatement的resultSets不为空则再次进行以上的操作
                              if (resultSets != null) {
                                  while (rsw != null && resultSetCount < resultSets.length) {
                                      ResultMapping parentMapping = nextResultMaps.get(resultSets[resultSetCount]);
                                      if (parentMapping != null) {
                                          String nestedResultMapId = parentMapping.getNestedResultMapId();
                                          ResultMap resultMap = configuration.getResultMap(nestedResultMapId);
                                          handleResultSet(rsw, resultMap, null, parentMapping);
                                      }
                                      rsw = getNextResultSet(stmt);
                                      cleanUpAfterHandlingResultSet();
                                      resultSetCount++;
                                  }
                              }
                          
                              return collapseSingleResultList(multipleResults);
                          }
                          
                          
                          //collapseSingleResultList方法
                          private List<Object> collapseSingleResultList(List<Object> multipleResults) {
                              return multipleResults.size() == 1 ? (List<Object>) multipleResults.get(0) : multipleResults;
                          }
                          ```
                      
                          
                      
                          - handleResultSet方法
                      
                          ```java
                          private void handleResultSet(ResultSetWrapper rsw, ResultMap resultMap, List<Object> multipleResults, ResultMapping parentMapping) throws SQLException {
                              try {
                                  if (parentMapping != null) {//没有父类ResultMap的情况
                                      handleRowValues(rsw, resultMap, null, RowBounds.DEFAULT, parentMapping);
                                  } else {
                                      if (resultHandler == null) {
                                          DefaultResultHandler defaultResultHandler = new DefaultResultHandler(objectFactory);//根据对象工厂创建ResultHandler
                                          //检查分页是否合法并将结果放入ResultHandler中
                                          handleRowValues(rsw, resultMap, defaultResultHandler, rowBounds, null);
                                          //向multipleResults添加ResultHandler中的结果
                                          multipleResults.add(defaultResultHandler.getResultList());
                                      } else {
                                          handleRowValues(rsw, resultMap, resultHandler, rowBounds, null);
                                      }
                                  }
                              } finally {
                                  // 关闭结果集
                                  closeResultSet(rsw.getResultSet());
                              }
                          }
                          ```
                      
                          - DefaultResultHandler的构造方法
                      
                            ```java
                              public DefaultResultHandler(ObjectFactory objectFactory) {
                                list = objectFactory.create(List.class);
                              }
                            ```
                      
                          - handleRowValues方法
                      
                          ```java
                          public void handleRowValues(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping) throws SQLException {
                              if (resultMap.hasNestedResultMaps()) {//是否有结果集的嵌套映射
                                ensureNoRowBounds();//检查分页是否合法
                                checkResultHandler();//确保ResultHandler不为空等配置项
                                   //将结果集放入ResultHandler中的list
                                handleRowValuesForNestedResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
                              } else {
                                  //将结果集放入ResultHandler中的list
                                handleRowValuesForSimpleResultMap(rsw, resultMap, resultHandler, rowBounds, parentMapping);
                              }
                            }
                          ```
                      
                          - handleRowValuesForSimpleResultMap方法
                      
                            ```java
                            private void handleRowValuesForSimpleResultMap(ResultSetWrapper rsw, ResultMap resultMap, ResultHandler<?> resultHandler, RowBounds rowBounds, ResultMapping parentMapping)
                                throws SQLException {
                                DefaultResultContext<Object> resultContext = new DefaultResultContext<>();//获取结果上下文，主要是为一些基础属性进行赋值
                                ResultSet resultSet = rsw.getResultSet();//获取JDBC的ResultSet
                                skipRows(resultSet, rowBounds);//根据分页情况跳过某些记录
                                while (shouldhandleRowValuesForSimpleResultMapProcessMoreRows(resultContext, rowBounds) && !resultSet.isClosed() && resultSet.next()) {//逐条获取结果
                                    //对该条结果使用校验器校验
                                    ResultMap discriminatedResultMap = resolveDiscriminatedResultMap(resultSet, resultMap, null);\
                                        //以对象的形式获取类
                                    Object rowValue = getRowValue(rsw, discriminatedResultMap, null);
                                    //将该条结果放入ResultHandler中的list
                                    storeObject(resultHandler, resultContext, rowValue, parentMapping, resultSet);
                                }
                            }
                            ```
                      
                            - getRowValue方法
                            
                            ```java
                            private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap, String columnPrefix) throws SQLException {
                                final ResultLoaderMap lazyLoader = new ResultLoaderMap();
                                //根据结果生成对象，这里获取到的类只是个模板并没有数据
                                Object rowValue = createResultObject(rsw, resultMap, lazyLoader, columnPrefix);
                                if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
                                    //使用MetaObject封装该类
                                  final MetaObject metaObject = configuration.newMetaObject(rowValue);
                                  boolean foundValues = this.useConstructorMappings;//是否使用了构造方法标签
                                  if (shouldApplyAutomaticMappings(resultMap, false)) {//判断是否采用自动映射根据是否有指定AutoMapping属性
                                      //根据是否采用自动映射赋值 ，自动映射既是将列名作为成员变量名称查找属性
                                    foundValues = applyAutomaticMappings(rsw, resultMap, metaObject, columnPrefix) || foundValues;
                                  }
                                  foundValues = applyPropertyMappings(rsw, resultMap, metaObject, lazyLoader, columnPrefix) || foundValues;
                                  foundValues = lazyLoader.size() > 0 || foundValues;
                                  rowValue = foundValues || configuration.isReturnInstanceForEmptyRow() ? rowValue : null;
                                }
                                return rowValue;
                              }
                            
                            //shouldApplyAutomaticMappings方法
                            private boolean shouldApplyAutomaticMappings(ResultMap resultMap, boolean isNested) {
                                if (resultMap.getAutoMapping() != null) {//是否指定AutoMapping属性
                                  return resultMap.getAutoMapping();
                                } else {
                                  if (isNested) {//是否采用延迟加载如果是则使用全自动映射
                                    return AutoMappingBehavior.FULL == configuration.getAutoMappingBehavior();
                                  } else {//否则不采用自动映射
                                    return AutoMappingBehavior.NONE != configuration.getAutoMappingBehavior();
                                  }
                                }
                              }
                            ```
                            
                            - createResultObject方法
                            
                              ```java
                              private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, ResultLoaderMap lazyLoader, String columnPrefix) throws SQLException {
                                  //使用构造方法构造对象
                                  this.useConstructorMappings = false; // reset previous mapping result
                                  //获取构造方法上参数的类型
                                  final List<Class<?>> constructorArgTypes = new ArrayList<>();
                                  //构造方法参数
                                  final List<Object> constructorArgs = new ArrayList<>();
                                  //通过对象包装数据，这里的resultObject只是个模板并没有任何数据
                                  Object resultObject = createResultObject(rsw, resultMap, constructorArgTypes, constructorArgs, columnPrefix);
                                  if (resultObject != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
                                      final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();
                                      for (ResultMapping propertyMapping : propertyMappings) {
                                          //循环遍历ResultMap判断是否存在延迟查询和懒加载，懒加载通过CGlib获取javassist
                                          if (propertyMapping.getNestedQueryId() != null && propertyMapping.isLazy()) {
                                              resultObject = configuration.getProxyFactory().createProxy(resultObject, lazyLoader, configuration, objectFactory, constructorArgTypes, constructorArgs);
                                              break;
                                          }
                                      }
                                  }
                                  this.useConstructorMappings = resultObject != null && !constructorArgTypes.isEmpty(); // set current mapping result
                                  return resultObject;//返回一个没有数据的类
                              }
                              ```
                            
                              - 重载createResultObject方法(根据构造方法创造对象)
                            
                                ```java
                                private Object createResultObject(ResultSetWrapper rsw, ResultMap resultMap, List<Class<?>> constructorArgTypes, List<Object> constructorArgs, String columnPrefix)
                                    throws SQLException {
                                    final Class<?> resultType = resultMap.getType();//获取ResultMap中指定的java类型
                                    //使用MetaClass包装返回类型
                                    final MetaClass metaType = MetaClass.forClass(resultType, reflectorFactory);
                                    //获取ResultMap构造方法标签
                                    final List<ResultMapping> constructorMappings = resultMap.getConstructorResultMappings();
                                    if (hasTypeHandlerForResultObject(rsw, resultType)) {
                                        //含有指定的TypeHandler创建对象
                                        return createPrimitiveResultObject(rsw, resultMap, columnPrefix);
                                    } else if (!constructorMappings.isEmpty()) {
                                        //使用构造方法创建对象
                                        return createParameterizedResultObject(rsw, resultType, constructorMappings, constructorArgTypes, constructorArgs, columnPrefix);
                                    } else if (resultType.isInterface() || metaType.hasDefaultConstructor()) {
                                        //使用默认的构造方法创建
                                        return objectFactory.create(resultType);
                                    } else if (shouldApplyAutomaticMappings(resultMap, false)) {
                                        return createByConstructorSignature(rsw, resultType, constructorArgTypes, constructorArgs);
                                    }
                                    throw new ExecutorException("Do not know how to create an instance of " + resultType);
                                }
                                ```
                            
                                - 使用指定的构造方法创建对象(来自DefaultObjectFactory)
                            
                                  ```java
                                  @Override
                                    public <T> T create(Class<T> type) {
                                      return create(type, null, null);
                                    }
                                  
                                  public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
                                      Class<?> classToCreate = resolveInterface(type);//获取需要创建的对象的Class
                                      //通过反射创建对象，根据传入的构造方法创建对象（构造方法为空则默认采用无参构造方法）
                                      return (T) instantiateClass(classToCreate, constructorArgTypes, constructorArgs);
                                    }
                                  
                                  //resolveInterface方法
                                  protected Class<?> resolveInterface(Class<?> type) {
                                      Class<?> classToCreate;
                                      //如果是list等集合类则直接创建ArrayList
                                      if (type == List.class || type == Collection.class || type == Iterable.class) {
                                        classToCreate = ArrayList.class;
                                      } else if (type == Map.class) {//如果是map集合则创建HashMap
                                        classToCreate = HashMap.class;
                                      } else if (type == SortedSet.class) { // 如果是SortSet则创建TreeSet
                                        classToCreate = TreeSet.class;
                                      } else if (type == Set.class) {//如果是Set集合则创建HashSet
                                        classToCreate = HashSet.class;
                                      } else {//否则直接创建该对象
                                        classToCreate = type;
                                      }
                                      return classToCreate;
                                    }
                                  ```
                            
                            - applyPropertyMappings方法
                            
                              ```java
                              private boolean applyPropertyMappings(ResultSetWrapper rsw, ResultMap resultMap, MetaObject metaObject, ResultLoaderMap lazyLoader, String columnPrefix)
                                  throws SQLException {
                                  //获取ResultMap中的行名称
                                  final List<String> mappedColumnNames = rsw.getMappedColumnNames(resultMap, columnPrefix);
                                  boolean foundValues = false;
                                  final List<ResultMapping> propertyMappings = resultMap.getPropertyResultMappings();
                                  for (ResultMapping propertyMapping : propertyMappings) {
                                      //拼接列前缀和列名称
                                      String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);
                                      if (propertyMapping.getNestedResultMapId() != null) {//如果采用列加载则不初始化列名
                                          // the user added a column attribute to a nested result map, ignore it
                                          column = null;
                                      }
                                      if (propertyMapping.isCompositeResult()//ResultMapp是复合结果
                                          //mappedColumnNames中包含有拼接后的列名
                                          || (column != null && mappedColumnNames.contains(column.toUpperCase(Locale.ENGLISH)))
                                          //含有结果集
                                          || propertyMapping.getResultSet() != null) {
                                          Object value = getPropertyMappingValue(rsw.getResultSet(), metaObject, propertyMapping, lazyLoader, columnPrefix);//获取结果值
                                          // issue #541 make property optional
                                          //获取在类中的名字
                                          final String property = propertyMapping.getProperty();
                                          if (property == null) {
                                              continue;
                                          } else if (value == DEFERRED) {
                                              foundValues = true;
                                              continue;
                                          }
                                          if (value != null) {
                                              foundValues = true;
                                          }
                                          if (value != null || (configuration.isCallSettersOnNulls() && !metaObject.getSetterType(property).isPrimitive())) {//如果值不为空且含有set方法且是基本类型
                                              // gcode issue #377, call setter on nulls (value is not 'found')
                                              metaObject.setValue(property, value);//调用SET方法
                                          }
                                      }
                                  }
                                  return foundValues;
                              }
                              ```
                            
                              - getPropertyMappingValue方法
                            
                                ```java
                                private Object getPropertyMappingValue(ResultSet rs, MetaObject metaResultObject, ResultMapping propertyMapping, ResultLoaderMap lazyLoader, String columnPrefix)
                                    throws SQLException {
                                    if (propertyMapping.getNestedQueryId() != null) {//是否懒加载
                                        return getNestedQueryMappingValue(rs, metaResultObject, propertyMapping, lazyLoader, columnPrefix);
                                    } else if (propertyMapping.getResultSet() != null) {
                                        addPendingChildRelation(rs, metaResultObject, propertyMapping);   // TODO is that OK?
                                        return DEFERRED;
                                    } else {
                                        final TypeHandler<?> typeHandler = propertyMapping.getTypeHandler();
                                        final String column = prependPrefix(propertyMapping.getColumn(), columnPrefix);
                                        return typeHandler.getResult(rs, column);
                                    }
                                }
                                ```
                            
                                - getResult方法
                            
                                  ```java
                                  public T getResult(ResultSet rs, String columnName) throws SQLException {
                                      try {
                                          //通过列名获取属性值
                                          return getNullableResult(rs, columnName);
                                      } catch (Exception e) {
                                          throw new ResultMapException("Error attempting to get column '" + columnName + "' from result set.  Cause: " + e, e);
                                      }
                                  }
                                  ```
                            
                                  
                  
                  - ensureNoOutParams
                  
                  
                  ```java
                    private void ensureNoOutParams(MappedStatement ms, BoundSql boundSql) {//确保没有输出参数
                      if (ms.getStatementType() == StatementType.CALLABLE) {//StatementType是个枚举类，除了CALLABLE（可调用的）以外还有PREPARED和STATEMENT
                        for (ParameterMapping parameterMapping : boundSql.getParameterMappings()) {//遍历每个参数
                          if (parameterMapping.getMode() != ParameterMode.IN) {//确保参数模式都为in，如果参数的 mode 为 OUT 或 INOUT，将会修改参数对象的属性值，以便作为输出参数返回
                            throw new ExecutorException("Caching stored procedures with OUT params is not supported.  Please configure useCache=false in " + ms.getId() + " statement.");
                          }
                        }
                      }
                    }
                  ```
                  
                  
            
            - cachedInvoker方法
            
            
            ```java
            
            private final Map<Method, MapperMethodInvoker> methodCache;
            
            private MapperMethodInvoker cachedInvoker(Method method) throws Throwable {//获取MapperMethodInvoker类（用于执行代理对象的方法）
                try {
                  return methodCache.computeIfAbsent(method, m -> {
                    if (m.isDefault()) {//判断该方法是否是default修饰，Default方法在接口中可以设置方法体，如果式deault则最终返回的类型是PlainMethodInvoker
                        //如果方法不是default类型则最终返回是DefaultMethodInvoker，这两种MethodInvoker都是MapperProxy中的内部类
                      try {
                        if (privateLookupInMethod == null) {//看不懂
                          return new DefaultMethodInvoker(getMethodHandleJava8(method));
                        } else {
                          return new DefaultMethodInvoker(getMethodHandleJava9(method));
                        }
                      } catch (IllegalAccessException | InstantiationException | InvocationTargetException
                          | NoSuchMethodException e) {
                        throw new RuntimeException(e);
                      }
                    } else {
                      return new PlainMethodInvoker(new MapperMethod(mapperInterface, method, sqlSession.getConfiguration()));//PlainMethodInvoker构造方法只是将内部的MapperMethod进行赋值，重点在MapperMethod的构造方法上
                    }
                  });
                } catch (RuntimeException re) {
                  Throwable cause = re.getCause();
                  throw cause == null ? re : cause;
                }
              }
            
            ```
            
            - computeIfAbsent方法
            
              - computeIfAbsent是JDK中自带的方法
            
                ```java
                 default V computeIfAbsent(K key,
                            Function<? super K, ? extends V> mappingFunction) {//mappingFunction是一个函数式接口
                        Objects.requireNonNull(mappingFunction);
                        V v;
                        if ((v = get(key)) == null) {//判断Map集合中是否存在该键，如果存在则返回其值
                            V newValue;
                            if ((newValue = mappingFunction.apply(key)) != null) {//如果不存在则通过mappingFunction创建该值并将其放入Map中并返回新值
                                put(key, newValue);
                                return newValue;
                            }
                        }
                
                        return v;
                    }
                ```
            
            - MapperMethod构造方法
            
              ```java
               public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
                  this.command = new SqlCommand(config, mapperInterface, method);
                  this.method = new MethodSignature(config, mapperInterface, method);
                }
              ```
            
              - MethodSignature构造方法
            
                ```java
                  public static class MethodSignature {
                
                    private final boolean returnsMany;
                    private final boolean returnsMap;
                    private final boolean returnsVoid;
                    private final boolean returnsCursor;
                    private final boolean returnsOptional;
                    private final Class<?> returnType;
                    private final String mapKey;
                    private final Integer resultHandlerIndex;
                    private final Integer rowBoundsIndex;
                    private final ParamNameResolver paramNameResolver;
                
                    
                
                    public MethodSignature(Configuration configuration, Class<?> mapperInterface, Method method) {
                      Type resolvedReturnType = TypeParameterResolver.resolveReturnType(method, mapperInterface);//如果返回类型是List<User>,则这里的Type为ParameterizedType
                      if (resolvedReturnType instanceof Class<?>) {
                        this.returnType = (Class<?>) resolvedReturnType;
                      } else if (resolvedReturnType instanceof ParameterizedType) {
                        this.returnType = (Class<?>) ((ParameterizedType) resolvedReturnType).getRawType();//更据以上假设returnType最终为List
                      } else {
                        this.returnType = method.getReturnType();
                      }
                      this.returnsVoid = void.class.equals(this.returnType);//是否返回空
                        //判断返回类型是否是集合或者数组
                      this.returnsMany = configuration.getObjectFactory().isCollection(this.returnType) || this.returnType.isArray();
                       // 当查询出大量数据时将这些数据包装为一个Cursor类通过迭代器访问（只能访问一次，一旦访问完所有元素，Cursor会自动销毁所有数据）
                      this.returnsCursor = Cursor.class.equals(this.returnType);
                        //Optional具体使用可看，https://blog.csdn.net/qq_34845394/article/details/100532507，该类主要作用是数据校验，比如查询某用户如果需要对结果为空的可能做出响应，可以采用返回Option简化开发
                      this.returnsOptional = Optional.class.equals(this.returnType);
                      this.mapKey = getMapKey(method);
                      this.returnsMap = this.mapKey != null;//判断是否返回Map类型
                        //RowBounds是mybatis提供的一个分页类，如果查询结果需要分页则将该类作为参数传入，以下这条语句的作用是找出RowBounds在第几个参数
                      this.rowBoundsIndex = getUniqueParamIndex(method, RowBounds.class);
                        //ResultHandler是一个结果处理器，在执行完SQL语句后执行，将SQL语句得到的结果进行自定义处理后返回，具体用法可看以下链接
                        //https://www.jb51.net/article/236321.htm
                        //下面该语句同样是获取ResultHandler在参数中的位置
                      this.resultHandlerIndex = getUniqueParamIndex(method, ResultHandler.class);
                        //参数名解析
                      this.paramNameResolver = new ParamNameResolver(configuration, method);
                    }
                      
                      private String getMapKey(Method method) {
                      String mapKey = null;
                          //isAssignableFrom用于判断method.getReturnType()是否是Map的子类或者实现
                      if (Map.class.isAssignableFrom(method.getReturnType())) {
                          //mapKey注解，mapkey放在mapper中的方法上用于表示该方法返回的是Map集合，该注解有一个属性用于表示每条记录的主键
                          //案例，其中的泛型可自定义，这里假设主键类型是Long，返回的Map集合中key为主键值，value为每条记录的字段名和对应的值
                          /*
                          @MapKey("id")
                    	public Map<Long, Map<String,String>> selectByMap();
                    		*/
                        final MapKey mapKeyAnnotation = method.getAnnotation(mapKey.class);
                        if (mapKeyAnnotation != null) {
                          mapKey = mapKeyAnnotation.value();
                        }
                      }
                      return mapKey;
                    }
                ```
            
                - 参数名解析器构造方法（ParamNameResolver）
            
                  ```java
                   public ParamNameResolver(Configuration config, Method method) {
                       //获取所有参数的类型
                      final Class<?>[] paramTypes = method.getParameterTypes();
                       //获取所有参数上的注解，一个参数上可以有多个注解，该数组第一维下标既是参数的序号如good(@a() int aw,@b() int bw)则@a在[0][0]，@b在[1][0]
                      final Annotation[][] paramAnnotations = method.getParameterAnnotations();
                      final SortedMap<Integer, String> map = new TreeMap<>();
                       //参数个数
                      int paramCount = paramAnnotations.length;
                      // get names from @Param annotations
                      for (int paramIndex = 0; paramIndex < paramCount; paramIndex++) {
                        if (isSpecialParameter(paramTypes[paramIndex])) {//判断该参数是否是特殊参数，也就是上文中的RowBounds和ResultHandler
                          // skip special parameters
                          continue;
                        }
                        String name = null;
                        for (Annotation annotation : paramAnnotations[paramIndex]) {
                          if (annotation instanceof Param) {//是否是Param注解的子类
                            hasParamAnnotation = true;
                            name = ((Param) annotation).value();//修改名称
                            break;
                          }
                        }
                        if (name == null) {
                          // 代表没有Param注解
                          if (config.isUseActualParamName()) {//查看配置文件中是否指定参数名称默认采用方法上的参数名称
                            name = getActualParamName(method, paramIndex);//通过反射和stream流获取
                          }
                          if (name == null) {
                            // use the parameter index as the name ("0", "1", ...)
                            // gcode issue #71
                              //如果配置文件中没有指定参数名称为方法上的参数名称则采用顺序编号如selectById(String id,String name),id在sql语句中用#{0},name在语句中用#{1}
                            name = String.valueOf(map.size());
                          }
                        }
                          //将名称放入Map中
                        map.put(paramIndex, name);
                      }
                       //unmodifiableSortedMap返回一个不可变的Map集合
                      names = Collections.unmodifiableSortedMap(map);
                    }
                  ```
            
                  
            
                - resolveReturnType方法
            
                  ```java
                  public static Type resolveReturnType(Method method, Type srcType) {//srcType是Mapper接口类型,这里我们假设以下参数
                      //method为public List<User> selectAll()
                      //srcType为public interface UserDao
                      Type returnType = method.getGenericReturnType();//returnType为List<User> 
                      Class<?> declaringClass = method.getDeclaringClass();//declaringClass为UserDao
                      return resolveType(returnType, srcType, declaringClass);
                    }
                  ```
            
                  - resolveType方法
            
                    ```java
                    private static Type resolveType(Type type, Type srcType, Class<?> declaringClass) {
                        if (type instanceof TypeVariable) {//根据上面的假设这里的type是 List<User> ，srcType为UserDao，declaringClass是UserDao.class
                          return resolveTypeVar((TypeVariable<?>) type, srcType, declaringClass);
                        } else if (type instanceof ParameterizedType) {//List<User>的Type属于ParameterizedType
                          return resolveParameterizedType((ParameterizedType) type, srcType, declaringClass);
                        } else if (type instanceof GenericArrayType) {//GenericArrayType数组泛型
                          return resolveGenericArrayType((GenericArrayType) type, srcType, declaringClass);
                        } else {
                          return type;
                        }
                      }
                    ```
            
                    - type类
            
                      - ParameterizedType类
            
                        ```java
                        public interface ParameterizedType extends Type {
                            /**
                             获取参数化类型的实际类型列表  Map<String,Integer>就是String Integer
                             */
                            Type[] getActualTypeArguments();
                        
                            /**
                              参数化类型中的原始类型，例子：List<String> -->List
                             */
                            Type getRawType();
                        
                            /**
                            返回类型所属的类型
                             */
                            Type getOwnerType();
                        }
                        
                        ```
            
                      - TypeVariable类
            
                        ```java
                        public interface TypeVariable<D extends GenericDeclaration> extends Type, AnnotatedElement {
                            /**
                             获取类型的上边界，默认Object List<T extends User> -->Use是上边界
                            */
                            Type[] getBounds();
                        
                            /**
                              获取声明该类型的原始类型，class Test<T extends User> -->Test
                             */
                            D getGenericDeclaration();
                        
                            /**
                             湖片区源码中定义的名字 T
                             */
                            String getName();
                             AnnotatedType[] getAnnotatedBounds();
                        }
                        ```
            
                        
            
                    - resolveParameterizedType方法
            
                      ```java
                      private static ParameterizedType resolveParameterizedType(ParameterizedType parameterizedType, Type srcType, Class<?> declaringClass) {
                          Class<?> rawType = (Class<?>) parameterizedType.getRawType();//同样的假设，rawType是List.class
                          Type[] typeArgs = parameterizedType.getActualTypeArguments();//typeArgs是User
                          Type[] args = new Type[typeArgs.length];
                          for (int i = 0; i < typeArgs.length; i++) {//对分离过后的类型再次进行处理并将其存入args中
                            if (typeArgs[i] instanceof TypeVariable) {//判断List<T>中的T是否属于TypeVariable
                              args[i] = resolveTypeVar((TypeVariable<?>) typeArgs[i], srcType, declaringClass);
                            } else if (typeArgs[i] instanceof ParameterizedType) {//判断List<T>中的T是否属于ParameterizedType
                              args[i] = resolveParameterizedType((ParameterizedType) typeArgs[i], srcType, declaringClass);
                            } else if (typeArgs[i] instanceof WildcardType) {
                              args[i] = resolveWildcardType((WildcardType) typeArgs[i], srcType, declaringClass);
                            } else {//按照假设最终User属于普通类型，这里为true
                              args[i] = typeArgs[i];
                            }
                          }
                          return new ParameterizedTypeImpl(rawType, null, args);
                        }
                      ```
            
                      - ParameterizedTypeImpl构造方法
            
                      ```java
                      //ParameterizedType中rawType、ownerType、actualTypeArguments都是其成员变量
                      public ParameterizedTypeImpl(Class<?> rawType, Type ownerType, Type[] actualTypeArguments) {
                          super();
                          this.rawType = rawType;//rawType为List
                          this.ownerType = ownerType;//ownerType为空
                          this.actualTypeArguments = actualTypeArguments;//actualTypeArguments为Use
                      }
                      ```
            
              - SqlCommand类
            
                ```java
                public static class SqlCommand {
                
                    private final String name;
                    private final SqlCommandType type;
                
                    public SqlCommand(Configuration configuration, Class<?> mapperInterface, Method method) {
                      final String methodName = method.getName();//获取方法名称
                      final Class<?> declaringClass = method.getDeclaringClass();//获取方法调用类
                      MappedStatement ms = resolveMappedStatement(mapperInterface, methodName, declaringClass,
                          configuration);//获取MappedStatement
                      if (ms == null) {//未找到MappedStatement
                        if (method.getAnnotation(Flush.class) != null) {//如果存在Flush注解
                          name = null;
                          type = SqlCommandType.FLUSH;//设置SQL语句类型为FLUSH
                        } else {
                          throw new BindingException("Invalid bound statement (not found): "
                              + mapperInterface.getName() + "." + methodName);
                        }
                      } else {
                        name = ms.getId();
                        type = ms.getSqlCommandType();
                        if (type == SqlCommandType.UNKNOWN) {
                          throw new BindingException("Unknown execution method for: " + name);
                        }
                      }
                    }
                ```
            
                - 通过resolveMappedStatement获取MappedStatement（MappedStatement类是Mapper.xml文件的封装类）
                
                  ```java
                    private MappedStatement resolveMappedStatement(Class<?> mapperInterface, String methodName,
                          Class<?> declaringClass, Configuration configuration) {
                        String statementId = mapperInterface.getName() + "." + methodName;
                        //statementId只是通过简单的拼接，为接口全路径名称+方法名称而Mapper进行注册时的接口可能是当前参数（mapperInterface）的父类所以采用递归查找该方法
                        if (configuration.hasStatement(statementId)) {
                          return configuration.getMappedStatement(statementId);
                        } else if (mapperInterface.equals(declaringClass)) {//如果方法调用者是Mapper接口本身则返回空
                          return null;
                        }
                        for (Class<?> superInterface : mapperInterface.getInterfaces()) {
                          if (declaringClass.isAssignableFrom(superInterface)) {//查看是否方法调用者实现了父类的接口
                            MappedStatement ms = resolveMappedStatement(superInterface, methodName,
                                declaringClass, configuration);//若实现了父类的接口则进入递归将父类作为参数传入直到可以从configuration中找到该方法
                            if (ms != null) {
                              return ms;
                            }
                          }
                        }
                        return null;
                      }
                  ```
                
                  - hasStatement方法
                  
                    ```java
                    //MappedStatement是SQL的包装类
                    protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection")//StrictMap继承了HashMap,与HashMap不同的是其中包含了一个BiFuction接口，BiFuction接口既是传入两个不同（也可以相同）的类型变量然后进行处理最后得到另一个可以不同类型的变量
                        //BiFunction<T, U, R>  （T,U）->R，在这里T和U都是MappedStatement类型，R是String类型
                        .conflictMessageProducer((savedValue, targetValue) ->//使用BiFuction描述错误信息
                                                 ". please check " + savedValue.getResource() + " and " + targetValue.getResource());
                    
                    public boolean hasStatement(String statementName) {//首先调用重载方法
                        return hasStatement(statementName, true);
                      }
                    
                     public boolean hasStatement(String statementName, boolean validateIncompleteStatements) {
                         //validateIncompleteStatements指出是否需要检查所有语句、Cache、Method都已解析完成
                        if (validateIncompleteStatements) {
                          buildAllStatements();//校验
                        }
                        return mappedStatements.containsKey(statementName);
                      }
                    ```
                  
                    - buildAllStatements方法
                  
                      ```java
                       
                      protected final Collection<CacheRefResolver> incompleteCacheRefs = new LinkedList<>();
                      
                      //MethodResolver中包含了MapperAnnotationBuilder和mehtod，而MapperAnnotationBuilder中包含了Configruation对象
                      //MethodResolver主要方法就是调用incompleteMethods的parseStatement方法，主要用于解析sql语句
                      //在初始化mapper的过程中就已经调用了parseStatement方法并将incompleteMethods中解析过的方法全部清楚了，所以buildAllStatements方法只是做一个保险
                      //除了Methods以外buildAllStatements其他的方法也都是这样
                        protected final Collection<MethodResolver> incompleteMethods = new LinkedList<>();
                        protected final Collection<XMLStatementBuilder> incompleteStatements = new LinkedList<>();
                      protected void buildAllStatements() {
                          parsePendingResultMaps();//加载ResultMap
                          if (!incompleteCacheRefs.isEmpty()) {//如果还有没有解析的Cache引用则解析该引用并删除incompleteCacheRefs中的数据
                            synchronized (incompleteCacheRefs) {
                              incompleteCacheRefs.removeIf(x -> x.resolveCacheRef() != null);
                            }
                          }
                          if (!incompleteStatements.isEmpty()) {//如果还有没有解析的SQL语句则解析该引用并删除incompleteStatements中的数据
                            synchronized (incompleteStatements) {
                              incompleteStatements.removeIf(x -> {
                                x.parseStatementNode();
                                return true;
                              });
                            }
                          }
                          if (!incompleteMethods.isEmpty()) {//如果还有没有解析的方法则解析该方法并删除incompleteMethods中的数据
                            synchronized (incompleteMethods) {
                              incompleteMethods.removeIf(x -> {
                                x.resolve();
                                return true;
                              });
                            }
                          }
                        }
                      ```
                      
                      - parsePendingResultMaps方法
                      
                        ```java
                         //ResultMapResolver是MapperBuilderAssistant的包装类，主要方法就是调用MapperBuilderAssistant的addResultMap方法
                        //MapperBuilderAssistant中含有Configuration类型，resource，namespace，cache成员变量
                        protected final Collection<ResultMapResolver> incompleteResultMaps = new LinkedList<>();
                        
                        private void parsePendingResultMaps() {
                            if (incompleteResultMaps.isEmpty()) {
                              return;
                            }
                            synchronized (incompleteResultMaps) {
                              boolean resolved;
                              IncompleteElementException ex = null;
                              do {
                                resolved = false;
                                Iterator<ResultMapResolver> iterator = incompleteResultMaps.iterator();
                                while (iterator.hasNext()) {
                                  try {
                                    iterator.next().resolve();//添加ResultMap
                                    iterator.remove();//添加完成后就剔除
                                    resolved = true;
                                  } catch (IncompleteElementException e) {
                                    ex = e;
                                  }
                                }
                              } while (resolved);//为了保证所有的ResultMap都被加载
                              if (!incompleteResultMaps.isEmpty() && ex != null) {
                                // At least one result map is unresolvable.
                                throw ex;
                              }
                            }
                          }
                        ```
                      
                        - ResultMapResolver的resolve方法
                      
                          ```java
                          public ResultMap resolve() {
                              return assistant.addResultMap(this.id, this.type, this.extend, this.discriminator, this.resultMappings, this.autoMapping);
                            }
                          //assistant的addResultMap方法
                          public ResultMap addResultMap(
                                String id,
                                Class<?> type,//这里的type是ResultMap的type
                                String extend,//extend是父类ResultMap的ID
                                Discriminator discriminator,
                                List<ResultMapping> resultMappings,//resultMappings是在mapper文件中的编写的result标签，不包含父类的result标签，只包含子类特有的
                                Boolean autoMapping) {
                              id = applyCurrentNamespace(id, false);//这里的ID指的是ResultMap的ID
                              extend = applyCurrentNamespace(extend, true);
                          
                              if (extend != null) {
                                if (!configuration.hasResultMap(extend)) {
                                  throw new IncompleteElementException("Could not find a parent resultmap with id '" + extend + "'");
                                }
                                ResultMap resultMap = configuration.getResultMap(extend);//获取父类的ResultMap
                                List<ResultMapping> extendedResultMappings = new ArrayList<>(resultMap.getResultMappings());//获取父ResultMap中的每一行
                                extendedResultMappings.removeAll(resultMappings);//单纯的对子类中重复定义的result标签进行去重
                                // 删除父类的构造器
                                boolean declaresConstructor = false;
                                for (ResultMapping resultMapping : resultMappings) {
                                  if (resultMapping.getFlags().contains(ResultFlag.CONSTRUCTOR)) {//getFlags方法:在ResultMapping中含有
                                    declaresConstructor = true;//只要存在构造器就退出并将declaresConstructor赋值为true以便删除
                                    break;
                                  }
                                }
                                if (declaresConstructor) {//删除父类构造器
                                  extendedResultMappings.removeIf(resultMapping -> resultMapping.getFlags().contains(ResultFlag.CONSTRUCTOR));
                                }
                                resultMappings.addAll(extendedResultMappings);//重新添加父类中的ResultMap中的每一行
                              }
                              ResultMap resultMap = new ResultMap.Builder(configuration, id, type, resultMappings, autoMapping)//autoMapping是否自动映射，该build方法也只是简单赋值
                                  .discriminator(discriminator)//鉴别器是可以在ResultMap中配置的属性，discriminator的作用类似于Swith case，可以根据数据的不同而采用不同的映射
                                  .build();
                              configuration.addResultMap(resultMap);//添加该resultMap
                              return resultMap;
                            }
                          ```
                          
                  
            
    
  - ## Mybatis插件扩展开发
  
    - 在mybatis中提供了一个plugin系列的类用于增强其他类
  
    - 可以在mybatis配置文件中添加plugin标签指定拦截器
  
    - 分析插件执行原理
  
      - 入口(拦截器链类)
  
        ```java
        public class InterceptorChain {
        
          private final List<Interceptor> interceptors = new ArrayList<>();//存放所有拦截器
        
            //pluginAll执行的地方一共有4个
            //ParameterHandler、ResultSetHandler、StatementHandler、Executor
            //也就是说可以在参数解析时，结果返回时，语句解析时，语句执行时对方法进行加强
          public Object pluginAll(Object target) {//target是被拦截的类
            for (Interceptor interceptor : interceptors) {
              target = interceptor.plugin(target);//使用所有拦截器对该类的方法进行加强
            }
            return target; //返回被加强后的类
          }
        
          public void addInterceptor(Interceptor interceptor) {
            interceptors.add(interceptor);
          }
        
          public List<Interceptor> getInterceptors() {
            return Collections.unmodifiableList(interceptors);
          }
        
        }
        
        
        ```
  
        - 拦截器接口
  
          ```java
          public interface Interceptor {
          
            Object intercept(Invocation invocation) throws Throwable;
          
            default Object plugin(Object target) {
              return Plugin.wrap(target, this);//加强被拦截类
            }
          
            default void setProperties(Properties properties) {
              // NOP
            }
          
          }
          ```
  
          - 插件类的wrap方法
  
            ```java
            public static Object wrap(Object target, Interceptor interceptor) {
                Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);//获取拦截器所拦截方法的方法签名
                Class<?> type = target.getClass();//获取被拦截类的Class
                //获取被拦截类所继承的接口且这些接口也在signatureMap中（即这些接口也需要被加强）
                Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
                if (interfaces.length > 0) {
                  return Proxy.newProxyInstance(//动态代理加强被拦截类
                      type.getClassLoader(),
                      interfaces,
                      new Plugin(target, interceptor, signatureMap));
                }
                return target;//返回被拦截类
              }
            ```
  
            - 动态代理的invoke方法
  
              ```java
              @Override
              public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                  try {
                      Set<Method> methods = signatureMap.get(method.getDeclaringClass());//获取被加强方法
                      if (methods != null && methods.contains(method)) {
                          return interceptor.intercept(new Invocation(target, method, args));//执行加强
                      }
                      return method.invoke(target, args);//如果该方法不需要加强则直接执行
                  } catch (Exception e) {
                      throw ExceptionUtil.unwrapThrowable(e);
                  }
              }
              ```
  
              - Invocation类
  
                ```java
                public class Invocation {//被拦截方法的封装
                
                  private final Object target;
                  private final Method method;
                  private final Object[] args;
                
                  public Invocation(Object target, Method method, Object[] args) {
                    this.target = target;
                    this.method = method;
                    this.args = args;
                  }
                //省略get方法
                  public Object proceed() throws InvocationTargetException, IllegalAccessException {
                    return method.invoke(target, args);//被拦截方法的调用
                  }
                
                }
                ```
  
              - intercept方法
  
                ```java
                 */
                public interface Interceptor {
                
                  Object intercept(Invocation invocation) throws Throwable;//等待被实现的方法，这也是我们自己需要实现的方法，通过这个方法实现动态代理
                
                
                }
                ```
  
                
  
            - plugin的getSignatureMap方法
  
              ```java
              private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
                  //获取拦截器类上的拦截器注解，这也代表了我们开发的插件上需要添加拦截器注解
                  Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
                  // issue #251
                  if (interceptsAnnotation == null) {
                      throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());
                  }
                  Signature[] sigs = interceptsAnnotation.value();//需要被拦截的方法
                  Map<Class<?>, Set<Method>> signatureMap = new HashMap<>();//key为方法所属类，value为该类中的方法
                  for (Signature sig : sigs) {
                      //判断signatureMap是否存在类如果不存在则创建并返回该set<Method>
                      Set<Method> methods = signatureMap.computeIfAbsent(sig.type(), k -> new HashSet<>());
                      try {
                          Method method = sig.type().getMethod(sig.method(), sig.args());
                          methods.add(method);//将需要被拦截的方法添加至methods中
                      } catch (NoSuchMethodException e) {
                          throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + e, e);
                      }
                  }
                  return signatureMap;
              }
              ```
  
              - 拦截器注解
  
                ```java
                @Documented
                @Retention(RetentionPolicy.RUNTIME)
                @Target(ElementType.TYPE)
                public @interface Intercepts {
                  /**
                   * Returns method signatures to intercept.
                   *
                   * @return method signatures
                   */
                  Signature[] value();//被拦截的方法签名注解
                }
                //signature注解
                
                @Documented
                @Retention(RetentionPolicy.RUNTIME)
                @Target({})
                public @interface Signature {
                
                  Class<?> type();//方法所属类
                
                  String method();//方法名
                
                  Class<?>[] args();//方法参数
                }
                
                ```
  
    - 编写一个插件（只是在查询前简单输出一句话）
  
      - 插件代码
  
        ```java
        @Intercepts(
                {
                        @Signature(type= Executor.class,method = "query",args = {//对执行器的两个查询方法进行拦截
                                MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class
                        }),
                        @Signature(type= Executor.class,method = "query",args = {
                                MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class
                        }),
                }
        )
        public class MyPlugin implements Interceptor {
           private Properties properties;
        
        
            @Override
            public void setProperties(Properties properties) {
                this.properties=properties;//通过mybatis主配置文件中指定的properties进行赋值
            }
        
            @Override
            public Object intercept(Invocation invocation) throws Throwable {
                System.out.println(properties.getProperty("someProperty"));//输出properties中的值
                return invocation.proceed();//原方法的执行
            }
        }
        ```
  
        - 主配置文件(mybatis对于各个标签有顺序要求，plugins要在环境前)
  
        ```xml
         <plugins>
                <plugin interceptor="MyPlugins.MyPlugin">
                    <property name="someProperty" value="this is a Interceptor"/>
                </plugin>
            </plugins>
        ```
  
        
