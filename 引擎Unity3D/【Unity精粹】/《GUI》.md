
---
title: 《GUI》
aliases: []
tags: []
create_time: 2023-06-07 23:35
uid: 202306072335
banner: "![[diablo-iv-beta-vendor-3.png]]"
---

# 九宫格 UI 理论
**相对屏幕位置 ScreenPos：**
![[Pasted image 20230607232730.png]]
widgetPos.x = ScreenPos + widgetCenterPos + offsetPos;

**控件中心点位置 widgetCenterPos：**
cw 是控件自身的宽度
ch 是控件自身的高度
![[Pasted image 20230608080527.png|850]]

最后通过操作偏移位置 offset 来摆放UI
# GUI
## 1 原理及作用
**IMGUI**  （Immediate Mode Graphical User Interface）即时模式图形化交互界面
IMGUI 在 Unity 中一般简称为 GUI，它是一个代码驱动的 UI 系统

作用：
1. 作为程序员的调试工具，创建游戏内调试工具
2. 为脚本组件创建自定义检视面板
3. 创建新的编辑器窗口和工具以拓展 Unity 本身（一般用作内置游戏工具）
4. 用于进行 Unity 内置编辑器，调试工具编辑工具等等相关开发，**不适合用它为玩家制作 UI 功能**

**GUI 的工作原理**：在继承 MonoBehaviour 的脚本中的特殊函数里，调用 GUI 提供的方法，类似生命周期函数。
```cs
private void OnGUI()
{
    
}
```
1. 它**每帧执行**相当于是用于专门绘制 GUI 界面的函数 
2. 一般只在其中执行 GUI 相关界面绘制和操作逻辑 
3. 该函数**在 OnDisable 之前 LateUpdate 之后执行** 
4. 只要是继承 Mono 的脚本都可以在 OnGUI 中绘制 GUI

**缺点**
- 重复工作量繁多
- 控件（widget）绘制相关代码很多
- 最大缺点:必须运行时才能去查看结果（不能所见即所得，通过 `[ExecuteAlways]` 特性解决），不支持分辨率自适应


## 2 重要参数

> [!warning] 
> GUI 的屏幕原点是左上角

**GUI 控件绘制的共同点：**
1. 他们都是 GUI 公共类中提供的静态函数直接调用即可 
2. 他们的参数都大同小异
    - 位置参数: Rect  （xy 位置 wh 尺寸）
    - 显示文本: string 
    - 图片信息: Texture 
    - 参数综合信息: GUIContent 
    - 参数自定义样式: GUIStyle 
3. **每一种控件都有多种重载，都是各个参数的排列组合必备的参数内容，是位置信息和显示信息**


## 3 Label 标签
![[Pasted image 20230607151421.png|700]]
![[Pasted image 20230607151337.png|450]]

```cs
public Texture texture;  

//骷髅
public Rect rect1;  

//小黄脸
public Rect rect2;  
public GUIContent content2;

public GUIStyle style;

private void OnGUI()  
{
    //1.文本
    GUI.Label(new Rect(0,0,100,20),"Hello World"); //text，传位置信息也可以直接声明公共变量Rect，如下：
    
    //2.图片
    GUI.Label(rect1, texture);  
    
    //3.文本+图片
    //GUIContent可以控制text,image,tooltip
    GUI.Label(rect2,content2);
    Debug.Log(GUI.tooltip);  //获取当前鼠标或者键盘选中的GUI控件对应的tooltip信息


    //4.自定义格式，第三个参数传入GUIStyle
    GUI.Label(new Rect(0,0,100,20),"Hello World",style);
}
```


## 4 Button 按钮
自定义格式：
![[Pasted image 20230607151820.png|450]]


```cs
public Rect rect;  
public GUIContent content;  
public GUIStyle style;

void OnGUI()  
{
    //鼠标按下松开为一次点击
    //无style参数时，使用默认style
    if (GUI.Button(rect, content, style))  //判断是否点击
    {  
        //处理按钮点击逻辑
        print("Button Clicked");  
    }
    
    //鼠标长按
    if(GUI.RepeatButton(rect, content, style))  
    {  
        print("Button Clicked");  
    }
}
```

## 5 Toggle 开关
Toggle 意为（两种状态之间）切换

```cs
//设置方法类似上面的按钮，多一个点击选中的判断，我们要自己声明一个bool值
public bool isSelect;  
  
public Rect rect;  
public GUIContent content;  
public GUIStyle style;

void OnGUI()  
{
    isSelect = GUI.Toggle(rect, isSelect, "Toggle");
    isSelect = GUI.Toggle(rect, isSelect, content,style);
}
```

排版攻略：
1. 修改固定宽高 fixedwidth 和 fixedHeight
2. 修改从 GUIStyle 边缘到内容起始处的空间 padding


![[2022306071646.gif]]
```cs file:基于Toggle实现多选框
public bool isSelect;  
public Rect rect;  
private int nowSelIndex = 0;

void OnGUI()  
{
    //基于Toggle实现多选框
    if (GUI.Toggle(new Rect(0, 60, 100, 30), nowSelIndex == 1, "选项一"))
    {
        nowSelIndex = 1;
    }

    ;
    if (GUI.Toggle(new Rect(0, 90, 100, 30), nowSelIndex == 2, "选项二"))
    {
        nowSelIndex = 2;
    }

    if (GUI.Toggle(new Rect(0, 120, 100, 30), nowSelIndex == 3, "选项三"))
    {
        nowSelIndex = 3;
    }
}
    
```

## 6 输入框和拖动条
![[Pasted image 20230607210356.png]]

```cs
private string inputStr = "";
private string password = "";   
private float nowValue = 0.5f;
public GUIStyle thumbStyle; //滑块样式
//输入框
//比较特别是的第三个参数，最大输入长度
inputStr = GUI.TextField(new Rect(0,0,100,50), inputStr,5); 

//密码输入,输入全被*遮盖
password = GUI.PasswordField(new Rect(100,0,100,50), password, '*');

//水平拖动条
nowValue = GUI.HorizontalSlider(new Rect(200, 0, 100, 100), nowValue,0.0f,1.0f);
//带风格的拖动条，多一个style参数
nowvalue = GUI.HorizontalSlider(new Rect(200, 0, 100, 100),, nowvalue, minValue, maxValue,style,thumbStyle); //默认的style时滑动条的style，滑块style要自己声明

//竖直拖动条GUI.VerticalSlider()
```

