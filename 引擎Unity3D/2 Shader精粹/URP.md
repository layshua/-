
---
title: URP
aliases: []
tags: []
create_time: 2023-06-27 12:20
uid: 202306271220
banner: "![[Pasted image 20230627122009.png]]"
---


# 规范
## 总体结构

1. 在 SubShader 的 Tags 中添加 `"RenderPipeline" = "UniversalPipeline"` 
2. 所有 URP 着色器都是 HLSL 编写的，使用宏 `HLSL` 包含的 shader 代码 
3. 使用 HLSLINCLUDE 替代 CGINCLUDE

| 内置管线                  | URP         |
|-----------------------|-------------|
| CGPROGRAM HLSLPROGRAM | HLSLPROGRAM |
| ENDCG ENDHLSL         | ENDHLSL     |
| CGINCLUDE HLSLINCLUDE | HLSLINCLUDE |

## 坐标系

![[image-20220702145045963.png]]
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
|OS |Object Space 对象空间/局部空间/模型空间 |
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
    - struct AttributesDefault 顶点函数输入结构体
    - struct VaryingsDefault 顶点函数输出结构体
    - 调用结构体时使用 input/output 作为变量名
4. 顶点函数和片元函数名称：VertDefault、FragDefault/FragForward/FragDeferred
5. 浮点常数写作 1.0，不能是 1，1.0f，1.0h
6. 函数如果需要显式地定义精度，可以使用 float 或者 half 修饰，当这两种精度都支持的时候，则使用 real 修饰。
7. 如果一个函数同时会用到 half 和 float 版本，那么这两个版本都需要显式定义。
8. 禁止使用 in，只能使用 out 和 inout 作为函数参数的修饰词，也不要使用 inline，该修饰词无效。函数的 out 参数要放在最后
9. uniform 变量使用下划线作为前缀，后面的首字母大写，例如：`_MainTex`。
10. **所有的 uniform 变量都应该放在常量缓冲区**。这是因为要保证 Compute Shader 的常数缓冲区布局在内核中保持一致，有时候这是全局命名空间无法控制的（uniform 在不被使用的时候会得到优化，因此每核都会调整全局常数缓冲区的布局）
11. ShaderLibray 中的函数是无状态的，声明时不需要使用 uniform
12. ShaderLibrary 的头文件不包含"common.hlsl"，这应该包含在".shader" 中使用它(或 "Material.hlsl")。
13. cs 和 hlsl 之间共享的结构体定义需要在 float4上对齐，以遵守着色器语言中的各种打包规则。这意味着这些结构体需要填充。规则如下：当为常量缓冲区变量执行数组时，我们总是使用 float4来避免任何打包问题，特别是在计算着色器和像素着色器之间。即不要使用 `SetGlobalFloatArray` 或 `SetComputeFloatParams`
14. 数组可以是hlsl中的别名。示例：
```cs
//hlsl
uniform float4 packedArray[3]；

//cs
static float unpackedArray[12] =（float[12]）packedArray；
```

# 文件
**内置 Shader 文件路径**：Packages/Universal RP/Shaders

**两个重要路径**：Packages/Core RP Libraries 和 Packages/Universal RP ，这两个文件都不是真正的名字，为了方便查看，Unity 在 UI 上对其进行了处理，真实名称如下：
Packages/Core RP Libraries：`com.unity.render-pipelines.core`
Packages/Universal RP: `com.unity.render-pipelines.universal/ShaderLibrary`


| 内容                                                                                        | 内置管线            | URP                                                                       |
|-------------------------------------------------------------------------------------------|-----------------|---------------------------------------------------------------------------|
| Core                                                                                      | Unity.cginc     | Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl     |
| Light                                                                                     | AutoLight.cginc | Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl |
| Shadow                                                                                    | AutoLight.cginc | Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl  |
| 表面着色器                                                                                     | Lighting.cginc  | URP内没有，可以参考项目：在此处                                                         |

其他有用的包括：
- [Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl)
- [Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl)
- [Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl)
- [Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl)
- [Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl)
- [Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl)
- [Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareOpaqueTextue.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/DeclareOpaqueTexture.hlsl)


## Core RP Library
**Core RP Libraries/ShaderLibrary 路径保存了大量用于 shader 计算的库文件**

|文件名称|描述|
|:--|:--|
|Common|定义了新的数据类型 real 和一些通用的函数|
|CommonLighting|定义了灯光计算的通用函数|
|CommonMaterial|定义了粗糙度的计算函数和一些纹理叠加混合的计算函数|
|EntityLighting|定义了光照贴图采样和环境光解码相关操作的函数 |
|ImageBasedLighting|定义了 Skybox 光照相关的函数 |
|Macros|包含了很多宏定义|
|Packing|定义了数据解包相关的函数|
|Refraction|定义了折射函数 |
|SpaceTransforms|定义了大量空间变换相关的函数 |
|Tessellation|定义多种了不同类型的曲面细分函数|
### Common

|函数|说明|
|:--|:--|
|real DegToRad(real deg) |角度转弧度|
|real RadToDeg(real rad)|弧度转角度|
|float Length2(float3 v)|返回向量长度的平方：dot(v,v)|
|real3 SafeNormalize(float3 inVec)|返回标准化向量，与Normalize()不同的是，本函数会兼容长度为0的向量|
|real SafeDiv(real numer, real denom)|返回相处之后的商，不同于直接进行除法运算，当两个数值同时为无穷大或者0时，函数返回1|

```cs file:SafeNormalize
//检查了一下向量的长度的平方，是否为0，如果小于FLT_MIN则取FLT_MIN,即最小的正32位浮点数。`rsqrt`这个hlsl函数计算平方根的倒数，和原向量相乘就做了归一化。这个函数可以避免向量长度过小引起的除0错误。
real3 SafeNormalize(float3 inVec)
{
    real dp3 = max(FLT_MIN, dot(inVec, inVec));
    return inVec * rsqrt(dp3);
}
```
## Universal RP
**Universal RP/ShaderLibrary 路径保存了 URP 内置的 Shader 所关联的包含文件**

|文件名称|描述|
|:--|:--|
|Core|URP 的核心文件, 包含了大量顶点数据、获取数据的函数等|
|Input |定义了 InputData 结构体、常量数据和空间变换矩阵的宏定义 |
|Lighting |定义了光照计算相关的函数, 包括全局照明、多种光照模型等 |
|Shadows|定义了计算阴影相关的函数 |
|SurfaceInput |定义 SurfaceData 结构体和几种纹理的采样函数|
|UnityInput |包含了大量可以直接使用的全局变量和变换矩阵|

### Input

结构体：
```cs
struct InputData
{
    float3  positionWS;
    float4  positionCS;
    half3   normalWS;
    half3   viewDirectionWS;
    float4  shadowCoord;             //阴影坐标
    half    fogCoord;                //雾坐标
    half3   vertexLighting;          //顶点光照
    half3   bakedGI;                 //全局光照
    float2  normalizedScreenSpaceUV; //标准化屏幕空间UV
    half4   shadowMask;              //阴影遮罩
    half3x3 tangentToWorld;          //切线空间到世界空间变换矩阵
};
```

变量

|变量名称|说明|
|:--|:--|
|`half4 _GlossyEnvironmentColor`|环境的反射颜色|
|`half4 _SubtractiveShadowColor`|阴影颜色|
|`float4 _MainLightPosition`|主光源位置|
|`half4 _MainLightColor`|主光源颜色|

