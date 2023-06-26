GIF

![[871d10f3633649756e1730e1a8d69d5e_MD5.webp]]

![[f1455871ecd17447fe2f787cc92b1380_MD5.webp]]

        这一篇的重点是 sobel 算子，关于 sobel 算子，在官方工程里有一个开源工程里有个魔改版的 sobel 算子的描边检测 https://github.com/phi-lira/UniversalShaderExamples，该算子是直接对深度图进行采样得到的描边，性能比较好，但是由于缺少法线，缺少细节；而在《入门精要》的 13.4 章里，乐乐女神通过深度和法线同时进行描边检测，得到的细节和效果更好。但是遗憾的是，在 URP 管线里，已经不支持直接获取法线深度图了。  

![[95f323eff00dc5532a488e14e60551f2_MD5.webp]]

虽然 urp 不在直接支持了，但是这篇文章里（https://alexanderameye.github.io/outlineshader）有提到过 URP 里获取法线深度图的方式，它是通过额外定义一个 RenderFeather 去插在 PrePasses 之后，不透明物体渲染之前，把场景的物体覆盖上一个 "Hidden/Internal-DepthNormalsTexture" 的 shader 的材质进行渲染，得到一张图渲染到名为 “_CameraDepthNormalsTexture”RT 里发送给全局的 shader。然后就可以像 build 里一样去使用这张图，我们可以把这张图在 Frame Debug 里查看到。

![[82c3ef8ddd90e6a30d2d4d5c41e96ea5_MD5.webp]]

        得到的这张图不仅包含了法线，还额外包含了深度信息，索性就直接弃用_CameraDepthTexture 而使用它来得到深度信息，岂不美哉。这张图的 xy 通道存了法线，zw 通道存了深度，我们需要通过它得到正常的线性深度。而获得的方式很简单，查一下 unityCG 的解码公式，即可知线性深度 = z+w/255，我们把它输出到屏幕看看，看着没啥毛病。

![[19d9d399d55f63119748f3ed9b2f449c_MD5.webp]]

        而关于法线部分是存到 xy 通道的，这个 xy 并非真实的法线的 xy 值，需要进行解码后才能得到正常的法线值，它的解码函数如下，它定义在 unityCG 里，直接在 hlsl 里用会报错。  

![[5c0959e8e02c5abb6a4b03f840ceaaad_MD5.webp]]

        本次特效里会用的法线地方就只有 sobel 算子检测，而解码后的法线是和 xy 成正比（本人已经求证过了，有兴趣的读者可以自行推导证明），所以我们无需对 xy 进行解码，直接使用 xy 即可用来作为 sobel 的检测算子。然后我们定义一个算子检测函数，采样了当前像素的四个对角线的值，如果深度差异或者法线差异过大都会被认定是边缘。

![[0e7f9739a9580a76534ee773c1cbe89c_MD5.webp]]

        我们把检测的算子输出看看效果。细节很多，但是也有一些我们不想要的细节出现了，读者可以根据自己喜好调整采样范围和灵敏度，或者修改 sobel 算子检测函数。

![[f68216b07de86a85756c9a1a8f2a9a5f_MD5.webp]]

我们给检测到的边缘指定一个颜色，然后和屏幕图像混合，输出看看效果。

