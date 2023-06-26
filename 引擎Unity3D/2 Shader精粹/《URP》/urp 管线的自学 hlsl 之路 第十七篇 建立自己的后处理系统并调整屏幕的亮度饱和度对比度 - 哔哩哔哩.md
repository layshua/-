        首先祝各位读者们端午节快乐！

       URP 管线的自定义后处理比较头疼，目前情况是没法在 volume 里拓展自定义后处理，而且官方的后处理一直在反复横跳，比如官方人员的这样回复。

![[ca681567c427f218b24234ca0dce05c5_MD5.webp]]

volume 不支持自定义；官方在各个版本疯狂左右横跳

       然后下面就有网友的调侃。

![[48bee3e587c0d4e6fe3cc29b56a15a3f_MD5.webp]]

PPSV2 我到底是用呢还是不用呢

       虽然我们没法拓展 ppsV3 来自定义后处理，但是我们可以通过 render feather 去实现类似自定义后处理的效果，如果要学习 urp 的自定义后处理，强烈安利这个官方 demo，git 的地址如下 https://github.com/Unity-Technologies/UniversalRenderingExamples。

       先创建一个 URP 管线的配置文件，并把它设置为当前的渲染管线。然后在添加 render feather 里会发现有个 render object 的 feather，这个是官方提供的用来渲染模型的 feather，而我们想要的是后处理处理屏幕图像的 feather，所以我们得自己单独创建一个 C# 脚本，这个用来作为处理屏幕图像的 render feather，下面让我们开始写这个 feather 吧。

![[bfe3b8418282a71afc47d59c2ea3666c_MD5.webp]]

官方提供的渲染模型的 render feather

        feather 的类要继承 ScriptableRendererFeature，再定义了一个子类 mysetting，用来定义自己的参数。

![[90430c632bf9fca3d872bbaf5e529094_MD5.webp]]

类 mysetting

         然后定义 create 函数，用于初始化。

![[fb5b99e795f0563db6905d47dbadb073_MD5.webp]]

create 函数

        然后定义 AddRenderPasses，该函数里把参数传到了 pass 里。后面就要写一个自定义 pass 的类。

![[8a1325fe041342041945cdebee46f3a2_MD5.webp]]

AddRenderPasses

        然后在写一个子类，自定义一个 pass，来对屏幕图像进行操作。这样，我们就成功的建立了一个自己的 feather。

![[2749a2aedb333f4677e007e473b86d5a_MD5.webp]]

类 CustomRenderPass

        端午节一直在研究这个自定义的 feather 怎么写，现在总算搞定了，这个脚本里的大部分内容我都做好了注释，写得比较细致，如果有写得不对的地方希望大佬指出。

![[4859a57d75d03dcec1a6fd45cff00fd6_MD5.webp]]

然后就可以在这里指定一个材质球来处理图像

       关于 shader 这块就比较简单了，对于明度，饱和度，对比度的算法直接白嫖乐乐的内容即可，本人就不多说了。客官，上一份源码，要 renderfeather 和 shader 的套餐哦。

renderfeather 源码

using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

public class mybilt : ScriptableRendererFeature

{

    [System.Serializable]public class mysetting// 定义一个设置的类

    {

        public RenderPassEvent passEvent=RenderPassEvent.AfterRenderingTransparents;// 默认插到透明完成后

        public Material mymat;

        public int matpassindex=-1;

    }

    public mysetting setting=new mysetting();

    class CustomRenderPass : ScriptableRenderPass// 自定义 pass

    {

        public Material passMat=null;

        public int passMatInt=0;

        public FilterMode passfiltermode{get;set;}// 图像的模式

        private RenderTargetIdentifier passSource{get;set;}// 源图像, 目标图像

        RenderTargetHandle passTemplecolorTex;// 临时计算图像

        string passTag;

        public   CustomRenderPass(RenderPassEvent passEvent,Material material,int passint,string tag)// 构造函数

        {

            this.renderPassEvent=passEvent;

            this.passMat=material;

            this.passMatInt=passint;

            passTag=tag;

        }

        public void setup(RenderTargetIdentifier sour)// 接收 render feather 传的图

        {

            this.passSource=sour;  

        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)// 类似 OnRenderimagePass

        {

            CommandBuffer cmd=CommandBufferPool.Get(passTag);

            RenderTextureDescriptor opaquedesc=renderingData.cameraData.cameraTargetDescriptor;

            opaquedesc.depthBufferBits=0;

            cmd.GetTemporaryRT(passTemplecolorTex.id,opaquedesc,passfiltermode);// 申请一个临时图像

            Blit(cmd,passSource,passTemplecolorTex.Identifier(),passMat,passMatInt);// 把源贴图输入到材质对应的 pass 里处理，并把处理结果的图像存储到临时图像；

            Blit(cmd,passTemplecolorTex.Identifier(),passSource);// 然后把临时图像又存到源图像里 

            context.ExecuteCommandBuffer(cmd);// 执行命令缓冲区的该命令

            CommandBufferPool.Release(cmd);// 释放该命令

            cmd.ReleaseTemporaryRT(passTemplecolorTex.id);// 释放临时图像

        } 

    }

    CustomRenderPass mypass;

    public override void Create()// 进行初始化, 这里最先开始

    {

        int passint=setting.mymat==null?1:setting.mymat.passCount-1;// 计算材质球里总的 pass 数，如果没有则为 1

        setting.matpassindex=Mathf.Clamp(setting.matpassindex,-1,passint);// 把设置里的 pass 的 id 限制在 - 1 到材质的最大 pass 数

         mypass = new CustomRenderPass(setting.passEvent,setting.mymat,setting.matpassindex,name);// 实例化一下并传参数, name 就是 tag

    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)// 传值到 pass 里

    {

        var src=renderer.cameraColorTarget;

        mypass.setup(src);

        renderer.EnqueuePass(mypass);

    }

}

第二份菜，就是 shader 了。这是 shader 的效果图。

![[cd26c9ff544532121e5b8dff39f32cfb_MD5.webp]]

亮度

![[db5a4be6b69637e4983c367b67f095aa_MD5.webp]]

饱和度

![[5adb9d240260fa44eed233a7b5c6a6a5_MD5.webp]]

对比度

Shader "WX/URP/Post/post"  

{

    Properties

    {

      [HideInInspector]_MainTex("MainTex",2D)="white"{}

      _brightness("Brightness",Range(0,1))=1

      _saturate("Saturate",Range(0,1))=1

      _contranst("Constrast",Range(-1,2))=1

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

        }

        Cull Off ZWrite Off ZTest Always

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float _brightness;

        float _saturate;

        float _contranst;

        CBUFFER_END

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

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

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=i.texcoord;

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord);

                float gray=0.21*tex.x+0.72*tex.y+0.072*tex.z;// 灰度图，即计算明度

                tex.xyz*=_brightness;// 计算亮度

                tex.xyz=lerp(float3(gray,gray,gray),tex.xyz,_saturate);// 饱和度

                tex.xyz=lerp(float3(0.5,0.5,0.5),tex.xyz,_contranst);// 对比度

                return tex;

            }

            ENDHLSL

        }

    }

}