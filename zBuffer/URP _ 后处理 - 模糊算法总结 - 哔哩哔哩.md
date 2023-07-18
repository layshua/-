内容偏多 

![[d609e2f91c6ea168755329c5bfe98b59_MD5.webp]]

*   URP 管线下怎么计算后期模糊效果。
    
*   模糊算法的学习使用。
    
*   都有那些模糊算法，都有哪些优点。
    

目前看到大佬总结的十种模糊的方式，实现经常使用的几种。

高品质后处理：十种图像模糊算法的总结与实现 - 知乎 (zhihu.com)

*   10 种模糊方式对比
    

![[ab3d54e972ed36232d450ee83000f115_MD5.webp]]

不同模糊性能对比。

![[68fb7c1361dbf3c586356b35dfac4846_MD5.webp]]


高斯模糊主要分为横模糊和纵模糊，俩次模糊以后就可以得到结果了。

*   对图像处理的矩阵称之为高斯核（Gaussian Kernel）
    

![[5a96ad5b0448a0370368e18f9bae79f7_MD5.webp]]

详细 参考

高品质后处理：十种图像模糊算法的总结与实现 - 知乎 (zhihu.com)

*   创建一个 Shader，在创建一个函数库，命名 Blur.hlsl
    

![[291a7752b1da1e1cf768b09e3de89f37_MD5.webp]]

*   Shader 内导入函数库就可以了，
    

```
Shader "URP/1_GaussianBlur"
{
    Properties
    {
        _MainTex ("_MainTex", 2D) = "white" {}
        _BlurRange("_BlurRange",Float) = 0
    }
    SubShader
    {
        Tags { "RenderPipeline"="UniversalPipeline" }
         Cull Off ZWrite Off ZTest Always


        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment Gaussianfrag

            #include "Blur.hlsl"           //函数库

            ENDHLSL
        }
    }
}
```

*   函数库里增加计算
    

```
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

CBUFFER_START(UnityPerMaterial)
float4 _MainTex_ST;
float _BlurRange;
float blurrange;
CBUFFER_END

TEXTURE2D(_MainTex);             SAMPLER(sampler_MainTex);

struct appdata
{
    float4 positionOS : POSITION;
    float2 texcoord : TEXCOORD0;
};

struct v2f
{
    float2 uv : TEXCOORD0;
    float4 vertex : SV_POSITION;

};

v2f vert(appdata v)
{
    v2f o;
    o.vertex = TransformObjectToHClip(v.positionOS.xyz);
    o.uv = v.texcoord;
    return o;
}

half4 Gaussianfrag(v2f i) : SV_Target
{

    float4 col = float4(0, 0, 0, 0);
    blurrange = _BlurRange / 300;

    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(0.0, 0.0)) * 0.147716f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(blurrange, 0.0)) * 0.118318f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(0.0, -blurrange)) * 0.118318f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(0.0, blurrange)) * 0.118318f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(-blurrange, 0.0)) * 0.118318f;

    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(blurrange, blurrange)) * 0.0947416f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(-blurrange, -blurrange)) * 0.0947416f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(blurrange, -blurrange)) * 0.0947416f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(-blurrange, blurrange)) * 0.0947416f;

    return col;
}
```

*   定义我们的片元着色器（Gaussianfrag），
    

**注意：因为我们要制作不同的模糊效果使用一个 hlsl. 函数库，调用不同的片元着色器，执行不同的效果。**

扩展 卷积核是什么？

三分钟彻底理解图像高斯模糊 - 知乎 (zhihu.com)

