
---
title: 《CS Primer》
aliases: []
tags: []
create_time: 2023-05-26 12:41
uid: 202305261241
cssclass: academia, academia-rounded
banner: "![[Pasted image 20230526124440.png]]"
---

# 零、特性
## 折叠代码
```cs
#region 折叠块名字
... //代码
#endregion
```

## 控制台方法
```cs file:打印输入输出
Console.Write("xxx"); // 打印，不自动空行
Console.WriteLine("xxx"); // 打印，自动空行
Console.ReadLine(); //等待直到用户按下回车，一次读入一行。
Console.ReadKey();  // 等待用户按下任意键，一次读入一个字符。

Console.ReadKey(true).KeyChar; 
//Console 类的一个静态方法，它读取当前控制台上的任意键盘输入。参数 true 表示在读取输入后不显示读入的字符，如果是 false 则会显示读入的字符。
//返回值是 ConsoleKeyInfo 类型，包含了该字符的 KeyChar 属性（即按下的按键字符），以及关于按键是否有控制字符等其他信息。
```

```cs file:其他方法
//1.清空
Console.Clear();  

//2. 设置控制台大小
//注意：
//1.先设置窗口大小，在设置缓冲区大小
//2.缓冲区大小不能小于窗口大小
//3.窗口大小不能大于控制台的最大尺寸
Console.SetWindowSize(50,40);  // 设置窗口大小
Console.SetBufferSize(1000, 1000); // 设置缓冲区大小（可打印内容区域的宽高）

//3.设置光标的位置
//控制台左上角为原点，右侧是x轴正方向，下方是Y轴正方向，它是一个平面二维坐标系
//注意:
//1.边界问题
//2.横纵距离单位不同 1y = 2x 视觉上的
Console.SetCursorPosition(10,5);

//4.设置颜色相关
//文字颜色设置
Console.ForegroundColor = ConsoleColor.Red;
//背景颜色设置
Console.BackgroundColor = ConsoleColor.White;

//5.光标显隐
Console.cursorVisible = false;

//6.关闭控制台
Environment.Exit(0);
```

## Path 类

用于操作路径

```cs
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

## 垃圾回收 GC
垃圾回收, 英文简写 GC (Garbage Collector)

垃圾回收的过程是在遍历堆 (Heap)上动态分配的所有对象，通过识别它们是否被引用来确定哪些对象是垃圾，哪些对象仍要被使用。
所谓的垃圾就是没有被任何变量，对象引用的内容。垃圾就需要被回收释放，

垃圾回收有很多种算法，比如
- 引用计数 (Reference Counting)
- 标记清除 (Mark Sweep)
- 标记整理 (Mark Compact)
- 复制集合 (Copy collection)


> [!NOTE] 
> Gc 只负责堆 (Heap)内存的垃圾回收，引用类型都是存在堆 (Heap)中的，所以它的分配和释放都通过垃圾回收机制来管理

GC 只负责堆 (Heap)内存的垃圾回收，引用类型都是存在堆 (Heap)中的，所以它的分配和释放都通过垃圾回收机制来管理
栈 (Stack)上的内存是由系统自动管理的，值类型在栈 (Stack)中分配内存的。他们有自己的生命周期，不用对他们进行管理，会自动分配和释放

**CS 中内存回收机制的大概原理:**
0 代内存 1 代内存 2 代内存
**代的概念:** 
1. 代是垃圾回收机制使用的一种算法 (分代算法)
2. 新分配的对象都会被配置在第 0 代内存中
3. 每次分配都可能会进行垃圾回收以释放内存 (0 代内存满时)

- 在一次内存回收过程开始时，垃圾回收器会认为堆中全是垃圾，会进行以下两步
    1. 标记对象从根（静态字段、方法参数）开始检查引用对象（引用类型，比如类、数组），标记后为可达对象，未标记为不可达对象，不可达对象就认为是垃圾
    2. 搬迁对象压缩堆 (挂起执行托管代码线程), 释放未标记的对象，搬迁可达对象，修改引用地址

- 大对象总被认为是第二代内存，目的是减少性能损耗，提高性能
- 不会对大对象进行搬迁压缩（85080 字节 (83kb）以上的对象为大对象）
![](<images/1684809112351.png>)  

```cs file:手动垃圾回收
// 一般情况下，我们不会频繁调用、
// 都是在Loading过场景时，才调用
CG.Collect();
``` 

# 一、变量和基本类型
## 内置类型

**值类型：** 无符号整形，有符号整形浮点数 char bool enum 结构体

**引用类型：** string, 类, 数组，集合，object 接口

值类型和引用类型**区别：**
1. 值类型和引用类型在内存上存储的地方不一样。
2. 在传递值类型和传递引用类型的时候，传递的方式不一样。值类型我们称之为值传递，引用类型我们称之为引用传递。
3. 值类型的值存储在内存的**栈**中 (系统自动回收，小而快)，引用类型的值存储在内存的**堆**中（手动释放，大而慢）
![image-20220623150217728|650](image-20220623150217728.png)
- @ 引用类型的数据存在堆中，栈中只存一个地址指向堆中存储的数据
```cs
//原始变量
int a = 1;                   //值类型
int[] arr1 = { 1, 2, 3, 4 }; //引用类型

//新增变量
int b = a;         //值传递
int[] arr2 = arr1; // 引用传递

//修改新增变量
b = 5; 
arr2[0] = 5;
            
