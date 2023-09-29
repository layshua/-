我的世界，我做主

## 引言

上文我们说到在 Actor 层次，UE 用 Controller 来充当 APawn 的逻辑控制者，也有了可以接受玩家输入的 PlayerController，和能自行行动的 AIController。Actor 的逻辑编写介绍完了，那么本篇，我们继续爬升，对于由 Actors 组成的 Level 这一层次，UE 又是怎么控制的呢？  
对 Level 记不太清楚的朋友，可以翻回去查看 “GamePlay 架构（二）Level 和 World” 的讲述，简单概括就是 World 是由一个 PersistentLevel 和一些 subLevels 组成的，PersistentLevel 切换了，相应的 World 也会切换。所以本文的关注点是在这么一个对象层次结构下，UE 是怎么设计的，我们又能做些什么。

## GameMode

Level，在游戏里的概念里，就是关卡的意思。同时作为游戏的玩家和开发者，我们总是会非常经常的提起关卡，但是关卡具体又是个什么定义呢？游戏里的哪些部分可以算是一个关卡？简单的我们都知道有《愤怒的小鸟》或《植物大战僵尸》的关卡，复杂的有大型 FPS 游戏里的关卡，而对于更大型的《暗黑 3》或者大型无缝地图 RPG 游戏《巫师 3》，甚至是号称超级广阔宇宙《无人深空》，我们能直接了当的说出哪部分是关卡吗？游戏行业发展如今，为了更好的组织游戏逻辑和内容资源，也发展出了一些概念来更好的理解和阐述，虽然叫法不同，不过含义理念都是相通的。比如，Cocos2dx 会认为游戏就是由不同的 Scene 切换组成的，每个 Scene 又由 Layer 组成；Unity 也认为游戏就是一个个 Scene；而 UE 的视角的是，游戏是由一个个 World 组成的，World 又是由 Level 组成的。这些概念有什么不同？  
让我们从游戏本身的机制上分析：

*   游戏或玩家的节奏，游戏可以分成一个个阶段，马里奥里的关卡就是一个阶段，而 RPG 游戏的一个大地图也是一个阶段。一个游戏也可能只有一个阶段，比如一直在宇宙里漫游的游戏。通常一个阶段结束后，会有一个结算，阶段之间，玩家也能明显感觉到切换感。
*   游戏的机制，有时候即使是同样的场景，玩家却也能感觉就像在玩两个不同的游戏，比如 MOBA 里的同一张地图上的各种不同挑战模式。
*   游戏的资源划分，有时候也能遇见同一个玩法应用在不同的场景上，比如赛车游戏的不同跑道。有时候也会在游戏的大地图里从酷热的沙漠到寒冷的极地。游戏开发中也总是倾向于给游戏用到的资源划分成组的进行载入和释放。

通过以上的分析，也和以前的一贯思路一样，我们发现在思考 “关卡” 这件事情上，也是要保持头脑清晰的分清 “表示” 和“逻辑”。玩法就是“逻辑”，场景就是“表示”。所以我们如果以逻辑来划分游戏，得到的就是一个个 World 的概念；如果以表示来划分，得到就是一个个 Level。一场游戏中，玩法再复杂但也只有一个，场景却可以无限大，所以可以有很多个表示拼接组装，因此是 World 包含 Level，而不是反过来。现在回过头来回想一下 Cocos2dx 和 Unity 的世界观，它们的概念还只是在表示层，在游戏实例和关卡之间少了一个更高级的逻辑概念。  
因此 UE 的世界观是，World 更多是逻辑的概念，而 Level 是资源场景表示。以《巫师 3》为例，有好几个国家之间通过传送切换，国家内大地图无缝漫游，显然我们知道不可能把一个国家的所有资源都加载进内存，因此在 UE 里，一个国家就是许多个 Level 拼接的，而一个国家就是一个 World，它们可以有不同的模式玩法。但毕竟 AAA 游戏很少，通常的，我们的游戏比较简单的用一个 Level 就够了，否则这个场景表示的概念就应该叫 Area 更合适了，也因此通常的这里的 Level 也常常对应游戏里玩家面对的 "关卡"，也因此 UE 里 Level 的 Settings 叫做 WorldSettings 了。  
厘清了这些概念了之后，我们就知道，当我们在谈 Level 的业务逻辑控制的时候，我们实际上谈的是 World 的业务逻辑。按照 UE 的设计理念和经过 Controller 的经历，我想我也不用多解释了从 Actor 再派生出一个 WorldController 的方式了，可以直接的享受 Actor 已经提供的一切福利。一个 World 的 Controller 想不出有什么需要展示渲染的，因此可以直接从 AInfo 派生吧。哦，WorldController 是我瞎编的，在 UE3 里它叫做 GameInfo，到了 UE4 它改名为了 GameMode。笼统的讲，一个 World 就是一个 Game，把玩法叫做 Mode，我们应该也能接受吧。那我们来看看它：  

