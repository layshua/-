
> [!question] 问题
> 模块机制很厉害，但是模块是用什么样的方式配置、编译、启动和运行呢？大牛们常常提到的“UBT”和“UHT”又是什么呢？

# UBT 虚幻引擎构建工具

UBT 全称为 **UnrealBuildTool** ，虚幻引擎的构建工具。它主要负责分析 build.cs 和 Target.cs 等配置文件，并把虚幻的 .h 和 .cpp 文件编译并链接为二进制可执行文件 exe。

> [!NOTE]
> 它编译后生成的 exe 文件位于 Engine\Binaries\DotNET\UnrealBuildTool 目录，我们可以进入控制台（cmd）在这个目录，输入 UnrealBuildTool. exe 启动这个工具：
> 
> ![[5a8d1d6bae0871194740f988ca56f66c_MD5.jpg]]
> 

![[60d60ced445bf2553f492337a2351578_MD5.jpg]]

大致而言，UBT 的工作分为三个阶段：
- **收集阶段**：收集信息。UBT 收集环境变量、虚幻引擎源代码目
录、虚幻引擎目录、工程目录等一系列的信息。
- **参数解析阶段**：UBT 解析传入的命令行参数，确定自己需要生成的目标类型。
- **实际生成阶段**：UBT 根据环境和参数，开始生成 makefil，确定 C++的各种目录的位置。最终开始构建整个项目。此时编译工作交给标准 C++编译器。

同时UBT也负责监视是否需要热加载。并且调用UHT收集各个模块的信息。

请注意，UBT 被设计为一个跨平台的构建工具，因此针对不同的平台有相对应的类来进行处理。UBT 生成的 makefil 会被对应编译器平台的 ProjectFileGenerater 用于生成解决方案，而不是一开始就针对某个平台的解决方案来确定如何生成。
## 完整编译流程

1. UBT 收集目录中的 `*.cs` 文件，解析各种编译配置；
2. 然后 UBT 调用 UHT 分析需要分析的 `*.h` 文件，根据文件是否有 `UCLASS ()`，`UFUNCTION()` 等宏，来反射信息，并生成 ` *.generated.h` 和 `*. gen.cpp` 文件；
3. 最后 UBT 调用 MSBuild，将相关 `*.h/ *.cpp` 代码，以及生成的 `*.generated. h` 和 `*.gen.cpp` 一起编译。

![[30fc0c7e1f64141c9591546259720ca2_MD5.jpg]]

## UBT 命令行模式

**一，生成项目解决方案**

我们知道当我们刚创建一个名为 MyGame 游戏项目时，会右键点击 MyGame. uproject，然后点击 “Generate Visual Studio project file” 以便生成 MyGame. sln 的解决方案，方便用 Visual Studio 打开它来编译。

![[74500a8f59359cb267c68d829b8862ab_MD5.jpg]]

其实，这个操作也能直接调用 UBT 的命令行来实现，我们先在命令行模式，cd 到 Engine\Binaries\DotNET\UnrealBuildTool 目录，然后输入：

UnrealBuildTool. exe -projectfiles -project=I:/ue5/Projects/MyGame/MyGame. uproject -game -rocket -progress -log=I:/ue5/log1. txt  
也可以生成 MyGame. sln 文件。

![[742532f0b9380794279df474b7379c49_MD5.jpg]]

同样，我们也可以在 Visual Studio 中调试这个 UnrealBuildTool 工具的 C# 源代码。我们先右键点击这个项目，然后点击 “属性”，在属性那里选择“调试”，然后在“命令行参数” 那里输入：-projectfiles -project=I:/ue5/Projects/MyGame/MyGame. uproject -game -rocket -progress -log=I:/ue5/log1. txt

如下图：  

![[69831e049fd22104945a0a1be7902edd_MD5.jpg]]

然后选择这个项目，并设置为 “启动项目”，然后按 F5 即可断点调试 UnrealBuildTool 的 C# 源代码：

![[a12b24de496258bea8fc8cffdde236a6_MD5.png]]

