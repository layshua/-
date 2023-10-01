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

![PlayerState Variables](data:image/png;base64,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 "PlayerState Variables")
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

## 示例和用法​


在多人游戏中，我们主要使用 Pawn 的 `Replication` 部分来显示角色并与其他人共享一些信息。一个简单的例子是角色的“Health”。  

我们不仅仅复制“Health”以使其对其他玩家可见，我们还复制它以使服务器对其具有权限，以防止客户端作弊。

### Blueprint[​](https://cedric-neukirchen.net/docs/multiplayer-compendium/common-classes/pawn#blueprint "Direct link to Blueprint") 蓝图​

尽管有标准的可重载函数，Pawn 也有两个事件可以让您对它被 PlayerController 或 AIController 拥有时做出反应。
![[Pasted image 20231001180252.png|300]]

> [!NOTE] 
由于 possess 逻辑发生在服务器上，这些事件仅在 Pawn/Character 的服务器版本上调用。 

还有一个名为 `ReceiveControllerChanged` 的​​函数，它允许您对相同的事件做出反应，但在客户端。


## Examples and Usage[​](https://cedric-neukirchen.net/docs/multiplayer-compendium/common-classes/pawn#examples-and-usage "Direct link to Examples and Usage") 示例和用法​

In Multiplayer we mostly use the Replication Part of the Pawn to display the Character and share some information with others. A simple example is the 'Health' of a Character.  
在多人游戏中，我们主要使用 Pawn 的复制部分来显示角色并与其他人共享一些信息。一个简单的例子是角色的“健康”。  
But we don't only replicate 'Health' to make it visible for others, we also replicate it so that the Server has authority over it and the Client can't cheat.  
但我们不仅复制“健康”以使其对其他人可见，我们还复制它以便服务器对其拥有权限并且客户端无法作弊。

### Blueprint[​](https://cedric-neukirchen.net/docs/multiplayer-compendium/common-classes/pawn#blueprint "Direct link to Blueprint") 蓝图​

Despite the standard overridable functions, the Pawn also has two which let you react to it being un-/possessed by a Player- or AIController.  
尽管有标准的可重写函数，Pawn 也有两个可以让您对它被玩家或 AIController 取消/占有做出反应。