![[ff5cbe14bd917023fc60dd39a31662a2_MD5.png]]

既然勇敢的承担了游戏逻辑的职责，说他是 AInfo 家族里的扛把子也不为过，因此 GameMode 身为一场游戏的唯一逻辑操纵者身兼重任，在功能实现上有许多的接口，但主要可以分为以下几大块：

1.  **Class 登记**，GameMode 里登记了游戏里基本需要的类型信息，在需要的时候通过 UClass 的反射可以自动 Spawn 出相应的对象来添加进关卡中。前文说过的 Controller 的类型登记也是在此，GameMode 就是比 Controller 更高一级的领导。

![[08a9083228c88c88ed0e7c9869db8e03_MD5.png]]

1.  **游戏内实体的 Spawn**，不光登记，GameMode 既然作为一场游戏的主要负责人，那么游戏的加载释放过程中涉及到的实体的产生，包括玩家 Pawn 和 PlayerController，AIController 也都是由 GameMode 负责。最主要的 SpawnDefaultPawnFor、SpawnPlayerController、ShouldSpawnAtStartSpot 这一系列函数都是在接管玩家实体的生成和释放，玩家进入该游戏的过程叫做 Login（和服务器统一），也控制进来后在什么位置，等等这些实体管理的工作。GameMode 也控制着本场游戏支持的玩家、旁观者和 AI 实体的数目。
2.  **游戏的进度**，一个游戏支不支持暂停，怎么重启等这些涉及到游戏内状态的操作也都是 GameMode 的工作之一，SetPause、ResartPlayer 等函数可以控制相应逻辑。
3.  **Level 的切换**，或者说 World 的切换更加合适，GameMode 也决定了刚进入一场游戏的时候是否应该开始播放开场动画（cinematic），也决定了当要切换到下一个关卡时是否要 bUseSeamlessTravel，一旦开启后，你可以重载 GameMode 和 PlayerController 的 GetSeamlessTravelActorList 方法和 GetSeamlessTravelActorList 来指定哪些 Actors 不被释放而进入下一个 World 的 Level。
4.  **多人游戏的步调同步**，在多人游戏的时候，我们常常需要等所有加入的玩家连上之后，载入地图完毕后才能一起开始逻辑。因此 UE 提供了一个 MatchState 来指定一场游戏运行的状态，意义看名称也是不言自明的，就是用了一个状态机来标记开始和结束的状态，并触发各种回调。  
    /** Possible state of the current match, where a match is all the gameplay that happens on a single map */  
    namespace MatchState  
    {  
    extern ENGINE_API const FName EnteringMap; // We are entering this map, actors are not yet ticking  
    extern ENGINE_API const FName WaitingToStart; // Actors are ticking, but the match has not yet started  
    extern ENGINE_API const FName InProgress; // Normal gameplay is occurring. Specific games will have their own state machine inside this state  
    extern ENGINE_API const FName WaitingPostMatch; // Match has ended so we aren't accepting new players, but actors are still ticking  
    extern ENGINE_API const FName LeavingMap; // We are transitioning out of the map to another location  
    extern ENGINE_API const FName Aborted; // Match has failed due to network issues or other problems, cannot continue  
    }