**二，编译项目**

我们也可以用 UBT 的命令行来编译一个 UE5 的项目，比如我们可以 cd 到 Engine\Binaries\DotNET\UnrealBuildTool 目录，然后输入：

UnrealBuildTool. exe MyGameEditor Win64 Development -Project=I:/ue5/Projects/MyGame/MyGame. uproject

它就会编译 MyGame 项目在 Win64 平台的 Development Editor 配置的目标 exe 出来，如下：

![[a4d378601c6755849bb037444430b5fb_MD5.jpg]]

我们看到第一个参数为 MyGameEditor，这个是什么意思呢？它表示生成 MyGame 项目的 Editor 目标，即带 UE5 编辑器的目标，方便内部开发与调试。如果这个输入 MyGame 的话，它是编译发布给玩家的 game. exe，不带 UE5 编辑器，资源也必须提前 cook 好，不然运行不了程序。其中，这两个命令是跟解决方案的 *. Target. cs 文件一一对应的，如下图：

![[0a5d8a392ce82482fc017cce32733e9e_MD5.jpg]]

到这里，我们发现其实我们只要安装过 Visual Studio 的 C++ 编译环境，其实脱离 Visual Studio 编辑器也可以在命令行编译 UE5 引擎与游戏了。这也是为什么 Rider 等其他编辑器也能编辑与编译 UE5 引擎代码的原因。甚至我们还可以用 vim 修改引擎代码，然后用命令行编译和运行引擎。

那么 UBT 我们先介绍到这里，如果读者对 UBT 本身的代码感兴趣的话，可以在 Visual Studio 把 UnrealBuildTool 项目设置为 “启动项目”，并且右击项目，点击“属性”，在属性那里选择“调试”，然后在“命令行参数” 输入类似：

MyGameEditor Win64 Development -Project=I:/ue5/Projects/MyGame/MyGame. uproject

就能在 Visual Studio 中断点调试这个 UBT 工具本身的 C# 代码与原理了。笔者暂时对 UBT 的底层原理这块就不求甚解了。另外书中还详细提到了 WinMain 函数代码是怎么编译进 exe 的，这块有兴趣的同学也请自己参照下书本上的内容。

# UHT 虚幻引擎头文件工具

**UnrealHeaderTool**，虚幻引擎的头文件工具
 
主要负责解析各个头文件，并分析其中的有 `UCLASS()`，`USTRUCT()`等前缀的类，以及` UFUNCTION()`等前缀的函数，并且把这些类与函数等信息通过生成 (`*.generate.h`) 等代码反射给蓝图使用，如下图：

![[1924824db73600cd0c5646c61ffc4777_MD5.jpg]]


## UHT：一个引擎独立运行程序

> [!NOTE] 作者定义的引擎独立程序
> 引擎独立应用程序是指，这种应用程序依赖引擎。比如依赖 UBT 以
>配置编译环境，从而能跨平台编译，依赖引擎的某些模块。
>
但是这样的应用程序又不是一个引擎，也不是一个游戏。它最终输出为一个.exe文件。但是又不需要引擎完全启动，甚至不需要Renderer模块，以至于有可能只是一个命令行工具——比如UHT。

首先，UHT 是一个引擎独立程序，它是 C++ 写的一个 exe 程序，依赖部分引擎代码：
通过虚幻引擎源代码，你会发现 UHT 拥有自己的.target.cs 和.build.cs文件。

在其.target.cs文件中，它把自己设置为exe的输出模块。
在.build.cs文件中，它指出了自己的依赖模块：
![[4bffd93405637610b09bdb97b9108575_MD5.jpg]]

![[b62435f0a48ec59af3fc237f671f3084_MD5.jpg]]

然后又包含了 Launch 模块的 public/private 文件夹，以便让自己能够调用GEngine->PreInit函数。
最终，UHT会被编译成一个.exe文件，通过命令行参数调用。很奇妙不是吗？一个能够用来编译引擎的程序，居然依赖引擎本身。
## UHT 大致工作流程
UHT 的 Main 函数在 UnrealHeadToolMain.cpp 文件中。这个文件提供了 Main 函数，并且也通过 `IMPLEMENT_APPLICATION` 宏声明了这是个独立应用程序。

