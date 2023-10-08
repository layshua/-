---
title: 0 GamePlay精粹
create_time: 2023-09-27 15:33
uid: "202309271533"
reference: []
banner: "[[Pasted image 20230928150745.png]]"
banner_y: 0.322
banner_header: 
banner_icon: 🎮
---
# 虚幻类
## UObject

### 创建 Object

- **`UObject` 构造函数不支持参数**。所有的 C++ `UObject` 都会在引擎启动的时候初始化，然后引擎会调用其默认构造器。如果没有默认的构造器，那么 `UObject` 将不会编译。
-  `UObject` 构造函数应该轻量化，仅用于设置默认的数值和子对象，构造时不应该调用其它功能和函数。
- **`UObject` 永远都不应使用 `new` 运算符。** 所有的 UObjects 都由虚幻引擎管理内存和垃圾回收。如果通过 new 或者 delete 手动管理内存，可能会导致内存出错。

---

- **`NewObject<T>`** ：创建一个 `UObject` 实例，仅在运行时使用
- **`CreateDefaultSubobject<T>`** ：创建一个组件或者子对象，在构造函数中使用

例子：创建静态网格体
```c++
// 构造函数中

// 创建网格体组件，以便查看球体位置  
UStaticMeshComponent* SphereVisual = CreateDefaultSubobject<UStaticMeshComponent> (TEXT("VisualRepresentation"));    

//依附到根组件
SphereVisual->SetupAttachment(RootComponent); 

//加载资源
static ConstructorHelpers::FObjectFinder<UStaticMesh> SphereVisualAsset(TEXT("/Game/StarterContent/Shapes/Shape_Sphere.Shape_Sphere"));  

if(SphereVisualAsset.Succeeded())  
{  
    SphereVisual->SetStaticMesh(SphereVisualAsset.Object);  
    SphereVisual->SetRelativeLocation(FVector(0.0f, 0.0f, -40.0f));  
    SphereVisual->SetWorldScale3D(FVector(0.8f));  
}

```

### 更新 Object

Ticking 代表虚幻引擎中对象的更新方式。所有 Actors 均可在每帧被 tick，便于您执行必要的更新计算或操作。

-  `Actor` 和 Actor 组件在注册时会自动调用它们的 Tick 函数
- **`UObjects` 不具有嵌入的更新能力。在必须的时候，可以使用 `inherits` 类说明符从 `FTickableGameObject` 继承即可添加此能力。**  这样即可实现 `Tick()` 函数，引擎每帧都将调用此函数。

### 销毁 Object

**`UPROPERTY` 宏定义对象不被引用后，垃圾回收系统将自动进行对象销毁。** 
垃圾回收器会定期从根节点 Root 开始检查，当一个 UObject 没有被别的任何 UObject 引用，就会被垃圾回收。你可以通过 `AddToRoot` 函数来让一个 UObject 一直不被回收。

**主动销毁的方法：**
- `UObject::ConditionalBeginDestroy()`
    - 异步执行且对象在当前帧内持续有效
    - 等待下次GC
```c++
Obj->ConditionalBeginDestroy();
Obj = nullptr;
```

- `Obj->MarkAsGarbage()` ：标记为垃圾
    - 标记为垃圾等待回收。如果 `gc.PendingKillEnabled=true` ，那么所有标记为 `PendingKill` 的对象会被垃圾回收器自动清空并销毁。

### 查找
#### 按类型查找 Object
```c++
// 按类型查找UObject
for (TObjectIterator<UMyObject> It; It; ++it)
{
    UMyObject* MyObject = *It;
    // ...
}
```
#### 从文件查找 Object
```c++
//从文件查找Object
static ConstructorHelpers::FObjectFinder<UStaticMesh> CylinderAsset(TEXT("/Game/StarterContent/Shapes/Shape_Cylinder.Shape_Cylinder")); 
```

## AActor 
对于 Actor 和 Actor 组件，初始化功能应该输入 **`BeginPlay()`** 方法。
### 实例化 Actor

