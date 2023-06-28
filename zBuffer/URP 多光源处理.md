
## 思路

1.  使用 **GetMainLight** 函数获取主光源，然后进行漫反射与高光反射计算
2.  使用 **GetAdditionalLightsCount** 函数获取副光源个数，在一个 For 循环中使用 **GetAdditionalLight** 函数获取其他的副光源，进行漫反射与高光反射计算，叠加到光源结果上

在 URP 中，光照的计算不再像 Built-in 管线那样死板，全部由 Unity 的光照路径决定；现如今是由 URP 管线的脚本收集好场景中所有的光照信息，再传输给 Pass，由我们开发者在 Pass 中决定采用那些光源进行光照计算。

## 使用到的语法

*   用于声明属性为开关的标签，该开关可以在材质的 Inspector 面板中看到，用与开启和关闭多光源的计算
    
    `[Toggle(_AdditionalLights)]`
    
*   一个关键字，用于后续代码中的宏校验，注意后面的名称对应了属性中的标签名
    
    `#pragma shader_feature _AdditionalLights`
    
*   用于获取主光源，返回 Light。
    
    `GetMainLight();`
    
*   用于获取场景中除主光源外的副光源个数。
    
    `GetAdditionalLightsCount();`
    
    副光源个数是有限制的，最大不能超过 URP 管线中的灯光限制（管线的 Inspector 界面：Lighing->PerObjectLimit），超过部分不会纳入计算。
    
*   通过传入副光源索引和片元的世界坐标，返回一个该片元的副光源参数（Light）。
    
    `GetAdditionalLight(lightIndex,positionWS);`
    
*   使用球谐函数计算环境光
    
    `alf3 ambient = SampleSH(normalWS);`
    

## 完整代码

```c
Shader "URP/MultiLight"
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
            
            #pragma shader_feature _AdditionalLights
            
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
                half3 normalWS = NormalizeNormalPerPixel(i.normalWS);
                half3 viewDirWS = SafeNormalize(i.viewDirWS);
                
                // 使用HLSL的函数获取主光源数据
                Light mainLight = GetMainLight();
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
                
                // 采用球谐光照计算环境光
                half3 ambient = SampleSH(normalWS);
                return half4(ambient + diffuse, 1.0);
            }
            
            ENDHLSL
            
        }
    }
    FallBack "Packages/com.unity.render-pipelines.universal/FallbackError"
}
```