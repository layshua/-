**BuiltIn 管线下的阴影**

*   阴影的接收

概要：

```c
//1、添加变体
 #pragma multi_compile DIRECTIONAL
 #pragma multi_compile SHADOWS_SCREEN

// 2、在v2f结构体中添加
UNITY_SHADOW_COORDS(1)
float3 worldPos :TEXCOORD2;

// 3、在顶点着色器中添加
o.worldPos = mul(unity_ObjectToWorld,v.vertex);
TRANSFER_SHADOW(o)

// 4、在片段着色器中添加
UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos)
return col * atten;
```

代码：

```c
Pass
        {
            Tags {
                "LightMode"="ForwardBase"
            }

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            // make fog work
            #pragma multi_compile_fog

             // 采样阴影时需要对应的变体开关 (这些其实是涉及调用的宏，这些宏里面有需要用到当前这些变体的状态)
            #pragma multi_compile DIRECTIONAL
            #pragma multi_compile SHADOWS_SCREEN

            #include "UnityCG.cginc"
            #include "AutoLight.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 pos : SV_POSITION;

                  // 阴影采样
                UNITY_SHADOW_COORDS(1)
                float3 worldPos :TEXCOORD2;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            v2f vert (appdata v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);

                 // 阴影采样
                o.worldPos = mul(unity_ObjectToWorld,v.vertex);
                TRANSFER_SHADOW(o)
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                // sample the texture
                fixed4 col = tex2D(_MainTex, i.uv);
                 // 阴影采样
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos)
                return col * atten;
            }
            ENDCG
        }
```

*   阴影的生成

概要：

```
// 1、添加tag Tags {"LightMode"="ShadowCaster"}

// 2、v2f结构体中添加 
V2F_SHADOW_CASTER;

// 3、顶点着色器中添加
TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)

// 4、片段着色器中添加
SHADOW_CASTER_FRAGMENT(i)
```

代码：

```
Pass
        {
            Tags {
                "LightMode"="ShadowCaster"
            }

            CGPROGRAM
 #pragma vertex vert
 #pragma fragment frag
 #pragma multi_compile_fog
 #include "UnityCG.cginc"
 #include "AutoLight.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
                half3 normal:NORMAL;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 pos : SV_POSITION;
                // 生成阴影
                V2F_SHADOW_CASTER;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            v2f vert (appdata v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                // 生成阴影
                TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {

                fixed4 col = tex2D(_MainTex, i.uv);
                // 生成阴影
                SHADOW_CASTER_FRAGMENT(i)
                return col;
            }
            ENDCG
        }
```

**URP 管线下的阴影**

*   阴影的接收

先来拆代码，找到默认的 URP 中的 shader，以 Lit.shader 为例。从 BuiltIn 管线中我们可以知道，阴影的名称常为 shadowxxx，然后还需要一个世界空间的坐标。

从 Lit.shader 出发，找到定义的顶点着色器和片段着色器来之 LitForwardPass.hlsl 。查找 shadow 可以在顶点着色器中找到

```
#if defined(REQUIRES_VERTEX_SHADOW_COORD_INTERPOLATOR)
	inputData.shadowCoord = input.shadowCoord;
#elif defined(MAIN_LIGHT_CALCULATE_SHADOWS)
        // 发现这里比较接近我们想要的内容
	inputData.shadowCoord = TransformWorldToShadowCoord(inputData.positionWS);
#else
	inputData.shadowCoord = float4(0, 0, 0, 0);
#endif
```

然后来到片段着色器中，这里不是很好看，直接贴目标相关代码

```
// shadowCoord 从顶点着色器中获取，然后传入到 GetMainLight 中
Light mainLight = GetMainLight(inputData.shadowCoord, inputData.positionWS, shadowMask);
// mainLight.shadowAttenuation 这个实际上就是获取的阴影颜色
half3 attenuatedLightColor = mainLight.color * (mainLight.distanceAttenuation * mainLight.shadowAttenuation);
```

概要：

```
// 1、添加变体
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS
#pragma multi_compile _ _SHADOWS_SOFT

// 2、Varyings 结构体中添加
float4 shadowCoord : TEXCOORD4;

// 3、顶点着色器
float3 positionWS = TransformObjectToWorld(v.positionOS.xyz);
// 通过世界坐标获取阴影坐标位置
o.shadowCoord = TransformWorldToShadowCoord(positionWS);

// 4、片段着色器
Light mainLight = GetMainLight(i.shadowCoord);
c *= mainLight.shadowAttenuation;
```

代码：

