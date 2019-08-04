---
layout: post
title: "Erlang 容器性能测试"
date: 2018-10-21 13:47:57
img: workflow.jpg 
category: blog
tags: [blog]
---

erlang 进程字典、ets、tuple、list的性能情况。

<div class="divider"></div>

##### 一、List和Ets的性能对比

```
test1()->
  Time1 = z_lib:now_millisecond(),
  datas(100000.1231,1,[]),
  Time2 = z_lib:now_millisecond(),
  io:format("recive ret : list use_time = ~p ~n",[Time2-Time1]),
  ets:new(ghj,[named_table]),
  datas2(100000.1231,1,0),
  Time3 = z_lib:now_millisecond(),
  io:format("recive ret : ets use_time = ~p ~n",[Time3-Time2]).

datas2(Count,Index,Amount) when Amount < Count ->
  Data = {float_to_list(Count*100*(Amount+1)+Index), Count*100+Index, 1, z_lib:now_millisecond()},
  %% 可换成进程字典
  ets:insert(ghj,[Data]),
  datas2(Count,Index,Amount+1);
datas2(_,_,R)-> R.
```

这个代码中，我们认为构建数据所产生的时间开销都是一致的。
##### 结论:
list在比较大的数据操作情况下的操作性能开销巨大，tuple类似在2w+后急剧下降
- 退recive ret : list use_time = 135833 
- recive ret : ets use_time = 335 
- recive ret : dist use_time = 149 

我之前研究极致游戏引擎代码的时候，就发现其select函数客户端和服务器的结果采用tuple组装，在数据变大后性能急剧下滑。

所以后面我在进行大数据操作的时候，通常开辟一个临时ets表，在数据进程装载数据后，将其归属（give_away）直接转交给使用进程。

##### 二、进程消息发送与接收
- 同节点消息传输，1000W条简单消息发送接收时间3~4秒，200W/s受消息长度影响
- 不同节点，消息传输太大可能会影响net_kernet的ticker导致io阻塞，节点失联可以通过这个函数尝试sys:resume(net_kernel). [link](https://groups.google.com/forum/#!searchin/erlang-programming/node$20split|sort:date/erlang-programming/4V9jxL292ug/LVTwN8aaBAAJ)


```
test:test_proc_msg(10000000, {"test","test"}).
{3731000,ok}
test:test_proc_msg(10000000, {"testtesttesttest"}).
{4428000,ok}
```

##### 三、Ets的查询性能
ets查询性能
```
3> test:test_ets(10000000, "test").    
{insert,{17341000,ok}},{lookup,{1717000,ok}}
```
- 1000W次写入花了17.3S，平均57W/S，较多的是0.4/us 
- 1000W次查询花了1.7S，平均580W/S，约为写入时间的十分之一

##### 四、Match和Iterator的速度
首先测试match，需要ets:fun2ms这里注意需要导入一个库,官方是这样说的：

The parse transform is provided in the ms_transform module and the source must include file ms_transform.hrl in STDLIB for this pseudo function to work. Failing to include the hrl file in the source results in a runtime error, not a compile time error. The include file is easiest included by adding line **-include_lib("stdlib/include/ms_transform.hrl").** to the source file.

通过测试，100w，select用时1480ms,itorator用时901ms,match用时908.
怎么会这样我泪流满面,总以为match和select是nif要快点！
```
lookup ets=ghj,size=1000001
MSResult: use_time = 1491 ,size=1000001 
 ItoratorResult: use_time = 901 ,size=1000001 
 MatchResult: use_time = 908 ,size=1000001 
 ok
```

