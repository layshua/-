         色彩分离，正如其名，把一张图片的 r，g，b 通道的 uv 进行稍微的偏移，就可以得到很酷的色彩分离效果，如下图效果。

![[13f304207e14e9304f94e4a38cda1fdd_MD5.webp]]

        其中的 uv 偏移也仅仅偏移了 0.01 的单位，三个通道中，选择了 2 个进行偏移，如下所示。

![[b4d78dbd25f67bdde0cb2bfaa5455ce7_MD5.webp]]

        这样，我们就有了思路，如果让偏移进行连续快速偏移，就有了故障的动态效果了。为了更好得控制我们想要的效果，我们需要自定义一个函数出来，我使用 GGB（全名 GeoGebra，一个非常好用的数学图形软件）随机定义了几个顶点，让它去拟合一条函数。

![[737be06198d19574442bf4981136626a_MD5.webp]]

        我随机定义了一些点，让它大概在 0-1 之间波动，然后我使用 polynomial 去模拟了一根线，得到的函数如下。

![[87d4ccf23d8067c0fa942153455448e3_MD5.webp]]

        然后该曲线对应的函数为 12 次函数，心理万 * 奔腾，没法用，还是得改改。

![[de4d9a4fced315c83fe1f2143f8caf68_MD5.webp]]

经过我多次调整，得到了一个比较舒服的曲线，并且计算量并不是很大。

![[9895548df5c5aa9afcf3854f795bc4d2_MD5.webp]]

其对应的函数如下

![[76f6254ac090cd456cce88b0c011a46c_MD5.webp]]

这样，我们把它写在 shader 里定义成函数，该函数的定义域为 [0,1], 值域为 [-1，1]。

![[50172941e781c34871fe9d9c3464a826_MD5.webp]]

然后我们把时间取小数作为参数输入，就可以得到一个 “鬼畜” 的数字了。

![[b1ee2394b196fc1ad87b8fd95c781b67_MD5.webp]]

在使用该数字去偏移图像的 R,B 通道的 uv，就可以得到你想要的效果，这里的数值可以根据你的喜好修改，我的参数并不唯一答案。

![[23fa38738178e611f530230d4f5f19d6_MD5.webp]]

![[0bc917a88ddb4866e81e2dd99cb00422_MD5.webp]]

看起来是很酷，但是一直这么闪，感觉很傻 @￥*，所有我用 uv 去做了个 mask，如下图。

![[510c04a0a0d79436d6b516a6ef3fc9e8_MD5.webp]]

![[4309d53801d44fe7d9d3015400e577e0_MD5.webp]]

然后再把原图和闪图 lerp 一下，这样中间部分就不会闪了，只有边缘在闪。

![[58fbf0a4bdb27ff5c1b10a42cf57c1ee_MD5.webp]]

![[9825cc14eac58141faf3f6adbf5e9fc4_MD5.webp]]

最后，我再把这个 mask 图和 0 通过我自定义的_Instensity 参数进行 lerp 一下，最后就可以通过

_Instensity 参数进行调整。

![[0d7c7a2af01eb3ff7198846a57ced5d8_MD5.webp]]

RendererFeature 里就一个 blit 搞定，同时控制这个材质的参数即可。

![[1720def324daf154f7737e2cc933f5da_MD5.webp]]

看这个色彩分离看多了，感觉自己电脑屏幕都有花屏的错觉，还是附上源码吧。

SHADER 源码

Shader "WX/URP/Post/NewImageEffectShader"

{

    Properties

    {

      [HideInInspector]_MainTex("MainTex",2D)="white"{}

      [HideInInspector]_Instensity("Instensity",Range(0,1))=0.5

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

        }

        Cull Off ZWrite Off ZTest Always

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float4 _MainTex_TexelSize;

        float _Instensity;

        CBUFFER_END

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

         struct a2v

         {

             float4 positionOS:POSITION;

             float2 texcoord:TEXCOORD;

         };

         struct v2f

         {

             float4 positionCS:SV_POSITION;

             float2 texcoord:TEXCOORD;

         };

        ENDHLSL

        pass

        {

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=i.texcoord;

                return o;

            }

            float cur(float x);

            half4 FRAG(v2f i):SV_TARGET

            {

               float noise=cur(frac(_Time.y));

                half R=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord+float2(0.02,0.04)*noise).x;

                half G=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord).y;

                half B=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord-float2(0.05,-0.07)*noise).z;

                real4 mainTEX=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord);

                real4 SplitRGB=real4(R,G,B,1);

                //return SplitRGB;

                float2 center=i.texcoord*2-1;

                float mask=saturate(dot(center,center));

                mask=lerp(0,mask,_Instensity);

                //return mask;

                SplitRGB=lerp(mainTEX,SplitRGB,mask);

                return SplitRGB;

            }

            float cur(float x)

            {

                return -4.71*pow(x,3)+6.8*pow(x,2)-2.65*x+0.13+0.8*sin(48.15*x-0.38);

            }

            ENDHLSL

        }

    }

}

RendererFeature

using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

[ExecuteInEditMode]

public class GlitchBlit : ScriptableRendererFeature

{

    [System.Serializable]public class setting

        {

            public Material mat=null;

            [Range(0,1)]public float Instensity=0.5f;

            public RenderPassEvent passEvent=RenderPassEvent.AfterRenderingOpaques;

        }

        public setting mysetting;

    class GlitchColorSplit : ScriptableRenderPass

    {

        setting mysetting=null;

        RenderTargetIdentifier sour;

        public void stetup(RenderTargetIdentifier source)

        {

            this.sour=source;

        }

        public GlitchColorSplit(setting set)

        {

            mysetting=set;

        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)

        {

            mysetting.mat.SetFloat("_Instensity",mysetting.Instensity);

            CommandBuffer cmd=CommandBufferPool.Get("GlitchColorSplit");

            RenderTextureDescriptor desc=renderingData.cameraData.cameraTargetDescriptor;

            int sourID=Shader.PropertyToID("_SourTex");

            cmd.GetTemporaryRT(sourID,desc);

            cmd.CopyTexture(sour,sourID);

            cmd.Blit(sourID,sour,mysetting.mat);

            context.ExecuteCommandBuffer(cmd);

            cmd.ReleaseTemporaryRT(sourID);

            CommandBufferPool.Release(cmd);

        }

    }

    GlitchColorSplit m_ColorSplit;

    public override void Create()

    {

        m_ColorSplit = new GlitchColorSplit(mysetting);

        m_ColorSplit.renderPassEvent =mysetting.passEvent;

    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)

    {

        m_ColorSplit.stetup(renderer.cameraColorTarget);

        renderer.EnqueuePass(m_ColorSplit);

    }

}