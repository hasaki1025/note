# Rabbit MQ

- ## 基本作用

  - 流量削峰
    - 对访问流量进行缓冲
  - 应用解耦
    - 不同系统之间的调用可以使用MQ代替
  - 异步处理
    - MQ作为中间商，假设A调用B，A发起调用后就可以继续执行之前的任务，当B执行完后将结果放置在MQ中，MQ再将消息转发给A

- ## 基本分类

  - ActiveMQ 
    - 功能齐全，稳定性高，官方维护较少
  - Kafka
    - 大数据杀手，具有百万级TPS吞吐量，社区更新慢
  - RocketMQ
    - 单机吞吐量10w级，阿里巴巴的开源产品，java编写，缺点是支持的客户端语言不多
  - RabbitMQ
    - 吞吐量万级，功能完备，稳定，易用，社区活跃度高

- ## 基本概念

  - 生产者
    - 发送消息的角色
  - 交换机
    - 一方面接受生产者的消息，另一方面将消息推送到队列中
  - 队列
    - 存放消息的缓冲区
  - 消费者
    - 获取消息的角色

- ## 核心部分

  - 简单模式

    - 当生产者发送消息到交换机,交换机根据消息属性发送到队列,消费者监听绑定队列实现消息的接收和消费逻辑编写.简单模式下,强调的一个队列queue只被一个消费者监听消费.

      ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221185542407.png)

    - 1.2 应用场景
      常见的应用场景就是一发,一接的结构
      例如:
      手机短信,邮件单发

  - 工作模式

    - ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221185625265.png)
    - 生产者：发送消息到交换机
      交换机：根据消息属性将消息发送给队列
      消费者：多个消费者,同时绑定监听一个队列,之间形成了争抢消息的效果
    - 2.2 应用场景
      抢红包
      资源分配系统

  - 发布订阅模式

    - 交换机将消息发送给所有队列
    - ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221190423935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTMwNzg5Ng==,size_16,color_FFFFFF,t_70)
    - 应用场景
      邮件的群发,广告的群发

  - 路由模式

    - 将消息携带routing key和队列绑定的routing key，如果匹配上了，就把消息发送给指定队列

      ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221190103316.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTMwNzg5Ng==,size_16,color_FFFFFF,t_70)

      

  - 主题模式

    - 类似于路由模式，区别在于队列可以接受具有相同规则的消息

      ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221190710636.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTMwNzg5Ng==,size_16,color_FFFFFF,t_70)

    - 应用场景
      实现多级传递的路由筛选工作

  - RPC异步调用模式（不常用）

- ## 基本原理

  - [Rabbit原理]: https://blog.csdn.net/weixin_43498985/article/details/119026198

- ## 安装以及启动

  - 安装教程 https://blog.csdn.net/shishishilove/article/details/122086399

  - 启动命令

    ```shell
    /sbin/service rabbitmq-server start
    ```

  - 停止服务

    ```shell
    /sbin/service rabbitmq-server stop
    ```

  - 查看服务当前状态

    ```shell
    /sbin/service rabbitmq-server status
    ```

  - 开启web管理插件

    ```shell
    rabbitmq-plugins enable rabbitmq_management
    ```

    - 随后通过15672端口登录
    - 默认初始密码和账号都是guest

  - 添加新用户

    ```shell
    rabbitmqctl add_user root root
    ```

    - 用户名为root，密码是root

  - 设置用户角色

    ```shell
    rabbitmqctl set_user_tags root administrator
    ```

    - 一共有五种角色(同一用户可以设置多个角色)

      ```shell
      administrator
      
      monitoring 
      可登陆管理控制台(启用management plugin的情况下)，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)
      
      policymaker
      可登陆管理控制台(启用management plugin的情况下), 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。
      
      management
      仅可登陆管理控制台(启用management plugin的情况下)，无法看到节点信息，也无法对策略进行管理。
      
      其他
      无法登陆管理控制台，通常就是普通的生产者和消费者。
      ```

      

  - 删除用户

    ```shell
    rabbitmqctl delete_user root 
    ```

  - 修改用户密码

    ```shell
    rabbitmqctl change_password root root
    ```

  - 查看当前用户列表

    ```shell
    rabbitmqctl list_users
    ```

  - 设置权限

    ```shell
    rabbitmqctl set_permissions -p "/" root ".*" ".*" ".*"
    ```

    - 具体格式为 set_permissions [-p <vhostpath>] <user> <conf> <write> <read>

  - 清除用户权限

    ```shell
    rabbitmqctl clear_permissions [-p VHostPath] Username
    ```

  - 登录后也可以通过Web页面直接添加用户

