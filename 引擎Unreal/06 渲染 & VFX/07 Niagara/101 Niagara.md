# 大世界坐标
**大世界坐标** （ **LWC** ），ue5中数据类型FVector现在是 `double` 而不是 `float`

Niagara实现与主引擎的实现不同。因为需要高效执行计算，无论是在CPU还是GPU上，都存在限制，导致无法处理double。Niagara中改用了一种存储位置数据的新方法。

世界大到一定程度时，将划分为图块网格。想象一下空间中的位置，它在图块单元中有自己的相对位置，还有该图块在世界中的位置。请参见下图，了解其外观的概念呈现。
![[Pasted image 20230116162203.jpg]]
之前，Niagara位置与向量可互换使用，定义为从世界原点出发的方向和距离。现在，要让信息有意义，需要额外数据才能根据所在图块定位发射器。

在UE5中，要保存相对于原点的位置以及系统所在图块的信息，我们需要新的数据格式。这样一来，我们可以区分其他类型的向量和带有这些额外数据的位置。
## UE4和UE5中的位置数据
UE4和UE5之间的主要差异是粒子位置数据的存储方式，即 `Particles.Position` 。
![[Pasted image 20230116162312.png]]
# 堆栈和堆栈组
组Group（系统、发射器或粒子）
组包含有阶段Stage（更新Update、生成Spawn、事件Event或模拟Simulation）
每个阶段都有不同的**模块Modue**

从概念上讲，Niagara中的粒子模拟作为堆栈运行，模拟从堆栈顶部流向底部，依次执行各模块。关键点是，每个模块都指定到组，以确定模块执行时间。

# 数据读写
堆栈组关联**namespace**；此类命名空间将定义**该组中的模块可读取或写入的数据**。
namespace在变量名前面查看：
![[Pasted image 20230117203208.png]]
![[Pasted image 20230116152027.png]]
# 发射器

发射器可用来在Niagara系统中生成粒子。发射器将控制粒子的生成、粒子在生命周期中的遭遇，以及粒子的外观和行为。

发射器位于堆栈中。在该堆栈中有几个**组**，而在组中可以放置用于实现各个任务的**模块Modue**（模块就是每个组“+”号里的功能，可以将模块添加到组中来形成堆栈。模块按照自上而下的顺序处理。）。 组如下所示。
![[Pasted image 20230116143526.png]]
Properties默认开启Interpolated Spawning
![[Pasted image 20230117213835.png]]
开启后，粒子产生时的那一帧，Spawn和Update会同时执行，如果关闭可以节省性能。

开启**Determinism（确定性）** 可以让粒子所有Radom属性值失效，每帧不在随机值，将各种可能的结果用随机种子Random Seed做索引，每个随机种子值对应不一样的结果。
![[Pasted image 20230119233628.png]]


# Emitter Spawn
 此组将定义在CPU上首次创建发射器时将会发生什么。使用此组可以定义初始设置和默认值。
    
# Emitter Update
 此组将定义CPU上每一帧发生的发射器级模块。如果你希望粒子在每一帧上持续生成，可以使用此组来定义粒子的生成。

# Particle Spawn
  当粒子生成时，每个粒子将调用一次此组。此时你可能需要定义粒子的初始化细节，例如粒子的生成位置、粒子的颜色、大小和其他特征。

# Particle Update
每一帧上的每个粒子都会调用此组。你需要在此处定义在粒子生命周期中将会逐帧更改的所有特征。例如，粒子的颜色会随着时间逐渐变化。或者，粒子受到各种力的影响，例如重力、旋度噪点或点吸引。你甚至可能需要让粒子随着时间改变大小。
    
# Event Handler
在事件处理器组中，你可以在一个或多个用于定义特定数据的发射器中创建"生成"事件。然后，你可以在用于触发某个行为以响应该生成的事件的其他发射器中创建"侦听"事件。

它们可以被添加到生成和更新流程中。每当粒子生成一个新事件，并设置一个发射器去处理该事件时，就会触发 **事件**。如果有可能，事件句柄阶段会发生在同一帧中，但位于发起事件之后。
![[Pasted image 20230116151119.png|300]]
# Simulation Stages
 **模拟阶段** 是一个高级GPU功能。该功能可让一个序列中发生多个生成和更新阶段，对于构建流体模拟之类的复杂结构非常有用。

