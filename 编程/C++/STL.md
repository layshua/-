# 关于容器

STL中容器的概述

## 容器-Container

**容器**用于表示由同类型元素构成的、长度可变的元素序列。

容器是由类模板来实现的，模板的参数是容器中元素的类型。

STL中包含了很多种容器，虽然这些容器提供了一些相同的操作，但由于它们采用了不同的内部实现方法，因此，**不同的容器**往往适合于**不同的应用场合**。

## STL的主要容器

### vector<元素类型>

用于需要快速定位（访问）任意位置上的元素以及主要在元素序列的尾部增加/删除元素的场合。

在头文件`vector`中定义，用**动态数组实现。**

### list<元素类型>

用于经常在元素序列中任意位置上插入/删除元素的场合。

在头文件`list`中定义，用**双向链表**实现。

> [!NOTE]
> 在C++11标准中增加了`forward_list`容器，本质上是一个**单向链表**，定义在头文件`forward_list`中。
> 

### deque<元素类型>

用于主要在元素序列的两端增加/删除元素以及需要快速定位（访问）任意位置上的元素的场合。

在头文件`deque`中定义，用**分段的连续空间结构**实现。

### stack<元素类型>

用于仅在元素序列的尾部增加/删除元素的场合。

在头文件`stack`中定义，可基于`deque`、`list`或`vector`来实现。

### queue<元素类型>[](https://cui-jiacai.gitbook.io/c++-stl-tutorial/gai-shu/guan-yu-rong-qi#queue-yuan-su-lei-xing)

用于仅在元素序列的尾部增加、头部删除元素的场合。

在头文件`queue`中定义，可基于`deque`和`list`来实现。

### priority_queue<元素类型>

它与`queue`的操作类似，不同之处在于：每次增加/删除元素之后，它将对元素位置进行调整，使得头部元素总是最大的。也就是说，每次删除操作总是把最大（优先级最高）的元素去掉。

在头文件`queue`中定义，可基于`deque`和`vector`来实现。

### map<关键字类型，值类型> 和 multimap<关键字类型，值类型>

用于需要根据关键字来访问元素的场合。容器中每个元素是一个`pair`结构类型，该结构有两个成员：`first`和`second`，关键字对应`first`，值对应`second`，元素是根据其关键字排序的。

对于`map`，不同元素的关键字不能相同；

对于`multimap`，不同元素的关键字可以相同。

它们在头文件`map`中定义，常常用某种**二叉树**来实现。

> [!NOTE]
> 有时候我们不需要排序，所以在C++11标准中新增加了`unordered_map` 和`unordered_multimap`容器。

### set<元素类型> 和 multiset<元素类型>

它们分别是`map`和`multimap`的特例，每个元素只有关键字而没有值，或者说，关键字与值合一了。

在头文件`set`中定义。

> [!NOTE]
> C++11标准中增加了`unordered_set`和`unordered_multiset`容器。
> 

### basic_string<字符类型>

与`vector`类似，不同之处在于其元素为字符类型，并提供了一系列与**字符串**相关的操作。

`string`和`wstring`分别是它的两个实例：

- `basic_string<char>`
- `basic_string<wchar_t>`
    

在头文件`string`中定义。

## 容器的基本操作

容器类模板提供了一些成员函数来实现容器的**基本操作**，其中包括：

- 往容器中增加元素
- 从容器中删除元素
- 获取容器中指定位置的元素
- 在容器中查找元素
- 获取容器首/尾元素的迭代器
- ......

> 这些成员函数往往具有通用性（大部分容器类模板都包含它们）。

> [!NOTE]
> 注意：如果容器的元素类型是一个类，则针对该类可能需要：
> 
> - 自定义拷贝构造函数和赋值操作符重载函数
>     - 对容器进行的一些操作中可能会创建新的元素（对象的拷贝构造）或进行元素间的赋值（对象赋值）。
>     - 重载小于操作符（<） 对容器进行的一些操作中可能要进行元素比较运算。


# 关于迭代器

## 迭代器-Iterator

迭代器实现了**智能指针**功能，它们用于指向容器中的元素，对容器中的元素进行访问和遍历。

在STL中，迭代器是作为类模板来实现的（在头文件`iterator`中定义），它们可分为以下几种类型：

### 根据访问修改权限分类[](https://cui-jiacai.gitbook.io/c++-stl-tutorial/gai-shu/guan-yu-die-dai-qi#gen-ju-fang-wen-xiu-gai-quan-xian-fen-lei)