[卷积究竟卷了啥？——17 分钟了解什么是卷积_哔哩哔哩_bilibili](https://www.bilibili.com/video/av713651125/)

[游戏 Bloom 实现方法 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv12792138)

创建一个脚本在后处理里控制属性

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class GaussianBlurVolume : VolumeComponent, IPostProcessComponent
{

    [Range(0f, 10f), Tooltip("模糊的迭代次数")]
    public IntParameter      BlurTimes = new ClampedIntParameter(1, 0, 10);
    [Range(0f, 10f), Tooltip("模糊半径")]
    public FloatParameter    BlurRange = new ClampedFloatParameter(1.0f, 0.0f, 10.0f);
    [Range(0f, 10f), Tooltip("降采样次数")]
    public IntParameter RTDownSampling = new ClampedIntParameter(1, 1, 10);

    public bool IsActive() => RTDownSampling.value > 0f;

    public bool IsTileCompatible() => false;

}
```

*   后处理效果
    

![[a94ddadac90ef8aa25038bf4947631a8_MD5.webp]]

后处理组件

*   创建一个脚本，命名 **GaussianBlurRenderFeature**
    

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class GaussianBlurRenderFeature : ScriptableRendererFeature
{
    public override void Create()
    {

    }
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {

    }
}

public class GaussianBlurPass : ScriptableRenderPass
{
    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {

    }
}
```

*   RenderFeature 是处理渲染逻辑，设置 Shader
    
*   设置渲染事件
    
*   同步渲染事件
    
*   执行函数，传入 Volume，传入 Command
    
*   渲染 Render 函数，后处理逻辑。
    

**公开设置**

*   外部可以设置渲染层级，指定 Shader，
    

```
[System.Serializable]
    public class Settings
    {
        public RenderPassEvent renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
        public Shader shader;
    }
    public Settings settings = new Settings();

GaussianBlurPass gaussianBlurPass;           // 定义我们创建出Pass
```

[URP | 后处理 - 自定义后处理 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv17805609/?from=readlist)

*   定义 Pass 我们创建的 Pass 开始同步渲染方式。
    

```
public override void Create()
    {
        this.name = "GaussianBlurPass";    // 模糊渲染的名字
        gaussianBlurPass = new GaussianBlurPass(RenderPassEvent.BeforeRenderingPostProcessing, settings.shader);    // 初始化 我们的渲染层级和Shader

    }
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {

    }
```

**PASS 部分**

**渲染事件**

我们开始处理 Pass 阶段的渲染内容，

*   创建一个类， 创建构造函数。
    

```
public class GaussianBlurPass : ScriptableRenderPass
{


    public GaussianBlurPass(RenderPassEvent evt, Shader blurshader)
    {
        renderPassEvent = evt;
        var shader = blurshader;
    }

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {

    }


}
```

就是上面调用的构造函数设置的层级和 Shader

*   我们现在定义一些变量，比如 Shader 的材质，控制模糊的变量等。
    

```
static readonly string RenderTag = " Gaussian Effects";                         // 设置渲染标签

    static readonly int TempTargetId = Shader.PropertyToID("_TempTargetGaussian");                     // 设置储存图像信息
    static readonly int MainTexId = Shader.PropertyToID("_MainTex");                // 主贴图

    GaussianBlurVolume gaussianBlur;                                                // 定义组件类型

    Material Gaussianmaterial;                                                      // 后处理材质
    RenderTargetIdentifier renderTargetIdentifier;                                  // 设置当前渲染目标
```

*   我们回到到构造函数，判断 Shader 是否为什么空，如果不为空创建一个材质。  
    

```
public GaussianBlurPass(RenderPassEvent evt, Shader blurshader)
    {
        renderPassEvent = evt;
        var shader = blurshader;

        if (shader = null)
        {
            Debug.LogError("没有指定Shader");
            return;
        }
        Gaussianmaterial = CoreUtils.CreateEngineMaterial(blurshader);
    }
```

**初始化渲染**

*   定义一个 Setup 函数 ，初始化渲染目标
    

```
public void Setup(in RenderTargetIdentifier renderTargetIdentifier)
    {
        this.renderTargetIdentifier = renderTargetIdentifier;
    }
```

回到 RendererFeature 初始化

```
public class GaussianBlurRenderFeature : ScriptableRendererFeature
{
    [System.Serializable]
    public class Settings
    {
        public RenderPassEvent renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
        public Shader shader;
    }
    public Settings settings = new Settings();

    GaussianBlurPass gaussianBlurPass;           // 定义我们创建出Pass

    public override void Create()
    {
        this.name = "GaussianBlurPass";    // 模糊渲染的名字
        gaussianBlurPass = new GaussianBlurPass(RenderPassEvent.BeforeRenderingPostProcessing, settings.shader);    // 初始化 我们的渲染层级和Shader

    }
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        gaussianBlurPass.Setup(renderer.cameraColorTarget);   // 初始化
        renderer.EnqueuePass(gaussianBlurPass);
    }
}
```

初始化完成，

**执行**

*   主要是判断 材质，摄像机， 组件是否准备好，准备好执行渲染。
    

```
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        if (Gaussianmaterial == null)
        {
            Debug.LogError("材质初始化失败");
            return;
        }

        if (!renderingData.cameraData.postProcessEnabled)
        {
            return;
        }

        var stack = VolumeManager.instance.stack;                                    // 传入 volume
        gaussianBlur = stack.GetComponent<GaussianBlurVolume>();                     // 获取到后处理组件

        if (gaussianBlur == null)
        {
            Debug.LogError("获取组件失败");
            return;
        }

        var cmd = CommandBufferPool.Get(RenderTag);    // 渲染标签

        Render(cmd, ref renderingData);                 // 调用渲染函数

        context.ExecuteCommandBuffer(cmd);              // 执行函数，回收。
        CommandBufferPool.Release(cmd);

    }
```

**注意：这里的 Render 函数还没有定义**

接下来定义渲染部分。

**渲染**

*   渲染
    

```
void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        ref var cameraData = ref renderingData.cameraData;      // 获取摄像机属性
        var camera = cameraData.camera;                         // 传入摄像机
        var source = renderTargetIdentifier;                    // 获取渲染图片
        int destination = TempTargetId;                         // 渲染结果图片

        Gaussianmaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);             // Shader变量  和 Volume 组件属性 绑定

    }
```

执行渲染渲染计算，

```
ref var cameraData = ref renderingData.cameraData;      // 获取摄像机属性
        var camera = cameraData.camera;                         // 传入摄像机
        var source = renderTargetIdentifier;                    // 获取渲染图片
        int destination = TempTargetId;                         // 渲染结果图片

        Gaussianmaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);             // Shader变量  和 Volume 组件属性 绑定

        cmd.SetGlobalTexture(MainTexId, source);                                           // 渲染原图 储存到 source
        cmd.GetTemporaryRT(
            destination,
            cameraData.camera.scaledPixelWidth,
            cameraData.camera.scaledPixelHeight,
            0,
            FilterMode.Trilinear,
            RenderTextureFormat.Default
            );
```

**注意：GetTemporaryRT 是括号括号括号，这样处理是为了看的清楚。**

*   计算完，传入材质
    

```
void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        ref var cameraData = ref renderingData.cameraData;      // 获取摄像机属性
        var camera = cameraData.camera;                         // 传入摄像机
        var source = renderTargetIdentifier;                    // 获取渲染图片
        int destination = TempTargetId;                         //  Shader中的属性绑定

// 定义屏幕尺寸
        var width = (int)(camera.scaledPixelWidth / gaussianBlur.RTDownSampling.value);
        var height = (int)(camera.scaledPixelHeight / gaussianBlur.RTDownSampling.value);


        Gaussianmaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);             // Shader变量  和 Volume 组件属性 绑定

        cmd.SetGlobalTexture(MainTexId, source);                                           // 渲染原图 储存到 source
        cmd.GetTemporaryRT(                                                                // 创建临时RT，对临时RT降采样，
            destination,
            width,
            height,
            0,
            FilterMode.Trilinear,
            RenderTextureFormat.Default
            );

        // 计算模糊
        for (int i = 0; i < gaussianBlur.BlurTimes.value; i++)
        {
            cmd.Blit(source, destination);                         //  前一次的渲染传入，
            cmd.Blit(destination, source, Gaussianmaterial, 0);    //  第二个Pass
        }

    }
```

我们使用 BlurTimes 控制迭代次数。

扩展 Blit 函数的使用用法、

```
cmd.Blit(destination, source, Gaussianmaterial, 0);
         // 原数据     现在      材质球       第几个Pass
```

这个表示 后面是表示使用的 Shader 中第几个 Pass 。

**Blit 函数的使用**

*   官方函数介绍
    

![[56bc5620314281af7d4bf3363fb066c9_MD5.webp]]

最简单的作用就是把一张纹理绘制到另一张纹理中，可以使用材质对纹理进行处理。

**效果**

*   处理效果
    

GIF

![[1cf30755be9af3f5dacdbdd5263d28ed_MD5.webp]]

**注意：迭代次数越多越费。**

**代码**

shader

```
Shader "URP/1_GaussianBlur"
{
    Properties
    {
        _MainTex ("_MainTex", 2D) = "white" {}
        _BlurRange("_BlurRange",Float) = 0
    }
    SubShader
    {
        Tags { "RenderPipeline"="UniversalPipeline" }
         Cull Off ZWrite Off ZTest Always


        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment Gaussianfrag

            #include "Blur.hlsl"           //函数库

            ENDHLSL
        }
    }
}
```

hlsl  

```
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

CBUFFER_START(UnityPerMaterial)
float4 _MainTex_ST;
float _BlurRange;
float blurrange;
CBUFFER_END

TEXTURE2D(_MainTex);SAMPLER(sampler_MainTex);

struct appdata
{
    float4 positionOS : POSITION;
    float2 texcoord : TEXCOORD0;
};

struct v2f
{
    float2 uv : TEXCOORD0;
    float4 vertex : SV_POSITION;

};

v2f vert(appdata v)
{
    v2f o;
    o.vertex = TransformObjectToHClip(v.positionOS.xyz);
    o.uv = v.texcoord;
    return o;
}

half4 frag(v2f i) : SV_Target
{

    float4 col = float4(0, 0, 0, 0);
    blurrange = _BlurRange / 300;

    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(0.0, 0.0)) * 0.147716f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(blurrange, 0.0)) * 0.118318f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(0.0, -blurrange)) * 0.118318f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(0.0, blurrange)) * 0.118318f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(-blurrange, 0.0)) * 0.118318f;

    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(blurrange, blurrange)) * 0.0947416f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(-blurrange, -blurrange)) * 0.0947416f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(blurrange, -blurrange)) * 0.0947416f;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(-blurrange, blurrange)) * 0.0947416f;

    return col;
}
```

Volume

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class GaussianBlurVolume : VolumeComponent, IPostProcessComponent
{

    [Range(0f, 10f), Tooltip("模糊的迭代次数")]
    public IntParameter      BlurTimes = new ClampedIntParameter(1, 0, 10);
    [Range(0f, 10f), Tooltip("模糊半径")]
    public FloatParameter    BlurRange = new ClampedFloatParameter(1.0f, 0.0f, 10.0f);
    [Range(0f, 10f), Tooltip("降采样次数")]
    public IntParameter RTDownSampling = new ClampedIntParameter(1, 1, 10);

    public bool IsActive() => RTDownSampling.value > 0f;

    public bool IsTileCompatible() => false;

}
```

Feature

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class GaussianBlurRenderFeature : ScriptableRendererFeature
{
    [System.Serializable]
    public class Settings
    {
        public RenderPassEvent renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
        public Shader shader;
    }
    public Settings settings = new Settings();

    GaussianBlurPass gaussianBlurPass;           // 定义我们创建出Pass

    public override void Create()
    {
        this.name = "GaussianBlurPass";    // 模糊渲染的名字
        gaussianBlurPass = new GaussianBlurPass(RenderPassEvent.BeforeRenderingPostProcessing, settings.shader);    // 初始化 我们的渲染层级和Shader

    }
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        gaussianBlurPass.Setup(renderer.cameraColorTarget);   // 初始化
        renderer.EnqueuePass(gaussianBlurPass);
    }
}

public class GaussianBlurPass : ScriptableRenderPass
{
    static readonly string RenderTag = " Gaussian Effects";                         // 设置渲染标签

    static readonly int TempTargetId = Shader.PropertyToID("_TempTargetGaussian");                     // 设置储存图像信息
    static readonly int MainTexId = Shader.PropertyToID("_MainTex");                // 主贴图

    GaussianBlurVolume gaussianBlur;                                                // 定义组件类型

    Material Gaussianmaterial;                                                      // 后处理材质
    RenderTargetIdentifier renderTargetIdentifier;                                  // 设置当前渲染目标

    public GaussianBlurPass(RenderPassEvent evt, Shader blurshader)
    {
        renderPassEvent = evt;
        var shader = blurshader;

        if (shader == null)
        {
            Debug.LogError("没有指定Shader");
            return;
        }
        Gaussianmaterial = CoreUtils.CreateEngineMaterial(blurshader);
    }

    public void Setup(in RenderTargetIdentifier renderTargetIdentifier)
    {
        this.renderTargetIdentifier = renderTargetIdentifier;
    }

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        if (Gaussianmaterial == null)
        {
            Debug.LogError("材质初始化失败");
            return;
        }

        if (!renderingData.cameraData.postProcessEnabled)
        {
            Debug.LogError("摄像机是否开启后处理");
            return;
        }

        var stack = VolumeManager.instance.stack;                                    // 传入 volume
        gaussianBlur = stack.GetComponent<GaussianBlurVolume>();                     // 获取到后处理组件

        if (gaussianBlur == null)
        {
            Debug.LogError("获取组件失败");
            return;
        }

        var cmd = CommandBufferPool.Get(RenderTag);    // 渲染标签

        Render(cmd, ref renderingData);                 // 调用渲染函数

        context.ExecuteCommandBuffer(cmd);              // 执行函数，回收。
        CommandBufferPool.Release(cmd);

    }

    void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        ref var cameraData = ref renderingData.cameraData;      // 获取摄像机属性
        var camera = cameraData.camera;                         // 传入摄像机
        var source = renderTargetIdentifier;                    // 获取渲染图片
        int destination = TempTargetId;                         // 渲染结果图片

        // 定义屏幕尺寸
        var width = (int)(camera .scaledPixelWidth / gaussianBlur.RTDownSampling.value);
        var height = (int)(camera .scaledPixelHeight / gaussianBlur.RTDownSampling.value);


        Gaussianmaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);             // Shader变量  和 Volume 组件属性 绑定

        cmd.SetGlobalTexture(MainTexId, source);                                           // 渲染原图 储存到 source
        cmd.GetTemporaryRT(
            destination,
            width,
            height,
            0,
            FilterMode.Trilinear,
            RenderTextureFormat.Default
            );

        // 计算模糊
        for (int i = 0; i < gaussianBlur.BlurTimes.value; i++)
        {
            cmd.Blit(source, destination);                         // 设置后处理
            cmd.Blit(destination, source, Gaussianmaterial, 0);    //  第二个Pass
        }

    }
}
```

![[8ee97bf9bbc88681364c992db823a9c4_MD5.png]]

*   Box 模糊和 上面的 高斯模糊差不多，
    

![[0421e10da0d674b490bfc83aeb700856_MD5.webp]]

*   主要是他们的计算 高斯核不一样。
    

**注意：方框模型和高斯模糊都使用同样的渲染管线就可以，只需要修改 Shader 就可以，他们指定不同的 Shader 就可以。**

![[7c34f0c863c3b429a1f9b081455192f0_MD5.webp]]

Shader

```
Shader "URP/2_BoxBlur"
{
    Properties
    {
        _MainTex ("_MainTex", 2D) = "white" {}
        _BlurRange("_BlurRange",Float) = 0
        _MainTex_TexelSize("_MainTex_TexelSize", Vector) = (0,0,0,0)
    }
    SubShader
    {
        Tags { "RenderPipeline"="UniversalPipeline" }
         Cull Off ZWrite Off ZTest Always


        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment Boxfrag

            #include "Assets/Post/02_Blur/GaussianBlur/Blur.hlsl"           //函数库

            ENDHLSL
        }
    }
}
```

*   Shader 阶段需要注意的是，定义新的片元着色器，指定我们调用的函数库的位置。
    

![[25680dc01a0e26fe20c0ca6dd8682f1a_MD5.webp]]

调用 Box Frag 片元着色器

hlsl

```
//  方框模糊 的片元着色器阶段
half4 Boxfrag(v2f i) : SV_Target
{

    float4 col = float4(0, 0, 0, 0);
    float2 UV_Offset;

    float Box_Weight = 0.11111;

    for (int x = -1; x <= 1; x++)
    {
        for (int y = -1; y <= 1; y++)
        {
            UV_Offset = i.uv;
            UV_Offset.x += x * _MainTex_TexelSize.x * _BlurRange / 3;
            UV_Offset.y += y * _MainTex_TexelSize.y * _BlurRange / 3;
            col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, UV_Offset);
        }
    }
    col *= 0.11111;
    return col;
}
```

主要是定义一个新的 片元着色器，命名— Box Frag.

**扩展 俩种模糊方式的对比效果。**

*   方框模糊
    
    Blur Times 6      Blur Range  2     RT Down Sampling    3
    

![[0c6ab3ccee4689538b86156c4e5874db_MD5.webp]]

*   高斯模糊
    

![[02899199fc09f3c59b5fd49dc10a3610_MD5.webp]]

*   第二步 实现同样类似的模糊，看俩种模糊的算法对比。
    
    这是俩种差不多的模糊，
    

![[42c780060cd200124866b53fecc4cc00_MD5.webp]]

![[838abb4f1ef98eb982516268c9c0479a_MD5.webp]]

**总结：方框模糊比高斯模糊迭代次数少， 优点就是比高斯模糊省。**

![[8ee97bf9bbc88681364c992db823a9c4_MD5.png]]

Kawase Blur 于 Masaki Kawase 在 GDC2003 的分享《Frame Buffer Postprocessing Effects in DOUBLE-S.T.E.A.L (Wreckless)》中提出。

Kawase Blur 最初用于 Bloom 后处理特效，但其可以推广作为专门的模糊算法使用，且在模糊外观表现上与高斯模糊非常接近。

Kawase Blur 的思路是对距离当前像素越来越远的地方对四个角进行采样，且在两个大小相等的纹理之间进行乒乓式的 blit。

创新点在于，采用了随迭代次数移动的 blur kernel，而不是类似高斯模糊，或 box blur 一样从头到尾固定的 blur kernel。

![[a94c7d46b64e316cc53038004889a5db_MD5.webp]]

**优点：Kawase Blur 比经过优化的高斯模糊的性能约快 1.5 倍到 3 倍。**

*   我们前在 blur.hlsl 计算片元着色器
    

```
half4 Kawasefrag(v2f i) : SV_Target
{
float4 col = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex,i.uv);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(-1, -1) * _MainTex_TexelSize.xy * _BlurRange);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(1, -1) * _MainTex_TexelSize.xy * _BlurRange);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(-1, 1) * _MainTex_TexelSize.xy * _BlurRange);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv + float2(1, 1) * _MainTex_TexelSize.xy * _BlurRange);
    col /= 5;
    return col;
}
```

对像素斜四角像素进行采样，然后取平均值。

**注意：第一个是主像素的位置，剩下 4 个是 4 个点的像素。**

Shader

```
Shader "URP/3_KawaseBulr"
{
    Properties
    {
        _MainTex ("_MainTex", 2D) = "white" {}
        _BlurRange("_BlurRange",Float) = 0
    }
    SubShader
    {
        Tags { "RenderPipeline"="UniversalPipeline" }
         Cull Off ZWrite Off ZTest Always


        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment Kawasefrag

            #include "Assets/Post/02_Blur/GaussianBlur/Blur.hlsl"     // 函数

            ENDHLSL
        }
    }
}
```

*   我们前创建渲染流程  
    

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class KawaseBulrRenderFeature : ScriptableRendererFeature
{

    [System.Serializable]
    public class Settings
    {
        public RenderPassEvent renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
        public Shader shader;
    }
    public Settings settings = new Settings();

    KawaseBulrPass kawaseBulrpass;           // 定义我们创建出Pass
    public override void Create()
    {
        this.name = "KawaseBulrPass";    // 模糊渲染的名字
        kawaseBulrpass = new KawaseBulrPass(RenderPassEvent.BeforeRenderingPostProcessing, settings.shader);

    }
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        kawaseBulrpass.Setup(renderer.cameraColorTarget);   // 初始化
        renderer.EnqueuePass(kawaseBulrpass);
    }
}

