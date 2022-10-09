# Mybatis plus

- mybatis的增强工具

- 使用步骤

  - 通过阿里巴巴建立springboot工程，选择mybatis-plus，lombok，dev，mysql驱动

  - 配置yml文件

    ```yaml
    # DataSource Config
    spring:
      datasource:
        Druid:
          driver-class-name: com.mysql.cj.jdbc.Driver
          url: jdbc:mysql://localhost:3306/user?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&autoReconnect=true&useSSL=false
          username: root
          password: 123
          
    #开启日志功能查看编写的SQL语句      
    mybatis-plus:
      configuration:
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    
    ```

  - 编写pojo

    ```java
    @Data
    public class User {
        private Long id;
        private String name;
        private Integer age;
        private String email;
    }
    ```

  - 编写mapper类

    ```java
    public interface UserMapper extends BaseMapper<User> {
    
    }
    ```

    - BaseMapper是mp所提供的类，其中实现了许多SQLmapper的方法（从而不需要编写mapper文件和Dao方法）

  - 编写springboot主配置类

    ```java
    @SpringBootApplication
    @MapperScan("com.plus.mybatis_plus.DAO")
    public class MybatisPlusApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(MybatisPlusApplication.class, args);
        }
    
    }
    ```

  - 通过测试类测试

    ```java
    @SpringBootTest
    class MybatisPlusApplicationTests {
    
        @Resource
        UserMapper mapper;
    
        @Test
        void contextLoads() {
            List<User> users = mapper.selectList(null);
            users.forEach(System.out::println);
        }
    
    }
    ```

    - selectList是通过条件查找，如果无需条件直接使用null即可

  - 大致原理

    - plus通过反射机制查找到pojo类对应的表根据表生成对应的sql语句

  - BaseMapper方法解析

    - 增加一条记录

    ```java
    int insert(T entity);
    ```

    - 根据id删除

    ```java
        /**
         * 根据 ID 删除
         *
         * @param id 主键ID
         */
        int deleteById(Serializable id);
    ```

    - Serializable id表示可以序列化的对象都可以传值

    - 通过map删除

      ```java
       @Test
          void contextLoads3() {
              HashMap<String, Object> map = new HashMap<String, Object>();
              map.put("name","Tom");
              map.put("id",3);
              System.out.println(mapper.deleteByMap(map));
          }
      //sql语句 DELETE FROM user WHERE name = ? AND id = ?
      ```

    - 通过id批量删除
    
      ```java
      /**
       * 删除（根据ID 批量删除）
       *
       * @param idList 主键ID列表(不能为 null 以及 empty)
       */
      int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
      ```
    
    - 通过对象修改
    
      ```java
          /**
           * 根据 ID 修改
           * 只考虑id不考虑其他值
           * @param entity 实体对象
           */
          int updateById(@Param(Constants.ENTITY) T entity);
      ```
    
    - 查询语句
    
      ```java
      /**
       * 根据 ID 查询
       *
       * @param id 主键ID
       */
      T selectById(Serializable id);
      
      /**
       * 查询（根据ID 批量查询）
       *
       * @param idList 主键ID列表(不能为 null 以及 empty)
       */
      List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
      
      /**
       * 查询（根据 columnMap 条件）
       *
       * @param columnMap 表字段 map 对象
       使用and连接
       */
      List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
      ```
    
    - 如果想要自定义sql语句只需要按照mybatis的写法就可以了
    
  - IService接口
  
    - 通过IService接口实现了Service类方法模板
  
    - 使用方法
  
      - Remove开头的方法是删除
  
      - update开头是修改
  
      - get和list开头hi查询
  
      - save是insert，save方法中都不提供id，但是
  
        ```java
            /**
             * 批量修改插入
             *
             * @param entityList 实体对象集合
             * @param batchSize  每次的数量
             */
            boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
        ```
  
        - 在该方法中如果记录存在则进行修改如果不存在则添加，如何判断是否有记录，如果Collection<T> entityList中有id则是修改，如果没有id则是添加
  
    - 使用步骤
  
      - 编写MyService接口
  
        ```java
        public interface MyService extends IService<User> {
            public List<User> getAllUser();
        }
        
        ```
  
      - 编写MyService实现类
  
        ```java
        @Service
        public class MyServiceImpl extends ServiceImpl<UserMapper,User> implements MyService {
        
            @Resource
            UserMapper userMapper;
        
        
            @Override
            public List<User> getAllUser() {
                return userMapper.selectBlog();
            }
        }
        ```
  
      - 使用测试类
  
        ```java
        @SpringBootTest
        class MybatisPlusApplicationTests {
        
            @Resource
            MyServiceImpl service;
            @Test
            void userService() {
                service.getAllUser().forEach(System.out::println);
            }
        
        }
        
        ```
  
      - 使用批量添加方法
  
        ```java
         @Test
            void addBatch()
            {
                ArrayList<User> list = new ArrayList<User>();
                for (int i = 0; i < 10; i++) {
                    User user = new User();
                    user.setName("zhangsan"+i);
                    user.setAge(12+i);
                    list.add(user);
                }
        
                System.out.println(service.saveBatch(list));
            }
        ```
  
        - 如果user实体类中不提供id则save方法为添加，id通过雪花算法赋值
  
      
  
  - ServiceImpl
  
    - 是IService接口的实现类，其中有两个泛型
  
      ```java
      <M extends BaseMapper<T>>
      ```
  
      M 是UserMapper，T是User
  
    - 不建议使用该类
  
  - Mybatis-plus提供的注解
  
    - @TableName（“表名”）
  
      - 在使用mybatis-plus中并没有指定表名原因是我们继承的BaseMapper<T>中T这个泛型的名称作为表名，而实际上User可能不是表名，可以在User类上添加TableName注解表示该类对应的表名是什么
  
      - 有时候比如表名为t_User，而类名为User，也可以通过配置文件实现一致性
  
        ```yaml
        mybatis-plus:
          configuration:
            log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
          global-config:
            db-config:
              table-prefix: t_
        ```
  
        - 这样每个BaseMapper<T>中T的名称都会添加上table-prefix指定的前缀后再去数据库中查找表名，这样做的好处是如果pojo的名称和表名有前缀关联的话，不需要一个一个添加TableName
  
    - @TableId
  
      - 使用
  
        ```java
        
        @Data
        public class User {
            @TableId("表中主键名称")
            private Long id;
            private String name;
            private Integer age;
            private String email;
        }
        ```
  
        - @TableId下的成员将对应表的主键
  
        - 该注解还有一个Type属性
  
          ```java
          IdType type() default IdType.NONE;
          ```
  
          - 以下是IdType类型的原码
  
            ```java
            public enum IdType {
                AUTO(0),
                NONE(1),
                INPUT(2),
                ASSIGN_ID(3),
                ASSIGN_UUID(4),
               
            
                private final int key;
            
                private IdType(int key) {
                    this.key = key;
                }
            
                public int getKey() {
                    return this.key;
                }
            }
            
            ```
  
          - AUTO代表使用的是mysql中的id自增，NONE和ASSIGN_ID代表使用雪花算法生成id，
  
          - 主键设置策略也可以通过配置文件全局指定
  
            ```yaml
            mybatis-plus:
              configuration:
                log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
              global-config:
                db-config:
                  table-prefix: t_
                  id-type: auto
            ```
  
          - @TableField
  
            - 如果pojo中成员名和字段名不一致可以在pojo的成员上标注TableField（“表中字段名”）
  
          - @TableLogic逻辑删除
  
            - 物理上不删除，只是逻辑上删除，具体做法是在表中添加一个字段作为是否删除的标志，如果已删除标志为1，如果未删除标志为0
  
            - 例
  
              ```java
              @Data
              public class User {
                  @TableId
                  private Long id;
                  private String name;
                  private Integer age;
                  private String email;
                  @TableLogic
                  private Integer is_been_deleted;
              }
              ```
  
            - 之后只要使用remove方法就可以逻辑删除，之后（通过ServiceImpl的方法）将查找不到该记录
  
          - Wrapper条件构造器
  
            - ![image-20220504193118583](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220504193118583.png)
  
            - 具体使用方法
  
              - 选择合适的wrapper类并创建
  
                ```java
                @Test
                void selectlike()
                {
                    QueryWrapper<User> wrapper = new QueryWrapper<>();
                    wrapper.between("age",0,22)
                            .isNotNull("name")
                            .like("name","T");
                    System.out.println(service.getOne(wrapper));
                }
                ```
  
              - QueryWrapper的每个方法的返回值都是QueryWrapper所以可以连续使用“.”来连续添加条件，QueryWrapper条件也可以使用在删除语句中
  
            - 实现条件的嵌套
  
              - ```java
                default Children and(Consumer<Param> consumer) {
                        return and(true, consumer);
                    }
                ```
  
                - Consumer
  
                  ```java
                  void accept(T t);
                  ```
  
                - 综合上面两个可得
  
                  ```java
                  and(new Consumer(){
                      accept(Wrapper w)
                      {
                          //对Wrapper进行操作
                      }
                  })
                      
                      //如果在这里使用lambda表达式即为
                      and(i->i.like().isNotNull);
                  //i这里就是Wrapper，而最后这个语句实现的sql语句是
                  where ... and (...like...and...isNotNull)
                  ```
  
            - Wrapper的select方法
  
              - 如果在select中不需要查询出所有字段可以通过select方法指定要查询的字段
  
              - 使用实例
  
                ```java
                @Test
                    void delbywrapper()
                    {
                        QueryWrapper<User> wrapper = new QueryWrapper<>();
                        wrapper.select("name","age","email");
                        service.list(wrapper).forEach(System.out::println);
                    }
                ```
  
            - 嵌套查询
  
              ```java
              @Test
              void delbywrapper()
              {
                  QueryWrapper<User> wrapper = new QueryWrapper<>();
                  wrapper.select("name","age","email");
                  wrapper.inSql("name","select name from user");
                  service.list(wrapper).forEach(System.out::println);
              }
              ```
  
            - UpdateWrapper使用
  
              ```java
              @Test
                  void updatewrapper()
                  {
                      UpdateWrapper<User> wrapper = new UpdateWrapper<>();
                      wrapper.between("age",2,20);
                      wrapper.set("name","zhangsan");
                      service.update(wrapper);
                  }
              ```
  
            - 在实际开发中需要通过用户的选择动态装填wrapper的条件
  
              - 在每个wrapper中的方法都存在一个版本，该版本中带有一个boolean类型的condition参数,比如以下
  
                ```java
                    /**
                     * BETWEEN 值1 AND 值2
                     *
                     * @param condition 执行条件
                     * @param column    字段
                     * @param val1      值1
                     * @param val2      值2
                     * @return children
                     */
                    Children between(boolean condition, R column, Object val1, Object val2);
                ```
  
              - 这样就可动态装配wrapper
  
            - LambdaQueryWrapper
  
              - 与QueryWrapper不同的是，原先String 类型的column的参数被Function接口取代，如果此时需要限制age的范围可以这样写，这样写的目的是防止输入错的column
  
                ```java
                @Test
                    void lambda()
                    {
                        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>();
                        wrapper.between(User::getAge,10,19);
                       service.list(wrapper).forEach(System.out::println);
                    }
                ```
  
              - LambdaUpdateWrapper使用方法相同
  
  - 分页插件
  
    - 使用步骤
  
      - 创建mybatis-plus配置类
  
      ```java
      @Configuration
      @MapperScan("com.plus.mybatis_plus.DAO")
      public class mybatisConfig {
      
          @Bean
          public MybatisPlusInterceptor mybatisPlusInterceptor()
          {
              MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
              interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
              return interceptor;
          }
      }
      
      ```
  
      - MapperScan包扫描注解可以放这里来了
  
      - MybatisPlusInterceptor是mp的拦截器，PaginationInnerInterceptor是分页拦截器
  
      - DbType.MYSQL指的是数据库类型
  
      - 测试分页插件
  
        ```java
        @Test
            void testpage()
            {
                Page<User> page = new Page<>(2,2);
                Page<User> userPage = userMapper.selectPage(page, null);
                System.out.println("Current:"+userPage.getCurrent());
                userPage.getRecords().forEach(System.out::println);
                System.out.println("total:"+userPage.getTotal());
                System.out.println("是否有下一页:"+userPage.hasNext());
                System.out.println("是否有上一页:"+userPage.hasPrevious());
                System.out.println("getMaxLimit:"+userPage.getMaxLimit());
                System.out.println("getSize:"+userPage.getSize());
        
                /*Current:2
                User(id=1521783637165596673, name=zhangsan, age=12, email=null, is_been_deleted=null)
                User(id=1521783637241094145, name=zhangsan, age=13, email=null, is_been_deleted=null)
                total:12 总共多少条记录（不分页仍然保持逻辑删除）
                是否有下一页:true
                是否有上一页:true
                getMaxLimit:null
                getSize:2 页面大小
                */
            }
        ```
  
      - 使用类型别名
  
        ```yaml
        mybatis-plus:
          configuration:
            log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
          global-config:
            db-config:
              id-type: auto
          type-aliases-package: com.plus.mybatis_plus.pojo.User
        ```
  
      - 通过xml文件自定义分页SQL
  
        - 编写Mapper接口
  
        ```java
        public Page<User> selectbyPage(@Param("page") Page<User> page);
        ```
  
        - 编写XML文件
  
          ```xml
          <!--public Page<User> selectbyPage(@Param("page") Page<User> page);-->
              <select id="selectbyPage" resultType="User">
                  select *from user
              </select>
          ```
  
          - 这里resultType="User"同样是代表 Page<User>中的User，Page<User> page这个参数不需要管，mp会处理
  
        - 编写测试类
  
          ```java
          @Test
          void testmypage()
          {
              Page<User> userPage = new Page<>(2, 2);
              Page<User> page = userMapper.selectbyPage(userPage);
              page.getRecords().forEach(System.out::println);
          }
          ```
      
    - 乐观锁插件
    
      - 为数据库表中添加Version字段用于作为乐观锁版本号
    
      - 为User类添加Version成员并在Version上添加注解@Version
    
        ```java
        @Data
        public class User {
            @TableId
            private Long id;
            private String name;
            private Integer age;
            private String email;
            @TableLogic
            private Integer is_been_deleted;
            @Version
            private Integer version;
        }
        ```
    
      - 在mp主配置类中添加拦截器
    
        ```java
        
        @Configuration
        @MapperScan("com.plus.mybatis_plus.DAO")
        public class mybatisConfig {
        
            @Bean
            public MybatisPlusInterceptor mybatisPlusInterceptor()
            {
                MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
                interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
                interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
                return interceptor;
            }
        }
        ```
    
      - 要使用mp中的乐观锁必须先获取对象后再修改
    
    - 枚举类
    
      - 在数据库中添加一个字段为sex（int），创建一个枚举类Sex
    
        ```java
        @Getter
        public enum Sex {
            Male(1,"男"),
            Female(0,"女");
        
            @EnumValue
            private int s;
            private String sexName;
        
            Sex(int s, String sexName) {
                this.s = s;
                this.sexName = sexName;
            }
        }
        ```
    
        - EnumValue标注只是需要注入到数据库中的值
    
      - 测试
    
        ```java
        @Test
            void testsex()
            {
                User user = new User();
                user.setName("kk");
                user.setAge(78);
                user.setSex(Sex.Male);
                System.out.println(userMapper.insert(user));
            }
        
        ```
    
    - 代码生成器
      - 在官网上可以找到代码生成器代码，执行就可以得到一个mp的项目（无实体内容）

