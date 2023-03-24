# Spring Cloud Netflix

- ## 基本服务注册和服务发现

  - 注册中心配置

    - 添加EurekaServer依赖

    - 主应用类(注册中心端口就是Web端口)

      ```java
      @SpringBootApplication
      @EnableEurekaServer
      public class Main {
          public static void main(String[] args) {
              SpringApplication.run(Main.class);
          }
      }
      ```

  - EurekaClient配置

    - 主应用类

      ```java
      @SpringBootApplication
      @RestController
      public class Main {
      
          @RequestMapping("/test")
          String test()
          {
              return "123";
          }
          public static void main(String[] args) {
              new SpringApplicationBuilder(Main.class).web(WebApplicationType.SERVLET).run(args);
          }
      }
      
      ```

    - 配置文件

      ```yml
      
      eureka:
        client:
          serviceUrl:
            defaultZone: http://localhost:8761/eureka/
      spring:
        application:
          name: provider
      server:
        port: 8081
      ```

  - ## Eureka客户端

    - 默认应用程序名称（即服务ID），虚拟主机和非安全端口（从`Environment`获取）分别为`${spring.application.name}`，`${spring.application.name}`和`${server.port}`。

    - 添加了Eureka客户端依赖可以是该应用同时具有服务调用方和服务提供方的功能

    - 要禁用Eureka Discovery Client，可以将`eureka.client.enabled`设置为`false`。当`spring.cloud.discovery.enabled`设置为`false`时，Eureka Discovery Client也将被禁用。

    - 如果其中一个`eureka.client.serviceUrl.defaultZone` URL内嵌了凭据，则HTTP基本身份验证会自动添加到您的eureka客户端（卷曲样式，如下：`http://user:password@localhost:8761/eureka`）。对于更复杂的需求，您可以创建类型为`DiscoveryClientOptionalArgs`的`@Bean`并将`ClientFilter`实例注入其中，所有这些实例都应用于从客户端到服务器的调用。

    - 使用https

      - 如果您希望通过HTTPS与您的应用进行联系，则可以在`EurekaInstanceConfig`中设置两个标志：
        - `eureka.instance.[nonSecurePortEnabled]=[false]`
        - `eureka.instance.[securePortEnabled]=[true]`

    - 心跳检测(只能在application.yml中设置)

      ```yml
      eureka:
        client:
          healthcheck:
            enabled: true
      ```

      - 如果您需要对运行状况检查进行更多控制，请考虑实施自己的`com.netflix.appinfo.HealthCheckHandler`。

    - eureka实例ID为主机名称:服务名称:端口,可以在eureka.instance.instanceId中修改名称

    - 可以通过注入EurekaClient 类获取到需要的实例信息，也可以通过注入DiscoveryClient 获取服务实例地址

      ```java
      @Autowired
      private EurekaClient discoveryClient;
      
      public String serviceUrl() {
          InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
          return instance.getHomePageUrl();
      }
      
      @Autowired
      private DiscoveryClient discoveryClient;
      
      public String serviceUrl() {
          List<ServiceInstance> list = discoveryClient.getInstances("STORES");
          if (list != null && list.size() > 0 ) {
              return list.get(0).getUri();
          }
          return null;
      }
      ```

    - 为什么注册服务这么慢

      - 成为实例还涉及到注册表的定期心跳（通过客户端的`serviceUrl`），默认持续时间为30秒。直到实例，服务器和客户端在其本地缓存中都具有相同的元数据后，客户端才能发现该服务（因此可能需要3个心跳）。您可以通过设置`eureka.instance.leaseRenewalIntervalInSeconds`来更改周期。将其设置为小于30的值可以加快使客户端连接到其他服务的过程。在生产中，最好使用默认值，因为服务器中的内部计算对租约续订期进行了假设。
    
    - 如何优先调用同一地区的客户端
    
      ```java
      eureka.instance.metadataMap.zone = zone1
      eureka.client.preferSameZoneEureka = true
      ```
    
    - EurekaClient刷新
      - 默认情况下，`EurekaClient` bean是可刷新的，这意味着Eureka客户端属性可以更改和刷新。发生刷新时，客户端将从Eureka服务器中注销，并且可能会在短暂的时间内不提供给定服务的所有实例。消除这种情况的一种方法是禁用刷新Eureka客户端的功能。为此，请设置`eureka.client.refresh.enable=false`。
  
- ## EurekaServer

  - 单机注册中心

    ```yml
    eureka:
      client:
        registerWithEureka: false #是否将自己注册到Server中
        fetchRegistry: false #是否从Server中获取注册信息
    ```

  - 多个注册中心

    ```yml
    spring:
      profiles: peer1
    eureka:
      instance:
        hostname: peer1
      client:
        serviceUrl:
          defaultZone: http://peer2/eureka/
    
    
    spring:
      profiles: peer2
    eureka:
      instance:
        hostname: peer2
      client:
        serviceUrl:
          defaultZone: http://peer1/eureka/
    ```

  - Eureka注册中心的保护

    - 将spring-boot-starter-security添加到类路径上将要求在每次向应用程序发送请求时都发送有效的CSRF令牌。

    - Eureka客户通常不会拥有有效的跨站点请求伪造（CSRF）令牌，您需要为`/eureka/**`端点禁用此要求。例如：

      ```java
      
      @EnableWebSecurity
      class WebSecurityConfig extends WebSecurityConfigurerAdapter {
      
          @Override
          protected void configure(HttpSecurity http) throws Exception {
              http.csrf().ignoringAntMatchers("/eureka/**");
              super.configure(http);
          }
      }
    ```
      

