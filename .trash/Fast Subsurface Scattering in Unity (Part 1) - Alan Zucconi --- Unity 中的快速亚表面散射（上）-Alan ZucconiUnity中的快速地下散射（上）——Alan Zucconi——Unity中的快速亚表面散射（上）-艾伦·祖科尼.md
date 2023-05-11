Most (if not all) optical phenomena that materials exhibit can be replicated by simulating how the individual rays of light propagate and interact. This approach is referred in the scientific literature as **ray tracing**, and it is often too computationally expensive for any real-time application. Most modern engines rely on massive simplifications that, despite being unable to reproduce photorealism, can produce a believable approximation. This tutorial introduces a _fast, cheap and convincing_ solution that can be used to simulate translucent materials which exhibit subsurface scattering.  
材料表现出的大多数（如果不是全部的话）光学现象可以通过模拟单个光线如何传播和相互作用来复制。这种方法在科学文献中被称为射线追踪，对于任何实时应用来说，它的计算成本往往太高。大多数现代引擎都依赖于大规模的简化，尽管无法再现真实感，但可以产生可信的近似值。本教程介绍了一种快速、廉价且令人信服的解决方案，可用于模拟表现出次表面散射的半透明材料。

This is a two part series:  
这是一个由两部分组成的系列：

*   Part 1. [Fast Subsurface Scattering in Unity](https://www.alanzucconi.com/?p=7053)  
    第1部分。Unity中的快速次表面散射
*   Part 2. [Fast Subsurface Scattering in Unity](https://www.alanzucconi.com/?p=7101)  
    第2部分。Unity中的快速次表面散射

At the end of this post, you will find a link to **download** the **Unity project**.  
在这篇文章的最后，你会找到一个下载Unity项目的链接。

#### Introduction

The Standard material in Unity comes with a Transparency mode, which allows rendering transparent materials. Transparency, in this context, is implemented with **alpha blending**. A transparent object is rendered on top of existing geometry, partially showing what is behind. While this works for many materials, transparency is a special case of a more general property, called **translucency** (sometimes also called **translucidity**). While transparent materials only affect the amount of light they let through (below, left), translucent ones can alter its path (below, right).  
Unity中的“标准”材质带有“透明度”模式，该模式允许渲染透明材质。在这种情况下，透明度是通过alpha混合来实现的。透明对象在现有几何体的顶部渲染，部分显示后面的内容。虽然这适用于许多材质，但透明度是一种更为普遍的特性的特例，称为半透明性（有时也称为半透明性）。虽然透明材质只影响它们通过的光量（下图，左），但半透明材质可以改变其路径（下图，右）。

![](<images/1683794158290.png>)

The result of this behaviour should be clear: translucent materials diffuse the light rays they let through, blurring what was behind them. Such a behaviour is rarely seen in games, since it is significantly more complex to implement. Transparent materials can be implemented naively with alpha blending, without ray tracing. Translucent materials, on the other hand, require simulating the deviation of the light rays. Such a computation is very expensive and is rarely worth it in real time rendering.  
这种行为的结果应该是清楚的：半透明材料会散射它们所透过的光线，模糊它们背后的东西。这种行为在游戏中很少出现，因为它的实现要复杂得多。透明材质可以在不进行光线跟踪的情况下通过alpha混合天真地实现。另一方面，半透明材质需要模拟光线的偏移。这样的计算非常昂贵，并且在实时渲染中很少值得。

This often prevents from achieving other optical phenomena, such as **subsurface scattering**. When light hits the surface of a translucent material, a part propagates inside, bouncing between the molecules until it finds its way out. This often causes light absorbed at a specific point to be reemitted somewhere else. Subsurface scattering results in a diffuse glow that can be seen in materials such as skin, marble, and milk.  
这通常会妨碍实现其他光学现象，例如次表面散射。当光照射到半透明材料的表面时，一部分在内部传播，在分子之间反弹，直到找到出路。这通常会导致在特定点被吸收的光在其他地方再次出现。亚表面散射会产生漫射辉光，这种辉光可以在皮肤、大理石和牛奶等材料中看到。

#### Real Time Translucency 实时半透明

There are two main obstacles that make translucency so expensive. The first one is that it requires simulating the scattering of light rays inside a material. Each ray can split in multiple ones, reflecting hundreds or even thousands of times inside a material. The second obstacle is that light received at one point is reemitted somewhere else. While this seems a minor issue, in reality, is a big deal.  
半透明之所以如此昂贵，主要有两个障碍。第一个是它需要模拟光线在材料内部的散射。每条光线可以分裂成多条，在一种材料内反射数百甚至数千次。第二个障碍是在一个点上接收到的光在其他地方重新发射。虽然这看起来是一个小问题，但实际上却是一件大事。

To understand why, we first need to look at how most shaders work. In the realm of real-time rendering, GPUs expect a shader to be able to calculate the final colour of a material simply using local properties. For each vertex, shaders are designed to efficiently access only the properties that are local to that vertex. Reading the normal direction and albedo of a vertex is easy; retrieving the ones of its neighbours is not. Most real-time solution must work around these constraints, and find a way to _fake_ the propagation of light within a material without relying on non-local information.  
要了解原因，我们首先需要了解大多数着色器是如何工作的。在实时渲染领域，GPU希望着色器能够简单地使用局部属性来计算材质的最终颜色。对于每个顶点，着色器被设计为仅有效地访问该顶点的局部属性。读取顶点的法线方向和反照率很容易；检索它的邻居并不是。大多数实时解决方案必须绕过这些限制，并找到一种方法来伪造光在材料内的传播，而不依赖于非局部信息。

The approach described in this tutorial is based on the solution presented at GDC 2011 by Colin Barré-Brisebois and Marc Bouchard in a talk called [Approximating Translucency for a Fast, Cheap and Convincing Subsurface Scattering Look](https://colinbarrebrisebois.com/2011/03/07/gdc-2011-approximating-translucency-for-a-fast-cheap-and-convincing-subsurface-scattering-look/). Their solution is integrated into the **Frostbite 2** engine, which was used for DICE’s **Battlefield 3**. While not being physically accurate, the approach presented by Colin and Marc produces very believable results at a very small cost.  
本教程中描述的方法是基于Colin Barré-Brisebois和Marc Bouchard在2011年GDC上发表的一篇题为“近似半透明以实现快速、廉价和令人信服的亚表面散射外观”的演讲中提出的解决方案。他们的解决方案集成到了用于DICE的《战地3》的霜咬2发动机中。虽然在物理上并不准确，但Colin和Marc提出的方法以非常小的成本产生了非常可信的结果。

The idea behind their solution is very simple. In opaque materials, the light contribution comes directly from the light source. Vertices that are inclined more than 90 degrees in respects to the direction of the light,  
他们的解决方案背后的想法非常简单。在不透明材质中，光的贡献直接来自光源。相对于光的方向倾斜超过90度的顶点，

![](<images/1683794163163.png>)

, receive no light (bottom, left). According to the model proposed in the presentation, translucent materials have an additional light contribution which is related to  
，不接收灯光（底部，左侧）。根据演示中提出的模型，半透明材料具有额外的光贡献，这与

![](<images/1683794164571.png>)

. Geometrically,

![](<images/1683794166037.png>)

can be seen as if some of the light actually passed through the material and made it to the other side (bottom, right).  
可以看到，就好像一些光实际上穿过了材料并到达了另一侧（底部，右侧）。

![](<images/1683794168435.png>)

Each light now accounts for two, distinct reflectances contributions: the front and back illuminations. Since we want our materials to be as realistic as possible, we will use Unity’s Standard PBR lighting models for the front illumination. What we need is to find a way to describe the contribution from  
现在，每种光线都有两种不同的反射率贡献：前照明和后照明。由于我们希望我们的材料尽可能逼真，我们将使用Unity的标准PBR照明模型进行正面照明。我们需要的是找到一种方式来描述

![](<images/1683794169267.png>)

, and render it in a way that somehow simulates the diffusion process which might have occurred inside the material.  
，并以某种方式渲染它，以某种方式模拟可能发生在材质内部的扩散过程。

##### 

![](<images/1683794170042.png>)

Suggested Unity Assets  建议的Unity资产

![](<images/1683794170218.png>)

Unity is free, but you can upgrade to [**Unity Pro**](http://prf.hn/click/camref:1100l45Ay/destination:https://store.unity.com/products/unity-pro) or [**Unity Plus**](http://prf.hn/click/camref:1100l45Ay/destination:https://store.unity.com/products/unity-plus) subscriptions plans to get more functionality and training resources to power up your projects.  
Unity是免费的，但您可以升级到Unity Pro或Unity Plus订阅计划，以获得更多功能和培训资源，为您的项目提供动力。

#### Back Translucency

As discussed before, the final colour of our pixels depend is the sum of two components. The first one is the “traditional” lighting. The second one is the light contribution from a virtual light source illuminating the back of our model. This gives the impression that light from the original source actually passed through the material.  
如前所述，我们像素的最终颜色取决于两个分量的总和。第一个是“传统”照明。第二个是来自照亮我们模型背面的虚拟光源的光贡献。这给人的印象是，来自原始光源的光实际上穿过了材料。

To understand how to model this mathematically, let’s picture the following two scenarios (diagrams below). We are currently drawing the red point; since it’s in the “dark” side of the material, it should be illuminated by  
为了理解如何对其进行数学建模，让我们描绘以下两种场景（下图）。我们目前正在绘制红点；由于它位于材料的“暗”面，因此应该由

![](<images/1683794170392.png>)

. From the perspective of an external viewer, let’s analyse the two extreme cases. We can see that  
。让我们从外部观众的角度来分析这两个极端的案例。我们可以看到

![](<images/1683794171157.png>)

is perfectly aligned with  
与完全一致

![](<images/1683794172573.png>)

, meaning that the viewer  
，意味着观众

![](<images/1683794173402.png>)

should see the back translucency at its fullest. On the other hand, viewer  
应该能看到背部的半透明感。另一方面，观众

![](<images/1683794174194.png>)

should see the least amount of backlight as it is perpendicular to  
应该看到最少的背光，因为它垂直于

![](<images/1683794174916.png>)

.

![](<images/1683794175753.png>)

If you are not new to shader coding, this kind of reasoning should sound familiar. We have encountered something similar in the tutorial on [Physically Based Rendering and Lighting Models in Unity 5](https://www.alanzucconi.com/2015/06/24/physically-based-rendering/), where we showed how such a behaviour can be obtained using a mathematical operator called the **dot product**.  
如果您不是着色器编码的新手，那么这种推理听起来应该很熟悉。我们在Unity 5中的基于物理的渲染和照明模型教程中遇到了类似的情况，我们在教程中展示了如何使用称为点积的数学运算符来获得这种行为。

As a first approximation, we can say that the amount of back lighting due to translucency  
作为第一个近似值，我们可以说由于半透明而产生的背光量

![](<images/1683794177386.png>)

is proportional to  与成比例

![](<images/1683794178813.png>)

. In a traditional diffuse shader, this would be  
。在传统的漫反射着色器中

![](<images/1683794180233.png>)

. We can see that we have not included the **surface normal** in the calculation, as light is simply coming out of the material, not reflecting on it.  
。我们可以看到，我们没有将表面法线包括在计算中，因为光只是从材料中出来，而不是在材料上反射。

#### Subsurface Distortion 地下变形

However, the surface normal should have some influence, even if minor, on the angle at which the light is leaving the material. The authors of this technique introduced a parameter, called **subsurface distortion**   
但是，曲面法线应该对光线离开材质的角度有一些影响，即使影响很小。这项技术的作者引入了一个参数，称为地下畸变

![](<images/1683794182631.png>)

, which forces the vector  
，它强制矢量

![](<images/1683794183965.png>)

to point towards  指向

![](<images/1683794184697.png>)

. Physically speaking, this the subsurface distortion controls how strongly the surface normal deflects the outgoing back light. Following the solution proposed, the intensity of the back translucency component becomes:  
。从物理上讲，这是次表面扭曲控制表面法线偏转出射背光的强度。根据所提出的解决方案，背面半透明成分的强度变为：

![](<images/1683794185461.png>)

Where

![](<images/1683794186945.png>)

is a unit vector that points in the same direction of  
是指向的同一方向的单位向量

![](<images/1683794188412.png>)

. If you are familiar with Cg/HLSL, that is the  
。如果你熟悉Cg/HLSL，那就是

normalize

`normalize`

function.

When

![](<images/1683794189860.png>)

, we return to the  
，我们返回

![](<images/1683794191317.png>)

derived in the previous paragraph. When  
源自上一段。什么时候

![](<images/1683794192742.png>)

, however, we are calculating the dot product between the view direction and  
然而，我们正在计算视图方向和

![](<images/1683794193544.png>)

. If you are familiar with the **Blinn-Phong reflectance**, you should know that  
。如果你熟悉Blinn Phong反射率，你应该知道

![](<images/1683794194988.png>)

is the vector “in between”  
是“介于”之间的矢量

![](<images/1683794196369.png>)

and

![](<images/1683794197139.png>)

. For this reason, we will call it as the **halfway direction**  
。因此，我们将其称为中途方向

![](<images/1683794198619.png>)

.

![](<images/1683794199381.png>)

The diagram above shows all the directions used so far.  
上图显示了迄今为止使用的所有方向。

![](<images/1683794200862.png>)

is indicated in purple, and you can see that it rests in between  
用紫色表示，你可以看到它位于中间

![](<images/1683794202400.png>)

and

![](<images/1683794203797.png>)

. Geometrically speaking, varying  
从几何角度来说，变化

![](<images/1683794204568.png>)

from

![](<images/1683794205309.png>)

to

![](<images/1683794206141.png>)

causes a shift in the perceived direction of the light  
导致光的感知方向发生偏移

![](<images/1683794207518.png>)

. The light shaded area shows the range of directions the backlight will come from. In the image below you can see that with  
浅色阴影区域显示背光将来自的方向范围。在下图中，您可以看到

![](<images/1683794208284.png>)

, the object seems to be illuminated from the purple light source. When  
，物体似乎是由紫色光源照亮的。什么时候

![](<images/1683794210136.png>)

moved towards

![](<images/1683794210942.png>)

, the perceived direction of the light source shifts towards the purple one.  
，光源的感知方向向紫色方向移动。

![](<images/1683794211766.png>)

The purpose of  目的

![](<images/1683794212519.png>)

is to simulate the tendency of certain translucent materials to diffuse the backlight with different intensities. Higher values of  
是模拟某些半透明材料以不同强度散射背光的趋势。的较高值

![](<images/1683794213276.png>)

will cause the back light to scatter more.  
将导致背光散射更多。

![](<images/1683794214121.png>)

Is this H the same H used in the Blinn-Phong Reflectance?  
这个H与“Blinn Phong Reflectance”中使用的H相同吗？

![](<images/1683794214295.png>)

Is 𝛿 really interpolating between L and L+N?  
是𝛿 真的在L和L+N之间插值吗？

![](<images/1683794214456.png>)

How come the authors did not normalise L+N?  
为什么作者没有将L+N标准化？

#### Back Light Diffusion 背光漫射

At this point in the tutorial, we already have an equation that we can use simulate translucent materials. The quantity  
在本教程的这一点上，我们已经有了一个可以用来模拟半透明材质的方程。数量

![](<images/1683794214606.png>)

can not be used to calculate the final light contribution.  
不能用于计算最终的光贡献。

There are two main approaches that can be used. The first one relies on a texture. If you want to have full artistic control on the way light diffuses in the material, you should clamp  
可以使用两种主要方法。第一个依赖于纹理。如果你想对光线在材质中的漫射方式有完全的艺术控制，你应该夹紧

![](<images/1683794215442.png>)

between

![](<images/1683794218612.png>)

and

![](<images/1683794220097.png>)

, and use it to sample the final intensity of the back light. Different ramp textures will simulate the light transport within different materials. We will see in the next part of this tutorial how this can be used to change the result of this shader dramatically.  
，并使用它对背光的最终强度进行采样。不同的渐变纹理将模拟不同材质中的光传输。我们将在本教程的下一部分中看到如何使用它来显著更改此着色器的结果。

The approach used by the author of this technique, however, does not rely on a texture. It creates a curve using Cg code only:  
然而，这项技术的作者所使用的方法并不依赖于纹理。它仅使用Cg代码创建曲线：

![](<images/1683794221814.png>)

The two new parameters,  这两个新参数，

![](<images/1683794222614.png>)

(_power_) and

![](<images/1683794224046.png>)

(_scale_) are used to change the properties of the curve.  
（比例）用于更改曲线的属性。

#### Conclusion

This post explains the technical challenges in rendering translucent materials. An approximate solution is introduced, followed the approach presented by [Approximating Translucency for a Fast, Cheap and Convincing Subsurface Scattering Look](https://colinbarrebrisebois.com/2011/03/07/gdc-2011-approximating-translucency-for-a-fast-cheap-and-convincing-subsurface-scattering-look/). The next part of this tutorial will focus on how to actually implement this effect in a shader in Unity.  
这篇文章解释了渲染半透明材质的技术挑战。介绍了一种近似解，遵循近似半透明的方法，获得快速、廉价和令人信服的亚表面散射外观。本教程的下一部分将重点介绍如何在Unity中的着色器中实际实现此效果。

*   Part 1. **Fast Subsurface Scattering in Unity**  
    第1部分。Unity中的快速次表面散射
*   Part 2. [Fast Subsurface Scattering in Unity](https://www.alanzucconi.com/?p=7101)  
    第2部分。Unity中的快速次表面散射

If you are interested in more sophisticated approaches to simulate subsurface scattering for real time applications, [GPU Gems](https://developer.nvidia.com/gpugems/gpugems/part-iii-materials/chapter-16-real-time-approximations-subsurface-scattering) provides one of the best tutorials you can find.  
如果您对更复杂的方法感兴趣，可以为实时应用程序模拟次表面散射，GPU Gems提供了您能找到的最好的教程之一。

  
You can download all the necessary files to run this project (shader, textures, models, scenes) on [**Patreon**](https://www.patreon.com/posts/14122322).  
您可以下载在Patreon上运行此项目所需的所有文件（着色器、纹理、模型、场景）。

##### 

![](<images/1683794225514.png>)

Support this blog  支持此博客

This websites exists thanks to the contribution of patrons on Patreon. If you think these posts have either helped or inspired you, please consider supporting this blog.  
这个网站的存在要归功于Patreon上顾客的贡献。如果你认为这些帖子对你有帮助或启发，请考虑支持这个博客。

[![](https://www.alanzucconi.com/wp-content/uploads/2017/07/patreon_logo.png)](https://www.patreon.com/AlanZucconi)

[![](https://www.alanzucconi.com/wp-content/uploads/2020/03/youtube_logo_square.png)](https://www.youtube.com/c/AlanZucconi)

##### 

![](<images/1683794267859.png>)

Stay updated

You will be notified when a new tutorial is relesed!  
当新教程发布时，您将收到通知！

##### 

![](<images/1683794268049.png>)

Licensing

You are free to use, adapt and build upon this tutorial for your own projects (even commercially) as long as you credit me.  
只要你相信我，你就可以自由地使用、改编和构建本教程，用于你自己的项目（甚至是商业项目）。

You are not allowed to redistribute the content of this tutorial on other platforms. Especially the parts that are only available on Patreon.  
您不允许在其他平台上重新分发本教程的内容。尤其是Patreon上仅有的零件。

If the knowledge you have gained had a significant impact on your project, a mention in the credit would be very appreciated.  
如果你所获得的知识对你的项目产生了重大影响，请在学分中提及。

![](<images/1683794268200.png>)

![](<images/1683794268351.png>)