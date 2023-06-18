
# ⭐ 前言

在[游戏开发](https://so.csdn.net/so/search?q=%E6%B8%B8%E6%88%8F%E5%BC%80%E5%8F%91&spm=1001.2101.3001.7020)过程中，我们会经常与数据打交道。很多时候，我们会为游戏对象配置数据，比如玩家初始生命值、技能伤害等。那么就需要有个能够存储数据的东西来和游戏对象进行交互。

假设现在要做一个 “打飞机” 的游戏，玩家操控的飞机在吃了不同道具后会发射不同类型的子弹，这些子弹的飞行速度和伤害各不相同。那么显然我要为每一种子弹配置好它的属性值。

实现方式有很多，比如我为每种子弹创建预制体，然后写脚本去定义代表子弹各个属性的成员变量，再把脚本挂载到子弹预制体上。为了方便在开发时调试我们可以将这些变量声明为 public ，以便我们在编辑器面板中对数据进行改动。  

![[4e1d663836a8844b9b98207bded7f381_MD5.png]]

这么做其实是有缺点的：

1） **每生成一个子弹就会对原来的预制体进行拷贝，其挂载的 Bullet 数据脚本同样会被拷贝，因此同样的数据会被拷贝多次**。而我们规定同一种子弹的数据是相同的，也就是同一种子弹，不管生成了多少个，它们都共用一套数据。因此，这种方法会**创建多余的数据脚本，造成内存浪费**。  
2） 如果预制体上的脚本丢失，之前在 Inspector 面板中配置的数据也会消失。  
😫😫😫😫😫😫😫

不过解决方法有很多，比如：

1️⃣ 我们可以创建一个全局的数据管理中心脚本，通过静态变量去调用每种子弹的数据。不过这种数据配置方式不能实现数据的持久化，而且必须要打开代码文件进行修改，面对茫茫代码可能不是那么直观。  
2️⃣ 我们还可以用上像 **excel，Json，xml 等持久化数据存储**的方法，结合 Unity 对准备好的数据文件进行数据读写。这么做的好处是可以实现数据的持久化，比如在游戏过程中修改了数据，退出游戏后下一次打开游戏使用的就是之前修改过的数据。

那么现在，我将介绍另一种用于数据存储的解决方案——ScriptableObject。它也能弥补通过挂载继承自 MonoBehaviour 的脚本来配置数据的一些不足。相较于 excel，Json，xml 这类持久化数据存储的方法，它有些额外的优点，但是也存在一些局限性。我将会在接下来的部分详细说明。

# 🔍 什么是 ScriptableObject

*   ScriptableObject 是 Unity 提供的一个**数据配置存储基类**，它是一个可以用来保存大量数据的**数据容器**，我们可以将它保存为自定义的**数据资源文件**。
*   ScriptableObject 是一个类似 MonoBehaviour 的基类，继承自 UnityEngine.Object 。要想使用它，需要我们写个脚本去继承 ScriptableObject 。需要注意的是，继承自 SctiptableObject 的脚本无法挂载到游戏物体上，毕竟它不是继承自 MonoBehaviour。
*   ScriptableObject 类的**实例**会被保存成资源文件（.asset 文件），和预制体，材质球，音频文件等类似，都是一种资源文件，存放在 Assets 文件夹下，创建出来的实例也是唯一存在的。

简单的看一下 ScriptableObject 的实例长啥样：  

![[1e5f113aea457891c1416f9a301fb188_MD5.png]]

上图是编辑器窗口中的样子，再来看看本地文件夹中的模样：  

![[693da9ec55e4e252b4d18f0a25991a8b_MD5.png]]

  
可以看到 ScriptableObject 实例就是一种资源文件。细心的你可能会发现我这里强调的是 **ScriptableObject 的实例（instance）**，那么你也许会联想到类与实例（或者叫对象）的关系。其实 ScriptableObject 也是如此，**它本身是个类，实例化之后得到的就是数据资源文件**。

那么，要如何使用 ScriptableObject 这个类，又如何创建出编辑器面板中的这个数据资源文件呢？在此之前，我先介绍一下 ScriptableObject 的主要作用，来加深大家对它的理解。刚刚已经介绍了它是什么东西，那么现在将进一步介绍它一般能用来做什么。

# 🔍 ScriptableObject 的主要作用

大体上可以分成三点：  
1） 编辑模式下的[数据持久化](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E6%8C%81%E4%B9%85%E5%8C%96&spm=1001.2101.3001.7020)  
2） 配置文件 （配置游戏中的数据）  
3） 数据复用 （多个对象共用一套数据）

## 💪 编辑模式下的数据持久化

数据持久化：使用数据时从硬盘中读取，数据改变后保存到硬盘上，游戏退出后数据信息被存储到硬盘上，达到持久化的目的。

当我们在编辑模式下修改了继承自 ScriptableObject 对象的数据文件内容时，修改的数据将被保存到磁盘上。但是在发布运行后，即使在游戏中修改了 ScriptableObject 的数据，改后的数据并不会保存在本地，**重新打开运行时数据并还是配置的初始数据**。

因此 ScriptableObject 适合在编辑模式下调试数据，但不适合存储在游戏打包发布后的运行期间会改变的数据 ❗

## 💪 配置文件

ScriptableObject 非常适合用来做配置文件。因为：  
1）配置文件的数据在游戏发布之前就定好了规则  
2）配置文件的数据在游戏运行时只会读出来使用，不会修改数据的内容  
3）传统的配置文件一般会通过 xml、json、excel 等方式来配置游戏数据  
，相对来说都是在 Unity 外部通过其它格式的文件对数据进行配置。  
而通过 ScriptableObject 我们可以直接**在 Unity 内部的 Inspector 面板中进行数据的配置**，有时候会更加方便。

我们也可以利用 ScriptableObject 数据文件来制作编辑器相关功能，比如制作 Unity 内置的技能编辑器、关卡编辑器等。因为内置编辑器只在编辑模式下运行，在编辑模式下 ScriptableObject 正好具有数据持久化的特性。

## 💪 数据复用

对于只用不变的数据，通过使用 ScriptableObject 可以有效避免内存的浪费，因为它将共用的数据单独抽离出来，供相同的一类对象使用。

还是利用前言中举的例子，比如一个子弹对象，通过面向对象的思想，会写一个继承自 MonoBehaviour 的脚本，声明相关的属性，然后挂载到子弹预设体上，把子弹需要的数据赋给子弹对象。如果我们要求子弹的数据是**不会改变**的，那么这样每次实例化一个子弹，对内存来说会造成一定的浪费，因为每次生成一个子弹都会复制 Assets 下子弹预制体的值，也就是多次复制了相同的数据。  

![[65d2a4a611ac044c5abbacc9b610ec39_MD5.png]]

![[4f6b0bcd41992e531c687a571c9fdd21_MD5.png]]

**这样每一个子弹预设体上都有该脚本，该脚本中的所有的属性都会分配一次内存**。

但是如果是用 ScriptableObject，对于某一种子弹，我们**只需要拥有一份 ScriptableObject 的实例，也叫做数据资源文件，然后这一种子弹的所有游戏物体都引用这个数据资源文件**，就可实现不管生成多少个子弹，只要它们属于同一类，都会共用一份数据。  

![[553cf2048f555db8969ad84bbfa5c3e5_MD5.png]]

这里每个子弹预制体中的子弹脚本都只是持有 Bullet Data 这一个 ScriptableObject 实例的引用，真正在内存当中分配空间的只有红线所指向的 ScriptableObject 实例，也就是我们的数据资源文件。

# 🔍 如何创建 ScriptableObject

ScriptableObject 是个类，因此自然会有先声明，后实例化的步骤。

## 📕 步骤一：声明自定义的数据容器

第一步：创建一个脚本继承自 ScriptableObject 类  
第二步：在该类中声明成员，规定要存哪些类型的数据 （如果后续创建了数据资源文件，要在 Inspector 窗口中看到这些成员，需要把它们声明为 public ）

```
public class BulletData : ScriptableObject
{
    public float speed;
    public float damage;
}
```

这一步就相当于定义了一个数据的模板。

## 📕 步骤二：根据自定义的 ScriptableObject 数据容器创建数据文件

这一步就相当于根据定义的模板实例化 ScriptableObject ，具体有两种方法：

### 😊 方法一：为类添加 CreateAssetMenu 特性，在编辑器的菜单中创建资源文件

```
[CreateAssetMenu(fileName = "BulletData", menuName = "ScriptableObject/子弹数据", order = 0)]
public class BulletData : ScriptableObject
{
    public float speed;
    public float damage;
}
```

这种创建 ScriptableObject 资源文件的方法还是比较简单且常见的，主要就是在类的上方加一个 CreateAssetMenu 特性。

fileName 表示数据资源文件创建出来的文件名。  
menuName 表示在 Assets/Create 下的名字。  
order 表示在 Assets/Create 下的位置顺序。

这里涉及到一点编辑器扩展的知识。光看这一段文字描述也许看不出个所以然，那么接下来我会带着大家实操一遍，过程其实非常的简单粗暴😉。  
既然要创建资源文件，大家其实可以类比创建材质、预制体，它们也是一种资源文件。要想创建它们，我们只要在编辑器中的 Asset 目录或者它的子目录中按下鼠标右键（或者直接点击编辑器最上方菜单栏的 Assets），然后点击 Create，找到我们想要创建的资源就行了。  

![[6a54d05cff033c5abbd0c5e5f1968a7c_MD5.png]]

![[cce25a63f89be9ccaecf1528e07dcaeb_MD5.png]]

  
（注：点击 Project 面板中的 “+” 也能实现）

那么 ScriptableObject 也是一样，只不过**我们要手动为继承自 ScriptableObject 的类添加 CreateAssetMenu 特性，让它能像预制体、材质球这类资源文件一样能够在编辑器面板中手动创建。**

如果你像我演示的代码那样添加了 CreateAssetMenu 特性，你会发现 Create 的下一级目录的最上方出现了一个 ScritableObject。  

![[f331e30c66459ff2c433feadbea5a632_MD5.png]]

我们还可以把鼠标移到菜单中 ScriptableObject 的位置，可以发现它又展开了下一级目录：  

![[d82ba5e8713ff5a2433f2b8e28b60252_MD5.png]]

这时候出现了我们的 “子弹数据” （这一级目录下还有我之前创建的其他 ScriptableObject） ，ScriptableObject 下的“子弹数据” 就是我们刚刚在 menuName 中定义的，然后我们点击它。  

![[1e5f113aea457891c1416f9a301fb188_MD5.png]]

这时候就成功创建了我们的数据资源文件。然后我们就可以在 Inspector 面板中配置数据了。  
回看 CreateAssetMenu 特性, 你是否能理解它的一些属性了呢？

fileName 的值就对应了我们创建出来的数据资源文件的文件名。  
menuName 的值对应了我们要如何在编辑器面板中创建出数据资源文件的路径，从 Assets/Create 开始，我们如果想将路径分级，可以在路径名之间用 “/” 隔开。  
那 order 是怎么一回事呢？它的默认值是 0，我们之前看到 ScriptableObject 目录下 “子弹数据” 是排在第一位的，那我如果把 order 的值由 0 改成 1 会发生什么事呢？请看：  

![[4ca118182146752514fc333a59e78251_MD5.png]]

  
事先说明一下，我的另外两个 ScriptableObject 的 order 也是设成 0，那么你可以看到 “子弹数据” 现在排在了第三位。  
那么这个 order 其实就规定了创建 ScriptableObject 数据资源文件的路径在菜单上显示的位置，order 越低，就显示在越上面。如果 order 相同，则是按照继承自 ScriptableObject 的脚本创建时间排序，新创建的排在上面。

### 😊 方法二：利用 ScriptableObject 的静态方法创建数据对象，然后将数据对象保存在工程目录下

可以新建一个脚本（可以不用继承自 MonoBehavoiur，这个脚本不用挂载到游戏物体上），引入 UnityEditor 命名空间：

```
using UnityEngine;
using UnityEditor;
public class ScriptableObjectTool 
{
    [MenuItem("ScritableObject/CreateMyData")]
    public static void CreateMyData()
    {
        //创建数据资源文件
        //泛型是继承自ScriptableObject的类
        BulletData asset = ScriptableObject.CreateInstance<BulletData>();
        //前一步创建的资源只是存在内存中，现在要把它保存到本地
        //通过编辑器API，创建一个数据资源文件，第二个参数为资源文件在Assets目录下的路径
        AssetDatabase.CreateAsset(asset, "Assets/Resources/ScriptableObject/BulletData.asset");
        //保存创建的资源
        AssetDatabase.SaveAssets();
        //刷新界面
        AssetDatabase.Refresh();
    }
}
```

保存脚本后，会发现编辑器最上方的菜单栏多出了这个（MenuItem 特性起的作用）：  

![[372912b963bcc47098df28a5274fa252_MD5.png]]

  
点击后，可以看到 Assets/Resources/ScriptableObject 文件夹下多了我们想要的数据资源文件：  

![[9169cae06e4925acec6959d3a8bf1e3f_MD5.png]]

**注意：**  
1️⃣ 使用这种方法无需在继承自 ScriptableObject 的类上增加 CreateAssetMenu 特性。  
2️⃣ **刚刚创建的 ScriptableObjectTool 脚本需要放在 Assets 文件夹下任一位置的 Editor 文件夹下**（这个文件夹放哪都行，看自己需求，只要在 Assets 文件夹或其子文件夹下就好）。**因为我们引入了 UnityEditor 命名空间，这意味着这个脚本只在编辑模式下会用到，实际打包发布后是不会用到的**。如果没放在 Editor 文件夹下，Unity 打包时会认为此脚本是会被一起打包，作用于游戏运行期间，与 Editor 命名空间的性质相矛盾，所以会报错。（这里就涉及到一些扩展编辑器的知识）  

![[8b32c27a0db09a13b6f0ffe140ed0698_MD5.png]]

### 😊 总结创建 ScriptableObject 的步骤

1）写个脚本，继承 ScriptableObject 类  
2）声明需要的数据变量  
3）添加特性来创建数据资源文件。这相当于一种专门用来记录数据的资源，和预制体，材质球，音频文件一样都是资源，只不过是通过继承自 ScriptableObject 类生成的数据资源文件。

# 🔍 如何使用 ScriptableObject

我们刚刚介绍了如何创建 ScriptableObject，但是此时我们只是创建了一个数据资源，并没有将它存储的数据和我们的游戏对象关联起来。那么接下来，我将介绍如何去运用创建出来的数据资源文件。

## 📕 Ⅰ. 数据文件的使用

### 😊方法一：通过 Inspector 面板中的 public 变量进行关联

步骤一：创建一个数据文件  
步骤二：在继承自 MonoBehaviour 类中声明数据容器类型的成员，在 Inspector 面板中进行关联（拖拽的是数据文件而不是继承自 ScriptableObject 类的脚本）  

![[564200183c038bc2577039137227c635_MD5.png]]

不使用 ScriptableObject 的时候，我们的子弹脚本是这么写的：  

![[733ebd2f19481b7dc3415d4562c806a5_MD5.png]]

  
那么现在我们只需把数据有关的部分替换成 ScriptableObject 的引用就行了：  

![[c9dbc2093c66bf771b0190672238f238_MD5.png]]

  
然后在 Inspector 面板中拖拽赋值：  

![[b09f13f16b2b6fc509d2f5a3be836038_MD5.png]]

这个时候游戏对象就和数据资源关联起来了，不管有多少个物体挂载了 Bullet 脚本，它们关联的都是同一份数据资源。

### 😊方法二：直接加载数据资源文件

可以用 Resources，AddressBundle，Addressables 等方式加载数据资源文件。

## 📕 Ⅱ. 生命周期函数

ScriptableObject 和 MonoBehaviour 类似，也存在生命周期函数，但是数量会少很多。

```
Awake 数据文件创建时调用
OnDestroy 对象将被销毁时调用
OnEnable 创建或加载对象时调用
OnDisable 对象销毁时，即将加载脚本程序集时调用
OnValidate 编辑器才会调用的函数，Unity在加载脚本或者Inspector面板中更改值时调用
```

除此之外继承自 ScriptableObject 的类中也可以自定义函数，并不是只能声明和数据有关的变量。

## 📕 Ⅲ. 实现非持久化数据

我们知道 ScriptableObject 在打包发布后是不具备数据持久化的功能的，但是它能实现编辑模式下的持久化数据。只要我们在编辑器面板中手动创建一个数据资源文件，这个时候相当于我们在磁盘中真正创建了一个文件，之后在编辑模式中 ScriptableObject 的数据改动会被保存在磁盘中。

其实，对于某些数据资源，我们不一定要将数据保存为磁盘中的资源文件占据空间，而只希望运行期间在内存中临时生成一组共用的数据给对象使用就够了，退出游戏后就释放掉生成的数据资源。**这个时候不论是在编辑模式还是打包发布后，数据都是非持久化的，也就是改动的数据不会被保存到磁盘中而是内存中**。退出游戏后重新打开，读取的还是初始配置的数据。

**如何生成非持久化数据？**  
利用 ScriptableObject 类中的静态方法 CreateInstance<>() 。该方法可以在运行时创建出指定继承自 ScriptableObject 的对象，该对象只存在于内存中，可以被 GC 垃圾回收，调用一次就创建一次。

```
public BulletData bulletData;
void Start(){
//通过这种方式创建的数据对象，它里面的默认值不会受到脚本中设置的影响 
    bulletData=ScriptableObject.CreateInstance<BulletData>();   
}
```

现在我们把刚刚创建的数据资源文件在文件夹中删掉，做个小测试，在脚本中用上 CreateInstance<>() 方法：  

![[de5e3d91f9b2b2c8ce783f010fa29e16_MD5.png]]

  
在编辑模式中，游戏还未运行时 bulletData 引用的是空数据。  
然后运行游戏，可以发现 bulletData 已经关联上了一个子弹数据资源：  

![[edf8e6b1e0a54551985d38c40c071bd1_MD5.png]]

  
这个数据资源文件是被动态创建出来的，只被创建在内存中。当我们关闭游戏后 bulletData 引用的资源重新变成空数据。

## 📕 Ⅳ. 让 ScriptableObject 真正意义上持久化

既然 ScriptableObject 本身无法在游戏打包发布后实现数据持久化，那么我们可以配合 Json，PlayerPrefs， xml ，二进制等方式来实现 ScriptableObject 真正意义上的数据持久化。

以 Json 为例：  
**利用 Json 结合 ScriptableObject 存储数据**

```
public class TestScriptableObject : MonoBehaviour
{
    public BulletData bulletData;
    void Start() {
        bulletData= ScriptableObject.CreateInstance<BulletData>();
        bulletData.speed = 9.5f;
        bulletData.damage = 100.0f;
        //将数据对象序列化为Json字符串
        string str = JsonUtility.ToJson(bulletData);
        //将数据序列化后的结果存入指定路径当中
        File.WriteAllText(Application.persistentDataPath + "/testJson.json", str);
    }

}
```

**利用 Json 结合 ScriptableObject 读取数据**

```
//从本地读取Json字符串
        string text = File.ReadAllText(Application.persistentDataPath + "/testJson.json");
        JsonUtility.FromJsonOverwrite(str, bulletData); //根据Json字符串反序列化出数据，将内容覆盖到bulletData数据文件中
```

但是实际上，让 ScriptableObject 真正意义上数据持久化有点画蛇添足的感觉。因为既然已经要用到持久化数据的方法，那为什么不全部用它们来进行数据交互呢？其实可以自定义一个数据结构类，让它与数据持久化的方法进行交互，而不是再去创建一个类继承自 ScriptableObject。

## 📕 Ⅴ. 单例模块获取数据

之前介绍使用数据文件的时候，要么是通过声明 public 变量在 Inspector 面板中进行拖拽关联，要么是使用资源加载的方法。

如果用拖拽的方式，物体之间的拖拽关系可能会随着项目量的增长而变复杂，不利用后续的维护。😟  
如果用资源加载的方式，以 Resources 为例，可能就要写大量的 Resources.Load 方法，其实是有一点重复工作的。😟

因此可以将 ScriptableObject 实例**通过单例模式化**去获取，减少重复代码，提高编码效率：

```
public class SingleScriptableObject<T> : ScriptableObject where T :ScriptableObject
{
    //所有数据资源文件都放在Resources文件夹下加载对应的数据资源文件
    //对需要复用的唯一的数据资源文件名定一个规则：文件名和类名一致
    private static string scriptableObjectPath = "ScriptableObject/"+typeof(T).Name;
    private static T instance;
    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                //如果为空，首先应该去资源路径下加载对应的数据资源文件
                instance = Resources.Load<T>(scriptableObjectPath);
            }
            //如果没有这个文件，直接创建一个数据
            if (instance == null)
            {
                instance = CreateInstance<T>();
            }
            return instance;
        }      
    }
}
```

假如说我要去调用子弹数据，我就让 BulletData 类继承自 SingleScriptableObject 类，然后直接这样调用就行了：

```
BulletData.Instance.speed
```

# 🔍 总结

那么 ScriptableObject 的相关基础知识点差不多就介绍完了。  
对于**只用不变**的数据，就适合用 ScriptableObject 做**数据配置文件**，再加上**编辑模式下可实现数据持久化**的特点，我们可以在 **Inspector 面板中进行数据的配置与调试**，有的时候是更加方便的，并且可以达到**数据复用**的目的，减少内存消耗。但是它无法在游戏打包发布后将数据的变动存储到磁盘中。因此我们要熟悉 ScriptableObject 的优缺点，结合实际需求选择使用。🌹