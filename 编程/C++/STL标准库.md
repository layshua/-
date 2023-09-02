
---
title: STL标准库
aliases: []
tags: []
create_time: 2023-08-31 11:03
uid: 202308311103
banner: "![[Pasted image 20230831110348.png]]"
---

# 一、通用容器操作
以下操作基本适用于所有容器

![[Pasted image 20230211213826.png]]
![[Pasted image 20230211214254.png]]
>size ()的返回值为 size_type 类型，比如 string. size ()返回值为 string:: size_type 类型，值得一提的是 string[x]的下标 x 也为 string:: size_type 类型。

## 容器定义和初始化
![[Pasted image 20230211220558.png|650]]
#### 将一个容器初始化为另一个容器的拷贝
将一个容器初始化为另一个容器的拷贝的方法有两种：
1. 直接拷贝整个容器（两容器类型及元素类型必须相同）
2. 拷贝迭代器指定的元素范围（不要求类型相同，只要能进行类型转换即可，我们称之为**相容**）
```c++
//每个容器有三个元素，用给定的初始化器进行初始化
list<string> authors = {"Milton" , "Shakespeare", "Austen"};
vector<const char*> articles= { "a", "an", "the" };

//直接拷贝
list<string> list2(authors) ;//正确:类型匹配
deque<string> authList(authors); //错误:容器类型不匹配
//迭代器指定
forward_list<string> words(articles.begin(), articles.end()); //正确:可以将const char*元素转换为string
```
#### **特别的 `array`**
- 定义一个 array 时，除了指定元素类型，还要指定容器大小
```c++
array<int, 10>
```
- 此外，其他容器默认构造都是空容器，而 array 默认构造是非空的：它包含了与其大小一样多的默认初始化元素。
- 无法对内置数组类型进行拷贝或对象赋值，但 array 可以

> [!NOTE] 关键概念：容器元素是拷贝
> 当我们用一个对象来初始化容器时，或将一个对象插入到容器中时，实际上放入到容器中的是对象值的一个拷贝，而不是对象本身。
> 容器中的元素与提供值的对象之间没有任何关联。随后对容器中元素的任何改变都不会影响到原始对象，反之亦然。

## swap 和 assign
![[Pasted image 20230211221941.png]]
assign 允许我们从一个**不同但相容**的类型赋值，或者从容器的一个子序列赋值。assign 操作用参数指定的元素的拷贝替换左边容器的所有元素。
## 迭代器
迭代器实现了**智能指针**功能，它们用于指向容器中的元素，对容器中的元素进行访问和遍历。

在 STL 中，迭代器是作为类模板来实现的（在头文件 `iterator` 中定义），它们可分为以下几种类型：

### 根据访问修改权限分类

- 输出迭代器（output iterator，记为：**OutIt**）
    - 可以修改它所指向的容器元素
    - 间接访问操作（`*`）
    - `++` 操作
- 输入迭代器（input iterator，记为：**InIt**）
    - 只能读取它所指向的容器元素
    - 间接访问操作（`*`）和元素成员间接访问（`->`）
    - `++`、`==`、`!=` 操作。

### 根据迭代方式分类

- 前向迭代器（forward iterator，记为：**FwdIt**）
    - 可以读取/修改它所指向的容器元素
    - 元素间接访问操作（`*`）和元素成员间接访问操作（`->`）
    - `++`、`==`、`!=` 操作
- 双向迭代器（bidirectional iterator，记为：**BidIt**）
    - 可以读取/修改它所指向的容器元素
    - 元素间接访问操作（`*`）和元素成员间接访问操作（`->`）
    - `++`、`--`、`==`、`!=` 操作
- 随机访问迭代器（random-access iterator，记为：**RanIt**）
    - 可以读取/修改它所指向的容器元素
    - 元素间接访问操作（`*`）、元素成员间接访问操作（`->`）和下标访问元素操作（`[]`）
    - `++`、`--`、`+`、`-`、`+=`、`-=`、`==`、`!=`、`<`、`>`、`<=`、`>=` 操作

