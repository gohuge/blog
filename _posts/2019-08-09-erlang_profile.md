---
layout: post
title: Erlang 性能相关测试
date: 2019-08-02 11:32:20
img: mac.jpg
tags: [erlang]
---

## Dets和Ets
Erlang自带了名为ETS(Erlang Term Storage)的纯RAM存储系统，以及名为DETS(Disk-based Erlang Term Storage)的RAM/Disk混合存储系统。为了评价ETS/DETS的性能并和Memcached进行比较，需要进行本测试。

## 测试环境
笔记本，Intel Duo 1.8GHz(双核)，1GB RAM，60G SATA HD，Linux kernel 2.6.24(Ubuntu 8.04 Hardy)。
Erlang OTP/R12B4，使用自带的ETS/DETS模块。
Memcached 1.2.2，使用libmemcached 0.22作为客户端。

## 测试用例和结果
循环插入100w次简单键值对，键值均为32-bit整数循环变量值，因此每个键值对的有效大小为(4+4)=8个byte。
- Erlang ETS 运行时间1.914s，QPS 522466.04
- Erlang DETS 运行时间45.603s，QPS 21928.38
- Memcached 运行时间54.22s，QPS 18443.3

## 初步结论
在测试使用的场景下，可以看出Erlang ETS的性能遥遥领先，这应该是因为ETS存储资源都在单个OS进程内，无须额外协议进行交互所致；
Erlang DETS和Memcached的性能基本一致。

## 修复Dets文件非常慢
Dets文件的修复时间与文件中的记录数成正比，虽然Dets文件修复以前很慢，但是Dets的实现已被大量改写和改进。