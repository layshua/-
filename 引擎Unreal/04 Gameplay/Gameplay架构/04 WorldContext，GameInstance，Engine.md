## 引言  

前文提到说一个 World 管理多个 Level，并负责它们的加载释放。那么，问题来了，一个游戏里是只有一个 World 吗？

## WorldContext

答案是否定的，首先 World 就不是只有一种类型，比如编辑器本身就也是一个 World，里面显示的游戏场景也是一个 World，这两个 World 互相协作构成了我们的编辑体验。然后点播放的时候，引擎又可以生成新的类型 World 来让我们测试。简单来说，UE 其实是一个平行宇宙世界观。  
以下是一些世界类型：

```
namespace EWorldType
{
	enum Type
	{
		None,		// An untyped world, in most cases this will be the vestigial worlds of streamed in sub-levels
		Game,		// The game world
		Editor,		// A world being edited in the editor
		PIE,		// A Play In Editor world
		Preview,	// A preview world for an editor tool
		Inactive	// An editor world that was loaded but not currently being edited in the level editor
	};
}
```

而 UE 用来管理和跟踪这些 World 的工具就是 WorldContext：  

![[4d6b1ac2e1f7b9da9d34c1396b65b0b9_MD5.png]]

  
FWorldContext 保存着 ThisCurrentWorld 来指向当前的 World。而当需要从一个 World 切换到另一个 World 的时候（比如说当点击播放时，就是从 Preview 切换到 PIE），FWorldContext 就用来保存切换过程信息和目标 World 上下文信息。所以一般在切换的时候，比如 OpenLevel，也都会需要传 FWorldContext 的参数。一般就来说，对于独立运行的游戏，WorldContext 只有唯一个。而对于编辑器模式，则是一个 WorldContext 给编辑器，一个 WorldContext 给 PIE（Play In Editor）的 World。一般来说我们不需要直接操作到这个类，引擎内部已经处理好各种 World 的协作。  
不仅如此，同时 FWorldContext 还保存着 World 里 Level 切换的上下文：

```
struct FWorldContext
{
    [...]
	TEnumAsByte<EWorldType::Type>	WorldType;

	FSeamlessTravelHandler SeamlessTravelHandler;

	FName ContextHandle;

	/** URL to travel to for pending client connect */
	FString TravelURL;

	/** TravelType for pending client connects */
	uint8 TravelType;

	/** URL the last time we traveled */
	UPROPERTY()
	struct FURL LastURL;

	/** last server we connected to (for "reconnect" command) */
	UPROPERTY()
	struct FURL LastRemoteURL;

}
```

这里的 TravelURL 和 TravelType 就是负责设定下一个 Level 的目标和转换过程。  

```
// Traveling from server to server.
UENUM()
enum ETravelType
{
	/** Absolute URL. */
	TRAVEL_Absolute,
	/** Partial (carry name, reset server). */
	TRAVEL_Partial,
	/** Relative URL. */
	TRAVEL_Relative,
	TRAVEL_MAX,
};

void UEngine::SetClientTravel( UWorld *InWorld, const TCHAR* NextURL, ETravelType InTravelType )
{
	FWorldContext &Context = GetWorldContextFromWorldChecked(InWorld);
	// set TravelURL.  Will be processed safely on the next tick in UGameEngine::Tick().
	Context.TravelURL    = NextURL;
	Context.TravelType   = InTravelType;
    [...]
}
```

粗略的流程是 UE 在 OpenLevel 的时候， 先设置当前 World 的 Context 上的 TravelURL，然后在 UEngine::TickWorldTravel 的时候判断 TravelURL 非空来真正执行 Level 的切换。具体的 Level 切换详细流程比较复杂，目前先从大局上理解整体结构。总而言之，WorldContext 既负责 World 之间切换的上下文，也负责 Level 之间切换的操作信息。  

**思考：为何 Level 的切换信息不放在 World 里？**  
因为 UE 有一个逻辑，一个 World 只有一个 PersistentLevel（见上篇），而当我们 OpenLevel 一个 PersistentLevel 的时候，实际上引擎做的是先释放掉当前的 World，然后再创建个新的 World。所以如果我们把下一个 Level 的信息放在当前的 World 中，就不得不在释放当前 World 前又拷贝回来一遍了。  
而 LoadStreamLevel 的时候，就只是在当前的 World 中载入对象了，所以其实就没有这个限制了。

```
void UGameplayStatics::LoadStreamLevel(UObject* WorldContextObject, FName LevelName,bool bMakeVisibleAfterLoad,bool bShouldBlockOnLoad,FLatentActionInfo LatentInfo)
{
	if (UWorld* World = GEngine->GetWorldFromContextObject(WorldContextObject))
	{
		FLatentActionManager& LatentManager = World->GetLatentActionManager();
		if (LatentManager.FindExistingAction<FStreamLevelAction>(LatentInfo.CallbackTarget, LatentInfo.UUID) == nullptr)
		{
			FStreamLevelAction* NewAction = new FStreamLevelAction(true, LevelName, bMakeVisibleAfterLoad, bShouldBlockOnLoad, LatentInfo, World);
			LatentManager.AddNewAction(LatentInfo.CallbackTarget, LatentInfo.UUID, NewAction);
		}
	}
}
```

World->GetLatentActionManager() 其实也算是保存在当前 World 里了。  

