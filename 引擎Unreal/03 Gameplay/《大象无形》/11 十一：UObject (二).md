## 概述

接上回[大象无形 UE 笔记十：UObject (一)](https://zhuanlan.zhihu.com/p/585055258)，本节主要探讨 UObject 的序列化，对应原书中的 10.1.2 小节。让我们开始吧！

序列化时指将一个对象变为更容易保存的形式，写入持久存储中（比如硬盘）。反序列化则反之，从持久存储中读取数据，然后还原原来的对象。如下图：

![[414094de45ca8fc8d785c2c892b1d5cd_MD5.jpg]]

UObject 的序列化和反序列化都对应同一个函数 Serialize。正向写入和反向读取需要按同样的方式进行。

需要注意的是，序列化 / 反序列化的过程是一个 “步骤”，而不是一个完整的对象初始化过程。反序列化的主要步骤是：先实例化对象，然后再反序列化对象的属性。

我们先来研究一下反序列化对象。因为理解对象如何读取出来后，写入就显得更简单了。

## 反序列化

### 主要步骤

首先我们实例化一个对象出来，步骤大致如下，我们以加载 Content/link.uasset 的名为 “link” 的 UPlayerObject 对象来举例（参照 [UObject(一)](https://zhuanlan.zhihu.com/p/585055258)）：

![[8885c012d3ad6357a613057f8acc8c3c_MD5.jpg]]

让我们详细看看最后一个步骤，即载入所有脚本成员变量信息，这里的代码主要在 UObject::Serialize(...) 函数中，它主要有如下几个步骤：

1，调用 FArchive::MarkScriptSerializationStart 函数，标记脚本数据序列化开始。

2，调用 ObjClass 对象的 SerializeTaggedProperties 函数，进而调用类的每个 Tag 的 Tag.SerializeTaggedProperty() 函数，载入脚本定义的成员变量，比如我们的 UPlayerObject 里面自定义的 CurPlayerName 和 CurAge 两个成员变量：

![[705b83a4d6d5379bed76a7f9153ad8bb_MD5.jpg]]

3，调用 MarkScriptSerializationEnd 标记脚本数据序列化结束。

### 切豆腐理论

对于硬盘上保存的数据来说，其本身不具备 “意义”，其含义取决于我们如何**解释**这一段数据。我们每次反序列化一个 C++ 基本对象，例如一个 float 浮点数，我们是从豆腐最开头切下一块和浮点数长度一样大的豆腐，然后把这块豆腐**解释**为一个浮点数。

![[17675d6e7dab7c35a1111a332b36d52e_MD5.jpg]]

那读者会问，你怎么知道豆腐这一块区域就是浮点数？答案是，我们按照什么顺序把豆腐排起来的，并且按同样的顺序切，那就能保证每次切出来都是正确的。比如我放了三块豆腐：豆腐 1（float），豆腐 2（bool），豆腐 3（double）。把它们按顺序靠近摆放，组合成一块大豆腐，然后把它放冰箱里冻起来。下次吃的时候，由于我知道豆腐 1，豆腐 2，豆腐 3 的长度。所以我在 1 号长度切一刀，1 和 2 号长度总和处切一刀，妥了！三块豆腐就分开了。

![[494f8a04b36f7d99c7619dc834ccea9e_MD5.jpg]]

### 递归序列化 / 反序列化

一个类的成员会有以下几种类型：C++ 基本类型，自定义类型的对象，自定义类型的指针：

![[d0184ab56a4828c20cd0540fcefad7e9_MD5.jpg]]

对于 C++ 基础类型对象，直接调用切豆腐理论序列化，对于自定义对象则调用该类型的序列化函数自行去切豆腐，然后把切好的豆腐还回来即可。对于自定义类型的指针，这里暂时不讲解，由我们后面的 UPakcage 包的对象引用那部分再讲。

![[842cc1b66bd0b461637e448ac60163db_MD5.jpg]]

理解这两个概念之后，我们就能开始分析虚幻引擎的反序列化过程。这其实是一个两步的步骤：

![[16ea66dd71c4d7173cd6787364e9c814_MD5.jpg]]

同时，虚幻引擎，对序列化后的大小进行了优化。对于每个类都有一个类的默认对象即 CDO，我们的序列化只保存跟 CDO 的差异部分即可。比如如果 PlayerObject 的 CurPlayerName == "" 的话， 则不需要占用序列化后的存储空间。

接下来的问题就是，自定义对象的指针如何序列化，这里就涉及到 UObject 对象的互相引用，以及加载 UPackage 包，如何加载引用问题了，我们来探讨一下 UPackage 包的相关知识吧。

## UPackage 资源包

### 概述

虚幻引擎中，在 Content 目录中保存资源的 *.uasset 文件，都是 UPackage 资源包。无论是纹理资源，材质资源，静态模型，骨骼模型，骨架，动画序列，动画蒙太奇，蓝图，场景 *.umap 等等，其实都是 UPackage 包。也就是 UPackage 就是虚幻引擎存储资源的统一方式。

### 包结构

UPackage 包体的数据结构大致如下：

![[763f6c9c67a1bb3a78a9de62672161ab_MD5.jpg]]

如上图，我们重点关注的是 Import table，Exports table，以及 Export-objects 这几个项目，我们先把 [上一节](https://zhuanlan.zhihu.com/p/585055258)节保存的 link.uasset 加载起来，然后看看 link.uasset 的包体结构。我们在[上一节](https://zhuanlan.zhihu.com/p/585055258)保存了一个 Content/link.uasset，并且用 LoadPackage 把它加载出来了，我们可以在 Package->FullyLoad(); 这句的后面打一个断点，并在命中断点时用 VisualStudio 的变量查看功能来观察这个包体数据：

![[d5d3a64c8e1a17023b4daae596268f82_MD5.jpg]]

经过我们的调试分析，我们得出：link.uasset 的 UPackage 数据如下：

![[eb2d1630f7b607b67472aa8be84b4c05_MD5.jpg]]

可以看到我们包体保存的 UObject 对象主要是 Link，它是 UPlayerObject 的对象，另外还有一个 PackageMetaData 应该是一些额外数据，我们暂时忽略。在导入表（import table）中它引用的主要都是 C++ 对象，即 Script/CoreUObject 里面的对象，包括 UPlayerObject 类以及 MyGame 模块等。

### UPackage 包的引用关系

我们可以在内容浏览器中，右键点击一个资源，并点击 “引用查看器”，可以查看该资源引擎哪些其他资源，我们右击 link.uasset 试试：

![[ac98f0d00e5d8318841bf93b66382eba_MD5.jpg]]

可以看到 link.uasset 现在没有引用任何其他资源，也没有被其他资源所引用。现在让我们来给它加一些引用了。我们在 MyGame/Private/Tests/ 目录来创建一个 ObjectRefTest.cpp，关闭 VisualStudio，并右击 MyGame.uproject，重新生成 MyGame.sln 项目。

然后我们修改 PlayerObject.h，加上一个 UPROPERTY 成员变量：

```
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "PlayerObject")
TObjectPtr<UPlayerObject> Lover;
```

![[63278106a16e9e8b56f8ae962d121f4f_MD5.jpg]]

然后我们把之前创建的 Link.uasset 文件删除掉，并打开在 ObjectRefTest.cpp 输入如下代码：

```
#include "CoreMinimal.h"
#include "../PlayerObject.h"

DEFINE_LOG_CATEGORY_STATIC(TestLog, Log, All);

IMPLEMENT_SIMPLE_AUTOMATION_TEST(FObjectRefTest, "MyTest.PublicTest.ObjectRefTest", EAutomationTestFlags::EditorContext | EAutomationTestFlags::EngineFilter)

// 保存 两个UPlayerObject的对象到*uasset资产
bool SaveTwoPlayerAsset(const FString &AssetPath1, const FString &PackageFileName1, const FString &ObjectName1,
	const FString& AssetPath2, const FString& PackageFileName2, const FString& ObjectName2)
{
	// 创建两个空的资源Package
	UPackage* Package1 = CreatePackage(nullptr, *AssetPath1);
	UPackage* Package2 = CreatePackage(nullptr, *AssetPath2);

	// 创建“林克”对象时，指定他对应的Package就是刚才创建的空资源Package1
	UPlayerObject* Link = NewObject<UPlayerObject>(Package1, FName(*ObjectName1), EObjectFlags::RF_Public | EObjectFlags::RF_Standalone);
	// 创建“塞尔达”对象时，指定她对应的Package就是刚才创建的空资源Package2
	UPlayerObject* Zelda = NewObject<UPlayerObject>(Package2, FName(*ObjectName2), EObjectFlags::RF_Public | EObjectFlags::RF_Standalone);


	// 设置“林克”的对象属性
	Link->CurPlayerName = ObjectName1;
	Link->CurAge = 117;
	Link->Lover = Zelda; // 林克的恋人是塞尔达。哈哈
	Package1->MarkPackageDirty();

	// 设置“塞尔达”的对象属性
	Zelda->CurPlayerName = ObjectName2;
	Zelda->CurAge = 117;
	Package2->MarkPackageDirty();

	// 保存这个“林克”对象到一个指定路径的*.uasset资源文件
	bool bSaved = UPackage::SavePackage(Package1, Link, EObjectFlags::RF_Public | EObjectFlags::RF_Standalone, *PackageFileName1, GError, nullptr, true, true, SAVE_NoError);
	// 保存这个“塞尔达”对象到一个指定路径的*.uasset资源文件
	bSaved = UPackage::SavePackage(Package2, Zelda, EObjectFlags::RF_Public | EObjectFlags::RF_Standalone, *PackageFileName2, GError, nullptr, true, true, SAVE_NoError);
	return bSaved;
}

// 测试入口函数
bool FObjectRefTest::RunTest(const FString& Param)
{
	// 资源包名
	FString AssetPath = TEXT("/Game/Link");
	// 资源路径，这里是： MyGame/Content/Link.uasset
	FString PackageFileName = FPackageName::LongPackageNameToFilename(AssetPath, FPackageName::GetAssetPackageExtension());
	// 对象名，就是 我们PlayerObject在资源包里面的名字，这里是Link，
	// 一般资源包里面的主要对象的名字，跟资源包名的最后字段是一致的。
	// 比如 资源包名是 /Game/Link， 那么对象名就是Link。对象全路径名就是 /Game/Link.Link
	FString ObjectName = TEXT("Link");

	// 资源包名
	FString AssetPath2 = TEXT("/Game/Zelda");
	// 资源路径，这里是： MyGame/Content/Link.uasset
	FString PackageFileName2 = FPackageName::LongPackageNameToFilename(AssetPath2, FPackageName::GetAssetPackageExtension());
	// 对象名，就是 我们PlayerObject在资源包里面的名字，这里是Zelda
	FString ObjectName2 = TEXT("Zelda");


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
	// 否则MyGame/Content/Link.uasset不存在，让我们来保存一下资源包
	else
	{
		bool bSaved = SaveTwoPlayerAsset(AssetPath, PackageFileName, ObjectName, AssetPath2, PackageFileName2, ObjectName2);

		if (bSaved)
		{
			UE_LOG(TestLog, Display, TEXT("Save Package Success! %s "), *PackageFileName);
		}
	}

	UE_LOG(TestLog, Display, TEXT("Done ......"));
	return true;
}
```

记得把之前创建的 Link.uasset 文件删除掉。

然后我们打开 “工具”--> “会话前端” -->“自动化”，并勾选“MyTest/PublicTest” 的 "ObejctRefTest"，“开始测试”：

![[3a7d93a3720fb5fbf3782f089659077f_MD5.jpg]]

然后我们就可以发现 Content 里面多了两个文件，Link.uasset 和 Zelda.uasset：

![[7005fe8dcd3dc17765fc0a0cb1fe4b82_MD5.jpg]]

我们再打开引用查看器看看 Link.uasset 的引用，可以发现，Link 是依赖 Zelda 的：

![[8223f1432995bdf685f33c0fc152304c_MD5.jpg]]

我们也可以双击打开 Link.uasset 看到他的三个属性：

![[16e80cd3ad857b0a60f8488a5868c791_MD5.jpg]]

然后，我们在 ObjectRefTest.cpp 中的 Package->FullyLoad(); 的下一行打一个断点，并观察一下 Package 的结构：

![[8ae57670b7af9afc633c31991bb01edc_MD5.jpg]]

经过我们的调试分析，我们得出：当前 link.uasset 的 UPackage 数据如下：

![[537bb9a7c0926b224312588136cb0743_MD5.jpg]]

如上，可以看出，Link 的 UPackage 的导入表 (Import table) 多了 /Game/Zelda 的 UPackage 包，以及 Zelda 的依赖。另外，我们再调试一下加载过程：

![[0384c9037bf3602aafd92758c10e1ced_MD5.png]]

如上可以看出在加载 Link.uasset 时，会调用另一个 VerityImport 的函数，这里的 importIndex 传的是 2，当它发现导入表 (import table) 的 第二个元素 /Game/Zelda 还没有加载进来，于是它会触发 LoadPackageInternal 去把 / Game/Zelda 的 uasset 也加载进来。这个就是 UE 加载包引用的方式。

好的，我们的 UObject 的序列化，差不多就讲到这里，原书上对加载 UPackage 的相互引用时，还有一个 “班级早恋” 的例子，我这里就不重复了，大家有兴趣可以自己翻一下书。

下一节我们会分享 UObject 的垃圾收集相关，敬请期待！