public class KawaseBulrPass : ScriptableRenderPass
{
    static readonly string RenderTag = " Blur Effects";                             // 设置渲染标签
    static readonly int TempTargetId = Shader.PropertyToID("_TempTargetGaussian");                     // 设置储存图像信息
    static readonly int MainTexId = Shader.PropertyToID("_MainTex");                // 源贴图

    GaussianBlurVolume gaussianBlur;                                                // 定义组件类型
    Material KawaseMaterial;                                                        // 后处理材质
    RenderTargetIdentifier renderTargetIdentifier;                                  // 设置当前渲染目标

    public KawaseBulrPass(RenderPassEvent evt, Shader blurshader)
    {
        renderPassEvent = evt;
        var shader = blurshader;

        if (shader = null)
        {
            Debug.LogError("没有指定Shader");
            return;
        }
        KawaseMaterial = CoreUtils.CreateEngineMaterial(blurshader);
    }

    public void Setup(in RenderTargetIdentifier sur)
    {
        this.renderTargetIdentifier = sur;
    }

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        if (KawaseMaterial == null)
        {
            Debug.LogError("材质初始化失败");
            return;
        }

        if (!renderingData.cameraData.postProcessEnabled)
        {
            Debug.LogError("摄像机是否开启后处理");
            return;
        }

