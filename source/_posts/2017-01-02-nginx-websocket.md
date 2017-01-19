---
title: WebSocket的实现以及通过Nginx进行反向代理
date: 2017-01-02 13:29:36
category: server
tags:
- Nginx
- HTTPS
- WebSocket
---

# 什么是Websocket

`Websocket` 是HTML5开始推出的一种新的协议，实现了浏览器与服务端的全双工通信，在使用WebSocket时，，只要和服务端做一个握手(handshaking)动作，浏览器首先要向服务端发起一个特殊的HTTP请求，其头部附加了信息`Upgrade: WebSocket`，表明这是一个申请协议升级的HTTP请求，服务端解析出来这些信息后，产生一个应答给客户端，这样双方的WebSocket连接就建立起来了，即可形成一条全双工的数据通道，两者之间可以进行互相通信，直到客户端和服务端中的某一方主动关闭连接。

在`WebSocket`出现之前，为了解决浏览器和服务端之间的实时推送问题，采取了很多解决方案，通常使用的是轮询(Polling)和Comet技术，这些方案带来很明显的缺陷就是需要由浏览器主动发出HTTP Request，大量消耗服务器的带宽和资源，所以为了解决这些问题，HTML5推出了WebSocket。

`Socket.io` 是WebSocket协议的一个拓展，由于浏览器端对WebSocket的支持程度不一，为了能够兼容不同的浏览器，提升用户体验，简化程序员编写代码时的复杂度，`Socket.io`封装了以上几种不同的实时通讯机制，并实现了统一的接口，在实际应用的过程中，如果浏览器支持WebSocket，就会以WebSocket方式与服务端进行实时数据交互，如果浏览器端不支持WebSocket特性，那么`Socket.io`会主动进行降级，使用轮询等其他方式。以下为`Socket.io`兼容的几种不同机制：

- **Adobe® Flash® Socket**：通过将Flash插件嵌入到浏览器中而实现的一种Socket通信模式，由于是第三方实现，不在W3C规范内，并且绝大部分手机端浏览器不支持这种方式，所以在逐渐淘汰中。
- **AJAX Long Polling**： 所有浏览器都支持，定时向服务器端发送HTTP请求，兼容性广，但是会给服务器带来很大压力，并且不能保证数据的及时更新。
- **AJAX multipart streaming**： 在XMLHttpRequest对象上，使用某些浏览器(比如FireFox)支持的multi-part标志，Ajax请求发送给服务端并保持挂起状态，每次需要向客户端发送信息的时候，就寻找一个挂起的HTTP请求进行响应，并且所有的响应请求都会通过统一的连接来写入。
- **Forever Iframe**： 该技术设计了一个置于页面中的隐藏Iframe标签，该标签的src属性指向返回服务器端事件的servlet路径，每次在事件到达时，servlet写入并刷新一个新的script标签，该标签内部带有Javascript代码，iframe的内容被附加上这一script标签，标签中的内容就会得到执行，这种方式的缺点是连接和数据都是有浏览器通过HTML标签处理的，你无法知道连接何时在哪一端已经被断开了，并且Iframe标签在浏览器中将逐步被取消。
- **JSONP Polling**： JSONP轮询基本上与HTTP轮询一样，不同之处是JSONP可以发出跨域请求。

# WebSocket 简单demo(Socket.io实现)

简单客户端：
```
<head>
    <meta charset="UTF-8">
    <title>WebSocket Demo</title>
    <script src="//cdn.bootcss.com/socket.io/1.7.2/socket.io.min.js"></script>
</head>
<body>
    <h1>WebSocket Demo</h1>

    <script>
        var io = io.connect('http://127.0.0.1:3000');
        io.on('data', function (data) {
            console.log('on server data: ' + data);
        });
    </script>
</body>
</html>
```

简单服务端：
```
var app = require('koa')();
var server = require('http').createServer(app.callback());
var io = require('socket.io')(server);
io.on('connection', function (client) {
    client.emit('data', 'Hello WebSocket.');
});
server.listen(3000);
```

我们打开控制台，访问html页面，即可在控制台看到输出的`Hello WebSocket`字样，即表明服务端已经具备向客户端推送数据的能力。

# Nginx反向代理WebSocket

