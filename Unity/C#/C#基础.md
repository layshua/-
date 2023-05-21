

---
title: C#基础
aliases: []
tags: []
create_time: 2023-05-21 21:50
uid: 202305212150
cssclass: academia, academia-rounded
banner: "![[Pasted image 20230521215136.png]]"
---


# 入门
## 打印输入输出
```C#
Console.Write("xxx"); // 打印，不自动空行
Console.WriteLine("xxx"); // 打印，自动空行
string text = Console.ReadLine(); //接收输入，按回车结束
Console.ReadKey();  // 检测是否按键，只要按了就结束
```
## 3. 转义符

2.1 转义符
①换行\n：

```C#
Console.WriteLine("我爱你\n爱着你")
```

(windous操作系统只认识\r\n，不认识\n)
②英文半角双引号：\"\"  中文半角可以直接打印出
③一个Tab空格:\t
④退格：\b
⑤\:\\   
⑥@作用：
1.取消转义符的作用（用来存路径）

```C#
string a = @"C：\mycode\a\文件.txt";
```

2.将字符串按照原格式输出

## 4. 类型转换

我们要求等号两边参与运算的操作数必须一致，如果不一致，满足下列条件会发生转换。

### **隐式类型转换**

①两种类型兼容

例如：int和double兼容（都是数字类型）

②目标类型大于原类型

例如：double>int 小的转大的

```C#
int->double//隐式类型转换
int number = 10；
double d = number；  
```

### **显式类型转换**

①两种类型兼容

②大的转成小的

语法：（待转换的类型）要转换的值

```C#
double->int //强制类型转换（显式类型转换）
double a =(int)b;

//如果n1/n2有一个是double类型则整个式子提升为double类型：
//全int：输出结果d=3
int n1 = 10;
int n2 = 3;
double d = n1 / n2;
Console.WriteLine(d);

//将n1改为double类型：输出d=3.333...
double d = n1*1.0 / n2;  //n1*1.0将n1转换为double类型
Console.WriteLine(d);
```

### **保留指定小数位数**

```c#
语法：
变量：0.00  //保留两位
double n1 = 3.33333;
Console.WriteLine($"{n1:0.00}");

输出：3.33
```

### Convert类型转换

如果两个变量类型不兼容，比如string与int或string与double，可以使用Convert函数进行转换。

Convert.ToInt32()

Convert.ToDouble()

![image-20220622124705460](image-20220622124705460.png)

```C# string a = "123";
string a = "123";  //但是只能转换数字，如果a=”123abc“则会异常
int b = Convert.ToInt32(a);
//double b = Convert.ToDouble(a);
//Convert.ToDouble(b);
Console.WriteLine(b);
```

### .Parse类型转换

效果同Convert:

int.Parse()

double.Parse()

```C#
string a = "123"; 
int b = int.Parse(a);
//double b = double.Parse(a)
```

### .TryParse类型转换

int.TryParse()

```C#
int a = 0;
bool b = int.TryParse("123", out a);
//尝试将”123“转换为int类型，如果成功就将转换后的123赋值给a，并返回true给b。如果失败则返回false给b，a赋值为0。
Console.WriteLine(b);
Console.WriteLine(a);
```



## 5. 异常捕获

增加代码健壮性：哪行代码有可能出现异常，就try它。

语法：

```C#
try
{
    可能出现异常的代码；
}
catch
{
    出现异常后要执行的代码；
}
如果无异常，则catch内代码不会执行。如果出现异常，则后续代码不再执行，而是跳到catch的代码。（try-catch中间不能有其他代码）
```

## 6. 断点调试

 1)、写完一段程序后，想看一下这段程序的执行过程。
2)、当你写完这段程序后，发现，程序并没有按照你想象的样子去执行。

调试方法：
1)F11逐语句调试(单步调试)
2)F10逐过程调试
3)断点调试：

程序运行到断点处，就不再向下执行了。

设置断点后，先F5执行，然后再从断点处F11逐句调试

## 7. 特殊类型

### 随机数

```C#
//1. 创建能够产生随机数的对象
Random r = new Random();
//2. 让产生随机数的这个对象调用方法来产生随机数
int rnumber = r.Next(1, 10);  //确定范围，这是个[ )的区间
Console.WriteLine(rnumber);
```

### 常量

```C#
const 变量类型 变量名 = 值
const int a = 1；
```

### 枚举

用于规范开发流程

```C#
public enum 枚举名
{
    值1，
    值2，
    值3，
    ......
    值n  //最后一个逗号可加可不加
}

public：访问修饰符
enum：声明枚举的关键字
枚举名：要符合Pascal命名规范 首字母大写
```

