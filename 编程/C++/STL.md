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

# 容器
### string 字符串

## string 和 C风格字符 串对比

- `char*`是一个指针，`string`是一个类
    `string`封装了`char*`，管理这个字符串，是一个`char*`型的容器。
    

- `string` 封装了很多实用的成员方法
    查找`find`，拷贝`copy`，删除`erase`，替换`replace`，插入`insert`......
    

- 不用考虑内存释放和越界
    `string`管理`char*`所分配的内存，每一次`string`的复制/赋值，取值都由`string`类负责维护，不用担心复制越界和取值越界等。

> `string` 本质上是一个动态的 char 数组。

## string 容器常用操作

### string 构造函数