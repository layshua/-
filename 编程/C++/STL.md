


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

