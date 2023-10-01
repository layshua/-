---
title: UE网络精粹
create_time: 2023-10-01 17:51
uid: "202310011751"
reference: []
banner: "[[1696179112450.png]]"
banner_header: 
banner_lock: true
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

# GamePlay架构与网络
## GameMode（服务器）
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

## GameState（客户端+服务器）

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

## PlayerState （客户端+服务器）

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

## Pawn / Character

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

## PlayerController
The class **APlayerController** might be the most interesting and complicated class that we come across. It's also the center for a lot of client logic since this is the first class that the client actually 'owns'.  
APlayerController 类可能是我们遇到的最有趣、最复杂的类。它也是大量客户端逻辑的中心，因为**这是客户端真正 "拥有 (owns) "的第一个类**。

The PlayerController can be seen as the 'Input' of the player. It is the link of the player with the server. This further means every client has one PlayerController.  
PlayerController 可以看作是玩家的 "输入"。它是玩家与服务器的链接。这进一步意味着每个客户端都有一个 PlayerController。  
A client's PlayerController only ever exists on their end, as well as on the server. A client cannot access other clients' PlayerControllers.  
客户端的 PlayerController 只存在于客户端和服务器端。客户端无法访问其他客户端的 PlayerController。

**Every client only knows about their own PlayerController!  
每个客户端只知道自己的 PlayerController！**

The result of that is that the server has a reference of all client PlayerControllers!  
这样做的结果是，服务器拥有所有客户端 PlayerControllers 的引用！

The term 'Input' does not directly mean that all actual Input (Button Presses, Mouse Movement, Controller Axis, etc.) needs to be placed in the PlayerController.  
输入 "一词并不直接意味着所有实际输入（按键、鼠标移动、控制器轴等）都需要放在 PlayerController 中。

It is a good practice to place Pawn/Character specific Input (cars work differently than humans) into your APawn/ACharacter classes and to place Input that should work with all Characters, or even when the Character object is not valid, into your PlayerController.  
一个好的做法是，将 "棋子"/"角色 "特定的输入（汽车的工作方式与人类不同）放入 APawn/ACharacter 类中，而将适用于所有角色的输入，或甚至当角色对象无效时，放入 PlayerController 中。

**Furthermore, an important thing to know is:  
此外，还有一件重要的事情需要了解：**

_How do I get the correct PlayerController?  
如何获取正确的 PlayerController？_

The famous node 'GetPlayerController(0)' or code line 'UGameplayStatics::GetPlayerController(GetWorld(), 0);' works differently on the server and clients.  
著名的节点 "GetPlayerController(0) "或代码行 "UGameplayStatics::GetPlayerController(GetWorld(), 0); "在服务器和客户端上的工作方式不同。

- Calling it on the Listen-Server will return the Listen-Server's PlayerController  
    在监听服务器上调用它将返回监听服务器的 PlayerController
- Calling it on a Client will return the Client's PlayerController  
    在客户端上调用它将返回客户端的 PlayerController
- Calling it on a Dedicated Server will return the first Client's PlayerController  
    在专用服务器上调用它将返回第一个客户端的 PlayerController

Other numbers than '0' will not return other clients' PlayerControllers for a client. This index is meant to be used for local players (splitscreen), which we won't cover here.  
除 "0 "以外的其他数字将不会返回某个客户端的其他客户端 PlayerControllers。该索引用于本地玩家（分屏），我们在此不做介绍。

## Examples and Usage[​](https://cedric-neukirchen.net/docs/multiplayer-compendium/common-classes/playercontroller#examples-and-usage "Direct link to Examples and Usage") 示例和用法

Even though the APlayerController is one of the most important classes for networking, there isn't much to it by default.  
尽管 APlayerController 是网络中最重要的类之一，但默认情况下它的功能并不多。

So we will create a small example just to make clear why it's needed. In the chapter about ownership, you will read about why the PlayerController is important for RPCs.  
因此，我们将创建一个小示例来说明为什么需要它。在 "所有权 "一章中，你会了解到为什么 PlayerController 对于 RPC 非常重要。

The following example will show you how to utilize the PlayerController to increment a replicated variable in the GameState by pressing a UserWidget button.  
下面的示例将向您展示如何利用 PlayerController，通过按下 UserWidget 按钮来递增 GameState 中的一个复制变量。

_Why do we need the PlayerController for this?  
为什么需要使用 PlayerController？_

Well, I don't want to write down the RPC and Ownership chapter twice, so just a short explanation:  
好吧，我不想把 RPC 和所有权章节写两遍，所以就简单解释一下吧：

