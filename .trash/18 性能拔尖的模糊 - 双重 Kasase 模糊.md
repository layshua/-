        该模糊算法取自该文章 https://blog.csdn.net/poem_qianmo/article/details/105350519。

眼馋它极好的性能，故本人把该模糊移植到 URP 管线下使用。双重模糊里双重的意思是采样迭代过程中的先降低 RT 尺寸在提升 RT 尺寸，最终就以更小的内存，更少的绘制次数达到很好的模糊效果。下图是只绘制 6 次就达到非常出色的模糊效果的展示图。

![[e9c6333d9f81c5eeb0f15376d72e3c09_MD5.webp]]

效果图

       第一步：先介绍他的 shader 部分，shader 所使用的算子在升采样和降采样的 pass 不一样，这里分开介绍。降采样的算子如下。

![[f69add6966c2f6e59f7a222b695605b9_MD5.webp]]

降采用的算子

降采样过程中，每个像素采样了 5 个像素，中间的占 1/2，边缘 4 个占 1/8，这样它的着色器的写法如下图，非常简单。

![[b01718ff2d2b0b9a84be889bcb0346d3_MD5.webp]]

降采样 pass

         而升采样过程中，他采样了 8 个算子。

![[c463f91dedb0c83f1c53971f6d36c220_MD5.webp]]

升采样算子

        四个近的算子权重为 1/6，远的算子权重为 1/12，着色器写法也简单，如下图所示。

![[9ab73e96e133e4382006adb42fa8a14e_MD5.webp]]

升采样 pass

  shader 是开胃菜，而自定义的 render feather 才是真正的麻烦（各位程序大大请轻喷，本人只是个小美术，对美术摸 urp 的新东西确实挺头疼的）。下面本人说明一下 render feather 的内容。

  第二步，自定义 render feather。先定义一个 setting，这个大家都懂。

![[36d688e4fd9977276cf85c9d89f0b41b_MD5.webp]]

定义 seting

初始化和传参数的函数。之前专栏里说了多次，也不多讲了。

![[52b71987969f4a8761b99f9a4f7bfc04_MD5.webp]]

         然后就是定义 custompass 类了。熟悉的老伙计，但是本人改了不少内容和之前的不一样，这里说明下。

        定义 level 结构体，存 down 和 up，它是为了记录升采样和降采样对应的 RT 的 id 的。由于我们循环过程之中，要定义很多个不同尺寸大小 RT 的 id，索性在定义一个 level 的数组来存储。然后在初始化的时候，把他们初始化一下，但是注意 id 不能重复哦。

![[af8ad5d77b4005733b7d2c12371b9905_MD5.webp]]

初始化

        执行函数还是和之前一致。后面的循环略用得比较取巧：在第一个降采样的循环过程中，就把升采样的 RT 也申请了。故在第二个升采样的循环中并未申请 RT.

![[c912a946f60fec889b933c9c82cf7725_MD5.webp]]

执行函数

        我手绘了一张图来解释这个执行函数里的每一步对应的那个阶段，字写得不好请轻喷，参考我这张图，读者应该可以更快速理解每一步的作用了吧。

![[33bf1936cd4704688b7304fd12545dea_MD5.webp]]

图例

        至此，双重模糊的解析就完成了，下面再附录我的 render feather 的源码和 shader 源码。

Render Feather：

using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

public class Doualblur : ScriptableRendererFeature

{

    [System.Serializable]public class mysetting// 定义一个设置的类

    {

        public RenderPassEvent passEvent=RenderPassEvent.AfterRenderingTransparents;// 默认插到透明完成后

        public Material mymat;

        [Range(1,8)]public int downsample=2;

        [Range(2,8)]public int loop=2;

        [Range(0.5f,5f)]public float blur=0.5f;

        public string RenderFeatherName="我的双重 Kawase 模糊";//render feather 的名字

    }

    public mysetting setting=new mysetting();

    class CustomRenderPass : ScriptableRenderPass// 自定义 pass

    {

        public Material passMat=null;

        public int passdownsample=2;// 降采样

        public int passloop=2;// 模糊的迭代次数

        public float passblur=4;

        private RenderTargetIdentifier passSource{get;set;}

        RenderTargetIdentifier buffer1;//RTa1 的 ID

        RenderTargetIdentifier buffer2;//RTa2 的 ID

        string RenderFeatherName;//feather 名

        struct LEVEL//

        {

            public int down;

            public int up;

        };

        LEVEL[] my_level;

        int maxLevel=16;// 指定一个最大值来限制申请的 ID 的数量，这里限制到 16 个，这么多肯定用不完了

        public   CustomRenderPass(string name)// 构造函数

        {

            RenderFeatherName=name;

        }

        public void setup(RenderTargetIdentifier sour)// 初始化，接收 render feather 传的图

        {

            this.passSource=sour;

            my_level = new LEVEL[maxLevel];

            for (int t = 0; t < maxLevel; t++)// 申请 32 个 ID 的，up 和 down 各 16 个，用这个 id 去代替临时 RT 来使用

            {

                my_level[t] = new LEVEL

                {

                    down = Shader.PropertyToID("_BlurMipDown"+t),

                    up = Shader.PropertyToID("_BlurMipUp"+t)

                };

            }

        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)// 执行

