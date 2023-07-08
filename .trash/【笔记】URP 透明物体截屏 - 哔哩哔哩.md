*   **前言**


想做一个物体扭曲的效果，但是 URP 管线里是没有 Grab Pass 这个功能的，而_CameraColorTexture 和_CameraOpaqueTexture 是不包含透明物体的 (应该包含裁切物体)，所以做出来的扭曲效果就忽略了透明物体。一般情况无所谓，但是当特效比较多的时候，就有点麻烦了。  

解决方法就是，当当当！后处理截图。

好久没写这么简单又实用的东西了，真开心。

```
TEXTURE2D(_AfterPostProcessTexture);
SAMPLER(sampler_AfterPostProcessTexture);
```


参考教程：[URP | 热扭曲效果](https://www.bilibili.com/read/cv15512651)

*   **开启后处理截图**


正常情况下，是没有后处理截图的。  

但是，当我们开启了 Overlay 模式的摄像机的时候，Unity 会自动帮我们拿到_AfterPostProcessTexture 这张截图 (Overlay 模式的摄像机就是在前一个摄像机的基础上，进行渲染)

![[926a52150dd578be3da6d95d65f409ca_MD5.webp]]

    那么 Overlay 摄像机怎么设置呢？

    随便创建一个新的摄像机，然后记得关掉不必要的功能。  

![[471d3d2ebe53fa5556d7af640eabd3d6_MD5.webp]]

    然后在主摄像机的 Stack 下添加刚才创建的摄像机，就完成了 Overlay 摄像机的设置。

![[9edb98fec29a09288b377340531e3c9f_MD5.webp]]

*   **Render Object**


    然后我们需要在拿到后处理截图之后再对扭曲物体进行渲染。  

    这里就需要用到 Render Object 这个功能。

    这里特别简单，就是找到 URP 的管线配置，一般默认是 ForwardRenderer，然后新建 RenderObject。

![[a4796502fc603594c2e227538746da8a_MD5.webp]]

    配置看下这里就好了，关于这个配置的解释就是，把 Shader 中 LightMode 为 Distortion 的 Pass 的渲染顺序调整到 AfterRenderingPostProcess。

    如果 Overrides 里加了材质，那么在原来渲染了一遍的基础上，再用新的材质渲染一遍。

    当然这里我们就不用加 Override Material 了。  

![[ac52e34d213ad932c51e53756def023d_MD5.webp]]

    好，现在，我们在 Shader 中添加 LightMode=Distortion 这个标签，就可以在后处理之后渲染物体了。  

    完整 Shader 如下：

```
Shader "Effects/Ult_Distortion"
{
Properties
{
    _XDistortionIntensity("X Distort Intensity", float) = 1.0
    _YDistortionIntensity("Y Distort Intensity", float) = 1.0
    [Enum(UnityEngine.Rendering.CompareFunction)]_ZTest("Z Test", float) = 4.0
    [Enum(Off, 0, Front, 1, Back, 2)]_Cull("Cull", float) = 2.0
}
SubShader
{
    Tags
    {
        "RenderType"="Transparent"
        "Queue"="Transparent"
        "RenderPipeline"="UniversalRenderPipeline"
    }

    Pass
    {
        Tags{ "LightMode"="Distortion" }

        ZTest [_ZTest]
        Cull [_Cull]
        ZWrite Off

        HLSLPROGRAM

        #pragma vertex vert
        #pragma fragment frag

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)
            float _XDistortionIntensity;
            float _YDistortionIntensity;
        CBUFFER_END

        TEXTURE2D(_AfterPostProcessTexture);
        SAMPLER(sampler_AfterPostProcessTexture);

        struct a2v
        {
            float4 positionOS : POSITION;
            float2 uv : TEXCOORD0;
        };

        struct v2f
        {
            float4 positionCS : SV_POSITION;
            float2 uv : TEXCOORD0;
            float4 scrPos: TEXCOORD1;
        };


        v2f vert (a2v v)
        {
            v2f o;

            o.positionCS = TransformObjectToHClip(v.positionOS);
            o.scrPos = ComputeScreenPos(o.positionCS);
            o.uv = v.uv;
            #if UNITY_UV_STARTS_AT_TOP
                o.uv.y = 1 - o.uv.y;
            #endif

            return o;
        }

        real4 frag (v2f i) : SV_Target
        {
            float2 uvScr = i.scrPos.xy/i.positionCS.w + float2(_XDistortionIntensity,_YDistortionIntensity);
            real4 col = SAMPLE_TEXTURE2D(_AfterPostProcessTexture, sampler_AfterPostProcessTexture, uvScr);
            return col;
        }
        ENDHLSL
    }
}
}
```