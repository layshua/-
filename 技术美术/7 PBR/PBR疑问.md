# Albedo 为什么是 0 到 1 之间的数，与反射率有什么不同？

![[52a9c6f91d6a0779879d60745aa8c980_MD5.jpg]]

马小刀​

物理学术语，PBR 渲染也会用到此概念。直接放自己的文章好了。见文中的第二节：2. 反照率。

[马小刀：为什么 BRDF 的漫反射项要除以π？](https://zhuanlan.zhihu.com/p/342807202)**反照率（**albedo）是行星物理学中用来表示天体反射本领的物理量，定义为物体的**辐射度（**radiosity）与**辐照度**（irradiance）之比。射是出，照是入，出射量除以入射量，得到**无量纲量**，即没有单位的数值。  
绝对**黑体**（black body）的反照率是 0。煤炭呈黑色，反照率接近 0，因为它吸收了投射到其表面上的几乎所有可见光。镜面将可见光几乎全部反射出去，其反照率接近 1。albedo 翻译成**反照率**，与 reflectance（**反射率**）是有区别的。反射率用来表示单一一种波长的反射能量与入射能量之比；而反照率用来表示全波段的反射能量与入射能量之比。  
BRDF 里的 R 是 reflectance，方程仅关注一种波长。自然光是全波段的复色光，不同频率的光混合在一起，通过三棱镜发生色散，白光散开后单色光依次为红、橙、黄、绿、蓝、靛、紫七种颜色。由于光是 R/G/B 三原色的混合，所以你看到有的渲染会考虑这三种波长的 BRDF。

文中提到的辐射率（radiance）与辐照度（irradiance）的概念解释可以参考我在这里的回答。与别人交流的时候，把中文名称与英文单词对应上。

[关于光照的辐射率的定义为什么 Real-Time Rendering4th 和计算机图形学 (虎书) 不一样？](https://www.zhihu.com/question/428761049/answer/1561257717)

![[51ed0fcdf50f1167d612ec4eb116ad11_MD5.jpg]]

三号

Albedo 纹理为每一个金属的纹素 (Texel)（纹理像素）指定表面颜色或者基础反射率。这和漫反射纹理相当类似，不同的是所有光照信息都是由一个纹理中提取的。漫反射纹理的图像当中常常包含一些细小的阴影或者深色的裂纹，而反照率纹理中是不会有这些东西的。它应该只包含表面的颜色（或者折射吸收系数）。——摘自《[LearnOpenGL CN](https://learnopengl-cn.github.io/07%20PBR/01%20Theory/)》

![[cd7fb3b0946a9d30272ed919725278b6_MD5.jpg]]

Neilyodog

非金属下是漫反射颜色，金属是反射率

　　刚学 PBR 时，一下子要接受很多的光学物理量定义。 $E = \frac {d\phi}{dS}$ 、 $L = \frac {d^2 \phi}{dA_{\perp} d\omega}$ 、 $dA_{\perp} = dA \cdot max(\cos \theta, 0)$ …… 然后文档都是直接放公式，为什么 Lambert 光照模型的漫反射项要除以π呢？没作过多说明，这也是很多同学的疑惑。一言以蔽之， $\int _{\Omega} \cos \phi\, d\omega = \pi$ 。下面给出详细解说。

![[608b18b9383d930215355f1e6767b346_MD5.jpg]]

## 1. 反射比方程

 **基于物理的渲染**（Physically Based Rendering，缩写 PBR）光照模型比传统的 Phong、Blinn-Phong 光照模型渲染更真实。PBR 基于**微平面**（microfacet）模型，遵循能量守恒定律。它对现实场景的光照计算是基于物理学定律的，是一种比较接近的模拟，但不能称之为物理渲染（physical shading）。我们用反射比方程（reflectance equation）

$$L_o(p, \vec v) = \int _\Omega f(\vec l, \vec v) \ L_i(p, \vec l) \ (\vec n \cdot \vec l) \ d\omega $$

![[60d7ac994c37849bbdf8bfd22987f2f2_MD5.jpg]]

来计算在 p 点处， $\vec v$ 方向来的辐射率 $L_o(p, \vec v)$ 。 $\vec l, \vec n, \vec v$ 三个均为**单位向量**，已在上图中标注，各符号代表的参数说明一下：

*   $\vec n$ 为物体表面 p 点处的法向量（normal）；
*   $\vec l$ 为入射光线（light）反方向上的向量，注意是反方向，看成从 p 点发射出；
*   $\vec v$ 为观察（view）方向来的向量；
*   Ω 表示积分区域为半球面（有没有觉得这里的希腊字母 Ω 很是形象？其小写字母 ω 在后面表示立体角）；
*   dω 表示球面微立体角。平面角扩展到空间立体角，概念相通。平面角等于张成的弧长除以半径 $\theta = l / r$，立体角等于张成的球面积除以半径的平方 $\omega = A / r^2$。除后都是**无量纲量**，但为了与数区分开，会分别加上后缀 rad、sr ，但这不是单位！比方说 2π，你是指无理数 6.28...，还是圆周角度，还是半球立体角？；
*   $L_i$ 表示入射（incoming）方向上的辐射率；
*   $L_o$ 表示出射（outgoing）方向上的辐射率；
*   f 表示 BRDF（ Bi-directional Reflectance Distribution Function）**双向反射分布函数**，双向是指物体表面任意一点都有入射和出射两个方向。它表示任意出射方向的光线 $\vec v$ 与任意入射的光线 $\vec l$ 的一种比例关系，记作 $f(\vec l, \vec v) = \frac {dL_o(\vec v)}{d E(\vec l)}$ 。L 是**辐射率**（radiance = power / (area * solid_angle)），E 是**辐照度**（irradiance = power / area），前面附着的字母 d 表示微分。f 值也因光的波长而异，由于不同波长的光能用 **RGB 三原色**表示，所以渲染的时候 f 是以向量表示的。

![[5294f9e2ea4607dcc0ee4bf059490f44_MD5.jpg]]

　　光照模型有很多种 [[1]] ( https://zhuanlan.zhihu.com/p/342807202#ref_1 )，Cook-Torrance 光照模型是最常用的。光照一般划分为漫反射和高光。漫反射模型提出的有 [Lambert](https://en.wikipedia.org/wiki/Lambertian_reflectance)、 [Oren-Nayar](https://en.wikipedia.org/wiki/Oren%E2%80%93Nayar_reflectance_model)、[Minnaert](https://en.wikipedia.org/wiki/Minnaert_function) 三种，它们有着不同的计算公式。Cook-Torrance 采用 Lambert 漫反射后的计算如下：

$$f = k_d f_{lambert} + k_s f_{cook-torrance},\hspace{5mm} f_{lambert} = \frac {c_{diffuse}} {\pi} $$

$k_d$ 与 $k_s$ 分别是漫反射、高光系数，能量守恒要求 $k_d + k_s \le 1$。Phong 光照模型在 Lambert 光照模型基础上加入了高光成分为 $k_d (\vec l \cdot \vec n) + k_s (\vec v \cdot \vec r)^n$ 。两者的漫反射项计算很像，区别在于前者是积分形式的光照计算，对结果除以 π 了。那么，为什么 Lambert 漫反射项要除以 π 呢？（高光项也是要除以一个系数，系数放在[这篇文章](https://zhuanlan.zhihu.com/p/508559622)里的 PBR 一节讲解，并推导了积分 $\int _{\Omega} \cos^{n} \theta \ d\omega = \frac {2\pi}{n + 1}$ ）

## 2. 反照率

　　一个粗糙的、无光泽的物体（如粉笔）表面对光的反射表现为漫反射（diffuse reflection），这样的表面对入射光在各个方向上呈强度相同的反射，因而无论从哪个角度观看，该点的亮度都是相同的。没有镜面反射，所以 $k_d = 1,\ k_s = 0$ 。然后漫反射的 $f_{lambert}$ 一项与 $p, \vec l, \vec v$ 参数无关，可以提到积分符号外面去，成了常数。方程就变成了

$L_o(p, \vec v) = f_{lambert} \cdot L_i(p, \vec l) \int _\Omega \cos\phi \ d\omega$ ，由于积分 $\int _{\Omega} \cos \phi\, d\omega = \pi$ （后面会给出推导过程），得到 $f_{lambert} = \frac {albedo} {\pi}$ ， $albedo = \frac {\sum L_o} {\sum L_i}$ 。这里出现了除以π。

　　这里科普一下几个术语。**反照率（**albedo）是行星物理学中用来表示天体反射本领的物理量，定义为物体的**辐射度（**radiosity）与**辐照度**（irradiance）之比。射是出，照是入，出射量除以入射量，得到**无量纲量**。绝对**黑体**（black body）的反照率是 0。煤炭呈黑色，反照率接近 0，因为它吸收了投射到其表面上的几乎所有可见光。镜面将可见光几乎全部反射出去，其反照率接近 1。albedo 翻译成**反照率**，与 reflectance（**反射率**）是有区别的。反射率用来表示某一种波长的反射能量与入射能量之比；而反照率用来表示全波段的反射能量与入射能量之比。BRDF 的 R 是 reflectance，方程仅关注一种波长。

　　术语 albedo 与 diffuse 语义重叠，有时候会混用。当只存在漫反射，不存在镜面反射的情况下，albedo 才等于 diffuse [[2]] ( https://zhuanlan.zhihu.com/p/342807202#ref_2 )。其取值范围在 0（光全部吸收）到 1（光全部反射）之间，不同波长的光，其 reflectance 值不一样。由于不同波长的光能用 **RGB 三原色**表示，所以 albedo 也用 0 到 1 区间的 vec3 向量表示。场景中的物体不同表面处的 albedo 不一样，就要用上 albedo 纹理贴图了。

## 3. 球面坐标系

　　关于球面积分的计算，转换到**球面坐标系**（ρ, θ, ϕ）中会更方便，而不是在**平面直角坐标系** (x, y, z) 计算。这里提一下，有的地方（网站、课程）交换了字母θ, ϕ的意义，感觉不太好。在二维平面一直用θ表示角度的。从二维延伸到三维，在后面追加字母比较好，免得意义弄混了。这样的话，增加或降低维度都能很好的理解。θ的范围在 [0, 2π)，ϕ的范围在 [0, π]，留意到这里是**闭区间**。地理上倾向于用左右上下描述方位，所以范围关于 0 对称。θ在 [-π, π)，ϕ在 [-π/2, +π/2]。

![[077018734d114a969d0df6e8960b273c_MD5.png]]

　　从平面直角坐标系转到球面坐标系：

$\begin{align} \rho &= \sqrt{x^2 + y^2 + z^2} \\ \theta &= \arctan (y/x) \hspace{8mm} \color{blue}{注意用 std::atan2(y, x)， 而不是 std::atan(y / x) 去计算}\\ \phi &= \arctan(\sqrt{x^2 + y^2} / z) = \arccos(z / \rho) \end{align}$

　　从球面坐标系转到平面直角坐标系：

$\begin{align} x &= \rho \sin \phi \cos \theta \\ y &= \rho \sin \phi \sin \theta \\ z &= \rho \cos \phi \\ \end{align}$

$\vec l$ 和 $\vec v$ 都是单位向量，ρ = 1，于是成了两个**自由度**的变量，可以用（θ, ϕ）表示。反射比方程写成球面坐标系形式就是

$$L_o(\theta _o, \phi _o) = \int _{\theta _i = 0} ^{2 \pi} \int _{\phi _i = 0} ^{\pi / 2} f(\theta _i, \phi _i, \theta _o, \phi _o)\ L_i(\theta _i, \phi _i) \ \sin \phi _i \cos \phi _i\ d \phi _i\ d \theta _i $$

其中， $dx\ dy\ dz = \rho^2 \sin \phi\ d\rho\ d\phi\ d\theta$ ，在[上一篇文章](https://zhuanlan.zhihu.com/p/148759248)提到过 **Jacobi 行列式**。**立体角** ω 定义成与半径 ρ 无关的量，如同圆的弧度与圆半径无关一样， $d \omega = d A / \rho^2$ 。ρ 取 1，或者说拿掉有 $d\omega = \sin \phi\ d\phi\ d\theta$ ，下面的图也会给出证明。

## 4. 积分 $\int _{\Omega} \cos \phi\, d\omega$

　　我们要计算的积分 $\int _{\Omega} \cos \phi\, d\omega$ ，转成球面坐标系为 $\int _{0} ^{2\pi} \int _{0} ^{\pi / 2} \sin \phi \,\cos \phi \,d\phi \,d\theta$ 。

　　θ 与 ϕ相互独立，可以分离成两个积分的乘积。其中 $\int _{0} ^{2\pi} d\theta = 2\pi$ ，

$\int _{0} ^{\pi / 2} \sin \phi \cos \phi\ d\phi = \frac 1 2 \int _{0} ^{\pi / 2} \sin(2\phi)\ d\phi = \frac 1 4 \int _{0} ^{\pi / 2}\sin(2\phi)\ d(2\phi) = -\frac 1 4 \cos(2\phi) | _0 ^{\pi / 2} = -\frac 1 4 (-1 - 1) = \frac 1 2$

　　上面用的二倍角公式 $\sin 2\phi = 2\sin \phi \cos \phi$ 计算出 1/2，其实，根据 $(\sin \phi)' = \cos \phi$ ，也可以这么来计算，殊途同归。

$\int _{0} ^{\pi / 2} \sin \phi \cos \phi d\phi = \int _{0} ^{\pi / 2} \sin \phi\, d(\sin \phi) = \frac 1 2 \sin^2(2\phi) | _{0} ^{\pi / 2} = \frac 1 2 (1^2 - 0^2) = \frac 1 2$

两个积分的结果乘后， $2 \pi * \frac 1 2 = \pi$ 。这就是 π 的由来。

## 5. 答疑解惑

$\color{blue}{问}$ ：**半径为 1 的半球面，表面积是 2π，立体角是 2π。应该除以 2π 的啊，怎么是除以 π ？**

$\color{blue}{答}$ ： $\int _{\Omega} d\omega = 2\pi,\ \int _{\Omega} \cos \phi\, d\omega = \pi$ ，多出的余弦项导致结果不同。其中，Ω 表示积分区域为半球面，dω 表示球面微立体角，ϕ 是与光线 $\vec l$ 与法线 $\vec n$ 的夹角， $\cos \phi = \vec l \cdot \vec n$ ，各符号在第 1 节都有说。本来，计算 ϕ 的式子是 $\cos \phi = \frac {\vec l \cdot \vec n} {||\vec l || \cdot ||\vec n||}$ ，因为都是单位向量，所以可以直接丢掉分母，写成 $\cos \phi = \vec l \cdot \vec n$ 。

　　光沿着垂直其投射方向 $\vec l$ 的圆盘投影到地面（法线为 $\vec n$ ），是一个椭圆（正投影则是圆）。圆的面积是πrr，椭圆的面积是 πab，a、b 分别是长半轴、短半轴。其中短轴的长度不变 b = r，长轴拉伸 a = r / cosϕ。对于正方形，甚至推广到任意闭合曲面，投射到平面的面积 $A_{\perp}$ 跟垂直光线的面积 A 有 $A_{\perp} = A / \cos\phi$ 这种关系。一定强度的光束，想象成手电筒，倾斜照射让投影面积变大（除以 cosϕ），导致单位面积上接收到的光变弱（乘以 cosϕ）。ϕ = 0 时是正投影，cosϕ = 1，没有减弱。ϕ = π/2 时，平行与投射平面，cosϕ = 0，没有接收到光照。

　　稍微提一下，看 PBR 相关的英文文档时，会经常遇到关于余弦的两个术语。一个是 **cosine-weighted**，式子中多了 cosθ 项；另一个是 **clamped cosine**，在代码中中写成 max (dot (L, V), 0.0) 。光照模型只处理半球面，而向量的夹角大于 π/2 时，光在物体内部，其点乘的结果小于 0，没有贡献应该是 0，而不是给出负值。

$\color{blue}{问}$ ：**半径为 1 的圆的面积是π。除以π，那不刚好是除以单位半球面投影到平面上那个圆的面积吗？是数值上的巧合，还是可以理解成除以圆的面积呢？**

$\color{blue}{答}$ ：可以理解成除以圆的面积。r = ρ sin (ϕ) 是投影到 XOY 平面上的半径。

![[9fb967bf2925a6a357a99734129397b1_MD5.jpg]]

　　球面微元 dA 可以看成小块矩形， $dA = \rho d\phi \cdot \rho \sin \phi\, d\theta = \rho^2 \sin\phi\, d\phi \,d\theta$ ，立体角 $d\omega = \frac {dA}{\rho^2} = \sin\phi\, d\phi \,d\theta$ 。球的表面积公式是 $S(r) = 4\pi r^2$，我在[这里](https://zhuanlan.zhihu.com/p/148405054)推导过。整个球面是 4π sr（sr 是 steradian，**球面度**），半球面是 2π sr。然后投影面是 $dA_{\perp} = dA\, \cos\phi$ 。

$dA_{\perp}= \rho^2 \sin\phi\, d\phi \, \cos \phi \, d\theta = \rho \sin \phi\, d(\rho\, \sin \phi)\, d \theta$

　　半径为 r 的圆，平面微元 $dS = dx\ dy = r dr\ d\theta$ 。平面直角坐标系到极坐标系的转换，要乘以**雅可比行列式**，我在[这里](https://zhuanlan.zhihu.com/p/148759248)有过证明。把 r = ρ sin (ϕ) 代入，就是一样的式子了。 $dA_{\perp} = dS$ 。两者的积分区间也是一样的，所以结果也是一样的。

![[2f19ffce101e5be064b7b4ff170e13e2_MD5.jpg]]

![[8722047e0e4b90239ba53949564c6c29_MD5.jpg]]

　　你也可以从上面的图去理解。因为与角度θ无关（或者说任意θ角表现一致，体现在θ是独立变化的，二重积分可拆开单独计算，是个常量），我们只用考虑不同的仰角 ϕ 了。表面积微元 ds （以绿色标记）随光线投影到物质表面的面积是 ds / cosϕ（以红色标记） ，垂直投影到物质表面的面积 dp 是 ds * cosϕ 。从任意角度 ϕ 看，BRDF 方程中的 $\cos \phi\ ds$ 数值大小可以看成垂直方向上的 $\cos \phi\ ds$ （以蓝色标记）。 $\int _\Omega \cos \phi\ d\omega = \int _{\Omega_\perp} 1\ dp = \pi$ 。从数值上来讲，立体角 ω 与半径为 1 的球表面积 s 一样，就如同平面角 θ 与半径为 1 的圆弧长度一样。

$\color{blue}{问}$ ：如何生成在半球面上均匀分布的采样函数 randomHemisphere ()，以及带余弦权重（cosine-weighed）的采样函数 randomHemisphereCosine ()？

$\color{blue}{答}$ ：在[上一篇](https://zhuanlan.zhihu.com/p/148759248)，我们知道了如何生成在球面均匀分布的采样函数 randomSphere ()，稍稍处理就能得到我们要的半球面均匀分布。if (z < 0) z = -z; 的写法有分支判断，还是用 z = abs (z)。这是切向空间（tangent space）下的分布，必要时换算到世界空间（world space）中去。

```
vec3f randomHemisphere() {
	vec3f n = randomSphere();
	float& z = n.z;
	z = std::abs(z);
	return n;
}
```

　　因为积分函数有 $\cos \phi$ 一项，有时需要生成带余弦权重的分布。根据上面一问的思路，做法是生成圆盘上均匀分布的函数 randomDisk（代码见[上一篇](https://zhuanlan.zhihu.com/p/148759248)），然后垂直映射到覆盖的半球面上。

```
vec3f randomHemisphereCosine() {
	vec2f p = randomDisk();
	z = std::sqrt(1 - dot(p, p));  // 1 - x*x - y*y
	return vec3f(p.x, p.y, z);
}
```

　　有一点点可以优化的地方。randomDisk 函数的代码抄进来，不用计算 1 - x*x - y*y，取而代之的是 1 - u1。

```
vec3f randomHemisphereCosine() {
	float u1 = random(), u2 = random();
	float r = std::sqrt(u1);
	float theta = 2 * M_PI * u2;

	float x = r * std::cos(theta);
	float y = r * std::sin(theta);
	float z = std::sqrt(1 - u1);  // 1 - x*x - y*y = 1 - r*r = 1 - u1
	return vec3f(x, y, z);
}
```

## 参考

1.  [^](https://zhuanlan.zhihu.com/p/342807202#ref_1_0) Surface Reflection Models. Frank Losasso [http://www.cs.uns.edu.ar/cg/clasespdf/SRM.pdf](http://www.cs.uns.edu.ar/cg/clasespdf/SRM.pdf)
2.  [^](https://zhuanlan.zhihu.com/p/342807202#ref_2_0) Albedo vs Diffuse [ https://computergraphics.stackexchange.com/questions/350/albedo-vs-diffuse#:~:text=Albedo%20is%20the%20proportion%20of%20incident%20light%20that ,just%20one%20direction%20like%20a%20mirror%20%28specular%20reflection%29]( https://computergraphics.stackexchange.com/questions/350/albedo-vs-diffuse#:~:text=Albedo is the proportion of incident light that,just one direction like a mirror (specular reflection))
