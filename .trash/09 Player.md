# 8 Player
你们对力量一无所知

## 引言

回顾上文，我们谈完了 World 和 Level 级别的逻辑操纵控制，如同分离组合的 AController 一样，UE 在 World 的层次上也采用了一个分离的 AGameMode 来抽离了游戏关卡逻辑，从而支持了逻辑的组合。本篇我们继续上升一个层次，考虑在 World 之上，游戏还需要哪些逻辑控制？  
暂时不考虑别的功能系统（如社交系统，统计等各种），单从游戏性来讨论，现在闭上眼睛，想象我们已经藉着 UE 的伟力搭建了好了一个个 LevelWorld，嗯，就像《西部世界》一样，场景已经搭建好了，世界规则故事也编写完善，现在需要干些什么？当然是开始派玩家进去玩啦！  
大家都是老玩家了，想想我们之前玩的游戏类型：

*   玩家数目是单人还是多人
*   网络环境是只本地还是联网
*   窗口显示模式是单屏还是分屏
*   输入模式是共用设备还是分开控制（比如各有手柄）
*   也许还有别的不同

假如你是个开发游戏引擎的，会怎么支持这些不同的模式？以笔者见识过的大部分游戏引擎，解决这个问题的思路就是不解决，要嘛是限制功能，要嘛就是美名其曰让开发者自己灵活控制。不过想了一下，这也不能怪他们，毕竟很少有引擎能像 UE 这样历史悠久同时又能得到足够多的游戏磨练，才会有功夫在 GamePlay 框架上雕琢。大部分引擎还是更关注于实现各种绚丽的功能，至于怎么在上面开展游戏逻辑，那就是开发者自己的事了。一个引擎的功能是否强大，是基础比拼指标；而 GamePlay 框架作为最高层直面用户的对接接口，是一个引擎的脸面。所以有兴趣游戏引擎研究的朋友们，区分一个引擎是否 “优秀”，第二个指标是看它是否设计了一个优雅的游戏逻辑编写框架，一般只有基础功能已经做得差不多了的引擎开发者才会有精力去开发 GamePlay 框架，因为游戏引擎不止渲染！  
言归正传，按照软件工程的理念，没有什么问题是不能通过加一个间接层解决的，不行就加两层！所以既然我们在处理玩家模式的问题，理所当然的是加个间接层，将玩家这个概念抽象出来。  
那么什么是玩家呢？狭义的讲，玩家就是真实的你，和你身旁的小伙伴。广义来说，按照图灵测试理论，如果你无法分辨另一方是 AI 还是人，那他其实就跟玩家毫无区别，所以并不妨碍我们将网络另一端的一条狗当作玩家。那么在游戏引擎看来，玩家就是输入的发起者。游戏说白了，也只是接受输入产生输出的一个程序。所以有多少输入，这些输入归多少组，就有多少个玩家。这里的输入不止包括本地键盘手柄等输入设备的按键，也包括网线里传过来的信号，是广义的该游戏能接受到的外界输入。注意输出并不是玩家的必要属性，一个玩家并不一定需要游戏的输出，想象你闭上眼睛玩马里奥或者有个网络连接不断发送来控制信号但是从来不接收反馈，虽然看起来意义不大，但也确实不能说这就不是游戏。  
在 UE 的眼里，玩家也是如此广义的一个概念。本地的玩家是玩家，网络联机时虽然看不见对方，但是对方的网络连接也可以看作是个玩家。当然的，本地玩家和网络玩家毕竟还是差别很大，所以 UE 里也对二者进行了区分，才好更好的管理和应用到不同场景中去，比如网络玩家就跟本地设备的输入没多大关系了嘛。

## UPlayer

让我们假装自己是 UE，开始编写 Player 类吧。为了利用上 UObject 的那些现有特性，所以肯定是得从 UObject 继承了。那能否是 AActor 呢？Actor 是必须在 World 中才能存在的，而 Player 却是比 World 更高一级的对象。玩游戏的过程中，LevelWorld 在不停的切换，但是玩家的模式却是脱离不变的。另外，Player 也不需要被摆放在 Level 中，也不需要各种 Component 组装，所以从 AActor 继承并不合适。那还是保持简单吧：  

