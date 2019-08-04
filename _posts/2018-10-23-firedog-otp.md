---
layout: post
title: "FireDog OTP"
date: 2018-10-22 13:46:57
img: js-1.png
category: firedog
tags: [firedog]
---

在Erlang的世界里到处都是进程，一切的开发都基于进程为单位，除了其平行的调度，和强大的容错。
还有一点就是进程的使用特别方便，所以我计划用Golang实现一套。

<div class="divider"></div>

#### 目的

1. 有封装好的开发模型，对于项目程序具备一定的规范性codereview也方便一些；
2. 我自己构建的一套golang框架很多地方也用的上
下面会直接上代码，不正确的地方希望能指出。

#### 设计说明

在erlang中管理进程的模块为gen_server,它负责构建，管理，并提供访问等业务。
通过pid ！xx直接通信。

Golang这边我构建的管理进程的单元是Family,协程运行单元为room，通信管道为roomaisle；
从字面上解释就是，家庭，房间和过道，负责房间之间交互的介质就是过道roomaisle。

业务层golang的模块中，只需要声明一个room，并绑定几个接口，就实现了一个被family管理和封装的协程运行单元。

#### 开始上代码：family

<div class="divider"></div>

还是建议去库里面看，这里不会随库更新
```
import (
   "errors"
   "fmt"
   "sync"
   "time"
)

/**************************** export ********************************/

var Family *family

type family struct {
   rooms map[string]*Room
   sync.RWMutex
}

func init() {
   Family = &family{}
   Family.rooms = make(map[string]*Room)
}

func (f *family) Register(id string, v *Room) {
   f.Lock()
   defer f.Unlock()
   f.rooms[id] = v
}

func (f *family) UnRegister(id string) {
   f.Lock()
   defer f.Unlock()
   delete(f.rooms, id)
}

func (f *family) Get(name string) *Room {
   f.RLock()
   defer f.RUnlock()
   return f.rooms[name]
}

func (f *family) Count() int {
   f.RLock()
   defer f.RUnlock()
   return len(f.rooms)
}

func (f *family) Stop(id string) (err error) {
   defer func() {
      if r := recover(); r != nil {
         err = fmt.Errorf("%v", r)
      }
   }()
   v := f.Get(id)
   if v != nil {
      v.chanControl <- struct{}{}
   }
   return
}

func (f *family) Start(room *Room) (pid string, err error) {
   defer func() {
      if r := recover(); r != nil {
         err = fmt.Errorf("%v", r)
      }
   }()
   done := make(chan string)
   timeD := room.Time
   var loop func()
   if timeD <= 0 {
      loop = func() {
         room.oninit()
         done <- pid
         defer room.onclose()
         for {
            select {
            case input := <-room.chanMsg:
               if input == nil {
                  break
               }
               roomhandler(room, input)
            case <-room.chanControl:
               return
            }
         }
      }
   } else {
      timer := time.NewTimer(timeD)
      loop = func() {
         room.oninit()
         done <- pid
         defer room.onclose()
         for {
            select {
            case input := <-room.chanMsg:
               if input == nil {
                  break
               }
               roomhandler(room, input)
            case <-timer.C:
               //timer = time.After(igo.Timer())
               timer.Reset(room.Time)
               room.Timer(room)
            case <-room.chanControl:
               return
            }
         }
      }
   }
   go loop()
   pid = <-done
   if pid != "exit" {
      return pid, nil
   }
   return "exit", errors.New("create room failed")
}

func (f *family) Call(pid string, msg string, args []interface{}, timeout int) ([]interface{}, error) {
   var err error
   msgS := &roomaisle{
      chanRecv: make(chan []interface{}, 1),
      msg:      msg,
      args:     args,
   }
   defer func() {
      if r := recover(); r != nil {
         err = fmt.Errorf("%v", r)
      }
      close(msgS.chanRecv)
   }()
   v := Family.Get(pid)
   if v == nil {
      err = errors.New("room is not exist")
      return nil, err
   }
   v.chanMsg <- msgS
   select {
   case ret := <-msgS.chanRecv:
      return ret, err
   case <-time.After(time.Duration(timeout) * time.Second):
      return nil, errors.New("call time out")
   }
}

func (f *family) Cast(pid string, msg string, args []interface{}) error {
   var err error
   defer func() {
      if r := recover(); r != nil {
         err = fmt.Errorf("%v", r)
      }
   }()
   v := Family.Get(pid)
   if v == nil {
      return errors.New("room is not exist")
   }
   select {
   case v.chanMsg <- &roomaisle{
      msg:  msg,
      args: args,
   }:
   default: // 默认丢包
   }
   return nil
}

func (f *family) Pending(pid string) int {
   v := Family.Get(pid)
   if v == nil {
      return 0
   }
   return len(v.chanMsg)
}

func (f *family) IsAlive(pid string) bool {
   v := Family.Get(pid)
   if v == nil {
      return false
   }
   return true
```
#### 开始上代码：room
<div class="divider"></div>
```
import "time"
import "../log"

type Room struct {
   Args    interface{}
   Name    string
   OnInit  roomOpen
   Handler roomHandler
   Time    time.Duration
   Timer   roomTimer
   Close   roomClose

   chanMsg     chan *roomaisle // 信息走廊
   chanControl chan struct{}   // 内控通道
}

type roomaisle struct {
   chanRecv chan []interface{} // 接受消息
   msg      string             // 消息标识
   args     []interface{}      // 消息参数
}

type roomOpen func(room *Room, args []interface{})

type roomHandler func(room *Room, msg string, args []interface{}, ret chan []interface{})

type roomTimer func(room *Room)

type roomClose func(room *Room, reason string)

func (room *Room) oninit() {
   room.chanMsg = make(chan *roomaisle, 10000)
   room.chanControl = make(chan struct{}, 1)

   Family.Register(room.Name, room)
   room.OnInit(room, nil)
}

func (room *Room) onclose() {
   Family.UnRegister(room.Name)
   close(room.chanControl)
   close(room.chanMsg)
}

func roomhandler(room *Room, msg *roomaisle) {
   defer func() {
      if r := recover(); r != nil {
         log.Error("root handler panic msg %v error: %v", msg, r)
      }
   }()
   room.Handler(room, msg.msg, msg.args, msg.chanRecv)
}
```