[Spawning Actors in Unreal Engine | 虚幻引擎5.2文档](https://docs.unrealengine.com/5.2/zh-CN/spawning-actors-in-unreal-engine/)

- Actor 通过 `UWorld` 对象（可以通过 `GetWorld()` 获得）的 **`SpawnActor`** 方法生成。
>UObject 为 Actor 提供了 `GetWorld` 方法

```c++
AKAsset* SpawnedActor1 =
(AKAsset*) GetWorld()->SpawnActor(AKAsset::StaticClass(), NAME_None, &Location);
```

`::StaticClass()` 以 `UCLASS*` 的形式为我们提供一个原始 C++类 

```c++
TSubclassOf<ATreasure> TreasureClasses;

UWorld* World = GetWorld();  
if(World)
{
    World->SpawnActor<ATreasure>(TreasureClasses, Location, GetActorRotation());
}
```
### 销毁 Actor

```c++
MyActor->Destroy(); //AActor销毁
```

即使 Actor 被调用了 `Destroy()`，并且被从关卡中移除，它还是会等到所有对它的引用都解除之后才会被垃圾回收。

### 生命周期
延迟 1s 销毁

- 构造函数初始化生命周期 `InitialLifeSpan`
- BeginPlay 里使用 `SetLifeSpan()` 设置生命周期
```c++
InitialLifeSpan = 8.0f;  

MyActor->SetLifeSpan(1); //延迟1s销毁
```

### 禁用 Actor
```c++
// 隐藏可见组件
MyActor->SetActorHiddenInGame(true);

// 禁用碰撞组件
MyActor->SetActorEnableCollision(false);

// 禁止Actor更新
MyActor->SetActorTickEnabled(false);
```

### 查找/获取 Actor
#### 单个Actor
```c++
// 按名称查找Actor（也适用于UObject）
AActor* MyActor = FindObject<AActor>(nullptr, TEXT("MyNamedActor"));

// 按类型查找Actor（需要UWorld对象）
// 使用 Actor 迭代器
for (TActorIterator<AMyActor> Iter(GetWorld()); Iter; ++Iter)
{
    //查找
    AMyActor* MyActor = *Iter;
    
   // 直接调用这个Actor的某个你需要成员函数等
   Iter->YourFunction();
   //等价
   *(Iter).YourFunction(); 
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

![[52ee0d707b4ee7d5c92ad8ebc0c1cbc8_MD5.jpg]]

![[573de7ae173939da688330a4b765228d_MD5.jpg]]

#### Actor 数组
![[Pasted image 20230903002855.png]]
```c++
TArray<AActor*> ActorsToFind;  
if(UWorld* World= GetWorld())  
{
UGameplayStatics::GetAllActorsOfClassWithTag(GetWorld(),AFireEffect::StaticClass(),FName("FireTag"),ActorsToFind);
}
```
### 标签Tags
#### 添加标签 Actor/ActorComponent 
```c++
Tags.Add(FName("FireTag"));
```

```c++
// Actor可以有多个标签
MyActor.Tags.AddUnique(TEXT("MyTag"));
```

![[28af4af0a61c63fee028cc86dd959a4c_MD5.jpg]]

```c++
// 组件有自己的标签数组
MyComponent.ComponentTags.AddUnique(TEXT("MyTag"));
```
#### 比较标签 Actor/ActorComponent 

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

### Damage
`take damage` 函数
![[Pasted image 20230911143017.png]]

###### DamageType 损坏类型

顾名思义，损伤类型是一种用于描述损伤“类型”的对象，与损伤的起源无关。**如果你有很多损坏源，并且你想要它们之间有共同的功能，这可能是一个非常有用的概念。**


一个简单的例子可以说明这一点，那就是火灾造成的损失。比方说，你希望任何受到火灾伤害的人大喊“哇，太热了”，然后跑到最近的水里。  

与其将代码复制到每一个可以烧毁玩家的角色（或每一种可能被烧毁的角色）中，您可以定义火的伤害类型（UDamageTypeFire），赋予它某种类型的HandleDamagedCharacter（）函数，并从TakeDamage（）调用链中适当地调用它。

###### Instigator 煽动者

`Instigator`是造成损害的人，通常是 PlayerController 或 AIController。在火灾损坏的情况下，可能是玩家或 AI 点燃了火。

###### DamageCauser 损坏原因

`DamageCauser` 通常是造成损坏的原因，比如你刚刚走过的 ACampFire actor。

##### Damage in C++ C++中的伤害

Let’s look first at damage support in native code.   
让我们先来看看本机代码中的损坏支持。

In this case, damaging an actor is simple -- just call TakeDamage() on it.  
在这种情况下，损坏一个 actor 很简单——只需对其调用 TakeDamage（）即可。

```c++
virtual float TakeDamage(float DamageAmount, struct FDamageEvent const& DamageEvent, class AController* EventInstigator, class AActor* DamageCauser);  
```

同样，为了响应损坏，只需在接收 actor 上重写 `TakeDamage()` 并插入自定义处理即可。

您会注意到TakeDamage（）调用接受DamageEvent作为参数。此`FDamageEvent`数据结构包含有关损坏事件的特定情况的数据，以便您的响应代码能够做出适当的反应。**UE4内置了三种类型的伤害事件。**

###### FPointDamageEvent  点伤害事件

点损伤事件模拟在受害者特定点施加的损伤，例如子弹或重拳。它包含撞击的方向和描述表面撞击的FHitResult。

###### FRadialDamageEvent 径向伤害事件

径向损伤事件模拟点源的径向损伤，爆炸就是一个明显的例子。它包含爆炸的震中、描述空间中损失衰减的数据以及受影响组件的列表。

###### FDamageEvent F损坏事件

这是可用的最通用的损伤模型，**只包含一个可选的DamageTypeClass。**

如果这些内置事件类型都不能满足您的需求，那么您**可以从FDamageEvent派生自己的结构，并存储所需的任何数据。**

##### Damage in Blueprints 蓝图中的损坏

在蓝图中处理损坏是类似的，只是损坏应用程序和响应已经按事件类型进行了分解。有可全局访问的节点可用于造成伤害，如ApplyDamage、ApplyPointDamage和ApplyRadialDamage。为了响应损坏事件，该级别中的actor类和actor实例都有一组类似的“受到损坏”事件。

如果您为项目定义自定义损坏事件，您可能希望公开一组类似的函数和委托，以便在蓝图中使用。
## Actor 组件
### 注册
#### 注册组件
引擎必须注册组件，才能让 Actor 组件能够逐帧更新。如果在 Actor 产生过程中，作为 Actor 子对象自动创建了组件，则这类组件会自动注册。
但是游戏期间创建的组件可以使用手动注册。`RegisterComponent` 函数提供了这个功能，要求是组件与 Actor 关联。

> [!NOTE] Title
> 游戏期间注册组件可能会影响性能，因此只应在必要时进行此操作。
#### 注册事件

在注册组件的过程中，引擎会将组件与场景关联起来，让其可用于逐帧更新，并运行以下 `UActorComponent` 函数：

|函数|描述|
|---|---|
|`OnRegister`|在注册组件时，可以覆写此函数来添加代码。|
|`CreateRenderState`|初始化组件的[渲染状态](https://docs.unrealengine.com/4.26/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/Components/#%E5%9C%BA%E6%99%AF%E4%BB%A3%E7%90%86)。|
|`OnCreatePhysicsState`|初始化组件的[物理状态](https://docs.unrealengine.com/4.26/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/Components/#%E7%89%A9%E7%90%86%E7%8A%B6%E6%80%81)。|

#### 取消注册

要从更新、模拟或渲染过程中移除 Actor 组件，可以使用 `UnregisterComponent` 函数将其取消注册。

### 更新
组件能够以类似于 Actor 的方法逐帧更新。`TickComponent` 函数允许组件逐帧运行代码。
例如，**USkeletalMeshComponent** 使用其 `TickComponent` 函数来更新动画和骨架控制器，而 **UParticleSystemComponent** 使用 `TickComponent`更新其发射器和处理粒子事件`。

**默认情况下，Actor 组件不更新**。

**为了让 Actor 组件逐帧更新：**
1. 必须在构造函数中将 `PrimaryComponentTick.bCanEverTick` 设置为 `true` 来启用 tick。
2. 之后，在构造函数中或其他位置处，必须调用 `PrimaryComponentTick.SetTickFunctionEnable(true)` 以开启更新。
3. 之后可调用 `PrimaryComponentTick.SetTickFunctionEnable(false)` 停用 tick。
4. 如果您知道组件永远不需要更新，或者打算手动调用自己的更新函数（也许从拥有的 Actor 类），将 `PrimaryComponentTick.bCanEverTick` 保留为默认值 `false` 即可，这样可以稍微改善性能。
#### 渲染状态

为进行渲染，Actor 组件必须创建渲染状态。此渲染状态还会告诉引擎，需要更新渲染数据的组件已发生变更。当发生此类变更时，渲染状态会被标记为"dirty"。
如果编译您自己的组件，可以使用 `MarkRenderStateDirty` 函数将渲染数据标记为 dirty。在一帧结束时，所有 dirty 组件的渲染数据都会在引擎中更新。
**场景组件（包括 Primitive 组件）默认会创建渲染状态，而 Actor 组件则不会。**

#### 物理状态

要与引擎的物理模拟系统交互，Actor 组件需要物理状态。物理状态会在发生变化时立即更新，防止出现"帧落后"瑕疵等问题，也不需要"dirty"标记。
**默认情况下，`UActorComponent` 和 `USceneComponent` 没有物理状态，但 `UPrimitiveComponent` 有。** 

覆盖 `ShouldCreatePhysicsState` 函数以确定组件类实例是否需要物理状态。
- 如果类使用物理，则不建议只返回 `true`。请参阅函数的 `UPrimitiveComponent` 版本，了解不应创建物理状态的情况（例如在组件破坏期间）。
- 在正常返回 `true` 的情况下，还可以返回 `Super::ShouldCreatePhysicsState`。

### 可视化组件
 
**可视化组件**：**只在编辑器中工作时存在的普通组件。** 用于辅助开发，在编辑器中运行时或运行打包版本时不会打包这些组件。

**创建可视化组件：**
- 创建常规组件并在其上方调用 `SetIsVisualizationComponent`。
- 由于组件无需存在于编辑器之外，所有对它的引用都应当处在对 `WITH_EDITORONLY_DATA` 或 `WITH_EDITOR` 的预处理器检查之中。这将确保打包版本不受这些组件的影响，并保证不会在代码中的任何位置引用它们。举例而言，**摄像机组件** 使用多个其他组件来在编辑器中显示实用信息，包括用于显示视图视锥的 **绘制视锥组件**。在头文件中，绘制视锥组件在类中进行如下定义：
```c++
#if WITH_EDITORONLY_DATA
    // 用于显示摄像机视野所在位置的视锥组件
    class UDrawFrustumComponent* DrawFrustum;
    // ...
#endif
```
`DrawFrustum` 现仅存在于编辑器中，被视为可视化组件，即在编辑器中进行游戏测试时不会显示。

- 同样，对这个组件的所有引用应当位于源文件中对 `WITH_EDITORONLY_DATA` 的预处理器检查之中。`OnRegister` 中的 `WITH_EDITORONLY_DATA` 检查内部的这段代码，将检查确认摄像机组件是否连接到有效 Actor，然后添加绘制视锥组件代码：

```c++
void UCameraComponent::OnRegister()
{
#if WITH_EDITORONLY_DATA
    if (AActor* MyOwner = GetOwner())
    {
        // ...
        if (DrawFrustum == nullptr)
        {
            DrawFrustum = NewObject<UDrawFrustumComponent>(MyOwner, NAME_None, RF_Transactional | RF_TextExportTransient);
            DrawFrustum->SetupAttachment(this);
            DrawFrustum->SetIsVisualizationComponent(true);
            // ...
        }
    }
    // ...
#endif
    Super::OnRegister();
    // ...在此处编写其他代码（在所有版本中运行）...
}
```

### 销毁

```c++
MyActorComponent->DestroyComponent(); //销毁UActorComponent
```

### 获取组件依附的 Actor

```c++
AActor* ParentActor = MyComponent->GetOwner();
```
### 获取 Actor 上的组件

```c++
UMyComponent* MyComp = MyActor->FindComponentByClass<UMyComponent>();
```

![[e5c08b782929e1ae15f583195a515fb2_MD5.jpg]]

### 设置组件层级关系
- **`RootComponent` 或 `SetRootComponent()`**：设置根组件
- **`SetupAttachment`**：将场景组件附加指定组件。**在构造函数中、以及处理尚未注册的组件时使用**
- **`AttachToComponent`**：将场景组件附加到指定组件。**在游戏进行中使用**

```cpp
//设置根组件
RootComponent = outCollison;  
SetRootComponent(outCollison);

//在构造函数中、以及处理尚未注册的组件时设置
paddle1->SetupAttachment(body, TEXT("paddle1"));

//游戏进行中使用
paddle3->AttachToComponent(body, FAttachmentTransformRules::KeepRelativeTransform, TEXT("paddle3"));

```

### 各类组件创建与初始化

#### UStaticMeshComponent

```cpp
paddle1 = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("paddle1"));
auto paddleMesh = ConstructorHelpers::FObjectFinder<UStaticMesh>(TEXT("StaticMesh'/Game/Demo_Drone/SM/paddle.paddle'"));
if (paddleMesh.Object != nullptr)
{
	paddle1->SetStaticMesh(paddleMesh.Object);
}
```

#### UBoxComponent

```cpp
outCollison = CreateDefaultSubobject<UBoxComponent>(TEXT("outCollison"));
outCollison->SetBoxExtent(FVector(60, 60, 15));
outCollison->SetSimulatePhysics(true);
outCollison->SetCollisionProfileName(TEXT("WorldDynamic"));
outCollison->SetCollisionEnabled(ECollisionEnabled::NoCollision);
```

#### USphereComponent

```cpp
sphereComp = CreateDefaultSubobject<USphereComponent>(TEXT("SphereComp"));
sphereComp->InitSphereRadius(5.0f);
sphereComp->SetCollisionProfileName(TEXT("WorldDynamic"));
sphereComp->SetGenerateOverlapEvents(true);
sphereComp->OnComponentBeginOverlap.AddDynamic(this, &Amissile::Overlaphandler);
```

```cpp
// overlap 函数绑定
void Amissile::Overlaphandler(UPrimitiveComponent* OverlappedComponent,
				AActor* OtherActor, 
				UPrimitiveComponent* OtherComp,
				int32 OtherBodyIndex,
				bool bFromSweep,
				const FHitResult& SweepResult){}
```

#### USkeletalMeshComponent

```cpp
SkeletalMeshComp = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("SkeletalMeshComp"));
SkeletalMeshComp->SetGenerateOverlapEvents(true);
SkeletalMeshComp->SetSimulatePhysics(true);
```

#### USpringArmComponent 和 UCameraComponent

```cpp
springArmComp = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
springArmComp->SetupAttachment(RootComponent);
cameraComp = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
cameraComp->SetupAttachment(springArmComp);
```

#### UPhysicsThrusterComponent

```cpp
upThrusterComp = CreateDefaultSubobject<UPhysicsThrusterComponent>(TEXT("upThrusterComp"));
upThrusterComp->SetupAttachment(RootComponent);
upThrusterComp->SetWorldRotation(UKismetMathLibrary::MakeRotFromX(FVector(-this->GetActorUpVector())));
upThrusterComp->ThrustStrength = 980.0f;
upThrusterComp->SetAutoActivate(true);
```

#### UTextRenderComponent

```cpp
CountdownText = CreateDefaultSubobject<UTextRenderComponent>(TEXT("CountdownNumber"));
CountdownText->SetHorizontalAlignment(EHTA_Center);
CountdownText->SetWorldSize(150.0f);
CountdownText->AttachTo(RootComponent);
CountdownTime = 3;
CountdownText->SetText(FString::FromInt(FMath::Max(CountdownTime, 0)));
```

```cpp
projectileMovementComp = CreateDefaultSubobject<UProjectileMovementComponent>(TEXT("projectileMovement"));
projectileMovementComp->InitialSpeed = 500.0f;
```

### 类型转换

在此例中，我们获取了一个已知的组件，将其转换为特定类型，然后判断能否执行一些操作。
```c++
UPrimitiveComponent* Primitive = 
MyActor->GetComponentByClass(UPrimitiveComponent::StaticClass());

USphereComponent* SphereCollider = Cast<USphereComponent>(Primitive);
if (SphereCollider != nullptr)
{
        // ...
}
````

## Pawn 类
### 获取 Pawn

```c++
//GetPlayerPawn()
APawn* myPawn = Cast<ADrone>(UGameplayStatics::GetPlayerPawn(GetWorld(), 0));

//GetPawn()
APawn* myPawn = GetWorld()->GetFirstPlayerController()->GetPawn();
```

### PlayerController 控制玩家
Pawn 默认的 AutoPossessPlayer 是未设置的，设为 Player0 即代表着将控制权交给 World 中第一个 Controller。如果是多人游戏就会有多个Player
![[Pasted image 20230904133218.png]]

```c++
AutoPossessPlayer = EAutoReceiveInput::Player0;
```
![[Pasted image 20230829162058.png]]

### PlayerController 控制旋转
飞行物 Pawn可以开启这两项，
![[Pasted image 20230904162541.png]]

对于人形 Pawn 通常都关闭，只在 SpringArm 和 Camera 中开启

![[Pasted image 20230904163941.png]]

```c++ h:4,5,6,23,29,34
AMyCharacter::AMyCharacter()
{
	PrimaryActorTick.bCanEverTick = true;
	AutoPossessPlayer = EAutoReceiveInput::Player0;
	bUseControllerRotationPitch = true;
	bUseControllerRotationYaw = true;
}


void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);
	PlayerInputComponent->BindAxis(TEXT("MoveForward"), this, &AMyCharacter::MoveForward);
	PlayerInputComponent->BindAxis(TEXT("Turn"), this, &AMyCharacter::Turn);
	PlayerInputComponent->BindAxis(TEXT("LookUp"), this, &AMyCharacter::LookUp);
}

