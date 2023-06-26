![[818311b576f8532954245a5ace46b288_MD5.webp]]

主光源阴影透射和接受

      阴影这块官方改了一些东西，和 build in 的实现方式不一样，翻了官方文档和网上的一些资料，研究出来了怎么实现阴影的接受和投射，这里分开讲一下，这篇只涉及主光源的阴影内容，额外多光源的阴影下一篇再讲（还没研究出来）。

     顶点着色器和之前的一样无变化，而片元着色器里，这里使用 TransformWorldToShadowCoord(i.positionWS) 函数，该函数定义在 Shadow.hlsl 里，作用是把模型的世界空间顶点坐标输入，得到阴影坐标，用于在 shadowmap 下进行比较。

![[81a90680769e2050b28aea3f296694cd_MD5.webp]]

TransformWorldToShadowCoord 函数定义

    我们注意到该函数里是要定义关键字_MAIN_LIGHT_SHADOWS_CASCADE，这样才能得到正确的阴影坐标。

    第二个函数还是 GetMainLight（float4 shadowcoord），该函数是之前用的 GetMainLight 的重载形式，它会调用另外一个函数 half MainLightRealtimeShadow(float4 shadowCoord)，该函数定义在 Shadow.hlsl 下，它的定义如下。

![[3fb58cc1064590e16dd3b3cb880cffb1_MD5.webp]]

MainLightRealtimeShadow 函数定义

    该函数是专门用来计算阴影衰减的，使用它时，需声明关键字 MAIN_LIGHT_CALCULATE_SHADOWS，但是这里直接定义该关键字是不会生效的，猜测还需要额外的关键字，找到了_MAIN_LIGHT_SHADOWS，定义它时则定义了 calculate shadow，猜测它还定义了其他的内容，这里我们直接使用_MAIN_LIGHT_SHADOWS 关键字即可，如有大佬明白其中原理希望评论指出。

![[8327d7983d0df9ba302b099690425f7c_MD5.webp]]

_MAIN_LIGHT_SHADOWS 关键字

    下面来说明一下代码。以下为上面两个函数需要的关键字定义，还额外多了个_SHADOWS_SOFT，该关键字是用来设置软阴影的（但是也会造成性能的降低），请酌情使用。

![[8ac630b5c8a76cde5174fb46adab5511_MD5.webp]]

关键字编译多个变体 shader

     计算带阴影衰减的主光源。

![[ba94b02c7ef9f7437b85f56c8c62ae4f_MD5.webp]]

计算主光源

     剩下的就常规的计算高光和半兰伯特。

![[a4db55b3c95c2d36e59015f9ae6e4f08_MD5.webp]]

片元的剩余部分

    这样，接受阴影就完成了，如果想实现阴影的投射，直接使用 urp 自带的 shadowcaster 的 pass 把模型的信息写到 shadowmap 上去让其他模型接受本模型的阴影。最后，附上 shader 源码：

Shader "URP/MainLightShadow"

{

    Properties

    {

        _MainTex("MainTex",2D)="white"{}

        _BaseColor("BaseColor",Color)=(1,1,1,1)

        _Gloss("gloss",Range(10,300))=20

        _SpecularColor("SpecularColor",Color)=(1,1,1,1)

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

        half _Gloss;

        real4 _SpecularColor;

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

             float3 positionWS:TEXCOORD1; 

             float3 normalWS:NORMAL;

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

            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS

            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE

            #pragma multi_compile _ _SHADOWS_SOFT// 柔化阴影，得到软阴影

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);

                o.positionWS=TransformObjectToWorld(i.positionOS.xyz);

                o.normalWS=TransformObjectToWorldNormal(i.normalOS);

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord)*_BaseColor;

                Light mylight=GetMainLight(TransformWorldToShadowCoord(i.positionWS));

                float3 WS_L=normalize(mylight.direction);

                float3 WS_N=normalize(i.normalWS);

                float3 WS_V=normalize(_WorldSpaceCameraPos-i.positionWS);

                float3 WS_H=normalize(WS_V+WS_L);

                tex*=(dot(WS_L,WS_N)*0.5+0.5)*mylight.shadowAttenuation*real4(mylight.color,1);

                float4 Specular =pow(max(dot(WS_N,WS_H),0) ,_Gloss)*_SpecularColor*mylight.shadowAttenuation;

                return tex+Specular  ;

            }

            ENDHLSL

        }

        UsePass "Universal Render Pipeline/Lit/ShadowCaster"

    }

}