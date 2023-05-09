# Spring Cloud基础

- ## 上下文配置

  - bootstrap.yml

    - Spring Cloud应用程序通过创建“ bootstrap ”上下文来运行，该上下文是主应用程序的父上下文。它负责从外部源加载配置属性，并负责解密本地外部配置文件中的属性。这两个上下文共享一个`Environment`，它是任何Spring应用程序的外部属性的来源。默认情况下，引导程序属性（不是`bootstrap.properties`，而是引导程序阶段加载的属性）具有较高的优先级，**因此它们不能被本地配置覆盖。**
    - 可以在这里配置一个基本属性（服务名称、环境配置）
    - 可以通过设置`spring.cloud.bootstrap.name`（默认值：`bootstrap`），`spring.cloud.bootstrap.location`（默认值：空）或`spring.cloud.bootstrap.additional-location`（默认值：空）来指定`bootstrap.yml`（或`.properties`）位置。
      - additional-location：添加新的bootstrap.yml位置
      - bootstrap.location：指定bootstrap.yml唯一位置
    - 如果有远程的配置文件，默认情况下远程的配置文件优先级低于本地的，如果想要取消该机制则配置（在远程的配置文件中设置）spring.cloud.config.allowOverride=true

    - 日志配置请放在bootstrap中

  - 配置文件动态刷新

    - @RefreshScope类可以标注需要动态刷新配置的类，远程配置文件修改时，Spring容器将重新创建该Bean并重新读取配置值
    - RefreshScope作用域的beans是惰性代理，它们在使用时（即在调用方法时）进行初始化。
    - 如果bean是“不可变的”，则必须用`@RefreshScope`注释bean或在属性键`spring.cloud.refresh.extra-refreshable`下指定类名。

  - 配置文件加密和解密

    - springcloud提供基本的配置文件加密和解密功能

      ```sh
      //在jdk的jre目录的bin目录下有一个keytool.exe可以生成密钥
      keytool -genkeypair -alias config-server -keyalg RSA -keystore config-server.keystore -validity 365
      //指令执行成功后会在 bin 目录生成一个 config-server.keystore 文件，把该文件 copy 到配置中 心服务工程中 resources
      ```

      - 服务端工程配置 Properties 配置文件 添加秘钥配置

        ```java
        encrypt.key-store.location=config-server.keystore
        encrypt.key-store.alias=config-server
        encrypt.key-store.password=123456
        encrypt.key-store.secret=123456
            //之后在maven中添加该keystore文件
        ```

      - 通过请求http://localhost:8085/encrypt?data=123456可以加密或者解密（localhost:8085是配置中心地址）

  - Spring Boot Actuator（boot程序监控）

    - 对于Spring Boot Actuator应用程序，可以使用一些其他管理端点。您可以使用：
      - 从`POST`到`/actuator/env`以更新`Environment`并重新绑定`@ConfigurationProperties`和日志级别。
      - `/actuator/refresh`重新加载引导上下文并刷新`@RefreshScope` beans。
      - `/actuator/restart`关闭`ApplicationContext`并重新启动（默认情况下禁用）。
      - `/actuator/pause`和`/actuator/resume`用于调用`Lifecycle`方法（`ApplicationContext`中的`stop()`和`start()`）。

