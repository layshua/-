sr-annote { all: unset; }

## 总体结构

1、在 SubShader 的 Tags 中添加`"RenderPipeline" = "UniversalPipeline"` 2、所有 URP 着色器都是 HLSL 编写的，使用宏`HLSL`包含的 shader 代码 3、使用 HLSLINCLUDE 替代 CGINCLUDE

<table><thead><tr><th>内置管线</th><th>URP</th></tr></thead><tbody><tr><td>CGPROGRAM HLSLPROGRAM</td><td>HLSLPROGRAM</td></tr><tr><td>ENDCG ENDHLSL</td><td>ENDHLSL</td></tr><tr><td>CGINCLUDE HLSLINCLUDE</td><td>HLSLINCLUDE</td></tr></tbody></table>

## Include 文件

<table><thead><tr><th>内容</th><th>内置管线</th><th>URP</th></tr></thead><tbody><tr><td>Core</td><td>Unity.cginc</td><td><a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl" target="_blank" rel="noopener">Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl</a></td></tr><tr><td>Light</td><td>AutoLight.cginc</td><td><a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl" target="_blank" rel="noopener">Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl</a></td></tr><tr><td>Shadow</td><td>AutoLight.cginc</td><td><a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl" target="_blank" rel="noopener">Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl</a></td></tr><tr><td>表面着色器</td><td>Lighting.cginc</td><td>URP 内没有，可以参考项目：<a href="https://github.com/phi-lira/UniversalShaderExamples/tree/master/Assets/_ExampleScenes/51_LitPhysicallyBased" target="_blank" rel="noopener">在此处</a></td></tr></tbody></table>

其他有用的包括：

*   [Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl)
*   [Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl)
*   [Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl)
*   [Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl)
*   [Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl)
*   [Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl)
*   [Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareOpaqueTextue.hlsl](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/DeclareOpaqueTexture.hlsl)

## 灯光模式 LightMode

<table><thead><tr><th>内置管线</th><th>URP</th></tr></thead><tbody><tr><td>ForwardBase</td><td>UniversalForward</td></tr><tr><td>ForwardAdd</td><td>移除</td></tr><tr><td>Deferred 以及相关</td><td>尚未支持</td></tr><tr><td>Vertex 及相关</td><td>移除</td></tr><tr><td>ShadowCaster</td><td>ShadowCaster</td></tr><tr><td>MotionVectors</td><td>尚未支持</td></tr></tbody></table>

支持的其他照明模式包括：

*   DepthOnly
*   Meta (for lightmap baking)
*   Universal2D

## 变体 Variants

URP 支持某些变体，因此，根据你使用的功能，可能需要使用`#pragma multi_compile`添加一些关键字：

*   `_MAIN_LIGHT_SHADOWS`
*   `_MAIN_LIGHT_SHADOWS_CASCADE`
*   `_ADDITIONAL_LIGHTS_VERTEX`
*   `_ADDITIONAL_LIGHTS`
*   `_ADDITIONAL_LIGHT_SHADOWS`
*   `_SHADOWS_SOFT`
*   `_MIXED_LIGHTING_SUBTRACTIVE`

## 预定义的着色器宏

### 辅助宏

<table><thead><tr><th>内置管线</th><th>URP</th></tr></thead><tbody><tr><td><strong>UNITY_PROJ_COORD</strong>(<em>a</em>)</td><td>移除了，使用<strong> a.xy / a.w</strong> 代替</td></tr><tr><td><strong>UNITY_INITIALIZE_OUTPUT</strong>(<em>type</em>，<em>name</em>)</td><td><strong>ZERO_INITIALIZE</strong>(<em>type</em>，<em>name</em>)</td></tr></tbody></table>

### 阴影贴图

必须 include [“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl”](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl)

