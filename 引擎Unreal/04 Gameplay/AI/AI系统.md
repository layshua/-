---
title: AI系统
create_time: 2023-09-11 21:21
uid: "202309112121"
banner: "![[]]"
reference: []
---

# AI 感知系统
除可用于决定所执行逻辑的[行为树](https://docs.unrealengine.com/5.2/zh-CN/behavior-trees-in-unreal-engine)，以及用于获取环境信息的[场景查询系统（EQS）](https://docs.unrealengine.com/5.2/zh-CN/environment-query-system-in-unreal-engine)之外，AI 框架中可用于为 AI 提供感官数据的另一个工具是 **AI 感知系统（AI Perception System）** 。通过 AI 感知组件实现。

## AIPerception 感知组件
AI 组件允许 Pawn 感知周围环境中的数据，例如噪声来源位置、或 Pawn 能够看到某个对象。
![[Pasted image 20230912133213.png]]
- **`AIPerception`** 组件相当于刺激源的监听器，用于收集已注册的刺激信号。
- **`AI Perception Stimuli Source`** 组件自动将 Actor 注册成为感知系统中指定感官的一个刺激源。 
- 刺激源被注册后将调用 AIPerception 组件的 **`On Perception Updated`** （或用于目标选择的 **`On Target Perception Updated`** ）事件

### 感知配置
![[Pasted image 20230912132010.png]]
>可以配置多个感官，并且指定一个**主导感官（Dominant Sense）**，该感官应优先于其他感官。

![[Pasted image 20230912132405.png]]
- <font color="#ff0000">Al Damage sense config 伤害</font>：对伤害事件（如 **Event Any Damage** 、 **Event Point Damage** 或 **Event Radial Damage** ）作出反应。
- <font color="#ff0000">Al Hearing config 听觉</font>：检测由 **报告噪点事件（Report Noise Event）** 产生的声音，例如发射物击中某物发出的声音
- <font color="#ff0000">Al Prediction sense config 感知</font> ：这要求感知系统（Perception System）在 PredictionTime 秒内向请求者提供 PredictedActor 的预计位置。
- <font color="#ff0000">Al Sight config 视觉</font>：决定着 AI 角色在关卡中所能"看见"的事物。当一个 Actor 进入 **视觉半径** 后，AI 感知系统将发出更新信号，并穿过被看到的 Actor（举例而言，一个玩家进入该半径，并被具备视觉感知的 AI 所察觉）。
- <font color="#ff0000">Al Team sense config 团队</font>：通知感知组件的拥有者同团队中有人处在附近（发送该事件的游戏代码也会发送半径距离）。
- <font color="#ff0000">Al Touch config 触觉</font>：检测到 AI 与物体发生主动碰撞，或是与物体发生被动碰撞。举例而言，在潜入类型的游戏中，你可能希望玩家在不接触敌方 AI 的情况下偷偷绕过他们。使用此感官可以确定玩家与 AI 发生接触，并能用不同逻辑做出响应。


- <font color="#ff0000">Detection by Affiliation</font> 从属检测：默认情况下 **Actor** 不会被指定归属，会被视为中立（neutrals）。

> [!NOTE]
> 目前，你还不能通过蓝图分配归属，因此为了检测玩家，我们要启用 **检测中立（Detect Neutral）** 标签。另外一种方法是使用 **Actor 标签（Actor Tagging）** 来确定哪个角色是玩家，并强制 AI 角色只追逐被标记为玩家的 Actor。

### 感知函数调用

以下函数可以通过蓝图调用，进而从感知系统获取信息，或对感知系统产生影响。

|函数|描述|
|---|---|
|**Get Actors Perception**|获取针对给定Actor的感知，并返回被感知Actor的数据结构。|
|**Get Currently Perceived Actors**|返回基于给定感官被感知的全部Actor。如果未指定感官，则返回当前以任意方式感知到的所有Actor。|
|**Get Known Perceived Actors**|返回基于给定感官被感知的（且尚未被遗忘）的所有Actor。如果未指定感官，则被感知到的所有Actor均将被返回。|
|**Get Perceived Hostile Actors**|返回敌方Actor列表（发出的刺激已被感知且未失效或已成功感知的所有敌方Actor)。本方法可以在蓝图中重写，返回用户所需的任意Actor列表。|
|**Request Stimuli Listener Update**|手动强制AI感知系统更新指定目标的刺激监听器的属性。|
|**Set Sense Enabled**|启用或禁用指定的 **感官类** 。<br><br>只有在针对目标组件实体配置给定感官后，该设置才有效。|

