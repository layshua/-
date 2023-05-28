# 面向对象的三大特性  

封装 : 用程序语言来形容对象  

继承 ：复用封装对象的代码；儿子继承父亲，复用现成代码  

多态：同样行为的不同表现，儿子继承父亲基因但是有不同的行为表现  

类（class关键词）  

# 封装  

## 类和对象  

### 基本概念  

一般在namespace中声明，命名所以 首字母大小  

具相同特征，相同行为，一类事物的抽象，类是对象的模板，可以通过类创建对象。  

关键词：class  

类的声明和类对象声明是两个概念  

类的声明 类似枚举和结构体的声明 类的声明相当于是声明了一个自定义的变量类型，用来抽象现实实物的  

对象是类创建出来的 是用来表象现实中的对象个体  

对象的声明相当于声明一个指定类的变量  

类创建对象的过程 称之为实例化对象  

类 对象 都是引用类型的  

null ：空 引用类型为null的时候指的是内存堆没有分配  

```
class 类名
{
    //特征——成员变量
    //行为——成员方法
    //保护特征——成员属性
    
    //构造函数和析构函数
    //索引器
    //运算符重载
    //静态成员
}

``` 

### 实例化对象的基本语法  

用new来完成实例化  

```c#
//类名 变量名； 			 (没用分配堆内存)
//类名 变量名 = null；	 (没用分配堆内存)
//类名 变量名 = new 类名；	(在堆中新开了个房间)

namespace Les
{
    class Person
    {
        
    }

    class Machine
    {

    }
    class Program
    {
        static void Main(string[] args)
        {
            //类名 变量名； 			 (没用分配堆内存)
            //类名 变量名 = null；	 (没用分配堆内存)
            //类名 变量名 = new 类名；	(在堆中新开了个房间)
            Person p ;
            Person p2 = null;
            Person p3 = new Person; 
        }
    }
}


``` 

## 成员变量和访问修饰符  

### 成员变量  

用于描述对象的特征  

可以是任意变量类型（枚举，结构体，类对象）  

是否赋值根据需求而定  

```
class Person
{
    //特征——成员变量
    //行为——成员方法
    //保护特征——成员属性
    
    //构造函数和析构函数
    //索引器
    //运算符重载
    //静态成员
}

``` 

如果要声明一个和自己相同类型的成员变量时，不能对他进行实例化（会死循环！！）  

```
//不能这么做，会死循环
class Person
{
    Person boy = new Person();
}

``` 

### 成员变量的使用和初始值  

默认的初始值，对值类型来说都是0（bool为false），引用类型来说都是null  

default(变量); //可以得到一个变量的默认值  

```
class Person
{
    public int a;
    public float b;
    public char c;
    public bool d;
}

//--------------------------------
Person r = new Person();
//点出来使用，和结构体一样
int a =r. a;
Console.WriteLine(default(int));

``` 

### 访问修饰符  

public: 公开的，所有对象都能访问和使用  

private: 私有的，只有自己内部才能访问和使用， 变量前不写默认为该状态  

protected: 只有自己内部和子类才能访问和使用，继承的时候用到  

```
class Person
{
    public 	int a;
    private int b;
    protected int c;
}

``` 

## 成员方法  

和结构体中函数的声明使用差不多  

用来描述对象的行为,在类中声明  

受访问修饰符的影响  

不需要加static关键字  

成员方法需要实例化才能使用  

```
class Person
{
    //特征——成员变量
    public bool sex;
    public string nanme;
    public float high;
    public int age;
    public Person[] friend;
    
    //行为——成员方法
    /// /// 扩容friend数组
    /// 
    /// 
    public void AddFriend(Person p)
    {
        if (friend ==null)
        {
            friend = new Person[] { p };
        }
        else
        {
            //数组扩容+1
            Person[] newFriend = new Person[friend.Length + 1];
            for (int i = 0; i < friend.Length; i++)
            {
                newFriend[i] = friend[i];
            }
            
            //将新成员p存在新数组的最后一个索引
            newFriend[newFriend.Length - 1] = p;
        }
    }
    
    
}

``` 

