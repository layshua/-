
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
|TXS|Texture Space 纹理空间 |
|SS|Screen Space 屏幕空间|

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
|CommonLighting |定义了灯光计算的通用函数|
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

### Core

| 名称                                                                  | 说明             |
|---------------------------------------------------------------------|----------------|
| GetVertexPositionInputs(float3 positionOS)                          | 获取输入顶点坐标信息     |
| GetVertexNormalInputs(float3 normalOS)                              | 获取输入顶点法线信息     |
| GetVertexNormalInputs(float3 normalOS, float4 tangentOS)            | 获取输入顶点法线信息（重载） |
| GetScaledScreenParams()                                             | 获取屏幕缩放参数信息     |
| NormalizeNormalPerVertex(real3 normalWS)                            | 逐顶点法线正交        |
| NormalizeNormalPerPixel(real3 normalWS)                             | 逐像素法线正交        |
| ComputeScreenPos(float4 positionCS)                                 | 计算屏幕坐标信息       |
| （real）ComputeFogFactor(float z)                                     | 计算雾参数          |
| （real）ComputeFogIntensity(real fogFactor)                           | 计算雾强度          |
| （half3）MixFogColor(real3 fragColor, real3 fogColor, real fogFactor) | 混合雾颜色          |
| （half3）MixFog(real3 fragColor, real fogFactor)                      | 混合雾            |

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
| `_MainLightColor` |主光源颜色|
|`_MainLightPosition`|主光源位置|
|int **GetAdditionalLightsCount**();|获取其他光源数量|
|Light **GetAdditionalLight**(int index, float3 WS_Pos)|获取其他的光源|
|half3 **SampleSH**(half3 normalWS)|环境光函数|
|half3  `_GlossyEnvironmentColor`|Unity 内置环境光 |
|half3 **LightingLambert**|Lambert  |
|half3 **LightingSpecular**|BlinnPhong|
|  |  |


| 名称                                                                                                                                                                                   | 说明              |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| GetMainLightShadowStrength()                                                                                                                                                         | 获取主光源阴影强度       |
| GetAdditionalLightShadowStrenth(int lightIndex)                                                                                                                                      | 获取额外光源阴影强度      |
| SampleScreenSpaceShadowmap(float4 shadowCoord)                                                                                                                                       | 屏幕空间阴影贴图        |
| SampleShadowmap(float4 shadowCoord, TEXTURE2D_SHADOW_PARAM(ShadowMap, sampler_ShadowMap), ShadowSamplingData samplingData, half shadowStrength, bool isPerspectiveProjection = true) | 阴影贴图            |
| TransformWorldToShadowCoord(float3 positionWS)                                                                                                                                       | 把顶点的世界坐标转换到阴影坐标 |
| MainLightRealtimeShadow(float4 shadowCoord)                                                                                                                                          | 主光源实时阴影         |
| AdditionalLightRealtimeShadow(int lightIndex, float3 positionWS)                                                                                                                     | 额外光源实时阴影        |
| GetShadowCoord(VertexPositionInputs vertexInput)                                                                                                                                     | 获取阴影坐标信息        |
| ApplyShadowBias(float3 positionWS, float3 normalWS, float3 lightDirection)                                                                                                           | 应用阴影偏移          |



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

| 名称                                                                                                                                                                                   | 说明              |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| GetMainLightShadowStrength()                                                                                                                                                         | 获取主光源阴影强度       |
| GetAdditionalLightShadowStrenth(int lightIndex)                                                                                                                                      | 获取额外光源阴影强度      |
| SampleScreenSpaceShadowmap(float4 shadowCoord)                                                                                                                                       | 屏幕空间阴影贴图        |
| SampleShadowmap(float4 shadowCoord, TEXTURE2D_SHADOW_PARAM(ShadowMap, sampler_ShadowMap), ShadowSamplingData samplingData, half shadowStrength, bool isPerspectiveProjection = true) | 阴影贴图            |
| TransformWorldToShadowCoord(float3 positionWS)                                                                                                                                       | 把顶点的世界坐标转换到阴影坐标 |
| MainLightRealtimeShadow(float4 shadowCoord)                                                                                                                                          | 主光源实时阴影         |
| AdditionalLightRealtimeShadow(int lightIndex, float3 positionWS)                                                                                                                     | 额外光源实时阴影        |
| GetShadowCoord(VertexPositionInputs vertexInput)                                                                                                                                     | 获取阴影坐标信息        |
| ApplyShadowBias(float3 positionWS, float3 normalWS, float3 lightDirection)                                                                                                           | 应用阴影偏移          |



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

# 其他
## 辅助函数

屏幕空间辅助函数

|内置管线| URP                                        |
|---------------------------------------------|--------------------------------------------|
| float4 ComputeScreenPos(float4 clipPos)     |float4 ComputeScreenPos（float4 positionCS）|
| float4 ComputeGrabScreenPos(float4 clipPos) |移除了|

