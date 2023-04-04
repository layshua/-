

# Unity2020如何从内置管线转到URP、SRP渲染管线
下文均是以Unity2020.1.5版本为准。  
默认读者熟悉shader编写 和渲染管线。

下文是一个项目的快速转换管线的参考。

## **项目安装部分**

**1.安装 URP包，包含了 shader Graph。**然后拖到Quality上

![[2197a58dff1f68e0f4f8461c1013b201_MD5.webp]]

  

2020版本不再独立出 core RP library。

包含在URP内，对HDRP/SRP同理。  

**2.创建URP的Asset**

并创建渲染管线实例。渲染管道实例包含中间资源和渲染管道实现。

  
在编辑器的“项目”窗口中右键单击，然后选择 Create > Rendering > 通用渲染管道 > 管道资产。

**拖到 Graphics上**

![[25a94aa0608c441aa5c3ff27cf769c3d_MD5.webp]]

然后拖到Quality上

![[7b8b153f10a047c45f23dfa715526cf2_MD5.webp]]

这样，我们的URP管线实例的设置就会覆盖它。

![[dcda549f8487fa70e3318c4807c64beb_MD5.webp]]

这是管线实例设置。

里面大部分设置都是以前常用设置。

只说下URP独有的设置项

**General**：

General设置管线渲染每一帧中的核心部分

**1.Depth Texture**

允许URP生成 _CameraDepthTexture 。URP会使用这个深度贴图，作为场景中所有摄像机的深度贴图。但是我们也可以在某个摄像机中重写这个深度贴图

  
**2.不透明贴图**

为场景中的全部摄像机生成一个不透明贴图。不透明贴图在URP渲染透明网格之前提供一个场景快照。使用这个可以在透明Shader中实现毛玻璃、水面反射、热浪等效果。

  
另一个是 Advanced的

SRP Batcher 和 Mixed Lighting

默认打开即可。

最后，Unity2020默认就是linear颜色空间了。

---

## **Shader部分**

  
**1.CG关键字**

凡是 CG关键字 ，都改为HLSL

比如 CGPROGRAM 改完 HLSLPROGRAM 等等

ENDCG 改为 ENDHLSL

  

**2. FallBack**

FallBack "Hidden/Universal Render Pipeline/FallbackError"

  

**3."LightMode"**

"LightMode" 要改为URP支持的模式， 比如

"LightMode" = "Forward"

替换为

"LightMode" = "UniversalForward"

其他常用URP的mode类型：

"LightMode" = "Universal2D"

"LightMode" = "Meta"

"LightMode" = "DepthOnly"

"LightMode" = "ShadowCaster"

  

**4. URP自带shader的Tags**

添加 RenderPipeline关键字了， 我们自己的shader可以不加

例如：

Tags{"RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" "IgnoreProjector" = "True"}

  

**5. include的库**

内置管线库，默认使用cg文件

URP管线，默认使用hlsl文件。

内置管线的shader库全部删除，不能再使用。

需要重新使用URP管线的库。

自己编写shader主要使用

Packages/Core RP Library/ShaderLibrary

和

Packages/Universal RP/ShaderLibrary

这两个库  

一些例子：

内置管线

#include "Lighting.HLSLinc"

替换为：URP管线

#include "LitInput.hlsl"

#include "LitForwardPass.hlsl"

#include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"

  

阴影：

#include "Packages/com.unity.render-pipelines.universal/Shaders/ShadowCasterPass.hlsl"

  

**6. 内置结构体、内置函数 和矩阵**

appdata_full 这种结构体不能用了，如何一一替换我就不举例了：

内置函数 和矩阵需要查阅Packages/Universal RP/ShaderLibrary/UnityInput

内置结构体、函数需要查阅

Packages/Universal RP/ShaderLibrary/Input

Packages/Universal RP/ShaderLibrary/Core

  

**后处理部分**

后处理统一使用 Post Processsing V2

# Unity URP/SRP 渲染管线浅入深出【匠】
[Unity URP/SRP 渲染管线浅入深出【匠】 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/353687806)
## SRP/URP/HDRP之间的关系

