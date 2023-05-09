# Mysql存储引擎

- ## MyISAM

  - mysql5.5之前默认采用引擎

  - 不支持事务、外键、采用表锁，优势是访问速度快

  - ### 文件系统

    - 表定义存储采用.frm
    - 表数据采用.MYD
    - 索引文件采用.MYI

    - 创建表时可以通过data directory 和index directory 指定存储位置(绝对路径)

  - MyISAM引擎管理的表可能会损害（原因很多），可以通过check table_name检查表是否损坏，repair table指令可以修复损坏表

  - ### 存储格式

    - 静态表(默认)
      - 所有字段采用非变长字段
      - 优势
        - 访问迅速，修复快
      - 劣势
        - 占用空间多，未达到指定宽度的字段采用空格在后面填充（但是如果数据本身有空格也会被去除）
    - 动态表
      - 采用变长
      - 优势
        - 占用空间小
      - 劣势
        - 易产生碎片，需要定期执行optimize table 或者myisamchk-r清除碎片
    - 压缩表
      - 占用空间小，访问开支小

- ## innoDB引擎

  - 5.5之后默认采用的引擎

  - 相比于MyISAM写的效率要差一点，占用空间大

  - 特点

    - 自增序列

      - 可以通过alter table  table_name auto_increment = n 强制设置自增初始值,但是自增初始值是保存在内存中的，重启机器后需要重新设置
      - 可以通过last_insert_id()函数查询最后插入记录所用的值
      - 使用自增序列的字段必须有索引，如果是组合索引也必须是组合索引的第一个（但是MYISAM可以不用）

    - 外键

      - 只有innoDB才有外键

      - 外键要求父表必须含有索引，子表创建外键后也会自动创建索引

      - 创建索引时可以指定在更新和删除父表记录时子表的行为

        - restrict
          - 拒绝操作

        - cascade
          - 级联（更新或删除）
        - set null
          - 设置为null
        - no action
          - 和restrict一样

      - 被引用的字段上的索引被禁止删除，在执行数据导入时可以通过以下命令暂时屏蔽外键检查

        ```sql
        //执行前
        set foreign_key_checks=0
        //执行后
        set foreign_key_checks=1
        ```

      - 可以通过show create table 或者 show table status查看外键

  - ### 存储方式

    - 共享表空间存储

      - 表结构保存在.frm文件中，数据和索引发布保存在innodb_data_home_dir和innodb_data_file_path定义的表空间中

    - 多表空间存储

      - 表结构在.frm文件中，但是每个表的数据和 索引放在.ibd文件中，如果是个分区表则每个分区对应单独的.ibd文件中，文件名为 表名+分区名,创建分区时可以指定每个分区的数据文件位置

      - 要使用这种方式需要设置参数innodb_file_per_table并重新启动

      - 这种方式方便进行数据备份和恢复，复制指定的.ibd文件后执行以下指令

        ```sql
        alter table table_name  discard tablespace
        alter table table_name  import tablespace
        ```

        - 这种方式只能恢复到该数据本来所在的数据库位置，不能到其他数据库中
        - 这种方式下共享表空间仍然是必须的（用于存储内部数据词典和在线重做日志）
        
        