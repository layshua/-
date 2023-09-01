---
title: U++精粹
create_time: 2023-09-01 13:03
uid: "202309011303"
banner: "![[]]"
reference: []
---

1.  [大钊老师啊：虚幻C++进阶之路](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1C7411F7RF)
2.  [梁迪_梁迪腾讯课堂官网](https://link.zhihu.com/?target=http%3A//didi.ke.qq.com/%23tab%3D1%26category%3D-1) （迪迪老师 教你 C 草实战）
3.  [技术宅阿棍儿](https://link.zhihu.com/?target=https%3A//space.bilibili.com/92060300/video)（大标题直击要领，很棒）
4.  [来自程序员的暴击：虚幻四C++入坑指南合集版](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV14K411J7v2)

# 一、属性 UPROPERTY
## 属性声明

属性使用标准的 C++变量语法声明，前面用 `UPROPERTY` 宏来定义属性元数据和变量说明符。
`UPROPERT` 宏作为声明序言的变量可被引擎执行垃圾回收，也可在虚幻编辑器中显示和编辑。
当你需要将一个 UObject 类的子类的成员变量注册到蓝图中时，你只需要借助 `UPROPERTY` 宏即可完成。

```c++
UPROPERTY([specifier, specifier, ...], [meta(key=value, key=value, ...)])
Type VariableName;
```

## 核心数据类型

### 整数

整数数据类型转换是"int"或"uint"后跟位大小。

|变量类型|说明|
|---|---|
|**uint8**|8位无符号|
|**uint16**|16位无符号|
|**uint32**|32位无符号|
|**uint64**|64位无符号|
|**int8**|8位有符号|
|**int16**|16位有符号|
|**int32**|32位有符号|
|**int64**|64位有符号|

#### 作为位掩码
##### 整数属性位掩码
整数属性现在可**以位掩码形式公开给编辑器**。要将整数属性标记为位掩码，只需在 meta 分段中添加 `Bitmask` 即可，如下所示：

```c++
/*~ BasicBits appears as a list of generic flags in the editor, instead of an integer field. */
//BasicBits在编辑器中显示为一个通用flag标志列表，而不是一个整数字段
UPROPERTY(EditAnywhere, Meta = (Bitmask))
int32 BasicBits;
```

添加此元标记将使整数作为下拉列表形式可供编辑，它们使用笼统命名标记（"Flag 1"、"Flag 2"、"Flag 3"等等），可以 单独打开或关闭。

![[cd2166f5df34fcddfa25e7e5196ce3a7_MD5.jpg]]
##### 蓝图整数位掩码
**也可以让蓝图可调用函数的整型参数表现为位掩码**，方法是在参数的 `UPARAM` 指定器上添加 `Bitmask` 元标签（不需要值）。

```c++
/*~ You can set MyFunction using a generic list of flags instead of typing in an integer value. */
//你可以使用一个通用的flag标志列表来设置MyFunction，而不是输入一个整数值。
UFUNCTION(BlueprintCallable)
void MyFunction(UPARAM(meta=(Bitmask)) int32 BasicBitsParam)
```
##### 自定义位标记名称
**自定义位标记名称**，首先必须使用`Bitflags`元标记来创建 `UENUM`：

```c++
UENUM(Meta = (Bitflags))
enum class EColorBits
{
    ECB_Red,
    ECB_Green,
    ECB_Blue
};
```

比特掩码枚举类型的范围是0到31，包括0和31。其对应于32位整型变量的位数（从第0位开始）。在上面的例子中，第0位是 `ECB_Red`，第1位是 `ECB_Green`，第2位是 `ECB_Blue`。

作为**另一种声明方式，你可以使用 `ENUM_CLASS_FLAGS` 在定义完枚举类型后，将其变成一个位掩码。** 为了在编辑器中使用标志选择器（flag selector），我们还必须添加元字段 `UseEnumValuesAsMaskValuesInEditor` 并将其设置为 `true`。关键的区别在于，这个方法直接使用掩码值，而不是比特数。使用此方法制作的等效枚举类型看起来像这样：

```c++
UENUM(Meta = (Bitflags, UseEnumValuesAsMaskValuesInEditor = "true"))
enum class EColorBits
{
    ECB_Red = 0x01,
    ECB_Green = 0x02,
    ECB_Blue = 0x04
};

ENUM_CLASS_FLAGS(EColorBits);
```

**创建该UENUM后，可以使用`BitmaskEnum`元标记来引用它**，如：

```c++
/*~ This property lists flags matching the names of values from EColorBits. */
//这个属性列出了与EColorBits中值的名称相匹配的标志
UPROPERTY(EditAnywhere, Meta = (Bitmask, BitmaskEnum = "EColorBits"))
int32 ColorFlags;
```

完成这个更改后，下拉框中列出的位标记将使用列举类条目的名称和值。在上述示例中， ECB_Red 值为0，表示它被选中时将激活位0（将ColorFlags增加1）。ECB_Green对应于位1（将ColorFlags增加2），ECB_Blue 对应于位2（将ColorFlags增加4）。

![[47dbb9ed69b8e8601e913aa90c304f74_MD5.jpg]]

**同样，你可以在 `UPARAM` 标签的meta部分添加 `BitmaskEnum` 和对应的枚举类型名称来定制它。**

```c++
/*~ MyOtherFunction shows flags named after the values from EColorBits. */
UFUNCTION(BlueprintCallable)
void MyOtherFunction(UPARAM(meta=(Bitmask, BitmaskEnum = "EColorBits")) int32 ColorFlagsParam)
```

虽然列举类型包含超过32个条目，但在属性编辑器UI中，位掩码关联中只会看到前32个值。同样，虽然可接受显式值条目，但显式值介于0-31的条目不会包含在下拉列表中。

### 浮点类型

虚幻使用标准C++浮点类型、浮点和双精度。

### 布尔类型

布尔类型可以使用 C++ bool 关键字表示或表示为位域。
```c++
uint32 bIsHungry : 1;
bool bIsThirsty;
```

## 属性说明符

声明属性时，**属性说明符** 可被添加到声明，以控制属性与引擎和编辑器诸多方面的相处方式。

|属性标签|效果|
|---|---|
|`AdvancedDisplay`|属性将被放置在其出现的任意面板的**高级（下拉） 部分**中。|
|`AssetRegistrySearchable`|`AssetRegistrySearchable` 说明符说明此属性与其值将被自动添加到将此包含为成员变量的所有资源类实例的资源注册表。不可在结构体属性或参数上使用。|
|⭐ **`BlueprintAssignable`**|⭐只能与**多播委托**共用。公开属性在蓝图中指定。|
|`BlueprintAuthorityOnly`|此属性必须为一个多播委托。在蓝图中，其只接受带 `BlueprintAuthorityOnly` 标签的事件。 |
|⭐ **`BlueprintCallable`**|⭐ 仅用于**多播委托**。应公开属性在蓝图代码中调用。|
|`BlueprintGetter=GetterFunctionName` |此属性指定一个自定义存取器函数。如此属性不带 `BlueprintSetter` 或 `BlueprintReadWrite` 标签，则其为隐式 `BlueprintReadOnly`。 |
|⭐ **`BlueprintReadOnly`**|⭐此属性可由蓝图读取，但不能被修改。|
|⭐ **`BlueprintReadWrite`**|⭐可从蓝图读取或写入此属性。|
|`BlueprintSetter=SetterFunctionName`|此属性拥有一个自定义编译函数，被隐式标记为 `BlueprintReadWrite`。注意：必须对变异函数进行命名，并为相同类的一部分。 |
|⭐ **`Category="TopCategory\|SubCategory\|..."`**|⭐指定在蓝图编辑工具中显示时的属性类别。使用 \| 运算符定义嵌套类目。|
|`Config`|此属性将被设为可配置。当前值可被存入与类相关的 `.ini` 文件中，创建后将被加载。无法在默认属性中给定一个值。暗示为 `BlueprintReadOnly`。|
|`DuplicateTransient` |说明在任意类型的复制中（复制/粘贴、二进制复制等），属性的值应被重设为类默认值。|
|⭐ **`EditAnywhere`** |⭐说明此属性可通过属性窗口在原型和实例上进行编辑。|
|⭐ **`EditDefaultsOnly`**|⭐说明此属性可通过属性窗口进行编辑，但只能在原型上进行。|
|`EditFixedSize`|只适用于动态数组。这能防止用户通过虚幻编辑器属性窗口修改数组长度。|
|`EditInline`|允许用户在虚幻编辑器的属性查看器中编辑此属性所引用的Object的属性（只适用于Object引用，包括Object引用的数组）。|
|⭐ **`EditInstanceOnly`**|⭐说明此属性可通过属性窗口进行编辑，但只能在实例上进行，不能在原型上进行。|
|`Export`|只适用于Object属性（或Object数组）。说明Object被复制时（例如复制/粘贴操作）指定到此属性的Object应整体导出为一个子Object块，而非只是输出Object引用本身。|
|`GlobalConfig`|工作原理与 `Config` 相似，不同点是无法在子类中进行覆盖。无法在默认属性中对其给定一个值。暗示为 `BlueprintReadOnly`。|
|`Instanced`|仅限Object（`UCLASS`）属性。此类的一个实例创建时，其将被给定一个Object的特殊副本，指定到默认项中的此属性。用于实例化类默认属性中定义的子Object。暗示为 `EditInline` 和 `Export`。|
|`Interp`|说明值可随时间由Sequencer中的一个轨道驱动。|
|`Localized`|此属性的值将拥有一个定义的本地化值。多用于字符串。暗示为 `ReadOnly`。|
|`Native`|属性为本地：C++代码负责对其进行序列化并公开到[垃圾回收](https://docs.unrealengine.com/5.2/zh-CN/unreal-object-handling-in-unreal-engine)。|
|`NoClear`|阻止从编辑器将此Object引用设为空。隐藏编辑器中的清除（和浏览）按钮。|
|`NoExport`|只适用于本地类。此属性不应包含在自动生成的类声明中。|
|`NonPIEDuplicateTransient`|属性将在复制中被重设为默认值，除非其被复制用于PIE会话。|
|`NonTransactional`|说明对此属性值的修改不会包含在编辑器的撤销/重新执行历史中。|
|`NotReplicated`|跳过复制。这只会应用到服务请求函数中的结构体成员和参数。|
|`Replicated`|属性应随网络进行复制。|
|`ReplicatedUsing=FunctionName`|`ReplicatedUsing` 说明符指定一个回调函数，其在属性通过网络更新时执行。|
|`RepRetry`|只适用于结构体属性。如果此属性未能完全发送（举例而言：Object引用尚无法通过网络进行序列化），则重新尝试对其的复制。对简单引用而言，这是默认选择；但对结构体而言，这会产生带宽开销，并非优选项。因此在指定此标签之前其均为禁用状态。|
|`SaveGame`|此说明符可简便地将域显式包含，用于属性关卡中的检查点/保存系统。应在作为游戏存档一部分的所有域上设置此标签，并使用代理归档器对其进行读写。|
|`SerializeText`|本地属性应被序列化为文本（`ImportText`、`ExportText`）。|
|`SkipSerialization`|此属性不会被序列化，但仍能导出为一个文本格式（例如用于复制/粘贴操作）。|
|`SimpleDisplay`|出现在 **细节** 面板中的可见或可编辑属性，无需打开"高级"部分即可见。|
|`TextExportTransient`|此属性将不会导出为一个文本格式（因此其无法用于复制/粘贴操作）。|
|`Transient`|属性为临时，意味着其无法被保存或加载。以此方法标记的属性将在加载时被零填充。|
|⭐ **`VisibleAnywhere`**|⭐ 说明此属性在所有属性窗口中**可见**，但**无法被编辑**。此说明符与"Edit"说明符不兼容。|
|⭐ **`VisibleDefaultsOnly`**|⭐ 说明此属性只在原型的属性窗口中可见，**无法被编辑**。此说明符与所有"Edit"说明符均不兼容。|
|⭐ **`VisibleInstanceOnly`**|⭐ 说明此属性只在实例的属性窗口中可见（在原型属性窗口中不可见），**无法被编辑**。此说明符与所有"Edit"说明符均不兼容。|

# 二、对象 / 类 UCLASS
虚幻引擎包含一个用于处理游戏对象的强大系统。虚幻引擎中**所有对象的基类都是 `UObject`**。

声明包含一个类的标准 C++ 类声明。在标准声明之上，描述符（如类说明符和元数据）将被传递到 `UCLASS` 宏。
**`UCLASS` 宏的作用**是标记 `UObject` 的子类，以便 **UObject** 处理系统可以识别它们。

```c++
UCLASS([specifier, specifier, ...], [meta(key=value, key=value, ...)])
class ClassName : public ParentName
{
    GENERATED_BODY()
}
```

## 1 UCLASS 宏

**UCLASS** 宏为 `UObject` 提供了一个 `UCLASS` 引用，用于描述它在虚幻引擎中的类型。每个 `UCLASS` 都保留一个称作 **类默认对象（Class Default Object）** 的对象，简称`CDO`。`CDO` 本质上是一个默认"模板"对象，由类构建函数生成，之后就不再修改。

你可以为指定对象获取其 `UCLASS` 和 `CDO`，虽然它们通常都是只读类型。
>使用 **`GetClass()`** 函数即可随时访问对象实例的 UCLASS。


> [!NOTE]
> `UObject` 类还可包括仅限本地的属性，这些属性没有用 `UFUNCTION` 或者 `UPROPERTY` 指定器标记用于反射。**只有用指定器宏标记过的函数和属性会列举在它们对应的 `UCLASS` 中。**

## 2 类说明符

声明类时，可以为声明添加 **类说明符**，以控制类相对于引擎和编辑器的各个方面的行为。

|类说明符|效果|
|---|---|
| `Abstract` |**Abstract** 说明符会将类声明为"**抽象基类**"，阻止用户向关卡中添加此类的 Actor。对于单独存在时没有意义的类，此说明符非常有用。例如，`ATriggerBase` 基类是抽象类，而 `ATriggerBox` 子类不是抽象类，可以放置在关卡中。|
| `AdvancedClassDisplay` |**AdvancedClassDisplay** 说明符强制类的所有属性仅在显示这些属性的 ["细节面板（Details Panel）"](https://docs.unrealengine.com/5.2/zh-CN/level-editor-details-panel-in-unreal-engine) 的"高级（Advanced）"部分中显示。要覆盖单个属性上的此说明符，在该属性上使用 `SimpleDisplay` 说明符。|
| `AutoCollapseCategories=(Category1, Category2, ...)` | `AutoCollapseCategories` 说明符使父类上的 **AutoExpandCategories** 说明符的列出类别的效果无效。|
| `AutoExpandCategories=(Category1, Category2, ...)` |为此类的对象指定应自动在虚幻编辑器属性窗口中展开的一个或多个类别。要自动展开未使用类别声明的变量，请使用声明变量的类的名称。 |
|⭐ **`Blueprintable`**|⭐将此类公开为用于**创建蓝图的可接受基类**。默认为 `NotBlueprintable`，除非继承时就并非如此。此说明符由子类继承。|
|⭐ **`BlueprintType`**|将此类公开为可用于蓝图中的变量的类型。|
| `ClassGroup=GroupName` |指示在虚幻编辑器的 Actor 浏览器中启用 **组视图（Group View）** 时，**Actor 浏览器** 应在指定的 `GroupName` 中包含此类及此类的所有子类。|
| `CollapseCategories` |指示此类的属性不应划分到虚幻编辑器属性窗口的类别中。此说明符会传播到子类，可由 `DontCollapseCategories` 说明符覆盖。|
| `Config=ConfigName` |指示此类可在配置文件（`.ini`）中存储数据。如果存在任何使用 `config` 或 `globalconfig` 说明符声明的类属性，此说明符将使这些属性存储在指定的配置文件中。此说明符会传播到所有子类并且无法使此说明符无效，但是子类可通过重新声明 `config` 说明符并提供不同的 `ConfigName` 来更改配置文件。常见的 `ConfigName` 值是"Engine"、"Editor"、"Input"和"Game"。|
| `Const` |此类中的所有属性和函数都是 `const` 并且导出为 `const`。此说明符由子类继承。|
| `ConversionRoot` |根转换，将子类限制为仅可沿层级向上转换为第一个根类的子类。|
| `CustomConstructor` |阻止构造函数声明自动生成。|
| `DefaultToInstanced` |此类的所有实例都被认为是"实例化的"。实例化的类（组件）将在构造时被复制。此说明符由子类继承。|
| `DependsOn=(ClassName1, ClassName2, ...)` |列出的所有类将先于此类被编译。提供的类名必须指示同一个或前一个包中的类。可以使用单个 `DependsOn` 行（以逗号分隔）来标识多个依赖类，或者可以通过为每个类使用单独的 `DependsOn` 行来指定多个依赖类。当某个类使用在另一个类中声明的结构体或枚举时，这非常重要，因为编译器仅知道它已编译了类中的哪些部分。|
| `Deprecated` |此类已弃用，序列化时将不保存此类的对象。此说明符由子类继承。|
| `DontAutoCollapseCategories=(Category, Category, ...)` |使列出的类别的继承自父类的 `AutoCollapseCategories` 说明符无效。|
| `DontCollapseCategories` |使继承自基类的 `CollapseCatogories` 说明符无效。|
| `EditInlineNew` |指示可以从虚幻编辑器"属性（Property）"窗口创建此类的对象，而非从现有资源引用。默认行为是仅可通过"属性（Property）"窗口指定对现有对象的引用。此说明符会传播到所有子类；子类可通过 `NotEditInlineNew` 说明符覆盖它。|
| `HideCategories=(Category1, Category2, ...)` |列出一个或多个应该对用户完全隐藏的分类。要隐藏未使用类别声明的属性，请使用声明变量的类的名称。此说明符会传播到子类。|
| `HideDropdown` |阻止此类在属性窗口组合框中显示。|
| `HideFunctions=(Category1, Category2, ...)` |让指定分类中的所有函数都对用户完全隐藏。|
| `HideFunctions=FunctionName` |将提到的函数对用户完全隐藏。|
| `Intrinsic` |指示此类直接在 C++中声明，无 **Unreal Header Tool** 生成的样板。请勿在新类上使用此说明符。|
| `MinimalAPI` |导致仅导出此类的类型信息，以供其他模块使用。可以以此类为目标进行强制转换，但此类的函数无法被调用（除了使用内联方法）。这可以缩短编译时间，因为没有针对无需从其他模块访问其所有函数的类导出一切。|
| `NoExport` |指示此类的声明不应包含在由标头生成器自动生成的 C++头文件中。必须在单独的头文件中手动定义该 C++类声明。仅对本地类有效。请勿对新类使用此说明符。|
| `NonTransient` |使继承自基类的 `Transient` 说明符无效。|
| `NotBlueprintable` |指定此类不是可用于创建蓝图的可接受基类。此为默认说明符，将由子类继承。|
| `NotPlaceable` |使继承自基类的 `Placeable` 说明符无效。指示不可以在编辑器中将此类的对象放置到关卡、UI 场景或蓝图中。|
| `PerObjectConfig` |此类的配置信息将按对象存储，在 ``.ini`文件中，每个对象都有一个分段，根据对象命名，格式为`` [ObjectName ClassName] `。此说明符会传播到子类。|
| `Placeable` |指示可在编辑器中创建此类，而且可将此类放置到关卡、UI 场景或蓝图（取决于类类型）中。此标志会传播到所有子类；子类可使用 `NotPlaceable` 说明符覆盖此标志。|
| `ShowCategories=(Category1, Category2, ...)` |使列出的类别的继承自基类的 `HideCategories` 说明符无效。|
| `ShowFunctions=(Category1, Category2, ...)` |在属性查看器中显示列出的类别中的所有函数。|
| `ShowFunctions=FunctionName` |在属性查看器中显示指定的函数。|
| `Transient` |从不将属于此类的对象保存到磁盘。当与播放器或窗口等本质上不持久的特定种类的原生类配合使用时，它非常有用。此说明符会传播到子类，但是可由 `NonTransient` 说明符覆盖。|
| `Within=OuterClassName` |此类的对象无法在 `OuterClassName` 对象的实例之外存在。这意味着，要创建此类的对象，需要提供 `OuterClassName` 的一个实例作为其 `Outer` 对象。|

## 3 TSubclassOf


**`TSubclassOf`** 是提供 `UClass` 类型安全性的模板类。它**在分配从特定类型派生出来的类时很有效**。

- 例如，你可以把这个变量公开给蓝图，设计者可以为玩家角色指定生成的武器类别。
- 例如您在创建一个投射物类，允许设计者指定伤害类型。您可只创建一个 `UClass` 类型的 `UPROPERTY`，让设计者指定派生自 `UDamageType` 的类；或者您可使用 `TSubclassOf` 模板强制要求此选择。以下示例代码展示了不同之处：

```c++
/** type of damage */
UPROPERTY(EditDefaultsOnly, Category=Damage)
UClass* DamageType;
```

Vs.

```c++
/** type of damage */
UPROPERTY(EditDefaultsOnly, Category=Damage)
TSubclassOf<UDamageType> DamageType;
```

在第二个声明中，模板类告知编辑器的属性窗口只列出派生自 UDamageType 的类（作为属性选择）。在第一个声明中可选择任意 UClass。下图对此进行了说明。

![[0373df02883ba7ef2855694b2d997626_MD5.jpg]]
>策略游戏投射物蓝图的范例

除 `UPROPERTY` 安全外，您还能获得 C++ 层级上的类型安全。**如尝试进行不兼容 `TSubclassOf` 类型的相互指定，将出现编译错误。** 
尝试指定泛型 `UClass` 时，它将执行一个运行时检查，以确定它可执行指定。如运行时检查失败，结果数值为 nullptr。

```c++
UClass* ClassA = UDamageType::StaticClass();

TSubclassOf<UDamageType> ClassB;

ClassB = ClassA; // Performs a runtime check

TSubclassOf<UDamageType_Lava> ClassC;

ClassB = ClassC; // Performs a compile time check
```


## 4 UObject 创建

**`UObjects` 不支持构造器参数**。所有的 C++ `UObject` 都会在引擎启动的时候初始化，然后引擎会调用其默认构造器。如果没有默认的构造器，那么 `UObject` 将不会编译。

**`UObject` 构造器应该轻量化，仅用于设置默认的数值和子对象，构造时不应该调用其它功能和函数。** 对于 [Actor](https://docs.unrealengine.com/5.2/zh-CN/actors-in-unreal-engine) 和 [Actor组件](https://docs.unrealengine.com/5.2/zh-CN/components-in-unreal-engine)，初始化功能应该输入 **`BeginPlay()`** 方法。

`UObject` 应该仅在运行时使用 `NewObject` 构建，或者将 `CreateDefaultSubobject` 用于构造器。

|方法|描述|
|---|---|
|[`NewObject<class>`](https://docs.unrealengine.com/en-US/API/Runtime/CoreUObject/UObject/NewObject "NewObject")|使用所有可用创建选项的可选参数创建一个新实例。提供极高的灵活性，包括带自动生成命名的简单使用案例。|
|`CreateDefaultSubobject<class>`|创建一个组件或者子对象，可以提供创建子类和返回父类的方法。<br><br> 创建默认子对象时，由于它们在引擎启动时构造，UObject 的类构造器应该仅适用于本地数据或者本地加载的静态资产。|

> [!warning]
> **`UObjects` 永远都不应使用 `new` 运算符。所有的 UObjects 都由虚幻引擎管理内存和垃圾回收。如果通过 new 或者 delete 手动管理内存，可能会导致内存出错。**

## 4 虚幻头文件工具 UHT

为利用 `UObject` 派生类型所提供的功能，需要在头文件上为这些类型执行一个预处理步骤，以核对需要的信息。该预处理步骤由 **UnrealHeaderTool（简称 UHT）** 执行。`UObject` 派生的类型需要遵守特定的结构。

## 5 头文件格式

`UObject` 在源（.cpp）文件中的实现与其他 C++ 类相似，其**在头（.h）文件中的定义必须遵守特定的基础结构**，以便在虚幻引擎 4 中正常使用。使用编辑器的"New C++ Class"命令是设置格式正确头文件的最简单方法。
`UObject` 派生类的基础头文件可能看起来与此相似，假定 UObject 派生物被称为 UMyObject，其创建时所在的项目被称为 MyProject：

```c++
#pragma once

#include 'Object.h'
#include 'MyObject.generated.h'

UCLASS()
class MYPROJECT_API UMyObject : public UObject
{
    GENERATED_BODY()

};
```

虚幻引擎特定的部分如下：

1. 此行预计为此文件中最后一个 `#include` 指令。如此头文件需要了解其他类，可将它们在文件中的任意位置提前声明，或包括在 MyObject.generated.h 上。

```c++
#include "MyObject.generated.h"
```

2.  `UCLASS` 宏使虚幻引擎能识别 `UMyObject`。此宏支持大量参数[类说明符](https://docs.unrealengine.com/5.2/zh-CN/class-specifiers)，参数决定类功能的开或关。

```c++
UCLASS()
```

3. **如 MyProject 希望将 UMyObject 类公开到其他模块，则需要指定 `MYPROJECT_API`**。这对游戏项目将包括的模块或插件十分实用。这些模块和插件将故意使类公开，在多个项目间提供可携的自含式功能。

```c++
class MYPROJECT_API UMyObject : public UObject
```

4. **`GENERATED_BODY` 宏不获取参数，但会对类进行设置，以支持引擎要求的基础结构。所有 `UCLASS` 和 `USTRUCT` 均有此要求。**
```c++
GENERATED_BODY() //GENERATED_BODY()宏必须被放置在类体的最前方。
```

5. 虚幻头文件工具支持最下 C++集。当使用自定义 `#ifdefs` 宏包裹 UCLASS 的部分时，UHT 会忽略不包含 `WITH_EDITOR` 或者 `WITHEDITORONLY_DATA` 宏的宏。

## 6 更新对象

Ticking 代表虚幻引擎中对象的更新方式。所有Actors均可在每帧被 tick，便于您执行必要的更新计算或操作。

-  `Actor` 和 Actor 组件在注册时会自动调用它们的 Tick 函数
- **`UObjects` 不具有嵌入的更新能力。在必须的时候，可以使用 `inherits` 类说明符从 `FTickableGameObject` 继承即可添加此能力。**  这样即可实现 `Tick()` 函数，引擎每帧都将调用此函数。


## 7 销毁对象

**对象不被引用后，垃圾回收系统将自动进行对象销毁。** 这意味着没有任何 `UPROPERTY` 指针、引擎容器、`TStrongObjectPtr` 或类实例会拥有任何对它的强引用。

注意，无论对象是否被垃圾回收，[弱指针](https://docs.unrealengine.com/5.2/zh-CN/weak-pointers-in-unreal-engine)对其都没有影响。

垃圾回收器运行时，寻找到的未引用对象将被从内存中移除。此外，函数`MarkPendingKill() `可在对象上直接调用。此函数将把指向对象的所有指针设为` NULL`，并从全局搜索中移除对象。对象将在下一次垃圾回收过程中被完全删除。

**[智能指针](https://docs.unrealengine.com/5.2/zh-CN/smart-pointers-in-unreal-engine)不适用于 UObject。**

- `Object->MarkPendingKill()` 被 `Obj->MarkAsGarbage()` 所替代。这个新的函数现在仅用于追踪旧对象。如果 `gc.PendingKillEnabled=true` ，那么所有标记为 `PendingKill` 的对象会被垃圾回收器自动清空并销毁。
    
- **强引用会将 UObject 保留。如果你不想让这些引用保留 UObject，那么这些引用应该转换来使用弱指针，或者变为一个普通指针由程序员手动清除（如果对性能有较高要求的话）。**

你可以用弱指针替换强指针，并且在游戏运作时作为垃圾回收取消引用，因为垃圾回收仅在帧之间运行。

- `IsValid()` 用于检查它是 null 还是垃圾，但是大部分情况下 IsValid 可以被更正规的编程规则替换，比如在调用 `OnDestroy` 事件时将指针清除至 Actor。
- 如果禁用了 `PendingKill()`， `MarkGarbage()` 将会提醒对象的所有者该对象将要被销毁，但是对象本身直到所有对它的引用都解除之后才会被垃圾回收。
- **对于 Actor，即使 Actor 被调用了 `Destroy()`，并且被从关卡中移除，它还是会等到所有对它的引用都解除之后才会被垃圾回收。**
- 对于证书持有者的主要区别在于，对花费较大的对象进行垃圾回收的函数 `MarkPendingKill()` 不再起效。
- 已有的用于 nullptr 的检查应该被 `IsValid()` 调用所替代，除非你进行手动清除，因为指针不再会被垃圾回收器通过 `MarkPendingKill()` 自动清除。


# 三、结构体 USTRUCT
**结构体（Struct）** 是一种数据结构，帮助你组织和操作相关属性。在虚幻引擎中，结构体会被引擎的反射系统识别为 **`USTRUCT`**，但**不属于 [UObject](https://docs.unrealengine.com/5.2/zh-CN/objects-in-unreal-engine) 生态圈,且不能在[UClasses]( https://docs.unrealengine.com/en-US/API/Runtime/CoreUObject/UObject/UClass "UClass")的内部使用。**

- 在相同的数据布局下， `USTRUCT` 比 `UObject` 能更快创建。
- `USTRUCT`支持`UPROPERTY`, 但它不由垃圾回收系统管理，不能提供`UFUNCTION`

## 实现USTRUCT

要把一个结构体变成 `USTRUCT`，请遵循以下步骤：

1. 打开你要定义结构体的 **header (.h)** 文件。
2. 要定义你的C++结构体，请将 `USTRUCT` 宏放在结构体定义的上方。
3. 将 `GENERATED_BODY()` 宏作为定义的第一行。

其结果应该与下面的的例子一致：

```c++
USTRUCT([Specifier, Specifier, ...])
struct FStructName
{
    GENERATED_BODY()
};
```

你可以用`UPROPERTY`来标记结构体的相关变量，使它们在虚幻反射系统和蓝图脚本中可见。

## 结构体说明符

**结构体说明符** 提供元数据，控制你的结构在引擎和编辑器中各方面的表现。

-  `Atomic` ：表示该结构体应始终被序列化为一个单元。将不会为该类创建自动生成的代码。标头仅用于解析元数据。
-  `BlueprintType`： 将此结构体作为一种类型公开，可用于蓝图中的变量。
-  `NoExport` ：将不会为该类创建自动生成的代码。标头仅用于解析元数据。

## 最佳做法与技巧

下面是一些使用 `USTRUCT` 时需要记住的有用提示：

1. `USTRUCT` 可以使用虚幻引擎的[智能指针](https://docs.unrealengine.com/5.2/zh-CN/smart-pointers-in-unreal-engine)和垃圾回收系统来防止垃圾回收删除 `UObjects`。
2. 结构体最好用于简单数据类型。对于你的项目中更复杂的交互，也许可以使用 `UObject` 或 `AActor` 子类来代替。
3. `USTRUCTs` **不可以** 用于复制。但是 `UPROPERTY` 变量 **可以** 用于复制。
4. 虚幻引擎可以自动为结构体创建Make和Break函数。
    1. Make函数出现在任何带有 `BlueprintType` 标签的 `USTRUCT` 中。
    2. 如果在USTRUCT中至少有一个 `BlueprintReadOnly` 或 `BlueprintReadWrite` 属性，Break函数就会出现。
    3. Break函数创建的纯节点为每个标记为 `BlueprintReadOnly` 或 `BlueprintReadWrite` 的资产提供一个输出引脚。
# 四、接口 UINTERFACE
接口类用于确保一组可能不相关的类实现一组公共的函数。在一些游戏功能可能由原本不相似的大型复杂类共享的情况下，这很有用。

游戏可能有这样一个系统，玩家角色进入触发器体积时可以激活陷阱、提醒敌人或向玩家奖励积分。这可以通过陷阱、敌人或积分奖励上的"ReactToTrigger"函数来实现。但是，陷阱可能派生自[AActor](https://docs.unrealengine.com/5.2/zh-CN/actors-in-unreal-engine)，敌人派生自专门的[APawn](https://docs.unrealengine.com/5.2/zh-CN/pawn-in-unreal-engine)或[ACharacter](https://docs.unrealengine.com/5.2/zh-CN/characters-in-unreal-engine)子类，而积分奖励派生自 `UDataAsset` 。

所有这些类都需要共享的功能，但除了 `UObject` 之外，没有其他公共的父类。在这种情况下，推荐使用接口。

## 1 接口声明

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

```c++ file:ReactToTriggerInterface.h
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


## 2 接口说明符

接口说明符用于向虚幻反射系统公开你的类，详见下表：

|接口说明符|含义|
|---|---|
|⭐ **`Blueprintable`**|⭐将该类公开为可接受的用于创建蓝图的基类，让蓝图实现此接口|
|⭐ **`BlueprintType`** |⭐将该类公开为可用于蓝图中的变量的类型。|
|`DependsOn=(ClassName1, ClassName2, ...)` |所有列出的类都将在该类之前编译。ClassName必须在同一个（或上一个）包中指定一个类。多个依赖性类可以使用以逗号分隔的单个"DependsOn"行来指定，也可以使用单个"DependsOn"行为每个类指定。当一个类使用在另一个类中声明的结构体或枚举时，这一点非常重要，因为编译器只知道它已经编译的类中的内容。|
|`MinimalAPI`|仅导致该类的类型信息被导出以供其他模块使用。你可以向该类转换，但不能调用该类的函数（内联方法除外）。对于不需要其所有函数在其他模块中均可供访问的类，通过不导出这些类的所有内容，这可以缩短编译时间。|

## 3 在 C++中实现接口

若要在一个新的类中使用你的接口，只需从前缀为 `I` 的接口类继承（除了你正在使用的任何基于 `UObject`的类）即可。

```c++ file:Trap.h
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

## 4 声明接口函数

有几种方法可以在接口中声明函数，由环境决定能够实现或调用哪种方法。
所有方法都必须在前缀为 `I` 的类中为接口声明，而且必须为 `public` ，以便对外部的类可见。

### 仅C++的接口函数

可以在接口的头文件中声明一个**不带 `UFUNCTION` 说明**的虚函数。这些函数必须为 virtual，以便在实现接口的类中重载它们。

```c++ file:ReactToTrigger.h
public:
virtual bool ReactToTrigger();
```

然后，可以在头文件本身或接口的 `.cpp` 文件中提供一个默认的实现。

```c++ file:ReactToTrigger.cpp
bool IReactToTriggerInterface::ReactToTrigger()
{
    return false;
}
```

当在一个Actor类中实现接口后，可以创建并实现一个针对该类的重载。

```c++ file:Trap.h
public:
virtual bool ReactToTrigger() override;
```


```c++ file:Trap.cpp
    bool ATrap::ReactToTrigger()
    {
        return false;
    }
```

**但是，这些C++接口函数对蓝图不可见。**

### 蓝图可调用接口函数

**要创建蓝图可调用的接口函数，必须在带 `BlueprintCallable` 说明符的函数声明中提供一个 `UFUNCTION` 宏。还必须使用 `BlueprintImplementableEvent` 或 `BlueprintNativeEvent` 说明，而且函数不能为virtual。**

```c++ file:ReactToTrigger.h
public:
/**只能在蓝图中实现的React To Trigger版本。*/
UFUNCTION(BlueprintCallable, BlueprintImplementableEvent, Category=Trigger Reaction)
bool ReactToTrigger();
```


```c++ file:ReactToTrigger.h
public:
/**可以在C++或蓝图中实现的React To Trigger版本。*/
UFUNCTION(BlueprintCallable, BlueprintNativeEvent, Category=Trigger Reaction)
bool ReactToTrigger();
```

`BlueprintCallable`：**引用实现接口的对象的 C++或蓝图**可以调用使用 `BlueprintCallable` 说明符的函数。

`BlueprintImplementableEvent`：使用 `BlueprintImplementableEvent` 的函数**不能在 C++中被重载，但可以在任何实现或继承接口的蓝图类中被重载。**

`BlueprintNativeEvent`：在 C++中，可通过重载一个同名函数来实现使用 `BlueprintNativeEvent` 的函数，但要在末尾添加上后缀 `_Implementation` 。该说明符还**允许在蓝图中重载实现**。

```c++ file:Trap.h
public:
bool ReactToTrigger_Implementation() override;
```

```c++ file:Trap.cpp
bool ATrap::ReactToTrigger_Implementation() const
{
    return false;
}
```


## 5 确定类是否实现了接口

为了与实现接口的C++和蓝图类兼容，可以使用以下任意函数：

```c++
bool bIsImplemented = OriginalObject->GetClass()->ImplementsInterface(UReactToTriggerInterface::StaticClass()); // 如果OriginalObject实现了UReactToTriggerInterface，则bisimplemated将为true。

bIsImplemented = OriginalObject->Implements<UReactToTriggerInterface>(); // 如果OriginalObject实现了UReactToTrigger，bIsImplemented将为true。

IReactToTriggerInterface* ReactingObjectA = Cast<IReactToTriggerInterface>(OriginalObject); // 如果OriginalObject实现了UReactToTriggerInterface，则ReactingObject将为非空。
```

如果 **`StaticClass()`** 函数在前缀为 `I` 的类中没有实现，尝试在前缀为 `U` 的类上使用 `Cast` 将失败，代码将无法编译。

## 6 转换类型

虚幻引擎的转换系统支持**从一个接口转换到另一个接口**，或者在适当的情况下，**从一个接口转换到一个虚幻类型**。

```c++
IReactToTriggerInterface* ReactingObject = Cast<IReactToTriggerInterface>(OriginalObject); // 如果接口被实现，则ReactingObject将为非空。

ISomeOtherInterface* DifferentInterface = Cast<ISomeOtherInterface>(ReactingObject); // 如果ReactingObject为非空而且还实现了ISomeOtherInterface，则DifferentInterface将为非空。

AActor* Actor = Cast<AActor>(ReactingObject); // 如果ReactingObject为非空且OriginalObject为AActor或AActor派生的类，则Actor将为非空。
```

# 五、函数 UFUNCTION

**`UFUNCTION`** 是一种 C++函数，可以被虚幻引擎（UE）反射系统识别。 `UObject` 或蓝图函数库可将**成员函数**声明为 `UFUNCTION`，方法是将 `UFUNCTION` 宏放在头文件中函数声明上方的行中。

宏将支持 **函数说明符** 更改虚幻引擎解译和使用函数的方式。

```c++
UFUNCTION([specifier1=setting1, specifier2, ...], [meta(key1="value1", key2, ...)])
ReturnType FunctionName([Parameter1, Parameter2, ..., ParameterN1=DefaultValueN1, ParameterN2=DefaultValueN2]) [const];
```

可利用函数说明符将`UFUNCTION`对蓝图公开，以便开发者从蓝图资源调用或扩展`UFUNCTION`，而无需更改C++代码。

## 函数说明符

声明函数时，可以为声明添加 **函数说明符**，以控制函数相对于引擎和编辑器的各个方面的行为方式。

**常用：**
1. **`BlueprintCallable`** ：函数可从**蓝图**中调用，但**只能通过 C++代码修改**。
    - 码将更改它所调用的对象的某些内容或一些其他全局状态，意味着必须调用它，必须连接白色执行线（调度它）。
2. **`BlueprintPure`**：函数可从**蓝图**中调用，但**只能通过 C++代码修改**。
    - 不会更改它所调用的对象的任何内容或任何其他全局状态，它只需要输入，然后告诉您输出，类似于数学运算节点，不需要连接白色执行线。
3. **`BlueprintImplementableEvent`**：函数由其**蓝图**的子类实现，不应该在 C++中给出函数的实现，这会导致链接错误。
    - 如果没有返回值或输出参数，那么在蓝图中为一个事件
    - 如果它有返回值或任何输出参数，那么在蓝图中为一个函数
4. **`BlueprintNativeEvent`** ：函数提供一个 C++的默认实现，同时也可以被蓝图重载。默认实现的函数名为 `函数名_Implementation` 。
    - 如果需要，您仍然可以调用默认实现，方法是在事件或函数条目节点上 **右键单击**，并选择"将调用添加到父项（Add call to parent）"。
```c++
//.h中声明
UFUNCTION(BlueprintNativeEvent)
void TestFunction(); //由蓝图重载
virtual void TestFunction_Implementation();  //由cpp实现

//.cpp中实现
//只需要实现名为_Implementation的默认实现
void ATestActor::TestFunction_Implementation()  
{  
    //功能...
}

//.cpp中调用
//仍旧使用原版本，如果该函数没有被蓝图重载，则自动生成代码调用TestFunction_Implementation()
TestFunction(); 
```

|函数说明符|效果|
|---|---|
|`BlueprintAuthorityOnly` |如果在具有网络权限的机器上运行（服务器、专用服务器或单人游戏），此函数将仅从蓝图代码执行。|
|⭐ **`BlueprintCallable`**|⭐**此函数可在蓝图或关卡蓝图图表中执行。**|
|`BlueprintCosmetic`|此函数为修饰性的，无法在专用服务器上运行。 |
|⭐ **`BlueprintImplementableEvent`**|⭐**此函数可在蓝图或关卡蓝图图表中实现。**|
|⭐ **`BlueprintNativeEvent`**|⭐**此函数旨在被蓝图覆盖掉，但是也具有默认原生实现。** 用于声明名称与主函数相同的附加函数，但是末尾添加了`_Implementation`，是写入代码的位置。如果未找到任何蓝图覆盖，该自动生成的代码将调用`_ Implementation` 方法。 |
|`BlueprintPure` |此函数不对拥有它的对象产生任何影响，可在蓝图或关卡蓝图图表中执行。|
|⭐ **`CallInEditor`**|⭐**可通过细节（Details）面板中的按钮在编辑器中的选定实例上调用此函数。** |
|⭐ **`Category = "TopCategory\|SubCategory\|Etc"`**|⭐**在蓝图编辑工具中显示时指定函数的类别**。使用 \| 运算符定义嵌套类别。|
|`Client`|此函数仅在拥有在其上调用此函数的对象的客户端上执行。用于声明名称与主函数相同的附加函数，但是末尾添加了`_Implementation`。必要时，此自动生成的代码将调用`_ Implementation` 方法。|
|`CustomThunk`|`UnrealHeaderTool` 代码生成器将不为此函数生成thunk，用户需要自己通过 `DECLARE_FUNCTION` 或 `DEFINE_FUNCTION` 宏来提供thunk。|
|`Exec`|此函数可从游戏内控制台执行。仅在特定类中声明时，Exec命令才有效。|
|`NetMulticast`|此函数将在服务器上本地执行，也将复制到所有客户端上，无论该Actor的 `NetOwner` 为何。|
|`Reliable`|此函数将通过网络复制，并且一定会到达，即使出现带宽或网络错误。仅在与`Client`或`Server`配合使用时才有效。|
|`SealedEvent`|无法在子类中覆盖此函数。``SealedEvent`关键词只能用于事件。对于非事件函数，请将它们声明为`static`或`final``，以密封它们。|
|`ServiceRequest`|此函数为RPC（远程过程调用）服务请求。这意味着 `NetMulticast` 和 `Reliable`。|
|`ServiceResponse`|此函数为RPC服务响应。这意味着 `NetMulticast` 和 `Reliable`。|
|`Server`|此函数仅在服务器上执行。用于声明名称与主函数相同的附加函数，但是末尾添加了 `_Implementation`，是写入代码的位置。必要时，此自动生成的代码将调用 `_Implementation` 方法。|
|`Unreliable`|此函数将通过网络复制，但是可能会因带宽限制或网络错误而失败。仅在与`Client`或`Server`配合使用时才有效。|
|`WithValidation`|用于声明名称与主函数相同的附加函数，但是末尾需要添加`_Validate`。此函数使用相同的参数，但是会返回`bool`，以指示是否应继续调用主函数。|

## 函数参数说明符

|参数说明符|描述|
|---|---|
|Out|声明由引用传递的参数，使函数对其进行修改。|
|Optional|通过任选关键词可使部分函数参数变为任选，便于调用。任选参数的数值（调用方未指定）取决于函数。例如， `SpawnActor` 函数使用任选位置和旋转，默认为生成的 Actor 根组件的位置和旋转。添加 `= [value]` 参数可指定任选参数的默认值。例如： `function myFunc(optional int x = -1)` 。在多数情况下，如无数值被传递到任选参数，将使用变量类型的默认值或零（例如 0、false、""、none）。|
# 六、参数 UPARAM
主要用于将 C++ 代码公开到蓝图

若要使参数通过**引用传递并仍然显示为输入**，请使用 `UPARAM()` 宏。

![[Pasted image 20230829220832.png]]

```c++
UFUNCTION(BlueprintCallable, Category = "Example Nodes")
static void HandleTargets(UPARAM(ref) TArray<FVector>& InputLocations, TArray<FVector>& OutputLocations);
```

您还可以使用`UPARAM()`来**更改引脚的显示名称**。例如， MakeRotator 函数使用"`UPARAM()`和`DisplayName`关键字来更改绕 Z 轴的旋转、绕 X 轴的旋转、绕 Y 轴的旋转参数在蓝图中的显示方式。

![[54948c4fcd6856e0a406abb1d203171e_MD5.jpg]]

```c++
/** 根据采用度数的旋转值制作一个旋转体{绕Z轴的旋转、绕X轴的旋转、绕Y轴的旋转} */
UFUNCTION(BlueprintPure, Category="Math|Rotator", meta=(Keywords="construct build rotation rotate rotator makerotator", NativeMakeFunc))
static FRotator MakeRotator(
UPARAM(DisplayName="X (Roll)") float Roll,
UPARAM(DisplayName="Y (Pitch)") float Pitch,
UPARAM(DisplayName="Z (Yaw)") float Yaw);
```

# 七、元数据说明符
文档：
[虚幻引擎元数据说明符 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/metadata-specifiers-in-unreal-engine/)

声明类、接口、结构体、列举、列举值、函数，或属性时，可添加 **元数据说明符** 来**控制其与引擎和编辑器各方面的交互方式**。
每一种类型的数据结构或成员都有自己的元数据说明符列表。

> [!warning] 
> Metadata 只存在于编辑器中。请不要编写能够访问到 Metadata 的游戏逻辑。

**要添加元数据说明符，需使用单词 `meta`，后接说明符列表**。如有必要，可以将它们的值添加到 `UCLASS`、`UENUM`、`UINTERFACE`、`USTRUCT`、`UFUNCTION` 或 `UPROPERTY` 宏，如下所示：

```c++
{UCLASS/UENUM/UINTERFACE/USTRUCT/UFUNCTION/UPROPERTY}(SpecifierX, meta=(MetaTag1="Value1", MetaTag2, ..), SpecifierY)
```

**要添加元数据说明符到列举值，可将 `UMETA` 标签添加到值本身。如果存在用于分隔的逗号，则要添加到逗号之前，如下所示：**

```c++
UENUM()
enum class EMyEnum : uint8
{
    // DefaultValue Tooltip
    DefaultValue = 0 UMETA(MetaTag1="Value1", MetaTag2, ..),

    // ValueWithoutMetaSpecifiers Tooltip
    ValueWithoutMetaSpecifiers,

    // ValueWithMetaSpecifiers Tooltip
    ValueWithMetaSpecifiers UMETA((MetaTag1="Value1", MetaTag2, ..),

    // FinalValue Tooltip
    FinalValue (MetaTag1="Value1", MetaTag2, ..)
};
```

# 八、智能指针
## 概述
UE 智能指针不能用于与 `UObject` 及其派生类不兼容。**常用于 `Uobject` 系统外的数据对象（比如自己创建的类）** 
引擎具有 `UObject` 管理的单独内存管理系统（[对象处理](https://docs.unrealengine.com/5.2/zh-CN/unreal-object-handling-in-unreal-engine)文档），两个系统未互相重叠。

### 智能指针类型

智能指针可影响其包含或引用对象的寿命。不同智能指针对对象有不同的限制和影响。下表可用于协助决定各类型智能指针的适用情况：

共享指针：最常用的指针, 用来存储。

共享引用
- 共享引用永远不会为 null，且总是可以进行解引用。
- 共享指针的性能消耗最小。所有操作所占时间都是固定的。

弱指针
1. 弱指针允许您安全地检查一个对象是否已经被销毁。
2. 使用弱引用来断开引用循环。
缺点: 最慢

### 智能指针优点
**为什么原则使用 UE 智能指针：**
1. C++原生智能指针不是在所有平台上都可用。UE 智能指针可以跨平台。
2. 可以和其他虚客器及类型无缝地协作。
3. 更好地控制平台特性、包括线程处理和优化。
4. 我们想提供线程安全的功能以获得好的性能。
5. 我们想在性能方面有更多的控制权 (内联函数、内存、虚函数的应用等)。
6. 在不需要的时候倾向于不引入新的第三方依赖。


|优点|描述|
|---|---|
|**防止内存泄漏**|共享引用不存在时，智能指针（弱指针除外）会自动删除对象。|
|**弱引用**|弱指针会中断引用循环并阻止悬挂指针。|
|**可选择的线程安全**）|虚幻智能指针库包括线程安全代码，可跨线程管理引用计数。如无需线程安全，可用其换取更好性能。|
|**运行时安全**|共享引用从不为空，可固定随时取消引用。|
|**授予意图**|可轻松区分对象所有者和观察者。|
|**内存**|智能指针在 64 位下仅为 C++指针大小的两倍（加上共享的 16 字节引用控制器）。唯一指针除外，其与 C++指针大小相同。|

**在性能上的优势**
1. 所有运算均为常量时间。
2. 共享指针解引用的速度和 C+指针一样快。
3. 复制共享指针永远不会分配内存。
4. 线程安全的版本是无锁的。
5. 和 STL 相比，其实现更快。

**劣势**
1. 创建和复制智能指针比创建和复制原始 C++指针需要更多开销
2. 保持引用计数增加基本运算的周期
3. 共享指针使用的内存比 C++指针多
4. 引用控制器有两个堆分配。使用 `MakeShared` 代替 `MakeShareable` 可避免二次分配，并可提高性能。
5. 由多个共享指针引用的每个独立对象都有性能消耗。
6.  弱指针访问速度比共享指针访问速度略慢。

**内存使用情况（在 32 位操作系统）**
1. 所有的共享指针 (TSharedPtr，TSharedRef，TWeakPtr)都占 8 个字节当针对 32 一位系统编译时)
2.  C 指针 (无符号 32 位整型)
3. 引用控制器指针无符号 32 位整型)
4. TSharedFromThis 也占 8 个字节, 因为它内嵌了弱指针/

**引用控制器（当针对 32-位系统编译时）**
1. 引用控制器对象占 12 个字节
2.  C+指针无符号 32 位整型)
3. 共享引用计数 (无符号 32 位整型)
4. 弱引]用计数 (无符号 32 位整型)
注意: 无论有多少个共享指针/弱指针引用一个对象，都仅为每个对象创建一个引用控制器。.



##  TSharedPtr

> [!NOTE] 共享指针（强指针）
> **共享指针（Shared Pointers）** 是指既健壮、又能为空指针的智能指针。**共享指针沿袭了普通智能指针的所有优点**，它能避免出现内存泄漏、悬挂指针，还能避免指针指向未初始化的内存。
> 
> **还有一些其他特点**：
> - **共享所有权（Shared Ownership）：** 引用计数支持多个共享指针，以确保它们引用的数据对象永远不被删除，前提是它们中的任意一个仍指向数据对象。
> - **自动失效（Automatic Invalidation）：** 你可安全引用易变对象，无需担心出现悬挂指针。
> - **弱引用：** 弱指针可中断引用循环。
> - **意向指示（Indication of Intent）：** 区分拥有者（参见[共享引用](https://docs.unrealengine.com/5.2/zh-CN/shared-references-in-unreal-engine)）和观察者，并提供不可为空的引用。
>     
> **共享指针有一些值得注意的基本特性，包括：**
> - 语法非常健壮
> - 非侵入性（但能反射）
> - 线程安全（视情况而定）
> - 性能佳，占用内存少
> 
> > [!NOTE] 非侵入性
> **共享指针是非侵入性的，即对象不知道其是否为智能指针拥有**。
> 


**共享指针类似于共享引用，<font color="#ff0000">主要区别在于共享指针可以指向空对象，共享引用不可为空</font>。**
除非需要空对象或可为空的对象，否则建议你**优先选择共享引用**。

### 1 声明和初始化

**共享指针可为空**，所以无论有无数据对象，都可以对它们进行初始化。

- **`MakeShared<T>() / MakeShareable() ` ：创建共享指针
- MakeShared：在单个内存块中分配新的对象实例和引用控制器，但要求对象提供公共构造函数。
- MakeShareable：效率较低，但即使对象的构造函数是私有的也可以工作，使您能够获得不是您创建的对象的所有权，并在删除对象时支持自定义行为。
- 注意**只能用于动态分配内存！`new`**
```c++
// 创建空白的共享指针
TSharedPtr<FMyObjectType> EmptyPointer;

// 为新对象创建共享指针
TSharedPtr<FMyObjectType> NewPointer(new FMyObjectType()); 
TSharedPtr<FMyObjectType> NewPointer = MakeShareable(new FMyObjectType());

// 从共享引用创建共享指针
TSharedRef<FMyObjectType> NewReference(new FMyObjectType());
TSharedPtr<FMyObjectType> PointerFromReference = NewReference;

// 创建线程安全的共享指针
TSharedPtr<FMyObjectType, ESPMode::ThreadSafe> NewThreadsafePointer = MakeShared<FMyObjectType, ESPMode::ThreadSafe>(MyArgs);
```

### 2 复制/转移

**复制共享指针时，系统将向它引用的对象添加一个引用。**

```c++
// 增加对象ExistingSharedPointer引用的引用数。
TSharedPtr<FMyObjectType> AnotherPointer = ExistingSharedPointer;
```


- **使用 `MoveTemp`（或 `MoveTempIfPossible`）函数将一个共享指针的内容转移到另一个共享指针，将原始的共享指针保留为空：**（对应 C++的 std::move）

```c++
// 将PointerOne的内容移至PointerTwo。在此之后，PointerOne将引用nullptr。
PointerTwo = MoveTemp(PointerOne);
// 将PointerTwo的内容移至PointerOne。在此之后，PointerTwo将引用nullptr。
PointerOne = MoveTempIfPossible(PointerTwo);
```

`MoveTemp` 和 `MoveTempIfPossible` 的唯一不同之处在于 `MoveTemp` 包含静态断言，强制其只能在非常量左值（lvalue）上执行。

### 3 重置
**使用 `Reset` 函数、或分配一个空指针来重置共享指针

```c++
PointerOne.Reset();
PointerTwo = nullptr;
// PointerOne和PointerTwo现在都引用nullptr。
```

### 5 比较 / 有效性

`==` `!=`：共享指针是否相等
相等被定义为两个共享指针引用同一对象。

```c++
TSharedPtr<FTreeNode> NodeA, NodeB;
if (NodeA == NodeB)
{
    // ...
}
```

- **`IsValid` 函数（是否为有效的）和 `bool` 运算符有助于判断共享指针是否引用了有效对象。
-  **`Get`**：**返回对象的原生C++指针，若为空返回 null**

```c++
if (Node.IsValid())
{
    // ...
}
if (Node)
{
    // ...
}
if (Node.Get() != nullptr)
{
    // ...
}

//更严格的检查
if (Node.IsValid() && Node.Get())
{
    //...
}
```

### 6 解引用和访问

你可以像使用普通C++指针那样解引用，调用方法和访问成员。你也可以像使用其他C++指针那样，通过调用 **`IsValid`** 函数或使用重载的 `bool` 运算符，在取消引用之前执行空检查。

```c++
// 在解引用前，检查节点是否引用了一个有效对象。
if (Node)
{
    // 以下三行代码中的任意一行都能解引用节点，并且对它的对象调用ListChildren：
    Node->ListChildren();
    Node.Get()->ListChildren();  //推荐
    (*Node).ListChildren();
}
```

## TSharedRef
共享引用（强引用）

- **不可为空**，初始化必须有数据对象
- 无法重置 `Reset` 共享引用、向其指定空对象，或创建空白引用。因此**固定包含有效对象**，甚至**没有 `IsValid` 方法**。
- **与标准的 C++引用不同，可在创建后将共享引用重新指定到另一对象。

**在共享引用和共享指针之间进行选择时，除非需要空对象或可为空的对象，否则建议你优先选择共享引用。**

避免将数据作为 `TSharedRef` 或 `TSharedPtr` 参数传到函数，此操作将因取消引用和引用计数而产生开销。相反，**建议将引用对象作为 `const &` 进行传递。**
### 声明和初始化

- **`MakeShared<T>() / MakeShareable() ` ：创建共享引用

```c++
//创建共享引用
TSharedRef<FMyObjectType> NewReference(new FMyObjectType());
TSharedRef<FMyObjectType> NewReference = MakeShared<FMyObjectType>();
```

在无有效对象的情况下尝试创建的共享引用将不会编译，并尝试将共享引用初始化为空指针变量：

```c++
//以下两者均不会编译：
TSharedRef<FMyObjectType> UnassignedReference; 
TSharedRef<FMyObjectType> NullAssignedReference = nullptr;
//以下会编译，但如NullObject实际为空则断言。
TSharedRef<FMyObjectType> NullAssignedReference = NullObject;
```

### 比较
`==` `!=`：共享引用是否相等
相等表示引用相同对象。
```c++
    TSharedRef<FMyObjectType> ReferenceA, ReferenceB;
    if (ReferenceA == ReferenceB)
    {
        // ...
    }
```

## TWeakPtr


- **弱指针**存储对象的弱引用，不参与引用计数。
- **在访问弱指针引用的对象前，应使用 `Pin` 函数生成共享指针。此操作确保使用该对象时其将继续存在。**
- 如只需要确定弱指针是否引用对象，可将其与 `nullptr` 比较，或在之上调用 `IsValid`。

### 声明、初始化

**可创建空白弱指针，或通过共享指针和共享引用创建

```c++
//创建共享引用。
TSharedRef<FMyObjectType> ObjectOwner = MakeShared<FMyObjectType>();
//创建指向对象的弱指针。
TWeakPtr<FMyObjectType> ObjectObserver(ObjectOwner);
```

### 复制
与共享指针相同，弱指针是否引用有效对象，均可进行安全复制：

```c++
TWeakPtr<FMyObjectType> AnotherObjectObserver = ObjectObserver;
```

### 重置
`Reset ()` 或 `nullptr` 
```c++
//可通过将弱指针设为nullptr进行重置。
ObjectObserver = nullptr;
//也可使用重置函数。
AnotherObjectObserver.Reset();
```

### 有效性

- **`IsValid` 函数（是否为有效的）和 `bool` 运算符有助于判断共享指针是否引用了有效对象。
-  **`Get`**：返回对象指针，若为空返回 null
  
### 转换为共享指针

**`Pin` 函数将创建指向弱指针对象的共享指针。** 

共享指针（包括由 `Pin` 函数返回的指针）可在条件句中作为 `bool` 类型进行求值，其中 `true` 表示有效对象。

以下代码检查弱指针是否引用有效对象。如是，至少在共享指针（由 `Pin` 函数创建）超出范围或被显式清除前，将保证其持续有效。

```c++
    //获取弱指针中的共享指针，并检查其是否引用有效对象。
    if (TSharedPtr<FMyObjectType> LockedObserver = ObjectObserver.Pin())
    {
        //共享指针仅在此范围内有效。
        //该对象已被验证为存在，而共享指针阻止其被删除。
        LockedObserver->SomeFunction();
    }
```

### 访问对象

1. 首先使用 **`Pin`** 函数，将其转换为共享指针。
2. 然后通过共享指针或弱指针上的 **`Get`** 函数进行访问。此方法可确保使用该对象时，其将持续有效。

### 打破引用循环

两个或多个对象使用智能指针保持彼此间的强引用时，将出现引用循环。在此类情况下，对象间会相互保护以免被删除。各对象固定被另一对象引用，因此对象无法在另一对象存在时被删除。如外部对象未对引用循环中对象进行引用，其实际上将出现内存泄漏。

**弱指针不会保留自身引用的对象，因此其可中断此类引用循环。要在未拥有对象时对其进行引用，并延长其寿命时，可使用弱指针。**
### 使用警告
在以下情况中请谨慎使用弱指针：

- **在Set或Map中用作键。弱指针可能会在未通知容器的情况下随时无效，因此共享指针或共享引用更适用于充当键。可安全地将弱指针用作数值。
- 虽然弱指针提供 `IsValid` 函数，但是检查 `IsValid` 无法保证对象在任何时间长度内均可持续有效。线程安全共享指针可能会因另一线程上的活动而随时无效，因此使用线程安全共享指针应尤其注意。`Pin` 返回的共享指针将使对象在代码将其清除或其超出范围前保持活跃状态，**因此 `Pin` 函数是用于检查的首选方法，此类检查会导致取消引用或访问存储对象。**

## TUniquePtr

- 唯一指针仅会显式拥有其引用的对象。
- 唯一指针可转移所有权，但无法共享。复制唯一指针的任何尝试都将导致编译错误。
- 唯一指针超出范围时，其将自动删除其所引用的对象。
- 不要为共享指针或共享引用引用的对象创建唯一指针

### 创建 / 初始化 / 判断 / 解引用 / 重置

*   `MakeUnique ()` 创建唯一指针
*   `IsValid ()`
*   `->` 运算符
*   `Get()` 函数
*   **`Release()`** **释放并移交所有权**
*   `Reset()` 或 `nullptr` 重置
    
```c++
// 创建唯一指针
TUniquePtr<SimpleObject> ObjUniquePtr = MakeUnique<SimpleObject>();

// 判断有效性
if (ObjUniquePtr.IsValid()) 
{
    ObjUniquePtr->ExeFun(); // 解引用
}

// 释放指针，移交
TUniquePtr<SimpleObject> ObjUniquePtr2(ObjUniquePtr.Release());

// 重置
ObjUniquePtr.Reset();		
ObjUniquePtr2 = nullptr;
```
    
## 智能指针类型转换

### 引用转指针
**共享引用隐式转换为共享指针**
```c++
    TSharedPtr<FMyObjectType> MySharedPointer = MySharedReference;
```

### 指针转引用
**`ToSharedRef`**：**从共享指针创建共享引用**
- 要求共享指针引用了一个非空对象
- 从空共享指针创建共享引用将触发断言。

```c++
// 在解引用之前，请确保共享指针有效，以避免可能出现的断言。
if (MySharedPointer.IsValid())
{
    MySharedReference = MySharedPointer.ToSharedRef();
}
```

### 弱指针转强指针
**`Pin` 函数将创建指向弱指针对象的共享指针。** 

```c++
//ObjectObserver为弱指针
TSharedPtr<FMyObjectType> LockedObserver = ObjectObserver.Pin()
```


### 子类转父类
隐式转换
```c++
//SimpleObject是ComplexObject的父类
TSharedPtr<SimpleObject> simpleObj;
TSharedPtr<ComplexObject> complexObj = MakeShared<ComplexObject>();

// 派生类转基类
simpleObj = complexObj;
```

### 父类转子类
`StaticCastSharedRef` 和 `StaticCastSharedPtr` 
```c++
//SimpleObject是ComplexObject的父类
// 基类转派生类
TSharedPtr<ComplexObject> complexObj2 = StaticCastSharedPtr<ComplexObject>(simpleObj);
```

### `const` 转 `mutable`
[[《C++ Primer》#mutable关键字]]：简单理解就是把 const 转换成非const

`ConstCastSharedRef` 和 `ConstCastSharedPtr` ：将 `const` 智能引用或智能指针分别转换为 `mutable` 智能引用或智能指针。

```c++
//创建常量指针
const TSharedPtr<SimpleObject> simpleObj_const(new SimpleObject());
//常量指针转非常量指针
TSharedPtr<SimpleObject> simpleObj_mutable = ConstCastSharedPtr<SimpleObject>(simpleObj_const);
```

## TSharedFromThis 
共享指针是非侵入性的，意味对象不知道其是否为智能指针拥有。
有些函数的参数为共享引用或共享指针，我们就需要传进去一个对象，但**如何让对象知道自己就是智能指针？**
将一个类继承自 `TSharedFromThis` 后，那么这个类的对象就会知道自己是属于哪一个共享指针。

>对标的是原生 C++的 `std::enable_shared_from_this`。用法也非常相似?? 存疑

**`TSharedFromThis` 意思就使用 `this` 指针来构造一个共享指针，通过这个共享指针可以安全的使用 this 指针。** 
其内部有一个弱指针，若要获取类实例的 this 指针，它提供两类接口 **AsShared()** 和 **SharedThis()**，它们会**通过 `TWeakPtr` 返回一个共享引用；

*   自定义类继承 `TSharedFromThis` 模板类
*   `AsShared()` 将 C++原生指针转换为共享引用，如果需要，我们可以再隐式转为共享指针
*   `SharedThis(this)` 会返回具备 "this" 类型的共享引用

```c++
class BaseClass : public TSharedFromThis<BaseClass>
{
    public:
        void printf(){};
}

void Func()
{
    //创建共享指针访问成员函数
    TSharedPtr<BaseClass> sharePtr = MakeShareable(new BaseClass());
    sharePtr->printf();

    //通过.Get()将共享指针解引用, 我们将可以通过智能指针获得原生C++指针
    //原生C++指针ptr指向sharePtr所指的对象
    BaseClass* ptr = sharePtr.Get(); 

    //对于普通的类，我们如果想把ptr在转换为共享指针，需要再次调用MakeShareable创建新的共享指针
    //⭐对于继承了TSharedFromThis的类，类对象知道自己是共享指针
    //因此我们可以直接使用AsShared()将指向BaseClass的C++原生指针ptr转换为共享引用
    //然后隐式转换为共享指针
    if(ptr)
    {
         TSharedPtr<BaseClass> sharePtr2 = ptr->AsShared();    
    }
```

**需要注意的是：**
① **调用 AsShared () 的对象必须是一个智能指针**，否则仍然不能保证使用 this 裸指针或对内存重复释放，在 UE4 中会触发断言；
② 在**类外部调用静态方法 `SharedThis()` 时，当前操作模块的类也必须公有继承其自身的 `TSharedFromThis`**；
③ `AsShared()` 和 `SharedThis()` **不能在构造函数内部使用**，共享引用此时并未初始化，将导致崩溃或断言。

```c++
class MyClass : public TSharedFromThis<MyClass>
{
public:
    TSharedRef<MyClass> SharedMyself()
    {
        return SharedThis(this);
    }
};

// 普通指针或对象，使用TSharedFromThis内的方法会触发断言 
TSharedPtr<MyClass> ptr = MakeShared<MyClass>();

// 通过接口获取类实例的智能引用，维护的是同一块内存，同一个计数器 
TSharedRef<MyClass> pRef1 = ptr->AsShared();
TSharedRef<MyClass> pRef2 = ptr->SharedMyself();

// 在类外部使用该接口，那么操作模块的类也必须继承其自身的TSharedFromThis 
TSharedRef<MyClass> pRef3 = SharedThis(ptr.Get());
```

## 自定义删除器

**共享指针和共享引用支持对它们引用的对象使用自定义删除器**。如需运行自定义删除代码，请**提供 lambda 函数**，作为创建智能指针时使用的参数，就像这样：

```c++
void DestroyMyObjectType(FMyObjectType* ObjectAboutToBeDeleted)
{
    // 此处添加删除代码。
}
// 这些函数使用自定义删除器创建指南指针。
TSharedRef<FMyObjectType> NewReference(new FMyObjectType(), [](FMyObjectType* Obj){ DestroyMyObjectType(Obj); });

TSharedPtr<FMyObjectType> NewPointer(new FMyObjectType(), [](FMyObjectType* Obj){ DestroyMyObjectType(Obj); });
```

## 线程安全

**通常仅在单线程上访问智能指针的操作才是安全的。**

**如需访问多线程，请使用智能指针类的线程安全版本：**

- `TSharedPtr<T, ESPMode::ThreadSafe>`
- `TSharedRef<T, ESPMode::ThreadSafe>`
- `TWeakPtr<T, ESPMode::ThreadSafe>`
- `TSharedFromThis<T, ESPMode::ThreadSafe>`

**由于原子引用计数，此类线程安全版本比默认版本稍慢，但其行为与常规 C++指针一致：**
- 读取和复制固定为线程安全。
- 写入和重置须同步后才安全。

如了解多线程永不访问指针，可通过避免使用线程安全版本获得更好性能。

# 九、容器

## TArray 数组

`TArray` 是UE4中最常用的容器类。其速度快、内存消耗小、安全性高。`TArray` 类型由两大属性定义：元素类型和可选分配器。

**同质容器：所有元素均为完全相同类型**。

**分配器常被省略，默认为最常用的分配器。** 其定义对象在内存中的排列方式；以及数组如何进行扩展，以容纳更多的元素。若默认行为不符合要求，可选取多种不同的分配器，或自行编写。

`Tarray` 为数值类型，**建议在实际操作中勿使用 `new` 和 `delete` 创建或销毁 `TArray` 实例**。
元素也为数值类型，为容器所拥有。`TArray` 被销毁时其中的元素也将被销毁。若在另一 `TArray` 中创建 `TArray` 变量，其元素将复制到新变量中，且不会共享状态。

### 创建和填充数组

如要**创建数组**，将其以此定义：
```c++
    TArray<int32> IntArray; //创建用于存储整数序列的空白数组
```
- 元素类型是可根据普通 C++值规则进行复制和销毁的数值类型，例如 `int32`、`FString`、`TSharedPtr` 等。
- 由于无指定分配器，因此 `TArray` 将**采用基于堆的默认分配器**。
- 此时尚未分配内存。

有多种**填充** `Tarray` 的方法。
-  **`Init`** :将大量元素副本填入数组。
```c++
IntArray.Init(10, 5);
// IntArray == [10,10,10,10,10]
```

-  **`Add`** 和 **`Emplace`** 函数用于在数组末尾新建对象：
```c++
TArray<FString> StrArr;
StrArr.Add    (TEXT("Hello"));
StrArr.Emplace(TEXT("World"));
// StrArr == ["Hello","World"]
```

> [!NOTE] Add 和 Emplace 的选择
> 新元素添加到数组时，数组的分配器将根据需要分配内存。当前数组大小超出时，默认分配器将添加足够的内存，用于存储多个新元素。
> 
> `Add` 和 `Emplace` 函数的多数效果相同，**细微区别**在于：
> - `Add`（或 `Push`）将元素类型的实例**复制（或移动）** 到数组中。
> - `Emplace` 使用给定参数**构建元素类型的新实例**。
> 
> 因此在 `TArray<FString>` 中，`Add` 将用字符串文字创建临时 `FString`，然后将该临时 `FString` 的内容移至容器内的新 `FString` 中；而 `Emplace` 将用字符串文字直接新建 `FString`。最终结果相同，**但 `Emplace` 可避免创建临时文件。对于 `FString` 等非浅显数值类型而言，临时文件通常有害无益。**
> 
> **总体而言，`Emplace` 优于 `Add`，因此其可避免在调用点创建无需临时变量**，并将此类变量复制或移动到容器中。**根据经验，可将 `Add` 用于浅显类型，将 `Emplace` 用于其他类型。`Emplace` 的效率始终高于 `Add`，但 `Add` 的可读性可能更好。**
 
- **`Append`** ：一次性添加其他 `TArray` 中的多个元素，或者指向常规 C 数组的指针及该数组的大小：
```c++
FString Arr[] = { TEXT("of"), TEXT("Tomorrow") };
StrArr.Append(Arr, ARRAY_COUNT(Arr));
// StrArr == ["Hello","World","of","Tomorrow"]
```

- **`AddUnique`** ：仅在尚不存在等值元素时，才会向容器添加新元素。使用以下元素类型的运算符检查等值性：`运算符==`:
```c++
    StrArr.AddUnique(TEXT("!"));
    // StrArr == ["Hello","World","of","Tomorrow","!"]

    StrArr.AddUnique(TEXT("!"));
    // StrArr is unchanged as "!" is already an element
```

- **`Insert`** ：在给定索引处添加单个元素或元素数组的副本：
```c++
StrArr.Insert(TEXT("Brave"), 1);
// StrArr == ["Hello","Brave","World","of","Tomorrow","!"]
```

**`SetNum`** ：直接设置数组元素的数量。如新数量大于当前数量，则使用元素类型的默认构造函数新建元素：
```c++
StrArr.SetNum(8);
// StrArr == ["Hello","Brave","World","of","Tomorrow","!","",""]
```

如新数量小于当前数量，`SetNum` 将移除元素。移除元素的更多相关详情将稍后讨论：
```c++
StrArr.SetNum(6);
// StrArr == ["Hello","Brave","World","of","Tomorrow","!"]
```

### 遍历

- 建议使用**范围 for 循环**：
```c++
FString JoinedStr;
for (auto& Str :StrArr)
{
    JoinedStr += Str;
    JoinedStr += TEXT(" ");
}
// JoinedStr == "Hello Brave World of Tomorrow !"
```

- 基于索引的**常规遍历**：
```c++
for (int32 Index = 0; Index != StrArr.Num(); ++Index)
{
    JoinedStr += StrArr[Index];
    JoinedStr += TEXT(" ");
}
```

- **还可通过数组迭代器类型控制迭代。** 函数 `CreateIterator` 和 `CreateConstIterator` 可分别用于元素的**读写**和**只读**访问：
```c++
for (auto It = StrArr.CreateConstIterator(); It; ++It)
{
    JoinedStr += *It;
    JoinedStr += TEXT(" ");
}
```

### 排序

-  **`Sort`** ：快排算法，从小到大排序（数值按元素类型的 `<` 排序）

```c++
StrArr.Sort();
// StrArr == ["!","Brave","Hello","of","Tomorrow","World"]
```

二元谓词可以实现不同的排序语义，例如：字符串按长度从小到大排序
```c++
StrArr.Sort([](const FString& A, const FString& B) {
    return A.Len() < B.Len();
});
// StrArr == ["!","of","Hello","Brave","World","Tomorrow"]
```

> [!NOTE]
> 注意：与之前相比，数组中三个长度相同的字符串"Hello"、"Brave"和"World"的相对排序发生了变化。这是**因为 `Sort` 不稳定，等值元素（因为断言只比较长度，所以此处字符串为等值）的相对排序无法保证。**

- **`HeapSort`** ：无论是否使用二元谓词，均可用于执行堆排序。使用 HeapSort 函数与否，取决于特定数据与 Sort 函数相比时的排序效率。与 `Sort` 一样，`HeapSort` 也不稳定。若在上述范例中使用 `HeapSort` 而非 `Sort`，结果将如下所示（此例中结果相同）：

```c++
StrArr.HeapSort([](const FString& A, const FString& B) {
    return A.Len() < B.Len();
});
// StrArr == ["!","of","Hello","Brave","World","Tomorrow"]
```

-  **`StableSort`** ：归并排序算法，用于在排序后保证等值元素的相对顺序
```c++
StrArr.StableSort([](const FString& A, const FString& B) {
    return A.Len() < B.Len();
});
// StrArr == ["!","of","Brave","Hello","World","Tomorrow"]
//"Brave"、"Hello"和"World"的相对排序不会改变。
```

### 查询

-  **`Num`** ：数组保存的元素数量：
```c++
int32 Count = StrArr.Num();
// Count == 6
```

- **`GetTypeSize`**：可针对元素大小对容器进行询问，相当于 `sizeof`
```c++
uint32 ElementSize = StrArr.GetTypeSize();
// ElementSize == sizeof(FString)
```

- **`GetData`**：直接访问数组内存（如用于确定 C 类 API 的互操作性），将指针返回到数组中的元素。仅在数组存在且未执行更改数组的操作时，此指针方有效。仅 `StrPtr` 的首个 `Num` 指数才可被解除引用。
    - **返回的是指针**，如容器为常量，则返回的指针也为常量。
```c++
FString* StrPtr = StrArr.GetData();
// StrPtr[0] == "!"
// StrPtr[1] == "of"
// ...
// StrPtr[5] == "Tomorrow"
// StrPtr[6] - undefined behavior
```

- **索引运算符 `[]`**： 获取元素
    - **返回的是引用**，因此其还可用于改变数组中的元素（假定数组不为常量）。
    - 传递小于 0 或大于等于 `Num()` 的无效索引将导致运行时错误。
    - **`IsValidIndex`** ：询问容器，可确定特定索引是否有效
    - 与 `GetData` 函数相同：如数组为常量，`[]` 将返回常量引用。
```c++
FString Elem1 = StrArr[1];
// Elem1 == "of"

StrArr[3] = StrArr[3].ToUpper(); 
// StrArr == ["!","of","Brave","HELLO","World","Tomorrow"]

bool bValidM1 = StrArr.IsValidIndex(-1);
// bValidM1 == false
```

-  **`Last`** ：从数组末端反向索引。索引默认为零。`Top` 函数是 `Last` 的同义词，唯一区别是其不接受索引：

```c++
FString ElemEnd  = StrArr.Last(); 
FString ElemEnd0 = StrArr.Last(0);
FString ElemTop  = StrArr.Top();
// ElemEnd  == "Tomorrow"
// ElemEnd0 == "Tomorrow"
// ElemTop  == "Tomorrow"
```

- **`Contains`**：询问数组是否包含特定元素：
```c++
bool bHello   = StrArr.Contains(TEXT("Hello"));
// bHello   == true
```

- **`ContainsByPredicate`**：或询问数组是否包含与特定谓词匹配的元素：

```c++
//数组是否长度为5
bool bLen5 = StrArr.ContainsByPredicate([](const FString& Str){
    return Str.Len() == 5;
});
// bLen5 == true
```

-  **`Find`**：查找元素（只需要一次查找）。确定元素是否存在并返回其索引（返回的是找到的首个元素）
-  **`FindLast`** ：如存在重复元素而希望找到最末元素的索引
- **两个形参，返回布尔**，指出是否已找到元素，同时在找到元素索引时将其写入变量。
- **一个形参，返回索引**。如不将索引作为显式参数传递，这两个函数便会执行此操作。如未找到元素，将返回特殊 `INDEX_NONE` 值：

```c++
int32 Index;
if (StrArr.Find(TEXT("Hello"), Index))
{
    // Index == 3
}
```

```c++
int32 IndexLast;
if (StrArr.FindLast(TEXT("Hello"), IndexLast))
{
    // IndexLast == 3, because there aren't any duplicates
}
```

```c++
int32 Index2     = StrArr.Find(TEXT("Hello"));
int32 IndexLast2 = StrArr.FindLast(TEXT("Hello"));
int32 IndexNone  = StrArr.Find(TEXT("None"));
// Index2     == 3
// IndexLast2 == 3
// IndexNone  == INDEX_NONE
```

-  **`IndexOfByKey`** 的工作方式类似，不同元素可与任意对象比较。开始搜索前，使用 `Find` 函数会将参数实际转换为元素类型（此本例中为 `FString`）。使用 `IndexOfByKey`，可直接对不"键"，因此即使键类型无法直接转换为元素类型，也可进行搜索。

`IndexOfByKey` 适用于存在 `运算符==(ElementType, KeyType)` 的键类型。`IndexOfByKey` 将返回找到的首个元素的**索引**；如未找到元素，则返回 `INDEX_NONE`：

```c++
int32 Index = StrArr.IndexOfByKey(TEXT("Hello"));
// Index == 3
```

- **`IndexOfByPredicate`**: 用于查找与特定谓词匹配的首个元素的**索引**；如未找到，同样返回特殊 `INDEX_NONE` 值。可将指针返回指向找到的元素，而不返回指数。
```c++
int32 Index = StrArr.IndexOfByPredicate([](const FString& Str){
    return Str.Contains(TEXT("r"));
});
// Index == 2
```

- **`FindByKey`**： 与 `IndexOfByKey` 相似，将元素和任意对象进行对比，但返回指向所找到元素的**指针**。如未找到元素，则返回 `nullptr`。

```c++
auto* OfPtr  = StrArr.FindByKey(TEXT("of")));
auto* ThePtr = StrArr.FindByKey(TEXT("the")));
// OfPtr  == &StrArr[1]
// ThePtr == nullptr
```

- **`FindByPredicate`** ：的使用方式和 `IndexOfByPredicate` 相似，不同点是返回**指针**而非索引：
```c++
auto* Len5Ptr = StrArr.FindByPredicate([](const FString& Str){
    return Str.Len() == 5;
});
auto* Len6Ptr = StrArr.FindByPredicate([](const FString& Str){
    return Str.Len() == 6;
});
// Len5Ptr == &StrArr[2]
// Len6Ptr == nullptr
```

-  **`FilterByPredicate`** ：函数可获取与特定谓词匹配的元素数组：
```c++
auto Filter = StrArray.FilterByPredicate([](const FString& Str){
    return !Str.IsEmpty() && Str[0] < TEXT('M');
});
```

### 移除

- **`Remove`**：根据元素类型的 `运算符==` 函数**移除所有与提供元素等值的元素**。
```c++
TArray<int32> ValArr;
int32 Temp[] = { 10, 20, 30, 5, 10, 15, 20, 25, 30 };
ValArr.Append(Temp, ARRAY_COUNT(Temp));
// ValArr == [10,20,30,5,10,15,20,25,30]

ValArr.Remove(20);
// ValArr == [10,30,5,10,15,25,30]
```

- **`RemoveSingle`** ：**移除数组中的首个匹配元素**。以下情况尤为实用——此函数在数组中可能存在重复，而只希望擦除一个时；或作为优化，数组只能包含一个匹配元素时：
```c++
ValArr.RemoveSingle(30);
// ValArr == [10,5,10,15,25,30]
```

- **`RemoveAt`** ：**按照从零开始的索引移除元素**。可使用 `IsValidIndex` 确定数组中的元素是否使用计划提供的索引，将无效索引传递给此函数会导致运行时错误：

```c++
ValArr.RemoveAt(2); // Removes the element at index 2
// ValArr == [10,5,15,25,30]

ValArr.RemoveAt(99); // This will cause a runtime error as
                       // there is no element at index 99
```

- **`RemoveAll`** ：移除与谓词匹配的元素。
```c++
//移除为3倍数的所有数值：
ValArr.RemoveAll([](int32 Val) {
    return Val % 3 == 0;
});
// ValArr == [10,5,25]
```

- 移除某个元素后其后的元素将前移，该移动过程存在开销。如不需要剩余元素排序，可使用 **`RemoveSwap`**、**`RemoveAtSwap`** 和 **`RemoveAllSwap`** 函数减少此开销。此类函数的工作方式与其对应的非 Swap 函数相似，**不同之处在于其不保证剩余元素的排序，因此可更快地完成任务：**
```c++
TArray<int32> ValArr2;
for (int32 i = 0; i != 10; ++i)
    ValArr2.Add(i % 5);
// ValArr2 == [0,1,2,3,4,0,1,2,3,4]

ValArr2.RemoveSwap(2);
// ValArr2 == [0,1,4,3,4,0,1,3]

ValArr2.RemoveAtSwap(1);
// ValArr2 == [0,3,4,3,4,0,1]

ValArr2.RemoveAllSwap([](int32 Val) {
    return Val % 3 == 0;
});
// ValArr2 == [1,4,4]
```

-  **`Empty`** ：移除数组中所有元素：
```c++
ValArr2.Empty();
// ValArr2 == []
```

### 运算符

数组是常规数值类型，可使用标准复制构造函数或赋值运算符进行复制。由于数组严格拥有其元素，复制数组的操作是深层的，因此新数组将拥有其自身的元素副本：

```c++
TArray<int32> ValArr3;
ValArr3.Add(1);
ValArr3.Add(2);
ValArr3.Add(3);

auto ValArr4 = ValArr3;
// ValArr4 == [1,2,3];
ValArr4[0] = 5;
// ValArr3 == [1,2,3];
// ValArr4 == [5,2,3];
```

作为 `Append` 函数的替代，可使用  **`+=`**  对数组进行串联：

```c++
    ValArr4 += ValArr3;
    // ValArr4 == [5,2,3,1,2,3]
```

`TArray` 还**支持移动语义**，使用 **`MoveTemp`** 函数可调用这些语义。**移动后，源数组必定为空：**
```c++
    ValArr3 = MoveTemp(ValArr4);
    // ValArr3 == [5,2,3,1,2,3]
    // ValArr4 == []
```

使用 `运算符==` 和 `运算符!=` 可对数组进行比较。**元素的排序很重要：只有元素的顺序和数量相同时，两个数组才被视为相同。** 元素通过其自身的 `运算符==` 进行比较：

```c++
TArray<FString> FlavorArr1;
FlavorArr1.Emplace(TEXT("Chocolate"));
FlavorArr1.Emplace(TEXT("Vanilla"));
// FlavorArr1 == ["Chocolate","Vanilla"]

auto FlavorArr2 = Str1Array;
// FlavorArr2 == ["Chocolate","Vanilla"]

bool bComparison1 = FlavorArr1 == FlavorArr2;
// bComparison1 == true

for (auto& Str :FlavorArr2)
{
    Str = Str.ToUpper();
}
// FlavorArr2 == ["CHOCOLATE","VANILLA"]

bool bComparison2 = FlavorArr1 == FlavorArr2;
// bComparison2 == true, because FString comparison ignores case

Exchange(FlavorArr2[0], FlavorArr2[1]);
// FlavorArr2 == ["VANILLA","CHOCOLATE"]

bool bComparison3 = FlavorArr1 == FlavorArr2;
// bComparison3 == false, because the order has changed
```

### 堆

`TArray` 拥有支持二叉堆数据结构的函数。堆是一种二叉树，其中父节点的排序等于或高于其子节点。作为数组实现时，树的根节点位于元素0，索引N处节点的左右子节点的指数分别为2N+1和2N+2。子节点彼此间不存在特定排序。

-  **`Heapify`** ：将现有数组转换为堆。此会重载为是否接受谓词，无谓词的版本将使用元素类型的 `运算符<` 确定排序：
```c++
TArray<int32> HeapArr;
for (int32 Val = 10; Val != 0; --Val)
{
    HeapArr.Add(Val);
}
// HeapArr == [10,9,8,7,6,5,4,3,2,1]
HeapArr.Heapify();
// HeapArr == [1,2,4,3,6,5,8,10,7,9]
```

下图为树的展示：

![[ebc25c012f14a4da679ba8e79239d9fd_MD5.jpg]]

树中的节点按堆化数组中元素的排序从左至右、从上至下读取。注意：数组在转换为堆后无需排序。排序数组也是有效堆，但堆结构的定义较为宽松，同一组元素可存在多个有效堆。

- **`HeapPush`**：将新元素添加到堆，对其他节点进行重新排序，以对堆进行维护：
```c++
HeapArr.HeapPush(4);
// HeapArr == [1,2,4,3,4,5,8,10,7,9,6]
```

![[3c72916abfbce477554a0ac63d9711eb_MD5.jpg]]

- **`HeapPop`** 和 **`HeapPopDiscard`** ：用于移除堆的顶部节点。这两个函数的区别在于前者引用元素的类型来返回顶部元素的副本，而后者只是简单地移除顶部节点，不进行任何形式的返回。两个函数得出的数组变更一致，重新正确排序其他元素可对堆进行维护：

```c++
int32 TopNode;
HeapArr.HeapPop(TopNode);
// TopNode == 1
// HeapArr == [2,3,4,6,4,5,8,10,7,9]
```

![[132b01cd79f6a706f2f89baae1a8bf1b_MD5.jpg]]

- **`HeapRemoveAt`**： 将删除数组中给定索引处的元素，然后重新排列元素，对堆进行维护：
```c++
HeapArr.HeapRemoveAt(1);
// HeapArr == [2,4,4,6,9,5,8,10,7]
```

![[f54cd1b8099a40e6a5e5518a08c5f2ba_MD5.jpg]]

`Heapify` 调用、其他堆操作或手动将数组操作到堆中之后），才应调用 `HeapPush`、`HeapPop`、`HeapPopDiscard` 和 `HeapRemoveAt`

此类函数（包括 `Heapify`）都可选择使用二元谓词决定堆中节点元素的排序。堆操作默认使用元素类型的 `运算符<` 确定排序。如使用自定义谓词，须在所有堆操作中使用相同谓词。

-  **`HeapTop` ：检查堆的顶部节点，无需变更数组：
```c++
int32 Top = HeapArr.HeapTop();
// Top == 2
```

### Slack

为避免每次添加元素时重新分配内存，分配器提供的内存通常会超过必要内存，使之后调用 `Add` 时不会因重新分配内存而降低性能。同样，**删除元素通常不会释放内存.此操作会使数组拥有 `Slack` 元素，也就是当前未使用的有效预分配元素储存槽**。
**数组中存储的元素量与数组使用分配内存可存储的元素数量间的差值即为数组中的Slack量。**
默认构建的数组不分配内存，Slack 初始为零。

-  **`GetSlack`** 函数可找出数组中的 Slack 量。
-  `Max` 函数可获取容器重新分配前数组可保存的最大元素数量。
- `GetSlack` 等同 `Max` 和 `Num` 间的差值：

```c++
TArray<int32> SlackArray;
// SlackArray.GetSlack() == 0
// SlackArray.Num()      == 0
// SlackArray.Max()      == 0

SlackArray.Add(1);
// SlackArray.GetSlack() == 3
// SlackArray.Num()      == 1
// SlackArray.Max()      == 4

SlackArray.Add(2);
SlackArray.Add(3);
SlackArray.Add(4);
SlackArray.Add(5);
// SlackArray.GetSlack() == 17
// SlackArray.Num()      == 5
// SlackArray.Max()      == 22
```

分配器确定重新分配后容器中的Slack量。因此，用户不应认为Slack是常量。

**虽然无需管理 Slack，但可管理 Slack 对数组进行优化，以满足需求。** 例如，如需要向数组添加大约100个新元素，则可在添加前确保拥有可至少存储100个新元素的 Slack，以便添加新元素时无需分配内存。上文所述的 `Empty` 函数接受可选 Slack 参数：

```c++
SlackArray.Empty();
// SlackArray.GetSlack() == 0
// SlackArray.Num()      == 0
// SlackArray.Max()      == 0
SlackArray.Empty(3);
// SlackArray.GetSlack() == 3
// SlackArray.Num()      == 0
// SlackArray.Max()      == 3
SlackArray.Add(1);
SlackArray.Add(2);
SlackArray.Add(3);
// SlackArray.GetSlack() == 0
// SlackArray.Num()      == 3
// SlackArray.Max()      == 3
```

-  `Reset` 函数与 Empty 函数类似，不同之处是若当前内存分配已提供请求的 Slack，该函数将不释放内存。但若请求的 Slack 较大，其将分配更多内存：

```c++
SlackArray.Reset(0);
// SlackArray.GetSlack() == 3
// SlackArray.Num()      == 0
// SlackArray.Max()      == 3
SlackArray.Reset(10);
// SlackArray.GetSlack() == 10
// SlackArray.Num()      == 0
// SlackArray.Max()      == 10
```

-  `Shrink` 函数可移除所有 Slack。此才做将把内存分配调整为保存当前元素所需的最小内存。`Shrink` 不会对数组中的元素产生影响。

```c++
SlackArray.Add(5);
SlackArray.Add(10);
SlackArray.Add(15);
SlackArray.Add(20);
// SlackArray.GetSlack() == 6
// SlackArray.Num()      == 4
// SlackArray.Max()      == 10
SlackArray.Shrink();
// SlackArray.GetSlack() == 0
// SlackArray.Num()      == 4
// SlackArray.Max()      == 4
```

### 原始内存

本质上而言，`TArray` 只是分配内存周围的包装器。直接修改分配的字节和自行创建元素即可将其用作包装器，此操作十分实用。`Tarray` 将尽量利用其拥有的信息进行执行，但有时需降低一个等级。

`AddUninitialized` 和 `InsertUninitialized` 函数可将未初始化的空间添加到数组。两者工作方式分别与 `Add` 和 `Insert` 函数相同，**只是不调用元素类型的构造函数**。**若要避免调用构造函数，建议使用此类函数**。类似以下范例的情况中建议使用此类函数，其中计划用 `Memcpy` 调用完全覆盖结构体：

```c++
int32 SrcInts[] = { 2, 3, 5, 7 };
TArray<int32> UninitInts;
UninitInts.AddUninitialized(4);
FMemory::Memcpy(UninitInts.GetData(), SrcInts, 4*sizeof(int32));
// UninitInts == [2,3,5,7]
```

也可使用此功能保留计划自行构建对象所需内存：

```c++
TArray<FString> UninitStrs;
UninitStrs.Emplace(TEXT("A"));
UninitStrs.Emplace(TEXT("D"));
UninitStrs.InsertUninitialized(1, 2);
new ((void*)(UninitStrs.GetData() + 1)) FString(TEXT("B"));
new ((void*)(UninitStrs.GetData() + 2)) FString(TEXT("C"));
// UninitStrs == ["A","B","C","D"]
```

`AddZeroed` 和 `InsertZeroed` 的工作方式相似，不同点是会将添加/插入的空间字节清零：

```c++
struct S
{
    S(int32 InInt, void* InPtr, float InFlt)
        :Int(InInt)
        , Ptr(InPtr)
        , Flt(InFlt)
    {
    }
    int32 Int;
    void* Ptr;
    float Flt;
};
TArray<S> SArr;
SArr.AddZeroed();
// SArr == [{ Int:0, Ptr: nullptr, Flt:0.0f }]
```

`SetNumUninitialized` 和 `SetNumZeroed` 函数的工作方式与 `SetNum` 类似，不同之处在于新数量大于当前数量时，将保留新元素的空间为未初始化或按位归零。与 `AddUninitialized` 和 `InsertUninitialized` 函数相同，必要时需将新元素正确构建到新空间中：

```c++
SArr.SetNumUninitialized(3);
new ((void*)(SArr.GetData() + 1)) S(5, (void*)0x12345678, 3.14);
new ((void*)(SArr.GetData() + 2)) S(2, (void*)0x87654321, 2.72);
// SArr == [
//   { Int:0, Ptr: nullptr,    Flt:0.0f  },
//   { Int:5, Ptr:0x12345678, Flt:3.14f },
//   { Int:2, Ptr:0x87654321, Flt:2.72f }
// ]

SArr.SetNumZeroed(5);
// SArr == [
//   { Int:0, Ptr: nullptr,    Flt:0.0f  },
//   { Int:5, Ptr:0x12345678, Flt:3.14f },
//   { Int:2, Ptr:0x87654321, Flt:2.72f },
//   { Int:0, Ptr: nullptr,    Flt:0.0f  },
//   { Int:0, Ptr: nullptr,    Flt:0.0f  }
// ]
```

应谨慎使用"Uninitialized"和"Zeroed"函数族。如函数类型包含要构建的成员或未处于有效按位清零状态的成员，可导致数组元素无效和未知行为。此类函数适用于固定的数组类型，例如FMatrix和FVector。

### 其他

`BulkSerialize` 函数是序列化函数，可用作替代 `运算符<<`，将数组作为原始字节块进行序列化，而非执行逐元素序列化。如使用内置类型或纯数据结构体等浅显元素，可改善性能。

`CountBytes` 和 `GetAllocatedSize` 函数用于估算数组当前内存占用量。`CountBytes` 接受 `FArchive`，可直接调用 `GetAllocatedSize`。此类函数常用于统计报告。

`Swap` 和 `SwapMemory` 函数均接受两个指数并交换此类指数上的元素值。这两个函数相同，不同点是 `Swap` 会对指数执行额外的错误检查，并断言索引是否超出范围。

## TMap 映射
`TMap` 与 `TSet` 类似，它们的结构均基于对键进行散列运算。但与 `TSet` 不同的是，**此容器将数据存储为键值对（`TPair<KeyType, ValueType>`），只将键用于存储和获取。**

映射有两种类型：`TMap` 和 `TMultiMap`。
-  **`TMap`** 中的**键是唯一**的。**添加新的键值时，若所用的键与原有的对相同，新对将替换原有的对。**
-  **`TMultiMap `** 可存储多个相同的键。容器可以同时存储新对和原有的对。

**在 `TMap` 中，键值对被视为映射的元素类型，相当于每一对都是个体对象。在本文中，元素就意味着键值对，而各个组件就被称作元素的键或元素的值。元素类型实际上是 `TPair<KeyType, ElementType>`，但很少需要直接引用 `TPair` 类型。**

**同质容器：所有元素均为完全相同类型**。

`TMap` 也是值类型，支持通常的复制、赋值和析构函数运算，以及它的元素的强所有权。在映射被销毁时，它的元素都会被销毁。键和值也必须为值类型。

`TMap` 是散列容器，这意味着键类型必须支持 **`GetTypeHash`** 函数，并提供 `运算符==` 来比较各个键是否等值。

`TMap` 也可使用任选分配器来控制内存分配行为。但不同于 `TArray`，这些是集合分配器，而不是 `FHeapAllocator` 和 `TInlineAllocator` 之类的标准UE4分配器。集合分配器（`TSetAllocator`类）定义映射应使用的散列桶数量，以及应使用哪个标准UE4分配器来存储散列和元素。

**`KeyFuncs`** 是最后一个 `TMap` 模板参数，**该参数告知映射如何从元素类型获取键，如何比较两个键是否相等，以及如何对键进行散列计算。**

**这些参数有默认值，它们只会返回对键的引用，使用 `运算符==` 确定相等性，并调用非成员 `GetTypeHash` 函数进行散列计算。** 如果您的键类型支持这些函数，可使用它作为映射键，不需要提供自定义 `KeyFuncs`。

与 `TArray` 不同的是，内存中 `TMap` 元素的相对排序既不可靠也不稳定，对这些元素进行迭代很可能会使它们返回的顺序和它们添加的顺序有所不同。这些元素也不太可能在内存中连续排列。映射的支持数据结构是稀疏数组，这种数组可有效支持元素之间的空位。当元素从映射中被移除时，稀疏数组中就会出现空位。将新的元素添加到数组可填补这些空位。但是，即便 `TMap` 不会打乱元素来填补空位，指向映射元素的指针仍然可能失效，因为如果存储器被填满，又添加了新的元素，整个存储可能会重新分配。

### 创建和填充映射

`TMap` 的创建方法如下：

```c++
TMap<int32, FString> FruitMap;
```

`FruitMap` 现在是一个字符串的空 `TMap`，该字符串由整数键标识。我们既没有指定分配器，也没有指定 `KeyFuncs`，所以映射将执行标准的堆分配，使用 `运算符==` 对键进行对比（`int32` 类型），并使用 `GetTypeHash` 进行散列运算。此时没有分配任何内存。

 - **`Add`** ：填充映射 （键，值）
 - 可接受不带值的键，值被默认构建为`""`

```c++
FruitMap.Add(5, TEXT("Banana"));
FruitMap.Add(2, TEXT("Grapefruit"));
FruitMap.Add(7, TEXT("Pineapple"));
FruitMap.Add(4);
// FruitMap == [
//  { Key:5, Value:"Banana"     },
//  { Key:2, Value:"Grapefruit" },
//  { Key:7, Value:"Pineapple"  }
//  { Key:4, Value:""          }
// ]
```

**此处的元素按插入顺序排列，但不保证这些元素在内存中实际保留此排序。** 如果是新的映射，可能会保留插入排序，但插入和删除的次数越多，新元素不出现在末尾的可能性就越大。

-  **`Emplace`** ：和 `TArray` 一样，还可使用 `Emplace` 代替 `Add`，防止插入映射时创建临时文件。此处直接将键和值传递给了各自的构造函数，**与 `TArray` 不同的是，只能通过单一参数构造函数将元素安放到映射中。**
```c++
    FruitMap.Emplace(3, TEXT("Orange"));
    // FruitMap == [
    //  { Key:5, Value:"Banana"    },
    //  { Key:2, Value:"Pear"      },
    //  { Key:7, Value:"Pineapple" },
    //  { Key:4, Value:""          },
    //  { Key:3, Value:"Orange"    }
    // ]
```

-  **`Append`** ：合并映射，将一个映射的所有元素移至另一个映射：

```c++
TMap<int32, FString> FruitMap2;
FruitMap2.Emplace(4, TEXT("Kiwi"));
FruitMap2.Emplace(9, TEXT("Melon"));
FruitMap2.Emplace(5, TEXT("Mango"));
FruitMap.Append(FruitMap2);
// FruitMap == [
//  { Key:5, Value:"Mango"     },
//  { Key:2, Value:"Pear"      },
//  { Key:7, Value:"Pineapple" },
//  { Key:4, Value:"Kiwi"      },
//  { Key:3, Value:"Orange"    },
//  { Key:9, Value:"Melon"     }
// ]
// FruitMap2 is now empty.
```

- 如果用 `UPROPERTY` 宏和一个可编辑的关键词（`EditAnywhere`、`EditDefaultsOnly` 或 `EditInstanceOnly`）标记 `TMap`，即可在编辑器中添加和编辑元素。
```c++
UPROPERTY(Category = MapsAndSets, EditAnywhere)
TMap<int32, FString> FruitMap;
```

### 遍历

`TMaps` 的迭代类似于 `TArrays`。可使用 C++的设置范围功能，注意元素类型是 **`TPair`**：

```c++
for (auto& Elem :FruitMap)
{
    FPlatformMisc::LocalPrint(
        *FString::Printf(
            TEXT("(%d, \"%s\")\n"),
            Elem.Key,
            *Elem.Value
        )
    );
}
// Output:
// (5, "Mango")
// (2, "Pear")
// (7, "Pineapple")
// (4, "Kiwi")
// (3, "Orange")
// (9, "Melon")
```

也可以用 `CreateIterator` 和 `CreateConstIterators` 函数来创建迭代器。
-  `CreateIterator` 返回拥有**读写**访问权限的迭代器
-  `CreateConstIterator` 返回拥有**只读**访问权限的迭代器。无论哪种情况，均可用这些迭代器的 `Key` 和 `Value` 来检查元素。使用迭代器显示"fruit"范例映射将产生如下结果：

```c++
for (auto It = FruitMap.CreateConstIterator(); It; ++It)
{
    FPlatformMisc::LocalPrint(
        *FString::Printf(
            TEXT("(%d, \"%s\")\n"),
            It.Key(),   // same as It->Key
            *It.Value() // same as *It->Value
        )
    );
}
```

### 查询

-  **`Num`** ：查询映射中保存的**元素数量**：
```c++
int32 Count = FruitMap.Num();
// Count == 6
```

- **`Contains`**： 映射是否包含特定**键**

```c++
bool bHas7 = FruitMap.Contains(7);
// bHas7 == true
```

-  **`[键]`**： 如果知道映射中存在某个特定键，**将键用作索引来查找相应值**。使用非常量映射执行该操作将返回非常量引用，使用常量映射将返回常量引用。
```c++
FString Val7 = FruitMap[7];
// Val7 == "Pineapple"
```

-  **`Find`** ：查找键
- 如果包含该键，`Find` 将返回**指向元素数值的指针**。
- 如果不包含该键，则返回 `nullptr`。
- 在常量映射上调用 `Find`，返回的指针也将为常量。
```c++
    FString* Ptr7 = FruitMap.Find(7);
    FString* Ptr8 = FruitMap.Find(8);
    // *Ptr7 == "Pineapple"
    //  Ptr8 == nullptr
```

- **`FindOrAdd`** ：返回对与给定键关联的**值的引用**。
    - 如果不存在该键，`FindOrAdd` 将返回新创建的元素（使用给定键和默认构建值），该元素也会被添加到映射。
    - `FindOrAdd` 可修改映射，因此**仅适用于非常量映射**。不要被名称迷惑，
- **`FindRef`** 会返回与给定键关联的**值副本**；
    - 若未找到给定键，则返回默认构建值。
    - `FindRef` 不会创建新元素，因此**既可用于常量映射，也可用于非常量映射**。
- 即使在映射中找不到键，`FindOrAdd` 和 `FindRef` 也会成功运行，因此无需执行常规的安全规程（如提前检查 `Contains` 或对返回值进行空白检查）就可安全地调用。
```c++
FString& Ref7 = FruitMap.FindOrAdd(7);
// Ref7     == "Pineapple"
// FruitMap == [
//  { Key:5, Value:"Mango"     },
//  { Key:2, Value:"Pear"      },
//  { Key:7, Value:"Pineapple" },
//  { Key:4, Value:"Kiwi"      },
//  { Key:3, Value:"Orange"    },
//  { Key:9, Value:"Melon"     }
// ]
FString& Ref8 = FruitMap.FindOrAdd(8);
// Ref8     == ""
// FruitMap == [
//  { Key:5, Value:"Mango"     },
//  { Key:2, Value:"Pear"      },
//  { Key:7, Value:"Pineapple" },
//  { Key:4, Value:"Kiwi"      },
//  { Key:3, Value:"Orange"    },
//  { Key:9, Value:"Melon"     },
//  { Key:8, Value:""          }
// ]

FString Val7 = FruitMap.FindRef(7);
FString Val6 = FruitMap.FindRef(6);
// Val7     == "Pineapple"
// Val6     == ""
// FruitMap == [
//  { Key:5, Value:"Mango"     },
//  { Key:2, Value:"Pear"      },
//  { Key:7, Value:"Pineapple" },
//  { Key:4, Value:"Kiwi"      },
//  { Key:3, Value:"Orange"    },
//  { Key:9, Value:"Melon"     },
//  { Key:8, Value:""          }
// ]
```

- **`FindKey`** ：逆向按值查找，返回指向与所提供**值**配对的第一个键的指针。搜索映射中不存在的值将返回空键。
```c++
    const int32* KeyMangoPtr   = FruitMap.FindKey(TEXT("Mango"));
    const int32* KeyKumquatPtr = FruitMap.FindKey(TEXT("Kumquat"));
    // *KeyMangoPtr   == 5
    //  KeyKumquatPtr == nullptr
```

按值查找比按键查找慢（线性时间）。这是因为映射按键排序，而非按值排序。

- **`GenerateKeyArray`**  和 **`GenerateValueArray`** 分别使用所有键和值的副本来填充 `TArray`。在这两种情况下，都会在填充前清空所传递的数组，因此产生的元素数量始终等于映射中的元素数量。
```c++
    TArray<int32>   FruitKeys;
    TArray<FString> FruitValues;
    FruitKeys.Add(999);
    FruitKeys.Add(123);
    FruitMap.GenerateKeyArray  (FruitKeys);
    FruitMap.GenerateValueArray(FruitValues);
    // FruitKeys   == [ 5,2,7,4,3,9,8 ]
    // FruitValues == [ "Mango","Pear","Pineapple","Kiwi","Orange",
    //                  "Melon","" ]
```

### 移除

-  **`Remove`** ：提供要移除元素的键。返回值是被移除元素的数量。
    - 如果映射不包含与键匹配的元素，则返回值可为零
    - 移除元素将在数据结构中留下空位
```c++
    FruitMap.Remove(8);
    // FruitMap == [
    //  { Key:5, Value:"Mango"     },
    //  { Key:2, Value:"Pear"      },
    //  { Key:7, Value:"Pineapple" },
    //  { Key:4, Value:"Kiwi"      },
    //  { Key:3, Value:"Orange"    },
    //  { Key:9, Value:"Melon"     }
    // ]
```

- **`FindAndRemoveChecked`**： 可用于从映射移除元素并返回其值。名称的"已检查"部分表示若键不存在，映射将调用 `check`（UE4中等同于 `assert`）。

```c++
FString Removed7 = FruitMap.FindAndRemoveChecked(7);
// Removed7 == "Pineapple"
// FruitMap == [
//  { Key:5, Value:"Mango"  },
//  { Key:2, Value:"Pear"   },
//  { Key:4, Value:"Kiwi"   },
//  { Key:3, Value:"Orange" },
//  { Key:9, Value:"Melon"  }
// ]

FString Removed8 = FruitMap.FindAndRemoveChecked(8);
// Assert!
```

-  **`RemoveAndCopyValue`** ：作用与 `Remove` 相似，**不同点是会将已移除元素的值复制到引用参数**。如果映射中不存在指定的键，则输出参数将保持不变，函数将返回 `false`。

```c++
    FString Removed;
    bool bFound2 = FruitMap.RemoveAndCopyValue(2, Removed);
    // bFound2  == true
    // Removed  == "Pear"
    // FruitMap == [
    //  { Key:5, Value:"Mango"  },
    //  { Key:4, Value:"Kiwi"   },
    //  { Key:3, Value:"Orange" },
    //  { Key:9, Value:"Melon"  }
    // ]

```

-  **`Empty`** 或 **`Reset`** ：将映射中的所有元素移除。
    -  `Empty` 和 `Reset` 相似，但 `Empty` 可采用参数指示映射中保留的 slack 量
```c++
    TMap<int32, FString> FruitMapCopy = FruitMap;
    // FruitMapCopy == [
    //  { Key:5, Value:"Mango"  },
    //  { Key:4, Value:"Kiwi"   },
    //  { Key:3, Value:"Orange" },
    //  { Key:9, Value:"Melon"  }
    // ]

    FruitMapCopy.Empty(); // We could also have called Reset() here.
    // FruitMapCopy == []
```


### 排序

`TMap` 可以进行排序。排序后，迭代映射会以排序的顺序显示元素，但下次修改映射时，排序可能会发生变化。**排序是不稳定的，因此等值元素在 MultiMap 中可能以任何顺序出现。**

-  **`KeySort`** ：按键排序
-  **`ValueSort`**： 按值排序
- 两个函数均使用二元谓词来进行排序：

```c++
FruitMap.KeySort([](int32 A, int32 B) {
    return A > B; // sort keys in reverse
});
// FruitMap == [
//  { Key:9, Value:"Melon"  },
//  { Key:5, Value:"Mango"  },
//  { Key:4, Value:"Kiwi"   },
//  { Key:3, Value:"Orange" }
// ]

FruitMap.ValueSort([](const FString& A, const FString& B) {
    return A.Len() < B.Len(); // sort strings by length
});
// FruitMap == [
//  { Key:4, Value:"Kiwi"   },
//  { Key:5, Value:"Mango"  },
//  { Key:9, Value:"Melon"  },
//  { Key:3, Value:"Orange" }
// ]
```

### 运算符

和 `TArray` 一样，`TMap` 是常规值类型，可通过标准复制构造函数或赋值运算符进行复制。因为映射严格拥有其元素，复制映射的操作是深层的，所以新的映射将拥有其自己的元素副本。

```c++
TMap<int32, FString> NewMap = FruitMap;
NewMap[5] = "Apple";
NewMap.Remove(3);
// FruitMap == [
//  { Key:4, Value:"Kiwi"   },
//  { Key:5, Value:"Mango"  },
//  { Key:9, Value:"Melon"  },
//  { Key:3, Value:"Orange" }
// ]
// NewMap == [
//  { Key:4, Value:"Kiwi"  },
//  { Key:5, Value:"Apple" },
//  { Key:9, Value:"Melon" }
// ]
```

`TMap` 支持移动语义，使用 `MoveTemp` 函数可调用这些语义。在移动后，源映射必定为空：

```c++
FruitMap = MoveTemp(NewMap);
// FruitMap == [
//  { Key:4, Value:"Kiwi"  },
//  { Key:5, Value:"Apple" },
//  { Key:9, Value:"Melon" }
// ]
// NewMap == []
```

### Slack

Slack是不包含元素的已分配内存。调用 `Reserve` 可分配内存，无需添加元素；通过非零slack参数调用 `Reset` 或 `Empty` 可移除元素，无需将其使用的内存取消分配。Slack优化了将新元素添加到映射的过程，因为可以使用预先分配的内存，而不必分配新内存。它在移除元素时也十分实用，因为系统不需要将内存取消分配。在清空希望用相同或更少的元素立即重新填充的映射时，此方法尤其有效。

`TMap` 不像 `TArray` 中的 `Max` 函数那样可以检查预分配元素的数量。

在下列代码中，`Reserve` 函数预先分配映射，最多可包含10个元素。

```c++
    FruitMap.Reserve(10);
    for (int32 i = 0; i < 10; ++i)
    {
        FruitMap.Add(i, FString::Printf(TEXT("Fruit%d"), i));
    }
    // FruitMap == [
    //  { Key:9, Value:"Fruit9" },
    //  { Key:8, Value:"Fruit8" },
    //  ...
    //  { Key:1, Value:"Fruit1" },
    //  { Key:0, Value:"Fruit0" }
    // ]
```

使用 `Collapse` 和 `Shrink` 函数可移除 `TMap` 中的全部slack。`Shrink` 将从容器的末端移除所有slack，但这会在中间或开始处留下空白元素。

```c++
    for (int32 i = 0; i < 10; i += 2)
    {
        FruitMap.Remove(i);
    }
    // FruitMap == [
    //  { Key:9, Value:"Fruit9" },
    //  <invalid>,
    //  { Key:7, Value:"Fruit7" },
    //  <invalid>,
    //  { Key:5, Value:"Fruit5" },
    //  <invalid>,
    //  { Key:3, Value:"Fruit3" },
    //  <invalid>,
    //  { Key:1, Value:"Fruit1" },
    //  <invalid>
    // ]
    FruitMap.Shrink();
    // FruitMap == [
    //  { Key:9, Value:"Fruit9" },
    //  <invalid>,
    //  { Key:7, Value:"Fruit7" },
    //  <invalid>,
    //  { Key:5, Value:"Fruit5" },
    //  <invalid>,
    //  { Key:3, Value:"Fruit3" },
    //  <invalid>,
    //  { Key:1, Value:"Fruit1" }
    // ]
```

在上述代码中，`Shrink` 只删除了一个无效元素，因为末端只有一个空元素。要移除所有slack，首先应调用 `Compact` 函数，将空白空间组合在一起，为调用 `Shrink` 做好准备。

```c++
    FruitMap.Compact();
    // FruitMap == [
    //  { Key:9, Value:"Fruit9" },
    //  { Key:7, Value:"Fruit7" },
    //  { Key:5, Value:"Fruit5" },
    //  { Key:3, Value:"Fruit3" },
    //  { Key:1, Value:"Fruit1" },
    //  <invalid>,
    //  <invalid>,
    //  <invalid>,
    //  <invalid>
    // ]
    FruitMap.Shrink();
    // FruitMap == [
    //  { Key:9, Value:"Fruit9" },
    //  { Key:7, Value:"Fruit7" },
    //  { Key:5, Value:"Fruit5" },
    //  { Key:3, Value:"Fruit3" },
    //  { Key:1, Value:"Fruit1" }
    // ]
```

### KeyFuncs
**`KeyFuncs`** 是最后一个 `TMap` 模板参数，**该参数告知映射如何从元素类型获取键，如何比较两个键是否相等，以及如何对键进行散列计算。**

只要类型具有 `运算符==` 和非成员 `GetTypeHash` 重载，就可用作 `TMap` 的键类型，不需要任何更改。但是，您可能需要将类型用作键，而不重载这些函数。在这些情况下，可对 `KeyFuncs` 进行自定义。为键类型创建 `KeyFuncs`，必须定义两个typedef和三个静态函数，如下所示：

- `KeyInitType` —— 用于传递键的类型。
- `ElementInitType` —— 用于传递元素的类型。
- `KeyInitType GetSetKey(ElementInitType Element)`——返回元素的键。
- `bool Matches(KeyInitType A, KeyInitType B)` —— 如果 `A` 和 `B` 等值将返回 `true`，否则返回 `false`。
- `uint32 GetKeyHash(KeyInitType Key)` —— 返回 `Key` 的散列值。
    
`KeyInitType` 和 `ElementInitType` 是键类型和值类型的常规传递约定的typedef。它们通常为浅显类型的一个值，和非浅显类型的一个常量引用。请记住，映射的元素类型是 `TPair`。

自定义 `KeyFuncs` 的示例可能如下所示：

```c++
    struct FMyStruct
    {
        // String which identifies our key
        FString UniqueID;

        // Some state which doesn't affect struct identity
        float SomeFloat;

        explicit FMyStruct(float InFloat)
            :UniqueID (FGuid::NewGuid().ToString())
            , SomeFloat(InFloat)
        {
        }
    };
    template <typename ValueType>
    struct TMyStructMapKeyFuncs :
        BaseKeyFuncs<
            TPair<FMyStruct, ValueType>,
            FString
        >
    {
    private:
        typedef BaseKeyFuncs<
            TPair<FMyStruct, ValueType>,
            FString
        > Super;

    public:
        typedef typename Super::ElementInitType ElementInitType;
        typedef typename Super::KeyInitType     KeyInitType;

        static KeyInitType GetSetKey(ElementInitType Element)
        {
            return Element.Key.UniqueID;
        }

        static bool Matches(KeyInitType A, KeyInitType B)
        {
            return A.Compare(B, ESearchCase::CaseSensitive) == 0;
        }

        static uint32 GetKeyHash(KeyInitType Key)
        {
            return FCrc::StrCrc32(*Key);
        }
    };
```

`FMyStruct` 具有唯一标识符，以及一些与身份无关的其他数据。`GetTypeHash` 和 `运算符==` 不适用于此，因为 `运算符==` 为实现通用目的不应忽略任何类型的数据，但同时又需要如此才能与 `GetTypeHash` 的行为保持一致，后者只关注 `UniqueID` 字段。以下步骤有助于为 `FMyStruct` 创建自定义 `KeyFuncs`：

1. 首先，继承 `BaseKeyFuncs`，因为它可以帮助定义某些类型，包括 `KeyInitType` 和 `ElementInitType`。
    
    `BaseKeyFuncs` 使用两个模板参数：映射的元素类型和键类型。和所有映射一样，元素类型是 `TPair`，使用 `FMyStruct` 作为其 `KeyType`，`TMyStructMapKeyFuncs` 的模板参数作为其 `ValueType`。将备用 `KeyFuncs` 用作模板，可为每个映射指定 `ValueType`，因此每次要在 `FMyStruct` 上创建键控 `TMap` 时不必定义新的 `KeyFuncs`。第二个 `BaseKeyFuncs` 参数是键类型，不要与元素存储的键区（`TPair` 的 `KeyType`）混淆。因为此映射应使用 `UniqueID`（来自 `FMyStruct`）作为键，所以此处使用 `FString`。
    
2. 然后，定义三个必需的 `KeyFuncs` 静态函数。第一个是 `GetSetKey`，该函数返回给定元素类型的键。由于元素类型是 `TPair`，而键是 `UniqueID`，所以该函数可直接返回 `UniqueID`。
    
    第二个静态函数是 `Matches`，该函数接受两个元素的键（由 `GetSetKey` 获取），然后比较它们是否相等。在 `FString` 中，标准的等效测试（`运算符==`）不区分大小写；要替换为区分大小写的搜索，请用相应的大小写对比选项使用 `Compare` 函数。
    
3. 最后，`GetKeyHash` 静态函数接受提取的键并返回其散列值。由于 `Matches` 函数区分大小写，`GetKeyHash` 也必须区分大小写。区分大小写的 `FCrc` 函数将计算键字符串的散列值。
    
4. 现在结构已满足 `TMap` 要求的行为，可创建它的实例。

`KeyFuncs` 参数处于最后，所以这个 `TMap`

```c++
    TMap<
        FMyStruct,
        int32,
        FDefaultSetAllocator,
        TMyStructMapKeyFuncs<int32>
    > MyMapToInt32;

    // Add some elements
    MyMapToInt32.Add(FMyStruct(3.14f), 5);
    MyMapToInt32.Add(FMyStruct(1.23f), 2);

    // MyMapToInt32 == [
    //  {
    //      Key:{
    //          UniqueID:"D06AABBA466CAA4EB62D2F97936274E4",
    //          SomeFloat:3.14f
    //      },
    //      Value:5
    //  },
    //  {
    //      Key:{
    //          UniqueID:"0661218447650259FD4E33AD6C9C5DCB",
    //          SomeFloat:1.23f
    //      },
    //      Value:5
    //  }
    // ]
```

`TMap` 假设两个项目使用 `Matches` 比较的结果相等，则它们会从 `GetKeyHash` 返回相同的值。此外，如果对现有映射元素的键进行的修改将会改变来自这两个函数中任一个的结果，那么系统会将这种修改视作未定义的行为，因为这会使映射的内部散列失效。这些规则也适用于使用默认 `KeyFuncs` 时 `运算符==` 和 `GetKeyHash`

### 其他

`CountBytes` 和 `GetAllocatedSize` 函数用于估计内部数组的当前内存使用情况。`CountBytes` 接受 `Farchive` 参数，而 `GetAllocatedSize` 则不会。这些函数常用于统计报告。

`Dump` 函数接受 `FOutputDevice`，并写出关于映射内容的实现信息。此函数常用于调试。

## TSet 集合
`TSet` 使用值本身作为键（**键值合一**）。`TSet` 可以非常快速地添加、查找和删除元素（恒定时间）。**默认情况下，`TSet` 不支持重复的键（添加重复的键会覆盖旧的键），但使用模板参数可激活此行为。**

**同质容器：所有元素均为完全相同类型**。

`TSet` 是一种快速容器类，用于在排序不重要的情况下存储唯一元素。在大多数情况下，只需要一种参数——元素类型。但是，`TSet` 可配置各种模板参数来改变其行为，使其更全面。除了可指定从 `DefaultKeyFuncs` 的派生结构体来提供散列功能，还可允许集合中的多个键拥有相同的值。它和其它容器类一样，可设置自定义内存分配器来存储数据。

`TSet` 也是**值类型**，支持常规复制、赋值和析构函数操作，以及其元素较强的所有权。`TSet` 被销毁时，其元素也将被销毁。键类型也必须是值类型。

`TSet` 使用散列，即如果给出了 `KeyFuncs` 模板参数，该参数会告知集合如何从某个元素确定键，如何比较两个键是否相等，如何对键进行散列，以及是否允许重复键。它们默认只返回对键的引用，使用 `运算符==` 对比相等性，使用非成员函数 `GetTypeHash` 进行散列。默认情况下，集合中不允许有重复的键。如果您的键类型支持这些函数，则可以将其用作集合键，无需提供自定义 `KeyFuncs`。要写入自定义 `KeyFuncs`，可扩展 `DefaultKeyFuncs` 结构体。

最后，`TSet` 可通过任选分配器控制内存分配行为。标准虚幻引擎4（UE4）分配器（如 `FHeapAllocator` 和 `TInlineAllocator`）不能用作 `TSet` 的分配器。实际上，`TSet` 使用集合分配器，该分配器可定义集合中使用的散列桶数量以及用于存储元素的标准UE4分配器。请参见 `TSetAllocator` 了解更多信息。

与 `TArray` 不同的是，内存中 `TSet` 元素的相对排序既不可靠也不稳定，对这些元素进行迭代很可能会使它们返回的顺序和它们添加的顺序有所不同。这些元素也不太可能在内存中连续排列。集合中的后台数据结构是稀疏数组，即在数组中有空位。从集合中移除元素时，稀疏数组中会出现空位。将新的元素添加到阵列可填补这些空位。但是，即便 `TSet` 不会打乱元素来填补空位，指向集元素的指针仍然可能失效，因为如果存储器被填满，又添加了新的元素，整个存储可能会重新分配。

### 创建和填充集合

-  `TSet` 的创建方法如下：
```c++
    TSet<FString> FruitSet;
```

这会创建一个空白 `TSet`，用于存储 `FString` 数据。`TSet` 会直接使用 `运算符==` 比较元素，使用 `GetTypeHash` 对其进行散列，然后使用标准的堆分配器。此时尚未分配内存。

-  **`Add`**：填充集合，提供键（元素）：
```c++
FruitSet.Add(TEXT("Banana"));
FruitSet.Add(TEXT("Grapefruit"));
FruitSet.Add(TEXT("Pineapple"));
// FruitSet == [ "Banana", "Grapefruit", "Pineapple" ]
```

此处的元素按插入顺序排列，但不保证这些元素在内存中实际保留此排序。如果是新集合，可能会保留插入排序，但插入和删除的次数越多，新元素不出现在末尾的可能性越大。

-  **`Emplace`**：和 `TArray` 一样，还可使用 **`Emplace`** 代替 `Add`，避免插入集合时创建临时文件。与 `TArray` 不同的是，只能使用单一参数构造函数将元素放到集合中。
```c++
FruitSet.Emplace(TEXT("Orange"));
// FruitSet == [ "Banana", "Grapefruit", "Pineapple", "Pear", "Orange" ]
```

-  **`Append`** ：合并来插入另一个集合中的所有元素：
```c++
TSet<FString> FruitSet2;
FruitSet2.Emplace(TEXT("Kiwi"));
FruitSet2.Emplace(TEXT("Melon"));
FruitSet2.Emplace(TEXT("Mango"));
FruitSet2.Emplace(TEXT("Orange"));
FruitSet.Append(FruitSet2);
// FruitSet == [ "Banana", "Grapefruit", "Pineapple", "Pear", "Orange", "Kiwi", "Melon", "Mango" ]
```

### 编辑UPROPERTY TSet

如果用 `UPROPERTY` 宏和一个可编辑的关键词（`EditAnywhere`、`EditDefaultsOnly` 或 `EditInstanceOnly`）标记 `TSet`，则可在虚幻编辑器中添加和编辑元素。

```c++
UPROPERTY(Category = SetExample, EditAnywhere)
TSet<FString> FruitSet;
```

### 遍历

`TSet` 的遍历类似于 `TArray`。可使用 C++的设置 for 循环：
```c++
for (auto& Elem :FruitSet)
{
    FPlatformMisc::LocalPrint(
        *FString::Printf(
            TEXT(" \"%s\"\n"),
            *Elem
        )
    );
}
// Output:
//  "Banana"
//  "Grapefruit"
//  "Pineapple"
//  "Pear"
//  "Orange"
//  "Kiwi"
//  "Melon"
//  "Mango"
```

也可以用 `CreateIterator` 和 `CreateConstIterators` 函数来创建迭代器。
-  `CreateIterator` 返回拥有读写访问权限的迭代器
-  `CreateConstIterator` 返回拥有只读访问权限的迭代器。
- 无论哪种情况，均可用这些迭代器的 `Key` 和 `Value` 来检查元素。

```c++
for (auto It = FruitSet.CreateConstIterator(); It; ++It)
{
    FPlatformMisc::LocalPrint(
        *FString::Printf(
            TEXT("(%s)\n"),
            *It
        )
    );
}
```

### 查询

-  **`Num`** ：查询集合中保存的元素数量：

```c++
    int32 Count = FruitSet.Num();
    // Count == 8
```

-  **`Contains`** ：确定集合是否包含特定元素

```c++
bool bHasBanana = FruitSet.Contains(TEXT("Banana"));
bool bHasLemon = FruitSet.Contains(TEXT("Lemon"));
// bHasBanana == true
// bHasLemon == false
```

- 使用 **`FSetElementId`** 结构体可**查找集合中某个键的索引**。然后，就可使用该索引与 `运算符[]` 查找元素。
    - 在非常量集合上调用 `operator[]` 将返回非常量引用，而在常量集合上调用将返回常量引用。

```c++
FSetElementId BananaIndex = FruitSet.Index(TEXT("Banana"));
// BananaIndex is a value between 0 and (FruitSet.Num() - 1)
FPlatformMisc::LocalPrint(
    *FString::Printf(
        TEXT(" \"%s\"\n"),
        *FruitSet[BananaIndex]
    )
);
// Prints "Banana"

FSetElementId LemonIndex = FruitSet.Index(TEXT("Lemon"));
// LemonIndex is INDEX_NONE (-1)
FPlatformMisc::LocalPrint(
    *FString::Printf(
        TEXT(" \"%s\"\n"),
        *FruitSet[LemonIndex]
    )
); // Assert!
```

如果不确定集合中是否包含某个键，可使用 `Contains` 函数和 `运算符[]` 进行检查。但这并非理想的方法，因为同一键需要进行两次查找才能获取成功。

-  **`Find`** 函数查找一次即可完成这些行为。
- 如果中包含该键，`Find` 将返回指向元素数值的指针。
- 如果不包含该键，则返回 null。
- 对常量集合调用 `Find`，返回的指针也将为常量。

```c++
    FString* PtrBanana = FruitSet.Find(TEXT("Banana"));
    FString* PtrLemon = FruitSet.Find(TEXT("Lemon"));
    // *PtrBanana == "Banana"
    //  PtrLemon == nullptr
```

- **`Array`** ：会返回一个 `TArray`，其中填充了 `TSet` 中每个元素的一份副本。被传递的数组在填入前会被清空，因此元素的生成数量将始终等于集合中的元素数量：

```c++
TArray<FString> FruitArray = FruitSet.Array();
// FruitArray == [ "Banana","Grapefruit","Pineapple","Pear","Orange","Kiwi","Melon","Mango" ] (order may vary)
```

### 移除

-  **`Remove`** 函数可**按索引移除元素**，但仅建议在通过元素迭代时使用：Remove 函数会返回已删除元素的数量。
    - 如果给定的键未包含在集合中，则会返回0。
    - 如果 `TSet` 支持重复的键，`Remove` 将移除所有匹配元素。
    - 移除元素将在数据结构中留下空位
```c++
FruitSet.Remove(0);
// FruitSet == [ "Grapefruit","Pineapple","Pear","Orange","Kiwi","Melon","Mango" ]
```

```c++
int32 RemovedAmountPineapple = FruitSet.Remove(TEXT("Pineapple"));
// RemovedAmountPineapple == 1
// FruitSet == [ "Grapefruit","Pear","Orange","Kiwi","Melon","Mango" ]
FString RemovedAmountLemon = FruitSet.Remove(TEXT("Lemon"));
// RemovedAmountLemon == 0
```

-  **`Empty`** 或 **`Reset`** ：将集合中的所有元素移除。
    -  `Empty` 和 `Reset` 相似，但 `Empty` 可采用参数指示集合中保留的 slack 量 `
```c++
    TSet<FString> FruitSetCopy = FruitSet;
    // FruitSetCopy == [ "Grapefruit","Pear","Orange","Kiwi","Melon","Mango" ]

    FruitSetCopy.Empty();
    // FruitSetCopy == []
```

### 排序

`TSet` 可以排序。排序后，迭代集合会以排序的顺序显示元素，但下次修改集合时，排序可能会发生变化。由于排序不稳定，可能按任何顺序显示集合中支持重复键的等效元素。

**`Sort`**： 函数使用指定排序顺序的二进制谓词
```c++
FruitSet.Sort([](const FString& A, const FString& B) {
    return A > B; // sort by reverse-alphabetical order
});
// FruitSet == [ "Pear", "Orange", "Melon", "Mango", "Kiwi", "Grapefruit" ] (order is temporarily guaranteed)

FruitSet.Sort([](const FString& A, const FString& B) {
    return A.Len() < B.Len(); // sort strings by length, shortest to longest
});
// FruitSet == [ "Pear", "Kiwi", "Melon", "Mango", "Orange", "Grapefruit" ] (order is temporarily guaranteed)
```

### 运算符

和 `TArray` 一样，`TSet` 是常规值类型，可通过标准复制构造函数或赋值运算符进行复制。因为集合严格拥有其元素，复制集合的操作是深层的，所以新集合将拥有其自身的元素副本：

```c++
TSet<FString> NewSet = FruitSet;
NewSet.Add(TEXT("Apple"));
NewSet.Remove(TEXT("Pear"));
// FruitSet == [ "Pear", "Kiwi", "Melon", "Mango", "Orange", "Grapefruit" ]
// NewSet == [ "Kiwi", "Melon", "Mango", "Orange", "Grapefruit", "Apple" ]
```

### Slack

Slack是不包含元素的已分配内存。调用 `Reserve` 可分配内存，无需添加元素；通过非零slack参数调用 `Reset` 或 `Empty` 可移除元素，无需将其使用的内存取消分配。Slack优化了将新元素添加到集合的过程，因为可以使用预先分配的内存，而不必分配新内存。它在移除元素时也十分实用，因为系统不需要将内存取消分配。在清空希望用相同或更少的元素立即重新填充的集合时，此方法尤其有效。

`TSet` 不像 `TArray` 中的 `Max` 函数那样可检查预分配元素的数量。

以下代码可在不取消任何内存的情况下移除集合中的所有元素，从而产生slack：

```c++
    FruitSet.Reset();
    // FruitSet == [ <invalid>, <invalid>, <invalid>, <invalid>, <invalid>, <invalid> ]
```

使用 `Reserve` 函数可直接创建slack，例如在添加元素之前预分配内存。

```c++
    FruitSet.Reserve(10);
    for (int32 i = 0; i < 10; ++i)
    {
        FruitSet.Add(FString::Printf(TEXT("Fruit%d"), i));
    }
    // FruitSet == [ "Fruit9", "Fruit8", "Fruit7" ..."Fruit2", "Fruit1", "Fruit0" ]
```

预先分配slack会导致以倒序添加新元素。与数组不同，集合不维护元素排序，处理集合的代码不能指望元素排序稳定或可预测。

使用 `Collapse` 和 `Shrink` 函数可移除 `TSet` 中的全部slack。`Shrink` 将从容器的末端移除所有slack，但这会在中间或开始处留下空白元素。

```c++
    // Remove every other element from the set.
    for (int32 i = 0; i < 10; i += 2)
    {
        FruitSet.Remove(FSetElementId::FromInteger(i));
    }
    // FruitSet == ["Fruit8", <invalid>, "Fruit6", <invalid>, "Fruit4", <invalid>, "Fruit2", <invalid>, "Fruit0", <invalid> ]

    FruitSet.Shrink();
    // FruitSet == ["Fruit8", <invalid>, "Fruit6", <invalid>, "Fruit4", <invalid>, "Fruit2", <invalid>, "Fruit0" ]
```

在上述代码中，`Shrink` 只删除了一个无效元素，因为末端只有一个空元素。要移除所有slack，首先应调用 `Compact` 或 `CompactStable` 函数，将空白空间组合在一起，为调用 `Shrink` 做好准备。顾名思义，`CompactStable` 可在合并空元素时保持元素的排序。

```c++
    FruitSet.CompactStable();
    // FruitSet == ["Fruit8", "Fruit6", "Fruit4", "Fruit2", "Fruit0", <invalid>, <invalid>, <invalid>, <invalid> ]
    FruitSet.Shrink();
    // FruitSet == ["Fruit8", "Fruit6", "Fruit4", "Fruit2", "Fruit0" ]
```

### DefaultKeyFuncs

只要类型具有 `运算符==` 和非成员 `GetTypeHash` 重载，就可为TSet所用，因为此类型既是元素又是键。然而，不便于重载这些函数时可将类型作为键使用。在这些情况下，可对 `DefaultKeyFuncs` 进行自定义。为键类型创建 `KeyFuncs`，必须定义两个typedef和三个静态函数，如下所示：

- `KeyInitType` —— 用于传递键的类型。通常抽取自ElementType模板参数。
    
- `ElementInitType` —— 用于传递元素的类型。同样通常抽取自ElementType模板参数，因此与KeyInitType相同。
    
- `KeyInitType GetSetKey(ElementInitType Element)`——返回元素的键。在集合中，通常是元素本身。
    
- `bool Matches(KeyInitType A, KeyInitType B)` —— 如果 `A` 和 `B` 等值将返回 `true`，否则返回 `false`。
    
- `uint32 GetKeyHash(KeyInitType Key)` —— 返回 `Key` 的散列值。
    

`KeyInitType` 和 `ElementInitType` 是键/元素类型普通传递惯例的typedef。它们通常为浅显类型的一个值和非浅显类型的一个常量引用。请注意，集合的元素类型也是键类型，因此 `DefaultKeyFuncs` 仅使用一种模板参数 `ElementType` 定义两者。

`TSet` 假定在 `DefaultKeyFuncs` 中使用 `Matches` 进行对比结果为相等的两个项也将在 `KeyFuncs` 的 `GetKeyHash` 中返回相同的值。

`DefaultKeyFuncs` 的默认实现时，此规则也适用于 `运算符==` 和 `GetKeyHash`

### 其他

`CountBytes` 和 `GetAllocatedSize` 函数用于估计内部数组的当前内存使用情况。`CountBytes` 接受 `FArchive` 参数，而 `GetAllocatedSize` 则不接受。这些函数常用于统计报告。

`Dump` 函数接受 `FOutputDevice` 并写出关于集合内容的实现信息。还有一个名为 `DumpHashElements` 的函数，可列出来自所有散列条目的所有元素。这些函数常用于调试。
# 十、字符串
## TCHAR 与 TEXT()
对于大多数情况下，虚幻依靠 `TCHAR` 类型来表示字符。`TEXT()` 宏可用于表示 `TCHAR` 文字。

```c++
MyDogPtr->DogName = FName(TEXT("Samson Aloysius"));
```

## 打印信息
### UE_LOG

Log 作为开发中常用到的功能，可以在任何需要情况下记录程序运行情况。

**查看 Log**：
**Game 模式**
在 Game（打包）模式下，记录 Log 需要在启动参数后加 `-Log`。 
**编辑器模式**
在编辑器下，需要打开 Log 窗口（Window->DeveloperTools->OutputLog）。

```c++
UE_LOG(LogMy, Warning, TEXT("Hell World"));
UE_LOG(LogMy, Warning, TEXT("Show a String %s"),*FString("Hello")) 
UE_LOG(LogMy, Warning, TEXT("Show a Int %d"),100);
```
`UE_LOG` 宏输出 log：
**参数一**： Log 的分类（需要预先定义）。 
**参数二**：类型，有 Log、Warning、Error   三种类型。这三种类型区别是颜色不同，Log 为灰色，Warning 为黄色，Error 为红色。
**参数三**：具体的输出内容为 TEXT，可以根据需要自行构造。

几种常用的符号如下：
1.  `%s` 字符串（FString）
2.  `%d` 整型数据（int32）
3.  `%f` 浮点形（float）

### 打印到屏幕
```c++
GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Red, TEXT("Screen Message"));
```
### 打印矢量
```c++
GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Orange, FString::Printf(TEXT("My Location is: %s"), *GetActorLocation().ToString()));
```

```c++
// 定义打印消息函数以打印到屏幕
#define print(text) if (GEngine) GEngine->AddOnScreenDebugMessage(-1, 1.5, FColor::Green,text)
#define printFString(text, fstring) if (GEngine) GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Magenta, FString::Printf(TEXT(text), fstring))

void ATestActor::BeginPlay()
{
	Super::BeginPlay();
	
	// 标准控制台打印
	UE_LOG(LogTemp, Warning, TEXT("I just started running"));

	// 打印到屏幕
	GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Red, TEXT("Screen Message"));

	// 打印向量
	FVector MyVector = FVector(200,100,900);
	GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Orange, FString::Printf(TEXT("My Location is: %s"), *MyVector.ToString()));

	// 使用上面定义的快捷方式打印消息
	print("Hello Unreal");	
	printFString("My Variable Vector is: %s", *MyVector.ToString());
}
```
## FName

> [!NOTE]
> 简单而言，FName 是无法被修改的字符串，大小写不敏感。从语义上讲，名字也应该是唯一的。不管同样的字符串出现了多少次，在字符串表里只被存储一次。而借助这个哈希表，从字符串到 FName 的转换，以及根据 Key 查询 FName 会变得非常快速。

[虚幻引擎中的FName | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/fname-in-unreal-engine/)

`FName` 是**对全局字符串表中不可变且不区分大小写的字符串的引用**。相较于 FString，它的大小更小，更能高效的传递，但更难以操控。

在 **内容浏览器中为新资源命名时** ，**变更动态材质实例中的参数**或**访问骨骼网格体中的一块骨骼时**需要使用 **`FName`**。 FName 通过一个轻型系统使用字符串。在此系统中，特定字符串即使会被重复使用，在数据表中也只存储一次。

`FName` 不区分大小写。它们为不可变，无法被操作。FNames 的存储系统和静态特性决定了通过键进行 FNames 的查找和访问速度较快。 FName 子系统的另一个功能是使用散列表为 FName 转换提供快速字符串。

## FText

> [!NOTE]
> FText 表示一个“被显示的字符串”。所有你希望“显示”的字符串都应该是 FText。因为 FText 提供了内置的本地化支持，也通过一张查找表来支持运行时本地化。FText 不提供任何的更改操作，对于被显示的字符串来说，“修改”是一个非常不安全的操作。

[虚幻引擎中的FText | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/ftext-in-unreal-engine/)

- `FText` 是指定**用于处理本地化的更可靠的字符串表示**。

在 **虚幻引擎** 中，[文本本地化](https://docs.unrealengine.com/5.2/zh-CN/text-localization-in-unreal-engine)的主要组件是 `FText` 类。此类通过提供下列功能支持文本本地化，**因此面向用户的所有文本都应使用此类：**

- [创建本地化的文本文字。](https://docs.unrealengine.com/5.2/zh-CN/text-localization-in-unreal-engine)
- [设置文本格式](https://docs.unrealengine.com/5.2/zh-CN/text-localization-in-unreal-engine)（根据占位符模式生成文本）。
- [根据数字生成文本。](https://docs.unrealengine.com/5.2/zh-CN/text-localization-in-unreal-engine)
- [根据日期时间生成文本。](https://docs.unrealengine.com/5.2/zh-CN/text-localization-in-unreal-engine)
- [生成衍生文本](https://docs.unrealengine.com/5.2/zh-CN/text-localization-in-unreal-engine)，如将文本设为大写或小写。

`Ftext` 同时具有 `AsCultureInvariant` 函数（或 `INVTEXT` 宏），可创建非本地化的（即"语言不变"）文本。这在进行如将玩家名从外部API转换为可在用户界面显示的文本等操作时，十分有用。

可使用 `FText::GetEmpty()` 或仅使用 `FText()`,创建空白 `FText`。

## FString

> [!NOTE]
> FString 是唯一提供修改操作的字符串类。同时也意味着 FString 的消耗要高于 FName 和 FText。
> 事实上，一般我们都使用 FString 来传递。尽管如此，Slate 控件的文字参数往往是 FText。这是为了强制要求本地化。

[虚幻引擎中的FString | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/fstring-in-unreal-engine/)
  
`FString` 是典型的"动态字符数组"字符串类型。

与 `FName` 和 `FText` 不同，`FString` 可以与搜索、修改并且与其他字符串比较。不过，这些操作会导致 `FString` 的开销比不可变字符串类更大。这是因为 **`FString` 对象保存自己的字符数组，而 `FName` 和 `FText` 对象保存共享字符数组的指针，并且可以完全根据索引值建立相等性。**


# 十一、委托 UDELEGATE
**委托** 是一种**泛型但类型安全**的方式，可在 C++对象上调用成员函数。可使用委托动态绑定到任意对象的成员函数，之后在该对象上调用函数，即使调用程序不知对象类型也可进行操作。

- 复制委托对象很安全。你也可以利用值传递委托，但这样操作需要在堆上分配内存，因此通常并不推荐。**请尽量通过引用 `&` 传递委托**。
- **委托函数**使用 **`UDELEGATE` 宏**，支持与 `UFUNCTION` 相同的说明符。
## 1 使用方法

**发送者步骤：**
1. 声明委托，定义参数列表【可以简单理解为构建一个委托类】
2. 声明委托对象【理解为实例化一个类】
3. 在需要的位置触发委托（执行或者广播），需要传入参数【相当于发送消息】

**接收者步骤：**
1. 在其他类中需要接收消息位置，绑定实际执行函数【接收消息，并且根据消息内容执行绑定的操作】
2. 实现具体的执行函数内容【被触发后，实际执行的内容】

---

**例子**：为角色添加一个血条 component，通过血条 component 接收血条变化并将其广播给角色本身，让角色接收消息做出一定反应，比如扣血和死亡
**发送者**：血条组件
**观察者（接收者）**：角色

1. 先在头文件上面声明一个委托, 名字要以 F 开头（位置：HealthComponent. h)
```c++ file:HealthComponent.h
//声明一个动态广播委托
//第一个参数事件的名称（自己定义）
//后面的参数是事件所需要的参数
//因为有6个参数 所以添加_SixParams
DECLARE_DYNAMIC_MULTICAST_DELEGATE_SixParams(FOnHealthChangeSignature, USHeathComponent*, HealthComponent, float, Health, float, HealthDelta, const class UDamageType*, DamageType, class AController*, InstigatedBy, AActor*, DamageCauser);
```

2. 声明一个委托实例（位置：HealthComponent. h)
```c++ file:HealthComponent.h
public:
//创建事件实例
//属性暴露给蓝图
UPROPERTY(BlueprintAssignable, Category = "Events")
FOnHealthChangeSignature OnHealthChange;
```

3. 在需要的位置触发广播（位置：HealthComponent. cpp)
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
USHeathComponent* HealthComponent; //观察者引用发送者

​
//要绑定的函数参数要和声明委托的参数一致
//委托函数使用`UDELEGATE` 宏，支持与 `UFUNCTION` 相同的说明符。
//委托参数的变量类型后面要加 `,` 与变量命分隔开。和常规函数不同
UFUNCTION()
void OnHealthChange(USHeathComponent* HealthComponent, float Health, float HealthDelta, const class UDamageType* DamageType, class AController* InstigatedBy, AActor* DamageCauser);

```

```c++ file:Character.cpp
//在创建HealthComponent对象后，绑定函数OnHealthChange函数到其委托OnHealthChange上（名字可以不相同），OnHealthChange函数执行具体的内容
HealthComponent->OnHealthChange.AddDynamic(this, &ASCharacter::OnHealthChange);
```

解释下，这样使用**委托的好处**：血条组件中使用 `OnHealthChange` 委托，那么 Character 和其他 Actor 使用血条组件时，只有单方面的引用（**观察者引用发送者**。很好理解，观察者要监视发送者）（观察者设计模式原理），血条组件并不需要关心谁用了它，也就是血条组件无需包含使用它的 Character 或者 Actor 的引用，这就实现了一种解耦，这就是委托的好处。

**进一步理解，发送者只管发送，不必直到谁会接收。观察者只需要监听发送者并定义自己收到消息后要执行的行为。** 举例来说，主角释放了一个技能，释放时主角类发出一个委托，受该技能影响的其他类接受这个委托并定义被影响的效果。主角类并不需要直到谁会受到影响。

## 2 单播委托
只能绑定一个函数指针，单播执行时只能触发一个绑定函数（如果绑定多个，只能触发最后绑定的那个函数）
### 声明
单播委托可以有参数或无参数，有返回值或无返回值

当前，支持以下使用任意组合的委托签名：
- 返回一个值的函数。
- 声明为 `const` 函数。
- 最多4个**载荷变量**（调用时被直接传到绑定函数的变量）。
- 最多8个函数参数。

使用此表格查找要用于声明委托的生命宏。
- `Param`：形参
- `RetVal` ：返回值
```c++
//无参，无返回值
DECLARE_DELEGATE(DelegateName)
//有参，无返回值
DECLARE_DELEGATE_OneParam(DelegateName, Param1Type) //一个参数
DECLARE_DELEGATE_TwoParams(DelegateName, Param1Type, Param2Type) //两个参数
DECLARE_DELEGATE_<Num>Params(DelegateName, Param1Type, Param2Type, ...)//多参
//无参，有返回值
DECLARE_DELEGATE_RetVal(RetValType, DelegateName)
//有参有返回值
DECLARE_DELEGATE_RetVal_OneParam(RetValType, DelegateName, Param1Type)
DECLARE_DELEGATE_RetVal_TwoParams(RetValType, DelegateName, Param1Type, Param2Type)
DECLARE_DELEGATE_RetVal_<Num>Params(RetValType, DelegateName, Param1Type, Param2Type, ...)
```

声明了委托后，只是相当于编程声明了一个类，还要对其进行 "实例化"，即声明委托对象。

### 绑定/解绑

| 函数                                                              | 描述                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|:----------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Bind`                                                          |绑定到现有委托对象。|
| `BindStatic`                                                    |绑定原始C++对象static(全局)函数委托。 |
|`BindRaw`|绑定原始C++对象的成员函数。<div>注意原始指针不使用任何类型的引用，删除目标对象后调用`Execute&nbsp;`或`&nbsp;ExecuteIfBound` 会不安全。</div>|
|`BindLambda` |绑定一个Lambda函数。 |
|`BindWeakLambda`|绑定弱引用Lambda函数|
| `BindSP`                                                        |绑定**共享指针/共享引用**指向的原始C++对象的成员函数。<div>共享指针委托会保留对象的弱引用。可使用`ExecuteIfBound()`进行调用。</div> |
| <span style="color: rgb(15, 15, 15);">`BindThreadSafeSP`</span> |绑定线程安全的**共享指针/共享引用**指向的原始C++对象的成员函数。|
| `BindUObject`                                                   |绑定`UObject` 对象的成员函数。<div>委托保留 `UObject`对象的弱引用。可使用`ExecuteIfBound()`进行调用。</div>|
|  `BindFunction`                                                 |绑定 `UObject` 对象的 `UFunction` 成员函数|
| `UnBind`                                                        |取消绑定此委托。|

### 执行

通过调用委托的 `Execute()` 函数执行绑定到委托的函数。执行前须用 `IsBound()` 检查委托是否已绑定。
此操作是为了使代码更安全，因为有时委托可能含有未初始化且被后续访问的返回值和输出参数。执行未绑定的委托在某些情况下确实可能导致内存混乱。

|执行函数 |描述 |
|---|---|
| `IsBound` |检查一个委托是否已绑定，经常出现在包含 `Execute` 调用的代码前|
|`Execute`|执行一个委托|
|`ExecuteIfBound`|检查一个委托是否已绑定，如是，则调用Execute **(只能用于无返回值的委托)** |

## 3 多播委托
**可以绑定到多个函数并一次性同时执行它们的委托。**

多播委托拥有大部分与单播委托相同的功能。它们只拥有对对象的弱引用，可以与结构体一起使用，可以四处轻松复制等等。  
就像常规委托一样，多播委托可以远程加载/保存和触发；

**多播委托函数不能使用返回值**。。

###  声明

```c++
//无参无返回值
DECLARE_MULTICAST_DELEGATE(DelegateName);

//有参无返回值
DECLARE_MULTICAST_DELEGATE_ONEPARAM(DelegateName, Param1Type); //一个参数
DECLARE_MULTICAST_DELEGATE_<Num>PARAMS(DelegateName, Param1Type,...); //多参
```

### 绑定/移除

多播委托可以绑定多个函数，当委托触发时，将调用所有这些函数。因此，绑定函数在语义上与数组更加类似。

|函数 |说明 |
|---|---|
|`Add()`|将函数委托添加到该多播委托的调用列表中。|
| `AddStatic()` |添加原始 C++对象的 static(全局)函数 |
|`AddRaw()`|添加原始 C++对象的成员函数。<div>原始指针不使用任何类型的引用，因此如果从委托下面删除了对象，则调用此函数可能不安全。调用 `Execute()` 时请小心！</div>|
| `AddSP()` |添加**共享指针/共享引用**指向的原始 C++对象的成员函数。<div>共享指针委托保留对对象的弱引用。</div> |
|`AddThreadSafeSP()`|添加线程安全**共享指针/共享引用**指向的原始 C++对象的成员函数。<div>共享指针委托保留对对象的弱引用。</div> |
| `AddUObject()` |添加 `UObject` 对象的成员函数。<div>UObject 委托保留对对象的弱引用。</div> |
| `AddFunction()` |添加 `UObject` 对象的 `UFunction` 成员函数 |
| `AddLambda()` |添加一个 Lambda 函数|
|`AddWeakLambda()`|添加一个弱引用 Lambda 函数|

|函数 |说明 |
|---|---|
|`Remove()` |从该多播委托的调用列表中删除函数（性能为O(N)）。请注意，委托的顺序可能不会被保留！ |
|`RemoveAll()`|从该多播委托的调用列表中删除绑定到指定UserObject的所有函数。请注意，委托的顺序可能不会被保留！|

`RemoveAll()`将删除绑定到所提供指针的所有已注册委托。切记，未绑定到对象指针的原始委托不会被该函数所删除！

### 广播

- 多播委托允许您附加多个函数委托，然后通过调用多播委托的 `Broadcast()` 函数一次性同时执行它们。
- 无返回值
- 在多播委托上调用 `Broadcast()` 总是安全的，即使是在没有任何绑定时也是如此。
- 唯一需要注意的是，如果您使用委托来初始化输出变量，通常会带来非常不利的后果。调用 `Broadcast()` 时绑定函数的执行顺序尚未定义。执行顺序可能与函数的添加顺序不相同。

|函数|说明|
|---|---|
| `Broadcast()`  |将该委托广播给所有绑定的对象，但可能已过期的对象除外。|
## 4 事件 Event
#Event
**事件 Event 是一种特殊类型的多播委托，它在访问 `Broadcast()` 、`IsBound()` 和 `Clear()` 函数时会受到限制。** 
![[c8c18a5716876d4db31a9b76687a1c4c_MD5.jpg]]

### 声明
```c++
//无参无返回值
DECLARE_EVENT(OwningType, EventName);

//有参无返回值
DECLARE_EVENT_ONEPARAM(OwningType, DelegateName, Param1Type); //一个参数
DECLARE_EVENT_<Num>PARAMS(OwningType, DelegateName, Param1Type,...); //多参
```

**注意 1**：事件第一个参数为  '拥有'此事件的类型 B，即类型 B 为事件的友元类，可以访问事件中的私有成员，例如 broadcast 函数

**注意 2：** **定时事件宏一般放在一个类的内部，即访问该类型时需要带上所在类的名称前缀**，如 `（ATPSProjectCharacter::FCharacterEvent）`, 可以有限减少事件类型名称冲突

### 绑定和广播
和多播相同
## 5 动态委托
- **可序列化且支持反射，令其可以在蓝图中进行调用。
- 绑定的成员函数所属的类**必须继承自 UObject**
- 其函数可按命名查找，但其执行速度比常规委托慢。
- **动态委托**可以作为函数参数使用，只有**动态多播委托**可以作为变量给蓝图绑定和调用
### 声明

```c++
//动态单播（可以有返回值）
//无参，无返回值
DECLARE_DYNAMIC_DELEGATE(DelegateName) 
//有参，无返回值
DECLARE_DYNAMIC_DELEGATE_<Num>Params(DelegateName, Param1Type, ...)//多参
//无参，有返回值
DECLARE_DYNAMIC_DELEGATE_RetVal(RetValType, DelegateName)
//有参有返回值
DECLARE_DYNAMIC_DELEGATE_RetVal_<Num>Params(RetValType, DelegateName, Param1Type, Param2Type, ...)

//动态多播（无返回值）
//无参无返回值
DECLARE_DYNAMIC_MULTICAST_DELEGATE(DelegateName);
//有参，无返回值
DECLARE_DYNAMIC_MULTICAST_DELEGATE_<Num>Params(DelegateName, Param1Type, ...)//多参
```

### 绑定/移除

|动态单播 |说明|
|---|---|
|`BindDynamic( UserObject, FuncName )` |用于在**动态单播委托**上调用BindDynamic()的辅助宏。自动生成函数命名字符串。|
|`BindFunction( UserObject, FuncName )` |绑定`UObject`对象的`UFunction`成员函数|
|`Clear`|解绑，内部调用`Unbind`|

|动态多播|说明|
|---|---|
|`AddDynamic( UserObject, FuncName )`|用于在**动态多播委托**上调用AddDynamic()的辅助宏。自动生成函数命名字符串。|
| `AddUniqueDynamic( UserObject, FuncName )` ||
|`RemoveDynamic( UserObject, FuncName )`|用于在动态多播委托上调用RemoveDynamic()的辅助宏。自动生成函数命名字符串。|

在 ue4 中封装了较为简单的宏，通过宏拆解出函数名，后面可以通过存入的函数名来反射找到对应类对应的函数：
```c++
#define BindDynamic( UserObject, FuncName ) __Internal_BindDynamic( UserObject, FuncName, STATIC_FUNCTION_FNAME( TEXT( #FuncName ) ) )
 
#define AddDynamic( UserObject, FuncName ) __Internal_AddDynamic( UserObject, FuncName, STATIC_FUNCTION_FNAME( TEXT( #FuncName ) ) )
```


### 执行

通过调用委托的 `Execute()` 函数执行绑定到委托的函数。执行前须检查委托是否已绑定。此操作是为了使代码更安全，因为有时委托可能含有未初始化且被后续访问的返回值和输出参数。执行未绑定的委托在某些情况下确实可能导致内存混乱。

可调用 `IsBound()` 检查是否可安全执行委托。 同时，对于无返回值的委托，可调用 `ExecuteIfBound()`，但需注意输出参数可能未初始化。

|动态单播 |描述|
|---|---|
| `IsBound` |检查一个委托是否已绑定，经常出现在包含 `Execute` 调用的代码前|
|`Execute`|执行一个委托 |
| `ExecuteIfBound` |检查一个委托是否已绑定，如是，则调用 Execute |

|动态多播|描述|
|---|---|
|`Broadcast` |广播|

### 开放到蓝图
1. 首先在 C++ 中定义对象及动态多播委托
```c++ h:1,8,9,10
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMyDelegate,int32,MyInt);
UCLASS()

class SMARTPTR_API AMyActor : public AActor
{
	GENERATED_BODY()

    //使用BlueprintAssignable说明符
	UPROPERTY(BlueprintAssignable,Category="MyDelegate")
	FMyDelegate MyDelegate;
};
```

2. 在蓝图中绑定即可
![[Pasted image 20230901163755.png|550]]
![[Pasted image 20230901163812.png|550]]

## 6 全局事件派发（鸽）

虽然 UE4 原生提供的委托功能已经满足需求，但是它总归并不是真正意义上全局的事件，**需要监听者知道事件派发者是谁**（甚至需要在 C++ 包含其派发者头文件），**因此我们需要一个中间者来帮我们进行转发事件，监听者和派发者并不需要彼此知道，只要知道中间者就行，从而实现解耦。**

这里我们使用 GameState 作为这个中间者。在 GameState 创建中构建一个委托（当然也可以在 GameInstance，放在 GameState 比较好管理事件生命周期）

### **构建全局事件**

*   为支持蓝图，这里我们选择动态多播构建全局派发事件
*   为方便派发者方便派发事件，这里派发者可以以事件名 FString 直接动态注册和绑定事件

```c++
// 定义委托样式
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FActorTrigger, const FString&, TriggerName);

//绑定事件函数
UFUNCTION(BlueprintCallable, Category = "Event")
void GlobalBindEvent(UserClass* Object, FString TriggerName, typename FActorTrigger::FDelegate::TMethodPtrResolver<UserClass>::FMethodPtr FuncPtr, FName FuncName);

//派发事件
UFUNCTION(BlueprintCallable, Category = "Event")
void GlobalDispatchEvent(FString TriggerName);

//清除所有事件
UFUNCTION(BlueprintCallable, Category = "Event")
void ClearAllEvents();

// 全局绑定事件
template< class UserClass >
void MyGameState::GlobalBindTriggerEvent(UserClass* Object, FString TriggerName, typename FActorTrigger::FDelegate::TMethodPtrResolver<UserClass>::FMethodPtr FuncPtr, FName FuncName) 
{
    FActorTrigger& Trigger = TriggerDelagates.FindOrAdd(TriggerName);
    Trigger.__Internal_AddDynamic(Object, FuncPtr, FuncName);
}

// 全局派发事件
void AMyGameState::GlobalDispatchEvent(FString TriggerName)
{
    if (TriggerName.Len() > 0)
    {
        FPlaceableActorTrigger* Trigger = TriggerDelagates.Find(TriggerName);
        if (Trigger)
        {
            Trigger->Broadcast(TriggerName);
        }
    }
}

// 清除事件
void AMyGameState::ClearAllEvents()
{
    for (auto DeleagtePair : TriggerDelagates)
    {
        DeleagtePair.Value.Clear();
    }
    TriggerDelagates.Empty();


}
```

### **应用全局事件**

```c++
/////////////////// 绑定事件 /////////////////////
#include "UserClass.h"
TriggerEventName = "MyTriggerEvent";
// 获取GameState
AMyGameState* myInstance = Cast<AMyGameState>(GetWorld()->GetGameState());
myInstance->GlobalBindTriggerEvent<UUserClass>(this, TriggerEventName, &UUserClass::OnMyTestFunction, FName(TEXT("OnMyTestFunction")));

/////////////////// 派发事件 /////////////////////
AMyGameState* myGameState = Cast<AMyGameState>(GetWorld()->GetGameState());
myGameState ->GlobalDispatchEvent(TriggerEventName);
```

### **进阶：可传参事件**

上面的事件目前只支持简单的全局事件派发，但是大多数情况我们是需要参数传递的，因此上述方法并不实用，但是 UE 的委托委托蛋疼的地方在于你想要定义不同参数的委托的需要提前宏定义好。因此我们只能取巧。

这里我的解决方案是利用一个容器去存储传递的数据，这样我只要传递当前这个容器的值即可。

*   **结构体**：在结构体可以定义 TMap 参数，然后直接传递结构体变量，缺点是当需要传递多种类型时，我需要存储在多个不同类型的 TMap 中，读取解析麻烦。当然也有解决方法，我可以将结构体参数定义为可变参数的模板类型，但是过程会比较复杂，这里抛砖引玉一下，当然该方法**不支持动态多播**，也即不支持蓝图委托，只能用于普通单播和多播，结构体实现大致如下：

```c++
template <typename... Args>
struct FMultiType {
    std::tuple<Args...> values;


    template<typename T>
    FMultiType(T value): values(value) {}


    template<typename T, typename... Rest>
    FMultiType(T arg, Rest... rest): values(arg, rest...) {}
};
```

*   **Json**：UE5 中内置了 JsonObject 的第三方库，UE4 中也可以自己添加头文件引入 Josn 类。我们可以利用 Json 对象来帮助我们传递任意类型参数  
    大致思路如下：创建一个 JsonObject, 用智能指针管理，然后将数据添加到里面，然后用序列化为 FString 进行数据传输（动态多播也不支持 JsonObject 非 UE 相关类传输），在监听者收到 String 后对其反序化。  
    苦笑：其实这样写也蛮麻烦的，在蓝图端还得写一个 JsonObject 的 Helper 来帮助解析。

```c++
////////////////序列化参数//////////////////
// 创建一个新的 JSON 对象
TSharedPtr<FJsonObject> MyJsonObject = MakeShareable(new FJsonObject);

// 向 JSON 对象中添加字段
MyJsonObject->SetStringField(TEXT("Name"), TEXT("John"));
MyJsonObject->SetNumberField(TEXT("Age"), 25);
MyJsonObject->SetBoolField(TEXT("IsStudent"), true);

// 将 JSON 对象转换为字符串
FString JsonString;
TSharedRef<TJsonWriter<>> Writer = TJsonWriterFactory<>::Create(&JsonString);
FJsonSerializer::Serialize(MyJsonObject.ToSharedRef(), Writer);

///////////////重新定义委托参数///////////////
// 定义委托样式
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FActorTrigger, const FString&, TriggerName, FString&, Prams);

////////////////反序列化参数//////////////////
// 传入参数为JsonString
TSharedPtr<FJsonObject> MyJsonObject;
TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(JsonString);
if (FJsonSerializer::Deserialize(Reader, MyJsonObject) && MyJsonObject.IsValid())
{
    // 反序列化成功
   
    // 获取字符串字段
    FString Name;
    MyJsonObject->TryGetStringField(TEXT("Name"), Name);

    // 获取数值字段
    float Age;
    MyJsonObject->TryGetNumberField(TEXT("Age"), Age);

    // 获取布尔字段
    bool IsStudent;
    MyJsonObject->TryGetBoolField(TEXT("IsStudent"), IsStudent);
}
else
{
    UE_LOG(LogTemp, Log, TEXT("Fail Deserialize JsonObject"));
}
```

*   **FVarient**

如果不考虑蓝图实现，其实这个全局委托事件就可以放飞自我了，我们无需考虑蓝图能接受的类型，比如我们可以使用 UE 内置的**无类型变量 FVarient** 来兼容所有变量的传递（这么好用的参数类型，UE 竟然只允许在 C++ 中使用，可恶啊！！！），直接像这样 `DispatchEvent(FName EventName, TArray<FVariant>& Params)` 的参数传递即可。

**FVarient** 位于引擎 `Core/Misc` 模块中

**FVarient** 的使用方式如下：

```c++
// 存储整数类型的变量
FVariant Var = 123;

// 存储浮点数类型的变量
float FloatValue = 3.14f;
Var = FloatValue;

// 存储字符串类型的变量
FString StringValue = TEXT("Hello World");
Var = StringValue;

// 存储对象指针类型的变量
AActor* ActorPtr = GetWorld()->GetFirstPlayerController()->GetCharacter();
Var = FObjectPtr(ActorPtr);

// 存储枚举类型的变量
EMyEnum MyEnumValue = EMyEnum::Value1;
Var = MyEnumValue;

// 存储结构体类型的变量
FVector Location(1.0f, 2.0f, 3.0f);
Var = Location;

// 存储对象类型的变量
UClass* MyClass = UMyClass::StaticClass();
UMyClass* MyObj = NewObject<UMyClass>();
Var = FSoftObjectPath(MyClass->GetPathName() + TEXT(".") + MyObj->GetName());
```

# 十二、定时器

## 定时器管理

定时器在 **全局定时器管理器**（`FTimerManager` 类型）中管理。全局定时器管理器存在于 **游戏实例** 对象上以及每个 **场景** 中。
**如果要对其调用定时器的对象（如 Actor）在时间结束前被销毁，则相关定时器会自动取消**。在此情况下，定时器句柄将变为无效，并且不会调用该函数。

- **定时器句柄**（**`FTimerHandle`** 类）：可用于暂停（和恢复）倒计时，查询或更改剩余时间，甚至可以取消定时器。
- **访问定时器管理器**：可以使用 `AActor` 函数 **`GetWorldTimerManager()`**，它会在 `UWorld` 中调用 `GetTimerManager()` 函数，来访问 `FTimerManager`。
- **访问全局定时器管理器**：使用 `UGameInstance` 函数 `GetTimerManager()`。如果场景因为任何原因而没有自己的定时器管理器，也可以退而求其次，使用全局定时器管理器。全局管理器可以用于与任何特定场景的存在没有相关性或依赖性的函数调用。
- **使用定时器管理器来设置定时器**：`SetTimer` 和 `SetTimerForNextTick`，它们各自都有一些重载。每个函数都可以连接到任意类型的对象或函数委托，`SetTimer` 可以设为根据需要定期重复。请参阅[定时器管理器 API 页面]( https://docs.unrealengine.com/en-US/API/Runtime/Engine/FTimerManager "FTimerManager")以了解有关这两个函数的更多详细信息。
- 定时器可以与标准的C++函数指针、[`TFunction`对象](https://docs.unrealengine.com/en-US/API/Runtime/Core/GenericPlatform/TFunction "TFunction")或[委托](https://docs.unrealengine.com/5.2/zh-CN/delegates-and-lamba-functions-in-unreal-engine)一起使用。

### 设置和清空定时器
以下函数为 **`FTimerManager`** 类的成员函数，用 `GetWorldTimerManager()` 即可调用：
- **`SetTimer()`** 函数**将定时器设置为在一段延迟后调用函数或委托**，可以设置为不限次重复调用该函数。该函数将填充 **定时器句柄**（`FTimerHandle` 类型）
    - 使用现有定时器句柄调用 `SetTimer` 将清空该定时器句柄引用的定时器，并将它换成新定时器。
- **`SetTimerForNextTick`**： **设置定时器在下一帧运行**，而不是按固定间隔。但需要注意的是，**该函数不填充定时器句柄**。
- **`ClearTimer`：清空定时器**，将 `SetTimer` 期间填充的 `FTimerHandle` 传递到  `ClearTimer` 中。定时器句柄将在此刻失效，并可以再次用于管理新定时器。
- **`ClearAllTimersForObject`** ：**清空所有**与特定对象关联的所有定时器。

示例：
```c++ 
//声明
int32 CountdownTime;
FTimerHandle MemberTimerHandle;
```

```c++
ACountdown::ACountdown()
{
    CountdownTime = 3; //初始化倒计时
}

void ACountdown::BeginPlay()
{
	Super::BeginPlay();
	 // 从现在开始5s后，每秒调用一次AdvanceTimer函数
	GetWorldTimerManager().SetTimer(CountdownTimerHandle, this, &ACountdown::AdvanceTimer, 1.0f, true,5.0f);
}

void ACountdown::AdvanceTimer()
{
    //倒计时结束时，清空定时器
	--CountdownTime;
	if(CountdownTime<=0)
	{
		GetWorldTimerManager().ClearTimer(CountdownTimerHandle);
		// CountdownTimerHandle现在可以复用于其他任意定时器。
	}
}
```

> [!warning] 
>以小于等于0的速率调用`SetTimer`等效于调用`ClearTimer`。

### 暂停和恢复定时器

**`PauseTimer`**：使用定时器句柄来暂停正在运行的定时器。这样可阻止定时器执行其函数调用，但经过的时间和剩余时间将保持暂停时的状态。
**`UnPauseTimer`**：使暂停的定时器恢复运行。

## 定时器信息

除了管理定时器，定时器管理器还提供了**用于获取特定定时器信息的函数，如速率、经过的时间和剩余时间等。**

### 定时器是否活跃

- **`IsTimerActive`**：用于确定指定定时器当前是否活跃且未暂停。

```c++
// 这个武器是否正在等待再次射击？
GetWorldTimerManager().IsTimerActive(this, &AUTWeapon::RefireCheckTimer);
```

### 定时器速率

- **`GetTimerRate`** ：用于从定时器句柄**获取定时器的当前速率**（两次激活之间的时间）。
    - 定时器速率不能直接更改，但可以使用其定时器句柄调用 ` SetTimer ` 来清空定时器并创建新定时器，新定时器除了速率不同，其他保持不变。来模拟更改定时器速率。
    - 如果定时器句柄无效，则 ` GetTimerRate ` 将返回值 ` -1 `。

```c++
// 该武器的射击速率在预热时变化。当前是否正在等待射击，如果是，两次射击之间的当前间隔是多久？
GetWorldTimerManager().GetTimerRate(this, &AUTWeapon::RefireCheckTimer);
```

### 经过时间和剩余时间

- **`GetTimerElapsed`** ：返回与定时器句柄关联的定时器的**经过时间**
-  **`GetTimerRemaining`**：返回与定时器句柄关联的定时器的**剩余时间**
- 如果定时器句柄无效，则这两个函数将返回 `-1`。
- 定时器的**经过时间和剩余时间之和应该等于定时器的速率**。

```c++
// 该武器准备好再次射击之前将经过多长时间？如果答案为-1，则表示现在已准备就绪。
GetWorldTimerManager().GetTimerElapsed(this, &AUTWeapon::RefireCheckTimer);
```


# 十三、断言

**`assert` 的关键特性之一是不存在于发布代码中，这意味着不但不会影响发布产品的性能，也没有任何副作用**。**对 `assert` 最简单的理解就是："断言"必须一律为 true，否则程序会停止运行。**

虚幻引擎提供 `assert` 等同项的三个不同族系：`check`、`verify` 和 `ensure`。各个功能的行为略有不同，但它们都是开发期间使用的诊断工具，目标大致相同。

## Check

Check 族系最接近基础 `assert`，因为当第一个参数得出的值为 false 时，此族系的成员会停止执行，且默认不会在发布版本中运行。以下 Check 宏可用：

```c++
//当Expression为以下值时触发断言
//空指针
//0
//false
check(Expression); 
check(Expression && "Expression can not be NULL"); 

//附加报错文本
checkf(Expression ,TEXT("Expression can not be NULL"))

//代码执行到这一行就触发断言
checkNoEntry();
```

|宏|参数|行为|
|---|---|---|
| `check` 或 `checkSlow` |`Expression`|若 `Expression` 为 false，停止执行|
| `checkf` 或 `checkfSlow` | `Expression`、`FormattedText`、`...` |若 `Expression` 为 false，则停止执行并将 `FormattedText` 输出到日志|
| `checkCode` | `Code` |在运行一次的 do-while 循环结构中执行 `Code`；主要用于准备另一个 Check 所需的信息|
| `checkNoEntry` |（无）|若此行被 hit，则停止执行，类似于 `check(false)`，但主要用于应不可到达的代码路径 |
| `checkNoReentry` |（无）|若此行被 hit 超过一次，则停止执行|
| `checkNoRecursion` |（无）|若此行被 hit 超过一次而未离开作用域（递归），则停止执行 |
| `unimplemented` |（无）|若此行被 hit，则停止执行，类似于 `check(false)`，但主要用于应被覆盖而不会被调用的虚拟函数 |

> [!NOTE] 此行被 hit 的含义
> 这句话意思就是代码执行到这一行

## Verify

**若某个函数执行操作，然后返回 `bool` 来说明该操作是否成功，则应使用 Verify 而非 Check 来确保该操作成功。** 
因为在发布版本中 Verify 将忽略返回值，但仍将执行操作。而 Check 在发布版本中根本不调用该函数，所以行为才会有所不同。

```c++
// 使用Verify而非Check，因为表达式需要先执行操作———设置网格体。
verify((Mesh = GetRenderMesh()) != nullptr);
```

|宏|参数|行为|
|---|---|---|
| `verify` 或 `verifySlow` | `Expression` |若 `Expression` 为 false，停止执行|
|`verifyf` 或 `verifyfSlow`| `Expression`、`FormattedText`、`...` |若 `Expression` 为 false，则停止执行并将 `FormattedText` 输出到日志|

## Ensure

**Ensure 宏的表达式计算得出的值为 false，将报告崩溃，<font color="#ff0000">仍会继续运行（即可以在编辑器点击继续运行）</font>。**

为避免崩溃报告器收到太多通知，Ensure 宏在每次引擎或编辑器会话中仅报告一次。若实际情况需要 Ensure 宏在每次表达式计算得值为 false 时都报告一次，则使用"Always"版本的宏。

```c++
// 这行代码捕获了在产品发布版本中可能出现的小错误。
// 此错误较小，无需停止执行便可解决。
// 虽然该bug已修复，但开发者仍然希望了解之前是否曾经出现过此bug。
void AMyActor::Tick(float DeltaSeconds)
{
    Super::Tick(DeltaSeconds);
    // 确保bWasInitialized为true，然后再继续。若为false，则在日志中记录该bug尚未修复。
    if (ensureMsgf(bWasInitialized, TEXT("%s ran Tick() with bWasInitialized == false"), *GetActorLabel()))
    {
        //（执行一些需要已正确初始化AMyActor的操作。)
    }
}
```

|宏|参数|行为|
|---|---|---|
| `ensure` | `Expression` | `Expression` 首次为 false 时通知崩溃报告器|
| `ensureMsgf` | `Expression`、`FormattedText`、`...` | `Expression` 首次为 false 时通知崩溃报告器并将 `FormattedText` 输出到日志|
| `ensureAlways` | `Expression` | `Expression` 为 false 时通知崩溃报告器|
| `ensureAlwaysMsgf` | `Expression`, `FormattedText`, `...` | `Expression` 为 false 时通知崩溃报告器并将 `FormattedText` 输出到日志|


# GameplayTags
[使用虚幻引擎中的Gameplay标签 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/using-gameplay-tags-in-unreal-engine/)

# 创建蓝图 API：提示和技巧
[在虚幻引擎中将C++暴露给蓝图 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/exposing-cplusplus-to-blueprints-visual-scripting-in-unreal-engine/)
程序员创建对蓝图公开的 API 时需要考虑以下几点：

- 可选参数便于在蓝图中处理：
    
```c++
/**
 * 将字符串显示到日志中，也可选择显示到屏幕上。
 * 如 Print To Log 为 true，它将显示在 Output Log 窗口中。否则它将被记录为"Verbose"，通常不会显示。
 *
 * @param   InString        登出字符串
 * @param   bPrintToScreen  是否将输出显示到屏幕上
 * @param   bPrintToLog     是否将输出保存到日志中
 * @param   bPrintToConsole 是否将输出显示到控制台
 * @param   TextColor       是否将输出显示到控制台
 */
UFUNCTION(BlueprintCallable, meta=(WorldContext="WorldContextObject", CallableWithoutWorldContext, Keywords = "log print", AdvancedDisplay = "2"), Category="Utilities|String")
static void PrintString(UObject* WorldContextObject, const FString& InString = FString(TEXT("Hello")), bool bPrintToScreen = true, bool bPrintToLog = true, FLinearColor TextColor = FLinearColor(0.0,0.66,1.0));
```


- 在带大量返回参数的函数和返回结构体的函数之间优先前者。以下片段显示如何在节点上创建多个输出引脚：
    
```c++
UFUNCTION(BlueprintCallable, Category = "Example Nodes")
static void MultipleOutputs(int32& OutputInteger, FVector& OutputVector);
```
    
- 可在现有函数上添加新参数，但如果要进行移除或变更，则需要否决原始函数并添加一个新函数。必须使用否决元数据，使新函数的信息显示在蓝图中：
    
```c++
UFUNCTION(BlueprintCallable, Category="Collision", meta=(DeprecatedFunction, DeprecationMessage = "Use new CapsuleOverlapActors", WorldContext="WorldContextObject", AutoCreateRefTerm="ActorsToIgnore"))
static ENGINE_API bool CapsuleOverlapActors_DEPRECATED(UObject* WorldContextObject, const FVector CapsulePos, float Radius, float HalfHeight, EOverlapFilterOption Filter, UClass* ActorClassFilter, const TArray<AActor*>& ActorsToIgnore, TArray<class AActor*>& OutActors);
```
    
- 如果函数需要接受枚举，考虑将"expand enum as execs"用作元数据，可使节点更易于使用。
    
```c++
UFUNCTION(BlueprintCallable, Category = "DataTable", meta = (ExpandEnumAsExecs="OutResult", DataTablePin="CurveTable"))
static void EvaluateCurveTableRow(UCurveTable* CurveTable, FName RowName, float InXY, TEnumAsByte<EEvaluateCurveTableResult::Type>& OutResult, float& OutXY);
```
    
- 许多完成耗时较长的操作（如 move here）均为隐藏函数。
    
```c++
/**
 * 执行带延迟的隐藏操作。
 *
 * @param WorldContext  世界背景。
 * @param Duration      延迟长度。
 * @param LatentInfo    隐藏操作。
 */
UFUNCTION(BlueprintCallable, Category="Utilities|FlowControl", meta=(Latent, WorldContext="WorldContextObject", LatentInfo="LatentInfo", Duration="0.2"))
static void Delay(UObject* WorldContextObject, float Duration, struct FLatentActionInfo LatentInfo );
```
    
- 如有可能，考虑将函数放入共享库。便于在多个类之间使用，避开"target"引脚。
    
```c++
class DOCUMENTATIONCODE_API UTestBlueprintFunctionLibrary : public UBlueprintFunctionLibrary
```
    
- 尽可能将节点标记为纯，可避免在节点上使用连线的执行引脚。
    
```c++
/* 在 0 和 最大 - 1 之间返回一致分配的随机数 */
UFUNCTION(BlueprintPure, Category="Math|Random")
static int32 RandomInteger(int32 Max);
```
    
- 将一个函数标记为 `const` 也可使蓝图节点不带执行引脚：
    
```c++
/**
 * 获得 actor 到世界的转换。
 * @return 从 actor 空间转换到世界空间的转换。
 */
UFUNCTION(BlueprintCallable, meta=(DisplayName = "GetActorTransform"), Category="Utilities|Transformation")
FTran
```

# 游戏模块
[使用虚幻引擎中的Gameplay标签 | 虚幻引擎5.2文档 (unrealengine.com)](https://docs.unrealengine.com/5.2/zh-CN/using-gameplay-tags-in-unreal-engine/)