
# 📕单例模式介绍

单例模式是程序开发中很常用的一种设计模式。有时候为了节省内存资源、保证数据内容的一致性，对某些类要求**只能创建一个实例**，这就是单例模式。在 [unity](https://so.csdn.net/so/search?q=unity&spm=1001.2101.3001.7020) 中，一些充当管理者的脚本就很适合使用单例模式，比如 UI 管理、事件管理，他们一般都是唯一存在的。

## 🔍实现思路

通常要获取类的实例，是通过 new 的方式，new 一次就获取一个类的实例，并且我们可以在任何地方 new。**但是频繁地创建实例会给内存造成很大的压力，对于有些类，我们只需用到它的一个实例就够了**，这个时候单例模式就派上了用场。那我们要怎样保证一个类只能被 new 一次呢？

前提条件：  
1️⃣ 首先要保证在 xxx 类的外部不能使用 new xxx() 的方式来获取该类的实例。我们可以给 xxx 类的构造方法私有化，这样就确保在类的外部无法不停地通过 new 创建多个对象。那么这时我们**只能从该类内部产生对象**，**该类实例的引用就可以成为该类的一个成员变量** ，等待在类的内部被实例化。（什么是类的实例的引用？比如 Person 类，Person person=new Person(); new Person() 得到该类的实例， person 是该类实例的引用）

2️⃣ 构造方法一旦私有化，外部就不可访问，只能**提供一个公共的方法来获取该类的实例**，而且这个方法不能是实例调用的方法（不能够通过创建该类的实例调用此方法），所以应该是一个**静态方法**。

3️⃣ 因为我们要通过单例类向外提供的公共静态方法来获取该类的实例，所以**该类实例的引用也要定义成静态**，才能被静态方法访问到。

因此接下来我们只用考虑如何在类的内部只实例化一次对象。

单例模式的实现分为饿汉模式与懒汉模式。  
根据刚刚介绍的思路，我们用代码来分别实现。

## 🔍饿汉模式

```
public class Singleton
{
//类一加载的时候，就创建对象，而且只会创建一次对象，这样就能保证实例唯一
    private static Singleton instance = new Singleton();
    private Singleton(){}
//我这里用静态属性替代静态方法来实现外界对该类实例的访问。 
    public static Singleton Instance
    {
        get
        {
            return instance;
        }       
    }
}
```

饿汉模式就有种 “这个类饥渴难耐地想要被实例化” 的意思。这个类被加载时，就会自动实例化这个类。它是线程安全的。不过这个可能会导致该类的实例过早地被加载出来，从而占据内存空间。

## 🔍懒汉模式

### ⭐经典版

```
public class Singleton
{
    private static Singleton instance;

    private Singleton(){}

    public static Singleton Instance
    {
        get
        {
            if (instance == null)
                instance = new Singleton();
            return instance;
        }       
    }
}
```

懒汉模式就有种 “该类的实例懒得一开始就被加载” 的意思，只有别人伸手要了，它才会出动。这个时候，只有我们第一次调用 Singleton.Instance 去获取该类的实例时才会被实例化。也就是**需要使用时才会创建实例**。  
不过这种写法并不是线程安全的。可能出现两个线程同时去获取 instance 实例，且此时 instance 仍为 null，那么就会出现两个线程分别创建了 instance，违反了单例规则。

### ⭐多线程加锁版

```
public class Singleton
{
    private static Singleton instance;
    private static readonly object locker = new object();
    private Singleton(){}

    public static Singleton Instance
    {
        get
        {
            lock (locker) {
                if (instance == null)
                    instance = new Singleton();
                return instance;
            }            
        }       
    }
}
```

通过把 locker 锁住可以挂起其他线程，保证只有当前一个线程能够执行 lock 包裹的代码块。执行完毕后，之前挂起的其他线程中又会有一个线程被唤醒，执行 lock 内的代码，但是这个时候 instance 变量已经在第一次就被实例化了，所以它会直接返回之前创建好的那个实例，这就解决了懒汉模式线程不安全的问题。  
不过这么写仍有一个弊端：我只需要在第一次创建实例时才用加锁，因为之后想获取该类的实例就直接返回创建好的那个实例了，无需再次 lock，造成性能消耗。

### ⭐加锁改进版

```
public class Singleton
{
    private static Singleton instance;
    private static readonly object locker = new object();
    private Singleton(){}

    public static Singleton Instance
    {
        get
        {
            if (instance == null)
            {
                lock (locker)
                {
                    if (instance == null)
                        instance = new Singleton();                    
                }
            }
            return instance;
        }       
    }
}
```

使用了两个条件判断语句。  
内层的 if 保证了第一次获取 instance 只能有一个线程进入[实例化对象](https://so.csdn.net/so/search?q=%E5%AE%9E%E4%BE%8B%E5%8C%96%E5%AF%B9%E8%B1%A1&spm=1001.2101.3001.7020)的语句块，其他同时进入的线程要先被锁住，等到对象实例化完毕后再释放，但这时 instance 已经被创建好了，因此不会重复创建实例，而是直接执行 return instance。  
外层的 if 保证只有第一次获取 instance 才需要加锁，因为此时还未创建实例，要避免线程安全问题。之后再次获取就直接执行 return instance，节省了加锁的性能开销。

# 📕单例基类

一个项目中可能会有多个类需要应用到单例模式，如果每个类都重复写一遍单例的思路，会显得很繁琐。**为了提高代码的复用性，我们可以把单例的思路封装成一个基类，然后让需要实现单例模式的类继承这个单例基类**。  
那基类的 instance 类型要设成什么呢？我们要保证这个类型能够适配任何类型，因此需要用上泛型来实现。

## 🔍普通单例基类

基于懒汉模式：

```
public class SingletonBase<T> where T : new()
{
    private static T instance;
    private static readonly object locker = new object();
    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                lock (locker)
                {
                    if (instance == null)
                        instance = new T();
                }
            }
            return instance;
        }
    }
}
```

几个注意点：

1.  我们要用 where T:new() 对泛型做个约束，保证是能够被实例化的类型。
2.  这里没有设置私有构造器是因为这个基类要用于继承，那么设置私有构造器就没有意义了。虽然这种单例类仍然可以通过 new 的方式不停地创建实例。但实际上我们看到一个类是单例模块时也不会闲着没事干去随便实例化。😂

## 🔍继承自 MonoBehaviour 的单例基类（手动添加到游戏对象上）

unity 中的游戏脚本要继承自 MonoBehaviour，然后可以去调用如 Start()，Update() 之类的一些生命周期函数和 MonoBehaviour 类提供的一些函数。**继承自 MonoBehaviour 的类不能通过 new 的方式去实例化**。而是**先把脚本手动拖给游戏物体或者调用 AddComponent 方法附加给游戏物体**，之后 unity 会自动帮我们实例化。所以这个时候不能使用刚刚介绍的单例基类。我们要创建一个继承于 MonoBehaviour 的单例基类，那么继承于该基类的类也会继承于 MonoBehaviour 。

类似于饿汉模式：

```
public class SingletonMonoBase<T> : MonoBehaviour where T:MonoBehaviour
{
    private static T instance;
    public static T Instance
    {
        get
        {
            return instance;
        }
    }
    protected virtual void Awake() {
        
        if (instance != null)
        {
            Destroy(gameObject);
        }       
        else
        {
            instance = this as T;
        }
        
    }
}
```

注：  
1）此单例基类的 Awake 方法要设成 **protected 虚方法**，便于它的子类去重写进行功能拓展。不过子类重写了 Awake 必须要在方法的第一行调用 base.Awake()，否则会把基类 Awake 的单例功能覆盖掉。  
2）T 的约束不再是一个能实例化的类，而是 MonoBehaviour 或者 SingletonMonoBase  
3）当 instance 不为空时销毁游戏物体是为了保证一个继承于此单例基类的脚本无法挂在多个游戏物体上，保证当前场景只有唯一的这个脚本。不过我们在手动添加脚本时也要注意**不能将一个单例模块加到多个物体上**，否则可能会销毁我们原本不想销毁的物体。  
4）这里用类似饿汉模式的写法是便于将 “判断一个单例脚本有没挂载到多个物体上” 的逻辑放到脚本生命周期函数 Awake 里执行，直接通过**判断实例有无重复来保证在脚本初始化时就生成单例**。如果用懒汉模式 “有需则用” 的思想不大好保证一个单例脚本只能挂在一个物体上，就算有实现方法（比如查找场景中有几个这种脚本）也肯定没有饿汉模式来得简单。

继承于此单例基类的脚本示例：

```
//虽然这个类没继承 MonoBehaviour ，但是它的父类继承了
public class SingleTest : SingletonMonoBase<SingleTest> {
//如果要扩展父类的 Awake 方法要这么写
    protected override void Awake() {
        base.Awake();      
    }
    public void Test() {
        print(gameObject.name);
    }
}
```

用于调用单例的脚本（此脚本我挂到了一个名叫 Manager 的物体上：

```
public class GameController : MonoBehaviour
{
    void Start() {
        SingleTest.Instance.Test();
    }
}
```

注：这里我把逻辑写在了 Start 方法里而不是 Awake 是因为 unity 脚本的执行顺序是随机的，可能 GameController 的 Awake 比 SingleTest 的 Awake 先执行，那么这个时候 **SingleTest 的实例还未初始化**，直接调用它的方法会报空引用异常！！！虽然可以手动在 Edit-> Project Settings->Script Execution Order 中调整脚本执行顺序，不过显然会比较麻烦！

**一般来说，一个单例模块会挂在一个空物体上。** 我把单例脚本挂在一个 GameObject 上，然后运行程序：  

![[57d7c082e2c1d3d5a0ceea069468dcc3_MD5.png]]

![[acbcc4d9edd9dff2068e043427b36f2d_MD5.png]]

但是如果我不小心将这个脚本挂到两个物体上：  

![[2e2e1e6b7fe027d69393979f10db9e11_MD5.png]]

  
运行结果：  

![[954298b2e6555e2abfbfbfea8fe2a437_MD5.png]]

可以看到同样挂载 SingletTest 脚本的 GameObject 物体被销毁了。这是因为此时 Main Camera 中的 SingleTest 先执行了 Awake ，创建了一个单例。然后等到 GameObject 中的 SingleTest 执行 Awake 时，因为 instance 已经存在，所以会销毁脚本挂载的游戏物体。尽管此时场景中确实只有一个 SingleTest 实例，但我们的本意是让这个单例脚本挂在我们创建的一个空物体 GameObject 上。因此，**继承于这种单例基类的脚本需要我们人为地保证只有一个游戏物体挂载了这个脚本**。

不过一般来说，充当管理者的脚本要作用于全局，也就是切换场景时这个脚本不能被销毁，脚本所挂载的物体不能被销毁。如果要实现这个需求，需要对继承于此单例基类的脚本进行修改，比如：

```
public class SingleTest : SingletonMonoBase<SingleTest> {
    protected override void Awake() {
        base.Awake();
        DontDestroyOnLoad(gameObject);
    }
   public void Test() {
        print(gameObject.name);
    }
}
```

如果在你的游戏中，所有的单例模块全是无法销毁的管理者，你甚至可以把 DontDestroyOnLoad 写在单例基类的 Awake 方法中，提高代码复用性。

但是这种手动添加到游戏对象上的单例个人感觉不是很好用。原因如下：  
1）手动添加可能会因疏忽而出错，比如不小心把单例模块挂到多个物体上。  
2）可能得考虑脚本执行顺序（Awake 顺序）的问题。

## 🔍继承自 MonoBehaviour 的单例基类（自动添加到游戏对象上）

调用此单例基类的子类时会将子类的脚本挂在自动创建的物体上，无需我们手动挂载。  
基于懒汉模式：

```
public class SingletonAutoMonoBase<T> : MonoBehaviour where T:MonoBehaviour
{
    private static T instance;
    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                GameObject obj = new GameObject();
                obj.name = typeof(T).ToString();
                DontDestroyOnLoad(obj);//这行代码根据实际需求决定是否写在基类中
                instance = obj.AddComponent<T>();
            }
            return instance;
        }
    }
}
```

另外，我把物体无法销毁写在了基类中，如果在实际项目中有些单例模块需要过场景销毁，那么 DontDestroyOnLoad 要写在每个子类里。  
创建一个脚本继承此单例基类：

```
public class AutoSingleTest : SingletonAutoMonoBase<AutoSingleTest>
{    
    public void Test() {
        print(gameObject.name);
    }
}
```

然后在另一个脚本去获取 instance （此脚本需要手动添加到游戏对象中）：

```
public class GameController : MonoBehaviour
{
    void Start() {
        AutoSingleTest.Instance.Test();
    }
}
```

![[1f35edcd368e1c8c5493437d49f5af78_MD5.png]]

未运行程序前的面板：  

![[8a05138da9e0c699ff314a2b5f001d1d_MD5.png]]

接下来运行程序：  

![[09343734f29d64ebe1a578b5995a492f_MD5.png]]

  
程序自动帮我们创建了一个不可销毁游戏物体 AutoSingleTest ，并且挂载了 AutoSingleTest 脚本。不像前一种单例基类，还要手动把单例基类的子类挂到一个物体上。而且这种实现方式还保证了一个单例脚本只挂载到一个物体上。

以上便是三种单例基类的用法，具体用哪个根据实际需求来定。对于那些管理器对象，一般使用普通的单例基类就能满足需求。希望本篇博客大家有所帮助！🌹