下图是各个管线的关系图

![[c864a43d0e93b734eb7d9fcf74d8c5e3_MD5.webp]]

根据上图所示，URP是Unity可编程渲染管线(SRP)的一种，所以了解URP之前需要先了解SRP是什么。(在文章最下面，额外内容里附了张渲染管线流程图

### SRP是什么？

SRP全称为Scriptable Render Pipeline（可编程渲染管线/脚本化渲染管线），是Unity提供的新渲染系统，可以在Unity通过C#脚本调用一系列API配置和执行渲染命令的方式来实现渲染流程，SRP将这些命令传递给Unity底层图形体系结构，然后再将指令发送给图形API。

**说白了就是我们可以用SRP的API来创建自定义的渲染管线，可用来调整渲染流程或修改或增加功能。**

它主要把渲染管线拆分成二层：

-   一层是比较底层的渲染API层，像OpenGL，D3D等相关的都封装起来。
-   另一层是渲染管线上层，上层代码使用C#来编写。在C#这层不需要关注底层在不同平台上渲染API的差别，也不需要关注具体如何做一个Draw Call

### URP是什么？

它的全称为Universal Render Pipeline(通用渲染管线), 它是Unity官方基于SRP提供的模板，它的前身是LWRP(Lightweight RP即轻量级渲染管线), 在2019.3开始改名为URP，它涵盖了范围广泛的不同平台，是针对跨平台开发而构建的，性能比内置管线要好，另外可以进行自定义，实现不同风格的渲染，通用渲染管线未来将成为在Unity中进行渲染的基础 。

**平台范围：可以在Unity当前支持的任何平台上使用

### HDRP是什么？

它的全称为High Definition Render Pipeline（高清晰度渲染管线），它也是Unity官方基于SRP提供的模板，它更多是针对高端设备，如游戏主机和高端台式机，它更关注于真实感图形和渲染，该管线仅于以下平台兼容：

-   Windows和Windows Store，带有DirectX 11或DirectX 12和Shader Model 5.0
-   现代游戏机（Sony PS4和Microsoft Xbox One）
-   使用金属图形的MacOS（最低版本10.13）
-   具有Vulkan的Linux和Windows平台

在此文章对HDRP不过多描述。

## 为什么诞生SRP？

**内置渲染管线的缺陷**

-   **定制性差**：过去，Unity有一套内置渲染管线，渲染管线全部写在引擎的源码里。大家基本上不能改动，除非是买了Unity源码客户，当然大部分开发者是不会去改源码，所以过去的管线对开发者来说，很难进行定制。
-   **代码脓肿，效果效率无法做到最佳：内置渲染管线在一个渲染管线里面支持所有的二十多个平台，包括非常高端的PC平台，也包括非常低端的平台，很老的手机也要支持，所以代码越来越浓肿，很难做到使效率和效果做到最佳。

**目的：**

-   为了解决仅有一个默认渲染管线，造成的可配置型、可发现性、灵活性等问题。决定在C++端保留一个非常小的渲染内核，让C#端可以通过API暴露出更多的选择性，也就是说，Unity会提供一系列的C# API以及内置渲染管线的C#实现；这样一来，一方面可以保证C++端的代码都能严格通过各种白盒测试，另一方面C#端代码就可以在实际项目中调整。

![[0b05321db5f8dff9a1bcdb41efa694be_MD5.webp]]

​在看URP 和 内置渲染管线 性能对比之前最好先了解DrawCall，Batches，SetPassCalls分别是什么值。

DrawCall，Batches，SetPass calls文章：

[https://zhuanlan.zhihu.com/p/353856280](https://zhuanlan.zhihu.com/p/353856280)

## URP 和 内置渲染管线 性能对比

主要提速的有两个方面

1. 光照处理（包括阴影）

2. SRP Bacher (SRP 批处理)（重点）

其他可以看看官网图，下面是官网的对比表链接和图。

[https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/universalrp-builtin-feature-comparison.html](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Packages/com.unity.render-pipelines.universal%408.2/manual/universalrp-builtin-feature-comparison.html)

![[9387b2d0c90a3762ca720b6fbbc05717_MD5.webp]]

1. **首先来说说光照处理部分**

![[14e03deac5e8a1337ad499e706633146_MD5.webp]]

​如上图所示，老的渲染管线使用Multi-Pass的Forward Rendering，就是多Pass的正向渲染。最大的问题是如果要在场景里要加很多动态光的话，每一个动态光都有可能会增加一个Pass，这个动态光所影响的物体要多画一遍。

这就导致如果游戏里想要有多个动态光的话，可能这个场景会被画很多遍，性能会很差。它带来的问题是所有的游戏几乎都不会用多个动态光，因为实在太费性能了。

在过去制作移动的游戏的过程中，大家的标准做法都是烘焙Lightmap。

现在URP就解决了这个问题。实现了一个单PASS的正向渲染。可以支持多盏动态光，但是全部动态灯光都会放在一个Pass里渲染，这样带来的问题是要限制灯光的数量，因为每次Draw Call去画的时候，传给GPU的参数是有限的。

如果灯光数量特别多，参数太多，那就会无法在一次Draw Call里完成很多个灯光。所以我们有一些限制，在轻量级渲染管线LWRP里，目前是支持1盏平行光，每个对象可能只能接受4个动态光。每个摄像机也有一些限制，这是为了我们可以把所有的计算放在一个Pass里面。

**接下来看看内置渲染管线和URP各种情况下的光照处理实验对比**

以下是分别在四种情况下对比所得出的结论

1.  无光源。 **（没区别）**
2.  一个平行光，无阴影。**（没区别）**
3.  一个平行光，一个点光源，无阴影。  
    **结论：**内置渲染管线跟只有一个平行光时比起来Batches将近增加了一倍，而URP的Batches和SetPass calls跟一个平行光时一样，一点都没有增加。
4.  一个动态光，有阴影。  
    **结论：**在阴影的处理方面URP性能比内置渲染管线好很多。  
    

**URP光照处理最终结论：**

> 1. 性能上阴影处理方面比内置渲染管线好很多。  
> 2. URP平行光基础上添加动态光没有带来额外的Batches和SetPass calls性能开销。

**下面有图有真相**

**1. 无光源情况下的对比**

> **内置渲染管线：**

![[0c493d7f210a7948e1a290bb3c8494f7_MD5.webp]]

​  
**URP：**  

![[d95dabbc36b36fe41ce40d45c7e196fd_MD5.webp]]

​  
**结论：**Batches，SetPass Calls 基本是一样的，没区别。

**2. 一个平行光，无阴影下的对比**

> **内置渲染管线：**

![[1673477296b01831866fe30e28003795_MD5.webp]]

​  
**URP：**  

![[bdc191cf2aa3d002b3bd18d8ab2512d3_MD5.webp]]

​结论：Batches，SetPass Calls 基本是一样的，没区别。

**3. 一个平行光，一个点光源，无阴影下的对比**

> **内置渲染管线：**

![[f62ce770d52e74f4c86ba754aa072a82_MD5.webp]]

  
**URP：**  

![[7aa86ef89d7acfe3a14a1993987dfc0f_MD5.webp]]

​  
**结论：**可以发现内置渲染管线的Batches将近增加了一倍，而URP的Batches和SetPass calls跟一个平行光时一样，一点都没有增加。

**4. 一个平行光，有阴影下的对比**

> **内置渲染管线：**

![[fd91225e847f64bfe9be037a2a0213a7_MD5.webp]]

​  
**URP：**  

![[5980a8538958da24359a10f865568bf7_MD5.webp]]

​  
**结论：**内置渲染管线处理阴影面数增加了45k，Batches增加了759，URP面数增加了11K，Batches增加了188。处理阴影性能上URP好很多。

## SRP Batcher（重点）

### **SRP Batcher** 是什么？

> **官网解释：SRP Batcher 是一个底层渲染循环**，可通过许多使用同一着色器变体的材质来加快场景中的 CPU 渲染速度。  
> **个人解释：**就算是不同的材质球，只要是用一个着色器变体的物体都可以批处理到一块。（在2019.3.4版本 渲染顺序也会打断批处理，这点上官网没有说明，也许后续版本已经处理了），它的主要目的是减少渲染状态设置的开销，还有就是把物体属性用专用代码快速更新。  
> **上面解释都提到了变体，那么变体怎么理解呢？**  
> multi_compile和shader_feature这两个关键字就是实现着色器变体的指令。  
> 下面是自定义的multi_compile 关键字。

![[533ea63a86c093c05ce5cd11add6e726_MD5.webp]]

​  
上图中定义了两个**multi_complie**，white和black两个关键字，此时Unity会生成两个变体，一个是走white逻辑的变体，另外一个是走black逻辑的变体。然后在脚本中通过Material.EnableKeyWord和Shader.EnableKeyword来开启某功能，通过Material.DisableKeyword和Shader.DisableKeyword来关闭某功能。  
。  
除了关键字还有这些

![[9e46f402126456bc522930477b6e0bd9_MD5.webp]]

​参数不一致的话也无法批处理到一块。  
shader_feature关键字跟multi_compile不一样的是未被选择的变体会在打包的时候被舍弃（multi_complie不会），shader_feature主要是在材质球选项上控制开关，比如

![[9f429710eb9f460b025eeccdf479b58a_MD5.webp]]

​这个选项。  
URP的Lit Shader里有很多实现变体指令，所以这些Shader生成的变体也有很多。如图：  

![[98b24d75f7c08d8e4f7399225f48d746_MD5.webp]]

​  
主要参考来自以下文章：  
多版本shader的编写-multi_compile和shader_feature：[https://zhuanlan.zhihu.com/p/190233160?utm_source=wechat_session](https://zhuanlan.zhihu.com/p/190233160?utm_source=wechat_session)  
[https://docs.unity.cn/cn/2019.3/Manual/SRPBatcher.html](https://link.zhihu.com/?target=https%3A//docs.unity.cn/cn/2019.3/Manual/SRPBatcher.html)  
[https://connect.unity.com/p/srp-batcher-jia-su-xuan-ran](https://link.zhihu.com/?target=https%3A//connect.unity.com/p/srp-batcher-jia-su-xuan-ran)

过去，Unity 中，可以在一帧内的任何时间修改任何材质的属性。但是，这种做法有一些缺点。当Draw Calls使用新材质时，需要进行很多处理。场景内的材质越多，设置GPU数据所需的CPU资源就越多。解决此问题的传统方法是减少 DrawCall 的数量以优化 CPU 渲染成本，因为 Unity 在发出 DrawCall 之前必须进行很多设置。实际的 CPU 成本来自该设置，而不是来自 GPU DrawCall 本身（DrawCall 只是 Unity 需要推送到 GPU 命令缓冲区的少量字节）。

这是官网说的提速效果：

> Unity 2018引入了可编程渲染管线SRP，其中包含新的底层渲染循环SRP Batcher批处理器，它可以大幅提高CPU在渲染时的处理速度，根据场景内容的不同，提升效果为原来的1.2～4倍不等。

### **SRP Batcher是怎么优化的？**

> SRP Batcher使材质数据持久保留在 GPU 内存中。如果材质内容不变，SRP Batcher 不需要设置缓冲区并将缓冲区上传到 GPU。还有 SRP Batcher 会使用专用的代码路径来快速更新大型 GPU 缓冲区中的 Unity 引擎属性，如下图。  

![[2b2f946829e4b48d9cdd9af9aa054d75_MD5.webp]]

​  
上面的功能能解决什么问题呢？也就是CPU不需要再设置渲染状态和一大堆渲染数据设置，只需要物体跟缓冲区的数据绑定就可以了。  
SRP Batcher 正是通过批处理一系列 `Bind` 和 `Draw` GPU 命令来减少 DrawCall 之间的 GPU 设置。

### 内置渲染管线和URP的CPU原理图对比：

![[a3779c07807f9ee7d7d8fc4355eed3a2_MD5.webp]]

![[01fd4abf539dd58ddbc5aae51af5f508_MD5.webp]]

​  
**内置渲染管线：**（红框部分就是SRP Batcher优化的性能部分）  

![[6e373c35c0aa200678feeed209fa8ea8_MD5.webp]]

​  
  
**URP：**  
在把材质数据和物体数据上传好后的流程图：（GPU没有详细画，主要看CPU）  
  

![[5c4c6f7b3431c87db526f0fd5eba4489_MD5.webp]]

​  
上面流程图中绑定的意思是大家都知道Shader里有很多变量，如纹理贴图，Property定义的变量以及内置变量等，个人理解是把缓冲区里存的渲染数据设置给了Shader变量。  
  
再看看OpenGL API调用情况  
**内置渲染管线**  

![[3a3500a8c89f8f299c446e61058e8fbb_MD5.webp]]

​  
**URP**  

![[2d51196c68d8937f88f7e53e2dad77f7_MD5.webp]]

​  
可以发现内置渲染管线有 glUniform4fv API，这是设置一大堆着色器数据的函数，也是耗时的部分，而在URP取而代之的是Bind接口，省去了设置着色器数据的开销。

  

根据上面内容我们可以知道SRP Batcher并没有减少DrawCall，而是优化了DrawCall之前的设置开销。

  

SRP Batch值我们可以在Frame Debug窗口可以看得到。

  

![[912ecd2c836f04c1e6de2882108d62ae_MD5.webp]]

​

Statistics窗口上的SetPass calls值，其实就是SRP Batch数量 加上 未能批处理的DrawCall数量。

在下图中可以看到SRP Batch批处理了 189个 DrawCalls.

  

![[75aa4f81ecf8d9ef8458a967d2670fa0_MD5.webp]]

​

![[59d272f9849b52822848f286f432d651_MD5.webp]]

### SRP Batcher 兼容性

为了使 SRP Batcher 代码路径能够渲染对象：

-   渲染的对象必须是mesh或者skinned mesh。该对象不能是粒子。
-   着色器必须与 SRP Batcher 兼容。HDRP 和 URP 中的所有光照和无光照着色器均符合此要求（这些着色器的“粒子”版本除外）。

为了使着色器与 SRP Batcher 兼容：

-   必须在一个名为“UnityPerDraw”的 CBUFFER 中声明所有内置引擎属性。例如：`unity_ObjectToWorld` 或 `unity_SHAr`。
-   必须在一个名为 `UnityPerMaterial` 的 CBUFFER 中声明所有材质属性。

  

![[5ac0b7329d2b5c4a35a5f32382725602_MD5.webp]]

​Property定义的属性也是属于PerMaterial.

![[431addf417f3ab74e6e8e67bee45eec3_MD5.webp]]

​可以在 Inspector 面板中查看着色器的兼容性状态。

![[c31c4c83d9cd8dba4730a358b9ce3081_MD5.webp]]

可以在 Inspector 面板中检查特定着色器的兼容性。

  

在看看时间对比

**内置渲染管线（未开启任何合批）**

![[99a6885c9291c3750547f24c3b42c481_MD5.webp]]

**内置渲染管线（开启静态批处理）**

![[b9aab04c4a4759b85a2d39f1911e8209_MD5.webp]]

**URP未开启SRP Batcfh**

![[b2cfd13742aa9d4fd6687d9038e54a6b_MD5.webp]]

​**URP环境下（开启SRP Batcher）**

![[61bd2d403ee77f6f0bdaa445327f31ba_MD5.webp]]

## URP的增加功能

主要有以下功能

1. 渲染管线扩展/自定义渲染器

2. 摄像机堆叠 （待更新）

3. Shader Graphic（Shader图形编程）（待更新）

### 1.渲染管线扩展/自定义渲染器（待补充）

官网案例Github地址：[https://github.com/Unity-Technologies/UniversalRenderingExamples](https://link.zhihu.com/?target=https%3A//github.com/Unity-Technologies/UniversalRenderingExamples)

以下是官网案例效果（最后一个是自己实现）

### 1. 第一人称对象

此Demo主要演示的是摄像机堆叠功能，摄像机堆叠功能下面会单独专门讲。

![[4d2d48db249e22016c69542c80499692_MD5.webp]]

### 2. 卡通描边

展示一种创建卡通样式轮廓效果的设置，示例中有两种方法，一种是后处理方法，另一种是船体网格方法。一个示例使用自定义RendererFeature，并且都使用自定义着色器

![[2ad579140a9b58ab5a830c1e30b7f263_MD5.webp]]

​

![[e3c901cfa1da70b2b423da2de84c61cb_MD5.webp]]

​此Demo有两个渲染器

我们来看看第二个渲染器

![[293d8b6afb30265ffa2995114b8150ab_MD5.webp]]

主要的卡通描边效果就是在上图中的 Sobel Outlines (Blit)实现，

### 3. 对象遮挡

当一个对象被另一个对象遮挡时，可以创建效果。

如下图所示，一个角色被建筑挡住时用指定的材质效果来显示角色被遮挡的部分。

![[381b613d5c315de79d5e956bbad0ec0a_MD5.webp]]

​  
以下是自己实现的扩展功能

### 4. 模糊清晰混合

此效果是在模糊的基础上把某层的物体清晰显示的效果。

![[454249fbcda153c44d5e12f83b38d7de_MD5.webp]]

### ​2. 摄像机堆叠（待更新）

## URP源码解析（待更新）

详解属性：

![[d3c455f99243e78586f9d5f6bc0b37d0_MD5.webp]]

​渲染器列表

HDR（High Dynamic Range Imaging）：高动态范围成像，

Anti Aliasing（MSAA）：抗锯齿

## 参考文章：

Unity轻量级渲染管线LWRP源码及案例解析 - 上：[https://connect.unity.com/p/unityqing-liang-ji-xuan-ran-guan-xian-lwrpyuan-ma-ji-an-li-jie-xi-shang](https://link.zhihu.com/?target=https%3A//connect.unity.com/p/unityqing-liang-ji-xuan-ran-guan-xian-lwrpyuan-ma-ji-an-li-jie-xi-shang)

Unity轻量级渲染管线LWRP源码及案例解析 - 下：[https://connect.unity.com/p/unityqing-liang-ji-xuan-ran-guan-xian-lwrpyuan-ma-ji-an-li-jie-xi-](https://link.zhihu.com/?target=https%3A//connect.unity.com/p/unityqing-liang-ji-xuan-ran-guan-xian-lwrpyuan-ma-ji-an-li-jie-xi-shang)xia

SRP Batcher：加速渲染：[https://connect.unity.com/p/srp-batcher-jia-su-xuan-ran](https://link.zhihu.com/?target=https%3A//connect.unity.com/p/srp-batcher-jia-su-xuan-ran)

URP扩展案例 Github地址： [https://github.com/Unity-Technologies/UniversalRenderingExamples](https://link.zhihu.com/?target=https%3A//github.com/Unity-Technologies/UniversalRenderingExamples)

Unite Now - （中文字幕）使用URP提升游戏画面效果：[https://www.bilibili.com/video/BV1fK4y1a78s](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1fK4y1a78s)

关于静态批处理/动态批处理/GPU Instancing /SRP Batcher的详细剖析：[https://zhuanlan.zhihu.com/p/165574008](https://zhuanlan.zhihu.com/p/165574008)

关于静态批处理/动态批处理/GPU Instancing /SRP Batcher的详细剖析：[https://zhuanlan.zhihu.com/p/98642798](https://zhuanlan.zhihu.com/p/98642798)

Batch, Draw Call, Setpass Call：[https://zhuanlan.zhihu.com/p/76562300](https://zhuanlan.zhihu.com/p/76562300)

Unity3D优化技巧系列（一）：Draw Call优化：[https://gameinstitute.qq.com/community/detail/113025](https://link.zhihu.com/?target=https%3A//gameinstitute.qq.com/community/detail/113025)

Unity ConstantBuffer的一些解析和注意：[https://zhuanlan.zhihu.com/p/137455866](https://zhuanlan.zhihu.com/p/137455866)

## 额外内容

渲染流水线图：

  

![[f812b6bdc830ae8341a52c499b0ab31f_MD5.webp]]