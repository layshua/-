原文地址: [Blueprints vs. C++ : How They Fit Together and Why You Should Use Both](https://link.zhihu.com/?target=http%3A//awforsythe.com/unreal/blueprints_vs_cpp/)

youtube: [Blueprints vs. C++: How They Fit Together and Why You Should Use Both](https://link.zhihu.com/?target=https%3A//youtu.be/VMZftEVDuCE)

bilibili: [蓝图 vs C++ 它们如何一起工作 为什么你应该一起使用](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV11o4y1C7aW%3Fspm_id_from%3D333.999.0.0)

## 简介

虚幻引擎提供多种游戏编程方式：你可以使用 C++ 或者 蓝图 (Blueprints), 也可以使用一些脚本语言 (比如: Python、Lua、TypeScript ), 本文讨论是 C++ 和蓝图。C++ 和蓝图之间的差异很大，C++ 是一种基于文本的编程语言; 而蓝图像是专门为更上层的游戏流程而量身定制：其编程方式是将事件、流程控制、函数调用等用图形节点串联起来，通过编辑器就可以定义变量、方法和接口

![](1671947338927.png)

C++ 和蓝图的差异这么大，你可能会疑问：“我应该用 C++ 还是蓝图来开发游戏呢？” 其实你不应该这么问，虚幻引擎本身的设计就是让 C++ 和蓝图互补，所以你应该问：“C++ 和蓝图分别适合用在什么地方？”

## C++ 和蓝图的共同点

如果我希望在游戏开始时生成一个 Actor , 用 C++ 实现方式如下：

```
void ACoyote::BeginPlay()
{
    Super::BeginPlay();

    if (bSpawnAnvil)
    {
        const FVector SpawnOffset(100.0f, 0.0f, 1500.0f);
        const FVector SpawnLocation = GetActorTransform().TransformPosition(SpawnOffset);
        const FTransform SpawnTransform(FQuat::Identity, SpawnLocation);

        FActorSpawnParameters SpawnInfo;
        SpawnInfo.Owner = this;

        AAnvil *Anvil = GetWorld()->SpawnActor<AAnvil>(AnvilClass, SpawnTransform, SpawnInfo);
        if (Anvil)
        {
            Anvil->BeginFalling();
        }
    }
}
```

而蓝图的实现方式如下：

![](1671947339015.png)

这两种方式看起来差异很大，实际上执行的结果一致。实现一个功能你可以使用 C++ 也可以使用蓝图，无论你使用哪一种方式，你其实都是在编程。**编程不是按照语法规则编写代码，而是定义程序运行时的行为**，因此我们大部分工作都是在搭建好的项目框架内创建新的对象，定义其行为以及与其他对象之间的交互规则。换句话说，编程其实就是在设计软件

## 设计理念：高级与低级

当你设计一个像《堡垒之夜》这种规模宏大的游戏软件时，从垂直的角度去思考问题会好一些；通常我们的目标是实现一些复杂炫酷的高级功能，那么就需要将高级功能拆分为很多个可以实现的基础功能

![](1671947339061.png)

如果你想你的游戏有个超酷的导弹发射器，它发射的导弹能追踪敌人并爆炸，要怎么实现呢？万丈高楼平地起，需要实现这些炫酷的功能那必须要有坚实的底层基础，而虚幻引擎、项目框架以及一些第三方插件就是我们的底层基础

![](1671947339151.png)

而作为游戏开发工程师（码农）的工作内容，就是把这个设计过程中缺失的实现细节填补完整

## 设计示例：武器系统

对于武器系统，可以先继承引擎提供的类型来构造我们需要的类，然后决定每个类的具体职责以及和其他对象的交互方式，以及它们需要用到哪些引擎底层功能

![](1671947339222.png)

例如：Weapon 类需要处理玩家的输入、弹药的管理、开火以及冷却等逻辑；以及酷炫的武器皮肤、枪花特效等可视化内容。而为了实现这些，我们需要用的引擎底层功能则有如下：

*   为了实现武器的开火功能，我们需要使用 APawn 提供的输入组件 InputComponent
*   为了检测武器是否击中目标，我们需要使用引擎提供的射线检测 LineTraceSingleByChannel
*   为了让受击目标响应被击中，我们需要使用内置的伤害系统 TakeDemage
*   为了让这一切表现的更加酷炫，我们就需要引擎提供的渲染、动画、特效以及声音系统

以下是 C++ 实现武器的射线检测功能

```
void AWeapon::RunWeaponTrace(const FTransform & MuzzleTransform, float TraceDistance)
{
    const FVector TraceStart = MuzzleTransform.GetLocation();
    const FVector TraceEnd = TraceStart + (MuzzleTransform.GetUnitAxis(EAxis::X) * TraceDistance);
    const FCollisionQueryParams QueryParams(TEXT("WeaponTrace"), false, this);

    FHitResult Hit;
    if (GetWorld()->LineTraceSingleByChannel(Hit, TraceStart, TraceEnd, ECC_WeaponFire, QueryParams))
    {
        if (Hit.Actor.IsValid())
        {
            const float DamageAmount = 1.0f;
            const FVector ShotFromDirection = (TraceEnd - TraceStart).GetSafeNormal();
            const TSubclass<UDamageType> DamageTypeClass = UDamageType_WeaponFire::StaticClass();
            const FPointDamageEvent DamageEvent(DamageAmount, Hit, ShotFromDirection, DamageTypeClass);
            Hit.Actor->TakeDamage(DamageAmount, DamageEvent, OwningController, this);
        }
    }
}
```

蓝图也可以轻松实现相同的功能：

![](1671947339313.png)

## 设计理念：脚本与编程

即便是设计一个简单的功能，比如武器：在不同的抽象层级上，都有不同的问题需要去解决。在最底层可能遇到的问题是：“如何向操作系统申请存储武器对象所需的内存空间？”，在最高层的表现上则会有这类问题：“当我被敌人集火时，我身上应该显示哪种紫色阴影？”

![](1671947339405.png)

那些底层问题通常属于**引擎编程**领域，引擎编程涉及所有游戏开发需要的核心技术，而不局限于某一类型的游戏。当我们开始基于这些核心技术来开发特定游戏，并实现这个游戏的玩法时，我们就进入了**游戏编程**领域

基于搭建好的项目框架，我们还需做很多工作来充实玩家的游戏体验，这些工作我们称之为**脚本**。脚本编写侧重于更高层次的功能，例如：游戏的整体流程和进度、不同游戏对象之间的交互、或者某个具体游戏对象的外观和行为方式等

因此，”通常 “编程” 是指解决一些底层的问题，而 “脚本” 则是指在高级系统之上填充细节

## C++ 和蓝图作为编程和脚本

C++ 是一门编程语言，而蓝图则是脚本系统。因此 C++ 适合实现游戏底层系统，而蓝图则更适合用于定义高级行为、交互、资产整合以及一些需要微调的装饰性细节等。一般来说，C++ 和蓝图大多都是按照这样的界限被使用

![](1671947339672.png)

关于虚幻引擎你要明白一点就是: 它并没有为你清晰的划分这条界限，也没指定你必须使用 C++ 或蓝图来解决某些问题，你需要自己划分这条界线。你大可以在前期利用蓝图快速搭建游戏原型，然后在游戏设定变得清晰的时候，再将部分蓝图重构为 C++

虚幻引擎的设计旨在提供这种灵活性：

*   引擎不存在单独的 “脚本 API”——无论你使用 C++ 还是蓝图，你都在以几乎相同的方式使用同一套引擎系统
*   C++ 和蓝图的高度集成使得两者之间更加容易互通，你可以轻松在 C++ 和蓝图之间切换
*   所有蓝图内容都可以” 翻译” 成 C++

以后如果有人问你 “项目开发中，C++ 或者蓝图哪个更好？” 你就可以对他笑而不语

## 性能: 编译 C++ / 蓝图

_译者注：本段与原文有很大差异，因为原文阐述了很细的点，本文只浅显阐述基本内容_

在 C++ 和蓝图都可选的情况下，选择哪一种方式实现功能通常需要从多个方面来考虑，经常被提到的一个重要因素是性能；当你编写 C++ 代码时，最后得到一个 .cpp 格式的文本文件：

```
void AMissile::Tick(float DeltaSeconds)
{
    Super::Tick(DeltaSeconds);

    const float OffsetForward = MovementSpeed * DeltaSeconds;
    const FVector Offset(OffsetForward, 0.0f, 0.0f);
    AddActorLocalOffset(Offset);
}
```

从源码构建项目时，C++ 代码会被编译为机器码（也就是能够直接在 CPU 上运行的处理器指令列表）

```
; AMissile::Tick
    push 40 53
     sub 48 83 EC 60
  movaps 0F 29 74 24 50
     mov 48 8B D9
  movaps 0F 28 F1
            ; AActor::Tick
    call FF 15 F9 9B 00 00
   mulss F3 0F 59 B3 F0 02 00 00
     lea 48 8D 54 24 30
     mov C7 44 24 48 00 00 00 00
     xor 45 33 C9
     mov 8B 44 24 48
   xorps 0F 57 D2
     xor 45 33 C0
     mov 89 44 24 38
  movaps 0F 28 C6
     mov C6 44 24 20 00
unpcklps 0F 14 C2
     mov 48 8B CB
   movsd F2 0F 11 44 24 30
            ; AActor::AddActorLocalOffset
    call FF 15 01 A0 00 00
  movaps 0F 28 74 24 50
     add 48 83 C4 60
     pop 5B
     ret C3
```

而使用蓝图最终会得到一张由一堆节点组成的事件图表，并保存在一个蓝图资产中

![](1671947339759.png)

蓝图也会被编译，但不会编译为机器码。脚本编译器会将蓝图编译成脚本字节码 (一种可移植的中间代码)，由引擎的脚本虚拟机在运行时执行

```
; ExecuteUbergraph_Missile
    EX_ComputedJump 4E
   EX_LocalVariable 00 C0 51 A3 FA 6A 01 00 00 ; ReceiveTick entry
      EX_Tracepoint 5E
  EX_WireTracepoint 5A
             EX_Let 0F 60 52 A3 FA 6A 01 00 00 ; OffsetForward
   EX_LocalVariable 00 60 52 A3 FA 6A 01 00 00 ; OffsetForward
        EX_CallMath 68 00 57 00 D0 6A 01 00 00 ; UKismetMathLibrary::Multiply_FloatFloat
   EX_LocalVariable 00 80 64 A3 FA 6A 01 00 00 ; - A: DeltaSeconds
EX_InstanceVariable 01 C0 5B A3 FA 6A 01 00 00 ; - B: MovementSpeed
EX_EndFunctionParms 16
             EX_Let 0F E0 63 A3 FA 6A 01 00 00 ; Offset
   EX_LocalVariable 00 E0 63 A3 FA 6A 01 00 00 ; Offset
        EX_CallMath 68 00 D8 02 D0 6A 01 00 00 ; UKismetMathLibrary::MakeVector
   EX_LocalVariable 00 60 52 A3 FA 6A 01 00 00 ; - X: OffsetForward
      EX_FloatConst 1E 00 00 00 00             ; - Y: 0.0
      EX_FloatConst 1E 00 00 00 00             ; - Z: 0.0
EX_EndFunctionParms 16
      EX_Tracepoint 5E
   EX_FinalFunction 1C 00 72 03 CE 6A 01 00 00 ; AActor::K2_AddActorLocalOffset
   EX_LocalVariable 00 E0 63 A3 FA 6A 01 00 00 ; - DeltaLocation: Offset
           EX_False 28                         ; - bSweep: false
   EX_LocalVariable 00 20 65 A3 FA 6A 01 00 00 ; - [out] HitResult
           EX_False 28                         ; - bTeleport: false
EX_EndFunctionParms 16
  EX_WireTracepoint 5A
          EX_Return 04
```

编译器会对我们编写的 C++ 代码进行一些优化，从而对性能更加友好。但是蓝图编译成的机器码则不会，因此蓝图相对 C++ 来说，消耗更大。因此对于一些复杂的数学计算以及需要被频繁调用的函数（Tick），那么最好用 C++ 实现

## 性能：结论和分析

上文关于性能方面，我们能得出什么结论？

如果有两个等效的函数，一个用 C++ 编写另一个用蓝图编写，那么 C++ 函数会更快；C++ 函数可以在 CPU 级别进行全面优化，并且不会产生任何脚本执行开销

![](1671947339844.png)

顺便说一下，引擎的 [Blueprint Nativization](https://link.zhihu.com/?target=https%3A//docs.unrealengine.com/en-US/ProgrammingAndScripting/Blueprints/TechnicalGuide/NativizingBlueprints/index.html) 功能就是用来避免这种开销。如果启用了蓝图本地化，那么脚本编译器不会生成脚本字节码，而是会吐出 C++ 源代码，该源代码可以直接编译为机器代码；生成的代码不具备可读性，也无法编辑：

```
void AWeapon_C__pf2513711887::bpf__RunWeaponTrace__pf(FTransform bpp__MuzzleTransform__pf, float bpp__TraceDistance__pf)
{
    FVector bpfv__TraceEnd__pf(EForceInit::ForceInit);
    FVector bpfv__TraceStart__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_MakeVector_ReturnValue__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_BreakTransform_Location__pf(EForceInit::ForceInit);
    FRotator bpfv__CallFunc_BreakTransform_Rotation__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_BreakTransform_Scale__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_TransformLocation_ReturnValue__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_GetDirectionUnitVector_ReturnValue__pf(EForceInit::ForceInit);
    TArray<AActor*> bpfv__Temp_object_Variable__pf{};
    FHitResult bpfv__CallFunc_LineTraceSingle_OutHit__pf{};
    bool bpfv__CallFunc_LineTraceSingle_ReturnValue__pf{};
    bool bpfv__CallFunc_BreakHitResult_bBlockingHit__pf{};
    bool bpfv__CallFunc_BreakHitResult_bInitialOverlap__pf{};
    float bpfv__CallFunc_BreakHitResult_Time__pf{};
    float bpfv__CallFunc_BreakHitResult_Distance__pf{};
    FVector bpfv__CallFunc_BreakHitResult_Location__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_BreakHitResult_ImpactPoint__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_BreakHitResult_Normal__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_BreakHitResult_ImpactNormal__pf(EForceInit::ForceInit);
    UPhysicalMaterial* bpfv__CallFunc_BreakHitResult_PhysMat__pf{};
    AActor* bpfv__CallFunc_BreakHitResult_HitActor__pf{};
    UPrimitiveComponent* bpfv__CallFunc_BreakHitResult_HitComponent__pf{};
    FName bpfv__CallFunc_BreakHitResult_HitBoneName__pf{};
    int32 bpfv__CallFunc_BreakHitResult_HitItem__pf{};
    int32 bpfv__CallFunc_BreakHitResult_FaceIndex__pf{};
    FVector bpfv__CallFunc_BreakHitResult_TraceStart__pf(EForceInit::ForceInit);
    FVector bpfv__CallFunc_BreakHitResult_TraceEnd__pf(EForceInit::ForceInit);
    float bpfv__CallFunc_ApplyPointDamage_ReturnValue__pf{};
    bool bpfv__CallFunc_IsValid_ReturnValue__pf{};
    int32 __CurrentState = 1;
 do
 {
 switch( __CurrentState )
 {
        case 1:
 {
                UKismetMathLibrary::BreakTransform(bpp__MuzzleTransform__pf,
 /*out*/ bpfv__CallFunc_BreakTransform_Location__pf,
 /*out*/ bpfv__CallFunc_BreakTransform_Rotation__pf,
 /*out*/ bpfv__CallFunc_BreakTransform_Scale__pf);
                bpfv__TraceStart__pf = bpfv__CallFunc_BreakTransform_Location__pf;
 }
        case 2:
 {
                bpfv__CallFunc_MakeVector_ReturnValue__pf = UKismetMathLibrary::MakeVector(
                    bpp__TraceDistance__pf, 0.000000, 0.000000);
                bpfv__CallFunc_TransformLocation_ReturnValue__pf = UKismetMathLibrary::TransformLocation(
                    bpp__MuzzleTransform__pf, bpfv__CallFunc_MakeVector_ReturnValue__pf);
                bpfv__TraceEnd__pf = bpfv__CallFunc_TransformLocation_ReturnValue__pf;
 }
        case 3:
 {
                bpfv__CallFunc_LineTraceSingle_ReturnValue__pf = UKismetSystemLibrary::LineTraceSingle(
 this, bpfv__TraceStart__pf, bpfv__TraceEnd__pf, ETraceTypeQuery::TraceTypeQuery3,
 false, bpfv__Temp_object_Variable__pf, EDrawDebugTrace::None,
 /*out*/ bpfv__CallFunc_LineTraceSingle_OutHit__pf, true,
 FLinearColor(1.000000,0.000000,0.000000,1.000000),
 FLinearColor(0.000000,1.000000,0.000000,1.000000), 5.000000);
 }
        case 4:
 {
 if (!bpfv__CallFunc_LineTraceSingle_ReturnValue__pf)
 {
                    __CurrentState = -1;
                    break;
 }
 }
        case 5:
 {
                UGameplayStatics::BreakHitResult(bpfv__CallFunc_LineTraceSingle_OutHit__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_bBlockingHit__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_bInitialOverlap__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_Time__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_Distance__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_Location__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_ImpactPoint__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_Normal__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_ImpactNormal__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_PhysMat__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_HitActor__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_HitComponent__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_HitBoneName__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_HitItem__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_FaceIndex__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_TraceStart__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_TraceEnd__pf);
                bpfv__CallFunc_IsValid_ReturnValue__pf = UKismetSystemLibrary::IsValid(
                    bpfv__CallFunc_BreakHitResult_HitActor__pf);
 if (!bpfv__CallFunc_IsValid_ReturnValue__pf)
 {
                    __CurrentState = -1;
                    break;
 }
 }
        case 6:
 {
                bpfv__CallFunc_GetDirectionUnitVector_ReturnValue__pf = UKismetMathLibrary::GetDirectionUnitVector(
                    bpfv__TraceStart__pf, bpfv__TraceEnd__pf);
                UGameplayStatics::BreakHitResult(bpfv__CallFunc_LineTraceSingle_OutHit__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_bBlockingHit__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_bInitialOverlap__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_Time__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_Distance__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_Location__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_ImpactPoint__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_Normal__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_ImpactNormal__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_PhysMat__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_HitActor__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_HitComponent__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_HitBoneName__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_HitItem__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_FaceIndex__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_TraceStart__pf,
 /*out*/ bpfv__CallFunc_BreakHitResult_TraceEnd__pf);
                bpfv__CallFunc_ApplyPointDamage_ReturnValue__pf = UGameplayStatics::ApplyPointDamage(
                    bpfv__CallFunc_BreakHitResult_HitActor__pf, 1.000000,
                    bpfv__CallFunc_GetDirectionUnitVector_ReturnValue__pf,
                    bpfv__CallFunc_LineTraceSingle_OutHit__pf, bpv__OwningController__pf, this,
                    CastChecked<UClass>(
                        CastChecked<UDynamicClass>(AWeapon_C__pf2513711887::StaticClass())->UsedAssets[0],
                        ECastCheckedType::NullAllowed));
                __CurrentState = -1;
                break;
 }
        default:
            break;
 }
 } while( __CurrentState != -1 );
}
```

但最终产生的是相同的原生函数调用，无需运行在脚本虚拟机中

![](1671947340064.png)

因此我们可以对蓝图节点中开销比较大的位置用 C++ 重构，而这些位置可能包括各种底层系统，或是处理大量运算的循环遍历，以及涉及到处理大量 Actor 对象

如果一个在蓝图中实现的功能，你花了一周的时间用 C++ 重写整个功能，实现了 20 倍的性能提升，但其实用蓝图实现该功能所花费的时间也就 0.1ms ，那么你其实在做” 无用功”。你应该使用分析器衡量性能，并根据具体数据来做判断

![](1671947340128.png)

## 项目组织: 类的设计

游戏编程不仅仅是实现某个函数，虚幻是一个面向对象的引擎，因此通常将这些函数编写为类的一部分；在开始实现类的函数之前，首先需要定义类

```
/** Flies forward from where it's spawned, exploding on contact. */
UCLASS()
class AMissile : public AActor
{
 GENERATED_BODY()
  // Declare member variables and member functions here
};
```

定义一个类意味着确定它应该负责什么，然后弄清楚它需要哪些属性和函数，同时还将决定哪些属性和方法会对外开放。 C++ 的类定义写在头文件中：

```
/** Flies forward from where it's spawned, exploding on contact. */
UCLASS()
class AMissile : public AActor
{
 GENERATED_BODY()

public:
 /** Root collision sphere. */
 UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Components")
 class USphereComponent* SomeComponent;

public:
 /** How fast we should move forward, in centimeters per second. */
 UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Missile")
    float MovementSpeed;

 /** If we fly this far without hitting anything, we'll explode. */
 UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Missile")
    float SelfDestructDistance;

private:
 /** How far we've flown since spawning. */
 UPROPERTY(VisibleAnywhere, Category="Missile|State")
    float DistanceTraveled;

public:
 AMissile(const FObjectInitializer& ObjectInitializer);
    virtual void Tick(float DeltaSeconds) override;

private:
 void Explode(const FHitResult& Hit);
};
```

而函数的实现，一般写在对应的. cpp 文件中

```
AMissile::AMissile(const FObjectInitializer &ObjectInitializer)
    : Super(ObjectInitializer)
{
    PrimaryActorTick.bCanEverTick = true;
    PrimaryActorTick.bStartWithTickEnabled = true;

    CollisionComponent = ObjectInitializer.CreateDefaultSubobject<USphereComponent>(this, TEXT("CollisionComponent"));
    CollisionComponent->SetCollisionProfileName(UCollisionProfile::BlockAllDynamic_ProfileName);
    RootComponent = CollisionComponent;

    MovementSpeed = 500.0f;
}

void AMissile::Tick(float DeltaSeconds)
{
    Super::Tick(DeltaSeconds);

    const float OffsetForward = MovementSpeed * DeltaSeconds;
    const FVector Offset(OffsetForward, 0.0f, 0.0f);
    DistanceTraveled += OffsetForward;

    FHitResult Hit;
    AddActorLocalOffset(Offset, true, &Hit);
    if (Hit.bBlockingHit || SelfDestructDistance > 0.0f && DistanceTraveled >= SelfDestructDistance)
    {
        Explode(Hit);
    }
}

void AMissile::Explode(const FHitResult &Hit)
{
    SetActorTickEnabled(false);
    SetLifeSpan(1.0f);
}
```

蓝图则大致相当于这两个文件 (.h 和 .cpp) 之和，蓝图的父类及其组件、属性和函数列表构成了它的类定义

![](1671947340200.png)

事件图（和其他函数图）包含函数实现

![](1671947340252.png)

在这个级别上，C++ 和蓝图定义并实现新的类或数据类型的方式大致相等，但是当谈到类之间的依赖关系时差异就出现了

## 设计理念: 类型和依赖

在 C++ 或者编辑器中创建类、结构体、枚举，都是在定义一个新**类型**

![](1671947340311.png)

一种类型需要知道其它某个类型的时候，就产生依赖关系；如果可以的话，依赖最好做到单向。例如：游戏中发射导弹的发射器，那就意味着一种单向依赖：发射器需要导弹的具体类型才能够生成导弹实例，但是导弹并不需要知道有关发射器的任何逻辑

![](1671947340496.png)

假设现在有个新的需求：游戏内任一时刻, 只允许一枚导弹存在。发射器想发射新的导弹，必须要等到旧的导弹销毁了才能发射新的导弹，该怎么做？我们可以让导弹调用发射器上的相关函数来让发射器知道可以再次发射导弹

![](1671947340556.png)

但这会创建双向依赖关系。能否发射导弹这是发射器的事儿，而导弹该做的事儿就是找到目标并爆炸。因此，为了保持单向依赖，我们给导弹一个在爆炸时调用的委托 (蓝图中，委托被称为事件调度器 (Event Dispatcher))。导弹只需在爆炸的时候调用这个委托，无需关心调用之后会发生什么。而发射器则可以监听这个委托并绑定回调方法用于更新自己内部的状态

![](1671947340777.png)

随着项目变得越来越大，管理这些依赖关系变得越来越重要，要确保代码不同部分之间的边界得到明确定义

## 项目组织: C++ 模块

C++ 中实现这种分离的一种方法是使用模块 (modules)。通常有一个主游戏模块，其中包含核心游戏玩法类，例如 GameMode、PlayerController 和 Pawn 等。随着项目变得越来越复杂，可能会将不同的功能和系统拆分为各自独立的模块

![](1671947340830.png)

为了让一个模块中的类引用另一个模块中的类，两个模块之间需要存在一个显式依赖关系，被引用的类或函数需要作为模块公共 API 的一部分导出

![](1671947340900.png)

由于模块依赖通常应该始终是严格的单向，这导致了一种分层架构：

![](1671947340979.png)

在这个例子中，武器模块位于核心游戏模块下方，所以我们可以让我们的 Pawn 生成一个 Weapon，并且 Pawn 可以从 Weapon 类调用函数和访问数据，但是 Weapon 永远不应该知道 Pawn 的任何信息。我们加这样的限制，就形成了一种设计方式：武器模块不应该依赖于核心模块

如果我们尝试编写违反既定设计的代码，那么构建系统将不允许这样做：我们将收到一个链接器错误，表明我们正在尝试使用来自非显式依赖项的模块的代码

```
// Compile error on #include:
// (Module has not been added as a dependency)

 [1/4] Missile.cpp
    E:\Cobalt\Source\CobaltWeapons\Private\Missile.cpp(7):
      fatal error C1083:
        Cannot open include file: 'CobaltPlayerController.h':
          No such file or directory /* [in the include path for this module] */

// Linker error on use of class:
// (Module is a dependency, but class is not exported)

 [1/2] UE4Editor-CobaltWeapons.dll
    Missile.cpp.obj : error LNK2019:
      unresolved external symbol
 "private: static class UClass * __cdecl
         ACobaltPlayerController::GetPrivateStaticClass(void)"
 (?GetPrivateStaticClass@ACobaltPlayerController@@CAPEAVUClass@@XZ)
      referenced in function
 "private: void __cdecl
         AMissile::Explode(struct FHitResult const &)"
 (?Explode@AMissile@@AEAAXABUFHitResult@@@Z)
```

这通常表明我们需要更仔细地考虑我们在做什么，要么更改我们的代码以更好地适应既定设计，要么就得重新评估这些设计限制是否真的合理？

模块如果使用得当，带来的好处有如下几点：

*   使用模块可以控制构建时间
*   在团队中可以更轻松地确定哪些团队成员对代码库的不同部分拥有所有权
*   减轻了认知负担，在单个模块内工作，不需要考虑其他模块的内容只专注当前模块的功能设计

当然，模块化是把双刃剑。将代码分离到单独的模块最关键的好处就是：让你能够按照既有的设计模式来添加新类型或者依赖项。而最主要的缺点是：在你添加新的类型或依赖时，他会强制你去考虑你的设计是否合理

## 项目组织: C++/ 蓝图 依赖

蓝图中没有这样的模块概念，你可以认为项目里的蓝图是个依赖于所有 C++ 模块的” 特殊模块”。蓝图可以自由引用 C++ 代码模块中声明的任何类型 (只要它被标记为: BlueprintType)

![](1671947341036.png)

蓝图也是资产，所以对蓝图的管理也相当于资产管理，项目的资产管理因项目和团队而异。值得指出的是，可以使用编辑器的引用查看器来获取蓝图之间依赖关系，这些信息非常有用

![](1671947341078.png)

蓝图中并不存在模块化的概念，但是你得知道，C++ 和蓝图之间其实存在概念意义上的单向模块依赖关系：蓝图可以依赖 C++ 类型，但 C++ 类不会知道任何蓝图类型相关的信息（比如：无法直接调用蓝图内的方法，获取蓝图定义的属性）

![](1671947341341.png)

## 设计示例: 蓝图 到 C++

假设我们一直在蓝图中工作，我们有一个自定义的 Pawn 类和 Weapon 类

![](1671947341398.png)

我们需要在这两个类之间实现一种简单的单向交互：在触发 BeginPlay 事件时 Pawn 对象要生成一个 Weapon 对象，如果玩家按下开火按钮，Pawn 对象就会调用 Weapon 的 Fire 函数

![](1671947341470.png)

我们已经设置好了武器蓝图，当执行 Fire 函数时，我们会执行一次射线检测，然后生成粒子特效。现在假设我们想开始将一些核心类重构为 C++

![](1671947341569.png)

如果我们将 Pawn 类移到 C++ 中， Weapon 类保留在蓝图，那么我们必须面对这样一个事实，即在我们的 C++ 模块中，Weapon 类尚不存在；由于虚幻的反射系统，我们依然可以生成一个 Weapon 对象，任何 UObject 类，无论在哪里定义，必定存在一个 UClass 对象与之对应，只要我们能够获得 Weapon 类的 UClass 引用，就能够生成它的 Actor 对象

![](1671947341636.png)

C++ 重构的 Pawn 类如下:

```
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/DefaultPawn.h"

#include "CobaltPawn.generated.h"

UCLASS()
class ACobaltPawn : public ADefaultPawn
{
 GENERATED_BODY()

public:
 UPROPERTY()
    TSubclassOf<AActor> WeaponClass;

 UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Cobalt")
    AActor* Weapon;

public:
 ACobaltPawn(const FObjectInitializer& ObjectInitializer);

protected:
    virtual void BeginPlay() override;
    virtual void SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) override;

private:
 UFUNCTION() void OnFirePressed();
};
```

这里我们给了 Pawn 两个属性: WeaponClass 是要生成的武器类的 UClass 引用 ，而 Weapon 则是用来保存生成的武器实例

```
#include "CobaltPawn.h"

#include "UObject/ConstructorHelpers.h"
#include "Components/InputComponent.h"

ACobaltPawn::ACobaltPawn(const FObjectInitializer& ObjectInitializer)
 : Super(ObjectInitializer)
{
    static ConstructorHelpers::FClassFinder<AActor>WeaponClassFinder(TEXT("/Game/Core/Weapon"));
    WeaponClass = WeaponClassFinder.Class;
}
```

在构造函数中，我们根据路径直接获取武器蓝图类的 UClass 引用然用 WeaponClass 属性来保存它，当然这种根据路径获取的硬编码方式不太好，下文会给出替代方案。在 中`BeginPlay`我们可以生成该类的一个实例

```
void ACobaltPawn::BeginPlay()
{
    Super::BeginPlay();

    if (WeaponClass != nullptr)
    {
        FActorSpawnParameters SpawnInfo;
        SpawnInfo.Owner = this;
        SpawnInfo.Instigator = this;

        const FTransform SpawnOffset(FQuat::Identity, FVector(0.0f, 15.0f, -15.0f));
        const FTransform SpawnTransform = GetActorTransform() * SpawnOffset;
        Weapon = GetWorld()->SpawnActor<AActor>(WeaponClass, SpawnTransform, SpawnInfo);
    }
}
```

注意，Weapon 属性的类型是 AActor*，因为 Weapon 类是在蓝图中定义的，所以 C++ 实现的 Pawn 类获取不到 Weapon 类的任何信息 (只知道它是 Actor 的子类)

```
void ACobaltPawn::SetupPlayerInputComponent(UInputComponent *PlayerInputComponent)
{
 Super::SetupPlayerInputComponent(PlayerInputComponent);

 PlayerInputComponent->BindAction(TEXT("Fire"), IE_Pressed, this, &ACobaltPawn::OnFirePressed);
}

void ACobaltPawn::OnFirePressed()
{
 if (Weapon)
    {
        // Manually call a Blueprint function from C++: this is dumb and you shouldn't do it
        UFunction *FireFunction = Weapon->FindFunction(TEXT("Fire"));
 if (FireFunction)
        {
 Weapon->ProcessEvent(FireFunction, nullptr);
        }
    }
}
```

这意味着我们不能调用 Weapon 类的 Fire 函数，技术上我们可以通过名称查找函数并动态调用它，但这种做法非常不推荐

好的解决方案是将 Weapon 类重构为 C++ 类。但是 Weapon 类会生成粒子特效，而对于这些装饰性质的效果，我们更偏向于在蓝图中实现。最终在 C++ 中定义 Weapon 基类，然后蓝图中继承它并进一步完善细节

![](1671947341684.png)

在我们的 C++ Weapon 类中，我们实现底层相关功能：Fire 函数，以及它需要的其他数据或辅助函数等

```
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"

#include "Weapon.generated.h"

UCLASS()
class AWeapon : public AActor
{
 GENERATED_BODY()

public:
 /** Placed at the end of the weapon, +X pointing out in the direction of fire. */
 UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Components")
 class USceneComponent* MuzzleComponent;

public:
 AWeapon(const FObjectInitializer& ObjectInitializer);
 void Fire();

private:
 void RunWeaponTrace(const FTransform& MuzzleTransform, float TraceDistance);
};
```

我们可以对 Pawn 进行一些更改：

```
public:
 UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Cobalt")
    TSubclassOf<class AWeapon> WeaponClass;

 UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Cobalt")
 class AWeapon* Weapon;
```

在. cpp 文件中, 我们不在需要直接引用蓝图，我们只需要将 WeaponClass 初始化为我们的 Weapon 基类

```
#include "Weapon.h"

ACobaltPawn::ACobaltPawn(const FObjectInitializer& ObjectInitializer)
 : Super(ObjectInitializer)
{
    WeaponClass = AWeapon::StaticClass();
}
```

生成的方法区别不大，只是我们可以更加精确定义它的类型为 AWeapon

Weapon = GetWorld()->SpawnActor< AWeapon >(WeaponClass, SpawnTransform, SpawnInfo);

输入绑定方法中，我们可以正常调用 Fire 函数了

```
void ACobaltPawn::OnFirePressed()
{
    if (Weapon)
    {
        Weapon->Fire();
    }
}
```

## 设计示例: 用 C++ 实现所有功能

我们要坚信：” 真正的” 程序员是不屑于用蓝图的，万物皆可 C++ ; 所以我们需要在 C++ 的 Weapon 类添加一个网格组件（武器肯定是有自己的样式的嘛），其次因为枪口会产生火花特效，因此我们还得添加一个粒子组件

```
public:
 /** Placed at the end of the weapon, +X pointing out in the direction of fire. */
 UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Components")
 class USceneComponent* MuzzleComponent;

 UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Components")
 class UStaticMeshComponent* MeshComponent;

public:
 UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Weapon")
 class UParticleSystem* MuzzleFlashParticleSystem;
```

在构造函数中，需要获取对武器资产的引用。我们可以使用静态 `FObjectFinder` 来确保此资产查找仅在游戏首次启动时执行

```
AWeapon::AWeapon(const FObjectInitializer& ObjectInitializer)
 : Super(ObjectInitializer)
{
    static ConstructorHelpers::FObjectFinder<UStaticMesh> WeaponMeshFinder(
 TEXT("StaticMesh'/Game/Assets/Weapon/SM_Weapon.SM_Weapon'"));
    static ConstructorHelpers::FObjectFinder<UParticleSystem> MuzzleFlashParticleSystemFinder(
 TEXT("ParticleSystem'/Game/Assets/Weapon/PS_Weapon_MuzzleFlash.PS_Weapon_MuzzleFlash'"));

    RootComponent = ObjectInitializer.CreateDefaultSubobject<USceneComponent>(this, TEXT("RootComponent"));

    MuzzleComponent = ObjectInitializer.CreateDefaultSubobject<USceneComponent>(this, TEXT("MuzzleComponent"));
    MuzzleComponent->SetupAttachment(RootComponent);
    MuzzleComponent->SetRelativeLocation(FVector(48.0f, 0.0f, 0.0f));

    MeshComponent = ObjectInitializer.CreateDefaultSubobject<UStaticMeshComponent>(this, TEXT("MeshComponent"));
    MeshComponent->SetupAttachment(RootComponent);
    MeshComponent->SetStaticMesh(WeaponMeshFinder.Object);
    MeshComponent->SetCollisionProfileName(UCollisionProfile::NoCollision_ProfileName);
    MeshComponent->SetRelativeLocation(FVector(20.0f, 0.0f, 0.0f));

    MuzzleFlashParticleSystem = MuzzleFlashParticleSystemFinder.Object;
}
```

我们还将创建和配置一个_UStaticMeshComponent_，确保我们手动输入正确的偏移量以匹配我们之前在编辑器中配置的内容，然后我们可以使用和蓝图中调用相同的功能在武器开火时产生枪口闪光粒子效果

```
void AWeapon::Fire()
{
    const FTransform MuzzleTransform = MuzzleComponent->GetComponentTransform();
    RunWeaponTrace(MuzzleTransform, 5000.0f);

    if (MuzzleFlashParticleSystem)
    {
        UGameplayStatics::SpawnEmitterAttached(MuzzleFlashParticleSystem, MuzzleComponent);
    }
}
```

上面的代码会产生以下问题：

*   资产路径硬编码到 C++ 源代码中，一旦资产路径有改动，代码就失效
*   游戏启动并注册了`AWeapon` 类，就会立即加载这些资产并且保留在内存中

而且，就我们组织项目的方式而言，这种做法也存在问题。要使用哪些资产的是非常高层的问题，而一般来说 C++ 基础类则旨在处理更底层的功能。相对来说，在蓝图中处理资产，会获得更自然的用户体验（所见即所得），即时的视觉反馈方便编辑和微调各种属性，同时资产的路径更改引擎也会帮我们重定向 (在编辑器内移动资产)

![](1671947341760.png)

## 设计示例: C++ 实现基础，蓝图填充细节

纯 C++ 实现缺点太多，那么让我们看看我们如何利用蓝图来处理这些装饰性细节

首先，让我们处理粒子特效部分。我不希望 C++ 类关心特定的视觉效果，我们只希望它确保这个特效能够生成。所以我们可以声明一个名为`PlayFireEffects`的函数（标记`BlueprintImplementableEvent`），而我们只需在 C++ 中调用这个函数，剩下的显示效果交给蓝图实现

```
UFUNCTION(BlueprintImplementableEvent, Category="Weapon")
void PlayFireEffects();
```

![](1671947341817.png)

接下来，让我们看看网格组件。如果我们的 C++ 代码需要控制网格——例如，在运行时打开或关闭碰撞——那么将这个组件声明为我们的 C++ 类的一部分是完全合理的，我们可以省略资产引用，让蓝图子类负责完全自定义网格组件。但在我们的示例中，网格纯粹是为了展示所以我们将把它完全排除在基类之外

新的 Weapon 基类头文件如下：

```
#pragma once
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "Weapon.generated.h"

UCLASS()
class AWeapon : public AActor
{
 GENERATED_BODY()

public:
 UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category="Components")
 class USceneComponent* MuzzleComponent;

public:
 AWeapon(const FObjectInitializer& ObjectInitializer);
 void Fire();

 UFUNCTION(BlueprintImplementableEvent, Category="Weapon")
 void PlayFireEffects();

private:
 void RunWeaponTrace(const FTransform& MuzzleTransform, float TraceDistance);
};
```

对应的. cpp 如下：

```
#include "Weapon.h"
#include "Components/SceneComponent.h"
#include "Engine/World.h"
#include "DamageType_WeaponFire.h"

static const ECollisionChannel ECC_WeaponFire = ECC_GameTraceChannel1;

AWeapon::AWeapon(const FObjectInitializer &ObjectInitializer) : Super(ObjectInitializer)
{
    RootComponent = ObjectInitializer.CreateDefaultSubobject<USceneComponent>(this, TEXT("RootComponent"));
    MuzzleComponent = ObjectInitializer.CreateDefaultSubobject<USceneComponent>(this, TEXT("MuzzleComponent"));
    MuzzleComponent->SetupAttachment(RootComponent);
    MuzzleComponent->SetRelativeLocation(FVector(100.0f, 0.0f, 0.0f));
}

void AWeapon::Fire()
{
    const FTransform MuzzleTransform = MuzzleComponent->GetComponentTransform();
    RunWeaponTrace(MuzzleTransform, 5000.0f);
    PlayFireEffects();
}

void AWeapon::RunWeaponTrace(const FTransform &MuzzleTransform, float TraceDistance)
{
    const FVector TraceStart = MuzzleTransform.GetLocation();
    const FVector TraceEnd = TraceStart + (MuzzleTransform.GetUnitAxis(EAxis::X) * TraceDistance);
    const FCollisionQueryParams QueryParams(TEXT("WeaponTrace"), false, this);

    FHitResult Hit;
    if (GetWorld()->LineTraceSingleByChannel(Hit, TraceStart, TraceEnd, ECC_WeaponFire, QueryParams))
    {
        if (Hit.Actor.IsValid())
        {
            const float DamageAmount = 1.0f;
            const FVector ShotFromDirection = (TraceEnd - TraceStart).GetSafeNormal();
            const TSubclassOf<UDamageType> DamageTypeClass = UDamageType_WeaponFire::StaticClass();
            const FPointDamageEvent DamageEvent(DamageAmount, Hit, ShotFromDirection, DamageTypeClass);
            Hit.Actor->TakeDamage(DamageAmount, DamageEvent, GetInstigatorController(), this);
        }
    }
}
```

进入编辑器，打开我们原来的武器蓝图，删除其他所有内容，只留下粒子效果和网格组件，然后将它的父类改为我们 C++ 中定义的 Weapon 类

![](1671947341858.png)

我们所要做的就是将`PlayFireEffects`事件与我们的粒子特效绑定，这样我们的武器就算完成了。现在我们有了一个扩展我们的 C++ Weapon 类的蓝图，编译蓝图后，我们最终会得到一个从蓝图生成的新武器类

![](1671947341997.png)

现在唯一的问题是：我们如何使用这个新的 `UBlueprintGeneratedClass` 而不是我们 C++ 中指定的 Weapon 基类。我们所要做的就是创建 Pawn 的蓝图子类，并更改该`WeaponClass`属性的默认值。然后我们可以使用这个新的 Pawn 类作为我们 GameMode 的默认 Pawn 类

![](1671947342059.png)

通过这种方式，我们保持了一个简洁的设计，其中的依赖关系为单向：我们有一个更高级的蓝图层，它构建在较底的 C++ 层之上，并且每一层都处理一组明确定义的职责，并确保层之间的最小耦合度

![](1671947342114.png)

## 传统编程 / 脚本细分

这是一个非常简单的例子，但我希望它能说明原理。这种以结构化，互补的方式使用蓝图和 C++ 开发模式，就是我之前提到的传统方法，在中间画了一条线

![](1671947342168.png)

这是一种经典模型，对于中大型游戏开发团队来说，往往是最佳方法。正如我们已经讨论过的，虚幻引擎提供了足够的灵活性，所以即便你开发的是小型游戏或者缺乏 C++ 开发经验，那么还有蓝图可以拯救你

## 设计示例: 蓝图函数库

如果实在觉得麻烦，也不一定必须将整个类重构为 C++。如果你创建一个基于 UBlueprintFunctionLibrary 的类，你可以添加静态 BlueprintCallable 函数。所以我们也可以保留我们原来的 Pawn 和 Weapon 蓝图类，并在需要时将单个函数重构为 C++（在蓝图函数库中）

![](1671947342213.png)

我们将射线检测代码放入蓝图函数库中，示例如下 `WeaponStatics.h`:

```
#pragma once

#include "CoreMinimal.h"
#include "Kismet/BlueprintFunctionLibrary.h"

#include "WeaponStatics.generated.h"

UCLASS()
class UWeaponStatics : public UBlueprintFunctionLibrary
{
 GENERATED_BODY()

public:
 UFUNCTION(BlueprintCallable, Category="Cobalt|Weapon", meta=(WorldContext="WorldContextObject"))
 static bool RunWeaponTrace( UObject* WorldContextObject, const FTransform& MuzzleTransform, float TraceDistance, FHitResult& OutHit);
};
```

对应的 `WeaponStatics` .cpp 如下：

```
#include "WeaponStatics.h"

#include "Engine/Engine.h"
#include "Engine/World.h"
#include "GameFramework/Actor.h"

#include "DamageType_WeaponFire.h"

static const ECollisionChannel ECC_WeaponFire = ECC_GameTraceChannel1;

bool UWeaponStatics::RunWeaponTrace(UObject *WorldContextObject, const FTransform &MuzzleTransform, float TraceDistance, FHitResult &OutHit)
{
    UWorld *World = GEngine->GetWorldFromContextObject(WorldContextObject, EGetWorldErrorMode::LogAndReturnNull);
    AActor *Actor = Cast<AActor>(WorldContextObject);
    if (World && Actor)
    {
        const FVector TraceStart = MuzzleTransform.GetLocation();
        const FVector TraceEnd = TraceStart + (MuzzleTransform.GetUnitAxis(EAxis::X) * TraceDistance);
        const FCollisionQueryParams QueryParams(TEXT("WeaponTrace"), false, Actor);

        if (World->LineTraceSingleByChannel(OutHit, TraceStart, TraceEnd, ECC_WeaponFire, QueryParams))
        {
            if (OutHit.Actor.IsValid())
            {
                const float DamageAmount = 1.0f;
                const FVector ShotFromDirection = (TraceEnd - TraceStart).GetSafeNormal();
                const TSubclassOf<UDamageType> DamageTypeClass = UDamageType_WeaponFire::StaticClass();
                const FPointDamageEvent DamageEvent(DamageAmount, OutHit, ShotFromDirection, DamageTypeClass);
                OutHit.Actor->TakeDamage(DamageAmount, DamageEvent, Actor->GetInstigatorController(), Actor);
            }
            return true;
        }
    }
    return false;
}
```

蓝图调用方式如下：

![](1671947342259.png)

通过以上示例我们了解了：引擎底层在编译或者运行时发生了什么，实际项目通常是如何组织并保持其简洁与可维护的设计模式，以及 C++ 和蓝图对性能以及项目组织形式的影响

既然我们已经讲完了那些比较底层的方面，那么就通过查看 C++ 和蓝图之间更加直接明显的区别来结束这个话题吧。C++ 和蓝图各自都有比对方更有优势的点

## 蓝图优势: 资产, 可视, 脚本事件

蓝图更适合处理资产和视觉效果。C++ 代码中只能盲目推测运行时将出现哪些资产；而蓝图本身就是资产，编辑蓝图时，你可以浏览所有的资产并在蓝图中使用他们而且所见即所得，而且方便调整各种参数

而在 C++ 代码直接引用资产时，会在编译的游戏模块和资产之间创建依赖关系。如果资产发生变化，需手动更新源代码；而蓝图则不用担心, 引擎会自动处理

![](1671947342490.png)

在流程控制方面，蓝图也有明显优势。如果你使用的是事件而不是函数，则可以充分利用事件和回调函数从而非常直观的方式编写异步代码。我们看下面一个例子：

如果我想让一个角色移动到某个位置 A，然后等待 3 秒，接着每半秒检查一次门是否打开，如果打开了就射击敌人直到敌人死亡，之后再穿过这个门到下一个位置 B

![](1671947342550.png)

在蓝图中，可以很快速的实现，并且整个过程一目了然非常清晰易懂。在 C++ 中也可以做同样的事情，实现如下：

```
void ATestSequence::Start()
{
    AAIController *Controller = Character ? Character->GetController<AAIController>() : nullptr;
    if (Controller && PointA)
    {
        Controller->ReceiveMoveCompleted.AddDynamic(this, &ATestSequence::OnFinishedMove);
        if (Controller->MoveToActor(PointA, 5.0f) == EPathFollowingRequestResult::RequestSuccessful)
        {
            MoveToPointARequestID = Controller->GetCurrentMoveRequestID();
        }
    }
}

void ATestSequence::OnFinishedMove(FAIRequestID RequestID, EPathFollowingResult::Type Result)
{
    if (RequestID == MoveToPointARequestID)
    {
        GetWorldTimerManager().SetTimer(CheckDoorTimer, this, &ATestSequence::CheckDoor, 3.0f);
    }
}

void ATestSequence::CheckDoor()
{
    if (Door && Door->IsOpen())
    {
        if (Character && Enemy)
        {
            Enemy->Died.AddUObject(this, &ATestSequence::OnEnemyDied);
            Character->SetAttackTarget(Enemy);
        }
    }
    else
    {
        GetWorldTimerManager().SetTimer(CheckDoorTimer, this, &ATestSequence::CheckDoor, 0.5f);
    }
}

void ATestSequence::OnEnemyDied()
{
    AAIController *Controller = Character ? Character->GetController<AAIController>() : nullptr;
    if (Controller && PointB)
    {
        Controller->MoveToActor(PointB, 5.0f);
    }
}
```

虽然它的工作原理是一样的，但它的表现力要差得多，更难调整和迭代。像这样的脚本事件通常使用 [Sequencer](https://link.zhihu.com/?target=https%3A//docs.unrealengine.com/en-US/AnimatingObjects/Sequencer/Overview/index.html) 来实现，它允许您使用[事件轨道](https://link.zhihu.com/?target=https%3A//docs.unrealengine.com/en-US/AnimatingObjects/Sequencer/Workflow/EventTrackOverview/index.html)来轻松整合关卡蓝图以及 Actor 的蓝图函数

事件图还提供了时间轴，这一直是一种非常方便的制作时间驱动的动画效果的工具。C++ 中也可以完成类似的事情，但是这通常涉及到一些曲线资产，当然你也可以在代码中用方程组实现那些曲线，但是这很蛋疼

## 蓝图优势: 简单好用

蓝图允许你非常快速的进行测试和迭代，整个蓝图开发都是在编辑器环境，还可以直接在编辑器中播放 (PIE)，同时还可以浏览事件图标检查属性值以及对脚本进行调试

相对 C++ 来说，蓝图的用户群体范围更大一些，对于 C++ 小白来说，使用蓝图是一个很好的起点。可能你已经是个 C++ 大佬，但不代表每个人都是。当策划，美术，程序，QA 都可以很安全轻松的用蓝图为项目做贡献的时候，项目肯定会变得更好，众人拾柴火焰高嘛

无论哪种方式，高质量的代码都需要慢慢的积累提升——有人写出漏洞百出的 C++ 代码，也会有人连出设计精妙且易懂的蓝图。但是想对于鬼画符的糟糕蓝图，存在漏洞的 C++ 代码往往对项目更为致命，因为蓝图通常不会导致崩溃

蓝图具有可探索性，所有你能使用的类型和函数都整合在蓝图编辑器中，你几乎可以在无需阅读文档的情况下，就能够探索并了解这些蓝图功能

如果你刚开始使用虚幻引擎，即便你有 C++ 开发经验，先从蓝图入手也是不错的主意，反正你使用蓝图学到的东西都可以直接转移到 C++

那么，C++ 又有哪些优势呢？

## C++ 优势: 性能

最大的优势就是运行时的性能。C++ 代码会在编译时针对目标运行平台进行充分优化。在构建 Shipping 版本时，项目源代码会被编译为机器码，高度原生话且没有多余的开销

尽管[蓝图原生化](https://link.zhihu.com/?target=https%3A//docs.unrealengine.com/en-US/ProgrammingAndScripting/Blueprints/TechnicalGuide/NativizingBlueprints/index.html)在某些情况下会得到大的性能提升，但它却为项目构建过程增加了一些相当不稳定的复杂性

## C++ 优势: 引擎功能

引擎在 C++ 中公开了更多的功能，哪些底层功能往往又非常有用。你可以充分利用日志记录系统，通过输出日志来查看代码功能或者查找问题

```
// Log.h
#include "Logging/LogMacros.h"

// Log.cpp
#include "Log.h"
DEFINE_LOG_CATEGORY(LogCobaltCore);

// DefaultEngine.ini
[Core.Log]
LogCobaltCore=VeryVerbose

// Log a critical error message and halt execution
UE_LOG(LogCobaltCore, Fatal, TEXT("Oh no!"));

// Log an error (red) or warning (yellow), with printf-style formatting
UE_LOG(LogCobaltCore, Error, TEXT("Error: %d"), SomeIntValue);
UE_LOG(LogCobaltCore, Warning, TEXT("Warning: '%s'"), *SomeStringValue);

// Log normal messages which may or many not be shown depending on the verbosity level
UE_LOG(LogCobaltCore, Display, TEXT("Something any developer should see"));
UE_LOG(LogCobaltCore, Log, TEXT("Feedback about routine operation"));
UE_LOG(LogCobaltCore, Verbose, TEXT("Diagnostic info to aid debugging"));
UE_LOG(LogCobaltCore, VeryVerbose, TEXT("Spammy diagnostic info"));
```

你可以添加断言来保持系统稳定性，确保你的代码在预期的情况下执行，并在出现问题时提供有用的错误消息

还可以自定义控制台命令，让你在运行时通过控制台命令在运行时实时控制游戏的行为：

```
// At the top of a .cpp file:
static TAutoConsoleVariable<float> CVarControllerInterpSpeed( TEXT("CobaltCore.Controller.InterpSpeed"),
 8.0f,
 TEXT("Speed for smoothing out controller transforms,\n")
 TEXT(" or 0 to disable interpolation entirely")
);

// Within function bodies in the same .cpp file:
const float InterpSpeed = CVarControllerInterpSpeed.GetValueOnGameThread();

// At runtime, open the console with (~) and run:
// - `CobaltCore.Controller.InterpSpeed` to get the current value
// - `CobaltCore.Controller.InterpSpeed [new-value]` to update the value
```

还可以添加自定义统计类别来捕获详细的分析信息，来衡量游戏中每个系统和功能对性能的影响

还可以更好地控制你的类型和接口的公开方式、C++ 代码库的其他部分、蓝图脚本以及编辑器中的用户：

```
class FSomeClass
{
public:
  // Accessible to all other code

protected:
  // Accessible to subclasses

private:
  // Internal to this class alone
};

// Not exposed to Blueprints at all:
 UCLASS(NotBlueprintType, NotBlueprintable)
// Can be referenced but not extended:
 UCLASS(BlueprintType, NotBlueprintable)
// Can be extended in Blueprints (default for AActor):
 UCLASS(BlueprintType, Blueprintable)

// Read-only to both users and Blueprints:
 UPROPERTY(VisibleAnywhere, BlueprintReadOnly)
// Can't be modified per-instance, but a new default value can be set per-Blueprint:
 UPROPERTY(EditDefaultsOnly, BlueprintReadWrite)
// Can be modified in the Details panel and by Blueprints:
 UPROPERTY(EditAnywhere, BlueprintReadWrite)
```

还可以更精确的控制网络赋值，使用优先级和相关性的自定义规则，你还可以利用复制图表系统等高级功能

还可以在底层建立原始的 TCP 和 UDP 套接字发送和接受数据（虽然 UE 本身就提供了网络），你也可以使用 Http 和 Json 模块与 Web API 进行通讯

还可以自定义序列化规则用来规定数据结构和类型写入到磁盘或者压缩进行网络复制

```
/** Example struct with custom serialization */
USTRUCT(BlueprintType)
struct FBoardCell
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadOnly)
    float Height;

    UPROPERTY(EditAnywhere, BlueprintReadOnly)
    int32 Flags;

    UPROPERTY(EditAnywhere, BlueprintReadOnly)
    int32 PlaneIndex;

    bool Serialize(FArchive &Ar);
};

template <>
struct TStructOpsTypeTraits<FBoardCell> : public TStructOpsTypeTraitsBase2<FBoardCell>
{
    enum
    {
        WithSerializer = true,
    }
};

bool FBoardCell::Serialize(FArchive &Ar)
{
    Ar.UsingCustomVersion(FBoardCustomVersion::GUID);

    if (Ar.IsLoading() || Ar.IsSaving())
    {
        const int32 BoardVer = Ar.CustomVer(FBoardCustomVersion::GUID);
        if (BoardVer < FBoardCustomVersion::SerializeRawCellValues)
        {
            UScriptStruct *Struct = FBoardCell::StaticClass();
            Struct->SerializeTaggedProperties(Ar, (uint8 *)this, Struct, nullptr);
        }
        else
        {
            Ar << Height;
            if (BoardVer < FBoardCustomVersion::StoreCellTransformInPlane)
            {
                FVector_NetQuantize Normal = FVector::ZeroVector;
                Ar << Normal;
            }
            Ar << Flags;
            Ar << PlaneIndex;
        }
    }

    return true;
}
```

可以绑定保存和加载期间发生的底层事件，从而可以在加载时操作数据并提升向后的兼容性。可以添加特定于编辑器或者烘焙过程的代码和数据，而这些代码和数据在非编辑器构建过程中不会参与编译

```
UCLASS()
class ASomeActor : public AActor
{
    GENERATED_BODY()

public:
#if WITH_EDITORONLY_DATA
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
    class UArrowComponent *ArrowComponent;
#endif

public:
    ASomeActor(const FObjectInitializer &ObjectInitializer);
    virtual void OnConstruction(const FTransform &Transform) override;
};

ASomeActor::ASomeActor(const FObjectInitializer &ObjectInitializer)
    : Super(ObjectInitializer)
{
    RootComponent = ObjectInitializer.CreateDefaultSubobject<USceneComponent>(this, TEXT("Root"));

#if WITH_EDITORONLY_DATA
    ArrowComponent = ObjectInitializer.CreateEditorOnlyDefaultSubobject<UArrowComponent>(this, TEXT("Arrow"));
    if (ArrowComponent)
    {
        ArrowComponent->SetupAttachment(RootComponent);
    }
#endif
}

void ASomeActor::OnConstruction(const FTransform &Transform)
{
    Super::OnConstruction(Transform);

#if WITH_EDITORONLY_DATA
    if (ArrowComponent)
    {
        ArrowComponent->SetRelativeTransform(FTransform::Identity);
    }
#endif
}
```

您可以添加编辑器模块，创建自定义界面布局、资产编辑器窗口和导入器以及新的编辑器模式和视口工具[扩展编辑器](https://link.zhihu.com/?target=https%3A//lxjk.github.io/2019/10/01/How-to-Make-Tools-in-U-E.html)。

还可以绑定各种引擎和编辑器委托，从而在不同事件发生时运行自定义代码

## C++ 优势: 链接库

C++ 模块中，无论是作为项目的一部分，还是在插件中，你都可以导入第三方插件库。 如果想将 C 或 C++ 库集成到我们的项目中，你可以针对你支持的平台把它构建成为静态或者共享库，然后更新你的模块 Bulid.cs 文件来把它链接进来，你就可在项目中是该该代码了

```
using System.IO;
using UnrealBuildTool;

public
class MyModule : ModuleRules
{
public
    MyModule(ReadOnlyTargetRules Target) : base(Target)
    {
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
        bEnforceIWYU = true;
        PublicDependencyModuleNames.AddRange(new string[]{"Core", "CoreUObject", "Engine"});

        // Let's say we have a library in MyModule/ThirdParty/somelib:
        // - somelib/include/somelib.h defines library functions
        // - somelib/lib/x64/somelib.lib has been built for our target platform
        // (This example assumes a single supported platform)
        string ModuleThirdPartyDir = Path.Combine(ModuleDirectory, "ThirdParty");
        string LibraryIncludeDir = Path.Combine(ModuleThirdPartyDir, "somelib/include");
        string LibraryStaticLibPath = Path.Combine(ModuleThirdPartyDir, "somelib/lib/x64/somelib.lib");

        // Code in MyModule can now #include "somelib.h" and call functions
        // whose implementations are compiled as part of somelib.lib
        PublicIncludePaths.Add(LibraryIncludeDir);
        PublicAdditionalLibraries.Add(LibraryStaticLibPath);
    }
}
```

这是 C++ 最明显的优势之一，也是最强大的优势。理论上来说，你可以在你的游戏里面插入一个 Windows 系统（手动狗头）

## C++ 优势: 比较和合并

从工作流的角度来看，C++ 与蓝图不同。C++ 代码非常容易进行差异比较以及合并。对于较小的项目，这可能不是一个大问题，但在较大的团队中，这就非常重要了。

向项目的版本控制系统 (svn,git 等) 提交代码之前，一般都会比较修改过的文件，用于查看本次修改的内容。或者查看历史版本来定位 bug。在这些情况下，你希望能够快速查看某个文件在每个提交版本的修改内容。C++ 代码是纯文本，比较差异则是一种基本功能，有很多工具可以逐行列出一个文本文件在两个不同版本之间的差异。合并是纯文本代码的另一个主要优势，两个人可以同时处理同一个源文件，版本控制系统可以自动将它们的修改合并在一起

但蓝图是二进制文件，而且为了查看、编辑它们必须打开项目编辑器；对蓝图进行比较差异或合并非常困难。幸运的是，编辑器包含一个用于比较蓝图的内置工具，针对一些简单情况效果还不错。

![](1671947342604.png)

但要是想针对当前版本已经不存在的旧版本蓝图进行比较差异，那你可能会遇到问题

![](1671947342655.png)

对蓝图进行代码审查在技术上是可行的，但相对检查文本代码修改这个处理过程要麻烦得多

![](1671947342717.png)

蓝图并不是真的可以合并——虽然有一个内置的合并工具，当你需要解决蓝图修改冲突时，这个工具会派上用场，但是任何对于蓝图资产的合并，始终需要人工干预，即使是没有冲突的修改。合并工具相当有限，它基本上只向你显示相关修改，让你选择其中一个接受的版本——除此之外，你还需要手动修复。

所以传统观点是，你应该像对待任何其他资产一样对待蓝图，对于一个蓝图文件，同一时间只能由一个人进行编辑和提交

![](1671947342949.png)

但最终这些都是可以接受的代价，考虑到蓝图的强大和有用之处，我不认为就因为蓝图不好合并就不使用

## 个人偏好

萝卜青菜各有所爱，我们都有个人偏好。但是必须保持一定的自我意识，以确保我们不会因为个人偏好影响判断

![](1671947343010.png)

游戏开发很复杂，需要团队合作。当做出影响整个项目和整个团队的决策时 (例如如何平衡 C++ 和蓝图)，必须权衡许多更重要的因素：

![](1671947343102.png)

*   哪种方式对性能最好？
*   哪种方式最适合项目的整体设计？
*   哪种方式维护性更强？
*   考虑到上线日期和预算限制，用哪种方式能更快速完成工作？

也许你早已有答案， “我不喜欢记 C++ 语法” 或者 “我不喜欢连连看” 都属于个人偏好。如果基于上面的问题两中方式都可以，或者你只是一个游戏开发爱好者，那么你就尽管按照自己喜欢的方式来吧，毕竟乐于其中往往效果更好。我个人觉得 C++ 和蓝图都应该去尝试一下，如果你花些时间了解他们各自的优势和劣势所在，我想你会发现它们都很有趣

## 总结

无论你当前负责项目哪个模块，我都鼓励你尝试 C++ 和蓝图组合使用；感谢阅读，我希望你学到了一些新东西