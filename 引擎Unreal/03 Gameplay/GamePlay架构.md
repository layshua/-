[《InsideUE4》GamePlay架构（一）Actor和Component - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/22833151)

# 0 GamePlay 框架
以一个简单的游戏情景为例：兔子与蜗牛赛跑。

游戏框架的基础是 **`GameMode`**。**GameMode** 设置的是游戏规则，如首个跨过终点线的玩家即是冠军。其同时可生成玩家。

在 **`PlayerController`** 中设置一名玩家，其同时会产生一个 **Pawn**。
**`Pawn`** 是玩家在游戏中的物理代表，**控制器则拥有 Pawn 并设置其行为规则**。本范例中共有2个 Pawn，一个用于蜗牛而另一个用于兔子。
兔子实际为 **`Character`**，是 pawn 的一个特殊子类，拥有跑跳等内置移动功能。另一方面，蜗牛拥有不同的移动风格，可从 Pawn 类处直接延展。

Pawn 可包含自身的移动规则和其他游戏逻辑，但控制器也可拥有该功能。控制器可以是获取真人玩家输入的 PlayerController 或是电脑自动控制的 `AIController`。

在本范例中，玩家控制的是蜗牛，因此 PlayerController 拥有的是蜗牛 Pawn。
而 AI 则控制兔子，AIController 则拥有兔子角色，其中已设有停止、冲刺或打盹等行为。
相机提供的视角仅对真人玩家有效，因此PlayerCamera仅会使用蜗牛Pawn的其中一个CameraComponent。

进行游戏时，玩家的输出将使蜗牛在地图中四处移动，同时 HUD 将覆盖在相机提供的视角上，显示目前游戏中的第一名和已进行的游戏时间。

![[Pasted image 20230115210832.png]]
Object：虚幻引擎中对象的基类。所有其他类都是 Object 类的子类。
Actor：可以在关卡中放置或产生的对象所使用的基类。
Actor Component：所有组件的基类，这些组件定义可添加到 Actor 的可复用行为。
**因此，每个 Actor 都是 Object，但并非所有 Object 都是 Actor。例如，ActorComponent 是 Object，但不是 Actor。**
![[Pasted image 20230115222649.png]]
-   **继承**（空心箭头）
-   **聚合**（实心箭头），一个对象拥有另一个对象。Actor 拥有 Actor Component
蓝色都是 Actor，橘色是 Actor Component

## Actor
所有可以放入关卡的对象都是 **Actor**，比如摄像机、静态网格体、玩家起始位置。Actor 支持三维变换，例如平移、旋转和缩放。你可以通过游戏逻辑代码（C++或蓝图）创建（生成）或销毁 Actor。

在C++中，AActor是所有Actor的基类。

注意：Actor 不直接保存变换（位置、旋转和缩放）数据；如 Actor 的根组件存在，则使用它的变换数据。

### Actor 组件

组件被创建时与其包含的 Actor 相关联。

组件的主要类型有：

-    `UActorComponent`：最适用于抽象行为，例如移动、物品栏或属性管理，以及其他非物理概念。Actor 组件没有变换，即它们在场景中不存在任何物理位置或旋转。**不存在于场景中的任意特定位置。它们通常用于概念上的功能，如 AI 或解译玩家输入。**

-    `USceneComponent`：SceneComponents 是拥有变换的 ActorComponents。变换是场景中的位置，由位置、旋转和缩放定义。SceneComponents 能以层级的方式相互附加。Actor 的位置、旋转和缩放取自位于层级根部的 SceneComponent。支持基于位置的行为，这类行为不需要几何表示。这包括弹簧臂、摄像机、物理力和约束（但不包括物理对象），甚至音频。
    
-  `UPrimitiveComponent`：PrimitiveComponent 是拥有一类图像表达（如网格体或粒子系统）的 SceneComponent。诸多有趣的物理和碰撞设置均在此处。

Actor 支持拥有一个 SceneComponent 的层级。每个 Actor 也拥有一个 `RootComponent` 属性，将指定作为 Actor 根的组件。Actor 自身不含变换，因此不带位置、旋转，或缩放。它们依赖于其组件的变换，具体来说是其根组件的变换。**如果此组件是一个 SceneComponent，其将提供 Actor 的变换信息。否则 Actor 将不带变换。其他附加的组件拥有相对于其附加到的组件的变换。**

