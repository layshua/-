# 5 Controller
那一天  
Pawn 又回想起了  
被 Controller 所支配的恐惧

## 引言

如上文所述，UE 从 Actor 中分化了一些专门可供玩家 “控制” 的 Pawn，那我们这篇就专门来谈谈该怎么个控制法！  
所谓的控制，本质指的就是我们游戏的业务逻辑。比如说玩家按 A 键，角色自动找一个最近的敌人并攻击，这个自动寻找目标并攻击的逻辑过程，就是我们所谈的控制。  
Note1：重申一下，Controller 特别是 PlayerController，跟网络，AI 和 Input 的关系都非常的紧密，目前都暂且不讨论，留待各自模块章节再叙述。

## MVC

不管是游戏，还是其他 App，Web 或 Server 等，本质上都是程序，所以也都是或多或少需要一些程序逻辑。从 1843 年拜伦的女儿 Ada Lovelace 用穿孔机编写第一个程序开始，到 2016 的今天我们能方便地用蓝图连线组织程序逻辑，应该归功于一代代软件工程师们孜孜不倦的探索。时代在发展，技术在进步，软件也愈趋于复杂多变，很多软件的庞大也已经超越了个人的理解容量极限（UE？），因此我们就越来越需要设计方法来让我们可管理庞大的复杂度。几十年的迭代，旧的模型被放弃，新的模型被提出验证，工程师们在这过程中总结积累出了一些设计模式。最负有盛名的应该是 GOF 的《设计模式》，以及 MVC，MVP，MVVM 等。本文的重点不在于细谈论各种设计模式，如果有对设计模式不清楚的读者，请务必仔细去研究学习，因 UE 如此庞大的代码框架也是充斥着各种设计模式的应用，设计模式理解得越好，也越能理解 UE 的框架设计。  
言归正传，设计模式的本质就是抽象变化。如果依照纯朴的 "程序 = 数据 + 算法" 的结构来看，再算上用于用户显示和输入的界面，那么就得到 “程序 = 数据 + 算法 + 显示”。这三大基本块（数据，算法，显示）构成了程序的三大变化，而如何把这三者“+” 到一起，用的就是我们的种种设计框架模式。  
典型的，对于游戏：

*   “显示” 指的是游戏的 UI，是屏幕上显示的 3D 画面，或是手柄上的输入和震动，也可以是 VR 头盔的镜片和定位，是与玩家直接交互的载体；
*   “数据” 指的是 Mesh，Material，Actor，Level 等各种元素组织起来的内存数据表示；
*   “算法” 可以是各种渲染算法，物理模拟，AI 寻路，本文咱们就先暂时特指游戏开发者们编写的游戏业务逻辑。

抽象这三个变化，并归纳关系，就是典型的 MVC 模式了：

![[aa98f2f1365ec787c09249b5f0bc7958_MD5.png]]

有些人可能会说 MVC 是 UI 专用的模式，如 IOS 的 MVC 或 WPF 的 MVVM，也或者说因为游戏的类型千差万别所以一个通用的框架并不能都适用，因此就有一点点想要 “返璞归真” 的意味，觉得游戏引擎只需要提供一个基本的渲染框架，其他的逻辑框架不需要设计复杂，开发者们可自行根据游戏类型再设计。这种观点有一定的道理，对于简单的游戏或 Demo，确实也还不到需要 “设计” 的地步；而对于复杂大型的游戏，需要的架构知识也确实远不是 MVC 这么简单。但缺点在于，说这话的人要嘛就已经是架构高手，各种设计模式信手拈来，早已经到了无招胜有招的地步；要嘛就是回避了问题，游戏也是软件，软件的固有复杂度摆在那里，总得需要个办法去解决，今天如果我们不是在探讨尝试用 MVC 模式去掌控它，也是在谈一个别的名字的模式。在我看来，一个好的游戏引擎，应该是能尽力的帮助用户，并减少麻烦。MV 当然也有它的缺陷和不足，所以我们应该研究的是 UE 为何选择了 MVC，有什么优点缺点，我们怎么利用和规避，让 UE 的 Controller 们尽责的为我们服务，少造成麻烦。  
对于简单的游戏或者引擎来说，有时并不需要把这三者分的很清，如 Cocos2dx 就没有 Controller，它的 MVC 就是混杂在一起，因为代码量少所以也还算勉强能凑合；Unity 的 MonoBehavior 其实也相当于把 MC 放在了一起，用得方便的同时也得小心太顺手了出现组件之间互相网状引用一团乱麻的情况；UE 在这个问题上的思考就有些一脉相承，既然 Actor 们形形色色，我们之前也谈过甚至有 AInfo 这种书记官，那为何不让一些 Actor 专门来承载逻辑呢？于是，Actor 再度分化出 Controller。下面我们就来一一介绍 Actor 旗下 Controller 家族的指挥官们。

