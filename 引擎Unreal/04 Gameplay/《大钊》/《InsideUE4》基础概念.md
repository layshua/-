# 创建测试项目  

接上文的准备工作，双击生成的 UE4Editor.exe，选择创建测试 C++ 空项目 Hello（以后的源码分析都会基于该最简单的项目）  

![[38ca4571e3aea97ce23ea9a5ee7afbac_MD5.jpg]]

## 项目文件结构

VS 项目和文件目录：

![[75fcc8463f106ebd6d18d543c89ab268_MD5.png]]

![[28c76fbd1a32f10695dba150251b2a31_MD5.png]]

可以看到，Config 目录里带着 3 个最主要的配置，Editor,Engine,Game。代码方面自动生成了用于编译系统的 3 个. cs 文件，C++ 代码方面生成了一个 Hello "Game Module"，和 HelloGameMode。  
文件目录：

*   Binaries: 存放编译生成的结果二进制文件。该目录可以 gitignore, 反正每次都会生成。
*   Config: 配置文件。
*   Content: 平常最常用到，所有的资源和蓝图等都放在该目录里。
*   DerivedDataCache：“DDC”，存储着引擎针对平台特化后的资源版本。比如同一个图片，针对不同的平台有不同的适合格式，这个时候就可以在不动原始的 uasset 的基础上，比较轻易的再生成不同格式资源版本。gitignore。
*   Intermediate：中间文件（gitignore），存放着一些临时生成的文件。有：  
    *   Build 的中间文件，.obj 和预编译头等
    *   UHT 预处理生成的. generated.h/.cpp 文件
    *   VS.vcxproj 项目文件，可通过. uproject 文件生成编译生成的 Shader 文件。
    *   AssetRegistryCache：Asset Registry 系统的缓存文件，Asset Registry 可以简单理解为一个索引了所有 uasset 资源头信息的注册表。CachedAssetRegistry.bin 文件也是如此。
*   Saved：存储自动保存文件，其他配置文件，日志文件，引擎崩溃日志，硬件信息，烘培信息数据等。gitignore
*   Source：代码文件。

## 编译类型

很多人在使用 UE4 的时候，往往只是依照默认的 DevelopmentEditor，但实际上编译选项是非常重要的。  
UE4 本身包含网络模式和编辑器，这意味着你的工程在部署的时候将包含 Server 和 Client，而在开发的时候，也将有 Editor 和 Stand-alone 之分；同时你也可以单独选择是否为 Engine 和 Game 生成调试信息，接着你还可以选择是否在游戏里内嵌控制台等。

![[f2a843ddf837ffbe13441fcfa97951f6_MD5.png]]

