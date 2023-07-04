Unity 版本：2022

URP 版本：14.0.0

项目连接：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/ColorGradingLUT)

## 基础概念

### Color Adjustment

色彩调整（Color adjustment）是指对图像中的颜色进行调整，以达到特定的色彩效果或者色彩校正的目的。色彩调整通常包括三个步骤：色彩校正 (color correction)、色彩分级(color grading) 和色调映射(tone mapping)。 色彩矫正 vs 色彩分级： “Correcting is a balance. Grading is a look.”

*   色彩矫正： 色彩校正统一你的电影镜头。 色彩校正使得电影的色调与真实世界的色调保持一致。通过色彩校正，可以让每个视频片段之间的颜色相匹配，从而使它们的色调统一起来。 在色彩矫正过程中，你可以调整诸如曝光、对比度和白平衡，并确保像肤色这样的重要色调得到准确体现。如果你的相机或灯光情况使真实世界中的白色在镜头中出现蓝色，你将在这个阶段纠正这些区域，使其更接近真实的白色。纠正白平衡有助于使你的所有颜色更加真实。 色彩校正不在乎你的风格，而是颜色的正确性。
*   色彩分级： 色彩分级使你的画面更有优势。 在色彩分级阶段，你可以为你影片的着色应用一种整体风格。这为你的项目注入了视觉基调，传达了你希望观众感受到的情感。在调色之前，先进行色彩矫正，以确保开始时有平衡的、自然的色彩。 在色彩矫正过后，可以尝试用色彩分级为影片奠定基调和氛围。如果你的电影是一部残酷的犯罪剧，可以尝试冷色调。而更快乐的视频类型可以尝试用暖色调。

![[8486b8705b6fbbd8b38f205d98c3c3a8_MD5.jpg]]

**色彩校正和色彩分级可以同时进行，合并为同一个步骤，即色彩分级。**（并且一般都合并处理）

### Color Grading LUT

LUT(Look-up Texture) 指的是一种预处理方法。将 key 作为采样坐标，计算出来的 value 作为纹理，预先渲染到纹理中。在需要用到这个键对映射关系时，通过 key 对纹理进行采样，直接得到 value 的值，从而减少了中间计算的消耗。 Color Grading 恰好可以满足这种键对映射关系，所以可以采用 LUT 的方法。

![[8e71591a64f06755052c5e7e78809fa0_MD5.jpg]]

它的缺陷是只能对颜色进行 color grading，无法对饱和度、对比度等使用（因为 RGB 通道被单独分出来了，无法进行组合运算）。

3D LUT 指的是将原 RGB 矢量作为 key，经过 color grading 后的 RGB 矢量作为 value 的键对映射。

![[2013939056f500753d92574b63861860_MD5.jpg]]

3D LUT 不仅可以对颜色进行 color grading，还可以对饱和度、对比度等进行。 3D LUT 纹理的每个像素代表一个颜色，称为 node。3D 纹理的尺寸决定了 nodes 的数量，从而决定了 color grading 的精度。 我们可以将 3D LUT 看作是一个向量空间变换。假设原 RGB 空间的颜色为向量 $\vec{c_{in}}$ ，分级 RGB 空间的颜色为向量 $\vec{c_{out}}$ 。那么 3D LUT 就是一个函数，或者说变换 $\mathbf{M}$ ，它使得 $\vec{c_{out}}=\mathbf{M}\vec{c_{in}}$ 。函数 $\mathbf{M}$ 的值如何确认？将它想象为一个已经画出所有点的以原 RGB 为自变量，分级 RGB 为值的 3D 坐标系。在这个坐标系上找到 $\vec{c_{in}}$ 对应的值 $\vec{c_{out}}$ 即可。

在实际应用中，我们不存储 3D cube 纹理，而是将这张 cube 纹理展开成一张 2D 纹理。（可以想象成把 cube 的每一层平铺为一层）（没找到合适的图，稍微 p 了一下原图）

![[23f91a5a432ca48e2c438b7ed0ca6189_MD5.jpg]]

从这张 3D LUT cubemap 到 2D LUT texture 可以看作一种映射关系，或者说一个变换。

## URP 的 ColorGradingLUT

### 项目设置

首先，确保项目的颜色空间为线性空间。Project Settings->Player->Color Space->Linear。

![[c37a4df1ee24bf415f1666b40b57b742_MD5.jpg]]

然后，打开 RP Asset 的 HDR。

![[e984a574e4b9a98403e45d3cc91023a6_MD5.jpg]]

最后，将相机的 HDR 设置为随项目设置。

![[a8e0058340f5225bced3e58ceb657ce0_MD5.jpg]]

### HDR 分析

下面，我们关闭 Unity 中所有的后处理，仅仅给两个球体赋予 Unlit 材质，左边使用 LDR 颜色，右边使用 HDR 颜色。 先关闭 HDR 选项。 对左边的 LDR 颜色取 (0.5, 1.0, 0.5)，右边的 HDR 颜色吸取左边物体颜色。此时我们发现 HDR 颜色值为 (0.2, 1, 0.2)。这是由于 HDR 颜色最终会显示在 LDR 空间，而这个转换是非线性的。

![[45ba3bcd782088e8b9c5437c6fe20a8d_MD5.jpg]]

此时将 HDR 颜色亮度上调 1，HDR 颜色就更亮了。

![[8f624f7607d858c81f850c1797c531d2_MD5.jpg]]

我们再复制一个 HDR 的 Unlit，将左边 Unlit 的亮度设为 3.0，右边设为 16.0。在关闭 HDR 的情况下，它们都是颜色值一样的纯白。

![[e5ccd2546f4c623b3e06b7537428c972_MD5.jpg]]

此时打开项目的 HDR，发现三个物体的颜色并不会发生任何变化。可是按道理来说右边 Unlit 的亮度应该比左边更亮。这是因为更改为 HDR 后，变化的中间纹理的存储格式，而中间纹理最终会转换到 LDR 空间以显示在屏幕上。我们并没有用中间纹理干事，所以此时打开 HDR 是没有任何效果的。

为了观察打开 HDR 与否的区别，我们打开 Frame Debugger。 关闭项目 HDR，发现中间纹理`_CameraColorAttachmentA`的格式为 sRGB。

![[58669f3ffc89420c344a5040c6ea4241_MD5.jpg]]

打开项目 HDR，发现中间纹理的格式变为了 sFloat。

![[6d24e222e2477ebd51b9736eea0234c4_MD5.jpg]]

### tonemapping

那么，我们该如何利用这些 HDR 格式的中间纹理搞事呢？这就引入了 tonemapping，它将整体颜色压暗，使得极亮之间表现出区别。中间纹理一般与后处理挂钩，所以 tonemapping 一般在后处理阶段进行。我们为场景添加一个挂载了 ACES tonemapping 的 Global Volume。

