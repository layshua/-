## 引言  

如果让你来制作一款 3D 游戏引擎，你会怎么设计其结构？

尽管游戏的类型有很多种，市面上也有众多的 3D 游戏引擎，但绝大部分游戏引擎都得解决一个基本问题：抽象模拟一个 3D 游戏世界。根据基本的图形学知识，我们知道，为了展示这个世界，我们需要一个个带着 “变换” 的“游戏对象”，接着让它们父子嵌套以表现更复杂的结构。本质上，其他的物理模拟，游戏逻辑等功能组件，最终目的也只是为了操作这些“游戏对象”。  
这件事，在 Unity 那里就直接成了 “GameObject” 和“Component”；在 Cocos2dx 那里是一个个的“CCNode”，操纵部分直接内嵌在了 CCNode 里面；那么在 UE4 的眼中，它是怎么看待游戏的 3D 世界的？

## 创世记

## UE 创世，万物皆 UObject，接着有 Actor。

### UObject:

起初，UE 创世，有感于天地间 C++ 原始之气一片混沌虚无，便撷取凝实一团 C++ 之气，降下无边魔力，洒下秩序之光，便为这个世界生成了坚实的土壤 UObject，并用 UClass 一一为此命名。

![[fdaabac496005895c71298f5274b99db_MD5.png]]

  
藉着 UObject 提供的元数据、反射生成、GC 垃圾回收、序列化、编辑器可见，Class Default Object 等，UE 可以构建一个 Object 运行的世界。（后续会有一个大长篇深挖 UObject）

### Actor:

世界有了土壤之后，但还少了一些生动色彩，如果女娲造人一般，UE 取一些 UObject 的泥巴，派生出了 Actor。在 UE 眼中，整个世界从此了有了一个个生动的 “演员”，众多的“演员” 们，一起齐心协力为观众上演一场精彩的游戏。  

![[36b2d8ae7a110d280d3b465031def26c_MD5.png]]

脱胎自 Object 的 Actor 也多了一些本事：Replication（网络复制）,Spawn（生生死死），Tick(有了心跳)。  
Actor 无疑是 UE 中最重要的角色之一，组织庞大，最常见的有 StaticMeshActor, CameraActor 和 PlayerStartActor 等。Actor 之间还可以互相 “嵌套”，拥有相对的“父子” 关系。

**思考：为何 Actor 不像 GameObject 一样自带 Transform？**  
我们知道，如果一个对象需要在 3D 世界中表示，那么它必然要携带一个 Transform matrix 来表示其位置。关键在于，在 UE 看来，Actor 并不只是 3D 中的 “表示”，一些不在世界里展示的“不可见对象” 也可以是 Actor，如 AInfo(派生类 AWorldSetting,AGameMode,AGameSession,APlayerState,AGameState 等)，AHUD,APlayerCameraManager 等，代表了这个世界的某种信息、状态、规则。你可以把这些看作都是一个个默默工作的灵体 Actor。所以，Actor 的概念在 UE 里其实不是某种具象化的 3D 世界里的对象，而是世界里的种种元素，用更泛化抽象的概念来看，小到一个个地上的石头，大到整个世界的运行规则，都是 Actor.  
当然，你也可以说即使带着 Transform，把坐标设置为原点，然后不可见不就行了？这样其实当然也是可以，不过可能因为 UE 跟贴近 C++ 一些的缘故，所以设计哲学上就更偏向于 C++ 的哲学 “不为你不需要的东西付代价”。一个 Transform 再加上附带的逆矩阵之类的表示，内存占用上其实也是挺可观的。要知道 UE 可是会抠门到连 bool 变量都要写成 uint bPending:1; 位域来节省一个字节的内存的。  
换一个角度讲，如果把带 Transform 也当成一个 Actor 的额外能力可以自由装卸的话，那其实也可以自圆其说。经过了 UE 的权衡和考虑，把 Transform 封装进了 SceneComponent, 当作 RootComponent。但在权衡到使用的便利性的时候，大部分 Actor 其实是有 Transform 的，我们会经常获取设置它的坐标，如果总是得先获取一下 SceneComponent，然后再调用相应接口的话，那也太繁琐了。所以 UE 也为了我们直接提供了一些便利性的 Actor 方法，如 (Get/Set)ActorLocation 等，其实内部都是转发到 RootComponent。