void AMyCharacter::MoveForward(float Value)
{
	if((Controller!=nullptr) && (Value!=0.0f))
	{
		FVector Forward = GetActorForwardVector();
		AddMovementInput(Forward, Value); //移动组件
	}
}

void AMyCharacter::Turn(float Value)
{
	AddControllerYawInput(Value);
}

void AMyCharacter::LookUp(float Value)
{
	AddControllerPitchInput(Value);
}
```


## Character
### 获取 Character
```c++
//GetPlayerCharacter
ACharacter* myPawn = UGameplayStatics::GetPlayerCharacter(GetWorld(), 0);

//GetCharacter
ACharacter* myPawn = GetWorld()->GetFirstPlayerController()->GetCharacter();
```
## Controller
### 获取 Actor 的 controller
```c++
GetController(); //获取Controller
Cast<AAIController>(GetController()); //获取AIController
```

### 获取 playerController
可配合 Cast 转换成对应的 controller

`UGameplayStatics::GetPlayerController`
```c++
// 查找处理本地玩家控制的actor。
APlayerController* OurPlayerController = UGameplayStatics::GetPlayerController(this, 0);
```

`UWorld::GetFirstPlayerController`
```c++
APlayerController* playerController = GetWorld()->GetFirstPlayerController();
```
### 查找 Player 位置
```c++
FVector MyCharacter = GetWorld()->GetFirstPlayerController()->GetPawn()->GetActorLocation();
```


# 7 物理

## 刚体与图元组件
`RigidBody`/`Primitive Component`

在 Unity 中，假如要为 GameObject 赋予物理特征，首先必须为其提供**刚体**组件。

在UE中，**任何图元组件（C++ 中的 `UPrimitiveComponent`）都可以是物理对象。**
一些常见图元组件如下：
- 形状组件（胶囊体、球体和盒体）
- 静态网格体组件
- 骨骼网格体组件

Unity 将碰撞和可视性划分到不同的组件中，虚幻引擎则将 **"潜在的物理碰撞"（potentially physical）** 和 **"潜在的可视效果"（potentially visible）** 组合到了单个图元组件中。凡是在世界中具有形状的组件，只要能通过物理方式渲染或交互，都是 `PrimitiveComponent` 的子类。
## 碰撞

![[Pasted image 20230115201320.png|300]]
关于碰撞的处理方式，需要记住几点规则：
![[Pasted image 20230115200719.png]]

- 两个对象设置为**互相阻挡 (block)** 才可以产生碰撞, 但是如果想要发生**Hit 事件**, 则需要勾选 `Simulation Generates Hit Events
-   将 Actor 设置为 **overlap** 往往看起来它们彼此 **iganore**，如果没有 **生成重叠事件 `Generate Overlap Events`**，则二者基本相同。
-   对于两个或更多模拟对象：如果一个设置为重叠对象，另一个设置为阻挡对象，则发生重叠，而不会发生阻挡。
-   block 同时也可以是 overlap，但是**不建议同时勾选 Simulation Generates Hit Events 和 Generate Overlap Events**。逻辑上来讲，不会有物体既能处于 Hit 状态又可以重叠，需要手动处理的部分太多。
-   如果一个对象设置为忽略，另一个设置为重叠，则不会触发重叠事件。