通用

| 内置管线                                                    | URP                                                                           |
|---------------------------------------------------------|-------------------------------------------------------------------------------|
| float3 WorldSpaceViewDir（float4 v）                      | float3 GetWorldSpaceViewDir（float3 positionWS）                                |
| float3 ObjSpaceViewDir（float4 v）                        | 移除了，可以使用TransformWorldToObject(GetCameraPositionWS()) - objectSpacePosition ; |
| float2 ParallaxOffset（half h，half height，half3 viewDir） | 移除了。可以从UnityCG.cginc复制过来                                                      |
| fixed Luminance（fixed3 c）                               | real Luminance（real3 linearRgb）                                               |
| fixed3 DecodeLightmap（fixed4 color）                     | real3 DecodeLightmap(real4 encodedIlluminance, real4 decodeInstructions)      |
| float4 EncodeFloatRGBA（float v）                         | 移除了。可以从UnityCG.cginc复制过来                                                      |
| float DecodeFloatRGBA（float4 enc）                       | 移除了。可以从UnityCG.cginc复制过来                                                      |
| float2 EncodeFloatRG（float v）                           | 移除了。可以从UnityCG.cginc复制过来                                                      |
| float DecodeFloatRG（float2 enc）                         | 移除了。可以从UnityCG.cginc复制过来                                                      |
| float2 EncodeViewNormalStereo（float3 n）                 | 移除了。可以从UnityCG.cginc复制过来                                                      |
| float3 DecodeViewNormalStereo（float4 enc4）              | 移除了。可以从UnityCG.cginc复制过来                                                      |


## 雾

有关雾的更多信息，“Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl”

|内置管线|URP|
|---|---|
|**UNITY_FOG_COORDS**(_x_)|移除了。DIY，例如`float fogCoord : TEXCOORD0;`|
|**UNITY_TRANSFER_FOG**(o*, outpos)|_o.fogCoord_ = **ComputeFogFactor**(_clipSpacePosition.z_);|
|**UNITY_APPLY_FOG**(_coord_, _col_)|_color_ = **MixFog**(_color_, _i.fogCoord_);|

## 深度

要使用相机深度纹理，需要 include “Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl” ，然后会自动声明 `_CameraDepthTexture`，也会包含辅助函数 `SampleSceneDepth(...)` 和 `LoadSceneDepth(...)`。

|内置管线|URP|
|---|---|
|**LinearEyeDepth**（_sceneZ_） |**LinearEyeDepth**（sceneZ，_ZBufferParams）|
|**Linear01Depth**（_sceneZ_）|**Linear01Depth**（_sceneZ_，__ZBufferParams_）|
#  Lighting
### Light 组件
![[Pasted image 20230630210404.png|450]]

**Mode**：光源模式（都是局部光照）
- Realtime：实时光源。每帧实时计算，效果好，性能消耗大
    - 优点：灯光位置、强度、颜色会发生移动
    - 缺点：
        - 实时运算，开销大（逐个顶点运算/逐个像素运算）
        - 不能计算多次光线反射的效果，**只能计算一次光线反射效果**（只有直接光）
- Baked：烘焙光源，事先把光线经过多次反射（间接光）的效果计算在一张场景光照贴图（uv2）上，这张贴图作用于场景内的所有物体的表面。
    - 优点：光线模拟的效果真实，由于是在游戏开始前计算的，所以没有实时计算的开销
    - 缺点：不能实时运算