将枚举声明到命名空间的下面，类的外面，表示这个命名空间下，所有的类都可以使用这个枚举。

[枚举类型的数据类型转换](https://www.bilibili.com/video/BV1FJ411W7e5?p=66&spm_id_from=pageDriver&vd_source=9d1c0e05a6ea12167d6e82752c7bc22a)

枚举就是一个变量类型 ，int--double  string  decimal.
只是枚举声明、赋值、使用的方式跟那些普通的变量类型不一样。

我们可以将一个枚举类型的变量跟int类型和string类型互相转换。
枚举类型默认是跟int类型相互兼容的，所以可以通过强制类型转换的语法互相转换。
当转换一个枚举中没有的值的时候，不会抛异常，而是直接将数字显示出来。

枚举同样也可以跟string类型互相转换，如果将枚举类型转换成string类型，则直接调用ToString().
如果将字符串转换成枚举类型则需要下面这样一行代码：
(要转换的枚举类型)Enum.Parse(typeof(要转换的枚举类型),"要转换的字符串");
如果转换的字符串是数字，则就算枚举中没有，也会不会抛异常。
如果转换的字符串是文本，如果枚举中没有，则会抛出异常。

### 数组

```C#
//数组的声明方式（主要掌握这两种）
//数组类型[] 数组名 = new 数组类型[长度]
int[] nums = new int[10];
int[] nums = {1,2,3,4,5};

//其他：
//int[] nums = new int[3]{1,2,3};
//int[] nums = new int[]{1,2,3,4,5};
```



## 8. 结构

```C#
 public struct Person
{
    public string _name;  //字段前要加_，用来区分变量
    public int _age;
    public char _gender;
}
```

## 9. 方法（函数）

```C#
pubilc static 返回值类型 方法名（参数列表）
{
	方法体;
}
public：访问修饰符
static：静态的
返回值类型：如果不需要写返回值，写void
方法名：Pascal 每个单词首字母都大写
参数列表：完成这个方法所必须要提供给这个方法的条件
    
 public static int GetMax(int n1, int n2)
{
    return n1> n2 ? n1 : n2;
}
```

### 调用

```C#
namespcae 调用问题
{
    class Program
    {
        //使用静态字段来模拟全局变量
        public static int _number = 10;
     
        ___________________________________  
        //报错
        static void Main(string[] args)
        {
            int a = 3;//不是全局变量，所有Test（）中的a不是这个a
            Test();
            Console.WriteLine(a);
        }

        public static int Test()
        {
            a = a + 5;  
        }

        //正确写法：
        static void Main(string[] args)
        {
            int a = 3;
            Test(a);
            Console.WriteLine(a);
        }

        public static int Test(int n)
        {
            a = a + 5;
            return a
        }
    }
}
```

我们在Main()函数中，调用Test()函数，我们管Main()函数称之为调用者，管Test()函数称之为被调用者。
**如果被调用者想要得到调用者的值：**
**1)、传递参数。**
**2)、使用静态字段来模拟全局变量。**
如果调用者想要得到被调用者的值：
1)、返回值

2）、不管是实参还是形参，都是在内存中开辟了空间的。



### out参数

如果你在一个方法中，返回多个相同类型的值的时候，可以考虑返回一个数组。
但是，如果返回多个不同类型的值的时候，返回数组就不行了，那么这个时候，
我们可以考虑使用out参数。**out参数就侧重于在一个方法中可以返回多个不同类型的值。**

```C#
public static void Test(int[]nums,out int max.out int min,out int sum,out float avr)
//out int max写到形参列表中
{
    max = nums[0];
    min = nums[1];
    sum = 0;
    avr = sum / nums.Length;
}

外部调用：
Test（nums,out max,out min,out sun,out avr）;
```



### ref参数（引用Reference）

```C#
 static void Main(string[] args)
{
    double salary = 5000;
    JiangJin(ref salary);
    Console.WriteLine(salary);
}

public static void JiangJin(ref double s)
{
    s += 500;  //即使不返回值也能改变salary
}
```

能够将一个变量带入一个方法中进行改变，改变完成后，再讲改变后的值带出方法。
ref参数要求在方法外必须为其赋值，而方法内可以不赋值。

### params可变参数

**将实参列表中跟可变参数数组类型一致的元素都当做数组的元素去处理。**
params可变参数**必须是形参列表中的最后一个元素。**