**思考：多个 Level 配置不同的 GameMode 时采用的是哪一个 GameMode？**  
我们知道除了配置全局的 GameModeClass 之外，我们还能为每个 Level 单独的配置不同的 GameModeClass。但是当一个 World 由多个 Level 组成的时候，这样就相当于配置了多个 GameModeClass，那么应用的是哪一个？首先第一个原则需要记住的就是，一个 World 里只会有一个 GameMode 实例，否则肯定乱套了。因此当有多个 Level 的时候，一定是 PersistentLevel 和多个 StreamingLevel，这时就算它们配置了不同的 GameModeClass，UE 也只会为第一次创建 World 时加载 PersistentLevel 的时候创建 GameMode，在后续的 LoadStreamingLevel 时候，并不会再动态创建出别的 GameMode，所以 GameMode 从始至终只有一个，PersistentLevel 的那个。

**思考：Level 迁移时 GameMode 是否保持一致？**  
在在 travelling 的时候，如果下一个 Level 的配置的 GameModeClass 和当前的不同，那么迁移后是哪个 GameMode？  
无论 travelling 采用哪种方式，当前的 World 都会被释放掉，然后加载创建新的 World。但这个过程中，有点区别的是根据 bUseSeamlessTravel 的不同，UE 可以选择哪些 Actor 迁移到下一个 World 中去（实现方式是先创建个中间过渡 World 进行二段迁移（为了避免同时加载进两个大地图撑爆内存），具体见引用 3）。分两种情况：  
不开启 bUseSeamlessTravel，那么在 travelling 的时候（ServerTravel 或 ClientTravel），当前的 World 会被释放，所以当前的 GameMode 就被释放掉。新的 World 加载，就会根据新的 GameModeClass 创建新的 GameMode。所以这时是不同的。  
开启 bUseSeamlessTravel，travelling 时，当前 World 的 GameMode 会调用 GetSeamlessTravelActorList：

```
void AGameMode::GetSeamlessTravelActorList(bool bToTransition, TArray<AActor*>& ActorList)
{
	UWorld* World = GetWorld();

	// Get allocations for the elements we're going to add handled in one go
	const int32 ActorsToAddCount = World->GameState->PlayerArray.Num() + (bToTransition ?  3 : 0);
	ActorList.Reserve(ActorsToAddCount);

	// always keep PlayerStates, so that after we restart we can keep players on the same team, etc
	ActorList.Append(World->GameState->PlayerArray);

	if (bToTransition)
	{
		// keep ourselves until we transition to the final destination
		ActorList.Add(this);
		// keep general game state until we transition to the final destination
		ActorList.Add(World->GameState);
		// keep the game session state until we transition to the final destination
		ActorList.Add(GameSession);

		// If adding in this section best to increase the literal above for the ActorsToAddCount
	}
}
```

在第一步从 CurrentWorld 到 TransitionWorld 的迁移时候，bToTransition==true，这个时候 GameMode 也会迁移进 TransitionWorld（TransitionMap 可以在 ProjectSettings 里配置），也包括 GameState 和 GameSession，然后 CurrentWorld 释放掉。第二步从 TransitionWorld 到 NewWorld 的迁移，GameMode（已经在 TransitionWorld 中了）会再次调用 GetSeamlessTravelActorList，这个时候 bToTransition==false，所以第二次的时候如代码所见当前的 GameMode、GameState 和 GameSession 就被排除在外了。这样 NewWorld 再继续 InitWorld 的时候，一发现当前没有 GameMode，就会根据配置的 GameModeClass 重新生成一个出来。所以这个时候 GameMode 也是不同的。  
结论是，UE 的流程 travelling，GameMode 在新的 World 里是会新生成一个的，即使 Class 类型一致，即使 bUseSeamlessTravel，因此在 travelling 的时候要小心 GameMode 里保存的状态丢失。不过 Pawn 和 Controller 默认是一致的。