![[08ea48412e7da06a3b1f901ba94683d4_MD5.webp]]

        边缘检测终于搞定了，下面再说一下以世界空间坐标作为遮罩制作一个动画。把世界坐标当成 uv 坐标去做一个 mask 图，该原理比较简单，有映像的读者还记得该原理在这篇文章里用过一次 [https://www.bilibili.com/read/cv6519977](https://www.bilibili.com/read/cv6519977)。本次我定义了一个 multi_compile_local，去在世界坐标的三个轴向去切换 mask 图的滚动方向，然后在 RenderFeather 里去实时修改。

![[3834bc5e5b66836a3dbf4a439cd75a9b_MD5.webp]]

            根据不同的关键字，定义不同的 mask 滚动。

![[36ccff9e4d39fa0030c687397d3bf424_MD5.webp]]

GIF

![[5341e1c35d11b18706b0d38a96472ea0_MD5.webp]]

z 方向

GIF

![[9ac9eb67071311b90b46507b91d7ec7a_MD5.webp]]

Y 方向

把 mask 图输出，就可以得到这样一张遮罩，当然读者还可以加点培林噪波或者维诺德噪波丰富一下细节。  

![[042402eab80e213473b1cba5fbad412b_MD5.webp]]

        最后合并图像，我希望在有 mask 图扫过的地方，tex 的亮度压暗，所以我这样定义了输出公式。

![[c61c78915afee023e480b0d8e75313ab_MD5.webp]]

        图像输出后，还需要读者自行调整材质参数。该图效果有地方明显过曝了，特效大忌，大忌！

![[d4febbcb51678a055d0dae6cdfa427d6_MD5.webp]]

一个过曝的特效，猝

如果让他沿着 Y 轴进行运动的话......emmm，懒得调了就这样吧。

![[e30b367c4e8f0769afd342037ed16ae9_MD5.webp]]

 这次熟悉了下在 urp 里获取深度法线的方式，顺便复习了下 sobel 算子检测，整体原理呢，就是不停叠加效果，最终得到我们想要的效果，读者可以根据自己的想法做一些骚操作，比如上一篇里的提到的世界坐标系就是个很好的操作对象（开始迫害 WSpos....），下面再附上源码。

using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

public class DepthNormalsFeature : ScriptableRendererFeature

{

    class DepthNormalsPass : ScriptableRenderPass

    {

        private RenderTargetHandle destination {get; set;}

        private Material depthNormalsMaterial = null;

        private FilteringSettings m_FilteringSettings;

        ShaderTagId m_ShaderTagId = new ShaderTagId("DepthOnly");

        public DepthNormalsPass(RenderQueueRange renderQueueRange, LayerMask layerMask, Material material)

        {

            m_FilteringSettings = new FilteringSettings(renderQueueRange, layerMask);

            this.depthNormalsMaterial = material;

        }

        public void Setup(RenderTargetHandle destination)

        {

            this.destination = destination;

        }

        // This method is called before executing the render pass.

        // It can be used to configure render targets and their clear state. Also to create temporary render target textures.

        // When empty this render pass will render to the active camera render target.

        // You should never call CommandBuffer.SetRenderTarget. Instead call <c>ConfigureTarget</c> and <c>ConfigureClear</c>.

        // The render pipeline will ensure target setup and clearing happens in an performance manner.

        public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)

        {

            RenderTextureDescriptor descriptor = cameraTextureDescriptor;

            descriptor.depthBufferBits = 32;

            descriptor.colorFormat = RenderTextureFormat.ARGB32;

            cmd.GetTemporaryRT(destination.id, descriptor, FilterMode.Point);

            ConfigureTarget(destination.Identifier());

            ConfigureClear(ClearFlag.All, Color.black);

        }

        // Here you can implement the rendering logic.

        // Use <c>ScriptableRenderContext</c> to issue drawing commands or execute command buffers

        // https://docs.unity3d.com/ScriptReference/Rendering.ScriptableRenderContext.html

        // You don't have to call ScriptableRenderContext.submit, the render pipeline will call it at specific points in the pipeline.

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)

        {

            CommandBuffer cmd = CommandBufferPool.Get("深度法线获取 pass");

            using (new ProfilingSample(cmd, "DepthNormals Prepass"))

            {

                context.ExecuteCommandBuffer(cmd);

                cmd.Clear();

                var sortFlags = renderingData.cameraData.defaultOpaqueSortFlags;

                var drawSettings = CreateDrawingSettings(m_ShaderTagId, ref renderingData, sortFlags);

                drawSettings.perObjectData = PerObjectData.None;

                ref CameraData cameraData = ref renderingData.cameraData;

                Camera camera = cameraData.camera;

                if (cameraData.isStereoEnabled)

                    context.StartMultiEye(camera);

                drawSettings.overrideMaterial = depthNormalsMaterial;

                context.DrawRenderers(renderingData.cullResults, ref drawSettings,

                    ref m_FilteringSettings);

                cmd.SetGlobalTexture("_CameraDepthNormalsTexture", destination.id);

            }

            context.ExecuteCommandBuffer(cmd);

            CommandBufferPool.Release(cmd);

        }

        /// Cleanup any allocated resources that were created during the execution of this render pass.

        public override void FrameCleanup(CommandBuffer cmd)

        {

            if (destination != RenderTargetHandle.CameraTarget)

            {

                cmd.ReleaseTemporaryRT(destination.id);

                destination = RenderTargetHandle.CameraTarget;

            }

        }

    }

    DepthNormalsPass depthNormalsPass;

    RenderTargetHandle depthNormalsTexture;

    Material depthNormalsMaterial;

    public override void Create()

    {

        depthNormalsMaterial = CoreUtils.CreateEngineMaterial("Hidden/Internal-DepthNormalsTexture");

        depthNormalsPass = new DepthNormalsPass(RenderQueueRange.opaque, -1, depthNormalsMaterial);

        depthNormalsPass.renderPassEvent = RenderPassEvent.AfterRenderingPrePasses;

        depthNormalsTexture.Init("_CameraDepthNormalsTexture");

    }

    // Here you can inject one or multiple render passes in the renderer.

    // This method is called when setting up the renderer once per-camera.

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)

    {

        depthNormalsPass.Setup(depthNormalsTexture);

        renderer.EnqueuePass(depthNormalsPass);

    }

}

