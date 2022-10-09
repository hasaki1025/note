# Spring Security

- 实现密码加密

  ```java
  @Configuration
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
      @Bean
      public PasswordEncoder passwordEncoder()
      {
          return new BCryptPasswordEncoder();
      }
  }
  ```

  - 在容器中注入BCryptPasswordEncoder作为加密方式

  - BCryptPasswordEncoder的作用

    ```java
    @Test
    void testPasswordEncode()
    {
        String encode = password.encode("123");//加密
        System.out.println(password.matches("123", encode));//校验
    }
    ```

- 使用jwt

  - 导入依赖

    ```xml
    <!-- https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.1</version>
    </dependency>
    
    ```

  - 