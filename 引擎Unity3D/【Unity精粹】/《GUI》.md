
---
title: 《GUI》
aliases: []
tags: []
create_time: 2023-06-07 23:35
uid: 202306072335
banner: "![[diablo-iv-beta-vendor-3.png]]"
---
[Screenshots | Interface In Game | Video games UI](https://interfaceingame.com/screenshots/)
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