## AController

虽然我在之前已经一再的剧透过 AController 是继承自 AActor 的一个子类，但是为了更好理解思考 UE 里的 Controller 机制，请先把脑袋放空，也别去偷看 UE 里的源码，像张无忌一样暂时忘记 AController 这回事，问自己一个问题：如果我想实现一种机制去控制游戏里的 Actor，该怎么设计？  
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
关联 Pawn 的能力，有 Possess 和 UnPossess，源码里也有 PawnPendingDestroy 等这些函数（未一一列出）；GameMode 中也保存着 AIControllerClass 和 PlayerControllerClass 的配置，用于在适当的时候 Spawn 出 Controller；继承于 Actor 也就有了 EnableInput 和 Tick；Controller 本身还可以继续派生下去（如 AIController 和 PlayerController），也可以容纳 Components；也带着一个 SceneComponent 所以可以摆放在世界中；自身也可以添加成员变量来记忆存储游戏状态；自身也有一个 FName StateName（Playing、Spectating、Inactive），切换自身的状态（运行，观察，非激活）；因为跟 Pawn 是平级的关系，只在运行的时候引用关联，所以对彼此独立存在不做强制约束，提高了灵活性。一个 Pawn 自身上也可以配置策略：

```
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

**思考：Controller 和 Pawn 必须 1:1 吗？**  
观察 UE 实现里我们发现 Controller 里只是保存了一个 Pawn 指针，而不是数组，这和一开始希望的多对多关系有些出入。理想和现实总是有差距，一个愿景落实到工程实践上也不免得有一些妥协。首先我们再来梳理理解一下这个 Possess(拥有 / 占用) 的概念。一个 Controller 能灵活的 Possess/UnPossess 一个 Pawn，虽然一次只能控制一个，但在游戏中我们也可以在不同的 Pawn 中切换，比如操纵一个主角坐进然后控制一辆汽车，或者端起固定的机关枪扫射，这些功能琢磨一下其实只是涉及操作实体 Pawn 的变化。如果我们能妥善的用好 Pawn 和 Controller 的切换功能，大部分基本的游戏功能也是能够比较方便的实现的。那么有哪些是不太适合的呢？UE 官方其实也承认了，见 [Controller](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/Controller/index.html) 文档说明：

By default, there is a one-to-one relationship between Controllers and Pawns; meaning, each Controller controls only one Pawn at any given time. This is acceptable for most types of games, but may need to be adjusted as certain types of games - real-time strategy comes to mind - may require the ability to control multiple entities at once.

对于 RTS 这种需要一下子控制多个单位的游戏来说，这种 1v1 的关系确实比较僵硬，就需要在 Controller 里自己实现扩展一下，额外保存多个 Pawn，然后自己实现一些需要的控制实现，但总体上也只能说得绕一下，也算不上特别复杂，所以就也不能说 UE 做不了某一些类型的游戏，Epic 是个游戏引擎公司，卖的毕竟是个通用游戏引擎。  
OK，那 UE 为何不实现成多对多呢？我觉得理由往往很简单，就是想保持一定的简单。游戏引擎的每个模块的设计，甚至函数接口的设计，无时无刻不在权衡决定。太简单了概念清晰用起来方便但是灵活扩展力不足，太灵活扩展无限了往往也会让人无从适从容易出错。当前 1:1 的时候，我们的脑袋逻辑很清晰，我们可以在 Controller 里直接 GetPawn，也可以在 Pawn 中 GetController，都非常方便。调试逻辑 Bug 的时候，我们也能很快找到查错的目标。而对比想象，如果是 M：N，灵活性是满满了，但是你能轻易的说出当前 Pawn 是被哪些 Controller 控制吗？你也得时时记着这个 Controller 当前控制了哪些 Pawn。OMG！这些 Pawn 和 Controller 多对多的构成了网状结构，项目越庞大复杂，这张网也越能套住你。再从另一个方面说，一旦提供了这种多对多的直接支持，以我们人类的性格，免费现成的东西，我们总是倾向于去找机会能用上它，而不是去琢磨到底应不应该用。所以一旦就这么直接提供了，对于刚入门的新手，压根就没什么指引，怎么来好像都可以，就非常容易收不住把项目逻辑关系搞得不必要的复杂。所以以后 UE 就算想在这一方面优化加强，应该也会比较克制。  
索性再聊开一些，我们用 Unity 来做一下对比。Unity 就是 GameObject+Component，你自己组合去吧，非常的灵活自由，也不做什么限制，但造成的后果就是常常各种 Component 互相引用来引用去，网状互联一团乱麻。另外几乎每个人都可以在上面搞出一套游戏系统出来，互相之间都是自成一派。所以经常网上就会有各种帖子问怎么在 Unity 中实现 MVC 模式的，也有分析炉石传说游戏逻辑框架的。Unity 当然是个好引擎，目前来说热度也是比 UE 要高一些，但我们也不能因为它火用得人多，就权威崇拜从众的认为 Unity 各个方面都比别的引擎好。设计架构游戏的时候，工程师们要抵挡住灵活性的诱惑，保持克制往往是更难得珍贵的美德。要认识到，引擎的终极目的是方便人使用的，我们程序员往往很容易太沉迷于程序功能的灵活强大，而疏忽了易用性鲁棒性等社会工程需求。

**思考：为何 Controller 不能像 Actor 层级嵌套？**  
我们都知道 Actor 可以藉着身上的 SceneComponent 互相嵌套。那么 AController 同样也是 Actor，为何不也实现这么一个父子机制？从功能上来说，一个 Controller 可以有子 Controllers，听起来也是非常灵活强大啊。但是冷静想一下，Controller 表达的 “控制” 的概念，所以在这里你实际上想要表达的是一种 “控制” 互相嵌套的概念，感觉又给 “控制” 给分了层，有 “大控制”，也有“小控制”，但是“控制” 的“大小”又是个什么概念呢？我们应该怎么划分控制的大小？“控制”本质上来说就是一些代码，不管怎么设计，目的都是用来表达游戏游戏逻辑的。而针对游戏逻辑的复杂，怎么更好的管理组织逻辑代码，我们有状态机，分层状态机，行为树，GOAL（目标导向），甚至你还能搞些神经网络遗传算法机器学习啥的。所以在我们已经有这么多工具的基础上，徒增复杂性是很危险的做法。如果有必要，也可以把 Controller 本身再当作其他 AI 算法的容器，所以就没必要在对象层次上再做文章了。

**思考：Controller 可以显示吗？**  
既然 Actor 本身可以带着 Mesh 组件来渲染显示，那 Controller 可不可以呢？是不是 Controller 都是不可见的？这个答案可说是也可以说不是，因为 Controller 本身确实就是一个特殊点的 Actor 而已，你依然可以在 Controller 中添加 Mesh 组件，添加别的子 Actor 等，所以从这个方面说 Controller 是有可以渲染显示的能力的。但是一个控制者毕竟只是表达一个逻辑的概念，所以为了分工明确，UE 就干脆在 Controller 的构造函数里把自己给隐藏了：

```
bHidden = true;
#if WITH_EDITORONLY_DATA
    bHiddenEd = true;
