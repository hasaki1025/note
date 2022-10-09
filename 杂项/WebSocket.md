# WebSocket

## 1、关于web端即时通讯

web端即时通讯技术简单的说就是实现这样一种功能：服务器端可以即时地将数据的更新或变化反应到客户端，例如消息即时推送等功能都是通过这种技术实现的。

但是在Web中，由于浏览器的限制，实现即时通讯需要借助一些方法。这种限制出现的主要原因是，一般的Web通信都是浏览器先发送请求到服务器，服务器再进行响应完成数据的现实更新。

实现即时通讯主要有四种方式，它们分别是：短轮询、长轮询(comet)、长连接(SSE)、WebSocket。

- 它们大体可以分为两类，一种是在HTTP基础上实现的，包括短轮询、comet和SSE；另一种不是在HTTP
- 基础上实现是，即WebSocket。下面分别介绍一下这四种轮询方式，以及它们各自的优缺点。

#### 1、ajax短轮询

短轮询的基本思路就是浏览器每隔一段时间向服务器发送http请求，服务器端在收到请求后，不论是否有数据更新，都直接进行响应。

#### 2、ajax comet -长轮询

comet 指的是，当服务器收到客户端发来的请求后，不会直接进行响应，而是先将这个请求挂起，然后判断服务器端数据是否有更新。如果有更新，则进行响应，如果一直没有数据，则到达一定的时间限制（服务器端设置）后关闭连接。明显减少了很多不必要的http请求次数，相比之下节约了资源。长轮询的缺点在于，连接挂起也会导致资源的浪费

#### 3、[SSE](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fgoloving%2Fp%2F9196066.html)

SSE是HTML5新增的功能，全称为Server-SentEvents。它可以允许服务推送数据到客户端。SSE在本质上就与之前的长轮询、短轮询不同，虽然都是基于http协议的。而SSE最大的特点就是不需要客户端发送请求，可以实现只要服务器端数据有更新，就可以马上发送到客户端。

SSE的优势很明显，它不需要建立或保持大量的客户端发往服务器端的请求，节约了很多资源，提升应用性能SSE的实现非常简单，并且不需要依赖其他插件。(不支持IE浏览器，单向通道)

#### 4、WebSocket

HTML5 定义的 WebSocket 协议，WebSocket 是独立的、创建在 TCP 上的协议，与传统的http协议不同，该协议可以实现服务器与客户端之间全双工通信。

简单来说，首先需要在客户端和服务器端建立起一个连接，这部分需要http。连接一旦建立，客户端和服务器端就处于平等的地位，可以相互发送数据，不存在请求和响应的区别。它能更好的节省服务器资源和带宽，并且能够更实时地进行通讯[
 
 
 
 
 ](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fweixin_42313701%2Farticle%2Fdetails%2F107139352)

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35fc0234c30e4c348d14b9b9a17e180e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)请求细节分析：

浏览器先向服务器发送个url以ws://开头的http的GET请求，服务器根据请求头中

Connection ：Upgrade 我要升级

Upgrade:websocket把客户端的请求升级websocket协议。

响应头中也包含了内容Upgrade:websocket，表示升级成WebSocket协议。

响应101: 握手成功，http协议切换成websocket协议了，连接建立成功，浏览器和服务器可以随时主动发送消息给对方了。

这里 Sec-WebSocket-Accept 和 Sec-WebSocket-Key 是配套的，主要作用在于提供基础的防护，减少恶意连接、意外连接。

Sec-WebSocket-Accept 的计算方法是：

计算公式为：

```scss
将 Sec-WebSocket-Key 跟 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 拼接。
 
通过 SHA1 计算出摘要，并转成 base64 字符串。

伪代码：

toBase64( sha1( Sec-WebSocket-Key + 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 )  )
复制代码
```

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ea1b4f00eeb4f9293255ca685535916~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

对比：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39869cec7ae146758d7adb17b494df16~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp) 从兼容性角度考虑，短轮询>长轮询>长连接SSE>WebSocket

从性能方面考虑，WebSocket>长连接SSE>长轮询>短轮询

