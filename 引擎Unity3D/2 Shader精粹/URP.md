
---
title: URP
aliases: []
tags: []
create_time: 2023-06-27 12:20
uid: 202306271220
banner: "![[Pasted image 20230627122009.png]]"
---

# 规范
## 坐标系

![image-20220702145045963](image-20220702145045963.png)
>注意观察空间是右手坐标系

- 模型和世界空间中，$Y$ 轴朝上，前向是 $Z$ 轴
- 观察空间中，摄像机的前向为 $-Z$ 轴，深度越大，Z 轴坐标越小
- 对于反射探针获取的 cubemap，查看空间依然使用左手坐标系

光照模型默认按照 1 单位为 1 米进行运算，因此为了实现更逼真的渲染效果，要确保模型的比例尺寸是正确的。
## 变量命名规范
1. 变量名称后面的缩写表示的是所在空间名称

|缩写 |说明|
|:---|:--|
|WS|World space 世界空间 |
|RWS|Camera-Relative World Space 相对于摄像机的世界空间，在这个空间中，为了提高摄像机的精度，会将摄像机的平移减去|
|VS |View Space 观察空间|
|OS |Object Space 对象空间/局部空间/模型空间|
|CS|HomogeneousClip Space 齐次裁剪空间|
|TS|Tangent Space 切线空间|
|TXS|Texture Space 纹理空间|

1. 标准化/为标准化（$normalized$ 和 $unormalized$）：所有可以直接使用的向量都已经标准化处理了，除非使用 $un$ 标记的向量，如 `unL` 表示未标准化的灯光向量
2. 常用的向量用大写字母表示，向量总是从像素位置指向外边，并且可以直接用于光照计算。大写字母表示向量是规范化的，除非我们在它前面加上`un`。
     -  V：观察方向
     -  L：灯光方向
     - N：法线向量
     - H：半角向量
3. 顶点函数的输入输出结构体使用帕斯卡命名，并以输入类型为前缀
    - struct AttributesDefault
    - struct VaryingsDefault

//使用这些结构时使用input/output作为变量名
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

```c
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

```c
FallBack "Hidden/Universal Render Pipeline/FallbackError"  //显示错误紫色
CustomEditor "UnityEditor.Rendering.Universal.ShaderGUI.LitShader" //ShaderGUI
```


## Pass

### ForwardLit
前向渲染 Pass，在一个 Pass 中计算所有光照，包括全局光照 GI，自发光 Emission，雾效 Fog。

```c
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
    
    // -------------------------------------
    // 通用管线关键词 Universal Pipeline keywords
    // 对应Universal Render Pipeline Asset和Lighting中的设置
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
    
    // -------------------------------------
    // Unity定义的关键词 Unity defined keywords
    // 对应Lighting中的设置
    #pragma multi_compile _ LIGHTMAP_SHADOW_MIXING
    #pragma multi_compile _ SHADOWS_SHADOWMASK
    #pragma multi_compile _ DIRLIGHTMAP_COMBINED
    #pragma multi_compile _ LIGHTMAP_ON
    #pragma multi_compile _ DYNAMICLIGHTMAP_ON
    #pragma multi_compile_fragment _ LOD_FADE_CROSSFADE
    #pragma multi_compile_fog
    #pragma multi_compile_fragment _ DEBUG_DISPLAY
    
    // -------------------------------------
    // GPU Instancing
    #pragma multi_compile_instancing
    #pragma instancing_options renderinglayer
    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"

    // -------------------------------------
    // 变量声明、顶点函数和片元函数都在包含文件中
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitForwardPass.hlsl"
    ENDHLSL
}

```

#### LitForwardPass.hlsl
```c
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

