# 简单的数据结构类  

数据结构学的是增删查改  

## ArrayList  

追加命名空间：System.Collections  

本质是object[ ]类（所以也可以存类）  

型的数组，可以存储任何类型数据  

![](<images/1684855295542.png>)  

### 增删查改  

### 装箱拆箱  

![](<images/1684855295573.png>)  

```
int i = 1;
array[0] = i;  //装箱 把存在栈上的变量存到堆上
i = (int)array[0] //拆箱 把存在堆上的变量存回栈上

``` 

## Stack（栈）  

追加命名空间：System.Collections  

本质是object[ ]数组  

![](<images/1684855295621.png>)  

存储规则： 先进后出  

![](<images/1684855295663.png>)  

### 增取查改  

### 装箱拆箱  

![](<images/1684855295696.png>)  

## Queue（队列）  

追加命名空间：System.Collections  

本质是object[ ] 数组  

与Stack（栈）的概念非常相似，一个是先进后出 一个是先进先出  

  

先进先出  

  

### 增取查改  

### 装箱拆箱  

  

###   

## Hashtalbe  

追加命名空间：System.Collections  

通过键去找值  

  

键值对 一个key（键）对应一个value（值）  

key1------>value1  

key2------>value2  

key3------>value3  

### 增删查改  

### 装箱拆箱  

# 泛型  

![](<images/1684855295732.png>)  

![](<images/1684855295787.png>)  

## 泛型接口&泛型类  

```
class MyClass {
    public T value;
}
//泛型占位字母有多个
class MyClass2 {
    public T1 value1;
    public T2 value2;
    public K value3;
    public M value4;
}

//****************主函数***********************
MyClass t = new MyClass();
t.value = 10;
MyClass t2 = new MyClass();
t2.value = "参数";

MyClass2 > 
t3 = new MyClass2>(); 
``` 

## 泛型方法  

```
//普通类中泛型方法
class MyClass4
{
    public void MyFun (T value)
    {
        Console.WriteLine(value);
    }
}
//****************主函数***********************
MyClass4 t4 = new MyClass4();
t4.MyFun("你好"); 
``` 

## 泛型的作用  

![](<images/1684855295835.png>)  

![](<images/1684855295885.png>)  

![](<images/1684855295928.png>)  

## 泛型约束  

语法：where 泛型字母 : (约束的类型)  

约束的类型：struct，class，new()，类名，接口名，另一个泛型字母  

泛型约束可以组合使用  

多个泛型约束 用where连接  

![](<images/1684855295961.png>)  

泛型实现单例：  

