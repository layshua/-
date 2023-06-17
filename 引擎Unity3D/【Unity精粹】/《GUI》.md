
---
title: 《GUI》
aliases: []
tags: []
create_time: 2023-06-07 23:35
uid: 202306072335
banner: "![[diablo-iv-beta-vendor-3.png]]"
---

# 九宫格 UI
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

## 5 Toggle 切换
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

## 六大基础组件

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
3. 如果需要在 UI 上显示 3D 模型，直接在 Canvas 上创建即可
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
    - 有三种模式：最常使用的是 Match Width Or Height 模式，套路如下：
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

## 三大基础控件
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

## 组合控件
### Button
**Button 是按钮组件，是 UGUI 中用于处理玩家按钮相关交互的关键组件**
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
