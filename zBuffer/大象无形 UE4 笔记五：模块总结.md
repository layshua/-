我们首先来总结一下前面所学的 UE5 的总体代码结构与模块相关知识。

## 总体结构

首先，我们可以先看看总体结构，首先是 UnrealEngine 部分：

![](https://pic3.zhimg.com/v2-1754cca923d73cdb9555e68936792d3e_r.jpg)

如上图，我们可以看到 UnrealEngine 的 Visual Studio 解决方案里，主要有 引擎主体的工程 UE5，其中 UE5 里面有一个 Plugins 目录，即为引擎插件，插件其实是一起编译在引擎里面的，然后还有一个 Programs 目录，会编译很多个独立程序出来，这些独立程序可以脱离引擎独立运行，主要是执行一些类似 Pak 打包，引擎后台服务，编译 Shader 等独立 exe 程序，如下图：

![](https://pic3.zhimg.com/v2-034e9314392be58f69a4c8ec3e6fcfde_r.jpg)

我们会通过 UnrealEngine 生成一个自己的游戏项目，比如名字为 MyGame，然后我们通过 MyGame.uproject 可以生成一个 MyGame.sln 的 visual studio 解决方案，这里的结构更为简单一些：

![](https://pic1.zhimg.com/v2-b82a60e062f23c6e3a23558232af2d1c_r.jpg)

如上，MyGame 依赖了 UE5 引擎，然后还有自己的游戏模块，插件，配置等。

## 代码模块

UE5 最小的代码组织单位是**模块**。每一个模块会有一个 *.Build.cs 文件来描述模块主要结构，源码部分会有 Public 和 Private 两个文件夹：

![](https://pic4.zhimg.com/v2-52eb074551a5f8673ef4b299679d1573_r.jpg)

## 程序 & 插件类别

模块可以组成插件和各种程序，如下图：

![](https://pic1.zhimg.com/v2-ebddce85a33da6ac6264def794eb7e18_r.jpg)

如上图，我们的 独立程序（Program），引擎（UE5），以及 游戏（MyGame）还需要有一个 *.Target.cs 文件，那么这个文件是干嘛的呢？

## *.Target.cs 文件概述

这个 *.Target.cs 是用来描述如何生成一个可执行 exe 的，包括这个 exe 的类型，它的链接方式，以及各种编译选项，编译宏定义等，我们来看几个 exe 来了解一下这些类型。

### **Type 参数详解**

首先我们来看看第一个参数 Type：

![](https://pic2.zhimg.com/v2-f114fdec2c02e5421eb52b81d4e292a5_r.jpg)

我们看了多个 *.Target.cs 文件的内容后，得出，这个 Type 参数大概有如下五种值：

![](https://pic2.zhimg.com/v2-a61a29f62e3d51dee5f878fb404db6a9_r.jpg)

其中前三种 Program，Editor，Game 用得比较多，我们来重点介绍一下：

首先，Program 是独立程序，就是类似 UnrealPak.exe 等，可以脱离 UnrealEngine 环境也能独立运行的 exe，主要是作为处理单一功能的可执行文件。比如 UnrealPak 主要为了把 指定列表的所有 *.uasset 资源，打包成一个 *.pak 包。

![](https://pic2.zhimg.com/v2-49bbfbc167606e636584468660d143cd_r.jpg)

然后，Editor 是附带 UE5 编辑器的程序，我们日常开发游戏 MyGame 时，需要在 UE5 编辑器里面查看各种美术资源，修改关卡和角色蓝图等，然后可以点击运行按钮，来查看游戏内效果，用的资源就是美术提交的散装的原始资源 uasset，有时候进入场景有点卡顿，因为需要运行时编译 shader，压缩贴图等，为了方便我们在开发时修改资源等。所以我们有一个 MyGameEditor.Target.cs 来生成这个附带 UE5 编辑器的 MyGame 项目：

![](https://pic1.zhimg.com/v2-f4e83ee966fe1ebf1dba829df5da350c_r.jpg)

最后，Game 是发布给玩家的游戏包体的程序，比如我们的 MyGame 开发好了，我们会打一个 MyGame 的游戏包体给玩家玩，这个游戏里面没有编辑器，用的资源也是 cook 好并打好 pak 包的二进制资源，因此进入场景会非常流畅，较大得改善了玩家的游戏体验。所以我们有一个 MyGame.Target.cs 来生成这个不带 UE5 编辑器的 MyGame 项目：

![](https://pic3.zhimg.com/v2-89eca2bc7b857fced23193bd300548fa_r.jpg)

还有两个 Client，Server，暂时用的不多，就不做详细介绍了。

### LinkType 参数详解

然后第二个可选参数为 LinkType，并不是每个 *.Target.cs 都定义了这个参数，我们可以看到这个参数主要是：Modular 和 Monolithic 两个值。如果不定义这个参数，默认为 Modular。那么什么是 Modular ，什么是 Monolithic 呢？其实就是说这个 exe 使用得每个模块，需不需要编译成一个 dll？我们可以对比一下 UE5 的 UnrealPak 和 BlankProgram 两个独立程序，可以看出：

![](https://pic3.zhimg.com/v2-f4f3f442cbae13ce3d67b0ac88fc3f4e_r.jpg)

我们可以看到 UnrealEngine 的编辑本身的 LinkType 也是 Modular 的，所以它每个模块都是一个单独的 dll，我们可以打开 UnrealEngine/Engine/Binaries/Win64 文件夹进行查看：

![](https://pic4.zhimg.com/v2-8e5fb654496066b9af1dcabe66197dc7_r.jpg)

### 其他编译选项 & 宏

最后 *.Target.cs 文件还包含了一些编译选项和预定义宏等，我们大概看看：

![](https://pic4.zhimg.com/v2-1824e7e2ddea75ac5b2d2540e7455c33_r.jpg)

## *.Target.cs 文件 vs *.Build.cs 文件

让我们来对比一下 *.Target.cs 文件与 *.Build.cs 文件。它们同样作为给 UBT 处理的描述文件，也是有不同的地方的，首先 *.Target.cs 文件是每个要编译成一个比较独立的程序的目标需要一个，而 *.Build.cs 更广泛，每个模块都有一个。另外，*.Target.cs 主要描述链接信息，编译选项，一些预定义宏等。而 *.Build.cs 文件 主要描述它需要包含哪些头文件目录，需要依赖哪些模块等。

![](https://pic1.zhimg.com/v2-873c68540dd7f133177966e7c550a648_r.jpg)

好的。这一节复习了 UE5 的总体结构，以及模块相关支持，并且详细介绍了一些配置文件，为下一节介绍 UBT 打下了踏实的基础。下一节我们会介绍 UBT 和 UHT 相关，敬请期待！