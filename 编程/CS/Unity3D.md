
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
## Object 类
**Object 是 Gameobject 的父类** 
- unity 里面的 Object 不是指的 cs 中的万物之父 object（cs 中的 object 命名空间是 system ）
- unity 里的 Object 命名空间是 UnityEngine ，也是继承万物之父的一个自定义类
## GameObject 类
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

### 静态方法

> [!warning] 
> 如果是继承 MonoBehaviour 的类，可以不加 `.GameObject` 前缀

#### 查找对象
得到某一个单个对象目前有 2 种方式
1. 是 public 从外部面板拖进行关联（推荐）
2.  通过 API 去找

以下方法通过 API 去找：
- 只能找到被激活的对象
- 如果场景中存在多个满足条件的对象 (比如同名、同 tag)，无法准确找到是谁

```cs
//创建几何体
//只要得到了一个Gameobject 对象我就可以得到它身上挂在的任何脚本信息
//通过obj.GetComponent来得到脚本信息
GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Sphere);
obj.name = "球体";
obj.tag = "Player";

//1 查找单个对象
//通过对象名查找，效率低，需要遍历所有对象
GameObject obj2 = GameObject.Find("球体");
if (obj2 != null)
{
    print(obj2);
}

//通过tag查找象，需要遍历所有对象
GameObject obj3 = GameObject.FindWithTag("Player");
if (obj3 != null)
{
    print(obj3);
}

//找到场景中挂载的某一个脚本对象 ，效率太低，需要遍历所有对象，还要便利对象上挂载的脚本 
TestScript ts = GameObject.FindObjectOfType<TestScript>();

//2.查找多个对象（只能通过tag）
GameObject[] objs = GameObject.FindGameObjectsWithTag("Player");
```

#### 实例化对象（Clone）
实例化对象 (克隆对象)的方法
作用：根据一个 Gameobject 对象创建出一个和它一模一样的对象
```cs
//准备克隆的GameObject
public GameObject obj;
void Start()
{
    //准备用来克隆的对象
    //1.直接是场景上的某个对象
    //2.可以是一个预设体对象
    GameObject objClone = GameObject.Instantiate(obj);
}
```

#### 删除对象
`Destrdy` 方法不会马上移除对象，一般情况下它会在下一帧时把这个对象移除并从内存中移除

```cs
//删除GameObject对象 
GameObject.Destroy(obj,5);  //第二个参数可选，表示延迟几秒删除

//删除脚本对象
GameObject.Destroy(this);

//立即移除
//如果没有特殊需求，不用该方法，因为该方法不是异步的，可能会卡顿
GameObject.DestroyImmediate(obj);

//过场景不移除
//默认情况在切换场景时场景中对象都会被自动删除掉
//如果你希望某个对象过场景不被移除就使用该方法
//一般都是传依附的Gameobject对象
//比如下面这句代码的意思就是自己依附的Gameobject对象过场景不被删除
GameObject.DontDestroyOnLoad(this.gameObject);
```

## 成员方法

#### 创建GameObject
```cs
//创建空GameObject
GameObject obj1 = new GameObject(); //默认名字New Game Object
GameObject obj2 = new GameObject("物体"); //自定义名字
GameObject obj3 = new GameObject("物体", typeof(TestScript)); //自定义名字，添加脚本，可以添加多个
```

#### 添加/获取脚本
```cs
//为对象添加脚本
//继承MonoBehaviour的脚本是无法new的
//如果给GameObject对象动态添加继承MonoBehaviour的脚本，需要使用AddComponent方法
TestScript ts1 = obj1.AddComponent(typeof(TestScript)) as TestScript;
//⭐用泛型更方便，推荐！
TestScript script = obj1.AddComponent<TestScript>();
//通过返回值，可以得到脚本的信息，来进行一些处理

//获取脚本
TestScript ts2 = obj1.GetComponent<TestScript>();
```

#### 标签比较
```cs
 //标签比较
if (this.gameObject.CompareTag("Player"))
{
    
}
//显式字符串比较效率低下，建议改用‘CompareTag'
if (this.gameObject.tag == "Player")
{
}
```
#### 激活失活
```cs
 //设置激活失活
 obj1.SetActive(false);
```

#### 发送消息
以下方法不建议使用，效率比较低

