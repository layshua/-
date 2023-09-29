## 引言  

上文谈到 Actor 和 Component 的关系，UE 利用 Actor 的概念组成一片游戏对象森林，并利用 Component 组装扩展 Actor 的能力，让世界里拥有了形形色色的 Actor 们，拥有了自由表达 3D 世界的能力。  
那么，这些 Actor 们，到底是怎么组织起来的呢？

既然提到了世界，我们的直觉反应是采用一个 "World" 对象来包容所有的 Actor 们。但是当游戏的虚拟世界非常巨大时，这种方式就捉襟见肘了。首先，目前虽然 PC 的性能日益强大，但是依然内存也限制了不能一下子加载进所有的游戏资源；其次，因为玩家的活动和可见范围有限，为了最优性能，把即使是很远的跟玩家无关的对象也考虑进来也明显是不明智的。所以我们需要一种更细粒度的概念来划分世界。

不同的游戏引擎们，看待这个过程的角度和理念也不一样。Cocos2dx 会认为游戏世界是由 Scene 组成的，Scene 再由一个个 Layer 层叠表现，然后再有一个 Director 来导演整个游戏。Unity 觉得世界也是由 Scene 组成的，然后一个 Application 来扮演上帝来 LoadLevel，后来换成了 SceneManager。其他的，有的会称为关卡（Level）或地图（map）等等。而 UE 中把这种拆分叫做关卡（Level），由一个或多个 Level 组成一个 World。  
不要觉得这种划分好像很随意，只是个名字不同而已。实际上一个游戏引擎的 “世界观” 关系到了一整串后续的内容组织，玩家的管理，世界的生成，变换和毁灭。游戏引擎内部的资源的加载释放也往往都是和这种划分（Level）绑定在一起的。

## Level

在 UE 的世界中，我们之前已经有了空气（C++）, 土壤（UObject），物件（Actor）。而现在 UE 又施展神力创建了一片片大陆（Level），在这片大陆上（.map 文件），Actor 们秩序井然，各种地形拔地而起，植被繁茂，天空雾云缭绕，圣光普照，这也是玩家们降生开始精彩冒险的地方。  

![[72477eba63e36061f9a9b028f933c3d9_MD5.png]]

可以从 ULevel 的前缀 U 看出来 Level（大陆）也确实是继承于 UObject（土壤）的。那既然同属于 Object 下面的各 Actor 们都拥有了一定的智能能力（支持蓝图脚本），Level 自然也得体现出大地的意志，所以默认带了一个土地公（ALevelScriptActor），允许我们在关卡里编写脚本，可以对本关卡里的所有 Actor 通过名字呼之则来，关卡蓝图实际上就代表着该片大陆上的运行规则。  
在 Level 已经有了管理者之后，一开始大家都挺满意，但渐渐的就发现，好像各个 Level 需要的功能好像都差不多，都是修改一下光照，物理等一些属性。所以为了方便起见，UE 便给每一个 Level 也都默认配了一个书记官（Info），他一一记录着本 Level 的各种规则属性，在 UE 需要的时候便负责相告。更重要的是，在 Level 需要有其他管理人员一起协助的时候，他也记录着 “游戏模式” 的名字来让 UE 可以指派。  
前面我们说过，有一些 Actor 是不 “显示” 的（没有 SceneComponent），是不能 “摆放” 到 Level 里的，但是它依然可以在关卡里出力。其中一个家族系列就是 AInfo 和其之类。今天我们只简单介绍一下跟 Level 直接相关的一位书记官：AWorldSettings。  

![[5980f02a5ea5f4f706d7c33329d8f688_MD5.jpg]]

其实虽然名字叫做 WorldSettings，但其实只是跟 Level 相关，我猜可能是在上古时代，当时整个世界只有一块大陆，人们就以为当前的大陆就是整个世界，所以给这块大陆的设置就起名为 WorldSettings，后来等技术进步了，发现必须有其他大陆了，这个名字已经用得太多反而不好改了，就只好遗留下来了。当然也有可能是因为当 Level 被添加进 World 后，这个 Level 的 Settings 如果是主 PersistentLevel，那它就会被当作整个 World 的 WorldSettings。  
注意，Actors 里也保存着 AWorldSettings 和 ALevelScriptActor 的指针，所以 Actors 实际上确实是保存了所有 Actor。

