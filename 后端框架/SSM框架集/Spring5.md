# Spring5

- ## 入门案例

  - 1.通过IDEA添加Spring5框架

    [添加教程]https://blog.csdn.net/mmmm0584/article/details/114453819

  - 2.编写测试类

    ```java
    package com.java.Springtext;
    
    public class text01 {
        public void fuck()
        {
            System.out.println("fucking...high");
        }
    }
    
    ```

  - 3.编写xml文件（文件放置在项目根目录中，就是常用的Src中）

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id ="fuck" class="com.java.Springtext.text01"></bean> <!-->只有这一句是自己写的<-->
       
    </beans>
    ```

  - 4.编写测试Demo

    ```java
    package com.java.Springtext;
    
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.support.ClassPathXmlApplicationContext;
    
    public class Springtext {
        public static void main(String[] args) {
            ApplicationContext context= new ClassPathXmlApplicationContext("springtext.xml");
            //载入XML文件
           text01 fuck= context.getBean("fuck", text01.class);
            //通过XML文件获得测试类
           fuck.fuck();
            //使用测试类
        }
    }
    
    ```

    

    

    ------------------------



- ## IOC

  - ### 基本原理

    - 通过IOC容器实现
  
  - ### IOC实现底层原理
  
    - XML配置文件
    - 工厂模式类（IOC容器底层实现）

    - 反射机制

  - ### 过程(模拟)
  
    - XML文件，配置创建对象

      ```xml
      <bean id="User" class="类的全名，包括路径"></bean>
      ```
  
    - 工厂模式（类）
  
      ```java
      class Userfactory
      {
          public User getUser()
          {
              String classname="xml解析得到的结果";
              Class user=class.forname(classname);
              return (User)user.newInstance();
          }
      }
      ```
  
  - ### IOC 容器
  
    - #### 实现的两种方式（Spring5提供的接口）
  
      - ##### BeanFactory接口
  
        - Spring自带的接口，实现IOC的基本接口，一般不提供给开发人员使用
        
      - ##### ApplicationContext接口
      
        - BeanFactory的一个子接口，功能更加强大，提供给开发人员使用
        - 两个实现类
          - ClassPathXmlApplicationContext
            - 配置XML文件时默认根目录是SRC下，即项目根目录下
          - FileSystemXmlApplicationContext
            - 配置XML文件时默认根目录是C盘，即XML需在C盘，且编写程序需提供该文件在C盘的全路径
      
      - **两者区别**
      
        - BeanFactory在获取XML文件时不会创建对象，在使用对象时才会创建，ApplicationContext会创建
      
    - #### Bean管理
    
      - 过程
        - 创建对象
        - 注入属性
        
      - 方式一：基于xml配置文件方式实现
        - （1）创建对象：使用bean标签（案例一所使用的方式），创建对象时默认采用无参构造方法，如果没有无参构造方法会报错
        
        - （2）注入属性：DI（依赖注入）
        
          - **方法一：通过set方法注入**
        
            第一步：创建类并提供一个set方法
        
            ```java
            public class text01 {
                private String hello;
                private String world;
            
                public void setHello(String hello) {
                    this.hello = hello;
                }
            
                public void setWorld(String world) {
                    this.world = world;
                }
            
                public void fuck()
                {
                    System.out.println(hello+"\n"+world);
                }
            }
            
            ```
        
            第二步：在bean标签中添加property标签
        
            ```xml
            <bean id ="fuck" class="com.java.Springtext.text01">
                    <property name="hello" value="kick you ass!!!"/>
                    <property name="world" value="kick you ass!!!"/>
                </bean> 
            ```
        
            第三步：同案例方式调用测试类
          
          - **方法二：通过有参构造方法注入属性**
          
            - 第一步：编写一个带有有参构造方法的类
          
              ```java
              
              public class text01 {
                  private String hello;
                  private String world;
              
                  public text01(String hello, String world) {
                      this.hello = hello;
                      this.world = world;
                  }
              
                  public void fuck()
                  {
                      System.out.println(hello+"\n"+world);
                  }
              }
              
              ```
          
            - 第二步：编写xml文件
          
              ```xml
              <bean id ="fuck" class="com.java.Springtext.text01">
                      <constructor-arg name="hello" value="傻逼自动提示"></constructor-arg>
                      <constructor-arg name="world" value="真tm傻逼"></constructor-arg>
                      <!-->
                       <constructor-arg index="1" value="真tm傻逼"></constructor-arg>
                      这样写也可以，index的值为0则对有参构造方法中的第一个参数注入属性
                      1则为第二个
                      <-->
                     
                  </bean> 
              ```
          
            - 第三步：同案例运行测试类
          
          - **方法三：P名称空间注入（通过set方法操作）**
          
            - 编写一个带set方法的类
          
            - 修改xml文件
          
              ```xml
              
              <?xml version="1.0" encoding="UTF-8"?>
              <beans xmlns="http://www.springframework.org/schema/beans"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                     xmlns:p="http://www.springframework.org/schema/p"<!-- 在这里添加一个p标签-->
                     xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
                  <bean id ="fuck" class="com.java.Springtext.text01" p:hello="kick you ass" p:world="kick you ass">
                  </bean><!-- 通过P标签注入属性-->
              
              </beans>
              ```
          
            - 测试类
          
          - 如何给属性注入特殊值
          
            ```xml
            <property name="hello" >
                        <null/>
                    </property>
            ```
          
            - 如果注入的属性具有特殊符号如<kick you ass>
          
              ```xml
              <property name="hello" >
                         <value><![CDATA[<kick you ass>]]></value>
                      </property>
              ```
          
              通过CDATA方式解决格式为:<![[属性值]]>
          
          - 如何注入类
          
            - 编写测试类
          
            - 例：假设text01中含有son_of_text01成员
          
              ```java
              public class son_of_text01 {
                  private String gg;
                  private String mm;
              
                  public void setGg(String gg) {
                      this.gg = gg;
                  }
              
                  public void setMm(String mm) {
                      this.mm = mm;
                  }
                  void gg()
                  {
                      System.out.println(gg +" "+mm);
                  }
              }
              ```
          
              ```java
              public class text01 {
                  private String hello;
              
                  private son_of_text01 son;
              
                  public void setHello(String hello) {
                      this.hello = hello;
                  }
              
                  public void setSon(son_of_text01 son) {
                      this.son = son;
                  }
              
                  public void fuck()
                  {
                      System.out.println(hello);
                      son.gg();
                  }
              }
              
              ```
          
              - 编写xml文件
          
                ```xml
                
                    <bean id="son" class="com.java.Springtext.son_of_text01">
                        <property name="gg" value="飞 欸 飞"></property>
                        <property name="mm" value="芜湖"></property>
                    </bean>
                
                    <bean id="fuck" class="com.java.Springtext.text01">
                        <property name="hello" value="肉蛋充饥"></property>
                        <property name="son" ref="son"></property>
                
                    </bean>
                ```
          
                在xml文件中先注入son类，再将son类注入到text01中
          
            - 也可以使用内部bean写法
          
              ```xml
              <bean id="fuck" class="com.java.Springtext.text01">
                      <property name="hello" value="肉蛋充饥"></property>
                      <property name="son" >
                          <bean id="son" class="com.java.Springtext.son_of_text01">
                              <property name="gg" value="飞 欸 飞"></property>
                              <property name="mm" value="芜湖"></property>
                          </bean>
                      </property>
              
                  </bean>
              ```
          
            - 或者通过get方法先生成内部类
          
              - text01
          
                ```java
                public class text01 {
                    private String hello;
                
                    private son_of_text01 son;
                
                    public son_of_text01 getSon() {
                        return son;
                    }
                
                    public void setHello(String hello) {
                        this.hello = hello;
                    }
                
                    public void fuck()
                    {
                        System.out.println(hello);
                        son.gg();
                    }
                
                    public void setSon(son_of_text01 son) {
                        this.son = son;
                    }
                }
                
                ```
          
              - xml文件
          
                ```xml
                <bean id="fuck" class="com.java.Springtext.text01">
                    <property name="hello" value="肉蛋充饥"></property>
                    <property name="son" ref="son"></property>
                    <property name="son.gg" value="飞 欸 飞"></property>
                    <property name="son.mm" value="芜湖"></property>
                </bean>
                <bean id="son" class="com.java.Springtext.son_of_text01">
                    <property name="gg" value="gg"></property>
                    <property name="mm" value="mm"></property>
                </bean>
                ```
          
                执行顺序为先执行id为son的bean后执行id为fuck的bean
          
              - 通过xml方式注入集合或数组
          
                - xml文件编写
          
                  ```xml
                  <bean id="fuck" class="com.java.Springtext.text01">
                          <property name="s">
                              <array>
                                  <value>芜湖</value>
                                  <value>起飞</value>
                              </array>
                          </property>
                  
                          <property name="s2">
                              <list>
                                  <value>肉蛋充饥</value>
                                  <value>接下来我将一次不死并且完成超神</value>
                              </list>
                          </property>
                  
                          <property name="map">
                              <map>
                                  <entry key="1" value="kick you ass"></entry>
                                  <entry key="2" value="asshole"></entry>
                              </map>
                          </property>
                  
                      </bean>
                  ```
          
                  list标签和array标签效果相同，List集合和Set集合用list标签和Set标签即可
          
                - 如果将数组元素更换为对象
          
                  - xml文件编写
          
                    ```xml
                    <bean id="fuck" class="com.java.Springtext.text01">
                            <property name="set">
                                <set>
                                    <ref bean="son1"></ref>
                                    <ref bean="son2"></ref>
                                    <ref bean="son3"></ref>
                                </set>
                            </property>
                    
                        </bean>
                        <bean id="son1" class="com.java.Springtext.son_of_text01">
                            <property name="gg" value="肉蛋充饥"></property>
                            <property name="mm" value="芜湖起飞"></property>
                        </bean>
                    
                        <bean id="son2" class="com.java.Springtext.son_of_text01">
                            <property name="gg" value="sb"></property>
                            <property name="mm" value="上票"></property>
                        </bean>
                    
                        <bean id="son3" class="com.java.Springtext.son_of_text01">
                            <property name="gg" value="我是闪电！！！"></property>
                            <property name="mm" value="死亡如风，常伴吾身"></property>
                        </bean>
                    ```
          
                - 使用util命名空间注入（和p标签空间注入相似）
          
                  ```xml
                  <?xml version="1.0" encoding="UTF-8"?>
                  <beans xmlns="http://www.springframework.org/schema/beans"
                         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                         xmlns:util="http://www.springframework.org/schema/util"
                  
                      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">此处有修改
                  
                      <util:set id="set">
                          <ref bean="son1"></ref>
                          <ref bean="son2"></ref>
                          <ref bean="son3"></ref>
                      </util:set>此处有修改
                  
                  
                  
                      <bean id="fuck" class="com.java.Springtext.text01">
                          <property name="set" ref="set"></property>
                      </bean>此处有修改
                  
                  
                      <bean id="son1" class="com.java.Springtext.son_of_text01">
                          <property name="gg" value="肉蛋充饥"></property>
                          <property name="mm" value="芜湖起飞"></property>
                      </bean>
                  
                      <bean id="son2" class="com.java.Springtext.son_of_text01">
                          <property name="gg" value="sb"></property>
                          <property name="mm" value="上票"></property>
                      </bean>
                  
                      <bean id="son3" class="com.java.Springtext.son_of_text01">
                          <property name="gg" value="我是闪电！！！"></property>
                          <property name="mm" value="死亡如风，常伴吾身"></property>
                      </bean>
                  </beans>
                  ```
          
              - 工厂bean
              
                - xml文件
              
                  ```xml
                    <bean id="bean" class="com.java.Springtext.text01"></bean>
                  ```
              
                  - 工厂类
              
                    ```java
                    public class text01  implements FactoryBean<String> {
                    
                    
                        @Override
                        public String getObject() throws Exception {
                            return "xxxxx";
                        }
                    
                        @Override
                        public Class<?> getObjectType() {
                            return null;
                        }
                    
                        @Override
                        public boolean isSingleton() {
                            return FactoryBean.super.isSingleton();
                        }
                    }
                    
                    ```
              
                    工厂类可以使bean返回不同的类型
              
                - 多实例bean
              
                  - spring 在生成对象时默认是单实例的
              
                    ```java
                    ApplicationContext context=new ClassPathXmlApplicationContext("springtext02.xml");
                            son_of_text01 s1=context.getBean("bean", son_of_text01.class);
                            son_of_text01 s2=context.getBean("bean",son_of_text01.class);
                            s1.setGg("12123");
                            s1.gg();
                            s2.gg();
                    
                    //如果修改s1的值，s2的值同样会被修改，即bean返回的是一个引用
                    ```
              
                  - 在xml配置文件中可以修改为多实例生成对象
              
                    ```xml
                    <bean id="bean" class="com.java.Springtext.text01" scope="prototype"></bean>
                    ```
              
                    - scope属性可以修改生成对象方式，单实例（即默认）为singleton,多实例为prototype为多实例，单实例方式生成对象是在context创建时生成一个对象，而多实例对象则是在getbean（）方法调用时生成且每次生成的对象不同
              
                - bean的生命周期
              
                  - （1）通过构造器构造bean实例（调用无参构造方法构造类）
              
                  - （2）调用set方法注入属性
              
                  - （3）调用bean实例初始化方法（该方法可以在xml文件中设置）
              
                  - （4）bean返回需要的类
              
                  - （5）IOC容器关闭，bean调用结束方法，bean生命周期结束（可以在xml文件中设置具体方法）
              
                    - 案例实现
              
                      - 测试类
              
                        ```java
                        public class text01   {
                        
                        
                            String a;
                        
                            public text01() {
                                System.out.println("（1）通过构造器构造bean实例（调用无参构造方法构造类）");
                            }
                        
                            public void setA(String a) {
                                System.out.println("（2）调用set方法注入属性");
                                this.a = a;
                            }
                        
                            void init()
                            {
                                System.out.println("（3）调用bean实例初始化方法（该方法可以在xml文件中设置）");
                            }
                        
                            void close()
                            {
                                System.out.println("（5）IOC容器关闭，bean调用结束方法，bean生命周期结束（可以在xml文件中设置具体方法）");
                            }
                        }
                        
                        ```
              
                      - xml文件
              
                        ```xml
                        <bean id="bean" class="com.java.Springtext.text01"  init-method="init" destroy-method="close">
                            <property name="a" value="kksk"></property>
                        </bean>
                        ```
              
                      - 运行类
              
                        ```java
                        public class Springtext {
                            public static void main(String[] args) {
                                ClassPathXmlApplicationContext context=new ClassPathXmlApplicationContext("springtext02.xml");
                                text01 t=context.getBean("bean",text01.class);
                                System.out.println("（4）bean返回需要的类");
                                context.close();//主动关闭bean
                        
                            }
                        }
                        ```
              
                      - 输出结果
              
                        ```java
                        （1）通过构造器构造bean实例（调用无参构造方法构造类）
                        （2）调用set方法注入属性
                        （3）调用bean实例初始化方法（该方法可以在xml文件中设置）
                        （4）bean返回需要的类
                        （5）IOC容器关闭，bean调用结束方法，bean生命周期结束（可以在xml文件中设置具体方法）
                        ```
              
                        - 需要注意的是bean如果生成对象方式为多实例生成，则在close方法后不会输出第五条，也许是对于多实例的对象bean无法知道执行哪个对象的方法，尽管每个实例的方法可能都是一样的。
              
                    - 除了以上五个步骤之外还有两个步骤在第三步前后通过后置处理器执行
              
                      - xml文件
              
                        ```xml
                        <bean id="bean" class="com.java.Springtext.text01"  init-method="init" destroy-method="close">
                            <property name="a" value="kksk"></property>
                        </bean>
                        
                        <bean id="bean2" class="com.java.Springtext.post"></bean>
                        ```
              
                      - 后置处理器类
              
                        ```java
                        public class post implements BeanPostProcessor {
                            @Override
                            public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
                                System.out.println("后置处理器初始化操作");
                                return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
                            }
                        
                            @Override
                            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                                System.out.println("后置处理器结束化操作");
                                return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
                            }
                        }
                        ```
              
                      - 结果输出
              
                        ```java
                        （1）通过构造器构造bean实例（调用无参构造方法构造类）
                        （2）调用set方法注入属性
                        后置处理器初始化操作
                        （3）调用bean实例初始化方法（该方法可以在xml文件中设置）
                        后置处理器结束化操作
                        （4）bean返回需要的类
                        （5）IOC容器关闭，bean调用结束方法，bean生命周期结束（可以在xml文件中设置具体方法）
                        ```
              
                        - 设计一个类，实现BeanPostProcessor类，更改其中的两个方法可以在调用bean实例化方法前后执行，同时所有再xml文件中的其他bean都会受该后置处理器影响
              
                  
                  - spring属性自动装配
                  
                    -   autowirie属性
                  
                      - byname
                  
                        - 根据名称注入属性（外部引入的bean的id需要和注入的属性的名字相同）
                  
                          ```xml
                          <bean id="bean" class="com.java.Springtext.text01"  autowire="byName">
                              </bean>
                          
                              <bean id="son" class="com.java.Springtext.son_of_text01" >
                                  <property name="gg" value="fuck"></property>
                                  <property name="mm" value="you"></property>
                              </bean>
                          ```
                  
                          第二个bean的id值需要和text01中son_of_text01的名称相同。
                  
                      - byType
                  
                        - 根据类型注入属性
                  
                          ```xml
                          <bean id="bean" class="com.java.Springtext.text01"  autowire="byType">
                              </bean>
                          
                              <bean id="son" class="com.java.Springtext.son_of_text01" >
                                  <property name="gg" value="fuck"></property>
                                  <property name="mm" value="you"></property>
                              </bean>
                          ```
                  
                          - 注意：如果根据类型自动装配，那么只能有一个外部bean，如果有两个会报错
                  
                    - 外部xml文件注入
      

----------------------------------------------------------------

- ### 通过注解方式注入属性

  - #### 注解方式创建对象常用注解
  
    - @Component
    - @Service
    - @Controller
    - @Repository
  
  - 基于注解方式案例
  
    - ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context"//在此处添加一个新的空间，将beans改为context
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">//添加一个新的url同样将beans改为
      context
          
         
          <context:component-scan base-package="com.java.springannotation"></context:component-scan>
      </beans>
      <!--开启包扫描,对该包下的所有类进行扫描，寻找注解-->
      ```
  
      - 除此以外可以选择不同的扫描方式
  
        ```xml
        <context:component-scan base-package="com.java.springannotation" use-default-filters="false">
                <context:include-filter type="annotation" expression="org.springframework.stereotype.Component"/>
            </context:component-scan>
        	
        <!--设置属性值use-default-filters为false，这会使得扫描需要指定，在下方添加include-filter标签，使类型为注解以及expression(类名)是Component才会扫描-->
        
        
            <context:component-scan base-package="com.java.springannotation">
                <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Component"/>
            </context:component-scan>
        
        <!--默认属性值use-default-filters为true，将会扫描指定的包下所有类，在下方添加exclude-filter标签，使类型为注解以及expression(类名)是Component不会扫描-->
        
        ```
  
        
  
    - 被创建的类
  
      ```java
      @Component(value = "text01")//相当于<bean id=“text01” class="">中的id，包括Component以及其他三种注解均可
      public class text01 {
          public void gg()
          {
              System.out.println("do something!!!");
          }
      }
      ```
  
    - 测试创建类
  
      ```java
      public class springtext {
          public static void main(String[] args) {
              ApplicationContext context = new ClassPathXmlApplicationContext("spring02.xml");
              text01 t=context.getBean("text01", text01.class);
              t.gg();
          }
      }
      ```
  
      - 注意：当前spring5框架不支持jdk16
    
  - #### 注解方式注入属性常用注解
  
    - Autowired
  
      - 根据类型注入属性
  
        ```java
        @Autowired
        private text02 a;
        ```
  
        - a可以在xml文件中以bean标签的方式出现也可以通过注解生成
  
    - Qualifier
  
      - 根据名称注入
  
        ```java
        @Autowired
        @Qualifier("text02")
        private text02 a;
        ```
  
        - Autowired和Qualifier必须同时使用，否则Qualifier默认为null注入
  
    - Resource
  
      - 以上两者的结合
  
        ```java
        @Resource(name = "text02")
        private text02 a;
        
        @Resource
        private text02 a;
        ```
  
        - 不添加name则为按类型注入。Resource为javax包下的，并不是Spring中的
  
    - Value
  
      - 普通类型（基本类型）注入，以上三种都是注入对象
  
        ```java
        @Value("hhh")
        String a;
        ```
  
  - #### 完全注解注入（取代xml文件）
  
    - 配置类
  
      - @Configuration注解
  
        - 指定某个类为配置类
  
      - @ComponentScan注解
  
        - 实现扫描功能
  
        ```java
        @Configuration
        @ComponentScan(basePackages={"com.java.springannotation"})
        public class springconfig {
            
        }
        ```
  
        - 同时需要修改测试的类
  
          ```java
          public class springtext {
              public static void main(String[] args) {
                  ApplicationContext context = new AnnotationConfigApplicationContext("com.java.springannotation.springconfig");//原先这里为指定xml文件地址，改为AnnotationConfigApplicationContext类，指定配置类
                  text01 t=context.getBean("text01", text01.class);
                  t.gg();
              }
          }
          ```
  
          - 执行效果和xml文件相同