## 7 图片绘制和Box框
![[Pasted image 20230607212445.png]]
```cs file:GUI.DrawTexture
public Rect texPos;
public Texture texture;

public ScaleMode scaleMode; //可切换三种缩放模式
//scaleAndCrop:通过宽高比来计算图片，但是会进行裁剪
//ScaleToFit:会自动根据宽高比进行计算，不会拉变形，会一直保持图片完全显示的状态
//stretchToFill:始终填充满你传入的 Rect范围

public bool alphaBlend ;    //默认为true使用alpha透明通道

private void OnGUI()
{
    GUI.DrawTexture(texPos, texture,scaleMode,alphaBlend);
}
```

简单的 Box 边框，没特殊功能
![[Pasted image 20230607212756.png]]
```cs file:GUI.Box
GUI.Box(new Rect(0,0,100,100),"123");
```

## 8 工具栏和选择网格
工具栏特点，多个按钮只能同时选择一个
![[Pasted image 20230607215332.png]]

```cs file:GUI.Toolbar
private int toolbarIndex = 0;
private string[] toolbarInfos = {"选项一", "选项二", "选项三"};
private void OnGUI()
{
    toolbarIndex = GUI.Toolbar(new Rect(0,0,300,30),toolbarIndex,toolbarInfos);
    switch (toolbarIndex)
    {
        case 0:
            break;
        case 1:
            break;
        case 2:
            break;
    }
}
```

选择网格和工具栏具有相同的特点，相对 toolbar 多了一个参数 xCount，**代表水平方向最多显示的按钮数量。**

当 xCount 为 3 时，和上面 toolbar 绘制的 ui 一样。
![[Pasted image 20230607215332.png]]
当 xCount 为 2 时：
![[Pasted image 20230607215701.png]]
当 xCount 为 1 时：
![[Pasted image 20230607215722.png]]
```cs file:GUI.SelectionGrid
private int selGridIndex = 0;
private string[] selGridInfos = {"选项一", "选项二", "选项三"};
private void OnGUI()
{
    selGridIndex = GUI.SelectionGrid(new Rect(0, 50, 300, 30), selGridIndex, selGridInfos,3);
}
```
## 9 滚动视图和分组
**分组**
- 用于批量控制控件位置
- 可以理解为包裹着的控件加了一个父对象
- 可以通过控制分组来控制包裹控件的位置
```cs file:GUI.BeginGroup
public Rect groupPos;  

private void OnGUI()  
{
    //批量控制控件位置
    GUI.BeginGroup(groupPos);
    GUI.Button(new Rect(0,0,100,50),"按钮1");
    GUI.Button(new Rect(0,50,100,50),"按钮2");
    GUI.EndGroup();
}
```

**滚动视图：**
![[Pasted image 20230607222051.png]]

```cs file:GUI.BeginScrollView
public Rect uiPos;
public Vector2 scrollPos;
public Rect viewPos;

private void OnGUI()
{
    GUI.BeginScrollView(uiPos, scrollPos, viewPos);
    GUI.Button(new Rect(0,0,100,50),"按钮1");
    GUI.Button(new Rect(0,50,100,50),"按钮2");
    GUI.EndScrollView();
}
```

## 10 窗口
就是单独的一个窗口，在绘制窗口的函数中写 UI 代码，以窗口的左上角为原点
![[202306072241.gif]]
```cs file:GUI.Window
public Rect windowPos;
   
private void OnGUI()
{
   //参数一：窗口唯一ID，
   //委托参数是用于绘制窗口用的函数，传入即可
   GUI.Window(1,new Rect(100,100,200,150),DrawWindow,"窗口1");
   GUI.Window(2,new Rect(400,100,200,150),DrawWindow,"窗口2");

   //可拖动窗口
   //1.位置赋值
   //2.绘制函数调用GUI.DragWindow();
   windowPos = GUI.Window(3,windowPos,DrawWindow,"拖动窗口");

}

private void DrawWindow(int id)
{
   switch (id)
   {
       case 1:
           GUI.Button(new Rect(0,0,50,50),"按钮");
           break;
       case 2:
           GUI.Box(new Rect(0,0,100,100),"123");
           break;
       case 3:
           //传入Rect参数的重载作用
           //决定窗口中哪一部分位置可以被拖动
           //默认不填,就是无参重载,默认窗口的所有位置都能被拖动
           GUI.DragWindow();  
           
           //传参限制可拖动位置，可以实现只能通过拖动顶栏移动窗口
           //GUI.DragWindow(new Rect(0,0,100,20));  
           break;
   }
}
```

**模态窗口**
- 可以让窗口外的其它控件无法点击
- 你可以理解该窗口在最上层，其它按钮都点击不到了，只能点击该窗口上控件
```cs file:GUI.ModalWindow
GUI.ModalWindow(2,new Rect(400,100,200,150),DrawWindow,"模态窗口");
```

## 11 颜色和皮肤
```cs file:设置颜色
//全局着色，同时影响背景和字体，不常用
GUI.color = Color.blue;
      
GUI.contentColor = Color.green;     //文本颜色
GUI.backgroundColor = Color.yellow; //背景颜色
GUI.Button(new Rect(100, 100, 100, 100), "按钮1");
      
GUI.contentColor = Color.white;  //设置白色就可以恢复原色
GUI.backgroundColor = Color.white;
GUI.Button(new Rect(300, 300, 100, 100), "按钮2");
```

右键创建 GUI Skin，相当于 GUI style 的集合体，支持修改所有控件样式。
可以在这里面修改，通过代码传给 UI 控件。
![[Pasted image 20230607225608.png|400]]

```cs file:设置皮肤
public GUISkin guiSkin;

private void OnGUI()
{
  GUI.skin = guiSkin;
  GUI.Button(new Rect(100,100,100,100),"按钮1");
  
  GUI.skin = null;  //恢复默认皮肤
  GUI.Button(new Rect(300,300,100,100),"按钮2");
}
```
## 12 布局
不需要传 Rect 位置参数，自动布局，主要用于编辑器开发（编辑器 UI 排列比较整齐简单）, 不适合作为游戏UI

![[Pasted image 20230607230430.png]]
```cs
GUILayout.BeginArea(new Rect(100,100,50,50));   //也可以使用group等统一管理位置
GUILayout.Button("123");
GUILayout.Button("123");
GUILayout.Button("123142");
GUILayout.Button("阿斯顿123142");
GUILayout.EndArea();
```