        var stack = VolumeManager.instance.stack;                                    // 传入 volume
        gaussianBlur = stack.GetComponent<GaussianBlurVolume>();                     // 获取到后处理组件

        if (gaussianBlur == null)
        {
            Debug.LogError("获取组件失败");
            return;
        }

        var cmd = CommandBufferPool.Get(RenderTag);          // 渲染标签

        Render(cmd, ref renderingData);                 // 调用渲染函数

        context.ExecuteCommandBuffer(cmd);              // 执行函数，回收。
        CommandBufferPool.Release(cmd);
    }

    void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        ref var cameraData = ref renderingData.cameraData;      // 获取摄像机属性
        var camera = cameraData.camera;                         // 传入摄像机

        var source = renderTargetIdentifier;                    // 获取渲染原图像
        int destination = TempTargetId;                         // 渲染结果图片

        cmd.SetGlobalTexture(MainTexId, source);                                           // 渲染原图像 储存到 source
        // 定义屏幕尺寸
        var width = (int)(camera.scaledPixelWidth / gaussianBlur.RTDownSampling.value);
        var height = (int)(camera.scaledPixelHeight / gaussianBlur.RTDownSampling.value);

        cmd.GetTemporaryRT(
            destination,
            width,
            height,
            0,
            FilterMode.Trilinear,
            RenderTextureFormat.Default
            );
        KawaseMaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);             // Shader变量  和 Volume 组件属性 绑定
        cmd.Blit(source, destination);

        cmd.Blit(destination, source, KawaseMaterial, 0);
    }
}
```

这个阶段是把 destination 的效果输出到屏幕，降采样和 模糊起作用了，我接下来处理核心代码。

*   创建 2 个 RT 我们需要来回切换。  
    

```
void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        ref var cameraData = ref renderingData.cameraData;      // 获取摄像机属性
        var camera = cameraData.camera;                         // 传入摄像机

        var source = renderTargetIdentifier;                    // 获取渲染原图像

        int bufferid1 = Shader.PropertyToID("bufferblur1");         // 临时图像，也可以用 Handle，ID不乱就可以
        int bufferid2 = Shader.PropertyToID("bufferblur2");         // 临时图像，也可以用 Handle，ID不乱就可以

        cmd.SetGlobalTexture(MainTexId, source);                                           // 渲染原图像 储存到 source
        // 定义屏幕尺寸
        var width = (int)(camera.scaledPixelWidth / gaussianBlur.RTDownSampling.value);
        var height = (int)(camera.scaledPixelHeight / gaussianBlur.RTDownSampling.value);

        cmd.GetTemporaryRT(bufferid1, width, height, 0, FilterMode.Bilinear, RenderTextureFormat.DefaultHDR); //申请一个临时图像，并设置相机rt的参数进去
        cmd.GetTemporaryRT(bufferid2, width, height, 0, FilterMode.Bilinear, RenderTextureFormat.DefaultHDR); //申请一个临时图像，并设置相机rt的参数进去

        RenderTargetIdentifier buffer1 = bufferid1;
        RenderTargetIdentifier buffer2 = bufferid2;


        KawaseMaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);             // Shader变量  和 Volume 组件属性 绑定
        cmd.Blit(source, buffer1, KawaseMaterial);

        for (int i = 1; i < gaussianBlur.BlurTimes.value; i++)
        {
            KawaseMaterial.SetFloat("_BlurRange", i * gaussianBlur.BlurRange.value + 1);             // Shader变量  和 Volume 组件属性 绑定
            cmd.Blit(bufferid1, bufferid2, KawaseMaterial);
            var temRT = buffer1;
            buffer1 = buffer2;
            buffer2 = temRT;
        }
        KawaseMaterial.SetFloat("_BlurRange", gaussianBlur.BlurTimes.value * gaussianBlur.BlurRange.value + 1);             // Shader变量  和 Volume 组件属性 绑定
        cmd.Blit(buffer1, source, KawaseMaterial);
    }
