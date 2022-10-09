# Nginx

- 启动Nginx

  - 切换到sbin目录下直接使用./nginx

    - 启动成功后会有两个nginx进程，一个是master，一个是worker，master管理worker

  - 指定配置文件启动

    ```shell
    ./nginx -c /usr/local/nginx/conf/nginx.conf
    ```

- 关闭Nginx

  - 较慢的关闭

    ```shell
    ps -ef | grep nginx
    kill -QUIT Masterpid
    ```

    - 这种关闭方式会先让nginx执行当前任务后才会关闭

  - 较快的关闭

    ```shell
    ps -ef | grep nginx
    kill -TERM Masterpid
    ```

    - 立即关闭Nginx

  - 重启Nginx

    ```shell
    ./nginx -s reload
    ```

  - 查看版本

    ```shell
    ./nginx -v
    ./nginx -V#更加详细
    ```

    

- ##Nginx主配置文件

  - 检查配置文件是否有问题

    ```shell
    ./nginx -c /usr/local/nginx/conf/nginx.conf -t
    ```

  - ###基础配置部分
  
    - 运行用户配置
  
      ```conf
      #user  nobody;
      ```
  
      - 在nginx运行后master进程通过root运行，而nobody运行worker进程
  
    - 配置worker进程的数量
  
      ```conf
      worker_processes  1;
      ```
  
      - 通常等于CPU数量或者是两倍CPU数量
  
    - 配置全局错误日志及类型
  
      ```conf
      #error_log  logs/error.log;
      ```
  
      - 可以配置debug，info，notice，warn，error，crit等级别
  
    - 配置进程pid文件
  
      ```conf
      pid        logs/nginx.pid;
      ```
  
  - ###events配置
  
    - 配置工作模式和连接数
  
      ```conf
      events {
          worker_connections  1024; 
      }
      ```
  
      - 一个worker进程的连接上限，上限是65535
  
  - ###http配置
  
    - 配置nginx支持的多媒体类型
  
      ```conf
      include       mime.types;
      ```
  
      - 在conf下有一个mime.types文件，可以查看支持的多媒体类型
  
    - 如果文件类型没有再mime中则默认以二进制流文件解析
  
      ```conf
      default_type  application/octet-stream;
      ```
  
    - 日志格式
  
      ```conf
      #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
          #                  '$status $body_bytes_sent "$http_referer" '
          #                  '"$http_user_agent" "$http_x_forwarded_for"';
      ```
  
    - 访问日志文件配置
  
      ```conf
      #access_log  logs/access.log  main;
      ```
  
    - 开启高效文件传输模式
  
      ```conf
         sendfile        on;
      ```
  
    - 防止网络堵塞
  
      ```conf
         #tcp_nopush     on;
      ```
  
    - 长连接超时连接，单位是秒
  
      ```conf
      keepalive_timeout  65;
      ```
  
    - 开启gzip压缩输出
  
      ```conf
       #gzip  on;
      ```
  
    - ####server配置
  
      - 可以有多个server，默认只配置一个
  
      - 服务端口号
  
        ```conf
        listen       80;
        ```
  
      - 服务主机名
  
        ```conf
        server_name  localhost;
        ```
  
        - 多个server之间要求主机名+端口号不能相同
  
      - 默认字符编码
  
        ```conf
        #charset koi8-r;
        ```
  
        - nginx默认utf-8
  
      - 虚拟主机访问日志
  
        ```conf
        #access_log  logs/host.access.log  main;
        ```
  
        - 访问该server才会记录
  
      - 匹配请求
  
        ```conf
        location / {
            root   html;
            index  index.html index.htm;
        }
        ```
  
        - 默认匹配斜杠/的请求，当访问路径中有/时会匹配到location配置中并处理
        - root表示项目根目录，默认是nginx中html文件夹
        - index表示默认首页位置
        - 简单来说就是将localhost:80绑定到html文件夹，之后访问localhost/index.html就是直接在html文件夹下找
  
      - 精准匹配
  
        ```conf
        location = /50x.html {
                    root   html;
                }
        ```
  
        - 如果要访问localhost/50x.html就会转发到html文件夹下
  
      - 配置错误页面
  
        ```conf
        #error_page  404              /404.html;
        
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        ```
  
