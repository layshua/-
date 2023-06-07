# 1
## 创建各种蓝图
![[Pasted image 20221223111338.png]]
WorldSetting中设置
![[Pasted image 20221223111420.png]]

## 攻击动画和连招
右键动画文件，骨骼重定向
![[Pasted image 20221223111642.png]]
![[Pasted image 20221223111713.png]]
右键创建蒙太奇
![[Pasted image 20221223112125.png]]
![[Pasted image 20221223112143.png]]

使用蒙太奇需要在动画蓝图中国添加Slot
![[Pasted image 20221223112823.png]]
![[Pasted image 20221223112705.png]]
要和蒙太奇中的设置一致
![[Pasted image 20221223112758.png]]
## 武器资源绑定
找到右手的骨骼，右键增加Socket插褿
![[Pasted image 20221223111833.png]]
右键新建的插槽，找到武器模型文件
![[Pasted image 20221223111941.png]]
调整好位置和大小

注意：在骨骼中的操作只是预览效果，还需要在人物蓝图中添加武器组件
![[Pasted image 20221223113026.png]]
在“雾切”细节面板如下设置，位置就和骨骼中的设置一样了。![[Pasted image 20221223113110.png]]
## 武器特效
在右手骨骼中增加两个Socket：![[Pasted image 20221223114318.png]]
分别放在武器顶部和中部
![[Pasted image 20221223114344.png|200]]![[Pasted image 20221223114351.png|200]]
在Montage面板Notifies处右键增加Trail，可以增加多个notifes叠加效果
![[Pasted image 20221223114125.png]]
![[Pasted image 20221223114522.png]]
导入特效文件进行设置：
![[Pasted image 20221223114606.png]]

**Root Motion**
我们发现，每次释放Montage之后人物会瞬间移动到原点，这需要我们在Montage对应的**动画原文件**的细节面板勾选开启根骨骼运动（Root Motion）。
![[Pasted image 20221223114957.png]]


