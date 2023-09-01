## 概述

本节主要探讨虚幻引擎的 UObject 的实例化构造相关，并会简单探讨 UObject 保存到资源包与加载资源包等内容，主要对应原书第 10 章的 10.1.1 和 10.1.2 两个小节，让我们开始吧！

我们先来复习一下，UObject 是 UE 引擎大部分类的基类，因为它有如下特点：

![[411d555ce08e632b6c2b0957af93492e_MD5.jpg]]

我们把新建一个类的对象的操作叫做实例化：

![[89f6c4254f6f39d2f85f7af3de5f18fd_MD5.jpg]]

其中我们的 UObject 的实例化，需要调用 NewObject<T>() 函数，而不能直接用 C++ 的 new 操作符。

## 测试 UObject 类

我们先新建一个自己的 UObject 类的子类，并利用这个子类来理解 UObject 的实例化，与序列化操作。首先我们打开我们的 MyGame 项目，然后在内容浏览器的 All --> C++ 类 --> MyGame 那里右键，点击 “新建 C++” 类：

![[71ddbaf3f77c3fc6843cb1a72df6dfa2_MD5.jpg]]

这样，我们就创建一个 UObject 的子类，UPlayerObject 类了！我们打开这个 PlayerObject.h 代码，并修改如下：

![[3a08ce701ebbd4722152698a136a7f81_MD5.jpg]]

好的，我们的 UObject 测试子类准备好了。让我们开始测试它吧！首先是实例化，这里我们用之前的 Automation System 来做代码例子。

## UObject 的对象实例化

首先我们在项目目录的 Source/MyGame/Private/Tests/ 文件夹创建了一个空的 ObjectTest.cpp 文件，并右击 MyGame.uproject，重新生成了项目文件 MyGame.sln，然后用 Visual Studio 打开项目，如下图：

![[78363221e307f05f2b88ca845e7a811c_MD5.jpg]]

我们打开 ObjectTest.cpp 并输入如下测试代码：

```
#include "CoreMinimal.h"
#include "../PlayerObject.h"

DEFINE_LOG_CATEGORY_STATIC(TestLog, Log, All);

IMPLEMENT_SIMPLE_AUTOMATION_TEST(FObjectTest, "MyTest.PublicTest.ObjectTest", EAutomationTestFlags::EditorContext | EAutomationTestFlags::EngineFilter)

// 测试主函数
bool FObjectTest::RunTest(const FString& Param)
{
	UPlayerObject *Link = NewObject<UPlayerObject>();
	Link->CurPlayerName = TEXT("Link");
	Link->CurAge = 117;

	UE_LOG(TestLog, Display, TEXT("Done ......"));
	return true;
}
```

然后我们编译运行，并在 UE5 编辑器打开 “工具”--> “会话前端”：

![[abd8523b8814bc308124fae079ec0aeb_MD5.jpg]]

然后在打开的 “会话前端” 窗口中，选择“自动化”Tab 按钮，然后勾上 MyTest/PublicTest 里面的 ObjectTest，并点击“开始测试”，记得在 UPlayerObject *Link = NewObject<UPlayerObject>(); 这行代码打上断点，并开始断点调试它。

经过我的断点调试发现，UE5 的创建 UObject 的过程，跟书中的附图的过程基本上是一样的：  

![[a4951495af3b82d909f1384da2854dba_MD5.jpg]]

我们先看看上图的**内存分配阶段**：

![[774729949f6a9aa3172453d48c736452_MD5.jpg]]

我们先来看看对象大小，首先我们这个 UPlayerObject 的内存大小是 72 个字节，那么这 72 个字节是由哪些内容组成呢？我们可以调试打印一下 sizeof(UObjectBase)， 以及 sizeof(UObject)， 发现两者都是 48 个字节，经过断点调试与变量的内存分析，可以得出 UPlayerObject 的内存布局如下：

![[15e06b5de6b3340924d076e5895fb7d2_MD5.jpg]]

所以我们 GetPropertiesSize() 得到 72 字节，然后我们调用 FMemory::Malloc 函数分配这 72 个字节的内存，并调用 FMemory::Memzero 函数把这 72 个字节的内容清空，这时它的所有成员变量的值为空，包括虚函数表指针：

![[349b4d122d76186f1422d9ca8c3363d9_MD5.jpg]]

然后我们给它调用 UObjectBase 的构造函数，初始化好它的 UObectBase 的各个成员变量：

![[adf141d5620436e1c972554e8354ffc8_MD5.jpg]]

这样就完成了我们的内存分配阶段。

然后我们看看**对象构造阶段**：

这里我们先构造一个 FObjectInitializer 对象作为构造函数参数，然后通过 InClass->ClassConstructor() 函数调用到这个类的默认构造函数，这里为什么不用 PlacementNew 来直接完成构造，而需要用函数指针呢？主要是为了灵活，虚幻引擎希望能够获得某个类的构造函数，就像一个点金手一样，划出一片内存，然后点一下就会模塑出一个类的对象。甚至在点金手的时候，**不需要知道这个点金手所属的到底是哪个类**。如果使用 PlacementNew 方案就地构造，就必然需要传入类型参数作为模板参数，不够灵活。

