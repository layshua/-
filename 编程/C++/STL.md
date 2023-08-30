
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

