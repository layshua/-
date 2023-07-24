

## 哇哦，好像很爽，那怎么用呢？

想要使用 Renderer Feature 我们要学会写两个类：

继承自 ScriptableRendererFeature 的 Feature 类。

继承自 ScriptableRenderPass 的 Pass 类。

以之前写的一个选中描边效果为例：

![[445aa015c0d135f15add7cad058c34f3_MD5.jpg]]


不写自定义的情况下，根据官方给出的 UI，**一个官方 Feature** 可以很轻易地实现：

根据物体的 Layer，在不同的流程阶段（Event）插入一次同 shader 下指定了

Tag{"LightMode" = "XXX"}

Pass 的渲染 (可多个 Pass，填写在 LightMode Tags 里)。

或用另一个材质的第 N 个 Pass 渲染一次物体。（只能指定一个 Pass，同时还可以写入深度、指定相机参数等）

这些 UI 上显示的都可以在 **RenderObjects.cs** 中找到对应代码

![[54f0c9871727de986de02f916eb9627a_MD5.jpg]]

![[4e95ba6a64aeb79a7d500292aead1001_MD5.png]]

如果官方的提供的 Feature 可以满足需求的话自然是最好，不用写任何代码也能爽用功能。

## 如何扩展呢？

当需求越来越多，Feature 面板越来越臃肿，需要更多拓展功能（如根据 RenderingLayerMask 来指定渲染对象）时，我们就需要自己写 Feature 和 Pass。（美术同学就叫程序来帮忙吧）

这里偷懒直接贴代码和注释：

Pass：

```cs
/// <summary>
    /// 自定义RenderPass
    /// </summary>
    public class OutlineRenderPass : ScriptableRenderPass
    {
        //for debugger
        private string m_ProfilerTag;
        private ProfilingSampler m_ProfilingSampler;
        private RenderStateBlock m_RenderStateBlock;

        private RenderQueueType m_renderQueueType;
        private FilteringSettings m_FilteringSettings;

        public Material overrideMaterial { get; set; }
        public int overrideMaterialPassIndex { get; set; }

        private List<ShaderTagId> m_ShaderTagIdList = new List<ShaderTagId>()
        {
            new ShaderTagId("SRPDefaultUnlit"),
            new ShaderTagId("UniversalForward"),
            new ShaderTagId("UniversalForwardOnly"),
            new ShaderTagId("LightweightForward")
        };
        //Pass的构造方法，参数都由Feature传入
        public OutlineRenderPass(string profilerTag ,RenderPassEvent renderPassEvent,FilterSettings filterSettings)
        {
            base.profilingSampler = new ProfilingSampler(nameof(FillRenderPass));
            m_ProfilerTag = profilerTag;
            m_ProfilingSampler = new ProfilingSampler(profilerTag);

            this.renderPassEvent = renderPassEvent;
            m_renderQueueType = filterSettings.renderQueueType;
            RenderQueueRange renderQueueRange = (filterSettings.renderQueueType == RenderQueueType.Transparent)
                ? RenderQueueRange.transparent
                : RenderQueueRange.opaque;
            uint renderingLayerMask = (uint)1 << filterSettings.renderingLayerMask - 1;
            m_FilteringSettings = new FilteringSettings(renderQueueRange, filterSettings.layerMask, renderingLayerMask);

            m_RenderStateBlock = new RenderStateBlock(RenderStateMask.Nothing);
        }

        public void SetDepthState(bool writeEnabled, CompareFunction function = CompareFunction.Less)
        {
            m_RenderStateBlock.mask |= RenderStateMask.Depth;
            m_RenderStateBlock.depthState = new DepthState(writeEnabled, function);
        }

        /// <summary>
        /// 最重要的方法，用来定义CommandBuffer并执行
        /// </summary>
        /// <param name="context"></param>
        /// <param name="renderingData"></param>
        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
            SortingCriteria sortingCriteria = (m_renderQueueType == RenderQueueType.Transparent)
                ? SortingCriteria.CommonTransparent
                : renderingData.cameraData.defaultOpaqueSortFlags;

            var drawingSettings = CreateDrawingSettings(m_ShaderTagIdList, ref renderingData, sortingCriteria);
            drawingSettings.overrideMaterial = overrideMaterial;
            drawingSettings.overrideMaterialPassIndex = overrideMaterialPassIndex;
            //这里不需要所以没有直接写CommandBuffer，在下面Feature的AddRenderPasses加入了渲染队列，底层还是CB
            context.DrawRenderers(renderingData.cullResults, ref drawingSettings, ref m_FilteringSettings);
        }
    }
```