```

这个阶段主要是注意，2 个 RT 来回切换。

*   在模糊迭代开始，材质我们需要使用迭代次数来控制模糊次数
    

![[63691323eabed6dc55f30bf64dfcd5e3_MD5.webp]]

*   来回 Blit 处理
    

![[93abd45a7ec88c5c3967100863ba9234_MD5.webp]]

*   当前迭代次数，对每次模糊的半径进行设置
    

![[62af3a7afbae90d930c5ba46daf6c325_MD5.webp]]

*   效果
    

![[7cc06e5c61649f165dd70bcc6e4bf11f_MD5.webp]]

*   属性数值
    

![[90a531c36057be6889138bca1b5f9c33_MD5.webp]]

这样迭代 2 次就有一个不错的模糊效果。

**KawaseBulrRenderFeature**

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class KawaseBulrRenderFeature : ScriptableRendererFeature
{

    [System.Serializable]
    public class Settings
    {
        public RenderPassEvent renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
        public Shader shader;
    }
    public Settings settings = new Settings();

    KawaseBulrPass kawaseBulrpass;           // 定义我们创建出Pass
    public override void Create()
    {
        this.name = "KawaseBulrPass";    // 模糊渲染的名字
        kawaseBulrpass = new KawaseBulrPass(RenderPassEvent.BeforeRenderingPostProcessing, settings.shader);

    }
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        kawaseBulrpass.Setup(renderer.cameraColorTarget);   // 初始化
        renderer.EnqueuePass(kawaseBulrpass);
    }
}

public class KawaseBulrPass : ScriptableRenderPass
{
    static readonly string RenderTag = " Blur Effects";                             // 设置渲染标签

    static readonly int MainTexId = Shader.PropertyToID("_MainTex");                // 源贴图

    GaussianBlurVolume gaussianBlur;                                                // 定义组件类型
    Material KawaseMaterial;                                                        // 后处理材质
    RenderTargetIdentifier renderTargetIdentifier;                                  // 设置当前渲染目标

    public KawaseBulrPass(RenderPassEvent evt, Shader blurshader)
    {
        renderPassEvent = evt;
        var shader = blurshader;

        if (shader = null)
        {
            Debug.LogError("没有指定Shader");
            return;
        }
        KawaseMaterial = CoreUtils.CreateEngineMaterial(blurshader);
    }

    public void Setup(in RenderTargetIdentifier sur)
    {
        this.renderTargetIdentifier = sur;
    }

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        if (KawaseMaterial == null)
        {
            Debug.LogError("材质初始化失败");
            return;
        }

        if (!renderingData.cameraData.postProcessEnabled)
        {
            Debug.LogError("摄像机是否开启后处理");
            return;
        }

        var stack = VolumeManager.instance.stack;                                    // 传入 volume
        gaussianBlur = stack.GetComponent<GaussianBlurVolume>();                     // 获取到后处理组件

        if (gaussianBlur == null)
        {
            Debug.LogError("获取组件失败");
            return;
        }

        var cmd = CommandBufferPool.Get(RenderTag);          // 渲染标签

        Render(cmd, ref renderingData);                 // 调用渲染函数

        context.ExecuteCommandBuffer(cmd);              // 执行函数，回收。
        CommandBufferPool.Release(cmd);
    }

    void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        ref var cameraData = ref renderingData.cameraData;      // 获取摄像机属性
        var camera = cameraData.camera;                         // 传入摄像机

        var source = renderTargetIdentifier;                    // 获取渲染原图像

        int bufferid1 = Shader.PropertyToID("bufferblur1");         // 临时图像，也可以用 Handle，ID不乱就可以
        int bufferid2 = Shader.PropertyToID("bufferblur2");         // 临时图像，也可以用 Handle，ID不乱就可以

        cmd.SetGlobalTexture(MainTexId, source);                                           // 渲染原图像 储存到 source
        // 定义屏幕尺寸
        var width = (int)(camera.scaledPixelWidth / gaussianBlur.RTDownSampling.value);
        var height = (int)(camera.scaledPixelHeight / gaussianBlur.RTDownSampling.value);

        cmd.GetTemporaryRT(bufferid1, width, height, 0, FilterMode.Bilinear, RenderTextureFormat.DefaultHDR); //申请一个临时图像，并设置相机rt的参数进去
        cmd.GetTemporaryRT(bufferid2, width, height, 0, FilterMode.Bilinear, RenderTextureFormat.DefaultHDR); //申请一个临时图像，并设置相机rt的参数进去

        RenderTargetIdentifier buffer1 = bufferid1;
        RenderTargetIdentifier buffer2 = bufferid2;


        KawaseMaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);             // Shader变量  和 Volume 组件属性 绑定
        cmd.Blit(source, buffer1, KawaseMaterial);

        for (int i = 1; i < gaussianBlur.BlurTimes.value; i++)
        {
            KawaseMaterial.SetFloat("_BlurRange", i * gaussianBlur.BlurRange.value + 1);             // Shader变量  和 Volume 组件属性 绑定
            cmd.Blit(bufferid1, bufferid2, KawaseMaterial);
            var temRT = buffer1;
            buffer1 = buffer2;
            buffer2 = temRT;
        }
        KawaseMaterial.SetFloat("_BlurRange", gaussianBlur.BlurTimes.value * gaussianBlur.BlurRange.value + 1);             // Shader变量  和 Volume 组件属性 绑定
        cmd.Blit(buffer1, source, KawaseMaterial);
    }
}
```

