一人之下，万人之上

## 引言

上篇我们讲到了 UE 在 World 之上，继续抽象出了 Player 的概念，包含了本地的 ULocalPlayer 和网络的 UNetConnection，并以此创建出了 World 中的 PlayerController，从而实现了不同的玩家模式策略。一路向上，依照设计里一个最朴素的原理：自己是无法创建管理自身的，所以 Player 也需要一个创建管理和存储的地方。另一方面，上文提到 Player 固然可以负责一些跟玩家相关的业务逻辑，但是对于 World 之上协调管理的逻辑却也仍然无处安放。  
如果是有一定的游戏开发实战经验的朋友也一定能体会到，在自己开发的游戏中，往往除了我们上文提到的 Player 类，常常会创建一个 Game 类，比如 BattleGame、WarGame 或 HappyGame 等等。Game 之前的名词往往都是游戏的开发代号。这倒不是因为我们如此热衷创建各种 Manager 类，而是确实需要一个大管家来干一些协调的活。一般的游戏引擎都只会暴露给你它自己引擎的管理类，如 Director，Engine 或 Application 之类的，但是却不会主动在 Game 类的创建管理上为你提供方便。游戏引擎的出现，最开始其实只是因为一些人发现游戏做着做着，有一大部分功能是可以复用的，于是就把它抽离了出来方便做下一款游戏。在那个时候，人们对游戏还是处于开荒探索的阶段，游戏引擎只是一大堆功能的复合体，就像叮当猫的口袋一样，互相比谁掏出的工具最强大。然而即使到了现代，绝大部分的引擎的思想却还停留在上个世纪，仍然执着于罗列 Feature 列表，却忘了真正的游戏开发人员天天面对的游戏业务逻辑编写，没有思考在那方面如何也下一番功夫去帮助开发者。人们对比 UE 和其他游戏引擎时，也会常常说出的一句话是：“别忘了 Epic 自己也是做游戏的”（虚幻竞技场，战争机器，无尽之剑……）。从这一点也可以看出，UE 很大的得益于 Epic 实战游戏开发的反哺，这一方面 Unity 就有点吃亏了，没有自己亲自下手干脏活累活，就不懂得急人民群众之所急。所以如果一个游戏引擎能把 GamePlay 也做好了，那就不止是口袋了，而是知你懂你的叮当猫本身。

## GameInstance

简单的事情就不用多讲了，UE 提供的方案是一以贯之的，为我们提供了一个 GameInstance 类。为了受益于 UObject 的反射创建能力，直接继承于 UObject，这样就可以依据一个 Class 直接动态创建出来具体的 GameInstance 子类。  

![[a38d1f5a8c38225e0f83b6e2a0a4d5fc_MD5.png]]

我并不想罗列所有的接口，UGameInstance 里的接口大概有 4 类：

1. 引擎的初始化加载，Init 和 ShutDown 等（在引擎流程章节会详细叙述）

2. Player 的创建，如 CreateLocalPlayer，GetLocalPlayers 之类的。

3. GameMode 的重载修改，这是从 4.14 新增加进来改进，本来你只能为特定的某个 Map 配置好 GameModeClass，但是现在 GameInstance 允许你重载它的 PreloadContentForURL、CreateGameModeForURL 和 OverrideGameModeClass 方法来 hook 改变这一流程。

4. OnlineSession 的管理，这部分逻辑跟网络的机制有关（到时候再详细介绍），目前可以简单理解为有一个网络会话的管理辅助控制类。

而 GameInstance 是在 GameEngine 里创建的（先不谈 UEditorEngine）：