## 对象的保存与加载

我们先保存一个我们 UPlayerObject 对象到到一个资源包文件 (*.uasset) 出来，以便以后可以随时加载它。首先，修改之前的 ObjectTest.cpp 增加一个 SavePlayerAsset 来保存我们的 PlayerObject，代码如下：

```
// 保存 一个UPlayerObject的对象到指定路径的.uasset资产
bool SavePlayerAsset(const FString &AssetPath, const FString &PackageFileName, const FString &ObjectName)
{
	// 创建一个空的资源Package
	UPackage* Package = CreatePackage(nullptr, *AssetPath);
	Package->FullyLoad();

	// 创建“林克”对象时，指定他对应的Package就是刚才创建的空资源Package
	UPlayerObject* Link = NewObject<UPlayerObject>(Package, FName(*ObjectName), EObjectFlags::RF_Public | EObjectFlags::RF_Standalone);

	// 设置“林克”的对象属性
	Link->CurPlayerName = ObjectName;
	Link->CurAge = 117;
	Package->MarkPackageDirty();

	// 保存这个“林克”对象到一个指定路径的*.uasset资源文件
	bool bSaved = UPackage::SavePackage(Package, Link, EObjectFlags::RF_Public | EObjectFlags::RF_Standalone, *PackageFileName, GError, nullptr, true, true, SAVE_NoError);
	return bSaved;
}
```

然后我们修改测试入口函数：RunTest 如下：

```
// 测试入口函数
bool FObjectTest::RunTest(const FString& Param)
{
	// 资源包名
	FString AssetPath = TEXT("/Game/Link");
	// 资源路径，这里是： MyGame/Content/Link.uasset
	FString PackageFileName = FPackageName::LongPackageNameToFilename(AssetPath, FPackageName::GetAssetPackageExtension());
	// 对象名，就是 我们PlayerObject在资源包里面的名字，这里是Link，
	// 一般资源包里面的主要对象的名字，跟资源包名的最后字段是一致的。
	// 比如 资源包名是 /Game/Link， 那么对象名就是Link。对象全路径名就是 /Game/Link.Link
	FString ObjectName = TEXT("Link");

	// 如果资源包MyGame/Content/Link.uasset已存在，则不需要重复保存
	if (FPaths::FileExists(PackageFileName))
	{
	}
	// 否则MyGame/Content/Link.uasset不存在，让我们来保存一下资源包
	else
	{
		bool bSaved = SavePlayerAsset(AssetPath, PackageFileName, ObjectName);

		if (bSaved)
		{
			UE_LOG(TestLog, Display, TEXT("Save Package Success! %s "), *PackageFileName);
		}
	}

	UE_LOG(TestLog, Display, TEXT("Done ......"));
	return true;
}
```

然后我们我们编译运行，并在 UE5 编辑器打开 “工具”--> “会话前端”，并在“会话前端” 界面选择“自动化”Tab 按钮，然后勾上 MyTest/PublicTest 里面的 ObjectTest，并点击“开始测试”。

然后我们可以看到结果：

![[f0956a7bd2f27332980075caf71a7ccb_MD5.png]]

然后我们在 MyGame/Content 目录，也看到了这个 Link.uasset 文件了：

![[de37f93a5ead4409586bb390bb9b3035_MD5.jpg]]

好的，让我们来尝试加载一下它，我们修改 ObjectTest 的测试入口函数 RunTest，在 if(FPaths::FileExists(PackageFileName)) 判断分支，增加如下代码：

```
// 如果资源包MyGame/Content/Link.uasset已存在，则不需要重复保存
	if (FPaths::FileExists(PackageFileName))
	{
		UPackage* Package = LoadPackage(nullptr, *AssetPath, LOAD_None);
		Package->FullyLoad();

		const TArray<FObjectExport> &ExportMap = Package->LinkerLoad->ExportMap;

		UPlayerObject* Link = nullptr;
		for (const FObjectExport& CurObjExport : ExportMap)
		{
			if (CurObjExport.ObjectName.ToString() == ObjectName)
			{
				Link = (UPlayerObject *)(CurObjExport.Object);
			}
		}

		UE_LOG(TestLog, Display, TEXT("Load Package Success! %s "), *PackageFileName);
	}
```

然后我们运行，并打断点调试，可以看出 Link 对象也从加载出来，Link.uasset，并且它的 CurPlayerName 与 CurAge 跟之前设置的是一致的。

## 总结

好的，这一节我们学习了 UObject 的实例化构造过程，并且了解了 UObject 如何保存到资源包，如何从之前保存的资源包中加载资源和对象的方法。

下一节我们会详细探讨 UObject 的序列化，敬请期待！~