## HTTP/HTTPS 协议

- 一个 **request** 对应一个 **response**。一般使用 **轮询** 机制获取信息。
- 虽然后期有 **keep-alive**，可以一次连接中处理多个 **request** 合并发送，接收多个 **response**，对每个请求仍然需要单独发 **header**。
- **HTTP** 的 **response** 是被动的，通信只能由客户端发起，做不到服务端主动向客户端推送信息。
- **HTTP** 是无状态协议。
- 需要三次握手。
- HTTP：TCP + HTTP
- HTTPS：TCP + HTTP + TLS

#### ajax 轮询

**ajax 轮询** 是指客户端 **不断** 向服务器发送资源请求 **request**，服务端不管有无目标资源，都会返回 **response** 结果。这样就要求服务器拥有更好的处理速度。

#### long poll / 长轮询

**long poll** 比 **ajax 轮询** 好点，**不会不断** 向服务器发送资源请求 **request**，一旦接受客户端的请求，有资源就返回；无资源就挂起，直到有资源才返回 **response** 结果（有且仅有返回一次）。高并发，需要服务器可以同时处理多个请求的能力。

## websocket 协议

- **websocket** 是 **html5** 出的协议。
- **websocket** 是一个持久化的协议，**websocket** 只需要一次请求（一次 **HTTP 握手** ），就可以得到所需的资源。
- 服务端可以 **主动** 向客户端推送信息，客户端也可以 **主动** 向服务端推送消息，实现“双向平等”，是属于服务器推送技术的一种。
- 与 **HTTP** 有较好的兼容性，在握手阶段采用 **HTTP** 协议。
- 默认端口 **80** and **443**。
- 数据格式轻量，性能开销小，通信高效，可以发送文本，也可以二进制数据。
- 没有同源策略。
- WS：TCP + WS
- WSS：TCP + WS + TLS

#### websocket 握手

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

- Upgrade/Connection：通知服务器发起的是 **wesocket** 协议。
- Sec-WebSocket-Key：浏览器随机生成的 **Base64 encode** 的值，用于验证服务器是不是 **websocket 协议**。
- Sec-WebSocket-Protocol：用户定义的字符串，用来区分 **同 url** 下，不同的服务所需要的协议。
- Sec-WebSocket-Version：协议版本。

#### 服务器返回

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

- Upgrade/Connection：通知客户端升级的是 **websocket** 协议。
- Sec-WebSocket-Accept：经服务器确认，加密后的 **Sec-WebSocket-Key**。
- Sec-WebSocket-Protocol：最终使用的协议。

#### 代码实现

```
ws.addEventListener('open', function () {
    console.log('Open the websocket...');
    ws.send('Hello websocket...');
});
ws.addEventListener('message', function (e) {
    console.log('Ready state: ' + ws.readyState, 'Receive the websocket message...' + e.data);
    ws.close();
    setTimeout(() => {
        console.log('Ready state: ' + ws.readyState);
    }, 1000)
});
ws.addEventListener('close', function () {
    console.log('Ready state: ' + ws.readyState, 'Close the websocket...');
})
```

结果：
![image.png](http://upload-images.jianshu.io/upload_images/659084-021a1bb1915811bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参考：

![image.png](http://upload-images.jianshu.io/upload_images/659084-d06ed1bfe9c9fadb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- https://www.zhihu.com/search?type=content&q=websocket
- http://www.ruanyifeng.com/blog/2017/05/websocket.html