使用布局选项：


```cs file:布局选项
GUILayout.Button("123",GUILayout.Width(300)); //布局选项作为第二个参数传入


//布局选项：
//控件的固定宽高
GUILayout.Width(300);
GUILayout.Height(200);
//允许控件的最小宽高
GUILayout.MinWidth(50);
GUILayout.MinHeight(50);
//允许控件的最大宽高
GUILayout.MaxWidth(100);
GUILayout.MaxHeight(100);
//允许或禁止水平拓展
GUILayout.ExpandWidth(true);    //允许
GUILayout.ExpandHeight(false);  //禁止
```

## 13 自适应
![[Pasted image 20230608080626.png|350]]

# UGUI
UGUI 是 Unity 引擎内自带的 UI 系统官方称之为: Unity Ul
是目前 Unity 商业游戏开发中使用最广泛的 UI 系统开发解决方案
它是基于 Unity 游戏对象的 UI 系统，**只能用来做游戏 UI 功能，不能用于开发 Unity 编辑器中内置的用户界面**
![[Pasted image 20230616154516.png]]

## 1 六大基础组件

**Canvas 对象上依附的:**
`Rect Transform`：UI 对象位置锚点控制组件，主要用于控制位置和对其方式 
`Canvas`：画布组件，主要用于渲染 UI 控件 
`Canvas Scaler`：画布分辨率自适应组件，主要用于分辨率自适应  
`Graphic Raycaster`：射线事件交互组件，主要用于控制射线响应相关  


**EventSystem 对象上依附的:**
`Event System` ：玩家输入事件响应系统，主要用于监听玩家操作 
`Standalone Input Module` ：独立输入模块组件，主要用于监听玩家操作 

### Rect Transform
**UI 对象位置锚点控制组件，主要用于控制位置和对其方式** 

Rect Transform 意思是矩形变换
**是专门用于处理 UI 元素位置大小相关的组件**

- **RectTransform 继承于 Transform**，Transform 组件只处理位置、角度、缩放
- **RectTransform 在此基础上加入了矩形相关，将 UI 元素当做一个矩形来处理加入了中心点、锚点、长宽等属性**，其目的是更加方便的控制其大小以及分辨率自适应中的位置适应。

![[Pasted image 20230616211120.png]]

![[Pasted image 20230616210849.png]]


- @ Pivot：Pivot 轴心点默认为（0.5，0.5）
- 轴心点是旋转的中心（通过调节 Rotation. z 来旋转控件）
- 和锚点配合控制位置
![[Pasted image 20230616204627.png]]

- @ Anchors：Anchors 轴心点默认为（0.5，0.5）
![[Pasted image 20230616205128.png]]
### Canvas 
**画布组件，主要用于渲染 UI 控件** 
![[Pasted image 20230616160418.png]]
- 它是 UGUI 中所有 UI 元素能够被显示的根本
- 它主要负责渲染自己的所有 UI 子对象
- 如果 UI 控件对象不是 Canvas 的子对象，那么控件将不能被渲染
- 我们可以通过修改 Canvas 组件上的参数修改渲染方式
- 场景中可以有多个 Canvas 对象，可以分别管理不同画布的渲染方式，分辨率适应方式等等参数。如果没有特殊需求，—般情况场景上一个 Canvas 即可。

#### RenderMode 渲染模式
![[Pasted image 20230616160333.png]]
##### Screen Space - Overlay
覆盖模式，UI 始终显示在场景内容前方

![[Pasted image 20230616160500.png]]

##### Screen Space - Camera
摄像机模式，3D 物体可以显示在 UI 之前

![[Pasted image 20230616160834.png|700]]

1. 不建议使用 Main Camera，避免场景模型遮挡 UI。
2. **使用一个单独的 Camera（后文称之为 UI Camera） 负责渲染 UI。**
    - 主摄像机 Depth 保持默认的-1，UI Camera 的 Depth 要大于-1（深度较大的绘制在深度较小的上方）
    - 主摄像机 Culling Mask 取消勾选 UI
    - UI Camera 的 Culling Mask 只选择 UI，**Clear Flags**设置为 Depth Only（只画该层，背景透明，这样才不会让 UI 遮挡后面的内容）
3. 如果需要在 UI 上显示 3D 模型，直接在 Canvas 上创建即可，Layer 要设置成UI
4. 通过设置 Sorting Layer，也可以对 Canvas 进行排序，后面的层覆盖前面的层。
Order in Layer，适用于相同 Layer 中进行排序

##### Screen Space - Camera
3D 模式，可以把 UI 对象像 3D 物体一样处理，常用于 VR 或者 AR
![[Pasted image 20230616163024.png|350]]
![[Pasted image 20230616162932.png]]

**Event Camera**：用于处理 UI 事件的摄像机（ 如果不设置，不能正常注册 UI 事件）

### Canvas Scaler 
**画布缩放控制器，用于画布分辨率自适应的组件  

它主要负责在不同分辨率下 UI 控件大小自适应
**它并不负责位置，位置由之后的 Rect Transform 组件负责**

**提供了三种用于分辨率自适应的模式**（按需选择）
1. Constant Pixel Size（恒定像素模式)∶
无论屏幕大小如何，U 始终保持相同像素大小

2. Scale With Screen Size (随屏幕尺寸缩放模式)∶
根据屏幕尺寸进行缩放，随着屏幕尺寸放大缩小

3. Constant Physical Size（恒定物理模式)：
无论屏幕大小和分辨率如何，UI 元素始终保持相同物理大小

#### 分辨率
1. **屏幕分辨率**——当前设备的分辨率，编辑器下 Game 窗口中 Stats 可以查看到
![[Pasted image 20230616164340.png]]
2. **参考分辨率** Reference Resolution——在 Scale With Screen Size 缩放模式中出现的关键参数，参与分辨率自适应的计算
3. **画布宽高和缩放系数**——分辨率自适应会改变的参数，通过屏幕分辨率和参考分辨率计算而来。选中 Canvas 对象后在 Rect Transform 组件中看到的宽高和缩放系数
```
//分辨率为（x,y）,则：
Width * Scale. x = 分辨率x
Height * Scale. y = 分辨率y
```

4. **分辨率大小自适应**——通过一定的算法以屏幕分辨率和参考分辨率参与计算得出缩放系数该结果会影响所有 UI 控件的缩放大小

