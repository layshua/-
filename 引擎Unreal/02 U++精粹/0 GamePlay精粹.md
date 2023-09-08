# 1 UObject

## 创建 Object

- **`UObject` 构造函数不支持参数**。所有的 C++ `UObject` 都会在引擎启动的时候初始化，然后引擎会调用其默认构造器。如果没有默认的构造器，那么 `UObject` 将不会编译。
-  `UObject` 构造函数应该轻量化，仅用于设置默认的数值和子对象，构造时不应该调用其它功能和函数。
- **`UObject` 永远都不应使用 `new` 运算符。** 所有的 UObjects 都由虚幻引擎管理内存和垃圾回收。如果通过 new 或者 delete 手动管理内存，可能会导致内存出错。

---

- **`NewObject<T>`** ：创建一个 `UObject` 实例，仅在运行时使用
- **`CreateDefaultSubobject<TT>`** ：创建一个组件或者子对象，在构造函数中使用

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

## 更新 Object

Ticking 代表虚幻引擎中对象的更新方式。所有 Actors 均可在每帧被 tick，便于您执行必要的更新计算或操作。

-  `Actor` 和 Actor 组件在注册时会自动调用它们的 Tick 函数
- **`UObjects` 不具有嵌入的更新能力。在必须的时候，可以使用 `inherits` 类说明符从 `FTickableGameObject` 继承即可添加此能力。**  这样即可实现 `Tick()` 函数，引擎每帧都将调用此函数。


## 销毁 Object

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

## 查找
### 按类型查找 Object
```c++
// 按类型查找UObject
for (TObjectIterator<UMyObject> It; It; ++it)
{
    UMyObject* MyObject = *It;
    // ...
}
```
### 从文件查找 Object
```c++
//从文件查找Object
static ConstructorHelpers::FObjectFinder<UStaticMesh> CylinderAsset(TEXT("/Game/StarterContent/Shapes/Shape_Cylinder.Shape_Cylinder")); 
```

# 2 AActor 
对于 Actor 和 Actor 组件，初始化功能应该输入 **`BeginPlay()`** 方法。
## 实例化 Actor