变换矩阵宏定义
```c file:变换矩阵宏定义
#ifndef BUILTIN_TARGET_API
#define UNITY_MATRIX_M     unity_ObjectToWorld
#define UNITY_MATRIX_I_M   unity_WorldToObject
#define UNITY_MATRIX_V     unity_MatrixV
#define UNITY_MATRIX_I_V   unity_MatrixInvV
#define UNITY_MATRIX_P     OptimizeProjectionMatrix(glstate_matrix_projection)
#define UNITY_MATRIX_I_P   (float4x4)0
#define UNITY_MATRIX_VP    unity_MatrixVP
#define UNITY_MATRIX_I_VP  (float4x4)0
#define UNITY_MATRIX_MV    mul(UNITY_MATRIX_V, UNITY_MATRIX_M)
#define UNITY_MATRIX_T_MV  transpose(UNITY_MATRIX_MV)
#define UNITY_MATRIX_IT_MV transpose(mul(UNITY_MATRIX_I_M, UNITY_MATRIX_I_V))
#define UNITY_MATRIX_MVP   mul(UNITY_MATRIX_VP, UNITY_MATRIX_M)
#define UNITY_PREV_MATRIX_M   unity_MatrixPreviousM
#define UNITY_PREV_MATRIX_I_M unity_MatrixPreviousMI
#else
```
### Lighting

|函数|说明|
|:--|:--|
|Light **GetMainLight**()|获取主光源|
|int **GetAdditionalLightsCount**();|获取其他光源数量|
|Light **GetAdditionalLight**(int index, float3 WS_Pos)|获取其他的光源|
|half3 **SampleSH**(half3 normalWS)|环境光函数|
|half3  `_GlossyEnvironmentColor`|Unity 内置环境光 |
|half3 **LightingLambert**|Lambert  |
|half3 **LightingSpecular**|BlinnPhong|
|  |  |

```cs
struct Light
{
    half3   direction;
    half3   color; //主光源颜色*强度
    float   distanceAttenuation;  //距离衰减
    half    shadowAttenuation;    //阴影衰减
    uint    layerMask;
};
```

*   采样光照贴图不支持实时 GI

```c
half3 SampleLightmap(float2 lightmapUV, half3 normalWS);
half3 SubtractDirectMainLightFromLightmap(Light mainLight, half3 normalWS, half3 bakedGI);
```

*   混合实时光和 bakeGI

```c
void MixRealtimeAndBakedGI(inout Light light, half3 normalWS, inout half3 bakedGI)
```

### Shadows
表明该 Pass 选择阴影渲染模式

```c
Tags { "LightMode" = "ShadowCaster" }
```

- 用于获取应用阴影的深度偏移后的阴影坐标

```c
ApplyShadowBias(positionWS, normalWS, _LightDirection)
```

- 获取阴影坐标

```c
TransformWorldToShadowCoord(positionWS)
```

- 用于接受阴影用的关键字

```c
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE
#pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
#pragma multi_compile _ _SHADOWS_SOFT
```

示例

```c
Pass         
{             
	Name "ShadowCaster" 
	Tags { "LightMode" = "ShadowCaster" } 
	Cull Off 
	ZWrite On 
	ZTest LEqual 
	                    
	HLSLPROGRAM         
 #pragma vertex vert 
 #pragma fragment frag 
 #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"
	
	float3 _LightDirection; 
	//为了让shader的SRP Batcher能够使用，所以每个pass的Cbuffer都要保持一直。（应该是这样吧）
	
	CBUFFER_START(UnityPerMaterial)                 
	//...             
	CBUFFER_END         
	    
	struct Attributes 
	{                 
		float4 positionOS: POSITION; 
		float3 normalOS: NORMAL; 
		float2 texcoord: TEXCOORD0; 
	};             
	
	struct Varyings 
	{                 
		float2 uv: TEXCOORD0; 
		float4 positionCS: SV_POSITION; 
	};             
	
	// 获取裁剪空间下的阴影坐标 
	float4 GetShadowPositionHClips(Attributes input) 
	{                 
		float3 positionWS = TransformObjectToWorld(input.positionOS.xyz); 
		float3 normalWS = TransformObjectToWorldNormal(input.normalOS); 
		//_LightDirection这个是官方自己定义的特定的float3，它可以获得主光源和额外光源的方向，替换掉MainLight.direction后，就可以正常获取addlight的灯光方向了                 
		float4 positionCS = TransformWorldToHClip(ApplyShadowBias(positionWS, normalWS, _LightDirection)); 
		return positionCS; 
	}             
	
	Varyings vert(Attributes input) 
	{                 
		Varyings output; 
		output.uv = TRANSFORM_TEX(input.texcoord, _MainTex); 
		output.positionCS = GetShadowPositionHClips(input); 
		return output; 
	}             
	
	half4 frag(Varyings input): SV_TARGET 
	{                 
		half4 albedoAlpha = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, input.uv); 
		//这里要是否需要裁剪透明通道。现在BASE是Opaque渲染模式，没有裁剪。需要用的自己改一下。                 //clip(albedoAlpha.a - _Cutoff); 
		return 0; 
	}           
	ENDHLSL
}
```

*   或者使用 UsePass 调用内置的 ShadowCaster

```c
UsePass "Universal Render Pipeline/Lit/ShadowCaster"
```

URP 只支持使用单个 Pass 渲染如果要开启多 Pass 需要按以下方式
*   第二个 Pass 添加

```c
Tags{ "LightMode" = "SRPDefaultUnlit" }
```

*   即可让这两个 pass 同时生效
### SpaceTransforms

|获取空间变换矩阵|说明|
|:--|:--|
|float4x4 **GetObjectToWorldMatrix()** |模型空间到世界空间，反之|
|float4x4 **GetViewToWorldMatrix()**|观察空间到世界空间，反之|
|float4x4 **GetWorldToHClipMatrix()**|世界空间到齐次裁剪空间|
|float4x4 **GetViewToHClipMatrix()**|观察空间到齐次裁剪空间|


|顶点变换函数|说明|
|:--|:--|
|float3 **TransformObjectToWorld(float3 positionOS)**|模型空间到世界空间，反之|
|float3 **TransformViewToWorld(float3 positionVS)** |观察空间到世界空间，反之|
|float4 **TransformObjectToHClip(float3 positionOS)** |模型空间到齐次裁剪空间 |
|float4 **TransformWViewToHClip(float3 positionVS)**|观察空间到齐次裁剪空间|
|float4 **TransformWorldToHClip(float3 positionWS)** |世界空间到齐次裁剪空间|