![[32c8f589047a395201560da90a135133_MD5.jpg]]

应用上 ACES，明显感到所有颜色都变暗了。 当然，在 LDR 空间下，两个极亮的 HDR Unlit 颜色还是没区别。

![[f3a0ce12a9a5a23de0cfaa8574cb8e44_MD5.jpg]]

但我们打开项目的 HDR，发现颜色变亮了，而且右边强度为 16 的 Unlit 物体更亮。

![[1d669e114158f3bb075275449ea2f26f_MD5.jpg]]

### Bloom

另外，Bloom 也能体现出 HDR 的优势。用 Global Vlomue 为场景配置一个 Bloom 效果。 首先开启 HDR，然后调整 HDR 物体的 Unlit 颜色亮度，这样可以看到 Bloom 的辉光效果会变得越来越亮。

![[dc01d68984c66a46c71b392e8077b781_MD5.gif]]

然而，如果关闭 HDR，亮度立刻消失。这是因为 LDR 的纹理不支持如此高的亮度精度，亮度超过 1 的部分被钳位到 1，导致 Bloom 无法判断这些区域是否为极亮的区域。最后，经过 ACES 的压暗，最终左右两个 HDR 颜色显示为一样的亮度。

![[e1153f37c3c13e7d1672795adbd52d1a_MD5.gif]]

### ColorGradingLUT

下面继续分析 Frame Debugger。 在 LDR 空间下，Frame Debugger 长这样。可以看到一个多出来的 Color Grading LUT，注意此时的渲染目标以及 RT 格式。

![[1b61846a845e06cb07339af5eb3a99f3_MD5.jpg]]

使用 Shader 是 LutBuilderLdr。

![[61dee8f06135d27f9eaf041cc38008cb_MD5.png]]

下面打开项目的 HDR，并且把 RP Asset 中 Post-processing 的 Grading Mode 改为 High Dynamic Range。

![[faf151359b64a28b70e0a2d7bbfbf984_MD5.jpg]]

发现 Frame Debugger 中的 ColorGradingLUT 多了一坨新颜色，并且 RT 的格式也变成了 sFloat。并且我们关闭后处理的 tonemapping，这时候的 LUT 几乎是纯色块。

![[20da5ef0f6ff4b2d6d4b6a7ca0e26980_MD5.jpg]]

并且使用的 Shader 是 LutBuilderHdr。

![[1618a096927126a5ea367cd2d25f6da1_MD5.png]]

然后我们打开 ACES Tonemapping，此时的 LUT 有了亮度区分。

![[d2894b936422199d6f944043ae7de8f0_MD5.jpg]]

也就是说，HDR LUT 会根据 tonemapping 的选项，对 HDR 颜色应用 tonemapping。

但若将 RP Asset 中 Post-processing 的 Grading Mode 改为 Low Dynamic Range，ColorGradingLut 又会变为 LDR 的情况。 也就是说，Unity 判断是否生成 HDR LUT 的条件有两个：打开 HDR 并且 RP Asset 的 Grading Mode 为 High Dynamic Range。注意这只是 LUT 是否是 HDR 的，与项目是否是 HDR 无关。

我们应该如何改变 LUT 的颜色对应表呢？答案是给 Global Volume 注入 Color Grading 相关后处理。我们打开 RP 源文件`ColorGradingLutPass.cs`，可以发现 ColorGrading 与以下后处理组件有关。

![[c82dc1a7f98cafba518fa98939f38f58_MD5.jpg]]

我们给 Global Volume 注入一个 Color Adjustments，调节参数，可以发现 LUT 的确发生变化。

![[55efa89b687fe4af6e5e0d5592930919_MD5.jpg]]

也就是说，不论是否进行 Color Grading，ColorGradingLUT 始终是后处理的一部分。

并且还可以发现，LUT 的渲染在 RenderingPrePasses 后，并且将 LUT 渲染给了一张中间纹理`_InternalGradingLUT`。 那么何时应用它呢？我们跳到最后一步 Render PostProcessing Effects 的最后一步 Draw Procedural。 发现它把`_InternalGradingLUT`以及一张`_UserLUT`(尽管我们并没有为后处理添加自己的 LUT) 传给`UberPost.hlsl`。

![[f22e8d4b05b2b7e328f9ad76985788f7_MD5.jpg]]

最后，我们注意到帧调试器里是没有出现 tonemapping 步骤的，这说明 tonemapping 实际上是根据 HDR LUT 开启与否，在最后上图最后一步中根据选项进行的。具体可以从下面代码讨论中分析。

可以发现，ColorGrading 是每一帧都进行的，这是因为判断 colorgrading 是否变换是很麻烦的，特别是在 OverlayCamera 的情况下。由于 lut 的 texel 肯定小于屏幕 texel，所以说性能还是优化了不少的。

### UberPost.hlsl

下面打开`UberPost.hlsl`。 首先可以找到一行多重编译指令。

![[682c9e543ee29da1dd03f44eab0144ab_MD5.png]]

这代表是否启用了 HDR，如果未启用 HDR，则选择使用 aces tonemap 或 neutral tonemap。在 Frame Debugger 上也可以验证这一点。

继续向下，找到片元着色器。其中一行是应用 ColorGrading。注意这一行注释，它也对应了我们之前的推测，即只要打开后处理，color grading 就一直存在。

![[21105919a636d5da576f4dbf9227a687_MD5.png]]

跟踪这个`ApplyColorGrading`。 首先可以确定在渲染 LUT 时，并没有把后曝光渲染进去，而是在空间转换前对输入进行处理。

![[21f4cca2878cf660c75de0a86733f098_MD5.png]]

然后根据 HDR Grading 或 LDR Grading 进行判断。 首先是 HDR Grading。

![[4fa1bc863832a16ebfb2e2fbb2e5c682_MD5.jpg]]

可以发现，HDR Grading，采样 HDR LUT 的作用有两个：第一，将原颜色应用 Color Adjustment。第二，将原颜色转到 LDR 空间。

然后分析 LDR Grading。

![[a526d9f9e38c1ce78fcd2e6746ac2448_MD5.jpg]]

可以发现，LDR Grading，采样 LDR LUT 的作用有一个：将原颜色应用 Color Adjustment。而 HDR 空间到 LDR 空间的转换是通过对输入应用 tonemapping 进行的。

### 总结

最后总结一下：