具体执行的内容如下：
1. 调用`GEngineLoop->PreInit`函数，初始化Log、文件系统等基础系统。从而允许借助UE_LOG宏输出log信息。
2. 调用`UnrealHeaderTool_Main`函数，执行真正的工作内容。
3. 调用`FEngineLoop::AppExit`退出。

## UHT 做了什么
首先，UBT 会通过命令行参数告诉 UHT，游戏模块对应的定义文件在哪。这个文件是一个. manifest 文件（由 UBT 生成），内容是 Json 字符串。这个字符串包含了所有 Module 的编译相关的信息。包括各种路径，以及预编译头文件位置等。

然后 UHT 开始了自己的三遍编译：Public Classes Headers、PublicHeaders、Private Headers。第一遍其实是为了兼容虚幻引擎3时代的代码。虚幻引擎3时代的 Classes 文件夹会向 Unreal Script 暴露这些类。接下来的两遍则是解析头文件定义，并生成 C++代码以完成需要的功能。生成的代码会和现有代码一起联合编译以产生结果。

> [!question] 问题
> UHT 生成的是代码么？代码如何完成“类有哪些成员变量、成员函
>数”这样的信息的注册呢？

经过UHT的三遍编译，最终生成的是`.generated.cpp`和`.generated.h`这两种文件。
## 反射

书中的给的定义就是，我们需要在运行时获取一个类，并且能获取这些类有哪些成员变量，成员函数；还能获取成员变量的名字等。C++ 本身并没有提供这样一套机制。所以需要自己实现。

### 新建测试项目

我们先从 Hello World 程序开始自己实现一套最简单的反射机制。

首先我们新建一个空白的控制台项目：

![[452983b8ad94da9536c6c32c28082ddf_MD5.jpg]]

### 初始测试代码

把项目名命名为 reflect1，然后，我们把初始代码修改如下：

```c++
#include <iostream>

using namespace std;

class Animal
{
public:
	virtual void call() { cout << "I'm Animal" << endl; }
};

class Duck: public Animal
{
public:
	virtual void call() { cout << "Quack!" << endl; }
};

class Cat: public Animal
{
public:
	virtual void call() { cout << "Meow!" << endl; }
};

int main() {
	Duck duck;
	duck.call();
	return 0;
}
```

如上代码，我们有一个动物基类，有鸭子和猫两个子类，每个子类都有一个 call 函数，表示动物叫。我们创建了一只鸭子，然后让它叫了一声：

Quack!

好的。由于没有反射，目前这个程序非常不灵活，需要用代码指定好自己想创建对象的类 Duck，不能根据用户需求来灵活创建不同的类的对象。比如用户这次运行程序想要创建一个鸭子，下次运行程序又想要创建一只猫了。

### 对象创建工厂

其实我们想要程序更为灵活一些，我们先用一种对象创建工厂的方法，我们在所有动物类定义之后，增加了一个 ClassFactory 的类，并且定义了这个类的一个全局静态实例变量：

```c++
class ClassFactory
{
public:
	Animal* CreateObj(string ClassName) {
		if (ClassName == "Duck")
		{
			return new Duck();
		}
		else if (ClassName == "Cat")
		{
			return new Cat();
		}
		else
		{
			return new Animal();
		}
	}
};

static ClassFactory g_classFactory;
```

然后，我们通过 cin 让用户输入自己想创建的类的名字，并且创建这个类的对象，让它叫一声：

```c++
int main() {
	cout << "please input the animal name: " << endl;
	string name;
	cin >> name;

	Animal *anim = g_classFactory.CreateObj(name);

	anim->call();

	delete anim;

	return 0;
}
```

我们编译并运行程序，发现会提示我们输入类的名字，我们输入 Cat，会得到喵喵：

![[6e92d001cbb7e460dd550105c5bd0d3e_MD5.jpg]]