##   

## 构造函数  

在实例化对象时会调用的用于初始化的函数，如果不写就默认存在一个无参构造函数  

和结构体中构造函数的写法一致，（类允许自己申明一个无参构造函数 结构体是不允许的）  

无返回值，函数名和类名必须相同，一般都是public，可以重载构造函数  

声明有参构造函数之前最好一起声明一个无参构造函数，声明有参构造时默认的无参构造就不存在了，要手动声明  

```
class Person
{
    //特征——成员变量
    public bool sex;
    public string nanme;
    public float high;
    public int age;
    public Person[] friend;

    
    //构造函数 实现对象实例化时 初始化
    //构造函数可以重载
    //无参构造函数
    public Person()
    {
        nanme = "苏同学";
        age = 18;
        sex = true;
        high = 180;
    }
    
    //有参构造函数
    public Person(string name, int age,bool sex,float high)
    {
        this.name = name;
        this.age = age;
        this.sex = sex;
        this.high = high;

    }
    
    
}

//----------------------------------------------
//在main主函数中 使用构造函数初始化对象
Person p = new Person("苏同学",18,true,180f);
Person p1 = new Person("李同学", 18, false, 171f);


``` 

特殊写法 （构造函数的继承）较少使用  

在构造函数后添加 :this(指定的重载参数)  

可以实现执行该构造函数前执行this指定的构造函数  

```
//无参构造函数
public Person()
{
    nanme = "苏同学";
    age = 18;
    sex = true;
    high = 180;
}

//有参构造函数，this指定了先执行无参
public Person(string name, int age,bool sex,float high):this()
{
    this.name = name;
    this.age = age;
    this.sex = sex;
    this.high = high;

}

``` 

## 析构函数  

当引用类型的堆内存被回收时，会调用该函数  

对于需要手动管理内存的语言（比如c++），需要在析构函数中做一些内存回收处理  

c#中有自动垃圾回收机制GC  ，所以几乎不使用析构函数   
 
```cs
class Person
{
     ~Person()
    {
    
    }
}


``` 

## 垃圾回收机制  

![](<images/1684809112253.png>)  

![](<images/1684809112302.png>)  



手动触发垃圾回收  

```
// 一般情况下，我们不会频繁调用，都是在Loading过场景时，才调用
CG.Collect();
``` 

## 成员属性  

### 基础概念  

用于保护成员变量  

为成员属性的获取和赋值添加逻辑处理  

解决3p局限性问题  

get,set可以写一个（起到保护作用）  

```
private string name;

public string Name
{
  get
  {
       //可以在返回之前添加逻辑规则
       //意味着这个属性可以获取的内容
       return name;
  }
  set
  {
      //可以在设置前添加逻辑规则
      // value 关键字 用于表示外部传入的值
      name = value;
  }
}

``` 

### 数值保护和加密处理  

 ```
       public float High
        {
            get
            {
                //可以在返回之前添加逻辑规则
                //意味着这个属性可以获取的内容
                //解密处理
                return High - 5;
            }
            set
            {
                //可以在设置前添加逻辑规则
                if (High < 0)
                {
                    high = 0;
                    Console.WriteLine("身高不能为负数，已设置为0");
                }
                // value 关键字 用于表示外部传入的值
                //加密处理
                High = value + 5;
            }
        }

``` 

### get和set前可加访问修饰符  

private  

默认不加 会使用属性声明时的访问权限  

加的访问修饰符要低于属性的访问权限  

不能让get和set的访问权限都低于属性的权限  

### 自动属性  

外部能得不能改的特征 很少使用  

## 索引器  

作用：可以以中括号的形式范围自定义类中的元素 规则自定义 访问时和数组一样  

适用于 在类中有数组变量时使用  

索引器可以重载  

锦上添花的作用，功能和成员属性相同，可以不写  

结构体也支持索引器  

## 静态成员  

static关键字修饰的成员变量，成员方法，成员属性等  