*   Unity 打开 HDR，更改的是中间纹理的格式
*   打开后处理，ColorGradingLUT（渲染 LUT）就会存在，不论是否进行 Color Grading
*   LUT 的渲染在 RenderingPrePasses 后
*   渲染 HDR LUT 条件
*   项目为 HDR
*   RP Asset 的 Grading Mode 为 HDR
*   LUT 的作用有两种情况
*   LDR LUT：应用 color grading。渲染时只用渲染 color grading 的颜色转换。
*   HDR LUT：应用 color adjustment。渲染时要渲染 color grading 和 tonemapping 的颜色转换。（根据 tonemapping 的选项，如果没有就不 tonemapping）
*   打开后处理，HDR 空间到 LDR 空间输出转换有两种情况
*   RP Asset 的 Grading Mode 为 LDR：应用 tonemapping，模式依照 tonemapping 后处理选项
*   RP Asset 的 Grading Mode 为 HDR：采样 HDR LUT

## LUT 的生成与分析

### 生成 Neutral LUT

想要生成一张 LUT，我们首先需要得到一张中性 LUT(Neutral LUT)，然后对这张中性 LUT 进行 Color Grading。 如何获取一张中性 LUT 呢？这里提供一种通过 Unity 获取的方法。 首先将 RP Asset 中 Post-Processing 的 Grading Mode 改为 LDR，LUT Size 改为 16。

![[12e52fc0c7b95b30184d5ca28d6ea463_MD5.jpg]]

然后打开相机的 Post-Processing。

![[9daac126bb9492599abd4605e76b95dd_MD5.jpg]]

这会让 RP 执行 ColorGradingLUT 步骤，我们只需要保存这一步生成的图像即可。

![[69685a04dca357086ab453614b916353_MD5.jpg]]

用 RenderDoc 截帧，在资源里搜索 lut，找到这张纹理。

![[e9a10d9c6fa133cdca0f821101a52f59_MD5.jpg]]

点`View Contents`转到纹理，再保存即可。需要注意的是，RenderDoc 的纹理是**垂直翻转**了的，我们用 PS 翻转回来即可。

![[087fe1f58ce25cd31c5b3ac2a1195fab_MD5.jpg]]

HDR LUT 同理。这里简述一下在 Unity 里生成 HDR LUT 的设置。打开 HDR(项目和相机)；RP Asset 中 Post-processing 的 Grading Mode 为 HDR，LUTsize 为 32；添加一个 Global Volume，挂载一个 Tonemapping，改为 ACES。

两张 LUT 如下：

![[d4443f50c020c134db836e416008fd8c_MD5.png]]

![[4fac7cbe759523d449eeb492665315a2_MD5.png]]

