在讨论 PBR 时，经常会遇到直接甩出那坨 Cook-Torrance BRDF 公式的情况，这对初次接触 PBR 的小伙伴们可能不太友好。毕竟，PBR 这个名字中带着 “基于物理” 的概念，因此我们需要一些物理基础知识的铺垫，而直接面对公式可能会让人感到有些云里雾里。

因此，在本专栏中，我们将从物理的角度开始，然后逐步过渡到反射方程和微表面理论，最终推导出 Cook-Torrance BRDF 模型。个人认为这样的曲线会更加平滑。如果你具备一定的渲染基础 (可以写出 BlinnPhong 光照模型)，那么阅读完本专栏后，一定能清楚地理解 Cook-Torrance BRDF 模型中每一项的含义以及它的推导过程。

寒霜引擎在 SIGGRAPH 2014 的分享《Moving Frostbite to PBR》中提出，基于物理的渲染的范畴，由三部分组成：

1.  基于物理的材质
2.  基于物理的光照
3.  基于物理适配的摄像机

![[0aacbee6a7db7477fc451dfab16366a9_MD5.jpg]]

所以，第一部分，我们会从物理角度讨论这三个模型。

第一章物理光照，讨论了光的物理定义以及关键的辐射度量学。辐射度量学是最最最基本的物理量，只有理解了它才能知道后面在干嘛。

[TheTus：从零开始的基于物理的渲染 (PBR)(1): 物理光照](https://zhuanlan.zhihu.com/p/633519853)

第二章物理材质和物理相机，讨论了物质对光的相互作用方式以及光学相机的物理定义。我们还将详细讨论微观几何学，这是微表面模型的物理基础。

[TheTus：从零开始的基于物理的渲染 (PBR)(2)：物理材质和物理相机](https://zhuanlan.zhihu.com/p/633527357)

第二部分，我们讨论 PBR 的渲染基础。

第三章 BRDF 和反射方程，介绍 BRDF 的定义和反射方程，这是所有渲染相关方程的基础。

[TheTus：从零开始的基于物理的渲染 (PBR)(3)：BRDF 和反射方程](https://zhuanlan.zhihu.com/p/633534278)

第四章菲涅尔反射、微表面模型和宏观 BRDF。这是为推导 Cook-Torrance BRDF 模型做铺垫的关键章节。我们将详细讨论基于微表面模型的 BRDF 中每个函数的含义和推导过程，并给出宏观 BRDF 方程，它可以看作是所有 BRDF 模型的基础。

[TheTus：从零开始的基于物理的渲染 (PBR)(4)：菲涅尔反射、微表面理论和宏观 BRDF](https://zhuanlan.zhihu.com/p/633554265)

至此，我们有了推导 Cook-Torrance BRDF 模型的全部铺垫了。

第三部分，我们讨论 Cook-Torrance 模型和环境光照。

第五章镜面反射 BRDF 和 Cook-Torrance BRDF 模型。我们将先讨论镜面反射 BRDF，然后推导出 Cook-Torrance BRDF 模型。Cook-Torrance BRDF 是镜面反射 BRDF 模型的一种简化形式，它基于上一章讨论的宏观 BRDF 模型，并在镜面反射条件和微表面理论下得到具体的形式。

[TheTus：从零开始的基于物理的渲染 (PBR)(5)：镜面反射 BRDF 和 Cook-Torrance BRDF 推导](https://zhuanlan.zhihu.com/p/633562876)

第六章讨论环境光照。

待更...

## 参考

下面给出一些我在写本专栏时的参考资料。

[GAMES101](https://sites.cs.ucsb.edu/~lingqi/teaching/games101.html)，渲染基础

[GAMES202](https://www.bilibili.com/video/BV1YK4y1T7yY)，渲染基础

《Real-time Rendering 4th》，个人感觉用来梳理 PBR 很棒

《Physically Base Rendering 3rd》，pbrt，挑着看的

[基于物理渲染 (PBR) 白皮书](https://github.com/QianMo/PBR-White-Paper/tree/master)，毛星云前辈总结的文章

[孙小磊的计算机图形学笔记](https://www.zhihu.com/column/c_1249465121615204352)，复习基础渲染知识

最后还有 PBR 经典的三套 Paper / 演讲，主要是讲 Cook-Torrance BRDF 怎么理解，怎么推导。三篇的推导方式不同，个人感觉 GDC 的最简单：

[Paper: Microfacet Models for Refraction through Rough Surfaces](https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf)

[Paper: Understanding the Masking-Shadowing Function in Microfacet-Based BRDFs](https://jcgt.org/published/0003/02/03/paper.pdf)

[GDC2017: PBR Diffuse Lighting for GGX+Smith Microsurfaces](https://ubm-twvideo01.s3.amazonaws.com/o1/vault/gdc2017/Presentations/Hammon_Earl_PBR_Diffuse_Lighting.pdf)

如果觉得有帮助可以点点赞