上节讲到了 UE5 里面的模块的基础知识，这一节我们继续探讨模块，主要讲一下引擎初始化的模块加载顺序相关。

## 引擎初始化模块加载顺序

Windows 平台下 UE5 的入口函数以及初始化流程大概如下图：

![](https://pic3.zhimg.com/v2-876b1c17ae2b9607ce3f22cd3f5874d6_r.jpg)

我们知道 Windows 程序都有一个 WinMain 函数，让我们先来找找这个 WinMain 函数，它位于 Runtime 的 Launch 模块中，WinMain 函数在 Launch 模块的 LaunchWindows.cpp 文件中：

![](https://pic1.zhimg.com/v2-6f980cc9e817c2c931fb0d3a836ffe20_r.jpg)

无论那个平台的 main 函数，都会进而调用到 Launch.cpp 的 GuardedMain 函数，如下：

![](https://pic1.zhimg.com/v2-66a31f239d8ee591ed50ae9d6335ae1c_r.jpg)

我们可以把 GuardedMain 的函数简化为如下：

![](https://pic3.zhimg.com/v2-ea17a5b8dda5d1e3702498f39b0965be_r.jpg)

可见初始化的过程主要会调用 PreInit 和 Init 两个函数。我们先来看看 PreInit 的初始化过程。

在 UE5 中，PreInit 首先会加载如下模块：

1，文件相关：StreamingFile NetworkFile PakFile SandboxFile 等

2，通过 LoadCoreModules 函数加载核心模块 CoreUObject。该模块包括了 UClass，UObject 等基础类实现

3，通过 LoadPreInitModules 加载 Engine， Renderer，AnimGraphRuntime 等预初始化模块

如下图：

![](https://pic2.zhimg.com/v2-ddd98c85ba74fd3722411c0ec08589e5_r.jpg)

接下来，同样在 LoadPreInitModules() 函数，还会依次加载

SlateRHIRenderer

Landscape

RenderCore

TextureCompressor （WITH_EDITOR）

AudioEditor（WITH_EDITOR）

PropertyEditor（WITH_EDITOR）

AnimationModifiers（WITH_EDITOR）

进而加载如下模块：

DesktopPlatform

Media

D3D12RHI

InputCore

……

然后通过 LoadStartupCoreModules() 函数，加载如下模块：

Core （核心模块）

Networking

然后是一些平台相关模块：

HeadMountedDisplay

Messaging

MRMesh

UnrealEd（WITH_EDITOR）

EditorStyle（WITH_EDITOR）

SlateCore

Slate

UMG

MessageLog

……

接下来，会根据启用的插件，加载对应的模块，然后是：

TaskGraph

ProfilerService

……

至此，虚幻引擎初始化完成所需的模块加载。

好的，这节就分享到这里了，下一节会讲 UBT 和 UHT，敬请期待！