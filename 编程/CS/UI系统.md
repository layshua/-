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

## 2 重要参数及文本和按钮

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


### 文本控件
![[Pasted image 20230607151421.png|700]]
![[Pasted image 20230607151337.png|450]]

```cs
public Texture texture;  

//骷髅
public Rect rect1;  

//小黄脸
public Rect rect2;  
public GUIContent content2;

private void OnGUI()  
{
    //基本使用
    GUI.Label(new Rect(0,0,100,20),"Hello World"); //text，传位置信息也可以直接声明公共变量Rect，如下：
    GUI.Label(rect1, texture);  //图片
    
    //综合使用：GUIContent可以控制text
    GUI.Label(rect2,content2);
    Debug.Log(GUI.tooltip);  //获取当前鼠标或者键盘选中的GUI控件对应的tooltip信息
}
```