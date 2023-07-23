# 快速次表面散射
**Fast Subsurface Scattering**
[Fast Subsurface Scattering in Unity (Part 1) - Alan Zucconi --- Unity中的快速亚表面散射-Alan Zucconi](https://www.alanzucconi.com/2017/08/30/fast-subsurface-scattering-1/)

材料表现出的大多数光学现象可以通过模拟单个光线如何传播和相互作用来复制。这种方法在科学文献中被称为射线追踪（**ray tracing**），并且对于实时应用来说它通常计算量太大。大多数现代引擎依赖于大规模的简化，尽管无法再现照片写实，但它可以产生可信的近似。本教程介绍了一种快速，廉价且令人信服的解决方案，可用于模拟表现出次表面散射的半透明材料。

## Introduction

Unity 中的 Standard 材质带有透明模式 (Transparency mode)，可以渲染透明材质。在这种情况下，透明度是通过 alpha 混合 (**alpha blending**) 实现的。透明对象渲染在现有几何体的顶部 (top of existing geometry)，部分显示后面的内容。虽然这适用于许多材料，但透明度是一种更普遍的属性的特殊情况，称为半透明 (**translucency**)。透明 (transparent) 材料仅影响它们透过的光量（左下图），但半透明（translucent）材料可以改变其路径（右下图）。

![](https://pic3.zhimg.com/v2-fd84a70e43f61aeabd9570e6fcdbda1a_r.jpg)

这种行为的结果应该是清楚的：半透明（translucent）材料会散射它们所穿过的光线，模糊背后的光线。这种行为在游戏中很少见，因为它实现起来要复杂得多。透明（Transparency）材料可以通过 alpha 混合天真地实现，而无需光线跟踪。另一方面，半透明材料需要模拟光线的偏差（deviation of the light rays）。这样的计算非常昂贵，并且在实时渲染中很少值得。

这限制了实现其他光学现象，例如次表面散射（**subsurface scattering**）。当光线照射到半透明（translucent）材料的表面时，一部分在内部传播，在分子之间反弹，直到它找到出路。这通常会导致在特定点吸收的光在其他地方重新发射。次表面散射会产生漫射光（diffuse glow），可以在皮肤，大理石和牛奶等材料中看到。

## Real Time Translucency

有两个主要障碍使半透明如此昂贵。第一个是它需要模拟材料内部光线的散射。每条光线可以分成多个光线，在材料内部反射数百甚至数千次。第二个障碍是在某一点接收的光在其他地方重新发射。虽然这似乎是一个小问题，但事实上，这是一个大问题。

要了解原因，我们首先需要了解大多数着色器的工作原理。在实时渲染领域，GPU 希望着色器能够使用本地属性计算材质的最终颜色。对于每个顶点，着色器旨在有效地仅访问该顶点的局部属性。读取顶点的法线方向（normal direction）和反照率（albedo of a vertex）很容易; 但是检索它的邻居（retrieving the ones of its neighbours）不是。大多数实时解决方案必须解决这些约束，并找到一种方法来伪造（_fake_）材料中的光传播而不依赖于非本地信息。

本教程中描述的方法基于 ColinBarré-Brisebois 和 Marc Bouchard 在 GDC 2011 上提出的解决方案，该解决方案称为 [Approximating Translucency for a Fast, Cheap and Convincing Subsurface Scattering Look.](https://colinbarrebrisebois.com/2011/03/07/gdc-2011-approximating-translucency-for-a-fast-cheap-and-convincing-subsurface-scattering-look/) 他们的解决方案被整合到 Frostbite 2 引擎中，该引擎用于 DICE 的战地 3。虽然不是在物理上准确，但 Colin 和 Marc 提出的方法以非常低的成本产生了非常可信的结果。

他们的解决方案背后的想法非常简单。在不透明材料（opaque materials）中，光贡献直接来自光源。相对于光的方向倾斜超过 90 度的顶点，接收不到收光（左下图）。根据演示文稿中提出的模型，半透明材料具有与之相关的额外光贡献。在几何学上，可以看出一些光实际上是穿过材料并使其到达另一侧（右下图）。

![](https://pic2.zhimg.com/v2-be3ef644bff5c7d25811dce4be778bcd_r.jpg)

现在每个光都有两个不同的反射贡献：正面和背面照明。由于我们希望我们的材料尽可能真实，我们将使用 Unity 的标准 PBR 光照模型进行前照明（front illumination）。我们需要的是找到一种方法来描述来自 - L 的贡献，并以某种方式模拟材料内部可能发生的扩散过程。

## Back Translucency

如前所述，我们像素的最终颜色取决于两个组件的总和。第一个是 “传统”(traditional) 照明。第二个是虚拟光源照亮 (virtual light source illuminating) 我们模型背面的光线贡献。这给人的感觉是来自原始光源的光实际上穿过了材料。

为了理解如何以数学方式对其进行建模，让我们想象以下两种情况（下图）。我们正在绘制红点; 因为它在材料的 “黑暗” 一侧，它应该被 - L 照亮。从外部观察者的角度来看，让我们分析两种极端情况。我们可以看到 - L 和 $V_{B}$ 完全一致，这意味着观众 B 应该充分看到后半透明 (see the back translucency at its fullest.)。另一方面，观察者 A 应该看到量最少 (least amount of backlight), 以为他和 - L 垂直。

![](https://pic2.zhimg.com/v2-15551c55090dcc8f169acd7c319791a9_r.jpg)

我们在 [Physically Based Rendering and Lighting Models in Unity 5](https://www.alanzucconi.com/2015/06/24/physically-based-rendering/) 教程中遇到了类似的东西，其中我们展示了如何使用称为点积（**dot product**）的数学运算符来获得这种行为。

作为第一个近似值 (approximation)，我们可以说由半透明度引起的背光照明量 $I_{back}$ 与 $(V\cdot-L)$ 成正比。在传统的漫反射着色器中，这将是 $(N\cdot L)$ 。我们可以看到，我们没有在计算中包括曲面法线 (**surface normal**)，因为光线只是从材料中出来，而不是反射在材料上。

## Subsurface Distortion (次表面失真)

然而，表面法线应该对光离开材料的角度有一些影响，即使是微小的影响。该技术的作者引入了一个称为次表面失真的参数 $\delta$ (**subsurface distortion**)，该参数强制向量 - L 趋向 N。从物理上讲，法线偏转出射背光（-L）的强度。根据提出的解决方案，后半透明 (back translucency) 成分的强度变为： $I_{back} = V\cdot-(L+N\delta)$ .

$(X) = \frac{X}{||X||}$ 是和 X 同方向的单位向量。如果您熟悉 Cg / HLSL，那就是 normalize 函数。

当 $\delta$ = 0，我们得到 $(V\cdot-L)$ 和前边推导的一样。但是，当 $\delta=1$ 时，我们计算的是观察方向 V 和 （）$-（L+N）$ 之间的点积。

如果您熟悉 Blinn-Phong 反射 (**Blinn-Phong reflectance**)，您应该知道 $(L+N)$ 这是 “介于 L 和 N 两者之间的向量。出于这个原因，我们将其称为中途方向（**halfway direction**）H。

![](https://pic4.zhimg.com/v2-a12f8de80f9fb3d04571bd88af8bdbc7_r.jpg)

上图显示了目前使用的所有方向。H 用紫色表示，你可以看到它位于 L 和 N 之间。**从几何学上讲， $\delta$ 从 0 到 1 导致光 L 的感知方向的变化。粉色阴影区域显示背光源的方向范围。在下图中您可以看到， $\delta=0$ 对象似乎是从紫色光源照亮的。当 $\delta$ 朝向 1 移动时，感知的光源方向向紫色方向移动。**

![](https://pic2.zhimg.com/v2-67e05be69b24c6f51927454bbce6384d_r.jpg)

$\delta$ **目的是模拟某些半透明材料以不同强度扩散背光的趋势。较高的 $\delta$ 值将导致背光散射更多。**

这个 H 与 Blinn-Phong 反射中使用的 H 相同吗？  
不一样，Blinn-Phong 反射率定义 H 为 $(L+V)$ 。在这里，我们使用相同的字母 H 来表示 $(L+N)$ 。  
为什么作者没有 normalise L+N?  
从几何学上讲，L+N 没有单位长度; 因此，它需要规范化。在他们的最终解决方案中，作者没有执行此标准化步骤。最终，这整个效果既不是照片真实的，也不是基于物理的。在他们的介绍中，作者非常清楚地表明它旨在用作半透明和次表面散射行为的快速近似。归一化不会过多地改变结果，但会引入显着的延迟。

## Back Light Diffusion (背光扩散)

我们已经有了一个方程式模拟半透明材料。 $I_{back}$ 不能用于计算最终的光贡献。

有两种主要方法可以使用。第一个依赖于纹理。如果你想对光在材料中漫射的方式进行全面的艺术控制，你应该 clamp $I_{back}$ 在 0 和 1 之间，并用它来采样背光的最终强度。不同的 ramp textures 将模拟不同材料内的光传输。我们将在本教程的下一部分中看到如何使用它来显著更改此着色器的结果。

然而，该技术的作者使用的方法不依赖于纹理。它仅使用 Cg 代码创建曲线： $I_{back} = saturate(V\cdot-(L+N\delta))^{p}\cdot s$

两个新参数 p (power) 和 s (scale) 用于更改曲线的属性。

## Introduction

本教程的前一部分解释了允许近似半透明材料外观的机制。传统表面基于来自方向 L 的光线着色。我们要编写的着色器将添加一个额外的组件 - L，这实际上就像材料被相对的光源照亮一样。这使得它看起来好像光线 L 穿过了材料。

![](https://pic2.zhimg.com/v2-9bbdaab0fa71de6eebb82fb78ba37a4d_r.jpg)

最后，我们得出了一个依赖于观察方向的方程来模拟背光的反射率： $I_{back} = saturate(V\cdot-(L+N\delta))^{p}\cdot s$

*   L 是光的方向 (**light direction**)
*   V 是观察的方向（**view direction**）
*   N 是我们渲染的点（表面法线) 的表面方向。 (**surface normal**).

还有其他参数可用于控制材料的最终外观。例如 $\delta$ ，改变背光的感知方向，使其更符合表面法线：

![](https://pic1.zhimg.com/v2-c3fe02c2d180b9f08c034c5d885ba184_r.jpg)

最后，p 和 s（代表 power 和 scale）确定背光如何传播，并以类似于 **Blinn-Phong reflectance** 中的同音参数的方式工作。

## Extending the Standard Shader（扩展标准着色器）

如前所述，我们希望这种效果尽可能真实。我们最好的选择是扩展 Unity 的 Standard Shader，它已经为非半透明材料提供了非常好的结果。

怎么拓展 Standard Shader？  
如果您不熟悉该过程，则本博客中已广泛介绍了向标准着色器添加功能的具体主题。两个很好的开始教程是 [3D Printer Shader Effect](https://www.alanzucconi.com/2016/10/02/3d-printer-shader-effect-part-1/) 和 [CD-ROM Shader: Diffraction Grating](https://www.alanzucconi.com/2017/07/15/cd-rom-shader-1/)。  
总而言之，基本思想是创建一个新的 **surface shader**，并用自定义着色器替换其 **lighting function**。在那里，我们将调用原始_Standard lighting function_，以获取使用 Unity 的 PBR 着色器渲染的材质。  
一旦我们有了这个，我们就可以计算出背光的贡献，并将它与标准照明功能提供的原始颜色混合。为了一个很好的近似，你可以在这里找到一个很好的起点：

```
#pragma surface surf StandardTranslucent fullforwardshadows
#pragma target 3.0

sampler2D _MainTex;

struct Input {
	float2 uv_MainTex;
};

half _Glossiness;
half _Metallic;
fixed4 _Color;

#include "UnityPBSLighting.cginc"
inline fixed4 LightingStandardTranslucent(SurfaceOutputStandard s, fixed3 viewDir, UnityGI gi) {
	// Original colour
	fixed4 pbr = LightingStandard(s, viewDir, gi);
	
	// ...
	// Alter "pbr" here to include the new light
	// ...

	return pbr;
}

void LightingStandardTranslucent_GI(SurfaceOutputStandard s, UnityGIInput data, inout UnityGI gi) {
	LightingStandard_GI(s, data, gi);		
}

void surf (Input IN, inout SurfaceOutputStandard o) {
	// Albedo comes from a texture tinted by color
	fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
	o.Albedo = c.rgb;
	// Metallic and smoothness come from slider variables
	o.Metallic = _Metallic;
	o.Smoothness = _Glossiness;
	o.Alpha = c.a;
}
```

让我们调用新的 lighting function 来用于此效果 StandardTranslucent。背光将具有与原始光相同的颜色。我们可以控制的是它的强度 I ：

```
#pragma surface surf StandardTranslucent fullforwardshadows

#include "UnityPBSLighting.cginc"
inline fixed4 LightingStandardTranslucent(SurfaceOutputStandard s, fixed3 viewDir, UnityGI gi) {
	// Original colour
	fixed4 pbr = LightingStandard(s, viewDir, gi);
	
	// Calculate intensity of backlight (light translucent)
	float I = ... 
	pbr.rgb = pbr.rgb + gi.light.color * I;

	return pbr;
}
```

为什么 pbr 没有 clamped?  
当将两种颜色加在一起时，应该注意不要超越 1。这通常通过 saturate 完成，该功能简单地将每个颜色分量夹在 0 和 1 之间。  
如果您使用的相机设置为支持 HDR（**high-dynamic range**），则超过 1 的值将用于后期处理效果，例如 bloom。在这个特定的着色器中，我们不会 saturate 最终颜色，因为 bloom filter 将应用于最终渲染。

## Back Lighting

按照本教程第一部分中描述的等式，我们可以继续编写以下代码：

```
inline fixed4 LightingStandardTranslucent(SurfaceOutputStandard s, fixed3 viewDir, UnityGI gi)
{
	// Original colour
	fixed4 pbr = LightingStandard(s, viewDir, gi);
	
	// --- Translucency ---
	float3 L = gi.light.dir;
	float3 V = viewDir;
	float3 N = s.Normal;

	float3 H = normalize(L + N * _Distortion);
	float I = pow(saturate(dot(V, -H)), _Power) * _Scale;

	// Final add
	pbr.rgb = pbr.rgb + gi.light.color * I;
	return pbr;
}
```

上面的代码是本文第一部分中方程式的直接翻译。产生的半透明效果是可信的（下图），但确实与材料的厚度（thickness of the material）没有任何关系。这使得它很难控制。

![](https://pic3.zhimg.com/v2-4192619644aeece6b32a999ef9d5dade_r.jpg)

## Local Thickness（局部厚度）

很明显，背光的量很大程度上取决于材料的密度和厚度。理想情况下，我们需要知道光在材料内部传播的距离，并相应地对其进行衰减。您可以在下图中看到具有相同入射角的三种不同光线如何在材料中穿过非常不同的长度。

![](https://pic4.zhimg.com/v2-c7c09070751f9bfb421177caedd254b7_r.jpg)

但是，从着色器的角度来看，我们无法访问局部几何体或光线的历史（local geometry or the history of the light rays）。不幸的是，没有办法在本地解决这个问题。提出的最佳方法是依赖外部局部厚度图（**local thickness map**）。这是一个纹理，映射到我们的表面，表明该部分材料的 “厚度”。 “厚度” 的概念被宽松地使用，因为实际厚度实际上取决于光来自的角度。

![](https://pic1.zhimg.com/v2-1b41786328096fe4e72e2511c0387bb8_r.jpg)

上图显示了光通过的材料量确实取决于光角。

话虽如此，我们必须记住，这种半透明的方法并不是要在物理上准确，而是要足够现实以愚弄玩家的眼睛。在（[credits](https://colinbarrebrisebois.com/2011/03/07/gdc-2011-approximating-translucency-for-a-fast-cheap-and-convincing-subsurface-scattering-look/)）下方，您可以看到在雕像模型上可视化的良好局部厚度图。白色对应于模型中半透明效果更强的部分，近似于厚度的概念。

![](https://pic4.zhimg.com/v2-351cb0575928f499198b3ae8aa285dff_r.jpg)

How to generate the local thickness map?  
该技术的作者提出了一种从任何模型自动创建局部厚度图的有趣方法。这些是步骤：  
1. Flip the faces of the model  
2. 将环境光遮挡渲染到纹理  
3. 反转纹理的颜色  
这个过程背后的基本原理是，通过在背面上渲染环境遮挡（rendering ambient occlusion），可以大致 “_averages all light transport happening inside the shape“_。  
厚度也可以直接存储在顶点中，而不是纹理

## The Final Version

我们现在知道我们需要考虑材料的局部厚度。最简单的方法是提供我们可以采样的纹理贴图。虽然在物理上不准确，但它可以产生可信的结果。此外，局部厚度的编码方式允许艺术家保持对效果的完全控制。

在此实现中，在 **surf** 函数中采样的附加纹理的红色通道中提供局部厚度：

```
float thickness;

void surf (Input IN, inout SurfaceOutputStandard o) {
	// Albedo comes from a texture tinted by color
	fixed4 c = tex2D (_MainTex, IN.uv_MainTex) * _Color;
	o.Albedo = c.rgb;
	// Metallic and smoothness come from slider variables
	o.Metallic = _Metallic;
	o.Smoothness = _Glossiness;
	o.Alpha = c.a;

	thickness = tex2D (_LocalThickness, IN.uv_MainTex).r;
}
```

为什么不在 lightning function 中采样纹理？  
我已选择将此值存储在名为 thickness 的变量中，稍后将由 lighting function 访问该变量。作为个人喜好，每次我必须对照明功能稍后需要的纹理进行采样时，我倾向于这样做。  
如果您愿意，可以直接在 lighting function 中对纹理进行采样。在这种情况下，您需要传递 UV 坐标（可能扩展 SurfaceOutputStandard）并使用 tex2Dlod 而不是 tex2D。该函数需要两个额外的坐标; 对于此特定应用程序，您可以将它们都设置为零：thickness = tex2Dlod (_LocalThickness, fixed4 (uv, 0, 0)). r;

Colin and Mark 提出了一个稍微不同的方程来计算背光的最终强度。这考虑了厚度（thickness）和附加衰减参数（an additional **attenuation parameter**）。此外，它们还允许始终存在的其他环境组件 (they allow for an additional **ambient component** that is present at all time:)：

```
inline fixed4 LightingStandardTranslucent(SurfaceOutputStandard s, fixed3 viewDir, UnityGI gi)
{
	// Original colour
	fixed4 pbr = LightingStandard(s, viewDir, gi);
	
	// --- Translucency ---
	float3 L = gi.light.dir;
	float3 V = viewDir;
	float3 N = s.Normal;

	float3 H = normalize(L + N * _Distortion);
	float VdotH = pow(saturate(dot(V, -H)), _Power) * _Scale;
	float3 I = _Attenuation * (VdotH + _Ambient) * thickness;

	// Final add
	pbr.rgb = pbr.rgb + gi.light.color * I;
	return pbr;
}
```

最终结果：

![](https://pic4.zhimg.com/v2-5162bedbe8544330b5a149627084750f_r.jpg)