- ## 编写入门案例(使用RabbitMQ简单模式)

  - 编写生产者代码
  
    ```java
    @SpringBootTest
    class RebbitMQdemoApplicationTests {
    
        public static final String QueueName="hello";
        @Test
        void contextLoads() throws IOException, TimeoutException {
            //创建连接工厂
            ConnectionFactory factory=new ConnectionFactory();
    
            //设置主机IP等信息
            factory.setHost("192.168.10.10");
            factory.setUsername("root");
            factory.setPassword("root");
    
            //创建连接
            Connection connection = factory.newConnection();
            //创建信道
            Channel channel = connection.createChannel();
            //创建队列
            //第一个参数为队列名称
            //第二个参数为是否进行持久化
            //第三个参数代表是否消费费者独占，即该队列是否只能由一个消费者接受
            //第四个参数代表最后一个消费者断开连接是否自动删除队列信息
            //第五个参数是Map集合，用作其他参数
            channel.queueDeclare(QueueName,false,false,true,null);
            //消息发送
            String message="hell world";
            /*
                第一个参数：交换机名称
                第二个参数：路由key，本次为队列名称
                第三个参数：其他参数信息
                第四个参数：消息体
            * */
            channel.basicPublish("",QueueName,null,message.getBytes(StandardCharsets.UTF_8));
    
        }
    
    }
    ```
  
    - 注意：这里factory的用户需要设置权限和访问路径，不然为报错
    - 发送过后可以在Web页面中查看到
  
  - 编写消费者代码
  
    ```java
     @Test
        void consume() throws IOException, TimeoutException {
            //创建连接工厂
            ConnectionFactory factory=new ConnectionFactory();
    
            //设置主机IP等信息
            factory.setHost("192.168.10.10");
            factory.setUsername("root");
            factory.setPassword("root");
    
    
            //创建连接
            Connection connection = factory.newConnection();
            //创建信道
            Channel channel = connection.createChannel();
            //接受消息
            /*
            basicConsume有很多方法重载，以下为常用的一种
            第一个参数：String队列名称
            第二个参数：boolean是否自动回复，true为自动应答
            第三个参数：成功接受消息时的回调接口，类型是DeliverCallback
            第四个参数：当一个消费者取消订阅时的回调接口类型是一个CancelCallback
            * */
            DeliverCallback deliverCallback=(s, delivery) -> System.out.println(new String(delivery.getBody()));
            CancelCallback cancelCallback=System.out::println;
    
            channel.basicConsume(QueueName,false, deliverCallback, cancelCallback);
        }
    ```
  
    - 其中成功接受消息时的回调接口为
  
      ```java
      @FunctionalInterface
      public interface DeliverCallback {
          void handle(String var1, Delivery var2) throws IOException;
      }
      
      ```
  
    - 取消订阅时的回调接口是
  
      ```java
      @FunctionalInterface
      public interface CancelCallback {
          void handle(String var1) throws IOException;
      }
      
      ```
  
- ## 消息应答

  - ### 概念

    - 当一个接受一个很长的消息，如果采用自动应答，在消息还没有接受完成后就自动应答会导致消息被删除，如果此时接受消息过程中中断会导致该消息的丢失。
    - 消息应答机制就是在消费者就接受到消息之后告诉RabbitMQ该消息已被处理，RabbitMQ可以将该消息删除

  - ### 自动应答

    - 需要良好的环境，在高吞吐量和数据传输安全方面不会产生问题时采用

  - ### 手动应答

    - 使用信道调用basicAck方法手动应答
    - 三个方法
      - basicAck（肯定应答）
      - basicNack（否定应答）可批量应答，也就是将后续所有消息全部应答
      - basicReject（否定应答）不可批量应答

    - 手动应答(消费者)代码实现

      ```java
      RabbitMQUtil mqUtil = new RabbitMQUtil();
              Channel channel = mqUtil.getChannel();
              DeliverCallback deliverCallback=(var1,delivery)-> {
                  //输出消息
                  System.out.println(new String(delivery.getBody()));
                  try {
                      //等待一段时间后发送确认
                      Thread.sleep(1000);
                      //获取消息的编号
                      long deliveryTag = delivery.getEnvelope().getDeliveryTag();
                      //应答消息
                      channel.basicAck(deliveryTag,false);
                  } catch (InterruptedException e) {
                      throw new RuntimeException(e);
                  }
              };
              CancelCallback cancelCallback=System.out::println;
              //接受消息
              channel.basicConsume(QueueName,false,deliverCallback,cancelCallback);
      ```

  - ### 消息重新入队机制

    - 当消费者由于某些原因导致没有对消息发送ACK确认则通过RabbitMQ将消息重新入队
    - 多线程下，也就是工作模式下，假设线程C1和线程C2都在接受hello队列中的消息，当C1在接受消息的途中突然宕机，则消息会重新入队并之后会交给C2处理