通过广播或者发送消息的形式，让自己或者别人执行某些行为
```cs
void Start()
{
    //通知自己执行什么行为
    //命令自己去执行这个Test这个函数 会在自己身上挂载的所有脚本去找这个名字的函数
    //它会去找到自己身上所有的脚本有这个名字的函数去执行
    this.gameObject.SendMessage("TestFunc");
    this.gameObject.SendMessage("TestFunc1",10); //第二个参数可以传参
    
    //广播，让自己和自己的子对象执行函数
    this.gameObject.BroadcastMessage("TestFunc");
    
    //向父对象和自己发送消息并执行
    this.gameObject.SendMessageUpwards("TestFunc");
}

void TestFunc()
{
    print("Hello World!");
}

void TestFunc1(int i)
{
    print("Hello World!"+i);
}
```

## Time 类
```cs file:游戏时间
void Update()
{
    //时间缩放比例 
    Time.timeScale = 0; //时间停止
    Time.timeScale = 1; //时间正常
    Time.timeScale = 2; //2倍速 
    
    //帧间隔时间（最近的两帧之间的时间间隔）主要用于计算位移
    //路程=速度*时间
    //受Scale影响的帧间隔时间
    print(Time.deltaTime);   //如果希望游戏暂停时就不动的，就是用deltaTime
    //不受Scale影响的帧间隔时间
    print(Time.unscaledDeltaTime); //如果希望游戏暂停时还能动的，就是用unscaledDeltaTime
    
    //游戏开始到现在的时间
    //受Scale影响
    print(Time.time);
    //不受Scale影响
    print(Time.unscaledTime);
    
    //游戏开始到现在跑了多少帧
    print(Time.frameCount);
    
}
```

```cs file:物理时间
private void FixedUpdate()
{
    //物理帧间隔时间
    //受Scale影响
    print(Time.fixedTime);
    
    //不受Scale影响
    print(Time.fixedUnscaledTime);
}
```

## Transform 类
游戏对象（Gameobject）位移、旋转、缩放、父子关系、坐标转换等相关操作都由它处理，它是 unity 提供的极其重要的类
#### Vector3
```cs file:Vector3
//Vector3的初始化
Vector3 v1 = new Vector3();
v1.x = 10;
v1.y = 10;
v1.z = 10;

Vector3 v2 = new Vector3(10, 10, 10);

Vector3 v3 = Vector3.zero; // (0, 0, 0)
Vector3 v4 = Vector3.one; // (1, 1, 1)
Vector3 v5 = Vector3.right; // (1, 0, 0)
Vector3 v6 = Vector3.left; // (-1, 0, 0)
Vector3 v7 = Vector3.up; // (0, 1, 0)
Vector3 v8 = Vector3.down; // (0, -1, 0)
Vector3 v9 = Vector3.forward; // (0, 0, 1)
Vector3 v10 = Vector3.back; // (0, 0, -1)

//计算点之间的距离
float distance = Vector3.Distance(v1, v2);
```

#### 位置
> [!NOTE] Inspector 面板上的 Transfrom 信息
> 对于父对象来说，positon 是世界空间位置
> 对于子对象来说，position 是相对于父对象的位置，即在父对象为原点的局部空间中位置
> 

```cs file:position
 //世界空间位置
print(this.transform.position);

//局部空间位置
print(this.transform.localPosition);

//position的赋值不能单独改变x,y,z，只能整体改变
//this.transform.position.x = 1; error！
this.transform.position = new Vector3(1, 1, 1);
this.transform.position = new Vector3(this.transform.position.x+100, this.transform.position.y, this.transform.position.z);

print(this.transform.forward); //局部空间的z轴方向，注意和Vector3.forward区分
print(this.transform.right);   //局部空间的x轴方向
print(this.transform.up);      //局部空间的y轴方向
```
#### 位移
```cs file:位移
//理解坐标系下的位移计算公式
//路程–方向*速度*时间
//方式一：自己计算
//想要变化的就是 position
this.transform.position += this.transform.forward * (1 * Time.deltaTime);  //朝对象局部空间的z轴前进
this.transform.position += Vector3.forward * (1 * Time.deltaTime);  //朝世界空间Z轴前进

//方式二：API
//参数一:表示位移多少路程=方向*速度*时间
//参数二:表示相对坐标系 ,默认该参数是自身局部空间
this.transform.Translate(Vector3.forward, Space.World); //始终朝向世界空间Z轴移动
this.transform.Translate(Vector3.forward,Space.Self);  //始终朝向局部空间Z轴移动
this.transform.Translate(this.transform.forward, Space.Self);  //方向错误   
this.transform.Translate(this.transform.forward, Space.World); //始终朝向局部空间Z轴移动

//实际上我们就是想用始终朝向局部空间Z轴的移动，无非就是两种情况：
//1. 局部空间的（0，0，1）
//2. 局部空间的（0，0，1）在世界空间中的坐标，即this.transform.forward
```
#### 角度和旋转
```cs file:角度
//和角度设置一样，不能单独设置x,y,z
        
//是inspector界面上显示的Rotation是欧拉角
print(this.transform.eulerAngles); //该方法返回欧拉角,
print(this.transform.localEulerAngles);

print(this.transform.rotation);  //该方法返回四元数
print(this.transform.localRotation); 
```