<table><thead><tr><th>内置管线</th><th>URP</th></tr></thead><tbody><tr><td><strong>UNITY_DECLARE_SHADOWMAP</strong>（<em>tex</em>）</td><td><strong>TEXTURE2D_SHADOW_PARAM</strong>（<em>textureName</em>，<em>samplerName</em>）</td></tr><tr><td><strong>UNITY_SAMPLE_SHADOW</strong>（<em>tex</em>，<em>uv</em>）</td><td><strong>SAMPLE_TEXTURE2D_SHADOW</strong>（<em>textureName</em>，<em>samplerName</em>，<em>coord3</em>）</td></tr><tr><td><strong>UNITY_SAMPLE_SHADOW_PROJ</strong>（<em>tex</em>，<em>uv</em>）</td><td><strong>SAMPLE_TEXTURE2D_SHADOW</strong>（<em>textureName</em>，<em>samplerName</em>，<em>coord4.xyz/coord4.w</em>）</td></tr></tbody></table>

### 纹理 / 采样器声明宏

Unity 有很多纹理 / 采样器宏来改善 API 之间的交叉兼容性，但是人们并不习惯使用它们。URP 中这些宏的名称有所变化。由于数量很多，全部的宏可以在 [API includes 中](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/API)查看，下面主要列举一些常用的：

<table><thead><tr><th>内置管线</th><th>URP</th></tr></thead><tbody><tr><td><strong>UNITY_DECLARE_TEX2D</strong>（<em>name</em>）</td><td><strong>TEXTURE2D</strong>（<em>textureName</em>）; <strong>SAMPLER</strong>（<em>samplerName</em>）;</td></tr><tr><td><strong>UNITY_DECLARE_TEX2D_NOSAMPLER</strong>（<em>name</em>）</td><td><strong>TEXTURE2D</strong>（<em>textureName</em>）;</td></tr><tr><td><strong>UNITY_DECLARE_TEX2DARRAY</strong>（<em>name</em>）</td><td><strong>TEXTURE2D_ARRAY</strong>（<em>textureName</em>）; <strong>SAMPLER</strong>（<em>samplerName</em>）;</td></tr><tr><td><strong>UNITY_SAMPLE_TEX2D</strong>（<em>name</em>，<em>uv</em>）</td><td><strong>SAMPLE_TEXTURE2D</strong>（<em>textureName</em>，<em>samplerName</em>，<em>coord2</em>）</td></tr><tr><td><strong>UNITY_SAMPLE_TEX2D_SAMPLER</strong>（<em>name</em>，<em>samplername</em>，<em>uv</em>）</td><td><strong>SAMPLE_TEXTURE2D</strong>（<em>textureName</em>，<em>samplerName</em>，<em>coord2</em>）</td></tr><tr><td><strong>UNITY_SAMPLE_TEX2DARRAY</strong>（<em>name</em>，<em>uv</em>）</td><td><strong>SAMPLE_TEXTURE2D_ARRAY</strong>（<em>textureName</em>，<em>samplerName</em>，<em>coord2</em>，<em>index</em>）</td></tr><tr><td><strong>UNITY_SAMPLE_TEX2DARRAY_LOD</strong>（<em>name</em>，<em>uv</em>，<em>lod</em>）</td><td><strong>SAMPLE_TEXTURE2D_ARRAY_LOD</strong>（<em>textureName</em>，<em>samplerName</em>，<em>coord2</em>，<em>index</em>，<em>lod</em>）</td></tr></tbody></table>

需要注意`SCREENSPACE_TEXTURE`变成了`TEXTURE2D_X`。如果你想要在 VR 中（_Single Pass Instanced_ 或 _Multi-view_ 模式）制作屏幕效果，你必须使用`TEXTURE2D_X`定义纹理。这个宏会为你处理正确的纹理声明（是否为数组）。对这个纹理采样的时候必须使用`SAMPLE_TEXTURE2D_X`，并且对 uv 使用`UnityStereoTransformScreenSpaceTex`。

## Shader 辅助函数

下列函数可以在此文件中找到：[“Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl”](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl).

### 顶点转换函数

<table><thead><tr><th>内置管线</th><th>URP</th></tr></thead><tbody><tr><td><em>float4</em> <strong>UnityObjectToClipPos</strong>（<em>float3 pos</em>）</td><td><em>float4</em> <strong>TransformObjectToHClip</strong>（<em>float3 positionOS</em>）</td></tr><tr><td><em>float3</em> <strong>UnityObjectToViewPos</strong>（<em>float3 pos</em>）</td><td><strong>TransformWorldToView</strong>（<strong>TransformObjectToWorld</strong>（<em>positionOS</em>））</td></tr></tbody></table>

