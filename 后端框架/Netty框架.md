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

  - 堵塞模式创建服务器

    ```java
    public class ServerIO {
        private static final Logger log = LoggerFactory.getLogger(ServerIO.class);
    
        public static void main(String[] args) throws IOException {
            ServerSocketChannel socketChannel = ServerSocketChannel.open();
            socketChannel.bind(new InetSocketAddress(8080));
            ArrayList<SocketChannel> channels = new ArrayList<>();
            while (true)
            {
                log.info("waiting for connect...");
                SocketChannel channel = socketChannel.accept();
                log.info("connect successfully");
                channels.add(channel);
                for (SocketChannel c : channels) {
                    ByteBuffer buffer = ByteBuffer.allocate(16);
                    c.read(buffer);
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

    

