高质量的次表面散射使用光线追踪来实现，对于实时渲染来说计算成本太高。大多数现代引擎依赖于大规模的简化，尽管无法再现真实感，但可以产生可信的近似值。**本教程介绍了一种快速、廉价且令人信服的解决方案，可用于模拟表现出次表面散射的半透明材料。**

基于 Alpha Blend 实现的**透明（Transparent）材质**光线是没有散射的（下图左），只影响通过的光量。真实的**半透明（Translucent）材质**会发生散射，改变光线的路径。（下图右）
![[Pasted image 20230723142312.png]]

次表面散射：当光照射到半透明材料的表面时，一部分在内部传播，在分子之间反弹，直到找到出路。这通常会导致在特定点被吸收的光在其他地方再次射出。
次表面散射会产生漫射辉光，这种辉光可以在皮肤、大理石和牛奶等材质中看到。
<iframe width="700" height="394" src="https://www.youtube.com/embed/1K9kQi9UZM0" title="Stanford dragon subsurface scattering test with Blender" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

半透明昂贵的原因：
1. 它需要模拟光线在材料内部的散射。每条光线可以分成多条，在一种材料内反射数百甚至数千次。
2. 在一个点上接收到的光在其他地方重新发射。这是不容易的。在实时渲染领域，GPU 希望着色器能够简单地使用局部属性来计算材质的最终颜色。每个顶点可以读取自己的属性，但不能读取临近顶点的。大多数实时解决方案必须绕过这些限制，并找到一种在不依赖非局部信息的情况下伪造光在材料内传播的方法。

本教程中描述的方法基于 Colin Barré-Brisebois 和 Marc Bouchard 在 2011 年 GDC 发表的 [Approximating Translucency for a Fast, Cheap and Convincing Subsurface Scattering Look](https://colinbarrebrisebois.com/2011/03/07/gdc-2011-approximating-translucency-for-a-fast-cheap-and-convincing-subsurface-scattering-look/) 用于 DICE 的《战地 3》寒霜 2 引擎

# 原理

The idea behind their solution is very simple. In opaque materials, the light contribution comes directly from the light source. Vertices that are inclined more than 90 degrees in respects to the direction of the light,

![[3e10ca3f0fa97a107f57c751bd63093b_MD5.svg]]

, receive no light (bottom, left). According to the model proposed in the presentation, translucent materials have an additional light contribution which is related to

![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]]

. Geometrically,

![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]]

can be seen as if some of the light actually passed through the material and made it to the other side (bottom, right).  
该方案非常简单。在不透明材质中，光的贡献直接来自光源。相对于灯光 ![[3e10ca3f0fa97a107f57c751bd63093b_MD5.svg]] 的方向倾斜超过90度的顶点不接收灯光（底部，左侧）。根据演示中提出的模型，半透明材料具有与 ![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]] 相关的额外光贡献。从几何角度来看， #2可以被视为一些光实际上穿过了材料并到达了另一侧 （底部，右侧）。

![[f2ea9f985f9f15a64bac2cfeadd2ceef_MD5.png]]

Each light now accounts for two, distinct reflectances contributions: the front and back illuminations. Since we want our materials to be as realistic as possible, we will use Unity’s Standard PBR lighting models for the front illumination.  
现在，每种光都有两种不同的反射率：前照明和后照明。由于我们希望我们的材料尽可能逼真，我们将使用Unity的标准PBR照明模型进行正面照明。  
What we need is to find a way to describe the contribution from

![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]]

, and render it in a way that somehow simulates the diffusion process which might have occurred inside the material.  
我们需要的是找到一种方法来描述 ![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]] 的贡献，并以某种方式模拟材料内部可能发生的扩散过程来渲染它。

##### ⭐ Suggested Unity Assets ⭐  
⭐ 建议的Unity资产⭐

Unity is free, but you can upgrade to [**Unity Pro**](http://prf.hn/click/camref:1100l45Ay/destination:https://store.unity.com/products/unity-pro) or [**Unity Plus**](http://prf.hn/click/camref:1100l45Ay/destination:https://store.unity.com/products/unity-plus) subscriptions plans to get more functionality and training resources to power up your projects.  
Unity是免费的，但您可以升级到Unity Pro或Unity Plus订阅计划，以获得更多功能和培训资源，为您的项目提供动力。

#### Back Translucency 背面半透明

As discussed before, the final colour of our pixels depend is the sum of two components. The first one is the “traditional” lighting. The second one is the light contribution from a virtual light source illuminating the back of our model.  
如前所述，像素的最终颜色取决于两个分量的总和。第一种是“传统”照明。第二个是来自照亮我们模型背面的虚拟光源的光贡献。  
This gives the impression that light from the original source actually passed through the material.  
这给人的印象是，来自原始光源的光实际上穿过了材料。

To understand how to model this mathematically, let’s picture the following two scenarios (diagrams below). We are currently drawing the red point; since it’s in the “dark” side of the material, it should be illuminated by

![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]]

