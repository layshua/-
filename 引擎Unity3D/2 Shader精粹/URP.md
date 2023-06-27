
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

## Core RP Library
**Core RP Libraries/ShaderLibrary 路径保存了大量用于 shader 计算的库文件**

|文件名称|描述|
|:--|:--|
|Common|定义了新的数据类型 real 和一些通用注的函数|
|CommonLighting|定义了灯光计算的通用函数|
|CommonMaterial|定义了粗糙度的计算函数和一些纹理叠加混合的计算函数|
|EntityLighting|定义了光照贴图采样和环境光解码相关操作的函数 |
|ImageBasedLighting|定义了 Skybox 光照相关的函数 |
|Macros|包含了很多宏定义|
|Packing|定义了数据解包相关的函数|
|Refraction|定义了折射函数 |
|SpaceTransforms|定义了大量空间变换相关的函数 |
|Tessellation|定义多种了不同类型的曲面细分函数|

## **Universal RP
**Universal RP/ShaderLibrary 路径保存了 URP 内置的 Shader 所关联的包含文件**

|文件名称|描述|
|:--|:--|
|Core|URP 的核心文件, 包含了大量顶点数据、获取数据的函数等|
|Input|定义了 InputData 结构体、常量数据和空间变换矩阵的宏定义 |
|Lighting|定义了光照计算相关的函数, 包括全局照明、多种光照模型等 |
|Shadows|定义了计算阴影相关的函数 |
|SurfaceInput |定义 SurfaceData 结构体和几种纹理的采样函数|
|UnityInput |包含了大量可以直接使用的全局变量和变换矩阵|

## 文件包含关系
![[Pasted image 20230627140250.png]]

1. SurfaceInput 包含了 Core、Packing、CommonMaterial
2. Core 包含了 Common、Packing、Input
3. Input 包含了 UnityInput、SpaceTransforms
4. Lighting 包含了 Core、Common、EntityLighting、 ImageBasedLighting、Shadow
5. ImageBasedLighting 包含了 CommonLighting、CommonMaterial
6. Shadow 包含了 Core 

# Lit. shader 解析

> [!NOTE] 版本
> Unity2022.3.0f1c1：URP14.0

![[Litshader.km]]
## SubShader
Lit. shader 只有一个 SubShader，SubShader 的 Tags 如下：

```cs
Tags  
{  
    "RenderType" = "Opaque"  
    "RenderPipeline" = "UniversalPipeline"  
    "UniversalMaterialType" = "Lit"  
    "IgnoreProjector" = "True"  
}
```

SubShader 需要添加 `"RenderPipeline" = "UniversalPipeline"` 标签，告诉 Unity 当前 SubShader 需要在 URP 运行。
>如果想要一个 Shader 既可以在 URP 也可以在 Builtin 运行，可以加一个 SubShader，或者通过 FallBack 指令回到适配的 Shader。

```cs
FallBack "Hidden/Universal Render Pipeline/FallbackError"  //显示错误紫色
CustomEditor "UnityEditor.Rendering.Universal.ShaderGUI.LitShader" //ShaderGUI
```


## Pass

### ForwardLit
前向渲染 Pass，在一个 Pass 中计算所有光照，包括全局光照 GI，自发光 Emission，雾效 Fog。
将该 Pass 拆解，解析如下：
```cs
Pass
{
    //Lightmode 标签 与 UniversalRenderPipeline.cs 中设置的 ShaderPassName 匹配
    //在SRP创建的Unlit Shader（SRPDefaultUnlit） 即便Pass中没有 Lightmode 标签也可以在URP中正常渲染。即Unlit shader可以不加 Lightmode 标签 
    Name "ForwardLit"
    Tags
    {
        "LightMode" = "UniversalForward"
    }

    // -------------------------------------
    // Render State Commands
    Blend[_SrcBlend][_DstBlend], [_SrcBlendAlpha][_DstBlendAlpha]
    ZWrite[_ZWrite]
    Cull[_Cull]
    AlphaToMask[_AlphaToMask]

    HLSLPROGRAM
    #pragma target 2.0

    // -------------------------------------
    // Shader Stages
    #pragma vertex LitPassVertex
    #pragma fragment LitPassFragment

    // -------------------------------------
    //材质属性关键词
    //通用管线关键词
    //Unity定义的关键词
    // -------------------------------------

    // -------------------------------------
    // GPU Instancing
    #pragma multi_compile_instancing
    #pragma instancing_options renderinglayer
    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"
    
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitForwardPass.hlsl"
    ENDHLSL
}

```

#### 材质属性关键词
![[Pasted image 20230627155115.png|450]]

```cs
// 材质属性关键词 Material Keywords

//对应材质inspector面板中的设置
//例如：添加NormalMap就会传入_NORMALMAP关键词
#pragma shader_feature_local _NORMALMAP 
#pragma shader_feature_local _PARALLAXMAP 
#pragma shader_feature_local _RECEIVE_SHADOWS_OFF 
#pragma shader_feature_local _ _DETAIL_MULX2 _DETAIL_SCALED
#pragma shader_feature_local_fragment _SURFACE_TYPE_TRANSPARENT
#pragma shader_feature_local_fragment _ALPHATEST_ON
#pragma shader_feature_local_fragment _ _ALPHAPREMULTIPLY_ON _ALPHAMODULATE_ON
#pragma shader_feature_local_fragment _EMISSION
#pragma shader_feature_local_fragment _METALLICSPECGLOSSMAP
#pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A
#pragma shader_feature_local_fragment _OCCLUSIONMAP
#pragma shader_feature_local_fragment _SPECULARHIGHLIGHTS_OFF
#pragma shader_feature_local_fragment _ENVIRONMENTREFLECTIONS_OFF
#pragma shader_feature_local_fragment _SPECULAR_SETUP
```

#### 通用管线关键词
```cs

// 通用管线关键词 Universal Pipeline keywords
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MAIN_LIGHT_SHADOWS_SCREEN
#pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
#pragma multi_compile _ EVALUATE_SH_MIXED EVALUATE_SH_VERTEX
#pragma multi_compile_fragment _ _ADDITIONAL_LIGHT_SHADOWS
#pragma multi_compile_fragment _ _REFLECTION_PROBE_BLENDING
#pragma multi_compile_fragment _ _REFLECTION_PROBE_BOX_PROJECTION
#pragma multi_compile_fragment _ _SHADOWS_SOFT
#pragma multi_compile_fragment _ _SCREEN_SPACE_OCCLUSION
#pragma multi_compile_fragment _ _DBUFFER_MRT1 _DBUFFER_MRT2 _DBUFFER_MRT3
#pragma multi_compile_fragment _ _LIGHT_LAYERS
#pragma multi_compile_fragment _ _LIGHT_COOKIES
#pragma multi_compile _ _FORWARD_PLUS
#include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/RenderingLayers.hlsl"
```
#### Unity 定义的关键词
```cs
// Unity定义的关键词 Unity defined keywords
#pragma multi_compile _ LIGHTMAP_SHADOW_MIXING
#pragma multi_compile _ SHADOWS_SHADOWMASK
#pragma multi_compile _ DIRLIGHTMAP_COMBINED
#pragma multi_compile _ LIGHTMAP_ON
#pragma multi_compile _ DYNAMICLIGHTMAP_ON
#pragma multi_compile_fragment _ LOD_FADE_CROSSFADE
#pragma multi_compile_fog
#pragma multi_compile_fragment _ DEBUG_DISPLAY
```