```C#
 static void Main(string[] args)
{
    int[] s = { 100, 80, 95 };
    Test("张三", s);   //主要区别再第二个参数这里
    Console.ReadKey();
}

public static void Test (string name, int[] score)
{
    int sum = 0;
    for(int i=0;i<score.Length;i++)
    {
        sum += score[i];
    }
    Console.WriteLine($"{name}这次考试总成绩是{sum}");
}

//改用params后：
 static void Main(string[] args)
{
   // int[] s = { 100, 80, 95 };
    Test("张三", 100, 80, 95); //可变长度，可以增加其他成绩
    Console.ReadKey();
}

public static void Test (string name, params int[] score)
{
    int sum = 0;
    for(int i=0;i<score.Length;i++)
    {
        sum += score[i];
    }
    Console.WriteLine($"{name}这次考试总成绩是{sum}");
}
```



### 重载

概念：方法的重载指的就是**方法的名称相同，但是参数不同**。
**参数不同，分为两种情况**
**1)、如果参数的个数相同，那么参数的类型就不能相同。**
**2)、如果参数的类型相同，那么参数的个数就不能相同。**
方法的重载跟返回值没有关系。

```C#
public static void M (int n1,int n2)
{
    int result = n1 + n2;
}

public static double M(double d1,double d2)
{
    return d1 + d2;
}

public static void M(int n1,int n2,int n3)
{
    int result = n1 + n2 + n3;
}

public static string M(string s1,string s2)
{
    return s1 + s2;
}
```

## 10. 访问修饰符

public：公开的

private：私有的，只能在当前类的内部访问

protected：受保护的，只能在当前类的内部以及该类的子类中访问

internal：只能在当前项目中访问，在本项目中和public权限一样

protected internal：protected+internal

（1）能够修饰类的访问修饰符：public，internal

（2）子类的访问权限不能高于父类的访问权限，会暴露父类的成员

# :star: C# 进阶

## :shamrock:1. 类

```C#
语法：
public class 类名
{
	字段;
	属性;
	方法;
}

//规范:每写一个类要新建一个类文件
```

```C#
类：
public class Person
{
    public string _name;
    public int _age;
    public string _gender;

    public void CHLSS()
    {
        Console.WriteLine($"我叫{this._name},我今年{this._age}岁了，我性别是{this._gender}。");
//this:表示当前这个类的对象。类是不占内存的，而对象是占内存的。
    }
}

对类调用：
static void Main(string[] args)
{
    //写好了一个类之后，我们需要创建这个类的对象，那么，我们管创建这个类的对象过程称之为类的实例化。使用关键字 new.
    Person sunQuan = new Person();
    sunQuan._name = "孙权";
    sunQuan._age = 23;
    sunQuan._gender = "男";
    sunQuan.CHLSS();
}
```

### 部分类（partial）

```C#
可以理解为将Person类分开，两部分共同组成Person类，数据共享
public partial class  Person
{ }
public partial class  Person
{ }
```

### 密封类（sealed）

```C#
密封类不能被继承，但可以继承其他父类
public sealed calss Person : Test
{ }
```



## 2. 属性

3、属性
属性的作用就是**保护字段、对字段的赋值和取值进行限定**。
属性的本质就是两个方法，一个叫get()一个叫set()。

```C#
set（）源码：
public void set_Name(string value)
{
    this._name = value;
}

get（）源码：
public string get_Name
{
    return this._name;
}
```

```C#
用法：
public class Person
{
    private int _age;//字段在类中必须是私有的，如果想访问只能通过属性！
    public int Age  //属性必须是公有的，可以外部访问 
    {
        //输出属性的值的时候，会执行get方法
        get { return _age; }
        //给属性赋值的时候，首先会执行set方法
        set 
        { 
            //添加额外条件起到限定作用
            if（value < 0 || value > 100）
            {
                value = 0;
            }
            //默认功能
            _age = value; 
        }
    }
    
     public void CHLSS()
    {
       	//调用属性this.Age
        Console.WriteLine($"我今年{this.Age}岁了");
    }
}

对类调用：
static void Main(string[] args)
{
    Person sunQuan = new Person();
    sunQuan.Age = "10";   //调用属性sunQuan.Age
    sunQuan.CHLSS();
}
```

```C#
新写法：
public int Age { get => _age; set => _age = value; }
```

既有get()也有set()我们诚之为可读可写属性。
只有get()没有set()我们称之为只读属性
没有get()只有set()我们称之为只写属性



## 3. 静态和非静态

1)、在非静态类中，既可以有实例成员（非静态），也可以有静态成员。
2)、在调用实例成员的时候，需要使用**对象名.实例成员**;
    在调用静态成员的时候，需要使用**类名.静态成员名**;