**思考：为何 AWorldSettings 要放进在 Actors[0] 的位置？而 ALevelScriptActor 却不用？**

```
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

实际上通过这一段代码可知，Actors 们的排序依据是把那些 “非网络” 的 Actor 放在前面，而把 “网络可复制” 的 Actor 们放在后面，然后加一个起始索引标记 iFirstNetRelevantActor，相当于为网络 Actor 划分了一个缓存，从而加速了网络复制时的检测速度。AWorldSettings 因为都是静态的数据提供者，在游戏运行过程中也不会改变，不需要网络复制，所以也就可以一直放在前列，而如果再加个规则，一直放在第一个的话，也能同时把 AWorldSettings 和其他的前列 Actor 们再度区分开，在需要的时候也能加速判断。ALevelScriptActor 因为是代表关卡蓝图，是允许携带 “复制” 变量函数的，所以也有可能被排序到后列。

**思考：既然 ALevelScriptActor 也继承于 AActor, 为何关卡蓝图不设计能添加 Component？**  
观察到，平常我们在创建 Actor 的时候，我们蓝图界面是可以创建 Component 的。  
那为什么在关卡蓝图里，却不能这么做（没有提供该界面功能）？  
我虽然在图里标出了 Level 中拥有 ModelComponents，但那其实只是针对 BSP 应用的一个子集。通过源码发现，其实 UE 自己也是在 C++ 里往 ALevelScriptActor 添加 UInputComponent 来实现关卡蓝图可以响应事件。

```
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
那么好好想想，为啥 UE 要给你这么一个关卡蓝图界面呢？  
在此，我也只能进行一番猜测，ALevelScriptActor 作为一个特化的 Actor, 却把 Components 列表界面给隐藏了，说明 UE 其实是不希望我们去复杂化关卡构成的。  
假设说 UE 开放了关卡 Component，那么我们在创建组件时就必然要考虑一个问题：哪些是 ActorComponent，哪些是 LevelComponent，再怎么 ALevelScriptActor 本质是个 Actor，但 Level 的概念还是要突出，ALevelScriptActor 的 Actor 本质是要隐藏的。所以用户就会多一些心智负担，可能混淆。而如果像这样不开放，大家的思路就都转向先创建个 Actor，然后再往之上添加 component，思路会比较统一清晰。  
再之，从游戏逻辑的组织上来说，Level 其实更应该表现为一个 Actor 的容器。UE 其实也是不鼓励在 Level 里编写太复杂的逻辑的。所以才接着会有了之后的 GameMode,Controller 那些真正的逻辑控制类（后续会再细讨论）。  
所以游戏引擎也并不是说最大化的暴露一切功能给你就是最好的，有时候选择太多了反而容易出错。在这一点上，我觉得 UE 很好的保持了克制，为我们提供了一个优秀的清晰的不易出错的框架，同时也对高阶用户保留了灵活性。

## World

终于，到了把大陆们（Level）拼装起来的时候了。可以用 SubLevel 的方式：

![[3d39261b1e08684bf04385aee979521c_MD5.png]]

也支持 WorldComposition 的方式自动把项目里的所有 Level 都组合起来，并设置摆放位置：

![[5518cb072e076559c9320782e502e763_MD5.jpg]]

具体摆放的操作和技巧并不是本文的重点。简单本质来说，就是一个 World 里有多个 Level，这些 Level 在什么位置，是在一开始就加载进来，还是 Streaming 运行时加载。  
UE 里每个 World 支持一个 PersistentLevel 和多个其他 Level：  

![[db0281595e0687c54921e249851819e0_MD5.png]]

Persistent 的意思是一开始就加载进 World，Streaming 是后续动态加载的意思。Levels 里保存有所有的当前已经加载的 Level，StreamingLevels 保存整个 World 的 Levels 配置列表。PersistentLevel 和 CurrentLevel 只是个快速引用。在编辑器里编辑的时候，CurrentLevel 可以指向其他 Level，但运行时 CurrentLevel 只能是指向 PersistentLevel。

