链接: https://pan.baidu.com/s/1i5dpX1V 密码:hvmn
# FPS
**迁移至空项目**
新建一个没有初学者包的空项目，右键关卡地图
![[Pasted image 20230109131854.png]]
Migare，选择空项目的Content文件夹，即可将该关卡设计的资源迁移
## input控制
Project Setting->Input
![[Pasted image 20230109153636.png]]

两种Binding的区别：
Action：按下和松开两种分支
Axis:按下后每帧持续调用
![[Pasted image 20230109153811.png]]
## 动画蓝图
新建角色蓝图类->Character，设置Mesh的骨骼网格体
![[Pasted image 20230109135912.png]]
新建动画蓝图Animation blueprint，将动画文件拖入AnimGraph
![[Pasted image 20230109135806.png]]
回到角色蓝图，更改Mesh的Animation设置，选择动画蓝图
![[Pasted image 20230109140100.png]]
### 多线程警告
动画蓝图的多线程更新的好处：提高动画更新的速度
出现警告的原因：
1 在AnimGraph 中调用一些函数
2 这些函数没有声明为线程安全

避免警告的方法：
1 在设置中关闭多线程更新 ，AssetDetail的关闭多线程更新
1.1 既可以在当前动画蓝图中关闭
1.2 也可以在设置全部关闭
2 在AnimGraph中读取变量，在EventGrapth中更新变量

### 查看动画序列帧时间
打开东画拉动红条到最后
![[Pasted image 20230110121028.png]]

## 添加武器
打开动画蓝图，找到相应骨骼，右键添加socket
![[Pasted image 20230109143709.png]]
添加预览资源，用于在动画蓝图中调试socket位置
![[Pasted image 20230109143736.png]]
![[Pasted image 20230109143927.png]]
打开角色蓝图，添加组件（组件添加到角色Mesh下面，这样才能通过socket调整武器位置）
![[Pasted image 20230109144101.png]]
设置武器Mesh
![[Pasted image 20230109144118.png]]
设置父插槽，即在我们新添加的那个socket
![[Pasted image 20230109144336.png]]
发现位置还不对？
![[Pasted image 20230109144429.png|300]]
将武器mesh的Transform信息恢复默认，位置就和在动画蓝图中设置的一样了
![[Pasted image 20230109144459.png]]
## BlendSpace
用于混合动画，我们先创建1D的，1D混合变量有一个，上面那个是2D，有两个混合变量
![[Pasted image 20230109151619.png]]
设置轴变量名为Speed，我们根据角色的移动速度来混和动画。最大值设置为最大移动速度600（在角色蓝图的CharacterMovement组件中设置）
![[Pasted image 20230109151417.png]]
![[Pasted image 20230109151540.png|300]]
在动画蓝图中传入角色速度
![[Pasted image 20230109152338.png]]
## Montage
![[Pasted image 20230109152616.png]]
或者右键动画文件，直接创建
![[Pasted image 20230109160306.png]]
在AnimGraph中调用对应的slot
![[Pasted image 20230109160255.png]]
**开启循环播放**，这样我们按住鼠标左键就可以循环播放动画
![[Pasted image 20230110103258.png]]

## 开枪特效
WeaponMesh下新建一个Arrow，调整位置
![[Pasted image 20230109160607.png]]
## AI
### 设置寻路范围

