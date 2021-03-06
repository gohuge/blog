---
layout: post
title: FireDog介绍
date: 2018-10-22 13:47:57
description: You’ll find this post in your
img: workflow.jpg 
category: firedog
tags: [firedog]
---
框架支持大量网络IO、密集性运算和存储服务，开发效率高，复杂度低，招聘容易，支持一下行业特性.

## 行业现状

1. 随时开服，一天多服
2. 业务收尾，多服合并
3. 疯狂灌人，配置逐步增大
4. 快速回滚，时间点局部恢复

##  设计预期

* 高性能、高容灾、灰度版本管理、热更新、监控
* 灵活调度、分布式、去中心化
* 高扩展性、云计算、超大容量、动态扩容
* 数据安全、快速备份还原、每日增量、每周快照、实时写入

##  框架元件

* 网络：网络通信包含http、tcp、websocket、udp 协议和消息路由
* 配置启动：基础组件运行、流程控制、控制台调用
* 数据存储：数据库操作、本地化、数据散列、灵活转移
* 分布式：动态管理、灵活调度、云计算和结果归并、脑裂、主从竞争

## 其他服务

* 日志服务：日志分类输入、日志转储、日志筛选
* 缓存服务：本地缓存、独立缓存
* 全局服务：全局管控服务，中控