|向量变换函数|说明 |
|:--|:--|
|float3 **TransformObjectToWorldDir**(float3 dirOS, bool doNormalize = true)|模型到世界，是否将向量标准化。反之 |
|real3 **TransformViewToWorldDir**(real3 dirVS, bool doNormalize = false)|观察到世界，是否将向量标准化。反之 |
|real3 **TransformWorldToHClipDir**(real3 directionWS, bool doNormalize = false)|世界到齐次裁剪，是否将向量标准化。反之|
|float3 **TransformObjectToWorldNormal**(float3 normalOS, bool doNormalize = true)|（用于法线）模型到世界，是否将向量标准化，反之 |
|real3 **TransformViewToWorldNormal**(real3 normalVS, bool doNormalize = false)|（用于法线）观察到世界，是否将向量标准化，反之|
|real3x3 **CreateTangentToWorld**(real3 normal, real3 tangent, real flipSign)|（用于法线）传入法线，切线，方向符号，返回法线从切线空间到世界空间的3x3变换矩阵tangentToWorld |
|real3 **TransformTangentToWorld**(float3 normalTS, real3x3 tangentToWorld, bool doNormalize = false)|（用于法线）用tangentToWorld矩阵，切线到世界，反之|
|real3 **TransformTangentToWorldDir**(real3 dirWS, real3x3 tangentToWorld, bool doNormalize = false)|（用于向量）切线到世界，是否将向量标准化，反之|
|real3 **TransformObjectToTangent**(real3 dirOS, real3x3 tangentToWorld) |（用于向量）模型到世界，是否将向量标准化，反之|



## 文件包含关系
![[Pasted image 20230627140250.png]]

1. SurfaceInput 包含了 Core、Packing、CommonMaterial
2. Core 包含了 Common、Packing、Input
3. Input 包含了 UnityInput、SpaceTransforms
4. Lighting 包含了 Core、Common、EntityLighting、 ImageBasedLighting、Shadow
5. ImageBasedLighting 包含了 CommonLighting、CommonMaterial
6. Shadow 包含了 Core 

# 着色器宏
## 辅助宏

|内置管线|URP|
|---|---|
|**UNITY_PROJ_COORD**(_a_)|移除了，使用**a.xy / a.w**代替|
|**UNITY_INITIALIZE_OUTPUT**(_type_，_name_) |**ZERO_INITIALIZE**(_type_，_name_)|

## 纹理和采样器
Unity 有很多纹理/采样器宏来改善 API 之间的交叉兼容性，但是人们并不习惯使用它们。URP 中这些宏的名称有所变化。由于数量很多，全部的宏可以在 [API includes中](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/API)查看，下面主要列举一些常用的：

| 内置管线                                            | URP                                                                  |
|-------------------------------------------------|----------------------------------------------------------------------|
| UNITY_DECLARE_TEX2D（name）                       | TEXTURE2D（textureName）; SAMPLER（samplerName）;                        |
| UNITY_DECLARE_TEX2D_NOSAMPLER（name）             | TEXTURE2D（textureName）;                                              |
| UNITY_DECLARE_TEX2DARRAY（name）                  | TEXTURE2D_ARRAY（textureName）; SAMPLER（samplerName）;                  |
| UNITY_SAMPLE_TEX2D（name，uv）                     | SAMPLE_TEXTURE2D（textureName，samplerName，coord2）                     |
| UNITY_SAMPLE_TEX2D_SAMPLER（name，samplername，uv） | SAMPLE_TEXTURE2D（textureName，samplerName，coord2）                     |
| UNITY_SAMPLE_TEX2DARRAY（name，uv）                | SAMPLE_TEXTURE2D_ARRAY（textureName，samplerName，coord2，index）         |
| UNITY_SAMPLE_TEX2DARRAY_LOD（name，uv，lod）        | SAMPLE_TEXTURE2D_ARRAY_LOD（textureName，samplerName，coord2，index，lod） |

需要注意 `SCREENSPACE_TEXTURE` 变成了 `TEXTURE2D_X`。如果你想要在 VR 中（_Single Pass Instanced_ 或 _Multi-view_ 模式）制作屏幕效果，你必须使用 `TEXTURE2D_X` 定义纹理。这个宏会为你处理正确的纹理声明（是否为数组）。对这个纹理采样的时候必须使用 `SAMPLE_TEXTURE2D_X`，并且对 uv 使用 `UnityStereoTransformScreenSpaceTex`。

### 常用
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
### 法线贴图
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


### 阴影贴图
必须 include [“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl”](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl)

| 内置管线                             | URP                                                                  |
|----------------------------------|----------------------------------------------------------------------|
| UNITY_DECLARE_SHADOWMAP（tex）     | TEXTURE2D_SHADOW_PARAM（textureName，samplerName）                      |
| UNITY_SAMPLE_SHADOW（tex，uv）      | SAMPLE_TEXTURE2D_SHADOW（textureName，samplerName，coord3）              |
| UNITY_SAMPLE_SHADOW_PROJ（tex，uv） | SAMPLE_TEXTURE2D_SHADOW（textureName，samplerName，coord4.xyz/coord4.w） |


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

```c file:ForwardLit
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
##### 顶点输入输出结构体
```c file:顶点输入结构体
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

```c file:顶点输出结构体
struct Varyings
{
    float2 uv                       : TEXCOORD0;
    float3 positionWS               : TEXCOORD1;
    float3 normalWS                 : TEXCOORD2;
    half4  tangentWS                : TEXCOORD3;  // xyz:切线分量, w: 切线方向
    half4  fogFactorAndVertexLight   : TEXCOORD5; // 雾系数和顶点光照，x: fogFactor, yzw: vertex light
    half   fogFactor                 : TEXCOORD5; // 雾系数
    float4 shadowCoord              : TEXCOORD6;
    half3  viewDirTS                : TEXCOORD7;
    DECLARE_LIGHTMAP_OR_SH(staticLightmapUV, vertexSH, 8);//光照贴图纹理坐标，光照贴图名称，球谐光照名称，纹理坐标索引
    float2  dynamicLightmapUV : TEXCOORD9; // Dynamic lightmap UVs

    float4  positionCS               : SV_POSITION; //齐次裁剪空间顶点坐标
    
    UNITY_VERTEX_INPUT_INSTANCE_ID //GPU Instancing获取实例ID
    UNITY_VERTEX_OUTPUT_STEREO //用于VR平台的宏定义
};
```

