
---
title: 4 PBR推导
aliases: []
tags: []
create_time: 2023-07-10 14:10
uid: 202307101410
banner: "![[Pasted image 20230710141009.png]]"
---


# Cook-Torrance BRDF 推导

说明一下，不同的文献给出了不同的推导方式，下面给出我认为最简单的 [GDC2017: PBR Diffuse Lighting for GGX+Smith Microsurfaces](https://ubm-twvideo01.s3.amazonaws.com/o1/vault/gdc2017/Presentations/Hammon_Earl_PBR_Diffuse_Lighting.pdf) 中的推导。另外两种推导方式，我会在本节最后给出相应 paper 和参考文章。

由于 Cook-Torrance 模型基于镜面反射，每个微平面都是完美菲涅尔镜，只有反射向量为观察向量时，这个微平面才对接收能量有贡献。因此，我们只考虑 $m=h$ 的情况，将其带入宏观 BRDF 公式：

 $f(l,v)=\int_{h\in\Omega}f_{\mu}(l,v,h)G_{2}(l,v,h)D(h)\frac{(h\cdot l)^{+}}{|n\cdot l|}\frac{(h\cdot v)^{+}}{|n\cdot v|}dh$

我们需要推导出来的式子为：

 $f_{spec}(l,v)=\frac{F(h,l)G_{2}(l,v,h)D(h)}{4|n\cdot l||n\cdot v|}$

所以我们的 **推导目标 1** 为：

，并且搞掉积分  $f_{\mu}(l,v,h)=\frac{F(h,l)}{4(h\cdot l)(h\cdot v)}，并且搞掉积分$

下面正式开始推导。

前面提到，只有当微平面法线 $m$ 等于入射光向量 $l$ 与观察向量的半程向量 $h=\frac{l+v}{||l+v||}$ 时，微表面 brdf 项 $f_{\mu}(l,v,m)\neq 0$ 。将这个描述用数学方程表达出来：

 $f_{\mu}(l,v,m)=\rho k\delta_{m}(h,m)$

其中， $\delta_{m}(h,m)$ 为称为狄拉克 (dirac delta) 的概念函数，它在除了零以外的点函数值都等于零，而其在整个定义域上的积分等于 1。由于我们要把这个函数放在积分里，所以这里这样理解：当 $h=m$ 时， $\delta_{m}(h,m)$ 对积分的贡献为 1，否则为 0。另外一个重要的性质是， $\delta_{m}(h,m)$ 消除积分，并且设置 $m=h$ 。 $\rho$ 是表面对入射光线的反射率 (剧透一下即菲涅尔项)

k 是我们要找到的，将这个函数进行归一化的因子。这里称它为 **推导目标 2**。

将这个带狄拉克的微表面 brdf 方程带入最开始提到的 BRDF 归一化方程：

 $\begin{align} \int_{\Omega}f_{\mu}(l,v,m)(v\cdot m)dv&=1\\ \int_{\Omega}\rho k\delta_{m}(h,m)(v\cdot m)dv&=1 \end{align}$

由于狄拉克函数 $\delta_{m}(h,m)$ 是关于微平面法线 $m$ 的函数，所以我们应该将上式的积分项改为 $dm$ ，即：

 $\int_{\Omega}\rho k\delta_{m}(h,m)(v\cdot m)\frac{dv}{dm}dm=1$

所以我们的 **推导目标 3**，最关键的目标，就是找到 $\frac{dv}{dm}$ 是什么，将它用向量表示出来。

由于狄拉克函数 $\delta_m(h,m)$ 将积分的贡献集中到了 $m=h$ 的评估上，所以我们令 $dm=dh$ 。规定所有向量都定义在单位半球面上，所以向量的微元可以认为是一小段立体角。我们在这个单位球面上找到 $dv$ 和 $dm=dh$ 。

![[7af040bd34482bad06189d8132a80d87_MD5.jpg]]

为了建立 $dm$ 和 $dv$ 的联系 (推导目标 2)，我们将 $dv$ 平移到 $dm$ 延长线上，也就是与 $l+v$ 向量相交，如下图绿线。

![[aded02f52b2122c96aed4a95a839fa07_MD5.jpg]]

接下来，我们需要将 $dv$ 投影到 $l+v$ 所定义的半球面上，使得它与 $dm$ 呈缩放关系。即把上图绿线投影为红线。

 $dv\bot=(h\cdot v)dv$

然后，我们需要将 $dv\bot$ 缩放到单位半球面，缩放因子为单位球面和 $l+v$ 定义球面的立体角比：

 $Scaler=\frac{4\pi\cdot 1^{2}}{4\pi\cdot |l+v|^{2}}=\frac{1}{|l+v|^{2}}$

![[f41738cdc3216db8aca6220fdb17b6ab_MD5.jpg]]

最后，将 $dv\bot$ 进行缩放，得到 $dm$ 。

 $dm=Scaler~dv\bot=\frac{h\cdot v}{|l+v|^{2}}dv$

总结一下我们上述的推导过程：

![[084564ee023db080d1fa1b75d6a90441_MD5.jpg]]

然后，我们对 $|l+v|$ 作进一步展开，**注意向量都是单位向量**：

 $\begin{align} |l+v|&=h\cdot (l+v)\\ &=h\cdot l+h\cdot v\\ &=2h\cdot v \end{align}$

把 $|l+v|=2h\cdot v$ 带入上面推导的 $dm$ 中。

 $\begin{align} dm&=\frac{h\cdot v}{|l+v|^{2}}dv\\ &=\frac{h\cdot v}{4(h\cdot v)^{2}}dv \end{align}$

也就是说，

 $\frac{dv}{dm}=4h\cdot v$

(终于得到分母那个 4 了！)

至此，我们的推导目标 3 完成。

将 $\frac{dv}{dm}=4h\cdot v$ 带入 BRDF 归一化方程：

 $\begin{align} \int_{\Omega}f_{\mu}(l,v,m)(v\cdot m)dv&=1\\ \int_{\Omega}\rho k\delta_{m}(h,m)(v\cdot m)\frac{dv}{dm}dm&=1\\ \int_{\Omega}\rho k\delta_{m}(h,m)(v\cdot m)(4h\cdot v)dm&=1 \end{align}$

由狄拉克函数积分方程：

 $\int\delta(x)dx=1$

且

 $\begin{align} m&=h\\ h\cdot v&=h\cdot l \end{align}$

所以，

 $k=\frac{1}{4(h\cdot l)(h\cdot v)}$

至此，我们的推理目标 2 完成。

把 $k$ 带入微 brdf 定义式：

 $\begin{align} f_{\mu}(l,v,m)&=\rho k\delta_{m}(h,m)\\ &=\rho\frac{\delta_{m}(m,h)}{4(h\cdot l)(h\cdot v)} \end{align}$

其中， $\rho$ 是表面对入射光的反射率，在镜面反射中，它就是菲涅尔项 $F(l,m)$ 。

即

 $f_{\mu}=F(l,m)\frac{\delta_{m}(m,h)}{4(h\cdot l)(h\cdot v)}$

把 $f_{\mu}$ 带入宏观 BRDF 方程：

 $\begin{align} f(l,v)=\int_{h\in\Omega}f_{\mu}(l,v,h)G_{2}(l,v,h)D(h)\frac{(h\cdot l)^{+}}{|n\cdot l|}\frac{(h\cdot v)^{+}}{|n\cdot v|}dh \end{align}$

得到

 $f(l,v)=\int_{h\in{\Omega}}\frac{F(l,h)D(h)G_{2}(l,v,h)}{4|n\cdot l||n\cdot v|}\delta_{m}(m,h)$

至此，推导目标 1 完成。

由狄拉克函数的性质， $\delta_{m}(m,h)$ 将积分贡献集中设置为 $m=h$ ，并且消除积分，最终得到：

 $f_{spec}(l,v)=\frac{F(h,l)G_{2}(l,v,h)D(h)}{4|n\cdot l||n\cdot v|}$

推导完毕。

描述出来可能有点繁琐，但核心还是变量替换，在 [Walter. 的 Paper: Microfacet Models for Refraction through Rough Surfaces](https://jcgt.org/published/0003/02/03/paper.pdf) 中，用的是雅可比式。在 [Eric Heitz 的 Paper: Understanding the Masking-Shadowing Function in Microfacet-Based BRDFs](https://jcgt.org/published/0003/02/03/paper.pdf) 中，用的球面度坐标。

放一下分别解释这两篇推导的文章：

[孙小磊：基于物理的渲染：微平面理论 (Cook-Torrance BRDF 推导)](https://zhuanlan.zhihu.com/p/152226698) [赵航：Cook-Torrance 的光照模型分母公式推导](https://zhuanlan.zhihu.com/p/34417784)

pbrt 中也解释了第二篇 paper 的推导： [Physically Based Rendering 3ed: Microfacet Models](https://www.pbr-book.org/3ed-2018/Reflection_Models/Microfacet_Models)

还是觉得上面这篇 GDC 的推导更人话一点，如果我没解释清楚可以看看 [GDC 的视频报告](https://www.gdcvault.com/play/1033723/PBR-Diffuse-Lighting-for-GGX)。

## 参考

《Real-time Rendering 4th》

[基于物理渲染 (PBR) 白皮书](https://github.com/QianMo/PBR-White-Paper/tree/master)

[《Games202》](https://www.bilibili.com/video/BV1YK4y1T7yY)

[GDC2017: PBR Diffuse Lighting for GGX+Smith Microsurfaces](https://ubm-twvideo01.s3.amazonaws.com/o1/vault/gdc2017/Presentations/Hammon_Earl_PBR_Diffuse_Lighting.pdf)

[Paper: Microfacet Models for Refraction through Rough Surfaces](https://jcgt.org/published/0003/02/03/paper.pdf)

[孙小磊：基于物理的渲染：微平面理论 (Cook-Torrance BRDF 推导)](https://zhuanlan.zhihu.com/p/152226698)

[Paper: Understanding the Masking-Shadowing Function in Microfacet-Based BRDFs](https://jcgt.org/published/0003/02/03/paper.pdf)

[赵航：Cook-Torrance 的光照模型分母公式推导](https://zhuanlan.zhihu.com/p/34417784)

[Physically Based Rendering 3ed: Microfacet Models](https://www.pbr-book.org/3ed-2018/Reflection_Models/Microfacet_Models)