---
title: LyraStarterGame 03.人物移动和转身动画解析
date: 2024-12-04 15:56:41 +0800
categories: [UE]
tags: [UE] # TAG names should always be lowercase
media_subpath: /assets/img/UE
---
# 距离匹配和步幅适配
1. 距离匹配（Distance Matching）
   
    调整动画的播放速率，例如开始、停止和着陆动画等运动动画资产。
    
    ![](distancematching.png)


2. 步幅适配（Stride Warping）
   
   用于动态调整角色的步幅长度，这适用于不调整播放速率的情况，例如角色进入慢跑（Jog）状态时。

   ![](stridewarping.png)
   ![](stridewarp.gif)

结合这两种方法之后，你可以动态地选择侧重于其中一种方法。在开始状态期间，我们首先使用距离匹配（Distance Matching）来保留姿势，然后在接近慢跑（Jog）状态时使用步幅适配（Stride Warping）来混合。

![](updatestartanim.png)

> 挖个坑，实现的详细解析。

# TurnInPlace
## TurnInPlace #1
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/AnimGraph`

当Pawn的所有者旋转时，网格组件会随之旋转，这会导致角色的脚部滑动。

在这里，我们通过抵消角色的旋转来保持脚部位置不动。

## TurnInPlace #2
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/UpdateRootYawOffset`

此函数处理根据Pawn所有者当前状态来更新偏航偏移量。

## TurnInPlace #3
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/SetRootYawOffset`

我们会限制偏移量，因为当偏移量过大时，角色需要转动得太远，导致脊柱扭曲得过度。虽然转身动画通常能跟上偏移量，但如果用户旋转相机过快，这会导致脚部滑动。

如果需要，可以将这个限制替换为使用能达到180度的瞄准动画，或者更积极地触发转身动画。

## TurnInPlace #4
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/SetRootYawOffset`

我们希望通过瞄准动作来抵消偏航偏移量，以保持武器的瞄准方向与相机对齐。

## TurnInPlace #5
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/ProcessTurnYawCurve`

当偏航偏移量变得过大时，我们触发转身动画来将角色旋转回来。例如，如果相机旋转了90度到右侧，它将面向角色的右肩。如果我们播放一个旋转角色90度到左侧的动画，角色将重新面朝相机。

我们使用“TurnYawAnimModifier”动画修饰符在每个转身动画中生成必要的曲线。

有关触发转身动画的示例，请参见 **ABP_ItemAnimLayersBase**。

## TurnInPlace #6 (also see AnimBP_Mannequin_Base)

当偏航（yaw）偏移量变得足够大时，我们触发一个“TurnInPlace”动画来减少偏移量。

“TurnInPlace”动画通常会以一些平稳的动作结束，表示旋转完成。在这个过程中，我们会进入“TurnInPlaceRecovery”状态。如果偏移量再次变大，系统会从该状态切换回“TurnInPlaceRotation”状态。

通过这种方式，如果Pawn的所有者持续旋转，我们可以继续播放“TurnInPlace”动画的旋转部分，而无需等待平稳动作结束。

## 总结
![](v2-354a0f12e44e511a101e113902e20181_r.jpg)

大致思路就是在**Idle**和**Stop**状态下每帧都计算 `RootYawOffset` ，用于抵消下半身的旋转，防止产生滑步旋转，通过独立的下半身转身动画来完成转身。

如果旋转角度超过一个小阈值状态机就转到 `TurnInPlaceRotation` ，此时上半身旋转下半身不旋转，在 `TurnInPlaceRecovery` 状态进行下半身旋转，但是如果旋转角度过大，在 `TurnInPlaceRotation` 就会让下半身进行旋转，实现在 `ABP_Mannequin_Base/ProcessTurnYawCurve` 。

[UE5 白话Lyra动画系统](https://zhuanlan.zhihu.com/p/654430436)

http://supervj.top/2023/11/14/Lyra_%E5%8A%A8%E7%94%BB/

[Unreal Engine 5 - Exploring Lyra - Part 3 (Animation)](https://www.youtube.com/watch?v=ys_kSOKpTtg&list=PLNBX4kIrA68lSY6Pj3zDVH6kGDIMgwOvr&index=3)