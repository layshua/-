这一节我们讲的内容会比较多，不过难度不大。主要内容如下：

![](https://pic1.zhimg.com/v2-8652c518d7d6e8e73f2f84719ee69ec0_r.jpg)

## 创建自己的 C++ 类

### 使用 UE 创建 C++ 类

我们在 UE4 里面创建一个 C++ 类，有如下三个步骤：

![](https://pic3.zhimg.com/v2-280c79c6a646f680105de3684757ddae_r.jpg)

点击创建后，虚幻引擎会打开 Visual Studio。产生 *.h 和 *.cpp 文件，然后会自动编译，并且加载到引擎中。

![](https://pic1.zhimg.com/v2-0a5b1b63e1f6694b3b610329ac569a28_r.jpg)

### 手工创建 C++ 类

如果是需要手工创建 C++ 类的话，推荐有 public 目录，以及 private 目录，其中 *.h 推荐放到 public 目录，而 *.cpp 推荐放到 private 目录：

![](https://pic1.zhimg.com/v2-9456a6465c0331d701132af75b238830_r.jpg)

如果我们的类是继承自 UObject。类名上方需要加入 UClass() 宏，类体的第一行需要添加 GENERAED_BODY() 宏，主要是为了实现类的反射用，另外头文件记得包含一个 *.generated.h 文件：

![](https://pic1.zhimg.com/v2-b0484afef0bd6be226f737450b60de88_r.jpg)

### 虚幻引擎类名规则

我们发现，虽然我们在创建 C++ 类的时候，给的类名是 MyObject，但是它的类名实际上是 UMyObject。那么这前面的 U 表示什么意思呢？这是按照虚幻引擎的命名规则，添加的命名前缀，常用的命名前缀如下：

![](https://pic1.zhimg.com/v2-8082ff1b3b850f07912ec58f3c04cf48_r.jpg)

## 对象

### 类对象的产生

在标准 C++ 中，一个类产生一个对象，被称为 “实例化”。标准 C++ 一般是通过 new 的关键字来实例化一个对象。

![](https://pic3.zhimg.com/v2-543f1289b7e5b2a5e87adcd58b9f238a_r.jpg)

在虚幻引擎中，实例化一个类的对象需要判断类的父类，使用不同的实例化方法：

![](https://pic3.zhimg.com/v2-a83f529323a23499669fc1cc2965ceca_r.jpg)

### 类对象的获取

我们可以通过某种方式获取到这个对象的指针或引用，然后对它进行操作。

比如我们可以通过 Actor 迭代器：TActorItertor，来获取当前 World 的所有 Actor，如下代码：

```
for (TActorItertor<AActor> Iter(GetWorld()); Iter; ++Iter)
{
   // 调用这个Actor的某个你需要成员函数等
   Iter->YourFunction();
}
```

### 类对象的销毁

一个类的对象需要销毁，一有如下几种方式：

![](https://pic1.zhimg.com/v2-f630a7cb6db267d2db5b8f13a1fcd018_r.jpg)

**纯 C++ 对象有三种方式：**

一，是局部变量自动销毁，如下代码：

```
void func1() {
   FMyClass myObj = FMyClass();
   // do something 
   // 函数结束时，局部变量自动销毁
}
```

二，是使用 delete 来手动删除：

```
void func1() {
  FMyClass *myObj = new FMyClass();
  // do something
  delete myObj;
}
```

三，是使用 TSharedPtr 的智能指针来自动管理。底层使用了引用计数的原理。

**UObject 对象**

对于 UObject 的对象，一般其他对象用 UPROPERTY 的成员引用了它，只要不引用了，就会自动垃圾回收销毁了，也可以手动 AddToRoot，让它不被回收。后续需要删除时再移除出 Root 即可。

**AActor 对象**

AActor 对象比较简单，需要销毁时调用 Destroy() 函数请求销毁即可，后续引擎会自动调用垃圾回收真正删除它。

## 从 C++ 到蓝图

我们可以通过 UPROPERTY 宏把成员变量注册到蓝图，通过 UFUNCTION 宏把函数注册到蓝图。比如我们创建了一个 MyActor 类，并注册了 MyPlayerName 的成员变量和 HelloUE5() 的函数，如下图：

![](https://pic2.zhimg.com/v2-46da7b26cde3dd0f956deba636b68fd1_r.jpg)

### UPROPERTY 宏

我们使用 UPROPERTY 宏来注册成员变量到蓝图，比如 UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "MyActor") 其中 EditAnywhere 表示可以在蓝图任意地方编辑，BlueprintReadWrite 表示蓝图可以读写这个成员变量，Category = "MyActor" 表示它放在属性编辑器的 “MyActor” 子栏。

### UFUNCTION 宏

我们使用 UFUNCTION 宏来注册成员函数到蓝图，比如 UFUNCTION(BlueprintCallable, Category = "MyActor")，其中 BlueprintCallable 表示蓝图可以调用，Category = "MyActor" 表示函数的子分类为 "MyActor" 可以在蓝图选择调用函数时快速定位到它。

## 引擎系统相关类简介

引擎中常用的一些工具实用类如下：

![](https://pic3.zhimg.com/v2-a59fa1225041d80930c5d62aa03f4d12_r.jpg)

### 正则表达式

需要包含头文件 #include "Regex.h"，通过 FRegexPattern 来创建一个待查找或匹配的模式，然后通过 FRegexMatcher 来查找与匹配字符串中符合模式的部分，如下代码：

```
#include "Regex.h"
FString TextStr("ABCDEFGHIJKLMN");
FRegexPattern TestPattern(TEXT("C.+H"));
FRegexMatcher TestMacher(TestPattern, TestStr);

if (TestMacher.FindNext())
{
  UE_LOG(MyLog, Warning, TEXT("找到匹配内容 %d ~ %d"), 
                TestMacher.GetMatchBeginning(),
                TestMacher.GetMatchEnding());
}
```

### FPaths 路径管理器

在 Core 模块中，虚幻引擎提供了一个用于路径相关处理的类：FPaths。主要提供了 3 类 “工具” 性质 API：

1，**具体路径类**： FPaths::GameDir() 获取游戏根目录

2，**工具类**：FPaths::FileExists() 判断一个文件是否存在。

3，**路径转换类**：FPaths::ConvertRelativePathToFull() 将相对路径转换为绝对路径。

### 配置文件解析

这里涉及到 XML，JSON 与 INI 等三种文件：

**XML**：可以使用 FastXML 与 FXmlFile 进行解析与处理。

**Json**：可以使用 FJsonSerializer 与 FJsonValue 等来进行解析与处理。

**ini**：使用 GConfig 类进行读写与解析。

### 文件读写与访问

使用 FPlatformFileManager::Get()->GetPlatformFile() 进行文件的读写相关。常用函数如下：

CopyDirctorTree 拷贝目录树

CopyFile 拷贝当前文件

CreateDirectory 创建目录

DeleteDirectory 删除目录

DeleteFile 删除指定文件

FileExists 检查文件是否存在

OpenRead 打开一个文件进行读取

OpenWrite 打开一个文件进行写入

等等。

### LOG

Log 作为开发中常用到的功能，可以在任何需要情况下记录程序运行情况。

**查看 Log**

在 Game 模式下可以加入 - Log 命令行参数来查看 Log。在编辑器模式，直接打开 Log 窗口就行（Window->DeveloperTools->OutputLog）。

**写 Log**

可以通过 UE_LOG(LogMy, Warning, TEXT("Hello world")); 来输出 log，第一个参数为 Log 的分类，需要预设。第二个参数为类型，有 Log，Warning，Error 三种类型，其中 Log 为灰色，Warning 为黄色，Error 为红色。具体的输出为 TEXT，内容可以自己构造。

**自定义 Log 的 Category**

我们自己定义 Category 的宏，如下代码：

```
DEFINE_LOG_CATEGORY_STATIC(LogMy, Warning, All);
```

### 字符串

在虚幻引擎中字符串有三种类型：FName，FText，FString。

**FName**：无法修改，大小写不敏感，其实 FName 内部只是一个整数索引，实际的字符串内容是在一个公用的字符串表中保存，非常省内存。

**FText**：无法修改，用来显示，支持本地化和多国语言。

**FString**：允许修改，区分大小写。最为灵活，不过比其他两种字符串，略为耗一些内存。

引擎相关类的总体内容如下：

![](https://pic4.zhimg.com/v2-62cb9d9aa4bc7d42360af891066eee23_r.jpg)

至此，本节的内容就讲的差不多了！我们学会了 创建自己的类，对象，C++ 与蓝图，以及一些引擎相关实用类等！下节我们会分享虚幻引擎的模块机制相关，敬请期待！！