---
layout: post
title: "【ouhot】开发模型"
date: 2018-08-21 16:05:57
category: ouhot
tags: [ouhot]
---

 开发模型，通俗来讲就是一种观念，一种程序范式

<div class="divider"></div>

#### 一、目录结构

1. 代码文件目录结构
- boot 应用启动相关的文件，运行环境目录
- deps
  - ebin 框架组件目录
  - apps 应用目录（项目代码位置）
  - 优先解析执行的文件夹，存放 .cfg文件
  - .ebin 项目源码编译后的生成目录
  - game 游戏项目目录
  - 存放游戏的各个功能模块
  - template 模板代码目录
  - proto 通信协议目录
  - test 测试代码目录
  - tool 工具目录
2. 功能目录
- game目录下面每个功能文件夹
  - src 源码目录
  - include 头文件
  - xxx.cfg 功能配置
3. 下面给一张template的功能图  
![template](/assets/template.png)

#### 二、命名规则

1. 功能启动，关闭，入口模块 （module_app）
2. 功能数据库操作，缓存操作模块 （module_db）
3. 功能C层封装模块 （module_nif）
4. 功能通信端口（module_port）
5. 功能库支持（module_lib）
6. 功能消息监听（module_listener）
7. 功能相关配置（module.cfg）

#### 三、通信说明

后台通过cfg配置消息路由至哪个模块，模块返回给前台的消息直接传protobuf生成的记录文件即可。
程序读取到的cmd即是protobuf文件的模块名,下面一个通信接收的模型；
返回{reply,Msg}消息立即返回（同步通信业务），{noreply,[]}消息不返回（异步通信业务）

<div class="divider"></div>

```
-module(user_port).
-description("user_port").
-author({gouhj, 'gohuge@qq.com'}).

%%%===============================INCLUDE================================
-include("../../../proto/src/login.hrl").

%%%================================EXPORT================================
-export([login/4]).

login(_A,Attr,Connect,#'LoginInfo'{id = Id}=Msg)->

    log:debug(?MODULE,?LINE,'LoginInfo',[_A,Attr,Connect,Id,Msg]),

    {reply,Msg}.

```
