## **UE4 委托 (代理） 概览**

## **一、简介 & 理解**

委托也叫做代理，其作用就是提供一种消息机制，都知道消息的传递需要发送方和接收方，而委托的过程也可分为这两大部分，我们可以换个名字分别叫做：发送者和观察者，这就是委托的主要部分，记住这个两个点就能记住委托原理。其实委托的方法就是和软件模式中的 `观察者模式` 是同一原理。

### **步骤**

**发送者步骤：**
1. 声明委托，定义参数列表【可以简单理解为构建一个委托类】
2. 声明委托对象【理解为实例化一个类】
3. 在需要的位置触发委托（执行或者广播），需要传入参数【相当于发送消息】

**观察者步骤：**
1. 在其他类中需要接收消息位置，绑定实际执行函数【接收消息，并且根据消息内容执行绑定的操作】
2. 实现具体的执行函数内容【被触发后，实际执行的内容】

---

**例子**：为角色添加一个血条 component，通过血条 component 接收血条变化并将其广播给角色本身，让角色接收消息做出一定反应，比如扣血和死亡
**发送者**：血条组件
**观察者**：角色

1. 先在头文件上面声明一个委托, 名字要以 F 开头（位置：HealthComponent.h)
```c++ file:HealthComponent.h
//声明一个动态广播委托
//第一个参数事件的名称（自己定义）
//后面的参数是事件所需要的参数
//因为有6个参数 所以添加_SixParams
DECLARE_DYNAMIC_MULTICAST_DELEGATE_SixParams(FOnHealthChangeSignature, USHeathComponent*, HealthComponent, float, Health, float, HealthDelta, const class UDamageType*, DamageType, class AController*, InstigatedBy, AActor*, DamageCauser);
```

2. 声明一个委托实例（位置：HealthComponent.h)
```c++ file:HealthComponent.h
public:
//创建事件实例
//属性暴露给蓝图
UPROPERTY(BlueprintAssignable, Category = "Events")
FOnHealthChangeSignature OnHealthChange;
```

3. 在需要的位置触发广播（位置：HealthComponent.cpp)
```c++ file:HealthComponent.cpp
//事件广播
OnHealthChange.Broadcast(this,CurrentHealth,Damage,DamageType,InstigatedBy, DamageCauser);
```

4. 最后要在蓝图类中，选中声明的委托 C++ 类组件，找到委托事件进行绑定（位置：Character 蓝图)
![[c093a0ded68ab870163bd68435ac5ce8_MD5.jpg]]

或者在 c++ 进行绑定（位置：Character. h/. cpp)
```c++ file:Character.h
//添加HealthComponent组件，Character相当于一个容器
UPROPERTY(VisibleAnywhere,BlueprintReadOnly,Category="Health")
USHeathComponent* HealthComponent;
​
//要绑定的函数参数要和声明委托的参数一致
 UFUNCTION()
 void OnHealthChange(USHeathComponent* HealthComponent, float Health, float HealthDelta, const class UDamageType* DamageType, class AController* InstigatedBy, AActor* DamageCauser);
​
```

```c++ file:Character.cpp
//在创建HealthComponent对象后，绑定函数OnHealthChange函数到其委托OnHealthChange上（名字可以不相同），OnHealthChange函数执行具体的内容
HealthComponent->OnHealthChange.AddDynamic(this, &ASCharacter::OnHealthChange);
```

解释下，这样使用**委托的好处**：血条组件中使用`OnHealthChange`委托，那么 Character 和其他 Actor 使用血条组件时，只有**单方面的引用**（观察者引用消息发送者）（观察者设计模式原理），血条组件并不需要关心谁用了它，也就是血条组件无需包含使用它的 Character 或者 Actor 的引用，这就实现了一种解耦，这就是委托的好处。

### **委托与虚函数**

参考：

