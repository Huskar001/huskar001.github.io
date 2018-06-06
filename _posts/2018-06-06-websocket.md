---
layout: post
title:  "WebSocket相关"
date:   2018-06-06 09:22:00 +0800
categories: web
tag: web
---
* content
{:toc}








# websocket

初次接触 WebSocket 的人，都会问同样的问题：我们已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？
答案很简单，因为 HTTP 协议有一个缺陷：通信只能由客户端发起。同上一篇一样的目的，聊天，需要推送，轮询的效率太低。

WebSocket 协议在2008年诞生，2011年成为国际标准。所有浏览器都已经支持了。
它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。

## 特点

其他特点包括：

（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

（6）协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。
ws://example.com:80/some/path

## demo

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


## API

WebSocket 客户端的 API 如下。

4.1 WebSocket 构造函数
WebSocket 对象作为一个构造函数，用于新建 WebSocket 实例。


var ws = new WebSocket('ws://localhost:8080');
执行上面语句之后，客户端就会与服务器进行连接。

webSocket.readyState

readyState属性返回实例对象的当前状态，共有四种。
CONNECTING：值为0，表示正在连接。
OPEN：值为1，表示连接成功，可以通信了。
CLOSING：值为2，表示连接正在关闭。
CLOSED：值为3，表示连接已经关闭，或者打开连接失败。
需要注意的是 状态需要即时的去自己判断 没有回调

除了动态判断收到的数据类型，也可以使用binaryType属性，显式指定收到的二进制数据类型。

	// 收到的是 blob 数据
	ws.binaryType = "blob";
	ws.onmessage = function(e) {
	  console.log(e.data.size);
	};

	// 收到的是 ArrayBuffer 数据
	ws.binaryType = "arraybuffer";
	ws.onmessage = function(e) {
	  console.log(e.data.byteLength);
	};
实例对象的send()方法用于向服务器发送数据。
ws.send('your message');
发送 Blob 对象的例子。

	var file = document
	  .querySelector('input[type="file"]')
	  .files[0];
	ws.send(file);

发送 ArrayBuffer 对象的例子。

	// Sending canvas ImageData as ArrayBuffer
	var img = canvas_context.getImageData(0, 0, 400, 320);
	var binary = new Uint8Array(img.data.length);
	for (var i = 0; i < img.data.length; i++) {
	  binary[i] = img.data[i];
	}
	ws.send(binary.buffer);

如果想兼容低版本的浏览器 可以考虑使用sockjs 在无法使用websocket推送的时候 使用轮询方案。

欢迎交流 feiqiyun@126.com

