#SM框架整合（mybatis、spring）

- 整合原理
  - Mybatis将对象交给spring的IOC创建，对于开发人员就可以直接通过spring创建Mybatis
  
- mybatis使用步骤
  - 定义DAO接口
  - 定义mapper文件
  - 定义主配置文件
  - 创建代理对象

- spring需要做的

  - 创建连接池对象：druid连接池
  - 创建SqlsessionFactory对象
  - 创建DAO对象

- 步骤

  - 添加Maven依赖

    - mybatis

    - mysql驱动

    - spring：core,context,expression,beans四个核心包，在这里只使用了context（负责IOC）以及tx和jdbc包（负责事务）

    - 日志：这里不需要

    - spring和mybatis的集合包：mybatis-spring

    - druid连接池

      ```xml
      <dependencies>
      
          <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis-spring</artifactId>
              <version>2.0.6</version>
          </dependency>
      
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.11</version>
              <scope>test</scope>
          </dependency>
      
      
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>5.3.18</version>
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
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-tx -->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-tx</artifactId>
              <version>5.3.18</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
          <dependency>
              <groupId>com.alibaba</groupId>
              <artifactId>druid</artifactId>
              <version>1.2.8</version>
          </dependency>
      
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-jdbc</artifactId>
              <version>5.3.18</version>
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
      </build>
      ```

  - 创建dao和mapper

    - dao

      ```java
      public interface UserDao {
          int insert(User user);
          List<User> selectAll();
      }
      ```
  
    - mapper

      ```xml
      <mapper namespace="org.SM.DAO.UserDao">
          <select id="selectAll" resultType="org.SM.User.User">
              select *
              from user
        </select>
          <insert id="insert" parameterType="org.SM.User.User">
              insert into user(id,name,password) values(#{id},#{name},#{password})
          </insert>
      </mapper>
      ```
  
  - 创建mybatis主配置文件
  
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <settings>
            <setting name="logImpl" value="STDOUT_LOGGING"/>
        </settings>
        <typeAliases>
            <package name="org.SM.User"/>
        </typeAliases>
        <mappers>
            <package name="org.SM.DAO"/>
        </mappers>
    </configuration>
    ```
  
  - 创建Service接口和实现类
  
    - Service接口
  
      ```java
      public interface UserServices {
          int addUser(User user);
          List<User> findAllUsers();
      }
      ```
  
    - 实现类
  
      ```java
      public class UserServicesImpl implements UserServices {
          UserDao userdao;
      
          public UserDao getUserdao() {
              return userdao;
          }
      
          public void setUserdao(UserDao userdao) {
              this.userdao = userdao;
          }
      
          @Override
          public int addUser(User user) {
              return userdao.insert(user);
          }
      
          @Override
          public List<User> findAllUsers() {
              return userdao.selectAll();
          }
      }
      ```
  
  - 创建spring配置文件：声明mybatis对象交给spring创建创建
  
    - 数据源
  
      - druid连接池通过spring中的bean配置
  
        ```xml
         <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 
             <property name="url" value="${jdbc_url}" />
             <property name="username" value="${jdbc_user}" />
             <property name="password" value="${jdbc_password}" />
             <property name="maxActive" value="20" />
        
         </bean>
        ```
  
        - url等信息同样可以通过配置文件指定
  
          ```xml
          <context:property-placeholder location="jdbc.properties"/>
          ```
  
    - SqlsessionFactory
  
      - 声明一个bean可以创建SqlsessionFactory，在mybatis和spring的整合包中含有一个SqlsessionFactoryBean的类可以完成这个功能
  
        ```xml
        <bean id="SqlsessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
            <!-- 配置数据源，ref指的是数据源bean的id-->
            
            <property name="configLocation" value="classpath:mybatis_config.xml"/>
             <!-- 配置mybatis主配置文件，对于spring中配置其他框架的位置需要使用classpath-->
        </bean>
        ```
  
    - Dao对象
  
      - MapperScannerConfigurer类可以为每个DAO对象创建代理对象，我们只需要指定SqlsessionFactory的bean和DAO所在的包
  
        ```xml
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
           <!-- 指定SqlsessionFactory的bean-->
            <property name="sqlSessionFactoryBeanName" value="SqlsessionFactory"/>
            <!-- 指定DAO包名,为该包下的每一个DAO接口执行一次getMapper()并将代理对象放置spring容器中-->
            <property name="basePackage" value="org.SM.DAO"/>
        </bean>
        ```
  
        - 包名可以指定多个只需要在后面添加','即可，DAO的代理对象名称是DAO接口的首字母小写
  
    - 声明自定义service
  
      ```xml
      <bean id="userservice" class="org.SM.services.iml.UserServicesImpl">
          <property name="userdao" ref="userDao"/>
      </bean>
      ```
  
  - 获取Service对象，通过该对象调用DAO完成数据库的访问
  
  - SM框架大致执行流程

![SM框架执行流程](D:\java框架\SM框架执行流程.png)