- 输出迭代器（output iterator，记为：**OutIt**）
    - 可以修改它所指向的容器元素
    - 间接访问操作（`*`）
    - `++`操作
- 输入迭代器（input iterator，记为：**InIt**）
    - 只能读取它所指向的容器元素
    - 间接访问操作（`*`）和元素成员间接访问（`->`）
    - `++`、`==`、`!=`操作。

### 根据迭代方式分类

- 前向迭代器（forward iterator，记为：**FwdIt**）
    - 可以读取/修改它所指向的容器元素
    - 元素间接访问操作（`*`）和元素成员间接访问操作（`->`）
    - `++`、`==`、`!=`操作
- 双向迭代器（bidirectional iterator，记为：**BidIt**）
    - 可以读取/修改它所指向的容器元素
    - 元素间接访问操作（`*`）和元素成员间接访问操作（`->`）
    - `++`、`--`、`==`、`!=`操作
- 随机访问迭代器（random-access iterator，记为：**RanIt**）
    - 可以读取/修改它所指向的容器元素
    - 元素间接访问操作（`*`）、元素成员间接访问操作（`->`）和下标访问元素操作（`[]`）
    - `++`、`--`、`+`、`-`、`+=`、`-=`、`==`、`!=`、`<`、`>`、`<=`、`>=`操作

## 各容器的迭代器类型

- 对于`vector`、`deque`以及`basic_string`容器类，与它们关联的迭代器类型为**随机访问迭代器（RanIt）**
- 对于`list`、`map/multimap`以及`set/multiset`容器类，与它们关联的迭代器类型为**双向迭代器（BidIt）**

`queue`、`stack`和`priority_queue`容器类，不支持迭代器！

## 迭代器之间的相融关

![[cefca8bcc1571137315093623ec613f2_MD5.png]]

迭代器之间的相融关系
**在需要箭头左边迭代器的地方可以用箭头右边的迭代器去替代。**

除了上面五种基本迭代器外，STL还提供了一些迭代器的适配器，用于一些特殊的操作，如：

### 反向迭代器（reverse iterator）

用于对容器元素从尾到头进行反向遍历，可以通过容器类的成员函数`rbegin`和`rend`可以获得容器的尾和首元素的反向迭代器。

需要注意的是，对反向迭代器，`++`操作是往容器首部移动，`--`操作是往容器尾部移动。

### 插入迭代器（insert iterator）

用于在容器中指定位置插入元素，其中包括：

- `back_insert_iterator`（用于在尾部插入元素）
- `front_insert_iterator`（用于在首部插入元素）
- `insert_iterator`（用于在任意指定位置插入元素）
    
它们可以分别通过函数`back_inserter`、`front_inserter`和`inserter`来获得，函数的参数为容器。

# 关于算法

STL中算法的概述

## 算法-Algorithm

在STL中，除了用容器类模板自身提供的成员函数来操作容器元素外，还提供了一系列通用的对容器中元素进行操作的**函数模板**，称为**算法**。

STL算法实现了**对序列元素的一些常规操作**，用函数模板实现的，主要包括：

- 调序算法
- 编辑算法
- 查找算法
- 算术算法
- 集合算法
- 堆算法
- 元素遍历算法 

**除了算术算法在头文件 `numeric` 中定义外，其它算法都在头文件 `algorithm` 中定义。**

大部分STL算法都是**遍历**指定容器中某范围内的元素，对**满足条件**的元素执行**某项操作**。

算法的内部实现往往隐含着循环操作，但这对使用者是隐藏的。

- 使用者只需要提供：容器（迭代器）、操作条件以及可能的自定义操作。
- 算法的**控制逻辑则由算法内部实现**，这体现了一种**抽象**的编程模式。

## 算法与容器之间的关系[](https://cui-jiacai.gitbook.io/c++-stl-tutorial/gai-shu/guan-yu-suan-fa#suan-fa-yu-rong-qi-zhi-jian-de-guan-xi)

在STL中，不是把容器传给算法，而是把容器的某些迭代器传给它们，在算法中通过迭代器来访问和遍历相应容器中的元素。

这样做的好处是：使得**算法不依赖于具体的容器，提高了算法的通用性**。

虽然容器各不相同，但它们的迭代器往往具有相容关系，一个算法往往可以接受相容的多种迭代器。

## 算法接受的迭代器类型

一个算法能接收的迭代器的类型是通过**算法模板参数的名字**来体现的。例如：

