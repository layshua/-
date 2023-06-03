
---
title: Unity3D
aliases: []
tags: []
create_time: 2023-06-02 22:33
uid: 202306022233
banner: "![[Pasted image 20230602223746.png]]"
---
# 零、Unity 工作原理
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

# 一、基本规则
## 创建规则
 1. 不在 VS 中创建脚本了
 2. 可以放在 Assets 文件夹下的任何位置（建议同一文件夹管理)
 3. 类名和文件名必须一致,不然不能挂载 (因为反射机制创建对象，会通过文件名去找 Type)
 4. 建议不要使用中文名命名
 5. 没有特殊需求不用管命名空间
 6. 创建的脚本默认继承 MonoBehavior

默认脚本内容路径：`Editor\DataResources\ScriptTemplates`
## MonoBehavior 基类
1. 创建的脚本默认都**继承 MonoBehaviour**，继承了它才能够挂载在 GameObject 
>当我们把脚本拖到 GameObject 上时，引擎会根据文件名通过反射得到对应的类，如果该类继承了 MonoBehaviour，则允许挂载。
2. 继承了 MonoBehavior 的脚本不能 new ，**只能挂载**！
3. 继承了 MonnBehavior 的脚本不要去写构造函数，因为我们不会去 new 它，写构造函数没有任何意义
4. 继承了 MonoBehavior 的脚本可以在一个对象上挂多个 (如果没有加 DisallowMultipleComponent 特性)
   ![[Pasted image 20230603130352.png]]
5. 继承 MonoBehavior 的类也可以再次被继承，遵循面向对象继承多态的规则

### 不继承 MonoBehaviour 的类
1. 不能挂载在 GameObject 上
2. 想怎么写怎么写，如果要使用需要自己 new
3. **一般是单例模式的类（用于管理模块）或者数据结构类（ 用于存储数据）**
4. 不用保留默认出现的几个函数

## 打印
在 Unity 中打印信息的两种方式
```cs
//1.没有继承MonoBehaviour的类的时候，可以使用Debug.Log
Debug.Log("Awake Hello!");
Debug.LogError("Awake Error");
Debug.LogWarning("Awake Warning");  
//2. 继承了MonoBehaviour的类，可以使用线程方法print
print("Awake Hello!");
```
## 生命周期函数
游戏的本质就是一个死循环，每一次循环处理游戏逻辑就会更新一次画面，一帧就是执行一次循环。
Unity 底层已红帮助我们做好了死循环，我们需要学习 Unity 的生命周期函数，利用它做好的规则来执行我们的游戏逻辑就行了。

> [!NOTE] 生命周期函数的概念
> - 所有继承 MonoBehavior 的脚本最终都会挂载到 Gameobject 游戏对象上
> - 生命周期函数就是该脚本对象依附的 Gameobject 对象从出生到消亡整个生命周期中会**通过反射自动调用的一些特殊函数**
>- Unity 帮助我们记录了一个 Gameobject 对象依附了哪些脚本，会自动的得到这些对象，通过反射去执行生命周期函数

- 生命周期函数的访问修饰符一般为 private 和 protected（默认为private）
- 因为不需要再外部自己调用生命周期函数都是 Unity 自己帮助我们调用的
- 支持继承多态

常用的生命周期函数：
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