**思考：为何 World 和 Level 的切换要放在下一帧再执行？**  
首先 Level 的加载显然是比较慢的，需要载入 Map，相应的 Mesh，Material…… 等等。所以这个操作就必须异步化，异步的话其实就剩下两种方式，一种是先记录下来信息之后再执行；一种是命令模式立马往队列里压个命令之后再执行。注意，因为 OpenLevel 还要相应在主线程生成相应 Actor 对象，所以有些部分还是要在主线程完成的。这两种模式其实都可以达成需求，前者更加简单明了，后者相对统一。UE 也是个进化过来的引擎，也并不是所有的代码都完美无缺。猜想其实也是一开始这么简单就这么做了，后来也没有特别大的改动的动力就一直这样了。引擎最终比的是生产效率的提高，确实也不是代码有多优雅。

## GameInstance

那么这些 WorldContexts 又是保存在哪里的呢？追根溯源：  

![[b3c2ba26d2f5d1c20eba93180028298a_MD5.png]]

  
GameInstance 里会保存着当前的 WorldConext 和其他整个游戏的信息。明白了 GameInstance 是比 World 更高的层次之后，我们也就能明白为何那些独立于 Level 的逻辑或数据要在 GameInstance 中存储了。  
这一点其实也很好理解，大凡游戏引擎都会有一个 Game 的概念，不管是叫 Application 还是 Director，它都是玩家能直接接触到的最根源的操作类。而 UE 的 GameInstance 因为继承于 UObject，所以就拥有了动态创建的能力，所以我们可以通过指定 GameInstanceClass 来让 UE 创建使用我们自定义的 GameInstance 子类。所以不论是 C++ 还是 BP，我们通常会继承于 GameInstance，然后在里面编写应用于整个游戏范围的逻辑。  
因为经常有初学者会问到：我的 Level 切换了，变量数据就丟了，我应该把那些数据放在哪？再清晰直白一点，GameInstance 就是你不管 Level 怎么切换，还是会一直存在的那个对象！

## Engine

让我们继续再往上，终于得见 UE 大神：  

![[fe68f967adb22cfeea680d61a78da666_MD5.png]]

此处 UEngine 分化出了两个子类：UGameEngine 和 UEditorEngine。众所周知，UE 的编辑器也是 UE 用自己的引擎渲染出来的，采用的也是 Slate 那套 UI 框架。好处有很多，比如跨平台比较统一，UI 框架可以复用一套控件库，Dogfood 等等，此处不再细讲。所以本质上来说，UE 的编辑器其实也是个游戏！我们是在编辑器这个游戏里面创造我们自己的另一个游戏。话虽如此，但比较编辑器和游戏还是有一定差别的，所以 UE 会在不同模式下根据编译环境而采用不同的具体 Engine 类，而在基类 UEngine 里通过一个 WorldList 保存了所有的 World。

*   Standalone Game：会使用 UGameEngine 来创建出唯一的一个 GameWorld，因为也只有一个，所以为了方便起见，就直接保存了 GameInstance 指针。
*   而对于编辑器来说，EditorWorld 其实只是用来预览，所以并不拥有 OwningGameInstance，而 PlayWorld 里的 OwningGameInstance 才是间接保存了 GameInstance.

目前来说，因为 UE 还不支持同时运行多个 World（当前只能一个，但可以切换），所以 GameInstance 其实也是唯一的。提前说些题外话，虽然目前网络部分还没涉及到，但是当我们在 Editor 里进行 MultiplePlayer 的测试时，每一个 Player Window 里都是一个 World。如果是 DedicateServer 模式，那 DedicateServer 也会是一个 World。  
最后实例化出来的 UEngine 实例用一个全局的 GEngine 变量来保存。至此，我们已经到了引擎的最根处:

```
//UnrealEngine\Engine\Source\Runtime\Engine\Private\UnrealEngine.cpp
ENGINE_API UEngine*	GEngine = NULL;
```

GEngine 可以说是一切开始的地方了。翻看引擎源码，到处也可以看见从 GEngine-> 出来的引用。  

## GamePlayStatics

既然我们在引擎内部 C++ 层次已经有了访问 World 操作 Level 的能力，那么在暴露出的蓝图系统里，UE 为了我们的使用方便，也在 Engine 层次为我们提供了便利操作蓝图函数库。

```
UCLASS ()
class UGameplayStatics : public UBlueprintFunctionLibrary
```

我们在蓝图里见到的 GetPlayerController、SpawActor 和 OpenLevel 等都是来至于这个类的接口。这个类比较简单，相当于一个 C++ 的静态类，只为蓝图暴露提供了一些静态方法。在想借鉴或者是查询某个功能的实现时，此处往往会是一个入口。  

## 总结

从结构上而言，我们已经来到了最根源的地方。GEngine 仿佛就是一棵大树的根，当我们拎起它的时候，也会带出整个游戏世界的各个对象。但目前这些对象：Object->Actor+Component->Level->World->WorldContext->GameInstance->Engine，确实已经足够表达 UE 游戏世界的各个部分。  
那作为 GamePlay 部分而言，我们还有一个问题：UE 是如何把在该对象结构上表达游戏逻辑的？  
如果说：“程序 = 数据 + 算法”的话，那 UE 的 GamePlay 我们已经讨论完了数据部分，而下篇我们将开始讨论 UE 的游戏逻辑 “算法” 部分。

上篇：[《InsideUE4》GamePlay 架构（二）Level 和 World](https://zhuanlan.zhihu.com/p/22924838)

下篇：[《InsideUE4》GamePlay 架构（四）Pawn](https://zhuanlan.zhihu.com/p/23321666?refer=insideue4)

_UE4.14_

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**