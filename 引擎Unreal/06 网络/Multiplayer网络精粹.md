---
title: UE网络精粹
create_time: 2023-10-01 17:51
uid: "202310011751"
reference: []
banner: "[[1696179112450.png]]"
banner_header: 
banner_lock: true
banner_icon: 🗺
---

# 虚幻中的网络
虚幻引擎使用标准的**客户端 - 服务器 （Client-Server）架构**。这意味着服务器是**权威（Authoritative）** 的，所有数据必须首先从客户端发送到服务器。之后，服务器验证数据并根据您的代码做出反应。

## 一个小例子​

当您作为客户端在多人游戏中移动角色时，您不会自己移动角色，而是告诉服务器您想要移动它。然后，服务器会为其他人（包括您）更新角色的变换。

> [!info] 
>
> 此外，为了防止本地客户端有 “滞后” 的感觉，程序员通常还让本地客户端直接控制他们的角色——尽管当客户端开始作弊时，服务器仍然可能覆盖角色的位置！这意味着客户端（几乎）永远不会直接与其他客户端“交谈”。

##  另一个例子​

When sending a chat message to another client you are sending it to the server first, which then passes it to the client you wanted to reach. This could also be a team, guild, group, etc.  
当向另一个客户端（个人、公会、队伍等）发送聊天消息时，您首先将其发送到服务器，然后服务器将其传递给您想要联系的客户端。

Important 重要的

**Never** trust the client! Trusting the client here means you don't test the client's actions before executing them.  

> [!danger] 
> 永远不要相信客户端！信任客户端意味着您在执行客户端的操作之前不会测试它们。
这会允许他们作弊！
一个简单的例子是发射武器：确保在服务器上测试客户端是否拥有所需数量的弹药，之后再允许射击而不是直接处理射击！

# GamePlay 架构 + 网络
## 1 架构总结
根据前面关于虚幻引擎的 CS 架构和常用类的信息，我们可以将虚幻类分为四类： 
- **Server Only** -  仅服务器 - 这些对象只存在于服务器上
- **Server & Clients** - 服务器和所有客户端 - 这些对象存在于服务器和所有客户端中
- **Server & Owning Client** - 服务器和拥有客户端（即本地客户端） - 这些对象只存在于服务器和拥有客户端上
- **Owning Client Only** - 仅拥有客户端，这些对象只存在于拥有客户端上

>**拥有客户端（Owning Client）** 是指拥有相关 Actor 的 player/client。就像你拥有自己的电脑一样。所有权（Ownership）对于后面章节中的 "RPC "非常重要。

**下面两幅图向您展示了一些常见的类别，以及它们属于哪些类别。**
![[8a1f656c0ccacdb389055b2e883b707f_MD5.svg|"Common Classes layed out in the four sections mentioned above."]]

第二幅图展示了一个有两个连接客户端的专用服务器（dedicated server）的示例。

![[27b08f9e7b9ed6bc9c57882c9cbe197a_MD5.svg|"Venn Diagram of the Classes in a Dedicated Server with two connected Clients example."]]


## 2 GameMode（仅服务器）
> [!NOTE]
> 在 4.14 中，AGameMode 类分为 AGameModeBase 和 AGameMode。 GameModeBase 的功能较少，因为某些游戏可能不需要旧 AGameMode 类的完整功能列表。
> 

AGameMode 类用于定义游戏规则。这包括要生成的其他游戏框架类，例如 APawn、APlayerController、APlayerState 等。

**它仅在服务器上可用。客户端没有 AGameMode 类的实例，并且在尝试检索它时只会得到 nullptr。**

### 示例和用法​

游戏模式的一些用例可能来自较早的第一人称射击游戏，例如《虚幻竞技场》：

**Deathmatch**, **Team Deathmatch** or **Capture the Flag**.  
死亡竞赛、团队死亡竞赛或夺旗。

这意味着 GameMode 可以定义如下内容：

* 团队赛还是个人赛？
- 获胜条件是什么？
    *   杀敌数到达多少胜利？
* 积分是如何获得的？
    *   杀人？
    *   夺旗？
*   将使用什么角色？
*    允许携带哪些武器？
    * 只有手枪吗？
    * 只有刀？

对于多人游戏场景，GameMode 还具有一些有趣的功能，可以帮助我们管理玩家和比赛的总体流程。


#### 函数

GameMode蓝图的 Override 函数部分：
![[cbde69313852572dab83dc34073ca75f_MD5.png]]

您可以实现这些函数的逻辑，以适应您的游戏的特定规则。 这包括更改 GameMode 生成 DefaultPawn 的方式或您想要如何决定游戏是否已准备好开始。

