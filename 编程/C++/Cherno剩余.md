# 十八、cherno其他

##  多返回值

### 方法一：通过函数参数传引用或指针的方式

把函数定义成`void`，然后通过**参数引用传递**的形式 “返回” 两个字符串，这个实际上是修改了目标值，而不是返回值，但某种意义上它确实是返回了两个字符串，而且没有复制操作，技术上可以说是很好的。但这样做会使得函数的形参太多了，可读性降低，有利有弊 。

```cpp
#include <iostream>
void GetUserAge(const std::string& user_name,bool& work_status,int& age) {
    if (user_name.compare("xiaoli") == 0)
    {
        work_status = true;
        age = 18;
    }
    else
    {
        work_status = false;
        age = -1;
    }
}

int main() {
    bool work_status = false;
    int age = -1;
    GetUserAge("xiaoli", work_status, age);
    std::cout << "查询结果：" << work_status << "    " << "年龄：" << age << std::endl;
    getchar();
    return 0;
}
```

### 方法二： 通过函数的返回值是一个array或 vector

当然，这里也可以返回一个 vector，同样可以达成返回多个数据的目的。  
不同点**是 Array 是在栈上创建，而 vector 会把它的底层储存在堆上**，所以从技术上说，返回 Array 会更快  
但以上方法都**只适用于相同类型**的多种数据的返回

```cpp
//设置是array的类型是stirng，大小是2
std::array<std::string, 2> ChangeString() {
    std::string a = "1";
    std::string b = "2";

    std::array<std::string, 2> result;
    result[0] = a;
    result[1] = b;
    return result;

    //也可以return std::array<std::string, 2>(a, b);
}
```

### 方法三：使用 std::pair 返回两个返回值

可以**返回两个不同类型**的数据返。  
使用 std::pair 这种抽象数据结构，该数据结构可以绑定两个异构成员。这种方式的弊端是**只能返回两个值。**

```cpp
#include <iostream>

std::pair<bool, int> GetUserAge(const std::string& user_name)
{
    std::pair<bool, int> result;

    if (user_name.compare("xiaoli") == 0)
    {
        result = std::make_pair(true, 18);
    }
    else
    {
        result = std::make_pair(false, -1);
    }

    return result;
}

int main() {
    std::pair<bool, int> result = GetUserAge("xiaolili");
    std::cout << "查询结果：" << result.first << "   " << "年龄：" << result.second << std::endl;
    getchar();
    return 0;
}
```

### 方法四：使用 std::tuple 返回三个或者三个以上返回值

std::tuple 这种抽象数据结构可以将三个或者三个以上的异构成员绑定在一起，返回 std::tuple 作为函数返回值理论上可以返回三个或者三个以上的返回值。  
`tuple`相当于一个类，它可以包含 x 个变量，但他不关心类型，用`tuple`需要包含头文件`#include`

```cpp
#include <iostream>
#include <tuple>

std::tuple<bool, int,int> GetUserAge(const std::string& user_name)
{
    std::tuple<bool, int,int> result;

    if (user_name.compare("xiaoli") == 0)
    {
        result = std::make_tuple(true, 18,0);
    }
    else
    {
        result = std::make_tuple(false, -1,-1);
    }

    return result;
}

int main() {
    std::tuple<bool, int,int> result = GetUserAge("xiaolili");

    bool work_status;
    int age;
    int user_id;

    std::tie(work_status, age, user_id) = result;
    std::cout << "查询结果：" << work_status << "    " << "年龄：" << age <<"   "<<"用户id:"<<user_id <<std::endl;
    getchar();
    return 0;
}
```

### 方法五：返回一个结构体 (推荐)

结构体是在栈上建立的，所以在技术上速度也是可以接受的  
而且不像用 pair 的时候使用只能`temp.first, temp.second`，这样不清楚前后值是什么，可读性不佳。而如果换成`temp.str, temp.val`后可读性极佳，永远不会弄混！

```cpp
#include <iostream>
struct result {
    std::string str;
    int val;
};
result Function () {
    return {"1", 1};//C++新特性，可以直接这样子让函数自动补全成结构体
}
int main() {
    auto temp = Function();
    std::cout << temp.str << ' ' << temp.val << std::endl;
}
--------------------------------------------
#include <iostream>
using namespace std;

struct Result
{
    int add;
    int sub;
};

Result operation(int a, int b) {
    Result ret;
    ret.add = a + b;
    ret.sub = a - b;
    return ret;
}

int main() {
    Result res;
    res = operation(5, 3);
    cout << "5+3=" << res.add << endl;
    cout << "5-3=" << res.sub << endl;
}
```



### 方法六：【C++17】结构化绑定

1. 结构化绑定 struct binding 是 C++17 的新特性，能让我们更好地处理多返回值。可以在将函数返回为 tuple、pair、struct 等结构时且赋值给另外变量的时候，**直接得到成员**，而不是结构。


**老方法**（tuple、pair）

结构体方法这里不再演示，具体见之前的笔记。

```c++
#include <iostream>
#include <string>
#include <tuple>

// std::pair<std::string,int> CreatPerson() // 只能有两个变量
std::tuple<std::string, int> CreatPerson() // 可以理解为pair的扩展
{
    return {"Cherno", 24};
}

int main() {
    // 元组的数据获取易读性差，还不如像结构体一样直接XXX.age访问更加可读。
    // std::tuple<std::string, int> person = CreatPerson();
     auto person = CreatPerson(); //用auto关键字
     std::string& name = std::get<0>(person);
     int age = std::get<1>(person);

    //tie 可读性好一点
     std::string name;
     int age;
     std::tie(name, age) = CreatPerson();
}
```

**C++17 新方法：结构化绑定处理多返回值**

