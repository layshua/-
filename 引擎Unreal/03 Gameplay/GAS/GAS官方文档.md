设计计虚幻引擎的GAS时，有三大注意事项。
1. **追踪技能的所有者**（和网络复制中的 ownership 不是一个概念）：技能及其效果必须维持所有权的概念。当一种效果运行计算时，它需要知道其所有者是谁，以便使用其属性，当一种技能的作用会让玩家得分时，它需要知道哪个玩家拥有它，以便正确给分。
2. **追踪技能的状态**：
    - 技能激活时。
    - 技能目前正在执行中时。
    - 技能完全完成并且不再处于活动状态时。
3. **协调技能的执行**：技能必须能够在执行期间使用特定时序与多个不同的系统交互。这些交互可以包括：
# GAS的组件

GAS指在处理所有这些用例，方法是**将技能建模为负责自身执行的完全独立的实体**。

该系统由多个组件构成：
- 带有 **技能系统组件（`Ability System Component`）** 的所属Actor，维持该Actor拥有的所有技能的列表，并处理激活。
- **Gameplay 技能（`Gameplay Ability`）蓝图** ，表示各个技能，并协调其游戏内执行。
    - 由 **Gameplay技能任务（`Gameplay Ability Tasks`）** 以及其他函数构成。
- **属性集（`Attribute Set`）** ，附加到 Ability System Component。
    - 包含 **Gameplay属性（`Gameplay Attributes`）** ，用于驱动计算或表示资源。
- **Gameplay效果（`Gameplay Effects`）** ，处理Actor因使用技能而发生的更改。
    - **Gameplay效果计算（`Gameplay Effect Calculations`）** ，提供模块化、可复用的方法来计算效果。
    - **Gameplay提示（`Gameplay Cues`）** ，与Gameplay效果关联，并提供数据驱动的方法来处理视觉效果。

## Ability System Component

你需要将 `Ability System Component` 附加到 Actor，该 Actor 才能使用 Gameplay 技能。
- 此组件负责为 Actor 添加和去除技能，追踪 Actor 拥有的技能，以及激活技能。
- 它还是技能系统上下文中所属 Actor 的主要表示，提供一个系统来追踪属性、持续进行的效果、**Gameplay Tags（Gameplay Tags）** 和 **Gameplay Events（Gameplay Events）** 以及直接访问所属 Actor 的接口。  

在多人游戏中，技能系统组件还负责将信息复制到客户端，将玩家动作传达到服务器，并验证客户端是否有权更改技能系统组件的状态。技能系统组件的父Actor必须属于本地控制的玩家才能远程激活。

## Gameplay Ability

`Gameplay Ability` 是一个蓝图对象，**负责执行技能的所有事件**，包括播放动画，触发效果，从所有者获取属性，以及显示视觉效果。

### 授予和撤销技能
Granting / Revoking

**在 Actor 可以使用某项技能之前，必须向其 `Ability System Component` 授予该技能。**

**技能系统组件的以下函数可以`授予`对某项技能的访问：**
- **`GiveAbility`**：使用 `FGameplayAbilitySpec` 指定要添加的技能，并返回 `FGameplayAbilitySpecHandle`。
- **`GiveAbilityAndActivateOnce`**：使用 `FGameplayAbilitySpec` 指定要添加的技能，并返回 `FGameplayAbilitySpecHandle`。技能必须实例化，并且必须能够在服务器上运行。尝试在服务器上运行技能后，将返回 `FGameplayAbilitySpecHandle`。如果技能没有满足所需条件，或者无法执行，返回值将无效，并且技能系统组件将不会被授予该技能。

**以下函数可以利用授予技能后返回的 `FGameplayAbilitySpecHandle`，来`撤销`对技能系统组件中该技能的访问。**
- `ClearAbility`: 从技能系统组件中移除指定技能。
- `SetRemoveAbilityOnEnd`：当该技能执行完毕时，将该技能从技能系统组件中移除。如果未执行该技能，将立即移除它。如果正在执行该技能，将立即清除其输入，这样玩家就无法重新激活它或与它互动。
- `ClearAllAbilities`：从技能系统组件中移除所有技能。此函数是唯一不需要 `FGameplayAbilitySpecHandle` 的函数。

