
# 一、顺序容器
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
**双向链表**。只支持双向顺序访问。在 list 中任何位置进行插入/删除操作速度都很快

`forward_list`
**单向链表**。只支持单向顺序访问。在链表任何位置进行插入/删除操作速度都很快

## 容器定义和初始化
![[Pasted image 20230211220558.png|650]]

**特别的 `array`**
定义一个 array 时，除了指定元素类型，还要指定容器大小
```c++
array<int, 10>
```
此外，其他容器默认构造都是空容器，而 array 默认构造是非空的：它包含了与其大小一样多的默认初始化元素。


> [!NOTE] 关键概念：容器元素是拷贝
> 当我们用一个对象来初始化容器时，或将一个对象插入到容器中时，实际上放入到容器中的是对象值的一个拷贝，而不是对象本身。就像我们将一个对象传递给非引用参数一样，容器中的元素与提供值的对象之间没有任何关联。随后对容器中元素的任何改变都不会影响到原始对象，反之亦然。

## 通用容器操作
以下操作基本适用于所有容器
![[Pasted image 20230211213826.png]]
![[Pasted image 20230211214254.png]]

### swap 和 assign
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
【C++11】接受元素个数或范围的 insert 版本返回指向第一个新加入元素的迭代器。如果范围为空，不插入任何元素，insert 操作会将第一个参数返回。

【C++11】**`emplace`**
emplace 操作不是拷贝而是构造元素。

emplce_front 对应 push_front
emplce 对应 insert
emplce_back 对应 push_back

当调用 push 或 insert 成员函数时，我们将元素类型的对象传递给它们，这些对象被拷贝到容器中。
而当我们调用一个 emplace 成员函数时，则是将参数传递给元素类型的构造函数。emplace 成员使用这些参数在容器管理的内存空间中直接构造元素。
### 访问元素
![[Pasted image 20230211224406.png]]
- 在容器中访问成员函数返回的都是引用，如果容器是一个 const 对象，则返回值是 const 引用。
- `at` 能够检测下标是否合法，防止越界。 
### 删除元素
![[Pasted image 20230211224643.png]]

**特殊的 forward_list 操作**
![[Pasted image 20230211232259.png]]
### 改变容器的大小
![[Pasted image 20230211232558.png]]

### 额外的 string 操作

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

如果 vector 对象或 string 对象是一个常量，那么只能使用 const_iterator；
如果不是常量，那么两种迭代器都可以使用。

### begin 和 end 成员
begin 和 end 有多个版本
rbegin：返回反向迭代器 (reverse_iterator)
cbegin：返回 const 迭代器（const_iterator）

### 运算
![[Pasted image 20230209204934.png]]
forward_list 的迭代器不支持递减运算符（--）

**以下运算只能应用于 string、vector、deque 和 array 的迭代器**
![[Pasted image 20230209205951.png]]
迭代器相减的结果是两个迭代器的距离，类型是名为 `difference_type` 的带符号整型数。

### 容器操作导致的迭代器失效
在向容器添加元素后：
1. 如果容器是 vector 或 string, 且存储空间被重新分配，则指向容器的迭代器、指针和引用都会失效。如果存储空间未重新分配，指向插入位置之前的元素的迭代器、指针和引用仍有效，但指向插入位置之后元素的迭代器、指针和引用将会失效。
2. 对于 deque, 插入到除首尾位置之外的任何位置都会导致迭代器、指针和引用失效。如果在首尾位置添加元素，迭代器会失效，但指向存在的元素的引用和指针不会失效。
3. 对于 list 和 forward_list, 指向容器的迭代器（包括尾后迭代器和首前迭代器)、指针和引用仍有效。

当我们从一个容器中删除元素后，指向被删除元素的迭代器、指针和引用会失效，这应该不会令人惊讶。毕竟，这些元素都已经被销毁了。

当我们删除一个元素后：
1. 对于 list 和 forward_list, 指向容器其他位置的迭代器（包括尾后迭代器和首前迭代器)、引用和指针仍有效。
2. 对于 deque, 如果在首尾之外的任何位置删除元素，那么指向被删除元素外其他元素的迭代器、引用或指针也会失效。如果是删除 deque 的尾元素，则尾后迭代器也会失效，但其他迭代器、引用和指针不受影响：如果是删除首元素，这些也不会受影响。
3. 对于 vector 和 string, 指向被删元素之前元素的迭代器、引用和指针仍有效。注意：当我们删除元素时，尾后迭代器总是会失效。