##### InitializeInputData 函数
```c file:InitializeInputData函数
void InitializeInputData(Varyings input, half3 normalTS, out InputData inputData)
{
    inputData = (InputData)0; //初始化

    //世界空间顶点
#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    inputData.positionWS = input.positionWS;
#endif
    
    //世界空间观察向量
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);

//------------------------------------------------------------------------
    //⭐NormalMap处理步骤：
    //TBN矩阵
#if defined(_NORMALMAP) || defined(_DETAIL)
    float sgn = input.tangentWS.w;      // should be either +1 or -1
    float3 bitangent = sgn * cross(input.normalWS.xyz, input.tangentWS.xyz);
    half3x3 tangentToWorld = half3x3(input.tangentWS.xyz, bitangent.xyz, input.normalWS.xyz);

    #if defined(_NORMALMAP)
    inputData.tangentToWorld = tangentToWorld;
    #endif
    //使用TBN矩阵将法线转换到世界空间
    inputData.normalWS = TransformTangentToWorld(normalTS, tangentToWorld);
#else
    inputData.normalWS = input.normalWS;
#endif
    //将法线标准化
    inputData.normalWS = NormalizeNormalPerPixel(inputData.normalWS);
    inputData.viewDirectionWS = viewDirWS;
    
//--------------------------------------------------------------------
    //⭐获取阴影坐标
#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR) //如果需要顶点阴影坐标
    inputData.shadowCoord = input.shadowCoord;
#elif defined(MAIN_LIGHT_CALCULATE_SHADOWS) //如果主光开启了计算阴影
    //传入世界空间顶点坐标得到阴影坐标
    inputData.shadowCoord = TransformWorldToShadowCoord(inputData.positionWS);
#else
    inputData.shadowCoord = float4(0, 0, 0, 0);
#endif

//--------------------------------------------------------------------
#ifdef _ADDITIONAL_LIGHTS_VERTEX //当额外灯光选择了逐顶点光照
    //保存雾系数和顶点光照
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactorAndVertexLight.x); 
    inputData.vertexLighting = input.fogFactorAndVertexLight.yzw; 
#else
    //只保存雾系数
    inputData.fogCoord = InitializeInputDataFog(float4(input.positionWS, 1.0), input.fogFactor);
#endif

//--------------------------------------------------------------------
#if defined(DYNAMICLIGHTMAP_ON)
    //调用SAMPLE_GI宏定义得到全局光照（球谐函数）
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.dynamicLightmapUV, input.vertexSH, inputData.normalWS);
#else
    inputData.bakedGI = SAMPLE_GI(input.staticLightmapUV, input.vertexSH, inputData.normalWS);
#endif

    inputData.normalizedScreenSpaceUV = GetNormalizedScreenSpaceUV(input.positionCS);
    inputData.shadowMask = SAMPLE_SHADOWMASK(input.staticLightmapUV);

    #if defined(DEBUG_DISPLAY)
    #if defined(DYNAMICLIGHTMAP_ON)
    inputData.dynamicLightmapUV = input.dynamicLightmapUV;
    #endif
    #if defined(LIGHTMAP_ON)
    inputData.staticLightmapUV = input.staticLightmapUV;
    #else
    inputData.vertexSH = input.vertexSH;
    #endif
    #endif
}
```

```c file:将法线标准化的函数
//普通的normalize
float3 NormalizeNormalPerVertex(float3 normalWS)
{
    return normalize(normalWS);
}

//兼容长度为0的向量
float3 NormalizeNormalPerPixel(float3 normalWS)
{
    //使用XYZ法线映射编码，我们偶尔采样接近零长度的法线，导致Inf/NaN
    #if defined(UNITY_NO_DXT5nm) && defined(_NORMALMAP)
        return SafeNormalize(normalWS);
    #else
        return normalize(normalWS);
    #endif
}
```

```c file:传入世界空间顶点坐标得到阴影坐标
float4 TransformWorldToShadowCoord(float3 positionWS)
{
#ifdef _MAIN_LIGHT_SHADOWS_CASCADE //如果开启了级联阴影
    half cascadeIndex = ComputeCascadeIndex(positionWS); //获取级联索引
#else
    half cascadeIndex = half(0.0);
#endif
    //根据级联索引获取对应的阴影坐标
    float4 shadowCoord = mul(_MainLightWorldToShadow[cascadeIndex], float4(positionWS, 1.0));

    return float4(shadowCoord.xyz, 0);
}
```

##### 顶点着色器函数
```c
Varyings LitPassVertex(Attributes input)
{
    Varyings output = (Varyings)0; //初始化

    UNITY_SETUP_INSTANCE_ID(input); //GPU Instancing获取实例ID
    UNITY_TRANSFER_INSTANCE_ID(input, output); //将实例ID从输入结构体传递到输出结构体
    UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output); //用于VR平台的宏定义

    //获取顶点位置信息
    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);

    // normalWS and tangentWS已经标准化
    // this is required to avoid skewing the direction during interpolation
    // also required for per-vertex lighting and SH evaluation
    VertexNormalInputs normalInput = GetVertexNormalInputs(input.normalOS, input.tangentOS);

    //计算顶点光照
    half3 vertexLight = VertexLighting(vertexInput.positionWS, normalInput.normalWS);

    //计算雾系数
    half fogFactor = 0;
    #if !defined(_FOG_FRAGMENT)
        fogFactor = ComputeFogFactor(vertexInput.positionCS.z);
    #endif
    
    //得到纹理坐标
    output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);

    // 保存法线切线信息
    output.normalWS = normalInput.normalWS;
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR) || defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    real sign = input.tangentOS.w * GetOddNegativeScale();
    half4 tangentWS = half4(normalInput.tangentWS.xyz, sign);
#endif
#if defined(REQUIRES_WORLD_SPACE_TANGENT_INTERPOLATOR)
    output.tangentWS = tangentWS;
#endif


#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    //世界空间观察方向
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(vertexInput.positionWS);
    //切线空间观察方向
    half3 viewDirTS = GetViewDirectionTangentSpace(tangentWS, output.normalWS, viewDirWS);
    output.viewDirTS = viewDirTS;
#endif

    //OUTPUT_LIGHTMAP_UV()宏得到光照贴图纹理坐标lightmapUV
    OUTPUT_LIGHTMAP_UV(input.staticLightmapUV, unity_LightmapST, output.staticLightmapUV);
#ifdef DYNAMICLIGHTMAP_ON
    output.dynamicLightmapUV = input.dynamicLightmapUV.xy * unity_DynamicLightmapST.xy + unity_DynamicLightmapST.zw;
#endif
    //OUTPUT_SH宏得到顶点球谐光照
    OUTPUT_SH(output.normalWS.xyz, output.vertexSH);

    //保存雾系数和顶点光照
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    output.fogFactorAndVertexLight = half4(fogFactor, vertexLight);
#else
    output.fogFactor = fogFactor;
#endif

#if defined(REQUIRES_WORLD_SPACE_POS_INTERPOLATOR)
    output.positionWS = vertexInput.positionWS;
#endif

#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
    output.shadowCoord = GetShadowCoord(vertexInput);
#endif

    output.positionCS = vertexInput.positionCS;

    return output;
}
```

###### 获取顶点位置和法线信息