. From the perspective of an external viewer, let’s analyse the two extreme cases. We can see that

![[3f0f1123c3afcf4db8101323106954ed_MD5.svg]]

is perfectly aligned with

![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]]

, meaning that the viewer

![[2f5577fff48b59357424c3e7b4ae467d_MD5.svg]]

should see the back translucency at its fullest. On the other hand, viewer

![[02a9c8f55c26a899a6baef3776e90d6c_MD5.svg]]

should see the least amount of backlight as it is perpendicular to

![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]]

.  
为了理解如何对其进行数学建模，让我们描绘以下两个场景（下图）。我们目前正在绘制红点；由于它位于材质的“暗”面，因此应该由 ![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]] 照亮。让我们从外部观察者的角度来分析这两个极端的案例。我们可以看到 ![[3f0f1123c3afcf4db8101323106954ed_MD5.svg]] 与 ![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]] 完全对齐，这意味着观看者 ![[2f5577fff48b59357424c3e7b4ae467d_MD5.svg]] 应该看到最充分的背部半透明。另一方面，观看者#4应该看到最少量的背光，因为它垂直于 ![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]] 。

![[b5ab8a1abc61020e59dcc6fc96a9886e_MD5.png]]

If you are not new to shader coding, this kind of reasoning should sound familiar. We have encountered something similar in the tutorial on [Physically Based Rendering and Lighting Models in Unity 5](https://www.alanzucconi.com/2015/06/24/physically-based-rendering/), where we showed how such a behaviour can be obtained using a mathematical operator called the **dot product**.  
如果您不是着色器编码的新手，那么这种推理听起来应该很熟悉。我们在Unity 5中的基于物理的渲染和照明模型教程中遇到了类似的情况，我们在教程中展示了如何使用称为点积的数学运算符来获得这种行为。

As a first approximation, we can say that the amount of back lighting due to translucency

![[77be0d22c18fa441799ca59d997d6530_MD5.svg]]

is proportional to

![[a6755234b26e09e34643504574dc533e_MD5.svg]]

. In a traditional diffuse shader, this would be

![[71aaed56eafdeb65c7c2232a4625eebf_MD5.svg]]

. We can see that we have not included the **surface normal** in the calculation, as light is simply coming out of the material, not reflecting on it.  
作为第一近似，我们可以说由于半透明性 ![[77be0d22c18fa441799ca59d997d6530_MD5.svg]] 引起的背光量与 ![[a6755234b26e09e34643504574dc533e_MD5.svg]] 成比例。在传统的漫反射着色器中，这将是#2。我们可以看到，我们没有将表面法线包括在计算中，因为光只是从材料中出来，而不是在材料上反射。

#### Subsurface Distortion 地下变形

However, the surface normal should have some influence, even if minor, on the angle at which the light is leaving the material. The authors of this technique introduced a parameter, called **subsurface distortion** 

![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]]

, which forces the vector

![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]]

to point towards

![[8a94681732dbd73fe32666914e2f3c3a_MD5.svg]]

. Physically speaking, this the subsurface distortion controls how strongly the surface normal deflects the outgoing back light. Following the solution proposed, the intensity of the back translucency component becomes:  
但是，曲面法线应该对光离开材质的角度有一些影响，即使影响很小。该技术的作者引入了一个称为次表面失真 ![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]] 的参数，该参数迫使向量 ![[de6b0f7e4174ac10effa6705cd6fac87_MD5.svg]] 指向 ![[8a94681732dbd73fe32666914e2f3c3a_MD5.svg]] 。从物理上讲，这种次表面扭曲控制了曲面法线偏转出射背光的强度。根据所提出的解决方案，背面半透明成分的强度变为：

![[3c0b27df270d9ce1735988f1dc1fda2d_MD5.svg]]

Where

![[116071e829dca877596167040768da2a_MD5.svg]]

is a unit vector that points in the same direction of

![[5e9cfe60d018a42d920f0469bbb1bfea_MD5.svg]]

. If you are familiar with Cg/HLSL, that is the  
其中 ![[116071e829dca877596167040768da2a_MD5.svg]] 是指向 ![[5e9cfe60d018a42d920f0469bbb1bfea_MD5.svg]] 的相同方向的单位向量。如果您熟悉Cg/HLSL，那就是

normalize 规范化

`normalize`

function.

When

![[34fbd8f7c4af29ae87ced60c97f93fcd_MD5.svg]]

, we return to the

![[e8281c5f786ec63a08b539c3f47862e4_MD5.svg]]

derived in the previous paragraph. When

