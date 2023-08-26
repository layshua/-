
---
banner: "![[Pasted image 20221223234905.png]]"
title: 《C++ Primer》
aliases: []
create_time: 2023-05-06 13:57
uid: 202305061357
cssclass: academia, academia-rounded
---
# 零、 预处理器

确保头文件多次包含仍能安全工作的常用技术是**预处理器**，预处理器是在编译之前执行的一段程序，任何以 # 开头的东西，都被称为预处理器命令或者预处理器指令。如 `#include` ，当预处理器看到 `#include`标记时就会用指定的头文件内容代替 `#include` 。
## 头文件保护符
下面来说 `# pargma once`

pragma 本质上是一个被发送到编译器或预处理器的预处理指令。
**pargma once 阻止我们单个头文件多次被 include 在同一个 cpp 文件里。** 

```c++
//新方法
#pargma once

//旧方法
#ifndef  _PROJECT_A
#define  _PROJECT_A
...
#endif
```

**有的 include 是 <> 有的是“”，原因是**：  
当我们编译程序的时候，它们有两种不同的含义，我们有能力告诉编译器，包含文件的路径是什么。  
如果我们要包含的文件是在其中的一个文件夹里（在包含路径文件夹里），我们可以使用尖括号在所有 include 路径里搜索文件； 
而引号用于 include 文件存在于该文件的相对位置（相对路径），现在也有尖括号的功能，所以作用范围更大。 

**所以尖括号只用于编译器包含路径，引号可以做一切。不过 cherno 一般引号中都是只用当前路径，即不用…/ 啥的，其他都是用尖括号。**  
建议：如果包含了一些 visual studio 解决方案之外的东西，就会用尖括号，也就是一些完全与项目无关的外部依赖项。
而像 `#include <iostream>`，iostream 其实是一个文件（虽然没有后缀。。）这是写 C++ 标准库的人决定要这样做的。将 C++ 标准库与 C 标准库进行区分。（有没有. h，如 stdlib. h，这是一种区分 C 标准库和 C++ 标准库的方法）
## 宏 macro 
#define #宏 #macro
1. **预处理阶段** ：当编译 C++ 代码时，首先**预处理器**会过一遍 C++ 所有的**以 # 符号开头（这是预编译指令符号）的语句，当预编译器将这些代码评估完后给到编译器去进行实际的编译**。

2. **宏和模板的区别**：**发生时间**不同，宏是在**预处理阶段**就被评估了，而模板会被评估的更晚一点。

3. **用宏的目的：**写一些宏将代码中的文本**替换**为其他东西（纯文本替换）**（不一定是简单的替换，是可以自定义调用宏的方式的）
```c++
#defind WAIT std::cin.get()
//这里可以不用放分号，如果放分号就会加入宏里面了
int main() {
    WAIT;
    //等效于std::cin.get()，属于纯文本替换
    //但单纯做这种操作是很愚蠢的，除了自己以外别人读代码会特别痛苦
}
```

4. 宏的用法之一：**宏是可以发送参数的**

```c++ 
#include <iostream>
#define log(x) std::cout << x << std::endl

int main() {
    log("hello");
    //这样子会输出“hello”
    return 0;
}
```

5. 宏可以辅助调试

在 Debug 模式下会有很多日志的输出，但是在 Release 模式下就不需要日志的输出了。正常的方法可能会删掉好多的输出日志的语句或者函数，**但是用宏可以直接取消掉这些语句**

利用宏中的 `#if，#else`，`endif` 来实现。如：

```c++
#include <iostream>

#defind PR_DEBUG 1 //可以在这里切换成0，作为一个开关
#if PR_DEBUG == 1   //如果PR_DEBUG为1
#defind LOG(x) std::cout << x << std::endl  //则执行这个宏
#else   //反之
#defind LOG(x)   //这个宏什么也不定义，即是无意义
#endif    //结束

int main() {
    LOG("hello");
    return 0;
}
```

如果在 Debug (PR_DEBUG == 1) 模式下，则会打印日志，如果在 Release (PR_DEBUG == 0) 模式，则在**预处理阶段就会把日志语句给删除掉**。

利用 `#if 0` 和 `#endif` 删除一段宏.

```c++
#include <iostream>

#if 0   //从这里到最后的endif的宏都被无视掉了，某种意义上的删除

#defind PR_DEBUG 1 
#if PR_DEBUG == 1 
#defind LOG(x) std::cout << x << std::endl 
#else 
#defind LOG(x) 
#endif 

#endif  //结束

int main() {
    LOG("hello");
    return 0;
}
```

### 续行符 `\`
普通代码行后面编译器自动判断续行的，宏定义则不行。

**宏定义规定必须用一行完成:**
```c++
#define SomeFun(x, a, b) if(x)x=a+b;else x=a-b;
```

这一行定义是没有问题的，但是这样代码很不容易被理解，如果写成：

```c++
#define SomeFun(x, a, b)
    if (x)
        x = a + b;
    else
        x = a - b;
//这样理解是好理解了,但是编译器会出错,因为它会认为#define SomeFun(x, a, b)是完整的一行,if (x)以及后面的语句与#define SomeFun(x, a, b)没有关系.这时候我们就必须使用这样的写法:
#define SomeFun(x, a, b)\
    if (x)\
        x = a + b;\
    else\
        x = a - b; //最后一行不加续行符
```

### `#` 和 `##`

**单个 `#`：**
在 C 语言的宏中，`#` 的功能是将其后面的宏参数进行字符串化操作，简单说就是对他所引用的宏变量通过替换后再其左右各加上一个双引用。  

例子：
```c++
# define WARNIF(EXP) \
do{ \
    if (EXP) \
    {    \ 
        fprintf(stderr, "warning:" #EXP "\n"); \
    }   \  
}while(0)    
// 在实际使用中会出现下面所示的替换过程：
// WARN_IF(div == 0);被替换成以下代码
do{
    if (div == 0) 
    {        
        fprintf(stderr, "warning:" "div == 0" "\n"); 
    }       
}while(0)
```

**两个 `##`:**
`##` 被称为连接符，用来将两个 Token 链接成一个 Token. 注意这里的连接的对象是 Token 就行，而不一定是宏的变量。比如你要做一个菜单项命令名和函数指针组成的结构体的数组，并且希望在函数和菜单项命令名直接有直观的名字上的关系，那么下面的代码就非常实用

```c++ 
struct command
{
    char *name；
    void (*function)(void);
};
 
#define COMMAND(NAME){NAME, NAME##_command}
//然后就用一些预定义好的命令来方便的初始化一个command结构的数据了：
struct command commands[] = {
    COMMAND(quit),
    COMMAND(help),
    ...
}
//COMMAND宏在这里充当一个代码生成器的作用，这样可以在一定程度上减少代码的密度， 间接的也可以减少粗心所造成的错误。
//我们还可以n个##符号链接n+1个Token，这个特性也是#符号所不具备的。
//例如： 
#define LINK_MULTIPLE(a, b, c, d) a##_##b##_##c##_###d
typedef struct_record_type LINK_MULTIPLE(name, company, position, salary);
//展开内容为 ： 
typedef struct_record_type name_company_position_salary;
```


### `...` 变参宏
`...` 在 C 语言中被称为**变参宏**   

```c++
#define myprintf (templt, ...)  fprintf (stderr, templt, ##__VA_ARGS__)  
```

这里 `##` 这个连接符充当的作用就是当 `__VA_ARGS__` 为空的时候，消除前面的那个逗号。
# 一、变量和基本类型
## 1 对象

对象：具有某种数据类型的内存空间

* 基本上，当我们编写了一个类并且到了我们实际开始使用该类的时候，就需要实例化它 (除非它是完全静态的类)
* 实例化类有两种选择，这两种选择的区别是内存来自哪里，我们的对象实际上会创建在哪里。
* 应用程序会把内存分为两个主要部分：堆和栈。还有其他部分，比如源代码部分，此时它是机器码。

### 栈分配

```c++
// 栈中创建
Entity entity;
Entity entity("lk");
```

*   什么时候栈分配？几乎任何时候，因为在 C++ 中这是初始化对象最快的方式和最受管控的方式。
*   什么时候不栈分配？ 如果创建的**对象太大**，或是需要显示地控制对象的**生存期**，那就需要堆上创建 。  

### 堆分配

```c++
// 堆中创建
Entity* entity = new Entity("lk");
delete entity； //清除
```

* 当我们调用 new Entity 时，实际发生的就是我们在堆上分配了内存，我们调用了构造函数，然后这个 **new Entity 实际上会返回一个 Entity 指针，它返回了这个 entity 在堆上被分配的内存地址**，这就是为什么我们要声明成 Entity * 类型。
* 如果你使用了 new 关键字，那你就要用 delete 来进行清理。  

### new操作符

*   new 的主要目的是分配内存，具体来说就是在堆上分配内存。
*   如果你用 new 和 **`[]` 来分配数组，那么也用  `delete[]` 。
*   new 主要就是找到一个满足我们需求的足够大的内存块，然后**返回一个指向那个内存地址的指针**。

```c++
  int* a = new int; //这就是一个在堆上分配的4字节的整数,这个a存储的就是他的内存地址.
  int* b = new int[50];//在堆上需要200字节的内存。
  delete a;
  delete[] b;
  
  //在堆上分配Entity类
  Entity* e = new Entity();
  Entity* e = new Entity;//或者这我们不需要使用括号，因为他有默认构造函数。
  Entity* e0 = new Entity[50]; //如果我们想要一个Entity数组，我们可以这样加上方括号,在这个数组里，你会在内存中得到50个连续的Entity
  delete e;
  delete[] e0;
```

*   在 new 类时，该关键字做了两件事
- 分配内存 
- 调用构造函数

```c++
Entity* e = new Entity();//1.分配内存 2.调用构造函数
Entity* e = (Entity*)malloc(sizeof(Entity);//仅仅只是分配内存**然后给我们一个指向那个内存的指针

//这两行代码之间仅有的区别就是第一行代码new调用了Entity的构造函数
delete e;//new了，必须要手动清除
```

*   new 是一个**操作符**，就像加、减、等于一样。它是一个操作符，这意味着你可以重载这个操作符，并改变它的行为。
*   通常调用 new 会调用隐藏在里面的 C 函数 malloc，但是 **malloc 仅仅只是分配内存**然后给我们一个指向那个内存的指针，而 **new 不但分配内存，还会调用构造函数**。同样，**delete 则会调用 destructor 析构函数。**
*   new 支持一种叫 **placement new** 的用法，这决定了他的内存来自哪里, 所以你并没有真正的分配内存。在这种情况下，你只需要调用构造函数，并在一个特定的内存地址中初始化你的 Entity，可以通过 new() 然后指定内存地址，例如：

```c++
int* b = new int[50]; 
Entity* entity = new(b) Entity(); 
```

## 2 变量
### 初始化 
初始化不是赋值，初始化的含义是创建变量时赋予其一个初始值，而赋值的含义是把对象的当前值擦除，而以一个新值来替代。

使用等号初始化变量——拷贝初始化
不使用等号——直接初始化
```c++
//下列四种初始化方式可以等价使用，但不是任何情况下！
int a = 0;
int a(0);

int a = {0};
int a{0};
```

**初始化方式<font color="#ff0000">不可以</font>等价使用的情况：**
1. 初始值只有一个时，使用直接初始化 或拷贝初始化都行。如果初始化多个值，一般只使用直接初始化。如果非要用拷贝初始化的方式，就要显式创建一个临时对象用于拷贝。
```c++
string str = string(10,'c'); 
```

2. 如果提供的是一个【C++11】**类内初始值**，则只能使用拷贝初始化或使用花括号的形式初始化，不能使用圆括号。

3. 如果提供的是初始元素值的列表，则只能把初始值都放在花括号里进行列表初始化，而不能放在圆括号里。
```c++
vector<string> v1{“a”, "an", "the"}; //正确的列表初始化
vector<string> v1(“a”, "an", "the"); //错误
```

#### 列表初始化
【C++11】**使用花括号进行列表初始化**
当用于内置类型的变量时，花括号初始化形式有一个重要特点：如果我们使用列表初始化且初始值存在丢失信息的风险，则编译器将报错
```c++
long double ld = 3.1415926536;
int a{ld},b={ld};  //报错，因为存在丢失信息的风险（long double->int）
int a(ld),b=ld;  //正确，可以进行转换，但确实丢失了部分值
```

####  默认初始化
内置变量未被初始化，它的值由定义的位置决定。定义于任何函数体之外的变量被初始化为0，**定义在函数体内部的内置变量将不被初始化，其值是未定义的（如果试图拷贝或访问会引发错误）。**

类的对象未被显式初始化，其值由类确定。大多数类支持无须显示初始化而定义对象，并为其提供一个合适的默认值，一些类要求每个对象都显式初始化。

### 声明和定义
C++支持分离式编译机制，该机制允许将程序分割为若干个文件，每个文件可被独立编译。为了支持分离式编译，C++将声明和定义区分开来，声明使得名字未程序所知，一个文件如果想用别处定义的名字就必须包含对这个名字的声明，而定义负责创建与名字关联的实体。

**变量、函数能且只能被定义一次，但是可以多次被声明。**

声明：这个符号、这个函数是存在的。  
定义：这个函数到底是什么。

声明和定义都规定了变量的类型和名字，此外定义还申请存储空间，还可以为变量赋予初始值。
**如果想声明一个变量而非定义它，就在变量名前加关键字 `extern`，且不要显式初始化变量：**

```c++
extern int a； //只声明不定义
int b; //声明并定义

extern int a = 1; //如果包含了显示初始化，那么声明就会变成定义，在函数体内部会报错
```

函数声明（又称函数原型）建议放在头文件中
#### extern 关键字

在 `a.cpp` 文件中使用在 `b.cpp` 中定义的变量/函数。
非 const 变量默认为 extern。

默认情况下，const 对象被设定为尽在文件内有效，多个文件中出现了同名的 const 变量时，等同于在不同文件中分别定义了独立的变量。
**有些情况下我们想要只在一个文件中定义 const，而在其他多个文件中声明并使用它，要使 const 变量能够在其他的文件中访问，必须地指定它为 extern。
## 3 类型
###  指针

对计算机来说内存就是一切。x86 即 32 位地址为 32 位（8 位 16 进制数，4 * 8 = 32），那么对应指针就是 4 字节；x64 则为 64 位地址（16 位 16 进制数），则对应指针 8 字节。

**指针是一个整数，一种存储内存地址的数字**。 就是这样。

```c++
int main()
{
	int var = 8;
	//指针就是一个整数，值为内存地址
	int* ptr = &var;	//&取地址：返回变量var的内存地址
	std::cout << *ptr;	//*解引用：返回指针ptr指向内存所存储的值，即8
}
```

**指针的类型要和他所指向的对象严格匹配，有两种例外：**
1. 允许令一个指向常量的指针指向一个非常量对象
```c++
double dval = 3.14; //dval不是常量
const double *ctpr = &dval; //正确，但是不能通过cptr改变dval的值
```
2. 存在继承关系的类。我们可以将基类的指针或引用绑定到派生类对象上，比如我们可以将基类引用 parent&指向一个子类对象 son，也可以把一个 son 对象的地址赋值给 parent*（父类装子类）。
    - 原因：当使用基类引用或指针时，实际上我们并不清楚该引用（或指针）所绑定对象的真是类型。该对象可能是基类的对象，也可能是派生类的对象。**智能指针和内置指针都支持派生类向基类的类型转换。**

> [!command] 建议：初始化所有指针
> 使用未经初始化的指针是引发运行时错误的一大原因。
> 
> 和其他变量一样，访问未经初始化的指针所引发的后果也是无法预计的。通常这一行为将造成程序崩溃，而且一旦崩溃，要想定位到出错位置将是特别棘手的问题。
> 
> 在大多数编译器环境下，如果使用了未经初始化的指针，则该指针所占内存空间的当前内容将被看作一个地址值。**访问该指针，相当于去访问一个本不存在的位置上的本不存在的对象。糟糕的是，如果指针所占内存空间中恰好有内容，而这些内容又被当作了某个地址，我们就很难分清它到底是合法的还是非法的了。**
> 
> 因此建议初始化所有的指针，并且在可能的情况下，尽量等定义了对象之后再定义指向它的指针。如果实在不清楚指针应该指向何处，就把它初始化为nullptr或者0，这样程序就能检测并知道它没有指向任何具体的对象了。

###  引用

根本上，引用通常只是指针一种形式，只是在指针上的语法糖（Syntactic sugar）。
也可以理解成变量的别名
当我们使用术语“引用”时，通常指的是“左值引用”
```c++
int main()
{
	int var = 8;
	int& ref = var;  //ref是var的别名
	std::cout << ref << std::endl;
}
```

- 引用必须被初始化，一旦定义了引用，就无法将其绑定到其他对象
- **引用本身不是对象**，所以不能定义引用的引用，也不能定义指向引用的指针（指针是对象，所以存在对指针的引用）
- 引用只能绑定在对象上，而不能与字面值或某个表达式的计算结果绑定在一起
- **引用的类型要与之绑定的对象严格匹配，有两种例外情况：**
    1. 在**初始化常量引用时**允许用任意表达式作为初始值，只要该表达式的结果能转换成引用的类型即可(编译器内部进行自动转换)。允许为一个常量引用绑定非常量的对象、字面值，甚至是一个一般表达式
```c++
int i = 42;
const int &r1 = i;
const int &r2 = 42;
const int &r3 = r1 * 2;
int &r4 = r1 * 2;
```

编译器内部转换的过程
```c++
double dval = 3.14；
const int &ri = dval;

//内部转换：使用一个临时对象temp进行转换
const int temp = dval; //由双精度浮点数生成一个临时的整型常量
const int &ri = temp; // 让ri绑定这个临时量
```

2. 存在继承关系的类。我们可以将基类的指针或引用绑定到派生类对象上，比如我们可以将基类引用 parent&指向一个子类对象 son，也可以把一个 son 对象的地址赋值给 parent*（父类装子类）。
    - 原因：当使用基类引用或指针时，实际上我们并不清楚该引用（或指针）所绑定对象的真是类型。该对象可能是基类的对象，也可能是派生类的对象。**智能指针和内置指针都支持派生类向基类的类型转换。**
### const限定符
#const
#### 常量引用
**指向常量的引用**：常量对象只能用常量引用绑定！
const int &

```c++
const int ci = 1024;
const int &r1 = ci; //正确,初始化常量引用时允许用任意表达式作为初始值
r1 = 42; //错误，r1是常量的引用，不允许为ci赋值，也就不能通过引用改变ci
int &r2 = ci; //错误，非常量引用不能绑定常量对象。
```


> [!comment] 建议
> 如果函数无须改变引用形参的值，最好将其声明为常量引用，否则会有很多不良后果p192


```c++
bool isShorter(const string &s1, const string &s2)
{
		return s1.size() < s2.size();
}
```

#### 常量指针
**指向常量的指针**：类似于常量引用，存放常量对象的地址必须使用常量指针。
const int*（同 int const*）  
对于常量指针，const写在类型之前和类型之后是等价的：
```c++
void f1(const Widget* pw);
void f2(Wiget const* pw);
```

**可以改变指针指向的地址, 不能再去修改指针指向的内容**

```c++
const int* a = new int;
*a = 2; //error! 不能再去修改指针指向的内容了。
a = (int*)&Age  //可以改变指针指向的地址
```

- **常量指针也没有规定所指的对象必须是一个常量**，所谓指向常量的指针仅仅要求不能通过该指针改变对象的值，没有规定那个对象的值能不能通过其他途径改变。


#### 指针常量
**指针本身是常量**
int* const  

**可以改变指针指向的内容, 不能再去修改指针指向的地址**

```c++
int* const a = new int;
*a = 2; //ok
a = (int*)&Age  //error
```

#### const int* const  
**指针和被指对象都是常量**

既不可以改变指针指向的内容, 也不能再去修改指针指向的地址
#### 顶层const、底层const
顶层const：对象本身是个常量
底层const：指针所指的对象或引用绑定的对象是一个常量

```c++
int i = 0;
const int ci = 42;  //顶层const

int *const p1 = &i;  //顶层const：p1是指针常量，不可改变指向的地址，即p1是一个常量。
const int *p2 = &ci; //底层const:p2是常量指针，可以改变指向的地址，所以p2不是常量。不可以改变指向的内容，即指向的ci是一个常量。
const int *const p3 = p2; //左边的const是底层const，右边的是顶层const

const int &r = ci;  //用于声明引用的const都是底层const

```
##### 拷贝操作的区别
顶层const和底层const在执行对象的拷贝操作时区别明显

顶层const不受影响：拷入或拷出的对象是否是常量没什么影响
```c++
i = ci;  //正确：拷贝ci的值，ci是一个顶层const
p2 = p3; //正确：p2 p3指向的类型相同，p3顶层const的部分不影响。
```

底层const有限制：执行对象的的拷贝操作时，**拷入和拷出的对象必须具有相同的底层const资格，或者两个对象的数据类型必须能够转换。** 
**非常量可以转换为常量，反之不行：**
```c++
int *p = p3;  //错误：p3包含底层const的定义，而p没有
p2 = p3;  //正确：p2和p3都是都是底层const
p2 = &i;  //正确：int*能转换成const int*;
int &i = ci;  //错误，普通的int&不能绑定const int
const int &r2 = i;  //正确：const int&可以绑定到一个普通int上
```

#### const形参和实参
和其他初始化过程一样，**当用实参初始化形参时会忽略掉顶层const**。即当形参有const时，传给它常量或非常量对象都是可以的。
```c++
void fcn(const int i) { };  //因为忽略了const，所以这句等价于fcn(int i)
void fcn(int i) { };  //错误: 重新定义了fcn(int)
```

#### 常量表达式
**常量表达式（const expression）是指值不会改变并且在编译过程就能得到计算结果的表达式。**

哪些是常量表达式？
> [!NOTE] 字面值常量
>
> - 整型和浮点型字面值：12、3.14等
> - 字符和字符串字面值：'a'、"hello world!"等
> - 转义序列：转义序列均以反斜线作为开始，如换行符\n等
> - 指定字面值的类型：1E、3.15L等
> - 布尔字面值：true、bool
> - 指针字面值：nullptr
>


- 用常量表达式初始化的 const 对象
- **一个对象是不是常量表达式由他的数据类型和初始值共同决定**
	- 数据类型必须为const
	- 初始值可以在编译过程获取
```c++
const int a = 20; //a是
const int b = a + 1； //b是
int c = 27; //c不是，因为不是const int
const int d = get_size(); //d不是，因为初始值get_size()必须在运行时才能获取，不符合在编译过程中获取。
	
```

##### 【C++11】constexpr 变量
#constexpr
将变量声明为 constexpr 类型，由编译器来验证变量的值是否是一个常量表达式。声明为 constexpr 的变量一定是一个常量，而且必须用常量表达式初始化

```c++
constexpr int a = 20;
constexpr int b = test(); //该函数必须为constexpr函数
```

**声明constexpr的数据类型必须是“字面值类型”**