![[8ee97bf9bbc88681364c992db823a9c4_MD5.png]]

上面这些模糊算法我们可以看出，一旦模糊距离增大，必须得加大迭代次数来消除方格感。

Dual Blur 是利用降采样和升采样，解决方格感，还有 Dual Blur 方式在顶点着色器阶段对 UV 进偏移。这样可以减少计算量。

*   一个降采样，一个升采样。
    

![[54e291b68d6b85a7ccab3e464e8b5c29_MD5.webp]]

![[d2699eded17cb4d91cfcb9f54246a3e2_MD5.webp]]

Shader 部分，我们是使用俩个 Pass 完成，

为了带来更好的性能表现，可以将 uv 的偏移放在 Vert Shader 中进行，而 Fragment Shader 中基本上仅进行采样即可。

**DualKawaseBlur.hlsl**

*   需要重新定义顶点结构体，
    

```
struct appdata
{
    float4 positionOS : POSITION;
    float2 texcoord : TEXCOORD0;
};

struct v2f_DualBlurDown
{
    float2 uv[5] : TEXCOORD0;             // uv 处理
    float4 vertex : POSITION;             // 顶点
};

struct v2f_DualBlurUp
{
    float2 uv[8] : TEXCOORD0;
    float4 vertex : SV_POSITION;
};
```

我们准备俩个输出结构体，一个降采样，一个升采样。

*   降采样      第一个 UV 一个是定义 5 的数组
    
*   升采样     第二个 UV 需要定义 8 个的 数组。
    

![[8b6ad1b10323c4abe8c61e261f4bbe5e_MD5.webp]]

**降采样**

*   顶点着色器是对 UV 进行偏移。
    

```
//  高斯模糊 的片元着色器阶段

v2f_DualBlurDown DualKawaseDownvert(appdata v)
{
    //降采样
    v2f_DualBlurDown o;
    o.vertex = TransformObjectToHClip(v.positionOS.xyz);
    o.uv[0] = v.texcoord;


	//
    o.uv[1] = v.texcoord + float2(-1, -1) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5; //↖
    o.uv[2] = v.texcoord + float2(-1, 1) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5; //↙
    o.uv[3] = v.texcoord + float2(1, -1) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5; //↗
    o.uv[4] = v.texcoord + float2(1, 1) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5; //↘
    //
    return o;
}

float4 DualKawaseDownfrag(v2f_DualBlurDown i) : SV_TARGET
{
    //降采样
    float4 col = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[0]) * 4;

    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[1]);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[2]);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[3]);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[4]);

    return col * 0.125; //sum / 8.0f
}
```

**升采样**

*   顶点着色器
    

```
v2f_DualBlurUp DualKawaseUpvert(appdata v)
{
    //升采样
    v2f_DualBlurUp o;
    o.vertex = TransformObjectToHClip(v.positionOS.xyz);
    o.uv[0] = v.texcoord;

	//
    o.uv[0] = v.texcoord + float2(-1, -1) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5;
    o.uv[1] = v.texcoord + float2(-1, 1) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5;
    o.uv[2] = v.texcoord + float2(1, -1) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5;
    o.uv[3] = v.texcoord + float2(1, 1) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5;
    o.uv[4] = v.texcoord + float2(-2, 0) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5;
    o.uv[5] = v.texcoord + float2(0, -2) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5;
    o.uv[6] = v.texcoord + float2(2, 0) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5;
    o.uv[7] = v.texcoord + float2(0, 2) * (1 + _BlurRange) * _MainTex_TexelSize.xy * 0.5;
    //
    return o;
}

float4 DualKawaseUpfrag(v2f_DualBlurUp i) : SV_TARGET
{
    //升采样
    float4 col = 0;

    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[0]) * 2;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[1]) * 2;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[2]) * 2;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[3]) * 2;
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[4]);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[5]);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[6]);
    col += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[7]);

    return col * 0.0833; //sum / 12.0f
}
```

Shader 部分

```
Shader "URP/4_DualkawaseBlur"
{
    Properties
    {
        _MainTex ("_MainTex", 2D) = "white" {}
        _BlurRange("_BlurRange",Float) = 0
    }
    SubShader
    {
        Tags { "RenderPipeline"="UniversalPipeline" }
         Cull Off ZWrite Off ZTest Always


        Pass
        {
            Name "DownSample"
            HLSLPROGRAM
            #pragma vertex DualKawaseDownvert
            #pragma fragment DualKawaseDownfrag

            #include "Assets/Post/02_Blur/DualKawase/DualKawaseBlur.hlsl"           //函数库

            ENDHLSL
        }

        Pass
        {
            Name "UpSample"
            HLSLPROGRAM
            #pragma vertex DualKawaseUpvert
            #pragma fragment DualKawaseUpfrag

            #include "Assets/Post/02_Blur/DualKawase/DualKawaseBlur.hlsl"           //函数库

            ENDHLSL
        }
    }
}
```