连接：[github](https://github.com/dyxdyxdyx/TA-100/tree/master/Assets/2.7%20HDR/Textures)

### 应用 ColorGrading

这里简单讲一下思路，就是对上面一步得到的 Neutral LUT 进行 ColorGrading。为了方便观察，我们可以先对一张照片进行 ColorGrading，然后再将这些 ColorGrading 的参数应用到 Neutral LUT 中。

下面是我用 PS 对 Neutral LUT 应用 ColorGrading 后的一张 LUT。

![[46a5d945aaaaef3549207f19f3116bc5_MD5.png]]

连接：[github](https://github.com/dyxdyxdyx/TA-100/tree/master/Assets/2.7%20HDR/Textures)

### LUT 分析

### 应用 LUT

以 LDR Neutral LUT 为例，HDR LUT 在 LDR LUT 的基础上做了一个 tonemapping（前面也分析了）。 首先，我们将 LUT 逐通道分开。如下图，可以发现不同程度的 blue 将 LUT 分为了 16 块。每个块中，red 从左到右由 0 到 1，green 从下到上由 0 到 1。LUT 中，最左下角一定是黑色，右上角一定是白色。

![[19c1995cc8610fb73b36ee37873e1b3b_MD5.jpg]]

也就是说，对于每一个块，red 和 blue 映射了该块程度 blue 下的一种颜色。

![[32f861524f74ae453e2e5b64d35de084_MD5.jpg]]

那么，对这张 LUT 的采样步骤也很明确了。首先，通过 blue 确定当前块的索引。然后，计算当前块中的 red 偏移得到 $u$ ，最后计算 green 偏移得到 $v$ 。 以这张 16x16 的 LUT 为例。 首先我们需要计算得到当前 blue 对应的块的索引。 $blue\in[0,1]$ ，一共有 16 块，我们需要将它映射到 $block\in[0,15]$ 。 $block = ⌊blue*15⌋$ _。然后计算这个块的基准_ $u$ _坐标，_ $u_{base}\in[0, 16*15=240]$ ，则 $u_{base}=u_{perBlockOffset}\times block=\frac{240}{15}\times block$ 。然后通过 red 计算 $u$ 基准偏移。同理，red 将 block 分为 16 小块， $u_{offset}=⌊red*15⌋/15\times 15$ _。_此时 $u=u_{base}+u_{offset}\in[0,255]$ _，_则 $u=(u_{base}+u_{offset})/255$ 。 然后计算 $v$ ，同理，green 将 block 分为 16 小块，则 $v=⌊green*15⌋/15$ 。 以上代码对应描述如下：

```
float u = floor(color.b * 15.0) / 15.0 * 240.0;
    u = (floor(color.r * 15.0) / 15.0 * 15.0) + u;
    u /= 255.0;

    float v = floor(color.g * 15.0);
    v /= 15.0;

    half3 left = SAMPLE_TEXTURE2D(tex, samplerTex, half2(u, v)).rgb;
```

然而这样采样出来的颜色过渡十分突兀，这是因为两个 block 间的 blue 直接进行了一个 level 的突变。

![[0b86ab3c5fd6f9f27db5b39f21cf6c38_MD5.jpg]]

所以，我们还需要采样偏右边的 block，然后对左边和右边 block 结果进行混合。

![[8ee94280ceaa75dc40ce6b29a51f3f25_MD5.jpg]]

代码如下。

```
u = ceil(color.b * 15.0) / 15.0 * 240.0;
    u = (ceil(color.r * 15.0) / 15.0 * 15.0) + u;
    u /= 255.0;

    v = ceil(color.g * 15.0) / 15.0;

    half3 right = SAMPLE_TEXTURE2D(tex, samplerTex, half2(u, v)).rgb;

    color = lerp(left, right, frac(color * 15));
```

## URP 应用 LUT

首先在 Unity 放一张 LUT 纹理，由于我们需要在 sRGB 对这张贴图采样，所以取消勾选 sRGB，让这张贴图不进行 Degamma。其次，由于在 Shader 计算时，为了减少性能消耗，不采用上面的对 rg 也进行 floor 的操作，将 FilterMode 改为 Bilinear（否则为 Point）。最后，取消 Mipmap 的生成，将 Warp Mode 改为 Clamp，MaxSize 改为 LUT 大小，Compression 改为 High Quality。

![[7013fd75b43e5b4f6875476fd05770fe_MD5.jpg]]

### LDR LUT

下面先写一个最基础的 LDR LUT。 我们需要在 RP 中插入一个 RenderPass，所以需要先创建一个 RenderFeature。它创建了一个`ApplyColorLut`Shader 的材质，并且将材质和参数传递给 RenderPass 的构造函数。最后，RenderPass 入队。 **ColorGradingLutRendererFeature.cs**

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;
using UnityEngine.Rendering.Universal.Internal;

struct ColorGradingLutParams{
    public Texture customLut;
    public float contribution;
}

internal class ColorGradingLutRendererFeature : ScriptableRendererFeature{
    private const string mShaderName = "Hidden/PostProcess/ApplyColorLut";

    #region Params Define

    public Texture customLut = null;
    [Range(0.0f, 1.0f)] public float contribution = 0.0f;

    #endregion

    private Material mMaterial;
    private ColorGradingLutParams mLutParams;

    private ApplyColorLutRenderPass mApplyColorLutRenderPass = null;

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData) {
        if (renderingData.cameraData.postProcessEnabled)
            if (mApplyColorLutRenderPass.isActive())
                renderer.EnqueuePass(mApplyColorLutRenderPass);
    }

    public override void Create() {
        mMaterial = CoreUtils.CreateEngineMaterial(mShaderName);
        mLutParams.customLut = customLut;
        mLutParams.contribution = contribution;
        mApplyColorLutRenderPass = new ApplyColorLutRenderPass(mMaterial, mLutParams);
        mApplyColorLutRenderPass.renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
    }

    protected override void Dispose(bool disposing) {
        CoreUtils.Destroy(mMaterial);
    }
}
```

然后写对应的 RenderPass。在构造函数中对传递参数赋值，然后在`Execute`函数中写相应的渲染逻辑，首先用 Shader 把 source 渲染到临时纹理，然后把临时纹理复制到 des。 在上述 LUT 图像分析中，我们需要将其参数一般化。为了避免在 Shader 里进行除法，我们以 LUT 纹素 $texel\in[0,1]$ 为范围，我们需要计算出一个颜色格子的大小 $gridSize=1/width\times 1/height$ ，然后计算出一个 block 的大小 $blockSize=1/height\times 1/height$ ，最后需要颜色格的范围 $[0,height-1]$ 。所以，我们需要向 Shader 传递的参数 $lutParams = (1/width, 1/height, height-1)$ 。

**ApplyColorLutRenderPass.cs**

```
using UnityEngine;  
using UnityEngine.Rendering;  
using UnityEngine.Rendering.Universal;  

internal class ApplyColorLutRenderPass : ScriptableRenderPass{  
    private ProfilingSampler mProfilingSampler = new ProfilingSampler("ApplyColorLut");  
    private Material mMaterial;  

    private ColorGradingLutParams mLutParams;  

    private RTHandle mTempRT0;  
    private const string mTempRT0Name = "_TemporaryRenderTexture0";  

    private int mCustomLutId = Shader.PropertyToID("_CustomLut"),  
        mCustomLutParamsId = Shader.PropertyToID("_CustomLutParams"),  
        mContributionId = Shader.PropertyToID("_Contribution");  

    private const string mHDRGradingKeyword = "_HDR_GRADING",  
        mTonemapACESKeyword = "_TONEMAP_ACES";  

    public bool isActive() => mMaterial != null && mLutParams.customLut != null;  

    public ApplyColorLutRenderPass(Material material, ColorGradingLutParams lutParams) {  
        mMaterial = material;  
        mLutParams = lutParams;  

        mTempRT0 = RTHandles.Alloc(mTempRT0Name, name: mTempRT0Name);  
    }  
    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData) {  
        var cmd = CommandBufferPool.Get("ApplyColorLut");  
        context.ExecuteCommandBuffer(cmd);  
        cmd.Clear();  

        mMaterial.SetTexture(mCustomLutId, mLutParams.customLut);  
        mMaterial.SetFloat(mContributionId, mLutParams.contribution);  
        mMaterial.SetVector(mCustomLutParamsId, new Vector4(1.0f / mLutParams.customLut.width, 1.0f / mLutParams.customLut.height, mLutParams.customLut.height - 1.0f, 0.0f));  

        var renderer = renderingData.cameraData.renderer;  
        var source = renderer.cameraColorTargetHandle;  
        var destination = renderer.cameraColorTargetHandle;  

        var descriptor = renderingData.cameraData.cameraTargetDescriptor;  
        descriptor.msaaSamples = 1;  
        descriptor.depthBufferBits = 0;  

        RenderingUtils.ReAllocateIfNeeded(ref mTempRT0, descriptor, name: mTempRT0Name, filterMode: FilterMode.Bilinear);  
        using (new ProfilingScope(cmd, mProfilingSampler)) {  
            cmd.Blit(source, mTempRT0, mMaterial, 0);  
        }        
        cmd.Blit(mTempRT0, destination);  

        cmd.ReleaseTemporaryRT(Shader.PropertyToID(mTempRT0.name));  

        context.ExecuteCommandBuffer(cmd);  
        cmd.Clear();  
        CommandBufferPool.Release(cmd);  
    }}
```

关于注入点：按理来说，ApplyLut 应该是所有后处理之后进行，但是这里面涉及到 tonemapping，如果在系统后处理后面，系统后处理的最后一步会调用 tonemapping 将颜色钳位 0 到 1，导致我们获取不了 HDR 颜色。所以将注入点放到后处理前。这样做会导致所有系统自带后处理用不了的情况，但是实际情况下根本不可能自己写一个 Lut 后处理，所以这里只是用来理解一下原理。

然后写 shader 文件。 **ApplyColorLut.shader**

```
Shader "Hidden/PostProcess/ApplyColorLut" {
    Properties {
        [HideInInspector] _MainTex ("Base (RGB)", 2D) = "white" {}
    }

    SubShader {
        Tags {
            "RenderType" = "Opaque"
            "RenderPipeline" = "UniversalPipeline"
        }
        LOD 200
        HLSLINCLUDE
        #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"
        #include "../Common/PostProcessing.hlsl"
        #include "ApplyColorLutPass.hlsl"
        ENDHLSL
        Pass {
            name "ApplyColorLUT Pass"

            HLSLPROGRAM
            #pragma vertex Vert
            #pragma fragment frag

            #pragma shader_feature _TONEMAP_ACES
            ENDHLSL
        }
    }
}
```

**PostProcessing.hlsl**

```
#ifndef POSTPROCESSING_INCLUDED
#define POSTPROCESSING_INCLUDED

#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

TEXTURE2D(_MainTex);
SAMPLER(sampler_MainTex);

