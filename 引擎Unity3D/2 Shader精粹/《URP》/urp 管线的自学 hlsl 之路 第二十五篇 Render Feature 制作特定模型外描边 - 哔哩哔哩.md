![[44a9a2573ed3aa699912a0bae84273a3_MD5.webp]]

      本文是记录本人自学 urp 的一个月写的最后一篇专栏，最近因为 urp 这个新的 Renderer eFeature 花了很多精力看官方源码，自己也是一边啃一边问，很多细节的东西自己也没有摸清楚，本文仅供读者参考，实际情况请以读者的实际项目进行调整。本效果的支持如下：

![[89d189f2e9e0c57228769e9d4622d277_MD5.webp]]

*   支持颜色修改；
    
*   支持队列的过滤；
    
*   支持层级的过滤；
    
*   支持 pass 的插入位置；
    
*   支持模糊的半径；
    
*   支持模糊的迭代数；
    
*   支持 2 种颜色的表现类型，如下图。
    

![[8e35b62e8a0753b7affed5b41865a981_MD5.webp]]

左为带内描边，右为不带

        基本原理如下：1. 使用 DrawRenderers 对目标过滤的模型进行渲染，渲染使用材质球里的第一个 pass，将得到的图像存到名为 temp 的 RT 里；2. 然后在将 temp 的 RT 在材质的第二三个 pass 里进行 kawase 升降采样模糊，把模糊后的图存在_BlurTex 的 RT 里；3. 再把屏幕图像 sour，_BlurTex 的 RT，temp 的 RT 在材质的第 4 个 pass 里合并计算，根据不同的计算方式可以得到 2 种描边效果（如上图）。

    **先说明 shader，**shader 比较简单，带了 4 个 pass，其部分内容也是之前在专栏里提到过的内容。