**Renderer Feature 扫描效果**

using UnityEngine;

using UnityEngine.Rendering;

using UnityEngine.Rendering.Universal;

[ExecuteInEditMode]

public class scan : ScriptableRendererFeature

{

    public enum ax

    {

        X,

        Y,

        Z

    }

    [System.Serializable]public class setting

    {

        public Material mat=null;

        public RenderPassEvent Event=RenderPassEvent.AfterRenderingTransparents;

        [ColorUsage(true,true)]public Color ColorX=Color.white;

        [ColorUsage(true,true)]public Color ColorY=Color.white;

        [ColorUsage(true,true)]public Color ColorZ=Color.white;

        [ColorUsage(true,true)]public Color ColorEdge=Color.white;

        [ColorUsage(true,true)]public Color ColorOutline=Color.white;

        [Range(0,0.2f),Tooltip("线框宽度")]public float Width = 0.1f;

        [Range(0.1f, 10), Tooltip("线框间距")] public float Spacing = 1;

        [Range(0, 10), Tooltip("滚动速度")] public float Speed = 1;

        [Range(0, 3), Tooltip("边缘采样半径")] public float EdgeSample = 1;

        [Range(0, 3), Tooltip("法线灵敏度")] public float NormalSensitivity = 1;

        [Range(0, 3), Tooltip("深度灵敏度")] public float DepthSensitivity = 1;

        [Tooltip("特效方向")]public ax AXIS;

    }

    public setting mysetting =new setting();

    class CustomRenderPass : ScriptableRenderPass

    {

        public Material mat=null;

        RenderTargetIdentifier sour;

         public Color ColorX = Color.white;

        public Color ColorY = Color.white;

        public Color ColorZ = Color.white;

        public Color ColorEdge = Color.white;

        public Color ColorOutline = Color.white;

        public float Width = 0.05f;

        public float Spacing = 2;

        public float Speed = 0.7f;

        public float EdgeSample = 1;

        public float NormalSensitivity = 1;

        public float DepthSensitivity = 1;

        public ax AXIS;

        public void set(RenderTargetIdentifier sour)

        {

            this.sour=sour;

            mat.SetColor("_colorX", ColorX);

            mat.SetColor("_colorY", ColorY);

            mat.SetColor("_colorZ", ColorZ);

            mat.SetColor("_ColorEdge", ColorEdge);

            mat.SetColor("_OutlineColor", ColorOutline);

            mat.SetFloat("_width", Width);

            mat.SetFloat("_Spacing", Spacing);

            mat.SetFloat("_Speed", Speed);

            mat.SetFloat("_EdgeSample", EdgeSample);

            mat.SetFloat("_NormalSensitivity", NormalSensitivity);

            mat.SetFloat("_DepthSensitivity", DepthSensitivity);

            if (AXIS == ax.X)

            {

                mat.DisableKeyword("_AXIS_Y");

                mat.DisableKeyword("_AXIS_Z");

                mat.EnableKeyword("_AXIS_X");

            }

            else if (AXIS == ax.Y)

            {

                mat.DisableKeyword("_AXIS_Z");

                mat.DisableKeyword("_AXIS_X");

                mat.EnableKeyword("_AXIS_Y");

            }

            else

            {

                mat.DisableKeyword("_AXIS_X");

                mat.DisableKeyword("_AXIS_Y");

                mat.EnableKeyword("_AXIS_Z");

            }

        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)