**一个例子可能是检查所有玩家是否已加入服务器并准备好：**
- @ 蓝图：
![[Pasted image 20231001162035.png]]
>玩家数到达最大玩家数时返回 true

- @ C++：
由于 `ReadyToStartMatch` 是 `BlueprintNativeEvent`，因此该函数的实际 C++ 实现称为 `ReadyToStartMatch_Implementation`。这是我们想要覆盖的：

```c++ file:MyGameMode.h
// 本场比赛所需/允许的最大Player人数
int32 MaxNumPlayers;

virtual bool ReadyToStartMatch_Implementation() override;
```

```c++ file:MyGameMode.cpp
bool ATestGameMode::ReadyToStartMatch_Implementation()
{
    Super::ReadyToStartMatch();

    return MaxNumPlayers == NumPlayers;
}
```

---
**但也有一些事件可以用来对整个比赛中发生的某些事情做出反应。**
我经常使用的一个很好的例子是事件 `OnPostLogin`。每次新玩家加入游戏时都会调用此方法。该事件会向您传递一个有效的 PlayerController 引用，该 Controller 由连接玩家的 UConnection 拥有（稍后也会详细介绍）。

- @ 蓝图
![[Pasted image 20231001160607.png|300]]
![[Pasted image 20231001162106.png]]

 - @ C ++
`OnPostLogin` 函数是虚函数，在 C++ 中简称为 `PostLogin`
```c++ file:MyGameMode.h
// List of PlayerControllers
UPROPERTY()
TArray<APlayerController*> PlayerControllerList;

// Overriding the PostLogin function
virtual void PostLogin(APlayerController* NewPlayer) override;
```

```c++ file:MyGameMode.cpp
void ATestGameMode::PostLogin(APlayerController* NewPlayer)
{
    Super::PostLogin(NewPlayer);

    PlayerControllerList.Add(NewPlayer);
}
```
这可以用于与该玩家进行交互，例如，为他们生成一个新的 Pawn，或者只是将其 PlayerController 保存在数组中以供以后使用。

正如已经提到的，您可以使用 GameMode 来管理游戏的一般比赛流程。为此，您可以找到一些功能，其中一些功能是可覆盖的，例如`Ready To Start Match`。

这些函数和事件可用于控制当前的 `MatchState（匹配状态）`。当“`Ready To Start Match`”函数返回 **TRUE** 时，它们中的大多数将被自动调用，但您也可以手动使用它们。
![[Pasted image 20231001162147.png]]
>“`New State`”是一个简单的“FName”类型。您现在可能会问，“为什么这不在 AGameState 类中处理？”嗯，确实如此。这些 GameMode 函数与 GameState 协同工作。
**这只是为了给您一个点来管理任何客户端都无法访问的 `MatchState`，因为 GameMode 只存在于服务器上！**

#### 变量​

这是已经继承的变量的列表。其中一些可以通过 GameMode 蓝图的 ClassDefaults 进行设置：

![[80d227b1ee73d70c373d2144046809bd_MD5.png]]

![[ac948e6651eb07cead5c485776186c8c_MD5.png]]

其中大多数命名都很直白，例如`Default Player Name`，它使您能够为每个连接的玩家提供一个可以通过 `APlayerState` 类访问的默认玩家名称。
还有 `bDelayedStart`，这将使游戏无法开始，即使 `Ready To Start Match` 的默认实现满足所有其他条件。 

更重要的变量之一是 `Options String`。这些是选项，用“`?`”分隔，您可以通过`OpenLevel`函数或当您将`ServerTravel`作为控制台命令调用时传递这些选项。 

您可以使用 `Parse Option` 来提取传递的选项，例如`MaxNumPlayers`：  
![[Pasted image 20231001163900.png|400]]
![[Pasted image 20231001164757.png]]

## 3 GameState（所有客户端+服务器）

> [!info] 
> 在 4.14 中，GameState 类被分为 AGameStateBase 和 AGameState。 GameStateBase 的功能较少，因为某些游戏可能不需要旧 GameState 类的完整功能列表。

AGameState 类可能是服务器和客户端之间共享信息的最重要的类。
GameState 用于跟踪游戏 / 比赛的当前状态。对于多人游戏来说，这**包括已连接玩家的列表 (`APlayerState`)。**

**此外，它会复制给所有客户端，因此每个人都可以访问它**。这使得 GameState 成为多人游戏中信息方面最为核心的类之一。 
  
虽然 GameMode 会告诉您需要多少杀敌数才能获胜，但 GameState 将跟踪每个玩家和 / 或团队当前的杀敌数！

您在这里存储什么信息完全取决于您。它可以是得分数组或自定义结构数组