```cs file:旋转
 void Update()
{
    //绕轴自转
    //方法一：
    //参数一：每帧旋转的角度
    //参数二：默认Space.Self
    this.transform.Rotate(new Vector3(0,10,0) * Time.deltaTime,Space.World);
    
    //方法二：
    //参数一：绕哪个轴旋转
    //参数二：是每帧转动的角度
    //参数三：默认Space.Self
    this.transform.Rotate(Vector3.up, 10 * Time.deltaTime, Space.World);
    this.transform.Rotate(Vector3.up, 10 * Time.deltaTime, Space.Self); 


    //绕点转
    //点，轴，旋转速度
    this.transform.RotateAround(Vector3.zero, Vector3.up, 10 * Time.deltaTime);
}
```

### 缩放和LookAt
```cs file:缩放
//相对世界坐标系的缩放大小只能得，不能改
print(this.transform.lossyScale); 

//相对局部坐标系(父对象)
this.transform.localScale  = new Vector3(1.0f, 1.0f, 1.0f);
//和角度设置一样，不能单独设置x,y,z

//Unity没有提供关于缩放的API，只能自己修改localScale
```

```cs file:LookAt
this.transform.LookAt(Vector3.zero); //看向点
this.transform.LookAt(obj); //看向一个对象，参数为对象的Transform
```

### 父子关系
#### 获取和设置父对象
```cs
//获取父对象
print(this.transform.parent.name);

//断绝父子关系
this.transform.parent = null;

//设置父对象
this.transform.parent = GameObject.Find("FatherObject").transform;

//通过API设置，差别主要是多了一个参数二
//参数一:我的父亲
//参数二:是否保留世界坐标的位置角度缩放信息
//true会保留世界，坐标下的状态和父对象进行计算得到本地坐标系的信息
//false不会保留，会直接把世界坐标系下的位置角度缩放直接赋值到本地坐标系下,通常会改变原位置
this.transform.SetParent(null); //断绝父子关系
this.transform.SetParent(GameObject.Find("FatherObject").transform,false); //设置父对象
this.transform.DetachChildren(); //和自己的所有儿子断绝关系，不会影响儿子和孙子的关系
```

#### 获取子对象
```cs
//按名字查找儿子
//只能找儿子，不能找孙子
//Find方法效率比GameObject.Find()高，前提要知道父亲是谁
//Find方法是能够找到失活的对象的! Gameobject相关的查找是不能找到失活对象的
print(this.transform.Find("Son").name);

//遍历儿子
for (int i = 0; i < this.transform.childCount; i++) 
{
    //通过索引号找到特定的儿子
    print(this.transform.GetChild(i).name);
}
```

#### 儿子的操作
```cs
//判断是不是我的儿子
if (son.IsChildOf(this.transform))
{
}

//得到自己作为儿子的编号  sibling:兄弟姐妹
print(son.GetSiblingIndex());

//把自己设置为第一个儿子
son.SetAsFirstSibling();

//把自己设置为最后一个儿子
son.SetAsLastSibling();

//把自己设置为指定索引号的儿子，编号超出范围不会报错，自动设置会最后一个编号
son.SetSiblingIndex(5);

```

### 自定义拓展方法
1. 为 Transform 写一个**拓展方法**，可以将它的子对象按名字的长短进行排序改变他们的顺序名字短的在前面，名字长的在后面。
![[Pasted image 20230604162029.png]]
```cs file:tool.cs
//写一个Transfrom类的拓展方法
public static class Tools
{
    //为Transform添加一个拓展方法
    //可以将它的子对象按名字的长短进行排序改变他们的顺序，名字短的在前面，名字长的在后面
    public static void Sort(this Transform obj)
    {
        List<Transform> list = new List<Transform>();
        for (int i = 0; i < obj.childCount; i++) 
        {
            list.Add(obj.GetChild(i));
        }
        //这是根据名字长短进行排序利用的是list的排序
        list.Sort((a, b) =>
        {
            if(a.name.Length < b.name.Length)
                return -1;
            else if(a.name.Length > b.name.Length)
                return 1;
            else
                return 0;
        });
        
        //根据list中的排序结果重新设置每一个对象的索引编号
        for (int i = 0; i < list.Count; i++)
        {
            list[i].SetSiblingIndex(i);
        }
    }
}

//然后在父对象挂载的脚本中调用即可
void Start()
{
    this.transform.Sort();
} 
```

