# GUI
## 1 原理及作用
**IMGUI**  （Immediate Mode Graphical User Interface）即时模式图形化交互界面
IMGUI 在 Unity 中一般简称为 GUI，它是一个代码驱动的 UI 系统

作用：
1. 作为程序员的调试工具，创建游戏内调试工具
2. 为脚本组件创建自定义检视面板
3. 创建新的编辑器窗口和工具以拓展 Unity 本身（一般用作内置游戏工具）
4. **不适合用它为玩家制作 UI 功能**

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

//输入框
//比较特别是的第三个参数，最大输入长度
inputStr = GUI.TextField(new Rect(0,0,100,50), inputStr,5); 

//密码输入,输入全被*遮盖
password = GUI.PasswordField(new Rect(100,0,100,50), password, '*');

//水平拖动条
nowValue = GUI.HorizontalSlider(new Rect(200, 0, 100, 100), nowValue,0.0f,1.0f);

//竖直拖动条GUI.VerticalSlider()
```

注意要初始化 string ，不然会报错：
![[Pasted image 20230607210648.png]]
## 7 图片绘制和Box框
![[Pasted image 20230607212445.png]]
```cs
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
```cs
GUI.Box(new Rect(0,0,100,100),"123");
```

## 8 工具栏和选择网格