//结果
//值传递：b是a的一个拷贝，修改b的值，不会改变a的值
// 引用传递：arr2是arr1的一个引用，修改arr2，会改变arr1
Console.WriteLine(a);       //输出1
Console.WriteLine(arr1[0]); //输出5
```
### 值类型

|有符号整数类型   |描述|范围| 默认值 |
|----|----|----|-----|
| sbyte | 8 位有符号整数类型 | -128 到 127 | 0 |
| short |16 位有符号整数类型| -32,768 到 32,767 | 0 |
|int| 32 位有符号整数类型 | -2,147,483,648 到 2,147,483,647 | 0 |
|long| 64 位有符号整数类型 | -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807 | 0L |

|无符号整数| 描述 | 范围 | 默认值 |
|----|----|----|-----|
| byte | 8 位无符号整数 | 0 到 255 | 0 |
| ushort | 16 位无符号整数类型 | 0 到 65,535 | 0 |
| uint | 32 位无符号整数类型 | 0 到 4,294,967,295 | 0 |
| ulong | 64 位无符号整数类型 | 0 到 18,446,744,073,709,551,615 | 0 |

|浮点数|描述| 范围 | 默认值 |
|----|----|----|-----|
| float | 32 位单精度浮点型 | -3.4 x 1038 到 + 3.4 x 1038 | 0.0F |
| double | 64 位双精度浮点型 | (+/-) 5.0 x 10-324 到 (+/-) 1.7 x 10308 | 0.0D |
|decimal| 128 位精确的十进制值，28-29 有效位数 | (-7.9 x 1028 到 7.9 x 1028) / 100 到 28 | 0.0M |

|其他类型| 描述 | 范围 | 默认值 |
|----|----|----|-----|
| char | 16 位 Unicode 字符 | U +0000 到 U +ffff | '\0' |
| bool |8 位布尔值| True 或 False | False |


1. c# 中的小数默认为 double 类型，所以声明 float 时末尾加 f (或大写 F)显示表示 ：
```cs
float a = 0.1654646f;
```
2.  `sizeof() ` 返回**值类型**变量的大小（字节）
3. **保留指定小数位数**

```cs
//语法：
//{变量：0.00}  保留两位
double n1 = 3.33333;
Console.WriteLine($"{n1:0.00}");

输出：3.33
```

#### 结构体 struct

```cs
 public struct Person
{
    public string _name;  //字段前要加_，用来区分变量
    public int _age;
    public char _gender;
    
    void Speak()
    {
    ...
    }
}
```
- ? 结构体和类的区别

### 引用类型

|C# 类型关键字|. NET 类型|
|---|---|
|object| [System.Object](https://learn.microsoft.com/zh-cn/dotnet/api/system.object) |
|string| [System.String](https://learn.microsoft.com/zh-cn/dotnet/api/system.string) |
|dynamic| [System.Object](https://learn.microsoft.com/zh-cn/dotnet/api/system.object) |

在上表中，左侧列中的每个 C# 类型关键字（[dynamic](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/reference-types#the-dynamic-type) 除外）都是相应 .NET 类型的别名。它们是可互换的。例如，以下声明声明了相同类型的变量：
```cs
int a = 123;
System.Int32 b = 123;
```
#### string
##### 独特的值类型特征
string 虽然是引用类型，但他有值类型的特征
```c++
string str1 = "123"
string str2 = str1;

//若改变str2, str1不会发生改变,str2会在堆中重新分配空间
str2 = "321";
//因此，频繁对string赋值会产生内存垃圾
```
![[Pasted image 20230526153510.png|350]]

##### 字符串类型拼接方式
1. "+" "+="号，不能用其他运算符
2. `string. Format ("待拼接的内容",内容 1,内容 2，......)`
   使用占位符 `{数字}` 控制拼接顺序
```cs
string s = string.Format("我是{0},我今年{1}岁,我喜欢{2}","小明","16","玩游戏");
Console.WriteLine(s);

// 等价
Console.WriteLine(string.Format("我是{0},我今年{1}岁,我喜欢{2}","小明","16","玩游戏"));

```
3. `$` 替代 `string.format()` 
   原先赋值需要占位符和变量，当需要拼接多个变量会造成语句过长等不易理解问题，`$` 可以把字符串中的变量 `{}` 包含起来达到识别变量的目的 `$"{id}"`；也支持表达式，使用 `$"{(你的表达式)}"`
```cs
var k = "a";  
var a0 = "User";  
var a1 = "Id";  
var a2 = 5;  
  
var ccb = $"select * from {a0} where {a1}={a2}";
//等价
var ccc = string.Format("select * from {0} where {1} = {2}", a0, a1, a2);
```
## 特殊类型
### 常量

```cs
const 变量类型 变量名 = 值
const int a = 1；
```

### 随机数

```cs
//1. 创建能够产生随机数的对象
Random r = new Random();
//2. 让产生随机数的这个对象调用方法来产生随机数
r.Next(); //生成一个非负的随机数  
r.Next(100); //生成[0,99)的随机数  
r.Next(5,100); //生成[5,99)的随机数
```

### 枚举

```cs file:声明枚举
// 声明枚举
public enum 枚举名
{
    值1, //默认值为0，后面依次递增
    值2,
    值3,
    ......
    值n  //最后一个逗号可加可不加
}

```

- 枚举通常声明到 namespace 的下面，class 的外面，表示这个命名空间下，所有的类都可以使用这个枚举。
- 不可以在函数中声明

```cs file:枚举搭配switch使用：
namespace ConsoleApp1
{
    enum EPlayer
    {
        singer,
        writer,
        teacher,
        student
    }
    
    internal class Program
    {
        static void Main(string[] args)
        {
            EPlayer Player1 = EPlayer.singer; //定义枚举变量
            
            switch (Player1)
            {
                case EPlayer.singer:
                    ...
                    break;
                case EPlayer.student:
                    ...
                    break;
                default:
                    break;
            }
        }
    } 
}
```

```cs file:枚举类型转换
EPlayer Player = EPlayer.singer;  
// 枚举转int  
int i = (int)Player;  
// int转枚举  
Player = 0;  
// 枚举转string  
string str = Player.ToString();  
// string转枚举  
Player = (EPlayer)Enum.Parse(typeof(EPlayer), "teacher"); // 注意第二个变量值必须是枚举声明中的成员
```

### 数组
数组声明后不可以改变长度，若想在原数组的基础上进行收缩，需要新建一个数组，将值复制到新数组。
#### 一维数组
```cs file:一维数组的声明
int[] nums;  //只声明不初始化
int[] nums = new int[5]; //全部为默认值0

