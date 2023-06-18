## 前言：

系列目录：[Unity3d Editor 编辑器扩展功能详解（1） 目录索引](https://zhuanlan.zhihu.com/p/503154643)

上一章节：[Unity3d Editor 编辑器扩展功能详解（5）Handles](https://zhuanlan.zhihu.com/p/515544387)

下一章节：[Unity3d Editor 编辑器扩展功能详解（6）Editor 常用辅助类（下）](https://zhuanlan.zhihu.com/p/518149541)

本节主要介绍在编辑器扩展中，经常打交道的几个编辑器类的常用方法，他们几乎只会在 Editor 模式下使用，不会在 Runtime 使用。

常见的有：ScriptableObject， EditorUtility，AssetDatabase，Selection，Event 等。

## 一、 ScriptableObject

### 1. 简介与定义

这里初略介绍一下【ScriptableObject】，我们可以把他理解为一种存储于【硬盘】而不是【内存】上的 Config 资产文件。

就像一张 Excel 表一样，在一个【ScriptableObject】中配置的数据是持久化保存的。

简单案例：

```
//定义创建资产菜单，menuName : 路径名称， fileName：资产文件名
[CreateAssetMenu(menuName = "MyConfig/SimpleObj", fileName = "SimpleScriptableObejct")]
public class SimpleScriptableObject : ScriptableObject //继承自【ScriptableObject】
{
    [SerializeField] //定义序列化
    public List<InfoPair> pairs; //具体的数据字段
}

[System.Serializable] //定义序列化
public class InfoPair //一种数据类定义
{
    public int someID;
    public string someName;
    public Texture someTex;
}
```

在【Project】视图中，右键打开菜单，即可创建我们自定义的文件：

![](https://pic2.zhimg.com/v2-47d8940d352a275fe425bd0686459671_r.jpg)

![](https://pic1.zhimg.com/v2-a232c7c5bd8584c6aae7fbffa8526908_b.jpg)

![](https://pic2.zhimg.com/v2-046ae19ca9b166ab90e887bff47a8fbd_r.jpg)

### 2. 基本使用

在使用时，可以在一个 Mono 文件里面定义字段：

```
public SimpleScriptableObject scriptableObject; //定义字段

private void Start() //测试用，在Start时写逻辑
{
    if (scriptableObject != null) //非空（比如提前在Inspector中赋值）
    {
        if (scriptableObject.pairs != null && scriptableObject.pairs.Count > 0) //list非空且有数据
        {
            var name = scriptableObject.pairs[0].someName; //获取下标0的数据
            Debug.Log($"SO element name : {name}"); //打印信息
        }
    }
}
```

![](https://pic4.zhimg.com/v2-3f57df74dab045771a698409861c4e43_r.jpg)

![](https://pic2.zhimg.com/v2-1b8a42fbad25e3c695443359bf8b78c5_r.jpg)

基本上，所有的【配置数据】都可以用【ScriptableObject】保存（当然你也可以选择直接序列化为 bytes 文件，或者 json 文件等等，但是没有 unity 原生支持的 ScriptableObject 直观，而且可以直接在 Inspector 中修改数据）。

### 3. Tip

值得注意的是，我们选中一个【ScriptableObject】时，在【Inspector】是有显示 GUI 的，这意味着我们也可以自定义【ScriptableObject】的【Editor】，方法就跟自定义一个【MonoBehaviour】的【Editor】一样。

同时，手动在【ScriptableObject】面板中写入数据也不是我们想要的，更多是使用代码对其进行数据填充（比如读取一张 Excel 的数据转换成【ScriptableObject】保存形式）

### 4. 保存方式（重要）

但是！重点来了，我们在【Inspector】中的数据填充操作，编辑器是自动标识 “脏数据” 且刷新保存的，如果使用【代码填充】数据，切记使用以下方法进行保存！否则数据将不会保存！

```
//一般这段代码写在EditorWindow的OnGUI()的某个button下，实现“点击保存”功能
EditorUtility.SetDirty(simpleScriptableObject); //标记脏数据
AssetDatabase.SaveAssets(); //对所有未写入硬盘保存的脏数据信息保存
```

### 5. 使用代码创建 ScriptableObject

```
var simpleScriptableObject = ScriptableObject.CreateInstance<SimpleScriptableObject>();
//值得注意的是，这种创建方式没有创建资产实例，仅仅是在内存中创建，不会保存在硬盘上。

//如果想保存在硬盘上，需要使用如下方式
AssetDatabase.CreateAsset(simpleScriptableObject, "Assets/SomePath/simpleSO.asset");
//记得路径必须合法，也要加上文件后缀
```

## 二、 EditorUtility

### 1. 脏标记

```
EditorUtility.SetDirty(someObject); //使用例见上文
```

### 2. 提示框

关键字： EditorUtility.DisplayDialog

![](https://pic1.zhimg.com/v2-5e075c4957918ac512d863ff129c0510_r.jpg)

```
var flag = EditorUtility.DisplayDialog("标题", "显示消息", "确认键");
//当然，这方法有许多重载，请自行查看
```

### 3. 进度条

关键字：EditorUtility.DisplayProgressBar，EditorUtility.ClearProgressBar

![](https://pic1.zhimg.com/v2-b1f7975b84fac7f1ec8915a40692d184_r.jpg)

```
private float value; //定义进度条填充值
private void OnGUI() //EditorWindow.OnGUI
{
    if (GUILayout.Button("增加进度")) //手动增加进度
    {
        value += 0.1f;
        value = Mathf.Clamp01(value); //约束value值到0~1
    }
    EditorUtility.DisplayProgressBar("进度条", "显示信息", value); //显示进度条
    if (value == 1)
    {
        EditorUtility.ClearProgressBar(); //关闭进度条
    }
}
```

当然还有其他许多 API，有兴趣请自行查看。

## 三、 EditorGUIUtility

### 1. 搜索框（ObjectPicker）

关键字：EditorGUIUtility.ShowObjectPicker

![](https://pic2.zhimg.com/v2-b9ee94a674d3c0de212a3cc96ee436e1_r.jpg)

```
private Texture tex;
...
if (GUILayout.Button("查找法线贴图"))
{
    //参数释义
    //1. 查找对象的引用
    //2. 是否允许查找场景对象
    //3. 查找对象名称过滤（比如这里的normal是指文件名称中有normal的会被搜索到）
    //4. controlID, 默认写0
    EditorGUIUtility.ShowObjectPicker<Texture>(tex, false, "normal", 0);
}
```

### 2. 选中提示

关键字： EditorGUIUtility.PingObject

![](https://pic3.zhimg.com/v2-f29f7f96f344303ab68df3711eb08faa_r.jpg)

![](https://pic2.zhimg.com/v2-1436780ae8623d0fd7b560b52e1b47b5_b.jpg)

```
EditorGUIUtility.PingObject(someObj);
```

## 四、AssetDatabase

### 1. 获取资产路径

关键字：AssetDatabase.GetAssetPath

![](https://pic1.zhimg.com/v2-a180a6bc857971187b5b11efdb367488_r.jpg)

```
private Texture tex;
private string path;
...
tex = EditorGUILayout.ObjectField(tex, typeof(Texture), false) as Texture;
if (GUILayout.Button("获取文件路径"))
{
    path = AssetDatabase.GetAssetPath(tex);
}
EditorGUILayout.LabelField("资产路径是", path);
```

### 2. 加载资产

关键字： AssetDatabase.LoadAssetAtPath

![](https://pic4.zhimg.com/v2-63e86e864427376739f90a2a1df1a727_r.jpg)

```
private string texPath = "Assets/MyShaders/Brick_Normal.JPG"; //定义好路径
private Texture myTex; //定义好对象
...
EditorGUILayout.LabelField("自定义文件路径", texPath);
if (GUILayout.Button("加载资产"))
{
    myTex = AssetDatabase.LoadAssetAtPath<Texture>(texPath); //加载文件
}
EditorGUILayout.ObjectField(myTex, typeof(Texture), false);
```

### 3. 创建资产

关键字：AssetDatabase.CreateAsset

详见上文： 5. 使用代码创建 ScriptableObject

实际上，【AssetDatabase】几乎涵盖了所有关于资产文件的 CRUD（增删改查）方法，其 API 的使用方式也是顾名思义，更多的请自行查阅。

## 五、小结

这些玩意儿纯粹是【经验性质】的使用方法，基本没啥技术含量，多读读 API 文档就好了，比如：

[AssetDatabase 的方法总结_Real_JumpChen 的博客 - CSDN 博客](https://blog.csdn.net/BillCYJ/article/details/108429497)

当然，看文档自然是枯燥乏味的，想想就觉得头痛。