> [!command] 建议：管理迭代器
> 当你使用迭代器（或指向容器元素的引用或指针）时，**最小化要求迭代器必须保持有效**的程序片段是一个好的方法。由于向迭代器添加元素和从迭代器删除元素的代码可能会使迭代器失效，因此必须保证每次改变容器的操作之后都正确地重新定位迭代器。
> 这个建议对 vector、string 和 deque 尤为重要。
> 


> [!Warning] warning
> 凡是使用了迭代器的循环体，都不要向迭代器所属的容器添加元素
> 
> 如果在一个循环中插入/删除 deque、string 或 vector 中的元素，不要缓存 end 返回的迭代器。
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

使用 std:: array 的好处是可以**访问它的大小**（通过 s**ize ()** 函数），它是一个**类**。

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

![[Pasted image 20230211234640.png]]

只有当需要的内存空间超过当前容量时，reserve 调用才会改变 vector 的容量。
如果需求大小大于当前容量，reserve 至少分配与需求一样大的内存空间（可能更大）。
如果需求大小小于或等于当前容量，reserve 什么也不做。特别是，当需求大小小于当前容量时，容器不会退回内存空间。因此，在调用 reserve 之后，capacity 将会大于或等于传递给 reserve 的参数。
这样，调用 reserve 永远也不会减少容器占用的内存空间。类似的，resize 成员函数（参见 9.3.5 节，第 314 页）只改变容器中元素的数目，而不是容器的容量。我们同样不能使用 resize 来减少容器预留的内存空间。

### capacity 和 size
**区别：**
size 是指容器已经保存的元素的数目；
capacity 则是在不分配新的内存空间的前提下它最多可以保存多少元素。

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

## string
标准库类型 String 表示可变长的字符序列

### 初始化
![[Pasted image 20230209190416.png]]
### 操作
![[Pasted image 20230209190604.png]]

#### getline 函数
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
#### string:: size_type 类型
size ()函数返回 string:: size_type 类型的值，string[x]的下标 x 也是 string:: size_type 类型。
该类型是无符号整型，而且能足够存放下任何 string 对象的大小。

> [!bug] 防止混用
> 如果一条表达式中已经有了 size ()函数，就不要再使用 int 了，这样可以避免混用带来的问题，问题主要出自于 int 是有符号数，和无符号数运算会出问题

#### 比较 string 长度
如果两个 string 对象在某些写对应的位置上不一致，则 string 对象比较的结果其实是 string 对象中第一对相异字符比较的结果
```c++
string a = "Hello";
string b = "Hello World"
string c = "Hiya"
```
a < b
c > a 且 c > b

#### 字面值和 string 相加
当 string 对象和字符字面值以及字符串字面值混在一条语句中使用时，要确保每个+运算符两侧的运算对象至少有一个是 string：
```c++
string s1 = "Hello"
string s2 = s1 + "World"; //正确，string+字面值 
string s3 = "hello" + "world"; //错误，两个运算对象都不是string 
string s4 =  "world" + "!" + s1; //错误，不能把字面值直接相加
```
	
### 额外的 String 操作
**除了顺序容器通用的操作外，string 类型还提供了一些额外的操作。**
- 提供 string 类和 C 风格字符串之间的相互转换
- 增加了允许我们用下标代替迭代器的版本

#### 初始化
![[Pasted image 20230212153613.png]]

#### substr 操作
返回一个 string，他是原始 string 的一部分或全部的拷贝，可以传递给 substr 一个可选的开始位置和计数值：
```c++
//substr(开始位置，计数值)
string s("hello world");
string s2 = s.substr(0,5);  //s2=he11o
string s3 = s.substr(6);  //s3=world
string s4 = s.substr(6,11);  //s3=world
string s5 = s.substr(12);  //抛出一个out_of_range异常
```

#### 改变 string 的其他方法
![[Pasted image 20230212154457.png]] ![[Pasted image 20230212154543.png]]
#### string 搜索操作
![[Pasted image 20230212154657.png]]
![[Pasted image 20230212154705.png]]

> [!warning] 
> string 搜索函数返回 string: size_type 值，该类型是一个 unsigned 类型。因此，用一个 int 或其他带符号类型来保存这些函数的返回值不是一个好主意。