// 以下方式等价
int[] nums = new int[]{1,2,3,4,5};
int[] nums = new int[5]{1,2,3,4,5};
int[] nums = {1,2,3,4,5};
```

```cs file:一维数组方法
int len = nums.Lenght()  //数组长度
```

```cs file:增加和减少数组中的元素
//增加数组中的元素
int[] array = { 1, 2, 3, 4, 5 };
int[] array1 = new int[10];
for (int i = 0; i < array.Length; i++)
{
    array1[i] = array[i];
}
array = array1;  //最后将新数组赋值给旧数组

//遍历打印array结果为
//1，2，3，4，5，0，0，0，0，0

//j减少数组中的元素
int[] array = {1,2,3,4,5,6,7,8,9,10};
int[] array2 = new int[5];
for (int i = 0; i < array2.Length; i++)
{
    array2[i] = array[i];
}
array = array2;

//遍历打印array结果为
//1，2，3，4，5

``` 

#### 二维数组
```cs file:二维数组的声明
int[,] nums;  //只声明不初始化
int[,] nums = new int[3,3]; //全部为默认值0

// 以下方式等价
int[,] nums = new int[,]{{1,1,1},
                           {2,2,2},
                           {3,3,3}};
                           
int[,] nums = new int[3,3]{{1,1,1},
                           {2,2,2},
                           {3,3,3}};
                           
int[,] nums = {{1,1,1},
               {2,2,2},
               {3,3,3}};
```

```cs file:二维数组方法
nums.GetLength(0) //获取行数
nums.GetLength(1) //获取列数
```
#### 交错数组
不常用，和二维数组的区别在于，每行的列数可以不同
```cs   file:交错数组的声明
int[][] arr1;
int[][] arr2 = new int[3][];


int[][] arr3 = new int[][]
            {
                new int[] { 1 },
                new int[] { 1, 2 },
                new int[] { 1, 2, 3 }
            };
            
int[][] arr4 = new int[3][]
            {
                new int[] { 1 },
                new int[] { 1, 2 },
                new int[] { 1, 2, 3 }
            };
            
int[][] arr5 = 
            {
                new int[] { 1 },
                new int[] { 1, 2 },
                new int[] { 1, 2, 3 }
            };
```


```c++ file:交错数组方法
nums.GetLength(0) //获取行数
nums[0].Length(1) //获取某一行的列数
```
# 二、函数（方法）

```cs
pubilc static 返回值类型 函数名（参数列表）
{
	函数体;
}
//public：访问修饰符
//static：静态的
//返回值类型：如果不需要写返回值，写void
//方法名：Pascal 每个单词首字母都大写
//参数列表：完成这个方法所必须要提供给这个方法的条件
    
 public static int GetMax(int n1, int n2)
{
    return n1> n2 ? n1 : n2;
}
```

## ref 和 out 参数  
他们使用的方式和效果都是一样，解决值类型和引用类型在函数内部改值或者重新声明能够影响外部传入的变量让其也被修改（使传入的参数在函数外也修改 ）


**ref 和 out 的区别：**  
1. ref 传入的变量 (参数) 必须初始化，out 不用。  
2. out 传入的变量必须在内部赋值，ref 不用。  

```cs file:ref参数
static void ChangeValue (int a)
{
    a = 20;
}

int b = 10
ChangeValue(b); 
//因为值传递的原因，b的值没有改变，我们想让b被改成20可以使用ref：

//参数前添加 ref 修饰符 
static void ChangeValue ( ref int a)
{
    a=20;
}
int b = 10
ChangeValue(b);
//b改变为20 在函数内修改传入参数 传入的参数在外部也会修改

``` 

如果你在一个方法中，返回多个相同类型的值的时候，可以考虑返回一个数组。
但是，如果返回多个不同类型的值的时候，返回数组就不行了，那么这个时候，
我们可以考虑使用 out 参数。**out 参数就侧重于在一个方法中可以返回多个不同类型的值。**

```cs file:out参数
public static void Test(int[]nums,out int max.out int min,out int sum,out float avr)
//out int max写到形参列表中
{
    max = nums[0];
    min = nums[1];
    sum = 0;
    avr = sum / nums.Length;
}

//外部调用：
Test（nums,out max,out min,out sun,out avr）;
```

## params 可变参数

- 可以输入不定的多个参数，并把这些参数存入数组。将实参列表中跟可变参数数组类型一致的元素都当做数组的元素去处理。

- 在函数参数中只能最多出现一个 params 关键字且一定在形参列表最后  

```cs
// 未使用可变参数：
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
## 可选参数
有参数默认值的参数一般称为可选参数
作用是当调用函数时可以不传入参数，不传就会使用默认值作为参数的值
```cs
static void Speak (string str == "hello")
{
    Console.WriteLine (str);
}
```
- 支持多个参数默认值
- 可选参数必须写在普通参数后面
## 函数重载

概念：方法的重载指的就是**方法的名称相同，但是参数不同**。
**参数不同，分为三种情况**
1. 如果参数的个数相同，那么参数的类型就不能相同。
2. 果参数的类型相同，那么参数的个数就不能相同。
3. 参数顺序不同

```cs
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

# 三、表达式和运算符
## 转义字符
1. 换行 `\n`  ：(windows 操作系统只认识 `\r\n`，不认识 `\n`)
2. 英文半角双引号：`\"\"`  中文半角可以直接打印出
3. 一个 Tab 空格: `\t`
4. 警报音：`\a`
5. 退格：`\b`
6. 斜杠：`\\ `
7.  `@` ：
1. 取消转义符的作用（用来存路径）