- ## OpenFeign

  - 基本使用

    - 创建公共接口模块

    - 添加spring-cloud-starter-openfeign依赖

    - 创建Feign调用公共接口

      ```java
      @FeignClient("Provider")
      //可以是服务名称（自带负载均衡），也可以指定URL属性定向指定
      //qualifier属性可以指定该Bean注入的时候的名称
      public interface HelloService {
      
          @GetMapping("/hello")
          String hello();
      }
      ```

    - 创建消费者模块并在主应用类上添加EnableFeignClients注解并添加扫描包（公共接口的包）

      ```java
      @EnableFeignClients(basePackages = {"com.alibabaTest.Consumer","Service"})
      ```

    - 使用Autowired等注入接口类并调用

  - @FeignClient属性

    - name/value：指定FeignClient的名称，如果项目使用了Ribbon，name属性会作为微服务的名称，用于服务发现

    - contextId：如果没有指定别名的话就使用contextId+FeignClient作为Bean的别名（使用Qualifier注入）

    - url: url一般用于调试，可以手动指定@FeignClient调用的地址

      - `name`和`url`属性中支持占位符。

        ```java
        @FeignClient(name = "${feign.name}", url = "${feign.url}")
        public interface StoreClient {
            //..
        }
        ```

    - decode404:当发生http 404错误时，如果该字段位true，会调用decoder进行解码，否则抛出FeignException

    - configuration: Feign配置类，可以自定义Feign的Encoder、Decoder、LogLevel、Contract

      - `Feign配置类`不需要用`@Configuration`进行注释。但是，如果是的话，请注意将其从任何可能包含此配置的`@ComponentScan`中排除，因为它将成为`feign.Decoder`，`feign.Encoder`，`feign.Contract`等的默认来源，指定时。可以通过将其与任何`@ComponentScan`或`@SpringBootApplication`放在单独的，不重叠的包中来避免这种情况，也可以在`@ComponentScan`中将其明确排除在外。
      - Spring Cloud Netflix 默认情况下*不会*为feign提供以下beans，但仍会从应用程序上下文中查找以下类型的beans以创建feignClient：
      - 除了使用配置类还可以通过配置文件设置相关类的实现
      - 除了在FeignClient注解上声明配置类还可以在EnableFeignClients上设置配置类但是这样的配置是全局配置

    - fallback: 定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback指定的类必须实现@FeignClient标记的接口

      - fallback使用

        ```java
        @Component
        @Slf4j
        public class HelloFallBack implements HelloService {
            @Override
            public String hello() {
                log.error("request Error");
                return "error";
            }
        }
        ```

        - **fallback和下面的fallbackFactory都是基于断路器实现的，所以使用前需要开启断路器**

    - fallbackFactory: 工厂类，用于生成fallback类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码

      ```java
      @Component
      @Slf4j
      public class HelloFallBackFactory  implements FallbackFactory<HelloService> {
      
          @Autowired
          HelloFallBack helloFallBack;
          @Override
          public HelloService create(Throwable cause) {
              log.error("exception :{}================",cause.getMessage());
              cause.printStackTrace();
              return helloFallBack;
          }
      }
      
      ```

      - fallbackFactory和fallback两者只能有一个生效

    - path: 定义当前FeignClient的统一前缀(请求的URL)

    - primary：默认为true，代表带有FeignClient的类注册的Bean都类似于添加了@Primary，优先进行注入，不然对于fallback而言，同样实现了相同的接口，使用autowired可能会产生歧义

  - 手动创建feignClient

    - 可以通过以下形式手动创建特定的feignClient
  
      ```java
      @Import(FeignClientsConfiguration.class)
      class FooController {
      
          private FooClient fooClient;
      
          private FooClient adminClient;
      
          @Autowired
          public FooController(Decoder decoder, Encoder encoder, Client client, Contract contract) {
              this.fooClient = Feign.builder().client(client)
                  .encoder(encoder)
                  .decoder(decoder)
                  .contract(contract)
                  .requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
                  //设置URL
                  .target(FooClient.class, "http://PROD-SVC");
      
              this.adminClient = Feign.builder().client(client)
                  .encoder(encoder)
                  .decoder(decoder)
                  .contract(contract)
                  .requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
                  .target(FooClient.class, "http://PROD-SVC");
          }
      }
      ```

      - FeignClientsConfiguration是feignClient的默认配置（编码器、解码器等）

  - 我们可以通过继承的形式创建公共的API

    - UserService(公共接口)
  
      ```java
      public interface UserService {
      
          @RequestMapping(method = RequestMethod.GET, value ="/users/{id}")
          User getUser(@PathVariable("id") long id);
      }
      ```

    - 服务方实现类
  
      ```java
      @RestController
      public class UserResource implements UserService {
      //实现
      }
      ```

    - 消费者创建FeignClient
  
      ```java
      package project.user;
      
      @FeignClient("users")
      public interface UserClient extends UserService {
      
      }
      ```

      - 通常不建议在服务器和客户端之间共享接口。它引入了紧密耦合，并且实际上也不能与当前形式的Spring MVC一起使用（方法参数映射不被继承，其就是实现类实现了接口的方法后，方法参数上的注解会丢失）。建议使用上面示例采用的方法（使用公共接口最后在EnableFeignClients上指定包）

  - 日志设置
  
    ```yaml
    logging.level.project.user.UserClient: DEBUG
    ```

    - `NONE`，无日志记录（**DEFAULT**）。

    - `BASIC`，仅记录请求方法和URL以及响应状态代码和执行时间。

    - `HEADERS`，记录基本信息以及请求和响应头。

    - `FULL`，记录请求和响应的标题，正文和元数据。

    - 也可以通过配置类实现
  
      ```java
      @Slf4j
      public class FeignConfiguration {
      
          @Bean
          Logger.Level  logLevel()
          {
              System.out.println("logger full");
              return Logger.Level.FULL;
          }
      
      
      }
      ```
      
    - **注意：一定要配置springboot的日志级别（记得导入lombok）**
    
      - 如果采用上面示例的方式，feignclient接口所在的包是没有配置文件的（也就是没有springboot的日志覆盖，所以需要对接口所在的包进行日志配置）
    
  - 请求参数POJO支持
  
    - Spring Cloud OpenFeign提供等效的`@SpringQueryMap`注解，该注解能够将实体类转换为get请求的表单数据
  
      - User类
  
      ```java
      @Data
      public class User {
      
          String name;
          String password;
      }
      
      ```
  
      - 接口
  
        ```java
        @FeignClient( name ="Provider" ,contextId = "hello",configuration = FeignConfiguration.class)
        public interface HelloService {
        
            @GetMapping("/hello")
            String hello();
            @PostMapping("/register")
            String registerUser(@SpringQueryMap User user);
        }
        
        ```
  
      - 错误的调用方式
  
        ```java
        //POJO
        @Data
        public class User {
        
            String name;
            String password;
        }
        
        @Data 
        publice class User2
        {
            String name;
        }
        
        @PostMapping("/register")
        String registerUser(@SpringQueryMap User user,@SpringQueryMap User2 user2);
        
        ```
  
        - User和User2具有相同的字段名称---name,但是url上无法区分这两个属性
  
