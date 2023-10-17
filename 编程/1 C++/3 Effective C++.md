---
title: 《Effective C++ 3th》
aliases: []
tags: []
create_time: 2023-04-25 22:17
uid: "202304252217"
banner: "[[Pasted image 20230317143626.jpg]]"
banner_header: 
banner_lock: true
---
# 零、导读

##  术语
###  声明式declaration
声明式：告诉编译器某个东西的名称和类型。

```c++ 
//对象声明式
extern int x;

//函数声明式
std::size_t  NumDigits(int number);

//类声明式
class Widget;

//模板声明式
template <typename T>
class GraphNode;
```

注意作者在这里把看基本类型看作对象（Obejct）。  
顺带一提，size_t 本质是 unsigned int，是一个 typedef

###  函数签名式signature

函数的声明揭示了其签名式。也就是**参数和返回值**。  
一个函数的签名等同于该函数的类型。
C++ 对于签名的官方定义中并不包含函数的返回类型，这意味着**函数的重载不能根据返回值判断**。<font color="#ff0000">（本书把返回类型视为签名的一部分）</font>

```c++
void Fun() 
{
	    
}

int Fun() //error:has already been declared with return type 'void'. 
{
    return 0;
}
```

### 定义式definition

提供给编译器一些声明式所遗漏的细节：

对对象而言，定义式是编译器为此对象分配内存的地点。  
对函数或函数模板而言，定义式提供了代码本体。  
对类或类模板而言，定义式列出了它们的成员。

### 初始化initialization

是**给对象赋初值**的过程。  

对用户自定义类型的对象而言，初始化由构造函数执行。

- `explicit`关键字 ：
可以阻止类型之间的隐式转换。  
隐式转换可能带来意想不到的问题。  
**除非必须使用隐式转换，否则就声明为`explicit`**

- `copy构造函数和copy赋值运算符`
- 比较好的方式是传`常引用`（const T&）
```c++
//重载赋值运算符(=)
class Foo
{
public:
		Foo& operator=(const Foo&); //赋值运算符
		//...
}:
```

### 接口Interface

C++没有接口，**本书中的接口指的是一般性的设计概念**，如：
- 函数的签名（signature）
- class的可访问元素（例如class的“public接口”“private接口”等）
- 针对某template类型参数需为有效的一个表达式（条款41）

## 本书命名习惯

*   Left-Hand Side：lhs  
    左手端
*   Right-Hand Side：rhs  
    右手端
*   Ptr to Class：pc 
    指向对象的指针（如果指向一个名为widget的对象，则命名为pw，以此类推）
*   Ref to Class：rc  
    指向对象的引用（如果引用一个名为widget的对象，则命名为rw，以此类推）
 -  member function：mf
成员函数
-   Constructor and Destructor：ctor和dtor
构造函数和析构函数
# 一、让自己习惯C++
## 条款01：视C++为一个语言联邦
今天的 C++ 已经是个 **多重范型编程语言**，同时支持：
- 过程形式（procedural）
- 面向对象（object-oriented）
- 函数形式（functional）
- 泛型形式（generic）
- 元编程形式（metaprogramming）

**如何理解C++？**
将C++视为一个由相关语言组成的联邦而非单一语言，某个**次语言 (sublanguage)** 中，各种守则与通例都倾向简单、直观易懂、并且容易记住。然而当你从一个次语言移往另一个次语言，守则可能改变。

**主要的次语言：**
1.  传统的面向过程 C：说到底，C++仍然以 C 为基础，区块（blocks），语句（statements），预处理器（preprocessor），内置数据类型（built-in data type），数组（arrays），指针（pointers）等都来自于 C。当你用 C++ 时，就会发现 C 语言的局限：没有模板，没有异常，没有重载…
2. 面向对象的 C++：包括构造函数，析构函数，封装，继承，多态。虚函数...
3. 模板编程 Template C++：这是关于 C++ 的泛型编程。
4. 标准库STL：是个 template 的程序库。

> [!NOTE] C++高效编程守则视情况变化，取决于你使用C++的哪一部分
> - 对于来自 C 的内置数据类型，使用值传递比引用传递高效。
> - 对于面向对象 C++、Template C++，由于用户自定义的数据类型构造函数和析构函数的存在，使用常引用传递往往更好。
> - 对于 STL，迭代器和函数都是在 C 指针之上塑造出来的，所以对 STL 的迭代器和函数对象，对于的 C 的值传递守则再次适用。

**因此，C++斌不是一个带有一组守则的一体语言，他是从四个次语言组成的联邦政府。**

## 条款02：尽量以const,enum,inline 替换 # define

> [!NOTE] 总结
>1.  对于单纯常量，最好是以 const 对象或者 enums 替换 `#define`（C++11 使用 constexpr)
>2.  对于形似函数的宏（macros），最好改用 template inline 函数替换 `#define`


### 尽量以编译器替换预处理器
 
```c++
//宏定义是预处理器指令，在编译之前的预处理阶段就被展开了。记号名称ASPECT_PATIO可能未进入记号表，导致后续难以追踪。
#define ASPECT_PATIO 1.653   

//解决方法：以常量替换上述的宏，可以被编译器看到，记入记号表内
const double AspectRatio = 1.653
```

