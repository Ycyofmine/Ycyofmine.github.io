---
title: LyraStarterGame 04.AI
date: 2024-12-05 23:33:04 +0800
categories: [UE]
tags: [UE] # TAG names should always be lowercase
media_subpath: /assets/img/UE
---
在这之前，请先阅读[行为树快速入门](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/behavior-tree-in-unreal-engine---quick-start-guide?application_version=5.5)

# 行为树节点参考
## 黑板
在 行为树（Behavior Tree）中，Blackboard 是一个全局的数据存储区域，用于存放和共享 AI 的状态信息、变量以及条件。这些信息可以在树中的任何节点（包括选择器、序列、装饰器等）之间进行访问和更新。

## 复合节点
1. 选择器
    
    按从**左到右**的顺序执行其子节点。当其中**一个子节点**执行成功时，选择器节点将停止执行，并算作选择器节点执行成功。**所有子节点**运行失败，则选择器运行失败。

2. 序列
   
   节点按从**左到右**的顺序执行其子节点。当其中**一个子节点**失败时，序列节点也将停止执行，并算作序列节点执行失败。**所有子节点**运行成功，序列节点才算运行成功。

3. 简单平行节点

    与 **序列节点（Sequence）** 和 **选择器节点（Selector）** 不同，平行节点不依赖于子节点的成功或失败，它通常允许多个任务同时并行执行，常用于处理一些需要并发进行的任务。

## 装饰器节点
有非常多的特殊节点，实现功能的时候再进行学习吧[🔗](https://dev.epicgames.com/documentation/zh-cn/unreal-engine/unreal-engine-behavior-tree-node-reference-decorators)

## 服务节点
1. 默认聚焦(Default Focus)

    通过设置控制器的聚焦来创建访问 蓝图 和代码中Actor的快捷方式。将AI控制器的聚焦设置到Actor上后，你便能直接从AI控制器对其进行访问，而不需要访问黑板键。

2. EQS(Environment Query System)
   
   用于指定的时间间隔定期执行操作，对指定的黑板键进行更新。通常用来选择目标、位置、路径等。它能处理较复杂的环境决策，适用于需要考虑多个目标的情况。

3. 自定义服务节点

    在行为树中持续执行的任务或周期性检查，通常用于更新状态、触发事件或辅助行为树做决策。它用于更细粒度的行为树状态更新。

EQS和自定义服务节点虽然都可以影响 AI 的行为，但用途和实现方式有所不同。EQS 是用于环境决策的工具，而自定义服务模板则是用于定期更新 AI 状态或执行持续任务的工具。

# Lyra Bot
`/ShooterCore/Bot/BT/BT_Lyra_Shooter_Bot.BT_Lyra_Shooter_Bot` 直接进文件看吧，基础逻辑还是非常清晰的，按照节点的命名很快就能看明白。

EQS 内的事件执行顺序是**从上往下**的。

我们主要聚焦在 EQS 节点上。

`EQS_MoveAgainstEnnemy` 是 `Find Best Position` 的 `Query Template` 。

![EQS_MoveAgainstEnnemy](QQ20241208-232702.png)

0. 生成一个 400 * 400 的网格
1. 在网格上寻找可以到达的点
2. 通过黑板键得到TargetEnemy坐标，黑板键的TargetEnemy则是通过 `AIPerception used detect enemy` 节点更新的。在 `trace data` 中可以设置感官。
3. 在 Details 的 Preview 曲线中，可以看到权重是越远越大的
   
    ![](QQ20241209-010945.png)

    


# 参考
[Unreal Engine 5 - Exploring Lyra - Part 4 (AI)](https://www.youtube.com/watch?v=jZFgTEGRJxg&list=PLNBX4kIrA68lSY6Pj3zDVH6kGDIMgwOvr&index=4)