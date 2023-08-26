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
# 属性
## 属性声明

属性使用标准的C++变量语法声明，前面用UPROPERTY宏来定义属性元数据和变量说明符。

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

同样，你可以在 `UPARAM` 标签的meta部分添加 `BitmaskEnum` 和对应的枚举类型名称来定制它。

```c++
/*~ MyOtherFunction shows flags named after the values from EColorBits. */
UFUNCTION(BlueprintCallable)
void MyOtherFunction(UPARAM(meta=(Bitmask, BitmaskEnum = "EColorBits")) int32 ColorFlagsParam)
```

虽然列举类型包含超过32个条目，但在属性编辑器UI中，位掩码关联中只会看到前32个值。同样，虽然可接受显式值条目，但显式值介于0-31的条目不会包含在下拉列表中。

### 浮点类型

虚幻使用标准C++浮点类型、浮点和双精度。

### 布尔类型

布尔类型可以使用C++ ool关键字表示或表示为位域。

```
    uint32 bIsHungry : 1;
    bool bIsThirsty;
```

### 字符串

虚幻引擎4支持三种核心类型的字符串。

- FString是典型的"动态字符数组"字符串类型。
    
- FName是对全局字符串表中不可变且不区分大小写的字符串的引用。相较于FString，它的大小更小，更能高效的传递，但更难以操控。
    
- FText是指定用于处理本地化的更可靠的字符串表示。
    

对于大多数情况下，虚幻依靠TCHAR类型来表示字符。TEXT()宏可用于表示TCHAR文字。

```
    MyDogPtr->DogName = FName(TEXT("Samson Aloysius"));
```