![[9e7acbe58ba077f365e8711c89863003_MD5.webp]]

       上纯色 pass，灰常简单，就是给模型画一下纯色。

        双重模糊 DownPass 和 双重模糊 upPass，在以前的专栏已经细致讲过，这里不再重复，跳转链接 [https://www.bilibili.com/read/cv6597443](https://www.bilibili.com/read/cv6597443)。

        合并图像 pass，分 2 种情况进行处理。

![[fee7e8d950968180ea21d89e48811714_MD5.webp]]

        然后就是 Renderer Feature 部分，rendererFeature 里有 2 个 CustomRenderPass，第一个 CustomRenderPass 插在绘制不透明物体之前，仅仅是给指定模型上纯色画在一张 rt 上；第一个 pass 是在画完不透明物体之后（或者半透明物体之后，具体插入位置由读者指定），进行模糊绘制，最后合并所有图像。

        第一个绘制纯色的 CustomRenderPass 里，设置好过滤的队列和层级，指定渲染目标，使用 DrawRenderers 进行渲染即可，渲染后并不释放 rt（这张图我们还需要继续使用，不能丢回 pool 里），并且把对应的 id 存到外面，方便第二个 CustomRenderPass 使用这张图。

![[b7b78c6da0bf098dccf481a567d91ae1_MD5.webp]]

第一个绘制纯色的 CustomRenderPass  

![[01f83b0e9788fff81c53aa88e16cee99_MD5.webp]]

绘制得到的纯色图和对应的位置

                第二个 CustomRenderPass 里，先对第一个 CustomRenderPass 的图像进行模糊计算，计算之前对纯色图做个备份，因为我们后续会用到它。下图为模糊后的效果，使用了 3 次循环，3 次降采样 3 次升高采样，模糊算子使用 kawase 算子进行升降，具体的原理请移步该专栏 [https://www.bilibili.com/read/cv6597443](https://www.bilibili.com/read/cv6597443)。

![[81785734864d46ac85f9dd6bfa6e3945_MD5.webp]]

双重 kawase 模糊后的效果

        我们将这张模糊的图像减去之前的纯色图会得到什么结果？模糊的部分只绘制在了外面，貌似一切正常。

![[173c4439d6dc1e70542ac5e2644a952b_MD5.webp]]

        但是若再叠加上屏幕原本的颜色，就发现模型内部的颜色并不对有些 “反色” 的效果，原因是模型内部的颜色为负数，叠加后就是反色的效果了。

![[e80136341abb2943aba4a172ef023ce6_MD5.webp]]

处理方法很简单，直接把为负数的部分直接舍弃掉即可，这里我们使用 saturate 函数。

![[9c27d5c9dcc134557137b4199ae1f0bd_MD5.webp]]

舍弃掉负数部分后的效果

当然，我们把负数的部分变成正数会如何？把 saturate 函数换成 abs 函数取绝对值试试。看起来效果还不错。

![[1391ba0748493b31d73343062d5f45aa_MD5.webp]]

使用 abs 函数

为了保留这 2 种效果，本人拿出 multi_compile_local 做 shader 变体，然后在 RenderFeature 外部去控制选择。

![[010715fa94215eb2fa690c36b1a336c2_MD5.webp]]

         计算完毕后，我们再把所有的 rt 丢回 pool 里，包括第一个 CustomRenderPass 保留的纯色 rt。

         这样，我们就得到了我们想要的效果了，更多的细节建议读者在源码里去查看，如果有疑问欢迎留言讨论。

SHADER 源码

Shader "WX/URP/Post/SelectOutline"

{

    Properties

    {

      [HideInInspector]_MainTex("MainTex",2D)="white"{}

      [HideInInspector]_SoildColor("SoildColor",Color)=(1,1,1,1)

      [HideInInspector]_Blur("Blur",float)=1

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

        real4 _SoildColor;

        float _Blur;

        CBUFFER_END

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

        TEXTURE2D(_BlurTex);

        SAMPLER(sampler_BlurTex);

        TEXTURE2D(_SourTex);

        SAMPLER(sampler_SourTex);

         struct a2v

         {

             float4 positionOS:POSITION;

             float2 texcoord:TEXCOORD;

         };

         struct v2f

         {

             float4 positionCS:SV_POSITION;

             float4 texcoord[4]:TEXCOORD;

         };

        ENDHLSL

        pass// 上纯色

        {

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                return _SoildColor;

            }

            ENDHLSL

        }

        pass// 双重模糊 DownPass

        {

            NAME"Down"

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord[2].xy=i.texcoord;

                o.texcoord[0].xy=i.texcoord+float2(1,1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[0].zw=i.texcoord+float2(-1,1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[1].xy=i.texcoord+float2(1,-1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[1].zw=i.texcoord+float2(-1,-1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[2].xy)*0.5;

                for(int t=0;t<2;t++)

                {

                    tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[t].xy)*0.125;

                    tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[t].zw)*0.125;

                }

                return tex;

            }

            ENDHLSL

        }

        pass// 双重模糊 upPass

        {

            NAME"Up"

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord[0].xy=i.texcoord+float2(1,1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[0].zw=i.texcoord+float2(-1,1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[1].xy=i.texcoord+float2(1,-1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[1].zw=i.texcoord+float2(-1,-1)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[2].xy=i.texcoord+float2(0,2)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[2].zw=i.texcoord+float2(0,-2)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[3].xy=i.texcoord+float2(-2,0)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                o.texcoord[3].zw=i.texcoord+float2(2,0)*_MainTex_TexelSize.xy*(1+_Blur)*0.5;

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=0;

                for(int t=0;t<2;t++)

                {

                    tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[t].xy)/6;

                    tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[t].zw)/6;

                }

                for(int k=2;k<4;k++)

                {

                    tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[k].xy)/12;

                    tex+=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[k].zw)/12;

                }

                return tex;

            }

            ENDHLSL

        }

        pass// 合并所有图像

        {

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            #pragma multi_compile_local _INCOLORON _INCOLOROFF

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord[0].xy=i.texcoord.xy;

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                real4 blur=SAMPLE_TEXTURE2D(_BlurTex,sampler_BlurTex,i.texcoord[0].xy);

                real4 sour=SAMPLE_TEXTURE2D(_SourTex,sampler_SourTex,i.texcoord[0].xy);

                real4 soild=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord[0].xy);

                real4 color;

                #ifdef _INCOLORON

                color= abs(blur-soild)+sour;

                #elif _INCOLOROFF

                color=saturate(blur-soild)+sour;

                 #endif

                return color;

            }

            ENDHLSL

        }

    }

}

