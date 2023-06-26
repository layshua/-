![[a3c6f88b0c1992be984bc8e390e25d4c_MD5.webp]]

效果图

    单纯的 alpha test 比较简单，所以加上一些其他的东西来做些小特效（才不是纯粹想凑字数呢），这里制作了一个简单的溶解的效果，不过由于是使用的 alpha test 制作，比 alphablend 还稍微麻烦一丢丢，但是在移动端 alphablend 的性能还可能好一些些，alpha test 危。不多说了，开始分析代码和注意事项。

![[98844dd3b6db427e853141981c4a600a_MD5.webp]]

参数栏

    _MainTex 为主遮罩图，我们用他的 r 通道即可；_Cutoff，要使用 alphatest 的关键字，不能改不能打错 o 是小写的 o，不是_CutOff；_BurnColor，即我们想要的灼烧色。

![[6e64c43ea5c218c0f6289d8cbf67da93_MD5.webp]]

标签

    标签要声明一个渲染类型和渲染队列，你们都懂。a2v，v2f，顶点着色器都不多说了，下面说明一下 clip。

![[d91021958578efe69ca5647c66dbfaba_MD5.webp]]

片元着色器

    clip 大家都知道是对参数里低于 0 的部分舍弃掉，这里直接 tex.r-_Cutoff 即可；但是我并未这样做，而是使用 step 函数，使用 step 函数是为了更好得配合下面代码的 lerp 函数，但是这样得到的值在 0-1 之间，值为 0 的部分并不会被 clip 舍弃，所以我们需要减去 0.01，这样让原来为 0 的部分变成 - 0.01，就会被舍弃了。 而下面的 lerp 函数里，我也再次使用了 step 函数，但是将 step 函数的两个变量调换位置，这样就会得到与上面函数的值相反的区间，注意我的_Cutoff+0.1 是为了让它稍微偏移一点点并非 100% 与上面 clip 相反，它两是有一丢丢的相交部分，这就是火焰的边缘烧灼的部分了，在来个 lerp，即可得到我们想要的结果。

    看来这次的字数凑得比较多，后面的笔记我会不光只复制《入门精要》的内容，而是加入一些小效果，不然做起来也挺枯燥的。最后，献上 shader 源码

Shader "URP/alpha tset"

{

    Properties

    {

        _MainTex("MainTex",2D)="white"{}

        _BaseColor("BaseColor",Color)=(1,1,1,1)

        _Cutoff("Cutoff",float)=1

        [HDR]_BurnColor("BurnColor",Color)=(2.5,1,1,1)// 灼烧光颜色

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

        "RenderType"="TransparentCutout"

        "Queue"="AlphaTest"

        }

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float4 _MainTex_ST;

        half4 _BaseColor;

        float _Cutoff;

        real4 _BurnColor;

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

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord)*_BaseColor;

                clip(step(_Cutoff,tex.r)-0.01);// 这里减去 0.01 是因为 clip 对 0 是还会保留 所以要减去 0.01 让本身为 0 的部分被抛弃

                tex=lerp(tex,_BurnColor,step(tex.r,saturate(_Cutoff+0.1)));//lerp 一下灼烧色和原色 +0.1 是控制灼烧区域范围

                return tex;

            }

            ENDHLSL

        }

    }

}