![[d97b9610c75470a253a8370fa71d2835_MD5.svg]]

, however, we are calculating the dot product between the view direction and

![[5ddf21469c28af4c04473ba8e1914c79_MD5.svg]]

. If you are familiar with the **Blinn-Phong reflectance**, you should know that

![[bac4e4acfb1e3ab7893fa4c6b2c420a6_MD5.svg]]

is the vector “in between”

![[3e10ca3f0fa97a107f57c751bd63093b_MD5.svg]]

and

![[8a94681732dbd73fe32666914e2f3c3a_MD5.svg]]

. For this reason, we will call it as the **halfway direction**

![[3a8afad22c8964cae762a4502d073895_MD5.svg]]

.  
当 ![[34fbd8f7c4af29ae87ced60c97f93fcd_MD5.svg]] 时，我们返回到前一段中派生的 ![[e8281c5f786ec63a08b539c3f47862e4_MD5.svg]] 。然而，当#2时，我们正在计算视图方向和#3之间的点积。如果您熟悉Blinn Phong反射，您应该知道#4是“介于” ![[3e10ca3f0fa97a107f57c751bd63093b_MD5.svg]] 和 ![[8a94681732dbd73fe32666914e2f3c3a_MD5.svg]] 之间的向量。因此，我们将其称为中间方向 ![[3a8afad22c8964cae762a4502d073895_MD5.svg]] 。

![[418302b2701d3908854aab294af5ba8c_MD5.png]]

The diagram above shows all the directions used so far.

![[3a8afad22c8964cae762a4502d073895_MD5.svg]]

is indicated in purple, and you can see that it rests in between

![[3e10ca3f0fa97a107f57c751bd63093b_MD5.svg]]

and

![[8a94681732dbd73fe32666914e2f3c3a_MD5.svg]]

. Geometrically speaking, varying

![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]]

from

![[410dbcc5a2353eb7b4744509d1576118_MD5.svg]]

to

![[5eff21c228bf24ccad0cf9832e9bca51_MD5.svg]]

causes a shift in the perceived direction of the light

![[3e10ca3f0fa97a107f57c751bd63093b_MD5.svg]]

. The light shaded area shows the range of directions the backlight will come from. In the image below you can see that with

![[34fbd8f7c4af29ae87ced60c97f93fcd_MD5.svg]]

, the object seems to be illuminated from the purple light source. When

![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]]

moved towards

![[5eff21c228bf24ccad0cf9832e9bca51_MD5.svg]]

, the perceived direction of the light source shifts towards the purple one.  
上图显示了迄今为止使用的所有方向 ![[3a8afad22c8964cae762a4502d073895_MD5.svg]] 用紫色表示，您可以看到它位于 ![[3e10ca3f0fa97a107f57c751bd63093b_MD5.svg]] 和 ![[8a94681732dbd73fe32666914e2f3c3a_MD5.svg]] 之间。从几何角度讲，从#4到 ![[5eff21c228bf24ccad0cf9832e9bca51_MD5.svg]] 改变#3会导致光 ![[3e10ca3f0fa97a107f57c751bd63093b_MD5.svg]] 的感知方向发生偏移。浅色阴影区域显示背光的方向范围。在下图中，您可以看到使用 ![[34fbd8f7c4af29ae87ced60c97f93fcd_MD5.svg]] ，对象似乎是由紫色光源照亮的。当 ![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]] 向 ![[5eff21c228bf24ccad0cf9832e9bca51_MD5.svg]] 移动时，光源的感知方向向紫色移动。

![[3c974e7fba06cace6044d51ca5a5bc15_MD5.png]]

The purpose of

![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]]

is to simulate the tendency of certain translucent materials to diffuse the backlight with different intensities. Higher values of

![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]]

will cause the back light to scatter more. ![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]] 的目的是模拟某些半透明材料以不同强度散射背光的趋势。 ![[7b83dd97a534ee5f4956f01f429280f3_MD5.svg]] 的值越高，背光散射越大。

❓ Is this H the same H used in the Blinn-Phong Reflectance?  
❓ 这个H与“Blinn Phong Reflectance”中使用的H相同吗？

❓ Is 𝛿 really interpolating between L and L+N?  
❓ 是𝛿 真的在L和L+N之间插值？

❓ How come the authors did not normalise L+N?  
❓ 为什么作者没有将L+N标准化？

#### Back Light Diffusion 背光漫射

At this point in the tutorial, we already have an equation that we can use simulate translucent materials. The quantity

![[77be0d22c18fa441799ca59d997d6530_MD5.svg]]

can not be used to calculate the final light contribution.  
在教程的这一点上，我们已经有了一个可以用来模拟半透明材质的方程式。数量 ![[77be0d22c18fa441799ca59d997d6530_MD5.svg]] 不能用于计算最终的光贡献。