```cs
string a = @"C：\mycode\a\文件.txt";
```

2. 将字符串按照原格式输出
## 逻辑运算符
逻辑与：&&
逻辑或：||
逻辑非：！

优先级：
1. 逻辑**非**优先级最高
2. 逻辑**与**优先级大于逻辑**或**
3. **逻辑与**、**逻辑或**优先级低于算术运算符和条件运算符
## 位运算符
位运算符主要用数值类型进行计算，将数值转换为 2 进制，在进行位运算

1. 位与：&
对位运算，有 0 则 0
```cs
int a = 1; //001
int b = 5; //101
int c = a & b;
//得 c = 1  //001
```

2. 位或：|
对位运算，有 1 则 1

3. 异或：^
对位运算，相同为 0，不同为 1

4. 位取反：~
对位运算，0 变 1，1 变 0
5. 左移：<<
6. 右移：>>
让一个数的 2 进制数进行左移和右移
- 左移几位，右侧就加几个 0
```c++
a = 5; // 101
c = a << 5 
// 1位 1010
// 2位 10100
// 3位 101000
// 4位 1010000
// 5位 10100000 = 160
```
- 右移几位，右侧去掉几个数
```cs
a = 5;  // 101
c = a >> 2;
// 1位 10
// 2位 1
```

## 运算符重载

> [!success] 可重载运算符
> 算数运算符： + - * / % ++ --  
>
逻辑运算符： ！  
>
位运算符： & | ^ ~ << >>  
>
条件运算符: < <= > >= == !=  

> [!error] 不可重载运算符
> 
逻辑运算符 ： && || [ ] () . = ?:  

**作用**
让自定义类和结构体，能够使用运算符

使用关键字 `operator`

**特点**
1. 一定是一个公共的静态方法 `public static` 
2. 返回值写在 `operator` 前
3. 逻辑处理自定义
   
**注意**
1. 条件运算符需要成对实现
2. 一个符号可以多个重载
3. 不能使用 `ref` 和 `out`

```cs file:语法
public static 返回类型 operator 运算符(参数列表)
```

```cs file:案例
Point p1 = new Point();
p1.x =1;
p1.y =1;
Point p2 = new Point();
p2.x= 2;
p2.y = 2;
Point p3 =p1 + p2; //使用重载的+
Console.WriteLine(p3.x);

class Point
{
    public int x;
    public int y;

    public static Point operator +(Point p1, Point p2)
    {
        Point p = new Point();
        p.x = p1.x + p2.x;
        p.y = p1.y + p2.y;
        return p;
    }
}
```

### 可重载的运算符  


### 不可重载的运算符  

# 五、语句（控制流）

## 11. foreach 循环

foreach 循环用于列举出集合中所有的元素，foreach 语句中的表达式由关键字 in 隔开的两个项组成。

in 右边的项是集合名，in 左边的项是变量名，用来存放该集合中的每个元素。

该循环的运行过程如下：每一次循环时，从集合中取出一个新的元素值。放到只读变量中去，如果括号中的整个表达式返回值为 true，foreach 块中的语句就能够执行。

一旦集合中的元素都已经被访问到，整个表达式的值为 false，控制流程就转入到 foreach 块后面的执行语句。

**foreach 语句经常与数组一起使用**，在 C# 语言中提供了 foreach 语句**遍历数组中的元素**，具体的语法形式如下。

```cs
foreach(数据类型 变量名 in 数组名)
{
  //语句块；
}

```

这里变量名的数据类型必须与数组的数据类型相兼容。

在 foreach 循环中，如果要输出数组中的元素，不需要使用数组中的下标，直接输出变量名即可。

**foreach 语句仅能用于数组、字符串或集合类数据类型。**

```cs
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

# 四、面向对象 OOP

封装
## 1 类 class

```cs
语法：
public class 类名
{
	字段;
	属性;
	方法;
}

//规范:每写一个类要新建一个类文件
```

```cs
//类：
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

//类调用：
static void Main(string[] args)
{
    //类的实例化，使用关键字 new.
    Person sunQuan = new Person();
    sunQuan._name = "孙权";
    sunQuan._age = 23;
    sunQuan._gender = "男";
    sunQuan.CHLSS();

    // 以下都是空对象，没有申请堆空间，不可以访问成员变量和成员函数
    Person p1;  
    Person p2 = null; //等价
}
```

- 可以在类中声明一个和自己相同类型的成员变量，但**不能对类内部它进行实例化**
```cs
class person
{
    Person girlfriend;
    Person girlfriend = new Person(); //error!
    
    Person[] boyfriend;
}
```
- 成员变量的默认初始值
    - 值类型，数字类型默认为 0，bool 类型默认为 false
    - 引用类型，默认为 null
    - `default（类型）` 得到该类型的默认值

### 嵌套类
内部类，类中的类
```cs
Person p = new Person();
Person.Body body = new Person.Body(); // 实例化时指出外部类

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
```
### 分部类 partial 
把一个类分成几部分申明

关键字
partial
 
**作用**
1. 分部描述一个类
2. 增加程序的拓展性

**注意**
1. 分部类可以写在多个脚本文件中，数据共享
2. 分部类的访问修饰符要一致
3. 分部类中不能有重复成员

```cs file:分布类
//可以理解为将Person类分开，两部分共同组成Person类，数据共享
public partial class  Person
{ }
public partial class  Person
{ }
```

**分部方法**：将方法的声明和实现分离

**特点**
1. 不能加访问修饰符，默认private
2. 只能在分部类中声明
3. 返回值只能是 void
4. 可以有参数但不用 out 关键字
```cs
public partial class  Person
{
    public bool sex;
    partial void Speak(); // 声明
}


public partial class Person
{
    public int number;