**思考：哪些逻辑应该写在 GameMode 里？哪些应该写在 Level Blueprint 里？**  
我们依旧要问这个老土的问题。根据我们前面的知识，我们知道每个 Level 其实也是有自己的 LevelScriptActor 的，那么这两个有什么区别？可以从这几个方面来回答：

*   概念上，Level 是表示，World 是逻辑，一个 World 如果有很多个 Level 拼在一起，那么也就是有了很多个 LevelScriptActor，无法想象在那么多个地方写一个完整的游戏逻辑。所以 GameMode 应该专注于逻辑的实现，而 LevelScriptActor 应该专注于本 Level 的表示逻辑，比如改变 Level 内某些 Actor 的运动轨迹，或者某一个区域的重力，或者触发一段特效或动画。而 GameMode 应该专注于玩法，比如胜利条件，怪物刷新等。
*   组合上，同 Controller 应用到 Pawn 一样道理，因为 GameMode 是可以应用在不同的 Level 的，所以通用的玩法应该放在 GameMode 里。
*   GameMode 只在 Server 存在（单机游戏也是 Server），对于已经连接上 Server 的 Client 来说，因为游戏的状态都是由 Sever 决定的，Client 只是负责展示，所以 Client 上是没有 GameMode 的，但是有 LevelScriptActor，所以 GameMode 里不要写 Client 特定相关的逻辑，比如操作 UI 等。但是 LevelScriptActor 还是有的，而且支持 RPC，即使如此，LevelScriptActor 还是应该只专注于表现，比如网络中触发一个特效火焰。至于 UI，可以通过 PlayerController 的 RPC，然后转发到 GameInstance 来操作。
*   跟下层的 PlayerController 比较，GameMode 关心的是构建一个游戏本身的玩法，PlayerController 关心的玩家的行为。这两个行为是独立正交可以自由组合的。所以想想哪些逻辑属于游戏，哪些属于玩家，就应该清楚写在哪里了。
*   跟上层的 GameInstance 比较，GameInstance 关注的是更高层的不同 World 之间的逻辑，虽然有时候他也把手伸下来做些 UI 的管理工作，不过严谨来说，在 UE 里 UI 是独立于 World 的一个结构，所以也还算能理解。因此可以把不同 GameMode 之间协调的工作交给 GameInstance，而 GameMode 只专注自己的玩法世界。

## GameState

上回说到了 APlayerState 用来保存玩家的游戏数据，那么同样的，对于一场游戏，也需要一个 State 来保存当前游戏的状态数据，比如任务数据等。跟 APlayerState 一样，GameState 也选择从 AInfo 里继承，这样在网络环境里也可以 Replicated 到多个 Client 上面去。  

![[568304d68e5bb5a41c29dbd4931ae780_MD5.png]]

比较简单，第一个 MatchState 和相关的回调就是为了在网络中传播同步游戏的状态使用的（记得 GameMode 在 Client 并不存在，但是 GameState 是存在的，所以可以通过它来复制），第二部分是玩家状态列表，同样的如果在 Client1 想看到 Client2 的游戏状态数据，则 Client2 的 PlayerState 就必须广播过来，因此 GameState 把当前 Server 的 PlayerState 都收集了过来，方便访问使用。  
关于使用，开发者可以自定义 GameState 子类来存储本 GameMode 的运行过程中产生的数据（那些想要 replicated 的!），如果是 GameMode 游戏运行的一些数据，又不想要所有的客户端都可以看到，则也可以写在 GameMode 的成员变量中。重复遍，PlayerState 是玩家自己的游戏数据，GameInstance 里是程序运行的全局数据。

