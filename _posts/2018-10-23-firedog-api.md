---
layout: post
title: "FireDog 简要API"
date: 2018-10-22 13:46:57
img: workflow.jpg 
category: firedog
tags: [firedog]
---

Firedog 模块 API简要功能说明

<div class="divider"></div>

#### Event (fd_event)
事件系统，用于应用系统解耦，结构设计多样化，使用方式如下：
1. 创建派发器 Dispatcher = new(fd.EventDispatcher) ，通常一个系统一个
2. 添加事件类型 Dispatcher.AddEventListener("ghj",handler)
3. 实现handler函数，函数需要具备一个event参数
4. 唤醒事件 Dispatcher.Notify("ghj",nil) 参数可为任意类型

#### Family （fd_family）
协程运行单元管理，每个单元为一个room，使用方式如下：
1. 声明一个结构体作为room，并实现room必要的函数,如下
```
room = &fd.Room{}
	room.Name = "activity"
	room.OnInit = OnInit
	room.Handler = Handler
	room.Close = Close
```
2. 通过family 运行一个room，并向room发消息，如下
```
fd.Family.Start(room)

fd.Family.Cast("activity", "ghj", nil)

ret, err := fd.Family.Call("activity", "ttt", nil, 1000)
```
family cast 或者 call room的消息，都会在handler处理器中收到，并处理返回。

#### Timer (fd_timer)
定时器，提供定时执行任务，循环执行任务的服务，使用方式：
1. 确认Timer服务已经启动，可调用Timer.Start()启动服务，可通过配置启动服务
2. 设置执行函数和执行规则 fd.Timer.SetInterval(1 * time.Second,0,timer,nil)
3. 实现执行函数，函数将会安装规则定时调用


#### Log (log)
日志系统，使用时通过xml配置，在任何使用的地方导入log包即可使用