[

c#泛型与单例_飞浪纪元[FWC–FE]的博客-CSDN博客_c# 泛型单例

](https://blog.csdn.net/weixin_38531633/article/details/108975868?utm_medium=distribute.pc_aggpage_search_result.notask-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-1-108975868-null-null.pc_agg_new_rank&utm_term=%E6%B3%9B%E5%9E%8B%E4%BD%BF%E7%94%A8%E5%8D%95%E4%BE%8B&spm=1000.2123.3001.4430)

  

# 常用泛型数据结构类  

## List  

追加命名空间：using System.Collections.Generic;  

![](<images/1684855296007.png>)  

### 增删查改  

## Dictionary  

追加命名空间：using System.Collections.Generic;  

与Hashtable比起来 键值对类型变为可以自己定义的泛型  

![](<images/1684855296050.png>)  

### 增删查改  

## 顺序存储&链式存储  

### 数据结构是什么  

![](<images/1684855296091.png>)  

![](<images/1684855296133.png>)  

### 顺序存储和链式存储的优缺点  

![](<images/1684855296165.png>)  

### 顺序存储  

![](<images/1684855296201.png>)  

![](<images/1684855296235.png>)  

### 链式存储  

每次添加的时候不会存在垃圾产生（对比顺序存储）  

![](<images/1684855296267.png>)  

![](<images/1684855296301.png>)  

#### 单向链表  

![](<images/1684855296339.png>)  

实现单向链表  

#### 双向链表  

![](<images/1684855296380.png>)  

#### 循环链表  

单向循环  

![](<images/1684855296415.png>)  

双向循环  

![](<images/1684855296454.png>)  

## Linkedlist  

追加命名空间：using System.Collections.Generic;  

![](<images/1684855296496.png>)  

![](<images/1684855296572.png>)  

### 增删查改  

Previous Next Value  

增：AddLast AddFirst AddAfter AddBefore  

删：RemoveFirst RemoveLast Remove Clear  

查：First Last Find  

改：Value  

## 泛型 Stack(栈) & Queue(队列)  

![](<images/1684855296608.png>)  

![](<images/1684855296641.png>)  

![](<images/1684855296673.png>)  

![](<images/1684855296705.png>)  

追加命名空间：using System.Collections.Generic;  

声明后 使用方式与 栈和队列一模一样  

# 委托&事件  

## 委托  

委托变量是函数的容器 装载传递函数的容器  

存储行为 观察者设计模式  

委托支持泛型  

![](<images/1684855296737.png>)  

### 基本语法  

关键字：delegate  

![](<images/1684855296779.png>)  

![](<images/1684855296818.png>)  

扩展  

### 多播委托  

使用+=运算符 为委托新增方法  

使用-=运算符 为委托移除方法  

使用=null 为委托清空方法  

### 系统常用的委托  

Action 无参无返  

Action<,> 有参无返 可以传多个参数  

Func<> 无参有返  

Func<,> 有参有反 可以传多个参数 最后一个是返回值  

## 事件  

关键字：event  

事件相当于对委托进行了一次封装 让其不能被外部置空和调用 更安全  

![](<images/1684855296870.png>)  

![](<images/1684855296909.png>)  

![](<images/1684855296954.png>)  

## 匿名函数  

关键字：delegate  

![](<images/1684855296990.png>)  

### 基本语法  

![](<images/1684855297022.png>)  

作业  

![](<images/1684855297060.png>)  

### 匿名函数的缺点  

添加到委托或事件容器后 不记录 无法单独移除（因为没有名字）  

## Lambda表达式  

关键字：lambda  

用法和匿名函数一样  

![](<images/1684855297103.png>)  

![](<images/1684855297134.png>)  

### 基本语法  

### 闭包  

![](<images/1684855297166.png>)  

委托中的方法引用了包含在他外面的函数的临时变量时 临时变量就会存进堆里  

一直都不会释放 生命周期被改变了  

临时变量被包裹进事件函数存起来了  

作业  

![](<images/1684855297209.png>)  

# List排序  

关键字：Sort  

系统自带的变量一般都可以直接Sort  

自定义类Sort有两种方式  

*   在类中继承接口 IComparable  
    

*   在Sort中传入委托函数  
    

## 系统默认的排序方法  

一般只能排序int float double  

```
List list = new List();
list.Add(4);
list.Add(3);
list.Add(5);
list.Add(2);
list.Add(1);
list.Add(6);

//List提供的排序方法
list.Sort();
for (int i = 0; i < list.Count; i++)
{
    Console.WriteLine(list[i]);
}
//打印结果为 1 2 3 4 5 6
//ArrayList中也有Sort排序方法 
``` 

## 自定义类的排序1  

通过继承IComparable接口排序  

## 自定义类的排序2  

通过委托函数进行排序  

作业  

![](<images/1684855297241.png>)  

# 协变逆变  

关键字：out （协变） in（逆变）  

用于修饰泛型替代符 只能修饰接口和委托中的泛型  

![](<images/1684855297276.png>)  

![](<images/1684855297312.png>)  

![](<images/1684855297344.png>)  

返回值和参数  

*   out修饰的泛型 只能作为返回值  
    

*   in修饰的泛型 只能作为参数  
    

```
//⭕out修饰的泛型 只能作为返回值
delegate T TestOut();
//⭕in修饰的泛型 只能作为参数
delegate void TestIn(T t); 
``` 

结合里氏替换 少用  

*   协变：用父类泛型委托装子类泛型委托  
    

*   逆变：用子类泛型委托装父类泛型委托  
    

# 多线程  

追加命名空间：using System.Threading;  

关键字：Thread  

![](<images/1684855297379.png>)  

![](<images/1684855297413.png>)  

![](<images/1684855297444.png>)  

![](<images/1684855297478.png>)  

## 基本语法  

![](<images/1684855297516.png>)  

![](<images/1684855297550.png>)  

![](<images/1684855297592.png>)  

![](<images/1684855297633.png>)  

![](<images/1684855297667.png>)  

```
static bool isRuning = true;

static void Main(string[] args)
{
    //1.声明一个新线程
    //新线程要执行的逻辑封装在一个函数中传入
    Thread t = new Thread(NewThreadLogic);

    //2.启动线程
    t.Start();

    //3.设置为后台线程
    t.IsBackground = true;

    //4.关闭线程如果不是死循环不用刻意管 t=null
    //死循环中bool标识
    Console.ReadKey();
    isRuning = false;
    Console.ReadKey();
    //通过提供的方法 控制台这个版本会报错
    //t.Abort();

    //5.线程休眠
    //让当前线程(Main) 休眠 毫秒
    //Thread.Sleep(1000);


}

static void NewThreadLogic ()
{
    while (isRuning)
    {
        //5.线程休眠
        //让当前线程(Main) 休眠 毫秒
        Thread.Sleep(1000);

``` 

![](<images/1684855297705.png>)  

```
static object obj = new object();

static void Main(string[] args)
{
    //1.声明一个新线程
    Thread t = new Thread(NewThreadLogic);
    //2.启动线程
    t.Start();
    //3.设置为后台线程
    t.IsBackground = true;

    while (true)
    {
        lock(obj)
        {
            Console.SetCursorPosition(0, 0);
            Console.ForegroundColor = ConsoleColor.Red;
            Console.Write("▲");
        }
    }
}

static void NewThreadLogic ()
{
    while (true)
    {
        lock (obj)
        {
            Console.SetCursorPosition(10, 5);
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.Write("■");
        }
    }
}

``` 

## 线程的作用  

改善性能 尽可能地利用 CPU 资源，充分利用 CPU ，从而提高性能。  

![](<images/1684855297743.png>)  

# 预处理器指令  

在实际编译之前开始对信息进行预处理  

在unity中会进行一些版本或平台的判断  

关键字：  

*   #define  
    

*   #undef  
    

*   #region #endregion  
    

*   #if  
    

*   #elif  
    

*   #else  
    

*   #endif  
    

![](<images/1684855297777.png>)  

![](<images/1684855297818.png>)  

![](<images/1684855297860.png>)  

![](<images/1684855297893.png>)  

```
//定义一个符号
#define Unity4
#define IOS
#define Android
//取消定义一个符号
#undef Android

//折叠代码
#region 命名空间
using System;
using System.Threading;
#endregion

namespace Lesson_预处理器指令
{
    class Program
    {
        static void Main(string[] args)
        {
//如果定义了Unity4和IOS则执行以下代码
#if Unity4 && IOS
            Console.WriteLine("版本为Unity4 IOS");
            //警告
            #warning 这个版本不合法
            //报错
            #error 这个版本不准执行
#endif
        }
    }
}

``` 

# 反射&特性  

![](<images/1684855297926.png>)  

![](<images/1684855297957.png>)  

## 反射  

追加命名空间：using System.Reflection;  

![](<images/1684855297989.png>)  

可以在程序编译后获得信息  

![](<images/1684855298037.png>)  

### Type  

访问元数据  

![](<images/1684855298072.png>)  

### Activator  

快速实例化一个对象  

Activator.CreateInstance  

![](<images/1684855298114.png>)  

```
using System;
using System.Reflection;

namespace Lesson_13_反射和特性
{
    class Test
    {…}

    class Program
    {
        static void Main(string[] args)
        {
            //Activator
            //用于将Type对象快捷实例化为对象
            Type testType = typeof(Test);
            //1.无参构造
            Test testObj = Activator.CreateInstance(testType) as Test;
            Console.WriteLine(testObj.str);
            //2.有参构造
            testObj = Activator.CreateInstance(testType, 99) as Test;

            testObj = Activator.CreateInstance(testType, 888,"你好你好") as Test;
            Console.WriteLine(testObj.str);
        }
    }
}


``` 

### Assembly  

加载程序集  

Assembly.LoadFrom  

![](<images/1684855298156.png>)  

## 特性  

关键字：Attribute  

![](<images/1684855298198.png>)  

![](<images/1684855298233.png>)  

过时特性  

![](<images/1684855298266.png>)  

调用者信息特性  

![](<images/1684855298301.png>)  

![](<images/1684855298338.png>)  

条件编译特性  

![](<images/1684855298369.png>)  

#define Fun  

![](<images/1684855298404.png>)  

Fun();  

外部Dll包函数特性  

![](<images/1684855298437.png>)  

![](<images/1684855298486.png>)  

# 迭代器  

![](<images/1684855298521.png>)  

## 标准迭代器实现方法  

![](<images/1684855298567.png>)  

![](<images/1684855298609.png>)  

## yield return实现迭代器  

![](<images/1684855298648.png>)  

```
//yield return实现迭代器
class CustomList2 : IEnumerable
{
    private int[] list;
    public CustomList2()
    {
        list = new int[] { 1, 2, 3, 4, 5, 6, 7, 8 };
    }
    public IEnumerator GetEnumerator()
    {
        for (int i = 0; i < list.Length; i++)
        {
            //yield关键词 配合迭代器使用
            //暂时返回保留当前的状态
            yield return list[i];
        }
    }
}

class Program
{
    static void Main(string[] args)
    {
        CustomList2 customList2 = new CustomList2();
        foreach (int item in customList2)
        {
            Console.WriteLine(item);
        }
            
    }
}


``` 

yield return为泛型类实现迭代器  

```
//yield return为泛型类实现迭代器
class CustomList3 : IEnumerable
{
    private T[] array;
    public CustomList3(params T[] array)
    {
        this.array = array;
    }
    public IEnumerator GetEnumerator()
    {
        for (int i = 0; i < array.Length; i++)
        {
            yield return array[i];
        }
    }
}

class Program
{
    static void Main(string[] args)
    {
        CustomList3 customList3 = 
            new CustomList3("你","好","世","界");
        foreach (string item in customList3)
        {
            Console.WriteLine(item);
        }
            
    }
} 
``` 

# 特殊语法  

var隐式类型  

![](<images/1684855298692.png>)  

```
var i = 10;
var s = "你好i世界";
var array = new int[] { 1, 2, 3, 4 };

``` 

设置对象初始值  

![](<images/1684855298724.png>)  

```
class Person
{
    private int money;
    public bool sex;
    public string Name
    {…}
    public int Age
    {…}
    public Person(int money)
    {…}
}
//*******************************
//声明对象时 直接写大括号初始化公共成员变量和属性
Person person = 
    new Person (99) { sex = true, Name = "苏同学", Age = 20 };

``` 

设置集合的初始值  

![](<images/1684855298758.png>)  

```
//声明集合对象时 直接通过大括号初始化内部属性
List listInt = new List() { 1, 2, 3, 4, 5, 6 };
//套娃上一个设置对象初始值
List listPerson = new List()
{
    new Person(100),
    new Person(230){ Age=10},
    new Person(999){Name="苏同学",sex=true}
};
//字典也可以
Dictionary dic = new Dictionary()
{
    { 1,"你"},
    { 2,"好"},
    { 3,"世"},
    { 4,"界"}
}; 
``` 

匿名类型  

var 变量可以声明为自定义的匿名类  

匿名类中只能有成员变量，不能有函数  

```
//匿名类型
//var 变量可以声明为自定义的匿名类型
var v = new { name="苏同学", money = 999 };
Console.WriteLine(v.money);

``` 

可空类型  

```
//1.值类型不能赋值为空
//int c = null;
//2.声明时在值类型后面加 ? 可赋值为空
int? c = null;
//3.判断是否为空
if (c.HasValue)
{
    Console.WriteLine(c);
    Console.WriteLine(c.Value);
}
//4.安全获取可空类型值
//如果为空 返回值类型的默认值
Console.WriteLine(c.GetValueOrDefault());
//如果为空 返回指定默认值
Console.WriteLine(c.GetValueOrDefault(100));

//⭕引用类型的特殊用法
object o = null;
if(o != null)
{
    o.ToString();
}
//等价上面的简便写法
o?.ToString();

//⭕委托的特殊用法
Action action = null;
if(action != null)
{
    action();
}
//等价上面的简便写法
action?.Invoke();

``` 

空合并操作符  

```
//空合并操作符
//左边值 ?? 右边值
//如果左边值为null 就返回右边值 否则返回左边值
int? intV = null;
int intI = intV == null ? 100 : intV.Value;
//等价上面的简便写法
intI = intV ?? 100;

``` 

内插字符串  

```
//关键字: $
//用$来构造字符串 让字符串可以拼接变量
string name = "苏同学";
int age =18;
Console.WriteLine($"昏昏欲睡{name},今年{age}");

``` 

单据逻辑简略写法  

逻辑中只有单行代码 才能省去大括号  

```
//单句逻辑的简略写法
if(true) Console.WriteLine("1111111111");
for (int j=0; j<10; j++) Console.WriteLine("1111111111");

``` ```
//在属性中的单据简略写法
class Test
{
    private string name = "";
    public string Name
    {
        get => name;
        set => name=value;
    }
}

``` ```
//在函数中的简略写法
public int Add (int x,int y) => x+y;
public void Speek (string str) => Console.WriteLine(str);

``` 

# 值类型和引用类型  

![](<images/1684855298792.png>)  

![](<images/1684855298837.png>)  

![](<images/1684855298869.png>)  

![](<images/1684855298900.png>)  

![](<images/1684855298932.png>)  

![](<images/1684855298972.png>)  

![](<images/1684855299007.png>)  

![](<images/1684855299051.png>)  

![](<images/1684855299087.png>)  

![](<images/1684855299120.png>)  

# 排序进阶  

目前用不着  

## 插入排序  

![](<images/1684855299152.png>)  

![](<images/1684855299188.png>)  

![](<images/1684855299221.png>)  

![](<images/1684855299255.png>)  

![](<images/1684855299288.png>)  

![](<images/1684855299320.png>)  

![](<images/1684855299358.png>)  

![](<images/1684855299401.png>)  

![](<images/1684855299444.png>)  

![](<images/1684855299476.png>)  

## 希尔排序  

## 归并排序  

## 快速排序  

## 堆排序