TEXTURE2D(_CameraDepthTexture);
SAMPLER(sampler_CameraDepthTexture);

struct Attributes {
    float4 positionOS : POSITION;
    float2 uv : TEXCOORD0;
};

struct Varyings {
    float2 uv : TEXCOORD0;
    float4 vertex : SV_POSITION;
    UNITY_VERTEX_OUTPUT_STEREO
};

half4 GetSource(float2 uv) {
    return SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, uv);
}

half4 GetSource(Varyings input) {
    return GetSource(input.uv);
}

Varyings Vert(Attributes input) {
    Varyings output = (Varyings)0;
    // 分配instance id
    UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);

    VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
    output.vertex = vertexInput.positionCS;
    output.uv = input.uv;

    return output;
}

#endif
```

在具体的 Pass 文件中，我们需要在片元着色器中获取片元颜色，对它进行 Lut，然后输出。 **ApplyColorLutPass.hlsl**

```
TEXTURE2D(_CustomLut);  
SAMPLER(sampler_CustomLut);  

float4 _CustomLutParams; 
float _Contribution;

half4 frag(Varyings input) : SV_Target {
    half3 color = GetSource(input).xyz;
    color = ApplyColorGrading(color, TEXTURE2D_ARGS(_CustomLut, sampler_CustomLut));
    return half4(color, 1.0);
}
```

在应用 ColorGrading 函数中，我们需要先将颜色从线性空间转到 sRGB 空间（因为 Lut 取消勾选 sRGB，此时它不会进行 Degamma，它一直在 sRGB 空间内），然后引用这个 sRGB color 采样 Lut，最后转换回线性空间进行输出。 **ApplyColorLutPass.hlsl**

```
half3 ApplyColorGrading(half3 input, TEXTURE2D_PARAM(customLutTex, customLutSampler)) {
    saturate(input);
    input.rgb = LinearToSRGB(input);
    half3 outLut = ApplyLut(TEXTURE2D_ARGS(customLutTex, customLutSampler), input, _CustomLutParams.xyz);
    input = lerp(input, outLut, _Contribution);
    input.rgb = SRGBToLinear(input.rgb);
    return input;
}
```

在 ApplyLut 函数中，我们需要首先计算出 block 的索引，然后计算出

 $\begin{align} u=u_{base}+u_{offset}+u_{toCenter}=block*blockWidth+gridIndex_{red}*gridWidth+0.5*gridWidth\\ v=v_{offset}+v_{toCenter}=gridIndex_{green}*gridHeight+0.5*gridHeight \end{align}$

前面提到， $gridSize=1/width\times 1/height,~blockSize=1/height\times 1/height$ 。 为了减少消耗，我们取消对 $gridIndex$ 计算时的 floor。 代码如下：

```
// scaleOffset = (1 / lut_width, 1 / lut_height, lut_height - 1)
// gridSize = scaleOffset.x x scaleOffset.y, blockSize = scaleOffset.y
real3 ApplyLut(TEXTURE2D_PARAM(tex, samplerTex), float3 color, float3 scaleOffset) {
    // 计算block索引
    color.b *= scaleOffset.z;
    float block = floor(color.b);
    // 计算在u = uBase+uOffset
    float u = block * scaleOffset.y + color.r * scaleOffset.z * scaleOffset.x + scaleOffset.x * 0.5;
    float v = color.g * scaleOffset.z * scaleOffset.y + scaleOffset.y * 0.5;
    color.rgb = lerp(
        SAMPLE_TEXTURE2D_LOD(tex, samplerTex, float2(u,v), 0.0).rgb,
        SAMPLE_TEXTURE2D_LOD(tex, samplerTex, float2(u,v)+float2(scaleOffset.y, 0.0), 0.0).rgb,
        color.b - block); // 根据Blue在block位置插值

    return color;
}
```

根据前面在 URP 的 ColorGradingLUT 中的分析，LDR Lut 还需要对颜色进行一次 tonemapping，这里直接使用 Color 库的函数，本来写了个丐版 aces，结果效果也太丐了。注意，添加 shader_feature，并且在脚本里进行控制，这里就不展示了。

```
float3 ApplyTonemapping(half3 input) {
    #ifdef _TONEMAP_ACES
    input = min(input, 60.0);
    input = AcesTonemap(unity_to_ACES(input));
    #endif

    // 把颜色钳位到0-1 输出LDR颜色
    return saturate(input);
}
half3 ApplyColorGrading(half3 input, TEXTURE2D_PARAM(customLutTex, customLutSampler)) {
    input = ApplyTonemapping(input);
    ...
}
```

如图，这是应用 LDR Lut 后的效果。

![[facad6c90b9438538bd7cd61e8a8a7ae_MD5.gif]]

可以看到，由于精度问题，应用 LUT 后的颜色过渡显然更明显了，比如绿色球的高光部分。 这是 16x32 的 LDR Lut 应用另一个 Color Grading 的效果。

![[720516e9d18ff4fa4fa6268fbd985d87_MD5.jpg]]

最后，写了一个高斯 Bloom 注入在 ApplyLut 相同注入点前面，测试一下 tonemapping 的效果。效果和系统的是一样的。

![[52f433b62dd8b0034b9695ad75482b4f_MD5.gif]]

### HDR LUT

前面 URP 中的 ColorGradingLut 提到，HDR LUT 的存储信息有两个：第一个，ColorGrading。第二个，Tonemapping。 由于 URP Lut 自带 Tonemapping，所以我们只需要对 HDR LUT 进行采样即可。需要注意的是，HDR LUT 的定义空间在 LogC，所以需要将输入值转到 LogC 后进行采样。

```
half3 ApplyColorGrading(half3 input, TEXTURE2D_PARAM(customLutTex, customLutSampler)) {
 #ifdef _HDR_GRADING
    float3 inputLutSpace = saturate(LinearToLogC(input));
    half3 outLut = ApplyLut(TEXTURE2D_ARGS(customLutTex, customLutSampler), inputLutSpace, _CustomLutParams.xyz);
    input = lerp(input, outLut, _Contribution);
 #else
    input = ApplyTonemapping(input);
    input.rgb = LinearToSRGB(input);
    half3 outLut = ApplyLut(TEXTURE2D_ARGS(customLutTex, customLutSampler), input, _CustomLutParams.xyz);
    input = lerp(input, outLut, _Contribution);
    input.rgb = SRGBToLinear(input.rgb);
 #endif

    return input;
}
```

应用 HDR LUT 前：

![[29c0cf4e0310d0ea437e7a488b11a3e8_MD5.jpg]]

应用 HDR LUT 后：

![[b8b43c9ad9df5c32f9af160b2df1cf9f_MD5.jpg]]

## URP 渲染 LUT

### 生成默认 LUT 带

首先需要在 RendererFeature 中添加生成 LUT 的 RenderPass。 **ColorGradingRendererFeature.cs**

```
using Unity.VisualScripting;
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;
using UnityEngine.Rendering.Universal.Internal;
using UnityEngine.Serialization;

