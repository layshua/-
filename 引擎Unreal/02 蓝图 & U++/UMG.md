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

# 富文本块
Rich Text Block 用法：
- 创建数据表格，选择 `RichTextStyleRow`![[Pasted image 20230825205729.png|450]]![[Pasted image 20230825205743.png|395]]
- 在数据表格中点击 Add 添加配置表，必须要有一行名为 Default 的数据，配置里记得设置一个字体 ![[Pasted image 20230825210534.png|500]]![[Pasted image 20230825210843.png]]
- 富文本快中选中对应的数据表格，在文本中使用数据名字标签显示富文本（需要编译）
    - Text 默认使用 Default 的配置，要使用第二行的配置只需要输入 `<Blue>text</>` 即可使用
![[Pasted image 20230825211035.png|500]]![[Pasted image 20230825211220.png]]

# 自定义控件
我们自己创建控件也可以作为其他 UI 控件的子控件

在 User Create 中可以直接使用我们创建的其他控件蓝图
![[Pasted image 20230825211737.png]]

当用户创建的 UI 成为其他 UI 的子控件时，默认情况下是不能拥有子控件的
![[Pasted image 20230825211850.png]]
## Named Slot
要想拥有子控件，就必须给该 UI 添加一个 Named Slot（命名插槽），这样这个 UI 就可以拥有子控件了
![[Pasted image 20230825212421.png]]
步骤：
- 创建带有命名插槽的 UI 控件
- 在其他 UI 面板中导入 UI 控件，在插槽处添加对应控件
心