- ## 持久化

  - ### 队列持久化

    - 在说明队列的时候可以将持久化的标志(第二个参数)设置为true

      ```java
      channel.queueDeclare(QueueName, true, false, true, null);
      ```

    - 可以在Web页面上查看是否队列存在持久化

      ![image-20220702161636074](C:/Users/pcdn/AppData/Roaming/Typora/typora-user-images/image-20220702161636074.png)

  - ### 消息持久化

    - 在消息发布时添加参数完成持久化(第三个参数从NULL修改为MessageProperties.PERSISTENT_BASIC)

      ```java
       channel.basicPublish("",QueueName, MessageProperties.PERSISTENT_BASIC,message.getBytes(StandardCharsets.UTF_8));
      ```

  - ### 不公平分发

    - 通过basicQos控制RabbitMQ发送速率

      ```java
      int prefetchCount=1;
      //设置为1表示当该消费者从接受到该消息到发送应答之前RabbitMQ都不会将消息转发给该消费者
      channel.basicQos(prefetchCount);
      ```

      - 其实就是限制接受速率，一次只接受一条消息

  - ### 发布确认
  
    - 生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式， 所有在该信道上面发布的
      消息都将会被指派一个唯一的 ID(从 1 开始)，一旦消息被投递到所有匹配的队列之后，broker
      就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队
      列了
      
      ```java
      //开启发布确认
      channel.confirmSelect();
      ```
      
      - 发布确认的三种模式
      
        - 单个确认发布
      
          - 发送一条信息就确认一条信息
      
            ```java
             //这里使用自定义的工具类创建信道
            RabbitMQUtil rabbitMQUtil = new RabbitMQUtil();
            Channel channel = rabbitMQUtil.getChannel();
            channel.confirmSelect();
            channel.queueDeclare(QueueName, true, false, true, null);
            for (int i = 0; i < 10; i++) {
                String message="hello"+i;
               	 channel.basicPublish("","hello",MessageProperties.PERSISTENT_BASIC,message.getB	ytes(StandardCharsets.UTF_8));
                //等待确认，为true代表发送成功
                if (channel.waitForConfirms()) {
                    System.out.println("send message successfully");
                }
            }
            ```
      
            - 缺点：发布速度较慢
      
        - 批量发布确认
      
          - 一次发送一堆消息，同时确认也是一次确认一堆消息，如果中间消息存在错误也不知道是哪一条消息错误
        
            ```java
            RabbitMQUtil rabbitMQUtil = new RabbitMQUtil();
                    Channel channel = rabbitMQUtil.getChannel();
                    channel.confirmSelect();
                    channel.queueDeclare(QueueName,true,false,false,null);
            
                    for (int i = 0; i < 10; i++) {
                        channel.basicPublish("","hello",MessageProperties.PERSISTENT_BASIC,new String("hello"+i).getBytes(StandardCharsets.UTF_8));
                    }
                    if (channel.waitForConfirms()) {
                        System.out.println("send batch message successfully");
                    }
            ```
        
        - 异步批量发布确认
        
          - 生产者只负责发送，而消息到达信道后，信道将消息发送给broker
          
            ![image-20220712130819404](Rabbit MQ.assets/image-20220712130819404.png)
          
            ```java
            RabbitMQUtil rabbitMQUtil = new RabbitMQUtil();
                    Channel channel = rabbitMQUtil.getChannel();
                    channel.confirmSelect();
                    channel.queueDeclare(QueueName,true,false,false,null);
                    //设置消息发送成功时回调的接口
                    ConfirmCallback ackconfirmCallback=(long var1, boolean var3)-> System.out.println("消息"+var1+"成功接受");
                    //设置消息发送失败时回调的接口
                    ConfirmCallback nackconfirmCallback=(long var1, boolean var3)-> System.out.println("消息"+var1+"接受失败");
                    //添加消息监视器
                    channel.addConfirmListener(ackconfirmCallback,nackconfirmCallback);
                    for (int i = 0; i < 10; i++) {
                        String message="hello"+i;
                        //慢点发，太快发送会来不及确认
                        Thread.sleep(1000);
                        channel.basicPublish("","hello",MessageProperties.PERSISTENT_BASIC,message.getBytes(StandardCharsets.UTF_8));
                    }
            ```
          
          - 以上代码只对成功发送的消息进行了处理，如果消息未发送成功，却没有任何处理，事实上开启消息监听器是开启一个新的线程，该线程负责监听消息的发送和确认，如果消息没有成功发送，消息监听器需要对其做出响应
          
          - 最好的解决方法是为发送消息的线程和消息监听器的线程之间创建一个线程共享哈希Map，每次发送将消息保存在Map中，成功收到消息后将消息从Map中删除，未成功收到则将消息保存
          
          - 添加一个线程安全的数据结构
          
            ```java
            ConcurrentHashMap<Long, String> map = new ConcurrentHashMap<>();
            ```
          
          - 修改消息成功发送时回调的接口
          
            ```java
            ConfirmCallback ackconfirmCallback=(long var1, boolean var3)->{
                        map.remove(var1);
                        System.out.println("消息"+var1+"成功接受");
                    };
            ```
          
          - 修改消息发送失败时回调的接口
          
            ```java
             ConfirmCallback nackconfirmCallback=(long var1, boolean var3)-> {
                        System.out.println("消息"+var1+"接受失败");
                    };
            ```
          
          - 消息发送
          
            ```java
              for (int i = 0; i < 10; i++) {
                        String message="hello"+i;
                        //慢点发，太快发送会来不及确认
                        Thread.sleep(1000);
                        map.put(channel.getNextPublishSeqNo(),message);
                        channel.basicPublish("","hello",MessageProperties.PERSISTENT_BASIC,message.getBytes(StandardCharsets.UTF_8));
                    }
            ```
          
          - 最后遍历Map集合
          
            ```java
             BiConsumer<Long, String> biConsumer=(l,r)-> System.out.println("消息："+l+"为"+r);
                    map.forEach(biConsumer);
            ```
          
          - 最后结果
          
            ```java
            消息1成功接受
            消息2成功接受
            消息3成功接受
            消息4成功接受
            消息5成功接受
            消息6成功接受
            消息7成功接受
            消息8成功接受
            消息9成功接受
            消息：10为hello9
            消息10成功接受
            ```
          
          - 可以看到消息1~9发送之后执行了map集合的遍历，但是此时消息确认线程还未全部确认完毕