```cs file:Core.hlsl：顶点位置/法线输入结构体
struct VertexPositionInputs
{
    float3 positionWS; 
    float3 positionVS; 
    float4 positionCS; 
    float4 positionNDC;
};

struct VertexNormalInputs
{
    real3 tangentWS;
    real3 bitangentWS;
    float3 normalWS;
};

//输入模型空间坐标，得到其他空间坐标
VertexPositionInputs GetVertexPositionInputs(float3 positionOS)
{
    VertexPositionInputs input;
    input.positionWS = TransformObjectToWorld(positionOS);
    input.positionVS = TransformWorldToView(input.positionWS);
    input.positionCS = TransformWorldToHClip(input.positionWS);

    float4 ndc = input.positionCS * 0.5f;
    input.positionNDC.xy = float2(ndc.x, ndc.y * _ProjectionParams.x) + ndc.w;
    input.positionNDC.zw = input.positionCS.zw;

    return input;
}


//输入模型空间法线，获得世界法线，其他分量只是简单填充
VertexNormalInputs GetVertexNormalInputs(float3 normalOS)
{
    VertexNormalInputs tbn;
    tbn.tangentWS = real3(1.0, 0.0, 0.0);
    tbn.bitangentWS = real3(0.0, 1.0, 0.0);
    tbn.normalWS = TransformObjectToWorldNormal(normalOS);
    return tbn;
}

//输入模型空间法线和切线，获取TBN矩阵及其分量
VertexNormalInputs GetVertexNormalInputs(float3 normalOS, float4 tangentOS)
{
    VertexNormalInputs tbn;

    // 兼容mikktsSpace.在片元处提取法线时只进行归一化
    real sign = real(tangentOS.w) * GetOddNegativeScale();
    tbn.normalWS = TransformObjectToWorldNormal(normalOS);
    tbn.tangentWS = real3(TransformObjectToWorldDir(tangentOS.xyz));
    tbn.bitangentWS = real3(cross(tbn.normalWS, float3(tbn.tangentWS))) * sign;
    return tbn;
}
```
###### 计算顶点光照
```cs file:Lighting.hlsl:计算顶点光照
half3 VertexLighting(float3 positionWS, half3 normalWS)
{
    half3 vertexLightColor = half3(0.0, 0.0, 0.0);

//当开启额外灯光并将其设置为顶点光照
#ifdef _ADDITIONAL_LIGHTS_VERTEX
    uint lightsCount = GetAdditionalLightsCount(); //获取所有额外灯光数量

    //遍历所有灯光并累加颜色
    LIGHT_LOOP_BEGIN(lightsCount)
        Light light = GetAdditionalLight(lightIndex, positionWS);
        half3 lightColor = light.color * light.distanceAttenuation;
        vertexLightColor += LightingLambert(lightColor, light.direction, normalWS);
    LIGHT_LOOP_END
#endif

    return vertexLightColor;
}

```
###### 计算雾系数
```cs file:Core.hlsl:计算雾系数
real ComputeFogFactor(float zPositionCS)
{
    //由于OpenGL和Direct3D保存深度值的范围不同
    //使用UNITY_Z_0_FAR_FROM_CLIPSPACE()宏针对不哦她那个平台重新映射深度值的范围
    float clipZ_0Far = UNITY_Z_0_FAR_FROM_CLIPSPACE(zPositionCS); 
    
    return ComputeFogFactorZ0ToFar(clipZ_0Far);
}

real ComputeFogFactorZ0ToFar(float z)
    {
    //当使用线性雾
    #if defined(FOG_LINEAR) 
    // factor = (end-z)/(end-start) = z * (-1/(end-start)) + (end/(end-start))
    float fogFactor = saturate(z * unity_FogParams.z + unity_FogParams.w);
    return real(fogFactor);

    //当使用指数雾
    #elif defined(FOG_EXP) || defined(FOG_EXP2)
    // factor = exp(-(density*z)^2)
    // -density * z computed at vertex
    return real(unity_FogParams.x * z);
    #else
        return real(0.0);
    #endif
}
```

###### 光照贴图和球谐光照宏定义

```c file:lighting.hlsl:光照贴图和球谐光照宏定义
//如果使用了光照贴图
#if defined(LIGHTMAP_ON) 
    //该宏用于声明光照贴图的纹理坐标
    #define DECLARE_LIGHTMAP_OR_SH(lmName, shName, index) float2 lmName : TEXCOORD##index
    //该宏用于计算光照贴图的纹理坐标（就是一个正常的uv计算）
    #define OUTPUT_LIGHTMAP_UV(lightmapUV, lightmapScaleOffset, OUT) OUT.xy = lightmapUV.xy * lightmapScaleOffset.xy + lightmapScaleOffset.zw;
    //空定义
    #define OUTPUT_SH(normalWS, OUT) 

//否则使用球谐光照
#else
    //该宏用于声明球谐光照贴图的纹理坐标
    #define DECLARE_LIGHTMAP_OR_SH(lmName, shName, index) half3 shName : TEXCOORD##index
    //空定义
    #define OUTPUT_LIGHTMAP_UV(lightmapUV, lightmapScaleOffset, OUT)
    //该宏用于计算球谐光照
    #define OUTPUT_SH(normalWS, OUT) OUT.xyz = SampleSHVertex(normalWS)
#endif
```
###### 获取阴影坐标
```c file:Shadows.hlsl:获取阴影坐标
float4 GetShadowCoord(VertexPositionInputs vertexInput)
{
#if defined(_MAIN_LIGHT_SHADOWS_SCREEN) && !defined(_SURFACE_TYPE_TRANSPARENT)
    return ComputeScreenPos(vertexInput.positionCS);
#else
    return TransformWorldToShadowCoord(vertexInput.positionWS);
#endif
}
```

##### 片元着色器函数
```c
void LitPassFragment(
    Varyings input
    , out half4 outColor : SV_Target0
#ifdef _WRITE_RENDERING_LAYERS
    , out float4 outRenderingLayers : SV_Target1
#endif
)
{
    UNITY_SETUP_INSTANCE_ID(input); //GPU Instancing获取实例ID
    UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input); //用于VR平台的宏定义

    //视察贴图
#if defined(_PARALLAXMAP)
#if defined(REQUIRES_TANGENT_SPACE_VIEW_DIR_INTERPOLATOR)
    half3 viewDirTS = input.viewDirTS;
#else
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(input.positionWS);
    half3 viewDirTS = GetViewDirectionTangentSpace(input.tangentWS, input.normalWS, viewDirWS);
#endif
    ApplyPerPixelDisplacement(viewDirTS, input.uv);
#endif

    //表面数据结构体
    SurfaceData surfaceData;
    //初始化
    InitializeStandardLitSurfaceData(input.uv, surfaceData);

#ifdef LOD_FADE_CROSSFADE
    LODFadeCrossFade(input.positionCS);
#endif
    
    //声明InputData结构体
    InputData inputData;
    //初始化
    InitializeInputData(input, surfaceData.normalTS, inputData);
    SETUP_DEBUG_TEXTURE_DATA(inputData, input.uv, _BaseMap);

#ifdef _DBUFFER
    ApplyDecalToSurfaceData(input.positionCS, surfaceData, inputData);
#endif

    //SurfaceData结构体和InputData结构体最终要传入UniversalFragmentPBR函数中
    //经过一系列内置函数最终计算出颜色
    half4 color = UniversalFragmentPBR(inputData, surfaceData);
    
    //将渲染颜色与三种类型的雾效进行混合
    color.rgb = MixFog(color.rgb, inputData.fogCoord);
    //计算透明度
    color.a = OutputAlpha(color.a, IsSurfaceTypeTransparent(_Surface));

    outColor = color;

#ifdef _WRITE_RENDERING_LAYERS
    uint renderingLayers = GetMeshRenderingLayer();
    outRenderingLayers = float4(EncodeMeshRenderingLayer(renderingLayers), 0, 0, 0);
#endif
}
```

