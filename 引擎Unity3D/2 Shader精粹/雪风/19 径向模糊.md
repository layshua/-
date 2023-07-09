![[2b099f03b15a1fb2d7b95cdcb8824bee_MD5.webp]]

径向模糊

        实现了个比较简单的模糊效果，这篇专栏内容虽然简单，但是效果却很常见，基本上赛车游戏，或者动作游戏的加速，又或者大型 boss 的出场都可见这种特效。在 render feather 里我定义了如下参数用于调试效果。

![[646fd02181a96672131b8491a0b4468e_MD5.webp]]

参数设置

      passname：当前 renderFeather 的名字。可以在 frame debug 里查看。

![[e1303036b26af53549b5678134e9c635_MD5.webp]]

Radial Blur Mat，径向模糊材质球。

X，径向模糊的中心水平位置。

Y，径向模糊的竖直方向位置。

loop，迭代次数。

blur，模糊采样的距离。

downsample，降采样的程度。

instensity，模糊强度，0 为不模糊，1 为全模糊，用于调整过渡切换。

      说明该后处理所使用的 shader。shader 定义了 2 个 pass，第一个 pass 用于计算径向模糊的图像，第二个 pass 用于把模糊图像和原高清图混合 lerp。第一个 pass 里的算子如下图。

![[6e2b5bc5c98690478ec476b0c16e1cb2_MD5.webp]]

采样示意图

        根据以中心为起点，当前像素坐标为终点作为采样方向，分别对 1，2，3，4...... 进行迭代采样，然后对采样的结果采取平均取权重，除以迭代次数，把所有的采样图加起来，即可得到我们需要的径向模糊 pass。

![[3a666f446c29396f241e3bc66261b20f_MD5.webp]]

径向模糊的 pass

        第二个 pass 的处理就更简单了，直接采样原始图像和第一个 pass 处理后的模糊图像，然后将它两 lerp 一下即可。

![[e57289c8270691105aa0ff9701e51b7f_MD5.webp]]

片元着色

       然后就是 render feather 的部分。先把原图像分别储存在一张降采样的 temp1 和一张与屏幕同大小的 temp2 里。这里降采样 temp1 考虑到移动端设备的优化，因为第一个 pass 里是有一个 for 循环计算，并且还是在片元着色器里计算，在现在移动设备动不动 2K 屏的时代实在伤不起。

![[cf89fddbcea5e41db32cec18be82b6e8_MD5.webp]]

核心计算过程

        使用 pass1 对 temp1 处理得到模糊贴图 blurtex，然后将 blurtex 和 temp2 在 pass2 里进行混合 lerp，其混合程度由 instensity 参数控制，这样即可得到我们需要的屏幕图像，我画了简易示例图来解释这个过程，希望能帮到读者。

![[6293352dc5ff1cdfb258a4ea22501013_MD5.webp]]

计算过程

      本 render feather 的计算应该还有一下可以优化的地方，读者若有其他想法欢迎留言讨论。

下面附源码，注意网页的空格复制过去有 bug，需手动替换空格不然代码会语法报错。

using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

[ExecuteInEditMode]

public class radialBlur : ScriptableRendererFeature

{

    [System.Serializable]public class setting

    {

        public string PassName="径向模糊";

        public Material RadialBlurMat=null;

       [Range(0,1)] public float x=0.5f;

       [Range(0,1)] public float y=0.5f;

      [Range(1,8)] public int loop=5;

      [Range(1,8)] public float blur=3;

       [Range(1,5)]public int downsample=2;

       [Range(0,1)] public float instensity=0.5f;

       public RenderPassEvent passEvent=RenderPassEvent.AfterRenderingTransparents;

    }

    public setting mysetitng=new setting();

    class CustomRenderPass : ScriptableRenderPass

    {

        public Material mymat;

        public string name;

        public float x;

        public float y;

        public int loop;

        public float instensity;

      public float blur;

        public int downsample;

        public RenderTargetIdentifier Source{get;set;}

        public RenderTargetIdentifier BlurTex;

        public RenderTargetIdentifier Temp1;

        public RenderTargetIdentifier Temp2;

        int ssW;

        int ssH;

        public void setupmypass(RenderTargetIdentifier source)

        {

            this.Source=source;

        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)