# Render

### Ribbons条带
常用模块如下：
![[Pasted image 20230117234811.png]]
#### 调节曲线张力
Ribbon Renderer中设置
![[Pasted image 20230117234044.png]]
影响曲线的尖锐程度

#### Beam Emitter Setup
默认条带是直线，要向效果中添加弧或者曲线，勾选复选框来启用 **使用光束切线（Use Beam Tangents）** ，这样会显示 **光束开始切线（Beam Start Tangent）** 和 **光束结束切线（Beam End Tangent）** 设置。
![[Pasted image 20230117234322.png]]
#### JitterPosition
在Particle Update中增加该模块，可以让条带抖动
必须添加更新光束（**Update Beam**）模块才能使抖动正常
![[Pasted image 20230117235115.png]]
如果将 **抖动延迟（Jitter Delay）** 减小到 **0.1**，你将开始看到光束弯曲并呈角度锯齿状运动。如果值为 **0.1**，你仍然可以看到锯齿状光束下的原始弧形，这并非理想效果。要修复此问题，你必须将抖动延迟（Jitter Delay）设置为负数。将 **抖动延迟（Jitter Delay）** 设置为 **-0.01**
#### 自定义光束位置
默认情况下，光束的结束位置设为一个静止的世界位置数值，可以手动进行调整。
然而，很多时候我们需要将结束位置链接至场景中的一个Actor，这样我们就可以通过移动Actor来编辑光束的结束位置。可以使用蓝图来达到这样的目的，另一个更简单的办法是用暂存区（Scratch Pad）设置一个动态输入。
![[Pasted image 20230117235256.png]]
点击 **光束结束位置（Beam End）** 旁边的下拉箭头，找到 **暂存（Scratch）** 然后选择 **新动态暂存输入（New Dynamic Scratch Input）**。
![[Pasted image 20230117235340.jpg]]
![[Pasted image 20230117235430.png]]
在编辑器左下角Local Modules可以看到新建的Dynamic input
![[Pasted image 20230117235646.png]]


在Niagara编辑器中，再次选择 **光束发射器设置（Beam Emitter Setup）** 模块，会出现一个 **新Actor组件界面（New Actor Component Interface）** 并已经链接至 **光束结束位置（Beam End）** 。点击下拉箭头，选择 **制作（Make）** > **从新用户参数读取（Read from new User parameter）** 。 这样会创建一个新的可以在Niagara系统之外设置的用户参数。
![[Pasted image 20230117235549.jpg]]
在编辑器左下角User Parameter中可以看到新建的用户参数
![[Pasted image 20230117235738.png]]

将Niagara拖入场景中，在 **细节（Details）** 面板中，向下滚动至 **覆盖参数（Override Parameters）** 部分。在这里可以看到之前创建暂存模块时设置的用户参数 **Beam_End** 。打开 **源Actor（Source Actor）** 下拉菜单，然后选择 **Sphere_BeamEnd**。或者你也可以使用滴管工具在你的关卡中选择任何Actor。现在你再去移动那个Actor时，光束结束位置会跟随它移动。
![[Pasted image 20230117235900.png]]
最终效果：
![[beam-effect-final.gif]]
# 事件
在许多情况下，需要一个系统中的多个发射器相互交互，才能打造出所需的效果。通常情况下，这意味着一个发射器生成一部分数据，然后其他发射器侦听该数据，并执行一些行为来响应该数据。在Niagara中，此操作使用 **事件（Events）** 和 **事件处理器（Event Handlers）** 来完成。
**事件（Events）** 是生成粒子生命周期中发生的特定事件的模块。
**事件处理器（Event Handlers）** 是侦听生成事件然后启动某种行为来响应该事件的模块。

> [!Danger] 
> 当前版本中，事件无法结合GPU模拟使用。事件仅能CPU模拟使用。

