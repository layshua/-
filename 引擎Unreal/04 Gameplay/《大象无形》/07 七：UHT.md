本节我们来重点学习一下 UHT 工具的概念。对应书中第 8.4 节 道常无名：UBT 和 UHT 简介 的后半节。

## UHT 概述

那么什么是 UHT 呢？UHT 的全称是 Unreal Header Tool，即虚幻头文件工具，它主要分析头文件，找出 UClass()，UFUNCTION() 等标记，并把它反射给蓝图脚本用的工具：

![[1924824db73600cd0c5646c61ffc4777_MD5.jpg]]

### UHT：一个引擎独立运行程序

首先，UHT 是一个引擎独立程序，它是 C++ 写的一个 exe 程序，依赖部分引擎代码：

![[4bffd93405637610b09bdb97b9108575_MD5.jpg]]

我们可以从 UnrealHeaderTool.Build.cs 中看到它其实也依赖少数引擎的核心模块：

![[b62435f0a48ec59af3fc237f671f3084_MD5.jpg]]

在了解 UHT 的具体工作之前，我们需要先了解什么是反射。

## 反射

书中的给的定义就是，我们需要在运行时获取一个类，并且能获取这些类有哪些成员变量，成员函数；还能获取成员变量的名字等。C++ 本身并没有提供这样一套机制。所以需要自己实现。

### 新建测试项目

我们先从 Hello World 程序开始自己实现一套最简单的反射机制。

首先我们新建一个空白的控制台项目：

![[452983b8ad94da9536c6c32c28082ddf_MD5.jpg]]

### 初始测试代码

把项目名命名为 reflect1，然后，我们把初始代码修改如下：

```
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

如上代码，我们有一个动物基类，有 鸭子 和 猫 两个子类，每个子类都有一个 call 函数，表示动物叫。我们创建了一只鸭子，然后让它叫了一声：

Quack!

好的。由于没有反射，目前这个程序非常不灵活，需要用代码指定好自己想创建对象的类 Duck，不能根据用户需求来灵活创建不同的类的对象。比如用户这次运行程序想要创建一个鸭子，下次运行程序又想要创建一只猫了。

### 对象创建工厂

其实我们想要程序更为灵活一些，我们先用一种对象创建工厂的方法，我们在所有动物类定义之后，增加了一个 ClassFactory 的类，并且定义了这个类的一个全局静态实例变量：

```
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

```
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

```
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

```
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

```
#define REGIST(className) \
	Animal *Create##className() { return new className(); } \
	RegistAction g_RegAction##className(#className, Create##className)
```

### 测试工程完整代码

最后我们整个测试程序的代码如下：

```
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

```
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

我们在 MyGame\Intermediate\Build\Win64\MyGame\Inc\MyGame 目录可以找到 MyActor.generated.h 和 MyActor.gen.cpp 两个由 UHT 生成的文件，我们打开 MyActor.gen.cpp 可以看到如下代码：

![[ab0d0fb037f864f0f0064f664508dfde_MD5.jpg]]

如上我们可以看到我们的成员函数 HelloUE5 函数的反射代码。

然后我们在看到如下代码：

![[8c545aabeb3e8436853b4963385c52ef_MD5.jpg]]

这里可以看到 我们的 成员变量 MyPlayerName 的反射代码。所以 UE5 是用 UHT 把 MyActor.h/cpp 自动生成 MyActor.generated.h 和 MyActor.gen.cpp 代码，这两个生成的代码就包括了把 UFUNCTION 的函数名以及实际函数地址，以及 UPROPERTY 的变量名与变量地址注册到某个字典。

## UHT 反射机制步骤

书中把反射机制比作一个已经过时的 policy，如今已经 encourage baby 了。所以我不再延续那个比喻。这里我就简单的把 UHT 比作是一个户口登记员。

我们把一个 UClass 比作一本户口本，然后 UFUNCTION 和 UPROPERTY 比作家庭成员。我们需要记录 UFUNCTION 的名字和它的实际地址，比如

-----------------------------------------------

家庭地址： AMyAction

----------------------------------

名字： HelloUE5

类型：成员函数

地址： AMyActor::HelloUE5()

-------------------------------------

名字：MyPlayerName

类型：成员变量

地址：&(AMyActor::MyPlayerName)

我们可以拿着这本户口本去找人，比如我们通过 "HelloUE5" 的名字，可以找到它的函数地址：AMyActor::HelloUE5()。从而找到它。

那么该什么时候去构建这本户口本呢？由于每次启动时，成员函数地址和成员变量地址可能都会发生改变。所以我们这个户口登记工作，需要在引擎每次启动都做一次。

我们把整个户口调查的过程，记录在 MyActor.generated.h 和 MyActor.gen.cpp 两个文件中。如果 AMyActor 类代码发生改变的话，UBT 也会通知到 UHT，把整个户口调查过程的代码重新生成一遍，以免有成员遗漏。

在生成好的 MyActor.gen.cpp 中，我们会看到一些静态变量的创建，这些静态变量会在 main 函数之前初始化，以以便提前记录好各个类的户口本信息。

所以 UHT 只是一个分析 MyActor 等代码头文件，生成一份调查户口过程的代码 *.generated.h 和 *.gen.cpp 的工具。

## 总结

好了，UHT 我们就暂时分享到这里了。下一节我们可能会分享并发与线程相关的内容，敬请期待！