**思考：为何要有主 PersistentLevel？**  
首先，World 至少得有一个 Level，就像你也得先出生在一块大陆上才可以继续谈起去探索别的新大陆。所以这块玩家出生的大陆就是主 Level 了。当然了，因为我们也可以同时配置别的 Level 一开始就加载进来，其实跟 PersistentLevel 是差不多等价的，但再考虑到另一问题：Levels 拼接进 World 一起之后，各自有各自的 worldsetting，那整个 World 的配置应该以谁的为主？

```
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

可以看出，World 的 Settings 也是以 PersistentLevel 为主的，但这也并不意味着其他 Level 的 Settings 就完全没有作用了，本篇也无法一一列出所有配置选项来说明，简单来说，就是需要在整个世界范围内起作用的配置选项（比如 VR 的 WorldToMeters，KillZ，WorldGravity 其他大部分都是）就是需要从主 PersistentLevel 的配置中提取。而一些配置选项可以在单独 Level 中起作用的，比如在编辑 Level 时的光照质量配置就是一个个 Level 单独的，目前这种配置很少，但可能以后也会增加。在这里只是阐明一个为主其他为辅的 Level 配置系统。

**思考：Levels 们的 Actors 和 World 有直接关系吗？**  
当别的 Level 被添加进当前 World 之后，我们能直接在 WorldOutliner 里看到其他 Level 的 Actor 们。  

![[f574ef7fcefa7b1d4986b1ea6b322ac0_MD5.png]]

但这并不代表着 World 直接引用了 Level 里的 Actor 们。TActorIteratorBase（World 的 Actor 迭代器）内部的实现也只是在遍历 Levels 来获得所有 Actor。当然 World 为了更快速的操作 Controllers 和 Pawn 也都保存了引用。但 Levels 却共享着 World 的一个 PhysicsScene，这也意味着 Levels 里的 Actors 的物理实体其实都是在 World 里的，这也好理解，毕竟物理的碰撞之类的当然要是全局的了。再说到导航，World 在拼接 Level 的时候，也是会同时把两个 Level 的导航网格给 “拼接” 起来的。当然目前还不是深入细节的时候，现在只要从大局上明白 World-Level-Actor 的关系。

**思考：为什么要在 Level 里保存 Actors，而不是把所有 Map 的 Actors 配置都生成在 World 一个总 Actors 里？**  
这肯定也是一种实现方式，好处是把整个 World 看成一个整体，所有的 actors 都从属于 world，这样就不存在 Level 边界，可以更整体的处理 Actors 的作用范围和判定问题，实现上也少了拼接导航等步骤。当然坏处也是模糊了 Level 边界，这样在加载进一个 Level 之后，之后再动态释放，就需要再重新再从整体中抽离出部分来释放，这个筛选过程也会产生比较大的损耗。试着去理解 UE 的权衡，应该是尽量的把损耗平摊（这里是把 Level 加载释放的损耗尽量减小），才不会产生比较大的帧率波动，让玩家感觉到卡帧。

## 总结

Level 作为 Actor 的容器，同时也划分了 World，一方面支持了 Level 的动态加载，另一方面也允许了团队的实时协作，大家可以同时并行编辑不同的 Level。一般而言，一个玩家从游戏开始到结束，UE 会创造一个 GameWorld 给玩家并一直存在。玩家切换场景或关卡，也只是在这个 World 中加载释放不同的 Level。既然 Level 拥有了管理者（LevelScriptActor），玩家可以编写特定关卡的逻辑，那么我们能否对 World 这种层次编写逻辑呢？答案是肯定的，不过本文篇幅有限，敬请期待下篇。

上篇：[《Inside UE4》GamePlay 架构（一）Actor 和 Component](https://zhuanlan.zhihu.com/p/22833151)

下篇：[《InsideUE4》GamePlayer 架构（三）WorldContext，GameInstance，Engine](https://zhuanlan.zhihu.com/p/23167068)

_UE4.14_

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**