```c++
#include <iostream>
#include <string>
#include <tuple>

std::tuple<std::string, int> CreatPerson() 
{
    return {"Cherno", 24};
}

int main() {
    auto[name, age] = CreatPerson(); //直接用name和age来储存返回值
    std::cout << name;
}
```











## 线程（鸽）

1. 使用多线程，首先要添加头文件`#include <thread>`。

2. 在 **Linux 平台**下编译时需要加上 **"-lpthread" 链接库**

3. 创建一个线程对象：**`std::thread objName (一个函数指针以及其他可选的任何参数)`**

4. 等待一个线程完成它的工作的方法 :`worker.join()`

**这里的线程名字是 worker，换其他的也可以, 自己决定的）** 调用`join`的目的是：**在主线程上等待 工作线程 完成所有的执行之后，再继续执行主线程**

```cpp
//这个代码案例相当无用，只是为了展示多线程的工作而展示的。
#include <iostream>
#include <thread>
void DoWork() {
    std::cout << "hello" << std::endl;
}
int main() {
    //DoWork即是我们想要在另一个执行线程中发生的事情
    std::thread worker(DoWork); //这里传入的是函数指针！！！函数作为形参都是传函数指针！！！
    //一旦写完这段代码，它就会立即启动那个线程，一直运行直到我们等待他退出
    worker.join();  //join函数本质上，是要等待这个线程加入进来（而线程加入又是另一个复杂的话题了）

    //因为cin.get()是join语句的下一行代码，所以它不会运行，直到DoWork函数中的所有内容完成！
    std::cin.get();
}
#include <iostream>
#include <thread>

static bool is_Finished = false;
void DoWork() {
    using namespace std::literals::chrono_literals; //等待时间的操作可以先using一个命名空间，为 1s 提供作用域
    while (is_Finished) {
        std::cout << "hello" << std::endl;
        std::this_thread::sleep_for(1s);    //等待一秒
    }
}

int main() {
    std::thread worker(DoWork); //开启多线程操作

    std::cin.get(); //此时工作线程在疯狂循环打印，而主线程此时被cin.get()阻塞
    is_Finished = true;// 让worker线程终止的条件，如果按下回车，则会修改该值，间接影响到另一个线程的工作。

    worker.join();  //join:等待工作线程结束后，才会执行接下来的操作

    std::cin.get();
}
```

如果是正常情况，`DoWork`应该会一直循环下去，但因为这里是多线程，所以可以在另一个线程中修改工作线程的变量，来停止该线程的循环。 **多线程对于加速程序是十分有用的，线程的主要目的就是优化。**

## 计时（鸽）

**作用**:

计时的使用很重要。在逐渐开始集成更多复杂的特性时，如果编写性能良好的代码时，需要用到计时来看到差异。

**利用 chrono 类计时**：

1. 包含头文件**`#include`** 2. 获取当前时间：

```cpp
std::chrono::time_point<std::chrono::steady_clock> start = std::chrono::high_resolution_clock::now();
//或者，使用auto关键字
auto  start = std::chrono::high_resolution_clock::now();
auto  end = std::chrono::high_resolution_clock::now();
----------------------------------------------------------
//实例
#include <iostream>
#include <chrono>
#include <thread>

int main() {
    //literals：文字
    using namespace std::literals::chrono_literals; //有了这个，才能用下面1s中的's'
    auto start = std::chrono::high_resolution_clock::now(); //记录当前时间
    std::this_thread::sleep_for(1s);    //休眠1s，实际会比1s大。函数本身有开销。
    auto end = std::chrono::high_resolution_clock::now();   //记录当前时间
    std::chrono::duration<float> duration = end - start;    //也可以写成 auto duration = end - start; 
    std::cout << duration.count() << "s" << std::endl;
    return 0;
}
```

3. 获得时间差：

```cpp
std::chrono::duration<float> duration = end - start;
//或者
auto duration = end - start;
```

注意：在**自定义计时器类的构造函数、析构函数**中，**不要使用 auto 关键字**，应该在计时器类的构造函数、析构函数**前**定义 start、end、duration 变量。

```cpp
struct Timer   //写一个计时器类。
{
    std::chrono::time_point<std::chrono::steady_clock> start, end;
    std::chrono::duration<float> duration;

    Timer()
    {
        start = std::chrono::steady_clock::now(); //如果使用auto关键字会出现警告
    }

    ~Timer()
    {
        end = std::chrono::steady_clock::now();
        duration = end - start;

        float ms = duration.count() * 1000;
        std::cout << "Timer took " << ms << " ms" << std::endl;
    }
};
void Function() {
    Timer timer;
    for (int i = 0; i < 100; i++)
        std::cout << "Hello\n"; //相比于std::endl更快
}

int main() {
    Function();
}
```



## 类型双关 (type punning)(不理解)
类型双关，是对内存的操作。意思是把一段内存当作不同类型的内存来对待，将该类型作为指针，然后将其转换到另一个指针。（难以理解hhh...）

1. 将同一块内存的东西通过不同 type 的指针给取出来

把一个 int 型的内存，换成 double 去解释，当然**这样做很糟糕**，因为添加了四字节不属于原本自己的内存，只是作为演示。 原始方法：**（取地址，换成对应类型的指针，再解引用）**

```cpp
#include <iostream>
int main() {
    int a = 50;
    double value = *(double*)&a;
    std::cout << value << std::endl;

    std::cin.get();
}
//可以用引用，这样就可以避免拷贝成一个新的变量：（只是演示，这样做很糟糕）
#include <iostream>
int main() {
    int a = 50;
    double& value = *(double*)&a;
    std::cout << value << std::endl;

    std::cin.get();
}
```