### 刺激源配置
![[Pasted image 20230912133432.png]]

还可以指定基于 **AISense** 类的所有自定义感官。
### 刺激函数调用
以下函数可以通过 **AI 感知刺激源** 组件的蓝图调用：

|函数|描述|
|---|---|
|**注册为感官（Register for Sense）**|将拥有Actor注册为指定感官类的刺激源。|
|**注册到感知系统（Register with Perception System）**|将拥有Actor注册为 **注册为感官源（Register as Source for Senses）** 属性中所指定感官的刺激源，并通过 **Register for Sense** 函数调用。<br><br>如果启用了自动注册为源（Auto Register as Source）属性，则不需要调用此函数。|
|**从感知系统中注销（Unregister from Perception System）**|取消将拥有Actor注册为感知刺激源。|
|**从感知中注销（Unregister from Sense）**|

## PawnNoiseEmitter 噪声发射器组件
![[Pasted image 20230912134050.png]]
**`PawnNoiseEmitterComponent`** 追踪 **`PawnSensingComponents`** 使用的噪声事件数据来监听 Pawn。该组件主要存在于 Pawn 或其控制器上。它在网络客户端上不进行任何操作。

## PawnSensing 感应组件

Pawn 的感应组件（`Sensing Component`）用于封装 Actor 的感知（例如视觉和听觉）设置及功能，以便 Actor 在游戏世界中观察/监听 Pawn。其在网络客户端上不进行任何操作。

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

**服务节点**：[服务](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-behavior-tree-node-reference-services)节点是与任意合成节点（选择器节点、序列节点或简单平行节点）相关联的一种特殊节点。只有在合成节点的分支被执行，它们就会以定义的频率执行。**常用于检查和更新黑板**。

它**能够针对指定秒数的每个回调进行注册，并能对多种需要周期性出现的类型进行更新。**
举例而言，当 AI Pawn 面对当前敌人、继续在其行为树中正常行动时，可以使用服务节点为该 Pawn 确定最适合追逐的敌人。