除了这些基本迭代器外，STL 还提供了一些**迭代器适配器**，可以生成对应的迭代器 [[#泛型迭代器]]，用于一些特殊的操作。
- 插入迭代器
- 反向迭代器
- 流迭代器
### 迭代器遍历
**迭代器正序遍历**
```c++
for (vector<T>::iterator it = v.begin(); it != v.end(); it++)
{
    cout << *it << endl;
}
/**
 * 1. 迭代器的声明方式:  容器类型::迭代器类型
 * 2. 顺序首尾迭代器由begin()和end()方法生成
*/
```

**迭代器逆序遍历**
```c++
for (vector<T>::reverse_iterator it = v.rbegin(); it != v.rend(); it++)
{
    cout << *it << endl;
}
/**
  * 1. 逆向迭代器不再是iterator，而是reverse_iterator
  * 2. 逆序首位迭代器由rbegin()和rend()方法生成
*/
```


### 各容器的迭代器类型

- 对于 `vector`、`deque` 以及 `basic_string` 容器类，与它们关联的迭代器类型为**随机访问迭代器（RanIt）**
- 对于 `list`、`map/multimap` 以及 `set/multiset` 容器类，与它们关联的迭代器类型为**双向迭代器（BidIt）**

`queue`、`stack` 和 `priority_queue` 容器类，不支持迭代器！

**判断迭代器是否能随机访问的方法**
用多了自然就背上了，下面给出一种现场测试的方法。
```c++
iterator++；
iterator--；
//通过编译，至少是双向迭代器
  
iterator = iterator + 1；
//通过编译，则是随机访问迭代器
```

### 迭代器之间的相融关系

![[cefca8bcc1571137315093623ec613f2_MD5.png]]

迭代器之间的相融关系
**在需要箭头左边迭代器的地方可以用箭头右边的迭代器去替代。**

## 迭代器相关
`distance`：返回两个迭代器之间的距离。

使用这个函数，能完成迭代器与下标之间的转换：下标 = 当前迭代器位置-容器头部
```c++
//结合begin（）和返回的迭代值进行求下标的计算。
auto iter  = find(nums.begin(), nums.end(), target);
if(iter!=nums.end())
{
    return distance(nums.begin(),iter); //返回target值在数组中的下标
}
```

# 二、顺序容器
**容器**用于表示由同类型元素构成的、长度可变的元素序列。
容器是由类模板来实现的，模板的参数是容器中元素的类型。

-  `vector`：**可变大小数组**。支持快速随机访问。在尾部之外的位置插入或删除元素可能很慢
-  `array`：**固定大小数组**。支持快速随机访问。不能添加或删除元素
-  `string`：与 vector 相似的容器，但专门用于保存字符。随机访问快。在尾部插入/删除速度快
-  `deque`：**双端队列**。支持快速随机访问。在头尾位置插入删除速度很快
-  `list`：**双向链表**。只支持双向顺序访问。在 list 中任何位置进行插入/删除操作速度都很快
 -  `forward_list`：**单向链表**。只支持单向顺序访问。在链表任何位置进行插入/删除操作速度都很快

## 顺序容器操作
顺序容器特有的操作
### 添加元素
- 这些操作会改变容器的大小; array 不支持这些操作。
- forward_list 有自己专有版本的 insert 和 emplace;
- forward_list 不支持 push_back 和 emplace_back。
- vector 和 string 不支持 push_front 和 emplace_front。

```c++
c.push_back (t)       //在c的尾部创建一个值为t或由args 创建的元素。返回void
c.emplace_back(args)

c.push_front(t)       //在c的头部创建一个值为t或由args创建的元素。返回void
c.emplace_front (args)

c.insert (p,t)        //在迭代器p指向的元素之前创建一个值为t或由args创建的元素。返回指向新添加的元素的迭代器
c.emplace(p, args)

c.insert(p,n,t)       //在迭代器p指向的元素之前插入n个值为t的元素。返回指向新添加的第一个元素的迭代器;若n为0，则返回p

c.insert (p,b,e)      //将迭代器b和e指定的范围内的元素插入到迭代器p指向的元素之前。b和e不能指向c中的元素。返回指向新添加的第-个元素的迭代器;若范围为空，则返回p

c.insert(p,il)        //il是一个花括号包围的元素值列表。将这些给定值插入到迭代器p指向的元素之前。返回指向新添加的第一个元素的迭代器;若列表为空，则返回p
```

> [!warning] 
> 向一个 vector ，string 或 deque 插入元素会使所有指向容器的迭代器、引用和指针失效。

【C++11】**接受元素个数或范围的 insert 版本**返回指向第一个新加入元素的迭代器。如果范围为空，不插入任何元素，insert 操作会将第一个参数返回。

【C++11】**`emplace_back_front / emplace / emplace_back`**
- 和 insert 效果相同，但 emplace 操作不是拷贝而是**构造元素**（调用元素类型的构造函数）。
- 传递给 emplace 函数的参数必须与元素类型的构造函数相匹配。

### 访问元素
- at 和下标操作只适用于 string、vector、 deque 和 array
- back 不适用于 forward list。
```c++
c.back ()  //返回c中尾元素的引用。若c为空，函数行为未定义
c.front () //返回c中首元素的引用。若c为空，函数行为未定义
c[n]       //返回c中下标为n的元素的引用,n是一个无符号整数。若n>=c.size(),则函数行为未定义
c.at (n)   //返回下标为n的元素的引用。如果下标越界，则抛出一 out_of_range异常
```

- 在容器中**访问成员函数返回的都是引用**，如果容器是一个 const 对象，则返回值是 const 引用。
- 对一个空容器调用 front 和 back，就像使用一个越界的下标一样，是一种严重的程序设计错误。
- `at` 能够检测下标是否合法，防止越界。如果下标越界，at 方法内部会抛出异常（`exception`），可以使用 `try-catch` 捕获并处理该异常。示例如下：
```c++ fold file:捕获异常
#include <stdexception>  //标准异常头文件
#incldue <iostream>
using namespace std;

int main()
{
    string s = "hello world";
    try
    {
        //s[100]不会抛出异常，程序会直接挂掉
        s.at(100);
    }
    catch (out_of_range& e) 
        //如果不熟悉异常类型，可以使用多态特性， catch(exception& e)。
    {
        cout << e.what() << endl; 
        //打印异常信息
    }
    return 0;
}
```

### 删除元素
- 这些操作会改变容器的大小，所以不适用于 array。
- forward_list 有特殊版本的 erase
- forward_list 不支持 pop_back;
-  vector 和 string 不支持 pop_front。

```c++
c.pop_back ( ) //删除c中尾元素。若c为空，则函数行为未定义。函数返回void
c.pop_front () //删除c中首元素。若c为空，则函数行为未定义。函数返回void

c.erase (p)    //删除迭代器 p所指定的元素，返回一个指向被删元素之后元素的迭代器，若p指向尾元素，则返回尾后（off-the-end）迭代器。若p是尾后迭代器，则函数行为未定义

c.erase (b,e)  //删除迭代器b和e所指定范围内的元素。返回一个指向最后一个被删元素之后元素的迭代器，若e本身就是尾后迭代器，则函数也返回尾后迭代器

c.clear ( )    //删除c中的所有元素。返回void

```

> [!warning] Title
> 删除 deque 中除首尾位置之外的任何元素都会使所有迭代器、引用和指针失效。指向 vector 或 string 中删除点之后位置的迭代器、引用和指针都会失效。


### 改变容器的大小
- `resize` 不适用于 array

```c++
c.resize(n)      //调整 c 的大小为 n 个元素。若 n<c.size()，则多出的元素被丢弃。若必须添加新元素，对新元素进行值初始化
c.resize(n, t)  //调整 c 的大小为 n 个元素。任何新添加的元素都初始化为值 t
```

> [!warning] 
> 如果 resize 缩小容器，则指向被删除元素的迭代器、引用和指针都会失效; 对 vector、string 或 deque 进行 resize 可能导致迭代器、指针和引用失效。

### 容器容量管理

> [!NOTE] size 和 capacity 的区别
> -  `size` 是指容器已经保存的元素的数目；
>-  `capacity` 则是在不分配新的内存空间的前提下它最多可以保存多少元素。即容器的容量
>- `resize` 成员函数只改变容器中元素的数目，而不是容器的容量。

-  `shrink_to_fit` 只适用于 vector、 string 和 deque。
- `capacity` 和 `reserve` 只适用于 vector 和 string。

```c++
c.shrink_to_fit() //请将capacity ()减少为与size ()相同大小
c.capacity()      //不重新分配内存空间的话，c可以保存多少元素
c.reserve(n)      //分配至少能容纳n个元素的内存空间
```

只有当需要的内存空间超过当前容量时，`reserve` 调用才会改变 vector 的容量。如果需求大小大于当前容量，`reserve` 至少分配与需求一样大的内存空间（可能更大）。

如果需求大小小于或等于当前容量，`reserve` 什么也不做。特别是，当需求大小小于当前容量时，容器不会退回内存空间。因此，在调用 `reserve` 之后，capacity 将会大于或等于传递给 `reserve` 的参数。

调用 `reserve` 永远也不会减少容器占用的内存空间。类似的，**`resize` 成员函数（参见 9.3.5 节，第 314 页）只改变容器中元素的数目，而不是容器的容量**。我们同样不能使用 resize 来减少容器预留的内存空间。


**巧用 swap 来收缩空间：**
[C++ vector容器的swap方法（容器互换）_vector的swap_对的时间点的博客-CSDN博客](https://blog.csdn.net/qq_41929943/article/details/103190891)
```c++
vector<int>(v).swap(v);
// vector<int>(v): 利用拷贝构造函数初始化匿名对象
// swap(v): 交换的本质其实只是互换指向内存的指针
// 匿名对象指针指向的内存会由系统自动释放
```

## 顺序容器迭代器

```c++
//通常使用auto简写迭代器类型
auto b = v.begin(); // begin()返回指向第一个元素的迭代器
auto c = v.end(); // end()返回指向尾元素的下一位置的迭代器（尾后迭代器）
```

> [!NOTE] 
> 特殊的，如果容器为空，则 begin 和 end 返回的是同一个迭代器，都是尾后迭代器。
> 
迭代器主要有 iterator（本质是 `T*`）和 const_iterator（本质是常量指针 `const T*`）两种类型

```c++
//迭代器类型
vector<int>::iterator it;  //it能读写vector<int>的元素
vector<int>::const_iterator it; //只能读元素，不能写元素

string<int>::iterator it;  //it能读写vector<string>的元素
string<int>::const_iterator it; //只能读元素，不能写元素
```

**如果 vector 对象或 string 对象是一个常量，那么只能使用 const_iterator；
如果不是常量，那么两种迭代器都可以使用。**

### begin 和 end 成员
begin 和 end 有多个版本
rbegin：返回反向迭代器 (reverse_iterator)
cbegin：返回 const 迭代器（const_iterator）

### 运算
![[Pasted image 20230209204934.png]]
forward_list 的迭代器不支持递减运算符`--`

**以下运算只能应用于 string、vector、deque 和 array 的迭代器**
![[Pasted image 20230209205951.png]]
**迭代器相减的结果是两个迭代器的距离**，类型是名为 `difference_type` 的**带符号整型数**。

### 容器操作导致的迭代器失效
**在向容器添加元素后：**
1. 如果容器是 vector 或 string, 且存储空间被重新分配，则指向容器的迭代器、指针和引用都会失效。如果存储空间未重新分配，指向插入位置之前的元素的迭代器、指针和引用仍有效，但指向插入位置之后元素的迭代器、指针和引用将会失效。
2. 对于 deque, 插入到除首尾位置之外的任何位置都会导致迭代器、指针和引用失效。如果在首尾位置添加元素，迭代器会失效，但指向存在的元素的引用和指针不会失效。
3. 对于 list 和 forward_list, 指向容器的迭代器（包括尾后迭代器和首前迭代器)、指针和引用仍有效。

当我们从一个容器中删除元素后，指向被删除元素的迭代器、指针和引用会失效，这应该不会令人惊讶。毕竟，这些元素都已经被销毁了。

**当我们删除一个元素后：**
1.  对于 vector 和 string, 指向被删元素之前元素的迭代器、引用和指针仍有效。注意：当我们删除元素时，尾后迭代器总是会失效。
2. 对于 deque, 如果在首尾之外的任何位置删除元素，那么指向被删除元素外其他元素的迭代器、引用或指针也会失效。如果是删除 deque 的尾元素，则尾后迭代器也会失效，但其他迭代器、引用和指针不受影响：如果是删除首元素，这些也不会受影响。
3. 对于 list 和 forward_list, 指向容器其他位置的迭代器（包括尾后迭代器和首前迭代器)、引用和指针仍有效。

> [!command] 建议：管理迭代器
> 当你使用迭代器（或指向容器元素的引用或指针）时，**最小化要求迭代器必须保持有效**的程序片段是一个好的方法。由于向迭代器添加元素和从迭代器删除元素的代码可能会使迭代器失效，因此必须保证每次改变容器的操作之后都正确地重新定位迭代器。
> 这个建议对 vector、string 和 deque 尤为重要。
> 

> [!Warning] warning
> 凡是使用了迭代器的循环体，都不要向迭代器所属的容器添加元素
> 
> 如果在一个循环中插入/删除 deque、string 或 vector 中的元素，不要缓存 end 返回的迭代器。
## 【array】 固定数组

- 当**创建 array** 时就要**初始化其大小**，不可再改变。
- array 和原生数组都是**创建在栈上**的（vector 是在堆上创建底层数据储存的）
- 原生数组越界的时候不会报错，而 **array 会有越界检查**，会报错提醒。
- 使用 std:: array 的好处是可以**访问它的大小**（通过 s**ize ()** 函数），它是一个**类**。

```c++
#include<iostream>
#include<array>

void PrintArray(const std::array<int, 5>& data)  //显式指定了大小 
{
    for (int i = 0;i < data.size();i++)  //访问数组大小
    {
        std::cout << data[i] << std::endl;
    }
}

int main() {
    std::array<int, 5> data; //定义，有两个参数，一个指定类型，一个指定大小
    data[0] = 0;
    data[1] = 1;
    data[2] = 2;
    data[3] = 3;
    data[4] = 4;
    PrintArray(data);
    std::cin.get();
}
```

## 【vector】 可变数组
vector 提供的是**随机访问迭代器（Random Access Iterator）**，其内部用普通指针实现。
### 初始化
![[Pasted image 20230209195935.png]]
### 操作
![[Pasted image 20230209200056.png]]

> [!bug] 下标操作
> vector 对象的下标类型仍是对应的 size_type，和 string 一样，只能对已存在的元素执行下标操作，否则会造成缓冲区溢出，在运行时产生一个不可预估的值。
> 
> **确保下标合法的一种有效手段就是尽可能使用范围 for 语句。**

### 内存空间增长策略
vector 本质上是一个动态数组, 内存连续存储。

**Vector 对象是如何增长的？**
假定容器中元素是连续存储的，且容器的大小是可变的，考虑向 vector 或 string 中添加元素会发生什么：如果没有空间容纳新元素，容器不可能简单地将它添加到内存中其他位置，因为元素必须连续存储。容器必须分配新的内存空间来保存已有元素和新元素，将已有元素从旧位置移动到新空间中，然后添加新元素，释放旧存储空间。
如果我们每添加一个新元素，vector 就执行一次这样的内存分配和释放操作，性能会慢到不可接受。
为了避免这种代价，标准库实现者采用了可以减少容器空间重新分配次数的策略。**当不得不获取新的内存空间时，vector 和 string 的实现通常会分配比新的空间需求更大的内存空间。容器预留这些空间作为备用，可用来保存更多的新元素。这样，就不需要每次添加新元素都重新分配容器的内存空间了。**

vs 编辑器每次扩容 1.5 倍：[C++vector的动态扩容，为何是1.5倍或者是2倍](https://blog.csdn.net/qq_44918090/article/details/120583540)

### vector 使用优化

vecctor 的优化策略：

**问题 1：** 当向 vector 数组中**添加新元素**时，为了扩充容量，**当前的 vector 的内容会从内存中的旧位置复制到内存中的新位置** (产生一次复制)，然后删除旧位置的内存。简单说，push_back 时，容量不够，会自动调整大小，重新分配内存。这就是将代码拖慢的原因之一。
**解决办法：** vertices.reserve (n) ，直接指定容量大小，避免重复分配产生的复制浪费。  

**问题 2：** 在非 vector 内存中创建对象进行初始化时，即 push_back () 向容器尾部添加元素时，首先会创建一个临时容器对象（不在已经分配好内存的 vector 中）并对其追加元素，然后再将这个对象拷贝或者移动到【我们真正想添加元素的容器】中。这其中，就造成了一次复制浪费。
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

## 【string】 字符串
标准库类型 String 表示可变长的字符序列

- `char*` 是一个指针，`string` 是一个类
    `string` 封装了 `char*`，管理这个字符串，是一个 `char*` 型的容器。

> `string` 本质上是一个动态的 char 数组。

- 不用考虑内存释放和越界
    `string` 管理 `char*` 所分配的内存，每一次 `string` 的复制/赋值，取值都由 `string` 类负责维护，不用担心复制越界和取值越界等。

### 1 初始化
![[Pasted image 20230209190416.png]]

**除了顺序容器通用的操作外，string 类型还提供了一些额外的操作。**
- 提供 string 类和 C 风格字符串之间的相互转换
- 增加了允许我们用下标代替迭代器的版本

![[Pasted image 20230212153613.png]]
```c++
//string 转 const char*
string str = "demo";
const char* cstr = str.c_str();

//const char* 转 string
const char* cstr = "demo";
string str(cstr); // 本质上其实是一个有参构造
```


### 2 substr 子串
返回一个 string，他是原始 string 的一部分或全部的拷贝，可以传递给 substr 一个可选的开始位置和计数值：
```c++
//substr(开始位置，计数值)
string s("hello world");
string s2 = s.substr(0,5);  //s2=he11o
string s3 = s.substr(6);  //s3=world
string s4 = s.substr(6,11);  //s3=world
string s5 = s.substr(12);  //抛出一个out_of_range异常
```

### 3 修改
- `insert` 和 `erase` 有多个版本：
    - 接收迭代器
    - **接收下标**
    - **接受 C 风格字符数组**

```c++
string& insert(int pos, const char* s); // 在pos位置插入C风格字符数组
string& insert(int pos, const string& str); // 在pos位置插入字符串str
string& insert(int pos, int n, char c); // 在pos位置插入n个字符c

string& erase(int pos, int n = npos); // 删除从pos位置开始的n个字符，默认一直删除到末尾。
```

string 类定义了两个额外的成员函数： `append` 和 `repalce`
```c++
string& append(const char* s); 
// 把C风格字符数组s连接到当前字符串结尾
string& append(const char* s, int n); 
// 把C风格字符数组s的前n个字符连接到当前字符串结尾
string& append(const string &s); 
// 将字符串s追加到当前字符串末尾
string& append(const string&s, int pos, int n); 
// 把字符串s中从pos开始的n个字符连接到当前字符串结尾
string& append(int n, char c); 
// 在当前字符串结尾添加n个字符c
```

```c++
string& replace(int pos, int n, const string& str); 
// 替换从pos开始n个字符为字符串s
string& replace(int pos, int n, const char* s);
// 替换从pos开始的n个字符为字符串s
```

![[Pasted image 20230212154457.png]] ![[Pasted image 20230212154543.png]]
### 4 搜索
![[Pasted image 20230212154657.png]]
![[Pasted image 20230212154705.png]]

**`find`** 
```c++
int find(const string& str, int pos = 0) const; 
// 查找str在当前字符串中第一次出现的位置，从pos开始查找，pos默认为0
int find(const char* s, int n = 0) const; 
// 查找C风格字符串s在当前字符串中第一次出现的位置，从pos开始查找，pos默认为0
int find(const char* s, int pos, int n) const; 
// 从pos位置查找s的前n个字符在当前字符串中第一次出现的位置
int find(const char c, int pos = 0) const; 
// 查找字符c第一次出现的位置，从pos开始查找，pos默认为0
```
当查找失败时，find 方法会返回-1，-1 已经被封装为 string 的静态成员常量 `string::npos`。
`static const size_t nops = -1;`

**`rfind`**
```c++
int rfind(const string& str, int pos = npos) const; 
// 从pos开始向左查找最后一次出现的位置，pos默认为npos
int rfind(const char* s, int pos = npos) const; 
// 查找s最后一次出现的位置，从pos开始向左查找，pos默认为npos
int rfind(const char* s, int pos, int n) const; 
// 从pos开始向左查找s的前n个字符最后一次出现的位置
int rfind(const char c, int pos = npos) const; 
// 查找字符c最后一次出现的位置
```
find 方法通常查找字串第一次出现的位置，而 rfind 方法通常**查找字串最后一次出现的位置。**
rfind (str, pos)的实际的开始位置是 pos + str. size ()，即从该位置开始（不包括该位置字符）向前寻找匹配项，如果有则返回字符串位置，如果没有返回 string:: npos。
-1 其实是 size_t 类的最大值（学过补码的同学应该不难理解），所以 string:: npos 还可以表示“直到字符串结束”，这样的话 rfind 中 pos 的默认参数是不是就不难理解啦？

![[Pasted image 20230209190604.png]]

### 5 compare 比较
**比较规则：**
如果两个 string 对象在某些写对应的位置上不一致，则 string 对象比较的结果其实是 string 对象中第一对相异字符比较的结果
```c++
string a = "Hello";
string b = "Hello World"
string c = "Hiya"
```
a < b
c > a 且 c > b

![[Pasted image 20230212155018.png]]

```c++
int compare(const string& s) const; // 与字符串s比较
int compare(const char* s) const; // 与C风格字符数组比较
```
compare 函数**依据字典序比较**，在当前字符串比给定字符串小时返回-1，在当前字符串比给定字符串大时返回 1，相等时返回 0。

### 6 getline 函数
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
### 8 字面值和 string 相加
当 string 对象和字符字面值以及字符串字面值混在一条语句中使用时，要确保每个+运算符两侧的运算对象至少有一个是 string：
```c++
string s1 = "Hello"
string s2 = s1 + "World"; //正确，string+字面值 
string s3 = "hello" + "world"; //错误，两个运算对象都不是string 
string s4 =  "world" + "!" + s1; //错误，不能把字面值直接相加
```
### 9 类型转换
![[Pasted image 20230212155134.png]]
#### 【C++11】to_string
c++11 标准新增了全局函数 `std::to_string`，十分强大，可以将**任何算术类型**变成 string 类型。

> [!note] 算术类型
> 即 int，char，bool ... 这些类型

```c++
#include <string>
using namespace std;

/** 带符号整数转换成字符串 */
string to_string(int val);
string to_string(long val);
string to_string(long long val);

/** 无符号整数转换成字符串 */
string to_string(unsigned val);
string to_string(unsigned long val);
string to_string(unsigned long long val);

/** 实数转换成字符串 */
string to_string(float val);
string to_string(double val);
string to_string(long double val);
```
#### stoX
将 string 转换为其他类型X
```c++
#include <cstdlib>
#include <string>
using namespace std;

/** 字符串转带符号整数 */
int stoi(const string& str, size_t* idx = 0, int base = 10);
long stol(const string& str, size_t* idx = 0, int base = 10);
long long stoll(const string& str, size_t* idx = 0, int base = 10);

/**
  * 1. idx返回字符串中第一个非数字的位置，即数值部分的结束位置
  * 2. base为进制
  * 3. 该组函数会自动保留负号和自动去掉前导0
 */

/** 字符串转无符号整数 */
unsigned long stoul(const string& str, size_t* idx = 0, int base = 10);
unsigned long long stoull(const string& str, size_t* idx = 0, int base = 10);

/** 字符串转实数 */
float stof(const string& str, size_t* idx = 0);
double stod(const string& str, size_t* idx = 0);
long double stold(const string& str, size_t* idx = 0);
//与之类似的在同一个库里的还有一组基于字符数组的函数如下。
// 'a' means array, since it is array-based. 

int atoi(const char* str); // 'i' means  int
long atol(const char* str); // 'l' means long
long long atoll(const char* str); // 'll' means long long

double atof(const char* str); // 'f' means double
```

> [!NOTE] 
> 如果 string 不能转换为一个数值，这些函数抛出一个 invalid_argument 异常（参见 5.6 节，第 173 页）。如果转换得到的数值无法用任何类型来表示，则抛出一个 out_of_range 异常。

### 10 处理字符
![[Pasted image 20230209193035.png]]

**大小写转换**
```c++
int tolower(int c); // 如果字符c是大写字母，则返回其小写形式，否则返回本身
int toupper(int c); // 如果字符c是小写字母，则返回其大写形式，否则返回本身
```

> [!comment] 建议
> 建议：使用 C++版本的 C 标准库头文件
>
>C++标准库中除了定义 C+语言特有的功能外，也兼容了 C 语言的标准库。C 语言的头文件形如 name. h, C++则将这些文件命名为 cname。也就是去掉了. h 后缀，而在文件名 name 之前添加了字母 c, 这里的 c 表示这是一个属于 C 语言标准库的头文件。
>
>因此，cctype 头文件和 ctype. h 头文件的内容是一样的，只不过从命名规范上来讲更符合 C++语言的要求。特别的，在名为 cname 的头文件中定义的名字从属于命名空间 std, 而定义在名为. h 的头文件中的则不然。
>
>一般来说，C++程序应该使用名为 cname 的头文件而不使用 name. h 的形式。

## 【list】 双向链表
**list 不仅仅是一个双向链表，而且是一个循环的双向链表。**

**list 有一个重要的性质，插入和删除操作都不会造成原有 list 迭代器的失效**。这在 vector 是不成立的，因为 vector 的插入操作可能会造成内存的重新配置，导致原有的迭代器全部失效
**而 list 元素的删除只会使得被删除元素的迭代器失效**
> [!NOTE] 
> 对于 list 和 forward_list，应该**优先使用成员函数版本**的算法而不是通用算法。
>
链表特有的操作会改变底层的容器

链表是一种物理存储单元上非连、续非顺序的储存结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。
- 链表由一系列结点（链表中每一个元素称为结点）组成
- 结点可以在运行时动态生成
- 每个结点包括两个部分
    - 储存数据元素的数据域
    - 储存下一个结点地址的指针域
-   链表灵活，但是空间和时间的额外消耗会比较大。


相较于 vector 的连续线性空间，list 就显得复杂许多。
- 它的好处是每次插入或者删除一个元素，就是配置或者释放一个元素的空间
- 因此，list 对于空间的运用有绝对的精准，一点也不浪费
- 而且，list 对于任何位置插入或删除元素都是常数项时间

![[Pasted image 20230825104451.jpg]]

 ![[Pasted image 20230212235051.png]]
![[Pasted image 20230212235106.png]]

特有的 `splice` 成员函数
![[Pasted image 20230212235202.png]]

## 【forward_list】单向链表

![[Pasted image 20230211232259.png]]

## 【deque】双端队列
vector 容器是单向开口的连续内存空间，deque 则是一种双向开口的连续线性空间。
所谓的双向开口，意思是可以在头尾两端分别做元素的插入和删除操作
vector 虽然也能在头尾插入元素，但是在头部插入元素的效率很低，需要大量进行移位操作
![[Pasted image 20230825103846.jpg]]
![[Pasted image 20230825103917.jpg]]
deque **允许使用常数项时间在头部插入或删除元素**
deque 没有容量的概念，因为它是由动态的分段连续空间组合而成，随时可以增加一块新的空间并链接起来
虽然 deque 也提供了 Random Access Iterator，但其实现相比于 vector 要复杂得多，所以**需要随机访问的时候最好还是用 vector**。
### 双端插入删除
```c++
push_back(T elem); // 在容器尾部添加一个元素
push_front(T elem); // 在容器头部插入一个元素
​
pop_back(); // 删除容器最后一个数据
pop_front(); // 删除容器第一个数据
```


## 顺序容器适配器
容器、迭代器和函数都有**适配器（adaptor）**，标准库定义了三个顺序容器适配器：
**`stack`** 栈适配器
**`queue`** 队列适配器
**`priority_queue`**

> [!NOTE] 适配器
> #适配器
> **本质上，一个适配器是一种机制，能使某种事物的行为看起来像另外一种事物一样。一个容器适配器接受一种己有的容器类型，使其行为看起来像一种不同的类型。**
> 
> 例如，stack 适配器接受一个顺序容器（除 array 或 forward_list 外），并使其操作起来像一个 stack 一样。

![[Pasted image 20230212163058.png]]
### 定义适配器
每个适配器定义两个构造函数：
1. 默认构造函数创建空对象
2. 接收一个容器的构造函数拷贝该容器来初始化适配器

```c++
stack<int> intStack; //空栈
stack<int> intStack(deq); //接受一个容器的构造函数，拷贝该容器来初始化适配器，假设deq是一个deque<int>
```
**默认情况**下，stack 和 queue 是基于 deque 实现的，priority_queue 是在 vector 之上实现的。
我们可以在创建一个适配器时**将一个命名的顺序容器作为第二个类型参数**，来**重载默认容器类型**。
```c++
//在vector上实现的空栈
stack<string,vector<string>> str_stk;
//str stk2在vector上实现，初始化时保存svec的拷贝
stack<string,vector<string>> str_stk2(svec);
```

每个容器适配器都基于底层容器类型的操作定义了自己的特殊操作，**我们只能使用适配器操作，而不能使用底层容器类型的操作！**

> [!warning] 重载类型限制
> 所有适配器都要求容器具有添加和删除元素的能力。**因此，适配器不能构造在 array 之上。**
> 
类似的，我们**也不能用 forward_list 来构造适配器**，因为所有适配器都要求容器具有添加、删除以及访问尾元素的能力。
>
>- **`stack`** 只要求 push_back、pop_back 和 back 操作，因此**可以使用除 array 和 forward list 之外的任何容器类型来构造 stack。默认基于 deque 实现**
>
>- **`queue` 适配器**要求 back、push_back、front 和 push_front, 因此它可以构造于 list 或 deque 之上，但**不能基于 vector 构造**。**默认基于 deque 实现**
>
>- **`priority_queue`** 除了 front、push_back 和 pop_back 操作之外还要求随机访问能力，因此它**可以构造于 vector 或 deque 之上，但不能基于 list 构造。**默认基于 vector 实现。
>

## 【stack】 栈适配器
- 先进后出
- 没有迭代器，不能遍历

![[Pasted image 20230825104239.jpg|300]]
```c++
s.pop () //删除栈顶元素，但不返回该元素值

s.push (item) //创建一个新元素压入栈顶，该元素通过拷贝或移动item而来，或者由args构造
s.emplace( args)

s.top () //返回栈顶元素，但不将元素弹出栈
```

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
        int value = intStack.top(); //使用栈顶值的代码
        intStack.pop(); //出栈，弹出栈顶元素
        
        return 0;
    }
}
```

## 【queue/priority_queue】 队列适配器 
- 先进先出
- 没有迭代器，不能遍历
- `priority_queue` 允许我们为队列中的元素建立**优先级**，新加入的元素会排在所有优先级比他低的已有元素之前。
![[Pasted image 20230825104341.jpg]]
```c++
q.pop ()   //返回【queue的首元素】或【priority_queue的最高优先级的元素】,但不删除此元素
q.front () //返回首元素，但不删除此元素
q.back ()  //返回尾元素【只适用于queue】
q.top ()   //返回最高优先级元素，但不删除该元素【只适用于priority_queue】