        {

            int temp=Shader.PropertyToID("temp");

            CommandBuffer cmd=CommandBufferPool.Get("扫描特效");

            RenderTextureDescriptor desc=renderingData.cameraData.cameraTargetDescriptor;

            Camera cam= renderingData.cameraData.camera;

            float height=cam.nearClipPlane*Mathf.Tan(Mathf.Deg2Rad*cam.fieldOfView*0.5f);

            Vector3 up=cam.transform.up*height;

            Vector3 right=cam.transform.right*height*cam.aspect;

            Vector3 forward=cam.transform.forward*cam.nearClipPlane;

            Vector3 ButtomLeft=forward-right-up;

            float scale=ButtomLeft.magnitude/cam.nearClipPlane;

            ButtomLeft.Normalize();

            ButtomLeft*=scale;

            Vector3 ButtomRight=forward+right-up;

            ButtomRight.Normalize();

            ButtomRight*=scale;

            Vector3 TopRight=forward+right+up;

            TopRight.Normalize();

            TopRight*=scale;

            Vector3 TopLeft=forward-right+up;

            TopLeft.Normalize();

            TopLeft*=scale;

            Matrix4x4 MATRIX=new Matrix4x4();

            MATRIX.SetRow(0,ButtomLeft);

            MATRIX.SetRow(1,ButtomRight);

            MATRIX.SetRow(2,TopRight);

            MATRIX.SetRow(3,TopLeft);

            mat.SetMatrix("Matrix",MATRIX);

            cmd.GetTemporaryRT(temp,desc);

            cmd.Blit(sour,temp,mat);

            cmd.Blit(temp,sour);

            context.ExecuteCommandBuffer(cmd);

            cmd.ReleaseTemporaryRT(temp);

            CommandBufferPool.Release(cmd);

        }

    }

    CustomRenderPass m_ScriptablePass;

    public override void Create()

    {

        m_ScriptablePass = new CustomRenderPass();

        m_ScriptablePass.mat=mysetting.mat;

        m_ScriptablePass.renderPassEvent = mysetting.Event;

        m_ScriptablePass.ColorX = mysetting.ColorX;

        m_ScriptablePass.ColorY = mysetting.ColorY;

        m_ScriptablePass.ColorZ = mysetting.ColorZ;

        m_ScriptablePass.ColorEdge = mysetting.ColorEdge;

        m_ScriptablePass.ColorOutline = mysetting.ColorOutline;

        m_ScriptablePass.Width = mysetting.Width;

        m_ScriptablePass.Spacing = mysetting.Spacing;

        m_ScriptablePass.Speed = mysetting.Speed;

        m_ScriptablePass.EdgeSample = mysetting.EdgeSample;

        m_ScriptablePass.NormalSensitivity = mysetting.NormalSensitivity;

        m_ScriptablePass.DepthSensitivity = mysetting.DepthSensitivity;

        m_ScriptablePass.AXIS = mysetting.AXIS;

    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)

    {

        m_ScriptablePass.set(renderer.cameraColorTarget);

        renderer.EnqueuePass(m_ScriptablePass);

    }

}

**Shader 源码**

Shader "WX/URP/Post/scan"