2. 把一个结构体转换成数组进行操作

```cpp
#include <iostream>
struct Entity
{
    int x, y;
};

int main() {
    Entity e = {5, 8};
    int *position = (int *)&e;
    std::cout << position[0] << ", " << position[1] << std::endl;

    int y = *(int *)((char *)&e + 4);
    std::cout << y << std::endl;
}
```



## 预编译头文件

1. 作用：

为了解决一个项目中同一个头文件被反复编译的问题。使得写代码时不需要一遍又一遍的去`#include`那些常用的头文件，而且能**大大提高编译速度**

2. 使用限制：预编译头文件中的内容最好都是**不需要反复更新修改的东西**。

每修改一次，预编译头文件都要重新编译一次，会导致变异速度降低。但像 **C++ 标准库**，window 的 api 这种不会大改的文件可以放到预编译头文件中，可以节省编译时间

3. 缺点：

预编译头文件的使用会隐藏掉这个 cpp 文件的依赖。比如用了`#include <vector>`，就清楚的知道这个 cpp 文件中需要 vector 的依赖，而如果放到预编译头文件中，就会将该信息隐藏。

4.. 使用流程：

在 Visual Studio 中：[https://www.bilibili.com/video/BV1eu411f736?share_source=copy_web&vd_source=48739a103c73f618758b902392cb372e](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1eu411f736%3Fshare_source%3Dcopy_web%26vd_source%3D48739a103c73f618758b902392cb372e)

视频讲解更为详细。

在 g++ 中：

首先确保 main.cpp（主程序文件）、pch.cpp(包含预编译头文件的 cpp 文件)、pch.h（预编译头文件）在同一源文件目录下

注：pch.h 文件的名字是自己命名的，改成其他名称也没问题。

```cpp
g++ -std=c++11 pch.h //先编译pch头文件
  //time的作用是在控制台显示编译所需要的时间。
  time g++ -std=c++11 main.cpp  //然后编译主程序文件即可，编译速度大大提升。
```

## 基准测试

1. 编写一个计时器对代码测试性能。记住要**在 release 模式**去测试，这样才更有意义 。

2. 该部分内容基本同 "C++ 计时" 一节（对应视频 P63）

```
#include <iostream>
#include <memory>
#include <chrono>   //计时工具
#include <array>
class Timer {
public:
    Timer() {
        m_StartTimePoint =  std::chrono::high_resolution_clock::now();
    }
    ~Timer() {
        Stop();
    }
    void Stop() {
        auto endTimePoint = std::chrono::high_resolution_clock::now();
        auto start = std::chrono::time_point_cast<std::chrono::microseconds>(m_StartTimePoint).time_since_epoch().count();
        //microseconds 将数据转换为微秒
        //time_since_epoch() 测量自时间起始点到现在的时长
        auto end = std::chrono::time_point_cast<std::chrono::microseconds>(endTimePoint).time_since_epoch().count();
        auto duration = end - start;
        double ms = duration * 0.001; ////转换为毫秒数
        std::cout << duration << "us(" << ms << "ms)\n";
    }
private:
    std::chrono::time_point<std::chrono::high_resolution_clock> m_StartTimePoint;
};

int main() {
    struct Vector2 {
        float x, y;
    };

    {
        std::array<std::shared_ptr<Vector2>, 1000> sharedPtrs;
        Timer timer;
        for (int i = 0; i < sharedPtrs.size(); i++) {
            sharedPtrs[i] = std::make_shared<Vector2>();
        }
    }

    {
        std::array<std::shared_ptr<Vector2>, 1000> sharedPtrs;
        Timer timer;
        for (int i = 0; i < sharedPtrs.size(); i++) {
            sharedPtrs[i] = std::shared_ptr<Vector2>(new Vector2());
        }
    }

    {
        Timer timer;
        std::array<std::unique_ptr<Vector2>, 1000> sharedPtrs;
        for (int i = 0; i < sharedPtrs.size(); i++) {
            sharedPtrs[i] = std::make_unique<Vector2>();
        }
    }
}
```

## 76. optional 数据 (std::optional)

1.C++17 在 STL 中引入了`std::optional`，就像`std::variant`一样，`std::optional`是一个 “**和类型 (sum type)**”，也就是说，`std::optional`类型的变量要么是一个`T`类型的**变量**，要么是一个表示 “什么都没有” 的**状态**。

2. 基本用法：

首先要包含`#include <optional>`

3.**has_value()**

我们可以通过 has_value() 来判断对应的 **optional 是否处于已经设置值**的状态, 代码如下所示:

```
int main() {
std::string text = /*...*/;
std::optional<unsigned> opt = firstEvenNumberIn(text);
if (opt.has_value())  //直接if(opt)即可，代码更简洁
{
 std::cout << "The first even number is "
           << opt.value()
           << ".\n";
}
}
```

4. 访问 optional 对象中的数据

```
1. opt.value()
2. (*opt)
3. value_or() //value_or()可以允许传入一个默认值, 如果optional为std::nullopt, 
              //则直接返回传入的默认值.（如果数据确实存在于std::optional中，
              //它将返回给我们那个字符串。如果不存在，它会返回我们传入的任何值）
```

`std::optional`是 C++17 的新东西，用于检测数据是否存在 or 是否是我们期盼的形式，用于处理那些可能存在，也可能不存在的数据 or 一种我们不确定的类型 。

比如在读取文件内容的时候，往往需要判断读取是否成功，常用的方法是传入一个引用变量或者判断返回的 std::string 是否为空，C++17 引入了一个更好的方法，std::optional

老方法：传入一个引用变量或者判断返回的 std::string 是否为空

```
#include <iostream>
#include <fstream>
#include <string>
std::string ReadFile(const std::string &fileapath, bool &outSuccess) {
    std::ifstream stream(filepath);
    //如果成功读取文件
    if (stream) {
        std::string result;
        getline(stream,result);
        stream.close();
        outSuccess = true;  //读取成功，修改bool
        return result;
    }
    outSuccess = false; //反之
}

int main() {
    bool flag;
    auto data = ReadFile("data.txt", flag);
    //如果文件有效，则接着操作
    if (flag) {

    }
}
```

**新方法：std::optional**

```
// 用g++编译时需要加上‘-std=c++17’ or ‘-std=gnu++17’
// std::optional同样是C++17的新特性，可以用来处理可能存在、也可能不存在的数据
//data.txt在项目目录中存在，且其中的内容为"data!"
#include <iostream>
#include <fstream>
#include <optional>
#include <string>

std::optional<std::string> ReadFileAsString(const std::string& filepath)
{
    std::ifstream stream(filepath);
    if (stream)
    {
        std::string result;
        getline(stream, result);
        stream.close();
        return result;
    }
    return {};
    //如果文本存在的话，它会返回所有文本的字符串。如果不存在或者不能读取；则返回optional {}
}

int main() {
     std::optional<std::string> data = ReadFileAsString("data.txt");
    //auto data = ReadFileAsString("data.txt"); //可用auto关键字
    if (data)
    {   
       // std::string& str = *data;
       // std::cout << "File read successfully!" << str<< std::endl;
        std::cout << "File read successfully!" << data.value() << std::endl;      
    }
    else
    {
        std::cout << "File could not be opened!" << std::endl;
    }

    std::cin.get();
}
//输出
File read successfully!"data!"
```

如果文件无法打开，或者文件的特定部分没有被设置或读取，也许我们有一个默认值，这很常见。此时就可以使用 value_or() 函数。其作用就是：如果数据确实存在于 std::optional 中，它将返回给我们那个字符串。如果不存在，它会返回我们传入的任何值。

删除 data.txt, 此时文件不存在打不开，则被设置为默认值

```
#include <iostream>
#include <fstream>
#include <optional>
#include <string>

std::optional<std::string> ReadFileAsString(const std::string& filepath)
{
    std::ifstream stream(filepath);
    if (stream)
    {
        std::string result;
        //getline(stream, result);
        stream.close();
        return result;
    }

    return {}; //返回空
}

int main() {
    std::optional<std::string>  data = ReadFileAsString("data.txt");

    std::string value = data.value_or("Not present");
    std::cout << value << std::endl;

    if (data)
    {
        std::cout << "File read successfully!" << std::endl;
    }
    else
    {
        std::cout << "File could not be opened!" << std::endl;
    }
}
//输出
Not present
File could not be opened!
```

## 77. C++ 单一变量存放多种类型的数据 (std::variant)

1.std::variant 是 C++17 的新特性，可以让我们不用担心处理的确切数据类型 , 是一种 一种可以容纳多种类型变量的结构 。

它和`option`很像，它的作用是让我们不用担心处理确切的数据类型，只有一个变量，之后我们在考虑它的具体类型  
故我们做的就是指定一个叫`std::variant`的东西，然后列出它可能的数据类型

2. 与 union 的区别

1)union 中的成员内存共享。union 更有效率。 2)std::variant 的大小是 <> 里面的大小之和 。**variant** 更加**类型安全**，不会造成未定义行为，**所以应当去使用它, 除非做的是底层优化，非常需要性能**。

