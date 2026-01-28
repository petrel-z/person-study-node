是什么？
它和HTTP协议类似，都是基于TCP协议的
HTTP和WebSocket是网络通讯协议

这两种协议有什么区别？
HTTP1.1是半双工的，在同一时刻只能一方发送数据给另一方。HTTP协议是一种‘请求-响应’模式，只能通过客户端发送http请求，接受服务端http响应，以此来传输数据，服务端不能主动给客户端发送信息。
WebSocket是全双工的，在同一时刻能同时发送数据，一旦客户端和服务端建立了TCP连接，二者就能时刻保持通讯状态。

![[Pasted image 20250906161320.png]]
举个例子：
我们在访问一个网址时，客户端需要向目标服务器请求网页资源，此时客户端向服务器发送http请求，发送之前二者需要建立TCP连接，服务端接受到请求后，做出响应，返回给客户端所需的资源，之后二者关闭TCP连接。如果再需要向服务端请求资源，就需要重新建立TCP连接，并发送http请求，服务端响应完毕后，二者关闭TCP连接。
这是HTTP的使用场景，有一些场景，例如扫码登录。
当用户扫码后，用户信息传递给服务端，由于HTTP协议1.1不支持服务端主动向客户端发送数据，所以客户端需要不断向服务端发送查询请求，来询问服务器用户是否扫码，这实现方式就是轮询。
短轮询：客户端每隔一定时间间隔向服务端发送查询请求，例如每隔1秒发送查询请求。
长轮询：客户端发送查询请求后，该请求会保持挂起状态一段时间（例如保持30秒），如果用户在这30秒内还没有扫码，服务端就返回一个空响应，客户端会重新发送查询请求。
长轮询避免了在用户扫码前发送过多的查询请求，消耗过多的资源，又能保证实时处理用户操作，从而实现用户无感知享受服务器推送功能。
在这种场景下，WebSocket更适配。

那服务端如何判断本次的协议是HTTP还是WebSocket呢？
通过TCP建立客户端和服务端的连接后，客户端发送http请求
完整http请求包含必要的请求行、请求头、请求体
请求行：
1. 请求方法 （GET/POST/PUT/DELETE...)
2. 请求路径url
3. 协议邦本
请求头字段：
Host：目标服务器的域名和端口（必需）
User-Agent：客户端（浏览器 / APP）的身份信息
Content-Type：声明请求体的数据格式
Authorization：身份验证信息（token）
...
请求体：
当需要向服务端发送数据时存在，最常用的就是以json格式传送数据
```json
{
  "username": "admin",
  "password": "123456",
}
```
WebSocket在建立TCP连接后，需要发送一个特殊的http请求
这个http请求中的请求头比较特殊，它必须包含以下特定字段
Upgrade: websocket
**关键升级字段**：告诉服务器 “我想把协议从 HTTP 升级到 WebSocket”，值必须是 `websocket`（大小写不敏感，但常规用小写）。
Connection: Upgrade
**配合 Upgrade 字段**：告诉服务器 “这是一个协议升级请求，不要按普通 HTTP 响应处理”，值必须包含 `Upgrade`
Sec-WebSocket-Key：
**安全验证字段**：客户端生成的随机的base64编码的字符串
Sec-WebSocket-Version：指定 WebSocket 协议版本

服务端接受后，处理该请求并返回响应。
如果请求成功，服务端返回101状态码，表示协议切换成功，从http协议切换为websocket协议
并返回Sec-WebSocket-Accept响应字段，
该字段是服务器根据客户端的 `Sec-WebSocket-Key` 计算得出的值，客户端会验证该值是否正确，若不正确则断开连接（防止伪造响应）。
完成以上操作后，就成功连接websocket了，客户端和服务端就可以进行实时的双向通讯了。

下面我们通过一个demo，用websocket完成一个实时接收服务端返回的数据的小案例
1. 安装node.js
2. 在编辑器里新建文件夹
```bash
# 初始化项目
npm init -y 
# 安装ws
npm install ws
```
3. 新建server.js文件
```js
// 引入ws库，创建websocket服务器
const WebSocket = require('ws');
// 监听8080端口
const ws = new WebSocket.Server({ port: 8080 });
  
// 监听用户连接
ws.on('connection', (ws) => {
	console.log('用户连接成功');

	let timer = setInterval(() => {

	ws.send(createRandom());

}, 1000);

  

// 监听客户端发送的信息

ws.on('message', (message) => {

console.log('用户信息：', message.toString());

ws.send(message.toString());

})

  

// 监听客户端是否断开连接

ws.on('close', () => {

console.log('用户断开连接');

})

})

  

// 生成随机数

const createRandom = () => {

let num = Math.floor(Math.random()*100);

return num;

}

  

console.log('WebSocket 服务器已启动，端口 8080');
```
4. 新建index.html文件
5. 新建websocket.js文件，实现客户端与服务端的连接
6. 最后附上index.css文件

我们就实现啦
![[Pasted image 20250906165450.png]]