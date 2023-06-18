## 前言：

他喵的，我不得不吐槽一下国内的 UnityEditor 相关资料又少又散又难找，有时候想实现一个小功能他喵的得先找半小时 API，关键字（主要是英文关键字）没写对就楞是找不到，吐血了。

顺便再骂一句毒瘤 CSDN，天下文章一大抄，剩下又是各种 “转载”“索引”，然后就是 copy-paste 部分代码写点注释，真正有价值的文章简直是凤毛麟角，实属是“屎里淘金” 了。

那怎么办？只有我行我上了呗。

## 一、简介

本系列会对 Unity3d 的 Editor 使用场景下 ，比较常用的一些类、方法等进行相对详细的描述与解释，目标是完成比较全面的查阅功能。

实现诸如 “我想实现一个编辑器折叠功能”-->“查找“编辑器 -- 折叠” 关键字”-->“找到文章内容”这样的流程，是本文的夙愿。

废话少说，开坑！

## 二、重要的几个大类

### 1. GUI/GUILayout

**关键字**：GameView GUI， [MonoBehaviour.OnGUI()](https://docs.unity3d.com/ScriptReference/MonoBehaviour.OnGUI.html)

**UnityEngine.GUI** ：可以用于运行时（在 Game 视图中显示）/ 编辑器（诸如在 Inspector）面板下显示相关 UI 组件与内容，需要自行计算 Rect[[1]](https://zhuanlan.zhihu.com/p/503154643#ref_1)

**Rect**  
这个类型在编辑器拓展中十分常见，官方解释为  
A 2D Rectangle defined by X and Y position, width and height.  
一个由 X，Y 坐标，width，height 宽高定义的 2D 矩形  
其以左上角为坐标原点，X 往右递增，Y 往下递增  
更加详细介绍可参照：[Unity Rect 官方文档](https://docs.unity3d.com/ScriptReference/Rect.html)

**UnityEngine.GUILayout** ：是基于 GUI 的实现，自动进行排版，计算坐标与宽高。

![](https://pic1.zhimg.com/v2-9021e760c52bc40dd90a550679e05714_r.jpg)

详情： [Unity Editor 编辑器扩展功能详解（2） GUI/GUILayout](https://zhuanlan.zhihu.com/p/503964190)

### 2. Editor/EditorLayout

**关键字**：编辑器扩展，[Editor](https://docs.unity3d.com/ScriptReference/Editor.html)，[EditorWindow](https://docs.unity3d.com/ScriptReference/EditorWindow.html)

**UnityEditor.EditorGUI**：只可用于编辑器，需要自行计算 Rect。

**UnityEditor.EditorGUILayout**：只可用于编辑器，自动计算 Rect

![](https://pic4.zhimg.com/v2-750943db4285961a1ce3f20b6b424da3_r.jpg)

详情：[Unity Editor 编辑器扩展功能详解（3） EditorGUI/EditorGUILayout（上）](https://zhuanlan.zhihu.com/p/505159658)

[Unity Editor 编辑器扩展功能详解（3）EditorGUI/EditorGUILayout（下）](https://zhuanlan.zhihu.com/p/512794841)

### 3. Handles

**关键字**：SceneView， [3D GUI](https://docs.unity3d.com/ScriptReference/Handles.html)，[Editor.OnSceneGUI()](https://docs.unity3d.com/ScriptReference/Editor.OnSceneGUI.html), 3D Label

主要是在 Scene View 下，用于显示 / 控制物体的某些属性，便于 Debug 查看。

比如 Collider(碰撞盒) 的绿色线框就属于 Handles 的处理。

![](https://pic1.zhimg.com/v2-04764e01291bb8e3d21eaee7053eaf38_b.jpg)

详情： [Unity3d Editor 编辑器扩展功能详解（5）Handles](https://zhuanlan.zhihu.com/p/515544387)

### 4. Gizmos/Debug

**关键字**： [Gizmos](https://docs.unity3d.com/ScriptReference/Gizmos.html)，Monobehavior.OnDrawGizmos(), DrawLine，DrawRay，DrawSphere

这两者是在 Scene 视图下，进行图形绘制。

比如一个经典应用：画出 AI 敌人的视锥范围，就可以用 Gizmos.DraeRay 进行绘制，便于 Debug。

![](https://pic1.zhimg.com/v2-7a040c79d2c129ab83decad9e97e6528_r.jpg)

详情： [Unity3d Editor 编辑器扩展功能详解（4）Gizmos](https://zhuanlan.zhihu.com/p/515008289)

### 5. EditorUtility， AssetDatabase， Selection

**关键字**：EditorUtility, AssetDatabase, Selection, 序列化对象修改 / 保存，资源读取 / 保存

**EditorUtility** ： unity 内置的关于 editor 使用的一些工具类，用的最多的应该是：

```
EditorUtility.SetDirty(Object obj); //标记目标资源为“脏”
AssetDatabase.SaveAssets(); //保存项目内的“脏数据”
```

**AssetDatabase** ： 在编辑器模式下，对项目资源的管理。类似：

```
public static T LoadAssetAtPath<T>(string assetPath)whereT:UnityEngine.Object;//读取路径下的资源文件
public static string GetAssetPath(UnityEngine.Object assetObject);//获取目标资源文件的在项目中的路径
```

**Selection** : 含有当前选择的资源文件 / 文件夹的各种信息。

一个应用就是通过代码检索资源，然后选择目标资源，Inspector 就自动显示了目标资源文件信息。使用类似于

```
Selection.activeObject = obj; //obj就是目标资源文件
```

详情： [Unity3d Editor 编辑器扩展功能详解（6）Editor 常用辅助类（上）](https://zhuanlan.zhihu.com/p/517304104)

[Unity3d Editor 编辑器扩展功能详解（6）Editor 常用辅助类（下）](https://zhuanlan.zhihu.com/p/518149541)

## 三、 总目录一览

[李恒：Unity3d Editor 编辑器扩展功能详解（2） GUI/GUILayout](https://zhuanlan.zhihu.com/p/503964190)

[李恒：Unity3d Editor 编辑器扩展功能详解（3）EditorGUI/EditorGUILayout（上）](https://zhuanlan.zhihu.com/p/505159658)

[李恒：Unity3d Editor 编辑器扩展功能详解（3）EditorGUI/EditorGUILayout（下）](https://zhuanlan.zhihu.com/p/512794841)

[李恒：Unity3d Editor 编辑器扩展功能详解（4）Gizmos](https://zhuanlan.zhihu.com/p/515008289)

[李恒：Unity3d Editor 编辑器扩展功能详解（5）Handles](https://zhuanlan.zhihu.com/p/515544387)

[李恒：Unity3d Editor 编辑器扩展功能详解（6）Editor 常用辅助类（上）](https://zhuanlan.zhihu.com/p/517304104)

[李恒：Unity3d Editor 编辑器扩展功能详解（6）Editor 常用辅助类（下）](https://zhuanlan.zhihu.com/p/518149541)

[李恒：Unity3d Editor 编辑器扩展功能详解（7）（完结）Editor 美化](https://zhuanlan.zhihu.com/p/519559441)

## 四、 小结

道理我都懂，为什么不试试天下无敌的 Odin 插件呢？一个 Attribute 解决所有手写 Editor 的烦恼。

其实是笔者太菜，久仰大名但是没用过。

什么时候有机会嫖到资源 / 购买了再说吧（雾）。

## 参考

1.  [^](https://zhuanlan.zhihu.com/p/503154643#ref_1_0)1 [https://zhuanlan.zhihu.com/p/259283786](https://zhuanlan.zhihu.com/p/259283786)