- ## 交换机

  - ### 发布订阅模式
  
    - 一条消息可以被多个消费者接收
  
    - 消息能由路由发送到队列通过routingkey绑定的key指定的队列，如果交换机采用默认交换机则routingkey为basicPublish的第二个参数指定的交换机名称
  
      ![image-20220713150002838](Rabbit MQ.assets/image-20220713150002838.png)
  
  - ### 交换机的类型
  
    - 直接类型（direct）
      - 根据RouteKey对消息进行转发，一个direct可以绑定多个routekey
    - 主题类型（topic）
    - 标题类型（headers）
    - 扇出（fanout）
      - 将发送给自己的消息全部转发出去（洪泛转发）
      - routKey在该类交换机上不起作用
  
  - ### 临时队列
  
    - 一旦断开消费者的连接则该队列就会被删除，临时队列的名称由RabbitMQ随机指定，也就是在创建队列时指定第四个参数（是否自动删除）为true
  
      ![image-20220713150826145](Rabbit MQ.assets/image-20220713150826145.png)
  
    - 临时队列的Features标志为空
  
    - 创建一个临时队列
  
      ```java
      System.out.println(channel.queueDeclare().getQueue());
      //输出结果
      //amq.gen-qbMnaYx1UcFp6sDeYNCwlA
      ```
  
  - ### 绑定操作
  
    - 绑定交换机和队列之间的关系
  
      ```java
      channel.queueBind(queue,"exchange1","123");
      //第一个参数是队列名称
      //第二个参数是交换机名称
      //第三个次数是路由key
      ```
  
      - 也可以在Web网页上手动添加交换机和队列并绑定
  
  - ### Fanout交换机
  
    - 类似广播
  
    - 只要队列绑定了该交换机就会收到该交换机收到的所有消息，无关routeingkey，且该交换机的速度也是最快的
  
    - 案例：两个消费者共同消费
  
      - 消费者
  
      ```java
      public class fanoutConsumer {
          public static final String EXCHANE_NAME="fanoutExchange";
          public static final String FANOUT_ROUTING_KEY="fanout1";
          public static void main(String[] args) {
              try {
                  Channel channel = RabbitMqUtil.getChannel();
                  //创建fanout交换机
                  channel.exchangeDeclare(EXCHANE_NAME, BuiltinExchangeType.FANOUT);
                  //创建临时队列
                  String queueName = channel.queueDeclare().getQueue();
                  //绑定队列和交换机
                  channel.queueBind(queueName,EXCHANE_NAME,FANOUT_ROUTING_KEY);
                  //接受消息
                  channel.basicConsume(queueName,true,(tag,deliver)->{
                      System.out.println("consumer message > message tag: "+tag);
                      System.out.println("message body: "+new String(deliver.getBody()));
                  },System.out::println);
              } catch (IOException | TimeoutException e) {
                  e.printStackTrace();
              }
          }
      }
      
      ```
  
      - 生产者
  
        ```java
        public class fanoutProvider {
            public static void main(String[] args) throws IOException, TimeoutException {
                Channel channel = RabbitMqUtil.getChannel();
                for (int i = 0; i < 10; i++) {
                    String message="message"+i;
                    channel.basicPublish(fanoutConsumer.EXCHANE_NAME,fanoutConsumer.FANOUT_ROUTING_KEY, MessageProperties.MINIMAL_BASIC,message.getBytes(StandardCharsets.UTF_8));
                }
        
            }
        }
        
        ```
  
  - ### Direct交换机
  
    - 通过不同的Routingkey进行路由转发
  
    - 队列和交换机声明与绑定
  
      ```java
      public class directExchangeDeclare {
          public static final String EXCHANGE_NAME="direct";
      
          public static void main(String[] args) throws IOException, TimeoutException {
              Channel channel = RabbitMqUtil.getChannel();
              channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
              channel.queueDeclare("info",true,false,true,new HashMap<>());
              channel.queueDeclare("debug",true,false,true,new HashMap<>());
              channel.queueDeclare("error",true,false,true,new HashMap<>());
              channel.queueBind("info",EXCHANGE_NAME,"1");
              channel.queueBind("debug",EXCHANGE_NAME,"2");
              channel.queueBind("error",EXCHANGE_NAME,"3");
          }
      }
      
      ```
  
  - ### Topic交换机
  
    - 类似与广播但和广播不同，可以理解为可以该交换机可以通过类似与正则表达的形式将消息转发给符合条件的队列
  
    - Topic交换机绑定的routekey可以使用*(代表一个单词)和#（代表多个单词），如果发送message.info和message.debug消息，则绑定的key为message. * 的队列便可收到消息
  
      - 群发案例
  
        - 队列和交换机声明
  
        ```java
        public class TopicExchangeDeclare {
            public static final String EXCHANGE_NAME="topic";
            public static void main(String[] args) throws IOException, TimeoutException {
                Channel channel = RabbitMqUtil.getChannel();
                channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
                channel.queueDeclare("message",false,false,false,new HashMap<>());
                channel.queueDeclare("message.info",false,false,false,new HashMap<>());
                channel.queueDeclare("message.debug",false,false,false,new HashMap<>());
                channel.queueBind("message.info",EXCHANGE_NAME,"message.1",new HashMap<>());
                channel.queueBind("message.debug",EXCHANGE_NAME,"message.2",new HashMap<>());
                channel.queueBind("message",EXCHANGE_NAME,"message.*",new HashMap<>());
            }
        }
        
        ```
        
        - 消费者
        
          ```java
          public class MessageConsumer {
              public static void main(String[] args) throws IOException, TimeoutException {
                  Channel channel = RabbitMqUtil.getChannel();
                  channel.basicConsume("message.info",true,(tag,deliver)->{
                      System.out.println("message routing key: "+deliver.getEnvelope().getRoutingKey());
                      System.out.println("message body: "+ new String(deliver.getBody()));
                  },System.out::println);
                  channel.basicConsume("message.debug",true,(tag,deliver)->{
                      System.out.println("message routing key: "+deliver.getEnvelope().getRoutingKey());
                      System.out.println("message body: "+ new String(deliver.getBody()));
                  },System.out::println);
                  channel.basicConsume("message",true,(tag,deliver)->{
                      System.out.println("message routing key: "+deliver.getEnvelope().getRoutingKey());
                      System.out.println("message body: "+ new String(deliver.getBody()));
                  },System.out::println);
              }
          }
          
          ```
        
        - 生产者
        
          ```java
          public class Provider {
              public static void main(String[] args) throws IOException, TimeoutException {
                  Channel channel = RabbitMqUtil.getChannel();
                  for (int i = 0; i < 100; i++) {
                      String msg="message"+i;
                      String routKey="message."+(i%2+1);
                      channel.basicPublish(EXCHANGE_NAME,routKey,MessageProperties.PERSISTENT_BASIC,msg.getBytes(StandardCharsets.UTF_8));
                  }
          
              }
          }
          ```
    
  - ### 死信队列
  
    - 当消息无法被正常消费时将消息存入一个死信队列
    - 情况一：消息 TTL 过期
    
    

​			