![[Pasted image 20230826161240.png]]


### Actor 通信

### Ticking

[Ticking](https://docs.unrealengine.com/5.2/zh-CN/actor-ticking-in-unreal-engine)代表Actor在虚幻引擎中的更新方式。所有Actor均能每帧tick，或以用户定义的最小间隔进行tick，以便执行必要的更新计算或操作。

所有Actor均可通过 `Tick()` 函数默认被tick。

**ActorComponents** 能够默认被更新，但其使用的是 `TickComponent()` 函数进行操作。 参见组件页面的[更新部分](https://docs.unrealengine.com/5.2/zh-CN/components-in-unreal-engine)了解详情。

### 生命周期

此文档是 **Actor** 生命周期的高级概述：Actor 如何被实例化（**生成**）到关卡中，以及如何被移除（**销毁**）。

以下流程图展示了 Actor 被实例的三种主要路径。无论 Actor 的创建方式如何，销毁路径均相同。

#### 生命周期详解

[![[363deeb792195fa4b90c00bbd9f8b116_MD5.jpg]]]( https://docs.unrealengine.com/5.2/Images/making-interactive-experiences/interactive-framework/actors/actor-lifecycle/ActorLifeCycle1.png )

#### 从磁盘加载

已位于关卡中的 Actor 使用此路径，如 LoadMap 发生时、或 AddToWorld（从流关卡或子关卡）被调用时。

1. 包/关卡中的 Actor 从磁盘中进行加载。
    
2. **PostLoad** - 在序列化 Actor 从磁盘加载完成后被调用。在此处可执行自定义版本化和修复操作。PostLoad 与 **PostActorCreated** 互斥。
    
3. **InitializeActorsForPlay**
    
4. 为未初始化的 Actor 执行 **RouteActorInitialize**（包含无缝行程携带）
    
    1. **PreInitializeComponents** - 在 Actor 的组件上调用 InitializeComponent 之前进行调用。
        
    2. **InitializeComponent** - Actor 上定义的每个组件的创建辅助函数。
        
    3. **PostInitializeComponents** - Actor 的组件初始化后调用。
        
5. **BeginPlay** - 关卡开始后调用。
    

#### Play in Editor

Play in Editor 路径与 Load from Disk 十分相似，然而 Actor 却并非从磁盘中加载，而是从编辑器中复制而来。

1. 编辑器中的 Actor 被复制到新场景中。
    
2. **PostDuplicate** 被调用。
    
3. **InitializeActorsForPlay**
    
4. 为未初始化的 Actor 执行 **RouteActorInitialize**（包含无缝行程携带）。
    
    1. **PreInitializeComponents** - 在 Actor 的组件上调用 InitializeComponent 之前进行调用。
        
    2. **InitializeComponent** - Actor 上定义的每个组件的创建辅助函数。
        
    3. **PostInitializeComponents** - Actor 的组件初始化后调用。
        
5. **BeginPlay** - 关卡开始后调用。
    

#### 生成

这是生成（实例）Actor 时的路径。

1. **SpawnActor** 被调用。
    
2. **PostSpawnInitialize**
    
3. **PostActorCreated** - 创建后即被生成的 Actor 调用，构建函数类行为在此发生。PostActorCreated 与 PostLoad 互斥。
    
4. **ExecuteConstruction**：
    
    - **OnConstruction** - Actor 的构建。蓝图 Actor 的组件在此处创建，蓝图变量在此处初始化
        
5. **PostActorConstruction**：
    
    1. **PreInitializeComponents** - 在 Actor 的组件上调用 InitializeComponent 之前进行调用。
        
    2. **InitializeComponent** - Actor 上定义的每个组件的创建辅助函数。
        
    3. **PostInitializeComponents** - Actor 的组件初始化后调用。
        
6. **OnActorSpawned** 在 UWorld 上播放。
    
7. **BeginPlay** 被调用。
    

#### 延迟生成

将任意属性设为"Expose on Spawn"即可延迟 Actor 的生成。

1. **SpawnActorDeferred** - 生成程序化 Actor，在蓝图构建脚本之前进行额外设置。
    
2. SpawnActor 中的所有操作发生；PostActorCreated 之后发生以下操作：
    
    1. 通过一个有效但不完整的 Actor 实例设置/调用多个"初始化函数"。
        
    2. **FinishSpawningActor** -调用后对 Actor 进行最终化，在 Spawn Actor 行中选取 ExecuteConstruction。
        

#### 生命走向终点

销毁 Actor 的方式有许多种，但终结其存在的方式始终如一。

##### 在游戏Gameplay期间

它们完全为任选，因为许多 Actor 在游戏进程中不会实际消亡。

**Destroy** - 游戏在 Actor 需要被移除时手动调用，但游戏进程仍在继续。Actor 被标记为等待销毁并从关卡的 Actor 阵列中移除。

**EndPlay** - 在数个地方调用，保证 Actor 的生命走向终点。在游戏过程中，如包含流关卡的 Actor 被卸载，Destroy 将发射此项和关卡过渡。调用 EndPlay 的全部情形：

- 对 Destroy 显式调用。
    
- Play in Editor 终结。
    
- 关卡过渡（无缝行程或加载地图）。 包含 Actor 的流关卡被卸载。
    
- Actor 的生命期已过。
    
- 应用程序关闭（全部 Actor 被销毁）。
    

无论这些情形出现的方式如何，Actor 都将被标记为 RF_PendingKill，因此在下个垃圾回收周期中它将被解除分配。此外，可以考虑使用更整洁的 `FWeakObjectPtr<AActor>` 代替手动检查"等待销毁"。

**OnDestroy** - 这是对 Destroy 的旧有反应。也许应该将这里的所有内容移到 EndPlay，因为它被关卡过渡和其他游戏清理函数调用。

#### 垃圾回收

一个对象被标记待销毁的一段时间后，垃圾回收会将其从内存中实际移除，释放其使用的资源。

在对象的销毁过程中，以下函数将被调用：

1. **BeginDestroy** - 对象可利用此机会释放内存并处理其他多线程资源（即为图像线程代理对象）。与销毁相关的大多数游戏性功能理应在 `EndPlay` 中更早地被处理。
    
2. **IsReadyForFinishDestroy** - 垃圾回收过程将调用此函数，以确定对象是否可被永久解除分配。返回 `false`，此函数即可延迟对象的实际销毁，直到下一个垃圾回收过程。
    
3. **FinishDestroy** - 最后对象将被销毁，这是释放内部数据结构的另一个机会。这是内存释放前的最后一次调用。
    

##### 高级垃圾回收

**虚幻引擎** 中的垃圾回收过程将构建共同被销毁对象的集群。较之于单个删除对象，集群可减少垃圾回收相关的总体时间和整体内存流失。可能随对象的加载创建子对象。将对象与其子对象组合到垃圾回收器的单个集群后，引擎可延迟释放集群使用的资源，直到整个对象可被释放时一次性释放全部资源。

多数项目中无需对垃圾回收进行配置或修改，但存在一些特定情况 - 可以如下方式对垃圾回收器的"集群"行为进行调整，以提高效率：

- **Clustering** - 关闭集群。在 **Project Settings** 中的 **Garbage Collection** 部分下，可将 **Create Garbage Collector UObject Clusters** 选项设为 false。对多数项目而言，此操作将导致垃圾回收效率降低，因此只建议在性能测试证明其绝对有益的情况下使用。
[![[08be5c512c7d1e7e6f3fdd9d48f96b20_MD5.jpg]]](https://docs.unrealengine.com/5.2/Images/making-interactive-experiences/interactive-framework/actors/actor-lifecycle/AdvancedGC.png)

垃圾回收的集群合并选项位于项目设置菜单中。

#### 复制

**复制** 用于在处理联网多人游戏时对场景中的Actor进行同步。属性值和函数调用均可被复制， 以便对客户端上游戏的状态进行完整控制。

#### 销毁Actor

Actor 通常不会被垃圾回收，因为场景对象保存一个 Actor 引用的列表。调用 `Destroy()` 即可显式销毁 Actor。这会将其从关卡中移除，并将其标记为"待销毁"，这说明其在下次垃圾回收中被清理之前都将存在。

## Level 与 World
一个或多个 Level 组成 World，每个 Level 保存当前所有的 Actors。
WorldSetting 并不是设置 World 的属性（不要混肴），是针对 Level 的设置。仅代表当前一个关卡，并不是所有关卡。

Level 作为 Actor 的容器，同时也划分了 World，一方面支持了 Level 的动态加载，另一方面也允许了团队的实时协作，大家可以同时并行编辑不同的 Level。一般而言，一个玩家从游戏开始到结束，UE 会创造一个 GameWorld 给玩家并一直存在。玩家切换场景或关卡，也只是在这个 World 中加载释放不同的 Level。

## GameMode 和GameState
两个主要类负责处理进行中游戏的相关信息：**Game Mode** 和 **Game State**。

即使最开放的游戏也拥有基础规则，而这些规则构成了 **Game Mode**。在最基础的层面上，这些规则包括：
- 出现的玩家和观众数量，以及允许的玩家和观众最大数量。
- 玩家进入游戏的方式，可包含选择生成地点和其他生成/重生成行为的规则。
- 游戏是否可以暂停，以及如何处理游戏暂停。
- 关卡之间的过渡，包括游戏是否以动画模式开场。
    

基于规则的事件在游戏中发生，需要进行追踪并和所有玩家共享时，信息将通过 **Game State** 进行存储和同步。这些信息包括：
- 游戏已运行的时间（包括本地玩家加入前的运行时间）。
- 每个个体玩家加入游戏的时间和玩家的当前状态。
- 当前 Game Mode 的基类。
- 游戏是否已开始。

### GameMode
**游戏模式 GameMode**
Game Modes 的任务是定义和实现规则。Game Modes 当前常用的基类有两个。
4.14 版本中加入了 `AGameModeBase`，这是所有 Game Mode 的基类，是经典的 `AGameMode` 简化版本。`AGameMode` 是 4.14 版本之前的基类，仍然保留，功能如旧，但现在是 `AGameModeBase` 的子类。
由于其比赛状态概念的实现，`AGameMode` 更适用于标准游戏类型（如多人射击游戏）。
`AGameModeBase` 简洁高效，是新代码项目中包含的全新默认游戏模式。

**一款游戏可拥有任意数量的 Game Mode**，因此也可拥有任意数量的 `AGameModeBase` 类子类；然而，**给定时间上只能使用一个 Game Mode**。每次关卡进行游戏实例化时 Game Mode Actor 将通过 `UGameEngine::LoadMap()` 函数进行实例化。

**Game Mode 不会复制到加入多人游戏的远程客户端；它只存在于服务器上**，因此本地客户端可看到之前使用过的留存 Game Mode 类（或蓝图）；但无法访问实际的实例并检查其变量，确定游戏进程中已发生哪些变化。
如玩家确实需要更新与当前 Game Mode 相关的信息，可将信息保存在一个 `AGameStateBase` Actor 上，轻松保持同步。`AGameStateBase` Actor 随 Game Mode 而创建，之后被复制到所有远程客户端。

默认设置：
![[Pasted image 20230115223623.png]]
在关卡编辑器的 WorldSetting 中重载，实际运行以重载的 GameMode 为准：
WorldSetting 并不是设置 World 的属性（不要混肴），是针对 Level 的设置。仅代表当前一个关卡，并不是所有关卡。
![[Pasted image 20230115223741.png|300]]

### GameState
**游戏状态 GameState，主要用于联机游戏**

**Game State** 负责启用客户端监控游戏状态。从概念上而言，Game State 应该管理所有已连接客户端已知的信息（特定于 Game Mode 但不特定于任何个体玩家）。它能够追踪游戏层面的属性，如已连接玩家的列表、夺旗游戏中的团队得分、开放世界游戏中已完成的任务，等等。

Game State 并非追踪玩家特有内容（如夺旗比赛中特定玩家为团队获得的分数）的最佳之处，因为它们由 **Player State** 更清晰地处理。**整体而言，GameState 应该追踪游戏进程中变化的属性。这些属性与所有人皆相关，且所有人可见。** 
**Game mode 只存在于服务器上，而 Game State 存在于服务器上且会被复制到所有客户端，保持所有已连接机器的游戏进程更新。**

## controller
**控制器（Controller）** 是一种可以控制 Pawn（或 Pawn 的派生类，例如角色类`Character`），从而控制其动作的非实体 Actor。

人类玩家使用 `PlayerController` 控制 Pawn，而 `AIController` 则对它们控制的 Pawn 实加 AI 效果。
控制器用 `Possess` 函数控制 Pawn，用 `Unpossess` 函数放弃控制 Pawn。

控制器会接收其控制的Pawn所发生诸多事件的通知。因此控制器可借机实现 响应此事件的行为，拦截事件并接替Pawn的默认行为。 可以让控制器在给定的Pawn之前运行， 从而从而最大限度减少输入处理与Pawn移动之间的延迟。

默认情况下，控制器与Pawn之间存在**一对一**的关系；也就是说，每个控制器在任何给定的时间只控制一个Pawn。这对于大多数 类型的游戏都是可以接受的，但对于某些类型的游戏可能需要进行调整，因为实时策略可能需要能够同时控制多个实体。
### ** PlayerController**
Pawn 和角色类如果由玩家控制器支配，则仅接收输入事件。

**PlayerController（玩家控制器）** 是 Pawn 和控制它的人类玩家间的接口。PlayerController 本质上代表了人类玩家的意愿。

当您设置 PlayerController 时，您需要考虑的一个事情就是您想在 PlayerController 中包含哪些功能及内容。
您可以在 **Pawn** 中处理所有输入，尤其是不太复杂的情况下。但是，**如果您的需求非常复杂，比如在一个游戏客户端上的多玩家、或实时地动态修改角色的功能，那么最好 `PlayerController` 中处理输入。** **在这种情况中，PlayerController 决定要干什么，然后将命令（比如"开始蹲伏"、"跳跃"）发布给 Pawn。**

同时，**某些情况下，则必须把输入处理或其他功能放到 PlayerController 中**。PlayerController 在整个游戏在过程中都是一直存在的，但是 Pawn 可能是临时存在的。比如，在死亡竞技模式的游戏中，您可能死了又重生，所以您将获得一个新的 Pawn，但是您的 PlayerController 都是一样的。在这个示例中，如果您将分数保存到您的 Pawn 上，那么分数将会重置，但是如果您将分数保存到 PlayerController 上，它将不会重置。

由玩家控制器支配的 Pawn 类可以在游戏中更改。下图来自于关卡蓝图，显示了支配 **Possess 函数** 的用法。在该示例中，当按下 Enter 键时，将由玩家控制器支配关卡中的 otherPawn。
![[Pasted image 20230115225037.png]]
## PlayerState
**玩家状态 PlayerState，主要用于联机游戏**

玩家状态类用于记录特定玩家的信息，这些信息在多人游戏中需要与其他客户端共享。
玩家控制器仅存在于客户端上，而玩家状态会从服务器复制到所有客户端。
要使用基于玩家状态类的新蓝图，必须在游戏模式的玩家状态类参数中设置。
## Pawn
Pawn 是可以由控制器（(玩家或 Al)控制的 Actor。**Pawn 类表示身体，控制器类表示头脑。**
下图显示了从 Pawn 类继承的一些参数。Pawn 类可以使用支配它的控制器的旋转值。
其他属性表示控制器如何支配 Pawn。
![[Pasted image 20230115225722.png]]
## Character
![[Pasted image 20230115225829.png]]
角色类是 Pawn 类的子类，它用来表示可以行走、奔跑、跳跃、游泳和飞翔的两足角色。该类已经有一组用来帮助达到这一目的的组件。
右图显示了角色类中存在的组件。CapsuleComponent 用于碰撞测试。ArrowComponent 表示角色的当前方向。
Mesh 组件是用于视觉呈现角色的骨架网格体。Mesh 组件的动画由动画蓝图来控制。
CharacterMovement 组件用于定义各种类型的角色运动，如行走、奔跑、跳跃、游泳和飞翔。
CharacterMovement 组件是用 C++编写的，用于处理运动以及多人游戏中的复制和预测。
各种类型的运动都有很多属性可以在组件上调整。

## GameInstance
**GameInstance 游戏实例**
游戏实例类的实例会在游戏开始时创建，并仅在游戏关闭时移除。
关卡中的所有 Actor 和其他对象会完全销毁，并在每次关卡加载时重新产生。
游戏实例类和它包含的数据在各个关卡之间保持不变。游戏实例类仅存在于每个客户端上，不进行复制。
要分配游戏中使用的游戏实例类，前往"编辑”(Edit)>“项目设置”(Project Settings)>“地图和模式”(MapsModes)修改项目设置。
![[Pasted image 20230115230553.png]]
