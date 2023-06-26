    本次我们需要使用光照的，故需要引用光照库函数，如下。

#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

打开该库函数 lighting.hlsl，我们可以查阅到此次我们应该使用到的结构体和函数。

![[2746c03555c046b62449b10b8c752c7e_MD5.webp]]

灯光结构体

![[c939c05f4c7de657ee2b0c9a99543e7e_MD5.webp]]

获取主光源的函数

新的变量类型 real，定义在 commom.hlsl 库里，根据不同的平台编译 float 或 half。

法线的空间变换，从模型空间变化到世界空间的法线变换函数，定义在 spaceTransforms.hlsl 库函数里，其中第二个变量让我们确定是否将向量标准化，一般填 true。

![[b38e31afed50fe97812ec12532d1ba6c_MD5.webp]]

O2W 的法线变换函数定义

片元着色器里，先定义一个光照类型的结构体，并使用 GetMainLight() 函数把返回值给他，然后我们就可以获得主光源的颜色，方向，距离衰减，阴影衰减。以下为 shader 源码。其中为兰伯特光照模型，若需要改成半兰伯特，则 LightAten 的结果 * 0.5+0.5 即可，距离衰减未乘因为直射光无距离衰减，阴影衰减在后面的文章牵扯到阴影再使用。

Shader "URP / 兰伯特"

{

    Properties

    {

        _MainTex("MainTex",2D)="White"{}

        _BaseColor("BaseColor",Color)=(1,1,1,1)

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

        "RenderType"="Opaque"

        }

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float4 _MainTex_ST;

        half4 _BaseColor;

        CBUFFER_END

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

         struct a2v

         {

             float4 positionOS:POSITION;

             float4 normalOS:NORMAL;

             float2 texcoord:TEXCOORD;

         };

         struct v2f

         {

             float4 positionCS:SV_POSITION;

             float2 texcoord:TEXCOORD;

             float3 normalWS:TEXCOORD1;

         };

        ENDHLSL

        pass

        {

        Tags{

        "LightMode"="UniversalForward"

        }

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);

                o.normalWS=TransformObjectToWorldNormal(i.normalOS.xyz,true);

                return o;

            }

            real4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord)*_BaseColor;

                Light mylight=GetMainLight();

                real4 LightColor=real4(mylight.color,1);

                float3 LightDir=normalize(mylight.direction);

                float LightAten=dot(LightDir,i.normalWS);

                return tex*LightAten*LightColor;

                 //return tex*LightAten*LightColor*0.5+0.5;// 半兰伯特光照模型

            }

            ENDHLSL

        }

    }

}