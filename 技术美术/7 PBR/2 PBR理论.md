
---
title: 2 PBR理论
aliases: []
tags: []
create_time: 2023-07-10 12:29
uid: 202307101229
banner: "![[Pasted image 20230710142347.png]]"
---

**PBR 是指使用基于物理原理和微表面理论建模的着色/光照模型，以及使用从现实中测量的表面参数来准确表示真实世界材质的渲染理念。**

**三大组成部分：**
1. **基于物理的材质**（Material）
2. 基于物理的光照（Lighting）
3. 基于物理的摄像机（Camera）
![[0aacbee6a7db7477fc451dfab16366a9_MD5.jpg]]
> SIGGRAPH 2014 《Moving Frostbite to PBR》

物理渲染（Physical Rendering）是指跟真实世界完全一致的计算机渲染效果。基于现阶段的知识水平和硬件水平，还不能渲染跟真实世界完全一致的效果，只能一定程序上模拟接近真实世界的渲染画面，故而叫**基于物理的渲染**（**Physically Based Rendering**），而非**物理渲染**（**Physical Rendering**）。

# 2 PBR 基础基础理论和推导
满足以下条件的光照模型才能称之为 PBR 光照模型（基于物理的材质三大条件）：
*   基于微表面理论的表面模型（Be based on the microfacet surface model）。
*   能量守恒（Be energy conserving）。
*   使用基于物理的双向反射分布函数 BRDF（Use a physically based BRDF）。

## 微表面理论
**微表面理论**是将物体表面建模成无数微观尺度上有随机朝向的**理想镜面反射**的小平面（microfacet）的理论。

> [!NOTE] 理想镜面反射
> 理想镜面反射即严格遵循反射定律，反射角等于入射角

光在与非光学平坦表面（Non-Optically-Flat Surfaces）的交互时，非光学平坦表面表现得像一个微小的光学平面表面的大集合。表面上的每个点都会以略微不同的方向对入射光反射，而最终的表面外观是许多具有不同表面取向的点的聚合结果。


![[Pasted image 20230703155510.png]]
>微表面理论：在微观尺度上，表面越粗糙，反射越分散。表面越光滑，反射越集中，就会有更明显的高光。

从微观角度来说，没有任何表面是完全光滑的。由于这些微表面已经微小到无法逐像素地继续对其进行细分，因此我们只有假设一个 Roughness 参数，然后用**统计学方法**来概略的估算微表面的粗糙程度。

我们可以基于一个平面的粗糙度来计算出某个向量的方向与微表面平均取向方向一致的概率。这个向量便是位于光线向量 $l$ 和视线向量 $v$ 之间的中间向量，被称为**半角向量 (Halfway Vector)**。  