###  示例和用法​

在多人游戏中，AGameState 类用于跟踪游戏的当前状态，其中还包括**玩家及其 PlayerState**。

GameMode 确保调用 GameState 的 `MatchState` 函数，并且 **GameState 本身也允许您在客户端上使用它们**。

与 GameMode 相比，GameState 并没有给我们太多的帮助，但这仍然允许我们创建我们的逻辑，**该逻辑主要应该尝试将信息传播给客户端。**

#### 变量​

![[460a118ae262e844e17ac30050cdac94_MD5.png]]

我们从 AGameState 基类中获取一些可以利用的变量。 PlayerArray、MatchState 和 ElapsedTime 都会被复制，因此客户端也可以访问它们。
>`AuthorityGameMode` 除外。只有服务器可以访问它，因为 GameMode 仅存在于服务器上。

**PlayerArray 不会直接复制，但是，每个 PlayerState 都会被复制，并且它们会在构造时将自己添加到 PlayerArray 中**。此外，它们由 GameState 收集，只是为了确保竞争条件不会导致问题。

以下是 C++代码示例，展示了将 PlayerState 收集到 PlayerArray 中的快速插入方法： 
PlayerState 类本身的内部
```c++ 
void APlayerState::PostInitializeComponents()
{
    // […]

    UWorld* World = GetWorld();
    // Register this PlayerState with the Game's ReplicationInfo
    if (World->GameState != NULL)
    {
        World->GameState->AddPlayerState(this);
    }

    // […]
}
```

并且在 GameState 中
```c++
void AGameState::PostInitializeComponents()
{
    // […]

    for (TActorIterator<APlayerState> It(World); It; ++It)
    {
        AddPlayerState(*It);
    }
}

void AGameState::AddPlayerState(APlayerState* PlayerState)
{
    if (!PlayerState->bIsInactive)
    {
        PlayerArray.AddUnique(PlayerState);
    }
}

```

所有这一切都发生在服务器以及 Player 和 GameState 的客户端实例上！

#### 示例
我可以为您提供的一个小的函数示例是**跟踪 “A” 和“B”两支球队的得分。** 假设我们有一个 `CustomEvent`，当球队得分时会调用该事件。

它传递一个布尔值，这样我们就知道哪支球队得分了。我们还可以传递 PlayerState、Team 或任何您用来识别得分者的信息。

稍后在 “Replication 复制” 章节中，您将了解**只有服务器可以（并且应该）复制变量的规则**，因此我们确保只有服务器可以调用此事件。

该事件是从另一个类调用的（例如杀死某人的武器），并且这应该发生在服务器上（总是！），因此我们在这里不需要 RPC。

- @ 蓝图
![[Pasted image 20231001173813.png]]
由于这些变量和 GameState 是复制的，因此您可以使用这两个变量并将它们放入您需要的任何其他类中。例如，将它们显示在记分板 widget 中。 

- @ C++

为了重新创建这个例子，我们需要更多的代码，但是除了函数本身之外，设置复制所需的代码只需要每个类一次。

```c++ file:MyGameState.h
// You need this included to get the replication working.
#include “UnrealNetwork.h”

// Replicated specifier used to mark this variable to replicate
UPROPERTY(Replicated)
int32 TeamAScore;

UPROPERTY(Replicated)
int32 TeamBScore;

// Function to increase the score of a team
void AddScore(bool bTeamAScored);
```

```c++ file:MyGameState.cpp
void ATestGameState::AddScore(bool bTeamAScored)
{
    if (bTeamAScored)
    {
        TeamAScore++;
    }
    else
    {
        TeamBScore++;
    }
}
```

## 4 PlayerState （所有客户端+服务器）

`APlayerState` 类是**共享特定玩家信息的最重要的类**。它旨在保存有关玩家的当前信息。**每个玩家都有自己的 PlayerState**。

**PlayerState 也会复制给每个人，并可用于在其他客户端上检索和显示数据。
访问所有 PlayerState 的一个简单方法是 AGameState 类中的 `PlayerArray`。**

您可能想要存储在 PlayerState 中的示例信息：
- PlayerName - 玩家的当前名称
- Score - 玩家当前的分数
- Ping - 玩家当前的 ping
- TeamID - 玩家所在团队的 ID
- 或其他玩家可能需要了解的其他复制信息

### 示例和用法​

我能提供的大多数例子都非常具体。因此，我们将看看一些已经可用的属性，以及一些更有趣的函数。

#### 蓝图示例​

蓝图暴露了一些变量，它们或多或少有用。遗憾的是，其中一些并未公开其所有函数，因此最好用您自己的函数替换它们。