**追踪响应 Trace Responses**用于追踪（光线投射），例如蓝图节点 **按频道进行线迹追踪（Line Trace by Channel）**。
**可视性（Visibility）**：泛型可视性测试频道。
**摄像机（Camera）**：通常用于从摄像机到某个对象的追踪。

碰撞设置：
```c++
WeaponCollision->SetCollisionEnabled(ECollisionEnabled::QueryOnly);

WeaponCollision->SetCollisionResponseToAllChannels
(ECollisionResponse::ECR_Overlap);

WeaponCollision->SetCollisionResponseToChannel
(ECollisionChannel::ECC_Pawn, ECollisionResponse::ECR_Ignore);
```

![[Pasted image 20230908140129.png|450]]
## 刚体/碰撞组件

**碰撞组件和刚体组件是同一个组件**。其基类是 `UPrimitiveComponent`，它有许多子类（`USphereComponent`、`UCapsuleComponent` 等）可满足你的需要。

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

### Component 碰撞

- 注意` Overlap Begin/End` 的函数参数
- 注意 `OnHit` 的函数参数
- 注意 `Generated Hit Event` 对应函数名 `SetNotifyRigidBodyCollision`
- 绑定函数可以用 `AddDynamic`，也可以用 `FScriptDelegate` 委托
- 部分设置可不写，蓝图使用时再手动设置
- 写法支持 UShapeComponent及其派生类，如 USphereComponent 、UBoxComponent 等

```cpp
UPROPERTY(EditAnywhere)
USceneComponent* Root;

UPROPERTY(EditAnywhere)
UStaticMeshComponent* Cube;

UFUNCTION()
virtual void OnOverlapBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

UFUNCTION()
virtual void OnOverlapEnd(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

UFUNCTION()
virtual void OnHit(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit);
```

```cpp
ACollisionActor::ACollisionActor()
{
    // Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
    PrimaryActorTick.bCanEverTick = true;

    Root = CreateDefaultSubobject<USceneComponent>(TEXT("RootScene"));
    SetRootComponent(Root);

    Cube = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Cube"));
    Cube->SetupAttachment(Root);
    static ConstructorHelpers::FObjectFinder<UStaticMesh> mesh(TEXT("StaticMesh'/Engine/BasicShapes/Cube.Cube'"));
    if (mesh.Succeeded())
    {
        Cube->SetStaticMesh(mesh.Object);
    }

    // 设置是否开启物理模拟
    Cube->SetSimulatePhysics(false);

    // 开启 Generated Hit Event
    Cube->SetNotifyRigidBodyCollision(true);
    
    // 开启CCD Continuous collision detection (CCD) 连续式碰撞检测
    Cube->BodyInstance.SetUseCCD(true);
    
    // 开启Generate Overlap Events
    Cube->SetGenerateOverlapEvents(true);

    // 设置碰撞预设
    Cube->SetCollisionProfileName(TEXT("OverlapAll"));
    //Cube->SetCollisionResponseToAllChannels(ECR_Overlap);

    // 设置碰撞响应设置
    Cube->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);

    // 绑定函数
    Cube->OnComponentBeginOverlap.AddDynamic(this, &ACollisionActor::OnOverlapBegin);

    // 另一个绑定函数的方法： 使用FScriptDelegate委托
    FScriptDelegate OverlapEndDelegate;
    OverlapEndDelegate.BindUFunction(this, TEXT("OnOverlapEnd"));
    Cube->OnComponentBeginOverlap.Add(OverlapEndDelegate);
    
    // 绑定碰撞函数
    Cube->OnComponentHit.AddDynamic(this, &ACollisionActor::OnHit);
}

void ACollisionActor::OnOverlapBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
    UE_LOG(LogTemp, Warning, TEXT("Overlap Begin"));
}

void ACollisionActor::OnOverlapEnd(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
    UE_LOG(LogTemp, Warning, TEXT("Overlap End"));
}

void ACollisionActor::OnHit(UPrimitiveComponent* HitComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, FVector NormalImpulse, const FHitResult& Hit)
{
    UE_LOG(LogTemp, Warning, TEXT("Hit"));
    Destroy();
}
```

### ATriggerBox 碰撞

