---
title: UE Gameplay架构
create_time: 2023-09-29 17:05
uid: "202309291705"
reference:
  - https://zhuanlan.zhihu.com/insideue4
banner: "[[Pasted image 20230929170757.png]]"
banner_header: 
banner_y: 1
banner_icon: 🎮
banner_lock: true
---
# 1 Actor 和 Component

## UE 创世，万物皆 UObject，接着有 Actor。

### UObject:
[[01 虚幻类#UObject 类]]
起初，UE 创世，有感于天地间 C++ 原始之气一片混沌虚无，便撷取凝实一团 C++ 之气，降下无边魔力，洒下秩序之光，便为这个世界生成了坚实的土壤 UObject，并用 UClass 一一为此命名。

![[fdaabac496005895c71298f5274b99db_MD5.png]]

  
藉着 UObject 提供的元数据、反射生成、GC 垃圾回收、序列化、编辑器可见，Class Default Object 等，UE 可以构建一个 Object 运行的世界。（后续会有一个大长篇深挖 UObject）

### Actor:

世界有了土壤之后，但还少了一些生动色彩，如果女娲造人一般，UE 取一些 UObject 的泥巴，派生出了 Actor。在 UE 眼中，整个世界从此了有了一个个生动的 “演员”，众多的“演员” 们，一起齐心协力为观众上演一场精彩的游戏。  

![[36b2d8ae7a110d280d3b465031def26c_MD5.png]]

脱胎自 Object 的 Actor 也多了一些本事：Replication（网络复制）,Spawn（生生死死），Tick(有了心跳)。  
Actor 无疑是 UE 中最重要的角色之一，组织庞大，最常见的有 StaticMeshActor, CameraActor 和 PlayerStartActor 等。Actor 之间还可以互相 “嵌套”，拥有相对的“父子” 关系。

> [!question] 思考：为何 Actor 不像 GameObject 一样自带 Transform？
> 
> 我们知道，如果一个对象需要在 3D 世界中表示，那么它必然要携带一个 Transform matrix 来表示其位置。关键在于，**在 UE 看来，Actor 并不只是 3D 中的 “表示”，一些不在世界里展示的“不可见对象” 也可以是 Actor，如 AInfo (派生类 AWorldSetting, AGameMode, AGameSession, APlayerState, AGameState 等)，AHUD, APlayerCameraManager 等，代表了这个世界的某种信息、状态、规则**。你可以把这些看作都是一个个默默工作的灵体 Actor。所以，Actor 的概念在 UE 里其实不是某种具象化的 3D 世界里的对象，而是世界里的种种元素，**用更泛化抽象的概念来看，小到一个个地上的石头，大到整个世界的运行规则，都是 Actor**.  
> 
> 当然，你也可以说即使带着 Transform，把坐标设置为原点，然后不可见不就行了？这样其实当然也是可以，不过可能因为 UE 跟贴近 C++ 一些的缘故，所以设计哲学上就更偏向于 C++ 的哲学 “不为你不需要的东西付代价”。一个 Transform 再加上附带的逆矩阵之类的表示，内存占用上其实也是挺可观的。要知道 UE 可是会抠门到连 bool 变量都要写成 uint bPending: 1; 位域来节省一个字节的内存的。  
> 
> 换一个角度讲，如果把带 Transform 也当成一个 Actor 的额外能力可以自由装卸的话，那其实也可以自圆其说。经过了 UE 的权衡和考虑，把 Transform 封装进了 SceneComponent, 当作 RootComponent。但在权衡到使用的便利性的时候，大部分 Actor 其实是有 Transform 的，我们会经常获取设置它的坐标，如果总是得先获取一下 SceneComponent，然后再调用相应接口的话，那也太繁琐了。所以 UE 也为了我们直接提供了一些便利性的 Actor 方法，如 (Get/Set)ActorLocation 等，其实内部都是转发到 RootComponent。

```c++
/*~
 * Returns location of the RootComponent 
 * this is a template for no other reason than to delay compilation until USceneComponent is defined
 */ 
template<class T>
static FORCEINLINE FVector GetActorLocation(const T* RootComponent)
{
    return (RootComponent != nullptr) ? RootComponent->GetComponentLocation() : FVector(0.f,0.f,0.f);
}
bool AActor::SetActorLocation(const FVector& NewLocation, bool bSweep, FHitResult* OutSweepHitResult, ETeleportType Teleport)
{
    if (RootComponent)
    {
        const FVector Delta = NewLocation - GetActorLocation();
        return RootComponent->MoveComponent(Delta, GetActorQuat(), bSweep, OutSweepHitResult, MOVECOMP_NoFlags, Teleport);
    }
    else if (OutSweepHitResult)
    {
        *OutSweepHitResult = FHitResult();
    }
    return false;
}
```

同理，Actor 能接收处理 Input 事件的能力，其实也是转发到内部的 UInputComponent* InputComponent; 同样也提供了便利方法。

## Component


**下面我们来细细看一下 Actor 和 Component 的关系：**  
TSet<UActorComponent*> OwnedComponents 保存着这个 Actor 所拥有的所有 Component, 一般其中会有一个 SceneComponent 作为 RootComponent。  
TArray<UActorComponent*> InstanceComponents 保存着实例化的 Components。实例化是个什么意思呢，就是你在蓝图里 Details 定义的 Component, 当这个 Actor 被实例化的时候，这些附属的 Component 也会被实例化。这其实很好理解，就像士兵手上拿着把武器，当我们拥有一队士兵的时候，自然就一一对应拥有了不同实例化的武器。但 OwnedComponents 里总是最全的。ReplicatedComponents，InstanceComponents 可以看作一个预先的分类。

一个 Actor 若想可以被放进 Level 里，就必须实例化 USceneComponent* RootComponent。但如果你光看代码的话，OwnedComponents 其实也是可以包容多个不同 SceneComponent 的，然后你可以动态获取不同的 SceneComponent 来当作 RootComponent，只不过这种用法确实不太自然，而且也得非常小心维护不同状态，不推荐如此用。在我们的直觉印象里，一个封装过后的 Actor 应该是一个整体，它能被放进 Level 中，拥有变换，这一整个整体的概念更加符合自然意识，所以我想，这也是 UE 为何要在 Actor 里一一对应一个 RootComponent 的原因。

再来说说 Component 下面的家族（为了阐明概念，只列出了最常见的）：

![[dd7a997f259b317110c12031d2bfc994_MD5.png]]

ActorComponent 下面最重要的一个 Component 就非 SceneComponent 莫属了。SceneComponent 提供了两大能力：一是 Transform，二是 SceneComponent 的互相嵌套。

![[6ed5ef4dbe67c9b5a2db08151d8bc1a0_MD5.jpg]]

  

> [!question] 思考：为何 ActorComponent 不能互相嵌套？而在 SceneComponent 一级才提供嵌套？
> 首先，ActorComponent 下面当然不是只有 SceneComponent，一些 UMovementComponent，AIComponent，或者是我们自己写的 Component，都是会直接继承 ActorComponent 的。但很奇怪的是，ActorComponent 却是不能嵌套的，在 UE 的观念里，好像只有带 Transform 的 SceneComponent 才有资格被嵌套，好像 Component 的互相嵌套必须和 3D 里的 transform 父子对应起来。
>   
> 老实说，如果让我来设计 Entity-Component 模式，我很可能会为了通用性而在 ActorComponent 这一级直接提供嵌套，这样所有的 Component 就与生俱来拥有了组合其他 Component 的能力，灵活性大大提高。但游戏引擎的设计必然也经过了各种权衡，虽然说架构上显得并不那么的统一干净，但其实也大大减少了被误用的机会。实体组件模式推崇的 “组合优于继承” 的概念确实很强大，但其实同时也带来了一些问题，如 Component 之间如何互相依赖，如何互相通信，嵌套过深导致的接口便利损失和性能损耗，真正一个让你随便嵌套的组件模式可能会在使用上更容易出问题。  
> 
> 从功能上来说，UE 更倾向于编写功能单一的 Component（如 UMovementComponent）, 而不是一个整合了其他 Component 的大管家 Component（当然如果你偏要这么干，那 UE 也阻止不了你）。  
> 而从游戏逻辑的实现来说，UE 也是不推荐把游戏逻辑写在 Component 里面，所以你其实也没什么机会去写一个很复杂的 Component.

> [!NOTE] **思考：Actor 的 SceneComponent 哲学**  
>
> 很多其他游戏引擎，还有一种设计思路是 “万物皆 Node”。Node 都带变换。比如说你要设计一辆汽车，一种方式是车身作为一个 Node, 4 个轮子各为车身的子 Node，然后移动父 Node 来前进。而在 UE 里，一种很可能的方式就变成，汽车是一个 Actor，车身作为 RootComponent，4 个轮子都作为 RootComponent 的子 SceneComponent。请读者们细细体会这二者的区别。两种方式都可以实现出优秀的游戏引擎，只是有些理念和侧重点不同。 
>  
> 从设计哲学上来说，其实你把万物看成是 Node，或者是 Component，并没有什么本质上的不同。看作 Node 的时候，Node 你就要设计的比较轻量廉价，这样才能比较没有负担的创建多个，同理 Component 也是如此。Actor 可以带多个 SceneComponent 来渲染多个 Mesh 实体，同样每个 Node 带一份 Mesh 再组合也可以实现出同样效果。  
> 
> 个人观点来说，关键的不同是在于你是怎么划分要操作的实体的粒度的。当看成是 Node 时，因为 Node 身上的一些通用功能（事件处理等），其实我们是期望着我们可以非常灵活的操作到任何一个细小的对象，我们希望整个世界的所有物体都有一些基本的功能（比如说被拾取），这有点完美主义者的思路。而注重现实的人就会觉得，整个游戏世界里，有相当大一部分对象其实是不那么动态的。比如车子，我关心的只是整体，而不是细小到每一个车轱辘。这种理念就会导成另外一种设计思路：把要操作的实体按照功能划分，而其他的就尽量只是最简单的表示。所以在 UE 里，其实是把 5 个薄薄的 SceneComponent 表示再用 Actor 功能的盒子装了起来，而在这个盒子内部你可以编写操作这 5 个对象的逻辑。换做是 Node 模式，想编写操作逻辑的话，一般就来说就会内化到父 Node 的内部，不免会有逻辑与表现掺杂之嫌，而如果 Node 要把逻辑再用组合分离开的话，其实也就转化成了某种 ScriptComponent。

> [!question] **思考：Actor 之间的父子关系是怎么确定的？**  
> 
> 你应该已经注意到了 Actor 里面的 TArray<AActor*> Children 字段，所以你可能会期望看到 Actor:AddChild 之类的方法，很遗憾。在 UE 里，Actor 之间的父子关系却是通过 Component 确定的。同一般的 Parent:AddChild 操作原语不同，UE 里是通过 `Child:AttachToActor` 或 `Child:AttachToComponent` 来创建父子连接的。

```c++
void AActor::AttachToActor(AActor* ParentActor, const FAttachmentTransformRules& AttachmentRules, FName SocketName)
{
    if (RootComponent && ParentActor)
    {
        USceneComponent* ParentDefaultAttachComponent = ParentActor->GetDefaultAttachComponent();
        if (ParentDefaultAttachComponent)
        {
            RootComponent->AttachToComponent(ParentDefaultAttachComponent, AttachmentRules, SocketName);
        }
    }
}
void AActor::AttachToComponent(USceneComponent* Parent, const FAttachmentTransformRules& AttachmentRules, FName SocketName)
{
    if (RootComponent && Parent)
    {
        RootComponent->AttachToComponent(Parent, AttachmentRules, SocketName);
    }
}
```

3D 世界里的 “父子” 关系，我们一般可能会认为就是 3D 世界里的变换的坐标空间 “父子” 关系，但如果再度扩展一下，如上所述，一个 Actor 可是可以带有多个 SceneComponent 的，这意味着一个 Actor 是可以带有多个 Transform“锚点”的。创建父子时，你到底是要把当前 Actor 当作对方哪个 SceneComponent 的子？再进一步，如果你想更细控制到 Attach 到某个 Mesh 的某个 Socket（关于 Socket Slot，目前可以简单理解为一个虚拟插槽，提供变换锚点），你就更需要去寻找到特定的变换锚点，然后 Attach 的过程分别在 Location,Roator,Scale 上应用 Rule 来计算最后的位置。  

```c++
/** Rules for attaching components - needs to be kept synced to EDetachmentRule */
UENUM()
enum class EAttachmentRule : uint8
{
    /** Keeps current relative transform as the relative transform to the new parent. */
    KeepRelative,
    /** Automatically calculates the relative transform such that the attached component maintains the same world transform. */
    KeepWorld,
    /** Snaps transform to the attach point */
    SnapToTarget,
};
```

所以 Actor 父子之间的 “关系” 其实隐含了许多数据，而这些数据都是在 Component 上提供的。Actor 其实更像是一个容器，只提供了基本的创建销毁，网络复制，事件触发等一些逻辑性的功能，而把父子的关系维护都交给了具体的 Component，所以更准确的说，其实是不同 Actor 的 SceneComponent 之间有父子关系，而 Actor 本身其实并不太关心。  

接下来的左侧派生链依次提供了物理，材质，网格最终合成了一个我们最普通常见的 StaticMeshComponent。而右侧的 ChildActorComponent 则是提供了 Component 之下再叠加 Actor 的能力。  

**聊一聊 ChildActorComponent**  
同作为最常用到的 Component 之一，ChildActorComponent 担负着 Actor 之间互相组合的胶水。这货在蓝图里静态存在的时候其实并不真正的创建 Actor，而是在之后 Component 实例化的时候才真正创建。

```c++
void UChildActorComponent::OnRegister()
{
    Super::OnRegister();
    if (ChildActor)
    {
        if (ChildActor->GetClass() != ChildActorClass)
        {
            DestroyChildActor();
            CreateChildActor();
        }
        else
        {
            ChildActorName = ChildActor->GetFName();
            USceneComponent* ChildRoot = ChildActor->GetRootComponent();
            if (ChildRoot && ChildRoot->GetAttachParent() != this)
            {
                // attach new actor to this component
                // we can't attach in CreateChildActor since it has intermediate Mobility set up
                // causing spam with inconsistent mobility set up
                // so moving Attach to happen in Register
                ChildRoot->AttachToComponent(this, FAttachmentTransformRules::SnapToTargetNotIncludingScale);
            }
            // Ensure the components replication is correctly initialized
            SetIsReplicated(ChildActor->GetIsReplicated());
        }
    }
    else if (ChildActorClass)
    {
        CreateChildActor();
    }
}
void UChildActorComponent::OnComponentCreated()
{
    Super::OnComponentCreated();
    CreateChildActor();
}
```

这就导致了一个问题，当你把一个 ActorClass 拖进 Level 后，这个 Actor 实际是已经实例化了, 你可以直接调整这个 Actor 的属性。但是你把它拖到另一个 Actor Class 里，它只会给你空空白白的 ChildActorComponent 的 DetailsPanel，你想调整 Actor 的属性，就只能等生成了之后，用蓝图或代码去修改。这一点来说，其实还是挺不方便的，我个人觉得应该是还有优化的空间。

## 修订

### 4.14 Child Actor Templates

UE 终于听到了人民群众的呼声，在 4.14 里增加了 Child Actor Templates 来支持在子 ChildActor 的 DetailsPannel 里查看和修改属性。

![[50aefb50cc62c8b650889c838af882db_MD5.jpg]]

```c++
/** The class of Actor to spawn */
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=ChildActorComponent, meta=(OnlyPlaceable, AllowPrivateAccess="true", ForceRebuildProperty="ChildActorTemplate"))
TSubclassOf<AActor>	ChildActorClass;

/** The actor that we spawned and own */
UPROPERTY(Replicated, BlueprintReadOnly, Category=ChildActorComponent, TextExportTransient, NonPIEDuplicateTransient, meta=(AllowPrivateAccess="true"))
TObjectPtr<AActor>	ChildActor;

/** Property to point to the template child actor for details panel purposes */
UPROPERTY(VisibleDefaultsOnly, DuplicateTransient, Category=ChildActorComponent, meta=(ShowInnerProperties))
TObjectPtr<AActor> ChildActorTemplate;
```

最新的 UE 代码里已经提供了 ChildActorTemplate 这个对象，来为 ChildActorClass 生成一个 Actor 模板，方便我们编辑属性，之后在生成 ChildActor 的时候，可以把 ChildActorTemplate 身上的属性拷贝给 ChildActor，这样运作看起来就像生成了你配置的那个 Actor，也比较流畅自然了。

# 2 Level 和 World
## 引言  

上文谈到 Actor 和 Component 的关系，UE 利用 Actor 的概念组成一片游戏对象森林，并利用 Component 组装扩展 Actor 的能力，让世界里拥有了形形色色的 Actor 们，拥有了自由表达 3D 世界的能力。  
那么，这些 Actor 们，到底是怎么组织起来的呢？

既然提到了世界，我们的直觉反应是采用一个 "World" 对象来包容所有的 Actor 们。但是当游戏的虚拟世界非常巨大时，这种方式就捉襟见肘了。首先，目前虽然 PC 的性能日益强大，但是依然内存也限制了不能一下子加载进所有的游戏资源；其次，因为玩家的活动和可见范围有限，为了最优性能，把即使是很远的跟玩家无关的对象也考虑进来也明显是不明智的。所以我们**需要一种更细粒度的概念来划分世界。**

不同的游戏引擎们，看待这个过程的角度和理念也不一样。 UE 中用关卡（Level）来划分世界，由一个或多个 Level 组成一个 World。  
不要觉得这种划分好像很随意，只是个名字不同而已。实际上一个游戏引擎的 “世界观” 关系到了一整串后续的内容组织，玩家的管理，世界的生成，变换和毁灭。游戏引擎内部的资源的加载释放也往往都是和这种划分（Level）绑定在一起的。

## Level

在 UE 的世界中，我们之前已经有了空气（C++）, 土壤（UObject），物件（Actor）。而现在 UE 又施展神力创建了一片片大陆（Level），在这片大陆上（. map 文件），Actor 们秩序井然，各种地形拔地而起，植被繁茂，天空雾云缭绕，圣光普照，这也是玩家们降生开始精彩冒险的地方。  

![[72477eba63e36061f9a9b028f933c3d9_MD5.png]]

可以从 ULevel 的前缀 U 看出来 Level（大陆）也确实是继承于 UObject（土壤）的。那既然同属于 Object 下面的各 Actor 们都拥有了一定的智能能力（支持蓝图脚本），Level 自然也得体现出大地的意志，所以默认带了一个土地公（ALevelScriptActor），允许我们在关卡里编写脚本，可以对本关卡里的所有 Actor 通过名字呼之则来，关卡蓝图实际上就代表着该片大陆上的运行规则。  
在 Level 已经有了管理者之后，一开始大家都挺满意，但渐渐的就发现，好像各个 Level 需要的功能好像都差不多，都是修改一下光照，物理等一些属性。所以为了方便起见，UE 便给每一个 Level 也都默认配了一个书记官（Info），他一一记录着本 Level 的各种规则属性，在 UE 需要的时候便负责相告。更重要的是，在 Level 需要有其他管理人员一起协助的时候，他也记录着 “游戏模式” 的名字来让 UE 可以指派。  
前面我们说过，有一些 Actor 是不 “显示” 的（没有 SceneComponent），是不能 “摆放” 到 Level 里的，但是它依然可以在关卡里出力。其中一个家族系列就是 AInfo 和其之类。今天我们只简单介绍一下跟 Level 直接相关的一位书记官：AWorldSettings。  

![[5980f02a5ea5f4f706d7c33329d8f688_MD5.jpg]]

**其实虽然名字叫做 WorldSettings，但其实只是跟 Level 相关**，我猜可能是在上古时代，当时整个世界只有一块大陆，人们就以为当前的大陆就是整个世界，所以给这块大陆的设置就起名为 WorldSettings，后来等技术进步了，发现必须有其他大陆了，这个名字已经用得太多反而不好改了，就只好遗留下来了。当然也有可能是因为当 Level 被添加进 World 后，这个 Level 的 Settings 如果是主 PersistentLevel，那它就会被当作整个 World 的 WorldSettings。  
注意，Actors 里也保存着 AWorldSettings 和 ALevelScriptActor 的指针，所以 Actors 实际上确实是保存了所有 Actor。

> [!question]  思考：为何 AWorldSettings 要放进在 Actors[0] 的位置？而 ALevelScriptActor 却不用？
>
> 实际上通过下面这段代码可知，Actors 们的排序依据是把那些 “非网络” 的 Actor 放在前面，而把 “网络可复制” 的 Actor 们放在后面，然后加一个起始索引标记 iFirstNetRelevantActor，相当于为网络 Actor 划分了一个缓存，从而加速了网络复制时的检测速度。
> 
> **AWorldSettings 因为都是静态的数据提供者，在游戏运行过程中也不会改变，不需要网络复制，所以也就可以一直放在前列，而如果再加个规则，一直放在第一个的话，也能同时把 AWorldSettings 和其他的前列 Actor 们再度区分开，在需要的时候也能加速判断。**
> 
>**ALevelScriptActor 因为是代表关卡蓝图，是允许携带 “复制” 变量函数的，所以也有可能被排序到后列。**

```c++
void ULevel::SortActorList()
{
    //[...]
    TArray<AActor*> NewActors;
    TArray<AActor*> NewNetActors;
    NewActors.Reserve(Actors.Num());
    NewNetActors.Reserve(Actors.Num());
    // The WorldSettings tries to stay at index 0
    NewActors.Add(WorldSettings);
    // Add non-net actors to the NewActors immediately, cache off the net actors to Append after
    for (AActor* Actor : Actors)
    {
        if (Actor != nullptr && Actor != WorldSettings && !Actor->IsPendingKill())
        {
            if (IsNetActor(Actor))
            {
                NewNetActors.Add(Actor);
            }
            else
            {
                NewActors.Add(Actor);
            }
        }
    }
    iFirstNetRelevantActor = NewActors.Num();
    NewActors.Append(MoveTemp(NewNetActors));
    Actors = MoveTemp(NewActors);   // Replace with sorted list.
    // Add all network actors to the owning world
    //[...]
}
```



> [!question] 思考：既然 ALevelScriptActor 也继承于 AActor, 为何关卡蓝图不设计能添加 Component？
> 
> 平常我们在创建 Actor 的时候，我们蓝图界面是可以创建 Component 的。  那为什么在关卡蓝图里，却不能这么做（没有提供该界面功能）？  
> 我虽然在图里标出了 Level 中拥有 ModelComponents，但那其实只是针对 BSP 应用的一个子集。通过源码发现，其实 UE 自己也是在 C++ 里往 ALevelScriptActor 添加 UInputComponent 来实现关卡蓝图可以响应事件。
> 

```c++
void ALevelScriptActor::PreInitializeComponents()
{
    if (UInputDelegateBinding::SupportsInputDelegate(GetClass()))
    {
        // create an InputComponent object so that the level script actor can bind key events
        InputComponent = NewObject<UInputComponent>(this);
        InputComponent->RegisterComponent();
        UInputDelegateBinding::BindInputDelegates(GetClass(), InputComponent);
    }
    Super::PreInitializeComponents();
}
```

其实既然 ALevelScriptActor 是个 Actor，那意味着我们当然可以为它添加组件，实际上也确实可以这么做。比如你可以在关卡蓝图里这么干：  

![[2bca26faf954e0069c1f4128b6ef3524_MD5.png]]

而如果你实际意识到关卡蓝图本身就是一个看不见的 Actor，你就可以在上面用 Actor 的各种操作：  

![[05692ae305831c4fc34577f462cd4afa_MD5.png]]

在关卡蓝图里的 self 其实也是个 Actor！虽然一般这么干也没什么毛用。  

**那么好好想想，为啥 UE 要给你这么一个关卡蓝图界面呢？**  
在此，我也只能进行一番猜测，ALevelScriptActor 作为一个特化的 Actor, 却把 Components 列表界面给隐藏了，**说明 UE 其实是不希望我们去复杂化关卡构成的。**  
假设说 UE 开放了关卡 Component，那么我们在创建组件时就必然要考虑一个问题：哪些是 ActorComponent，哪些是 LevelComponent，再怎么 ALevelScriptActor 本质是个 Actor，但 Level 的概念还是要突出，ALevelScriptActor 的 Actor 本质是要隐藏的。所以用户就会多一些心智负担，可能混淆。而如果像这样不开放，大家的思路就都转向先创建个 Actor，然后再往之上添加 component，思路会比较统一清晰。  
再之，从游戏逻辑的组织上来说，Level 其实更应该表现为一个 Actor 的容器。**UE 其实也是不鼓励在 Level 里编写太复杂的逻辑的。所以才接着会有了之后的 GameMode, Controller 那些真正的逻辑控制类（后续会再细讨论）。**  
所以游戏引擎也并不是说最大化的暴露一切功能给你就是最好的，有时候选择太多了反而容易出错。在这一点上，我觉得 UE 很好的保持了克制，为我们提供了一个优秀的清晰的不易出错的框架，同时也对高阶用户保留了灵活性。

## World

终于，到了把大陆们（Level）拼装起来的时候了。可以用 SubLevel 的方式：

![[3d39261b1e08684bf04385aee979521c_MD5.png]]

也支持 WorldComposition 的方式自动把项目里的所有 Level 都组合起来，并设置摆放位置：

![[5518cb072e076559c9320782e502e763_MD5.jpg]]

具体摆放的操作和技巧并不是本文的重点。简单本质来说，就是一个 World 里有多个 Level，这些 Level 在什么位置，是在一开始就加载进来，还是 Streaming 运行时加载。  
**UE 里每个 World 支持一个 PersistentLevel 和多个其他 Level：**  

![[db0281595e0687c54921e249851819e0_MD5.png]]

Persistent 的意思是一开始就加载进 World，Streaming 是后续动态加载的意思。Levels 里保存有所有的当前已经加载的 Level，StreamingLevels 保存整个 World 的 Levels 配置列表。PersistentLevel 和 CurrentLevel 只是个快速引用。**在编辑器里编辑的时候，CurrentLevel 可以指向其他 Level，但运行时 CurrentLevel 只能是指向 PersistentLevel。**

> [!question] 
> **思考：为何要有主 PersistentLevel？**  

首先，World 至少得有一个 Level，就像你也得先出生在一块大陆上才可以继续谈起去探索别的新大陆。所以这块玩家出生的大陆就是主 Level 了。当然了，因为我们也可以同时配置别的 Level 一开始就加载进来，其实跟 PersistentLevel 是差不多等价的，**但再考虑到另一问题：Levels 拼接进 World 一起之后，各自有各自的 worldsetting，那整个 World 的配置应该以谁的为主？**

```c++
AWorldSettings* UWorld::GetWorldSettings( bool bCheckStreamingPesistent, bool bChecked ) const
{
    checkSlow(IsInGameThread());
    AWorldSettings* WorldSettings = nullptr;
    if (PersistentLevel)
    {
        WorldSettings = PersistentLevel->GetWorldSettings(bChecked);
        if( bCheckStreamingPesistent )
        {
            if( StreamingLevels.Num() > 0 &&
                StreamingLevels[0] &&
                StreamingLevels[0]->IsA<ULevelStreamingPersistent>()) 
            {
                ULevel* Level = StreamingLevels[0]->GetLoadedLevel();
                if (Level != nullptr)
                {
                    WorldSettings = Level->GetWorldSettings();
                }
            }
        }
    }
    return WorldSettings;
}
```

**可以看出，World 的 Settings 也是以 PersistentLevel 为主的**，但这也并不意味着其他 Level 的 Settings 就完全没有作用了，本篇也无法一一列出所有配置选项来说明，简单来说，就是需要在整个世界范围内起作用的配置选项（比如 VR 的 WorldToMeters，KillZ，WorldGravity 其他大部分都是）就是需要从主 PersistentLevel 的配置中提取。而一些配置选项可以在单独 Level 中起作用的，比如在编辑 Level 时的光照质量配置就是一个个 Level 单独的，目前这种配置很少，但可能以后也会增加。在这里只是阐明一个为主其他为辅的 Level 配置系统。

> [!question] 
> **思考：Levels 们的 Actors 和 World 有直接关系吗？**  

当别的 Level 被添加进当前 World 之后，我们能直接在 WorldOutliner 里看到其他 Level 的 Actor 们。  

![[f574ef7fcefa7b1d4986b1ea6b322ac0_MD5.png]]

但这并不代表着 World 直接引用了 Level 里的 Actor 们。`TActorIteratorBase`（World 的 Actor 迭代器）内部的实现也只是在遍历 Levels 来获得所有 Actor。当然 World 为了更快速的操作 Controllers 和 Pawn 也都保存了引用。但 Levels 却共享着 World 的一个 PhysicsScene，这也意味着 Levels 里的 Actors 的物理实体其实都是在 World 里的，这也好理解，毕竟物理的碰撞之类的当然要是全局的了。再说到导航，World 在拼接 Level 的时候，也是会同时把两个 Level 的导航网格给 “拼接” 起来的。当然目前还不是深入细节的时候，现在只要从大局上明白 World-Level-Actor 的关系。

> [!question] 
> **思考：为什么要在 Level 里保存 Actors，而不是把所有 Map 的 Actors 配置都生成在 World 一个总 Actors 里？**  

这肯定也是一种实现方式，好处是把整个 World 看成一个整体，所有的 actors 都从属于 world，这样就不存在 Level 边界，可以更整体的处理 Actors 的作用范围和判定问题，实现上也少了拼接导航等步骤。当然坏处也是模糊了 Level 边界，这样在加载进一个 Level 之后，之后再动态释放，就需要再重新再从整体中抽离出部分来释放，这个筛选过程也会产生比较大的损耗。试着去理解 UE 的权衡，应该是尽量的把损耗平摊（这里是把 Level 加载释放的损耗尽量减小），才不会产生比较大的帧率波动，让玩家感觉到卡帧。

## 总结


 - 一个或多个 Level 组成 World，每个 Level 保存当前所有的 Actor。
- WorldSetting 并不是设置 World 的属性（不要混肴），是针对 Level 的设置。仅代表当前一个关卡，并不是所有关卡。
- Level 作为 Actor 的容器，同时也划分了 World，一方面支持了 Level 的动态加载，另一方面也允许了团队的实时协作，大家可以同时并行编辑不同的 Level。
- 一般而言，一个玩家从游戏开始到结束，UE 会创造一个 GameWorld 给玩家并一直存在。玩家切换场景或关卡，也只是在这个 World 中加载释放不同的 Level。

既然 Level 拥有了管理者（LevelScriptActor），玩家可以编写特定关卡的逻辑，那么我们能否对 World 这种层次编写逻辑呢？答案是肯定的，不过本文篇幅有限，敬请期待下篇。

# 3 WorldContext，GameInstance，Engine
## 引言  

前文提到说一个 World 管理多个 Level，并负责它们的加载释放。那么，问题来了，**一个游戏里是只有一个 World 吗？**

## WorldContext

**答案是否定的**，首先 World 就不是只有一种类型，比如编辑器本身就也是一个 World，里面显示的游戏场景也是一个 World，这两个 World 互相协作构成了我们的编辑体验。然后点播放的时候，引擎又可以生成新的类型 World 来让我们测试。简单来说，UE 其实是一个平行宇宙世界观。  
以下是一些世界类型：

```c++
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

**而 UE 用来管理和跟踪这些 World 的工具就是 `WorldContext`：**  

![[4d6b1ac2e1f7b9da9d34c1396b65b0b9_MD5.png]]

  
FWorldContext 保存着 ThisCurrentWorld 来指向当前的 World。而当需要从一个 World 切换到另一个 World 的时候（比如说当点击播放时，就是从 Preview 切换到 PIE），FWorldContext 就用来保存切换过程信息和目标 World 上下文信息。所以一般在切换的时候，比如 OpenLevel，也都会需要传 FWorldContext 的参数。一般就来说，对于独立运行的游戏，WorldContext 只有唯一个。而对于编辑器模式，则是一个 WorldContext 给编辑器，一个 WorldContext 给 PIE（Play In Editor）的 World。一般来说我们不需要直接操作到这个类，引擎内部已经处理好各种 World 的协作。  
不仅如此，同时 FWorldContext 还保存着 World 里 Level 切换的上下文：

```c++
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

```c++
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

粗略的流程是 UE 在 OpenLevel 的时候，先设置当前 World 的 Context 上的 TravelURL，然后在 UEngine:: TickWorldTravel 的时候判断 TravelURL 非空来真正执行 Level 的切换。具体的 Level 切换详细流程比较复杂，目前先从大局上理解整体结构。**总而言之，WorldContext 既负责 World 之间切换的上下文，也负责 Level 之间切换的操作信息。**  

> [!question] 
> **思考：为何 Level 的切换信息不放在 World 里？**  

因为 UE 有一个逻辑，一个 World 只有一个 PersistentLevel（见上篇），而**当我们 OpenLevel 一个 PersistentLevel 的时候，实际上引擎做的是先释放掉当前的 World，然后再创建个新的 World**。所以如果我们把下一个 Level 的信息放在当前的 World 中，就不得不在释放当前 World 前又拷贝回来一遍了。  
而 LoadStreamLevel 的时候，就只是在当前的 World 中载入对象了，所以其实就没有这个限制了。

```c++
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

World->GetLatentActionManager () 其实也算是保存在当前 World 里了。  

> [!question] 
> **思考：为何 World 和 Level 的切换要放在下一帧再执行？**  

首先 Level 的加载显然是比较慢的，需要载入 Map，相应的 Mesh，Material…… 等等。所以这个操作就必须异步化，异步的话其实就剩下两种方式，一种是先记录下来信息之后再执行；一种是命令模式立马往队列里压个命令之后再执行。注意，因为 OpenLevel 还要相应在主线程生成相应 Actor 对象，所以有些部分还是要在主线程完成的。这两种模式其实都可以达成需求，前者更加简单明了，后者相对统一。UE 也是个进化过来的引擎，也并不是所有的代码都完美无缺。猜想其实也是一开始这么简单就这么做了，后来也没有特别大的改动的动力就一直这样了。引擎最终比的是生产效率的提高，确实也不是代码有多优雅。

## GameInstance

那么这些 `WorldContexts` 又是保存在哪里的呢？追根溯源：  

![[b3c2ba26d2f5d1c20eba93180028298a_MD5.png]]

  
**GameInstance 里会保存着当前的 WorldConext 和其他整个游戏的信息**。明白了 GameInstance 是比 World 更高的层次之后，我们也就能明白为何那些独立于 Level 的逻辑或数据要在 GameInstance 中存储了。  

这一点其实也很好理解，大凡游戏引擎都会有一个 Game 的概念，不管是叫 Application 还是 Director，它都是玩家能直接接触到的最根源的操作类。而 UE 的 GameInstance 因为继承于 UObject，所以就拥有了动态创建的能力，所以我们可以通过指定 GameInstanceClass 来让 UE 创建使用我们自定义的 GameInstance 子类。所以不论是 C++ 还是 BP，我们通常会继承于 GameInstance，然后在里面编写应用于整个游戏范围的逻辑。  

**因为经常有初学者会问到：我的 Level 切换了，变量数据就丟了，我应该把那些数据放在哪？再清晰直白一点，GameInstance 就是你不管 Level 怎么切换，还是会一直存在的那个对象！**

- 游戏实例类的实例会在游戏开始时创建，并仅在游戏关闭时移除。
- 关卡中的所有 Actor 和其他对象会完全销毁，并在每次关卡加载时重新产生。
- 游戏实例类和它包含的数据在各个关卡之间保持不变。游戏实例类仅存在于每个客户端上，不进行复制。
- 要分配游戏中使用的游戏实例类，前往"编辑”(Edit)>“项目设置”(Project Settings)>“地图和模式”(MapsModes)修改项目设置。
![[Pasted image 20230115230553.png]]

## Engine

让我们继续再往上，终于得见 UE 大神：  

![[fe68f967adb22cfeea680d61a78da666_MD5.png]]

此处 UEngine 分化出了两个子类：UGameEngine 和 UEditorEngine。众所周知，UE 的编辑器也是 UE 用自己的引擎渲染出来的，采用的也是 Slate 那套 UI 框架。好处有很多，比如跨平台比较统一，UI 框架可以复用一套控件库，Dogfood 等等，此处不再细讲。**所以本质上来说，UE 的编辑器其实也是个游戏**！我们是在编辑器这个游戏里面创造我们自己的另一个游戏。话虽如此，但比较编辑器和游戏还是有一定差别的，所以 UE 会在不同模式下根据编译环境而采用不同的具体 Engine 类，而在基类 UEngine 里通过一个 WorldList 保存了所有的 World。

*   Standalone Game：会使用 UGameEngine 来创建出唯一的一个 GameWorld，因为也只有一个，所以为了方便起见，就直接保存了 GameInstance 指针。
*   而对于编辑器来说，EditorWorld 其实只是用来预览，所以并不拥有 OwningGameInstance，而 PlayWorld 里的 OwningGameInstance 才是间接保存了 GameInstance.

目前来说，因为 UE 还不支持同时运行多个 World（当前只能一个，但可以切换），所以 GameInstance 其实也是唯一的。提前说些题外话，虽然目前网络部分还没涉及到，但是当我们在 Editor 里进行 MultiplePlayer 的测试时，每一个 Player Window 里都是一个 World。如果是 DedicateServer 模式，那 DedicateServer 也会是一个 World。  
最后实例化出来的 UEngine 实例用一个全局的 `GEngine` 变量来保存。至此，我们已经到了引擎的最根处:

```c++
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

# 4 Pawn

## 引言

欢迎来到 GamePlay 架构章节的下半部分！  
在上一篇的内容里，我们谈到了 UE 的 3D 游戏世界是由 Object->Actor+Component->Level->World->WorldContext->GameInstance->Engine 来逐渐层层构建而成的。那么从这下半章节开始，我们就将要开始逐一分析，UE 是如何在每一个对象层次上表达游戏逻辑的。和分析对象节点树一样，我们也将采用自底向上的方法，从最原始简单的对象开始。

## Component

Actor 可以说是由 Component 组成的，所以 Component 其实是我们对象树里最底层的员工了。在 UE 里，Component 表达的是 “功能” 的概念。比如说你要实现一个可以响应的 WASD 移动的功能，或者是 VR 里抓取的功能，甚至是嵌套另一个 Actor 的功能，这些都是一个个组件。正确理解 “功能” 和“游戏业务逻辑”的区分是理解 Component 的关键要点。  
所以我们在这一个层级上要编写的逻辑，是实现一个个 “与特定游戏无关” 的功能。理想情况下，等你的一个游戏完成，你那些已经实现完成的 Components 是可以无痛迁移到下一个游戏中用的。换言之，一旦你发现你在 Component 中含有游戏的业务逻辑代码，这就是所谓的 “Bad Smell” 了，要警惕游戏架构是否恰当，是否没有很清晰的概念划分。

## Actor

如果说 UE 是一个大国家的话，那 Actor 无疑就是人口最大的民族了。StaticMeshActor，CameraActor…… 我们天天口里嚷嚷的也都是它。和 Unity 的 Prefab 对应的，在 UE 里我们用的最多的也是 BlueprintActor 了，我们也常常自定义我们的 Actor 子类来组装其他 Component 和 Actor，然后再编写一些协作逻辑代码，就似乎完成了一个骁勇善战的特种兵，接下来就可以撒豆成兵般的往 Level 中扔了。  
用的越广泛越多，往往错的也越多。似乎是受到了一种朴素的子承父业的精神感染，也或许是我们的面向对象编程都学得太好的缘故，我们都非常倾向于直接在 Actor 里堆砌逻辑。右键一个 BlueprintActor，刚添加完 Component，就立马撸起袖子来，Event、Function 和 Variable 一个个罗列开来，噼里啪啦无不快活！但是且慢，这是最好的方式了吗？让我们一路带着这个问题，试着从 UE 角度去推演一下，重走一下 Actor 进化之路。在本章节旅程的终点，我保证，我们可以比较清楚的回答这个问题。

其实所有的游戏引擎在构建完节点树之后，都会面临这么一个问题，**我的游戏逻辑写在哪里？**  

有的原始的如 Cocos2dx 懒得想那么多，干脆就直接码在 Node 里面得了，所以你翻看 Cocos2dx 的源码你就会经常发现它的逻辑和表现往往是交杂在一起的，简单直接暴力美学，面向对象继承玩得溜。而面向组合阵营的领军 Unity 则干脆就把 Component 思想再应用极致一点，我的逻辑为什么不能也是一个组件？所以 Unity 里的 ScriptComponent 也是这种组合思想的体现，模型统一架构优雅，MonoBehavior 立大功了！但是在一个 Component（ScriptComponent）里去操作管理其他的 Components，本身却其实并不是那么优雅，因为有些 Component 之上的协调管理的事务，从层次上来说，应该放在更高的一个概念上实现。

UE 在思考这个问题时，却是感觉有些理想主义，颇有些 C++ 的理念，力求不为你不需要的东西付代价，宁愿有时候折衷，也想保住最优性能。UE 的架构中也大量应用了各种继承，有些继承链也能拉得很长，同时一方面也吸纳了组合的优点，我们也能见到 UE 的源码中类的成员变量也是组合了好多其他对象。所以接下来的该介绍的就是 UE 综合应用这两种思想的设计产物。面向对象派生下来的 Pawn 和 Character，支持组合的 Controller 们。

## Pawn

那么第二个至关重要的的问题是，**哪些 Actor 需要附加逻辑？**  
在游戏中，我们之所以会觉得一个角色生动，是因为它会响应我们的交互，并给出恰当的反应。而我们所谓的游戏业务逻辑，实际上编写的就是该如何对玩家的输入提供反馈。同样，一个 Actor 想要变得 “生动”，就得有响应外部输入的能力，否则就只是自动运转麻木的机器人。但是在一个比较大型的 3D 游戏中，Actor 有千千万万，然后并不是所有的 Actor 都需要和玩家互动，得宠的能直接面圣和玩家互动的 Actor 也是比较少的。我们经常都只是操作我们的“角色”，让“角色” 和场景里的其他物体互动。比如 FPS 游戏里我们操作的主角或者是 FlappyBird 里的那只小鸟。所以从这一点上来看，UE 中 Actor 就立马又可以划分出一个类别了，这些 Actor 们可谓是玩家们的宠儿，它们是玩家们的亲卫兵，对，它的名字就是 Pawn!  

![[b353121689143ee5d97a32483adcae38_MD5.png]]

为了更好理解这个概念，让我们看一下用搜索引擎搜一下 Pawn 得到的图：  

![[86fe4bb1adf1647d7ea6ed66ef93c39f_MD5.jpg]]

**思考：为何 Actor 也能接受 Input 事件？**  
我上述的对 Pawn 的描述可能会让你觉得，似乎 Pawn 既然就是用来被玩家控制的，那么理所当然的我们应该在 Pawn 上同时实现对输入的接受。但我们会发现实际上 EnableInput 接口却是在 Actor 上的，同时 InputComponent 也是在 Actor 里面的，意味着实际上你也可以在 Actor 上绑定处理输入事件。官方的输入事件处理流程图也是表明了这一点：  

![[6cf2eb79e60f257cf046c90207b6586c_MD5.jpg]]

## DefaultPawn，SpectatorPawn，Character

让我一口气介绍下面这三位：  

![[fa0b3d4f5061a95f05a8546ff00ac86b_MD5.png]]

### DefaultPawn

因为我们每次想自己搞 Pawn 都得从 Pawn 派生过来，然后再一个个添加组件。UE 知道我们大家都很懒，所以提供了一个默认的 Pawn：DefaultPawn，默认带了一个 DefaultPawnMovementComponent、spherical CollisionComponent 和 StaticMeshComponent。也是上述 Pawn 阐述过的三件套，只不过都是默认套餐。

### SpectatorPawn

UE 的 FPS 做的太好了，就会有一些观众想要观战。观战的玩家们虽然也在当前地图里，但是我们并不需要真正的去表示它们，只要给他们一些摄像机 “漫游” 的能力。所以派生于 DefaultPawn 的 SpectatorPawn 提供了一个基本的 USpectatorPawnMovement（不带重力漫游），并关闭了 StaticMesh 的显示，碰撞也设置到了 “Spectator” 通道。

### Character

因为我们是人，所以在游戏中，代入的角色大部分也都是人。大部分游戏中都会有用到人形的角色，既然如此，UE 就为我们直接提供了一个人形的 Pawn 来让我们操纵。  

![[d5ed90b80b9781382025abc5deb75025_MD5.jpg]]

## 总结

本篇主要探讨了从 Actor 到 Pawn 的分化过程，请读者们也好好自己体会一下这一过程中 UE 的设计和思量。一个游戏引擎对 3D 游戏世界的抽象是建立在很多概念之上的，UE 的逻辑和实现也都是基于对这些概念的实现和封装。而如果读者你并不清晰理解这些概念，那么就很难正确的应用和组织游戏的逻辑各个部分。本系列教程一如开篇所说，并不会教你应用的各种技巧，而把重点放在讨论 UE 背后的各种概念，这些才是让我们的头脑保持清晰的关键之处。  
因为在下笔力有限，很遗憾，我们心心念念的 Controller 只好留待下篇了。我在谈 Pawn 的时候，因为 Pawn 和 Controller 是那么紧密的关联着，所以也不得不事先一再的剧透提到 Controller。但 Controller 作为 GamePlay 逻辑的最最重要的一个载体，可探讨的点也非常的多，所以留待下篇吧。

# 5 Controller
## 引言

如上文所述，UE 从 Actor 中分化了一些专门可供玩家 “控制” 的 Pawn，那我们这篇就专门来谈谈该怎么个控制法！  
所谓的控制，本质指的就是我们游戏的业务逻辑。比如说玩家按 A 键，角色自动找一个最近的敌人并攻击，这个自动寻找目标并攻击的逻辑过程，就是我们所谈的控制。  
Note1：重申一下，Controller 特别是 PlayerController，跟网络，AI 和 Input 的关系都非常的紧密，目前都暂且不讨论，留待各自模块章节再叙述。

## MVC
**MVC 模式是软件工程中常见的一种软件架构模式，该模式把软件系统（项目）分为三个基本部分：模型（Model）、视图（View）和控制器（Controller)**

设计模式的本质就是抽象变化。如果依照纯朴的 "程序 = 数据 + 算法" 的结构来看，再算上用于用户显示和输入的界面，那么就得到 “程序 = 数据 + 算法 + 显示”。这三大基本块（数据，算法，显示）构成了程序的三大变化，而如何把这三者“+” 到一起，用的就是我们的种种设计框架模式。  

典型的，对于游戏：
*   “显示” 指的是游戏的 UI，是屏幕上显示的 3D 画面，或是手柄上的输入和震动，也可以是 VR 头盔的镜片和定位，是与玩家直接交互的载体；
*   “数据” 指的是 Mesh，Material，Actor，Level 等各种元素组织起来的内存数据表示；
*   “算法” 可以是各种渲染算法，物理模拟，AI 寻路，本文咱们就先暂时特指游戏开发者们编写的游戏业务逻辑。

抽象这三个变化，并归纳关系，就是典型的 MVC 模式了：

![[aa98f2f1365ec787c09249b5f0bc7958_MD5.png]]

有些人可能会说 MVC 是 UI 专用的模式，如 IOS 的 MVC 或 WPF 的 MVVM，也或者说因为游戏的类型千差万别所以一个通用的框架并不能都适用，因此就有一点点想要 “返璞归真” 的意味，觉得游戏引擎只需要提供一个基本的渲染框架，其他的逻辑框架不需要设计复杂，开发者们可自行根据游戏类型再设计。这种观点有一定的道理，对于简单的游戏或 Demo，确实也还不到需要 “设计” 的地步；而对于复杂大型的游戏，需要的架构知识也确实远不是 MVC 这么简单。但缺点在于，说这话的人要嘛就已经是架构高手，各种设计模式信手拈来，早已经到了无招胜有招的地步；要嘛就是回避了问题，游戏也是软件，软件的固有复杂度摆在那里，总得需要个办法去解决，今天如果我们不是在探讨尝试用 MVC 模式去掌控它，也是在谈一个别的名字的模式。在我看来，一个好的游戏引擎，应该是能尽力的帮助用户，并减少麻烦。MVC 当然也有它的缺陷和不足，所以我们应该研究的是 UE 为何选择了 MVC，有什么优点缺点，我们怎么利用和规避，让 UE 的 Controller 们尽责的为我们服务，少造成麻烦。  

对于简单的游戏或者引擎来说，有时并不需要把这三者分的很清，如 Cocos2dx 就没有 Controller，它的 MVC 就是混杂在一起，因为代码量少所以也还算勉强能凑合；Unity 的 MonoBehavior 其实也相当于把 MC 放在了一起，用得方便的同时也得小心太顺手了出现组件之间互相网状引用一团乱麻的情况；UE 在这个问题上的思考就有些一脉相承，既然 Actor 们形形色色，我们之前也谈过甚至有 AInfo 这种书记官，那为何不让一些 Actor 专门来承载逻辑呢？于是，Actor 再度分化出 Controller。下面我们就来一一介绍 Actor 旗下 Controller 家族的指挥官们。

## AController

虽然我在之前已经一再的剧透过 AController 是继承自 AActor 的一个子类，但是为了更好理解思考 UE 里的 Controller 机制，请先把脑袋放空，也别去偷看 UE 里的源码，像张无忌一样暂时忘记 AController 这回事，**问自己一个问题：如果我想实现一种机制去控制游戏里的 Actor，该怎么设计？**  

巧妇难为无米之炊，咱们先来看看当前手上都有些什么：
1.  **UObject**，反射序列化等机制
2.  **UActorComponent**，功能的载体，一定程度的嵌套组装能力（SceneComponent）
3.  **AActor**，基础的游戏对象，Component 的容器
4.  **APawn**，分化出来的 AActor，物理表示和基本的移动能力，当前正翘首以待。
5.  **没了**，在控制 Actor 这个层级，我们还暂时不需要去考虑 Level 等更高层次的对象

针对 APawn，再想想我们希望达成的控制愿景，没事，你尽管放开想象的想，做不做得到咱们先放一边，但至少别在一开始就被想象力限制住了。“控制”本身虽然只是一段逻辑算法代码，但是它也需要有个载体去承载和运行，某种意义上来说也算得上是个实体。所以下面我们不妨就脑洞大开，以 “控制” 这个实体的视角口吻，讲讲 “我，作为一个——控制” 希望拥有哪一些本领：

1.  **能够和 Pawn 对应起来**，理想情况下，极端的灵活性应该是多对多。我希望我能同时控制多个 Pawn，当然，一个 Pawn 也可以被多个我的兄弟姐妹们一起控制。想想那些 RTS 游戏和多人协作游戏，你应该能明白我有时候需要协调调度 Pawn 们走个方阵，有时候也得多人合作才能操纵得了一台机甲。当然越灵活也往往意味着越容易出错，但总之我们需要一个和 Pawn 关联的机制。
2.  **多个控制实例**，在需要的时候，我不介意可以克隆出多个我来，比如一段逻辑 A，我们希望可以有多个实例在同时运行。就像行为树一样，可以有多个运行实例，彼此算法一样，但互不干扰。
3.  **可挂载释放**，我可以选择当前控制 PawnA，也可以选择之后把它释放掉不再控制让她自生自灭，然后再另寻新欢控制 PawnB，我必须拥有灵活的运行时增删控制 Pawn 的能力。
4.  **能够脱离 Pawn 存在**，我思故我在，就算当前没有任何 Pawn 控制，我也可以继续存在，这样我就可以延时动态的选择 Pawn 对象。有些 Pawn 值得我去等。
5.  **操纵 Pawn 生死的能力**，谁规定必须一定去控制世界当前存在的 Pawn 才行。当世界里没有 Pawn 可供我控制时，我希望可以自己造一个出来。你要说她是玩具、亦或傀儡也好，我不在乎。有时候我很羡慕暗黑里的沉沦魔巫师，身边总是围绕着一群沉沦魔，一个沉沦魔挂了，他可以紧接着再复活一个出来，这样永远都不会感动寂寞，你说多好？那索性再霸道一点吧，要是我这个控制实体不在了，我希望可以选择是否带 Pawn 们跟我一起走，没了我，她们都傻得让人心疼。当然如果有哪个 Pawn 能让我这个霸道总裁爱上，我也愿意陪她一起去死。
6.  **根据配置自动生成**，我（控制）虽然只是一段代码，但也不能无中生有，所以也得有个机制可以生成我这个控制实体，不过想来这应该是组织里更上层领导的事，但至少他应该知道怎么创建我出来。
7.  **事件响应**，游戏事件的一些控制关心的事件应该能够传到我这里，我可以酌情处理。同样，Pawn 也可以向我汇报，我会好好研究决定的，嗯。
8.  **持续的运行**，没事的时候，我喜欢听世界大钟的每一次 Tick，跟我的心跳同步起来，就仿佛真的活过来一样，可以自主的做一些我想做的事，这是我最自在的时候。
9.  **自身有状态**，你累了要休息，我也一样。我可以选择自身的状态，选择工作或者是休息，也可以选择今天是哪个 Pawn 和心情最配。
10.  **拥有一定的扩展继承组合能力**，一方面我希望我的家族开枝散叶繁荣昌盛，我的一身本领继承自我的父亲，而我也将有我的儿，大家各有天赋。另一方面，那些普通的 Actor 们都可以身背各个 Component，更高贵的我当然也想有。
11.  **保存数据状态**，听说金鱼的记忆只有 7 秒，可是我却想记住你一辈子。所以我希望我能拥有一些记忆，人的过去成就了现在，也将指引着未来。以前有一个人跟我说过，当你不能再拥有的时候，唯一能做的就是令自己不要忘记。
12.  **可在世界里移动**，我可以选择帐中千里之外遥控 Pawn，也可以选择附身在一个 Pawn 身上，这样我才能多角度无死角的观察我可爱的 Pawn 们，嘿嘿。
13.  **可探查世界的对象**，我要有眼睛，让我干活，基本的我得看得见知道当前世界里已经有哪些对象吧，否则不就抓瞎了嘛。
14.  **可同步**，这年头，要是不能适应网络环境，可真的没有竞争力。这个 Object，Actor 基本都有的能力，我当然也得有。位于服务器或客户端上的我也必须有能力在其他客户端上影分身，让他们都跟随我的步伐一致行动。

在仔细考察了 "控制" 的需求和手头上的原料之后，我们试着从 UE 的角度来权衡一下。  
首先 Controller 不能是一个 Component，一是因为 Component 的层级太低，表达的是功能的概念而非逻辑；二是 Component 必须依附于 Actor 存在，而我们的 Controller 希望能独立存在。  
其次如果从 UObject 直接继承下来 UController，倒是也可行，UObject 也能复制同步，其他的控制 Pawn 的能力和事件响应等倒也是能改改接口想想办法，但是要想在世界里移动，就得有个位置表示，再加上还希望能容纳 Components，这就麻烦了，基本就是把 Actor 的工作再做一套，有点累人，搞起来也怕两套班子出错闹矛盾。  
再来考察下从 AActor 继承下来 AController 怎么样，Actor 比 Object 多了一些我们正需要的配置动态生成、输入事件响应、Tick、可继承、可容纳 Component、可在世界里出现、可在网络间同步。好了，现在就差控制 Pawn 的能力，那我们就在这个分化出来的 AController 增加一些控制 Pawn 的接口，这个思路正是和我们从 Actor 从分化出 Pawn 的时候不谋而合！现在我们来看看 UE 里的 AController:  

![[65338ebec11c6c740455b572516d8eb6_MD5.png]]

跟我们的设计八九不离十，我们再一一仔细验证一番：  
- 关联 Pawn 的能力，有 Possess 和 UnPossess，源码里也有 PawnPendingDestroy 等这些函数（未一一列出）；
- GameMode 中也保存着 AIControllerClass 和 PlayerControllerClass 的配置，用于在适当的时候 Spawn 出 Controller；
- 继承于 Actor 也就有了 EnableInput 和 Tick；
- Controller 本身还可以继续派生下去（如 AIController 和 PlayerController），也可以容纳 Components；
- 也带着一个 SceneComponent 所以可以摆放在世界中；
- 自身也可以添加成员变量来记忆存储游戏状态；
- 自身也有一个 FName StateName（Playing、Spectating、Inactive），切换自身的状态（运行，观察，非激活）；
- 因为跟 Pawn 是平级的关系，只在运行的时候引用关联，所以对彼此独立存在不做强制约束，提高了灵活性。一个 Pawn 自身上也可以配置策略：

```c++
namespace EAutoReceiveInput
{
    enum Type
    {
        Disabled,
        Player0,
        Player1,
        Player2,
        Player3,
        Player4,
        Player5,
        Player6,
        Player7,
    };
}
TEnumAsByte<EAutoReceiveInput::Type> AutoPossessPlayer;
enum class EAutoPossessAI : uint8
{
    /** Feature is disabled (do not automatically possess AI). */
    Disabled,
    /** Only possess by an AI Controller if Pawn is placed in the world. */
    PlacedInWorld,
    /** Only possess by an AI Controller if Pawn is spawned after the world has loaded. */
    Spawned,
    /** Pawn is automatically possessed by an AI Controller whenever it is created. */
    PlacedInWorldOrSpawned,
};
EAutoPossessAI AutoPossessAI;
TSubclassOf<AController> AIControllerClass;
```

这样在运行时 UE 也可以根据 Pawn 创建配套的 Controller。毕竟只是为了阐明概念，而不是纠结技术细节，我对 Controller 的功能接口都只是粗略带过，如果读者自己去看 Contoller 的 UE 源码，顺便可以对我当前说的概念验证一下，还会发现一些 Movement 和 ViewPoint 的接口，这些也算是和控制移动和视角配套吧。  

>[!question] 
> **思考：Controller 和 Pawn 必须 1:1 吗？**  

观察 UE 实现里我们发现 Controller 里只是保存了一个 Pawn 指针，而不是数组，这和一开始希望的多对多关系有些出入。理想和现实总是有差距，一个愿景落实到工程实践上也不免得有一些妥协。首先我们再来梳理理解一下这个 Possess(拥有 / 占用) 的概念。一个 Controller 能灵活的 Possess/UnPossess 一个 Pawn，虽然一次只能控制一个，但在游戏中我们也可以在不同的 Pawn 中切换，比如操纵一个主角坐进然后控制一辆汽车，或者端起固定的机关枪扫射，这些功能琢磨一下其实只是涉及操作实体 Pawn 的变化。如果我们能妥善的用好 Pawn 和 Controller 的切换功能，大部分基本的游戏功能也是能够比较方便的实现的。那么有哪些是不太适合的呢？UE 官方其实也承认了，见 [Controller](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/Controller/index.html) 文档说明：

>默认情况下，控制器（Controller）和角色（Pawn）之间存在一对一的关系；也就是说，每个控制器在任何给定时间只能控制一个角色。这对于大多数类型的游戏来说是可以接受的，但对于某些类型的游戏，比如实时策略游戏，可能需要调整，以便同时控制多个实体。

对于 RTS 这种需要一下子控制多个单位的游戏来说，这种 1v1 的关系确实比较僵硬，就需要在 Controller 里自己实现扩展一下，额外保存多个 Pawn，然后自己实现一些需要的控制实现，但总体上也只能说得绕一下，也算不上特别复杂，所以就也不能说 UE 做不了某一些类型的游戏，Epic 是个游戏引擎公司，卖的毕竟是个通用游戏引擎。  

**OK，那 UE 为何不实现成多对多呢？我觉得理由往往很简单，就是想保持一定的简单**。游戏引擎的每个模块的设计，甚至函数接口的设计，无时无刻不在权衡决定。太简单了概念清晰用起来方便但是灵活扩展力不足，太灵活扩展无限了往往也会让人无从适从容易出错。当前 1:1 的时候，我们的脑袋逻辑很清晰，我们可以在 Controller 里直接 GetPawn，也可以在 Pawn 中 GetController，都非常方便。调试逻辑 Bug 的时候，我们也能很快找到查错的目标。而对比想象，如果是 M：N，灵活性是满满了，但是你能轻易的说出当前 Pawn 是被哪些 Controller 控制吗？你也得时时记着这个 Controller 当前控制了哪些 Pawn。OMG！这些 Pawn 和 Controller 多对多的构成了网状结构，项目越庞大复杂，这张网也越能套住你。再从另一个方面说，一旦提供了这种多对多的直接支持，以我们人类的性格，免费现成的东西，我们总是倾向于去找机会能用上它，而不是去琢磨到底应不应该用。所以一旦就这么直接提供了，对于刚入门的新手，压根就没什么指引，怎么来好像都可以，就非常容易收不住把项目逻辑关系搞得不必要的复杂。所以以后 UE 就算想在这一方面优化加强，应该也会比较克制。  

索性再聊开一些，我们用 Unity 来做一下对比。Unity 就是 GameObject+Component，你自己组合去吧，非常的灵活自由，也不做什么限制，但造成的后果就是常常各种 Component 互相引用来引用去，网状互联一团乱麻。另外几乎每个人都可以在上面搞出一套游戏系统出来，互相之间都是自成一派。所以经常网上就会有各种帖子问怎么在 Unity 中实现 MVC 模式的，也有分析炉石传说游戏逻辑框架的。Unity 当然是个好引擎，目前来说热度也是比 UE 要高一些，但我们也不能因为它火用得人多，就权威崇拜从众的认为 Unity 各个方面都比别的引擎好。设计架构游戏的时候，工程师们要抵挡住灵活性的诱惑，保持克制往往是更难得珍贵的美德。要认识到，引擎的终极目的是方便人使用的，我们程序员往往很容易太沉迷于程序功能的灵活强大，而疏忽了易用性鲁棒性等社会工程需求。

> [!question] 
> **思考：为何 Controller 不能像 Actor 层级嵌套？**  

我们都知道 Actor 可以藉着身上的 SceneComponent 互相嵌套。那么 AController 同样也是 Actor，为何不也实现这么一个父子机制？从功能上来说，一个 Controller 可以有子 Controllers，听起来也是非常灵活强大啊。但是冷静想一下，Controller 表达的 “控制” 的概念，所以在这里你实际上想要表达的是一种 “控制” 互相嵌套的概念，感觉又给 “控制” 给分了层，有 “大控制”，也有“小控制”，但是“控制” 的“大小”又是个什么概念呢？我们应该怎么划分控制的大小？“控制”本质上来说就是一些代码，不管怎么设计，目的都是用来表达游戏游戏逻辑的。而针对游戏逻辑的复杂，怎么更好的管理组织逻辑代码，我们有状态机，分层状态机，行为树，GOAL（目标导向），甚至你还能搞些神经网络遗传算法机器学习啥的。所以在我们已经有这么多工具的基础上，徒增复杂性是很危险的做法。如果有必要，也可以把 Controller 本身再当作其他 AI 算法的容器，所以就没必要在对象层次上再做文章了。

> [!question] 
> **思考：Controller 可以显示吗？**  

既然 Actor 本身可以带着 Mesh 组件来渲染显示，那 Controller 可不可以呢？是不是 Controller 都是不可见的？这个答案可说是也可以说不是，因为 Controller 本身确实就是一个特殊点的 Actor 而已，你依然可以在 Controller 中添加 Mesh 组件，添加别的子 Actor 等，所以从这个方面说 Controller 是有可以渲染显示的能力的。但是一个控制者毕竟只是表达一个逻辑的概念，所以为了分工明确，UE 就干脆在 Controller 的构造函数里把自己给隐藏了：

```c++
bHidden = true;
#if WITH_EDITORONLY_DATA
    bHiddenEd = true;
#endif // WITH_EDITORONLY_DATA
```

事了拂衣去，深藏功与名。为了验证我的说法，读者你可以亲自在 PlayController 下挂一些 Cube 之类的 Actor，然后在源码层把这两个值改为 false，重新编译运行看下结果，看能否正确显示出来，这里我就不贴图了，留给读者验证，很好玩的哦。  

> [!question] 
> **思考：Controller 的位置有什么意义？**  

既然 Controller 本身只是控制者，那它在场景中的位置和移动有什么意义吗？Controller 为何还需要个 SceneComponent? **意义在于如果 Controller 本身有位置信息，就可以利用该信息更好的控制 Pawn 的位置和移动。**  
首先说下 Controller 的 Rotation，这个比较好理解一点，如果我想让我的 Pawn 和 Controller 保持旋转朝向一致，因为是 Controller 作主控制 Pawn 的关系，所以 Controller 就得维护自己的 Rotation。再来说位置，如果 Controller 有自己的位置，这样在 Respawn 重新生成 Pawn 的时候，你就可以选择在当前位置创建。**因此为了自动更新 Controller 的位置，UE 还提供了一个 `bAttachToPawn` 的开关选项，默认是关闭的，UE 不会自动的更新 Controller 的位置信息；而如果打开，就会把 Controller 附加到 Pawn 的子节点里面去，让 Controller 跟随 Pawn 来移动**。你可以把这两种模式想象成一种是上帝视角在千里之外心电感应控制 Pawn，另一种是骑在 Pawn 肩上来指挥。  
当然如果这个 Controller 确实只是纯朴的逻辑控制的话（如 AIController），那确实位置也没什么意义。所以 UE 甚至还隐藏了 Controller 的一些更新位置的接口，尽量避免让人手动去操纵：

```c++
private:
    // Hidden functions that don't make sense to use on this class.
    HIDE_ACTOR_TRANSFORM_FUNCTIONS();
//展开后：
//////////////////////////////////////////////////////////////////////////
// Macro to hide common Transform functions in native code for classes where they don't make sense.
// Note that this doesn't prevent access through function calls from parent classes (ie an AActor*), but
// does prevent use in the class that hides them and any derived child classes.
#define HIDE_ACTOR_TRANSFORM_FUNCTIONS() private: \
    FTransform GetTransform() const { return Super::GetTransform(); } \
    FVector GetActorLocation() const { return Super::GetActorLocation(); } \
    FRotator GetActorRotation() const { return Super::GetActorRotation(); } \
    FQuat GetActorQuat() const { return Super::GetActorQuat(); } \
    FVector GetActorScale() const { return Super::GetActorScale(); } \
    bool SetActorLocation(const FVector& NewLocation, bool bSweep=false, FHitResult* OutSweepHitResult=nullptr) { return Super::SetActorLocation(NewLocation, bSweep, OutSweepHitResult); } \
    bool SetActorRotation(FRotator NewRotation) { return Super::SetActorRotation(NewRotation); } \
    bool SetActorRotation(const FQuat& NewRotation) { return Super::SetActorRotation(NewRotation); } \
    bool SetActorLocationAndRotation(FVector NewLocation, FRotator NewRotation, bool bSweep=false, FHitResult* OutSweepHitResult=nullptr) { return Super::SetActorLocationAndRotation(NewLocation, NewRotation, bSweep, OutSweepHitResult); } \
    bool SetActorLocationAndRotation(FVector NewLocation, const FQuat& NewRotation, bool bSweep=false, FHitResult* OutSweepHitResult=nullptr) { return Super::SetActorLocationAndRotation(NewLocation, NewRotation, bSweep, OutSweepHitResult); } \
    virtual bool TeleportTo( const FVector& DestLocation, const FRotator& DestRotation, bool bIsATest, bool bNoCheck ) override { return Super::TeleportTo(DestLocation, DestRotation, bIsATest, bNoCheck); } \
    virtual FVector GetVelocity() const override { return Super::GetVelocity(); } \
    float GetHorizontalDistanceTo(AActor* OtherActor)  { return Super::GetHorizontalDistanceTo(OtherActor); } \
    float GetVerticalDistanceTo(AActor* OtherActor)  { return Super::GetVerticalDistanceTo(OtherActor); } \
    float GetDotProductTo(AActor* OtherActor) { return Super::GetDotProductTo(OtherActor); } \
    float GetHorizontalDotProductTo(AActor* OtherActor) { return Super::GetHorizontalDotProductTo(OtherActor); } \
    float GetDistanceTo(AActor* OtherActor) { return Super::GetDistanceTo(OtherActor); }
```

UE 这里其实想说的是，这些更新位置的操作还是让我来为你管理吧，我真的担心你会用错搞出什么乱子来。顺便再说些题外话，对于 PlayerController 来说，因为玩家需要一个视角来观察世界，所以常常 PlayerController 常常会扛着个摄像机出现（蓝图里没有，但是会运行时生成 PlayerCameraManager 和 CameraActor），所以就算没有角色可供操作，玩家也依然希望是可以视角漫游观察整个世界的（试试看把默认 Level 里的 PlayerStart 删掉后运行看看）。这个时候 PlayerController 常常会默认创建出一个 ASpectatorPawn 或者 DefaultPawn（根据 GameMode 里配置），我们虽然看不见 Pawn，但依然可以观察世界，靠得就是跟 Controller 关联的旋转和摄像机。  

> [!question] 
> **思考：哪些逻辑应该写在 Controller 中？**  

如同当初我们在思考 Actor 和 Component 的逻辑划分一样，我们也得要划分哪些逻辑应该放在 Pawn 中，哪些应该放在 Contrller 中。上文我们也说过，Pawn 也可以接收用户输入事件，所以其实只要你愿意，你甚至可以脱离 Controller 做一个特立独行的 Pawn。那么在那些时候需要 Controller？哪些逻辑应该由 Controller 掌管呢？可以从以下一些方面考虑：

*   从概念上，Pawn 本身表示的是一个 “能动” 的概念，重点在于 “能”。而 Controller 代表的是动到“哪里” 的概念，重点在于“方向”。**所以如果是一些 Pawn 本身固有的能力逻辑，如前进后退、播放动画、碰撞检测之类的就完全可以在 Pawn 内实现；而对于一些可替换的逻辑，或者智能决策的，就应该归 Controller 管辖。**
*   从对应上来说，如**果一个逻辑只属于某一类 Pawn，那么其实你放进 Pawn 内也挺好。而如果一个逻辑可以应用于多个 Pawn，那么放进 Controller 就可以组合应用了**。举个例子，在战争游戏中，假设说有坦克和卡车两种战车（Pawn），只有坦克可以开炮，那么开炮这个功能你就可以直接实现在坦克 Pawn 上。而这两辆战车都有的自动寻找攻击目标功能，就可以实现在一个 Controller 里。
*   从存在性来说，Controller 的生命期比 Pawn 要长一些，比如我们经常会实现的游戏中玩家死亡后复活的功能。Pawn 死亡后，这个 Pawn 就被 Destroy 了，就算之后再 Respawn 创建出来一个新的，但是 Pawn 身上保存的变量状态都已经被重置了。**所以对于那些需要在 Pawn 之外还要持续存在的逻辑和状态，放进 Controller 中是更好的选择。**

## APlayerState
- 玩家状态类用于记录特定玩家的信息，这些信息在多人游戏中需要与其他客户端共享。
- 玩家控制器仅存在于客户端上，而玩家状态会从服务器复制到所有客户端。
- APlayerState 存数据，APlayerController 存逻辑方法

我们上文提到过 Controller 希望也能有一些记忆，保存住一些游戏状态。那么到底应该怎么保存呢？**AController 自身当然可以添加成员变量来保存，这些变量也可以网络复制，一般来说也够用。但是终究还是遗忘了一个最重要的数据——状态。** 整个游戏世界构建起来就是为了玩家服务的，而玩家在游戏过程中，肯定要存取产生一些状态。而 Controller 作为游戏业务逻辑最重要的载体，势必要和玩家的状态打交道。**所以 Controller 如果可以动态存取玩家的状态就会大为方便了**。因此我们会在 Controller 中见到：

```c++
/** PlayerState containing replicated information about the player using this controller (only exists for players, not NPCs). */
UPROPERTY(replicatedUsing=OnRep_PlayerState, BlueprintReadOnly, Category="Controller")
class APlayerState* PlayerState;
```

而 APlayerState 的继承体系是：  

![[9e9af6cb3406c49ece9b425c4996a5c7_MD5.png]]

至于为啥 APlayerState 是从 AActor 派生的 AInfo 继承下来的，我们聪明的读者相信也能猜得到了，所以也就不费口舌论证了。**无非就是贪图 AActor 本身的那些特性以网络复制等**。而 AInfo 们正是这种不爱表现的纯数据书呆子们的大本营。而这个 **PlayerState 我们可以通过在 GameMode 中配置的 PlayerStateClass 来自动生成。**  
注意，这个 APlayerState 也理所当然是生成在 Level 中的，跟 Pawn 和 Controller 是平级的关系，**Controller 里只不过保存了一个 APlayerState 的指针引用罢了**。注释里说的 PlayerState 只为 players 存在，不为 NPC 生成，指的是 PlayerState 是跟 UPlayer 对应的，换句话说当前游戏有多少个真正的玩家，才会有多少个 PlayerState，而那些 AI 控制的 NPC 因为不是真正的玩家，所以也不需要创建生成 PlayerState。但是 UE 把 PlayerState 的引用变量放在了 Controller 一级，而不是 PlayerController 之中，说明了其实 AIController 也是可以设置读取该变量的。一个 AI 智能能够读取玩家的比分等状态，有了更多的信息来作决策，想来也没有什么不对嘛。  
Controller 和网络的结合很紧密，很多机制和网络也非常强关联，但是在这里并不详细叙述，这里先可以单纯理解成 Controller 也可以当作玩家在服务器上的代理对象。把 PlayerState 独立构成一个 Actor 还有一个好处，当玩家偶尔因网络波动断线，因为这个连接不在了，所以该 Controller 也失效了被释放了，服务器可以把对应的该 PlayerState 先暂存起来，等玩家再紧接着重连上了，可以利用该 PlayerState 重新挂接上 Controller，以此提供一个比较顺畅无缝的体验。至于 AIController，因为都是运行在 Server 上的，Client 上并没有，所以也就无所谓了。

> [!question] 
> **思考：哪些数据应该放在 PlayerState 中？**  

从应用范围上来说，PlayerState 表示的是**玩家的游玩数据**（攻击力，血量等）

**不应该放在 PlayerState 中的数据：**
- 关卡内的其他游戏数据（放在GameState 是个好选择），
- Controller 本身运行需要的临时数据。
- 跨关卡的统计数据（玩家在切换关卡的时候，APlayerState 也会被释放掉，所有 PlayerState 实际上表达的是当前关卡的玩家得分等数据），应该放在外面的 GameInstance，然后用 SaveGame 保存起来。

## 总结

在游戏里，如果要评劳模，那 Controller 们无疑是最兢兢业业的，虽然有时候蛮横霸道了一些，但是经常工作在第一线，下面的 Pawn 们常常智商太低，上面的 Level，GameMode 们又有点高高在上，让他们直接管理数量繁多的 Pawn 们又有点太折腾，于是事无巨细的真正干那些脏活累活的还得靠 Controller 们。本文虽然没有在网络一块留太多笔墨，但是 Controller 也是同时作为联机环境中最重要的沟通渠道，身兼要职。  
回顾总结一下本文要点，UE 在 Pawn 这个层级演化构成了一个最基本和非常完善的 Component-Actor-Pawn-Controller 的结构：

![[49911dbb6255e84669230b4fba2d521e_MD5.png]]

通过分化出来后的 Actor 的互相控制，既充分利用了现有的机制功能，又提供了足够的灵活性，而且做的更改还很少，不用再设计额外另一套框架。

本想着一篇介绍完 Controller、PlayerController 和 AIController 这三个对象，但是 Controller 本身是 UE 里极为重要的核心概念，自身的功能非常的丰富，牵扯的模块也比较多，因此想抽离阐述最核心的概念和功能并不是一件容易的事。

花了这么长的篇幅，只讨论揣摩了 Controller 的设计过程和最基本的职责（还有输入网络等都没有解释），顺便先简单介绍了下 PlayerState 出场（PlayerState 实际上是跟 UPlayer 关联更大一些，PlayerController 等后续章节会继续讨论它），对于 PlayerController 和 AIController，目前也只是语焉不详的含糊带过。不过还是希望读者们能从中吸取到设计的营养，把握清楚概念了，才能更好的组织游戏逻辑，开发出更好的游戏。

# 6 PlayerController 和 AIController

## 引言

上文我们谈到了 Component-Actor-Pawn-Controller 的结构，追溯了 AController 整个家族的崛起和身负的使命。本篇我们继续来探讨 Controller 家族中最为人所知的 PlayerController 和 AIController。  

作为一个 Controller，我们讨论的依然是该如何控制。我们已经知道了 Controller 可以 Possess 并控制 Pawn，但是 Controller 本身又是怎么驱动起来的呢？一个游戏里的控制角色大抵都可以分为两类：玩家和 AI。不管是单机游戏或者分屏多玩家，还是网络玩家联机对战，游戏都是为了玩家服务的，所以也必然会有一个或多个玩家，就算是如《山》那种纯看的游戏，也是有一个 “可观察不可动” 的玩家的。而 AI 的实体的数量就可以是零或者多个。  
**Note1：** 依旧重申：输入、网络、AI 行为树等模块虽跟 PlayerController 和 AIController 关系紧密，但目前都暂且不讨论，留待各自模块章节再叙述。

## APlayerController

让咱们先从简单的单机游戏开始讨论吧，比如一款单机 FPS 游戏，这个游戏里已经用各种各样的 Actor 们构建完成了世界场景，你的主角和敌人 Pawn 们也都在整装待发，这个时候你思考这么一个问题，我该怎么玩这个游戏？壮丽的舞台已经准备好了，就等你入场了。先抛开具体的引擎而言，首先你需要能**看见**（拥有 Camera 和位置），其次你必须能**响应输入**（玩家按 WASD 你应该能接收到），然后你可以根据输入操控一些 Pawn（**Possess 然后传递 Input**），这样一个单机游戏中的简单玩家控制器就差不多了。一个游戏中只有一个 PlayerController，在不同的关卡中你可以使用不同的 PlayerController，但是同一时刻响应的只能是一个 PlayerController。  

插上多个手柄，咱们再拓展一下，比如像《街霸》那种单 PC 但是多玩家对抗或者协作的游戏。两个玩家可以分别用两个手柄，或者一个用键盘一个用鼠标，甚至是键盘上的不同区域，形式可以多种多样。这个时候如果依然只有一个 PlayerController，实现起来其实也是可行的，把两个手柄——所有的输入都由这个 PlayerController 来接收，然后在 PlayerController 内部再分别根据情况去处理不同的 Pawn。但是这种方式的缺点显然也在于很容易把玩家 1、2 的输入和控制混杂在一起，没有清晰的区分开。因此，为了支持这种情况，我们可以开始允许游戏中同时出现多个 PlayerController，每个 PlayerController 甚至都可以拥有自己的 Viewport（分屏或者不同窗口），这样我们通过配置，可以精确的路由手柄 1 的输入给玩家 1，各自的逻辑也很好的区分和复用。  
再插上网线继续，到了网游时代，我们的游戏就开始允许有多人联机对战了。玩家在自己的 PC 上控制的只是自己的本地的角色，而屏幕游戏里其他的玩家角色是由网线另一端的玩家控制的。为了更好的适应这种情况，我们就又得扩展一下 PlayerController 的概念，PlayerController 不仅能控制本地的 Pawn，而且还能 “控制” 远程的 Pawn（实际上是通过 Server 上的 PlayerController 控制 Server 上的 Pawn，然后再复制到远程机器上的 Pawn 实现的）。  
因此我们来看看 UE 里的 PlayerController：  

![[c8b2380d53809dfb6360496f8d954ce3_MD5.png]]

PlayerController 因为是直接跟玩家打交道的逻辑类，因此是 UE 里使用最多的类之一。UE4.13.2 版本里 1632 行的. h 文件和 4686 行的. cpp 文件，里面实现了很多的功能，初阅读起来往往深陷其中不得要领。但是在上述的分析了之后，我们也可以在其中大概归纳出几个模块：

*   **Camera 的管理**，目的都是为了控制玩家的视角，所以有了 `PlayerCameraManager` 这一个关联很紧密的摄像机管理类，用来方便的切换摄像机。PlayerController 的 ControlRotation、ViewTarget 等也都是为了更新 Camera 的位置。因为跟 Camera 的关系紧密，而 Camera 最后输出的是屏幕坐标里的图像，所以为了方便一些拾取的 HitResult 函数也都是实现在这里面。渲染章节会再详细介绍 UE 的摄像机管理。
*   **Input 系统**，包括构建 InputStack 用来路由输入事件，也包括了自己对输入事件的处理。所以包含了 UPlayerInput 来委托处理。
*   **UPlayer 关联**，既然顾名思义是 PlayerController，那自然要和 Player 对应起来，这也是 PlayerController 最核心的部分。一个 UPlayer 可以是本地的 LocalPlayer，也可以是一个网络控制 UNetConnection。PlayerController 只有在 SetPlayer 之后，才可以开始正常工作。
*   **HUD 显示**，用于在当前控制器的摄像机面前一直显示一些 UI，这是从 UE3 迁移过来的组件，现在用 UMG 的比较多，等介绍 UI 模块的时候再详细介绍。
*   **Level 的切换**，PlayerController 作为网络里通道，在一起进行 Level Travelling 的时候，也都是先通过 PlayerController 来进行 RPC 调用，然后由 PlayerController 来转发到自己 World 中来实际进行。
*   **Voice**，也是为了方便网络中语音聊天的一些控制函数。

简单来说，PlayerController 作为玩家直接控制的实体，很多的跟玩家直接相关的操作也都得委托它来完成。目前来说 PlayerController 里旗下的 100 + 的函数也大概可以分为以上几大模块，也根据需要重载了 Controller 里的一些其他函数。  
UE 的思想是具象化一个 “玩家实体”，并把所有的跟该玩家相关的操作和接口都交给它完成。一般其他的游戏引擎只是个 “功能引擎”，提供了一些图形渲染 UI 系统等组件，但是在 GamePlay 这个层次就都非常欠缺了，一般都需要开发者自己搭建一套。而回想你写过的游戏，是不是也往往有一个 Player 类（一般是单件或者全局变量）？里面几乎是放着所有跟该玩家相关的业务逻辑代码。UE 里的 PlayerController 就是这种概念，优点当然是直接方便好理解，缺点也如你所见，会代码膨胀得比较快。不过目前来说还算能接受，等某一块功能真的比较大了之后，可以再把它抽出一个单独的类来，如 PlayerInput 和 PlayerCameraManager 一样。

> [!question] 
> **思考：哪些逻辑应该放在 PlayerController 中？**  

回想我们上篇的问题：“哪些逻辑应该写在 Controller 中？”，该处的答案观点在本处也依然适用。不过我还想再补充几点：

*   对实现游戏逻辑来说，如果是按照 MVC 的视角，那么 View 对应的是 Pawn 的表现，而 PlayerController 对应的是 Controller 的部分，那 Model 就是游戏业务逻辑的数据了。拿超级马里奥游戏来举例子，把问题先局限在一个关卡内，假设要实现的是金币的逻辑，那么 View 指的是游戏右上角的金币数目 UI，而玩家用 PlayerController 来控制马里奥来蹦跳行走，而马里奥（Pawn）通过触碰金币的事件又上报给 PlayerController 来相应增加金币。而 PlayerController 存储金币的数据就是在 PlayerState 中。即 PlayerState 中有一个 int coin，也有相应的 AddCoin(int coin)。**而 PlayerController 的职责应该是一边控制 Pawn，一边负责内部正确的调用 PlayerState 的 Coin 接口**。那么 PlayerController 里的成员变量有什么用？根据单一职责原则，我们写在哪个类里的变量应该尽量只符合该类的作用，**所以 PlayerController 里的变量的意义在于更好的实现控制**。比如假设玩家在一个关卡内可以按 AABB 来作弊获得 100 金币，但是限最多 3 次。那么这个按键的响应就应该由 PlayerController 来接收，然后调用 AddCoin(100)，并更新 PlayerController 里的成员变量 CoinCheatCount。也或者想实现马里奥的加速跑，也可以在 PlayerController 里增加 Speed 的成员变量。
*   记住 PlayerController 是可被替换的，不同的关卡里也可能是不一样的。比如马里奥在水下的时候控制的方式明显就不一样，所以就不能像 “Player” 单件类那样什么都往里面塞。这样一旦被替换掉了之后数据就都丢失了。
*   PlayerController 也不一定存在，考虑一下如果把马里奥做成联机游戏，那么对方玩家被同步过来的将只有 PlayerState，对方玩家的 PlayerController 只在服务器上存在。所以这个时候，如果你把金币数据放在 PlayerController 里的话就非常尴尬了。所以为了扩展性来说，还是根据职责分明的原则来正确划分业务逻辑会比较好。
*   在任一刻，Player:PlayerController:PlayerState 是 1:1:1 的关系。但是 PlayerController 可以有多个备选用来切换，PlayerState 也可以相应多个切换。**UPlayer 的概念会在之后讲解，但目前可以简单理解为游戏里一个全局的玩家逻辑实体，而 PlayerController 代表的就是玩家的意志，PlayerState 代表的是玩家的状态。**

## AAIController

从某种程度上来说，AI 也可以算是一个 Player，只不过它不需要接收玩家的控制，可以自行决策行动。从玩家控制的逻辑需要有一个载体一样，AI 的逻辑算法也需要有一个运行的实体。而这就是 UE 里的 AIController：  

![[e180a5fc180032be9200afc3d0967915_MD5.png]]

同 PlayerController 对比，少了 Camera、Input、UPlayer 关联，HUD 显示，Voice、Level 切换接口，但也增加了一些 AI 需要的组件：

*   **Navigation**，用于智能根据导航寻路，其中我们常用的 MoveTo 接口就是做这件事情的。而在移动的过程中，因为少了玩家控制的来转向，所以多了一个 SetFocus 来控制当前的 Pawn 视角朝向哪个位置。
*   **AI 组件**，运行启动行为树，使用黑板数据，探索周围环境，以后如果有别的 AI 算法方法实现成组件，也应该在本组件内组合启动。
*   **Task 系统**，让 AI 去完成一些任务，也是实现 GameplayAbilities 系统的一个接口。目前简单来说 GameplayAbilities 是为 Actor 添加额外能力属性集合的一个模块，比如 HP，MP 等。其中的 GamePlayEffect 也是用来实现 Buffer 的工具。另外 GamePlayTags 也是用来给 Actor 添加标签标记来表明状态的一种机制。目前来说该两个模块似乎都是由 Epic 的 Game Team 在维护，所以完成度不是非常的高，用的时候也往往需要根据自己情况去重构调整。

本文重点不在于讨论 AI 内部的各种组件功能，因此我们先把目光聚焦在 AIController 对象本身上。**同 PlayerController 一样，AIController 也只存在于 Server 上（单机游戏也可看作是 Server）**。游戏里必须有玩家参与，而 AI 可以没有，所以 AIController 并不一定会存在。我们可以在 Pawn 上配置 AIControllerClass 来让该 Pawn 产生的时候自动为它分配一个 AIController，之后自动释放。

> [!question] 
> **思考：哪些逻辑应该放在 AIController 中？**  

我们依然要思考这个问题，大部分思想和原则和 PlayerController 是一样的，只不过 AI 算法的多种多样，所以我们推荐尽量利用 UE 提供的行为树黑板等组件实现，而不是直接在 AIController 硬编码再度实现。也请把目光仅仅局限在当前的 Pawn 身上，不要在里面写其他无关的逻辑。另外，因为 AIController 都是在关卡内比较短暂存在的，一般不太有跨 Level 的数据保存，所以你可以用 AIController 的成员变量来保存状态。而如果真的需要用到 PlayerController 的状态，则也可以引用一个 PlayerState 过来。如果想引用关卡的全局状态，也可以引用 GameState，再更高级别的，甚至可以直接和 GameInstance 接触。  
但是 AIController 也可以通过配置 `bWantsPlayerState` 来获得自己的 PlayerState，所以 PlayerState 其实也并不是跟 UPlayer 绑定的，毕竟从本质上来说 APlayerState 也只是个 AInfo（AActor），跟其他 Actor 一样可以有多个，并没有什么稀奇的，区别是你自己怎么创建并利用它。

## 总结

到此，我们也算讨论完了 Actor（Pawn）层次的控制，在这个层次上，我们关注的焦点在于如何更好的控制游戏世界里各种 Actor 交互和逻辑。UE 采用了分化 Actor 的思维创建出 AController 来控制 APawn 们，因为玩家玩游戏也全都是控制着游戏里的一个化身来行动，所以 UE 抽象总结分化了一个 APlayerController 来上接 Player 的输入，下承 Pawn 的控制。对于那些自治的 AI 实体，UE 给予了同样的尊重，创建出 AIController，包含了一些方便的 AI 组件来实现游戏逻辑。并利用 PlayerState 来存储状态数据，支持在网络间同步。  

![[5dd82cccaaa8c958f85a95b558b84815_MD5.png]]

上图应该可以比较清晰的阐明，UE 是如何充分利用 Actor 的本身机制来反过来实现对 Actor 的逻辑控制，相信亲爱的读者朋友们也能自行体会到它的优雅之处。对比其他的游戏引擎，往往它们都止步于 Actor 这一个层次，只提供了最基本的对象层次，美名其曰交给玩家控制。UE 为我们提供了这一套简洁强大的机制，大大方便了我们编写逻辑的难度。

而下篇我们的逻辑之旅将再继续拔高一个层次，将开始讲解 World 层次的逻辑，这个世界的意志：GameMode！

# 7 GameMode 和 GameState

## 引言

上文我们说到在 Actor 层次，UE 用 Controller 来充当 APawn 的逻辑控制者，也有了可以接受玩家输入的 PlayerController，和能自行行动的 AIController。Actor 的逻辑编写介绍完了，那么本篇，我们继续爬升，对于由 Actors 组成的 Level 这一层次，UE 又是怎么控制的呢？  
对 Level 记不太清楚的朋友，可以翻回去查看 “GamePlay 架构（二）Level 和 World” 的讲述，简单概括就是 World 是由一个 PersistentLevel 和一些 subLevels 组成的，PersistentLevel 切换了，相应的 World 也会切换。所以本文的关注点是在这么一个对象层次结构下，UE 是怎么设计的，我们又能做些什么。

## GameMode

Level，在游戏里的概念里，就是关卡的意思。同时作为游戏的玩家和开发者，我们总是会非常经常的提起关卡，但是关卡具体又是个什么定义呢？游戏里的哪些部分可以算是一个关卡？简单的我们都知道有《愤怒的小鸟》或《植物大战僵尸》的关卡，复杂的有大型 FPS 游戏里的关卡，而对于更大型的《暗黑 3》或者大型无缝地图 RPG 游戏《巫师 3》，甚至是号称超级广阔宇宙《无人深空》，我们能直接了当的说出哪部分是关卡吗？游戏行业发展如今，为了更好的组织游戏逻辑和内容资源，也发展出了一些概念来更好的理解和阐述，虽然叫法不同，不过含义理念都是相通的。比如，Cocos2dx 会认为游戏就是由不同的 Scene 切换组成的，每个 Scene 又由 Layer 组成；Unity 也认为游戏就是一个个 Scene；而 UE 的视角的是，游戏是由一个个 World 组成的，World 又是由 Level 组成的。这些概念有什么不同？  

**让我们从游戏本身的机制上分析：**
*   **游戏或玩家的节奏**，游戏可以分成一个个阶段，马里奥里的关卡就是一个阶段，而 RPG 游戏的一个大地图也是一个阶段。一个游戏也可能只有一个阶段，比如一直在宇宙里漫游的游戏。通常一个阶段结束后，会有一个结算，阶段之间，玩家也能明显感觉到切换感。
*   **游戏的机制**，有时候即使是同样的场景，玩家却也能感觉就像在玩两个不同的游戏，比如 MOBA 里的同一张地图上的各种不同挑战模式。
*   **游戏的资源划分**，有时候也能遇见同一个玩法应用在不同的场景上，比如赛车游戏的不同跑道。有时候也会在游戏的大地图里从酷热的沙漠到寒冷的极地。游戏开发中也总是倾向于给游戏用到的资源划分成组的进行载入和释放。

通过以上的分析，也和以前的一贯思路一样，我们发现在思考 “关卡” 这件事情上，也是要保持头脑清晰的分清 “表示” 和“逻辑”。**玩法就是“逻辑”，场景就是“表示”。所以我们如果以逻辑来划分游戏，得到的就是一个个 World 的概念；如果以表示来划分，得到就是一个个 Level**。一场游戏中，玩法再复杂但也只有一个，场景却可以无限大，所以可以有很多个表示拼接组装，因此是 World 包含 Level，而不是反过来。现在回过头来回想一下 Cocos2dx 和 Unity 的世界观，它们的概念还只是在表示层，在游戏实例和关卡之间少了一个更高级的逻辑概念。  
**因此 UE 的世界观是，World 更多是逻辑的概念，而 Level 是资源场景表示。** 以《巫师 3》为例，有好几个国家之间通过传送切换，国家内大地图无缝漫游，显然我们知道不可能把一个国家的所有资源都加载进内存，因此在 UE 里，一个国家就是许多个 Level 拼接的，而一个国家就是一个 World，它们可以有不同的模式玩法。但毕竟 AAA 游戏很少，通常的，我们的游戏比较简单的用一个 Level 就够了，否则这个场景表示的概念就应该叫 Area 更合适了，也因此通常的这里的 Level 也常常对应游戏里玩家面对的 "关卡"，也因此 UE 里 Level 的 Settings 叫做 WorldSettings 了。  
厘清了这些概念了之后，我们就知道，当我们在谈 Level 的业务逻辑控制的时候，我们实际上谈的是 World 的业务逻辑。按照 UE 的设计理念和经过 Controller 的经历，我想我也不用多解释了从 Actor 再派生出一个 WorldController 的方式了，可以直接的享受 Actor 已经提供的一切福利。一个 World 的 Controller 想不出有什么需要展示渲染的，因此可以直接从 AInfo 派生吧。哦，WorldController 是我瞎编的，在 UE3 里它叫做 GameInfo，到了 UE4 它改名为了 `GameMode`。笼统的讲，一个 World 就是一个 Game，把玩法叫做 Mode，我们应该也能接受吧。那我们来看看它：  

![[ff5cbe14bd917023fc60dd39a31662a2_MD5.png]]

既然勇敢的承担了游戏逻辑的职责，说他是 AInfo 家族里的扛把子也不为过，因此 GameMode 身为一场游戏的唯一逻辑操纵者身兼重任，在功能实现上有许多的接口，但主要可以分为以下几大块：

1.  **Class 登记**，GameMode 里登记了游戏里基本需要的类型信息，在需要的时候通过 UClass 的反射可以自动 Spawn 出相应的对象来添加进关卡中。前文说过的 Controller 的类型登记也是在此，GameMode 就是比 Controller 更高一级的领导。

![[08a9083228c88c88ed0e7c9869db8e03_MD5.png]]

1.  **游戏内实体的 Spawn**，不光登记，GameMode 既然作为一场游戏的主要负责人，那么游戏的加载释放过程中涉及到的实体的产生，包括玩家 Pawn 和 PlayerController，AIController 也都是由 GameMode 负责。最主要的 SpawnDefaultPawnFor、SpawnPlayerController、ShouldSpawnAtStartSpot 这一系列函数都是在接管玩家实体的生成和释放，玩家进入该游戏的过程叫做 Login（和服务器统一），也控制进来后在什么位置，等等这些实体管理的工作。GameMode 也控制着本场游戏支持的玩家、旁观者和 AI 实体的数目。
2.  **游戏的进度**，一个游戏支不支持暂停，怎么重启等这些涉及到游戏内状态的操作也都是 GameMode 的工作之一，SetPause、ResartPlayer 等函数可以控制相应逻辑。
3.  **Level 的切换**，或者说 World 的切换更加合适，GameMode 也决定了刚进入一场游戏的时候是否应该开始播放开场动画（cinematic），也决定了当要切换到下一个关卡时是否要 bUseSeamlessTravel，一旦开启后，你可以重载 GameMode 和 PlayerController 的 GetSeamlessTravelActorList 方法和 GetSeamlessTravelActorList 来指定哪些 Actors 不被释放而进入下一个 World 的 Level。
4.  **多人游戏的步调同步**，在多人游戏的时候，我们常常需要等所有加入的玩家连上之后，载入地图完毕后才能一起开始逻辑。因此 UE 提供了一个 MatchState 来指定一场游戏运行的状态，意义看名称也是不言自明的，就是用了一个状态机来标记开始和结束的状态，并触发各种回调。  
```c++
/** Possible state of the current match, where a match is all the gameplay that happens on a single map */  
namespace MatchState  
{  
extern ENGINE_API const FName EnteringMap; // We are entering this map, actors are not yet ticking  
extern ENGINE_API const FName WaitingToStart; // Actors are ticking, but the match has not yet started  
extern ENGINE_API const FName InProgress; // Normal gameplay is occurring. Specific games will have their own state machine inside this state  
extern ENGINE_API const FName WaitingPostMatch; // Match has ended so we aren't accepting new players, but actors are still ticking  
extern ENGINE_API const FName LeavingMap; // We are transitioning out of the map to another location  
extern ENGINE_API const FName Aborted; // Match has failed due to network issues or other problems, cannot continue  
}```

**思考：多个 Level 配置不同的 GameMode 时采用的是哪一个 GameMode？**  
我们知道除了配置全局的 GameModeClass 之外，我们还能为每个 Level 单独的配置不同的 GameModeClass。但是当一个 World 由多个 Level 组成的时候，这样就相当于配置了多个 GameModeClass，那么应用的是哪一个？首先第一个原则需要记住的就是，**一个 World 里只会有一个 GameMode 实例**，否则肯定乱套了。因此**当有多个 Level 的时候，一定是 PersistentLevel 和多个 StreamingLevel，这时就算它们配置了不同的 GameModeClass，UE 也只会为第一次创建 World 时加载 PersistentLevel 的时候创建 GameMode，在后续的 LoadStreamingLevel 时候，并不会再动态创建出别的 GameMode，所以 GameMode 从始至终只有一个，PersistentLevel 的那个。**

**思考：Level 迁移时 GameMode 是否保持一致？**  
在在 travelling 的时候，如果下一个 Level 的配置的 GameModeClass 和当前的不同，那么迁移后是哪个 GameMode？  
无论 travelling 采用哪种方式，当前的 World 都会被释放掉，然后加载创建新的 World。但这个过程中，有点区别的是根据 `bUseSeamlessTravel` 的不同，UE 可以选择哪些 Actor 迁移到下一个 World 中去（实现方式是先创建个中间过渡 World 进行二段迁移（为了避免同时加载进两个大地图撑爆内存），具体见引用 3）。分两种情况：  
1. 不开启 bUseSeamlessTravel，那么在 travelling 的时候（ServerTravel 或 ClientTravel），当前的 World 会被释放，所以当前的 GameMode 就被释放掉。新的 World 加载，就会根据新的 GameModeClass 创建新的 GameMode。所以这时是不同的。  
2. 开启 bUseSeamlessTravel，travelling 时，当前 World 的 GameMode 会调用 GetSeamlessTravelActorList：

```c++
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

在第一步从 CurrentWorld 到 TransitionWorld 的迁移时候，`bToTransition==true`，这个时候 GameMode 也会迁移进 TransitionWorld（TransitionMap 可以在 ProjectSettings 里配置），也包括 GameState 和 GameSession，然后 CurrentWorld 释放掉。第二步从 TransitionWorld 到 NewWorld 的迁移，GameMode（已经在 TransitionWorld 中了）会再次调用 GetSeamlessTravelActorList，这个时候 `bToTransition==false`，所以第二次的时候如代码所见当前的 GameMode、GameState 和 GameSession 就被排除在外了。这样 NewWorld 再继续 InitWorld 的时候，一发现当前没有 GameMode，就会根据配置的 GameModeClass 重新生成一个出来。所以这个时候 GameMode 也是不同的。  
**结论是，UE 的流程 travelling，GameMode 在新的 World 里是会新生成一个的，即使 Class 类型一致，即使 bUseSeamlessTravel，因此在 travelling 的时候要小心 GameMode 里保存的状态丢失。不过 Pawn 和 Controller 默认是一致的。**

> [!question] 
> **思考：哪些逻辑应该写在 GameMode 里？哪些应该写在 Level Blueprint 里？**  

我们依旧要问这个老土的问题。根据我们前面的知识，我们知道每个 Level 其实也是有自己的 LevelScriptActor 的，那么这两个有什么区别？可以从这几个方面来回答：

*   概念上，Level 是表示，World 是逻辑，一个 World 如果有很多个 Level 拼在一起，那么也就是有了很多个 LevelScriptActor，无法想象在那么多个地方写一个完整的游戏逻辑。**所以 GameMode 应该专注于逻辑的实现，而 LevelScriptActor 应该专注于本 Level 的表示逻辑**，比如改变 Level 内某些 Actor 的运动轨迹，或者某一个区域的重力，或者触发一段特效或动画。而 GameMode 应该专注于玩法，比如胜利条件，怪物刷新等。
*   组合上，同 Controller 应用到 Pawn 一样道理，因为 GameMode 是可以应用在不同的 Level 的，所以**通用的玩法应该放在 GameMode 里。**
*   **GameMode 只在 Server 存在**（单机游戏也是 Server），对于已经连接上 Server 的 Client 来说，因为游戏的状态都是由 Sever 决定的，Client 只是负责展示，所以 Client 上是没有 GameMode 的，但是有 LevelScriptActor，所以 **GameMode 里不要写 Client 特定相关的逻辑，比如操作 UI 等**。但是 LevelScriptActor 还是有的，而且支持 RPC，**即使如此，LevelScriptActor 还是应该只专注于表现，比如网络中触发一个特效火焰**。至于 UI，可以通过 PlayerController 的 RPC，然后转发到 GameInstance 来操作。
*   跟下层的 PlayerController 比较，GameMode 关心的是构建一个游戏本身的玩法，PlayerController 关心的玩家的行为。这两个行为是独立正交可以自由组合的。所以**想想哪些逻辑属于游戏，哪些属于玩家，就应该清楚写在哪里了。**
*   跟上层的 GameInstance 比较，GameInstance 关注的是更高层的不同 World 之间的逻辑，虽然有时候他也把手伸下来做些 UI 的管理工作，不过严谨来说，在 UE 里 UI 是独立于 World 的一个结构，所以也还算能理解。**因此可以把不同 GameMode 之间协调的工作交给 GameInstance，而 GameMode 只专注自己的玩法世界。**

## GameState

上回说到了 APlayerState 用来保存玩家的游戏数据，那么同样的，对于一场游戏，也需要一个 State 来保存当前游戏的状态数据，比如任务数据等。跟 APlayerState 一样，GameState 也选择从 AInfo 里继承，这样在网络环境里也可以 Replicated 到多个 Client 上面去。  

![[568304d68e5bb5a41c29dbd4931ae780_MD5.png]]

比较简单，第一个 MatchState 和相关的回调就是为了在网络中传播同步游戏的状态使用的（记得 GameMode 在 Client 并不存在，但是 GameState 是存在的，所以可以通过它来复制），第二部分是玩家状态列表，同样的如果在 Client1 想看到 Client2 的游戏状态数据，则 Client2 的 PlayerState 就必须广播过来，因此 GameState 把当前 Server 的 PlayerState 都收集了过来，方便访问使用。  
关于使用，开发者可以自定义 GameState 子类来存储本 GameMode 的运行过程中产生的数据（那些想要 replicated 的!），如果是 GameMode 游戏运行的一些数据，又不想要所有的客户端都可以看到，则也可以写在 GameMode 的成员变量中。重复遍，**PlayerState 是玩家自己的游戏数据，GameInstance 里是程序运行的全局数据。**

### GameSession

是在网络联机游戏中针对 Session 使用的一个方便的管理类，并不存储数据，本文重点也不在网络，故不做过多解释，可暂时忽略，留待网络章节再讨论。在单机游戏中，也存在该类对象用来 LoginPlayer，不过因为只是作为辅助类，那也可看作 GameMode 本身的功能，所以不做过多讨论。

## 总结

现在，我们也算讨论完了 Level（World）层次的控制，对于一场游戏而言，我们最关心的是怎么协调好整个场景的表现（LevelBlueprint）和游戏玩法的编写（GameMode）。UE 再次用 Actor 分化派生的思想，用同样套路的 AGameMode 和 AGameState 支持了玩法和表现的解耦分离和自由组合，并很好的支持了网络间状态的同步。同时也提供了一个逻辑的实体来负责创建关系内那些关键的 Pawn 和 Controller 们，在关卡切换（World）的时候，也有了一个负责对象来处理一些本游戏的特定情况处理。  

![[6ae32c6c874f92f335bb4564cf5b58c0_MD5.png]]

我们的逻辑之旅还没到终点，让我们继续爬升，下篇将介绍 Player。


### 4.14 版本 GameMode，GameState 的清理
根据搜索到的最早记录 "[[Request/Improvment] GameMode cleanup.](https://forums.unrealengine.com/showthread.php?39840-Request-Improvment-GameMode-cleanup)"(09-14-2014)，是有人抱怨当前的 GameMode 实现了太多的默认逻辑（例如多人的 Match），虽然方便了一些人使用，但是也确实加大了理解的难度，并且有时候还得去屏蔽删除一些默认逻辑。然后顺便吐槽了一番 AActor 里的 Damage，笔者也表示这确实不是 AActor 应该管的事情。  
言归正传，UE 在 2016-08-24 的时候开始加进 roadmap，并终于在 4.14 里实现完成了。如前所述，就是把 GameMode 和 GameState 的一些共同最基础部分抽到基类 AGameModeBase 和 AGameStateBase 里，并把现在的 GameMode 和 GameState 依然当作多人联机的默认实现。所以以后大家**如果想实现一个比较简单的单机 GameMode 就可以直接从 AGameModeBase 里继承了。**

![[2eeb7821615ea2f3fc91b706e01956c1_MD5.png]]

可以看到，其实就是把 MatchState 给往下拉了一层，并把一些多人玩家控制的逻辑，合起来就是**网络联机游戏的默认逻辑给抽离开了（如果做单机，只需要使用 GameModeBase）**。同样的对于 GameState 也做了处理：  

![[e31ff406ddb69178ad7119d6d819cfed_MD5.png]]

把 MatchState 也抽离到了下层，并增加了几个方便的字段引用（如 AuthorityGameMode）。总体功能职责架构上还是没有什么大变化的，吓死我了。

### GameMode
即使最开放的游戏也拥有基础规则，而这些规则构成了 **Game Mode**。在最基础的层面上，这些规则包括：
- 出现的玩家和观众数量，以及允许的玩家和观众最大数量。
- 玩家进入游戏的方式，可包含选择生成地点和其他生成/重生成行为的规则。
- 游戏是否可以暂停，以及如何处理游戏暂停。
- 关卡之间的过渡，包括游戏是否以动画模式开场。

Game Modes 的任务是定义和实现规则。Game Modes 当前常用的基类有两个。
4.14 版本中加入了 `AGameModeBase`，这是所有 Game Mode 的基类，是经典的 `AGameMode` 简化版本。`AGameMode` 是 4.14 版本之前的基类，仍然保留，功能如旧，但现在是 `AGameModeBase` 的子类。
由于其比赛状态概念的实现，`AGameMode` 更适用于标准游戏类型（如多人射击游戏）。
`AGameModeBase` 简洁高效，是新代码项目中包含的全新默认游戏模式。

**一款游戏可拥有任意数量的 Game Mode**，因此也可拥有任意数量的 `AGameModeBase` 类子类；然而，**给定时间上只能使用一个 Game Mode**。每次关卡进行游戏实例化时 Game Mode Actor 将通过 `UGameEngine::LoadMap()` 函数进行实例化。

**Game Mode 不会复制到加入多人游戏的远程客户端；它只存在于服务器上**，因此本地客户端可看到之前使用过的留存 Game Mode 类（或蓝图）；但无法访问实际的实例并检查其变量，确定游戏进程中已发生哪些变化。
如玩家确实需要更新与当前 Game Mode 相关的信息，可将信息保存在一个 `AGameStateBase` Actor 上，轻松保持同步。`AGameStateBase` Actor 随 Game Mode 而创建，之后被复制到所有远程客户端。

默认设置：
![[Pasted image 20230115223623.png]]
在关卡编辑器的 WorldSetting 中重载，实际运行以重载的 GameMode 为准：
WorldSetting 并不是设置 World 的属性（不要混肴），是针对 Level 的设置。仅代表当前一个关卡，并不是所有关卡。
![[Pasted image 20230115223741.png|300]]

### GameState
基于规则的事件在游戏中发生，需要进行追踪并和所有玩家共享时，信息将通过 **Game State** 进行存储和同步。这些信息包括：
- 游戏已运行的时间（包括本地玩家加入前的运行时间）。
- 每个个体玩家加入游戏的时间和玩家的当前状态。
- 当前 Game Mode 的基类。
- 游戏是否已开始。

**Game State** 负责启用客户端监控游戏状态。从概念上而言，Game State 应该管理所有已连接客户端已知的信息（特定于 Game Mode 但不特定于任何个体玩家）。它能够追踪游戏层面的属性，如已连接玩家的列表、夺旗游戏中的团队得分、开放世界游戏中已完成的任务，等等。

Game State 并非追踪玩家特有内容（如夺旗比赛中特定玩家为团队获得的分数）的最佳之处，因为它们由 **Player State** 更清晰地处理。**整体而言，GameState 应该追踪游戏进程中变化的属性。这些属性与所有人皆相关，且所有人可见。** 
**Game mode 只存在于服务器上，而 Game State 存在于服务器上且会被复制到所有客户端，保持所有已连接机器的游戏进程更新。**

# 8 Player
## 引言

回顾上文，我们谈完了 World 和 Level 级别的逻辑操纵控制，如同分离组合的 AController 一样，UE 在 World 的层次上也采用了一个分离的 AGameMode 来抽离了游戏关卡逻辑，从而支持了逻辑的组合。本篇我们继续上升一个层次，考虑在 World 之上，游戏还需要哪些逻辑控制？  
暂时不考虑别的功能系统（如社交系统，统计等各种），单从游戏性来讨论，现在闭上眼睛，想象我们已经藉着 UE 的伟力搭建了好了一个个 LevelWorld，嗯，就像《西部世界》一样，场景已经搭建好了，世界规则故事也编写完善，现在需要干些什么？当然是开始派玩家进去玩啦！

大家都是老玩家了，想想我们之前玩的游戏类型：
*   玩家数目是单人还是多人
*   网络环境是只本地还是联网
*   窗口显示模式是单屏还是分屏
*   输入模式是共用设备还是分开控制（比如各有手柄）
*   也许还有别的不同

按照软件工程的理念，没有什么问题是不能通过加一个间接层解决的，不行就加两层！所以既然我们在处理玩家模式的问题，理所当然的是加个间接层，将玩家这个概念抽象出来。  
**在游戏引擎看来，玩家就是输入的发起者**。游戏说白了，也只是接受输入产生输出的一个程序。所以有多少输入，这些输入归多少组，就有多少个玩家。这里的输入不止包括本地键盘手柄等输入设备的按键，也包括网线里传过来的信号，是广义的该游戏能接受到的外界输入。注意输出并不是玩家的必要属性，一个玩家并不一定需要游戏的输出，想象你闭上眼睛玩马里奥或者有个网络连接不断发送来控制信号但是从来不接收反馈，虽然看起来意义不大，但也确实不能说这就不是游戏。  
**在 UE 的眼里，玩家也是如此广义的一个概念。本地的玩家是玩家，网络联机时虽然看不见对方，但是对方的网络连接也可以看作是个玩家**。当然的，本地玩家和网络玩家毕竟还是差别很大，所以 UE 里也对二者进行了区分，才好更好的管理和应用到不同场景中去，比如网络玩家就跟本地设备的输入没多大关系了嘛。

## UPlayer

让我们假装自己是 UE，开始编写 Player 类吧。为了利用上 UObject 的那些现有特性，所以肯定是得从 UObject 继承了。那能否是 AActor 呢？Actor 是必须在 World 中才能存在的，而 Player 却是比 World 更高一级的对象。玩游戏的过程中，LevelWorld 在不停的切换，但是玩家的模式却是脱离不变的。另外，Player 也不需要被摆放在 Level 中，也不需要各种 Component 组装，所以从 AActor 继承并不合适。那还是保持简单吧，直接继承自UObject：  

![[dd8b6f82366c93fce61070c65ca5b9e7_MD5.png]]

## ULocalPlayer

然后是本地玩家，从 Player 中派生下来 LocalPlayer 类。对本地环境中，一个本地玩家关联着输入，也一般需要关联着输出（无输出的玩家毕竟还是非常少见）。玩家对象的上层就是引擎了，所以会在 GameInstance 里保存有 LocalPlayer 列表。  

![[762eea989265e1cd724ec1976965eeb2_MD5.png]]

```c++
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

```c++
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

```c++
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

可见，对于 ULocalPlayer，APlayerController 内部会开始 InitInputSystem()，接着会创建相应的 UPlayerInput，BuildInputStack 等初始化出和 Input 相关的组件对象。现在先明白到 **LocalPlayer 才是 PlayerController 产生的源头，也因此才有了 Input** 就够了，特定的 Input 事件流程分析在后续章节再细述。

> [!question] 
> **思考：为何不在 LocalPlayer 里编写逻辑？**  

作为游戏开发者，相信大家都有这么个体会，往往在游戏逻辑代码中总会有一个自己的 Player 类，里面放着这个玩家的相关数据和逻辑业务。可是在 UE 里为何就不见了这么个结构？也没见 UE 在文档里有描述推荐你怎么创建自己的 Player。  
这个可能有两个原因，一是 UE 从 FPS-Specify 游戏起家，不像现在的各种手游有非常重的玩家系统，在 UE 的眼中，Level 和 World 才是最应该关注的对象，因此 UE 的视角就在于怎么在 Level 中处理好 Player 的逻辑，而非在 World 之外的额外操作。二是因为在一个 World 中，上文提到其实已经有了 Pawn-PlayerController 和 PlayerState 的组合了，表示、逻辑和数据都齐备了，也就没必要再在 Level 掺和进 Player 什么事了。当然你也可以理解为 PlayerController 就是 Player 在 Level 中的话事人。  
凡事留一线，日后好相见。尽管如此，UE 还是给了我们自定义 ULocalPlayer 子类的机会：

```c++
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

本篇我们抽象出了 Player 的概念，并依据使用场景派生出了 LocalPlayer 和 NetConnection 这两个子类，从此 Player 就不再是一个虚无缥缈的概念，而是 UE 里的逻辑实体。UE 可以根据生成的 Player 对象的数量和类型的不同，在此上实现出不同的玩家控制模式，**LocalPlayer 作为源头 Spawn 出 PlayerController 继而 PlayerState** 就是实证之一。而在网络联机时，把一个网络连接看作是一个玩家这个概念，把在 World 之上的输入实体用 Player 统一了起来，从而可以实现出灵活的本地远程不同玩家模式策略。  
尽管如此，UPlayer 却像是深藏在 UE 里的幕后功臣，UE 也并不推荐直接在 Player 里编程，而是利用 Player 作为源头，来产生构建一系列相关的机制。但对于我们游戏开发者而言，知道并了解 UE 里的 Player 的概念，是把现实生活同游戏世界串联起来的很重要的纽带。我们在一个个 World 里向上仰望，还能清楚的看见一个个 LocalPlayer 或 NetConnection 仿佛在注视着这片大地，是他们为 World 注入了生机。  
已经到头了？并没有，我们继续向上逆风飞翔，终将得见游戏里的神：GameInstance。

# 9 GameInstance
一人之下，万人之上

## 引言

上篇我们讲到了 UE 在 World 之上，继续抽象出了 Player 的概念，包含了本地的 ULocalPlayer 和网络的 UNetConnection，并以此创建出了 World 中的 PlayerController，从而实现了不同的玩家模式策略。一路向上，依照设计里一个最朴素的原理：自己是无法创建管理自身的，所以 Player 也需要一个创建管理和存储的地方。另一方面，上文提到 Player 固然可以负责一些跟玩家相关的业务逻辑，但是对于 World 之上协调管理的逻辑却也仍然无处安放。  
如果是有一定的游戏开发实战经验的朋友也一定能体会到，在自己开发的游戏中，往往除了我们上文提到的 Player 类，常常会创建一个 Game 类，比如 BattleGame、WarGame 或 HappyGame 等等。Game 之前的名词往往都是游戏的开发代号。这倒不是因为我们如此热衷创建各种 Manager 类，而是确实需要一个大管家来干一些协调的活。一般的游戏引擎都只会暴露给你它自己引擎的管理类，如 Director，Engine 或 Application 之类的，但是却不会主动在 Game 类的创建管理上为你提供方便。游戏引擎的出现，最开始其实只是因为一些人发现游戏做着做着，有一大部分功能是可以复用的，于是就把它抽离了出来方便做下一款游戏。在那个时候，人们对游戏还是处于开荒探索的阶段，游戏引擎只是一大堆功能的复合体，就像叮当猫的口袋一样，互相比谁掏出的工具最强大。然而即使到了现代，绝大部分的引擎的思想却还停留在上个世纪，仍然执着于罗列 Feature 列表，却忘了真正的游戏开发人员天天面对的游戏业务逻辑编写，没有思考在那方面如何也下一番功夫去帮助开发者。人们对比 UE 和其他游戏引擎时，也会常常说出的一句话是：“别忘了 Epic 自己也是做游戏的”（虚幻竞技场，战争机器，无尽之剑……）。从这一点也可以看出，UE 很大的得益于 Epic 实战游戏开发的反哺，这一方面 Unity 就有点吃亏了，没有自己亲自下手干脏活累活，就不懂得急人民群众之所急。所以如果一个游戏引擎能把 GamePlay 也做好了，那就不止是口袋了，而是知你懂你的叮当猫本身。

## GameInstance

简单的事情就不用多讲了，UE 提供的方案是一以贯之的，为我们提供了一个 GameInstance 类。为了受益于 UObject 的反射创建能力，直接继承于 UObject，这样就可以依据一个 Class 直接动态创建出来具体的 GameInstance 子类。  

![[a38d1f5a8c38225e0f83b6e2a0a4d5fc_MD5.png]]

我并不想罗列所有的接口，UGameInstance 里的接口大概有 4 类：

1. **引擎的初始化加载**，Init 和 ShutDown 等（在引擎流程章节会详细叙述）
2. **Player 的创建**，如 CreateLocalPlayer，GetLocalPlayers 之类的。
3. **GameMode 的重载修改**，这是从 4.14 新增加进来改进，本来你只能为特定的某个 Map 配置好 GameModeClass，但是现在 GameInstance 允许你重载它的 PreloadContentForURL、CreateGameModeForURL 和 OverrideGameModeClass 方法来 hook 改变这一流程。
4. **OnlineSession 的管理**，这部分逻辑跟网络的机制有关（到时候再详细介绍），目前可以简单理解为有一个网络会话的管理辅助控制类。

而 **GameInstance 是在 GameEngine 里创建的**（先不谈 UEditorEngine）：

```c++
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

> [!question] 
> **思考：GameInstance 只有一个吗？**  

一般而言，是的。对于我们自己开发的游戏而言，我们始终只需要关注自己的一亩三分地，那么你可以认为你子类化的那个 GameInstance 就像个单件一样，全局唯一只有一个，从游戏的开始到结束。但既然是本系列文章的读者，自然也是不甘于只了解这么多的。  
正如把网络连接也当作 Player 这个概念一样，我们此时也需要重新审视一下 Game 这个概念。**什么是一个 Game?** 对于玩家而言，Game 就是从打开到关闭的这整个过程说展现的内容。但是对于开发者来说，这个概念就需要扩充一下了。假设有个引擎支持双击图标一下子开出 4 个窗口来让 4 个玩家独立运行，你能说得清这是一个 Game 还是 4 个 Game 在运行吗？哪一种说法都能自圆其说，但关键是哪一种概念划分能更好的让我们管理组织结构。因此针对这种情况，如果是这 4 个窗口一点都不互相关联，或者只是单独的共用地图资源，那么用 4 个 Game 的概念来管理就更为合适。如果这 4 个窗口里运行的内容，实际上只是在同一个关卡里本地对战，内存里互相直接通信，那用一个 Game 加上 4 个 Player 的概念就会变得更合适。所以针对这点，你**可以把 Game 理解为就像进程一样，进程可以在同一个 exe 上多开，Game 也可以在同一份游戏资源上开出多个运行实例；进程之间可以互相通信协作，Game 的不同实例也可以互相沟通，不管是内存中直接在 Engine 的协调下完成，还是通过 Socket 通信。**  
另一方面，一般游戏引擎都只是服务于游戏本身，而对于其配套的各种编辑器就像是对待外来的打工者一样，编辑器往往只负责最终输出游戏资源。由于应用场景的不同，编辑器的架构也常常根据相应平台而定，五花八门，有用 Qt，MFC，WPF 等各种平台 UI 框架。而对于另一些有大志向的引擎，比如 Unity 和 UE，其编辑器就是采用引擎自绘的方案（其优劣暂不分析，以后聊到 UI 框架再细说）。所以游戏引擎这个时候，就更加的拔高了一个层次，就不再只是个 “游戏” 引擎了，而是个 “程序” 引擎了。因此 UE 本身的这套框架不光要服务游戏，还要服务编辑器，甚至是另外一些辅助程序。所以，Game 的概念也就扩充到了更上层的“程序”，变得更广义了。  
**言归正传，因为 UE 的这套 Editor 自绘机制，还有 PIE（PlayInEditor），进程里其实是可以同时有多个 GameInstance 的，如正在编辑的 EditorWorld 所属于的，和 Play 之后的 World 属于的。** 我想，这也就是为何 UE 把它叫做 GameInstance 而不是简单的 Game 的含义，其名字中就隐含了多个 Instance 的深意。我们现在再次回顾一下 ([GamePlay 架构（三）WorldContext，GameInstance，Engine](https://zhuanlan.zhihu.com/p/23167068)) 最后的结构图，了解一下 GameInstance 又是被谁管理的：  

![[fe68f967adb22cfeea680d61a78da666_MD5.png]]

当初我们是以数据的视角，在考察 WorldContext 的从属的时候讨论过这个结构。现在以逻辑的角度，明白了 GameInstance 也会被上层的 Engine 实例出来多个，就会有更深的理解了。

再扩充一下，在 Engine 之下允许同时运行多个 GameInstance，还会有许多其他好处，就像操作系统允许一份资源运行多个进程实例一样，Engine 就可以站在更高的层次上管理协调多个 Game，同时也能更加的深入到 Game 内部去得到更多的优化。比如未来要实现游戏本地的 host 多开并管理，或者在 Server 同时 Host 一个 Map 的多个实例 (现在只能一个…… 还是有很多工作要做啊)，这对于开发 MMO 网游是非常需要的功能，虽然目前 UE 在这一块的具体工作还有些薄弱，但至少可扩展的可能性是已经保证了的（动手能力强的高手可以在此基础上定制）。一般而言，间接多一层，就多了一层的灵活性，所以很多引擎其实就是把 Game 和 Engine 揉在了一块没有为了 GamePlay 框架而分开。

> [!question] 
> **思考：哪些逻辑应该放在 GameInstance？**  

第二个惯例的问题是，这一层应该写些什么逻辑。顾名思义，既然是**作为游戏中全局唯一的长者，我们就应该给他全局的控制权**。

在逻辑层面，GameInstance 往下看是：  
1. Worlds，Level 的切换实际发生地是 Engine，而 GameInstance 可以说是 UE 之神其下的唯一代言人，所以 GameInstance 也可以代之管理 World 的切换等。我们可以在 GameInstance 里实现各种逻辑最后调用 Engine 的 OpenLevel 等接口。  
2. Players，虽然一般来说我们直接控制 Players 的机会不多，都是配置好了就行。但要是到了需要的时候，GameInstance 也实现了许多的接口可以让你动态的添加删除 Players。  
3. UI，UE 的 UI 是另一套 World 之外的系统，虽然同属于 Viewport 的显示之下，但是控制结构跟 Actor 们并不一样。所以我们常常会需要控制 UI 各种切换的业务逻辑，虽然在 Widget 的 Graph 里也可以写些简单的切换，但是要想复用某些切换逻辑的时候，在特定的 Wdiget 里就不合适了，而 GameMode 一方面局限于 Level，另一方面又只存在于 Server；PlayerController 也是会切换掉的，同时又只存在于 World 中，所以最后比较合适的就剩下 GameInstance 了，以后当然有可能了可能会扩展出个 UI 的业务逻辑 Manger 类，不过那是后话了。  
4. 全局的配置，也常常需要根据平台改变一些游戏的配置，Execute 一些 ConsoleCommand，GameInstance 也是这些命令的存放地。  
5. 游戏的额外第三方逻辑，如果你的游戏需要其他一些控制，比如自己写的网络通信、自定义的配置文件或者自己的一些程序算法，如果简单的话，GameInstance 也可以一放，等复杂起来了，也可以把 GameInstance 当作一个模块容器，你可以在里面再扩展出来其他的子逻辑模块。当然如果是插件的话，还是在自己的插件 Module 里面自行管理逻辑，然后把协调工作交给 GameInstance 来做。

而在数据层面上，我们层层上来，已经有了针对一个 Player 的 Contoller 的 PlayerState，也有了针对 World 的 GameMode 的 GameState，到了更全局之上，自然的 GameInstance 就应该存储一些全局的状态数据。所以你可以在 GameInstance 的成员变量中添加一些全局的状态，或者是那些想要在 Level 之外持续存在的对象。**不过需要注意的一点是，GameInstance 成员变量中最好只保存那些 “临时” 的数据，而对于那些想要持久序列化保存的数据，我们就需要接下来的 SaveGame 了**。把持久的数据直接放在 SaveGame，用的时候直接读取出来，之后再直接在其上更新，好处是只用维护一份，省得要保存的时候，还去想到底要选 GameInstance 的哪些成员变量中来保存，一开始就设计选好，以后就方便了。

## SaveGame

UE 连玩家存档都帮你做了！得益于 UObject 的序列化机制，现在你只需要继承于 USaveGame，并添加你想要的那些属性字段，然后这个结构就可以序列化保存下来的。玩家存档也是游戏中一个非常常见的功能，差的引擎一般就只提供给你读写文件的接口，好一点的会继续给你一些序列化机制，而更好的则会服务得更加周到。UE 为我们在蓝图里提供了 SaveGame 的统一接口，让你只用关心想序列化的数据。  
USaveGame 其实就是为了提供给 UE 一个 UObject 对象，本身并不需要其他额外的控制，所以它的类是如此的简单以至于我能直接把它的全部声明展示出来：

```c++
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

关于存档数据关联的逻辑，再重复几句，对于那些需要直接在全局处理的数据逻辑，也可以直接在 SaveGame 中写方法来实现。比如实现 AddCoin 接口，对外隐藏实现，对内可以自定义附加一些逻辑。**USaveGame 可以看作是一个全局持久数据的业务逻辑类。跟 GameInstance 里的数据区分就是，GameInstance 里面的是临时的数据，SaveGame 里是持久的。清晰这一点区分，到时就不会纠结哪些属性放在哪里，哪些方法实现在哪里了。**

注意一下，SaveGameToSlot 里的 SlotName 可以理解为存档的文件名，UserIndex 是用来标识是哪个玩家在存档。UserIndex 是预留的，在目前的 UE 实现里并没有用到，只是预留给一些平台提供足够的信息。你也可以利用这个信息来为多个不同玩家生成不同的最后文件名什么的。而 ISaveGameSystem 是 IPlatformFeaturesModule 提供的模块接口，关于模块的机制，等引擎流程章节再说吧，目前可以简单理解为一个单件对象里提供了一些平台相关的接口对象。

## 总结

至此，我们可以说已经介绍完了 GamePlay 下半部分——逻辑控制。在蓝图层，UE 并不向 BP 直接暴露 Engine 概念，即使在 C++ 层，在实现 GamePlay 业务时也是很少需要真正直接操纵 Engine 的时候。如果 GamePlay 已经足够好，那么 Engine 自然就可以隐居幕后了。UE 用 GameInstance 实现了全局的控制，并支持多 GameInstance 来实现编辑器，最后在存档的时候还可以用到 SaveGame 的方便的接口。  
下篇，就是 GamePlay 章节的最终章，我们将会对 GamePlay 架构的（一到九）篇进行回顾归纳总结巩固，以一个承上启下总览的眼光，再来重新审视一下 UE 的整套 GamePlay 框架，下个章节见。



# 10 总结

## 引言

通过对前九篇的介绍，至此我们已经了解了 UE 里的游戏世界组织方式和游戏业务逻辑的控制。行百里者半九十，前述的篇章里我们的目光往往专注在于特定一个类或者对象，一方面固然可以让内容更有针对性，但另一方面也有了身在山中不见山的困惑。本文作为 GamePlay 章节的最终章，就是要回顾我们之前探讨过的内容，以一个更高层总览的眼光，把之前的所有内容有机组织起来，思考整体的结构和数据及逻辑的流向。

## 游戏世界

如果我们在最初篇所问的，如果让你来制作一款 3D 游戏引擎，你会怎么设计其结构？已经知道，在 UE 的眼里，游戏世界的万物皆 Actor，Actor 再通过 Component 组装功能。Actor 又通过 UChildActorComponent 实现 Actor 之间的父子嵌套。([GamePlay 架构（一）Actor 和 Component](https://zhuanlan.zhihu.com/p/22833151))

![[6ed5ef4dbe67c9b5a2db08151d8bc1a0_MD5.jpg]]

众多的各种 Actor 子类又组装成了 Level([GamePlay 架构（二）Level 和 World](https://zhuanlan.zhihu.com/p/22924838)):

![[083ab4270ff6f7bd035e59204ab8ebbf_MD5.png]]

如此每一个 Level 就拥有了一座 Actor 的森林，你可以根据自己的需要定制化 Level，比如有些 Level 是临时 Loading 场景，有些只是保存光照，有些只是一块静态场景。UE 用 Level 这种细一些粒度的对象为你的想象力提供了极大的自由度，同时也能方便团队内的平行协作。

一个个的 Level，又进一步组装成了 World:

![[9897c66a4c07ad9e4b4ec2f2d6cb7406_MD5.png]]

就像地球上的大陆板块一样，World 允许多个 Level 静态的通过位置摆放在游戏世界中，也允许运行时动态的加载关卡。

而 World 之间的切换，UE 用了一个 WorldContext 来保存切换的过程信息。玩家在切换 PersistentLevel 的时候，实际上就相当于切换了一个 World。而再往上，就是整个游戏唯一的 GameInstance，由 Engine 对象管理着。([GamePlay 架构（三）WorldContext，GameInstance，Engine](https://zhuanlan.zhihu.com/p/23167068))  

![[794292b1c7ae24baa5e24d70256fedea_MD5.png]]

到了 World 这一层，整个游戏的渲染对象就齐全了。但是游戏引擎并不只是渲染，因此为了让玩家也各种方式接入 World 中开始游戏。GameInstance 下不光保存着 World，同时也存储着 Player，有着 LocalPlayer 用于表示本地的玩家，也有 NetConnection 当作远端的连接。（[GamePlay 架构（八）Player](https://zhuanlan.zhihu.com/p/23826859)）：  

![[ea3becbbc94b76fbde904a173793f6fc_MD5.png]]

玩家利用 Player 对象接入 World 之后，就可以开始控制 Pawn 和 PlayerController 的生成，有了附身的对象和摄像的眼睛。最后在 Engine 的 Tick 心跳脉搏驱动下开始一帧帧的逻辑更新和渲染。

## 数据和逻辑

说完了游戏世界的表现组成，那么对于一个 GamePlay 框架而言自然需要与其配套的业务逻辑架构。GamePlay 架构的后半部分就自底向上的逐一分析了各个层次的逻辑载体，按照 MVC 的思想，我们可以把整个游戏的 GamePlay 分为三大部分：表现（View）、逻辑（Controller）、数据（Model）。一图胜千言：  

![[411cebb2b416dac68b7240c94768a758_MD5.jpg]]

(请点击看大图)  
最左侧的是我们已经讨论过的游戏世界表现部分，从最最根源的 UObject 和 Actor，一直到 UGameEngine，不断的组合起来，形成丰富的游戏世界的各种对象。

1.  从 UObject 派生下来的 AActor，拥有了 UObject 的反射序列化网络同步等功能，同时又通过各种 Component 来组装不同组件。UE 在 AActor 身上同时利用了继承和组合的各自优点，同时也规避了彼此的一些缺点，我不得不说，UE 在这一方面度把握得非常的平衡优雅，既不像 cocos2dx 那样继承爆炸，也不像 Unity 那样走极端全部组件组合。
2.  AActor 中一些需要逻辑控制的成员分化出了 APawn。Pawn 就像是棋盘上的棋子，或者是战场中的兵卒。有 3 个基本的功能：可被 Controller 控制、PhysicsCollision 表示和 MovementInput 的基本响应接口。代表了基本的逻辑控制物理表示和行走功能。根据这 3 个功能的定制化不同，可以派生出不同功能的的 DefaultPawn、SpectatorPawn 和 Character。([GamePlay 架构（四）Pawn](https://zhuanlan.zhihu.com/p/23321666))
3.  AController 是用来控制 APawn 的一个特殊的 AActor。同属于 AActor 的设计，可以让 Controller 享受到 AActor 的基本福利，而和 APawn 分离又可以通过组合来提供更大的灵活性，把表示和逻辑分开，独立变化。([GamePlay 架构（五）Controller](https://zhuanlan.zhihu.com/p/23480071))。而 AController 又根据用法和适用对象的不同，分化出了 APlayerController 来充当本地玩家的控制器，而 AAIController 就充当了 NPC 们的 AI 智能。([GamePlay 架构（六）PlayerController 和 AIController](https://zhuanlan.zhihu.com/p/23649987))。而数据配套的就是 APlayerState，可以充当 AController 的可网络复制的状态。
4.  到了 Level 这一层，UE 为我们提供了 ALevelScriptActor（关卡蓝图）当作关卡静态性的逻辑载体。而对于一场游戏或世界的规则，UE 提供的 AGameMode 就只是一个虚拟的逻辑载体，可以通过 PersistentLevel 上的 AWorldSettings 上的配置创建出我们具体的 AGameMode 子类。AGameMode 同时也是负责在具体的 Level 中创建出其他的 Pawn 和 PlayerController 的负责人，在 Level 的切换的时候 AGameMode 也负责协调 Actor 的迁移。配套的数据对象是 AGameState。([GamePlay 架构（七）GameMode 和 GameState](https://zhuanlan.zhihu.com/p/23707588))
5.  World 构建好了，该派玩家进来了。但游戏的方式多样，玩家的接入方式也多样。UE 为了支持各种不同的玩家模式，抽象出了 UPlayer 实体来实际上控制游戏中的玩家 PlayerController 的生成数量和方式。([GamePlay 架构（八）Player](https://zhuanlan.zhihu.com/p/23826859))
6.  所有的表示和逻辑汇集到一起，形成了全局唯一的 UGameInstance 对象，代表着整个游戏的开始和结束。同时为了方便开发者进行玩家存档，提供了 USaveGame 进行全局的数据配套。([GamePlay 架构（九）GameInstance](https://zhuanlan.zhihu.com/p/24005952))

UE 为我们提供了这些 GamePlay 的对象，说多其实也不多，而且其实也是这么优雅有机的结合在一起。但是仍然会把一些朋友给迷惑住了，常常就会问哪些逻辑该写在哪里，哪些数据该放在哪里，这么多个对象，好像哪个都可以。比如 Pawn，有些人就会说我就是直接在 Pawn 里写逻辑和数据，游戏也运行的好好的，也没什么不对。

如果你是一个已经对设计架构了然于心，也预见到了游戏未来发展变化，那么这么直接干也确实比较快速方便。但是这么做其实隐含了两个前提，一是这个 Pawn 的逻辑足够简单，把 MVC 的三者混合在一起依然不超过你的心智负担；二是已经断绝了逻辑和数据的分离，如果以后本地想复用一些逻辑创建另一个 Pawn 就会很麻烦，而且未来联机多玩家的状态复制也不支持。但说回来，人类的一个最常见的问题就是自大，对自己能力的过度自信，对未来变化的虚假掌控感。程序员在自己的编程世界里，呼风唤雨操作内存设备惯了，这种强大的掌控感非常容易地就外延到其他方面去了。你现在写的代码，过几个月后再回头看，是不是经常觉得非常糟糕？那奇怪了，当初写的时候怎么就感觉信心满满呢？所以踩坑多了的人就会自然的保守一些。另一方面，作为团队里的技术高手或老人，我个人觉得也有支持同行和提携后辈的责任，对自己而言只是多花一点点力气，却为别人树立一个清晰的程序结构典范，也传播了设计思想。程序员何苦为难程序员。

但还有一些人喜欢那么硬怼着干的原因要嘛是对未来的可预见性不足（经验不足），要嘛是对程序设计的基本原则不够了解（程序能力不够），比如最简单的 “单一职责”。在新手期，面对着 UE 的程序世界，虽然在已经懂的人眼里就那么几个对象，但是在新手眼里，往往就感觉复杂无比，面对未知，我们本能的反应是逃避，往往就倾向于哪些看起来这么用能工作，就像玩游戏一样，形成了你的 “专属套路”。跟穷人忙于工作而没力气提高自己是一个道理。相信我，所有的高手都是从小白过来的，我敢保证，他出生的时候脑袋也肯定是一片空白！区别是有些人后来不怕麻烦的勤能补拙，他努力的去理解这种设计模式的优劣，不局限于自己已经掌握的一片舒适区内，努力去设想未来的各种变化和应对之法，最终形成自己的独立思考。高手只是比新手懂得更多想得更多一些而已。

闲话说完。在分析 UE 这么一个 GamePlay 系统的时候，就像 UML 有各种图一样，我们也应该从各个切面去分析它的构成。这里有两大基本原则：单一职责和变化隔离，但也可以说只有一个。所有的程序设计模式都只是在抽象变化，把变化都抽离开了，剩下的不就是单一职责了嘛。所以 UE 里对 MVC 的实践其实也只是在不断抽离出各个对象的变化部分，把 Pawn 的逻辑抽出来是 Controller，把数据抽出来是 PlayerState。把 World 的 Level 静态逻辑抽出来是关卡蓝图，把动态的游戏玩法抽离出来是 GameMode，把游戏数据抽离出来是 GameState。具体的每个层次的数据和逻辑的关系前文已经一一详细说过了，此处就不再赘述了。但也再次着重探讨一些分析方法：

*   从竖直的角度来看，左侧是表示，中间是逻辑，右侧是数据。  
*   当我们谈到表示的时候，脑袋里想的应该是一个单纯的展示对象，就像一个基本的网络物体，它可以带一些基本的动画，再多一些功能，也顶多只能像一个木偶，有着一些非常机械原始的行为。我们让他前进，他可以知道左腿右腿交替着迈，但他是无知觉的。所以左侧的那一串对象，你应该尽量得让他们保持简单。
*   实现中间的逻辑的时候，你应该专注于逻辑本身，尽量的忘记两旁的表示和数据。去思考哪些逻辑是表示固有的还是比较智能判断的。哪些 Controller 或 Mode 我们应该尽量的让它们通用，哪些就让它们特定的负责某一块，有些也不能强求，自己把握好度。
*   右侧的数据，同样的保持简单。我们把它们分离出来的目的就是为了独立变化和在网络间同步，注意一下别走回头路了就好。我们应该只在此放置纯数据。

*   从水平的切面上看，依次自底向上，记住一个原则，**哪个层次的应该尽量只负责哪个层次的东西，不要对上层或下层的细节知道得太多，也尽量不要逾矩越权去指手画脚别的对象里的内务事**。大家通力协作，注重隐私，保持安全距离，不就社会和谐了嘛。  
    
*   最底层的 Component，应该只是实现一些与游戏逻辑无关的功能。理解这个 “无关” 是关键。**换个游戏，你这些 Component 依然可以用，就是所谓的游戏无关。**
*   Actor 层，通过 Pawn、Controller 和 PlayerState 的合作，根据需要旗下再派生出特定的 Character，或 PlayerController，AIController，但它们的合作模式，三大家族的长老们已经定下了，后辈们应该尽量遵守。这一层，关键的地方在于分清楚哪些是操作 Actor 的，别向下把 Actor 内部的功能给抽了出来，也别大包大揽把整个游戏的玩法也管了过来。脑袋保持清醒，这一层所做的事，就是为了让 Actor 们显得更加的智能。换句话说，这些智能的 Actor 组合，理论上是可以在随便哪个 Level 里用的。
*   Level 和 World 层，分清楚静态的关卡蓝图和动态可组合 GameMode。静态的意思是这个场景本身的运作机制，动态的指的是可以像切换比赛方式一样切换一场游戏的目的。在这一层上，你得有总览游戏大局的自觉了，咱们都是干大事的人，眼光就不要局限在那些一兵一卒那些小事了。制定好游戏规则，赋予这一场游戏以意义，是 GameMode 最重要的职责。注意两点，一是脑袋里有跟弦，**一旦开始联机环境了，GameMode 就升职到 Server 里去了，Client 就没有了，所以千万要小心别在 GameMode 做些客户端的小事**；二是 GameState 是表示一场游戏的数据的，而 PlayerState 是表示 Controller 的数据，对象和范围都不同，不能混了。
*   GameInstance 层，一般来说 Player 不需要你做太多事情，UE 已经帮你处理好了。虽说力量越大，责任就越大，但领导日理万机累坏了也不行是吧。所以 GameInstance 作为全局的唯一逻辑对象，我们如果能不打扰他就尽量少把事推给他，否则你很快就会看着 GameInstance 里堆着一山东西。GameInstance 身在高层，应该只尽量做一些 Level 之间的协调工作。而 SaveGame 也应该尽量只保存游戏持久的数据。

自始至终，回顾一下每个类的本身的职责，该是他的就是他的，别人的不要抢。读者朋友们，如果到此觉得似乎懂了一些，但还是觉得不够深刻理解的话，也没关系，凡事不能一蹴而就，在开发过程中多想多琢磨自然而然就会慢慢领悟了。

## 整体类图

从类的继承层次上，咱们再加深一下理解。下图只列出了 GamePlay 架构里一些相关的重要的类：  

![[a6116bb849954485b938b8394f46b861_MD5.jpg]]

(请点击看大图)  
由此也可以看出来，UE 基于 UObject 的机制出发，构建出了纷繁复杂的游戏世界，几乎所有的重要的类都直接或间接的继承于 UObject，都能充分利用到 UObject 的反射等功能，大大加强了整体框架的灵活度和表达能力。比如 GamePlay 中最常用到根据某个 Class 配置在运行时创建出特定的对象的行为就是利用了反射功能；而网络里的属性同步也是利用了 UObject 的网络同步 RPC 调用；一个 Level 想保存成 uasset 文件，或者 USaveGame 想存档，也都是利用了 UObject 的序列化；而利用了 UObject 的 CDO（Class Default Object），在保存时候也大大节省了内存；这么多 Actor 对象能在编辑器里方便的编辑，也得益于 UObject 的属性编辑器集成；对象互相引用的从属关系有了 UObject 的垃圾回收之后我们就不用担心会释放问题了。想象一下如果一开始没有设计出 UObject，那么这个 GamePlay 框架肯定是另一番模样了。

## 总结

对于 GamePlay 我们从构建游戏世界开始，再到一层层的逻辑控制，本篇也从各个切面上总结归纳了整体架构。希望读者们好好领会 UE 的 GamePlay 架构思想，别贪快，整体上慢慢琢磨以上的架构图，细节上可以回顾过往的单篇来了解。

对于这一套 UE 提供的 GamePlay 框架，我们既然选择了用 UE 引擎，那么自然就应该想着怎么充分利用好它。框架就是你如果在它的规则下办事，那它就是事半功倍的助力器，你会常常发现 UE 怎么连这个也帮你做完了；而如果你在不了解的情况下想逆着它行事，就常常感受到怎么哪里都受到束缚。我们对于框架的理念应该就像是对待一辆汽车一般，我们关心的是怎么驾驶它到达想要的目的地，而不是折腾着怪它四个轮子不能按照你的心意朝不同方向乱转。对比隔壁的 Cocos2dx、或 Unity、或 CryEngine，UE 能够提供这么一个完善的 GamePlay 框架，对我们开发者而言，是一件幸福的事，不是吗？