[Spawning Actors in Unreal Engine | 虚幻引擎5.2文档](https://docs.unrealengine.com/5.2/zh-CN/spawning-actors-in-unreal-engine/)

- Actor 通过 `UWorld` 对象（可以通过 `GetWorld()` 获得）的 **`SpawnActor`** 方法生成。
>UObject 为 Actor 提供了 `GetWorld` 方法

```c++
AKAsset* SpawnedActor1 =
(AKAsset*) GetWorld()->SpawnActor(AKAsset::StaticClass(), NAME_None, &Location);
```


## 销毁 Actor

```c++
MyActor->Destroy(); //AActor销毁
```

即使 Actor 被调用了 `Destroy()`，并且被从关卡中移除，它还是会等到所有对它的引用都解除之后才会被垃圾回收。

## 生命周期
延迟 1s 销毁

- 构造函数初始化生命周期 `InitialLifeSpan`
- BeginPlay 里使用 `SetLifeSpan()` 设置生命周期
```c++
InitialLifeSpan = 8.0f;  

MyActor->SetLifeSpan(1); //延迟1s销毁
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

## 查找/获取 Actor
### 单个Actor
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

### Actor 数组
![[Pasted image 20230903002855.png]]
```c++
TArray<AActor*> ActorsToFind;  
if(UWorld* World= GetWorld())  
{
UGameplayStatics::GetAllActorsOfClassWithTag(GetWorld(),AFireEffect::StaticClass(),FName("FireTag"),ActorsToFind);
}
```
## 标签Tags
### 添加标签 Actor/ActorComponent 
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
### 比较标签 Actor/ActorComponent 

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


# 3 Actor 组件
## 注册
### 注册组件
引擎必须注册组件，才能让 Actor 组件能够逐帧更新。如果在 Actor 产生过程中，作为 Actor 子对象自动创建了组件，则这类组件会自动注册。
但是游戏期间创建的组件可以使用手动注册。`RegisterComponent` 函数提供了这个功能，要求是组件与 Actor 关联。

> [!NOTE] Title
> 游戏期间注册组件可能会影响性能，因此只应在必要时进行此操作。
### 注册事件

在注册组件的过程中，引擎会将组件与场景关联起来，让其可用于逐帧更新，并运行以下 `UActorComponent` 函数：

|函数|描述|
|---|---|
|`OnRegister`|在注册组件时，可以覆写此函数来添加代码。|
|`CreateRenderState`|初始化组件的[渲染状态](https://docs.unrealengine.com/4.26/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/Components/#%E5%9C%BA%E6%99%AF%E4%BB%A3%E7%90%86)。|
|`OnCreatePhysicsState`|初始化组件的[物理状态](https://docs.unrealengine.com/4.26/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Actors/Components/#%E7%89%A9%E7%90%86%E7%8A%B6%E6%80%81)。|

### 取消注册

要从更新、模拟或渲染过程中移除 Actor 组件，可以使用 `UnregisterComponent` 函数将其取消注册。

## 更新
组件能够以类似于 Actor 的方法逐帧更新。`TickComponent` 函数允许组件逐帧运行代码。
例如，**USkeletalMeshComponent** 使用其 `TickComponent` 函数来更新动画和骨架控制器，而 **UParticleSystemComponent** 使用 `TickComponent`更新其发射器和处理粒子事件`。

**默认情况下，Actor 组件不更新**。

**为了让 Actor 组件逐帧更新：**
1. 必须在构造函数中将 `PrimaryComponentTick.bCanEverTick` 设置为 `true` 来启用 tick。
2. 之后，在构造函数中或其他位置处，必须调用 `PrimaryComponentTick.SetTickFunctionEnable(true)` 以开启更新。
3. 之后可调用 `PrimaryComponentTick.SetTickFunctionEnable(false)` 停用 tick。
4. 如果您知道组件永远不需要更新，或者打算手动调用自己的更新函数（也许从拥有的 Actor 类），将 `PrimaryComponentTick.bCanEverTick` 保留为默认值 `false` 即可，这样可以稍微改善性能。
### 渲染状态

为进行渲染，Actor 组件必须创建渲染状态。此渲染状态还会告诉引擎，需要更新渲染数据的组件已发生变更。当发生此类变更时，渲染状态会被标记为"dirty"。
如果编译您自己的组件，可以使用 `MarkRenderStateDirty` 函数将渲染数据标记为 dirty。在一帧结束时，所有 dirty 组件的渲染数据都会在引擎中更新。
**场景组件（包括 Primitive 组件）默认会创建渲染状态，而 Actor 组件则不会。**

### 物理状态

要与引擎的物理模拟系统交互，Actor 组件需要物理状态。物理状态会在发生变化时立即更新，防止出现"帧落后"瑕疵等问题，也不需要"dirty"标记。
**默认情况下，`UActorComponent` 和 `USceneComponent` 没有物理状态，但 `UPrimitiveComponent` 有。** 

覆盖 `ShouldCreatePhysicsState` 函数以确定组件类实例是否需要物理状态。
- 如果类使用物理，则不建议只返回 `true`。请参阅函数的 `UPrimitiveComponent` 版本，了解不应创建物理状态的情况（例如在组件破坏期间）。
- 在正常返回 `true` 的情况下，还可以返回 `Super::ShouldCreatePhysicsState`。

## 可视化组件
 
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

## 销毁

```c++
MyActorComponent->DestroyComponent(); //销毁UActorComponent
```

## 获取组件依附的 Actor

```c++
AActor* ParentActor = MyComponent->GetOwner();
```
## 获取 Actor 上的组件

```c++
UMyComponent* MyComp = MyActor->FindComponentByClass<UMyComponent>();
```

![[e5c08b782929e1ae15f583195a515fb2_MD5.jpg]]

## 设置组件层级关系
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

## 各类组件创建与初始化

### UStaticMeshComponent

```cpp
paddle1 = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("paddle1"));
auto paddleMesh = ConstructorHelpers::FObjectFinder<UStaticMesh>(TEXT("StaticMesh'/Game/Demo_Drone/SM/paddle.paddle'"));
if (paddleMesh.Object != nullptr)
{
	paddle1->SetStaticMesh(paddleMesh.Object);
}
```

### UBoxComponent

```cpp
outCollison = CreateDefaultSubobject<UBoxComponent>(TEXT("outCollison"));
outCollison->SetBoxExtent(FVector(60, 60, 15));
outCollison->SetSimulatePhysics(true);
outCollison->SetCollisionProfileName(TEXT("WorldDynamic"));
outCollison->SetCollisionEnabled(ECollisionEnabled::NoCollision);
```

### USphereComponent

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

### USkeletalMeshComponent

```cpp
SkeletalMeshComp = CreateDefaultSubobject<USkeletalMeshComponent>(TEXT("SkeletalMeshComp"));
SkeletalMeshComp->SetGenerateOverlapEvents(true);
SkeletalMeshComp->SetSimulatePhysics(true);
```

### USpringArmComponent 和 UCameraComponent

```cpp
springArmComp = CreateDefaultSubobject<USpringArmComponent>(TEXT("SpringArm"));
springArmComp->SetupAttachment(RootComponent);
cameraComp = CreateDefaultSubobject<UCameraComponent>(TEXT("Camera"));
cameraComp->SetupAttachment(springArmComp);
```

### UPhysicsThrusterComponent

```cpp
upThrusterComp = CreateDefaultSubobject<UPhysicsThrusterComponent>(TEXT("upThrusterComp"));
upThrusterComp->SetupAttachment(RootComponent);
upThrusterComp->SetWorldRotation(UKismetMathLibrary::MakeRotFromX(FVector(-this->GetActorUpVector())));
upThrusterComp->ThrustStrength = 980.0f;
upThrusterComp->SetAutoActivate(true);
```

### UTextRenderComponent

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
````

# 4 Pawn 类
## 获取 Pawn

```c++
//GetPlayerPawn()
APawn* myPawn = Cast<ADrone>(UGameplayStatics::GetPlayerPawn(GetWorld(), 0));

//GetPawn()
APawn* myPawn = GetWorld()->GetFirstPlayerController()->GetPawn();
```

## PlayerController 控制玩家
Pawn 默认的 AutoPossessPlayer 是未设置的，设为 Player0 即代表着将控制权交给 World 中第一个 Controller。如果是多人游戏就会有多个Player
![[Pasted image 20230904133218.png]]

```c++
AutoPossessPlayer = EAutoReceiveInput::Player0;
```
![[Pasted image 20230829162058.png]]

## PlayerController 控制旋转
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


# 5 Character
## 获取 Character
```c++
//GetPlayerCharacter
ACharacter* myPawn = UGameplayStatics::GetPlayerCharacter(GetWorld(), 0);

//GetCharacter
ACharacter* myPawn = GetWorld()->GetFirstPlayerController()->GetCharacter();
```
# 6 PlayerController
## 获取 playerController
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
## 查找 Player 位置
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
    - 是交互角色可能做出的任何动作，独立于原始输入，可以设置多种类型的输入值-
-  `UInputMappingContext` 输入映射上下文
    - 将用户的输入映射到操作，并可以动态地为每个用户添加、移除或安排优先次序。通过本地玩家子系统（Enhanced Input Local Player Subsystem)将一个或多个映射应用到本地玩家，并安排它们的优先次序，避免多个操作由于尝试使用同一输入而发生冲突
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
```c++
//声明
UPROPERTY()  
UParticleSystemComponent* OurParticleSystemComponent;

// 创建粒子系统  
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

# 理解 DeltaTime
UE 世界坐标单位是 cm
涉及移动旋转计算时，我们通常声明一个常量乘以 `DeltaTime`，原理如下：

```cs
void AMyClass::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);
	float MovementRate = 50.0f; //移动速度为50cm/s
	float RotationRate = 45.0f; //旋转速度为90°/s
	//Speed * DeltaTime (cm/s)*(s/frame) = (cm/frame) 即每帧移动多少cm
	//通过这种方法可以保证在不同帧率下移动速度一致
	AddActorWorldOffset(FVector(0.f, 0.f, MovementRate * DeltaTime));
	AddActorWorldRotation(FRotator(0.f, RotationRate*DeltaTime, 0.f));
}
```