**虚幻示意图形界面设计器** (Unreal Motion Graphics Ul Designer)简称 UMG，用于实现游戏中的 HUD 元素和前端菜单等UI 功能

# UI 控件显示到蓝图
在关卡蓝图中
-  `CreateWidget` 
-  `Add to Viewport`
-  显示鼠标指针方便 UI 交互：通过PlayController `Set Show Mouse Cursor`
![[Pasted image 20230825000028.png]]

# 动态设置控件值
以进度条控件为例：
![[Pasted image 20230825001608.png]]
![[Pasted image 20230825001600.png]]

方法一：在蓝图中直接引用变量进行修改：
![[Pasted image 20230825001837.png|500]]

方法二：创建 Bind 函数，外部可以通过修改新建的 Percent 变量来控制百分比
![[Pasted image 20230825002126.png|500]]