---
layout: post
title: HTTP和HTTPS那点事
date: 2019-08-04 11:47:57
img: js-1.jpg
category: blog
tags: [blog]
---

## HTTP简介
HTTP（超文本传输协议）协议是属于应用层的协议，基于请求-响应模式的无状态的应用层协议，基于TCP的连接方式，HTTP1.1版本给出一种持续连接的方式Keep-alive（长连接）。

它指定了客户端可能发送给服务器什么样的消息以及得到什么样的响应。请求和响应消息的头以ASCII码形式给出；而消息内容则具有一个类似MIME的格式。这个简单模型是早期Web成功的有功之臣，因为它使得开发和部署是那么的直截了当。

<br/>

## HTTP特点
1. 支持客户/服务器模式
2. 简单高效 （客户端向服务器请求接口的时候只需发送请求方法【GET、POST、PUT、HEAD等】和路径，基于这种简单，使得它很高效）
3. 灵活（允许传输任意类型的数据对象）
4. 无连接（每次连接只处理一个请求，服务器收到客户端的请求，处理完请求，收到客户端的应答后，断开连接）
5. 无状态（协议对事物的处理没有记忆能力，后续处理如果需要前面的信息，则必须再次传送）

## HTTP请求结构
![avatar](/assets/img/http_req.png)
## HTTP响应结构
![avatar](/assets/img/http_pos.png)
## 请求响应的步骤
1. 客户端连接Web服务器：浏览器与Web服务器的HTTP端口（80）建立TCP套接字连接
2. 客户端向服务器发送HTTP请求（请求报文）
3. 服务器接收请求，返回HTTP响应（服务器将客户端请求的内容写到TCP套接字，由客户端读取）
4. 释放连接TCP连接
5. 客户端解析HTML内容

## 状态码普及
1XX：指示信息（请求已接受，继续处理）

2XX：成功（请求成功接收）

3XX：重定向（完成请求需要更进一步操作）

4XX：客户端错误（请求有语法错误或请求无法实现）

5XX：服务端错误（可能是服务器接口出bug，服务器不能实现合法请求）

## GET和POST的区别
HTTP报文层：GET请求信息位于URL、POST

数据库层面：GET符合幂等性（对数据库的一次操作或多次操作获得的结果是一致的）和安全性（没有改变数据库中的数据），因为GET操作对数据库是查询操作，所以不会改变数据库；POST不符合（POST会改变数据库）

其他：get请求参数可以被浏览器缓存和存储（比如收藏啥的），POST不行

## Cookie和Session
HTTP请求是无状态的，所以Web可以通过Cookie和Session来保存用户的唯一标示

Cookie：服务端发给客户端的特殊信息（用户登录，服务端响应的时候把Cookie放在响应头里），以文本文件形式存在客户端的某处，客户端每次向服务器发送请求时都会带上这些信息（放在请求头），然后服务器会解析请求头里的Cookie从而动态的获取对应用户的信息返回。

Session：和Cookie不同，被保存在服务器端。程序要为某个请求创建Session的时候。

流程：服务器先检查客户端请求头是否包含session id，若包含，说明之前创建过Session，服务器会根据Session id吧Session找出来拿出里面的用户信息。没有Session id就创建一个Session，并生成一个唯一的Session id，Session id会发给客户端保存。

Session实现方式：

服务器生成Session id会通过Cookie发给客户端，客户端下次把Session id在Cookie里带上。
URL回写方式，把Session id写在URL里。

## HTTPS（超文本传输安全协议）
以计算机网络通信为目的的传输协议。在HTTP的基础上加上一个SSL层来保护交换数据隐私和完整性，也就是安全版HTTP

## SSL（Security Socket Layer）安全套接层
为网络通信提供安全及数据完整性的一种安全协议，是操作系统对外的API，SSL3.0后更名为TLS，它通过身份验证和数据加密保证网络通信的安全和数据的完整性。

HTTP协议传输的数据其实就是一种形式的“裸奔”，随便开个抓包工具，请求响应看的一清二楚。那么HTTPS安全在哪里？

## HTTPS数据传输过程

在数据交换前会进行一次握手确定信息
1. 客户端将支持的加密信息算法发到服务器
2. 服务器选择一种客户端支持的加密算法，SSL 协议的版本号，加密算法的种类，随机数以及其他相关信息，以证书的形式（证书发布CA机构、证书有效期公钥、所有者、签名等）发回给客户端
3. 客户端（浏览器）验证证书的合法性：
4. 证书是否过期，发行服务器证书的CA 是否可靠，发行者证书的公钥能否正确解开服务器证书的“发行者的数字签名”，服务器证书上的域名是否和服务器的实际域名相匹配。
5. 客户端会随机生成一串密码结合证书公钥使用约定好的加密算法加密，用约定好的HASH计算握手消息，并用生成的一串密码对其加密发送给服务器
6. 服务器使用私钥解密信息确定密码，使用密码解密握手消息验证HASH是否和客户端一致，使用密码加密响应信息发回客户端。
7. 浏览器解密响应信息，对消息验证，握手结束，之后用之前生成的随机密码和对称算法对交互信息加密。

## HTTP与HTTPS的区别
1. HTTPS密文传输，HTTP明文传输
2. HTTPS使用443端口，HTTP使用80端口
3. HTTPS是安全版的HTTP = HTTP  加密  认证  完整性保护
4. 连接方式不同
5. HTTPS需要向CA申请证书

## HTTPS基础上的更安全方案
HSTS（HTTP Strict Transport Security）国际互联网工程组织IETF正在推行一种新的Web安全协议
HSTS的作用是强制客户端（如浏览器）使用HTTPS与服务器创建连接。
因为在重定向HTTP跳HTTPS的时候，用户其实还是使用了HTTP有一定危险。