### 基本用法

将 `Gameplay Ability` 授予 Actor 的技能系统组件后，`Gameplay Ability` 的**基本执行生命周期**如下：

1. 即使调用者没有尝试执行某项技能，**`CanActivateAbility` 也可以让调用者知道是否可执行该技能。** 例如，您可以在户界面上将玩家无法使用的图标变成灰色并停用这些图标，或者对角色播放声音或粒子效果来显示某项技能是否可用。
2. **`CallActivateAbility` 执行技能相关的游戏代码**，但不会检查该技能是否可用（通常在 ` CanActivateAbility` 检查及执行技能之间需要某些逻辑时才会调用该函数）
    - 用户需要使用技能的定制功能覆盖的主代码要么是名为`ActivateAbility`的C++函数，要么是名为`Activate Ability`的蓝图事件。
    - 与 Actor 和组件不同，**玩法技能不会使用"`tick`"函数完成主要工作，而是在激活过程中启动技能任务，异步完成大部分工作**，然后通过委托（在 C++中）处理这些任务的输出，或者连接节点以输出执行引脚（在蓝图中）。
    - 如果**从 `Activate` 中调用 `CommitAbility` 函数**，它将应用执行**技能的消耗**，例如从玩法属性中减去资源（例如"魔法值"、"体力值"或游戏系统所用的任何其他资源）和应用冷却。
    - **`CancelAbility`提供了取消技能的机制，不过技能的`CanBeCanceled`函数可以拒绝请求**。与`CommitAbility`不同，该函数可供技能外调用者使用。成功的取消先播放给`On Gameplay Ability Cancelled`，然后通过标准代码路径结束技能，让技能可运行特殊的清理代码，否则取消时的行为将与自行结束时的行为不同。
3. **`TryActivateAbility` 是执行技能的典型方式。该函数调用 `CanActivateAbility` 来确定技能是否可以立即运行，如果可以，则继续调用 `CallActivateAbility`。**
4. **`EndAbility` 会在技能执行完毕后将其关闭**。如果技能被取消，`UGameplayAbility` 类会将其作为取消流程的一部分自动处理，但其他情况下，开发者都必须调用 C++函数或在技能的蓝图图表中添加节点。如果未能正常结束技能，将导致玩法技能系统认为技能仍在运行，从而带来一些影响，例如禁止将来再使用该技能或任何被该技能阻止的技能。例如，如果游戏的"喝生命药剂"玩法技能没有正常结束，那么使用该技能的角色就无法执行任何在喝血量药剂时无法执行的操作（例如喝其他药剂、快跑、爬梯子等）。这种阻碍会一直存在，因为玩法技能系统会认为角色还在喝药剂。

### Gameplay Tags

Gameplay Tags 有助于确定玩法技能之间的交互方式。每种技能都拥有一组标记，以可影响其行为的方式识别和分类技能，还有玩法标记容器和游戏标记查询，用于支持与其他技能进行交互。

|玩法标记变量|目的|
|---|---|
|Cancel Abilities With Tag|如果任何已在执行的技能带有与执行此技能时提供的列表匹配的标记，则取消那些技能。|
|Block Abilities With Tag|在执行此技能时，阻止执行具有匹配标记的任何其他技能。|
|Activation Owned Tags|在执行此技能时，技能的所有者将被给予这组标记。|
|Activation Required Tags|只有激活的Actor或组件具有所有这些标记时，技能才会被激活。|
|Activation Blocked Tags|只有激活的Actor或组件没有任何这些标记时，技能才会被激活|
|Target Required Tags|只有目标Actor或组件具有所有这些标记时，技能才会被激活。|
|Target Blocked Tags|只有目标Actor或组件没有任何这些标记时，技能才会被激活。|

### 复制

