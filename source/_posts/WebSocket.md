---
title: WebSocket
date: 2018-07-20 15:37:57
tags: [js原生,node,WebSocket]
categoies: [js原生,node,WebSocket]
---

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。
<!--more-->
**特点**
1. 建立在 TCP 协议之上，服务器端的实现比较容易。
2. 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
3. 数据格式比较轻量，性能开销小，通信高效。
4. 可以发送文本，也可以发送二进制数据。
5. 没有同源限制，客户端可以与任意服务器通信。
6. 协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。
url格式
```javascript
  ws://example.com:80/some/path
```
## 客户端实例
```javascript
  var ws = new WebSocket("wss://echo.websocket.org");

  ws.onopen = function(evt) { 
    console.log("Connection open ..."); 
    ws.send("Hello WebSockets!");
  };

  ws.onmessage = function(evt) {
    console.log( "Received Message: " + evt.data);
    ws.close();
  };

  ws.onclose = function(evt) {
    console.log("Connection closed.");
  };      
```
## 客户端API
### WebSocket 构造函数
`WebSocket` 对象作为一个构造函数，用于新建 `WebSocket` 实例。
```javascript
  var ws = new WebSocket('ws://localhost:8080');
```
执行上面语句之后，客户端就会与服务器进行连接。

### webSocket.readyState
`readyState` 属性返回实例对象的当前状态，共有四种。
* `CONNECTING`：值为0，表示正在连接。
* `OPEN`：值为1，表示连接成功，可以通信了。
* `CLOSING`：值为2，表示连接正在关闭。
* `CLOSED`：值为3，表示连接已经关闭，或者打开连接失败。 
```javascript
  switch (ws.readyState) {
    case WebSocket.CONNECTING:
      // do something
      break;
    case WebSocket.OPEN:
      // do something
      break;
    case WebSocket.CLOSING:
      // do something
      break;
    case WebSocket.CLOSED:
      // do something
      break;
    default:
      // this never happens
      break;
  }
```

### webSocket.onopen
实例对象的`onopen`属性，用于指定连接成功后的回调函数。
```javascript
  ws.onopen = function () {
    ws.send('Hello Server!');
  }
```
如果要指定多个回调函数，可以使用`addEventListener`方法。
```javascript
  ws.addEventListener('open', function (event) {
    ws.send('Hello Server!');
  });
```

### webSocket.onclose
实例对象的onclose属性，用于指定连接关闭后的回调函数。
```javascript
  ws.onclose = function(event) {
    var code = event.code;
    var reason = event.reason;
    var wasClean = event.wasClean;
    // handle close event
  };

  ws.addEventListener("close", function(event) {
    var code = event.code;
    var reason = event.reason;
    var wasClean = event.wasClean;
    // handle close event
  });
```

### webSocket.onmessage
实例对象的`onmessage`属性，用于指定收到服务器数据后的回调函数。
```javascript
  ws.onmessage = function(event) {
    var data = event.data;
    // 处理数据
  };

  ws.addEventListener("message", function(event) {
    var data = event.data;
    // 处理数据
  });
```
### webSocket.send()
实例对象的`send()`方法用于向服务器发送数据。
```javascript
  ws.send('your message');
```

### webSocket.onerror
```javascript
  socket.onerror = function(event) {
    // handle error event
  };

  socket.addEventListener("error", function(event) {
    // handle error event
  });
```

