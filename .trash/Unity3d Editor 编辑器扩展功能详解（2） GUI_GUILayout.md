## 前言

总目录传送：[李恒：Unity Editor 编辑器扩展功能详解（1） 目录索引](https://zhuanlan.zhihu.com/p/503154643)

下一章节： [Unity Editor 编辑器扩展功能详解（3） EditorGUI/EditorGUILayout（上）](https://zhuanlan.zhihu.com/p/505159658)

GUI/GUILayout 主要是用于 Game 视图的基本 UI 信息显示与逻辑操作，例如

![](https://pic4.zhimg.com/v2-40d8076cb62de6134c1e1dc928bd6bf7_r.jpg)

这在某些插件 / demo 工程里经常见到。

由于本系列旨在快速查询对应功能模块，故行文不遵循 “是什么 / 为什么 / 怎么做” 的方法论，而是 “关键字 / 图例 / 代码” 的格式，便于快速查阅目标功能。

## 一、按钮类组件

### 1. 普通按钮（Button）

关键字： GUILayout.Button

样例：

![](https://pic1.zhimg.com/v2-8be618ecccd084b114be659ccd56a3f8_b.jpg)

示例代码：

```
public class GUITutorialDemo : MonoBehaviour //这是在Game视图中绘制GUI，所以需要继承于一个Mono
{
    private void OnGUI() //所有GUI绘制都必须在这个函数内部： MonoBehaviour.GUI() {
        if (GUILayout.Button("I'm a button")) //一个简单的button绘制代码
        {
            Debug.Log("hello world"); //点击按钮触发的逻辑
        }
    }
}
```

注：所有 GUI 绘制都会在 MonoBehaviour.GUI()，为了方便阅读，后文只会给出组件关键代码，实际位置如上述所示。

### 2. 选择按钮（Selection）

关键字： GUILayout.SelectionGrid

样例：

![](https://pic3.zhimg.com/v2-60e84ce0c45c4b11ce7259a82f837b56_b.jpg)

示例代码：

```
private int index; //定义多按钮选择的index
private string[] buttonNames = new string[] { "First", "Second", "Third" }; //定义多按钮的名称

...

//第一个参数是当前选择按钮的index。
//第二个参数是多个按钮的名称，数组长度决定了按钮个数。
//第三个参数是表示一行按钮有多少个，这里是2，表示一行最多2个按钮。
index = GUILayout.SelectionGrid(index, buttonNames, 2);
```

### 3. 工具栏（Toolbar）

关键字： GUILayout.Toolbar

样例：

![](https://pic4.zhimg.com/v2-07dffe79077a51e2989f098cc3191c2b_b.jpg)

示例代码：

```
private int index;
private string[] buttonNames = new string[] { "First", "Second", "Third" };

...
//第一个参数是当前选择 按钮 的index。
//第二个参数是多个 按钮 的名称，数组长度决定了 按钮 个数。
index = GUILayout.Toolbar(index, buttonNames);
```

### 4. 切换键（Toggle）

关键字： GUILayout.Toggle

样例：

![](https://pic1.zhimg.com/v2-8587cd4f30aa130ec26f87acc6fcbab0_b.jpg)

示例代码：

```
private bool toggleValue; //记录当前toggle真值情况
...
toggleValue = GUILayout.Toggle(toggleValue, "This is a toggle"); //第二个参数是Toggle文本信息
```

## 二、 文本标签类

### 1. 文本标签（Label）

关键字： GUILayout.Label

样例：

![](https://pic2.zhimg.com/v2-cb180bf68aafd44677a20d58df559f6d_b.jpg)

示例代码：

```
GUILayout.Label("This is a Label Text");
```

### 2. 文本输入区域（TextField）

关键字： GUILayout.TextArea， GUILayout.TextField

样例：

![](https://pic3.zhimg.com/v2-666dfc0d5168f9513d67e2df6792e8d6_r.jpg)

示例代码：

```
private string areaText = "multi-line text";
private string singleText = "single-line text..";
... 
areaText = GUILayout.TextArea(areaText); // Make a multi-line text field where the use can edit a string
singleText = GUILayout.TextField(singleText); //Make a single-line text field where the use can edit a string
//TextArea 支持多行编辑，就是回车之后文本也能换行。
//Textfield 不支持多行编辑，回车之后没反应。
//（但是如果文本有类似\n的换行符，文本也会换行，就是不支持换行编辑而已）
```

## 三、滚动滑条类

### 1. 滑条（Slider）

关键字：GUILayout.HorizontalSlider， GUILayout.VerticalSlider

样例：

![](https://pic4.zhimg.com/v2-a482da1f3f2a9f28c1165483ba91f7b3_r.jpg)

示例代码：

```
private float horizontalValue = 0;
private float verticalValue = 0;
...
//为了保证水平滑条显示，最后一个参数使用了GUILayout.Width(300),这是一个控件布局参数，之后会介绍。
horizontalValue = GUILayout.HorizontalSlider(horizontalValue, 0, 100, GUILayout.Width(300)); 
verticalValue = GUILayout.VerticalSlider(verticalValue, 0, 100);
```

### 2. 滚动条（ScrollBar）

关键字： GUILayout.HorizontalScrollbar， GUILayout.VerticalScrollbar

样例：

![](https://pic4.zhimg.com/v2-223f43c6788649542c6a994a5f55355f_r.jpg)

示例代码：

```
private float horizontalValue = 0; //滑动条数值
private float verticalValue = 0; //滑动条数值
private float blockSize = 10; //滑动块大小
private float leftValue = 0; //最左侧/最上侧数值
private float rightValue = 100; //最右侧/最下侧数值
...
horizontalValue = GUILayout.HorizontalScrollbar(horizontalValue, blockSize, leftValue, rightValue, GUILayout.Width(300));
verticalValue = GUILayout.VerticalScrollbar(verticalValue, blockSize, leftValue, rightValue);
```

## 四、区块布局类

### 1. 方形区域（Box）

关键字： GUILayout.Box

样例：

![](https://pic2.zhimg.com/v2-d336d6415a1599cb2e77b23a2bb17631_r.jpg)

示例代码：

```
//后两个参数定义了box area的长宽，后文会介绍参数意义
GUILayout.Box("Box Area", GUILayout.Width(200), GUILayout.Height(200));
```

### 2. 可视化区域（Area）

关键字： GUILayout.BeginArea， GUILayout.EndArea

样例：

![](https://pic3.zhimg.com/v2-e88a48a8af3ec9bbce86dbdde2d2a7e2_r.jpg)

示例代码：

```
GUILayout.BeginArea(new Rect(0, 0, 300, 700)); //定义区域开始，rect参数： x坐标，y坐标，宽度，高度
GUILayout.Button("Button in Area");  //用一个button作为显示
GUILayout.EndArea(); //结束区域定义
```

### 3. 水平 / 垂直布局（Horizontal/Vertical）

关键字：GUILayout.BeginHorizontal，GUILayout.EndHorizontal，GUILayout.BeginVertical，GUILayout.EndVertical

样例：

![](https://pic3.zhimg.com/v2-0295830014d6d3a6eb9bd47d5c31da32_b.jpg)

示例代码：

```
GUILayout.BeginHorizontal(); //开始水平布局
GUILayout.Button("Horizontal Button No.1"); //组件1
GUILayout.Button("Horizontal Button No.2"); //组件2
GUILayout.EndHorizontal(); //结束水平布局

GUILayout.BeginVertical();//开始垂直布局
GUILayout.Button("Veritical Button No.1"); //组件1
GUILayout.Button("Veritical Button No.2"); //组件2
GUILayout.EndVertical(); //结束垂直布局
//值得注意的是，布局方法一般需要成对使用：定义开始/结束，否则会在布局上出现奇怪的bug
```

### 4. 滑动视图（ScrollView）

关键字：GUILayout.BeginScrollView，GUILayout.EndScrollView

样例：

![](https://pic4.zhimg.com/v2-fe11bd7766ffbf6583d8e64c33830c3b_r.jpg)

示例代码：

```
private Vector2 scrollViewRoot; //定义滑动窗口当前滑动值，如果不使用这个vector2，滑动窗口将无法滑动
...
GUILayout.BeginArea(new Rect(0, 0, 400, 400)); //定义可视化区域
scrollViewRoot = GUILayout.BeginScrollView(scrollViewRoot); //定义滑动视图
GUILayout.Button("Buttons", GUILayout.Height(200)); //定义组件1，高度200
GUILayout.Button("Buttons", GUILayout.Height(200)); //定义组件2，高度200
GUILayout.Button("Buttons", GUILayout.Height(200)); //定义组件3，高度200
GUILayout.EndScrollView(); //结束滑动窗口
GUILayout.EndArea();// 结束区域定义

//由于区域是400x400，有3个高度为200的button，3*200>400，视图内容会越界，滑动窗口才有意义。
```

### 5. 空白间隔（Space）

关键字： GUILayout.Space，GUILayout.FlexibleSpace

样例：

![](https://pic1.zhimg.com/v2-32f06b86804746d62d2401169faa7df4_r.jpg)

示例代码：

```
GUILayout.BeginArea(new Rect(0, 0, 400, 400)); //定义400 x 400 可视化区域

GUILayout.BeginHorizontal(); //开始水平布局
GUILayout.Button("B1", GUILayout.Width(100)); //定义宽度100的按钮
GUILayout.FlexibleSpace(); //插入一个“灵活的”空白组件
GUILayout.Button("B2", GUILayout.Width(50)); //定义宽度50的按钮
GUILayout.EndHorizontal(); //结束水平布局
GUILayout.Button("length"); //定义默认宽度按钮，其宽度会自动扩展到区域宽度：400

GUILayout.Space(100); //手动间隔100个像素单位

GUILayout.BeginHorizontal(); //开始水平布局
GUILayout.Button("B1", GUILayout.Width(100)); //定义宽度100的按钮
GUILayout.Space(200); //手动间隔200个像素单位
GUILayout.Button("B2", GUILayout.Width(50));//定义宽度50的按钮
GUILayout.EndHorizontal();//结束水平布局
GUILayout.Button("length"); //定义默认宽度按钮，其宽度会自动扩展到区域宽度：400

GUILayout.EndArea(); //结束区域定义

//两者区别就在上述样例图片中，这是手动间隔排版所用方法
```

## 五、控件属性（GUILayoutOption）

关键字：

```
GUILayout.Width(float width) // 设置控件的宽度
GUILayout.Height(float height) // 设置控件的高度
GUILayout.MinWidth(float width) // 设置控件的最小宽度
GUILayout.MinHeight(float height) // 设置控件的最小高度
GUILayout.MaxWidth(float width)// 设置控件的最大宽度
GUILayout.MaxHeight(float width) // 设置控件的最大高度
GUILayout.ExpandHeight(bool expand) // 是否允许自动扩展高度
GUILayout.ExpandWidth(bool expand) // 是否允许自动扩展宽度
```

样例：

![](https://pic1.zhimg.com/v2-f0ce54ed57a46c08fe8bfb2bc0fdecf8_r.jpg)

示例代码：

```
GUILayout.BeginArea(new Rect(0, 0, 700, 400)); //定义宽度700的区域
GUILayout.Button("Default Button"); //默认button，会自动扩展宽度
GUILayout.Button("Button with 100 width", GUILayout.Width(100)); //定义100宽度，不会自动扩展
GUILayout.Button("Button with 200 minWidth", GUILayout.MinWidth(200)); //定义最小宽度，会自动扩展，由于扩展的目标是700，所以与默认button相同
GUILayout.Button("Button with 300 maxWidth", GUILayout.MaxWidth(300)); //定义最大宽度，会自动扩展，但是扩展目标是700，大于300，所以还是300宽度
GUILayout.Button("Button with false expand", GUILayout.ExpandWidth(false)); //禁止自动扩展，宽度是文本自适应宽度。
GUILayout.EndArea(); //结束区域定义
```

## 结语

一般的，GUILayout 除了在 Game 视图进行部分 runtime 的逻辑操作，还会与 EditorGUILayout 配合，用于在 Editor 编辑器模式下的扩展功能。

## Reference:

[UnityEditor 编辑器扩展_w_mumu_q 的博客 - CSDN 博客](https://blog.csdn.net/w_mumu_q/article/details/107240127)

[Unity 编辑器扩展: GUILayout、EditorGUILayout 控件整理](https://blog.csdn.net/linxinfa/article/details/87863123)