特点：不用new一个，可以直接类名点出来  

静态成员和程序同生共死  

静态函数中不能使用非静态成员（生命周期的差异）  

![](<images/1684809112385.png>)  

作用：常用变量的申明，常用的唯一的方法声明 如：同规则的数学计算相关函数  

```
class MyCalss
{
    public static float PI = 3.1415926f;
    
    public static float AreaOfCircle (float r)
    {
        return Person.PI * r * r;
    }
}

//**********************************************
float areaCircle =  MyCalss.AreaOfCircle(7);

``` 

常量和静态变量  

![](<images/1684809112452.png>)  

补充 设计模式：单例模式（线程安全相关）；  

## 静态类  

使用static关键字修饰的类  

只能包含静态成员  

不能被实例化 具有唯一性 适合用作工具类（计算公式 等）  

## 静态构造函数  

使用static关键字修饰的构造函数  

无访问修饰符 无参数 自动调用一次  

作用： 主要用于初始化静态成员、  

静态类和普通类中的静态构造函数功能一样，调用类时都会优先执行静态构造函数进行初始化  

与构造函数（针对实例对象）不同的是，静态构造函数（针对类）只执行一次，并且是在第一个实例对象创建前被调用，所以它可以用于那些只需要执行一次的操作；而且它不允许有public等修饰符，由程序自动调用，不能被外界调用。  

总结：静态构造函数用于初始化任何静态数据，或者用于执行仅需执行一次的操作；在创建第一个实例对象或者引用任何静态变量之前，将自动调用静态构造函数  

所以一般静态构造函数用来为静态成员初始化，或者作为单件模式中创建对象的唯一入口。  

![](<images/1684809112509.png>)  

## 拓展方法  

### 基本概念  

为现有非静态变量类型添加新方法  

![](<images/1684809112560.png>)  

### 基本语法  

访问修饰符 static 返回值 函数名（this 拓展类名 参数名，参数类型 参数名）  

![](<images/1684809112612.png>)  

![](<images/1684809112665.png>)  

```
public class Tools
{
    public void Fun()
    {
        Console.WriteLine("方法1");
    }

    public void Fun2()
    {
        Console.WriteLine("方法2");
    }
}

public static class TuoZhan
{
    public static void Fun3(this Tools2 value ,int a)
    {
        Console.WriteLine("方法3");
    }   
}

//***************************************************
Tools t = new Tools();
t.fun3();

``` 

### 作用  

提升程序的拓展性 为别人封装的类型写额外方法 （在一些闭源的黑盒中拓展）  

不需要继承来添加方法  

搞接口  

## 运算符重载  

用的比较少  

### 基本概念  

关键词：operator  

实现自定义类型的运算  

可以多个重载  

参数的数量 和运算符对应 （++ 单个参数 * 两个参数）  

提升程序的扩展性  

![](<images/1684809112711.png>)  

### 基本语法  

public static 返回类型 operator 运算符（参数列表）  

```
//运算符重载 实现自定义类型的运算
public static Person operator +(Person p1, Person p2)
{
    Person p = new Person();
    p.age = p1.age + p2.age;
    p.high = p1.high + p2.high;
    return p;
}

//***************************************************

Person p2 = p + p1;

``` 



## 内部类和分布类  

不常使用做了解即可  

### 内部类  

在类中再声明一个类 亲密关系的体现  

不常用  

```
class Person
{
    //人
    public int age;
    public string name;
    public Body body;
    
    public class Body
    {
        //身体
        Arm leftArm;
        public class Arm
        {
            //手臂
        }
    }
}

Person p = new Person();
Person.Body = new Body();

``` 

### 分布类  

关键字：partial  

![](<images/1684809112744.png>)  

```
partial class Student
{
    public bool sex;
    public string name;
}
partial class Student
{
    public int number;
    public float high;
}

``` 

### 分布方法  

![](<images/1684809112790.png>)  

![](<images/1684809112825.png>)  

# 继承  

## 继承的基本规则  

### 基本概念  

不能多继承 单根性  

![](<images/1684809112885.png>)  

