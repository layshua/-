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

## GameState（服务器+客户端）

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

The PlayerArray is not directly replicated, however, every PlayerState is replicated and they add themselves to the PlayerArray on construction. Additionally, they are collected by the GameState, just to ensure no race-condition causes problems.  
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

All of this happens on the server and the client instances of Player- and GameState!  
所有这一切都发生在服务器以及 Player 和 GameState 的客户端实例上！

#### Functions[​]( #functions "Direct link to Functions") 功能​

A small function example I could provide you with would be keeping track of the score of two teams 'A' and 'B'. Let's say we have a CustomEvent which is called when a team scores.  
我可以为您提供的一个小功能示例是跟踪 “A” 和“B”两支球队的得分。假设我们有一个 CustomEvent，当球队得分时会调用该事件。

It passes a boolean so we know which team scored. We could also pass in a PlayerState, Team, or whatever you utilize to identify who scored.  
它传递一个布尔值，这样我们就知道哪支球队得分了。我们还可以传递 PlayerState、Team 或任何您用来识别得分者的信息。

Later in the “Replication” chapter, you will read about the rule that only the server can (and should) replicate variables, so we make sure only he can call this event.  
稍后在 “复制” 章节中，您将了解只有服务器可以（并且应该）复制变量的规则，因此我们确保只有他可以调用此事件。

The event is called from another class (for example a weapon that killed someone) and this should happen on the Server (always!), so we don't need an RPC here.  
该事件是从另一个类调用的（例如杀死某人的武器），并且这应该发生在服务器上（总是！），因此我们在这里不需要 RPC。

Since these variables and the GameState are replicated you can use these two variables and get them in any other class you need them. For example, to display them in a scoreboard widget.  
由于这些变量和 GameState 是复制的，因此您可以使用这两个变量并将它们放入您需要的任何其他类中。例如，将它们显示在记分板小部件中。

#### UE++ Examples[​]( #ue -examples "Direct link to UE++ Examples") UE++ 示例​

To recreate this small example we need a bit more code, but despite the function itself the code needed to set up the replication is only needed once per class.  
为了重新创建这个小示例，我们需要更多的代码，但是尽管函数本身，每个类只需要一次设置复制所需的代码。

Header file of our AGameState class inside of the class declaration  
我们的 AGameState 类的头文件位于类声明中

```
// You need this included to get the replication working.#include “UnrealNetwork.h”// Replicated specifier used to mark this variable to replicateUPROPERTY(Replicated)int32 TeamAScore;UPROPERTY(Replicated)int32 TeamBScore;// Function to increase the score of a teamvoid AddScore(bool bTeamAScored);
```

You will read more about this function in the Replication part!  
您将在复制部分阅读有关此功能的更多信息！

CPP file of our AGameState child class  
AGameState 子类的 CPP 文件

```
// This function is required through the replicated specifier in the UPROPERTY macro and is declared by itvoid ATestGameState::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const{    Super::GetLifetimeReplicatedProps(OutLifetimeProps);    DOREPLIFETIME(ATestGameState, TeamAScore);    DOREPLIFETIME(ATestGameState, TeamBScore);}
```

CPP file of our AGameState child class  
AGameState 子类的 CPP 文件

```
void ATestGameState::AddScore(bool bTeamAScored){    if (bTeamAScored)    {        TeamAScore++;    }    else    {        TeamBScore++;    }}
```