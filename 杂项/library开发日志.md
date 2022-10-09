# library开发日志

- Day01(基本数据库设计和项目部署)
  - spring.profiles.group在springboot2.4版本后才支持(换成2.4.13版本就可以用了)
  - 更换Springboot版本时出现Maven无法下载相关依赖的问题，原因是本地仓库配置错误
    - 解决方法https://blog.csdn.net/weixin_44466651/article/details/107874452
  - Error creating bean with name 'sqlSessionFactory'错误，原因是表中主键是联合主键，使用Mybatis-plusX插件生成代码时会自动给两个字段添加@TableId，删除这些@TableId即可

- Day02（登录系统设计）

  - 对私钥进行base64加密时报错，原因是私钥的长度不是4的倍数

  - 使用FastJson将Json转为User对象时报错，原因是直接使用以下代码

    ```java
    User parse = (User) JSON.parse(s); //错误代码
    User user = JSONObject.parseObject(s, User.class); //正确代码
    ```
    
  - 修改数据库User表中主键id为非自增长类型时报错
  
    ```java
     Cannot change column 'u_id': used in a foreign key constraint 'book_user_fk' of table 'library.book'.
    ```
  
    - 原因是存在外键依赖无法修改
  
- Day03（登录系统开发）

  - 将User表的uid类型修改为char时先删除了有关外键，修改完成后无法添加外键且持续报错

    ```sql
    Create table 'library/#sql-13e8_35' with foreign key constraint failed. There is no index in the referenced table where the referenced columns appear as the first columns.
    
    Cannot add foreign key constraint
    ```

    - 原因在于两张表中的字符集编码不同，通过以下命令修改表的字符集编码,外键要求两个相连的字段的字符集和排序必须相同，这里采用的排序规则为utf8_general_ci  (utf8_bin不行)

      ```sql
      alter table `tablename` convert to character set utf8;
      ```
      
      - 一定要执行以下语句，不能直接从Dategrip中修改

- Day???

  - 请求报错

    ```java
    No primary or single unique constructor found for interface java.util.List
    ```

    - 原因没加@RequestBody
    
  - 获取请求体时使用fastjson将对象数组（list）的json直接转换为对象数组报错
  
    ```java
    List<BookOfCate> list = JSONObject.parseObject(body, List.class);
    ```
  
    - 解决
  
      ```java
      String json = "[" +
                      "{\"userId\":\"1\",\"username\":\"上海带刀沪卫\",\"password\":\"带刀大佬\"}" +
                      ",{\"userId\":\"1\",\"username\":\"上海辟谣专属队\",\"password\":\"职业辟谣，不信谣，不传谣，呵呵\"}" +
                      "]";
              ObjectMapper objectMapper = new ObjectMapper();
              try {
                  TypeReference<List<User>> typeReference = new TypeReference<List<User>>() {
                  };
                  List<User> list = objectMapper.readValue(json, typeReference);
                  list.forEach(user -> {
                      System.out.println(user.getUsername());
                  });
      
              } catch (JsonProcessingException e) {
                  log.error("error: ", e);
              }
      ————————————————
      版权声明：本文为CSDN博主「Hatakefiftyfifty」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
      原文链接：https://blog.csdn.net/Hatakefiftyfifty/article/details/124596654
      ```
  
  - 使用拦截器时想要在拦截器中注入service对象结果service对象为空
  
    - 拦截器配置
  
      ```java
      @Configuration
      public class WebInterceptor implements WebMvcConfigurer {
          /**
           * 添加Web项目的拦截器
           */
      
          @Override
          public void addInterceptors(InterceptorRegistry registry) {
              // 对所有访问路径，都通过MyInterceptor类型的拦截器进行拦截
              registry.addInterceptor(new CategoryIntercepter()).addPathPatterns("/cate/**")
                      .excludePathPatterns("/static/**","/cate");
              //放行登录页，登陆操作，静态资源
          }
      }
      ```
  
      - 解决
  
        ```java
        @Configuration
        public class WebInterceptor implements WebMvcConfigurer {
            /**
             * 添加Web项目的拦截器
             */
        
            @Autowired
            CategoryIntercepter categoryIntercepter;
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                // 对所有访问路径，都通过MyInterceptor类型的拦截器进行拦截
                registry.addInterceptor(categoryIntercepter).addPathPatterns("/cate/**")
                        .excludePathPatterns("/static/**","/cate");
                //放行登录页，登陆操作，静态资源
            }
        }
        
        ```
  
        - 采用注入的形式将拦截器注入至容器中而不是new
  
  - 使用拦截器对请求拦截后使用request.getReader()对请求体进行读取，后进入controller方法中时报错
  
    ```java
    getReader() has already been called for this request
    ```
  
    - 原因时controller中需要使用getReader方法而getReader方法只能调用一次，解决方法是采用aop代替拦截器，参考https://blog.csdn.net/noseew/article/details/79179892
    
    - 添加AOP
    
      ```java
      
      @Component
      @Aspect
      public class CategoryAOP {
      
      
          @Autowired
          BookService bookService;
      
          @Before("execution(* com.boot.library.Controller.Category.BookOfCategoryController.addCateToBook(..))")
          void checkUserOfBook(JoinPoint joinPoint)
          {
              Object[] args = joinPoint.getArgs();
              List<BookOfCate> arg = (List) args[0];
              Integer bId = arg.get(0).getBId();
              LoginUser user = (LoginUser) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
              String uId = user.getUser().getUId();
              Wrapper<Book> wrapper=new QueryWrapper<Book>().eq("b_id",bId).eq("u_id",uId);
              if (bookService.getOne(wrapper)==null) {
                  throw new CategoryException();
              }
          }
      
          @AfterReturning("execution(* com.boot.library.Controller.Category.BookOfCategoryController.addCateToBook(..))")
          public ResponseEntity<String> afterReturning()
          {
              return new ResponseUtil<String>().addMessage("add category to book fail", HttpStatus.BAD_REQUEST,null);
          }
      
      }
      
      ```
    
    - 但是AfterReturning并没有收集到before抛出的异常原因是SpringMVC将请求过程中的所有异常统一使用一个异常处理器处理了，这会导致请求失败但是HTTP状态码仍然是200，所以添加一个自定义异常处理器以此代替默认的SpringMVC异常处理器
    
      ```java
      
      @ControllerAdvice//用于标注该类是一个异常处理器
      public class CategoryExceptionHandle {
      
          @ExceptionHandler(CategoryException.class)//拦截指定异常
          ResponseEntity<String> handleCategoryException(CategoryException e)
          {
              e.printStackTrace();
              return new ResponseUtil<String>().addMessage("Category change fail", HttpStatus.BAD_REQUEST,null);
          }
      }
      ```
    
  
- 使用邮箱发送附件时报错

  - 解决方法将spring.mail.protocol设置为smtps

  - 后续报错

    ````java
    Exception writing Multipart
    ````

    - 原因是未设置mail body

      ```java
      //设置body
      messageHelper.setText("this is you download book:"+file.getName(),true);
      ```

      