UserWidgets only exist on the local player (being a client or a ListenServer) and even if they are owned by the client a ServerRPC has no instance of them on the server to run on.  
UserWidgets 只存在于本地播放器（客户端或 ListenServer）上，即使它们被客户端拥有，ServerRPC 也无法在服务器上运行它们的实例。

It's simply **not** replicated!  
根本无法复制！

This means we need a way to get the button Press over to the server so it can then increment the variable.  
这意味着我们需要一种方法，将按钮 Press 发送到服务器，这样服务器就可以递增变量。

_Why not call the RPC on the GameState directly?  
为什么不直接调用 GameState 上的 RPC？_

Because it's owned by the server. A ServerRPC needs the client as the owner!  
因为它归服务器所有。ServerRPC 需要客户端作为所有者！

### Blueprint[​](https://cedric-neukirchen.net/docs/multiplayer-compendium/common-classes/playercontroller#blueprint "Direct link to Blueprint") 蓝图

So first of all, we need a simple UserWidget with a button that we can press.  
因此，首先，我们需要一个简单的 UserWidget，上面有一个可以按下的按钮。

I will post the images in the opposite order, so you can see where it ends and what events call the events of the previous images.  
我将以相反的顺序张贴图片，这样你就能看到图片的结尾，以及哪些事件呼应了前面图片中的事件。

So starting with our goal, the GameState. It gets a normal event that increments a replicated integer variable:  
因此，从我们的目标 GameState 开始。它会收到一个普通事件，该事件会递增一个复制的整数变量：
![[Pasted image 20231001200904.png]]
This event will get called on the server side, inside of our ServerRPC in our PlayerController:  
该事件将在服务器端调用，就在我们的 PlayerController 中的 ServerRPC 内部：
![[Pasted image 20231001200911.png]]
And at last, we have our button, which gets pressed and calls the ServerRPC:  
最后，我们的按钮被按下并调用 ServerRPC：
![[Pasted image 20231001200919.png]]

So when we click on the button (client side), we use the ServerRPC in our PlayerController to get to the server side (possible, because the PlayerController is owned by the client!) and then call the 'IncreaseVariable' event of the GameState to increment the replicated integer variable.  
因此，当我们点击按钮（客户端）时，我们使用 PlayerController 中的 ServerRPC 来进入服务器端（这是可能的，因为 PlayerController 是客户端所有的！），然后调用 GameState 的 "IncreaseVariable "事件来递增复制的整数变量。

This integer variable, since it is replicated and set by the server, will now update on all instances of the GameState so that clients can also see the update!  
由于这个整数变量是由服务器复制和设置的，因此现在会在 GameState 的所有实例上更新，这样客户端也能看到更新！

#### UE++[​](https://cedric-neukirchen.net/docs/multiplayer-compendium/common-classes/playercontroller#ue "Direct link to UE++") UE++

For the C++ version of this example, I will replace the UserWidget with the BeginPlay of the PlayerController. This doesn't make much sense, however, implementing UserWidgets in C++ needs some more code which I don't want to post here.  
在本例的 C++ 版本中，我将用 PlayerController 的 BeginPlay 代替 UserWidget。不过，用 C++ 实现 UserWidget 需要更多代码，我不想在此赘述。

```c++ file:TestPlayerController.h
// Server RPC. You will read more about this in the RPC chapter  
UFUNCTION(Server, unreliable, WithValidation)  
void Server_IncreaseVariable();  
  
// Also overriding the BeginPlay function for this example  
virtual void BeginPlay() override;
```

```c++ file:TestGameState.h
// Replicated integer variable
UPROPERTY(Replicated)
int32 OurVariable;

public:
// Function to increment the variable
void IncreaseVariable();
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

    // BeginPlay is called on every instance of an Actor, so also on the server version of this PlayerController.
    // We want to ensure, that only the local player calls this RPC. Again, this example doesn't necessarily make much sense
    // since we could just flip the condition and wouldn't need the RPC at all, but C++ Widget, you know...
    // We could also use "IsLocalPlayerController()" here
    if (Role < ROLE_Authority)
    {
        Server_IncreaseVariable();
    }
}
```

```c++ file:file:TestGameState.cpp
// This function is required and the replicated specifier in the UPROPERTY macro causes it to be declared for us. We only need to implement it
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

That's quite some code. If you don't understand the use of some of the functions and their naming yet, don't worry. The upcoming sections will help you understand why it's done like this.  
这是相当多的代码。如果你还不理解其中一些函数的用法和命名，不用担心。接下来的章节将帮助你理解为什么要这样做。