```
void UGameEngine::Init(IEngineLoop* InEngineLoop)
{
    //[...]
	// Create game instance.  For GameEngine, this should be the only GameInstance that ever gets created.
	{
		FStringClassReference GameInstanceClassName = GetDefault<UGameMapsSettings>()->GameInstanceClass;
		UClass* GameInstanceClass = (GameInstanceClassName.IsValid() ? LoadObject<UClass>(NULL, *GameInstanceClassName.ToString()) : UGameInstance::StaticClass());
		if (GameInstanceClass == nullptr)
		{
			UE_LOG(LogEngine, Error, TEXT("Unable to load GameInstance Class '%s'. Falling back to generic UGameInstance."), *GameInstanceClassName.ToString());
			GameInstanceClass = UGameInstance::StaticClass();
		}
		GameInstance = NewObject<UGameInstance>(this, GameInstanceClass);
		GameInstance->InitializeStandalone();
	}
	//[...]
 }
//在BaseEngine.ini或DefaultEngine.init里你可以配置GameInstanceClass
[/Script/EngineSettings.GameMapsSettings]
GameInstanceClass=/Script/Engine.GameInstance
```

先从配置中取出 GameInstanceClass，然后动态创建，一目了然。

**思考：GameInstance 只有一个吗？**  
一般而言，是的。对于我们自己开发的游戏而言，我们始终只需要关注自己的一亩三分地，那么你可以认为你子类化的那个 GameInstance 就像个单件一样，全局唯一只有一个，从游戏的开始到结束。但既然是本系列文章的读者，自然也是不甘于只了解这么多的。  
正如把网络连接也当作 Player 这个概念一样，我们此时也需要重新审视一下 Game 这个概念。什么是一个 Game? 对于玩家而言，Game 就是从打开到关闭的这整个过程说展现的内容。但是对于开发者来说，这个概念就需要扩充一下了。假设有个引擎支持双击图标一下子开出 4 个窗口来让 4 个玩家独立运行，你能说得清这是一个 Game 还是 4 个 Game 在运行吗？哪一种说法都能自圆其说，但关键是哪一种概念划分能更好的让我们管理组织结构。因此针对这种情况，如果是这 4 个窗口一点都不互相关联，或者只是单独的共用地图资源，那么用 4 个 Game 的概念来管理就更为合适。如果这 4 个窗口里运行的内容，实际上只是在同一个关卡里本地对战，内存里互相直接通信，那用一个 Game 加上 4 个 Player 的概念就会变得更合适。所以针对这点，你可以把 Game 理解为就像进程一样，进程可以在同一个 exe 上多开，Game 也可以在同一份游戏资源上开出多个运行实例；进程之间可以互相通信协作，Game 的不同实例也可以互相沟通，不管是内存中直接在 Engine 的协调下完成，还是通过 Socket 通信。  
另一方面，一般游戏引擎都只是服务于游戏本身，而对于其配套的各种编辑器就像是对待外来的打工者一样，编辑器往往只负责最终输出游戏资源。由于应用场景的不同，编辑器的架构也常常根据相应平台而定，五花八门，有用 Qt，MFC，WPF 等各种平台 UI 框架。而对于另一些有大志向的引擎，比如 Unity 和 UE，其编辑器就是采用引擎自绘的方案（其优劣暂不分析，以后聊到 UI 框架再细说）。所以游戏引擎这个时候，就更加的拔高了一个层次，就不再只是个 “游戏” 引擎了，而是个 “程序” 引擎了。因此 UE 本身的这套框架不光要服务游戏，还要服务编辑器，甚至是另外一些辅助程序。所以，Game 的概念也就扩充到了更上层的“程序”，变得更广义了。  
言归正传，因为 UE 的这套 Editor 自绘机制，还有 PIE（PlayInEditor），进程里其实是可以同时有多个 GameInstance 的，如正在编辑的 EditorWorld 所属于的，和 Play 之后的 World 属于的。我想，这也就是为何 UE 把它叫做 GameInstance 而不是简单的 Game 的含义，其名字中就隐含了多个 Instance 的深意。我们现在再次回顾一下 ([GamePlay 架构（三）WorldContext，GameInstance，Engine](https://zhuanlan.zhihu.com/p/23167068)) 最后的结构图，了解一下 GameInstance 又是被谁管理的：  

![[fe68f967adb22cfeea680d61a78da666_MD5.png]]

当初我们是以数据的视角，在考察 WorldContext 的从属的时候讨论过这个结构。现在以逻辑的角度，明白了 GameInstance 也会被上层的 Engine 实例出来多个，就会有更深的理解了。

再扩充一下，在 Engine 之下允许同时运行多个 GameInstance，还会有许多其他好处，就像操作系统允许一份资源运行多个进程实例一样，Engine 就可以站在更高的层次上管理协调多个 Game，同时也能更加的深入到 Game 内部去得到更多的优化。比如未来要实现游戏本地的 host 多开并管理，或者在 Server 同时 Host 一个 Map 的多个实例 (现在只能一个…… 还是有很多工作要做啊)，这对于开发 MMO 网游是非常需要的功能，虽然目前 UE 在这一块的具体工作还有些薄弱，但至少可扩展的可能性是已经保证了的（动手能力强的高手可以在此基础上定制）。一般而言，间接多一层，就多了一层的灵活性，所以很多引擎其实就是把 Game 和 Engine 揉在了一块没有为了 GamePlay 框架而分开。

**思考：哪些逻辑应该放在 GameInstance？**  
第二个惯例的问题是，这一层应该写些什么逻辑。顾名思义，既然是作为游戏中全局唯一的长者，我们就应该给他全局的控制权。在逻辑层面，GameInstance 往下看是：  
1. Worlds，Level 的切换实际发生地是 Engine，而 GameInstance 可以说是 UE 之神其下的唯一代言人，所以 GameInstance 也可以代之管理 World 的切换等。我们可以在 GameInstance 里实现各种逻辑最后调用 Engine 的 OpenLevel 等接口。  
2. Players，虽然一般来说我们直接控制 Players 的机会不多，都是配置好了就行。但要是到了需要的时候，GameInstance 也实现了许多的接口可以让你动态的添加删除 Players。  
3. UI，UE 的 UI 是另一套 World 之外的系统，虽然同属于 Viewport 的显示之下，但是控制结构跟 Actor 们并不一样。所以我们常常会需要控制 UI 各种切换的业务逻辑，虽然在 Widget 的 Graph 里也可以写些简单的切换，但是要想复用某些切换逻辑的时候，在特定的 Wdiget 里就不合适了，而 GameMode 一方面局限于 Level，另一方面又只存在于 Server；PlayerController 也是会切换掉的，同时又只存在于 World 中，所以最后比较合适的就剩下 GameInstance 了，以后当然有可能了可能会扩展出个 UI 的业务逻辑 Manger 类，不过那是后话了。  
4. 全局的配置，也常常需要根据平台改变一些游戏的配置，Execute 一些 ConsoleCommand，GameInstance 也是这些命令的存放地。  
5. 游戏的额外第三方逻辑，如果你的游戏需要其他一些控制，比如自己写的网络通信、自定义的配置文件或者自己的一些程序算法，如果简单的话，GameInstance 也可以一放，等复杂起来了，也可以把 GameInstance 当作一个模块容器，你可以在里面再扩展出来其他的子逻辑模块。当然如果是插件的话，还是在自己的插件 Module 里面自行管理逻辑，然后把协调工作交给 GameInstance 来做。

而在数据层面上，我们层层上来，已经有了针对一个 Player 的 Contoller 的 PlayerState，也有了针对 World 的 GameMode 的 GameState，到了更全局之上，自然的 GameInstance 就应该存储一些全局的状态数据。所以你可以在 GameInstance 的成员变量中添加一些全局的状态，或者是那些想要在 Level 之外持续存在的对象。不过需要注意的一点是，GameInstance 成员变量中最好只保存那些 “临时” 的数据，而对于那些想要持久序列化保存的数据，我们就需要接下来的 SaveGame 了。把持久的数据直接放在 SaveGame，用的时候直接读取出来，之后再直接在其上更新，好处是只用维护一份，省得要保存的时候，还去想到底要选 GameInstance 的哪些成员变量中来保存，一开始就设计选好，以后就方便了。

## SaveGame

UE 连玩家存档都帮你做了！得益于 UObject 的序列化机制，现在你只需要继承于 USaveGame，并添加你想要的那些属性字段，然后这个结构就可以序列化保存下来的。玩家存档也是游戏中一个非常常见的功能，差的引擎一般就只提供给你读写文件的接口，好一点的会继续给你一些序列化机制，而更好的则会服务得更加周到。UE 为我们在蓝图里提供了 SaveGame 的统一接口，让你只用关心想序列化的数据。  
USaveGame 其实就是为了提供给 UE 一个 UObject 对象，本身并不需要其他额外的控制，所以它的类是如此的简单以至于我能直接把它的全部声明展示出来：

```
UCLASS(abstract, Blueprintable, BlueprintType)
class ENGINE_API USaveGame : public UObject
{
 /**
 * @see UGameplayStatics::CreateSaveGameObject
 * @see UGameplayStatics::SaveGameToSlot
 * @see UGameplayStatics::DoesSaveGameExist
 * @see UGameplayStatics::LoadGameFromSlot
 * @see UGameplayStatics::DeleteGameInSlot
 */

 GENERATED_UCLASS_BODY()
};
```

而 UGameplayStatics 作为暴露给蓝图的接口实现部分，其内部的实现是：  

![[60ff3a0b77a9a16f3b85d5e1e0884963_MD5.png]]

先在内存中写入一些 SavegameFileVersion 之类的控制文件头，然后再序列化 USaveGame 对象，接着会找到 ISaveGameSystem 接口，最后交于真正的子类实现文件的保存。目前的默认实现是 FGenericSaveGameSystem，其内部也只是转发到直接的文件读写接口上去。但你也可以实现自己的 SaveGameSystem，不管是写文件或者是网络传输，保存到不同的地方去。或者是内部调用 OnlineSubsystem 的 Storage 接口，直接把玩家存档保存到 Steam 云存储中也可以。

因此可见，单单是玩家存档这件边角的小事，UE 作为一个深受游戏开发淬炼过的引擎，为了方便自己，也同时造福我们广大开发者，已经实现了这么一套完善的机制。

关于存档数据关联的逻辑，再重复几句，对于那些需要直接在全局处理的数据逻辑，也可以直接在 SaveGame 中写方法来实现。比如实现 AddCoin 接口，对外隐藏实现，对内可以自定义附加一些逻辑。USaveGame 可以看作是一个全局持久数据的业务逻辑类。跟 GameInstance 里的数据区分就是，GameInstance 里面的是临时的数据，SaveGame 里是持久的。清晰这一点区分，到时就不会纠结哪些属性放在哪里，哪些方法实现在哪里了。

注意一下，SaveGameToSlot 里的 SlotName 可以理解为存档的文件名，UserIndex 是用来标识是哪个玩家在存档。UserIndex 是预留的，在目前的 UE 实现里并没有用到，只是预留给一些平台提供足够的信息。你也可以利用这个信息来为多个不同玩家生成不同的最后文件名什么的。而 ISaveGameSystem 是 IPlatformFeaturesModule 提供的模块接口，关于模块的机制，等引擎流程章节再说吧，目前可以简单理解为一个单件对象里提供了一些平台相关的接口对象。

## 总结

至此，我们可以说已经介绍完了 GamePlay 下半部分——逻辑控制。在蓝图层，UE 并不向 BP 直接暴露 Engine 概念，即使在 C++ 层，在实现 GamePlay 业务时也是很少需要真正直接操纵 Engine 的时候。如果 GamePlay 已经足够好，那么 Engine 自然就可以隐居幕后了。UE 用 GameInstance 实现了全局的控制，并支持多 GameInstance 来实现编辑器，最后在存档的时候还可以用到 SaveGame 的方便的接口。  
下篇，就是 GamePlay 章节的最终章，我们将会对 GamePlay 架构的（一到九）篇进行回顾归纳总结巩固，以一个承上启下总览的眼光，再来重新审视一下 UE 的整套 GamePlay 框架，下个章节见。

上篇：[《InsideUE4》GamePlay 架构（八）Player](https://zhuanlan.zhihu.com/p/23826859)

下篇：[《InsideUE4》GamePlay 架构（十）总结](https://zhuanlan.zhihu.com/p/24170697)

## 引用

1.  [SaveGame](https://docs.unrealengine.com/latest/INT/Gameplay/SaveGame/index.html)

_UE4.14_

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**