#### compare 比较函数
![[Pasted image 20230212155018.png]]
#### 数值转换
![[Pasted image 20230212155134.png]]

> [!NOTE] 
> 如果 string 不能转换为一个数值，这些函数抛出一个 invalid_argument 异常（参见 5.6 节，第 173 页）。如果转换得到的数值无法用任何类型来表示，则抛出一个 out_of_range 异常。

### 处理 string 对象中的字符
![[Pasted image 20230209193035.png]]

> [!comment] 建议
> 建议：使用 C++版本的 C 标准库头文件
>
>C++标准库中除了定义 C+语言特有的功能外，也兼容了 C 语言的标准库。C 语言的头文件形如 name. h, C++则将这些文件命名为 cname。也就是去掉了. h 后缀，而在文件名 name 之前添加了字母 c, 这里的 c 表示这是一个属于 C 语言标准库的头文件。
>
>因此，cctype 头文件和 ctype. h 头文件的内容是一样的，只不过从命名规范上来讲更符合 C++语言的要求。特别的，在名为 cname 的头文件中定义的名字从属于命名空间 std, 而定义在名为. h 的头文件中的则不然。
>
>一般来说，C++程序应该使用名为 cname 的头文件而不使用 name. h 的形式。

## 顺序容器适配器
容器、迭代器和函数都有**适配器（adaptor）**，标准库定义了三个顺序容器适配器：
**stack** 栈适配器
**queue** 队列适配器
**priority_queue**

**本质上，一个适配器是一种机制，能使某种事物的行为看起来像另外一种事物一样。一个容器适配器接受一种己有的容器类型，使其行为看起来像一种不同的类型。**

例如，stack 适配器接受一个顺序容器（除 array 或 forward_list 外），并使其操作起来像一个 stack 一样。

