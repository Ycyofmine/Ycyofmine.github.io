---
title: LyraStarterGame 01.默认地图传送门加载
date: 2024-11-25 11:25:46 +0800
categories: [UE]
tags: [UE] # TAG names should always be lowercase
media_subpath: /assets/img/UE
---

> 初学ue，有错误多包涵。

# DefaultEditorOverview

在预览界面，我们并没有看到传送门，开始游戏后能看到六个传送门，所以这节就来探究传送门如何生成的。

在 Outliner 内，我们可以看到一个蓝图类 `B_ExperienceList3D` ，这里头存储的便是生成六个传送门的逻辑。

我们先来探究一下蓝图类的成员变量 `UserFacingExperienceList`。

## Primary Asset Type
在 Project Settings 内的 Asset Manager ，存储着 8 种数据类型。
![](QQ20241129-101317.png)


![](QQ20241129-101555.png)



`B_ExperienceList3D` 通过循环，异步加载所有 `LyraUserFacingExperienceDefinition` 类型的资产。

处理完数据后，我们进入 `B_TeleportToUserFacingExperience` ，这里头存储着传送门的使用逻辑。

`sequence 0和1` 分别加载的是传送门后牌子(tile)和传送台的灯光颜色。