## 2、[各浏览器对webscoket的支持情况](https://link.juejin.cn?target=https%3A%2F%2Fcaniuse.com%2Fwebsockets)

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77f333beaaf147d6ae98f4889f57e565~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 3、websocket应用场景

决定手头的工作是否需要使用WebSocket技术的方法很简单：

- 你的应用提供多个用户相互交流吗？
- 你的应用是展示服务器端经常变动的数据吗？

以下是一些典型的应用场景：

1. 协同办公 / 编辑

我们生活在分散式办公的时代，时常需要在不同地点同时编辑同一份文档，比如 腾讯在线office文档、编程文件等。

1. 社交 / 订阅

比如微信朋友圈的实时更新提醒、点赞或评论的红点通知，比如qq的特别关注人的动态提醒，比如聊天信息的实时同步，比如新闻客户端的订阅通知等等。

1. 多玩家游戏

对于在线实时的多人游戏，互动效率是非常重要的，你可不想在扣动扳机之后，你的对手却已经在10秒钟之前移动了位置。

1. 股市基金报价

金融界瞬息万变——几乎是每毫秒都在变化。过时的信息也只能导致损失，我们人类的大脑不能持续以那样的速度处理那么多的数据，需要一些算法来帮我们处理这些事情。当你有一个显示盘来跟踪你感兴趣的公司时，你肯定想要随时知道他们的价值，而不是10秒前的数据。使用WebSocket可以流式更新这些数据变化而不需要等待。

1. 体育实况播放

在体育播报的体验中，减低时延是最重要的一点。

1. 音视频聊天 / 视频会议 / 在线教育

用WebSockets getUserMedia API和HTML5音视频元素明显是个不错的选择。WebRTC的出现顺理成章的成为我刚才概括的组合体，它看起来很有希望，但其缺乏目前浏览器的支持。

1. 基于位置的应用

越来越多的开发者借用移动设备的GPS功能来实现他们基于位置的网络应用。比如共享单车、共享汽车、百度天眼，地图GPS服务、疫情监控目标人的实时运动轨迹、运动员的轨迹分析。借用WebSocket TCP链接可以让数据飞起来。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09fc6f49c14c48e69bc9b487c3cf0d03~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

## 4、[WebSocket、SockJs、STOMP三者关系](https://link.juejin.cn?target=https%3A%2F%2Fwww.cnblogs.com%2Fgoloving%2Fp%2F14735257.html)

#### SockJs

SockJS是一个JavaScript库，为了应对许多浏览器不支持WebSocket协议的问题，设计了备选SockJs。SockJS 是 WebSocket 技术的一种模拟。SockJS会尽可能对应 WebSocket API，会优先选择WebSocket进行连接，但是当服务器或客户端不支持WebSocket时，会自动在 XHR流、XDR流、iFrame事件源、iFrame HTML文件、XHR轮询、XDR轮询、iFrame XHR轮询、JSONP轮询 这几个方案中择优进行连接。

#### Stompjs

STOMP—— Simple Text Oriented Message Protocol——面向消息的简单文本协议。

SockJS 为 WebSocket 提供了 备选方案。但无论哪种场景，对于实际应用来说，这种通信形式层级过低。 STOMP协议：来为浏览器 和 server 间的 通信增加适当的消息语义。

WebSocket协议定义了两种类型的消息（文本和二进制），但是没有定义消息语义。协议定义了一种机制，供客户端和服务器协商子协议（即更高级别的消息传递协议），以便在WebSocket上使用它来定义每个消息可以发送哪些类型、格式是什么、每个消息的内容等等。子协议的使用是可选的，但无论如何，客户端和服务器都需要就一些定义消息内容的协议达成一致。

\

简而言之，WebSocket 是底层协议，SockJS 是WebSocket 的备选方案，也是底层协议，而 STOMP 是基于 WebSocket（SockJS）的上层协议。

1、HTTP协议解决了 web 浏览器发起请求以及 web 服务器响应请求的细节，假设 HTTP 协议 并不存在，只能使用 TCP 套接字来 编写 web 应用。