RendererFeature 源码

using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

[ExecuteInEditMode]

public class SelectOutline : ScriptableRendererFeature

{

    public enum TYPE

        {

            INcolorON,INcolorOFF

        }

    [System.Serializable]public class setting

    {

        public Material mymat;

        public Color color=Color.blue;

        [Range(1000,5000)]public int QueueMin=2000;

        [Range(1000,5000)]public int QueueMax=2500;

        public LayerMask layer;

        public RenderPassEvent passEvent=RenderPassEvent.AfterRenderingSkybox;

       [Range(0.0f,3.0f)] public float blur=1.0f;

        [Range(1,5)]public int passloop=3;

        public TYPE ColorType=TYPE.INcolorON;

    }

    public setting mysetting=new setting();

    int solidcolorID;

    // 第一个 pass 绘制纯色的图像

    class DrawSoildColorPass : ScriptableRenderPass

    {

        setting mysetting=null;

        SelectOutline SelectOutline=null;

        ShaderTagId shaderTag = new ShaderTagId("DepthOnly");// 只有在这个标签 LightMode 对应的 shader 才会被绘制

        FilteringSettings filter;

        public DrawSoildColorPass(setting setting,SelectOutline render)

        {

            mysetting=setting;

            SelectOutline=render;

            // 过滤设定

            RenderQueueRange queue=new RenderQueueRange();

            queue.lowerBound=Mathf.Min(setting.QueueMax,setting.QueueMin);

            queue.upperBound=Mathf.Max(setting.QueueMax,setting.QueueMin);

            filter=new FilteringSettings(queue,setting.layer);

        }

        public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)

        {

            int temp=Shader.PropertyToID("_MyTempColor1");

            RenderTextureDescriptor desc=cameraTextureDescriptor;

            cmd.GetTemporaryRT(temp,desc);

            SelectOutline.solidcolorID=temp;

            ConfigureTarget(temp);

            ConfigureClear(ClearFlag.All,Color.black);

        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)