玩法技能支持复制内蕴状态和[玩法事件](https://docs.unrealengine.com/5.2/zh-CN/using-gameplay-abilities-in-unreal-engine#%E8%A7%A6%E5%8F%91%E7%8E%A9%E6%B3%95%E4%BA%8B%E4%BB%B6)，或关闭复制以节省网络带宽和缩短CPU周期。技能的 **玩法技能复制策略（Gameplay Ability Replication Policy）** 可以设置为"是"或"否"，这控制着技能是否复制自身实例、更新状态或跨网络发送玩法事件。对于复制技能的多人游戏，你可以选择复制的处理方式，这些选项被称为 **玩法网络执行策略（Gameplay Net Execution Policy）**：

1. **本地预测：（Local Predicted:）** 此选项有助于在响应能力和准确性之间实现良好的平衡。在本地客户端发出命令后，技能将立即在客户端上运行，但服务器起着决定性作用，并且可以根据技能的实际影响来覆盖客户端。客户端实际上是从服务器请求执行"技能"的权限，但也在本地进行处理，就像服务器需要同意客户端对结果的看法一样。因为客户端在本地预测技能的行为，所以只要客户端的预测与服务器不矛盾，它就会非常顺利地运行且无滞后。
    
2. **仅限本地：（Local Only:）** 客户端仅在本地运行技能。如果使用服务器的客户端是主机（在物理服务器计算机上播放）或者是在单人游戏中，尽管技能将在服务器上运行，也不会对服务器应用复制。这种情况不适用于专用服务器游戏，因为在专用服务器游戏中客户端永远不会在服务器计算机上运行。客户端通过该技能带来的任何影响都将继续遵循常规复制协议，包括可能从服务器接收更正。
    
3. **服务器启动：（Server Initiated:）** 在服务器上启动的技能将传播到客户端。从客户端的角度来看，这可以更准确地复制服务器上实际发生的情况，但使用技能的客户端会因缺少本地预测而发生延迟。虽然这种延迟非常短，但某些类型的技能（特别是在压力情况下快速执行的操作）将不会像在本地预测模式中那样顺畅。
    
4. **仅限服务器：（Server Only:）**"仅限服务器"技能将在服务器上运行，不会复制到客户端。被这些技能修改的任何变量都将像往常一样进行复制，这意味着该技能能够影响服务器的权威数据，并会将影响传递给客户端。以这种方式，技能仍然可以影响客户端的观察，尽管技能本身只在服务器上运行。

### 实例化策略

在执行玩法技能时，通常会产生一个（技能类型的）新对象，用于跟踪正在进行的技能。**在某些情况下可能会非常频繁地执行技能，会出现因快速创建技能对象而对性能产生负面影响的情况。**

为了解决这个问题，技能可以选择**三种不同的实例化策略**，以在效率和功能之间达到平衡。支持的三种实例化类型：

1. **按执行实例化：（Instanced per Execution:）** 每次技能运行时，都会产生技能对象的副本。
    - 优点：可以自由使用蓝图图表和成员变量，并且所有内容都将在执行开始时初始化为默认值。这是最简单的实例化策略，
    - 缺点：开销较大，该策略更适合不会频繁运行的技能。
2. **按 Actor 实例化：（Instanced per Actor:）** 在技能首次执行时，每个 Actor 会生成该技能的一个实例，该对象会在以后的执行中重复使用。这就要求在两次技能执行之间清理成员变量，同时也使保存多个执行的信息成为可能。按 Actor 是较为理想的复制方法，因为技能具有可处理变量和 RPC 的复制对象，而不是浪费网络带宽和 CPU 时间，在每次运行时产生新对象。**该策略适用于大规模的情况，因为大量使用技能的 Actor（例如在大型战斗中）只会在第一次使用技能时产生对象。**
3. **非实例化：（Non-Instanced:）** 这是所有类别中**最高效**的实例化策略。在运行时，技能不会生成任何对象，而是使用[类默认对象](https://docs.unrealengine.com/5.2/zh-CN/objects-in-unreal-engine)（CDO）。
    - 但是，在高效的同时也伴随着一些限制：
        - 首先，该策略特别**要求技能完全用 C ++编写**，因为创建蓝图图表需要对象实例。你可以创建非实例化技能的蓝图类，但这只能更改已公开属性的默认值。
        - 此外，在**执行技能期间**，即使在本机 C ++代码中，技能也**不能更改成员变量，不能绑定代理，也不能复制变量或处理 RPC。**
    - 该策略**仅适用于不需要内部变量存储**（尽管可以针对技能用户设置属性）**并且不需要复制任何数据的技能**。它尤其适合频繁运行且被许多角色使用的技能，例如大型 RTS 或 MOBA 作品中部队使用的基本攻击。
### Gameplay Events

**玩法事件（Gameplay Events）** 是可以传递的数据结构，能够直接触发玩法技能，无需通过正常通道，即可根据情境发送数据有效负载。
常用的方法是调用 `Send Gameplay Event To Actor` 并提供实施 `IAbilitySystemInterface` 接口的 Actor 和玩法事件所需的上下文信息。
但也可以直接在技能系统组件上调用 `Handle Gameplay Event`。因为这不是调用 `Gameplay Ability` 的正常方式，所以技能可能需要的上下文信息将通过 `FGameplayEventData` 数据结构传递。该结构是一种通用结构，不会针对任何特定的玩法事件或技能进行扩展，但应该能够满足任何用例的要求。多态 `ContextHandle` 字段会根据需要提供其他信息。 

> [!NOTE]
> 当 Gameplay Events 触发 Gameplay Ability 时， Gameplay Ability 不会通过 `Activate Ability` 代码路径运行，而是使用提供附加上下文数据作为参数的 `Activate Ability From Event`）。
> **如果希望技能响应游戏事件，请务必处理此代码路径**，同时还应注意，一旦在 Gameplay Ability 的蓝图中实施，`Activate Ability From Event`）将取代 `Activate Ability `

### 控制激活 Activate

你主要有四种方法 **激活（Activate）** `Gameplay Ability`：

1. **使用 Gameplay 技能句柄（`Gameplay Ability Handle`）** 通过蓝图或 C++代码显式激活技能。这在授予技能时由 `Ability System Component` 提供。
2. **使用 [[#触发 Gameplay Events]] 。这会使用匹配的技能触发器触发所有技能。如果你需要抽象输入和决策机制，此方法非常适合，因为它提供了最大的灵活度。
3. **通过匹配的 [[#Gameplay Tags]] 使用 Gameplay 效果**。这会使用匹配的技能触发器触发所有技能。这是<mark style="background: #FF5582A6;">触发技能的 Gameplay 效果的首选方法</mark>。典型用例是休眠减益，它触发的技能会播放禁用动画，并禁止其他游戏动作。
4. 使用 **输入代码（Input Codes）** 。这些会添加到 `Ability System Component`，在调用时会触发匹配的所有技能。其运作方式类似于 Gameplay Events。

> [!tip]
> `Gameplay Ability` 可以表示一组范围广泛的游戏内动作，不限于玩家显式使用的能力或咒语。击中反应或以上例子的休眠动画，都是很好的例子。

当你 **激活** `Gameplay Ability` 时，系统会将该技能识别为进行中。接着，它会触发 Attach 到激活事件的代码，遍历每个函数和 Gameplay Tasks，直到你调用 `Finish` 函数来表示技能已完成执行。如果你需要执行额外的清理，你可以将更多代码附加到 `On Finished` 事件。你还可以 `Cancel` 技能，使其在执行中途停止。

`Gameplay Ability` 使用 [[#Gameplay Tags]] 来限制执行。所有技能都有在激活时会添加到其所属 Actor 的标签列表，以及阻止激活或自动取消该技能的标签列表。虽然你可以使用自己的代码手动取消、阻止或允许技能的执行，但这里提供了在整个系统内一致的方法。 

### 控制执行 Execution

`Gameplay Ability` 支持各种常见用例，例如技能冷却和分配资源成本，并且有一个预制的 `Gameplay Ability Tasks` 库，用于处理动画和其他常见的引擎系统。

虽然标准蓝图函数节点会立即完成执行，但 `Gameplay Ability Tasks` 会追踪它们是处于不活动状态、进行中还是已完成，并且可以编程为在执行期间触发其他事件。它们还可以追踪其父 `Gameplay Ability` 是否已取消并相应清理。
游戏通过扩展 `Gameplay Ability Tasks` 来实现自定义 Gameplay 逻辑是很常见的做法。

`Gameplay Ability` 还可以响应 Gameplay Events，它们是通用事件监听器，等待从所属 Actor 接收 Gameplay Tags和 **事件数据（Event Data）** 结构体。

## 属性和属性集

> [!NOTE] 
> 必须在本地代码中创建属性和属性集——它们不能在蓝图中创建。

GAS主要通过 **属性集（Attribute Sets）** 与 Actor 交互，其中包含 **Gameplay 属性（Gameplay Attributes）** 。这些是可在计算中使用或由 Gameplay 技能修改的浮点值。它们可用于你需要的任意目的，但**常见用例包括追踪角色的生命值或击中点**，以及角色的核心统计数据值（例如力量和智能）。

虽然你可以使用基本变量来表示这些值，但 `Gameplay Attributes` 可带来多项**优势**：
- 属性集提供了一组一致、可复用的属性，可用于构建系统。
- Gameplay Ability 可以通过反射访问 Gameplay Attributes，以便可以直接在蓝图编辑器中创建简单的计算和效果。
- Gameplay Attributes 会**单独**追踪默认值、当前值和最大值，这样就更容易创建临时修改（增益和减益）以及持久效果。
- Gameplay Attributes 还会将其值复制到所有客户端，适合直观地显示敌方血条等本地 UI。

> [!quote] 使用方法
> - 要创建 `Gameplay Attributes`，你必须首先新建一个属性集（Attribute Set）。然后就可以添加到属性集中。
> - 要使 Actor 能够使用 `Gameplay Attributes`，你必须将其附加到其 `Ability System Component`。在此之后，`Ability System Component` 可以自动访问你分配给 `Attribute Sets` 的属性。  

> [!NOTE]
> 在某些情况下，游戏属性可以在没有属性集的情况下存在。这通常表明，某个游戏属性被保存在一个 **技能系统组件（Ability System Component）** 上，而这个组件没有一个包含适当类型游戏属性的属性集。这种方法并**不推荐**，因为游戏属性除了作为浮点值保存外，不会与GAS的任何部分交互。
### 定义和设置
首先，设置一个带有一个或多个 GamePlay Attribute 的属性集，然后将其注册到你的 Ability System Component 中。

1. 扩展基本属性集类 `UAttributeSet`，并将游戏玩法属性添加为 `UProperties() FGameplayAttributeData `，以下是一个简单的单个游戏玩法属性的属性集：
    ```c++
    UCLASS()
    class MYPROJECT_API UMyAttributeSet : public UAttributeSet
    {
        GENERATED_BODY()
    
        public:
        /** Sample "Health" Attribute, publicly accessible */
        UPROPERTY(EditAnywhere, BlueprintReadOnly)
        FGameplayAttributeData Health;
    };
    ```

2. **将属性集存储在 Actor 上**，并将使其对虚幻引擎开放。使用 **`const` 关键字** 来确保代码不能直接修改属性集，将其添加到你的 Actor 的类定义中：
    ```c++
    /** Sample Attribute Set. */
    UPROPERTY()
    const UMyAttributeSet* AttributeSet;
    ```

3. 把属性集注册到相应的 Ability System Component 中。这会在实例化属性集时**自动进行**，你可以在 Actor 的构造函数中进行，也可以在 `BeginPlay` 时进行，但前提是 Actor 的 `GetAbilitySystemComponent` 函数在实例化时返回一个有效的技能系统组件。
    - 你也可以编辑 Actor 的蓝图，并将属性集类添加到技能系统组件的默认起始数据中。
    - 第三种方法是指示技能系统组件实例化属性集，然后属性集会自动注册，以下就是一个案例：

    ```c++
    // 像这样的代码通常出现在BeginPlay()中，但也可以是
    // 获取相应的技能系统组件。它可能在另一个Actor上，所以使用GetAbilitySystemComponent并检查结果是否有效。
    AbilitySystemComponent* ASC = GetAbilitySystemComponent();
    // 确保AbilitySystemComponent有效。如果失败是不可接受的，用check()语句替换这个if()条件。
    if (IsValid(ASC))
    {
        // 从我们的技能系统组件中获取UMYAttributeSet。如有需要，技能系统组件将创建并注册一个UMYAttributeSet。
        AttributeSet = ASC->GetSet<UMyAttributeSet>();
    
        // 我们现在有了一个指向新的UMyAttributeSet的指向器，以后可以使用该指向器。如果它有初始化函数，这里是调用它的好地方。
    }
    ```
  

> [!warning] 注意
>- 一个 Ability System Component 可以有多个属性集，但每个属性集必须与所有其它属性集的**类**不同。
>- 如果使用 `Gameplay Effects`  来修改 Ability System Component **没有的 `Gamplay Attributes`** ，这样做 会使技能系统组件为自己创建一个匹配的游戏玩法属性。然而，这个方法并不会创建一个属性集，也不会将游戏玩法属性添加到任何现有的属性集中。


4. **Getter Setter 宏**。这是一个可选的步骤，添加一些基本的辅助函数来与游戏玩法属性交互。**最好是将游戏玩法属性本身做成受保护或私有的，而将与之交互的函数公开**。（也可以自定义 Getter Setter 函数，来控制属性访问）
    
| 宏（带参数）                                                 | 生成函数的签名                          | 行为/用途                                                  |
| ------------------------------------------------------------ | --------------------------------------- | ---------------------------------------------------------- |
|`GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UMyAttributeSet, Health)`| `static FGameplayAttribute GetHealth()` | 静态函数从虚幻引擎的反射系统中返回`FGameplayAttribute`结构 |
| `GAMEPLAYATTRIBUTE_VALUE_GETTER(Health)`                     | `float GetHealth() const`               | 返回"生命值"游戏玩法属性的当前值                           |
| `GAMEPLAYATTRIBUTE_VALUE_SETTER(Health)`                     | `void SetHealth(float NewVal)`          | 将"生命值"游戏玩法属性的值设置为`NewVal`                   |
| `GAMEPLAYATTRIBUTE_VALUE_INITTER(Health)`                    | `void InitHealth(float NewVal)`         | 将"生命值"游戏玩法属性的值初始化为`NewVal`                 |

添加完这些之后，你的属性集类定义应该是如下所示：

```c++
UCLASS()
class MYPROJECT_API UMyAttributeSet : public UAttributeSet
{
    GENERATED_BODY()

    protected:
    /** Sample "Health" Attribute */
    UPROPERTY(EditAnywhere, BlueprintReadOnly)
    FGameplayAttributeData Health;

    //~ ... Other Gameplay Attributes here ...

    public:
    //~ Helper functions for "Health" attributes
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UMyAttributeSet, Health);
    GAMEPLAYATTRIBUTE_VALUE_GETTER(Health);
    GAMEPLAYATTRIBUTE_VALUE_SETTER(Health);
    GAMEPLAYATTRIBUTE_VALUE_INITTER(Health);

    //~ ... Helper functions for other Gameplay Attributes here ...
};
```


### 使用数据表初始化

如果你选择不通过调用有硬编码值的初始化函数来初始化你的属性集和游戏玩法属性，你可以使用一个[数据表](https://docs.unrealengine.com/5.2/zh-CN/data-driven-gameplay-elements-in-unreal-engine)来初始化，使用`AttributeMetaData`类。你可以从外部文件导入数据，或者在编辑器中手动填充数据表。

![AttributeMetaData.png](https://docs.unrealengine.com/5.2/Images/making-interactive-experiences/gameplay-ability-system/GameplayAttributesAndAttributeSets/AttributeMetaData.jpg)

#### 导入数据表

开发者通常会从.csv文件中导入数据表，如下所示：

```c++
---,BaseValue,MinValue,MaxValue,DerivedAttributeInfo,bCanStack
MyAttributeSet.Health,"100.000000","0.000000","150.000000","","False"
```

>"MinValue"和"MaxValue"栏不会在默认的GAS插件中执行，这些值不会有任何影响。

![ImportAttributeMetaData.png](https://docs.unrealengine.com/5.2/Images/making-interactive-experiences/gameplay-ability-system/GameplayAttributesAndAttributeSets/ImportAttributeMetaData.jpg)

将.csv文件导入为数据表资产时，请选择"AttributeMetaData"行类型。

#### 手动填充数据表

如果你喜欢在虚幻编辑器中编辑数值，而不是在外部电子表格或文本编辑程序中编辑数值，你可以创建表格，然后像其它蓝图资产一样打开它来编辑数值。使用窗口顶部的"添加"按键为每个游戏玩法属性添加一行。请记住，命名惯例是"`AttributeSetName.AttributeName`"，也就是"属性集名称.属性名称"，而且是区分大小写的。

### 与 Gameplay Effects 互动

对游戏玩法属性的值进行控制的常见方法是处理与之相关的 `Gameplay Effects`

1. 首先在属性集的类定义中覆盖`PostGameplayEffectExecute`函数，该函数应该是公共访问级别的。
    
    ```c++
    void PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData& Data) override;
    ```
    
2. 在属性集的源文件中编写函数主体，务必要调用父类的执行。
    
    ```c++
    void UMyAttributeSet::PostGameplayEffectExecute(const struct FGameplayEffectModCallbackData& Data)
    {
        // 记得要调用父类的执行。
        Super::PostGameplayEffectExecute(Data);
    
        // 通过使用属性获取器来查看这个调用是否会影响生命值。
        if (Data.EvaluatedData.Attribute == GetHealthAttribute())
        {
            // 这个游戏玩法效果是改变生命值。应用它，但要先限制数值。
            // 在这种情况下，生命值的基础值不可是负值。
            SetHealth(FMath::Max(GetHealth(), 0.0f));
        }
    }
    ```

### 复制

对于多人游戏项目，你可以通过属性集复制游戏玩法属性，其方式类似于复制其它属性的方式。

1. 首先在属性集标头的属性定义中加入`ReplicatedUsing`[Specifier](https://docs.unrealengine.com/5.2/zh-CN/class-specifiers)，这将设置一个回调函数，有助于在远程系统上进行预测。
    
    ```c++
    protected:
    /** Sample "Health" Attribute */
    UPROPERTY(EditAnywhere, BlueprintReadOnly, ReplicatedUsing = OnRep_Health)
    FGameplayAttributeData Health;
    ```
    
2. 声明你的复制回调函数：
    
    ```c++
    /** 当新的生命值达到网络时被调用 */
    UFUNCTION()
    virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
    ```
    
3. 在属性集的源文件中，定义你复制的回调函数。函数的主体可以用游戏玩法技能系统定义的一个宏来表示。
    
    ```c++
    void UMyAttributeSet::OnRep_Health(const FGameplayAttributeData& OldHealth)
    {
        // 使用默认的游戏玩法属性系统更新通知行为。
        GAMEPLAYATTRIBUTE_REPNOTIFY(UMyAttributeSet, Health, OldHealth);
    }
    ```
    
4. 如果这是你的属性集中的首个复制的属性，你要对公共的`GetLifetimeReplicatedProps`函数设置一个覆盖。
    
    ```c++
    /** Marks the properties we wish to replicate */
    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;
    ```
    
5. 将游戏玩法属性添加到属性集源文件中的`GetLifetimeReplicatedProps`函数中，如下所示：
 
    ```c++
    void UMyAttributeSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
    {
        // 调用父函数。
        Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    
        // 为生命值添加复制功能。
        DOREPLIFETIME_CONDITION_NOTIFY(UMyAttributeSet, Health, COND_None, REPNOTIFY_Always);
    }
    ```


## Gameplay Cues

**`Gameplay Cues` 是负责运行视觉和声音效果的 Actor 和 UObject，是在多人游戏中复制美化 (Cosmetic)反馈的首选方法。**


-  **Gameplay Cue Display 显示**：`Gameplay Cue` 是可通过 GAS 控制的管理装饰效果（例如，粒子或音效）的方法，它可以节约网络资源。游戏性技能和游戏性效果可以触发它们，它们通过四个可在本地或蓝图代码中覆盖的主函数来产生作用：On Active、While Active、Removed 及 Executed（仅由游戏性效果使用）。**所有 `GameplayCue` 必须与"GameplayCue"开头的游戏性标记相关联，例如"GameplayCue. ElectricalSparks"或"GameplayCue. WaterSplash. Big"。

创建 `Gameplay Cues` 时，你会运行要在事件图表中播放的效果的逻辑。`Gameplay Cues` 可以与一系列 `Gameplay Tags`关联，并且匹配这些标签的 `Gameplay Effects` 将自动应用它们。

例如，**如果你将标签 Ability.Magic.Fire.Weak 添加到 Gameplay 提 `Gameplay Cues`，拥有 Ability.Magic.Fire.Weak 的 `Gameplay Effects` 将自动生成该 `Gameplay Cues` 并运行它**。这样就可以快速轻松创建视觉效果的通用库，而不必手动从代码触发它们。
 
或者，你也可以触发没有 Gameplay Effects 关联的 Cue 。有关此实现的例子，你可以查看 Lyra 示例游戏的武器发射反馈。

`Gameplay Cues` **不使用可靠的复制**，因此有可能一些客户端没有接收到提示或显示其反馈。如果你将 Gameplay 代码绑定到这些 Cue，这可能造成不同步。因此，`Gameplay Cues` 应该仅用于美化反馈。对于需要复制到所有客户端的 Gameplay 相关反馈，你应该转而依赖 Ability Tasks 来处理复制。**播放蒙太奇（Play Montage）** 技能任务就是很好的例子。 

# 支持网络多玩家 (鸽)

Gameplay技能提供了一些内置功能来支持联网多人游戏，但存在一些局限性和指南，你应该加以注意。其中许多指南反映了一般网络多人游戏最佳实践。

### 技能系统组件和复制

为节省带宽和防止作弊，技能系统组件不将其完整状态复制到所有客户端。具体而言，它们不将技能和Gameplay效果复制到所有客户端，只复制它们影响的Gameplay属性和标签。

### 复制技能和使用预测

网络游戏中的大部分技能都应该在服务器上运行并复制到客户端，因此技能激活时通常有延迟。这在大部分快节奏的多人游戏中是不能接受的。要掩盖这种延迟，你可以在本地激活技能，然后向服务器表明你已将其激活，以便服务器可以跟上。

有可能服务器会拒绝技能激活，这意味着它必须撤销技能在本地做出的更改。你可以使用 **本地预测的** 技能处理这些情况。为提供帮助，当授予效果的技能被服务器拒绝时，一些Gameplay效果支持回滚。这包括大部分非瞬时Gameplay效果，但要注意，伤害以及其他瞬时属性/标签更改等不包括在内。

### 使用技能与服务器拥有的对象交互

Gameplay技能可以处理与机器人、NPC以及其他由服务器拥有的Actor和对象之间的交互。你必须通过本地拥有的Actor（通常是玩家的Pawn）以及复制的技能或调用服务器函数的另一个非GAS机制来执行此操作。这会将交互复制到服务器，后者接着有权执行对NPC的更改。