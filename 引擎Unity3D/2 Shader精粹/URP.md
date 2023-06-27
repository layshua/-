
---
title: URP
aliases: []
tags: []
create_time: 2023-06-27 12:20
uid: 202306271220
banner: "![[Pasted image 20230627122009.png]]"
---

# 文件
**内置 Shader 文件路径**：Packages/Universal RP/Shaders

**两个重要路径**：Packages/Core RP Libraries 和 Packages/Universal RP ，这两个文件都不是真正的名字，为了方便查看，Unity 在 UI 上对其进行了处理，真实名称如下：
Packages/Core RP Libraries：`com.unity.render-pipelines.core`
Packages/Universal RP: `com.unity.render-pipelines.universal/ShaderLibrary`


**Core RP Libraries/ShaderLibrary 路径保存了大量用于 shader 计算的库文件**

|文件名称|描述|
|:--|:--|
|Common|定义了新的数据类型 real 和一些通用注的函数|
|CommonLighting|定义了灯光计算的通用函数|
|CommonMaterial|定义了粗糙度的计算函数和一些纹理叠加混合的计算函数|
|EntityLighting|定义了光照贴图采样和环境光解码相关操作的函数|
|ImageBasedLighting|定义了 Skybox 光照相关的函数|
|Macros|包含了很多宏定义|
|Packing|定义了数据解包相关的函数|
|Refraction|定义了折射函数|
|SpaceTransforms|定义了大量空间变换相关的函数|
|Tessellation|定义多种了不同类型的曲面细分函数|

**Universal RP/ShaderLibrary 路径保存了 URP 内置的 Shader 所关联的包含文件**


|文件名称|描述|
|:--|:--|
|Core|URP 的核心文件, 包含了大量顶点数据、获取数据的函数等|
|Input|定义了 InputData 结构体、常量数据和空间变换矩阵的宏定义|
|Lighting|定义了光照计算相关的函数, 包括全局照明、多种光照模型等|
|Shadows|定义了计算阴影相关的函数|
|SurfaceInput|定义 SurfaceData 结构体和几种纹理的采样函数|
|UnityInput|包含了大量可以直接使用的全局变量和变换矩阵|