struct ColorGradingLutParams{
    public bool hdr;
    public bool aces;
    public int lutSize;
}

struct ApplyColorLutParams{
    public bool hdr;
    public bool aces;
    public Texture customLut;
    public float contribution;
}

internal class ColorGradingLutRendererFeature : ScriptableRendererFeature{
    private const string mApplyColorLutShaderName = "Hidden/PostProcess/ApplyColorLut";
    private const string mLutBuilderShaderName = "Hidden/PostProcess/LutBuilder";

    #region Params Define

    [Header("ColorGradingLut")] public bool hdr = false;
    public bool aces = false;
    public int lutSize = 16;

    [Header("CustomColorLut")] 

    public Texture customLut = null;
    [Range(0.0f, 1.0f)] public float contribution = 0.0f;

    #endregion

    private Material mApplyColorLutMaterial;
    private Material mColorGradingLutMaterial;

    private ApplyColorLutParams mApplyColorLutParams;
    private ColorGradingLutParams mColorGradingLutParams;

    private ApplyColorLutRenderPass mApplyColorLutRenderPass = null;
    private ColorGradingLutRenderPass mColorGradingLutRenderPass = null;

    private const string mInternalLutName = "_InternalLut";

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData) {
        if (renderingData.cameraData.postProcessEnabled) {
            if (mApplyColorLutRenderPass.isActive())
                renderer.EnqueuePass(mApplyColorLutRenderPass);
            if (mColorGradingLutRenderPass.isActive())
                renderer.EnqueuePass(mColorGradingLutRenderPass);
        }
    }

    public override void Create() {
        mColorGradingLutMaterial = CoreUtils.CreateEngineMaterial(mLutBuilderShaderName);
        mApplyColorLutMaterial = CoreUtils.CreateEngineMaterial(mApplyColorLutShaderName);

        mColorGradingLutParams.hdr = hdr;
        mColorGradingLutParams.aces = aces;
        mColorGradingLutParams.lutSize = lutSize;

        mApplyColorLutParams.hdr = hdr;
        mApplyColorLutParams.aces = aces;
        mApplyColorLutParams.customLut = customLut;
        mApplyColorLutParams.contribution = contribution;

        mColorGradingLutRenderPass = new ColorGradingLutRenderPass(mColorGradingLutMaterial, mColorGradingLutParams);
        mColorGradingLutRenderPass.renderPassEvent = RenderPassEvent.AfterRenderingPrePasses;

        mApplyColorLutRenderPass = new ApplyColorLutRenderPass(mApplyColorLutMaterial, mApplyColorLutParams);
        mApplyColorLutRenderPass.renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
    }

    protected override void Dispose(bool disposing) {
        CoreUtils.Destroy(mApplyColorLutMaterial);
    }
}
```

然后填写这个 RenderPass。传递的参数在后面进行解释。 **ColorGradingRenderPass.cs**

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

internal class ColorGradingLutRenderPass : ScriptableRenderPass{
    private ProfilingSampler mProfilingSampler = new ProfilingSampler("CustomColorGradingLut");
    private Material mMaterial;

    private ColorGradingLutParams mColorGradingLutParams;

    private RTHandle mInternalLut;

    private int mLutParamsId = Shader.PropertyToID("_LutParams");

    private const string mInternalLutName = "_CustomInternalLut";

    public ColorGradingLutRenderPass(Material material, ColorGradingLutParams colorGradingLutParams) {
        mMaterial = material;
        mColorGradingLutParams = colorGradingLutParams;

        mInternalLut = RTHandles.Alloc(mInternalLutName, name: mInternalLutName);
    }

    public bool isActive() => mMaterial != null;

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData) {
        var cmd = CommandBufferPool.Get();
        context.ExecuteCommandBuffer(cmd);
        cmd.Clear();

        var lutHeight = mColorGradingLutParams.lutSize;
        var lutWidth = lutHeight * lutHeight;

        // params = (lut_height, 0.5 / lut_width, 0.5 / lut_height, lut_height / lut_height - 1)
        mMaterial.SetVector(mLutParamsId, new Vector4(lutHeight, 0.5f / lutWidth, 0.5f / lutHeight, lutHeight / (lutHeight - 1.0f)));

        RenderTextureDescriptor descriptor = new RenderTextureDescriptor(lutWidth, lutHeight, RenderTextureFormat.ARGB32, 0, 1);

        RenderingUtils.ReAllocateIfNeeded(ref mInternalLut, descriptor, name: mInternalLutName, filterMode: FilterMode.Bilinear);
        cmd.SetRenderTarget(mInternalLut, RenderBufferLoadAction.DontCare, RenderBufferStoreAction.Store);

        using (new ProfilingScope(cmd, mProfilingSampler)) {
            cmd.DrawProcedural(Matrix4x4.identity, mMaterial, 0, MeshTopology.Triangles, 3);
        }

        // 设置全局纹理，让ApplyColorLut可以访问到
        cmd.SetGlobalTexture(mInternalLut.name, mInternalLut.nameID);
        cmd.ReleaseTemporaryRT(Shader.PropertyToID(mInternalLut.name));

        context.ExecuteCommandBuffer(cmd);
        cmd.Clear();
        CommandBufferPool.Release(cmd);
    }
}
```

下面写 Shader 代码。首先，我们使用绘制一个全屏三角形的方式进行纹理渲染，在顶点着色器中，根据顶点 id 判断顶点在全屏三角形的裁剪空间位置和 uv 插值坐标并返回。

![[fb446d851602aaa80bdcbd7e58d12041_MD5.jpg]]

```
float4 _LutParams;

struct Varyings {
    float4 positionCS : SV_POSITION;
    float2 uv : VAR_SCREEN_UV;
};

Varyings vert(uint vertexID : SV_VertexID) {
    Varyings output;
    // 根据id判断三角形顶点的坐标
    // 坐标顺序为(-1, -1) (-1, 3) (3, -1)
    output.positionCS = float4(vertexID <= 1 ? -1.0 : 3.0, vertexID == 1 ? 3.0 : -1.0, 0.0, 1.0);
    output.uv = float2(vertexID <= 1 ? 0.0 : 2.0, vertexID == 1 ? 2.0 : 0.0);
    // 不同API可能会产生颠倒的情况 进行判断
    if (_ProjectionParams.x < 0.0) {
        output.uv.y = 1.0 - output.uv.y;
    }
    return output;
}
```

