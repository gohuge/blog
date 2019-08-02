---
layout: post
title: "【ouhot】启动运行"
date: 2018-08-25 10:53:57
category: ouhot
tags: [ouhot]
---

- 通过boot目录的可执行文件start启动程序，也可以自定义里面的各个参数；

- 程序启动后，会先解析配置启动服务插件，再启动自身功能。
&nbsp;

#### 启动说明
1. 第一个节点启动后默认为总控
2. 总控会根据配置或设置去启动远端节点
3. 启动其他节点后会把代码推送给远端节点
4. 总控节点会通知远端节点正式运行
5. 远端节点启动ouhot服务
6. ouhot根据远端节点属性开始建立本地环境
7. ouhot为该节点启动相应服务

#### 环境构建
1. 启动后会在对应计算机创建slave目录
2. 将运行环境复制至slave目录为runtime
3. 在slave目录创建对应节点名称的文件夹
4. 在对接节点名称文件夹创建disk目录和temp目录

#### 环境限制
- 进程限制，所以连接数也受到了限制
- erlang:system_info(process_limit).
262144
- 如果一个连接构建一个ets，ets也有限制
- cowboy的默认连接数是1024
- 当然可以设置为不限制{max_connections, infinity}

```
才26万个，不够用。在启动脚本(start.sh)处添加允许创建的最大线程支持：

#!/bin/sh
erl +K true +P 10240000 -sname testserver -pa ebin -pa deps/*/ebin -s htmlfilesimple\
-eval "io:format(\"Server start with port 8000 Success!~n\")."
脚本启动后现在在erl shell中测试一下：

erlang:system_info(process_limit).
16777216
数量完全够用了。
```

```
开启erlang的epoll属性
+K true | false 是否开启kernel poll，就是epoll；
不开启，测试过程中，在内存完好情况下，经常会有连接失败情况。
```
