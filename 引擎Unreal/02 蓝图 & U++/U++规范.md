
## 类组织

在组织 **类** 时，应该从阅读者而不是编写者的角度考虑。由于大部分阅读者会使用类的public接口，public实现应该首先声明，然后是类的private实现。

```c++
UCLASS()
class EXAMPLEPROJECT_API AExampleActor : public AActor
{
    GENERATED_BODY()

public: 
    // 为此Actor的属性设置默认值
    AExampleActor();

protected:

    // 当游戏开始或重生（Spawn）时调用
    virtual void BeginPlay() override;

};
```

## 版权声明

凡是Epic为公开发布而提供的源文件（ `.h` 、 `.cpp` 、 `.xaml` ），都必须在文件的第一行包含版权声明。声明必须严格采用如下所示的格式：

```
    // 版权所有Epic Games, Inc.保留所有权利。
```

如果该行缺失或格式不恰当，CIS会生成错误并失败。

## 命名规范

使用命名规范时，所有代码和注释都应该使用美式英语拼写和语法。

- 名称（例如类型名称或变量名称）中每个单词的第一个字母要大写。单词之间通常不用下划线。例如， `Health` 和 `UPrimitiveComponent` 是正确写法，但 `lastMouseCoordinates` 和 `delta_coordinates` 不正确。
    

有些用户可能熟悉的是其他面向对象的编程语言，请注意，这是帕斯卡命名法（PascalCase）格式

- **类型名称以额外的大写字母作为前缀，以与变量名称区分开**。例如， `FSkin` 是类型名称，而 `Skin` 是类型 `FSkin` 的实例。

- **模板类带有前缀T。**
```
    class TAttribute
```