![[Pasted image 20230212163058.png]]
### 定义适配器
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
stack<string,vector<string>> str stk2(svec);
```


> [!warning] 重载类型限制
> 所有适配器都要求容器具有添加和删除元素的能力。因此，适配器不能构造在 array 之上。
> 
类似的，我们也不能用 forward_list 来构造适配器，因为所有适配器都要求容器具有添加、删除以及访问尾元素的能力。
>
stack 只要求 push back、pop back 和 back 操作，因此可以使用除 array 和 forward list 之外的任何容器类型来构造 stack。
>
queue 适配器要求 back、push_back、front 和 push_front, 因此它可以构造于 list 或 deque 之上，但不能基于 vector 构造。
>
priority_queue 除了 front、push_back 和 pop_back 操作之外还要求随机访问能力，因此它可以构造于 vector 或 deque 之上，但不能基于 list 构造。

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
queue 使用先入先出（FIFO）策略
priority_queue 允许我们为队列中的元素建立优先级，新加入的元素会排在所有优先级比他低的已有元素之前。

![[Pasted image 20230212165605.png]]
![[Pasted image 20230212165613.png]]
# 二、关联容器
- 关联容器中的元素是按关键字来保存和访问的
- 关联容器的迭代器都是双向的
![[Pasted image 20230213162157.png]]
- 有序关联容器使用比较运算符 `<` 来组织元素，按关键值字典排序从小到大 
- 无序关联容器使用哈希函数（hash function）和关键字类型 `==` 运算符来组织元素 
## 1 map 映射
又称关联数组：关键字-值（键值对）
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
只有关键字，支持关键字查询操作——检查一个给定关键字是否再 set 中
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
## 3 pair 类型
定义在头文件 utility 中
- 一个 pair 保存两个数据成员，必须提供两个类型名
- pair 的数据成员是 public 的，两个成员分别命名为 first，second
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
while (map_it != word count.cend())
{
		//解引用迭代器，打印关键字-值对
		cout << map_it->first << "出现" < <map_it->second << "次"<< endl;
		++map_it;//递增迭代器，移动到下一个元素
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

# 三、泛型算法

## 0 算法文档 P770

> [!Tip] 
> 标准库容器定义的操作集合很小。标准库并未给每个容器添加大量功能，而是提
供了一组算法，这些算法中的大多数都独立于任何特定的容器。这些**算法是通用的
(generic, 或称泛型的)：它们可用于不同类型的容器和不同类型的元素。**
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

- **`beg` 和 `end` 是表示元素范围的迭代器。** 几乎所有算法都对一个由 beg 和 end 表示的序列进行操作。
- **`beg2` 是表示第二个输入序列开始位置的迭代器。`end2` 表示第二个序列的末尾位置（如果有的话）。**
    -  如果没有 end2, 则假定 beg2 表示的序列与 beg 和 end 表示的序列一样大。
    - beg 和 beg2 的类型不必匹配，但是，必须保证对两个序列中的元素都可以执行特定操作或调用给定的可调用对象。
- **`dest` 是表示目的序列的迭代器。** 对于给定输入序列，算法需要生成多少元素，目的序列必须保证能保存同样多的元素。
- **`unaryPred` 和 `binaryPred` 是一元和二元谓词**，分别接受一个和两个参数，都是来自输入序列的元素，两个谓词都返回可用作条件的类型。
- **`comp` 是一个二元谓词**，满足关联容器中对关键字序的要求。
- **`unaryOp` 和 `binaryOp` 是可调用对象**，可分别使用来自输入序列的一个和两个实参来调用。
## 查找算法
这些算法在一个输入序列中搜索一个指定值或一个值的序列。

每个算法都提供两个重载的版本：
- 第一个版本使用底层类型的相等运算符 `==` 来比较元素;
- 第二个版本使用用户给定的 `unaryPred` 和 `binaryPred` 比较元素。
### 简单查找


## 1 只读算法
只读取其输入范围内的元素，不改变元素
`find`
`count`

`accumulate`（定义在头文件 numeric）
accumulate 函数接受三个参数，前两个指出了需要求和的元素的范围，第三个参数是和的初值。第三个参数的类型决定了函数中使用哪个加法运算符以及返回值的类型。
假定 vec 是一个整数序列，则：
```c++
//对vec中的元素求和，和的初值是0
int sum accumulate(vec.cbegin(),vec.cend(),0);
```
这条语句将 sum 设置为 vec 中元素的和，和的初值被设置为0。

`equal`：用于确定两个序列是否保存相同的值。如果所有对应元素都相等，则返回
true, 否则返回 false。
此算法接受三个迭代器：前两个表示第一个序列中的元素范围，第三个表示第二个序列的首元素：
```c++
//roster2中的元素数目应该至少与roster1一样多
equal(roster1.cbegin(),roster1.cend(),roster2.cbegin());
```

**equal 基于一个非常重要的假设：它假定第二个序列至少与第一个序列一样长。**

> [!NOTE]  关键概念：迭代器参数
**一些算法从两个序列中读取元素。构成这两个序列的元素可以来自于不同类型的容
器，两个序列中元素的类型也不要求严格匹配。** 算法要求的只是能够比较两个序列中的元素。例如，对 equal 算法，元素类型不要求相同，但是我们必须能使用 == 来比较来自两个序列中的元素。
>
**用一个单一迭代器表示第二个序列的算法都假定第二个序列至少与第一个一样长。**

## 2 写算法
`fill`：接受一对迭代器表示一个范围，还接受一个值作为第三个参数，将给定的这个值赋予输入序列中的每个元素。
```c++
fi1l(vec.begin(),vec.end(),0);//将每个元素重置为0
//将容器的一个子序列设置为10
fill(vec.begin(),vec.begin(),vec.size()/2,10);
```

`fill_n` 接受一个单迭代器、一个计数值和一个值。它将给定值赋予迭代器指向的元素开始的指定个元素。我们可以用 fill_n 将一个新值赋予 vector 中的元素：
```c++
vector<int>vec;  // 空vector
fill_n(vec.begin(),10,0);  //错误：vec是空的，不能写入10个元素。