- OnActorBeginOverlap
- OnActorEndOverlap
  
  ```cpp
  UFUNCTION()
  	void HandleOverlap(AActor* OverlappedActor, AActor* OtherActor );
  ```
  
  ```cpp
  void AMyTriggerBox::BeginPlay()
  {
  	//放在构造函数好像不起作用
  	OnActorBeginOverlap.AddDynamic(this, &AMyTriggerBox::HandleOverlap);
  }
  
  void AMyTriggerBox::HandleOverlap(AActor* OverlappedActor, AActor* OtherActor )
  {
  	UClass* ActorClass = OtherActor->GetClass();
  	// 其他处理逻辑
  }
  ```
- 
## 触发器体积 Trigger Volumes
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


## 射线检测
### World.h 
- Trace模式
    - TraceSingle 单个结果
    - TraceMulti 多个结果
- Trace 的检测依据
    - ByChanne
    - ByObjectType
    - ByProfile

由于**一般不使用 World 里的Trace API**，故本小结只举 LineTraceSingleByChannel 一个例子
```cpp
// 碰撞参数
FCollisionQueryParams CollisonQueryParams(TEXT("QueryParams"),true,NULL);
CollisonQueryParams.bTraceComplex = true;
CollisonQueryParams.bReturnPhysicalMaterial = false;
CollisonQueryParams.AddIgnoredActor(this);

// 起始点和检测结果
FVector BeginLoc = GetActorLocation();
FVector EndLoc = BeginLoc + GetActorForwardVector() * 1000;
FHitResult HitResult;

// 射线检测
GetWorld()->LineTraceSingleByChannel(HitResult, BeginLoc, EndLoc, ECollisionChannel::ECC_Visibility, CollisonQueryParams);
// 绘制射线
DrawDebugLine(GetWorld(), BeginLoc, HitResult.GetActor() ? HitResult.Location : EndLoc, FColor::Red, false, 1.0f);
if (HitResult.GetActor())
{
    UKismetSystemLibrary::PrintString(GetWorld(), HitResult.GetActor()->GetName());
}
```

### UKismetSystemLibrary
- 与蓝图使用的 Trace 节点对应，
- 实际调用的是 World.h 里的 Trace 函数
![[Pasted image 20230903111939.png]]