- **继承自 [UObject](https://docs.unrealengine.com/5.2/zh-CN/objects-in-unreal-engine) 的类带有前缀 `U`。**
```
    class UActorComponent
```

- **继承自 [AActor](https://docs.unrealengine.com/5.2/zh-CN/actors-in-unreal-engine) 的类带有前缀 `A`。**
```
    class AActor
```

- **继承自[SWidget](https://docs.unrealengine.com/5.2/zh-CN/slate-user-interface-programming-framework-for-unreal-engine)的类带有前缀`S`。**
```
    class SCompoundWidget
```

- **作为抽象接口的类带有前缀 `I`。**
```
    class IAnalyticsProvider
```

- **Epic的类似概念的类类型带有前缀`C`。**
```
    template <typename Concept, typename... Ts>
```

- **枚举带有前缀`E`。**
```
    enum class EColorBits
    {
       ECB_Red,
       ECB_Green,
       ECB_Blue
    };
```

- **布尔变量必须带有前缀`b`。**
```
    bPendingDestruction

    bHasFadedIn.
```

- **其他大部分类带有前缀`F`，不过一些子系统会使用其他字母。**
    
- Typedef应该带有适合该类型的任何前缀，例如：
    - F用于结构体的typedef
    - U用于 `UObject` 的typedef
- 特定模板实例化的typedef不再是模板，应该相应带有前缀。

```
typedef TArray<FMytype> FArrayOfMyTypes;
```

- C#中会省略前缀。
- 虚幻标头工具在大部分情况下都需要正确的前缀，因此请务必提供正确的前缀。
- 类型模板参数和基于这些模板参数的嵌套类型别名不受以上前缀规则制约，因为类型类别未知。
- 在说明性术语之后使用类型后缀会更好。
- 使用`In`前缀，消除模板参数与别名的歧义：
```
template <typename InElementType>
            class TContainer
            {
            public:
                using ElementType = InElementType;
            };
```

- 类型和变量名称是名词。
- 方法名称是说明方法效果的动词，或没有效果的方法的返回值。
- **宏名称应该全部大写，单词之间以下划线分隔，并带上前缀 `UE_` 。**
```
    #define UE_AUDIT_SPRITER_IMPORT
```
    

**返回bool的所有函数应该提出true/false问题**，例如 `IsVisible()` 或 `ShouldClearBuffer()` 。

过程（没有返回值的函数）应该使用强动词，后跟一个对象。如果方法的对象是它所在的对象，则例外。在这种情况下，对象会根据上下文进行理解。要避免的名称包括以"Handle"和"Process"开头的名称，因为这些动词有歧义。

在以下情况下，我们鼓励你为函数参数名称带上前缀"`Out`"：
- 函数参数按引用传递。
- 函数应该写入该值。

这样一来，显然此参数中传递的值会被函数替换。

如果`In`或`Out`参数也是布尔值，将"`b`"放在In/Out前缀之前，例如 `bOutResult` 。

返回值的函数应该说明返回值。名称应该清楚表明函数返回什么值。这对于布尔值函数尤其重要。考虑以下两个示例方法：

```c++
// 这里true表示什么意思？
bool CheckTea(FTea Tea);

// 名称清楚表明true表示茶是新鲜的
bool IsTeaFresh(FTea Tea);

float TeaWeight;
int32 TeaCount;
bool bDoesTeaStink;
FName TeaName;
FString TeaFriendlyName;
UClass* TeaClass;
USoundCue* TeaSound;
UTexture* TeaTexture;
```

## 包容性选词





## 标准库的用法

过去，UE避免直接使用C和C++标准库，原因如下：

- 将缓慢的实现替换为我们自己的实现，提供对内存分配的额外控制。
- 在新功能广泛可用之前进行添加，例如：
    
    - 做出合适但非标准的行为更改。
        
    - 在基本代码中采用一致的语法。
        
    - 避免与UE的惯用法不兼容的构造。
        

但是，标准库已经成熟，并且包括我们不想封装到抽象层中或自行重新实现的功能。

如果可以选择标准库功能而不是我们自己的功能，你应该首选可以得到出色结果的选项。此外还应牢记，一致性非常重要。如果旧版UE实现已经无法达到目的，我们可以选择将其弃用，并将所有用法向标准库迁移。

避免在相同API中混用UE惯用法和标准库惯用法。下表列出了常见惯用法以及关于何时使用它们的推荐做法。

|惯用法|说明|
|---|---|
|`<atomic>`|Atomic惯用法应该在新代码中使用，并在改动时迁移旧词。Atomic应该在所有支持的平台上完全、高效地实现。我们自己的 `TAtomic` 仅部分实现，对其进行维护和改进并不符合我们的利益。|
|`<type_traits>`|在旧版UE特征和标准特征之间存在重叠的情况下，应该使用类型特征惯用法。 特征常常实现为编译器固有属性，以确保正确，编译器可以知道标准特征并选择更快速的编译路径，而不是将其视为普通C++。有一个问题是，我们的特征通常有大写的 `Value` static或 `Type` typedef，而标准特征预期会使用 `value` 和 `type` 。这是很重要的差异，因为组合特征应该使用特定语法，例如 `std::conjunction` 。我们添加的新特征应该使用小写的 `value` 或 `type` 来编写，以支持组合。现有特征应该更新，以支持任一大小写格式。|
|`<initializer_list>`|必须使用初始化器列表惯用法以支持加大括号的初始化器语法。这种情况下，语言和标准库有重叠。如果你想支持它，别无他法。|
|`<regex>`|Regex惯用法可以直接使用，但其使用应该封装在仅限编辑器的代码中。 我们不打算实现自己的regex解决方案。|
|`<limits>`|`std::numeric_limits` 可以完整使用。|
|`<cmath>`|此头文件中的所有浮点函数都可以使用。|
|`<cstring>`: `memcpy()` 和 `memset()`|这些惯用法在有明显的性能优势时，可以分别代替 `FMemory::Memcpy` 和 `FMemory::Memset` 使用。|

除了interop代码中之外，应该避免标准容器和字符串。

## 注释


### Const正确性

Const既是文档，也是编译器指令。所有代码都应力求const正确性。这包括以下准则：

- 如果函数参数不应该被函数修改，则按const指针或引用传递这些参数。
- 将不会修改对象的方法标记为const。
- 如果循环不应该修改容器，则使用const迭代取代容器。
    

Const示例：

```c++
void SomeMutatingOperation(FThing& OutResult, const TArray<Int32>& InArray)
{
    // InArray在此处不会被修改，但OutResult可能会被修改
}

void FThing::SomeNonMutatingOperation() const
{
    // 此代码不会修改调用它时所在的FThing
}

TArray<FString> StringArray;
for (const FString& : StringArray)
{
    // 此循环的主体不会修改StringArray
}
```

对于按值传递的函数参数和局部参数，Const也是首选项。如此一来，阅读者就知道变量在函数主体中不会被修改，这样更容易理解。如果你这样做，请确保声明和定义匹配，因为这可能会影响JavaDoc过程。

```c++
void AddSomeThings(const int32 Count);

    void AddSomeThings(const int32 Count)
    {
        const int32 CountPlusOne = Count + 1;
        // 在函数主体内，Count和CountPlusOne都不能更改
    }
```

按值传递的参数是一个例外，它们会移入容器中。如需详细信息，请参阅本页面上的"移动语义"小节。

```c++
    void FBlah::SetMemberArray(TArray<FString> InNewArray)
    {
        MemberArray = MoveTemp(InNewArray);
    }
```

将指针本身设为const（而不是它所指向的内容）时，请将const关键字放在末尾。引用本来也无法"重新赋值"，因此不能以相同方式设为const。

```c++
// 非const对象-指针的const指针不能重新赋值，但T仍可以修改
T* const Ptr = ...;

// 非法
T& const Ref = ...;
```

**切勿对返回类型使用 const。** 这会禁止复杂类型的移动语义，但会对内置类型给出编译警告。此规则仅适用于返回类型本身，而不适用于所返回的指针或引用的目标类型。

```c++
// 糟糕 - 返回const数组
const TArray<FString> GetSomeArray();

// 可接受 - 返回const数组的引用
const TArray<FString>& GetSomeArray();

// 可接受 - 返回const数组的指针
const TArray<FString>* GetSomeArray();

// 糟糕 - 返回const数组的const指针
const TArray<FString>* const GetSomeArray();
```



## 现代C++语言语法



**虚幻引擎默认使用语言版本C++20进行编译，并且至少需要版本C++17才能编译**。我们使用的许多现代语言功能在各个现代编译器中都得到了很好的支持。在一些情况下，我们将这些功能的使用封装在预处理器条件语句中。但是，有时出于可移植性或其他原因，我们会完全避开特定语言功能。

**除了下面指定的情况之外，作为我们支持的现代C++编译器功能，你不应该使用特定于编译器的语言功能，除非它们封装在预处理器宏或条件语句中并偶尔使用。**

### Static Assert

在你需要编译时断言的情况下， `static_assert` 关键字可以使用。

### Override和Final

`override` 和 `final` 关键字可以使用，并且强烈推荐。有可能在许多地方，这些被省略了，但这些问题会逐渐加以修复。

### Nullptr

你应该始终使用 `nullptr` 而不是C样式的 `NULL` 宏。

在C++/CX版本（例如Xbox One的版本）中使用 `nullptr` 是一个例外。在此情况下， `nullptr` 的使用实际上是受管的null引用类型。除了类型和一些模板实例化上下文之外，它与原生C++中的 `nullptr` 大体兼容，因此你应该使用 `TYPE_OF_NULLPTR` 宏而不是更常用的 `decltype(nullptr)` ，以便兼容。

### Auto

你不应该使用C++代码中的 `auto` ，但下面列出的少数例外情况除外。始终明确指出你要初始化的类型。这意味着，类型必须对阅读者显而易见。此规则还适用于C#中 `var` 关键字的使用。

C++17的结构化绑定功能也不应该使用，因为它实际上相当于参数个数可变的 `auto` 。

可接受的 `auto` 用法：

- 当你需要将lambda绑定到变量时，因为lambda类型无法在代码中表达。
    
- 对于迭代器变量，但仅在迭代器类型冗长并且会损害可读性的情况下。
    
- 在模板代码中，其中表达式的类型无法轻松辨别。这是一种高级情况。
    

务必确保类型对代码阅读者清晰可辨。尽管一些IDE能够推断类型，但这样做依赖于代码处于可编译的状态。它也不会辅助merge/diff工具的用户，或者孤立查看单独的源文件时（例如在GitHub上）也不起作用。

如果你确信自己是以可接受的方式使用 `auto` ，始终记住正确使用 `const` 、 `&` 、 `*` ，就像使用类型名称那样。使用 `auto` ，这会迫使推断的类型为你需要的类型。

### 基于范围的For

这对于使代码更易于理解、更好维护而言非常有利。当你迁移使用旧 `TMap` 迭代器的代码时，请注意，旧 `Key()` 和 `Value()` 函数过去是迭代器类型的方法，现在就是底层键值 `TPair` 的 `Key` 和 `Value` 字段。

示例：

```
TMap<FString, int32> MyMap;

    // 旧样式
    for (auto It = MyMap.CreateIterator(); It; ++It)
    {
        UE_LOG(LogCategory, Log, TEXT("Key: %s, Value: %d"), It.Key(), *It.Value());
    }

    // 新样式
    for (TPair<FString, int32>& Kvp : MyMap)
    {
        UE_LOG(LogCategory, Log, TEXT("Key: %s, Value: %d"), *Kvp.Key, Kvp.Value);
    }
```

我们还将一些独立的迭代器类型替换为范围样式。

示例：

```
    // 旧样式
    for (TFieldIterator<UProperty> PropertyIt(InStruct, EFieldIteratorFlags::IncludeSuper); PropertyIt; ++PropertyIt)
    {
        UProperty* Property = *PropertyIt;
        UE_LOG(LogCategory, Log, TEXT("Property name: %s"), *Property->GetName());
    }

    // 新样式
    for (UProperty* Property : TFieldRange<UProperty>(InStruct, EFieldIteratorFlags::IncludeSuper))
    {
        UE_LOG(LogCategory, Log, TEXT("Property name: %s"), *Property->GetName());
    }
```

### Lambda和匿名函数

Lambda可以自由使用。最好的lambda应该最多一两行语句，尤其是在用作更大表达式或语句的一部分时，例如作为通用算法中的谓词。

示例：

```
// 找到其名称包含单词"Hello"的第一个事物

Thing* HelloThing = ArrayOfThings.FindByPredicate([](const Thing& Th){ return Th.GetName().Contains(TEXT("Hello")); });

    // 按名称逆序对数组排序    
Algo::Sort(ArrayOfThings, [](const Thing& Lhs, const Thing& Rhs){ return Lhs.GetName() > Rhs.GetName(); });|
```

请注意，有状态的lambda不能赋给函数指针，后者我们往往用得很频繁。重要的lambda应该按普通函数的相同方式记录。

#### 捕获和返回类型

应该使用显式捕获而不是自动捕获（ `[&]` 和 `[=]` ）。这对于提高可读性、可维护性和性能非常重要，尤其是在用于大型lambda和延迟执行时。

显式捕获可声明创建者的意图，因此，审核代码时会发现错误。不正确的捕获可能造成负面后果，随着代码的逐渐维护，这更有可能带来问题。就lambda捕获而言，还有一些额外的事项需要注意：

- 如果lambda的执行延迟，指针的按引用捕获和按值捕获（包括 `this` 指针）可能导致意外悬空的引用。
    
- 按值捕获如果为非延迟lambda创建不必要的副本，可能会造成性能问题。
    
- 意外捕获的UObject指针对垃圾回收器不可见。如果引用了成员变量，自动捕获会隐式捕捉 `this` ，尽管 `[=]` 会让人以为lambda对一切都有自己的副本。
    

显式返回类型应该用于大型lambda，或在你返回另一个函数调用的结果时使用。这些应该按照与 `auto` 关键字相同的方式考虑：

### 强类型枚举

枚举类可取代旧样式命名空间的枚举，对于普通枚举和 `UENUM` 都是如此。例如：

```
  // 旧枚举
    UENUM()
    namespace EThing
    {
        enum Type
        {
            Thing1,
            Thing2
        };
    }

    // 新枚举
    UENUM()
    enum class EThing : uint8
    {
        Thing1,
        Thing2
    }
```

枚举作为 `UPROPERTY` 受支持，并取代旧的 `TEnumAsByte<>` 变通方案。枚举属性也可以是任意大小，不仅仅是字节：

```
 // 旧属性
   UPROPERTY()
   TEnumAsByte<EThing::Type> MyProperty;

   // 新属性
   UPROPERTY()
   EThing MyProperty;
```

公开给蓝图的枚举必须继续基于 `uint8` 。

用作标记的枚举类可以利用 `ENUM_CLASS_FLAGS(EnumType)` 宏自动定义所有按位运算符：

```
  enum class EFlags
    {
        None = 0x00,
        Flag1 = 0x01,
        Flag2 = 0x02,
        Flag3 = 0x04
    };

    ENUM_CLASS_FLAGS(EFlags)
```

在 _事实_ 上下文中使用标记是一个例外，这是语言的局限性。相反，所有枚举标记都应该有一个名为 `None` 的枚举器，设置为0以进行比较：

```
   // 旧
    if (Flags & EFlags::Flag1)  

    // 新
    if ((Flags & EFlags::Flag1) != EFlags::None) 
```

### 移动语义

所有主要容器类型（ `TArray` 、 `TMap` 、 `TSet` 、 `FString` ）都有移动构造函数和移动赋值运算符。这些常常会在按值传递或返回这些类型时自动使用。它们还可以通过使用 `MoveTemp` （ `std::move` 的UE等效项）来显式调用。

按值返回容器或字符串有利于更好地表达，而不会带来临时副本的惯常开销。围绕按值传递和使用 `MoveTemp` 的规则仍在制定中，但已经可以在基本代码的一些优化区域中找到。

### 默认成员初始化器

默认成员初始化器可用于在类本身中定义类的默认值：

```
    UCLASS()
    class UTeaOptions : public UObject
    {
        GENERATED_BODY()

    public:
        UPROPERTY()
        int32 MaximumNumberOfCupsPerDay = 10;

        UPROPERTY()
        float CupWidth = 11.5f;

        UPROPERTY()
        FString TeaType = TEXT("Earl Grey");

        UPROPERTY()
        EDrinkingStyle DrinkingStyle = EDrinkingStyle::PinkyExtended;
    };
```

像这样编写的代码有以下优势：

- 它不需要在多个构造函数之间复制初始化器。
    
- 初始化顺序和声明顺序不会搞混。
    
- 成员类型、属性标记和默认值全都在一个地方。这有助于提高可读性和可维护性。
    

但是，也有一些不利的地方：

- 更改默认值就需要重新编译所有依赖项文件。
    
- 头文件无法在引擎的补丁版本中更改，所以此样式会限制可能的修复种类。
    
- 一些事项无法以这种方式初始化，例如基类、 `UObject` 子对象、前向声明的类型的指针、从构造函数参数推断的值，以及通过多个步骤初始化的成员。
    
- 将一些初始化器放在头文件中，而将其余初始化器放在.cpp文件中的构造函数中，可能会降低可读性和可维护性。
    

在决定是否使用默认成员初始化器时，请自行做出最佳判断。 根据经验，默认成员初始化器用在游戏内代码中比用在引擎代码中更有意义。考虑将配置文件用于默认值。

## 第三方代码

每当你修改我们在引擎中所用库的代码时，请务必使用//@UE5注释标记你的更改，同时说明更改原因。这样就可以更轻松地将更改合并到该库的新版本中，并确保持证人可以轻松找到我们做出的修改。

引擎中包含的所有第三方代码都应该使用注释标记，注释应采用可轻松搜索的格式。例如：

```
// @第三方代码 - BEGIN PhysX
#include <physx.h>
// @第三方代码 - END PhysX
// @第三方代码 - BEGIN MSDN SetThreadName
// [http://msdn.microsoft.com/en-us/library/xcb2z8hs.aspx]
// 用于在调试器中设置线程名称
...
//@第三方代码 - END MSDN SetThreadName
```

## 代码格式

### 大括号

关于大括号的用法，人们争得不可开交。Epic一直以来都采用了将大括号换行的用法模式。请遵守此用法。

始终将大括号包含在单语句块中。例如：

```
    if (bThing)

    {

        return;

    }
```

### If - Else

一个if-else语句中每个执行块都应该放在大括号中。这有助于防止编辑错误。不使用大括号时，有人可能会无意中向某个if块添加另一行。额外的行不受if表达式控制，这是很糟糕的。当有条件编译的项导致if/else语句中断时，也会带来糟糕的结果。所以始终使用大括号。

```
if (bHaveUnrealLicense)
{
    InsertYourGameHere();
}
else
{
    CallMarkRein();
}
```

多路if语句应该缩进显示，每个 `else if` 缩进数量与第一个 `if` 相同；这样结构更清晰易读：

```
    if (TannicAcid < 10)
    {
        UE_LOG(LogCategory, Log, TEXT("Low Acid"));
    }
    else if (TannicAcid < 100)
    {
        UE_LOG(LogCategory, Log, TEXT("Medium Acid"));
    }
    else
    {
        UE_LOG(LogCategory, Log, TEXT("High Acid"));
    }
```

### 制表符和缩进

下面是代码缩进的一些标准。

- 按执行块缩进代码。
    
- 使用制表符表示行首的空白，而不使用空格。将制表符大小设置为4个字符。请注意，有时为使代码保持对齐，空格是必要和允许的，无论制表符中的空格数量是多少。例如，当你要对齐的代码采用非制表符字符时。
    
- 如果你使用C#编写代码，也请使用制表符，而不是空格。这样做的原因是，程序员常常在C#和C++之间切换，大部分人偏好为制表符使用一致的设置。Visual Studio默认将空格用于C#文件，因此你需要记住在虚幻引擎代码上工作时更改此设置。
    

### Switch语句

除了空case（多个case有相同的代码）之外，switch case语句应该显式标注一个case通达下一个case。要么包括一个break，要么在每个case中包括"falls through"注释。其他代码控制-传输命令（return、continue等）也可接受。

始终要有默认case。 包括一个break，以防有人在默认case之后添加新的case。

```
switch (condition)
    {
        case 1:
            ...
            // falls through

        case 2:
            ...
            break;

        case 3:
            ...
            return;

        case 4:
        case 5:
            ...
            break;

        default:
            break;
    }
```

## 命名空间

你可以使用命名空间来相应整理你的类、函数和变量。如果确实要使用它们，请遵守下面的规则。

- 大部分UE代码当前未封装在全局命名空间中。
    
    - 仔细避免全局范围内的冲突，尤其是在使用或包括第三方代码时。
        
- UnrealHeaderTool不支持命名空间。
    
    - 定义 `UCLASS` 、 `USTRUCT` 等时，不应该使用命名空间。
        
- 不是 `UCLASS` 、 `USTRUCT` 等的新API应该放在 `UE::` 命名空间中，最好是嵌套的命名空间中，例如 `UE::Audio::` 。  
    
    - 若命名空间用于保存不属于面向公众的API的实现细节，应该放在 `Private` 命名空间中，例如 `UE::Audio::Private::` 。
        
- `Using` 声明：
    
    - 禁止将 `using` 声明放在全局范围内，即使是在.cpp文件中（这会导致我们的"unity"编译系统出现问题）。
        
- 将 `using` 声明放在另一个命名空间中或函数主体中是可接受的。
    
- 如果你将 `using` 声明放在命名空间中，这会结转到相同转换单元中该命名空间的其他发生情况只要你保持一致，都是可以接受的。
    
- 只有遵守以上规则，才能在头文件中安全地使用 `using` 声明。
    

*前向声明的类型需要在其相应的命名空间中声明。

```
* 如果不这样做，你会收到链接错误。
```

- 如果你在命名空间中声明大量类或类型，可能很难在其他全局范围的类中使用这些类型（例如，函数签名将需要在类声明中出现时使用显式命名空间）。
    
- 你只能使用 `using` 声明对你所在范围的命名空间中的特定变量设定别名。
    
    - 例如，使用 `Foo::FBar` 。但是，我们通常不会在虚幻引擎代码中那样做。
        
- 宏不能放在命名空间中。
    
    - 它们应该带上前缀 `UE_` ，而不是放在命名空间中，例如 `UE_LOG` 。
        

## 物理依赖性

- 文件名应该尽可能不带前缀。
    
    - 例如，采用 `Scene.cpp` 而不是 `UScene.cpp` 。这样会减少识别你想要的文件所需的字母数量，可以轻松使用Workspace Whiz或Visual Assist的Open File in Solution等工具。
        
- 所有头文件都应该防止带有 `#pragma once` 指令的多个include语句。
    
    - 请注意，我们使用的所有编译器都支持 `#pragma once` 。
        

```
#pragma once
        //<file contents>
```

- 尽量减少物理耦合。
    
    - 尤其是，避免包括来自其他头文件的标准库头文件。
        
- 前向声明比包括头文件更好。
    
- 包括头文件时，尽可能精细。
    
    - 例如，不要包括 `Core.h` 。相反，你应该包括Core中你需要其中定义的特定头文件。
        
- 尽量直接包括你需要的每个头文件，以便更轻松地包括精细的内容。
    
- 不要依赖你包括的另一个头文件所间接包括的头文件。
    
- 不要依赖通过另一个头文件包括的任何内容。包括你需要的一切内容。
    
- 模块有私有和公共源文件目录。
    
    - 其他模块需要的所有定义都必须在公共目录中的头文件中。其他一切都应该在私有目录中。在旧版虚幻引擎模块中，这些目录可能仍称为"Src"和"Inc"，但这些目录旨在以相同方式分隔私有和公共代码，并非用于分隔头文件和源文件。
        
- 不用费心为预编译的头文件生成而设置头文件。
    
    - UnrealBuildTool可以比你更好地完成这项工作。
        
- 将大型函数拆分为多个逻辑子函数。
    
    - 编译器优化的一个方面是消除了常见的子表达式。函数越大，编译器就要执行越多的工作来识别它们。这会大幅提高编译时间。
        
- 不要使用大量内联函数。
    
    - 使用内联函数，就会迫使系统在并不使用它们的文件中也进行重新编译。内联函数只应该用于不重要的访问器，以及在分析表明这样做有利的情况下。
        
- 谨慎使用 `FORCEINLINE` 。
    
    - 所有代码和局部变量都将扩展到调用函数中。这会导致大型函数所导致的相同编译时间问题。
        

# 封装

使用保护关键字强制封装。类成员应该几乎始终声明为私有，除非它们是类的public/protected接口的一部分。请自行做出最佳判断，但随时注意，缺少访问器，以后进行重构时很难做到不破坏插件和现有项目。

如果特定字段应该仅供派生类使用，请将其设为private并提供protected访问器。

如果你的类不允许继续派生，请使用final。

## 一般样式问题

- 尽量缩小依赖性距离。
    
    - 当代码依赖某个变量有特定值时，尽量在设置该变量的值之后立即使用它。若在执行块顶部初始化变量，而在接下来上百行代码中都不使用它，极有可能有人会意外更改其值，却没有意识到依赖性。在下一行使用它，就能清楚表明如此初始化变量的原因以及它在哪里使用。
        
- 尽可能将方法分拆为多个子方法。
    
    - 阅读者先了解概况，然后再深入了解有意思的细节，比一开始就从细节着手并据以重构大局要轻松得多。同样，简单的方法理解起来更加轻松，即调用一系列命名良好的子方法，比一个直接包含这些子方法中的所有代码的等效方法要更容易理解。
        
- 在函数声明或函数调用站点中，禁止在函数名称与参数列表前的圆括号之间添加空格。
    
- 解决编译器警告。
    
    - 编译器警告消息表明出了问题。修复编译器向你警告的情况。如果你完全无法加以解决，使用 `#pragma` 消除警告，但这应该仅作为最后的办法。
        
- 在文件末尾保留一个空白行。
    
    *所有.cpp和.h文件应该包括一个空白行，以与gcc协调。
    
- 调试代码应该要么有用、完美，要么不检入。
    
    - 若调试代码与其他代码混杂在一起，会使其他代码更难阅读。
        
- 始终将字符串字面值括在 `TEXT()` 宏内。
    
    - 如果没有 `TEXT()` 宏，从字面值构造 `FString` 的代码会造成意外的字符串转换过程。
        
- 避免在循环中冗余地重复相同操作。
    
    - 将常见子表达式移出循环，避免冗余计算。在一些情况下利用static，避免函数调用之间全局冗余的运算，例如从字符串字面值构造 `FName` 。
        
- 留意热重载。
    
    - 尽量减少依赖性，缩短迭代时间。不要将内联或模板用于在重新加载时很可能会更改的函数。仅将static用于预期在重新加载时会保持不变的事项。
        
- 使用中间变量，简化复杂的表达式。
    
    - 如果你有复杂的表达式，将其分拆为多个子表达式并赋给中间变量，用名称说明子表达式在父表达式中的含义，这样更容易理解。例如：
        
    
    ```
        if ((Blah->BlahP->WindowExists->Etc && Stuff) &&
            !(bPlayerExists && bGameStarted && bPlayerStillHasPawn &&
            IsTuesday())))
        {
            DoSomething();
        }
    ```
    
    应该替换为：
    
    ```
       const bool bIsLegalWindow = Blah->BlahP->WindowExists->Etc && Stuff;
        const bool bIsPlayerDead = bPlayerExists && bGameStarted && bPlayerStillHasPawn && IsTuesday();
        if (bIsLegalWindow && !bIsPlayerDead)
        {
            DoSomething();
        }
    ```
    
- 指针和引用应该仅在指针或引用右侧有一个空格。
    
    - 这样就可轻松快速地将 **在文件中查找（Find in Files）** 用于特定类型的所有指针或引用。例如：
        
    
    ```
    // 使用这些
    
         FShaderType* Ptr
    
        *// 不要使用这些：*
    
            FShaderType *Ptr
            FShaderType * Ptr
    ```
    
- 不允许影子变量。
    
    - C++允许变量从外层范围投影，但这样会使人读起来有歧义。例如，以下成员函数中有三个可使用的 `Count` 变量：
        
    
    ```
     class FSomeClass
            {
            public:
                void Func(const int32 Count)
                {
                    for (int32 Count = 0; Count != 10; ++Count)
                    {
                        // 使用Count
                    }
                }
    
            private:
                int32 Count;
            }
    ```
    
- 避免在函数调用中使用匿名字面值。
    
    - 首选说明其含义的命名常量。这样阅读者一眼就能看出意图，不必查阅函数声明即可理解。
        
    
    ```
    // 旧样式
            Trigger(TEXT("Soldier"), 5, true);.
    
            // 新样式
            const FName ObjectName                = TEXT("Soldier");
            const float CooldownInSeconds         = 5;
            const bool bVulnerableDuringCooldown  = true;
            Trigger(ObjectName, CooldownInSeconds, bVulnerableDuringCooldown);
    ```
    
- 避免在头文件中定义重要的static变量。
    
    - 重要的static变量会导致实例编译到包括该头文件的每个转换单元中。
        
    
    ```
    // SomeModule.h
            static const FString GUsefulNamedString = TEXT("String");
    
    // 应该替换为
    
            // SomeModule.h
            extern SOMEMODULE_API const FString GUsefulNamedString;
    
            // SomeModule.cpp
            const FString GUsefulNamedString = TEXT("String");    
    ```
    

## API设计准则

- 应该避免布尔值函数参数。
    
    - 尤其是，对于传递到函数的标记，应该避免布尔值参数。这些会带来与前面所述相同的匿名字面值问题，但随着API扩展更多行为，它们也往往会与日俱增。相反，首选枚举（请参阅[强类型枚举](https://docs.unrealengine.com/5.2/zh-CN/epic-cplusplus-coding-standard-for-unreal-engine#%E5%BC%BA%E7%B1%BB%E5%9E%8B%E6%9E%9A%E4%B8%BE)小节中关于使用枚举作为标记的建议）：
        
    
    ```
      // 旧样式
            FCup* MakeCupOfTea(FTea* Tea, bool bAddSugar = false, bool bAddMilk = false, bool bAddHoney = false, bool bAddLemon = false);
            FCup* Cup = MakeCupOfTea(Tea, false, true, true);
    
            // 新样式
            enum class ETeaFlags
            {
                None,
                Milk  = 0x01,
                Sugar = 0x02,
                Honey = 0x04,
                Lemon = 0x08
            };
            ENUM_CLASS_FLAGS(ETeaFlags)
    
            FCup* MakeCupOfTea(FTea* Tea, ETeaFlags Flags = ETeaFlags::None);
            FCup* Cup = MakeCupOfTea(Tea, ETeaFlags::Milk | ETeaFlags::Honey);
    ```
    
- 这种格式可防止标记意外转置，避免从指针和整型参数意外转换，不必重复冗余的默认值，并且效率更高。  
    
- 如果参数是要传递到setter等函数的完整状态，使用 `bool` 作为参数是可接受的，例如 `void FWidget::SetEnabled(bool bEnabled)` 。但如果这发生更改，请考虑重构。
    
- 避免过长的函数参数列表。
    
    - 如果函数采用许多参数，请考虑改为传递专用结构体：
        
    
    ```
        // 旧样式
        TUniquePtr<FCup[]> MakeTeaForParty(const FTeaFlags* TeaPreferences, uint32 NumCupsToMake, FKettle* Kettle, ETeaType TeaType = ETeaType::EnglishBreakfast, float BrewingTimeInSeconds = 120.0f);
    
        // 新样式
        struct FTeaPartyParams
        {
            const FTeaFlags* TeaPreferences       = nullptr;
            uint32           NumCupsToMake        = 0;
            FKettle*         Kettle               = nullptr;
            ETeaType         TeaType              = ETeaType::EnglishBreakfast;
            float            BrewingTimeInSeconds = 120.0f;
        };
        TUniquePtr<FCup[]> MakeTeaForParty(const FTeaPartyParams& Params);
    ```
    
- 避免使用 `bool` 和 `FString` 来重载函数。
    
    - 这可能带来意外的行为：
        
    
    ```
        void Func(const FString& String);
            void Func(bool bBool);
    
            Func(TEXT("String")); // 调用布尔重载！|
    ```
    
- 接口类应该始终为抽象类。 '
    
    - 接口类要带上前缀"I"，不得包含成员变量。接口可以包含非pure virtual的方法，并可以包含非virtual或static的方法，只要它们是内联实现的即可。
        
- 声明重写方法时，请使用 `virtual` 和 `override` 关键字。
    

在重写父类中的virtual函数的派生类中声明virtual函数时，你必须同时使用 `virtual` 和 `override` 关键字。例如：

```
~~~  
     class A
        {
        public:
            virtual void F() {}
        };

        class B : public A
        {
        public:
            virtual void F() override;
        }
~~~ 
```

有许多现有代码还没有遵守这一点，因为 `override` 关键字是最近添加的。`override` 关键字应该在方便的时候添加到该代码中。

## 特定于平台的代码

特定于平台的代码应该在恰当命名的子目录中特定于平台的源文件中抽象和实现，例如：

```
    Engine/Platforms/[PLATFORM]/Source/Runtime/Core/Private/[PLATFORM]PlatformMemory.cpp
```

一般来说，你应该避免添加 `PLATFORM_[PLATFORM]` 的用法。例如，避免将 `PLATFORM_XBOXONE` 添加到名为 `[PLATFORM]` 的目录之外的代码中。相反，扩展硬件抽象层以添加一个static函数，例如在FPlatformMisc中：

```
   FORCEINLINE static int32 GetMaxPathLength()
    {
        return 128;
    }
```

接着，平台可以重写此函数，返回特定于平台的常量值，甚至也可以使用平台API确定结果。如果你强制将函数变为内联，性能特征与使用define相同。

在绝对有必要使用define的情况下，创建新的 `#define` 指令来说明可以应用于平台的特定属性，例如 `PLATFORM_USE_PTHREADS` 。在 `Platform.h` 中设置默认值，并在特定于平台的 `Platform.h` 文件中为需要它的平台重写。

例如，在 `Platform.h` 中，我们包含了：

```
  #ifndef PLATFORM_USE_PTHREADS 
        #define PLATFORM_USE_PTHREADS 1
    #endif
```

`WindowsPlatform.h` 包含了：

```
#define PLATFORM_USE_PTHREADS 0
```

接着，跨平台代码可以直接使用define，而无需知道平台。

```
#if PLATFORM_USE_PTHREADS 
        #include "HAL/PThreadRunnableThread.h"
    #endif
```

我们将引擎中特定于平台的细节加以集中，这样细节就可以完全包含在特定于平台的源文件中。这样做就更容易在多个平台之间维护引擎，此外，你还能够将代码移植到新平台，而无需在基本代码中搜查特定于平台的define。

将平台代码保持在特定于平台的文件夹中，也是PlayStation、Xbox和Nintendo Switch等NDA平台的要求。

务必确保无论 `[PLATFORM]` 子目录是否存在，代码都能编译并运行。换句话说，跨平台代码绝不应该依赖特定于平台的代码。