### **RenderFeature**

首先来了解一下 **renderfeature**。在 Urp 中，renderfeature 是把我们自定义的 pass 插入到渲染队列里的一个工具。通过编写 renderfeature，可以设定该 pass 的渲染时机。本文的目的是在画面渲染完毕以后进行高斯模糊后处理。

urp 自带了 renderfeature 模板，只要填充该模板即可。

**右键，Create->rendering->URP renderfeature**，创建一个名为 **GaussianBlurRenderFeature** 的模板，打开后里面就是 urp 提供的模板：

![[0790072802812eb8fa387847abd38320_MD5.png]]

我们先来大概了解一下 renderfeature 是怎么起作用的：

可以看到 **GaussianBlurRenderFeature** 继承自 **ScriptableRendererFeature**，它提供了一个需要重写的函数 **Create**，用来创建你自定义的渲染 Pass 类。然后通过 **AddRenderPasses** 函数将创建的 Pass 压入渲染队列。

在 **GaussianBlurRenderFeature** 里还有一个自动生成的类 **CustomRenderPass**，继承自 **ScriptableRenderPass**，也就是用来渲染自定义 Pass 的。压入渲染队列后，Unity 会去调用这三个 override 的函数。我们编写的后处理暂时用不上两个 On 函数，直接删了就行。

**Create** 函数里还有一个 **renderPassEvent**，这个变量决定了编写的 Pass 会在什么时候执行。后处理的设置一般为 **RenderPassEvent.BeforeRenderingPostProcessing**。

### **准备操作**

接下来正式开始编写后处理，不过别急，先做一下准备工作。

首先我们创建一个继承自 **VolumeComponent** 的类，这个类可以让你编写的参数暴露在 Volume 组件上，直接用 Volume 组件就可以更改参数。(写在一个类里面就行了)

```
class GaussianBlurParameter : VolumeComponent
{
    public ClampedIntParameter iterations = new ClampedIntParameter(3, 0, 4);
    public ClampedFloatParameter blurSpread = new ClampedFloatParameter(0.6f, 0.2f, 3.0f);
    public ClampedIntParameter downSample = new ClampedIntParameter(2, 1, 8);
}
```

然后在 **CustomRenderPass** 中添加以下成员，具体干什么看注释：

```
private static readonly string shader_name = "URP/PostProcessing/GaussianBlur";     
static readonly string k_RenderTag = "Render GaussianBlur";                             // 显示在frame debug里的名称
static readonly int TempTargetId0 = Shader.PropertyToID("_TempTargetColorTint0");       // 用来暂存纹理
static readonly int TempTargetId1 = Shader.PropertyToID("_TempTargetColorTint1");       // 用来暂存纹理
​
GaussianBlurParameter parameter;            // 参数类
Material postMaterial;                      // 后处理材质
RenderTargetIdentifier currentTarget;       // 用来获取相机rt的id
​
```

接着写个构造函数来初始化材质，顺带把渲染时机给他设定了：

```
public CustomRenderPass(RenderPassEvent evt)
{
    this.renderPassEvent = evt;
    Init();
}
​
void Init()
{
    var shader = Shader.Find(shader_name);
    postMaterial = CoreUtils.CreateEngineMaterial(shader);
}
```

我们还需要指定处理的 rt 是从哪来的，在这里就是相机：

```
public void SetTarget(ScriptableRenderer renderer) {
    currentTarget = renderer.cameraColorTarget;
}
```

前文说过，urp 会调用类的 **Execute** 函数，这个函数才是真正去渲染的函数。该函数会传入一个 **Context** 和一个 **RenderingData**。在渲染前，我们先检查一下资产有没有准备完毕：

```
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
{
    if (postMaterial == null) return;
​
    if (!renderingData.cameraData.postProcessEnabled)   // 相机有没有打开后处理
    {
        return;
    }
​
    var stack = VolumeManager.instance.stack;
    parameter = stack.GetComponent<GaussianBlurParameter>();
    if (parameter == null) return;
}
```

然后从池里面取出 **cmd**，参数就用之前设置的名字。渲染操作加入 cmd 后，让上下文执行 cmd 内的操作，最后把 cmd 给放了。这个 **Render** 是用来做后处理的函数，把操作都封装进去了，等下说。

```
var cmd = CommandBufferPool.Get(k_RenderTag);
Render(cmd, ref renderingData);
context.ExecuteCommandBuffer(cmd);
CommandBufferPool.Release(cmd);
```

这样类内部的准备部分就完毕了，最后把 renderfeature 的两个函数改了:

```
public override void Create()
{
    m_ScriptablePass = new CustomRenderPass(RenderPassEvent.BeforeRenderingPostProcessing);
}
​
public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
{
    m_ScriptablePass.SetTarget(renderer);
    renderer.EnqueuePass(m_ScriptablePass);
}
```

### **高斯模糊**

高斯模糊就不细说了，相关文章有很多。稍微讲一下原理：用高斯核对原图像进行卷积。由于高斯函数是能拆成 G(x)G(y) 形式的，所以核也能拆成一维横向和一维竖向的核，而且这两个核是完全一致的。因此横着的 Pass 来一次，竖着的 Pass 来一次，就完成高斯模糊了。

先写刚才遗留的 **Render** 函数。其实大部分和入门精要都差不多，稍微有点不同的就是用 **cmd** 来做 **Blit**、**GetTemporaryRT**。