- ## Hystrix断路器

  - 雪崩效应

    - 当A服务依赖于B服务，而D、C服务依赖于B服务，一旦A服务崩了，B随即也会崩，D、C之后也会崩溃，最终整个系统不可用，为了防止雪崩效应则有了断路器

  - Hystrix是一个可以解决雪崩效应，可用于**处理分布式系统的延迟和容错开源库**，**能够保证在一个依赖出问题的情况下，不会导致整个体系服务失败，避免级联故障，从而提高分布式系统的弹性**。

    - 我们可以把Hystrix当成是一个“断路器”，当我们某个服务单元发生故障之后，通过断路器的故障监控 (类似熔断保险丝) ，向调用方返回一个服务预期的，可处理的备选响应 (FallBack) ，而不是长时间的等待或者抛出调用方法无法处理的异常，这样就可以保证了服务调用方的线程不会被长时间，不必要的占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

  - 具体作用

    - 服务降级
    - 服务熔断
    - 服务限流
    - 实时监控

    - 较低级别的服务中的服务故障可能会导致级联故障，直至用户。在`metrics.rollingStats.timeInMilliseconds`定义的滚动窗口中，当对特定服务的调用超过`circuitBreaker.requestVolumeThreshold`（默认：20个请求）并且失败百分比大于`circuitBreaker.errorThresholdPercentage`（默认：> 50％）时（默认：10秒） ），则电路断开并且无法进行呼叫

  - 使用

    - 添加依赖

    - 主应用类上添加@EnableHystrix

    - 在需要服务降级的方法上添加HystrixCommand注解

      ```java
      @RequestMapping("/hello")
      @Override
      @HystrixCommand(fallbackMethod = "fallback",commandProperties = {
          //2秒钟以内就是正常的业务逻辑
          @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000")
      })
      public String hello()
      {
          try {
              sleep(5000);
          } catch (InterruptedException e) {
              throw new RuntimeException(e);
          }
          return "nihao";
      }
      public String fallback()
      {
          return "error";
      }
      ```

  - HystrixProperty注解

    - 可以通过HystrixProperty配置Hystrix

  - OpenFeign开启Hystrix

    ```yaml
    feign:
      circuitbreaker:
        enabled: true    
    ```

  - 

