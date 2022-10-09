# Docker

- ## 简介

  - 将整个软件开发环境和配置等全部打包成一个镜像文件

  - 基本组成

    - 镜像：模板
    - 容器：实例
    - 仓库：存放镜像

  - 基本流程

    - 客户端（Docker client）发起命令至后台守护进程（Docker Daemon）

      - bulid
        -  通过镜像创建容器
      - pull
        - 如果没有镜像则从远程仓库中获取镜像并创建容器

      - run
        - 运行容器

- ## 安装

  - 系统要求

    - Centos7内核3.8以上系统64bit，可通过 uname -r查看内核版本

    - 安装教程请看https://docs.docker.com/engine/install/centos/

    - 注意以下安装命令中的url需替换为国内aliyun的镜像

    - ```
      国外
      yum-config-manager \
          --add-repo \
          https://download.docker.com/linux/centos/docker-ce.repo
       
       国内
      yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
      ```

- ## Docker常用命令

  - ### 帮助启动类命令

    - 启动

      ```shell
      systemctl start docker
      ```

    - 停止

      ```shell
      systemctl stop docker
      ```

    - 重启

      ```shell
      systemctl restart docker
      ```

    - 查看docker状态

      ```shell
      systemctl status docker
      ```

    - 开机启动docker

      ```shell
      systemctl enable docker
      ```

    - 查看docker概要信息

      ```shell
      docker info
      ```

    - 查看docker总体帮助文档

      ```shell
      docker --help
      docker 具体命令 --help
      ```

  - ### 镜像命令

    - 查看所有镜像

      ```shell
      docker images
      docker images -a 列出所有镜像包括历史镜像
      docker images -q 只显示镜像ID
      ```

      - 显示的信息中tag表示版本号，若无指定版本号则默认为latest

    - 在仓库中查找某一镜像

      ```shell
      docker search 关键字
      docker search --limit 4 关键字 #代表只列出4个镜像
      ```

      - 显示的信息中automated为是否自动构建，official代表是否官方认证

    - 拉取镜像

      ```shell
      docker pull 镜像名称
      docker pull 镜像名称:tag
      ```

    - 查看某个镜像/容器/数据卷所占用空间

      ```shell
      docker system df
      ```

    - 删除某个镜像（可以同时删除多个）

      ```shell
      docker rmi 镜像ID1/仓库名1 镜像ID2/仓库名2...
      docker rmi -f 镜像ID/仓库名 #强制性删除
      docker rmi -f $(docker images -qa) #类似于LINUX管道命令
      ```

  - ### 容器命令

    - 新建+启动容器

      ```shell
      docker run [options] image [command][arg..]
      #常用option
      --name=容器新名称 重命名
      -d 后台运行容器并返回容器ID
      -i 以交互模式运行容器通常和-t同时使用
      -t 为容器重新分配一个伪输入终端,退出终端输入exit
      -P 随机端口映射
      -p 指定端口映射 如-p 80:8080
      ```

      - docker run -d image 代表后台启动某个容器，但是对于某些应用如果不配合-it,没有前台输入终端的镜像会立即自杀，导致无法后台启动容器

    - 列出所有正在运行的容器

      ```shell
      docker ps [options]
      #常用option
      -a 列出所有当前正在运行的容器+历史上运行过的容器
      -l 显示最近创建的容器
      -n 显示最近n个创建的容器
      -q 只显示容器编号
      ```

    - 进入正在运行的容器并以命令行交互

      ```shell
      docker exec -it 容器ID/容器名称 /bin/bash
      docker attach 容器ID/容器名称
      ```

      - 可以理解为每一个Docker容器都是一个缩小版linux系统，而安装的软件镜像运行在该容器上也就是linux系统上，所以指定/bin/bash就可以将该容器视为linux并进入到该容器中

      - attach和exec的区别

        - attach直接进入容器启动命令的终端，用exit退出会直接导致容器的停止
        - exec在容器中打开新的终端并通过exit退出不会导致容器的停止

        

      - 退出容器

      ```shell
      exit #退出容器并停止容器
      ctrl+p+q #退出容器但容器不停止
      ```

      - 启动已经停止运行的容器

      ```shell
      docker start 容器ID/容器名称
      ```

      - 重启容器

      ```shell
      docker restart 容器ID/容器名称
      ```

      - 停止容器

      ```shell
      docker stop 容器ID/容器名称
      ```

      - 强行停止容器

      ```shell
      docker kill 容器ID/容器名称
      ```

      - 删除已停止的容器

      ```shell
      docker rm 容器ID/容器名称
      docker rm -f 容器ID/容器名称 #强制删除
      docker rm -f ${docker ps -a -q} #类似管道符
      ```

      - 查看容器日志

      ```shell
      docker logs 容器ID/容器名称
      ```

      - 查看容器内进程运行

      ```shell
      docker top 容器ID/容器名称
      ```

      - 查看容器内部细节

      ```shell
      docker inspect 容器ID/容器名称
      ```

      - 从容器内拷贝文件到主机上
      
        ```shell
        docker cp 容器ID/容器名称:容器内路径 目的主机路径
        ```
      
      - 导入和导出容器
      
        - 导出
      
          ```shell
          docker export 容器ID/容器名称 > 文件名.tar
          ```
      
        - 导入
      
          ```shell
          cat 文件名.tar | docker import-镜像用户/镜像名：镜像版本号
          ```
  