> [!NOTE] 字面值类型
> - 算术类型（包含字符、整形数、布尔值、浮点数）
> - 引用和指针（初始值必须是nullptr或0或存储与某个固定地址中的对象）
> -  [[#字面值常量类]]
> - 枚举类型

##### 【C++11】constexpr函数
不能使用普通函数作为`constexpr`变量的初始值，允许定义一种特殊的`constexpr`函数，**这种函数可以在编译时计算器结果**。

> [!info] 定义遵循的约定
> - 函数的返回类型及所有形参的类型都得是字面值类型
> - 函数体中必须有且只有一条return语句

```c++
constexpr int new_func()
{
  return 12;
}

constexpr int c = new_func(); //正确，c是常量表达式
```

执行初始化任务时，编译器把对`constexpr`函数的调用替换成结果值，并且`constexpr`函数被隐式指定为内联函数，得以在编译过程中展开。

- `constexpr`函数体内可以包含其他语句，只要这些语句在运行时不执行任何操作就行，例如空语句、类型别名以及using声明。
- 允许`constexpr`函数返回值并非一个常量


> [!bug] 把内联函数和constexpr函数放在头文件内
> 和其他函数不一样，内联函数和constexpr函数可以在程序中多次定义，不过对于某个给定的内联函数或者constexpr函数来说，它的多个定义必须完全一致，因此通常定义在头文件中。

##### constexpr指针
限定符`constexpr`只对指针有效，与指针所指的对象无关
```c++
const int *a = nullptr;  //指向整型常量的指针，底层const
constexpr int *b = nullptr; //指针本身是constexpr，顶层const
```

#### const成员函数
#const成员函数
**`const` 的第三种用法**，他和变量没有关系，而是用在方法名的后面 ( **只有类才有这样的写法** )  
本质上是将`this`指针类型改变为常量指针，因此**不能修改类的成员变量**。

```c++
class Entity
{
private:
    int m_x,m_y;
public:
    int Getx() const  //const的第三种用法，他和变量没有关系，而是用在方法名的后面 
    {
        return m_x; //不能修改类的成员变量
        m_x = 2; //ERROR!
    }
    
    void Getx(int a) {
        m_x = a; //ok
    }
};

void PrintEntity(const Entity& e)  //const Entity调用const函数 {
    std::cout << e.Getx() << std::endl;
}

int main() {
    Entity e;
}
```

然后有时我们就会写两个` Getx` 版本，一个有 `const` 一个没有，然后上面面这个传 `const Enity &` 的方法就会调用 `const` 的 `GetX` 版本。

所以，我们把成员方法标记为 const 是因为**如果我们真的有一些 const Entity 对象，我们可以调用 const 方法**。如果没有 const 方法，那 const Entity & 对象就掉用不了该方法。

*   如果实际上没有修改类或者它们不应该修改类，**总是**标记你的方法为 const，否则在有常量引用或类似的情况下就用不了你的方法。


#### mutable关键字
#mutable
**可变数据成员（mutable data member）** 永远不会是const，即使它是const对象的成员。因此，**一个const成员函数可以改变一个可变数据成员的值。**

把类成员标记为 mutable，意味着类中的 const 方法可以修改这个成员。

```c++
class Entity
  {
  private:
      int m_x,m_y;
      mutable var;
  public:
      int Getx() const 
      {   
          var = 2; //ok mutable var
          return m_x; //不能修改类的成员变量
          m_x = 2; //ERROR!
      }
  };
```

### inline内联函数
总结：加上inline的函数在调用时和普通函数不同，相当于把这段函数展开后复制到执行语句前，减少了时间上的开销，增大了空间开销。
  
作为特别注重程序执行效率，适合编写底层系统软件的高级程序设计语言，[C++](http://c.biancheng.net/cplus/) 用 inline 关键字较好地解决了函数调用开销的问题。

在 C++ 中，可以在定义函数时，在返回值类型前面加上 inline 关键字。如：
```c++
inline int Max (int a, int b)
{
    if(a >b)
        return a;
    return b;
}
```
增加了 inline 关键字的函数称为“**内联函数**”。

**内联函数和普通函数的区别在于：当编译器处理调用内联函数的语句时，不会将该语句编译成函数调用的指令，而是直接将整个函数体的代码插人调用语句处，就像整个函数体在调用处被重写了一遍一样。**  
  
**有了内联函数，就能像调用一个函数那样方便地重复使用一段代码，而不需要付出执行函数调用的额外开销**。很显然，使用内联函数会使最终可执行程序的体积增加。以时间换取空间，或增加空间消耗来节省时间，这是计算机学科中常用的方法。  
  
内联函数中的代码应该只是很简单、执行很快的几条语句。如果一个函数较为复杂，它执行的时间可能上万倍于函数调用的额外开销，那么将其作为内联函数处理的结果是付出让代码体积增加不少的代价，却只使速度提高了万分之一，这显然是不划算的。  
  
有时函数看上去很简单，例如只有一个包含一两条语句的循环，但该循环的执行次数可能很多，要消耗大量时间，那么这种情况也不适合将其实现为内联函数。  
  
**另外，需要注意的是，调用内联函数的语句前必须已经出现内联函数的<mark style="background: #FF5582A6;">定义</mark>（即整个函数体），而不能只出现内联函数的声明。**

###  【C++11】using类型别名
```c++
typedef int zhengxing; //zhengxing是int的类型别名
using zhengxing = int; //等价
```

### 【C++11】auto类型说明符
**让编译器通过初始值推算变量类型，所以auto定义的变量必须有初始值**

**auto 的使用场景：**

在使用 iterator 的时候，如：

迭代器：
```c++
std::vector<std::string> strings;
strings.push_back("Apple");
strings.push_back("Orange");
//不使用auto
for (std::vector<std::string>::iterator it = strings.begin(); 
    it != strings.end(); it++)
{
    std::cout << *it << std::endl;
}

//使用auto
for (auto it = strings.begin(); it != strings.end(); it++) 
{
    std::cout << *it << std::endl;
}
```

当类型名过长的时候可以使用 auto

```c++
#include <iostream>
#include <string>
#include <vector>
#include <unordered_map>

class Device{};

class DeviceManager
{
private:
    std::unordered_map<std::string, std::vector<Device *>> m_Devices;
public:
    const std::unordered_map<std::string, std::vector<Device *>> &GetDevices() const
    {
        return m_Devices;
    }
};

int main() 
{
    DeviceManager dm;
    
    //不使用auto
    const std::unordered_map<std::string, std::vector<Device *>> &devices = dm.GetDevices();
    
    //使用auto
    const auto& devices = dm.GetDevices(); 

    std::cin.get();
}
```

**auto 使用建议**：如果不是上面两种应用场景，请尽量不要使用 auto！能不用，就不用！

### 类型别名
除auto之外类型名过长的时候也可以使用 `using` 或 `typedef` 方法：

```c++
using DeviceMap = std::unordered_map<std::string, std::vector<Device*>>;
typedef std::unordered_map<std::string, std::vector<Device*>> DeviceMap;

const DeviceMap& devices = dm.GetDevices();
```

### 【C++11】decltype类型指示符
decl/type：declare type（声明类型）
作用：选择并返回操作数的数据类型
**从表达式的类型推断出要定义的变量的类型，但是不用该表达式的值初始化变量。**
```c++
decltype(func()) sum = x; //sum的类型就是函数func()的返回类型
```

如果decltype使用的表达式是一个变量，则返回该变量的类型
```c++
const int a = 0, &b = a; 
decltype(a) x = 0;  //x的类型是const int
decltype(b) y = x;  //y的类型是const int&,y绑定到变量x
decltype(b) z;  //错误，z是引用，必须初始化
//引用从来都作为其所指对象的同义词出现，只有在用在decltype处是一个例外。
```

如果decltype使用的表达式是不是一个变量，则返回表达式结果对应的类型。
```c++
//decltype的结果可以是引用类型
int i = 42, *p = &i, &r = i;
decltype(r + 0) b; //加法结果是int，所以b是一个（未初始化）的int

//decltype的表达式如果加上了括号，编译器会把它当成一个表达式，得到引用类型
decltype((i)) d; //错误，d是int &，必须初始化
decltype(i) e;  //正确，e是一个（未初始化的）int
```

> [!bug] 多层括号
> `decltype((i))`（注意是双层括号）的返回结果永远是引用，而`decltype(i)`的返回结果只有当`i`本身是引用时才是引用

### 枚举enum
#enum #枚举
*   枚举类型(enumeration ）将**一组整型常数**组织在一起。
*   默认情况下，编译器设置第一个 枚举变量值为 0，下一个为 1，以此类推（也可以手动给每个枚举量赋值），且 **未被初始化的枚举值的值默认将比其前面的枚举值大 1。** ）

分类：
- 【C++11】限定作用域的枚举类型，默认成员类型位int
```c++
//限定作用域的枚举类型
//关键字enum class + 枚举类型名 + {枚举成员};
enum class color{red, yellow, green};
enum struct color{red, yellow, green}; //等价
```

- 不限定作用域的枚举类型，无默认类型，分配一个足够容纳枚举值的类型
```c++
//不限定作用域的枚举类型，省略关键字class或struct
enum color{red, yellow, green};  

//未命名的、不限定作用域的枚举类型
enum {red = 1, yellow = 2, green = 3};  
```

在限定作用域的枚举类型中，枚举成员的名字遵循常规的作用域准则，并且在枚举类型的作用域外是不可访问的。
与之相反，在不限定作用域的枚举类型中，枚举成员的作用域与枚举类型本身的作用域相同：
```c++
enum color {red, yellow, green};  //不限定作用域的枚举类型
enum stoplight (red, yellow, green};  //错误：重复定义了枚举成员
enum class peppers {red, yellow, green};//正确：枚举成员被隐藏了
color eyes = green;  //正确：不限定作用域的枚举类型的枚举成员位于有效的作用域中
peppers p = green;   //错误：peppers的枚举成员不在有效的作用域中
						//color::green在有效的作用域中，但是类型错误
color hair = color::red;  //正确：允许显式地访问枚举成员
peppers p2 = peppers::red;  //正确：使用pappers的red
```

#### 【C++11】指定enum的大小
只能指定不同大小的整型！
```c++
enum example : unsigned char //将类型指定成unsigned char，枚举变量变成了8位整型，减少内存使用。  char型是大小为1字节（8位）的整型
{
 Aa, Bb = 10, Cc
};

enum intValues unsigned long long //将类型指定成unsigned longlong
{
		charTyp = 255,shortTyp = 65535,intTyp = 65535,
		longTyp = 4294967295UL,
		1ong_ongTyp = 18446744073709551615ULL
}

enum example : float  //错误！枚举量必须是一个整数，float不是整数（double也不行）。
{
 Aa, Bb = 10, Cc
};
```

#### 【C++11】枚举类型的前置声明
提前声明 enum，前置声明必须指定其成员的类型
```c++
//不限定作用域的枚举类型intValues的前置声明
enum intvalues : unsigned long long;//不限定作用域的，必须指定成员类型
enum class open modes;  //限定作用域的枚举类型可以使用默认成员类型int
```

#### 枚举的定义和初始化
可利用新的枚举类型 **example** 声明这种类型的变量 example Dd，可以在定义枚举类型时定义枚举变量：
```c++ file:三种等价实现
//普通定义+初始化
enum  example 
{
     Aa, Bb, Cc
}
example Dd = Aa;

//在声明枚举的时候直接定义了枚举变量Dd,并初始化为Aa 
enum  example 
{
     Aa, Bb, Cc
}Dd = Aa; 

//只定义Dd而不初始化
enum  example 
{
     Aa, Bb, Cc
}Dd
Dd=Aa
```

与基本变量类型不同的地方是，**在不进行强制转换的前提下**，只能将定义的**枚举量**赋值给该种枚举的变量 (非绝对的，可用强制类型转换将其他类型值赋给**枚举变量**)

```c++
Dd = Bb; //ok
Dd = Cc; //ok

Dd = 5; //Error!因为5不是枚举量
```

枚举量可赋给非枚举变量

```c++
int a = Aa; //ok.枚举量是符号常量，赋值时编译器会自动把枚举量转换为int类型。
```

对于枚举，**只定义了赋值运算符，没有为枚举定义算术运算** ，但**能参与其他类型变量的运算**

```c++
Aa++;          //非法！
Dd = Aa + Cc   //非法！
int a = 1 + Aa //Ok,编译器会自动把枚举量转换为int类型。
```

可以通过**强制转换**将其他类型值赋给枚举变量

```c++
Dd = example(2);
//等同于
Dd = Cc
//若试图将一个超出枚举取值范围的值通过强制转换赋给枚举变量
Dd = example(10); //结果将是不确定的，这么做不会出错，但得不到想要的结果
```

#### 枚举应用
**枚举**和 **switch** 是最好的搭档：

```c++
enum enumType{
    Step0, Step1, Step2
}Step=Step0;      //注意这里在声明枚举的时候直接定义了枚举变量Step,并初始化为Step0 

switch (Step)
{
case Step0:{…;break;}

case Step1:{…;break;}

case Step2:{…;break;}

default:break;
}
```

# 二、字符串和数组
##  数组

*   C++ 数组就是表示一堆的变量组成的集合，一般是一行相同类型的变量。

### 初始化数组
```c++
int a[] = {0,1,2,}
int b[5] = {0,1,2,0,0} //多的元素初始化为默认值
```

> [!bug] 不允许拷贝和赋值
> 不能将数组的内容拷贝给其他数组作为其初始值，也不能用数组为其他数组赋值

```c++
int a[] = {0,1,2};
int b[] = a; //错误
b = a; //错误
```
### 数组的特殊性
字符数组：用字符串字面值对数组初始化，使用这种方式时注意字符串字面值的结尾处还有一个'\0'空字符，这个空字符也被拷贝到字符数组中去。
```c++
const char a[6] = “abcdef”; //错误，没有空间存放空字符，数组大小至少为7
```

直接使用数组名字时，编译器自动将其转换为指向数组首元素的指针。
```c++
string nums[] = {"one","two"."three"};
string *p2 = nums; //等价于p2 = &nums[0]
```
当使用decltype关键字时，转换不会发生：
```c++
//返回的仍是数组类型
decltype（nums） num =  {"one","two"."three"};
```

数组的下标是 `size_t` 无符号类型，定义在 cstddef 头文件中，内置的下标运算符所用的索引值是有符号类型，可以处理负数下标，只是我们用负数作为下标的情况比较少。
而标准库类型限定使用下标必须是无符号类型
### 【C++11】标准库函数begin和end
数组的指针也是迭代器。数组的begin和end函数与容器中的begin和end成员函数类似，但数组不是类，所以这里的begin和end不是成员函数，直接调用即可
```c++
int ia[] = {1,2,3,4,5};
int *beg = begin(ia); //指向ia首元素的指针
int *last = end(ia); //指向ia尾元素的下一位置的指针

cout << *beg << endl; //返回1
```

两个指针相减的结果是一种名为`ptrdiff_t`的带符号类型。

### 栈数组和堆数组
*   不能把栈上分配的数组（字符串）作为返回值，**除非**你传入的参数是一个内存地址。
*   如果你想返回的是在函数内新创建的数组，那你就要用 new 关键字来创建。
*   栈数组`int example[5];` 堆数组`int* another = new int[5];`

```c++
int main()
{
    int example[5]; //这个是创建在栈上的，它会在跳出这个作用域时被销毁
    for (int i = 0; i< 5;i++) //5个元素全部设置为2
    example[i] = 2; 
    int* another = new int[5];//这行代码和之前的是同一个意思，但是它们的生存期是不同的.因为这个是创建在堆上的,实际上它会一直存活到直到我们把它销毁或者程序结束。所以你需要用delete关键字来删除它。
    for (int i = 0; i< 5;i++) //5个元素全部设置为2
    another[i] = 2; 
    delete[] another;
    std::cin.get();
}
```

上述的两个数组在内存上看都是一样的，元素都是 5 个 2；  
那为什么要使用 new 关键字来动态分配，而不是在栈上创建它们呢？**最大的原因是因为生存期**, 因为 new 分配的内存，会一直存在，直到你手动删除它。  
如果你有个函数要**返回新创建的数组**，那么你**必须要使用 new 来分配**，**除非**你传入的参数是一个内存地址。

##  字符串
#char 
- 字符型变量用于显示**单个字符**
- 字符型变量并不是把字符本身放到内存中存储，而是将对应的ASCII编码放入到存储单元

```c++
int main() {
	
	char ch = 'a';
	//ch = "abcde"; //错误，不可以用双引号
	//ch = 'abcde'; //错误，单引号内只能引用一个字符

	cout << (int)ch << endl;  //查看字符a对应的ASCII码
	ch = 97; //可以直接用ASCII给字符型变量赋值
	cout << ch << endl;     //输出的是97对应的字符a

	return 0;
}
```

c++中的string是字符串类，const char * 是字符串常量指针。
### c_str()
虽然 C++ 提供了 string 类来替代C语言中的字符串，但是在实际编程中，有时候必须要使用C风格的字符串（例如打开文件时的路径），为此，string 类为我们提供了一个转换函数 `c_str()`，该函数能够将 string 字符串转换为C风格的字符串，并返回该字符串的 const 指针（const char*）。请看下面的代码：
```c++
//为了使用C语言中的 fopen() 函数打开文件，必须将 string 字符串转换为C风格的字符串。
string path = "D:\\demo.txt";
FILE *fp = fopen(path.c_str(), "rt");
```

###  C++ 字符串字面量

*   字符串字面量就是双引号中的内容。  
*   字符串字面量是存储在**内存**的**只读部分**的，不可对只读内存进行写操作。  
*   C++11 以后，默认为`const char*`, 否则会报错。  

```c++
char* name = "cherno";//Error!
name[2] = 'a'; 
//ERROR! 未定义行为；是因为你实际上是在用一个指针指向那个字符串字面量的内存位置，
//但字符串字面量是存储在内存的只读部分的，而你正在试图对只读内存进行写操作
--------------------------------------
const char* name = "cherno"; //Ok!
name[2] = 'a'; //ERROR!const不可修改
//如果你真的想要修改这个字符串，你只需要把类型定义为一个数组而不是指针
char name[] = "cherno"; //Ok!
name[2] = 'a'; //ok
```

*   从 C++11 开始，有些编译器比如 Clang，实际上只允许你编译`const char*`, 如果你想从一个字符串字面量编译 char, 你必须手动将他转换成`char*`

```c++
char* name = (char*)"cherno"; //Ok!
name[2] = 'a'; //OK
```

*   别的一些字符串

基本上，`char` 是一个字节的字符，`char16_t` 是两个字节的 16 个比特的字符（utf16），`char32_t` 是 32 比特 4 字节的字符（utf32），`const char` 就是 utf8. 那么 `wchar_t` 也是两个字节，和 `char16_t` 的区别是什么呢？事实上宽字符的大小，实际上是由编译器决定的，可能是一个字节也可能是两个字节也可能是 4 个字节，实际应用中通常不是 2 个就是 4 个（Windows 是 2 个字节，Linux 是 4 个字节），所以这是一个变动的值。如果要两个字节就用 char16_t，它总是 16 个比特的。

```c++
const char* name = "lk";
const wchar_t* name2 = L"lk";
const char16_t* name3 = u"lk";
const char32_t* name4 = U"lk";
const char* name5 = u8"lk";
```

**`string_literals`**

```c++
#include <iostream>
#include <string>

int main() {
    using namespace std::string_literals;

    std::string name0 = "hbh"s + " hello";

    std::cin.get();
}
```

string_literals 中定义了很多方便的东西，这里字符串字面量末尾加 s，可以看到实际上是一个操作符函数，它返回标准字符串对象（std::string）

然后我们就还能方便地这样写等等：

```c++
std::wstring name0 = L"hbh"s + L" hello";
```

string_literals 也可以忽略转义字符

```c++
#include <iostream>
#include <string>

int main() {
    using namespace std::string_literals;

    const char* example =R"(line1
    line2
    line3
    line4)"

    std::cin.get();
}
```

# 三、表达式
 
## 1 左值与右值 (lvalue and rvalue)


> [!NOTE] 简单总结
> 当一个对象被用作左值的时候，用的是对象的身份（在内存中的位置）
> 
> 当一个对象被用作右值的时候，用的是对象的值（内容）
> 
> 需要右值的地方，可以用左值来代替，此时使用的是左值的值（内容）。
> 
> 但是不能把右值当作左值
> 

1. 左值：
有地址 数值 有存储空间的值，往往长期存在； 左值是**由某种存储支持的变量**；**左值有地址和值**，可以出现在赋值运算符左边或者右边。

2. 左值引用:
左值引用仅仅接受左值，除非用了 const 兼容（ 非 const 的左值引用只接受左值 ） 所以 C++ 常用**常量**引用。**因为它们兼容临时的右值和实际存在的左值变量**

3. 右值：
是**临时量**，无地址（或者说有地址但访问不到，它只是一个临时量） 没有存储空间的短暂存在的值 。

4. 右值引用：
右值引用不能绑定到左值 可以通过常引用或者右值引用延长右值的生命周期 “有名字的右值引用是左值”

5. 右值引用的优势：**优化**

如果我们知道传入的是一个临时对象的话，那么我们就不需要担心它们是否活着，是否完整，是否拷贝。我们可以简单地偷它的资源，给到特定的对象，或者其他地方使用它们。因为我们知道它是暂时的，它不会存在很长时间 而如果如上使用 `const string& str`，虽然可以兼容右值，但是却不能从这个字符串中窃取任何东西！因为这个 str 可能会在很多函数中使用，不可乱修改！（所以才加了 const）

6. 在给函数形参列表传参时，有四种情况：

```c++
#include<iostream>
void PrintName(std::string name) // 可接受左值和右值 {
    std::cout<<name<<std::endl;
}
void PrintName(std::string& name) // 只接受左值引用，不接受右值 {
    std::cout << name << std::endl;
}
void PrintName(const std::string& name) // 接受左值和右值，把右值当作const lvalue& {
    std::cout << name << std::endl;
}
void PrintName(std::string&& name) // 接受右值引用 {
    std::cout << name << std::endl;
}
int main() {
    std::string firstName = "yang";
    std::string lastName = "dingchao";
    std::string fullName = firstName + lastName; //右边的表达式是个右值。
    PrintName(fullName);
    PrintName(firstName+lastName);
    std::cin.get();
```


## 2 递增递减运算符

> [!comment] 建议：除非必须，否则不用递增递减运算符的后置版本
> 前置版本避免了不必要的工作，它把值加1后直接返回了改变了的运算对象。
>
> 后置版本需要将原始值存储下来以便于返回这个未修改的内容，如果我们不需要修改前的值，那么后置版本的操作就是一种浪费。

```c++
++i；
i++；
```

## 3 箭头操作符->

**1. 特点：**

*   箭头运算符必须是类的成员。
*   一般将箭头运算符定义成了 const 成员，这是因为与递增和递减运算符不一样，获取一个元素并不会改变类对象的状态。

**2. 对箭头运算符返回值的限定**

箭头运算符的重载**永远不能丢掉成员访问**这个最基本的含义。当我们重载箭头时，可以改变的是箭头从哪个对象当中获取成员，而箭头获取成员这一事实则永远不变。 
**对于形如 point->mem 的表达式来说，point 必须是指向类对象的指针或者是一个重载了 operator-> 的类的对象。** 根据 point 类型的不同，point->mem 分别等价于

```c++
point->mem;

(*point).mem; //point 是一个内置的指针类型, 点运算符获取类对象的一个成员
point.operator()->mem; //point是类的一个对象
```

重载的箭头运算符**必须返回类的指针或者自定义了箭头运算符的某个类的对象。**

**3. 三种应用场景**

1) 可用于指针调用成员：p->x 等价于 (* p).x (最常见的情况)
```c++
class Entity 
{
	public:
		int x;
};

int main()
{
	Entity e;
	Entity* ptr = &e;
    
	//两种方式等价
	(*ptr).x=2;
	ptr->x = 2;
}
```

2) 重载箭头操作符
```c++
#include <iostream>
class Entity
{
private:
    int x;
public:
    void Print() 
    {
        std::cout << "Hello!" << std::endl;
    }
};

class ScopedPtr
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr): m_Ptr(ptr) { }
    ~ScopedPtr()
    {
        delete m_Ptr;
    }
    Entity* operator->()  //重载操作符
    {
        return m_Ptr;
    }
};

int main() {
    {
        ScopedPtr entity = new Entity();
        entity->Print();
    }
    std::cin.get();
}
```

进一步, 可以写为 const 版本的：

```c++
#include <iostream>
class Entity
{
private:
    int x;
public:
    void Print() const   //添加const {
        std::cout << "hello!" << std::endl;
    }
};

class ScopedPtr
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr)
        : m_Ptr(ptr)
    {
    }
    ~ScopedPtr()
    {
        delete m_Ptr;
    }
    Entity* operator->()
    {
        return m_Ptr;
    }
    const Entity* operator->() const //添加const
    {
        return m_Ptr;
    }
};

int main() {
    {
        const ScopedPtr entity = new Entity(); //如果是const，则上面代码要改为const版本的。
        entity->Print();
    }
    std::cin.get();
}
```

3) 可用于计算成员变量的 offset：

引自 B 站评论：  
因为 "指针 -> 属性" 访问属性的方法实际上是通过把指针的值和属性的偏移量相加，得到属性的内存地址进而实现访问。 而把指针设为 nullptr(0)，然后 -> 属性就等于 0 + 属性偏移量。编译器能知道你指定属性的偏移量是因为你把 nullptr 转换为类指针，而这个类的结构你已经写出来了 (float x,y,z)，float4 字节，所以它在编译的时候就知道偏移量 (0,4,8)，所以无关对象是否创建

```c++
struct vec2
{
    int x,y;
    float pos,v;
};
int main() {   
    int offset = (int)&((vec2*)nullptr)->x; // x,y,pos,v的offset分别为0,4,8,12
    std::cout<<offset<<std::endl;
    std::cin.get();
}
```

## 4 sizeof 运算符
返回一条表达式或一个类型所占的字节数，结果是 `size_t` 类型的常量表达式。 
```c++
// 两种形式：
sizeof(type);
sizeof expr; //返回表达式结果类型的大小
```

## 5 强制类型转换

### c 风格强制类型转换
直接用括号进行转换，可以实现 cast 相同功能，缺点是表现形式不清晰，不容易 debug。

### static_cast

static_cast 用于进行比较 “自然” 和低风险的转换，如整型和浮点型、字符型之间的互相转换, **不能用于指针类型的强制转换**

任何具有明确定义的类型转换，**只要不包含底层 const，都可以使用 static_cast**。

```c++
double dPi = 3.1415926;
int num = static_cast<int>(dPi);  //num的值为3
```

### reinterpret_cast

reinterpret_cast 用于进行各种**不同类型的指针**之间强制转换。

通常为运算对象的位模式提供较低层次上的重新解释。危险，不推荐。

```c++
int *ip;
char *pc = reinterpret_cast<char *>(ip);
```

### const_cast

const_cast 添加或者移除 const 性质

用于改变运算对象的**底层 const**。常用于有函数重载的上下文中。  


```c++
const string &shorterString(const string &s1, const string &s2)   
{
    return s1.size() <= s2.size() ? s1 : s2;
}

//上面函数返回的是常量string引用，当需要返回一个非常量string引用时，可以增加下面这个函数
string &shorterString(string &s1, string &s2) //函数重载 
{
    auto &r = shorterString(const_cast<const string &>(s1), 
                            const_cast<const string &>(s2));
    return const_cast<string &>(r);
}
```

### 运行时类型识别
**运行时类型识别（run-time type identification,RTTI）**由两个运算符实现：
- typeid运算符，用于返回表达式的类型
- dynamic_cast运算符，用于将基类的指针或引用安全地转换成派生类的指针或引用。

使用RTTI的例子：P733
#### dynamic_cast
1. dynamic_cast 是专门用于沿继承层次结构进行的强制类型转换。并且 **dynamic_cast 只用于多态类类型**。
2. 如果转换失败会返回 NULL，使用时需要保证是多态，即**基类**里面**含有虚函数**。
3. dynamic_cast 运算符，用于**将基类的指针或引用安全地转换成派生类的指针或引用。**

**支持运行时类型识别**。  
适用于以下情况：我们想使用基类对象的指针或引用执行某个派生类操作并且该操作不是虚函数。

> [!warning] 
> 我们应该尽量使用虚函数，使用 RTTI 运算符有潜在风险，程序员必须清楚知道转换的目标类型并且必须检查类型转换是否被成功执行。

**使用形式：**
其中，**type 必须是一个类类型**，并且通常情况下该**类型含有虚函数**。
```c++
dynamic cast<type*> (e) //e必须是一个有效的指针
dynamic cast<type&> (e) //e必须是一个左值
dynamic cast<type&&> (e) //e不能是左值
```

在上面的所有形式中，e 的类型必须符合以下三个条件中的任意一个：
 - e 的类型是目标 type 的**公有派生类**
 - e 的类型是目标 type 的**公有基类**
 - e 的类型就是**目标 type 的类型**。
如果符合，则类型转换可以成功。否则，转换失败。

##### 指针类型的dynamic_cast 
如果一条 dynamic_cast 语句的转换目标是**指针类型**并且**失败**了，则**结果为 0**。
```c++
//假定Base类至少含有一个虚函数，Derived是Base的公有派生类。
//如果有一个指向Base的指针bp，则我们可以在运行时将它转换成指向Derived的指针。
if (Derived *dp = dynamic_cast<Derived *>bp) //在条件部分执行dynamic_cast操作可以确保类型转换和结果检查在同一条表达式中完成。
{
    //成功。使用dp指向的Derived对象
}
else
{
    //失败。使用bp指向的Base对象
}
```

> [!NOTE] Title
> 可以对一个空指针执行dynamic_cast，结果是所需类型的空指针

##### 引用类型的dynamic_cast 
如果转换目标是**引用类型**并且**失败**了，则 dynamic_cast 运算符将**抛出一个 bad cast 异常**。

```c++
void f(const Base&b)
{
		try{
		    const Derived &d = dynamic cast<const Derived&>（b）；
		    //使用b引用的Derived对象
		}catch(bad cast){
			//处理类型转换失败的情况
		}
}
```

#### typeid运算符
typeid运算符允许向程序表达式提问：你的对象是什么类型？
```c++
typeid(e)  //e可以是任意表达式或类型的名字
//返回一个常量对象的引用，该对象的类型时标准库类型type_info或type_info的公有派生类型（type_info类在不同编译器上有所区别）。
```
typeid运算符可以作用于任意类型的表达式。和往常一样，顶层const被忽略，如果表达式是一个引用，则typeid返回该引用所引对象的类型。不过当typeid作用于数组或函数时，并不会执行向指针的标准类型转换。也就是说，如果我们对数组a执行typeid(a),则所得的结果是数组类型而非指针类型。
当运算对象不属于类类型或者是一个不包含任何虚函数的类时，typeid运算符指示的是运算对象的静态类型。而当运算对象是定义了至少一个虚函数的类的左值时，typeid的结果直到运行时才会求得。

通常我们使用typeid比较两条表达式的类型是否相同，或者比较一条表达式的类型是否与指定类型相同：
```c++
Derived *dp = new Derived;
Base *bp = dp;
//两个指针都指向Derived对象
//在运行时比较两个对象的类型
if (typeid(*bp)==typeid(*dp)){
//bp和dp指向同一类型的对象
}
//检查运行时类型是否是某种指定的类型
if (typeid(*bp)==typeid(Derived)){
//bp实际指向Derived对象
```

## 6 运算符优先级
**从上到下优先级递减**
![[Pasted image 20230317224005.png]]
![[Pasted image 20230317224036.png]]
# 四、控制流
## switch
执行多条件分支语句

> **注意1：switch语句中表达式类型只能是整型或者字符型**
> **注意2：case里如果没有break，那么程序会一直向下执行**
> 
> 总结：与if语句比，对于多条件判断时，switch的结构清晰，执行效率高，**缺点是switch不可以判断区间**

```c++
switch(表达式)

{

	case 结果1:
		执行语句;
		break;

	case 结果2
		执行语句;
		break;

	...

	default:
		执行语句;
		break;
}
```

```c++
int main() 
{
	//请给电影评分 
	//10 ~ 9   经典   
	// 8 ~ 7   非常好
	// 6 ~ 5   一般
	// 5分以下 烂片
	int score = 0;
	cout << "请给电影打分" << endl;
	cin >> score;

	switch (score)
	{
		case 10:
		case 9:
			cout << "经典" << endl;
			break;
			
		case 8:
			cout << "非常好" << endl;
			break;
			
		case 7:
		case 6:
			cout << "一般" << endl;
			break;
			
		default:
			cout << "烂片" << endl;
			break;
	}
	system("pause");
	return 0;
}
```

## for
```c++
//语法形式
for(init-statement;condition;expression)
{
		statement;
}
```

### 语句头多重定义
init-statement可以定义多个对象，但是只能由一条声明语句，因此，所有变量的基础类型必须相同。

```c++
//init-statement定义了两个对象：i和sz
for(decltype(v.size()) i = 0, sz = v.size(); i!=sz; ++i)
{
		v.pushback(v[i]);
}
```


### 省略语句头的某些部分
for语句头能省略掉init-statement、condition、expression中的任何一个（或全部）。

省略condition：等价于在条件部分写了一个true，所以循环体内必须有语句负责退出循环，否则将会无限循环。

省略expression：要求condition部分或者循环体必须有改变迭代变量的值。
```c++
vector<int> v;
for(int i; cin>>i; /*expression为空*/)  //condition能改变i的值
{
		v.push_back(i);
}
```

### 【C++11】范围for

```c++
//range for语法
for(declaration : expression)
		statement
//expression表示的必须是一个序列，比如用花括号括起来的初始值列表、数组、vector、string，这些类型的共同特点是拥有能返回迭代器的begin和end成员。
//declaration定义一个变量，该变量用于访问序列中的基础元素，每次迭代，该部分的变量都会初始化为expression部分的下一个元素值
```

```c++
//遍历string对象中的每个字符
string str("Hello world!");
for(auto i : str)
{
		cout<<i<<endl;
}

//对string元素执行写操作，必须把循环变量定义为引用类型
for(auto &i : str)
{
		i = 1;  //将string全部改为1
}
// 原理
for(auto beg = v.begin, end = v.end(); beg!=end; ++beg)
{
		auto &i = *beg;
		i = 1;
}
```

## do while
与while的区别在于do...while会先执行一次循环语句，再判断循环条件。
```c++
do
		statement;
while(condition);
```

## break
**终止离他最近的**while、do while、for或swtich语句，并从这些语句之后的第一条语句开始执行。

break使用的时机：
-   出现在**switch条件语句中，作用是 终止case 并跳出switch**
-   出现在**循环语句中，作用是跳出 当前 的循环语句**
-   出现在**嵌套循环中，跳出 最近的 内层循环语句**
## continue
终止最近的循环中的当前迭代，并立即开始下一次迭代。

# 五、函数 

函数就是我们写的代码块，被设计为用来执行特定的任务。**在 class 中这些代码块则被称为方法 method。**

**这里所说函数单独指类外的。**

每次调用函数，编译器生成一个 call 指令（类外的，因此没有什么动态绑定，也暂时不考虑内联）。这基本上意义着，在一个运行的程序中，为了调用一个函数，我们需要创建一个堆栈结构，这意味着我们必须把像参数这样的东西推进堆栈。我们还需要一个叫做返回地址的东西压入堆栈。然后我们要做的是跳到二进制执行文件的不同部分，以便开始执行我们的函数指令。  
为了将 push 进去的结果返回，然后我们得回去到最初调用函数之前。跳跃和执行这些都需要时间，所以它会减慢我们的程序。

而对于 main 函数，返回值是 int，并且只有 main 函数可以不 return——它会自动假设返回 0.（这是现代 C 和 C++ 的一个特性）

## 空形参列表
```c++
//形参列表可以为空，下列两种方式等价
void func() {};
void func(void) {};  //兼容c
```

## main处理命令行选项
有时我们需要给main传递实参，一种常见的情况是用户通过设置一组选项来确定函数所要执行的操作。
例如，假定main函数位于可执行文件prog内，我们可以向程序传递下面的选项：
```c++
prog -d -o ofile date0
```
这些命令通过两个（可选的）形参传递给main函数
```c++
int main(int argc, char *argv[]) {......}
int main(int argc, char **argv) {......} //等价表示，其中argv指向char*
//argc表示数组中字符串的数量
//argv是一个数组，他的元素是指向C风格字符串的指针
```
当实参传递给main函数之后，argv的第一个元素指向程序的名字或者一个空字符串，接下里的元素依次传递命令行提供的是实参，最后一个指针之后的元素值保证为0

例子中，argc的值等于5，argv应该包含如下的C风格字符串：
```c++
argv[0] = "prog"; //指向程序的名字或者一个空字符串
argv[1] = "-d";
argv[2] = "-o";
argv[3] = "ofile";
argv[4] = "date0";
argv[5] = "0";  //最后一个指针之后的元素值保证为0
```

> [!warning] 
> 当使用 argv 中的实参时，一定要记得可选的实参从 argv[1]开始；argv[0]保证是程序的名字，而非用户输入

### 深入理解argc和argv
 argc 是 argument count的缩写，表示传入main函数的参数个数；
 
 argv 是 argument vector的缩写，表示传入main函数的参数序列或指针，并且**第一个参数argv[0]一定是程序的名称，并且包含了程序所在的完整路径**，所以确切的说需要我们输入的main函数的参数个数应该是argc-1个；

简单用法示例，新建工程键代码：
```c++
#include <iostream>
 
using namespace std;
 
void main(int argc,char *argv[])
{
		for(int i=0;i<argc;i++)
		{
			cout<<"argument["<<i<<"] is: "<<argv[i]<<endl;
		}
		system("pause");
}
```
argv是指向指针的指针，main函数的第二个参数替换为 char** argv，两者是等价的。

在编译环境下按F5运行，输出如下：
![[Pasted image 20230105102141.png]]
可见，在没有参数传入的情况下，保存程序名称的第一个变量argv[0]依然存在。

传参数给main函数有两种方法，第一种方式是在编译环境中设置，以vs2012为例，右击项目—>属性—>配置属性—>调试—>命令参数，在命令参数中输入，每个参数之间用空格隔开。
![[Pasted image 20230105102227.png]]
之后点击确定并应用，运行之后显示如下：
![[Pasted image 20230105102233.jpg]]
第二种方式也是经常会用到的方式是通过命令提示符传入。首先需要打开命令提示符窗口，点击开始菜单在“搜索程序和文件”里输入命令“cmd”或者直接按**快捷键 Windows+R**，在弹出的对话框里输入“cmd”即可打开命令提示符窗口：

打开命令提示符窗口后需要输入生成的exe文件所在的完整路径，一个简便的方法是把exe文件直接拖入提示符窗口即可，之后输入传入参数，以空格分隔，之后回车，显示如下：
![[Pasted image 20230105102321.jpg]]
如果你坚持要手工输入完整路径的话，你会发现等你“Ctrl+C”路径后，在提示符窗口中按“Ctrl+V”却不能粘贴，这时候可以在窗口中右键单击一下试试，你会发现右键菜单里的粘贴功能还是有效的。

下一个例子演示使用opencv显示一幅图片：
```c++
#include <iostream>
#include <core/core.hpp>
#include <highgui/highgui.hpp>
using namespace std;
using namespace cv;
 
void main(int argc,char **argv)
{
		Mat image=imread(argv[1]);
		imshow("Lena",image);
		waitKey();
}
```
注意读入的参数是argv[1]，在命令提示符窗口运行：
![[Pasted image 20230105102410.jpg]]
最后说明一下：**一般编译器默认使用argc和argv两个名称作为main函数的参数，但这两个参数如此命名并不是必须的，你可以使用任何符合C++语言命名规范的变量名作为入参**，效果是一样的：

```c++
#include <iostream>
#include <core/core.hpp>
#include <highgui/highgui.hpp>
using namespace std;
using namespace cv;
 
void main(int value,char **point)
{
	for(int i=0;i<value;i++)
	{
		cout<<"argument["<<i<<"] is: "<<point[i]<<endl;
	}
	system("pause");
}
```


## 【C++11】 含有可变形参的函数
编写处理不同数量形参的函数，C++11有两种方法：
1. 如果所有实参类型相同，可以使用 `initialize_list` 的标准库类型 
2. 如果实参类型不同，可以编写可变参数模板P618

还有一种特殊的形参类型（省略符 `.....`），可以用它传递可变数量的形参，这种功能一般只用于与 C 函数交互得接口程序。
### initialize_list
如果函数的实参数量未知但是全部实参的类型相同，可以使用 `initialize_list` 类型的形参。`initialize_list` 是一种标准库类型，用于表示某种特定类型的值的数组。
![[Pasted image 20230823162816.png]]

`initialize_list` 对象中的元素永远是常量值
```c++
//例子：输出错误信息
void error_msg (initializer_list<string> il)
{
    for (auto beg = il.begin() ; beg != il.end ( ) ; ++beg)
        cout << *beg << " " ;
    cout cc endl;
}

//也可以同时拥有其他形参
//void error msg(ErrCode e, initializer_list<string> il)
//{
//
//}
```

如果想向 `initializer_list` 形参中传递一个值的序列, 则必**须把序列放在一对花括号内**:
```c++
// expected和actual是string对象
if (expected != actual)
    error__msg( { " functionx" , expected, actual }) ;
else
    error_msg ({"functionx" ,"okay" } ) ;
```

### 省略符形参
省略符形参是为了便于 C++程序访问某些特殊的 C 代码而设置的, 这些代码使用了名为 varargs 的 C 标准库功能。通常，省略符形参不应用于其他目的。你的 C 编译器文档会描述如何使用 varargs。

> [!bug] Title
> 省略符形参应该仅仅用于 C 和 C++通用的类型。特别应该注意的是，大多数类类型的对象在传递给省略符形参时都无法正确拷贝。

省略符形参只能出现在形参列表的最后一个位置，它的形式无外乎以下两种:
```c++
void foo (parm_list, ...);
void foo (...);
```

第一种形式指定了 foo 函数的部分形参的类型，对应于这些形参的实参将会执行正常的类型检查。省略符形参所对应的实参无须类型检查。在第一种形式中，形参声明后面的逗号是可选的。

## 【C++11】尾置返回类型

任何函数的定义都能使用尾置返回，但是**这种形式对于返回类型比较复杂的函数最有效，比如返回类型是数组的指针或者数组的引用**。

下面的例子是返回一个指向维度为10的数组指针的函数定义方法：

```c++
 int (*func(int i))[10]
```

下面逐层理解上述例子的含义：

-   func(int i)表示调用函数时，需要一个int类型的参数；
-   * func(int i)表示对调用func的结果执行解引用的操作；
-   (* func(int i))[10]表示解引用之后得到一个维度为10的数组；
-   int (* func(int i))[10]表示数组的数据类型为int；

若使用尾置返回类型，上述函数的定义可以写成：

尾置返回类型跟在形参列表后面并以一个 -> 符号开头。为了表示函数真正的返回类型跟在形参列表之后，我们在本应该出现返回类型的地方放置一个 auto：

```c++
// func接受了一个int类型的实参，返回了一个指针，该指针指向一个含有10个整数的数组
auto func(int i) -> int (*)[10];
```

因为我们把函数的返回类型放在了形参列表之后，所以我们可以很清晰地看到func函数返回的是一个指针，并且该指针指向了一个含有10个整数的数组。
## 默认实参
- 局部变量不能作为默认实参
- 调用含有默认实参的函数时，可以包含该实参，也可以省略该实参
- 设计时尽量让不怎么使用默认值的参数出现在前面
```c++
//函数具有默认实参
string screen(int height = 24, int width = 80, char background = ' ');

//函数调用时实参按位置解析，默认实参负责填补函数调用缺少的尾部实参
	string window;
	window = screen(); // 等价于screen(24, 80, ' ')
	window = screen(66); // 等价于screen(66, 80, ' ')
	window = screen(66,256); // 等价于screen(66, 256, ' ')
	window = screen(66,256,'#'); // 等价于screen(66, 256, '#')
	window = screen( , , "S") // 错误，只能从尾部填充
```

## 函数指针
#函数指针

```c++
bool lengthCompare(const string &, const string &);

// 声明一个指向该函数的指针，只需要用指针替换掉函数即可
bool (*pf)(const string &, const string &);
```

> [!NOTE] Title
> （*pf）两端的括号必不可少，如果不写括号，则pf是一个返回值为bool指针类型的函数
>
> bool *pf(const string &, const stirng &);

### 使用函数指针
当把函数名作为一个值使用时，该函数自动转换成指针
```c++
pf = lengthCompare; //pf指向lengthCompare函数
pf = &lengthCompare; //等价，&取地址符是可选的

//使用auto简写
auto pf = lengthCompare;
```

可以直接使用函数指针调用该函数，无须提前解引用
```c++
bool b = pf("hello","world");
bool b = (*pf)("hello","world"); //等价，*解引用符是可选的
bool b = lengthCompare("hello","world"); //等价，常规调用方法

//使用auto简写
auto b = pf("hello","world")
```

### 函数指针作为参数
函数指针可以作为一个参数传递给另一个函数。这时函数指针的使用就像普通的常量和变量一样。
当函数指针作为参数传递的时候，这时接收参数传递的函数通常需要根据这个指针调用这个函数。作为参数传递的函数指针通常表示 **回调函数（Callback Functions）**。

> [!NOTE] 回调函数
> 回调函数就是一个通过函数指针调用的函数。如果你把函数的指针（地址）作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，我们就说这是回调函数。

```c++
typedef int (*FuncPtr)(int, int);
using FuncPtr = int(*)(int, int);  //等价

int calculate(int a, int b, FuncPtr operation)
{
     int result;
     result = operation(a, b); // 运算
     return result;
}

```
### 为什么要首先使用函数指针

和数组类似，不能定义函数类型的形参，但是形参可以是指向函数的指针，这样就能**将一个函数作为另一个函数的形参**。

```c++
void Print(int val) {
    std::cout << val << std::endl;
}

//下面就将一个函数作为形参传入另一个函数里了
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (int temp : values) {
        function(temp); //就可以在当前函数里用其他函数了
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    ForEach(values, Print); //这里就是传入了一个函数指针进去！！！！
}
```

**优化：lambda**

**lambda 本质上是一个普通的函数**，只是它不像普通函数这样声明，它是我们的代码**在过程中生成的，用完即弃的函数**，不算一个真正的函数，是**匿名函数** 。  
格式：`[] ({形参表}) {函数内容}`

```c++
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (int temp : values) {
        function(temp);     //正常调用lambda函数
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    ForEach(values, [](int val){ std::cout << val << std::endl; });     //如此简单的事就交给lambda来解决就好了
}
```

## 【C++11】lambda表达式
lambada 即 λ 的读音

可以理解为一个临时函数，适用于那种只在一两处使用的简单操作。
如果一个需要多次使用的功能，最好还是用函数。

> [!NOTE] 可调用对象
> 对于一个对象或一个表达式，如果可以对其使用调用运算符（），则称它为可调用的。
>
可调用对象有四种：
> 1. 函数
> 2. 函数指针
> 3. 重载了函数调用运算符的类p506
> 4. lambda表达式

普通函数的不同：
1. lambda可以定义在函数内部
2. 可以忽略参数列表和返回类型，但必须包含捕获列表和函数体
3. 当我们需要为一个lamba定义返回类型时，必须使用尾置返回类型。
4. 不能有默认参数

`[捕获列表](参数列表)->返回类型 {函数体}`
`[capture list](parameter list)->return type { function body }`

```c++
//可以忽略参数列表和返回类型，但必须包含捕获列表和函数体
auto f = [] { return 42;}  //auto为int型

//调用方式和普通函数相同，注意要使用调用运算符()
cout<<f()<<endl; //输出42
```

### 指定lambda返回类型

> [!NOTE] lambda返回类型
> 如果lambda的函数体只有一个return语句，则返回类型从返回的表达式的类型推断。
> 
> 如果lambda的函数体有多个return语句，则要求所有return的表达式类型相同。
> 
> 如果lambda的函数体没有return语句，则返回类型为void。

```c++
//将序列中的负数都改为正数
int main()
{
		vector<int> v{-1,-2,3,4,-5};
		//正确：无需指定返回类型，编译器推断返回int
		transform(v.begin(), v.end(), v.begin(),
			[](int i) {return i < 0 ? -i : i; });

		for (auto value : v)
		{
			cout << value << endl; //输出1 2 3 4 5
		}
}


//用if改写，看起来等价
//正确：return的表达式类型都是int
transform(v.begin(), v.end(), v.begin (),
		  [](int i) {if(i < 0) return -i; else return i;}) 

//使用尾置返回类型，指定lambda返回类型
transform(v.begin(), v.end(), v.begin (),
		  [](int i)->int {if(i < 0) return -i; else return i;}) 


//无return，返回类型为void
//错误，lambda返回了void，而vector元素是int类型，无法完成转换
transform(v.begin(), v.end(), v.begin(),
		[](int i) {if (i < 0)  i*=-1; });
	
```
### 捕获列表
**捕获列表**是一个lambda所在函数中定义的局部变量的列表（通常为空）
![[Pasted image 20230212201442.png]]
显式捕获：[name]显示列出所在函数的变量
隐式捕获：让编译器根据lambda体中的代码推断我们要使用哪些变量
注意identifier_list是个列表：可以指定多个显示捕获变量

> [!NOTE] 捕获外部变量
> lambda只有在其捕获列表中捕获一个它所在函数中的局部变量，才能在函数体中使用该变量。
> 
> 捕获列表只用于局部非static变量，lambda可以直接使用局部static变量和在它所在函数之外声明的名字。

```c++
struct S2 
{ 
	void f(int i); 
};

void S2::f(int i)
{
    [&]{};          // OK: 隐式引用捕获列表
    [&, i]{};       // OK: i使用值捕获，其他局部变量都是引用捕获
    [&, &i] {};     // Error，i只能值捕获
   
	   [=]{};          // OK: 隐式值捕获列表
    [=, &i]{};      // OK: i使用引用捕获，其他局部变量都是值捕获
    [=, =i]{};      // Eroor: i只能使用引用捕获获
    [=, &i,this]{}; // Error：C++11暂不支持this
}


```

事例：

```c++
#include <iostream>
#include <vector>
#include <functional>
void ForEach(const std::vector<int>& values, void(*function)(int)) 
{
    for (int temp : values) 
    {
        function(temp);     //正常调用lambda函数
    }
}

int main() 
{
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    //函数指针的地方都可以用auto来简化操作，lambda亦是
    //这样子来定义lambda表达式会更加清晰明了
    auto lambda = [](int val){ std::cout << val << std::endl; }
    ForEach(values, lambda);    
}
-------------------------------------------------
//lambda可以使用外部（相对）的变量，而[]就是表示打算如何传递变量
#include <functional>   //要用捕获就必须要用C++新的函数指针！
//新的函数指针的签名有所不同！
void ForEach(const std::vector<int>& values, const std::function<void(int)>& func) 
{
    for (int temp : values)
     {
        func(temp);     
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    //注意这里的捕获必须要和C++新带的函数指针关联起来！！！
    int a = 5;  //如果lambda需要外部的a向量
    //则在捕获中写入a就好了
    auto lambda = [a](int val){ std::cout << a << std::endl; }
    ForEach(values, lambda);    
}
```

### 可变lambda
默认情况下，对一个值背靠背的变量，lambda不会改变其值。**如果我们希望能改变一个被捕获的变量的值，就必须在函数体前加上关键字 mutable。**

```c++
void fcn3()
{
		size t v1=42://局部变量
		//f可以改变它所捕获的变量的值
		auto f = [v1]() mutable { return ++v1;}
		v1=0:
		auto j=f();//j改变为43
}
```

# 六、面向对象 OOP

> [!NOTE] 面向对象程序设计（object-oriented programming）核心思想
> - 数据抽象：将类的接口与实现分离
> - 继承：定义相似的类型并对相似关系建模
> - 动态绑定(又称运行时绑定)：在运行时选择函数的版本，使用相同的代码处理不同类型的对象。
>> [!NOTE]
> 当我们使用基类的引用或（指针）调用一个虚函数时，将发生动态绑定。

## 0 类

###  类成员

**定义在类内部的成员函数是自动inline的**

#### 类成员指针
成员指针是指**可以指向类的非静态成员**的指针。
成员指针的类型囊括了类的类型以及成员的类型。当初始化一个这样的指针时，我们令其指向类的某个成员，但是不指定该成员所属的对象：直到使用成员指针时，才提供成员所属的对象。

```c++
//该类作为示例
class Screen
{
public:
    typedef std::string::size_type pos;
    char get_cursor() const { return contents[cursor]; }
    char get() const;
    char get(pos ht, pos wd) const;
private:
    std::string contents;
    pos cursor;
    pos height, width;
};
```
##### 数据成员指针
成员指针包含成员所属的类。再 `*` 之前添加`classname::`表示指针可以指向`classname`的成员：
```c++
//pdata可以指向一个常量（非常量）Screen对象的string成员
const string Screen::*pdata;
pdata = &Screen::contents;  //指向某个非特定Screen对象的contents成员

//更简单的方法，C++11 auto或decltype
auto pdata = &Screen::contents;
```

与成员访问运算符`.`和`->`类似，也有两种成员指针访问运算符：`·*` 和`->*` ，这两个运算符使得我们可以解引用指针并获得该对象的成员：
```c++
Screen myScreen;
*pScreen = &myScreen;
//*解引用pdata以获得myScreen对象的contents成员
auto s = myScreen.*pdata;
//->*解引用pdata以获得pScreen所指对象的contents成员
s = pScreen->*pdata;
```

由于访问控制规则，数据成员一般为私有的，我们通常无法直接访问content，如果想访问，可以定义一个函数，令其返回值是指向该成员的指针：
```c++
class Screen
{
public:
		//data是一个静态成员，返回一个成员指针
		static const std::string Screen::*data()
			{ return &Screen::contents; }
		//其他成员与之前的版本一致
};

//data()返回一个指向Screen类的contents成员的指针
const string Screen::*pdata = Screen::data();
//获得myScreen对象的contents成员
auto s = myScreen.*pdata;
```
##### 成员函数指针
指向类的成员函数,语法和数据成员指针一样
```c++
Screen myScreen;
*pScreen = &myScreen;
//通过pScreen所指的对象调用pmf所指的函数
char c1 (pScreen->*pmf)();
//通过myScreen对象将实参0,0传给含有两个形参的get函数
char c2 (myScreen.*pmf2)(0,0);
```

使用类型别名或typedef可以让成员指针更容易理解：
```c++
//Action是一种可以指向Screen成员函数的指针，它接受两个pos实参，返回一个char
using Action = char (Screen::*)(Screen::pos,Screen::pos)const;

Action get = &Screen::get;  //get指向Screen的get成员
```

##### 成员指针函数表
具有相同参数和返回类型的函数成员可以存入一个[[#函数表]]中。
```c++
//每个函数负责将光标向指定方向移动
class Screen
{
public:
		//其他接口和实现成员与之前一致
		Screen& home ()  //光标移动函数
		Screen& forward();
		Screen& back();
		Screen& up();
		Screen& down ()
			
		//Action是一个指针，可以用任意一个光标移动函数对其赋值
		using Action = Screen&(Screen::*)()
		//指定具体要移动的方向
		enum Directions { HOME,FORWARD,BACK,UP,DOWN )
		//定义一个move函数，使其可以调用上述任意光标函数
		Screen& move(Directions);
private:
		static Action Menu[];  //函数表
};

Screen& Screen::move (Directions cm)
{
		//运行this对象中索引值为cm的元素
		return(this->*Menu[cm])();//Menu[cm]指向一个成员函数
}

//函数表定义：
Screen::Action Screen::Menu[] = 
{
		&Screen:home,
		&Screen::forward,
		&Screen:back,
		&Screen:up,
		&Screen:down,
};

//当我们调用move函数时，给它传入一个表示光标移动方向的枚举成员：
Screen myScreen;
myScreen.move(Screen::HOME); //调用myScreen.home
myScreen.move(Screen::DOWN); //调用myScreen.down
```
##### 将成员函数用作可调用对象
和普通的函数指针不同，成员指针不是一个可调用对象，不支持函数调用运算符()。
```c++
auto fp = &string::empty; //fp指向string的empty函数
//错误，必须使用.*或->*调用成员指针
find_if(svec.begin(), svec.end(), fp); 
```

**从指向成员函数的指针获取可调用对象**
**方法一：** 是使用标准模板库[[#【C++11】function类型]]
```c++
vector<string*> pvec;
function<bool (const string*)> fp = &string::empty;
//fp接受一个指向string的指针，然后使用->*调用empty
find_if(svec.begin(), svec.end(), fp);
```

**方法二：** 使用【C++11】**mem_fn**生成一个可调用对象
标准库功能men_fn让编译器负责推断成员的类型，可以从成员指针生成一个可调用对象。和function不同的是，men_fn可以根据成员指针的类型推断可调用对象的类型，而无须用户显式指定：
```c++
find_if (svec.begin(),svec.end(),mem_fn(&string:empty));

//mem_fn生成的可调用对象可以通过对象调用，也可以通过指针调用：
auto f = mem_fn(&string::empty);  //f接受一个string或者一个string*
f (*svec.begin ()); //正确：传入一个string对象，f使用.*调用empty
f(&svec[0]);  //正确：传入一个string的指针，f使用->*调用empty
```

**方法三：** 使用 [[#【C++11】参数绑定 bind 函数]] 从成员函数生成一个可调用对象
```c++
//选择范围中的每个string,并将其bind到empty的第一个隐式实参上
auto it = find_if(svec.begin(),svec.end(),bind(&string:empty,_1));
```

和function类似的地方是，当我们使用bind时，必须将函数中用于表示执行对象的隐式形参转换成显式的。和mem_fn类似的地方是，bind生成的可调用对象的第一个实参既可以是string的指针，也可以是string的引用：
```c++
auto f = bind(&string:empty,_1);
f(*svec.begin());  //正确：实参是一个string,f使用.*调用empty
f(&svec[0]);  //正确：实参是一个string的指针，f使用->*调用empty
```

### 嵌套类
P746
嵌套类（嵌套类型）:**定义在类的内部的类**

**常用于定义作为实现部分的类。**

嵌套类是一个独立的类，与外层类基本没什么关系。特别是，外层类的对象和嵌套类的对象是相互独立的。在嵌套类的对象中不包含任何外层类定义的成员；类似的，在外层类的对象中也不包含任何嵌套类定义的成员。

### 局部类
**定义在函数内部的类**
局部类定义的类型只在定义它的作用域内可见。和嵌套类不同，局部类的成员受到严格限制。

> [!NOTE] 
> 局部类的所有成员（包括函数在内）都必须完整定义在类的内部。因此，局部类的作用与嵌套类相比相差很远。

在实际编程的过程中，因为局部类的成员必须完整定义在类的内部，所以成员函数的复杂性不可能太高。局部类的成员函数一般只有几行代码，否则我们就很难读懂它了。
类似的，**在局部类中也不允许声明静态数据成员**，因为我们没法定义这样的成员。

**局部类不能使用函数作用域中的变量：**
局部类只能访问外层作用域定义的类型名、静态变量、枚举成员。
```c++
int a, val;
void foo(int val)
{
		static int si;
		enum Loc { a = 1024,b }
		
		//Bar是foo的局部类
		struct Bar
		{
			Loc locVal;  //正确：使用一个局部类型名
			int barVal;
			void fooBar(Loc::a)  //正确：默认实参是Loc::a
			{
				barVal val;  //错误：val是foo的局部变量
				barVal = val;  //正确：使用一个全局对象
				barVal si;  //正确：使用一个静态局部对象
				locVal b;  //正确：使用一个枚举成员
			}
		}
};
```

#### 嵌套的局部类
可以在局部类的内部再嵌套一个类。此时，嵌套类的定义可以出现在局部类之外。不过，嵌套类必须定义在与局部类相同的作用域中。
```c++
void foo()
{
		class Bar
		{
		public:
			class Nested;//声明Nested类
		};
		
		//定义Nested类
		class Bar::Nested
};
```

### union联合类
**联合(union)是一种特殊的类。**
**一个union可以有多个数据成员，但是在任意时刻只有一个数据成员可以有值。** 当我们给union的某个成员赋值之后，该union的其他成员就变成未定义的状态了。分配给一个union对象的存储空间至少要能容纳它的最大的数据成员。

union不能含有引用类型的成员。

在C++11新标准中，含有构造函数或析构函数的类类型也可以作为union的成员类型。union可以为其成员指定public、protected和private等保护标记。**默认情况下，union的成员都是公有的**，这一点与struct相同。
union可以定义包括构造函数和析构函数在内的成员函数。但是**由于union既不能继承自其他类，也不能作为基类使用，所以在union中不能含有虚函数。

```c++
// Token类型的对象只有一个成员，该成员的类型可能是下列类型中的任意一种
union Token
{
    //默认情况下成员是公有的
    char cval;
    int ival;
    double dval;
};

// 显式初始化
Token first_token = {'a'};  //初始化cva1成员
Token last_token;  //未初始化的Token对象
Token *pt = new Token;  //指向一个未初始化的Token对象的指针

// 使用通用的成员访问运算符访问一个union对象的成员：
last_token.cval ='z';
pt->ival = 42;
```

#### 匿名union

> [!NOTE] Title
> 匿名union不能包含protected成员或private成员，也不能定义成员函数。

匿名union是一个**未命名的union，并且在右花括号和分号之间没有任何声明**。一旦我们定义了一个匿名union,**编译器就自动地为该union创建一个未命名的对象**：

```c++
//匿名union: 自动创建一个未命名的对象，我们可以直接访问它的成员
union
{
		char cval;
		int ival;
		double dval;
};

//在匿名union的定义所在的作用域内该union的成员都是可以直接访问的。
cval ='c';  //为刚刚定义的未命名的匿名union对象赋一个新值
iva1=42:  //该对象当前保存的值是42
```

#### 使用类管理union成员
对于union来说，要想构造或销毁类类型的成员必须执行非常复杂的操作，因此我们通常把含有类类型成员的union内嵌在另一个类当中。这个类可以管理并控制与union的类类型成员有关的状态转换。
举个例子，我们为union添加一个string成员，并将我们的union定义成匿名union,最后将它作为Token类的一个成员。此时，Token类将可以管理union的成员。
为了**追踪union中到底存储了什么类型的值**，我们通常会定义一个独立的对象，该对象称为union的**判别式(discriminant)**。我们可以使用判别式辨认union存储的值。
为了保持union与其判别式同步，我们将判别式也作为Token的成员。我们的类将定义个枚举类型的成员来追踪其union成员的状态。
在我们的类中定义的函数包括默认构造函数、拷贝控制成员以及一组赋值运算符，这些赋值运算符可以将union的某种类型的值赋给union成员：
```c++
class Token
{
public:
		//因为union含有一个string成员，所以Token必须定义拷贝控制成员
		//定义移动构造函数和移动赋值运算符的任务留待本节练习完成
		Token ()tok(INT),ival(0}
		Token (const Token &t):tok(t.tok){copyUnion(t);}
		Token &operator=(const Token&);
		//如果union含有一个string成员，则我们必须销毁它，参见19.1.2节（第729页)
		~Token(){if (tok =STR)sval.~string();
		//下面的赋值运算符负责设置union的不同成员
		Token &operator=(const std:string&);
		Token &operator=(char);
		Token &operator=(int);
		Token &operator=(double);
		
private:
		enum 
		{
			INT,CHAR,DBL,STR
		} tok;  //tok为判别式，当union存储int值时，tok的值为INT，以此类推。
		
		union  //匿名union
		{
			char cval;
			int ival;
			double dval;
			std::string sval;
		};  //每个Token对象含有一个该未命名union类型的未命名成员

		//检查判别式，然后酌情拷贝union成员
		void copyUnion(const Token&);
}:
```

更多细节P751

## 1 封装
封装的意义：
-   将属性和行为作为一个整体，表现生活中的事物
-   将属性和行为加以权限控制

### 成员访问控制
三种访问说明符，控制外部访问成员的权限

|说明符 |含义|
|-----------|-----------------------------------------------------------|
|private|声明为 private 的类成员只能由类的成员函数和友元（类或函数）使用。|
|protected|声明为 protected 的类成员可由类的成员函数和友元（类或函数）使用。 此外，它们还可由派生类使用。 |
| public    |声明为 public 的类成员可由任意函数使用。|

> [!NOTE] 建议：基类的成员访问控制：
> public：基类的接口成员
> protected：供派生类访问的成员函数
> private：供基类及基类的友元访问的成员函数

### class与struct
**class与struct的区别**
作用上：class 默认 private，struct 默认 public。  
使用上：**引入 struct 是为了让 C++ 向后兼容 C。**

推荐选用：
若只包含一些变量结构或 POD(plain old data) 时，选用 struct。例如数学中的向量类。
```c++
struct Vec2
{
		 float x, y;
		 void Add(const Vec2& other)
		 {
		     x += other.x;
		     y += other.y;
		 }
};
```
若要实现很多功能的类，则选用 class

### 友元friend
类允许其他类或者函数访问它的**非public成员**，方法是令其他类或者函数称为它的友元。

成员函数和友元函数区别：
- 成员函数有this指针，友元函数没有
- 友元不具备传递性，a的友元是b，b的友元是c，但是c不能访问a的非public成员。
- 友元不能继承

#### 全局函数做友元 
```c++
class Building
{
    //告诉编译器 goodGay全局函数 是 Building类的好朋友，可以访问类中的私有内容
    friend void goodGay(Building *building);

public:
    Building()
    {
        this->m_SittingRoom = "客厅";
        this->m_BedRoom = "卧室";
    }

public:
    string m_SittingRoom; //客厅

private:
    string m_BedRoom; //卧室
};

//复制到public上面，加上 friend
void goodGay(Building * building)
{
	cout << "好基友正在访问： " << building->m_SittingRoom << endl;
	cout << "好基友正在访问： " << building->m_BedRoom << endl;
}

int main()
{
		Building b;
		goodGay(&b);
		return 0;
}
```
#### 类做友元
```c++
class Building;
class goodGay
{
public:
		goodGay()
		{
			building = new Building();
		}
		void visit()
		{
			cout << "好基友正在访问" << building->m_SittingRoom << endl;
			cout << "好基友正在访问" << building->m_BedRoom << endl;
		}

private:
		Building *building;
};

class Building
{
		//告诉编译器 goodGay类是Building类的好朋友，可以访问到Building类中私有内容
		friend class goodGay;
		
public:
		Building()
		{
			this->m_SittingRoom = "客厅";
			this->m_BedRoom = "卧室";
		}
		
public:
		string m_SittingRoom; //客厅
private:
		string m_BedRoom;//卧室
};

int main()
{
		goodGay gg;
		gg.visit();
		return 0;
}    
```
#### 成员函数做友元
```c++
class Building;
class goodGay
{
public:

goodGay()
{
	building = new Building();
}

//让visit函数作为Building的好朋友，可以发访问Building中私有内容
void visit()
{
    cout << "好基友正在访问" << building->m_SittingRoom << endl;
    cout << "好基友正在访问" << building->m_BedRoom << endl;
} 

private:
		Building *building;
};


class Building
{
    //告诉编译器  goodGay类中的visit成员函数 是Building好朋友，可以访问私有内容
    friend void goodGay::visit();

public:
    Building()
    {
        this->m_SittingRoom = "客厅";
        this->m_BedRoom = "卧室";
    }

public:
    string m_SittingRoom; //客厅
private:
    string m_BedRoom;//卧室
};


int main()
{
		goodGay  gg;
		gg.visit();
		return 0;
}
```
## 2 继承

*   派生类拥有基类的全部成员函数和成员变量，不论是 private、protected、public。**需要注意的是：在派生类的各个成员函数中，不能访问基类的 private 成员。**

**继承的格式**

```c++
class 派生类名：public 基类名
{
};
```

例子如下，分析：

*   这个 Player 类不再仅仅只是 Player 类型，它也是 Entity 类型，就是说它**同时是这两种类型**。意思是我们可以在**任何想要用 Entity 的地方使用 Player**
*   Player 总是 Entity 的一个超集，它拥有 Entity 的所有内容。
*   因为对于 Player 来说，在 Entity 中任何**不是私有**的（private）成员，Player 都可以**访问**到

```c++
//基类，一个类被用作基类，那么它必须已定义而非仅仅声明
class Entity
{
public:
    float x,y;
 void move(float xa, float ya) {
     x += xa;
     y += ya;
 }
};

//派生类的定义
class Player : public Entity
{
public:
		 const char* Name;
//  float x,y;    //因为是派生类，所以这些是重复代码，只保留新代码即可
//  void() move(float xa, float ya)
//  {
//      x += xa;
//     y += ya;
// }
		 void printName()   //在派生类中，可以扩充新的成员变量和成员函数 
		 {
		     std::cout << Name << std::endl;
		 }
};


//如果只是声明派生类，则不需要添加派生列表
class Player; //正确
class Player : public Entity; //错误
```
###  继承方式
继承方式主要作用是影响子类的成员访问控制

**继承方式一共有三种：**
-   公共继承（public）
-   保护继承（protected）
-   私有继承（private）

![[Pasted image 20230222223037.png]]
1. 父类私有的变量子类皆不可访问
2. 公有继承，子类中对应的public protected变量不变
3. 保护继承，子类中对应的public protected变量都为protected
4. 私有继承，子类对应的变量全为私有


###  继承与静态成员
**如果基类定义了一个静态成员，则在整个继承体系中只存在该成员的唯一定义。不论从基类中派生出来多少个派生类，对于每个静态成员来说都只存在唯一的实例。**

```c++
class Base
{
public:
    static void statmem();
}

class Derived : public Base
{
    void f(const Derived&);
};
```

**静态成员遵循通用的访问控制规则，如果基类中的成员是 private 的，则派生类无权访问它。**

假设某静态成员是可访问的，则我们既能通过基类使用它也能通过派生类使用它：

```c++
void Derived::f(const Derived &derived obj)
{
    Base::statmem();  //正确：Base定义了statmem
    Derived::statmem();  //正确：Derived继承了statmem 
    
    //正确: 派生类的对象能访问基类的静态成员
    derived_obj.statmem();  //通过Derived对象访问
    statmem();  //通过this对象访问
}
```

### 【C++11】防止继承 final
#final
对于一个类，我们不想其他类继承它，或者不想考虑他是否适合作为一个基类，可以在类名后跟一个关键字`final`：
```c++
class NoDerived final{/* */}  //NoDerived不能作为基类

class Base {/* */}
class Last final : Base {/*/} //Last是final的；我们不能继承Last
```

### using改变访问权限
**使用 `using` 关键字可以改变基类成员在派生类中的访问权限**，例如将 public 改为 private、将 protected 改为 public。

注意：using 只能改变基类中 public 和 protected 成员的访问权限，**不能改变 private 成员的访问权限**。因为基类中 private 成员在派生类中是不可见的。
```c++
class Base
{
public:
		std::size_t size() const {return n};
protected:
		std::size_t n;
};

class Derived : private Base  //注意：private继承
{
protected:
		using Base::size;    //将public改为protected
private:
		using Base::n;  //将protected改为private
};
```

### 类作用域
派生类的作用域嵌套在基类的作用域之内。
#### 名字冲突

> [!NOTE] 
> - 派生类的成员将隐藏同名的基类成员，即使派生类成员和基类成员的参数列表不一致，基类成员仍会被隐藏掉。
> - 除了重写虚函数，派生类最好不要重用其他定义在基类中的名字。

和其他作用域一样，派生类也能重用定义在其直接基类或间接基类中的名字，此时定义在内层作用域（即派生类）的名字将隐藏定义在外层作用域（即基类）的名字。
```c++
struct Base
{
public:
		Base() : mem(0) { }
		int memfcn();
protected:
		int mem;
};

struct Derived Base
{
public:
		Derived(int i) : mem(i) { }  //用i初始化Derived::mem
		int memfcn(int);  //隐藏基类的memfcn()							
		int get_mem() { return mem; }  //返回Derived::mem
protected:
		int mem;  //隐藏基类中的mem
}；
```

get_mem中mem引用的解析结果是定义在Derived中的名字，下面的代码
```c++
Derived d(42);
cout << d.get_mem() << endl;  //打印42
```
的输出结果将是42。

#### 通过作用域运算符来使用隐藏的成员
```c++
struct Derived : Base
{
		int get_base_mem() { return Base::mem; } //输出0
		......
}
```
### 函数调用的解析过程
> [!Tip] 函数调用的解析过程
> 理解函数调用的解析过程对于理解C++的继承至关重要。
> 假定我们调用p->mem()(或者obj.mem()),则依次执行以下4个步骤：
> 
>1. 首先确定p(或obj)的静态类型。因为我们调用的是一个成员，所以该类型必然是类类型。
>2. 在p(或obj)的静态类型对应的类中查找mem。如果找不到，则依次在直接基类中不断查找直至到达继承链的顶端。如果找遍了该类及其基类仍然找不到，则编译器将报错。
>3. 一旦找到了mem,就进行常规的类型检查（参见6.1节，第183页）以确认对于当前找到的mem,本次调用是否合法。
>4. 假设调用合法，则编译器将根据调用的是否是虚函数而产生不同的代码：
>- 如果mem是虚函数且我们是通过引用或指针进行的调用，则编译器产生的代码将在运行时确定到底运行该虚函数的哪个版本，依据是对象的动态类型。
>- 反之，如果mem不是虚函数或者我们是通过对象（而非引用或指针）进行的调用，则编译器将产生一个常规函数调用。
> 
> 

### 派生类的拷贝控制

> [!warning] 
> 当派生类定义了拷贝或移动操作时，该操作负责拷贝或移动包括基类部分成员在内的整个对象。

- 派生类构造函数在初始化阶段不但要初始化派生类自己的成员，还负责初始化**派生类对象的基类部分**。
- 派生类的的拷贝和移动构造函数在拷贝和移动自有成员的同时，也要拷贝和移动基类部分的成员。
- 派生类赋值运算符也必须为基类部分的成员赋值
- 析构函数只负责销毁派生类自己分配的资源，派生类对象的基类部分是自动销毁的。
#### 派生类的拷贝或移动构造函数
```c++
class Base {/*...*/};

class D:public Base
{
public:
    //默认情况下，基类的默认构造函数初始化对象的基类部分
    //要想使用拷贝或移动构造函数，我们必须在构造函数初始值列表中显式地调用该构造函数
    D(const D& d) : Base(d) {/*...*/}  //拷贝基类成员
                    
    D(D&& d) : Base(std::move(d)) {/*...*/}  //移动基类成员
		
}
```

> [!warning] 
> 在默认情况下，基类默认构造函数初始化派生类对象的基类部分。如果我们想拷贝（或移动）基类部分，则必须在派生类的构造函数初始值列表中显式地使用基类的拷贝（或移动）构造函数。

#### 派生类赋值运算符
与拷贝和移动构造函数一样，派生类的赋值运算符也必须显式地为其基类部分赋值：
```c++
//Base::operator=(const Base&)  不会被自动调用
D &D::operator=(const D &rhs)
{
    Base::operator=(rhs);//为基类部分赋值
    //按照过去的方式为派生类的成员赋值
    //酌情处理自赋值及释放已有资源等情况
    return *this;
}
```
#### 派生类析构函数
在析构函数体执行完成后，对象的成员会被隐式销毁。类似的，对象的基类部分也是隐式销毁的。
因此，和构造函数及赋值运算符不同的是，派生类析构函数只负责销毁由派生类自己分配的资源：
```c++
class D:public Base
public:
    //Base::~Base被自动调用执行
    ~D() {/*该处由用户定义清除派生类成员的操作*/}
}；
```
**对象销毁的顺序正好与其创建的顺序相反：** 派生类析构函数首先执行，然后是基类的析构函数，以此类推，沿着继承体系的反方向直至最后。

### 继承的构造函数
【C++11】派生类能够重用其直接基类定义的构造函数。

派生类只继承其直接基类的构造函数，不能继承默认、拷贝、移动构造函数，如果派生类没有定义这些构造函数，编译器将自动合成。

**派生类继承基类构造函数的方式**是提供一条注明了（直接)基类名的using声明语句:
```c++
class Bulk quote : public Disc_quote
{
public:
    using Disc_quote::Disc_quote;  //继承Disc_quote的构造函数
    ......
}
```

和普通成员的using声明不同，构造函数的using声明不会改变该构造函数的访问级别。
### 容器与继承
当我们使用容器存放继承体系中的对象时，通常必须采取间接存储的方式。因为不允许在容器中保存不同类型的元素，所以我们不能把具有继承关系的多种类型的对象直接存放在容器当中。
由此，我们想在容器中存放具有继承关系的对象时，不应该在容器中直接存储对象，而应该存储（智能）指针。
```c++
//Quote是基类，Bulk_quote是派生类，派生类指针可以转换为基类指针
vector<shared_ptr<Quote>> basket;
basket.push_back (make_shared<Quote>("0-201-82470-1",50));
basket.push_back(make_shared<Bulk_quote>("0-201-54848-8",50,10,.25)）: 
//make_shared<Bulk_quote>返回一个shared_ptr<Bulk_quote>对象，当我们调用push_back时，转换成shared_ptr<Quote>
				 
//调用Quote定义的版本；打印562.5，即在15*&50中扣除掉折扣金额
cout << basket.back()->net_price(15) << endl;
```

### 多重继承
继承多个基类
```c++
//派生类的派生类列表中可以包含多个基类
class Bear : public ZooAnimal {...}
class Panda : public Bear, public Endagered {...}
```
![[Pasted image 20230228233714.png|350]]

派生类构造函数初始化所有基类
```c++
//显式地初始化所有基类
Panda::Panda(std::string name, bool onExhibit)
		: Bear (name,onExhibit,"Panda"),
		  Endangered(Endangered::critical) { }
		  
//隐式地使用Bear的默认构造函数初始化Bear子对象
Panda::Panda() : Endangered (Endangered::critical){
```
初始化顺序：
1. ZooAnima是整个继承体系的最终基类，Bear是Panda的直接基类，ZooAnimal是Bear的基类，所以首先初始化ZooAnimal。
2. 接下来初始化Panda的第一个直接基类Bear。
3. 然后初始化Panda的第二个直接基类Endangered。
4. 最后初始化Panda。

【C++11】在C++11新标准中，允许派生类从它的一个或几个基类中继承构造函数。但是如果从多个基类中继承了相同的构造函数（即形参列表完全相同），则程序将产生错误：
```c++
struct Base1
{
		Base1()=default;
		Base1(const std::string&);
		Base1(std:shared ptr<int>);
};
struct Base2
{
		Base2()=default;
		Base2(const std::string&);
		Base2(int);
};

//错误：D1试图从两个基类中都继承D1::D1(const string&)
struct D1 : public Base1,public Base2
{
		using Base1::Base1;  //从Base1继承构造函数
		using Base2::Base2;  //从Base2继承构造函数
}

//正确：如果一个类从它的多个基类中继承了相同的构造函数，则这个类必须为该构造函数定义它自己的版本：
struct D2 : public Base1,public Base2
{
		using Base1::Basel;  //从Base1继承构造函数
		using Base2::Base2;  //从Base2继承构造函数
		//D2必须自定义一个接受string的构造函数
		D2(const string &s) : Base1(s),Base2(s)  {}
		D2() = default;  //一旦D2定义了它自己的构造函数，则必须出现
}
```

### 虚继承
派生类可以通过多个直接基类多次继承同一个间接基类。比如 ZooAnimal 的子类有 Bear、Raccoon，Panda 继承于 Bear、Raccoon，这时 Panda 继承了两次 ZooAnimal。
有时候我们不想让 ZooAnimal 被重复继承，通过虚继承来实现。
![[Pasted image 20230302232604.png]]
**虚继承的目的是令某个类做出声明，承诺愿意共享它的基类**。其中，共享的基类子对象称为**虚基类(virtual base class)**。在这种机制下，**不论虚基类在继承体系中出现了多少次，在派生类中都只包含唯一一个共享的虚基类子对象。**

> [!NOTE] 
> 虚派生只影响从指定了虚基类的派生类中进一步派生出的类，它不会影响派生类本身。

```c++
//指定虚基类的方式是在派生列表中添加virtual关键字
//关键字public和virtual的顺序随意
class Raccoon : public virtual ZooAnimal {/*...*/)
class Bear : virtual public ZooAnimal {/*...*/}	

//如果某个类指定了虚基类，则该类的派生仍按常规方式进行：
class Panda : public Bear,public Raccoon,public Endangered
{};
```

**虚继承的对象的构造方式**
**含有虚基类的对象的构造顺序与一般的顺序稍有区别**：首先使用提供给最低层派生类构造函数的初始值初始化该对象的虚基类子部分，接下来按照直接基类在派生列表中出现的次序依次对其进行初始化。
例如，当我们创建一个Panda对象时：
- 首先使用Panda的构造函数初始值列表中提供的初始值构造虚基类ZooAnimal部分。
- 接下来构造Bear部分。
- 然后构造Raccoon部分。
- 然后构造第三个直接基类Endangered。
- 最后构造Panda部分。
如果Panda没有显式地初始化ZooAnimal基类，则ZooAnimal的默认构造函数将被调用。如果ZooAnima1没有默认构造函数，则代码将发生错误。

析构函数的次序与构造顺序相反
## 3 多态
多态含义是”多种形式“

多态分为两类：
-   **静态多态**: **函数重载** 和 **运算符重载**属于静态多态，**复用函数名**
-   **动态多态**: **派生类和虚函数**实现运行时多态

静态多态和动态多态**区别**：
-   静态多态的函数地址早绑定 - 编译阶段确定函数地址
-   动态多态的函数地址晚绑定 - 运行阶段确定函数地址

- **基类的指针或引用可以绑定到派生类的对象，反之不可以。**
- 当我们用一个派生类对象初始化或赋值一个基类对象时，只有该派生类对象中继承的基类部分会被拷贝、移动或赋值，它的派生类部分将被忽略掉。
### 静态类型与动态类型
**静态类型**：编译时的类型，运行前就确定了，是**变量声明时的类型**或**表达式生成的类型**
**动态类型**：运行时才确定的类型，是**变量或表达式表示的内存中的对象的类型**

```c
Quote* p = new b_Quote;  // Quote 是基类，b_Quote 是子类
```
指针 p 的静态类型是 Quote，在编译时已经确定了，但它的动态类型是 b_Quote，运行时才知道

举例
```c
Bulk_quote bulk;
Quote* pQuote = &bulk;  //pQuote静态类型为Quote，动态类型为bulk
Quote& pQuote = bulk;   //pQuote静态类型为Quote，动态类型为bulk
```

> [!NOTE] 
> 基类的指针或引用的静态类型可能与动态类型不一致

关于一个表达式的对象（变量）的静态类型和动态类型需要从两点进行分析：
1.  该对象（变量）的静态类型和动态类型是否相等；
2.  如果该对象的静态类型和动态类型不相等，其动态类型是否有作用。

对于第（1）点，其实我们可以认为每个对象（变量）都存在动态类型和静态类型，只是大多数情况下一个对象的动态类型和静态类型是相等的，所以我们平常都没有强化这两个概念，如表达式int a; 对象a的静态类型和动态类型就是相等的，所以平常我们都是直接称呼变量a的类型是int。
对于第（2）点，首先**只有当我们的表达式使用的是指针或引用时对象的静态类型和动态类型才有可能不同**，而且就算一个对象的静态类型和动态类型不相等，其还不一定能享受到动态类型所带来的方便（即动态绑定），因为我们知道，**动态绑定的前提条件是继承体系中的子类继承并重写了基类的虚函数。**  
**简单来说，一个对象（变量）的静态类型就是其声明类型，如表达式int a中的int就是对象a的声明类型，即静态类型；而一个对象（变量）的动态类型就是指程序执行过程中对象（指针或引用）实际所指对象的类型**，如Base* pB = new Drived; 其中class Drived继承于class Base，则指针对象pB的静态类型就是Base（声明类型），动态类型就是Drived（实际所指对象的类型）。

### 虚函数virtual

*   虚函数可以让我们在子类中**覆盖（又称重写)** 父类的函数，要求形参严格匹配。
*   如果虚函数使用默认实参，则基类和派生类中定义的默认实参最好一致。
*   任何构造函数之外的非静态函数都可以是虚函数
*   关键字virtual只能出现在类内部的声明语句之前，而不能用于类外部的函数定  
义。
*   如果基类把一个函数声明成虚函数，则该函数在派生类中为隐式的虚函数，即在所有派生类中都是虚函数

```c++
claee 父类名{
   //virtual + 函数
   virtual void GetName(){
       .....
   }
}
```

成员函数如果没有被声明为虚函数，则其解析过程发生在编译时而非运行时：
```c++
//基类
class Entity
{
public:
    std::string GetName() {return "Entity";} //注意这里没有用virtual
};

//派生类
class Player : public Entity
{
private: 
		 std::string m_Name; 
public: 
		 Player(const std::string& name):m_Name (name) {}  //构造函数
		 std::string GetName() {return m_Name;}
};

void printName(Entity* entity){
 std::cout << entity -> GetName() << std::endl;
}

int main(){
 Entity* e = new Entity();
 printName(e); //我们这儿做的就是调用entity的GetName函数，我们希望这个GetName作用于Entity
 Player* p = new Player("cherno"); 
 printName(p); //printName(Entity* entity)，没有报错是因为Player也是 Entity类型。同样我们希望这个GetName作用于Player
}

//输出：
Entity
Entity //不是我们想要的输出cherno，为什么会这样？
```

两次输出都是 Entity，**原因在于如果我们在类中正常声明函数或方法，当调用这个方法的时候，它总是会去调用属于这个类型的方法**，而`void printName(Entity* entity);`参数类型是`Entity*`, 意味着它会调用 Entity 内部的 GetName 函数，**它只会在 Entity 的内部寻找和调用 GetName**.

但是我们希望 C++ 能意识到, 在这里我们传入的其实是一个 Player，所以请调用 Player 的 GetName。**此时就需要使用虚函数了。**

虚函数的例子，通常有三步。
- 第一步，定义基类，声明基类函数为 `virtual` 的。  
- 第二步，定义派生类 (继承基类)，派生类实现了定义在基类的 `virtual` 函数。  
- 第三步，声明基类指针，并指向派生类，调用`virtual`函数，此时虽然是基类指针，但调用的是派生类实现的基类`virtual` 函数。


*   虚函数引入了一种要**动态分配**的东西，一般通过**虚表（vtable）** 来实现编译。虚表就是一个包含类中所有虚函数映射的列表，通过虚表我们就可以在运行时找到正确的被重写的函数。
*   简单来说，你需要知道的就是**如果你想重写一个函数，你么你必须要把基类中的原函数设置为虚函数**

### 【C++11】overide说明符
新标准允许给被重写的函数用 **"override"** 关键字标记,增强代码可读性。
**如果子类中的函数声明与父类中的函数声明不匹配，就会报错。**
```c++
//基类
class Entity
{
public:
    virtual std::string GetName() {return "Entity";} //第一步，定义基类，声明基类函数为 virtual的。
};

//派生类
class Player : public Entity
{
private: 
    std::string m_Name; 
public: 
    Player(const std::string& name):m_Name (name) {} 
    //第二步，定义派生类(继承基类)，派生类实现了定义在基类的 virtual 函数。
    std::string GetName() override {return m_Name;}  //C++11新标准允许给被重写的函数用"override"关键字标记,增强代码可读性。
};

void printName(Entity* entity){
    std::cout << entity -> GetName() << std::endl;
}

int main(){
    Entity* e = new Entity();
    printName(e); 
    //第三步，声明基类指针，并指向派生类，调用`virtual`函数，此时虽然是基类指针，但调用的是派生类实现的基类virtual函数。Entity* p = new Player("cherno");也可以
    Player* p = new Player("cherno"); 
    printName(p); 
}

//输出
Entity
cherno
```

### 纯虚函数（接口类）
#interface #纯虚函数 #接口
- 防止派生类忘记实现虚函数，**纯虚函数使得派生类必须实现基类的虚函数**。  
- 含有纯虚函数的类称为**抽象基类**，抽象基类负责定义接口，子类可以重写该接口。
- 不能创建抽象基类的对象，但可以创建子类的对象，前提是子类重写了父类的纯虚函数。

**抽象类特点**：
-   无法实例化对象
-   **类中只要有一个纯虚函数**就称为抽象类
-   **子类必须重写抽象类中的纯虚函数**，否则也属于抽象类

*   **声明方法**: 在基类中纯虚函数的方法的后面加 **=0**

```c++
virtual void funtion() = 0;
virtual std::string GetName() = 0;
```

*   在面向对象程序设计中，创建一个只包含未实现方法然后交由子类去实际实现功能的类是非常普遍的, 这通常被称为接口。**接口就是一个只包含未实现的方法并作为一个模板的类**。并且由于此**接口类**实际上不包含方法实现，所以我们**无法实例化**这个类。
*   C++ 中的纯虚函数本质上与其他语言（如 C#）中的接口相同。实际上，其他语言有 interface 关键字而不是叫 class，但 C++ 没有。接口只是 C++ 的类而已。
*   纯虚函数与虚函数的区别在于，纯虚函数的基类中的`virtual`函数，只定义了，但不实现。实现交给派生类来做。
*   **只能实例化一个实现了所有纯虚函数的类**。**纯虚函数必须被实现**，然后我们才能创建这个类的实例。
*   纯虚函数允许我们在基类中定义一个没有实现的函数，然后**强制子类**去实现该函数。


例子：

```c++
//基类
class Entity
{
public:
    virtual std::string GetName() = 0; //声明为纯虚函数。请记住，这仍然被定义为虚函数，但是=0实际上将它变成了一个纯虚函数，这意味着如果你想实例化这个类，那么这个函数必须在子类中实现
};

//派生类
class Player : public Entity
{
private: 
	 std::string m_Name; 
public: 
	 Player(const std::string& name):m_Name (name) {} 
	 std::string GetName() override {return m_Name;} //实现纯虚函数
};

void printName(Entity* entity)
{
 std::cout << entity -> GetName() << std::endl;
}

int main()
{
// Entity* e = new Entity();  //会报错，在这里我们必须给它一个实际上实现了这个函数的子类
 Entity* e = new Player("");  //ok
 printName(e); 

 Player* p = new Player("cherno"); 
 printName(p); 
}
```

再看一个更好的例子  
假设我们想编写一个打印这儿所有类名的函数。

```c++
class Printable
{   
//接口。其实就是个类。之所以称它为接口，只是因为它有一个纯虚函数，仅此而已。
public:
    virtual std::string GetClassName()= 0;
};
//基类
class Entity : public Printable
{
public:
    virtual std::string GetName() {return "Entity";}
 std::string GetClassName() override {return "Entity";} //实现接口的纯虚函数
};

//派生类
class Player : public Entity //因为Player已经是Entity（已有接口），所以Player不用去实现Printable接口
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {} 
 std::string GetName() override {return m_Name;} 
 std::string GetClassName() override {return "Player";} //实现接口的纯虚函数
};

void  Print(Printable* obj)
{  //我们需要某种类型能提供GetClassName函数，这就是接口。所有的打印都来自于这个Print函数，它接受printable对象，它不关心实际是什么类
    std::cout <<obj ->GetClassName() << std::endl;
}

int main(){
 Entity* e = new Entity();
 Player* p = new Player("cherno"); 
 Print(e);
 Print(p);
}
//输出：
Entity
Player
```

上例中，如果 Player 不是 Entity 的话，就要添加接口，如下

```c++
class Player : public OtherClass,Printable  //加逗号，添加接口Printable
{
 ....
 std::string GetName() override {return m_Name;} 
};
```

### 虚析构与纯虚析构
继承时，要养成的一个好习惯就是，**基类析构函数中，一定加上 virtual。**

当我们delete一个动态分配的对象的指针时执行析构函数，如果该指针指向继承体系中的某个类型，则有可能出现静态类型与被删除对象的动态类型不符的情况。例如：删除一个Quote* 类型的指针，此时它可能实际指向一个Bulk_quote派生类的的对象。
我们可以在基类中将析构函数定义成虚函数以确保执行正确的析构函数版本：
```c++
class Quote
{
public:
    //如果我们删除的是一个指向派生类对象的基类指针，则需要虚析构函数
    virtual ~Quote() = default;  //动态绑定析构函数
};

Quote *itemp = new Quote;  //静态类型与动态类型一致
delete itemP;  //调用Quote的析构函数
Quote *itemp = new Bulk_quote;  //静态类型与动态类型不一致
delete itemP;  //调用Bulk quote的析构函数
```

> [!NOTE] Title
>- 如果基类的析构函数不是虚函数，则delete一个指向派生类对象的基类指针将产生未定义的行为。
>- 虚析构函数将阻止编译器为这个类合成移动操作

虚析构语法：`virtual ~类名(){}`

纯虚析构语法：`virtual ~类名() = 0;`  

析构调用过程：
基类中只要定义了虚析构,在编译器角度来讲，那么由此基类派生出的所有子类地析构均为对基类的虚析构的重写，当多态发生时，用父类引用，引用子类实例时，此时的虚指针保存的子类虚表的地址，该函数指针数组中的第一元素永远留给虚析构函数指针。所以当 delete 父类引用时，即第一个调用子类虚表中的子类重写的虚析构函数此为第一阶段。然后进入第二阶段：（二阶段纯为内存释放而触发的逐级析构与虚析构就没有半毛钱关系了）而当子类发生析构时，子类内存开始释放，因内存包涵关系，触发父类析构执行，层层向上递进，至到子类所包涵的所有内存释放完成。

## 4 this指针

* C++ 中有 this 关键字，通过他我们可以访问成员函数，成员函数就是属于某个类的函数或方法。
* this 在一个 **const 函数**中，this 是一个`const Entity* const`或者是`const Entity*`, 在一个**非 const 函数**中，那么它就是一个`Entity*`类型的
* 在函数内部，我们可以引用 this，**this 是指向这个函数所属的当前对象实例的指针**

所以我们可以在 C++ 中写一个**非静态**的方法，为了去调用这个方法，我们需要先**实例化**一个对象，然后再去调用这个方法，所以这个方法必须由一个**有效对象**来调用，而 **this 关键字**就是指向那个对象的**指针**

```c++
class Entity
  {
    public:
      int x,y;
      Entity(int x, int y)
      {
          
      }
  };
```

对于上面 Entity 类中的代码，构造函数中如果过要对**成员变量 x,y** 进行赋值我可以用成员初始化的方式来进行赋值。

但是假如我不想这么做，我想要**在函数内部**做这个操作，那可能就会遇到点问题了，因为**成员变量**的名字和**传入参数**的名字**相同**，所以，如果：

```c++
Entity(int x, int y)
 {
     x = x;     
 }
```

这其实只是在用它自己给这个 x 参数进行赋值操作，这相当于啥都没干。

我其实真正要做的是引用属于**这个类的 x 和 y**，这个类的成员。而 **this 关键字**可以让我们做到这点，因为 **this 是指向当前对象的指针**。

```c++
Entity(int x, inty)
 {
     Entity* e = this;
        e->x = x;     
 }
```

更清晰一点

```c++
Entity(int x, inty)
 {
        this->x = x;  //同(*this).x = x; 
        this->y = y; 
 }
```

如果我们要写一个返回其中一个变量的函数的话，比如:

```c++
class Entity
{
public:
 int x,y;
 Entity(int x, inty)
 {
        this->x = x;   
        this->y = y; 
 }
 int GetX() const  //在函数后面加上const是很常见的，因为他不会修改这个类 {
   Entity* e = this;//ERROR!
        const Entity* e= this;//ok
     e->x = 5；//ERROR！
 }
};
```

在这个 **const 函数** (`int GetX() const`) 里，我们**不能写**`Entity* e= this`，而应该是`const Entity* e= this`。
因为**函数后面加上 const** 就意味着我们不能修改这个类，所以 **this 必须是 const** 的。
所以也不能写`e->x = 5;`如果没有这个 const 倒是可以这么写。


另一个用到的场景就是，如果我们想要调用这个 Entity 类外面的函数，他不是 Entity 的方法，但是我们想在这个类内部调用一个外部的函数，然后这个函数接受一个 Entity 类型作为参数，这时候就可以使用 **this**

```c++
class Entity;  //前置声明。
void PrintEntity(Entity* e); //在这里声明
class Entity  
{
  public:
    int x,y;
    Entity(int x, inty)
    {
        // Entity* e = this;
        this->x = x;   
        this->y = y; 
        PrintEntity(this); //我们希望能在这个类里调用PrintEntity,就可以传入this，这就会传入我已经设置了x和y的当前实例
    }
}; 
void PrintEntity(Entity* e) //在这里定义 {
    //print something
}
```

如果我想**传入一个常量引用**，我要做的就是在这里进行**解引用this**

  ```c++
  void PrintEntity(const Entity& e);
  class Entity  
  {
    public:
      int x,y;
      Entity(int x, inty)
      {
          // Entity* e = this;
            this->x = x;   
            this->y = y; 
          PrintEntity(*this); //
      }
  }; 
  void PrintEntity(const Entity& e) {
      //print something
  }
```

在**非 const 函数**里通过**解引用 this**，我们就可**赋值给 Entity&**，如果是在 **const 方法**中，我们会得到一个 **const 引用**

```c++
void PrintEntity(const Entity& e);
  class Entity  
  {
    public:
      int x,y;
      Entity(int x, inty)
      {
          // Entity* e = this;
     		this->x = x;   
     		this->y = y; 
          Entity& e = *this;  //在非const函数里通过解引用this，我们就可赋值给Entity&
          PrintEntity(*this); //解引用this
      }
      int GetX() const {
  		const Entity& e = *this; //在const方法中，我们会得到一个const引用
      }
  }; 
  void PrintEntity(const Entity& e) {
      //print something
  }
```

## 5 static静态
#static
### （面向对象的）静态成员变量
在类内成员变量的声明前加上关键字 static，该数据成员就是类内的静态数据成员。

```c++
#include <iostream>
class Myclass
{
public:
		Myclass(int a,int b,int c);
		void GetSum();
private:
		int a,b,c;
		static int Sum;//声明静态数据成员
};

int Myclass::Sum=0;    //定义并初始化静态数据成员

Myclass::Myclass(int a,int b,int c)
{
		this->a=a;
		this->b=b;
		this->c=c;
		Sum+=a+b+c;
}

void Myclass::GetSum()
{
		cout<<"Sum="<<Sum<<endl;
}

void main() {
		Myclass M(1,2,3);
		M.GetSum(); //输出6
}
```

**静态成员变量有以下特点：**

1.  类的静态成员存在于任何对象之外，对象中不包含任何与静态数据成员有关的数据，因此，每个Myclass对象将包含三个数据成员a，b，c。**只存在一个Sum对象而且被所有Myclass对象共享**。
2.  因为**静态数据成员在全局数据区分配内存，由本类的所有对象共享**，所以，它不属于特定的类对象，不占用对象的内存，而是在所有对象之外开辟内存，在没有产生类对象时其作用域就可见。因此，在没有类的实例存在时，静态成员变量就已经存在，我们就可以操作它；
3.  静态成员变量存储在全局数据区。static 成员变量的内存空间既不是在声明类时分配，也不是在创建对象时分配，而是在初始化时分配。**静态成员变量必须初始化，而且只能在类体外进行。否则，编译能通过，链接不能通过。** 语句 int Myclass::Sum=0; 是定义并初始化静态成员变量。初始化时可以赋初值，也可以不赋值。如果不赋值，那么会被默认初始化，一般是 0。静态数据区的变量都有默认的初始值，而动态数据区（堆区、栈区）的变量默认是垃圾值。
4.  static 成员变量和普通 static 变量一样，编译时在静态数据区分配内存，到程序结束时才释放。这就意味着，static 成员变量不随对象的创建而分配内存，也不随对象的销毁而释放内存。而普通成员变量在对象创建时分配内存，在对象销毁时释放内存。
5.  **静态数据成员初始化与一般数据成员初始化不同。初始化时可以不加 static，但必须要有数据类型**。被 private、protected、public 修饰的 static 成员变量都可以用这种方式初始化。静态数据成员初始化的格式为：`＜数据类型＞＜类名＞::＜静态数据成员名＞=＜值＞`
6.  类的静态成员变量访问形式 1：`＜类对象名＞.＜静态数据成员名＞`
7.  类的静态成员变量访问形式 2：`＜类类型名＞::＜静态数据成员名＞`，也即，**静态成员不需要通过对象就能访问**。
8.  静态数据成员和普通数据成员一样遵从 public,protected,private 访问规则；
9.  如果静态数据成员的访问权限允许的话（即 public 的成员），可在程序中，按上述格式来引用静态数据成员 ；
10.  sizeof 运算符不会计算 静态成员变量。

```c++
class CMyclass{
    int n;
    static int s;
};    //sizeof（CMyclass）等于4，不包括s的大小
```

**何时采用静态数据成员？**

设置静态成员（变量和函数）这种机制的目的是将某些和类紧密相关的全局变量和函数写到类里面，看上去像一个整体，易于理解和维护。**如果想在同类的多个对象之间实现数据共享，又不要用全局变量，那么就可以使用静态成员变量。** 也即，静态数据成员主要用在各个对象都有相同的某项属性的时候。比如对于一个存款类，每个实例的利息都是相同的。所以，应该把利息设为存款类的静态数据成员。这有两个好处：

1.  不管定义多少个存款类对象，利息数据成员都共享分配在全局数据区的内存，节省存储空间。
2.  一旦利息需要改变时，只要改变一次，则所有存款类对象的利息全改变过来了。

你也许会问，用全局变量不是也可以达到这个效果吗？
**同全局变量相比，使用静态数据成员有两个优势：**
1.  静态成员变量没有进入程序的全局命名空间，因此不存在与程序中其它全局命名冲突的可能。
2.  可以实现信息隐藏。静态成员变量可以是 private 成员，而全局变量不能。

### （面向对象的）静态成员函数

与静态成员变量类似，我们也可以声明一个静态成员函数。

**静态成员函数为类服务而不是为某一个类的具体对象服务。** 静态成员函数与静态成员变量一样，都是类的内部实现，属于类定义的一部分。普通成员函数必须具体作用于某个对象，而**静态成员函数不与任何对象绑定到一起**。

普通的成员函数一般都隐含了一个 this 指针，this 指针指向类的对象本身，因为普通成员函数总是具体地属于类的某个具体对象的。当函数被调用时，系统会把当前对象的起始地址赋给 this 指针。通常情况下，this 是缺省的。如函数 fn() 实际上是 this->fn()。

与普通函数相比，静态成员函数属于类本身，而不作用于对象，因此**它不具有 this 指针**。正因为它没有指向某一个对象，所以**它无法访问属于类对象的非静态成员变量和非静态成员函数，它只能调用其余的静态成员函数和静态成员变量**。

```c++
#include <iostream>
using namespace std;

class Student{
private:
   char *name;
   int age;
   float score;
   static int num;  	//学生人数
   static float total;  //总分
public:
   Student(char *, int, float);
   void say();
   static float getAverage();  //静态成员函数，用来获得平均成绩
};

int Student::num = 0;
float Student::total = 0;

//Student类构造函数
Student::Student(char *name, int age, float score)
{
   this->name = name;
   this->age = age;
   this->score = score;
   num++;
   total += score;
}

void Student::say()
{
   cout<<name<<"的年龄是 "<<age<<"，成绩是 "<<score<<"（当前共"<<num<<"名学生）"<<endl;
}

float Student::getAverage()
{
   return total / num;
}

int main() {
   (new Student("小明", 15, 90))->say();
   (new Student("李磊", 16, 80))->say();
   (new Student("张华", 16, 99))->say();
   (new Student("王康", 14, 60))->say();
   cout<<"平均成绩为 "<<Student::getAverage()<<endl;
   return 0;
}

运行结果：
小明的年龄是 15，成绩是 90（当前共1名学生）
李磊的年龄是 16，成绩是 80（当前共2名学生）
张华的年龄是 16，成绩是 99（当前共3名学生）
王康的年龄是 14，成绩是 60（当前共4名学生）
平均成绩为 82.25
```

**静态成员函数的特点：**

1.  出现在类体外的函数定义不能指定关键字 static；
2.  静态成员之间可以相互访问，即静态成员函数（仅）可以访问静态成员变量、静态成员函数；
3.  **非静态成员函数可以任意地访问静态成员函数和静态数据成员；**
4.  由于没有 this 指针的额外开销，静态成员函数与类的全局函数相比速度上会稍快；
5.  调用静态成员函数，两种方式：
*   通过成员访问操作符 (`.`) 和(`->`)，也即通过类对象或指向类对象的指针调用静态成员函数。
*   直接通过类来调用静态成员函数。`＜类名＞::＜静态成员函数名＞（＜参数表＞）`。也即，**静态成员不需要通过对象就能访问。**

**拷贝构造函数的问题**
在使用包含静态成员的类时，有时候会调用拷贝构造函数生成临时的隐藏的类对象，而这个临时对象在消亡时会调用析构函数有可能会对静态变量做操作（例如 total_num--），可是这些对象在生成时却没有执行构造函数中的 total_num++ 的操作。解决方案是为这个类写一个拷贝构造函数，在该拷贝构造函数中完成 total_num++ 的操作。

### （面向过程的）静态全局变量

在全局变量前，加上关键字 static，该变量就被定义成为一个静态全局变量。

```c++
//Example 1
#include <iostream.h> 

void fn();
static int n; //定义静态全局变量 

void main() {
	n=20;
	cout<<n<<endl;
	fn();
} 

void fn() {
	n++;
	cout<<n<<endl;
}
```

**静态全局变量有以下特点：**

1.  该变量在全局数据区分配内存；
2.  未经初始化的静态全局变量会被程序自动初始化为 0（自动变量的自动初始化值是随机的）；
3.  静态全局变量在声明它的整个文件都是可见的，而**在文件之外是不可见的**； 　
4.  静态变量都在全局数据区分配内存，包括后面将要提到的静态局部变量。

**定义全局变量就可以实现变量在文件中的共享，但定义静态全局变量还有以下好处：**
1.  **静态全局变量不能被其它文件所用；**
2.  **其它文件中可以定义相同名字的变量，不会发生冲突；**

将上述示例代码改为如下：

```c++
//Example 2
//File1
#include <iostream.h> 

void fn();
static int n; //定义静态全局变量 

void main() {
	n=20;
	cout<<n<<endl;
	fn();
} 

//File2 
#include <iostream.h> 
extern int n;
void fn() {
	n++;
	cout<<n<<endl;
}
```

编译并运行 Example 2，会发现上述代码可以分别通过编译，但运行时出现错误。 这就是因为**静态全局变量不能被其它文件所用，即使在其它文件中使用 extern 进行声明也不行。**

我们将

```c++
static int n; //定义静态全局变量
```

改为

```c++
int n; //定义全局变量
```

再次编译运行程序，程序可正常运行。

**因此，在一个文件中，静态全局变量和全局变量功能相同；而在两个文件中，要使用同一个变量，则只能使用全局变量而不能使用静态全局变量。**

### （面向过程的）静态局部变量

在局部变量前，加上关键字 static，该变量就被定义成为一个静态局部变量。

```c++
//Example 3
#include <iostream.h>
void fn();
void main() {
	fn(); //10
	fn(); //11
	fn(); //12
}

void fn() {
	static n=10;
	cout<<n<<endl;
	n++;
}
```

通常，在函数体内定义了一个变量，每当程序运行到该语句时都会给该局部变量分配栈内存。但随着程序退出函数体，系统就会收回栈内存，局部变量也相应失效。

但有时候我们需要在两次调用之间对变量的值进行保存。通常的想法是定义一个全局变量来实现。但这样一来，变量已经不再属于函数本身了，不再仅受函数的控制，这给程序的维护带来不便。

静态局部变量正好可以解决这个问题。静态局部变量保存在全局数据区，而不是保存在栈中，每次的值保持到下一次调用，直到下次赋新值。

**静态局部变量有以下特点：**

1.  静态局部变量在全局数据区分配内存；
2.  静态局部变量在程序执行到该对象的声明处时被首次初始化，即以后的函数调用不再进行初始化；
3.  静态局部变量一般在声明处初始化，如果没有显式初始化，会被程序自动初始化为 0；
4.  静态局部变量始终驻留在全局数据区，直到程序运行结束。但其作用域为局部作用域，当定义它的函数或语句块结束时，其作用域随之结束；

### （面向过程的）静态函数

在函数的返回类型前加上 static 关键字, 函数即被定义为静态函数。**静态函数与普通函数不同，它只能在声明它的文件当中可见，不能被其它文件使用。**

```c++
//Example 4
#include <iostream.h>
static void fn();//声明静态函数

void main() {
	fn();
}

void fn()//定义静态函数 {
	int n=10;
	cout<<n<<endl;
}
```

**定义静态函数的好处：（类似于静态全局变量）**

1.  静态函数不能被其它文件所用；
2.  其它文件中可以定义相同名字的函数，不会发生冲突；

## 6 构造函数

*   当创建对象的时候，构造函数被调用
*   构造函数最重要的**作用就是初始化类**

```c++
class Entity {
public:
  int x, y;
  Entity(){}  //默认构造函数，不带参数
  Entity(int m_x, int m_y) : x(m_x), y(m_y) {}  //类内定义构造函数，带参数，用来初始化x和y
  Entity(int x1); //声明，用于类外定义
	 
  void print() 
  {
    std::cout << x << ',' << y << std::endl;
  }
};

Entity::Entity(int x1) //类外定义构造函数
{
	this->x = x1;
	this->y = 0;
}
```

*   构造函数没有返回类型
*   构造函数不能被声明为const
*   **构造函数的命名必须和类名一样**
*   如果你不指定构造函数，你仍然有一个构造函数，这叫做默认构造函数（default constructor），是默认就有的。但是，我们仍然可以删除该默认构造函数：

```c++
class Log{
public:
    Log() = delete;  //删除默认构造函数
    ......
}
```

*   构造函数不会再你没有实例化对象的时候运行，所以如果你只是使用类的静态方法，构造函数是不会执行的。
*   当你用 new 关键字创建对象实例的时候也会调用构造函数。

**不适用默认构造函数的情况：**
- 如果类包含有内置类型或者复合类型的成员，则只有当这些成员全都被赋予了类内初始值是，这个类才适合于使用默认构造函数。
- 如果类中包含一个其他【类】类型的成员且这个成员的类型没有默认构造函数，那么编译器将无法初始化该成员，此时我们必须自定义默认构造函数。
- p449其他一些情况导致编译器无法生成默认构造函数。

**构造函数必有有初始值的情况：**
- 如果成员是const或者是引用，必须将其初始化。
- 党成员属于某种【类】类型且该类没有定义默认构造函数时，必须将这个成员初始化。

### 成员初始化顺序
**成员初始化的顺序和他们在类定义中的出现顺序一致**，第一个成员先初始化，以此类推。构造函数初始值列表的前后位置不会影响实际的初始化顺序。

> [!bug] 
> 最好令构造函数初始值顺序与成员声明的顺序保持一致，尽量避免使用某些成员初始化其他成员。
### 默认构造函数
#### 由来
先看下面代码  
![](1676131590482.png)
代码内容很简单，定义了一个包含成员 x，y 的类 Point。在序列的地方可以使用这个类  
![](1676131590589.png)
虽然我们并没有定义 Point 类的构造函数，我们依然可以定义 Point 类的 pt 对象并使用它，其原因是编译器会自动生成一个缺省的构造函数，其效果相当于  
![](1676131590652.png)
但是，一旦添加了其他有参数的构造函数，编译器就不再生成缺省的构造函数了  
![](1676131590717.png)
#### 【C++11】=default

C++11 允许我们使用`=default`来要求编译器生成一个默认构造函数：  

```c++
struct Point {
    Point() = default; //定义这个构造函数的目的是我们既需要其它形式的构造函数，也需要默认的构造函数，我们希望这个函数的作用完全等同于之前使用的默认构造函数。
    Point(int _x, int _y) : x(_x), y(_y){}
    int x;
    int y;
};

int main() {
    Point point;
    printf("%d, %d", point.x, point.y);
}
```

![](1676131590845.png)

C++11中，如果我们需要默认的行为，那么可以通过在参数列表后面写上`= default`来要求编译器生成构造函数。
`=default`既可以和声明一起出现在类的内部，也可以作为定义出现在类的外部。
和其他函数一样，如果`=default`在类的内部，那么默认构造函数是inline的；如果他在类的外部，则该成员默认不是inline的。

[[#7 拷贝控制#7.5 【C++11】=default 补充]]
#### 定义
默认构造函数：是**无参调用的构造函数（重点是无参调用）**，包括两种：
*   没有参数
*   每个参数有初始值（默认实参）

**调用时机**
如果构造函数在未指定参数或者提供了一个空初始化器列表，则会调用默认构造函数：

```c++
vector v1;
vector v2{};
```

> [!NOTE] Title
> 如果一个构造函数为所有参数都提供了默认实参，那它实际上也定义了默认构造函数

```c++
class Box 
{
public:
		Box() {}  //默认构造函数1（没有参数）
		Box(int m_x = 1, int m_y = 2) : x(m_x), y(m_y) {} //默认构造函数2（每个参数有默认实参）

		void print()
		{
			std::cout << x << ',' << y << std::endl;
		}

private:
		int x, y;
};


int main()
{
		Box b; //一个类中默认构造函数只能有一个，上面的Box类中我们写了两个默认构造函数，编译器会报错。
		b.print();
		
		//不报错，因为指定了参数，这行代码不会调用默认构造函数
		//Box b(2,3) 
		//b.print();
		return 0;
}
```

#### 说明
默认构造函数是一种**特殊的成员函数**。如果未在类中声明任何构造函数，则编译器将提供隐式的`inline默认构造函数`

```c++
#include <iostream>
using namespace std;

class Box {
public:
    int Volume() {return m_width * m_height * m_length;}
private:
    int m_width { 0 };
    int m_height { 0 };
    int m_length { 0 };
};

int main() {
    Box box1; //调用编译器生成的构造函数
    cout << "box1.Volume: " << box1.Volume() << endl; // Outputs 0
}
```

**如果声明了任何非默认构造函数、编译器不会提供默认构造函数**：

```c++
class Box {
public:
    Box(int width, int length, int height)
        : m_width(width), m_length(length), m_height(height){}
private:
    int m_width;
    int m_length;
    int m_height;

};

int main(){

    Box box1(1, 2, 3);
    Box box2{ 2, 3, 4 };
    Box box3; // C2512: no appropriate default constructor available
}
```


### 【C++11】委托构造函数
委托构造函数使用他所属类的其他构造函数执行它自己的初始化过程，或者说它把它自己的一些（或全部）职责委托给了其他构造函数。

在委托构造函数内，成员初始值列表只有一个唯一的入口，就是类名本身。
```c++
class Sales_data
{
public:
	//非委托构造函数使用对应的实参初始化成员
	Sales_data(std::string s, unsigned cnt, double price):
	bookNo(s), units_sold(cnt),revenue(cnt * price) {}

	//其余构造函数全部委托给另一个构造函数
	Sales_data() : Sales_data("",0,0) {}
	Sales_data(std::string s) : Sales_data(s, 0, 0) {}

	...
};
```

### 隐式类类型转换

隐式转换**只能进行一次**。

```c++
#include <iostream>

class Entity
{
private:
    std::string m_Name;
    int m_Age;
public:
    Entity(const std::string& name)
        : m_Name(name), m_Age(-1) {}

    Entity(int age)
        : m_Name("Unknown"), m_Age(age) {}
};

int main() {
    Entity test1("lk");
    Entity test2(23); 
    Entity test3 = "lk"; //error!只能进行一次隐式转换
    Entity test4 = std::string("lk");
    Entity test5 = 23; //发生隐式转换

    std::cin.get();
}
```

如上，在 test5 中，int 型的 23 就被隐式转换为一个 Entity 对象，这是**因为 Entity 类中有一个 Entity(int age) 构造函数，因此可以调用这个构造函数，然后把 23 作为他的唯一参数，就可以创建一个 Entity 对象。**

同时我们也能看到，对于语句`Entity test3 = "lk";`会报错，原因是**只能进行一次隐式转换**，`"lk"`是`const char`数组，这里需要先转换为`std::string`，再从 string 转换为 Entity 变量，两次隐式转换是不行的，所以会报错。但是写为`Entity test4 = std::string("lk");`就可以进行隐式转换。

最好不写`Entity test5 = 23;`这样的函数，应尽量避免隐式转换。因为`Entity test2(23);`更清晰。

### **explicit 关键字**
#explicit
explicit（显式）
将构造函数声明为explicit，**禁止构造函数进行隐式转换。**
**只能用于直接初始化，不能用于拷贝初始化（使用=进行初始化）**

```c++
#include <iostream>
  class Entity {
  private:
    std::string m_Name;
    int m_Age;
  public:
    Entity(const std::string& name)
        : m_Name(name), m_Age(-1) {}

    explicit Entity(int age)  //声明为explicit
        : m_Name("Unknown"), m_Age(age) {}
  };

  int main()
  {
    Entity test1("lk");
    Entity test2(23); 
      Entity test3 = "lk"; 
    Entity test4 = std::string("lk");
    Entity test5 = 23; //error！禁用隐式转换

    std::cin.get();
  }
```

### 聚合类
#聚合类
当一个类满足如下条件时，我们说它是聚合的：
- 所有成员都是public
- 没有定义任何构造函数
- 没有类内初始值
- 没有基类
- 没有virtual函数

```c++
//聚合类例子
class Date
{
public:
		int a; 
		string s;
}
```

聚合类使得用户可以直接访问其成员，且具有特殊的初始化语法形式。
初始值的顺序必须与生命的顺序一致
```c++
Data vall = {0, "test"};
//vall.a = 0; vall.s = "test"  //等价
```

### 字面值常量类
#字面值常量类
**数据成员都是字面值类型的聚合类是字面值常量类**

如果不是聚合类，符合下属要求也是字面值常量类
![[Pasted image 20230210170426.png]]

constexpr构造函数
```c++
class Debug
{
public:
		constexpr Debug(int a):a(_a) {}
		......
}
```
## 7 拷贝控制
综合案例P460

定义一个类时，显式或隐式地指定此类型的对象拷贝、移动、赋值和销毁，通过五钟特殊地成员函数来控制这些操作，称为**拷贝控制操作：**
- **拷贝/移动构造函数**：定义用同类型的另一个对象初始化本对象时做什么
- **拷贝/移动赋值运算符**：定义将一个对象赋予同类型的另一个对象时做什么
- **析构函数**：定义当此类型销毁时做什么

> [!NOTE] 
>如果一个类没有定义这些拷贝控制成员，编译器会自动为它定义缺失的操作，但有时这不是我们想要的，所以才要自己定义。

### 7.1 拷贝构造函数
**如果一个构造函数的第一个参数是自身类类型的引用，且任何额外参数都有默认值，则此构造函数是拷贝构造函数。**
拷贝构造函数的第一个参数必须是一个引用类型，且通常是隐式地使用

拷贝构造函数的格式：
```c++
//声明：
T(const T& var);

//定义
T(const T& var)
{
    //函数体，进行深拷贝 分配空间放副本
}

//不使用拷贝函数，禁止赋值
T(const T& var) = delete;
```

```c++
class Foo
{
	public:
		Foo ()  //默认构造函数
		Foo(const Foo&);//拷贝构造函数
		......
}
```
默认拷贝构造函数（又称合成拷贝构造函数）**通常时将每个非static成员拷贝到正在创建地对象中**，也有一些特别的会阻止拷贝该类的对象。

```c++
class Sales_data
{
public:
		//这种最简单的拷贝构造函数声明与默认拷贝构造函数等价
		Sales_data(const Sales_data&);
private:
		std::string bookNo;
		int units_sold 0;
		double revenue 0.0;
};

//原理：这个自定义的拷贝构造函数地等价于默认拷贝构造函数
Sales_data::Sales_data(const Sales_data &orig):
		bookNo (orig.bookNo),  //使用string的拷贝构造函数
		units sold(orig.units_sold),//orig.units_sold
		revenue(orig.revenue)  //拷贝orig.revenue
		{}  //空函数体
```

这样我们就理解了**直接初始化和拷贝初始化之间的区别**了：
**直接初始化**：使用构造函数，根据参数匹配
**拷贝初始化**：使用拷贝构造函数，将右侧运算对象拷贝到正在创建的对象中
```c++
string dots(10,'.') //直接初始化
string s(dots); //直接初始化
string s2 = dots; //拷贝初始化
string nu11book = "9-999-99999-9"; //拷贝初始化
string nines = string(100,9); //拷贝初始化
```


> [!warning] 拷贝构造函数调用时机
> 1. 用=定义变量时
> 2. 将一个对象作为实参**传递给一个非引用类型的形参**
> 3. 从一个返回类型为非引用类型的函数返回一个对象
> 4. 用花括号列表初始化一个数组中的元素或一个聚合类中的成员
> 5. 某些类的操作，如insert、push，容器会对元素进行拷贝初始化


#### 深拷贝与浅拷贝

**浅拷贝：简单的赋值拷贝操作
![[Pasted image 20230318234723.png]]

**深拷贝：在堆区重新申请空间，进行拷贝操作**
![[Pasted image 20230318235705.png]]

**示例：**
```c++
class Person 
{  
public:  
    //无参（默认）构造函数  
    Person() 
    {  
     cout << "无参构造函数!" << endl;  
    } 
    
    //有参构造函数  
    Person(int age ,int height) 
    {  
       
     cout << "有参构造函数!" << endl;  
    ​  
     m_age = age;  
     m_height = new int(height);  //开辟堆区空间
       
    }  
    
    //默认拷贝构造函数的等价实现
    //Person(const Person& p) 
    //{  
    //    m_age = p.m_age;
    //}
    
    //自定义拷贝构造函数，解决浅拷贝带来的问题    
    Person(const Person& p) 
    {  
     cout << "拷贝构造函数!" << endl;  
     //如果不利用深拷贝在堆区创建新内存，会导致浅拷贝带来的重复释放堆区问题  
     m_age = p.m_age;  
     m_height = new int(*p.m_height);  
       
    }  
    ​  
    //析构函数：将堆区开辟的数据做释放  
    ~Person()
    {  
     cout << "析构函数!" << endl;  
     if (m_height != NULL)  
     {  
         delete m_height;  
         m_height = NULL;
     }  
    }  
 
public:  
    int m_age;  
    int* m_height;  
};  
​  
void test01()  
{  
    Person p1(18, 180);  
    ​  
    Person p2(p1);  
    ​  
    cout << "p1的年龄： " << p1.m_age << " 身高： " << *p1.m_height << endl;  
    ​  
    cout << "p2的年龄： " << p2.m_age << " 身高： " << *p2.m_height << endl;  
}  
​  
int main() 
{  
​  
    test01();  
    ​  
    system("pause");  
    ​  
    return 0;  
}

```
> 总结：如果属性有在堆区开辟的，一定要自己提供拷贝构造函数，防止浅拷贝带来的问题
![[Pasted image 20230318235230.png]]

对于基本类型的数据以及简单的对象，它们之间的拷贝非常简单，就是按位复制内存。例如：

```c++
class Base
{
public:
    Base(): m_a(0), m_b(0){ }
    Base(int a, int b): m_a(a), m_b(b){ }
private:
    int m_a;
    int m_b;
};

int main()
{
    int a = 10;
    int b = a;  //拷贝

    Base obj1(10, 20);
    Base obj2 = obj1;  //拷贝

    return 0;
}
```

`b` 和 `obj2` 都是以拷贝的方式初始化的，具体来说，就是**将 `a` 和 `obj1` 所在内存中的数据按照二进制位（Bit）复制到 `b` 和 `obj2` 所在的内存，这种默认的拷贝行为就是<font color="#ff0000">浅拷贝</font>

对于简单的类，默认的拷贝构造函数一般就够用了，我们也没有必要再显式地定义一个功能类似的拷贝构造函数。**但是当类持有其它资源时，例如动态分配的内存、指向其他数据的指针等，默认的拷贝构造函数就不能拷贝这些资源了，我们必须显式地定义拷贝构造函数，以完整地拷贝对象的所有数据。**

#### 使用浅拷贝还是深拷贝

**如果一个类拥有指针类型的成员变量，那么绝大部分情况下就需要深拷贝，因为只有这样，才能将指针指向的内容再复制出一份来，让原有对象和新生对象相互独立，彼此之间不受影响。如果类的成员变量没有指针，一般浅拷贝足以。**

**另外一种需要深拷贝的情况就是在创建对象时进行一些预处理工作**，比如统计创建过的对象的数目、记录对象创建的时间等，请看下面的例子：

```c++
#include <iostream>
#include <ctime>
#include <windows.h>  //在Linux和Mac下要换成 unistd.h 头文件
using namespace std;

class Base{
public:
    Base(int a = 0, int b = 0);
    Base(const Base &obj);  //拷贝构造函数
public:
    int getCount() const { return m_count; }
    time_t getTime() const { return m_time; }
private:
    int m_a;
    int m_b;
    time_t m_time;  //对象创建时间
    static int m_count;  //创建过的对象的数目
};

int Base::m_count = 0;

Base::Base(int a, int b): m_a(a), m_b(b){
    m_count++;
    m_time = time((time_t*)NULL);
}

Base::Base(const Base &obj){  //拷贝构造函数
    this->m_a = obj.m_a;
    this->m_b = obj.m_b;
    this->m_count++;
    this->m_time = time((time_t*)NULL);
}

int main(){
    Base obj1(10, 20);
    cout<<"obj1: count = "<<obj1.getCount()<<", time = "<<obj1.getTime()<<endl;
   
    Sleep(3000);  //在Linux和Mac下要写作 sleep(3);
   
    Base obj2 = obj1;
    cout<<"obj2: count = "<<obj2.getCount()<<", time = "<<obj2.getTime()<<endl;

    return 0;
}
```

运行结果：  
obj1: count = 1, time = 1488344372  
obj2: count = 2, time = 1488344375

运行程序，先输出第一行结果，等待 3 秒后再输出第二行结果。Base 类中的 m_time 和 m_count 分别记录了对象的创建时间和创建数目，它们在不同的对象中有不同的值，所以需要在初始化对象的时候提前处理一下，这样浅拷贝就不能胜任了，就必须使用深拷贝了。

### 7.2 拷贝赋值运算符
```c++
Sales_data trans, accum;
trans = accum;  //使用Sales_data类的的拷贝赋值运算符
```

重载赋值运算符(=)
```c++
class Foo
{
public:
		Foo& operator=(const Foo&); //赋值运算符
		//...
}:
```


> [!NOTE] 
> 赋值运算符通常应该返回一个指向其左侧运算对象的引用

如果类未定义自己的拷贝复制运算符，编译器就会为它合成一个默认拷贝复制运算符。类似拷贝构造函数，**通常默认拷贝赋值运算符将右侧运算对象的每个非static成员赋予左侧运算对象的对应成员**，某些类的默认拷贝赋值运算符运算符用来禁止该类型对象赋值。
```c++
//下面的代码等价于Sales_data的默认拷贝赋值运算符：

Sales_data& Sales_data::operator=(const Sales_data &rhs)
{
		bookNo =rhs.bookNo;  //调用string::operator=
		units_sold=rhs.units_sold;  //使用内置的int赋值
		revenue =rhs.revenue;  //使用内置的double赋值
		return *this;  //返回一个此对象的引用
	}
```

### 7.3 析构函数
执行与构造函数相反的操作，释放对象使用的资源，并销毁对象的非static数据成员。
- 构造函数中，成员初始化在函数体执行之前，按照在类中定义顺序进行初始化
- 析构函数中，首先执行函数体，然后销毁成员，成员**按初始化顺序的逆序销毁**
```c++
class Foo
{
 public:
		 ~Foo();  //析构函数没有返回值，也不接受参数，因此不能重载（唯一性）
}
```


> [!warning] 析构函数调用时机
> 无论何时一个对象被销毁，就会自动调用其析构函数：
> 
>1. 变量在离开其作用域时被销毁。
>2. 当一个对象被销毁时，其成员被销毁。
>3. 容器（无论是标准库容器还是数组）被销毁时，其元素被销毁。
>4. 对于动态分配的对象，当对指向它的指针应用delete运算符时被销毁
>5. 对于临时对象，当创建它的完整表达式结束时被销毁。
>
> **当指向一个对象的引用或指针离开作用域时，析构函数不会执行。**

类中未定义时，编译器会合成一个默认析构函数，通常合成析构函数函数体为空，某些类的默认析构函数被用来阻止该类型的对象被销毁。

```c++
//下面代码等价于Sales_data的默认析构函数
class Sales_data
{
public:
		~Sales_data(){}
		......
}:
```


> [!tip] 疑惑
> C++primer p446结尾说到：
> 认识到**析构函数体自身**并不**直接销毁成员**是非常重要的。 **成员**是在**析构函数体**之后隐含的 **析构**阶段 中被**销毁**的。 在整个对象**销毁**过程中，**析构函数体**是作为**成员销毁**步骤之外的另一部分而进行的
> 不太理解

例子：

```c++
//Student.h
#include <string>
using namespace std;

class Student {
private:
    int num;
    string name;
    char gender;
public:
    Student();
    Student(int num, string name, char gender);
    ~Student();
    void display();
};

-------------------------------------------------------------------------------

//Student.cpp:
#include "Student.h"
#include <iostream>
using namespace std;

// 无参构造
Student::Student() : num(-1), name("None"), gender('N') {}

Student::Student(int n, string p, char g) : num(n), name(p), gender(g) {
    cout << "执行构造函数: " << "Welcome, " << name << endl;
}

void Student::display() {
    cout << "num: " << num << endl;
    cout << "name: " << name << endl;
    cout << "gender: " << gender << endl;
    cout << "===============" << endl;
}

Student::~Student() {
    cout << "执行析构函数: " << "Bye bye, " << name << endl;
}

------------------------------------------------------------------------------

//main.cpp
#include "Student.h"
#include <iostream>
using namespace std;

int main() {

    Student student1(1, "Little white", 'f');
    Student student2(2, "Big white", 'f');

    student1.display();
    student2.display();

    return 0;
}

//输出结果，注意看函数执行的顺序
执行构造函数: Welcome, Little white
执行构造函数: Welcome, Big white
num: 1
name: Little white
gender: f
===============
num: 2
name: Big white
gender: f
===============
执行析构函数: Bye bye, Big white
执行析构函数: Bye bye, Little white
```
### 7.4 三/五法则
当定义一个类时，我们显式或隐式地定义了此类型的对象在拷贝、赋值和销毁时做什么。一个类通过定义三种特殊成员成员函数来控制拷贝操作：拷贝构造函数、拷贝赋值运算符、析构函数（C++11新标准加入了移动构造函数和移动赋值运算符）。

#### 三法则

> [!NOTE] 三法则
> - 如果一个类需要自定义析构函数，那么它一定也需要自定义拷贝赋值运算符和拷贝构造函数
> - 如果一个类需要一个拷贝构造函数，那么肯定也需要一个拷贝赋值运算符，反之成立。
> - 如果一个函数需要拷贝构造运算符或者拷贝赋值运算符，不必然意味着需要析构函数
> > [!NOTE] 特例
> 基类的析构函数不遵循该原则，如果基类将析构函数设定为虚函数，此时该析构函数内容为空，我们无法由此判断该基类还需要赋值运算符或拷贝构造函数
> > >


如何理解三法则，通常，若一个类需要析构函数，则代表其合成的析构函数不足以释放类所拥有的资源，其中最典型的就是指针成员。所以，我们需要自己写析构函数来释放给指针所分配的内存来防止内存泄露。

那么为什么说“一定需要拷贝构造函数和赋值操作符”呢？

原因还是这样：类中出现了指针类型的成员。**有指针类型的成员，我们必须防止浅拷贝问题**，所以，一定需要拷贝构造函数和赋值操作符，这两个函数是防止浅拷贝问题所必须的。
```c++
class rule_of_three
{
    char* cstring; // 以裸指针为动态分配内存的句柄
 
    void init(const char* s)
    {
        std::size_t n = std::strlen(s) + 1;
        cstring = new char[n];
        std::memcpy(cstring, s, n); // 填充
    }
 public:
    rule_of_three(const char* s = "") { init(s); }
 
    ~rule_of_three()
    {
        delete[] cstring;  // 解分配
    }
 
    rule_of_three(const rule_of_three& other) // 复制构造函数
    { 
        init(other.cstring);
    }
 
    rule_of_three& operator=(const rule_of_three& other) // 复制赋值
    {
        if(this != &other) {
            delete[] cstring;  // 解分配
            init(other.cstring);
        }
        return *this;
    }
};

```

通过可复制句柄来管理不可复制资源的类，可能必须将其复制赋值和复制构造函数声明为私有的并且不提供其定义，或将它们定义为弃置的。这是三之法则的另一种应用：删除其中之一而遗留其他被隐式定义很可能会导致错误。

#### 五法则

在较新的 C++11 标准中，为了支持移动语义，又增加了移动构造函数和移动赋值运算符，这样共有五个特殊的成员函数，所以又称为“C++五法则”；
```c++
class rule_of_five
{
    char* cstring; // 以裸指针为动态分配内存的句柄
 public:
    rule_of_five(const char* s = "")
    : cstring(nullptr)
    { 
        if (s) {
            std::size_t n = std::strlen(s) + 1;
            cstring = new char[n];      // 分配
            std::memcpy(cstring, s, n); // 填充
        } 
    }
 
    ~rule_of_five()//析构函数
    {
        delete[] cstring;  // 释放内存
    }
 
    rule_of_five(const rule_of_five& other) // 复制构造函数
    : rule_of_five(other.cstring)
    {}
 
    rule_of_five(rule_of_five&& other) noexcept // 移动构造函数
    : cstring(std::exchange(other.cstring, nullptr))
    {}
 
    rule_of_five& operator=(const rule_of_five& other) // 复制赋值
    {
         return *this = rule_of_five(other);
    }
 
    rule_of_five& operator=(rule_of_five&& other) noexcept // 移动赋值
    {
        std::swap(cstring, other.cstring);
        return *this;
    }
 
// 另一种方法是用以下函数替代两个赋值运算符
//  rule_of_five& operator=(rule_of_five other) noexcept
//  {
//      std::swap(cstring, other.cstring);
//      return *this;
//  }
};

```
**也就是说，“三法则”是针对较旧的 C++89 标准说的，“五法则”是针对较新的 C++11 标准说的；为了统一称呼，后来人们干把它叫做“C++ 三/五法则”；**
### 7.5 【C++11】=default 补充

我们可以通过将拷贝控制成员定义为=default来显式地要求编译器生成合成的版本
```c++
class Sales data
{
	public:
		//拷贝控制成员：使用default
		Sales data() = default;
		Sales data(const Sales data&) = default;
		Sales data&operator=(const Sales data &);
		~Sales data() = default;
		//其他成员的定义，如前
};

Sales data&Sales data::operator=(const Sales data&)=default; //类外定义=default
```

当我们在类内用=default修饰成员的声明时，合成的函数将隐式地声明为内联的(就像任何其他类内声明的成员函数一样)。
如果我们不希望合成的成员是内联函数，应该只对成员的类外定义使用=default,就像对拷贝赋值运算符所做的那样。

**我们只能对具有合成版本的成员函数使用=default**(即，默认构造函数或拷贝控制成员)。

### 7.6 阻止拷贝
有些类不需要拷贝构造函数和拷贝赋值运算符，比如iostream类阻止了拷贝，以避免多个对象写入或读取相同的IO缓冲。
#### 【C++11】 =delete删除函数
```c++
struct NoCopy
{
		NoCopy() = default;  //使用合成的默认构造函数
		NoCopy(const NoCopy&) = delete;  //阻止拷贝
		NoCopy &operator=(const NoCopy&) = delete; //阻止赋值

		~NoCopy() = default;  //使用合成的析构函数
		//其他成员
};
```

=delete通知编译器（以及我们代码的读者），我们不希望定义这些成员。

> [!NOTE] 与=default的不同点
> 1. =delete必须出现在函数第一次声明的时候
> 2. 可以对普通函数指定=delete(=defalut只能用于具有合成版本的成员函数)
> 3. 不可删除析构函数

C++11之前，类是通过将其拷贝构造函数和拷贝赋值运算符声明为private的来阻止拷贝：
```c++
class PrivateCopy
{
		//无访问说明符；接下来的成员默认为private的
		//拷贝控制成员是private的，因此普通用户代码无法访问
		PrivateCopy(const PrivateCopy&);
		PrivateCopy &operator=(const PrivateCopy&);
		//其他成员
		public:
		PrivateCopy()=default;//使用合成的默认构造函数
		~PrivateCopy();//用户可以定义此类型的对象，但无法拷贝它们
}

```

### 7.7 拷贝控制和资源管理
P452
通常，管理类外资源的类必须定义拷贝控制成员。这种类需要通过析构函数来释放对象所分配的资源。一旦一个类需要析构函数，那么它几乎肯定也需要一个拷贝构造函数和一个拷贝赋值运算符。
**为了定义这些成员，我们首先必须确定此类型对象的拷贝语义。一般来说，有两种选择：可以定义拷贝操作，使类的行为看起来像一个值或者像一个指针。**
**类的行为像一个值，意味着它应该也有自己的状态。** 当我们拷贝一个像值的对象时，副本和原对象是完全独立的。改变副本不会对原对象有任何影响，反之亦然。例如标准库容器和string类的行为像一个值。
**行为像指针的类则共享状态。** 当我们拷贝一个这种类的对象时，副本和原对象使用相同的底层数据。改变副本也会改变原对象，反之亦然。例如shared ptr类提供类似指针的行为。
通常，类直接拷贝内置类型（不包括指针）成员：这些成员本身就是值，因此通常应该让它们的行为像值一样。**我们如何拷贝指针成员决定了像HasPtr这样的类是具有类值行为还是类指针行为。**
#### 行为像值的类
为了提供类值的行为，对于类管理的资源，每个对象都应该拥有一份自己的拷贝。这意味着对于ps指向的string,每个HasPtr对象都必须有自己的拷贝。为了实现类值行为，HasPtr需要
- 定义一个拷贝构造函数，完成string的拷贝，而不是拷贝指针
- 定义一个析构函数来释放string
- 定义一个拷贝赋值运算符来释放对象当前的string,并从右侧运算对象拷贝string

类值版本的HasPtr如下所示
```c++
class Hasptr
{
	public:
		// 构造函数
		Hasptr(const std::string &s = std:string()) : 
				ps(new std::string(s)), i(0) {}
				
		// 拷贝构造函数
		// 对ps指向的string,每个HasPtr对象都有自己的拷贝
		Hasptr(const HasPtr &p) : 
				ps(new std:string(*p.ps)),i(p.i) {}
				
		Hasptr& operator=(const Hasptr &)  // 拷贝赋值运算符
		~Hasptr() { delete ps };  //析构
	private:
		std::string *ps;
		int i;
}

// 定义拷贝赋值运算符
Hasptr& Hasptr::operator=(const HasPtr &rhs)
{
		auto newp = new string(*rhs.ps);  // 拷贝底层string
		delete ps;  // 释放旧内存
		ps = newp;  // 从右侧运算对象拷贝数据到本对象
		i = rhs.i;
		return *this;  // 返回本对象
}
```
我们的类足够简单，在类内就已定义了除赋值运算符之外的所有成员函数。第一个构造函数接受一个（可选的）string参数。这个构造函数动态分配它自己的string副本，并将指向string的指针保存在ps中。拷贝构造函数也分配它自己的string副本。析构函数对指针成员ps执行delete,释放构造函数中分配的内存。
赋值运算符通常组合了析构函数和构造函数的操作。类似析构函数，赋值操作会销毁左侧运算对象的资源。类似拷贝构造函数，赋值操作会从右侧运算对象拷贝数据。但是，非常重要的一点是，这些操作是以正确的顺序执行的，即使将一个对象赋予它自身，也保证正确。而且，如果可能，我们编写的赋值运算符还应该是异常安全的一当异常发生时能将左侧运算对象置于一个有意义的状态。

#### 行为像指针的类
对于行为类似指针的类，我们需要为其定义拷贝构造函数和拷贝赋值运算符，来拷贝指针成员本身而不是它指向的string。我们的类仍然需要自己的析构函数来释放接受string参数的构造函数分配的内存。但是，在本例中，析构函数不能单方面地释放关联的string。只有当最后一个指向string的HasPtr销毁时，它才可以释放string。
令一个类展现类似指针的行为的最好方法是使用shared_ptr来管理类中的资源。拷贝（或赋值）一个shared_ptr会拷贝（赋值）shared_ptr所指向的指针。shared_ptr类自己记录有多少用户共享它所指向的对象。当没有用户使用对象时，shared_ptr类负责释放资源。
但是，有时我们希望直接管理资源。在这种情况下，使用引用计数(reference count)就很有用了。为了说明引用计数如何工作，我们将重新定义HasPtr,令其行为像指针一样，但我们不使用shared_ptr,而是设计自己的引用计数。
引用计数的工作方式如下：
- 除了初始化对象外，每个构造函数（拷贝构造函数除外）还要创建一个引用计数，用来记录有多少对象与正在创建的对象共享状态。当我们创建一个对象时，只有一个对象共享状态，因此将计数器初始化为1。
- 拷贝构造函数不分配新的计数器，而是拷贝给定对象的数据成员，包括计数器。拷贝构造函数递增共享的计数器，指出给定对象的状态又被一个新用户所共享。
- 析构函数递减计数器，指出共享状态的用户少了一个。如果计数器变为0，则析构函数释放状态。
- 拷贝赋值运算符递增右侧运算对象的计数器，递减左侧运算对象的计数器。如果左侧运算对象的计数器变为0，意味着它的共享状态没有用户了，拷贝赋值运算符就必须销毁状态。
- 将计数器保存在动态内存中。当创建一个对象时，我们也分配一个新的计数器。当拷贝或赋值对象时，我们拷贝指向计数器的指针。使用这种方法，副本和原对象都会指向相同的计数器。
```c++
class Hasptr
{
	public:
		//构造函数分配新的string和新的计数器，将计数器置为1
		Hasptr(const std:string &s = std::string()):
			ps(new std::string(s)),i(0),use(new std::size t(1)){}
			
		//拷贝构造函数拷贝所有三个数据成员，并递增计数器
		Hasptr(const Hasptr &p) : ps(p.ps),i(p.i),use(p.use)
		{ ++*use };
		
		Hasptr&operator=(const Hasptr&);
		~Hasptr();
private:
		std::string *ps;
		int i;
		std::size_t *use;//用来记录有多少个对象共享*ps的成员
};

Hasptr::~HasPtr()
{
		//如果引用计数变为0
		if(--*use==0)
		{
			delete ps;  //释放string内存
			delete use;  //释放计数器内存
		}
}

Hasptr& Hasptr::operator=(const Hasptr &rhs)
{
		++*rhs.use;//递增右侧运算对象的引用计数
		
		//然后递减本对象的引用计数
		if (--*use == 0){
			delete ps;  //如果没有其他用户
			delete use;  //释放本对象分配的成员
		}
		
		ps = rhs.ps;
		//将数据从rhs拷贝到本对象
		i = rhs.i;
		use = rhs.use;
		return *this;
		//返回本对象
}
```

#### 自定义类中的swap函数
交换两个类值对象得代码可能是这样得：
```c++
Hasptr temp = v1;  //创建v1的值的一个临时副本
v1 = V2;  //将v2的值赋予v1
v2 = temp;  //将保存的v1的值赋予v2
```
拷贝了三次，我们不希望有这些多余的内存操作，可以通过交换指针实现
```c++
string *temp = v1.ps;  //为v1.ps中的指针创建一个副本
v1.ps = v2.ps;  //将v2.ps中的指针赋予v1.ps
v2.ps = temp;  //将保存的v1.ps中原来的指针赋予v2.ps
```

这样我们就能编写优化后的swap函数
```c++
class Hasptr
{
		friend void swap(Hasptr&, Hasptr&);
		//其他成员定义，与"行为像值的类"小节中一样
}:

inline
void swap(HasPtr &lhs,Hasptr &rhs)
{
		//std::swap是标准库中的swap函数，因为本例数据成员是内置类型，所以可以这样是哟个，如果一个类的数据成员有自己特定的swap函数，就不能使用std::swap
		std::swap(Ihs.ps,rhs.ps);  //交换指针，而不是string数据
		std::swap(Ihs.i,rhs.i);  //交换int成员
}
```

**定义swap的类通常用swap来定义它们的赋值运算符**。这些运算符使用了一种名为**拷贝并交换(copy and swap)** 的技术。这种技术将左侧运算对象与右侧运算对象的一个副本进行交换：
```c++
//注意rhs是按值传递的，意味着HasPtr的拷贝构造函数将右侧运算对象中的string拷贝到rhs
HasPtr& Hasptr::operator=(Hasptr rhs)
{
		//交换左侧运算对象和局部变量rhs的内容
		swap(*this, rhs);  //rhs现在指向本对象曾经使用的内存
		return *this;  //rhs被销毁，从而delete了rhs中的指针
}
```
在这个版本的赋值运算符中，参数并不是一个引用，我们将右侧运算对象以传值方式传递给了赋值运算符。因此，rhs是右侧运算对象的一个副本。参数传递时拷贝HasPtr的操作会分配该对象的string的一个新副本。
在赋值运算符的函数体中，我们调用swap来交换rhs和* this中的数据成员。这个调用将左侧运算对象中原来保存的指针存入rhs中，并将rhs中原来的指针存入* this中。因此，在swap调用之后，* this中的指针成员将指向新分配的string一右侧运算对象中string的一个副本。
当赋值运算符结束时，rhs被销毁，HasPtr的析构函数将执行。此析构函数delete rhs现在指向的内存，即，释放掉左侧运算对象中原来的内存。
**这个技术的有趣之处是它自动处理了自赋值情况且天然就是异常安全的**。它通过在改变左侧运算对象之前拷贝右侧运算对象保证了自赋值的正确。

### 7.8 【C++11】对象移动
重新分配内存的过程中，从旧内存拷贝到新内存是不必要的，更好的方式是移动元素，开销更低。（旧C++标准只能通过拷贝）

> [!NOTE] 
> 1. 标准库容器、string和shared_ptr类既支持移动也支持拷贝。
> 2. IO类和unique_ptr类可以移动但不能拷贝（因为包含不能被共享的资源（如指针或IO缓冲））。

#### 右值引用
”右值引用“就是必须绑定到右值的引用，通过&&获得。
”右值引用“只能绑定到一个将要销毁的对象，因此可以自由的将右值引用的资源“移动" 到另一个对象中。
常规引用又称为”左值引用“
```c++
int i = 42;
int &r = i;  //正确：r引用i
int &&rr = i;  //错误：不能将一个右值引用绑定到一个左值上
int &r2 = i * 42;  //错误：i * 42是一个右值
const int &r3 =i * 42;  //正确：我们可以将一个const的引用绑定到一个右值上
int &&rr2 = i * 42;  //正确：将rr2绑定到右值上
```
#### std::move函数
std::move显式地将一个左值对象转换为对应的右值引用类型，该对象称为”移后源对象“。

```c++
int &&rr = std::move(i); //正确
```
调用`std::move`就意味着承诺：除了对i赋值或销毁它外，我们将不再使用它。

**原理P610**

【C++11】可以使用`static_cast`显示的将一个左值转换为一个右值引用，不安全，还是用`std::move`比较好
#### 移动构造函数和移动赋值运算符
```c++
//移动构造函数，引用参数使用了右值引用符号&&
StrVec::StrVec(StrVec &&s) noexcept //移动操作不应抛出任何异常
//成员初始化器接管s中的资源
: elements(s.elements),first free(s.first free),cap(s.cap)
{
		//令S进入这样的状态：对其运行析构函数是安全的
		s.elements = s.first_free =  s.cap = nullptr;
	}

//移动赋值运算符
StrVec &StrVec::operator=(StrVec &&rhs) noexcept
{
		//直接检测自赋值
		if (this != rhs)
		{
			free();  //释放已有元素
			elements = rhs.elements;  //从rhs接管资源
			first_free = rhs.first_free;
			cap = rhs.cap;  
			//将rhs置于可析构状态
			rhs.elements = rhs.first_free = rhs.cap = nullptr;
		}
			return *this;
	}

```

#### noexcept
noexcept通知标准库我们的构造函数不抛出任何异常，必须在类头文件中的声明和定义中（如果定义在类外的话）都指定noexcept
```c++
class StrVec
{
public:
		StrVec(StrVec&& ) noexcept;//移动构造函数
		//其他成员的定义，如前
};
		StrVec::StrVec(StrVec &&s) noexcept:/*成员初始化器*/
		{ /*构造函数体*/ }
```

#### 默认移动函数
只有当一个类没有定义任何自己版本的拷贝控制成员，**且它的所有数据成员都能移动构造或移动赋值时**，编译器才会为它合成移动构造函数或移动赋值运算符。
```c++
//编译器会为X和hasX合成移动操作
struct X
{
		int i;  //内置类型可以移动
		std:string s;  //string定义了自己的移动操作
};

struct hasX
{
		X mem; //X有合成的移动操作
}
X x;
X x2 = std:move(x);  //使用合成的移动构造函数
hasx hx,hx2 = std:move(hx);  //使用合成的移动构造函数
```

#### 移动迭代器
一般的迭代器的解引用返回指向一个元素的左值，但移动迭代器的解引用生成一个右值引用。
`make_move_iterator`函数将一个普通迭代器转换为一个移动迭代器，此函数接受一个迭代器参数，返回一个移动迭代器（可以使用和普通迭代器同样的操作）。
```c++
void StrVec::reallocate()
{
		//分配大小两倍于当前规模的内存空间
		auto newcapacity = size()? 2 * size() : 1;
		auto first = alloc.allocate(newcapacity);
		//移动元素
		auto last = uninitialized_copy(make_move_iterator(begin()),make move iterator (end()),first);
		free();  //释放旧空间
		elements = first;  //更新指针
		first free = last;
		cap = elements + newcapacity;
	}
```

#### 引用限定符

在参数列表后放置一个引用限定符，可以是&或&&，分别指出this可以指向一个左值或右值。
只能用于非static成员函数，且必须同时出现在函数的声明和定义中。

对于&限定的函数，我们只能将它用于左值：对于&&限定的函数，只能用于右值：
```c++
Foo &retFoo();  //返回一个引用；retFoo调用是一个左值
Foo retval();  //返回一个值；retVal调用是一个右值
Foo i,j;  //i和j是左值
i=j;  //正确：1是左值
retFoo() = j;  //正确：retFoo()返回一个左值
retval() = j;  //错误：retVal()返回一个右值
i = retval();  //正确：我们可以将一个右值作为赋值操作的右侧运算对象
```

一个函数可以同时用const和引用限定。在此情况下，引用限定符必须跟随在const限定符之后：
```c++
class Foo
{
public:
		Foo anotherMem() const &  //正确：const限定符在前
		Foo someMem() & const;  //错误：const限定符必须在前
};
```

> [!NOTE] Title
> 如果一个成员函数有引用限定符，则具有相同参数列表的所有版本都必须有引用限定符


# 七、IO 库
## 1 IO 类
cin>> 读进去
cout <<写出去
endl是一个被称为**操纵符**的特殊值，写入endl的效果是结束当前行，并将与设备关联的缓冲区（buffer）中的内容刷到设备中。

![[Pasted image 20230211200011.png]]
![[Pasted image 20230211195929.png]]

![[Pasted image 20230211200859.png]]
![[Pasted image 20230211200910.png]]
- 以上操作都是操纵char数据的，为了支持宽字符wchar_t的语言，只需要在操作符前面加上w就可，如wcout、wistream
- ifstream和istringstream都继承自istream，因此他们也可以执行cin、cout、>>、getline等操作。
- 不能拷贝或对IO对象赋值

> [!NOTE] Title
> 本节剩下部分所介绍的标准库流特性都可以无差别地应用于普通流、文件流和string流以及char或宽字符流版本。

## 2 管理输出缓冲
每个输出流都管理一个缓冲区，用来保存程序读写的数据
- 导致缓冲刷新（即，数据真正写到输出设备或文件）的原因有很多：
- 程序正常结束，作为main函数的return操作的一部分，缓冲刷新被执行。
- 缓冲区满时，需要刷新缓冲，而后新的数据才能继续写入缓冲区。
- 我们可以使用操纵符（如endl）来显式刷新缓冲区。
- 在每个输出操作之后，我们可以用操纵符unitbuf设置流的内部状态，来清空缓冲区。默认情况下，对cerr是设置unitbuf的，因此写到cerr的内容都是立即刷新的。
- **一个输出流可能被关联到另一个流。** 在这种情况下，**当读写被关联的流时，关联到的流的缓冲区会被刷新**。例如，默认情况下，cin和cerr都关联到cout。因此，读cin或写cerr都会导致cout的缓冲区被刷新。
```c++
cout<<"hi!"<<endl;  //输出hi和一个换行，然后刷新缓冲区
cout<<"hi！"<<flush:  //输出hi，然后刷新缓冲区，不附加任何额外字符
cout<<"hi!"<<ends;  //输出hi和一个空字符，然后刷新缓冲区

int i;
cin>>i; //cin和cout关联，因此该句将cout的缓冲区刷新
```

> [!warning] 警告
> 程序崩溃，输出缓冲区不会被刷新

## 3 文件 IO
文件类型分为两种：

1.  **文本文件** - 文件以文本的**ASCII码**形式存储在计算机中
2.  **二进制文件** - 文件以文本的**二进制**形式存储在计算机中，用户一般不能直接读懂它们

头文件 < fstream > 定义了三个类型来支持文件IO：

`ifstream` 读文件
`ofstream` 写文件
`fstream` 读写文件
![[Pasted image 20230211202030.png]]
- 当一个fstream对象被销毁时（比如离开作用域时），close会自动被调用，文件自动关闭。

![[Pasted image 20230211202818.png]]
- 每个文件流类型都定义了一个默认的文件模式，ifstream是in，ofstream是out，fstream是in和out。后面的例子我显式的写了出来
- 文件模式可以组合使用，利用 “|” 操作符。
例如：用二进制方式写文件 ofstream :: binary | ofstream :: out
```c++
ofstream ofs("test.txt", ofstream::binary | ofstream::out);
```
- 以out模式打开文件，文件的内容会被丢弃，若想保留ofstream打开的文件中已有的数据，唯一办法是显式指定app或in模式
> [!NOTE] 【C++11】文件名
> C++11中，文件名既可以是库类型string对象，也可以是指向C风格字符串的指针

### 写文件ofstream
写文件步骤如下：

1.  **包含头文件**
`#include <fstream>
2.  **创建流对象**
    `ofstream ofs;`
3.  **打开文件**
    `ofs.open("文件名",打开方式);`
4.  **写数据**
`ofs << "写入的数据";`
5.  **关闭文件**
`ofs.close();`

```c++
#include <fstream>
using namespace std;

int main()
{
	ofstream ofs;
	ofs.open("test.txt", ofstream::out);
	//ofstream ofs("test.txt", ofstream::out); //等价上面两句，提供文件名时，open自动被调用
	
	ofs << "姓名：张三" << endl;
	ofs.close();

	return 0;
}
```

### 读文件ifstream
步骤相似，有三种读取数据的方法

```c++
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

int main()
{

	ifstream ifs("test.txt", ifstream::in);

	if (!ifs.is_open())
	{
		cout << "文件打开失败" << endl;
		return 0;
	}

	//第一种方式  将数据存入数组
	char buf[1024] = { 0 };  //初始化一个数组
	while (ifs >> buf)
	{
		cout << buf << endl;
	}

	//第二种
	char buf[1024] = { 0 };
	while (ifs.getline(buf, sizeof(buf)))
	
	{
		cout << buf << endl;
	}
	
	//第三种
	string buf;
	while (getline(ifs, buf))
	{
		cout << buf << endl;
	}
	
	ifs.close();

	return 0;
}
```

## 4 string 流
`sstream`头文件定义了三个类型来支持内存IO，这些类型可以向string写入数据，
从string读取数据，就像string是一个IO流一样。

`istringstream`从string读取数据
`ostringstream`向string写入数据
`stringstream`既可从string读数据也可向string写数据。

![[Pasted image 20230211210322.png]]

当我们想对整行文本进行处理，同时对行内每个单词进行处理时可以使用`istringstream`
```c++
//文件内列举了姓名和他拥有的电话号码
//test.txt
morgan 2015552368 8625550123
drew 9735550130
1ee 6095550132 2015550175 8005550000

//程序：
#include <iostream>
#include <sstream>
#include <string>
#include <vector>
using namespace std;

struct PersonInfo
{
	string name;	//存姓名
	vector<string> phones;	//村电话
};

int main()
{
		string line, word;	//分别保存来自输入的一行和单词
		vector<PersonInfo>people;	//保存来自输入的所有记录

	//逐行从输入读取数据，直至cin遇到文件尾（或其他错误）
	while (getline(cin, line))
	{
		PersonInfo info;	//创建一个保存此记录数据的对象
		istringstream record(line);	//将记录绑定到刚读入的行
		record >> info.name;	//读取名字
		while (record >> word)	//读取电话号码
			info.phones.push_back(word);	//保存

		people.push_back(info);	//将此记录追加到people末尾
	}
	return 0；
}
```

## 5 格式控制
除了条件状态外，每个iostream对象还维护一个**格式状态**来控制IO如何格式化的细节。
格式状态控制格式化的某些方面，如整型值是几进制、浮点值的精度、一个输出元素的宽度等。
标注库定义了一组**操纵符**来修改流的格式状态。
![[Pasted image 20230227221944.png]]
![[Pasted image 20230227221955.png]]
- 一个操纵符是一个函数或者是一个对象，会影响流的状态，并能用作输入或输出运算符的运算对象。
- 类似输入和输出运算符，操纵符也返回它所处理的流对象，因此我们可以在一条语句中组合操纵符和数据。

操作符用于两大类输出控制：控制数值的输出形式以及控制补白的数量和位置。

大多数改变格式状态的操纵符都是设置/复原成对的，当操纵符改变流的格式状态时，通常改变后的状态对所有后续IO都生效。

例：
```c++
//boolalpha控制布尔值的格式
//默认情况下，bool打印1或0，一个true输出整数1，使用boolalpha之后会直接打印true
cout <<"default bool values:" << true << "" <<false
		<<"\nalpha bool values:" << boolalpha
		<< true << "" << false << endl;

//输出
//default bool values:1 0
//alpha bool values:true false

//使用noboolapha复原
bool bool_val = get_status();
cout << boolalpha  //设置cout的内部状态
<< bool_val
<< noboolalpha;    //将内部状态恢复为默认格式
```

```c++
//指定整型值的进制
cout<<"default:"<<20<<""<<1024<<endl;  //默认十进制
cout<<"octa1:"<<oct<<20<<""<<1024<<endl;  //十六进制
cout<<"hex:"<<hex<<20<<""<<1024<<endl;  //八进制
cout<<"decimal:"<<dec<<20 <<""<<1024<<endl;  //十进制

//输出
//default:20 1024
//octa1:242000
//hex:14400
//decimal:20 1024
```

# 八、顺序容器
容器均定义为模板类

`vector`
**可变大小数组**。支持快速随机访问。在尾部之外的位置插入或删除元素可能很慢

`array`
**固定大小数组**。支持快速随机访问。不能添加或删除元素

`string`
与 vector 相似的容器，但专门用于保存字符。随机访问快。在尾部插入/删除速度快

`deque`
**双端队列**。支持快速随机访问。在头尾位置插入删除速度很快

`list`
**双向链表**。只支持双向顺序访问。在list中任何位置进行插入/删除操作速度都很快

`forward_list`
**单向链表**。只支持单向顺序访问。在链表任何位置进行插入/删除操作速度都很快

## 容器定义和初始化
![[Pasted image 20230211220558.png|650]]

**特别的`array`**
定义一个array时，除了指定元素类型，还要指定容器大小
```c++
array<int, 10>
```
此外，其他容器默认构造都是空容器，而array默认构造是非空的：它包含了与其大小一样多的默认初始化元素。


> [!NOTE] 关键概念：容器元素是拷贝
> 当我们用一个对象来初始化容器时，或将一个对象插入到容器中时，实际上放入到容器中的是对象值的一个拷贝，而不是对象本身。就像我们将一个对象传递给非引用参数一样，容器中的元素与提供值的对象之间没有任何关联。随后对容器中元素的任何改变都不会影响到原始对象，反之亦然。

## 通用容器操作
以下操作基本适用于所有容器
![[Pasted image 20230211213826.png]]
![[Pasted image 20230211214254.png]]

### swap和assign
![[Pasted image 20230211221941.png]]

### 反向迭代器
对于反向迭代器，递增（以及递减）操作的含义会颠倒过来。递增一个反向迭代器（++it）会移动到前一个元素; 递减一个迭代器 (—-it）会移动到下一个元素。
除了 `forward_list` 之外, 其他容器都支持反向迭代器。我们可以通过调用 rbegin、rend、crbegin 和 crend 成员函数来获得反向迭代器。**这些成员函数返回指向容器尾元素和首元素之前一个位置的迭代器**。与普通迭代器一样，反向迭代器也有 const 和非 const 版本。
![[Pasted image 20230824095050.png]]
```c++
vector<int> vec = {0,1,2,3,4,5,6,7,8,9};//从尾元素到首元素的反向迭代器
for (auto r_iter = vec.crbegin( );  //将r_iter绑定到尾元素
    r_iter != vec.crend ( ) ; //crend指向首元素之前的位置
    ++r_iter) //实际是递减，移动到前一个元素
{
    cout <<*r_iter<< endl; //打印9,8，7， ... 0
}
```

通过向 sort 传递一对反向迭代器来讲 vector 在整理为递减序列
```c++
sort (vec.begin () , vec.end ()); //按“正常序”排序vec
sort (vec.rbegin () , vec.rend ( )); //按逆序排序:将最小元素放在vec的末尾
```

## 顺序容器操作
顺序容器特有的操作
### 添加元素
![[Pasted image 20230211222902.png]]
【C++11】接受元素个数或范围的insert版本返回指向第一个新加入元素的迭代器。如果范围为空，不插入任何元素，insert操作会将第一个参数返回。

【C++11】**`emplace`**
emplace操作不是拷贝而是构造元素。

emplce_front对应push_front
emplce对应insert
emplce_back对应push_back

当调用push或insert成员函数时，我们将元素类型的对象传递给它们，这些对象被拷贝到容器中。
而当我们调用一个emplace成员函数时，则是将参数传递给元素类型的构造函数。emplace成员使用这些参数在容器管理的内存空间中直接构造元素。
### 访问元素
![[Pasted image 20230211224406.png]]
- 在容器中访问成员函数返回的都是引用，如果容器是一个const对象，则返回值是const引用。
- `at` 能够检测下标是否合法，防止越界。 
### 删除元素
![[Pasted image 20230211224643.png]]

**特殊的forward_list操作**
![[Pasted image 20230211232259.png]]
### 改变容器的大小
![[Pasted image 20230211232558.png]]

### 额外的string操作

## 顺序容器迭代器

```c++
//通常使用auto简写迭代器类型
auto b = v.begin(); // begin()返回指向第一个元素的迭代器
auto c = v.end(); // end()返回指向尾元素的下一位置的迭代器（尾后迭代器）
```

> [!NOTE] 
> 特殊的，如果容器为空，则begin和end返回的是同一个迭代器，都是尾后迭代器。
> 
迭代器主要有iterator（本质是`T*`）和const_iterator（本质是常量指针`const T*`）两种类型

```c++
//迭代器类型
vector<int>::iterator it;  //it能读写vector<int>的元素
vector<int>::const_iterator it; //只能读元素，不能写元素

string<int>::iterator it;  //it能读写vector<string>的元素
string<int>::const_iterator it; //只能读元素，不能写元素
```

如果vector对象或string对象是一个常量，那么只能使用const_iterator；
如果不是常量，那么两种迭代器都可以使用。

### begin和end成员
begin和end有多个版本
rbegin：返回反向迭代器(reverse_iterator)
cbegin：返回const迭代器（const_iterator）

### 运算
![[Pasted image 20230209204934.png]]
forward_list的迭代器不支持递减运算符（--）

**以下运算只能应用于string、vector、deque和array的迭代器**
![[Pasted image 20230209205951.png]]
迭代器相减的结果是两个迭代器的距离，类型是名为`difference_type`的带符号整型数。

### 容器操作导致的迭代器失效
在向容器添加元素后：
1. 如果容器是vector或string,且存储空间被重新分配，则指向容器的迭代器、指针和引用都会失效。如果存储空间未重新分配，指向插入位置之前的元素的迭代器、指针和引用仍有效，但指向插入位置之后元素的迭代器、指针和引用将会失效。
2. 对于deque,插入到除首尾位置之外的任何位置都会导致迭代器、指针和引用失效。如果在首尾位置添加元素，迭代器会失效，但指向存在的元素的引用和指针不会失效。
3. 对于list和forward_list,指向容器的迭代器（包括尾后迭代器和首前迭代器)、指针和引用仍有效。

当我们从一个容器中删除元素后，指向被删除元素的迭代器、指针和引用会失效，这应该不会令人惊讶。毕竟，这些元素都已经被销毁了。

当我们删除一个元素后：
1. 对于list和forward_list,指向容器其他位置的迭代器（包括尾后迭代器和首前迭代器)、引用和指针仍有效。
2. 对于deque,如果在首尾之外的任何位置删除元素，那么指向被删除元素外其他元素的迭代器、引用或指针也会失效。如果是删除deque的尾元素，则尾后迭代器也会失效，但其他迭代器、引用和指针不受影响：如果是删除首元素，这些也不会受影响。
3. 对于vector和string,指向被删元素之前元素的迭代器、引用和指针仍有效。注意：当我们删除元素时，尾后迭代器总是会失效。


> [!command] 建议：管理迭代器
> 当你使用迭代器（或指向容器元素的引用或指针）时，**最小化要求迭代器必须保持有效**的程序片段是一个好的方法。由于向迭代器添加元素和从迭代器删除元素的代码可能会使迭代器失效，因此必须保证每次改变容器的操作之后都正确地重新定位迭代器。
> 这个建议对vector、string和deque尤为重要。
> 


> [!Warning] warning
> 凡是使用了迭代器的循环体，都不要向迭代器所属的容器添加元素
> 
> 如果在一个循环中插入/删除deque、string或vector中的元素，不要缓存end返回的迭代器。
## array

当**创建 array** 时就要**初始化其大小**，不可再改变。

```c++
#include <array>  // 先要包含头文件
int main() {
    std::array<int, 5> data;  //定义，有两个参数，一个指定类型，一个指定大小
    data[0] = 1;
    data[4] = 10;
    return 0;
}
```

array 和原生数组都是**创建在栈上**的（vector 是在堆上创建底层数据储存的）

原生数组越界的时候不会报错，而 **array 会有越界检查**，会报错提醒。

使用 std::array 的好处是可以**访问它的大小**（通过 s**ize()** 函数），它是一个**类**。

```c++
#include<iostream>
#include<array>

void PrintArray(const std::array<int, 5>& data)  //显式指定了大小 {
    for (int i = 0;i < data.size();i++)  //访问数组大小
    {
        std::cout << data[i] << std::endl;
    }
}
int main() {
    std::array<int, 5> data;
    data[0] = 0;
    data[1] = 1;
    data[2] = 2;
    data[3] = 3;
    data[4] = 4;
    PrintArray(data);
    std::cin.get();
}
```

如何传入一个**标准数组作为参数**，但**不知道数组的大小**？

方法：使用模板

```c++
#include <iostream>
#include <array>

template <typename T>
void printarray(const T &data) {
    for (int i = 0; i < data.size(); i++)
    {
        std::cout << data[i] << std::endl;
    }
}

template <typename T, unsigned long N> // or // template <typename T, size_t N>
void printarray2(const std::array<T, N> &data) {
    for (int i = 0; i < N; i++)
    {
        std::cout << data[i] << std::endl;
    }
}

int main() {
    std::array<int, 5> data;
    data[0] = 2;
    data[4] = 1;
    printarray(data);
    printarray2(data);
}
```

## vector
### 初始化
![[Pasted image 20230209195935.png]]
### 操作
![[Pasted image 20230209200056.png]]

> [!bug] 下标操作
> vector对象的下标类型仍是对应的size_type，和string一样，只能对已存在的元素执行下标操作，否则会造成缓冲区溢出，在运行时产生一个不可预估的值。
> 
> **确保下标合法的一种有效手段就是尽可能使用范围for语句。**

### 内存空间增长策略
vector 本质上是一个动态数组, 内存连续存储。

**Vector对象是如何增长的？**
假定容器中元素是连续存储的，且容器的大小是可变的，考虑向vector或string中添加元素会发生什么：如果没有空间容纳新元素，容器不可能简单地将它添加到内存中其他位置，因为元素必须连续存储。容器必须分配新的内存空间来保存已有元素和新元素，将已有元素从旧位置移动到新空间中，然后添加新元素，释放旧存储空间。
如果我们每添加一个新元素，vector就执行一次这样的内存分配和释放操作，性能会慢到不可接受。
为了避免这种代价，标准库实现者采用了可以减少容器空间重新分配次数的策略。**当不得不获取新的内存空间时，vector和string的实现通常会分配比新的空间需求更大的内存空间。容器预留这些空间作为备用，可用来保存更多的新元素。这样，就不需要每次添加新元素都重新分配容器的内存空间了。**

vs编辑器每次扩容1.5倍：[C++vector的动态扩容，为何是1.5倍或者是2倍](https://blog.csdn.net/qq_44918090/article/details/120583540)

![[Pasted image 20230211234640.png]]

只有当需要的内存空间超过当前容量时，reserve调用才会改变vector的容量。
如果需求大小大于当前容量，reserve至少分配与需求一样大的内存空间（可能更大）。
如果需求大小小于或等于当前容量，reserve什么也不做。特别是，当需求大小小于当前容量时，容器不会退回内存空间。因此，在调用reserve之后，capacity将会大于或等于传递给reserve的参数。
这样，调用reserve永远也不会减少容器占用的内存空间。类似的，resize成员函数（参见9.3.5节，第314页）只改变容器中元素的数目，而不是容器的容量。我们同样不能使用resize来减少容器预留的内存空间。

### capacity和size
**区别：**
size是指容器已经保存的元素的数目；
capacity则是在不分配新的内存空间的前提下它最多可以保存多少元素。

### vector 使用优化

vecctor 的优化策略：

**问题 1：** 当向 vector 数组中**添加新元素**时，为了扩充容量，**当前的 vector 的内容会从内存中的旧位置复制到内存中的新位置** (产生一次复制)，然后删除旧位置的内存。 简单说，push_back 时，容量不够，会自动调整大小，重新分配内存。这就是将代码拖慢的原因之一。
**解决办法：** vertices.reserve(n) ，直接指定容量大小，避免重复分配产生的复制浪费。  

**问题 2：** 在非 vector 内存中创建对象进行初始化时，即 push_back() 向容器尾部添加元素时，首先会创建一个临时容器对象（不在已经分配好内存的 vector 中）并对其追加元素，然后再将这个对象拷贝或者移动到【我们真正想添加元素的容器】中 。这其中，就造成了一次复制浪费。
**解决办法：** **emplace_back**，直接在容器尾部创建元素，即直接在已经分配好内存的那个容器中直接构造元素，不创建临时对象。

简单的说：
**reserve 提前申请内存**，避免动态申请开销 **emplace_back 直接在容器尾部构造元素**，省略拷贝或移动过程

```c++
#include <iostream>
#include <vector>

struct Vertex
{
    float x, y, z;

    Vertex(float x, float y, float z)
        : x(x), y(y), z(z)
    {
    }

    Vertex(const Vertex& vertex)
        : x(vertex.x), y(vertex.y), z(vertex.z)
    {
        std::cout << "Copied!" << std::endl;
    }
};

int main() {
    std::vector<Vertex> vertices;
    vertices.push_back(Vertex(1, 2, 3 )); //同vertices.push_back({ 1, 2, 3 });
    vertices.push_back(Vertex(4, 5, 6 ));
    vertices.push_back(Vertex(7, 8, 9 ));

    std::cin.get();
}
```

输出：

```c++
Copied!
Copied!
Copied!
Copied!
Copied!
Copied!
```

**发生六次复制的原因：**

理解一：

环境: VS2019，x64，C++17 标准，经过我自己的测试，vector 扩容因子为 1.5，初始的 capacity 为 0.  
第一次 push_back，capacity 扩容到 1，临时对象拷贝到真正的 vertices 所占内存中，第一次 Copied；第二次 push_back，发生扩容，capacity 扩容到 2，vertices 发生内存搬移发生的拷贝为第二次 Copied，然后再是临时对象的搬移，为第三次 Copied；接着第三次 push_back，capacity 扩容到 3（2 * 1.5 = 3，3 之后是 4，4 之后是 6...），vertices 发生内存搬移发生的拷贝为第四和第五个 Copied，然后再是临时对象的搬移为第六个 Copied；

理解二：

```c++
std::vector<Entity> e;
    Entity data1 = { 1,2,3 }; 
    e.push_back( data1); // data1->新vector内存
    Entity data2 = { 1,2,3 }; 
    e.push_back( data2 ); //data1->新vector内存   data2->vector新vector内存  删除旧vector内存
    Entity data3 = { 1,2,3 };
    e.push_back(data3);  // data1->新vector内存  data2->vector新vector内存  data3->vector新vector内存  删除旧vector内存
所以他的输出的次数分别是1，3，6
他的复制次数你可以这样理解递增。 1+2+3+4+5+....
```

解决:

```c++
int main() {   
    std::vector<Vertex> vertices;
    //ver 1 : copy 6 times
    vertices.push_back({ 1,2,3 });
    vertices.push_back({ 4,5,6 });
    vertices.push_back({ 7,8,9 });

    //ver 2 : copy 3 times
    vertices.reserve(3);
    vertices.push_back({ 1,2,3 });
    vertices.push_back({ 4,5,6 });
    vertices.push_back({ 7,8,9 });

    //ver 3 : copy 0 times
    vertices.reserve(3);
    vertices.emplace_back(1, 2, 3);
    vertices.emplace_back(4, 5, 6);
    vertices.emplace_back(7, 8, 9);

    std::cin.get();
}
```

## string
标准库类型String表示可变长的字符序列

### 初始化
![[Pasted image 20230209190416.png]]
### 操作
![[Pasted image 20230209190604.png]]

#### getline函数
虽然可以使用 cin 和 >> 运算符来输入字符串，但它可能会导致一些需要注意的问题。

当 cin 读取数据时，它会传递并忽略任何前导白色空格字符（空格、制表符或换行符）。一旦它接触到第一个非空格字符即开始阅读，当它读取到下一个空白字符时，它将停止读取。以下面的语句为例：

 cin >> name;

可以输入 "Liu" 或 "Ke"，但不能输入 "Liu Ke"，因为 cin 不能输入包含嵌入空格的字符串。下面程序演示了这个问题：
```c++
 #include <iostream>  
 #include <string>   
 using namespace std;  
 int main()  
 {  
     string name;  
     string city;  
     cout << "Please enter your name: ";  
     cin >> name;  
     cout << "Enter the city you live in: ";  
     cin >> city;  
     cout << "Hello, " << name << endl;  
     cout << "You live in " << city << endl;  
     return 0;  
 }
```

程序输出结果：
```c++
 //若输入不带空格  
 Please enter your name： Liu  
 Enter the city you live in: Shandong  
 //前两行 信息分别手动输入  
 Hello,Liu  
 You live in Shandong  
       
 //若输入带空格  
 Please enter your name: Liu Ke  
 //输入Liu Ke后直接输出结果，没有机会输入城市名  
 Enter the city you live in: Hello, Liu  
 You live in Ke  
```

 因为在第一个输入语句中，当 cin 读取到 Liu 和 Ke 之间的空格时，它就会停止阅读，只存储 Liu 作为 name 的值。在第二个输入语句中， cin 使用键盘缓冲区中找到的剩余字符，并存储 Ke 作为 city 的值。

为了解决带空格字符串的输入问题，使用**getline**函数，该函数可读取整行，包括前导和嵌入的空格，并将其存储在字符串对象中。
```c++
 getline(cin, inputLine);  
 //cin 是正在读取的输入流  
 //inputLine 是接收输入字符串的 string 变量的名称
```

```c++
#include <iostream>  
 #include <string>   
 using namespace std;  
 int main()  
 {  
     string name;  
     string city;  
     cout << "Please enter your name: ";  
     getline(cin, name);  
     cout << "Enter the city you live in: ";  
     getline(cin, city);  
     cout << "Hello, " << name << endl;  
     cout << "You live in " << city << endl;  
     return 0;  
 }

输出结果：

 Please enter your name: Liu Ke  
 Enter the city you live in: Shandong  
 //可正常输入带空格字符串  
 Hello, Liu Ke  
 You live in Shandong
```
#### string::size_type类型
size()函数返回string::size_type类型的值，string[x]的下标x也是string::size_type类型。
该类型是无符号整型，而且能足够存放下任何string对象的大小。

> [!bug] 防止混用
> 如果一条表达式中已经有了size()函数，就不要再使用int了，这样可以避免混用带来的问题，问题主要出自于int是有符号数，和无符号数运算会出问题

#### 比较string长度
如果两个string对象在某些写对应的位置上不一致，则string对象比较的结果其实是string对象中第一对相异字符比较的结果
```c++
string a = "Hello";
string b = "Hello World"
string c = "Hiya"
```
a < b
c > a且c > b

#### 字面值和string相加
当string对象和字符字面值以及字符串字面值混在一条语句中使用时，要确保每个+运算符两侧的运算对象至少有一个是string：
```c++
string s1 = "Hello"
string s2 = s1 + "World"; //正确，string+字面值 
string s3 = "hello" + "world"; //错误，两个运算对象都不是string 
string s4 =  "world" + "!" + s1; //错误，不能把字面值直接相加
```
	
### 额外的String操作
**除了顺序容器通用的操作外，string类型还提供了一些额外的操作。**
- 提供string类和C风格字符串之间的相互转换
- 增加了允许我们用下标代替迭代器的版本

#### 初始化
![[Pasted image 20230212153613.png]]

#### substr操作
返回一个string，他是原始string的一部分或全部的拷贝，可以传递给substr一个可选的开始位置和计数值：
```c++
//substr(开始位置，计数值)
string s("hello world");
string s2 = s.substr(0,5);  //s2=he11o
string s3 = s.substr(6);  //s3=world
string s4 = s.substr(6,11);  //s3=world
string s5 = s.substr(12);  //抛出一个out_of_range异常
```

#### 改变string的其他方法
![[Pasted image 20230212154457.png]]![[Pasted image 20230212154543.png]]
#### string搜索操作
![[Pasted image 20230212154657.png]]
![[Pasted image 20230212154705.png]]

> [!warning] 
> string搜索函数返回string:size_type值，该类型是一个unsigned类型。因此，用一个int或其他带符号类型来保存这些函数的返回值不是一个好主意。

#### compare比较函数
![[Pasted image 20230212155018.png]]
#### 数值转换
![[Pasted image 20230212155134.png]]

> [!NOTE] 
> 如果string不能转换为一个数值，这些函数抛出一个invalid_argument异常（参见5.6节，第173页）。如果转换得到的数值无法用任何类型来表示，则抛出一个out_of_range异常。

### 处理string对象中的字符
![[Pasted image 20230209193035.png]]

> [!comment] 建议
> 建议：使用C++版本的C标准库头文件
>
>C++标准库中除了定义C+语言特有的功能外，也兼容了C语言的标准库。C语言的头文件形如name.h,C++则将这些文件命名为cname。也就是去掉了.h后缀，而在文件名name之前添加了字母c,这里的c表示这是一个属于C语言标准库的头文件。
>
>因此，cctype头文件和ctype.h头文件的内容是一样的，只不过从命名规范上来讲更符合C++语言的要求。特别的，在名为cname的头文件中定义的名字从属于命名空间std,而定义在名为.h的头文件中的则不然。
>
>一般来说，C++程序应该使用名为cname的头文件而不使用name.h的形式。

## 顺序容器适配器
容器、迭代器和函数都有**适配器（adaptor）**，标准库定义了三个顺序容器适配器：
**stack** 栈适配器
**queue** 队列适配器
**priority_queue**

**本质上，一个适配器是一种机制，能使某种事物的行为看起来像另外一种事物一样。一个容器适配器接受一种己有的容器类型，使其行为看起来像一种不同的类型。**

例如，stack适配器接受一个顺序容器（除array或forward_list外），并使其操作起来像一个stack一样。

![[Pasted image 20230212163058.png]]
### 定义适配器
```c++
stack<int> intStack; //空栈
stack<int> intStack(deq); //接受一个容器的构造函数，拷贝该容器来初始化适配器，假设deq是一个deque<int>
```
**默认情况**下，stack和queue是基于deque实现的，priority_queue是在vector之上实现的。
我们可以在创建一个适配器时**将一个命名的顺序容器作为第二个类型参数**，来**重载默认容器类型**。
```c++
//在vector上实现的空栈
stack<string,vector<string>> str_stk;
//str stk2在vector上实现，初始化时保存svec的拷贝
stack<string,vector<string>> str stk2(svec);
```


> [!warning] 重载类型限制
> 所有适配器都要求容器具有添加和删除元素的能力。因此，适配器不能构造在array之上。
> 
类似的，我们也不能用forward_list来构造适配器，因为所有适配器都要求容器具有添加、删除以及访问尾元素的能力。
>
stack只要求push back、pop back和back操作，因此可以使用除array和forward list之外的任何容器类型来构造stack。
>
queue适配器要求back、push_back、front和push_front,因此它可以构造于list或deque之上，但不能基于vector构造。
>
priority_queue除了front、push_back和pop_back操作之外还要求随机访问能力，因此它可以构造于vector或deque之上，但不能基于list构造。

### 栈适配器 stack 

```c++
#include <stack> //头文件

int main()
{
		stack<int> intStack; //空栈

		//入栈
		for (size_t i = 0; i!=10; ++i)
		intStack.push(i);
	
		//intStack保存0到9十个数
		while (!intStack.empty()) 
		{
			int value = intStack.top();
			//使用栈顶值的代码
			intStack.pop();//出栈，弹出栈顶元素
			
			return 0;
}
```

![[Pasted image 20230212164240.png]]

### 队列适配器 queue
queue使用先入先出（FIFO）策略
priority_queue允许我们为队列中的元素建立优先级，新加入的元素会排在所有优先级比他低的已有元素之前。

![[Pasted image 20230212165605.png]]
![[Pasted image 20230212165613.png]]
# 九、泛型算法

## 0 算法文档P770

> [!Tip] 
> 标准库容器定义的操作集合很小。标准库并未给每个容器添加大量功能，而是提
供了一组算法，这些算法中的大多数都独立于任何特定的容器。这些**算法是通用的
(generic,或称泛型的)：它们可用于不同类型的容器和不同类型的元素。**
```c++
//头文件
#include <algorithm> //包含大多数泛型算法     
#include <numeric> //数值泛型算法
```

> [!NOTE] 关键概念：算法永远不会执行容器的操作
> 
泛型算法本身不会执行容器的操作，它们只会运行于迭代器之上，执行迭代器的操作。
**算法永远不会改变底层容器的大小。** 算法可能改变容器中保存的元素的值，也可能在容器内移动元素，但永远不会直接添加或删除元素。
当一个算法操作**插入迭代器**时，插入迭代器可以完成向容器添加元素的效果，但算法自身永远不会做这样的操作。

- **`beg`和`end`是表示元素范围的迭代器。** 几乎所有算法都对一个由beg和end表示的序列进行操作。
- **`beg2`是表示第二个输入序列开始位置的迭代器。`end2`表示第二个序列的末尾位置（如果有的话）。** 如果没有end2,则假定beg2表示的序列与beg和end表示的序列一样大。beg和beg2的类型不必匹配，但是，必须保证对两个序列中的元素都可以执行特定操作或调用给定的可调用对象。
- **`dest`是表示目的序列的迭代器。** 对于给定输入序列，算法需要生成多少元素，目的序列必须保证能保存同样多的元素。
- **`unaryPred`和`binaryPred`是一元和二元谓词**，分别接受一个和两个参数，都是来自输入序列的元素，两个谓词都返回可用作条件的类型。
- **`comp`是一个二元谓词**，满足关联容器中对关键字序的要求。
- **`unaryOp`和`binaryOp`是可调用对象**，可分别使用来自输入序列的一个和两个实参来调用。

待摘抄...
## 1 只读算法
只读取其输入范围内的元素，不改变元素
`find`
`count`

`accumulate`（定义在头文件numeric）
accumulate函数接受三个参数，前两个指出了需要求和的元素的范围，第三个参数是和的初值。第三个参数的类型决定了函数中使用哪个加法运算符以及返回值的类型。
假定vec是一个整数序列，则：
```c++
//对vec中的元素求和，和的初值是0
int sum accumulate(vec.cbegin(),vec.cend(),0);
```
这条语句将sum设置为vec中元素的和，和的初值被设置为0。

`equal`：用于确定两个序列是否保存相同的值。如果所有对应元素都相等，则返回
true,否则返回false。
此算法接受三个迭代器：前两个表示第一个序列中的元素范围，第三个表示第二个序列的首元素：
```c++
//roster2中的元素数目应该至少与roster1一样多
equal(roster1.cbegin(),roster1.cend(),roster2.cbegin());
```

**equal基于一个非常重要的假设：它假定第二个序列至少与第一个序列一样长。**

> [!NOTE]  关键概念：迭代器参数
**一些算法从两个序列中读取元素。构成这两个序列的元素可以来自于不同类型的容
器，两个序列中元素的类型也不要求严格匹配。** 算法要求的只是能够比较两个序列中的元素。例如，对equal算法，元素类型不要求相同，但是我们必须能使用 == 来比较来自两个序列中的元素。
>
**用一个单一迭代器表示第二个序列的算法都假定第二个序列至少与第一个一样长。**

## 2 写算法
`fill`：接受一对迭代器表示一个范围，还接受一个值作为第三个参数，将给定的这个值赋予输入序列中的每个元素。
```c++
fi1l(vec.begin(),vec.end(),0);//将每个元素重置为0
//将容器的一个子序列设置为10
fill(vec.begin(),vec.begin(),vec.size()/2,10);
```

`fill_n`接受一个单迭代器、一个计数值和一个值。它将给定值赋予迭代器指向的元素开始的指定个元素。我们可以用fill_n将一个新值赋予vector中的元素：
```c++
vector<int>vec;  // 空vector
fill_n(vec.begin(),10,0);  //错误：vec是空的，不能写入10个元素。

fill_n(vec.begin(),vec.size(),0); // 正确，将所有元素重置为0
```

> [!warning] 
>向目的位置迭代器写入数据的算法假定目的位置足够大，能容纳要写入的元素。

## 3 拷贝算法
此算法接受三个迭代器，前两个表示一个输入范围，第三个表示目的序列的起始位置。此算法**将输入范围中的元素拷贝到目的序列中**。传递给copy的目的序列至少要包含与输入序列一样多的元素，这一点很重要。

我们可以用coPy实现内置数组的拷贝，如下面代码所示：
```c++
inta1[]={0,1,2,3,4,5,6,7,8,9}:
inta2[sizeof(a1)/sizeof(*a1)】;//a2与a1大小一样
//ret指向拷贝到a2的尾元素之后的位置
auto ret=copy(begin(a1),end(a1),a2);//把a1的内容拷贝给a2
```

copy返回目的位置迭代器（递增后）的值，即ret指向拷贝到a2的尾元素之后的位置。

**多个算法都提供“拷贝”版本。** 这些算法计算新元素的值，但不会将它们放置在输入序列的末尾，而是创建一个新序列保存这些结果。
`replace`算法读入一个序列，并将其中所有等于给定值的元素都改为另一个
值。此算法接受4个参数：前两个是迭代器，表示输入序列，后两个一个是要搜索的值，
另一个是新值。它将所有等于第一个值的元素替换为第二个值：
```c++
//将所有值为0的元素改为42
replace (ilst.begin(),ilst.end(),0,42);
```
此调用将序列中所有的0都替换为42。如果我们希望保留原序列不变，可以调用`replace_copy`。此算法接受额外第三个迭代器参数，指出调整后序列的保存位置：
```c++
//使用back inserter按需要增长目标序列
replace_copy(ilst.cbegin(),ilst.cend(),
back_inserter (ivec),0,42);
```
此调用后，i1st并未改变，ivec包含i1st的一份拷贝，不过原来在ilst中值为0的元素在ivec中都变为42。

## 4 排序算法
`sort`：从小到大排序，使用<运算符
`unique`：sort后将相邻的重复项隐藏（并不是删除，放在了末尾），返回的迭代器指向最后一个不重复元素之后的位置

原vector：
![[Pasted image 20230212174503.png]]
sort后：
![[Pasted image 20230212174441.png]]
unique后：![[Pasted image 20230212174447.png]]
`stable_sort`：等长元素字典排序
## 5 定制操作
**用lambda表达式定制操作**

### 谓词：向算法传递函数

> [!NOTE] 谓词
> **谓词是一个可调用的表达式，是返回值为 `bool` 的普通函数或者函数对象**
> 
> 标准库算法所使用的谓词分为两类：
> 一元谓词(unary predicate,意味着它们只接受单一参数)
> 二元谓词(binary predicate,意味着它们有两个参数)。
> 
> 接受谓词参数的算法对输入序列中的元素调用谓词。因此，元素类型必须能转换为谓词的参数类型。

###  内置的sort函数

1. sort(vec.begin(), vec.end(), 谓词)

**谓词可以设置排序的规则，谓语可以是内置函数，也可以是 lambda 表达式。**

2. **默认**是从小到大排序

```c++
#include<iostream>
#include<vector>
#include<algorithm>

int main() {
    std::vector<int>  values = {3, 5, 1, 4, 2};
    std::sort(values.begin(), values.end());  //默认用法，不写谓词
    for (int value : values)
		    std::cout << value << std::endl; // 1 2 3 4 5
    std::cin.get();
}
```

3. 使用内置函数，添加头文件 functional，使用 **std::greater** 函数，则会按照**从大到小**顺序排列。

```c++
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main() {
    std::vector<int>  values = {3, 5, 1, 4, 2};             
    std::sort(values.begin(), values.end(),std::greater<int>()); 
    for (int value : values)
		    std::cout << value << std::endl; // 5 4 3 2 1
    std::cin.get();
}
```

4. 使用 lambda 进行灵活排序

```c++
std::sort(values.begin(), values.end(), 
		  [](int a, int b){return a < b});//从小到大排序
    

a < b //返回true，a排在前面。此时为升序排列（如果a小于b，那么a就排在b的前面）
a > b //返回true, a排在前面，此时为降序排列（如果a大于b，那么a就排在b的前面）
```

对于已定的传入参数的顺序`[](int a, int b)`，函数体中如果参数 a 在前面，则返回 true，如果参数 a 在后面则返回 false

```c++
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main() {
    std::vector<int>  values = {3, 5, 1, 4, 2};          

    std::sort(values.begin(), values.end(), [](int a, int b)
    {
            return a < b;  // 如果a小于b，那么a就排在b的前面。 1 2 3 4 5
    });

    for (int value : values)
    std::cout << value << std::endl;

    std::cin.get();
}
```

5. 如果把 1 排到最后

如果 a == 1，则把它移到后面去，即返回 false，不希望它在 b 前。 如果 b == 1，我们希望 a 在前面，要返回 true。

```c++
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main() {
    std::vector<int>  values = {3, 5, 1, 4, 2};          
    std::sort(values.begin(), values.end(), [](int a, int b)
    {
            if (a == 1)
                return false;
            if(b == 1)
                return true;
            return a < b;   //2 3 4 5 1
    });
    for (int value : values)
    std::cout << value << std::endl;
    std::cin.get();
}
```

### find_if
我们还可以写一个 lambda 接受 vector 的整数元素，遍历这个 vector 找到比 3 大的整数，然后返回它的迭代器，也就是满足条件的第一个元素。 

**`find_if`是一个搜索类的函数，区别于`find`的是：它可以接受一个函数指针来定义搜索的规则，返回满足这个规则的第一个元素的迭代器**。这个情况就很适合 lambda 表达式的出场了

```c++
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> values = { 1, 5, 3, 4, 2 };
    //下面就用到了lambda作为函数指针构成了find_it的规则
    auto it = std::find_if(values.begin(), values.end(), [](int value) { return value > 3; });  //返回第一个大于3的元素的迭代器 
    std::cout << *it << std::endl;  //将其输出
}
```
### for_each
for_each时vector的一个操作
```c++
//for_each原理 这里的MayPrint可以用lambda改写
for_each(v.begin(),v.end(),MyPrint); 

void MyPrint(int val) 
{ 
		cout<<val<<endl; 
}
```

```c++
//每次打印后面接一个空格
vector<string> v;

for_each(v.end, v.end(), [](const string &s){cout<< s <<" ";});
cout<<endl;
```


### 【C++11】参数绑定 bind 函数
#bind
#### 绑定普通函数
可以将 bind 函数看作一个通用的函数适配器，接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表。

```c++
//调用bind的一般形式：
auto newCallable = bind (callable, arg_list);

//newCallable本身是一个可调用对象
//arg_list是一个逗号分隔的参数列表，对应给定的callable的参数。
//即当我们调用newCallable时，newCallable会调用callable,并传递给它arg list中的参数。
```
`arg_list`中的参数可能包含形如`_n`的名字，其中n是一个整数。这些参数是“占位符”,表示newCallable的参数，它们占据了传递给newCallable的参数的“位置”。
数值n表示生成的可调用对象中参数的位置：`_1`为newCallable的第一个参数，`_2`为第二个参数，依此类推。
名字_n都定义在名为**placeholders命名空间**，该命名空间定义在std命名空间。使用时，都要写上。

#### 绑定类的成员函数
类的成员函数必须通过类的对象或者指针调用，因此在 bind 时， `arg_list` 中的 第一个参数的位置来指定一个类的实列、指针或引用。
```c++
class Test
{
public:
    int funs(int val)
    {
        std::cout << "hello world" << val << std::endl;
        return val;
    }
};
 
class message
{
public:
    std::function<int()> fun;
};
 
int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);
 
    Test test;
    message *mes = new message;
    mes->fun = std::bind(&Test::funs,test,2);  //test为类的实例
    cout << mes->fun() <<endl;
 
    return a.exec();
}
```

**bind功能如下：**
#### 修正参数的值
```c++
using namespace std::placeholders; 

bool check_size(const std::string &s, std::string::size_type sz)
{
		return s.size()>=sz;
}

//check6是一个可调用对象，接受一个string类型的参数
//并用此string和值6来调用check_size
auto check6 = bind(check_size, _1, 6);
//占位符出现在arg_list的第一个位置，表示check6的此参数对应check_size的第一个参数。此参数是一个const string&
//因此调用check6必须传递给它一个string类型的参数，check6会将此参数传递给check_size
string s = "hello";
bool b1 = check6(s); //check6(s)会调用check size(s,6)
```

#### 重排参数顺序
例如，假定 f 是一个可调用对象，它有5个参数，则下面对 bind 的调用：
```c++
//g是一个有两个参数的可调用对象
auto g = bind(f,a,b,_2,c,_1);
```

生成一个新的可调用对象，它有两个参数，分别用占位符_2和_1表示。这个新的可调用对象将它自己的参数作为第三个和第五个参数传递给f。f的第一个、第二个和第四个参数分别被绑定到给定的值a、b和c上。
传递给g的参数按位置绑定到占位符。即，第一个参数绑定到_1，第二个参数绑定到_2。因此，当我们调用g时，其第一个参数将被传递给f作为最后一个参数，第二个参数将被传递给f作为第三个参数。实际上，这个bind调用会将`g(_1,_2)`映射为`f(a,b,_2,c,1)`即，对g的调用会调用f,用g的参数代替占位符，再加上绑定的参数a、b和c。例如，调用`g(X,Y)`会调用`f(a,b,Y,c,X)`

下面是用bind重排参数顺序的一个具体例子，我们可以用bind颠倒isShroter
的含义：
```c++
//按单词长度由短至长排序
sort (words.begin(), words.end(), isShorter);
//按单词长度由长至短排序
sort (words.begin(), words.end(), bind(isShorter,2,1));
```
在第一个调用中，当sort需要比较两个元素A和B时，它会调用isShorter(A,B)。
在第二个对sort的调用中，传递给isShorter的参数被交换过来了。因此，当sort比较两个元素时，就好像调用isShorter(B,A)一样。

#### ref 函数：绑定引用参数
#ref
默认情况下，bind的那些不是占位符的参数被拷贝到bind返回的可调用对象中。但是，与lambda类似，**有时对有些绑定的参数我们希望以引用方式传递，或是要绑定参数的类型无法拷贝。**
例如，为了替换一个引用方式捕获ostream的lambda:
```c++
//os是一个局部变量，引用一个输出流
//c是一个局部变量，类型为char
for_each(words.begin(), words.end(), [&os,c](const string &s){os << s <<c;})

//可以很容易地编写一个函数，完成相同的工作：
ostream &print(ostream &os, const string &s, char c)
{
		return os << s << c;
}

//但是，不能直接用bind来代替对os的捕获：
//错误：不能拷贝os
for_each (words.begin(), words.end(), bind(print, os, _1,' '));
```

原因在于 bind 拷贝其参数，而我们不能拷贝一个 ostream。如果我们希望传递给 bind 一个对象而又不拷贝它，就必须使用标准库 `ref` 函数：
```c++
for_each (words.begin(),words.end(), bind(print,ref(os),1,''));
```
函数ref返回一个对象，包含给定的引用，此对象是可以拷贝的。标准库中还有一个`cref`函数，生成一个保存const引用的类。与bind一样，函数ref和cref也定义在头文件functional中。

## 6 泛型迭代器
除了为每个容器定义的迭代器之外，标准库在头文件`iterator`中还定义了额外几种迭代器。

1. **插入迭代器(insert iterator)**：绑定到一个容器上，可用来向容器插入元素。
3. **流迭代器(stream iterator)**：绑定到输入或输出流上，可用来遍历所关联的IO流。
4. **反向迭代器(reverse iterator)**：向后而不是向前移动。除了forward_list之外的标准库容器都有反向迭代器。
6. **移动迭代器(move iterator)**：这些专用的迭代器不是拷贝其中的元素，而是移动它们。

### 插入迭代器
插入器是一种迭代器适配器，它接受一个容器，生成一个迭代器，能实现向给定容器添加元素。当我们通过一个插入迭代器进行赋值时，该迭代器调用容器操作来向给定容器的指定位置插入一个元素。

![[Pasted image 20230212223317.png]]

通常情况，当我们通过一个迭代器向容器元素赋值时，值被赋予迭代器指向的元素。而当我们通过一个插入迭代器赋值时，一个与赋值号右侧值相等的元素被添加到容器中。

**插入器有三种类型，差异在于元素插入的位置。**
#### back_inserter
- `back_inserter` 创建一个使用push back的迭代器。back inserter接受一个指向容器的引用，返回一个与该容器绑定的插入迭代器。当我们通过此迭代器赋值时，赋值运算符会调用push_back将一个具有给定值的元素添加到容器中：
```c++
vector<int> vec;//空向量
auto it=back_inserter(vec);//通过它赋值会将元素添加到vec中
*it=42;//vec中现在有一个元素，值为42
```
我们常常使用back_inserter来创建一个迭代器，作为算法的目的位置来使用。例如：
```c++
vector<int> vec;//空向量
//正确：back_inserter创建一个插入迭代器，可用来向vec添加元素
fill_n(back_inserter(vec),10,0);//添加10个元素到vec
```

#### front_inserter
- `front_inserter` 创建一个使用 push_front 的迭代器。元素总是插入到容器第一个元素之前。
```c++
list<int> lst = {1,2,3,4};
list<int> lst2,list3; //空list

//拷贝完成之后，lst2包含4 3 2 1
copy(lst.cbegin(), lst.cend(), front_inserter(lst2));
```

#### inserter
- `inserter` 创建一个使用insert的迭代器。此函数接受第二个参数，这个参数必须是一个指向给定容器的迭代器。元素将被插入到给定迭代器所表示的元素之前。
```c++
auto it = inserter(c,iter);
*it = val;

//等价
it = c.insert(it,val);
++it //递增it使他指向原来的元素
```

> [!NOTE] 
> 只有在容器支持push front的情况下，我们才可以使用front_inserter。类似的，只有在容器支持push_back的情况下，我们才能使用back_inserter。

### iostream流迭代器
iostream类型不是容器，但是标准库也定义了用于这些IO对象的迭代器。**迭代器将它们对应的流当作一个特定类型的元素序列来处理**。通过使用流迭代器，**可以用泛型算法**从流对象读取数据以及写入数据。

创建流迭代器，必须指定迭代器要读写的类型。
#### istream_iterator
**可以为任何具有输入运算符（>>）的类型定义ostream iterator对象**。
![[Pasted image 20230212231129.png]]

```c++
istream_iterator<int> int_it(cin);  //从cin读取int
istream_iterator<int> int_eof;  //默认初始化迭代器，可以组为尾后值使用

ifstream ifs("afile");
istream_iterator<string> str_it(ifs);  //从"afile"读取字符串
```

```c++
//例子：从标准输入读取数据，存入vector
istream_iterator<int> in_iter(cin);  //从cin读取int
istream_iterator<int> eof;  //尾后迭代器

//当有数据可供读取时
while(in_iter != eof)  
		//后置递增运算读取流，返回迭代器的旧值
		//解引用迭代器，获得从流读取的前一个值
		vec.push_back(*in_iter++);

//等价
vector<int> vec(in_iter, eof); //从迭代器范围构造vec
```

#### ostream_iterator
**可以为任何具有输出运算符（<<）的类型定义ostream iterator对象**。

创建ostream_iterator时，我们可以**提供（可选的）第二参数，它是一个字符串，在输出每个元素后都会打印此字符串**。此字符串必须是一个C风格字符串（即，一个字符串字面常量或者一个指向以空字符结尾的字符数组的指针)。
必须将ostream_iterator绑定到一个指定的流，不允许空的或表示尾后位置的ostream_iterator。

![[Pasted image 20230212231331.png]]
```c++
//输出值的序列
ostream_iterator<int> out_iter(cout, " ");
for(auto e : vec)
		*out_ite++ = e; //赋值语句实际将元素写到cout
cout<<endl; 

//可以用copy来打印vec中的元素，比循环更简单
copy(vec.begin(), vec.end(), out_iter);
cout<<endl;
```

### 反向迭代器
rbegin、rend、crbegin、crend
对于反向迭代器，递增递减操作含义会倒过来，递增迭代器会移动到前一个元素。
```c++
vector<int> vec = {0,1,2,3,4};

//从尾元素到首元素的反向迭代器
for(auto r_iter = vec.rbegin();r_iter!=vec.rend();++r_iter)
		cout<< *r_iter <<endl;
```

## 7 泛型算法结构
### 迭代器类别
![[Pasted image 20230212233213.png]]
### 算法形参模式
```c++
alg (beg, end, other args);
alg (beg, end, dest, other args);
alg (beg, end, beg2, other args);
alg (beg, end, beg2, end2, other args);
```

beg、end、dest都是迭代器参数。除了这些参数，一些算法还接受额外的、非迭代器的特定参数。

**接受单个目标迭代器的算法**
dest参数是一个表示算法可以写入的目的位置的迭代器。
算法假定(assume)：按其需要写入数据，不管写入多少个元素都是安全的。

> [!NOTE] 
> 向输出迭代器写入数据的算法都假定目标空间足够容纳写入的数据。

**接受第二个输入序列的算法**
如果一个算法接受beg2和end2,这两个迭代器表示第二个范围。这类算法接受两个完整指定的范围：[ beg,end ) 表示的范围和 [ beg2end2 ) 表示的第二个范围。

只接受单独的beg2 ( 不接受end2 ) 的算法将beg2作为第二个输入范围中的首元素。此范围的结束位置未指定，这些算法假定从beg2开始的范围与beg和end所表示的范围至少一样大。

> [!NOTE] 
> 接受单独beg2的算法假定从beg2开始的序列与beg和end所表示的范围至少一样大。

### 算法命名规范
一些算法使用重载形式传递一个谓词
```c++
unique(beg, end);  //使用==运算符比较元素
unique(beg, end, comp)  //使用comp比较元素
```

_ if版本的算法
接受一个元素值得算法通常都有一个接受谓词的版本(不是重载)
```c++
find(beg,end,val);   
find_if(beg,end,lambada);
```

## 8 链表list容器算法 
 
> [!NOTE] 
> 对于list和forward_list，应该**优先使用成员函数版本**的算法而不是通用算法。
>
链表特有的操作会改变底层的容器

 ![[Pasted image 20230212235051.png]]
![[Pasted image 20230212235106.png]]

特有的splice成员
![[Pasted image 20230212235202.png]]

# 十、关联容器
- 关联容器中的元素是按关键字来保存和访问的
- 关联容器的迭代器都是双向的
![[Pasted image 20230213162157.png]]
- 有序关联容器使用比较运算符 `<` 来组织元素，按关键值字典排序从小到大 
- 无序关联容器使用哈希函数（hash function）和关键字类型 `==` 运算符来组织元素 
## 1 map映射
又称关联数组：关键字-值（键值对）
```c++
//初始化
map<string, size_t> word_count; //空容器
map<string,string> m = { {"a","b"},
							{"c","d"},
							{"e","f"} };
```

**使用map：**
```c++
//统计每个单词在输入中出现的次数
map<string, size_t> word_count;  //string到size_t的空map
string word;
while (cin >> word)
    ++word_count[word]; //提取word的计数器并加1

for(const auto &w : word_count)  
{
    cout << w.first << "出现" << w.second << "次" << endl;
}
```

![[Pasted image 20230213163856.png]]
## 2 set集合
只有关键字，支持关键字查询操作——检查一个给定关键字是否再set中
```c++
//初始化
set<string> s = {"a","b","c"};
```

**使用set：**
```c++
//统计每个单词在输入中出现的次数，并忽略指定的单词
	map<string, size_t> word_count;  //string到size_t的空map
	set<string> exclude = { "I","i","You","you" }; //忽略单词集合

	string word;
	while (cin >> word)
	{
		//只统计不在exclude中的单词
		if(exclude.find(word) == exclude.end())
			++word_count[word]; //提取word的计数器并加1
	}

	for(const auto &w : word_count)  
	{
		cout << w.first << "出现" << w.second << "次" << endl;
	}

```
![[Pasted image 20230213170315.png]]
find调用返回一个迭代器，如果给定关键字在set中，迭代器指向该关键字，否则，find返回尾后迭代器。
## 3 pair类型
定义在头文件utility中
- 一个pair保存两个数据成员，必须提供两个类型名
- pair的数据成员是public的，两个成员分别命名为first，second
```c++
//初始化
pair<string,size_t> p; //pair的默认构造函数对数据成员进行值初始化，一个是空string，一个是0
pair<string,size_t> p{"a",2};
```

![[Pasted image 20230213172051.png]]

【C++11】**可以对返回值进行列表初始化**

```c++
pair<string,int> process(vector<string>& v)
{
    //处理v
    //若v不为空，我们返回一个由v中最后一个string及其大小组成的pair。
    //否则，隐式构造一个空pair,并返回它。
    if (!v.empty ()
        return{ v.back(),v.back().size() };//列表初始化
    else
        return pair<string,int>();//隐式构造返回值
}
```

在较早的C++版本中，不允许用花括号包围的初始化器来返回pair这种类型的对象，
必须显式构造返回值：
```c++
if (!v.empty ()
		return pair<string,int>(v.back(),v.back().size());
```

**我们还可以用`make_pair`来生成pair对象**，pair的两个类型来自于make pair的
参数：
```c++
if (!v.empty ()
		return make_pair(v.back(),v.back().size());

```

## 4 关联容器操作
![[Pasted image 20230213172656.png]]

```c++
set<string>::value_type v1;  //v1是一个string
set<string>::key_type v2;  //v2是一个string
map<string,int>:value_type v3;  //v3是一个pair<const string,int>
map<string,int>:key_type v4;  //v4是一个string
map<string,int>:mapped_type v5;  //v5是一个int
```

### 关联容器迭代器
解引用关联容器迭代器，得到类型为value_type，**value_type是一个pair类型，其first成员保存const关键字（const key_type），second保存值（maped_type）**

> [!NOTE] 
> 一个 map 的 value_type 是一个 pair，可以改变 pair 的值，不能改变 pair 的关键字。
> 同样，set的关键字也不能改变

**map的value_type就是pair类型**，所以支持first，second操作
```c++
map<string, size_t> word_count;
//获得指向word_count中一个元素的迭代器
auto map_it =  word_count.begin ()  //*map_it是指向一个pair<const string,size t>对象的引用
cout << map_it->first;  //打印此元素的关键字
cout << map_it->second;  //打印此元素的值
map_it->first = "new key";  //错误：关键字是const的
++map_it->second;  //正确：我们可以通过迭代器改变元素

set<int> iset = {0,1,2,3}
set<int>::iterator set_it = iset.begin();
*set_it = 42;  //错误，set关键字是const的
```

### 遍历关联容器
支持begin和end

```c++
//获得一个指向首元素的迭代器
auto map_it = word_count.cbegin();
//比较当前迭代器和尾后迭代器
while (map_it != word count.cend())
{
		//解引用迭代器，打印关键字-值对
		cout << map_it->first << "出现" < <map_it->second << "次"<< endl;
		++map_it;//递增迭代器，移动到下一个元素
}
```

### insert添加元素
insert向容器中添加一个元素或一个元素范围（begin，end）
![[Pasted image 20230213180409.png]]

```c++
//map添加
//向word_count插入word的4种方法
word_count.insert ({word,1});  //最方便
word_count.insert (make_pair (word,1));
word_count.insert (pair<string,size t> (word,1));
word_count.insert (map<string,size t>::value_type (word,1));
```

```c++
//set添加
vector<int> ivec = {2,4,6,8,2,4,6,8}:  //ivec有8个元素
set<int> set2;  //空集合
set2.insert (ivec.cbegin(), ivec.cend());  //set2有4个元素
set2.insert((1,3,5,7,1,3,5,7}):  //set2现在有8个元素，重复的不会加入
```

### erase删除元素
![[Pasted image 20230213195217.png]]
### map的下标操作
![[Pasted image 20230213195731.png]]
map下标运算符接受一个关键字，获取此关键字相关联的值，如果关键字不在map中就会创建一个元素插入到map中，值进行初始化。
```c++
map <string, size_t> word_count;  //空map
// 插入一个关键字为“Anna”的元素，值初始化为0；
// 然后将1赋给值，值变为1
word_count[”Anna“] = 1;

//我们指向查找值，不想改变map，那就不能使用下标操作，可以用find代替
//find查找失败返回尾后迭代器，即end迭代器
if(word_count.find("foobar") == word_count.end())
		cout<< "foobar is not in the map" <<endl;
```

vector,string解引用迭代器返回的类型与下标运算符返回的类型一样，但是map不一样。
**map下标操作获得mapped_type对象，解引用获得value_type对象**
### 访问元素
find：查找
count：统计数量
![[Pasted image 20230213200150.png]]
![[Pasted image 20230213200158.png]]

> [!NOTE] Title
> lower bound返回的迭代器可能指向一个具有给定关键字的元素，但也可能
不指向。
>
**如果关键字不在容器中，lower_bound和upper_bound返回相同的迭代器**（返回关键字的第一个安全插入点）一一不影响容器中元素顺序的插入位置

#### multimap或multiset中查找
Multimap和multiset允许关键字重复，这些元素再容器中相邻存储

给定一个作者到著作题目的映射，我们想打印一个特定作者的所有著作
方法一：使用find和count：
```c++
string search_iter("Alain de Botton");  //要查找的作者
auto entries = authors.count(search_item);  //元素的数量
auto iter = authors.find(search_item);  //作者的第一本书

//用一个循环查找此作者的所有著作
while(entries)
{
		cout<< iter-> second <<endl;
		++iter;
		--entries;
}
```

#### lower_bound
lower_bound 指向第一个具有给定关键字的元素
upper_bound指向给定关键字的下一个位置

方法二：
```c++
// authors 和 search_item的定义，与前面的程序一样
//beg和end对应此作者元素的范围
for(auto beg = authors.lower_bound(search_item), end = authors.upper_bound(search_item); beg!=end; ++beg)
{
		cout<< beg->second <<endl;
}
```

#### equal_range函数
equal_range函数s接受一个关键字，返回一个迭代器pair。
若关键字存在，则第一个迭代器指向第一个与关键字匹配的元素，第二个迭代器指向最后一个匹配元素之后的位置
若未找到匹配元素，则两个迭代器都指向关键字可以插入的位置

相当于lower_bound和upper_bound的结合体。

方法三：
```c++
// authors 和 search_item的定义，与前面的程序一样
// pos保存迭代器对，表示与关键字匹配的元素范围
for(auto pos = authors.equal_range(search_item);pos.first != pos.second; ++pos.first)
{
		cout<< pos.first->second <<endl;
}
```

## 5 无序关联容器
unorderd：不按关键字顺序，使用哈希函数（hash function）和关键字类型== 运算符来组织元素
**使用有序容器相同的操作，只是不排序**

> [!Tip] 
> 如果关键字类型固有就是无序的，或者性能测试发现问题可以用哈希技术解决，就可以使用无序容器。
> 使用无序容器通常性能更好
### 管理桶
- 无序容器在存储上组织为一组**桶（bucket）**，每个桶保存零个或多个元素。
- 使用一个哈希函数将元素映射到桶。
- 为了访问一个元素，容器首先计算元素的哈希值，它指出应该搜索哪个桶。
- 容器将具有一个特定哈希值的所有元素都保存在相同的桶中。如果容器允许重复关键字，所有具有相同关键字的元素也都会在同一个桶中。因此，无序容器的性能依赖于哈希函数的质量和桶的数量和大小。

无序容器提供了一组管理桶的函数，这些成员函数允许我们查询容器的状态以及在必要时强制容器进行重组。
![[Pasted image 20230213213719.png]]

# 十一、动态内存
到目前为止，我们编写的程序中所使用的对象都有着严格定义的生存期。
**全局对象**：在程序启动时分配，在程序结束时销毁。
**局部自动对象**：当我们进入其定义所在的程序块时被创建，在离开块时销毁。
**局部static对象**：在第一次使用前分配，在程序结束时销毁。

> [!NOTE] 自动对象
>  **只有当定义它的函数被调用时才存在的对象**称为自动对象。

除了自动和static对象外，C++还支持**动态分配对象**。动态分配的对象的生存期与它们在哪里创建是无关的，**只有当显式地被释放时，这些对象才会销毁。**
动态对象的正确释放被证明是编程中极其容易出错的地方。为了更安全地使用动态对象，标准库定义了两个智能指针类型来管理动态分配的对象。当一个对象应该被释放时，指向它的智能指针可以确保自动地释放它。

我们的程序到目前为止只使用过静态内存或栈内存。

**静态内存**：保存局部static对象、类static数据成员以及定义在任何函数之外的变量。 
**栈内存**：保存定义在函数内的非static对象。
	
**分配在静态或栈内存中的对象由编译器自动创建和销毁**。对于栈对象，仅在其定义的程序块运行时才存在：static对象在使用之前分配，在程序结束时销毁。

除了静态内存和栈内存，每个程序还拥有一个内存池。
这部分内存被称作**自由空间(free store)或堆(heap)**：程序**用堆来存储动态分配(dynamically allocate)的对象**，即那些在程序运行时分配的对象。**动态对象的生存期由程序来控制，也就是说，当动态对象不再使用时，我们的代码必须显式地销毁它们**。

new：在动态内存中为对象分配空间并返回一个指向该对象的指针
delete：接受一个动态对象的指针，销毁该对象，并释放与之关联的内存

> [!bug] 手动释放内存很不安全
> 忘记释放内存：内存泄漏
> 尚有指针引用内存的情况下释放了内存：产生引用非法内存的指针
> 

## 手动管理内存
#### new动态分配和初始化对象
在自由空间分配的内存是无名的，所以new无法为其分配的对象命名，而是返回一个指向该对象的指针。
默认情况下，动态分配的对象是默认初始化的，这意味着内置类型或组合类型的对象的值是未定义的，类类型用默认构造函数初始化
```c++
// 默认初始化
int *pi = new int; //pi指向一个动态分配的、未初始化的无名对象
string *ps = new string;  //初始化为空string

// 直接初始化/列表初始化
int *pi = new int(1024); 
string *ps = new string(3, '9'); //初始化为"999"
vector<int> *pv = new vector<int>{0,1,2,3}; //列表初始化

// 值初始化
string *ps = new string(); //值初始化未空string，和默认初始化一样，都是调用构造函数
int *pi = new int(); //！！！和默认初始化不同，int值初始化结果为0，即*pi为0
```

**动态分配const对象**
合法
```c++
const int *pci = new const int(1024); //分配并初始化一个const int
const string *str = new const string; //分配并默认初始化一个const的空string
```

**内存耗尽**
内存不能分配所要求的内存空间，会抛出bad_alloc的异常，可以使用**定位new（pacement new）** 的方式阻止抛出异常。
```c++
int *p2 = new (nothrow) int;
```

#### delete释放动态内存
delete销毁给定的指针指向的对象，释放对应的内存。

```c++
delete p； //p必须指向一个动态分配的对象或是一个空指针
 
const int *pc = new const int(1024); 
delete pci;  //const对象不能改变，但可以销毁
```

释放一块非new分配的内存，或将相同的指针值释放多次，其行为是未定义的。

#### 空悬指针
空悬指针：delete之后的指针，指向一块曾经保存数据对象但现在已经无效的内存。

delete一个指针后，指针值就变为无效了，但很多机器上指针仍然保存着（已经释放了的）动态内存的地址。

空悬指针有未初始化指针的所有缺点，有一种方法可**避免空悬指针**的问题：
在指针即将要离开其作用域之前释放掉它所关联的内存。这样，指针关联的内存被释放掉之后，就没有机会继续使用指针了。如果我们需要保留指针，可以在 delete 之后将 nullptr 赋予指针，这样就清楚地指出指针不指向任何对象。

但是这只能保护这一个指针，对其他任何仍指向（已释放）内存的指针没有作用。
```c++
int *p(new int(42));
auto q = p; //p和q指向相同的内存
delete p;  //释放这块内存，p和q均变为无效
p = nullptr; //指出p不再指向任何对象，但是q呢？
```

## 【C++11】智能指针
**类似vector，智能指针也是模板。**

**智能指针（smart pointer）：自动释放所指向的对象**
`shared_ptr`：允许多个`shared_ptr`指针指向同一个对象，一旦最后一个shared_ptr被销毁，对象就会被释放。

`unique_ptr`：一对一，独占一个对象，当`unique_ptr`被销毁时，它所指向的对象也被销毁。

`weak_ptr`：一种弱引用。指向一个`shared_ptr`管理的对象，不控制所指向对象的生存期，不会改变`shared_ptr`的引用计数。一旦最后一个`shared_ptr`被销毁，对象就会被释放，即使有`weak_ptr`指向该对象。

> [!warning] 注意：智能指针也不能乱用
>
>优先使用 `unique_ptr`，因为它有一个较低的开销，但如果你需要在对象之间共享，不能使用 `unique_ptr` 的时候，就使用 `shared_ptr`
>
>智能指针可以提供对动态分配的内存安全而又方便的管理，但这建立在正确使用的前提下。
>
**为了正确使用智能指针，我们必须坚持一些基本规范：**
>1. 不使用相同的内置指针值初始化（或reset)多个智能指针。
>2. 不delete get()返回的指针。
>3. 不使用get()初始化或reset另一个智能指针。
>4. 如果你使用get()返回的指针，记住当最后一个对应的智能指针销毁后，你的指针就变为无效了。
>6. 如果你使用智能指针管理的资源不是new分配的内存，记住传递给它一个删除器（deleter)。

### 操作
#### 通用操作
![[Pasted image 20230214204519.png]]

#### shared_ptr独有操作
![[Pasted image 20230214204528.png]]
**定义和改变shared_ptr的方法**
![[Pasted image 20230213233552.png]]
![[Pasted image 20230213233645.png]]

> [!NOTE] 删除器（deleter）
> 上图中参数d更换为一个自定义的释放智能指针的函数，这个函数就是删除器

#### unique_ptr独的操作
![[Pasted image 20230214204404.png]]

#### weak_ptr独有操作
![[Pasted image 20230214211500.png]]

### shared_ptr类

```c++
//默认初始化，空指针
shared_ptr<string> p1;  //指向string
shared_ptr<list<int>> p2;  //指向元素类型为int的list
```

#### make_shared函数
最安全的分配和使用动态内存的方法
```c++
//指向一个值为42的int的shared ptr
shared_ptr<int> p3 = make_shared<int>(42);

//指向一个值为"9999999999"的string
shared_ptr<string> p4 = make_shared<string>(10,'9');

//指向一个值初始化的int 
shared_ptr<int> p5 = make_shared<int>();

//通常使用auto简写
auto p6 = make_shared<vector<string>>();
```

#### shared_ptr的拷贝和赋值
当进行拷贝或赋值操作时，每个share_ptr都会记录有多少个其他shared_ptr指向相同的对象：
```c++
auto p = make_shared<int>(42): //p指向的对象只有p一个引用者
auto q(p);  //p和g指向相同对象，此对象有两个引用者
```

每个`shared_ptr`指向的对象都有一个关联的计数器，通常称其为**引用计数(reference count)**。无论何时我们拷贝一个指向该对象的shared_ptr，计数器都会递增。

**计数器递增：**
- 用一个shared_ptr初始化另一个shared_ptr
- 将shared_ptr作为参数传递给一个函数，函数调用实参会拷贝一次
- shared_ptr作为函数的返回值时

**计数器递减：**
- 给shared_ptr赋予一个新值
- shared_ptr被销毁（例如一个局部的shared ptr离开其作用域）

**一旦计数器变为0，它就会自动释放自己所管理的对象：**
```c++
auto r=make_shared<int>(42);//r指向的int只有一个引用者
r=q;
//给r赋值，令它指向另一个地址
//递增q指向的对象的引用计数
//递减r原来指向的对象的引用计数
//r原来指向的对象已没有引用者，会自动释放
```

**shared_ptr自动销毁所管理的对象**
通过**析构函数**实现，析构函数控制对象销毁时做什么操作。
**shared_ptr自动释放相关联的内存**

> [!warning] 容器存放动态指针
> 如果你将shared_ptr存放于一个容器中，而后不再需要全部元素，而只使用其中一部分，要记得用erase删除不再需要的shared_ptr元素。

#### shared_ptr和new结合使用

```c++
//可以用new返回的指针来初始化智能指针
shared_ptr<double>pl; //shared_ptr可以指向一个double
shared_ptr<int>p2 (new int(42));  //p2指向一个值为42的int
```

接受指针参数的**智能指针构造函数是 explicit（显式）的**，禁止隐式转换，只能进行直接初始化，不能拷贝初始化。

```c++
shared_ptr<int> p1 = new int(1024);  //错误：不能将一个内置指针隐式转换为一个智能指针，不能拷贝初始化
shared_ptr<int> p2(new int (1024));  //正确：使用了直接初始化形式
```

相同的原因，一个返回shared_ptr的函数不能在其返回语句中隐式转换一个普通指针：
```c++
shared_ptr<int> clone(int p)
{
    return new int(p); //错误：隐式转换为shared_ptr<int>
}

shared_ptr<int> clone(int p)
{
    return shared_ptr<int>(new int (p));  
    //正确：显式地用int*创建shared_ptr<int>
}
```

默认情况下，一个用来初始化智能指针的普通指针必须指向动态内存，因为智能指针默认使用delete释放它所关联的对象。
我们可以将智能指针绑定到一个指向其他类型的资源的指针上，但是为了这样做，必须提供自己的操作来替代delete，使用第二个谓词。

> [!bug] 警告
> 不要混合使用普通指针和智能指针，上述结合使用会引发错误，建议使用make_shared而不是new
> 
> 不要使用get初始化另一个智能指针或为智能指针赋值，get返回一个内置指针指向智能指针管理的对象

#### 其他shared_ptr操作
reset将一个新的指针赋予一个shared_ptr：
```c++
p = new int (1024);  //错误：不能将一个指针赋予shared_ptr
p.reset(new int(1024));  //正确：p指向一个新对象
```

reset会更新引用计数，经常与unique一起使用，来控制多个shared_ptr共享的对象。
```c++
if (!p.unique())
		p.reset(new string(*p));  //我们不是对象的唯一用户；分配新的拷贝
*p += newVal;  //现在我们知道自己是新对象唯一的用户，可以放心改变对象的值
```

### unique_ptr类
#### 初始化
接受指针参数的**智能指针构造函数是explicit（显示）的**，禁止隐式转换，只能进行直接初始化，不能拷贝初始化。

当我们定义一个 `unique_ptr` 时，需要将其绑定到一个 new 返回的指针上 
```c++
unique_ptr<double> p1;  //可以指向一个double的unique ptr
unique_ptr<int> p2(new int(42));  //p2指向一个值为42的int

//make_unique返回一个unique_ptr在C++14引入，C++11 不支持。这种方式更安全，原理和make_shared类似
unique_ptr<int> i = make_unique<int>();
```
#### 拷贝和赋值
由于独占对象，所以unique_ptr不支持普通的拷贝或赋值
```c++
unique_ptr<string> p1(new string("Stegosaurus")); 
unique_ptr<string> p2(p1);  //错误：unique_ptr不支持拷贝

unique_ptr<string> p3;
p3 = p2;  //错误：unique_ptr不支持赋值
```

例外：我们可以拷贝或赋值一个将要被销毁的unique_ptr

```c++
//1. 从函数返回一个unique_ptr
unique ptr<int>clone(int p)  //正确：从int*创建一个unique_ptr<int>
{  
    return unique_ptr<int>(new int (p));
}

//2. 返回一个局部对象的拷贝：
unique_ptr<int>clone(int p)
{
    unique_ptr<int>ret (new int (p));
    return ret;
}
```

#### 转移对象
通过release或reset将指针的所有权从一个（非const）unique_ptr转移给另一个unique_ptr：
```c++
//将所有权从p1(指向string Stegosaurus)转移给p2
unique_ptr<string> p2(p1.release());//release将p1置为空
unique_ptr<string> p3(new string("Trex"));

//将所有权从p3转移给p2
p2.reset(p3.release()); //reset释放了p2原来指向的内存，指向p3释放的内存
```

release返回的是`unique_ptr<string>::pointer`类型的指针，是一个`string*`类型的普通指针，需要手动释放。
```c++
p2.release();  //错误

auto o = p2.release(); //正确，记得delete
delete o;
```

### weak_ptr类
创建一个weak_ptr时，要哟个一个shared_ptr来初始化它：
```c++
auto p = make_shared<int>(42);
weak_ptr<int> wp(p);  //wp弱共享p;p的引用计数未改变
```

lock检查指向的对象是否存在，如果存在，lock返回一个指向共享对象的shared_ptr，这样可以保证访问对象是安全的。
```c++
if(shared_ptr<int> np = wp.lock())  //如果np不为空则条件成立
{
		//在if中，np与p共享对象
}
```

## 动态数组
new和delete一次都是操作一个对象
C++提供两种一次分配一个对象数组的方法

> [!warning] 
> **大多数应用应该使用标准库容器而不是动态分配的数组**。使用容器更为简单
更不容易出现内存管理错误并且可能有更好的性能。
### new/delete数组
用new分配一个动态数组，后面要跟方括号，指明分配数量，必须为整型。
new返回一个指向数组元素类型的指针，而不是数组类型
```c++
int *p = new int[1024]; //p指向第一个int

//等价，使用类型别名
typedef int arrT[1024]; 
int *p = new arrT;

delete[] a2;  //按逆序销毁动态数组中的元素

```

> [!warning] 
> 动态数组并不是数组类型，不支持begin、end、范围for循环

#### 初始化
```c++
int *p1= new int[10];  //默认初始化，10个未初始化的int
int *p2 = new int[10]();  //加（）是值初始化，10个值初始化为0的int
string *p3 = new string[10];  //10个空string
string *p4 = new string[10](); //10个空string

//C++11支持列表初始化
//10个int分别用列表中对应的初始化器初始化
int *p5 = new int[10]{0,1,2,3,4,5,6,7,8,9}:
//10个string,前4个用给定的初始化器初始化，剩余的进行值初始化
string *p6 = new string[10]{"a","an","the",string(3,'x')};
```

### unique_ptr管理动态数组
标准库提供了一个可以管理new分配的动态数组的unique_ptr版本
![[Pasted image 20230214221250.png]]
```c++
//up指向一个包含10个未初始化int的数组
unique_ptr<int[]> up(new int[10]);  //对象类型后加一个方括号
up.release(); //释放后，自动用delete[]销毁其指针
```

unique_ptr 指向数组（不是对象），不能用点和箭头运算符，但可以用下标运算符访问数组元素
```c++
for (size_t i=0;i !10;++i)
		up[i]=i;  //为每个元素赋予一个新值
```

shared_ptr若要管理动态数组，必须提供自定义的删除器
```c++
shared ptr<int>sp(new int[10], [](int *p) {delete[] p;})
sp.reset();  //使用我们提供的lambda释放数组，它使用delete[]
```
shared_ptrmei没有定义下标运算符，可以用get获取一个内置指针，用它来访问数组元素
```c++
//shared ptr未定义下标运算符，并且不支持指针的算术运算
for (size_t i = 0; i != 10; ++i)
		*(sp.get()+i)=i;  //使用get获取一个内置指针
```

### allocator类
new、delete将内存分配/释放和对象构造/销毁组合在了一起，不太灵活。
- allocator类将内存分配和对象构造分离
- allocator类也是一个模板
![[Pasted image 20230214222842.png]]
```c++
allocator<string> alloc;  //可以分配string的allocator对象
auto const p = alloc.allocate(n);  //为n个未初始化的string分配内存

auto q = p; //q指向最后构造的元素之后的位置
alloc.construct(q++);  //*q为空字符串
alloc.construct(q++,10,'c');  //*q为cccccccccc
alloc.construct(q++,"hi");  //*q为hi!

//不能使用未构造的内存
cout<< *p <<endl;  //正确：p已经分配内存，使用string的输出运算符
cout<< *g <<endl;  //错误：q指向未构造的内存！

//用完之后使用destroy销毁构造的对象
while (q != p)
		alloc.destroy(--q);  //释放我们真正构造的string

//销毁后内存可以保存其他string，或者释放内存
alloc.deallocate(p, n);
```

**拷贝和填充未初始化内存**
![[Pasted image 20230214224148.png]]

## 自定义内存分配细节
#### 重载new和delete
P726
#### 定位new表达式
P729

# 十二、重载运算与类型转换

![[Pasted image 20230219235211.png]]

> [!NOTE] Title
> 当一个重载运算符是成员函数时，this绑定到左侧运算对象。成员运算符函数的（显式）参数数量比运算对象的数量少一个。

1. 对于一个运算符函数来说，它或者是类的成员，或者至少含有一个类类型的参数：
```c++
//错误：不能为int重定义内置的运算符
int operator+(int, int);
```
这一约定意味着当运算符作用于内置类型的运算对象时，我们无法改变该运算符的含义。
2. 我们只能重载已有的运算符，不能发明新的运算符号。
3. 可以直接调用一个重载的运算符函数
```c++
//调用非成员函数operator+
datal + data2;  //普通的表达式
operator+(datal,data2);  //等价的函数调用

//调用成员函数operator+=
data1 += data2;  //基于“调用”的表达式
data1.operator+=(data2);  //对成员运算符函数的等价调用，将this绑定到data1地址，将data2作为实参传入了函数。
```

## 1 输入输出运算符
### 输出运算符<<重载
第一个形参是（非常量）ostream对象的引用
第二个形参是一个常量的引用，该常量是我们想要打印的类类型
```c++
ostream &operator<<(ostream &os,const Sales data &item)
{
		os << item.x << " " << item.y;
		return os;
}
```

> [!NOTE] 
> 1. 输出运算符应该负责打印对象的内容而非控制格式，输出运算符不应该打印换行符。
> 2. 自定义IO运算符，必须将其定义为非成员函数
> 3. IO运算符通常需要读写类的非公有数据成员，所以一般声明为友元

### 输入运算符>>重载
第一个形参是运算符将要读取的流的引用
第二个形参是将要读入的（非常量）对象的引用

```c++
istream &operator>>(istream &is, Sales data &item)
{
		double price;//不需要初始化，因为我们将先读入数据到price,之后才使用它
		is >> item.bookNo >> item.units sold >> price;
		if (is)
		//检查输入是否成功
		item.revenue = item.units sold price;
		else
		item = Sales_data();//输入失败：对象被赋予默认的状态
		return is;
}
```

> [!NOTE] 
> 输入运算符必须处理输入可能失败的情况，输出运算符不需要。
> 当读取操作发生错误时，输入运算符应该负责从错误中恢复。

## 2 算术和关系运算符
通常情况下定义为非成员函数以允许对左侧或右侧的运算对象进行转换。
运算符一般不需要改变运算对象的状态，所以形参都是常量的引用

### 相等运算符 == 和!=

```c++
#include <iostream>
struct Vector2
{
    ....
    bool operator==(const Vector2& other) const  //定义操作符的重载,如果！=，这里做相应修改即可
    {
        return x == other.x && y == other.y;
    }

    bool operator！=(const Vector2& other) const  //如果！=，这里做相应修改即可
    {
        return ！(*this == other);
    }
};

    ....

int main() {
    ....
    Vector2 result1 = position.Add(speed.Multiply(powerup));
    Vector2 result2 = position + speed * powerup; 

    if (result1 == result2)   //需要对==进行重载操作 （！=同理）
    {
      ....
    }
    std::cin.get();
}
```


### 关系运算符
如<
## 3 赋值运算符=

> [!NOTE] 
> 赋值运算符必须定义为成员函数
```c++
class StrVec
public:
StrVec &operator=(std::initializer list<std::string>);
//其他成员与13.5节（第465页)一致
};
```

 ### **复合赋值运算符+=**
```c++
// 作为成员的二元运算符：左侧运算对象绑定到隐式的this指针
// 假定两个对象表示的是同一本书
Sales data &Sales data:operator+=(const Sales_data &rhs)
{
		units sold +rhs.units sold;
		revenue +rhs.revenue;
		return *this;
}

```

> [!NOTE] 
> 赋值运算符必须定义成类的成员，复合赋值运算符通常情况下也应该这样做。
这两类运算符都应该返回左侧运算对象的引用。
## 4 下标运算符 []
表示容器的类通常可以通过元素在容器中的访问元素，这些类一般会定义下标运算符operator[]

> [!NOTE] 
> - 下标运算符必须是成员函数
> - 下标运算符通常重载两个版本：一个返回普通引用，另一个是类的常量成员并且返回常量引用。
>


举个例子，我们按照如下形式定义StrVec(参见13.5节，第465页)的下标运算符：
```c++
class StrVec
{
public:
		std::string& operator[](std::size_t n)
			return elements[n];
		const std::string& operator[](std::size_t n) const
			return elements[n];
		//其他成员与13.5（第465页)一致
	private:
		std:string *elements;
		//指向数组首元素的指针
}
```

上面这两个下标运算符的用法类似于vector或者数组中的下标。因为下标运算符返回的是元素的引用，所以当StrVec是非常量时，我们可以给元素赋值：而当我们对常量对象取下标时，不能为其赋值：
```c++
//假设svec是一个非const StrVec对象
const StrVec cvec = svec;  //把svec的元素拷贝到cvec中
//如果svec中含有元素，对第一个元素运行string的empty函数
if (svec.size() && svec[0].empty())
{
		svec[0] = "zero";  //正确：下标运算符返回string的引用
		cvec[0] = "Zip";  //错误：对cvec取下标返回的是常量引用
}
```

## 5 递增递减运算符

> [!NOTE] 
> 建议设定为成员函数
> 同时定义前置和后置两个版本
> - 前置运算符返回递增或递减后的原对象的引用
> - 后置运算符返回对象的原值（递增或递减之前的值）

```c++
class StrBlobptr
public:
		//前置运算符
		StrBlobPtr& operator++();  
		StrBlobPtr& operator--();
		
		//后置运算符
		StrBlobPtr operator++(int); //增加一个额外的int类型的形参，不参与运算，只是为了区分前置后置运算符，无须命名。
		StrBlobptr operator--(int);
		//其他成员和之前的版本一致
};

//前置版本：返回递增/递减对象的引用
StrBlobPtr& StrBlobPtr::operator++()
{
		//如果curr已经指向了容器的尾后位置，则无法递增它
		check(curr,"increment past end of StrBlobPtr");
		++curr;
		//将curr在当前状态下向前移动一个元素
		return *this;

StrBlobPtr& StrBlobPtr::operator--()
{
		//如果curr是0，则继续递减它将产生一个无效下标
		--curr;
		//将curr在当前状态下向后移动一个元素
		check(curr,"decrement past begin of StrBlobPtr");
		return *this;
	}
	
//后置版本：递增/递减对象的值但是返回原值
StrBlobPtr StrBlobPtr:operator++(int)
{
		//此处无须检查有效性，调用前置递增运算时才需要检查
		StrBlobptr ret = *this;  //记录当前的值
		++*this;  //向前移动一个元素，前置++需要检查递增的有效性
		return ret;  //返回之前记录的状态
}
StrBlobPtr StrBlobPtr::operator--(int)
{
		//此处无须检查有效性，调用前置递减运算时才需要检查
		StrBlobPtr ret = *this;   //记录当前的值
		--*this;  //向后移动一个元素，前置一-需要检查递减的有效性
		return ret;  //返回之前记录的状态
}
```


## 6 成员访问运算符 * 和->

> [!NOTE] Title
> 箭头运算符->必须是类的成员，解引用运算符 * 不一定
> 重载的箭头运算符必须返回类的指针或者自定义了箭头运算符的某个类的对象

```c++
class StrBlobptr
{
	public:
		std:string& operator*() const
		{
			auto p = check(curr,"dereference past end"); //检查curr是否在作用范围内，如果是则返回curr所指元素的引用
			return (*p)[curr];
			//(*p)是对象所指的vector
		}
		
		std:string*operator->() const
		{
			return this->operator*();  //将实际工作委托给解引用运算符
			//其他成员与之前的版本一致
		}
	}
```

## 7 函数对象（仿函数）与函数调用运算符()
#函数对象 #仿函数

**重载函数调用操作符的类，其对象**常称为**函数对象(function object)**，也叫**仿函数(functor)**，使得类对象可以像函数那样调用。
STL 提供的算法往往有两个版本，一种是按照我们常规默认的运算来执行，另一种允许用户自己定义一些运算或操作，通常通过回调函数或模版参数的方式来实现，此时 functor 便派上了用场，特别是作为模版参数的时候，只能传类型。



> [!NOTE] Title
> 函数调用运算符必须是成员函数。一个类可以定义多个不同版本的调用运算符，相互之间应该在参数数量或类型上有所区别。

即使 `absnum` 只是一个对象而非函数，我们也能“调用”该对象。**调用对象实际上是在运行重载的调用运算符**。在此例中，该运算符接受一个 int 值并返回其绝对值：
```c++
//下面这个名为absInt的struct含有一个调用运算符，该运算符负责返回其参数的绝对值：
struct absInt
{
    int operator()(int val) const
    {
        return val < 0 ? -val : val;
    }
}

int i = 10;
absInt absnum;  //含有函数调用运算符的对象
int j = absnum(i);  //将i传给absnum.operator()

```

函数对象常常作为泛型算法的实参，例如使用标准库for_each算法
```c++
for_each(vs.begin(),vs.end(),absInt()); //第三个实参是类型absInt的一个临时对象
```

**lambda也是一种函数对象**

**标准库定义的函数对象**
![[Pasted image 20230221223643.png]]
```c++
plus<int> intAdd;       //可执行int加法的函数对
negate<int> intNegate;  //可对int值取反的函数对象
//使用intAdd::operator(int,int)求10和20的和
int sum =  intAdd (10,20);  //等价于sum=30
sum = intNegate(intAdd(10,20));  //等价于sum=30
//使用intNegate::operator(int)生成-l0
//然后将-l0作为intAdd::operator(int,int)的第二个参数
sum = intAdd(10,intNegate(10));  //sum 0
```

**这些函数对象通常用来替换算法中的默认运算符。**
```c++
//默认下排序算法使用operator<按升序排列
//传入greater可以执行降序排列
sort (svec.begin(),svec.end(),greater<string>());
```

### 仿函数的优缺点
优点：
1）仿函数比函数指针的执行速度快，函数指针时通过地址调用，而仿函数是对运算符 operator 进行自定义来提高调用的效率。
2）仿函数比一般函数灵活，可以同时拥有两个不同的状态实体，一般函数不具备此种功能。
3）仿函数可以作为模板参数使用，因为每个仿函数都拥有自己的类型。
缺点：
1）需要单独实现一个类。
2）定义形式比较复杂。

### 仿函数作用
仿函数通常有下面四个作用：  
1）作为排序规则，在一些特殊情况下排序是不能直接使用运算符<或者>时，可以使用仿函数。  
2）作为判别式使用，即返回值为bool类型。  
3）同时拥有多种内部状态，比如返回一个值得同时并累加。  
4）作为算法 for_each 的返回值使用。


## 8 函数表与function类型
### 函数表

C++语言中有几种可调用的对象：函数、函数指针、lambda表达式、bind创建的对象以及重载了函数调用运算符的类。
和其他对象一样，可调用的对象也有类型。例如，每个lambda有它自己唯一的（末命名)类类型：函数及函数指针的类型则由其返回值类型和实参类型决定，等等。
**不同类型的可调用对象却可能共享同一种调用形式**。调用形式指明了调用返回的类型以及传递给调用的实参类型。一种调用形式对应一个函数类型，例如：`int (int,int)`是一个函数类型，它接受两个int、返回一个int。

对于几个可调用对象共享同一种调用形式的情况，有时我们会希望把它们看成具有相同的类型。例如，考虑下列不同类型的可调用对象：
```c++
//普通函数
int add(int i,int j) {return i+j;}
//lambda,其产生一个未命名的函数对象类
auto mod=[](int i,int j ){return i % j；};
//函数对象类
struct divide
{
		int operator()(int denominator,int divisor)
		{
			return denominator divisor;
		}
}
```
上面这些可调用对象分别对其参数执行了不同的算术运算，尽管它们的类型各不相同，但是共享同一种调用形式：`int (int,int)`

> [!NOTE] 函数表
> ****
我们可能希望使用这些可调用对象构建一个简单的桌面计算器。为了实现这一目的，需要定义一个**函数表**用于存储指向这些可调用对象的“指针”。当程序需要执行某个特定的操作时，从表中查找该调用的函数。

### 【C++11】function 类型
#function
常用function类型来定义函数表。
![[Pasted image 20230221224156.png]]
```c++
//在这里我们声明了一个function类型，它可以表示接受两个int、返回一个int的可调用对象。因此，我们可以用这个新声明的类型表示任意一种桌面计算器用到的类型；
function<int(int,int)>

function<int(int,int)> f1 = add;  //函数指针
function<int (int,int)> f2 = divide();  //函数对象类的对象
function<int (int,int)> f3 = [](int i,int j){ return i*j; }; //lambda

cout<<f1(4,2)<<end1:  //打印6
cout<<f2(4,2)<<end1:  //打印2
cout<<f3(4,2)<<end1;  //打印8
```
 
## 9 类型转换运算符
P514 慎用
> [!NOTE] Title
> 一个类型转换函数必须是类的成员函数，他不能声明返回类型，形参列表也必须为空。类型转换函数通常是const
> operator type() const;
# 十三、模板与泛型编程
OOP能处理类型在程序运行之前都未知的情况，**在泛型编程中，在编译时就能获知类型。**

> [!NOTE] Title
> - 函数模板和类模板成员函数的定义通常放在头文件中
> - 模板程序应该尽量减少对参数类型的要求，即追求通用性

编写泛型代码的两个重要原则：
- **模板中的函数参数是const的引用。保证了函数可以用于不能拷贝的类型**
- 函数体条件判断使用`<`可以用标准库自带的函数对象`less<Type>`代替。
## 1 模板参数
通常将模板参数设为T，也可以用任意名字

模板参数名不可重用
```c++
template <typename T, typename T>  //错误，重用T
```
### 模板声明
与函数参数相同，声明中的模板参数的名字可以和定义中不同
```c++
//3个ca1c都指向相同的函数模板
template <typename T>T calc(const T&,const T&);  //
template <typename U>U calc(const U&,const U&);  //

//模板的定义
template <typename Type>
Type calc(const Type& a,const Type& b) {/*...*/};
```
### 使用类的类型成员
使用一个模板类型参数的类型成员，必须显示告诉编译器该名字是一个类型，使用tyname关键字：
```c++
template <typename T>
typename T::value_type top(const T& c)
{
		if(!c.empty())
			return c.back();
		else
			return typename T::value_type();
}
```

> [!NOTE] 
> 当我们希望通知编译器一个名字表示类型时，必须使用关键字typename，而不能使用class
### 【C++11】默认模板实参
可以为函数模板和类模板提供默认实参
```c++
// compare有一个默认模板实参less<T>和一个默认函数实参F()
template <typename T,typename F = less<T> >
int compare(const T &v1,const T &v2,F f = F())
{
		if (f(v1,v2))  return -1;
		if (f(v2,v1))  return 1;
		return 0;
	}
```

如果一个类模板为其所有模板参数都提供了默认实参，我们使用默认实参时就必须在模板名之后跟一个**空的<>**
```c++
template<class T=int> class Numbers  //T默认为int
{
public:
		Numbers(T v = 0) : val(v) { }  //对数值的各种操作
private:
		T val;
};
Numbers<long double> lots_of_precision; //不使用默认
Numbers<> average_precision; //空<>表示我们希望使用默认类型
```

## 2 函数模板
不使用模板：
```c++
void Print(int temp) 
{
      cout << temp;
}
  void Print(string temp) 
{
      cout << temp;
}
  void Print(double temp)
{
      cout << temp;
}

int main() 
{
      Print(1);
      Print("hello");
      Print(5.5);
      //如果要用一个函数输出三个类型不同的东西，则要手动定义三个不同重载函数
      //这其实就是一种复制粘贴就可以完成的操作
}
```

**使用模板：**

格式：
**`template<typename T>`**：类型模板参数
**`template<unsigned T>`**：非类型模板参数

- 模板参数列表不能为空
- typename也可以写成class
- **typename表示类型参数**，
- **unsigned表示非类型参数，非类型模板参数的模板实参必须是常量表达式。**

```c++
template<typename T> 
void Print(const T &temp) 
{
    //把类型改成模板类型的名字如T就可以了
    cout << temp;
}

//干净简洁
int main() 
{
	   //实例化出void Print(int temp);
    Print(1);
    //实例化出void Print(string temp);
    Print("hello");
    //实例化出void Print(double temp);
    Print(5.5);
}
```

只有当我们实际调用时，编译器用推断出的模板实参代替对应的模板参数为我们**实例化**一个特定版本的函数。每一个版本称为一个**实例**。

当编译器遇到模板定义时，它并不生成代码，只有当我们实例化模板的一个特定版本时，编译器才生成代码。
### inline和constexpr的函数模板
inline或constexpr说明符放在函数参数列表之后，返回类型之前
```c++
//正确：inline说明符跟在模板参数列表之后
template <typename T> inline T min (const T&,const T&);
```

## 3 类模板

> [!NOTE] Title
> - 一个类模板的每个实例都形成一个单独的类
> - 默认情况下，对于一个实例化的类模板，成员只有在使用时才被实例化
> - 在一个类模板的作用域内，我们可以直接使用模板名而不必指定模板实参（简化书写：`Array`等价于`Array<N>`）

与函数模板不同，**编译器不能为类模板推断模板参数类型**。我们必须在模板名后的尖括号中提供额外信息，用来代替模板参数的模板实参列表。
类模板外定义成员函数要先在模板内声明
传多个规则给模板，**用逗号隔开就行**
```c++
//可以传类型，也可以传数字，功能太强大了
//两个模板参数：类型和大小
template<typename T, int size> class Array 
{
private:
    T m_Array[size]; //在栈上分配一个数组，而为了知道它的大小，要用模板传一个数字N过来
};

int main() 
{
    ////用尖括号给模板传递构造的规则
    Array<int, 5> array;  //实例化生成一个类
}
```

#### 类模板和友元
```c++
template<typename T> class BlobPtr;  //前置声明


template<typename T> class Blob
{
	friend class BlobPtr<T>;  //友元
}

Blob<char> ca; //BlobPtr<char>是ca的友元
Blob<int> ia;  //BlobPtr<int>是ia的友元
```

**【C++11】新标准中我们可以将模板类型参数声明为友元**
```c++
template<typename T> class Bar
{
		friend T;  //将访问权限授予用来实例化Bar的类型
		...
};
```
将用来实例化Bar的类型声明为友元，因此对于每个类型名`Foo`，`Foo`将成为`Bar<Foo>`的友元，`Sales_data`将成为`Bar<Sales_data>`的友元，以此类推。

#### 【C++11】模板类型别名
不能使用typedef为模板类型设置别名

可以使用using：
```c++
//将twin定义为pair<T,T>的别名
template<typename T> using twin = pair<T, T>;
twin<string> authors; //authors是一个pair<string,string>

//当我们定义一个模板类型别名时，可以固定一个或多个模板参数：
template <typename T> using partNo = pair<T,unsigned>;
partNo<string> books;  //books是一个pair<string,unsigned>
partNo<Vehicle> cars;  //cars是一个pair<Vehicle,unsigned>
partNo<Student> kids;  //kids是一个pair<Student,unsigned>
```

#### 类模板static成员
```c++
template <typename T>class Foo
{
public:
		static std::size_t count() {return ctr};  
		
private:
		static std::size_t ctr;  
}
```

## 4 成员模板
一个类（无论是普通类还是类模板）可以包含本身是模板的成员函数。这种成员被称为**成员模板**。
**成员模板不能是虚函数**
#### 普通类的成员模板

```c++
//函数对象类，对给定指针执行delete
class DebugDelete
{
public:
		DebugDelete(std::ostream &s = std::cerr) : os(s) {}
		//与任何函数模板相同，T的类型由编译器推断
		template<typename T> void operator()(T *p) const
		{
			os << "deleting unique ptr" << std::endl;
			delete p;
		}
private:
		std:ostream &os;
};

double* p = new double;
DebugDelete d;  //可像delete表达式一样使用的对象
d(p); //调用DebugDelete::operator()(double*),释放p
int* ip = new int;
DebugDelete()(ip); //在一个临时DebugDelete对象上调用operator()(int*)
```
#### 类模板的成员模板
```c++
template <typename T> class Blob
{
		template <typename It> Blob(It b,It e);
		//...
};
```

**在类模板外定义一个成员模板时**，必须同时为类模板和成员模板提供参数列表。类模板的参数列表在前，后跟成员自己的模板参数列表。
```c++
template <typename T>  //类的类型参数
template <typename It>  //构造函数的类型参数
Blob<T>::Blob(It b, It e) : data(std::make_shared<std::vector<T>(b,e)) {}
```

实例化类模板的成员模板，必须同时提供类和函数模板的实参。
```c++
int ia[] = {0,1,2,3,4,5,6,7,8,9};
vector<long> vi = {0,1,2,3,4,5,6,7,8,9}:
list<const char*> w = {"now","is","the","time"};  
//实例化Blob<int>类及其接受两个int*参数的构造函数
Blob<int> a1(begin(ia), end(ia)); 

//实例化Blob<int>类的接受两个vector<long>::iterator的构造函数
Blob<int> a2(vi.begin(), vi.end());  

//实例化Blob<string>及其接受两个list<const char*>::iterator参数的构造函数
Blob<string> a3(w.begin(), w.end());
```

## 5 【C++11】extern显式实例化
大系统中，多个文件中可能实例化相同的模板，有不必要的额外开销。
新标准中，可以通过**显式实例化**来避免这种开销。
```c++
//显式实例化格式:
//declaration是一个类或函数声明，其中所有模板参数已被替换为模板实参
extern template declaration;  //实例化声明
template declaration;  //实例化定义
```

```c++
//实例化声明与定义
extern template class Blob<string>;  //声明
template int compare(const int&, const int&); //定义
```

> [!NOTE] 实例化定义会实例化所有成员
> 实例化定义会实例化所有成员，包括内联的成员函数。因此显式实例化一个类模板的类型时，所用类型必须能用于模板的所有成员函数。

当编译器遇到extern模板声明时，它不会在本文件中生成实例化代码。将一个实例化声明为extern就表示承诺在程序其他位置有该实例化的一个非extern声明（定义）。
**对于一个给定的实例化版本，可能有多个extern声明，但必须只有一个定义。**
由于编译器在使用一个模板时自动对其实例化，因此**extern声明必须出现在任何使用此实例化版本的代码之前**：
```c++
//Application.cc
//这些模板类型必须在程序其他位置进行实例化
extern template class Blob<string>;
extern template int compare(const int&,const int&);
Blob<string> sa1,sa2;  //实例化会出现在其他位置
//B1ob<int>及其接受initializer1ist的构造函数在本文件中实例化
B1ob<int> a1={0,1,2,3,4,5,6,7,8,9}:
Blob<int> a2(a1);  //拷贝构造函数在本文件中实例化
int i=compare(a1[0],a2[0]);  //实例化出现在其他位置

//templateBuild.cc
//实例化文件必须为每个在其他文件中声明为extern的类型和函数提供一个（非extern)的定义
template int compare (const int&,const int&);
template class Blob<string>;  //实例化类模板的所有成员
```

## 6 模板实参推断
对于函数模板，编译器利用调用中的函数来确定其模板参数。从函数实参来确定模板实参的过程称为**模板实参推断**。
### 模板类型参数的类型转换

> [!NOTE] Title
> 将实参传递给带模板类型的函数形参时，能够自动应用的类型转换只有const转换及数组或函数到指针的转换。

如果一个函数形参的类型使用了模板类型参数，那么它和使用普通类型参数初始化规则是不同的。**调用时通常不会直接进行类型转换（比如将T转换为int）**，而是如前文所说，编译器用推断出的模板实参代替对应的模板参数为我们**实例化**一个特定版本的函数。
如果混用了模板类型参数和普通类型参数，则对普通类型形参对应的实参进行正常的类型转换。模板类型参数则执行上述实例化规则。

**模板类型参数有三种情况会直接进行类型转换：**
- **const转换**：可以将一个非const对象的引用（或指针）传递给一个const的引用（或指针）形参。
- **数组或函数指针转换**：如果函数形参<mark style="background: #FFB86CA6;">不是引用类型</mark>，则可以对数组或函数类型的实参应用正常的指针转换。一个数组实参可以转换为一个指向其首元素的指针。类似的，一个函数实参可以转换为一个该函数类型的指针。
- **显式指定的实参**
```c++
template<typename T> T fobj(T,T);  //实参被拷贝
template <typename T> T fref(const T&,const T&);  //引用
string s1("a value");
const string s2("another value");
fobj(s1,s2);  //调用fobj(string,string); 顶层const被忽略
fref(s1,s2);  //调用fref(const string&,const string&)
				 //将s1转换为const是允许的
int a[10], b[42];
fobj(a,b);  //调用f(int*,int*)
fref(a,b);  //错误：数组类型不匹配


//对于模板参数类型已经显式指定了的函数实参，也进行正常的类型转换：
template<typename T> int compare(const T &v1,const T &v2)
{
		if(v1<v2)  return -1;
		if(v2<v1)  return 1;
		return 0; 
}

long lng;
compare(1ng,1024);  //错误：模板参数不匹配
compare<long>(1ng,1024);  //正确：1024是int转换为long，实例化compare(1ong,long)
compare<int>(1ng,1024);  //正确：lng是long转换为int，实例化compare(int,int)
```
### 使用相同模板参数类型的函数形参
传递给想给模板参数类型的实参必须具有相同的类型
```c++
template<typename T> int compare(const T &v1,const T &v2)
{
		if(v1<v2)  return -1;
		if(v2<v1)  return 1;
		return 0; 
}

long l;
compare(l, 1024); //错误：不能实例化compare(long,int);
```

想使用不同类型就要将函数模板定义为两个类型参数
```c++
template<typename A， typename B> 
int ABcompare(const A &v1,const B &v2)
{
		if(v1<v2)  return -1;
		if(v2<v1)  return 1;
		return 0; 
}

long l;
ABcompare(l, 1024); //正确
```

### 函数模板显式实参
当函数返回类型与参数列表中任何类型都不相同时，编译器无法推断出模板实参的类型

**指定显示模板实参**
```c++
template <typename T1,typename T2,typename T3>
T1 sum(T2,T3); 
//编译器无法推断T1,它未出现在函数参数列表中
//每次调用sum时调用者都必须为T1提供一个显式模板实参
```

提供显示模板实参的方式和定义类模板实例的方式相同。显示模板实参在尖括号中给出，位于函数名之后，实参列表之前：
```c++
//T1是显式指定的，T2和T3是从函数实参类型推断而来的
auto val3 = sum<long long>(i,j);
```

### 尾置返回类型与类型转换
```c++
// 尾置返回允许我们在参数列表之后声明返回类型
template <typename It>
auto fcn(It beg, It end) -> decltype(*beg)
{
		// 处理序列
		return *beg;  // 返回序列中一个元素的引用
}
```
此例中我们通知编译器fcn的返回类型与解引用beg参数的结果类型相同。解引用运算符返回一个左值（参见4.l.1节，第121页)，因此通过decltype推断的类型为beg表示的元素的类型的引用。
因此，如果对一个string序列调用fcn,返回类型将是string&。如果是int序列，则返回类型是int&。

标准类型转换模板P606

### 函数指针的实参推断
当我们用一个函数模板初始化一个函数指针或为一个函数指针赋值时，编译器使用指针的类型来推断模板实参。
```c++
template<typename T> int compare (const T&,const T&);
//pf1指向实例int compare(const int&,const int&)
int (*pfl)(const int&,const int&) = compare;
```

显式模板实参消除func调用歧义：
```c++
//func的重载版本；每个版本接受一个不同的函数指针类型
void func(int (*)(const string&,const string&));
void func(int (*)(const int&,const int&));
func(compare);//错误：无法确定使用compare的哪个实例

//正确：显式指出实例化哪个compare版本
func(compare<int>);//compare (const int&,const int&)
```
### 引用的实参推断
```c++
template <typename T> void f(T &p); //底层const
```
#### 从左值引用函数参数推断类型
当函数参数是模板类型参数是普通（左值）引用T&时：
```c++
template<typename T>void f1(T&); //实参必须是一个左值
//对f1的调用使用实参所引用的类型作为模板参数类型

f1(i);   //i是一个int;模板参数类型T是int
f1(ci);  //ci是一个const int;模板参数T是const int
f1(5);   //错误：传递给一个&参数的实参必须是一个左值
```

当函数参数是模板类型参数是const T&时：
```c++
template<typename T> void f2(const T&); //可以接受一个右值
//f2中的参数是const &;实参中的const是无关的
//在每个调用中，f2的函数参数都被推断为const int&

f2(i);   //i是一个int;模板参数T是int
f2(ci);  //ci是一个const int,但模板参数T是int
f2(5);   //一个const &参数可以绑定到一个右值；T是int
```

#### 从右值引用函数参数推断类型
当函数参数是一个右值引用T&&时
```c++
template <typename T> void f3(T&&);
f3(42);  //实参是一个int类型的右值；模板参数T是int
```

**引用折叠和右值引用参数:**
假设 i 是一个 int 对象，则它是一个左值，C++允许我们这样调用：f3(i)
很奇怪，f3形参是右值引用类型吗，怎么能绑定到左值上呢？这是因为C++在正常绑定规则之外定义了两个例外规则，允许这种绑定：
当我们调用f3(i)时，编译器推断模板类型参数为**实参的左值引用类型**。即编译器推断T为int&而不是int。
这样f3的函数参数看起来是一个类型为int&的右值引用（int& &&）。通常我们不能直接定义一个引用的引用，但是**通过类型别名或通过模板类型参数间接定义**是可以的。
**间接创建引用的引用会发生“引用折叠”：**
● 绝大部分情况下，引用折叠成普通的左值引用类型：
			X&  &、X&  &&和X&& &都折叠成类型X&
● 【C++11】例外的，右值引用的右值引用折叠成右值引用类型：
			类型X&&  &&折叠成X&&

> [!NOTE] 引用折叠
> 1. 引用折叠只能应用于间接创建的引用的引用，如类型别名或模板参数。
> 2. 如果一个函数参数是指向模板参数类型的右值引用（如，T&&),则可以传递给它任意类型的实参。如果将一个左值传递给这样的参数，则函数参数被实例化为一个普通的左值引用(T&)。

## 7 转发
**某些函数需要将其一个或多个实参连同类型不变地转发给其他函数。** 我们需要保持被转发实参的所有性质，包括实参类型是否是const以及实参是左值还是右值。

```c++
//接受一个可调用对象和另外两个参数的模板
//对“翻转”的参数调用给定的可调用对象
//f1ip1是一个不完整的实现：顶层const和引用丢失了
template <typename F, typename T1, typename T2>
void flip1(F f,T1 t1,T2 t2)
{
	f(t2,t1);
}

//这个函数一般情况下工作得很好，但当我们希望用它调用一个接受引用参数的函数时就会出现问题：
void f(int v1,int &v2) //注意v2是一个引用
{
	cout << vl << "" << ++v2 << end1;
}

f(42,i)  //直接调用f改变了实参i
flip1(f,j,42); //通过flip1调用f不会改变j，因为j对应的是f中的v1
```

我们重写函数，，让参数能保持给定实参的“左值性”
通过**将一个函数参数定义为一个指向模板类型参数的右值引用**，我们可以保持其对应的实参的所有类型信息。而使用引用参数（无论是左值还是右值）使得我们可以保持const属性，因为在引用类型中的const是底层的。如果我们将函数参数定义为T1&&和T2&&,通过**引用折叠**就可以保持翻转实参的左值/右值属性

```c++
template <typename F,typename T1,typename T2>
void flip2(Ff,T1 &&t1,T2 &&t2)
{
		f(t2,t1);
}
```

> [!NOTE] 
> 如果一个函数参数是指向模板类型参数的右值引用（如T&&),它对应的实参的const属性和左值/右值属性将得到保持。

但是**flip2对于接受一个左值引用的函数工作得很好，但不能用于接受右值引用参数得函数**。解决这个问题就要使用std::forward
```c++
void g(int &&i, int&j)
{
		cout << i << "" << j << endl;
}

flip2(g,i,42);  //错误：不能从一个左值实例化int&&
```
### 【C++11】std::forward保持类型信息

> [!NOTE] Title
> 当用于一个指向模板参数类型的右值引用函数参数(T&&)时，forward会保持实参类型的所有细节。

forward必须通过显式模板实参来调用，forward返回该显式实参类型的右值引用，即forward< T >返回类型时T&&
通常情况下，我们**使用forward传递那些定义为模板类型参数的右值引用的函数参数**。通过其返回类型上的引用折叠，forward可以保持给定实参的左值/右值属性：

```c++
template <typename Type> intermediary(Type &&arg)
{
		finalFcn(std::forward<Type>(arg));
		...
}
```

```c++
template <typename F, typename Tl, typename T2>
void flip(F f, T1 &&t1, T2 &&t2)
{
		f(std:forward<T2>(t2), std:forward<T1>(t1));
}

flip(g,i,42);  //正确：i将以int&类型传递给g，42将以int&&类型传递给g
```

## 8 模板重载
详细P614
```c++
//名字相同的函数必须具有不同数量或类型的参数
template <typename T> string debug rep(const T &t)

template <typename T> string debug rep(T *p)
```

如果一个非函数模板与一个函数模板提供同样好的匹配，则选择非模板版本
## 9 【C++11】可变参数模板
#可变参数模板
可变参数模板**接受可变数目的参数**

可变数目的参数被称为**参数包**，存在两种参数包：
- **模板参数包**：表示0个或多个模板参数
- **函数参数包**：表示0个或多个函数参数

用一个省略号来指出一个模板参数或函数参数表示一个包。
**在一个模板参数列表中，`class…`或`typename...`指出接下来的参数表示零个或多个类型的列表；**
一个类型名后面跟一个省略号表示零个或多个给定类型的非类型参数的列表。
在函数参数列表中，如果一个参数的类型是一个模板参数包，则此参数也是一个函数参数包。例如：
```c++
//Args是一个模板参数包；rest是一个函数参数包
//ArgS表示零个或多个模板类型参数
//rest表示零个或多个函数参数
template <typename T,typename ... Args>
void foo(const T &t,const Args& ... rest);


int i=0;
double d = 3.14;
string s = "how now brown cow";

foo(i,s,42,d);  //包中有三个参数
foo(s,42,"hi");  //包中有两个参数
foo(d,s);  //包中有一个参数
foo("hi");  //空包

//实例化版本如下：
void foo(const int&,const string&,const int&,const double&);
void foo(const string&,const int&,const char[3]&);
void foo(const double&,const string&);
void foo(const char[3]&);
```

### sizeof...运算符
当我们需要知道**包中有多少元素**时，可以使用sizeof.…运算符。类似sizeof，sizeof...也返回一个常量表达式，而且不会对其实参求值：
```c++
template<typename ... Args> void g(Args ... args)
{
		cout << sizeof...(Args) << end1;//类型参数的数目
		cout << sizeof...(args) << endl;//函数参数的数目
}
```

### 包扩展
扩展一个包时，我们要提供用于每个扩展元素的**模式**。
扩展一个包就是将它分解为构成的元素，对每个元素应用**模式**，获得扩展后的列表。
```c++
template <typename T,typename...Args>
ostream& print (ostream &os,const T &t,const Args&...rest)
//扩展Args
{
		os << t << ",";
		return print(os, rest...);  //扩展rest，为print调用生成参数列表
}
```

扩展Args：为print生成函数参数列表。编译器将模式const Args&引用到模板参数包Args中的每个元素。此模式的扩展结果为0个或多个const type&类型的列表。
```c++
print(cout,i,s,42); //包中有两个参数:s和42

//最后两个实参的类型和模式一起确定了尾置参数的类型。此调用被实例化为：
ostream&
print (ostream&,const int&,const string&,const int&);
```

扩展rest：为print调用生成参数列表。在此情况下，模式是函数参数包的名字（即rest)。此模式扩展出一个由包中元素组成的的列表。因此，这个调用等价于：
```c++
print (os,s,42);
```

### 转发参数包
可变参数函数通常将它们的参数转发给其他函数。
```c++
// fun有零个或多个参数，每个参数都是一个模板参数类型的右值引用
template<typename ... Args>
void fun(Args&& ... args)//将Args扩展为一个右值引用的列表
{
		//work的实参既扩展Args又扩展args
		work(std::forward<Args>(args)...);
}
```
这里我们希望将fun的所有实参转发给另一个名为work的函数，假定由它完成函数的实际工作。work调用中的扩展既扩展了模板参数包也扩展了函数参数包。
由于fun的参数是右值引用，因此我们可以传递给它任意类型的实参；由于我们使用std::forward传递这些实参，因此它们的所有类型信息在调用work时都会得到保持。
## 10 模板特例化
一个特例化版本就是模板的一个独立的定义，在其中一个或多个模板参数被指定为特定的类型。
当我们特例化一个函数模板时，**必须为原模板中的每个模板参数都提供实参**。为了指出我们正在实例化一个模板，应使用关键字 template 后跟一个空尖括号对(`< >`)。空尖括号指出我们将为原模板的所有模板参数提供实参。

理解此特例化版本的困难之处是函数参数类型。当我们定义一个特例化版本时，函数参数类型必须与一个先前声明的模板中对应的类型匹配。本例中我们特例化：
```c++
template <typename T> int compare(const T&,const T&);
```
我们希望定义此函数的一个特例化版本，其中T为const char*。我们的函数要求一个指向此类型const版本的引用。我们需要在特例化版本中使用的类型是const char* const&,即一个指向const char的const指针的引用。
```c++
//compare的特殊版本，处理字符数组的指针
template <>
int compare(const char* const &p1,const char* const &p2)
{
		return strcmp(p1,p2);
}

//函数调用时，匹配一致情况下的优先级别：非模板版本>特例化模板>普通模板
compare ("hi","mom");  //调用特例化版本
```

> [!NOTE] 
> 1. 特例化的本质是实例化一个模板，而非重载它。因此，特例化不影响函数匹配。
> 2. 模板及其特例化版本应该声明在同一个头文件中。所有同名模板的声明应该放在前面，然后是这些模板的特例化版本。
> 3. 类模板可以部分特例化，可以只指定一部分而非所有模板参数。还可以只特例化成员而不是类。

# 十四、命名空间
大型程序使用多个库，难免命名冲突，**命名空间**就是防止名字冲突的机制。
**命名空间分割了全局命名空间，其中每个命名空间是一个作用域**。通过在某个命名空间中定义库的名字，库的作者（以及用户）可以避免全局名字固有的限制。
## 命名空间定义
命名空间的定义包含两部分：关键字namespace和命名空间的名字
只要能出现在全局作用域中的声明就能置于命名空间内，主要包括：类、变量（及其初始化操作）、函数（及其定义）、模板和其他命名空间

```c++
namespace cplusplus_primer
{
		class Sales_data { /*...*/ };
		Sales_data operator+(const Sales_data&, const Sales_data&);
		
		class Query {/*...*/}
		class Query base {/*...*/)
}//命名空间结束后无须分号，这一点与块类似

//在命名空间之外的代码必须指出所属命名空间
cplusplus_primer::Query q = AddisonWesley::Query("Hello");

//命名空间可以是不连续的
namespace cplusplus_primer
{
		//可以为cplusplus_primer添加新的成员
}
```

### 全局命名空间
全局命名空间以隐式的方式声明，并且在所有程序中都存在。全局作用域中定义的名字被隐式的添加到全局命名空间中。
```c++
::member_name  //表示全局命名空间中的一个成员
```
### 嵌套命名空间
顾名思义，在命名空间中定义命名空间
```c++
namespace cplusplus_primer
{
		//嵌套的命名空间：定义了库的Query部分
		namespace QueryLib
		{
			class Query /*...*/)
		}
		
}

//调用方法
cplusplus_primer::QueryLib::Query
```

### 【C++11】内联命名空间
和普通的嵌套命名空间不同，**内联命名空间可以被外层命名空间直接使用**。也就是说，我们无需在内联命名空间的名字前添加表示该命名空间的前缀，通过外层命名空间就可以直接访问。
```c++
namespace cplusplus_primer
{
		//嵌套的命名空间：定义了库的Query部分
		inline namespace QueryLib
		{
			class Query /*...*/)
		}	

		//关键字inline必须出现在命名空间第一次定义的地方，后续再打开命名空间的时候可以写inline,也可以不写：
		namespace QueryLib //隐式内联
		{
			//其他声明
		}
}

//调用方法
cplusplus_primer::Query
```
### 未命名的命名空间
**未命名的命名空间中定义的变量拥有静态生命周期**：它们在第一次使用前创建，并且直到程序结束才销毁。
一个未命名的命名空间可以在某个给定的文件内不连续，但是不能跨越多个文件。
每个文件定义自己的未命名的命名空间，如果两个文件都含有未命名的命名空间，则这两个空间互相无关。在这两个未命名的命名空间中可以定义相同的名字，并且这些定义表示的是不同实体。
**如果一个头文件定义了未命名的命名空间，则该命名空间中定义的名字将在每个包含了该头文件的文件中对应不同实体。**

> [!NOTE] 
> 和其他命名空间不同，未命名的命名空间仅在特定的文件内部有效，其作用范围不会横跨多个不同的文件。

定义在未命名的命名空间中的名字可以直接使用，毕竟我们找不到什么命名空间的名字来限定它们。同样的，我们也不能对未命名的命名空间的成员使用作用域运算符。
未命名的命名空间中定义的名字的作用域与该命名空间所在的作用域相同。如果未命名的命名空间定义在文件的最外层作用域中，则该命名空间中的名字一定要与全局作用域中的名字有所区别：
```c++
int i;  //i的全局声明
namespace
{
		int i;
	}
// 二义性：i的定义既出现在全局作用域中，又出现在未嵌套的未命名的命名空间中。
i = 10;
```

其他情况下，未命名的命名空间中的成员都属于正确的程序实体。

一个未命名的命名空间也能嵌套在其他命名空间当中。此时，未命名的命名空间中的成员可以通过外层命名空间的名字来访问：

```c++
namespace local
{
		namespace
		{
			int i;
		}
	}
//正确：定义在嵌套的未命名的命名空间中的i与全局作用域中的i不同
1ocal::1=42;
```
## 命名空间成员
### 命名空间的别名
```c++
namespace primer = cplusplus_primer; //primer作为别名，更短更方便使用
namespace Qlib = cplusplus_primer::QueryLib; //别名也可以指向一个嵌套的命名空间
Qlib::Query q;
```

> [!NOTE] 
> 一个命名空间可以有好几个同义词或别名，所有别名都与命名空间原来的名字等价。

### using声明
前文我们通过`cplusplus_primer::Query`这样的方式调用成员，更简便的方法使用命名空间的成员：**using声明**。

一条**using声明**语句一次只引入命名空间的一个成员。

一条using声明语句可以出现在全局作用域、局部作用域、命名空间作用域以及类的作用域中。在类的作用域中，这样的声明语句只能指向基类成员。

```c++
using std::cin; //有了using声明后续就无需使用前缀::了
using std::cout; //每个名字都需要独立的using声明

int main()
{
		int i;
		cin>>i;  //正确：cin和std::cin含义相同
}
```

> [!warning] 
> 头文件不应包含using声明，因为头文件的内容会拷贝到所有引用它的文件中去,会造成名字冲突。
### using指示（不要使用）
和using声明不同，using指示使得某个特定命名空间中所有的名字都可见。
```c++
using namespace 命名空间的名字;

//常见的
using namespace std; 
```
using指示可以出现在全局作用域、局部作用域和命名空间作用域中，但是不能出现在类的作用域中。

## 重载与命名空间
P708
# 十五、标准库特殊设施
## 【C++11】tuple元组类型
#tuple
![[Pasted image 20230226231649.png]]
-   tuple是类似pair的类型，可以简单地**保存类型不同的任意数量的对象**。
-   tuple也可以进行比较运算，但是必须元素数量相同时才能比较
-   tuple的**常见用途是从一个函数返回多个值**，类似于在外部定义一个struct，但是更加方便且低耦合

### 定义和初始化tuple
```c++
//当我们定义一个tuple时，需要指出每个成员的类型：
tuple<size_t, size_t, size_t> threeD; //默认初始化，三个成员都设置为0
tuple<string, vector<double>, int, list<int>>
someVa1("constants",{3.14,2.718},42,{0,1,2,3,4,5})
```

**tuple的构造函数是explicit的**，因此必须使用直接初始化语法：
```c++
tuple<size_t, size_t, size_t> threeD = (1,2,3);  //错误
tuple<size_t, size_t, size_t> threeD{1,2,3};  //正确
```
类似make_pair函数，标准库定义了make_tuple函数，我们还可以用它来生成tuple对象：
```c++
//表示书店交易记录的tuple,包含：ISBN、数量和每册书的价格
auto item = make_tup1e("0-999-78345-X", 3, 20.00);
```
make_tuple函数使用初始值的类型来推断tuple的类型。在本例中，item是一个tuple,类型为`tuple<const char*,int,double>`。

### 访问tuple的成员
tuple的成员都是未命名的（没有first、second），需要使用`get`标准库函数模板访问。
```c++
// 使用get,我们必须指定一个显式模板实参，它指出我们想要访问第几个成员。
// 我们传递给get一个tuple对象，它返回指定成员的引用：
auto book = get<0>(item);  //返回item的第一个成员
auto cnt = get<1>(item);  //返回item的第二个成员
auto price = get<2>(item)/cnt;  //返回item的最后一个成员
get<2>(item) *= 0.8;  //打折20号
```

## bitset类型
#bitset
-   bitset类型可以很好地处理位运算问题，比直接使用位操作符清晰方便很多
-   bitset类似array，定义的时候模板参数是这个bitset的位数
![[Pasted image 20230226231850.png]]
![[Pasted image 20230226231958.png]]
## 正则表达式
略
## 随机数
旧版c和C++使用**rand函数**生成随机数，此函数生成均匀分布的伪随机数，每个随机数的范围在0和一个系统相关最大值（至少为32767之间）。
有很多程序需要不同范围的随机数，对rand进行定制会有很多问题，对此C++11引入了**随机数引擎类(random-number engines)和随机数分布类(random-number distribution)。**
一个引擎类可以生成unsigned随机数序列，一个分布类使用一个引擎类生成指定类型的、在给定范围内的、服从特定概率分布的随机数。
![[Pasted image 20230226232655.png]]

> [!tip] 
> C++程序不应该使用库函数rand,而应使用default_random_engine类和恰当的分布类对象。

### 随机数引擎和分布
标准库定义了多个随机数引擎类P783，这里只介绍最常用的`default_random_engine`类。
![[Pasted image 20230226234230.png]]
随机数引擎是函数对象类（参见14.8节，第506页），它们定义了一个调用运算符，该运算符不接受参数并返回一个随机unsigned整数。我们可以通过调用一个随机数引擎对象来生成原始随机数：
```c++
default_random_engine e;  //生成随机无符号数
for (size_t i = 0; i < 10; ++i)
		//e()“调用”对象来生成下一个随机数
		cout << e() << "";
```

为了得到一个指定范围内的数，我们使用一个**分布**类型的对象
```c++
//生成0到9之间（包含）均匀分布的随机数
uniform int distribution<unsigned> u(0,9);
default_random_engine e;//生成无符号随机整数
for (size_t i = 0; i = 10; ++i)
		//将u作为随机数源
		//每个调用返回在指定范围内并服从均匀分布的值
		cout<<u(e)<<""; //得到0到9之间（包含）的数
```
此处我们将u定义为`uniform int distribution<unsigned>`。此类型生成均匀分布的unsigned值。当我们定义一个这种类型的对象时，可以提供想要的最小值和最大值。
分布类型也是函数对象类。分布类型定义了一个调用运算符，它接受一个随机数引擎作为参数。分布对象使用它的引擎参数生成随机数，并将其映射到指定的分布。

> [!NOTE] 
> 1. 当我们说**随机数发生器**时，是指分布对象和引擎对象的组合。
> 2. 一个给定的随机数发生器一直会生成相同的随机数序列。一个函数如果定义了局部的随机数发生器，应该将其（包括引擎和分布对象）定义为static的。否则，每次调用函数都会生成相同的序列。

### 随机种子
随机数发生器会生成相同的随机数序列，我们可以提供一个**随机种子**来生成不同的随即结果。种子就是一个数值，引擎可以利用它从序列中一个新位置重新生成随机数。
为引擎设置种子有两种方式：在创建引擎对象时提供种子，或者调用引擎的seed成员：
```c++
default_random_engine el;  // 使用默认种子
default_random_engine e2(2147483646);  // 使用给定的种子值

// e3和e4将生成相同的序列，因为它们使用了相同的种子
default_random_engine e3;  //使用默认种子值
e3.seed(32767);  //调用seed设置一个新种子值
default_random_engine e4(32767);  //将种子值设置为32767
for(size t i=0;i!=100;++1)
{
		if(e1()==e2())
			cout <"unseeded match at iteration:"<i<<endl;
		if(e3()！=e4())
			cout <"seeded differs at iteration:"<i<<endl;
}
```

通常调用系统函数time作为种子。由于time返回以秒计的时间，因此这种方式只适用于生成种子的间隔为秒级或更长的应用。
```c++
default_random_engine el(time(0));//稍微随机些的种子
```

> [!warning] 
> 如果程序作为一个自动过程的一部分反复运行，将time的返回值作为种子的方式就无效了；它可能多次使用的都是相同的种子。

### 其他随机数分布
P665

# 十六、异常处理
C++中异常处理包括：
throw表达式
try语句块
异常类

入门：p172
深入：P684




