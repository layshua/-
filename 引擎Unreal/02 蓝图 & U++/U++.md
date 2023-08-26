1.  [大钊老师啊：虚幻C++进阶之路](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1C7411F7RF)
2.  [梁迪_梁迪腾讯课堂官网](https://link.zhihu.com/?target=http%3A//didi.ke.qq.com/%23tab%3D1%26category%3D-1) （迪迪老师 教你 C 草实战）
3.  [技术宅阿棍儿](https://link.zhihu.com/?target=https%3A//space.bilibili.com/92060300/video)（大标题直击要领，很棒）
4.  [来自程序员的暴击：虚幻四C++入坑指南合集版](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV14K411J7v2)

# 反射系统
**虚幻引擎反射系统** 使用**宏**为提供引擎和编辑器各种功能，封装你的类。在使用 **虚幻引擎（UE） 时，可以使用标准的 C++类、函数和变量。

- 虚幻中对象的基类是 [UObject](https://docs.unrealengine.com/5.2/zh-CN/objects-in-unreal-engine)。每个类都新定义了一个用于[Actor](https://docs.unrealengine.com/5.2/zh-CN/actors-in-unreal-engine)或对象（Object）的模板。
- 你可以**使用 `UCLASS` 宏来标记从 `Uobject` 派生的类**，以便[UObject处理系统](https://docs.unrealengine.com/5.2/zh-CN/unreal-object-handling-in-unreal-engine)可以注意到这些类。
- [TSubclassOf](https://docs.unrealengine.com/5.2/zh-CN/typed-object-pointer-properties-in-unreal-engine)是模板类，提供 `Uclass` 类型保险。 它在分配从特定类型派生出来的类时很有效。例如，你可以把这个变量公开给蓝图，设计者可以为玩家角色指定生成的武器类别。
- 类可以包含[结构体](https://docs.unrealengine.com/5.2/zh-CN/structs-in-unreal-engine)。结构体是帮助组织和操控其相关相关属性的数据结构。**结构体可以使用 `USTRUCT()` 宏来单独定义。**
- [虚幻智能指针库](https://docs.unrealengine.com/5.2/zh-CN/smart-pointers-in-unreal-engine)为C++11智能指针的自定义实现，旨在减轻内存分配和追踪的负担。该实现包括行业标准[共享指针](https://docs.unrealengine.com/5.2/zh-CN/shared-pointers-in-unreal-engine)，[弱指针](https://docs.unrealengine.com/5.2/zh-CN/weak-pointers-in-unreal-engine)，**唯一指针（Unique Pointers）**，和[共享引用](https://docs.unrealengine.com/5.2/zh-CN/shared-references-in-unreal-engine)，此类引用的行为与不可为空的共享指针相同。
- [接口](https://docs.unrealengine.com/5.2/zh-CN/interfaces-in-unreal-engine) 提供可以在多个或不同的类中实现函数和额外的游戏行为。 你的玩家角色可以与世界中的各种Actor互动。 每个这些互动都能引起对一个事件的不同反应。
- [Metadata说明符](https://docs.unrealengine.com/5.2/zh-CN/metadata-specifiers-in-unreal-engine)控制类、接口、结构体、列举、函数，或属性与引擎和编辑器各方面的交互方式。每一种类型的数据结构或成员都有自己的元数据说明符列表。
- [UFUNCTION](https://docs.unrealengine.com/5.2/zh-CN/ufunctions-in-unreal-engine)，以及 [UPROPERTY](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-uproperties) 宏使 UE 注意到新的类、函数和变量。这些宏由引擎进行垃圾收集。在说明宏时, 你可以在虚幻编辑器中编辑和显示它们。
# 接口
接口类用于确保一组可能不相关的类实现一组公共的函数。在一些游戏功能可能由原本不相似的大型复杂类共享的情况下，这很有用。

游戏可能有这样一个系统，玩家角色进入触发器体积时可以激活陷阱、提醒敌人或向玩家奖励积分。这可以通过陷阱、敌人或积分奖励上的"ReactToTrigger"函数来实现。但是，陷阱可能派生自[AActor](https://docs.unrealengine.com/5.2/zh-CN/actors-in-unreal-engine)，敌人派生自专门的[APawn](https://docs.unrealengine.com/5.2/zh-CN/pawn-in-unreal-engine)或[ACharacter](https://docs.unrealengine.com/5.2/zh-CN/characters-in-unreal-engine)子类，而积分奖励派生自 `UDataAsset` 。

所有这些类都需要共享的功能，但除了 `UObject` 之外，没有其他公共的父类。在这种情况下，推荐使用接口。

## 接口声明

声明接口类与声明普通的虚幻类相似，但仍有两个主要区别。

1. 首先，接口类使用 `UINTERFACE` 宏而不是 `UCLASS` 宏，且直接从 `UInterface` 而不是 `UObject` 继承。 

```c++
UINTERFACE([specifier, specifier, ...], [meta(key=value, key=value, ...)])
class UClassName : public UInterface
{
    GENERATED_BODY()
};
```

2. 其次，`UINTERFACE` 类不是实际的接口；它是一个空白类，它的存在只是为了向虚幻引擎反射系统确保可见性。将由其他类继承的实际接口必须具有相同的类名，但是**开头字母`U`必须改为`I`。**

在你的.h文件（例如 `ReactToTriggerInterface.h`）中：

ReactToTriggerInterface.h

```c++
#pragma once

#include "ReactToTriggerInterface.generated.h"

//确保反射系统可见性
UINTERFACE(MinimalAPI, Blueprintable)
class UReactToTriggerInterface : public UInterface
{
    GENERATED_BODY()
};

//实际接口
class IReactToTriggerInterface
{
    GENERATED_BODY()

public:
    /** 在此处添加接口函数声明 */
};
```

- 前缀为 `U`的类不需要构造函数或任何其他函数
- 前缀为 `I`的类将包含所有接口函数，且此类实际上将被你的其他类继承。

如果你想要让蓝图实现此接口，则需要 `Blueprintable` 说明符。

### 接口说明符

接口说明符用于向虚幻反射系统公开你的类，详见下表：

|接口说明符|含义|
|---|---|
|`BlueprintType`|将该类公开为可用于蓝图中的变量的类型。|
|`DependsOn=(ClassName1, ClassName2, ...)`|所有列出的类都将在该类之前编译。ClassName必须在同一个（或上一个）包中指定一个类。多个依赖性类可以使用以逗号分隔的单个"DependsOn"行来指定，也可以使用单个"DependsOn"行为每个类指定。当一个类使用在另一个类中声明的结构体或枚举时，这一点非常重要，因为编译器只知道它已经编译的类中的内容。|
|`MinimalAPI`|仅导致该类的类型信息被导出以供其他模块使用。你可以向该类转换，但不能调用该类的函数（内联方法除外）。对于不需要其所有函数在其他模块中均可供访问的类，通过不导出这些类的所有内容，这可以缩短编译时间。|

## 在C++中实现接口

若要在一个新的类中使用你的接口，只需从"前缀为I（I-prefixed）"的接口类继承（除了你正在使用的任何基于"UObject"的类）即可。

Trap.h

```
    #include "CoreMinimal.h"
    #include "GameFramework/Actor.h"
    #include "ReactToTriggerInterface.h"
    #include "Trap.generated.h"

    UCLASS(Blueprintable, Category="MyGame")
    class ATrap : public AActor, public IReactToTriggerInterface
    {
        GENERATED_BODY()

    public:
        /** 在此处添加接口函数重载。 */
    }
```

## 声明接口函数

有几种方法可以在接口中声明函数，由环境决定能够实现或调用哪种方法。所有方法都必须在"前缀为I（I-prefixed）"的类中为接口声明，而且必须为 `public` ，以便对外部的类可见。

### 仅C++的接口函数

可以在接口的头文件中声明一个不带 `UFUNCTION` 说明的虚拟C++函数。这些函数必须为虚拟的，以便在实现接口的类中覆盖它们。

ReactToTrigger.h

```
    public:
    virtual bool ReactToTrigger();
```

然后，可以在头文件本身或接口的 `.cpp` 文件中提供一个默认的实现。

ReactToTrigger.cpp

```
    bool IReactToTriggerInterface::ReactToTrigger()
    {
        return false;
    }
```

当在一个Actor类中实现接口后，可以创建并实现一个针对该类的覆盖。

Trap.h

```
    public:
    virtual bool ReactToTrigger() override;
```

Trap.cpp

```
    bool ATrap::ReactToTrigger()
    {
        return false;
    }
```

但是，这些C++接口函数对蓝图不可见。

### 蓝图可调用接口函数

要创建蓝图可调用的接口函数，必须在带 `BlueprintCallable` 说明符的函数声明中提供一个 `UFUNCTION` 宏。还必须使用 `BlueprintImplementableEvent` 或 `BlueprintNativeEvent` 说明，而且函数不能为虚拟的。

ReactToTrigger.h

```
    public:
    /**只能在蓝图中实现的React To Trigger版本。*/
    UFUNCTION(BlueprintCallable, BlueprintImplementableEvent, Category=Trigger Reaction)
    bool ReactToTrigger();
```

ReactToTrigger.h

```
    public:
    /**可以在C++或蓝图中实现的React To Trigger版本。*/
    UFUNCTION(BlueprintCallable, BlueprintNativeEvent, Category=Trigger Reaction)
    bool ReactToTrigger();
```

#### BlueprintCallable

引用实现接口的对象的C++或蓝图可以调用使用 `BlueprintCallable` 说明符的函数。

#### BlueprintImplementableEvent

使用 `BlueprintImplementableEvent` 的函数不能在C++中被覆盖，但可以在任何实现或继承接口的蓝图类中被覆盖。

#### BlueprintNativeEvent

在C++中，可通过覆盖一个同名函数来实现使用 `BlueprintNativeEvent` 的函数，但要在末尾添加上后缀 `_Implementation` 。

Trap.h

```
    public:
    bool ReactToTrigger_Implementation() override;
```

Trap.cpp

```
    bool ATrap::ReactToTrigger_Implementation() const
    {
        return false;
    }
```

该说明符还允许在蓝图中覆盖实现。

## 确定类是否实现了接口

为了与实现接口的C++和蓝图类兼容，可以使用以下任意函数：

```
    bool bIsImplemented = OriginalObject->GetClass()->ImplementsInterface(UReactToTriggerInterface::StaticClass()); // 如果OriginalObject实现了UReactToTriggerInterface，则bisimplemated将为true。

    bIsImplemented = OriginalObject->Implements<UReactToTriggerInterface>(); // 如果OriginalObject实现了UReactToTrigger，bIsImplemented将为true。

    IReactToTriggerInterface* ReactingObjectA = Cast<IReactToTriggerInterface>(OriginalObject); // 如果OriginalObject实现了UReactToTriggerInterface，则ReactingObject将为非空。
```

如果 `StaticClass` 函数在"前缀为I（I-prefixed）"的类中没有实现，尝试在"前缀为U（U-prefixed）"的类上使用 `Cast` 将失败，代码将无法编译。

## 转换到其他虚幻类型

虚幻引擎的转换系统支持从一个接口转换到另一个接口，或者在适当的情况下，从一个接口转换到一个虚幻类型。

```
    IReactToTriggerInterface* ReactingObject = Cast<IReactToTriggerInterface>(OriginalObject); // 如果接口被实现，则ReactingObject将为非空。

    ISomeOtherInterface* DifferentInterface = Cast<ISomeOtherInterface>(ReactingObject); // 如果ReactingObject为非空而且还实现了ISomeOtherInterface，则DifferentInterface将为非空。

    AActor* Actor = Cast<AActor>(ReactingObject); // 如果ReactingObject为非空且OriginalObject为AActor或AActor派生的类，则Actor将为非空。
```

## 蓝图可实现类

如果你想要蓝图能够实现此接口，则必须使用 `Blueprintable` 元数据说明符。蓝图类要覆盖的每个接口函数都必须是`BlueprintNativeEvent `或"BlueprintImplementableEvent"。标记为` BlueprintImplementableEvent` 的函数仍然可以被调用，但不能被覆盖。你将无法从蓝图访问所有其他函数。