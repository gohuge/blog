---
layout: post
title: "WebSocket"
date: 2019-09-22 13:47:57
img: js-1.png
category: blog
tags: [blog]
---

### 一、内容概览

> WebSocket 的出现，使得浏览器具备了实时双向通信的能力。
> HTML5 开始提供的一种浏览器与服务器进行全双工通讯的网络技术，属于应用层协议。它基于 TCP 传输协议，并复用 HTTP 的握手通道。
> 在WebSocket协议中，数据是通过一系列数据帧来进行传输的。为了避免由于网络中介（例如一些拦截代理）客户端必须在它发送到服务器的所有帧中添加掩码（Mask）
> 无论WebSocket协议是否使用了TLS，帧都需要添加掩码

### 二、如何建立连接

** 握手实现 **

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```
告诉Apache、Nginx等服务器：注意啦，窝发起的是Websocket协议，快点帮我找到对应的助理处理;
```
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
#首先，Sec-WebSocket-Key 是一个Base64 encode的值，这个是浏览器随机生成的
 告诉服务器：泥煤，不要忽悠窝，我要验证尼是不是真的是Websocket助理。

Sec-WebSocket-Protocol: chat, superchat
#Sec_WebSocket-Protocol 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。
  简单理解：今晚我要服务A，别搞错啦~
Sec-WebSocket-Version: 13
#Sec-WebSocket-Version 是告诉服务器所使用的Websocket Draft（协议版本）
 在最初的时候，Websocket协议还在 Draft 阶段，各种奇奇怪怪的协议都有

```
服务端响应

```
HTTP/1.1 101 Switching Protocols
#这里开始就是HTTP最后负责的区域了，告诉客户，我已经成功切换协议啦~

Upgrade: websocket
Connection: Upgrade
#告诉客户端即将升级的是Websocket协议，而不是mozillasocket，lurnarsocket或者shitsocket。
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
#Sec-WebSocket-Accept 这个则是经过服务器确认，并且加密过后Sec-WebSocket-Key。
#服务器：好啦好啦，知道啦，给你看我的ID CARD来证明行了吧。

Sec-WebSocket-Protocol: chat
#后面的，Sec-WebSocket-Protocol 则是表示最终使用的协议。
```
** 客户端连接报文 **
```
<pre class="displaycode" style="margin-top: 0px; margin-bottom: 0px; white-space: pre-wrap; word-wrap: break-word; box-sizing: border-box;">GET /webfin/websocket/ HTTP/1.1
Host: localhost
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: xqBt3ImNzJbYqRINxEFlkg==
Origin: http://localhost:8080
Sec-WebSocket-Version: 13</pre>
```
可以看到，客户端发起的 WebSocket 连接报文类似传统 HTTP 报文，”Upgrade：websocket”参数值表明这是 WebSocket 类型请求，“Sec-WebSocket-Key”是 WebSocket 客户端发送的一个 base64 编码的密文，要求服务端必须返回一个对应加密的“Sec-WebSocket-Accept”应答，否则客户端会抛出“Error during WebSocket handshake”错误，并关闭连接。

服务端收到报文后返回的数据格式类似：

** WebSocket 服务端响应报文 **

```
<pre class="displaycode" style="margin-top: 0px; margin-bottom: 0px; white-space: pre-wrap; word-wrap: break-word; box-sizing: border-box;">HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: K7DJLdLooIwIG/MOpvWFB3y3FE8=</pre>
```
“Sec-WebSocket-Accept”的值是服务端采用与客户端一致的密钥计算出来后返回客户端的,“HTTP/1.1 101 Switching Protocols”表示服务端接受 WebSocket 协议的客户端连接，经过这样的请求-响应处理后，客户端服务端的 WebSocket 连接握手成功, 后续就可以进行 TCP 通讯了。

### 三、数据帧格式

WebSocket 客户端、服务端通信的最小单位是帧（frame），由 1 个或多个帧组成一条完整的消息（message）。

发送端：将消息切割成多个帧，并发送给服务端；
接收端：接收消息帧，并将关联的帧重新组装成完整的消息；

1. 数据帧格式概览

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-------+-+-------------+-------------------------------+
 |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
 |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
 |N|V|V|V|       |S|             |   (if payload len==126/127)   |
 | |1|2|3|       |K|             |                               |
 +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
 |     Extended payload length continued, if payload len == 127  |
 + - - - - - - - - - - - - - - - +-------------------------------+
 |                               |Masking-key, if MASK set to 1  |
 +-------------------------------+-------------------------------+
 | Masking-key (continued)       |          Payload Data         |
 +-------------------------------- - - - - - - - - - - - - - - - +
 :                     Payload Data continued ...                :
 + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
 |                     Payload Data continued ...                |
 +---------------------------------------------------------------+
```