{

    Properties

    {

      [HideInInspector]_MainTex("MainTex",2D)="white"{}

      [HideInInspector][HDR]_colorX("ColorX",Color)=(1,1,1,1)

      [HideInInspector][HDR]_colorY("ColorY",Color)=(1,1,1,1)

      [HideInInspector][HDR]_ColorZ("ColorZ",Color)=(1,1,1,1)

      [HideInInspector][HDR]_ColorEdge("ColorEdge",Color)=(1,1,1,1)

      [HideInInspector]_width("Width",float)=0.1

      [HideInInspector]_Spacing("Spacing",float)=1

      [HideInInspector]_Speed("Speed",float)=1

      [HideInInspector]_EdgeSample("EdgeSample",Range(0,1))=1

      [HideInInspector]_NormalSensitivity("NormalSensitivity",float)=1

      [HideInInspector]_DepthSensitivity("DepthSensitivity",float)=1

      [HideInInspector][HDR]_OutlineColor("OutlineColr",Color)=(1,1,1,1)

      //[KeywordEnum(X,Y,Z)]_AXIS("Axis",float)=1

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

        CBUFFER_END

        real4 _colorX;

        real4 _colorY;

        real4 _ColorZ;

        real4 _ColorEdge;

        real4 _OutlineColor;

        float _width;

        float _Spacing;

        float _Speed;

        float _EdgeSample;

        float _NormalSensitivity;

        float _DepthSensitivity;

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

        //TEXTURE2D(_CameraDepthTexture);// 不需要这张图了

        //SAMPLER(sampler_CameraDepthTexture);// 不需要这张图了

        TEXTURE2D(_CameraDepthNormalsTexture);

        SAMPLER(sampler_CameraDepthNormalsTexture);

        float4x4 Matrix;

         struct a2v

         {

             float4 positionOS:POSITION;

             float2 texcoord:TEXCOORD;

         };

         struct v2f

         {

             float4 positionCS:SV_POSITION;

             float2 texcoord:TEXCOORD;

             float3 Dirction:TEXCOORD1;

         };

        ENDHLSL

        pass

        {

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            #pragma  multi_compile_local _AXIS_X _AXIS_Y _AXIS_Z 

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=i.texcoord;

                int t=0;

                if(i.texcoord.x<0.5&&i.texcoord.y<0.5)

                t=0;

                else if(i.texcoord.x>0.5&&i.texcoord.y<0.5)

                t=1;

                else if(i.texcoord.x>0.5&&i.texcoord.y>0.5)

                t=2;

                else

                t=3;

                o.Dirction=Matrix[t].xyz;

                return o;

            }  

            int sobel(v2f i);

            real4 FRAG(v2f i):SV_TARGET

            {

            int outline=sobel(i);

            //return outline;

            real4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord);

            //return  lerp(tex,_OutlineColor,outline);

            real4 depthnormal= SAMPLE_TEXTURE2D(_CameraDepthNormalsTexture,sampler_CameraDepthNormalsTexture,i.texcoord);

            float depth01= depthnormal.z*1.0+depthnormal.w/255.0;// 得到 01 线性的深度

            //return depth01;

            float3 WSpos= _WorldSpaceCameraPos+depth01*i.Dirction*_ProjectionParams.z;// 这样也可以得到正确的世界坐标

            //return real4(frac(ws),1);

                float3 WSpos01=WSpos*_ProjectionParams.w;

                float3 Line=step(1-_width,frac(WSpos/_Spacing));// 线框

                float4 Linecolor=Line.x*_colorX+Line.y*_colorY+Line.z*_ColorZ+outline*_OutlineColor;// 计算线框颜色

                //return Linecolor+tex;

                #ifdef _AXIS_X

                float mask=saturate(pow(abs(frac(WSpos01.x+_Time.y*0.1*_Speed)-0.75),10)*30);// 在 X 轴方向计算 mask

                mask+=step(0.999,mask);

                #elif _AXIS_Y

                float mask=saturate(pow(abs(frac(WSpos01.y-_Time.y*0.1*_Speed)-0.25),10)*30);// 在 Y 轴方向计算 mask

                mask+=step(0.999,mask);

                #elif _AXIS_Z

                float mask=saturate(pow(abs(frac(WSpos01.z+_Time.y*0.1*_Speed)-0.75),10)*30);// 在 Z 轴方向计算 mask

                mask+=step(0.999,mask);

                #endif

                //return mask;

                return tex*saturate(1-mask)+(Linecolor+_ColorEdge)*mask;

            }

            int sobel(v2f i)// 定义索伯检测函数

            {

                real depth[4];

                real2 normal[4];

                float2 uv[4];// 计算采样需要的 uv

                uv[0]=i.texcoord+float2(-1,-1)*_EdgeSample*_MainTex_TexelSize.xy;

                uv[1]=i.texcoord+float2(1,-1)*_EdgeSample*_MainTex_TexelSize.xy;

                uv[2]=i.texcoord+float2(-1,1)*_EdgeSample*_MainTex_TexelSize.xy;

                uv[3]=i.texcoord+float2(1,1)*_EdgeSample*_MainTex_TexelSize.xy;

                for(int t=0;t<4;t++)

                {

                real4 depthnormalTex=SAMPLE_TEXTURE2D(_CameraDepthNormalsTexture,sampler_CameraDepthNormalsTexture,uv[t]);

                    normal[t]=depthnormalTex.xy;// 得到临时法线

                    depth[t]=depthnormalTex.z*1.0+depthnormalTex.w/255.0;// 得到线性深度

                }

                //depth 检测

                int Dep=abs(depth[0]-depth[3])*abs(depth[1]-depth[2])*_DepthSensitivity>0.01?1:0;

                //normal 检测

                float2 nor=abs(normal[0]-normal[3])*abs(normal[1]-normal[2])*_NormalSensitivity;

                int Nor=(nor.x+nor.y)>0.01?1:0;

                return saturate(Dep+Nor);

            }

            ENDHLSL

        }

    }

}