    partial void Speak() //实现
    {
        // 逻辑
    }
}
```
### 密封类 sealed 
 
密封类不能被继承，但可以继承其他父类

加强面向对象程序设计的规范性、结构性、安全性

```cs
public sealed class Person : Test
{ }
```

## 2 访问修饰符
1. 不显式声明访问修饰符，则默认为 private
2. 分类：
`public`：公开的，可别类的内部外部访问（**可访问**可以理解为**可读写**）

`private`：私有的，只能在当前类的内部访问

`protected`：受保护的，只能在当前类的内部以及该类的子类中访问

`internal`：只能在当前项目中访问，在本项目中和 public 权限一样

`protected internal`：protected+internal

- 能够修饰类的访问修饰符：public，internal

- 子类的访问权限不能高于父类的访问权限，会暴露父类的成员
## 3 成员属性

1. **用于保护成员变量**  
2. **为成员属性的获取和赋值添加逻辑处理**  
3. **解决访问修饰符的局限性**
    - 访问修饰符只能同时控制读写，不能单独控制
    - 通过令属性的 get 或 set 为 private，可以让成员变量**在外部只能读不能写**或**只能写不能读**
4. **get 和 set 可以只有一个**
    - 既有 get ()也有 set ()我们诚之为可读可写属性。
    - 只有 get ()没有 set ()我们称之为只读属性
    - 没有 get ()只有 set ()我们称之为只写属性
```cs
//set（）源码：
public void set_Name(string value)
{
    this._name = value;
}

//get（）源码：
public string get_Name
{
    return this._name;
}
```

```cs file:用法
public class Person
{
    private int _age; //字段在类中必须是私有的，如果想访问只能通过属性！
    
    //属性必须是公有的，可以外部访问 
    public int Age   
    {
        //输出属性的值的时候，会执行get方法
        get { return _age; }
        
        //给属性赋值的时候，首先会执行set方法
        //value关键字用于表示外部传入的值
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
       	//调用属性this.Age，执行get方法
        Console.WriteLine($"我今年{this.Age}岁了");
    }
}

//对类调用：
static void Main(string[] args)
{
    Person sunQuan = new Person();
    sunQuan.Age = "10";   //调用属性sunQuan.Age，执行Set（）方法
    sunQuan.CHLSS();
}
```

```cs file:新写法
public int Age { get => _age; set => _age = value; }
```

5. **get 和 set 可以加访问修饰符**
- 默认不加，会使用属性声明时的访问权限
- 加的访问修饰符要低于属性的访问权限
- 不能让 get 和 set 的访问权限都低于属性的权限
```cs
public int age
{
    get { return _age }

    private set { _age = value }  // 给set加private，那么该属性只能读不能写
}
```

6. **自动属性**
```cs
public int age
{
    get;
    private set;
}
```
- 没有在 get 和 set 中写逻辑的需求时，可以使用自动属性。
- get set 仍可以添加 private。一般用于外部能读不能写的情况

## 4 静态 static 
1. 在非静态类中，既可以有实例成员（非静态），也可以有静态成员。
2. 在调用实例成员的时候，需要使用**对象名. 实例成员**;
3. 在调用静态成员的时候，需要使用**类名. 静态成员名**;

```cs
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


//调用实例成员
Person p = new Person();
p.M1(); //实例方法

//p.M2();  报错
Person.M2();  //静态方法

```

**总结：** 
- 静态函数中，只能访问静态成员，不允许访问实例成员。
- 实例函数中，既可以使用静态成员，也可以使用实例成员。
- **静态类**中只允许有静态成员，不能被实例化（适合作为工具类）。
- **静态构造函数**，用于初始化静态变量
    - 静态类和普通类中都可以有静态构造函数
    - 不能使用访问修饰符
    - 不能有参数
    - 只会自动调用一次
```cs file:静态构造函数
class Test
{
    public static int a = 100;
    // 静态构造函数
    static Test()
    {
        a = 200;    
    }
    
    // 普通构造函数
    public Test()
    {}
}
```


**什么时候使用：** 
1)、如果你想要你的类当做一个"工具类"去使用，这个时候可以考虑将类写成静态的。
2)、静态类在整个项目中资源共享。静态类存放在堆栈静态存储区域，只有在程序全部结束之后，静态类才会释放资源。

**const (常量)可以理解为特殊的 static (静态)**
- **相同点**
他们都可以通过类名点出使用
- **不同点**
    1. const 必须初始化，不能修改， static 没有这个规则
    2. const 只能修饰变量、static 可以修饰很多
    3. const 一定是写在访问修饰符后面的，static 没有这个要求

## 5 拓展方法
概念：为现有**非静态变量类型**添加新方法

**作用**
1. 提升程序拓展性
2. 不需要再对象中重新写方法
3. 不需要继承来添加方法
4. 为别人封装的类型写额外的方法

**特点**
1. 一定是写在静态类中
2.  一定是个静态函数
3. 第一个参数为拓展目标
4. 第一个参数用 this 修饰

```cs file:语法
访问修饰符 static 返回值 函数名(this 拓展类名 参数名，参数类型 参数名,参数类型 参数名....)
```

```cs
int i =10;
i.SpeakValue(); //int类型的拓展方法,i作为value值传入函数


static class Tools
{
    // 拓展方法写在
    静态类中
    public static void SpeakValue(this int value)
    {
        //拓展的方法的逻辑
        Console.WriteLine( value);
    }
}
```

也可以为类类型添加拓展方法，当拓展方法名和类成员函数重名时，只会调用类成员函数。
## 6 构造函数

**作用**：在实例化对象时（new 时），会调用用于初始化的函数，如果不写默认存在一个无参构造函数。

**构造函数是一个特殊的方法：**
1)、构造函数没有返回值，连 void 也不能写。
2)、构造函数的名称必须跟类名一样。
3)、没有特殊需求时，修饰符一般是 public


- **构造函数是可以有重载的。**
- 重载之后会失去默认的无参构造函数，如果需要可以显式声明以下
- 特殊写法 （构造函数的继承）较少使用  
    - 在构造函数后添加 : this (指定的重载参数)  
    - 可以实现执行该构造函数前执行 this 指定的构造函数  

```cs
public class Person
{
    //构造函数
    public Person()
    {
        ...
    }
    