```c++
template <class InIt, class OutIt>
OutIt copy(InIt src_first, InIt src_last, OutIt dst_first) {...}
```

- `src_first`和`src_last`是输入迭代器，算法中只能读取它们指向的元素。
- `dst_first`是输出迭代器，算法中可以修改它指向的元素。
- 以上参数可以接受其它相容的迭代器。

## 算法的操作范围

**用算法对容器中的元素进行操作时，大都需要用两个迭代器来指出要操作的元素的范围。**

例如：

```
void sort(RanIt first, RanIt last);
```

- `first`是第一个元素的位置
- `last`是最后一个元素的下一个位置

**有些算法可以有多个操作范围，这时，除第一个范围外，其它范围可以不指定最后一个元素位置，它由第一个范围中元素的个数决定。** 例如：

```
OutIt copy(InIt src_first, InIt src_last, OutIt dst_first);
```

一个操作范围的两个迭代器必须属于同一个容器，而不同操作范围的迭代器可以属于不同的容器。

## 算法的自定义操作条件

有些算法可以让使用者提供一个**函数**或**函数对象**来作为**自定义操作条件**（或称为**谓词**），其参数类型为相应容器的元素类型，返回值类型为bool。

自定义操作条件可分为：

- **Pred**：一元“谓词”，需要一个元素作为参数
- **BinPred**：二元“谓词”，需要两个元素作为参数
    

### 一元谓词举例[](https://cui-jiacai.gitbook.io/c++-stl-tutorial/gai-shu/guan-yu-suan-fa#yi-yuan-wei-ci-ju-li)

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

### 二元谓词举例

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

## 算法的自定义操作

有些算法可以让使用者提供一个函数或函数对象作为**自定义操作**，其**参数和返回值类型由相应的算法决定。**

自定义操作可分为：

- **Op**或**Fun**：一元操作，需要一个参数
- **BinOp**或**BinFun**：二元操作，需要两个参数

### 一元操作举例

例如，对于下面的“元素遍历”算法：

```c++
Fun for_each(InIt first, InIt last, Fun f);
```

可以有如下用法：
```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void f(int x) { cout << ' ' << x; }

int main()
{
     vector<int> v;
     ...... // 往容器中放了元素
     for_each(v.begin(), v.end(), f); // 对v中的每个元素去调用函数f进行操作
     return 0;
}
```
### 二元操作举例

例如，对于下面的“累积”算法：

```c++
T accumulate(InIt first, InIt last, T val); // 按“+”操作
T accumulate(InIt first, InIt last, T val, BinOp op); // 由op决定累积的含义
```

设元素为 $e_1, e_2,..., e_n$，算法返回 $t_{n}$：
$$t_1=op(val,e_1),t_2=op(t1,e_2),......t_n=op(t_{n-1},e_{n-1})$$

可以有如下用法：
```c++
#include <vector>
#include <numeric>
using namespace std;

int f1(int partial, int x) { return partial * x; }
int f2(int partial, int x) { return partial + x * x; }
double f3(double partial, int x) { return partial + 1.0 / x; }

int main
{
    vector<int> v;
    ...... // 往容器中放了元素
    
    int sum = accumulate(v.begin(), v.end(), 0); // 所有元素和
    int product = accumulate(v.begin(), v.end(), 1, f1); // 所有元素的乘积
    int sq_sum = accumulate(v.begin(), v.end(), 0, f2); // 所有元素平方和
    int rec_sum = accumulate(v.begin(), v.end(), 0.0, f3); // 元素倒数和
    
    return 0;
}
```

再例如，对于下面的元素“**变换/映射**”算法：

```c++
OutIt transform(InIt src_first, InIt src_last, OutIt dst_first, Op f);
OutIt transform(InIt1 src_first1, InIt1 src_last1, InIt2 src_first2, OutIt dst_first, BinOp f);
```

可以有如下用法：
```c++
#include <algorithm>
#include <vector>
using namespace std;

int f1(int x) { return x * x; }
int f2(int x1, int x2) { return x1 + x2; }

int main()
{
    vector<int> v1,v2,v3,v4;
    ...... // 往v1和v2容器中放了元素
    
    transform(v1.begin(),v1.end(),v3.begin(),f1); 
    // v3中的元素是v1相应元素的平方
    
    transform(v1.begin(),v1.end(),v2.begin(),v4.begin(),f2); 
    // v4中的元素是v1和v2相应元素的和
    
    return 0;
}
```

至此，读者应当已经明白了STL中的算法是怎样通过迭代器作用于容器的。



