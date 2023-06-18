## 前言：

系列目录：[Unity3d Editor 编辑器扩展功能详解（1） 目录索引](https://zhuanlan.zhihu.com/p/503154643)

上一章节：[Unity3d Editor 编辑器扩展功能详解（6）Editor 常用辅助类（上）](https://zhuanlan.zhihu.com/p/517304104)

下一章节：[Unity3d Editor 编辑器扩展功能详解（7）（完结）Editor 美化](https://zhuanlan.zhihu.com/p/519559441)

本节查漏补缺，简单介绍一些其他辅助类。

## 一、 Selection

Selection 主要是用于返回在编辑器中的选择对象（Object）信息，这个对象（Object）可以是一份资产文件（Asset、Prefab），也可是场景中的实例对象（Gameobject）。

一个经典应用就是，比如在 EditorWindow 中实例化了一个对象，出于操作友好性，应该把当前选择对象直接变为生成实例，这样在【Inspector】面板中就能显示对应信息了，这种操作就是通过 Selection 类实现的。

（注：Selection 在命名空间 UnityEditor 中定义）

### 1. Object Selection(对象选择)

```
Selection.activeGameObejct //返回当前点击的场景游戏物体或Project Prefab;选择多个则返回第一个选择；未选择则返回null
Selection.activeTransform //返回当前点击的场景游戏物体transform。
Selection.activeObject // 返回当前点击的场景游戏物体或Project资产文件。

Selection.gameObjects //返回一个数组，内容为当前点击场景物体或Project Prefab，非GameObject的选择不会加入数组。未选择返回长度为0的数组
Selection.objects //返回一个数组，内容为当前点击的场景物体或Project资源。
Selection.transforms //返回一个数组，内容为当前点击的场景物体。

Selection.selectionChanged //委托，选择变化时调用。
```

### 2. Static Methods(静态方法)

```
// Contains 选择项中是否包含物体
public static bool Selection.Contains(int instanceID) 
public static bool Selection.Contains(Object obj)

// GetFiltered :返回按类型与模式过滤当前选择
// T : 过滤的泛型，只有某一种类型的对象才会被选择
// SelectionMode ： 选择的模式，下文细讲
public static T[] GetFiltered<T>(SelectionMode mode);

// GetTransforms : 允许使用SelectionMode对选择类型进行颗粒度控制
public static Transform[] GetTransforms(SelectionMode mode);
```

### 3. SelectionMode 详解

```
Unfiltered 返回整个选择（无模式过滤）
TopLevel 只返回最上层transform， 子物体的transform将被过滤掉
Deep 返回选择对象与他所有的子对象
ExcludePrefab 排除选择中的prefab对象
Editable 排除任何无法被修改的对象（只选择可以编辑的对象）
Assets 返回Asset资源
DeepAssets 返回整个文件夹内容（根目录与子文件夹）
```

### 4. 示例案例

**目的：获取某个文件夹下所有的 Texture 资产**

![](https://pic1.zhimg.com/v2-fbfe5903b48bf6e64f4224ef5e392d88_r.jpg)

![](https://pic2.zhimg.com/v2-d44098522059663abe95f899cfcd8999_r.jpg)

示例代码：

```
private void OnGUI() //EditorWindow的OnGUI方法
{
    if (GUILayout.Button("选择一个文件夹，获取下面所有的Texture资源文件"))
    {
        var texs = Selection.GetFiltered<Texture>(SelectionMode.DeepAssets); //进行深度遍历所有文件夹
        Debug.Log($"Tex总计数：{texs.Length}");
        foreach (var tex in texs)
        {
            //使用AssetDatabase获取一次资产路径
            Debug.Log($"Tex 文件名：{tex.name}，文件路径 ：{AssetDatabase.GetAssetPath(tex)}"); 
        }
    }
}
```

**目的：新建一个 GameObject，同时选中**

![](https://pic2.zhimg.com/v2-e998a5b76515141d013949c680803291_r.jpg)

示例代码：

```
if (GUILayout.Button("创建一个 Cube，同时选中"))
{
    var obj = GameObject.CreatePrimitive(PrimitiveType.Cube); //创建Cube
    obj.name = "新建的Cube";
    Selection.activeGameObject = obj; //选中Cube
}
```

## 二、 Event

A UnityGUI event.  
Events correspond to user input (key presses, mouse actions), or are UnityGUI layout or rendering events.  
For each event [OnGUI](https://docs.unity3d.com/ScriptReference/MonoBehaviour.OnGUI.html) is called in the scripts; so OnGUI is potentially called multiple times per frame. [Event.current](https://docs.unity3d.com/ScriptReference/Event-current.html) corresponds to "current" event inside OnGUI call.

Event 主要用于编辑器模式下，对 IMGUI 操作输入的响应。

在 Editor 模式（而不是 Playing 模式）下，写在代码中的类似【Input.GetKeyDown】是无效的，是无法获取键盘 / 鼠标操作信息的，取而代之的是使用类似【Event.current.type == EventType.MouseDown】检测输入。

它的使用可以在 Editor 的面板下，也可以在场景视图中（SceneView）。

### 1. 简单输入

![](https://pic2.zhimg.com/v2-7a882c9dc9981b7f7dc655a1e72fca69_r.jpg)

示例代码：

```
public class EventTutorialWindow : EditorWindow
{
    public static EventTutorialWindow m_mainWindow;

    [MenuItem("MyWindows/EventWindow")]
    public static void OpenWindow()
    {
        m_mainWindow = EditorWindow.GetWindow<EventTutorialWindow>();
        m_mainWindow.Show();
    }
    /// <summary>
    /// 窗口激活时调用
    /// </summary>
    private void OnEnable()
    {
        SceneView.duringSceneGui += OnSceneView; //注册场景视图刷新回调
    }
    /// <summary>
    /// 窗口关闭时调用
    /// </summary>
    private void OnDisable()
    {
        SceneView.duringSceneGui -= OnSceneView; //移除场景视图刷新回调
    }
    /// <summary>
    /// SceneView刷新回调
    /// </summary>
    /// <param name="sceneView"></param>
    private void OnSceneView(SceneView sceneView)
    {
        if (Event.current.type == EventType.MouseDown && Event.current.button == 0)
        {
            Debug.Log("鼠标左键在场景中Clicked....");
        }
    }
    /// <summary>
    /// 面板绘制
    /// </summary>
    private void OnGUI()
    {
        if (Event.current.type == EventType.KeyDown && Event.current.keyCode == KeyCode.A)
        {
            Debug.Log("键盘A键在Window面板中按下....");
        }
    }
}
```

### 2. 组合键

![](https://pic2.zhimg.com/v2-3824661cef09802870cbfcc7da77577d_r.jpg)

示例代码：

```
...//在上文OnSceneView（）中加入以下代码即可
if (Event.current.type == EventType.MouseDown && Event.current.button == 0)
{
    if (Event.current.alt) //检测alt键是否被按下
    {
        Debug.Log("在场景视图中按下了Alt + 鼠标左键");
    }
    else if (Event.current.shift) //检测shift键是否被按下
    {
        Debug.Log("在场景视图中按下了Shift + 鼠标左键");
    }
}
```

### 3. 坐标转换

一个很常见的需求就是，我们在 SceneView 中点击了，希望在点击处发射一条射线检测场景中的物体，求一个交点然后放置什么东西（就像 Terrain 的笔刷一样），这是通过以下方式实现的。

```
private LayerMask layerMask = -1; //设置为-1，包含所有Layer
...
private void OnSceneView(SceneView sceneView)
{
    if (Event.current.type == EventType.MouseDown && Event.current.button == 0) //检测鼠标点击
    {
        var mousePos = Event.current.mousePosition; //获取鼠标在SceneView中的屏幕位置
        var ray = HandleUtility.GUIPointToWorldRay(mousePos); //生成屏幕坐标-->世界坐标的射线
        RaycastHit hit; //定义射线检测结果结构体
        if (Physics.Raycast(ray, out hit, float.MaxValue, layerMask)) //进行射线检测，layerMask是检测层级
        {
            var obj = GameObject.CreatePrimitive(PrimitiveType.Sphere); //如果检测成功，创建一个球体。
            obj.transform.position = hit.point; //把球体坐标设置为hit交点处。
        }
    }
}
```

样例：

![](https://pic1.zhimg.com/v2-85d93bac6c9aa715d205d66861945354_r.jpg)

### 4. Tip

查阅资料的时候发现有这样一种使用方法：

```
Event.current.Use();
```

文档解释：[Unity - Scripting API: Event.Use](https://docs.unity3d.com/ScriptReference/Event.Use.html)

文档上说，在某些 Event 检测后，使用 Event.current.Use(); 就能让其他 GUI elements 忽视这次 Event。

比如说使用 Alt + 鼠标左键创建物体，但是这个组合键也是 unity 场景自带的组合键，用于旋转 SceneView 的，使用

Event.current.Use(); 就会让自带的旋转功能失效。

主要是为了解决组合键冲突吧。

## Reference:

[1] [Unity - Scripting API: Selection](https://docs.unity3d.com/ScriptReference/Selection.html)

[2] [Unity - Scripting API: Event](https://docs.unity3d.com/ScriptReference/Event.html)

[3] [UnityEditor 的 Selection 类_庸人自扰 Eam 的博客 - CSDN 博客](https://blog.csdn.net/qq_33337811/article/details/72858209)