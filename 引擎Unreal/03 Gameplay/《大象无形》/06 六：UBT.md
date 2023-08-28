今天主要介绍分享一下书中第 8.4 节 道常无名：UBT 和 UHT 简介 一节。

本节会先着重介绍 UBT 工具，对 UHT 与代码反射的介绍详见下一节。

## 概述

那么 UBT 和 UHT 到底是什么呢？

### UBT

首先 UBT 全称为 **UnrealBuildTool ，**意思是虚幻引擎的构建工具，它主要负责分析 build.cs 和 Target.cs 等配置文件，并把虚幻的 *.h 和 *.cpp 文件编译并链接为二进制可执行文件 exe/dll。如下图：

![[60d60ced445bf2553f492337a2351578_MD5.jpg]]

### UHT

而 UHT 的全称为 **UnrealHeaderTool，**意思是虚幻引擎的头文件工具，主要负责解析各个头文件，并分析其中的有 UCLASS()，USTRUCT()等前缀的类，以及 UFUNCTION()等前缀的函数，并且把这些类与函数等信息通过生成 (*.generate.h) 等代码反射给蓝图使用，如下图：

![[1924824db73600cd0c5646c61ffc4777_MD5.jpg]]

### 完整编译流程

UBT 收集目录中的 *.cs 文件，解析各种编译配置；然后 UBT 调用 UHT 分析需要分析的 *.h 文件，根据文件是否有 UCLASS()，UFUNCTION（）等宏，来反射信息，并生成 *.generated.h 和 *.gen.cpp 文件；最后 UBT 调用 MSBuild，将相关 *.h/ *.cpp 代码，以及生成的 *.generated.h 和 *.gen.cpp 一起编译。

![[30fc0c7e1f64141c9591546259720ca2_MD5.jpg]]

## UBT 概述

UBT 是用 C# 做的一个命令行可执行程序 exe，它的代码在 Engine\Source\Programs\UnrealBuildTool，我们可以在 UE5 的解决方案的 Programs 目录中找到它：

![[fe840424eb723a2478debd037fc47322_MD5.jpg]]

它编译后生成的 exe 文件位于 Engine\Binaries\DotNET\UnrealBuildTool 目录，我们可以进入控制台（cmd）在这个目录，输入 UnrealBuildTool.exe 启动这个工具：

![[5a8d1d6bae0871194740f988ca56f66c_MD5.jpg]]

### UBT 命令行模式

**一，生成项目解决方案**

我们知道当我们刚创建一个名为 MyGame 游戏项目时，会右键点击 MyGame.uproject，然后点击 “Generate Visual Studio project file” 以便生成 MyGame.sln 的解决方案，方便用 Visual Studio 打开它来编译。

![[74500a8f59359cb267c68d829b8862ab_MD5.jpg]]

其实，这个操作也能直接调用 UBT 的命令行来实现，我们先在命令行模式，cd 到 Engine\Binaries\DotNET\UnrealBuildTool 目录，然后输入：

UnrealBuildTool.exe -projectfiles -project=I:/ue5/Projects/MyGame/MyGame.uproject -game -rocket -progress -log=I:/ue5/log1.txt  
也可以生成 MyGame.sln 文件。

![[742532f0b9380794279df474b7379c49_MD5.jpg]]

同样，我们也可以在 Visual Studio 中调试这个 UnrealBuildTool 工具的 C# 源代码。我们先右键点击这个项目，然后点击 “属性”，在属性那里选择“调试”，然后在“命令行参数” 那里输入：-projectfiles -project=I:/ue5/Projects/MyGame/MyGame.uproject -game -rocket -progress -log=I:/ue5/log1.txt

如下图：  

![[69831e049fd22104945a0a1be7902edd_MD5.jpg]]

然后选择这个项目，并设置为 “启动项目”，然后按 F5 即可断点调试 UnrealBuildTool 的 C# 源代码：

![[a12b24de496258bea8fc8cffdde236a6_MD5.png]]

**二，编译项目**

我们也可以用 UBT 的命令行来编译一个 UE5 的项目，比如我们可以 cd 到 Engine\Binaries\DotNET\UnrealBuildTool 目录，然后输入：

UnrealBuildTool.exe MyGameEditor Win64 Development -Project=I:/ue5/Projects/MyGame/MyGame.uproject

它就会编译 MyGame 项目在 Win64 平台的 Development Editor 配置的目标 exe 出来，如下：

![[a4d378601c6755849bb037444430b5fb_MD5.jpg]]

我们看到第一个 参数为 MyGameEditor，这个是什么意思呢？它表示生成 MyGame 项目的 Editor 目标，即带 UE5 编辑器的目标，方便内部开发与调试。如果这个输入 MyGame 的话，它是编译发布给玩家的 game.exe，不带 UE5 编辑器，资源也必须提前 cook 好，不然运行不了程序。 其中，这两个命令是跟解决方案的 *.Target.cs 文件一一对应的，如下图：

![[0a5d8a392ce82482fc017cce32733e9e_MD5.jpg]]

到这里，我们发现其实我们只要安装过 Visual Studio 的 C++ 编译环境，其实脱离 Visual Studio 编辑器也可以在命令行编译 UE5 引擎与游戏了。这也是为什么 Rider 等其他编辑器也能编辑与编译 UE5 引擎代码的原因。甚至我们还可以用 vim 修改引擎代码，然后用命令行编译和运行引擎。

那么 UBT 我们先介绍到这里，如果读者对 UBT 本身的代码感兴趣的话，可以在 Visual Studio 把 UnrealBuildTool 项目设置为 “启动项目”，并且右击项目，点击“属性”，在属性那里选择“调试”，然后在“命令行参数” 输入类似：

MyGameEditor Win64 Development -Project=I:/ue5/Projects/MyGame/MyGame.uproject

就能在 Visual Studio 中断点调试这个 UBT 工具本身的 C# 代码与原理了。笔者暂时对 UBT 的底层原理这块就不求甚解了。另外书中还详细提到了 WinMain 函数代码是怎么编译进 exe 的，这块有兴趣的同学也请自己参照下书本上的内容。

下一节我们会分享代码反射与 UHT 工具的基本原理，敬请期待！