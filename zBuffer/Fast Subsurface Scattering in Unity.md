# 原理
Subsurface Scatting
次表面散射可以使用**光线追踪**实现实时，游戏中更多的是对次表面散射的“快速模拟”。

![[02 PBR理论#^c36ln4]]
![[02 PBR理论#^lxo1ef]]

BSSRDF：
![[Pasted image 20230723134543.png|500]]


高质量的次表面散射使用光线追踪来实现，对于实时渲染来说计算成本太高。大多数现代引擎依赖于大规模的简化，尽管无法再现真实感，但可以产生可信的近似值。**本教程介绍了一种快速、廉价且令人信服的解决方案，可用于模拟表现出次表面散射的半透明材质。**

基于 Alpha Blend 实现的**透明（Transparent）材质**光线是没有散射的（下图左），只影响通过的光量。真实的**半透明（Translucent）材质**会发生散射，改变光线的路径。（下图右）
![[Pasted image 20230723144138.png]]

**次表面散射**：当光照射到半透明材质的表面时，一部分在内部传播，在分子之间反弹，直到找到出路。这通常会导致在特定点被吸收的光在其他地方再次射出。
次表面散射会产生漫射辉光，这种辉光可以在皮肤、大理石和牛奶等材质中看到。
<iframe width="700" height="394" src="https://www.youtube.com/embed/1K9kQi9UZM0" title="Stanford dragon subsurface scattering test with Blender" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

**半透明昂贵的原因：**
1. 它需要模拟光线在材质内部的散射。每条光线可以分成多条，在一种材质内反射数百甚至数千次。
2. 在一个点上接收到的光在其他地方重新发射。这是不容易的。在实时渲染领域，GPU 希望着色器能够简单地使用局部属性来计算材质的最终颜色。每个顶点可以读取自己的属性，但不能读取临近顶点的。大多数实时解决方案必须绕过这些限制，并找到一种在不依赖非局部信息的情况下伪造光在材质内传播的方法。

本教程中描述的方法基于 Colin Barré-Brisebois 和 Marc Bouchard 在 2011 年 GDC 发表的 [Approximating Translucency for a Fast, Cheap and Convincing Subsurface Scattering Look](https://colinbarrebrisebois.com/2011/03/07/gdc-2011-approximating-translucency-for-a-fast-cheap-and-convincing-subsurface-scattering-look/) 用于 DICE 的《战地 3》寒霜 2 引擎

他们的解决方案背后的想法非常简单。在不透明材质（opaque materials）中，光贡献直接来自光源。而半透明材质具有与之相关的额外光贡献。在几何学上，可以看出一些光实际上是穿过材质并使其到达另一侧（右下图）。

![[Pasted image 20230723144233.png]]

**现在每个光都有两个不同的反射贡献：正面和背面照明**。
正面照明（front illumination）：使用 Unity 的标准 PBR 光照模型
背面照明：找到一种方法来描述来自 $- L$ 的贡献，并以某种方式模拟材质内部可能发生的扩散过程。

## 背光
![[Pasted image 20230723135132.png]]
> $dot (N,+L)$ 是正光，$dot (V,-L)$ 是背光

如前所述，我们像素的最终颜色取决于两个组件的总和。第一个是 “传统”(traditional) 照明。第二个是虚拟光源照亮 (virtual light source illuminating) 我们模型背面的光线贡献。这给人的感觉是来自原始光源的光实际上穿过了材质。

数学建模：红点在 “黑暗” 一侧，它应该被 $- L$ 照亮。
![[Pasted image 20230723145227.png]]
作为第一个近似值 (approximation)，我们可以说由半透明度引起的背光照明量 $I_{back}$ 与 $(V\cdot-L)$ 成正比。在传统的漫反射着色器中，这将是 $(N\cdot L)$ 。我们可以看到，**我们没有在计算中包括法线，因为光线只是从材质中出来，而不是反射在材质上。**

## 次表面扰动/扭曲

表面法线应该对光离开材质的角度有微小的影响。该技术的作者引入了一个称为**次表面扰动的参数** $\delta$ ，该参数强制向量 $- L$ 趋向 $N$。从物理上讲， **$\delta$ 控制了法线偏转出射背光的强度**。根据提出的解决方案，背面半透明部分的强度变为：
 $$I_{back} = V\cdot-(L+N\delta)=V\cdot -H$$ .
当 $\delta$ = 0，我们得到 $(V\cdot-L)$ 和前边推导的一样。但是，当 $\delta=1$ 时，我们计算的是观察方向 $V$ 和 $-(L+N)$ 之间的点积。

$L+N$ 即半程向量 $H$（注意，和 BlinnPhong 的半程向量定义不同，BlinnPhong 中为 $H=\frac{L+V}{{|L+V|}}$）
![[Pasted image 20230723145357.png]]

**从几何学上讲， $\delta$ 从 $0$ 到 $1$ 导致光 $L$ 的感知方向的变化。粉色阴影区域显示背光源的方向范围。在下图中您可以看到， $\delta=0$ 对象似乎是从紫色光源照亮的。当 $\delta$ 朝向 1 移动时，感知的光源方向向紫色方向移动。**
![[Pasted image 20230723145824.png]]
> $\delta$ 越大，$-H$ 就越远离 $-L$，散射越多

$\delta$ **目的是模拟某些半透明材质以不同强度散射背光的趋势。较高的 $\delta$ 值将导致背光散射更多。**

## 背光扩散

我们已经有了一个方程式模拟半透明材质。 $I_{back}$ 不能用于计算最终的光贡献。

有两种主要方法可以使用。第一个依赖于纹理。如果你想对光在材质中漫射的方式进行全面的艺术控制，你应该 clamp $I_{back}$ 在 0 和 1 之间，并用它来采样背光的最终强度。不同的 ramp textures 将模拟不同材质内的光传输。我们将在本教程的下一部分中看到如何使用它来显著更改此着色器的结果。

然而，**该技术的作者使用的方法不依赖于纹理。它仅使用 Cg 代码创建曲线**： $I_{back} = saturate(V\cdot-(L+N\delta))^{p}\cdot s$

两个新参数 p(power) 和 s(scale) 用于更改曲线的属性。

# 编程

我们得出了一个依赖于观察方向的方程来模拟背光的反射率： $$I_{back} = saturate(V\cdot-(L+N\delta))^{p}\cdot s$$
* $L$是光源方向
* $V$ 是观察方向
* $N$ 是法线
* $\delta$ ，改变背光的感知方向，使其更符合法线
 - $p$ 和 $s$（代表 power 和 scale）确定背光如何传播，类似高光反射中的参数


## Extending the Standard Shader（扩展标准着色器）

如前所述，我们希望这种效果尽可能真实。我们最好的选择是扩展 Unity 的 Standard Shader，它已经为非半透明材质提供了非常好的结果。

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

上面的代码是本文第一部分中方程式的直接翻译。产生的半透明效果是可信的（下图），但确实与材质的厚度（thickness of the material）没有任何关系。这使得它很难控制。

![](https://pic3.zhimg.com/v2-4192619644aeece6b32a999ef9d5dade_r.jpg)

## 效果增强：局部厚度

很明显，背光量很大程度上取决于材质的密度和厚度。理想情况下，我们需要知道光在材质内部传播的距离，并相应地对其进行衰减。在下图中看到具有相同入射角的三种不同光线在材质中穿过不同的长度（厚度）。

![[Pasted image 20230723152354.png]]

但是，从着色器的角度来看，我们无法访问局部几何体或光线的历史。最佳解决方法是依赖**外部局部厚度图（local thickness map）**。这是一个纹理，映射到我们的表面，表明该部分材质的 “厚度”。“厚度”的概念使用不严格，因为实际厚度实际上取决于光线的角度。

![[Pasted image 20230723152400.png]]

>光线在材质中穿过的长度（厚度）取决于光源角度 $L$。

话虽如此，我们必须记住，这种半透明的方法并不是要在物理上准确，而是要足够现实以愚弄玩家的眼睛。您可以看到在雕像模型上可视化的局部厚度图。白色对应于模型中半透明效果更强的部分，近似于厚度的概念。
![[Pasted image 20230723152815.png]]

> [!question]  如何生成局部厚度图？
> 该技术的作者提出了一种从任何模型自动创建局部厚度图的方法。步骤如下：  
> 1. 翻转模型的面
> 2. 将 AO 渲染到纹理贴图
> 3. 反转纹理的颜色  
> 
> 这个过程背后的基本原理是，通过在背面上渲染 AO，可以大致“平均形状内部发生的所有光传输”。
> 
> 厚度也可以直接存储在顶点中，而不是纹理

我们现在知道我们需要考虑材质的局部厚度。最简单的方法是提供我们可以采样的纹理贴图。虽然在物理上不准确，但它可以产生可信的结果。此外，局部厚度的编码方式允许艺术家保持对效果的完全控制。

在此实现中，在 **surf** 函数中采样的附加纹理的红色通道中提供局部厚度：

```c
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
如果您愿意，可以直接在 lighting function 中对纹理进行采样。在这种情况下，您需要传递 UV 坐标（可能扩展 SurfaceOutputStandard）并使用 tex2Dlod 而不是 tex2D。该函数需要两个额外的坐标; 对于此特定应用程序，您可以将它们都设置为零：thickness = tex2Dlod (_LocalThickness, fixed4(uv, 0, 0)).r;

Colin and Mark 提出了一个稍微不同的方程来计算背光的最终强度。这考虑了厚度（thickness）和附加衰减参数（attenuation）。此外，它们还允许始终存在的其他环境组件：

```c
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
![[Pasted image 20230723153215.png]]

# 思考
![[Pasted image 20230723153544.png]]