```
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

世界纷繁复杂，光有一种 Actor 可不够，自然就需要有各种不同技能的 Actor 各司其职。在早期的远古时代，每个 Actor 拥有的技能都是与生俱有，只能父传子一代代的传下去。随着游戏世界的越来越绚丽，需要的技能变得越来越多和频繁改变，这样一组合，唯出身论的 Actor 数量们就开始爆炸了，而且一个个也越来越胖，最后连 UE 这样的神也管理不了了。终于，到了第 4 个纪元，UE 窥得一丝隔壁平行宇宙 Unity 的天机。下定决心，让 Actor 们轻装上阵，只提供一些通用的基本生存能力，而把众多的 “技能” 抽象成了一个个 “Component” 并提供组装的接口，让 Actor 随用随组装，把自己武装成一个个专业能手。

看见 UActorComponent 的 U 前缀，是不是想起了什么？没错，UActorComponent 也是基础于 UObject 的一个子类，这意味着其实 Component 也是有 UObject 的那些通用功能的。（关于 Actor 和 Component 之间 Tick 的传递后续再细讨论）

**下面我们来细细看一下 Actor 和 Component 的关系：**  
TSet<UActorComponent*> OwnedComponents 保存着这个 Actor 所拥有的所有 Component, 一般其中会有一个 SceneComponent 作为 RootComponent。  
TArray<UActorComponent*> InstanceComponents 保存着实例化的 Components。实例化是个什么意思呢，就是你在蓝图里 Details 定义的 Component, 当这个 Actor 被实例化的时候，这些附属的 Component 也会被实例化。这其实很好理解，就像士兵手上拿着把武器，当我们拥有一队士兵的时候，自然就一一对应拥有了不同实例化的武器。但 OwnedComponents 里总是最全的。ReplicatedComponents，InstanceComponents 可以看作一个预先的分类。

一个 Actor 若想可以被放进 Level 里，就必须实例化 USceneComponent* RootComponent。但如果你光看代码的话，OwnedComponents 其实也是可以包容多个不同 SceneComponent 的，然后你可以动态获取不同的 SceneComponent 来当作 RootComponent，只不过这种用法确实不太自然，而且也得非常小心维护不同状态，不推荐如此用。在我们的直觉印象里，一个封装过后的 Actor 应该是一个整体，它能被放进 Level 中，拥有变换，这一整个整体的概念更加符合自然意识，所以我想，这也是 UE 为何要在 Actor 里一一对应一个 RootComponent 的原因。

再来说说 Component 下面的家族（为了阐明概念，只列出了最常见的）：

![[dd7a997f259b317110c12031d2bfc994_MD5.png]]

ActorComponent 下面最重要的一个 Component 就非 SceneComponent 莫属了。SceneComponent 提供了两大能力：一是 Transform，二是 SceneComponent 的互相嵌套。

![[6ed5ef4dbe67c9b5a2db08151d8bc1a0_MD5.jpg]]

  
**思考：为何 ActorComponent 不能互相嵌套？而在 SceneComponent 一级才提供嵌套？**  

首先，ActorComponent 下面当然不是只有 SceneComponent，一些 UMovementComponent，AIComponent，或者是我们自己写的 Component，都是会直接继承 ActorComponent 的。但很奇怪的是，ActorComponent 却是不能嵌套的，在 UE 的观念里，好像只有带 Transform 的 SceneComponent 才有资格被嵌套，好像 Component 的互相嵌套必须和 3D 里的 transform 父子对应起来。  
老实说，如果让我来设计 Entity-Component 模式，我很可能会为了通用性而在 ActorComponent 这一级直接提供嵌套，这样所有的 Component 就与生俱来拥有了组合其他 Component 的能力，灵活性大大提高。但游戏引擎的设计必然也经过了各种权衡，虽然说架构上显得并不那么的统一干净，但其实也大大减少了被误用的机会。实体组件模式推崇的 “组合优于继承” 的概念确实很强大，但其实同时也带来了一些问题，如 Component 之间如何互相依赖，如何互相通信，嵌套过深导致的接口便利损失和性能损耗，真正一个让你随便嵌套的组件模式可能会在使用上更容易出问题。  
从功能上来说，UE 更倾向于编写功能单一的 Component（如 UMovementComponent）, 而不是一个整合了其他 Component 的大管家 Component（当然如果你偏要这么干，那 UE 也阻止不了你）。  
而从游戏逻辑的实现来说，UE 也是不推荐把游戏逻辑写在 Component 里面，所以你其实也没什么机会去写一个很复杂的 Component.

**思考：Actor 的 SceneComponent 哲学**  
很多其他游戏引擎，还有一种设计思路是 “万物皆 Node”。Node 都带变换。比如说你要设计一辆汽车，一种方式是车身作为一个 Node,4 个轮子各为车身的子 Node，然后移动父 Node 来前进。而在 UE 里，一种很可能的方式就变成，汽车是一个 Actor，车身作为 RootComponent，4 个轮子都作为 RootComponent 的子 SceneComponent。请读者们细细体会这二者的区别。两种方式都可以实现出优秀的游戏引擎，只是有些理念和侧重点不同。  
从设计哲学上来说，其实你把万物看成是 Node，或者是 Component，并没有什么本质上的不同。看作 Node 的时候，Node 你就要设计的比较轻量廉价，这样才能比较没有负担的创建多个，同理 Component 也是如此。Actor 可以带多个 SceneComponent 来渲染多个 Mesh 实体，同样每个 Node 带一份 Mesh 再组合也可以实现出同样效果。  
个人观点来说，关键的不同是在于你是怎么划分要操作的实体的粒度的。当看成是 Node 时，因为 Node 身上的一些通用功能（事件处理等），其实我们是期望着我们可以非常灵活的操作到任何一个细小的对象，我们希望整个世界的所有物体都有一些基本的功能（比如说被拾取），这有点完美主义者的思路。而注重现实的人就会觉得，整个游戏世界里，有相当大一部分对象其实是不那么动态的。比如车子，我关心的只是整体，而不是细小到每一个车轱辘。这种理念就会导成另外一种设计思路：把要操作的实体按照功能划分，而其他的就尽量只是最简单的表示。所以在 UE 里，其实是把 5 个薄薄的 SceneComponent 表示再用 Actor 功能的盒子装了起来，而在这个盒子内部你可以编写操作这 5 个对象的逻辑。换做是 Node 模式，想编写操作逻辑的话，一般就来说就会内化到父 Node 的内部，不免会有逻辑与表现掺杂之嫌，而如果 Node 要把逻辑再用组合分离开的话，其实也就转化成了某种 ScriptComponent。

**思考：Actor 之间的父子关系是怎么确定的？**  
你应该已经注意到了 Actor 里面的 TArray<AActor*> Children 字段，所以你可能会期望看到 Actor:AddChild 之类的方法，很遗憾。在 UE 里，Actor 之间的父子关系却是通过 Component 确定的。同一般的 Parent:AddChild 操作原语不同，UE 里是通过 Child:AttachToActor 或 Child:AttachToComponent 来创建父子连接的。

```
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

