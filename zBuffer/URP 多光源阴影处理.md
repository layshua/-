

## 思路

1.  阴影分为接收阴影和投射阴影两个部分，接收阴影开启部分关键字就可以了，但投射阴影需要写一个用与投射阴影的 Pass，该 Pass 的渲染路径采用 **ShadowCaster**
2.  第一个 Pass，开启一系列相关关键字，用于接收阴影
3.  然后在片元着色器中使用 **TransformWorldToShadowCoord** 函数获取阴影纹理坐标，使用该坐标获取主光源，再进行一系列光源计算
4.  第二个 Pass 参考 URP 的 `Universal Render Pipeline/Lit/ShadowCasterPass` 创建生成阴影的 Pass，主要任务在于将阴影从对象空间中转换到裁剪空间。

## 注意事项

**点光源不支持实时阴影，如果想实现点光源阴影应该采用光照烘焙，点光源和聚光灯都不支持间接反射阴影。**

![[443689b47fbc242d734db654b28b7f5c_MD5.png]]


想要副光源也能够产生阴影，需要先在 URP 管线的 Inspector 面板下为 Additional Lights 勾选上 Cast Shadows。

![[2529b3381c5682aad1576cd316d99654_MD5.png]]


然后在光源的 Inspector 面板中，将 Shadow Type 切换成 Soft Shadows


![[d9f48dcbd3888263097501ac8b7d002e_MD5.png]]


场景中背面搭建的 Plane 处于逆光状态，想要该 Plane 也产生阴影，需要将其 Mesh Rederer 组件下的 Cast Shadows 设置成 Two Sided。


![[4fc0f849946cf53e46af2565685e4ba6_MD5.png]]


## 使用到的语法

*   一些列用于接收阴影的关键字。

```c
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS
#pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE
#pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
#pragma multi_compile _ _SHADOWS_SOFT
```

*   传入片元的世界位置，获取该位置下的阴影纹理坐标，后续可以使用该坐标获取到主光源（Light），与之前不同的是该光源信息中 **shadowAttenuation** 阴影衰减不在为 1，方便后续的阴影计算
    
    `TransformWorldToShadowCoord(positionWS);`
    
*   表明该 Pass 采用阴影渲染模式
    
    `Tags { "LightMode" = "ShadowCaster" }`
    
*   用于获取应用阴影的深度偏移后的阴影坐标
    
    `ApplyShadowBias（positionWS, normalWS, _LightDirection）`
    
*   通过关键字判断法线归一化的计算方式
    
    `o.normalWS = NormalizeNormalPerVertex(normalInput.normalWS);`
    
```c
//"Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
real3 NormalizeNormalPerVertex(real3 normalWS) 
{
#if defined(SHADER_QUALITY_LOW) && defined(_NORMALMAP)
    return normalWS;
#else
    return normalize(normalWS);
#endif
}
```
    

## 完整代码

