
---
title: Unity3D
aliases: []
tags: []
create_time: 2023-06-02 22:33
uid: 202306022233
banner: "![[Pasted image 20230602223746.png]]"
---
# 零、工作原理
## 反射机制

> [!NOTE] 反射
> 1. 程序正在运行时，可以查看其它程序集或者自身的元数据。一个运行的程序查看本身或者其它程序的元数据的行为就叫做反射
> 2. 在程序运行时，通过反射可以得到其它程序集或者自己程序集中代码的各种信息，比如类，函数，变量，对象等等我们可以实例化它们，执行它们，操作它们

**Unity 开发的本质就是在 Unity 引擎的基础上，利用反射和引擎提供的各种功能进行的拓展开发。**

**场景中对象的本质是什么？**
GameObject 类对象是 Unity 引擎提供给我们的，作为场景中所有对象的根本。
在游戏场景中出现一个对象，不管是图片、模型、音效、摄像机等等都是依附于 GameObject 对象。
拟人化记忆: GameObject 就是没有剧本的演员。

除了 **Transform** 这个表示位置的**标配剧本**外，我们可以为这个演员 (GameObject）关联各种剧本（脚本 )，让它按照我们剧本中 (代码逻辑中)的命令来处理事情
而为演员添加剧本的这个过程，就是在利用反射 new 一个新的剧本对象和演员 (GameObject)对象进行关联，让其按我们的命令做事。
![[Pasted image 20230603122307.png]]


Unity 场景文件（. unity）它的本质就是一个配置文件
Unity 有一套自己识别处理它的机制，本质就是把场景对象相关信息读取出来，通过反射来创建各个对象关联各个脚本对象
![[Pasted image 20230603123624.png|650]]

## 预制体（Prefab）和资源包导入导出
预制体和资源包用于保存数据，方便数据管理

# 一、脚本基础
## 创建规则
 1. 不在 VS 中创建脚本了
 2. 可以放在 Assets 文件夹下的任何位置（建议同一文件夹管理)
 3. 类名和文件名必须一致,不然不能挂载 (因为反射机制创建对象，会通过文件名去找 Type)
 4. 建议不要使用中文名命名
 5. 没有特殊需求不用管命名空间
 6. 创建的脚本默认继承 MonoBehavior

默认脚本内容路径：`Editor\DataResources\ScriptTemplates`

7. 脚本之间的关系：
![[Pasted image 20230604101859.png|700]]

![[Pasted image 20230604101922.png]]
## Inspector 窗口
### 可编辑的变量

> [!NOTE] 
> 1. Inspector 窗口中的变量关联的就是对象的成员变量，运行时改变他们就是在改变成员变量 
>2. 拖拽到 Gameobject 对象后，再改变脚本代码中变量默认值，界面上不会改变 
>3. 运行中修改的信息不会保存

 1. Inspector 显示的可编辑内容就是脚本的成员变量

2. **public 成员变量可直接显示编辑**
   加上特性 `[HideInInspector]` 后不可显示编辑
```cs
[HideInInspector]  
public int i;
```

3. **private 和 protected 成员变量无法显示和编辑**
   加上**强制序列化字段特性 `[serializeField]`** 后可以编辑。所谓序列化就是把一个对象保存到一个文件或数据库字段中去。
```cs
[SerializeField]
private int z;
```

4. 大部分类型都能显示编辑，**不支持字典 Dictionary 和自定义类类型变量。**
加上序列化 `[Serializable]` 特性后可以显示自定义类类型
```cs
[Serializable]
public class Person
{
    public int age;
    public string name;
}
```

### 窗口排版
1. 分组说明特性 Header：为成员分组
`[Header ("分组说明")]`

```cs
 [Header("基础属性")] 
public int age;
public string name;

[Header("进阶属性")]
public float height;
public float weight;
```
![[Pasted image 20230603143748.png]]