3. 简单的运用：

```
std::variant<string, int> data; //列举出可能的类型
data = "hello";
// 索引的第一种方式：std::get，但是要与上一次赋值类型相同，不然会报错
cout << std::get<string>(data) <<endl;//print hello
data = 4;
cout << std::get<int>(data) <<endl;//print 4
cout << std::get<string>(data) <<endl;//编译通过，但是runtime会报错，显示std::bad_variant_access
data = false;//能编译通过
cout << std::get<bool>(data) <<endl;//这句编译失败
```

index() 索引

```
//std::variant的index函数
data.index();// 返回一个整数，代表data当前存储的数据的类型在<>里的序号，比如返回0代表存的是string, 返回1代表存的是int
```

get_if()

```
// std::get的变种函数，get_if
auto p = std::get_if<std::string>(&data);//p是一个指针，如果data此时存的不是string类型的数据，则p为空指针，别忘了传的是地址
// 如果data存的数据是string类型的数据
if(auto p = std::get_if<string>(&data)){
    string& s = *p;
}
```

**cherno 的代码：**

```
//参考：https://zhuanlan.zhihu.com/p/352420950
#include<iostream>
#include<variant>
int main() {
    std::variant<std::string,int> data; // <>里面的类型不能重复
    data = "ydc";
    // 索引的第一种方式：std::get，但是要与上一次赋值类型相同，不然会报错
    std::cout<<std::get<std::string>(data)<<std::endl;
    // 索引的第二种方式，std::get_if，传入地址，返回为指针
    if (auto value = std::get_if<std::string>(&data))
    {
        std::string& v = *value;
    }
    data = 2;
    std::cout<<std::get<int>(data)<<std::endl;
    std::cin.get();
}
```

## 78. C++ 如何存储任意类型的数据 (std::any)

1. 也是 C++17 引入的可以存储多种类型变量的结构，其本质是一个 union，但是不像 std::variant 那样需要列出类型。使用时要包含头文件`#include <any>`

2. 对于小类型 (small type) 来说，any 将它们存储为一个严格对齐的 Union， 对于大类型，会用 void*，动态分配内存 。

3. **评价：基本无用**。 当在一个变量里储存多个数据类型，用 any 的类型安全版本即可：`variant`