q.push (item)     //在queue末尾或priority_queue 中恰当的位置创建一个元素，其值为item，或者由 args构造
q.emplace (args ) 

```

# 三、关联容器
- 关联容器中的元素是按关键字来保存和访问的
- 关联容器的迭代器都是双向的
![[Pasted image 20230213162157.png]]
- 有序关联容器使用比较运算符 `<` 来组织元素，按关键值字典排序从小到大 
- 无序关联容器使用哈希函数（hash function）和关键字类型 `==` 运算符来组织元素 
## 1 map 映射
map 的特性是，所有的元素都会根据元素的键值自动排序；
map 的所有元素都是 `pair`，同时拥有实值和键值。
- pair 的第一元素被视为键值，第二元素被视为实值；
- map 不允许两个元素有相同的键值；
**和 set 类似的原因，我们不能通过迭代器改变 map 的键值，但我们可以任意修改实值。**
map 和 list 在增删元素的时候具有相似的性质。
map 和 multimap 的操作类似，唯一的区别是 multimap 键值可重复。
map 和 multimap 都是以**红黑树**作为底层实现机制。
map 和 multimap 包含在同一个头文件中。

```c++
//初始化
map<string, size_t> word_count; //空容器
map<string,string> m = { {"a","b"},
							{"c","d"},
							{"e","f"} };
