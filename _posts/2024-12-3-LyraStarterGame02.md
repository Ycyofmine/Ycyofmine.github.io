---
title: LyraStarterGame 02.动画系统逻辑
date: 2024-12-03 11:12:17 +0800
categories: [UE]
tags: [UE] # TAG names should always be lowercase
media_subpath: /assets/img/UE
---

# Anim System
以下内容是来自 `Lyra` 本身的注释，我对其翻译了一下。
## AnimBP 讲解 #1（也参见 ABP_ItemAnimLayersBase）
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/EventGraph`

这个 AnimBP 在其 Event Graph 中没有运行任何逻辑。

Event Graph 中的逻辑会在游戏线程上处理。每一帧，所有 AnimBP 的 Event Graph 必须依次按顺序运行，这会成为性能瓶颈。

为了解决这个问题，在这个项目中，我们使用了新的 **BlueprintThreadsafeUpdateAnimation** 函数（可以在“我的蓝图”标签中找到）。在 BlueprintThreadsafeUpdateAnimation 中的逻辑可以并行运行多个 AnimBP，消除了游戏线程的开销。

##  AnimBP 讲解 #2
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/BlueprintThreadSafeUpdateAnimation`

这个函数主要负责收集游戏数据并将其处理成有用的信息，用于选择和驱动动画。

使用线程安全的函数时，有一个限制，即我们无法像在 Event Graph 中那样直接访问游戏对象的数据。这是因为其他线程可能在同时运行并修改这些数据。相反，我们使用 **Property Access** 系统来访问数据。Property Access 系统会在安全的时候自动复制数据。

举个例子，我们可以通过“Property Access”来访问 Pawn 所属的位置（可以通过右键菜单搜索 "Property Access"）。