struct Attributes
{
    float4 positionOS   : POSITION;
    float3 normalOS     : NORMAL;
    float4 tangentOS    : TANGENT;
    float2 texcoord     : TEXCOORD0;
    float2 staticLightmapUV   : TEXCOORD1; //静态光照贴图
    float2 dynamicLightmapUV  : TEXCOORD2; //动态光照贴图
    UNITY_VERTEX_INPUT_INSTANCE_ID //GPU Instancing获取实例ID
};
```

```c
struct Varyings
{
    float2 uv                       : TEXCOORD0;
    float3 positionWS               : TEXCOORD1;
    float3 normalWS                 : TEXCOORD2;
    half4 tangentWS                : TEXCOORD3;    // xyz:切线分量, w: 切线方向
    half4 fogFactorAndVertexLight   : TEXCOORD5; // 雾系数和顶点光照，x: fogFactor, yzw: vertex light
    half  fogFactor                 : TEXCOORD5;
    float4 shadowCoord              : TEXCOORD6;
    half3 viewDirTS                : TEXCOORD7;
    DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 8);//光照贴图纹理坐标，光照贴图名称，球谐光照名称，纹理坐标索引
    float2  dynamicLightmapUV : TEXCOORD9; // Dynamic lightmap UVs

    float4 positionCS               : SV_POSITION; //齐次裁剪空间顶点坐标
    
    UNITY_VERTEX_INPUT_INSTANCE_ID //GPU Instancing获取实例ID
    UNITY_VERTEX_OUTPUT_STEREO //用于VR平台的宏定义
};
```
# 语法

## 数据类型
real 类型：参数同时支持 float 和 half 类型时使用
## 纹理和采样器
```c
//声明纹理和采样器
TEXTURE2D(_textureName); SAMPLER(sampler_textureName);

//如果需要使用Tilling和Scale功能，需要在CBuffer中声明float4类型的变量
float4 _textureName_ST;

//进行纹理采样
SAMPLE_TEXTURE2D(_textureName, sampler_textureName, uv)
```

采样器可以定义纹理设置面板上的 Wrap Mode（重复模式，clamp、repeat、mirror、mirrorOnce）和 Filter Mode （过滤模式，point、linear、triLinear）

采样器的三种定义方式：
1. `SAMPLER(sampler_textureName)`：表示该纹理使用设置面板设定的采样方式
2. `SAMPLER(filer_wrap)`：使用自定义的采样器设置，必须同时包含 Wrap Mode 和 Filter Mode 的设置，如 `SAMPLER(point_clamp) `
3. `SAMPLER(filer_wrapU_wrapV)` ：可以同时为 U 和 V 设置不同的 Wrap Mode 如：`SAMPLER(filer_clampU_mirrorV)`

### 宏定义
`TEXTURE2D_ARGS()` 宏：只是将纹理名称和采样器名称这两个参数合并在一起
```c file:DX11下的定义
#define TEXTURE2D_ARGS(textureName, samplerName) textureName, samplerName
```

`TEXTURE2D_PARAM()` 宏，传入纹理名称和采样器名称，转换为一个纹理变量和一个采样器，可以作为参数，节省字数。
```c file:DX11下的定义
#define TEXTURE2D_PARAM(textureName, samplerName)              TEXTURE2D(textureName),         SAMPLER(samplerName)
```

```c file:宏定义用法
//采样函数，TEXTURE2D_PARAM()接收参数
half4 SampleAlbedoAlpha(float2 uv, TEXTURE2D_PARAM(albedoAlphaMap, sampler_albedoAlphaMap))
{
    return SAMPLE_TEXTURE2D(albedoAlphaMap, sampler_albedoAlphaMap, uv);
}

//使用采样函数，TEXTURE2D_ARGS()很方便的将两个参数传入
float albedoAlpha = SampleAlbedoAlpha(uv, TEXTURE2D_ARGS(_BaseMap,sampler_BaseMap));


//等价实现：
half4 SampleAlbedoAlpha(float2 uv, TEXTURE2D(_textureName),  SAMPLER(sampler_textureName))
{
    return SAMPLE_TEXTURE2D(albedoAlphaMap, sampler_albedoAlphaMap, uv);
}

float albedoAlpha = SampleAlbedoAlpha(uv ,_BaseMap ,sampler_BaseMap);
```


>图形API宏定义路径：Packages/Core RP Library/ShaderLibrary/API
### 法线贴图采样
`real3 UnpackNormal(real4 packedNormal)`
`real3 UnpackNormalScale(real4 packedNormal, real bumpScale)`

```c
half4 n = SAMPLE_TEXTURE2D(_textureName, sampler_textureName, uv)
# if BUMP_SCALE_NOT_supported //如果平台不支持调节法线强度
    return UnpackNormal(n)
#else 
    return UnpackNormalScale(n,scale);
```

`UnpackNormal()` 和 `UnpackNormalScale`：当平台不使用 DXT5nm 压缩法线贴图（移动平台），函数内部会分别调用 `UnpackNormalRGBNoScale()` 和 `UnpackNormalRGB()` ；佛则，这两个函数内部都调用 `UnpackNormalmapRGorAG()` 函数，只不过 `UnpackNormal()` 法线强度始终为 1。