```c
Shader "URP/MultiLightShadow"
{
    Properties
    {
        _Diffuse ("Diffuse", Color) = (1, 1, 1, 1)
        _Specular ("Specular", Color) = (1, 1, 1, 1)
        _Gloss ("Gloss", Range(8.0, 256)) = 20
        [Toggle(_AdditionalLights)] _AddLights ("AddLights", Float) = 1
    }
    SubShader
    {
        Tags { "RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" }
        
        HLSLINCLUDE
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
        
        CBUFFER_START(UnityPerMaterial)
        float4 _Diffuse;
        float4 _Specular;
        float _Gloss;
        CBUFFER_END
        ENDHLSL
        
        Pass
        {
            Tags { "LightMode" = "UniversalForward" }
            
            HLSLPROGRAM
            
            // 设置关键字
            #pragma shader_feature _AdditionalLights
            
            // 接收阴影所需关键字
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE
            #pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS
            #pragma multi_compile _ _SHADOWS_SOFT
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

      
            struct Attributes
            {
                float4 positionOS: POSITION;
                float3 normalOS: NORMAL;
                float4 tangentOS: TANGENT;
            };
            
            struct Varyings
            {
                float4 positionCS: SV_POSITION;
                float3 positionWS: TEXCOORD0;
                float3 normalWS: TEXCOORD1;
                float3 viewDirWS: TEXCOORD2;
            };


            Varyings vert(Attributes v)
            {
                Varyings o;
                // 获取不同空间下坐标信息
                VertexPositionInputs positionInputs = GetVertexPositionInputs(v.positionOS.xyz);
                o.positionCS = positionInputs.positionCS;
                o.positionWS = positionInputs.positionWS;
                
                // 获取世界空间下法线相关向量
                VertexNormalInputs normalInput = GetVertexNormalInputs(v.normalOS, v.tangentOS);
                
                o.normalWS = NormalizeNormalPerVertex(normalInput.normalWS);
                o.viewDirWS = GetCameraPositionWS() - positionInputs.positionWS;
                
                return o;
            }
            
            /// lightColor：光源颜色
            /// lightDirectionWS：世界空间下光线方向
            /// lightAttenuation：光照衰减
            /// normalWS：世界空间下法线
            /// viewDirectionWS：世界空间下视角方向
            half3 LightingBased(half3 lightColor, half3 lightDirectionWS, half lightAttenuation, half3 normalWS, half3 viewDirectionWS)
            {
                // 兰伯特漫反射计算
                half NdotL = saturate(dot(normalWS, lightDirectionWS));
                half3 radiance = lightColor * (lightAttenuation * NdotL) * _Diffuse.rgb;
                // BlinnPhong高光反射
                half3 halfDir = normalize(lightDirectionWS + viewDirectionWS);
                half3 specular = lightColor * pow(saturate(dot(normalWS, halfDir)), _Gloss) * _Specular.rgb;
                
                return radiance + specular;
            }
            
            half3 LightingBased(Light light, half3 normalWS, half3 viewDirectionWS)
            {
                // 注意light.distanceAttenuation * light.shadowAttenuation，这里已经将距离衰减与阴影衰减进行了计算
                return LightingBased(light.color, light.direction, light.distanceAttenuation * light.shadowAttenuation, normalWS, viewDirectionWS);
            }
            
            half4 frag(Varyings i): SV_Target
            {
                half3 normalWS = NormalizeNormalPerPixel(i.viewDirWS);
                half3 viewDirWS = SafeNormalize(i.normalWS);
                
                  // 获取阴影坐标
                float4 shadowCoord = TransformWorldToShadowCoord(i.positionWS.xyz);
    
                // 使用HLSL的函数获取主光源数据
                Light mainLight = GetMainLight(shadowCoord);
                half3 diffuse = LightingBased(mainLight, normalWS, viewDirWS);
                
                // 计算其他光源
                #ifdef _AdditionalLights
                    uint pixelLightCount = GetAdditionalLightsCount();
                    for (uint lightIndex = 0u; lightIndex < pixelLightCount; ++ lightIndex)
                    {
                        // 获取其他光源
                        Light light = GetAdditionalLight(lightIndex, i.positionWS);
                        diffuse += LightingBased(light, normalWS, viewDirWS);
                    }
                #endif
                
                half3 ambient = SampleSH(normalWS);
                return half4(ambient + diffuse, 1.0);
            }
            
            ENDHLSL
            
        }
        
        //下面计算阴影的Pass可以直接通过使用URP内置的Pass计算
        UsePass "Universal Render Pipeline/Lit/ShadowCaster"
        
        // or
        // 计算阴影的Pass
        Pass
        {
            Name "ShadowCaster"
            Tags { "LightMode" = "ShadowCaster" }
            Cull Off
            ZWrite On
            ZTest LEqual
            
            HLSLPROGRAM
            
            // 设置关键字
            #pragma shader_feature _ALPHATEST_ON
            
            #pragma vertex vert
            #pragma fragment frag
            
            #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/CommonMaterial.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"
            
            float3 _LightDirection;
            
            struct Attributes
            {
                float4 positionOS: POSITION;
                float3 normalOS: NORMAL;
            };
            
            struct Varyings
            {
                float4 positionCS: SV_POSITION;
            };            
            
            // 获取裁剪空间下的阴影坐标
            float4 GetShadowPositionHClips(Attributes input)
            {
                float3 positionWS = TransformObjectToWorld(input.positionOS.xyz);
                float3 normalWS = TransformObjectToWorldNormal(input.normalOS);
                // 获取阴影专用裁剪空间下的坐标
                float4 positionCS = TransformWorldToHClip(ApplyShadowBias(positionWS, normalWS, _LightDirection));
                
                // 判断是否是在DirectX平台翻转过坐标
                #if UNITY_REVERSED_Z
                    positionCS.z = min(positionCS.z, positionCS.w * UNITY_NEAR_CLIP_VALUE);
                #else
                    positionCS.z = max(positionCS.z, positionCS.w * UNITY_NEAR_CLIP_VALUE);
                #endif
                
                return positionCS;
            }
            
            Varyings vert(Attributes input)
            {
                Varyings output;
                output.positionCS = GetShadowPositionHClips(input);
                return output;
            }

       
            half4 frag(Varyings input): SV_TARGET
            {
                return 0;
            }
            
            ENDHLSL
            
        }
    }
    FallBack "Packages/com.unity.render-pipelines.universal/FallbackError"
}
```