[UE4 的委托 - redeyerabbit - 博客园](https://www.cnblogs.com/redeyerabbit/p/6564387.html)

对 Event 的触发主要有虚函数和委托，虚函数只能在派生类中使用，而委托可以在其他类和蓝图中调用

我们以常见的`overlap`为例，下面虚函数的方法

```c++
//base
virtual void NotifyActorBeginOverlap(AActor* OtherActor);
​
//derived
virtual void NotifyActorBeginOverlap(AActor* OtherActor) override;
```

在重叠事件发生时，会触发该函数，但是该`overlap`函数只能在派生类中进行重载。而委托 / 委托则没有限制，定义好后的委托可以在不同类中进行调用。（注意：委托在`观察者`主要是通过声明和定义委托的那个类的`对象`来获取）

```c++
//UE4中声明的overlap的委托
//FActorBeginOverlapSignature
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams( FActorBeginOverlapSignature, AActor*, OverlappedActor, AActor*, OtherActor );
```

综上，**我们可以根据实际情况来灵活使用虚函数或者委托来触发 Event, 虚幻 4 中也为开发者提供了这两种方案。**

## **二、委托类型**

UE4 提供了五类 Delegate，分别是单播、多播、动态委托、动态多播委托、事件

### **1. 单播委托**

**简介：**单播委托只能绑定一个函数指针的委托，单播执行时只能触发一个绑定函数（如果绑定多个，只能触发最后绑定的那个函数）

**其他特性：**

![[0e61248b90f580a59888582eb7758374_MD5.png]]

### **1）定义和声明委托**

声明定义单播委托可以有参数或无参数，有返回值或无返回值，通过如下几种宏声明

```c++
DECLARE_DELEGATE( FSimpleDelegate ); // 无参、无返回值
DECLARE_DELEGATE_OneParam(FPakEncryptionKeyDelegate, uint8[32]); // 1个参数、无返回值
DECLARE_DELEGATE_TwoParams(FPakSigningKeysDelegate, TArray<uint8>&, TArray<uint8>&); // 2个参数、无返回值
DECLARE_DELEGATE_RetVal_ThreeParams(bool, FOnMountPak, const FString&, int32, IPlatformFile::FDirectoryVisitor*); // 3个参数、bool返回值
```

定义了委托后，只是相当于编程声明了一个类，还要对其进行 "实例化"，即声明委托对象，以下是完整流程

```c++
//函数开头声明委托
DECLARE_DELEGATE(FTestDelegate);
class XXX_API AMyTestActor : public AActor
{
    GENERATED_BODY()
public: 
    AExecuteDelegateActor();
protected:
    virtual void BeginPlay() override;
public: 
    virtual void Tick(float DeltaTime) override;
public:
    //"实例化"单播委托，此处无法返回给蓝图，因此不需要UFUNCTION
    FTestDelegate TestDelegate;
}
```

### **2）绑定函数到委托**

单播委托支持 8 中绑定方法和 1 种解绑方法

![[f57337e4528c8e7c018b67c1ad5fa8a8_MD5.jpg]]

实际案例，仅作部分参考

```c++
// Bind Static
CharacterDelegate1.BindStatic(StaticDelegateProc);
CharacterDelegate2.BindStatic(ATPSProjectCharacter::StaticCharacterDelegateProc);
​
// Bind Raw
DelegateCppTestClass Obj1;
CharacterDelegate3.BindRaw(&Obj1, &DelegateCppTestClass::CppDelegateProc);
​
// Bind Lambda
auto LambdaDelegateProc = [](int nCode)->void
{
    UE_LOG(LogTemp, Log, TEXT("LambdaDelegateProc : %d"), nCode);
};
CharacterDelegate4.BindLambda(LambdaDelegateProc);
​
CharacterDelegate5.BindLambda(
    [](int nCode)->void
    {
        UE_LOG(LogTemp, Log, TEXT("LambdaDelegateProc2 : %d"), nCode);
    }
);
​
// Bind Weak Lambda
auto WeakLambdaDelegateProc = [](int nCode)->void
{
    UE_LOG(LogTemp, Log, TEXT("WeakLambdaDelegateProc : %d"), nCode);
};
UDelegatepTestClass* UObj1 = NewObject<UDelegatepTestClass>(this, UDelegatepTestClass::StaticClass());
CharacterDelegate6.BindWeakLambda(UObj1, WeakLambdaDelegateProc);
​
UDelegatepTestClass* UObj2 = NewObject<UDelegatepTestClass>(this, UDelegatepTestClass::StaticClass());
CharacterDelegate7.BindWeakLambda(
    UObj2, [](int nCode)->void
    {
        UE_LOG(LogTemp, Log, TEXT("WeakLambdaDelegateProc2 : %d"), nCode);
    }
);
​
// Bind SP(Shared Pointer)
TSharedRef<DelegateCppTestClass> ObjSP1 = MakeShareable(new DelegateCppTestClass());
CharacterDelegate8.BindSP(ObjSP1, &DelegateCppTestClass::CppDelegateProc2);
​
TSharedRef<DelegateCppTestClass> ObjSP2 = MakeShared<DelegateCppTestClass>();
CharacterDelegate9.BindSP(ObjSP2, &DelegateCppTestClass::CppDelegateProc3);
​
// Bind Thread Safe SP(Shared Pointer)
TSharedRef<DelegateCppTestClass, ESPMode::ThreadSafe> ObjSafeSP1 = MakeShareable(new DelegateCppTestClass());
CharacterDelegate10.BindThreadSafeSP(ObjSafeSP1, &DelegateCppTestClass::CppDelegateProc4);
​
TSharedRef<DelegateCppTestClass, ESPMode::ThreadSafe> ObjSafeSP2 = MakeShared<DelegateCppTestClass, ESPMode::ThreadSafe>();
CharacterDelegate11.BindThreadSafeSP(ObjSafeSP2, &DelegateCppTestClass::CppDelegateProc5);
​
// Bind UObject
UDelegatepTestClass* UObj3 = NewObject<UDelegatepTestClass>(this, UDelegatepTestClass::StaticClass());
CharacterDelegate12.BindUObject(UObj3, &UDelegatepTestClass::DelegateProc1);
​
// Bind UFunction
UDelegatepTestClass* UObj4 = NewObject<UDelegatepTestClass>(this, UDelegatepTestClass::StaticClass());
CharacterDelegate13.BindUFunction(UObj4, STATIC_FUNCTION_FNAME(TEXT("UDelegatepTestClass::DelegateUFunctionProc1")));
```

### **3）触发或执行绑定函数（调用）**

单播委托调用有三个方法，但是注意：对于单播来说有返回值或者无返回值的情况

![[774fd80f8f0b10e913ddaf18c74a42c2_MD5.png]]

实际案例：

下面案例中，声明和调用都在一个类中完成只为简明过程，一般在不同类中实现

```
DECLARE_DELEGATE_OneParam(FTestDelegate, int);
​
static void StaticDelegateProc(int nCode) {
    UE_LOG(LogTemp, Log, TEXT("StaticDelegateProc : %d"), nCode);
}
​
FTestDelegate DelegateObj1;
DelegateObj1.BindStatic(StaticDelegateProc);
​
//通过ExcuteIfBound方法绑定
DelegateObj1.ExecuteIfBound(1);
```

### **2. 多播委托**

### **1）定义**

```
//多播委托
DECLARE_MULTICAST_DELEGATE(DelegateName);
DECLARE_MULTICAST_DELEGATE_ONEPARAM(DelegateName, Param1Type);
DECLARE_MULTICAST_DELEGATE_XXXPARAMS(DelegateName, Param1Type,...);
```

### **2）绑定函数**

![[75eda287dbc14d7959cf572ce4cf3928_MD5.jpg]]

实际案例，仅作部分参考

### **3）多播触发**

多播委托不可能有返回值

![[69406e62cfc3d657d92d7040e0922942_MD5.png]]

### **4）案例**

```c++
//定义
DECLARE_MULTICAST_DELEGATE_OneParam(FCharacterDelegate_Multicast, int);  // TMulticastDelegate<void, int>
​
//声明委托对象
FCharacterDelegate_Multicast CharacterDelegateMulticast1;
​
//绑定函数  Add Static
bool b1 = CharacterDelegateMulticast1.IsBound(); // 查看是否绑定实例，此时为false
FDelegateHandle HandleMC1 = CharacterDelegateMulticast1.AddStatic(StaticDelegateProc); // 绑定实例-案例1
CharacterDelegateMulticast1.AddStatic(ATPSProjectCharacter::StaticCharacterDelegateProc); // 绑定实例-案例2
​
//广播
CharacterDelegateMulticast1.Broadcast(100); // 执行绑定实例，执行顺序可能与绑定顺序不同
```

### **3. 事件**

事件实质上是一个`多播委托`，功能相差无几, 下图展示了事件和多播的区别



### **1）事件定义和声明**

![[6c4b45888f0384acb6f2e9984d506ab8_MD5.jpg]]

事件有有参和无参两种情况

![[88b76c92fa0839cfb6d7590bf9b24b50_MD5.png]]

**注意 1**：事件第一个参数为'拥有'此事件的类型 B，即类型 B 为事件的友元类，可以访问事件中的私有成员，例如 broadcast 函数

**注意 2：**定时事件宏一般放在一个类的内部，即访问该类型时需要带上所在类的名称前缀，如`（ATPSProjectCharacter::FCharacterEvent）`, 可以有限减少事件类型名称冲突

```
//定义
DECLARE_EVENT(UWorld, FOnTickFlushEvent);  // 无参
DECLARE_EVENT_OneParam(ATPSProjectCharacter, FCharacterEvent, int);  // 一个参数TBaseMulticastDelegate<void, int>
​
//声明
FCharacterEvent CharacterEvent;
```

### **2）绑定和触发**

**绑定**

绑定方式和多播相同，参考多播绑定

**触发**

![[9d121b38d3d9fb0d0f010012a3fe4af9_MD5.png]]

### **4. 动态委托和动态多播委托**

动态委托相比于标准委托，动态委托可序列化，让其可以在**蓝图中**进行调用，其函数可按命名查找，但其执行速度比常规委托慢

注意：**动态委托**可以作为函数参数使用，只有**动态多播委托**可以作为变量给蓝图绑定和调用

### **1）定义和声明动态委托**

```c++
//定义
DECLARE_DYNAMIC_DELEGATE(DelegateName); //动态单播委托
DECLARE_DYNAMIC_DELEGATE_XXXPARAMS(DelegateName, Param1Type,...);
DECLARE_DYNAMIC_DELEGATE_RetVal(EMouseCursor::Type, FGetMouseCursor); 
DECLARE_DYNAMIC_DELEGATE_RetVal_TwoParams(FEventReply, FOnPointerEvent, FGeometry, MyGeometry, const FPointerEvent&, MouseEvent); 
​
DECLARE_DYNAMIC_MULTICAST_DELEGATE(DelegateName); //动态多播委托
DECLARE_DYNAMIC_MULTICAST_DELEGATE_ONEPARAM(DelegateName, Param1Type, Param1Name);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_XXXPARAM(DelegateName, Param1Type, Param1Name,...);
​
​
​
//声明
FCharacterDelegate_Dynamic DelegateName; //动态单播委托
​
UPROPERTY(BlueprintAssignable, Category = "Test") //动态多播委托，使用BlueprintAssignable可以让其在蓝图中被使用
FCharacterDelegate_DynamicMulticast DelegateName;
```

### **2）动态委托 (委托) 绑定**

在 ue4 中封装了较为简单的宏，通过宏拆解出函数名，后面可以通过存入的函数名来反射找到对应类对应的函数

```c++
#define BindDynamic( UserObject, FuncName ) __Internal_BindDynamic( UserObject, FuncName, STATIC_FUNCTION_FNAME( TEXT( #FuncName ) ) )
 
#define AddDynamic( UserObject, FuncName ) __Internal_AddDynamic( UserObject, FuncName, STATIC_FUNCTION_FNAME( TEXT( #FuncName ) ) )
```

因此对于动态委托可以使用以下的宏

*   **动态委托绑定**

![[664cd2c7ad814fd391b9871ec1cd9832_MD5.png]]

*   **动态多播委托绑定**

![[1da3f18b9bfbb3a03fe2a23b68e3bed0_MD5.png]]

### **3）执行动态委托**

*   **执行动态委托**

![[510fa98e9d83851287a372aa42f73b3f_MD5.png]]

*   **执行动态多播委托**

![[abd8cd819345a9b683c78545e6b142ee_MD5.png]]

### **4）具体实例**

*   **动态单播委托**

以`FCharacterDelegate_Dynamic`为例，讲述动态委托的绑定和触发

```c++
//定义
DECLARE_DYNAMIC_DELEGATE_OneParam(FCharacterDelegate_Dynamic, int, nCode);
​
//声明
FCharacterDelegate_Dynamic CharacterDelegateDynamic;
​
//绑定和触发
void ATPSProjectCharacter::OnDelegateDynamicTest()
{
    //使用BindUFunction宏
    bool bd1 = CharacterDelegateDynamic.IsBound(); // false
​
    UDelegatepTestClass* UObjDy1 = NewObject<UDelegatepTestClass>(this, UDelegatepTestClass::StaticClass());
    CharacterDelegateDynamic.BindUFunction(UObjDy1, STATIC_FUNCTION_FNAME(TEXT("UDelegatepTestClass::DelegateUFunctionProc1")));// 绑定实例个数为：1
​
    bool bd2 = CharacterDelegateDynamic.IsBound(); // true
 
    CharacterDelegateDynamic.ExecuteIfBound(200);  //执行，发送数字200
​
    CharacterDelegateDynamic.Unbind();
​
    // 使用BindDynamic宏
    CharacterDelegateDynamic2.BindDynamic(this, &ATPSProjectCharacter::DynamicMulticastProc);
    if (CharacterDelegateDynamic2.IsBound()) // true
    {
        CharacterDelegateDynamic2.Execute(201); //执行，发送数字201
    }    
​
    CharacterDelegateDynamic2.Clear(); // 功能与Unbind一样，内部是直接调用Unbind方法
}
```

*   **动态多播委托**

以`FCharacterDelegate_DynamicMulticast`为例，讲述动态多播委托的绑定和触发

```c++
//定义
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FCharacterDelegate_DynamicMulticast, int, nCode);
​
//声明
FCharacterDelegate_DynamicMulticast CharacterDelegateDynamicMulticast;
​
//绑定和触发
void ATPSProjectCharacter::OnDelegateDynamicMulticastTest()
{
    UDelegatepTestClass* UObjDMC1 = NewObject<UDelegatepTestClass>(this, UDelegatepTestClass::StaticClass());
    
    // 使用AddDynamic宏
    CharacterDelegateDynamicMulticast.AddDynamic(UObjDMC1, &UDelegatepTestClass::DelegateUFunctionProc1); // 绑定实例个数为：1
​
    // 使用AddUniqueDynamic宏
    CharacterDelegateDynamicMulticast.AddUniqueDynamic(this, &ATPSProjectCharacter::DynamicMulticastProc); // 绑定实例个数为：2
​
 
    FScriptDelegate delegateVar1; // FScriptDelegate即为TScriptDelegate<>
    delegateVar1.BindUFunction(this, STATIC_FUNCTION_FNAME(TEXT("ATPSProjectCharacter::DynamicMulticastProc3")));
    CharacterDelegateDynamicMulticast.Add(delegateVar1); // 绑定实例个数为：3
    
    CharacterDelegateDynamicMulticast.Broadcast(300); // 执行绑定实例列表（共1个）  注：执行顺序可能与函数的添加顺序不相同
    CharacterDelegateDynamicMulticast.Clear(); // 清除所有绑定实例
​
}
```

### **5）蓝图中实现**

以下案例来自博客：[https://blog.csdn.net/yangxuan0261/article/details/52097699](https://blog.csdn.net/yangxuan0261/article/details/52097699)

以下为动态多播委托在蓝图中进行绑定和触发相关的操作

1. 首先在 C++ 中定义对象及动态多播委托

```c++
//在蓝图中只能使用动态多播委托
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMyDelegate2, int32, abc);
​
UCLASS()
class AMyChar : public ACharacter
{
    GENERATED_BODY()
public:
    //声明为BlueprintAssignable，只给蓝图绑定这个委托
    UPROPERTY(BlueprintAssignable, Category = "MyChar")
        FMyDelegate2 OnMyDelegate2;
};
```

2. 在蓝图中绑定函数

`MyCharBp`为继承自`AMyChar`的蓝图对象

绑定函数`charDelegate`到 OnMyDelegate2 委托上

![[354d2ce95f2a0925b4429a0469c24a06_MD5.jpg]]

3. 广播触发函数

![[294c043475d30f513dc7b76c65cef267_MD5.jpg]]

4. 结果

![[54738c567b164ca27a2cbfbdac11d289_MD5.jpg]]

## **四、参考文章**

本文有些案例和说法借用及参考了以下博客链接

[https://www.cnblogs.com/kekec/p/10678905.html](https://www.cnblogs.com/kekec/p/10678905.html)  
[https://zhuanlan.zhihu.com/p/261717182](https://zhuanlan.zhihu.com/p/261717182)  
[https://blog.csdn.net/github_38111866/article/details/104668701](https://blog.csdn.net/github_38111866/article/details/104668701)  
[https://jerish.blog.csdn.net/article/details/78527142?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&dist_request_id=45117f50-7ad7-4d5c-bed9-2b7fa5503df6&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control](https://jerish.blog.csdn.net/article/details/78527142?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control&dist_request_id=45117f50-7ad7-4d5c-bed9-2b7fa5503df6&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.control)  
[https://blog.csdn.net/yangxuan0261/article/details/52097699](https://blog.csdn.net/yangxuan0261/article/details/52097699)  
[https://docs.unrealengine.com/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Dynamic/index.html](https://docs.unrealengine.com/zh-CN/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Dynamic/index.html)