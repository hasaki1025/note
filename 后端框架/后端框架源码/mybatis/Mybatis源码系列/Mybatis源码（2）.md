# Mybatis源码（2）

- 入口语句(通过mybatis主配置文件的IO流获取SqlSessionFactoryBuilder类)

  ```java
  SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();//直接调用无参构造方法
  SqlSessionFactory factory=builder.build(in)//通过IO流和SqlSessionFactoryBuilder
  ```

- SqlSessionFactoryBuilder类

  ```java
  public class SqlSessionFactoryBuilder {//用于创建SqlSessionFactory的工具类
  
      //properties是JDK中对properties文件的包装类，environment是Mybatis环境标签的名称,Reader是主配置文件的reader,reader也可以用InputStream代替
    public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
      try {
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);//通过build中的参数创建XMLConfigBuilder类
        return build(parser.parse());//parser.parse()返回一个Configuration类，该类是Mybatis主配置文件的包装类
      } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
      } finally {
        ErrorContext.instance().reset();//重新设置错误上下文
        try {
          inputStream.close();
        } catch (IOException e) {
          // Intentionally ignore. Prefer previous error.
        }
      }
    }
  
  
    public SqlSessionFactory build(Configuration config) {
      return new DefaultSqlSessionFactory(config);
    }
  
  }
  ```

