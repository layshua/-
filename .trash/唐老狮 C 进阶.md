# 简单的数据结构类  

数据结构学的是增删查改  

## ArrayList  

追加命名空间：System.Collections  

本质是object[ ]类（所以也可以存类）  

型的数组，可以存储任何类型数据  

![](<images/1684855239127.png>)  

### 增删查改  

### 装箱拆箱  

![](<images/1684855239210.png>)  

```
int i = 1;
array[0] = i;  //装箱 把存在栈上的变量存到堆上
i = (int)array[0] //拆箱 把存在堆上的变量存回栈上

``` 

## Stack（栈）  

追加命名空间：System.Collections  

本质是object[ ]数组  

![](<images/1684855239247.png>)  

存储规则： 先进后出  

![](<images/1684855239286.png>)  

### 增取查改  

### 装箱拆箱  

![](<images/1684855239336.png>)  

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

  

  

## 泛型接口&泛型类  

## 泛型方法  

## 泛型的作用  

  

  

  

## 泛型约束  

语法：where 泛型字母 : (约束的类型)  

约束的类型：struct，class，new()，类名，接口名，另一个泛型字母  

泛型约束可以组合使用  

多个泛型约束 用where连接  

  

泛型实现单例：  

[

c#泛型与单例_飞浪纪元[FWC–FE]的博客-CSDN博客_c# 泛型单例

](https://blog.csdn.net/weixin_38531633/article/details/108975868?utm_medium=distribute.pc_aggpage_search_result.notask-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-1-108975868-null-null.pc_agg_new_rank&utm_term=%E6%B3%9B%E5%9E%8B%E4%BD%BF%E7%94%A8%E5%8D%95%E4%BE%8B&spm=1000.2123.3001.4430)

  

# 常用泛型数据结构类  

## List  

追加命名空间：using System.Collections.Generic;  

  

### 增删查改  

## Dictionary  

追加命名空间：using System.Collections.Generic;  

与Hashtable比起来 键值对类型变为可以自己定义的泛型  

  

### 增删查改  

## 顺序存储&链式存储  

### 数据结构是什么  

  

  

### 顺序存储和链式存储的优缺点  

  

### 顺序存储  

  

  

### 链式存储  

每次添加的时候不会存在垃圾产生（对比顺序存储）  

  

  

#### 单向链表  

  

实现单向链表  

#### 双向链表  

  

#### 循环链表  

单向循环  

  

双向循环  

  

## Linkedlist  

追加命名空间：using System.Collections.Generic;  

  

  

### 增删查改  

Previous Next Value  

增：AddLast AddFirst AddAfter AddBefore  

删：RemoveFirst RemoveLast Remove Clear  

查：First Last Find  

改：Value  

## 泛型 Stack(栈) & Queue(队列)  

  

  

  

  

追加命名空间：using System.Collections.Generic;  

声明后 使用方式与 栈和队列一模一样  

# 委托&事件  

## 委托  

委托变量是函数的容器 装载传递函数的容器  

存储行为 观察者设计模式  

委托支持泛型  

  

### 基本语法  

关键字：delegate  

  

  

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

  

  

  

## 匿名函数  

关键字：delegate  

  

### 基本语法  

  

作业  

  

### 匿名函数的缺点  

添加到委托或事件容器后 不记录 无法单独移除（因为没有名字）  

## Lambda表达式  

关键字：lambda  

用法和匿名函数一样  

  

  

### 基本语法  

### 闭包  

  

委托中的方法引用了包含在他外面的函数的临时变量时 临时变量就会存进堆里  

一直都不会释放 生命周期被改变了  

临时变量被包裹进事件函数存起来了  

作业  

  

# List排序  

关键字：Sort  

系统自带的变量一般都可以直接Sort  

自定义类Sort有两种方式  

*   在类中继承接口 IComparable  
    

*   在Sort中传入委托函数  
    

## 系统默认的排序方法  

一般只能排序int float double  

## 自定义类的排序1  

通过继承IComparable接口排序  

## 自定义类的排序2  

通过委托函数进行排序  

作业  

  

# 协变逆变  

关键字：out （协变） in（逆变）  

用于修饰泛型替代符 只能修饰接口和委托中的泛型  

  

  

  

返回值和参数  

*   out修饰的泛型 只能作为返回值  
    

*   in修饰的泛型 只能作为参数  
    

结合里氏替换 少用  

*   协变：用父类泛型委托装子类泛型委托  
    

*   逆变：用子类泛型委托装父类泛型委托  
    

# 多线程  

追加命名空间：using System.Threading;  

关键字：Thread  

  

  

  

  

## 基本语法  

  

  

  

  

  

  

## 线程的作用  

改善性能 尽可能地利用 CPU 资源，充分利用 CPU ，从而提高性能。  

  

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
    

  

  

![](<images/1684855239369.png>)  

![](<images/1684855239406.png>)  

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

![](<images/1684855239449.png>)  

![](<images/1684855239499.png>)  

## 反射  

追加命名空间：using System.Reflection;  

![](<images/1684855239543.png>)  

可以在程序编译后获得信息  

![](<images/1684855239590.png>)  

### Type  

访问元数据  

![](<images/1684855239626.png>)  

### Activator  

快速实例化一个对象  

Activator.CreateInstance  

![](<images/1684855239673.png>)  

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

![](<images/1684855239707.png>)  

## 特性  

关键字：Attribute  

![](<images/1684855239744.png>)  

![](<images/1684855239786.png>)  

过时特性  

![](<images/1684855239821.png>)  

调用者信息特性  

![](<images/1684855239870.png>)  

![](<images/1684855239908.png>)  

条件编译特性  

![](<images/1684855239943.png>)  

#define Fun  

![](<images/1684855239977.png>)  

Fun();  

外部Dll包函数特性  

![](<images/1684855240014.png>)  

![](<images/1684855240060.png>)  

# 迭代器  

![](<images/1684855240105.png>)  

## 标准迭代器实现方法  

![](<images/1684855240145.png>)  

![](<images/1684855240183.png>)  

## yield return实现迭代器  

![](<images/1684855240218.png>)  

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

![](<images/1684855240258.png>)  

```
var i = 10;
var s = "你好i世界";
var array = new int[] { 1, 2, 3, 4 };

``` 

设置对象初始值  

![](<images/1684855240290.png>)  

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

![](<images/1684855240328.png>)  

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

![](<images/1684855240361.png>)  

![](<images/1684855240398.png>)  

![](<images/1684855240435.png>)  

![](<images/1684855240468.png>)  

![](<images/1684855240501.png>)  

![](<images/1684855240539.png>)  

![](<images/1684855240583.png>)  

![](<images/1684855240622.png>)  

![](<images/1684855240668.png>)  

![](<images/1684855240700.png>)  

# 排序进阶  

目前用不着  

## 插入排序  

![](<images/1684855240745.png>)  

![](<images/1684855240783.png>)  

![](<images/1684855240819.png>)  

![](<images/1684855240865.png>)  

![](<images/1684855240901.png>)  

![](<images/1684855240934.png>)  

![](<images/1684855240986.png>)  

![](<images/1684855241024.png>)  

![](<images/1684855241061.png>)  

![](<images/1684855241108.png>)  

## 希尔排序  

## 归并排序  

## 快速排序  

## 堆排序