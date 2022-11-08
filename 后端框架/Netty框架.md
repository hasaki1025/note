# Netty框架

- ## NIO基础

  - 三大组件
    - channel
      - FileChannel
      - DatagramChannel(UDP)
      - SocketChannel(TCP)
      - ServerSocketChannel(TCP)
    - buffer
    - selector

- ## 文件系统

  - 使用一

    ```java
    public class TestByteBuffer {
    
        public static void main(String[] args) {
            try (FileChannel channel = new FileInputStream(new File("src/main/resources/123.txt")).getChannel()) {
                //创建缓冲区，缓冲区大小为10字节（分配内存）
                ByteBuffer byteBuffer = ByteBuffer.allocate(10);
                //使用信道读取字节流,并向缓冲区中写数据,返回值是读取到的字节流的数量，如果为-1则代表没有读取任何数据
                int readLen=0;
               do{
                   //使用信道读取文件并将数据流写入缓冲区中，返回值是此次读取的字节数量，-1代表没有读取
                   readLen = channel.read(byteBuffer);
                   //缓冲区切换为读模式
                   byteBuffer.flip();
                   //hasRemaining用于返回缓冲区中是否含有未读数据
                   while (byteBuffer.hasRemaining())
                   {
                       //获取缓冲区中的数据，一次一个字节（get方法）
                       System.out.print((char) byteBuffer.get());
                   }
                   System.out.println();
                   //清空缓冲区并切换为写模式
                   byteBuffer.clear();
               }while (readLen!=-1);
    
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```

  - Buffer基本结构

    - 三个重要属性

      - capacity(容量)
    - position(写入指针)
      - limit(写入限制)

  - 信道传输

    ```java
    public class ChannelTransfer {
        public static void main(String[] args) {
            try (
                FileChannel in = new FileInputStream("src/main/resources/123.txt").getChannel();
                FileChannel out = new FileOutputStream("src/main/resources/321.txt").getChannel();
            ) {
                in.transferTo(0,in.size(),out);//返回值为本次传输的字节数
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    
    ```
  
    - 信道直接传输一次最大2g，可以通过size判断是否传输完成
  
  - Path和Paths
  
    - Paths是获取Path的工具类
  
      ```java
       Path path = Paths.get("src/main/resources/123.txt");
      ```
  
    - Path类中内置许多和文件位置有关的方法，如文件路径正常化，获取上一级目录等方法
  
  - Files
  
    - 工具类，主要使用静态方法
  
      ```java
      Path path = Paths.get("src/main/resources/123.txt");
      Files.exists(path);
      ```
  
    - 含有拷贝方法，创建目录方法，递归遍历某个目录下的所有文件等
  
      ```java
       public static void main(String[] args) throws IOException {
              Path path = Paths.get("src/main");
              Files.walkFileTree(path, new SimpleFileVisitor<>() {
                  @Override
                  public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                      System.out.println(file.getFileName());
                      return super.visitFile(file, attrs);
                  }
              });
      
          }
      ```
  
      - FileVisitor中每个方法的返回值是FileVisitResult（枚举类）
  
        ```java
        public enum FileVisitResult {
        
            CONTINUE,//继续访问
        
            TERMINATE,//终止访问
        
            SKIP_SUBTREE,//跳过子树
        
            SKIP_SIBLINGS;//跳过兄弟姐妹
        }
        
        ```
  
      - 除了自己实现FileVisitor之外还可以使用jdk中的实现类SimpleFileVisitor,需要修改的方法自己覆盖就行（建议使用SimpleFileVisitor且不修改其父类调用的返回值）
  
      - 除了walkTree方法还可以使用walk方法只不过用walk会返回一个stream流
  