```cs
struct SurfaceData
{
    half3 albedo;
    half3 specular;
    half  metallic;
    half  smoothness;
    half3 normalTS;
    half3 emission;
    half  occlusion;
    half  alpha;
    half  clearCoatMask;
    half  clearCoatSmoothness;
};
```
### ShadowCaster
阴影投射 Pass，计算灯光的阴影贴图
```cs
Pass
{
    Name "ShadowCaster"
    Tags
    {
        "LightMode" = "ShadowCaster"
    }

    // -------------------------------------
    // Render State Commands
    ZWrite On
    ZTest LEqual
    ColorMask 0  //只保存阴影信息，不需要颜色绘制
    Cull[_Cull]

    HLSLPROGRAM
    #pragma target 2.0

    // -------------------------------------
    // Shader Stages
    #pragma vertex ShadowPassVertex
    #pragma fragment ShadowPassFragment

    // -------------------------------------
    // Material Keywords
    // 计算阴影贴图会用到透明度裁切属性
    #pragma shader_feature_local_fragment _ALPHATEST_ON 
    #pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A

    //--------------------------------------
    // GPU Instancing
    #pragma multi_compile_instancing
    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"

    // -------------------------------------
    // Universal Pipeline keywords

    // -------------------------------------
    // Unity defined keywords
    #pragma multi_compile_fragment _ LOD_FADE_CROSSFADE 

    // This is used during shadow map generation to differentiate between directional and punctual light shadows, as they use different formulas to apply Normal Bias
    #pragma multi_compile_vertex _ _CASTING_PUNCTUAL_LIGHT_SHADOW

    // -------------------------------------
    // Includes
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/Shaders/ShadowCasterPass.hlsl"
    ENDHLSL
}
```
#### ShadowCasterPass.hlsl
```c
//这些变量在应用阴影法线偏移时使用，并由UnityEngine.Rendering.Universal.ShadowUtils.SetupShadowCasterConstantBuffer在com.unity.render-pipelines.universal/Runtime/ShadowUtils.cs中设置
float3 _LightDirection; //对于方向灯，_LightDirection在应用阴影法线偏移时使用。
float3 _LightPosition; //对于聚光灯和点光源，_LightPosition用于计算实际的光方向，因为它在每个阴影投射几何顶点是不同的。

struct Attributes
{
    float4 positionOS   : POSITION;
    float3 normalOS     : NORMAL;    //用于法线偏移
    float2 texcoord     : TEXCOORD0; //用于采样透明贴图
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

struct Varyings
{
    float2 uv           : TEXCOORD0;
    float4 positionCS   : SV_POSITION;
};

//获得齐次裁剪空间下的阴影坐标
float4 GetShadowPositionHClip(Attributes input)
{
    float3 positionWS = TransformObjectToWorld(input.positionOS.xyz);
    float3 normalWS = TransformObjectToWorldNormal(input.normalOS);

#if _CASTING_PUNCTUAL_LIGHT_SHADOW
    float3 lightDirectionWS = normalize(_LightPosition - positionWS);
#else
    float3 lightDirectionWS = _LightDirection;
#endif

    float4 positionCS = TransformWorldToHClip(ApplyShadowBias(positionWS, normalWS, lightDirectionWS));

//反向Z方式Z-Fighting
#if UNITY_REVERSED_Z
    positionCS.z = min(positionCS.z, UNITY_NEAR_CLIP_VALUE);
#else
    positionCS.z = max(positionCS.z, UNITY_NEAR_CLIP_VALUE);
#endif

    return positionCS;
}

Varyings ShadowPassVertex(Attributes input)
{
    Varyings output;
    UNITY_SETUP_INSTANCE_ID(input);

    output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
    output.positionCS = GetShadowPositionHClip(input);
    return output;
}

half4 ShadowPassFragment(Varyings input) : SV_TARGET
{
    //透明度裁剪
    Alpha(SampleAlbedoAlpha(input.uv, TEXTURE2D_ARGS(_BaseMap, sampler_BaseMap)).a, _BaseColor, _Cutoff);

#ifdef LOD_FADE_CROSSFADE
    LODFadeCrossFade(input.positionCS);
#endif

    return 0;
}
```

```cs file:Shadow.hlsl:得到偏移后的阴影坐标
float3 ApplyShadowBias(float3 positionWS, float3 normalWS, float3 lightDirection)
{
    //_ShadowBias：x值是Depth Bias深度偏移，y是Normal Bias法线偏移
    //这两个值从灯光属性中设置
    //URP也可以在URPAsset中对除了点光源以外的所有灯光统一设置
    
    //得到背向灯光的暗面
    float invNdotL = 1.0 - saturate(dot(lightDirection, normalWS));
    //相乘得到法线方向的偏移程度
    float scale = invNdotL * _ShadowBias.y;

    //阴影世界空间坐标沿着灯光方向偏移
    positionWS = lightDirection * _ShadowBias.xxx + positionWS;
    //再沿着法线偏移
    positionWS = normalWS * scale.xxx + positionWS;
    return positionWS;
}
```

### GBUFFER
待补充

### DepthOnly
计算摄像机的深度信息
```cs
Pass
{
    Name "DepthOnly"
    Tags
    {
        "LightMode" = "DepthOnly"
    }

    // -------------------------------------
    // Render State Commands
    ZWrite On
    ColorMask R  //深度图只需要R通道
    Cull[_Cull]

    HLSLPROGRAM
    #pragma target 2.0

    // -------------------------------------
    // Shader Stages
    #pragma vertex DepthOnlyVertex
    #pragma fragment DepthOnlyFragment

    // -------------------------------------
    // Material Keywords
    // 计算深度图会用到透明度裁切属性
    #pragma shader_feature_local_fragment _ALPHATEST_ON
    #pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A

    // -------------------------------------
    // Unity defined keywords
    #pragma multi_compile_fragment _ LOD_FADE_CROSSFADE

    //--------------------------------------
    // GPU Instancing
    #pragma multi_compile_instancing
    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"

    // -------------------------------------
    // Includes
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/Shaders/DepthOnlyPass.hlsl"
    ENDHLSL
}
```
#### DepthOnlyPass.hlsl

```c file:DepthOnlyPass.hlsl
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#if defined(LOD_FADE_CROSSFADE)
    #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/LODCrossFade.hlsl"
#endif

struct Attributes
{
    float4 position     : POSITION;
    float2 texcoord     : TEXCOORD0;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

struct Varyings
{
    float2 uv           : TEXCOORD0;
    float4 positionCS   : SV_POSITION;
    UNITY_VERTEX_INPUT_INSTANCE_ID
    UNITY_VERTEX_OUTPUT_STEREO
};

Varyings DepthOnlyVertex(Attributes input)
{
    Varyings output = (Varyings)0;
    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);

    output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
    output.positionCS = TransformObjectToHClip(input.position.xyz);
    return output;
}

half DepthOnlyFragment(Varyings input) : SV_TARGET
{
    UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);
    //透明度裁剪
    Alpha(SampleAlbedoAlpha(input.uv, TEXTURE2D_ARGS(_BaseMap, sampler_BaseMap)).a, _BaseColor, _Cutoff);

#ifdef LOD_FADE_CROSSFADE
    LODFadeCrossFade(input.positionCS);
#endif
    //返回深度
    return input.positionCS.z;
}
```
### DepthNormals
绘制 `_CameraNormalsTexture` 纹理时使用
```cs
// This pass is used when drawing to a _CameraNormalsTexture texture
Pass
{
    Name "DepthNormals"
    Tags
    {
        "LightMode" = "DepthNormals"
    }

    // -------------------------------------
    // Render State Commands
    ZWrite On
    Cull[_Cull]

    HLSLPROGRAM
    #pragma target 2.0

    // -------------------------------------
    // Shader Stages
    #pragma vertex DepthNormalsVertex
    #pragma fragment DepthNormalsFragment

    // -------------------------------------
    // Material Keywords
    #pragma shader_feature_local _NORMALMAP
    #pragma shader_feature_local _PARALLAXMAP
    #pragma shader_feature_local _ _DETAIL_MULX2 _DETAIL_SCALED
    #pragma shader_feature_local_fragment _ALPHATEST_ON
    #pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A

    // -------------------------------------
    // Unity defined keywords
    #pragma multi_compile_fragment _ LOD_FADE_CROSSFADE

    // -------------------------------------
    // Universal Pipeline keywords
    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/RenderingLayers.hlsl"

    //--------------------------------------
    // GPU Instancing
    #pragma multi_compile_instancing
    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"

    // -------------------------------------
    // Includes
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitDepthNormalsPass.hlsl"
    ENDHLSL
}
```