# 如何选取合适的容器

根据前面的叙述，容器分为序列容器和容器适配器两种，其选择的主要流程如下：
![[Pasted image 20230825095732.png]]
![[spaces_4jcp9JrFSUV0Enu5fcXK_uploads_IkWXvNJMNfxAsypAnqkX_容器适配器.webp]]

# string 字符串

## string 和 C风格字符 串对比

- `char*`是一个指针，`string`是一个类
    `string`封装了`char*`，管理这个字符串，是一个`char*`型的容器。
    

- `string` 封装了很多实用的成员方法
    查找`find`，拷贝`copy`，删除`erase`，替换`replace`，插入`insert`......
    

- 不用考虑内存释放和越界
    `string`管理`char*`所分配的内存，每一次`string`的复制/赋值，取值都由`string`类负责维护，不用担心复制越界和取值越界等。

> `string` 本质上是一个动态的 char 数组。

## string 常用操作
### string 构造函数
```c++
string();
// 默认构造函数，创建一个空的字符串
string(const string& str);
// 拷贝构造函数，使用一个string对象初始化另一个string对象
string(const char* s);
// 含参构造函数，使用C风格字符串初始化
string(int n, char c);
// p含参构造函数，使用n个字符c初始化
```

### string 赋值操作
#### `=` 赋值操作符
```c++
string& operator=(const char* s);
// C风格字符串赋值给当前的字符串
string& operator=(const string& s);
// 把字符串s赋给当前的字符串
string& operator=(const char c);
//字符赋值给当前的字符串
```

#### assign 成员函数
```c++
string& assign(const char* s); 
// C风格字符串赋值给当前的字符串
string& assign(const char* s, int n); 
// 把C风格字符串s的前n个字符赋给当前的字符串
string& assign(const string& s); 
// 把字符串s赋给当前字符串
string& assign(int n, char c); 
// 把n个字符c赋给当前的字符串
string& assign(const string& s, int start, int n); 
// 将字符串s中从start开始的n个字符赋值给当前字符串
```

### string 存取字符操作

#### `[]` 下标获取操作符
```c++
char& operator[](int n); 
// 通过[]下标方式获取字符
```
使用下标操作符获取字符时，如果下标越界，程序将会强制终止。
#### `at` 成员函数
```c++
char& at(int n); 
// 通过at方法获取字符
```

使用 at 方法获取字符时，如果下标越界，at 方法内部会抛出异常（`exception`），可以使用 `try-catch` 捕获并处理该异常。示例如下：
```c++
#include <stdexception> 
//标准异常头文件
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
### string 拼接操作
#### `+=`复合操作符
```c++
string& operator+=(const string& str); 
// 将字符串str追加到当前字符串末尾
string& operator+=(const char* str); 
// 将C风格字符数组追加到当前字符串末尾
string& operator+=(const char c); 
// 将字符c追加到当前字符串末尾
/* 上述操作重载了复合操作符+= */
```
#### `append` 成员函数
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
### string 查找和替换
#### find成员函数
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
当查找失败时，find方法会返回-1，-1已经被封装为string的静态成员常量`string::npos`。
`static const size_t nops = -1;`
#### rfind成员函数
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
find方法通常查找字串第一次出现的位置，而rfind方法通常**查找字串最后一次出现的位置。**
rfind(str, pos)的实际的开始位置是pos + str.size()，即从该位置开始（不包括该位置字符）向前寻找匹配项，如果有则返回字符串位置，如果没有返回string::npos。
-1其实是size_t类的最大值（学过补码的同学应该不难理解），所以string::npos还可以表示“直到字符串结束”，这样的话rfind中pos的默认参数是不是就不难理解啦？
#### replace成员函数
```c++
string& replace(int pos, int n, const string& str); 
// 替换从pos开始n个字符为字符串s
string& replace(int pos, int n, const char* s);
// 替换从pos开始的n个字符为字符串s
```

### string 比较操作
#### compare 成员函数
```c++
int compare(const string& s) const; // 与字符串s比较
int compare(const char* s) const; // 与C风格字符数组比较
```
compare函数**依据字典序比较**，在当前字符串比给定字符串小时返回-1，在当前字符串比给定字符串大时返回1，相等时返回0。
#### 比较操作符
```c++
bool operator<(const string& str) const;
bool operator<(const char* str) const;
bool operator<=(const string& str) const;
bool operator<=(const char* str) const;
bool operator==(const string& str) const;
bool operator==(const char* str) const;
bool operator>(const string& str) const;
bool operator>(const char* str) const;
bool operator>=(const string& str) const;
bool operator>=(const char* str) const;
bool operator!=(const string& str) const;
bool operator!=(const char* str) const;
```
string类重载了所有的比较操作符，其含义与比较操作符本身的含义相同。
### string 子串
#### substr成员函数
```c++
string substr(int pos = 0, int n = npos) const;
// 返回由pos开始的n个字符组成的字符串
```
### string 插入和删除操作
#### insert 成员函数
```c++
string& insert(int pos, const char* s); // 在pos位置插入C风格字符数组
string& insert(int pos, const string& str); // 在pos位置插入字符串str
string& insert(int pos, int n, char c); // 在pos位置插入n个字符c
```
返回值是插入后的字符串结果，erase同理。其实就是指向自身的一个引用。

#### erase 成员函数
```c++
string& erase(int pos, int n = npos); // 删除从pos位置开始的n个字符，默认一直删除到末尾。
```
### string 和 C-Style 字符串的转换

#### string 转 const char*

```c++
string str = "demo";
const char* cstr = str.c_str();
```

#### const char* 转 string
```c++
const char* cstr = "demo";
string str(cstr); // 本质上其实是一个有参构造
```
在 c++中存在一个从 `const char*` 到 `string` 类的隐式类型转换，但却不存在从一个 `string` 对象到` const char*`的自动类型转换。对于 string 类型的字符串，可以通过 `c_str()`方法返回 `string` 对象对应的 `const char*` 字符数组。
比如说，当一个函数的参数是`string`时，我们可以传入`const char*`作为参数，编译器会自动将其转化为`string`，但这个过程不可逆。
为了修改`string`字符串的内容，下标操作符`[]`和`at`都会返回字符串的引用，但当字符串的内存被重新分配之后，可能发生错误。（结合字符串的本质是动态字符数组的封装便不难理解了）
### 和 string 相关的全局函数

> [!NOTE]
> 注：有的可能需要C++11标准。

#### 大小写转换
```c++
#include <cctype>
// 在iostream中已经包含了这个头文件，如果没有包含iostream头文件，则需手动包含cctype

