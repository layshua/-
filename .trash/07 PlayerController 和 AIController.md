# 6 PlayerController 和 AIController
PlayerController：你不懂，伴君如伴虎啊  
AIController：上来，我自己动

## 引言

上文我们谈到了 Component-Actor-Pawn-Controller 的结构，追溯了 AController 整个家族的崛起和身负的使命。本篇我们继续来探讨 Controller 家族中最为人所知的 PlayerController 和 AIController。  
作为一个 Controller，我们讨论的依然是该如何控制。我们已经知道了 Controller 可以 Possess 并控制 Pawn，但是 Controller 本身又是怎么驱动起来的呢？一个游戏里的控制角色大抵都可以分为两类：玩家和 AI。不管是单机游戏或者分屏多玩家，还是网络玩家联机对战，游戏都是为了玩家服务的，所以也必然会有一个或多个玩家，就算是如《山》那种纯看的游戏，也是有一个 “可观察不可动” 的玩家的。而 AI 的实体的数量就可以是零或者多个。  
**Note1：**依旧重申：输入、网络、AI 行为树等模块虽跟 PlayerController 和 AIController 关系紧密，但目前都暂且不讨论，留待各自模块章节再叙述。

## APlayerController

让咱们先从简单的单机游戏开始讨论吧，比如一款单机 FPS 游戏，这个游戏里已经用各种各样的 Actor 们构建完成了世界场景，你的主角和敌人 Pawn 们也都在整装待发，这个时候你思考这么一个问题，我该怎么玩这个游戏？壮丽的舞台已经准备好了，就等你入场了。先抛开具体的引擎而言，首先你需要能看见（拥有 Camera 和位置），其次你必须能响应输入（玩家按 WASD 你应该能接收到），然后你可以根据输入操控一些 Pawn（Possess 然后传递 Input），这样一个单机游戏中的简单玩家控制器就差不多了。一个游戏中只有一个 PlayerController，在不同的关卡中你可以使用不同的 PlayerController，但是同一时刻响应的只能是一个 PlayerController。  
插上多个手柄，咱们再拓展一下，比如像《街霸》那种单 PC 但是多玩家对抗或者协作的游戏。两个玩家可以分别用两个手柄，或者一个用键盘一个用鼠标，甚至是键盘上的不同区域，形式可以多种多样。这个时候如果依然只有一个 PlayerController，实现起来其实也是可行的，把两个手柄——所有的输入都由这个 PlayerController 来接收，然后在 PlayerController 内部再分别根据情况去处理不同的 Pawn。但是这种方式的缺点显然也在于很容易把玩家 1、2 的输入和控制混杂在一起，没有清晰的区分开。因此，为了支持这种情况，我们可以开始允许游戏中同时出现多个 PlayerController，每个 PlayerController 甚至都可以拥有自己的 Viewport（分屏或者不同窗口），这样我们通过配置，可以精确的路由手柄 1 的输入给玩家 1，各自的逻辑也很好的区分和复用。  
再插上网线继续，到了网游时代，我们的游戏就开始允许有多人联机对战了。玩家在自己的 PC 上控制的只是自己的本地的角色，而屏幕游戏里其他的玩家角色是由网线另一端的玩家控制的。为了更好的适应这种情况，我们就又得扩展一下 PlayerController 的概念，PlayerController 不仅能控制本地的 Pawn，而且还能 “控制” 远程的 Pawn（实际上是通过 Server 上的 PlayerController 控制 Server 上的 Pawn，然后再复制到远程机器上的 Pawn 实现的）。  
因此我们来看看 UE 里的 PlayerController：  

![[c8b2380d53809dfb6360496f8d954ce3_MD5.png]]

PlayerController 因为是直接跟玩家打交道的逻辑类，因此是 UE 里使用最多的类之一。UE4.13.2 版本里 1632 行的. h 文件和 4686 行的. cpp 文件，里面实现了很多的功能，初阅读起来往往深陷其中不得要领。但是在上述的分析了之后，我们也可以在其中大概归纳出几个模块：

*   Camera 的管理，目的都是为了控制玩家的视角，所以有了 PlayerCameraManager 这一个关联很紧密的摄像机管理类，用来方便的切换摄像机。PlayerController 的 ControlRotation、ViewTarget 等也都是为了更新 Camera 的位置。因为跟 Camera 的关系紧密，而 Camera 最后输出的是屏幕坐标里的图像，所以为了方便一些拾取的 HitResult 函数也都是实现在这里面。渲染章节会再详细介绍 UE 的摄像机管理。
*   Input 系统，包括构建 InputStack 用来路由输入事件，也包括了自己对输入事件的处理。所以包含了 UPlayerInput 来委托处理。
*   UPlayer 关联，既然顾名思义是 PlayerController，那自然要和 Player 对应起来，这也是 PlayerController 最核心的部分。一个 UPlayer 可以是本地的 LocalPlayer，也可以是一个网络控制 UNetConnection。PlayerController 只有在 SetPlayer 之后，才可以开始正常工作。
*   HUD 显示，用于在当前控制器的摄像机面前一直显示一些 UI，这是从 UE3 迁移过来的组件，现在用 UMG 的比较多，等介绍 UI 模块的时候再详细介绍。
*   Level 的切换，PlayerController 作为网络里通道，在一起进行 Level Travelling 的时候，也都是先通过 PlayerController 来进行 RPC 调用，然后由 PlayerController 来转发到自己 World 中来实际进行。
*   Voice，也是为了方便网络中语音聊天的一些控制函数。