依照[官方介绍](https://docs.unrealengine.com/latest/INT/Programming/Development/CompilingProjects/index.html)

每种编译配置包含两种关键字。第一种表明了引擎以及游戏项目的_状态_。第二个关键字表明正在编译的_目标_。

![[bd0e7bfa1856beb62a7e85f895470ce1_MD5.png]]

![[46a4cfd59a87d70d4c8d9fed23fac08e_MD5.jpg]]

**组合的各种情况：**

![[b54db34c036f47a0b26358d554725632_MD5.png]]

所以为了我们的调试代码方便，我们选择 DebugEditor 来加载游戏项目，当需要最简化流程的时候用 Debug 来运行独立版本。

## 命名约定

客观来说，相比其他引擎的源码，UE4 的源码还是非常清晰的，模块组织也比较明了。但阅读源码的学习曲线依然陡峭，我想有以下原因：  
1. UE4 包含的模块众多，拢共有几十个模块，虽然采用了 Module 架构来解耦，但难免还是要有依赖交叉的地方，在阅读的时候就很难理清各部分的关系。  
2. UE4 的功能优秀，作为业界顶尖的成熟游戏引擎，在一些具体的模块内部实现上就脱离了简单粗暴，而是采用了各种设计模式和权衡。同时也需要阅读的人有相关的业务知识。比如材质编辑器编译生成 Shader 的过程就需要读者拥有至少差不多的图形学知识。  
3. 被魔改后的 C++，UE4 为了各平台的编译和其他考量（具体以后说到编译系统的时候再细讨论），对标准的 C++ 和编译，进行了相当程度的改造，在 UHT 代码生成和各种宏的嵌套之后，读者就很难一下子看清背后的各种的机制了。

但万丈高楼平地起，咱们也可以从最简单的一步步开始学起，直到了解掌握整个引擎的内部结构。  
在阅读代码之前，就必须去了解一下 [UE4 的命名约定](http://docs.unrealengine.com/latest/INT/Programming/Development/CodingStandard/)，具体的自己去查看官网文档，下面是一些基本需要知道的：  

*   模版类以 T 作为前缀，比如 TArray,TMap,TSet UObject 派生类都以 U 前缀  
    
*   AActor 派生类都以 A 前缀  
    
*   SWidget 派生类都以 S 前缀  
    
*   抽象接口以 I 前缀  
    
*   枚举以 E 开头  
    
*   bool 变量以 b 前缀，如 bPendingDestruction  
    
*   其他的大部分以 F 开头，如 FString,FName  
    
*   typedef 的以原型名前缀为准，如 typedef TArray FArrayOfMyTypes;  
    
*   在编辑器里和 C# 里，类型名是去掉前缀过的  
    
*   UHT 在工作的时候需要你提供正确的前缀，所以虽然说是约定，但你也得必须遵守。（编译系统怎么用到那些前缀，后续再讨论）  
    

## 基础概念

和其他的 3D 引擎一样，UE4 也有其特有的描述游戏世界的概念。在 UE4 中，几乎所有的对象都继承于 UObject（跟 Java,C# 一样），UObject 为它们提供了基础的垃圾回收，反射，元数据，序列化等，相应的，就有各种 "UClass" 的派生们定义了属性和行为的数据。  
跟 Unity（GameObject-Component）有些像的是，UE4 也采用了组件式的架构，但细品起来却又有些不一样。在 UE 中，3D 世界是由 Actors 构建起来的，而 Actor 又拥有各种 Component，之后又有各种 Controller 可以控制 Actor（Pawn）的行为。Unity 中的 Prefab，在 UE4 中变成了 BlueprintClass，其实 Class 的概念确实更加贴近 C++ 的底层一些。  
Unity 中，你可以为一个 GameObject 添加一个 ScriptComponent，然后继承 MonoBehaviour 来编写游戏逻辑。在 UE4 中，你也可以为一个 Actor 添加一个蓝图或者 C++ Component, 然后实现它来直接组织逻辑。 UE4 也支持各种插件。  
其他的下篇再一一细说。

## 编译系统

UE4 支持众多平台，包括 Windows,IOS，Android 等，因此 UE4 为了方便你配置各个平台的参数和编译选项，简化编译流程, UE4 实现了自己的一套编译系统，否则我们就得接受各个平台再单独配置一套项目之苦了。  
这套工具的编译流程结果，简单来说，就是你在 VS 里的运行，背后会运行 UE4 的一些命令行工具来完成编译，其他最重要的两个组件：  

*   UnrealBuildTool（UBT，C#）：UE4 的自定义工具，来编译 UE4 的逐个模块并处理依赖等。我们编写的 Target.cs，Build.cs 都是为这个工具服务的。
*   UnrealHeaderTool （UHT，C++）：UE4 的 C++ 代码解析生成工具，我们在代码里写的那些宏 UCLASS 等和 #include "*.generated.h" 都为 UHT 提供了信息来生成相应的 C++ 反射代码。  
    

一般来说，UBT 会先调用 UHT 会先负责解析一遍 C++ 代码，生成相应其他代码。然后开始调用平台特定的编译工具 (VisualStudio,LLVM) 来编译各个模块。最后启动 Editor 或者是 Game.

更细的留待 “编译系统” 再细细讨论

## 引用

1.  [BuildConfigurationsReference](http://static.zybuluo.com/fjz13/ic5iglbsxohdgtilnzwi80r4/CreateHelloProject.pn)
2.  [UE4 Terminology](http://docs.unrealengine.com/latest/INT/GettingStarted/Terminology/index.html)

上篇：[《Inside UE4》开篇](http://zhuanlan.zhihu.com/p/22814051)

下篇：[《Inside UE4》GamePlay 架构（一）Actor 和 Component](http://zhuanlan.zhihu.com/p/22833151)

---------------------------------------------------------------------------------------------------------------------------

知乎专栏：[InsideUE4](https://zhuanlan.zhihu.com/insideue4)

UE4 深入学习 QQ 群：**456247757**(非新手入门群，请先学习完官方文档和视频教程)

微信公众号：**aboutue**，关于 UE 的一切新闻资讯、技巧问答、文章发布，欢迎关注。

**个人原创，未经授权，谢绝转载！**