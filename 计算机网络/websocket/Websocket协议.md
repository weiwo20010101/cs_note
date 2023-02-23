# Websocket协议

https://websockets.readthedocs.io/en/stable/

应用层协议：目的：即时通讯，替代轮询

应用场景：网站上的即时通讯是很常见的，比如网页的QQ，聊天系统等。按照以往的技术能力通常是采用轮询、Comet技术解决。

HTTP协议是非持久化的，单向的网络协议，在建立连接后只允许浏览器向服务器发出请求后，服务器才能返回相应的数据。当需要即时通讯时，通过轮询在特定的时间间隔（如1秒），由浏览器向服务器发送Request请求，然后将最新的数据返回给浏览器。这样的方法最明显的缺点就是需要不断的发送请求，而且通常HTTP request的Header是非常长的，为了传输一个很小的数据 需要付出巨大的代价，是很不合算的，占用了很多的宽带。

缺点：会导致过多不必要的请求，浪费流量和服务器资源，每一次请求、应答，都浪费了一定流量在相同的头部信息上

然而WebSocket的出现可以弥补这一缺点。在WebSocket中，只需要服务器和浏览器通过HTTP协议进行一个握手的动作，然后单独建立一条TCP的通信通道进行数据的传送。

原理：WebSocket同HTTP一样也是应用层的协议，但是它是一种双向通信协议，是建立在TCP之上的。

#### 连接过程 —— 握手过程

\1. 浏览器、服务器建立TCP连接，三次握手。这是通信的基础，传输控制层，若失败后续都不执行。
\2. TCP连接成功后，浏览器通过HTTP协议向服务器传送WebSocket支持的版本号等信息。（开始前的HTTP握手）
\3. 服务器收到客户端的握手请求后，同样采用HTTP协议回馈数据。
\4. 当收到了连接成功的消息后，通过TCP通道进行传输通信。



浏览器发出的 WebSocket 握手请求类似于下面的样子：

```
GET / HTTP/1.1Connection: UpgradeUpgrade: websocketHost: example.comOrigin: nullSec-WebSocket-Key: sN9cRrP/n9NdMgdcy2VJFQ==Sec-WebSocket-Version: 13
```

上面的头信息之中，有一个 HTTP 头是`Upgrade`。HTTP1.1 协议规定，`Upgrade`字段表示将通信协议从`HTTP/1.1`转向该字段指定的协议。`Connection`字段表示浏览器通知服务器，如果可以的话，就升级到 WebSocket 协议。`Origin`字段用于提供请求发出的域名，供服务器验证是否许可的范围内（服务器也可以不验证）。`Sec-WebSocket-Key`则是用于握手协议的密钥，是 Base64 编码的16字节随机字符串。

服务器的 WebSocket 回应如下。

```
HTTP/1.1 101 Switching ProtocolsConnection: UpgradeUpgrade: websocketSec-WebSocket-Accept: fFBooB7FAkLlXgRSz0BT3v4hq5s=Sec-WebSocket-Origin: nullSec-WebSocket-Location: ws://example.com/
```

上面代码中，服务器同样用`Connection`字段通知浏览器，需要改变协议。`Sec-WebSocket-Accept`字段是服务器在浏览器提供的`Sec-WebSocket-Key`字符串后面，添加 [RFC6456](http://tools.ietf.org/html/rfc6455) 标准规定的“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”字符串，然后再取 SHA-1 的哈希值。浏览器将对这个值进行验证，以证明确实是目标服务器回应了 WebSocket 请求。`Sec-WebSocket-Location`字段表示进行通信的 WebSocket 网址。

完成握手以后，WebSocket 协议就在 TCP 协议之上，开始传送数据。

>建立连接的过程借助了http，实际传输通信的过程是使用tcp



#### 与http的关系

相同点

- 都是一样基于TCP的，都是可靠性传输协议。
- 都是应用层协议。

不同点

- WebSocket是双向通信协议，模拟Socket协议，可以双向发送或接受信息。HTTP是单向的。
- WebSocket是需要握手进行建立连接的。

联系

- WebSocket在建立握手时，数据是通过HTTP传输的。但是建立之后，在真正传输时候是不需要HTTP协议的。

#### websocket机制

WebSocket 是 HTML5 一种新的协议。它实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯，它建立在 TCP 之上，同 HTTP 一样通过 TCP 来传输数据，但是它和 HTTP 最大不同是：

WebSocket 是一种双向通信协议，在建立连接后，WebSocket 服务器和 Browser/Client Agent 都能主动的向对方发送或接收数据，就像 Socket 一样；WebSocket 需要类似 TCP 的客户端和服务器端通过握手连接，连接成功后才能相互通信。