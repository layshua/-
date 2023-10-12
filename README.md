---
title: README
aliases: []
tags: []
create_time: 2023-04-28 15:28
uid: 202304281528
banner: "[[Pasted image 20230109001000.jpg]]"
---
# Game Dev Road
test
## 框架图解

在深入学习 UE 之前，需要对 UE 包含哪些东西有一个大概的了解。正好 UE 官方自己曾出过一个**关于 UE 各个模块的引擎架构图**，是非常好的学习资料。如下图是 Character 模块的部分架构图。全模块完整架构图见 [Gitub 地址](https://github.com/drstreit/unreal_schematics)。

![[ba1a574e4de79376486d372f7b02c469_MD5.jpg]]

在这里插入图片描述

## Programming

### 必修

[官方文档——编程与脚本编写](https://docs.unrealengine.com/4.26/zh-CN/ProgrammingAndScripting/)

[UProperty 关键字全说明](https://benui.ca/unreal/uproperty/#editdefaultsonly)

[《InsideUE4》UObject 类型系统 - 反射实战](https://zhuanlan.zhihu.com/p/61042237)

### 选修

[深入蓝图开发——理解蓝图架构](https://neil3d.github.io/unreal/bp_in_depth.html) [深入蓝图开发——理解异步节点](https://neil3d.github.io/unreal/bp-async.html)

[一文搞懂 StaticClass、GetClass 和 ClassDefaultObject](https://zhuanlan.zhihu.com/p/380809095)

[一文搞懂 NativeClass、GeneratedClass、BlueprintClass、ParentClass](https://zhuanlan.zhihu.com/p/438501014)

[《InsideUE4》UObject 类型系统系列](https://zhuanlan.zhihu.com/p/24445322)

这个模块选择了一些 C++ 编程和蓝图开发相关的资料。对于蓝图和 UE 的 UObject 整个框架的理解将帮助你更好的开发上层玩法系统。另外还加了一个 UProperty 关键字的说明文章，方便查阅。

## GamePlay

### 必修

[《InsideUE4》GamePlay 架构系列](https://zhuanlan.zhihu.com/p/22833151)

[GAS 入门](https://www.bilibili.com/video/BV1X5411V7jh)

[官方文档——GAS](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/GameplayAbilitySystem/)

[GAS 各个模块的介绍和使用](https://zhuanlan.zhihu.com/p/486808688)

### 选修

[移动组件源码解读](https://zhuanlan.zhihu.com/p/34257208)

[GAS 预测机制讲解](https://zhuanlan.zhihu.com/p/143637846)

这个模块是玩法相关的学习资料。首先要做的肯定是配合前面的框架图来理解 UE 的 GamePlay 架构。然后就是学习 GAS (Game Ability System) 了，即 UE 的技能系统。虽然 GAS 叫作技能系统，但是可以使用的地方远远不止角色技能。比如一局 pvp 游戏阶段的切换也是可以用 GAS 来做的。可以说满足**需要复制、可以用 Tag 表示状态、有开始和结束时机**的一切事情都可以考虑用 GAS 来实现。

## Animation

### 必修

[官方文档——骨骼网格动画](https://docs.unrealengine.com/4.27/zh-CN/AnimatingObjects/SkeletalMeshAnimation/)

[深入浅出学习 ALS 高级运动系统视频教程](https://www.bilibili.com/video/BV12f4y1r71N?p=2)

### 选修

[过场动画 Sequencer 系列教程](https://www.youtube.com/watch?v=uEnfMV-4afA&list=PLn-Uu1ECw9nsW9pb8ttV5ZqJwV1PsvZU6)

[UE4/UE5 动画的原理和性能优化](https://zhuanlan.zhihu.com/p/545596818)

[RootMotion 原理分析](https://zhuanlan.zhihu.com/p/74554876)

[动画蓝图的初始化和更新流程解析](http://supervj.top/2022/01/06/%E5%8A%A8%E7%94%BB%E8%93%9D%E5%9B%BE%E6%B7%B1%E5%85%A5%E5%88%86%E6%9E%90/)

动画方面还是一样先看官方文档，然后是 ALS 的视频教程。ALS 高级运动系统，是 UE 的一个动画插件，实现了 3A 级别水准的走跑跳蹲的各种基础移动姿势，以及换武器、攀爬、八方向移动、IK 等各种动画相关的功能，而且是纯蓝图实现的。**学习 ALS 能够对 UE 的动画蓝图能够做什么功能以及怎么做有一个非常清晰的了解**。基本上你跟着第二个视频教程完整学一遍，UE 动画蓝图的所有功能你就可以掌握了。选修部分包含了一个**制作过场动画的教程**（一般客户端涉及这方面的需求比较少有需要再看），还有**动画底层的实现细节源码解析**（比较难懂，根据个人需要学习）。

## UI

### 必修

UMG 各个控件使用讲解

*   [CanvasPanel](https://www.cnblogs.com/timy/p/10264191.html)、[HorizontalBox](https://www.cnblogs.com/timy/p/10271319.html)、[Overlay](https://www.cnblogs.com/timy/p/10271719.html)、[UniformGrid Panel](https://www.cnblogs.com/timy/p/10272057.html)、[GridPanel](https://www.cnblogs.com/timy/p/10286541.html)
*   [WrapBox](https://www.cnblogs.com/timy/p/10287046.html)、[ScrollBox](https://www.cnblogs.com/timy/p/10287108.html)、[SizeBox](https://www.cnblogs.com/timy/p/10288338.html)、[Switcher](https://www.cnblogs.com/timy/p/10288843.html)、[SafeZone](https://www.cnblogs.com/timy/p/10289112.html)
*   [ListView](https://www.cnblogs.com/timy/p/10289456.html)、[TileView](https://www.cnblogs.com/timy/p/10289889.html)、[TreeView](https://www.cnblogs.com/timy/p/10289983.html)
*   [DynamicEntryBox](https://www.cnblogs.com/timy/p/10293433.html)、[InvalidationBox](https://www.cnblogs.com/timy/p/10293631.html)、[RetainerBox](https://www.cnblogs.com/timy/p/10296721.html)、[TextBox](https://www.cnblogs.com/timy/p/10299701.html)、[RichTextBlock](https://www.cnblogs.com/timy/p/10301234.html)
*   [Border](https://www.cnblogs.com/timy/p/10306095.html)、[Button](https://www.cnblogs.com/timy/p/10306472.html)、[CheckBox](https://www.cnblogs.com/timy/p/10306617.html)、[Image](https://www.cnblogs.com/timy/p/10307502.html)、[NamedSlot](https://www.cnblogs.com/timy/p/10310078.html)
*   [ProgressBar](https://www.cnblogs.com/timy/p/10312067.html)、[Slider](https://www.cnblogs.com/timy/p/10317047.html)、[ExpandableArea](https://www.cnblogs.com/timy/p/10317301.html)、[SpinBox](https://www.cnblogs.com/timy/p/10317508.html)、[CircularThrobber](https://www.cnblogs.com/timy/p/10317602.html)
*   [MenuAnchor](https://www.cnblogs.com/timy/p/10325705.html)、[Space](https://www.cnblogs.com/timy/p/10325905.html)、[BackgroundBlur](https://www.cnblogs.com/timy/p/10325967.html)

[ListView 踩坑](https://zhuanlan.zhihu.com/p/127184008)

[UMG 生命周期](https://www.cnblogs.com/sin998/p/15490311.html)

[Geometry 应用——获得 UMG 屏幕位置](https://gamedevworks.com/tutorials/unreal-engine/ue4-how-to-get-umg-widget-position-in-screen-space/)

### 选修

[UI 的制作与优化](https://zhuanlan.zhihu.com/p/584543674)

[Slate 源码分析——点击事件的触发流程](https://zhuanlan.zhihu.com/p/448050955)

[动态创建 UMG 的 Animation](https://zhuanlan.zhihu.com/p/364150799)

UI 方面先看各种控件的使用细节。熟练之后就是研究 Slate，或者是实现一些根据项目定制比较骚的功能。比如说想让一个按钮既可以拖动，又可以点击，又可以在列表里面滑动。或者是双层嵌套 LIstview，里面和外面的 ListView 都可以滚。

## AI

### 必修

[官方文档——AI](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/ArtificialIntelligence/)

[对行为树几个特性的理解](https://zhuanlan.zhihu.com/p/139514376)

### 选修

[AI_Perception 感知模块的使用和部分源码讲解](http://supervj.top/2023/02/01/AI_Percenption/)

[AI 行为树源码解读](https://zhuanlan.zhihu.com/p/368889019)

[Navmesh 底层 (Recast 基础和寻路算法)](https://zhuanlan.zhihu.com/p/74537236)

[状态树介绍](https://www.bilibili.com/video/BV1ed4y1b7Zk/)

AI 的话先跟官方文档过一遍熟悉行为树。UE 的行为树多了 Service 节点的概念。熟练之后研究行为树源码和寻路底层。值得一提的 **UE5 新增加的状态树**也值得了解一下。**状态树是一个通用的分层状态机结构**，是以状态为维度去设计的。用行为树来做 AI 有时候会发现状态之间的切换条件不够清晰，以及不容易知道对象处于哪个状态。而且行为树不能挂在非 Actor 身上。所以 UE5 引入了状态树解决了前面的问题。但是行为树也有行为树的好处。个人认为行为树和状态树是互相补充的，没有谁可以替代谁。状态树可以用来做一些**状态清晰，逻辑相对简单**的机制。比如一个门它有锁住、可开门、已开门这几种状态，就比较适合用状态树来做。

## Network

### 必修

[官方文档——多人联网](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Networking/)

[多人游戏的调试技巧](https://michaeljcole.github.io/wiki.unrealengine.com/Networking_%E5%A4%9A%E4%BA%BA%E5%9C%A8%E7%BA%BF%E6%B8%B8%E6%88%8F%E4%B9%8B%E5%A6%82%E4%BD%95%E6%B5%8B%E8%AF%95/)

[(如何理解属性复制)How to Understand Network Replication](https://www.youtube.com/watch?v=JOJP0CvpB8w)

### 选修

[网络同步原理深入](https://zhuanlan.zhihu.com/p/34723199)

[详解 ReplicationGraph 方案](https://nashnie.github.io/none/2019/08/05/UE-replication-graph.html)

网络方面的话肯定是研究 UE 自己的网络架构，包括 DS 架构以及属性复制、Actor 复制这些。用 UE 做网游如果不用 DS 而是自己研发一套服务器架构是非常麻烦且低效的，鸡佬所在项目曾经尝试过不用 DS，做了一年各种蛋疼，转 DS 后才发现香的一批。必修第三个链接是一个非常优秀的视频，把属性复制讲的非常清晰，看完之后会觉得打通了任督二脉。选修部分还加了一个 **ReplicationGraph 插件**的使用。ReplicationGraph 插件是用来解决大世界 Actor 复制效率低下的问题。如果是做地图比较大的游戏强烈推荐使用这个插件。

## Editor

### 选修

[Plugin 开发概要](https://arenas0.com/2019/02/14/Extending_the_UE4_Editor_1/)

[自定义菜单栏](http://supervj.top/2021/08/09/%E7%BC%96%E8%BE%91%E5%99%A8%E6%89%A9%E5%B1%95%EF%BC%9A%E8%87%AA%E5%AE%9A%E4%B9%89%E8%8F%9C%E5%8D%95%E6%A0%8F/)

[关卡编辑器扩展](https://zhuanlan.zhihu.com/p/129708783)

[ContentBrowser 拓展](https://zhuanlan.zhihu.com/p/129709982)

[资源类型扩展](https://zhuanlan.zhihu.com/p/135315547)

[属性面板拓展](https://zhuanlan.zhihu.com/p/135316945)

[创建自定义 Filter](https://yaksue.blog.csdn.net/article/details/120929455)

[创建一个预览视窗](https://yaksue.blog.csdn.net/article/details/109258860)

Editor 方面主要是编辑器扩展了。UE 的编辑器扩展比 Unity 比起来难搞了不少，你不光要懂 Slate 还得熟悉反射系统。一般对于客户端来说工具向的需求不多，所以这整个模块的内容我都列为了选修。上面这个文章也只是编辑器扩展的冰山一角，如果要深入学习还是建议看 UE 它自己的一些编辑器是怎么做的。

## Render & Effect

### 选修

[材质编辑器入门 Intro to Materials](https://www.youtube.com/watch?v=lngF4VVNER4&list=PLZlv_N0_O1gbQjgY0nDwZNYe_N8IcYWS-&index=1)

[Cascade 粒子系统入门 Into to Cascade](https://www.youtube.com/watch?v=OXK2Xbd7D9w&list=PLZlv_N0_O1gYDLyB3LVfjYIcbBe8NqR8t)

[Niagara 粒子系统官方文档](https://docs.unrealengine.com/4.27/zh-CN/RenderingAndGraphics/Niagara/)

渲染视觉效果方面我只列了材质编辑器和粒子系统的使用。毕竟我们是客户端路线，不是 TA 路线，看太多渲染相关的东西有点本末倒置了。

## UE5

### 必修

[官方文档——Lyra 工程](https://docs.unrealengine.com/5.0/en-US/lyra-sample-game-in-unreal-engine/)

### 选修

[Lyra 解读大纲](https://zhuanlan.zhihu.com/p/542781905)

[UE5 Lyra 示例项目解读（输入、移动、属性）](https://zhuanlan.zhihu.com/p/518029029) [UE5 Lyra 示例项目解读（武器射击、资源管理）](https://zhuanlan.zhihu.com/p/533685455)

[Lyra 的动画蓝图](https://zhuanlan.zhihu.com/p/517368184)

[GameFeature 框架](https://zhuanlan.zhihu.com/p/467236675)

[ControlFlows 插件](https://zhuanlan.zhihu.com/p/599766218)

Lyra 是 UE5 官方做的第三人称多人射击游戏学习项目。包含了基础的 3C (Character, Controller, Camera) 功能，输入 (EnhancedInputSystems)，GAS 技能，动画，消息转发 (GameplayMessageRouter)，多人联网等很多可以学习的内容。即使你是使用 UE4 的开发者也强烈建议学习。GameFeature 和 ControlFlows 都是在 Lyra 里面使用的插件，根据个人需要去看吧。

## 优秀个人博客

*   [vj 东](http://supervj.top/)
*   [南京周润发](https://www.zhihu.com/people/xu-chen-71-65/posts)
*   [大钊](https://www.zhihu.com/people/fjz13/posts)
*   [Jerish](https://www.zhihu.com/people/chang-xiao-qi-86/posts)
*   [stone](https://stonelzp.github.io/archives/)
*   [quabqi](https://www.zhihu.com/people/quabqi/posts)
*   [查里鹏](https://imzlp.com/tags/UnrealEngine/)
*   [(日)alwei](https://unrealengine.hatenablog.com/archive)
*   [(英)Noah zuo](https://arrowinmyknee.com/about/)
*   [(英)ben](https://benui.ca/unreal/)

关于作者:

*   **水曜日鸡**，喜欢 ACG 的游戏程序员。曾参与索尼中国之星项目《硬核机甲》的开发。目前在某大厂做 UE4 项目。

CSDN 博客：[https://blog.csdn.net/j756915370](https://blog.csdn.net/j756915370)  
知乎专栏：[https://zhuanlan.zhihu.com/c_1241442143220363264](https://zhuanlan.zhihu.com/c_1241442143220363264)