---
layout: post
title: "【ouhot】配置说明"
date: 2018-08-21 16:15:57
category: ouhot
tags: [ouhot]
---

- 配置是项目中一种解耦和操控便捷化的手段，配置描述程序运行参数和运行过程，

- 以下为本程序的功能配置说明。
&nbsp;

##### Net 网络配置
```
{start,{ouhot_server,start,[{6000,[ouhot_connect,ouhot_coder]}]}}.
```
##### Node 节点配置
```
{node,{slave_1,[{ip,'192.168.1.1'},{username,"root"},{password,"Jzyx2016!"}]}}.
{node,{slave_2,[{ip,'192.168.1.1'},{username,"root"},{password,"Jzyx2016!"}]}}.
```
##### App 应用插件配置,应用配置的较少，所以可以尽可能详细点
```
%% 配置网络服务组件
{app,{cowboy,{application, 'cowboy', [
    {description, "Small, fast, modern HTTP server."},
    {vsn, "2.1.0"},
    {modules, ['cowboy','cowboy_app','cowboy_bstr','cowboy_children',
        'cowboy_clear','cowboy_clock','cowboy_compress_h','cowboy_constraints',
        'cowboy_handler','cowboy_http','cowboy_http2','cowboy_iolists',
        'cowboy_loop','cowboy_metrics_h','cowboy_middleware','cowboy_req',
        'cowboy_rest','cowboy_router','cowboy_static','cowboy_stream',
        'cowboy_stream_h','cowboy_sub_protocol','cowboy_sup','cowboy_tls',
        'cowboy_tracer_h','cowboy_websocket']},
    {registered, [cowboy_sup,cowboy_clock]},
    {applications, [kernel,stdlib,crypto]},
    {mod, {cowboy_app, []}},
    {env, [{layer,[gateway]}]}
]}}}.
```
##### Port 配置
```
{port,{"cmd",[
  {template_port,transmit}
],1000}}.
```
##### DB 配置
* db接口直接使用mnesia，用户层是用的时候需要进行一次封装，如：db:read
* 多数情况下底层封装后，上层逻辑仍然会进行一次封装，所以底层不封db操作
```
% reacord_name = role
{mnesia,{role,[
  {attributes, [name,age,sex]},{type, set}
]}}.
```

##### Timer 定时器配置
```
%% 配置一个常驻定时器
{timer,{timer_name,[
  {module,function,[]},
  %% 按日期执行，到达指定日期执行
  {time,[{y,m,d},{h,m,s}]}，
  %% 按间隔执行，每多少秒执行一次
  {time,10}，
  %% 按每日执行，每日几时执行
  {time,{h,m,s}},
  %% 按每周执行，每日几时执行
  {time,{d,h,m,s}},
]}}.
```

##### Event 事件配置
```
%% 事件配置 ：事件,MFA,最后一个参数大于0则表示异步执行
{event,{test_event,[{template_app,listener,[abc]},1000]}}.
```

##### Logger 日志配置
```
%% 名称为info的日志进程，日志文件格式为info.date.extension
%% limit为限制生成的数量、interval是周期（分钟），format为日志解析格式，extension为文件后缀
{logger,{info,[{limit,5},{interval,1440},{format,"aa"},{dir,"/log"},{extension,".log"}]}}.
```