### 通用辅助函数

<table><thead><tr><th>内置管线</th><th>URP</th><th></th></tr></thead><tbody><tr><td><em>float3</em> <strong>WorldSpaceViewDir</strong>（<em>float4 v</em>）</td><td><em>float3</em> <strong>GetWorldSpaceViewDir</strong>（<em>float3 positionWS</em>）</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl”</a></td></tr><tr><td><em>float3</em> <strong>ObjSpaceViewDir</strong>（<em>float4 v</em>）</td><td>移除了，可以使用<code>TransformWorldToObject(GetCameraPositionWS()) - objectSpacePosition</code> ;</td><td></td></tr><tr><td><em>float2</em> <strong>ParallaxOffset</strong>（<em>half h</em>，<em>half height</em>，<em>half3 viewDir</em>）</td><td>移除了。可以从 UnityCG.cginc 复制过来</td><td></td></tr><tr><td><em>fixed</em> <strong>Luminance</strong>（<em>fixed3 c</em>）</td><td><em>real</em> <strong>Luminance</strong>（<em>real3 linearRgb</em>）</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl”</a></td></tr><tr><td><em>fixed3</em> <strong>DecodeLightmap</strong>（<em>fixed4 color</em>）</td><td><em>real3</em> <strong>DecodeLightmap</strong>(<em>real4 encodedIlluminance</em>, <em>real4 decodeInstructions</em>)</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/blob/master/com.unity.render-pipelines.core/ShaderLibrary/EntityLighting.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.core/ShaderLibrary/EntityLighting.hlsl”</a> URP 中的<code>decodeInstructions</code>是<code>half4(LIGHTMAP_HDR_MULTIPLIER, LIGHTMAP_HDR_EXPONENT, 0.0h, 0.0h)</code></td></tr><tr><td><em>float4</em> <strong>EncodeFloatRGBA</strong>（<em>float v</em>）</td><td>移除了。可以从 UnityCG.cginc 复制过来</td><td></td></tr><tr><td><em>float</em> <strong>DecodeFloatRGBA</strong>（<em>float4 enc</em>）</td><td>移除了。可以从 UnityCG.cginc 复制过来</td><td></td></tr><tr><td><em>float2</em> <strong>EncodeFloatRG</strong>（<em>float v</em>）</td><td>移除了。可以从 UnityCG.cginc 复制过来</td><td></td></tr><tr><td><em>float</em> <strong>DecodeFloatRG</strong>（<em>float2 enc</em>）</td><td>移除了。可以从 UnityCG.cginc 复制过来</td><td></td></tr><tr><td><em>float2</em> <strong>EncodeViewNormalStereo</strong>（<em>float3 n</em>）</td><td>移除了。可以从 UnityCG.cginc 复制过来</td><td></td></tr><tr><td><em>float3</em> <strong>DecodeViewNormalStereo</strong>（<em>float4 enc4</em>）</td><td>移除了。可以从 UnityCG.cginc 复制过来</td><td></td></tr></tbody></table>

### 前向渲染辅助函数

<table><thead><tr><th>内置管线</th><th>URP</th><th></th></tr></thead><tbody><tr><td><em>float3</em> <strong>WorldSpaceLightDir</strong>(<em>float4 v</em>)</td><td><em>_MainLightPosition.xyz</em> - <strong>TransformObjectToWorld</strong>(<em>objectSpacePosition</em>)</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl”</a></td></tr><tr><td><em>float3</em> <strong>ObjSpaceLightDir</strong>(<em>float4 v</em>)</td><td><strong>TransformWorldToObject</strong>(<em>_MainLightPosition.xyz</em>)<em>-objectSpacePosition</em></td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl”</a></td></tr><tr><td><em>float3</em> <strong>Shade4PointLights</strong>(<em>…</em>)</td><td>可以使用<code>half3 VertexLighting(float3 positionWS, half3 normalWS)</code></td><td>对于<code>VertexLighting(...)</code> include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl”</a></td></tr></tbody></table>

### 屏幕空间辅助函数