定义 2 个 Pass

因为我们都是使用的一个 Volume，所以我们调用同一个 组件，

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class DualKawaseRenderFeature : ScriptableRendererFeature
{
    [System.Serializable]
    public class Settings
    {
        [Tooltip("指定pass渲染时机")]
        public RenderPassEvent renderPassEvent = RenderPassEvent.AfterRenderingTransparents;
        [Tooltip("指定Shader")]
        public Shader shader;
    }
    public Settings settings = new Settings();

    DualKawaseBlurPass dualKawaseBlur;           // 定义我们创建出Pass

    public override void Create()
    {
        this.name = "DualKawaseBlurPass";    // 模糊渲染的名字
        dualKawaseBlur = new DualKawaseBlurPass(RenderPassEvent.AfterRenderingTransparents, settings.shader);    // 初始化 我们的渲染层级和Shader

    }
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        renderer.EnqueuePass(dualKawaseBlur);
    }
}
```

这里只是 设置好渲染内容，

我们要定义 Pass 设置渲染逻辑

```
public class DualKawaseBlurPass : ScriptableRenderPass
{
    static readonly string RenderTag = " DualKawase Effects";                         // 设置渲染标签

    GaussianBlurVolume gaussianBlur;                                                // 定义组件类型

    Material Gaussianmaterial;                                                      // 后处理材质

    int[] downSampleRT;                // 定义2个数组类型的变量
    int[] upSampleRT;


    public DualKawaseBlurPass(RenderPassEvent evt, Shader blurshader)
    {
        renderPassEvent = evt;
        var shader = blurshader;

        if (shader == null)
        {
            Debug.LogError("没有指定Shader");
            return;
        }
        Gaussianmaterial = CoreUtils.CreateEngineMaterial(blurshader);
    }


    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        if (Gaussianmaterial == null)
        {
            Debug.LogError("材质初始化失败");
            return;
        }

        if (!renderingData.cameraData.postProcessEnabled)
        {
            Debug.LogError("摄像机是否开启后处理");
            return;
        }

        var stack = VolumeManager.instance.stack;                                    // 传入 volume
        gaussianBlur = stack.GetComponent<GaussianBlurVolume>();                     // 获取到后处理组件

        if (gaussianBlur == null)
        {
            Debug.LogError("获取组件失败");
            return;
        }

        var cmd = CommandBufferPool.Get(RenderTag);    // 渲染标签

        Render(cmd, ref renderingData);                 // 调用渲染函数

        context.ExecuteCommandBuffer(cmd);              // 执行函数，回收。
        CommandBufferPool.Release(cmd);

    }
}
```

这个部分是我们的算法核心，

```
void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        RenderTargetIdentifier sourceRT = renderingData.cameraData.renderer.cameraColorTarget;                 // 定义RT
        RenderTextureDescriptor inRTDesc = renderingData.cameraData.cameraTargetDescriptor;
        inRTDesc.depthBufferBits = 0;                                                                          // 清除深度

        Gaussianmaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);                                 // 绑定组件的属性

        // 定义屏幕尺寸
        var width = (int)(inRTDesc.width / gaussianBlur.RTDownSampling.value);
        var height = (int)(inRTDesc.height / gaussianBlur.RTDownSampling.value);


        downSampleRT = new int[gaussianBlur.BlurTimes.value];                     // 绑定属性控制采样
        upSampleRT = new int[gaussianBlur.BlurTimes.value];

        for (int i = 0; i < gaussianBlur.BlurTimes.value; i++)
        {
            downSampleRT[i] = Shader.PropertyToID("DownSample" + i);
            upSampleRT[i] = Shader.PropertyToID("UpSample" + i);
        }
        RenderTargetIdentifier tmpRT = sourceRT;
        //downSample
        for (int i = 0; i < gaussianBlur.BlurTimes.value; i++)
        {
            cmd.GetTemporaryRT(downSampleRT[i], width, height, 0, FilterMode.Bilinear, RenderTextureFormat.Default);
            cmd.GetTemporaryRT(upSampleRT[i], width, height, 0, FilterMode.Bilinear, RenderTextureFormat.Default);
            width = Mathf.Max(width / 2, 1);
            height = Mathf.Max(height / 2, 1);
            cmd.Blit(tmpRT, downSampleRT[i], Gaussianmaterial, 0);                                // 调用第一个 pass 降采样
            tmpRT = downSampleRT[i];
        }

        //upSample
        for (int j = gaussianBlur.BlurTimes.value - 2; j >= 0; j--)                               // 注意，这里是j 输入的是的降采样
        {
            cmd.Blit(tmpRT, upSampleRT[j], Gaussianmaterial, 1);                                  // 调用第二个 pass 降采样
            tmpRT = upSampleRT[j];
        }
        //final pass
        cmd.Blit(tmpRT, sourceRT);
        //Release All tmpRT
        for (int i = 0; i < gaussianBlur.BlurTimes.value; i++)
        {
            cmd.ReleaseTemporaryRT(downSampleRT[i]);
            cmd.ReleaseTemporaryRT(upSampleRT[i]);
        }

    }
