

---
title: C#基础
aliases: []
tags: []
create_time: 2023-05-21 21:50
uid: 202305212150
cssclass: academia, academia-rounded
banner: "![[Pasted image 20230521215136.png]]"
---



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