- ## Docker镜像

  - 底层原理

    - 镜像是分层的，底层采用UnionFS（联合文件系统），底层采用bootfs引导文件系统
    - 采用分层的好处是底层的镜像可以被复用
    - 所有镜像都是只读的，但是镜像之上的容器层是可修改的

  - 容器提交

    - 将docker容器提交使其成为一个新的镜像

      ```shell
      docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]
      ```

      - 注意：将容器制作为镜像时不要停止被制作的容器，在容器中的所有操作都不会被写入镜像文件中，需要在容器还在运行时制作镜像。

  - 容器上传至腾讯云容器服务

    - 在控制台中新建命名空间

    - 创建镜像仓库

    - 登录腾讯云容器镜像服务 Docker Registry

      ```shell
      docker login ccr.ccs.tencentyun.com --username=100025281492
      ```

    - 向 Registry 中推送镜像

      ```shell
      docker tag [imageId] ccr.ccs.tencentyun.com/yuxidocker/docker:[tag]
      docker push ccr.ccs.tencentyun.com/yuxidocker/docker:[tag]
      ```

    - 从腾讯云中拉取仓库

      ```shell
      docker pull ccr.ccs.tencentyun.com/yuxidocker/docker:[tag]
      ```

  - Docker私服
  
    - 下载镜像docker registry
  
      ```shell
      docker pull registry
      ```
  
    - 运行registry
  
      ```shell
      docker run -d  -p 5000:5000 -v /zzyyuse/myregistry/:/tmp/registry --privileged=true registry
      ```
  
    - 查看私服仓库列表
  
      ```shell
      curl -XGET http://192.168.10.10:5000/v2/_catalog
      ```
  
    - 制作需要推送的镜像
  
      ```shell
      docker tag 镜像ID或者镜像名称 192.168.10.10:5000/镜像ID或者镜像名称
      ```
  
      - 此时会在镜像中克隆一份需要推送的镜像
  
    - docker 默认不支持http推送，可以通过配置文件修改
  
      ```shell
      vim /etc/docker/daemon.json
      #新增 
      "insecure-registries": ["192.168..10.10:5000"]
      #重启docker 
      systemctl restart docker
      #重新启动registry服务
      ```
  
    - 镜像推送
  
      ```shell
      docker push 镜像名称
      ```
  
- ## Docker容器数据卷

  - 容器卷实现宿主机和docker之间文件共享和同步

    ```shell
    docker run -v 宿主文件路径:容器内文件路径
    #注意有时候docker 修改本地文件可能会宿主机拒绝，这是由于权限不足，在命令后添加--privileged=true即可
    docker run -v 宿主文件路径:容器内文件路径 --privileged=true
    ```

    - 尽管当时创建容器卷的容器被删除了，宿主机中被共享的文件还在

  - 可以通过inspect查看容器的详细信息，其中包括了容器卷信息，在mounts中

  - 也可以设置数据卷为只读

    ```shell
    docker run -v 宿主文件路径:容器内文件路径:ro
    #ro代表只读，rw代表可读可写
    ```

    - 但是这种限制也仅限于容器内部，对于宿主机仍然是可读可写的

  - 容器卷的继承

    ```shell
    docker run --volumes-from 父类容器 --name 子类容器
    ```

