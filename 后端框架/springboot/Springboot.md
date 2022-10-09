# Springboot

- ## springboot入门

  - 作用

    - 简化其他框架的配置
      - javaConfig
        - 使用一个类（javaConfig）作为XML配置文件的替代
        - 使用的注解
          - @Configuration（指定某个类为javaConfig）
          - @Bean（声明对象注入到容器中）

  - 传统spring使用步骤

    - 构建空maven工程

      ```xml
      <dependencies>
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>5.3.18</version>
          </dependency>
      
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.12</version>
              <scope>test</scope>
          </dependency>
          <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
      <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>druid</artifactId>
          <version>1.2.9</version>
      </dependency>
      
      
      </dependencies>
      
      <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <configuration>
                  <source>1.8</source>
                  <target>1.8</target>
                  <encoding>UTF-8</encoding>
              </configuration>
          </plugin>
      </plugins>
      </build>
      ```
  
    - 编写用户类
  
      ```java
      public class User {
          private String name;
          private String password;
          private int age;
          }
      //忽略set和get方法以及有参和无参构造方法
      ```
  
    - 编写javaConfig
  
      ```java
      @Configuration
      public class javaconfig {
      
          @Bean(name="User")//表示该方法的返回值会自动注入到容器中
          User createUser()
          {
              return  new User("zhansan", "123", 20);
          }
      
      }
      ```
  
      - @Bean下的方法的返回值在容器中的id默认为方法名，也可以通过name指定也可以使用value，效果是一样的
  
      - 这里同样可以采用xml文件为User注入从而代替createUser方法，只需要在javaconfig上添加@ImportResource注解
  
        ```xml
        <bean id="myuser" class="com.springboot.User.User">
            <property name="name" value="zhangsan"/>
            <property name="password" value="123"/>
            <property name="age" value="10"/>
        </bean>
        ```
  
        - javaconfig

          ```java
          @ImportResource("classpath:beans.xml")
          public class javaconfig {
              
          }
          ```
  
        - 对于某些配置文件（如mybatis中url等配置通过properties文件配置）可以采用@PropertyResource配置
  
          - 编写student类
  
            ```java
            @Component
            public class student {
            
                @Value("${student.name}")
                private String name;
                @Value("${student.pwd}")
                private String pwd;
                @Value("${student.id}")
                private String id;
                @Value("${student.age}")
                private int age;
                //省略set和get以及无参构造和有参构造方法
            }
            ```
  
            - 属性文件注入

            ```properties
            student.name=zhangsan
            student.pwd=123
            student.age=10
            student.id=20046209
            ```
  
            - javaconfig
  
              ```java
              @PropertySource("classpath:student.properties")
              @ComponentScan("com.springboot")
              public class javaconfig {
                  
              }
              ```
  
    - 编写测试单元
  
      ```java
      @Test
          void test()
          {
              ApplicationContext apt=new AnnotationConfigApplicationContext(javaconfig.class);
               User u = (User) apt.getBean("User");
              System.out.println(u);
          }
      ```
  
  - Springboot入门
  
    - 创建maven工程
  
      ```xml
      <parent>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-parent</artifactId>
              <version>2.6.7</version>
          </parent>
      
      
          <dependencies>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-web</artifactId>
              </dependency>
          </dependencies>
      ```
  
    - 编写springboot主应用类
  
      ```java
      @RestController
      @SpringBootApplication
      public class MyApplication {
      
          @RequestMapping("/hello")
          String home() {
              return "Hello World!";
          }
      
          public static void main(String[] args) {
              ConfigurableApplicationContext context = SpringApplication.run(MyApplication.class, args);
          }
      
      }
      ```
  
      - @RestController是@Controller（表示该类是控制类）和@ResponseBody（告诉String返回值是字符串而不是uri）的结合
      - @SpringBootApplication：标识这个是springboot应用类
      - SpringApplication.run(MyApplication.class, args);启动springboot应用类
  
    - 运行springboot应用类
  
      - Tomcat自动配置
  
        - 可以在resource文件夹下新增一个application.properties(springboot会自动识别这个文件），在这个配置文件中配置Tomcat的各种信息比如端口号等
  
          ```properties
          server.port=8080
          ```
  
        - 关于可以配置的所有信息在https://docs.spring.io/spring-boot/docs/2.6.7/reference/htmlsingle/#appendix.application-properties中可以找到
  
      - 简化Tomcat项目部署
  
        - 导入springboot插件
  
          ```xml
          <build>
                  <plugins>
                      <plugin>
                          <groupId>org.springframework.boot</groupId>
                          <artifactId>spring-boot-maven-plugin</artifactId>
                      </plugin>
                  </plugins>
              </build>
          ```
  
        - 通过这个插件可以直接点击将项目打包成指定格式（jar或war）,然后在本地通过java -jar执行
      
    - 入门案例分析
    
      - 父项目依赖
    
        ```xml
        <parent>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>2.6.7</version>
            </parent>
        ```
    
        - 在父项目中声明了许多依赖，基本上开发常用的依赖都声明了，子项目继承
    
        - 在父项目中依赖的版本号都规定好了，所以无需关心版本号，如果想要修改项目使用的依赖的版本可以自行声明版本号(以mysql为例)
    
          ```xml
          <properties>
              <mysql.version>5.1.43</mysql.version>
          </properties>
          ```
    
      - springboot依赖
    
        - spring-boot-starter-*，在后续开发中常见到这种依赖包，对某些场景比如spring-boot-starter-web是专门用于开发web的，那么springboot就将该应用场景下的所有jar打包好，这也是springboot官方给出的依赖包
      
        - 除此以外我们也可以自己定义一个springboot的starter包，命名为thirdpartyproject-spring-boot-starter
      
        - *-spring-boot-starter是第三方给出的依赖包
      
        - springboot依赖配置了springmvc，Tomcat等环境
          - springmvc
            - DispatchServlet
            - 字符编码过滤器
            - 视图解析器
            
          - Tomcat
          
            - 如果不想使用Tomcat只需要添加
          
              ```xml
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-web</artifactId>
                  <exclusions>
                      <exclusion>
                          <groupId>org.springframework.boot</groupId>
                          <artifactId>spring-boot-starter-tomcat</artifactId>
                      </exclusion>
                  </exclusions>
              </dependency>
              ```
          
        - 包管理
          - 只要是在springboot规定下的springboot主程序所在的包及其下的所有子包都会被组件扫描器扫描
          - 如果不满意springboot提供的包扫描，可以在@SpringBootApplication中修改scanBasePackages属性
          - @SpringBootApplication=@SpringBootConfiguration+@EnableAutoConfiguration+ComponentScan
          
        - 配置管理
          - 对于Tomcat等组件都可以在application.properties中配置，而这些配置最后都会以类的形式保存
          - springboot所有自动配置的包都在spring-boot-autoconfigure这个包下，而在项目执行过程中只有引入了springboot-starter的会生效
    
      - @Configuration新特性
      
        - proxyBeanMethods
      
          - 默认为true
      
          - 表示这个配置类将会被springboot代理
      
          - full模式和lite模式
      
            - proxyBeanMethods为true时，每当调用方法获取对象时都会检查IOC容器中是否存在该对象，如果有直接取得这个对象，这种模式成为full模式，也间接形成了单例模式
            - lite模式，也称为轻量级模式，proxyBeanMethods为false，获取对象时不再从IOC容器中检查是否存在该对象
      
            - 对于需要单例的对象使用full模式而其他的使用lite模式以此较少springboot的消耗
      
      - @Import组件
      
        - 可以使用在任何组件类上，一般用在测试类上
        - Import注解中含有Class[]属性，用来代表导入的组件，导入的组件会自动调用构造方法创建，构造出的组件的名称默认为类的全类名
      
      - @Conditional
      
        - 满足Conditional条件时进行组件注入，而@Conditional有很多派生类，条件的判断也是根据派生类来实现的
        
        - 例：
        
          - 假设在User类中存在另一个类为major，如果没有major则不生成User
        
            ```java
            
                @Bean
                @ConditionalOnBean(name="major")
                public User createUser()
                {
                    User user = new User();
                    return user;
                }
            ```
        
            - @ConditionalOnBean就是仅当某个指定的成员存在时才会生成Bean，在ConditionalOnBean中有三个属性，一个是Class[] value，一个是String[] type,一个是String[] name，分别是通过类型、类型名称、成员名来判断是否存在成员
            - ConditionalOnBean可以标注在方法上也可以标注在类上，标注在类上代表如果不符合条件则类中的所有注入bean方法都不会执行
            - 与ConditionalOnBean效果相反的是@ConditionalOnMissingBean仅当没有该bean时才会执行方法
        
      - @ConfigurationProperties
      
        - 用于Properties文件绑定
      
        - 属性
      
          - prefix：前缀，比如在properties中的mysql.pwd中的mysql,在该注解中别名为value，springboot会根据前缀和类中的成员的名称进行注入
      
        - 注意：springboot注解只对IOC容器中的有效，又或者是只对组件有效，所以添加了ConfigurationProperties的类必须要有@Component或者是其他类型的组件注解，且properties文件默认为application.properties
      
        - 案例：
      
          - User类
      
            ```java
            @ConfigurationProperties("user")
            @Component("user")
            public class User {
                private String name;
                private String id;
                private int age;
            
                @Autowired
                private Major major;
                //省略set和get方法
            }
            ```
      
          - major类
      
            ```java
            @ConfigurationProperties("major")
            @Component("major")
            public class Major {
                private String name;
                private String id;
                //省略set和get方法
            }
            ```
      
          - springboot主执行类和Controller类
      
            ```java
            @RestController
            @SpringBootApplication
            public class MyApplication {
            
            
            
                @Autowired
                User user;
            
            
                public static void main(String[] args) {
                    ConfigurableApplicationContext context = SpringApplication.run(MyApplication.class, args);
                }
            
                @Bean
                @RequestMapping("/user")
                public User createUser()
                {
                    System.out.println(user);
                    return user;
                }
            }
            ```
      
          - 分析大致过程：@SpringBootApplication启动组件扫描器扫描到Major和User上的@Component，并将其注入到容器中，其中User的major通过类型从容器中获取并自动注入
      
      - @EnableConfigurationProperties注解
      
        - 在javaConfig上中配置，可以用于指定需要绑定Properties文件的类，可以在没有@Component的User的情况下使用
        - EnableConfigurationProperties中有一个class[] value属性，通过这个属性指定需要配置文件绑定的类
      
      - @SpringBootApplication
      
        - @SpringBootApplication=@SpringBootConfiguration+@EnableAutoConfiguration+@Componentscan
      
        - @SpringBootConfiguration
      
          - 作用
            - 表示该注解下的类是一个SpringBoot配置类
      
        - @EnableAutoConfiguration
      
          - 合成注解，合成部分有
      
            ```java
            @AutoConfigurationPackage
            @Import({AutoConfigurationImportSelector.class})
            ```
      
          - @AutoConfigurationPackage(自动配置包)
      
            - 原码
      
              ```java
              @Import({Registrar.class})
              ```
      
            - Registrar
      
              ```java
              public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
                  AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
              }
              ```
      
              - 方法
                - 参数：AnnotationMetadata，主要是关于SpringBootApplication注解下类的信息（包名等）
                - 方法内部：new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames()最后结果就是添加了SpringBootApplication注解的包名
                - 总结：registerBeanDefinitions方法将主应用类包下的所有包名返回
      
          - @Import({AutoConfigurationImportSelector.class})
      
            - AutoConfigurationImportSelector（自动配置导入选择器）
      
              - 方法（selectImports：选择导入）
      
                - ```java
                   AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
                  ```
      
                - 利用getAutoConfigurationEntry(annotationMetadata)方法（获取自动配置条目）给容器中批量导入组件
      
                - getAutoConfigurationEntry(annotationMetadata)
      
                  - 通过调用getCandidateConfigurations（获取候选配置）批量导入组件
      
                  - getCandidateConfigurations
      
                    ```java
                    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
                    ```
      
                    - 利用SpringFactoriesLoader（spring工厂加载器加载）调用loadFactoryNames方法加载，返回值是Map集合，而在loadFactoryNames中
      
                      ```java
                      (List)loadSpringFactories(classLoaderToUse)
                      ```
      
                    - loadSpringFactories就是这个方法中的主要方法
      
                      ```java
                      Enumeration urls = classLoader.getResources("META-INF/spring.factories");
                      ```
      
                    - 从META-INF/spring.factories文件来加载文件，在大部分jar包中这个文件夹中都没有什么东西，但是对于某些包来说有许多核心，比如spring-boot-autoconfigure这个包，这些包下包含了一个spring.factories文件
      
                    - spring-boot-autoconfigure
      
                      - 在这个包下的spring.factories中包含了许多需要加载的类的全名称，大部分是**AutoConfiguration
      
                    - 总结：**当springboot一启动就会加载容器中的所有配置类（大约127个），但是实际上IOC容器中并没有这么多的组件，这是因为springboot的按需开启功能，而按需开启的功能是由多个@ConditionalOnclass等按照条件加载的@Conditional注解实现的，通俗的将也就是当我们编程时导入需要的包时这些类才会被加载到容器中，如果用户自己配置了组件则大部分以用户的优先，而且大部分的组件开启后都会顺便开启properties文件绑定**
                    
                    - 可以在properties文件中添加debug=true,可以在springboot启动时查看到未启用的组件
  
  --------------------
  
  - ## 使用工具简化开发
  
    - lombok工具简化javabean开发
  
    - 引入依赖
  
      ```xml
      <dependency>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <optional>true</optional>
      </dependency>
      ```
  
    - 安装插件（在IDEA插件市场上）
  
    - 对于User类和Major类这种以后就不需要编写get和set以及构造方法了，
  
      - set和get以及tostring方法都采用@Data注解
  
      - 如果需要生成tostring方法只需要添加@ToString(Data注解中默认自带，如果使用data则不需要写)，
  
      - 如果需要有参构造可以使用@AllArgsConstructor
  
      - 需要无参构造可以添加@NoargsConstructor
  
      - 重写hashcode方法,@EqualsAndHashCode
  
      - @Slf4j,日志注解，可以不需要使用sout，使用log将打印信息输出到日志中（也就是springboot启动时的那些行）
  
        ```java
        log.info("使用日志")
        ```
  
    - dev工具
  
      - 引入依赖
  
        ```xml
        
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
                <optional>true</optional>
            </dependency>
        ```
  
      - 每当项目内容发生改变（比如web项目修改页面），只需要Ctrl+F9就可以生效，而无需重启项目
  
    - spring Initailizr
  
      - 在IDEA创建项目时使用maven上面一栏的spring Initailizr创建项目
      - 在spring initailizr可以选择下载包的url，可以选择阿里云的服务器下载，http://start/aliyun.com
  
  --------------------
  
  - ## Springboot核心
  
    - ### Yaml配置文件
  
      - 类似于XML的标记式配置文件
  
      - 采用key: value的格式
  
      - 使用空格来代表层级关系
  
      - 使用的空格数不重要只需要同层级的元素左对齐即可
  
      - “#”表示注释
  
      - 在yaml中字符串不需要添加引号，添加引号代表转义（单引号）/不转义（双引号），例如单引号会将\n作为字符串输出，双引号会将其作为换行输出，建议字符串都添加引号，不然可能会出现以下错误
  
        - 例：
  
          ```yaml
          password: 0127
          ```
    
        - 实际读取时读到的却是87，这是因为没有添加引号，yaml按照数字解析了，而yaml又支持8进制，而8进制的标志是开头是0后面的数字不超过8，这就导致springboot获取该值时按照8进制解析了。
    
      - 基本使用
    
        - 字面量
    
          ```yaml
          k: v
          #k+:+空格+v
          ```
    
        - 对象
    
          ```yaml
          k:
            k1: v1
            k2: v2
            k3: v3
            
          #或者是
          k: {k1: v1,k2: v2,k3: v3}
          #Map的写法和对象相同
          ```
    
        - 数组或者集合（array，list，queue）
    
          ```yaml
          k: [v1,v2,v3,v4]
          #或者是
          k:
          - v1
          - v2
          - v3
          ```
          
        - 变量引用
    
          ```yaml
          k: 123
          g: ${k}123
          #g=123123
          ```
  
      - Yaml在springboot中的使用
    
        - 创建application.yaml或者是application.yml
    
        - 使用@ConfigurationProperties注解绑定该配置文件，yml的使用和properties相同，而且如果项目中两个文件都有那么这两个文件都会生效
    
        - yml同样可以配置springboot中的内容，比如mysql的驱动等，配置时会发现有自动提示，如果想要自己的类也有自动提示可以添加以下依赖
    
          ```xml
          <dependency>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-configuration-processor</artifactId>
                      <optional>true</optional>
          </dependency>
          ```
    
        - 但是这个插件仅适用在开发过程中对于结果的jar并不需要所以我们添加一个插件
    
          ```xml
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
              <configuration>
                  <excludes>
                      <exclude>
                          <groupId>org.springframework.boot</groupId>
                          <artifactId>spring-boot-autoconfigure-processor</artifactId>
                      </exclude>
                  </excludes>
              </configuration>
          </plugin>
          ```
    
          - 这个插件使得在打包时忽略我们的自动配置包
          
        - 通过成员获取yml文件中的数据
        
          ```java
          @Autowired
          private Environment env;
              
              @RequestMapping(value="/a",method= RequestMethod.GET)
              public String findstudent()
              {
                  System.out.println(env.getProperty("student.name"));
                  return "asdasdasdsad";
              }
          ```
        
          - Environment env用于获取配置文件中的信息，@Autowired自动注入，getProperty方法获取信息
  
    - ### 使用Springboot开发Web
    
      - Springboot依旧使用SpringMVC完成Web功能
    
        - 静态资源访问
    
          - 在spring Initailizr创建的项目中的resource下存在一个static目录用于存放静态资源目录，除此以外 or `/public` or `/resources` or `/META-INF/resources`等目录都可存放静态资源
    
          - servlet请求优先于静态资源的访问，而在springboot配置文件中
    
            ```yml
            spring:
              mvc:
                static-path-pattern: /**
            ```
    
            - 所有请求都会先寻找是否存在servlet对应如果没有则寻找静态资源，而静态资源的访问路径默认是/**即所有路径，可以修改该路径
    
            - 同样静态资源存放文件夹名称也可自定义
    
              ```yaml
              spring:
                web:
                  resources:
                    static-locations: classpath:/view
              ```
    
        - 欢迎页面设置
    
          - 在静态文件夹下新建index.html
          - 能处理/index的Controller类
    
        - 有参构造所有参数都会从容器中寻找
    
    - REST风格开发
    
      - 概述：简化URL访问
    
      - 例：
    
        ```java
        http://localhost:8080/user/saveUser 保存数据(简化前)
        http://localhost:8080/user    保存数据(简化后)		post
        
        其他的还有
        http://localhost:8080/user/1  删除数据(简化后)		delete
        http://localhost:8080/user/1  查询数据(简化后)		get
        http://localhost:8080/user  修改数据(简化后)		put
        ```
    
        - 虽然以上删除，查询，修改的url都一样，但是可以通过不同的提交方法判别
    
        - REST风格代码案例
    
          ```java
          
          @RequestMapping(value="/a/{id}",method= RequestMethod.GET)
              public String findstudent(@PathVariable Integer id)
              {
                  System.out.println(id);
                  return "123";
              }
          ```
    
          - 通过RequestMethod限制提交方式
    
          - @PathVariable和{id}表达式从url中获取数据
    
          - RequestMapping可以通过其他注解代替
    
            ```java
            @GetMapping("/a/{id}")
            ```
    
    - SpringBoot整合Junit
    
      - 在类上添加@SpringBootTest
    
        ```java
        @SpringBootTest
        class DemoApplicationTests {
        
            @Test
            void contextLoads() {
            }
        
        }
        ```
    
        - DemoApplicationTests类必须在@SpringBootApplication标注的类同包下或子包下，否则需要指定SpringBootApplication标注的类的类名
    
        ```java
        @SpringBootTest(classes=DemoApplication.class)//指定spring容器地址
        ```
    
    - SpringBoot整合Mybatis
    
      - 使用步骤
    
        - 配置数据库配置信息
    
          ```yaml
          spring:
           datasource:
             password: 123
             driver-class-name: com.mysql.cj.jdbc.Driver
             username: root
             url: jdbc:mysql://localhost:3306/user?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
             type: com.alibaba.druid.pool.DruidDataSource
          ```
          
        - 编写用户类
        
          ```java
          @Data
          public class User {
              private String name;
              private String password;
              private int id;
          }
          ```
        
        - 编写dao层
        
          ```java
          
          public interface UserDao {
              public User getbyid(int id);
          }
          ```
          
        - 编写映射文件
        
          ```xml
          <mapper namespace="com.boot.mybatis_boot.domain.UserDao.UserDao">
              <select id="selectbyid" resultType="com.boot.mybatis_boot.domain.User.User">
                  select *
                  from user
                  where id = #{id}
            </select>
          </mapper>
          ```
        
        - 编写springboot主应用类
        
          ```java
          @SpringBootApplication
          @RestController
          @MapperScan(basePackages="com.boot.mybatis_boot.domain.UserDao")
          public class MybatisBootApplication {
          
              @Resource
              UserDao userdao;
          
              public static void main(String[] args) {
                  SpringApplication.run(MybatisBootApplication.class, args);
              }
          
          
              @RequestMapping("/a")
              public String find(Integer id)
              {
                  System.out.println(userdao.selectbyid(3));
                  return "123";
              }
          
          }
          ```
        
          - **@MapperScan用于指定mapper类所在的包，而mapper.xml的位置在yml中指定,这两个都一定要指定，不然直接寄**
        
            ```yaml
            mybatis-plus:
              configuration:
                log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
              global-config:
                db-config:
                  id-type: auto
              type-aliases-package: com.plus.mybatis_plus.pojo.Book
              mapper-locations: mp默认为resource下的mapper文件夹下的所有（classpath:mapper/**/*.xml）
            ```
          
            
          
          - 使用注意：使用时记得查看target包下是否存在mapper文件 （要添加配置信息）
          
            ```xml
            <resources>
                    <resource>
                        <directory>
                            src/main/java
                        </directory>
                        <includes>
                            <include>**/*.properties</include>
                            <include>**/*.xml</include>
                        </includes>
                        <filtering>false</filtering>
                    </resource>
            
                </resources>
            ```
          
          - 
    
  - ## 运维篇
  
    - 临时属性配置
  
      - 假设临时想要修改Tomcat访问端口号可以在启动时配置临时属性
  
      ```
      java -jar springboot.jar --server.port=80 --属性名=值
      ```
  
      - 属性配置优先级：临时属性优先级>配置文件，其他优先级可以在springboot官网上查看
  
      - 而临时属性配置会通过Springboot主程序配置中的main方法中的String[] args传入，当然我们在程序中也可以自行制定arg的值
  
        ```
        
        @SpringBootApplication
        public class Ssmp1Application {
        
            public static void main(String[] args) {
                String[] strings = new String[1];
                strings[0]="--server.port=80";
                SpringApplication.run(Ssmp1Application.class, strings);
            }
        
        }
        
        ```
  
        - 如果不需要该参数也可以去掉
  
          ```
           SpringApplication.run(Ssmp1Application.class);
          ```
  
    - 四层属性配置文件
  
      - 第一层：classpath:config/application.yml
      - 第二层：classpath:application.yml 
      - 第三层：jar包同层文件夹下/application.yml
      - 第四层：jar包同层文件夹下/config/application.yml
      - 层数越高级别越高，不同层级的配置文件中的不同配置会相互叠加
  
    - springboot默认配置文件名称为application，这个是可以修改的
  
      ```java
      public static void main(String[] args) {
              String[] strings = new String[1];
              strings[0]="--spring.config.name=你自己的属性配置文件名";
              SpringApplication.run(Ssmp1Application.class, strings);
          }
      ```
  
    - springboot配置文件位置也可以修改
  
      ```java
      strings[0]="--spring.config.location=classpath:配置文件位置(也可以去掉classpath：然后填写全局路径)";
      ```
  
      - 如果存在多个配置文件可以用“，”隔开，最后一个配置文件优先级最高
  
    - 模拟多环境开发
  
      - 在软件开发过程中程序可能需要在多个环境中运行，比如生产环境，开发环境，测试环境，每个环境的配置可能不同，在配置文件中可以实现多环境开发
  
        ```yaml
        
        spring:
          datasource:
           #password: mysql@wdnmd$pwd!-1026
           driver-class-name: com.mysql.cj.jdbc.Driver
           username: root
           #url: jdbc:mysql://localhost:3306/boot?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
           type: com.alibaba.druid.pool.DruidDataSource
          profiles:
            active: kf
        
        
        mybatis-plus:
          configuration:
            log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
          global-config:
            db-config:
              id-type: auto
          type-aliases-package: com.plus.mybatis_plus.pojo.Book
        
        ---
        #开发环境
        
        spring:
          datasource:
            password: 123
            url: jdbc:mysql://localhost:3306/user?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
          profiles: kf
        
        
        ---
        #测试环境
        spring:
          datasource:
            password: mysql@wdnmd$pwd!-1026
            url: jdbc:mysql://localhost:3306/boot?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
          profiles: test
        
        ```
  
        - 每个环境间通过---分割，启动指定环境
  
          ```yaml
          spring:
            profiles:
              active: kf #环境名称
          ```
  
        - 创建环境并命名
  
          ```yaml
          spring:
            profiles: kf
          ```
  
        - 但是实际开发中我们可以将不同环境的配置以一个单独的配置文件存放，比如测试环境的配置文件名称为appliction-test.yml
  
          - 例
  
            - 测试环境配置文件(application-test.yml)
  
            ```yaml
            
            
            #测试环境
            spring:
              datasource:
                password: mysql@wdnmd$pwd!-1026
                url: jdbc:mysql://localhost:3306/boot?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
            
            ```
  
            - 主配置文件application.yml
  
              ```yaml
              spring:
                datasource:
                 driver-class-name: com.mysql.cj.jdbc.Driver
                 username: root
                 type: com.alibaba.druid.pool.DruidDataSource
                profiles:
                  active: dev
              ```
  
          - 对于不同应用的配置文件可以作为一个单独的配置文件放置
  
            ```yaml
            #application-mp.yml
            mybatis-plus:
              configuration:
                log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
              global-config:
                db-config:
                  id-type: auto
              type-aliases-package: com.plus.mybatis_plus.pojo.Book
            
            ```
  
          - 主配置文件application.yml
  
            ```yaml
            spring:
              datasource:
               #password: mysql@wdnmd$pwd!-1026
               driver-class-name: com.mysql.cj.jdbc.Driver
               username: root
               #url: jdbc:mysql://localhost:3306/boot?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
               type: com.alibaba.druid.pool.DruidDataSource
              profiles:
                active: dev
                include: mp
            #添加一个include即可
            ```
  
          - 如果对于不同的active需要对应不同的include可以这样写
          
            ```yaml
            spring:
              datasource:
               #password: mysql@wdnmd$pwd!-1026
               driver-class-name: com.mysql.cj.jdbc.Driver
               username: root
               #url: jdbc:mysql://localhost:3306/boot?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
               type: com.alibaba.druid.pool.DruidDataSource
              profiles:
                active: dev
                group:
                  "dev": devmp,devMysql
                  "test": testmp,testMysql
            
            ```
          
            - 在Resource目录下存放着dev环境专用的mysql配置文件(application-devMysql.yml),而test同理，这里只需要使用group就可以动态加载这些配置文件
          
          - 在主要开发中还是通过Maven来指定开发环境（因为Maven为主，spring只是依赖Maven）
          
            ```xml
            <profiles>
                    <profile>
                        <id>env_dev</id>
                        <properties>
                            <!--profile.active中的值就是在配置文件中actvie需要设置的名称-->
                            <profile.active>dev</profile.active>
                        </properties>
                        <activation>
                            <!--默认采用这个环境-->
                            <activeByDefault>true</activeByDefault>
                        </activation>
                    </profile>
            
                    <profile>
                        <id>env_test</id>
                        <properties>
                            <!--profile.active中的值就是在配置文件中actvie需要设置的名称-->
                            <profile.active>test</profile.active>
                        </properties>
                    </profile>
                </profiles>
            ```
          
            - 主yml,profile.active的值就是maven中profile.active标签中的值
          
              ```yaml
              
              spring:
                profiles:
                  active: @profile.active@
                  group:
                    "dev": devmp,devMysql
                    "test": testmp,testMysql
              
              ```
          
    
    - Springboot日志功能
    
      - 使用日志功能
    
        - 添加项目依赖
    
          ```xml
          <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-logging -->
                  <dependency>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-starter-logging</artifactId>
                      <version>2.6.7</version>
                  </dependency>
          
          ```
    
        - 创建日志类
    
          ```java
          private static final Logger logger= LoggerFactory.getLogger(BookController.class);
          ```
    
          - BookController是该日志对象所在的类
          - 该步骤可以使用注解代替，只需要在需要使用日志的类上添加@Slf4j（酸辣粉4斤）,而默认的日志对象名称为log
    
        - 使用日志
    
          ```java
           //使用日志
                  logger.info("info..");
                  logger.error("error..");
                  logger.warn("warning..");
                 logger.debug("debug..");
                 logger.trace("堆栈...");
                 //logger.fatal("灾难性,已经和error合并");
          
          //使用日志打印信息
          2022-05-09 14:19:28.293  INFO 20500 --- [nio-8080-exec-1] c.s.BookWeb.Controller.BookController    : info..
          2022-05-09 14:19:28.293 ERROR 20500 --- [nio-8080-exec-1] c.s.BookWeb.Controller.BookController    : error..
          2022-05-09 14:19:28.293  WARN 20500 --- [nio-8080-exec-1] c.s.BookWeb.Controller.BookController    : warning..
          ```
    
        - 日志级别：debug<info<warn<error,默认系统启动为info级别所以debug看不到，可以在配置文件中修改
    
          ```yaml
          logging:
            level:
              root: info
              com.ssmp.BookWeb.Controller: debug
          ```
    
          - 其中root表示所有包下的信息，  com.ssmp.BookWeb.Controller: debug代表设置指定包下的日志级别
    
          - 也可以设置分组，对每个分组设置级别
    
            ```yaml
            logging:
              group:
                infobackage: com.ssmp.BookWeb.pojo,com.ssmp.BookWeb.mapper
                errorbackage: com.ssmp.BookWeb.Controller
              level:
                root: info
                infobackage: info
                errorbackage: error
            ```
    
        - 日志格式
    
          - 时间+级别+PID（进程ID）+所属线程名称+日志所属类（在哪创建的日志对象，对于boot内置的包会进行包名省略）+日志消息
    
            ```java
            2022-05-09 14:28:06.654  INFO 4500 --- [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
            ```
    
          - 设置日志输出格式
    
            ```java
            
            
            %d{HH: mm:ss.SSS}——日志输出时间。
            
            %thread——输出日志的进程名字，这在Web应用以及异步任务处理中很有用。
            
            %-5level——日志级别，并且每个级别名称占5个空格并且左对齐（正数5是右对齐）。也可以使用%p
            
            %logger{36}——日志输出者的名字。%c和这个一样
            
            %msg——日志消息，也可以简写为%m
            
            %n——平台的换行符。
            %clr(字段){颜色}--表示该字段用彩色显示，比如%clr(%msg){red}用红色表示msg
            ```
    
          - 设置日志文件
    
            ```yaml
            logging:
              file:
                name: logging.log
            ```
    
            - 默认存放位置在项目根目录下，如果需要当日志文件过大时新增日志文件记录
    
              ```yaml
              logging:
                logback:
                  rollingpolicy:
                    max-file-size: 2KB
                    file-name-pattern: logging.%d{yyyy-MM-dd}==%i.log
              
              ```
    
              - 超过2KB后会自动生成新的文件，命名为logging.年-月-日==第几个日志文件.log
    
                ![image-20220509150802061](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220509150802061.png)
    
            
    
            
    
    -------------------------
    
  - ## SpringBoot开发篇
  
    - ### 热部署
  
      - 手动热部署
  
      - 添加项目依赖(可以在spring Initailizr中添加)
  
        ```xml
        <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-devtools</artifactId>
                    <scope>runtime</scope>
                    <optional>true</optional>
                </dependency>
        ```
  
      - 使用Ctrl+F9，不会加载jar资源（不会修改jar包）
  
      - 自动热部署
  
        - 参考博客：https://blog.csdn.net/qq_19007335/article/details/124069635
  
      - 配置文件更改热部署参与的文件夹
  
        ```yaml
        spring:
          devtools:
            restart:
              additional-exclude: static/**
        ```
  
        - additional-exclude:表示该文件夹下的文件不参与热部署，devtools默认配置了一些无需热部署的文件夹
  
      - 设置热部署开关
  
        - 配置文件修改
  
          ```yaml
          spring:
            devtools:
              restart:
                enabled: true
          ```
  
        - 为了防止其他的配置文件覆盖我们的配置，可以在Main方法中
  
          ```java
          System.setProperty("spring.devtools.restart.enabled","true");
          ```
  
          - System的优先级高于配置文件
  
    - ### 第三方bean配置绑定
  
      - 时间变量
  
        ```java
        @DurationUnit(ChronoUnit.DAYS)
        private Duration time;
        ```
  
        - Duration是java8中自带的时间计量类，DurationUnit注解可以指定该类的单位
  
      - 存储容量变量
  
        ```java
        @DataSizeUnit(DataUnit.BYTES)
        private DataSize datasize;
        ```
  
        - DataSize是java8中自带的容量大小类，DataSizeUnit可以设置容量单位
        - 赋值也可以直接使用MB等单位如datasize=10MB
  
      - 数据校验
  
        - 数据校验包在javaEE中就有规范（只是一组接口），所以需要导入相关包（导入Springboot相关实现类）
  
          ```xml
          <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation -->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-validation</artifactId>
              <version>2.6.7</version>
          </dependency>
          ```
  
        - 在需要数据校验的类上添加@Validated
  
        - 在需要校验的数据成员上添加相关条件，比如@Max(value=8888,message="不能超过该最大值"),其他相关注解在javax.validation.constraints包中可以查询到，同时springboot也扩展了一些注解供使用
  
        - 以下是一些常用的注解
  
          | 注解      | 作用                                         |
          | --------- | -------------------------------------------- |
          | @NotNull  | 判断包装类是否为null                         |
          | @NotBlank | 判断字符串是否为null或者是空串(去掉首尾空格) |
          | @NotEmpty | 判断集合是否为空                             |
          | @Length   | 判断字符的长度(最大或者最小)                 |
          | @Min      | 判断数值最小值                               |
          | @Max      | 判断数值最大值                               |
          | @Email    | 判断邮箱是否合法                             |
  
    
    - ### 测试
    
      - 在springboot测试类中添加临时属性(优先级高于配置文件)
    
        ```java
        @SpringBootTest(properties={"test.properties=123"})
        class Ssmp1ApplicationTests {
        
            @Value("${test.properties}")
            String testpro;
        
            @Test
            void contextLoads() {
                System.out.println(testpro);
            }
        
        }
        ```
    
        - SpringBootTest中还有args属性，同样可以实现该功能，但是格式不一样
    
          ```java
          @SpringBootTest(properties={"--test.properties=123"})
          ```
    
        - 而这种优先级更高
    
      - 启动Web环境测试
    
        - 在@springtest有一个WebEnvironment属性，为枚举类型，其值有
    
          ```java
          public static enum WebEnvironment {
                  MOCK(false),
                  RANDOM_PORT(true),
                  DEFINED_PORT(true),
                  NONE(false);
              }
          ```
    
        - NONE:无Web环境
    
        - DEFINED_PORT：使用定义的端口启动Web环境（按配置文件中的端口来）
    
        - RANDOM_PORT：随机端口
    
      - 模拟发出请求
    
        ```java
        @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
        @AutoConfigureMockMvc//添加虚拟Mvc，使用该注解后会在容器中注入一个MockMvc,该对象可以发起请求
        class Ssmp1ApplicationTests {
        
        
            //@Resource
            //MockMvc mvc;
        
            @Test
            void contextLoads(@Autowired MockMvc mvc) throws Exception {
                MockHttpServletRequestBuilder requestBuilders =  MockMvcRequestBuilders.get("/b");
                ResultActions perform = mvc.perform(requestBuilders);
            }
        
        }
        ```
    
      - 设置预期值并比对
    
        - 比对预期响应状态
    
          ```java
          //执行请求
          ResultActions perform = mvc.perform(requestBuilders);
          //创建比较对象
          StatusResultMatchers status = MockMvcResultMatchers.status();
          //定义预期状态值
          ResultMatcher ok = status.isOk();
          //比对预期值和实际值
          perform.andExpect(ok);
          ```
    
        - 比对响应体
    
          ```java
          //执行请求
          ResultActions perform = mvc.perform(requestBuilders);
          //创建比较对象
          ContentResultMatchers content = MockMvcResultMatchers.content();
          //定义预期响应体值（字符串形式）
          ResultMatcher string = content.string("123");
          //比对预期值和实际值
          perform.andExpect(string);
          ```
    
        - 比较响应json
    
          ```java
          //执行请求
          ResultActions perform = mvc.perform(requestBuilders);
          //创建比较对象
          ContentResultMatchers content = MockMvcResultMatchers.content();
          //定义预期json值
          ResultMatcher json = content.json("json字符串");
          //比对预期值和实际值
          perform.andExpect(json);
          ```
    
        - 比较响应头
    
          ```java
          //执行请求
          ResultActions perform = mvc.perform(requestBuilders);
          
          //创建比较对象
          HeaderResultMatchers header = MockMvcResultMatchers.header();
          //定义预期响应头值
          ResultMatcher string = header.string("Content-Type", "application/json");
          //比对预期值和实际值
          perform.andExpect(string);
          ```
    
      - 测试DAO层
    
        - 测试中不对数据库进行修改，只需要使用事务回滚即可
    
          ```java
          @Transactional
          class Ssmp1ApplicationTests {}
          ```
    
          - 这样事务就会自动回滚
    
        - 如果想要提交数据可以添加@Rollback(false),Rollback中的默认值是true
    
          ```java
          @Transactional
          @Rollback(false)
          class Ssmp1ApplicationTests {}
          ```
    
        - 使用随机值测试
    
          - 在配置文件中可以使用random类所提供的的方法
    
          ```yaml
          testcase:
            book:
              id: ${random.int(10)}#限制生成数范围最大为10
              name: ${random.value}}
              id2: ${random.int[1,2]}}#限制生成数范围为1到2
          ```
    
          - testcase等都是自定义标签
      
    - ### 数据库有关
    
      - sringboot内置数据源
    
        - 在使用springboot整合Druid需要添加Druid-boot-starter包，只要导入了这个包，springboot默认的数据源管理就会转为Druid，即使在yaml文件中不填写Druid而直接编写数据库有关信息
        - 而springboot在未导入Druid时默认采用HikariCP数据源，如果在web环境下默认是Tomcat自带的DataSource，除此以外还有Commons DBCP数据源供使用
        - 在配置文件中可以对这些数据源配置（注意：hikari数据源的url无需配置，放置在其上层即可）
    
      - springboot内置数据库
    
        - H2,HSQL,Derby
    
      - springboot整合Redis
    
        - 导入start包和连接池包
    
          ```xml
           <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-redis -->
                  <dependency>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-starter-data-redis</artifactId>
                  </dependency>
          
                  <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-pool2 -->
                  <dependency>
                      <groupId>org.apache.commons</groupId>
                      <artifactId>commons-pool2</artifactId>
                  </dependency>
          ```
        
        - 配置主机地址和端口以及连接池的一些配置
        
          ```yaml
          spring:
            redis:
              host: 43.138.191.71
              port: 7379
              timeout: 5000ms # 连接超时时间（毫秒）
              password: 'q$q^qPmjp3'
              lettuce:
                pool:
                  max-active: 20 # 连接池最大连接数（使用负值表示没有限制）
                  max-idle: 10 # 连接池中的最大空闲连接
                  min-idle: 5 # 连接池中的最小空闲连接
                  max-wait: 5000ms # 连接池最大阻塞等待时间（使用负值表示没有限制
          
          ```
          
        - 创建RedisTemplate ，并通过RedisTemplate 执行
        
          ```java
          @SpringBootTest
          class BookApplicationTests {
          
              @Autowired
              private RedisTemplate redisTemplate;
          
              @Test
              void contextLoads() {
                  //获取操作对象，可以是Hash也可以是value（字符串）
                  ValueOperations ops = redisTemplate.opsForValue();
                  System.out.println(ops.get("tt"));
              }
          
          }
          
          ```
        
          - RedisTemplate操作之后存储的key和value是通过序列化的，RedisTemplate的原型是RedisTemplate<k,v>,如果不指定是Object，在存储时会经过序列化，所以存储了gg这个key但在Redis客户端中找不到，如果想要避免序列化问题可以采用StringRedisTemplate进行操作
        
            ```java
            @Autowired
                private StringRedisTemplate stringredisTemplate;
            
                @Test
                void contextLoads() {
                    ValueOperations<String, String> ops = stringredisTemplate.opsForValue();
                    System.out.println(ops.get("gg"));
                }
            ```
        
        - 在springboot内部有两种实现对Redis的操作，一种是jedis，一种是lettuce，而默认采用lettuce,如果需要指定操作方法可以使用配置文件指定，最终操作Redis的都是RedisTemplate，当然也可以使用原生的jedis
        
          ```yml
          
          spring:
            redis:
              host: 43.138.191.71
              port: 7379
              timeout: 5000ms # 连接超时时间（毫秒）
              password: 'q$q^qPmjp3'
              client-type: jedis
              lettuce:
                pool:
                  max-active: 20 # 连接池最大连接数（使用负值表示没有限制）
                  max-idle: 10 # 连接池中的最大空闲连接
                  min-idle: 5 # 连接池中的最小空闲连接
                  max-wait: 5000ms # 连接池最大阻塞等待时间（使用负值表示没有限制
              jedis:
                pool:
                  max-active: 20 # 连接池最大连接数（使用负值表示没有限制）
                  max-idle: 10 # 连接池中的最大空闲连接
                  min-idle: 5 # 连接池中的最小空闲连接
                  max-wait: 5000ms # 连接池最大阻塞等待时间（使用负值表示没有限制
                  
          ```
        
        - jedis连接Redis是直连模式，在多线程模式下使用jedis会存在线程安全问题
    
  - ## 整合第三方技术
  
    - ### 缓存
  
      - 使用Springboot缓存
  
        - 启动缓存
  
          - 导入依赖
  
          ```xml
          <dependency>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-starter-cache</artifactId>
                  </dependency>
          ```
  
          - 在boot主类上添加注解EnableCaching
  
            ```java
            @SpringBootApplication
            @EnableCaching
            public class BookApplication {
            
                public static void main(String[] args) {
                    SpringApplication.run(BookApplication.class, args);
                }
            
            }
            ```
  
          - 在需要使用缓存的方法上标注注解@Cacheable
  
            - Service接口
  
              ```java
              public Book getByIdincache(int id);
              ```
  
            - Service实现类
  
              ```java
              @Override
                  @Cacheable(value = "cachespace",key="#id")
                  public Book getByIdincache(int id) {
                      return mapper.selectById(id);
                  }
              ```
  
              - 之后该方法有关的数据就会存储在cachespace中并且标识为id
  
            - 测试
  
              ```java
              @RequestMapping("/getbyid")
                  Book getbyid(Integer id) throws NoSuchMethodException {
                      return service.getByIdincache(id);
                  }
              ```
  
              - 请求一次之后，再次请求便不会显示MP的查询信息这就代表没有使用数据库
  
        - 实现手机验证码发送
  
          - 发送
  
            - 编写生成随机验证码工具类
  
              ```java
              @Component
              public class CodeUtil {
              
                  public String getRandomCode(int length)
                  {
                      StringBuilder buffer = new StringBuilder();
                      Random random = new Random();
                      for (int i = 0; i < length; i++) {
                          buffer.append(String.valueOf(random.nextInt(10)));
                      }
                      return buffer.toString();
                  }
              
              }
              
              ```
  
            - 添加Service接口方法
  
              ```java
              public String getCode(String phone);
              ```
  
            - 实现方法并添加缓存注解
  
              ```java
              @Override
              @Cacheable(value = "phoneCode",key="#phone")
              public String getCode(String phone) {
                  return codeUtil.getRandomCode(6);
              }
              ```
  
              - 也可以采用以下方式编写
  
                ```java
                @Override
                    @CachePut(value = "phoneCode",key="#phone")
                    public String getCode(String phone) {
                        return codeUtil.getRandomCode(6);
                    }
                ```
  
              - CachePut和Cacheable的区别是CachePut一定会执行注解下的方法（如果key值存在则会覆盖），而Cacheable会先查询，如果key值存在则直接返回，如果不存在则添加key。
  
            - 编写测试（Web）
  
              ```java
              @RequestMapping("/sendCode")
              String sendCode(String phone)
              {
                  return service.getCode(phone);
              }
              ```
            
          - 检查验证码是否正确
  
            - 编写工具类
  
              ```java
              @Cacheable(value = "phoneCode",key="#phone")
                  public String check(String phone)
                  {
                      return null;
                  }
              
              ```
        
              - Cacheable采取的是如果没有该key则执行方法，如果有则返回缓存中的数据
        
            - 编写Service方法
        
              ```java
              @Override
              public boolean checkCode(String phone,String code) {
                  String right_code = codeUtil.check(phone);
                  if(right_code !=null)
                  {
                      return code.equals(right_code);
                  }
                  return false;
              }
              ```
        
            - 编写Controller
        
              ```java
              @RequestMapping("/check")
              String checkCode(String phone,String code)
              {
                  return String.valueOf(service.checkCode(phone, code));
              }
              ```
        
        - 指定缓存数据库为Redis
        
          - 添加start-Redis依赖
        
          - 配置文件配置缓存数据库为Redis
        
            ```yml
            
            spring:
              cache:
                type: redis
                redis:
                  cache-null-values: true
                  key-prefix: phone_
                  time-to-live: 10s
                  use-key-prefix: true
            ```
        
            - key-prefix不仅代表了Cacheable中的value也代表了key-prefix，如果为false则两者都不会添加
        
          - 编写Service方法
        
            ```java
            @Override
            @Cacheable(value = "phoneCode",key="#phone")
            public String getCode(String phone) {
                return codeUtil.getRandomCode(6);
            }
            ```
        
          - 生成的Redis的key(生存时间为10s)为
        
            ```shell
            "phone_phoneCode::17748650331"
            ```
        
        - 指定缓存数据库是EhCache
        
          - 导入EhCache maven
        
            ```xml
            dependency>
                        <groupId>net.sf.ehcache</groupId>
                        <artifactId>ehcache</artifactId>
                    </dependency>
            ```
        
          - 编写Ehcache配置文件
        
            ```xml
            <?xml version="1.0" encoding="UTF-8"?>
            <ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                     xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">
                <!-- 磁盘缓存位置 -->
                <diskStore path="java.io.tmpdir/ehcache"/>
            
                <!-- 默认缓存 -->
                <!--
                   name:缓存名称。和Cacheabled中的value相关
                   maxElementsInMemory：缓存最大个数。
                   eternal:对象是否永久有效，一但设置了，timeout将不起作用。
                   timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。
                                         仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
                   timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。
                                         仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
                   overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。
                   diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
                   maxElementsOnDisk：硬盘最大缓存个数。
                   diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
                   diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
                   memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。
                                                 默认策略是LRU（最近最少使用）。
                                                 你可以设置为FIFO（先进先出）
                                                 或是LFU（较少使用）。
                   clearOnFlush：内存数量最大时是否清除。     
                -->
                <defaultCache
            
                        maxElementsInMemory="10"
                        eternal="false"
                        timeToIdleSeconds="3600"
                        timeToLiveSeconds="3600"
                        overflowToDisk="true"
                        memoryStoreEvictionPolicy="LRU"/>
            
                <!-- deptrole缓存 -->
                <cache name="deptrole"
                       maxElementsInMemory="10"
                       eternal="false"
                       timeToIdleSeconds="86400"
                       timeToLiveSeconds="86400"
                       overflowToDisk="true"
                       memoryStoreEvictionPolicy="LRU"/>
            
            </ehcache>
            ```
        
            
        
        - 使用阿里巴巴的jetCache连接Redis
        
          - jetCache是对SpringCache的封装的框架
        
          - 使用jetCache
        
            - 导入依赖
        
              ```xml
              <!-- https://mvnrepository.com/artifact/com.alicp.jetcache/jetcache-starter-redis -->
                      <dependency>
                          <groupId>com.alicp.jetcache</groupId>
                          <artifactId>jetcache-starter-redis</artifactId>
                          <version>2.6.2</version>
                      </dependency>
              ```
        
              - 这里要使用2.6.2的版本如果非版本在后续编程中会报错
        
            - 配置jetCache
          
              ```yml
              jetcache:
                remote:
                  default:
                    type: redis
                    host: 43.138.191.71
                    port: 7379
                    password: 'q$q^qPmjp3'
                    poolConfig:
                      maxTotal: 50
              #配置多个环境
                  sms:
                    type: redis
                    host: 43.138.191.71
                    port: 7379
                    password: 'q$q^qPmjp3'
                    poolConfig:
                      maxTotal: 50
              ```
        
            - 开启缓存（一定要加不然启动就报错，不管用不用）
          
              ```java
              @SpringBootApplication
              @EnableCreateCacheAnnotation//开启缓存的注解
              public class BookApplication {
              
                  public static void main(String[] args) {
                      SpringApplication.run(BookApplication.class, args);
                  }
              
              }
              ```
        
            - 手动创建缓存空间
          
              ```java
              @CreateCache(area="sms",name="jetcache",expire = 1,timeUnit = TimeUnit.MINUTES)//expire（过期时间）默认单位为秒
                  private Cache<String,String> jetcache;
              ```
        
            - 编写Service方法
          
              ```java
              @Override
                  public String getCode(String phone) {
                      String code=codeUtil.getRandomCode(6);
                      jetcache.put(phone,code);
                      return code;
                  }
              
                  @Override
                  public boolean checkCode(String phone,String code) {
                      return code.equals(jetcache.get(phone));
                  }
              
              ```
        
          - 配置jetCache本地方案
        
            - 配置文件
          
              ```yml
              jetcache:
                local:
                  default:
                    type: linkedhashmap
                    keyConvertor: fastjson
                remote:
                  default:
                    type: redis
                    host: 43.138.191.71
                    port: 7379
                    password: 'q$q^qPmjp3'
                    poolConfig:
                      maxTotal: 50
              ```
          
              - keyConvertor
                - 在Cache中可以将key作为非String类型，但是在缓存中必须是字符串，所以必须指定对象转String的方案，这里使用的是阿里自带的fastjson（在jetCache包中自带）
              - type: linkedhashmap
                - jetcache支持两种本地缓存，一种是linkedhashmap，一种是caffeine
        
            - 创建缓存空间（和远程一样）
          
              ```java
              @CreateCache(name="bookCache",expire = 1,timeUnit = TimeUnit.MINUTES,cacheType= CacheType.LOCAL)
              private Cache<String,Book> bookCache;
              ```
        
              - cacheType指定缓存方案，CacheType中包含了LOCAL，REMOTE，BOTH，默认为远程（如果配置文件中不配置远程可以正常使用）
        
          - jetCache规范配置文件
          
            ```yml
            jetcache:
              statIntervalMinutes: 15 #设置统一间隔，每过一分钟在控制台上显示关于这一分钟内Cache的命中次数等信息
              areaInCacheName: false #在缓存中key值是否带有area前缀
              local:
                default:
                  type: linkedhashmap
                  keyConvertor: fastjson
                  limit: 100 #缓存数据量
              remote:
                default:
                  type: redis
                  host: 43.138.191.71
                  port: 7379
                  password: 'q$q^qPmjp3'
                  keyConvertor: fastjson
                  valueEncoder: java #存储的值在转换时采取的格式
                  valueDecoder: java #存储的值在转换时采取的格式
                  poolConfig:
                    maxTotal: 50
                    minIdle: 5
                    maxIdle: 20
            
            
            ```
          
          - 使用类似于SpringCache的操作(使用方法注解配置缓存)
          
            - 开启方法注解
          
              ```java
              @EnableMethodCache(basePackages = "com.boot2.BookWeb.service")
              ```
          
              - 添加在主应用类上，basePackages是使用了JetCache注解的类所在的包（该注解功能相当于注解扫描）
          
            - 编写Service类
          
              ```java
              @Override
              @Cached(name="BookCache",expire=1,timeUnit = TimeUnit.MINUTES,key = "#id")
              public Book getByIdincache(int id) {
                  return mapper.selectById(id);
              }
              
              
              @Override
              @CacheUpdate(name = "BookCache", key = "#book.id", value = "#book")
              public int updateOneBook(Book book) {
                  return mapper.updateById(book);
              }
              
              @Override
              @CacheInvalidate(name="BookCache",key="#id")
              public int deleteOneBookByid(int id) {
                  return mapper.deleteById(id);
              }
              ```
          
              - Cached注解
                - 类似于Cacheable
              - CacheUpdate注解
                - BookCache用于指定是在哪个cache中，key代表是修改哪个记录，value代表修改后的内容（替代的内容）
              - CacheInvalidate注解
                - 删除缓存中的某个键值对
          
        - j2cache缓存整合框架
        
          - 使用
        
            - 导入maven
        
              ```xml
              <!-- https://mvnrepository.com/artifact/net.oschina.j2cache/j2cache-core -->
                      <dependency>
                          <groupId>net.oschina.j2cache</groupId>
                          <artifactId>j2cache-core</artifactId>
                          <version>2.8.4-release</version>
                      </dependency>
              
                      <!-- https://mvnrepository.com/artifact/net.oschina.j2cache/j2cache-spring-boot2-starter -->
                      <dependency>
                          <groupId>net.oschina.j2cache</groupId>
                          <artifactId>j2cache-spring-boot2-starter</artifactId>
                          <version>2.8.0-release</version>
                      </dependency>
              ```
        
            - 配置yml文件
        
              ```yml
              j2cache:
                config-location: j2cache.properties
              ```
        
            - 编写j2cache.properties文件(也可以直接在j2cache中的Core核心包下就可以找到),下面只展示基础配置
        
              ```properties
              
              #配置一级缓存的供应商
              j2cache.L1.provider_class = ehcache
              #配置ehcache的配置文件名
              ehcache.configXml=ehcache.xml
              
              
              #配置二级缓存的供应商为Redis，这里不能直接填写Redis，而是需要填写j2cache在springboot中关于Redis的提供类，具体的类可以在依赖包（与springboot整合的包下）中找到
              j2cache.L2.provider_class = net.oschina.j2cache.cache.support.redis.SpringRedisProvider
              #二级缓存有关配置命名，命名后可以通过名称配置属性（类似于分组）
              j2cache.L2.config_section=MyRedis
              MyRedis.hosts=43.138.191.71:7379
              MyRedis.password=q$q^qPmjp3
              #添加key前缀
              MyRedis.namespace=j2cache
              #MyRedis.mode=single 这里可以配置Redis的模式如主从模式，哨兵模式等，默认为single也就是单机模式
              #一级缓存和二级缓存之间的交互
              
              #两个数据库之间的交互是通过发布和订阅实现的，下面的配置也就是配置Redis广播类，该类可以在j2cache中的springboot-start包下找到
              j2cache.broadcast=net.oschina.j2cache.cache.support.redis.SpringRedisPubSubPolicy
              #是否启用二级缓存
              j2cache.l2-cahce-open=true
              ```
        
              
        
              ![image-20220512202704199](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220512202704199.png)
        
            - 具体使用
        
              - 定义频道
        
                ```java
                @Autowired
                private CacheChannel channel;
                ```
        
              - 使用频道
        
                ```java
                @Override
                public String getCode(String phone) {
                    String code=codeUtil.getRandomCode(6);
                    channel.set("ehcache",phone,code);
                    return code;
                }
                
                @Override
                public boolean checkCode(String phone,String code) {
                    CacheObject ehcache = channel.get("ehcache", phone);
                    return code.equals(ehcache.asString());
                }
                ```
        
                - 其中返回值CacheObject其实就只是个结果封装，其中包含了该键值对的key，value，以及来自哪个缓存（在ehcache中自定义的缓存），而该类其中有一个AsString方法可以直接 返回vlaue- 
    
    -  ### 定时任务
    
      - 开发中常常遇到需要定时执行某个任务的场景，最简单的想法就是新建一个线程通过sleep完成这项任务，而市面上也有关于这种技术的整合和扩展，也就是Quartz,而springboot在整合了Quartz后也推出了自己的产品Spring Task
    
      - 相关概念
    
        - 工作job：具体要执行的任务
        - 工作明细jobDetail：执行任务的细节
        - 触发器Trigger：触发工作的规则
        - 调度器Scheduler：工作明细和触发器之间的对应关系
    
      - 整合Quartz
    
        - 导入坐标
    
          ```xml
          <dependency>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-starter-quartz</artifactId>
                  </dependency>
          ```
    
        - 编写自定义Quartz类
    
          ```java
          public class MyQuartz extends QuartzJobBean {
              @Override
              protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
                  System.out.println("定时任务执行中。。。。。");
              }
          }
          ```
    
          - 必须要继承QuartzJobBean类（抽象类）并实现其中的方法，该方法中的内容其实就是需要执行的内容
          
        - 编写QuartzJobConfig（其中包含Trigger和Scheduler）
        
          ```java
          @Configuration
          public class QuartzConfig {
          
              @Bean
              public JobDetail printJobDetail() {
                  return JobBuilder.newJob(MyQuartz.class).storeDurably().build();
              }
          
              @Bean
              public Trigger printJobTrigger() {
                  ScheduleBuilder scaedBuilder=CronScheduleBuilder.cronSchedule("0/5 0-30 * * * ? ");
                  return TriggerBuilder.newTrigger().forJob(printJobDetail()).withSchedule(scaedBuilder).build();
              }
          }
          
          ```
        
          - cronSchedule中时linux Cron表达式，指明执行周期，可以在网页上生成
          - storeDurably表示如果不执行则序列化该MyQuartz类
        
      - 以上整合Quartz过于复杂，下面采用Spring task方法创建定时任务
      
        - 开启定时任务注解(在主boot类上添加)
      
          ```java
          @EnableScheduling
          ```
      
        - 编写定时类
      
          ```java
          
          @Component
          public class MyTask {
              
              
              @Scheduled(cron="0/5 0-30 * * * ? ")
              void print()
              {
                  System.out.println("我是spring task");
              }
          }
          
          ```
      
          - Scheduled注解标明需要定时指定的方法并在其中指定Cron表达式
      
        - 除此以外可以配置关于task的相关信息
      
          ```yml
          #配置项大概如下
          spring:
            task:
              execution:
                pool:
                  allow-core-thread-timeout: #是否允许核心线程超时，开启后线程池能够动态扩展收缩
                  core-size:               #核心线程数
                  keep-alive:         #线程在终止前容许空闲的时间
                  max-size:                #容许的最大线程数，如果正在填充线程队列则扩展到该大小适应负载
                  queue-capacity:             #队列容量。无限容量不会增加池，因此会忽略“max size”属性。
                shutdown:
                  await-termination:          #执行器是否应等待计划任务在关机时完成
                  await-termination-period:   #执行器等待剩余任务完成的最长时间
                thread-name-prefix:          #用于新创建线程名称的前缀
          ```
    
    - ### 使用springboot整合javamail
    
      - 导入maven坐标
    
        ```xml
        <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-mail</artifactId>
                </dependency>
        ```
    
      - 编写配置文件
    
        ```yml
        #debug: true
        spring:
          #  邮箱发送配置
          mail:
            #    host不配置会注入失败
            host: smtp.163.com
            username: wdnmdwgzdl666@163.com
            password: EFAEOYTWMSJYSZQF
            default-encoding: utf-8
            protocol: smtp
            properties:
              mail:
                smtp:
                  connectiontimeout: 5000
                  timeout: 3000
                  writetimeout: 5000
        
        
        mymail:
          From: wdnmdwgzdl666@163.com
          Subject: 验证码
        
        ```
    
        
    
      - 创建Service类
    
        ```java
        @Service
        public class SendServiceImpl implements SendService {
        
        
            @Autowired
            CodeUtil codeUtil;
        
            @Value("${mymail.From}")
            String From;
        
            @Value("${mymail.Subject}")
            String Subject;
        
            @Autowired
            JavaMailSender javaMailSender;
        
            @Autowired
            private CacheChannel channel;
        
            @Override
            public boolean checkCode(String mail, String code) {
               return code.equals(channel.get("mailCode",mail).asString());
            }
        
            @Override
            public void sendMailCode(String userMail) {
                String code = codeUtil.getRandomCode(6);
                channel.set("mailCode",userMail,code);
                SimpleMailMessage msg = new SimpleMailMessage();
                msg.setFrom(From);
                msg.setTo(userMail);
                msg.setSubject(Subject);
                msg.setText("你的验证码是："+code);
                javaMailSender.send(msg);
            }
        }
        
        ```

-------------------------



- ## Springboot原理篇

  - ### 自动配置

    - #### bean加载方式（Spring有关）

      - bean的声明方式：XML文件方式（一）

          - 使用xml文件方式定义bean

            ```xml
            <bean id="god" class="pojo.god"/>
                <bean class="pojo.god"/>
                <bean class="pojo.god"/>
                <bean class="pojo.god"/>
            ```

          - 打印所有bean名称

            ```java
            public class springMain {
                public static void main(String[] args) {
                    ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
                    for (String s : ctx.getBeanDefinitionNames()) {
                        System.out.println(s);
                    }
                }
            }
            ```

          - 最终结果

            ```java
            god
            pojo.god#0
            pojo.god#1
            pojo.god#2
            ```

            - 也就是如果bean标签中定义了id则以id为名称，如果没有则默认是全路径类名#编号（第三方bean也是如此）
          
      - bean的声明方式：XML方式+注解方式（二）
      
          - 在需要注入容器的类上添加注解
      
            ```java
            @Component("god")
            public class god {
            }
            ```
      
          - 除此之外还需要添加注解扫描器告诉spring哪些包下存在bean(使用 context命名空间需要在xml文件上创建命名空间)
      
            ```xml
            <context:component-scan base-package="pojo"/>
            ```
      
          - 以上是通过注解配置自定义类的bean而第三方的bean是无法在源代码上添加注解的，所以采用以下方式
      
            ```java
            @Component
            public class DruidConfig {
                @Bean
                DruidDataSource dataSource()
                {
                    return new DruidDataSource();
                }
            }
            ```
      
          - 再次打印所有bean名称
      
            ```java
            druidConfig
            god
            org.springframework.context.annotation.internalConfigurationAnnotationProcessor
            org.springframework.context.annotation.internalAutowiredAnnotationProcessor
            org.springframework.context.annotation.internalCommonAnnotationProcessor
            org.springframework.context.event.internalEventListenerProcessor
            org.springframework.context.event.internalEventListenerFactory
            dataSource
            ```
      
          - 其中dataSource则是DruidDataSource的bean，它以方法名作为名称注入到容器中
      
      - bean的声明方式：纯注解方式（三）
      
          - 添加spring配置类
      
            ```java
            @ComponentScan("pojo")
            public class SpringConfig {
            }
            ```
      
            - ComponentScan本身除了作为组件扫描器之外，同时也会作为bean加载到spring容器中
      
          - 编写springmain类
      
            ```java
            public class springMain {
                public static void main(String[] args) {
                    ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
                    for (String s : ctx.getBeanDefinitionNames()) {
            
                        System.out.println(s);
                    }
                }
            }
            ```
      
      - bean的声明方式：使用Import注解创建Bean（四）
      
          - 在配置类上添加@Import导入bean
      
          ```java
          @ComponentScan("pojo")
          //@ImportResource("beans.xml")
          @Import(god.class)
          public class SpringConfig {
          }
          ```
      
          - 使用该方式创建出的Bean采用的名称为类的全路径名，如果god上有Component注解的话则不会创建
      
          - 这种方式比使用@Bean注解引入外部Bean更加方便
      
      - 工厂Bean
      
          - 编写一个工厂类(必须实现一个接口FactoryBean<K>,其中的泛型表示造出来的对象的类型)
      
            ```java
            public class MyFactoryBean implements FactoryBean<god> {
            
            
                @Override
                //你造出来的对象是什么对象
                public god getObject() throws Exception {
                    return new god();
                }
            
                @Override
                //你造出来的对象是什么类型的对象
                public Class<?> getObjectType() {
                    return god.class;
                }
            
                @Override
                //你造出来的对象是单例的吗
                public boolean isSingleton() {
                    return false;
                }
            }
            ```
      
          - 通过@Bean创建该工厂
      
            ```java
            @Component
            public class DruidConfig {
                @Bean
                MyFactoryBean dataSource()
                {
                    return new MyFactoryBean();
                }
            }
            ```
      
          - 打印该bean
      
            ```java
            pojo.god@5123a213
            ```
      
            - 会发现这个bean是god对象而非MyFactoryBean对象，这就是FactoryBean<god>接口的作用，这样其实就是把bean创建给细分了，降低耦合度，可以对创造的对象更加灵活（如单例或多例等）
      
      - 使用注解导入XML文件
      
          - 假设原有项目采用XML文件开发而新项目需要使用注解开发，可以通过@ImportResource注解导入XML文件
      
            ```java
            @ComponentScan("pojo")
            @ImportResource("beans.xml")
            public class SpringConfig {
            }
            
            ```
      
          - 而当XML文件中的Bean和注解方式的Bean重合时优先采用XML文件的Bean
          
      - bean的声明方式：使用AnnotationConfigApplicationContext创建Bean（五）
      
          ```java
          public class springMain {
              public static void main(String[] args) {
                  AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
                  ctx.registerBean(god.class,"1");
                  ctx.registerBean(god.class,"10086");
                  for (String s : ctx.getBeanDefinitionNames()) {
          
                      System.out.println(s+":"+ctx.getBean(s));
                  }
                  System.out.println(ctx.getBean("pojo.god"));
              }
          }
          
          ```
      
          - 最后god的id为10086，也就是后创建的bean会覆盖之前的bean，也可以使用register直接注册,只需要提供class即可，最后bean的名称为首字符小写
      
      - bean的声明方式：使用RegisterBean注册Bean（六）
      
          ```java
           ctx.registerBean(God.class,"123");
          ```
          
      - bean的声明方式：使用ImportSelector继承类批量创建Bean（七）
      
          - 编写MyImportSelector类
      
            ```java
            public class MyImportSelector implements ImportSelector {
            
                @Override
                public String[] selectImports(AnnotationMetadata annotationMetadata) {
                    return new String[]{"pojo.God"};
                }
            }
            
            ```
      
            - String数组中放置bean的全类名
      
          - 在spring配置类上添加Import注解导入MyImportSelector类,在springmain类中测试
      
            ```java
            @ComponentScan({"pojo","MyImportSelector"})
            @Import(MyImportSelector.class)
            public class SpringConfig {
            
            
            }
            ```
      
          - 在MyImportSelector的selectImports方法中AnnotationMetadata annotationMetadata参数可以获得关于SpringConfig类的信息，比如SpringConfig类上的注解信息，根据注解信息可以动态加载Bean
      
    - bean的声明方式：使用ImportSelector继承类批量创建Bean（八）
    
      - 创建一个类继承ImportBeanDefinitionRegistrar类并实现其中一个方法（两个方法差不多，一个参数中有Bean名称创造器，一个没有）
    
        ```java
        public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
            @Override
            public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
                //获取BeanDefinition（通过BeanDefinitionBuilder有很多方式获取）
                BeanDefinition  beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(God.class).getBeanDefinition();
                //BeanDefinition可以设置bean的属性
                beanDefinition.setLazyInit(false);
                //注册Bean
                registry.registerBeanDefinition("GodBean",beanDefinition);
            }
        }
        ```
    
        - 相比于selectImports，这种方法可操作空间更大，而且同样拥有AnnotationMetadata可以动态加载Bean
    
    - Bean的声明方式：使用BeanDefinitionRegistryPostProcessor继承类完成后置Bean创建（九）
    
      ```java
      public class MyBeanDefinitionRegistrarPostProcessor implements BeanDefinitionRegistryPostProcessor{
          @Override
          public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry beanDefinitionRegistry) throws BeansException {
              //获取BeanDefinition（通过BeanDefinitionBuilder有很多方式获取）
              BeanDefinition beanDefinition = BeanDefinitionBuilder.rootBeanDefinition(God.class).getBeanDefinition();
              //BeanDefinition可以设置bean的属性
              beanDefinition.setLazyInit(false);
              //注册Bean
              beanDefinitionRegistry.registerBeanDefinition("GodBean2",beanDefinition);
          }
      
          @Override
          public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
      
          }
      }
      ```
    
      - 两个需要实现的方法，其中postProcessBeanDefinitionRegistry方法的使用和ImportBeanDefinitionRegistrar（bean的加载方式八）相同但是通过后置处理器创建的Bean会覆盖之前创建的相同id的Bean，但是相比于registerBean，registerBean会覆盖后置处理器创建的Bean