```C#
public class Person
{
    public void M1()
    {
        Console.WriteLine("非静态");
    }
    public static void M2()
    {
        Console.WriteLine("静态");
    }
}

class Program
{
static void Main(string[] args)
{
    //调用实例成员
    Person p = new Person();
    p.M1(); //实例方法

    //c.M2();  报错
    Person.M2();  //静态方法

}
```

总结： **静态成员必须使用类名去调用，而实例成员使用对象名调用。**	  	  **静态函数中，只能访问静态成员，不允许访问实例成员。**
      **实例函数中，既可以使用静态成员，也可以使用实例成员。**
      **静态类中只允许有静态成员，不允许出现实例成员。**

**什么时候使用：** 
1)、如果你想要你的类当做一个"工具类"去使用，这个时候可以考虑将类写成静态的。
2)、静态类在整个项目中资源共享。只有在程序全部结束之后，静态类才会释放资源。

静态类存放在：堆  栈  静态存储区域

释放资源。GC Garbage Collection垃圾回收器



## 4. 构造函数

作用：帮助我们初始化对象(给对象的每个属性依次的赋值)
构造函数是一个特殊的方法：
1)、构造函数没有返回值，连void也不能写。
2)、构造函数的名称必须跟类名一样。

3)、访问修饰符必须是public

**创建对象的时候会执行构造函数**
**构造函数是可以有重载的。**

```C#
public class Person
{
    //构造函数
    public Person(string name,int age,string gender)
    {
        this.Name = name;
        this.Age = age; 
        this.Gender = gender;
    }
	
    //构造函数的重载
    public Person(string name,int age)
    {
        this.Name = name;
        this.Age = age; 
    }
    
    //析构函数
    ~Person（）
    {
        //手动回收内存
    }
    
    private string _name;
    public string Name
    {
        get { return _name; }        
        set { _name = value; }
    }

    private int _age;  
    public int Age 
    {
        get { return _age; }   
        set { _age = value; }
    }

    private string _gender;
    public string Gender
    {
        get { return _gender; }
        set { _gender = value; }
    }
}

 class Program
{
    static void Main(string[] args)
    {
        Person p1 = new Person("孙权",10,"男"); //初始化
        Person p2 = new Person("孙尚香",10）; //重载
    }
}
```

**类当中会有一个默认的无参数的构造函数**，当你写一个新的构造函数之后，不管是有参数的还是无参数的，那个默认的无参数的构造函数都被干掉了。

**new关键字**
Person p=new Person();
new帮助我们做了3件事儿：
1)、在内存中开辟一块空间
2)、在开辟的空间中创建对象
3)、调用对象的构造函数进行初始化对象

**this关键字**
1)、代表当前类的对象
2)、在类当中显示的调用本类的构造函数  :this （进阶）

```C#
 //构造函数
public Person(string name,int age,string gender)
{
    this.Name = name;
    this.Age = age; 
    this.Gender = gender;
}

//：this（name,age,""）再次调用构造函数，可以省区该函数中的数据
public Person(string name,int age)：this（name,age,""）
{
    //this.Name = name;
    //this.Age = age; 
}
```



## 5. 值类型和引用类型

**区别：**

1. 值类型和引用类型在内存上存储的地方不一样。

2. 在传递值类型和传递引用类型的时候，传递的方式不一样。

   值类型我们称之为值传递，引用类型我们称之为引用传递。

   

**值类型：**int,double,bool,char,decimal,struct,enum

**引用类型：**string,自定义类,数组，集合，object 接口



**存储：**

值类型的值存储在内存的栈中

引用类型的值存储在内存的堆中

![image-20220623150217728](image-20220623150217728.png)

## 6. 字符串

