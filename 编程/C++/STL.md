[仿函数(Functor) - C++ STL Tutorial (gitbook.io)](https://cui-jiacai.gitbook.io/c++-stl-tutorial/fang-han-shu-functor)
# 关于容器

## STL的主要容器

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


## 容器特点总结

|容器|vector |deque|list|set|multiset|map|multimap|
|---|---|---|---|---|---|---|---|
|典型内存结构|单端数组|双端数组|双向链表|二叉树|二叉树|二叉树|二叉树|
|可随机存储|是|是|否|否|否|对 key 而言，不是|否|
|元素搜索速度|慢|慢|非常慢|快|快|对 key 而言，快|对 key 而言，快|
|元素安插移除|尾端|头尾两端|任何位置|-|-|-|-|

## 使用场景

vector 可以涵盖其他所有容器的功能，只不过实现特殊功能时效率没有其他容器高。但如果只是简单存储，vector 效率是最高的。

deque 相比于 vector 支持头端元素的快速增删。

**vector 与 deque 的比较**

- `vector.at()` 比 `deque.at()` 的效率高
    - 比如 `vector.at(0)` 是固定的，`deque` 的开始位置是不固定的

- 如果有大量释放操作的话，`vector` 花的时间更少
- `deque` 支持头部的快速插入与快速删除，这是 `deque` 的优点

list 支持频繁的不确定位置元素的移除插入。
set 会自动排序。
map 是元素为键值对组并按键排序的 set。
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

## 谓词—算法的自定义操作条件

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

# vector - 可变数组

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

### 判断迭代器是否能随机访问的方法
用多了自然就背上了，下面给出一种现场测试的方法。
```c++
iterator++；
iterator--；
//通过编译，至少是双向迭代器
  
iterator = iterator + 1；
//通过编译，则是随机访问迭代器
```

## vector常用API操作
### vector 构造函数
```c++
vector<T> v; // 采用模版类实现，默认构造函数
vector<T> v(T* v1.begin(), T* v1.end()); // 将v1[begin(), end())区间中的元素拷贝给本身
vector<T> v(int n, T elem); // 将n个elem拷贝给本身
vector<T> v(const vector<T> v1); // 拷贝构造函数
```
下面对于第二种构造方式给出一个特殊的例子：
```c++
int array[5] = {1, 2, 3, 4, 5};
vector<int> v(array, array + sizeof(array) / sizeof(int));
// 联系我们之前提到的vector迭代器本质上是指针就不难理解了
```

### vector 常用赋值操作
由于 vector 采用模版类实现，其完整的函数声明会稍显复杂，下面方法的演示会省略类型界定。

```c++
assign(beg, end); // 将[beg, end)区间中的数据拷贝复制给本身
assign(n, elem); // 将n个elem拷贝给本身
vector& operator=(const vector& vec); // 重载赋值操作符
```

互换操作也可视为一种特殊的赋值：
```c++
swap(vec); //将vec与本身的元素互换
```

**巧用swap来收缩空间：**
```c++
vector<int>(v).swap(v);
// vector<int>(v): 利用拷贝构造函数初始化匿名对象
// swap(v): 交换的本质其实只是互换指向内存的指针
// 匿名对象指针指向的内存会由系统自动释放
```

### vector 大小操作
```c++
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
```

### vector 数据存取操作
```c++
T& at(int idx); // 返回索引idx所指的数据，如果idx越界，抛出out_of_range异常
T& operator[](int idx); // 返回索引idx所指的数据，如果idx越界，运行直接报错
​
T& front(); // 返回首元素的引用
T& back(); // 返回尾元素的引用
```

### vector 插入和删除操作
```c++
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
```

# deque - 双向队列 
STL中deque类的详细介绍
## duque 容器基本概念
vector 容器是单向开口的连续内存空间，deque 则是一种双向开口的连续线性空间。
所谓的双向开口，意思是可以在头尾两端分别做元素的插入和删除操作
vector 虽然也能在头尾插入元素，但是在头部插入元素的效率很低，需要大量进行移位操作
![[Pasted image 20230825103846.jpg]]
## deque 容器和 vector 最大的差异
deque **允许使用常数项时间在头部插入或删除元素**
deque 没有容量的概念，因为它是由动态的分段连续空间组合而成，随时可以增加一块新的空间并链接起来
虽然deque也提供了 Random Access Iterator，但其实现相比于vector要复杂得多，所以**需要随机访问的时候最好还是用vector**。
## deque 容器实现原理
![[Pasted image 20230825103917.jpg]]
## deque 的遍历
基本的遍历方式同 vector，不做赘述。这里提一下如何在遍历的时候防止内容被修改: 用 const_iterator 指向常量容器
```c++
#include <deque>
using namespace std;
​
const deque<T> d;
for (deque<T>::const_iterator it = d.begin(); it != d.end(); it++) 
// 要用const_iterator指向常量容器
{
  	// 如果在此处修改it指向空间的值，编译器会报错
  	cout << *it << endl;
}
​
/**
  * iterator 普通迭代器
  * reverse_iterator 反转迭代器
  * const_iteratoe 只读迭代器
*/
```

## deque 常用 API
### deque 构造函数
```c++
deque<T> deqT; // 默认构造函数
deque(beg, end); // 构造函数将[beg, end)区间中的元素拷贝给本身
deque(int n, T elem); // 构造函数将n个elem拷贝给本身
deque(const deque& deq); // 拷贝构造函数
```
### deque 赋值操作
```c++
assign(beg, end); // 将[beg, end)区间中的元素拷贝赋值给本身
assign(int n, T elem); // 将n个元素elem拷贝赋值给本身
​
deque& operator=(const deque& deq); // 重载赋值操作符
​
swap(deq); // 将deq与本身的元素互换
```
### deque 大小操作
```c++
int size(); // 返回容器中元素的个数
bool empty(); // 判断容器是否为空
​
void resize(int num); 
// 重新指定容器的长度为num，若容器变长，则以默认值填充新位置，
// 如果容器变短，则末尾超出容器长度的元素被删除
void resize(int num, T elem);
// 重新指定容器的长度为num，若容器变长，则以elem填充新位置，
// 如果容器变短，则末尾超出容器长度的元素被删除
```
### deque 的双端插入和删除操作
```c++
push_back(T elem); // 在容器尾部添加一个元素
push_front(T elem); // 在容器头部插入一个元素
​
pop_back(); // 删除容器最后一个数据
pop_front(); // 删除容器第一个数据
```

### deque 的数据存取
```c++
T& at(int idx); // 返回索引idx所指的数据，如果idx越界，抛出out_of_range异常
T& operator[](int idx); // 返回索引idx所指的数据，如果idx越界，运行直接报错
​
T& front(); // 返回首元素的引用
T& back(); // 返回尾元素的引用
```

### deque 插入操作
```c++
const_iterator insert(const_iterator pos, T elem); 
// 在pos位置处插入元素elem的拷贝，返回新数据的位置
void insert(const_iterator pos, int n, T elem); 
// 在pos位置插入n个元素elem，无返回值
void insert(pos, beg, end);
// 将[beg, end)区间内的元素插到位置pos，无返回值
```
### deque 删除操作
```c++
clear(); // 移除容器的所有数据
iterator erase(iterator beg, iterator end);
// 删除区间[beg, end)的数据，返回下一个数据的位置
iterator erase(iterator pos)；
// 删除pos位置的数据，返回下一个数据的位置
```





# list - 链表
## list 容器基本概念
链表是一种物理存储单元上非连、续非顺序的储存结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。
- 链表由一系列结点（链表中每一个元素称为结点）组成
- 结点可以在运行时动态生成
- 每个结点包括两个部分
    - 储存数据元素的数据域
    - 储存下一个结点地址的指针域
    - 链表灵活，但是空间和时间的额外消耗会比较大。


相较于vector的连续线性空间，list就显得复杂许多。
- 它的好处是每次插入或者删除一个元素，就是配置或者释放一个元素的空间
- 因此，list 对于空间的运用有绝对的精准，一点也不浪费
- 而且，list 对于任何位置插入或删除元素都是常数项时间

list 容器是一个双向链表：
![[Pasted image 20230825104451.jpg]]

## list 容器的迭代器
list 容器不能像vector一样以普通指针作为迭代器，因其节点不能保证在同一块连续的内存空间上。
list 迭代器必须有能力指向list的结点，并有能力进行正确的递增、递减、取值、成员存取操作。
由于 list 是一个双向链表，迭代器必须能够具备前移、后移的能力，所以 list 容器提供的是**双向迭代器**。

**list 有一个重要的性质，插入和删除操作都不会造成原有 list 迭代器的失效**。这在 vector 是不成立的，因为 vector 的插入操作可能会造成内存的重新配置，导致原有的迭代器全部失效
**而 list 元素的删除只会使得被删除元素的迭代器失效**

## list 容器的顺序遍历
```c++
for(list<T>::iterator it = lst.begin(); it != lst.end(); it++)
{
    cout << *it << endl;
}
```
## list 容器的逆序遍历
```c++
for(list<T>::reverse_iterator it = lst.rbegin(); it != lst.rend(); it++)
{
    cout << *it << endl;
}
```
## list 容器的数据结构
**list不仅仅是一个双向链表，而且是一个循环的双向链表。**
## list 常用API
### list 构造函数
```c++
list<T> lstT; // 默认构造形式，list采用模版类实现
list(beg, end); // 构造函数将[beg, end)区间内的元素拷贝给本身
list(int n, T elem); // 构造函数将n个elem拷贝给本身
list(const list& lst); // 拷贝构造函数
```
### list 数据元素插入和删除操作
```c++
void push_back(T elem); // 在容器尾部加入一个元素
void pop_back(); // 删除容器中最后一个元素
​
void push_front(T elem); // 在容器开头插入一个元素
void pop_front(); // 从容器开头移除第一个元素
​
insert(iterator pos, elem); // 在pos位置插入elem元素的拷贝，返回新数据的位置
insert(iterator pos, n, elem); // 在pos位置插入n个elem元素的拷贝，无返回值
insert(iterator pos, beg, end); // 在pos位置插入[beg, end)区间内的数据，无返回值
​
void clear(); // 移除容器的所有数据
​
erase(beg, end); // 删除[beg, end)区间内的所有数据，返回下一个数据的位置
erase(pos); // 删除pos位置的数据，返回下一个数据的位置
​
remove(elem); // 删除容器中所有与elem匹配的元素
```
### list 大小操作
```c++
int size(); // 返回容器中元素的个数
bool empty(); // 判断容器是否为空
​
void resize(int num);
// 重新制定容器的长度为num，若容器变长，则以默认值填充新位置；
// 若容器变短，则末尾超出容器长度的元素被删除
void resize(int num, T elem);
// 重新制定容器的长度为num，若容器变长，则以elem填充新位置；
// 若容器变短，则末尾超出容器长度的元素被删除
```
### list 赋值操作
```c++
assign(beg, end); // 将[beg, end)区间中的数据拷贝赋值给本身
assign(n, elem); // 将n个elem拷贝赋值给本身
​
list& operator=(const list& lst); // 重载等号操作符
​
swap(lst); // 将lst与本身的元素互换
```
#### list 数据的存取
```c++
T& front(); // 返回第一个元素
T& back(); // 返回最后一个元素
```
#### list 反转排序
```c++
void reverse(); // 反转链表
​
void sort(); // 默认list排序，规则为从小到大
void sort(bool (*cmp)(T item1, T item2)); // 指定排序规则的list排序
​
// 不能用sort(lst.begin(), lst.end())
// 因为所有系统提供的某些算法（比如排序），其迭代器必须支持随机访问
// 不支持随机访问的迭代器的容器，容器本身会对应提供相应的算法的接口
```

# set / multiset - 集合

STL中set/multiset类的详细介绍

## set/multiset 容器基本概念

### set 容器基本概念

set的特性是，所有的容器都会根据元素自身的键值进行自动被排序。

set的元素不像map那样可以同时拥有实值和键值，**set的元素既是实值又是键值**。

- set 不允许两个元素有相同的键值
- 我们不可以通过 set 的迭代器改变 set 元素的值。因为其元素值就是键值，任意改变会严重破坏set的组织
- 换句话说，set的iterator是一种const_iterator

set和list拥有某些相同的性质，当对容器中的元素进行插入操作或者删除操作的时候，除了被删除的元素之外，操作之前的所有迭代器，操作之后仍然有效。

### multiset 基本概念

- multiset特性及用法和set完全相同，唯一的差别在于它允许键值重复。
- set和multiset的底层实现是**红黑树**，红黑树为平衡二叉树的一种

> 注意，multiset和set共用一个头文件。


## set 的遍历
```c++
for (set<T>::iterator it = s.begin(); it != s.end(); it++)
{
  	cout << *it << endl;
}
```
## set 常用API
### set 构造函数
```c++
set<T> st; // set 默认构造函数
multiset<T> mst; // multiset 默认构造函数
set(const set& st); // 拷贝构造函数
```
### set 赋值操作
```c++
set& operator=(const set& st); // 重载等号操作符

swap(st); // 交换两个集合容器
```
### set 大小操作
```c++
int size(); // 返回容器中元素的数目
bool empty(); // 判断容器是否为空
```
### set 插入和删除操作
```c++
pair<iterator, bool> insert(T elem); 
// 在容器中插入元素，返回插入位置的迭代器（不成功则返回end()）和是否插入成功
// 如果是multiset，则返回值只有iterator
clear(); // 清除所有元素
iterator erase(pos); // 删除pos迭代器所指的元素，返回下一个元素的迭代器
iterator erase(beg, end); // 删除区间[beg, end)内的所有元素，返回下一个元素的迭代器
erase(T elem); // 删除容器中值为elem的元素
```
插入之前可以指定排序规则：
```c++
//利用仿函数 指定set容器的排序规则
class MyCompare
{
public:
    bool operator()(int v1, int v2)
    {
        return v1 > v2;
    }
};

set<int, MyCompare> s;

//模版类也是可以有默认值的，第二个模版参数的默认值为less
```
自定义的数据类型需要指出排序规则。
当然，也可以通过重载小于操作符的方式指出。
## set 查找操作
```c++
iterator find(T key); 
// 查找键key是否存在，若存在，返回该键的元素的迭代器；若不存在，返回set.end();
int count(T key);
// 查找键key的元素个数
iterator lower_bound(T keyElem);
// 返回第一个key>=keyElem元素的迭代器
iterator upper_bound(T keyElem);
// 返回第一个key>keyElem元素的迭代器
pair<iterator, iterator> equal_range(T keyElem);
// 返回容器中key与keyElem上相等的两个上下限迭代器
```

上述几个方法若不存在，返回值都是尾迭代器。
对组的构造和使用：

```c++
//构造
pair<T1, T2> p(k, v);
//另一种构造方式
pair<T1, T2> p = make_pair(k, v);
//使用
cout << p.first << p.second << endl;
```

# map / multimap - 映射
## map / multimap 基本概念
map 的特性是，所有的元素都会根据元素的键值自动排序；
map 的所有元素都是`pair`，同时拥有实值和键值。
- pair 的第一元素被视为键值，第二元素被视为实值；
- map 不允许两个元素有相同的键值；
**和set类似的原因，我们不能通过迭代器改变map的键值，但我们可以任意修改实值。**
map和list在增删元素的时候具有相似的性质。
map和multimap的操作类似，唯一的区别是multimap键值可重复。
map和multimap都是以**红黑树**作为底层实现机制。
map 和 multimap 包含在同一个头文件中。

## map 的遍历
```c++
for (map<T1, T2>::iterator it = m.begin(); it != m.end(); it++)
{
    cout << "key = " << it->first << " value = " << it->second << endl;
}
```
## map/multimap 常用API
### map 构造函数
```c++
map<T1, T2> mapTT; // map默认构造函数
map(const map& mp); // 拷贝构造函数
```
### map 赋值操作
```c++
map& operator=(const map& mp); // 重载等号操作符
swap(mp); // 交换两个集合容器
```
### map 大小操作
```c++
int size(); // 返回容器中元素的数目
bool empty(); // 判断容器是否为空
```
### map 插入元素操作
```c++
pair<iterator, bool> insert(pair<T1, T2> p); // 通过pair的方式插入对象
/*
1. 参数部分可以用pair的构造函数创建匿名对象
2. 也可以使用make_pair创建pair对象
3. 还可以用map<T1, T2>::value_type(key, value)来实现
*/
​
T2& operator[](T1 key); // 通过下标的方式插入值
// 如果通过下标访问新的键却没有赋值，会自动用默认值填充
```
map指定排序规则的方式和set类似，都是利用functor在模版类型表的最后一个参数处指定。
### map 删除操作
```c++
void clear(); // 删除所有元素
iterator erase(iterator pos); // 删除pos迭代器所指的元素，返回下一个元素的迭代器
iterator erase(beg, end); // 删除区间[beg, end)内的所有元素，返回下一个元素的迭代器
erase(keyElem); // 删除容器中key为keyElem的对组
```
### map 查找操作
```c++
iterator find(T1 key); 
// 查找键key是否存在，若存在，返回该键的元素的迭代器；若不存在，返回map.end()
int count(T1 keyElem);
// 返回容器中key为keyElem的对组个数，对map来说只可能是0或1，对于multimap可能大于1
​
iterator lower_bound(T keyElem);
// 返回第一个key>=keyElem元素的迭代器
iterator upper_bound(T keyElem);
// 返回第一个key>keyElem元素的迭代器
pair<iterator, iterator> equal_range(T keyElem);
// 返回容器中key与keyElem上相等的两个上下限迭代器
```


# 仿函数(Functor)
在详细讲常用的算法之前补充一下函数对象的相关内容，后面会用到。
## 函数对象 / 仿函数
1. **重载函数调用操作符的类，其对象常称为函数对象(function object)，也叫仿函数(functor)，使得类对象可以像函数那样调用。**
2. STL 提供的算法往往有两个版本，一种是按照我们常规默认的运算来执行，另一种允许用户自己定义一些运算或操作，通常通过回调函数或模版参数的方式来实现，此时 functor 便派上了用场，特别是作为模版参数的时候，只能传类型。
3. 函数对象超出了普通函数的概念，其内部可以拥有自己的状态(其实也就相当于函数内的 static 变量)，可以通过成员变量的方式被记录下来。
4. 函数对象可以作为函数的参数传递。
5. 函数对象通常不定义构造和析构函数，所以在构造和析构时不会发生任何问题，避免了函数调用时的运行时问题。
6. 模版函数对象使函数对象具有通用性，这也是它的优势之一。
7. STL 需要我们提供的 functor 通常只有一元和二元两种。
8. lambda 表达式的内部实现其实也是仿函数
## 谓词
返回值为 bool 的普通函数或者函数对象
在概述中有提到过，这里再重复一下。
## 内建函数对象
使用时需要包含头文件`<functional>`
STL 内建了一些函数对象，分为：
- 算术类函数对象
- 关系运算类函数对象
- 逻辑运算类函数对象

### 算术类函数对象
```c++
template<class T> T plus<T>; // 加法仿函数
template<class T> T minus<T>; // 减法仿函数
template<class T> T multiplies<T>; // 乘法仿函数
template<class T> T divides<T>; // 除法仿函数
template<class T> T modulus<T>; // 取模仿函数
template<class T> T negate<T>; // 取反函数
```
`negate` 是一元运算，其他都是二元运算。
### 关系运算类函数对象
```c++
template<class T> bool equal_to<T>; // 等于
template<class T> bool not_equal_to<T>; // 不等于
template<class T> bool greater<T>; // 大于
template<class T> bool greater_equal<T>; // 大于等于
template<class T> bool less<T>; // 小于
template<class T> bool less_equal<T>; // 小于等于
```
### 逻辑运算类运算函数
```c++
template<class T> bool logical_and<T>; // 逻辑与
template<class T> bool logical_or<T>; // 逻辑或
template<class T> bool logical_not<T>; // 逻辑非
```

## Bind 函数
### 说明

`bind1st()` 和 `bind2nd()`，在 C++11 里已经 deprecated 了，建议使用新标准的 `bind()`。  
下面先说明 `bind1st()` 和 `bind2nd()` 的用法，然后在说明 `bind()` 的用法。

### 头文件

`#include <functional>`

### 作用

`bind1st()` 和 `bind2nd()` 都是把二元函数转化为一元函数，方法是绑定其中一个参数。  
`bind1st()` 是绑定第一个参数。  
`bind2nd()` 是绑定第二个参数。

### 例子

```c++
#include <iostream>
#include <algorithm>
#include <functional>

using namespace std;

int main() {
    int numbers[] = { 10,20,30,40,50,10 };
    int cx;
    cx = count_if(numbers, numbers + 6, bind2nd(less<int>(), 40));
    cout << "There are " << cx << " elements that are less than 40.\n";

    cx = count_if(numbers, numbers + 6, bind1st(less<int>(), 40));
    cout << "There are " << cx << " elements that are not less than 40.\n";

    system("pause");
    return 0;
}
```

结果:

```c++
There are 4 elements that are less than 40.
There are 1 elements that are not less than 40.
```

分析  
`less()` 是一个二元函数，`less(a, b)` 表示判断 `a<b` 是否成立。

所以 `bind2nd(less<int>(), 40)` 相当于 `x<40` 是否成立, 用于判定那些小于 40 的元素。

`bind1st(less<int>(), 40)` 相当于 `40<x` 是否成立, 用于判定那些大于 40 的元素。

### bind ()

`bind1st()` 和 `bind2nd()`，在 C++11 里已经 deprecated 了. `bind()` 可以替代他们，且用法更灵活更方便。

上面的例子可以写成下面的形式：

```c++
#include <iostream>
#include <algorithm>
#include <functional>

using namespace std;

int main() {
    int numbers[] = { 10,20,30,40,50,10 };
    int cx;
    cx = count_if(numbers, numbers + 6, bind(less<int>(), std::placeholders::_1, 40));
    cout << "There are " << cx << " elements that are less than 40.\n";

    cx = count_if(numbers, numbers + 6, bind(less<int>(), 40, std::placeholders::_1));
    cout << "There are " << cx << " elements that are not less than 40.\n";

    system("pause");
    return 0;
}
```

结果:

```
There are 4 elements that are less than 40.
There are 1 elements that are not less than 40.
```

`std::placeholders::_1` 是占位符，标定这个是要传入的参数。  
所以 `bind()` 不仅可以用于二元函数，还可以用于多元函数，可以绑定多元函数中的多个参数，不想绑定的参数使用占位符表示。  
此用法更灵活，更直观，更便捷。

## 适配器
为了方便理解，我们以例子的形式来讲述这个部分的内容。
### 函数对象适配器
**原程序**
```c++
#include <iostream>
#inlcude <algorithm>
#include <vector>
using namespace std;
​
class myPrint
{
public:
    void operator()(int val) { cout << val << endl; }
}
​
int main()
{
    vector<int> v;
    
    for (int i = 0; i < 10; i++) 
        v.push_back(i);
        
    for_each(v.begin(), v.end(), myPrint());
    return 0;
}
```
**新需求**
我们希望在每个数据输出的时候加上一个基值，并且该基值由用户输入。
使用函数适配器做的改进
```c++
#include <iostream>
#inlcude <algorithm>
#include <vector>
#include <functional>
using namespace std;
​
class myPrint: public binary_function<int, int, void> 
// 2.做继承 参数1类型 + 参数2类型 + 返回值类型 binary_function
{
public:
    void operator()(int val, int base) const // 3. 加const, 和父类保持一致
    {
        cout << val + base << endl;
    }
}
​
int main()
{
    vector<int> v;
    for (int i = 0; i < 10; i++) v.push_back(i);
    int n;
    cin >> n;
    for_each(v.begin(), v.end(), bind2nd(myPrint(), n)); 
    // 1. 将参数进行绑定 bind2nd
    // bind1st 功能类似，不过n会被绑定到第一个参数中
    return 0;
};
```
### 取反适配器
**原程序**
```c++
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;
​
class GreaterThanFive
{
public:
    bool operator()(int val) { return val > 5; }
}
​
int main()
{
    vector<int> v;
    for (int i = 0; i < 10; i++) v.push_back(i);
    
    vector<int>::iterator pos = find_if(v.begin(), v.end(), GreaterThanFive());
    
    if (pos != v.end()) cout << *pos << endl;
    return 0;
}
```

**新需求**
我们希望找第一个不大于5的数，但又不想再写一个`LessEqualThanFive`。
使用取反适配器做的改进

```c++
 #include <iostream>
#include <algorithm>
#include <vector>
using namespace std;
​
class GreaterThanFive: public unary_function<int, bool> 
// 2. 做继承 参数类型 + 返回值类型 unary_function
{
public:
    bool operator()(int val) const // 3.加 const
    {
        return val > 5;
    }
}
​
int main()
{
    vector<int> v;
    for (int i = 0; i < 10; i++) v.push_back(i);
    
    vector<int>::iterator pos = find_if(v.begin(), v.end(), not1(GreaterThanFive()));  //1. 一元取反 not1
    
    if (pos != v.end()) cout << *pos << endl;
    return 0;
}
```

其实综合前面的内容会有更简便的方法：
```c++
vector<int>::iterator pos = find_if(v.begin(), v.end(), not1(bind2nd(greater<int>(), 5)));
```
这一行代码如果还半知半解的话还请驻足思考一番，思考透彻了上述内容才算是真的懂了。
### 函数指针适配器
沿用函数对象适配器的例子，假设myPrint是一个全局函数
```c++
for_each(v.begin(), v.end(), bind2nd(ptr_fun(myPrint), n));
// 函数指针适配器 ptr_fun 将函数指针适配成仿函数
```
### 成员函数适配器
我们假设有一个Dog类，Dog类内部有一个bark()成员方法，有一个装满了Dog的vector叫做v。
```c++
for_each(v.begin(), v.end(), mem_fun_ref(&Dog::bark));
// 成员函数适配器 mem_fun_ref
// 如果容器中存放的不是对象实体，而是对象指针时，则需使用 ptr_fun
```
## 偏函数/function 类型
对于一个多参数的函数，在某些应用场景下，它的一些参数往往取固定值，可以针对这样的函数，生成一个新函数，该新函数不包含原函数中已指定固定值的参数。（partial function application, 偏函数）
偏函数可缩小一个函数的适用范围，提高函数的针对性
例如，对于下面的print函数：
```c++
void print(int n, int base); // 按base进制来输出n
```
由于它常常用来按十进制输出，因此，可以基于print生成一个新函数print10，只接受一个参数n，base固定为10：
```c++
#include <functional>
using namespace std;
using namespace std::placeholders;
​
function<void(int)> print10 = bind(print, _1, 10);
print10(23); //相当于 print(23, 10)
```
function 类和 bind 的使用需要 c++11标准

至此，读者应当对于仿函数/函数对象有了一个稍微深入一点的理解了。如果还觉得有些云里雾里的话也没关系，以后熟悉了就好了。