#endif // WITH_EDITORONLY_DATA
```

事了拂衣去，深藏功与名。为了验证我的说法，读者你可以亲自在 PlayController 下挂一些 Cube 之类的 Actor，然后在源码层把这两个值改为 false，重新编译运行看下结果，看能否正确显示出来，这里我就不贴图了，留给读者验证，很好玩的哦。  

**思考：Controller 的位置有什么意义？**  
既然 Controller 本身只是控制者，那它在场景中的位置和移动有什么意义吗？Controller 为何还需要个 SceneComponent? 意义在于如果 Controller 本身有位置信息，就可以利用该信息更好的控制 Pawn 的位置和移动。  
首先说下 Controller 的 Rotation，这个比较好理解一点，如果我想让我的 Pawn 和 Controller 保持旋转朝向一致，因为是 Controller 作主控制 Pawn 的关系，所以 Controller 就得维护自己的 Rotation。再来说位置，如果 Controller 有自己的位置，这样在 Respawn 重新生成 Pawn 的时候，你就可以选择在当前位置创建。因此为了自动更新 Controller 的位置，UE 还提供了一个 bAttachToPawn 的开关选项，默认是关闭的，UE 不会自动的更新 Controller 的位置信息；而如果打开，就会把 Controller 附加到 Pawn 的子节点里面去，让 Controller 跟随 Pawn 来移动。你可以把这两种模式想象成一种是上帝视角在千里之外心电感应控制 Pawn，另一种是骑在 Pawn 肩上来指挥。  
当然如果这个 Controller 确实只是纯朴的逻辑控制的话（如 AIController），那确实位置也没什么意义。所以 UE 甚至还隐藏了 Controller 的一些更新位置的接口，尽量避免让人手动去操纵：

```
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