```

**使用 map：**
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
## 2 set 集合
set 的特性是，所有的容器都会根据元素自身的键值进行自动被排序。

set 的元素不像 map 那样可以同时拥有实值和键值，**set 的元素既是实值又是键值**。

- set 不允许两个元素有相同的键值
- 我们不可以通过 set 的迭代器改变 set 元素的值。因为其元素值就是键值，任意改变会严重破坏 set 的组织
- 换句话说，set 的 iterator 是一种 const_iterator

```c++
//初始化
set<string> s = {"a","b","c"};
```

**使用 set：**
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
find 调用返回一个迭代器，如果给定关键字在 set 中，迭代器指向该关键字，否则，find 返回尾后迭代器。

### multiset

- multiset 特性及用法和 set 完全相同，唯一的差别在于它允许键值重复。
- set 和 multiset 的底层实现是**红黑树**，红黑树为平衡二叉树的一种

> 注意，multiset 和 set 共用一个头文件。


## 3 pair 类型
定义在头文件 utility 中
- 一个 pair 保存两个数据成员，必须提供两个类型名
- pair 的数据成员是 public 的，两个成员分别命名为 first，second
```c++
//初始化
pair<string,size_t> p; //pair的默认构造函数对数据成员进行值初始化，一个是空string，一个是0
pair<string,size_t> p{"a",2};

pair<T1, T2> p(k, v);

pair<T1, T2> p = make_pair(k, v);