- Mixed：混合光源，可在运行时更改混合光源的属性。这样做将**更新光源的实时光照，但不会更新烘焙光照**。有三种烘焙模式可选：[[2 URP#^gmojg3]]

**lndirect Multiplier 间接乘数**：改变间接光的强度，定义由全局照明（GI）系统计算的反弹光的亮度。
低于 1，每次反弹会使光更暗
大于 1，每次反弹会使光更亮

**Render Mode**：渲染模式，设置选定灯光的渲染优先级
Auto 运行时确定，具体取决于附近灯光的亮度和当前的 Quality settings 
lmportanto 以逐像素模式渲染，效果逼真，消耗大。仅将该模式用于最显著的视觉效果（例如，玩家汽车的前灯）。
Not lmportant：以逐顶点/对象模式渲染，便宜

**Culling Mask**：剔除遮罩，可以选择性地排除对象组不受灯光影响

```cs file:代码控制
public Light light;
void Start()
{
    //Inspector面板上的属性都能获得
    light.intensity = 1;
}
```

**Shadow Type** **阴影类型**：
Baked Shadow Angle/Radius：为光的软阴影边缘添加软化
Strength 阴影暗度 0~1 之间，越大越黑
Resolution 阴影贴图渲染分辨率
Bias 阴影偏移，包括 depth bias 和 normal bias 
Normal Bias 阴影投射面沿法线收缩距离
Near Panel 渲染阴影的近裁剪面
Custom Shadow Layers: [[2 URP#自定义 Shadow Layers]]

**阴影图集设置**：URP Asset->Shadows
![[Pasted image 20230630213031.png|450]]
- 聚光灯渲染 1 张阴影贴图。
- 点光源渲染 6 张阴影贴图（立方体贴图中的面数）
- “平行光”为每个级联渲染 1 张阴影贴图，URP Asset->Shadows->Cascade Count 可以设置级联数量
- URP 使用一个用于所有实时灯光的阴影图集（shadow atlas）来渲染实时阴影，可以指定阴影图集的分辨率。并且要根据项目决定
    - 例如：如果场景有四个聚光灯和一个点光源；并且您希望每个阴影贴图的分辨率至少为256x256。场景需要渲染十个阴影贴图（每个聚光灯一个，点光源六个），每个贴图的分辨率为256x256。使用大小为512x512的阴影图集是不够的，因为它只能包含四个大小为256x256的贴图。因此，您应该使用大小为1024x1024的阴影图集，其中最多可以包含16个大小为256x256的贴图。
      
### Lighting 面板
Windows-Rendering-Lighting


![[Pasted image 20230605123417.png]]

![[Pasted image 20230605123701.png]]

![[Pasted image 20230701161135.png|450]]
实时 GI：[使用Enlighten-Unity的实时全球照明手册](https://docs.unity3d.com/cn/2022.3/Manual/realtime-gi-using-enlighten.html)

**混合灯光设置：**
Mixed Lighting ->**Lighting Mode**
[光照模式 - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/2022.3/Manual/lighting-mode.html) ^gmojg3
1. Baked Indirect：将实时直接照明与烘焙间接照明相结合。此模式提供具有实时阴影贴图的实时阴影。适用于中端硬件。
2. Shadowmask：将实时直接照明与烘焙间接照明相结合。它支持远距离游戏对象的烘焙阴影（带阴影遮罩），并将其与实时阴影（阴影贴图）自动混合。此照明模式适用于高端或中端硬件。
3. Subtractive：提供烘焙的直射和间接光照，**仅针对一个方向光渲染直接实时阴影**。适合于风格化的艺术效果或低端硬件。

**光照贴图设置：**[渐进光照贴图程序 - Unity 手册 (unity3d.com)](https://docs.unity3d.com/cn/2022.3/Manual/progressive-lightmapper.html)
Progressive Lightmapper：渐进光照贴图程序是一种基于路径追踪的光照贴图系统

### 烘焙光照贴图
Unity 提供了两种不同的技术来**预先计算全局光照 (GI)和反射光照**，它们分别是:
1. 烘焙光 (全称: Baked Global lllumination，烘焙全局光照)：作用在静态物体上
2. 预计算光 (全称: Precomputed Realtime Global Illumination，预计算实时全局光照)：作用在动态物体上

Unity 的 Enlighten 光照系统提供了这两种技术的解决方案，这两种技术都需要一个技术流程——**烘焙**

![[Pasted image 20230615003324.png]]

1. 静态 static 对象才能接收光照贴图，灯光 Mode 必须为 static 或 mixed
2. 导入的模型要确保有光照 uv，否则烘焙时模型不会受影响。也可以打开 [Mesh import settings](https://docs.unity3d.com/cn/2022.3/Manual/FBXImporter-Model.html) 开启生成光照贴图 UV ![[Pasted image 20230701163120.png|550]]
3. 若要包含在光照贴图中，对象的 Renderer 必须满足以下条件：
  - 具有 **Mesh Renderer** 或 **Terrain** 组件
  - Mesh Renderer->Lighting->开启 Contribute GI
  - 材质有 Meta Pass 

**烘焙模型有硬边缘接缝**：勾选 Stitch Seams
![[Pasted image 20230615004644.png|250]] ![[Pasted image 20230615004712.png|400]]

**烘焙后模型有小点**：由于渲 103 染精度太低造成，提高以下两个值即可

![[Pasted image 20230615005019.png|250]] ![[Pasted image 20230615005145.png|450]]

### 光照探针
光照贴图会记录场景中静态物体 Mesh 表面的光照信息，并将这些信息存储在光照贴图当中。
光照探针与光照贴图的相同点是, 两者都是对场景的光照信息进行的记录，但**光照探针记录的是是光线在场景中穿过空白区域后，其在场景中的信息。**

在场景中，如果一个非静态物体不进行烘培，在实时模式系也可以接受部分光源的直接光的效果，只需要在对应光源的 Light 组件当中将其 Mode 设置为是 Mixed 或者 Realtime 即可。但这种方法依然只能接受直接光，**如果想让场景中的非静态物体在没有光照贴图的情况下依然可以接收到场景的间接光, 就可以使用光照探针来达成。**

![[Pasted image 20230615164846.png|550]]
光照探针技术的核心，就是通过存储光线在空间中的关键信息，来插值计算出每个点的光照。

注意点：
- 光源设置为静态
- 物体设置为 mixed 或者 realtime
- 探针通过复制可以扩大范围
- **光照探针要布置在光照信息发生变化的位置**。

### 反射探针
有游戏当中会有很多物体具有反射效果，像是一些金属材质、玻璃等等。只要一个物体具有镜面特征，就可以产生反射。
但在游戏中实时的渲染反射的画面通常是一件性能消耗非常大的事情，为了减少运行时的性能消耗, Unity 提供了反射贴图技术，将这些反射的结果预存储在对应的反射贴图中, 在游戏进行时只需要加载即可实现反射的画面效果。和上文中讲解光照贴图时是一样的思路: 以空间换时间。

和光照探针的工作原理类似, 反射探针也是利用一些探针来记录环境中不同位置的信息。
当使用反射探针在场景中的关键点对其中心点周围的视觉环境进行采样与烘培后, 这些采样得到的反射信息结果会存储到一个立方体贴图上。此立方体贴图的六个面分别记录了其周围六个方向上面的视觉信息，当一个物体靠近了反射探针之后，采样得到的反射效果就会被应用到物体上。**当场景中存在多个反射探针时，其不同反射探针对应的反射效果会进行插值计算。**

注意点：
环境物体要设置为 static 才能被采集到

通常而言，要将其位置放置在场景中明显可以进行，比如让立方体的各个平面靠近房间墙壁的边缘。

URP Asset->Lighting
![[Pasted image 20230630213444.png]]


## 镜头光斑 Lens Flare
[Lens Flare (SRP) Data Asset | Universal RP | 14.0.8 --- 镜头光斑（SRP）数据资产|通用RP | 14.0.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/shared/lens-flare/lens-flare-asset.html)


# Camera
组件参考：[Camera component reference | Universal RP | 14.0.8 --- 相机组件参考|通用RP | 14.0.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/camera-component-reference.html#rendering)

urp 的相机和内置管线有较大差异
![[Pasted image 20230630214444.png]]

## URP 帧渲染循环
![[Pasted image 20230630202743.png]]

URP 渲染器为每个相机执行一个相机循环，该循环执行以下步骤：
1.  **Setup Culling Parameters 设置剔除参数**：配置用于确定剔除系统如何剔除灯光和阴影的参数。可以使用自定义渲染器 override 渲染管线的这一部分。 
2. **Culling 剔除**  ：使用上一步中的剔除参数来计算相机可见的可见渲染器、阴影投射器和灯光的列表。剔除参数和相机 [layer distances](https://docs.unity3d.com/ScriptReference/Camera-layerCullDistances.html) 会影响剔除和渲染性能。
3. **Build Rendering Data 生成渲染数据** ：捕获基于剔除输出和 URP Asset、Camera 和当前运行平台的质量设置的信息，以构建 `RenderingData` 。渲染数据告诉渲染器相机和当前选择的平台所需的渲染工作量和质量。
4. **Setup Renderer 设置渲染器** ： 构建 Render Pass  的列表，并根据渲染数据对其进行排队执行。可以使用自定义渲染器 overide 渲染管道的这一部分。
5. **Execute Renderer 执行渲染器**：执行队列中的每个 Pass 。渲染器将“Camera”图像输出到帧缓冲区（framebuffer） 。

## 通用附加摄影机数据组件
![[Pasted image 20230630221353.png]]
通用附加摄影机数据组件是通用渲染管道（URP）用于内部数据存储的组件。通用附加摄像头数据组件允许 URP 扩展和重载Unity 标准摄像头组件的功能和外观。
在 URP 中，具有相机组件的游戏对象也必须具有通用附加相机数据组件

脚本 API：[Class UniversalAdditionalCameraData | Universal RP | 14.0.8 (unity3d.com)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/api/UnityEngine.Rendering.Universal.UniversalAdditionalCameraData.html)
```cs
//使用脚本访问相机的通用附加相机数据组件 Universal Additional Camera Data component
var cameraData = camera.GetUniversalAdditionalCameraData();
```
## Render Type
![[Pasted image 20230630215440.png]]**Base**：通用相机，用于渲染到 Render target（屏幕或 Render Texture）
场景中必须至少有一个 Base 相机。在一个场景中可以有多个 Base 相机。可以单独使用Base 相机，也可以在相机堆栈中使用它。

![[Pasted image 20230630215445.png]]**Overlay**：在另一个相机的输出之上进行渲染。可以将 Base 相机的输出与一个或多个 Overlay 相机的输出组合。这被称为 [Camera stacking](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/camera-stacking.html)。可以使用 Overlay 相机在二维 UI 中创建3D 对象或车辆中的驾驶舱等效果。
必须将 Oberlay 相机与使用相机堆叠系统的一个或多个 Base 相机结合使用。您不能单独使用 Oberlay 相机。不属于“相机堆栈”（Camera Stack）的“Oberlay 相机”（Overlay Camera）不会执行其渲染循环的任何步骤，并且被称为孤立相机 orphan Camera.。

1. 相机堆栈中的Base相机决定相机堆栈的大部分属性。因为您只能在“相机堆栈”中使用“Oberlay相机”，所以在渲染场景时，URP 仅使用Oberlay相机的以下属性：
    - Projection
    - FOV Axis 
    - Field of View 
    - Physical Camera properties  
    - Clipping plans 
    - Renderer 
    - Clear Depth 
    - Render Shadows 
    - Culling Mask 
    - Occlusion Culling 
2. 不能将后处理应用于单个 Oberlay 相机。可以将后处理应用于单个 Base 相机或相机堆栈。

可以在脚本中更改相机的类型，方法是设置相机的通用附加相机数据组件的 `renderType` 属性，如下所示：
```cs file:更改相机的类型
var cameraData = camera.GetUniversalAdditionalCameraData();
cameraData.renderType = CameraRenderType.Base;
```

## 使用多摄像机
### Camera Stack 相机堆栈
![[Pasted image 20230630220146.png|450]]
使用 Camera Stack 对多个摄影机的输出进行分层，并创建单个组合输出。比如在2D UI 中创建3D 模型或车辆驾驶舱等效果。
摄影机堆栈由一个基本摄影机和一个或多个叠加摄影机组成。摄影机堆栈使用摄影机堆栈中所有摄影机的组合输出覆盖基础摄影机的输出。因此，**可以对“基本摄影机”的输出执行的任何操作，都可以对“摄影机堆栈”的输出进行。例如，可以将“摄影机堆栈”渲染到给定的渲染目标，应用后期处理效果，等等。**
要注意渲染顺序，减少 overdraw

**相机添加到堆栈**
1. 设置一个相机为 Base ，一个为 Overlay
2. Base 相机的 Camera 组件 —> Stack，添加 Overlay 相机
3. Overlay 相机的输出叠加在 Base 相机上面（栈结构，从上到下渲染，后面加的优先显示）
![[Pasted image 20230630220539.png]]

脚本控制 Base 相机的 Universal Additional Camera Data 组件的 `cameraStack` 属性
```cs file:相机堆栈操作
//将相机添加到堆栈
var cameraData = camera.GetUniversalAdditionalCameraData();
cameraData.cameraStack.Add(myOverlayCamera);

//将相机移出堆栈
var cameraData = camera.GetUniversalAdditionalCameraData();
cameraData.cameraStack.Remove(myOverlayCamera);
```

### 渲染到同一渲染目标
多个基本摄影机或摄影机堆栈可以渲染到同一渲染目标。这允许您创建诸如分屏渲染（split screen）之类的效果。

使用两个 Base Camera，设置 Inspector->Output->Viewport Rect

$XYWH$ 分别设置为 $(0,0,0.5,1) \quad(0.5,0,0.5,1)$
可以看到 Game 视口已经分屏：
![[Pasted image 20230630222857.png|500]]

```cs file:通过设置rect属性来更改摄影机的Viewport rect
myUniversalAdditionalCameraData.rect = new Rect(0.5f, 0f, 0.5f, 0f);
```

### 渲染到 RenderTexture
在 URP 中，渲染到 RenderTexture 的所有摄影机在渲染到屏幕的所有摄影机之前执行其渲染循环。这样可以确保 Render Textures 已准备好渲染到屏幕。

1. 创建 Render Texture，Assets > Create > Render Texture
2. 创建 Quad，将 RenderTexture 拖到 Quad 上作为纹理使用
3. 新建 Base Camera， Inpector->Output->Output Texture，放入创建的 Render Texture

通过设置摄影机的“通用附加摄影机数据”组件的 ``cameraOutput``属性，可以在脚本中设置摄影机的输出目标，如下所示：
```cs file:设置摄影机的输出目标
myUniversalAdditionalCameraData.cameraOutput = CameraOutput.Texture;
myCamera.targetTexture = myRenderTexture;
```
## Clearing 缓冲区清除
摄影机清除行为取决于摄影机的渲染类型

**Base Camera**
![[Pasted image 20230630225225.png]]
>Inspector->Environment

1. 颜色缓冲区 Color buffer：在渲染循环开始时清除为以下三种背景类型：
    - Skybox：如果没有找到 Skybox，默认为 Solid Color
    - Solid Color：将 Color buffer 清除为纯色
    - Uninitialized：使用未初始化的颜色缓冲区。未初始化的相机背景值未定义。只有当你渲染相机视图中的所有像素时才使用这个。
2. 深度缓冲区 Depth Buffer：始终在每个渲染循环开始时清除

**Overlay Camera**
1. 颜色缓冲区 Color buffer：在渲染循环开始时，Overlay Camera 会接收一个颜色缓冲区，该缓冲区包含摄影机堆栈中以前摄影机的颜色数据。它不会清除颜色缓冲区的内容。
2. 深度缓冲区 Depth Buffer：在渲染循环开始时，Overlay Camera 会接收一个深度缓冲区，其中包含来自摄影机堆栈中先前摄影机的深度数据。当“渲染类型”设置为“覆盖”时，可以使用 Camera Inspector->Rendering->Clear Depth 选项确定是否清除，默认清除。

## 剔除和渲染顺序
Culling 和 Rendering order

如果 URP 场景包含多个摄影机，Unity 将按可预测的顺序执行它们的剔除和渲染操作。


![[Pasted image 20230630232002.png|500]]
>优先级（Priority）设置： Camera inspector->Rendering
优先级较高的摄影机绘制在优先级较低的摄影机之上。优先级的范围从-100到100。仅当渲染类型设置为 Base 时，此属性才可见。  

**每一帧 Unity 执行以下操作：**
1. Unity 获取场景中所有活动的基本摄影机的列表
2. Unity 将活动的基本摄影机组织为两组：渲染到 Render Texture 和渲染到到屏幕
3. Unity 将渲染到 Render Texture 的 Base Camera 按“优先级”（Priority）顺序排序，以便最后绘制“优先级”值较高的摄影机。
4. 对于渲染到 Render Texture 的每个 Base Camera，Unity 执行以下步骤：
    1. 剔除 Base Camera
    2. 将 Base Camera 渲染到 Render Texture
    3. 对于作为 Base Camera 的摄影机堆栈一部分的每个 Overlay Camera，按照摄影机堆栈中定义的顺序：
        1. 剔除 Overlay Camera
        2. 将 Overlay Camera 渲染到 Render Texture
5. Unity 将渲染到屏幕的“Base Camera”按“优先级”顺序排序，以便“优先级”值较高的摄影机最后绘制。
6. 对于渲染到屏幕上的每个基本摄影机，Unity 执行以下步骤：
    1. 剔除 Base Camera
    2. 将 Base Camera 渲染到屏幕
    3. 对于作为 Base Camera 的摄影机堆栈一部分的每个 Overlay Camera，按照摄影机堆栈中定义的顺序：
        1. 剔除 Overlay Camera
        2. 将 Overlay Camera 渲染到屏幕


Unity 可以在一帧中多次渲染叠加摄影机的视图，或者是因为叠加摄影机出现在多个摄影机堆栈中，或者是叠加摄影机多次出现在同一摄影机堆栈中。当这种情况发生时，Unity 不会重用剔除或渲染操作的任何元素。按照上面详述的顺序，完全重复这些操作。（overdraw）

> [!bug] 重要提示
> 在此版本的 URP 中，只有在使用 Universal Renderer 时才支持叠加摄影机和摄影机堆栈。如果使用2D Renderer，叠加摄影机将不会执行其渲染循环的任何部分。

# Rendering Layers
渲染层功能允许将某些灯光配置为仅影响特定的游戏对象。
例如，在下图中，灯光 `A` 会影响球体 `D` ，但不会影响球体 `C` 。光 `B` 会影响球体 `C` ，但不会影响球体 `D` 。
![[Pasted image 20230630204416.png]]

可编辑渲染层名称
![[Pasted image 20230630204837.png|500]]
## 为灯光启用渲染层
  1. 灯光设置
![[Pasted image 20230630204527.png]]
![[eff088918bf5cc200f66ce8c5711f76e_MD5.png]]
>URP Asset > Lighting > Use Rendering Layers  

![[Pasted image 20230630205002.png|450]]
> Light > General > Rendering Layers

2. 对应 Mesh 设置，选择相应层即可
![[Pasted image 20230630205103.png|500]]
> Mesh Renderer > Additional Settings > Rendering Layer Mask
## 自定义 Shadow Layers
单独控制阴影投射
![[Pasted image 20230630205334.png|500]]
## 为 Decals 启用渲染层
![[Pasted image 20230630204720.png]]
>Decal Renderer feature
![[Pasted image 20230630204744.png]]

![[Pasted image 20230630205412.png]]
>在图像 `1` 中，油漆桶选择了 `Receive decals` 层。在图像 `2` 中，它没有，因此Decal Projector不会投影到桶上。

## 性能
1. 尽可能减少 Rendering Layers 的数量。避免创建不在项目中使用的 Rendering Layers。  
2. 将 Rendering Layers 用于贴花时，增加层数会增加所需的内存带宽并降低性能。 
3. 当仅对“正向渲染路径”中的灯光使用 Rendering Layers 时，性能影响很小。  
4. 当“Rendering Layers”计数超过8的倍数时，性能影响会更显著。例如：将层数从8层增加到9层比将层数从9层增加到10层具有更大的相对影响。 

# 多光源阴影

> [!bug] 
> 计算颜色时要将灯光衰减和阴影衰减属性考虑进去

**多光源思路：**
1.  使用 **GetMainLight** 函数获取主光源，然后进行漫反射与高光反射计算
2.  使用 **GetAdditionalLightsCount** 函数获取副光源个数，在一个 For 循环中使用 **GetAdditionalLight** 函数获取其他的副光源，进行漫反射与高光反射计算，叠加到光源结果上

在 URP 中，光照的计算不再像 Built-in 管线那样死板，全部由 Unity 的光照路径决定；现如今是由 URP 管线的脚本收集好场景中所有的光照信息，再传输给 Pass，由我们开发者在 Pass 中决定采用那些光源进行光照计算。

**多光源阴影思路：**
1.  阴影分为接收阴影和投射阴影两个部分，接收阴影开启部分关键字就可以了，但投射阴影需要写一个用与投射阴影的 Pass，该 Pass 的渲染路径采用 **ShadowCaster**（可以直接使用 URP 内置的投射阴影 pass，但导致不兼容 SRP Batecher）
2.  第一个 Pass，开启一系列相关关键字，用于接收阴影
3.  然后在片元着色器中使用 **TransformWorldToShadowCoord** 函数获取阴影纹理坐标，使用该坐标作为 `GetMainLight` 函数的参数获取主光源，再进行一系列光源计算
4.  第二个 Pass 参考 URP 的 `Universal Render Pipeline/Lit/ShadowCasterPass` 创建生成阴影的 Pass，主要任务在于将阴影从对象空间中转换到裁剪空间。

当我们通过 `UsePass` 方式进行投射阴影，SRP 的要求之一如下，：
- 必须在一个名为 `UnityPerMaterial` 的 CBUFFER 中声明所有材质属性。所以我们要共用一个 CBUFFER，把 CBUUFER 写在两个 Pass 之前。

![[Pasted image 20230629155628.png]]

可以自己写阴影的投射 pass，这样就可以继续开启 SRP Batcher，并支持 alpha test 的透明阴影。

```c file:接收阴影关键字
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS   //接收阴影
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE //TransformWorldToShadowCoord
#pragma multi_compile _ _ADDITIONAL_LIGHT_SHADOWS   //额外光源阴影
#pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS //开启额外光源计算
#pragma multi_compile _ _SHADOWS_SOFT  //软阴影
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

```c fold file:多光源阴影支持SRP和AlphaTest
Shader "Custom/MultipleLightingShadows for SRPBatche"
{
    Properties
    {
        [MainTexture] _MainTex ("MainTex", 2D) = "white" {}
        [MainColor] _BaseColor("BaseColor", Color) = (1,1,1,1)
        [Normal] _NormalMap("NormalMap", 2D) = "bump" {}
        _NormalScale("NormalScale", Range(0, 10)) = 1

        [Header(Specular)]
        _SpecularExp("SpecularExp", Range(1, 100)) = 32
        _SpecularStrength("SpecularStrength", Range(0, 10)) = 1
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)

        [Toggle] _AdditionalLights("开启多光源", Float) = 1
        
        [Toggle] _Cut("透明度裁剪", Float) = 1
        _Cutoff("透明度裁剪阈值", Range(0, 1)) = 1
    }
    
    SubShader
    {
        Tags
        {
            "RenderPipeline" = "UniversalPipeline"
        }
      
        HLSLINCLUDE
        
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/lighting.hlsl"
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"
        //开启多光源关键字
        #pragma shader_feature _ADDITIONALLIGHTS_ON

        //透明度裁剪关键字
        #pragma shader_feature _CUT_ON

        //阴影关键字
        #pragma multi_compile _ _MAIN_LIGHT_SHADOWS                          //接收阴影
        #pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE                  //TransformWorldToShadowCoord
        #pragma multi_compile _ _ADDITIONAL_LIGHT_SHADOWS                    //额外光源阴影
        #pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS //开启额外光源计算
        #pragma multi_compile _ _SHADOWS_SOFT                                //软阴影
    
        CBUFFER_START(UnityPerMaterial)
        float4 _MainTex_ST;
        float4 _BaseColor;
        float _NormalScale;
        float _SpecularExp;
        float _SpecularStrength;
        float4 _SpecularColor;
        float _Cutoff;
        CBUFFER_END

        TEXTURE2D(_MainTex);
        SAMPLER(sampler_MainTex);
        TEXTURE2D(_NormalMap);
        SAMPLER(sampler_NormalMap);

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
            float4 positionCS : SV_POSITION;
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
        ENDHLSL
        
        //阴影接收pass
        Pass
        {
            Tags
            {
                 "LightMode"="UniversalForward"
                 "RenderType"="TransparentCutout"
                 "Queue"="AlphaTest"
            }
            
            Cull off
            
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            
            
            
            #pragma multi_compile_instancing
            
            
            Varyings vert(Attributes input)
            {
                Varyings output = (Varyings)0;
                UNITY_SETUP_INSTANCE_ID(input);
                UNITY_TRANSFER_INSTANCE_ID(input, output);

                output.positionCS = TransformObjectToHClip(input.positionOS.xyz);
                output.uv = input.uv.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                output.positionWS = TransformObjectToWorld(input.positionOS.xyz);
                output.normalWS = TransformObjectToWorldNormal(input.normalOS);
                output.tangentWS.xyz = TransformObjectToWorldDir(input.tangentOS.xyz);
                output.viewDirWS = normalize(_WorldSpaceCameraPos.xyz - output.positionWS);

                return output;
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

            float4 frag(Varyings input) : SV_Target
            {
                UNITY_SETUP_INSTANCE_ID(input);

                //获取阴影坐标
                float4 shadowCoord = TransformWorldToShadowCoord(input.positionWS);
                
                //主光源(传入阴影坐标)
                Light mainLight = GetMainLight(shadowCoord);
                

                //纹理采样
                float4 MainTex = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, input.uv);

                #if _CUT_ON
                    clip(MainTex.a - _Cutoff);
                #endif
                
                float3 normalMap = UnpackNormalScale(
                    SAMPLE_TEXTURE2D(_NormalMap, sampler_NormalMap, input.uv), _NormalScale);

                //向量计算
                float3x3 TBN = CreateTangentToWorld(input.normalWS, input.tangentWS.xyz, input.tangentWS.w);
                float3 N = TransformTangentToWorld(normalMap, TBN, true);
                float3 L = normalize(mainLight.direction);
                float3 V = normalize(input.viewDirWS);
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
                for (int i = 0; i < addLightsCount; i++)
                {
                    Light addLight = GetAdditionalLight(i, input.positionWS);
                    
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
        Pass
        {
            Name "ShadowCaster"
            Tags
            {
                "LightMode" = "ShadowCaster"
            }
            
            ZWrite On
            ZTest LEqual
            ColorMask 0  //只保存阴影信息，不需要颜色绘制
            Cull Off

            HLSLPROGRAM
            #pragma target 2.0

            #pragma multi_compile_instancing
            #pragma vertex ShadowPassVertex
            #pragma fragment ShadowPassFragment
            
            //获得齐次裁剪空间下的阴影坐标
            float4 GetShadowPositionHClip(Attributes input)
            {
                float3 positionWS = TransformObjectToWorld(input.positionOS.xyz);
                float3 normalWS = TransformObjectToWorldNormal(input.normalOS);
            
                //光源方向
                //为什么只传了主光源方向，点光源也可以投射阴影
                //这个PASS走的 是 记录灯光视角下深度，点光源使用这个方向计算是错的，但只是影响一点offset，所以点光源还是能看到影子
                float3 lightDirectionWS = _MainLightPosition.xyz;

                //ApplyShadowBias()得到经过深度偏移和法线偏移后的世界空间阴影坐标
                //然后转换到齐次裁剪空间
                float4 positionCS = TransformWorldToHClip(ApplyShadowBias(positionWS, normalWS, lightDirectionWS));

            //反向Z防止Z-Fighting
            #if UNITY_REVERSED_Z
                positionCS.z = min(positionCS.z, UNITY_NEAR_CLIP_VALUE);
            #else
                positionCS.z = max(positionCS.z, UNITY_NEAR_CLIP_VALUE);
            #endif

                return positionCS;
            }
            
            Varyings ShadowPassVertex(Attributes input)
            {
                Varyings output = (Varyings)0;
                UNITY_SETUP_INSTANCE_ID(input);
                UNITY_TRANSFER_INSTANCE_ID(input, output);
                output.uv = input.uv.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                output.positionCS = GetShadowPositionHClip(input);
                return output;
            }

            half4 ShadowPassFragment(Varyings input) : SV_TARGET
            {
                 UNITY_SETUP_INSTANCE_ID(input);
                
                //纹理采样
                float4 MainTex = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, input.uv);

                #if _CUT_ON
                    clip(MainTex.a - _Cutoff);
                #endif
                
                return 0;
            }
            ENDHLSL
        }
    }
            
    FallBack "Packages/com.unity.render-pipelines.universal/FallbackError"
}
```

# Lit. shader 解析

> [!NOTE] 版本
> Unity2022.3.0f1c1：URP14.0

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

