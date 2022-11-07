# Spring5 AnnotationConfigApplicationContext容器

- 基本类图

  ![image-20221024145706329](C:\Users\YX\AppData\Roaming\Typora\typora-user-images\image-20221024145706329.png)

- ## Resource接口(资源文件的加载类)

  ```java
  public interface ResourceLoader {//加载资源文件的类
      String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;//某些资源文件路径前含有classpath:,之后将会去除
      Resource getResource(String location);//通过Resource包装指定位置的资源
      @Nullable
      ClassLoader getClassLoader();
  
  }
  
  ```

  - 默认实现为DefaultResourceLoader

  - 含有扩展接口ResourcePatternResolver(复数资源文件的读取)

    ```java
    public interface ResourcePatternResolver extends ResourceLoader {
    	Resource[] getResources(String locationPattern) throws IOException;
    }
    ```

- ## BeanFactory

  - Spring最基础的类，容器的最底层实现(以下值保留基础方法)

    ```java
    public interface BeanFactory {
    
    	String FACTORY_BEAN_PREFIX = "&";
    
    	Object getBean(String name) throws BeansException;
    
    	@Nullable
    	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
    
    	@Nullable
    	Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;
    
    	String[] getAliases(String name);
    
    }
    ```

    主要负责从容器中获取Bean的实例和实例化Bean

  - ### ListableBeanFactory接口

    ```java
    public interface ListableBeanFactory extends BeanFactory {
    
    	String[] getBeanDefinitionNames();//只是返回Bean名称，和BeanDefinition暂时没有关系
    
    	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType, boolean allowEagerInit);
    
    	String[] getBeanNamesForType(@Nullable Class<?> type, boolean includeNonSingletons, boolean allowEagerInit);
    
    	<T> Map<String, T> getBeansOfType(@Nullable Class<T> type) throws BeansException;
    
    
    	Map<String, Object> getBeansWithAnnotation(Class<? extends Annotation> annotationType) throws BeansException;
    
    
    	@Nullable
    	<A extends Annotation> A findAnnotationOnBean(
    			String beanName, Class<A> annotationType, boolean allowFactoryBeanInit)
    			throws NoSuchBeanDefinitionException;
    
    }
    
    ```

    BeanFactory的扩展，可以列出所有Bean，可根据类型列出Bean的集合，可根据注解查找Bean，该类实现类DefaultListableBeanFactory

  - ### HierarchicalBeanFactory接口

    - 分层的BeanFactory,

      ```java
      public interface HierarchicalBeanFactory extends BeanFactory {
      
          @Nullable
          BeanFactory getParentBeanFactory();
      
          boolean containsLocalBean(String name);
      
      }
      
      ```

  - ## AutowireCapableBeanFactory接口

    ```java
    public interface AutowireCapableBeanFactory extends BeanFactory {
    
    
    	int AUTOWIRE_NO = 0;
    	int AUTOWIRE_BY_NAME = 1;
    	int AUTOWIRE_BY_TYPE = 2;
    	int AUTOWIRE_CONSTRUCTOR = 3;
    	@Deprecated
    	int AUTOWIRE_AUTODETECT = 4;
    
    	String ORIGINAL_INSTANCE_SUFFIX = ".ORIGINAL";
    
    	<T> T createBean(Class<T> beanClass) throws BeansException;
    
    	Object configureBean(Object existingBean, String beanName) throws BeansException;
    
    	Object createBean(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException;
    
    	Object autowire(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException;
    
    	void autowireBeanProperties(Object existingBean, int autowireMode, boolean dependencyCheck)
    			throws BeansException;
    
    	void applyBeanPropertyValues(Object existingBean, String beanName) throws BeansException;
    
    	Object initializeBean(Object existingBean, String beanName) throws BeansException;
    
    	Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
    			throws BeansException;
    
    	Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    			throws BeansException;
    
    	void destroyBean(Object existingBean);
    
    	<T> NamedBeanHolder<T> resolveNamedBean(Class<T> requiredType) throws BeansException;
    
    	Object resolveBeanByName(String name, DependencyDescriptor descriptor) throws BeansException;
    
    	@Nullable
    	Object resolveDependency(DependencyDescriptor descriptor, @Nullable String requestingBeanName) throws BeansException;
    
    	@Nullable
    	Object resolveDependency(DependencyDescriptor descriptor, @Nullable String requestingBeanName,
    			@Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException;
    
    }
    
    ```

    支持Bean属性自动注入的BeanFactory，该类可以解析Bean的依赖关系，执行Bean后置处理器（Bean实例化前后），显式的Bean创建方法以及自动注入策略的声明	
    
  - ConfigurableListableBeanFactory接口（ListableBeanFactory和AutowireCapableBeanFactory的共同扩展接口），默认实现类是DefaultListableBeanFactory类，该接口主要是ListableBeanFactory接口和AutowireCapableBeanFactory接口的综合

    