![](<images/1684809112941.png>)  

### 基本语法  

```
class 类名 : 被继承的类名
{
    
}

```

 ```
    class Teacher
    {
        //名字
        public string name;
        //工号
        public int number;
        //介绍名字
        public void SpeakName()
        {
            Console.WriteLine(name);
        }
    }

    //继承Teacher类
    class TeachingTeacher : Teacher
    {
        //科目
        public string subject;
        //介绍科目
        public void SpeakSubject()
        {
            Console.WriteLine(subject+"老师");
        }
    }

``` 

### 访问修饰符  

关键字：protected  

不希望外部访问，子类可以访问  

子类和父类同名的成员 不建议使用！  

## 里氏替换原则  

### 基本概念  

父类容器转载子类对象  

方便进行对象存储和管理  

![](<images/1684809112982.png>)  

### 基本实现  

```
class GameObject
{
    
}

class Player : GameObject
{
    public void PlayerAtk()
    {
        Console.WriteLine("玩家攻击");
    }
}

class Moster : GameObject
{
    public void MosterAtk()
    {
        Console.WriteLine("怪物攻击");
    }
}

class Boss : GameObject
{
    public void BossAtk()
    {
        Console.WriteLine("Boss攻击");
    }
}

//****************************************************
//里氏替换原则，用父类容器转载子类对象
//但是Player类的功能无法使用要用is as转换
GameObject player = new Player();

//✅is和as
//is判断一个对象是否是指定的对象

``` 

补充：  

 ```
    class Person
    {
        public int a;
        public int b;
        public void Add ()
        {
            a = 10;
            b = 20;
        }
    }

    class Teather : Person
    {
        public int c;
        public int d;
        public void TAdd()
        {
            c = 10;
            d = 20;
        }
    }
    //********************************
    Person t = new Teather();
    //替换后直接使用是只能用父类的方法
    t.Add();
    //使用as 后才能使用子类和父类的方法
    (t as Teather).TAdd();

``` 

## 继承中的构造函数  

### 基本概念  

先执行父类的构造函数再执行子类构造函数  

子类实例化时 默认调用无参 父类没有无参构造就会报错  

![](<images/1684809113015.png>)  

1.始终保持申明一个无参构造  

2.通过base调用指定父类构造 （注意和this的区别）  

### base调用指定父类构造  

```
class Father 
{
    int a ;
    public Father (int a)
    {
        this a = a;
    }
}

class Son : Father
{
    public Son (int a) : base(a)
    {
        
    }
}

//*******************************
Son s = new Son(1);

``` 

## 万物之父&装箱拆箱  

### 基本概念  

关键词：object  

是一个基类 可以装任何东西  

![](<images/1684809113070.png>)  

### 使用方式  

引用类型和里氏替换原则一样用is和as  

值类型使用括号强转  

```
class Sb
{
    public Speak()
    {
        Console.WriteLin e("傻逼");
    }
}
//**************************
//装引用类型 和使用 ——里氏替换
object o = new Sb();
if(o is Sb)
{
    (o as Sb).Speak;
}
//装值类型 和使用 ——括号强转
object o2 = 10;
int a = (int)o2;
//装特殊类型 string
object o3 = "你好呀";
string str = o3.ToString;//也可以使用引用类型的 o3 as string
//装特殊类型 数组
object o4 = new int[10];
int[] arr = (int[])o4;//也可以使用引用类型的 o4 as int[]

``` 

### 装箱拆箱  

值类型和object之间发生的  

装箱：把值类型用引用类型存储 栈内存移到堆内存中  

拆箱：把引用类型存储的值类型取出来 堆内存移到栈内存中  

![](<images/1684809113111.png>)  

```
int a =10;
//装箱
object o = a;
//拆箱
int b = (int)o;

``` 

### 密封类  

关键字：sealed  

作用是让类无法被继承 结扎  

保证程序的规范性，安全性  

```
sealed class Father 
{
    
}


``` 

# 多态  