太棒了，这下比之前更加灵活了，不过 CreateObj 不太灵活，每次创建一个新的类，都要修改这个函数来增加一个判断分支，随着类的越来越多，这个 if / else if / else 会越来越长，代码会变得很丑陋，而且我们也没有一个地方可以获取总共有哪些类。

### 灵活管理类对象创建

我们先修改一下 ClassFactory 的类代码，用一个 map 来管理所有类对象的创建，首先我们需要把 ClassFactory 的类定义移到最前，然后修改如下：

```c++
// 前向声明基类
class Animal;

// 创建类对象的函数指针类型
typedef Animal* (*CreateObjFunc)();

// 类工厂
class ClassFactory
{
public:
	// 根据类名来创建类的对象
	Animal* CreateObj(const string &ClassName) {
		if (mNameToCreateFunc.count(ClassName) > 0)
		{
			return mNameToCreateFunc[ClassName]();
		}
		
		return nullptr;
	}

	// 注册类名与创建对象函数
	void RegistClass(const string& ClassName, CreateObjFunc func) {
		mNameToCreateFunc[ClassName] = func;
	}

	// 不同类名，映射不同的创建对象方法
	map<string, CreateObjFunc> mNameToCreateFunc;
};

// 定义全局静态变量
static ClassFactory g_classFactory;
```

然后我们需要增加一个注册动作类，主要需要给每个不同的类，提供注册动作相关：

```c++
// 注册动作类，需要利用它的构造函数来注册指定类到 类工厂 （ClassFactory）
class RegistAction
{
public:
	RegistAction(const string& ClassName, CreateObjFunc func)
	{
		g_classFactory.RegistClass(ClassName, func);
	}
};
```

然后，我们需要在每个类定义后，增加两行代码来注册创建对象的函数：

![[e437df26c1a6f8ebe53a5185c9f5e11c_MD5.jpg]]

如上，我们需要给每个类增加一个创建对象函数，并利用 RegistAction 静态变量把类信息注册到类工厂上。当然这两行代码我们也可以用一个宏来简化一下：

```c++
#define REGIST(className) \
	Animal *Create##className() { return new className(); } \
	RegistAction g_RegAction##className(#className, Create##className)
```

### 测试工程完整代码

最后我们整个测试程序的代码如下：

```c++
#include <iostream>
#include <string>
#include <map>

using namespace std;

// 前向声明基类
class Animal;

// 创建类对象的函数指针类型
typedef Animal* (*CreateObjFunc)();

// 类工厂
class ClassFactory
{
public:
	// 根据类名来创建类的对象
	Animal* CreateObj(const string &ClassName) {
		if (mNameToCreateFunc.count(ClassName) > 0)
		{
			return mNameToCreateFunc[ClassName]();
		}
		
		return nullptr;
	}

	// 注册类名与创建对象函数
	void RegistClass(const string& ClassName, CreateObjFunc func) {
		mNameToCreateFunc[ClassName] = func;
	}

	// 不同类名，映射不同的创建对象方法
	map<string, CreateObjFunc> mNameToCreateFunc;
};

static ClassFactory g_classFactory;

// 注册动作类，需要利用它的构造函数来注册指定类到 类工厂 （ClassFactory）
class RegistAction
{
public:
	RegistAction(const string& ClassName, CreateObjFunc func)
	{
		g_classFactory.RegistClass(ClassName, func);
	}
};

#define REGIST(className) \
	Animal *Create##className() { return new className(); } \
	RegistAction g_RegAction##className(#className, Create##className)

class Animal
{
public:
	virtual void call() { cout << "I'm Animal" << endl; }
};
REGIST(Animal);

class Duck: public Animal
{
public:
	virtual void call() { cout << "Quack!" << endl; }
};
REGIST(Duck);

class Cat: public Animal
{
public:
	virtual void call() { cout << "Meow!" << endl; }
};
REGIST(Cat);

int main() {
	cout << "please input the animal name: " << endl;
	string name;
	cin >> name;

	Animal *anim = g_classFactory.CreateObj(name);

	if (anim)
	{
		anim->call();
		delete anim;
	}
	
	return 0;
}
```