//使用
cout << p.first << p.second << endl;
```

![[Pasted image 20230213172051.png]]

【C++11】**可以对返回值进行列表初始化
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

在较早的 C++版本中，不允许用花括号包围的初始化器来返回 pair 这种类型的对象，
必须显式构造返回值：
```c++
if (!v.empty ()
		return pair<string,int>(v.back(),v.back().size());
```

**我们还可以用 `make_pair` 来生成 pair 对象**，pair 的两个类型来自于 make pair 的
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
解引用关联容器迭代器，得到类型为 value_type，**value_type 是一个 pair 类型，其 first 成员保存 const 关键字（const key_type），second 保存值（maped_type）**

> [!NOTE] 
> 一个 map 的 value_type 是一个 pair，可以改变 pair 的值，不能改变 pair 的关键字。
> 同样，set 的关键字也不能改变

**map 的 value_type 就是 pair 类型**，所以支持 first，second 操作
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
支持 begin 和 end

```c++
//获得一个指向首元素的迭代器
auto map_it = word_count.cbegin();
//比较当前迭代器和尾后迭代器
while (map_it != word_count.cend())
{
    //解引用迭代器，打印关键字-值对
    cout << map_it->first << "出现" < <map_it->second << "次"<< endl;
    ++map_it;//递增迭代器，移动到下一个元素
}
```

```c++
for (map<T1, T2>::iterator it = m.begin(); it != m.end(); it++)
{
    cout << "key = " << it->first << " value = " << it->second << endl;
}
```
### insert 添加元素
insert 向容器中添加一个元素或一个元素范围（begin，end）
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

### erase 删除元素
![[Pasted image 20230213195217.png]]
### map 的下标操作
![[Pasted image 20230213195731.png]]
map 下标运算符接受一个关键字，获取此关键字相关联的值，如果关键字不在 map 中就会创建一个元素插入到 map 中，值进行初始化。
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

vector, string 解引用迭代器返回的类型与下标运算符返回的类型一样，但是 map 不一样。
**map 下标操作获得 mapped_type 对象，解引用获得 value_type 对象**
### 访问元素
find：查找
count：统计数量
![[Pasted image 20230213200150.png]]
![[Pasted image 20230213200158.png]]

> [!NOTE] Title
> lower bound 返回的迭代器可能指向一个具有给定关键字的元素，但也可能
不指向。
>
**如果关键字不在容器中，lower_bound 和 upper_bound 返回相同的迭代器**（返回关键字的第一个安全插入点）一一不影响容器中元素顺序的插入位置

#### multimap 或 multiset 中查找
Multimap 和 multiset 允许关键字重复，这些元素再容器中相邻存储

给定一个作者到著作题目的映射，我们想打印一个特定作者的所有著作
方法一：使用 find 和 count：
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
upper_bound 指向给定关键字的下一个位置

方法二：
```c++
// authors 和 search_item的定义，与前面的程序一样
//beg和end对应此作者元素的范围
for(auto beg = authors.lower_bound(search_item), end = authors.upper_bound(search_item); beg!=end; ++beg)
{
		cout<< beg->second <<endl;
}
```

#### equal_range 函数
equal_range 函数 s 接受一个关键字，返回一个迭代器 pair。
若关键字存在，则第一个迭代器指向第一个与关键字匹配的元素，第二个迭代器指向最后一个匹配元素之后的位置
若未找到匹配元素，则两个迭代器都指向关键字可以插入的位置

相当于 lower_bound 和 upper_bound 的结合体。

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
unorderd：不按关键字顺序，使用**哈希函数（hash function）** 和关键字类型 `==` 运算符来组织元素
**使用有序容器相同的操作，只是不排序**

> [!Tip] 
> 如果关键字类型固有就是无序的，或者性能测试发现问题可以用哈希技术解决，就可以使用无序容器。
> 使用无序容器通常性能更好
### 管理桶
- 无序容器在存储上组织为一组**桶（bucket）**，每个桶保存零个或多个元素。
- 使用一个哈希函数将元素映射到桶。
- **为了访问一个元素，容器首先计算元素的哈希值，它指出应该搜索哪个桶**。
- 容器将具有一个特定哈希值的所有元素都保存在相同的桶中。如果容器允许重复关键字，所有具有相同关键字的元素也都会在同一个桶中。因此，无序容器的性能依赖于哈希函数的质量和桶的数量和大小。

无序容器提供了一组管理桶的函数，这些成员函数允许我们查询容器的状态以及在必要时强制容器进行重组。
![[Pasted image 20230213213719.png]]
### 对关键字类型的要求
默认情况下，无序容器使用关键字类型的 `==` 运算符来比较元素，它们还使用一个 `hash<key_type>` 类型的对象来生成每个元素的哈希值。标准库为内置类型〈包括指针)提供了 hash 模板。还为一些标准库类型，包括 string 和我们将要在第 12 章介绍的智能指针类型定义了 hash。因此，**我们可以直接定义关键字是内置类型（包括指针类型)、string 还是智能指针类型的无序容器**。
**但是，我们不能直接定义关键字类型为自定义类类型的无序容器。** 与容器不同，不能直接使用哈希模板，而**必须提供我们自己的 hash 模板版本**。我们将在 16.5 节 (第 626 页)中介绍如何做到这一点。

# 四、定制操作
**用 lambda 表达式定制操作**

###  谓词：向算法传递函数

> [!NOTE] 谓词
> **谓词是一个可调用的表达式，是返回值为 `bool` 的普通函数或者函数对象**
> 
> 标准库算法所使用的谓词分为两类：
> 一元谓词 (unary predicate, 意味着它们只接受单一参数)
> 二元谓词 (binary predicate, 意味着它们有两个参数)。
> 
> 接受谓词参数的算法对输入序列中的元素调用谓词。因此，元素类型必须能转换为谓词的参数类型。

#### 一元谓词举例

例如，对于下面的“**统计**”算法：
```c++
size_t count_if(InIt first, InIt last, Pred cond);
```

可以有如下使用方式：
```c++
#include <vector>
#include <algorithm>
#include <iostream>
using namespace std;

bool f(int x) { return x > 0; }

int main() 
{
    vector<int> v;
    ...... // 往容器中放了元素
    cout << count_if(v.begin(), v.end(), f); // 统计v中正数的个数
    return 0;
}
```

#### 二元谓词举例

例如，对于下面的“**排序**”算法：
```c++
void sort(RanIt first, RanIt last); // 按“<”排序
void sort(RanIt first, RanIt last, BinPred comp); // 按comp返回true规定的次序
```

可以有如下用法：
```c++
#include <vector>
#include <algorithm>
using namespace std;

bool greater2(int x1, int x2) { return x1 > x2; }

int main()
{
    vector<int> v;
    ...... // 往容器中放了元素
    sort(v.begin(), v.end()); // 从小到大排序
    sort(v.begin(), v.end(), greater2); // 从大到小排序
    return 0;
}
```

## 泛型迭代器
除了为每个容器定义的迭代器之外，标准库在头文件 `iterator` 中还定义了额外几种迭代器。

1. **插入迭代器 (insert iterator)**：绑定到一个容器上，可用来向容器插入元素。
3. **流迭代器 (stream iterator)**：绑定到输入或输出流上，可用来遍历所关联的 IO 流。
4. **反向迭代器 (reverse iterator)**：向后而不是向前移动。除了 forward_list 之外的标准库容器都有反向迭代器。
6. **移动迭代器 (move iterator)**：这些专用的迭代器不是拷贝其中的元素，而是移动它们。

### 插入迭代器

**插入迭代器适配器：接受一个容器，生成一个迭代器**
-  `back_inserter`
-  `front_inserter` 
-  `inserter` 

**插入迭代器：用于在容器中指定位置插入元素**
- `back_insert_iterator`（用于在尾部插入元素）
- `front_insert_iterator`（用于在首部插入元素）
- `insert_iterator`（用于在任意指定位置插入元素）

当我们通过一个插入迭代器进行赋值时，该迭代器调用容器操作来向给定容器的指定位置插入一个元素。
通常情况，当我们通过一个迭代器向容器元素赋值时，值被赋予迭代器指向的元素。而当我们通过一个插入迭代器赋值时，一个与赋值号右侧值相等的元素被添加到容器中。

![[Pasted image 20230212223317.png]]

#### back_inserter
- `back_inserter` 创建一个使用 `push_back` 的迭代器。`back_inserter` 接受一个指向容器的引用，返回一个与该容器绑定的插入迭代器。当我们通过此迭代器赋值时，赋值运算符会调用 `push_back` 将一个具有给定值的元素添加到容器中：
```c++
vector<int> vec;//空向量
auto it=back_inserter(vec);//通过它赋值会将元素添加到vec中
*it=42;//vec中现在有一个元素，值为42
```

**我们常常使用 `back_inserter` 来创建一个迭代器，作为算法的目的位置来使用。**例如：
```c++
vector<int> vec;//空向量
//正确：back_inserter创建一个插入迭代器，可用来向vec添加元素
fill_n(back_inserter(vec),10,0);//添加10个元素到vec
```

#### front_inserter
- `front_inserter` 创建一个使用 `push_front` 的迭代器。元素总是插入到容器第一个元素之前。
```c++
list<int> lst = {1,2,3,4};
list<int> lst2,list3; //空list

//拷贝完成之后，lst2包含4 3 2 1
copy(lst.cbegin(), lst.cend(), front_inserter(lst2));
//拷贝完成之后，lst2包含1 2 3 4
copy(lst.cbegin(), lst.cend(), inserter(lst3,lst3.begin()));
```

#### inserter
- `inserter` 创建一个使用 `insert` 的迭代器。此函数接受第二个参数，这个参数必须是一个指向给定容器的迭代器。元素将被插入到给定迭代器所表示的元素之前。
```c++
auto it = inserter(c,iter);
*it = val;

//等价
it = c.insert(it,val);
++it //递增it使他指向原来的元素
```

> [!NOTE] 
> 只有在容器支持 push front 的情况下，我们才可以使用 front_inserter。类似的，只有在容器支持 push_back 的情况下，我们才能使用 back_inserter。

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

通过向 sort 传递一对反向迭代器来将 vector 整理为递减序列
```c++
sort (vec.begin () , vec.end ()); //按“正常序”排序vec
sort (vec.rbegin () , vec.rend ( )); //按逆序排序:将最小元素放在vec的末尾
```

### iostream 流迭代器
iostream 类型不是容器，但是标准库也定义了用于这些 IO 对象的迭代器。**迭代器将它们对应的流当作一个特定类型的元素序列来处理**。通过使用流迭代器，**可以用泛型算法**从流对象读取数据以及写入数据。

创建流迭代器，必须指定迭代器要读写的类型。
#### istream_iterator
**可以为任何具有输入运算符（>>）的类型定义 ostream iterator 对象**。
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
**可以为任何具有输出运算符（<<）的类型定义 ostream iterator 对象**。

创建 ostream_iterator 时，我们可以**提供（可选的）第二参数，它是一个字符串，在输出每个元素后都会打印此字符串**。此字符串必须是一个 C 风格字符串（即，一个字符串字面常量或者一个指向以空字符结尾的字符数组的指针)。
必须将 ostream_iterator 绑定到一个指定的流，不允许空的或表示尾后位置的 ostream_iterator。

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

## 泛型算法结构
### 迭代器类别
![[Pasted image 20230212233213.png]]
### 算法形参模式
```c++
alg (beg, end, other args);
alg (beg, end, dest, other args);
alg (beg, end, beg2, other args);
alg (beg, end, beg2, end2, other args);
```

beg、end、dest 都是迭代器参数。除了这些参数，一些算法还接受额外的、非迭代器的特定参数。

**接受单个目标迭代器的算法**
dest 参数是一个表示算法可以写入的目的位置的迭代器。
算法假定 (assume)：按其需要写入数据，不管写入多少个元素都是安全的。

> [!NOTE] 
> 向输出迭代器写入数据的算法都假定目标空间足够容纳写入的数据。

**接受第二个输入序列的算法**
如果一个算法接受 beg2 和 end2, 这两个迭代器表示第二个范围。这类算法接受两个完整指定的范围：[ beg, end ) 表示的范围和 [ beg2end2 ) 表示的第二个范围。

只接受单独的 beg2 ( 不接受 end2 ) 的算法将 beg2 作为第二个输入范围中的首元素。此范围的结束位置未指定，这些算法假定从 beg2 开始的范围与 beg 和 end 所表示的范围至少一样大。

> [!NOTE] 
> 接受单独 beg2 的算法假定从 beg2 开始的序列与 beg 和 end 所表示的范围至少一样大。

### 算法命名规范
一些算法使用重载形式传递一个谓词
```c++
unique(beg, end);  //使用==运算符比较元素
unique(beg, end, comp)  //使用comp比较元素
```

_ if 版本的算法
接受一个元素值得算法通常都有一个接受谓词的版本 (不是重载)
```c++
find(beg,end,val);   
find_if(beg,end,lambada);
```

# 五、泛型算法

> [!Tip] 
> 标准库容器定义的操作集合很小。标准库并未给每个容器添加大量功能，而是提
供了一组算法，这些算法中的大多数都独立于任何特定的容器。这些**算法是通用的
(generic, 或称泛型的)：它们可用于不同类型的容器和不同类型的元素。**
```c++
//头文件
#include <algorithm> //包含除数值泛型算法以外的所有算法
#include <numeric> //数值泛型算法
```

> [!NOTE] 关键概念：算法永远不会执行容器的操作
> 
泛型算法本身不会执行容器的操作，它们只会运行于迭代器之上，执行迭代器的操作。这使得算法不依赖于具体的容器，提高了算法的通用性。
>
>**算法永远不会改变底层容器的大小。** 算法可能改变容器中保存的元素的值，也可能在容器内移动元素，但永远不会直接添加或删除元素。
>
当一个算法操作**插入迭代器**时，插入迭代器可以完成向容器添加元素的效果，但算法自身永远不会做这样的操作。

- **`beg` 和 `end` 是表示元素范围的迭代器。** 几乎所有算法都对一个由 beg 和 end 表示的序列进行操作。
- **`beg2` 是表示第二个输入序列开始位置的迭代器。`end2` 表示第二个序列的末尾位置（如果有的话）。**
    -  如果没有 end2, 则假定 beg2 表示的序列与 beg 和 end 表示的序列一样大。
    - beg 和 beg2 的类型不必匹配，但是，必须保证对两个序列中的元素都可以执行特定操作或调用给定的可调用对象。
- **`dest` 是表示目的序列的迭代器。** 对于给定输入序列，算法需要生成多少元素，目的序列必须保证能保存同样多的元素。
- **`unaryPred` 和 `binaryPred` 是一元和二元谓词**，分别接受一个和两个参数，都是来自输入序列的元素，两个谓词都返回可用作条件的类型。
- **`comp` 是一个二元谓词**，满足关联容器中对关键字序的要求。
- **`unaryOp` 和 `binaryOp` 是可调用对象**，可分别使用来自输入序列的一个和两个实参来调用。
![[《C++ Primer》#^6ekdwn]]
## 1 查找算法
这些算法在一个输入序列中搜索一个指定值或一个值的序列。

每个算法都提供两个重载的版本：
- 第一个版本使用底层类型的相等运算符 `==` 来比较元素;
- 第二个版本使用用户给定的 `unaryPred` 和 `binaryPred` 比较元素。
### 查找指定值
这些算法查找指定值，要求输入迭代器 ( input iterator)。

```c++
find(beg, end, val)
find_if(beg, end, unaryPred) 
find_if_not(beg, end, unaryPred) 
count(beg, end, val)
count_if(beg, end, unaryPred)
```
- **`find`** 返回一个迭代器，指向输入序列中**第一个**等于 `val` 的元素。
- **`find_if`** 返回一个迭代器，指向**第一个**满足 `unaryPred` 的元素。
- **`find_if_not`**返回一个迭代器，指向**第一个**令 `unaryPred` 为 false 的元素。
**上述三个算法在未找到元素时都返回 end。**

- **`count`** 返回一个计数器，指出 val 出现了多少次; 
- **`count_if`** 统计有多少个元素满足 `unaryPred`。

```c++
all_of(beg, end, unaryPred) 
any_of(beg, end, unaryPred) 
none_of(beg, end, unaryPred)
```
这些算法都返回一个 bool 值，分别指出 `unaryPred` 是否对所有元素都成功、对任意一个元素成功以及对所有元素都不成功。
如果序列为空：
**`any_of`** 返回 false
**`all_of`** 和 **`none_of`** 返回 true。

### 查找重复值
下面这些算法要求**前向迭代器** ( forward iterator )，在输入序列中查找重复元素。

```c++
adjacent_find (beg, end)
adjacent_find (beg, end, binaryPred)
```
**`adjacent_find`**：返回指向**第一对**相邻重复元素的迭代器。如果序列中无相邻重复元素，则返回 end。
>adjacent：邻近的

```c++
search_n(beg, end, count, val)
search_n(beg, end, count, val, binaryPred)
```
**`search_n`**：返回一个迭代器，从此位置开始有 count 个相等元素。如果序列中不存在这样的子序列，则返回 end。
### 查找子序列
在下面的算法中，除了 `find_first_of` 之外，都要求两个前向迭代器。`find_first_of` 用输入迭代器表示第一个序列，用前向迭代器表示第二个序列。这些算法搜索子序列而不是单个元素。

```c++
search (beg1, end1 , beg2 , end2)
search (beg1, end1 , beg2, end2, binaryPred)
```
**`search`**：返回第二个输入范围（子序列)在第一个输入范围中**第一次**出现的位置。如果未找到子序列，则返回 end1。

```c++
find_first_of (beg1, end1, beg2, end2)
find_first_of (beg1, end1, beg2, end2, binaryPred)
```
**`find_first_of`**：返回一个迭代器，指向第二个输入范围中任意元素在第一个范围中**首次**出现的位置。如果未找到匹配元素，则返回 end1。

```c++
find_end(beg1, end1, beg2 , end2)
find_end(beg1, end1 , beg2 , end2, binaryPred)
```
**`find_end`**：类似 search，但返回的是**最后一次**出现的位置。如果第二个输入范围为空，或者在第一个输入范围中未找到它，则返回 end1。

## 2 其他只读算法
这些算法要求前两个实参都是输入迭代器。
`equal` 和 `mismatch` 算法还接受一个额外的输入迭代器，表示第二个范围的开始位置。
这两个算法都提供两个重载的版本：
- 第一个版本使用底层类型的相等运算符 `==` 比较元素
- 第二个版本则用用户指定的 `unaryPred` 或 `binaryPred` 比较元素。

```c++
for_each(beg, end, unaryop)
```
- **`for_each`**：**对输入序列中的每个元素应用可调用对象 `unaryOp`**, 
    - `unaryOp` 的返回值被忽略。
    - 如果迭代器允许通过解引用运算符向序列中的元素写入值，则 `unaryOp` 可能修改元素。

```c++
mismatch (beg1, end1, beg2)
mismatch (beg1, end1, beg2, binaryPred)
```
- **`mismatch`**：比较两个序列中的元素。返回一个迭代器的 `pair`，表示两个序列中**第一个不匹配的元素**。
- 如果所有元素都匹配，则返回的 pair 中第一个迭代器为 end1，第二个迭代器指向 beg2 中偏移量等于第一个序列长度的位置。

```c++
equal (beg1, end1, beg2)
equal (beg1, end1 , beg2, binaryPred)
```
- **`equal`**：确定两个序列是否相等。如果输入序列中每个元素都与从 beg2 开始的序列中对应元素相等，则返回 true。

**equal 基于一个非常重要的假设：它假定第二个序列至少与第一个序列一样长。**

> [!NOTE]  关键概念：迭代器参数
**一些算法从两个序列中读取元素。构成这两个序列的元素可以来自于不同类型的容器，两个序列中元素的类型也不要求严格匹配。** 算法要求的只是能够比较两个序列中的元素。例如，对 equal 算法，元素类型不要求相同，但是我们必须能使用 `==` 来比较来自两个序列中的元素。
>
**用一个单一迭代器表示第二个序列的算法都假定第二个序列至少与第一个一样长。**

## 3 二分搜索算法
这些算法都要求前向迭代器，但这些算法都经过了优化，如果我们提供**随机访问迭代器 (random-access iterator)** 的话，它们的性能会好得多。
从技术上讲，无论我们提供什么类型的迭代器，这些算法都会执行对数次的比较操作。但是，当使用前向迭代器时，这些算法必须花费线性次数的迭代器操作来移动到序列中要比较的元素。
**这些算法要求序列中的元素已经是有序的。** 它们的行为类似关联容器的同名成员（参见 11.3.5 节，第 389 页)。`equal_range`、`lower_bound` 和 `upper_bound` 算法返回迭代器，指向给定元素在序列中的正确插入位置——插入后还能保持有序。如果给定元素比序列中的所有元素都大，则会返回尾后迭代器。

每个算法都提供两个版本: 
- 第一个版本用元素类型的小于运算符（`<`）来检测元素; 
- 第二个版本则使用给定的比较操作。在下列算法中，“x 小于 y”表示 `x<y` 或 `comp (x,y)` 成功。

```c++
lower_bound (beg, end, val)
lower_bound (beg, end, val, comp)
```
- **`lower_bound`**：返回一个迭代器，表示**第一个小于等于 val 的元素**, 如果不存在这样的元素，则返回 end。

```c++
upper_bound (beg, end, val)
upper_bound (beg, end, val, comp)
```
- **`upper_bound`**：返回一个迭代器，表示**第一个大于 val 的元素**，如果不存在这样的元素，则返回 end。

```c++
equal_range (beg, end, val)
equal_range (beg, end, val, comp)
```
- **`equal_range`**：返回一个 pair ，其 first 成员是 lower_bound 返回的迭代器，second 成员是 upper_bound 返回的迭代器。

```c++
binary_search (beg, end, val)
binary_search (beg, end, val, comp)
```
- **`binary_search`**：返回一个 bool 值，指出序列中**是否包含等于 val 的元素**。

## 4 写容器元素的算法

很多算法向给定序列中的元素写入新值。这些算法可以从不同角度加以区分: 
- 通过表示输入序列的迭代器类型来区分; 
- 通过是写入输入序列中元素还是写入给定目的位置来区分。

### 只写不读元素
这些算法要求一个**输出迭代器 (output iterator)**，表示目的位置。`_n` 结尾的版本接受第二个实参，表示写入的元素数目，并将给定数目的元素写入到目的位置中。

```c++
fill (beg, end, val)
fill_n (dest, cnt, va1) 
generate (beg, end, Gen) 
generate_n (dest, cnt, Gen)
```

**给输入序列中每个元素赋予一个新值。**
- **`fill`** ：将值 val 赋予元素; 
- **`generate`**： 执行生成器对象 `Gen ()` 生成新值。生成器是一个可调用对象，每次调用会生成一个不同的返回值。

`fill` 和 `generate` 都返回 void。
`_n` 版本返回一个迭代器，指向写入到输出序列的最后一个元素之后的位置。

```c++
fi1l(vec.begin(),vec.end(),0);//将每个元素重置为0
fill_n(vec.begin(),vec.size(),0); // 将所有元素重置为0
```
### 使用输入迭代器
这些算法读取一个输入序列，将值写入到一个输出序列中。它们要求一个名为 `dest` 的输出迭代器，而表示输入范围的迭代器必须是输入迭代器。

```c++
copy (beg, end, dest)
copy_if (beg, end, dest, unaryPred) copy_n (beg, n, dest)
```
**从输入范围将元素拷贝到 `dest` 指定的目的序列。**
- **`copy`**： 拷贝所有元素，
- **`copy_if`**： 拷贝满足 `unaryPred` 的元素
- **`copy_n`** ：拷贝前 n 个元素。输入序列必须有至少 n 个元素。

```c++
move (beg, end, dest)
```
- **`move`**： 对输入序列中的每个元素调用 `std: : move`，将其移动到迭代器 dest 开始的序列中。

```c++
transform (beg, end, dest, unaryop)
transform (beg, end, beg2, dest, binaryop)
```
- **`transform`**：**调用给定操作，并将结果写到 dest 中**。
    - 第一个版本对输入范围中每个元素应用一元操作。
    - 第二个版本对两个输入序列中的元素应用二元操作。

```c++
replace_copy (beg, end, dest, old_val, new_val)
replace_copy_if (beg, end, dest, unaryPred, new_val)
```
**将每个元素拷贝到 dest，将指定的元素替换为 new_val。**
- **`replace_copy`**：替换那些 `==old_val` 的元素。
- **`replace_copy_if`**：替换那些满足 `unaryPred` 的元素。

```c++
merge (beg1, end1, beg2, end2, dest)
merge (beg1, end1, beg2, end2, dest, comp)
```
- **`merge`**：两个输入序列必须都是**有序**的。**将合并后的序列写入到 dest 中**。
    - 第一个版本用 `<` 运算符比较元素;
    -  第二个版本则使用给定比较操作。

### 使用前向迭代器
**这些算法要求前向迭代器，由于它们是向输入序列写入元素，迭代器必须具有写入元素的权限。**

```c++
iter_swap (iter1, iter2)
swap_ranges (beg1, end1 ,beg2)
```
-  **`iter_swap`** 交换 `iter1` 和 `iter2` 所表示的元素，返回 void
-  **`swap_ranges`** 将输入范围中所有元素与 beg2 开始的第二个序列中所有元素进行交换。两个范围不能有重叠。
    - 返回递增后的 beg2，指向最后一个交换元素之后的位置。

```c++
replace (beg, end, old_val, new_val)
replace_if (beg, end, unaryPred, new_val)
```
用 `new_val` 替换每个匹配元素。

- **`replace`**：使用 `==` 比较元素与 `old_val`
- **`replace_if`**：替换那些满足 `unaryPred` 的元素。

### 使用双向迭代器的写算法
这些算法需要在序列中有反向移动的能力，因此它们要求双向迭代器。

```c++
copy_backward (beg, end, dest)
move_backward (beg, end, dest)
```
**从后往前，从输入范围中拷贝或移动元素到指定目的位置。**
与其他算法不同，**`dest` 是输出序列的尾后迭代器**（即，目的序列恰在 dest 之前结束)。
输入范围中的尾元素被拷贝或移动到目的序列的尾元素，然后是倒数第二个元素被拷贝/移动，依此类推。元素在目的序列中的顺序与在输入序列中相同。
如果范围为空，则返回值为 dest; 否则，返回值表示从`*beg` 中拷贝或移动的元素。

```c++
inplace_merge (beg, mid, end)
inplace_merge (beg, mid, end, comp)
```
**将同一个序列中的两个有序子序列合并为单一的有序序列。** beg 到 mid 间的子序列和 mid 到 end 间的子序列被合并，并被写入到原序列中。
- 第一个版本使用 `<` 比较元素
- 第二个版本使用给定的比较操作，返回 void。

## 5 划分和排序算法
每个排序和划分算法都提供**稳定（`stable_` 标记）和不稳定版本**（参见 10.3.1 节，第 345 页)。
**稳定算法保证保持相等元素的相对顺序**。由于稳定算法会做更多工作，可能比不稳定版本慢得多并消耗更多内存。
### 划分算法
**一个划分算法将输入范围中的元素划分为两组:**
- 第一组包含那些**满足给定谓词**的元素
- 第二组则包含**不满足谓词**的元素。

例如，对于一个序列中的元素，我们可以根据元素是否是奇数或者单词是否以大写字母开头等来划分它们。**这些算法都要求双向迭代器。**

```c++
is_partitioned (beg, end, unaryPred)
```
- **`is_partitioned`**：如果所有满足谓词 `unaryPred` 的元素都在不满足 `unaryPred` 的元素之前，则返回 true。若序列为空，也返回 true。

```c++
partition_copy (beg, end, dest1, dest2, unaryPred)
```
- **`partition_copy`**：将满足 `unaryPred` 的元素拷贝到 dest1, 并将不满足 `unaryPred` 的元素拷贝到 dest2。
    - 返回一个迭代器 `pair` ，其 first 成员表示拷贝到 dest1 的元素的末尾，second 表示拷贝到 dest2 的元素的末尾。
    - 输入序列与两个目的序列都不能重叠。
 
```c++
partition_point (beg, end,unaryPred)
```
`partition_point`：输入序列必须是已经用 `unaryPred` 划分过的。返回满足 `unaryPred` 的范围的尾后迭代器。如果返回的迭代器不是 end，则它指向的元素及其后的元素必须都不满足 unaryPred。

```c++
stable_partition (beg, end, unaryPred) partition (beg, end, unaryPred)
```
**`stable_partition`**：使用 `unaryPred` 划分输入序列。满足 `unaryPred` 的元素放置在序列开始，不满足的元素放在序列尾部。返回一个迭代器，指向最后一个满足 unaryPred 的元素之后的位置，如果所有元素都不满足 unaryPred，则返回 beg。

### 排序算法
这些算法要求**随机访问迭代器**。

每个排序算法都提供两个重载的版本。
- 一个版本用元素的 `<` 运算符来比较元素，
- 另一个版本接受一个额外参数来指定排序关系。

`partial_sort_copy` 返回一个指向目的位置的迭代器，其他排序算法都返回 void。
`partial_sort` 和 `nth_element` 算法都只进行部分排序工作，它们常用于不需要排序整个序列的场合。由于这些算法工作量更少, 它们通常比排序整个输入序列的算法更快。


```c++
sort (beg, end)
stable_sort (beg, end) 
sort (beg, end, comp)
stable_sort (beg, end, comp)
```
**`sort`**：**排序整个范围**

```c++
is_sorted (beg, end)
is_sorted (beg, end, comp) 
is_sorted_until (beg, end)
is_sorted_until (beg, end, comp)
```
**`is_sorted`**：返回一个 bool 值，指出整个输入序列是否有序。
**`is_sorted_until`** ：在输入序列中**查找最长初始有序子序列**，并返回子序列的尾后迭代器。

```c++
partial_sort (beg, mid, end)
partial_sort (beg, mid, end, comp)
```
**`partial_sort`**：排序 `mid-beg` 个**最小元素**。即，如果 mid-beg 等于 42，则此函数**将值最小**的 42 个元素有序放在序列前 42 个位置。当 partial_sort 完成后，从 beg 开始直至 mid 之前的范围中的元素就都已排好序了。**已排序范围中的元素都不会比 mid 后的元素更大。未排序区域中元素的顺序是未指定的。**

```c++
partial_sort_copy (beg, end, destBeg, destEnd)
partial_sort_copy (beg, end, destBeg, destEnd, comp)
```
- **`partial_sort_copy`**：排序输入范围中的元素，并将足够多的已排序元素放到 `destBeg` 和 `destEnd` 所指示的序列中。
    - 如果目的范围的大小`>=`输入范围，则排序整个输入序列并存入从 `destBeg` 开始的范围。
    - 如果目的范围大小`<`输入范围，则只拷贝输入序列中与目的范围一样多的元素。
    - 算法返回一个迭代器，指向目的范围中已排序部分的尾后迭代器。如果目的序列的大小小于或等于输入范围，则返回 `destEnd`。

```c++
nth_element (beg, nth, end)
nth_element (beg, nth, end, comp)
```
**`nth_element`**：参数 nth 必须是一个迭代器，指向输入序列中的一个元素。执行 `nth_element` 后，此**迭代器指向的元素恰好是整个序列排好序后此位置上的值。序列中的元素会围绕 nth 进行划分: nth 之前的元素都小于等于它，而之后的元素都大于等于它。**
## 6 重排顺序算法
**重排输入序列中元素的顺序**。

前两个算法 `remove` 和 `unique`，会重排序列, 使得排在序列第一部分的元素满足某种标准。它们返回一个迭代器，标记子序列的末尾。
其他算法，如 `reverse`、`rotate` 和 `random_shuffle` 都重排整个序列。

**这些算法的基本版本都进行“原址”操作，即，在输入序列自身内部重排元素**。三个重排算法提供 `_copy` 版本。这些 **`_copy` 版本完成相同的重排工作，但将重排后的元素写入到一个指定目的序列中**，而不是改变输入序列。这些算法要求输出迭代器来表示目的序列。

### 使用前向迭代器的重排算法
这些算法重排输入序列。它们要求迭代器至少是前向迭代器。

```c++
remove (beg, end, val)
remove_if (beg, end, unaryPred) 
remove_copy (beg, end, dest, va1)
remove_copy_if (beg, end, dest, unaryPred)
```
**`remove`**： **从序列中“删除”元素，采用的办法是用保留的元素覆盖要删除的元素。** 被删除的是那些 `==val` 或满足 `unaryPred` 的元素。算法返回一个迭代器，指向最后一个删除元素的尾后位置。

```c++
unique (beg, end)
unique (beg, end, binaryPred) 
unique_copy (beg, end, dest)
unique_copy_if (beg, end, dest, binaryPred)
```
**`unique`**：**重排序列，对相邻的重复元素，通过覆盖它们来进行“删除”**。返回一个迭代器，指向不重复元素的尾后位置。第一个版本用 `==` 确定两个元素是否相同，第二个版本使用谓词检测相邻元素。

```c++
rotate (beg, mid, end)
rotate_copy (beg, mid, end, dest)
```
**`rotate`**：**围绕 mid 指向的元素进行元素转动。** 元素 mid 成为首元素，随后是 mid+1 到 end 之前的元素，再接着是 beg 到 mid 之前的元素。返回一个迭代器，指向原来在 beg 位置的元素。

### 使用双向迭代器的重排算法
由于这些算法要反向处理输入序列，它们要求双向迭代器。

```c++
reverse (beg, end)
reverse_copy (beg, end, dest)
```
- **`reverse`**：**翻转序列中的元素。**
    - `reverse` 返回 void
    - `reverse_copy` 返回一个迭代器，指向拷贝到目的序列的元素的尾后位置。

### 使用随机访问迭代器的重排算法
由于这些算法要随机重排元素，它们要求随机访问迭代器。

```c++
random_shuff1e (beg, end)
random_shuffle (beg, end, rand) 
shuffle (beg, end, Uniform_rand)
```

**洗牌算法，混洗输入序列中的元素。**
第二个版本接受一个可调用对象参数，该对象必须接受一个正整数值，并生成 0到此值的包含区间内的一个服从均匀分布的随机整数。shuffle 的第三个参数必须满足均匀分布随机数生成器的要求 (参见 17.4 节，第 659 页)。所有版本都返回 void。

## 7 排列算法
排列算法生成序列的**字典序排列**。对于一个给定序列，这些算法通过重排它的一个排列来生成字典序中下一个或前一个排列。**算法返回一个 bool 值，指出是否还有下一个或前一个排列。**
为了理解什么是下一个或前一个排列，考虑下面这个三字符的序列: abc。它有六种可能的排列: abc、acb、bac、bca、cab 及 cba。这些排列是按字典序递增序列出的。即，abc 是第一个排列，这是因为它的第一个元素小于或等于任何其他排列的首元素，并且它的第二个元素小于任何其他首元素相同的排列。类似的，acb 排在下一位，原因是它以 a 开头，小于任何剩余排列的首元素。同理，以 b 开头的排列也都排在以 c 开头的排列之前。
**对于任意给定的排列，基于单个元素的一个特定的序，我们可以获得它的前一个和下一个排列。给定排列 bca，我们知道其前一个排列为 bac，下一个排列为 cab。序列 abc 没有前一个排列，而 cba 没有下一个排列。**
**这些算法假定序列中的元素都是唯一的**，即，没有两个元素的值是一样的。为了生成排列，必须既向前又向后处理序列，因此算法**要求双向迭代器。**

```c++
is_permutation (beg1, end1 , beg2)
is_permutation (beg1, end1, beg2, binaryPred)
```
**`is_permutation`**：如果第二个序列的某个排列和第一个序列具有相同数目的元素，且元素都相等，则返回 true。第一个版本用 `==` 比较元素，第二个版本使用给定的 binaryPred。

```c++
next_permutation (beg, end)
next_permutation (beg, end, comp)
```
**`next_permutation`**： 如果序列已经是最后一个排列，则将序列重排为最小的排列，并返回 false。否则，它将输入序列转换为字典序中下一个排列，并返回 true。第一个版本使用元素的 `<` 运算符比较元素，第二个版本使用给定的比较操作。

```c++
prev_permutation (beg, end)
prev_permutation (beg, end, comp)
```
**`prev_permutation`**：类似 `next_premutation`，但将序列转换为前一个排列。如果序列已经是最小的排列，则将其重排为最大的排列，并返回 false。
## 8 有序序列的集合算法
集合算法实现了有序序列上的一般集合操作。
**这些算法提供了普通顺序容器 (vector、list 等）或其他序列（如输入流）上的类集合行为。**

> [!warning] 
> 这些算法与标准库 set 容器不同，不要与 set 上的操作相混淆。

这些算法顺序处理元素，因此**要求输入迭代器**。他们还接受一个表示目的序列的输出迭代器，唯一的例外是 `includes`。这些算法返回递增后的 dest 迭代器，表示写入 dest 的最后一个元素之后的位置。
每种算法都有重载版本
- 第一个使用元素类型的 `<` 运算符
- 第二个使用给定的比较操作。

```c++
includes (beg, end, beg2, end2)
includes (beg,end,beg2, end2, comp)
```
**`includes`**：如果第二个序列中每个元素都包含在输入序列中，则返回 true。否则返回 false。

```c++
set_union (beg, end, beg2, end2, dest)
set_union (beg, end, beg2, end2, dest, comp)
```
**`set_union并集`**：对两个序列中的所有元素，创建它们的有序序列。两个序列都包含的元素在输出序列中只出现一次。输出序列保存在 dest 中。
```c++
set_intersection (beg, end, beg2, end2, dest)
set_intersection (beg, end, beg2, end2, dest, comp)
```
**`set_intersection交集`**：对两个序列都包含的元素创建一个有序序列。结果序列保存在 dest 中。

```c++
set_difference (beg, end, beg2, end2, dest)
set_difference (beg, end, beg2, end2, dest, comp)
```
**`set_difference差集`**:对出现在第一个序列中，但不在第二个序列中的元素，创建一个有序序列。

```c++
set_symmetric_difference (beg, end，beg2, end2, dest)
set_symmetric_difference (beg, end，beg2, end2，dest, comp)
```
**`set_symmetric_difference`**：对只出现在一个序列中的元素，创建一个有序序列。

## 9 最大值最小值
这些算法使用元素类型的`<`运算符或给定的比较操作。第一组算法对值而非序列进行操作。第二组算法接受一个序列，它们要求输入迭代器。

```c++
min (val1, val2)
min (val1, val2, comp) 
min (init_list)
min (init_list, comp) 
max (val1, val2)
max (val1, val2, comp) 
max (init_list)
max (init_list, comp)
```
返回 val1 和 val2 中的最小值/最大值，或 `initializer_list` 中的最小值/最大值。
- 两个实参的类型必须完全一致。
- 参数和返回类型都是 const 的引用，意味着对象不会被拷贝。

```c++
minmax (val1, val2)
minmax (va11, va12, comp) minmax (init_list)
minmax (init_list, comp)
```
返回一个 `pair `, 其 first 成员为提供的值中的较小者, second 成员为较大者。`initializer_list` 版本返回一个 pair，其 first 成员为 list 中的最小值，second 为最大值。

```c++
min_element (beg, end)
min_element (beg, end, comp) 
max_element (beg, end)
max_element (beg, end, comp) 
minmax_element (beg, end)
minmax_element (beg, end, comp)
```
`min_element` 和 `max_element` 分别返回指向输入序列中最小和最大元素的迭代器。
`minmax_element` 返回一个 pair，其 first 成员为最小元素，second 成员为最大元素。

**字典序比较**
**此算法比较两个序列，根据第一对不相等的元素的相对大小来返回结果。** 算法使用元素类型的 `<` 运算符或给定的比较操作。两个序列都要求用输入迭代器给出。

```c++
lexicographical_compare (beg1, end1, beg2, end2)
lexicographical_compare (beg1, end1，beg2, end2, comp)
```
**如果第一个序列在字典序中小于第二个序列，则返回 true。否则，返回 false。** 如果一个序列比另一个短，且所有元素都与较长序列的对应元素相等，则较短序列在字典序中更小。如果序列长度相等，且对应元素都相等，则在字典序中任何一个都不大于另外一个。
## 10 数值算法
数值算法定义在头文件 `numeric` 中。这些算法要求输入迭代器; 如果算法输出数据，则使用输出迭代器表示目的位置。

```c++
accumulate (beg, end, init)
accumulate (beg, end, init, binaryop)
```
- **`accumulate`**：返回输入序列中**所有值的和**。和的初值从 init 指定的值开始。返回类型与 init 的类型相同。
    - 第一个版本使用元素类型的`+`运算符
    - 第二个版本使用指定的二元操作。

```c++
inner_product (beg1, end1, beg2, init)
inner_product (beg1, end1, beg2, init, binop1, binop2)
```
- **`inner_product`**：返回两个序列的**内积**，即，对应元素的积的和。两个序列一起处理，来自两个序列的元素相乘，乘积被累加起来。和的初值由 `init` 指定，`init` 的类型确定了返回类型。
    - 第一个版本使用元素类型的乘法 (`*`）和加法 (`+`）运算符。
    - 第二个版本使用给定的二元操作，使用第一个操作代替加法，第二个操作代替乘法。

```c++
partial_sum (beg, end, dest)
partial_sum (beg, end, dest, binaryop)
```
- **`partial_sum`**：**将新序列写入 dest，每个新元素的值都等于输入范围中当前位置和之前位置上所有元素之和**。
    - 第一个版本使用元素类型的+运算符;
    -  第二个版本使用指定的二元操作。算法返回递增后的 dest 迭代器，指向最后一个写入元素之后的位置。

```c++
adjacent_difference (beg, end, dest)
adjacent_difference (beg, end, dest, binaryop)
```
- **`adjacent_difference` 邻差**：将新序列写入 dest，**每个新元素（除了首元素之外）的值都等于输入范围中当前位置和前一个位置元素之差。**
    -  第一个版本使用元素类型的-运算符，
    - 第二个版本使用指定的二元操作。

```c++
iota (beg, end, val)
```
- **`iota`**：将 val 赋予首元素并递增 val。将递增后的值赋予下一个元素，继续递增 val，然后将递增后的值赋予序列中的下一个元素。继续递增 val 并将其新值赋予输入序列中的后续元素。



# 六、标准库特殊设施
## 【C++11】tuple 元组类型
#tuple
![[Pasted image 20230226231649.png]]
-   tuple 是类似 pair 的类型，可以简单地**保存类型不同的任意数量的对象**。
-   tuple 也可以进行比较运算，但是必须元素数量相同时才能比较
-   tuple 的**常见用途是从一个函数返回多个值**，类似于在外部定义一个 struct，但是更加方便且低耦合

### 定义和初始化 tuple
```c++
//当我们定义一个tuple时，需要指出每个成员的类型：
tuple<size_t, size_t, size_t> threeD; //默认初始化，三个成员都设置为0
tuple<string, vector<double>, int, list<int>>
someVa1("constants",{3.14,2.718},42,{0,1,2,3,4,5})
```

**tuple 的构造函数是 explicit 的**，因此必须使用直接初始化语法：
```c++
tuple<size_t, size_t, size_t> threeD = (1,2,3);  //错误
tuple<size_t, size_t, size_t> threeD{1,2,3};  //正确
```
类似 make_pair 函数，标准库定义了 make_tuple 函数，我们还可以用它来生成 tuple 对象：
```c++
//表示书店交易记录的tuple,包含：ISBN、数量和每册书的价格
auto item = make_tup1e("0-999-78345-X", 3, 20.00);
```
make_tuple 函数使用初始值的类型来推断 tuple 的类型。在本例中，item 是一个 tuple, 类型为 `tuple<const char*,int,double>`。

### 访问 tuple 的成员
tuple 的成员都是未命名的（没有 first、second），需要使用 `get` 标准库函数模板访问。
```c++
// 使用get,我们必须指定一个显式模板实参，它指出我们想要访问第几个成员。
// 我们传递给get一个tuple对象，它返回指定成员的引用：
auto book = get<0>(item);  //返回item的第一个成员
auto cnt = get<1>(item);  //返回item的第二个成员
auto price = get<2>(item)/cnt;  //返回item的最后一个成员
get<2>(item) *= 0.8;  //打折20号
```

## bitset 类型
#bitset
-   bitset 类型可以很好地处理位运算问题，比直接使用位操作符清晰方便很多
-   bitset 类似 array，定义的时候模板参数是这个 bitset 的位数
![[Pasted image 20230226231850.png]]
![[Pasted image 20230226231958.png]]
## 正则表达式
略
## 随机数
旧版 c 和 C++使用**rand 函数**生成随机数，此函数生成均匀分布的伪随机数，每个随机数的范围在 0 和一个系统相关最大值（至少为 32767之间）。
有很多程序需要不同范围的随机数，对 rand 进行定制会有很多问题，对此 C++11 引入了**随机数引擎类 (random-number engines)和随机数分布类 (random-number distribution)。**
一个引擎类可以生成 unsigned 随机数序列，一个分布类使用一个引擎类生成指定类型的、在给定范围内的、服从特定概率分布的随机数。
![[Pasted image 20230226232655.png]]

> [!tip] 
> C++程序不应该使用库函数 rand, 而应使用 default_random_engine 类和恰当的分布类对象。

### 随机数引擎和分布
标准库定义了多个随机数引擎类 P783，这里只介绍最常用的 `default_random_engine` 类。
![[Pasted image 20230226234230.png]]
随机数引擎是函数对象类（参见 14.8 节，第 506 页），它们定义了一个调用运算符，该运算符不接受参数并返回一个随机 unsigned 整数。我们可以通过调用一个随机数引擎对象来生成原始随机数：
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
此处我们将 u 定义为 `uniform int distribution<unsigned>`。此类型生成均匀分布的 unsigned 值。当我们定义一个这种类型的对象时，可以提供想要的最小值和最大值。
分布类型也是函数对象类。分布类型定义了一个调用运算符，它接受一个随机数引擎作为参数。分布对象使用它的引擎参数生成随机数，并将其映射到指定的分布。

> [!NOTE] 
> 1. 当我们说**随机数发生器**时，是指分布对象和引擎对象的组合。
> 2. 一个给定的随机数发生器一直会生成相同的随机数序列。一个函数如果定义了局部的随机数发生器，应该将其（包括引擎和分布对象）定义为 static 的。否则，每次调用函数都会生成相同的序列。

### 随机种子
随机数发生器会生成相同的随机数序列，我们可以提供一个**随机种子**来生成不同的随即结果。种子就是一个数值，引擎可以利用它从序列中一个新位置重新生成随机数。
为引擎设置种子有两种方式：在创建引擎对象时提供种子，或者调用引擎的 seed 成员：
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

通常调用系统函数 time 作为种子。由于 time 返回以秒计的时间，因此这种方式只适用于生成种子的间隔为秒级或更长的应用。
```c++
default_random_engine el(time(0));//稍微随机些的种子
```

> [!warning] 
> 如果程序作为一个自动过程的一部分反复运行，将 time 的返回值作为种子的方式就无效了；它可能多次使用的都是相同的种子。

### 其他随机数分布
P665
