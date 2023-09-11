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
|**装饰器（Decorator）**|也称为条件语句。这种节点附着于另一个节点，决定着树中的一个分支，甚至单个节点是否能够被执行。 |
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
执行逻辑时，行为树会使用一种名为 **黑板** 的独立资源来存储它需要知道的信息（名为 **黑板键**），从而做出有根据的决策。常见工作流程是创建一块黑板，添加一些黑板键，然后创建一个使用黑板资源的行为树。

**运行前提：**
1. Pawn 关联 AIController![[Pasted image 20230911231500.jpg]]
2. AIController 中， 当该控制器"Possess"此Pawn时，运行一个行为树：![[Pasted image 20230911231525.jpg]]
3. 关卡中寻路需要添加 NavMesh

**执行顺序**：从左到右，从上到下（中左右）
![[Pasted image 20230911230545.jpg]]
>节点右上角的数字，表示操作的顺序 
>蓝色节点被称为[装饰器](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-decorators)节点（在其他行为树系统中被称为 _条件语句_）。它连接到一个[合成](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-composites)节点，用于验证该黑板键是否为 true。这决定了该分支的其余部分是否能够执行。
>紫色节点是[任务（Task）](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-tasks)节点，是AI可以完成的动作。

**特点：（与传统行为树的比较）**
1. 行为树由事件驱动，被动地监听可用于触发树中变化的"事件"，避免每帧进行不必要的共工作。
2. 条件语句并非叶节点，而是装饰器（Decorator）节点。所有叶节点都是操作任务节点。条件语句装饰器的另一个优点是可以轻松将装饰器设为树中关键节点上的观察者（等待事件）。要充分利用树事件驱动的特点，此特性十分关键。
3. 使用**简单平行**节点（[服务](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-services)节点），以及[装饰器](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-decorators)节点的 **观察者中止（Observer Aborts）** 属性来处理并发行为。
## 合成节点 Composite

> [!NOTE] 合成节点
> 是流控制的一种形式，决定了与其相连的子分支的执行方式。  
> 只有 **合成** 节点可以连接至 **行为树** 的 **Root** 节点。

|合成节点|描述 |
|---|---|
|**选择器（`Selector`）**|**从左到右顺序执行其子节点**。当其中一个子节点执行成功时，选择器节点将停止执行。<div>如果选择器的一个子节点成功运行，则选择器运行成功。如果选择器的所有子节点运行失败，则选择器运行失败。</div> |
|**序列（`Sequence`）**|**从左到右的顺序执行其子节点**。当其中一个子节点失败时，序列节点也将停止执行。<div>如果有子节点失败，那么序列就会失败。如果该序列的所有子节点运行都成功执行，则序列节点成功。</div>|
|**简单平行（`Simple Parallel`）**|简单平行节点有两个"连接"。<div>第一个是**主任务**，它只能分配一个任务节点（意味着没有合成节点）。</div><div>第二个连接是**后台分支**：是主任务仍在运行时应该执行的活动。</div><div>简单平行节点可能会在主任务完成后立即结束，或者等待后台分支的结束，具体依属性而定。</div> |

可以将简单平行节点理解为"执行 A 的同时，也在执行 B"。例如"攻击敌人，同时也朝敌人移动。"从基本上而言，A 是主任务，B 是后台分支。

## 装饰器节点 Decorator
条件语句，连接到[合成（Composite）](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-composites)或[任务（Task）](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-tasks)节点，并定义树中的分支，甚至单个节点是否可以执行。
![[Pasted image 20230911235256.png]]
- **Composite**：合成，自定义逻辑。以这种方式使用合成装饰器将影响内存和性能。也可以在C++中创建一个装饰器来执行同样的自定义行为，但效率更高。
- **Blackboard**：检查给定的 **黑板键（Blackboard Key）** 上是否设置了值。
- **Check Gameplay Tags on Actor**：## 检查Gameplay标签条件
- **Compare BBEntries**：比较两个 **黑板键** 的值，并根据结果（等于或不等）阻止或允许节点的执行。
- **Conditional Loop**：条件语句循环，只要满足了 **键查询（Key Query）** 条件，该装饰器将使它所连接节点进行循环。
- **Cone Check**：椎体检查装饰器，采用了三个矢量键：第一个确定椎体的起始位置，第二个用于定义锥体朝向的方向，第三个用于检查该位置是否在锥体内部。您可以使用 **锥体半角（Cone Half Angle）** 属性来定义锥体的角度。
- **Cooldown**：冷却装饰器， 将锁定节点或分支的执行，直到冷却时间结束。
- **Does Path Exist**：路径是否存在，会检查是否可以在以下两个矢量之间创建路径：黑板键A和黑板键B
- **Force Success**：强制成功装饰器，会将节点结果更改为成功。
- **Is at Location**：在位置处装饰器，节点将检查 AI 控制的 Pawn 是否位于给定位置。
- **ls BBEntry Of Class**：用于确定所指定的黑板键是否属于指定的类。
- **Keep in Cone**：保持在椎体内装饰器，节点会根据所观察到的位置是否仍位于锥体内部来确定其条件。当节点首次拥有相关性时，将计算锥体的方向。
- **Loop**：对节点或分支进行多次循环或无限次循环。
- **Set Tag Cooldown**：设置Gameplay标签的冷却时长。
- **Tag Cooldown**：基于Gameplay标签的冷却计时器是否过期。
- **Time Limit**：时间限制装饰器，会为一个分支或节点设置在终止其运行并失败前，完成运行所需的一段时间。每当该节点被聚焦时，计时器都会重置。