- ## Spring Cloud Commons

  -   @EnableDiscoveryClient

    - Spring Cloud Commons提供了`@EnableDiscoveryClient`批注。这将寻找`META-INF/spring.factories`与`DiscoveryClient`接口的实现。Discovery Client的实现在`org.springframework.cloud.client.discovery.EnableDiscoveryClient`键下将配置类添加到`spring.factories`。`DiscoveryClient`实现的示例包括[Spring Cloud Netflix Eureka](https://cloud.spring.io/spring-cloud-netflix/)，[Spring Cloud Consul发现](https://cloud.spring.io/spring-cloud-consul/)和[Spring Cloud Zookeeper发现](https://cloud.spring.io/spring-cloud-zookeeper/)。

    - 默认情况下，`DiscoveryClient`的实现会自动将本地Spring Boot服务器注册到远程发现服务器。可以通过在`@EnableDiscoveryClient`中设置`autoRegister=false`来禁用此行为。
    
    - 不再需要`@EnableDiscoveryClient`。您可以在类路径上放置`DiscoveryClient`实现，以使Spring Boot应用程序向服务发现服务器注册。
    
  - spring boot HealthIndicator整合
  
    - `DiscoveryClient`实现可以通过实现`DiscoveryHealthIndicator`来参与。要禁用复合`HealthIndicator`，请设置`spring.cloud.discovery.client.composite-indicator.enabled=false`。
  
  - 服务发现客户端顺序
  
    - `DiscoveryClient`接口扩展了`Ordered`。可以通过Order接口设置管理客户端加载顺序
    - 如果要为自定义`DiscoveryClient`实现设置不同的顺序，则只需覆盖`getOrder()`方法，以便它返回适合您的设置的值。
    - 除此之外，您可以使用属性来设置Spring Cloud提供的`DiscoveryClient`实现的顺序，其中包括`ConsulDiscoveryClient`，`EurekaDiscoveryClient`和`ZookeeperDiscoveryClient`。为此，您只需要将`spring.cloud.{clientIdentifier}.discovery.order`（对于Eureka，则为`eureka.client.order`）属性设置为所需的值。
  
  - 服务注册
  
    - 服务注册类都实现了ServiceRegistry 接口，该接口中包含了`register(Registration)`和`deregister(Registration)`之类的方法
  
      ```java
      public interface ServiceRegistry<R extends Registration> {
          void register(R registration);
      
          void deregister(R registration);
      
          void close();
      
          void setStatus(R registration, String status);
      
          <T> T getStatus(R registration);
      }
      
      //空类
      public interface Registration extends ServiceInstance {
      }
      //服务实例
      public interface ServiceInstance {
          default String getInstanceId() {
              return null;
          }
      
          String getServiceId();
      
          String getHost();
      
          int getPort();
      
          boolean isSecure();
      
          URI getUri();
      
          Map<String, String> getMetadata();
      
          default String getScheme() {
              return null;
          }
      }
      ```
  
    - 每个`ServiceRegistry`实现都有自己的`Registry`实现。
  
      - `ZookeeperRegistration`与`ZookeeperServiceRegistry`一起使用
      - `EurekaRegistration`与`EurekaServiceRegistry`一起使用
      - `ConsulRegistration`与`ConsulServiceRegistry`一起使用
  
  - 服务自动注册
  
    - 默认情况下，`ServiceRegistry`实现会自动注册正在运行的服务。要禁用该行为，可以设置：* `@EnableDiscoveryClient(autoRegister=false)`以永久禁用自动注册。* `spring.cloud.service-registry.auto-registration.enabled=false`通过配置禁用行为。
  
  - ServiceRegistry自动注册Events
  
    服务自动注册时将触发两个事件。注册服务之前会触发名为`InstancePreRegisteredEvent`的第一个事件。注册服务后，将触发名为`InstanceRegisteredEvent`的第二个事件。您可以注册`ApplicationListener`，以收听和响应这些事件。
    
  - Spring RestTemplate作为负载均衡器客户端
  
    - 示例
  
      ```java
      @Configuration
      public class MyConfiguration {
      
          @LoadBalanced
          @Bean
          RestTemplate restTemplate() {
              return new RestTemplate();
          }
      }
      
      public class MyClass {
          @Autowired
          private RestTemplate restTemplate;
      
          public String doOtherStuff() {
              //stores=服务名称
              String results = restTemplate.getForObject("http://stores/stores", String.class);
              return results;
          }
      }
      ```
  
      - 为了使用负载均衡的`RestTemplate`，您需要在类路径中具有负载均衡器实现。推荐的实现是`BlockingLoadBalancerClient`-添加`org.springframework.cloud:spring-cloud-loadbalancer`以便使用它。`RibbonLoadBalancerClient`也可以使用，但是目前正在维护中，我们不建议将其添加到新项目中。
      - 如果要使用`BlockingLoadBalancerClient`，请确保项目类路径中没有`RibbonLoadBalancerClient`，因为向后兼容的原因，默认情况下将使用它。
  
    - Spring WebClient作为负载均衡器客户端
    
      ```java
      @Configuration
      public class MyConfiguration {
      
      	@Bean
      	@LoadBalanced
      	public WebClient.Builder loadBalancedWebClientBuilder() {
      		return WebClient.builder();
      	}
      }
      
      public class MyClass {
          @Autowired
          private WebClient.Builder webClientBuilder;
      
          public Mono<String> doOtherStuff() {
              return webClientBuilder.build().get().uri("http://stores/stores")
              				.retrieve().bodyToMono(String.class);
          }
      }
      ```
    
  - 重试机制配置
  
    - 您可以使用`client.ribbon.MaxAutoRetries`，`client.ribbon.MaxAutoRetriesNextServer`和`client.ribbon.OkToRetryOnAllOperations`属性。
  
    - 如果要通过对类路径使用Spring重试来禁用重试逻辑，则可以设置`spring.cloud.loadbalancer.retry.enabled=false`
  
    - 自定义重试机制
  
      - 如果要在重试中实现`BackOffPolicy`，则需要创建`LoadBalancedRetryFactory`类型的bean并覆盖`createBackOffPolicy`方法：
  
        ```java
        @Configuration
        public class MyConfiguration {
            @Bean
            LoadBalancedRetryFactory retryFactory() {
                return new LoadBalancedRetryFactory() {
                    @Override
                    public BackOffPolicy createBackOffPolicy(String service) {
                		return new ExponentialBackOffPolicy();
                	}
                };
            }
        }
        ```
  
  - 导入负载均衡配置
  
    - 您还可以使用`@LoadBalancerClient`批注传递您自己的负载平衡器客户端配置，并传递负载平衡器客户端的名称和配置类，如下所示
  
    ```java
    @Configuration
    @LoadBalancerClient(value = "stores", configuration = StoresLoadBalancerClientConfiguration.class)
    public class MyConfiguration {
    
    	@Bean
    	@LoadBalanced
    	public WebClient.Builder loadBalancedWebClientBuilder() {
    		return WebClient.builder();
    	}
    }
    ```
  
    - LoadBalancerClients注解支持配置多个负载均衡客户端