![[dd8b6f82366c93fce61070c65ca5b9e7_MD5.png]]

## ULocalPlayer

然后是本地玩家，从 Player 中派生下来 LocalPlayer 类。对本地环境中，一个本地玩家关联着输入，也一般需要关联着输出（无输出的玩家毕竟还是非常少见）。玩家对象的上层就是引擎了，所以会在 GameInstance 里保存有 LocalPlayer 列表。  

![[762eea989265e1cd724ec1976965eeb2_MD5.png]]

```
ULocalPlayer* UGameInstance::CreateLocalPlayer(int32 ControllerId, FString& OutError, bool bSpawnActor)
{
	ULocalPlayer* NewPlayer = NULL;
	int32 InsertIndex = INDEX_NONE;
	const int32 MaxSplitscreenPlayers = (GetGameViewportClient() != NULL) ? GetGameViewportClient()->MaxSplitscreenPlayers : 1;
    //已略去错误验证代码，MaxSplitscreenPlayers默认为4
	NewPlayer = NewObject<ULocalPlayer>(GetEngine(), GetEngine()->LocalPlayerClass);
	InsertIndex = AddLocalPlayer(NewPlayer, ControllerId);
	if (bSpawnActor && InsertIndex != INDEX_NONE && GetWorld() != NULL)
	{
		if (GetWorld()->GetNetMode() != NM_Client)
		{
			// server; spawn a new PlayerController immediately
			if (!NewPlayer->SpawnPlayActor("", OutError, GetWorld()))
			{
				RemoveLocalPlayer(NewPlayer);
				NewPlayer = NULL;
			}
		}
		else
		{
			// client; ask the server to let the new player join
			NewPlayer->SendSplitJoin();
		}
	}
	return NewPlayer;
}
```

可以看到，如果是在 Server 模式，会直接创建出 ULocalPlayer，然后创建出相应的 PlayerController。而如果是 Client（比如 Play 的时候选择 NumberPlayer=2, 则有一个为 Client），则会先发送 JoinSplit 消息到服务器，在载入服务器上的 Map 之后，再为 LocalPlayer 创建出 PlayerController。  
而在每个 PlayerController 创建的过程中，在其内部会调用 InitPlayerState：

```
void AController::InitPlayerState()
{
	if ( GetNetMode() != NM_Client )
	{
		UWorld* const World = GetWorld();
		const AGameModeBase* GameMode = World ? World->GetAuthGameMode() : NULL;
		//已省略其他验证和无关部分
		if (GameMode != NULL)
		{
			FActorSpawnParameters SpawnInfo;
			SpawnInfo.Owner = this;
			SpawnInfo.Instigator = Instigator;
			SpawnInfo.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;
			SpawnInfo.ObjectFlags |= RF_Transient;	// We never want player states to save into a map
			PlayerState = World->SpawnActor<APlayerState>(GameMode->PlayerStateClass, SpawnInfo );
	
			// force a default player name if necessary
			if (PlayerState && PlayerState->PlayerName.IsEmpty())
			{
				// don't call SetPlayerName() as that will broadcast entry messages but the GameMode hasn't had a chance
				// to potentially apply a player/bot name yet
				PlayerState->PlayerName = GameMode->DefaultPlayerName.ToString();
			}
		}
	}
}
```

这样 LocalPlayer 最终就和 PlayerState 对应了起来。而网络联机时其他玩家的 PlayerState 是通过 Replicated 过来的。  
我们谈了那么久的玩家就是输入，体现在在每个 PlayerController 接受 Player 的时候：

```
void APlayerController::SetPlayer( UPlayer* InPlayer )
{
    //[...]
	// Set the viewport.
	Player = InPlayer;
	InPlayer->PlayerController = this;
	// initializations only for local players
	ULocalPlayer *LP = Cast<ULocalPlayer>(InPlayer);
	if (LP != NULL)
	{
		// Clients need this marked as local (server already knew at construction time)
		SetAsLocalPlayerController();
		LP->InitOnlineSession();
		InitInputSystem();
	}
	else
	{
		NetConnection = Cast<UNetConnection>(InPlayer);
		if (NetConnection)
		{
			NetConnection->OwningActor = this;
		}
	}
	UpdateStateInputComponents();
	// notify script that we've been assigned a valid player
	ReceivedPlayer();
}
```