1. 将 体积 / Nav [Mesh](https://so.csdn.net/so/search?q=Mesh&spm=1001.2101.3001.7020) Bounds Volume 拖拽到场景中

![](1673254091113.png)

2. 选中拖入的 NavMeshBoundsVolume，调整大小，将想要自动寻路的范围全部包裹。（尽量比场景轮廓大一点）

![](1673254091242.png)

![](1673254091335.png)

3. 选中 NavMeshBoundsVolume，按 P 查看寻路情况，再次按 P 消失。

![](1673254091411.png)
### 编辑寻路 AI
1. 创建AI机器人蓝图类，基于 Character 类创建，命名为 RobotShooter，主要赋值一个 mesh，然后调整好旋转角度。

![](1673254091496.png)

2. 创建动画蓝图，选择基于机器人对应的骨骼，创建完成后命名为 RobotShooter_AnimBP，主要设置机器人的状态，在动画图表里单一放置一个 Idle 的动画，并将状态连接。

![](1673254091596.png)

3. 在 RobotShooter 蓝图类中选中 Mesh，在细节面板的 Animation 栏中，将 Animation Mode 更改为使用动画蓝图，并将上面创建的 RobotShooter_AnimBP 选入。

![](1673254091687.png)

 ### AIController 蓝图类

创建方法，在文件夹空白处右键选择 BluePrint Class ，在创建面板的下方搜索 AI Controller，并选择，命名为 RobotController

![](1673254091741.png)

  创建完成后图标  

![](1673254091834.png)

5. 编辑 RobotController 内容

双击打开 RobotController，测试为游戏开始时就自动走到目标物旁，在事件图标空白处右键搜索 AIMoveTo ，选择 AI 移动到节点。

![](1673254091901.png)

  

![](1673254091956.png)

AI 移动到节点参数：

Pawn 代表机器人，

Destination 参数是目标移动到哪个点，

Target Actor 目标物 ，

Acceptance Radius 代表距离目标物多远的时候可以停下。

执行节点：空白的代表当执行到 AIMoveTo 此节点指令后 就会做的处理。

OnSuccess 代表当追上后执行操作

OnFail 没有追上的执行操作

到达位置后，使用 Delay 节点，可循环调用 AIMoveTo 节点，实现人物走开，机器人再次移动效果。

### 设置参数

使用 Get Controller Pawn 节点获取到当前 AI 类控制的角色也就是机器人。

使用 Get Controller Character 节点获取到玩家角色

![](1673254092011.png)

7. 编译后，返回 RobotShooter 蓝图类中，选中 RobotShooter（自身），在细节面板将 AIControllerClass 选择为 RobotController（上面创建的基于 AIController 的蓝图类）

![](1673254092076.png)

运行发现机器人走向角色，再给他加上跑步动画即可
## 射线追踪
角色蓝图中使用射线追踪实现弹道
注意，DrawDebug类型的选择，不然无法可视化射线
![[Pasted image 20230109172656.png]]
在AI的蓝图Construction Script里Set Collision Response to Channel
Channel要和LineTraceByChannel设置的TraceChannel一致
![[Pasted image 20230109172603.png]]

![[Pasted image 20230109172829.png]]

## UI
### 制作准星
创建widget blueprint，锚点设置为中间，将准星图片加入到画面中间
![[Pasted image 20230109183153.png]]
在gamemode中将准星加入视口
![[Pasted image 20230109182921.png]]

## 场景小优化
### post process volume
后处理体积设置无限范围，添加镜头耀斑
![[Pasted image 20230109194420.png]]
![[Pasted image 20230109195246.png]]
![[Pasted image 20230109195254.png]]
### box reflection Capture
盒体反射捕获，体积覆盖全图
# 面向对象

## ChildActor
我们将武器单独抽象成一个蓝图类，在角色蓝图中删掉了武器的mesh，那么如何让角色拿上武器呢？
在角色Mesh下增加ChildActor组件
![[Pasted image 20230109210518.png]]
选择socket和武器的蓝图类，调整位置即可
![[Pasted image 20230109210542.png]]
在角色蓝图中设置，这样就可以操作武器了
![[Pasted image 20230109211316.png]]

## UML类图
**统一建模语言**（**U**nified **M**odeling **L**anguage，UML）
十多种图，一种图表示一种策略

介绍其中之一：
类图 Class Diagram
表达类之间的关系
继承、组合
实现、聚合、依赖、关联

用来描述代码中类之间的关系，在设计架构阶段使用


## 蓝图重构
上文我们实现了一种武器，不适用于多种武器的情况，为了方便调用，我们需要建立一个武器的基类，类图如下：
![[Pasted image 20230109213533.png]]

**第一个方法：**
增加一个Gun类，让目前的Rifle从Gun继承，在基于Gun创建Launcher

新建父类BP_Gun，需要用到的的函数
![[Pasted image 20230109214201.png]]

武器蓝图将父类设置为BP_Gun
![[Pasted image 20230109214241.png]]

报错，说函数已经被使用
![[Pasted image 20230109214608.png]]
另外还需要改变角色蓝图中的Gun变量，导致要修改很多节点
这种方法很麻烦，我们换一种方法、

**第二种方法:**
把Rifle改名成成Gun，基于Gun新建Rifle和Launcher。

创建蓝图子类：
![[Pasted image 20230109215143.png|300]]
## 运行时创建Actor
我们只需要切换角色蓝图的ChildActorclass就可以实现武器切换了
![[Pasted image 20230109223529.png]]
但这种方法会使得其他函数无法执行

SpawnActor节点生成武器蓝图类并Attach到根组件的Socket
![[Pasted image 20230109224439.png]]

## 创建多把枪
将枪添加进数组
![[Pasted image 20230109225502.png]]
![[Pasted image 20230109225521.png]]
## 换枪
![[Pasted image 20230109215522.png]]

##  飞行的子弹
两种子弹：
瞬时子弹
飞行的子弹

飞行的子弹制作方法：
方法一：ProjectileMovement发射物品移动组件
方法二：模拟物理

### 方法一
一、创建 actor 为[基类](https://so.csdn.net/so/search?q=%E5%9F%BA%E7%B1%BB&spm=1001.2101.3001.7020)的蓝图，创建组件 ProjectileMovement

![](1673263964126.png)

1. 设置运行初速度，使物体生成时拥有初始速度

![](1673263964228.png)

2. 设置物体碰撞后是否反弹

![](1673263964310.png)

如果物体生成后没有抛物线运动，可以去查看物体的质量是否太大

二、**在使用 ProjectileMovement 组件时，要注意该组件使用的是根节点的碰撞**，如果在 Sphere 中的细节面板设置 Collision 并不会起作用，所以如果想使用物体碰撞需要 AddComponent 添加 SphereCollision，然后替换掉 DefaultSceneRoot 根节点并将碰撞模式更改为blockall。

将Sphere移动到DefaultSceneRoot会提示是否删除默认root：
![[Pasted image 20230110115302.png]]

注：如果将 Sphere 静态网格物体作为根节点并且设置碰撞阻挡，当生成时，因为spawn Actor 节点有设置生成物体大小，如果生成物体种类过多就会导致大小不一致，所以此处没有将静态网格物体设置为根节点。

![](1673263964479.png)

注意：ProjectileMovement 并没有在 Sphere 下，因为 ProjectileMovement 是 actorComponent 而不是 SceneComponent。

**actorComponent：没有变换数据结构，位置缩放朝向并无影响

**SceneComponent：带有变换数据结构，能够形成一定的层级结构，如上面做图的层级关系。actor 的根组件必须是 SceneComponent 类型的****

设置生命周期：
![[Pasted image 20230110115707.png]]

三、物体碰撞到物体后产生伤害

使用 HitEvent 节点
![[Pasted image 20230110121703.png]]
### 方法二
子弹蓝图类中新建sphere，开启模拟物理
![[Pasted image 20230110122845.png]]

eventgraph中设置速度
![[Pasted image 20230110123205.png]]

## 查看引用
![[Pasted image 20230110134235.png]]
## 游戏主循环
![[Pasted image 20230112130820.png]]
- 游戏主循环是单线程的，设计思想上模拟了并行（并不会真正的“同时”）
- 编译、构建、运行时与显卡的交互，可能是多线程的
- 用户可以自己创建多线程
## 变量引用
![[Pasted image 20230112131729.png]]
# AI和行为树
## 行为树
1  运行 行为树前提

注：（在 AI Controller 中运行行为树，如果在其他蓝图中将要获取到 AI Controller 这个类才可以进行运行并调用行为树中的一系列操作）

1. 在运行行为树之前要有一个 Pawn（可以为角色，或者其他实体），并且该 pawn 有一个关联的 AI Controller （AI 控制器在 AI 寻路中有创建说明，如果是 AI 巡逻操作行为树要注意使用寻路[网格](https://so.csdn.net/so/search?q=%E7%BD%91%E6%A0%BC&spm=1001.2101.3001.7020)）

2 行为树的创建

1. 文件夹空白处右键选择 Artificial Intelligence（人工智能）选择 Behavior Tree（行为树）

![](1673327340810.png)

2. 创建黑板：可以将黑白资源视为 AI 的大脑，他会**存储键（可以在蓝图中设置）**，以便行为树做出决策

方法一：第一步步骤选择 Blackboard 创建后可以重命名

方法二：打开创建的行为树，上方工具栏中选择创建黑板。

![](1673327341137.png)

3. 行为树与黑板关联

在行为树细节面板 AI 选项中设置使用的黑板，设置完成后，在右上方的两个选项可以来回切换行为树及黑板面板。

![](1673327341384.png)

黑板以及装饰节点黑板使用，后面会展示示例。

4.AI Controller 使用行为树

打开以 AIController 为基类的蓝图，在 Beginaplay 中设置使用黑板 Use Blackboard 以及运行行为树 RunBehaviorTree

![](1673327341714.png)

并设置需要引用的黑板及行为树，选择我们创建的黑板及行为树即可。

5. 编译运行，即可运行行为树，也可以对黑板中的变量进行操作。
## 选择器Selector
![[Pasted image 20230110213539.jpg]]
节点按从左到右的顺序执行其子节点。**当其中一个子节点执行成功时，选择器停止执行。**
如果选择器的**一个子节点成功运行，则选择器运行成功**。如果选择器的所有子节点运行失败，则选择器运行失败。

## 序列Sequence
![[Pasted image 20230110213626.jpg]]
按从左到右的顺序执行其子节点。**当其中一个子节点失败时，序列停止执行。**
**如果有子节点失败，那么序列就会失败**。如果该序列的所有子节点运行都成功执行，则序列节点成功。


## 自定义Task

1. 在[行为树](https://so.csdn.net/so/search?q=%E8%A1%8C%E4%B8%BA%E6%A0%91&spm=1001.2101.3001.7020)中的工具栏选择 New Task （新建任务）选择以 BTTask_BlueprintBase 作为基类后（之前有创建过的也会在下方进行显示，如果没有创建过默认以该类为基类，第二次创建则会有提示），创建的蓝图在行为树所在文件夹中，并重命名为 AlwaysTrue，AlwaysTrue 是一个蓝图类，可以像操作其他蓝图一样操作此任务蓝图，执行逻辑。

![](1673356256578.png)

![](1673356256658.png)

2. 打开创建的 AlwaysTrue 蓝图，创建 Event Receive Execute 节点，该节点发生在当 Task 要求被执行时。

![](1673356256928.png)

参数：Owner Actor 代表的是 AI Controller 的拥有者 

现在执行逻辑打印 Class 的名字

3.Event Receive Execute 节点要有 Finish Execute 节点返回 Success ，否则在行为树中该节点将无限等待。

![](1673356257003.png)

4. 执行逻辑编辑

![](1673356257082.png)

5. 在行为树中使用 Sequence 序列节点，执行搜索 Always True 以及 Wait 节点

![](1673356257237.png)

 完成：

![](1673356257436.png)

运行：

![](1673356257543.png)

6. 在 task 蓝图中可以设置 / 获取到 Blackboard 中的变量，也可以创建变量、函数。

## Decorator装饰器
**Decorator 是条件语句只能附加在其他节点上 并且定义所附加的节点是否执行** 如果 Decorator 是 true 它所在的子树会被执行，如果是 false 所在的子树不会被执行
### 自定义装饰器
1. 创建装饰器，在行为树界面工具栏创建装饰器，在行为树所在文件夹找到对应的蓝图，重命名 D_AlwaysTrue.

![](1673357323948.png)

![](1673357324121.png)

2. 覆盖函数

在 D_AlwaysTrue 蓝图函数列表中选择 Override 选择 PerformConditionCheck

![](1673357324280.png)

创建效果：

![](1673357324446.png)

将 return value 更改为 true

3. 在行为树中使用装饰器节点，因为要附着于其他节点，所以测试使用 Sequence 节点。选中 Sequence 节点，右键选择添加装饰器选择定义的装饰器名称。

![](1673357324613.png)

逻辑图：

![](1673357324784.png)

4. 因为返回值为 true，下面 Sequence 节点会被执行，如果将 D_AlwaysTrue 中返回值更改为 false，则不会执行下面的节点。

更改为 false 的效果：显示红线说明该节点返回的是 false，只有此节点不是最后一个节点时才会显示红线。

![](1673357324958.png)

### Blackboard
![[Pasted image 20230110214530.jpg]]
注意：与黑板重名，需要的变量是需要在黑板中设置的.

检查给定的 **黑板键（Blackboard Key）** 上是否设置了值。

1. 打开黑板，创建 NewKey，字符串格式 String，编译保存

![](1673357325126.png)

选择变量类型

![](1673357325298.png)

2. 将上面 Sequence 节点中的 D_AlwaysTrue 删除，右键选择添加装饰器 Blackboard

![](1673357325468.png)

创建效果：

![](1673357325634.png)

3. 创建完成后选中创建的装饰器，在细节面板中设置属性

![](1673357325832.png)

4. 在细节面板中，将 Blackboar 栏下的 BlackboardKey 选择为 TestString

![](1673357326113.png)

KeyQuery：选择判断的条件，里面为等于 不等于 包含 不包含 

![](1673357326281.png)

KeyValue  是该键的值，如果条件选择等于，则程序将黑板中设置的 TestString 值传递过来会与此值进行对比，如果相等，则执行该条件下的子树逻辑，如果不等，则此条件下的子树不会执行。

设置的值为 1：

![](1673357326453.png)

5. 通过按键 xz 设置 TestString 值 

逻辑：1. 在场景中放入一个使用 AIController 的 pawn 

![](1673357326621.png)

2. 通过使用 Get All Actors Of Class 节点获取到场景中指定类的 actor，我测试用的 actor 使用的是以 RobotController 的 AI 控制器类。

3. 获取到 actor 后直接 get Blackboard 得到黑板设置黑板中的值属性。

![](1673357326776.png)

在获取到黑板后，拖拽连接线搜索 set values  就会出现可设置的选项，我们要选择在黑板中存在的变量类型。选择 String 类型。

![](1673357326943.png)

![](1673357327158.png)

参数：target 指的是 黑板，keyName  要设置的变量名称，string Value 要设置的变量值。

设置 key name 可以将此值提升为变量进行设置，也可以使用 make Literal Name 节点进行设置

![](1673357327349.png)

Z 键的设置 将 string value 变为 " "

![](1673357327537.png)

运行结果：

按 X 键，按顺序执行两个节点

![](1673357327707.png)

   

![](1673357327896.png)

按 Z 键，执行完第一个返回 false 后面的节点就不再执行

![](1673357328080.png)

注意：当装饰节点下方的子树执行时，如果 wait 节点设置的等待时间过长，此时按下 Z 键会发现只有 wait 时间执行完毕后才会切换到 false 状态。

**将 Blackboard 装饰器细节中的 Observer aborts 观察者终止 选择 self ，表示，当不满足条件时，就会立刻结束当前子树的执行。**

![](1673357328345.png)

在观察者终止中还有其他的参数，解释如下。

![](1673357328560.png)

Key Query 会随着设置参数的变量类型的不同而不同，请注意区分。

## Service
service 节点通常连接在复合节点或者任务节点，只**要其分支被执行，它们就会以定义的频率执行。**
这些节点常用于检查和更新黑板。它们取代了其他行为树系统中的传统平行（Parallel）节点。

**自定义Service节点**

1. 通过单击包含蓝图逻辑和（或）参数的工具栏中的 新建服务（New Service） 按钮来创建 服务节点。

![](1673358034582.png)

创建完成后在行为树所在文件夹找到系统创建的服务节点，重命名，双击打开

2. 举例：有时间间隔打印名称

再打开的服务节点蓝图右键搜索 Receive 选择 Receive Tick 创建节点，使用 Print string 打印间隔时间

![](1673358034640.png)

![](1673358034682.png)

其中 owner actor 指的是  AI Controller 这个类的拥有者

3. 使用：

在行为树蓝图中的 复合节点 Sequene 右键选择 添加服务，选择我们创建的 服务节点

![](1673358034730.png)

4. 选中添加的服务节点，在细节面板中调节参数

![](1673358034865.png)

在 service 选项中

Interval ：定义 tick 的间隔

Random Deviation：随即偏差

范围在：（Interval - Random Deviation，Interval + Random Deviation）

运行 打印结果（可以调整 wait 节点的等待时间 也可以将顶层的 sequence 节点更改为 selector 节点）

![](1673358034919.png)

该节点可以使用在 敌人 AI 边走边射击的逻辑中。

## AimOffset动画偏移

AimOffset 简单说就是叠加动作，如射击游戏中敌人射击时，根据人物位置不同造成的身体旋转角度的不同。

1. 创建 AimOffset 文件夹空白处右键选择 Animation 下的目标偏移（AimOffset），并选择对应的骨骼进行创建

![](1673361529759.png)


![](1673361529824.png)

2. 参数设置

![](1673361529911.png)

水平坐标指的是动画融合框中 X 轴的值范围

垂直坐标指的是动画融合框中 Y 轴的值范围

注：根据需求设置不同的值，这些值可以控制模型动画的动作（这里设置为 -90 90 的范围）

要声明 Name  名称 （在动画蓝图中进行赋值时的名称参数）以及取值范围   也可以设置将融合框的轴 分为几个区域 可以存放不同的动画 

插值类型以及插值时间 暂无涉及 此处选择默认即可

![](1673361530166.png)

3. 设置需要融合的动画

在资源浏览器中选择要融合的动画，双击打开其中一个。（以 AimOffsetDown 为例，这是动画中的一帧）

![](1673361530299.png)

在资源详情中，设置 Additive Settings ，将 Additive Anim Type 设置为 Mesh Space ， Base Pose Type 设置为 Selected animation（将动画的一帧用作一个基础姿势） 参考动画 设置为基础动画此处选择 Idle（在 AimOffset 设置混合空间中的基础动画也选择 Idle）

![](1673361530393.png)

MeshSpace: [网格](https://so.csdn.net/so/search?q=%E7%BD%91%E6%A0%BC&spm=1001.2101.3001.7020)体空间是一种略有不同的提取附加动画的差量的方法，因为它只用于特定的实例，如瞄准偏移资源。网格体空间使用网格体的边界框作为其旋转的基础，允许向相同的方向旋转，而不管骨架网格体中骨骼链的方向如何。考虑这样一个角色，身体向一侧倾斜，但需要将手枪向上瞄准。如果向上的瞄准运动是在局部空间内，那么瞄准就会向外旋转，向着倾斜的方向。

示例：

![](1673361530457.png)

如果要设置的动画较多，挨个设置操作不方便可以，选中动画右键浏览至资源，选中所有需要设置的动画，右键选择资源操作 通过属性矩阵进行批量修改，将 Additive Setting 选项中的参数全部更该为上面的值

![](1673361530513.png)

  

![](1673361530573.png)

![](1673361530633.png)

![](1673361530689.png)

选择基础动画，点击右侧的九宫格选项进行动作设置。

![](1673361530756.png)

一定要记得设置完成后返回文件夹下保存所有修改，如果不保存会导致无法将帧动画拖入到融合框中的情况。

 4. 设置 AimOffset 融合框中的动画，打开 AimOffset

在资源详情 Additive Setting 设置基础动画姿势为 Idle

根据动画方向设置位置, 设置完成后保存。

注：设置融合框动画，设置动作左右的时候要注意是设置我们面对电脑方向的左右（按照自己的左右）来设置人物的动作朝向，在融合框中按住 Shift 并移动鼠标就可以查看融合动画效果。

![](1673361530820.png)

5. 将上面制作的目标偏移拖拽到动画蓝图中，可以在动画蓝图的动画图表中右键搜索创建的 AimOffset 创建目标偏移，也可以在资源浏览器中找到对应的目标偏移直接拖拽到动画图表中

![](1673361530924.png)

    

![](1673361531011.png)

6. 与状态机连接，编写给水平偏移 及 垂直偏移传递参数的方法 （如果关闭了动画蓝图多线程的限制，此处可以使用纯虚函数进行值的获取及传递，如果没有关闭多线程限制，则需要在事件图表中编辑逻辑，将获取到的值提升为变量，在动画图表中获取并连接值即可）

![](1673361531062.png)

7. 敌人 AI 向自己射击时，获取的水平及垂直方向的值

通过敌人及主角两个位置点，获取到射击朝向，再获取到敌人自身的朝向，此时只需要获取到射击朝向与自身朝向之间的差值（要转换成同一种格式，要么四元数，要么 Vector3 , 相减), 然后获取到的变量通过 Break Rotator 节点，转换为 Roll Pitch Yaw 水平方向获取 Z 轴得值，垂直方向获取 Y 轴的值即可。

![](1673361531117.png)
##  使用混合空间代替AimOffset动画偏移

一、创建 BlanceSpace 混合空间

二、按照 AimOffset（上一章）设置水平轴垂直轴方法设置混合空间中的轴信息。并将九个状态分别拖拽到混合框的九个位置中

![](1673361422121.png)

三、设置九个帧动画中的动画叠加类型（Additive Anim Type）必须要设置为 Local Space，此处与 AimOffset 不同，如果不设置将出现动作错误效果

![](1673361422186.png)

也可以将资源全部选中，使用属性矩阵进行批量修改，将 AdditiveSettings 中的 叠加动画类型更改为 Local Space。

![](1673361422292.png)

注：AimOffset 使用的是 网格体空间（[Mesh](https://so.csdn.net/so/search?q=Mesh&spm=1001.2101.3001.7020) Space）

BlandSpace 使用的是 局部空间（Local Space）

四、动画蓝图中使用 混合空间中的偏移

使用叠加型姿势节点，在动画图表空白处右键搜索 apply additive。

![](1673361422337.png)

![](1673361422407.png)

该节点有两个参数，一个是基础动作，另外一个是叠加的动作，将创建的混合空间拖入到空白处（在资源浏览器中即可找到，连接前要先编译混合空间），其余参数与上一章一致，不需要更改。将参数连接。

![](1673361422497.png)

注：需要注意一点就是使用混合空间进行动画偏移，需要将帧动画的叠加动画类型更改为局部空间（Local space）。

而 AimOffset(动画偏移) 需要将帧动画的叠加动画类型设置为 网格体空间（Mesh Space）

## 区分敌我
 **方法一：Tag**
选中场景中的Actor可以设置标签
![[Pasted image 20230110231418.png]]
蓝图中判断是否有标签
![[Pasted image 20230110231453.png]]

方法二：在角色父类中添加变量，0/1判断
![[Pasted image 20230110231715.png]]
角色父类中创建isEnemy函数
![[Pasted image 20230110231917.png]]

## 控制台命令
关卡蓝图中自定义事件：
![[Pasted image 20230110234442 1.png]]
运行后按~调出控制台，按`ce SpawnRobot 123`
即可打印出123
所以自定义说输入的input可以通过控制台进行赋值

我们修改最终的关卡蓝图
按`ce SpawnRobot 1`召唤一个Team=1的robot
`ce SpawnRobot 0`召唤一个Team=0的robot
![[Pasted image 20230111112355.png]]

# UI
**虚幻UI设计器UMG：Unreal Motion Graphic 虚幻示意图形**
创建控件Widget蓝图
![[Pasted image 20230111170016.png]]

组件提升为变量才能在蓝图中调用
![[Pasted image 20230111165626.png]]

Event graph中调用Widget，并添加进屏幕
![[Pasted image 20230111165951.png]]
### 元素定位
锚点控制UI在屏幕中的相对位置
![[Pasted image 20230111170239.png|300]]

## 更新UI的方式
正式的产品开发中，尽量使用事件驱动
![[Pasted image 20230111202832.png]]
复杂度对比
1 绑定:多处更新无影响
2 事件驱动:每处更新都要触发事件

依赖：A引用了B，则A依赖B
1 绑定: Fpp UI依赖FppShooter
2 事件驱动: 
FppShooter依赖ChangeGun_Dispatcher,
Fpp UI也依赖ChangeGun Dispatcher

性能：
绑定：梅帧更新
事件驱动：有变化时更新

### 函数绑定
![[Pasted image 20230111173609.png]]

### 属性绑定
![[Pasted image 20230111174328.png]]
### 基于事件驱动
BP_Fppshooter蓝图中创建事件分发器EventDispatchers
![[Pasted image 20230111200718.png]]
类似变量，如果类生成了两个实例，那么这两个实例的Dispacher是不同的。
还可以增加变量：
![[Pasted image 20230111202236.png]]

切枪函数最后添加
![[Pasted image 20230111202259.png]]

在UI的eventgraph中绑定
![[Pasted image 20230111202331.png]]
意思就是每次call函数被调用都会执行Bind绑定的GunChanged事件
## Size to Content
保持大小和内容一致
![[Pasted image 20230111181934.png|300]]

## HorizontalBox
![[Pasted image 20230111203807.png]]
水平排列
![[Pasted image 20230111203835.png]]
同理VerticalBox是竖直排列

![[Pasted image 20230111213143.png]]
从左到右从上到下，矩阵式排列
## 控件模板
比如右上角击杀提示反复出现，用同一个模板
新建一个WidgetBlueprint

多加一个SizeBox
![[Pasted image 20230111204831.png]]
![[Pasted image 20230111204854.png]]

切换
![[Pasted image 20230111204902.png]]
制作结果如下：
![[Pasted image 20230111205005.png]]

Graph中新建五个变量
![[Pasted image 20230111205238.png]]

在Create Widget中可以暴露出来
![[Pasted image 20230111205357.png]]