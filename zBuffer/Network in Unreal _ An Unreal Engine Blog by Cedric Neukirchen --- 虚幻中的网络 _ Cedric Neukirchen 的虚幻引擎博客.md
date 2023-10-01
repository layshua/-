# 虚幻中的网络
虚幻引擎使用标准的**客户端 - 服务器 （Client-Server）架构**。这意味着服务器是**权威（Authoritative）** 的，所有数据必须首先从客户端发送到服务器。之后，服务器验证数据并根据您的代码做出反应。

## A Small Example[​](#a-small-example "Direct link to A Small Example") 一个小例子​

当您作为客户端在多人游戏中移动角色时，您不会自己移动角色，而是告诉服务器您想要移动它。然后，服务器会为其他人（包括您）更新角色的变换。

> [!info] 
>
> 此外，为了防止本地客户端有 “滞后” 的感觉，程序员通常还让本地客户端直接控制他们的角色——尽管当客户端开始作弊时，服务器仍然可能覆盖角色的位置！这意味着客户端（几乎）永远不会直接与其他客户端“交谈”。

## Another Example[​](#another-example "Direct link to Another Example") 另一个例子​

When sending a chat message to another client you are sending it to the server first, which then passes it to the client you wanted to reach. This could also be a team, guild, group, etc.  
当向另一个客户端（个人、公会、队伍等）发送聊天消息时，您首先将其发送到服务器，然后服务器将其传递给您想要联系的客户端。

Important 重要的

**Never** trust the client! Trusting the client here means you don't test the client's actions before executing them.  

> [!danger] 
> 永远不要相信客户端！信任客户端意味着您在执行客户端的操作之前不会测试它们。
这会允许他们作弊！
一个简单的例子是发射武器：确保在服务器上测试客户端是否拥有所需数量的弹药，之后再允许射击而不是直接处理射击！

# gameplay架构与网络
## GameMode
> [!NOTE]
> 在 4.14 中，AGameMode 类分为 AGameModeBase 和 AGameMode。 GameModeBase 的功能较少，因为某些游戏可能不需要旧 AGameMode 类的完整功能列表。
> 

AGameMode 类用于定义游戏规则。这包括要生成的其他游戏框架类，例如 APawn、APlayerController、APlayerState 等。

**它仅在服务器上可用。客户端没有 AGameMode 类的实例，并且在尝试检索它时只会得到 nullptr。**

##  示例和用法​

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

###  蓝图示例​

#### 函数

GameMode蓝图的 Override 函数部分：
![[cbde69313852572dab83dc34073ca75f_MD5.png]]

您可以实现这些函数的逻辑，以适应您的游戏的特定规则。 这包括更改 GameMode 生成 DefaultPawn 的方式或您想要如何决定游戏是否已准备好开始。

一个例子可能是检查所有玩家是否已加入服务器并准备好：
![[Pasted image 20231001162035.png]]

但也有一些事件可以用来对整个比赛中发生的某些事情做出反应。
我经常使用的一个很好的例子是事件 `OnPostLogin`。每次新玩家加入游戏时都会调用此方法。稍后您将了解有关连接过程的更多信息。
![[Pasted image 20231001160607.png|300]]
![[Pasted image 20231001162106.png]]
该事件会向您传递一个有效的 PlayerController 引用，该 Controller 由连接玩家的 UConnection 拥有（稍后也会详细介绍）。

这可以用于与该玩家进行交互，例如，为他们生成一个新的 Pawn，或者只是将其 PlayerController 保存在数组中以供以后使用。


正如已经提到的，您可以使用 GameMode 来管理游戏的一般比赛流程。为此，您可以找到一些功能，其中一些功能是可覆盖的，例如`Ready To Start Match`。

这些函数和事件可用于控制当前的 `MatchState`。当“`Ready To Start Match`”函数返回 **TRUE** 时，它们中的大多数将被自动调用，但您也可以手动使用它们。
![[Pasted image 20231001162147.png]]
>“`New State`”是一个简单的“FName”类型。您现在可能会问，“为什么这不在 AGameState 类中处理？”嗯，确实如此。这些 GameMode 函数与 GameState 协同工作。
**这只是为了给您一个点来管理任何客户端都无法访问的 `MatchState`，因为 GameMode 只存在于服务器上！**

#### 变量​

这是已经继承的变量的列表。其中一些可以通过 GameMode 蓝图的 ClassDefaults 进行设置：

![[80d227b1ee73d70c373d2144046809bd_MD5.png]]

![[ac948e6651eb07cead5c485776186c8c_MD5.png]]

其中大多数命名都很直白，例如`Default Player Name`，它使您能够为每个连接的玩家提供一个可以通过 `APlayerState` 类访问的默认玩家名称。
还有 `bDelayedStart`，这将使游戏无法开始，即使 `Ready To Start Match` 的默认实现满足所有其他条件。 

One of the more important Variables is the so-called **'Options String'**. These are options, separated by a **'?'**, which you can pass via the **'OpenLevel'** function or when you call **'ServerTravel'** as a Console Command.  
更重要的变量之一是 `Options String`。这些是选项，用“?”分隔，您可以通过`OpenLevel`函数或当您将`ServerTravel`作为控制台命令调用时传递这些选项。 

您可以使用 `Parse Option` 来提取传递的选项，例如“`MaxNumPlayers`”：  
![[Pasted image 20231001163900.png]]
<iframe height="10000em" width="10000em" src="https://blueprintue.com/render/16_cndpr/" scrolling="no" allowfullscreen></iframe>
### UE++ Examples[​]( #ue -examples "Direct link to UE++ Examples") UE++ 示例​

All of the Blueprint stuff can also be done in C++. Without writing up the same information again I will provide some code examples on how to recreate the previous Blueprint examples.  
所有蓝图内容也可以用 C++ 完成。在不再次编写相同信息的情况下，我将提供一些有关如何重新创建之前的蓝图​​示例的代码示例。

Since **'ReadyToStartMatch'** is a **'BlueprintNativeEvent'**, the actual C++ Implementation of the function is called **'ReadyToStartMatch_Implementation'**. This is the one we want to override:  
由于“ReadyToStartMatch”是“BlueprintNativeEvent”，因此该函数的实际 C++ 实现称为“ReadyToStartMatch_Implementation”。这是我们想要覆盖的：

Header file of our AGameMode child class inside of the class declaration  
类声明中 AGameMode 子类的头文件

```
// Maximum Number of Players needed/allowed during this Matchint32 MaxNumPlayers;// Override Implementation of ReadyToStartMatchvirtual bool ReadyToStartMatch_Implementation() override;
```

CPP file of our GameMode child class  
GameMode 子类的 CPP 文件

```
bool ATestGameMode::ReadyToStartMatch_Implementation(){    Super::ReadyToStartMatch();    return MaxNumPlayers == NumPlayers;}
```

The **'OnPostLogin'** function is virtual and simply called **'PostLogin'** in C++.  
“OnPostLogin”函数是虚拟函数，在 C++ 中简称为“PostLogin”。

Let's override it too: 让我们也覆盖它：

Header file of our AGameMode child class inside of the class declaration  
类声明中 AGameMode 子类的头文件

```
// List of PlayerControllersUPROPERTY()TArray<APlayerController*> PlayerControllerList;// Overriding the PostLogin functionvirtual void PostLogin(APlayerController* NewPlayer) override;
```

CPP file of our GameMode child class  
GameMode 子类的 CPP 文件

```
void ATestGameMode::PostLogin(APlayerController* NewPlayer){    Super::PostLogin(NewPlayer);    PlayerControllerList.Add(NewPlayer);}
```