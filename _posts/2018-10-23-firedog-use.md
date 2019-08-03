---
layout: post
title: "FireDog使用说明"
date: 2018-10-22 13:46:57
img: we-in-rest.jpg
category: firedog
tags: [firedog]
---

firedog 使用说明

<div class="divider"></div>

##### Download & go get
1. get依赖库在库的readme中有描述需要get的相关库
2. get完成后，进入firedog目录 go run ./src/main.go 程序运行
3. 程序运行后，会启动控制台，exit 退出控制台（控制台调用还在增设中）
4. 程序启动后，xml会解析xml中配置的内容
5. xml中的配置解析完毕后，会将定义和声明保持至内存，然后执行init的内容
6. init中的内容执行完毕，将开始执行mfa

程序启动完毕！


##### 如何使用框架写构建应用
1. 框架中app目录为应用代码的目录，res为资源文件，conf为配置文件，log为日志文件
2. 在app中写的程序，可以自己到处调用实现服务启动，也可以通过xml配置调用启动服务
3. 如果要使用xml配置来启动服务，需要向使用util.Module 注册模块，app目录中有相关实例