2. 数据帧格式详解

针对前面的格式概览图，这里逐个字段进行讲解，如有不清楚之处，可参考协议规范，或留言交流。

FIN：1 个比特。

如果是 1，表示这是消息（message）的最后一个分片（fragment），如果是 0，表示不是是消息（message）的最后一个分片（fragment）。

RSV1, RSV2, RSV3：各占 1 个比特。

一般情况下全为 0。当客户端、服务端协商采用 WebSocket 扩展时，这三个标志位可以非 0，且值的含义由扩展进行定义。如果出现非零的值，且并没有采用 WebSocket 扩展，连接出错。

Opcode: 4 个比特。

操作代码，Opcode 的值决定了应该如何解析后续的数据载荷（data payload）。如果操作代码是不认识的，那么接收端应该断开连接（fail the connection）。可选的操作代码如下：

```
%x0：表示一个延续帧。当 Opcode 为 0 时，表示本次数据传输采用了数据分片，当前收到的数据帧为其中一个数据分片。
%x1：表示这是一个文本帧（frame）
%x2：表示这是一个二进制帧（frame）
%x3-7：保留的操作代码，用于后续定义的非控制帧。
%x8：表示连接断开。
%x8：表示这是一个 ping 操作。
%xA：表示这是一个 pong 操作。
%xB-F：保留的操作代码，用于后续定义的控制帧。
Mask: 1 个比特。
```

表示是否要对数据载荷进行掩码操作。从客户端向服务端发送数据时，需要对数据进行掩码操作；从服务端向客户端发送数据时，不需要对数据进行掩码操作。

如果服务端接收到的数据没有进行过掩码操作，服务端需要断开连接。

如果 Mask 是 1，那么在 Masking-key 中会定义一个掩码键（masking key），并用这个掩码键来对数据载荷进行反掩码。所有客户端发送到服务端的数据帧，Mask 都是 1。

**Payload length：数据载荷的长度，单位是字节。为 7 位，或 7+16 位，或 1+64 位。**

```
假设数 Payload length === x，如果
x 为 0~126：数据的长度为 x 字节。
x 为 126：后续 2 个字节代表一个 16 位的无符号整数，该无符号整数的值为数据的长度。
x 为 127：后续 8 个字节代表一个 64 位的无符号整数（最高位为 0），该无符号整数的值为数据的长度。
此外，如果 payload length 占用了多个字节的话，payload length 的二进制表达采用网络序（big endian，重要的位在前）。
```
**Masking-key：0 或 4 字节（32 位）**

所有从客户端传送到服务端的数据帧，数据载荷都进行了掩码操作，Mask 为 1，且携带了 4 字节的 Masking-key。如果 Mask 为 0，则没有 Masking-key。

备注：载荷数据的长度，不包括 mask key 的长度。

**Payload data：(x+y) 字节**

载荷数据：包括了扩展数据、应用数据。其中，扩展数据 x 字节，应用数据 y 字节。

扩展数据：如果没有协商使用扩展的话，扩展数据数据为 0 字节。所有的扩展都必须声明扩展数据的长度，或者可以如何计算出扩展数据的长度。此外，扩展如何使用必须在握手阶段就协商好。如果扩展数据存在，那么载荷数据长度必须将扩展数据的长度包含在内。

应用数据：任意的应用数据，在扩展数据之后（如果存在扩展数据），占据了数据帧剩余的位置。载荷数据长度 减去 扩展数据长度，就得到应用数据的长度。

3. 掩码算法
掩码键（Masking-key）是由客户端挑选出来的 32 位的随机数。掩码操作不会影响数据载荷的长度。掩码、反掩码操作都采用如下算法：

首先，假设：

original-octet-i：为原始数据的第 i 字节。
transformed-octet-i：为转换后的数据的第 i 字节。
j：为i mod 4的结果。
masking-key-octet-j：为 mask key 第 j 字节。
算法描述为： original-octet-i 与 masking-key-octet-j 异或后，得到 transformed-octet-i。

j = i MOD 4

transformed-octet-i = original-octet-i XOR masking-key-octet-j

### 四、数据传递

一旦 WebSocket 客户端、服务端建立连接后，后续的操作都是基于数据帧的传递。