可见，对于 ULocalPlayer，APlayerController 内部会开始 InitInputSystem()，接着会创建相应的 UPlayerInput，BuildInputStack 等初始化出和 Input 相关的组件对象。现在先明白到 LocalPlayer 才是 PlayerController 产生的源头，也因此才有了 Input 就够了，特定的 Input 事件流程分析在后续章节再细述。

**思考：为何不在 LocalPlayer 里编写逻辑？**  
作为游戏开发者，相信大家都有这么个体会，往往在游戏逻辑代码中总会有一个自己的 Player 类，里面放着这个玩家的相关数据和逻辑业务。可是在 UE 里为何就不见了这么个结构？也没见 UE 在文档里有描述推荐你怎么创建自己的 Player。  
这个可能有两个原因，一是 UE 从 FPS-Specify 游戏起家，不像现在的各种手游有非常重的玩家系统，在 UE 的眼中，Level 和 World 才是最应该关注的对象，因此 UE 的视角就在于怎么在 Level 中处理好 Player 的逻辑，而非在 World 之外的额外操作。二是因为在一个 World 中，上文提到其实已经有了 Pawn-PlayerController 和 PlayerState 的组合了，表示、逻辑和数据都齐备了，也就没必要再在 Level 掺和进 Player 什么事了。当然你也可以理解为 PlayerController 就是 Player 在 Level 中的话事人。  
凡事留一线，日后好相见。尽管如此，UE 还是给了我们自定义 ULocalPlayer 子类的机会：

```
//class UEngine：
/** The class to use for local players. */
UPROPERTY()
TSubclassOf<class ULocalPlayer>  LocalPlayerClass;

/** @todo document */
UPROPERTY(globalconfig, noclear, EditAnywhere, Category=DefaultClasses, meta=(MetaClass="LocalPlayer", DisplayName="Local Player Class"))
FStringClassReference LocalPlayerClassName;
```

你可以在配置中写上 LocalPlayer 的子类名称，让 UE 为你生成你的子类。然后再在里面写上一些特定玩家的数据和逻辑也未尝不可，不过这部分额外扩展的功能就得用 C++ 来实现了。

## UNetConnection

非常耐人寻味的是，在 UE 里，一个网络连接也是个 Player：  

![[ceb29d2b52829463d9d4b853d239f134_MD5.png]]

## 总结

本篇我们抽象出了 Player 的概念，并依据使用场景派生出了 LocalPlayer 和 NetConnection 这两个子类，从此 Player 就不再是一个虚无缥缈的概念，而是 UE 里的逻辑实体。UE 可以根据生成的 Player 对象的数量和类型的不同，在此上实现出不同的玩家控制模式，LocalPlayer 作为源头 Spawn 出 PlayerController 继而 PlayerState 就是实证之一。而在网络联机时，把一个网络连接看作是一个玩家这个概念，把在 World 之上的输入实体用 Player 统一了起来，从而可以实现出灵活的本地远程不同玩家模式策略。  
尽管如此，UPlayer 却像是深藏在 UE 里的幕后功臣，UE 也并不推荐直接在 Player 里编程，而是利用 Player 作为源头，来产生构建一系列相关的机制。但对于我们游戏开发者而言，知道并了解 UE 里的 Player 的概念，是把现实生活同游戏世界串联起来的很重要的纽带。我们在一个个 World 里向上仰望，还能清楚的看见一个个 LocalPlayer 或 NetConnection 仿佛在注视着这片大地，是他们为 World 注入了生机。  
已经到头了？并没有，我们继续向上逆风飞翔，终将得见游戏里的神：GameInstance。

上篇：[《InsideUE4》GamePlay 架构（七）GameMode 和 GameState](https://zhuanlan.zhihu.com/p/23707588)

下篇：[《InsideUE4》GamePlay 架构（九）GameInstance](https://zhuanlan.zhihu.com/p/24005952)

## 引用

_UE4.14_

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**