有关这三种字符串类型、何时使用哪个类型以及如何使用它们的更多信息，请参阅[字符串处理文档](https://docs.unrealengine.com/5.2/zh-CN/string-handling-in-unreal-engine)。

## 属性说明符

声明属性时，**属性说明符** 可被添加到声明，以控制属性与引擎和编辑器诸多方面的相处方式。

|属性标签|效果|
|---|---|
|`AdvancedDisplay`|属性将被放置在其出现的任意面板的高级（下拉）部分中。|
|`AssetRegistrySearchable`|`AssetRegistrySearchable` 说明符说明此属性与其值将被自动添加到将此包含为成员变量的所有资源类实例的资源注册表。不可在结构体属性或参数上使用。|
|`BlueprintAssignable`|只能与组播委托共用。公开属性在蓝图中指定。|
|`BlueprintAuthorityOnly`|此属性必须为一个组播委托。在蓝图中，其只接受带 `BlueprintAuthorityOnly` 标签的事件。|
|`BlueprintCallable`|仅用于组播委托。应公开属性在蓝图代码中调用。|
|`BlueprintGetter=GetterFunctionName`|此属性指定一个自定义存取器函数。如此属性不带 `BlueprintSetter` 或 `BlueprintReadWrite` 标签，则其为隐式 `BlueprintReadOnly`。|
|`BlueprintReadOnly`|此属性可由蓝图读取，但不能被修改。此说明符与 `BlueprintReadWrite` 说明符不兼容。|
|`BlueprintReadWrite`|可从蓝图读取或写入此属性。此说明符与 `BlueprintReadOnly` 说明符不兼容。|
|`BlueprintSetter=SetterFunctionName`|此属性拥有一个自定义编译函数，被隐式标记为 `BlueprintReadWrite`。注意：必须对变异函数进行命名，并为相同类的一部分。|
|`Category="TopCategory\|SubCategory\|..."`|指定在蓝图编辑工具中显示时的属性类别。使用 \| 运算符定义嵌套类目。|
|`Config`|此属性将被设为可配置。当前值可被存入与类相关的 `.ini` 文件中，创建后将被加载。无法在默认属性中给定一个值。暗示为 `BlueprintReadOnly`。|
|`DuplicateTransient`|说明在任意类型的复制中（复制/粘贴、二进制复制等），属性的值应被重设为类默认值。|
|`EditAnywhere`|说明此属性可通过属性窗口在原型和实例上进行编辑。此说明符与所有"可见"说明符均不兼容。|
|`EditDefaultsOnly`|说明此属性可通过属性窗口进行编辑，但只能在原型上进行。此说明符与所有"可见"说明符均不兼容。|
|`EditFixedSize`|只适用于动态数组。这能防止用户通过虚幻编辑器属性窗口修改数组长度。|
|`EditInline`|允许用户在虚幻编辑器的属性查看器中编辑此属性所引用的Object的属性（只适用于Object引用，包括Object引用的数组）。|
|`EditInstanceOnly`|说明此属性可通过属性窗口进行编辑，但只能在实例上进行，不能在原型上进行。此说明符与所有"可见"说明符均不兼容。|
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
|`VisibleAnywhere`|说明此属性在所有属性窗口中可见，但无法被编辑。此说明符与"Edit"说明符不兼容。|
|`VisibleDefaultsOnly`|说明此属性只在原型的属性窗口中可见，无法被编辑。此说明符与所有"Edit"说明符均不兼容。|
|`VisibleInstanceOnly`|说明此属性只在实例的属性窗口中可见（在原型属性窗口中不可见），无法被编辑。此说明符与所有"Edit"说明符均不兼容。|

#### 元数据说明符

声明类、接口、结构体、列举、列举值、函数，或属性时，可添加 **元数据说明符** 来控制其与引擎和编辑器各方面的相处方式。每一种类型的数据结构或成员都有自己的元数据说明符列表。

Metadata只存在于编辑器中。请不要编写能够访问到Metadata的游戏逻辑。

|属性元标签|效果|
|---|---|
|`AllowAbstract="true/false"`|用于 `Subclass` 和 `SoftClass` 属性。说明抽象类属性是否应显示在类选取器中。|
|`AllowedClasses="Class1, Class2, .."`|用于 `FSoftObjectPath` 属性。逗号分隔的列表，表明要显示在资源选取器中的资源类类型。|
|`AllowPreserveRatio`|用于 `Fvector` 属性。在细节面板中显示此属性时将添加一个比率锁。|
|`ArrayClamp="ArrayProperty"`|用于整数属性。将可在UI中输入的有效值锁定在0和命名数组属性的长度之间。|
|`AssetBundles`|用于 `SoftObjectPtr` 或 `SoftObjectPath` 属性。主数据资源中使用的束列表命名，指定此引用属于哪个束的一部分。|
|`BlueprintBaseOnly`|用于 `Subclass` 和 `SoftClass` 属性。说明蓝图类是否应显示在类选取器中。|
|`BlueprintCompilerGeneratedDefaults`|属性默认项由蓝图编译器生成，`CopyPropertiesForUnrelatedObjects` 在编译后调用时将不会被复制。|
|`ClampMin="N"`|用于浮点和整数属性。指定可在属性中输入的最小值 `N`。|
|`ClampMax="N"`|用于浮点和整数属性。指定可在属性中输入的最大值 `N`。|
|`ConfigHierarchyEditable`|此属性被序列化为一个配置（`.ini`）文件，可在配置层级中的任意处进行设置。|
|`ContentDir`|由 `FDirectoryPath` 属性使用。说明将使用 `Content` 文件夹中的Slate风格目录选取器来选取路径。|
|`DisplayAfter="PropertyName"`|在蓝图编辑器中，名为 `PropertyName` 的属性后即刻显示此属性。前提是两个属性属于同一类别，则忽略其在源代码中的顺序进行显示。如多个属性有相同的 `DisplayAfter` 值和相同的 `DisplayPriority` 值，将在指定属性之后，按照自身在标头文件中声明的顺序显示。|
|`DisplayName="Property Name"`|此属性显示的命名，不显示代码生成的命名。|
|`DisplayPriority="N"`|如两个属性有相同的 `DisplayAfter` 值，或属于同一类别且无 `DisplayAfter` 元标签，则此属性将决定其顺序。最高优先级值为1，表示 `DisplayPriority` 值为1的属性将在 `DisplayProirity` 值为2的属性之上显示。如多个属性有相同的 `DisplayAfter` 值，其将按照在标头文件中声明的顺序显示。|
|`DisplayThumbnail="true"`|说明属性是一个资源类型，其应显示选中资源的缩略图。|
|`EditCondition="BooleanPropertyName"`|对一个布尔属性进行命名，此属性用于说明此属性的编辑是否被禁用。将"!"放置在属性命名前可颠倒测试。|
|`EditFixedOrder`|使排列的元素无法通过拖拽来重新排序。<br><br>EditCondition元标签不再仅限于单个布尔属性。它现在由完全成熟的算式解析器计算，意味着可以包含一个完整的C++表达式。|
|`ExactClass="true"`|结合 `AllowedClasses` 用于 `FSoftObjectPath` 属性。说明是否只能使用 `AllowedClasses` 中指定的准确类，或子类是否同样有效。|
|`ExposeFunctionCategories="Category1, Category2, .."`|在蓝图编辑器中编译一个函数列表时，指定其函数应被公开的类目的列表。|
|`ExposeOnSpawn="true"`|指定此属性是否应在此类类型的一个Spawn Actor节点上公开。|
|`FilePathFilter="FileType"`|由 `FFilePath` 属性使用。说明在文件选取器中显示的路径过滤器。常规值包括"uasset"和"umap"，但这些并非唯一可能的值。|
|`GetByRef`|使该属性的"Get"蓝图节点返回对属性的常量引用，而不是其值的副本。只对稀疏类数据生效，只能在不存在 `NoGetter` 时使用。|
|`HideAlphaChannel`|用于 `Fcolor` 和 `FLinearColor` 属性。说明详细显示属性控件时 `Alpha` 属性应为隐藏状态。|
|`HideViewOptions`|用于 `Subclass` 和 `SoftClass` 属性。隐藏在类选取器中修改显示选项的功能。|
|`InlineEditConditionToggle`|表示出布尔属性只内联显示为其他属性中的一个编辑条件切换，不应显示在其自身的行上。|
|`LongPackageName`|由 `FDirectoryPath` 属性使用。将路径转换为一个长的包命名。|
|`MakeEditWidget`|用于变换或旋转体属性，或变换/旋转体的排列。说明属性应在视口中公开为一个可移动控件。|
|`NoGetter`|防止蓝图为该属性生成一个"get"节点。只对稀疏类数据生效。|
# 对象
虚幻引擎包含一个用于处理游戏对象的强大系统。虚幻引擎中**所有对象的基类都是 `UObject`**。

**`UCLASS` 宏的作用**是标记 `UObject` 的子类，以便 **UObject** 处理系统可以识别它们。

## 1 UCLASS 宏

**UCLASS** 宏为 `UObject` 提供了一个 `UCLASS` 引用，用于描述它在虚幻引擎中的类型。每个 `UCLASS` 都保留一个称作 **类默认对象（Class Default Object）** 的对象，简称`CDO`。`CDO` 本质上是一个默认"模板"对象，由类构建函数生成，之后就不再修改。

你可以为指定对象获取其 `UCLASS` 和 `CDO`，虽然它们通常都是只读类型。
>使用 **`GetClass()`** 函数即可随时访问对象实例的 UCLASS。

`UCLASS` 包含定义类的一套属性和函数。这些是本地代码可用的普通 C++ 函数和变量，但被虚幻引擎特有的元数据所标记，它们在对象系统中的行为也因此受到控制。 如需了解标记语法的更多细节，请查阅[编程参考](https://docs.unrealengine.com/5.2/zh-CN/programming-with-cpp-in-unreal-engine)。

> [!NOTE]
> `UObject` 类还可包括仅限本地的属性，这些属性没有用 `UFUNCTION` 或者 `UPROPERTY` 指定器标记用于反射。**只有用指定器宏标记过的函数和属性会列举在它们对应的 `UCLASS` 中。**

> [!NOTE] Title
> Contents

## 2 属性和函数类型

`UObjects` 可拥有成员变量（称作属性）或任意类型的函数。然而，为便于虚幻引擎识别并操控这些变量或函数，它们必须以特殊的宏进行标记，并符合一定类型的标准。如需了解这些标准的细节，请查阅 [属性](https://docs.unrealengine.com/5.2/zh-CN/unreal-engine-uproperties)和[UFunction](https://docs.unrealengine.com/5.2/zh-CN/ufunctions-in-unreal-engine)参考页面。

## 3 UObject 创建

**`UObjects` 不支持构造器参数**。所有的 C++ `UObject` 都会在引擎启动的时候初始化，然后引擎会调用其默认构造器。如果没有默认的构造器，那么 `UObject` 将不会编译。

**`UObject` 构造器应该轻量化，仅用于设置默认的数值和子对象，构造时不应该调用其它功能和函数。** 对于 [Actor](https://docs.unrealengine.com/5.2/zh-CN/actors-in-unreal-engine) 和 [Actor组件](https://docs.unrealengine.com/5.2/zh-CN/components-in-unreal-engine)，初始化功能应该输入 **`BeginPlay()`** 方法。

`UObject` 应该仅在运行时使用 NewObject 构建，或者将 CreateDefaultSubobject 用于构造器。

|方法|描述|
|---|---|
|[`NewObject<class>`](https://docs.unrealengine.com/en-US/API/Runtime/CoreUObject/UObject/NewObject "NewObject")|使用所有可用创建选项的可选参数创建一个新实例。提供极高的灵活性，包括带自动生成命名的简单使用案例。|
|`CreateDefaultSubobject<class>`|创建一个组件或者子对象，可以提供创建子类和返回父类的方法。<br><br> 创建默认子对象时，由于它们在引擎启动时构造，UObject 的类构造器应该仅适用于本地数据或者本地加载的静态资产。|

> [!warning]
> **`UObjects` 永远都不应使用 `new` 运算符。所有的 UObjects 都由虚幻引擎管理内存和垃圾回收。如果通过 new 或者 delete 手动管理内存，可能会导致内存出错。**

## 4 UObjects 提供的功能

此系统的使用不为强制要求，甚至有时不适合使用，但却存在以下益处：
- 垃圾回收
- 引用更新

当PendingKill默认禁用时，开发者不需要担心引用更新，这一行可以在该UE版本中移除。
- 反射
- 序列化
- 默认属性变化自动更新
- 自动属性初始化
- 自动编辑器整合
- 运行时类型信息可用
- 网络复制

> [!NOTE]
> 大部分这些益处适用于 [UStruct](https://docs.unrealengine.com/5.2/zh-CN/structs-in-unreal-engine)，它有着和 UObject 一样的反射和序列化能力。UStruct 被当作数值类型处理并且不会垃圾回收。更多关于这些系统的细节，请参考 [虚幻Object处理](https://docs.unrealengine.com/5.2/zh-CN/unreal-object-handling-in-unreal-engine) 文档。

## 5 虚幻头文件工具

为利用 `UObject` 派生类型所提供的功能，需要在头文件上为这些类型执行一个预处理步骤，以核对需要的信息。该预处理步骤由 **UnrealHeaderTool（简称 UHT）** 执行。`UObject` 派生的类型需要遵守特定的结构。

## 6 头文件格式

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

```c++
#include "MyObject.generated.h"
```

此行预计为此文件中最后一个 `#include` 指令。如此头文件需要了解其他类，可将它们在文件中的任意位置提前声明，或包括在 MyObject.generated.h 上。

```c++
UCLASS()
```

`UCLASS` 宏使虚幻引擎能识别 `UMyObject`。此宏支持大量参数[类说明符](https://docs.unrealengine.com/5.2/zh-CN/class-specifiers)，参数决定类功能的开或关。

```c++
class MYPROJECT_API UMyObject : public UObject
```

**如 MyProject 希望将 UMyObject 类公开到其他模块，则需要指定 `MYPROJECT_API`**。这对游戏项目将包括的模块或插件十分实用。这些模块和插件将故意使类公开，在多个项目间提供可携的自含式功能。

```c++
GENERATED_BODY()
```

**`GENERATED_BODY` 宏不获取参数，但会对类进行设置，以支持引擎要求的基础结构。所有 `UCLASS` 和 `USTRUCT` 均有此要求。**

**虚幻头文件工具支持最下C++集。当使用自定义 `#ifdefs` 宏包裹 UCLASS 的部分时，UHT 会忽略不包含 `WITH_EDITOR` 或者 `WITHEDITORONLY_DATA` 宏的宏。**

## 7 更新对象

Ticking 代表虚幻引擎中对象的更新方式。所有Actors均可在每帧被 tick，便于您执行必要的更新计算或操作。

`Actor` 和 Actor 组件在注册时会自动调用它们的 Tick 函数
**`UObjects` 不具有嵌入的更新能力。在必须的时候，可以使用 `inherits` 类说明符从 `FTickableGameObject` 继承即可添加此能力。**  这样即可实现 `Tick()` 函数，引擎每帧都将调用此函数。

注意，无论对象是否被垃圾回收，[Actors](https://docs.unrealengine.com/5.2/zh-CN/actors-in-unreal-engine)对其都没有影响。

## 8 销毁对象

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

## 9 虚幻 Object 处理
使用适当的宏标记类、属性和函数可以将它们转变为 `UClasses`、`UProperties` 和 `UFunctions`。这让虚幻引擎能够访问它们，从而允许实现一些后台处理功能。

### 自动属性初始化

在调用构造函数之前，`UObject`在初始化时自动归零。这针对整个类发生，`UProperties`和类似的原生成员。成员随后可以使用类构造函数中的自定义值进行初始化。

### 自动更新引用

**`AActor` 或 `UActorComponent` 被销毁或者从运行中删除时，对反射系统可见的对它的所有引用（`UProperty` 指针和虚幻引擎容器类中存储的指针，如 `TArray`）都将自动清空。** 这样的好处是防止悬挂指针持久存在并导致后续问题，但也意味着如果其他某段代码将 `AActor` 和 `UActorComponent` 指针销毁，这些指针也会变为空。最终的好处是空检查更可靠，因为会检测标准情况空指针和非空指针指向删除内存的情况。

必须理解的是，**这种功能仅适用于标记了 `UPROPERTY` 或存储在虚幻引擎容器类中的 `UActorComponent` 或 `AActor` 引用。** 存储在原始指针中的 Object 引用对于虚幻引擎将为未知，并且不会自动清空，也不会妨碍垃圾回收。请注意，这不意味着所有 `UObject*` 变量都必须是 `UProperties`。**如果你需要的 Object 指针不是 `UProperty`，请考虑使用 `TWeakObjectPtr`。这是"弱"指针，意味着不会妨碍垃圾回收，但可以查询有效性，然后再接受访问，并且它所指向的 Object 要被销毁时，它将被设置为空。**

另一种被引用UObject UProperty自动清空的情况是对编辑器中的资源使用"强制删除（Force Delete）"。因此，作用于属于资源的UObject的所有代码都必须处理这些变为空的指针。

### 序列化

**当 `UObject` 被序列化时，所有 `UProperty` 值都将被自动写入或读取，除非显式标记为"transient 瞬时"或无法从后构造函数默认值进行更改。** 例如，你可以在关卡中放入 `AEnemy` 实例，将其"体力（Health）"设置为500，保存并成功地重新加载，而不必在 `UClass` 定义之外编写一行代码。

当添加或删除UProperties时，系统会无缝处理加载预先存在的内容。新属性从新的CDO复制默认值。删除的属性将会被静默忽略。

如果需要自定义行为，则可以重载`UObject::Serialize`函数。这对于检测数据错误，检查版本号或执行自动转换或更新（如果数据格式有所更改）十分有用。

### 更新属性值

**当`UClass`的 类默认对象（CDO）更改，引擎将尝试在加载类的所有实例时对这些实例应用这些更改。对于给定Object实例，如果更新的变量值与旧CDO中的值相匹配，则将更新为它在新CDO中保存的值。如果变量包含任何其他值，系统会假设这个值是故意设置的，这些更改将会被保留。**

例如，假设你在一个关卡中放置了多个 `AEnemy` Object并保存，然后将 `AEnemy` 构造函数中的默认Health值设置为100。再假设将Enemy_3的Health值设置为500，因为它们特别难对付。现在，假设你改变注意了，将Health的默认值增加到150。下次加载关卡时，虚幻意识到你更改了CDO，并将使用旧默认Health值（100）的所有`AEnemy`实例更新为使用Health值150。Enemy_3的Health将保持在500，因为它不使用旧的默认值。

### 编辑器集成

编辑器理解`UObject`和`UProperties`，编辑器可以自动公开这些值以供编辑，而不必编写特殊代码。这可以选择在蓝图视觉脚本系统中融入集成。有许多选项可以控制变量和函数的可访问性和公开。

### 运行时类型信息和类型转换

由于`UObject`是虚幻引擎反射系统的一部分，它们始终知道它们是哪些`UClass`，并可以在运行时做出有关类型的决定和类型转换。

在原生代码中，**每个`UObject`类都将自定义`Super`类型定义设置为其父类**，从而可以轻松控制重载行为。示例：

```c++
class AEnemy : public ACharacter
{
    virtual void Speak()
    {
        Say("Time to fight!");
    }
};

class AMegaBoss : public AEnemy
{
    virtual void Speak()
    {
        Say("Powering up!");
        Super::Speak();
    }
};
```

如你所见，调用`Speak`将会让MegaBoss说"Powering up!Time to fight!"。

此外，**你可以使用模板化`Cast`函数或者查询（如果Object是使用`IsA`的特定类）安全地将Object从基类转换为更衍生性类**。另一个简短示例：

```c++
class ALegendaryWeapon : public AWeapon
{
    void SlayMegaBoss()
    {
        TArray<AEnemy> EnemyList = GetEnemyListFromSomewhere();

        // The legendary weapon is only effective against the MegaBoss
        for (AEnemy Enemy :EnemyList)
        {
            AMegaBoss* MegaBoss = Cast<AMegaBoss>(Enemy);
            if (MegaBoss)
            {
                Incinerate(MegaBoss);
            }
        }
    }
};
```

这里我们使用了 `Cast` 来尝试将 `AEnemy` 转换为 `AMegaBoss`。如果所提及 Object 实际上不是 `AMegaBoss`（或者其子类），则Cast会返回空指针，我们可以适当的做出反应。在以上代码中，`Incinerate `将仅对MegaBoss调用。

### 垃圾回收

**虚幻实现垃圾回收机制，不再被引用或已被显式标记为销毁的 `UObject` 将定期清除。** 引擎构建一个引用图表以确定哪些 `UObject` 仍在使用，哪些是孤立的。在该图表根部是一组指定为"根集"的 `UObject`。任何 `UObject` 都可以添加到根集。当进行垃圾回收时，引擎将从根集开始，搜索已知 `UObject` 引用树来跟踪所有引用的 `UObject`。**任何未被引用的 `UObject`（意味着未在树搜索中找到这些对象）将被假设为不再需要，因此被删除。**

**一个实际的影响是，你通常需要保持对希望保持活跃的任何 Object 的 `UPROPERTY` 引用，或者将指向它的指针存储在 `TArray` 或其他引擎容器类中。** 

Actor 及其组件通常属于例外情况，因为 Actor 通常被链接回到根集的 Object 引用（例如它们所属的关卡），而 Actor 的组件被 Actor 自身引用。Actor 可以显式标记为销毁，方法是调用它们的 `Destroy` 函数，这是从进行中游戏移除 Actor 的标准方法。组件可以使用 `DestroyComponent` 函数显式销毁，但它们通常在拥有它们的 Actor 从游戏中移除时被销毁。

虚幻引擎4中的垃圾回收速度快，效率高，内置大量的优化功能，能够尽量降低开销，如多线程可访问性分析可以标识孤立Object，优化的反加密代码能够尽快从容器中移除Actor。还有一些其他功能以调节，以更精准地控制如何以及何时执行垃圾回收，大部分都可以在 **项目设置（Project Settings）** 中的 **引擎 - 垃圾回收（Engine - Garbage Collection）** 下找到。以下设置通常用于为项目调节垃圾回收器性能：

|设置|功能描述|
|---|---|
|**创建垃圾回收器UObject集群（Create Garbage Collector UObject Clusters）**|可以在项目设置中打开或关闭（默认打开）。如果打开，相关Object将被分组到一起归入垃圾回收集群，这样只需要检查集群自身即可，而不必检查每个Object。这意味着可以更快速地执行可访问性，因为整个集群将被视为一个对象，但也意味着该集群中的单个项目将被反加密，并准备在同一帧中删除，如果集群足够大，这样可能会导致卡顿。一般而言，集群创建会提高垃圾回收性能，缩短可访问性分析耗费的时间。|
|**合并GC集群（Merge GC Clusters）**|可以启用集群合并，这样当一个集群的对象引用另一个集群的对象时，让集群合并起来。请注意，清空导致合并的引用不会让新合并的集群瓦解或拆散。**创建垃圾回收器UObject集群（Create Garbage Collector UObject Clusters）** 也必须打开，该功能才能工作。这会使垃圾回收器反加密和销毁对象的频率降低，但一次反加密和销毁的对象数量会增加。此外，有些情况下不会对合并集群进行垃圾回收，因为对该集群中任何对象的任何引用都会阻止对整个集群进行垃圾回收。|
|**启用Actor集群（Actor Clustering Enabled）**|通过在 **项目设置（Project Settings）** 中打开这个选项，并将`bCanBeInCluster`变量设置为`true``，或者重载代码中的`CanBeInCluster`函数以使其返回`true``，可以将Actor放入集群中。默认情况下，Actor和组件会将这个选项关闭，但静态网格体Actor和反射捕获组件除外。该功能可用于将应该一次性全部销毁的Actor分组在一起，通常是关卡中放置的不能被销毁的静态网格体，除非卸载包含这些网格体的子关卡。|
|**启用蓝图集群（Blueprint Clustering Enabled）**|蓝图的`UBlueprintGeneratedClass`和相关数据，如共享UPROPERTY和UFUNCTION数据，可以通过打开该设置来建立集群。必须要认识到的是，该集群引用蓝图生成的类自身，而不是蓝图的单个实例。|
|**待清除终止对象之间的间隔（Time Between Purging Pending Kill Objects）**|垃圾回收活动的频率可以在项目设置中调整。该高级控制对于防止卡顿尤其有用。通过缩短回收间隔，可以减少将在下一次可访问性分析阶段发现的无法访问的对象的可能数量，并避免同时清除大量Actor时可能会发生的卡顿。 |

![[43215e1447e6ee7832a054d83789ba81_MD5.jpg]]

项目设置中的垃圾回收设置

### 网络复制

`UObject`系统包含一组可靠的功能，能够促进[网络通信和多人游戏](https://docs.unrealengine.com/5.2/zh-CN/networking-and-multiplayer-in-unreal-engine)。

`UProperties`可以标记为告诉引擎[在网络游戏期间复制数据](making-interactive-experiences/network-multiplayer/Actors/Properties/)。常见模型是一个变量在服务器上发生更改，引擎检测到这个更改，并将其可靠地发送到所有客户端。当变量通过复制发生更改时，客户端可以选择性接收回调函数。

`UFunctions`也可以标记为[在远程机器上执行](https://docs.unrealengine.com/5.2/zh-CN/rpcs-in-unreal-engine)。例如，"server"函数在客户端上调用时，将会在服务器上执行这个函数以获取服务器版本的Actor。而另一方面，"client"函数可以从服务器调用，并在拥有这个函数的客户端版本的对应Actor上运行。
# 接口
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

在你的.h文件（例如 `ReactToTriggerInterface.h`）中：


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

如果你想要让蓝图实现此接口，则需要 `Blueprintable` 说明符。

### 接口说明符

接口说明符用于向虚幻反射系统公开你的类，详见下表：

|接口说明符|含义|
|---|---|
|`BlueprintType`|将该类公开为可用于蓝图中的变量的类型。|
|`DependsOn=(ClassName1, ClassName2, ...)`|所有列出的类都将在该类之前编译。ClassName必须在同一个（或上一个）包中指定一个类。多个依赖性类可以使用以逗号分隔的单个"DependsOn"行来指定，也可以使用单个"DependsOn"行为每个类指定。当一个类使用在另一个类中声明的结构体或枚举时，这一点非常重要，因为编译器只知道它已经编译的类中的内容。|
|`MinimalAPI`|仅导致该类的类型信息被导出以供其他模块使用。你可以向该类转换，但不能调用该类的函数（内联方法除外）。对于不需要其所有函数在其他模块中均可供访问的类，通过不导出这些类的所有内容，这可以缩短编译时间。|

## 2 在 C++中实现接口

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

## 3 声明接口函数

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

#### BlueprintCallable

**引用实现接口的对象的C++或蓝图**可以调用使用 `BlueprintCallable` 说明符的函数。

#### BlueprintImplementableEvent

使用 `BlueprintImplementableEvent` 的函数**不能在C++中被重载，但可以在任何实现或继承接口的蓝图类中被重载。**

#### BlueprintNativeEvent

在C++中，可通过重载一个同名函数来实现使用 `BlueprintNativeEvent` 的函数，但要在末尾添加上后缀 `_Implementation` 。

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

该说明符还**允许在蓝图中重载实现**。

## 4 确定类是否实现了接口

为了与实现接口的C++和蓝图类兼容，可以使用以下任意函数：

```c++
bool bIsImplemented = OriginalObject->GetClass()->ImplementsInterface(UReactToTriggerInterface::StaticClass()); // 如果OriginalObject实现了UReactToTriggerInterface，则bisimplemated将为true。

bIsImplemented = OriginalObject->Implements<UReactToTriggerInterface>(); // 如果OriginalObject实现了UReactToTrigger，bIsImplemented将为true。

IReactToTriggerInterface* ReactingObjectA = Cast<IReactToTriggerInterface>(OriginalObject); // 如果OriginalObject实现了UReactToTriggerInterface，则ReactingObject将为非空。
```

如果 `StaticClass` 函数在前缀为 `I`的类中没有实现，尝试在前缀为 `U` 的类上使用 `Cast` 将失败，代码将无法编译。

## 5 转换到其他虚幻类型

虚幻引擎的转换系统支持**从一个接口转换到另一个接口**，或者在适当的情况下，**从一个接口转换到一个虚幻类型**。

```c++
IReactToTriggerInterface* ReactingObject = Cast<IReactToTriggerInterface>(OriginalObject); // 如果接口被实现，则ReactingObject将为非空。

ISomeOtherInterface* DifferentInterface = Cast<ISomeOtherInterface>(ReactingObject); // 如果ReactingObject为非空而且还实现了ISomeOtherInterface，则DifferentInterface将为非空。

AActor* Actor = Cast<AActor>(ReactingObject); // 如果ReactingObject为非空且OriginalObject为AActor或AActor派生的类，则Actor将为非空。
```

## 6 蓝图可实现类

如果你想要蓝图能够实现此接口，则必须使用 `Blueprintable` 元数据说明符。、

蓝图类要重载的每个接口函数都必须是 `BlueprintNativeEvent ` 或 `BlueprintImplementableEvent`。

标记为 ` BlueprintImplementableEvent` 的函数仍然可以被调用，但不能被重载。你将无法从蓝图访问所有其他函数。