```
void Render(CommandBuffer cmd, ref RenderingData renderingData)
{
    var source = currentTarget;
    int destination0 = TempTargetId0;
    int destination1 = TempTargetId1;
​
    ref var cameraData = ref renderingData.cameraData;
​
    var data = renderingData.cameraData.cameraTargetDescriptor;
​
    var width = data.width/ parameter.downSample.value;
    var height = data.height / parameter.downSample.value;
​
    // 先存到临时的地方
    cmd.GetTemporaryRT(destination0, width, height, 0, FilterMode.Trilinear, RenderTextureFormat.ARGB32);
    cmd.Blit(source, destination0);
​
    for (int i = 0; i < parameter.iterations.value; ++i)
    {
        cmd.SetGlobalFloat("_BlurSize", 1.0f + i * parameter.blurSpread.value);
​
        // 第一轮
        cmd.GetTemporaryRT(destination1, width, height, 0, FilterMode.Bilinear, RenderTextureFormat.ARGB32);
        cmd.Blit(destination0, destination1, postMaterial, 0);
        cmd.ReleaseTemporaryRT(destination0);
​
        // 第二轮
        cmd.GetTemporaryRT(destination0, width, height, 0, FilterMode.Bilinear, RenderTextureFormat.Default);
        cmd.Blit(destination1, destination0, postMaterial, 1);
        cmd.ReleaseTemporaryRT(destination1);
    }
​
    cmd.Blit(destination0, source);
​
    cmd.ReleaseTemporaryRT(TempTargetId0);
}
```

然后是 Shader。这里有一点**非常非常重要！！！非常非常重要！！！非常非常重要！！！**

那就是 Properties 里一定要写**_MainTex**，不能只在 HLSL 里声明！

URP 不知道抽的什么风，不在 Properties 写**_MainTex** 不会直接黑，只会导致高斯模糊的结果出问题！比如增加 iterations 会让画面出现残影；downSample 调成 1，提高 iterations 后 RT 变黑。说实话要不是突然想起来要在 Properties 里写 **_MainTex**，我估计这辈子都搞不定 unity 的后处理了...

```
Shader "URP/PostProcessing/GaussianBlur"
{
    Properties
    {
         _MainTex ("Texture", 2D) = "white" {}
    }
​
    SubShader
    {
        Tags
        {
           "RenderPipeline" = "UniversalRenderPipeline"
        }
        LOD 100
​
        HLSLINCLUDE
 #pragma prefer_hlslcc gles
 #pragma exclude_renderers d3d11_9x
 #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"
 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
 #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/UnityInstancing.hlsl"
​
​
        struct Attributes
        {
            float4 positionOS : POSITION;
            float2 uv : TEXCOORD0;
        };
​
        struct Varyings
        {
            float2 uv[5] : TEXCOORD0;
            float4 positionCS : SV_POSITION;
        };
​
        float _BlurSize;
        sampler2D _MainTex;
        float4 _MainTex_TexelSize;
​
        Varyings VertBlurVertical(Attributes v)
        {
            Varyings o;
            o.positionCS = TransformObjectToHClip(v.positionOS.xyz);
            float2 uv = v.uv;
​
            o.uv[0] = uv;
            o.uv[1] = uv + float2(0.0, _MainTex_TexelSize.y * 1.0) * _BlurSize;
            o.uv[2] = uv - float2(0.0, _MainTex_TexelSize.y * 1.0) * _BlurSize;
            o.uv[3] = uv + float2(0.0, _MainTex_TexelSize.y * 2.0) * _BlurSize;
            o.uv[4] = uv - float2(0.0, _MainTex_TexelSize.y * 2.0) * _BlurSize;
​
            return o;
        }
​
        Varyings VertBlurHorizontal(Attributes v)
        {
            Varyings o = (Varyings)0;
            o.positionCS = TransformObjectToHClip(v.positionOS.xyz);
            float2 uv = v.uv;
​
            o.uv[0] = uv;
            o.uv[1] = uv + float2(_MainTex_TexelSize.x * 1.0, 0.0) * _BlurSize;
            o.uv[2] = uv - float2(_MainTex_TexelSize.x * 1.0, 0.0) * _BlurSize;
            o.uv[3] = uv + float2(_MainTex_TexelSize.x * 2.0, 0.0) * _BlurSize;
            o.uv[4] = uv - float2(_MainTex_TexelSize.x * 2.0, 0.0) * _BlurSize;
            return o;
        }
​
        float4 fragBlur(Varyings i) : SV_Target
        {
            float weight[3] = {0.4026, 0.2442, 0.0545};
​
            //中心像素值
            // float3 sum = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[0]).rgb * weight[0];
            float3 sum = tex2D(_MainTex, i.uv[0]).rgb * weight[0];
​
            for (int it = 1; it < 3; it++)
            {
                // sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[it * 2 - 1]).rgb * weight[it];
                // sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[it * 2]).rgb * weight[it];
                sum += tex2D(_MainTex, i.uv[it * 2 - 1]).rgb * weight[it];
                sum += tex2D(_MainTex, i.uv[it * 2]).rgb * weight[it];
            }
​
            return float4(sum, 1.0);
        }
        ENDHLSL
​
​
        ZTest Always
        Cull Off
        ZWrite Off
​
        Pass
        {
            Name "GaussianPass0"
            HLSLPROGRAM            
 #pragma vertex VertBlurVertical
 #pragma fragment fragBlur
            ENDHLSL
        }
​
        Pass
        {
            Name "GaussianPass1"
            HLSLPROGRAM            
 #pragma vertex VertBlurHorizontal
 #pragma fragment fragBlur
            ENDHLSL
        }
    }
    Fallback Off
}
```