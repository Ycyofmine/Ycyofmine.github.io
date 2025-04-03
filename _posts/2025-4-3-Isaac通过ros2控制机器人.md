---
title: 智能指针
date: 2025-04-03 10:41:31 +0800
categories: [机器人]
tags: [robot] # TAG names should always be lowercase
media_subpath: /assets/img/robot
---
好久没更新博客了，学习方向也发生了改变，不搞游戏啦。

不更新博客是因为使用了 Obisidian ，多端同步太爽了，由于我大量使用了双链，不适合发博客，所以只能偶尔整理出一篇文章来。

---

本教程大量使用了官方的插件，如果未找到笔者所使用的插件，请翻阅工作区所有子目录，若未找到，请去 `Extensions` 内开启或下载（Isaac不同版本之间工作区目录非常不一样，**FUCK NVIDIA**）。
![[Pasted image 20250403100958.png]]
## 导入模型
![[Pasted image 20250403101055.png]]

Input File选.urdf文件即可，设置如下：
![[Pasted image 20250403102543.png]]

官方是将URDF转换成USD并导入。
⚠️插件会将URDF中关节名中部分的 **-** 转换成 **\_** ，编写ros2代码的时候请注意。

## 机器人设置subscriber和publisher
![[Pasted image 20250403102407.png]]
`Articulation Root` 选导入进来的机器人的根目录即可

![[Pasted image 20250403103209.png]]
⚠️此时打开新建出来 Graph 下的 Graph 并开始仿真，会看到如下报错：

![[Pasted image 20250403103436.png]]

这是因为模型的 `Articulation Root` 不在模型的根目录，接下来我们通过筛选器找出模型的 `Articulation Root` 保存在哪，把他删除。 

![[Pasted image 20250403103540.png]]
再在根目录下添加`Articulation Root` 

![[Pasted image 20250403103721.png]]
开始仿真后即可在ros2中观察到：

![[Pasted image 20250403103945.png]]