```
Pass
        {
            Name "Unlit"
            HLSLPROGRAM
            // Required to compile gles 2.0 with standard srp library
 #pragma prefer_hlslcc gles
 #pragma exclude_renderers d3d11_9x
 #pragma vertex vert
 #pragma fragment frag
 #pragma multi_compile_fog
 #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"
 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
 #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/UnityInstancing.hlsl"

             //接收阴影 URP
 #pragma multi_compile _ _MAIN_LIGHT_SHADOWS
 #pragma multi_compile _ _SHADOWS_SOFT

            struct Attributes
            {
            	float4 positionOS       : POSITION;
                float2 uv               : TEXCOORD0;
            };

            struct Varyings
            {
                float4 positionCS       : SV_POSITION;
                float2 uv               : TEXCOORD0;
                float fogCoord          : TEXCOORD1;
                // 接收的阴影坐标 URP
                float4 shadowCoord : TEXCOORD4;
            };

            CBUFFER_START(UnityPerMaterial)
            half4 _BaseColor;
            float4 _BaseMap_ST;
            CBUFFER_END
            TEXTURE2D (_BaseMap);SAMPLER(sampler_BaseMap);

            Varyings vert(Attributes v)
            {
                Varyings o = (Varyings)0;

                o.positionCS = TransformObjectToHClip(v.positionOS.xyz);
                o.uv = TRANSFORM_TEX(v.uv, _BaseMap);
                o.fogCoord = ComputeFogFactor(o.positionCS.z);
                float3 positionWS = TransformObjectToWorld(v.positionOS.xyz);
                  // 通过世界坐标获取阴影坐标位置
                o.shadowCoord = TransformWorldToShadowCoord(positionWS);
                return o;
            }

            half4 frag(Varyings i) : SV_Target
            {
                half4 c;
                half4 baseMap = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, i.uv);
                c = baseMap * _BaseColor;
                c.rgb = MixFog(c.rgb, i.fogCoord);
				// 接收的阴影
                Light mainLight = GetMainLight(i.shadowCoord);
                c *= mainLight.shadowAttenuation;
                return c;
            }
            ENDHLSL
        }
```

*   阴影的生成

继续来到 Lit.shader 中，找到一个阴影的 Pass

```
Pass
        {
            Name "ShadowCaster"
            Tags{"LightMode" = "ShadowCaster"}

            ZWrite On
            ZTest LEqual
            ColorMask 0
            Cull[_Cull]

            HLSLPROGRAM
            #pragma exclude_renderers gles gles3 glcore
            #pragma target 4.5

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature_local_fragment _ALPHATEST_ON
            #pragma shader_feature_local_fragment _SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing
            #pragma multi_compile _ DOTS_INSTANCING_ON

            #pragma vertex ShadowPassVertex
            #pragma fragment ShadowPassFragment

            #include "Packages/com.unity.render-pipelines.universal/Shaders/LitInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/ShadowCasterPass.hlsl"
            ENDHLSL
        }
```

道理上直接将这个 Pass 扣过来用就好了。本着学习精神，我们继续往下翻，打开 ShadowCasterPass.hlsl，发现里面的内容其实并不多。

直接贴代码：

```
Pass
        {
            Name "ShadowCaster"
            Tags{"LightMode" = "ShadowCaster"}

            ZWrite On
            ZTest LEqual
            //Cull[_Cull]

            HLSLPROGRAM
            // Required to compile gles 2.0 with standard srp library
            // #pragma prefer_hlslcc gles
            // #pragma exclude_renderers d3d11_9x
            #pragma target 2.0

            // GPU Instancing
            #pragma multi_compile_instancing

            #pragma vertex vert
            #pragma fragment frag

            struct appdata
            {
                float4 positionOS   : POSITION;
                float3 normalOS     : NORMAL;
                float2 texcoord     : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
                float2 uv           : TEXCOORD0;
                float4 positionCS   : SV_POSITION;
            };


            v2f vert(appdata input)
            {
                v2f output;
                UNITY_SETUP_INSTANCE_ID(input);

                output.uv = TRANSFORM_TEX(input.texcoord, _BaseMap);
                //本来如果正常 转到齐次裁剪空间使用 TransformObjectToHClip 的
                //output.positionCS = TransformObjectToHClip(input.positionOS.xyz);
                // 但是发现 源码里面是用了 GetShadowPositionHClip ，里面做了其他运算
                // GetShadowPositionHClip 这个不加就会有摩尔纹(锯齿) 就是看到各种帖子抄来抄去说不太对，就是因为没有用源码里的
                output.positionCS = GetShadowPositionHClip(input);
                return output;
            }

            half4 frag(v2f input) : SV_TARGET
            {
                // 感觉这里其实是可以不要的
                //Alpha(SampleAlbedoAlpha(input.uv, TEXTURE2D_ARGS(_BaseMap, sampler_BaseMap)).a, _BaseColor, _Cutoff);
                return 0;
            }
            ENDHLSL
        }
```

补充 Alpha(SampleAlbedoAlpha.... 相关内容

搜了 URP 的源码可以找到都在 SurfaceInput.hlsl 内

```
half Alpha(half albedoAlpha, half4 color, half cutoff) {
#if !defined(_SMOOTHNESS_TEXTURE_ALBEDO_CHANNEL_A) && !defined(_GLOSSINESS_FROM_BASE_ALPHA)
    half alpha = albedoAlpha * color.a;
#else
    half alpha = color.a;
#endif

#if defined(_ALPHATEST_ON)
    clip(alpha - cutoff);
#endif

    return alpha;
}

half4 SampleAlbedoAlpha(float2 uv, TEXTURE2D_PARAM(albedoAlphaMap, sampler_albedoAlphaMap)) {
    return SAMPLE_TEXTURE2D(albedoAlphaMap, sampler_albedoAlphaMap, uv);
}
```

有透明度需求的这里会做剔除的处理