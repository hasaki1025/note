# Redis数据库

- **Nosql（Not only SQL）数据库**
  - 作用：session在分布式服务器中，如果只有台服务器有用户的session，那么对于登录等功能会有极大的限制，而Nosql数据库是一种缓存数据库，将数据放在cache中
  - 特点：非关系数据库，以键值对的方式存储数据，不遵循SQL标准，不支持ACID（**ACID**，是指[数据库管理系统](https://baike.baidu.com/item/数据库管理系统)（[DBMS](https://baike.baidu.com/item/DBMS)）在写入或更新资料的过程中，为保证[事务](https://baike.baidu.com/item/事务)（transaction）是正确可靠的，所必须具备的四个特性：[原子性](https://baike.baidu.com/item/原子性)（atomicity，或称不可分割性）、[一致性](https://baike.baidu.com/item/一致性)（consistency）、[隔离性](https://baike.baidu.com/item/隔离性)（isolation，又称独立性）、[持久性](https://baike.baidu.com/item/持久性)），远超SQL性能
  - 适用场景：对数据高并发读写，海量数据读写，对数据高可扩展性
  - 不适用场景：需要ACID支持，处理复杂的关系，

- linux上部署Redis

  - 官方上下载压缩包并解压（在虚拟机上）

  - 在解压后的目录中执行编译（要求系统中含有gcc）

    ```shell
    make
    ```

  - 编译后执行安装

    ```shell
    make install
    ```

  - 启动

    - 前台启动（不推荐）

      ```shell
      redis-server
      ```

      - 一旦关闭命令窗口后就会结束服务，该方式停止redis服务直接使用ctrl+c

    - 后台启动

      - 在redis目录下执行

        ```shell
        vim redis.conf
        ```

      - 更改配置文件将daemorize 从no改为yes

      - 启动redis服务

        ```shell
        [root@myvm redis-6.2.6]# redis-server redis.conf#redis.conf是redis.conf的路径
        ```

      - 可以查看到redis的服务

        ```shell
        
        [root@myvm redis-6.2.6]# ps -ef |grep redis
        root       9852      1  0 17:38 ?        00:00:02 src/redis-server *:6379
        root      66599  66518  0 18:17 pts/1    00:00:00 grep --color=auto redis
        ```

        - 6379为端口号

      - 访问客户端

        ```shell
        [root@myvm redis-6.2.6]# redis-cli
        127.0.0.1:6379> 
        ```

      - 退出

        ```shell
        127.0.0.1:6379> shutdown#单实例关闭
        not connected> redis-cli -p 6379 shutdown#多实例关闭
        Could not connect to Redis at 127.0.0.1:6379: Connection refused
        not connected> #在这里按ctrl+c退出
        ```

- ##Redis特点和常用指令

  - Redis默认16个数据库，初始默认使用0号数据库，可以通过select语句选择数据库

    ```shell
    select 1
    ```

  - 单线程+多路IO复用（多路复用可以实现同时面向多个用户，复用和计算机网络中的复用技术类似），而memcache使用的是多线程+锁，效率比redis差

  - 查看当前库中所有key

    ``` shell
    keys * #查看当前库
    keys *2#查看2号数据库中的所有key
    ```

  - 在当前库中添加键值对

    ```shell
    set keyname value
    ```

    - 如果库中存在该key则会覆盖该key所应的value

  - 仅在key不存在时设置key的值

    ```shell
    setnx keyname value
    ```

    - 成功添加返回1，不成功返回0

    - 同时设置多个键值对

      ```shell
      mset keyname1 value1 keyname2 value2 keyname3 value3....
      msetnx keyname1 value1 keyname2 value2 keyname3 value3....
      ```

      - 该操作具有原子性,要么都成功要么都失败

  - 在指定键值对的value后面添加

    ```shell
    append keyname appendvalue
    ```

    - 返回追加后的字符串的长度
    - 如果该key不存在则会创建并且value为追加的内容，返回的长度也是追加的value的长度

  - 使key(要求value是Integer类型)的数字值增加1

    ```shell
    incr keyname
    ```

    - 如果不是integer类型则会报错

    - 也可以使增加任意的数字

      ```shell\
      incrby keyname 10#增加10
      ```

    - incr具有原子性（不会被其他线程打断）

  - 使key(要求value是Integer类型)的数字值减少1

    ```shell
    decr keyname
    ```

    - 使用和incr相同

    - decr具有原子性（不会被其他线程打断）

    - 也可以减少任意数字

      ```shell
      decrby keyname 10#减少10 
      ```

  - 获取key长度

    ```shell
    strlen keyname
    ```

  - 判断key是否存在

    ```shell
    exists keyname
    ```

    - 返回（Integer）1表示含有该键，返回（Integer）0表示不含有

  - 查看key的类型

    ```shell
    type keyname
    ```

    - 如果该key不存在则返回none

  - 查看key对应的value

    ```shell
    get keyname
    ```

    - 不存在返回nil

    - 同时获取多个value

      ```shell
      get key1 key2 key3
      ```

  - 删除key

    ```shell
    del keyname
    ```

    - 存在则删除并返回（Integer）1，否则返回（Integer）0

  - 根据value选择非堵塞删除

    ```shell
    unlink keyname
    ```

    - 仅将keys从keyspace中删除，不会立刻在数据库中删除，真正的删除在后续的异步操作

  - 设置key的过期时间

    ```shell
    expire keyname 10#表示10秒后删除
    ```

  - 查看key过期时间

    ```shell
    ttl keyname
    ```

    - 返回负数代表已过期

  - 查看当前数据库中有多少个key

    ```shell
    dbsize
    ```

  - 清空当前库

    ```shell
    flushdb
    ```

  - 通杀全部库

    ```shell
    flushall
    ```

- ##Redis数据类型

  - string

    - string是最基本的数据类型，string是二进制安全的，可以包含任何数据如jpg图片或序列化对象，string的最大容量是521MB

    - 底层

      - 动态字符串，可以理解为ArrayList<Character>,会自动扩容（依次扩容1Mb）

    - 获取string类型的value的部分字符串

      ```shell
      getrange keyname 2 3#2表示起始下标，3表示终止下标
      ```

      - 与java中的substring类似，但是是闭区间
      - 如果终止下标大于（字符串长度-1）则以（字符串长度-1）作为终止点，如果出现负数则

    - 设置指定范围的字符串值

      ```shell
      setrange keyname 2 value
      ```

      - 从2开始用value替代

    - 设置键值对的同时设置过期时间

      ```shell
      setex keyname 3 value
      ```

      - 3代表过期时间

    - 获取旧值的同时设置新值

      ```shell
      getset keyname value
      ```

  - 列表List
  
    - 单键多值，底层是一个双向链表
  
    - 底层
  
      - 在元素较少时为ziplist，ziplist采用的是内存上相邻，类似于数组，当元素增加时会将多个ziplist连接成一个quicklist
  
    - 常用指令
  
      - lpush/rpush 从左/右添加值
  
        ```shell
        lpush kt 1 2 3 4
        rpush kt 4 5 6 7
        ```
  
        - 如果没有该键会创建
  
      - lpop/rpop左/右吐出一个值
  
        ```shell
        lpop k1
        ```
  
        - 如果吐出了最后一个值则list会被销毁
  
      - rpoplpush 从一个列表中的右边吐出一个值并插入到另一个列表的左边
  
        ```shell
        rpoplpush k1 k2
        ```
  
        - 返回吐出的值
  
      - 按照索引下标取值
  
        ```shell
        lrange k1 0 2
        ```
  
        - 0表示起始下标，2表示终止下标，同样闭区间，0代表最左边第一个，-1代表最右边第一个
  
      - 根据索引获取单个值（从左到右）
  
        ```shell
        linex k1 2
        ```
  
      - 获取列表长度
  
        ```shell
        llen keyname
        ```
  
      - 在某个值前/后插入一个新值
  
        ```shell
        linsert k1 before 2 3
        linsert k1 after 2 3
        ```
  
        - 如果采用before且列表为 1， 2， 3,  4 ，5 ，则插入后为1 3 2 3 4 5
  
      - lrem从左删除n个指定value
  
        ```shell
        lrem k1 2 1
        ```
  
        - 2 表示删除的个数，1表示需要删除的值
  
      - lset将指定下标的值替换为新值
  
        ```shell
        lset k1 1 99
        ```
  
        - 1代表需要替换的下标，99代表新值
  
  - Set集合
  
    - 无序无重，底层为hash表
    - 常用指令
      - 添加sadd <key> <value1> <value2>
      - 取出集合中的所有元素 
        - smembers <key>
      - 判断是否具有某值
        - sismember <key> <value>
        - 有则返回1
      - 返回元素个数
        - scard <key>
      - 删除集合中的元素
        - srem <key> <value1><value2> <value3>
      - 随机从集合中吐出n个值
        - spop <key><count>
        - 值在集合在，值无集合无
      - 随机从集合中取出n个值但不删除
        - srandmember <key><count>
      - 把集合中的某个值移动到另一个集合
        - smove <key1><key2> <value>
        - 从key1移动到key2
      - 返回交集的元素
        - sinter<key1><key2>
      - 返回并集元素
        - sunion<key1><key2>
      - 返回差集元素(只包含key1不包含key2)
        - sdiff<key1><key2>
  
  - hash表
  
    - 键值对集合
    - String类型的field和value的键值对的集合（Map<String,Object>），所以hash表适合存储对象
    - 在表中数据较少时采用ziplist，数据多时使用hashtable
    - 常用指令
      - 添加集合
        - hset<key><field1><value1><field2><value2><field3><value3>
      - 取出指定集合中的value
        - hget<key> <field>
      - 批量设置hash值
        - hmset<key><field1><value1><field2><value2><field3><value3>
      - 判断hash表中是否存在某个field
        - hexists<key><field>
      - 列出hash所有key
        - hkeys<key>
      - 列出hash所有value
        - hvals<key>
      - 为hash表中的field的value的值增加指定量
        - hincrby<key><field><increment>
        - increment正数和负数都可以
      - 仅当hash表中field不存在时设置为value
        - hsetnx <key><field><value>
  
  - Zset有序集合
  
    - 无重有序，这里的有序和存放顺序无关，而是与Zset中每一个元素的一个属性score有关，Zset通过score为集合排序，底层类似于Map<String,double>其中value为String，score为double，除此以外有序集合通过跳跃表查找元素，跳跃表可以在有序集合中快速找到指定元素
  
    -  常用指令
  
      - 添加
  
        - zadd<key><score1><value1><score2><value2><score3><value3>
  
      - 获取score指定排名内的
  
        - zrange <key><start><stop> [WIRHSCORES]
  
          ```shell
          zrange key 0 3 withscore
          ```
  
          - 排序后第0位到第3位，可以在语句后添加withscores，查询得出的结果会附带score
  
      - 获取score范围内的value
  
        - zrangebyscore <key> <min> <max> [withscores] [limit offset count]
        - offset起始位置，count显示数量
  
      - 降序获取score范围内的value
  
        - zrevrangebyscore <key><min><max> [withscores] [limit offset count]
  
      - 为score添加增量
  
        - zincrby <key><increment><value>
  
      - 删除该集合下指定值的元素
  
        - zrem <key><value>
  
      - 统计分数区间段下的元素数量
  
        - zcount <key><min><max>
        - 同样闭区间
  
      - 返回指定value的排名
  
        - zrank <key><value>
        - 排名从0开始
  
- ##Redis配置文件

  - Redis单位

    - 只支持字节不支持bit且大小写不敏感

  - Redis network

    - bind=127.0.0.1
      - 只接受本机访问请求
      - 如果不添加这条则不会对访问限制
    - protected-mode yes
      - 开启本机访问模式保护=yes，远程无法访问
    - port 6379
      - 默认端口号为6379
    - tcp-backlog 511
      - tcp协议：三次握手五次挥手
      - backlog：连接队列，backlog=511表示队列总和为511，可以增加这个值避免客户端连接缓慢问题
    - timeout 0
      - 一段时间后如果不对redis操作则断开连接，单位为秒
      - 设置为0代表永不超时
    - tcp-keepalive 300
      - TCP连接超过这个时间不操作会自动断开连接，单位为秒

  - general通用

    - pidfile /var/run/redis_6379.pid

      - pidfile：每次操作redis将进程号记录在该文件中

    - loglevel notice

      - 日志级别

      ```properties
      # debug (a lot of information, useful for development/testing)看到最详细的信息
      # verbose (many rarely useful info, but not a mess like the debug level)看到有用#的信息
      # notice (moderately verbose, what you want in production probably)默认
      # warning (only very important / critical messages are logged)只显示有用的重要的信息
      ```

    - logfile ""

      - 设置日志文件输出

    - databases 16

      - 默认16个数据库

  - security

    - requirepass root

      - 设置密码为root，默认该行是注释掉的

      - 也可以通过命令行改密码

        ```shell
        config get requirepass#获取密码
        config set requirepass root#设置密码
        auth root#密码认证
        ```

  - limit限制

    - maxmemory-policy 
      - 对于缓存数据库而言内存是有限的，内存满时会通过算法将一部分数据移除数据库
      - 移除算法
        - volatile-lru：通过LRU（最近最少使用算法）移除key，只对设置了过期时间的键
        - allkeys-lru：对于所有key集合
        - volatile-random：随机移除已设置了过期时间的键
        - allkeys-random：所有key随机移除
        - volatile-ttl：移除TTL（TTL是 Time To Live的缩写，该字段指定IP包被路由器丢弃之前允许通过的最大网段数量。TTL是IPv4报头的一个8 bit字段）值最小的key和即将过期的key
        - noeviction：不移除
    - maxmemory-samples
      - 设置样本数量
    - maxclients 10000
      - 最大连接数，默认10000
    
  - SNAPSHOTTING（有关RDB）
  
    - dbfilename dump.rdb
  
      - RDB临时文件名
  
    - dir ./
  
      - RDB文件目录
  
    - stop-writes-on-bgsave-error yes
  
      - 当Redis无法写入磁盘，注解关闭Redis的写操作
  
    - rdbcompression yes
  
      - 对于存储到磁盘中的数据快照是否进行压缩存储，如果是采用LZF算法
  
    - rdbchecksum yes
  
      - 存储快照后，是redis使用CRC64算法进行数据校验
  
    -  save ""
  
      - 保存时堵塞其他线程，手动保存，一般不采用，一般使用bgsave（Redis后台异步进行快照操作，同时也可以响应客户端请求）
  
      ```conf
      # save 3600 1
      # save 300 100
      # save 60 10000
      ```
      
      - 表示在3600内如果有一个或大于一个key发生变化则写入，300秒以内如果有100及以上的key发生变化则写入，如果在60秒以内有10000及以上个key发生变化则写入。在配置文件中这些是一般是注释的，但可以手动配置
    
  - AOF有关
  
    - appendonly no
      - 开启aof，默认为不开启
    - appendfilename "appendonly.aof"
      - AOF有关日志文件，生成路径和RDB相同
    - appendfsync
      - always
        - 每次写入都会立刻记录日志，数据完整性好
      - everysec
        - 每秒同步，默认设置为该值
      - no 
        - 不主动同步，将同步时机交给操作系统
    
  - 哨兵有关
  
    - replica-priority 100
      - 设置该redis的优先级，旧版为slave-priority，值越小优先级越大
  
- Redis中的发布与订阅

  - ![订阅者和发布者](D:\java框架\订阅者和发布者.png)
  - 操作步骤
    - 打开一个redis客户端，subscribe channel1订阅频道
    - 打开另一个客户端，publish channel1 message
    - 此时第一个客户端可以得到发布的消息

- Redis6中的新数据类型

  - Bitmaps

    - 二进制字符串,优点是可以减少空间使用
    - 可以进行位操作，对于bitmaps字符串中的每一位都一个类似于数组下标的标记称为偏移量

    - 常用命令

      - 设置bitmaps中某位的值为0或1

        - setbit <bit><offset><value> offset为偏移量

      - 获取偏移量的值

        - getbit<key><offset>

      - 统计字符串被设置为1的bit数

        - bitcount <key><start offset><end offset>

      - bitmaps复合操作

        - 将key1、key2、key3等通过and/or/not/xor(and表示与，or表示或，not表示非)等操作后得出的bitmaps放置在destkey中

        - bitop and(or/not/xor) <destkey> <key1><key2><key3><key4><key5>

  - HypetLogLog

    - 在实际应用中假设需要记录某个视频的点击量，规定每个用户最多记录点击一次，那么既要实现去重又要实现统计可以采用mysql中的去重和redis中的incr，但是这样需要耗费大量的空间，而hypetloglog可以通过小内存完成统计
    - 常用命令
      - 添加
        - pfadd <key><value1><value2>
        - 支持批量添加，支持去重
      - 获取key中value的数量
        - pfcount <key>
      - 合并
        - pfmerge <result key><source key1><source key2>
        - 将key1和key2中的key合并到result中
        - 合并后原先的依旧存在

  - Geospatial

    - Redis3.2增加了对GEO类型的支持，GEO为地理信息的缩写，redis提供该类型的经纬度设置，查询，范围查询，距离查询，经纬度hash

    - 常用指令

      - geoadd <key><longitude 经度><latitude 纬度><member 位置名称>

        ```shell
        geoadd china:city 121.47 31.23 shanghai 106.50 29.53 chongqing
        ```

        - 支持批量添加
        - 有效的经度：-180~ 180，有效的纬度：-85.05112878~85.6112878

      - 获取经纬度

        - geopos <key><member>

      - 获取两地直线距离

        - geodist <key><member1><member2>[m|km|ft|mi]
        - [m|km|ft|mi]用于指定单位，默认单位为米，km千米，mi英里，ft英尺

      - 获取经纬度地点范围内的所有元素

        - georadius <key> <longitude><latitude ><radius>[m|km|ft|mi]
        - radius为半径，longitude和latitude 为目标地点的经纬度

- ## jedis

  - 类似于JDBC，通过java连接Redis

  - 使用步骤

    - 添加maven依赖

      ```xml
      <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
      <dependency>
          <groupId>redis.clients</groupId>
          <artifactId>jedis</artifactId>
          <version>4.2.1</version>
      </dependency>
      
      ```

    - 编写测试类创建jedis对象

      ```java
      //创建jedis对象,第一个参数是redis所在的主机ip，port是Redis端口号
      Jedis jedis = new Jedis("192.168.76.107",6379);
      ```

    - 使用ping方法测试连接

      ```java
      System.out.println(jedis.ping());
      ```

      - 如果输出pong则代表连接成功

    - 使用redis

      ```java
      
      jedis.set("k1","1213");
      String s = jedis.get("k1");
      System.out.println(s);
      ```

      - 大部分jedis中的方法名和redis中的相同

- ## Redis事务

  - Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化、按照顺序执行。事务在执行的过程中不会被其他客户端发送来的命令请求打断
  
  - 基础命令
    - Multi
      - 事务开始命令，将接下来的命令以队列方式存储并不会执行，接下来的阶段（直到执行或终止）称为组队阶段，可以理解为start transcation
    - Exec
      - 执行命令队列中的命令（执行阶段），可以理解为commit
    - discard
      - 当程序员不想执行可以通过discard终止组队，可以理解为rollback
    
  - 事务的错误处理
    - 如果组队阶段发生错误（语法错误）则会终止事务，所有操作都不会执行
    - 如果在执行阶段发生错误（运行时错误比如类型不符合）则发生错误的语句会不执行，而其他语句都会执行
    
  - 事务的冲突问题
    - 对于某个账户存在多个用户进行操作，如果不添加事务则会产生问题
    
    - 锁
      - 悲观锁
        - 一个用户要对临界区进行操作时先上锁，上锁后其他用户堵塞，直到持有锁的用户使用完邻接区资源后释放锁
        - 许多传统的关系型数据库都会采用这种锁，比如行锁，表锁，读锁，写锁等，但是这种锁影响效率
      - 乐观锁
        - 每个用户操作时对数据添加一个版本号，操作后会修改版本号，如果接下来的用户对该数据操作时会先检查该数据版本号是否符合自己当时拿到的数据版本号一致，如果不一致则不操作
      
    - 乐观锁在Redis中的使用
      - Watch <key1><key2>
        - 在使用multi之前先使用watch关键字为key上乐观锁，如果事务在执行前，key被其他的命令改动则事务会被打断
        
      - 在jedis中的使用
      
        ```java
        jedis.watch("ppp");
        Transaction multi = jedis.multi();
        multi.set("123","123");
        multi.set("iii","123");
        List<Object> exec = multi.exec();
        ```
      
      - 问题：库存遗留问题，对于乐观锁，在用户一操作并修改版本号后用户二再操作会发现版本号不一致从而终止事务。
      
        - 解决方法：lua脚本
          - 将复杂的redis操作写成一个lua脚本一次提交给redis执行
          - lua脚本类似于redis事务，具有一定的原子性，不会被其他指令插队
          - 
      
    - Redis事务三特性
      - 单独的隔离操作
        - 所有命令都会序列化、按顺序执行，执行过程中不会被其他客户端打断
      - 没有隔离级别
        - 所有事务在提交前都不会实际执行
      - 不保证原子性
        - 执行阶段如果一条指令出错而其他的执行依旧执行
        - 192.168.209.2
    
  - 秒杀案例
  
    - ab工具模拟多用户请求
  
      - 常用指令
  
        - 多次请求
  
          ```shell
          ab -n 1000 -c 100 http://ip:8080
          ```
  
          - -n：请求该url1000次
  
          - -c：并发操作，同时进行100次操作
  
          - -T：如果使用POST或put请求，需要在后面添加请求格式
  
            ```shell
            -T 'text/html; charset=utf-8'
            -T 'application/x-www-form-urlencoded'
            ```
  
          - -p: postfile，如果使用post提交需要将数据放置在文件中
  
            ```shell
            ~/postfile
            ```
  
    - 多用户请求时可能Redis连接数不够，可以采用连接池





- ##Redis持久化

  - ###RDB

    - RDB默认开启
  
    - 在指定时间间隔内将内存中的数据集快照写入磁盘
    - Redis会单独创建一个子进程fork来进行持久化，会先将数据写入到一个临时文件中，待持久化过程结束，再将该临时文件替换上次持久化的文件，整个过程中主进程不会进行任何IO操作,相比于AOF，RDB更加高效但是最后一个持久化操作的数据可能会丢失
    - 优势：节省空间，恢复速度快
    - 劣势：通过临时文件的方式会导致内存占用增加一倍，数据量庞大时比较消耗性能，对于最后一次的持久化操作可能会导致数据丢失

  - ###AOF
  
    - AOF默认不开启
    - 以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来，只追加文件但不改写文件，redis启动之初会读取该文件构建数据，也就是redis重启后会将写指令从前到后执行一次
    - 如果开启AOF则不采用RDB
    - AOF具有异常恢复机制，对于不符合appedonly.aof文件语法的内容会被修复
      - 如果appedonly.aof文件中有错误内容，可以使用指令 redis-check-aof
    - 重写操作
      - 当文件过大是会采用重写操作，redis4.0后的重写操作实际上是把rdb的快照以二进制的形式附在aof的头部上，替换之前的繁琐的指令操作
  
- ## Redis主从复制

  -  ![image-20220428151234729](C:\Users\房间内红茶香气依旧\AppData\Roaming\Typora\typora-user-images\image-20220428151234729.png)

  - 读写分离

    - 主服务器作为写，从服务器作为读

  - 容灾恢复

    - 假设有从服务器寄了，其他的从服务器也可以读

  - 集群

    - 假设主服务器寄，如果有多个上图的结构，并且多个主服务器之间有关联，则其他主服务器也可以提供服务

  - 配置方法

    - 创建一个文件夹用于存放主配置文件

    - 创建从redis文件夹并新建配置文件

      - 编写从redis配置文件(以6380为例)

        ```conf
        include /Redis/myredis_config/redis.conf#引入主配置文件
        pidfile /var/run/redis_6380.pid#配置进程
        port 6380#配置端口
        dbfilename dump6380.rdb#配置RDB文件
        masterauth $root1026  # 设置主redis密码，否则无法同步
        slaveof 127.0.0.1 6379 # 设置为从本机的6379 端口的redis同步数据，也可使用replicaof命令
        ```

    - 关闭从redis的aof

    - 开启三个Redis服务

      ```shell
      redis-server /Redis/redis-6379/redis6379.conf
      redis-server /Redis/redis-6380/redis6380.conf
      redis-server /Redis/redis-6381/redis6381.conf
      ```

      - info replication查看主从情况

        - 在redis客户端中查看

          - 进入客户端

            ```shell
            redis-cli -p 6379
            ```

          - 查看主从情况

            ```shell
            info replication
            ```

            ```shell
            # Replication
            role:master
            connected_slaves:0
            master_failover_state:no-failover
            master_replid:6440efa48d2c3c924f289bc8e9f18b35bc49dc04
            master_replid2:0000000000000000000000000000000000000000
            master_repl_offset:0
            second_repl_offset:-1
            repl_backlog_active:0
            repl_backlog_size:1048576
            repl_backlog_first_byte_offset:0
            repl_backlog_histlen:0
            ```

            - 可以看到每个客户端都显示该redis为主redis

      - 配置主从关系(如果不在配置文件中声明主从关系的话可以采用以下方式)

        - 在redis6380和redis6381客户端中输入slaveof <ip地址><端口号>,在这里ip只要填写127.0.0.1就可以了，（注意如果redis配置文件中设置了密码则需要在配置文件中添加主Redis的密码）
        - 再次查看role则为slave

    - 在主Redis中执行set tt tt，可以在从Redis中查看到，但是在从Redis中添加键值对是违法的

    - 存在的某些问题

      - 如果从服务器寄了，再次连接上后，数据照样共享

    - 主从复制原理

      - 当从服务器连接到主服务器时会发送同步请求（sync）
      - 全量复制：主服务器接收到同步请求时会将数据通过RDB持久化并发送给从服务器，从服务器读取
      - 增量复制：每次主服务器进行写操作后都会与从服务器数据同步

    - 薪火相传

      - a，b，c是三个服务器，a是b的主服务器，b是c的主服务器，那么同步时只需要写入a就可以对b和c两台服务器同步，也就是主从关系具有继承特性，但是这种方式有缺陷，如果b服务器寄了，那么c也无法完成同步
      - 这样我们可以认为主从关系可以理解为一种树状结构，每个服务器只能有一个主服务器，但是可以有多个从服务器
      - 每个服务器在启动时如果不指定主服务器，那么他自己就是主服务器，在配置文件中可以在启动时指定主服务器，在之后也可以通过指令修改，修改过后的主服务器会覆盖配置文件中的主服务器，类似于指针

    - 反客为主

      - 可以通过slaveof no one将服务器的主服务器设置为空，此时该服务器就是master
      - 还是和树状结构一样，slaveof no one可以清除自己所指向的主机

    - 哨兵模式

      - 当主服务器寄了时，从服务器会一直等待主服务器回归，我们可以通过slaveof no one使主机寄了的从机称为主机，但是 这样需要手动配置，所哨兵模式就是实现自动化反客为主的一种模式

      - 使用步骤

        - 在配置文件夹中创建空sentinel.conf并添加

          ```conf
          sentinel monitor mymaster 127.0.0.1 6379 1
          ```

          - sentinel monitor声明这是一个哨兵
          - mymaster是为主服务器起的名（可自定义）
          - 127.0.0.1 6379为主服务器的ip和port
          - 1：如果该哨兵发现主机寄了会投出一票

        - 启动哨兵

          ```shell
          redis-sentinel Redis/sentinel.conf
          ```

        - 启动哨兵后如果哨兵发现主机寄了则会选取一个从机作为主机，让原先主机下的其他从机以新主机作为注解，而如果原先的主机重新连接后同样作为新主机的从机

          - 选举规则
            - 优先级靠前的
              - 在redis配置文件中有replica-priority作为优先级，replica-priority越低优先级越高
            - 偏移量大的
              - 与主机交换次数最多的
            - runid最小的
              - redis启动时会随机分配40位的runid，也就是随机选择新主机





- ## Redis集群

  - 集群作用

    - 容量不够，多个Redis形成集群解决

    - 并发写操作，集群分摊主服务器写操作压力

    - 主从模式、哨兵模式导致主服务器ip变化，之前采用代理服务器解决，但在redis3.0之后采用了无中心化集群配置

      - 代理服务器大致图

        ![代理服务器](D:\java框架\代理服务器.png)\

      - 无中心化集群大致图

        ![无中心化集群](D:\java框架\无中心化集群.png)

    - 操作步骤

      - 建立6个新的redis.conf(按照主从模式创建)

        - 这里采用6379，6380,6381,6390,6391,6392端口
        - [如果后期发现一直卡在Waiting for the cluster to join，请点击这里解决](https://blog.csdn.net/qq_39354297/article/details/99916860?ops_request_misc=&request_id=&biz_id=102&utm_term=redis集群总线端口&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-3-.pc_search_similar&spm=1018.2226.3001.4187)
        - 注意：如果使用集群不成功可以先将所有之前的持久化文件和结点文件删除

      - 打开集群模式
      
        - 在配置文件中添加
      
        ```shell
        cluster-enabled yes#打开集群模式
        cluster-config-file nodes-6379.conf#设置结点配置文件名
        cluster-node-timeout 15000#设置节点失联时间，超过该时间（毫秒），集群自动进行主从切换
        ```
      
        - 配置文件中记得关闭aof
      
      - 启动6个服务
      
        ```shell
        redis-server Redis_config/redis6379.conf
        redis-server Redis_config/redis6380.conf
        redis-server Redis_config/redis6381.conf
        redis-server Redis_config/redis6389.conf
        redis-server Redis_config/redis6390.conf
        redis-server Redis_config/redis6391.conf
        ```

      - 查看Redis端口占用

        ```shell
        ps -ef | grep redis
        ```

        - 顺便查看结点配置文件是否生成（如odes-6379.conf）（127.0.0.1可以，其他的ip不行）
      
          ```shell
          redis-cli --cluster create --cluster-replicas 1 117.50.172.112:6379 117.50.172.112:6380 117.50.172.112:6381 117.50.172.112:6390 117.50.172.112:6391 117.50.172.112:6392
          
          
          
          
          redis-cli -a 密码 --cluster create --cluster-replicas 1 117.50.172.112:6379 117.50.172.112:6380 117.50.172.112:6381 117.50.172.112:6390 117.50.172.112:6391 117.50.172.112:6392
          
          redis-cli --cluster create --cluster-replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6389 127.0.0.1:6390 117.50.172.112:6391
          
          
          #如果有密码
          redis-cli -a 密码(可能要加双引号) --cluster create --cluster-replicas 1 117.50.172.112:6379 117.50.172.112:6380 117.50.172.112:6381 117.50.172.112:6390 117.50.172.112:6391 117.50.172.112:6392
          ```
          
          - --cluster-replicas 1 中的1代表以最简单的方式创建集群，而一个最简单的集群至少需要三个主结点
          - 原则上每台服务器的ip尽量不同
        
      - 集群方式连接主机
      
        - 对于无中心化集群而言连接到任何服务器的端口都可以连接到整个集群
      
          ```shell
          redis-cli -c -p 6379
          ```
      
      - 集群结点信息查看
      
        ```shell
        cluster nodes
        ```
      
    - Redis集群插槽slot
    
      - 一个Redis集群中包含了16384个插槽，数据库中的每一个键都属于16384个插槽中的一个
    
      - 在cluster nodes可以看到每个主节点都分配了等量的插槽
    
        - 例：当新建一个key 为t1时Redis会根据CRC16(KEY)%16384来计算该key对应的插槽，再判断该key对应的插槽所在的数据库并切换到该数据库，最后将数据存入插槽
    
          ```shell
          set k1 yy
          -> Redirected to slot [12706] located at 117.50.172.112:6381
          OK
          ```
    
        -  作用将值平均分配到每个主节点中
    
      - 查看插槽信息
    
        - cluster slots
    
    - Redis集群中的命令
    
      - 在集群中使用mset批量添加要采取另一种方式
    
        ```shell
        mset name{student} lucy age{student} 20
        ```
    
        - 与普通的redis不同的是mset <key>{自定义组名} <value> <key2>{自定义组名} <value2>
    
      - 查找key对应插槽
    
        - cluster keyslot <keyname>
    
      - 查看该插槽内有几个值
    
        - cluster countkeysinslot <slotnumber>
        - 注意：在哪个Redis服务器就只能查看当前服务器所有拥有的插槽
    
      - 查询插槽中的值（可以指定值的数量）
    
        - cluster getkeysinslot <slot><count>
        - slot表示插槽，count表示显示的值的数量
    
    - Redis故障恢复
    
      - 当某个主节点死了后，从结点替代主节点(原先6392为slave)
    
        ```shell
        13fb9c1fa311c9457be7f16c0e13945d6b1c6008 127.0.0.1:6391@16391 slave 49a94d753f47acee4d6a1502dbdcb2366642113b 0 1651209254104 3 connected
        c6222e4694004a76d6cd983b033621550886df5d 127.0.0.1:6392@16392 myself,master - 0 1651209253000 7 connected 0-5460
        4a7cc66ec1d250c5a4184c1ee6f8fe7724468b60 127.0.0.1:6379@16379 master,fail - 1651209042749 1651209038686 1 disconnected
        1ea6d2614beed931c2c200fd9cf0dcb927659012 127.0.0.1:6390@16390 slave 97ac4d2e5c8ce3bd4ac83587f1cfe08fed9648dc 0 1651209252089 2 connected
        97ac4d2e5c8ce3bd4ac83587f1cfe08fed9648dc 127.0.0.1:6380@16380 master - 0 1651209253096 2 connected 5461-10922
        49a94d753f47acee4d6a1502dbdcb2366642113b 127.0.0.1:6381@16381 master - 0 1651209253000 3 connected 10923-16383
        ```
    
        - 如果6379之后连接便会称为6392的从机
    
      - 当一个结点的主机和从机全部宕机时，根据配置文件有两种可能
    
        - cluster-require-full-coverage
          - 为yes时集群失效，默认为yes
          - 为no时仅宕机的主节点所管理的插槽不能使用
    
  - jedis操作集群
  
    ```java
    
    public class demo {
        public static void main(String[] args) {
            String host="117.50.172.112";
            HostAndPort hostAndPort = new HostAndPort(host, 6379);
            JedisCluster cluster = new JedisCluster(hostAndPort);
            cluster.set("rr","www");
            System.out.println(cluster.get("rr"));
            cluster.close();
        }
    }
    ```
  
  - ##Redis应用问题
  
    - ###缓存穿透
  
      - 现象
        - 应用服务器压力变大
        - Redis命中率降低，不断查询数据库，数据库压力急剧增加导致数据库崩溃
      - 原因
        - redis查询不到数据库
        - 出现大量非正常url访问
        - 大部分原因是黑客攻击
      - 解决方案
        - 对空值缓存：如果一个查询返回数据为空，仍然将结果缓存在Redis中，并设置较短的过期时间，一般不超过5分钟
        - 设置可访问白名单：使用bitmaps定义一个可以访问的目的，访问者的id作为偏移量，每次访问和bitmap里的id比较，如果id不在bitmaps中则进行拦截不允许访问
        - 采用布隆过滤器：底层和bitmaps相同但效率更高，实际由一串很长的二进制向量和随机映射函数（哈希函数）组成
        - 实时监控：当发现Redis命中率开始急剧降低，需要排查访问对象和访问数据，和运维人员配合设置黑名单限制服务
  
    - ## 缓冲击穿
  
      - 现象
        - 数据库访问压力瞬时增大导致数据库崩溃
        - Redis中没有出现大量的key过期，Redis正常运行
      - 原因
        - Redis中某个key过期，但是对于这个key存在大量的访问
      - 解决方案
        - 预先设置热门数据
        - 实时调整：线程监控数据热门，实时调整key过期时长
        - 使用排他锁，效率低，但是能解决问题
  
    - ##缓存雪崩
  
      - 现象
        - 数据库压力变大导致Redis崩溃，应用崩溃，服务器崩溃
      - 原因
        - 在极短时间内出现大量key过期
      - 解决方案
        - 构建多级缓存
          - nginx缓存+redis缓存+其他缓存
        - 使用锁或队列
          - 保证不会有大量的线程对数据库进行一次性读写，但是不适合高并发情况
        - 设置过期标志更新缓存
          - 记录缓存数据是否已过期，如果过期则会通知另一个线程在后台更新实际key的缓存
        - 分散缓存过期时间
          - 在原有的失效时间的基础上增加一个随机值
  
  - ## 分布式锁
  
    - 在单机部署的情况下，若要对临界区的访问加上限制，一把锁即可，但是在分布式服务中，多个服务器无法通过一把锁得到限制，分布式锁就是用来解决这个问题的
  
    - 实现方法
  
      - 通过数据库实现
  
      - 通过Redis实现：性能最高
  
        - 实现方法1
  
          - 加锁：通过setnx添加一个key（mutex），最好设置一个锁的过期时间，避免进程过长占用锁
  
          - 释放锁：删除mutex
  
          - 但是对于上锁和设置过期时间这两个操作可能不能连贯的进行，可以通过指令使其同时进行
  
            ```shell
            set mutex 10 nx ex 10
            ```
  
            - nx表示setnx，ex表示过期时间
  
      - 通过Zookeeper ：可靠性最高
  
    - 问题
  
      - a服务器在获取锁后突然卡顿或宕机，当锁自动过期时，锁被b抢到并上锁，a回归继续操作并释放了b的锁
        - 解决方案1
          - 使用UUID设置mutex的值放置误删
          - 释放锁前判断自己的uuid和锁的uuid是否相同
          - 存在的问题，UUID的比较和删除可能无法连贯的进行任然可能导致误删
        - 解决方案2
          - 在解决方案一的基础上使用lua脚本
          - UUID的比较和删除通过lua脚本实现，lua脚本在Redis中具有原子性
  
    - 使用锁需要保证的四个特性
  
      - 互斥性：任意时刻只能有一个客户端拥有锁
      - 不会发生死锁：通过锁的过期实现
      - 加锁和释放锁的必须是同一个客户端
      - 加锁和解锁必须具有原子性
  
- ##Redis6新功能

  - ###ACL

    - 访问控制列表，增加权限控制
    - 常用命令使用
      - acl list（显示所有用户）
        - "user default on nopass ~* &* +@all"
          - default：用户名
          - on：是否启用该用户
          - nopass：是否需要密码
          - ~* &* ：表示能够操作的key（这里是全部key）
          - @all：可执行的命令（这里是全部命令）
      - acl whoami
        - 查看当前用户
      - acl cat（显示可操作的指令）
      - acl setuser（创建和编辑用户acl）
        - 例：acl setuser jj
          - 添加新用户jj
        - 例：acl setuser gg on >password ~cached:*+get
          - 表示创建一个可使用的用户 密码为password，只能对cached:开头的key进行操作，只能使用get命令
      - auth username password
        - 切换到username 用户

  - ###IO多线程

    - Redis的命令执行依旧是单线程，多线程部分用于处理网络数据读写和协议解析
    - 默认不开启，在配置文件中可开启（io-threands-doreads yes，io-threads 4）

  - ###工具包支持cluster

    - 在Redis5以前使用集群需要单独配置ruby环境但是Redis6之后的版本将ruby集成到了redis-cli工具中

