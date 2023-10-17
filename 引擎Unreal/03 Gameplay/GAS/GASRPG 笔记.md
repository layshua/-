
# 混合空间八方向
[【UE5】角色动画蓝图，制作八方向混合空间_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1TG4y1V75k/?spm_id_from=333.337.top_right_bar_window_custom_collection.content.click)

混合空间设置如图：
![[Pasted image 20231006093323.png|400]]
![[Pasted image 20231006093239.png]]

动画蓝图中计算 Direction 和 Velocity
![[Pasted image 20231006093631.png]]
# 角度问题
## 坐标系旋转角正负

在左手定则中，**从轴的正方向看向原点**，顺时针旋转方向为正方向；在右手定则中，从轴的正方向看向原点，逆时针旋转方向为正方向。

在UE4中，**坐标系遵循左手定则**，其**Z轴的旋转方向符号是遵循左手定则**的旋转方向，**但X和Y轴却遵循右手定则**：

（1）Z轴，旋转方向遵循左手定则，顺时针为正。
（2）X、Y轴，旋转方向遵循右手定则，逆时针为正。

## 鼠标XY轴

UE4中**鼠标的 XY 轴**遵循左手定则，从+Z 看向原点时，+X 指向右侧，+Y 指向下方。
![[Pasted image 20231006090308.png]]
又因为 UE4的坐标系遵循左手定则，但 XY 轴的旋转方向却遵循右手定则（即逆时针为正），故鼠标往+Y 方向移动时，Pitch（俯仰角）为正值（绕 Y 轴逆时针旋转）视角向上抬。
>注意鼠标 XY 值左上角为原点

需要特别注意的是：因为在BaseGame.ini中，InputPitchScale=-2.5，且PlayerController自带的AddPitchInput接口会将鼠标Pitch输入系数乘以该配置，所以一般**MouseY的轴映射为-1.0**，这样鼠标往上（-Y，Pitch<0）移动时，视角上抬，鼠标往下（+Y，Pitch>0）移动时，视角下移。
# 人物转向问题