```
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

```
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

```
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

## 后记

花了这么多篇幅，才刚刚讲到 Actor 和 Component 这两个最基本的整体设计，而关于 Actor,Component 生命周期，Tick，事件传递等机制性的问题，还都没有展开。UE 作为从 1 代至今 4 代，久经磨练的一款成熟引擎，GamePlay 框架部分其实也就不到十个类，而这些类之间怎么组织，为啥这么设计，有什么权衡和考虑，我相信这里面其实是非常有讲究的。如果是 UE 的总架构师来讲解的话，肯定能有非常多的心得体会故事。而我们作为学习者，也应该尽量去体会琢磨它的用心，一方面磨练我们自己的架构设计能力，一方面也让我们更能掌握这个游戏的引擎。  
从此篇开始，会循序渐进的探讨各个部分的结构设计，最后再从整体的框架上讨论该结构的优劣点。

上篇：[《InsideUE4》基础概念](https://zhuanlan.zhihu.com/p/22814098)

下篇：[《InsideUE4》GamePlay 架构（二）Level 和 World](https://zhuanlan.zhihu.com/p/22924838)

## 引用

1.  [Unreal Engine 4 Terminology](https://docs.unrealengine.com/latest/INT/GettingStarted/Terminology/index.html)

1.  [Gameplay Framework Quick Reference](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/QuickReference/index.html)

_UE4.14_

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**