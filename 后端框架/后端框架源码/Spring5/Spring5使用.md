# Spring5使用

- ## EventListener注解使用

  - ### 基本原理

    使用观察者模式，一个事件发布者以及多个事件监听者，事件发布者发布消息，对应的事件监听者收到消息后执行对应的程序

  - ### EventListener注解

    ```java
    @Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface EventListener {
    
    	/**
    	 * Alias for {@link #classes}.
    	 */
    	@AliasFor("classes")
    	Class<?>[] value() default {};
    
    	/**
    	 * 此侦听器处理的事件类。
    如果使用单个值指定此属性，则带注释的方法可以选择接受单个参数。但是，如果此属性指定了多个值，则注释方法不得声明任何参数。
    	 */
    	@AliasFor("value")
    	Class<?>[] classes() default {};
    
    	/**
    	 * Spring 表达式语言 (SpEL) 表达式用于使事件处理有条件。
    如果表达式的计算结果为布尔值true或以下字符串之一，则将处理该事件： "true" 、 "on" 、 "yes"或"1" 。
    默认表达式为"" ，表示始终处理事件。
    SpEL 表达式将针对提供以下元数据的专用上下文进行评估：
    #root.event或event用于引用ApplicationEvent
    #root.args或args用于对方法参数数组的引用
    方法参数可以通过索引访问。例如，第一个参数可以通过#root.args[0] 、 args[0] 、 #a0或#p0访问。
    如果参数名称在编译的字节码中可用，则可以通过名称（带有前面的哈希标记）访问方法参数。
    	 */
    	String condition() default "";
    
    	/**
    侦听器的可选标识符，默认为声明方法的完全限定签名（例如“mypackage.MyClass.myMethod()”）。
    	 */
    	String id() default "";
    
    }
    ```

    

  -  ### 使用

    - 创建事件实体类（也可以直接使用Spring自带的，也就是ApplicationEvent类）

      ```java
      public class StringMessage extends ApplicationEvent {
      
          public StringMessage(Object source) {
              super(source);
          }
      
          public StringMessage(Object source, Clock clock) {
              super(source, clock);
          }
      }
      
      //第二种事件
      public class UserMessage extends ApplicationEvent {
          public UserMessage(Object source) {
              super(source);
          }
      
          public UserMessage(Object source, Clock clock) {
              super(source, clock);
          }
      }
      
      ```

    - 创建事件监听器

      ```java
      @Service
      @Slf4j
      public class MyEventListener {
      
          @EventListener(StringMessage.class)
          public void eventListener01(StringMessage message)
          {
              log.info("EventListener01 GET Message:"+message.getSource().toString());
          }
      
      
          @EventListener(UserMessage.class)
          public void eventListener02(StringMessage message)
          {
              log.info("EventListener02 GET User Information:"+message.getSource().toString());
          }
      }
      
      ```

    - 发送消息

      ```java
      
      
      public class Test01 {
      
      
          public static void main(String[] args) {
              AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
              StringMessage msg1 = new StringMessage("this is a message");
              User user = new User();
              user.setName("zhangsan");
              user.setEmail("123");
              StringMessage msg2 = new StringMessage(user);
              context.publishEvent(msg1);
              context.publishEvent(msg2);
          }
      }
      //结果
      //2022-10-06 16:41:01 INFO [Test01.MyEventListener.MyEventListener] EventListener01 GET Message:this is a message
      //2022-10-06 16:41:01 INFO [Test01.MyEventListener.MyEventListener] EventListener01 GET Message:User(name=zhangsan, email=123)
      ```

      