### GameSession

是在网络联机游戏中针对 Session 使用的一个方便的管理类，并不存储数据，本文重点也不在网络，故不做过多解释，可暂时忽略，留待网络章节再讨论。在单机游戏中，也存在该类对象用来 LoginPlayer，不过因为只是作为辅助类，那也可看作 GameMode 本身的功能，所以不做过多讨论。

## 总结

现在，我们也算讨论完了 Level（World）层次的控制，对于一场游戏而言，我们最关心的是怎么协调好整个场景的表现（LevelBlueprint）和游戏玩法的编写（GameMode）。UE 再次用 Actor 分化派生的思想，用同样套路的 AGameMode 和 AGameState 支持了玩法和表现的解耦分离和自由组合，并很好的支持了网络间状态的同步。同时也提供了一个逻辑的实体来负责创建关系内那些关键的 Pawn 和 Controller 们，在关卡切换（World）的时候，也有了一个负责对象来处理一些本游戏的特定情况处理。  

![[6ae32c6c874f92f335bb4564cf5b58c0_MD5.png]]

我们的逻辑之旅还没到终点，让我们继续爬升，下篇将介绍 Player。

上篇：[《InsideUE4》GamePlay 架构（六）PlayerController 和 AIController](https://zhuanlan.zhihu.com/p/23649987)

下篇：[《InsideUE4》GamePlay 架构（八）Player](https://zhuanlan.zhihu.com/p/23826859)

## 修订

在笔者书写本篇的同时（UE4.13.2），UE 同时也完成了 4.14 的 preview3 的工作，roadmap 里 “GameMode Cleanup” 的工作也已经完成了，第二天发现 4.14 正式发布了。因此为了紧跟 UE 最新潮流时尚，以后要是文章内容所涉及内容被 UE 修改完善优化的，也会采用修订的方式进行补充说明，之后不再特意作此声明。

### 4.14 GameMode，GameState 的清理

根据搜索到的最早记录 "[[Request/Improvment] GameMode cleanup.](https://forums.unrealengine.com/showthread.php?39840-Request-Improvment-GameMode-cleanup)"(09-14-2014)，是有人抱怨当前的 GameMode 实现了太多的默认逻辑（例如多人的 Match），虽然方便了一些人使用，但是也确实加大了理解的难度，并且有时候还得去屏蔽删除一些默认逻辑。然后顺便吐槽了一番 AActor 里的 Damage，笔者也表示这确实不是 AActor 应该管的事情。  
言归正传，UE 在 2016-08-24 的时候开始加进 roadmap，并终于在 4.14 里实现完成了。如前所述，就是把 GameMode 和 GameState 的一些共同最基础部分抽到基类 AGameModeBase 和 AGameStateBase 里，并把现在的 GameMode 和 GameState 依然当作多人联机的默认实现。所以以后大家如果想实现一个比较简单的单机 GameMode 就可以直接从 AGameModeBase 里继承了。

![[2eeb7821615ea2f3fc91b706e01956c1_MD5.png]]

可以看到，其实就是把 MatchState 给往下拉了一层，并把一些多玩家控制的逻辑，合起来就是网络联机游戏的默认逻辑给抽离开了。同样的对于 GameState 也做了处理：  

![[e31ff406ddb69178ad7119d6d819cfed_MD5.png]]

把 MatchState 也抽离到了下层，并增加了几个方便的字段引用（如 AuthorityGameMode）。总体功能职责架构上还是没有什么大变化的，吓死我了。

## 引用

1.  [GameMode](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/GameMode/index.html)
2.  [GameState](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/GameState/index.html)
3.  [Travelling in Multiplayer](https://docs.unrealengine.com/latest/INT/Gameplay/Networking/Travelling/index.html)

_UE4.13.2_

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**