    //构造函数的重载
    public Person(string name,int age)
    {
        this.Name = name;
        this.Age = age; 
    }
    
    public Person(string name,int age,string gender)
    {
        this.Name = name;
        this.Age = age; 
        this.Gender = gender;
    }

    public Person(string name,int age,string gender):this()
    {
        this.Name = name;
        this.Age = age; 
        this.Gender = gender;
    }
	

    //析构函数
    //当引用类型的堆内存被回收时，会调用该函数  
    //c#中有自动垃圾回收机制GC，所以几乎不使用析构函数  
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

**new 关键字**
Person p=new Person ();
new 帮助我们做了 3 件事儿：
1)、在内存中开辟一块空间
2)、在开辟的空间中创建对象
3)、调用对象的构造函数进行初始化对象

### this 关键字
1)、代表当前类的对象
2)、在类当中显示的调用本类的构造函数  : this （进阶）

```cs
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


## 7 索引器
**作用：** 让对象可以像数组一样通过索引访问其中元素，使程序看起来更直观，更容易编写。
**语法：** 
```cs file:语法
访问修饰符 返回值 this[参数类型 参数名, 参数类型 参数名, ......] // 注意这里是中括号[]
{
    //内部的写法和规则和索引器相同
    get{}
    set{}
}
```

```cs file:用法
public class Person
{
    private string _name;
    private int _age;
    private Person[] _friend; // 类数组，也可以是用二维数组

    public Person this[int index] //索引器
    {
        get
        {
            return _friend[index];
        }
        set
        {
             _friend[index] = value;
        }
    }
}


// 使用索引器
Person p = new Person();  
p[0] = new Person();  
p[1] = new Person();
```

索引器 this 函数中的 get 和 set 可以写逻辑，this 函数支持重载




## 8 继承

```cs
//父类（基类)
public class Person

//子类（派生类）：
public class Student : Person
```

### 特性

1. 子类继承了父类的属性和方法，不能继承父类 private 字段和构造函数。

2. **单根性**（子类只能有一个父类）和**传递性** (子类可以间接继承父类的父类))

3. 子类成员函数和父类的**同名**时，会把父类的隐藏掉。（不建议写同名成员）

![image-20220623161508887](image-20220623161508887.png)

加 new 之后不再警告

```cs
public new void SayHello() 
{}
```

4. 子类对象可以调用父类中的成员，但是父类对象永远都只能调用自己的成员。
5. 当申明一个子类对象时，**先执行父类的构造函数，再执行子类的构造函数**
    - 父类的无参构造很重要，子类实例化时默认自动调用的是父类的无参构造，所以如果父类无参构造被顶掉，会报错！
    - 子类可以通过 base 关键字代表父类调用父类构造

### base 关键字

子类写的成员函数和父类的**同名**时，会把父类的隐藏掉。

**base 关键字用于从派生类中访问基类的成员：
调用基类上已被其他方法重写的方法。
指定创建派生类实例时应调用的基类构造函数。
基类访问只能在构造函数、实例方法或实例属性访问器中进行。**

```cs
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

### 里氏替换原则

**概念：**
任何父类出现的地方，子类都可以替代

**重点：**
语法表现—父类容器装子类对象, 因为子类对象包含了父类的所有内容

**作用：**
方便进行对象存储和管理

```cs
//假定：Student类是Person类的子类
 static void Main(string[] args)
{
    //里氏替换，父类容器装子类对象
    Person p = new Student();

    //如果父类中装的是子类对象，那么可以将这个父类强转为子类对象
    Student ss = (Student)p;
    ss.StudentSayHello();
}
```

### is as 关键字
**is**：判断一个对象是否是指定的类对象，如果能够转换成功，则返回一个 true，否则返回一个 false
**as**：将一个对象转换为指定的类对象，如果能够转换则转换为指定的类对象，否则返回一个 null

```cs file:is和as的用法
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
    ss.StudentSayHello();
    
    (ss as Student).StudentSayHello(); //等价
}
```

```cs file:游戏中的应用
//假定Gameobject是其他游戏类的基类，里氏替换如下：
Gameobject player = new Player();  
Gameobject monster = new Monster();  
Gameobject boss = new Boss();  

//使用父类数组来管理子类
Gameobject[] objects = new Gameobject[] { new Player(), new Monster(), new Boss() };

//判断各自使用的逻辑
for (int i = 0; i < objects.Length; i++)
{
    if( objects[i] is Player )
    {
        (objects[i] as Player) . PlayerAtk();
    }
    
    else if( objects[i] is Monster )
    {
        (objects[i] as Monster). MonsterAtk() ;
    }
    
    else if (objects[i] is Boss)
    {
    ...
    }
    
}
```

###  万物之父 Object 类
关键字：`object`

**概念：**
object 是**所有类型的基类**，它是一个类 (引用类型)

**作用：**
1. 可以利用里氏替换原则，用 object 容器装所有对象
2. 可以用来表示不确定类型，作为函数参数类型

#### 用法
```cs file:object类
// 上文讲过的里氏替换
Father f = new Son();
if (f is Son)
{
    (f as Son).Speak();
}

//使用object类
//引用类型
object o = new Son();
if (o is Son)
{
    (o as Son).Speak();
}

//值类型
object o2 = 10.0f;     //装箱
float f2 = (float)o2;  //拆箱，强转

//string
object ostr = "123123";
string str1 = ostr.ToString();
string str2 = ostr as string; //建议引用类型都用as的方式

//数组
object oarr = new int[10];
int[] arr1 = (int[])oarr;
int[] arr2 = oarr as int[];


public class Father
{
    
}

public class Son : Father
{
    public void Speak()
    {
        Console.WriteLine("Hello world!");
    }
}
```

