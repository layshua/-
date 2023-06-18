## 前言：

全系列目录 [Unity3d Editor 编辑器扩展功能详解（1） 目录索引](https://zhuanlan.zhihu.com/p/503154643)

上一章节：[Unity3d Editor 编辑器扩展功能详解（4）Gizmos](https://zhuanlan.zhihu.com/p/515008289)

下一章节：[Unity3d Editor 编辑器扩展功能详解（6）Editor 常用辅助类（上）](https://zhuanlan.zhihu.com/p/517304104)

我们已经初识了 Gizmos 的 3D 图形绘制功能，这节继续介绍 Handles。

## 一、 Handles 基础介绍

[Handles](https://docs.unity3d.com/ScriptReference/Handles.html)  
Custom 3D GUI controls and drawing in the Scene view.  
Handles are the 3D controls that Unity uses to manipulate items in the Scene view. There are a number of built-in Handle GUIs, such as the familiar tools to position, scale and rotate an object via the Transform component. However, it is also possible to define your own Handle GUIs to use with custom component editors. Such GUIs can be a very useful way to edit procedurally-generated Scene content, "invisible" items and groups of related objects, such as waypoints and location markers.  
在 Sceneview（场景视图中）自定义 3D GUI 控制器与绘制的类  
Handles 是 Unity 在场景视图中，用于操控物体的 3D 控制器，已内置许多操作 GUI，比如我们熟悉的基于 Transform 对位置、缩放、旋转坐标的操作工具。当然，我们使用自定义的 Editor，定义自己的 Handle GUI 操作显示也是可能的。这种 GUIs 将会非常有用于程序化生成的场景内容、“不可见” 的子对象与组。比如路径点与坐标标记点。

一言以蔽之，许多需要在场景中程序化生成的数据，使用 Handles 类能轻松地显示控制柄来进行编辑，典型使用就是 “地编”。

## 二、 Handles 使用方式

### 1. 基于 Editor 类的 OnSceneGUI

在继承自 Editor 的类中，可以定义【OnSceneGUI()】

这样当此 Edtior 在活跃状态时（比如一个 Inspector 面板展开），在【OnSceneGUI()】方法内的内容将根据 SceneView 的刷新而调用，触发对应逻辑。

比如进行基本定义：

```
------HandlesMono .cs------
using UnityEngine;
public class HandlesMono : MonoBehaviour
{

}

------HandlesMonoEditor.cs------
using UnityEngine;
using UnityEditor;
[CustomEditor(typeof(HandlesMono))]
public class HandlesMonoEditor : Editor
{
    private void OnSceneGUI()
    {
        Debug.Log("在Editor OnSceneGUI中 调用....");
    }
}
```

在【Hierarchy】中新建空对象，挂载脚本，点击对象在【Inspector】中显示【HandlesMono】信息，就可以看见信息打印：

![[b5dd698c4179afbc91c46102d969fb36_MD5.png]]

![[feba4702e358e90d9f14246482a816e2_MD5.png]]

当然，如果在【Hierarchy】中不选择对应物体，则【HandlesMono】不在检视器中显示，意味着【HandlesMono】 Disable 掉了，将不会调用【OnSceneGUI】，读者可自行尝试。

（Tips：如果想要它一直调用，但是又需要选择其他对象怎么办呢？可以点击【Inspector】右侧的【锁】按钮，加锁之后只会显示当前的【Inspector】信息，不会随着选择而更改。）

### 2. 基于 EditorWindow 的手动注册

有些时候，我们更想在一个 Window 中进行数据编辑与操作，但是 EditorWindow 可没有 OnSceneGUI，怎么办呢？这时候，需要手动对 SceneView 的刷新事件进行注册了。

比如：

```
using UnityEngine;
using UnityEditor;
public class HandlesWindow : EditorWindow
{
    public static HandlesWindow m_mainWindow;

    [MenuItem("MyWindows/HandlesWindow")]
    public static void OpenWindow() //打开窗口
    {
        m_mainWindow = EditorWindow.GetWindow<HandlesWindow>();
        m_mainWindow.Show();
    }
    private void OnEnable() 
    {
        SceneView.duringSceneGui += OnSceneGUI; //对SceneView的刷新事件进行注册
    }
    private void OnDisable() 
    {
        SceneView.duringSceneGui -= OnSceneGUI; //对SceneView的刷新事件取消注册
    }
    private void OnSceneGUI(SceneView sceneView) //自定义刷新事件的委托方法
    {
        Debug.Log("在 Window OnSceneGUI中 调用...."); //具体逻辑
    }
}
```

![[6eef0d8aab13398992d5c8e8b5387213_MD5.jpg]]

![[75d8ed240c4130f4c71c7dcfd9674857_MD5.png]]

注：下文实例代码都是在 EditorWindow 中编写调用。

## 三、 Handles 基本绘制方法

### 1. Label（文本）

关键字： Handles.Label

样例：

![[dfa8e9f3785f7d0da3fd4b43c3989310_MD5.png]]

示例代码：

```
private Vector3 labelPos = Vector3.zero;
private string labelText = "这是在Handles中绘制的文本信息！！！";
...
//Window GUI 绘制（使用EditorGUILayout）
...
Handles.Label(labelPos, labelText); // 绘制Label
```

注：EditorWindow 的绘制只是为了显示信息方便，Editor 绘制方式见：[Unity Editor 编辑器扩展功能详解（3）](https://zhuanlan.zhihu.com/p/505159658)

### 2. Line/Dotted Line（线段 / 虚线）

关键字： Handles.DrawLine，Handles.DrawDottedLine

样例：

![[79d7db52823fd45f874ee7924ea77271_MD5.png]]

示例代码：

```
//1.from 2.to 3.thickness（厚度）决定线段厚度
Handles.DrawLine(Vector3.zero, new Vector3(-1, 1, 1), 2f);
//1.from 2.to 3.ScreenSpaceSize（屏幕空间大小）决定虚线长度
Handles.DrawDottedLine(Vector3.zero, Vector3.one, 2f);
```

### 3. Arc/Disc（弧线 / 圆环）

关键字：Handles.DrawWireArc，Handles.DrawSolidDisc

样例：

![[e7912229603646508f07c34b76b1e48e_MD5.png]]

示例代码：

```
Handles.color = new Color(1, 0, 0, 0.3f); //提前使用Color进行颜色设置，便于观察
Handles.DrawWireArc(Vector3.zero, Vector3.up, Vector3.right, 90, 2); //绘制线框弧线
Handles.DrawSolidArc(Vector3.zero, Vector3.up, Vector3.back, 90, 2); // 绘制填充弧线

Handles.color = new Color(0, 1, 0, 0.3f);
Handles.DrawSolidDisc(new Vector3(0, 0, 5), Vector3.up, 5); //绘制填充圆环
Handles.DrawWireDisc(new Vector3(0, 0, -5), Vector3.up, 5); //绘制线框圆环
```

### 4. Rectangle（矩形）

关键字：Handles.DrawSolidRectangleWithOutline

样例：

![[b150ddea6ad1df033b57c2f860a6cfb1_MD5.png]]

示例代码：

```
Handles.DrawSolidRectangleWithOutline(new Rect(0, 0, 1, 1), Color.green, Color.red);
```

### 5. GUI

关键字：Handles.BeginGUI()，Handles.EndGUI()

样例：

![[f0b5c2458b3aa9b8d6926be1a06c05f2_MD5.png]]

示例代码：

```
Handles.BeginGUI();
GUILayout.Label("我是SceneView中的Label");
if (GUILayout.Button("我是SceneView中的Button"))
{
    Debug.Log("芜湖");
}
Handles.EndGUI();
```

## 四、 控制柄

实际上，Handles 核心功能是他的 Handle（控制）类，比如：Handles.PositionHandle 就是一个类似于 Position 移动的方法。经典使用例就是在一个 List<Vector3> 列表中，选择某个列表元素显示他的 Position 控制柄，然后进行移动。

又或者类似这种：

![[0cddc5ddb45f31146759d889fa0cd39e_MD5.png]]

```
private Vector3 root = Vector3.zero;
...
Handles.DrawSolidDisc(root, Vector3.up, 1); //画个圆，假设这是目标对象
root = Handles.PositionHandle(root, Quaternion.identity); //控制柄
```

但是笔者现阶段使用的较少，不是很熟悉，就不作解释了。

详情可见：[Unity 的 Handles 类_林新发的博客 - CSDN 博客_unity 里 handle](https://linxinfa.blog.csdn.net/article/details/89334603)

或者看看[官方 API 文档](https://docs.unity3d.com/ScriptReference/Handles.html)。

或者是 [Unity 编辑器扩展五 Handles 扩展 Scene 视图](https://www.jianshu.com/p/efd33e1a7e15)

笔者摸了。_(:з」∠)_

## 五、 小结

实际上，OnGUIScene + Handles 还有一种应用就是 “地形笔刷”，就像 Terrain 笔刷那样在 SceneView 里面刷刷刷调整高度、生成植被，这种功能我们也是能自己写一套的，核心是编辑模式下的 Event 事件。

具体的可见：[Unity - Scripting API: Event](https://docs.unity3d.com/ScriptReference/Event.html)

有空再写，大概（