<table><thead><tr><th>内置管线</th><th>URP</th><th></th></tr></thead><tbody><tr><td><em>float4</em> <strong>ComputeScreenPos</strong>(<em>float4 clipPos</em>)</td><td><em>float4</em> <strong>ComputeScreenPos</strong>（<em>float4 positionCS</em>）</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl”</a></td></tr><tr><td><em>float4</em> <strong>ComputeGrabScreenPos</strong>(<em>float4 clipPos</em>)</td><td>移除了</td><td></td></tr></tbody></table>

### 顶点照明辅助函数[↑](https://teodutra.com/unity/shaders/urp/graphics/2020/05/18/From-Built-in-to-URP/#summary)

<table><thead><tr><th>内置管线</th><th>URP</th><th></th></tr></thead><tbody><tr><td><em>float3</em> <strong>ShadeVertexLights</strong> (<em>float4 vertex</em>, <em>float3 normal</em>)</td><td>移除了，可以尝试使用<code>UNITY_LIGHTMODEL_AMBIENT.xyz + VertexLighting(...)</code></td><td>对于<code>VertexLighting(...)</code> include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl”</a></td></tr></tbody></table>

可以在 [“Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl”](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl) 中找到很多工具函数。

## 内置着色器变量

除了光照相关的变量外，其他的变量名都基本没变

### 照明[↑](https://teodutra.com/unity/shaders/urp/graphics/2020/05/18/From-Built-in-to-URP/#summary)

<table><thead><tr><th>内置管线</th><th>URP</th><th></th></tr></thead><tbody><tr><td><em>_LightColor0</em></td><td><em>_MainLightColor</em></td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl”</a></td></tr><tr><td><em>_WorldSpaceLightPos0</em></td><td><em>_MainLightPosition</em></td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl”</a></td></tr><tr><td><em>_LightMatrix0</em></td><td>移除了。目前尚不支持 Cookie</td><td></td></tr><tr><td><em>unity_4LightPosX0</em>，<em>unity_4LightPosY0</em>，<em>unity_4LightPosZ0</em></td><td>在 URP 中，其他光源存储在数组 / 缓冲区中（取决于平台）。使用<code>Light GetAdditionalLight(uint i, float3 positionWS)</code>获取额外光源信息</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl”</a></td></tr><tr><td><em>unity_4LightAtten0</em></td><td>在 URP 中，其他光源存储在数组 / 缓冲区中（取决于平台）。使用<code>Light GetAdditionalLight(uint i, float3 positionWS)</code>获取额外光源信息</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl”</a></td></tr><tr><td><em>unity_LightColor</em></td><td>在 URP 中，其他光源存储在数组 / 缓冲区中（取决于平台）。使用<code>Light GetAdditionalLight(uint i, float3 positionWS)</code>获取额外光源信息</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl”</a></td></tr><tr><td><em>unity_WorldToShadow</em></td><td><code>float4x4 _MainLightWorldToShadow[MAX_SHADOW_CASCADES + 1]</code> 或者<code>_AdditionalLightsWorldToShadow[MAX_VISIBLE_LIGHTS]</code></td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl”</a></td></tr></tbody></table>

如果要使用循环所有其他灯光`GetAdditionalLight(...)`，`GetAdditionalLightsCount()`可以使用来查询其他灯光计数。

## 其他

### 阴影

有关阴影的更多信息，[“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl”](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl)

<table><thead><tr><th>内置管线</th><th>URP</th><th></th></tr></thead><tbody><tr><td><strong>UNITY_SHADOW_COORDS</strong>（<em>x</em>）</td><td>移除了。DIY，例如<code>float4 shadowCoord : TEXCOORD0;</code></td><td></td></tr><tr><td><strong>TRANSFER_SHADOW</strong>（<em>a</em>）</td><td><em>a.shadowCoord</em> = <strong>TransformWorldToShadowCoord</strong>（<em>worldSpacePosition</em>）</td><td>启用 cascades 时，对片段执行此操作以避免视觉鬼影</td></tr><tr><td><strong>SHADOWS_SCREEN</strong></td><td>移除了。不支持。</td><td></td></tr></tbody></table>

### 雾

有关雾的更多信息，[“Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl”](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl).

