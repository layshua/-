
# 事件

## Actor
虚幻引擎 Actor 有一组类似于 Unity 的 `Start` 、 `OnDestroy` 和 `Update` 函数的方法。如下所示：

```c++
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

    // 游戏开始时调用。
    void BeginPlay();

    // 销毁时调用。
    void EndPlay(const EEndPlayReason::Type EndPlayReason);

    // 每帧调用以更新此Actor。
    void Tick(float DeltaSeconds);
};
```

![[cfedafa655eab03ddf40662aa013c483_MD5.jpg]]

## 组件
虚幻引擎中的组件包含不同的函数。下面是一个基本示例：

```c++
UCLASS()
class UMyComponent : public UActorComponent
{
    GENERATED_BODY()

    // 在创建所属Actor之后调用
    void InitializeComponent();

    // 在销毁组件或所属Actor时调用
    void UninitializeComponent();

    // 组件版的Tick函数
    void TickComponent(float DeltaTime, enum ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction);
};
```

![[267e8d4e22867a4951b46b9b18c86926_MD5.jpg]]

记住，在虚幻引擎中，你必须调用方法的父类版本。

例如，在 Unity C# 中，可能是调用 `base.Update()`，但在虚幻引擎 C++ 中，你将使用 `Super::TickComponent()`：

```c++
void UMyComponent::TickComponent(float DeltaTime, enum ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
    // 此处添加自定义更新内容
    Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
}
```

在 C++ 中，一些类使用以 `A` 开头，而另一些类以 `U` 开头。`A` 表示 Actor 子类，而 `U` 表示 **Object** 子类。另一个常用前缀是 `F` ，它用于大部分普通数据结构和非 UObject 类。

# 游戏逻辑
## 实例化 Object/Actor
虚幻引擎有两个不同的函数来实例化对象：

- `NewObject` 用于创建新的 `UObject` 类型。
- `SpawnActor` 用于生成 `AActor` 类型。
    
### UObject 和 NewObject

在虚幻引擎中创建 `UObject` 的子类与在 Unity 中创建 `ScriptableObject` 的子类非常类似。对于不需要生成到世界中或像 Actor 那样绑定了组件的 Gameplay 类，这些很有用。

```c++
//NewObject
//接收一个外部UObject或UClass，用自动生成的名称创建新实例
UMyObject* NewObj = NewObject<UMyObject>(); 

//NewNamedObject
//允许为新实例指定一个名称以及对象标记和一个要指定为参数的模板对象
UMyObject* NewObj = NewNamedObject<UMyObject>();

//ConstructObject
//更灵活
UMyObject* NewObj = ConstructObject<UMyObject>();
```