2. 鼠标悬停注释 Tooltip ：为变量添加说明
`Tooltip (“说明内容")]

 3. 间隔特性 Space：让两个字段间出现间隔
`[Space ()]`

4. 修饰数值的滑条范围 Range
   `[Range (最小值, 最大值)]`

5. 多行显示字符串，默认不写参数显示 3 行，写参数就是对应行
`[Multiline (行数)]`

6. 滚动条显示多行 字符串，默认不写参数就是超过 3 行显示滚动条
`[TextArea (3，4)]`：最少显示 3 行，最多 4 行，超过 4 行就显示滚动条

7. 为变量添加快捷方法 contextMenuItem 
   - 参数 1 显示按钮名
   - 参数 2 方法名不能有参数
   `[contextMenuItem ("显示按钮名"，“方法名")]`
```cs
[ContextMenuItem("重置钱","ResetMoney")]
public int money;

private void ResetMoney()
{
    money = 0;
}
```
 右键可以查看方法：  ![[Pasted image 20230603144613.png]]

8. 为方法添加特性能够在 Inspector 中执行 ContextMenu
   `[ContextMenu ("测试函数")]`
```cs
[ContextMenu("哈哈哈哈")]
private void TestFun()
{
   print("哈哈哈哈");
}
```
在脚本上可以调用该方法：
![[Pasted image 20230603144922.png]]

## 生命周期函数
游戏的本质就是一个死循环，每一次循环处理游戏逻辑就会更新一次画面，一帧就是执行一次循环。
Unity 底层已红帮助我们做好了死循环，我们需要学习 Unity 的生命周期函数，利用它做好的规则来执行我们的游戏逻辑就行了。

> [!NOTE] 生命周期函数的概念
> - 所有继承 MonoBehavior 的脚本最终都会挂载到 Gameobject 游戏对象上
> - 生命周期函数就是该脚本对象依附的 Gameobject 对象从出生到消亡整个生命周期中会**通过反射自动调用的一些特殊函数**
>- Unity 帮助我们记录了一个 Gameobject 对象依附了哪些脚本，会自动的得到这些对象，通过反射去执行生命周期函数
>- 生命周期函数并不是 MonoBehavio 基类中的成员，Unity 帮助我们记录了场景上的所有 GameObjgct 对象以及各个关联的脚本对象，在游戏执行的特定时机 (对象创建时，失活激活时，帧更新时)它会通过函数名反射得到脚本对象中对应的生命周期函数，然后再这些特定时机执行他们

- 生命周期函数的访问修饰符一般为 private 和 protected（默认为private）
- 因为不需要再外部自己调用生命周期函数都是 Unity 自己帮助我们调用的
- 支持继承多态

>常用的生命周期函数：
![[Pasted image 20230603132450.png]] 
```cs
//当对象(自己找个类对象)被创建时，才会调用该生命周期函数
//类似构造函数，一个对象只会调用一次
void Awake()

//依附的GameObject对象每次激活时调用（打勾）
//想要当一个对象被激活时进行一些逻辑处理,就可以写在这个函数
void OnEnable()

// 对象Awake后，第一次帧更新之前调用，一个对象只会调用一次
void Start()

//进行物理帧更新  
//固定间隔执行，间隔时间可以设置  
void FixedUpdate()

// 逻辑帧执行，每帧执行
// 处理游戏核心逻辑更新
void Update()

// 每帧执行，于Update之后执行（速度相同）
//一般用来处理摄像机位置更新相关内容的
//Update和LateUpdate之间，Unity会处理动画相关的更新，如果将摄像机放在Update中更新，可能会造成渲染上的问题
void LateUpdate()

//依附的GameObject对象每次失活时调用（去掉勾）
//想要当一个对象失活时进行一些逻辑处理,就可以写在这个函数
void OnDisable()

//当对象被销毁时调用(衣服的GameObject对象被删除时调用)
//一般用来做一些资源的释放
void OnDestroy()
```

**激活对象**：![[Pasted image 20230603133828.png]]
`
**设置固定时间步长：**
![[Pasted image 20230603134654.png]]

## MonoBehavior 基类
1. 创建的脚本默认都**继承 MonoBehaviour**，继承了它才能够挂载在 GameObject 
>当我们把脚本拖到 GameObject 上时，引擎会根据文件名通过反射得到对应的类，如果该类继承了 MonoBehaviour，则允许挂载。
2. 继承了 MonoBehavior 的脚本不能 new ，**只能挂载**！
3. 继承了 MonnBehavior 的脚本不要去写构造函数，因为我们不会去 new 它，写构造函数没有任何意义
4. 继承了 MonoBehavior 的脚本可以在一个对象上挂多个 (如果没有加 DisallowMultipleComponent 特性)
   ![[Pasted image 20230603130352.png]]
5. 继承 MonoBehavior 的类也可以再次被继承，遵循面向对象继承多态的规则

**不继承 MonoBehaviour 的类：**
1. 不能挂载在 GameObject 上
2. 想怎么写怎么写，如果要使用需要自己 new
3. **一般是单例模式的类（用于管理模块）或者数据结构类（ 用于存储数据）**
4. 不用保留默认出现的几个函数


> [!info] this
> this 代表脚本对象
> this. component 代表脚本挂载的 GameObject 
> this. transform 代表脚本挂载的 GameObject 的位置信息
> 等价写法：this.component.transform

### 打印
在 Unity 中打印信息的两种方式
```cs
//1.没有继承MonoBehaviour的类的时候，可以使用Debug.Log
Debug.Log("Awake Hello!");
Debug.LogError("Awake Error");
Debug.LogWarning("Awake Warning");  
//2. 继承了MonoBehaviour的类，可以使用线程方法print
print("Awake Hello!");
```
### 重要成员
1. 获取依附的 Gameobject
2. 获取依附的 Gameobject 的位置信息
3. 获取脚本是否激活
```cs
public TestScript testScript;  //其他脚本
 
void Start()
{
    //1. 获取依附的GameObject
    print(this.gameObject.name);
    
    //2. 获取依附的GameObject的位置信息
    //得到对象位置信息
    print(this.transform.position);  //位置
    print(this.transform.eulerAngles);  //角度
    print(this.transform.lossyScale);  //缩放大小
    //等价写法：this.gameObject.transform
    
    //3. 获取脚本是否激活
    this.enabled = true;   //激活脚本
    this.enabled = false;  //禁用脚本
    
    //获取别的脚本对象依附的gameobject和transfrom位置信息
    print(testScript.gameObject.name);
    print(testScript.transform.position);
}
```

### 重要方法
如何得到依附的 GameObject 对象上挂载的其它脚本?
1. 得到 GameObject 挂载的单个脚本
```cs file:得到自己挂载的单个脚本
//根据脚本名获取，较少使用
TestScript t1 = this.GetComponent("TestScript") as TestScript; 

//根据Type获取
TestScript t2 = this.GetComponent(typeof(TestScript)) as TestScript;

//⭐根据泛型获取，建议使用，不用as
TestScript t3 = this.GetComponent<TestScript>();

//只要你能得到场景中对象或者对象依附的脚本，那你就可以获取到它所有信息
```

安全的获取脚本，加一个判断：
```cs
//方法一：
MyScript s1 = this.GetComponent<MyScript>();
if (s1 != null)
{
    //do something
}

//方法二：
MyScript s2;
if(this.TryGetComponent<MyScript>(out s2))
{
    //do something
}
```

2. 得到 GameObject 挂载的多个脚本 (不常用，通常我们不会将同一个脚本挂载两次在同一个 GameObject 上)
```cs
//方法一
MyScript[] scripts = this.GetComponents<MyScript>();

//方法二
List<MyScript> scriptList = new List<MyScript>(); //定义一个存放MyScript类型的List
this.GetComponents<MyScript>(scriptList); //将找到的结果存在List中
```

3. 得到 GameObject 子孙对象挂载的脚本（默认会找本 GameObject 对象是否挂载该脚本）
```cs
//得到子孙对象挂载的单个脚本：
MyScript s1 = this.GetComponentInChildren<MyScript>(); //如果脚本失活，则无法找到
MyScript s2 = this.GetComponentInChildren<MyScript>(true); //true表示即使脚本失活，也可以找到

//得到子孙对象挂载的多个脚本：
//方法一：
MyScript[] ss1 =  this.GetComponentsInChildren<MyScript>(true); 
//方法二：
List<MyScript> ss2 = new List<MyScript>();
this.GetComponentsInChildren<MyScript>(true, ss2);
```

4. 得到 GameObject 长辈（包括父，爷爷...）对象挂载的脚本（默认会找本 GameObject 对象是否挂载该脚本）
```cs
//得到单个脚本
MyScript s3 = this.GetComponentInParent<MyScript>();

//得到多个脚本
MyScript[] ss3 = this.GetComponentsInParent<MyScript>(true);
```

# 二、重要组件和 API
## GameObject
### 成员变量
```cs
//名字
print(this.gameObject.name);

//是否激活
print(this.gameObject.activeSelf);

//static
print(this.gameObject.isStatic);

//层级
print(this.gameObject.layer);

//标签
print(this.gameObject.tag);

//transform
print(this.transform.position);
```

