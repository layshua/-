*    HLSL 和 CG 的大部分语法的内容是一致的，对于重复的内容，本人不在重述。
    
*    管线说明 tags 标签，URP 的 shader 都需标明使用的渲染管线的标签，即 "RenderPipeline"="UniversalRenderPipeline"。
    
*    CG 的引入变成了 HLSL 的引入，即 CGINCLUDE ENDCG 变成了 HLSLINCLUDE  ENDHLSL。CG 的编码，从 CGPROGRAM ENDCG 变成了 HLSL 的 HLSLPROGRAM ENDHLSL.
    
*    CBUFFER_START 和 CBUFFER_END, 对于变量是单个材质独有的时候建议放在这里面，以提高性能。CBUFFER(常量缓冲区) 的空间较小，不适合存放纹理贴图这种大量数据的数据类型，适合存放 float，half 之类的不占空间的数据，关于它的官方文档在下有详细说明。https://blogs.unity3d.com/2019/02/28/srp-batcher-speed-up-your-rendering/
    
*    新的 DXD11 HLSL 贴图的采样函数和采样器函数，TEXTURE2D (_MainTex) 和 SAMPLER(sampler_MainTex)，用来定义采样贴图和采样状态代替原来 DXD9 的 sampler2D，在不同的平台有不同的变化，在 GLcore 库函数里定义，具体的原理未知，如有大佬了解望给予评论和指出。
    
*   贴图的采用输出函数采用 DXD11 HLSL 下的 SAMPLE_TEXTURE2D(textureName, samplerName, coord2) ，具有三个变量，分别是 TEXTURE2D (_MainTex) 的变量和 SAMPLER(sampler_MainTex) 的变量和 uv，用来代替原本 DXD9 的 TEX2D(_MainTex,texcoord)。
    
*   UnityCG，cginc 已经不再适用，取而代之的是 Core.hlsl，在引用时注明 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl" 即可。
    
*   下面附 shader 源码
    
*   Shader "URP/URPUnlitShader"
    
*   {
    
*       Properties
    
*       {
    
*           _MainTex("MainTex",2D)="White"{}
    
*           _BaseColor("BaseColor",Color)=(1,1,1,1)
    
*       }
    
*       SubShader
    
*       {
    
*           Tags{
    
*           "RenderPipeline"="UniversalRenderPipeline"
    
*           "RenderType"="Opaque"
    
*           }
    
*           HLSLINCLUDE
    
*           #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
    
*           CBUFFER_START(UnityPerMaterial)
    
*           float4 _MainTex_ST;
    
*           half4 _BaseColor;
    
*           CBUFFER_END
    
*           TEXTURE2D (_MainTex);
    
*           SAMPLER(sampler_MainTex);
    
*            struct a2v
    
*            {
    
*                float4 positionOS:POSITION;
    
*                float4 normalOS:NORMAL;
    
*                float2 texcoord:TEXCOORD;
    
*            };
    
*            struct v2f
    
*            {
    
*                float4 positionCS:SV_POSITION;
    
*                float2 texcoord:TEXCOORD;
    
*            };
    
*           ENDHLSL
    

*           pass
    
*           {
    
*               HLSLPROGRAM
    
*               #pragma vertex VERT
    
*               #pragma fragment FRAG
    
*               v2f VERT(a2v i)
    
*               {
    
*                   v2f o;
    
*                   o.positionCS=TransformObjectToHClip(i.positionOS.xyz);
    
*                   o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);
    
*                   return o;
    
*               }
    
*               half4 FRAG(v2f i):SV_TARGET
    
*               {
    
*                   half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord)*_BaseColor;
    
*                   return tex;
    
*               }
    
*               ENDHLSL
    
*           }
    

*       }
    

*   }