```c++
const FVector Start = BoxTractStart->GetComponentLocation();
const FVector End = BoxTractEnd->GetComponentLocation();

TArray<AActor*> ActorsToIgnore;
ActorsToIgnore.Add(this);
FHitResult BoxHit;

UKismetSystemLibrary::BoxTraceSingle(
        this,
        Start,
        End,
        FVector(5.f,5.f,5.f),
        BoxTractStart->GetComponentRotation(),
        TraceTypeQuery1,
        false,
        ActorsToIgnore,
        EDrawDebugTrace::ForDuration,
        BoxHit,
        true,
        FLinearColor::Red,
        FLinearColor::Green,
        5.f
        );
	
```
# 8 输入系统
## 增强输入
Enhanced Input System 实际上就是对默认输入系统做了一个扩展，它以模块化的方式解耦了从输入的按键配置到事件处理的逻辑处理过程，提供了更灵活和更便利的输入配置和处理功能。同时又能向后兼容虚幻引擎 4 (UE4）中的默认输入系统。以插件的形式供我们使用。

**重要的类：**
-  `UInputAction` 输入操作
    - 是交互角色可能做出的任何动作，独立于原始输入，可以设置多种类型的输入值
-  `UInputMappingContext` 输入映射上下文
    - 将用户的输入映射到操作，并可以动态地为每个用户添加、移除或安排优先次序。通过**本地玩家子系统`UEnhancedInputLocalPlayerSubsystem`将一个或多个映射应用到本地玩家，并安排它们的优先次序，避免多个操作由于尝试使用同一输入而发生冲突
-  `UInputModifier` 输入修饰符。
    - 调整来自用户设备的原始输入的值，如使用 Negate 可以将输入值取负值。
-  `UnputTrigeer` 输入触发器。
    - 根据特定的条件来确定是否应该激活输入操作。

**增强输入系统优点**
1. 简化了绑定操作，现在绑定的都是 `InputAction`, 现在不分 Action 和 Axis，提供了更多的扩展性和灵活性。
2. 运行时控制添加、移除映射.
3. 模块化，不再只依赖 ini 配置，以资源 asset 方式配置.
4. 提高性能，不需要检查所有输入，只关心当前场景和绑定

### 蓝图
可以参考引擎的模板 Character

1. 创建 `InputAction` ![[Pasted image 20230829145313.png]]

2. 创建 `InputMappingContext` ![[Pasted image 20230829145347.png]]，设置映射 ![[Pasted image 20230829145439.png|350]]

3. `PlayerController` 添加创建的输入映射上下文，可以添加多个上下文
![[Pasted image 20230829144811.png]]

4. 使用 `InputAction` 对应的事件
![[Pasted image 20230829145511.png|350]]

## 增强输入用法：控制角色跳跃与移动

### 增强输入 - Input Action

定义一个 InputAcction，等同于一个自定义事件，声明输出值类型、触发时机，修改器可用于调整输出值

内容管理器右键 - Input-Input Action (IC)

![[8c9325bd8f38419a9c9ad0bfaa06314b_MD5.jpg]]

新建 IA_jump1 和 IA_move1：

IA_jump1 定义 bool 输出值:

控制跳跃只需要一个 bool 值或浮点数（一个轴的值）即可

![[1d0d80bae09d0a14bca2abd0514c3fa0_MD5.jpg]]

IA_move1 定义 2D 向量（两个轴的值）输出值，通过获取 x 轴与 y 轴序列值来分别控制左右移动与前后移动

![[a2f101de2ed17186abef0129eb515ec7_MD5.jpg]]

也可以通过定义两个浮点数的 input action 来分别控制左右移动与前后移动

### 增强输入 - Input Mapping Context (IMC)

IMC 定义 IC 与按键的绑定关系，实现按键触发动作事件

在 IMC 中也可以添加约束：触发器、修改器，和 IC 中的触发器、修改器会**叠加**生效，下面会详细说明

新建 IMC_Test

内容管理器右键 - Input-Input Mapping Context

![[ec0ded96c4a298d4e7575a08e3055cc4_MD5.jpg]]

IMC_Test：

空格控制跳跃，AD 控制左右移动，WS 控制前后移动

![[f4944342091e45dbbd8bd5123d4fef11_MD5.jpg]]

### 角色蓝图


角色蓝图添加动作映射：

通过 Enhanced Input Local Player Subsystem，将玩家控制器与 IMC 作代理绑定

绑定后 IMC 能够获取到玩家控制器的输入

IMC 再根据按键声明匹配，去触发 IC

蓝图可以添加多个 IMC，优先级高的起作用

Priority：优先级，数字约大，优先级越高

![[ea3d996019b3834469e921cb12e683eb_MD5.jpg]]

蓝图添加按键监听：

事件名即 IC 文件名，在 IMC 中一一对应

![[835ae481edf67456e118b1e8bcd6c499_MD5.jpg]]

![[b71f0a5d02c003c1789e225bca11ad11_MD5.jpg]]

IC_move1 的输出值是一个二维向量，可以拆分成 x 轴与 y 轴

按键 D 声明获取默认值，事件回调输出值为 x=1, y=0 的二维向量

按键 A 声明获取相反值，事件回调输出值为 x=-1, y=0 的二维向量

按键 W 声明交换输入轴的值，默认的 XYZ，将交换后输出 YXZ，事件回调输出值为 x=0, y=1 的二维向量

按键 S 声明交换输入轴的相反值，默认的 XYZ，将交换后输出 YXZ，事件回调输出值为 x=0, y=-1 的二维向量

对于 EnhancedInputAction IA_move1 事件而言，它负责监听按键，当按键输入与 IMC 表上声明的按键匹配，事件触发，在回调中返回输出值，输出 IC 中定义好的值

执行回调再值用于控制角色移动

### 效果呈现

新建关卡地图，将角色蓝图拖到地图中，点击运行

  
添加图片注释，不超过 140 字（可选）

### 三者关系

角色蓝图添加 IMC，IMC 添加 IC

角色蓝图添加 IC, 作按键监听, 蓝图双击事件，可以调到 IC

这样子 IMC 只作用于当前的角色蓝图，而非全局，缩小影响范围，也方便定义多套按键声明

![[79c79a13bcc385cbe5737e8153d540ce_MD5.jpg]]

### 补充：镜头旋转

IA_look1:

![[50e3f61d27bf57adacf188275a53eb62_MD5.jpg]]

通过 2D 轴值输出 XY 轴向数值

IMC_Test1:

Y 轴值取反，使镜头上下旋转的方向与鼠标方向一直

![[6dfde71fcae1abc770991fd8f1722582_MD5.jpg]]

角色蓝图，X 轴控制左右旋转，Y 轴控制上下旋转

![[d865780b81107439167626cc16dbaa11_MD5.jpg]]

设置机械臂：

允许鼠标进行旋转控制，否则上下旋转不生效

![[e11e83e4e5436a892b1d82ad57f2fb36_MD5.jpg]]

优化角色移动：

上下旋转镜头后，角色移动会变慢，因为角色正前方的向量被分解了

所以只获取正前方的向量值

![[bb20d7c7b1e3d6be4fd1d9f2662b0264_MD5.jpg]]

旋转镜头角色不跟着旋转：

![[ea15f9b3d47278684df4d423280fdef5_MD5.png]]

use controller rotation yaw 要取消勾选

![[cbd1edd0b46cd2e696ba290f0d2635b8_MD5.png]]

## 解析

新建 IA_test 用于测试

IMC:

![[45070408f63daefe72f01a86b4055b56_MD5.jpg]]

角色蓝图：

查看在事件开始、结束与触发过程中的输出值

### IC 输入值类型

控制事件输出值类型
*   整数 / 布尔值：事件开始与触发输出 true，结束输出 false
*   浮点数 / X 轴值：事件开始与触发输出 1.0，结束输出 0.0

![[dd9bb7d18e2762801121789774d9884f_MD5.jpg]]

*   二维向量 / XY 轴值：事件开始与触发输出 x=1.000 y=0.000，结束输出 x=0.000 y=0.000

![[82d4d038afc005722acf908a6363a60f_MD5.jpg]]

*   三维向量 / XYZ 轴值：事件开始与触发输出 x=1.000 y=0.000 z=0.000，结束输出 x=0.000 y=0.000 z=0.000

### IC 修改器 - IM

*   `Negate`：值取反，事件开始与触发输出 x=-1.000 y=-0.000 z=-0.000，结束输出 x=-0.000 y=-0.000 z=-0.000
>对整数 / 布尔的输入值类型不起作用，还是在事件开始与触发输出 true，结束输出 false

*   `SwizzleInputAxisValues`：互换轴值，当输入值为两个轴以上的时候，可以将输出值顺序从 XYZ 调整为其他顺序 (比如调整为 YXZ)，那么 W/S 的输出值在 Y 轴。
>输出值若为一个轴或 bool 值，将无法获得输出值

以三维向量为例：

此时 Z 轴有值
![[9ed3530e4c6e313ab42f419ba953e3de_MD5.jpg]]

*   `Scalar`：缩放一个标量
将输出值缩放 2 倍
>注意：此处对修改器的顺序有要求，对调轴值后，输出值在 Y 轴，此时对 Y 轴进行缩放才有效，否则无效
![[341f7a0a2b7e8fb20092c3522e0495c8_MD5.jpg]]

![[09afd22683bad8ed24daa69e52d1f257_MD5.jpg]]


其他的修改器有（多与摇杆有关，具体可向 GPT 提问）：

*   `DeadZone`： 限定触发值的范围
*   `FOVScaling`： FOV 缩放
*   `CurveExponential`：指数曲线，XYZ
*   `CurveUser`：自定义指数曲线，CurveFloat
*   `Smooth`：多帧之间平滑
*   `ToWorldSpace`：输入设备坐标系向世界坐标系转换

当将修改器设置为 ToWorldSpace 时，输入设备的输入值将被转换为世界空间（World Space）中的向量，而不是相对于游戏角色的本地空间（Local Space）中的向量。这意味着，无论游戏角色朝向何方，输入设备的输入值都将产生相同的效果，从而减少了玩家在控制角色时需要考虑本地坐标系的复杂性。

举例来说，如果你将一个摇杆的输入值（X，Y）转换为世界空间中的向量，那么这个向量的方向将与摇杆的偏移方向相同，但是大小将根据摇杆的偏移量来确定。这个向量的大小可以通过修改器的 Scale 属性进行缩放，从而控制游戏角色的移动速度。

需要注意的是，将修改器设置为 ToWorldSpace 可能会对游戏的响应速度产生一定的影响，因为它需要对输入值进行矩阵变换以将其从本地空间转换到世界空间。同时，ToWorldSpace 的效果也可能会因不同的游戏和输入设备而有所不同。

### IMC 与 IC 可同时设置 IM、IT

修改器和触发器可以在 Maping 和 InputAciton 中同时设置：

*   Mapping. Modifiers / Triggers ：针对当前 IMC 场景，和按键强相关的
*   InputAction. Modifiers / Triggers： 针对全局，不需要关心按键，主要关心值怎么动，处理逻辑相关的，
*   **执行顺序**：先 IMC 后 IC

![[bb354f4d73f12e07b8063433f34098d1_MD5.jpg]]

先交换轴值，再缩放 X 轴值 2 倍，输出值为 0,1,0

![[aecb2dafe451c04c676d6b3f34682d3e_MD5.jpg]]

IC 中将 Y 轴缩放值改为 2，输出 0,2,0

![[76c99246a1988065e21f79354cdd24f3_MD5.jpg]]

IC 的 IM 会覆盖 IMC 的 IM：IMC 和 IC 定义相同的 IM 时，IC 的 IM 最终生效

![[c569fea167b6e2290c3aaf49b40bdd03_MD5.jpg]]

![[47c460074da77aa02f9874372ce01f40_MD5.jpg]]

此时输出 0,0,1

![[71d532e713b497a2bec30d2330882882_MD5.jpg]]

### IC 触发器 - IT

*   `None`：此时使用的是 `Down` 类型
    1.  按下按键触发 Started
    2.  保持按下会不断触发 Triggered 事件
    3.  松开按键触发 Completed

*   `Pressed`：
    1. 按下按键依次触发 Started、Triggered 与 Completed
    2. 松开按键不触发任何事件。

*   `Down`：可用于**机枪的连续射击**，Started 开始射击，Completed 停止射击，Triggered 检查弹夹。
    1.  按下按键触发 Started 事件
    2.  保持按下会不断触发 Triggered 事件
    3.  松开按键触发 Completed 事件

*   `Hold`：
    1.  **`IsOneShot` 没有勾选**时：可用于**按下保持一段时间，松开才会触发**的动作。如**吃鸡里手雷投掷**。-
        - 按下按键触发 Started
        - 保持按下不断触发 Ongoing，
        - 松开时如果保持的时间达到或超过 HoldTimeThreshold 设置的时间则触发 Completed，否则触发 Canceled。
    2.  **`IsOneShot` 勾选**：用于**按下保持一段时间后自动触发**的动作，如 CS 里**拆包**。
    - 按下按键触发 Started
    - 保持按下不断触发 Ongoing，在达到 HoldTimeThreshold 设置的时间前松开触发 Canceled。
    - 保持按下达到 HoldTimeThreshold 设置的时间后会依次触发 Triggered 与 Completed。

*   `HoldAndRelease`：与 Hold 类型不勾选 IsOneShot 类似。
    1.  按下按键触发 Started
    2.  保持按下不断触发 Ongoing
    3.  松开按键时如果保持的时间达到 HoldTimeThreshold 设置的时间会依次触发 Triggered 与 Completed，否则触发 Canceled。

*   `Pulse`：用于**保持按下时每隔一段时间触发一次**的动作。如按下吃药键，每隔一段时间执行一次吃药的动作
    1.  按下时触发 Started 事件
    2.  如果勾选 TriggerOnStart 则立即触发一次 Triggered 事件，不勾选则间隔一段时间后触发 Triggered。
    3.  保持按下触发 Ongoing 事件。
    4.  距上一次 Trigger 的时间间隔达到 Interval 设置的时间触发一次 Triggered 事件，随后判断 Trigger 触发的数量是否达到 TriggerLimit 的限制 (TriggerLimit<=0 表示不限制次数)，未达到则继续上一步，否则触发 Completed。

*   `Release`：
    1.  按下时触发 Started 事件
    2.  保持按下触发 Ongoing 事件
    3.  松开时依次触发 Triggered 与 Completed

*   `Tap` **按下后快速抬起**（默认 0.2）：
    1.  按下按键触发 Started 事件
    2.  保持按下触发 Ongoing 事件
    3.  如果保持的时间超过 TapReleaseTimeThreshold 设置的时间触发 Canceled 事件。
    4.  松开按键时保持的时间小于 TapReleaseTimeThreshold 设置的时间依次触发 Triggered 与 Completed 事件。

*   `Chorded Action`：用于多个 Action 的串联形成**组合键**。
    1.  按下 Q+A/B/C 中一个或两个键，触发 Started 事件，后触发 Ongoing 事件
    2.  此时松开 Q，或松开其他键只按住 Q，触发 Cancel 事件
    3.  当 Q+A+B+C 触发 Triggered 事件，按住持续触发，松开其中一个键松开按键触发 Completed 事件 ![[9e4d90c7941f3920a964d5ed26f0ae2b_MD5.jpg]]


*   `Combo`：用于类似上上下下左右左右 BABA 在短时间内完成的组合键。
    1.  按下第一个键触发 Started
    2.  过程当中不断触发 Ongoing
    3.  如果某个键按下的时间没有达到 TimeToPressKey 或者按错键触发 Canceled
    4.  当所有的键完成后依次触发 Triggered 与 Completed

IT 设置 Combo 后，不需要按键触发，是通过 IC 组合来触发
示例：按下 AABBCC 触发：
![[696599e5b2744eed75358f0502fb2cd6_MD5.jpg]]

IA_Action123 设默认值即可

## 调试

命令行输入

```
showdebug enhancedinput
```

## 引用

[牵线木偶：UE5 InputAction 使用手册](https://zhuanlan.zhihu.com/p/615553951)  
[Unreal Engine 增强输入框架 EnhancedInput](https://juejin.cn/post/7195746542069628988#heading-8)
## 默认输入系统

### 轴映射与动作映射
轴映射每帧执行，动作映射一次性执行
![[Pasted image 20230904135119.png]]
![[Pasted image 20230904204143.png]]
![image](https://img2020.cnblogs.com/blog/2369154/202104/2369154-20210422201258708-740217416.png)

```cpp
void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAction("DropItem", EInputEvent::IE_Pressed, this, &AMyCharacter::DropItem);
	PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &AMyCharacter::Jump);
	PlayerInputComponent->BindAxis("MoveForward", this, &AMyCharacter::MoveForward);
	PlayerInputComponent->BindAxis("MoveRight", this, &AMyCharacter::MoveRight);
	PlayerInputComponent->BindAxis("PitchCamera", this, &AMyCharacter::PitchCamera);
	PlayerInputComponent->BindAxis("YawCamera", this, &AMyCharacter::YawCamera);
}

void AMyCharacter::MoveForward(float AxisValue)
{
	MovementInput.X = FMath::Clamp<float>(AxisValue, -1.f, 1.f);
}

void AMyCharacter::MoveRight(float AxisValue)
{
	MovementInput.Y = FMath::Clamp<float>(AxisValue, -1.f, 1.f);
}

void AMyCharacter::PitchCamera(float AxisValue)
{
	CameraInput.Y = AxisValue;
}

void AMyCharacter::YawCamera(float AxisValue)
{
	CameraInput.X = AxisValue;
}
```

- 设置按键再游戏暂停可以继续响应 `bExecuteWhenPaused`

```kotlin
InputComponent->BindAction("ESCEvent", IE_Pressed, this, &ASLAiPlayerController::ESCEvent).bExecuteWhenPaused=true;//游戏暂停可以执行
```

### 从C++中添加轴和动作映射

```cpp
//添加、绑定ActionKeyMapping轴映射 方法一
FInputActionKeyMapping onFire("OnFire", EKeys::LeftMouseButton, 0, 0, 0, 0);
UPlayerInput::AddEngineDefinedActionMapping(onFire);
PlayerInputComponent->BindAction("OnFire", IE_Pressed, this, &AMyCharacter::OnFire);

//添加、绑定ActionKeyMapping轴映射 方法二
UPlayerInput::AddEngineDefinedActionMapping(FInputActionKeyMapping("Sprint",EKeys::LeftShift));
PlayerInputComponent->BindAction("Sprint", IE_Pressed,this,&AMyCharacter::StartSprint);
PlayerInputComponent->BindAction("Sprint", IE_Released,this,&AMyCharacter::StopSprint);

//添加、绑定AxisMapping轴映射
UPlayerInput::AddEngineDefinedAxisMapping(FInputAxisKeyMapping("Turn", EKeys::MouseX, 1.0f));
PlayerInputComponent->BindAxis("Turn", this, &AMyCharacter::OnTurn);

```

roll  X
pitch Y
yaw Z
# 9 相机
## 获取和切换场景相机
```c++
// 查找处理本地玩家控制的actor。
APlayerController* OurPlayerController = UGameplayStatics::GetPlayerController(this, 0);

OurPlayerController->GetViewTarget(); //获取当前相机

OurPlayerController->SetViewTarget(CameraOne); //切换相机
OurPlayerController->SetViewTargetWithBlend(CameraTwo, SmoothBlendTime); //平滑切换
```
## 弹簧臂
```c++
// 使用弹簧臂给予摄像机平滑自然的运动感。  
USpringArmComponent* SpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("CameraAttachmentArm"));  
SpringArm->SetupAttachment(RootComponent);  
SpringArm->SetRelativeRotation(FRotator(-45.0f, 0.0f, 0.0f));  
SpringArm->TargetArmLength = 400.0f;  
SpringArm->bEnableCameraLag = true;  
SpringArm->CameraLagSpeed = 3.0f;

// 创建摄像机并附加到弹簧臂
UCameraComponent* Camera = CreateDefaultSubobject<UCameraComponent>(TEXT("ActualCamera"));
Camera->SetupAttachment(SpringArm, USpringArmComponent::SocketName);
```

# 10 粒子
Niagara 系统
```c++ title:粒子系统
//声明
UNiagaraSystem* HitParticle;

//Spawn
UNiagaraFunctionLibrary::SpawnSystemAtLocation(this,HitParticle,GetActorLocation(),GetActorRotation());

//发射一次
UGameplayStatics::SpawnEmitterAtLocation(World, HitParticle, ImpactPoint);
```

Niagara 系统组件
```c++ 
//声明
UParticleSystemComponent* OurParticleSystemComponent;

// 创建粒子系统组件
OurParticleSystemComponent = CreateDefaultSubobject<UParticleSystemComponent>(TEXT("MovementParticles"));  
OurParticleSystemComponent->SetupAttachment(SphereVisual);  
OurParticleSystemComponent->bAutoActivate = false;  
OurParticleSystemComponent->SetRelativeLocation(FVector(-20.0f, 0.0f, 20.0f));  

static ConstructorHelpers::FObjectFinder<UParticleSystem> ParticleAsset(TEXT("/Game/StarterContent/Particles/P_Fire.P_Fire"));  
if(ParticleAsset.Succeeded())  
{  
    //修改ParticleSystemComponent使用的ParticleSystem
    OurParticleSystemComponent->SetTemplate(ParticleAsset.Object);  
}


//Niagara:需要引入"Niagara"模块
UNiagaraComponent* NiagaraComponent;
NiagaraComponent->Deactivate(); //关闭

```
# 音效
```c++
USoundBase* PickUpSound;

UGameplayStatics::PlaySoundAtLocation(this, PickUpSound, GetActorLocation());
```
# 动画
## 蒙太奇 Montage
```c++
//声明
UAnimMontage* AttackMontage;

//播放指定SectionName的蒙太奇片段
void ABaseCharacter::PlayMontageSection(UAnimMontage* AnimMontage, const FName& SectionName)
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();

	if (AnimInstance && AnimMontage)
	{
		AnimInstance->Montage_Play(AnimMontage);
		AnimInstance->Montage_JumpToSection(SectionName, AnimMontage);
	}
}


```
# 玩法
## 不同受击方向
```c++
void ABaseCharacter::DirectionHitReact(const FVector& ImpactPoint)
{
	//点积计算Forward和ToHit的夹角，来确定播放hit动画
	const FVector Forward = GetActorForwardVector();
	const FVector ImpactLowered = FVector(ImpactPoint.X, ImpactPoint.Y, GetActorLocation().Z); //固定冲击点z轴，方便计算方向
	const FVector ToHit = (ImpactLowered - GetActorLocation()).GetSafeNormal();
	const double CosTheta = FVector::DotProduct(Forward, ToHit);
	double Theta = FMath::Acos(CosTheta);
	Theta = FMath::RadiansToDegrees(Theta); //弧度转角度

	//叉积判断Hit点在Forward的左边还是右边
	//左边叉积方向为正，右边为负
	const FVector CrossProduct = FVector::CrossProduct(Forward, ToHit);
	if (CrossProduct.Z < 0)
	{
		Theta *= -1; //右边为负角度，即顺时针角度从Forward为0开始：0~90~180~-90~0
	}

	FName SectionName;
	if (Theta >= -45.0f && Theta <= 45.0f)
	{
		SectionName = FName("FrontHit");
	}
	else if (Theta > 45.0f && Theta <= 135.0f)
	{
		SectionName = FName("LeftHit");
	}
	else if (Theta < -45.0f && Theta >= -135.0f)
	{
		SectionName = FName("RightHit");
	}
	else
	{
		SectionName = FName("BackHit");
	}
	PlayHitReactMontage(SectionName);
}
```
## 道具拾取
子组件吸附到父组件的 Socket 上
```c++
void AWeaponBase::AttachComponentToSocket(USceneComponent* InParentComponent, UStaticMeshComponent* InChildComponent,
                                          FName InSocketName)
{
	const FAttachmentTransformRules AttachmentTransformRules(EAttachmentRule::SnapToTarget,
	                                                         EAttachmentRule::SnapToTarget,
	                                                         EAttachmentRule::SnapToTarget, true);
	InChildComponent->AttachToComponent(InParentComponent, AttachmentTransformRules, InSocketName);
}
```
# 保存游戏
[在虚幻引擎中保存和加载游戏 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/saving-and-loading-your-game-in-unreal-engine/)

# 强引用弱引用
数据层和表现层分离
```c++
class IMyID
{
public:
	IMyID()
	{
		ID = FMath::RandRange(0,1000);
	}
private:
	int64 ID;
};

//数据类
class FData : public IMyID
{
public:
	float Health;
	uint8 bDeath;
	FName PlayerName;
};

//数据管理类
class FDataManage
{
public:
	TSharedRef<FDataManage> Get()
	{
		if(!DataManage.IsValid())
		{
			DataManage = MakeShareable(new FDataManage());
		}

		return DataManage.ToSharedRef();
	}

	~FDataManage()
	{
		for(auto &tmp : MyData)
		{
			delete tmp.Value;
		}
		
		MyData.Empty(); //清空数据
	}
private:
	static TSharedPtr<FDataManage> DataManage; 

	TMap<int64,TSharedPtr<FData>> MyData; //通过强指针来管理数据，可以控制数据的生命周期(在析构中清空数据)
};

//角色实例
class FCharacter 
{
	FORCEINLINE bool isValid()
	{
		return NewData.IsValid();
	}
	
private:
	TWeakPtr<FData> NewData; //通过这个弱指针就可以修改数据,避免野指针
};
```

# 抛物线路径、发射轨道相关
-  `UGameplayStatics::Blueprint_PredictProjectilePath_ByObjectType`：根据 Object Type，算出抛物线的点集合和检测结果

```c++ h:12
FVector BeginLoc = GetActorLocation();
FVector LaunchVelocity = GetActorForwardVector() * 1000.0f;
TArray<TEnumAsByte<EObjectTypeQuery> > ObjectTypes;
ObjectTypes.Add(EObjectTypeQuery::ObjectTypeQuery1);
TArray<AActor*> IgnoreActors;	

FHitResult HitResult;
TArray<FVector> OutPatnPositions;
FVector OutLastTraceDestination;

//开始模拟
bool bIsHit = UGameplayStatics::Blueprint_PredictProjectilePath_ByObjectType(
	GetWorld(),
	HitResult,
	OutPatnPositions,
	OutLastTraceDestination,
	BeginLoc,
	LaunchVelocity,
	true,
	0.0f,
	ObjectTypes,
	false,
	IgnoreActors,
	EDrawDebugTrace::ForDuration,
	0.0f
);
if (bIsHit)
{
	UKismetSystemLibrary::PrintString(GetWorld(), HitResult.GetActor()->GetName());
}
```

- `Blueprint_PredictProjectilePath_ByTraceChannel`:根据 ChannelChannel，算出抛物线的点集合和检测结果
- `PredictProjectilePath`：根据预测参数，推算结果
-  `Blueprint_PredictProjectilePath_Advanced`：根据预测参数，推算结果
- `BlueprintSuggestProjectileVelocity`：根据目标点，反算初速度
- `SuggestProjectileVelocity_CustomArc`：根据目标点，反算初速度