- ### AOP

  - 面向切面编程
    - 功能：不修改源代码添加新的功能
    
  - 底层原理
    - （1）有接口：jdk动态代理 
      - 创建接口实现类作为代理对象
    - （2）无接口，通过CGLIB动态代理
      - 创建当前类的子类作为代理对象
    
  - 专业术语
    
    - 连接点
      - 可被增强的方法
    - 切入点
      - 以被增强的方法
    - 通知
      - 被增强的部分（新添加的功能）
      - 前置通知（在方法前）、后置通知（在方法后）、环绕通知（前后都有）、异常通知（管理异常）、最终通知（类似于异常处理中的finally）
    - 切面
      - 添加新功能的动作
    
  - JDK动态代理
  
    - 常用类
  
      - Proxy
  
        - 代理类的实例
  
      - InvocationHandler
  
        - 修改需要代理的类的方法的接口
  
      - 两者的意义
  
        - InvocationHandler是对于需要修改的类的方法的升级
  
        - Proxy是将InvocationHandler方法修改后的得到的对象的实例
  
          ![image-20220318192509754](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220318192509754.png)
  
        - 需要升级的方法（接口表示）
  
          - ```java
            public interface sb {
                String fuck();
            }
            ```
  
        - 初步实现方法的类（该方法的原始版本）
  
          - ```java
            class gg implements sb
            {
                public String fuck()
                {
                    System.out.println("fuck you ass");
                    return "什么傻逼AOP";
                }
            }
            ```
  
        - InvocationHandler（改造方法的工厂）
  
          - ```java
            public class  InvocationHandleriml implements InvocationHandler{
            
                Object obj;//初步实现方法的类，也是需要改造方法的类
            
                public void setObj(Object obj) {
                    this.obj = obj;
                }
            
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    System.out.println("proxy到底有个√八用啊");
                   return  method.invoke(obj, args);
                }
            }
            
            ```
  
        - Proxy的应用(改造后的类的实例对象，也就是对改造完成的类的封装)
  
          ```java
          public class aoptext {
              public static void main(String[] args) throws NoSuchMethodException {
                  gg gg = new gg();
                  InvocationHandleriml handler = new InvocationHandleriml();
                  handler.setObj(gg);
                  sb t= (sb) Proxy.newProxyInstance(gg.getClass().getClassLoader(), gg.getClass().getInterfaces(),handler);
                  t.fuck();
              }
          }
          ```
  
          - 结果输出
  
            ```java
            proxy到底有个√八用啊
            fuck you ass
            ```
  
  - Spring AOP（基于Aspectj框架和spring框架实现）
  
    - AspectJ
  
      - 面向切面框架
  
    - 切入点表达式
  
      ```java
      execution([权限修饰符][返回类型][类全路径][方法名称][方法名称]([参数列表]))
      ```
  
      - 例
  
        ```java
         execution(* int com.java.springtext.text01.add(int i));
         //对某个类中的某个方法切入
        execution(* int com.java.springtext.text01.*(参数));
        //对某个类中的所有方法切入
        execution(* int com.java.springtext.*.*(参数));
        //对某个包下的所有类的所有方法切入
        ```
  
    - 注解方式实现AOP
  
      - 通过xml文件配置
  
        - （1）编写xml文件
      
          ```xml
          <?xml version="1.0" encoding="UTF-8"?>
          <beans xmlns="http://www.springframework.org/schema/beans"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xmlns:context="http://www.springframework.org/schema/context"
                 xmlns:aop="http://www.springframework.org/schema/aop"
                 xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                      http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                      http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
          
                  <context:component-scan base-package="com.java.AOP"></context:component-scan>
          
                  <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
          </beans>
          ```
      
          - 在xml文件中添加两个命名空间，context用于扫描注解，aop用于识别Aspect注解并建立代理类
      
        - (2)编写代理类和被代理类
      
          - 代理类
      
          ```java
          import org.aspectj.lang.annotation.Aspect;
          import org.aspectj.lang.annotation.Before;
          import org.springframework.stereotype.Component;
          
          @Component(value="myproxy")
          @Aspect//代表这个类是代理类
          public class myproxy {
          
              @Before("execution(* com.java.AOP.Proxy.user.eat_shit(..))")//表示这个方法是前置通知
              public void before_eat_shit()
              {
                  System.out.println("take shit!!!!!");
              }
              @Before("execution(* com.java.AOP.User.User.bikabika(..))")
              public void go_to_bikabika()
              {
                  System.out.println("OK!!!!");
              }
          }
          ```
      
          - 被代理类
      
            ```java
            //代理类1
            @Component("user")
            public class user {
                public void eat_shit()
                {
                    System.out.println("eating shit!!!!!!");
                }
            }
            
            //代理类2
            @Component("User")
            public class User {
                public void bikabika()
                {
                    System.out.println("bikabika........");
                }
            }
            
            ```
      
      - （3）测试
      
        - ```java
          public class aop_text {
              public static void main(String[] args) {
                  ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
                  user u=context.getBean("user",user.class);
                  User u2=context.getBean("User",User.class);
                  u.eat_shit();
                  u2.bikabika();
              }
          }
          
          ```
      
        - User和user类都得到了加强
      
    - 其余类型通知
    
      - （1）@After：最终通知，无论如何都会执行
    
      - （2）@AfterReturning：后置通知，在值返回后执行
    
      - （3）@AfterThrowing：异常通知，在异常抛出后执行
    
      - （4）@Around：环绕通知，在方法执行前和方法执行后执行
    
        - 执行顺序
    
          - 例
    
            - 代理类
    
              ```java
              @Component(value="myproxy")
              @Aspect//代表这个类是代理类
              public class myproxy {
              
                  @Before("execution(* com.java.AOP.User.User.my_method(..))")
                  public void before()
                  {
                      System.out.println("方法执行前.....");
                  }
              
                  @Around("execution(* com.java.AOP.User.User.my_method(..))")
                  public int around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
                      System.out.println("环绕执行前.....");
                     int k=(int)proceedingJoinPoint.proceed();
                      System.out.println("环绕执行后.....");
                      return k;
                  }
              
                  @AfterThrowing("execution(* com.java.AOP.User.User.my_method(..))")
                  public void afterThrowing()
                  {
                      System.out.println("发生异常后.....");
                  }
              
                  @AfterReturning("execution(* com.java.AOP.User.User.my_method(..))")
                  public void afterReturning()
                  {
                      System.out.println("方法返回值后.....");
                  }
              
                  @After("execution(* com.java.AOP.User.User.my_method(..))")
                  public void after()
                  {
                      System.out.println("方法结束后.....");
                  }
              }
              
              ```
    
              - 输出
    
                ```java
                
                无异常情况：
                环绕执行前.....
                方法执行前.....
                方法执行中........
                方法返回值后.....
                方法结束后.....
                环绕执行后.....
                    
                有异常情况：
                环绕执行前.....
                方法执行前.....
                方法执行中........
                发生异常后.....
                方法结束后.....
                ```
    
                - 执行顺序：环绕前--->before--->方法执行---->异常（如果有的话，后面除了最终通知都不会执行）---->方法返回值后---->最终通知---->环绕结束
    
        - 完全注解实现AOP
    
          - 新添加的注解
    
            - ```java
              @EnableAspectJAutoProxy(proxyTargetClass = true)
              ```
    
              - proxyTargetClass默认为false，改为true可以替代**<aop:aspectj-autoproxy></aop:aspectj-autoproxy>**
    
            - 定义配置类
    
              ```java
              @Configuration
              @ComponentScan(basePackages = "com.java.AOP")
              @EnableAspectJAutoProxy(proxyTargetClass = true)
              public class config {
              }
              ```
    
        - 通过xml文件实现AOP（仅作了解，使用不多）
    
          - ```xml
            <bean id="proxy" class="com.java.AOP.Proxy.proxy02"></bean>
                <bean id="User" class="com.java.AOP.Proxy.User_in_xml"></bean>
            
                <aop:config>
                    <aop:pointcut id="U" expression="execution(* com.java.AOP.Proxy.User_in_xml.eat_shit(..))"/>
                    <aop:aspect ref="proxy">
                        <aop:before method="before" pointcut-ref="U"/>
                    </aop:aspect>
                </aop:config>
            ```
    
            - （1）bean建立代理类和被代理类
            - （2）aop:config标签配置aop文件
            - （3）aop：pointcut标签配置切入点，设置切入点编号（id），通过切入点表达式配置。
            - （4）aop：aspect切面，ref属性引入代理类
            - （5）在代理类中通过aop：before等标签设置前置通知等，pointcut-ref引入切入点
    
  
  ---------
  
  - ##Spring事务
  
    - 事务：一组DDL语句的执行
  
    - 处理场景
  
      - 在Service类中，调用了多个DAO中的DDL方法
      - 使用spring事务处理完成多种事务的提交（使用myabtis，使用jdbc等），也就是spring规范了其他的数据库管理操作
  
    - 事务处理类
  
      - 事务管理器：PlatformTransactionManager接口
  
        - 定义了提交和回滚
        - 对于不同的数据库访问技术spring 中实现了不同的事务管理器
  
        - 如需使用事务管理器，只需要在spring配置文件中声明对应的事务管理器（使用bean标签）
  
    - 不同的事务的处理
  
      - 事务的隔离级别
  
        - 默认DEFAULT:采用数据库默认的事务隔离级别
        - 读未提交
        - 读已提交
        - 可重复读
        - 序列化
  
      - 事务的超时时间
  
        - 表示一个方法的最长执行时间，超过这个时间事务回滚，默认为-1，视为无限长
  
      - 事务的传播行为
  
        - 控制业务方法是否有事务，共有7个，掌握3个   
          - **propagation_required**:要求必须有事务，如果当前有事务则加入，如果没有则创建，最常见，也是spring默认，比如当前有dosome方法和doother方法在执行，dosome开启了事务，而doOther方法在dosome方法中被调用，则doOther会被加入到dosome方法的事务中
          - **propagation_supports**:指定方法支持当前事务，如果当前没有事务，则以非事务方式执行
          - **propagation_requires_new**：总是新建事务，每当一个方法执行就新建一个事务并把当前事务挂起，直到事务执行，比如当前有dosome方法和doother方法，dosome方法中有doOther方法，dosome 先执行事务，执行doOther方法时暂停dosome的事务并新建doOther的事务，直到doOther的事务执行完成后dosome的事务才会继续
  
      - spring对于事务的提交和回滚
  
        - 当方法顺利执行到结束自动提交
        - 当方法运行发生运行时异常，自动回滚
          - 可以通过自定义Runtime异常达到回滚的需求
        - 当方法发生非运行时异常时，自动提交
          - 非运行时异常：需要try/catch的异常
  
      - Transcational注解使用
  
        - @Transcational可以声明事务的隔离级别，事务的超时时间，事务的传播行为，在public的方法上使用，也可以放置在类上，代表该类所有public方法都采用@Transcational
  
        - @Transcational属性
  
          - propagation:设置事务的传播属性
          - isolation：隔离级别
          - readOnly：表示该方法对数据库操作是否只读，默认为false
          - timeout：超时时间，单位为秒，默认为-1
          - rollbackFor：指定需要回滚的异常类，类型为Class[]，默认为空
          - rollbackForClassName：指定需要回滚的异常类类名，类型为String[]，默认为空数组
          - norollbackFor：不需要回滚的异常类
          - norollbackForClassName：不需要回滚的异常类的类名
  
        - 使用步骤
  
          - 声明事务管理器对象
  
            ```xml
            <bean id="transactions" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
                    <property name="dataSource" ref="dataSource"/>
                </bean>
            ```
  
          - 开启注解驱动
  
            - 原理：在业务方法执行前通过环绕通知开启事务，结束后提交或回滚事务
  
            ```xml
            <!--开启事务注解驱动，transaction-manager指定事务管理器的bean-->
            <tx:annotation-driven transaction-manager="transactionManager"/>
            ```
  
          - @Transcational使用
  
            ```java
            @Transactional(
                    propagation = Propagation.REQUIRED,
                    isolation = Isolation.DEFAULT,
                    readOnly = false,
                    rollbackFor = {
                            NullPointerException.class,NullPointerException.class
                    }
            )
            ```
  
        - 关于异常抛出的事务提交和回滚
  
          - 当有异常抛出时会先检查是否在rollbackFor中，如果没有再取检查是否是RuntimeException异常
        
      - 大型项目中事务的操作
      
        - 大型项目中不可能对每一个方法逐个添加@Transcational注解,大型项目中采用Aspectj框架，完成事务和代码完全隔离，在xml文件中同一配置
      
        - 使用步骤
      
          - 添加Aspectj依赖
      
            ```xml
            <!-- https://mvnrepository.com/artifact/org.springframework/spring-aspects -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
                <version>5.3.18</version>
            </dependency>
            ```
      
          - 声明事务管理器对象
      
          - 事务增强器：声明业务方法和事务属性
      
            ```xml
            <tx:advice id="myadvice" transaction-manager="transactionManager">
                <tx:attributes>
                    <tx:method name="findAllUsers" propagation="REQUIRED" isolation="DEFAULT" read-only="true"/>
                    <tx:method name="add*" propagation="REQUIRED" rollback-for="java.lang.NullPointerException,java.lang.NullPointerException" read-only="true"/>
                    <tx:method name="*" propagation="REQUIRED" rollback-for="java.lang.NullPointerException,java.lang.NullPointerException" read-only="true"/>
                </tx:attributes>
            </tx:advice>
            ```
      
            - tx:attributes下对于业务方法的事务配置其中tx:method中的name只需要提供方法名称而不需要类名且可以使用*通配符
      
            - 但是这也会带来麻烦，如果只是想要精确到某些包下的某些方法，可以配置aop切入点
      
              ```xml
              <aop:config>
                  <!--切入点表达式声明-->
                  <aop:pointcut id="pointcut" expression="execution(* *..services..*.*(..))"/>
                  <!--pointcut-ref指定需要增强的方法-->
                  <aop:advisor advice-ref="myadvice" pointcut-ref="pointcut"/>
              </aop:config>
              ```
      
              - 切入点表达式："*****..services..*"表示包含所有services的包
              - aop:advisor：advice-ref表示事务增强器的引用，pointcut-ref表示切入点的引用，
      
          - 对于AOP与事务的总结
      
            - AOP是对方法的增强，而事务无非是在一系列DDL语句前后添加提交或回滚，而这种增强是固定的，所以可以通过spring的AOP完成事务与源代码的分离
    
    ------------
    
    ## SM框架和web
    
    - ContextLoaderListener监听器
    
      - 作用：在Tomcat启动时创建Spring容器
    
        ```xml
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring配置文件路径</param-value>
        </context-param>
        
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        ```
    
        - ContextLoaderListenerd的默认Spring配置文件为WEB-INF下的applicationContext.xml
    
        - 但是可以自己指定spring配置文件通过context-param
    
        - 底层原理
    
          - ContextLoaderListener这个类继承javase中的监听器，在Tomcat启动时创建ApplicationContext的子类WebApplicationContext并将其放置（通过setAttribute）在全局作用域（ServletContext）中
    
        - 在servlet中通过ServletContext的getAttribute（WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE）获取到容器对象。
    
        - 或者使用框架中的工具类
    
          ```java
          ServletContext context=getServletContext();
          WebApplicationContext app= WebApplicationContextUtils.getRequiredWebApplicationContext(context);
          ```
    
          

