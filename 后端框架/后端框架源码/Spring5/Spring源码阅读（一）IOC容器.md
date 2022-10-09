# Spring源码（一）IOC容器

- ## 程序入口

  ```java
  ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
  ```

- 初始化AbstractApplicationContext

  ```java
  private static final boolean shouldIgnoreSpel = SpringProperties.getFlag("spring.spel.ignore");//从SpringProperties中获取spring.spel.ignore的值（true或者false）
  
  
  static {
      // 急切地加载 ContextClosedEvent 类以避免在 WebLogic 8.1 中关闭应用程序时出现奇怪的类加载器问题。
      // on application shutdown in WebLogic 8.1. (Reported by Dustin Woods.)
      ContextClosedEvent.class.getName();//获取ContextClosedEvent的class类
  }
  ```

- 调用AnnotationConfigApplicationContext的构造方法

  ```java
  public AnnotationConfigApplicationContext() {
      StartupStep createAnnotatedBeanDefReader = this.getApplicationStartup().start("spring.context.annotated-bean-reader.create");
      this.reader = new AnnotatedBeanDefinitionReader(this);//创建AnnotatedBeanDefinitionReader
      createAnnotatedBeanDefReader.end();//设置启动阶段结束（什么都不做仅起标记作用）
      this.scanner = new ClassPathBeanDefinitionScanner(this);//创建类Bean扫描器
  }
  ```

  