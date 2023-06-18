## 前言：

全索引目录：[Unity3d Editor 编辑器扩展功能详解（1） 目录索引](https://zhuanlan.zhihu.com/p/503154643)

上一章节：[Unity Editor 编辑器扩展功能详解（3）EditorGUI/EditorGUILayout（下）](https://zhuanlan.zhihu.com/p/512794841)

下一章节：[Unity3d Editor 编辑器扩展功能详解（5）Handles](https://zhuanlan.zhihu.com/p/515544387)

想开新坑了，让我快马加鞭地把这系列坑填完吧。

## 一、 Gizmos 基础介绍

官方文档： [Important Classes - Gizmos & Handles](https://docs.unity3d.com/Manual/GizmosAndHandles.html)

Important Classes - Gizmos & Handles  
The Gizmos and Handles classes allows you to draw lines and shapes in the **Scene** view and Game view, as well as interactive handles and controls. These two classes together provide a way for you to extend what is shown in these views and build interactive tools to edit your project in any way you like. For example, rather than entering numbers in the **inspector**, you could create a draggable circle radius gizmo around a non-player character in a game, which represents the area within which they can hear or see the player.  
This page provides a simple overview of the Gizmos and Handles classes. For full documentation and an exhaustive reference of every member of the Gizmos and Handles classes, see the script reference pages for [Gizmos](https://docs.unity3d.com/ScriptReference/Gizmos.html) and [Handles](https://docs.unity3d.com/ScriptReference/Handles.html).  
重要的类： Gizmos 与 Handles  
Gizmos 与 Handles 类允许你在 “SceneView”（场景视图）与 “GameView”（游戏视图）中绘制线段与图形，或是可交互控制柄（比如 Transform 移动的三轴箭头）  
这两者一同作用，能让用户在这些视图中自定义扩展显示信息，或是构建自己喜欢的编辑、操作工具。  
比如，相比起在 Inspector（检视器）中输入数字，你可以在游戏里创建一个可拖拽的圆环附着在 NPC 上，这代表了 NPC 发现玩家的视听检测范围。  
本页面仅简单介绍了 Gizmos 与 Handles 的概览，对于详尽的文档与参考，请见：[Gizmos](https://docs.unity3d.com/ScriptReference/Gizmos.html) [Handles](https://docs.unity3d.com/ScriptReference/Handles.html)

简单地说，Gizmos 与 Handles 主要用于开发编辑中，可视化一些数据信息，便于 Debug。

## 二、 Gizmos 使用方式

关键字： MonoBehaviour.OnDrawGizmos， MonoBehaviour.OnDrawGizmosSelected

Gizmos 能且只能在 MonoBehaviour 相关子类中，使用特定的函数调用，其中：

**OnDrawGizmos()** 在每帧调佣。所有在 OnDrawGizmos 中的渲染都是可见的。

**OnDrawGizmosSelected()** 仅在脚本附加的物体被选择时调用。

示例代码：

```
public class GizmosTutotial : MonoBehaviour //新建脚本，继承自Mono
{
    private Vector3 size = new Vector3(10, 10, 10); //定义Cube大小
    private void OnDrawGizmos() //定义OnDrawGizmos方法，类似于Start，Update的定义方式 {
        Gizmos.DrawCube(Vector3.zero, size); //使用Gizmos绘制一个Cube
    } 
}
```

将此脚本挂载在场景中的 Gameobject 上，我们可以看见：

![[230b9cf4b478676cf3d767ebaccc3000_MD5.png]]

值得注意的是，场景上方工具栏的【Gizmos】开关定义了全局 “是否显示 Gizmos”，如果我们把此按钮取消激活，场景将不会绘制任何 Gizmos 图形。当然，点开小三角，能看见更详细的设置信息。

![[869ed04cd52a29e64e953a32ed7c70b5_MD5.png]]

## 三、 Gizmos 基本绘制 API

注：本文不会对所有的 API 的全部定义进行讲解，一个方法实际存在多种重载，只介绍最基本最常用的方法；若想查看方法详细情况与重载，请自行在 Visual Studio 中 “转到定义”。

### 1. 立方体（Cube）

关键字： Gizmos.DrawCube

样例：

![[c326a9a6763c84c4bfd12db2dfc8df8d_MD5.jpg]]

示例代码：

```
public class GizmosTutotial : MonoBehaviour
{
    private void OnDrawGizmos() {
        Gizmos.DrawCube(Vector3.zero, Vector3.one); //参数释义： 1.Cube中心点 2.Cube大小
    }
}
```

注：为了方便起见，下文将省略 Mono 类定义与 OnDrawGizmos 定义，请记得所有 Gizmos 方法都是在 OnDrawGizmos/OnDrawGizmosSelected 中使用。

### 2. 视锥（Frustum）

关键字：Gizmos.DrawFrustum

样例：

![[353da638ae7dc1e42065f6972480eabf_MD5.jpg]]

示例代码：

```
Gizmos.DrawFrustum(Vector3.zero, 60, 300, 0.3f, 1.7f); 
//参数释义：1. 绘制中心 2. FOV角度 3. 远裁切平面 4. 近裁切平面 5. aspect 屏幕长宽比
```

### 3. 贴图（Texture）

关键字：Gizmos.DrawGUITexture

样例：

![[9391233742dad45977fa2f6d68285e26_MD5.jpg]]

示例代码：

```
public Texture texture;
...
if (texture != null)
{
    Gizmos.DrawGUITexture(new Rect(0, 0, 10, 10), texture); //1.指定Rect 2.指定贴图
}
```

### 4. 图标（Icon）

关键字： Gizmos.DrawIcon

样例：

![[1800ab02ededdc321459bee495b37d9b_MD5.png]]

示例代码：

```
Gizmos.DrawIcon(Vector3.up, "MyIcon"); //1.绘制中心 2.图标名
```

注： 此处图标名 “MyIcon” 是指在路径：“Assets/Gizmos/...”中的图片名称。

DrawIcon 的前提是在资源文件夹 “Gizmos” 中存放想绘制的图片，定义好名称后，在代码中调用。

此处图片全路径为：“Assets/Gizmos/MyIcon.png”，所以写 “MyIcon”。

值得注意的是，所有在 Gizmos/Handles 中的绘制都不会发布到实际游戏中，只是在使用 Editor 开发时作为可视化辅助工具。

### 5. 线段（Line）

关键字： Gizmos.DrawLine

样例：

![[460b98a3ba29a85b6ab7a8fa8e6c10d5_MD5.png]]

示例代码：

```
Gizmos.DrawLine(Vector3.zero, Vector3.one);  //1.from（线段起点） 2.to（线段终点）
```

### 6. 网格（Mesh）

关键字：Gizmos.DrawMesh

样例：

![[561945fbcf6d8d26a26b6b1a7cc988d5_MD5.png]]

示例代码：

```
public Mesh mesh;
...
if (mesh != null)
{
    Gizmos.DrawMesh(mesh, 0); //1. mesh 2.submeshIndex
}
```

### 7. 射线（Ray）

关键字：Gizmos.DrawRay

样例：

![[235beaaba702c9a58866666a7f1e066e_MD5.png]]

示例代码：

```
Gizmos.DrawRay(Vector3.zero, Vector3.one); //1.from 2.direction
```

### 8. 球体（Sphere）

关键字：Gizmos.DrawSphere

样例：

![[247b28c04497b05b489e9cf139243735_MD5.jpg]]

示例代码：

```
Gizmos.DrawSphere(Vector3.up, 1); //1.center 2.radius
```

### 9. 网格线（Wire）

关键字：Gizmos.DrawWireCube

样例：

![[6b5a54c7edbceda19a193e2bc8704d0f_MD5.png]]

示例代码：

```
public Mesh mesh;
...
Gizmos.DrawWireCube(Vector3.left, Vector3.one);
Gizmos.DrawWireSphere(Vector3.right, 1);
if (mesh != null)
{
    Gizmos.DrawWireMesh(mesh, 0, Vector3.forward);
}
```

### 10. 颜色设置（Color）

关键字：Gizmos.Color

样例：

![[cb750620477aa4572ab72c03dace9c16_MD5.png]]

示例代码：

```
Gizmos.color = Color.red;
Gizmos.DrawSphere(Vector3.left, 0.5f);
Gizmos.color = Color.green;
Gizmos.DrawCube(Vector3.zero, Vector3.one);
Gizmos.color = Color.blue;
Gizmos.DrawLine(Vector3.right, Vector3.right * 2);
```

### 11. 矩阵设置（Matrix）

关键字：Gizmos.matrix

样例：

![[02f55548fd2da5c59c1c0cd89c8c9edf_MD5.png]]

示例代码：

```
Gizmos.matrix = this.transform.localToWorldMatrix; //这里使用挂载transform进行举例
Gizmos.DrawWireSphere(Vector3.zero, 1);
```

如果不对 Gizmos 的矩阵进行设置，默认为单位矩阵。

## 四、 Gizmos 经验使用

### 1. 绘制圆环（Circle）

由于 Gizmos 没有自带的圆环绘制 API，这里我们使用 DrawLine 自行计算圆环线段来实现圆环绘制。

样例：

![[022c8ae91e340dd2f539356a8be0a7ff_MD5.png]]

示例代码：

```
public Transform player; //绑定player
public Transform enemy; //绑定enemy
...
if (player != null && enemy != null)
{
    Gizmos.color = Color.green; //设置绿色
    Gizmos.DrawSphere(player.position, 0.5f); //在player处绘制球体
    Gizmos.color = Color.red; //设置红色
    Gizmos.DrawSphere(enemy.position, 0.5f); //在enemy处绘制球体
    Gizmos.color = Color.yellow; //设置黄色
    Gizmos.DrawLine(player.position, enemy.position); //绘制线段连接player enemy
    DrawCircle(enemy, 3, 0.1f, Color.red); //绘制半径3的红色圆环
    DrawCircle(enemy, 5, 0.1f, Color.green); //绘制半径5的绿色圆环
}
// theta越小，线段绘制数越多，圆环越平滑。
private void DrawCircle(Transform transform, float radius, float theta, Color color)
{
    var matrix = Gizmos.matrix; //保存原本的矩阵信息
    Gizmos.matrix = transform.localToWorldMatrix; //应用目标trans矩阵信息
    Gizmos.color = color; //设置颜色
    Vector3 beginPoint = new Vector3(radius, 0, 0); //定义起始点
    Vector3 firstPoint = new Vector3(radius, 0, 0); //定义起始点
    for (float t = 0; t < 2 * Mathf.PI; t += theta) //循环线段绘制
    {
        float x = radius * Mathf.Cos(t); //计算cos
        float z = radius * Mathf.Sin(t); //计算sin
        Vector3 endPoint = new Vector3(x, 0, z); //确定圆环采样点
        Gizmos.DrawLine(beginPoint, endPoint); //绘制线段
        beginPoint = endPoint; //迭代赋值
    }
    Gizmos.DrawLine(firstPoint, beginPoint); //绘制最后一段
    Gizmos.matrix = matrix; //还原Gizmos矩阵信息
}
```

### 2. 绘制弧线（Arc）

样例：

![[37cedb4fd956ac6e14e8bb9e95d0aacf_MD5.png]]

示例代码：

```
public Transform enemy;
...
Gizmos.color = Color.red;
Gizmos.DrawSphere(enemy.position, 0.5f);
DrawArc(enemy, 1, 90, 0.1f, Color.green);
...
private void DrawArc(Transform transform, float radius, float angle, float theta, Color color)
{
    var matrix = Gizmos.matrix;
    Gizmos.matrix = transform.localToWorldMatrix;
    Gizmos.color = color;
    Vector3 beginPoint = Vector3.zero;
    Vector3 firstPoint = Vector3.zero;
    var rad = Mathf.Deg2Rad * angle;
    for (float t = 0; t < rad; t += theta)
    {
        float x = radius * Mathf.Cos(t);
        float z = radius * Mathf.Sin(t);
        Vector3 endPoint = new Vector3(x, 0, z);
        Gizmos.DrawLine(beginPoint, endPoint);
        beginPoint = endPoint;
    }
    Gizmos.DrawLine(firstPoint, beginPoint);
    Gizmos.matrix = matrix;
}
```

值得注意的是，这种弧线的绘制算法，以单位圆为例，起点是从局部坐标（1,0,0）开始，也就是相对于 forward 的 right 方向，90 度弧线实际扫过的面积是从 right(1,0,0) 到 forward(0,0,1) 的面积。

所以这种算法不适用于 AI 视锥感知范围的绘制（90 度视觉范围应该是相对于 forward 方向的左右各 45 度）。

如何修改，那就是简单的数学题了，就待读者自行计算了。

## 五、小结

Gizmos 提供了基本的图形绘制，而场景文本（Text）绘制则是由 UnityEditor.Handles.Label 提供，我们将在后续介绍中对 Handles 进行说明。