fill_n(vec.begin(),vec.size(),0); // 正确，将所有元素重置为0
```

> [!warning] 
>向目的位置迭代器写入数据的算法假定目的位置足够大，能容纳要写入的元素。

## 3 拷贝算法
此算法接受三个迭代器，前两个表示一个输入范围，第三个表示目的序列的起始位置。此算法**将输入范围中的元素拷贝到目的序列中**。传递给 copy 的目的序列至少要包含与输入序列一样多的元素，这一点很重要。

我们可以用 coPy 实现内置数组的拷贝，如下面代码所示：
```c++
inta1[]={0,1,2,3,4,5,6,7,8,9}:
inta2[sizeof(a1)/sizeof(*a1)】;//a2与a1大小一样
//ret指向拷贝到a2的尾元素之后的位置
auto ret=copy(begin(a1),end(a1),a2);//把a1的内容拷贝给a2
```

copy 返回目的位置迭代器（递增后）的值，即 ret 指向拷贝到 a2 的尾元素之后的位置。

**多个算法都提供“拷贝”版本。** 这些算法计算新元素的值，但不会将它们放置在输入序列的末尾，而是创建一个新序列保存这些结果。
`replace` 算法读入一个序列，并将其中所有等于给定值的元素都改为另一个
值。此算法接受 4 个参数：前两个是迭代器，表示输入序列，后两个一个是要搜索的值，
另一个是新值。它将所有等于第一个值的元素替换为第二个值：
```c++
//将所有值为0的元素改为42
replace (ilst.begin(),ilst.end(),0,42);
```
此调用将序列中所有的 0 都替换为 42。如果我们希望保留原序列不变，可以调用 `replace_copy`。此算法接受额外第三个迭代器参数，指出调整后序列的保存位置：
```c++
//使用back inserter按需要增长目标序列
replace_copy(ilst.cbegin(),ilst.cend(),
back_inserter (ivec),0,42);
```
此调用后，i1st 并未改变，ivec 包含 i1st 的一份拷贝，不过原来在 ilst 中值为 0 的元素在 ivec 中都变为42。

## 4 排序算法
`sort`：从小到大排序，使用<运算符
`unique`：sort 后将相邻的重复项隐藏（并不是删除，放在了末尾），返回的迭代器指向最后一个不重复元素之后的位置

原 vector：
![[Pasted image 20230212174503.png]]
sort 后：
![[Pasted image 20230212174441.png]]
unique 后：![[Pasted image 20230212174447.png]]
`stable_sort`：等长元素字典排序
## 5 定制操作
**用 lambda 表达式定制操作**

### 谓词：向算法传递函数

> [!NOTE] 谓词
> **谓词是一个可调用的表达式，是返回值为 `bool` 的普通函数或者函数对象**
> 
> 标准库算法所使用的谓词分为两类：
> 一元谓词 (unary predicate, 意味着它们只接受单一参数)
> 二元谓词 (binary predicate, 意味着它们有两个参数)。
> 
> 接受谓词参数的算法对输入序列中的元素调用谓词。因此，元素类型必须能转换为谓词的参数类型。

###  内置的 sort 函数

1. sort (vec.begin (), vec.end (), 谓词)

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

对于已定的传入参数的顺序 `[](int a, int b)`，函数体中如果参数 a 在前面，则返回 true，如果参数 a 在后面则返回 false

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

如果 a == 1，则把它移到后面去，即返回 false，不希望它在 b 前。如果 b == 1，我们希望 a 在前面，要返回 true。

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

**`find_if` 是一个搜索类的函数，区别于 `find` 的是：它可以接受一个函数指针来定义搜索的规则，返回满足这个规则的第一个元素的迭代器**。这个情况就很适合 lambda 表达式的出场了

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
for_each 时 vector 的一个操作
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
`arg_list` 中的参数可能包含形如 `_n` 的名字，其中 n 是一个整数。这些参数是“占位符”, 表示 newCallable 的参数，它们占据了传递给 newCallable 的参数的“位置”。
数值 n 表示生成的可调用对象中参数的位置：`_1` 为 newCallable 的第一个参数，`_2` 为第二个参数，依此类推。
名字_n 都定义在名为**placeholders 命名空间**，该命名空间定义在 std 命名空间。使用时，都要写上。

#### 绑定类的成员函数
类的成员函数必须通过类的对象或者指针调用，因此在 bind 时， `arg_list` 中的第一个参数的位置来指定一个类的实列、指针或引用。
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

**bind 功能如下：**
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
例如，假定 f 是一个可调用对象，它有 5 个参数，则下面对 bind 的调用：
```c++
//g是一个有两个参数的可调用对象
auto g = bind(f,a,b,_2,c,_1);
```

生成一个新的可调用对象，它有两个参数，分别用占位符_2 和_1 表示。这个新的可调用对象将它自己的参数作为第三个和第五个参数传递给 f。f 的第一个、第二个和第四个参数分别被绑定到给定的值 a、b 和 c 上。
传递给 g 的参数按位置绑定到占位符。即，第一个参数绑定到_1，第二个参数绑定到_2。因此，当我们调用 g 时，其第一个参数将被传递给 f 作为最后一个参数，第二个参数将被传递给 f 作为第三个参数。实际上，这个 bind 调用会将 `g(_1,_2)` 映射为 `f(a,b,_2,c,1)` 即，对 g 的调用会调用 f, 用 g 的参数代替占位符，再加上绑定的参数 a、b 和 c。例如，调用 `g(X,Y)` 会调用 `f(a,b,Y,c,X)`

下面是用 bind 重排参数顺序的一个具体例子，我们可以用 bind 颠倒 isShroter
的含义：
```c++
//按单词长度由短至长排序
sort (words.begin(), words.end(), isShorter);
//按单词长度由长至短排序
sort (words.begin(), words.end(), bind(isShorter,2,1));
```
在第一个调用中，当 sort 需要比较两个元素 A 和 B 时，它会调用 isShorter(A,B)。
在第二个对 sort 的调用中，传递给 isShorter 的参数被交换过来了。因此，当 sort 比较两个元素时，就好像调用 isShorter (B,A)一样。

#### ref 函数：绑定引用参数
#ref
默认情况下，bind 的那些不是占位符的参数被拷贝到 bind 返回的可调用对象中。但是，与 lambda 类似，**有时对有些绑定的参数我们希望以引用方式传递，或是要绑定参数的类型无法拷贝。**
例如，为了替换一个引用方式捕获 ostream 的 lambda:
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
函数 ref 返回一个对象，包含给定的引用，此对象是可以拷贝的。标准库中还有一个 `cref` 函数，生成一个保存 const 引用的类。与 bind 一样，函数 ref 和 cref 也定义在头文件 functional 中。

## 6 泛型迭代器
除了为每个容器定义的迭代器之外，标准库在头文件 `iterator` 中还定义了额外几种迭代器。

1. **插入迭代器 (insert iterator)**：绑定到一个容器上，可用来向容器插入元素。
3. **流迭代器 (stream iterator)**：绑定到输入或输出流上，可用来遍历所关联的 IO 流。
4. **反向迭代器 (reverse iterator)**：向后而不是向前移动。除了 forward_list 之外的标准库容器都有反向迭代器。
6. **移动迭代器 (move iterator)**：这些专用的迭代器不是拷贝其中的元素，而是移动它们。

### 插入迭代器
插入器是一种迭代器适配器，它接受一个容器，生成一个迭代器，能实现向给定容器添加元素。当我们通过一个插入迭代器进行赋值时，该迭代器调用容器操作来向给定容器的指定位置插入一个元素。

![[Pasted image 20230212223317.png]]

通常情况，当我们通过一个迭代器向容器元素赋值时，值被赋予迭代器指向的元素。而当我们通过一个插入迭代器赋值时，一个与赋值号右侧值相等的元素被添加到容器中。

**插入器有三种类型，差异在于元素插入的位置。**
#### back_inserter
- `back_inserter` 创建一个使用 push back 的迭代器。back inserter 接受一个指向容器的引用，返回一个与该容器绑定的插入迭代器。当我们通过此迭代器赋值时，赋值运算符会调用 push_back 将一个具有给定值的元素添加到容器中：
```c++
vector<int> vec;//空向量
auto it=back_inserter(vec);//通过它赋值会将元素添加到vec中
*it=42;//vec中现在有一个元素，值为42
```
我们常常使用 back_inserter 来创建一个迭代器，作为算法的目的位置来使用。例如：
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
- `inserter` 创建一个使用 insert 的迭代器。此函数接受第二个参数，这个参数必须是一个指向给定容器的迭代器。元素将被插入到给定迭代器所表示的元素之前。
```c++
auto it = inserter(c,iter);
*it = val;

//等价
it = c.insert(it,val);
++it //递增it使他指向原来的元素
```

> [!NOTE] 
> 只有在容器支持 push front 的情况下，我们才可以使用 front_inserter。类似的，只有在容器支持 push_back 的情况下，我们才能使用 back_inserter。

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

## 8 链表 list 容器算法 
 
> [!NOTE] 
> 对于 list 和 forward_list，应该**优先使用成员函数版本**的算法而不是通用算法。
>
链表特有的操作会改变底层的容器

 ![[Pasted image 20230212235051.png]]
![[Pasted image 20230212235106.png]]

特有的 splice 成员
![[Pasted image 20230212235202.png]]