#### 装箱拆箱
**发生条件**
1. 用 object 存值类型（装箱)
2. 再把 object 转为值类型 (拆箱)

**装箱**
- 把值类型用引用类型存储，如 `object o = 10.0f;`
- 栈内存会迁移到堆内存中

**拆箱**
- 把引用类型存储的值类型取出来，如 `float f = (float)o;`
- 堆内存会迁移到栈内存中

**好处:** 不确定类型时可以方便参数的存储和传递
**坏处:** 存在内存迁移，增加性能消耗

##  10 多态
多态按字面的意思就是“多种状态”
让继承同一父类的子类们在执行相同方法时有不同的表现 (状态)

**主要目的**
同一父类的对象执行相同行为 (方法)有不同的表现

**解决的问题**
让同一个对象有唯一行为的特征


多态有两种：
- 编译时多态（函数重载，开始就写好的）
- 运行时多态（重写父类虚函数、抽象函数、接口）
### 虚函数 virtual

**当父类中的方法需要实现, 将父类的方法标记为虚方法，使用关键字 `virtual`**，这个函数可以被子类重写。
**子类的方法使用关键字 `override`**。

```cs
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
    
    //父类
    public class ReadDuck
    {
        public virtual void jiao()
        {
            Console.WriteLine("真的鸭子嘎嘎叫");
        }
    }

    //子类
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

### 抽象类 abstract 
被抽象关键字 `abstract` 修饰的类
**当父类中的方法不知道如何去实现的时候，可以考虑将父类写成抽象类，将方法写成抽象方法。**

特点:
1. 不能被实例化，其他封装特性都有
2. 可以包含抽象方法（即纯虚函数）
3. 继承抽象类必须重写其抽象方法

```cs
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

1. 抽象成员必须标记为 abstract, 并且不能有任何实现。
2. 抽象成员必须在抽象类中。
3. 抽象类不能被实例化
4. 子类继承抽象类后，必须把父类中的所有抽象成员都重写。（除非子类也是一个抽象类，则可以不重写）
5. 抽象成员的访问修饰符不能是 private
6. 在抽象类中可以包含实例成员。并且抽象类的实例成员可以不被子类实现
7. 抽象类是有构造函数的。虽然不能被实例化。
8. 如果父类的抽象方法中有参数，那么。继承这个抽象父类的子类在重写父类的方法的时候必须传入对应的参数。
9. 如果抽象父类的抽象方法中有返回值，那么子类在重写这个抽象方法的时候也必须要传入返回值。

**使用时机：**
1. 如果父类中的方法有默认的实现，并且父类需要被实例化，这时可以考虑将父类定义成一个普通类，用虚方法来实现多态。
2. 如果父类中的方法没有默认实现，父类也不需要被实例化，则可以将该类定义为抽象类。

## 8. 集合（Collection）

集合（Collection）类是专门用于数据存储和检索的类。这些类提供了对栈（stack）、队列（queue）、列表（list）和哈希表（hash table）的支持。大多数集合类实现了相同的接口。

集合（Collection）类服务于不同的目的，如为元素动态分配内存，基于索引访问列表项等等。这些类创建 Object 类的对象的集合。在 C# 中，Object 类是所有数据类型的基类。

各种集合类和它们的用法

下面是各种常用的 **System. Collection** 命名空间的类。点击下面的链接查看细节。