## 服务节点 Service

**服务节点**：[服务](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-services)节点是与任意**合成节点**（选择器节点、序列节点或简单平行节点）相关联的一种特殊节点。只有在合成节点的子节点正在被执行时才会激活。
它**能够针对指定秒数的每个回调进行注册，并能对多种需要周期性出现的类型进行更新。**
举例而言，当AI Pawn面对当前敌人、继续在其行为树中正常行动时，可以使用服务节点为该Pawn确定最适合追逐的敌人。

## 观察者中止 Observer Aborts

传统行为树的标准平行节点的一个常见用处是不断检查条件。一旦任务所需的条件变成 false，该任务就可以中止。

举例而言，有一只猫在执行序列节点（例如"发出嘶声"和"扑击"）。如果老鼠逃入洞里，则需要猫立即放弃。如果使用平行节点，你要设置一个子项检查是否可以扑击老鼠，然后设置另一个子项检查序列要执行的动作。
因为虚幻引擎行为树由事件驱动，所以我们要解决此问题。**可以通过条件装饰器观察其数值，并且在必要时中止**。在本例中，可以在序列上设置"是否可以扑击老鼠？（Mouse Can Be Pounced On?）"装饰器节点，把"观察者中止（Observer Aborts）"设置为"自身（Self）"。

四个选项：
- None 不中止执行。
- Self 中止此节点自身和在其下运行的所有子树。
- Lower Priority 中止此节点右侧的所有节点。
- Both ：中止 Self+Lower Priority

## 行为树节点实例化规则

行为树节点作为共享对象存在，这意味着使用同一行为树的所有代理将共享一组节点实例。这样不仅可以在降低内存使用率的同时提升 CPU 性能，还可以防止节点保存代理特定的数据。
不过，对于代理需要存储和更新节点相关信息的情况，虚幻引擎提供了以下三种解决方案：

### 实例化节点

将节点的 `bCreateNodeInstance` 变量设为 `true` 后，将使每个使用行为树的代理成为特殊的节点实例，以牺牲一定性能和内存使用率为代价来确保安全存储代理专属的数据。
包括 `UBTTask_BlueprintBase`、`UBTTask_PlayAnimation`、`UBTTask_RunBehaviorDynamic` 在内的部分虚幻引擎节点类均使用此功能。

### 存储在黑板上

常见的解决方案是将变量存储在黑板上。执行此操作的方法是从节点公开变量命名，然后在节点初始化过程中使用该命名获取和存储黑板键。然后便可以使用黑板键在代理的黑板实例上获取并设置该变量的值。此方法支持 `bool`、`float`、`FVector`、`int32`、`enum`（存储为 `uint8`）、`UObject*` 类型的变量。

### 存储在行为树节点上

可以创建自定义结构体或类，将变量存储在节点的内存中。例如，`UBTTask_MoteTo` 类利用 `FBTMoveToTaskMemory`。

您可以在 `BTTask_MoteTo.h` 中找到以下代码：
```c++
struct FBTMoveToTaskMemory
{
    /** Move request ID */
    FAIRequestID MoveRequestID;

    FDelegateHandle BBObserverDelegateHandle;
    FVector PreviousGoalLocation;

    TWeakObjectPtr<UAITask_MoveTo> Task;

    uint8 bWaitingForPath : 1;
    uint8 bObserverCanFinishTask : 1;
};
```

`UBTNode` 中的许多虚函数都将 `uint8*` 参数带到节点的内存中。此参数指示为代理分配的内存块，内存块大小将由 `GetInstanceMemorySize` 的覆盖版本返回。节点将为各个代理分配此大小的内存，并将此内存存储到单一连续块中，以优化性能。但此内存不属于UObject生态系统，也不属于虚幻引擎的反射系统，且无法通过垃圾回收查看。因此，`UPROPERTY` 支持将不可用，建议使用 `TWeakObjectPtr` 来存储可能需要的 `UObject` 指针。