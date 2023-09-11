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

**AI感知组件（AIPerception Component）** 用于在 **AI感知系统（AI Perception System）** 中创建一个刺激监听器，收集可以响应的已注册刺激（本例中我们可以使用视觉）。这将使我们能够确定AI何时能实际看到玩家，并做出相应的反应。
## PawnNoiseEmitter 噪声发射器组件

**`PawnNoiseEmitterComponent`** 追踪 **`PawnSensingComponents`** 使用的噪声事件数据来监听 Pawn。该组件主要存在于 Pawn 或其控制器上。它在网络客户端上不进行任何操作。

## PawnSensing 感应组件

Pawn 的感应组件（`Sensing Component`）用于封装 Actor 的感知（例如视觉和听觉）设置及功能，以便 Actor 在游戏世界中观察/监听 Pawn。其在网络客户端上不进行任何操作。

# 黑板
# 行为树
**合成（Composites）** 节点是流控制的一种形式，决定了与其相连的子分支的执行方式。  



|合成节点|描述 |
|---|---|
|**选择器（`Selector`）**|**从左到右执行分支**，通常用于在子树之间进行选择。**当选择器找到能够成功执行的子树时，将停留在该分支中，直到它的执行结束，然后转到选择器的父合成节点，继续决策流。**|
|**序列（`Sequence`）**|从左到右执行分支，通常用于按顺序执行一系列子项。与选择器节点不同，序列节点会持续执行其子项，直到它遇到失败的节点。|
|**简单平行（`Simple Parallel`）**|简单平行节点有两个"连接"。第一个是主任务，它只能分配一个任务节点（意味着没有合成节点）。第二个连接（后台分支）是主任务仍在运行时应该执行的活动。简单平行节点可能会在主任务完成后立即结束，或者等待后台分支的结束，具体依属性而定。|

节点右上角的数字，表示操作的顺序 

![The numbers in the upper-right corner of the nodes](https://docs.unrealengine.com/5.2/Images/making-interactive-experiences/artificial-intelligence/behavior-trees/behavior-tree-quick-start/behavior-tree-quick-start-step-3-6b.jpg)  

。**行为树** 会从左到右和自上而下执行（左中右），因此节点的排列很重要。对 AI 最重要的动作通常应该放在左边，而次要的动作（或退却行为）应该放在右边。子分支会以相同的方式执行，如果任何子分支失败，整个分支将会停止执行，导致失败并返回上级树。举例而言，如果 **追逐玩家（Chase Player）** 节点失败，它将返回至上级 **AI 根（AI Root）**，然后转变为 **巡逻** 节点。