在原书写成时 C++11 中的 `constexpr` 还未诞生，现在一般认为应当用 **`constexpr`** 定义编译期常量来替代大部分的 `#define` 宏常量定义：
[[1 C++ Primer#【C++11】constexpr 变量]]

```c++
constexpr auto aspect_ratio = 1.653;
```

**以常量替换`#define`，有两种特殊情况：**
1. **定义常量指针**  
    由于常量定义时通常放在头文件内（以便被不同的源码含入），有必要将指针本身声明为 const，即 int* const 形式。  

```c++
//若在头文件定义一个常量的 char* 的字符串，必须 const 两次：
const char*  const authorName = "Scott Meyers";

//string对象通常比char*更合适，所以上述authorName建议定义成这样：
const std::string authorName("Scott Meyers")
```
    
2. **class专属常量：static成员**
为了将常量的作用域限制在 class 内，你得让它成为 class 的一个成员，而**为了确保常量至多只有一份实体。你必须让它成为一个 static 成员。**
    
```c++
class GamePlayer
{
public:
    static const int NumTurns = 10;  //常量声明式
    static constexpr auto numTurns = 5; //C++11建议这样写
};
 ```

但是你所看到的只是常量声明式，而非定义式。**如果它是个 class 专属常量又是 static 且为整数类型（例如 ints，chars，bools），只要不取它们的地址，可以直接声明并使用它们 (无需提供定义式)。**

通常情况下 C++ 会要求对你所使用到的任何东西提供一个定义式。

```c++
// NumTurns的定义式应该放进一个实现文件（.cpp）而不是头文件(.h)
// 由于常量已在声明时获得初值，因此定义时不可以再获得初值
const int GamePlayer::NumTurns = 9;  //错误
const int GamePlayer::NumTurns;  //正确
```
    
> [!bug] static初始化时机
> 对于非const 的static数据成员，类内不提供初始值，只进行声明。在类外提供初始值
> 对于const的static数据成员，则需要在类内提供初始值。

### enum hack 补偿做法

当你在 class 编译期间需要一个 class 常量值，而编译器（错误的）不允许**static 整数型 class 常量**类内提供初始值时（存在于某些旧式编译器中的问题），则可以使用“enum hack”补偿做法
    
 ```c++
class GamePlayer
{
public:
		static const int NumTurns;  //编译器错误的不允许提供初始值
    	int Array[NumTurns]; 
    	//错误，因为编辑器坚持在编译期知道数组的大小，必须给ConstNumber初值
};

//可以使用的方法：“enum hack”
class GamePlayer
{
public:
    enum { NumTurns = 5 };
    int Array[NumTurns];
};
 ```
    
其**理论基础是：一个属于 enum 类型的数值可以充当 int 被使用**。

enum hack 的行为某方面来说比较像 `#define` 而不是 const，有时候这就是想要的。例如取一个 const 的地址是合法的，而取一个 enum 的地址是非法的，而取一个 `#define` 的地址通常也不合法。
如果不想别人获得指针或者引用指向某个整形常量，enum 可以实现这个约束。

### template inline 函数代替 define宏
 
使用`#define` 实现宏时，规定必须为宏中的所有实参加上小括号，即便如此可读性也不好。
```c++
#define CALL_WITH_MAX(a,b) f((a) > (b) ? (a) : (b))
```

并且这种方式在某些情况下不安全：
```c++
//在这里，调用f之前，a的递增次数竟然取决于“它被拿来和谁比较”！
int a = 5,b = 0;
CALL WITH MAX(++a, b);  //a被累加二次
CALL WITH MAX (++a, b+10);  //a被累加一次
```

 **使用template inline函数：更安全，更具可读性**
 ```c++
template <typename T>
inline T callWithMax(const T& a,const T& b) 
{
    f(a > b ? a : b)
}
```

需要注意的是，宏和函数的行为本身并不完全一致，宏只是简单的替换，并不涉及传参和复制。

有了 const，enum，inline，我们对于预处理器（特别是 `#define`）的需求降低了，但并非完全消除。#include 仍然是必需品，而 `#ifdef`，`#ifndef` 也继续扮演着控制编译的重要角色。目前还没到预处理器全面退出的时候，但你应该明确地慎用它。

## 条款 03：尽可能使用 const

> [!NOTE] 
> 只要某值保持不变是事实，就声明为const，编译器确保强制执行约束。

**用法：**
1. const 指针和引用：[[1 C++ Primer#const限定符]]
2. const_iterator：[[1 C++ Primer#顺序容器迭代器]]
3. const成员函数：[[1 C++ Primer#const成员函数]]

## 条款 04：确定对象使用前已先被初始化

> [!NOTE] 总结
> 1.  为内置对象进行手工初始化，因为 C++ 不保证初始化它们。
>2.  构造函数最好使用列表初始化，而不要在构造函数本体内使用赋值操作。列表初始化的成员变量，其排列顺序应该和它们在 class 中的声明次序相同。
>3. 对于类中的成员变量而言，两种方法完成初始化：类内初始化/成员初始化列表
>4.  为免除跨编译单元的初始化次序问题，请以局部 static 对象替换非局部 static 对象。

1.  读取未初始化的值会导致不明确的行为。
2.  c++ 初始化在不同语境表现不同，这些规则很复杂。
3.  **最佳处理方法**就是：**永远在使用对象之前先将它初始化**。
    - **对于无任何成员的内置类型**，必须手动完成初始化。
    - **对于非内置类型**，初始化职责落在构造函数，那么规则是：确保每一个构造函数都将对象的每一个成员初始化。
        - **规则很简单，重要的是别混淆了赋值和初始化：**
            - 构造函数中进行的是**赋值**（下图首先执行默认构造函数为变量设置初始值，然后在对他们赋予新值） ![[Pasted image 20231012153325.png]]
            - C++规定对象的成员函数的**初始化动作发生在进入构造函数本体之前**。一个比较好的写法是在构造函数**成员初始化列表**中列出所有成员变量。![[Pasted image 20231012153355.png]]
>只需要调用一次 copy 构造，通常效率更高（对于内置类型，初始化和赋值成本相同）

另外，无参构造函数也可以使用成员初始化列表
![[Pasted image 20231012154128.png]]
【C++11】 除了使用列表初始化，C++支持**类内初始值**对成员变量初始化。

```
class CTextBlock {
private:
    std::size_t textLength =  0;
    bool lengthIsValid = false;
};
```


- 如果成员变量是 `const` 或者引用类型，它们一定需要初始化，不能被赋值。


4. **不同编译单元内定义的非局部 static 对象的初始化次序。** 
    1. 函数内的 static 对象成为局部 static 对象，其他为非局部 static 对象
    2. 所谓编译单元，是指产出单一目标文件的那些源码：基本上是单一源码文件（cpp）加上其所含入的头文件 (h)。
    
     **问题在于**：如果某编译单内的某个非局部对象的初始化依赖于另一个编译单元内某个非局部对象，而这个对象可能尚未被初始化，然后就会导致未定义行为（ C++对“定义于不同编译单元内的非局部静态对象”的初始化次序并无明确定义，所以我们不能确定这个次序）。

```c++
// File 1
extern FileSystem tfs;

// File 2
class Directory {
public:
    Directory() {
        FileSystem disk = tfs;
    }
};

Directory tempDir;
```
>在上面这个例子中，你无法确保位于不同编译单元内的 `tfs` 一定在 `tempDir` 之前初始化完成。

**解决方法**：**将每一个非局部静态对象搬到自己专属函数中 (该对象在此函数内被声明为 static)，函数返回该该静态对象的引用**，然后由用户调用这些函数，而不直接涉及这些对象。换句话说非局部静态对象被局部静态对象替换了。如果熟悉设计模式，想必认出来这是 `单例模式` 常见的实现手法。

```c++
FileSystem& tfs() {
    static FileSystem fs;
    return fs;
}

Directory& tempDir() {
    static Directory td;
    return td;
}
```

**因为 C++ 保证：函数内的局部静态对象会在该函数第一次被调用时被初始化**。这样保证你获得的引用将指向一个历经初始化的对象。更好的是，如果你不调用这个函数，绝不会引发构造和析构成本。

但从多线程来看，这使得系统带有不确定性，一个比较好的做法就是：在程序的单线程启动阶段手工调用所有的单例函数。

如果你的初始化存在对象 A 初始化依赖对象 B，对象 B 的初始化又依赖于对象 A，那什么也救不了，你该避免这种病态的情况。

# 二、构造、析构和赋值运算

### 条款 05：了解 C++ 默默编写并调用了哪些函数

当 C++ 处理过 empty class(空类) 之后，如果你没有声明，则编译器会主动为**声明**一个**拷贝构造函数、拷贝复制操作符**和一个**析构函数**，同时如果你没有声明任何构造函数，编译器也会为你声明一个 **default 版本的拷贝构造函数**，这些函数都是 `public` 且 `inline` 的(见条款 30)。

```c++
//空类
class Empty {};

//编译器自动声明：
class Empty
{
public:
    Empty() {...}  //默认构造函数
    Epty(const Empty& rhs){...}  //拷贝构造函数
    ~Empty() {...}  //析构函数
    
    Empty&operator=(const Empty&rhs){...}//拷贝赋值运算符
};
```

注意，上边说的是声明，只有当这些函数有调用需求的时候，编译器才会创建它们。但是编译器替你实现的函数可能在**类内引用、类内指针、有 `const` 成员以及 virtual 类型属性**的情形下会出问题。
比如以下情况：在该类中，我们有一个 string **引用**类型，然而引用无法指向不同对象，因此编译器会拒绝为该类创建一个默认的拷贝赋值运算符。

```c++
class NamedObject {
private:
    std::string& nameValue;
};
```

除此之外，以下情形也会导致拷贝赋值运算符不会自动创建：
1.  类中含有 **const 成员** (更改 const 成员是不合法的)。
2.  基类中含有 **private 的拷贝赋值运算符**（因为要继承给子类，子类可可能无权调用基类的成员函数）。

*   对于拷贝构造函数，你要考虑到类内成员有没有**深拷贝**的需求，如果有的话就需要自己编写拷贝构造函数 / 操作符，而不是把这件事情交给编译器来做。
*   对于析构函数，如果该类有多态需求，请主动将析构函数声明为 `virtual`，默认是非 virtual 的（具体请看条款 07） 。

除了这些特殊的场景以外，如果不是及其简单的类型，请自己编写好**构造、析构、拷贝构造和赋值操作符、移动构造和赋值操作符**（C++11、如有必要）这六个函数。

### 条款 06：若不想使用编译器自动生成的函数，就该明确拒绝。

承接上一条款，如果你的类型在语义或功能上需要明确禁止某些函数的调用行为，比如禁止拷贝行为，那么你就应该禁止编译器去自动生成它。作者在这里给出了两种方案来实现这一目标：

*   将被禁止生成的函数声明为 `private` 并省略实现，这样可以禁止来自类外的调用。但是如果类内不小心调用了（成员函数、友元），那么会得到一个链接错误。 
*   将上述的可能的链接错误转移到编译期间。设计一不可拷贝的工具基类，将真正不可拷贝的基类私有继承该基类型即可，但是这样的做法过于复杂，对于已经有继承关系的类型会引入多继承，同时让代码晦涩难懂。

【**C++11】现在有了更好的做法**: [[1 C++ Primer#【C++11】 =delete删除函数]]

**我们可以直接使用 `= delete` 来删除拷贝构造函数，禁止编译器生成该函数。
``` c++
class Uncopyable {
public:
    Uncopyable(const Uncopyable&) = delete;
    Uncopyable& operator=(const Uncopyable&) = delete;
};
```
### 条款 07：为多态基类声明 virtual 析构函数

> [!NOTE] 总结
> - 多态 base classes 应该声明一个 virtual 析构函数。
> - 如果 class 带有任何 virtual 函数，它就应该拥有一个 virtual 析构函数。
> - Classes 的设计目的如果不是作为 base classes 使用，或不是为了具备多态性, 就不该声明 virtual 析构函数。

当派生类对象经由一个基类指针被删除，而该基类指针带着一个`非虚析构函数`，其结果是未定义的，可能会无法完全销毁派生类新增的成员，造成内存泄漏。**

消除这个问题的方法就是对基类使用 `虚析构函数`：它会消除整个对象
- 虚析构函数的运作方式是，最深层派生的那个类的析构函数最先被调用，然后是其上的基类的析构函数被依次调用。
```c++
class Base {
public:
    Base();
    virtual ~Base();
};
```


*   需要注意的是，普通的类无需也不应该有虚析构函数，因为虚函数无论在时间还是空间上都会有代价（额外存储的虚表指针会使类的体积变大。）例如：考虑一个用来表示 2D 坐标点的class
```c++
class point
{
public：
    Point（int x, int y）;
    ~Point();
    
private:
    int x, y;
};
```
如果 int 占用占用 32bit（32 位位计算器中），那么 Point 对象占用 64bit。如果析构函数是 virtual，那么对象必须携带**虚表指针**（vptr，virtual table pointer）指向一个由函数指针构成的数组（即虚表），用来在运行期决定哪一个 virtual 函数应该被调用。
如果 Point 类内含有 virtual，其**体积就会增加**，因为额外存储的虚表指针：32位操作系统上，指针通常是4字节（32位）大小。64位操作系统上，指针通常是8字节（64位）大小。C++标准并没有强制规定指针的大小，因此具体的实现可能会有所不同。
此外， **C++的 Point 对象也不能直接在 C 函数中使用**，因为 C 没有虚表机制，也因此**不再具有移植性**。

> [!NOTE] 虚表指针 vptr 和虚表
> 每一个带有 virtual 函数的 class 都有一个都有一个相应的 vptr。当对象调用某一 virtual 函数函数，实际被调用的函数取决于该对象的 vptr 所指所指的那个虚表——编译器在其中寻找适当的函数指针。


*   如果一个类型没有被设计成基类，又有被误继承的风险，请在类中声明为 `final`（C++ 11）[[1 C++ Primer#【C++11】防止继承 final]]，这样禁止继承可以防止误继承造成上述问题。
*   编译器自动生成的析构函数是非虚的，所以多态基类必须将析构函数**显示声明**为`virtual`。

### 条款 08：别让异常逃离析构函数

> [!NOTE] 总结
> 
> - 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。
> - 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么 class 应该提供一个普通函数（而非在析构函数中）执行该操作。


C++并不禁止析构函数吐出异常，但为了程序的可靠性，应当极力避免这种行为。
为了实现 RAII，我们通常会将对象的销毁方法封装在析构函数中，如下例子：
>RAII 是"Resource Acquisition Is Initialization"的缩写，是一种 C++编程中的重要设计原则。它是一种管理资源的策略，用于确保在对象生命周期内正确分配和释放资源，如内存、文件句柄、数据库连接等。RAII 原则的核心思想是：资源的获取应该在对象的构造阶段完成，而资源的释放应该在对象的析构阶段完成，从而确保资源的正确分配和释放，避免资源泄漏和错误使用。


```c++
class DBConn {
public:
    ...
    ~DBConn() {
        db.close();    // 该函数可能会抛出异常
    }

private:
    DBConnection db;
};
```

但这样我们就需要在析构函数中完成对异常的处理，以下是几种常见的做法：

第一种：杀死程序：

```c++
DBConn::~DBConn() {
    try { db.close(); }
    catch (...) {
        // 记录运行日志，以便调试
        std::abort();
    }
}
```

第二种：直接吞下异常不做处理，但这种做法不被建议。

第三种：重新设计接口，将异常的处理交给客户端完成：

```c++
class DBConn {
public:
    ...
    void close() {
        db.close();
        closed = true;
    }

    ~DBConn() {
        if (!closed) {
            try {
                db.close();
            }
            catch(...) {
                // 处理异常
            }
        }
    }

private:
    DBConnection db;
    bool closed;
};
```

在这个新设计的接口中，我们提供了 `close` 函数供客户手动调用，这样客户也可以根据自己的意愿处理异常；若客户忘记手动调用，析构函数才会自动调用 `close` 函数。

---

当一个操作可能会抛出需要客户处理的异常时，将其暴露在普通函数而非析构函数中是一个更好的选择。

析构函数一般情况下不应抛出异常，因为很大可能发生各种未定义的问题，包括但不限于内存泄露、程序异常崩溃、所有权被锁死等。  

一个直观的解释：析构函数是一个对象生存期的最后一刻，负责许多重要的工作，如线程，连接和内存等各种资源所有权的归还。如果析构函数执行期间某个时刻抛出了异常，就说明抛出异常后的代码无法再继续执行，这是一个非常危险的举动——因为析构函数往往是为类对象兜底的，甚至是在该对象其他地方出现任何异常的时候，析构函数也有可能会被调用来给程序擦屁股。在上述场景中，如果在一个异常环境中执行的析构函数又抛出了异常，很有可能会让程序直接崩溃，这是每一个程序员都不想看到的。  

**话说回来，如果某些操作真的很容易抛出异常，如资源的归还等，并且你又不想把异常吞掉，那么就请把这些操作移到析构函数之外，提供一个普通函数做类似的清理工作，在析构函数中只负责记录，我们需要时刻保证析构函数能够执行到底。**

### 条款 09：绝不在构造和析构过程中调用 virtual 函数

子类对象开始创建时，首先调用的是基类的构造函数，**在基类构造期间，该对象的类型是基类而不是子类**。
如果这时基类的构造函数中调用了 virtual 虚函数会被编译器解析至基类的虚函数版本，而不是子类的重载版本。若使用运行期类型信息（dynamic_cast 和 typeid）, 也会把对象是为基类对象。面对这种情况，最安全的做法就是视这种情况不存在，等子类对象调用完自身的构造函数再进行其他操作。

析构函数在调用的过程中也是这样的。

---

如果想要基类在构造时就得知派生类的构造信息，推荐的做法是在派生类的构造函数中将必要的信息向上传递给基类的构造函数：

```c++
class Transaction {
public:
    explicit Transaction(const std::string& logInfo);
    void LogTransaction(const std::string& logInfo) const;
    ...
};

Transaction::Transaction(const std::string& logInfo) {
    LogTransaction(logInfo);                           // 更改为了非虚函数调用
}

class BuyTransaction : public Transaction {
public:
    BuyTransaction(...)
        : Transaction(CreateLogString(...)) { ... }    // 将信息传递给基类构造函数
    ...

private:
    static std::string CreateLogString(...);
}
```

注意此处的 `CreateLogString` 是一个静态成员函数，这是很重要的，因为静态成员函数可以确保不会使用未完成初始化的成员变量。

### 条款 10：令 operator = 返回一个 reference to *this

简单来说：这样做可以让你的赋值操作符实现连锁赋值：

```c++
x = y = z = 10;
```

如：

```c++
class Widget
{
public:
    ...
    Widget& operator=(const Widget& rhs) //返回类型是个 reference，
    {                                    //指向当前对象。
        ...
        return *this；                   //返回左侧对象
    }
    ...
}
```

这个协议不仅适用于以上的标准赋值形式，也适用于所有**赋值**相关运算，例如：

```c++
class Widget
{
public:
    ...
    Widget& operator+=(const Widget& rhs)  //这个协议适用于
    {                                      //+=、-+、*=等等。
        ...
        return *this；                     
    }

    Widget& operator=(int rhs)             //／此函数也适用，即使
    {                                      //此一操作符的参数类型
        ...                                //不符协定。
        return *this；                   
    }
    ...
}
```

注意 **bool 操作符**重载的**返回值**有所不同，请留心，以免无限调用自身。

```c++
struct Vector2
{
    ....
    bool operator==(const Vector2& other) const  //定义操作符的重载,如果！=，这里做相应修改即可
    {
        return x == other.x && y == other.y;  //不能return *this 否则无限调用自己
    }

    bool operator！=(const Vector2& other) const  
    {
        return ！(*this == other);
    }
};
```

注意，这只是个协议，并无强制性。如果不遵循它，代码一样可通过编译。然而这份协议被所有内置类型和标准程序库提供的类型如 string,. vector, complex, trl:: shared ptr 或即将提供的类型（见条款 54）共同遵守。因此除非你有一个标新立异的好理由，不然还是随众吧。

在设计接口时一个重要的原则是，**让自己的接口和内置类型相同功能的接口尽可能相似**，所以如果没有特殊情况，就请让你的赋值操作符的返回类型为`ObjectClass&`类型并在代码中返回`*this`吧。

### 条款 11：在 operator = 中处理 “自我赋值”

自我赋值指的是将自己赋给自己。这是一种看似愚蠢无用但却在代码中出现次数比任何人想象的多得多的操作，这种操作常常需要假借指针来实现：

```
*pa = *pb;              //pa和pb指向同一对象，便是自我赋值。
arr[i] = arr[j];        //i和j相等，便是自我赋值
```

自我赋值是合法的操作，但在一些情况下可能会导致意外的错误，例如在复制堆上的资源时：

```
Widget& operator+=(const Widget& rhs) {
    delete pRes;                          // 删除当前持有的资源
    pRes = new Resource(*rhs.pRes);       // 复制传入的资源
    return *this;
}

```

但若 `rhs` 和 `*this` 指向的是相同的对象，就会导致访问到已删除的数据。

最简单的解决方法是在执行后续语句前先进行**证同测试（Identity test）**：

```
Widget& operator=(const Widget& rhs) {
    if (this == &rhs) return *this;        // 若是自我赋值，则不做任何事

    delete pRes;
    pRes = new Resource(*rhs.pRes);
    return *this;
}


```

另一个常见的做法是只关注异常安全性，而不关注是否自我赋值：

```
Widget& operator=(const Widget& rhs) {
    Resource* pOrigin = pRes;             // 先记住原来的pRes指针
    pRes = new Resource(*rhs.pRes);       // 复制传入的资源
    delete pOrigin;                       // 删除原来的资源
    return *this;
}
```

仅仅是适当安排语句的顺序，就可以做到使整个过程具有异常安全性。

还有一种取巧的做法是使用 copy and swap 技术，这种技术聪明地利用了栈空间会自动释放的特性，这样就可以通过析构函数来实现资源的释放：

```
Widget& operator=(const Widget& rhs) {
    Widget temp(rhs);
    std::swap(*this, temp);
    return *this;
}
```

上述做法还可以写得更加巧妙，就是利用按值传参，自动调用构造函数：

```
Widget& operator=(Widget rhs) {
    std::swap(*this, rhs);
    return *this;
}
```


### 条款 12：复制对象时勿忘其每一个成分

所谓 “每一个成分”，作者在这里其实想要提醒大家两点：
*   当你给类多加了成员变量时，请不要忘记在拷贝构造函数和赋值操作符中对新加的成员变量进行处理。如果你忘记处理，编译器也不会报错。  
*   如果你的类有继承，那么在你为子类编写拷贝构造函数时一定要格外小心复制基类的每一个成分，这些成分往往是 private 的，所以你无法访问它们，你应该让子类使用子类的拷贝构造函数去调用相应基类的拷贝构造函数：

```c++
class PriorityCustomer : public Customer {
public:
    PriorityCustomer(const PriorityCustomer& rhs);
    PriorityCustomer& operator=(const PriorityCustomer& rhs);
    ...

private:
    int priority;
}

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
    : Customer(rhs),                // 调用基类的拷贝构造函数
      priority(rhs.priority) {
    ...
}

PriorityCustomer::PriorityCustomer& operator=(const PriorityCustomer& rhs) {
    Customer::operator=(rhs);       // 调用基类的拷贝赋值运算符
    priority = rhs.priority;
    return *this;
}
```

除此之外，**拷贝构造函数和拷贝赋值操作符，他们两个中任意一个不要去调用另一个**，这虽然看上去是一个避免代码重复好方法，但是是荒谬的。其根本原因在于拷贝构造函数在构造一个对象——这个对象在调用之前并不存在；而赋值操作符在改变一个对象——这个对象是已经构造好了的。因此前者调用后者是在给一个还未构造好的对象赋值；而后者调用前者就像是在构造一个已经存在了的对象。不要这么做！

# 三、资源管理
**所谓资源就是，一旦用了它，将来必须还给系统**。如果不这样，糟糕的事情就会发生。C++程序中最常使用的资源就是动态分配内存（如果你分配内存却从来不曾归还它，会导致内存泄漏)，但内存只是你必须管理的众多资源之一。其他常见的资源还包括文件描述器 (file descriptors)、互斥锁 (mutex locks)、图形界面中的字型和笔刷、数据库连接、以及网络 sockets。 不论哪一种资源，重要的是，当你不再使用它时，必须将它还给系统。

### 条款 13：以对象管理资源

本条款的核心观点在于：以面向流程的方式管理资源（的获取和释放），总是会在各种意外出现时，丢失对资源的控制权并造成资源泄露。以面向过程的方式管理资源意味着，资源的获取和释放都分别被封装在函数中。这种管理方式意味着资源的索取者肩负着释放它的责任，但此时我们就要考虑一下以下几个问题：调用者是否总是会记得释放呢？调用者是否有能力保证合理地释放资源呢？不给调用者过多义务的设计才是一个良好的设计。

首先我们看一下哪些问题会让调用者释放资源的计划付诸东流：

*   一句简单的`delete`语句并不会一定执行，例如一个过早的`return`语句或是在`delete`语句之前某个语句抛出了异常。
*   谨慎的编码可能能在这一时刻保证程序不犯错误，但无法保证软件接受维护时，其他人在 delete 语句之前加入的 return 语句或异常重复第一条错误。

为了保证资源的获取和释放一定会合理执行，我们把获取资源和释放资源的任务封装在一个对象中。当我们构造这个对象时资源自动获取，当我们不需要资源时，我们让对象析构。这便是 “Resource Acquisition Is Initialization; RAII” 的想法，因为我们总是在获得一笔资源后于同一语句内初始化某个管理对象。无论控制流如何离开区块，一旦对象被销毁（比如离开对象的作用域）其析构函数会自动被调用。

具体实践请参考 C++11 的`shared_ptr`。

### 条款 14：在资源管理类中小心 copying 行为

*   复制 RAII 对象必须一并复制它所管理的资源，所以资源的 copying 行为决定 RAII 对象的 copying 行为。  
    
*   如果对想要自行管理 delete（或其他类似行为如上锁 / 解锁）的类处理复制问题，有以下方案，先创建自己的资源管理类，然后可选择：

1. 禁止复制，使用条款 6 的方法  
2. 对复制的资源做引用计数（声明为 shared_ptr），shared_ptr 支持初始化时自定义删除函数（auto_ptr 不支持，总是执行 delete）  
3. 做真正的深复制  
4. 转移资源的拥有权，类似 auto_ptr，只保持新对象拥有。

### 条款 15：在资源管理类中提供对原始资源的访问

*   许多 APIs 需要直接引用资源，而不是通过资源管理类。虽然它们不合规范，但很难避免使用它们。所以资源管理类应该提供访问原始资源的能力。
*   tr1::shared_ptr 和 auto_ptr 都提供一个 get 成员函数，用来执行显式转换，也就是它会返回：智能指针内部的原始指针的副本。
*   tr1::shared_ptr 和 auto_ptr 也重载了 operator-> 和 operator*，允许隐式转换至原始指针。
*   对于非智能指针管理类，可以编写 get() 函数，提供显式转换，重载 operator xxx()，提供隐式转换。
*   记住隐式转换可能存在问题，虽然它比较直观。

总结

1.  **APIs 往往要求访问原始资源（raw resources），所以每一个 RAII class 应该提供一个取得其所管理资源的方法。**
2.  **对原始资源的访问可能经由显式转换或隐式转换。一般而言显式转换比较安全，但隐式转换对客户比较方便。**

### 条款 16：成对使用 new 和 delete 时要采取相同形式

*   当你使用 new（也就是通过 new 动态生成一个对象），有两件事发生。

*   第一，内存被分配出来（通过名为 operator new 的函数，见条款 49 和条款 51）。
*   第二，针对此内存会有一个（或更多）构造函数被调用。

*   当你使用 delete，也有两件事发生：针对此内存会有一个（或更多）析构函数被调用，然后内存才被释放（通过名为 operator delete 的函数，见条款 51）。
*   如果你调用 new 时使用 []，你必须在对应调用 delete 时也使用 []。如果你调用 new 时没有使用 []，那么也不该在对应调用 delete 时使用 []。
*   对于数组，不建议使用 typedef 行为，这会让使用者不记得去 delete []。对于这种情况，建议使用 string 或者 vector。

### 条款 17：以独立语句将 newed 对象放入智能指针

假设有两个函数`priority`和`processWight`，其中`priority`函数返回处理程序的优先级，`processWidget`函数按照`priority`返回的优先级处理动态分配的`Widget`对象，函数原型如下：

```
int priority();
void processWidget(std::shared_ptr<Widget> pw, int priority);
```

如果按照如下方法调用`processWidget`函数，则有可能造成资源泄漏：

```
processWidget(std::shared_ptr<Widget>(new Widget()), priority());
```

这是因为编译器在生成`processWidget`函数调用码之前会核算即将被传递的各个实参，第二个实参是对`priority`函数的简单调用，而第一个实参包含两个部分：

1.  执行`new Widget()`表达式动态创建`Widget`对象。
2.  调用`shared_ptr`类的构造函数并使用`Widget`对象的指针作为构造参数。

所以，在调用`processWidget`函数之前编译器会做以下三件事情：

1.  执行`new Widget()`表达式动态创建`Widget`对象。
2.  调用`shared_ptr`类的构造函数并使用`Widget`对象的指针作为构造参数。
3.  调用`priority`函数生成优先级。

以何种顺序执行以上三个步骤，C++ 语句并没有给出严格规定（Java 和 C# 中必须按规定的顺序完成参数的核算），具体由编译器来决定。

能够明确知道的是执行`new Widget`表达式肯定是在调用`shared_ptr`构造函数之前，但调用`priority`函数则有可能是在第 1、2、3 中任意一步执行。假设调用`priority`函数在第 2 步执行，将获得如下执行顺序：

1.  执行`new Widget()`表达式动态创建`Widget`对象。
2.  调用`priority`函数生成优先级。
3.  调用`shared_ptr`类的构造函数并使用`Widget`对象的指针作为构造参数。

如果在调用`priority`函数的过程中发生了异常，那么`new Widget()`表达式返回的指针会被遗失，就有可能造成资源泄漏。原因是在【资源被创建】和【资源被管理对象接管】之间造成了异常干扰。

**解决办法：**

分离构造`Widget`对象的语句，在单独的语句中执行`new Widget()`表达式并调用`shared_ptr`类的构造函数，最后将智能指针传给`processWidget`函数。代码如下：

```
std::shared_ptr<Widget> pw(new Widget());
processWidget(pw, priority());
```

编译器对于**跨越语句的各项操作**没有重新排列的自由，只有在语句内才拥有某种自由度。

因此在上述代码中【（1）执行`new Widget()`表达式和（2）调用`shared_ptr`类的构造函数】与【对`priority`函数的调用】是在不同的语句中，被分隔开来了，所以编译器不得在它们之间任意选择执行次序。

**结论**

1.  以独立语句将 newed 对象存储于智能指针中，这样能够保证动态获取的资源一定能被资源管理对象接管，不会造成内存泄漏。
2.  如果不这样做，一旦在【资源申请成功】和【资源管理对象接管资源】之间抛出了异常，就有可能产生难以察觉的资源泄漏。因为异常本身就在意料之外的错误，不容易复现，从而导致资源泄漏无法轻易定位。

# 四、设计与声明

### 条款 18：让接口容易被正确使用，不易误使用

本条款告教你如何**帮助你的客户在使用你的接口时避免他们犯错误**。  
在设计接口时，我们常常会错误地假设，接口的调用者**拥有某些必要的知识来规避一些常识性的错误**。但事实上，接口的调用者并不总是像正在设计接口的我们一样 “聪明” 或者知道接口实现的”内幕信息“，结果就是，我们错误的假设使接口表现得不稳定。这些不稳定因素可能是由于调用者缺乏某些先验知识，也有可能仅仅是代码上的粗心错误。接口的调用者可能是别人，也可能是未来的你。所以一个合理的接口，应该尽可能的从**语法层面**并在**编译之时运行之前**，帮助接口的调用者规避可能的风险。

如下， 设计一个日期类

```
//参考原文：https://blog.csdn.net/hualicuan/article/details/27526033
class Date
{
public:
    Date(const int month, const int day, int const year) 
        : m_month(month)
        , m_day(day)
        , m_year(year)
    {

    }
private:
    int m_day;
    int m_month;
    int m_year;
};
```

错误调用

```
Date d1(29, 5, 2014);  //调用顺序错乱，应该是 5, 29, 2014
    Date d2(2, 30, 2014);  //传入参数有误，2月没有30号
```

*   使用**外覆类型（wrapper）**提醒调用者传参错误检查，将参数的附加条件限制在**类型本身**

当调用者试图传入数字 “13” 来表达一个 “月份” 的时候，你可以在函数内部做运行期的检查，然后提出报警或一个异常，但这样的做法更像是一种责任转嫁——调用者只有在尝试过后才发现自己手残把 “12” 写成了 “13”。如果在设计参数类型时就把“月份” 这一类型抽象出来，比如使用 enum class（强枚举类型），就能帮助客户在编译时期就发现问题，把参数的附加条件限制在类型本身，可以让接口更易用。

```
struct Day
{
    explicit Day(const int day) : m_day(day) {}
private:
    int m_day;
};

struct Month
{
    explicit Month(const int month) : m_month(month) {}
private:
    int m_month;
};

struct Year
{
    explicit Year(const int year) : m_year(year) {}
private:
    int m_year;
};

class Date
{
public:
    Date(const Month &month, const Day &day,  const Year &year) 
        : m_month(month)
        , m_day(day)
        , m_year(year)
    {

    }
private:
    Day m_day;
    Month m_month;
    Year m_year;
};
```

类型错误得到预防，但值还是没有得到保障

```
Date d2(2, 30, 2014);  //error,类型错误
    Date d3(Day(30), Month(2), Year(2014)); //error,类型错误
    Date d4(Month(2), Day(30), Year(2014)); //ok
```

可通过设计对应的类型的值限制来达到

```
struct Month
{
    enum E_MON{JAN = 1, FEC, MAR, APR, MAY, JUN, JUL, AGU, SEP, OCT, NOV, DEC};
    explicit Month(const E_MON month) : m_month(month) {}
private:
    int m_month;
};
```

调用

```
Date d4(Month(Month::E_MON::DEC), Day(30), Year(2014)); //ok
```

*   从**语法层面**限制调用者**不能做的事**

接口的调用者往往无意甚至没有意识到自己犯了个错误，所以接口的设计者必须在语法层面做出限制。一个比较常见的限制是加上`const`，比如在`operate*`的返回类型上加上`const`修饰，可以防止无意错误的赋值`if (a * b = c)`。

*   接口应表现出与内置类型的一致性

让自己的类型和内置类型的一致性，比如自定义容器的接口在命名上和 STL 应具备一致性，可以有效防止调用者犯错误。或者你有两个对象相乘的需求，那么你最好重载`operator*`而并非设计名为”multiply” 的成员函数。

*   从语法层面限制调用者**必须做的事**

**别让接口的调用者总是记得做某些事情**，接口的设计者应在假定他们**总是忘记**这些条条框框的前提下设计接口。比如用智能指针代替原生指针就是为调用者着想的好例子。如果一个核心方法需要在使用前后设置和恢复环境（比如获取锁和归还锁），更好的做法是将设置和恢复环境设置成纯虚函数并要求调用者继承该抽象类，强制他们去实现。在核心方法前后对设置和恢复环境的调用，则应由接口设计者操心。

当方法的调用者（我们的客户）责任越少，他们可能犯的错误也就越少。

请记住：

①好的接口易于正确使用，而难以错误使用。你应该在你的所有接口中为这个特性努力。 ②使易于正确使用的方法包括在接口和行为兼容性上与内建类型保持一致。 ③预防错误的方法包括创建新的类型，限定类型的操作，约束对象的值，以及消除客户的资源管理职责。 ④tr1::shared_ptr 支持自定义 deleter。这可以防止 cross-DLL 问题，能用于自动解锁互斥体（参见 Item 14）

### 条款 19：设计 class 犹如设计 type

如何设计 class：

*   对象该如何创建销毁：包括构造函数、析构函数以及 new 和 delete 操作符的重构需求。
*   对象的构造函数与赋值行为应有何区别：构造函数和赋值操作符的区别，重点在资源管理上。
*   对象被拷贝时应考虑的行为：拷贝构造函数。
*   对象的合法值是什么？最好在语法层面、至少在编译前应对用户做出监督。
*   新的类型是否应该复合某个继承体系，这就包含虚函数的覆盖问题。
*   新类型和已有类型之间的隐式转换问题，这意味着类型转换函数和非 explicit 函数之间的取舍。
*   新类型是否需要重载操作符。
*   什么样的接口应当暴露在外，而什么样的技术应当封装在内（public 和 private）
*   新类型的效率、资源获取归还、线程安全性和异常安全性如何保证。
*   这个类是否具备 template 的潜质，如果有的话，就应改为模板类。

### 条款 20：宁以 pass-by-reference-to-const 替换 pass-by-value

函数接口应该以`const`引用的形式传参，而不应该是按值传参，否则可能会有以下问题：

*   按值传参涉及大量参数的复制，这些副本大多是没有必要的。
*   如果拷贝构造函数设计的是深拷贝而非浅拷贝，那么拷贝的成本将远远大于拷贝某几个指针。
*   对于多态而言，将父类设计成按值传参，如果传入的是子类对象，仅会对子类对象的父类部分进行拷贝，即部分拷贝，而所有属于子类的特性将被丢弃，造成不可预知的错误，同时虚函数也不会被调用。（对象切割）
*   小的类型并不意味着按值传参的成本就会小。首先，类型的大小与编译器的类型和版本有很大关系，某些类型在特定编译器上编译结果会比其他编译器大得多。小的类型也无法保证在日后代码复用和重构之后，其类型始终很小。

请记住：

*   **尽量以 pass-by-reference-to-const 替换 pass-by-value**。前者通常比较高效，并可避免切割问题（slicing problem）。
*   以上规则**并不适用于内置类型，以及 STL 的选代器和函数对象**。对它们而言，pass-by-value 往往比较适当。

### 条款 21：必须返回对象时，别妄想返回其 reference

这个条款的核心观点在于，不要把总返回值写成引用类型，作者在条款内部详细分析了各种可能发生的错误，无论是返回一个 stack 对象还是 heap 对象，在这里不再赘述。作者最后的结论是，如果必须按值返回，那就让他去吧，多一次拷贝也是没办法的事，最多就是指望着编译器来优化。

但是对于 C++11 以上的编译器，我们可以采用给类型编写 “转移构造函数” 以及使用`std::move()`函数更加优雅地**消除由于拷贝造成的时间和空间的浪费。**

请记住

*   绝不要返回 pointer 或 reference 指向一个 local stack 对象，或返回 reference 指向一个 heap-allocated 对象，或返回 pointer 或 reference 指向一个 local static 对象而有可能同时需要多个这样的对象。条款 4 已经为 “在单线程环境中合理返回 reference 指向一个 local static 对象” 提供了一份设计实例。

### 条款 22：将成员变量声明为 private

先说结论——**请对 class 内所有成员变量声明为`private`，`private`意味着对变量的封装。**但本条款提供的更有价值的信息在于不同的属性控制——`public`,`private`和`protected`——**代表的设计思想**。

简单的来说，把所有成员变量声明为 private 的好处有两点。首先，所有的变量都是 private 了，那么所有的 public 和 protected 成员都是函数了，用户在使用的时候也就无需区分，这就是语法一致性；其次，对变量的封装意味着，**可以尽量减小因类型内部改变造成的类外外代码的必要改动。**

一旦所有变量都被封装了起来，外部无法直接获取，那么所有类的使用者（我们称为客户，客户也可能是未来的自己，也可能是别人）想利用私有变量实现自己的业务功能时，就**必须通过我们留出的接口**，这样的接口便充当了一层缓冲，将类型内部的升级和改动尽可能的对客户不可见——**不可见就是不会产生影响**，不会产生影响就不会要求客户更改类外的代码。因此，一个设计良好的类在内部产生改动后，对整个项目的影响只应是**需要重新编辑而无需改动类外部的代码**。

我们接着说明，**`public`和`protected`属性在一定程度上是等价的**。一个自定义类型被设计出来就是供客户使用的，那么客户的使用方法无非是两种——**用这个类创建对象**或者**继承这个类以设计新的类**——以下简称为第一类客户和第二类客户。那么从封装的角度来说，一个`public`的成员说明了**类的作者决定对类的第一种客户不封装此成员**，而一个`protected`的成员说明了**类的作者对类的第二种客户不封装此成员**。也就是说，当我们把类的两种客户一视同仁了以后，`public`、`protected`和`private`三者反应的即类设计者对类成员封装特性的不同思路——对成员封装还是不封装，如果不封装是对第一类客户不封装还是对第二类客户不封装。

请记住：

*   切记将成员变量声明为 private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供 class 作者以充分的实现弹性。
*   protected 并不比 public 更具封装性。

### 条款 23：宁以 non-member, non-friend 替换 member 函数

我宁愿多花一些口舌在这个条款上，一方面因为它真的很重要，另一方面是因为作者并没有把这个条款说的很清楚。

在一个类里，我愿把**需要直接访问 private 成员的 public 和 protected 成员函数**称为**功能颗粒度较低的函数**，原因很简单，他们涉及到对 private 成员的直接访问，说明他们处于封装表面的第一道防线。由若干其他 public（或 protected）函数集成而来的 public 成员函数，我愿称之为**颗粒度高的函数**，因为他们集成了若干颗粒度较低的任务，这就是本条款所针对的对象——那些**无需直接访问 private 成员**，而只是**若干 public 函数集成而来的 member 函数**。本条款告诉我们：这些函数应该尽可能放到类外。

```
class WebBrowser {          //  一个浏览器类
public:
    void clearCache();      // 清理缓存，直接接触私有成员
    void clearHistory();    // 清理历史记录，直接接触私有成员
    void clearCookies();    // 清理cookies，直接接触私有成员

    void clear();           // 颗粒度较高的函数，在内部调用上边三个函数，不直接接触私有成员，本条款告诉我们这样的函数应该移至类外
}
```

如果高颗粒度函数设置为类内的成员函数，那么一方面他会破坏类的封装性，另一方面降低了函数的包裹弹性。

1.  类的封装性

封装的作用是尽可能减小被封装成员的改变对类外代码的影响——我们希望类内的改变只影响有限的客户。一个量化某成员封装性好坏的简单方法是：看类内有多少（public 或 protected）函数直接访问到了这个成员，这样的函数越多，该成员的封装性就越差——该成员的改动对类外代码的影响就可能越大。回到我们的问题，高颗粒度函数在设计之时，设计者的本意就是**它不应直接访问任何私有成员**，而只是公有成员的简单集成，这样会最大程度维护封装性，但很可惜，**这样的愿望并没有在代码层面体现出来**。这个类未来的维护者（有可能是未来的你或别人）很可能忘记了这样的原始设定，而在此本应成为 “高颗粒度” 函数上大肆添加对私有成员的直接访问，这也就是为什么封装性可能会被间接损坏了。但设计为非成员函数就从语法上避免了这种可能性。

1.  函数的包裹弹性与设计方法

将高颗粒度函数提取至类外部可以允许我们从**更多维度组织代码结构**，并**优化编译依赖关系**。我们用上边的例子说明什么是 “更多维度”。`clear（）`函数是代码的设计者最初从浏览器的角度对低颗粒度函数做出的集成，但是如果从 “cache”、“history”、和“cookies” 的角度，我们又能够做出其他的集成。比如将 “搜索历史记录” 和“清理历史记录”集成为 “定向清理历史记录” 函数，将 “导出缓存” 和“清理缓存”集成为 “导出并清理缓存” 函数，这时，我们在浏览器类外做这样的集成会有更大的自由度。通常利用一些工具类如`class CacheUtils`、`class HistoryUtils`中的 static 函数来实现；又或者采用不同 namespace 来明确责任，将不同的高颗粒度函数和浏览器类纳入不同 namespace 和头文件，当我们使用不同功能时就可以 include 不同的头文件，而不用在面对 cache 的需求时不可避免的将 cookies 的工具函数包含进来，降低编译依存性。这也是`namespace`可以跨文件带来的好处。

*   **命名空间可以跨越多个源码文件而类则不可以。**

```
//在C++中，比较自然的做法是让clearBrowser()函数成为一个non-member函数并且位于WebBrowser类所在的同一个命名空间(namespace)中。
namespace WebBrowserStuff {	
   class WebBrowser {   //核心机能
       public :
           void clearCache();
           void clearHistory();
           void clearCookies();
       };

       // non-member函数，提供几乎所有客户都需要的核心机能
       void CoreFunc(WebBrowser& wb) {
           wb.clearCache();
           wb.clearHistory();
           wb.clearCookies();
       }
}
```

*   一个像 WebBrowser 这样的类中可能有大量的便利函数，如书签便利函数、打印便利函数、cookies 管理有关的便利函数。通常，大多数客户只对其中某些感兴趣。为了防止多个便利函数之间发生编译相互依赖性，分离它们的最直接方法是将书签便利函数声明在一个头文件中，将 cookies 管理有关的便利函数声明在另一个头文件中，再将打印便利函数声明于第三个头文件中。如下所示：

```
// 头文件webbrowser.h，这个头文件针对WebBrowser类
namespace WebBrowserStuff{
   class WebBrowser{
       // ...
   };
   // ...   non-member函数
}
// 头文件webbrowserbookmarks.h
namespace WebBrowserStuff{
   // ...   与书签相关的便利函数
}
// 头文件webbrowsercookies.h
namespace WebBrowserStuff{
   // ...   与cookies管理相关的便利函数
}
```

最后要说的是，本条款讨论的是那些**不直接接触私有成员的函数**，如果你的 public(或 protected) 函数必须直接访问私有成员，那请忘掉这个条款，因为把那个函数移到类外所需做的工作就比上述情况远大得多了。

### 条款 24：若所有参数皆需类型转换，请为此采用 non-member 函数

这个条款告诉了我们**操作符重载被重载为成员函数和非成员函数的区别**。作者想给我们提个醒，如果我们在使用操作符时**希望操作符的任意操作数都可能发生隐式类型转换**，**那么应该把该操作符重载成非成员函数。**

我们首先说明：如果一个操作符是成员函数，那么它的**第一个操作数（即调用对象）不会发生隐式类型转换**。

首先简单讲解一下当操作符被重载成员函数时，第一个操作数特殊的身份。操作符一旦被设计为成员函数，它在被使用时的特殊性就显现出来了——单从表达式你无法直接看出**是类的哪个对象在调用这个操作符函数**，不是吗？例如下方的有理数类重载的操作符”*”，当我们在调用`Rational z = x * y;`时，调用操作符函数的对象并没有直接显示在代码中——这个操作符的`this`指针指向`x`还是`y`呢？

```
class Rational {
public:
  //...
  Rational operator*(const Rational rhs) const; 
pricate:
  //...
}
```

作为成员函数的操作符的第一个隐形参数”`this`指针” 总是指向第一个操作数，所以上边的调用也可以写成`Rational z = x.operator*(y);`，这就是操作符的**更像函数的调用方法**。那么，做为成员函数的操作符默认操作符的第一个操作数应当是正确的类对象——**编译器正式根据第一个操作数的类型来确定被调用的操作符到底属于哪一个类的**。因而第一个操作数是不会发生隐式类型转换的，第一个操作数是什么类型，它就调用那个类型对应的操作符。

我们举例说明：当`Ratinoal`类的构造函数允许`int`类型隐式转换为`Rational`类型时，`Rational z = x * 2;`是可以通过编译的，因为操作符是被`Rational`类型的`x`调用，同时将`2`隐式转换为`Ratinoal`类型，完成乘法。但是`Rational z = 2 * x;`却会引发编译器报错，因为由于操作符的第一个操作数不会发生隐式类型转换，所以加号 “*” 实际上调用的是`2`——一个`int`类型的操作符，因此编译器会试图将`Rational`类型的`x`转为`int`，这样是行不通的。

因此在你编写诸如加减乘除之类的（但不限于这些）操作符、并假定允许每一个操作数都发生隐式类型转换时，**请不要把操作符函数重载为成员函数**。因为当第一个操作数不是正确类型时，可能会引发调用的失败。解决方案是，**请将操作符声明为类外的非成员函数**，你可以选择友元让操作符内的运算更便于进行，也可以为私有成员封装更多接口来保证操作符的实现，这都取决于你的选择。

```
class Rational {
public:
  Rational (int numerator = 0, int denminator = 1);
  // 下面的重载会导致错误(1)
  const Rational operator* (const Rational& rhs) {
    return Rational(n * rhs.n, d * rhs.d);
  }
  int numerator() const { return n; }
  int denminator() const { return d; }
private:
  int n, d; // 分子、分母
}
Rational A(1,8);
Rational result =  A * 2; // ok
result = 2 * a; // wrong (1)

const Rational operator* (const Rational& rhs, const Ratioanal& lhs) {
  // return Rational(rhs.n * lhs.n, rhs.d * lhs.d); // 私有成员不能访问
  return Rational(rhs.numerator() * lhs.numerator(), // ok
                  rhs.denminator() * lhs.denminator());
}

Rational result =  A * 2;  // ok
result = 2 * a; // wrong (1) // ok
```

希望这一条款能解释清楚操作符在作为成员函数与非成员函数时的区别。此条款并没有明确说明该法则只适用于操作符，但是除了操作符外，我实在想不到更合理的用途了。

题外话：如果你想禁止隐式类型转换的发生，请把你每一个单参数构造函数后加上关键字`explicit`。

### 条款 25: 考虑写出—个不抛异常的 swap 函数

[参考链接 1：https://www.cnblogs.com/wuchanming/p/3735189.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/wuchanming/p/3735189.html)

[参考原文 2：https://www.cnblogs.com/wuchanming/p/3735410.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/wuchanming/p/3735410.html)

# 五、实现

### 条款 27: 尽量少敝转型动作

**1.C 和 C++ 的三种形式的转型语法**

(1) 形式一：C 语言风格的转型语法：

```
(T)expression     //将expression转换为T类型
```

(2) 形式二: 函数风格的转型：

```
T(expression)     ////将expression转换为T类型
```

(3) 形式三：C++ 风格的转型语法

*   `const_cast(expression);//const->non const` const_cast 用来将对象的 const 属性去掉, 功能单一, 使用方便, 呵呵.
*   `dynamic_cast(expression);` dynamic_cast 用于继承体系下的 "向下安全转换", 通常用于将基类对象指针转换为其子类对象指针, 它也是唯一一种无法用旧式转换进行替换的转型, 也是唯一可能耗费重大运行成本的转型动作.
*   `reinterpret_cast(expression);` 低级转型, 结果依赖与编译器, 这因为着它不可移植, 我们平常很少遇到它, 通常用于函数指针的 转型操作.
*   `static_cast(expression);` static_cast 用来进行强制隐式转换, 我们平时遇到的大部分的转型功能都通过它来实现. 例如将 int 转换为 double, 将 void * 转换为 typed 指针, 将 non-const 对象转换为 const 对象, 反之则只有 const_cast 能够完成.

注意：形式一、二并无差别，统称旧式转型，形式三称为新式转型。

**2. 新式转型的优点**

*   在代码这种容易被识别出来 (无论是人工识别还是使用工具如 grep)，因而简化“找出类型系统在哪个点被破坏” 的过程(简化找错的过程)。  
    
*   各种转型动作的目标越窄化，编译器越能判断出出错的运用。例如：如果你打算将常量性去掉，除非使用新式转型的 const_cast 否则无法通过编译  
    

**3. 旧式转型的唯一适用场景**

我唯一使用旧式转型的时机是，当我调用一个 explicit 构造函数将一个对象传递给一个函数时。例如：

```
class Widget{
    public:
        explicit Widget(int size);
        ...
    };
    void doSomething(Widget& w);
    doSomething(Widget(15)); //"旧式转型"中的函数转型
    doSomething(static_cast<Widget>(15));//"新式转型"
```

消除一个误解：转型动作其实什么也没做

(1) 误解  
请不要认为转型什么都没做，其实就是告诉编译器把某种类型视为另一种类型。实际上，任何一种转型动作往往真的令编译器额外地编译出运行期间执行的代码。 例如将 int 转型为 double 就会发生这种情况, 因为在大部分的计算器体系结构中, int 的底层表述不同于 double 的底层表述。  
(2) 转型动作导致编译器在执行期间编译出不同的码的另外一个例子  
**单一的对象可能拥有一个以上的地址** (例如:" 以 base_指向它 "时的地址和" 以 Derived_指向它 " 时的地址这时会有一个偏移量在运行期间施加在 Derived_身上，用以取得正确的 base_的指针值）。实际上一旦使用多重继承, 这事几乎一直发生。即使在单一继承中也可能发生。  
(3)我们应该避免作做出 “对象在 C++ 中如何布局” 的假设  
有了偏移量这个经验后，我们也不能做出 “对象在 C++ 中如何布局” 的假设。因为对象的布局方式和它们的地址计算发式随着编译器的不同而不同, 这就以为着写出 "根据对象如何布局" 而写出的转型代码在某一平台上行得通, 在其它平台上则不一定。很多程序员历经千辛万苦才学到这堂课。

**4. 转型动作容易写出似是而非的代码**

有些场景下，需要在派生类的 virtual 函数中调用基类的版本的次函数：

```
class Window{
    public:
        virtual void onResize(){...}
        ...
    };
    class SpecialWindow:public Window{
    public:
        virtual void onResize(){
            static_cast<Window>(*this).onResize();//调用基类的实现代码
            ... //这里进行SpecialWindow的专属行为.
        }
        ...
    };
```

上述代码看着似乎合情合理，但是实际却是错误的。错在转型语句。为什么错呢？首先它确实执行了多态，调用的函数版本是正确的，但是由于做了转型，它并没有真正作用在派生类对象身上，而是作用在了派生类对象的基类部分的副本身上，改动的是副本。但是如果改动当前对象的派生类部分的话，不做转型动作就真的改变了当前对象的派生类部分。但是导致的最终结果就是：当前对象的基类部分没有被改动，但是派生类部分缺被改动了。

上述代码的正确写法：

```
void SpecialWindow::onResize(){
        Window::onResize(); //此时才是真正的调用基类部分的onResize实现.
        ...     //同上
    }
```

**5. 何时需要 dynamic_cast，以及避 dynamic_cast 的方法**

(1) 何时需要 dynamic_cast?

通常当你想在一个你认定为 derived class 对象上执行 derived class 操作函数时，但是你的手上只有一个指向 base 的指针或引用时，你会想到使用 dynamic_cast 进行转型

(2) 如何不做转型，实现上述需求？

通常有两种做法可以解决上述问题：

*   方法一：使用容器，并在其中存储直接指向 derived class 对象的指针 (通常是智能指针)，这样就避免了上述需求。  
    
*   方法二：在 base class 内提供 virtual 函数做你想对各个派生类想做的事情。这样可以使得你通过 base class 接口处理 “所有可能之各种派生类”。

6. **绝对避免一连串的 dynamic_cast**

一连串 dynamic_cast 的代码又大又慢，而且基础不稳，因为每次继承体系一有改变，所有这种代码必须再次进行检查看看是否需要修改。例如假如新的派生类，就要加新的分支。这样的代码应该使用 “基于 virtual 函数调用” 的东西取而代之。

**总结：**

*   如果可以，尽量避免转型，特别是在注重效率的代码中避免 dynamic_casts。如果有个设计需要转型动作，试着发展无需转型的替代设计。
*   如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内。
*   宁可使用 C++-style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。

### 条款 28：避免返回 handles 指向对象内部成分

reference、指针、迭代器系统都是所谓的 handles(号码牌，用来获得某个对象)。函数返回一个 handle，随之而来的便是 “减低对象封装性” 的风险。它也可能导致：虽调用 const 成员函数却造成对象状态被更改的风险。

如果返回 handles 指向对象内部成分，则可能带来：自相矛盾与虚吊问题

```
class Point{
public:
       Point(intx,inty);
       voidsetX(intnewVal);
       voidsetY(intnewVal);
};

struct RectData{
       Pointulhc;
       Pointlrhc;
};

class Rectangle{
public:
       Point&upperLeft()const{returnpData->ulhc;}
       Point&lowerRight()const{returnpData->lrhc;}
       ...
private:
       std::tr1::shared_ptr<RectData>pData;
};
```

point 类是一个代表点的类，RectData 代表一个矩形的结构，Rectangle 类则代表一个矩形，该类能够返回表示矩阵的左上和右下的两个点。由于这两个函数为 const 的，因此所要表达的意思就是，返回矩阵的两个点，但是不能修改他们。但是又由于返回的是点的 reference 形式，因此通过 reference，实际是可以改变返回的点的数据的。因此，造成了自相矛盾。问题的原因就是，函数返回了 handle。  
由此可知：  
1. **成员变量的封装性最多等于 “返回其 reference” 的函数的访问级别。**即使数据本身被声明为 private 的，但是如果返回他们的 reference 是 public 的，那么数据的访问权限就编程 public 了。 2. 如果 const 成员函数传出一个 reference，后者所指数据又不在自身对象内，那么这个函数的调用者可以修改此数据。(这是 bitwise constness 带来的后果。)

改进：返回 **const handles** 指向对象内部成分。则此时虽然解决了自相矛盾，却可能形成**虚吊问题**

```
class Rectangle{
public:
       //加const。这便解决了自相矛盾问题
       const Point&upperLeft()const{returnpData->ulhc;} 
       const Point&lowerRight()const{returnpData->lrhc;}
       ...
private:
       std::tr1::shared_ptr<RectData>pData;
};
```

综上，无论返回的 handle 是指针、reference、或者迭代器，也无论他是否为 const。只要一个 handle 被传出去了，都是比较危险的。

总结：

*   避免返回 handles（包括 references、指针、选代器）指向对象内部。遵守这个条款可增加封装性，帮助 const 成员函数的行为像个 const，并将发生 “虚吊号码牌”（dangling handles）的可能性降至最低。

### 条款 29: 为 “异常安全” 而努力是值得的

1.  异常安全的两个条件：当异常抛出时，①不泄漏任何资源；②不允许数据败坏；
2.  解决资源泄漏：使用 “资源管理类”；
3.  异常安全函数提供以下三个保证之一：①基本承诺：保证对象和数据结构，不保证程序状态；②强烈保证：程序状态不改变；③不抛掷承诺：总能完成功能（作用于内置类型上的所有操作都可做到）；
4.  为做到强烈保证：①见第 2 条；②重新排列对象内语句次序的策略：不要为了表示某件事情发生而改变对象状态，除非那件事情真的发生了；
5.  copy and swap 策略往往可以实现强烈保证：在对象的副本上完成修改后，再与原对象在一个不抛出异常的操作中置换（pimpl idiom 通过指针实现此策略）；
6.  “强烈保证” 并非对所有函数都可实现或具备现实意义；
7.  异常安全保证具有木桶效应；

总结

1.  资源管理类不仅防止了资源泄漏，同时缩短原函数的语句，减少了数据败坏的复杂度；
2.  尽量实现更高的异常安全保证等级。

### 条款 30：透彻了解 inlining 的里里外外

1.  inlining 函数可以免除函数调用成本的开销，且编译器可对其执行语境相关的最优化，但需考虑 object code 的大小；
2.  inline 的隐喻方式：将函数定义于 class 定义式内；明确声明方式：在其定义式前加上关键字 inline；
3.  inline 函数通常一定被置于头文件内，因为 inlining 大部分情况下都是编译期行为；template 通常也被置于头文件内，因为大部分建置环境都是在编译期完成具现化动作；
4.  如果一个 template 具现出来的函数都应该 inlined，则将此 template 声明为 inline，否则应避免此声明；
5.  inline **只是对编译器**的一个**申请**，不是强制命令。大多数编译器如无法将要求的函数 inline 化，会给出一个警告信息；
6.  慎重决定 inlining 施行范围：将大多数 inlining 限制在小型、被频繁调用的函数身上，以便于日后的调试和二进制升级。

*   编译器通常不对 “通过函数指针而进行的调用” 实施 inlining，且需考虑后续代码维护用到函数指针的可能；
*   构造函数和析构函数并不适合用于 inlining，往往会引起代码的膨胀（所不要随便地将构造函数和析构函数的定义体放在类声明中）；
*   inline 函数代码如发生改变，所有用到该 inline 函数的程序都必须重新编译；
*   大部分调试器都不能对 inline 函数进行调试；

总结：

1.  如果 inline 函数不能增强性能，就避免使用它；
2.  inline 修饰符用于解决一些频繁调用的小函数大量消耗栈空间（栈内存）的问题；
3.  inline 函数本身不能是直接递归函数；
4.  将成员函数的定义体放在类声明之中（隐喻方式）虽然能带来书写上的方便，但不是一种良好的编程风格；
5.  关键字 inline 必须与函数定义体放在一起才能使函数成为内联，仅将 inline 放在函数声明前面不起任何作用，即 inline 是一种 “用于实现的关键字”；声明前可以加 inline 关键字，但不符合高质量 C++/C 程序设计风格的一个基本原则：声明与定义不可混为一谈，用户没有必要、也不应该知道函数是否需要内联。

### 条款 31：将文件间的编译依存关系降至最低

[一个简洁明了的解答： https://www.zhihu.com/question/52832178/answer/192499529](https://www.zhihu.com/question/52832178/answer/192499529)  
[一个 pimpl 手法的例子 ：http://t.csdn.cn/qzlzt](https://link.zhihu.com/?target=http%3A//t.csdn.cn/qzlzt)  
**采用 pimpl 手法，可以使得 class 的数据成员的实现与定义分离**，这是真正的接口与实现分离。其关键在于以声明的依存性替换了定义的依存性。这揭示了一个本质：**尽可能让头文件自我满足**，万一做不到，则让它与其他文件内的声明式相依。其他每一件事都源自这个简单的策略：

**student.h**

```
#pragma once
#include <memory>

class StudentImpl;

class Student {
public:
    Student(int Id);
    
    /** 因为此处智能指针使用的是 unique_ptr ，它为了保证高效性，
     * 其删除器是自身的一部分，它必须保证 raw pointer 为 complete 对象。
     * 由于编译器默认生成的析构函数是 inline ，
     * 此时 Impl 所指之物仅仅是前置声明，是一个 non-complete 对象，所以会报错。
     * 因此如果使用 unique_ptr 而不是 shared_ptr 实现 Impl时，
     * 不要使用默认的析构行为，请自行额外实现。
     * 因为shared_ptr不使用自身的 deleter，无需这种保证。
     */
    ~Student();
    
    Student(const Student&) = delete;
    Student& operator=(const Student&) = delete;
    Student(Student&&) = delete;
    Student& operator=(Student&&) = delete;
    void SetId(int Id) const;
    int GetId() const;
private:
    std::unique_ptr<StudentImpl> Impl;
};
```

**student.cpp**

```
#include "Student.h"
class StudentImpl {
public:
    StudentImpl(int mId) {
        Id = mId;
    }

    ~StudentImpl() = default;
    StudentImpl(const StudentImpl&) = default;
    StudentImpl& operator=(const StudentImpl&) = default;
    StudentImpl(StudentImpl&&) = default;
    StudentImpl& operator=(StudentImpl&&) = default;

    void SetId(int mId) {
        Id = mId;
    }

    int GetId() const {
        return Id;
    }

private:
    int Id;
};

Student::Student(int Id): Impl(std::make_unique<StudentImpl>(Id)) {}
Student::~Student() = default;

void Student::SetId(int Id) const {
    Impl->SetId(Id);
}

int Student::GetId() const {
    return Impl->GetId();
}
```

**main.cpp**

```
#include <iostream> 
#include "student.h"

int main() {
    const Student Moota(233);
    std::cout << Moota.GetId() << "\n";

    Moota.SetId(666);
    std::cout << Moota.GetId() << "\n";

    return 0;

}
```

总结：

*   支持编译依存性最小化的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是 Handle classes 和 Interface classes。
*   程序库头文件应该以完全且仅声明式（full and declaration-only forms）的形式存在。这种做法不论是否涉及 templates 都适用。

# **六、继承与面向对象设计**

### 条款 32：确定你的 public 继承保证了 is-a 关系

*   public 继承的意思是：**子类是一种特殊的父类**，这就是所谓的 “is-a” 关系。
*   在使用 public 继承时，**子类必须涵盖父类的所有特点，必须无条件继承父类的所有特性和接口**。

如果你令 class D 以 public 的方式继承 class B，你便是告诉 C++ 编译器和你的客户：**每一个类型为 D 的对象，同时也是一个类型为 B 的对象**，反之则不成立。类型 B 比 类型 D 表现出更一般的概念，而类型 D 比类型 B 表现出更特殊化的概念。B 对象可使用到的地方，D 对象都可以派上用场。反之，如果你需要对象 D ，B 对象并不能效劳，因为 **B 对象并不具备 D 对象所含有的特殊化信息。**

*   虽然 public 继承意味着 is-a 的关系看似简单，但有时候如果单纯偏信生活经验，会犯错误。 例如：

案例 1：鸵鸟是不是鸟  
如果我们考虑飞行这一特性（或接口），那么鸵鸟类在继承中就绝对不能用 public 继承鸟类，因为鸵鸟不会飞，我们要在**编译阶段消除调用飞行接口的可能性**；但如果我们关心的接口是下蛋的话，按照我们的法则，鸵鸟类就可以 public 继承鸟类。

```
class Bird {  //没有声明 fly 函数
public:
    virtual ~Bird();
};
	
class Penguin : public Bird { 	   //没有声明 fly 函数
public:
    //...
};

inline void Try(){
	Penguin P;
	P.fly();  //编译阶段就会报错！
}
```

案例 2：矩形和正方形  
生活经验告诉我们正方形是特殊的矩形，但这并不意味着在代码中二者可以存在 public 的继承关系，矩形具有长和宽两个变量，但正方形无法拥有这两个变量——没有语法层面可以保证二者永远相等，那就不要用 public 继承。  

总结：

*   **public 继承意味 is-a。**适用于 base classes 身上的每一件事情一定也适用于 derived classes 身上，因为每一个 derived class 对象也都是一个 base class 对象。
*   在确定是否需要 public 继承的时候，我们首先要搞清楚**子类是否必须拥有父类每一个特性**，如果不是，则无论生活经验是什么，都不能视作”is-a” 的关系。**public 继承关系不会使父类的特性或接口在子类中退化，只会使其扩充。**

### 条款 33：避免遮掩继承而来的名称

何谓遮掩名称？

当编译器在 func 的作用域并使用 x 时，它会先在 local 作用域查找是否存在该变量，如果找不到在扩大作用域。显然，编译器会在 local 找到 double x，然后停止查找，这意味着在 local 中使用 x，将总是找到 double x，而非 global 作用域的 int x。这便是我们所说的：遮掩名称。

```
int x=1;   //全局变量
void func() {
   double x=1.1;  //局部变量
   std::cout<<x<<"\n";
}
```

该条款研究的是继承中多次重载的虚函数的**名称遮盖问题**，如果在你设计的类中没有涉及到对同名虚函数做多次重载，请忽略本条款。

在父类中，虚函数`foo()`被重载了两次，可能是由于参数类型重载（`foo(int)`），也可能是由于`const`属性重载 (`foo() const`)。如果子类仅对父类中的`foo()`进行了覆写，那么在子类中父类的另外两个实现 (`foo(int)` ,`foo() const`) 也无法被调用，这就是名称遮盖问题——名称在**作用域级别的遮盖是和参数类型以及是否虚函数无关的**，即使子类重载了父类的一个同名，父类的所有同名函数在子类中都被遮盖

**继承中的遮掩**

```
class Base {
public:
   virtual void mf1()=0;
   virtual void mf1(int);
   virtual void mf2();
   void mf3();
   void mf3(double);
private:
};

class Derived : public Base {
public:
   virtual void mf1();
   void mf3();
};

inline void Try() {
   Derived d;
   int x;
   d.mf1();   //没问题，调用Derived::mf1
   d.mf1(x);  //报错，因为 derived::mf1 遮掩了 base:: mf1 (int) 
   d.mf2();	  //没问题，调用Base::mf2
   d.mf3();	  //没问题，调用Derived::mf3
   d.mf3(4);  //报错，因为 derived::mf3 遮掩了 base::mf3 （double）
}
```

**避免遮掩名称的方式：**

**1. 使用 using 声明式**  
让 base class 内的某些事物可以在 derived class 作用域中可见。  
注意使用 using 声明式的权限符为 public，注意不要违反继承时的继承权限。对于 public 继承，并不是所有的函数都被继承，因而不是所有的函数都可以进行声明访问。尝试声明无法访问的函数，编译器会自动报错

```
class Derived : public Base {
public:
   using Base::mf1;
   using Base::mf3;
   virtual void mf1();
   void mf3();
};
```

**2. 使用转交函数（forwarding function）**  
例如假设 Derived 以 private 形式继承 Base，而 Derived 唯一想继承的 mf1 是那个无参数版本。using 声明式在这里派不上用场，因为 using 声明式会令继承而来的某给定名称之所有同名函数在 derived class 中都可见。不，我们需要不同的技术，即一个简单的转交函数 (forwarding function)

```
class Base {
public:
   virtual void mf1()=0;
   virtual void mf1(int);
   virtual void mf2();
   void mf3();
   void mf3(double);
private:
   int x;
};

class Derived : private Base {
public:
   virtual void mf1() { //转交函数（forwarding function）
      Base::mf1();      //暗自成为inline
   }
   ...
};

inline void Try() {
   Derived d;
   int x;
   d.mf1();   //没问题，调用Derived::mf1
   d.mf1(x);  //报错，  Base::mf1()被遮掩了 
}
```

总结：

1.  derived classes 内的名称会遮掩 base classes 内的名称。在 public 继承下从来没有人希望如此。
2.  为了让被遮掩的名称再见天日，可使用 using 声明式或转交函数（forwarding functions）。

### 条款 34：区分接口继承和实现继承

*   public 继承其实可以分成：`函数接口（function interfaces）`继承和`函数实现（function implemention）`继承。这意味着 derived class 不仅可以有 base class 函数的声明，还可以有 base class 函数的实现。
*   成员函数的`接口总是会被继承`。因为条款 32 曾说：public 继承是 is-a 的关系，任何可以作用于 base class 的函数也一定可以作用于 derived class。
*   不同类型的函数代表了**父类对子类实现过程中不同的期望**。
*   在父类中声明纯虚函数，是为了**强制子类拥有一个接口**，并**强制子类提供一份实现**。
*   在父类中声明虚函数，是为了**强制子类拥有一个接口**，并**为其提供一份缺省实现**。
*   在父类中声明非虚函数，是为了**强制子类拥有一个接口以及规定好的实现**，并不允许子类对其做任何更改（条款 36 要求我们不得覆写父类的非虚函数）。

在这其中，有可能出现问题的是普通虚函数，这是因为父类的缺省实现并不能保证对所有子类都适用 (具体请看 Cherno 的 CppSeriesP28、29)，因而当子类忘记实现某个本应有定制版本的虚函数时，父类应**从代码层面提醒子类的设计者做相应的检查**，很可惜，普通虚函数无法实现这个功能。一种解决方案是，在父类中**为纯虚函数提供一份实现**，作为需要主动获取的缺省实现，当子类在实现纯虚函数时，检查后明确缺省实现可以复用，则只需调用该缺省实现即可，这个主动调用过程就是在代码层面提醒子类设计者去检查缺省实现的适用性。

```
class Airplane {
public:
    virtual ~Airplane() = default;

    //pure-virtual  子类继承函数接口，强制覆写
    virtual void ToString() const =0;
    //impure-virtual/virtual  子类继承函数接口和默认函数实现，可覆写
    virtual void Fly();
    //non-virtual   子类继承函数接口和默认函数实现，不可覆写
    void DefaultFly();
};

class ModelA : public Airplane { //public 子类继承函数接口
public:
    void ToString() const override;
    void Fly() override;
};
```

*   将纯虚函数、虚函数区分开的并不是在父类有没有实现——纯虚函数也可以有实现，其二者本质区别在于父类对子类的要求不同，前者在于**从编译层面提醒子类主动实现接口**，后者则侧重于**给予子类自由度对接口做个性化适配**。非虚函数则没有给予子类任何自由度，而是要求子类坚定的遵循父类的意志，**保证所有继承体系内能有其一份实现**。  
    
*   两个常见错误：

**1. 将所有函数都声明为 non-virtual。**  
这会使得 derived class 没有余裕空间进行特化工作。non-virtual 函数还会带来析构问题，见条款 7。实际上任何 class 如果打算使用多态性质，都会有若干 virtual 函数。如果你关心 virtual 函数的成本，请参考 80-20 法则：一个典型的程序有 80% 的执行时间花费在 20% 的代码身上。这个法则十分重要，这意味着平均而言你的函数调用中可以有 80% 是 virtual，而不冲击程序的大体效率。所以当你担心是否有能力负担 virtual 函数的运行成本时，先关注那举足轻重的 20% 代码身上。  
**2. 将所有函数都声明为 virtual。**  
有时候是正确的，比如 interfaces class。然而这也可能是 class 设计者缺乏坚定立场的表现，某些函数就是不该在 derived class 中被重新实现，你就应该把它声明为 non-virtual。

总结：

1.  接口继承和实现继承不同。在 public 继承之下，derived class 总是继承 base class 的接口。
2.  pure virtual 函数只具体指定接口继承。
3.  简朴的（非纯）impure virtual 函数具体指定接口继承及缺省实现继承。
4.  non-virtual 函数具体指定接口继承以及强制性实现继承。

### 条款 35：考虑 virtual 函数以外的其他选择

C++ 的 virtual 函数让我们能方便地实现接口继承与实现继承，但同时也会让我们忽略可能的其他方案。本条款针对于 virtual 函数的功能设计了具有不同优缺点的替代方案。

假如你在设计一款游戏，涉及到各式角色的健康情况，但不同角色的健康度是不同的，这时候将计算健康度的函数**声明为 virtual** 似乎是再明白不过的做法：

```
class Character {
public:
    virtual ~Character()=default;
    virtual int CalculateHealthValue() const;
};

class NpcEvil:public Character {
public:
    virtual int CalculateHealthValue() const override;
};
```

这的确是再**明白不过的设计**，但是从某个角度说却**反而成了它的弱点**。由千这个设计如此明显，你可能因此没有认真考虑其他替代方案。为了帮助你跳脱面向对象设计路上的常轨，让我们考虑其他一些解法。

*   **藉由 `Non-Virtual Interface` 手法 （NVI） 实现 Template Method 模式**

有一个有趣的思想流派**主张 virtual 函数应该几乎总是 private**。这个流派建议：**保留 CalculateHealthValue 为 public 成员函数，但让它成为 non-virtual，并调用一个 private virtual 函数（例如 OnCalculateHealthValue）进行实际工作**

```
class Character {
public:
    virtual ~Character()=default;
    int CalculateHealthValue() const {
        const int Result= OnCalculateHealthValue();
        //...
        return Result;
    }
private:
    virtual int OnCalculateHealthValue()const=0;
};

class NpcEvil:public Character {
private:
    int OnCalculateHealthValue() const override {
        return 100;
    }
};
//这种设计方式：令客户通过 public non-virtual 成员函数间接调用 private virtual 函数，称为 non-virtual interface （NVI）手法。
```

值得注意的一点，**C++ 允许 derived class 覆写 base class 的 private virtual 方法**。看起来诡异，但这是真的。

优点

NVI 手法的一个优点是可以在调用 private virtual 函数前后做一些额外的事情，其实这也是封装带来的好处。调用之前可以做的工作：锁定互斥器，制造运转日志记录项，验证 class 约束条件，验证函数先决条件等等。调用之后可以做的工作：互斥器解除锁定，验证函数的事后条件，再次验证 class 约束条件等等。但假如没有这一层封装，直接调用 virtual 函数，就没有任何好办法可以做这些事。

缺点

在某些 class 继承体系中，virtual 函数必须调用其 base class 的版本，这就导致 virtual 函数必须是 protected 而不能是 private，有些时候 virtual 函数甚至一定得是 public。在这种情况下，non-virtual 成员函数和 virtual 成员函数都是 public 的，NVI 的 wrapper 手法显然就不成立了

*   **藉由 `Function Pointers` 实现 Strategy 模式**

NVI 手法对于 public virtual 函数而言是一个有趣的设计，但从某种设计角度来看，这仅仅多了一层装饰而已，毕竟我们还是使用 virtual 函数计算每一个角色的健康指数。另一种更具戏剧性的做法主张：人物健康指数的计算与人物类型无关，这样的计算完全不需要人物这个成分。  
例如我们可能要求每个人物的构造函数接受一个指针，指向一个健康计算函数，我们可以调用该函数进行实际计算：

```
class Character {
public:
    virtual ~Character() = default;
	// 函数指针类型定义
    typedef int (*FCalculateHealthValueFunc)(const Character&);

    explicit Character(FCalculateHealthValueFunc InCalculateHealthValueFunc) : CalculateHealthValueFunc(
        InCalculateHealthValueFunc) { }

    int CalculateHealthValue() const {
        const int Result = CalculateHealthValueFunc(*this);
        //...
        return Result;
    }

    void SetCalculateHealthValueFunc(FCalculateHealthValueFunc InCalculateHealthValueFunc) {
        CalculateHealthValueFunc = InCalculateHealthValueFunc;
    }

private:
    FCalculateHealthValueFunc CalculateHealthValueFunc;
};

class NpcEvil : public Character {
public:
    explicit NpcEvil(FCalculateHealthValueFunc InCalculateHealthValueFunc)
        : Character(InCalculateHealthValueFunc) {}
};

int main()
{
    //NpcEvil Moota([](const Character&) {
    //    return 100;
    //    });
    auto lam = [](const Character&) {
        return 1000;
    };
    NpcEvil Moota(lam);
    NpcEvil Vicky([](const Character&) {
        return 10000;
        });
    std::cout << Moota.CalculateHealthValue() << "\n";
    std::cout << Vicky.CalculateHealthValue() << "\n";

    //小宇宙爆发
    Moota.SetCalculateHealthValueFunc([](const Character&) {
        return 99999;
        });
    std::cout << Moota.CalculateHealthValue() << "\n";
}
```

优点：

同一人物类型的不同实例`可以有不同的健康计算函数`。只需提供不同的函数指针进行初始化。  
某已知人物的健康计算函数`可在运行期变更`。只需提供类似 Setter 函数即可替换用于计算健康度的函数。

缺点：

这种指针函数只能访问 class 的 public 成分  
如果人物的健康可纯粹根据人物 public 接口得来的，这种设计没有问题，但如果需要 non-public 信息进行精确计算，就有问题了。 实际上任何时候当你将 class 内的某个机能由 non-member 函数实现，都会存在这样的额外难题。  
唯一能解决该问题的办法就是弱化 class 的封装，比如声明外部函数为 friend，或者为函数的实现提供一些 public 访问函数。所以到底是要良好的封装性，还是要上面的两个优点，则需要根据具体情况仔细分析一下。  
另一种思路：  
把外部函数指针改成类成员函数指针可以同时享有这两个优点，并且不用弱化 class 的封装性。但这会导致 class 内部的函数冗杂，并且该健康计算函数不能再不同 class 中复用

*   **藉由 `tr1::function` 完成 Strategy 模式**

一旦习惯了 templates 以及它们对隐式接口的使用，见条款 41 的使用，基于函数指针的做法看起来就苛刻死板了。为什么要求计算健康指数必须是个函数，而不是**某些看起来像函数的东西**，例如函数对象？如果是函数，为什么不能是个成员函数，为什么一定返回 int 而不是任何可被转换为 int 的类型呢？  
实际上如果我们不再使用函数指针，而是改用一个类型为 **tr1::function** 的对象，这些约束就全不见了。就像条款 54 所说，这样的对象可持有任何可调用之物，只要其签名式满足于需求：

```
#include <functional>
class Character {
public:
    virtual ~Character() = default;
    //接受一个reference 指向const Character, 并返回int
    typedef std::function<int(const Character&)> FCalculateHealthValueFunc; //注意 const & 不能隐式转化

    explicit Character(FCalculateHealthValueFunc InCalculateHealthValueFunc) : CalculateHealthValueFunc(
        InCalculateHealthValueFunc) { }

    int CalculateHealthValue() const {
        const int Result = CalculateHealthValueFunc(*this);
        //...
        return Result;
    }

    void SetCalculateHealthValueFunc(FCalculateHealthValueFunc InCalculateHealthValueFunc) {
        CalculateHealthValueFunc = InCalculateHealthValueFunc;
    }

private:
    FCalculateHealthValueFunc CalculateHealthValueFunc;
};

class NpcEvil : public Character {
public:
    explicit NpcEvil(FCalculateHealthValueFunc InCalculateHealthValueFunc)
        : Character(InCalculateHealthValueFunc) {}
};

class FCalculator {
public:
    static double CalculateHealthGM(const Character& InNpcEvil) {
        return 2147483647;
    }

    int CalculateHealthNormal(const Character& InNpcEvil) {
        return 5;
    }

};

int main() {
    //函数指针仍然适用
    NpcEvil Moota([](const Character&) {
        return 100;
        });
    std::cout << Moota.CalculateHealthValue() << "\n";

    //使用返回值为 double 的 static member函数
    Moota.SetCalculateHealthValueFunc(&FCalculator::CalculateHealthGM);
    std::cout << Moota.CalculateHealthValue() << "\n";

    FCalculator Calculator;

    //使用某对象的 member函数
    Moota.SetCalculateHealthValueFunc(std::bind(&FCalculator::CalculateHealthNormal, Calculator,
        std::placeholders::_1));
    std::cout << Moota.CalculateHealthValue() << "\n";
}
```

总结：

*   virtual 函数的替换方案包括 NVI 手法以及 Strategy 设计模式的多种形式。NVI 手法自身是一个特殊形式的 Template Method 设计模式。
*   将机能从成员函数移到 class 外部函数，带来的一个缺点是，非成员函数无法访问 class 的 non-public 成员。
*   tr1::function 对象的行为就像一般函数指针。这样的对象可接纳与给定之目标签名式（target signature）兼容的所有可调用物（callable entites）。

### 条款 36：绝不重新定义继承而来的 non-virtual 函数

*   如果你的函数**有多态调用的需求**，一定记得把它**设为虚函数**，否则在动态调用（基类指针指向子类对象）的时候是不会调用到子类重载过的函数的，很可能会出错。
*   反之同理，如果一个函数父类没有设置为虚函数，一定不要在子类重载它。
*   原因：多态的动态调用中，只有虚函数是**动态绑定**，非虚函数是**静态绑定**的——指针（或引用）的静态类型是什么，就调用那个类型的函数，和动态类型无关。

条款 7 解释为什么**多态性质的 base classes 应该声明 virtual 析构函数**。如果你在多态性质下的 base class 声明了 non-virtual 函数，那么 derived class 便绝不应该重新定义一个继承而来的 non-virtual 析构函数。但即使你没有定义，条款 5 曾说，**编译器会默认为你生成它**，所以多态性质的 base classes 都需要 virtual 析构函数。因此就本质而言，条款 7 只不过是本条款的一个特殊案例，尽管它足够重要到单独成为一个条款。

### 条款 37：绝不重新定义继承而来的缺省参数值

静态绑定和动态绑定的差异

对象的所谓静态类型（static type），就是它在程序中被声明时采用的类型。对象的所谓动态类型（dynamic type），就是指目前所指对象的类型，可以决定一个对象将会有什么样的动态行为。virtual 函数是动态绑定的，所以调用一个 virtual 函数时，究竟调用那一份函数实现代码，取决于该对象的动态类型。

在继承中：

*   不要更改父类非虚函数的缺省参数值，其实**不要重载父类非虚函数的任何东西**，不要做任何改变！(见条款 36)
*   虚函数不要写缺省参数值，子类自然也不要改，**虚函数要从始至终保持没有缺省参数值**。

缺省参数值是属于**静态绑定**的，而**虚函数属于动态绑定**。虚函数在大多数情况是供动态调用，而在动态调用中，子类做出的缺省参数改变其实并没有生效，反而会引起误会，让调用者误以为生效了。

```
class B {
public:
    virtual std::string ToString(std::string Text="BBB") const{
        return Text;
    }
};

class D : public B {
public:
    virtual std::string ToString(std::string Text="DDD") const{
        return Text;
    }
};

inline void BTry() {
    D Derived;
    B* BasePointer = &Derived;
    std::cout << BasePointer->ToString() << "\n"; //BBB
}
//BasePointer 静态类型是 B，动态类型是 D。意味着 ToString 的定义取决于 D，而 ToString 的缺省参数值却取决于 B。
```

缺省参数值属于**静态绑定**的原因是为了提高**运行时效率**。

假设缺省参数值为动态绑定，编译器就必须要支持某种方式在运行期为 virtual 函数选择适当的缺省参数值，这意味着更慢更复杂。

假如你需要重新定义缺省参数值的需求

1. 替换 virtual 函数。条款 35 列出了不少 virtual 函数的替换设计。  
2. 如果你真的想让某一个虚函数在这个类中拥有缺省参数，那么就把这个虚函数设置成 private，在 public 接口中重制非虚函数，让非虚函数这个 “外壳” 拥有缺省参数值，当然，这个外壳也是一次性的——在被继承后不要被重载。

总结：

*   绝对不要重新定义一个继承而来的缺省参数值，因为缺省参数值都是静态绑定，而 virtual 函数——你唯一应该覆写的东西——却是动态绑定。

### 条款 38：通过复合塑膜出 has-a 关系，或 “根据某物实现出”

1.  两个类的关系除了继承之外，还有 “一个类的对象可以作为另一个类的成员”，我们称这种关系为 “类的复合”
2.  public 继承是一种 is-a 的意义，复合也有它们的意义。复合意味着 has-a（有一个）或 is-implemented-in-terms-of （根据某物实现出）。
3.  `is-a`（是一种 和 `is-implemented-in-terms-of` （根据某物实现出） 的区分

这两种关系其实是在不同领域的表现，如果对象只是你所塑造的世界中的某个物品，某些人物等，那这样的对象就属于应用域部分，如果对象需要负责你所塑造世界的细节部分，是规则的制定者和执行者，那这样的对象就属于实现域部分。当对象处于应用域，它就是 has-a 的关系，当对象处于实现域，它就是 is-implemented-in-terms-of 的关系。  
请牢记 “is-a” 关系的唯一判断法则，一个类的全部属性和接口是否必须**全部**继承到另一个类当中？另一方面，“用一个工具类去实现另一个类” 这种情况，是需要对工具类进行**隐藏**的，比如人们并不关心你使用 stack 实现的 queue，所以就藏好所有 stack 的接口，只把 queue 的接口提供给人们用就好了，而红芯浏览器的开发者自然也不希望人们发现 Google Chrome 的内核作为底层实现工具，也需要 “藏起来” 的行为。

4. 什么情况下我们应该用类的复合？  
当某一个类 “拥有” 另一个类对象作为一个属性（has-a），比如学生拥有铅笔、市民拥有身份证，一个人可以有名字，有地址，有手机号码等。

```
class Address;
class PhoneNumber;
class Person {
public:
    //...
private:
    std::string name;  //复合对象
    Address& Address;
    PhoneNumber& VoiceNumber;
    PhoneNumber& FaxNumber;
};
```

" 一个类根据另一个类实现”(is-implemented-in-terms-of)。比如 “用 stack 实现一个 queue”，更复杂一点的情况可能是 “用一个老版本的 Google Chrome 内核去实现一个红芯浏览器”。 再比如用 list 对象实现一个 sets

这里以 list 对象实现一个 sets 为例， **set 成员函数可大量倚赖 list 及标准程序库其他部分提供的机能来完成：**

```
template <typename T>
class Set {
public:
    bool Contains(const T& Item) const;
    void Insert(const T& Item);
    void Remove(const T& Item);
    size_t Size() const;
private:
    std::list<T> Rep;
};

template <typename T>
bool Set<T>::Contains(const T& Item) const {
    bool Result=std::find(Rep.begin(),Rep.end(),Item)!=Rep.end();
    return Result;
}

template <typename T>
void Set<T>::Insert(const T& Item) {
    if(!Contains(Item)) {
        Rep.push_back(Item);
    }
}

template <typename T>
void Set<T>::Remove(const T& Item) {
    typename std::list<T>::iterator It=std::find(Rep.begin(),Rep.end(),Item);
    if (It!=Rep.end()) {
        Rep.erase(Item);
    }
}

template <typename T>
size_t Set<T>::Size() const {
    return Rep.size();
}
//显然，set 和 list 的关系是 is-implemented-in-terms-of，而不单单是 has-a 的关系。
```

总结

*   复合（composition）的意义和 public 继承完全不同。
*   在应用域（application domain），复合意味 has-a （有一个）。在实现域（implementation domain），复合意味 is-implemented-in-terms-of（根据某物实现出）。

### 条款 39：明智而审慎地使用 private 继承

条款 32 中说到 **public 继承是一种 is-a** 关系。在这种继承体系下，编译器在必要时刻（为了让函数调用成功）会将 derived class 转换为 base class 。

```
class Person {...};
class Student : public Person {...};
void eat(const Person& p);
​
Person p;
Student s;
​
eat(p); // 没问题
eat(s); // 没问题，这里编译器将Student暗自转换为Person类型。
```

**private 继承的两个行为**

*   如果 derived class 和 base class 是 **private 继承**，那么从 derived class 到 base class 的**转换将失败**
*   在 **private 继承**下，base class 的成员无论是 private、protected 还是 public，继承后**都会变为 private**。

```
class Person {...};
class Student : private Person {...};
void eat(const Person& p);
​
Person p;
Student s;
​
eat(p); // 没问题
eat(s); // 错误！！！s是private继承，编译器无法转换。
```

**Private 继承的意义**

*   private 继承意味着：is-implemented-in-terms-of （根据某物实现出）。private 继承可以看作纯粹是`为了实现细节`，它需要的不是类似 public 继承可以向外提供接口，仅仅是为了让 derived class 采用 base class 中已经具备的某种特性。derived 和 base 之间并没有什么直接意义上的联系。

**那么当我们拥有 “用一个类去实现另一个类” 的需求的时候，如何在类的复合与 private 继承中做选择呢？**

*   尽可能用复合，**除非必要，不要采用 private 继承**。
*   当我们需要对工具类的某些方法（虚函数）做重载时，我们应选择 private 继承，这些方法一般都是工具类内专门为继承而设计的调用或回调接口，需要用户自行定制实现。

**案例一：能用复合，就不要用 private**

假设我们需要写一个 Widget（控件）。这个控件需要按某一频率定时检查 Widget 的某些信息，换句话说需要定时地调用某个函数。为了少写新的代码，我们在其他程序中翻到了一个 Timer class。

```
class Timer 
{
public:
    explicit Timer(int tickFrequency);
    virtual void onTick() const;
};
```

这个定时器的功能是每隔一段时间就调用一次 onTick 函数。  
为了让 Widget 重新定义 virtual 内的 virtual 函数，Widget 必须继承自 Timer。但此时不能使用 public 继承，**因为 Widget 并不是个 Timer**，所以我们必须以 private 形式继承 Timer。

```
class Widget : private Timer 
{
private:
    virtual void onTick() const override;
}
```

但 **private 继承并不是唯一的选择方案**，我们可以**使用复合**来替代这个方案。  
只要在 Widget 内声明一个**嵌套式 private class**, 后者以 public 形式继承 Timer 并重新定义 onTick, 然后放一个这种类型的对象千 Widget 内

```
class Widget 
{
private:  //private的
    class WidgetTimer : public Timer 
    {
    public:
        virtual void onTick() const override;  //本应被目标类覆写的方法在嵌套类中实现，这样TargetClass的子类就无法覆写该方法。
    };
    WidgetTimer timer;
};
```

**该复合设计相比于 private 继承有两个优点：**

**当 Widget 拥有 derived class 时，你可能同时想阻止 derived class 重新定义 onTick。**如果是 private 继承（Widget 继承了 Timer），那这个想法就不可能实现。（条款 35 曾说过：**derived class 可以重新定义 private virtual 函数**，即使它们不能调用它）。但如果 WidgetTimer 是 widget 内部的一个 private 成员并继承 Timer，Widget 的 derived classes 将无法取用 WidgetTimer，因此无法继承它或重新定义它的 virtual 函数。  
**降低 widget 的编译依存性。**如果继承 Timer，当 Widget 被编译时 Timer 的定义必须可见，所以定义 widget 的文件必须 #include Timer.h。如果 WidgetTimer 移出 Widget 之外，而 widget 内含指针指向一个 widgetTimer，widget 可以只带一个简单的 WidgetTimer 前置声明。（对大型系统而言非常重要）关于编译依存性的最小化，详见条款 31 。

**案例二：一个使用 private 的极端案例**

这种情况真是够激进的，只适用于你所处理的 class 不带任何数据时。这样的 `class 不存在任何成员函数或变量`。示例：

```
class Empty {};
​
class DemoWithEmpty {
private:
    int x;
    FEmpty Empty;
};
​
inline void Try(){
    DemoWithEmpty DemoWithEmpty;
    Empty Empty;
    std::cout<<sizeof(Empty)<<"\n";  //1
    std::cout<<sizeof(DemoWithEmpty)<<"\n"; //8
}
```

可以看到，一个不含任何成员的 class 的大小居然为 1。因为 C++ 规定凡是独立对象都必须有非零大小。所以你可以发现 sizeof（Empty）的大小为 1，而且几乎所有的编译器都这样做。至于为什么含一个 int 大小的 class 是 8，这涉及到内存对齐的问题，不必详细讨论。  
或许你注意到了，独立对象才需要有非零大小，这意味着继承而来的 Empty class 大小可以不受约束:

```
class FEmpty {};
​
class DemoWithEmpty :private FEmpty{
private:
    int x;
};
    
inline void Try(){
    DemoWithEmpty DemoWithEmpty;
    Empty Empty;
    std::cout<<sizeof(Empty)<<"\n";  //1
    std::cout<<sizeof(DemoWithEmpty)<<"\n"; //4
}
```

DemoWithEmpty 所用大小正好等于一个 int 的大小，而这种表现就是所谓的 **`EBO（empty base optimization）空白基类最优化`。**值得注意的是，EBO 一般在单一继承下才可行。

尽管有这些例外情况，让我们回到根本。大部分 class 并非 empty，这很少成为你使用 private 继承的理由。只有当你面对需要访问 base class 的 protected 成员或者覆写 virtual 函数时，private 继承才被纳入考虑。当你审视完所有方案，仍然认为 private 继承是最佳方法，才使用它。

总结：

1.  Private 继承意味 is-implemented-in-terms-of（根据某物实现出）。它通常比复合（composition）的级别低。但是当 derived class 需要访问 protected base class 的成员，或需要重新定义继承而来的 virtual 函数时，这么设计是合理的。
2.  和复合（composition）不同，private 继承可以造成 empty base 最优化。这对致力于对象尺寸最小化的程序库开发者而言，可能很重要。

### 条款 40：明智而审慎地使用多继承

C++ 社群对多重继承（multiple inheritance MI）持有两类观点。

单一继承是好的，但多重继承不值得使用。  
单一继承（single inheritance SI）是好的，多重继承更好。

两种观点的比较与选择  
观点一：**多重继承不值得使用**

原因一：**多重继承可能会引发歧义（ambiguity）行为。** **解决办法：指明调用**

```
// 图书馆可借内容的基类。
class BorrowableItem
{
public:
    void checkOut(); // 检查函数
};

// 一个电子小工具类。
class ElectronicGadget
{
private:
    bool checkOut() const; // 检查函数
};

// Mp3播放器类。
class MP3Player :
public : BorrowableItem, 
public : ElectronicGadget 
{...};

MP3Player mp;
// 这里会引发歧义，mp对象到底调用的是哪个checkout函数？
mp.checkout();
```

疑问：B~class 的 checkOut 函数是 public 的，E~class 的 checkOut 函数是 private 的，理应只有 B~class 的函数是可以调用，那为什么会引发歧义行为？  
原因：这与 **C++ 的解析机制**有关（与解析（resolving）重载函数调用的规则相符）。在看到是否有个函数可取用之前，C++ 会首先确认这个函数是不是此调用的最佳匹配，**找出最佳匹配函数后才检验其可取用性**。在该例中，两个 checkOuts 有相同的匹配程度（因此才造成歧义），没有所谓最佳匹配。因此 **ElectronicGadget::checkOut 的可取用性也就从未被编译器审查** (是不是 public 对该问题也就没有影响, 还没到这一步就错了)。

**解决方法**：如下调用即可

```
// 指定调用BorrowableItem的checkOut函数。
mp.BorrowableItem::checkOut();
// 错误行为！！！通过最佳匹配检查后发现，这个函数是个private函数。
mp.ElectronicGadget::checkOut();
```

**原因二：要命的 “钻石型多重继承” 解决办法：virtual 继承**

```
class File {...};
class InputFile: public File {...};
class OutputFile: public File {...};
class IOFile: public InputFile, public OutputFile {...};
```

这种继承体系必须面对的一个问题是：**是不是打算让 File class 内的成员变量经过每一条路径被复制？**

总结：

1.  多重继承比单一继承复杂。它可能导致新的歧义性，以及对 virtual 继承的需要。
2.  virtual 继承会增加大小，速度，初始化（赋值）复杂度等等成本。如果 virtual base classes 不带任何数据，将是最具有实用价值的情况。
3.  多重继承的确有正当用途。其中一个情节涉及 **“public 继承某个 Interface class”** 和 **"private 继承某个协助实现的 class"** 的两相组合。

# **七、模板与泛型编程**

### 条款 41：了解隐式接口和编译器多态

**显式接口**和**运行期多态**

面向对象的世界总是以显式接口和运行期多态解决问题。

显式接口的构成**：** 函数名称，参数类型，返回类型，常量性也包括编译器产生的 copy 构造函数和 copy assignment 操作符。(函数的签名式)

```
class Widget {
public:
    virtual ~Widget()=default;
    virtual void Normalize()=0;
};
class SpecialWidget:public Widget {
public:
    virtual void Normalize() override {
        std::cout<<"Special"<<"\n";
    }
};
class NormalWidget:public Widget {
public:
    virtual void Normalize() override {
        std::cout<<"Normal"<<"\n";
    }
};
inline void BeginPlayWithWidget(Widget& InWidget) {
    InWidget.Normalize();
}
inline void Try() {
    SpecialWidget SpecialWidget;
    NormalWidget NormalWidget;
    BeginPlayWithWidget(SpecialWidget); //Special
    BeginPlayWithWidget(NormalWidget);	//Normal
}
```

可以这样说 BeingPlayWithWidget 函数中的 InWidget

*   由于 InWidget 类型被声明为 Widget，所以 InWidget 必须支持 Widget 接口。我们可以在源码中找到这个接口，看看它们是什么样子，所以我们称之为一个**显式接口**（explicit interface），也就是它在源码中的确可见。
*   由于 Widget 的 BeingPlayWithWidget(或者说某些成员函数) 函数是 virtual ，InWidget 将对此函数的调用表现出**运行期多态**（runtime polymorphism），也就是说将**在运行期根据 InWidget 的动态类型决定究竟调用哪个函数。**

**隐式接口**和**编译期多态**

templates 及泛型编程的世界，与面向对象的世界有根本的不同。在此世界显式接口和运行期多态仍然存在，但重要性降低。

隐式接口的构成： 有效表达式（valid expression）

```
template <typename T>
inline void BeginPlayWithWidgetTemplate(T& InT) {
    if (InT.Size() > 0 && InT != EClassType::None) {
        InT.Normalize();
        std::cout << InT.Size() << "\n";
        //...
    }
}
```

对 InT 来说

*   InT 必须支持哪一种接口，是由函数体中对 InT 的操作决定的。从本例来看，InT 的类型 T 必须要支持 Normalize 、Size、不等比较等操作。看表面可能并非完全正确，但这组操作对于 T 类型的参数来说，是一定要支持的**隐式接口**（implicit interface）。
*   凡是涉及 InT 的任何函数调用，例如 operator> 和 operator != 有可能造成 template 的具现化，使得这些调用得以成功，这样的行为发生在编译器。以**不同的 template 参数具现化 function templates** 会导致调用不同的函数，这便是所谓的**编译期多态**（compile-time polymorphism）。

纵使你从未用过 templates，应该不陌生 “运行期多态” 和“编译期多态”之间的差异，因为它类似于“哪一个重载函数该被调用”（发生在编译期）和“哪一个 virtual 函数该被绑定”（发生在运行期）之间的差异。

隐式接口与显示接口不同，它不基于函数签名式，而是由**有效表达式（valid expression）**构成

```
template <typename T>
inline void BeginPlayWithWidgetTemplate(T& InT) {
    if (InT.Size() > 0 && InT != EClassType::None) {
        //...
    }
}
```

在该例子中， T 类型的隐式接口有这些约束：

1. T 必须提供名叫 Size 的成员函数，该函数返回一个整数值；  
2. T 必须支持 operator!= 函数，用来与两个 T 对象，这里假设 EClassType 为 T 类型；  
得益于操作符重载（operator overloading）带来的可能性，这两个约束都不需要满足。是的，T 必须支持 size 成员函数，然而这个函数也可能从 base class 继承而得。这个成员函数不需返回一个整数值，甚至不需返回一个数值类型。就此而言，它甚至不需要返回一个定义有 operator > 的类型！它唯一需要做的是返回一个类型为 x 的对象，而 x 对象加上一个 int（10 的类型）必须能够调用一个 operator>。这个 operator > 不需要非得取得一个类型为 x 的参数不可，因为它也可以取得类型 Y 的参数，只要存在一个隐式转换能够将类型 x 的对象转换为类型 y 的对象！  
同样道理，T 并不需要支持 operator!=，因为以下这样也是可以的：operator！= 接受一个类型为 x 的对象和一个类型为 Y 的对象，T 可被转换为 x 而 EClassType 的类型可被转换为 Y，这样就可以有效调用 operator !=。

总结：

1.  classes 和 templates 都支持接口（interfaces）和多态（polymorphism）。
2.  对 classes 而言接口是显式的（explicit），**以函数签名为中心**。多态则是通过 virtual 函数发生于**运行期**。
3.  对 template 参数而言，接口是隐式的（implicit），**奠基于有效表达式**。多态则是通过 template 具现化和函数重载解析（function overloading resolution）发生于**编译期**。

### 条款 42：了解 typename 的双重意义

本条款首先提出一个问题：以下 template 声明式中，class 和 typename 有什么不同

```
template<class T>
class Widget;

template<typename T>
class Widgt;
```

答案：没有不同。

**当我们声明 template 类型参数时， class 和 typename 的意义完全相同。**  
某些程序员喜欢 class，因为可以少打几个字, 有些人 (比如作者本人) 比较喜欢 typename ，因为它**暗示参数并非一定是个 class 类型**。

**然而 C++ 并不总是把 class 和 typename 视为等价**。有时候你一定得使用 typename。为了解其时机，我们必须先谈谈你可以在 template 内指涉（refer to）的两种名称： (嵌套) 从属名称和非从属名称 。

```
template<typename T>
void PrintContainer(const T& Container) {                                           //注意这不是有效的C++代码
    if (Container.Size()>0) {
        T::const_iterator iter(Container.begin()); //取得第一元素的迭代器  注意 iter
        ++iter;                         
        int value=*iter;                    //将该元素复制到某个int，注意 value
        std::cout<<value<<"\n";
    }
}
```

在上述代码中强调两个 local 变量：**iter** 和 **value**。  
iter 的类型是`T::const_iterator`，实际是什么取决于 template 参数 T。template 内出现的名称如果依赖于某个 template 参数，我们就称之为**从属名称**（dependent names）。如果从属名称在 class 内呈嵌套状，我们就称之为**嵌套从属名称**（nested dependent names）。T::const_iterator 就是这样的名称，实际上它还是一个**嵌套从属类型名称**（nested dependent type name），也就是个**嵌套从属名称**并且**指涉是什么类型**。  
value 的类型是 int 。它不依赖于任何 template 参数。我们便称之为非从属名称（non-dependent names）。

**嵌套从属名称可能导致解析（parsing）困难**

在我们知道 T 是什么之前，没有任何办法可以知道 T::const_iterator 是否为一个类型。而当编译器开始解析 template PrintContainer 时，尚未确知 T 是什么东西。 C++ 有个规则可以解析（resolve）此一歧义状态：**如果解析器在 template 中遭遇一个嵌套从属名称，它便假设这名称不是个类型，除非你告诉它是**。所以缺省情况下嵌套从属名称不是类型。此规则有个例外，稍后我会提到。

再次回顾上述代码:

```
template<typename T>
void PrintContainer(const T& Container) {                                           
    if (Container.Size()>0) {
        T::const_iterator iter(Container.begin());  //这个名称被假设为非类型
        ......
    }
}
```

现在应该很清楚为什么这不是有效的 C++ 代码了吧。iter 声明式只有在 T::const_iterator 是个类型时才合理，但我们并没有告诉 C++ 说它是，于是 C++ 假设它不是。若要矫正这个形势，我们必须告诉 C++ 说 T::const iterator 是个类型。**只要紧临它之前放置关键字 typename 即可：**

```
template<typename T>
void PrintContainer(const T& Container)      //这是合法的C++代码 {                                           
    if (Container.Size()>0) {
        typename T::const_iterator iter(Container.begin());  //ok
        ......
    }
}
```

一般性规则很简单：**任何时候当你想要在 template 中指涉一个嵌套从属类型名称，就必须在紧临它的前一个位置放上关键字 typename。**（再提醒一次，很快我会谈到一个例外。）

**typename 只被用来验明嵌套从属类型名称**；其他名称不该有它存在。例如下面这个 function template，接受一个容器和一个 “指向该容器” 的选代器：

```
template<typename C>                 //允许使用"typename"（或"class"）
void f(const C&container，           //不允许使用"typename"
typename C::iterator iter)；        //一定要使用"typename"
```

上述的 C 并不是嵌套从属类型名称（它并非嵌套于任何 “取决于 template 参数” 的东西内），所以声明 container 时并不需要以 typename 为前导，但 C::iterator 是个嵌套从属类型名称，所以必须以 typename 为前导。

“typename 必须作为嵌套从属类型名称的前缀词” 这一规则的例外是：**typename 不可以出现在 base classes list 内的嵌套从属类型名称之前，也不可在 member initialization list（成员初值列）中作为 base class 修饰符。**

```
class Message {
public:
    Message() = default;
    explicit Message(std::string InText): Text(std::move(InText)) {}

    void SetText(std::string InText) {
        Text = std::move(InText);
    }

    std::string GetText() {
        return Text;
    }

private:
    std::string Text;
};

template <typename T>
class MessageContainer {
public:
    typedef T ElementType;
};

template <typename T>
class Printer : private MessageContainer<T>::ElementType {//base classes list不使用typename
public:
    //使用 typename 表明是一个类型，而不是变量
    //使用 typedef 给过长的类型起别名，方便。
    typedef typename MessageContainer<T>::ElementType ElementType; 

    explicit Printer(): MessageContainer<T>::ElementType() { //mem.init.list中不使用typename

        std::cout << typeid(ElementType).name() << "\n"; //class Message

    }

    void Log(std::string Text) {
        ElementType::SetText(Text);
        std::cout << ElementType::GetText() << "\n";
    }
};

inline void TryWithPrinter() {
    Printer<Message> Printer;
    Printer.Log("HelloWorld");
}
```

总结：

1.  声明 template 参数时，前缀关键字 class 和 typename 可互换。
2.  请使用关键字 typename 标识**嵌套从属类型名称**；但**不得在 base class lists（基类列）**或 **member initialization lists（成员初值列）**内以它作为 base class 修饰符。

### 条款 43：学习处理模板化基类内的名称

从一个例子入手，假设我们要设计游戏中人物的相关列表，比如 buff 列表，物品列表等等，一个显而易见的设计是：

```
#include <iostream>
#include <set>

class Buff {
public:
    virtual ~Buff() = default;
    virtual void Start() = 0;
    virtual void End() = 0;
    virtual void OnTick() = 0;
};

class RedBuff : public Buff {
public:
    virtual void Start() override {}
    virtual void End() override {}
    virtual void OnTick() override {}
};

class BlueBuff : public Buff {
public:
    virtual void Start() override {}
    virtual void End() override {}
    virtual void OnTick() override {}
};

template <typename T>
class Container {
public:
    void Add(T Item) {
        std::cout << "Add" << "\n";
        Data.insert(Item);
    }

    void Clear() {
        std::cout << "Clear" << "\n";
        Data.clear();
    }

    size_t Size() {
        return Data.size();
    }

private:
    std::set<T> Data;
};

template <typename T>
class PlayerContainer : public Container<T> {
public:
    void RemoveAll() {
      Clear();   //在此无法访问 Clear 函数,找不到标识符
    }

    void ShowAll() {
        //...
    }
};

int main() {
    PlayerContainer<Buff*> PlayerBuffContainer;
    PlayerBuffContainer.Clear();

    RedBuff* RedBuffOne = new RedBuff;
    BlueBuff* BlueBuffOne = new BlueBuff;
    PlayerBuffContainer.Add(RedBuffOne);
    PlayerBuffContainer.Add(BlueBuffOne);

    std::cout << PlayerBuffContainer.Size() << "\n";
}
```

运行此代码，出现错误

```
Clear:找不到标识符!
```

而出错的原因在于：

当编辑器遭遇 class template PlayerContainer 时，其实并不知道它究竟继承哪个 class。当然它继承的是 Container ，但其中的 T 是一个 template 参数，**不到后来的具现化，是无法确切知道它是什么**。而如果不知道 T 是什么，就不清楚 class Container 看起来像什么——更确切地说是没办法知道它是否有个 Clear 函数。  
例如，如果有以下**特化版** class Container （模板全特化）

```
template<>                 //一个全特化的Container
class Container<Buff*> {   //它和一般的template相同，区别只在于它删掉了void Clear() 函数
public:
 void Add(T Item) {
     std::cout << "Add" << "\n";
     Data.insert(Item);
 }
 
 size_t Size() {
     return Data.size();
 }
};

//等价于
template<typename T=Buff*>
class Container<T> {
public:
  .....
};
```

现在，再让我们考虑 derived class PlayerContainer:

```
template <typename T>
class PlayerContainer : public Container<T> {
public:
 void RemoveAll() {
     Clear();   //如果T==Buff,这个函数不存在
 }

 void ShowAll() {
     //...
 }
};
```

正如注释所言，**当 base class 被指定为 Container 时，这段代码将不合法！**因为该版本的 Container template 类**被特化**，其中并**不存在 Clear 函数**，且由于编译器会**优先考虑特化版本**，意味着 Container 使用 Buff 具现化时类中只存在 Add、Size 函数，并未提供 Clear 函数。  
这正是前面所说，为什么 C++ 拒绝在 PlayerContainer 访问 Clear 函数的原因：它知道 base classes templates 有可能被特化，而那个**特化版本可能不提供和一般性 template 相同的接口。因此它往往拒绝在 base classes templates 寻找继承而来的名称**。因此它往往拒绝在 templatized base classes（模板化基类，本例的 Container）内寻找继承而来的名称（本例的 Clear）。

所以，**我们必须有某种办法令 C++" 不进入 templatized base classes 观察” 的行为失效**。幸运的是，我们有三个解决办法：

1. 在 base class template 函数调用动作之前**加上 this->**。this 指针可以访问所有成员函数。

```
template <typename T>
class PlayerContainer : public Container<T> {
public:
    void RemoveAll() {
      this -> Clear();   //成立，假设Clear将被继承
    }
	...
};
```

**2. 使用 using 声明式**。可以告诉编译器进入 base class 作用域寻找函数。

```
template <typename T>
class PlayerContainer : public Container<T> {
public:
    using Container<T>::Clear; //告诉编译器，请他假设Clear位于base class内
    void RemoveAll() {
       Clear();   
    }
	...
};
```

（虽然 using 声明式在这里或在条款 33 都可有效运作，但两处解决的问题其实不相同。这里的情况并不是 base class 名称被 derived class 名称遮掩，而是编译器不进入 base class 作用域内查找，于是我们通过 using 告诉它，请它那么做。）  

3. 明确指出被调用函数位于 base class 内。 (不推荐)

```
template <typename T>
class PlayerContainer : public Container<T> {
public:
    void RemoveAll() {
      Container<T>::Clear();   //成立，假设Clear将被继承
    }
	...
};
```

但这往往是最不让人满意的一个解法，因为**如果被调用的是 virtual 函数**，上述的明确资格修饰（explicit qualification）会**关闭 "virtual 绑定行为”**  
从名称可视点的角度来看，上述每一个解法做的事情都相同：**对编译器承诺 base class template 的任何特化版本都支持其泛化版本所提供的接口。如果承诺未被保证，编译器仍然会报错。**

总结：

1.  **可在 derived class templates 内通过 this-> 指涉 base class templates 内的成员名称，或藉由一个明白写出的 base class 资格修饰符完成。**

### 条款 44：将与参数无关的代码抽离 templates

*   **templates 是节省时间和避免代码重复的奇方妙法。**

你不再需要键入 20 个类似的 classes 并且每一个都带有 20 个 成员函数，你只需要键入一个 class template，留给编译器去具现化那 20 个你需要的相关 classes 即可，而且对于 20 个函数中未被调用的，编译器不会自动生成。这样的技术是不是很伟大，呵呵。

*   但这也很容易使得**代码膨胀**（code bloat），templates 产出码带着重复，或者几乎重复的代码，数据，或者两者。你可以通过：**共性与变形分析**（commonality and variability analysis）来避免代码膨胀。

这个概念其实你早在使用，即使你从未写过一个 templates。当你编写某个函数时，你明白其中某些部分的实现码和另一个函数的实现码实质相同，你会很单纯的重复它们吗？当然不，你会抽出这两个函数相同的部分，放进第三个函数中，然后令原先两个函数调用这个新函数。也就是说：你分析了两个函数的共性和变形，把公共的部分搬到一个新的函数中去，变化的部分保留在原来的函数不动。对于 class 也是这个道理，如果你明白某些 class 和另一个 class 具有相同的部分，你也会把共性搬到一个新的 class。

*   templates 的优化思路也是如此，以相同的方式避免重复，但其中有个窍门。在 non-template 代码中，重复很明确。然而**在 template 代码中，重复是隐晦的，**毕竟只存在一份 template 代码，所以你必须自己去感受 template 具现化时可能发生的重复。

造成代码膨胀的一个典型的例子： **template class 成员依赖 template 参数值**

```
// 典型例子
template <typename T, std::size_t n>
class SquareMatrix {
  public:
    void invert();
};

SquareMatrix<double, 5> m1;
SquareMatrix<double, 10> m2;
```

会具现两份非常相似的代码，因为除了一个参数 5，一个参数 10，其他都完全一样

**改进**一**使用带参数值的函数**

```
template <typename T>
class SquareMatrixBase {
  protected:            // protected 保证只有本类/子类本身可以调用
    void invert(std::size_t n);
}

template<typename T, std::size_t n>
class SquareMatrix  : private SquareMatrixBase<T> { // private继承，derived 和base不是is-a关系，base只是帮助实现derived 
  private:
    using SquareMatrixBase<T>::invert;  // derived class 会掩盖template base class的函数

  public:
    inline void invert() { this->invert(n);}
}
```

如上，SquareMatrixBase 只对 “矩阵元素对象的类型” 参数化，不对矩阵的尺寸参数化。因此对于某给定元素类型，所有矩阵共享同一个 SquareMatrixBase 类。  
SquareMatrixBase::invert 只是企图成为 “避免派生类代码重复” 的一种方法，所以它**用 protected 替换 public**。调用它而造成的额外成本应该是 0(因此派生类的 invert 调用基类版本的 invert 时是 inline 调用)。这里函数使用`this->`，否则**模板化基类的函数名称会被派生类掩盖**。注意这里是 **private 继承**，说明了这里的基类只是为了帮助派生类的实现，不是为了表现 SquareMatrixBase 和 SquareMatrix 的 is-a 关系。

目前为止一切都好，但还有一些问题没有解决：

SquareMatrixBase::invert 如何知道该操作什么数据？  
虽然它从参数中知道矩阵尺寸，但它如何知道哪个矩阵的数据在哪儿？想必只有派生类知道。  
派生类如何联络其基类做逆运算动作？

解决办法： **令 SquareMatrixBase 存储一个指针，指向矩阵数值所在的内存**

```
template <typename T>
class SquareMatrixBase {
public:
    SquareMatrixBase(std::size_t InN, T* InData): N(InN), Data(InData) {}

    void Invert() const {
        std::cout << N << "\n";
    }

private:
    std::size_t N;  // 矩阵的大小
    T* Data;        // 指向矩阵内容
};

template <typename T, size_t N>
class SquareMatrix : private SquareMatrixBase<T> {
public:
    SquareMatrix()
        : SquareMatrixBase<T>(N, Data) {
        Data = new T[N * N];
    }

    void Invert() {
        SquareMatrixBase<T>::Invert();
    }

private:
    T* Data;
};

inline void TryWithMatrix() {
    SquareMatrix<int, 5> SquareMatrixFive;
    SquareMatrixFive.Invert();
}
```

这类类型的对象不需要动态分配内存，但对象自身可能非常大。另一种做法是把每一个矩阵的数据放进 heap

```
template<typename T, std::size_t n>
class SquareMatrix : private SquareMatrixBase<T>{
public:
    SquareMatrix () :  
     SquareMatrixBase<T>(n, 0),             //将基类的数据指针设为null
     pData(new T[n * n])   // 为矩阵内容分配内存, 将指向该内存的指出存储起来
     {this->setDataPtr(pData.get();)}

private:
    boost::scoped_array<T>pData;
}
```

这个条款只讨论由 non-type template parameters(非类型模板参数) 带来的膨胀，其实 type parameters (类型参数) 也会导致膨胀。

1. 比如在很多平台上，int 和 long 有相同的二进制表述，所以 `vector<int>` 和 `vector< long >` 的成员函数可能完全相同。  
2. 同样的，大多数平台上，所有指针类型都有相同的二进制表述，因此凡模板持有指针者 (比如 `list< int>`、`list< const int >` 等)往往应该对每一个成员使用唯一一份底层实现。  
3. 也就是说，如果你实现成员函数而它们操作强类型指针（T），你应该令它们调用另一个无类型指针 (void) 的函数，由后者完成实际工作。

总结:

1.  Templates 生成多个 classes 和多个 functions，所以任何 template 代码都不该与某个造成膨胀的 template 参数产生相依关系。
2.  因非类型模板参数（non-type template parameters）而造成的代码膨胀，往往可以消除，做法是以函数参数或 class 成员变量替换 template 参数。
3.  因类型参数（type parameters）而造成的代码膨胀，往往可以降低，做法是让带有完全相同二进制表述（binary representations）的具现类型（instantiation types）共享实现码。

### 条款 45：运用成员函数模板接受所有兼容类型

所谓智能指针（smart pointer），是行为像指针的对象，并提供指针没有的机能：自动管理资源。但原始指针（raw pointer）做的很好的一件事是：支持隐式转换（implicit conversions）。比如 derived class 指针可以隐式转换为 base class 指针，指向 non-const 的指针可以转换为 指向 const 的指针

```
class Top {...}
class Middle: public Top {...}
class Bottom:public Middle {...}
Top* pt1 = new Middle;    //将Middle*转换为Top*
Top* pt2 = new Bottom;    //将Bottom*转换为Top*
const Top* pct2 = pt1;      //将Top*转换为const Top*
```

而对于 template 具现的类，并不能很好的进行像原始指针一样的隐式转换，比如想把一个具现类转换为另一个具现类，这是不可以的，它们不存在像 derived-base 一样的关系，它们是完全不同的类。**唯一的方式就是我们明确的编写构造函数。**

也许我们可以对于某个具现类，编写特定的构造函数去变成另一个具现类，但这存在一个问题。因为一个 template 可以被无限的具现，意味着我们要提供无限的构造函数。因此，更好的解决方法是：**为它编写一个模板构造函数：**

```
template<typename T>
class SmartPtr {
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& Other) {
        //...
    }
};
```

但并不是所有的构造行为都是我们期望的，**我们必须有能力对模板构造函数进行筛选和剔除**。条款 41 提及的隐式接口是值得注意的，我们可以结合这点去进行约束。良好的接口设计可以避免不必要的构造，比如 shared_ptr 的实现：

```
template <typename T>
class AutoPtr{};
template <typename T>
class WeakPtr{};

template <typename T>
class SharedPtr {
public:
    template <typename Y>
    explicit SharedPtr(Y* InPointer);

    template <typename Y>
    explicit SharedPtr(SharedPtr<Y> const& InR);

    template <typename Y>
    explicit SharedPtr(WeakPtr<Y> const& InR);

    template <typename Y>
    explicit SharedPtr(AutoPtr<Y>& InR);

    template <typename Y>
    SharedPtr& operator=(SharedPtr<Y> const& InR);

    template <typename Y>
    SharedPtr& operator=(AutoPtr<Y>& InR);
};
```

明确指出了可以进行转换的类型，要比上个版本的转换函数安全且容易甄别错误。此外，还可以看出，模板函数还可以被用于赋值操作。

```
template<typename T>
class shared_ptr {
public:
 shared_ptr(T* ptr);  // 构造
 shared_ptr(const shared_ptr& ptr); // 拷贝构造
 // 下面的泛化拷贝构造函数没有使用explicit，因为derived->base支持隐式转换
 // 成员初始化列表使用U* ptr 赋值给T* ptr_,只有U->T存在隐式类型转换，此处才能编译成功，
 // 正好满足Dervied可以转化为Base,反向却不行的约束
 template<U>   // 泛化拷贝构造
 shared_ptr(const shared_ptr<U>& ptr) : ptr_(ptr.get()); 
 shared_ptr& operator=(shared_ptr const& ptr); // 拷贝赋值
 template<U>  // 泛化拷贝赋值
 shared_ptr& operator=(shared_ptr<U> const& ptr);
 T* get() {return ptr_};
private:
 T* ptr_;
};
```

提及构造函数，其实模板构造函数并不会影响语言规则，如果你的程序需要一个拷贝构造函数，而你却没有声明它。编译器依旧会为你生成一个。尽管你已经声明了模板构造函数，但那是只对泛化类型而言的，如果你对一个非泛化类型机型拷贝构造，模板函数就失去了作用。所以最好的方式是：**同时提供模板构造函数和正常构造函数。**

总结：

1.  **请使用 member function templates（成员函数模板）生成可接受所有兼容类型的函数。**
2.  **如果你声明 member templates 用于泛化 copy 构造 或 泛化 assignment 操作，你还是需要声明正常的 copy 构造函数和 copy assignment 操作符。**

### 条款 46：需要类型转换时请为模板定义非成员函数

学习本条款前建议先熟悉一下条款 24。

条款 24 讨论了为什么只有 non-member 函数才有能力在所有实参身上实施隐式类型转换。那么，如果是 class template 情况下又会发生什么呢？

```
template <typename T>
class TRational {
public:
    TRational(const T& mNumerator, const T& mDenominator):
        Numerator(mNumerator), Denominator(mDenominator) {}

public:
    T GetNumerator() const {
        return Numerator;
    }

    T GetDenominator() const {
        return Denominator;
    }

private:
    T Numerator;
    T Denominator;
};

template <typename T>
inline TRational<T> operator*(const TRational<T>& RationalOne, const TRational<T>& RationalTwo) {
    return TRational<T>(RationalOne.GetNumerator() * RationalTwo.GetNumerator(),
                     RationalOne.GetDenominator() * RationalTwo.GetDenominator());
}

inline void TryWithTRational() {
    const TRational<int> TempOne(1, 8);
    const TRational<int> TempTwo(1, 2);
    TRational<int> Result = TempOne * TempTwo; //很好
    Result = Result * TempOne; //很好
    Result = Result * 2; //错误
    Result = 2 * Result; //错误
}
```

就像条款 24 所期望的那样，我们希望支持混合式算术运算。所以我们希望这段代码也能通过编译并正确运行，毕竟这段代码相比于条款 24 的代码，唯一不同的是 Rational 和 operator * 如今都成了 templates。  
但事与愿违， 由于模板化带来了一些不同，导致编译器无法找到相应的函数进行调用，致使编译失败。  
实际上编译器试图想出什么函数被名为 operator* 的 template 具现化。它知道它们应该可以 **“具现化某个名为 operator* 并接受两个 TRational 的参数”** 的函数，但为了完成这一具现化的任务，它必须先算出 T 是什么。问题就在于它没有这个能力。  
以 Result * 2 为例，Result 是一个类型为 TRational 的参数，所以编译器可以得知 T 为 int 。其他的参数就没这样顺利，2 是一个 int ，编译器如何从 int 推算出 TRational 的 T 是什么类型呢？你也许期待编译器用 TRational 进行构造，但这是不行的，因为在 template 实参推导过程中：**从不将隐式类型转换函数纳入考虑。因为相应的隐式转换函数也需要知道 T 是什么类型才能被具现化。**

解决方法：**template class 内的 friend 声明式可以指涉某个特定函数**。

这意味 class Rational 可以声明 operator * 是它的一个 friend 函数。Class templates 并不倚赖 template 实参推导（后者只施行于 function templates 身上），所以**编译器总是能够在 class Rational 具现化时得知 T**。

```
template <typename T>
class TRational {
public:
    friend TRational<T> operator*(const TRational<T>& RationalOne, const TRational<T>& RationalTwo);
    //...
};

template <typename T>
inline TRational<T> operator*(const TRational<T>& RationalOne, const TRational<T>& RationalTwo) {
    return TRational<T>(RationalOne.GetNumerator() * RationalTwo.GetNumerator(),
                     RationalOne.GetDenominator() * RationalTwo.GetDenominator());
}

inline void TryWithTRational() {
    const TRational<int> TempOne(1, 8);
    const TRational<int> TempTwo(1, 2);
    TRational<int> Result = TempOne * TempTwo; //很好
    Result = Result * TempOne; //很好
    Result = Result * 2;//OK
    Result = 2 * Result; //Ok
}
```

注意： 这段代码虽然可以通过编译，但是仍会有**链接错误**（稍后再说）  
现在对`operator*`的混合式调用可以通过编译了，因为当对象 Result 被声明为一个 TRational，class TRational 于是被具现化出来，而作为过程的一部分，friend 函数`operator*`（接受 TRational 参数）也就被自动声明出来。 后者身为一个函数而非函数模板（function template），因此编译器可在调用它时使用隐式转换函数（例如 TRational 的 non-explicit 构造函数），而这便是混合式调中之所以成功的原因。  
小技巧：当一个 class template 内，template 名称可以作为 template 声明 的简略表达形式，所以在 TRational 我们可以只写 Rational 而不必写 TRational，对于有很多参数的 template，这样可以节省一些时间，并让代码看起来干净，当然为了一致性，意义也并不大：

```
template <typename T>
class TRational {
public:
 friend TRational operator*(const TRational& RationalOne, const TRational& RationalTwo);
 //...
};
```

**那为什么会有链接错误呢？**

这是因为编译器虽然知道要调用这个函数，但该函数只被用 **friend 声明于 template 内，并没有实际定义**。可惜的是虽然声明式知道了 T 是 int，但是类外的函数模板仍不知道，因为它们并无实际关系。

解决办法就是：**将 operator* 的函数体从类外合并到 template 声明式中** 。

```
template <typename T>
class TRational {
public:
    TRational(const T& mNumerator =0, const T& mDenominator=1):
        Numerator(mNumerator), Denominator(mDenominator) {}

public:
    T GetNumerator() const {
        return Numerator;
    }

    T GetDenominator() const {
        return Denominator;
    }
    friend TRational operator*(const TRational& RationalOne, const TRational& RationalTwo) {
        return TRational<T>(RationalOne.GetNumerator() * RationalTwo.GetNumerator(),
                 RationalOne.GetDenominator() * RationalTwo.GetDenominator());
    }
private:
    T Numerator;
    T Denominator;
};

inline void TryWithTRational() {
    const TRational<int> TempOne(1, 8);
    const TRational<int> TempTwo(1, 2);
    TRational<int> Result = TempOne * TempTwo; //很好
    Result = Result * TempOne; //很好
    Result = Result * 2;//编译ok，运行ok too
    Result = 2 * Result; //编译ok，运行ok too
}
//终于，混合式运算的问题得到了解决！
```

该解决方法的有趣之处在于：

虽然我们使用 friend 关键字，却和其传统用途：访问 class 的 non-public 成分不同。我们是**为了让类型转换发生在所有可能的实参上，我们需要一个 non-member 函数**，而**为了使这个函数自动具现化，我们需要将它声明在 class 内部**，而在 class 内部声明 non-member 函数的唯一有效方法就是：**令它成为一个 friend。**

优化： **令该 friend 函数调用另一个辅助函数**

一如条款 30 所说，**定义于 class 内的函数都暗自成为 inline**，包括像`operator*`这样的 friend 函数。你可以将这样的 inline 声明所带来的冲击最小化，做法是令`operator*`不做任何事情，**只调用一个定义于 class 外部的辅助函数**。在本条款的例子中，这样做并没有太大意义，因为`operator*`已经是个单行函数，但对更复杂的函数而言，那么做也许就有价值。“令 friend 函数调用辅助函数” 的做法的确值得细究一番。

```
template <typename T>
class TRational {
public:
    friend TRational operator*(const TRational& RationalOne, const TRational& RationalTwo) {
        return OnMultiply(RationalOne,RationalTwo);
    }
    //...
};
template <typename T>
TRational<T> OnMultiply(const TRational<T>& RationalOne, const TRational<T>& RationalTwo){
    return TRational<T>(RationalOne.GetNumerator() * RationalTwo.GetNumerator(),
                     RationalOne.GetDenominator() * RationalTwo.GetDenominator());
}
```

总结：

1.  **当我们编写一个 class template，而它所提供之与此 template 相关的函数支持所有参数隐式类型转换时，请将那些函数定义为 class template 内部的 friend 函数。**

### 条款 47：请使用 traits classes 表现类型信息

第一次看该条款时，觉得云里雾里模模糊糊的，于是查了很多资料，其中觉得这一篇文章讲的最为清晰，由浅入深，结合书本一起看效果极佳。

[原文链接：http://t.csdn.cn/eAPtl](https://link.zhihu.com/?target=http%3A//t.csdn.cn/eAPtl)  
除此之外还强烈建议阅读：[http://t.csdn.cn/X2Hug](https://link.zhihu.com/?target=http%3A//t.csdn.cn/X2Hug) 以及 Cpp 技术文章 08

注意：看本条款时请先熟悉**模板特化**、**偏特化**以及 **typename 关键字**。

我们知道，在 STL 中，容器与算法是分开的，彼此独立设计，容器与算法之间通过迭代器联系在一起。那么，算法是如何从迭代器类中萃取出容器元素的类型的？没错，这正是我们要说的 traits classes 的功能。 迭代器所指对象的类型，称为该迭代器的 value_type。我们来简单模拟一个迭代器 traits classes 的实现。

```
template<class IterT>
struct my_iterator_traits {
    typedef typename IterT::value_type value_type;
};
```

my_iterator_traits 其实就是个类模板，其中包含一个类型的声明。有`typename`的基础，相信大家不难理解 `typedef typename IterT::value_type value_type;` 的含义：将迭代器的`value_type` 通过`typedef` 为 `value_type`。

对于`my_iterator_traits`，我们再声明一个偏特化版本。

```
template<class IterT>
struct my_iterator_traits<IterT*> {
    typedef IterT value_type;
};
```

即如果 `my_iterator_traits` 的实参为指针类型时，直接使用指针所指元素类型作为 `value_type`。

为了测试 `my_iterator_traits` 能否正确萃取迭代器元素的类型，我们先编写以下的测试函数。

```
void fun(int a) {
    cout << "fun(int) is called" << endl;
}

void fun(double a) {
    cout << "fun(double) is called" << endl;
}

void fun(char a) {
    cout << "fun(char) is called" << endl;
}
```

我们通过函数重载的方式，来测试元素的类型。

测试代码如下：

```
my_iterator_traits<vector<int>::iterator>::value_type a;
fun(a);  // 输出 fun(int) is called
my_iterator_traits<vector<double>::iterator>::value_type b;
fun(b);  // 输出 fun(double) is called
my_iterator_traits<char*>::value_type c;
fun(c);  // 输出 fun(char) is called
```

为了便于理解，我们这里贴出 vector 迭代器声明代码的简化版本：

```
template <class T, ...>
class vector {
public:
    class iterator {
    public:
        typedef T value_type;
        ...
    };
...
};
```

我们来解释 `my_iterator_traits::iterator>::value_type a;` 语句的含义。`vector::iterator` 为`vector` 的迭代器，该迭代器包含了 `value_type` 的声明，由 vector 的代码可以知道该迭代器的`value_type` 即为 int 类型。

接着，`my_iterator_traits::iterator>` 会采用 `my_iterator_traits` 的通用版本，即 `my_iterator_traits::iterator>::value_type` 使用 `typename IterT::value_type` 这一类型声明，这里 `IterT` 为 `vector::iterator`，故整个语句萃取出来的类型为 int 类型。

对 double 类型的 vector 迭代器的萃取也是类似的过程。  
而`my_iterator_traits<char*>::value_type`则使用`my_iterator_traits`的偏特化版本，直接返回 char 类型。  
由此看来，通过`my_iterator_traits`，我们正确萃取出了迭代器所指元素的类型。

总结一下我们设计并实现一个 traits class 的过程：

1）确认若干我们希望将来可取得的类型相关信息，例如，对于上面的迭代器，我们希望取得迭代器所指元素的类型；  
2）为该信息选择一个名称，例如，上面我们起名为 value_type；  
3）提供一个 template 和一组特化版本（例如，我们上面的 my_iterator_traits），内容包含我们希望支持的类型相关信息。

### 条款 48：认识 template 元编程

**template metaprogramming（TMP，模板元编程）**：编写 template C++ 程序并执行于编译期的过程。

所谓模板元程序就是：**以 C++ 写成，执行于 C++ 编译器内的程序**。该程序执行后产生具现的代码，和正常代码一并加入编译。即元编程可以做到**用代码去生成代码**。

由于 template metaprograms **执行于 C++ 编译期**，因此可以将很多工作从运行期转移到编译期。如：

某些错误原本通常在运行期才能检测到，现在可在编译器找出来。  
使用 TMP 的 C++ 程序可能在每一方面都更加高效：比如较小的可执行文件，较短的运行期，较少的内存需求。  
注意：将工作移至编译期，会**导致编译时间变长**

条款 47 实现一个 Move 函数的伪代码 , 曾提到可能存在编译问题

```
template <typename IteratorType>
    void Move(IteratorType& Iterator, int Distance) {
        // 使用类型信息
        if (typeid(IteratorTraits<IteratorType>::IteratorTag) == typeid(RandomAccessIteratorTag)) {   
            Iterator += Distance; //针对random access 迭代器使用迭代器算术运算
        }
        else {
            if (Distance >= 0) {  //针对其他迭代器类型,反复调用＋＋或－－
                while (Distance--)++Iterator;
            }
            else {
                while (Distance++)--Iterator;
            }
        }
    }
```

虽然我们这里根据迭代器类型进行不同的操作，或 +=，或 ++，–，我们知道只有 Random Access Iterator 可以有 += 运算，但是 C++ 要求：**编译器必须确保所有源码都有效，即使是不会执行的源码。**也就是说编译器会拿着其他不支持 += 的迭代器，进入 if 语句先测试是否支持 += 运算，无效则会报错。  
所以相比于要支持所有操作，Traits class 针对不同类型进行函数重载的做法显然更好。

TMP 已被证明是一个图灵完备（Turing-complete）机器

这意味着它可以计算任何事物，使用 TMP 你可以声明变量，执行循环，编写及调用函数… 但这些相对于正常的 C++ 的实现会有很大的不同。比如：TMP 并没有循环部件，所有的循环效果都由递归完成。

一个经典的初级案例—— 利用 TMP 在编译期计算阶乘

```
template <unsigned N>
struct Factorial {
    static const int Value = N * Factorial<N - 1>::Value;
};

template <>
struct Factorial<0> {
    static const int Value = 1;
};

inline void TryWithFactorial() {
    std::cout << Factorial<10>::Value << "\n";
}
```

和所有递归行为一样，我们**需要一个特殊情况来结束递归**。对于 TMP 而言就是**使用 tmeplate 的特化版本** Factorial<0> 。 正如 TryWithFactorial 函数所使用的，只要你声明 Factorial::Value 就可以得到 N 阶乘值。当然这里存在值溢出的问题。

总结：

1.  Template metaprogramming（TMP，模板元编程）可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率。
2.  TMP 可被用来生成基于政策选择组合（based on combination of policy choices）的客户定制代码，也可用来避免生成对某些特殊类型并不合适的代码。

# **八、定制 new 和 delete**

### **条款 49：了解 new-handler 行为**

当你调用 operator new 函数，程序无法满足某一内存需求时，它会抛出异常。老旧的编译器会返回 null 指针。而抛出异常之前，程序会先调用一个 operator new 错误处理函数，名叫 `new-handler`。

**new-handler 是一个 typedef**，指向一个无参数值无返回值的函数。我们可以通过 set_new_handler 函数去指定客户想要的 new-handler。  
set_new_handler 函数接受一个新的 new-handler 参数，返回被替换掉的 new-handler 函数。

一个设计良好的 new-handler 函数必须考虑以下几点：

1. **提供更多的可被使用的内存。**这可以保证下次在 operator new 内部尝试分配内存时能够成功。实现这个策略的一种方法是在程序的开始阶段分配一大块内存，然后在第一次调用 new-handler 的时候释放它。  
2. 安装一个不同的 new-handler。如果当前的 new-handler 不能够为你提供更多的内存，可能另外一个 new-handler 可以。如果是这样，可以在当前的 new-handler 的位置上安装另外一个 new-handler（通过调用 set_new_handler）。下次 operator new 调用 new-handler 函数的时候，它会调用最近安装的。（这个主题的一个变种是一个使用 new_handler 来修改它自己的行为，所以在下次触发这个函数的时候，它就会做一些不同的事情。达到这个目的的一个方法是让 new_handler 修改影响 new-handler 行为的 static 数据, 命名空间数据或者全局数据。）  
3. 卸载 new-handler，也就是为 set_new_handler 传递 null 指针。如果没有安装 new-handler，operator new 在内存分配失败的时候会抛出异常。  
4. 抛出 bad-alloc，或派生自 bad-alloc 的异常。  
5. 没有返回值，**调用 abort 或者 exit**。

有时候你或许希望以不同的方式处理内存分配的情况，比如按不同的 class 进行处理，但是 C++ 并不支持为每一个 class 提供专属版本的 new_handler，好在我们可以模仿这一行为，只要我们为 class 实现自己的 set_new_handler 函数 和 operator new 函数即可。

*   对于 set_new_handler ，我们根据参照默认实现即可

```
static std::new_handler SetNewHandler(std::new_handler NewHandler) throw() {
        const std::new_handler OldHandler=std::set_new_handler(NewHandler);
        CurrentHandler=OldHandler;
        return OldHandler;
    }
```

*   对于 operator new，我们要做以下事情。

调用标准版 set_new_handler 安装我们自定义的 new-handler，将返回的标准版 new-handler 保存起来。 调用标准版 operator new。如果标准版 operator new 异常，那么会调用我们自定义的 new-handler 处理函数。 调用标准版 set_new_handler 重新安装标准版的 new-handler。

为了确保可以重新安装标准版 new-handler，我们可以采用条款 13 所说`以对象管理资源`的方法：

```
class NewController {
public:
    explicit NewController(std::new_handler InHandler): Handler(InHandler) {}
​
    ~NewController() {
        std::set_new_handler(Handler);
    }
​
private:
    std::new_handler Handler;
};
```

所以 operator new 实现如下：

```
void* operator new(std::size_t Size) throw(std::bad_alloc) {
    NewController(std::set_new_handler(CurrentHandler));
    return ::operator new(Size);
}
```

但是上述代码还是不够简洁，每一个 class 都要自己实现一个 set_new_handler 和 operator new 版本。一个更好的方式是`使用 template 进行模板编程，然后根据不同 class 进行特化和具现化`。完整实现如下：

```
template <typename T>
class NewHandlerSupport {
public:
    static std::new_handler SetNewHandler(std::new_handler NewHandler) throw() {
        const std::new_handler OldHandler = std::set_new_handler(NewHandler);
        CurrentHandler = OldHandler;
        return OldHandler;
    }
​
    void* operator new(std::size_t Size) throw(std::bad_alloc) {
        NewController(std::set_new_handler(CurrentHandler));
        return ::operator new(Size);
    }
private:
    static std::new_handler CurrentHandler;
};
template <typename T>
std::new_handler  NewHandlerSupport<T>::CurrentHandler = nullptr;
​
class FDemo:public  NewHandlerSupport<FDemo> {
    
};
​
inline void TryWithNew() {
    FDemo::SetNewHandler([]() {
        std::cout<<"内存不够啦"<<"\n";
    });
    FDemo *Demos=new FDemo[1000123123100000]();
}
```

注意，**当 operator new 无法满足内存申请时，它会不断调用 new-handler 函数，直到找到足够内存或异常退出。** 当然，你想说为什么我们需要 template？我们似乎并没有使用到模板参数，是的，T 的确不被需要，我们只是希望，继承自 NewHandlerSupport 的 class 拥有各自的 CurrentHandler 成员。类型参数只是用来区分不同的派生类，**然后 template 机制会自动为每一个 T 具现化一份 CurrentHandler 成员，即使它是 static 的。** 也许你的焦虑还来自于 template class 导致的多重继承，可以先看看条款 40。

总结：

1.  **set_new_handler 允许客户指定一个函数，在内存分配无法获得满足时被调用。**
2.  **Nothrow new 是一个颇为局限的工具，因为它只适用于内存分配；后继的构造函数调用还是可能抛出异常。**

### **条款 50：了解 new 和 delete 的合理替换时机**

替换缺省 new/delete 的三个常见原因：

1. 用来检测运行上的错误。自定义 new 分配超额内存，在额外空间放置特定签名 / byte pattern。在 delete 时检查是否不变；反之，肯定存在 “overruns”（写入点在分配区块尾部之后）或 “unferruns”（写入点在分配区块头部之前），delete 也可 log 那个指针。 2. 为了强化效能。缺省版 new/delete 必然比定制版 new/delete 效率低。 3. 为了收集使用上的统计数据。自定义 new/delete 可以收集内存使用习惯与使用寿命。

当一定要写相关 new/delete 代码时，参考成熟的开源代码十分必要（条款 54/55：TR1 及 Boost 的 Pool 库）。

本条款的主题是，了**解何时可在 "全局性的" 或 "class 专属的" 基础上合理替换缺省的 new 和 delete**。在这之前，先对答案做一些摘要：

*   为了检测运用错误（如前所述）。
*   为了收集动态分配内存的使用统计信息（如前所述）。
*   为了增加分配和归还的速度。
*   为了降低缺省内存管理 s 器带来的空间额外开销。
*   为了弥补缺省分配器中的非最佳齐位。
*   为了将相关对象成簇集中。降低 “内存页错误”（page fault）的频率，new/delete 的 “placement 版本”（条款 52）有可能完成。
*   为了获得非传统的行为。

总结：

1.  **有许多理由需要写个自定的 new 和 delete，包括改善性能，对 heap 运用错误进行调用，收集 heap 使用信息。**

### **条款 52：写了 placement new 也要写 placement delete**

1.`placement new` 和 `placement delete` 在 C++ 中并不常见，如果不熟悉也不用太焦虑。 请回忆一下条款 16 和 17，当你写一个 new 表达式时：

```
String* Str = new String("Hello")；
```

共有两个函数被调用：一个是用以分配内存的 **operator new**，一个是 **String 的 default 构造函数**。  
假如第一个函数调用成功，第二个函数却抛出异常。那么运行期系统必须回收第一个函数分配的内存，否则就会发生资源泄漏。在这个时候，客户没有能力归还内存，因为如果 String 构造函数抛出异常，str 尚未被赋值，客户手上也就没有指针指向该被归还的内存。取消步骤一并恢复旧观的责任因此落到 C++ 运行期系统身上。 运行期系统就会调用步骤一所调用的 operator new 的相应 operator delete 版本，前提是，**系统必须知道哪一个 operator delete 该被调用**，因为可能存在多个 operator delete 函数（可能接受不同的参数列表）。

2. 对于`placement new/delete` ，它们接受额外的参数 。当人们谈及 placement new 时，大多数是指具有唯一额外实参 void* 的 operator new，少数时候才是指具有任意额外实参的 operator new。

**当抛出异常时，运行期系统会寻找参数个数和类型都与 operator new 相同的某个 operator delete**。比如 operator new 额外接受一个 string 参数，那么 operator delete 也需要提供一个额外的 string 参数。如果并没有这样的 operator delete 函数，那么系统什么也不会做，内存就会泄漏掉。

3. 值得注意的是，**placement delete 只有在 placement new 的调用构造函数异常时才会被系统调用** (即使我们可以显式调用 placement new，)。即使你对一个用 placement new 申请出的指针使用 delete，也绝不会调用 placement delete。这意味着额外的参数并不提供实际的作用。

所以，如果要处理 placement new 相关的内存泄漏问题，我们**必须同时提供一个正常版本的 delete 和 placement 版本的 delete**。前者用于构造期间无异常抛出，后者用于构造期间有异常抛出。

除此之外，还要注意**同名函数遮掩调用的问题**

当你为 class 声明了 placement new 时，客户是无法使用标准版的 operator new 的，因为 derived class 声明的 operator new 会遮掩标准版本和 base class 版本。 所以如果你需要的客户在使用标准版本不受影响，也**需要同时提供标准版的定义**。

满足以上注意事项的一个简单做法是，`建立一个 base class`，内含所有标准版本的 new/delete，凡是想以写 placement 版本的 class 都可以继承自它，并使用 `using 声明式`使得标准版本在类中可见：

```
class FNewDeleteSupport {
public:
    // normal new/delete
    static void* operator new (std::size_t Size) throw(std::bad_alloc) {
        return ::operator new(Size);
    }
    static void operator delete (void* RawMemory) throw() {
        ::operator delete(RawMemory);
    }
    //placement new/delete
    static void* operator new (std::size_t Size,void *Ptr) throw() {
        return ::operator new(Size,Ptr);
    }
    static void operator delete (void* RawMemory,void *Ptr) throw() {
        ::operator delete(RawMemory,Ptr);
    }
    //nothrow new/delete
    static void* operator new (std::size_t Size,const std::nothrow_t& Nothrow) throw() {
        return ::operator new(Size,Nothrow);
    }
    static void operator delete (void* RawMemory,const std::nothrow_t& Nothrow) throw() {
        ::operator delete(RawMemory);
    }
};
class FDemo:public FNewDeleteSupport {
public:
    using FNewDeleteSupport::operator new;
    using FNewDeleteSupport::operator delete;
​
    //custom new/delete
    static void* operator new (std::size_t Size,std::string User) throw(std::bad_alloc) {
        std::cout<<User<<"使用了内存";
        return ::operator new(Size);
    }
    static void operator delete (void* RawMemory,std::string User) throw() {
        ::operator delete(RawMemory);
    }
};
```

总结:

1.  当你写一个 placement operator new，请确定也写出了对应的 placement operator delete。如果没有这样做，你的程序可能会发生隐微而时断时续的内存泄漏。
2.  当你声明 placement new 和 placement delete，请确定不要无意识（非故意）地遮掩了它们的正常版本。

# **九、杂项讨论**

### **条款 53：不要轻忽编译器的警告**

许多程序员习惯性的忽略编辑器警告，这并不是一个好习惯。如：

```
class B{
public:
    virtual void f() const;
};
​
class D{
public:
    virtual void f()''
};
```

这里希望以 D::f 重新定义 virtual 函数 B::f，但其中有个错误：B 中的 f 是个 const 成员函数，而在 D 中它未被声明为 const。我手上的一个编译器于是这样说话了：`warning: D::f() hides virtual B::f()`  
如果你认为：“当然，D::f 遮掩了 B::f，那正是想象中该有的事！”  
那就大错特错了，该编译器试图告诉你声明于 B 中的 f 并未在 D 中被重新声明，而是被整个遮掩了（条款 33 描述为什么会这样）。如果忽略这个编译器警告，几乎肯定导致错误的程序行为，然后是许多调试行为，只为了找出编译器其实早就侦测出来并告诉你的事情。

因此，需要牢牢记住， **面对警告信息时，你一定要清楚的了解它的真实含义**，然后才可以选择性的处理或者忽略。

总结：

1.  **严肃对待编译器发出的警告信息。努力在你的编译器的最高（最严苛）警告级别下争取无任何警告的荣誉。**
2.  **不要过度倚赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦移植到另一个编译器上，你原来倚赖的警告信息有可能消失。**

### **条款 54：让自己熟悉包括 TR1 在内的标准程序库**

*   该部分建议学习`C++`新标准 (`C++11`等)。

### **条款 55：让自己熟悉 Boost**

`boost`库是一个优秀的，可移植的，开源的 `C++` 库，它是由 `C++` 标准委员会发起的，其中一些内容已经成为了下一代 `C++` 标准库的内容，在 `C++` 社区中影响甚大，是一个不折不扣的准标准库，它的功能十分强大，弥补了 `C++` 很多功能函数处理上的不足。

很多`boost`中的库功能堪称对语言功能的扩展，其构造用尽精巧的手法，不要贸然的花费时间研读。`boost`另外一面，比如`Graph`这样的库则是具有工业强度，结构良好，非常值得研读的精品代码，并且也可以放心的在产品代码中多多利用。

Boost 程序库涉及的领域很多：

*   字符串与文本处理，比如格式化字符串，正则表达式，语汇单元切割和解析。
*   容器，覆盖接口与 STL 相似的数组，bitsets，以及多维数组。
*   函数对象和高级编程，覆盖若干被作为 TR1 的程序库，一个有趣的程序库是 Lambda。
*   泛型编程，覆盖一大组 traits classes。
*   模板元编程，TMP 程序库。
*   数学和数值，包括有理数，八元数和四元数，公约数和少见的多重计算，随机数等等。
*   正确性和测试，覆盖用来将隐式模板接口形式化的程序库。
*   数据结构，覆盖类型安全的 unions，tuples。
*   语言间的支持，包括允许 C++ 和 Python 的无缝互操作性。
*   内存，覆盖 Pool 程序库，包括智能指针等。
*   杂项，包括 CRC 检验，日期和时间处理，文件系统操纵等。

总结：

1.  Boost 是一个社群，也是一个网站。致力于免费，源码开放，同僚复审的 C++ 程序库开发。 Boost 在 C++ 标准化过程中扮演深具影响力的角色。
2.  Boost 提供许多 TR1 组件实现品，以及其他许多程序库。


# 十、参考
[Effective C++学习笔记 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/552726093)
[https://normaluhr.github.io/2020/12/31/Effective-C++/](https://link.zhihu.com/?target=https%3A//normaluhr.github.io/2020/12/31/Effective-C%2B%2B/)  
[EffectiveC++_@Moota 的博客 - CSDN 博客](https://link.zhihu.com/?target=https%3A//blog.csdn.net/m0_51819222/category_11683735.html%3Fspm%3D1001.2014.3001.5482)  