![[104ed35e4fa0a9d27583ee86b3d4d64f_MD5.png]]
>**这些变量都会被复制，因此它们在所有客户端上保持同步。**

遗憾的是，它们在蓝图中不容易设置，但没有什么可以阻止您创建它们的版本。

设置 PlayerName 变量的一个示例是通过调用 GameMode 函数`ChangeName`，并将其传递给玩家的 PlayerController。
![[Pasted image 20231001174629.png]]

PlayerState 还用于**确保数据在无缝关卡更改或意外连接问题期间保持持久性。**

PlayerState 有两个专门用于处理重新连接玩家和与服务器无缝切换到新地图的玩家的功能。**PlayerState 负责将其已保存的信息复制到新的 PlayerState 中**。这要么是通过关卡更新创建的，要么是因为玩家重新连接而创建的。
![[Pasted image 20231001175334.png]]

- @ c++实现
让我们看一下 C++ 中的相同函数。
```c++ file:TestPlayerState.h
// Used to copy properties from the current PlayerState to the passed one
virtual void CopyProperties(class APlayerState* PlayerState) override;

// Used to override the current PlayerState with the properties of the passed one
virtual void OverrideWith(class APlayerState* PlayerState) override;
```
这些函数可以在您自己的 C++ PlayerState 子类中实现，以管理您添加到自定义 PlayerState 的数据。确保在末尾添加“override”说明符，并调用“Super::”，以便原始实现保持活动状态。

您的实现可能与此类似：
```c++ file:TestPlayerState.cpp
void ATestPlayerState::CopyProperties(class APlayerState* PlayerState)
{
    Super::CopyProperties(PlayerState);

    if (IsValid(PlayerState))
    {
        ATestPlayerState* TestPlayerState = Cast<ATestPlayerState>(PlayerState);
        if (IsValid(TestPlayerState))
        {
            TestPlayerState->SomeVariable = SomeVariable;
        }
    }
}

void ATestPlayerState::OverrideWith(class APlayerState* PlayerState)
{
    Super::OverrideWith(PlayerState);

    if (IsValid(PlayerState))
    {
        ATestPlayerState* TestPlayerState = Cast<ATestPlayerState>(PlayerState);
        if (IsValid(TestPlayerState))
        {
            SomeVariable = TestPlayerState->SomeVariable;
        }
    }
}
```

## 5 Pawn / Character（所有客户端+服务器）

PlayerController 一次只能拥有一个 Pawn，但可以通过-possess  和 unpossess 来轻松切换 Pawn。

**Pawn 大部分被复制到所有客户端。**

Pawn 的子类 ACharacter 经常被使用，因为它带有一个已经**联网**的 MovementComponent，用于处理复制玩家角色的位置、旋转等。

### 示例和用法​


在多人游戏中，我们主要使用 Pawn 的 `Replication` 部分来显示角色并与其他人共享一些信息。一个简单的例子是角色的“Health”。  

我们不仅仅复制“Health”以使其对其他玩家可见，我们还复制它以使服务器对其具有权限，以防止客户端作弊。

#### 蓝图​

尽管有标准的可重载函数，Pawn 也有两个事件可以让您对它被 PlayerController 或 AIController 拥有时做出反应。
![[Pasted image 20231001180252.png|300]]

> [!NOTE] 
>- 由于 possess 逻辑发生在服务器上，这些事件仅在 Pawn/Character 的服务器版本上调用。
>- `ReceiveControllerChanged` 事件：在 Controller 变更时后调用，在客户端和服务端都能调用。

下图将展示如何使用 `EventAnyDamage` 函数和复制的 `Health` 变量来降低玩家的生命值。
>这发生在服务器上而不是客户端上！

![[Pasted image 20231001181826.png]]

由于 Pawn 应该被复制，只要服务器调用 DestroyActor 节点，它也会销毁 Pawn 的客户端版本。  

在客户端站点上，我们可以将复制的“`Health`”变量用于 HUD 或每个人头顶上的健康栏。您可以通过创建带有 ProgressBar 和对 Pawn 的引用的 UserWidget 来轻松完成此操作。

假设我们的“BP_Character”类上有一个“Health”和“MaxHealth”变量，全部设置为**复制**（如果 MaxHealth 永远不会运行时改变，您可以不设置复制）。

现在，在 UserWidget 和 ProgressBar 内部创建“BP_Character”引用变量后，我们可以将该条的百分比绑定到以下函数：