**思考：哪些逻辑应该写在 Controller 中？**  
如同当初我们在思考 Actor 和 Component 的逻辑划分一样，我们也得要划分哪些逻辑应该放在 Pawn 中，哪些应该放在 Contrller 中。上文我们也说过，Pawn 也可以接收用户输入事件，所以其实只要你愿意，你甚至可以脱离 Controller 做一个特立独行的 Pawn。那么在那些时候需要 Controller？哪些逻辑应该由 Controller 掌管呢？可以从以下一些方面考虑：

*   从概念上，Pawn 本身表示的是一个 “能动” 的概念，重点在于 “能”。而 Controller 代表的是动到“哪里” 的概念，重点在于“方向”。所以如果是一些 Pawn 本身固有的能力逻辑，如前进后退、播放动画、碰撞检测之类的就完全可以在 Pawn 内实现；而对于一些可替换的逻辑，或者智能决策的，就应该归 Controller 管辖。
*   从对应上来说，如果一个逻辑只属于某一类 Pawn，那么其实你放进 Pawn 内也挺好。而如果一个逻辑可以应用于多个 Pawn，那么放进 Controller 就可以组合应用了。举个例子，在战争游戏中，假设说有坦克和卡车两种战车（Pawn），只有坦克可以开炮，那么开炮这个功能你就可以直接实现在坦克 Pawn 上。而这两辆战车都有的自动寻找攻击目标功能，就可以实现在一个 Controller 里。
*   从存在性来说，Controller 的生命期比 Pawn 要长一些，比如我们经常会实现的游戏中玩家死亡后复活的功能。Pawn 死亡后，这个 Pawn 就被 Destroy 了，就算之后再 Respawn 创建出来一个新的，但是 Pawn 身上保存的变量状态都已经被重置了。所以对于那些需要在 Pawn 之外还要持续存在的逻辑和状态，放进 Controller 中是更好的选择。

## APlayerState

我们上文提到过 Controller 希望也能有一些记忆，保存住一些游戏状态。那么到底应该怎么保存呢？AController 自身当然可以添加成员变量来保存，这些变量也可以网络复制，一般来说也够用。但是终究还是遗忘了一个最重要的数据状态。整个游戏世界构建起来就是为了玩家服务的，而玩家在游戏过程中，肯定要存取产生一些状态。而 Controller 作为游戏业务逻辑最重要的载体，势必要和玩家的状态打交道。所以 Controller 如果可以动态存取玩家的状态就会大为方便了。因此我们会在 Controller 中见到：

```
/** PlayerState containing replicated information about the player using this controller (only exists for players, not NPCs). */
    UPROPERTY(replicatedUsing=OnRep_PlayerState, BlueprintReadOnly, Category="Controller")
    class APlayerState* PlayerState;
```

而 APlayerState 的继承体系是：  

![[9e9af6cb3406c49ece9b425c4996a5c7_MD5.png]]

至于为啥 APlayerState 是从 AActor 派生的 AInfo 继承下来的，我们聪明的读者相信也能猜得到了，所以也就不费口舌论证了。无非就是贪图 AActor 本身的那些特性以网络复制等。而 AInfo 们正是这种不爱表现的纯数据书呆子们的大本营。而这个 PlayerState 我们可以通过在 GameMode 中配置的 PlayerStateClass 来自动生成。  
注意，这个 APlayerState 也理所当然是生成在 Level 中的，跟 Pawn 和 Controller 是平级的关系，Controller 里只不过保存了一个指针引用罢了。注释里说的 PlayerState 只为 players 存在，不为 NPC 生成，指的是 PlayerState 是跟 UPlayer 对应的，换句话说当前游戏有多少个真正的玩家，才会有多少个 PlayerState，而那些 AI 控制的 NPC 因为不是真正的玩家，所以也不需要创建生成 PlayerState。但是 UE 把 PlayerState 的引用变量放在了 Controller 一级，而不是 PlayerController 之中，说明了其实 AIController 也是可以设置读取该变量的。一个 AI 智能能够读取玩家的比分等状态，有了更多的信息来作决策，想来也没有什么不对嘛。  
Controller 和网络的结合很紧密，很多机制和网络也非常强关联，但是在这里并不详细叙述，这里先可以单纯理解成 Controller 也可以当作玩家在服务器上的代理对象。把 PlayerState 独立构成一个 Actor 还有一个好处，当玩家偶尔因网络波动断线，因为这个连接不在了，所以该 Controller 也失效了被释放了，服务器可以把对应的该 PlayerState 先暂存起来，等玩家再紧接着重连上了，可以利用该 PlayerState 重新挂接上 Controller，以此提供一个比较顺畅无缝的体验。至于 AIController，因为都是运行在 Server 上的，Client 上并没有，所以也就无所谓了。