### AActor 和 SpawnActor
[Spawning Actors in Unreal Engine | 虚幻引擎5.2文档](https://docs.unrealengine.com/5.2/zh-CN/spawning-actors-in-unreal-engine/)
Actor 使用 World（C++ 中的 `UWorld`）对象上的 `SpawnActor` 方法生成。一些 UObject 提供了 `GetWorld` 方法（例如，所有 Actor 都如此）。你会采用此方法获取 World 对象。

请注意，在下面的示例中，我们传入了我们想生成的 Actor 的类，而不是传入另一个 Actor。在我们的示例中，该类可以是 AMyEnemy 的任意子类。

要是你想创建另一个对象的副本，就像在 Unity 的 Instantiate 函数那样，你该怎么做呢？

`NewObject` 和 `SpawnActor` 函数也能给一个 "模板" 对象来工作。虚幻引擎将创建该对象的副本，而不是从头创建新对象。这将复制其所有 UPROPERTY 和组件。

```c++
AMyActor* CreateCloneOfMyActor(AMyActor* ExistingActor, FVector SpawnLocation, FRotator SpawnRotation)
{
    UWorld* World = ExistingActor->GetWorld();
    FActorSpawnParameters SpawnParams;
    SpawnParams.Template = ExistingActor;
    World->SpawnActor<AMyActor>(ExistingActor->GetClass(), SpawnLocation, SpawnRotation, SpawnParams);
}
```
你可能会好奇这里的 "从头开始" 到底是什么意思。你创建的每个对象类都有一个默认模板，其中包含其属性和组件的默认值。如果你不覆盖这些属性，也没有提供你自己的模板，虚幻引擎将使用这些默认值来构造你的对象。

```c++
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

    UPROPERTY()
    int32 MyIntProp;

    UPROPERTY()
    USphereComponent* MyCollisionComp;

    AMyActor()
    {
        MyIntProp = 42;

        MyCollisionComp = CreateDefaultSubobject<USphereComponent>(FName(TEXT("CollisionComponent"));
        MyCollisionComp->RelativeLocation = FVector::ZeroVector;
        MyCollisionComp->SphereRadius = 20.0f;
    }
};
```

在构造函数 `AMyActor` 中，我们为这个类设置了属性的默认值。请注意 `CreateDefaultSubobject` 函数的用法，我们可以使用此函数创建组件并向其分配默认属性。使用此函数创建的所有子对象都充当默认模板，因此我们可以在子类或蓝图中修改它们。

## 类型转换

在此例中，我们获取了一个已知的组件，将其转换为特定类型，然后判断能否执行一些操作。


```c++
UPrimitiveComponent* Primitive = 
MyActor->GetComponentByClass(UPrimitiveComponent::StaticClass());

USphereComponent* SphereCollider = Cast<USphereComponent>(Primitive);
if (SphereCollider != nullptr)
{
        // ...
}
```

## 销毁 Actor

```c++
MyActor->Destroy();
```

## 延迟销毁 Actor（延迟 1 秒）

```c++
MyActor->SetLifeSpan(1);
```

## 禁用 Actor
```c++
// 隐藏可见组件
MyActor->SetActorHiddenInGame(true);

// 禁用碰撞组件
MyActor->SetActorEnableCollision(false);

// 禁止Actor更新
MyActor->SetActorTickEnabled(false);
```
## 从组件访问 Actor
```c++
AActor* ParentActor = MyComponent->GetOwner();
```
## 从 Actor 访问组件


```c++
UMyComponent* MyComp = MyActor->FindComponentByClass<UMyComponent>();
```

![[e5c08b782929e1ae15f583195a515fb2_MD5.jpg]]


## 查找 Actor/UObject

```c++
// 按名称查找Actor（也适用于UObject）
AActor* MyActor = FindObject<AActor>(nullptr, TEXT("MyNamedActor"));

// 按类型查找Actor（需要UWorld对象）
for (TActorIterator<AMyActor> It(GetWorld()); It; ++It)
{
    AMyActor* MyActor = *It;
    // ...
}
```

![[52ee0d707b4ee7d5c92ad8ebc0c1cbc8_MD5.jpg]]

```c++
// 按类型查找UObject
for (TObjectIterator<UMyObject> It; It; ++it)
{
    UMyObject* MyObject = *It;
    // ...
}

// 按标签查找Actor（也适用于ActorComponent，需要改用TObjectIterator）
for (TActorIterator<AActor> It(GetWorld()); It; ++It)
{
    AActor* Actor = *It;
    if (Actor->ActorHasTag(FName(TEXT("Mytag"))))
    {
        // ...
    }
}
```

![[573de7ae173939da688330a4b765228d_MD5.jpg]]

## 添加标签 Actor/ActorComponent 

```c++
// Actor可以有多个标签
MyActor.Tags.AddUnique(TEXT("MyTag"));
```

![[28af4af0a61c63fee028cc86dd959a4c_MD5.jpg]]

```c++
// 组件有自己的标签数组
MyComponent.ComponentTags.AddUnique(TEXT("MyTag"));
```
## 比较标签 Actor/ActorComponent 

```c++
// 检查某个Actor是否有此标签
if (MyActor->ActorHasTag(FName(TEXT("MyTag"))))
{
    // ...
}
```

![[9c593e164aa04ca88b54e89ce83e02e3_MD5.jpg]]

```c++
// 检查某个ActorComponent是否有此标签
if (MyComponent->ComponentHasTag(FName(TEXT("MyTag"))))
{
    // ...
}
```

![[d1da59ffb195362329516a944aeabc1a_MD5.jpg]]

### 物理：刚体与图元组件
`RigidBody`/`Primitive Component`

在 Unity 中，假如要为 GameObject 赋予物理特征，首先必须为其提供刚体组件。

在虚幻引擎中，任何图元组件（C++ 中的 `UPrimitiveComponent`）都可以是物理对象。
一些常见图元组件如下：
- 形状组件（胶囊体、球体和盒体）
- 静态网格体组件
- 骨骼网格体组件

Unity 将碰撞和可视性划分到不同的组件中，虚幻引擎则将 **"潜在的物理碰撞"（potentially physical）** 和 **"潜在的可视效果"（potentially visible）** 组合到了单个图元组件中。凡是在世界中具有形状的组件，只要能通过物理方式渲染或交互，都是 `PrimitiveComponent` 的子类。


### 刚体运动

在虚幻引擎 4 中，碰撞组件和刚体组件是同一个组件。其基类是 `UPrimitiveComponent`，它有许多子类（`USphereComponent`、`UCapsuleComponent` 等）可满足你的需要。

```c++
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

    UPROPERTY()
    UPrimitiveComponent* PhysicalComp;

    AMyActor()
    {
        PhysicalComp = CreateDefaultSubobject<USphereComponent>(TEXT("CollisionAndPhysics"));
        PhysicalComp->SetSimulatePhysics(false);
        PhysicalComp->SetPhysicsLinearVelocity(GetActorRotation().Vector() * 100.0f);
    }
};
```
### RayTrace
```c++
APawn* AMyPlayerController::FindPawnCameraIsLookingAt()
{
    // 你可以在这里自定义有关追踪的各种属性
    FCollisionQueryParams Params;
    // 忽略玩家的Pawn
    Params.AddIgnoredActor(GetPawn());

    // 击中结果由线路追踪填充
    FHitResult Hit;

    // 来自摄像机的Raycast，仅与Pawn碰撞（它们在ECC_Pawn碰撞通道上）
    FVector Start = PlayerCameraManager->GetCameraLocation();
    FVector End = Start + (PlayerCameraManager->GetCameraRotation().Vector() * 1000.0f);
    bool bHit = GetWorld()->LineTraceSingle(Hit, Start, End, ECC_Pawn, Params);

    if (bHit)
    {
        // Hit.Actor包含指向追踪所击中的Actor的弱指针
        return Cast<APawn>(Hit.Actor.Get());
    }

    return nullptr;
}
```
[![[4e3dc3bf152452bb0663afe8869c952a_MD5.jpg]]
### 触发器体积 Trigger Volumes
```c++
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

    // 我的触发器组件
    UPROPERTY()
    UPrimitiveComponent* Trigger;

    AMyActor()
    {
        Trigger = CreateDefaultSubobject<USphereComponent>(TEXT("TriggerCollider"));

        // 两个碰撞物都需要将此项设置为true，才能触发事件
        Trigger.bGenerateOverlapEvents = true;

        // 设置碰撞物的碰撞模式
        // 此模式仅为光线投射、扫描和重叠启用碰撞物
        Trigger.SetCollisionEnabled(ECollisionEnabled::QueryOnly);
    }

    virtual void NotifyActorBeginOverlap(AActor* Other) override;

    virtual void NotifyActorEndOverlap(AActor* Other) override;
};
```

![[9ea03060c7ec432c5c18a49982f9d1ee_MD5.jpg]]


### 输入事件
```c++
UCLASS()
class AMyPlayerController : public APlayerController
{
    GENERATED_BODY()

    void SetupInputComponent()
    {
        Super::SetupInputComponent();

        InputComponent->BindAction("Fire", IE_Pressed, this, &AMyPlayerController::HandleFireInputEvent);
        InputComponent->BindAxis("Horizontal", this, &AMyPlayerController::HandleHorizontalAxisInputEvent);
        InputComponent->BindAxis("Vertical", this, &AMyPlayerController::HandleVerticalAxisInputEvent);
    }

    void HandleFireInputEvent();
    void HandleHorizontalAxisInputEvent(float Value);
    void HandleVerticalAxisInputEvent(float Value);
};
```

![[980dbfd9b4d7a19784b8e4608a0ca40a_MD5.jpg]]

你的项目设置中的输入属性可能如下所示：

![[83bcde708ee30addd568e23d2a180ccc_MD5.jpg]]

# 委托
```c++ file:BossActor.h

```