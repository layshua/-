
这里介绍其中的一种模糊算法：Kawase 模糊。这个在 unity 官方 URP 管线下自带的 volume 使用的 bloom，就是将此 Kawase 模糊采样过程中降升采样精度，得到的名为 “双重模糊” 的算法，这个在下一篇会在讲到。

![[5a6cd0347d9b50cb610d0050bd22d775_MD5.webp]]

模糊效果

先构建一个自定义 renderfeather，他要继承 ScriptableRendererFeature 类。然后再定义一个设置的类，方便我们调整参数。

![[a83f7585598732a38fb94ba1324fc633_MD5.webp]]

设置类

          creat 函数，把设置里的参数传递给自定义 pass。

![[a4dba82588944a7c521721dd24a555a5_MD5.webp]]

creat 函数

     AddRenderPasses 函数，把源图像传到自定义 pass 里计算。

![[64d3b3e26407e8ed3b6993920e5978ed_MD5.webp]]

AddRenderPasses 函数

   然后定义一个自定义 pass 类。这个自定义 pass 需要继承于 ScriptableRenderPass。然后在该类定义一堆参数来对应接受 setting 的数据。

![[39ffd1dc5e326795c63ca3d5ee8359fd_MD5.webp]]

自定义 pass 的参数

        在自定义 pass 里定义一个构造函数。

![[9a6047db3053e51cc0484a5bc09c23ff_MD5.webp]]

构造函数

        在自定义 pass 里定义 setup 函数，同时接受源图像。

![[40378ebaccf716809b8845e16742fb46_MD5.webp]]

setup 函数

      执行函数，用来计算模糊图像。

![[67c65d3a0e7f04efc3294976e9656e73_MD5.webp]]

Execute 函数

然后自定义一个 shader 用来处理图像。shader 非常简单，对目标像素和周围斜四个对角的像素的值进行采样，然后取均值即可。采样范围可以自定义调节。如果对 Kawase 模糊的算法不理解的可以看看文章首提到的链接。

![[7f6dd5f2fe8c5c5cd619fc0e90fb685c_MD5.webp]]

shader 里的采样

完成之后，我们可以在对不同的 renderevent 插入我们的 pass。

![[56492e7c043165e2f2a48d7865aca152_MD5.webp]]

在天空盒和不透明渲染完成后插入

![[b0a1086085f0391daabf34d88dd5c545_MD5.webp]]

在透明渲染完成之后插入

 最后，附上源码。

render feather：

using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

public class Kawaseblur : ScriptableRendererFeature

{

    [System.Serializable]public class mysetting// 定义一个设置的类

    {

        public RenderPassEvent passEvent=RenderPassEvent.AfterRenderingTransparents;// 默认插到透明完成后

        public Material mymat;

        [Range(2,10)]public int downsample=2;

        [Range(2,10)]public int loop=2;

        [Range(0.5f,5)]public float blur=0.5f;

        public string passTag="mypassTag";

    }

    public mysetting setting=new mysetting();

    class CustomRenderPass : ScriptableRenderPass// 自定义 pass

    {

        public Material passMat=null;

        public int passdownsample=2;

        public int passloop=2;

        public float passblur=4;

        public FilterMode passfiltermode{get;set;}// 图像的模式

        private RenderTargetIdentifier passSource{get;set;}// 源图像, 目标图像

        RenderTargetIdentifier buffer1;// 临时计算图像 1

        RenderTargetIdentifier buffer2;// 临时计算图像 2

        string passTag;

        public   CustomRenderPass(string tag)// 构造函数

        {

            passTag=tag;

        }

        public void setup(RenderTargetIdentifier sour)// 接收 render feather 传的图

        {

            this.passSource=sour;  

        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)// 类似 OnRenderimagePass

        {

            int bufferid1=Shader.PropertyToID("bufferblur1");

            int bufferid2=Shader.PropertyToID("bufferblur2");

            CommandBuffer cmd=CommandBufferPool.Get(passTag);

            RenderTextureDescriptor opaquedesc=renderingData.cameraData.cameraTargetDescriptor;

            int width=opaquedesc.width/passdownsample;

            int height=opaquedesc.height/passdownsample;

            opaquedesc.depthBufferBits=0;

            cmd.GetTemporaryRT(bufferid1,width,height,0,FilterMode.Bilinear,RenderTextureFormat.ARGB32);

            cmd.GetTemporaryRT(bufferid2,width,height,0,FilterMode.Bilinear,RenderTextureFormat.ARGB32);

            buffer1=new RenderTargetIdentifier(bufferid1);

            buffer2=new RenderTargetIdentifier(bufferid2);

            cmd.SetGlobalFloat("_Blur",1f);

            cmd.Blit(passSource,buffer1,passMat);

            for(int t=1;t<passloop;t++)

            {

                cmd.SetGlobalFloat("_Blur",t*passblur+1);

                cmd.Blit(buffer1,buffer2,passMat);

                var temRT=buffer1;

                buffer1=buffer2;

                buffer2=temRT;

            }

            cmd.SetGlobalFloat("_Blur",passloop*passblur+1);

            cmd.Blit(buffer1,passSource,passMat);

            context.ExecuteCommandBuffer(cmd);// 执行命令缓冲区的该命令

            CommandBufferPool.Release(cmd);// 释放该命令

        } 

    }

    CustomRenderPass mypass;

    public override void Create()// 进行初始化, 这里最先开始

    { 

        mypass = new CustomRenderPass(setting.passTag);// 实例化一下并传参数, name 就是 tag

        mypass.renderPassEvent=setting.passEvent;

        mypass.passblur=setting.blur;

        mypass.passloop=setting.loop;

        mypass.passMat=setting.mymat;

        mypass.passdownsample=setting.downsample;  

    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)// 传值到 pass 里

    {

        mypass.setup(renderer.cameraColorTarget);

        renderer.EnqueuePass(mypass);

    }

}

shader 源码

Shader "WX/URP/Post/Kawaseblur"

{

    Properties

    {

      _MainTex("MainTex",2D)="white"{}

      //_Blur("Blur",float)=2

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

        float _Blur;

        float4 _MainTex_TexelSize;

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

         v2f VERT(a2v i)// 水平方向的采样

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=i.texcoord;

                return o;

            }

         half4 FRAG(v2f i):SV_TARGET

            {float4 col;                

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord);

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord+float2(-1,-1)*_MainTex_TexelSize.xy*_Blur);

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord+float2(1,-1)*_MainTex_TexelSize.xy*_Blur);

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord+float2(-1,1)*_MainTex_TexelSize.xy*_Blur);

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord+float2(1,1)*_MainTex_TexelSize.xy*_Blur);

                return tex/5.0;

            }

        ENDHLSL

        pass

        {

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            ENDHLSL

        }

    }

}