        {

            CommandBuffer cmd=CommandBufferPool.Get(RenderFeatherName);// 定义 cmd

            passMat.SetFloat("_Blur",passblur);// 指定材质参数

            //cmd.SetGlobalFloat("_Blur",passblur);// 设置模糊, 但是我不想全局设置怕影响其他的 shader，所以注销它了用上面那个，但是 cmd 这个性能可能好些？

            RenderTextureDescriptor opaquedesc=renderingData.cameraData.cameraTargetDescriptor;// 定义屏幕图像参数结构体

            int width=opaquedesc.width/passdownsample;// 第一次降采样是使用的参数，后面就是除 2 去降采样了

            int height=opaquedesc.height/passdownsample;

            opaquedesc.depthBufferBits=0;

            //down

            RenderTargetIdentifier LastDown=passSource;// 把初始图像作为 lastdown 的起始图去计算

            for(int t=0;t<passloop;t++)

            {

                int midDown=my_level[t].down;//middle down ，即间接计算 down 的工具人 ID

                int midUp=my_level[t].up; //middle Up ，即间接计算的 up 工具人 ID

                cmd.GetTemporaryRT(midDown,width,height,0,FilterMode.Bilinear,RenderTextureFormat.ARGB32);// 对指定高宽申请 RT，每个循环的指定 RT 都会变小为原来一半

                cmd.GetTemporaryRT(midUp,width,height,0,FilterMode.Bilinear,RenderTextureFormat.ARGB32);// 同上，但是这里申请了并未计算，先把位置霸占了，这样在 UP 的循环里就不用申请 RT 了

                cmd.Blit(LastDown,midDown,passMat,0);// 计算 down 的 pass

                LastDown=midDown;// 工具人辛苦了

                width=Mathf.Max(width/2,1);// 每次循环都降尺寸

                height=Mathf.Max(height/2,1);

            }

            //up

            int lastUp=my_level[passloop-1].down;// 把 down 的最后一次图像当成 up 的第一张图去计算 up

            for(int j=passloop-2;j>=0;j--)// 这里减 2 是因为第一次已经有了要减去 1，但是第一次是直接复制的，所以循环完后还得补一次 up

            {

                int midUp=my_level[j].up;

                cmd.Blit(lastUp,midUp,passMat,1);// 这里直接开干就是因为在 down 过程中已经把 RT 的位置霸占好了，这里直接用，不虚

                lastUp=midUp;// 工具人辛苦了

            }

           cmd.Blit(lastUp,passSource,passMat,1);// 补一次 up，顺便就输出了

            context.ExecuteCommandBuffer(cmd);// 执行命令缓冲区的该命令

            CommandBufferPool.Release(cmd);// 释放 cmd

            for(int k=0;k<passloop;k++)// 清 RT，防止内存泄漏

            {

                cmd.ReleaseTemporaryRT(my_level[k].up);

                cmd.ReleaseTemporaryRT(my_level[k].down);

            }

        } 

    }

    CustomRenderPass mypass;

    public override void Create()// 进行初始化, 这里最先开始

    { 

        mypass = new CustomRenderPass(setting.RenderFeatherName);// 实例化一下并传参数, name 就是 tag

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

shader 源码，注意我的 2 个 pass 的 v2f 结构体是用的同一个，而 down 的 pass 并没有完全用完结构体，这样会造成一些性能损失，读者若是在正式项目建议分开写 v2f，不要学我偷懒。

Shader "WX/URP/Post/DualBlur"

{

    Properties

    {

      //

  [HideInInspector]_MainTex("MainTex",2D)="white"{}

       //_Blur("Blur",float)=3

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

        float4 _MainTex_ST;

        float4 _MainTex_TexelSize;

        float _Blur;

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

             float4 texcoord[4]:TEXCOORD;

         };

        ENDHLSL

        pass//Down

        {

        NAME"Down"

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord[2].xy=i.texcoord;

                o.texcoord[0].xy=i.texcoord+float2(1,1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[0].zw=i.texcoord+float2(-1,1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[1].xy=i.texcoord+float2(1,-1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[1].zw=i.texcoord+float2(-1,-1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[2].xy)*0.5;

                for(int t=0;t<2;t++)

                {

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[t].xy)*0.125;

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[t].zw)*0.125;

                }

                return tex;

            }

            ENDHLSL

        }

        pass//up

        {

        NAME"Up"

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord[0].xy=i.texcoord+float2(1,1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[0].zw=i.texcoord+float2(-1,1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[1].xy=i.texcoord+float2(1,-1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[1].zw=i.texcoord+float2(-1,-1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[2].xy=i.texcoord+float2(0,2)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[2].zw=i.texcoord+float2(0,-2)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[3].xy=i.texcoord+float2(-2,0)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[3].zw=i.texcoord+float2(2,0)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=0;

                for(int t=0;t<2;t++)

                {

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[t].xy)/6;

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[t].zw)/6;

                }

                for(int k=2;k<4;k++)

                {

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[k].xy)/12;

                tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[k].zw)/12;

                }

                return tex;

            }

            ENDHLSL

        }

    }

}