| 类                                                           | 描述和用法                                                   |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [动态数组（ArrayList）](https://www.runoob.com/csharp/csharp-arraylist.html) | 它代表了可被单独**索引**的对象的有序集合。它基本上可以替代一个数组。但是，与数组不同的是，您可以使用**索引**在指定的位置添加和移除项目，动态数组会自动重新调整它的大小。它也允许在列表中进行动态内存分配、增加、搜索、排序各项。 |
| [哈希表（Hashtable）](https://www.runoob.com/csharp/csharp-hashtable.html) | 它使用**键**来访问集合中的元素。当您使用键访问元素时，则使用哈希表，而且您可以识别一个有用的键值。哈希表中的每一项都有一个**键/值**对。键用于访问集合中的项目。 |
| [排序列表（SortedList）](https://www.runoob.com/csharp/csharp-sortedlist.html) | 它可以使用**键**和**索引**来访问列表中的项。排序列表是数组和哈希表的组合。它包含一个可使用键或索引访问各项的列表。如果您使用索引访问各项，则它是一个动态数组（ArrayList），如果您使用键访问各项，则它是一个哈希表（Hashtable）。集合中的各项总是按键值排序。 |
| [堆栈（Stack）](https://www.runoob.com/csharp/csharp-stack.html) | 它代表了一个**后进先出**的对象集合。当您需要对各项进行后进先出的访问时，则使用堆栈。当您在列表中添加一项，称为**推入**元素，当您从列表中移除一项时，称为**弹出**元素。 |
| [队列（Queue）](https://www.runoob.com/csharp/csharp-queue.html) | 它代表了一个**先进先出**的对象集合。当您需要对各项进行先进先出的访问时，则使用队列。当您在列表中添加一项，称为**入队**，当您从列表中移除一项时，称为**出队**。 |
| [点阵列（BitArray）](https://www.runoob.com/csharp/csharp-bitarray.html) | 它代表了一个使用值 1 和 0 来表示的**二进制**数组。当您需要存储位，但是事先不知道位数时，则使用点阵列。您可以使用**整型索引**从点阵列集合中访问各项，索引从零开始。 |

### ArrayList 集合（动态数组）

每次集合中实际包含的元素个数 (count)超过了可以包含的元素的个数 (capcity)的时候，集合就会向内存中申请多开辟一倍的空间，来保证集合的长度一直够用。

```cs
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

```cs
list.Remove("张三"); //指定删除单个元素
list.RemoveAt("0); //根据下标删除单个元素
list.RemoveRange(0，n); //（从下标0开始删除n个）
list.Clear(); //清空所有元素
list.Sort(); //升序排列
list.Reverse(); //反转
......
```

# 委托
# 事件
# 接口

接口是行为的抽象规范

**关键字**：`interface`

**接口声明的规范**
1. 不包含成员变量
2. 只包含方法、属性索引器、事件
3. 成员不能被实现
4. 成员可以不用写访问修饰符，不能是私有的
5. 接口不能继承类，但是可以继承另一个接口
  
**接口的使用规范**
1. 类可以继承多个接口
2. 类继承接口后，必须实现接口中所有成员


**特点:**
1. 它和类的声明类似
2. 接口是用来继承的
3. 接口不能被实例化，但是可以作为容器存储对象

```cs file:语法
public interface 接口名称（通常以I开头，如ICompute）
{
    接口成员；
}
```


### 接口的使用
类可以继承 1 个类，多个接口
继承了接口后，必须实现其中的内容，并且必须是 public 的
```cs
//接口
interface IFly
{
    string name { get; set; }

    int this[int index] { get; set; }

    event Action doSomthing;

    void Fly();
}


// 父类
public class Animal { }

//继承父类和接口
public class Person : Animal, IFly
{
    public string name { get; set; }

    public int this[int index]
    {
        get
        {
            return 0;
        }
        set
        {
        }
    }

    public event Action doSomthing;

    //实现的接口函数，可以作为虚函数继承
    public virtual void Fly()
    {

    }

//接口存储子类
static void Main(string[] args)
{
    //IFly f = new IFly();  //error
    IFly f = new Person();  // 里氏替换原则
}

```

![[f52e612d29b16288c18c4f4639d55bc5_MD5.svg]]

- 并不是所有动物都会飞，所以 Fly 放在动物父类中不合适，可以单独作为一个接口。
- 接口可以作为容器存储所以继承 Fly 的子类

### 接口可以继承接口
相当于将接口行为合并  

- 接口继承接口时，不需要实现
- 待类继承接口后，类自己去实现所有内容

### 隐式实现接口

**隐式实现接口成员是将接口的所有成员以 public 访问修饰符修饰。**

使用隐式方式来实现接口 ICompute 的成员，以计算机专业的学生类 (ComputerMajor) 实现 ICompute 接口，为其添加英语 (English)、编程 (Programming)、数据库 (Database) 学科成绩属性，代码如下。

```cs
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

```cs
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

### 显式实现接口

**显式实现接口是指在实现接口时所实现的成员名称前含有接口名称作为前缀。**

主要用于实现不同接口中的同名函数的不同表现


**使用显式实现接口的成员不能再使用修饰符修饰**，即 public、abstract、virtual、 override 等。

```cs
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
        Console.WriteLine ("总分数：" + sum);
    }
    void ICompute.Avg ()
    {
        double avg = (English + Programming + Database) / 3;
        Console.WriteLine("平均分为：" + avg);
    }
}
```

从上面的代码可以看出，在使用显式方式实现接口中的成员时，所有成员都会加上接口名称 ICompute 作为前缀，并且不加任何修饰符。

在 Main 方法中调用实现类中的成员, 代码如下

```cs
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

### 接口中多态的实现

使用接口实现多态需要满足以下两个条件。

- 定义接口并使用类实现了接口中的成员。
- 创建接口的实例指向不同的实现类对象。


假设接口名称为 ITest，分别定义两个实现类来实现接口的成员，示例代码如下。

```cs
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

# 类型转换

我们要求等号两边参与运算的操作数必须一致，如果不一致，满足下列条件会发生转换。

## **隐式类型转换**
- 低精度可以转换成高精度
- char→整数（有符号、无符号）→float→double $\nrightarrow$ decimal
- char→整数（有符号、无符号）→decimal
- string 和 bool 不参与隐式转换规则
- 有符号 $\nrightarrow$ 无符号
- 无符号→有符号（精度低到高）
## **显式（强制）类型转换**
### 括号强转
用于将高精度转换位低精度

```cs
//double->int 强制类型转换（显式类型转换）
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
### Convert 类型转换

- 如果两个变量类型不兼容，比如 string 与 int 或 string 与 double，可以使用 Convert 函数进行转换。
-  string 字符串的内容必须为要转换的类型。

`Convert.ToInt32()`
`Convert.ToDouble()`
...
```cs string a = "123";
string a = "123";  //但是只能转换int，如果a=”123abc“或“6.5”则会异常
int b = Convert.ToInt32(a);
Console.WriteLine(b);
```

### .Parse 类型转换

效果同 Convert:

`int.Parse()`
`double.Parse()`
```cs
string a = "123"; 
int b = int.Parse(a);
//double b = double.Parse(a)
```

### .TryParse 类型转换

`int.TryParse()`

```cs
int a = 0;
bool b = int.TryParse("123", out a);
//尝试将”123“转换为int类型，如果成功就将转换后的123赋值给a，并返回true给b。如果失败则返回false给b，a赋值为0。
Console.WriteLine(b);
Console.WriteLine(a);
```


# 泛型
# 枚举器和迭代器
# LINQ
# 异步编程
# 命名空间和程序集
# 异常

## 4 异常捕获

增加代码健壮性：哪行代码有可能出现异常，就 try 它。

语法：

```cs
try
{
    //可能出现异常的代码；
}
catch
{
    //出现异常后要执行的代码；
    //catch(Exception e)具体报错跟踪，通过e得到具体的错误信息
}
finally
{
    // （可选）最后执行的代码，不管有没有出错都会执行
}
//如果无异常，则catch内代码不会执行。如果出现异常，则后续代码不再执行，而是跳到catch的代码。（try-catch中间不能有其他代码）
// 注意三个语句后面不需要加;
```

# 反射和特性

