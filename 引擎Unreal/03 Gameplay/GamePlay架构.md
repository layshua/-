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

### 
## Level 与 World
一个或多个 Level 组成 World，每个 Level 保存当前所有的 Actors。
WorldSetting 并不是设置 World 的属性（不要混肴），是针对 Level 的设置。仅代表当前一个关卡，并不是所有关卡。

Level 作为 Actor 的容器，同时也划分了 World，一方面支持了 Level 的动态加载，另一方面也允许了团队的实时协作，大家可以同时并行编辑不同的 Level。一般而言，一个玩家从游戏开始到结束，UE 会创造一个 GameWorld 给玩家并一直存在。玩家切换场景或关卡，也只是在这个 World 中加载释放不同的 Level。
## GameMode
**游戏模式 GameMode**
默认设置：
![[Pasted image 20230115223623.png]]
在关卡编辑器的 WorldSetting 中重载，实际运行以重载的 GameMode 为准：
WorldSetting 并不是设置 World 的属性（不要混肴），是针对 Level 的设置。仅代表当前一个关卡，并不是所有关卡。
![[Pasted image 20230115223741.png|300]]
GameMode 类用于定义游戏的规则。
GameMode 还指定将用于创建 Pawn、玩家控制器、游戏状态、HUD 和其他类的默认类, 如上图。
每个关卡都可以有不同的 GameMode。如果不为关卡指定 GameMode，则将使用已经设置好的默认 GameMode。在多人游戏中，游戏模式仅存在于服务器上，不会复制到客户端。
## GameState
**游戏状态 GameState，主要用于联机游戏**

游戏状态类用于记录表示游戏当前状态的变量，在多人游戏中与所有客户端共享。
基本思路是通过游戏模式在服务器上定义规则，而游戏状态则管理游戏中更改的信息，并需要发送给客户端。要使用基于游戏状态类的新蓝图，必须将自定义游戏状态类分配给游戏模式的游戏状态类参数。
游戏状态类是从游戏状态基类延伸而来，会增加一些多人功能。
## PlayerController
**玩家控制器 PlayerController**

控制器类拥有两个主要子类。玩家控制器类由人类玩家使用，Al 控制器类使用人工智能来控制 Pawn。
Pawn 和角色类如果由玩家控制器支配，则仅接收输入事件。
由玩家控制器支配的 Pawn 类可以在游戏中更改。下图来自于关卡蓝图，显示了支配 **Possess 函数** 的用法。在该示例中，当按下 Enter 键时，将由玩家控制器支配关卡中的 otherPawn。
![[Pasted image 20230115225037.png]]
## PlayerState
**玩家状态 PlayerState，主要用于联机游戏**

玩家状态类用于记录特定玩家的信息，这些信息在多人游戏中需要与其他客户端共享。
玩家控制器仅存在于客户端上，而玩家状态会从服务器复制到所有客户端。
要使用基于玩家状态类的新蓝图，必须在游戏模式的玩家状态类参数中设置。
## Pawn
Pawn 是可以由控制器（(玩家或 Al)控制（支配)的 Actor。**Pawn 类表示身体，控制器类表示头脑。**
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