[虚幻4人物转向问题{User Controller Rotaion Yaw，User Controller Desired Rotation与Orient Rotaion to Movement}-CSDN博客](https://blog.csdn.net/u012249992/article/details/83186907)
![[Pasted image 20231006091114.png]]
![[Pasted image 20231006091124.png]]

通常我们新建第三人称项目时，默认如上图的设置，这个时候按 s 键人正面是面向镜头的，这在一般的 rpg 或 act 游戏都是如此设置，而在绝地求生这类 STG 里按 s 键是背对镜头的，如何设置人物转向就在上面三个 bool 值决定。

**人物后移不转向（即人物始终跟随镜头转向）：**
`Use Controller Rotaion Yaw` 与 `User Controlle Desired Rotation` 都将人物与镜头视角绑定，即让人物始终跟随镜头转向。
区别：
- `Use Controller Rotaion Yaw` ：强制同步 Controller 偏航角 Yaw，人物始终跟随镜头转向，向后移动时始终面向前向
- `User Controlle Desired Rotation` ：平滑的同步 Controller 偏航角 Yaw，也可以设置转向的速度。人物人物始终跟随镜头转向，向后移动时始终面向前向（Desired：期望）

**人物后移转向：**
- `Orient Rotaion to Movement` ：重载 `User Controlle Desired Rotation`，使人物朝向加速方向，人物向后移动时面向后方。

**经典MMO模式：**
![[Pasted image 20231015205432.png]]
![[Pasted image 20231015205440.png]]
![[Pasted image 20231015205929.png]]
>SpringArm 和Camera上的设置：默认不用动


**动作模式**
仅需要修改这里，然后 InputAction 要弄一个根据鼠标位置移动旋转视角的回调，然后隐藏鼠标即可。
![[Pasted image 20231015210910.png]]

**锁敌思路**：
仅需要修改这里，然后弄一个根据敌人位置旋转视角的回调。
![[Pasted image 20231015210910.png]]

# 描边
后处理体积设为全局：
![[Pasted image 20231006170107.png]]

描边材质需要使用 Custom Stencil
![[Pasted image 20231006170208.png]]

开启深度/模板缓冲：
![[Pasted image 20231006170244.png]]

后处理体积设置后处理材质：
![[Pasted image 20231006170817.png]]

开启 Actor 模型的 CustomDepth Pass
![[Pasted image 20231006171255.png]]

通过代码设置 CustomDepth Pass
```c++
void AMageEnemy::HighlightActor()
{
	GetMesh()->SetRenderCustomDepth(true);
	GetMesh()->SetCustomDepthStencilValue(CUSTOM_DEPTH_STENCIL_VALUE);
	WeaponMesh->SetRenderCustomDepth(true);
	WeaponMesh->SetCustomDepthStencilValue(CUSTOM_DEPTH_STENCIL_VALUE);
}

void AMageEnemy::UnHighlightActor()
{
	GetMesh()->SetRenderCustomDepth(false);
	WeaponMesh->SetRenderCustomDepth(false);
}
```

# GAS 框架
![[Pasted image 20231006200535.png]]

![[Pasted image 20231006200833.png]]

![[Pasted image 20231006202101.png]]

# debug
调式模式启动，控制台输入：showdebug abilitysystem
![[Pasted image 20231007164907.png]]

# UI 架构
## 血条
MVC 架构
![[Pasted image 20231007205904.png]]

![[Pasted image 20231007210219.png]]

WidgetController：从 Model 收集数据并广播到 UserWidget

## 属性菜单
同MVC
![[Pasted image 20231013214339.png]]

![[Pasted image 20231013215510.png]]

# 属性设计参考
[各职业属性加点综合讲解（新人篇） - 灵游记秘诀区 - 《灵游记》官方论坛 灵游记|论坛 (ttgames.net)](http://ghostbbs.ttgames.net/showtopic-15269.aspx)
![[Pasted image 20231011233729.png]]

# UI 参考
天刀
![[Pasted image 20231012234505.png]]

诛仙世界：
![[Pasted image 20231012234657.png]]

![[Pasted image 20231013173418.png]]

# 设置 AssetManager
DefaultEngine.ini

```c++
[/Script/Engine.Engine]
//...

//添加 = /Script/项目名/资产管理类名
AssetManagerClassName = /Script/ProjectGASRPG.MageAssetManager
```

# 委托绑定函数指针

直接利用 TMap 绑定委托：
```c++
/** 用于AttributeMenuWidgetController广播初始值 */
TMap<FGameplayTag, FAttributeSignature> TagsToAttributes;


FAttributeSignature HealthDelegate;
HealthDelegate.BindStatic(GetHealthAttribute); //绑定委托，每次调用委托都会返回对应的FGameplayAttribute
TagsToAttributes.Add(FMageGameplayTags::Get().Attribute_Vital_Health, HealthDelegate);
```

下面三行要对每一个属性进行绑定，重复很多行，很不优雅。

用 TBaseStaticDelegateInstance FunctionPtr 代替 FAttributeSignature 委托
```c++
//TBaseStaticDelegateInstance委托可以绑定一个C++函数指针
TMap<FGameplayTag, TBaseStaticDelegateInstance<FGameplayAttribute(),FDefaultDelegateUserPolicy>::FFuncPtr> TagsToAttributes;

//只需要一行即可
TagsToAttributes.Add(FMageGameplayTags::Get().Attribute_Vital_Health, GetHealthAttribute);
```

**核心在于**：TBaseStaticDelegateInstance 委托可以绑定一个 C++函数指针。
我们需要 TMap 的值返回 `FGameplayAttribute` 类型，使用 TBaseStaticDelegateInstance 直接绑定 GetHealthAttribute 函数比使用委托获取返回值更优雅。
```c++
//这么一大串只需要理解为声明了一个TestFuncPtr的无参函数指针，返回类型为FGameplayAttribute
TBaseStaticDelegateInstance<FGameplayAttribute(),FDefaultDelegateUserPolicy>::FFuncPtr TestFuncPtr;
//这是个模板，指定类型后面还可以添加参数！
```

如果使用原生 C++的函数指针声明方式也是可以的
```c++
typedef FGameplayAttribute (*FuncPtr)();

TMap<FGameplayTag, FuncPtr> TagsToAttributes;
```

当我们用 auto 循环这个 TMap 时，提示：
![[Pasted image 20231014212321.png]]

直接用提示里的函数指针替换那一大串就 OK 了！更优雅了
```c++
TMap<FGameplayTag, FGameplayAttribute (*)()> TagsToAttributes;
```

终极必杀：模板
```c++
//声明一个可以返回任何类型的无参函数指针
template<typename T>
using TFuncPtr = T (*)();

//更优雅的实现！
TMap<FGameplayTag, TFuncPtr<FGameplayAttribute>> TagsToAttributes; 
```

当然我们也可以回去使用 UE 的模板 该模板支持可变参数
```c++
template<typename T>
using TStaticFuncPtr = typename TBaseStaticDelegateInstance<T,FDefaultDelegateUserPolicy>::FFuncPtr;

```

# 点击移动

生成一条路径来绕过障碍物
![[Pasted image 20231015203438.png]]
解决方案：使用**样条线**生成平滑曲线
![[Pasted image 20231015203516.png|300]]

# 自定义蓝图 latent 节点
![[Pasted image 20231017102723.png]]

```c++
/** 节点的多个输出引脚都是由委托实现的 */
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMouseTargetDataSignature, const FVector&, Data);

UCLASS()
class PROJECTGASRPG_API UTargetDataUnderMouse : public UAbilityTask
{
	GENERATED_BODY()

public:
//自定义蓝图 latent 函数节点
	UFUNCTION(BlueprintCallable, Category = "Ability|Tasks",meta = (DisplayName = "TargetDataUnderMouse", HidePin = "OwningAbility", DefaultToSelf = "OwningAbility", BlueprintInternalUseOnly = "TRUE"))
	static UTargetDataUnderMouse* CreateTargetDataUnderMouse(UGameplayAbility* OwningAbility);

	UPROPERTY(BlueprintAssignable)
	FMouseTargetDataSignature ValidData;
};
```

只需要在需要的地方广播委托，拉线相当于绑定委托。