```
#include <iostream>
#include <any>
// 这里的new的函数，是为了设置一个断点，通过编译器观察主函数中何处调用了new，看其堆栈。
void *operator new(size_t size) {
    return malloc(size);
}

int main() {
    std::any data;
    data = 2;
    data = "Cherno";
    data = std::string("Cherno");

    std::string& string = std::any_cast<std::string&>(data); //用any_cast指定转换的类型,如果这个时候any不是想要转换的类型，则会抛出一个类型转换的异常
    // 通过引用减少复制操作，以免影响性能
}
```

## 79. 如何让 C++ 运行得更快 (std::async)

1. 利用 std::async，封装了异步编程的操作，提高了性能。

两个问题： 1、为什么不能传引用？ 线程函数的参数按值移动或复制。如果引用参数需要传递给线程函数，它必须被包装（例如使用 std :: ref 或 std :: cref）  
2、std::async 为什么一定要返回值？ 如果没有返回值，那么在一次 for 循环之后，临时对象会被析构，而析构函数中需要等待线程结束，所以就和顺序执行一样，一个个的等下去 如果将返回值赋值给外部变量，那么生存期就在 for 循环之外，那么对象不会被析构，也就不需要等待线程结束。

具体实现原理还不明白，此处留个坑，以后学了再填。

相关参考资料：

cherno 的视频讲解：[https://www.bilibili.com/video/BV1UR4y1j7YL?share_source=copy_web&vd_source=48739a103c73f618758b902392cb372e](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1UR4y1j7YL%3Fshare_source%3Dcopy_web%26vd_source%3D48739a103c73f618758b902392cb372e)

官方文档：[https://en.cppreference.com/w/cpp/thread/async](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/cpp/thread/async)

## 80. 如何让 C++ 字符串更快 in C++

1. 内存分配建议：能分配在栈上就别分配到堆上，因为把内存分配到堆上会降低程序的速度 。

2.std::string_view 同样是 C++17 的新特性

3.gcc 的 string 默认大小是 32 个字节，字符串小于等于 15 直接保存在栈上，超过之后才会使用 new 分配

4.string 的常用优化：**SSO**(短字符串优化)、**COW**（写时复制技术优化）

5. **为何优化字符串？**

1)`std::string`和它的很多函数都喜欢分配在堆上，这实际上并不理想 。 2) 一般处理字符串时，比如使用`substr`切割字符串时，这个函数会自己处理完原字符串后**创建出**一个全新的字符串，它可以变换并有自己的内存（new, 堆上创建）。 3) 在数据传递中减少拷贝是提高性能的最常用办法。在 C 中指针是完成这一目的的标准数据结构，而在 C++ 中引入了安全性更高的引用类型。所以在 C++ 中若传递的数据仅仅可读，const string & 成了 C++ 天然的方式。但这并非完美，从实践上来看，它至少有以下几方面问题：  
**字符串字面值、字符数组、字符串指针**的**传递依然要数据拷贝** 这三类低级数据类型与 string 类型不同，传入时编译器要做**隐式转换**，即需要拷贝这些数据生成 string 临时对象。const string & 指向的实际上是这个临时对象。通常字符串字面值较小，性能损失可以忽略不计；但字符串指针和字符数组某些情况下可能会比较大（比如读取文件的内容），此时会引起频繁的内存分配和数据拷贝，影响程序性能。  
**substr O(n) 复杂度** substr 是个常用的函数，好在 std::string 提供了这个函数，美中不足的时每次都要返回一个新生成的子串，很容易引起性能热点。实际上我们本意不是要改变原字符串，为什么不在原字符串基础上返回呢？

6. **如何优化字符串**？通过 **string_view**

std::string_view 是 C++ 17 标准中新加入的类，正如其名，它**提供一个字符串的视图**，即可以通过这个类以各种方法 “观测” 字符串，但不允许修改字符串。由于它只读的特性，它并不真正持有这个字符串的拷贝，而是与相对应的字符串共享这一空间。即——**构造时不发生字符串的复制**。同时，你也**可以自由的移动这个视图**，**移动视图并不会移动原定的字符串**。  
通过调用 string_view 构造器可将字符串转换为 string_view 对象。string 可隐式转换为 string_view。  
1）string_view 是只读的轻量对象，它对所指向的字符串没有所有权。  
2）string_view 通常用于函数参数类型，可用来取代 const char* 和 const string&。string_view 代替 const string&，可以避免不必要的内存分配。  
3）string_view 的成员函数即对外接口与 string 相类似，但只包含读取字符串内容的部分。 4）string_view::substr() 的返回值类型是 string_view，不产生新的字符串，不会进行内存分配。 5）string::substr() 的返回值类型是 string，产生新的字符串，会进行内存分配。  
6）string_view 字面量的后缀是 sv。（string 字面量的后缀是 s）

```
#include <iostream>
#include <string>

//一种调试在heap上分配内存的方法，自己写一个new的方法，然后设置断点或者打出log，就可以知道每次分配了多少内存，以及分配了几次
static uint32_t s_AllocCount = 0;
void* operator new(size_t size) {
    s_AllocCount++;
    std::cout << "Allocating " << size << " bytes\n";
    return malloc(size);
}

#define STRING_view 1
#if STRING_view
void PrintName(std::string_view name) {
    std::cout << name << std::endl;
}
#else
void PrintName(const std::string& name) {
    std::cout << name << std::endl;
}
#endif

int main() {
    const std::string name = "Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs";
    // const char *cname = "Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs"; // C-like的编码风格

#if STRING_view
    std::string_view firstName(name.c_str(), 3);
    std::string_view lastName(name.c_str() + 4, 9);
#else
    std::string firstName = name.substr(0, 3); //substr切割字符串
    std::string lastName = name.substr(4, 9);
#endif

    PrintName(name);
    PrintName(firstName);
    PrintName(lastName);

    std::cout << s_AllocCount << " allocations" << std::endl;

    return 0;
}
```