- ## EnvironmentCapable、Environment、MessageSourc、ApplicationEventPublisher接口

  - EnvironmentCapable：获取当前环境（Environment类）
  - Environment：包装当前环境的一下属性配置信息
  - MessageSource：跨国语言所用
  - ApplicationEventPublisher:事件发布器，只有publishEvent方法（用于发布事件）

- ## ApplicationContext接口

  - 核心接口，使用装饰器模式通过实现接口和收纳接口的实现类实现功能的集合

    ```java
    public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
    		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    
    	@Nullable
    	ApplicationContext getParent();//分层容器
    
    	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;//获取内部BeanFactory
    
    }
    ```

- ## Lifecycle

  - Lifecycle(标记作用)

  ```java
  public interface Lifecycle {
  
  	void start();
  
  	void stop();
  
  	boolean isRunning();
  
  }
  
  ```

- ## ConfigurableApplicationContext接口

  - Application的进一步实现

    ```java
    public interface ConfigurableApplicationContext extends ApplicationContext, Lifecycle, Closeable {
    
    	void setParent(@Nullable ApplicationContext parent);
    
    	void setEnvironment(ConfigurableEnvironment environment);
    
    	@Override
    	ConfigurableEnvironment getEnvironment();
    
    	void setApplicationStartup(ApplicationStartup applicationStartup);
    
    	ApplicationStartup getApplicationStartup();
    
    	void addBeanFactoryPostProcessor(BeanFactoryPostProcessor postProcessor);
    
    	void addApplicationListener(ApplicationListener<?> listener);
    
    	void setClassLoader(ClassLoader classLoader);
    
    	void addProtocolResolver(ProtocolResolver resolver);
    
    	void refresh() throws BeansException, IllegalStateException;
    
    	void registerShutdownHook();
    
    	@Override
    	void close();
    
    	boolean isActive();
    
    	ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;
    
    }
    
    ```

    内部采用ConfigurableListableBeanFactory作为Bean容器
  
  - AbstractApplicationContext虚类
  
    基本完成了Spring容器的大部分基本功能，少量的资源获取等方法由子类实现。
  
  - BeanDefinitionRegistry接口
  
    ```java
    public interface BeanDefinitionRegistry extends AliasRegistry {
    
       void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
             throws BeanDefinitionStoreException;
    
       void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
    
       BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
    
       boolean containsBeanDefinition(String beanName);
    
       String[] getBeanDefinitionNames();
    
       int getBeanDefinitionCount();
    
       boolean isBeanNameInUse(String beanName);
    
    }
    ```
  
    用于保存bean定义的注册中心的接口，例如RootBeanDefinition和ChildBeanDefinition实例。通常由内部使用AbstractBeanDefinition层次结构的beanfactory实现。实现类只有DefaultListableBeanFactory和GenericApplicationContext。
  
  - GenericApplicationContext类
  
    内部使用DefaultListableBeanFactory和ResourceLoader作为成员变量，使用装饰模式实现Application的方法
  
  - AnnotationConfigApplicationContext的子类，实现了AnnotationConfigRegistry接口，该子类相当于GenericApplicationContext的外部扩展用于完善注解的开发
  
    - AnnotationConfigRegistry接口(功能：通过注解注册Bean)
  
      ```java
      public interface AnnotationConfigRegistry {
      
      	void register(Class<?>... componentClasses);
      
      	void scan(String... basePackages);
      
      }
      ```
  
      

BeanDefinition类图

![image-20221024165022600](C:\Users\YX\AppData\Roaming\Typora\typora-user-images\image-20221024165022600.png)