如上代码，我们的反射测试程序差不多就讲到这里了，这里我们利用 C++ 的静态变量，把所有类名和类对象创建函数都在 main 函数之前注册到一个类工厂中，后续可以根据用户的需求灵活创建对象了，并且也可以利用类工厂的 mNameToCreateFunc 来获得当前注册的所有类信息等。如下图所示：

![[91c7de825ae19ec027fc3149305677ea_MD5.jpg]]

关于类的成员函数，以及成员变量的反射，这里就暂时不演示，其实原理和类信息的反射是差不多，就是利用 C++ 的静态变量的构造函数，先把信息都注册到一个地方，比如使用一个字典结构把变量名与变量实际指针地址对应就行。

## 初探 UE5 的反射

首先，我们有一个 MyGame 的项目，我们定义了一个自己的 AMyActor 类，代码如下：

```c++
UCLASS()
class MYGAME_API AMyActor : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	AMyActor();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "MyActor")
	FString MyPlayerName;

	UFUNCTION(BlueprintCallable, Category = "MyActor")
	void HelloUE5();

};
```

如上，我们这个 AMyActor 类增加了一个要反射的（UPROPERTY）字符串成员变量：MyPlayerName，以及一个要反射的（UFUNCTION）成员函数，HelloUE5。

我们在 MyGame\Intermediate\Build\Win64\MyGame\Inc\MyGame 目录可以找到 MyActor. generated. h 和 MyActor. gen. cpp 两个由 UHT 生成的文件，我们打开 MyActor. gen. cpp 可以看到如下代码：

![[ab0d0fb037f864f0f0064f664508dfde_MD5.jpg]]

如上我们可以看到我们的成员函数 HelloUE5 函数的反射代码。

然后我们在看到如下代码：

![[8c545aabeb3e8436853b4963385c52ef_MD5.jpg]]

这里可以看到我们的成员变量 MyPlayerName 的反射代码。所以 UE5 是用 UHT 把 MyActor. h/cpp 自动生成 MyActor. generated. h 和 MyActor. gen. cpp 代码，**这两个生成的代码就包括了把 UFUNCTION 的函数名以及实际函数地址，以及 UPROPERTY 的变量名与变量地址注册到某个字典。**

## UHT 反射机制步骤

书中把反射机制比作一个已经过时的 policy，如今已经 encourage baby 了。所以我不再延续那个比喻。这里我就简单的把 UHT 比作是一个户口登记员。

我们把一个 UClass 比作一本户口本，然后 UFUNCTION 和 UPROPERTY 比作家庭成员。我们需要记录 UFUNCTION 的名字和它的实际地址，比如

-----------------------------------------------

家庭地址： AMyAction

----------------------------------

名字： HelloUE5

类型：成员函数

地址： AMyActor:: HelloUE5 ()

-------------------------------------

名字：MyPlayerName

类型：成员变量

地址：&(AMyActor::MyPlayerName)

我们可以拿着这本户口本去找人，比如我们通过 "HelloUE5" 的名字，可以找到它的函数地址：AMyActor:: HelloUE5 ()。从而找到它。

那么该什么时候去构建这本户口本呢？由于每次启动时，成员函数地址和成员变量地址可能都会发生改变。所以我们这个户口登记工作，需要在引擎每次启动都做一次。

我们把整个户口调查的过程，记录在 MyActor. generated. h 和 MyActor. gen. cpp 两个文件中。如果 AMyActor 类代码发生改变的话，UBT 也会通知到 UHT，把整个户口调查过程的代码重新生成一遍，以免有成员遗漏。

**在生成好的 MyActor. gen. cpp 中，我们会看到一些静态变量的创建，这些静态变量会在 main 函数之前初始化，以以便提前记录好各个类的户口本信息。**

所以 UHT 只是一个分析 MyActor 等代码头文件，生成一份调查户口过程的代码 *. generated. h 和 *. gen. cpp 的工具。