- ##Nginx主要功能

  - ### 静态网站部署

    - Nginx只能部署静态资源

    - Nginx部署Vue项目

      - 打包Vue项目

        ```sh
        npm install
        npm run build
        ```

        - 使用后会在项目根目录下生成一个dist目录

      - 将dist目录上传到nginx中的html目录下（其他应该也行）

      - 修改配置文件

        ```conf
        location / {
            root   html;
            index  index.html index.htm;
        }
        
        location /Vue {
            root   /usr/local/nginx/html/dist;
            index  index.html index.htm;
        }
        ```

        - 注意：这里要在dist目录下创建Vue目录，比如访问路径是localhost/Vue/index.html,location会将其拼接为/usr/local/nginx/html/dist/Vue/index.html,也就是/永远和root等价

        - 但是如果只有一个Vue项目的话只能直接放置在/下而不能放置在/Vue类似目录下，可能与Vue配置有关，如果按照上面的配置会导致Vue读取不到js文件，所以实际情况下只需要将dist目录下的所有文件放置在html目录下，不改变配置文件即可

  - ###负载均衡

    - 将访问流量均分至其他服务器上

    - 负载均衡方式

      - 硬件负载
        - 造价较贵
      - 软件负载
        - 造价廉价

    - Nginx负载均衡

      - 修改配置文件

        ```conf
        location / {
            root   html;
            proxy_pass http://localhost;
            index  index.html index.htm;
        }
        ```

        - 其中proxy_pass的值http://固定不变，localhost可以随意更改比如abc等都可以，没有含义

      - 添加配置

        ```conf
        upstream localhost{
            server 127.0.0.1:8081;
            server 127.0.0.1:8082;
        }
        ```

        - localhost必须要和上面的proxy_pass的http://一致

    - 负载均衡策略

      - 轮询策略（默认）

        - 通过url的hash结果来分配请求，使得每一个url定向到同一个后端服务器上，主要应用域后盾服务器为缓存时的场景下，如果后端服务器down将自动剔除
        - 如果不同后端服务器的性能不同则会导致请求堆积的情况出现

      - 权重策略

        - 每个请求按一定比例分发到不同后端服务器上，weight越大访问比例越大，适用于性能不均的情况

          ```conf
          upstream localhost{
              server 127.0.0.1:8081 weight=5;
              server 127.0.0.1:8082 weight=2;
          }
          ```

      - 最小连接数

        - 请求会被转发给连接数最少的服务器上

          ```conf
          upstream localhost{
          	least_conn;
              server 127.0.0.1:8081 weight=5;
              server 127.0.0.1:8082 weight=2;
          }
          ```

      - ip_hash

        - 当用户发起请求时通过hash计算用户ip得到该ip所对应的服务器并将该请求转发给该服务器，这样就不会出现用户二次访问后找不到session

          ```conf
          upstream localhost{
          	ip_hash;
              server 127.0.0.1:8081 weight=5;
              server 127.0.0.1:8082 weight=2;
          }
          ```

    - 负载均衡其他配置

      - backup

        - 被指定为backup的服务器为备份服务器，只有当其他服务器全部宕机后才会工作，其他时候不工作，作用是当所有服务器需要更新代码时先在backup上更新，backup更新完后，关闭其他服务器并更新，此时backup起作用

          ```conf
          upstream localhost{
              server 127.0.0.1:8081 backup;
              server 127.0.0.1:8082;
          }
          ```

      - down

        - 让一个服务器永远不参与负载均衡

          ```conf
          upstream localhost{
              server 127.0.0.1:8081 down;
              server 127.0.0.1:8082;
          }
          ```

  - ### 静态代理

    - 将静态资源从Tomcat中抽离 

    - 静态代理的方式

      - （一）使用正则表达式匹配到所有静态资源

        ```conf
        location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
                root /opt/static;
        }
        ```

        

      - （二）使用正则表达式匹配静态资源路径

        ```conf
        location ~ .*/(css|js|img|images) {
            root   /opt/static;
        }
        ```

        - 匹配的路径

          ```conf
          xxx/css
          xxx/js
          xxx/img
          xxx/images
          ```

  - ### 动静分离

    - 将静态资源放置在Nginx中（html等文件），将动态资源放置在Tomcat中（java代码）
    - 

