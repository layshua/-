## 前言

全系列目录索引：[Unity3d Editor 编辑器扩展功能详解（1） 目录索引](https://zhuanlan.zhihu.com/p/503154643)

上一章节：[Unity3d Editor 编辑器扩展功能详解（2） GUI/GUILayout](https://zhuanlan.zhihu.com/p/503964190)

下一章节：[Unity Editor 编辑器扩展功能详解（3）EditorGUI/EditorGUILayout（下）](https://zhuanlan.zhihu.com/p/512794841)

虽然 Editor 开发是 “工欲善其事必先利其器” 的活儿，但是感觉不如 Gameplay... 开发。

那么问题来了，GamePlay 程序员和 Editor 程序员孰好孰坏呢，有一种后者就是 Unity 码农的感觉（摊手）。

## 一、Editor 的基本面板

### 1. 浮动窗口（EditorWindow）

关键字：UnityEditor.EditorWindow，MenuItem

样例：

![](https://pic2.zhimg.com/v2-c7280b8524dac9cd9e96d7a5dd36d2dd_r.jpg)

示例代码：

```
using UnityEditor;

public class TutorialWindow : EditorWindow
{
    private static TutorialWindow window; //窗口实例对象，必须是一个static

    [MenuItem("MyWindows/TutorialWindow")] //定义菜单栏位置
    public static void OpenWindow() //打开窗口函数，必须是static
    {
        window = GetWindow<TutorialWindow>(false, "TutorialWindow", true); //实例化窗口
        window.Show(); //显示窗口
    }
    /// <summary>
    /// 窗口内显示的GUI面板
    /// </summary>
    private void OnGUI()
    {

    }
}
```

在 Unity3d 上方菜单栏，我们可以看见：

![](https://pic2.zhimg.com/v2-c2e88d4d0293a8b2eb96ce6c81db2f59_r.jpg)

点击就可以生成自定义窗口啦。

之后所有的 GUI 绘制，都是在 OnGUI() 函数中，为了预览方便，后续只会给出核心代码，实际上都是放在 OnGUI() 中。

### 2. 检视面板（Inspector）

关键字：UnityEditor.Editor， CustomEditor

样例：

![](https://pic1.zhimg.com/v2-2ed1718f014e928c2d9698951d95b894_r.jpg)

示例代码：

Mono 代码：

```
public class TutorialMono : MonoBehaviour //基本就是个空的Mono
{
}
```

Editor 代码：

```
using UnityEditor;
using UnityEngine;

[CustomEditor(typeof(TutorialMono))] //指定自定义Editor所要绑定的Mono类型，这里就是typeof(TutorialMono)
public class TutorialMonoInspector : Editor //继承Editor
{
    private TutorialMono m_target; //在Inspector上显示的实例目标
    /// <summary>
    /// 当对象活跃时（在Inspector中显示时），unity自动调用此函数
    /// </summary>
    private void OnEnable()
    {
        m_target = target as TutorialMono; //绑定target，target官方解释： The object being inspected
    }
    /// <summary>
    /// 重写OnInspectorGUI，之后所有的GUI绘制都在此方法中。
    /// </summary>
    public override void OnInspectorGUI()
    {
        base.OnInspectorGUI(); //调用父类方法绘制一次GUI，TutorialMono中原本的可序列化数据等会在这里绘制一次。
        //如果不调用父类方法，则这个Mono的Inspector全权由下面代码绘制。

        if (GUILayout.Button("这是一个按钮"))   //自定义按钮
        {
            Debug.Log("Hello world");
        }
    }
}
```

之后所有的 GUI 绘制，都是在 OnInspectorGUI() 函数中，为了预览方便，后续只会给出核心代码，实际上都是放在 OnInspectorGUI() 中。

注：下文为了方便显示，样例都是写在 EditorWindow.OnGUI() 中

## 二、按钮控件

### 1. 普通按钮（Button）

关键字： GUILayout.Button

样例：

![](https://pic3.zhimg.com/v2-bcd234d0f9421727a778eb2a471f1432_r.jpg)

示例代码：

```
if (GUILayout.Button("这是一个按钮"))
{
    Debug.Log("按了下按钮");
}
```

### 2. 按下触发按钮（DropdownButton）

关键字： EditorGUILayout.DropdownButton

样例：

![](https://pic1.zhimg.com/v2-f29e98ed3b38f4036fe973fca9ce62c4_r.jpg)

示例代码：

```
private GUIContent content = new GUIContent("DropdownButton"); //定义一下显示内容
...
if (EditorGUILayout.DropdownButton(content, FocusType.Passive))
{
    Debug.Log("据说是按下就触发，而不是抬起");
}
```

看了看官方描述也是云里雾里，反正和 Button 的区别就是：

Button 是鼠标抬起触发。

Dropdown 是鼠标按下就触发。

咱也不是很清楚，几乎没用过这个组件，粗略了解就好（逃）。

## 三、显示域（Field）

### 1. 对象域（ObjectField）

关键字：EditorGUILayout.ObjectField

样例：

![](https://pic3.zhimg.com/v2-994cb0474fc206c66870bb837d04ebfa_r.jpg)

示例代码：

```
private GameObject m_objectValue; //定义Object
...
//第一个参数： 目标Object
//第二个参数： Object的类型
//第三个参数： 是否允许赋值场景对象（一般是赋值Project中的Asset资源对象）
//最后需要一个【as】进行类型转换
m_objectValue = EditorGUILayout.ObjectField(m_objectValue, typeof(GameObject), true) as GameObject;
```

这个组件几乎是必用控件，Editor 的根本目的就是在编辑模式下，对某些资源对象（Object）进行 “增删改查”。

同时，这个 “Object” 不只是 GameObject，只要是继承 UnityEngine.Object 的类型，都可以用这个方法来获取，比如：Texture，Material 等等。

![](https://pic1.zhimg.com/v2-c719bf5cc6ce55441458ef526eba4990_r.jpg)

值得注意的是，所有的 “Field” 类，如果不把数据放在左边赋值，那么在输入框内的修改将不会保存。

（仔细想想这个代码逻辑是这样的），这个特性可以拿来显示某些 “只读” 数据。

### 2. 整数域（IntField）

关键字：EditorGUILayout.IntField

样例：

![](https://pic3.zhimg.com/v2-4c8181fb80e83a0e13763ae96c974526_r.jpg)

示例代码：

```
private int m_intValue; //定义修改内容；
...
m_intValue = EditorGUILayout.IntField("整型输入框", m_intValue); //Title + Value
```

### 3. 浮点、字符串、向量等各种域

关键字：FloatField， TextField，Vector3Field...

样例：

![](https://pic1.zhimg.com/v2-43e5ce42ac8eeb497f5c9797f425a1d4_r.jpg)

示例代码：

```
private string m_textValue;
private float m_floatValue;
private Vector2 m_vec2;
private Vector3 m_vec3;
private Vector4 m_vec4;
private Bounds m_bounds;
private BoundsInt m_boundsInt;
...
m_floatValue = EditorGUILayout.FloatField("Float 输入：", m_floatValue);
m_textValue = EditorGUILayout.TextField("Text输入：", m_textValue);
m_vec2 = EditorGUILayout.Vector2Field("Vec2输入： ", m_vec2);
m_vec3 = EditorGUILayout.Vector3Field("Vec3输入： ", m_vec3);
m_vec4 = EditorGUILayout.Vector4Field("Vec4输入： ", m_vec4);
m_bounds = EditorGUILayout.BoundsField("Bounds输入： ", m_bounds);
m_boundsInt = EditorGUILayout.BoundsIntField("Bounds输入： ", m_boundsInt);
```

### 4. 标签文本域（LabelField）

关键字：EditorGUILayout.LabelField

样例：

![](https://pic2.zhimg.com/v2-c24a6c3588f6e7c1e1d354456540aa11_r.jpg)

示例代码：

```
EditorGUILayout.LabelField("文本标题", "文本内容");
```

### 5. 标签 / 层级选择域（**TagField/LayerField**）

关键字：EditorGUILayout.TagField， EditorGUILayout.LayerField

样例：

![](https://pic3.zhimg.com/v2-aebd5d2f470ea2d7fccf81ff32e7df76_r.jpg)

示例代码：

```
private int m_layer;
private string m_tag;
...
m_layer = EditorGUILayout.LayerField("层级选择", m_layer);
m_tag = EditorGUILayout.TagField("标签选择", m_tag);
```

### 6. 颜色域（ColorField）

关键字：EditorGUILayout.ColorField

样例：

![](https://pic2.zhimg.com/v2-d18dd974ed453d03b828775345108f8d_r.jpg)

示例代码：

```
private Color m_color;
private GUIContent colorTitle = new GUIContent("颜色选择");
...
//着重强调一下这个重载方法
//第一个参数： GUIContent，通常拿来作为Title
//第二个参数： Color，目标修改数据
//第三个参数： bool ，是否显示拾色器
//第四个参数： bool ，是否显示透明度通道
//第五个参数： bool ，是否支持HDR。
m_color = EditorGUILayout.ColorField(colorTitle, m_color, true, true, true);
```

现在 URP 基本标配，该上 HDR 啦！

### 7. 动画曲线域（CurveField）

关键字：EditorGUILayout.CurveField， AnimationCurve

样例：

![](https://pic2.zhimg.com/v2-facc70992b535183c0936a214c71e911_r.jpg)

示例代码：

```
private AnimationCurve m_curve = AnimationCurve.Linear(0, 0, 1, 1);
...
m_curve = EditorGUILayout.CurveField("动画曲线：", m_curve);
```

## 四、枚举选择

### 1. 单选枚举（EnumPopup）

关键字：EditorGUILayout.EnumPopup

样例：

![](https://pic4.zhimg.com/v2-0cdc8bb1edab81300d5326f1c3176693_r.jpg)

示例代码：

```
private enum TutorialEnum
{
    One,
    Two,
    Three
}
private TutorialEnum m_enum;
...
m_enum = (TutorialEnum)EditorGUILayout.EnumPopup("枚举选择", m_enum);
```

### 2. 多选枚举（EnumFlagsField）

关键字：EditorGUILayout.EnumFlagsField

样例：

![](https://pic1.zhimg.com/v2-596c67689fd30eb064788c8db025dee0_r.jpg)

示例代码：

```
private enum TutorialEnum
{
    None = 0,
    OneAndTwo = One | Two,
    One = 1 << 0,
    Two = 1 << 1,
    Three = 1 << 2
}
private TutorialEnum m_enum;
...
m_enum = (TutorialEnum)EditorGUILayout.EnumFlagsField("枚举多选", m_enum);
```

### 3. 单选 / 多选整型（IntPopup/MaskField）

关键字： EditorGUILayout.IntPopup， EditorGUILayout.MaskField

样例：

![](https://pic4.zhimg.com/v2-72890a466a86d36f1035e6ae05d21493_r.jpg)

示例代码：

```
private int m_singleInt;
private int m_multiInt;
private string[] intSelections = new string[] { "整数10", "整数20", "整数30" };
private string[] intMultiSelections = new string[] { "1号", "2号", "3号" };
private int[] intValues = new int[] { 10, 20, 30 };
...
m_singleInt = EditorGUILayout.IntPopup("整数单选框", m_singleInt, intSelections, intValues);
EditorGUILayout.LabelField($"m_singleInt is {m_singleInt}");
m_multiInt = EditorGUILayout.MaskField("整数多选框", m_multiInt, intMultiSelections);
EditorGUILayout.LabelField($"m_multiInt is {m_multiInt}");
```

笔者对这种用得少，也不是很熟悉。

## 五、小结

不写不知道，一写吓一跳，组件好多啊，我都没注意到。

鉴于实在太多，在此分个章节。

实在等不及想看全套，也可以看看下方的 Reference

## Reference

[https://blog.csdn.net/w_mumu_q/article/details/107240127](https://blog.csdn.net/w_mumu_q/article/details/107240127)