2、直接使用 WebSocket（SockJS） 就很类似于 使用 TCP 套接字来编写 web 应用，因为没有高层协议，就需要我们定义应用间所发送消息的语义，还需要确保连接的两端都能遵循这些语义；

3、同HTTP在TCP 套接字上添加请求-响应模型层一样，STOMP在WebSocket 之上提供了一个基于帧的线路格式层，用来定义消息语义；

消息格式：

连接

```csharp
[CONNECT
 accept-version:1.1,1.0
 heart-beat:10000,10000]
复制代码
```

发送消息

```javascript
[SEND
destination:/app/chat.addUser
content-length:36
{"sender":"tony","type":"JOIN"}]
复制代码
```

订阅消息

```typescript
[SUBSCRIBE
 id:sub-0
 destination:/topic/public
 "]
复制代码
```

服务器广播消息

```perl
[MESSAGE
 destination:/topic/public
 content-type:application/json;charset=UTF-8
 subscription:sub-0
 message-id:axsspnqm-15
 content-length:67
 {"type":"JOIN","content":null,"sender":"tony","receiver":null}]
复制代码
```

## 5、实战：

- Springboot使用WebSocket有两种方法，一种是采用Spring中的WebSocket，一种是采用Tomcat的WebSocket，这里采用Tomcat的方案

  - 具体步骤

    - 注入ServerEndpointExporter(如果项目采用内置Web容器则需要注入，如果采用外置Tomcat则无需注入由容器创建)

      ```java
      
      @Configuration
      public class WebSocketConfig {
          /**
           * 	注入ServerEndpointExporter，
           * 	这个bean会自动注册使用了@ServerEndpoint注解声明的Websocket endpoint
           */
          @Bean
          public ServerEndpointExporter serverEndpointExporter() {
              return new ServerEndpointExporter();
          }
      
      }
      ```

    - 创建一个ServerEndpoint

      ```java
      @Component
      @Slf4j
      @ServerEndpoint("/websocket/{name}")
      public class WebSocket {
      
          /**
           *  与某个客户端的连接对话，需要通过它来给客户端发送消息
           */
          private Session session;
      
          /**
           * 标识当前连接客户端的用户名
           */
          private String name;
      
          /**
           *  用于存所有的连接服务的客户端，这个对象存储是安全的
           */
          private static ConcurrentHashMap<String,WebSocket> webSocketSet = new ConcurrentHashMap<>();
      
      
          @OnOpen
          public void OnOpen(Session session, @PathParam(value = "name") String name){
              this.session = session;
              this.name = name;
              // name是用来表示唯一客户端，如果需要指定发送，需要指定发送通过name来区分
              webSocketSet.put(name,this);
              log.info("[WebSocket] 连接成功，当前连接人数为：={}",webSocketSet.size());
          }
      
      
          @OnClose
          public void OnClose(){
              webSocketSet.remove(this.name);
              log.info("[WebSocket] 退出成功，当前连接人数为：={}",webSocketSet.size());
          }
      
          @OnMessage
          public void OnMessage(String message){
              log.info("[WebSocket] 收到消息："+message);
              webSocketSet.get(name).session.getAsyncRemote().sendText("I get you message,you will die");
          }
      
      
      }
      
      ```

    - 使用原生的JS编写一个前端

      ```java
      //js代码
      function createWS() {
      
      
          var websocket = new WebSocket("ws://127.0.0.1/websocket/testname");
      
          websocket.onopen = function(){
              websocket.send('this a message from web');
              console.log("连接成功");
          }
      
          websocket.onclose = function(){
              console.log("退出连接");
          }
      
          websocket.onmessage = function (event){
              console.log("收到消息"+event.data);
          }
      
          websocket.onerror = function(){
              console.log("连接出错");
          }
      
          window.onbeforeunload = function () {
              websocket.close(num);
          }
      
      }
      //HTML代码
      <!DOCTYPE html>
      <html lang="en">
      <head>
          <meta charset="UTF-8">
          <title>WebSocket</title>
          <script src="WS.js"></script>
      </head>
      <body>
      
      
      <button onclick="createWS()">发送</button>
      </body>
      </html>
      
      ```

      