片元着色器中，我们根据 uv 坐标得到当前的 color 并输出。如果使用 HDR，则返回 LogC 空间颜色。

```
half4 frag(Varyings input) : SV_Target {
    half3 color = GetDeafultLutValue(input.uv, _LutParams);
    #ifdef _HDR_GRADING
    return half4(LogCToLinear(color), 1.0);
    #else
    return half4(color, 1.0);
    #endif
}half4 frag(Varyings input) : SV_Target {
    half3 color = GetDeafultLutValue(input.uv, _LutParams);
    #ifdef _HDR_GRADING
    return half4(LogCToLinear(color), 1.0);
    #else
    return half4(color, 1.0);
    #endif
}
```

下面推导如何根据 uv 坐标计算 color，这其实就是根据 color 计算 lut uv 的反过程。 首先写出根据 color 计算 uv 的公式：

 $\begin{align} block &= floor(b * (height-1))\\ u &= block * (1/height) + r * (height-1) * (1/width) + 0.5/width\\ v &= g * (height-1) * (1/height) + 0.5/height\\ \end{align}$

我们先将偏移到 texel 中心的 offset 减去， $u-=0.5/width,~v-=0.5/height$ 。 然后先计算简单的 $v=g*height/(height-1)$ 。 然后计算 $r$ 。

 $\begin{align} u * height &= block + r * (height-1) * (1/width) * height\\ &=block+r*(height-1)/height\\ r*(height-1)/height&=u*height-block\\ &=u*height-floor(b*(height-1)) \end{align}$

