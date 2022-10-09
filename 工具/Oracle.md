# Oracle

- ## 基本操作

  - 登录(使用SQLPlus)

    ```shell
    sqlplus 用户名/密码
    sqlplus [username]/[password][@connect_identifier][NOLOG][AS SYSDBA]
    
    connect_identifier是：host:[port][/service_name][:server][/instance_name]，如果是本地的数据库可以简写为数据库服务名称
    如：sqlplus system/123@ORCL
    ```

  - 查询OEM服务端口

    ```sql
    select dbms_xdb_config.gethttpsport from dual;#默认是5500端口
    ```

  - 启动OEM服务（默认开启）

    ```java
    exec DBMS_XDB_CONFIG.SETHTTPPORT(5500);
    ```

  - 通过Https://ip:5500/em/login访问OEM

  - CONNECT命令

    断开当前连接并建立新的连接

    ```sql
    CONNECT [username]/[password][@connect_identifier]
    ```

  - disconnect命令

    断开当前连接但是不退出SQL  plus环境

    ```sql
    disconnect
    ```

  - 每次执行SQL语句时Oracle都会将语句读入缓冲区，下一条SQL语句会覆盖原本缓冲区中的内容，可以通过edit指令保存缓冲区中的内容（默认用记事本打开）

  - 在SQLplus中编写完了SQL语句后可以通过RUN指令运行缓冲区中的SQL语句

  - 通过SAVE指令保存缓冲区中的SQL语句

    ```sql
    save filename [create] | [replace] | [append]
    #如果filename指定的文件不存在则会新建
    #如果要覆盖原本存在的文件则使用replace参数
    #如果要追加内容则使用append
    ```

  - 使用GET命令获取SQL文件

    ```sql
    GET filename [list] | [nolist]
    #使用list参数会将脚本文件导入到缓冲区中同时显示文件内容默认为list
    #使用nolist参数会将脚本文件导入到缓冲区中但是不显示文件内容
    ```

  - 使用start或者@执行sql文件

    ```sql
    start sqlfile
    @sqlfile
    #start和@的区别在于@不仅可以在登录后执行还可以在登录时执行（如下）
    sqlplus  system/123@orcl @sqlfile
    ```

  - Oracle中含有许多环境变量，如每页显示行数、自动提交方式、自动跟踪等可以通过show来显示值也可以通过set来设置其值

    

  - 查看数据库对象命令

    ```slq
    desc databaseName
    ```

  - 将SQL plus 中的内容存放至文件中

    ```sql
    spool D:\123.txt#开始记录内容
    spool off#到off之前的内容都会被写入到文件中
    ```

    