![Health Bar Settings]( https://cedric-neukirchen.net/assets/images/health_bar_settings-00ef62de597e323724e84acad719a3a1.png "Health Bar Settings")

![[Pasted image 20231001182325.png]]

此外，在设置 WidgetComponent 后，我​​们可以将“Widget Class To Use”设置为您的 HealthBar UserWidget，并在 BeginPlay 上执行以下操作：

![[Pasted image 20231001182420.png]]

**“BeginPlay”在 Pawn 的所有实例上（服务器和所有客户端上）调用

所以现在每个实例都将自己设置为它所拥有的 UserWidget 的 Pawn 引用。

由于 Pawn 和生命值变量被复制，我们在每个 Pawn 的头部上方都有正确的百分比。

![[Pasted image 20231001182430.jpg]]

#### C++

对于 C++ 示例，我不会重新创建 UserWidget 示例。要让 UserWidgets 在 C++ 中工作需要做太多的模板式的东西，我不想在这里讨论这个。

所以我们将重点关注占有和伤害事件。在 C++中，两个 Possess 事件被称为：
```c++
virtual void PossessedBy(AController* NewController);

virtual void UnPossessed();
```
>注意，`UnPossessed` 事件不会传递旧的 PlayerController。

And we also want to recreate the Health example in C++. As always, if you don't understand the steps of replication at this moment, don't worry, the upcoming chapters will explain it to you.  
我们还想用 C++ 重新创建 Health 示例。如果您现在不明白复制的步骤，请不要担心，接下来的章节将为您解释。
>如果示例在复制方面看起来太复杂，请暂时跳过这些示例。

“`TakeDamage`”函数相当于“`EventAnyDamage`”节点。为了造成伤害，您通常会对要对其造成伤害的 Actor 调用“TakeDamage”，如果该 Actor 实现了该函数，它将对此做出反应，类似于本示例的做法。

```c++ file:TestPawn.h
// Replicated Health variable
UPROPERTY(Replicated)
int32 Health;

// Overriding the TakeDamage event
virtual float TakeDamage(float Damage, struct FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser) override;
```

```c++ file:TestPawn.cpp
// 该函数是必需的，UPROPERTY 宏中的Replicated指示符会为我们声明该函数。我们只需实现它
void ATestPawn::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    // 告诉 UE 我们要复制这个变量
    DOREPLIFETIME(ATestPawn, Health);
}

float ATestPawn::TakeDamage(float Damage, struct FDamageEvent const& DamageEvent, AController* EventInstigator, AActor* DamageCauser)
{
    const float ActualDamage = Super::TakeDamage(Damage, DamageEvent, EventInstigator, DamageCauser);

    // Lower the Health of the Player
    Health -= ActualDamage;

    // And destroy it if the Health is less or equal 0
    if (Health <= 0.f)
    {
        Destroy();
    }

    return ActualDamage;
}
```

## 6 PlayerController （拥有客户端+服务器）

APlayerController 类可能是我们遇到的最有趣、最复杂的类。它也是大量客户端逻辑的中心，因为**这是客户端真正 "拥有 (owns) "的第一个类**。

PlayerController 可以看作是玩家的 "`Input`"。它是玩家与服务器的链接。这进一步意味着**每个客户端都有一个 PlayerController**。  

**客户端的 PlayerController 只存在于本客户端和服务器**：
- 每个客户端**只知道**自己的 PlayerController，**无法访问其他客户端的 PlayerController。**
- 服务器拥有**所有**客户端 PlayerControllers 的引用！

"Input"一词并不直接意味着所有实际输入（按键、鼠标移动、控制器轴等）都需要放在 PlayerController 中。一个好的做法是：
- 将 Pawn/Character **特定**的输入（汽车的工作方式与人类不同）放入 APawn/ACharacter 类中
- 将适用于**所有** Character 或甚至当 Character 对象无效时的输入，放入 PlayerController 中。 

> [!question] 
> 如何获取正确的 PlayerController？

节点 `GetPlayerController(0)` 或代码行 `UGameplayStatics::GetPlayerController(GetWorld(), 0);` 在服务器和客户端上的工作方式不同：
- 在监听服务器（Listen-Server）上调用它将返回**监听服务器**的 PlayerController
- 在客户端上调用它将返回**客户端**的 PlayerController
- 在专用服务器（Dedicated Server）上调用它将返回**第一个客户端**的 PlayerController

>除 "0 "以外的其他数字将不会返回某个客户端的其他客户端 PlayerControllers。该索引用于本地玩家（分屏），我们在此不做介绍。

### 示例和用法

尽管 APlayerController 是网络中最重要的类之一，但默认情况下它的功能并不多。

因此，我们将创建一个小示例来说明为什么需要它。在 "所有权 (ownership) "一章中，你会了解到**为什么 PlayerController 对于 RPC 非常重要。**

下面的示例将向您展示如何利用 PlayerController，通过按下 UserWidget 按钮来递增 GameState 中的一个复制变量。

> [!question] 为什么需要使用 PlayerController？
> UserWidgets 只存在于本地播放器（客户端或 ListenServer）上，即使它们被客户端拥有，ServerRPC 也无法在服务器上运行它们的实例。它根本无法复制！
>
这意味着我们需要一种方法，将 button Press 发送到服务器，这样服务器就可以递增变量。
>
>RPC 和所有权章节会有详细介绍！

> [!question] 为什么不直接调用 GameState 上的 RPC？
> 因为它归服务器所有。ServerRPC 需要客户端作为所有者！

#### 蓝图

因此，首先，我们需要一个简单的 UserWidget，上面有一个可以按下的按钮。


我将以相反的顺序张贴图片，这样你就能看到图片的结尾，以及哪些事件呼应了前面图片中的事件。

因此，从我们的目标 GameState 开始。它会收到一个普通事件，该事件会递增一个复制的整数变量：
![[Pasted image 20231001200904.png]]

该事件将在服务器端调用，就在我们的 PlayerController 中的 ServerRPC 内部：
![[Pasted image 20231001200911.png]]

最后，我们的按钮被按下并调用 ServerRPC：
![[Pasted image 20231001200919.png]]

因此，当我们点击按钮（客户端）时，我们**使用 PlayerController 中的 ServerRPC 来进入服务器端**（这是可能的，因为 PlayerController 是客户端所有的！），然后调用 GameState 的 "IncreaseVariable "事件来递增复制的整数变量。

由于这个整数变量是由服务器复制和设置的，因此现在会在 GameState 的所有实例上更新，这样客户端也能看到更新！

##### C++

```c++ file:TestGameState.h
// Replicated integer variable
UPROPERTY(Replicated)
int32 OurVariable;

public:
// Function to increment the variable
void IncreaseVariable();
```

```c++ file:file:TestGameState.cpp
//此函数是必需的，并且UPROPERTY宏中复制的说明符会为我们声明它。我们只需要实现它
void ATestGameState::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    // This tells UE that we want to replicate this variable
    DOREPLIFETIME(ATestGameState, OurVariable);
}

void ATestGameState::IncreaseVariable()
{
    OurVariable++;
}
```

在本例的 C++ 版本中，我将用 PlayerController 的 BeginPlay 代替 UserWidget。不过，用 C++ 实现 UserWidget 需要更多代码，我不想在此赘述。

```c++ file:TestPlayerController.h
// Server RPC. You will read more about this in the RPC chapter  
UFUNCTION(Server, unreliable, WithValidation)  
void Server_IncreaseVariable();  
  
// Also overriding the BeginPlay function for this example  
virtual void BeginPlay() override;
```

```c++ file:TestPlayerController.cpp
// Otherwise we can't access the GameState functions
#include “TestGameState.h”

// You will read later about RPCs and why '_Validate' is a thing
bool ATestPlayerController::Server_IncreaseVariable_Validate()
{
    return true;
}

// You will read later about RPCs and why '_Implementation' is a thing
void ATestPlayerController::Server_IncreaseVariable_Implementation()
{
    ATestGameState* GameState = Cast<ATestGameState>(UGameplayStatics::GetGameState(GetWorld()));
    GameState->IncreaseVariable();
}

void ATestPlayerController::BeginPlay()
{
    Super::BeginPlay();

    //BeginPlay在Actor的每个实例上都被调用，在该PlayerController的服务器版本上也是如此。
    //我们希望确保，只有本地player调用此RPC。同样，这个例子不一定有多大意义
    //因为我们可以翻转条件，根本不需要RPC，但是C++Widget，你知道。。。
    //我们也可以在这里使用“IsLocalPlayerController（）”
    if (Role < ROLE_Authority)
    {
        Server_IncreaseVariable();
    }
}
```

这是相当多的代码。如果你还不理解其中一些函数的用法和命名，不用担心。接下来的章节将帮助你理解为什么要这样做。

## 7 AHUD（仅拥有客户端）
**AHUD 类仅在每个客户端上可用，可通过 PlayerController 访问。它将由 PlayerController 自动生成。**

在 UMG（虚幻动态图形）发布之前，AHUD 类一直用于在客户端的视口中绘制文本、纹理等。

**现在，UserWidgets 在 99% 的情况下都取代了 HUD 类。**

您仍然可以使用 AHUD 类进行调试，或者使用一个独立区域来管理 UserWidget 的创建、显示、隐藏和销毁。

>由于 HUD 与多人游戏没有直接联系，因此示例只能显示单人游戏的逻辑，所以本课将跳过这些示例。

## 8 UUserWidget （仅拥有客户端）

UUserWidgets 用于 Epic Games 的用户界面系统，该系统被称为**虚幻动态图形（UMG，Unreal Motion Graphics）**。

它们**继承自 Slate**，Slate 是一种用于在 C++ 中创建用户界面的语言，同时也用于虚幻引擎编辑器本身。

Widgets are only available locally. They don't replicate and should not contain and replication code. Preferably they wouldn't contain any gameplay code either, but some games might require it.  
Widgets**只能在本地使用**。它们**不会复制，也不应包含复制代码**。**它们最好也不包含任何游戏代码**，但有些游戏可能需要。
 
要了解有关 UMG 和小工具的更多信息，请使用上面提供的 API 链接。

在 APawn 示例中，我们已经有一个使用 Widgets 的小例子。因此，我将在此略过。



# 专用服务器与监听服务器
## Dedicated Server 专用服务器

专用服务器是**不需要玩家托管的独立服务器。**

它与游戏客户端分离运行，主要用于运行一个服务器，玩家可以随时加入/离开，而服务器不会随之关闭。

专用服务器可在 Windows 和 Linux 下编译，也可在云服务器上运行，玩家可通过固定 IP 地址连接到云服务器。

专用服务器没有视觉（visual）部分，因此不需要 UI，也没有 PlayerController 。它们在游戏中也没有 Character 或类似的代表。  
 
## Listen Server 监听服务器

监听服务器是指同时也是客户端的服务器。（用自己的电脑开服务器，还能同时玩游戏~）

由于同时也是客户端，Listen-Server 需要 UI，并有一个代表客户端部分的 PlayerController。**在监听服务器上获取 `PlayerController(0)`将返回该客户端的  PlayerController 。**

**由于监听服务器在客户端上运行，其他人需要连接的 IP 就是客户端的 IP。与专用服务器相比，这往往会带来玩家没有静态 IP 的问题。**

不过，使用 OnlineSubsystem（稍后解释）可以解决更改 IP 的问题。
# 复制 Replication
##  简介

Replication 是服务器将信息 / 数据传递给客户端的行为。

这可以仅限于特定的实体和组。蓝图大多根据受影响 AActor 的设置执行复制。

**第一个可以复制属性的类是 AActor 类**。虽然您也可以复制 UObject，但它们是通过 AActor 复制的，因此仍然需要某种 AActor 来处理复制。
UActorComponent 就是一个很好的例子，它支持通过 AActor 复制 UObjects，而不需要我们做太多额外的工作。

**前面提到的所有类都在某种程度上继承自 AActor，从而使它们能够在需要时复制属性。不过，并非所有类的复制方式都相同。**
例如，**AGameMode 根本不会复制**，因为只存在于服务器上。**而 AHUD、UUserWidget 只存在于客户端，也不会复制。**

## 如何使用 Replication
![[Pasted image 20231001225526.png|450]]
复制可以在 AActor 子类的类默认设置 / 构造函数中激活：

Character构造函数示例

```c++
ATestCharacter::ATestCharacter(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer)
{
    bReplicates = true;
    bReplicateMovement = true;
}
```

- **如果一个 Actor 的 `bReplicates` 设置为 `true`，那么该角色将被生成并复制到所有客户端（如果该角色是由服务器生成的）。而且只有在服务器生成时才会复制。**
- **如果客户端生成了这个 Actor，该 Actor 将只存在于这个客户端上。**

##  复制属性
![[Pasted image 20231001225820.png|500]]

启用复制后，我们可以在 Actor 内部复制变量。有多种方法可以做到这一点。我们将从最基本的方法开始：

**将 "复制" 下拉菜单设置为 "`Replication`"，将确保此变量被复制到此 Actor 的所有复制实例中。**
变量可以在某些条件下复制。下面我们将进一步讨论。

![[Pasted image 20231001230029.png|298]]
>Replicated 变量用两个白圈标出。

在 C++ 中复制变量所需的工作稍多一些：


```c++ file:TestPlayerCharacter.h
// Create replicated health variable
UPROPERTY(Replicated)
float Health;
```

.cpp 文件将获得 `GetLifetimeReplicatedProps` 函数。在将变量标记为复制时，UE 已经为我们创建了该函数的头声明。

**在此函数中，您可以定义复制变量的规则。**

```c++ file:TestPlayerCharacter.cpp
void ATestPlayerCharacter::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    // 这里列出我们想要复制的变量
    DOREPLIFETIME(ATestPlayerCharacter, Health);
}
```

您也可以在这里进行有条件复制（对应蓝图中的复制条件）：

```c++
// 仅向该Object/Class的所有者复制变量 
DOREPLIFETIME_CONDITION(ATestPlayerCharacter, Health, COND_OwnerOnly);
```

| Condition 条件                          |说明|
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| COND_InitialOnly                        |该属性只会尝试在初始串（initial bunch）上发送|
| COND_OwnerOnly COND_OwnnerOnly          | 该属性只会发送给演员的所有者（owner）|
| COND_SkipOwner                          |此属性会发送给所有连接，但 owner 除外 |
| COND_SimulatedOnly                      |此属性只会发送到模拟的（simulated） Actors|
| COND_AutonomousOnly                     |该属性只会发送给自主行为autonomous Actor|
|COND_SimulatedOrPhysics | This property will send to simulated OR **bRepPhysics** Acto 该属性将发送到模拟 OR bRepPhysics Acto。                                                                                     |
|COND_InitialOrOwner| This property will send on the **initial bunch**, or to the **Actor's owner** 该属性将在初始束上发送，或发送给 Actor 的所有者                                                             |
| COND_Custom                             | This property has no particular condition, but wants the ability to toggle on/off via **SetCustomIsActiveOverride** 该属性没有特定条件，但希望能够通过 SetCustomIsActiveOverride 切换开关 |


It's important to understand that the whole replication process only works from Server to client and **NOT** the other wayround.  
重要的是要明白，整个复制过程只能从服务器到客户端，而不能反过来。

We will learn later how to get the server to replicate something that the client wants to share with others (for example their PlayerName).  
我们稍后将学习如何让服务器复制客户希望与他人共享的内容（例如他们的播放器名称）。

A different way to replicate a variable is to mark it as "ReplicatedUsing". In Blueprints this is called "RepNotify". It allows specifying a function that gets called on the client when the new value of the variable is replicated to them.  
复制变量的另一种方法是将其标记为 "ReplicatedUsing"。在 Blueprints 中，这被称为 "RepNotify"。它允许指定一个函数，当变量的新值被复制到客户端时，该函数将被调用。

![[fdaa3d70c111079ebb5e43e3f4b423e4_MD5.png]]

In Blueprints this function will be created automatically once you select “RepNotify” in the "Replication" Drop-Down menu:  
在 Blueprints 中，一旦在 "复制" 下拉菜单中选择 "RepNotify"，该功能就会自动创建：

![[8ac9f0165edf7c23cdd8d66b1f1a3e81_MD5.png]]

The C++ version needs a bit more but works the same:  
C++ 版本需要的更多，但工作原理相同：

Header file inside of the classes declaration  
类声明内的头文件

```
// Create RepNotify Health variableUPROPERTY(ReplicatedUsing=OnRep_Health)float Health;// Create OnRep function | UFUNCTION() Macro is important! | Doesn't need to be virtualUFUNCTION()virtual void OnRep_Health();
```

CPP file of the class  
类的 CPP 文件

```
void ATestCharacter::OnRep_Health(){    if (Health <= 0.f)    {        PlayDeathAnimation();    }}
```

With 'ReplicatedUsing=FUNCTIONNAME', we specify the function that should get called when the variable is successfully replicated. This function needs to have the 'UNFUNCTION ()' macro, even if the macro is empty!  
通过 "ReplicatedUsing=FUNCTIONNAME"，我们指定了变量复制成功后应调用的函数。该函数必须包含 "UNFUNCTION ()" 宏，即使该宏为空！

Rep Notify deffirence between C++ and Blueprints  
代表通知 C++ 和蓝图之间的区别

It's important to note here that C++ and Blueprints handle RepNotify slightly differently. In C++, RepNotify functions only call for the clients.  
值得注意的是，C++ 和 Blueprints 对 RepNotify 的处理方式略有不同。在 C++ 中，RepNotify 函数只调用客户端。

When a server changes the value and requires the OnRep function to call too, you will need to call it manually after adjusting the variable. That's because the OnRep function is meant as a callback for when the variable is replicated to the client.  
当服务器更改值并要求同时调用 OnRep 函数时，您需要在调整变量后手动调用该函数。这是因为 OnRep 函数的作用是在变量复制到客户端时进行回调。

In Blueprints, however, the OnRep function will call for clients **and** server. That's because the BP version of OnRep is a"Property Changed" callback. This means that the function will call for the server too, but also for the client if the client changes the variable locally.  
但在 Blueprints 中，OnRep 函数将调用客户端和服务器。这是因为 BP 版本的 OnRep 是 "属性已更改" 回调。这意味着该函数不仅会调用服务器，而且如果客户端在本地更改了变量，也会调用客户端。|  ||  ||  ||  |