简单来说，PlayerController 作为玩家直接控制的实体，很多的跟玩家直接相关的操作也都得委托它来完成。目前来说 PlayerController 里旗下的 100 + 的函数也大概可以分为以上几大模块，也根据需要重载了 Controller 里的一些其他函数。  
UE 的思想是具象化一个 “玩家实体”，并把所有的跟该玩家相关的操作和接口都交给它完成。一般其他的游戏引擎只是个 “功能引擎”，提供了一些图形渲染 UI 系统等组件，但是在 GamePlay 这个层次就都非常欠缺了，一般都需要开发者自己搭建一套。而回想你写过的游戏，是不是也往往有一个 Player 类（一般是单件或者全局变量）？里面几乎是放着所有跟该玩家相关的业务逻辑代码。UE 里的 PlayerController 就是这种概念，优点当然是直接方便好理解，缺点也如你所见，会代码膨胀得比较快。不过目前来说还算能接受，等某一块功能真的比较大了之后，可以再把它抽出一个单独的类来，如 PlayerInput 和 PlayerCameraManager 一样。

**思考：哪些逻辑应该放在 PlayerController 中？**  
回想我们上篇的问题：“哪些逻辑应该写在 Controller 中？”，该处的答案观点在本处也依然适用。不过我还想再补充几点：

*   对实现游戏逻辑来说，如果是按照 MVC 的视角，那么 View 对应的是 Pawn 的表现，而 PlayerController 对应的是 Controller 的部分，那 Model 就是游戏业务逻辑的数据了。拿超级马里奥游戏来举例子，把问题先局限在一个关卡内，假设要实现的是金币的逻辑，那么 View 指的是游戏右上角的金币数目 UI，而玩家用 PlayerController 来控制马里奥来蹦跳行走，而马里奥（Pawn）通过触碰金币的事件又上报给 PlayerController 来相应增加金币。而 PlayerController 存储金币的数据就是在 PlayerState 中。即 PlayerState 中有一个 int coin，也有相应的 AddCoin(int coin)。而 PlayerController 的职责应该是一边控制 Pawn，一边负责内部正确的调用 PlayerState 的 Coin 接口。那么 PlayerController 里的成员变量有什么用？根据单一职责原则，我们写在哪个类里的变量应该尽量只符合该类的作用，所以 PlayerController 里的变量的意义在于更好的实现控制。比如假设玩家在一个关卡内可以按 AABB 来作弊获得 100 金币，但是限最多 3 次。那么这个按键的响应就应该由 PlayerController 来接收，然后调用 AddCoin(100)，并更新 PlayerController 里的成员变量 CoinCheatCount。也或者想实现马里奥的加速跑，也可以在 PlayerController 里增加 Speed 的成员变量。
*   记住 PlayerController 是可被替换的，不同的关卡里也可能是不一样的。比如马里奥在水下的时候控制的方式明显就不一样，所以就不能像 “Player” 单件类那样什么都往里面塞。这样一旦被替换掉了之后数据就都丢失了。
*   PlayerController 也不一定存在，考虑一下如果把马里奥做成联机游戏，那么对方玩家被同步过来的将只有 PlayerState，对方玩家的 PlayerController 只在服务器上存在。所以这个时候，如果你把金币数据放在 PlayerController 里的话就非常尴尬了。所以为了扩展性来说，还是根据职责分明的原则来正确划分业务逻辑会比较好。
*   在任一刻，Player:PlayerController:PlayerState 是 1:1:1 的关系。但是 PlayerController 可以有多个备选用来切换，PlayerState 也可以相应多个切换。UPlayer 的概念会在之后讲解，但目前可以简单理解为游戏里一个全局的玩家逻辑实体，而 PlayerController 代表的就是玩家的意志，PlayerState 代表的是玩家的状态。

## AAIController

从某种程度上来说，AI 也可以算是一个 Player，只不过它不需要接收玩家的控制，可以自行决策行动。从玩家控制的逻辑需要有一个载体一样，AI 的逻辑算法也需要有一个运行的实体。而这就是 UE 里的 AIController：  

