#  websocket

WebSocket原理：

http://www.ruanyifeng.com/blog/2017/05/websocket.html

https://www.runoob.com/html/html5-websocket.html

https://www.zhihu.com/question/20215561



library库

重连库：https://github.com/joewalnes/reconnecting-websocket

https://www.runoob.com/html/html5-websocket.html

node+websocket：https://zhuanlan.zhihu.com/p/127889084



WebSocket-Node：https://github.com/theturtle32/WebSocket-Node

WebSocket客户端 和 服务器端Node的实现 

概述 

这是WebSocket协议版本8和Node版本13的(主要是)纯JavaScript实现。在“测试/脚本”文件夹中有一些客户端和服务器应用程序示例，它们实现了各种互操作性测试协议。

文档

[You can read the full API documentation in the docs folder.](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/index.md)

WebSocket-Node包括客户端和服务器功能，分别通过WebSocketClient和WebSocketServer提供。一旦建立了连接，无论您充当客户机还是服务器，用于发送和接收消息的API都是相同的。

单击下面的类来查看它的API文档：

- [WebSocketClient](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketClient.md)
- [WebSocketConnection](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketConnection.md)
- [WebSocketFrame](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketFrame.md)
- [WebSocketRequest](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketRequest.md)
- [WebSocketServer](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketServer.md)
- [W3CWebSocket](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/W3CWebSocket.md)



浏览器支持情况

所有的当前浏览器都充分支持：

- Firefox 7-9 (Old) (Protocol Version 8)
- Firefox 10+ (Protocol Version 13)
- Chrome 14,15 (Old) (Protocol Version 8)
- Chrome 16+ (Protocol Version 13)
- Internet Explorer 10+ (Protocol Version 13)
- Safari 6+ (Protocol Version 13)

安装 

```sh
$ npm install websocket
```

然后在你的代码中

```js
var WebSocketServer = require('websocket').server;
var WebSocketClient = require('websocket').client;
var WebSocketFrame  = require('websocket').frame;
var WebSocketRouter = require('websocket').router;
var W3CWebSocket = require('websocket').w3cwebsocket;
```

使用案例

服务器使用案例

这里有一个小的案例，展示了服务器发送utf-8或二进制的数据

```js
#!/usr/bin/env node
var WebSocketServer = require('websocket').server;
var http = require('http');

var server = http.createServer(function(request, response) {
    console.log((new Date()) + ' Received request for ' + request.url);
    response.writeHead(404);
    response.end();
});
server.listen(8080, function() {
    console.log((new Date()) + ' Server is listening on port 8080');
});

wsServer = new WebSocketServer({
    httpServer: server,
    // You should not use autoAcceptConnections for production
    // applications, as it defeats all standard cross-origin protection
    // facilities built into the protocol and the browser.  You should
    // *always* verify the connection's origin and decide whether or not
    // to accept it.
    autoAcceptConnections: false
});

function originIsAllowed(origin) {
  // put logic here to detect whether the specified origin is allowed.
  return true;
}

wsServer.on('request', function(request) {
    if (!originIsAllowed(request.origin)) {
      // Make sure we only accept requests from an allowed origin
      request.reject();
      console.log((new Date()) + ' Connection from origin ' + request.origin + ' rejected.');
      return;
    }
    
    var connection = request.accept('echo-protocol', request.origin);
    console.log((new Date()) + ' Connection accepted.');
    connection.on('message', function(message) {
        if (message.type === 'utf8') {
            console.log('Received Message: ' + message.utf8Data);
            connection.sendUTF(message.utf8Data);
        }
        else if (message.type === 'binary') {
            console.log('Received Binary Message of ' + message.binaryData.length + ' bytes');
            connection.sendBytes(message.binaryData);
        }
    });
    connection.on('close', function(reasonCode, description) {
        console.log((new Date()) + ' Peer ' + connection.remoteAddress + ' disconnected.');
    });
});
```

客户端案例

这是一个简单的示例客户端，它将打印它在控制台上收到的任何utf-8消息，并定期发送一个随机数。

下面代码示例是node.js客户端，而不是在浏览器中

```js
#!/usr/bin/env node
var WebSocketClient = require('websocket').client;

var client = new WebSocketClient();

client.on('connectFailed', function(error) {
    console.log('Connect Error: ' + error.toString());
});

client.on('connect', function(connection) {
    console.log('WebSocket Client Connected');
    connection.on('error', function(error) {
        console.log("Connection Error: " + error.toString());
    });
    connection.on('close', function() {
        console.log('echo-protocol Connection Closed');
    });
    connection.on('message', function(message) {
        if (message.type === 'utf8') {
            console.log("Received: '" + message.utf8Data + "'");
        }
    });
    
    function sendNumber() {
        if (connection.connected) {
            var number = Math.round(Math.random() * 0xFFFFFF);
            connection.sendUTF(number.toString());
            setTimeout(sendNumber, 1000);
        }
    }
    sendNumber();
});

client.connect('ws://localhost:8080/', 'echo-protocol');
```

使用W3CWebSocket API的客户端示例

和上面实例相同，但是用在W3C WebSocket API的中

```js
var W3CWebSocket = require('websocket').w3cwebsocket;

var client = new W3CWebSocket('ws://localhost:8080/', 'echo-protocol');

client.onerror = function() {
    console.log('Connection Error');
};

client.onopen = function() {
    console.log('WebSocket Client Connected');

    function sendNumber() {
        if (client.readyState === client.OPEN) {
            var number = Math.round(Math.random() * 0xFFFFFF);
            client.send(number.toString());
            setTimeout(sendNumber, 1000);
        }
    }
    sendNumber();
};

client.onclose = function() {
    console.log('echo-protocol Client Closed');
};

client.onmessage = function(e) {
    if (typeof e.data === 'string') {
        console.log("Received: '" + e.data + "'");
    }
};
```

请求路由案例

对于请求路由的使用案例，查看libwebsockets-test-server.js中的test文件夹。