![[1679148476577.png##pic_center]]
$$h = \frac{l + v}{\|l + v\|}$$
## 能量守恒
**能量守恒** ：出射光线的能量永远不能大于入射光线的能量。

当光线折射进入内部的时候会与物体的微小粒子不断发生碰撞并散射到随机方向，同时在碰撞的过程中一部分光线的能量会被吸收转换为热能，有些光线在多次碰撞之后能量消耗殆尽，则表示该光线完全被物体吸收。
还有一部分折射到物体内部的光线会因为散射方向的随机性重新离开表面，而这部分光线就形成了漫反射。

![[Pasted image 20230703155647.png]]

通常情况下，PBR 会简化折射光，将平面上所有折射光都视为被完全吸收而不会散开。而有一些被称为次表面散射 (Subsurface Scattering) 技术的着色器技术会计算折射光散开后的模拟，它们可以显著提升一些材质（如皮肤、大理石或蜡质）的视觉效果，不过性能也会随着下降。

金属 (Metallic) 材质会立即吸收所有折射光，故而金属只有镜面反射，而没有折射光引起的漫反射。

回到能量守恒话题。被表面反射出去的光无法再被材质吸收。故而，**进入材质内部的折射光就是入射光减去反射光后余下的能量。**

![[Pasted image 20230703155640.png]]
>基于微表面理论，我们可以观察到随着粗糙度的上升镜面反射区域的面积会增加。
>基于能量守恒，我们可以观察到镜面反射区域的平均亮度则会下降。

## 渲染方程和反射方程
如何实现能量守恒➡渲染方程
![|450](0f3c00735859275a4fa201b4e4561037.png)
**渲染方程** (Render Equation) 是用来模拟光的视觉效果最好的模型：
$$L_o(p,\omega_o) = L_{e}(p,\omega_{o})+\int\limits_{\Omega} f_r(p,\omega_i,\omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$
- $L_{e}(p,\omega_{o})$：$p 点的自发光辐射率$


在实时渲染中，我们常用的**反射方程(The Reflectance Equation)**，则是渲染方程的简化版本，或者说是一个特例：

$$L_o(p,\omega_o) = \int\limits_{\Omega} f_r(p,\omega_i,\omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$
- $p$：点
- $n$：$p$ 点法线
- $w_i,w_o$：无限小的入射光（光源方向）和出射光（观察方向）的立体角，可以看作方向向量。方向由 $p$ 点指向光源或观察者眼睛
- $(n\cdot w_i)$：入射光与法线的点乘，用来衡量入射光与平面法线夹角 $\cos \theta$ 对能量衰减的影响
- $f_r(p,\omega_i,\omega_o)$： BxDF（通常为 BRDF）。描述了入射光反射后在各个方向如何分布
- $L_i(p,\omega_i)$ ：入射光辐射率
- $L_o(p,\omega_o)=\int\limits_{\Omega} ... d\omega_i$：对所有光源方向的半球积分，即从各个方向 $\omega_i$ 射入半球 $\Omega$ 并打中点 $p$ 的入射光，经过反射函数 $f_r$ 进入观察者眼睛的所有反射光 $L_o$ 的辐射率之和。因为计算了所有光源方向的单位立体角，所以**总辐射率=辐照度，即我们最终得到了 $p$ 点的辐照度。**

**反射方程计算了点 $p$ 在所有视线方向 $\omega_0$ 上被反射出来的辐射率 $L_o(p,\omega_o)$ 的总和。换言之：$L_0$ 计算的是在 $\omega_o$ 方向的眼睛观察到的 $p$ 点的辐照度。**

反射方程里面使用的辐照度，必须要包含所有以 $p$ 点为中心的**半球** $\Omega$ 内的入射光，而不单单只是某一个方向的入射光。这个半球指的是围绕面法线 $n$ 的那一个半球：  

![](1679148476772.png)

> [!question] 为什么只计算半球而不计算整个球体呢？
> 因为另外一边的半球因与视线方向相反，不能被观察，也就是辐射通量贡献量为 0，所以被忽略。

入射光辐射度可以由光源处获得，此外还可以利用一个环境贴图来测算所有入射方向上的辐射度。

至此，反射方程中，只剩下 $f_r$ 项未描述。$f_r$ 通常是**双向反射分布函数** (Bidirectional Reflectance DistributionFunction, BRDF)，**它的作用是基于表面材质属性来对入射辐射度进行缩放或者加权。** 后文将对其进行推导。


> [!quote] 
> 积分计算面积的方法，有**解析 (analytically)** 和**渐近 (numerically)** 两种方法。目前尚没有可以满足渲染计算的解析法，所以只能选择离散渐近法来解决这个积分问题。
> 
> 具体做法是在半球 $\Omega$ 按一定的步长将反射方程离散地求解，然后再按照步长大小将所得到的结果平均化，这种方法被称为**黎曼和 (Riemann sum)**。下面是实现的伪代码：
> 
> ```cs
> int steps = 100; // 分段计算的数量，数量越多，计算结果越准确。
> float dW  = 1.0f / steps;
> vec3 P    = ...;
> vec3 Wo   = ...;
> vec3 N    = ...;
> float sum = 0.0f;
> for(int i = 0; i < steps; ++i) 
> {
>     vec3 Wi = getNextIncomingLightDir(i);
>     sum += Fr(P, Wi, Wo) * L(P, Wi) * dot(N, Wi) * dW;
> }
> ```
> 
> `dW` 的值越小结果越接近正确的积分函数的面积或者说体积，衡量离散步长的 `dW` 可以看作反射方程中的 $d\omega_i$。积分计算中我们用到的 $d\omega_i$ 是线性连续的符号，跟代码中的 `dW` 并没有直接关系，但是这种方式有助于我们理解，而且这种离散渐近的计算方法总是可以得到一个很接近正确结果的值。值得一提的是，通过增加步骤数 `steps` 可以提高黎曼和的准确性，但计算量也会增大。

### 双向反射分布函数（BRDF）
#### 数学建模
**双向反射分布函数**（Bidirectional Reflectance Distribution Function，BRDF）是一个使用入射光方向 $\omega_i$ 作为输入参数的函数，输出参数为出射光 $\omega_o$，表面法线为 $n$，参数 $a$ 表示的是微表面的粗糙度。

为了进一步建模，我们只考虑光线的 **局部反射 (local reflection)** 情况，即光线击中表面，然后从交点向外反射 包括表面的镜面反射和次表面散射）出来。**但在局部反射中，可以把它们统一看作从宏观表面交点反射出来的光线。**
 
局部反射由 BRDF 量化，表示为 $f_r(\omega_i,\omega_o)$。BRDF 被定义在均匀表面 (uniform surfaces)，这意味着任意点的 BRDF 相同。并且假设给定波长的入射光以相同的波长反射。

**光线沿 $\omega_i$ 打到表面上某一面积微元上后，光线的辐照度 $dE (\omega_i)$ 会在交点处沿不同方向辐射出辐射率 $dL_r(x,\omega_r)$。**
![[3e8a79f380ab67ab4c698c65c66b1fd2_MD5.jpg]]

反射光线的分布受到宏观表面的微观几何影响。当微观尺度越粗糙，反射 glossy 越分散，而微观尺度越平滑，反射 glossy 越集中。
- @ **因此，BRDF 就是描述从不同方向入射后，反射光线的分布情况。具体来说，BRDF 为朝某个方向发出反射光辐射率 radiance 与入射光辐射度 irrandiance 的比值**。
$$BRDF=\frac{反射光辐射率}{入射光辐射度}$$
用数学式表达就是

$$f_{r}(w_{i}, w_{r})=\frac{dL_{r}(w_{r})}{dE_{i}(w_{i})}=\frac{dL_{r}(w_{r})}{L_{i}(w_{i}cos\theta_{i}d\omega_{i})}~~[\frac{1}{sr}]$$

![[9fada69e566c10e4ac9847dd065da360_MD5.jpg]]
**BRDF 材质 有三个特性：**
1.  **可逆性**，即交换 BRDF 的两个输入向量，BRDF 的值不变。$f_r(\omega_i,\omega_o)=f_r(\omega_o,\omega_i)$
2. **能量守恒**。比如反射光能量总和永远不应该超过入射光。技术上来说，Blinn-Phong 光照模型跟 BRDF 一样使用了 $\omega_i$ 跟 $\omega_o$ 作为输入参数，但是没有像基于物理的渲染这样严格地遵守能量守恒定律。
3. **各向同性和各向异性**

#### Cook-Torrance BRDF

BRDF 有好几种模拟表面光照的算法，然而，基本上所有的**实时**渲染管线使用的都是 **Cook-Torrance BRDF**。

Cook-Torrance BRDF 分为漫反射和高光反射两个部分：

$$f_r = k_d f_{lambert} + k_s f_{cook-torrance}$$

- $k_d$ ：漫反射比例
- $k_s$ ：高光反射比例
- $f_{lambert}$ ：漫反射部分，这部分叫做兰伯特漫反射（Lambertian Diffuse）。它类似于我们之前的漫反射着色，是一个恒定的算式：

$$f_{lambert} = \frac{c}{\pi}$$

其中 $c$ 代表的是反射率 Albedo 或表面颜色，类似漫反射表面纹理。**除以 $\pi$ 是为了规格化漫反射光**，为后期的 BRDF 积分做准备。[[[为什么 BRDF 的漫反射项要除以π？]]]

> [!quote] 
> 此处的兰伯特漫反射跟以前用的漫反射之间的关系：以前的漫反射是用表面的漫反射颜色乘以法线与面法线的点积，这个点积依然存在，只不过是被移到了 BRDF 外面，写作 $n \cdot \omega_i$，放在反射方程 $L_o$ 靠后的位置。

- BRDF 的高光（镜面）反射部分更复杂：
$$f_{cook-torrance} = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

Cook-Torrance 镜面反射 BRDF 由 3 个函数（$D$，$F$，$G$）和一个标准化因子构成。$D$，$F$，$G$ 符号各自近似模拟了特定部分的表面反射属性：

*   **$D$ (Normal Distribution Function，NDF)**：法线分布函数，描述的是微表面的法线方向与半角向量对齐的概率，如果对齐那么认为该反射光可以被看到，否则没有。这是用来估算微表面的主要函数。
*   **$F$ (Fresnel equation)**：菲涅尔方程，描述的是在不同的表面角下表面反射的光线所占的比率。
*   **$G$ (Geometry function)**：几何函数，描述了微表面自成阴影的属性。当一个平面相对比较粗糙的时候，平面表面上的微表面有可能挡住其他的微表面从而减少表面所反射的光线。

以上的每一种函数都是用来估算相应的物理参数的，而且你会发现用来实现相应物理机制的每种函数都有不止一种形式。它们有的非常真实，有的则性能高效。你可以按照自己的需求任意选择自己想要的函数的实现方法。

Epic Games 公司的 Brian Karis 对于这些函数的多种近似实现方式进行了大量的研究。这里将采用 Epic Games 在 Unreal Engine 4 中所使用的函数，其中 $D$ 使用 Trowbridge-Reitz GGX，$F$ 使用 Fresnel-Schlick 近似法 (Approximation)，而 $G$ 使用 Smith's Schlick-GGX。

#### D：GGX/TR
对于微表面模型的一个重要性质即每个微平面都有自己的微平面法线 $m$ 。微平面法线的分布被称为表面的 **法线分布函数 (NDF, normal distribution function)** $D(m)$ 。

![[Pasted image 20221211101035.png]]
 >如果一个微表面法线分布集中我们认为他是 glossy 材质，如果分布分散则认为是漫反射材质
 
 
法线分布函数，从统计学上近似的表示了与某些（如中间向量 $h$）向量取向一致（即微平面法线 $n$ 与半角项链半角向量 $h$ 重合）的微表面在微观几何中的**密度**。
>只有当微平面的微平面法线 $n$ 是 $l$ 和 $v$ 的 **半程向量 (half vector)** $h=\frac{l+v}{||l+v||}$ 时，这个微平面才会将能量反射进眼睛，否则这个微平面的能量贡献为 0。

> [!quote]  数学定义
>
>  $D(m)=\int_{\cal M}\delta_{m}(m)\,d p_{m} ~~~~[\frac{m^{2}}{sr}]$
> 微表面法线分布 $D(m)$ 描述了微表面上表面法线 $m$ 的统计分布。给定以 $m$ 为中心的无穷小立体角 $dω$ 和无穷小的宏观表面积 $dS$，$D(m)dωdS$ 是相应微表面部分的总面积，其法线位于指定的立体角内。因此 $D$ 是单位为 1/sr 的密度函数。
>
![[24a7fb676f8588ade097e727c0b28bfb_MD5.jpg]]
>
>如上图微表面侧视图所示，则 NDF 函数 $D(m)$ 服从等式： $A=D(m)d\omega dS$ 其中， $A$ 为所有微平面法线为 $m$ 的微表面面积 (上图红线面积)， $\omega$ 为微平面法线 $m$ 的立体角， $dS$ 为无穷小的保证为 flat 的宏观表面面积微元，但它大于微表面面积微元。
>
>
通过这个等式，我们可以得出 $D(m)$ 的物理含义，即**每单位面积，每单位立体角，所有法向为 m 的微平面的面积**。
>
我们将微平面总面积规定为 1，**那么 $\frac{D(m)}{1}=D(m)$ 就是我们要的结果, 即密度！**

目前有很多种 NDF 都可以从统计学上来估算微表面的总体取向度，只要给定一些粗糙度的参数以及一个我们马上将会要用到的参数 Trowbridge-Reitz GGX（GGXTR）：

$$NDF_{GGX TR}(n, h, \alpha) = \frac{\alpha^2}{\pi((n \cdot h)^2 (\alpha^2 - 1) + 1)^2}$$

-  $h$ ：半角向量
 - $\alpha$ ：粗糙度
 - $n$ ：法线。


![](d9b94cd41cd6cea5cfe6c13c93784b69.png)
 >GGX 有更好的高光长尾

使用不同的粗糙度作为参数，可以得到下面的效果：  

m
![](1679148476814.png)
>当粗糙度很低（表面很光滑）时，与中间向量 $h$ 取向一致的微表面会高度集中在一个很小的半径范围内。由于这种集中性，NDF 最终会生成一个非常明亮的斑点。但是当表面比较粗糙的时候，微表面的取向方向会更加的随机，与向量 $h$ 取向一致的微表面分布在一个大得多的半径范围内，但是较低的集中性也会让最终效果显得更加灰暗。

Trowbridge-Reitz GGX 的 NDF 实现代码：

```c
float DistributionGGX(vec3 N, vec3 H, float a) {
    float a2     = a*a;
    float NdotH  = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;
	
    float nom    = a2;
    float denom  = (NdotH2 * (a2 - 1.0) + 1.0);
    denom        = PI * denom * denom;
	
    return nom / denom;
}
```

#### F：Fresnel-Schlick
**菲涅尔效应 (fresnel effect)** 指的是反射随着 **掠射角 (glancing angle, 入射光与表面的夹角)** 的增大而增强。
![[Pasted image 20230709222844.png|354]]
当光线碰撞到一个表面的时候，**菲涅尔方程会根据观察角度告诉我们被反射的光线所占的百分比（即高光反射比例）。根据这个比例和能量守恒定律我们可以直接知道剩余的能量就是会被折射的能量。**

当我们垂直观察每个表面或者材质时都有一个基础反射率，当我们以任意一个角度观察表面时所有的反射现象都会变得更明显（反射率高于基础反射率）。你可以从你身边的任意一件物体上观察到这个现象，当你以 90 度角观察你的桌子你会法线反射现象将会变得更加的明显，理论上以完美的 90 度观察任意材质的表面都应该会出现全反射现象（所有物体、材质都有菲涅尔现象）。

![[Pasted image 20230703160302.png|450]]![[Pasted image 20230703160305.png|253]]
菲涅尔（Fresnel）方程很复杂，计算量很大，实时渲染中广泛采用 Fresnel-Schlick 近似式，因为计算成本低廉，而且精度足够：

$$F_{Schlick}(h, v, F_0) = F_0 + (1 - F_0) ( 1 - (h \cdot v))^5$$
$$
F_0=\left(\frac{n_1-n_2}{n_1+n_2}\right)^2
$$
- $h$：半角向量，疑问：这一项到底是半角向量还是光照方向？好像有多种形式
- $v$：观察方向
- $F_0$ ：表面基础反射率
- $n1,n2$ ：两种介质的真实折射率（即相对于真空的折射率）

![](1679148476836.png)
>菲涅尔方程运用在球面上的效果，观察方向越是接近**掠射角**（grazing angle，又叫切线角，与正视角相差 90 度），菲涅尔现象导致的反射就越强


菲涅尔方程中有几个微妙的地方，菲涅尔方程仅仅对电介质（绝缘体）或者说非金属表面有定义，而对于导体表面，使用它们的折射率（导体的折射率为负数）计算并不能得出正确的结果。这样我们就需要使用一种不同的菲涅尔方程来对导体表面进行计算，但是这样很不方便。所以我们**预先计算出导体的基础反射率，然后用 Schlick 方法来对其进行插值**估算。这样我们就能对金属和非金属材质使用同一个公式了。

下面是一些常见材质的基础反射率：

![](1679148476872.png)

这里可以观察到的一个有趣的现象，所有电介质材质表面的基础反射率都不会高于 0.17，这其实是例外而非普遍情况。导体材质表面的基础反射率起点更高一些并且（大多）在 0.5 和 1.0 之间变化。此外，对于导体或者金属表面而言基础反射率一般是带有色彩的，这也是为什么要用 RGB 三原色来表示的原因（法向入射的反射率可随波长不同而不同）。这种现象我们只能在金属表面观察的到。

金属表面这些和电介质表面相比所独有的特性引出了所谓的金属工作流的概念。也就是我们需要额外使用一个被称为金属度 (Metalness) 的参数来参与编写表面材质。金属度用来描述一个材质表面是金属还是非金属的。

通过预先计算物体的基础反射率的值，我们可以对两种类型的表面使用相同的Fresnel-Schlick近似，但是如果是金属表面的话就需要对基础反射率添加色彩。我们一般是按下面这个样子来实现的：

```c
vec3 F0 = vec3(0.04);
F0      = mix(F0, surfaceColor.rgb, metalness);
```

我们为大多数电介质表面定义了一个近似的基础反射率。$F_0$ 取最常见的电解质表面的平均值，这又是一个近似值。不过对于大多数电介质表面而言使用 0.04 作为基础反射率已经足够好了，而且可以在不需要输入额外表面参数的情况下得到物理可信的结果。**然后，基于金属表面特性，我们要么使用电介质的基础反射率要么就使用 $F_0$ 作来为表面颜色。因为金属表面会吸收所有折射光线而没有漫反射，所以我们可以直接使用表面颜色纹理来作为它们的基础反射率。**

Fresnel Schlick 近似可以用 GLSL 代码实现：

```c
vec3 fresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
```

其中 `cosTheta` 是表面法向量 $n$ 与观察方向 $v$ 的点乘的结果。

####  G：Schlick-GGX

几何函数从统计学上近似的求得了微表面间相互遮蔽（自遮挡现象）的比率，这种相互遮蔽会损耗光线的能量。(除了被吸收，还有被遮蔽带来的能量损耗）

![](1679148476912.png)


> [!quote] 数学定义
> 我们是使用的微表面理论，也就是说每个面单独计算互不干扰。但现实世界中，物体的表面的凹凸存在相互遮蔽的情况。主要受粗糙度影响。对于渲染来说，我们只关心可见的微表面。
> 
> 基于上述事实，我们可以提出另外一种统计微平面法线到视角垂平面的投影面积：**统计所有可见微平面法线到视角垂平面的投影面积**。如下图，我们只考虑可见的红线部分的投影贡献。
> ![[22be779d5b79650eaf1533fe67a753df_MD5.jpg]]
> 
> 我们可以通过定义 **遮蔽函数 (masking function)** $G_{1}(m,v)$ 来数学的表示这一点，**该函数给出法线为 $m$ 且沿视角 $v$ 可见的微平面比例。**
> 
>  $\int_{m\in\Theta}G_{1}(m,v)D(m)(v\cdot m)^{+}dm=v\cdot n$
> 
> 其中， $(v\cdot m)^{+}$ 表示钳位到 0，它表示不可见的背微平面不会被计算。 $G_{1}(m,v)D(m)$ 为 **可见法线分布 (distrubition of visible normals)**。
>

对于给定法线分布函数 D (m)，可以有无数个遮蔽函数 G (m)。这是因为 D (m) 并没有完全指定微表面，它只告诉了微表面法线的分布，但不知道它们的排列。

> [!note]  Smith G1 和 Smith Shadow-Masking G2
> 一个被广泛使用的 G1 遮蔽函数为 **The Smith G1** 函数，它最初是针对高斯正态分布推导出来的，后来推广到任意的 NDFs 上。
> 
>  $\begin{align} G_{1}(\mathbf{m},\mathbf{v})&={\frac{\chi^{+}(\mathbf{m}\cdot\mathbf{v})}{1+\Lambda(\mathbf{v})}},\\ \chi^{+}(x)&=\left\{\begin{matrix}1,~~where~x>0.\\0,~~where~x\leq 0.\end{matrix}\right. \end{align}$
> 
> 其中， $m$ 为微平面法线， $v$ 为观察向量， $\Lambda$ 函数视 NDF 而不同。
> 
> **正如上面讨论的那样，遮蔽函数 G1 只考虑了微表面对视线的遮挡，即 Masking。而还存在微表面对光线的遮挡，即 Shadowing。**
> 
> ![[8c71e0b51185e84c583ab7a1df499b63_MD5.jpg]]
> 
> **为了考虑 Masking 对可见法线的影响，提出了联合遮蔽 - 阴影函数 (joint masking-shadowing function) $G_{2}(l,v,m)$，也被称为几何函数 (geometry function) 。**
> 
> 实际应用中，常用 **The Smith Shadow-Masking G2** 函数，它将 Shadowing 和 Masking 分开考虑。由于光路的可逆性，我们可以认为两种情况是近似等效的。
> 
>  $$G_{2}(l,v,m)=G_{1}(v,m)G_{1}(l,m)$$
> 
> 它建立在 Shadowing 和 Masking 不相关的基础上，但实际上它们是相关的。**使用这个 G2 会导致 BRDFs 结果偏暗。**
> 

类似 NDF，几何函数也使用粗糙度作为输入参数。几何函数使用由 GGX 和 Schlick-Beckmann 组合而成的模拟函数 Schlick-GGX：

$$G_{SchlickGGX}(n, v, k) = \frac{n \cdot v} {(n \cdot v)(1 - k) + k }$$

这里的 $k$ 是使用粗糙度 $\alpha$ 计算而来的，用于直接光照和 IBL 光照 (间接光)的几何函数的参数：

$$\begin{eqnarray*} k_{direct} &=& \frac{(\alpha + 1)^2}{8} \\ k_{IBL} &=& \frac{\alpha^2}{2} \end{eqnarray*}$$

需要注意的是这里 $\alpha$ 的值取决于你的引擎怎么将粗糙度转化成 $\alpha$。

为了有效地模拟几何体，我们需要同时考虑两个视角，观察方向（几何遮挡）跟光源方向（几何阴影），我们可以用 **Smith G2 函数**将两部分放到一起：

$$G_2(n, v, l, k) = G_{1}(n, v, k) G_{1}(n, l, k)$$
- $n$：法线
 - $v$ ：观察方向
 - $G_{1}(n, v, k)$ ：观察方向的几何遮挡
 - $l$ ：光源方向
 - $G_{1}(n, l, k)$ 表示光源方向的几何阴影。使用 Smith 函数与 Schlick-GGX 作为 $G_{1}$ 可以得到如下所示不同粗糙度 R 的视觉效果：  

![](1679148477023.png)
>几何函数是一个值域为 $[0.0, 1.0]$ 的乘数，其中白色 (1.0) 表示没有微表面阴影，而黑色 (0.0) 则表示微表面彻底被遮蔽。

使用 GLSL 编写的几何函数代码如下：

```cs
float GeometrySchlickGGX(float NdotV, float k) {
    float nom   = NdotV;
    float denom = NdotV * (1.0 - k) + k;
	
    return nom / denom;
}
  
float GeometrySmith(vec3 N, vec3 V, vec3 L, float k) {
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx1 = GeometrySchlickGGX(NdotV, k); // 视线方向的几何遮挡
    float ggx2 = GeometrySchlickGGX(NdotL, k); // 光线方向的几何阴影
	
    return ggx1 * ggx2;
}
```

#### Cook-Torrance 反射方程

Cook-Torrance 反射方程中的每一个部分我们我们都用基于物理的 BRDF 替换，可以得到最终的反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

上面的方程并非完全数学意义上的正确。前面提到菲涅尔项 $F$ 代表光在表面的反射比率，它直接影响 $k_s$ 因子，意味着反射方程的镜面反射部分已经隐含了因子 $k_s$。因此，最终的 Cook-Torrance 反射方程如下（去掉了 $k_s$）：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + \frac{D(n, h, \alpha)F(h, \omega_o, F_0)G(n, \omega_o, \omega_i, k)}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$
- ? G 项是法线参数是 n 还是 h？闫老师说是 h？

- 对于分母中的点积，仅仅避免负值是不够的，也必须避免零值。通常通过在常规的 clamp 或绝对值操作之后添加非常小的正值来完成。

这个方程完整地定义了一个基于物理的渲染模型，也就是我们一般所说的基于物理的渲染（PBR）。
#### 能量补偿项

通过包含 G2 函数，Microfacet BRDF 能够考虑遮蔽 (masking) 和阴影 (shadowing)，但依然没有考虑微平面之间的互反射 (interreflection)，或多表面反射 (multiple surface bounce)。而缺少微平面互反射 (interreflection) 是业界主流 Microfacet BRDF 的共有的限制。如图，虽然在小球上没有出现任何掠射角的问题，但随着粗糙度的变大，渲染的结果越来越暗。即使认为最左边是抛光，最右边的是哑光，这个结果也是错误的。如果对小球材质进行白炉测试 ( $F(i,h)\equiv 1$ ， $uniform irrdiance = 1$ 的天光，检测材质反射能量是否未 1)，这种现象更为明显。
![[Pasted image 20230710202811.png]]
出现这种问题的原因是标准 Microfacet BRDF 模型虽然能量守恒 (即不会产生任何能量)，但它们也不能在高粗糙度时维持能量 (即能量损失)。这是**由于建模微平面模型时所做出的单散射假设，没有模拟微表面上的多次散射，即缺少微平面互反射 (interreflection)。单散射的在高粗糙度时会有较大的能量损失，从而显得过暗。

![[Pasted image 20230710202818.png]]

对此，在实时渲染中常用的处理方法是对原先的模型添加一个**能量补偿项**来补足损失的能量。**核心思想**是将反射光看作两种情况：当不被遮挡时，这些光会被看到；当反射光被微表面遮挡时，这些遮挡住的光将会进行后续的弹射，直到能被看到。

**【Kulla-Conty 近似】** 通过经验去补全多次反射丢失的能量，其实是创建了一个多次反射表面反射的附加 BRDF 波瓣，利用这个 BRDF 算出消失的能量作为能量补偿项。 
**预计算出一张图表示 $E_{avg}$ ，代入 $f_{ms}$ 中，进而求出消失的能量 $E_{ms}$ 。**
![[Pasted image 20230710203604.png|250]]

![[Pasted image 20230710204205.png|700]]

k 次间接反射的能量为 $F^{k}{avg}(1-E{avg})^{k}\cdot F_{avg}E_{avg}$ 将以上所有能量累加，得 $\frac{F_{avg}E_{avg}}{1-F_{avg}(1-E_{avg})}$ ，再与无色 $f_{ms}$ 相乘，即可得到有色的能量补偿项。

最后，考虑了能量补偿项的渲染方程如下：

 $L_{o}(p,\omega_{o})=\int_{\Omega^{+}}L_{i}(p,\omega_{i})(f_{r}(p,\omega_{i},\omega_{o})+f_{ms}(\omega_{i}.\omega_{o}))cos\theta_{i}d\omega_{i}$

增加颜色项后的结果如下所示。

![[Pasted image 20230710204422.png]]
### BxDF

目前计算机图形渲染领域，基于物理的渲染方式主要有：

*   **辐射度（Radiance）**：计算光源的镜面反射和漫反射占总的辐射能量的比例，从而算出颜色。在实时渲染领域，是最主流的渲染方式。BRDF 大多数都是基于此种方式，包括 Cook-Torrance。
    
*   **光线追踪（Ray Tracing）**：即光线追踪技术，它的做法是将摄像机的位置与渲染纹理的每个像素构造一条光线，从屏幕射出到虚拟世界，每遇到几何体就计算一次光照，同时损耗一定比例的能量，继续分拆成反射光线和折射光线，如此递归地计算，直到初始光线及其所有分拆的光线能量耗尽为止。
    
    ![[1679148482590.png|500]]
    
    由于这种方式开销非常大，特别是场景复杂度高的情况，所以常用于离线渲染，如影视制作、动漫制作、设计行业等。
    
    近年来，随着 NVIDIA 的 RTX 系列和 AMD 的 RX 系列显卡问世，它们的共同特点是硬件级别支持光线追踪，从而将高大上的光线追踪技术带入了实时渲染领域。
    
*   **路径追踪（Path Tracing）**：实际上路径追踪是光线追踪的一种改进方法。它与光线追踪不同的是，引入了蒙特卡洛方法，利用 BRDF 随机跟踪多条反射光线，随后根据这些光线的贡献计算该点的颜色值。
    这种方法更加真实（下图），但同时也更加耗时，通常用于离线渲染领域。
    ![[1679148482649.png|350]]

上章已经详细描述了基于辐射度的 Cook-Torrance 的 BRDF 模型的理论和实现。实际上，Cook-Torrance 模型在整个渲染体系中，只是冰山一角。下面是 BRDF 光照模型体系：

![[1679148482675.png]]

限于篇幅和本文主题，下面将介绍基于辐射度方式的 BxDF 光照模型。

**BxDF 一般而言是对 BRDF、BTDF、BSDF、BSSRDF 等几种双向分布函数的一个统一的表示。可细分为以下几类：**

*   **BRDF**（双向反射分布函数，Bidirectional Reflectance Distribution Function）：用于**非透明**材质的光照计算。**Cook-Torrance 就是 BRDF 的一种实现方式**。
*   **BTDF**（双向透射分布函数，Bidirectional Transmission Distribution Function）：用于**透明材质**的光照计算。折射光穿透介质进入另外一种介质时的光照计算模型，只对有透明度的介质适用。
*   **BSDF**（双向散射分布函数，Bidirectional Scattering Distribution Function）：实际上是 BRDF 和 BTDF 的综合体，简单地用公式表达：**BSDF = BRDF + BTDF**。
    
    ![[1679148482726.png|350]]
*   **SVBRDF**（空间变化双向反射分布函数，Spatially Varying Bidirectional Reflectance Distribution Function）：**将含有双参数的柯西分布替代常规高斯分布**引入微面元双向反射分布函数 (BRDF) 模型，同时考虑了目标自身辐射强度的方向依赖性，在此基础上推导了长波红外偏振的数学模型，并在合理范围内对模型做简化与修正使之适用于仿真渲染。
*   **BTF**（双向纹理函数，Bidirectional Texture Function）：主要用于**模拟非平坦表面**，参数跟 SVBRDF 一致。但是，BTF 包含了非局部的散射效果，比如阴影、遮挡、相互反射、次表面散射等。用 BTF 给表面的每个点建模的方法被成为 **Apparent BRDFs**（表面双向反射分布函数）。
*   **SSS**（次表面散射，也称 3S，Subsurface Scattering）：它是**模拟光进入半透明或者有一定透明深度的材质（皮肤、玉石、大理石、蜡烛等）后，在内部散射开来，然后又通过表面反射出来的光照模拟技术**。下面是用 SSS 模拟的玉石效果图：
    ![[1679148482775.png|400]]
    关于次表面散射方面的研究，比较好的是 Jensen 的文章《A Practical Model for Subsurface Light Transport》，该文提出了一个较为全面的 SSS 模型，将它建模成一个双向表面散射反射分布函数 (BSSRDF)。
*   **BSSRDF**（双向表面散射分布函数，Bidirectional Surface Scattering Reflectance Distribution Function）：它常用于**模拟透明材质**，目前是主流技术。它**和 BRDF 的不同之处在于，BSSRDF 可以再现光线透射材质的效果，还可以指定不同的光线入射位置和出射位置：**
    
    ![[1679148482799.png|600]]

**从上面可以看出，BxDF 的形式多种多样，但由于它们都是基于辐射度的光照模型，所以最终可以用以下公式抽象出来：**

$$L_o(p,\omega_o) = \int\limits_{\Omega} f_r(p,\omega_i,\omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

用更简洁的方式描述，入射光 $\omega_i$ 在 $p$ 点的颜色的计算公式：

$$\begin{eqnarray*} p点颜色 & = & 光源颜色 \times 材质颜色 \times 反射系数 \times 光照函数 \\ 光照函数 & = & f(n_{法线}, \omega_{光源方向}, v_{视点方向}) \end{eqnarray*}$$

值得一提的是，BRDF 最终的光照计算结果是几何函数和油墨算法（ink-selection）结合的结果。其中油墨算法描述了如何计算各颜色分量的反射率，可参看论文 [《A Multi-Ink Color-Separation Algorithm Maximizing Color Constancy》](https://pdfs.semanticscholar.org/9e56/8b13ea51ca3c669186624566f672eb547857.pdf)。

![[1679148483056.png|500]]

### Disney
## 其他 DFG

### 法线分布函数 D

对于镜面反射 BRDF，法线分布函数 D 的常见模型可总结如下：

*   Beckmann[1963]
*   Blinn-Phong[1977]
*   GGX [2007] / Trowbridge-Reitz[1975]
*   Generalized-Trowbridge-Reitz (GTR) [2012]
*   Anisotropic Beckmann[2012]
*   Anisotropic GGX [2015]

**业界主流的法线分布函数是 GGX**。
![[Pasted image 20230710195651.png]]
>红线 Backmann，绿线 GGX。右图，左边为 GGX，右边为Backmann
#### Beckmann

Beckmann 是一种定义在坡度空间上的**类高斯分布**模型，这个函数**可以描述不同粗糙程度的表面，不同粗糙程度的意思是 NDF 中 lobe 是集中在一个点上，还是分布得比较开**。它的表达式为

 $$D_{Beckmann}(h)=\frac{e^{-\frac{tan^{2}\theta_{h}}{\alpha^{2}}}}{\pi\alpha^{2}cos^{4}\theta_{h}}$$

 $h$ ：半程向量
 $\alpha$ ：粗糙系数，粗糙程度这个值越小，表面就越光滑
  $\theta_{h}=(\hat{n}\cdot \hat{m})$ ：半程向量与宏观法线的夹角
  高斯分布函数 $X\sim N(\mu,\sigma^{2})=\frac{1}{\sqrt{2\pi}\sigma}^{-\frac{(x-\mu)^{2}}{2\sigma^{2}}}$ 中， $\sigma$ 控制胖瘦程度，同样的，在 Beckmann 表达式中， $\alpha$ 控制胖瘦程度。

![[fb9c19a8c393df03c7975ec33c889e0e_MD5.jpg]]

之所以幂的分子上使用 $tan\theta_{h}$ ，而不直接使用 $\theta_{h}$ 是因为 Beckmann 定义在**坡度空间**上，需要满足高斯部分的定义域无限大的性质，保证函数无论何时都具有对应的非负值，并且避免微表面出现法线朝下的问题 (但无法避免反射光朝下)。

![[ff979cf2a41139cb4a5100491c7ea536_MD5.jpg]]
>如图，随着 $\theta$ 不断增大不断增大，红色的向量永远不会朝下


#### GGX

GGX 模型的表达式为

 $$D_{GGX}(h)=\frac{\alpha^{2}}{\pi(cos^{2}\theta_{h}(\alpha^{2}-1)+1)^{2}}$$

其中， $h$ 为微观半程向量； $\alpha$ 为粗糙系数，粗糙程度这个值越小，表面就越光滑； $\theta_{h}=(\hat{n}\cdot \hat{m})$ 是半程向量与宏观法线的夹角。 GGX 相对于 Beckmann 在工业界得到了更为广泛的应用，因为它具有更好的高光拖尾 (Long tail 性质，衰减更加柔和)。

![[b55bc08e7043ee7b754a3e565fbaf616_MD5.jpg]]

这会带来两个好处：
*   Beckmann 的高光会逐渐消失，而 GGX 的高光会减少而不会消失，这就意味着高光的周围我们看到一种光晕的现象。
*   GGX 除了高光部分，其余部分会像 Diffuse 的感觉。

![[Pasted image 20230710200154.png]]

#### GTR
Generalized Trowbridge-Reitz (GGX/TR 模型增强版)

GTR 是根据对 GGX 等分布的观察，提出的**广义法线分布函数**，其目标是允许更多地控制 NDF 的形状，特别是分布的尾部。它的表达式为：
 $$D_{GTR}(h)=\frac{c}{(1+cos^{2}\theta_{h}(\alpha^{2}-1))^{\gamma}}$$

其中， $h$ 为微观半程向量； $\alpha$ 为粗糙系数，粗糙程度这个值越小，表面就越光滑； $\theta_{h}=(\hat{n}\cdot \hat{m})$ 是半程向量与宏观法线的夹角。 $\gamma$ 参数用于控制尾部形状。当 $\gamma=2$ 时，GTR 等同于 GGX。随着 $\gamma$ 的值减小，分布的尾部变得更长。而随着 $\gamma$ 值的增加，分布的尾部变得更短。越来越接近Backmann

![[4a7b3f86070440b46ab2f2c113de242b_MD5.jpg]]

### 菲涅尔项 F

对于镜面反射 BRDF，菲涅尔项 F 的常见模型可以总结如下：

*   Cook-Torrance [1982]
*   Schlick [1994]
*   Gotanta [2014]

**业界方案一般都采用 Schlick 的 Fresnel 近似**，因为计算成本低廉，而且精度足够：

 $F_{S c h l i c k}({\mathbf{v},\mathbf{h}})=F_{0}+({\mathbf{l}}-F_{0}){\bigl(}{\textbf{l}}-(v\cdot h){\bigr)}^{5}$

### 几何函数 G

对于镜面反射 BRDF，几何函数 G 的常见模型可以总结如下：

*   Smith [1967]
*   Cook-Torrance [1982]
*   Neumann [1999]
*   Kelemen [2001]
*   Implicit [2013]

另外，Eric Heitz 在 [Heitz14] 中展示了 Smith 几何阴影函数是正确且更准确的 G 项，并将其拓展为 Smith 联合遮蔽阴影函数 (Smith Joint Masking-Shadowing Function)，该函数具有四种形式：

*   分离遮蔽阴影型 (Separable Masking and Shadowing)
*   高度相关掩蔽阴影型 (Height-Correlated Masking and Shadowing)
*   方向相关掩蔽阴影型 (Direction-Correlated Masking and Shadowing)
*   高度 - 方向相关掩蔽阴影型 (Height-Direction-Correlated Masking and Shadowing)

目前较为常用的是其中最为简单的形式，分离遮蔽阴影 (Separable Masking and Shadowing Function)。

该形式将几何项 G 分为两个独立的部分：光线方向 (light) 和视线方向 (view)，并对两者用相同的分布函数来描述。根据这种思想，结合法线分布函数 (NDF) 与 Smith 几何阴影函数，于是有了以下新的 Smith 几何项：

*   Smith-GGX
*   Smith-Beckmann
*   Smith-Schlick
*   Schlick-Beckmann
*   Schlick-GGX

其中 UE4 的方案是上面列举中的 **“Schlick-GGX”** ，即基于 Schlick 近似，将 k 映射为 k=a/2, 去匹配 GGX Smith 方程：

$\begin{align} k&=\frac{\alpha}{2}\\ \alpha&=(\frac{roughness+1}{2})^{2}\\ G_{1}(v)&=\frac{n\cdot v}{(n\cdot v)(1-k)+k}\\ G(l,v,h)&=G_{1}(l)G_{1}v \end{align}$

# PBR 扩展
## LTC 多边形光源渲染
Linearly Transformed Cosines：线性变换余弦分布

![[Pasted image 20230710210648.png]]

**多边形光源**下的渲染我们需要取光源上很多采样点，并且与 shaing point 连线, 如果不考虑连线是否与场景有交点则不考虑 shadow, 如果需要考虑 shadow 还需要做一下 shadow test。
但不论如何都需要采样, 那么不可避免地会使速度变慢。为了避免采样光源带来的开销问题，可以应用 LTC 技术来避免采样，满足实时渲染的要求。

![[Pasted image 20230710211040.png]]
>lobe：波瓣（函数图像如同花瓣）


简单来说就行，将反射方程中原本需要采样才能计算的 BRDF (原始球面分布)通过一个 M 矩阵变换为余弦分布（新球面分布）（注意余弦分布不是常见的 cos，而是一个球面分布函数），而 M 可以**预计算**，余弦分布可以**解析计算**积分，所以能快速计算 BRDF 积分而避免了采样，这种方法就是线性变换余弦 (Linearly Transformed Cosines, LTCs)。

代码实现: [[04 利用 LTC 实现实时多边形面积光]]

## **Disney's principle BRDF**

首先我们来讨论一下为什么还需要 Disney's principle BRDF:

因为微表面模型是由一些问题的:

**1. 微表面模型无法解释多层材质**
我们来举个例子, 我们有一个木头桌子, 我们知道木头是 diffuse 的, 在桌子的表面刷一层清漆。从而我们的桌子变成了多层材质: 清漆 + 木头。清漆是无色的, 由于清漆是平坦的, 因此在光线打入时, 一部分反射出去产生了高光现象。另一部分打入内部并打到桌子上以 diffuse 发散出去, 因此我们应该会看到高光和 diffuse. 这是微表面模型无法做到的, 因为**微表面模型无法解释多层材质。**
>清漆 ClearCoat，又名[凡立水](https://baike.baidu.com/item/%E5%87%A1%E7%AB%8B%E6%B0%B4/9978483)，是由[树脂](https://baike.baidu.com/item/%E6%A0%91%E8%84%82/281282)为主要[成膜物质](https://baike.baidu.com/item/%E6%88%90%E8%86%9C%E7%89%A9%E8%B4%A8)再加上溶剂组成的[涂料](https://baike.baidu.com/item/%E6%B6%82%E6%96%99/2503539)。由于涂料和涂膜都是透明的，因而也称透明涂料。涂在物体表面，干燥后形成光滑薄膜，显出物面原有的纹理。

**2. 微表面模型对艺术家来说并不好用**

我们知道 PBR, PBR, 都是基于物理的, 我们以金属反射率来说，反射率由 n 和 k 这两个参数定义。对于 artist 来说, 他们是不知道的怎么调。因此 PBR 材质对于艺术家来说不好用。
*   Disney's principle BRDF 诞生的首要目的就是为了让 artist 使用方便, 因此它并不要求在物理上完全正确.
*   但是在 RTR 中我们认为 Disney's principle BRDF 也算是 PBR 材质.

**Disney's principle BRDF 有几个重要的设计原则:**
1.  应该使用更直观的名词而不是使用物理名词参数, 比如使用平缓, 饱和度等
2.  让 brdf 框架不太复杂, 也就是让参数数量少一点
3.  最好有一个拖动条左边最小值, 右边最大值供艺术家们进行调整
4.  有时候为了特殊的效果允许将参数值超过范围, 也就是允许小于 0 或大于 1
5.  所有参数的组合应尽可能可靠和合理, 也就是不论如何调整参数最后的结果应该是正常的.

因此在这套设计原则下, artist 可以根据自己需要去定义自己想得到的 BRDF:
![[Pasted image 20230710213136.png]]

*   subsurface: 次表面反射, 为了在 BRDF 中给你一种比 diffuse 还要平的效果. 可以看出当 subsurface 为 1 时与 0 相比像是被压扁了一样.

![[2551fe83ada6e138c25626fe73749a96_MD5.png]]

*   metallic: 金属性, 顾名思义看起来像金属的程度.

![[bc4a549ff7fbdc5d159a481651c53d96_MD5.png]]

*   specular: 控制有多少镜面反射的内容, 0 为完全没有镜面反射内容, diffuse, 1 则表示全是镜面反射内容.

![[e4d933eae00a5ad3f4df0846b52821d3_MD5.png]]

*   specular tint: 镜面反射出的颜色无色 (为 0), 还是偏向于自己物体本身的颜色 (1).

![[671ef64bfd5f8d2c97526a43c4f22482_MD5.png]]

*   roughness: 粗糙度, 为 0 表示全是镜面反射, 为 1 表示没有镜面反射.

![[ce23c41e089a420eb52b7eda260f68c6_MD5.png]]

*   anisotropic: 各向异性程度, 可以理解为当为 1 的时候带来一种像是被刷过一样的效果.

![[e791eeb270d82ab760161a1fbede89fa_MD5.png]]

*   sheen: 可以理解为, 在物体表面法线方向上长了绒毛, 这让你在 grazing angle (外圈) 处看起来有一种雾化的感觉.

![[8b09eed3621623e4dbe668359e661671_MD5.png]]

*   sheen tint: 可以理解为绒毛造成的雾化效果颜色是无色, 还是偏向物体本身的颜色.

![[0f0858a058d023ab34408103752c9795_MD5.png]]

*   clearcoat: 可以理解为透明层的明显程度, 0 时表示没有透明层, 1 则表示有一层透明层 (涂了一层清漆).

![[2cee53fe70af566ce4635a27d0146bb0_MD5.png]]

*   clearcoat gloss: 透明层的光泽层度, 为 0 就像被磨砂了一样, 为 1 则表示完全光滑.

![[21607352efb99fcaaa82648a04312b72_MD5.png]]


**Disney's principle BRDF 的优点:**

1. 容易理解和使用各参数 (属性)
2. 参数的混合组合使得可以在一个模型上显示出很多不同的材质.
3. 开源

**Disney's principle BRDF 的缺点:**
1. 并不是完全基于物理的
2. 巨大的参数空间使得拥有强大的表示能力, 但是会造成冗余现象.


# 3 PBR 的光照实现

3.1 章节阐述了 Cook-Torrance 反射方程的理论和公式意义。这节将探讨如何将前面讲到的理论转化成一个基于直接光照的渲染器：比如点光源，方向光和聚光灯。

### **3.2.1 辐照度计算**

3.1 章节解释了 Cook-Torrance 反射方程的大部分含义，但有一点未提及：具体要怎么处理场景中的辐照度（Irradiance，也就是辐射的总辐射率 $L$）？在计算机领域，场景的辐射率 $L$ 度量的是来自光源光线的辐射通量 $\phi$ 穿过指定的立体角 $\omega$，在这里我们假设立体角 $\omega$ 无限小，小到辐射度衡量的是光源射出的一束经过指定方向向量的光线的通量。

有了这个假设，我们又要怎么将之融合到之前教程讲的光照计算里去呢？想象我们有一个辐射通量以 RGB 表示为（23.47, 21.31, 20.79）的点光源，这个光源的辐射强度等于辐射通量除以所有出射方向。当为平面上某个特定的点 $p$ 着色的时候，所有可能的入射光方向都会经过半球 $\Omega$，但只有一个入射方向 $\omega_i$ 是直接来自点光源的，又因为我们的场景中只包含有一个光源，且这个光源只是一个点，所以 $p$ 点所有其它的入射光方向的辐射率都应该是 0.  

![](1679148477087.png)

如果我们暂时不考虑点光源的距离衰减问题，且无论光源放在什么地方入射光线的辐射率都一样大（忽略入射光角度 $\cos \theta$ 对辐射度的影响），又因为点光源朝各个方向的辐射强度都是一样的，那么有效的辐射强度就跟辐射通量完全一样：恒定值（23.47, 21.31, 20.79）。

然而，辐射率需要使用位置 $p$ 作为输入参数，因为现实中的灯光根据点 $p$ 和光源之间距离的不同，辐射强度多少都会有一定的衰减。另外，从原始的辐射方程中我们可以发现，面法线 $n$ 于入射光方向向量 $\omega_i$ 的点积也会影响结果。

用更精炼的话来描述：在点光源直接光照的情况里，辐射率函数 $L$ 计算的是灯光颜色，经过到 $p$ 点距离的衰减之后，再经过 $n \cdot \omega_i$ 缩放。能击中点 $p$ 的光线方向 $\omega_i$ 就是从 $p$ 点看向光源的方向。把这些写成代码：

```
vec3  lightColor  = vec3(23.47, 21.31, 20.79);
vec3  wi          = normalize(lightPos - fragPos);
float cosTheta    = max(dot(N, Wi), 0.0);
// 计算光源在点fragPos的衰减系数
float attenuation = calculateAttenuation(fragPos, lightPos); 
// 英文原版的radiance类型有误，将它改成了vec3
vec3 radiance  = lightColor * (attenuation * cosTheta);
```

你应该非常非常熟悉这段代码：这就是以前我们计算漫反射光的算法！**在只有单光源直接光照的情况下，辐射率的计算方法跟我们以前的光照算法是类似的。**

要注意我们这里假设点光源无限小，只是空间中的一个点。如果我们使用有体积的光源模型，那么就有很多的入射光方向的辐射率是非 0 的。

对那些基于点的其他类型光源我们可以用类似的方法计算辐射率，比如平行光源的入射角的恒定的且没有衰减因子，聚光灯没有一个固定的辐射强度，而是围绕一个正前方向量来进行缩放的。

这也将我们带回了在表面半球 $\Omega$ 的积分 $\int$。我们知道，多个单一位置的光源对同一个表面的同一个点进行光照着色并不需要用到积分，我们可以直接拿出这些数目已知的光源来，分别计算这些光源的辐照度后再加到一起，毕竟每个光源只有一束方向光能影响物体表面的辐射率。这样只需要通过相对简单的循环计算每个光源的贡献就能完成整个 PBR 光照计算。**当我们需要使用 IBL 将环境光加入计算的时候我们才会需要用到积分，因为环境光可能来自任何方向。**

### **3.2.2 PBR 表面模型（ PBR surface model）**

我们先从写一个能满足前面讲到的 PBR 模型的片源着色器开始。首先，我们需要将表面的 PBR 相关属性输入着色器：

```
#version 330 core
out vec4 FragColor;
in vec2 TexCoords;
in vec3 WorldPos;
in vec3 Normal;
  
uniform vec3 camPos;
  
uniform vec3  albedo;
uniform float metallic;
uniform float roughness;
uniform float ao;
```

我们能从顶点着色器拿到常见的输入，另外一些是物体表面的材质属性。

在片源着色器开始的时候，我们先要做一些所有光照算法都需要做的计算：

```
void main()
{
    vec3 N = normalize(Normal); 
    vec3 V = normalize(camPos - WorldPos);
    [...]
}
```

#### **3.2.2.1 直接光照（Direct lighting）**

在这个教程的示例中，我们将会有 4 个点光源作为场景辐照度来源。为了满足反射方程我们循环处理每一个光源，计算它独自的辐射率，然后加总经过 BRDF 跟入射角缩放的结果。我们可以把这个循环当作是积分运算的一种实现方案。首先，计算每个光源各自相关参数：

```
vec3 Lo = vec3(0.0);
for(int i = 0; i < 4; ++i) 
{
    vec3 L = normalize(lightPositions[i] - WorldPos);
    vec3 H = normalize(V + L);
  
    float distance    = length(lightPositions[i] - WorldPos);
    float attenuation = 1.0 / (distance * distance);
    vec3 radiance     = lightColors[i] * attenuation; 
    [...] // 还有逻辑放在后面继续探讨，所以故意在for循环缺了‘}’。
```

由于我们是在线性空间进行的计算（在最后阶段处理 Gamma 校正），所以光源的衰减会更符合物理上的反平方律（inverse-square law）。

反平方律虽然物理学正确，但我们可能还会使用常量、线性、二次方程式来更好地控制光照衰减，即便这些衰减不是物理学正确的。

然后，我们对每个光源计算所有的 Cook-Torrance BRDF 分量：

$$\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

我们要做的第一件事是计算高光跟漫反射之间的比例，有多少光被反射出去了又有多少产生了折射。前面的教程我们讲到过这个菲涅尔方程：

```
vec3 fresnelSchlick (float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
```

Fresnel-Schlick 算法需要的 `F0` 参数就是我们之前说的基础反射率，即以 0 度角照射在表面上的光被反射的比例。不同材质的 `F0` 的值都不一样，可以根据材质到那张非常大的材质表里去找。在 PBR 金属度流水线中我们做了一个简单的假设，我们认为大部分的电介质表面的 `F0` 用 0.04 效果看起来很不错。而金属表面我们将 `F0` 放到 albedo 纹理内，这些可以写成代码如下：

```
vec3 F0 = vec3(0.04); 
F0      = mix(F0, albedo, metallic);
vec3 F  = fresnelSchlick(max(dot(H, V), 0.0), F0);
```

如上述代码所见，非金属的 `F0` 永远是 0.04，除非我们通过金属度属性在 `F0` 跟 `albedo` 之间进行线性插值，才能得到一个不同的非金属 `F0`。

有了 `F`，还剩下法线分布函数 $D$ 跟几何函数 $G$ 需要计算。

在直接光照的 PBR 光照着色器中它们等价于如下代码：

```
float DistributionGGX(vec3 N, vec3 H, float roughness) {
    float a      = roughness*roughness;
    float a2     = a*a;
    float NdotH  = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;
	
    float num   = a2;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    denom = PI * denom * denom;
	
    return num / denom;
}

float GeometrySchlickGGX(float NdotV, float roughness) {
    float r = (roughness + 1.0);
    float k = (r*r) / 8.0;

    float num   = NdotV;
    float denom = NdotV * (1.0 - k) + k;
	
    return num / denom;
}

float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness) {
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2  = GeometrySchlickGGX(NdotV, roughness);
    float ggx1  = GeometrySchlickGGX(NdotL, roughness);
	
    return ggx1 * ggx2;
}
```

这里值得注意的是，相较于 3.1 理论篇教程，我们直接传入了粗糙度参数进函数。这样我们就可以对原始粗糙度做一些特殊操作。根据迪斯尼的原则和 Epic Games 的用法，在法线分布函数跟几何函数中使用粗糙度的平方替代原始粗糙度进行计算光照效果会更正确一些。

当这些都定义好了之后，在计算 NDF 和 G 分量就是很简单的事情了：

```
float NDF = DistributionGGX(N, H, roughness);       
float G   = GeometrySmith(N, V, L, roughness);
```

然后就可以计算 Cook-Torrance BRDF 了：

```
vec3 numerator    = NDF * G * F;
float denominator = 4.0 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0);
vec3 specular     = numerator / max(denominator, 0.001);
```

`denominator` 项里的 `0.001` 是为了防止除 0 情况而特意加上的。

到这里，我们终于可以计算每个光源对反射方程的贡献了。因为菲涅尔值相当于 $k_S$，可用 `F` 代表任意光击中表面后被反射的部分，根据能量守恒定律我们可以用 $k_S$ 直接计算得到 $k_D$：

```
vec3 kS = F;
vec3 kD = vec3(1.0) - kS;
  
kD *= 1.0 - metallic; // 由于金属表面不折射光，没有漫反射颜色，通过归零kD来实现这个规则
```

$k_S$ 表示的是光能有多少被反射了，剩下的被折射的光能我们用 $k_D$ 来表示。此外，由于金属表面不折射光，因此没有漫反射颜色，我们通过归零 $k_D$ 来实现这个规则。

有了这些数据，我们终于可以算出每个光源的出射光了：

```
const float PI = 3.14159265359;
  
    float NdotL = max(dot(N, L), 0.0);        
    Lo += (kD * albedo / PI + specular) * radiance * NdotL;
}
```

最终结果 `Lo`，或者说出射辐射度（Radiosity），实际上是反射方程在半球 $\Omega$ 的积分 $\int$ 的结果。这里要特别注意的是，我们将 $k_S$ 移除方程式，是因为我们已经在 BRDF 中乘过菲涅尔参数 `F` 了，此处不需要再乘一次。

我们没有真正的对所有可能的入射光方向进行积分，因为我们已经清楚的知道只有 4 个入射方向可以影响这个片元，所以我们只需要直接用循环处理这些入射光就行了。

剩下的就是要将 AO 运用到光照结果 `Lo` 上，我们就可以得到这个片元的最终颜色了：

```
vec3 ambient = vec3(0.03) * albedo * ao;
vec3 color   = ambient + Lo;
```

#### **3.2.2.2 线性和 HDR 渲染（ Linear and HDR rendering）**

以上我们假设所有计算都在线性空间，为了使用这个结果我们还需要在着色器的最后进行**伽马校正（Gamma Correct）**，在线性空间计算光照对于 PBR 是非常非常重要的，所有输入参数同样要求是线性的，不考虑这一点将会得到错误的光照结果。

另外，我们希望输入的灯光参数更贴近实际的物理参数，比如他们的辐射度或者颜色值可以是一个非常宽广的值域。这样作为结果输出的 `Lo` 也将变得很大，如果我们不做处理默认会直接 Clamp 到 0.0 至 1.0 之间以适配低动态范围（LDR）输出方式。

为了有效解决 `Lo` 的值域问题，我们可以使用色调映射（Tone Map）和曝光控制（Exposure Map），用它们将 `Lo` 的高动态范围（HDR）映射到 LDR 之后再做伽马校正：

```
color = color / (color + vec3(1.0)); // 色调映射
color = pow(color, vec3(1.0/2.2)); 	 // 伽马校正
```

这里我们使用的是莱因哈特算法（Reinhard operator）对 HDR 进行 Tone Map 操作，尽量在伽马矫正之后还保持高动态范围。我们并没有分开帧缓冲或者使用后处理，所以我们可以直接将 Tone Mapping 和伽马矫正放在前向片元着色器（forward fragment shader）。  

![](1679148477347.png)

对于 PBR 渲染管线来说，线性空间跟高动态范围有着超乎寻常的重要性，没有这些就不可能绘制出不同灯光强度下的高光低光细节，错误的计算结果会产生难看的渲染效果。

#### **3.2.2.3 完整的 PBR 直接光照着色器**

现在唯一剩下的就是将最终的色调映射和伽玛校正的颜色传递给片元着色器的输出通道，我们就拥有了一个 PBR 直接光照着色器。基于完整性考虑，下面列出完整的 `main` 函数：

```
#version 330 core
out vec4 FragColor;
in vec2 TexCoords;
in vec3 WorldPos;
in vec3 Normal;

// material parameters
uniform vec3 albedo;
uniform float metallic;
uniform float roughness;
uniform float ao;

// lights
uniform vec3 lightPositions[4];
uniform vec3 lightColors[4];

uniform vec3 camPos;

const float PI = 3.14159265359;
// --------------------------------------------------------------------
float DistributionGGX(vec3 N, vec3 H, float roughness)
{
    float a = roughness*roughness;
    float a2 = a*a;
    float NdotH = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;

    float nom   = a2;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    denom = PI * denom * denom;

    return nom / max(denom, 0.001); // prevent divide by zero for roughness=0.0 and NdotH=1.0
}
// --------------------------------------------------------------------
float GeometrySchlickGGX(float NdotV, float roughness)
{
    float r = (roughness + 1.0);
    float k = (r*r) / 8.0;

    float nom   = NdotV;
    float denom = NdotV * (1.0 - k) + k;

    return nom / denom;
}
// --------------------------------------------------------------------
float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness)
{
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2 = GeometrySchlickGGX(NdotV, roughness);
    float ggx1 = GeometrySchlickGGX(NdotL, roughness);

    return ggx1 * ggx2;
}
// --------------------------------------------------------------------
vec3 fresnelSchlick(float cosTheta, vec3 F0)
{
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
// --------------------------------------------------------------------
void main()
{		
    vec3 N = normalize(Normal);
    vec3 V = normalize(camPos - WorldPos);

    // calculate reflectance at normal incidence; if dia-electric (like plastic) use F0 
    // of 0.04 and if it's a metal, use the albedo color as F0 (metallic workflow) 
    vec3 F0 = vec3 (0.04); 
    F0 = mix(F0, albedo, metallic);

    // reflectance equation
    vec3 Lo = vec3(0.0);
    for(int i = 0; i < 4; ++i) 
    {
        // calculate per-light radiance
        vec3 L = normalize(lightPositions[i] - WorldPos);
        vec3 H = normalize(V + L);
        float distance = length(lightPositions[i] - WorldPos);
        float attenuation = 1.0 / (distance * distance);
        vec3 radiance = lightColors[i] * attenuation;

        // Cook-Torrance BRDF
        float NDF = DistributionGGX(N, H, roughness);   
        float G   = GeometrySmith (N, V, L, roughness);      
        vec3 F    = fresnelSchlick(clamp(dot(H, V), 0.0, 1.0), F0);
           
        vec3 nominator    = NDF * G * F; 
        float denominator = 4 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0);
        vec3 specular = nominator / max(denominator, 0.001); // prevent divide by zero for NdotV=0.0 or NdotL=0.0
        
        // kS is equal to Fresnel
        vec3 kS = F;
        // for energy conservation, the diffuse and specular light can't
        // be above 1.0 (unless the surface emits light); to preserve this
        // relationship the diffuse component (kD) should equal 1.0 - kS.
        vec3 kD = vec3(1.0) - kS;
        // multiply kD by the inverse metalness such that only non-metals 
        // have diffuse lighting, or a linear blend if partly metal (pure metals
        // have no diffuse light).
        kD *= 1.0 - metallic;	  

        // scale light by NdotL
        float NdotL = max(dot(N, L), 0.0);        

        // add to outgoing radiance Lo
        Lo += (kD * albedo / PI + specular) * radiance * NdotL;  // note that we already multiplied the BRDF by the Fresnel (kS) so we won't multiply by kS again
    }   
    
    // ambient lighting (note that the next IBL tutorial will replace 
    // this ambient lighting with environment lighting).
    vec3 ambient = vec3(0.03) * albedo * ao;

    vec3 color = ambient + Lo;

    // HDR tonemapping
    color = color / (color + vec3(1.0));
    // gamma correct
    color = pow(color, vec3(1.0/2.2)); 

    FragColor = vec4(color, 1.0);
}
```

希望在学习了前面教程的反射方程的理论知识之后，这个 shader 不再会让大家苦恼。使用这个 shader，4 个点光源照射在金属度和粗糙度不同的球上的效果大概类似这样：  

![](1679148477380.png)

从下往上金属度的值从 0.0 到 1.0，粗糙度从左往右从 0.0 增加到 1.0。可以通过观察小球之间的区别理解金属度和粗糙度参数的作用。

示例的源码可以从 [LearnOpenGL 的网站](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/1.1.lighting/lighting.cpp)找到。

### **3.2.3 使用纹理的 PBR（Textured PBR）**

3.2.2.3 小节的 PBR 实现中，部分重要的表面材质属性是 `float` 类型：

```
uniform float metallic;
uniform float roughness;
uniform float ao;
```

实际上，可以将它们用纹理代替，使用纹理的 PBR 可以更加精确地控制表面材质的细节，使得渲染效果更佳。Unity 支持这种方法。

为了实现逐像素的控制材质表面的属性我们必须使用纹理替代单个的材质参数：

```
[...]
uniform sampler2D albedoMap;
uniform sampler2D normalMap;
uniform sampler2D metallicMap;
uniform sampler2D roughnessMap;
uniform sampler2D aoMap;
  
void main()
{
    vec3 albedo     = pow(texture(albedoMap, TexCoords).rgb, 2.2);
    vec3 normal     = getNormalFromNormalMap();
    float metallic  = texture(metallicMap, TexCoords).r;
    float roughness = texture(roughnessMap, TexCoords).r;
    float ao        = texture(aoMap, TexCoords).r;
    [...]
}
```

要注意美术制作的 albedo 纹理一般都是 sRGB 空间的，因此我们要先转换到线性空间再进行后面的计算。根据美术资源的不同，AO 纹理也许同样需要从 sRGB 转换到线性空间。

将前面那些小球的材质属性替换成纹理之后，对比以前用的光照算法，PBR 有了一个质的提升：  

![](1679148477400.png)

可以在这里找到带纹理的 [Demo 源码](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/1.2.lighting_textured/lighting_textured.cpp)，所有用到的纹理在[这里](http://freepbr.com/materials/rusted-iron-pbr-metal-material-alt/)（用了白色的 AO 贴图）。记住金属表面在直接光照环境中更暗是因为他们没有漫反射。在环境使用环境高光进行光照计算的情况下看起来也是正常的，这个我们在下一个教程里再说。

这里没有其他 PBR 渲染示例中那样令人惊艳的效果，因为我们还没有加入基于图片的光照（Image Based Lighting）技术。尽管如此，这个 shader 任然算是一个基于物理的渲染，即使没有 IBL 你也可以法线光照看起来真实了很多。

# **3.3 基于图像的光照（Image Based Lighting，IBL）**

基于图像的光照（IBL）是对光源物体的技巧集合，与直接光照不同，它将周围环境当成一个大光源。IBL 通常结合 cubemap 环境贴图，cubemap 通常采集自真实的照片或从 3D 场景生成，这样可以将其用于光照方程：将 cubemap 的每个像素当成一个光源。这样可以更有效地捕获全局光照和常规感观，使得被渲染的物体更好地融入所处的环境中。

当基于图像的光照算法获得一些（全局的）环境光照时，它的输入被当成更加精密形式的环境光照，甚至是一种粗糙的全局光照的模拟。这使得 IBL 有助于 PBR 的渲染，使得物体渲染效果更真实。

在介绍 IBL 结合 PBR 之前，先回顾一下反射方程：

$$L_o (p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4 (\omega_o \cdot n)(\omega_i \cdot n)}) L_i (p,\omega_i) n \cdot \omega_i d\omega_i$$

如之前所述，我们的主目标是解决所有入射光 $w_i$ 通过半球 $\Omega$ 的积分 $\int$。与直接光照不同的是，在 IBL 中，**每一个**来自周围环境的入射光 $\omega_i$ 都可能存在辐射，这些辐射对解决积分有着重要的作用。为解决积分有两个要求：

*   需要用某种方法获得给定任意方向向量 $\omega_i$ 的场景辐射。
*   解决积分需尽可能快并实时。

对第一个要求，相对简单，采用环境 cubemap。给定一个 cubemap，可以假设它的每个像素是一个单独的发光光源。通过任意方向向量 $\omega_i$ 采样 cubemap，可以获得场景在这个方向的辐射。

获取任意方向向量 $\omega_i$ 的场景辐射很简单，如下：

```
vec3 radiance = texture(_cubemapEnvironment, w_i).rgb;
```

对要求二，解决积分能只考虑一个方向的辐射，要考虑环境贴图的半球 $\Omega$ 的所有可能的方向 $\omega_i$，但常规积分方法在片元着色器中开销非常大。为了有效解决积分问题，可采用预计算或预处理的方法。因此，需要深究一下反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

可将上述的 $k_d$ 和 $k_s$ 项拆分：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i+ \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

拆分后，可分开处理漫反射和镜面反射的积分。先从漫反射积分开始。

### **3.3.1 漫反射辐照度（Diffuse irradiance）**

仔细分析上面方程的漫反射积分部分，发现 Lambert 漫反射是个常量项（颜色 $c$，折射因子 $k_d$ 和 $\pi$）并且不依赖积分变量。因此，可见常量部分移出漫反射积分：

$$L_o(p,\omega_o) = k_d\frac{c}{\pi} \int\limits_{\Omega} L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

因此，积分只依赖 $\omega_i$（假设 $p$ 在环境贴图的中心）。据此，可以计算或预计算出一个新的 cubemap，这个 cubemap 存储了用**卷积**（convolution）计算出的每个采样方向（或像素）$\omega_o$ 的漫反射积分结果。

**卷积**（convolution）是对数据集的每个入口应用一些计算，假设其它所有的入口都在这个数据集里。此处的数据集就是场景辐射或环境图。因此，对 cubemap 的每个采样方向，我们可以顾及在半球 $\Omega$ 的其它所有的采样方向。

为了卷积环境图，我们要解决每个输出 $\omega_o$ 采样方向的积分，通过离散地采样大量的在半球 $\Omega$ 的方向 $\omega_i$ 并取它们辐射的平均值。采样方向 $\omega_i$ 的半球是以点 $p$ 为中心以 $\omega_o$ 为法平面的。  

![](1679148477451.png)

这个预计算的为每个采样方向 $\omega_o$ 存储了积分结果的 cubemap，可被当成是预计算的在场景中所有的击中平行于 $\omega_o$ 表面的非直接漫反射的光照之和。这种 cubemap 被称为**辐照度图（Irradiance map）**。

辐射方程依赖于位置 $p$，假设它在辐照度图的中心。这意味着所有非直接漫反射光需来自于同一个环境图，它可能打破真实的幻觉（特别是室内）。渲染引擎用放置遍布场景的反射探头（reflection probe）来解决，每个反射探头计算其所处环境的独自的辐照度图。这样，点 p 的辐射率（和辐射）是与其最近的反射探头的辐照度插值。这里我们假设总是在环境图的中心采样。反射探头将在其它章节探讨。

下面是 cubemap 环境图 (下图左）和对应的辐照度图（下图右）：  

![](1679148477478.png)

通过存储每个 cubemap 像素卷积的结果，辐照度图有点像环境的平均颜色或光照显示。从这个环境图采样任意方向，可获得这个方向的场景辐照度。

#### **3.3.1.1 球体图（Equirectangular map）**

球体图（Equirectangular map）有些文献翻译成全景图，它与 cubemap 不一样的是：cubemap 需要 6 张图，而球体图只需要一张，并且存储的贴图有一定形变：  

![](1679148477522.png)

cubemap 是可以通过一定算法转成球体图的，详见[这里](https://stackoverflow.com/questions/34250742/converting-a-cubemap-into-equirectangular-panorama)。

#### **3.3.1.2 从球体图到立方体图**

直接从球体图采样出环境光照信息是可能的，但它的开销远大于直接采样立方体图（cubemap）。因此，需要将球体图先转成立方体图，以便更好地实现后面的逻辑。当然，这里也会阐述如何从作为 3D 环境图的球体图采样，以便大家有更多的选择权。

为了将球体图映射到立方体图，首先需要构建一个立方体模型，渲染这个立方体模型的顶点着色器如下：

```
#version 330 core
layout (location = 0) in vec3 aPos;

out vec3 localPos;

uniform mat4 projection;
uniform mat4 view;

void main()
{
    localPos = aPos;  
    gl_Position =  projection * view * vec4(localPos, 1.0);
}
```

在像素着色器中，将会对变形的球体图的每个部位映射到立方体的每一边，具体实现如下：

```
#version 330 core
out vec4 FragColor;
in vec3 localPos;

uniform sampler2D equirectangularMap;

const vec2 invAtan = vec2(0.1591, 0.3183);
vec2 SampleSphericalMap(vec3 v)
{
    vec2 uv = vec2(atan(v.z, v.x), asin(v.y));
    uv *= invAtan;
    uv += 0.5;
    return uv;
}

void main()
{
    // make sure to normalize localPos
    vec2 uv = SampleSphericalMap(normalize(localPos)); 
    vec3 color = texture(equirectangularMap, uv).rgb;
    
    FragColor = vec4(color, 1.0);
}
```

渲染出来的立方体效果如下：  

![](1679148477573.png)

对于立方体图的采样，顶点着色器如下：

```
#version 330 core
layout (location = 0) in vec3 aPos;

uniform mat4 projection;
uniform mat4 view;

out vec3 localPos;

void main()
{
    localPos = aPos;

    // remove translation from the view matrix
    mat4 rotView = mat4(mat3(view)); 
    vec4 clipPos = projection * rotView * vec4(localPos, 1.0);

    gl_Position = clipPos.xyww;  // 注意这里的分量是`xyww`！！
}
```

对于立方体图的采样，像素着色器如下：

```
#version 330 core
out vec4 FragColor;

in vec3 localPos;
  
uniform samplerCube environmentMap;
  
void main()
{
    // 从cubemap采样颜色
    vec3 envColor = texture(environmentMap, localPos).rgb;
    
    // HDR -> LDR
    envColor = envColor / (envColor + vec3(1.0));
    // Gamma 校正（只在颜色为线性空间的渲染管线才需要）
    envColor = pow(envColor, vec3(1.0/2.2)); 
  
    FragColor = vec4(envColor, 1.0);
}
```

上述代码中，要注意在输出最终的颜色之前，做了 HDR 到 LDR 的转换和 Gamma 校正。

渲染的效果如下图：  

![](1679148477687.png)

#### **3.3.1.3 PBR 和非直接辐射度光照（indirect irradiance lighting）**

辐射度图提供了漫反射部分的积分，该积分表示来自非直接的所有方向的环境光辐射之和。由于辐射度图被当成是无方向性的光源，所以可以将漫反射镜面反射合成环境光。

首先，得声明预计算出的辐射度图的 sample：

```
uniform samplerCube irradianceMap;
```

通过表面的法线，获得环境光可以简化成下面的代码：

```
// vec3 ambient = vec3 (0.03);
vec3 ambient = texture(irradianceMap, N).rgb;
```

尽管如此，在之前所述的反射方程中，非直接光依旧包含了漫反射和镜面反射两个部分，所以我们需要加个权重给漫反射。下面采用了菲涅尔方程来计算漫反射因子：

```
vec3 kS = fresnelSchlick(max(dot(N, V), 0.0), F0);
vec3 kD = 1.0 - kS;
vec3 irradiance = texture(irradianceMap, N).rgb;
vec3 diffuse    = irradiance * albedo;
vec3 ambient    = (kD * diffuse) * ao;
```

由于环境光来自在半球内所有围绕着法线 `N` 的方向，没有单一的半向量去决定菲涅尔因子。为了仍然能模拟菲涅尔，这里采用了法线和视线的夹角。之前的算法采用了受表面粗糙度影响的微表面半向量，作为菲涅尔方程的输入。这里，我们并不考虑粗糙度，表面的反射因子被视作相当大。

非直接光照将沿用直接光照的相同的属性，所以，期望越粗糙的表面镜面反射越少。由于不考虑表面粗糙度，非直接光照的菲涅尔方程强度被视作粗糙的非金属表面（下图）。  

![](1679148477738.png)

为了缓解这个问题，可在 Fresnel-Schlick 方程注入粗糙度项（该方程的[来源](https://seblagarde.wordpress.com/2011/08/17/hello-world/)）：

```
vec3 fresnelSchlickRoughness(float cosTheta, vec3 F0, float roughness) {
    return F0 + (max(vec3(1.0 - roughness), F0) - F0) * pow(1.0 - cosTheta, 5.0);
}
```

考虑了表面粗糙度后，菲涅尔相关计算最终如下：

```
vec3 kS = fresnelSchlickRoughness(max(dot(N, V), 0.0), F0, roughness); 
vec3 kD = 1.0 - kS;
vec3 irradiance = texture(irradianceMap, N).rgb;
vec3 diffuse    = irradiance * albedo;
vec3 ambient    = (kD * diffuse) * ao;
```

如上所述，实际上，基于图片的光照计算非常简单，只需要单一的 cubemap 纹理采样。大多数的工作在于预计算或卷积环境图到辐射度图。

加入了 IBL 的渲染效果如下（竖向是金属度增加，水平是粗糙度增加）：  

![](1679148477776.png)

本节所有代码可在[这里](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.1.2.ibl_irradiance/ibl_irradiance.cpp)找到。

### **3.3.2 镜面的 IBL（Specular IBL）**

3.3.1 描述的是 IBL 的漫反射部分，本节将讨论 IBL 的镜面反射部分先回顾一下反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

上述的镜面反射部分（被 $k_s$ 相乘）不是恒定的，并且依赖于入射光方向和视线入射方向，尝试实时地计算所有入射光和所有入射视线的积分是几乎不可能的。Epic Games 推荐折中地使用预卷积镜面反射部分的方法来解决实时渲染的性能问题，这就是**分裂和近似法（split sum approximation）**。

**分裂和近似法**将镜面反射部分从反射方程分离出两个部分，这样可以单独地对它们卷积，后面在 PBR 的 shader 中为镜面的非直接 IBL 将它们结合起来。跟预卷积辐射度图类似，分裂和近似法需要 HDR 环境图作为输入。为了更好地理解分裂和近似法，下面着重关注反射方程的镜面部分：

$$\begin{eqnarray*} L_o(p,\omega_o) & = & \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)} L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i \end{eqnarray*}$$

出于跟辐射度图相同的性能问题的考虑，我们要预计算类似镜面 IBL 图的积分，并且用片元的法线采样这个图。辐射度图的预计算只依赖于 $\omega_i$，并且我们可以将漫反射项移出积分。但这次从 BRDF 可以看出，不仅仅是依赖于 $\omega_i$：

$$f_r(p, w_i, w_o) = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

如上方程所示，还依赖 $\omega_o$，并且我们不能用两个方向向量来采样预计算的 cubemap。预计算所有 $\omega_i$ 和 $\omega_o$ 的组合在实时渲染环境中不实际的。

Epic Games 的分裂和近似法将镜面反射部分从反射方程分离出两个部分，这样可以单独地对它们卷积，后面在 PBR 的 shader 中为镜面的非直接 IBL 将它们结合起来。分离后的方程如下：

$$\begin{eqnarray*} L_o(p,\omega_o) & = & \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)} L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} L_i(p,\omega_i) d\omega_i * \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d\omega_i \end{eqnarray*}$$

第一部分 $\int\limits_{\Omega} L_i(p,\omega_i) d\omega_i$ 是**预过滤环境图（pre-filtered environment map）**，类似于辐射度图的预计算环境卷积图，但会加入粗糙度。随着粗糙度等级的增加，环境图使用更多的散射采样向量来卷积，创建出更模糊的反射。

对每个卷积的粗糙度等级，循环地在预过滤环境图的 mimap 等级存储更加模糊的结果。下图是 5 个不同粗糙度等级的预过滤环境图：  

![](1679148477798.png)

生成采样向量和它们的散射强度，需要用到 Cook-Torrance BRDF 的法线分布图（NDF），而其带了两个输入：法线和视线向量。当卷积环境图时并不知道视线向量，Epic Games 用了更近一步的模拟法：假设视线向量（亦即镜面反射向量）总是等于输出采样向量 $\omega_o$。所以代码变成如下所示：

```
vec3 N = normalize(w_o);
vec3 R = N;
vec3 V = R;
```

这种方式预过滤环境图卷积不需要关心视线方向。这就意味着当从某个角度看向下面这张图的镜面表面反射时，无法获得很好的掠射镜面反射（grazing specular reflections）。然而通常这被认为是一个较好的妥协：  

![](1679148477844.png)

第二部分 $\int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d\omega_i$ 是**镜面积分**。假设所有方向的入射辐射率是全白的（那样 $L(p, x) = 1.0$），那就可以用给定的粗糙度和一个法线 $n$ 和光源方向 $\omega_i$ 之间的角度或 $n \cdot \omega_i$ 来预计算 BRDF 的值。Epic Games 存储了用变化的粗糙度来预计算每一个法线和光源方向组合的 BRDF 的值，该粗糙度存储于 2D 采样纹理（LUT）中，它被称为 **BRDF 积分图（BRDF integration map）**。

2D 采样纹理输出一个缩放（红色）和一个偏移值（绿色）给表面的菲涅尔方程式（Fresnel response），以便提供第二部分的镜面积分：  

![](1679148477893.png)

  
上图水平表示 BRDF 的输入 $n \cdot \omega_i$，竖向表示输入的粗糙度。

有了预过滤环境图和 BRDF 积分图，可以在 shader 中将它们结合起来：

```
float lod             = getMipLevelFromRoughness(roughness);
vec3 prefilteredColor = textureCubeLod(PrefilteredEnvMap, refVec, lod);
vec2 envBRDF          = texture2D(BRDFIntegrationMap, vec2(NdotV, roughness)).xy;
vec3 indirectSpecular = prefilteredColor * (F * envBRDF.x + envBRDF.y)
```

### **3.3.3 完整的 IBL**

首先是声明 IBL 镜面部分的两个纹理采样器：

```
uniform samplerCube prefilterMap;
uniform sampler2D   brdfLUT;
```

接着用法线 `N` 和视线 `-V` 算出反射向量 `R`，再结合 `MAX_REFLECTION_LOD` 和粗糙度等参数采样预过滤环境图：

```
void main()
{
    [...]
    vec3 R = reflect(-V, N);   

    const float MAX_REFLECTION_LOD = 4.0;
    vec3 prefilteredColor = textureLod(prefilterMap, R,  roughness * MAX_REFLECTION_LOD).rgb;    
    [...]
}
```

然后用视线、法线的夹角及粗糙度采样 BRDF 查找纹理，结合预过滤环境图的颜色算出 IBL 的镜面部分：

```
vec3 F        = FresnelSchlickRoughness(max(dot(N, V), 0.0), F0, roughness);
vec2 envBRDF  = texture(brdfLUT, vec2(max(dot(N, V), 0.0), roughness)).rg;
vec3 specular = prefilteredColor * (F * envBRDF.x + envBRDF.y);
```

自此，反射方程的非直接的镜面部分已经算出来了。可以将它和上一小节的 IBL 的漫反射部分结合起来：

```
vec3 F = FresnelSchlickRoughness(max(dot(N, V), 0.0), F0, roughness);

vec3 kS = F;
vec3 kD = 1.0 - kS;
kD *= 1.0 - metallic;	  
  
vec3 irradiance = texture(irradianceMap, N).rgb;
vec3 diffuse    = irradiance * albedo;
  
const float MAX_REFLECTION_LOD = 4.0;
vec3 prefilteredColor = textureLod(prefilterMap, R,  roughness * MAX_REFLECTION_LOD).rgb;   
vec2 envBRDF  = texture(brdfLUT, vec2(max(dot(N, V), 0.0), roughness)).rg;
vec3 specular = prefilteredColor * (F * envBRDF.x + envBRDF.y);
  
vec3 ambient = (kD * diffuse + specular) * ao;
```

此时可以算出由 IBL 的漫反射和镜面反射部分结合而成的环境光 `ambient`，渲染效果如下：  

![](1679148477938.png)

扩展一下，加入一些酷酷的[材质](https://freepbr.com/)：  

![](1679148477981.png)

或者加载[这些极好又免费的 PBR 3D 模型](http://artisaverb.info/PBT.html)（by Andrew Maximov）：  

![](1679148478025.png)

非常肯定地，加了 IBL 光照后，渲染效果更真实更加物理正确。下图展示了在未改变任何光照信息的情况下，在不同的预计算 HDR 图中的效果，它们看起来依然是物理正确的：  

![](1679148478048.png)

IBL 的教程结束了，本节的代码可在[球体场景](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.2.1.ibl_specular/ibl_specular.cpp)和[纹理场景](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.2.2.ibl_specular_textured/ibl_specular_textured.cpp)中找到。