![[e180a5fc180032be9200afc3d0967915_MD5.png]]

同 PlayerController 对比，少了 Camera、Input、UPlayer 关联，HUD 显示，Voice、Level 切换接口，但也增加了一些 AI 需要的组件：

*   Navigation，用于智能根据导航寻路，其中我们常用的 MoveTo 接口就是做这件事情的。而在移动的过程中，因为少了玩家控制的来转向，所以多了一个 SetFocus 来控制当前的 Pawn 视角朝向哪个位置。
*   AI 组件，运行启动行为树，使用黑板数据，探索周围环境，以后如果有别的 AI 算法方法实现成组件，也应该在本组件内组合启动。
*   Task 系统，让 AI 去完成一些任务，也是实现 GameplayAbilities 系统的一个接口。目前简单来说 GameplayAbilities 是为 Actor 添加额外能力属性集合的一个模块，比如 HP，MP 等。其中的 GamePlayEffect 也是用来实现 Buffer 的工具。另外 GamePlayTags 也是用来给 Actor 添加标签标记来表明状态的一种机制。目前来说该两个模块似乎都是由 Epic 的 Game Team 在维护，所以完成度不是非常的高，用的时候也往往需要根据自己情况去重构调整。

本文重点不在于讨论 AI 内部的各种组件功能，因此我们先把目光聚焦在 AIController 对象本身上。同 PlayerController 一样，AIController 也只存在于 Server 上（单机游戏也可看作是 Server）。游戏里必须有玩家参与，而 AI 可以没有，所以 AIController 并不一定会存在。我们可以在 Pawn 上配置 AIControllerClass 来让该 Pawn 产生的时候自动为它分配一个 AIController，之后自动释放。

**思考：哪些逻辑应该放在 AIController 中？**  
我们依然要思考这个问题，大部分思想和原则和 PlayerController 是一样的，只不过 AI 算法的多种多样，所以我们推荐尽量利用 UE 提供的行为树黑板等组件实现，而不是直接在 AIController 硬编码再度实现。也请把目光仅仅局限在当前的 Pawn 身上，不要在里面写其他无关的逻辑。另外，因为 AIController 都是在关卡内比较短暂存在的，一般不太有跨 Level 的数据保存，所以你可以用 AIController 的成员变量来保存状态。而如果真的需要用到 PlayerController 的状态，则也可以引用一个 PlayerState 过来。如果想引用关卡的全局状态，也可以引用 GameState，再更高级别的，甚至可以直接和 GameInstance 接触。  
但是 AIController 也可以通过配置 bWantsPlayerState 来获得自己的 PlayerState，所以 PlayerState 其实也并不是跟 UPlayer 绑定的，毕竟从本质上来说 APlayerState 也只是个 AInfo（AActor），跟其他 Actor 一样可以有多个，并没有什么稀奇的，区别是你自己怎么创建并利用它。

## 总结

到此，我们也算讨论完了 Actor（Pawn）层次的控制，在这个层次上，我们关注的焦点在于如何更好的控制游戏世界里各种 Actor 交互和逻辑。UE 采用了分化 Actor 的思维创建出 AController 来控制 APawn 们，因为玩家玩游戏也全都是控制着游戏里的一个化身来行动，所以 UE 抽象总结分化了一个 APlayerController 来上接 Player 的输入，下承 Pawn 的控制。对于那些自治的 AI 实体，UE 给予了同样的尊重，创建出 AIController，包含了一些方便的 AI 组件来实现游戏逻辑。并利用 PlayerState 来存储状态数据，支持在网络间同步。  

![[5dd82cccaaa8c958f85a95b558b84815_MD5.png]]

上图应该可以比较清晰的阐明，UE 是如何充分利用 Actor 的本身机制来反过来实现对 Actor 的逻辑控制，相信亲爱的读者朋友们也能自行体会到它的优雅之处。对比其他的游戏引擎，往往它们都止步于 Actor 这一个层次，只提供了最基本的对象层次，美名其曰交给玩家控制。UE 为我们提供了这一套简洁强大的机制，大大方便了我们编写逻辑的难度。

而下篇我们的逻辑之旅将再继续拔高一个层次，将开始讲解 World 层次的逻辑，这个世界的意志：GameMode！

上篇：[《InsideUE4》GamePlay 架构（五）Controller](https://zhuanlan.zhihu.com/p/23480071)  
下篇：[《InsideUE4》GamePlay 架构（七）GameMode 和 GameState](https://zhuanlan.zhihu.com/p/23707588)

## 引用

1.  [PlayerController](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/Controller/PlayerController/index.html)
2.  [AIController](https://docs.unrealengine.com/latest/INT/Gameplay/Framework/Controller/AIController/index.html)

_UE 4.13.2_

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**