#### UI Scale Mode UI 缩放模式
重点：
![[Pasted image 20230616172357.png]]
![[Pasted image 20230616171748.png]]
##### Constant Pixel Size 恒定像素模式 
**无论屏幕大小如何，U 始终保持相同像素大小**
它不会让 UI 控件进行分辨率大小自适应
会让 UI 控件始终保持设置的尺寸大小显示
**一般在进行游戏开发<mark style="background: #FF5582A6;">极少使用这种模式</mark>，除非通过代码计算来设置缩放系数**

![[Pasted image 20230616163416.png]]
- **Scale Factor: 缩放系数**，按此系数缩放画布中的所有 UI 元素 
- **Reference Pixels Per Unit：单位参考像素**，多少像素对应 Unity 中的一个单位（**默认一个单位为 100 像素**)，图片设置中的 Pixels Per Unit 设置，会和该参数一起参与计算

Set Native Size：恢复 Source Image 的原始尺寸，结果需要经过计算：
![[Pasted image 20230616165546.png|500]]
![[Pasted image 20230616165421.png]]


##### Scale With Screen Size  随屏幕尺寸缩放模式
**根据屏幕尺寸进行缩放，随着屏幕尺寸放大缩小，<mark style="background: #FF5582A6;">最常用</mark>**

![[Pasted image 20230616164147.png]]

- **Reference Resolution ：参考分辨率** (PC 常用 1920x1080，手机端也要适配对应分辨率，一般由美术人员决定)。缩放模式下的所有匹配模式都会基于参考分辨率进行自适应计算
- **Screen Match Mode：屏幕匹配模式**，当前屏幕分辨率宽高比不适应参考分辨率时，用于分辨率大小自适应的匹配模式。
    - 有三种模式：**最常使用的是 Match Width Or Height 模式，套路如下：**
![[Pasted image 20230616171748.png]]

- <mark style="background: #FF5582A6;">三种模式的详细解释</mark>
    1. <mark style="background: #D2B3FFA6;">Expand</mark>: 水平或垂直**拓展画布**区域，会根据宽高比的变化来放大缩小画布，可能有黑边： ![[Pasted image 20230616171229.png|300]] ![[Pasted image 20230616170712.png|500]] ![[Pasted image 20230616170854.png|500]]
    2. <mark style="background: #D2B3FFA6;">Shrink</mark>: 水平或垂直**裁剪画布**区域，会根据宽高比的变化来放大缩小画布，可能会裁剪  ![[Pasted image 20230616171328.png|500]]
     3. <mark style="background: #D2B3FFA6;">Match Width Or Height</mark>: **以宽高或者二者的平均值**作为参考来缩放画布区域（常用）![[Pasted image 20230616171520.png|450]] ![[Pasted image 20230616171534.png|500]] ![[Pasted image 20230616171554.png]]

##### Constant Physical Size 恒定物理模式 
无论屏幕大小和分辨率如何，UI 元素始终保持相同物理大小
 
![[Pasted image 20230616164157.png]]

**DPI: （Dots Per Inch，每英寸点数）图像每英寸长度内的像素点数**
Physical Unit：物理单位，使用的物理单位种类
Falback Screen DPI：备用 DPI，当找不到设备 DPI 时，使用此值 Default Sprite DPI: 默认图片 DPI

![[Pasted image 20230616172726.png]]
![[Pasted image 20230616172735.png]] ![[Pasted image 20230616172936.png]]

##### World 世界模式
![[Pasted image 20230616173212.png]]

![[Pasted image 20230616173244.png]]

### Graphic Raycaster 
Graphic Raycaster 意思是图形射线投射器（不是基于碰撞器，而是基于图形）
- **用于检测 UI 输入事件**
- 主要负责通过射线检测玩家和 UI 元素的交互，判断是否点击到了 UI 元素

![[Pasted image 20230616173440.png]]
**lgnore Reversed Graphics**: 是否忽略反转图形
- ? 反转指的是将控件的 Rect Transfrom 中的 Rotation 属性的 x 或 y 轴旋转 180 度
**Blocking Objects**:射线被哪些类型的碰撞器阻挡 (在覆盖渲染模式 Screen Space - Overlay下无效) 
**Blocking Mask**: 射线被哪些层级的碰撞器阻挡（在覆盖渲染模式下无效)

**演示：**
在一个 Button 控件前分别放一个 3D object（Cube） 和 2D object（Sprite），这两个 object 都要添加碰撞器，如下：
![[Pasted image 20230616175144.png]]
- 当 Blocking Objects 为 None 时，可以点击到 button
- 当 Blocking Objects 为 2D 时，右边点不到 button
- 当 Blocking Objects 为 3D 时，左边点不到 button
- 当 Blocking Objects 为 all 时，两边都点不到 button

### Event System
Event System 意思是事件系统
**玩家输入事件响应系统，主要用于监听玩家操作** 
![[Pasted image 20230616202924.png]]
- **它是用于管理玩家的输入事件并分发给各 UI 控件**
- 它是事件逻辑处理模块，**所有的 UI 事件都通过 EventSystem 组件中轮询检测并做相应的执行**
- 它类似一个中转站，和许多模块一起共同协作，如果没有它，所有点击、拖曳等等行为都不会被响应

`First Selected`: 首先选择的游戏对象，可以设置游戏一开始的默认选择 
`Send Navigation Events`: 是否允许导航事件（开启后可以通过键盘控制移动/按下/取消，wasd 移动，空格/回车选择)
`Drag Threshold`: 拖拽操作的阈值（移动多少像素的距离才算开始拖拽)

### Standalone Input Module
**独立输入模块组件，主要用于监听玩家操作** 
![[Pasted image 20230616203659.png]]
- 它主要针对处理鼠标/键盘/控制器/触屏 的输入
- 输入的事件通过 Event System 进行分发
- **它依赖于 Event System 组件，他们两缺一不可**

**和 Input Manager 中的设置绑定，一般不会进行修改：**
`Horizontal Axis`:水平轴按钮对应的热键名 (该名字对应 Input 管理器) 
`Vertical Axis`: 垂直轴按钮对应的热键名（该名字对应 Input 管理器)
`Submit Button`: 提交（确定)按钮对应的热建名（该名字对应 Input 管理器) 
`Cancel Button`:取消按钮对应的热建名 (该名字对应 Input 管理器)

