# Object

## 创建 Object

**`UObjects` 不支持构造器参数**。所有的 C++ `UObject` 都会在引擎启动的时候初始化，然后引擎会调用其默认构造器。如果没有默认的构造器，那么 `UObject` 将不会编译。

**`UObject` 构造器应该轻量化，仅用于设置默认的数值和子对象，构造时不应该调用其它功能和函数。** 

对于 Actor 和 Actor 组件，初始化功能应该输入 **`BeginPlay()`** 方法。

`UObject` 应该仅在运行时使用 `NewObject` 构建，或者将 `CreateDefaultSubobject` 用于构造器。

|方法|描述|
|---|---|
|[`NewObject<class>`]( https://docs.unrealengine.com/en-US/API/Runtime/CoreUObject/UObject/NewObject "NewObject")|使用所有可用创建选项的可选参数创建一个新实例。提供极高的灵活性，包括带自动生成命名的简单使用案例。|
| `CreateDefaultSubobject<class>` |创建一个组件或者子对象，可以提供创建子类和返回父类的方法。<br><br> 创建默认子对象时，由于它们在引擎启动时构造，UObject 的类构造器应该仅适用于本地数据或者本地加载的静态资产。|

> [!warning]
> **`UObjects` 永远都不应使用 `new` 运算符。所有的 UObjects 都由虚幻引擎管理内存和垃圾回收。如果通过 new 或者 delete 手动管理内存，可能会导致内存出错。**

**`CreateDefaultSubobject`** ：创建组件。**使用此函数创建的所有子对象都充当默认模板，因此我们可以在子类或蓝图中修改它们。**

```c++
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

    USphereComponent* MyCollisionComp;

    AMyActor()
    {
        MyCollisionComp = CreateDefaultSubobject<USphereComponent>(FName(TEXT("CollisionComponent"));
        MyCollisionComp->RelativeLocation = FVector::ZeroVector;
        MyCollisionComp->SphereRadius = 20.0f;
    }
};
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

# Actor 类
## 实例化 Actor

[Spawning Actors in Unreal Engine | 虚幻引擎5.2文档](https://docs.unrealengine.com/5.2/zh-CN/spawning-actors-in-unreal-engine/)

- Actor 通过 `UWorld` 对象（可以通过 `GetWorld()` 获得）的 **`SpawnActor`** 方法生成。
>UObject 为 Actor 提供了 `GetWorld` 方法

```c++
AKAsset* SpawnedActor1 =
(AKAsset*) GetWorld()->SpawnActor(AKAsset::StaticClass(), NAME_None, &Location);
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
```

## 销毁 Actor

```c++
MyActor->Destroy(); //AActor销毁
```

即使 Actor 被调用了 `Destroy()`，并且被从关卡中移除，它还是会等到所有对它的引用都解除之后才会被垃圾回收。

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

## 查找 Actor

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
```

![[52ee0d707b4ee7d5c92ad8ebc0c1cbc8_MD5.jpg]]

```c++
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

### 获取标签 Actor 数组
```c++
TArray<AActor*> ActorsToFind;  
if(UWorld* World= GetWorld())  
{UGameplayStatics::GetAllActorsOfClassWithTag(GetWorld(),AFireEffect::StaticClass(),FName("FireTag"),ActorsToFind);
}
```
## 创建静态网格体

```c++
// 创建并放置网格体组件，以便查看球体位置  
UStaticMeshComponent* SphereVisual = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("VisualRepresentation"));  

SphereVisual->SetupAttachment(RootComponent);  

static ConstructorHelpers::FObjectFinder<UStaticMesh> SphereVisualAsset(TEXT("/Game/StarterContent/Shapes/Shape_Sphere.Shape_Sphere"));  

if(SphereVisualAsset.Succeeded())  
{  
    SphereVisual->SetStaticMesh(SphereVisualAsset.Object);  
    SphereVisual->SetRelativeLocation(FVector(0.0f, 0.0f, -40.0f));  
    SphereVisual->SetWorldScale3D(FVector(0.8f));  
}
```
# ActorComponent

## 获取组件依附的 Actor

```c++
AActor* ParentActor = MyComponent->GetOwner();
```
## 获取 Actor 上的组件

```c++
UMyComponent* MyComp = MyActor->FindComponentByClass<UMyComponent>();
```

![[e5c08b782929e1ae15f583195a515fb2_MD5.jpg]]

## 创建

## 销毁

```c++
MyActorComponent->DestroyComponent(); //销毁UActorComponent
```


# Pawn 类
## PlayerController 控制默认玩家
```c++
AutoPossessPlayer = EAutoReceiveInput::Player0;
```
![[Pasted image 20230829162058.png]]

# PlayerController
## 查找 Player 位置
```c++
FVector MyCharacter = GetWorld()->GetFirstPlayerController()->GetPawn()->GetActorLocation();
```
## GetPlayerController
```c++
// 查找处理本地玩家控制的actor。
APlayerController* OurPlayerController = UGameplayStatics::GetPlayerController(this, 0);
```

# 物理
## 刚体与图元组件
`RigidBody`/`Primitive Component`

在 Unity 中，假如要为 GameObject 赋予物理特征，首先必须为其提供刚体组件。

在虚幻引擎中，任何图元组件（C++ 中的 `UPrimitiveComponent`）都可以是物理对象。
一些常见图元组件如下：
- 形状组件（胶囊体、球体和盒体）
- 静态网格体组件
- 骨骼网格体组件

Unity 将碰撞和可视性划分到不同的组件中，虚幻引擎则将 **"潜在的物理碰撞"（potentially physical）** 和 **"潜在的可视效果"（potentially visible）** 组合到了单个图元组件中。凡是在世界中具有形状的组件，只要能通过物理方式渲染或交互，都是 `PrimitiveComponent` 的子类。

## 刚体/碰撞组件

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
## RayTrace
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


## 输入事件
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


# 相机
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

# 粒子
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