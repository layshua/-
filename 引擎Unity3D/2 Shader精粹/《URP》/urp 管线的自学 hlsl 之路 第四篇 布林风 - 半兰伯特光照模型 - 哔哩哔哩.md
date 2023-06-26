*       在第三篇的基础上，增加了布林风的高光光照模型，核心公式为 spe=pow（dot（H,N），_gloss）, 其中 H=normalize（normalize（ViewDir）+normalize（LightDir）），_gloss 为高光系数，以上纯凑字数，我不扯公式了，毕竟是书上的重复内容，该篇内容只说明 urp 的 hlsl 要注意的地方。
    
*   获取模型顶点的世界坐标，使用 TransformObjectToWorld(i.positionOS.xyz) 函数即可，该函数在 SpaceTransform 里定义。
    
*   获取相机的世界坐标_WorldSpaceCameraPos，该变量为内置宏。与上面的顶点世界坐标相减在规范化后即可得到相机方向。
    
    获取灯光的世界方向，通过 GetMainLight() 函数得到灯光的结构体，灯光方向为其属性之一。
    

利用布林风高光公式计算高光，在利用半兰伯特光照公式计算漫反射（见第三篇），两者相加，即可得到最终的输出。shader 源码如下

Shader "Unlit / 布林风高光半兰伯特光照模型"

{

    Properties

    {

        _MainTex ("Texture", 2D) = "white" {}

        _BaseColor("Color",Color)=(1,1,1,1)

        _SpecularRange("SpecularRange",Range(10,300))=10

        _SpecularColor("SpecularColor",Color)=(1,1,1,1)

    }

    SubShader

    {

        Tags { "RenderType"="Opaque" 

        "RenderPipeline"="UniversalRenderPipeline"}

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float4 _MainTex_ST;

        real4 _BaseColor;

        float _SpecularRange;

        real4 _SpecularColor;

        CBUFFER_END

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

        struct  a2v

        {

            float4 positionOS:POSITION ;

            float3 normalOS:NORMAL;

            float2 texcoord:TEXCOORD0;

        } ;

        struct v2f

        {

            float4 positionCS:SV_POSITION;

            float3 normalWS:NORMAL;

            float3 viewDirWS:TEXCOORD0 ; 

            float2 texcoord:TEXCOORD1  ;

        };

        ENDHLSL

        Pass

        {NAME"MainPass"

            Tags{

                "LightMode"="UniversalForward"

            }

            HLSLPROGRAM

            #pragma vertex vert1

            #pragma fragment  frag1

            v2f vert1(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.normalWS=TransformObjectToWorldNormal(i.normalOS,true);

                o.viewDirWS=normalize(_WorldSpaceCameraPos.xyz-TransformObjectToWorld(i.positionOS.xyz));// 得到世界空间的视图方向

                o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);

                return  o;

            } 

            real4 frag1(v2f i):SV_TARGET

            {

                Light mylight=GetMainLight();

                float3 LightDirWS=normalize(mylight.direction);

                float spe=dot(normalize(LightDirWS+i.viewDirWS),i.normalWS);

                real4 specolor=pow(spe,_SpecularRange)*_SpecularColor;

                real4 texcolor=(dot(i.normalWS,LightDirWS)*0.5+0.5)*SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord)*_BaseColor;

                texcolor*=real4(mylight.color,1);

                return specolor+texcolor;

            }

            ENDHLSL

        }

    }

}