There are two main approaches that can be used. The first one relies on a texture. If you want to have full artistic control on the way light diffuses in the material, you should clamp

![[77be0d22c18fa441799ca59d997d6530_MD5.svg]]

between

![[410dbcc5a2353eb7b4744509d1576118_MD5.svg]]

and

![[5eff21c228bf24ccad0cf9832e9bca51_MD5.svg]]

, and use it to sample the final intensity of the back light. Different ramp textures will simulate the light transport within different materials. We will see in the next part of this tutorial how this can be used to change the result of this shader dramatically.  
可以使用两种主要方法。第一个依赖于纹理。如果希望对灯光在材质中的漫射方式进行完全的艺术控制，则应将 ![[77be0d22c18fa441799ca59d997d6530_MD5.svg]] 夹在 ![[410dbcc5a2353eb7b4744509d1576118_MD5.svg]] 和 ![[5eff21c228bf24ccad0cf9832e9bca51_MD5.svg]] 之间，并使用它对背光的最终强度进行采样。不同的渐变纹理将模拟不同材质中的光传输。我们将在本教程的下一部分中看到如何使用它来显著更改此着色器的结果。

The approach used by the author of this technique, however, does not rely on a texture. It creates a curve using Cg code only:  
然而，这项技术的作者所使用的方法并不依赖于纹理。它仅使用Cg代码创建曲线：

![[da7dd174778008ac8c26c31cc3b56af2_MD5.svg]]

The two new parameters,

![[64ceea966e694f8f5a1b7636ddcb027f_MD5.svg]]

(_power_) and

![[05c6a4cae1c47ec968f751cd6c9f08a9_MD5.svg]]

(_scale_) are used to change the properties of the curve.  
两个新参数 ![[64ceea966e694f8f5a1b7636ddcb027f_MD5.svg]] （功率）和 ![[05c6a4cae1c47ec968f751cd6c9f08a9_MD5.svg]] （比例）用于更改曲线的特性。

#### Conclusion 结论

This post explains the technical challenges in rendering translucent materials. An approximate solution is introduced, followed the approach presented by [Approximating Translucency for a Fast, Cheap and Convincing Subsurface Scattering Look](https://colinbarrebrisebois.com/2011/03/07/gdc-2011-approximating-translucency-for-a-fast-cheap-and-convincing-subsurface-scattering-look/). The next part of this tutorial will focus on how to actually implement this effect in a shader in Unity.  
这篇文章解释了渲染半透明材质的技术挑战。介绍了一种近似解，遵循了近似半透明的方法，以获得快速、廉价和令人信服的亚表面散射外观。本教程的下一部分将重点介绍如何在Unity中的着色器中实际实现此效果。

*   Part 1. **Fast Subsurface Scattering in Unity**  
    第1部分。Unity中的快速次表面散射
*   Part 2. [Fast Subsurface Scattering in Unity](https://www.alanzucconi.com/?p=7101)  
    第2部分。Unity中的快速次表面散射

If you are interested in more sophisticated approaches to simulate subsurface scattering for real time applications, [GPU Gems](https://developer.nvidia.com/gpugems/gpugems/part-iii-materials/chapter-16-real-time-approximations-subsurface-scattering) provides one of the best tutorials you can find.  
如果您对实时应用中模拟次表面散射的更复杂方法感兴趣，GPU Gems提供了您能找到的最好的教程之一。

  
You can download all the necessary files to run this project (shader, textures, models, scenes) on [**Patreon**](https://www.patreon.com/posts/14122322).  
您可以下载在Patreon上运行此项目所需的所有文件（着色器、纹理、模型、场景）。

##### 💖 Support this blog  
💖 支持此博客

This website exists thanks to the contribution of patrons on Patreon. If you think these posts have either helped or inspired you, please consider supporting this blog.  
这个网站的存在要归功于Patreon上的赞助人的贡献。如果你认为这些帖子对你有帮助或启发，请考虑支持这个博客。

[![[0c3d2dbb741234c033de23e6353206be_MD5.png]]](https://www.patreon.com/AlanZucconi)

[

![[f3c58f57f1780ee8adf69004e1e3fc20_MD5.png]]

](https://www.patreon.com/bePatron?u=850572)

[![[9575c46ce3be18fce4c45421b07185d2_MD5.png]]](https://www.youtube.com/c/AlanZucconi)

##### 📧 Stay updated

You will be notified when a new tutorial is released!

##### 📝 Licensing

You are free to use, adapt and build upon this tutorial for your own projects (even commercially) as long as you credit me.

You are not allowed to redistribute the content of this tutorial on other platforms, especially the parts that are only available on Patreon.

If the knowledge you have gained had a significant impact on your project, a mention in the credit would be very appreciated. ❤️🧔🏻