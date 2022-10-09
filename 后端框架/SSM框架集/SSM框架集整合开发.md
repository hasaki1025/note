# SSM框架集整合开发

- ##容器

  - SpringMVC容器
    - 管理控制器对象
    - SpringMVC容器属于Spring容器的子容器
  - Spring容器
    - 管理Service，DAO，工具类对象

![image-20220418173535984](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220418173535984.png)

- 实现步骤

  - 新建MAVEN项目添加项目依赖

    - 主要依赖

    - springmvc,spring,mybatis框架

    - jackson,mysql驱动，druid连接池，jsp，servlet

      ```xml
       <dependencies>
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
          <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
          <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.8</version>
          </dependency>
      
          <!-- https://mvnrepository.com/artifact/commons-logging/commons-logging -->
          <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
          </dependency>
      
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.18</version>
          </dependency>
      
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.18</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-beans -->
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.3.18</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-expression -->
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>5.3.18</version>
          </dependency>
      
      
          <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
          </dependency>
      
          <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.13.1</version>
          </dependency>
      
      
      
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc
      Springmvc的依赖-->
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.18</version>
          </dependency>
      
      
          <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api
        servlet包
        -->
          <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
          </dependency>
      
      
          <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
          <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.6</version>
          </dependency>
      
          <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
          <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.25</version>
          </dependency>
      
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.3.18</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-tx -->
          <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.3.18</version>
          </dependency>
          <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.6</version>
          </dependency>
        </dependencies>
      ```
  
    - 除此以外还需要添加resource标签帮助mybatis识别配置文件
  
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
  
  - 编写web.xml
  
    - 注册DispatcherServlet:创建springmvc容器对象
  
      ```xml
      <servlet>
          <servlet-name>spring_mvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:conf/dispatcherServlet.xml</param-value>
          </init-param>
          <load-on-startup>0</load-on-startup>
      </servlet>
      <servlet-mapping>
          <servlet-name>spring_mvc</servlet-name>
          <url-pattern>*.action</url-pattern>
      </servlet-mapping>
      ```
  
    - 注册spring监听器ContextLoaderListener：创建spring容器对象
  
      ```xml
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>classpath:conf/applicationContext.xml</param-value>
      </context-param>
       <listener>
              <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
          </listener>
      ```
  
    - 创建过滤器：解决乱码问题
  
      ```xml
      <filter>
          <filter-name>encoding</filter-name>
          <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
          <init-param>
              <param-name>encoding</param-name>
              <param-value>UTF-8</param-value>
          </init-param>
          <init-param>
              <param-name>forceRequestEncoding</param-name>
              <param-value>true</param-value>
          </init-param>
          <init-param>
              <param-name>forceResponseEncoding</param-name>
              <param-value>true</param-value>
          </init-param>
      </filter>
      
      <filter-mapping>
          <filter-name>encoding</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>
      ```
  
  - 编写框架配置文件
  
    - springmvc配置文件
  
      ```xml
          <!--注解扫描器-->
      <context:component-scan base-package="com.SSM01.controller"/>
      <!--设置视图解析器-->
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
          <property name="prefix" value="/WEB-INF/view/"/>
          <property name="suffix" value=".jsp"/>
      </bean>
      <!--设置注解驱动，方便使用ajax，处理静态资源-->
      <mvc:annotation-driven/>
      ```
  
    - spring配置文件
  
      ```xml
      <context:property-placeholder location="classpath:conf/jdbc.properties"/>
          <!--配置数据源-->
          <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
              <property name="url" value="${jdbc.url}" />
              <property name="username" value="${jdbc.username}" />
              <property name="password" value="${jdbc.password}" />
              <property name="maxActive" value="20" />
          </bean>
          <!--配置sqlsessionfactory-->
          <bean id="sqlsessionfactory" class="org.mybatis.spring.SqlSessionFactoryBean">
              <property name="dataSource" ref="dataSource"/>
              <property name="configLocation" value="classpath:conf/mybatis.xml"/>
          </bean>
          <!--创建所有DAO代理-->
          <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
              <property name="sqlSessionFactoryBeanName" value="sqlsessionfactory"/>
              <property name="basePackage" value="com.SSM01.dao"/>
          </bean>
          <!--组件扫描器，通过注解生成services类-->
          <context:component-scan base-package="com.SSM01.services"/>
      
          <!--事务处理可以采用注解也可以采用Aspectj方式-->
      ```
  
    - mybatis配置文件
  
      ```xml
      <!-- <settings>
           <setting name="logImpl" value="STDOUT_LOGGING"/>
       </settings>-->
       <typeAliases>
           <package name="com.SSM01.domain"/>
       </typeAliases>
       <mappers>
           <package name="com.SSM01.dao"/>
       </mappers>
      ```
  
    - 数据库属性配置文件
  
      ```properties
      jdbc.url=jdbc:mysql://localhost:3306/user
      jdbc.username=root
      jdbc.password=123
      ```
  
  - 编写DAO接口，mapping文件，service和实现类，controller，实体类
  
    - service和实现类
  
      ```java
      public interface Userservices {
          List<User> findAllUsers();
          int addUser(User user);
      }
      
      @Service
      public class UserservicesImpl implements Userservices {
      
          @Autowired
          Userdao dao;
          @Override
          public List<User> findAllUsers() {
              return dao.selectAll();
          }
      
          @Override
          public int addUser(User user) {
              return dao.insert(user);
          }
      }
      
      ```
  
    - controller
  
      ```java
      @Controller
      public class UserController {
      
          @Resource
          private Userservices us;
      
          @Transactional
          @RequestMapping(value = "/add.action")
          public ModelAndView addUser(User user)
          {
              int result =us.addUser(user);
              ModelAndView view = new ModelAndView();
              if(result>0)
              {
                  view.setViewName("addUser_safe");
              }
              else
              {
                  view.setViewName("addUser_defeat");
              }
              return view;
          }
      
      
      }
      ```
  
      