2. 请为 Transform 写一个拓展方法，传入一个名字查找子对象，即使是子对象的子对象也能查找到
```cs file:tool.cs
public static Transform CustomFind(this Transform father, string childName)
{
    //要找的子对象
    Transform target = null;
    //先从自己身上的子对象找
    target = father.Find(childName);
    if (target != null) 
        return target;
    
    //如果自己身上没有，就从自己的子对象的子对象找
    for (int i = 0; i < father.childCount; i++)
    {
        //递归
        target = father.GetChild(i).CustomFind(childName);
        if (target != null)
            return target;
    }

    return target;
}

//然后在父对象挂载的脚本中调用即可
print(this.transform.CustomFind("aaa").name);
```

### 坐标转换
以下是从正 Y 轴向下看的视角，中间有一个 Cube 模型

![[zip/images/Diagram.svg]]

世界坐标系的点 P(0，0，1)转换到局部空间，则 P 点坐标的 x，z 在局部空间为负数。
世界坐标系的向量P（0，0，1）转换为局部空间，将左边的向量平移到右边，可以观察到该方向向量的 x 为负数，z 为证书
```cs file:世界坐标转局部坐标
//世界坐标系的点转换为局部坐标系点（会受缩放影响）
//上图中的P即为Vector3.forward
this.transform.InverseTransformPoint(Vector3.forward);

//世界坐标系的向量转换为本地坐标系的向量
//不受缩放影响
this.transform.InverseTransformDirection(Vector3.forward);
//受缩放影响
this.transform.InverseTransformVector(Vector3.forward);
```


```cs file:局部坐标转世界坐标
//⭐点（受缩放影响）
print(this.transform.TransformPoint(Vector3.forward));  

//向量
//不受缩放影响
print(this.transform.TransformDirection(Vector3.forward));
//受缩放影响
print(this.transform.TransformDirection(Vector3.up));
```
其中**最重要的就是局部坐标系的点转世界坐标系的点**
比如现在玩家要在自己面前的 n 个单位前放一团火，这个时候我不用关心世界坐标系
通过相对于本地坐标系的位置转换为世界坐标系的点，进行特效的创建或者攻击范围的判断,
## Input 类
输入相关内容都写在 Update 中

### 鼠标键盘输入
```cs file:鼠标输入
//鼠标在屏幕上的位置
//屏幕坐标的原点是在屏幕的左下角，往右是x轴正方向，往上是Y轴正方向
//返回值是Vector3，但是只有x和y有值，z一直是0是，因为屏幕本来就是2D的不存在z轴
Input.mousePosition

//检测鼠标输入
//0左键 1右键 2中键

//按下Down
Input.GetMouseButtonDown(0)

//抬起Up
Input.GetMouseButtonUp(0)

//按住
Input.GetMouseButton(0)

//中键滚动
//它的返回值是（0，Y），返回值的Y -1往下滚  0没有滚  1往上滚
Input.mouseScrollDelta
```

```cs file:键盘输入
//键盘按下
//方法一(推荐)
Input.GetKeyDown(KeyCode.W)


//方法二：传入字符串的重载
//只能传入小写字符串
Input.GetKeyDown("w")


//键盘抬起
Input.GetKeyUp(KeyCode.W)
    
//键盘按住
Input.GetKey(KeyCode.W)

```

```cs file:任意键
//任意键 按下
Input.anyKeyDown

//任意键 抬起
Input.anyKeyUp

//任意键 按下
Input.anyKey
```

### 默认轴输入
![[Pasted image 20230604213230.png]]
我们学习鼠标键盘输入主要是用来控制玩家，比如旋转位移等等，所以 unity 提供了更方便的方法来帮助我们控制对象的位移和旋转。
```cs file:默认轴输入
//鼠标AD按下时，返回-1到1之间的浮点值
//相当于得到这个值，就是我们的左右方向，用于控制左右移、旋转
Input.GetAxis("Horizontal")

//鼠标WS按下时，返回-1到1之间的浮点值
//相当于得到这个值，就是我们的上下方向，用于控制上下移、旋转
Input.GetAxis("Vertical")

//鼠标横向移动时，返回-1到1之间的浮点值
Input.GetAxis("Mouse X")

//鼠标纵向移动时，返回-1到1之间的浮点值
Input.GetAxis("Mouse Y")


//GetAxisRaw方法和GetAxis使用方式相同
//只不过它的返回值只会是-1,0,1不会有中间值
```

