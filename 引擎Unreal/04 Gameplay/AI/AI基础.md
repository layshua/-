---
title: AI基础
create_time: 2023-09-11 21:21
uid: "202309112121"
banner: "![[]]"
reference: []
---

# AI 组件
AI 组件允许 Pawn 感知周围环境中的数据，例如噪声来源位置、或 Pawn 能够看到某个对象。
![[Pasted image 20230911211441.png]]

## AIPerception 感知组件

在 **`AIPerceptionSystem`** 中，**`AIPerceptionComponent`** 相当于刺激源的监听器，用于收集已注册的刺激信号。当组件获得新的刺激信号（批量）时，将会调用 **`UpdatePerception`**。

**AI 感知组件（AIPerception Component）** 用于在 **AI 感知系统（AI Perception System）** 中创建一个刺激监听器，收集可以响应的已注册刺激（本例中我们可以使用视觉）。这将使我们能够确定 AI 何时能实际看到玩家，并做出相应的反应。

Detection by Affiliation 从属检测：默认情况下 **Actor** 不会被指定归属，会被视为中立（neutrals）。

> [!NOTE]
> 目前，你还不能通过蓝图分配归属，因此为了检测玩家，我们要启用 **检测中立（Detect Neutral）** 标签。另外一种方法是使用 **Actor标签（Actor Tagging）** 来确定哪个角色是玩家，并强制AI角色只追逐被标记为玩家的Actor。

## PawnNoiseEmitter 噪声发射器组件

**`PawnNoiseEmitterComponent`** 追踪 **`PawnSensingComponents`** 使用的噪声事件数据来监听 Pawn。该组件主要存在于 Pawn 或其控制器上。它在网络客户端上不进行任何操作。

## PawnSensing 感应组件

Pawn 的感应组件（`Sensing Component`）用于封装 Actor 的感知（例如视觉和听觉）设置及功能，以便 Actor 在游戏世界中观察/监听 Pawn。其在网络客户端上不进行任何操作。

# 黑板

|子节点|描述|
|---|---|
|**装饰器（Decorator）**|也称为条件语句。这种节点附着于另一个节点，决定着树中的一个分支，甚至单个节点是否能够被执行。|
|**服务（Service）**|这类节点连接至 **任务（Task）** 节点和 **合成（Composite）** 节点，只要它们的分支正在执行，它们就会以所定义的频率执行。这些节点通常用于检查和更新 **黑板**。它们取代了其他行为树系统中的传统平行（Parallel）节点。|

装饰器例子：**黑板装饰器（Blackboard Decorator）** 来确定 **黑板键** 的数值，当它有效时，将会允许这个分支的执行。   

1. 选择所添加的 **Blackboard Based Condition**，并对 **细节（Details）** 面板进行以下设置。   
![[57221805c55025757d460fdb31218aaf_MD5.jpg]]
- 将观察者终止（Observer aborts） **设为** 两者（Both） 
- 将 **黑板键（Blackboard Key）** 设为 **HasLineOfSIght**  
- 将 **节点名称（Node Name）** 设为 **Has Line of Sight?**

我们在此说明，当 **HasLineOfSight** 的数值为 **已设置（Is Set）** （或为 true）时，会执行这个 **追逐玩家（Chase Player）** 分支。
**观察者终止（Observer aborts）** 中设置为 **两者（Both）** 意味着当我们分配的 **黑板键** 改变时，中止本身（**追逐玩家**）和任何低优先级的任务。
- 当 **HasLineOfSight** 的数值改变且未设置时，其将中止自身（**追逐玩家**），此时将执行下一个分支（**巡逻**）。
- 当 **HasLineOfSight** 数值再次变成 **已设置（Is Set）** 时，观察者将中止优先级较低的任务，并使 **追逐玩家（Chase Player）** 分支能够再次被执行。

# 行为树
执行逻辑时，行为树会使用一种名为 **黑板** 的独立资源来存储它需要知道的信息（名为 **黑板键**），从而做出有根据的决策。常见工作流程是创建一块黑板，添加一些黑板键，然后创建一个使用黑板资源的行为树（如下图所示，黑板被指定到行为树）。

**执行顺序**：从左到右，从上到下（中左右）
![[Pasted image 20230911230545.jpg]]
>节点右上角的数字，表示操作的顺序 
>蓝色节点被称为[装饰器](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-decorators)节点（在其他行为树系统中被称为 _条件语句_）。它连接到一个[合成](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-composites)节点，用于验证该黑板键是否为 true。这决定了该分支的其余部分是否能够执行。
>紫色节点是[任务（Task）](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-tasks)节点，是AI可以完成的动作。

**合成（Composites） 节点**是流控制的一种形式，决定了与其相连的子分支的执行方式。  


|合成节点|描述 |
|---|---|
|**选择器（`Selector`）**|**从左到右执行分支**，通常用于在子树之间进行选择。**当选择器找到能够成功执行的子树时，将停留在该分支中，直到它的执行结束，然后转到选择器的父合成节点，继续决策流。**|
|**序列（`Sequence`）**|从左到右执行分支，通常用于按顺序执行一系列子项。与选择器节点不同，序列节点会持续执行其子项，直到它遇到失败的节点。|
|**简单平行（`Simple Parallel`）**|简单平行节点有两个"连接"。第一个是主任务，它只能分配一个任务节点（意味着没有合成节点）。第二个连接（后台分支）是主任务仍在运行时应该执行的活动。简单平行节点可能会在主任务完成后立即结束，或者等待后台分支的结束，具体依属性而定。|