```

*   渲染效果
    

![[3e605568242edc3b2719b2109fae9ceb_MD5.webp]]

![[63a1c9e311be358e759498ae4708a853_MD5.webp]]

**全代码**

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class DualKawaseRenderFeature : ScriptableRendererFeature
{
    [System.Serializable]
    public class Settings
    {
        [Tooltip("指定pass渲染时机")]
        public RenderPassEvent renderPassEvent = RenderPassEvent.AfterRenderingTransparents;
        [Tooltip("指定Shader")]
        public Shader shader;
    }
    public Settings settings = new Settings();

    DualKawaseBlurPass dualKawaseBlur;           // 定义我们创建出Pass

    public override void Create()
    {
        this.name = "DualKawaseBlurPass";    // 模糊渲染的名字
        dualKawaseBlur = new DualKawaseBlurPass(RenderPassEvent.AfterRenderingTransparents, settings.shader);    // 初始化 我们的渲染层级和Shader

    }
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        renderer.EnqueuePass(dualKawaseBlur);
    }
}

public class DualKawaseBlurPass : ScriptableRenderPass
{
    static readonly string RenderTag = " DualKawase Effects";                         // 设置渲染标签

    GaussianBlurVolume gaussianBlur;                                                // 定义组件类型

    Material Gaussianmaterial;                                                      // 后处理材质

    int[] downSampleRT;                // 定义2个数组类型的变量
    int[] upSampleRT;


    public DualKawaseBlurPass(RenderPassEvent evt, Shader blurshader)
    {
        renderPassEvent = evt;
        var shader = blurshader;

        if (shader == null)
        {
            Debug.LogError("没有指定Shader");
            return;
        }
        Gaussianmaterial = CoreUtils.CreateEngineMaterial(blurshader);
    }


    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        if (Gaussianmaterial == null)
        {
            Debug.LogError("材质初始化失败");
            return;
        }

        if (!renderingData.cameraData.postProcessEnabled)
        {
            Debug.LogError("摄像机是否开启后处理");
            return;
        }

        var stack = VolumeManager.instance.stack;                                    // 传入 volume
        gaussianBlur = stack.GetComponent<GaussianBlurVolume>();                     // 获取到后处理组件

        if (gaussianBlur == null)
        {
            Debug.LogError("获取组件失败");
            return;
        }

        var cmd = CommandBufferPool.Get(RenderTag);    // 渲染标签

        Render(cmd, ref renderingData);                 // 调用渲染函数

        context.ExecuteCommandBuffer(cmd);              // 执行函数，回收。
        CommandBufferPool.Release(cmd);

    }

    void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        RenderTargetIdentifier sourceRT = renderingData.cameraData.renderer.cameraColorTarget;                 // 定义RT
        RenderTextureDescriptor inRTDesc = renderingData.cameraData.cameraTargetDescriptor;
        inRTDesc.depthBufferBits = 0;                                                                          // 清除深度

        Gaussianmaterial.SetFloat("_BlurRange", gaussianBlur.BlurRange.value);                                 // 绑定组件的属性

        int tw = (int)inRTDesc.width;
        int th = (int)inRTDesc.height;


        // 定义屏幕尺寸
        var width = (int)(inRTDesc.width / gaussianBlur.RTDownSampling.value);
        var height = (int)(inRTDesc.height / gaussianBlur.RTDownSampling.value);


        downSampleRT = new int[gaussianBlur.BlurTimes.value];                     // 绑定属性控制采样
        upSampleRT = new int[gaussianBlur.BlurTimes.value];

        for (int i = 0; i < gaussianBlur.BlurTimes.value; i++)
        {
            downSampleRT[i] = Shader.PropertyToID("DownSample" + i);
            upSampleRT[i] = Shader.PropertyToID("UpSample" + i);
        }
        RenderTargetIdentifier tmpRT = sourceRT;
        //downSample
        for (int i = 0; i < gaussianBlur.BlurTimes.value; i++)
        {
            cmd.GetTemporaryRT(downSampleRT[i], width, height, 0, FilterMode.Bilinear, RenderTextureFormat.Default);
            cmd.GetTemporaryRT(upSampleRT[i], width, height, 0, FilterMode.Bilinear, RenderTextureFormat.Default);
            width = Mathf.Max(width / 2, 1);
            height = Mathf.Max(height / 2, 1);
            cmd.Blit(tmpRT, downSampleRT[i], Gaussianmaterial, 0);                                // 调用第一个 pass 降采样
            tmpRT = downSampleRT[i];
        }

        //upSample
        for (int j = gaussianBlur.BlurTimes.value - 2; j >= 0; j--)                               // 注意，这里是j 输入的是的降采样
        {
            cmd.Blit(tmpRT, upSampleRT[j], Gaussianmaterial, 1);                                  // 调用第二个 pass 降采样
            tmpRT = upSampleRT[j];
        }
        //final pass
        cmd.Blit(tmpRT, sourceRT);
        //Release All tmpRT
        for (int i = 0; i < gaussianBlur.BlurTimes.value; i++)
        {
            cmd.ReleaseTemporaryRT(downSampleRT[i]);
            cmd.ReleaseTemporaryRT(upSampleRT[i]);
        }

    }
}
```

扩展 RT 是什么？

URP 源码学习（七）一些细节和理解 - 知乎 (zhihu.com)

首先 rt 是一张特殊贴图，这张贴图对应的是 GPU 上的 **FrameBuffer.**

**FrameBuffer 缓存区储存 2 种缓存**

*   颜色缓存，
    
*   深度缓存
    

![[8ee97bf9bbc88681364c992db823a9c4_MD5.png]]

到这里 4 种模糊算法算完事，

*   我们看到渲染结果，
    

![[19c6d33015cc7aba5fee754026ab509b_MD5.webp]]

*   这个是原图要输出的显示效果，
    

*   这个是没有增加后处理的原图备份。
    

![[8bbb9437fdc543b288caa9698854d58b_MD5.webp]]

*   这些是计算模糊的过程阶段图。
    

![[63a240c0335f9988b2f26fabf71f0e02_MD5.webp]]

*   最后输出到屏幕处理结果图。
    

![[19ceb989da6739ec919303f6460bb9c8_MD5.webp]]

*   高速模糊最简单，Box 模型也只是在 Shader 阶段进行处理，最复杂的是 Dual 和 Kawase 模糊方式， 相对 Dual 性能更好。
    
*   从细节梳理了后处理渲染流程，创建成模板，好维护处理。
    
*   对 RT 的类整理学习，经常使用的类型
    

*   **Blit 函数**
    
    一个纹理复制到另一个纹理。
    

![[0484c35a83947d642ec049a7140b47fc_MD5.webp]]

*   **GetTemporaryRT**
    
    设置 RT 的属性
    

![[bb14b2b3bf038c3a965a1cbf8ff32d23_MD5.webp]]

*   **RenderTargetIdentifier**
    
    临时纹理的一些属性
    

![[ffee0769e3a1180ae0f2ff3d29e952d5_MD5.webp]]

*   **RenderTextureDescriptor**
    
    RT 的一个类包含 RT 的所有属性
    

![[51752aa35d915f989b2f0d83dc9b1d58_MD5.webp]]

URP Gaussian/Box/Kawase/Dual Blur 实现 - 知乎 (zhihu.com)

Unity URP DualKawaseBlur TA (tajourney.games)

高品质后处理：十种图像模糊算法的总结与实现 - 知乎 (zhihu.com)

【Free Bird/URP 教学】7.URP 后处理 -- 高斯模糊 - 知乎 (zhihu.com)

[【Unity ShaderGraph UI 模糊特效教程】_哔哩哔哩_bilibili](https://www.bilibili.com/video/av803176932/?vd_source=3822296874da2e675b185092cbd665e9)