int tolower(int c); // 如果字符c是大写字母，则返回其小写形式，否则返回本身
int toupper(int c); // 如果字符c是小写字母，则返回其大写形式，否则返回本身

/**
  * C语言中字符就是整数，这两个函数是从C库沿袭过来的，保留了C的风格
*/
```
如果想要对整个字符串进行大小写转化，则需要使用一个for循环，或者配合和algorithm库来实现。例如：
```c++
#include <string>
#include <cctype>
#include <algorithm>

string str = "Hello, World!";
transform(str.begin(), str.end(), str.begin(), toupper); //字符串转大写
transform(str.begin(), str.end(), str.begin(), tolower); //字符串转小写
```

#### 字符串和数字的转换
##### int/double 转 string
c++11标准新增了全局函数`std::to_string`，十分强大，可以将很多类型变成 string 类型。

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
##### string 转 double / int
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
与之类似的在同一个库里的还有一组基于字符数组的函数如下。
// 'a' means array, since it is array-based. 

int atoi(const char* str); // 'i' means  int
long atol(const char* str); // 'l' means long
long long atoll(const char* str); // 'll' means long long

double atof(const char* str); // 'f' means double
```

# vector - 向量
STL中vector类的详细介绍
## vector 容器基本概念
vector的数据安排及操作方式，与array非常相似，两者的唯一差别在于空间的运用的灵活性。
array是静态空间，一旦配置了一般不能改变，如果要改变空间大小，需要自行完成以下三个步骤：
- 配置一块新的空间
- 将旧数据搬往新的空间
- 释放原来的空间
而 vector 是动态空间，但其实 vector 的动态也是对于上述过程的封装，并且 vector 配置空间的策略也考虑了运行成本，采用特定的扩展的策略（并不是简单的成倍扩展）。
![[spaces_4jcp9JrFSUV0Enu5fcXK_uploads_wOcATtLA7TPligYyNsnw_屏幕截图 06-07-2021 22.webp]]
## vector 迭代器
vector维护一个线性空间，所以不论元素类型如何，普通指针都可以作为vector的迭代器。
因为vector迭代器所需要的行为，如`operator*，operator->，operator++，operator--，operator+，operator-，operator+=，operator-=`，普通指针天生具备。
vector指针支持随机存取，而普通指针正有着这样的能力。
所以，vector提供的是**随机访问迭代器（Random Access Iterator）**，其内部用普通指针实现。
### 使用迭代器进行正序遍历
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