输出：

```
//无#define STRING_view 1
Allocating 8 bytes
Allocating 80 bytes
Allocating 8 bytes
Allocating 8 bytes
Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgsgsgsgsgsgsgsdgsgsgnj
Yan
Chernosaf
4 allocations

//有#define STRING_view 1
Allocating 8 bytes
Allocating 64 bytes
Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs
Yan
Chernosaf
2 allocations
```

可见 使用 string_view 减少了内存在堆上的分配。

**进一步优化：使用 C 风格字符串**

```
int main() {
    //const std::string name = "Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs";
    const char *cname = "Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs"; // C-like的编码风格

#if STRING_view
    std::string_view firstName(name, 3); //注意这里要去掉 .c_str()
    std::string_view lastName(name + 4, 9);
#else
    std::string firstName = name.substr(0, 3); 
    std::string lastName = name.substr(4, 9);
#endif

    PrintName(name);
    PrintName(firstName);
    PrintName(lastName);

    std::cout << s_AllocCount << " allocations" << std::endl;

    return 0;
}
```

输出

```
//有#define STRING_view 1
Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs
Yan
Chernosaf
0 allocations
```

注意：不同编译器的结果有所不同。

## 81. C++ 的可视化基准测试

1. 利用工具： chrome://tracing （chrome 浏览器自带的一个工具，将该网址输入即可）

2. 基本原理： cpp 的计时器配合自制简易 json 配置写出类，将时间分析结果写入一个 json 文件，用 chrome://tracing 这个工具进行可视化 。

3. 多线程可视化实现：

视频：[https://www.bilibili.com/video/BV1gZ4y1R7SG?share_source=copy_web&vd_source=48739a103c73f618758b902392cb372e](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1gZ4y1R7SG%3Fshare_source%3Dcopy_web%26vd_source%3D48739a103c73f618758b902392cb372e) 代码改进链接：[https://github.com/GavinSun0921/InstrumentorTimer](https://link.zhihu.com/?target=https%3A//github.com/GavinSun0921/InstrumentorTimer)

4. 实现代码：

```
#pragma once
#include <string>
#include <chrono>
#include <algorithm>
#include <fstream>
#include <cmath>
#include <thread>
#include <iostream>

struct ProfileResult
{
    std::string Name;
    long long Start, End;
    uint32_t ThreadID; //线程ID
};

struct InstrumentationSession
{
    std::string Name;
};

class Instrumentor
{
private:
    InstrumentationSession* m_CurrentSession;
    std::ofstream m_OutputStream;
    int m_ProfileCount;
public:
    Instrumentor()
        : m_CurrentSession(nullptr), m_ProfileCount(0)
    {
    }

    void BeginSession(const std::string& name, const std::string& filepath = "results.json") {
        m_OutputStream.open(filepath);
        WriteHeader();
        m_CurrentSession = new InstrumentationSession{ name };
    }

    void EndSession() {
        WriteFooter();
        m_OutputStream.close();
        delete m_CurrentSession;
        m_CurrentSession = nullptr;
        m_ProfileCount = 0;
    }

    void WriteProfile(const ProfileResult& result) {
        if (m_ProfileCount++ > 0)
            m_OutputStream << ",";

        std::string name = result.Name;
        std::replace(name.begin(), name.end(), '"', '\'');

        m_OutputStream << "{";
        m_OutputStream << "\"cat\":\"function\",";
        m_OutputStream << "\"dur\":" << (result.End - result.Start) << ',';
        m_OutputStream << "\"name\":\"" << name << "\",";
        m_OutputStream << "\"ph\":\"X\",";
        m_OutputStream << "\"pid\":0,";
        m_OutputStream << "\"tid\":" << result.ThreadID << ","; //多线程
        m_OutputStream << "\"ts\":" << result.Start;
        m_OutputStream << "}";

        m_OutputStream.flush();
    }

    void WriteHeader() {
        m_OutputStream << "{\"otherData\": {},\"traceEvents\":[";
        m_OutputStream.flush();
    }

    void WriteFooter() {
        m_OutputStream << "]}";
        m_OutputStream.flush();
    }

    static Instrumentor& Get() {
        static Instrumentor instance;
        return instance;
    }
};

class InstrumentationTimer
{
public:
    InstrumentationTimer(const char* name)
        : m_Name(name), m_Stopped(false)
    {
        m_StartTimepoint = std::chrono::high_resolution_clock::now();
    }

    ~InstrumentationTimer()
    {
        if (!m_Stopped)
            Stop();
    }

    void Stop() {
        auto endTimepoint = std::chrono::high_resolution_clock::now();

        long long start = std::chrono::time_point_cast<std::chrono::microseconds>(m_StartTimepoint).time_since_epoch().count();
        long long end = std::chrono::time_point_cast<std::chrono::microseconds>(endTimepoint).time_since_epoch().count();

        uint32_t threadID = std::hash<std::thread::id>{}(std::this_thread::get_id());//在timer中的stop函数看看timer实际上是在哪个线程上运行的
        Instrumentor::Get().WriteProfile({ m_Name, start, end, threadID }); 

        m_Stopped = true;
    }
private:
    const char* m_Name;
    std::chrono::time_point<std::chrono::high_resolution_clock> m_StartTimepoint;
    bool m_Stopped;
};

//测试代码
//计时器不是一直用的，应该有一个简单的方法来关闭这些，因为这会增加一些开销。通过定义宏可以做到。
//定义宏PROFILING，如果这个设置为1，那么PROFILING是启用的，这意味着我们会有PROFILE_SCOPE引入一个InstrumentationTimer，并做所有这些事情
#define PROFILING 1 //如果被禁用，比如设为0，则运行空代码，这意味着PROFILE_SCOPE没有任何代码.这可以有效地从PROFILING设为0的构建中，剥离计时器
#if PROFILING
//定义一个宏叫做PROFILE_SCOPE，它会把name作为参数，这会包装我们的InstrumentationTimer调用
#define PROFILE_SCOPE(name) InstrumentationTimer timer##__LINE__(name) //##__LINE__作用市拼接行号，让每个实例的实例名不一样，加上之后，就不是time了，而是time加行号。在这个例子中确实可以不用，但是如果你在一个作用域调用两次，就会造成实例名重定义错误。
#define PROFILE_FUNCTION() PROFILE_SCOPE(__FUNCSIG__) //定义一个PROFILE_FUNCTION()宏，会调用PROFILE_SCOPE宏，但对于name，它会接受函数的名字，我们可以用这个编译宏__FUNCSIG__来做(__FUNCSIG__可以避免函数重载带来的问题)
#else
#define PROFILE_SCORE(name)
#endif

//如果你的代码中有你想要分析的区域，特别的，不是在函数中你可以将PROFILE_SCOPE放到任何作用域当中
namespace Benmarks  //可以把PROFILE_FUNCTION()放到程序的每一个函数中。比如这个Benmarks命名空间。且因为用了宏__FUNCSIG__，可以得到空间内的所有信息
{
    void Function(int value) {
        // PROFILE_SCOPE("Function1"); //同InstrumentationTimer timer("Function1");
        PROFILE_FUNCTION(); //同 PROFILE_SCOPE("Function1");
        for (int i = 0; i < 1000; i++)
            std::cout << "hello world" << (i + value) << std::endl;
    }
    void Function() {
        //PROFILE_SCOPE("Function2");
        PROFILE_FUNCTION();
        for (int i = 0; i < 1000; i++)
            std::cout << "hello world" << i << std::endl;
    }

    void RunBenchmarks() {
        //PROFILE_SCOPE("RunBenchmarks");
        PROFILE_FUNCTION();
        std::cout << "Runint Benchmarks....\n";
        Function(2);
        Function();
    }
}
int main() {
    Instrumentor::Get().BeginSession("Profile");    
    Benmarks::RunBenchmarks();
    Instrumentor::Get().EndSession();
    std::cin.get();
}
```

## 82. C++ 的单例模式

1.Singleton 只允许被实例化一次，用于组织一系列全局的函数或者变量，与 namespace 很像。例子：随机数产生的类、渲染器类。

2.**C++ 中的单例只是一种组织一堆全局变量和静态函数的方法**

3. 什么时候用单例模式：

当我们想要**拥有应用于某种全局数据集的功能**，且我们**只是想要重复使用**时，单例是非常有用的

有些单例的例子，比如一个随机数生成器类 我们只希望能够查询它，得到一个随机数，而不需要实例化它去遍历所有这些东西我们只想实例化它一次（单例），这样它就会生成随机数生成器的种子，建立起它所需要的任何辅助的东西了 另一个例子就是渲染器，渲染器通常是一个非常全局的东西 我们通常不会有一个渲染器的多个实例，我们有一个渲染器，我们向它提交所有这些渲染命令，然后它就会为我们渲染

4. 实现单例的基本方法：

1) **将构造函数设为私有**，因为单例类不能有第二个实例

