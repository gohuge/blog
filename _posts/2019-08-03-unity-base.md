---
layout: post
title: Unity 相关知识
date: 2019-08-02 11:32:20
img: unity.jpg
tags: [unity]
---
有一点是我们考虑的前提，规范和标准，我们根据产品指标做出对应的数据标准，
只有在这个前提下才能达到产品预期，同时只有在一套商定的标准下各个部门才能有效配合。所以我们会根据方向的几点，推导出符合自己产品定位的标准。

## 方向【标准和性能】

一、标准和规范（资源，运行指标）

1、帧率
大多数现代游戏的目标是达到60 FPS的帧速率。通常，高于30 FPS的帧速率被认为是可接受的，特别是对于不需要快速反应的游戏，例如益智游戏或冒险游戏。有些项目有特殊要求; 例如，在VR中，90 FPS被认为是关键的。在帧速率低于30 FPS时，玩家通常会觉得这种体验不愉快; 

我们应该遵循公式1000 / [所需的帧速率]。使用这个公式，我们可以看到，对于每秒渲染30帧的游戏，它必须在33.3毫秒内渲染每个帧。对于以60 FPS运行的游戏，它必须在16.6毫秒内渲染每个帧。