[

C# 多态的实现_某某人_1024的博客-CSDN博客

](https://blog.csdn.net/weiweicuit1224/article/details/106861250?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165764187616781818730775%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=165764187616781818730775&biz_id=0&utm_medium=distribute.pc_search_result.notask-blog-2~all~first_rank_ecpm_v1~pc_rank_34-2-106861250-null-null.142^v32^pc_rank_34,185^v2^tag_show&utm_term=c%23%20%E5%A4%9A%E6%80%81&spm=1018.2226.3001.4187)

参考一下捏  

## Vob  

### 基本概念  

关键词：virtual（虚函数） override（重写） base（父类）  

在执行同一方法时有不同的表现  

重写的方法 输入参数的类型和数量要一致（也复合面向对象）  

多层继承中也可以使用 层层重写回到父类  

作用：其实多态的作用就是把不同的子类对象都当作父类来看，可以屏蔽不同子类对象之间的差异，写出更通用的程序。  

![](<images/1684809113150.png>)  

![](<images/1684809113192.png>)  

### 基本语法  

 ```
    class GameObeject
    {
        //虚函数 virtual
        public virtual void Atk()
        {
            Console.WriteLine("游戏对象进行攻击");
        }
    }

    class Player : GameObeject
    {
        //重写
        public override void Atk()
        {
            //base可以保留父类的方法
            //base.Atk();
            Console.WriteLine("玩家对象进行攻击");
        }
    }

    class Monster : GameObeject
    {
        //重写
        public override void Atk()
        {
            //base可以保留父类的方法
            //base.Atk();
            Console.WriteLine("怪物对象进行攻击");
        }
    }
    
//**********************************************
    GameObeject p = new Player();
    p.Atk();
    
    GameObeject m = new Monster();

``` 

## 抽象类和抽象方法  

### 抽象类  

关键字：abstract  

不能被实例化 可以包含抽象方法 遵循里氏替换  

![](<images/1684809113242.png>)  

```
abstract class Fruits
{
    
}

``` 

如何选择普通类还是抽象类  

不希望实例化的对象 相对抽象的类可以使用 如 ：人person 水果fruit  

父类中的行为不太需要被实现 只希望子类去定义具体的规则  

整体框架设计时会使用 让基类更安全  

### 抽象函数  

又叫纯虚方法  

关键字：abstract  

特点：  

1.只能在抽象类中声明  

2.没有方法体  

3.不能是私有的  

4.继承后必须要override重写  

```
abstract class Fruits
{
    public string name;
    public abstract  void Bad ();
}

class Apple : Fruits
{
    public override void Bad ()
    {
        Console.WriteLine("苹果坏了");
    }
}
//****************************************
//遵循里氏替换 父类装子类
Fruits f = new Apple();
f.Bad();

``` 

虚方法（vritual）和纯虚方法（abstract）的区别  

虚方法可以在抽象类和非抽象类中声明 纯虚方法只能在抽象类中声明  

虚方法可以由子类选择性实现 纯虚方法必须实现重写。虚方法有方法体可实现逻辑  

他们都可以被子类用override重写 可以多层重写 子子类重写子类重写父类  

## 接口  

### 基本概念  

![](<images/1684809113277.png>)  

### 基本语法  

接口的声明  

接口时抽象行为的基类  

![](<images/1684809113314.png>)  

```
interface IFly
{
    //方法
    void Fly();
    
    //属性
    string Name
    {
        get;
        set;
    }
    
    //索引器
    int this[int index]
    {
        get;
        set;
    }
    
    //事件 c#进阶讲
    event Action doSomthing;
}

``` 

接口的使用  

![](<images/1684809113352.png>)  

```
//声明接口
interface IFly
{
    //方法
    void Fly();
    
    //属性
    string Name
    {
        get;
        set;
    }
    
    //索引器
    int this[int index]
    {
        get;
        set;
    }
    
    //事件 c#进阶讲
    event Action doSomthing;
}

//实现接口
class Animal{}

class Persom : Animal,IFly
{
    //实现的接口函数 可以加virtual再在子类中重写
    public virtual void Fly()
    {
        
    }
    
    public string Name
}
``` 

### 接口可以继承接口  

相当于行为合并  

接口继承接口时 不需要实现  

待类继承接口后 类去实现所有内容  

![](<images/1684809113438.png>)  

### 显示实现接口  

不常用  

![](<images/1684809113486.png>)  

![](<images/1684809113533.png>)  

### 接口的作用和总结  

作用  

抽象行为  

把行为抽象出来供子类继承。个别功能独立在基类外，子类需要时继承。  

提高程序复用性  

总结  

继承类：是对象间的继承，包括特征行为等。  

接口继承：是行为间的继承，继承接口的行为规范，按照规范去实现内容。  

接口也是遵循里氏替换，所以可以用接口容器装对象，  

实现装载毫无关系但是却有相同行为的对象。  

接口包含：成员方法，属性，索引器，事件，且都不实现 都没有访问修饰符  

接口继承接口相当于行为合并  

## 密封方法  

不太重要  

关键字：sealed 修饰重写函数  

让虚方法或者抽象方法不能再被重写 结扎  

和override一同出现  

# 命名空间  

## 基本概念  

是用来组织和复用代码的  

像一个工具包 类就是工具都声明在命名空间中 （文件夹装文件）  

命名空间可以分开写  

同命名空间中类的名字不可以重复，不同命名空间的类可以同名  

```
namespace Mygame
{
    class Gameobject
    {
    
    }
}

//命名空间可以分开写
namespace Mygame
{
    //属于同一命名空间 可以正常继承
    class Player:Gameobject
    {
    
    }
}

``` 

## 不同命名空间中相互使用 需引用命名空间  

两种方法 1.using关键字 2.点出来引用  

```
using System
using Mygame
//使用Mygame命名空间的Gameobject类
Gameobject g = new Gameobject();

``` ```
//使用Mygame命名空间的Gameobject类
Mygame.Gameobject g = new Mygame.Gameobject();

//也可以应对不同命名空间中 同名类的引用标识
using Mygame
using Mygame2
Mygame.Gameobject g = new Mygame.Gameobject();
Mygame2.Gameobject g2 = new Mygame2.Gameobject();

``` 

## 命名空间可以包裹命名空间  

工具包的小工具包  

```
namespace MyGame
{
    namespace UI
    {
        class Image
        {
        
        }
    }
    
    namespace Game
    {
        class Image
        {
        
        }
    }
}
//****************************************
//通过命名空间依次点出其中的类来使用
MyGame.UI.Image img = new MyGame.UI.Image();
MyGame.Game.Image img2 = new MyGame.Game.Image();
//或者引用命名空间时点出
using MyGame.UI;
using MyGame.Game;

``` 

# 万物之父中的方法  

![](<images/1684809113564.png>)  

![](<images/1684809113607.png>)  

## 静态方法：Equals  

![](<images/1684809113642.png>)  

值类型判断 对比两个对象是否相等  

引用类型判断 对比的是两个对象 是否指向同一内存地址，并不是判断是否是相同类型  

## 静态方法：ReferenceEquals  

比较两个对象是否相同的引用，主要是用来比较引用类型的对象。不能用来比值 很少用  

## 成员方法：GetType  

反射相关知识，c#进阶讲  

![](<images/1684809113686.png>)  

## 成员方法：MemberwiseClone  

![](<images/1684809113725.png>)  

浅拷贝：值类型直接复制过来，引用类型复制的是内存地址。所以改变拷贝后的值类型变量与拷贝前的值类型变量无关，改变拷贝后的引用类型变量 拷贝前的引用类型变量也会跟着改变  

## 虚方法：Equals  

可以重写该方法定义自己的比较相等  

![](<images/1684809113762.png>)  

## 虚方法：GetHashCode  

极少用  

![](<images/1684809113810.png>)  

## 虚方法：Tostring  

![](<images/1684809113846.png>)  

```
class Test
{
    public override string ToString()
    {
        return "苏老师声明的Test类"
    }
}
//*************************************
Test t = new Test();
Console.WriteLine(t.ToString);
//打印结果为"苏老师声明的Test类"

``` 

# string 的常用方法  

```
//字符串本质是char数组
string str = "苏同学";
Console.WriteLine(str[0]);
//打印结果为"苏"

//⭕转为char数组
char[] chars = str.ToCharArray();

//⭕获取字符长度
str.Length

//⭕字符串拼接
str = string.Format("{0}{1}",1,222);

//⭕正向查找字符位置 
str = "我是苏同学？";
int index = str.IndexOf("苏");
//返回 2 字符串的索引 , 找不到就会返回-1

//⭕反向查找字符位置
str = "我是苏同学苏同学？";
index = str.LastIndexOf("苏同学");
//返回 5 从后面开始查找词组就返回第一个字的索引，找不到就返回-1

//⭕移除指定位置后的字符
str = "我是苏同学苏同学";
str = str.Remove(4);
//返回 "我是苏同"

//⭕执行两个参数进行移除 参数1开始的位置 参数2字符个数
str = "我是苏同学陈同学";
str = str.Remove(3,3);
//返回"我是陈同学" 

//⭕替换指定字符串
str = "我是苏同学陈同学";
str = str.Replace("苏同学","李同学");
//返回"我是李同学陈同学" 

//⭕大小写转换
str = "abcdefg";
str = str.ToUpper();
//返回"ABCDEFG"
str = str.ToLower();
//返回"abcdefg"

//⭕字符串截取 截取从指定位置开始之后的字符串
str = "苏同学陈同学";
str = str.Substring(3);
//返回 "陈同学"
//重载 参数1开始位置 参数2指定个数
str = "苏同学陈同学苏同学";
str = str.Substring(3,3);
//返回 "陈同学"

//⭕⭕⭕字符串切割 指定切割符号来切割字符串
str = "1|2|3|4|5|6|7|8";
string[] strs = str.Split("|");
//返回 string[]{1,2,3,4,5,6,7,8}

``` 

# StringBuilder  

解决多次修改string性能消耗问题  

![](<images/1684809113889.png>)  

![](<images/1684809113941.png>)  

引用命名空间：System.Text  

关键字：StringBuilder  

容量问题 每次增加时都会自动扩容  

无法直接重新赋值 要先清空再增加  

完全具有引用类型的特征，没有值类型特征了  

```
StringBuilder strBui = new StringBuilder("123123123");
//获得容量
int a = strBui.Capacity;
//默认为16现在已经用了9 自动扩容会x2 变成32 64 128....

``` 

增删查改替换  

```
StringBuilder strBui = new StringBuilder("123123123");

//⭕增
strBui.Append("444");
//结果为  "123123123444"
strBui.AppendFormat("{0}{1}",555,666);
//结果为  "123123123444555666"

//⭕插入 参数1插入的位置 参数2插入的内容
strBui.Insert(0,"苏同学");
//结果为  "苏同学123123123444555666"

//⭕删  参数1删除开始的位置 参数2删除的个数
strBui.Remove(0,3);
//结果为  "123123123444555666"

//⭕清空
strBui.Clear();
//结果为 ""

//⭕重新赋值 先清空再增加 
strBui.Clear();
strBui.Append("苏同学");

//⭕查 和数组一样
strBui[1];
//结果为 "同"

//⭕改 和数组一样
strBui[0]='李';
//strBui结果为 "李同学"

//⭕替换 参数1被替换的字符  参数2要替换的内容
strBui.Replace("同学","老师");
//strBui结果为 "李老师"

//⭕判断是否相等
strBui.Equals("李老师");
//返回为 true

``` 

# 结构体和类的区别  

![](<images/1684809113991.png>)  

![](<images/1684809114052.png>)  

结构体可以继承接口 因为接口是行为的抽象  

![](<images/1684809114099.png>)  

# 抽象类和接口的区别  

![](<images/1684809114153.png>)  

![](<images/1684809114208.png>)  

![](<images/1684809114247.png>)  