<table><thead><tr><th>内置管线</th><th>URP</th></tr></thead><tbody><tr><td><strong>UNITY_FOG_COORDS</strong>(<em>x</em>)</td><td>移除了。DIY，例如<code>float fogCoord : TEXCOORD0;</code></td></tr><tr><td><strong>UNITY_TRANSFER_FOG</strong>(o*, outpos)</td><td><em>o.fogCoord</em> = <strong>ComputeFogFactor</strong>(<em>clipSpacePosition.z</em>);</td></tr><tr><td><strong>UNITY_APPLY_FOG</strong>(<em>coord</em>, <em>col</em>)</td><td><em>color</em> = <strong>MixFog</strong>(<em>color</em>, <em>i.fogCoord</em>);</td></tr></tbody></table>

### 深度

要使用相机深度纹理，需要 include [“Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl”](https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl) ，然后会自动声明`_CameraDepthTexture`，也会包含辅助函数`SampleSceneDepth(...)`和`LoadSceneDepth(...)`。

<table><thead><tr><th>内置管线</th><th>URP</th><th></th></tr></thead><tbody><tr><td><strong>LinearEyeDepth</strong>（<em>sceneZ</em>）</td><td><strong>LinearEyeDepth</strong>（<em>sceneZ</em>，<em>_ZBufferParams</em>）</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl”</a></td></tr><tr><td><strong>Linear01Depth</strong>（<em>sceneZ</em>）</td><td><strong>Linear01Depth</strong>（<em>sceneZ</em>，<em>_ZBufferParams</em>）</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl”</a></td></tr></tbody></table>

### 其他中的其他

<table><thead><tr><th>内置管线</th><th>URP</th><th></th></tr></thead><tbody><tr><td><strong>ShadeSH9</strong>（<em>normal</em>）</td><td><strong>SampleSH</strong>（<em>normal</em>）</td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl”</a></td></tr><tr><td><em>unity_ColorSpaceLuminance</em></td><td>移除了。使用<code>Luminance()</code></td><td>Include <a href="https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl" target="_blank" rel="noopener">“Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl”</a></td></tr></tbody></table>

## 后处理 / 视觉特效

URP 不支持`OnPreCull`，`OnPreRender`，`OnPostRender`和`OnRenderImage`这些方法。URP 支持`OnRenderObject`和`OnWillRenderObject`，但是如果在 URP 中使用你可能会发现问题。因此，如果你曾经在旧管线创建视觉效果时使用它们，那么现在你需要学习新方法了。URP 包含以下注入点：

*   `beginCameraRendering(ScriptableRenderContext context, Camera camera)`
*   `endCameraRendering(ScriptableRenderContext context, Camera camera)`
*   `beginFrameRendering(ScriptableRenderContext context,Camera[] cameras)`
*   `endFrameRendering(ScriptableRenderContext context,Camera[] cameras)`

用法示例：

```
void OnEnable() {
    RenderPipelineManager.beginCameraRendering += MyCameraRendering;
}

void OnDisable() {
    RenderPipelineManager.beginCameraRendering -= MyCameraRendering;
}

void MyCameraRendering(ScriptableRenderContext context, Camera camera) {
    ...
    if(camera == myEffectCamera)
    {
    ...
    }
    ...
}
```

就像我说的那样，`OnWillRenderObject`是受支持的，但是，如果你需要在其中执行渲染调用（例如，水反射 / 折射），它将无法正常工作。调用`Camera.Render()`，你将看到以下消息：

_Recursive rendering is not supported in SRP (are you calling Camera.Render from within a render pipeline?)_

翻译过来就是：

SRP 不支持递归渲染（你是从渲染管道中调用 Camera.Render 吗？）

在这种情况下，URP 中应该将`OnWillRenderObject`替换为`begin/endCameraRendering`（如上面的例子），并调用`RenderSingleCamera()`，而不是 `Camera.Render()`。更改上面的示例，你将获得以下内容

```
void MyCameraRendering(ScriptableRenderContext context, Camera camera) {
    ...
    if(camera == myEffectCamera)
    {
    ...
        UniversalRenderPipeline.RenderSingleCamera(context, camera);
    }
    ...
}
```