        {

            mysetting.mymat.SetColor("_SoildColor",mysetting.color);

            CommandBuffer cmd=CommandBufferPool.Get("提取固有色 pass");

            // 绘制设定

            var draw=CreateDrawingSettings(shaderTag,ref renderingData,renderingData.cameraData.defaultOpaqueSortFlags);

            draw.overrideMaterial=mysetting.mymat;

            draw.overrideMaterialPassIndex=0;

            // 开始绘制（准备好了绘制设定和过滤设定）

            context.DrawRenderers(renderingData.cullResults,ref draw,ref filter);

            context.ExecuteCommandBuffer(cmd);

            CommandBufferPool.Release(cmd);

        }

    }

    // 第二个 pass 计算颜色

    class Calculate : ScriptableRenderPass

    {

        setting mysetting=null;

        SelectOutline SelectOutline=null;

        struct LEVEL//

        {

            public int down;

            public int up;

        };

        LEVEL[] my_level;

        int maxLevel=16;

        RenderTargetIdentifier sour;

        public Calculate(setting setting,SelectOutline render,RenderTargetIdentifier source)

        {

            mysetting=setting;

            SelectOutline=render;

            sour=source;

            my_level = new LEVEL[maxLevel];

            for (int t = 0; t < maxLevel; t++)// 申请 32 个 ID 的，up 和 down 各 16 个，用这个 id 去代替临时 RT 来使用

            {

                my_level[t] = new LEVEL

                {

                    down = Shader.PropertyToID("_BlurMipDown"+t),

                    up = Shader.PropertyToID("_BlurMipUp"+t)

                };

            }

        if(mysetting.ColorType== TYPE.INcolorON)

            {

            mysetting.mymat.EnableKeyword("_INCOLORON");

            mysetting.mymat.DisableKeyword("_INCOLOROFF");  

            }

        else

            {

            mysetting.mymat.EnableKeyword("_INCOLOROFF");

            mysetting.mymat.DisableKeyword("_INCOLORON");

            }

        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)

        {

            CommandBuffer cmd=CommandBufferPool.Get("颜色计算");

            RenderTextureDescriptor desc=renderingData.cameraData.cameraTargetDescriptor;

            int SourID=Shader.PropertyToID("_SourTex");

            cmd.GetTemporaryRT(SourID,desc);

            cmd.CopyTexture(sour,SourID);

            // 计算双重 kawase 模糊

            int BlurID=Shader.PropertyToID("_BlurTex");

            cmd.GetTemporaryRT(BlurID,desc);

            mysetting.mymat.SetFloat("_Blur",mysetting.blur);

            int width=desc.width/2;

            int height=desc.height/2;

            int LastDown=SelectOutline.solidcolorID;

            for(int t=0;t<mysetting.passloop;t++)

            {

                int midDown=my_level[t].down;//middle down ，即间接计算 down 的工具人 ID

                int midUp=my_level[t].up; //middle Up ，即间接计算的 up 工具人 ID

                cmd.GetTemporaryRT(midDown,width,height,0,FilterMode.Bilinear,RenderTextureFormat.ARGB32);// 对指定高宽申请 RT，每个循环的指定 RT 都会变小为原来一半

                cmd.GetTemporaryRT(midUp,width,height,0,FilterMode.Bilinear,RenderTextureFormat.ARGB32);// 同上，但是这里申请了并未计算，先把位置霸占了，这样在 UP 的循环里就不用申请 RT 了

                cmd.Blit(LastDown,midDown,mysetting.mymat,1);// 计算 down 的 pass

                LastDown=midDown;// 工具人辛苦了

                width=Mathf.Max(width/2,1);// 每次循环都降尺寸

                height=Mathf.Max(height/2,1);

            }

            //up

            int lastUp=my_level[mysetting.passloop-1].down;// 把 down 的最后一次图像当成 up 的第一张图去计算 up

            for(int j=mysetting.passloop-2;j>=0;j--)// 这里减 2 是因为第一次已经有了要减去 1，但是第一次是直接复制的，所以循环完后还得补一次 up

            {

                int midUp=my_level[j].up;

                cmd.Blit(lastUp,midUp,mysetting.mymat,2);

                lastUp=midUp;

            }

            cmd.Blit(lastUp,BlurID,mysetting.mymat,2);// 补一个 up，顺便在模糊一下

            cmd.Blit(SelectOutline.solidcolorID,sour,mysetting.mymat,3);// 在第 4 个 pass 里合并所有图像

            context.ExecuteCommandBuffer(cmd);

            // 回收

            for(int k=0;k<mysetting.passloop;k++)

            {

                cmd.ReleaseTemporaryRT(my_level[k].up);

                cmd.ReleaseTemporaryRT(my_level[k].down);

            }

            cmd.ReleaseTemporaryRT(BlurID);

            cmd.ReleaseTemporaryRT(SourID);

            cmd.ReleaseTemporaryRT(SelectOutline.solidcolorID);

            CommandBufferPool.Release(cmd);

        }

    }

    DrawSoildColorPass m_DrawSoildColorPass;

    Calculate m_Calculate;

    public override void Create()

    {

        m_DrawSoildColorPass = new DrawSoildColorPass(mysetting,this);

        m_DrawSoildColorPass.renderPassEvent = RenderPassEvent.AfterRenderingPrePasses;

    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)

    {

        if(mysetting.mymat!=null)

        {

        RenderTargetIdentifier sour= renderer.cameraColorTarget;

        renderer.EnqueuePass(m_DrawSoildColorPass);

        m_Calculate=new Calculate(mysetting,this,sour);

        m_Calculate.renderPassEvent=mysetting.passEvent;

        renderer.EnqueuePass(m_Calculate);

        }

        else

        {

            Debug.LogError("材质球丢失！请设置材质球");

        }

    }

}