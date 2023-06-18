## 前言：

系列目录：[Unity3d Editor 编辑器扩展功能详解（1） 目录索引](https://zhuanlan.zhihu.com/p/503154643)

上一章节：[Unity3d Editor 编辑器扩展功能详解（6）Editor 常用辅助类（下）](https://zhuanlan.zhihu.com/p/518149541)

本节介绍一些常用的编辑器面板美化方法，牵涉【GUIStyle、GUISkin】，你可以理解为这两者是 Unity 内置的 GUI 绘制约束与模板。

## 一、字体格式 / 颜色

关键字：GUIStyle

样例：

![](https://pic4.zhimg.com/v2-3ad8b94f2bf7ef95f0939c27a1b689d7_b.jpg)

示例代码：

```
using UnityEditor;
using UnityEngine;

public class PolishWindow : EditorWindow
{
    public static PolishWindow m_mainWindow; //主要窗口对象

    private GUIStyle defaultStyle = new GUIStyle(); //定义 GUIStyle

    [MenuItem("MyWindows/PolishWindow")] //定义窗口菜单栏
    public static void OpenWindow()
    {
        m_mainWindow = EditorWindow.GetWindow<PolishWindow>(); //创建窗口
        m_mainWindow.Show(); //打开窗口
    }
    private void OnGUI()
    {
        defaultStyle.fontSize = 10; //字体大小： 10
        defaultStyle.fontStyle = FontStyle.Normal; //字体样式：normal

        defaultStyle.alignment = TextAnchor.MiddleCenter; //字体对齐方式： 居中对齐
        defaultStyle.normal.textColor = Color.red; //字体颜色： 红色
        EditorGUILayout.LabelField("这是红色字体", defaultStyle); //绘制GUI组件，使用 defaultStyle

        defaultStyle.alignment = TextAnchor.MiddleLeft; //字体对齐方式： 水平靠左，垂直居中
        defaultStyle.normal.textColor = Color.yellow; //字体颜色：黄色
        defaultStyle.fontSize = 20; //字体大小： 20
        EditorGUILayout.LabelField("这是黄色字体", defaultStyle); //绘制GUI组件

        defaultStyle.normal.textColor = Color.green; //字体颜色： 绿色
        defaultStyle.fontSize = 12; //字体大小 ： 12
        defaultStyle.fontStyle = FontStyle.Bold;  //字体样式： Bold（加粗）
        EditorGUILayout.SelectableLabel("这是绿色字体", defaultStyle); //绘制GUI
    }
}
```

注：下文代码将只给出关键代码，实际使用位置如上所示在 OnGUI() 方法中。

## 二、GUI 颜色

关键字：GUI.color

样例：

![](https://pic2.zhimg.com/v2-1da387273ef4fac3547673855e343e6d_b.jpg)

示例代码：

```
GUI.color = Color.red;
GUILayout.Button("红色Button");
GUI.color = Color.green;
EditorGUILayout.LabelField("绿色Label");
GUI.color = Color.yellow;
EditorGUILayout.IntField("黄色Int Field", 20);
```

## 三、面板分区 / 搜索框

关键字：

样例：

![](https://pic2.zhimg.com/v2-cc3253b983647ef4a4148291587cc2a5_r.jpg)

示例代码：

```
EditorGUILayout.BeginVertical("box");
EditorGUILayout.LabelField("这是【box】样式，用于显示面板的分区");
EditorGUILayout.Vector3Field("比如这是一个Vec3 Field", Vector3.one);
EditorGUILayout.BeginToggleGroup("比如这是一个Toggle Group", false);
EditorGUILayout.HelpBox("比如这是一个HelpBox", MessageType.Warning);
EditorGUILayout.TextArea("比如这是一个TextArea");
EditorGUILayout.EndToggleGroup();
EditorGUILayout.EndVertical();

EditorGUILayout.Space(10);

EditorGUILayout.BeginVertical("frameBox");
EditorGUILayout.LabelField("这是【frameBox】样式，用于显示面板的分区");
EditorGUILayout.BoundsField("比如这是一个BoundField", new Bounds());
EditorGUILayout.Slider("比如这是一个Slider", 5, 0, 10);
EditorGUILayout.TextArea("这是一个【SearchTextField】样式的文本框", "SearchTextField");
EditorGUILayout.EndVertical();
```

GUIStyle、GUISkin 的许多内置样式咱也不是很清楚全貌，只介绍了自己常用的几个，有额外需求的可以自行搜索看看。

## 四、案例实战

好了，你已经学会 1+1=2 了，下面我们来算一个微积分吧。

样例：

![](https://pic3.zhimg.com/v2-1313c69b89e8fbf63faeab490dc9dba2_r.jpg)

示例代码：

```
using UnityEditor;
using UnityEngine;

public class PolishWindow : EditorWindow
{
    public static PolishWindow m_mainWindow; //主要窗口对象

    private const int arrayLength = 35; //定义元素数组长度
    private const int countPerPage = 10; //定义每一页的数量

    private GUILayoutOption maxWidth = GUILayout.MaxWidth(120); //定义一个GUI组件的最大宽度
    private Vector2 scrollRoot; //定义组件ScrollView的滚动Root
    private int page = 1; //当前page
    private int currentSelectionIndex = -1; //当前选择index
    private int totalPage; //页面总数
    private int[] someValues; //显示的元素
    private bool foldOut; //折叠flag

    [MenuItem("MyWindows/PolishWindow")] //定义窗口菜单栏
    public static void OpenWindow()
    {
        m_mainWindow = EditorWindow.GetWindow<PolishWindow>(); //创建窗口
        m_mainWindow.Show(); //打开窗口
    }
    /// <summary>
    /// 当窗口打开时调用
    /// </summary>
    private void OnEnable()
    {
        someValues = new int[arrayLength]; //实例化数组
        totalPage = (int)Mathf.Ceil((float)arrayLength / (float)countPerPage); //计算总页数
        for (int i = 0; i < someValues.Length; i++)
        {
            someValues[i] = Random.Range(0, 100); //进行随机数据填充
        }
    }
    /// <summary>
    /// Editor面板绘制方法
    /// </summary>
    private void OnGUI()
    {
        scrollRoot = EditorGUILayout.BeginScrollView(scrollRoot); //开启 滚动视图 
        foldOut = EditorGUILayout.BeginFoldoutHeaderGroup(foldOut, "假装这是一个检索页面"); //开启 折叠组

        if (foldOut) //如果折叠打开，则进行绘制
        {
            EditorGUILayout.BeginVertical("frameBox"); //开始垂直布局，使用GUIStyle ： frameBox
            for (int i = (page - 1) * countPerPage; i < someValues.Length; i++) //循环绘制，page 默认是1开始，这里【page-1】对齐数组下标。
            {
                if (i >= page * countPerPage) //一页只能绘制【countPerPage】个元素，但是arrayLength可能不止，在这里截断。
                    break;

                if (currentSelectionIndex == i) //判断当前index与选择index
                    GUI.color = Color.green; //为真，突出显示选择项
                else
                    GUI.color = Color.white; //为假，还原默认颜色

                EditorGUILayout.BeginHorizontal("box"); //开始水平布局
                if (GUILayout.Button("Select This One", maxWidth)) //绘制选择按钮，使用maxWidth限制组件宽度
                {
                    currentSelectionIndex = i; //记录选择Index
                }
                EditorGUILayout.LabelField($"这是【{i}】号，他的随机值是：{someValues[i]}"); //显示数据信息
                EditorGUILayout.EndHorizontal(); //结束水平布局
            }

            EditorGUILayout.BeginHorizontal(); //开始水平布局
            EditorGUILayout.LabelField($"当前页数：{page.ToString()} / 总页数：{totalPage}"); //绘制页数信息
            if (GUILayout.Button("上一页")) //绘制上一页Button
            {
                page -= 1; //当前页面-1
                page = Mathf.Clamp(page, 1, totalPage); //由于页面数取值是1~totalPage，这里进行一次取值范围约束
            }
            if (GUILayout.Button("下一页")) //绘制下一页Button
            {
                page += 1; //当前页面+1
                page = Mathf.Clamp(page, 1, totalPage); //由于页面数取值是1~totalPage，这里进行一次取值范围约束
            }
            EditorGUILayout.EndHorizontal(); //结束水平布局

            EditorGUILayout.EndVertical(); //结束垂直布局
        }

        EditorGUILayout.EndFoldoutHeaderGroup(); //关闭 折叠组
        EditorGUILayout.EndScrollView(); //关闭 滚动视图
    }
}
```

## 小结：

实际上，但我们足够熟悉 Editor 编辑器这一套时，我们完全可以遵循 OOP 思想，进行编辑器组件的封装继承。

比如定一个总的 EditorWindow 类，定义一个抽象基类，然后在总窗口使用反射进行类型检索，对所有继承于抽象基类的进行汇总，在面板上显示对应的按钮，点哪个就实例化哪个窗口进行显示。

这样在写一个新窗口时，只需要继承于抽象基类，定义基本的逻辑与绘制就可以了，总窗口会自动地把新窗口归纳整理到自己的菜单栏中。

当然，编辑器的奇技淫巧还有许多，全靠经验积累。

终于写完了，完结撒花呜呜呜。