### 使用迭代器逆序遍历
```

for (vector<T>::reverse_iterator it = v.rbegin(); it != v.rend(); it++)
{
    cout << *it << endl;
}
/**
  * 1. 逆向迭代器不再是iterator，而是reverse_iterator
  * 2. 逆序首位迭代器由rbegin()和rend()方法生成
*/
```

```c++
判断迭代器是否能随机访问的方法
用多了自然就背上了，下面给出一种现场测试的方法。
iterator++；
iterator--；
//通过编译，至少是双向迭代器
  
iterator = iterator + 1；
//通过编译，则是随机访问迭代器
```
vector 数据结构
vector采用的数据结构非常简单，线性连续空间，它以两个迭代器_Myfirst和_Mylast分别指向配置得来的连续空间中已被使用的范围，并以迭代器Myend指向整块连续内存空间的尾端。
为了降低空间配置时的成本，vector实际配置的大小可能比用户端需求大一些，以备将来可能的扩充，这便是容量的概念。
一个vector容器的容量永远大于等于其大小，一旦容量等于大小，便是满载，下次再有新增元素，整个vector容器就得另觅居所。
所谓动态增加大小，并不是在原空间之后续接新空间（因为无法保证原空间之后尚有可配置的空间），而是一块更大的内存空间，然后将原数据拷贝新空间，并释放原空间。
因此，对vector的任何操作，一旦引起空间的重新配置，指向原vector的所有迭代器就都失效了。
vector常用API操作
API = Applicational Programming Interface
vector 构造函数
vector<T> v; // 采用模版类实现，默认构造函数
vector<T> v(T* v1.begin(), T* v1.end()); // 将v1[begin(), end())区间中的元素拷贝给本身
vector<T> v(int n, T elem); // 将n个elem拷贝给本身
vector<T> v(const vector<T> v1); // 拷贝构造函数
下面对于第二种构造方式给出一个特殊的例子：
int array[5] = {1, 2, 3, 4, 5};
vector<int> v(array, array + sizeof(array) / sizeof(int));
// 联系我们之前提到的vector迭代器本质上是指针就不难理解了
vector 常用赋值操作
由于vector采用模版类实现，其完整的函数声明会稍显复杂，下面方法的演示会省略类型界定。
assign(beg, end); // 将[beg, end)区间中的数据拷贝复制给本身
assign(n, elem); // 将n个elem拷贝给本身
vector& operator=(const vector& vec); // 重载赋值操作符
互换操作也可视为一种特殊的赋值：
swap(vec); //将vec与本身的元素互换
巧用swap来收缩空间：
vector<int>(v).swap(v);
// vector<int>(v): 利用拷贝构造函数初始化匿名对象
// swap(v): 交换的本质其实只是互换指向内存的指针
// 匿名对象指针指向的内存会由系统自动释放
vector 大小操作
int size(); // 返回容器中的元素个数
bool empty(); // 判断容器是否为空
​
void resize(int num); 
// 重新指定容器的长度为num，若容器变长，则以默认值填充新位置。
// 若容器变短，则末尾超出容器长度的元素被删除
void resize(int num, T elem); 
// 重新指定容器的长度为num，若容器变长，则以elem填充新位置。
// 若容器变短，则末尾超出容器长度的元素被删除
​
int capacity(); // 返回容器的容量
void reserve(int len); 
// 容器预留len个元素长度，预留位置不初始化，元素不可访问
vector 数据存取操作
T& at(int idx); // 返回索引idx所指的数据，如果idx越界，抛出out_of_range异常
T& operator[](int idx); // 返回索引idx所指的数据，如果idx越界，运行直接报错
​
T& front(); // 返回首元素的引用
T& back(); // 返回尾元素的引用
vector插入和删除操作
insert(const_iterator pos, T elem); // 在pos位置处插入元素elem
insert(const_iterator pos, int n, T elem); // 在pos位置插入n个元素elem
insert(pos, beg, end); // 将[beg, end)区间内的元素插到位置pos
push_back(T elem); // 尾部插入元素elem
pop_back(); // 删除最后一个元素
​
erase(const_iterator start, const_iterator end); // 删除区间[start, end)内的元素
erase(const_iterator pos); // 删除位置pos的元素
​
clear(); // 删除容器中的所有元素
至此，读者应当对vector的特点及基本操作有了较为全面的认识，使用时API记不清可以回头多看。