### Meta
将材质的 Albedo 和 Emission 属性传递给 Unity 的烘焙系统，从而保证物体能够被准确计算出间接照明。因此只有 Shader 中带 MetaPass，物体才能烘焙出光照贴图，并且只有在烘焙光照贴图是，MetaPass 才会被执行。
```cs
// This pass it not used during regular rendering, only for lightmap baking.
    Pass
    {
        Name "Meta"
        Tags
        {
            "LightMode" = "Meta"
        }

        // -------------------------------------
        // Render State Commands
        Cull Off  //烘焙需要考虑到物体背面，因此关闭Cull

        HLSLPROGRAM
        #pragma target 2.0

        // -------------------------------------
        // Shader Stages
        #pragma vertex UniversalVertexMeta
        #pragma fragment UniversalFragmentMetaLit

        // -------------------------------------
        // Material Keywords
        #pragma shader_feature_local_fragment _SPECULAR_SETUP
        #pragma shader_feature_local_fragment _EMISSION
        #pragma shader_feature_local_fragment _METALLICSPECGLOSSMAP
        #pragma shader_feature_local_fragment _ALPHATEST_ON
        #pragma shader_feature_local_fragment _ _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A
        #pragma shader_feature_local _ _DETAIL_MULX2 _DETAIL_SCALED
        #pragma shader_feature_local_fragment _SPECGLOSSMAP
        #pragma shader_feature EDITOR_VISUALIZATION

        // -------------------------------------
        // Includes
        #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
        #include "Packages/com.unity.render-pipelines.universal/Shaders/LitMetaPass.hlsl"

        ENDHLSL
    }
```

#### UniversalMetaPass.hlsl
```cs file:UniversalMetaPass.hlsl
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/MetaInput.hlsl"

struct Attributes
{
    float4 positionOS   : POSITION;
    float3 normalOS     : NORMAL;
    float2 uv0          : TEXCOORD0; //模型uv
    float2 uv1          : TEXCOORD1; //静态光照贴图uv
    float2 uv2          : TEXCOORD2; //动态光照（实时GI）贴图uv
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

struct Varyings
{
    float4 positionCS   : SV_POSITION;
    float2 uv           : TEXCOORD0;
#ifdef EDITOR_VISUALIZATION
    float2 VizUV        : TEXCOORD1;
    float4 LightCoord   : TEXCOORD2;
#endif
};

Varyings UniversalVertexMeta(Attributes input)
{
    Varyings output = (Varyings)0;
    //获取齐次裁剪空间顶点位置
    output.positionCS = UnityMetaVertexPosition(input.positionOS.xyz, input.uv1, input.uv2);
    output.uv = TRANSFORM_TEX(input.uv0, _BaseMap);
#ifdef EDITOR_VISUALIZATION
    UnityEditorVizData(input.positionOS.xyz, input.uv0, input.uv1, input.uv2, output.VizUV, output.LightCoord);
#endif
    return output;
}

half4 UniversalFragmentMeta(Varyings fragIn, MetaInput metaInput)
{
#ifdef EDITOR_VISUALIZATION
    metaInput.VizUV = fragIn.VizUV;
    metaInput.LightCoord = fragIn.LightCoord;
#endif

    return UnityMetaFragment(metaInput);
}
#endif

```

```cs file:获取齐次裁剪空间顶点位置
float4 UnityMetaVertexPosition(float3 vertex, float2 uv1, float2 uv2, float4 lightmapST, float4 dynlightmapST)
{
#ifndef EDITOR_VISUALIZATION
    if (unity_MetaVertexControl.x)  //x分量表示使用uv1作为光栅化坐标
    {
        vertex.xy = uv1 * lightmapST.xy + lightmapST.zw;
        // OpenGL right now needs to actually use incoming vertex position,
        // so use it in a very dummy way
        vertex.z = vertex.z > 0 ? REAL_MIN : 0.0f;
        //REAL_MIN表示无线接近于0的数值
    }
    if (unity_MetaVertexControl.y) //x分量表示使用uv2作为光栅化坐标
    {
        vertex.xy = uv2 * dynlightmapST.xy + dynlightmapST.zw;
        // OpenGL right now needs to actually use incoming vertex position,
        // so use it in a very dummy way
        vertex.z = vertex.z > 0 ? REAL_MIN : 0.0f;
    }
    return TransformWorldToHClip(vertex);
#else
    return TransformObjectToHClip(vertex);
#endif
}
```
### Universal2D
使用 2D 渲染器绘制物体时调用，不需要进行光照计算

```cs
Pass
{
    Name "Universal2D"
    Tags
    {
        "LightMode" = "Universal2D"
    }

    // -------------------------------------
    // Render State Commands
    Blend[_SrcBlend][_DstBlend]
    ZWrite[_ZWrite]
    Cull[_Cull]

    HLSLPROGRAM
    #pragma target 2.0

    // -------------------------------------
    // Shader Stages
    #pragma vertex vert
    #pragma fragment frag

    // -------------------------------------
    // Material Keywords
    //透明度裁剪
    #pragma shader_feature_local_fragment _ALPHATEST_ON
    #pragma shader_feature_local_fragment _ALPHAPREMULTIPLY_ON

    #include_with_pragmas "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DOTS.hlsl"

    // -------------------------------------
    // Includes
    #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/Shaders/Utils/Universal2D.hlsl"
    ENDHLSL
}
```

#### Universal2D.hlsl
```cs file:Universal2D.hlsl

struct Attributes
{
    float4 positionOS       : POSITION;
    float2 uv               : TEXCOORD0;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

struct Varyings
{
    float2 uv        : TEXCOORD0;
    float4 vertex : SV_POSITION;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

Varyings vert(Attributes input)
{
    Varyings output = (Varyings)0;

    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_TRANSFER_INSTANCE_ID(input, output);

    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
    output.vertex = vertexInput.positionCS;
    output.uv = TRANSFORM_TEX(input.uv, _BaseMap);

    return output;
}

half4 frag(Varyings input) : SV_Target
{
    UNITY_SETUP_INSTANCE_ID(input);
    half2 uv = input.uv;
    half4 texColor = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, uv);
    half3 color = texColor.rgb * _BaseColor.rgb;
    half alpha = texColor.a * _BaseColor.a;
    //透明度裁切
    AlphaDiscard(alpha, _Cutoff);

    //是否开启透明度预乘
#ifdef _ALPHAPREMULTIPLY_ON
    color *= alpha;
#endif
    return half4(color, alpha);
}

```


# 多光源阴影

> [!bug] 
> 计算颜色时要将灯光衰减和阴影衰减属性考虑进去

**多光源思路：**
1.  使用 **GetMainLight** 函数获取主光源，然后进行漫反射与高光反射计算
2.  使用 **GetAdditionalLightsCount** 函数获取副光源个数，在一个 For 循环中使用 **GetAdditionalLight** 函数获取其他的副光源，进行漫反射与高光反射计算，叠加到光源结果上