Feature：

```cs
[Serializable]
    public class FilterSettings
    {
        public RenderQueueType renderQueueType;//透明还是不透明，Unity定义的enum
        public LayerMask layerMask;//渲染目标的Layer
        [Range(1, 32)] public int renderingLayerMask;//我想要指定的RenderingLayerMask

        public FilterSettings()
        {
            renderQueueType = RenderQueueType.Opaque;//默认不透明
            layerMask = -1;//默认渲染所有层
            renderingLayerMask = 32;//默认渲染32
        }
    }
    /// <summary>
    /// 自定义Feature，实现后会自动在RenderFeature面板上可供添加
    /// </summary>
    public class OutlinePassFeature : ScriptableRendererFeature
    {
        public RenderPassEvent Event = RenderPassEvent.AfterRenderingOpaques;//和官方的一样用来表示什么时候插入Pass，默认在渲染完不透明物体后
        public FilterSettings filterSettings;//上面的一些自定义过滤设置
        public Material material;//我想用的新的渲染指定物体的材质
        public int[] passes;//我想指定的几个Pass的Index
        [Space(10)]//下面三个是和Unity一样的深度设置
        public bool overrideDepthState = false;
        public CompareFunction depthCompareFunction = CompareFunction.LessEqual;
        public bool enableWrite = true;

        List<FillRenderPass> m_ScriptablePasses = new List<FillRenderPass>(2);

        /// <summary>
        /// 最重要的方法，用来生成RenderPass
        /// </summary>
        public override void Create()
        {
            if(passes == null) return;
            m_ScriptablePasses.Clear();
            //根据Shader的Pass数生成多个RenderPass
            for (int i = 0; i < passes.Length; i++)
            {
                var scriptablePass = new FillRenderPass(name, Event, filterSettings);
                scriptablePass.overrideMaterial = material;
                scriptablePass.overrideMaterialPassIndex = passes[i];

                if (overrideDepthState)
                    scriptablePass.SetDepthState(enableWrite, depthCompareFunction);

                m_ScriptablePasses.Add(scriptablePass);
            }
        }
        //添加Pass到渲染队列
        public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
        {
            if(passes == null) return;
            foreach (var pass in m_ScriptablePasses)
            {
                renderer.EnqueuePass(pass);
            }
        }
    }
```

Shader 的代码就不贴啦，直接暴风参考别的 dalao 写的就行

[acnestis：LWRP/URP/HDRP 中的多 Pass shader，以描边效果为例](https://zhuanlan.zhihu.com/p/354190065)

[大胖：使用 Render Feature 做描边效果](https://zhuanlan.zhihu.com/p/64138757)

[大胖：硬边外描边断边问题](https://zhuanlan.zhihu.com/p/130921684)

最后在 RenderFeature 面板上就会显示自定义的参数，进行简单的设置即可

![[bfbd735d965beed99f76bce79d22eaa9_MD5.jpg]]

可以看到我选择在渲染天空盒之后，把第 32 个 RenderingLayerMask 层的物体，用 Outline 这个材质的第 0 和第 1 个 Pass 重新渲染了一边。不用对物体材质进行编辑，不用添加宝贵的额外 Layer，也不用抓全屏 RT 就实现了选中物体的描边，效果如下：

是不是简简单单轻松愉快~

学会的请喵一下。

没学会可以问我，我也不是很会【

![[36eff526d3b6a4707bf547f33130d39c_MD5.gif]]