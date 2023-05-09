# Spring Cloud Alibaba

- ## Nacos

  - 基本配置

    - 添加nacos config依赖和nacos discovery依赖

    - 添加Nacos配置

      ```yaml
      spring:
        application:
          name: Consumer
        cloud:
          nacos:
            server-addr: 192.168.10.10:8848
            config:
              file-extension: yml
      ```

      - 在bootstrap.yml中添加（注意以下错误）

      - 在bootstrap.yml 文件中配置注册服务的地址信息时，启动服务报错：Param ‘serviceName’ is illegal, serviceName is blank 

        - 原因：在spring-cloud-dependencies 2020.0.0 版本不在默认加载bootstrap 文件，如果需要加载bootstrap 文件需要手动添加依赖。

      - 主应用类上添加@EnableDiscoveryClient(似乎不添加也可以)，在需要进行配置热更新的类上添加RefreshScope注解

        ```java
        @RestController
        @RefreshScope
        public class RefreshConfig {
        
            @Value("${myconfig.str}")
            String str;
        
        
            @RequestMapping("/test")
            String test()
            {
                return str;
            }
        }
        
        ```

      - 配置文件发布格式

        - 发布默认格式为：${prefix} - ${spring.profiles.active} . ${file-extension}，如果没有环境名称则为${prefix} . ${file-extension}

        - prefix默认为服务名称，可以通过spring.cloud.nacos.config.prefix配置

        - group配置

          - 可通过spring.cloud.nacos.config.group配置

        - 其他配置

          #### 更多配置项

          

          | 配置项                   | key                                       | 默认值                     | 说明                                                         |
          | ------------------------ | ----------------------------------------- | -------------------------- | ------------------------------------------------------------ |
          | 服务端地址               | spring.cloud.nacos.config.server-addr     |                            | 服务器ip和端口                                               |
          | DataId前缀               | spring.cloud.nacos.config.prefix          | ${spring.application.name} | DataId的前缀，默认值为应用名称                               |
          | Group                    | spring.cloud.nacos.config.group           | DEFAULT_GROUP              | 根据业务相互隔离                                             |
          | DataId后缀及内容文件格式 | spring.cloud.nacos.config.file-extension  | properties                 | DataId的后缀，同时也是配置内容的文件格式，目前只支持 properties |
          | 配置内容的编码方式       | spring.cloud.nacos.config.encode          | UTF-8                      | 配置的编码                                                   |
          | 获取配置的超时时间       | spring.cloud.nacos.config.timeout         | 3000                       | 单位为 ms                                                    |
          | 配置的命名空间           | spring.cloud.nacos.config.namespace       | public                     | 常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源隔离等。该配置项是拉取配置的命名空间，服务注册的命名空间配置为nacos.discovery.namespace |
          | AccessKey                | spring.cloud.nacos.config.access-key      |                            |                                                              |
          | SecretKey                | spring.cloud.nacos.config.secret-key      |                            |                                                              |
          | 相对路径                 | spring.cloud.nacos.config.context-path    |                            | 服务端 API 的相对路径                                        |
          | 接入点                   | spring.cloud.nacos.config.endpoint        |                            | 地域的某个服务的入口域名，通过此域名可以动态地拿到服务端地址 |
          | 是否开启监听和自动刷新   | spring.cloud.nacos.config.refresh-enabled | true                       |                                                              |
          | 集群服务名               | spring.cloud.nacos.config.cluster-name    |                            |                                                              |

          - 不同组或者不同namespace之间无法通信
  
- ## Sentinel

  - 使用

    - 引入依赖（已包含在spring cloud start依赖管理中）

    - 使用`@SentinelResource` 注解用来标识资源是否被限流、降级。

      ```java
      @SentinelResource(fallback = "fallback")
      @RequestMapping("/testSentinel")
      public String testSentinel() throws Throwable {
          throw new Throwable();
      }
      
      public String fallback(Throwable e)
      {
          e.printStackTrace();
          return "error";
      }
      ```

      - 除了可以添加在请求方法上还可以添加在普通类上（表示被限制的资源）

  - SentinelResource注解

    - blockHandler属性
      - 当服务被限流，被拒绝的请求将会走blockHandler指定的方法，blockHandler指定的方法必须在SentinelResource所在的类中
    - blockHandlerClass
      - 如果想要使用相同的限流逻辑可以指定Class（同时需要指定blockHandler，blockHandler同样作为方法名，且blockHandler指定的方法名称必须时static方法）
    - fallback
      - 请求失败的回调函数的方法名称
    - fallbackClass
      - 和blockHandlerClass基本类似（同样要求fallback指定的方法是静态方法）
    - exceptionsToTrace
      - 指定需要关注的异常类型
    - exceptionsToIgnore
      - 可忽略的异常类型
    - defaultFallback
      - 默认采用的回调函数方法名称，该方法不应该有任何参数，返回值和SentinelResource标注的方法相同

  - feign支持

    - 启动sentinel支持

      ```yml
      feign:
        sentinel:
          enabled: true
      ```

  - 