**思考：哪些数据应该放在 PlayerState 中？**  
从应用范围上来说，PlayerState 表示的是玩家的游玩数据，所以那些关卡内的其他游戏数据就不应该放进来（GameState 是个好选择），另外 Controller 本身运行需要的临时数据也不应该归 PlayerState 管理。而玩家在切换关卡的时候，APlayerState 也会被释放掉，所有 PlayerState 实际上表达的是当前关卡的玩家得分等数据。这样，那些跨关卡的统计数据等就也不应该放进 PlayerState 里了，应该放在外面的 GameInstance，然后用 SaveGame 保存起来。

## 总结

在游戏里，如果要评劳模，那 Controller 们无疑是最兢兢业业的，虽然有时候蛮横霸道了一些，但是经常工作在第一线，下面的 Pawn 们常常智商太低，上面的 Level，GameMode 们又有点高高在上，让他们直接管理数量繁多的 Pawn 们又有点太折腾，于是事无巨细的真正干那些脏活累活的还得靠 Controller 们。本文虽然没有在网络一块留太多笔墨，但是 Controller 也是同时作为联机环境中最重要的沟通渠道，身兼要职。  
回顾总结一下本文要点，UE 在 Pawn 这个层级演化构成了一个最基本和非常完善的 Component-Actor-Pawn-Controller 的结构：

![[49911dbb6255e84669230b4fba2d521e_MD5.png]]

通过分化出来后的 Actor 的互相控制，既充分利用了现有的机制功能，又提供了足够的灵活性，而且做的更改还很少，不用再设计额外另一套框架。读者朋友们，现在我们如果翻到第一小节，想想 UE 最初从 Object 分化出 Actor 的那一刻，是不是有很多感慨和感动呢？一个最初的很简单的游戏对象表示，慢慢演化派生充实起来，彼此之间通力配合，竟也能优雅的运转起来。

有时候架构的设计和搭建是一脉相承的，最初的时候选择了什么样的模型和骨架，后面再设计别的逻辑框架等其他模块，也基本上都得跟最初的设计配合着来。所以有时候往往也会发现，怎么感觉我架构设计的方案可选择数量并不多啊？其实是因为如果一开始铺垫的好，接下来的设计水到渠成自然而然，让你感觉不到用心设计的力气。UE 以 Actor 的视角来看待世间万物，自然得到的是一个 Actor 繁荣昌盛的世界；Unity 以 Component 来组装万物，得到的就是个各种插件组件组装出的世界；而如果如 Cocos2dx 一般万物都是 Node, 那么自然也会得到一棵挂满各种 Node 的世界之树。这也算是游戏引擎的基因吧。

本想着一篇介绍完 Controller、PlayerController 和 AIController 这三个对象，但是 Controller 本身是 UE 里极为重要的核心概念，自身的功能非常的丰富，牵扯的模块也比较多，因此想抽离阐述最核心的概念和功能并不是一件容易的事。花了这么长的篇幅，只讨论揣摩了 Controller 的设计过程和最基本的职责（还有输入网络等都没有解释），顺便先简单介绍了下 PlayerState 出场（PlayerState 实际上是跟 UPlayer 关联更大一些，PlayerController 等后续章节会继续讨论它），对于 PlayerController 和 AIController，目前也只是语焉不详的含糊带过。不过还是希望读者们能从中吸取到设计的营养，把握清楚概念了，才能更好的组织游戏逻辑，开发出更好的游戏。

本系列教程的一个重点也是尝试介绍引擎各种概念背后的考量，而不是单纯的叙述解释各个模块功能。笔者始终认为，只有我们愿意不吝口舌的去讨论，愿意耐下心来去思考学习，这些概念的领悟才会了然在心中。否则若只是单纯的介绍 Pawn 功能有 123，Controller 可以 ABC，相信读者在阅读完之后也并不会有什么深的印象，因为这些只是设计的结果，少了设计的过程。

上篇：[《InsideUE4》GamePlay 架构（四）Pawn](https://zhuanlan.zhihu.com/p/23321666?refer=insideue4)  

下篇我们将隆重介绍 Controller 家族中最耀眼的明星、上帝的宠儿：[《InsideUE4》GamePlay 架构（六）PlayerController 和 AIController](https://zhuanlan.zhihu.com/p/23649987)！

## 引用

1.  [Controller](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/Controller/index.html)

UE4 的版本更新实在太快，为了留下版本存照和供读者查证，以后在篇尾都会标注上本文研究使用的源码版本。以后不再特意做此声明。  
_UE 4.13.2_

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**