2) 提供一个静态访问该类的方法

设一个**私有的静态的实例**，**并且在类外将其定义！** 然后用一个静态函数返回其引用 or 指针，便可正常使用了

3) 为了安全，**标记拷贝构造函数为 delete**（删除拷贝构造函数）

```
#include <iostream>

class SingleTon {
    SingleTon(const SingleTon&) = delete; //删除拷贝构造函数
public:
    //static在类里表示将该函数标记为静态函数
    static SingleTon& get() {
        return m_temp;
    }

    void Function() {}  //比如说这里有一些方法可供使用

private:
    SingleTon() {}; //将构造函数标记为私有
    static SingleTon m_temp;    //在私有成员里创建一个静态实例
};

SingleTon SingleTon::m_temp;    //像定义任何静态成员一样定义它

int main() {
    //SingleTon temp2 = SingleTon::get();       //会报错，因为无法复制了
    SingleTon& temp2 = SingleTon::get();        //会报错，因为无法复制了
   // SingleTon::get().Function();  //这般使用便可
    temp2.Function();
}
```

一个简单的随机数类的例子

1) 将构造函数设为私有，防止从外部被实例化

2) 设置 get() 函数来返回静态引用的实例

直接在 get() 中设置静态实例就可以了，在调用 get() 的时候就直接设置静态实例

3) 标记复制构造函数为 delete

```
#include<iostream>
class Random
{
public:
    Random(const Random&) = delete; // 删除拷贝复制函数
    static Random& Get() // 通过Get函数来获取唯一的一个实例 {
        static Random instance; // 在此处实例化一次
        return instance;
    }
    static float Float(){ return Get().IFloat();} // 调用内部函数,可用类名调用
private:
    float IFloat() { return m_RandomGenerator; } // 将函数的实现放进private
    Random(){} // 不能让别人实例化，所以要把构造函数放进private
    float m_RandomGenerator = 0.5f;
};
// 与namespace很像
namespace RandomClass {
    static float s_RandomGenerator = 0.5f;
    static float Float(){return s_RandomGenerator;}
}
int main() {
    float randomNum = Random::Float();
    std::cout<<randomNum<<std::endl;
    std::cin.get();
}
```