`Input Actions Per Second`: 每秒允许键盘/控制器输入的数量 
`Repeat Delay`: 每秒输入操作重复率生效前的延迟时间
`ForceModule Active`: 是否强制模块处于激活状态

### 代码获取组件属性
```cs
//因为Transform是RectTransform父类，所以可以强转为RectTransform
print(((RectTransform)this.transform).sizeDelta);

//等价
print((this.transform as RectTransform).sizeDelta);
```

## 2 三大基础控件
### Image
![[Pasted image 20230616213350.png]]
- 是 UGUI 中用于显示精灵图片（Sprite）的关键组件
- 除了背景图等大图用RawImage，一般都使用 Image 来显示 UI 中的图片元素
![[Pasted image 20230616213356.png]]

- @ **控件显示顺序**：根据在 Canvas 下的层级，越后面的优先级越高:
![[Pasted image 20230616213645.png]]

- @ **Raycast Taget 示例**：在 Button 控件前加一个 Image 控件
    - 默认勾选，重叠部分无法点击Button
    - 取消勾选，重叠部分可以点击Button
![[Pasted image 20230616214219.png]]

- @ **ImageType 图片类型**
![[Pasted image 20230616215522.png]]
![[Pasted image 20230616220250.png#pic_center|400]]
- Simple：只用于固定尺寸的图片，美术出什么尺寸就用什么尺寸
- Sliced 切片模式：拉伸常用，需要设置图片边框 border
- Tiled-平铺模式：重复平铺中央部分，可以设置图片边框 border
- Filled-填充模式：效果较多，可以做血条 cd 等效果


**设置图片边框 border 的步骤：**
1. 找到图片，点击 SpriteEditor：
![[Pasted image 20230616214956.png|450]]
2. 拉动绿色线，将图片分割成九宫格区域。当拉伸图片时，横向拉伸只会拉伸竖向的中间一排，竖向拉伸只会拉伸横向的中间一排。四角不会发生拉伸
![[Pasted image 20230616215136.png|500]]

#### 代码获取 Image 属性
```cs file:代码获取Image属性
//修改当前Image控件的SourceImage
//图片必须放在Resources文件夹
Image img = this.GetComponent<Image>();
img.sprite = Resources.Load<Sprite>( "EmojiOne");
```

### Text
有两个版本
- Text(TMP)，基于 TextMeshPro
- Text(Legacy)，旧版
#### Text(TMP)

#### Text(Legacy)
![[Pasted image 20230616222442.png|450]]
![[Pasted image 20230616222359.png]]
#### 代码控制文本内容
```cs file:代码控制文本内容
//Text(TMP)
TextMeshPro text = this.GetComponent<TextMeshPro>();  
text.text = "Hello World";

//Text(Legacy)
Text txt = this.GetComponent<Text>();
txt.text ="Helloworld;
```

### RawImage
RawImage 是原始图像组件
**是 UGUI 中用于显示任何纹理图片的关键组件**

**和 Image 的区别：**
- 一般 RawImage **用于显示大图 (背景图、不需要打入图集的图片、网络下载的图等等)**。Image 则用于显示一些小的 UI 元素。
- RawImage 支持各种 Texture Type，Image 必须使用 Sprite

![[Pasted image 20230616223059.png]]
![[Pasted image 20230616223338.png]]
#### 代码控制 Texture
```cs file:代码控制Texture
RawImage img = this.GetComponent<RawImage>();
img.texture = Resources.Load<Texture>( "EmojiOne");
```

## 3 组合控件
### Button 按钮
**按钮组件**
是 UGUI 中用于处理玩家按钮相关交互的关键组件
![[Pasted image 20230616223909.png|450]]

![[Pasted image 20230616223923.png]]

![[Pasted image 20230616224314.png#pic_right]]

![[Pasted image 20230616232905.png]]

>1. Navigation 要联动 Event System：
> ![[Pasted image 20230616232956.png|500]]
>2. Explicit 指定周边控件：
>![[Pasted image 20230616233516.png|500]]
>3. 导航连线：
>![[Pasted image 20230616233439.png|350]]

#### 代码控制 button 属性
```cs file:代码控制button
Button btn = this.GetComponent<Button>();  
btn.interactable = true;  
btn.transition = Selectable.Transition.ColorTint;
...
```

#### 监听点击事件
点击事件是在按钮区域按下抬起一次就算一次点击
监听点击事件有两种方式：
1. 拖拽对象
![[Pasted image 20230616234235.png]]
只显示脚本上的 public 方法

2. 代码添加
```cs
void Start()
    {
        Button button = GetComponent<Button>();
        
        //添加监听，原理就是委托
        button.onClick.AddListener(ClickButton); 
        //也可以使用lambda表达式
        button.onClick.AddListener(() =>
        {
            print("另一种方式ClickButton");
        });
        
        //移除监听
        button.onClick.RemoveListener(ClickButton);
        
        //移除所有监听
        button.onClick.RemoveAllListeners();
    }

    public void ClickButton()
    {
        print("ClickButton");
    }
```

### 异形按钮
**异形即形状不规则**
普通的 Button 是根据矩形区域来响应点击，当我们使用带有透明部分的图片时，如图，点击透明区域也会响应，我们只想要不透明部分作为button
![[Pasted image 20230618194912.png]]

#### 方法一：添加子对象
按钮之所以能够响应点击，主要是根据图片矩形范围进行判断的
它的范围判断是**自下而上**的，意思是如果有子对象 Button，点击子对象 Button 的矩形范围也会让上面的 button 响应，那么我们就可运用多个透明图拼凑不规则图形作为按钮，子对象用于进行射线检测

如下图，先用一个 Image 作为背景图，然后修改各个 Button 按钮的矩形范围，拼出大致区域即可。
![[Pasted image 20230618201317.png]]

![[Pasted image 20230618201241.png]]

#### 方法二：通过代码改变图片的透明度响应阈值
1. 第一步: 修改图片参数开启 Read/ write Enabled 开关，会增大内存消耗
2. 第二步: 通过代码修改图片的响应阈值

```cs
public Image image;

private void Start()
{
    //该参数含义: 指定一个像素必须具有的最小 alpha 值，以便能够认为射线命中了图片。
    //说人话: 当像素点 alpha 值小于了该值就不会被射线检测了
    image.alphaHitTestMinimumThreshold = 0.1f;
}
```
### Toggle 开关
**开关组件**
是 UGUI 中用于处理玩家**单选框多选框相关交互的关键组件**
开关组件**默认是多选框**
>可以**通过配合 ToggleGroup 组件制作为单选框**（单选框就是多个框只能同时选择其中的一个）
>1. canvas 下创建一个空 object 命名为 GroupObject，添加 ToggleGroup 组件，然后将多个 Toggle 作为其子对象（Allow Switch Off 即是否允许所有选项都为关闭状态）![[Pasted image 20230617231302.png]] ![[Pasted image 20230617231207.png]]
>2. 每个 Toggle 的 Group 都设置为 GroupObject ![[Pasted image 20230617231154.png]]

默认创建的 Toggle 由 4 个对象组成
- 父对象：Toggle 组件依附
- 子对象：背景图 (必备)、选中图 (必备)、说明文字 (可选)

![[Pasted image 20230617230340.png|400]]

Interactable、Transition、Navigation 设置和 Button 一致
![[Pasted image 20230617230510.png]]

#### 代码控制
```cs
Toggle toggle = this.GetComponent<Toggle>();  
toggle.isOn = true;  
  
ToggleGroup toggleGroup = this.GetComponent<ToggleGroup>();  
toggleGroup.allowSwitchOff = true;  
  
//通过迭代器便利的到处于选中状态的Toggle  
foreach (Toggle item in toggleGroup.ActiveToggles())  
{  
print(item.name);  
}
```

#### 监听点击事件
点击事件是在按钮区域按下抬起一次就算一次点击
监听点击事件有两种方式：
1. 拖拽对象，注意选择的函数必须有 bool 形参，表示打开和关闭
![[Pasted image 20230617232706.png]]

1. 代码添加
```cs
void Update()  
{  
Toggle toggle = GetComponent<Toggle>();  
tog.onValueChanged.AddListener(ChangeValue);  
}  
  
//必须传bool形参  
public void ChangeValue(bool isOn)  
{  
print("状态改变" + isOn);  
}
```


### InputField  输入字段
**输入字段组件**
是 uG 中用于**处理玩家文本输入相关交互**的关键组件

默认创建的 InputField 由 3 个对象组成
父对象：InputField 组件依附对象，以及同时在其上挂载了一个 Image 作为背景图
子对象：文本显示组件 (必备)、默认显示文本组件 (必备)

#### InputField (TMP)

#### InputField (Legacy)
![[Pasted image 20230617233548.png]]

**Content Type：**
![[Pasted image 20230617233722.png|450]]

```cs
InputField input = this.GetComponent<InputField>();print(input.text);
input.text = "123123123123";
```

### Slider 滑动条
**滑动条组件**
是 UGUI 中用于处理滑动条相关交互的关键组件

默认创建的 Slider 由 4 组对象组成
父对象：Slider 组件依附的对象
子对象：背景图、进度图、滑动块三组鸡象

![[Pasted image 20230617234615.png]]

![[Pasted image 20230617234645.png|450]]
![[Pasted image 20230617234657.png]]

```cs
Slider slider = GetComponent<Slider>();
slider.value += 0.01f;
```

### Scrollbar 滚动条
**滚动条组件**

是 UGUI 中**用于处理滚动条相关交互**的关键组件

默认创建的 scrollbar 由 2 组对象组成
父对象：Scrollbar 组件依附的对象子对象
滚动块对象：**一般情况下我们不会单独使用滚动条，都是配合 ScrollView 滚动视图来使用**

![[Pasted image 20230617235146.png]]
### ScrollView 滚动视图
**滚动视图组件**
是 UGUI 中**用于处理滚动视图相关交互**的关键组件

默认创建的 ScrollRect 由 4 组对象组成
父对象：ScrollRect 组件依附的对象，还有一个 Image 组件最为背景图
子对象：
Viewport 控制**滚动视图可视范围**和 Content **控制内容范围** （内部控件都放在 Content 下面 ）
Scrollbar Horizontal 水平滚动条
Scrollbar Vertical 垂直滚动条

![[Pasted image 20230617235616.png|400]]
![[Pasted image 20230617235604.png]]

### DrawDown 下拉列表
**下拉列表（下拉选单)组件**
是 UGUI 中**用于处理下拉列表相关交互**的关键组件

默认创建的 DropDown 由 4 组对象组成
父对象：DropDown 组件依附的对象还有一个 Image 组件作为背景图
子对象：
Label 是当前选项描述 
Arrow 右侧小箭头
Template 下拉列表选单

#### DrawDown (TMP)

#### DrawDown (Legacy)
![[Pasted image 20230618001017.png|350]]
![[Pasted image 20230618000758.png|500]]


```cs
 Dropdown dropdown = GetComponent<Dropdown>();
print(dropdown.value);

print(dropdown.options[dropdown.value].text);
dropdown.options.Add(new Dropdown.OptionData("新增选项"));
```

## 4 自动布局组件
虽然 UGUI 的 RectTransform 已经非常方便的可以帮助我们快速布局，但 UGUI 中还提供了很多可以帮助我们对 UI 控件进行自动布局的组件，他们可以帮助我们**自动的设置 UI控件的位置和大小等**

**自动布局的工作方式**：自动布局控制组件 + 布局元素 = 自动布局 

**自动布局控制组件**：unity 提供了很多用于自动布局的管理性质的组件用于布局
**布局元素**： 具备布局属性的对象们，这里主要是指具备 RectTransform 的 UI 组件

**要参与自动布局的布局元素必须包含布局属性，布局属性主要有以下几条**
`Minmum width`: 该布局元素应具有的最小宽度
`Minmum height`: 该布局元素应具有的最小高度

`Preferred width`: 在分配额外可用宽度之前，此布局元素应具有的宽度
`Preferred height`: 在分配额外可用高度之前，此布局元素应具有的高度。

`Flexible width`: 此布局元素应相对于其同级而填充的额外可用宽度的相对量 
`Flexible height`: 此布局元素应相对于其同级而填充的额外可用高度的相对量

**在进行自动布局时都会通过计算布局元素中的这 6 个属性得到控件的大小位置**
- ! **一般情况下我们不会去手动修改他们**，但是如果你有这些需求，可以手动添加一个 `LayoutElement` 组件，可以修改这些布局属性。
![[Pasted image 20230618205443.png]]
>控件 Insepctor 最下方可以查看布局属性

在布局时，**布局元素大小设置的基本规则：**
1. 首先分配最小大小 `Minmum width` 和 `Minmum height`
2. 如果父类容器中有足够的可用空间，则分配 `Preferred width` 和 `Preferred height`
3. 如果上面两条分配完成后还有额外空间，则分配 `Flexible width` 和 `Flexible height`

一般情况下布局元素的这些属性都是 0，但是特定的 UI 组件依附的对象布局属性会被改变，比如 Image 和 Text

#### 水平垂直组件
**组件名**：Horizontal Layout Group 和 Vertical Layout Group
**将子对象并排或者竖直的放在一起**

通常将组件给父对象，那么子对象就会自动布局，如图，红色 Image 作为父对象，其他颜色 Image 作为子对象，父对象添加 Horizontal Layout Group 组件：
![[Pasted image 20230618210116.png]] ![[6a7sd15a1da.gif|500]]

![[Pasted image 20230618210413.png]]
参数相关:
Padding: 左右上下边缘偏移位置
Spacing: 子对象之间的间距
ChildAlignment: 九宫格对其方式
Control Child size: 是否控制子对象的宽高
Use child Scale: 在设置子对象大小和布局时，是否考虑子对象的缩放
child Force Expand: 是否强制子对象拓展以填充额外可用空间

#### 网格布局组件
**组件名**: Grid Layout Group
**将子对象当成一个个的格子设置他们的大小和位置**
![[6a7sd15a1da 1.gif|550]]
![[Pasted image 20230618211335.png]]
参数相关:
Padding: 左右上下边缘偏移位置 
Cell size: 每个格子的大小 
Spacing: 格子间隔
Start Corner:第一个元素所在位置 (4 个角)
Start Axis: 沿哪个轴放置元素：Horizontal 水平放置满换行，Vertical 竖直放置满换列 
Child Alignment: 格子对其方式（9宫格)
Constraint: 行列约束
Flexible: 灵活模式，根据容器大小自动适应 
Fixed column Count: 固定列数
Fixed Row Count: 固定行数

#### 内容大小适配器
**组件名**: Content size Fitter
它可以**自动的调整 RectTransform 的长宽来让组件自动设置大小**
一般在 Text 上使用或者配合其它布局组件一起使用

![[Pasted image 20230618212411.png]]
参数相关
Horizontal Fit: 如何控制宽度 
Vertical Fit: 如何控制高度
Unconstrained: 不根据布局元素伸展
Min size: 根据布局元素的最小宽高度来伸展
Preferred Size: 根据布局元素的偏好宽度来伸展宽度。

**常用情景，背包动态扩容**
![[Pasted image 20230618211900.png]]
为 Content 添加一个网格布局组件，然后不断添加 Image ，我们发现，随着 Image 数量增多，Content 的 Rect 高度并没有增加，这就导致，滚轮无法查看所有格子：
![[6a7sd15a1dafasf.gif]]
我们只需为 Content 添加内容大小适配器，将 Verticla Fit 设置为 Preferred Size，就可以了：
![[6a7sd15a1dafasf4.gif]]
#### 宽高比适配器
组件名: Aspect Ratio Fitter

让布局元素按照一定比例来调整自己的大小，使布局元素在父对象内部根据父对象大小进行适配
![[Pasted image 20230618212719.png]]
参数相关:
Aspect Mode: 适配模式, 如果调整矩形大小来实施宽高比
None: 不让矩形适应宽高比
width Controls Height: 根据宽度自动调整高度 
Height Controls width: 根据高度自动调整宽度
Fit In Parent: 自动调整宽度、高度、位置和锚点，使矩形适应父项的矩形，同时保持宽高比，会出现黑边
Envelope Parent: 自动调整宽度、高度、位置和锚点，使矩形覆盖父项的整个区域，同时保持宽高比，会出现“裁剪
Aspect Ratio: 宽高比; 宽除以高的比值

## 5  图集 (需要补一下 Unity 核心)
UGUI 和 NGUI 使用上最大的不同是：NGUI 使用前就要打图集，UGUI 可以再之后再打图集
**打图集的目的就是减少 Drawcall 提高性能**，我们可以通过打图集，将小图合并成大图，将本应 n 次的 Drawcall 变成 1 次 Drawcall 来提高性能。

### Sprite Packer
**Sprite Packer (精灵包装器，可以通过 Unity 自带图集工具生成图集)**
Edit->Project Setting->Editor
![[Pasted image 20230618001804.png]]

1. **Disabled**: 默认设置, 不会打包图集
2. **Enabled For Build（常用）**: Unity 仅在构建时打包图集，在编辑器模式下不会打包
3. **Always Enabled（常用）**: Unity 在构建时打包图集，在编辑模式下运行前会打包图集

### 图集参数
创建图集：create->2D->Sprite Atlas
![[Pasted image 20230618141523.png|500]]
图集打包后，不要让外部部件插入其中，这样会增加 drawcall

```cs
//加载图集注意:需要引用命名空间  
SpriteAtlas sa = Resources.Load<SpriteAtlas>( "MyAlas");  
//从图集中加载指定名字的小图  
sa.GetSprite("bk");
```

## 6 UI 事件接口
 
目前所有的控件都只提供了常用的事件监听列表
如果想做一些类似长按，双击，拖拽等功能是无法制作的，或者想让 Image 和 Text, RawImage 三大基础控件能够响应玩家输入也是无法制作的
而事件接口就是用来处理类似问题，让所有控件都能够添加更多的事件监听来处理对应的逻辑

**常用事件接口：**
`IPointerEnterHandler` - `OnPointerEnter` -当指针进入对象时调用 (鼠标进入) 
`IPointerExitHandler` - `OnPointerExit` -当指针退出对象时调用 (鼠标离开)
`IPointerDownHandler` - `OnPointerDown` -在对象上按下指针时调用  (按下)
`IPointerUpHandler` - `OnPointerUp` -松开指针时调用（在指针正在点击的游戏对象上调用)（抬起）
`IPointerClickHandler` - `OnPointerclick` -在同一对象上按下再松开指针时调用 (点击)

`IBeginDragHandler` - `OnBeginDrag`-即将开始拖动时在拖动对象上调用 (开始拖拽）
`IDragHandler` - `OnDrag` - 发生拖动时在拖动对象上调用 (拖拽中)
`IEndDragHandler`- `OnEndDrag` -拖动完成时在拖动对象上调用 (结束拖拽)

![[Pasted image 20230618142218.png]]

### 使用方法
1. 继承 MonoBehavior 的脚本继承对应的事件接口，引用命名空间 
2. 实现接口中的内容
3. 将该脚本挂载到想要监听自定义事件的 UI 控件上 
```cs
//需要什么接口就继承什么
public class Test : MonoBehaviour, IPointerEnterHandler,IPointerClickHandler
{
    //实现接口内容
    public void OnPointerEnter(PointerEventData eventData)
    {
        print("鼠标进入");
    }

    public void OnPointerClick(PointerEventData eventData)
    {
        print("鼠标点击");
    }
}
```
### PointerEventData
上面实现的接口内容都有一个 `PointerEventData` 类型的参数
父类: `BaseEventData` 

`pointerId`: 鼠标左右中键点击鼠标的 ID ，通过它可以判断左中右键点击，对一个 ID 分别为-1，-2，-3
`position`:当前指针位置 (屏幕坐标系)
`pressPosition`: 按下的时候指针的位置 delta: 指针移动增量
`clickCount`: 连击次数 clickTime: 点击时间
`pressEventCamera`: 最后一个 `onPointerPress` 按下事件关联的摄像机 
`enterEvetnCamera`: 最后一个 `onPointerEnter` 进入事件关联的摄像机

```cs file:使用方法
public void OnPointerClick(PointerEventData eventData)
{
    print("鼠标点击");

    //获取鼠标点击ID
    print(eventData.pointerId);
}
```

### EventTrigger 
事件触发器是 EventTrigger 组件
它是一个集成了上节课中学习的所有事件接口的脚本，它可以让我们更方便的为控件添加事件监听

直接在 UI 控件上添加 EventTrigger 即可：
![[Pasted image 20230618143456.png]]

**使用方法：**
1. 直接拖脚本关联，注意传入的函数参数为 BaseEventData 类型
![[Pasted image 20230618143557.png]]

```cs
public void TestPointerEnter(BaseEventData eventData)
    {
        //如果想获取其他信息，就转换成PointerEventData类型
        PointerEventData pointerEventData = eventData as PointerEventData;
        print("鼠标进入" +  pointerEventData.position);
    }
```

2. 代码关联
```cs
//声明事件
EventTrigger.Entry entry = new EventTrigger.Entry();

//设置事件类型
entry.eventID = EventTriggerType.PointerEnter;

//设置回调函数
entry.callback.AddListener((data)=>{ Debug.Log("鼠标进入"); });

//添加事件
eventTrigger.triggers.Add(entry);
```


## 7 屏幕坐标转 UI 坐标
`RectTransformUtility` 公共类是一个 `RectTransform` 的辅助类主要用于进行一些坐标的转换等等操作，其中对于我们目前来说最重要的函数是将屏幕空间上的点，转换成 UI 本地坐标下的点

方法:
`RectTransformUtility. ScreenPointToLocalPointInRectangle`
**参数一**：相对父对象
**参数二**：屏幕点
**参数三**：摄像机
**参数四**：最终得到的点
一般配合拖拽事件使用

![[Pasted image 20230618144851.png]]
![[Pasted image 20230618145514.png]]
将以下脚本挂在给子对象，可以通过鼠标拖动 image 子对象相对于父对象移动
```cs
public class Test : MonoBehaviour , IDragHandler
{
    public void OnDrag(PointerEventData eventData)
    {
        RectTransformUtility.ScreenPointToLocalPointInRectangle(
            this.transform.parent as RectTransfor,
            eventData.position,
            eventData.enterEventCamera,
            out Vector2 localPoint);
        
        this.transform.localPosition = localPoint;
    }
}
```

## 9 Mask 遮罩
实现遮罩效果的关键组件时 Mask 组件
通过**在父对象上添加 Mask 组件**即可遮罩其子对象

注意:
1. 想要被遮罩的 Image 需要勾选 Maskable
2. 只要父对象添加了 Mask 组件，那么所有的 UI 子对象都会被遮罩
3. 遮罩父对象图片的制作，不透明的地方显示，透明的地方被遮罩

 遮罩前： ![[Pasted image 20230618150128.png]]
 使用遮罩： ![[Pasted image 20230618150119.png]]

## 10 模型和粒子显示在 UI 之前
### 方法一：直接用摄像机渲染 3D 物体
Canvas 的渲染模式：摄像机模式和世界 (3D)模式都可以让模型显示在 UI 之前（(Z 轴在 UI 元素之前即可)
注意:
1. 摄像机模式时建议用专门的摄像机渲染 UI 相关 
2. 面板上的 3D 物体建议也用 UI 摄像机进行渲染

![[#Screen Space - Camera]]

### 方法二：渲染在 RT 上，通过 RawImage 显示

专门使用一个摄像机渲染 3D 模型，将其渲染内容输出到 Render Texture 上，类似小地图的制作方式
再将渲染的图显示在 UI 上
该方式不管 canvas 的渲染模式是哪种都可以使用

 ![[Pasted image 20230618151231.png|500]]
1. 创建一个专用摄像机，将想要渲染的模型单独设置一个 Layer，将摄像机的 CullingMask 设置为该 Layer，之渲染该模型
2. Create->RenderTexture，将创建的 RT 传给摄像机
3. Canvas 下创建一个 RawImage（RawImage 支持各种 Texture Type），将 RT 传过去就可以了。

### 方法三：粒子系统 Order in Layer
 
**粒子系统也可以使用方法一和方法二，同时有一个单独的方法：**
canvas 和粒子系统都有一个层级排序选项，通过修改粒子系统的序号，让值大于 Canvas，即可实现忽略 z 轴，粒子始终显示在 UI 前
![[Pasted image 20230618151821.png|500]]

![[Pasted image 20230618151734.png|550]]

## 11 CanvasGroup
为面板父对象添加 CanvasGroup 组件即可同时控制一组 Canvas
常用于整体控制一个面板的淡入淡出或者整体禁用

![[Pasted image 20230618212954.png]]
参数相关:
Alpha: 整体透明度控制
Interactable: 整体启用禁用设置 
Blocks Raycasts: 整体射线检测设置
Ignore Parent Groups: 是否忽略父级 CanvasGroup 的作用

## 12 常用插件
DoTween—缓动插件，可以制作一些缓动效果

TextMeshPro: 一文本网格插件，可以制作更多的特效文字
