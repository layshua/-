## 前言：

全系列目录索引：[Unity3d Editor 编辑器扩展功能详解（1） 目录索引](https://zhuanlan.zhihu.com/p/503154643)

上一章节：[Unity Editor 编辑器扩展功能详解（3） EditorGUI/EditorGUILayout（上）](https://zhuanlan.zhihu.com/p/505159658)

下一章节：[Unity3d Editor 编辑器扩展功能详解（4）Gizmos](https://zhuanlan.zhihu.com/p/515008289)

最近突然有了一种：这种文章不写白不写，写了也白写的感觉。

当 Editor 工具人很有意思嘛？看看别人 Gameplay 正反馈那么强，别人 TA 的优化渲染那么酷炫，就你个铁憨憨写开发工具，呜呜呜。

## 一. 折叠栏

### 1. 折叠（**Foldout**）

关键字： EditorGUILayout.Foldout

样例：

![](https://pic2.zhimg.com/v2-24e57148de2235c9b79c5afbb966b885_r.jpg)

示例代码：

```
private bool foldOut;
...
foldOut = EditorGUILayout.Foldout(foldOut, "一般路过折叠栏");
if (foldOut) //只有foldout为true时，才会显示下方内容，相当于“折叠”了。
{
        EditorGUILayout.LabelField("这是折叠标签内容");
}
```

### 2. 折叠组（**FoldoutGroup**）

关键字：EditorGUILayout.BeginFoldoutHeaderGroup， EditorGUILayout.EndFoldoutHeaderGroup

样例：

![](https://pic3.zhimg.com/v2-2182db3d8cd06a956dd75ad54710dd82_r.jpg)

示例代码：

```
private bool foldOut;
...
foldOut = EditorGUILayout.BeginFoldoutHeaderGroup(foldOut, "折叠栏组");
if (foldOut) //只有foldout为true时，才会显示下方内容，相当于“折叠”了。
{
        EditorGUILayout.LabelField("这是折叠标签内容");
}
EditorGUILayout.EndFoldoutHeaderGroup(); //只不过这种折叠需要成对使用，不然会有BUG
```

## 二. 开关控件

### 1. 单个开关（Toggle）

关键字： EditorGUILayout.Toggle，EditorGUILayout.ToggleLeft

样例：

![](https://pic3.zhimg.com/v2-6132af52d0c63c2f0e61988a9056cc56_r.jpg)

示例代码：

```
private bool toggle;
...
toggle = EditorGUILayout.Toggle("Normal Toggle", toggle);
toggle = EditorGUILayout.ToggleLeft("Left Toggle", toggle);
```

### 2. 开关组（ToggleGroup）

关键字：EditorGUILayout.BeginToggleGroup，EditorGUILayout.EndToggleGroup

样例：

![](https://pic4.zhimg.com/v2-716c04feabe3def4018bec80e00f22e3_r.jpg)

示例代码：

```
private bool toggle;
private string m_inputText;
...
toggle = EditorGUILayout.BeginToggleGroup("开关组", toggle);
EditorGUILayout.LabelField("------Input Field------");
m_inputText = EditorGUILayout.TextField("输入内容：", m_inputText);
EditorGUILayout.EndToggleGroup();
```

## 三. 滑动控件

### 1. 滑动条（Slider）

关键字：EditorGUILayout.Slider，EditorGUILayout.IntSlider

样例：

![](https://pic1.zhimg.com/v2-f0ab6ef60727500eecd240f56a4486c0_r.jpg)

示例代码：

```
private float m_sliderValue;
private int m_sliderIntValue;
...        
//参数详解：
//第一个参数： string label 控件名称标签 
//第二个参数： float value 滑动当前值 
//第三个参数： float leftValue 滑动最小值 
//第四个参数： float rightValue 滑动最大值
m_sliderValue = EditorGUILayout.Slider("滑动条Sample：", m_sliderValue, 0.123f, 7.77f);
//参数同上
m_sliderIntValue = EditorGUILayout.IntSlider("整数值滑动条", m_sliderIntValue, 2, 16);
```

### 2. 双滑块滑动条（**MinMaxSlider**）

关键字：EditorGUILayout.MinMaxSlider

样例：

![](https://pic1.zhimg.com/v2-dd299a4344be469cbebafc9bf01fe674_r.jpg)

示例代码：

```
private float m_leftValue;
private float m_rightValue;
...
EditorGUILayout.MinMaxSlider("双块滑动条", ref m_leftValue, ref m_rightValue, 0.25f, 10.25f);
EditorGUILayout.FloatField("滑动左值：", m_leftValue);
EditorGUILayout.FloatField("滑动右值：", m_rightValue);
```

## 四. 其他控件

### 1. 帮助框（HelpBox）

关键字： EditorGUILayout.HelpBox

样例：

![](https://pic4.zhimg.com/v2-8e5c83987ff5d536a951c2cd2134729b_r.jpg)

示例代码：

```
EditorGUILayout.HelpBox("一般提示，你应该这样做...", MessageType.Info);
EditorGUILayout.HelpBox("警告提示，你可能需要这样做...", MessageType.Warning);
EditorGUILayout.HelpBox("错误提示，你不能这样做...", MessageType.Error);
```

### 2. 间隔（Space）

关键字：EditorGUILayout.Space，GUILayout.FlexibleSpace

样例：

![](https://pic3.zhimg.com/v2-28b995912c637eefadfda255d8037d76_r.jpg)

示例代码：

```
EditorGUILayout.LabelField("----上面的Label----");
EditorGUILayout.Space(10); //进行10个单位的间隔
EditorGUILayout.LabelField("----下面的Label----");

EditorGUILayout.BeginHorizontal(); //开始水平布局
GUILayout.Button("1号button", GUILayout.Width(200)); //固定button长度： 200
GUILayout.FlexibleSpace(); // 自动填充间隔，如果窗口宽600px，那这种写法就是：【左button：200px】【自动间隔：200px】【右button ：200px】
GUILayout.Button("2号button", GUILayout.Width(200));//固定button长度： 200
EditorGUILayout.EndHorizontal(); //结束水平布局
```

### 3. GUILayout 派生控件

【EditorGUILayout 】本就是基于【GUILayout】进行的派生，用于 Editor GUI 的绘制，很多时候【[http://EditorGUILayout.xxx()](http://EditorGUILayout.xxx())】甚至完全可以用【[http://GUILayout.xxx()](http://GUILayout.xxx())】代替。

当然，这两个类都有独自的一些控件，比如只有【GUILayout.Button】，只有【EditorGUILayout.Foldout】，有时候想要的控件在【EditorGUILayout】里找不到就去【GUILayout】里面找，反之亦然。

下面简单举例一些布局操作。

```
private Vector2 scrollRoot;
...
EditorGUILayout.BeginHorizontal(); //开始水平布局
//一大堆控件...
EditorGUILayout.EndHorizontal();//结束水平布局

EditorGUILayout.BeginVertical();//开始垂直布局
//一大堆控件...
EditorGUILayout.EndVertical();//结束垂直布局

scrollRoot = EditorGUILayout.BeginScrollView(scrollRoot); //开启滚动视图
//一大堆控件...
EditorGUILayout.EndScrollView(); //结束滚动视图
```

详情可见：[李恒：Unity3d Editor 编辑器扩展功能详解（2） GUI/GUILayout](https://zhuanlan.zhihu.com/p/503964190)

## 五. 结语

终于把基础的 Editor 扩展写完了，撒花。

但是为什么别人写出来的 Editor 各种狂拽酷炫，咱写出来感觉就一坨 x 呢。

恐怕还是得在 “操作流程”“界面美化” 上下功夫，游戏 UI 是 UI，Editor 扩展 UI 也是 UI 啊，UI 的操作方法论也是需要重视的啊。

后续我专门开一篇文章好好讲讲 Editor 的界面优化，大概（