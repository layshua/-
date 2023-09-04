# 关卡实例
源码： [DruidMech/UE5_TheUltimateDeveloperCourse: Unreal Engine 5 C++ The Ultimate Developer Course (github.com)](https://github.com/DruidMech/UE5_TheUltimateDeveloperCourse)

导入其他关卡

- 删除下载的 map 资源里的所有 camera
- 原关卡已有的东西全都删了
![[Pasted image 20230903165715.png|430]]
![[Pasted image 20230903165637.png|400]]

新建关卡实例
![[Pasted image 20230903165925.png]]
# 动画 AnimInstance 类
因为动画事件即便不 play，也会一直运行，因此写代码时要十分注意内存泄露。建议在编译对该类的任何更改时关闭 UE 编辑器，防止编辑器崩溃

# IK + ControlRig
82 级别： https://www.bilibili.com/video/BV1EM411U7PX/?p=82&spm_id_from=pageDriver&vd_source=02e3d219e0c32801f6b50c2266e6a7be
IK：
两脚分别放一个球，用来做球体追踪
![[Pasted image 20230904220353.png|450]]
1. 球体追踪，分别计算两只脚和脚下表面之间的偏移量。
2. 找出最低的（偏移量大的），用它可以确定将骨盆骨骼向下移动的偏移量
3. 插值移动骨骼，过程平滑一些

末端骨骼移动上去之后，如何确定其他骨骼的位置？UE 内置的 IK 帮我们解这个方程。
![[Pasted image 20230904220708.png|500]]