[C# 字符串（String） | 菜鸟教程 (runoob.com)](https://www.runoob.com/csharp/csharp-string.html)

（1）字符串的不可变性

当年给一个字符串重新赋值之后，老值并没有被销毁，而是重新开辟一块空间存储新值。程序结束时，未被指向的老值自动销毁，释放空间。

![image-20220623151107009](image-20220623151107009.png)

（2）我们可以将字符串看作是char类型的一个**只读数组**，**可通过下标进行访问 **

对数组进行修改的方法：将数组转换为字符串

![image-20220623151814495](image-20220623151814495.png)



## :shamrock:7. 继承

```C#
父类（基类）：
public class Person

子类（派生类）：
public class Student : Person
```

### 特性

（1）子类继承了父类的属性和方法，不能继承父类私有字段和构造函数。

（2）单根性（一个类只能由一个父类）和传递性

（3）[子类与父类构造函数的关系](https://www.bilibili.com/video/BV1FJ411W7e5?p=115&spm_id_from=pageDriver&vd_source=9d1c0e05a6ea12167d6e82752c7bc22a)

（4）子类写的成员函数和父类的**同名**时，会把父类的隐藏掉。

![image-20220623161508887](image-20220623161508887.png)

加new之后不再警告

```C#
public new void SayHello() 
{}
```

（5）子类对象可以调用父类中的成员，但是父类对象永远都只能调用自己的成员。



### base()函数

由继承特性（4）子类写的成员函数和父类的**同名**时，会把父类的隐藏掉。

**base 关键字用于从派生类中访问基类的成员：
调用基类上已被其他方法重写的方法。
指定创建派生类实例时应调用的基类构造函数。
基类访问只能在构造函数、实例方法或实例属性访问器中进行。**

```C#
//1. 在派生类中调用基类方法。
public class BaseClass
{
    protected string _className = "BaseClass";
    public virtual void PrintName()
    {
        Console.WriteLine("Class Name: {0}", _className);
    }
}
class DerivedClass : BaseClass
{
    public string _className = "DerivedClass";
    public override void PrintName()
    {
        Console.Write("The BaseClass Name is {0}");
        //调用基类方法
        base.PrintName();
        Console.WriteLine("This DerivedClass is {0}", _className);
    }
}

//2. 在派生类中调用基类构造函数。
public class BaseClass
{
    int num;
    public BaseClass()
    {
        Console.WriteLine("in BaseClass()");
    }
    public BaseClass(int i)
    {
        num = i;
        Console.WriteLine("in BaseClass(int {0})", num);
    }
}
public class DerivedClass : BaseClass
{
    // 该构造器调用  BaseClass.BaseClass()
    public DerivedClass()
        : base()
    {
    }
    // 该构造器调用 BaseClass.BaseClass(int i)
    public DerivedClass(int i)
        : base(i)
    {

```



### 里氏转换

[里氏转换练习](https://www.bilibili.com/video/BV1FJ411W7e5?p=120&vd_source=9d1c0e05a6ea12167d6e82752c7bc22a)

```C#
 static void Main(string[] args)
{
    //里氏转换
    //（1）子类可以赋值给父类:如果有一个地方需要一个父类作为参数，我们可以给一个子类代替
    Person p = new Student();

    //（2）如果父类中装的是子类对象，那么可以讲这个父类强转为子类对象
    Student ss = (Student)p;
    ss.StudentSayHello();
}
```


**is**：表示类型转换，如果能够转换成功，则返回一个true，否则返回一个false
**as**：表示类型转换，如果能够转换则返回对应的对象，否则返回一个null

```C#
static void Main(string[] args)
{
    Person p = new Student();
    
    //is的用法
    if (p is Teacher)
    {
        Student ss = (Student)p;
        ss.StudentSayHello();
    }
    else
    {
        Console.WriteLine("转换失败");
    }
    
    //as的用法
    Student ss = ss as Student;  
}
```



## 8. 集合（Collection）

集合（Collection）类是专门用于数据存储和检索的类。这些类提供了对栈（stack）、队列（queue）、列表（list）和哈希表（hash table）的支持。大多数集合类实现了相同的接口。

集合（Collection）类服务于不同的目的，如为元素动态分配内存，基于索引访问列表项等等。这些类创建 Object 类的对象的集合。在 C# 中，Object 类是所有数据类型的基类。

各种集合类和它们的用法

下面是各种常用的 **System.Collection** 命名空间的类。点击下面的链接查看细节。

| 类                                                           | 描述和用法                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [动态数组（ArrayList）](https://www.runoob.com/csharp/csharp-arraylist.html) | 它代表了可被单独**索引**的对象的有序集合。它基本上可以替代一个数组。但是，与数组不同的是，您可以使用**索引**在指定的位置添加和移除项目，动态数组会自动重新调整它的大小。它也允许在列表中进行动态内存分配、增加、搜索、排序各项。 |
| [哈希表（Hashtable）](https://www.runoob.com/csharp/csharp-hashtable.html) | 它使用**键**来访问集合中的元素。当您使用键访问元素时，则使用哈希表，而且您可以识别一个有用的键值。哈希表中的每一项都有一个**键/值**对。键用于访问集合中的项目。 |
| [排序列表（SortedList）](https://www.runoob.com/csharp/csharp-sortedlist.html) | 它可以使用**键**和**索引**来访问列表中的项。排序列表是数组和哈希表的组合。它包含一个可使用键或索引访问各项的列表。如果您使用索引访问各项，则它是一个动态数组（ArrayList），如果您使用键访问各项，则它是一个哈希表（Hashtable）。集合中的各项总是按键值排序。 |
| [堆栈（Stack）](https://www.runoob.com/csharp/csharp-stack.html) | 它代表了一个**后进先出**的对象集合。当您需要对各项进行后进先出的访问时，则使用堆栈。当您在列表中添加一项，称为**推入**元素，当您从列表中移除一项时，称为**弹出**元素。 |
| [队列（Queue）](https://www.runoob.com/csharp/csharp-queue.html) | 它代表了一个**先进先出**的对象集合。当您需要对各项进行先进先出的访问时，则使用队列。当您在列表中添加一项，称为**入队**，当您从列表中移除一项时，称为**出队**。 |
| [点阵列（BitArray）](https://www.runoob.com/csharp/csharp-bitarray.html) | 它代表了一个使用值 1 和 0 来表示的**二进制**数组。当您需要存储位，但是事先不知道位数时，则使用点阵列。您可以使用**整型索引**从点阵列集合中访问各项，索引从零开始。 |

### ArrayList集合（动态数组）

每次集合中实际包含的元素个数(count)超过了可以包含的元素的个数(capcity)的时候，集合就会向内存中申请多开辟一倍的空间，来保证集合的长度一直够用。

```C#
  static void Main(string[] args)
{
    //非静态类，创建集合对象
    ArrayList list = new ArrayList();
    //集合：很多数据的一个集合，长度可以随意改变，类型不固定
    //数组：长度不可变，类型单一

    //添加单个元素用list.Add();
    list.Add(1);
    list.Add(2.1);
    list.Add(true);
    list.Add("张三");
    //添加集合用list.AddRange();
    list.AddRange(new int[] { 1, 2, 3, 4, 5 });
    Person p = new Person();
    list.AddRange(list);  //自己放自己

    for (int i = 0; i < list.Count; i++)
    {
        Console.WriteLine(list[i]);
    }
}

输出：
1
2.1
True
张三
1
2
3
4
5
1
2.1
True
张三
1
2
3
4
5
```

```C#
list.Remove("张三"); //指定删除单个元素
list.RemoveAt("0); //根据下标删除单个元素
list.RemoveRange(0，n); //（从下标0开始删除n个）
list.Clear(); //清空所有元素
list.Sort(); //升序排列
list.Reverse(); //反转
......
```

## 9. Path类

用于操作路径

```C#
static void Main(string[] args)
{
    string str = @"C:\Users\22625\Desktop\1.txt";
    //快速获得文件名，输出：1.txt
    Console.WriteLine(Path.GetFileName(str));
    //只获得扩展名，输出：.txt
    Path.GetExtension(str);
    //不包含扩展名，输出：1
    Path.GetFileNameWithoutExtension(str)
   	//文件目录，输出：C:\Users\22625\Desktop
   	Path.GetDirectoryName(str);
    //文件全路径
    Path.GetFullPath(str);
    //链接两个字符串作为路径
    Path.Combine(@"C:\Users\22625\Desktop\","1.txt")
    ......
}
```

## 10.文件IO

[C# 文件的输入与输出 | 菜鸟教程 (runoob.com)](https://www.runoob.com/csharp/csharp-file-io.html)

## 11. foreach循环

foreach 循环用于列举出集合中所有的元素，foreach 语句中的表达式由关键字 in 隔开的两个项组成。

in 右边的项是集合名，in 左边的项是变量名，用来存放该集合中的每个元素。

该循环的运行过程如下：每一次循环时，从集合中取出一个新的元素值。放到只读变量中去，如果括号中的整个表达式返回值为 true，foreach 块中的语句就能够执行。

一旦集合中的元素都已经被访问到，整个表达式的值为 false，控制流程就转入到 foreach 块后面的执行语句。

**foreach 语句经常与数组一起使用**，在 C# 语言中提供了 foreach 语句**遍历数组中的元素**，具体的语法形式 如下。

```C#
foreach(数据类型 变量名 in 数组名)
{
  //语句块；
}

```

这里变量名的数据类型必须与数组的数据类型相兼容。

在 foreach 循环中，如果要输出数组中的元素，不需要使用数组中的下标，直接输出变量名即可。

**foreach 语句仅能用于数组、字符串或集合类数据类型。**



```C#
【实例】在 Main 方法中创建一个 double 类型的数组，并在该数组中存入 5 名学生的考试成绩，计算总成绩和平均成绩。

根据题目要求，使用foreach语句实现该功能，代码如下。
class Program
{
    static void Main(string[] args)
    {
        double[] points = { 80, 88, 86, 90, 75.5 };
        double sum = 0;
        double avg = 0;
        foreach(double point in points)
        {
            sum = sum + point;
        }
        avg = sum / points.Length;
        Console.WriteLine("总成绩为：" + sum);
        Console.WriteLine("平均成绩为：" + avg);
    }
}
```

在计算平均成绩时，通过数组的 Length 属性即可得到数组中元素的个数，使用总成绩除以元素的个数即为结果。

执行上面的语句，效果如下图所示。



![求总成绩和平均成绩](4-1Z320162532159.gif)


从上面的执行效果可以看出，在使用 foreach 语句时可以免去使用下标的麻烦，这也给遍历数组中的元素带来很多方便。

## :shamrock:12. 多态

### 虚方法（virtual）

**当父类中的方法需要实现,将父类的方法标记为虚方法 ，使用关键字 virtual**，这个函数可以被子类重新写一个遍。**子类的方法使用关键字override**。

```C#
 class Program
    {
        static void Main(string[] args)
        {
            //真的鸭子嘎嘎叫，木头鸭子吱吱叫，橡皮鸭子唧唧叫
            ReadDuck rd = new ReadDuck();
            WoodDuck wd = new WoodDuck();
            XPDuck xd = new XPDuck();

            ReadDuck[] ducks = { rd, wd, xd };
            for (int i = 0;i < ducks.Length;i++)
            {
                ducks[i].jiao();
            }

        }
    }

    public class ReadDuck
    {
        public virtual void jiao()
        {
            Console.WriteLine("真的鸭子嘎嘎叫");
        }
    }

    public class WoodDuck : ReadDuck
    {
        public override void jiao()
        {
            Console.WriteLine("木头鸭子吱吱叫");
        }
    }
    public class XPDuck : ReadDuck
    {
        public override void jiao()
        {
            Console.WriteLine("橡皮鸭子唧唧叫");
        }
    }
```

### 抽象类（abstract）

**当父类中的方法不知道如何去实现的时候，可以考虑将父类写成抽象类，将方法写成抽象方法。**

```C#
static void Main(string[] args)
{
    //狗会叫，猫也会叫
    //Animal a = new Animal();  抽象类或接口无法创建对象
    Animal dog = new Dog();
    dog.jiao();
    Animal cat = new Cat();
    cat.jiao();
}

//加abstract
public abstract class Animal
{
	public abstract void jiao();  //抽象方法不写方法体
}

public class Dog: Animal
{
	public override void jiao()
	{
    	Console.WriteLine("狗会叫");
	}
}
public class Cat : Animal
{
	public override void jiao()
	{
   	 	Console.WriteLine("猫也会叫");
	}
}
```

1.抽象成员必须标记为abstract,并且不能有任何实现。
2.抽象成员必须在抽象类中。
3.抽象类不能被实例化

4.子类继承抽象类后，必须把父类中的所有抽象成员都重写。

（除非子类也是一个抽象类，则可以不重写）
5.抽象成员的访问修饰符不能是private
6.在抽象类中可以包含实例成员。
并且抽象类的实例成员可以不被子类实现

7.抽象类是有构造函数的。虽然不能被实例化。

8、如果父类的抽象方法中有参数，那么。继承这个抽象父类的子类在重写父类的方法的时候必须传入对应的参数。

如果抽象父类的抽象方法中有返回值，那么子类在重写这个抽象方法的时候 也必须要传入返回值。

======
如果父类中的方法有默认的实现，并且父类需要被实例化，这时可以考虑将父类定义成一个普通类，用虚方法来实现多态。

如果父类中的方法没有默认实现，父类也不需要被实例化，则可以将该类定义为抽象类。

### 接口（interface）

在C#语言中，接口也会定义一种标准，如果需要使用接口，必须满足接口中所定义的内容。

在C#语言中，类之间的继承关系仅支持单重继承，而接口是为了实现多重继承关系设计的。

一个类能同时实现多个接口，还能在实现接口的同时再继承其他类，并且接口之间也可以继承。

```C#
public interface 接口名称（通常以I开头，如ICompute）
{
    接口成员；
}

接口中定义的成员必须满足以下要求。
接口中的成员不允许使用 public、private、protected、internal 访问修饰符。
接口中的成员不允许使用 static、virtual、abstract、sealed 修饰符。
在接口中不能定义字段。
在接口中定义的方法不能包含方法体。
```

【实例】创建一个接口计算学生成绩的接口 ICompute,并在接口中分别定义计算总成绩、平均成绩的方法。

根据题目要求，在该接口中定义学生的学号、姓名的属性，并定义计算成绩的总分和 平均分的方法。

定义接口的代码如下。

```C#
interface ICompute
{
    int Id { get; set; }
    string Name { get; set; }
    void Total();
    void Avg();
}
```

通过上面的代码即可完成一个接口的定义，但是由于接口中的方法并没有具体的内容，直接调用接口中的方法没有任何意义。

#### 隐式方式

**隐式实现接口成员是将接口的所有成员以 public 访问修饰符修饰。**

使用隐式方式来实现接口 ICompute 的成员，以计算机专业的学生类 (ComputerMajor) 实现 ICompute 接口，为其添加英语 (English)、编程 (Programming)、数据库 (Database) 学科成绩属性，代码如下。

```C#
class ComputerMajor : ICompute
{
    public int Id { get; set; }     //隐式的实现接口中的属性
    public string Name { get; set; }    //隐式实现接口中的属性
    public double English { get; set; }
    public double Programming { get; set; }
    public double Database { get; set; }
    public void Avg()       //隐式实现接口中的方法
    {
        double avg = (English + Programming + Database) / 3;
        Console.WriteLine("平均分：" + avg);
    }
    public void Total()
    {
        double sum = English + Programming + Database;
        Console.WriteLine("总分为：" + sum);
    }
}

```

在 Main 方法中调用该实现类的成员，代码如下。

```C#
class Program
{
    static void Main(string[] args)
    {
        ComputerMajor computerMajor = new ComputerMajor();
        computerMajor.Id = 1;
        computerMajor.Name = "李明";
        computerMajor.English = 80;
        computerMajor.Programming = 90;
        computerMajor.Database = 85;
        Console.WriteLine("学号：" + computerMajor.Id);
        Console.WriteLine("姓名：" + computerMajor.Name);
        Console.WriteLine("成绩信息如下：");
        computerMajor.Total();
        computerMajor.Avg();
    }
}
```

执行上面的代码，效果如下图所示。

![使用隐式方式实现接口成员](4-1Z3221P219338.gif)

#### 显式接口

**显式实现接口是指在实现接口时所实现的成员名称前含有接口名称作为前缀。**

需要注意的是使用显式实现接口的成员不能再使用修饰符修饰，即 public、abstract、virtual、 override 等。

```C#
class ComputerMajor : ICompute
{
    public double English { get; set; }
    public double Programming { get; set; }
    public double Database { get; set; }
    int ICompute.Id { get; set; }           //显示实现接口中的属性
    string ICompute.Name { get; set; }      //显示实现接口中的属性
    void ICompute.Total()                   //显示实现接口中的方法
    {
        double sum = English + Programming + Database;
        Console.WriteLine("总分数：" + sum);
    }
    void ICompute.Avg()
    {
        double avg = (English + Programming + Database) / 3;
        Console.WriteLine("平均分为：" + avg);
    }
}
```

从上面的代码可以看出，在使用显式方式实现接口中的成员时，所有成员都会加上接口名称 ICompute 作为前缀，并且不加任何修饰符。

在Main方法中调用实现类中的成员,代码如下

```C#
class Program
{
    static void Main(string[] args)
    {
        ComputerMajor computerMajor = new ComputerMajor();
        ICompute compute = computerMajor;       //创建接口的实例
        compute.Id = 1;
        compute.Name = "李明";
        computerMajor.English = 80;
        computerMajor.Programming = 90;
        computerMajor.Database = 85;
        Console.WriteLine("学号：" + compute.Id);
        Console.WriteLine("姓名：" + compute.Name);
        Console.WriteLine("成绩信息如下：");
        compute.Total();
        compute.Avg();
    }
}
```

执行上面的代码，效果与上图一致。从调用的代码可以看出，在调用显式方式实现接口的成员时，必须使用接口的实例来调用，而不能使用实现类的实例来调用。

#### 接口中多态的实现

使用接口实现多态 需要满足以下两个条件。

- 定义接口并使用类实现了接口中的成员。
- 创建接口的实例指向不同的实现类对象。


假设接口名称为 ITest，分别定义两个实现类来实现接口的成员，示例代码如下。

```C#
interface ITest
{
    void methodA();
}
class Test1 : ITest
{
    public void methodA()
    {
        Console.WriteLine("Test1 类中的 methodA 方法");
    }
}
class Test2 : ITest
{
    public void methodA()
    {
        Console.WriteLine("Test2 类中的 methodA 方法");
    }
}

class Program
{
    static void Main(string[] args)
    {
        ITest test1 = new Test1();  //创建接口的实例test1指向实现类Test1的对象
        test1.methodA();
        ITest test2 = new Test2();  //创建接口的实例test2指向实现类Test2的对象
        test2.methodA();
    }
}
```

执行上面的代码，效果如下图所示。

![使用多态的方式调用实现类](4-1Z325101133329.gif)
