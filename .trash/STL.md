

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