由于事件会在粒子的整个生命周期内动态发生，会在"粒子更新（Particle Update）组"中添加事件。如果你点击粒子更新旁边的 **加 (+)**，你会看到一个名为 **事件（Event）** 的分段，其中可以在堆栈中添加更多事件模组。
![[Pasted image 20230116163508.jpg]]

> [!Danger] 
> 要使用事件，必须在发射器的发射器属性（Emitter Properties）中启用"需要持久ID（Requires Persistent IDs）"。
![[Pasted image 20230117202345.png]]

## 位置事件
将 **生成位置事件（Generate Location Event）** 模块放置到发射器的粒子更新（Particle Update）组中时，**该发射器中生成的每个粒子将在其生命周期内生成位置数据。然后可以设置事件处理器（Event Handler），接收该位置数据并触发其他行为。**

举例而言，若要为烟花火箭创建尾迹效果，则可将 **生成位置事件（Generate Location Event）** 模块放置到火箭发射器的粒子更新（Particle Update）组中。然后，尾迹发射器可使用位置数据生成跟随火箭的粒子。
![[Pasted image 20230116163727.jpg]]
## 消亡事件
将 **生成消亡事件（Generate Death Event）** 模块放置到发射器的粒子更新（Particle Update）组中时，**该发射器中生成的每个粒子将在其生命周期结束时生成事件。** 

使用此数据的方法有很多。可以在第一个发射器的粒子消亡时触发另一个发射器的粒子效果；也可以制造连锁反应，让每个发射器在前一个发射器的粒子消亡时生成各自的效果。可结合位置事件和消亡事件创建复杂的交互。

以烟花为例，可以在火箭粒子生命结束时生成爆炸效果。位置事件可确定火箭粒子的位置，即爆炸发生的位置。消亡事件可确定粒子的生命结束时间，即爆炸效果发生的时间。
![[Pasted image 20230116163752.jpg]]
## 碰撞事件
将 **生成碰撞事件（Generate Collision Event）** 模块放入发射器的粒子更新（Particle Update）组后，粒子与Actor（例如静态网格体或骨骼网格体）碰撞时，其将生成事件。
举例而言，若要将烟花效果改为武器效果，可以设置当火箭粒子与静态或骨骼网格体碰撞时发生爆炸。
![[Pasted image 20230116163904.jpg]]

> [!NOTE]
> 需要先向发射器添加 **碰撞（Collision）** 模块，然后才能向该发射器添加 **生成碰撞事件（Generate Collision Event）**。这样发射器的粒子便可以与场景中的对象碰撞。

## 事件处理器
事件处理器由两部分组成：**事件处理器属性（Event Handler Properties）** 和 **接收事件（Receive Event）**。
针对需要发射器予以响应的每个事件，添加 **事件处理器属性（Event Handler Properties）** 项和 **接收事件（Receive Event）** 模块。
点击发射器属性旁边的 **加号（+）**，就能为发射器添加一个 **事件处理器**。
![[Pasted image 20230116151119.png|300]]
在 **事件处理器属性（Event Handler Properties）** 中，使用下拉列表设置事件的 **源（Source）**。该下拉列表列出了所有可用的生成事件（Generate Event）模块。然后可以选择受事件影响的粒子，每帧事件发生的次数；若事件生成粒子，则可选择生成粒子的数量。
![[Pasted image 20230116164217.jpg]]
设置事件处理器（Event Handler）的属性后，请选中一个**接受事件（Receive Event）**。它必须与放置在生成事件发射器的粒子更新（Particle Update）组中的生成事件模块相匹配。
![[Pasted image 20230116164224.jpg]]
举例而言，若在发射器中放置 **生成位置事件（Generate Location Event）**，则可为事件处理器（Event Handler）选择 **接收位置事件（Receive Location Event）** 模块。
![[Pasted image 20230116164238.jpg]]
# 图像序列视图
将Niagara模拟烘焙成 **图像序列视图（Flipbook）** 。这样会创建许多平铺图像，并加载到材质上，以此作为特效，兼顾视效与性能。