### 移动设备
```cs file:移动设备
//移动设备触摸相关
if (Input.touchCount > 0)
{
    Touch t1 = Input.touches[0];
    
    //位置
    print(t1.position);
    
    //相对上次位置的变化
    print(t1.deltaPosition);
}
//是否启用多点触控
Input.multiTouchEnabled = false;

//陀螺仪
//是否启用陀螺仪
Input.gyro.enabled = true;

//陀螺仪的旋转速度 
print(Input.gyro.rotationRate);

//陀螺仪的重力加速度向量
print(Input.gyro.gravity);

//陀螺仪 当前的旋转四元数
//比如用这个角度信息来控制场景上的一个3D物体受到重力影响
//手机怎么动它怎么动
print(Input.gyro.attitude);
```
### 手柄输入
```cs file:手柄输入
//得到连接的手柄的所有按钮名字
string[] strs = Input.GetJoystickNames();
        
//某一个手柄键按下
Input. GetButtonDown("Jump")
  
//某一个手柄键抬起
Input.GetButtonup("Jump"))

//某一个手柄键长按
Input.GetButton("Jump"))
```

## Screen 类

### 静态属性
```cs
//当前设备屏幕分辨率
Resolution r = Screen.currentResolution;
print(r.width);
print(r.height);

//Game窗口宽高
print(Screen.width);
print(Screen.height);

//屏幕睡眠模式
Screen.sleepTimeout = SleepTimeout.NeverSleep;
Screen.sleepTimeout = SleepTimeout.SystemSetting;

//运行时是否全屏模式
Screen.fullScreen = true;

//窗口模式
//独占全屏FullscreenMode.ExclusiveFullscreen
//全屏窗口FullscreenMode.Fullscreenwindow
//最大化窗口FullscreenMode. Maximizedwindow
//窗口模式FullscreenMode.windowed
Screen.fullScreenMode = FullScreenMode.Windowed;

//移动设备屏幕转向相关
//允许自动旋转为左横向 Home键在左
Screen.autorotateToLandscapeLeft = true;
//允许自动旋转为右横向 Home键在右
Screen.autorotateToLandscapeRight = true;
//允许自动旋转到纵向 Home键在下
Screen.autorotateToPortrait = true;
//允许自动旋转到纵向倒着看 Home键在上
Screen.autorotateToPortraitUpsideDown = true;

//指定屏幕显示方向
Screen.orientation = ScreenOrientation.LandscapeLeft;
```
### 静态方法
```cs
//设置分辨率
Screen.SetResolution(1920, 1080, true); //第三个参数是是否全屏
```

## Camera
### 可编辑参数
![[Pasted image 20230604230509.png|450]]
**Clear Flags**：清除标志，如何清除背景
- skybox 天空盒 
- Solid Color 颜色填充
- Depth only 只画该层，背景透明
- Don't Clear 不移除，覆盖渲染（不会有）

**Culling Mask**：剔除遮罩，指定渲染哪些 Layer 

**Physical Camera**：物理摄像机
勾选后可以模拟真实世界中的摄像机焦距，传感器尺寸，透镜移位等等
- Focal Length 焦距
- Sensor Type  传感器类型
- Sensor Size 传感器尺寸
- Lens Shift 透镜移位
- Gate Fit 闸门配合


**Depth**：深度决定渲染顺序
- 多相机的时候要设置顺序。（比如 UI 一个摄像机，游戏一个摄像机，设置两个 Culling Mask 不同的摄像机绘制不同的 Layer） 
- 深度较大的绘制在深度较小的上方（覆盖小的），game 中的界面是深度最大相机渲染出来的
- 深度大的可以将 Clear Flags 设置为 Depth only，这样就可以渲染出多个摄像机拍摄的内容

**Viewport Rect**：视口范围
可以起到分屏的作用，这样可以放置多个摄像机，实现分屏多人游戏

**Target Texture**：渲染纹理
- 可以把摄像机画面渲染到一张图上，用于制作小地图
- 在 Project 右键创建 RenderTexture

## 代码相关