1. **默认聚焦(Default Focus）** 通过设置控制器的聚焦来创建访问 **蓝图** 和代码中 Actor 的快捷方式。**将 AIController 的聚焦设置到 Actor 上后，你便能直接从 AI 控制器对其进行访问，而不需要访问黑板键。**
2. **运行 EQS（Run EQS）** 服务节点可用于**以指定的时间间隔定期执行场景查询系统（EQS）模板，并可对指定的黑板键进行更新。**

## 任务节点 Tasks
**任务节点的功能是实现操作，例如移动 AI 或调整黑板值**。它们可以连接至装饰器节点或服务节点。

![[Pasted image 20230912112520.png]]

- <font color="#ff0000">Finish with Result</font>：**以结果完成** 任务节点可用于在完成时即时输出一个给定结果。该节点会基于所定义的结果强制分支结束或继续。
- <font color="#ff0000">Make Noise</font>：如果受控Pawn拥有 **PawnNoiseEmitter** 组件，**发出噪音（Make Noise）** 任务将使Pawn"产生噪声"（发送消息），使其它拥有 **PawnSensing** 组件的Pawn听到（接收消息）。
- <font color="#ff0000">Move Directly Toward</font>：**直接移动至**任务节点会将 AI Pawn 沿直线移向指定的 Actor 或位置（矢量）黑板条目，而不参考任何导航系统。如果需要 AI 按导航移动，请改用 **移动至（Move To）** 任务节点。
- <font color="#ff0000">Move To</font>：**移动至**任务将使拥有角色移动组件的Pawn使用**NavMesh**移动至矢量黑板键。
- <font color="#ff0000">Play Animation</font>：用于播放指定的动画资源。
- <font color="#ff0000">Play Sound</font>：播放指定的音效。
- <font color="#ff0000"><font color="#ff0000">Push Pawn Action</font></font>：**推送 Pawn 动作**节点使您能够将指定的动作推送至 Pawn 的控制器。
- <font color="#ff0000">Rotate to Face BBEntry</font>：**旋转至面向黑板条目**任务节点会使关连 Pawn 向指定的黑板键旋转。( Pawn 必须启用 Use Controller Rotation Yaw )
- <font color="#ff0000">Run Behavior</font>：**运行行为**任务节点将分支树推送到执行堆栈上，从而运行另一个行为树。（给子树的根等级装饰器节点提供支持。在运行时不能改变该子树资源，在运行时不能修改运行树的结构。如果您需要在运行时能修改的子树，使用 Run Behavior Dynamic）
- <font color="#ff0000">Run Behavior Dynamic</font>：**运行动态行为**任务节点能够在执行堆栈上推送子树。使用 **行为树组件** 上的 **SetDynamicSubtree** 函数即可以在运行时分配子树资源。（本函数不会为子树的根等级装饰器节点提供支持。）
- <font color="#ff0000">Run EQSQuery</font>：将运行指定的场景查询系统（EQS）资源。
- <font color="#ff0000">Set Tag Cooldown</font>：设置 **冷却标签（Cooldown Tag）** 数值，并与 **冷却标签装饰器（Cooldown Tag Decorators）** 一起使用，从而防止行为树的执行。
- <font color="#ff0000">Wait</font>：使树在此节点上等待，直至指定的 **等待时间（Wait Time）** 结束。
- <font color="#ff0000">Wait Blackboard Time</font>：与 **等待（Wait）** 任务节点的原理类似，由黑板键盘指定时间

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

`UBTNode` 中的许多虚函数都将 `uint8*` 参数带到节点的内存中。此参数指示为代理分配的内存块，内存块大小将由 `GetInstanceMemorySize` 的覆盖版本返回。节点将为各个代理分配此大小的内存，并将此内存存储到单一连续块中，以优化性能。但此内存不属于 UObject 生态系统，也不属于虚幻引擎的反射系统，且无法通过垃圾回收查看。因此，`UPROPERTY` 支持将不可用，建议使用 `TWeakObjectPtr` 来存储可能需要的 `UObject` 指针。

# 寻路系统

**虚幻引擎寻路系统** 用于为人工智能代理（AI Agent）提供寻路功能。

为了帮助AI确定起点和终点之间的路径，引擎会根据场景中的碰撞体生成寻路网格体。这种简化的多边形网格体代表了关卡中的可寻路空间。默认情况下，寻路网格体会细分为多个区块，允许你重新构建寻路网格体的局部区域。

生成的网格体由多个多边形组成，每个多边形都有一个"成本"值。搜索路径时，寻路算法会尝试找到总成本最低的路径。

---

寻路系统包含各种组件以及可修改寻路网格体生成方式的设置，例如指定给多边形的成本。这进而影响代理在你的关卡中寻路的方式。你还可以将寻路网格体中不连续的区域连接起来，如平台和桥梁。

寻路系统包含三种 **生成模式（Generation Modes）**：**静态（Static）**、**动态（Dynamic）** 和 **仅限动态修改器（Dynamic Modifiers Only）**。这些模式控制了项目中生成寻路网格体的方式，并提供了各种选项来满足你的需要。

该系统还为代理提供了两种规避方法：**相对速度障碍物(RVO)（Reciprocal Velocity Obstacles (RVO)）** 和 **大规模人群绕行避让管理器（Detour Crowd Manager）**。这些方法允许代理在游戏过程中绕行，避让动态障碍物和其他代理。

![[Pasted image 20230912114543.png]]
注意寻路网格体在楼梯上无法正确绘制，可以调整绘制偏移让覆盖面更广。
![[Pasted image 20230912114535.png]]

重要节点（由事件驱动）：
![[Pasted image 20230912121741.png]]
## 修改寻路系统
- 寻路修饰体积 ![[Pasted image 20230912121943.png|246]] ：使用区域类（Area class）来确定体积内寻路的 **默认成本（Default Cost）** 乘数。从而改变特定区域中的 Nav Mesh 成本，让指定区域可达或不可达。

> [!NOTE] 区域类
>  ![[Pasted image 20230912125311.png]]
>  - 区域类还定义了 **固定区域进入成本（Fixed Area Entering Cost）**，这是代理进入该区域时采用的初始成本。你可以根据需要创建任意数量的区域类来影响代理如何寻路关卡。
>  - 可以创建 Area Class 蓝图自定义区域类![[Pasted image 20230912125453.png|350]]

- 寻路查询筛选器（Navigation Query Filters）： 包含有关一个或多个区域类的信息，如有需要，可以重载成本值。你可以根据需要创建任意数量的查询筛选器来进一步自定义代理如何寻路关卡。[自定义寻路区域和查询筛选器](https://docs.unrealengine.com/5.2/zh-CN/overview-of-custom-navigation-areas-and-query-filters-in-unreal-engine/)
- 寻路链接代理 ![[Pasted image 20230912121957.png|196]] ：能将寻路网格体中没有直接寻路路径的两个区域连接起来。在搜索路径的同时，寻路链接代理会用作额外连接，供代理用于到达目的地。![[Pasted image 20230912122840.png]]
- 自定义蓝图，使用**智能链接代理**以允许代理在平台之间朝着目标跳跃或执行其他动作[教程链接](https://docs.unrealengine.com/5.2/zh-CN/overview-of-how-to-modify-the-navigation-mesh-in-unreal-engine/)
- 自定义蓝图，在运行时**动态生成 Nav Mesh**[教程链接](https://docs.unrealengine.com/5.2/zh-CN/overview-of-how-to-modify-the-navigation-mesh-in-unreal-engine/)

## 避障机制
[在虚幻引擎寻路系统中使用避障机制 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/using-avoidance-with-the-navigation-system-in-unreal-engine/)

寻路机制可以在静态对象周围生成路径，而避障算法主要用于**处理移动障碍物**。
AI 代理有两种方法来绕开移动障碍物，或在彼此间避障，分别是 **相对速度障碍物算法（Reciprocal Velocity Obstacles，即 RVO）** 和 **群组绕行管理器（Detour Crowd Manager）**。

1. **相对速度障碍物算法** 系统会计算每个代理的速度向量，避免和附近的其他代理碰撞。该系统会查看附近的代理，并假定它们在计算的每一步内都以恒速移动。根据代理向目标移动的速度，会选择最佳的速度向量进行匹配。
    - RVO 不使用寻路网格体进行避障，因此它无需寻路系统即可用于代理。该系统包含在角色类的 **角色移动（Character Movement）** 组件中。![[Pasted image 20230912130029.png]]

2. **群组绕行管理器（Detour Crowd Manager）** 系统**通过自适应 RVO 采样计算来解决代理之间的规避问题**。它会计算一个粗略的速度采样，并且着重在代理的移动方向，相较于传统的 RVO 规避方式，**显著提升规避性能**。该系统还使用可见度和拓扑路径优化项，进一步提升碰撞规避。
    - 群组绕行管理器系统可以高度配置特定示例模式选项、最大代理数和代理半径。该系统包括在 **群组绕行 AI 控制器（DetourCrowd AI Controller）** 类中，可以和任意 Pawn 类一起使用。

**两种系统各自独立工作，在你的项目中只能使用其中一种。**

|方法|描述|局限性|
|---|---|---|
|相对速度障碍物算法|- 代理使用特定半径内的速度向量规避障碍物。<br>    <br>- 包含在角色类的角色移动（Character Movement）组件中。|- 相较于群组绕行管理器更难配置。<br>    <br>- 仅限于角色类中。<br>    <br>- 不使用寻路网格体进行规避，因此代理可能会任性地"出走"。|
|群组绕行管理器|- 代理通过路径优化和特定半径内的速度向量规避障碍物。<br>    <br>- 包含在群组绕行AI控制器（DetourCrowd AI Controller）类中。\| 在项目设置中定义了固定的最大代理数量。|- 在项目设置中定义了固定的最大代理数量。|

## 寻路调用程序
**寻路调用程序（Navigation Invokers）** 是在运行时在代理周围生成寻路网格体的蓝图 Actor 组件。**使用寻路调用程序，就无需在编辑器中构建寻路网格体，并且还可以限制在运行时生成的图块数。**
寻路调用程序非常适合大型关卡，因为在编辑器中构建寻路网格体不切实际。
[使用虚幻引擎中的寻路调用程序 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/using-navigation-invokers-in-unreal-engine/)

## 优化 NavMesh 生成速度
[优化虚幻引擎寻路网格体的生成速度 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/optimizing-navigation-mesh-generation-speed-in-unreal-engine/)

## 世界分区静态导航网格
[世界分区导航网格 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/world-partitioned-navigation-mesh/)


# MassEntity（鸽）

MassEntity 是一个重点围绕游戏逻辑打造的框架，用于面向数据的计算。**基于 ECS**
[虚幻引擎MassEntity | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/mass-entity-in-unreal-engine/)

# 智能对象（鸽）
**智能对象（Smart Object）** 是可以放置在关卡中与AI代理和玩家交互的对象。这些对象包含交互所需的所有信息。
[虚幻引擎智能对象概述 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/smart-objects-in-unreal-engine---overview/)

# StateTree（鸽）

**StateTree** 是一种通用分层状态机，组合了行为树中的 **选择器（Selectors）** 与状态机中的 **状态（States）** 和 **过渡（Transitions）** 。使用 StateTree，你可以创建非常高效、保持灵活且井然有序的逻辑。

[虚幻引擎中的StateTree | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/state-tree-in-unreal-engine/)

# 环境查询系统 EQS（鸽）
**场景查询系统（EQS）** 是虚幻引擎5（UE5） AI 系统的一个功能，可将其**用于从环境中收集数据，获取环境信息**。在 EQS 中，可以通过不同种类的测试向收集的数据提问，这些测试会根据提出问题的类型来生成最适合的项目。 

可以从[行为树](https://docs.unrealengine.com/5.2/zh-CN/behavior-trees-in-unreal-engine)中调用 EQS 查询，并根据测试的结果将其用于后续操作的决定。EQS 查询主要由[生成器](https://docs.unrealengine.com/5.2/zh-CN/eqs-node-reference-generators-in-unreal-engine)节点（用于生成将被测试及加权的位置或 Actor）和[情境](https://docs.unrealengine.com/5.2/zh-CN/eqs-node-reference-contexts-in-unreal-engine)节点（被用作各种测试和生成器引用的框架）组成。**可以用 EQS 查询指引 AI 角色找到能够发现玩家并发起攻击的最佳位置、找到距离最近的体力值或弹药拾取物，或找到最近的掩体（以及其他可进行的动作）。**
[虚幻引擎中的场景查询系统 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/environment-query-system-in-unreal-engine/)