例如，你想创建3D流体效果，但无法在目标平台上实时运行它。因此，在创建该3D流体效果之后，你可以使用 **烘焙器（Baker）** 将其烘焙成图像序列视图，然后将其应用回2D Sprite发射器。这样一来，你就可以让游戏背景中远处的次要特效拥有更高效的性能。

-   设置图像序列视图的捕获
-   执行捕获
捕获序列图的方法：[虚幻引擎中的Niagara图像序列视图烘焙器快速入门指南](https://docs.unrealengine.com/5.1/zh-CN/niagara-flipbook-baker-quick-start-guide-in-unreal-engine/)
-   将图像序列视图连接到新发射器

使用SubUV：
Render模块中选择对应材质，并且Sub Image Size的XY分别设置对应的行列值
并启用Sub UV Blending Enable （子UV混合）
![[Pasted image 20230116171050.png|]]

Particle Spawn中添加Sub UVAnimationo模块，我们的图时8x8共 64帧，所以始末帧设置成（0~63）
![[Pasted image 20230116171343.png]]
![[Pasted image 20230116171328.png]]

# GPU 
![[Pasted image 20230116171823.png]]

> [!NOTE] 
> 由于粒子模拟是在GPU上完成的，因此系统无法读取效果有多大。这就是为什么有必要设置固定边界的原因。可以在发射器中执行此操作，如图所示，也可以在“系统属性”中为整个系统设置固定边界。

**生命周期模式**设置为**系统**。这使您的系统能够计算生命周期设置，这通常会优化性能。
![[Pasted image 20230116172529.png]]
系统的生命周期在SystemSpawn中设置。默认情况下，系统以 5 秒的间隔无限循环。
![[Pasted image 20230117225616.png]]
大多数功能在CPU和GPU模式下都能工作，但有些功能的表现是不同的，例如GPU碰撞在GPU上的处理性能更高，但精度更低，而某些渲染器，例如光线渲染器和带状渲染器，目前只在CPU上使用。

GPU模拟的好处是，较高的粒子数通常可以在GPU上得到很好的处理，缺点是如果你已经有大量的GPU约束，有时最好让CPU来代替模拟粒子的世界。

值得注意的是，只有Particle脚本是在GPU上运行的。System和Emitter只在CPU上执行，因此只限于CPU友好的操作，例如，系统和发射器脚本不能对纹理进行采样，也不能查询场景距离场。
# Niagara附加到动画
打开Animation Sequnence

找到 **通知（Notifies）** 分段。借助通知，你可以在动画上的某个位置进行标记，以便播放粒子效果。为了保持时间轴的整洁，创建一个新的轨道来放置Niagara事件。点击 **轨道（Track）**， 然后选择 **添加通知轨道（Add Notify Track）**。将其命名为 **Niagara**。
![[Pasted image 20230117231818.jpg]]
会看到一条位于时间轴拖动条下方的直线。右键点击播放头和Niagara动画通知轨道相交的位置。选择 **添加通知（Add Notify）> 播放Niagara粒子效果（Play Niagara Particle Effect）**。这样就能在动画中的该点位置上放置一个标记，并且带有默认标签PlayNiagaraEffect。
![[Pasted image 20230117231826.jpg]]
选中 **PlayNiagaraEffect** 通知，在 **细节（Details）** 面板中找到 **动画通知（Anim Notify）** 部分。在此处选择要添加到动画中的Niagara系统。点击 **Niagara系统（Niagara System）** 旁边的下拉菜单，然后选择在Niagara中创建的 **FX_DustCloud** 系统。通知上的标签更改为 **FX_DustCloud**。
![[Pasted image 20230117231859.jpg]]
# 材质交互
在材质编辑器中有很多接口可以接通niagara，对应着particle的属性
![[Pasted image 20230117232844.png]]![[Pasted image 20230117231422.png]]

材质编辑器中设置Dynamic Parameter
![[Pasted image 20230117232233.png]]

在Render中使用材质
![[Pasted image 20230117232542.png]]
在Particle Spawn或Update阶段可以添加Dynamic Material Parameter，我们可以看到Erode参数和我们在材质编辑器中设置的一致，通过点击箭头既可以对该值进行编辑。
![[Pasted image 20230117232501.png]]
# 蓝图交互