- ## 网络编程

  - ### 单线程堵塞模式创建服务器

    - 服务器
    
    ```java
    @Slf4j
    public class ServerIO {
    
        public static void main(String[] args) throws IOException {
            ServerSocketChannel socketChannel = ServerSocketChannel.open();
            socketChannel.bind(new InetSocketAddress(8080));
            ArrayList<SocketChannel> channels = new ArrayList<>();
            while (true)
            {
                log.info("waiting for connect...");
                SocketChannel channel = socketChannel.accept();//在这里堵塞线程
                log.info("connect successfully");
                channels.add(channel);
                for (SocketChannel c : channels) {
                    ByteBuffer buffer = ByteBuffer.allocate(16);
                    c.read(buffer);//建立链接后会在read方法堵塞
                    buffer.flip();
                    while (buffer.hasRemaining())
                    {
                        System.out.print((char) buffer.get());
                    }
                    System.out.println();
                }
            }
    
        }
    }
    
    ```
    
    - 客户端
    
    ```java
    public class Client {
        public static void main(String[] args) throws IOException {
            SocketChannel channel = SocketChannel.open();
            channel.connect(new InetSocketAddress("127.0.0.1",8080));
    
            System.out.println("waiting connect....");
        }
    }
    ```
    
    - 这种模式的服务端无法同时服务多个客户端，而且当服务端接受到客户端发送的数据后如果客户端再次发送数据，服务端将无法接受到（会执行下一次循环监听链接）
    
  - ### 单线程非堵塞服务器
  
    - 只需要设置
  
      ```java
      socketChannel.configureBlocking(false);
      ```
  
      设置了非堵塞链接后accept方法将不会堵塞并且如果没有建立链接则返回值为null，read方法也同理（没有数据返回0）
  
  - ### selector模式服务器
  
    - #### 服务器
  
      ```java
      @Slf4j
      public class SelectorServer {
          public static void main(String[] args) throws IOException {
              //创建selector用于管理channel
              Selector selector = Selector.open();
              //创建SocketChannel,设置缓冲区，设置非堵塞
              ServerSocketChannel server = ServerSocketChannel.open();
              ByteBuffer buffer = ByteBuffer.allocate(16);
              server.configureBlocking(false);
              //ServerSocketChannel向selector注册,并获取selectionkey
              SelectionKey selectionKey = server.register(selector, 0, null);
              //绑定accept事件
              selectionKey.interestOps(SelectionKey.OP_ACCEPT);
              server.bind(new InetSocketAddress(8080));
              while (true)
              {
                  //调用select方法监听事件发生,堵塞运行
                  log.info("selector waiting....");
                  selector.select();
                  log.info("selector get Event");
                  //发生事件，获取事件对应的selectionkey
                  Set<SelectionKey> keys = selector.selectedKeys();
                  log.info("keys size {}",keys.size());
                  Iterator<SelectionKey> keyIterator = keys.iterator();
                  while (keyIterator.hasNext())
                  {
                      SelectionKey key = keyIterator.next();
                      log.info("Event Type:{}",key.interestOps());
                      //根据事件类型进行不同的处理
                     if(key.interestOps()==SelectionKey.OP_ACCEPT)
                     {
                         try{
                             //连接类型获取信道
                             ServerSocketChannel channel = (ServerSocketChannel) key.channel();
                             log.info("{} Accept...",channel);
                             //监听
                             SocketChannel socketChannel = channel.accept();
                             //如果连接建立失败则取消执行该任务
                             if (socketChannel==null)
                             {
                                 key.cancel();
                             }
                             else {
                                 //设置监听后获取的连接的堵塞类型
                                 socketChannel.configureBlocking(false);
                                 //注册到selector中
                                 socketChannel.register(selector,SelectionKey.OP_READ,null);
                             }
                         }catch (Exception e)
                         {
                             e.printStackTrace();
                             //发生异常则取消执行
                             key.cancel();
                         }
                     }
                     else if (key.interestOps()==SelectionKey.OP_READ){
                          try
                          {
                              SocketChannel channel = (SocketChannel) key.channel();
                              int read = channel.read(buffer);
                              //如果没有数据可读则取消执行
                              if (read==-1)
                              {
                                  key.cancel();
                              }
                              else {
                                  buffer.flip();
                                  String s = BufferUtil.bufferToString(buffer);
                                  log.info("get message {} from {}",s,channel);
                              }
                          }catch (IOException e)
                          {
                              e.printStackTrace();
                              key.cancel();
                          }
                     }
                     //每次解决之后删除本次活跃的事件
                      keyIterator.remove();
                  }
              }
          }
      }
      
      ```
  
    - #### 客户端
  
      ```java
      public class Client {
          public static void main(String[] args) throws IOException {
              SocketChannel channel = SocketChannel.open();
              ByteBuffer buffer = ByteBuffer.allocate(16);
              try (FileChannel fileChannel = new FileInputStream("src/main/resources/123.txt").getChannel()) {
                  int read = fileChannel.read(buffer);
                  System.out.println("waiting connect....");
                  if (channel.connect(new InetSocketAddress("127.0.0.1",8080))) {
                      System.out.println("connect successfully");
                      buffer.flip();
                      channel.write(buffer);
                  }
              }catch (Exception e)
              {
                  e.printStackTrace();
              }
          }
      }
      
      ```
  
    - #### 常用方法
  
      - cancel方法（selectionkey）
  
        ```java
        key.cancel();
        ```
  
        如果发生了事件但是不处理则下次selector仍然会将该连接视为活跃连接（不在select堵塞），如果不打算处理该selectKey对应的事件应该使用cancel方法
  
    - #### selectedKeys集合元素手动删除
  
      - 在selector的selectedKeys返回的集合中不会主动删除key只会增加key，如果当key建立连接的channel执行玩任务后再次发生事件则会再次进入到建立连接的代码块中，者可能会产生异常，所以需要手动删除已经执行过的key
  
    - #### 异常处理
  
      - 如果不行进行异常处理则如果客户端的正常关闭和异常关闭都会导致服务端异常从而导致服务端程序停止
  
    - #### 消息边界问题
  
      - 目前的消息处理 
  
        ```java
        
        try
        {
            SocketChannel channel = (SocketChannel) key.channel();
            int read = channel.read(buffer);
            //如果没有数据可读则取消执行
            if (read==-1)
            {
                key.cancel();
            }
            else {
                buffer.flip();
                String s = BufferUtil.bufferToString(buffer);
                log.info("get message {} from {}",s,channel);
            }
        }catch (IOException e)
        {
            e.printStackTrace();
            key.cancel();
        }
        
        //处理Buffer的代码
        public static String bufferToString(ByteBuffer buffer)
        {
            StringBuilder stringBuffer = new StringBuilder();
            while (buffer.hasRemaining())
            {
                stringBuffer.append((char) buffer.get());
            }
            buffer.clear();
            return stringBuffer.toString();
        }
        ```
  
        - 问题一：如果数据一次读取不能完全读取可能会导致中文乱码问题，而且在消息长度不确定的情况下可能会出现半包和黏包的现象
        - 问题二：buffer如果为局部变量则出现半包现象会导致消息的丢失，如果buffer为全局变量对于多个Channel并不友好
  
      - ##### 问题一：解决方法
  
        - 固定最大消息长度
          - 缺点：浪费空间
        - 分割符分割方法
          - 缺点：效率低
  
        - TLV方式
          - 固定长度的字段用于指示后续消息长度，TLV主要是，类型，长度，数据
  
      - ##### 问题二：解决方法
  
        - 附件
          - 在selector注册时第三个参数可以作为附件与每个key相关联
  
      - ##### 改进
  
        - 读取Key注册
  
          ```java
          //创建附件
          ByteBuffer buffer = ByteBuffer.allocate(4);
          //设置监听后获取的连接的堵塞类型
          socketChannel.configureBlocking(false);
          //注册到selector中
          socketChannel.register(selector,SelectionKey.OP_READ,buffer);
          ```
  
        - 获取buffer
  
          ```java
          ByteBuffer buffer = (ByteBuffer) key.attachment();
          ```
  
        - 消息获取
  
          ```java
          public static String getMessgaeFromKey(SelectionKey key) throws IOException {
              SocketChannel channel = (SocketChannel) key.channel();
              ByteBuffer buffer = (ByteBuffer) key.attachment();
              int read = channel.read(buffer);
              if (read==-1)//无消息刻度
              {
                  key.cancel();
                  return "";
              }
              buffer.flip();
              StringBuilder s=new StringBuilder();
              while (buffer.hasRemaining())//读取并查看是否有\n
              {
                  char c = (char) buffer.get();
                  s.append(c);
                  if (c=='\n')
                  {
                      break;
                  }
              }
          
              if(s.toString().contains("\n"))//如果有\n代表已经读取了一条完整的消息
              {
                  buffer.compact();
              }
              else if (buffer.limit()==buffer.capacity())//没有\n且容量不足需要扩容
              {
                  ByteBuffer allocate = ByteBuffer.allocate(buffer.capacity() * 2);
                  allocate.put(StandardCharsets.UTF_8.encode(s.toString()));
                  key.attach(allocate);
              }
              return s.toString().endsWith("\n") ?  s.toString() : "";//返回获取的第一条消息（如果有\n则代表已经获取到了没有则代表还没读取完成）
          }
          ```
          
          
          
          
