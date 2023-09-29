
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

下篇：[《Inside UE4》GamePlay 架构（一）Actor 和 Component](http://zhuanlan.zhihu.com/p/22833151)