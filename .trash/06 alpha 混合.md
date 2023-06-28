![[daab21f92557f1de514210760dd424db_MD5.webp]]

alpha blend

在使用透明度混合的时候，tags 里需设置渲染类型为 transparent，渲染队列为 transparent，忽略透射可开可关视情况。

![[b62618c94fdce6605cdd9f4f7136d93e_MD5.webp]]

这里，我使用一张 mask 图取它的 r 通道作为 a 去混合，它和 maintex 一样进行 uv 采样。

![[a19b8edf0f71fc16f64fb072970a2611_MD5.webp]]

mask 图作为 a 混合

pass 里，设置好混合模式，当前颜色的混合因子为 SrcAlpha，而颜色缓存区的混合因子设为 OneMinusSrcAlpha。然后关掉深度写入。

![[1817ef79ef1a706b69bbe166366665f2_MD5.webp]]

pass 里设置渲染状态

片元着色器里，使用 maintex 的 rgb 通道作为当前颜色，mask 图的 r 通道作为混合因子。最终即可得到我们的效果。

![[4985ab941b57c03d6964933fc32814ac_MD5.webp]]

片元着色器

最后，再附上 shader 的源码
```c
Shader "WX/URP/alpha blend"

{

    Properties

    {

        _MainTex("MainTex",2D)="white"{}

        _BaseColor("BaseColor",Color)=(1,1,1,1)

        _AlphaTex("AlphaTex",2D)="white"{}

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

         "IgnoreProjector"="True"

         "RenderType"="Transparent"

         "Queue"="Transparent"

        }

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float4 _MainTex_ST;

        half4 _BaseColor;

        float4 _AlphaTex_ST;

        CBUFFER_END

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

        TEXTURE2D(_AlphaTex);

        SAMPLER(sampler_AlphaTex);

         struct a2v

         {

             float4 positionOS:POSITION;

             float4 normalOS:NORMAL;

             float2 texcoord:TEXCOORD;

         };

         struct v2f

         {

             float4 positionCS:SV_POSITION;

             float4 texcoord:TEXCOORD;

         };

        ENDHLSL

        pass

        {

        Tags{

        "LightMode"="UniversalForward"

        }

        Blend SrcAlpha OneMinusSrcAlpha

        ZWrite Off 

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord.xy=TRANSFORM_TEX(i.texcoord,_MainTex);

                o.texcoord.zw=TRANSFORM_TEX(i.texcoord,_AlphaTex);

                return o;

            }

            real4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord.xy)*_BaseColor;

                float alpha=SAMPLE_TEXTURE2D(_AlphaTex,sampler_AlphaTex,i.texcoord.zw).x;

                return real4(tex.xyz,alpha);

            }

            ENDHLSL

        }

    }

}
```