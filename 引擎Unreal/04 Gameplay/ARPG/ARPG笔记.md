# 关卡实例
源码： [DruidMech/UE5_TheUltimateDeveloperCourse: Unreal Engine 5 C++ The Ultimate Developer Course (github.com)](https://github.com/DruidMech/UE5_TheUltimateDeveloperCourse)

导入其他关卡

- 删除下载的 map 资源里的所有 camera
- 原关卡已有的东西全都删了
![[Pasted image 20230903165715.png|430]]
![[Pasted image 20230903165637.png|400]]

新建关卡实例
![[Pasted image 20230903165925.png]]

# 碰撞重叠
## 静态网格体添加碰撞体
![[Pasted image 20230904233655.png|650]]

## 相机碰撞
默认为阻挡，相机被挡住会前移靠近人物
![[Pasted image 20230904234627.png|550]]

选择忽略，相机不会移动，这样就可以实现被物体挡住
![[Pasted image 20230904234746.png]]

## 为单个组件添加事件
选中组件搜索
![[Pasted image 20230905000347.png|500]]

# 拾取武器
![[Pasted image 20230905151915.png]]

# FORCEINLINE
内联函数直接展开，不需要想普通函数那样根据函数名取内存中找对应的实现。
非常适合用于 Get Set 函数
```
FORCEINLINE void SetOverlappingWeapon(AWeaponBase* Weapon){WeaponBase = Weapon;};
```


# 动画
## 动画 AnimInstance 类
因为动画事件即便不 play，也会一直运行，因此写代码时要十分注意内存泄露。建议在编译对该类的任何更改时关闭 UE 编辑器，防止编辑器崩溃

## IK + ControlRig
82 级别： https://www.bilibili.com/video/BV1EM411U7PX/?p=82&spm_id_from=pageDriver&vd_source=02e3d219e0c32801f6b50c2266e6a7be
IK：
两脚分别放一个球，用来做球体追踪
![[Pasted image 20230904220353.png|450]]
1. 球体追踪，分别计算两只脚和脚下表面之间的偏移量。
2. 找出最低的（偏移量大的），用它可以确定将骨盆骨骼向下移动的偏移量
3. 插值移动骨骼，过程平滑一些

末端骨骼移动上去之后，如何确定其他骨骼的位置？UE 内置的 IK 帮我们解这个方程。
![[Pasted image 20230904220708.png|500]]



## 从 Mixamo 导入动画
P92： https://www.bilibili.com/video/BV1EM411U7PX/?p=92&spm_id_from=pageDriver&vd_source=02e3d219e0c32801f6b50c2266e6a7be

mixamo converto 插件给 mixamo 导出的模型添加根骨骼：P124 https://www.bilibili.com/video/BV1EM411U7PX/?p=124&spm_id_from=pageDriver&vd_source=02e3d219e0c32801f6b50c2266e6a7be
## 重定向 IK RIG
P93+P94 创建 IK Rig（IK Rig & Retargeting 动画复用）


## 蒙太奇
![[Pasted image 20230905203832.png]]

分段标记：
![[Pasted image 20230906230739.png]]
分段执行，一定要断开片段
![[Pasted image 20230906230719.png]]

```c++
void AMyCharacter::PlayAttackMontage()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
	if (AnimInstance && AttackMontage)
	{
		AnimInstance->Montage_Play(AttackMontage);
		FName NotifyName = FName();
		switch (AttackComboIndex)
		{
		case 0:
			NotifyName = FName("AttackCombo1");
			++AttackComboIndex;
			break;
		case 1:
			NotifyName = FName("AttackCombo2");
			++AttackComboIndex;
			break;
		case 2:
			NotifyName = FName("AttackCombo3");
			++AttackComboIndex;
			break;
		case 3:
			NotifyName = FName("AttackCombo4");
			AttackComboIndex = 0;
			break;
		default:
			break;
		}
		AnimInstance->Montage_JumpToSection(NotifyName, AttackMontage);
	}
}
```

## 引擎中编辑动画
114
# 声音
## 旧方法
UE4 旧方法：直接在动画资源或动画蒙太奇中添加动画通知
![[Pasted image 20230906135518.png]]

通过创建 cue，可以修改声音文件

## meta Sounds
程序化音效

新建一个变量
![[Pasted image 20230906142545.png]]

随机音高
![[Pasted image 20230906142524.png]]