- ## Docker File 

  - 可以理解为Docker的脚本文件，比如现在需要一个linux系统带有jdk环境，mysql，vim工具。这些改动一次一次进行过于麻烦，而使用一个文件让Docker一次执行就可以得到所需要的镜像，这既是Docker File
  
  - ### 基本语法
  
    - 每条保留字指令必须大写且后面至少要跟随一个参数
    - 指令从上到下依次执行
    - #为注释
    - 每条指令都会创建一个新的镜像并提交
  
  - ### 执行流程
  
    - 从一个镜像创建一个容器
    - 执行指令对容器进行修改
    - 对容器提交
    - 基于提交的容器之后产生的镜像再创建容器并再次进行修改
    - 直到DockerFile执行完成
  
  - ### Docker保留字指令
  
    - 首先需要介绍以下shell格式指令和exec格式指令和上下文路径
  
      ```dockerfile
      RUN shell指令
      RUN ["可执行文件", "参数1", "参数2"]
      # 例如：
      # RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
      # &&可以连接两个指令如以下,其中\代表换行
      RUN yum -y install wget \
          && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
          && tar -xvf redis.tar.gz
      ```
  
       - 上下文路径
  
      ```dockerfile
      $ docker build -t nginx:v3 上下文路径（如'.'代表当前路径）
      ```
  
      - 上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包
  
    - FROM
  
      - 父类镜像，DockerFile的第一行必须是FROM
  
        ```dockerfile
        FROM 镜像名称
        ```
  
    - MAINTAINER
  
      - 镜像维护者的姓名和邮箱
  
    - RUN
  
      - 容器构建时执行的命令（在build时执行）
  
        ```dockerfile
        RUN 命令（shell格式或者exec格式）
        RUN yum install vim
        RUN ["可执行文件","参数1"，"参数2"]
        ```
  
    - EXPOSE
  
      - 当前容器对外暴露端口
  
        ```dockerfile
        EXPOSE <端口1> [<端口2>...]
        ```
  
    - WORKDIR
  
      - 创建容器后，容器终端默认登录进入的工作目录
  
        ```dockerfile
        WORKDIR <工作目录路径>
        ```
  
    - USER
  
      - 指定该镜像由什么用户执行，如果不指定默认为root
  
        ```dockerfile
        USER <用户名>[:<用户组>]
        ```
  
    - ENV 
  
      - 在构建镜像过程中设置环境变量,这些环境变量可以在之后的DockerFile中使用
  
        ```dockerfile
        ENV <key> <value>
        ENV <key1>=<value1> <key2>=<value2>...
        ```
  
    - VOLUME 
  
      - 容器数据卷，用于数据保存和持久化工作
  
        ```dockerfile
        VOLUME ["<路径1>", "<路径2>"...]
        VOLUME <路径>
        ```
  
    - ADD
  
      - 将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar,相同需求下官方更加推荐使用COPY
  
    - COPY
  
      - 从上下文目录中复制文件或者目录到容器里指定路径
  
        ```dockerfile
        COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
        COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
        ```
  
        - 源文件和源目录可以使用通配符
  
    - CMD
  
      - 指定容器启动后(run之后)需要执行的指令
  
        ```dockerfile
        CMD  ["可执行文件","参数1"，"参数2"]
        CMD shell命令
        CMD ["参数1"，"参数2"] #这种写法是为了给ENTRYPOINT传递参数而使用的
        ```
  
        - CMD可以有多个，但是只有最后一个会生效，其次CMD会被docker run之后的参数取代
  
        - 示例
  
          ```dockerfile
          #假设dockerfile中最后的CMD为启动Tomcat，如果采用以下命令启动Tomcat镜像
          docker run -it Tomcat /bin/bash #/bin/bash既是run之后需要运行的指令
          #启动完成后原先DockerFile中最后一个CMD（用于启动Tomcat的CMD指令）将会被/bin/bash取代，这会导致Tomcat容器虽然运行了但是Tomcat并没有启动
          ```
  
    - ENTRYPOINT
  
      - 类似于CMD指令但是不会被docker run后面的命令覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序,但是,如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 ENTRYPOINT 指令指定的程序
  
      - 如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效
  
        ```dockerfile
        ENTRYPOINT ["指令","<param1>","<param2>",...]
        ```
  
        - 使用示例
  
          ```dockerfile
          FROM nginx
          
          ENTRYPOINT ["nginx", "-c"] # 定参
          CMD ["/etc/nginx/nginx.conf"] # 变参 
          ```
  
          - 如果直接使用docker run运行
  
            ```shell
            #nginx 容器实际启动命令
            nginx -c /etc/nginx/nginx.conf
            ```
  
          - 如果采用传参运行
  
            ```shell
            docker run nginx -c /etc/nginx/new.conf
            #实际nginx运行指令为
             docker run  nginx -c /etc/nginx/new.conf
            ```
  
          - 也就是说无论run是否传递参数,nginx都一定会携带ENTRYPOINT中的参数，如果不携带参数，则采用CMD中的参数，如果docker run携带参数则CMD的参数会被docker run之后的参数覆盖，用java来描述的话大概时以下形式
  
            ```dockerfile
            
            nginx -c run的参数==null ? CMD的参数：run之后的参数
            ```
  
    - ### ARG
  
      - 构建参数，与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。
  
        ```dockerfile
        ARG <参数名>[=<默认值>]
        ```
  
        