由于 U 轴将 blue 均分为 $height$ 个程度，所以 $u\in[b,b+1/height)$ ，所以 $u*height\in[b*height,b*height+1)$ _。 所以_ $floor(b*(height-1))\leq u*height<floor(b*(height-1))+1$ 。 即 $u*height-floor(b*(height-1))$ 为 $u*height$ _的小数部分_ $frac(u*height)$ 。 综上所述， $r*(height-1)/height=frac(u*height)$ ， $r=frac(u*height)*height/(height-1)$ 。 这这里，我们令 $floor(b*(height-1))=b*(height-1)$ ，则 $b*(height-1)=u*height-r*(height-1)/height$ _，得_ $b*(height-1)/height=u-r*(height-1)/(height*height)$ 。所以 $b=(u-r*(height-1)/(height*height))*height/(height-1)$ 。 将 rgb 的公共参数提出来，得到 $params=(height, 0.5/width, 0.5/height, height/height-1$ 。

整理得到：

 $\begin{align} uv -&= params.yz\\ r&=frac(uv.x*params.x)\\ b&=u-r/params.x\\ g&=v\\ rgb&=params.w \end{align}$

所以，生成默认 LUT 带的函数如下：

```
// params = (lut_height, 0.5 / lut_width, 0.5 / lut_height, lut_height / lut_height - 1)
real3 GetDeafultLutValue(float2 uv, float4 params) {
    uv -= params.yz;
    real3 color;
    color.r = frac(uv.x * params.x);
    color.b = uv.x - color.r / params.x;
    color.g = uv.y;
    return color * params.w;
}
```

首先看一条 16x16 的 LDR LUT。

![[cf0ee62d9583f13dc793a64cb5f7a1e9_MD5.jpg]]

然后再看一条 32x32 的 HDR LUT。由于没进行 tonemapping，结果是色块。

![[b1a5c09492051899c0ebdc85e0fb0767_MD5.jpg]]

### ColorGrading

首先，我们需要声明 Inspector 属性。 **ColorGradingLutRendererFeature.cs**

```
#region Params Define

    [Header("ColorGradingLut")] public bool hdr = false;
    public bool aces = false;
    public int lutSize = 16;

    [Header("ColorAdjustments")] public float postExposure;
    [Range(-100f, 100f)] public float contrast; // 对比度
    [ColorUsage(false, true)] public Color colorFilter = Color.white; // 颜色滤镜 没有alpha的HDR颜色
    [Range(-180f, 180f)] public float hueShift; // 色相偏移
    [Range(-100f, 100f)] public float saturation; // 饱和度

    [Header("CustomColorLut")] public Texture customLut = null;
    [Range(0.0f, 1.0f)] public float contribution = 0.0f;

    #endregion
```

然后，在 RenderPass 中传递相应的参数。

```
// 将颜色调整属性发给material 曝光度、对比度、色相偏移和饱和度
        mMaterial.SetVector(mColorAdjustmentsId, new Vector4(
            Mathf.Pow(2f, mColorAdjustmentsParams.postExposure), // 曝光度 曝光单位是2的幂次
            mColorAdjustmentsParams.contrast * 0.01f + 1f, // 对比度 将范围从[-100, 100]转换到[0, 2]
            mColorAdjustmentsParams.hueShift * (1f / 360f), // 色相偏移 将范围从[-180, 180]转换到[-1, 1] ([-0.5, 0.5] ?)
            mColorAdjustmentsParams.saturation * 0.01f + 1f // 饱和度 将范围从[-100, 100]转换到[0, 2]
        ));
        mMaterial.SetColor(mColorFilterId, mColorAdjustmentsParams.colorFilter.linear); // 颜色滤镜 线性
```

按照 URP ColorGradingLut 的方式，后曝光是不渲染进 Lut 的，它通过采样时的预乘计算。所以我们把后曝光值发给 GPU。并且把 internalLut 的参数也发给 GPU。

```
// 设置全局参数，让ApplyColorLut能够访问到
        cmd.SetGlobalTexture(mInternalLut.name, mInternalLut.nameID); // _CustomInternalLut 纹理
        cmd.SetGlobalFloat(mPostExposureId, Mathf.Pow(2f, mColorAdjustmentsParams.postExposure)); // _PostExposure
        // params (1.0/width, 1.0/height, height-1.0)
        cmd.SetGlobalVector(mCustomInternalLutPramsId, new Vector4(1.0f / lutWidth, 1.0f / lutHeight, lutHeight - 1.0f, 0.0f));// _CustomInternalLutParams
```

获取完默认 Lut 带后，我们需要对对应的 color 应用 ColorGrading，并且还需要传递 tonemapping 方式。 **LutBuilderPass.hlsl**

```
half4 frag(Varyings input) : SV_Target {
    float3 color = GetDeafultLutValue(input.uv, _LutParams);
    #ifdef _HDR_GRADING
    #ifdef _TONEMAP_ACES
    return half4(ColorGrade(LogCToLinear(color), true), 1.0);
    #else
    return half4(ColorGrade(LogCToLinear(color)), 1.0);
    #endif
    #else
    return half4(ColorGrade(color), 1.0);
    #endif
}
```

ColorGrading 函数如下，具体解释照注释。注意，HDR Lut 需要渲染 tonemapping 转换。 **LutBuilderPass.hlsl**

```
// 根据色调映设空间(是否ACES) 返回对应亮度
float Luminance(float3 color, bool useACES) {
    return useACES ? AcesLuminance(color) : Luminance(color);
}

// 对比度
float3 ColorGradingContrast(float3 color, bool useACES) {
    // 为了更好的效果 如果使用ACES 则将颜色从线性空间转换到ACEScc空间 否则转换到logC空间
    color = useACES ? ACES_to_ACEScc(unity_to_ACES(color)) : LinearToLogC(color);
    // 从颜色中减去均匀的中间灰度，然后通过对比度进行缩放，然后在中间添加中间灰度
    color = (color - ACEScc_MIDGRAY) * _ColorAdjustments.y + ACEScc_MIDGRAY;
    // 将颜色转换回线性空间
    return useACES ? ACES_to_ACEScg(ACEScc_to_ACES(color)) : LogCToLinear(color);
}

// 颜色过滤
float3 ColorGradeColorFilter(float3 color) {
    // 将颜色与颜色滤镜相乘
    return color * _ColorFilter.rgb;
}

// 色相偏移
float3 ColorGradingHueShift(float3 color) {
    // 将颜色格式从rgb转换为hsv
    color = RgbToHsv(color);
    // 将色相偏移添加到h
    float hue = color.x + _ColorAdjustments.z;
    // 如果色相超出范围 将其截断
    color.x = RotateHue(hue, 0.0, 1.0);
    // 将颜色格式从hsv转换为rgb
    return HsvToRgb(color);
}

// 饱和度
float3 ColorGradingSaturation(float3 color, bool useACES) {
    // 获取颜色的亮度
    float luminance = Luminance(color, useACES);
    // 从颜色中减去亮度，然后通过饱和度进行缩放，然后在中间添加亮度
    return (color - luminance) * _ColorAdjustments.w + luminance;
}

half3 ColorGrade(float3 color, bool useAces = false) {
    // 对比度
    color = ColorGradingContrast(color, useAces);
    color = ColorGradeColorFilter(color);
    // 当对比度增加时，会导致颜色分量变暗，在这之后将颜色钳位
    color = max(color, 0.0);
    // 色相偏移
    color = ColorGradingHueShift(color);
    color = ColorGradingSaturation(color, useAces);
    // 当饱和度增加时，可能产生负数，在这之后将颜色钳位
    // 如果是ACES空间，则把颜色从ACEcg空间转回ACES 并且应用Aces tonemapping
    return max(useAces ? AcesTonemap(ACEScg_to_ACES(color)) : color, 0.0);
}
```

首先检查一下默认 HDR Lut Aces。

![[06c55170d7b4e776a8ebf8d46e581b82_MD5.jpg]]

然后检查一下 ColorGrading 后的情况。

![[9fc09c805610c7d8a1572de22adae474_MD5.jpg]]

### ApplyColorGrading

回到之前写好的`ApplyColorLutPass.hlsl`，把 internalLut 的纹理采样的结果进行 lerp 即可。注意在 HDR Lut 下，我们认为自定义 lut 仅仅进行 ColorGrading，internalLut 进行 ColorGrading 和 Tonemapping，所以 InternalLut 在 LogC 空间下采样，自定义 lut 在 sRGB 空间下采样。

```
half3 ApplyColorGrading(half3 input, float postExposure, TEXTURE2D_PARAM(lutTex, lutSampler), float3 lutParams, TEXTURE2D_PARAM(customLutTex, customLutSampler), float3 customLutParams,
                        float customLutContrib) {
    input *= postExposure;
    #ifdef _HDR_GRADING
    // Internal HDR Lut 需要进行ColorGrading+Tonemapping，所以在LogC空间
    float3 inputLutSpace = saturate(LinearToLogC(input));
    input = ApplyLut2D(TEXTURE2D_ARGS(lutTex, lutSampler), inputLutSpace, lutParams);

    UNITY_BRANCH
    // Custom Lut 只需要进行ColorGrading，所以在sRGB空间
    if(customLutContrib > 0.0) {
        input = saturate(input);// LDR color
        input.rgb = LinearToSRGB(input.rgb);// In LDR do the lut in sRGB for the user Lut
        half3 outLut = ApplyLut(TEXTURE2D_ARGS(customLutTex, customLutSampler), inputLutSpace, _CustomLutParams.xyz);
        input = lerp(input, outLut, customLutContrib);
        input.rgb = SRGBToLinear(input.rgb);// turn back to Linear Space
    }
    #else
    // 首先进行tonemapping（根据设置）
    input = ApplyTonemapping(input);

    UNITY_BRANCH
    if (customLutContrib > 0.0) {
        // 转到sRGB空间采样LUT
        input.rgb = LinearToSRGB(input.rgb);
        half3 outLut = ApplyLut(TEXTURE2D_ARGS(customLutTex, customLutSampler), input, customLutParams);
        input = lerp(input, outLut, customLutContrib);
        input.rgb = SRGBToLinear(input.rgb);
    }

    input = ApplyLut(TEXTURE2D_ARGS(lutTex, lutSampler), input, lutParams);
    #endif

    return input;
}

half4 frag(Varyings input) : SV_Target {
    half3 color = GetSource(input).xyz;
    color = ApplyColorGrading(color, _PostExposure, TEXTURE2D_ARGS(_CustomInternalLut, sampler_CustomInternalLut), _CustomInternalLutParams.xyz, TEXTURE2D_ARGS(_CustomLut, sampler_CustomLut),
                              _CustomLutParams.xyz, _Contribution);
    return half4(color, 1.0);
}
```

如图，这是一张应用 LDR ColorGradingLut 的图像。

![[a7c4416b0a56d3a97fc5dc70503108a3_MD5.jpg]]

这是应用 HDR ColorGradingLut 的图像。

![[9f80a0c52e0f55b583265e2ade44dea1_MD5.jpg]]

这是应用 HDR ColorGradingLut Aces 的图像。

![[8298b8fffbb439940594339dd8f5828a_MD5.jpg]]

## 参考

[catlikecoding: custom srp](https://catlikecoding.com/unity/tutorials/custom-srp)

[Adobe: Color correction vs. color grading: What's the difference?](https://www.adobe.com/creativecloud/video/discover/color-correction-vs-color-grading.html)

[What is the LUT](https://www.youtube.com/watch?v=3ZpbUOGDWLE)

[The Beginner's Guide to LUTs](https://www.redsharknews.com/post-vfx/item/2966-the-beginners-guide-to-luts)

[3D Game Shaders For Beginners:LUT](https://lettier.github.io/3d-game-shaders-for-beginners/lookup-table.html)