## 83. C++ 的小字符串优化

VS 开发工具在 release 模式下面 （debug 模式都会在堆上分配） ，使用 size 小于 16 的 string，不会分配内存，而大于等于 16 的 string，则会分配 32bytes 内存以及更多，所以 16 个字符是一个分界线 （注：不同编译器可能会有所不同）

```
#include <iostream>

void* operator new(size_t size) {
    std::cout << "Allocated: " << size << " bytes\n";
    return malloc(size);
}

int main() {
    // debug模式都会在堆上分配
    std::string longName = "cap cap cap cap "; // 刚好16个字符，会在堆上分配32个bytes内存
    std::string testName = "cap cap cap cap"; // 15个字符，栈上分配
    std::string shortName = "cap";

    std::cin.get();
}
//debug模式输出
Allocated: 16 bytes
Allocated: 32 bytes
Allocated: 16 bytes
Allocated: 16 bytes

//release模式输出：
Allocated: 32 bytes
```

## 84. 跟踪内存分配的简单方法

重写 new 和 delete 操作符函数，并在里面打印分配和释放了多少内存，也可在重载的这两个函数里面设置断点，通过查看调用栈即可知道什么地方分配或者释放了内存

我们知道一个 class 的 new 是分为三步：operator new（其内部调用 malloc）返回 void*、static_cast 转换为这个对象指针、构造函数。而 delete 则分为两步：构造函数、operator delete。 new 和 delete 都是表达式，是不能重载的；而把他们行为往下分解则是有 operator new 和 operator delete，是有区别的。 直接用的表达式的行为是不能变的，不能重载的，即 new 分解成上图的三步与 delete 分解成上图的两步是不能重载的。这里内部的 operator new 和 operator delete 底层其实是调用的 malloc，这些内部的几步则是可以重载的。 原文链接：[https://blog.csdn.net/weixin_47652005/article/details/121026982](https://link.zhihu.com/?target=https%3A//blog.csdn.net/weixin_47652005/article/details/121026982)

```
// 重写new、free操作符之后就能方便地跟踪内存分配了(加断点)

#include <iostream>
#include <memory>

struct AllocationMetrics
{
    uint32_t TotalAllocated = 0; //总分配内存
    uint32_t TotalFreed = 0; //总释放内存

    uint32_t CurrentUsage() { return TotalAllocated - TotalFreed; } //写一个小函数来输出 当前用了多少内存
};

static AllocationMetrics s_AllocationMetrics; //创建一个全局静态实例

void *operator new(size_t size) {
    s_AllocationMetrics.TotalAllocated += size; //在每一个new里计算总共分配了多少内存
    // std::cout << "Allocate " << size << " bytes.\n";
    return malloc(size);
}

void operator delete(void *memory, size_t size) {
    s_AllocationMetrics.TotalFreed += size;
    // std::cout << "Free " << size << " bytes.\n";
    free(memory);
}

struct Object
{
    int x, y, z;
};
//可以用一个函数输出我们的内存使用情况
static void PrintMemoryUsage() {
    std::cout << "Memory Usage:" << s_AllocationMetrics.CurrentUsage() << " bytes\n";
}

int main() {
    PrintMemoryUsage();
    {
        std::unique_ptr<Object> obj = std::make_unique<Object>();
        PrintMemoryUsage();
    }

    PrintMemoryUsage();
    Object *obj = new Object;
    PrintMemoryUsage();
    delete obj;
    PrintMemoryUsage();
    std::string string = "Cherno";
    PrintMemoryUsage();

    return 0;
}
```


## 86. C++ 持续集成（CI）

1、 CI(Continuous integration，中文意思是持续集成) 是一种软件开发时间。持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。 2、主要讲解如何在 linode 租一个服务器，来运行 Jenkins

## 87. C++ 静态分析

*   主要讲了一个工具 PVS-studio 的用法，可以 static analyze 代码
*   开源的推荐 clang-tidy

[如何在 VS Code 中运行 clang-tidy?： https://zhuanlan.zhihu.com/p/446084601](https://zhuanlan.zhihu.com/p/446084601)

## 88. C++ 的参数计算顺序

1、讲了一个 undefined behavior 的例子：

```
//参考：https://zhuanlan.zhihu.com/p/352420950
#include<iostream>
void PrintSum(int a, int b) {
    std::cout<<a<<"+"<<b<<"="<<a+b<<std::endl;
}
int main() {
    int value = 0;
    PrintSum(value++,++value); //行为未定义！
    std::cin.get();
}
```

类似这样在传参时使用 ++，这种行为是不确定的，在不同编译器不同语言版本和配置下，其行为不一致，所以严禁这样使用


## numeric_limits＜T＞::max()
基本格式：**std::numeric_limits< T >::max()**——其中max()可替换为min()及lowest()；T表示数据类型；
```c++
std::numeric_limits<T>::max()——返回数据类型T的最大值；

std::numeric_limits<T>::min()——返回数据类型T的"最小正数"；

std::numeric_limits<T>::lowest()——返回数据类型T的最小值（为负数或0）；
```

用法举例：
```c++
#include<iostream>
#include<limits>
using namespace std;
int main(){
        cout << "max(short):  " << numeric_limits<short>::max() << endl;//输出short数据类型的最大值
 
        cout << "max(int):  " << numeric_limits<int>::max() << endl;
 
        cout << "max(long):  " << numeric_limits<long>::max() << endl;
 
        cout << "max(float):  " << numeric_limits<float>::max() << endl;
 
        cout << "max(double):  " << numeric_limits<double>::max() << endl;
 
        cout << "max(long double):  " << numeric_limits<long double>::max() << endl;
        return 0;
}
```
![[Pasted image 20230105134539.png]]