        {

            int BlurTexID=Shader.PropertyToID("_BlurTex");

            int TempID1=Shader.PropertyToID("Temp1");

            int TempID2 = Shader.PropertyToID("_SourceTex");

            int loopID=Shader.PropertyToID("_Loop");

            int Xid=Shader.PropertyToID("_X");

            int Yid=Shader.PropertyToID("_Y");

            int BlurID=Shader.PropertyToID("_Blur");

            int instenID=Shader.PropertyToID("_Instensity");

            RenderTextureDescriptor SSdesc=renderingData.cameraData.cameraTargetDescriptor;

            ssH=SSdesc.height/downsample;

            ssW=SSdesc.width/downsample;

            CommandBuffer cmd=CommandBufferPool.Get(name);

            cmd.GetTemporaryRT(TempID1, ssW,ssH,0,FilterMode.Bilinear,RenderTextureFormat.ARGB32);// 用来存降采样的

            cmd.GetTemporaryRT(BlurTexID, SSdesc);// 模糊图

            cmd.GetTemporaryRT(TempID2, SSdesc);//

            BlurTex =new RenderTargetIdentifier(BlurTexID);

            Temp1=new RenderTargetIdentifier(TempID1);

            Temp2 = new RenderTargetIdentifier(TempID2);

            cmd.SetGlobalFloat(loopID,loop);

            cmd.SetGlobalFloat(Xid,x);

            cmd.SetGlobalFloat(Yid,y);

            cmd.SetGlobalFloat(BlurID,blur);

            cmd.SetGlobalFloat(instenID,instensity);

            cmd.Blit(Source, Temp1);// 存储降采样的源图，用于 pass0 计算模糊图

            cmd.Blit(Source, Temp2);// 存储源图，用于计算 pass1 的混合

            cmd.Blit(Temp1, BlurTex, mymat, 0);//pass0 的模糊计算

            cmd.Blit(BlurTex, Source, mymat, 1);//pass1 的混合

            context.ExecuteCommandBuffer(cmd);

            cmd.ReleaseTemporaryRT(BlurTexID);

            cmd.ReleaseTemporaryRT(TempID1);

            cmd.ReleaseTemporaryRT(TempID2);

            CommandBufferPool.Release(cmd);

        }

    }

    CustomRenderPass m_ScriptablePass;

    public override void Create()

    {

        m_ScriptablePass = new CustomRenderPass();

        m_ScriptablePass.renderPassEvent = mysetitng.passEvent;

        m_ScriptablePass.blur=mysetitng.blur;

        m_ScriptablePass.x=mysetitng.x;

        m_ScriptablePass.y=mysetitng.y;

        m_ScriptablePass.instensity=mysetitng.instensity;

        m_ScriptablePass.loop=mysetitng.loop;

        m_ScriptablePass.mymat=mysetitng.RadialBlurMat;

        m_ScriptablePass.name=mysetitng.PassName;

        m_ScriptablePass.downsample=mysetitng.downsample;

    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)

    {

        if (mysetitng.RadialBlurMat != null)

        {

            m_ScriptablePass.setupmypass(renderer.cameraColorTarget);

            renderer.EnqueuePass(m_ScriptablePass);

        }

        else

        {

            Debug.LogError("径向模糊材质球丢失！");

        }

    }

}

Shader"WX/URP/Post/radialBrul"

{

    Properties

    {

        _MainTex("tex",2D)="Wwhite"{}

    }

        SubShader

    {

        Tags{

        "RenderPipeline" = "UniversalRenderPipeline"

        }

        Cull Off ZWrite Off ZTest Always

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        float4 _MainTex_ST;

        float _Loop;

        float _Blur;

        float _Y;

        float _X;

        float _Instensity;

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

        TEXTURE2D(_SourceTex);

        SAMPLER(sampler_SourceTex);

         struct a2v

         {

             float4 positionOS:POSITION;

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

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS = TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord = i.texcoord;

                return o;

            }

            half4 FRAG(v2f i) :SV_TARGET

            {

                float4 col = 0;

                float2 dir = (i.texcoord - float2(_X,_Y)) * _Blur * 0.01;

                for (int t = 0; t < _Loop; t++)

                {

                    col += SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord + dir * t) / _Loop;

                }

                return col;

            }

            ENDHLSL

        }

        pass

        {

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS = TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord = i.texcoord;

                return o;

            }

            half4 FRAG(v2f i) :SV_TARGET

            {

                float4 blur = SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord);// 得到模糊贴图

                float4 Source = SAMPLE_TEXTURE2D(_SourceTex,sampler_SourceTex,i.texcoord);// 得到屏幕原始图

                return lerp(Source,blur,_Instensity);

            }

            ENDHLSL

        }

    }

}