- XMLConfigBuilder类

  ```java
  public class XMLConfigBuilder extends BaseBuilder {
  
      private boolean parsed;//是否已解析该XML文件
      private final XPathParser parser;//XML解析器
      private String environment;//环境名称
      private final ReflectorFactory localReflectorFactory = new DefaultReflectorFactory();//反射工厂
      
      
    public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
      this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);//XMLMapperEntityResolver是指定本地DTD文件的类
    }
  
  ```

  - XPathParser类

    ```java
    public class XPathParser {
    
      private final Document document;//XML文件的包装类，通过XPathFactory创建
      private boolean validation;//是否进行校验
      private EntityResolver entityResolver;//DTD文件解析器
      private Properties variables;//属性配置
      private XPath xpath;//主要XML解析类
    
    }
    ```

    - XPathParser的构造方法

      ```java
      public XPathParser(InputStream inputStream, boolean validation, Properties variables, EntityResolver entityResolver) {
          commonConstructor(validation, variables, entityResolver);//初始化成员变量
          this.document = createDocument(new InputSource(inputStream));//通过IO流创建document对象
        }
      ```
  
      - commonConstructor方法
  
        ```java
        private void commonConstructor(boolean validation, Properties variables, EntityResolver entityResolver) {
            this.validation = validation;
            this.entityResolver = entityResolver;
            this.variables = variables;
            XPathFactory factory = XPathFactory.newInstance();
            this.xpath = factory.newXPath();
          }
        ```
  
        
  
  - 父类BaseBuilder类
  
    ```java
    public abstract class BaseBuilder {//只展示成员变量和构造方法
        protected final Configuration configuration;
        protected final TypeAliasRegistry typeAliasRegistry;//类型别名注册器，其中包含了一个Map<String, Class<?>> typeAliases，构造方法会向其中添加常用的别名
        protected final TypeHandlerRegistry typeHandlerRegistry;//类型处理器注册器，用于注册TypeHandler和其需要处理的类型，该类的构造方法会注册常用的类型处理器
    
    
        public BaseBuilder(Configuration configuration) {//构造方法
            this.configuration = configuration;
            this.typeAliasRegistry = this.configuration.getTypeAliasRegistry();
            this.typeHandlerRegistry = this.configuration.getTypeHandlerRegistry();
        }
    }
    ```
  
  
  - XMLConfigBuilder的parse方法
  
    ```java
    public Configuration parse() {
        if (parsed) {//是否解析过该XML文件
          throw new BuilderException("Each XMLConfigBuilder can only be used once.");
        }
        parsed = true;//设置为解析过
        parseConfiguration(parser.evalNode("/configuration"));//使用XPathParser先解析获取根标签,parseConfiguration才是最终的解析方法
        return configuration;
      }
    ```
  
    - XPathParser类的evalNode方法
  
      ```java
      public XNode evalNode(String expression) {
          return evalNode(document, expression);//expression是根标签名称
      }
      
      public XNode evalNode(Object root, String expression) {
          Node node = (Node) evaluate(expression, root, XPathConstants.NODE);//解析根标签并返回NODE类型
          if (node == null) {
              return null;
          }
          return new XNode(this, node, variables);//XNODE是node的包装类,这里一步暂时只是获取根标签
      }
      
      ```
  
      - XPathParser的evaluate方法
  
        ```java
        private Object evaluate(String expression, Object root, QName returnType) {
            try {
                return xpath.evaluate(expression, root, returnType);//使用XPath解析并返回Node对象
            } catch (Exception e) {
                throw new BuilderException("Error evaluating XPath.  Cause: " + e, e);
            }
        }
        ```
  
      - XNode对象
  
        ````java
        public class XNode {
        
          private final Node node;
          private final String name;//名称是标签名称
          private final String body;
          private final Properties attributes;
          private final Properties variables;
          private final XPathParser xpathParser;
        
          public XNode(XPathParser xpathParser, Node node, Properties variables) {
            this.xpathParser = xpathParser;
            this.node = node;
            this.name = node.getNodeName();
            this.variables = variables;
            this.attributes = parseAttributes(node);
            this.body = parseBody(node);
          }
        }
        ````
  
        - parseAttributes方法
  
          ```java
          private Properties parseAttributes(Node n) {
              Properties attributes = new Properties();
              NamedNodeMap attributeNodes = n.getAttributes();//获取Node标签上的属性并返回一个包装类
              if (attributeNodes != null) {
                  //遍历attributeNodes并将属性放入attributes中
                  for (int i = 0; i < attributeNodes.getLength(); i++) {
                      Node attribute = attributeNodes.item(i);
                      String value = PropertyParser.parse(attribute.getNodeValue(), variables);//获取标签属性值
                      attributes.put(attribute.getNodeName(), value);//将属性名称和值放入attributes中
                  }
              }
              return attributes;
          }
          ```
  
        - parseBody方法
  
          ```java
          private String parseBody(Node node) {
              String data = getBodyData(node);//获取标签内部的值
              if (data == null) {
                  NodeList children = node.getChildNodes();//如果标签内部含有子标签则遍历获取子标签中的值
                  for (int i = 0; i < children.getLength(); i++) {
                      Node child = children.item(i);
                      data = getBodyData(child);
                      if (data != null) {
                          break;
                      }
                  }
              }
              return data;//返回最小的子标签的内容
          }
          ```
  
    - parseConfiguration方法
  
      ```java
      private void parseConfiguration(XNode root) {
          try {
              //issue #117 read properties first
              propertiesElement(root.evalNode("properties"));//解析properties标签
              //解析settings标签如果为空则返回一个空的Properties类
              Properties settings = settingsAsProperties(root.evalNode("settings"));
              //以下两个暂时不知道
              loadCustomVfs(settings);
              loadCustomLogImpl(settings);
              //类别名标签解析
              typeAliasesElement(root.evalNode("typeAliases"));
              //插件解析
              pluginElement(root.evalNode("plugins"));
              //objectFactory解析
              objectFactoryElement(root.evalNode("objectFactory"));
              //objectWrapperFactory解析
              objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
              //反射工厂解析
              reflectorFactoryElement(root.evalNode("reflectorFactory"));
              //settings标签解析
              settingsElement(settings);
              // read it after objectFactory and objectWrapperFactory issue #631
              //环境标签解析
              environmentsElement(root.evalNode("environments"));
              //解析databaseId标签（数据库厂商标签）
              databaseIdProviderElement(root.evalNode("databaseIdProvider"));
              //typeHandler标签解析
              typeHandlerElement(root.evalNode("typeHandlers"));
              //mapper文件存放位置解析
              mapperElement(root.evalNode("mappers"));
          } catch (Exception e) {
              throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
          }
      }
      ```
  
      - typeAliasesElement
  
        ```java
        private void typeAliasesElement(XNode parent) {
            if (parent != null) {//如果含有别名标签
                for (XNode child : parent.getChildren()) {//遍历子标签
                    if ("package".equals(child.getName())) {//package标签
                        //获取别名包标签上的name属性
                        String typeAliasPackage = child.getStringAttribute("name");
                        //在Configuration中注册该别名
                        configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);
                    } else {
                        String alias = child.getStringAttribute("alias");//获取别名类的别名
                        String type = child.getStringAttribute("type");//获取别名类的类型
                        try {
                            Class<?> clazz = Resources.classForName(type);//初始化该别名类并获取class
                            if (alias == null) {
                                typeAliasRegistry.registerAlias(clazz);//注册该类别名采用@Alias上的别名
                            } else {
                                typeAliasRegistry.registerAlias(alias, clazz);//注册该别名采用标签上的别名
                            }
                        } catch (ClassNotFoundException e) {
                            throw new BuilderException("Error registering typeAlias for '" + alias + "'. Cause: " + e, e);
                        }
                    }
                }
            }
        }
        ```
  
        - registerAlias
  
          ```java
           private final Map<String, Class<?>> typeAliases = new HashMap<>();
          
          public void registerAlias(Class<?> type) {
            String alias = type.getSimpleName();//获取该类的名称
            Alias aliasAnnotation = type.getAnnotation(Alias.class);//获取该类上的Alias注解
            if (aliasAnnotation != null) {
              alias = aliasAnnotation.value();//获取Alias指定的别名
            }
            registerAlias(alias, type);//注册该别名
          }
          
          
          ```
  
          ```java
          public void registerAlias(String alias, Class<?> value) {
            if (alias == null) {
              throw new TypeException("The parameter alias cannot be null");
            }
            // issue #748
            String key = alias.toLowerCase(Locale.ENGLISH);//转小写
            if (typeAliases.containsKey(key) && typeAliases.get(key) != null && !typeAliases.get(key).equals(value)) {//重复注册则抛出异常
              throw new TypeException("The alias '" + alias + "' is already mapped to the value '" + typeAliases.get(key).getName() + "'.");
            }
            typeAliases.put(key, value);//放入map集合中
          }
          ```
  
    - objectFactoryElement方法
  
      ```java
      /*不指定该标签默认采用DefaultObjectFactory类作为ObjectFactory
      <!-- mybatis-config.xml -->/
      <objectFactory type="org.mybatis.example.ExampleObjectFactory">
        <property name="someProperty" value="100"/>
      </objectFactory>
      */
      private void objectFactoryElement(XNode context) throws Exception {
          if (context != null) {
              String type = context.getStringAttribute("type");//获取ObejctFactory上的type属性
              Properties properties = context.getChildrenAsProperties();//获取子标签中的信息并以Properties返回
              ObjectFactory factory = (ObjectFactory) resolveClass(type).getDeclaredConstructor().newInstance();//初始化ObejctFactory
              factory.setProperties(properties);//设置ObjectFactory中的properties
              configuration.setObjectFactory(factory);//设置configuration中的ObjectFactory
          }
      }
      ```
  
      - XNode中的getChildrenAsproperties方法
  
        ```java
        public Properties getChildrenAsProperties() {
            Properties properties = new Properties();
            for (XNode child : getChildren()) {
                String name = child.getStringAttribute("name");//获取标签上的name属性
                String value = child.getStringAttribute("value");//获取value属性
                if (name != null && value != null) {
                    properties.setProperty(name, value);//放入properties中
                }
            }
            return properties;
        }
        ```
  
    - reflectFactoryElement方法
  
      ```java
      private void reflectorFactoryElement(XNode context) throws Exception {//默认采用DefaultReflectorFactory
          if (context != null) {
              String type = context.getStringAttribute("type");//获取type属性，用于指定反射工厂类
              ReflectorFactory factory = (ReflectorFactory) resolveClass(type).getDeclaredConstructor().newInstance();//反射得到ReflectorFactory
              configuration.setReflectorFactory(factory);//赋值
          }
      }
      ```
  
    - settingsElement方法
  
      ```java
      
      private void settingsElement(Properties props) {
          //省略
          configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE")));//都是这种赋值操作
          //省略
      }
      ```
  
    - typeHandlerElement方法
  
      ```java
       private void typeHandlerElement(XNode parent) {//typehandler相当于jdbc的类型和java的类型之间的转换器
          if (parent != null) {
            for (XNode child : parent.getChildren()) {
              if ("package".equals(child.getName())) {//如果采用package标签
                String typeHandlerPackage = child.getStringAttribute("name");//获取name属性,该属性代表typeHandler全路径包名
                typeHandlerRegistry.register(typeHandlerPackage);//注册typeHandler
              } else {//如果指定TypeHandler的全路径类名
                String javaTypeName = child.getStringAttribute("javaType");//可以处理的java类型
                String jdbcTypeName = child.getStringAttribute("jdbcType");//可以处理的JDBC类型
                String handlerTypeName = child.getStringAttribute("handler");//handler名称
                Class<?> javaTypeClass = resolveClass(javaTypeName);//得到java类型的class
                JdbcType jdbcType = resolveJdbcType(jdbcTypeName);
                Class<?> typeHandlerClass = resolveClass(handlerTypeName);
                if (javaTypeClass != null) {//
                  if (jdbcType == null) {//如果xml文件中没有指定JDBCtype则采用MappedJdbcTypes注解上的JDBCtype
                    typeHandlerRegistry.register(javaTypeClass, typeHandlerClass);
                  } else {//都指定的情况
                    typeHandlerRegistry.register(javaTypeClass, jdbcType, typeHandlerClass);
                  }
                } else {//如果没有指定javaType和JDBCType则采用MappedTypes注解和MappedJdbcTypes注解上的信息
                  typeHandlerRegistry.register(typeHandlerClass);
                }
              }
            }
          }
        }
      ```
  
      - typeHandlerRegistry的register方法（通过类名注册,第三种情况）
  
        ```java
          private final Map<Type, Map<JdbcType, TypeHandler<?>>> typeHandlerMap = new ConcurrentHashMap<>();
        
        public void register(Class<?> javaTypeClass, JdbcType jdbcType, Class<?> typeHandlerClass) {
            register(javaTypeClass, jdbcType, getInstance(javaTypeClass, typeHandlerClass));
          }
        
        public <T> void register(Class<T> type, JdbcType jdbcType, TypeHandler<? extends T> handler) {
            register((Type) type, jdbcType, handler);
          }
        
        private void register(Type javaType, JdbcType jdbcType, TypeHandler<?> handler) {//将TypeHandler与JavaType、JDBCType的关系放入typeHandlerMap
            if (javaType != null) {
                Map<JdbcType, TypeHandler<?>> map = typeHandlerMap.get(javaType);
                if (map == null || map == NULL_TYPE_HANDLER_MAP) {
                    map = new HashMap<>();
                }
                map.put(jdbcType, handler);
                typeHandlerMap.put(javaType, map);
            }
            allTypeHandlersMap.put(handler.getClass(), handler);
        }
        ```
  
        - getInstance方法
  
          ```java
          public <T> TypeHandler<T> getInstance(Class<?> javaTypeClass, Class<?> typeHandlerClass) {
              if (javaTypeClass != null) {//javaTypeClass则采用有参构造方法且参数是javaTypeClass的class，如果没有该构造方法则采用无参构造方法
                  try {
                      Constructor<?> c = typeHandlerClass.getConstructor(Class.class);//
                      return (TypeHandler<T>) c.newInstance(javaTypeClass);//通过无参构造方法创建对象并返回TypeHandler类型
                  } catch (NoSuchMethodException ignored) {
                      // ignored
                  } catch (Exception e) {
                      throw new TypeException("Failed invoking constructor for handler " + typeHandlerClass, e);
                  }
              }
              try {
                  Constructor<?> c = typeHandlerClass.getConstructor();//javaTypeClass为空则采用无参构造方法
                  return (TypeHandler<T>) c.newInstance();
              } catch (Exception e) {
                  throw new TypeException("Unable to find a usable constructor for " + typeHandlerClass, e);
              }
          }
          ```
  
      - typeHandlerRegistry的register方法（通过包名注册）
  
        ```java
        public void register(String packageName) {
            ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
            resolverUtil.find(new ResolverUtil.IsA(TypeHandler.class), packageName);//判断主配置文件中的TypeHandler是否能转换为TypeHandler类
            Set<Class<? extends Class<?>>> handlerSet = resolverUtil.getClasses();//获取matches集合
            for (Class<?> type : handlerSet) {//handler不能是接口、抽象类、内部类
                //Ignore inner classes and interfaces (including package-info.java) and abstract classes
                if (!type.isAnonymousClass() && !type.isInterface() && !Modifier.isAbstract(type.getModifiers())) {
                    register(type);
                }
            }
        }
        ```
  
        - ResolverUtil类
  
          ```java
          public class ResolverUtil<T> {
          
              private static final Log log = LogFactory.getLog(ResolverUtil.class);
          
              private Set<Class<? extends T>> matches = new HashSet<>();
          
              public interface Test {
          
                  boolean matches(Class<?> type);
              }
          
              public static class IsA implements Test {
                  private Class<?> parent;
          
                  public IsA(Class<?> parentType) {
                      this.parent = parentType;
                  }
          
                  @Override
                  public boolean matches(Class<?> type) {//判断传入的type是否可以转换为parent类
                      return type != null && parent.isAssignableFrom(type);
                  }
          
                  @Override
                  public String toString() {
                      return "is assignable to " + parent.getSimpleName();
                  }
              }
          ```
  
        - find方法
  
          ```java
          public ResolverUtil<T> find(Test test, String packageName) {
              String path = getPackagePath(packageName);
          
              try {
                  List<String> children = VFS.getInstance().list(path);//VFS（虚拟文件系统），在不同操作系统中获取IO流的一个类
                  for (String child : children) {
                      if (child.endsWith(".class")) {
                          addIfMatching(test, child);//判断是否匹配
                      }
                  }
              } catch (IOException ioe) {
                  log.error("Could not read package: " + packageName, ioe);
              }
          
              return this;
          }
          ```
  
        - addIfMatching方法
  
          ```java
          protected void addIfMatching(Test test, String fqn) {//fqn是全路径类名
              try {
                  String externalName = fqn.substring(0, fqn.indexOf('.')).replace('/', '.');
                  ClassLoader loader = getClassLoader();
                  if (log.isDebugEnabled()) {
                      log.debug("Checking to see if class " + externalName + " matches criteria [" + test + "]");
                  }
          
                  Class<?> type = loader.loadClass(externalName);
                  if (test.matches(type)) {//如果可以转换为TypeHandler则添加至matches中
                      matches.add((Class<T>) type);
                  }
              } catch (Throwable t) {
                  log.warn("Could not examine class '" + fqn + "'" + " due to a " +
                           t.getClass().getName() + " with message: " + t.getMessage());
              }
          }
          ```
  
    - mapperElement方法
  
      ```java
      private void mapperElement(XNode parent) throws Exception {
          if (parent != null) {
              for (XNode child : parent.getChildren()) {
                  if ("package".equals(child.getName())) {
                      String mapperPackage = child.getStringAttribute("name");//通过包路径指定mapper.xml
                      configuration.addMappers(mapperPackage);//将包下的所有mapper.xml文件全部解析
                  } else {
                      String resource = child.getStringAttribute("resource");//mapper文件指定
                      String url = child.getStringAttribute("url");//url属性指定（使用完全限定资源定位符指定mapper.xml文件）
                      String mapperClass = child.getStringAttribute("class");//class属性指定（使用映射器接口实现类的完全限定类名 ）
                      if (resource != null && url == null && mapperClass == null) {
                          ErrorContext.instance().resource(resource);
                          InputStream inputStream = Resources.getResourceAsStream(resource);//通过相对路径获取mapper.xml文件IO流
                          XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
                          mapperParser.parse();
                      } else if (resource == null && url != null && mapperClass == null) {
                          ErrorContext.instance().resource(url);
                          InputStream inputStream = Resources.getUrlAsStream(url);
                          XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
                          mapperParser.parse();
                      } else if (resource == null && url == null && mapperClass != null) {
                          Class<?> mapperInterface = Resources.classForName(mapperClass);
                          configuration.addMapper(mapperInterface);
                      } else {
                          throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
                      }
                  }
              }
          }
      }
      ```
  
      - Configuration的getSqlFragments方法
  
        ```java
        //从以前的映射器解析的 XML 片段
        protected final Map<String, XNode> sqlFragments = new StrictMap<>
            
        public Map<String, XNode> getSqlFragments() {
            return sqlFragments;
        }
        ```
  
      - XMLMapperBuilder类
  
        ```java
        public class  XMLMapperBuilder extends BaseBuilder {
        
            private final XPathParser parser;
            private final MapperBuilderAssistant builderAssistant;
            private final Map<String, XNode> sqlFragments;
            private final String resource;
            //以下成员变量来则BaseBuilder
            protected final Configuration configuration;
            protected final TypeAliasRegistry typeAliasRegistry;
            protected final TypeHandlerRegistry typeHandlerRegistry;
            
            public XMLMapperBuilder(Reader reader, Configuration configuration, String resource, Map<String, XNode> sqlFragments, String namespace) {
                this(reader, configuration, resource, sqlFragments);
                this.builderAssistant.setCurrentNamespace(namespace);
        
            public XMLMapperBuilder(InputStream inputStream, Configuration configuration, String resource, Map<String, XNode> sqlFragments) {
                    this(new XPathParser(inputStream, true, configuration.getVariables(), new XMLMapperEntityResolver()),
                         configuration, resource, sqlFragments);
                }
        
            private XMLMapperBuilder(XPathParser parser, Configuration configuration, String resource, Map<String, XNode> sqlFragments) {
                    super(configuration);
                    this.builderAssistant = new MapperBuilderAssistant(configuration, resource);//通过Configuration和resource创建MapperBuilderAssistant
                    this.parser = parser;
                    this.sqlFragments = sqlFragments;
                    this.resource = resource;
                }
            }
        }
        ```
  
        - MapperBuilderAssistant类
  
          ```java
          public class MapperBuilderAssistant extends BaseBuilder {
          
              private String currentNamespace;//Mapper.xml文件中的NameSpace
              private final String resource;//该XML文件所在位置
              private Cache currentCache;//缓存
              private boolean unresolvedCacheRef; // 未解析的缓存引用
              
              //以下是BaseBuilder中的成员变量
              protected final Configuration configuration;
              protected final TypeAliasRegistry typeAliasRegistry;
              protected final TypeHandlerRegistry typeHandlerRegistry;
          
              public MapperBuilderAssistant(Configuration configuration, String resource) {
                  super(configuration);
                  ErrorContext.instance().resource(resource);
                  this.resource = resource;
              }
          }
          ```
  
      - XMLMapperBuilder的parse方法
  
        ```java
        public void parse() {
            if (!configuration.isResourceLoaded(resource)) {//如果没有解析该资源则进行解析
                configurationElement(parser.evalNode("/mapper"));//获取根标签Mapper（mapper.xml文件）
                configuration.addLoadedResource(resource);//设置该资源已被加载，configuration中包含以下成员变量
                //protected final Set<String> loadedResources = new HashSet<>();
                bindMapperForNamespace();//绑定Mapper和命名空间
            }
        
            parsePendingResultMaps();//解析configuration中未被完全解析的ResultMap
            parsePendingCacheRefs();//解析configuration中未被完全解析的Cache引用
            parsePendingStatements();//解析configuration中未被完全解析的SQL语句
        }
        ```
  
        - configurationElement方法
  
          ```java
          private void configurationElement(XNode context) {
              try {
                  String namespace = context.getStringAttribute("namespace");
                  if (namespace == null || namespace.equals("")) {
                      throw new BuilderException("Mapper's namespace cannot be empty");
                  }
                  builderAssistant.setCurrentNamespace(namespace);//设置builderAssistant的命名空间
                  cacheRefElement(context.evalNode("cache-ref"));//解析缓存引用
                  cacheElement(context.evalNode("cache"));//解析缓存
                  parameterMapElement(context.evalNodes("/mapper/parameterMap"));
                  resultMapElements(context.evalNodes("/mapper/resultMap"));
                  sqlElement(context.evalNodes("/mapper/sql"));
                  buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
              } catch (Exception e) {
                  throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
              }
          }
          ```
        
          - setCurrentNamespace方法
        
            ```java
            public void setCurrentNamespace(String currentNamespace) {
                if (currentNamespace == null) {
                    throw new BuilderException("The mapper element requires a namespace attribute to be specified.");
                }
            
                if (this.currentNamespace != null && !this.currentNamespace.equals(currentNamespace)) {//当前MapperBuilderAssistant中namespace必须和传入的currentNamespace相同
                    throw new BuilderException("Wrong namespace. Expected '"
                                               + this.currentNamespace + "' but found '" + currentNamespace + "'.");
                }
            
                this.currentNamespace = currentNamespace;//设置MapperBuilderAssistant的namespace为传入的参数
            }
            ```
        
          - cacheRefElement方法
        
            ```java
            private void cacheRefElement(XNode context) {
                if (context != null) {
                    configuration.addCacheRef(builderAssistant.getCurrentNamespace(), context.getStringAttribute("namespace"));//添加Cache引用（当前namespace到Cache-ref标签指定的namespace）
                    CacheRefResolver cacheRefResolver = new CacheRefResolver(builderAssistant, context.getStringAttribute("namespace"));//初始化cacheRefResolver（简单赋值）
                    try {
                        cacheRefResolver.resolveCacheRef();//解析Cache引用（调用MapperBuilderAssistant的useCacheRef方法），设置MapperBuilderAssistant中的Cache（通过Configuration获取）为引用Cache并返回该Cache
                    } catch (IncompleteElementException e) {
                        //如果被引用的Cache还未被创建则将该缓存引用放入未完全解析集合中
                        configuration.addIncompleteCacheRef(cacheRefResolver);
                    }
                }
            }
            ```
        
          - cacheElement方法
        
            ```java
            private void cacheElement(XNode context) {
                if (context != null) {
                    String type = context.getStringAttribute("type", "PERPETUAL");//获取缓存实现类
                    //从别名注册中查找该缓存实现类，如果找不到则以标签中提供的type属性作为全路径类名加载该缓存类并返回其Class
                    Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
                    String eviction = context.getStringAttribute("eviction", "LRU");
                    Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);//同上
                    Long flushInterval = context.getLongAttribute("flushInterval");//设置缓存刷新频率
                    Integer size = context.getIntAttribute("size");//设置缓大小
                    boolean readWrite = !context.getBooleanAttribute("readOnly", false);
                    boolean blocking = context.getBooleanAttribute("blocking", false);
                    Properties props = context.getChildrenAsProperties();
                    builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);//将该Cache添加值Configuration中的Caches集合中并将当前namespace使用的缓存为当前缓存
                }
            }
            ```
        
          - parameterMapElement方法
        
            ```java
            private void parameterMapElement(List<XNode> list) {
                for (XNode parameterMapNode : list) {
                    String id = parameterMapNode.getStringAttribute("id");
                    String type = parameterMapNode.getStringAttribute("type");
                    Class<?> parameterClass = resolveClass(type);//调用typeAliasRegistry的resolveAlias方法
                    //获取所有子标签parameter
                    List<XNode> parameterNodes = parameterMapNode.evalNodes("parameter");
                    List<ParameterMapping> parameterMappings = new ArrayList<>();
                    for (XNode parameterNode : parameterNodes) {
                        String property = parameterNode.getStringAttribute("property");
                        String javaType = parameterNode.getStringAttribute("javaType");
                        String jdbcType = parameterNode.getStringAttribute("jdbcType");
                        String resultMap = parameterNode.getStringAttribute("resultMap");
                        String mode = parameterNode.getStringAttribute("mode");
                        String typeHandler = parameterNode.getStringAttribute("typeHandler");
                        Integer numericScale = parameterNode.getIntAttribute("numericScale");
                        //mode指的是该参数的类型是输出参数还是输出参数，以下代码是将String转为枚举类型
                        ParameterMode modeEnum = resolveParameterMode(mode);
                        //String 转为Class
                        Class<?> javaTypeClass = resolveClass(javaType);
                        //String转JdbcType（枚举类型）
                        JdbcType jdbcTypeEnum = resolveJdbcType(jdbcType);
                        Class<? extends TypeHandler<?>> typeHandlerClass = resolveClass(typeHandler);
                        //将以上信息包装为一个ParameterMapping
                        ParameterMapping parameterMapping = builderAssistant.buildParameterMapping(parameterClass, property, javaTypeClass, jdbcTypeEnum, resultMap, modeEnum, typeHandlerClass, numericScale);
                        parameterMappings.add(parameterMapping);//向parameterMappings中添加该parameterMapping
                    }
                    //向configuration中添加该parameterMappings
                    builderAssistant.addParameterMap(id, parameterClass, parameterMappings);
                }
            }
            ```
        
            - buildParameterMapping方法
        
              ```java
              public ParameterMapping buildParameterMapping(
                  Class<?> parameterType,
                  String property,
                  Class<?> javaType,
                  JdbcType jdbcType,
                  String resultMap,
                  ParameterMode parameterMode,
                  Class<? extends TypeHandler<?>> typeHandler,
                  Integer numericScale) {
                  resultMap = applyCurrentNamespace(resultMap, true);
              
                  // Class parameterType = parameterMapBuilder.type();
                  Class<?> javaTypeClass = resolveParameterJavaType(parameterType, property, javaType, jdbcType);//将参数集合转换为JAVA对象Class，如果只有一个参数则parameterType为该参数类型，如果有多个参数则采用Map作为parameterType
                  TypeHandler<?> typeHandlerInstance = resolveTypeHandler(javaTypeClass, typeHandler)//获取typeHandler，先在typeHandler注册器中查找找不到再根据属性名称加载指定的TypeHandler
              
                  return new ParameterMapping.Builder(configuration, property, javaTypeClass)
                      .jdbcType(jdbcType)
                      .resultMapId(resultMap)
                      .mode(parameterMode)
                      .numericScale(numericScale)
                      .typeHandler(typeHandlerInstance)
                      .build();//创建ParameterMapping（简单赋值）
              }
              ```
        
              - resolveParameterJavaType方法
        
                ```java
                private Class<?> resolveParameterJavaType(Class<?> resultType, String property, Class<?> javaType, JdbcType jdbcType) {
                    if (javaType == null) {
                        if (JdbcType.CURSOR.equals(jdbcType)) {
                            javaType = java.sql.ResultSet.class;//游标类型则转换为ResultSet
                        } else if (Map.class.isAssignableFrom(resultType)) {//判断是否是Map类型
                            javaType = Object.class;//map类型则用Object包装
                        } else {
                            MetaClass metaResultType = MetaClass.forClass(resultType, configuration.getReflectorFactory());//普通类型则用MetaClass包装
                            javaType = metaResultType.getGetterType(property);
                        }
                    }
                    if (javaType == null) {
                        javaType = Object.class;
                    }
                    return javaType;
                }
                ```
        
          - resultMapElements方法
        
            ```java
            private void resultMapElements(List<XNode> list) {
                for (XNode resultMapNode : list) {
                    try {
                        resultMapElement(resultMapNode);
                    } catch (IncompleteElementException e) {
                        // ignore, it will be retried
                    }
                }
            }
            
            private ResultMap resultMapElement(XNode resultMapNode) {
                return resultMapElement(resultMapNode, Collections.emptyList(), null);
            }
            
            private ResultMap resultMapElement(XNode resultMapNode, List<ResultMapping> additionalResultMappings, Class<?> enclosingType) {
                ErrorContext.instance().activity("processing " + resultMapNode.getValueBasedIdentifier());
                //获取ResultMap最终需要转换的JAVA类型，优先采用type属性，其余优先级为ofType>resultType>javaType
                String type = resultMapNode.getStringAttribute("type",
                                                               resultMapNode.getStringAttribute("ofType",
                                                                                                resultMapNode.getStringAttribute("resultType",
                                                                                                                                 resultMapNode.getStringAttribute("javaType"))));
                Class<?> typeClass = resolveClass(type);//同上
                if (typeClass == null) {//如果没有指定Type
                    typeClass = inheritEnclosingType(resultMapNode, enclosingType);
                }
                Discriminator discriminator = null;
                List<ResultMapping> resultMappings = new ArrayList<>(additionalResultMappings);
                List<XNode> resultChildren = resultMapNode.getChildren();
                for (XNode resultChild : resultChildren) {
                    if ("constructor".equals(resultChild.getName())) {
                        processConstructorElement(resultChild, typeClass, resultMappings);
                    } else if ("discriminator".equals(resultChild.getName())) {
                        discriminator = processDiscriminatorElement(resultChild, typeClass, resultMappings);
                    } else {
                        List<ResultFlag> flags = new ArrayList<>();//用于存放该条标签的类型，只有ID和constructor两种
                        if ("id".equals(resultChild.getName())) {
                            flags.add(ResultFlag.ID);
                        }
                        resultMappings.add(buildResultMappingFromContext(resultChild, typeClass, flags));
                    }
                }
                String id = resultMapNode.getStringAttribute("id",
                                                             resultMapNode.getValueBasedIdentifier());//获取ID属性
                String extend = resultMapNode.getStringAttribute("extends");//获取resultMao继承属性
                Boolean autoMapping = resultMapNode.getBooleanAttribute("autoMapping");//获取autoMapping属性
                ResultMapResolver resultMapResolver = new ResultMapResolver(builderAssistant, id, typeClass, extend, discriminator, resultMappings, autoMapping);//简单赋值，ResultMapResolver包装了整个ResultMap
                try {
                    return resultMapResolver.resolve();
                } catch (IncompleteElementException  e) {
                    configuration.addIncompleteResultMap(resultMapResolver);
                    throw e;
                }
            }
            
            ```
        
            - buildResultMappingFromContext方法（单条Mapping的解析）
            
              ```java
              private ResultMapping buildResultMappingFromContext(XNode context, Class<?> resultType, List<ResultFlag> flags) {
                  String property;
                  if (flags.contains(ResultFlag.CONSTRUCTOR)) {
                      property = context.getStringAttribute("name");
                  } else {
                      property = context.getStringAttribute("property");
                  }
                  String column = context.getStringAttribute("column");
                  String javaType = context.getStringAttribute("javaType");
                  String jdbcType = context.getStringAttribute("jdbcType");
                  String nestedSelect = context.getStringAttribute("select");//嵌套Select
                  String nestedResultMap = context.getStringAttribute("resultMap", () ->
                                                                      processNestedResultMappings(context, Collections.emptyList(), resultType));//嵌套ResultMap（association，collection，case标签）使用递归处理
                  String notNullColumn = context.getStringAttribute("notNullColumn");
                  String columnPrefix = context.getStringAttribute("columnPrefix");
                  String typeHandler = context.getStringAttribute("typeHandler");
                  String resultSet = context.getStringAttribute("resultSet");
                  String foreignColumn = context.getStringAttribute("foreignColumn");
                  boolean lazy = "lazy".equals(context.getStringAttribute("fetchType", configuration.isLazyLoadingEnabled() ? "lazy" : "eager"));//加载类型：饥饿加载或者懒加载
                  Class<?> javaTypeClass = resolveClass(javaType);//同上
                  Class<? extends TypeHandler<?>> typeHandlerClass = resolveClass(typeHandler);//获取TypeHandler
                  JdbcType jdbcTypeEnum = resolveJdbcType(jdbcType);//将String转枚举
                  return builderAssistant.buildResultMapping(resultType, property, column, javaTypeClass, jdbcTypeEnum, nestedSelect, nestedResultMap, notNullColumn, columnPrefix, typeHandlerClass, flags, resultSet, foreignColumn, lazy);
              }
              ```
            
              - buildResultMapping方法（单条Mapping解析）
            
                ```java
                public ResultMapping buildResultMapping(
                    Class<?> resultType,
                    String property,
                    String column,
                    Class<?> javaType,
                    JdbcType jdbcType,
                    String nestedSelect,
                    String nestedResultMap,
                    String notNullColumn,
                    String columnPrefix,
                    Class<? extends TypeHandler<?>> typeHandler,
                    List<ResultFlag> flags,
                    String resultSet,
                    String foreignColumn,
                    boolean lazy) {
                    Class<?> javaTypeClass = resolveResultJavaType(resultType, property, javaType);//获取该成员变量名称类型
                    TypeHandler<?> typeHandlerInstance = resolveTypeHandler(javaTypeClass, typeHandler);//获取Typehandler
                    List<ResultMapping> composites;
                    if ((nestedSelect == null || nestedSelect.isEmpty()) && (foreignColumn == null || foreignColumn.isEmpty())) {
                        composites = Collections.emptyList();//如果没有select和foreignColumn则直接将composites设置为空
                    } else {
                        composites = parseCompositeColumnName(column);//
                    }
                    return new ResultMapping.Builder(configuration, property, column, javaTypeClass)//简单赋值操作，flags和composites都设置为空list，由后续的赋值语句进行二次赋值
                        .jdbcType(jdbcType)
                        .nestedQueryId(applyCurrentNamespace(nestedSelect, true))
                        .nestedResultMapId(applyCurrentNamespace(nestedResultMap, true))
                        .resultSet(resultSet)
                        .typeHandler(typeHandlerInstance)
                        .flags(flags == null ? new ArrayList<>() : flags)
                        .composites(composites)
                        .notNullColumns(parseMultipleColumnNames(notNullColumn))
                        .columnPrefix(columnPrefix)
                        .foreignColumn(foreignColumn)
                        .lazy(lazy)
                        .build();//设置ResultMap中的flags和composites为不可变集合，如果typehandler为空但是javaType不为空则从configuration中寻找合适的typeHandler
                }
                ```
            
                - resolveResultJavaType方法
            
                  ```java
                  private Class<?> resolveResultJavaType(Class<?> resultType, String property, Class<?> javaType) {
                      if (javaType == null && property != null) {//这里的property是成员变量名称
                          try {
                              MetaClass metaResultType = MetaClass.forClass(resultType, configuration.getReflectorFactory());//创建一个MetaClass并将Configuration中的反射工厂作为其成员变量
                              javaType = metaResultType.getSetterType(property);//获取set方法的参数类型
                          } catch (Exception e) {
                              //ignore, following null check statement will deal with the situation
                          }
                      }
                      if (javaType == null) {
                          javaType = Object.class;
                      }
                      return javaType;
                  }
                  ```
            
            - resultMapResolver的resolve方法
            
              ```java
              public ResultMap resolve() {
                  return assistant.addResultMap(this.id, this.type, this.extend, this.discriminator, this.resultMappings, this.autoMapping);
              }
              
              
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
                  configuration.addResultMap(resultMap);//向Configuration中的resultMaps中添加该resultMap
                  return resultMap;
                }
              ```
            
          - sqlElementf方法
        
            ```java
            
            private final Map<String, XNode> sqlFragments;
            
            private void sqlElement(List<XNode> list) {
                if (configuration.getDatabaseId() != null) {
                  sqlElement(list, configuration.getDatabaseId());
                }
                sqlElement(list, null);
              }
            
            
            private void sqlElement(List<XNode> list, String requiredDatabaseId) {
                for (XNode context : list) {
                    String databaseId = context.getStringAttribute("databaseId");
                    String id = context.getStringAttribute("id");
                    id = builderAssistant.applyCurrentNamespace(id, false);//如果是多级名称（含有“.”）则检查是否属于当前命名空间，如果不含有"."则在id前拼接当前命名空间
                    if (databaseIdMatchesCurrent(id, databaseId, requiredDatabaseId)) {//检查当前的databaseId是否符合要求的databaseId
                        sqlFragments.put(id, context);//放入sqlFragments中
                    }
                }
            }
            ```
        
          - buildStatementFromContextfa方法
        
            ```java
            private void buildStatementFromContext(List<XNode> list) {
                if (configuration.getDatabaseId() != null) {
                    buildStatementFromContext(list, configuration.getDatabaseId());
                }
                buildStatementFromContext(list, null);
            }
            
            
            private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
                for (XNode context : list) {
                    final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
                    try {
                        statementParser.parseStatementNode();
                    } catch (IncompleteElementException e) {
                        configuration.addIncompleteStatement(statementParser);
                    }
                }
            }
            ```
        
            - parseStatementNode方法
        
              ```java
              public void parseStatementNode() {
                  String id = context.getStringAttribute("id");//获取SQL语句标签ID
                  String databaseId = context.getStringAttribute("databaseId");
              
                  if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {//如果两者的databaseId不匹配则直接退出
                    return;
                  }
              
                  String nodeName = context.getNode().getNodeName();//获取标签名称
                  SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));//从标签名称获取SQL类型
                  boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
                  boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
                  boolean useCache = context.getBooleanAttribute("useCache", isSelect);
                  boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);
              
                  // Include Fragments before parsing
                  XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);//创建include标签XML翻译器
                  includeParser.applyIncludes(context.getNode());//解析include标签
              
                  String parameterType = context.getStringAttribute("parameterType");//获取parameterType属性
                  Class<?> parameterTypeClass = resolveClass(parameterType);//获取参数类型Class（从别名注册中找）
              
                  String lang = context.getStringAttribute("lang");
                  LanguageDriver langDriver = getLanguageDriver(lang);
              
                  // Parse selectKey after includes and remove them.
                  processSelectKeyNodes(id, parameterTypeClass, langDriver);//解析selectKey标签
              
                  // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
                  KeyGenerator keyGenerator;
                  String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
                  keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
                  if (configuration.hasKeyGenerator(keyStatementId)) {//查看Configuration中是否含有keyGenerator
                    keyGenerator = configuration.getKeyGenerator(keyStatementId);
                  } else {
                    keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
                        configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
                        ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
                  }
              
                  SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);//解析SQL语句（将动态SQL标签和参数占位符进行转换）
                  StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));//获取SQL语句类型
                  Integer fetchSize = context.getIntAttribute("fetchSize");
                  Integer timeout = context.getIntAttribute("timeout");
                  String parameterMap = context.getStringAttribute("parameterMap");
                  String resultType = context.getStringAttribute("resultType");
                  Class<?> resultTypeClass = resolveClass(resultType);
                  String resultMap = context.getStringAttribute("resultMap");
                  String resultSetType = context.getStringAttribute("resultSetType");
                  ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);
                  if (resultSetTypeEnum == null) {
                    resultSetTypeEnum = configuration.getDefaultResultSetType();
                  }
                  String keyProperty = context.getStringAttribute("keyProperty");
                  String keyColumn = context.getStringAttribute("keyColumn");
                  String resultSets = context.getStringAttribute("resultSets");
              
                  builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
                      fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
                      resultSetTypeEnum, flushCache, useCache, resultOrdered,
                      keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
                }
              ```
        
              - XMLIncludeTransformer类
              
                ```java
                public class XMLIncludeTransformer {
                
                  private final Configuration configuration;
                  private final MapperBuilderAssistant builderAssistant;
                
                  public XMLIncludeTransformer(Configuration configuration, MapperBuilderAssistant builderAssistant) {
                    this.configuration = configuration;
                    this.builderAssistant = builderAssistant;
                  }
                }
                ```
              
              - applyInclude方法
              
                ```java
                public void applyIncludes(Node source) {
                    Properties variablesContext = new Properties();
                    Properties configurationVariables = configuration.getVariables();
                    Optional.ofNullable(configurationVariables).ifPresent(variablesContext::putAll);
                    applyIncludes(source, variablesContext, false);//解析Include标签
                }
                ```
              
                - option类
              
                  ```java
                  public final class Optional<T> {
                  
                      private static final Optional<?> EMPTY = new Optional<>();
                  
                      private final T value;
                      
                      public static<T> Optional<T> empty() {
                          @SuppressWarnings("unchecked")
                          Optional<T> t = (Optional<T>) EMPTY;
                          return t;
                      }
                      
                      public static <T> Optional<T> of(T value) {
                          return new Optional<>(value);
                      }
                      public static <T> Optional<T> ofNullable(T value) {
                          return value == null ? empty() : of(value);//如果不为空则创建一个Option其中的value是传入的参数如果为空则返回一个空的Option
                      }
                      
                      public void ifPresent(Consumer<? super T> consumer) {
                          if (value != null)//如果value不为空则将value作为参数传入consumer接口中并执行consumer接口
                              consumer.accept(value);
                      }
                  }
                  ```
              
                - applyIncludes方法
              
                  ```java
                  private void applyIncludes(Node source, final Properties variablesContext, boolean included) {
                      if (source.getNodeName().equals("include")) {//检查该标签的名称是否是include
                          Node toInclude = findSqlFragment(getStringAttribute(source, "refid"), variablesContext);//通过include标签上的refid属性找到该SQL标签（在Configuration中的sqlFragments查找）
                          Properties toIncludeContext = getVariablesContext(source, variablesContext);
                          applyIncludes(toInclude, toIncludeContext, true);//递归include标签
                          if (toInclude.getOwnerDocument() != source.getOwnerDocument()) {
                              toInclude = source.getOwnerDocument().importNode(toInclude, true);
                          }
                          source.getParentNode().replaceChild(toInclude, source);//将所有的source节点更换为toInclude节点（include更换为sql）
                          while (toInclude.hasChildNodes()) {
                              toInclude.getParentNode().insertBefore(toInclude.getFirstChild(), toInclude);//将toinclude节点更换为toinclude第一个子节点（也就是将sql节点更换为文本节点）
                          }
                          toInclude.getParentNode().removeChild(toInclude);//删除toInclude节点（删除sql节点）
                      } else if (source.getNodeType() == Node.ELEMENT_NODE) {//如果属于Element——Node
                          if (included && !variablesContext.isEmpty()) {//如果参数included（用于表示当前传入的参数是否是include的有关的node）为true且variablesContext不为空
                              // replace variables in attribute values
                              NamedNodeMap attributes = source.getAttributes();//获取该node的所有属性
                              for (int i = 0; i < attributes.getLength(); i++) {
                                  Node attr = attributes.item(i);//逐个获取子标签并以Node封装
                                  attr.setNodeValue(PropertyParser.parse(attr.getNodeValue(), variablesContext));//设置Node值
                              }
                          }
                          NodeList children = source.getChildNodes();//获取该Node下的所有子Node
                          for (int i = 0; i < children.getLength(); i++) {
                              applyIncludes(children.item(i), variablesContext, included);//递归调用
                          }//如果是文本类Node或者CDATA_SECTION_NODE且variablesContext不为空
                      } else if (included && (source.getNodeType() == Node.TEXT_NODE || source.getNodeType() == Node.CDATA_SECTION_NODE)
                                 && !variablesContext.isEmpty()) {//最后SQL标签中内容也会作为文本Node进入到这里
                          // replace variables in text node
                          source.setNodeValue(PropertyParser.parse(source.getNodeValue(), variablesContext));
                      }
                  }
                  ```
              
              - langDriver的createSqlSource方法
              
                ```java
                public SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType) {
                    XMLScriptBuilder builder = new XMLScriptBuilder(configuration, script, parameterType);
                    return builder.parseScriptNode();//下面有
                }
                ```
              
                
        
        - bindMapperForNamespace
        
          ```java
          private void bindMapperForNamespace() {
              String namespace = builderAssistant.getCurrentNamespace();//获取当前Namespace
              if (namespace != null) {
                  Class<?> boundType = null;
                  try {
                      boundType = Resources.classForName(namespace);//加载Namespace对应的类
                  } catch (ClassNotFoundException e) {
                      //可忽视，boundType并非必须
                  }
                  if (boundType != null) {
                      if (!configuration.hasMapper(boundType)) {
                          // Spring 可能不知道真正的资源名称，所以我们设置了一个标志
                          // 防止从映射器界面再次加载此资源
                          // look at MapperAnnotationBuilder#loadXmlResource
                          configuration.addLoadedResource("namespace:" + namespace);
                          configuration.addMapper(boundType);//添加该Mapper
                      }
                  }
              }
          }
          ```
        
          - Configuration中的addMapper方法
        
            ```java
            public <T> void addMapper(Class<T> type) {
                mapperRegistry.addMapper(type);//boundType==type，也即是Mapper接口class
            }
            ```
        
            - MapperRegistry类
        
              ```java
              public class MapperRegistry {
              
                private final Configuration config;
                private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();//已知的Mapper
              
                public MapperRegistry(Configuration config) {
                  this.config = config;
                }
              }
              
              ```
        
            - MapperRegistry的addMapper方法
        
              ```java
              public <T> void addMapper(Class<T> type) {
                  if (type.isInterface()) {
                    if (hasMapper(type)) {
                      throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
                    }
                    boolean loadCompleted = false;
                    try {
                      knownMappers.put(type, new MapperProxyFactory<>(type));
                   // 在运行解析器之前添加类型很重要，否则映射器解析器可能会自动尝试绑定。如果类型已知，则不会尝试。
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
        
              - MapperProxyFactory类
        
                ```java
                public class MapperProxyFactory<T> {
                
                  private final Class<T> mapperInterface;
                  private final Map<Method, MapperMethodInvoker> methodCache = new ConcurrentHashMap<>();
                
                  public MapperProxyFactory(Class<T> mapperInterface) {
                    this.mapperInterface = mapperInterface;
                  }
                }
                ```
        
              - MapperAnnotationBuilder类
        
                ```java
                public class MapperAnnotationBuilder {
                
                    private static final Set<Class<? extends Annotation>> SQL_ANNOTATION_TYPES = new HashSet<>();//SQL 注释类型
                    //SQL提供者注解类型指的是有类来代替Mapper.xml文件，通过类来指定SQL文件，而SQL_PROVIDER注解就是指定实现SQL语句的类
                    private static final Set<Class<? extends Annotation>> SQL_PROVIDER_ANNOTATION_TYPES = new HashSet<>();//SQL 提供者注解类型
                
                    private final Configuration configuration;
                    private final MapperBuilderAssistant assistant;//关于Mapper中namespace等信息的封装
                    private final Class<?> type;
                
                    static {//初始化SQL_ANNOTATION_TYPES和SQL_PROVIDER_ANNOTATION_TYPES
                        SQL_ANNOTATION_TYPES.add(Select.class);
                        SQL_ANNOTATION_TYPES.add(Insert.class);
                        SQL_ANNOTATION_TYPES.add(Update.class);
                        SQL_ANNOTATION_TYPES.add(Delete.class);
                
                        SQL_PROVIDER_ANNOTATION_TYPES.add(SelectProvider.class);
                        SQL_PROVIDER_ANNOTATION_TYPES.add(InsertProvider.class);
                        SQL_PROVIDER_ANNOTATION_TYPES.add(UpdateProvider.class);
                        SQL_PROVIDER_ANNOTATION_TYPES.add(DeleteProvider.class);
                    }
                
                    public MapperAnnotationBuilder(Configuration configuration, Class<?> type) {
                        String resource = type.getName().replace('.', '/') + ".java (best guess)";
                        this.assistant = new MapperBuilderAssistant(configuration, resource);//通过configuration和resource创建MapperBuilderAssistant确立命名空间
                        this.configuration = configuration;
                        this.type = type;
                    }
                }
                ```
                
                - MapperAnnotationBuilder的parse方法
                
                  ```java
                  public void parse() {
                      String resource = type.toString();//获取mapper接口的全路径名称
                      if (!configuration.isResourceLoaded(resource)) {//检查该Mapper接口资源是否已加载
                          loadXmlResource();//加载该mapper接口(和上面的XMLMapperBuilder的构造方法一样，只是进行简单赋值)
                          configuration.addLoadedResource(resource);
                          assistant.setCurrentNamespace(type.getName());
                          parseCache();
                          parseCacheRef();
                          Method[] methods = type.getMethods();//获取Mapper接口中的所有方法
                          for (Method method : methods) {
                              try {
                                  // issue #237
                                  if (!method.isBridge()) {//如果该方法不是桥接方法则解析，桥接方法指的是JVM为了擦除泛型而自动生成的方法
                                      parseStatement(method);
                                  }
                              } catch (IncompleteElementException e) {
                                  //如果抛出IncompleteElementException异常则将该Method添加值未完全解析的Statement的集合中，之后使用时会再次进行解析
                                  configuration.addIncompleteMethod(new MethodResolver(this, method));
                              }
                          }
                      }
                      parsePendingMethods();
                  }
                  ```
                  
                  - parseCache方法
                  
                    ```java
                    /*mybatis二级缓存使用
                    <cache
                      eviction="FIFO"
                      flushInterval="60000"
                      size="512"
                      readOnly="true"/>
                      缓存只作用于 cache 标签所在的映射文件中的语句。如果你混合使用 Java API 和 XML 映射文件，在共用接口中的语句将不会被默认缓存。你需要使用 @CacheNamespaceRef 注解指定缓存作用域。
                     
                      除了上述自定义缓存的方式，你也可以通过实现你自己的缓存，或为其他第三方缓存方案创建适配器，来完全覆盖缓存行为。
                       <cache type="com.domain.something.MyCustomCache">
                      <property name="cacheFile" value="/tmp/my-custom-cache.tmp"/>
                    </cache>
                    */
                    
                    private void parseCache() {
                        CacheNamespace cacheDomain = type.getAnnotation(CacheNamespace.class);//这里的type是mapper接口的包名，这里是获取CacheNamespace注解，详细Cache使用可参考https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#cache
                        if (cacheDomain != null) {
                            Integer size = cacheDomain.size() == 0 ? null : cacheDomain.size();
                            Long flushInterval = cacheDomain.flushInterval() == 0 ? null : cacheDomain.flushInterval();//如果刷新频率未0则不刷新
                            Properties props = convertToProperties(cacheDomain.properties());//将Property注解中的内容转换为Properties
                            assistant.useNewCache(cacheDomain.implementation(), cacheDomain.eviction(), flushInterval, size, cacheDomain.readWrite(), cacheDomain.blocking(), props);
                        }
                    }
                    ```
                  
                    - CacheNamespace注解
                  
                      ```java
                      @Documented
                      @Retention(RetentionPolicy.RUNTIME)
                      @Target(ElementType.TYPE)
                      public @interface CacheNamespace {
                      
                        Class<? extends Cache> implementation() default PerpetualCache.class;
                      
                        Class<? extends Cache> eviction() default LruCache.class;
                      
                        long flushInterval() default 0;//刷新频率
                      
                        int size() default 1024;
                      
                        boolean readWrite() default true;
                      
                        boolean blocking() default false;
                      
                        Property[] properties() default {};//Property注解用于指定一些配置信息如缓存文件的位置指定
                      
                      }
                      
                      ```
                  
                      - Property注解
                  
                        ```java
                        @Documented
                        @Retention(RetentionPolicy.RUNTIME)
                        @Target({})
                        public @interface Property {
                          String name();
                          String value();
                        }
                        ```
                  
                    - MapperBuilderAssistant的useNewCache方法
                  
                      ```java
                      protected final Map<String, Cache> caches = new StrictMap<>("Caches collection");//key为Cache的Namespace
                      
                      public Cache useNewCache(Class<? extends Cache> typeClass,//缓存的实现类
                            Class<? extends Cache> evictionClass,//缓存策略
                            Long flushInterval,//刷新频率
                            Integer size,//缓存大小
                            boolean readWrite,//是否只读
                            boolean blocking,//是否堵塞
                            Properties props) {//一些配置
                          Cache cache = new CacheBuilder(currentNamespace)
                              .implementation(valueOrDefault(typeClass, PerpetualCache.class))
                              .addDecorator(valueOrDefault(evictionClass, LruCache.class))//设置缓存策略
                              .clearInterval(flushInterval)//设置刷新时间
                              .size(size)//设置缓存大小
                              .readWrite(readWrite)//设置是否只读
                              .blocking(blocking)//设置是否堵塞
                              .properties(props)//设置缓存文件存放位置
                              .build();//创建
                          configuration.addCache(cache);//放入caches中
                          currentCache = cache;//设置当前Cache是该Cache
                          return cache;
                        }
                      ```
                  
                    - PerpetualCache类
                  
                      ```java
                      public class PerpetualCache implements Cache {
                      
                          private final String id;
                      
                          private final Map<Object, Object> cache = new HashMap<>();
                      //省略get和put方法
                      }
                      ```
                  
                  - parseCacheRef方法
                  
                    ```java
                    private void parseCacheRef() {
                        //引用另外一个命名空间的缓存以供使用。注意，即使共享相同的全限定类名，在 XML 映射文件中声明的缓存仍被识别为一个独立的命名空间。属性：value、name。如果你使用了这个注解，你应设置 value 或者 name 属性的其中一个。value 属性用于指定能够表示该命名空间的 Java 类型（命名空间名就是该 Java 类型的全限定类名），name 属性（这个属性仅在 MyBatis 3.4.2 以上可用）则直接指定了命名空间的名字。
                        
                        //回想一下上一节的内容，对某一命名空间的语句，只会使用该命名空间的缓存进行缓存或刷新。 但你可能会想要在多个命名空间中共享相同的缓存配置和实例。要实现这种需求，你可以使用 cache-ref 元素来引用另一个缓存。
                    //<cache-ref namespace="com.someone.application.data.SomeMapper"/>
                        
                        CacheNamespaceRef cacheDomainRef = type.getAnnotation(CacheNamespaceRef.class);
                        if (cacheDomainRef != null) {
                            Class<?> refType = cacheDomainRef.value();
                            String refName = cacheDomainRef.name();
                            if (refType == void.class && refName.isEmpty()) {//必须指定name和value其中一个属性
                                throw new BuilderException("Should be specified either value() or name() attribute in the @CacheNamespaceRef");
                            }
                            if (refType != void.class && !refName.isEmpty()) {//不可以两个都指定
                                throw new BuilderException("Cannot use both value() and name() attribute in the @CacheNamespaceRef");
                            }
                            String namespace = (refType != void.class) ? refType.getName() : refName;//name和value两者取其一
                            try {
                                assistant.useCacheRef(namespace);//MapperBuilderAssistant中的Cache赋值为指定Cache
                            } catch (IncompleteElementException e) {
                                configuration.addIncompleteCacheRef(new CacheRefResolver(assistant, namespace));
                            }
                        }
                    }
                    ```
                  
                    - MapperBuilderAssistant的useCacheRef方法
                  
                      ```java
                      public Cache useCacheRef(String namespace) {
                          if (namespace == null) {
                              throw new BuilderException("cache-ref element requires a namespace attribute.");
                          }
                          try {
                              unresolvedCacheRef = true;
                              Cache cache = configuration.getCache(namespace);//从Caches中获取Cache
                              if (cache == null) {
                                  throw new IncompleteElementException("No cache for namespace '" + namespace + "' could be found.");
                              }
                              currentCache = cache;
                              unresolvedCacheRef = false;
                              return cache;
                          } catch (IllegalArgumentException e) {
                              throw new IncompleteElementException("No cache for namespace '" + namespace + "' could be found.", e);
                          }
                      }
                      ```
                  
                  - parseStatement方法
                  
                    ```java
                    void parseStatement(Method method) {
                        Class<?> parameterTypeClass = getParameterType(method);//(1)获取参数类型的Class，该方法源码在下方
                        LanguageDriver languageDriver = getLanguageDriver(method);//获取语言驱动(用于创建SQLsource)
                        SqlSource sqlSource = getSqlSourceFromAnnotations(method, parameterTypeClass, languageDriver);//(2)通过注解获取SQLsource
                        if (sqlSource != null) {
                          Options options = method.getAnnotation(Options.class);//获取Option注解
                          final String mappedStatementId = type.getName() + "." + method.getName();//获取mapperSQL语句ID（Mapper类全路径名称+方法名称）
                          Integer fetchSize = null;
                          Integer timeout = null;
                          StatementType statementType = StatementType.PREPARED;//设置SQL语句类型
                          ResultSetType resultSetType = configuration.getDefaultResultSetType();
                          SqlCommandType sqlCommandType = getSqlCommandType(method);
                          boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
                          boolean flushCache = !isSelect;
                          boolean useCache = isSelect;
                    
                          KeyGenerator keyGenerator;//selectkey的执行器
                          String keyProperty = null;//与SelectKey有关
                          String keyColumn = null;
                          if (SqlCommandType.INSERT.equals(sqlCommandType) || SqlCommandType.UPDATE.equals(sqlCommandType)) {
                            // first check for SelectKey annotation - that overrides everything else
                              
                             //下面 这个例子展示了如何使用 @SelectKey 注解来在插入前读取数据库序列的值：
                    
                    //@Insert("insert into table3 (id, name) values(#{nameId}, #{name})")
                    //@SelectKey(statement="call next value for TestSequence", keyProperty="nameId", before=true, resultType=int.class)
                    int insertTable3(Name name);
                            SelectKey selectKey = method.getAnnotation(SelectKey.class);
                            if (selectKey != null) {
                              keyGenerator = handleSelectKeyAnnotation(selectKey, mappedStatementId, getParameterType(method), languageDriver);
                              keyProperty = selectKey.keyProperty();
                            } else if (options == null) {//如果没有options注解则采用configuration中的配置生成keyGenerator
                              keyGenerator = configuration.isUseGeneratedKeys() ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
                            } else {//如果含有options则采用options中的配置
                              keyGenerator = options.useGeneratedKeys() ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
                              keyProperty = options.keyProperty();
                              keyColumn = options.keyColumn();
                            }
                          } else {//没有selectKey注解则设置为无KeyGenerator
                            keyGenerator = NoKeyGenerator.INSTANCE;
                          }
                    
                          if (options != null) {//根据options设置缓存策略
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
                            //这个注解为 @Select 或者 @SelectProvider 注解指定 XML 映射中 <resultMap> 元素的 id。这使得注解的 select 可以复用已在 XML 中定义的 ResultMap。如果标注的 select 注解中存在 @Results 或者 @ConstructorArgs 注解，这两个注解将被此注解覆盖。
                          ResultMap resultMapAnnotation = method.getAnnotation(ResultMap.class);//获取ResultMap注解
                          if (resultMapAnnotation != null) {
                            resultMapId = String.join(",", resultMapAnnotation.value());//以“，”拼接resultMapAnnotation的value并返回resultMapId
                          } else if (isSelect) {
                            resultMapId = parseResultMap(method);//解析ResultMap并返回resultMapId
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
                  
                    - (1)getParameterType方法
                  
                      ```java
                      private Class<?> getParameterType(Method method) {
                          Class<?> parameterType = null;
                          Class<?>[] parameterTypes = method.getParameterTypes();
                          for (Class<?> currentParameterType : parameterTypes) {
                            if (!RowBounds.class.isAssignableFrom(currentParameterType) && !ResultHandler.class.isAssignableFrom(currentParameterType)) {//如果现在的参数类型不是特殊类型则进入
                              if (parameterType == null) {//如果为空则代表这是第一个非特殊类型的参数类型进入该if块中
                                parameterType = currentParameterType;
                              } else {//这是第二个非特殊类型的参数进入if块中，这说明该SQL语句的非特殊参数不止一个则采用Map作为参数列表
                                // issue #135
                                parameterType = ParamMap.class;
                              }
                            }
                          }
                          return parameterType;
                        }
                      ```
                  
                    - (2)getSqlSourceFromAnnotations方法
                  
                      ```java
                      private SqlSource getSqlSourceFromAnnotations(Method method, Class<?> parameterType, LanguageDriver languageDriver) {
                          try {
                              Class<? extends Annotation> sqlAnnotationType = getSqlAnnotationType(method);//在MapperAnnotationBuilder中的sql语句类型注解set集合中查找该SQL的类型并返回
                              Class<? extends Annotation> sqlProviderAnnotationType = getSqlProviderAnnotationType(method);//获取该Mapper上单SqlProviderAnnotation注解
                              if (sqlAnnotationType != null) {
                                  if (sqlProviderAnnotationType != null) {
                                      throw new BindingException("You cannot supply both a static SQL and SqlProvider to method named " + method.getName());//两者不能同时指定
                                  }
                                  Annotation sqlAnnotation = method.getAnnotation(sqlAnnotationType);//获取SQL语句类型注解
                                  //获取sqlAnnotation上的SQL语句(value是String数组)
                                  final String[] strings = (String[]) sqlAnnotation.getClass().getMethod("value").invoke(sqlAnnotation);
                                  return buildSqlSourceFromStrings(strings, parameterType, languageDriver);
                              } else if (sqlProviderAnnotationType != null) {
                                  Annotation sqlProviderAnnotation = method.getAnnotation(sqlProviderAnnotationType);
                                  return new ProviderSqlSource(assistant.getConfiguration(), sqlProviderAnnotation, type, method);
                              }
                              return null;
                          } catch (Exception e) {
                              throw new BuilderException("Could not find value method on SQL annotation.  Cause: " + e, e);
                          }
                      }
                      ```
                  
                      - buildSqlSourceFromStrings
                  
                        ```java
                        private SqlSource buildSqlSourceFromStrings(String[] strings, Class<?> parameterTypeClass, LanguageDriver languageDriver) {
                            final StringBuilder sql = new StringBuilder();
                            for (String fragment : strings) {//传入的String数组是未完成拼接的SQL语句，将该数组转化为SQL语句并在其中添加空格
                                sql.append(fragment);
                                sql.append(" ");
                            }
                            return languageDriver.createSqlSource(configuration, sql.toString().trim(), parameterTypeClass);
                        }
                        ```
                  
                        - createSqlSource方法
                  
                          ```java
                               public SqlSource createSqlSource(Configuration configuration, String script, Class<?> parameterType) {
                                  // issue #3
                                  if (script.startsWith("<script>")) {//script标签是在使用SQL注解时如果要使用动态SQL标签则需要用script标签包含整个SQL语句
                                    XPathParser parser = new XPathParser(script, false, configuration.getVariables(), new XMLMapperEntityResolver());//获取parser
                                    return createSqlSource(configuration, parser.evalNode("/script"), parameterType);//解析SQL注解上的SQL语句
                                  } else {
                                    // issue #127
                                    script = PropertyParser.parse(script, configuration.getVariables());//解析SQL语句
                                    TextSqlNode textSqlNode = new TextSqlNode(script);
                                    if (textSqlNode.isDynamic()) {
                                      return new DynamicSqlSource(configuration, textSqlNode);
                                    } else {
                                      return new RawSqlSource(configuration, script, parameterType);
                                    }
                                  }
                                }
                          ```
                  
                        - PropertyParser的parse方法
                  
                          ```java
                          public static String parse(String string, Properties variables) {
                              VariableTokenHandler handler = new VariableTokenHandler(variables);//新建令牌处理器
                              GenericTokenParser parser = new GenericTokenParser("${", "}", handler);//新建解析器
                              return parser.parse(string);//将SQL语句解析为原生SQL语句（就是将${}替换为？）
                          }
                          ```
                  
                        - createSqlSource方法
                  
                          ```java
                          public SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType) {
                              XMLScriptBuilder builder = new XMLScriptBuilder(configuration, script, parameterType);
                              return builder.parseScriptNode();
                          }
                          ```
                  
                        - XMLScriptBuilder类
                  
                          ```java
                          public class XMLScriptBuilder extends BaseBuilder {
                          
                            private final XNode context;//SQL语句，根标签是Script标签
                            private boolean isDynamic;
                            private final Class<?> parameterType;
                            private final Map<String, NodeHandler> nodeHandlerMap = new HashMap<>();//用于存放动态SQL标签的解析类
                          
                            public XMLScriptBuilder(Configuration configuration, XNode context) {
                              this(configuration, context, null);
                            }
                          
                            public XMLScriptBuilder(Configuration configuration, XNode context, Class<?> parameterType) {
                              super(configuration);
                              this.context = context;
                              this.parameterType = parameterType;
                              initNodeHandlerMap();
                            }
                          
                          
                            private void initNodeHandlerMap() {//注册动态SQL标签
                              nodeHandlerMap.put("trim", new TrimHandler());
                              nodeHandlerMap.put("where", new WhereHandler());
                              nodeHandlerMap.put("set", new SetHandler());
                              nodeHandlerMap.put("foreach", new ForEachHandler());
                              nodeHandlerMap.put("if", new IfHandler());
                              nodeHandlerMap.put("choose", new ChooseHandler());
                              nodeHandlerMap.put("when", new IfHandler());
                              nodeHandlerMap.put("otherwise", new OtherwiseHandler());
                              nodeHandlerMap.put("bind", new BindHandler());
                            }
                          }
                          ```
                  
                          - parseScriptNode方法
                  
                            ```java
                            public SqlSource parseScriptNode() {
                              MixedSqlNode rootSqlNode = parseDynamicTags(context);
                              SqlSource sqlSource;
                              if (isDynamic) {//含有${}的sql为动态SQL语句，这里做一步判断并且将${}转换
                                sqlSource = new DynamicSqlSource(configuration, rootSqlNode);//创建动态SQLSource
                              } else {
                                sqlSource = new RawSqlSource(configuration, rootSqlNode, parameterType);//创建原生SQLsource（将#{}更换为？）
                              }
                              return sqlSource;
                            }
                            ```
                  
                            - parseDynamicTags方法
                  
                              ```java
                                protected MixedSqlNode parseDynamicTags(XNode node) {
                                  List<SqlNode> contents = new ArrayList<>();
                                  NodeList children = node.getNode().getChildNodes();//获取子节点
                                  for (int i = 0; i < children.getLength(); i++) {//遍历子节点
                                    XNode child = node.newXNode(children.item(i));//使用XNode包装Node
                                    if (child.getNode().getNodeType() == Node.CDATA_SECTION_NODE || child.getNode().getNodeType() == Node.TEXT_NODE) {
                                      String data = child.getStringBody("");//如果XNode中的Body为则返回传入的参数也就是空格
                                      TextSqlNode textSqlNode = new TextSqlNode(data);
                                      if (textSqlNode.isDynamic()) {//该SQLnode是否是动态的，如果是则放入contents中
                                        contents.add(textSqlNode);
                                        isDynamic = true;
                                      } else {
                                        contents.add(new StaticTextSqlNode(data));//如果SQLNode不是动态的则通过StaticTextSqlNode包装后放入contents
                                      }
                                    } else if (child.getNode().getNodeType() == Node.ELEMENT_NODE) { // 如果是动态SQL标签则通过NodeHandler解析
                                      String nodeName = child.getNode().getNodeName();
                                      NodeHandler handler = nodeHandlerMap.get(nodeName);//通过标签名称获取NodeHandler
                                      if (handler == null) {
                                        throw new BuilderException("Unknown element <" + nodeName + "> in SQL statement.");
                                      }
                                      handler.handleNode(child, contents);//解析动态SQL
                                      isDynamic = true;
                                    }
                                  }
                                  return new MixedSqlNode(contents);//通过MixedSqlNode包装contents
                                }
                              ```
                  
                    - Option注解
                  
                      ```java
                      @Documented
                      @Retention(RetentionPolicy.RUNTIME)
                      @Target(ElementType.METHOD)
                      public @interface Options {
                        /**
                         * 该注解允许你指定大部分开关和配置选项，它们通常在映射语句上作为属性出现。与在注解上提供大量的属性相比，Options 注解提供了一致、清晰的方式来指定选项。属性：useCache=true、flushCache=FlushCachePolicy.DEFAULT、resultSetType=DEFAULT、statementType=PREPARED、fetchSize=-1、timeout=-1、useGeneratedKeys=false、keyProperty=""、keyColumn=""、resultSets="", databaseId=""。注意，Java 注解无法指定 null 值。因此，一旦你使用了 Options 注解，你的语句就会被上述属性的默认值所影响。要注意避免默认值带来的非预期行为。 databaseId（3.5.5以上可用）, 如果有一个配置好的 DatabaseIdProvider, MyBatis 会加载不带 databaseId 属性和带有匹配当前数据库 databaseId 属性的所有语句。如果同时存在带 databaseId 和不带 databaseId 属性的相同语句，则后者会被舍弃。
                      
                             注意：keyColumn 属性只在某些数据库中有效（如 Oracle、PostgreSQL 等）。要了解更多关于 keyColumn 和 keyProperty 可选值信息，请查看“insert, update 和 delete”一节。
                         * 
                         */
                        enum FlushCachePolicy {
                          DEFAULT,
                          TRUE,
                          FALSE
                        }
                        boolean useCache() default true;
                      
                        FlushCachePolicy flushCache() default FlushCachePolicy.DEFAULT;
                      
                        ResultSetType resultSetType() default ResultSetType.DEFAULT;
                      
                        StatementType statementType() default StatementType.PREPARED;
                      
                        int fetchSize() default -1;
                      
                        int timeout() default -1;
                      
                        boolean useGeneratedKeys() default false;
                      
                        String keyProperty() default "";
                      
                      
                        String keyColumn() default "";
                      
                      
                        String resultSets() default "";
                      }
                      ```
                  
                    - assistant的addMappedStatement方法
                  
                      ```java
                      public MappedStatement addMappedStatement(
                            String id,
                            SqlSource sqlSource,
                            StatementType statementType,
                            SqlCommandType sqlCommandType,
                            Integer fetchSize,
                            Integer timeout,
                            String parameterMap,
                            Class<?> parameterType,
                            String resultMap,
                            Class<?> resultType,
                            ResultSetType resultSetType,
                            boolean flushCache,
                            boolean useCache,
                            boolean resultOrdered,
                            KeyGenerator keyGenerator,
                            String keyProperty,
                            String keyColumn,
                            String databaseId,
                            LanguageDriver lang,
                            String resultSets) {
                      
                          if (unresolvedCacheRef) {
                            throw new IncompleteElementException("Cache-ref not yet resolved");
                          }
                      
                          id = applyCurrentNamespace(id, false);
                          boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
                      
                          MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
                              .resource(resource)
                              .fetchSize(fetchSize)
                              .timeout(timeout)
                              .statementType(statementType)
                              .keyGenerator(keyGenerator)
                              .keyProperty(keyProperty)
                              .keyColumn(keyColumn)
                              .databaseId(databaseId)
                              .lang(lang)
                              .resultOrdered(resultOrdered)
                              .resultSets(resultSets)
                              .resultMaps(getStatementResultMaps(resultMap, resultType, id))
                              .resultSetType(resultSetType)
                              .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
                              .useCache(valueOrDefault(useCache, isSelect))
                              .cache(currentCache);
                      
                          ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
                          if (statementParameterMap != null) {
                              //为MappedStatement简单赋值
                            statementBuilder.parameterMap(statementParameterMap);
                          }
                      
                          MappedStatement statement = statementBuilder.build();
                          configuration.addMappedStatement(statement);
                          return statement;
                        }
                      ```
                      
                      - getStatementResultMaps方法
                      
                        ```java
                        private List<ResultMap> getStatementResultMaps(
                            String resultMap,
                            Class<?> resultType,
                            String statementId) {
                            resultMap = applyCurrentNamespace(resultMap, true);
                        
                            List<ResultMap> resultMaps = new ArrayList<>();
                            if (resultMap != null) {
                                String[] resultMapNames = resultMap.split(",");
                                for (String resultMapName : resultMapNames) {
                                    try {
                                        resultMaps.add(configuration.getResultMap(resultMapName.trim()));
                                    } catch (IllegalArgumentException e) {
                                        throw new IncompleteElementException("Could not find result map '" + resultMapName + "' referenced from '" + statementId + "'", e);
                                    }
                                }
                            } else if (resultType != null) {
                                ResultMap inlineResultMap = new ResultMap.Builder(
                                    configuration,
                                    statementId + "-Inline",
                                    resultType,
                                    new ArrayList<>(),
                                    null).build();
                                resultMaps.add(inlineResultMap);
                            }
                            return resultMaps;
                        }
                        ```
                      
                      - getStatementParameterMap方法
                      
                        ```java
                        private ParameterMap getStatementParameterMap(
                            String parameterMapName,
                            Class<?> parameterTypeClass,
                            String statementId) {
                            parameterMapName = applyCurrentNamespace(parameterMapName, true);//获取parameterMapName和Namespace的结合
                            ParameterMap parameterMap = null;
                            if (parameterMapName != null) {
                                try {
                                    parameterMap = configuration.getParameterMap(parameterMapName);
                                } catch (IllegalArgumentException e) {
                                    throw new IncompleteElementException("Could not find parameter map " + parameterMapName, e);
                                }
                            } else if (parameterTypeClass != null) {
                                List<ParameterMapping> parameterMappings = new ArrayList<>();
                                parameterMap = new ParameterMap.Builder(
                                    configuration,
                                    statementId + "-Inline",
                                    parameterTypeClass,
                                    parameterMappings).build();//如果不指定ParameterMapper则创建内联参数Mapper
                            }
                            return parameterMap;
                        }
                        ```
                      
                      - MappedStatement.Builder的build方法
                      
                        ```java
                        public MappedStatement build() {
                            assert mappedStatement.configuration != null;
                            assert mappedStatement.id != null;
                            assert mappedStatement.sqlSource != null;
                            assert mappedStatement.lang != null;
                            mappedStatement.resultMaps = Collections.unmodifiableList(mappedStatement.resultMaps);//设置ResultMap为不可修改集合
                            return mappedStatement;
                        }
                        ```
                      
                        
                  
                  - addIncompleteMethod方法
                  
                    ```java
                    private void parsePendingMethods() {
                        Collection<MethodResolver> incompleteMethods = configuration.getIncompleteMethods();//获取未完全解析的method集合
                        synchronized (incompleteMethods) {
                            Iterator<MethodResolver> iter = incompleteMethods.iterator();
                            while (iter.hasNext()) {//重新解析该集合中的所有method
                                try {
                                    iter.next().resolve();//重新调用parseStatement方法
                                    iter.remove();//从集合中移除
                                } catch (IncompleteElementException e) {
                                    // This method is still missing a resource
                                }
                            }
                        }
                    }
                    ```
                  
                    