使用后处理的另一种方法是使用`ScriptableRendererFeature`。[这篇文章](https://alexanderameye.github.io/outlineshader)很好地解释了使用 RenderFeature 的描边效果。`ScriptableRendererFeature`可以让你将`ScriptableRenderPass(es)`注入到渲染管线的不同阶段，因此是创建后处理效果的强大工具。注入位置可以包含以下：

*   `BeforeRendering`
*   `BeforeRenderingShadows`
*   `AfterRenderingShadows`
*   `BeforeRenderingPrepasses`
*   `AfterRenderingPrePasses`
*   `BeforeRenderingOpaques`
*   `AfterRenderingOpaques`
*   `BeforeRenderingSkybox`
*   `AfterRenderingSkybox`
*   `BeforeRenderingTransparents`
*   `AfterRenderingTransparents`
*   `BeforeRenderingPostProcessing`
*   `AfterRenderingPostProcessing`
*   `AfterRendering`

这是`ScriptableRendererFeature`使用自定义材质执行 blit 的简单示例：

```
public class CustomRenderPassFeature : ScriptableRendererFeature
{
    class CustomRenderPass : ScriptableRenderPass
    {
        CustomRPSettings _CustomRPSettings;
        RenderTargetHandle _TemporaryColorTexture;

        private RenderTargetIdentifier _Source;
        private RenderTargetHandle _Destination;

        public CustomRenderPass(CustomRPSettings settings)
        {
            _CustomRPSettings = settings;
        }

        public void Setup(RenderTargetIdentifier source, RenderTargetHandle destination)
        {
            _Source = source;
            _Destination = destination;
        }

        public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)
        {
            _TemporaryColorTexture.Init("_TemporaryColorTexture");
        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
            CommandBuffer cmd = CommandBufferPool.Get("My Pass");

            if (_Destination == RenderTargetHandle.CameraTarget)
            {
                cmd.GetTemporaryRT(_TemporaryColorTexture.id, renderingData.cameraData.cameraTargetDescriptor, FilterMode.Point);
                cmd.Blit(_Source, _TemporaryColorTexture.Identifier());
                cmd.Blit(_TemporaryColorTexture.Identifier(), _Source, _CustomRPSettings.m_Material);
            }
            else
            {
                cmd.Blit(_Source, _Destination.Identifier(), _CustomRPSettings.m_Material, 0);
            }

            context.ExecuteCommandBuffer(cmd);
            CommandBufferPool.Release(cmd);
        }

        public override void FrameCleanup(CommandBuffer cmd)
        {
            if (_Destination == RenderTargetHandle.CameraTarget)
            {
                cmd.ReleaseTemporaryRT(_TemporaryColorTexture.id);
            }
        }
    }

    [System.Serializable]
    public class CustomRPSettings
    {
        public Material m_Material;
    }

    public CustomRPSettings m_CustomRPSettings = new CustomRPSettings();
    CustomRenderPass _ScriptablePass;

    public override void Create()
    {
        _ScriptablePass = new CustomRenderPass(m_CustomRPSettings);

        _ScriptablePass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        _ScriptablePass.Setup(renderer.cameraColorTarget, RenderTargetHandle.CameraTarget);
        renderer.EnqueuePass(_ScriptablePass);
    }
}
```

你可以通过单击 **“Create> Rendering > Universal Render Pipeline > Renderer Feature”** 来创建一个`ScriptableRendererFeature`。你创建的功能必须添加到你的中`ForwardRenderer`。为此，选择`ForwardRenderer`，单击 **Add Renderer Feature**，然后选择要添加的功能。你可以在 Inspector 中公开属性，例如上面的例子中包含了一个材质球属性。

## com.unity.render-pipelines.universal

##### Core.hlsl

<table><thead><tr><th>名称</th><th>说明</th></tr></thead><tbody><tr><td>GetVertexPositionInputs(float3 positionOS)</td><td>获取输入顶点坐标信息</td></tr><tr><td>GetVertexNormalInputs(float3 normalOS)</td><td>获取输入顶点法线信息</td></tr><tr><td>GetVertexNormalInputs(float3 normalOS, float4 tangentOS)</td><td>获取输入顶点法线信息（重载）</td></tr><tr><td>GetScaledScreenParams()</td><td>获取屏幕缩放参数信息</td></tr><tr><td>NormalizeNormalPerVertex(real3 normalWS)</td><td>逐顶点法线正交</td></tr><tr><td>NormalizeNormalPerPixel(real3 normalWS)</td><td>逐像素法线正交</td></tr><tr><td>ComputeScreenPos(float4 positionCS)</td><td>计算屏幕坐标信息</td></tr><tr><td>（real）ComputeFogFactor(float z)</td><td>计算雾参数</td></tr><tr><td>（real）ComputeFogIntensity(real fogFactor)</td><td>计算雾强度</td></tr><tr><td>（half3）MixFogColor(real3 fragColor, real3 fogColor, real fogFactor)</td><td>混合雾颜色</td></tr><tr><td>（half3）MixFog(real3 fragColor, real fogFactor)</td><td>混合雾</td></tr></tbody></table>

##### Lighting.hlsl

<table><thead><tr><th>名称</th><th>说明</th></tr></thead><tbody><tr><td>DistanceAttenuation(float distanceSqr, half2 distanceAttenuation)</td><td>距离衰减</td></tr><tr><td>AngleAttenuation(half3 spotDirection, half3 lightDirection, half2 spotAttenuation)</td><td>角度衰减</td></tr><tr><td>GetMainLight()/GetMainLight(float4 shadowCoord)</td><td>获取主光源</td></tr><tr><td>GetPerObjectLightIndex(int index)</td><td>获取每个对象灯光 Index</td></tr><tr><td>GetAdditionalLightsCount()</td><td>获取额外灯光数量</td></tr><tr><td>ReflectivitySpecular(half3 specular)</td><td>高光反射率</td></tr><tr><td>OneMinusReflectivityMetallic(half metallic)</td><td>OneMinus 金属反射率</td></tr><tr><td>InitializeBRDFData(half3 albedo, half metallic, half3 specular, half smoothness, half alpha, out BRDFData outBRDFData)</td><td>初始化 BRDF</td></tr><tr><td>EnvironmentBRDF(BRDFData brdfData, half3 indirectDiffuse, half3 indirectSpecular, half fresnelTerm)</td><td>环境 BRDF</td></tr><tr><td>DirectBDRF(BRDFData brdfData, half3 normalWS, half3 lightDirectionWS, half3 viewDirectionWS)</td><td>BRDF</td></tr><tr><td>SampleLightmap(float2 lightmapUV, half3 normalWS)</td><td>光照贴图</td></tr><tr><td>GlossyEnvironmentReflection(half3 reflectVector, half perceptualRoughness, half occlusion)</td><td>环境光泽反射</td></tr><tr><td>GlobalIllumination(BRDFData brdfData, half3 bakedGI, half occlusion, half3 normalWS, half3 viewDirectionWS)</td><td>全局光照</td></tr><tr><td>MixRealtimeAndBakedGI(inout Light light, half3 normalWS, inout half3 bakedGI, half4 shadowMask)</td><td>实时烘培混合</td></tr><tr><td>LightingLambert(half3 lightColor, half3 lightDir, half3 normal)</td><td>兰伯特模型</td></tr><tr><td>LightingSpecular(half3 lightColor, half3 lightDir, half3 normal, half3 viewDir, half4 specular, half smoothness)</td><td>高光</td></tr><tr><td>LightingPhysicallyBased(BRDFData brdfData, half3 lightColor, half3 lightDirectionWS, half lightAttenuation, half3 normalWS, half3 viewDirectionWS)/LightingPhysicallyBased(BRDFData brdfData, Light light, half3 normalWS, half3 viewDirectionWS)</td><td>基于物理的光照模型</td></tr><tr><td>VertexLighting(float3 positionWS, half3 normalWS)</td><td>顶点光照颜色</td></tr><tr><td>LightweightFragmentPBR(InputData inputData, half3 albedo, half metallic, half3 specular,half smoothness, half occlusion, half3 emission, half alpha)</td><td>轻量级片元 PBR</td></tr><tr><td>LightweightFragmentBlinnPhong(InputData inputData, half3 diffuse, half4 specularGloss, half smoothness, half3 emission, half alpha)</td><td>轻量级片元布林 · 冯</td></tr></tbody></table>

##### Shadows.hlsl

<table><thead><tr><th>名称</th><th>说明</th></tr></thead><tbody><tr><td>GetMainLightShadowStrength()</td><td>获取主光源阴影强度</td></tr><tr><td>GetAdditionalLightShadowStrenth(int lightIndex)</td><td>获取额外光源阴影强度</td></tr><tr><td>SampleScreenSpaceShadowmap(float4 shadowCoord)</td><td>屏幕空间阴影贴图</td></tr><tr><td>SampleShadowmap(float4 shadowCoord, TEXTURE2D_SHADOW_PARAM(ShadowMap, sampler_ShadowMap), ShadowSamplingData samplingData, half shadowStrength, bool isPerspectiveProjection = true)</td><td>阴影贴图</td></tr><tr><td>TransformWorldToShadowCoord(float3 positionWS)</td><td>把顶点的世界坐标转换到阴影坐标</td></tr><tr><td>MainLightRealtimeShadow(float4 shadowCoord)</td><td>主光源实时阴影</td></tr><tr><td>AdditionalLightRealtimeShadow(int lightIndex, float3 positionWS)</td><td>额外光源实时阴影</td></tr><tr><td>GetShadowCoord(VertexPositionInputs vertexInput)</td><td>获取阴影坐标信息</td></tr><tr><td>ApplyShadowBias(float3 positionWS, float3 normalWS, float3 lightDirection)</td><td>应用阴影偏移</td></tr></tbody></table>

## com.unity.render-pipelines.core

##### SpaceTransforms.hlsl

变换矩阵：

<table><thead><tr><th>名称</th><th>说明</th></tr></thead><tbody><tr><td>TransformObjectToWorld(float3 positionOS)</td><td>当前模型空间转世界空间矩阵，通常用于把顶点 / 方向矢量从模型空间转到世界空间</td></tr><tr><td>TransformWorldToObject(float3 positionWS)</td><td>当前世界空间转模型空间矩阵，通常用于把顶点 / 方向矢量从世界空间转到模型空间</td></tr><tr><td>TransformWorldToView(float3 positionWS)</td><td>当前世界空间转相机空间矩阵，通常用于把顶点 / 方向矢量从世界空间转到相机空间</td></tr><tr><td>TransformObjectToHClip(float3 positionOS)</td><td>当前模型空间转裁剪空间矩阵，通常用于把顶点 / 方向矢量从模型空间转到裁剪空间</td></tr><tr><td>TransformWorldToHClip(float3 positionWS)</td><td>当前世界空间转裁剪空间矩阵，通常用于把顶点 / 方向矢量从世界空间转到裁剪空间</td></tr><tr><td>TransformWViewToHClip(float3 positionVS)</td><td>当前相机空间转裁剪空间矩阵，通常用于把顶点 / 方向矢量从相机空间转到裁剪空间</td></tr><tr><td>TransformObjectToWorldDir(real3 dirOS)</td><td>把方向矢量从模型空间转换到世界空间中</td></tr><tr><td>TransformWorldToObjectDir(real3 dirWS)</td><td>把方向矢量从世界空间转换到模型空间中</td></tr><tr><td>TransformWorldToViewDir(real3 dirWS)</td><td>把方向矢量从世界空间转换到相机空间中</td></tr><tr><td>TransformWorldToHClipDir(real3 directionWS)</td><td>把方向矢量从世界空间转换到裁剪空间中</td></tr><tr><td>TransformObjectToWorldNormal(float3 normalOS)</td><td>把法线从模型空间转换到世界空间中</td></tr><tr><td>CreateTangentToWorld(real3 normal, real3 tangent, real flipSign)</td><td>创建一个切线空间转为世界空间的 3x3 矩阵</td></tr><tr><td>TransformTangentToWorld(real3 dirTS, real3x3 tangentToWorld)</td><td>当前切线空间转世界空间矩阵，通常用于把顶点 / 方向矢量从切线空间转到世界空间</td></tr><tr><td>TransformWorldToTangent(real3 dirWS, real3x3 tangentToWorld)</td><td>当前世界空间转切线空间矩阵，通常用于把顶点 / 方向矢量从世界空间转到切线空间</td></tr><tr><td>TransformTangentToObject(real3 dirTS, real3x3 tangentToWorld)</td><td>当前切线空间转模型空间矩阵，通常用于把顶点 / 方向矢量从切线空间转到模型空间</td></tr></tbody></table>