WebSocket 根据opcode来区分操作的类型。比如0x8表示断开连接，0x0-0x2表示数据交互。

1. 数据分片
WebSocket 的每条消息可能被切分成多个数据帧。当 WebSocket 的接收方收到一个数据帧时，会根据FIN的值来判断，是否已经收到消息的最后一个数据帧。

FIN=1 表示当前数据帧为消息的最后一个数据帧，此时接收方已经收到完整的消息，可以对消息进行处理。FIN=0，则接收方还需要继续监听接收其余的数据帧。

此外，opcode在数据交换的场景下，表示的是数据的类型。0x01表示文本，0x02表示二进制。而0x00比较特殊，表示延续帧（continuation frame），顾名思义，就是完整消息对应的数据帧还没接收完。

2. 数据分片例子

第一条消息

FIN=1, 表示是当前消息的最后一个数据帧。服务端收到当前数据帧后，可以处理消息。opcode=0x1，表示客户端发送的是文本类型。

第二条消息

FIN=0，opcode=0x1，表示发送的是文本类型，且消息还没发送完成，还有后续的数据帧。
FIN=0，opcode=0x0，表示消息还没发送完成，还有后续的数据帧，当前的数据帧需要接在上一条数据帧之后。
FIN=1，opcode=0x0，表示消息已经发送完成，没有后续的数据帧，当前的数据帧需要接在上一条数据帧之后。服务端可以将关联的数据帧组装成完整的消息。
```
Client: FIN=1, opcode=0x1, msg="hello"
Server: (process complete message immediately) Hi.
Client: FIN=0, opcode=0x1, msg="and a"
Server: (listening, new message containing text started)
Client: FIN=0, opcode=0x0, msg="happy new"
Server: (listening, payload concatenated to previous message)
Client: FIN=1, opcode=0x0, msg="year!"
Server: (process complete message) Happy new year to you too!
```
### 五、连接保持 + 心跳

发送方 -> 接收方：ping (心跳Ping帧包含的操作码是0x9。)
接收方 -> 发送方：pong (心跳Ping帧包含的操作码是0xA。)
ping、pong 的操作，对应的是 WebSocket 的两个控制帧，opcode分别是0x9、0xA。

举例，WebSocket 服务端向客户端发送 ping，只需要如下代码（采用ws模块）

```
ws.ping('', false, true);
```
### 六、Sec-WebSocket-Key/Accept 的作用

前面提到了，Sec-WebSocket-Key/Sec-WebSocket-Accept在主要作用在于提供基础的防护，减少恶意连接、意外连接。

作用大致归纳如下：

避免服务端收到非法的 websocket 连接（比如 http 客户端不小心请求连接 websocket 服务，此时服务端可以直接拒绝连接）
确保服务端理解 websocket 连接。因为 ws 握手阶段采用的是 http 协议，因此可能 ws 连接是被一个 http 服务器处理并返回的，此时客户端可以通过 Sec-WebSocket-Key 来确保服务端认识 ws 协议。（并非百分百保险，比如总是存在那么些无聊的 http 服务器，光处理 Sec-WebSocket-Key，但并没有实现 ws 协议。。。）
用浏览器里发起 ajax 请求，设置 header 时，Sec-WebSocket-Key 以及其他相关的 header 是被禁止的。这样可以避免客户端发送 ajax 请求时，意外请求协议升级（websocket upgrade）
可以防止反向代理（不理解 ws 协议）返回错误的数据。比如反向代理前后收到两次 ws 连接的升级请求，反向代理把第一次请求的返回给 cache 住，然后第二次请求到来时直接把 cache 住的请求给返回（无意义的返回）。
Sec-WebSocket-Key 主要目的并不是确保数据的安全性，因为 Sec-WebSocket-Key、Sec-WebSocket-Accept 的转换计算公式是公开的，而且非常简单，最主要的作用是预防一些常见的意外情况（非故意的）。
强调：Sec-WebSocket-Key/Sec-WebSocket-Accept 的换算，只能带来基本的保障，但连接是否安全、数据是否安全、客户端 / 服务端是否合法的 ws 客户端、ws 服务端，其实并没有实际性的保证。

### 七、相关链接

RFC6455：websocket 规范

server-example

编写websocket 服务器

Talking to Yourself for Fun and Profit（含有攻击描述）

What is Sec-WebSocket-Key for?

Why are WebSockets masked?

How does websocket frame masking protect against cache poisoning?

What is the mask in a WebSocket frame?