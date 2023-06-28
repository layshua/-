*   主光源 该函数有几个重载

```c
Light GetMainLight();
Light GetMainLight(float4 shadowCoord);
Light GetMainLight(float4 shadowCoord, float3 positionWS, half4 shadowMask)
```

*   返回结构体有以下数据：

```c
float3 light.direction
float light.distanceAttenuation
float light.shadowAttenuation
float3 light.color
```

*   返回其他光源的数量

```c
int GetAdditionalLightsCount();
```

*   返回该光源的数据

```c
Light GetAdditionalLight(uint i, float3 positionWS)
{
    int perObjectLightIndex = GetPerObjectLightIndex(i);
    return GetAdditionalPerObjectLight(perObjectLightIndex, positionWS);
}
```

*   环境光函数

```c
half3 SampleSH(half3 normalWS)
```

*   Unity 内置环境光

```c
half3  _GlossyEnvironmentColor;
```

*   采样光照贴图 不支持实时 GI

```c
half3 SampleLightmap(float2 lightmapUV, half3 normalWS);
half3 SubtractDirectMainLightFromLightmap(Light mainLight, half3 normalWS, half3 bakedGI);
```

*   混合实时光和 bakeGI

```c
void MixRealtimeAndBakedGI(inout Light light, half3 normalWS, inout half3 bakedGI)
```

**Light Function**

*   Lambert 函数

```c
half3 LightingLambert(half3 lightColor, half3 lightDir, half3 normal)
```

-blinn-phong 函数

```c
half3 LightingSpecular(half3 lightColor, half3 lightDir, half3 normal, half3 viewDir, half4 specular, half smoothness)
```

**Shadow**

- 表明该 Pass 选择阴影渲染模式

```c
Tags { "LightMode" = "ShadowCaster" }
```

- 用于获取应用阴影的深度偏移后的阴影坐标

```c
ApplyShadowBias(positionWS, normalWS, _LightDirection)
```

- 获取阴影坐标

```
TransformWorldToShadowCoord(positionWS)
```

- 用于接受阴影用的关键字

```
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE
#pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
#pragma multi_compile _ _SHADOWS_SOFT
```

示例

```
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

```
UsePass "Universal Render Pipeline/Lit/ShadowCaster"
```

*   URP 只支持使用单个 Pass 渲染 如果要开启多 Pass 需要按以下方式
*   第二个 Pass 添加

```
Tags{ "LightMode" = "SRPDefaultUnlit" }
```

*   即可让这两个 pass 同时生效