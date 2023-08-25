
#### 本文归纳总结刷题常用到 STL 容器以及一些标准算法，主要包括：

```c++
//part1.序列容器
string
vector
queue
deque
stack
list  
//part2.关联容器
set
map
unordered_map

堆
heap
```

以及这些容器的常用操作：  
如插入、删除、查找、访问方式（迭代器 or 下标，C++11 关键字 auto 了解吗？顺序访问 or 随机访问）、初始化等。

### 一、序列容器

**常用容器：** [vector](https://so.csdn.net/so/search?q=vector&spm=1001.2101.3001.7020)、deque、list、queue、stack  
**概念：** 序列是对基本容器的一种改进，在保持其基础功能上增加一些我们需要的更为方便的功能。  
**要求：** 序列的元素必须是严格的线性顺序排序。因此序列中的元素具有确定的顺序，可以执行将值插入到特定位置、删除特定区间等操作。  
**序列容器基本特征：** 以下用 t 表示类型为 T（储存在容器中的值的类型）的值，n 表示整数，p、q、i 和 j 表示迭代器。  

![](https://img-blog.csdnimg.cn/2019071520340771.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbnRhY3kxMDAwMA==,size_16,color_FFFFFF,t_70)

  

![](https://img-blog.csdnimg.cn/20190715203429605.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbnRhY3kxMDAwMA==,size_16,color_FFFFFF,t_70)

### 二、关联容器

关联容器与序列容器有着根本性的不同，序列容器的元素是按照在容器中的位置来顺序保存和访问的，而关联容器的元素是按关键元素来保存和访问的。关联容器支持高效的关键字查找与访问。两个主要的关联容器类型是 map 与 set。

### 1.【string】字符串类型

```
1.插入操作： 
str.insert(pos,str)；                       //在下标pos处插入字符串str 

2.删除操作：
(1) 删除单个元素 str.erase(iter);            //删除迭代器iter指定的元素 
(2) 删除区间元素：
① str.erase(first_iter, last_iter)          //删除迭代器[first,second)之间的元素
② str.erase(pos,length)                     //删除下标pos开始的长度为length的区间元素 

3.截取字符串： 
string newstr = str.substr(pos,length)；    //截取从下标pos开始的长度为length的子串 

4.查找操作： 
1)str.find(str2)；               //如果str2是str的子串，则返回str2在str中首次出现的位置；否则，返回string::npos，这个值是-1 
2)str.find(str2,pos)             //从str下标为pos的位置开始匹配sr2 

注意，如果我在字符串str="3.1415"中查询小数点出现的位置，应该写成 str.find(".") 而不是 str.find('.') ，后者查找的是char型字符，而不是string型，是没有这种写法的！

5.字符串转数字函数： 
stoi()；          //如string str="123"，可以int val=stoi(str)；

6.string类型支持下标访问和迭代器访问，不过一般就用下标[]访问，和C的写法一样 。
另外，C++11之后，如果要完整遍历一个string，还可以用下面的写法。

7.数字转字符串函数：to_string();
```

```
示例代码：
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <string>
using namespace std;

int main() {
    //string对象的创建
    string str1("aaa");
    string str2="bbb";
    char buf[10]="hello";
    string str3(buf);     //这种初始化方法我一开始还以为不可以的呢。
    cout<<str1<<' '<<str2<<' '<<str3<<endl<<endl;

    //insert
    string s="kill PAT";
    cout<<"before insert: "<<s<<endl<<endl;
    s.insert(5,"fucking ");                   //在原序列s[5]处插入
    cout<<"after insert: "<<s<<endl<<endl;

    //遍历操作
    for(auto ch:s)
        cout<<ch;
    cout<<endl<<endl;

    //erase删除操作
    string s2="James go to Lakers Q";
    cout<<s2<<endl;
    s2.erase(s2.end()-1);//删除Q
    cout<<s2<<endl;
    s2.erase(s2.begin(),s2.begin()+6);
    cout<<s2<<endl;
    s2.erase(0,6);
    cout<<s2<<endl<<endl;

    //substr
    string s3="James go to Lakers Q";
    string subStr=s3.substr(0,5)+" NB";
    cout<<subStr<<endl<<endl;

    //find
    cout<<s3.find("s")<<endl;//4
    cout<<s3.find("s",5)<<endl;//17
    if(s3.find(".")==string::npos) cout<<"Not find\n";//或写成数s3.find(".")==-1
    else cout<<"Found\n";
    return 0;
}
```

### 2.【vector】数组

这个是用的最多的，基本就拿来当数组来用。  
vector 是最简单也是最重要的一个容器。其头文件为 vector  
vector 是数组的一种类表示，它有以下优点：  
[1] 自动管理内存、动态改变长度并随着元素的增减而增大或缩小。  
[2] 在尾部添加元素是固定时间，在头部或中间添加或删除元素是线性时间。

```
基本操作
vector<int> a;         //声明

a.front();
a.back();
a.push_back();
a.pop_back();
a[n]
a.at(t)

注：
*a[n]和a.at(n)都返回一个指向容器中第n个元素的引用。
区别在于：如果n落在容器有效区间之外，a.at(n)将执行边界检查，并引发out_of_range异常。
*之所以vector没有push_front()，是因为vector执行此表达式复杂度为线性时间，而deque为固定时间。
```

要稍微讲一下二维数组的提前申请空间，如下：

```
int main() {
    vector<vector<int>> matrix;        //假设这是一个4*3的矩阵
    int row=4,col=3;
    matrix.resize(row);
    for(int i=0;i<matrix.size();i++)
        matrix[i].resize(col);
    //接下来就可以用下标运算访问了
    matrix[1][2]=233;
    matrix[0][0]=422;
    for(int i=0;i<matrix.size();i++){
        for(int j=0;j<matrix[i].size();j++){
            printf("%d ",matrix[i][j]);
        }
        printf("\n");
    }
    return 0;
}
```

### 3.【queue】队列

头文件 queue  
queue 不允许随机访问队列元素，不允许遍历队列，可以进行队列基本操作；  
可以将元素添加到队尾，从队首删除元素，查看队尾和队首的值，检查元素数目和测试队列是否为空；

```
基本操作
queue<int> a;    //声明一个空队列

a.empty()        //判断队列是否为空，如果为空则返回 1，不为空则返回 0 
a.size()         //返回队列元素个数
a.front()        //返回指向队首元素的引用
a.back()         //返回指向队尾元素的引用
a.push(t)        //队尾插入元素
a.pop()          //删除队首元素

注：
pop() 是一个删除数据的方法，不是检索数据的方法！！！
如果要使用队列中的值，首先要使用 front() 来检索这个值，然后用 pop() 将他从队列中删除。
```

### 4.【deque】双端队列（double ended queue）

头文件 deque  
在 STL 中 deque 类似 vector，并且支持随机访问；  
区别在于：  
【1】从 deque 起始位置插入删除元素时间是固定的；  
【2】为了实现在 deque 俩段执行插入和删除操作的时间为固定这一目的，deque 对象设计比 vector 设计更为复杂一些。因此，在序列中部执行插入删除操作时，vector 更快一些；

```
基本操作
deque<int> a;           //声明一个空队列

a.empty()               //判断队列是否为空，如果为空则返回 1，不为空则返回 0 
a.size()                //返回队列元素个数
a.front()               //返回指向队首元素的引用
a.back()                //返回指向队尾元素的引用
a.push_front()          //从队首插入
a.push_back()           //从队尾插入
a.push(t)               //队尾插入元素
a.pop()                 //删除队首元素
a[n]
a.at(t)

注：
pop() 是一个删除数据的方法，不是检索数据的方法！！！
如果要使用队列中的值，首先要使用 front() 来检索这个值，然后用 pop() 将他从队列中删除。
```

### 5.【stack】堆栈

头文件 stack  
stack 是一个适配器，它给底层类（默认 vector）提供典型栈接口；  
stack 不允许随机访问栈元素，不允许遍历栈，把使用限制在定义栈的基本操作上；  
可以将值压入栈顶，从栈顶弹出元素，查看栈顶的值，检查元素数目，测试栈是否为空；

```
基本操作
stack<int> a;        //声明

a.empty()	         //如果栈为空，返回true，否则返回false
a.size()	         //返回栈中元素数目
a.top()	             //返回指向栈顶元素的引用
a.push(t)	         //在栈顶插入x
a.pop()	             // 删除栈顶元素

注：
pop() 是一个删除数据的方法，不是检索数据的方法！！！
如果要使用栈顶的值，首先要使用 top() 来检索这个值，然后用 pop() 将他从栈顶删除。
```

### 6.【set】集合

特点：  
【1】储存同一类型的数据元素（这点和 vector、queue 等其他容器相同）；  
【2】每个元素的值都唯一（没有重复的元素）；  
【3】根据元素的值自动排列大小（有序性）；  
【4】无法直接修改元素；  
【5】高效的插入删除操作；

```
基本操作
set<int> a;            //声明

a.begin();            //返回指向第一个元素的迭代器
a.end();              //返回指向超尾的迭代器
a.clear();            //清空容器a
a.empty();            //判断容器是否为空
a.size();             //返回当前容器元素个数
a.count(x);           //返回容器a中元素x的个数
a.insert(x);          //插入元素，其中a为set<T>型容器，x为T型变量

了解
a.insert(first,second);       //其中first为指向区间左侧的迭代器，second为指向右侧的迭代器。作用是将first到second区间内元素插入到a（左闭右开）。
a.erase(x)：删除建值为x的元素
a.erase(first,second)：删除first到second区间内的元素（左闭右开）
a.erase(iterator):删除迭代器指向的元素
注：set中的删除操作是不进行任何的错误检查的，比如定位器的是否合法等等，所以用的时候自己一定要注意。
lower_bound（x1）:返回第一个不小于键参数x1的元素的迭代器
upper_bound（x2）:返回最后一个大于键参数x2的元素的迭代器
由以上俩个函数，可以得到一个目标区间，即包含集合中从'x1'到'x2'的所有元素
set_union():对集合取并集
set_intersection():对集合取交集，它的接口与set_union()相同。
```

### 7.【map && unordered_map】映射

分别在头文件和 <unordered_map> 中，别混了。  
这两个刷题时用的也非常多，其中 map 的底层实现是 RB-Tree（红黑树），因此会根据键值自动进行排序，默认是字典序。而 unordered_map 的底层实现是哈希。因此，如果只是单纯的用来创建某种映射关系的话，推荐 unordered_map，效率会高一些。

**map 是一种 key-value 型容器，其中 key 是关键字，起到索引作用，而 value 就是其对应的值。与 set 不同的是它支持下标访问。头文件是 map**

特点：  
【1】增加和删除节点对迭代器的影响很小 (高效的插入与删除)；  
【2】快速的查找（同 set）；  
【3】自动建立 key-value 的对应，key 和 value 可以是任何你需要的类型；  
【4】可以根据 key 修改 value 的记录；  
【5】支持下标 [] 操作

```
map<string,int> m;         //声明一个key为string，value为int的map型容器
```

**常用方法：**  

![](https://img-blog.csdnimg.cn/20190715214827796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbnRhY3kxMDAwMA==,size_16,color_FFFFFF,t_70)

### 8.【heap】最大堆和最小堆

头文件是 algorithm

```
一般用到这四个：make_heap()、pop_heap()、push_heap()、sort_heap()；

（1）make_heap()        //构造堆 
void make_heap(first_pointer,end_pointer,compare_function); 
默认比较函数是(<)，即最大堆。 
函数的作用是将[begin,end)内的元素处理成堆的结构

（2）push_heap()        //添加元素到堆 
void push_heap(first_pointer,end_pointer,compare_function); 
新添加一个元素在末尾，然后重新调整堆序。该算法必须是在一个已经满足堆序的条件下。 
先在vector的末尾添加元素，再调用push_heap

（3）pop_heap()         //从堆中移出元素 
void pop_heap(first_pointer,end_pointer,compare_function); 
把堆顶元素取出来，放到了数组或者是vector的末尾。 
要取走，则可以使用底部容器（vector）提供的pop_back()函数。 
先调用pop_heap再从vector中pop_back元素

（4）sort_heap()        //对整个堆排序 
排序之后的元素就不再是一个合法的堆了。
```

```
void testHeap() {
    vector<int> data{ 3,1,2,7,5 };
    //构造堆，最大堆
    make_heap(data.begin(), data.end(), less<int>());
    //pop堆顶元素，最大的元素
    pop_heap(data.begin(), data.end(), less<int>());
    cout << data.back() << endl;//输出7
    data.pop_back();
    //往堆中添加元素
    data.push_back(4);
    push_heap(data.begin(), data.end(), less<int>());//调整堆
    //排序
    sort_heap(data.begin(), data.end(), less<int>());
    for (int x : data)
        cout << x << " ";
    cout << endl;//输出 1，2，3，4，5
}
```

### 9.【list】链表

```
链表的基本操作：

list<int> numbers;                //链表初始化
for(int i=0; i<n; i++ )
    numbers.push_back(i);         //链表的建立

numbers.begin();                  //链表头元素
numbers.end();                    //链表尾元素
numbers.erase(current);           //删除链表元素
numbers.size();                   //链表尺寸
```

### STL 容器之间的差异和联系

1.vector （连续的空间存储, 可以使用 [] 操作符）快速的访问随机的元素，快速的在末尾插入元素，但是在序列中间的插入，删除元素要慢（涉及元素复制移动），而且如果一开始分配的空间不够的话，有一个重新分配更大空间，此时需要拷贝的性能开销

2.deque 在开始和最后添加删除元素都一样快, 并提供了随机访问方法, 像 vector 一样使用 [] 访问任意元素, 但是随机访问速度比不上 vector 快, 因为它要内部处理堆跳转 deque 也有保留空间. 另外, 由于 deque 不要求连续空间, 所以可以保存的元素比 vector 更大, 这点也要注意一下. 还有就是在前面和后面添加元素时都不需要移动其它块的元素。对 deque 的排序操作，可将 deque 先复制到 vector，排序后在复制回 deque。  
1) 两端都能快速插入元素和删除元素（vector 只在尾端快速进行此类操作）。  
2）存取元素时，deque 的内部结构会多一个间接过程，所以元素的存取和迭代器的动作会稍稍慢一些。  
3）迭代器需要在不同区块间跳转，所以必须是特殊的智能型指针，非一般指针。  
4）在对内存区块有所限制的系统中（例如 PC 系统），deque 可以内含更多元素，因为它使用不止一块内存。因此 deque 的 max_size() 可能更大。  
5）deque 不支持对容量和内存重分配时机的控制。特别要注意的是，除了头尾两端，在任何地方插入或删除元素，都将导致指向 deque 元素的任何指针、引用、迭代器失效。不过，deque 的内存重分配优于 vector，因为其内部结构显示，deque 不必在内存重分配时复制所有元素。  
6）deque 的内存区块不再被使用时，会被释放。deque 的内存大小是可缩减的。

3.list（元素间使用链表相连）访问随机元素不如 vector 快，随机的插入元素比 vector 快，对每个元素分配空间，所以不存在空间不够，重新分配的情况。list 可以快速地在所有地方添加删除元素, 但是只能快速地访问最开始与最后的元素

4.set 内部元素唯一，用一棵平衡树结构来存储，因此遍历的时候就排序了，查找比较快。

5.map 一对一的映射的结合，key 不能重复。

6.stack 适配器，必须结合其他的容器使用，stl 中默认的内部容器是 deque。先进后出，只有一个出口，不允许遍历。

7.queue 适配器。是受限制的 deque，内部容器一般使用 list 较简单。先进先出，不允许遍历和元素的随机访问。

需要说明的是：  
由于 deque 可以从首位两端插入或剔除元素，所以只需要对其进行简单的封装就可以分别实现先进先出（FIFO）的 stack 和先进后出（FILO）的 queue 了。stack 和 queue 中都有一个 deque 类型的成员，用做数据存储的容器，然后对 deque 的部分接口进行简单的封装，例如 stack 只提供从末端插入和删除的接口以及获取末端元素的接口，而 queue 则只提供从尾部插入而从头部删除的接口以及获取首位元素的接口。像这样具有 “修改某物接口，形成另一种风貌” 的性质的，称为配接器（adapter），因此 STL 中 stack 和 queue 往往不被归类为容器（container），而被归类为容器配接器（container adapter）。