在 URP 中，光照的计算不再像 Built-in 管线那样死板，全部由 Unity 的光照路径决定；现如今是由 URP 管线的脚本收集好场景中所有的光照信息，再传输给 Pass，由我们开发者在 Pass 中决定采用那些光源进行光照计算。


**多光源阴影思路：**
1.  阴影分为接收阴影和投射阴影两个部分，接收阴影开启部分关键字就可以了，但投射阴影需要写一个用与投射阴影的 Pass，该 Pass 的渲染路径采用 **ShadowCaster**（可以直接使用内置的投射阴影 pass）
2.  第一个 Pass，开启一系列相关关键字，用于接收阴影
3.  然后在片元着色器中使用 **TransformWorldToShadowCoord** 函数获取阴影纹理坐标，使用该坐标作为 `GetMainLight` 函数的参数获取主光源，再进行一系列光源计算
4.  第二个 Pass 参考 URP 的 `Universal Render Pipeline/Lit/ShadowCasterPass` 创建生成阴影的 Pass，主要任务在于将阴影从对象空间中转换到裁剪空间。

```c file:接收阴影关键字
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE
#pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
#pragma multi_compile _ _SHADOWS_SOFT
```

```cs file:投射阴影pass
UsePass "Universal Render Pipeline/Lit/ShadowCaster"
```

Settings 文件夹，URP Assets 中配置阴影：
![[Pasted image 20230629144401.png|500]]

```c fold file:多光源阴影
Shader "Custom/MultipleLightingShadows URP Shader"
{
    Properties
    {

        _MainTex ("MainTex", 2D) = "white" {}
        _BaseColor("BaseColor", Color) = (1,1,1,1)
        [Normal] _NormalMap("NormalMap", 2D) = "bump" {}
        _NormalScale("NormalScale", Range(0, 10)) = 1

        [Header(Specular)]
        _SpecularExp("SpecularExp", Range(1, 100)) = 32
        _SpecularStrength("SpecularStrength", Range(0, 10)) = 1
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)

        [Toggle] _AdditionalLights("开启多光源", Float) = 1
    }
    
    
    SubShader
    {
        Tags
        {
            "RenderPipeline" = "UniversalPipeline"
            "RenderType"="Opaque"
        }
        LOD 100

        //阴影接收pass
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            //开启多光源关键字
            #pragma shader_feature _ADDITIONALLIGHTS_ON

            //阴影关键字
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE
            #pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
            #pragma multi_compile _ _SHADOWS_SOFT
            
            #pragma multi_compile_instancing
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/lighting.hlsl"

            struct Attributes
            {
                float4 positionOS : POSITION;
                float4 color : COLOR;
                float3 normalOS : NORMAL;
                float4 tangentOS : TANGENT;
                float2 uv : TEXCOORD0;

                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct Varyings
            {
                float4 positionoCS : SV_POSITION;
                float4 color : COLOR0;
                float2 uv : TEXCOORD0;
                float3 positionWS: TEXCOORD1;
                float3 normalWS : TEXCOORD2;
                float4 tangentWS : TEXCOORD3;
                float3 bitangentWS : TEXCOORD4;
                float3 viewDirWS : TEXCOORD5;
                float3 lightDirWS : TEXCOORD6;

                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            CBUFFER_START(UnityPerMateiral)
            float4 _MainTex_ST;
            float4 _BaseColor;
            float _NormalScale;
            float _SpecularExp;
            float _SpecularStrength;
            float4 _SpecularColor;
            CBUFFER_END

            TEXTURE2D(_MainTex);
            SAMPLER(sampler_MainTex);
            TEXTURE2D(_NormalMap);
            SAMPLER(sampler_NormalMap);

            Varyings vert(Attributes i)
            {
                Varyings o = (Varyings)0;
                UNITY_SETUP_INSTANCE_ID(i);
                UNITY_TRANSFER_INSTANCE_ID(i, o);

                o.positionoCS = TransformObjectToHClip(i.positionOS.xyz);
                o.uv = i.uv.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                o.positionWS = TransformObjectToWorld(i.positionOS.xyz);
                o.normalWS = TransformObjectToWorldNormal(i.normalOS);
                o.tangentWS.xyz = TransformObjectToWorldDir(i.tangentOS.xyz);
                o.viewDirWS = normalize(_WorldSpaceCameraPos.xyz - o.positionWS);

                return o;
            }

            //多光源光照计算
            float3 AdditionalLighting(float3 normalWS,float3 viewDirWS, float3 lightDirWS , float3 lightColor, float lightAttenuation)
            {
                float3 H = normalize(lightDirWS + viewDirWS);
                float NdotL = dot(normalWS, lightDirWS);
                float NdotH = dot(normalWS, H);

                float3 diffuse = (0.5 * NdotL + 0.5) * _BaseColor.rgb * lightColor;
                float3 specular = pow(max(0, NdotH), _SpecularExp) * _SpecularStrength * _SpecularColor.rgb * lightColor;

                return (diffuse + specular) * lightAttenuation;
            }

            float4 frag(Varyings i) : SV_Target
            {
                UNITY_SETUP_INSTANCE_ID(i);

                //获取阴影坐标
                float4 shadowCoord = TransformWorldToShadowCoord(i.positionWS);
                
                //主光源(传入阴影坐标)
                Light mainLight = GetMainLight(shadowCoord);
                

                //纹理采样
                float4 MainTex = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv);
                float3 normalMap = UnpackNormalScale(
                    SAMPLE_TEXTURE2D(_NormalMap, sampler_NormalMap, i.uv), _NormalScale);

                //向量计算
                float3x3 TBN = CreateTangentToWorld(i.normalWS, i.tangentWS.xyz, i.tangentWS.w);
                float3 N = TransformTangentToWorld(normalMap, TBN, true);
                float3 L = normalize(mainLight.direction);
                float3 V = normalize(i.viewDirWS);
                float3 H = normalize(L + V);
                float NdotL = dot(N, L);
                float NdotH = dot(N, H);
                
                //主光源颜色计算（乘阴影衰减）
                float3 diffuse = (0.5 * NdotL + 0.5) * _BaseColor.rgb * mainLight.color;
                
                float3 specular = pow(max(0, NdotH), _SpecularExp) * _SpecularStrength * _SpecularColor.rgb * mainLight.color;
                float3 mainColor = (diffuse + specular)* mainLight.shadowAttenuation;


                //其他光源颜色计算
                //如果开启多光源
                float3 addColor = float3(0, 0, 0);
                #if _ADDITIONALLIGHTS_ON
                int addLightsCount = GetAdditionalLightsCount();
                for (int idx = 0; idx < addLightsCount; idx++)
                {
                    Light addLight = GetAdditionalLight(idx, i.positionWS);
                    
                    // 注意light.distanceAttenuation * light.shadowAttenuation，这里已经将距离衰减与阴影衰减进行了计算
                    addColor += AdditionalLighting(N,V, normalize(addLight.direction),  addLight.color, addLight.distanceAttenuation * addLight.shadowAttenuation);
                }
                #endif

                
                float4 finalColor = MainTex * float4(mainColor + addColor + _GlossyEnvironmentColor.rgb, 1);
                return finalColor;
            }
            ENDHLSL
        }
        
        //阴影投射pass
        UsePass "Universal Render Pipeline/Lit/ShadowCaster"
    }
    FallBack "Packages/com.unity.render-pipelines.universal/FallbackError"
}
```