##  AnimBP 讲解 #3
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/AnimGraph`

这个 Anim Graph 不直接引用任何动画。它提供了蒙太奇（Montages）和链接动画层（Linked Animation Layers）的入口点，以便在图中某些点播放姿势。这个图的主要目的是将这些入口点融合在一起（例如，将上半身和下半身的动画姿势融合在一起）。

这种方法允许我们只在需要时加载动画。例如，武器会持有所需蒙太奇和链接动画层的引用，这样只有在武器加载时相关数据才会被加载。

例如，**B_WeaponInstance_Shotgun** 持有蒙太奇和链接动画层的引用。只有在 **B_WeaponInstance_Shotgun** 加载时，相关数据才会被加载。
**B_WeaponInstance_Base** 负责链接武器的动画层。

##  AnimBP 讲解 #4
`/Game/Characters/Heroes/Mannequin/Animations/ABP_Mannequin_Base.ABP_Mannequin_Base/LocomotionSM`

这个状态机处理高层角色状态之间的过渡。

每个状态的行为主要由 **ABP_ItemAnimLayersBase** 中的层（layers）来处理。

---

在这个页面还提到了**状态别名**这个概念：多个状态对应一个状态别名。
![](QQ20241202-195429.png)

可以注意一下状态别名的图标是特殊的。

有助于减少所需的连接数量。

## AnimBP Tour #5
`/Game/Characters/Heroes/Mannequin/Animations/LinkedLayers/ABP_ItemAnimLayersBase.ABP_ItemAnimLayersBase/EventGraph`


与 **AnimBP_Mannequin_Base** 一样，这个 **AnimBP** 在 **BlueprintThreadSafeUpdateAnimation** 中执行其逻辑。 

此外，这个 **AnimBP** 可以使用 **Property Access** 和 **GetMainAnimBPThreadSafe** 函数从 **AnimBP_Mannequin_Base** 中访问数据。以下是一个示例。

## AnimBP Tour #6
`/Game/Characters/Heroes/Mannequin/Animations/LinkedLayers/ABP_ItemAnimLayersBase.ABP_ItemAnimLayersBase/EventGraph`


这个 **AnimBP** 是为处理常见武器类型的逻辑而编写的，例如步枪和手枪。如果需要自定义逻辑（例如弓这样的武器），可以编写一个不同的 **AnimBP**，并实现 **ALI_ItemAnimLayers** 接口。  

这个 **AnimBP** 并不直接引用动画资源，而是有一组可以被子动画蓝图覆盖的变量。这些变量可以在 **My Blueprint** 标签中的 "Anim Set - X" 类别下找到。  

这使得我们能够在多个武器之间复用相同的逻辑，而无需在一个 **AnimBP** 中引用（从而加载）每个武器的动画内容。  

请参见 **ABP_RifleAnimLayers**，这是一个子动画蓝图，提供每个 "Anim Set" 变量的值的示例。

## AnimBP Tour #7
`/Game/Characters/Heroes/Mannequin/Animations/LinkedLayers/ABP_ItemAnimLayersBase.ABP_ItemAnimLayersBase/FullBody_IdleState`

在重载函数列表中可以找到。

这个AnimBP为**AnimBP_Mannequin_Base**中的每个状态实现了一个层。  
这些层可以播放单个动画，或者包含像状态机这样的复杂逻辑。

## AnimBP Tour #8
`/Game/Characters/Heroes/Mannequin/Animations/LinkedLayers/ABP_ItemAnimLayersBase.ABP_ItemAnimLayersBase/FullBody_StartState`

这是一个使用 **Anim Node Functions** 的示例用例。

**Anim Node Functions** 可以在动画节点上运行。它们只会在节点激活时运行，这使得我们可以将逻辑局部化到特定的节点或状态。 

在这个例子中，一个 **Anim Node Function** 会在节点变得相关时选择要播放的动画。另一个 **Anim Node Function** 管理动画的播放速率。

## AnimBP Tour #9
`/Game/Characters/Heroes/Mannequin/Animations/LinkedLayers/ABP_ItemAnimLayersBase.ABP_ItemAnimLayersBase/UpdateStartAnim`

这是一个使用 **Distance Matching**（距离匹配）的示例，用于确保 **Start** 动画的移动距离与 **Pawn** 的实际移动距离匹配。这样可以防止脚滑现象，通过保持动画与运动模型同步。  

这实际上控制了 **Start** 动画的播放速率。我们会将有效的播放速率限制在一定范围内，以防止动画播放得太慢或太快。  

如果有效的播放速率被限制，我们仍然可能看到一些滑动现象。为了解决这个问题，我们稍后使用 **Stride Warping**（步幅扭曲）来调整姿势，以修正剩余的差异。  

要访问 **Distance Matching** 函数，必须使用 **Animation Locomotion Library** 插件。

## AnimBP Tour #10
`/Game/Characters/Heroes/Mannequin/Animations/LinkedLayers/ABP_ItemAnimLayersBase.ABP_ItemAnimLayersBase/FullBody_StartState`

这是一个将动画的原始姿势与 **Pawn** 实际动作匹配的示例。

**Orientation Warping**（方向扭曲）会旋转动画的下半身姿势，使其与 **Pawn** 的移动方向对齐。我们只制作前后左右四个方向的动画，并依赖扭曲来填补其余方向的空缺。 

然后，**Orientation Warping** 会重新调整上半身，使角色继续面朝相机的方向。  

**Stride Warping**（步幅扭曲）会根据动画的预设速度与 **Pawn** 实际速度的差异，缩短或拉长腿部的步幅。
  
要访问这些节点，必须使用 **Animation Warping** 插件。

## 总结
状态机保存在`ABP_Mannequin_Base/LocomotionSM`，

![](QQ20241203-214437.png)

`ALI_ItemAnimLayers` 是一个接口，声明了动画层的变量，实现了状态机。

`ABP_Mannequin_Base` 则是负责提供入口点，将动作融合在一起，例如上半身和下半身融合。

`ABP_ItemAnimLayersBase` 是所有武器使用的基础链接层动画蓝图，它持有一组可以被子动画蓝图覆盖的变量，也就是 `Anim Set - X` ，这些变量持有的值为空。

![](QQ20241203-215727.png)

`ABP_XXXAnimLayers` 则是真正保存着动画资产的地方。

再往下的 `B_WeaponInstance_XXX` 则是数据蓝图，负责填入 `B_WeaponInstance_Base` 所需的数据，即可创造一把新的武器。

---
**好处**
- 只有被引用的动画资产才会被加载进内存中。
- 且极大的方便了新模块的加入。
- 手枪和步枪共用一个状态机，只需要用不同的资产配置蓝图即可。

# 参考
[UE5 白话Lyra动画系统](https://zhuanlan.zhihu.com/p/654430436)
