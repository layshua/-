
上章主要介绍了 PBR 的历史和逼真的效果特征。这章将重点介绍 PBR 的核心部分的基本原理及主流的实现方案，使读者对 PBR 的核心理论有一定了解，并能掌握相关的编码。

## **3.1 PBR 基础理论和推导**

本节的理论和推导尽量简化和精简，更深入的原理和理论将在下一章阐述。

满足以下条件的光照模型才能称之为 PBR 光照模型：

*   基于微平面模型（Be based on the microfacet surface model）。
*   能量守恒（Be energy conserving）。
*   使用基于物理的 BRDF（Use a physically based BRDF）。

### **3.1.1 微平面（Microfacet）**

大多数 PBR 技术都是基于微平面理论。在此理论下，认为在微观上所有材质表面都是由很多朝向不一的微小平面组成，有的材质表面光滑一些，有的粗糙一些。

真实世界的物体表面不一定是很多微小平面组成，也可能是带有弧度或者坑坑洼洼。但对于我们肉眼能观察到的维度，PBR 的微观近似模拟方法产生的结果跟实际差别甚微。

![[1679148476503.png]]

  
_所有材质表面由粗糙度不同的微小平面组成。左边材质更粗糙，右边的平滑一些。_

当光线射入这些微平面后，通常会产生镜面反射。对于越粗糙的表面，由于其朝向更无序，反射的光线更杂乱，反之，平滑的微平面，反射的光线更平齐。  

![[1679148476534.png]]

  
上图左边材质表面更粗糙，反射的光线更杂乱；图右的平滑许多，反射的光线更有规律。

从微观角度来说，没有任何表面是完全光滑的。由于这些微平面已经微小到无法逐像素地继续对其进行细分，因此我们只有假设一个粗糙度 (Roughness，即 2.4.1 中提到的粗糙度) 参数，然后用统计学的方法来概略的估算微平面的粗糙程度。

我们可以基于一个平面的粗糙度来计算出某个向量的方向与微平面平均取向方向一致的概率。这个向量便是位于光线向量 $l$和视线向量 $v$之间的中间向量，被称为**半角向量 (Halfway Vector)**。  

![[1679148476577.png]]

  
_半角向量 $h$是视线 $v$和入射光 $l$的中间单位向量。_

半角向量计算公式如下：

$$h = \frac{l + v}{\|l + v\|}$$

半角向量计算 GLSL 实现:

```
// lightPos是光源位置，viewPos是摄像机位置，FragPos是像素位置
vec3 lightDir   = normalize(lightPos - FragPos);
vec3 viewDir    = normalize(viewPos - FragPos);
vec3 halfwayDir = normalize(lightDir + viewDir);
```

越多的微平面取向与其半角向量一致，材质镜面反射越强越锐利。加上引入取值 0~1 的粗糙度，可以大致模拟微平面的整体取向。  

![[1679148476601.png]]

  
_粗糙度从 0.1~1.0 的变化图。粗糙度越小，镜面反射越亮范围越小；粗糙度越大，镜面反射越弱。_

### **3.1.2 能量守恒（Energy Conservation）**

在微平面理论中，采用近似的能量守恒：出射光的总能量不超过入射光的总能量（自发光材质除外）。3.1.1 的粗糙度变化图可以看出，材质粗糙度越大，反射的范围越大，但整体亮度变暗。

那么 PBR 是如何实现近似的能量守恒呢？

为了回答这个问题，先弄清楚**镜面反射**（specular）和**漫反射**（diffuse）的区别。

一束光照到材质表面上，通常会分成**反射**（reflection）部分和**折射**（refraction）部分。反射部分直接从表面反射出去，而不进入物体内部，由此产生了镜面反射光。折射部分会进入物体内部，被吸收或者散射产生漫反射。

折射进物体内部的光如果没有被立即吸收，将会持续前进，与物体内部的微粒产生碰撞，每次碰撞有一部分能量损耗转化成热能，直至光线能量全部消耗。有些折射光线在跟微粒发生若干次碰撞之后，从物体表面射出，便会形成漫反射光。  

![[1679148476642.png]]

  
_照射在平面的光被分成镜面反射和折射光，折射光在跟物体微粒发生若干次碰撞之后，有可能发射出表面，成为漫反射。_

通常情况下，PBR 会简化折射光，将平面上所有折射光都视为被完全吸收而不会散开。而有一些被称为次表面散射 (Subsurface Scattering) 技术的着色器技术会计算折射光散开后的模拟，它们可以显著提升一些材质（如皮肤、大理石或蜡质）的视觉效果，不过性能也会随着下降。

金属 (Metallic) 材质会立即吸收所有折射光，故而金属只有镜面反射，而没有折射光引起的漫反射。

回到能量守恒话题。反射光与折射光它们二者之间是互斥的，被表面反射出去的光无法再被材质吸收。故而，进入材质内部的折射光就是入射光减去反射光后余下的能量。

根据上面的能量守恒关系，可以先计算镜面反射部分，此部分等于入射光线被反射的能量所占的百分比。而折射部分可以由镜面反射部分计算得出。

```c
float kS = calculateSpecularComponent(...); // 反射/镜面部分
float kD = 1.0 - kS;                        // 折射/漫反射部分
```

通过以上代码可以看出，镜面反射部分与漫反射部分的和肯定不会超过 1.0，从而近似达到能量守恒的目的。

### **3.1.3 反射方程（Reflectance Equation）**

**渲染方程** (Render Equation) 是用来模拟光的视觉效果最好的模型。而 PBR 的渲染方程是用以抽象地描述 PBR 光照计算过程的特化版本的渲染方程，被称为**反射方程**。

PBR 的反射方程可抽象成下面的形式：

$$L_o(p,\omega_o) = \int\limits_{\Omega} f_r(p,\omega_i,\omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

反射方程看似很复杂，但如果拆分各个部分加以解析，就可以揭开其神秘的面纱。

为了更好地理解反射方程，先了解**辐射度量学 (Radiometry)**。辐射度量学是一种用来度量电磁场辐射（包括可见光）的手段。有很多种辐射度量 (radiometric quantities) 可以用来测量曲面或者某个方向上的光，此处只讨论和反射方程有关的一种量，它就是**辐射率 (Radiance)**，用 $L$来表示。

先用一个表展示辐射度量学涉及的概念、名词、公式等信息，后面会更加详细地介绍。

<table><thead><tr><th>名称</th><th>符号</th><th>单位</th><th>公式</th><th>解析</th></tr></thead><tbody><tr><td><strong>辐射能量</strong> (Radiant energy)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-18-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>Q</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-166" style="width: 1.031em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.803em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.75em, 2.631em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-167"><span class="mi" id="MathJax-Span-168" style="font-family: MathJax_Math-italic;">Q</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.282em; border-left: 0px solid; width: 0px; height: 1.289em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>Q</mi></math></span></span><script type="math/tex" id="MathJax-Element-18">Q</script></span></td><td>焦耳 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-19-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>J</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-169" style="width: 0.803em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.63em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-170"><span class="mi" id="MathJax-Span-171" style="font-family: MathJax_Math-italic;">J<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>J</mi></math></span></span><script type="math/tex" id="MathJax-Element-19">J</script></span>)</td><td>-</td><td>电磁辐射能量</td></tr><tr><td><strong>辐射通量</strong> (Radiant Flux)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-20-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-172" style="width: 0.974em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.69em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-173"><span class="mi" id="MathJax-Span-174" style="font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi></math></span></span><script type="math/tex" id="MathJax-Element-20">\Phi</script></span></td><td>瓦 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-21-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>W</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-175" style="width: 1.374em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.089em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1001.09em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-176"><span class="mi" id="MathJax-Span-177" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>W</mi></math></span></span><script type="math/tex" id="MathJax-Element-21">W</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-22-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi>Q</mi></mrow><mrow><mi>d</mi><mi>t</mi></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-178" style="width: 4.174em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.317em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.089em, 1003.32em, 2.86em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-179"><span class="mi" id="MathJax-Span-180" style="font-family: MathJax_Main;">Φ</span><span class="mo" id="MathJax-Span-181" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-182" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.92em, 4.289em, -999.997em); top: -4.569em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-183"><span class="mi" id="MathJax-Span-184" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-185" style="font-size: 70.7%; font-family: MathJax_Math-italic;">Q</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.63em, 4.174em, -999.997em); top: -3.597em; left: 50%; margin-left: -0.283em;"><span class="mrow" id="MathJax-Span-186"><span class="mi" id="MathJax-Span-187" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-188" style="font-size: 70.7%; font-family: MathJax_Math-italic;">t</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.03em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.031em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.568em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi>Q</mi></mrow><mrow><mi>d</mi><mi>t</mi></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-22">\Phi = \frac{dQ}{dt}</script></span></td><td>单位时间辐射的能量，也叫辐射功率 (Radiant Power) 或通量(Flux)</td></tr><tr><td><strong>辐照度</strong> (Irradiance)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-23-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>E</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-189" style="width: 0.974em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.75em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-190"><span class="mi" id="MathJax-Span-191" style="font-family: MathJax_Math-italic;">E<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>E</mi></math></span></span><script type="math/tex" id="MathJax-Element-23">E</script></span></td><td>瓦 / 平方米 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-24-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><msup><mi>m</mi><mn>2</mn></msup></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-192" style="width: 3.603em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.86em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1002.86em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-193"><span class="texatom" id="MathJax-Span-194"><span class="mrow" id="MathJax-Span-195"><span class="mi" id="MathJax-Span-196" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-197"><span class="mrow" id="MathJax-Span-198"><span class="mo" id="MathJax-Span-199" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-200"><span class="mrow" id="MathJax-Span-201"><span class="msubsup" id="MathJax-Span-202"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-203" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-204" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><msup><mi>m</mi><mn>2</mn></msup></mrow></math></span></span><script type="math/tex" id="MathJax-Element-24">{W}/{m^2}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-25-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-205" style="width: 4.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.774em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1003.77em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-206"><span class="mi" id="MathJax-Span-207" style="font-family: MathJax_Main;">Φ</span><span class="mo" id="MathJax-Span-208" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-209" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.489em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-210"><span class="mi" id="MathJax-Span-211" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-212" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1001.32em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -0.683em;"><span class="mrow" id="MathJax-Span-213"><span class="mi" id="MathJax-Span-214" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-215"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-216" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-217" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.49em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.489em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-25">\Phi = \frac{d\Phi}{dA^\perp}</script></span></td><td><em>到达</em>单位面积的辐射通量</td></tr><tr><td><strong>辐射度</strong> (Radiosity)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-26-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>M</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-218" style="width: 1.317em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1001.03em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-219"><span class="mi" id="MathJax-Span-220" style="font-family: MathJax_Math-italic;">M<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>M</mi></math></span></span><script type="math/tex" id="MathJax-Element-26">M</script></span></td><td>瓦 / 平方米 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-27-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><msup><mi>m</mi><mn>2</mn></msup></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-221" style="width: 3.603em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.86em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1002.86em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-222"><span class="texatom" id="MathJax-Span-223"><span class="mrow" id="MathJax-Span-224"><span class="mi" id="MathJax-Span-225" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-226"><span class="mrow" id="MathJax-Span-227"><span class="mo" id="MathJax-Span-228" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-229"><span class="mrow" id="MathJax-Span-230"><span class="msubsup" id="MathJax-Span-231"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-232" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-233" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><msup><mi>m</mi><mn>2</mn></msup></mrow></math></span></span><script type="math/tex" id="MathJax-Element-27">{W}/{m^2}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-28-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>M</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-234" style="width: 5.146em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.117em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1004.12em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-235"><span class="mi" id="MathJax-Span-236" style="font-family: MathJax_Math-italic;">M<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mo" id="MathJax-Span-237" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-238" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.489em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-239"><span class="mi" id="MathJax-Span-240" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-241" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1001.32em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -0.683em;"><span class="mrow" id="MathJax-Span-242"><span class="mi" id="MathJax-Span-243" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-244"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-245" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-246" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.49em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.489em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>M</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-28">M = \frac{d\Phi}{dA^\perp}</script></span></td><td><em>离开</em>单位面积的辐射通量，也叫辐出度、辐射出射度（Radiant Existance）</td></tr><tr><td><strong>辐射强度</strong> (Radiant Intensity)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-29-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>I</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-247" style="width: 0.689em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.517em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.52em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-248"><span class="mi" id="MathJax-Span-249" style="font-family: MathJax_Math-italic;">I<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>I</mi></math></span></span><script type="math/tex" id="MathJax-Element-29">I</script></span></td><td>瓦 / 立体弧度 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-30-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>s</mi><mi>r</mi></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-250" style="width: 3.089em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.46em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1002.46em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-251"><span class="texatom" id="MathJax-Span-252"><span class="mrow" id="MathJax-Span-253"><span class="mi" id="MathJax-Span-254" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-255"><span class="mrow" id="MathJax-Span-256"><span class="mo" id="MathJax-Span-257" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-258"><span class="mrow" id="MathJax-Span-259"><span class="mi" id="MathJax-Span-260" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-261" style="font-family: MathJax_Math-italic;">r</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><mi>s</mi><mi>r</mi></mrow></math></span></span><script type="math/tex" id="MathJax-Element-30">{W}/{sr}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-31-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>I</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><mi>&amp;#x03C9;</mi></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-262" style="width: 3.946em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.146em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1003.15em, 2.86em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-263"><span class="mi" id="MathJax-Span-264" style="font-family: MathJax_Math-italic;">I<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mo" id="MathJax-Span-265" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-266" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-267"><span class="mi" id="MathJax-Span-268" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-269" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.8em, 4.174em, -999.997em); top: -3.597em; left: 50%; margin-left: -0.397em;"><span class="mrow" id="MathJax-Span-270"><span class="mi" id="MathJax-Span-271" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-272" style="font-size: 70.7%; font-family: MathJax_Math-italic;">ω</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.03em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.031em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.568em; border-left: 0px solid; width: 0px; height: 1.789em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>I</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><mi>ω</mi></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-31">I = \frac{d\Phi}{d\omega}</script></span></td><td>通过单位立体角的辐射通量</td></tr><tr><td><strong>辐射率</strong> (Radiance)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-32-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>L</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-273" style="width: 0.86em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.63em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-274"><span class="mi" id="MathJax-Span-275" style="font-family: MathJax_Math-italic;">L</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>L</mi></math></span></span><script type="math/tex" id="MathJax-Element-32">L</script></span></td><td>瓦 / 平方米立体弧度 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-33-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><msup><mi>m</mi><mn>2</mn></msup><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>s</mi><mi>r</mi></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-276" style="width: 4.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.774em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1003.77em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-277"><span class="texatom" id="MathJax-Span-278"><span class="mrow" id="MathJax-Span-279"><span class="mi" id="MathJax-Span-280" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-281"><span class="mrow" id="MathJax-Span-282"><span class="mo" id="MathJax-Span-283" style="font-family: MathJax_Main;">/</span></span></span><span class="msubsup" id="MathJax-Span-284"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-285" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-286" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="texatom" id="MathJax-Span-287"><span class="mrow" id="MathJax-Span-288"><span class="mi" id="MathJax-Span-289" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-290" style="font-family: MathJax_Math-italic;">r</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><msup><mi>m</mi><mn>2</mn></msup><mrow class="MJX-TeXAtom-ORD"><mi>s</mi><mi>r</mi></mrow></math></span></span><script type="math/tex" id="MathJax-Element-33">{W}/m^2{sr}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-34-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>L</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><mi>&amp;#x03C9;</mi><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-291" style="width: 5.717em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.574em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1004.57em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-292"><span class="mi" id="MathJax-Span-293" style="font-family: MathJax_Math-italic;">L</span><span class="mo" id="MathJax-Span-294" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-295" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 2.289em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-296"><span class="mi" id="MathJax-Span-297" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-298" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1002.17em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -1.083em;"><span class="mrow" id="MathJax-Span-299"><span class="mi" id="MathJax-Span-300" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-301" style="font-size: 70.7%; font-family: MathJax_Math-italic;">ω</span><span class="mi" id="MathJax-Span-302" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-303"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-304" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-305" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.29em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.289em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>L</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><mi>ω</mi><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-34">L = \frac{d\Phi}{d\omega dA^\perp}</script></span></td><td>通过单位面积单位立体角的辐射通量</td></tr><tr><td><strong>立体角</strong> (Solid Angle)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-35-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>&amp;#x03C9;</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-306" style="width: 0.803em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.66em, 1000.63em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-307"><span class="mi" id="MathJax-Span-308" style="font-family: MathJax_Math-italic;">ω</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 0.718em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>ω</mi></math></span></span><script type="math/tex" id="MathJax-Element-35">\omega</script></span></td><td>立体弧度，球面度（<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-36-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>s</mi><mi>r</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-309" style="width: 1.146em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.917em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.66em, 1000.92em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-310"><span class="mi" id="MathJax-Span-311" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-312" style="font-family: MathJax_Math-italic;">r</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 0.718em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>s</mi><mi>r</mi></math></span></span><script type="math/tex" id="MathJax-Element-36">sr</script></span>）</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-37-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>&amp;#x03C9;</mi><mo>=</mo><mfrac><mi>S</mi><msup><mi>r</mi><mn>2</mn></msup></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-313" style="width: 3.717em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.974em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1002.97em, 2.917em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-314"><span class="mi" id="MathJax-Span-315" style="font-family: MathJax_Math-italic;">ω</span><span class="mo" id="MathJax-Span-316" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-317" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.46em, 4.174em, -999.997em); top: -4.454em; left: 50%; margin-left: -0.226em;"><span class="mi" id="MathJax-Span-318" style="font-size: 70.7%; font-family: MathJax_Math-italic;">S<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.63em, 4.174em, -999.997em); top: -3.54em; left: 50%; margin-left: -0.283em;"><span class="msubsup" id="MathJax-Span-319"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.29em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-320" style="font-size: 70.7%; font-family: MathJax_Math-italic;">r</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.346em;"><span class="mn" id="MathJax-Span-321" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1000.75em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 0.746em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.639em; border-left: 0px solid; width: 0px; height: 1.861em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>ω</mi><mo>=</mo><mfrac><mi>S</mi><msup><mi>r</mi><mn>2</mn></msup></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-37">\omega=\frac{S}{r^2}</script></span></td><td>是二维弧度在三维的扩展，1 球面度等于单位球体的表面面积</td></tr></tbody></table>

辐射率被用来量化单一方向上发射来的光线的大小或者强度。辐射率是由多个物理变量集合而成的，它涉及的物理变量有以下几种：

*   **辐射通量 (Radiant Flux)**：辐射通量用符号$\Phi$表示，表示一个光源输出的能量，以瓦特为单位。光是由多种不同波长的能量集合而成，每种波长与一种特定的（可见的）颜色相关。因此一个光源所放射出来的能量可以被视作这个光源包含的所有各种波长的一个函数。波长介于 390nm（纳米）到 700nm 的光被认为是处于可见光光谱中，也就是说它们是人眼可见的波长。
    
    ![[1679148476662.png]]
    
      
    _上图展示了太阳光中不同波长的光所具有的能量。_  
    传统物理学上的辐射通量将会计算这个由不同波长构成的函数的总面积，这种计算很复杂，耗费大量性能。在 PBR 技术中，不直接使用波长的强度，而是使用三原色编码（RGB）来简化辐射通量的计算。虽然这种简化会带来一些信息上的损失，但是这对于视觉效果上的影响基本可以忽略。
    
*   **立体角 (Solid Angle)**：用符号$\omega$表示，它描述投射到单位球体上的一个截面的大小或者面积。可以把立体角想象成为一个带有体积的方向：  
    
    ![[1679148476688.png]]
    
      
    更加形象地描述：观察者站在单位球面的中心，向着投影的方向看，在单位球体面上的投影轮廓的大小就是立体角。
    
*   **辐射强度 (Radiant Intensity)**：用符号 $I$表示，它描述的是在单位球面上，一个光源向每单位立体角所投送的辐射通量。举个例子，假设一个点光源向所有方向均匀地辐射能量，辐射强度就能计算出它在一个单位面积（立体角）内的能量大小：  
    
    ![[1679148476720.png]]
    
      
    计算辐射强度的公式：
    

$$I = \frac{d\Phi}{d\omega}$$

其中 $I$表示辐射通量$\Phi$除以立体角$\omega$的辐射强度。

理解以上物理变量后，可以继续讨论辐射率方程了。下面方程代表的意义是：一个辐射强度为$\Phi$的光通过立体角$\omega$辐射在区域 $A$的可被观察到的总能量。

$$L=\frac{I}{dA^\perp}=\frac{\frac{d\Phi}{d\omega}}{dA\cos\theta}=\frac{d\Phi}{ dA d\omega \cos\theta}$$

笔者注：原文的公式是 $L = \frac{d^2\Phi}{ dA d\omega \cos\theta}$，经推导之后，并没有平方。​

![[1679148476742.png]]

辐射率是一个区域内光照量的辐射学度量，按照光的入射（或者来源）角与平面法线的夹角$\theta$计算 $\cos \theta$。越是斜着照射在平面上光越弱，反之越是垂直照射在表面上的光越强，类似基础光照中的漫反射颜色计算，$\cos \theta$直接等于光的方向和表面法线的点积。

```
float cosTheta = dot(lightDir, N);
```

上面的物理符号似乎和 PBR 的反射方程没有直接的关系。但是，如果将立体角$\omega$跟区域 $A$都看作无限小，就可以使用辐射率来分析一束光线打在空间上一个点的通量，也就是说能够计算单束光线对单个（片元）点的辐射率影响。进一步地，将立体角$\omega$转化为方向向量$\omega$，将区域 $A$转化成点 $p$，因此在 shader 中直接使用辐射率来计算单束光线对每个片元的贡献。

实际上，当谈及光的辐射率时，通常只关注的是所有射入点 $p$的光线，这些光的辐射度总和称为**辐照度 (Irradiance)**。理解了辐射率和辐照度，回到反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} f_r(p,\omega_i,\omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

渲染方程式中 $L$代表某个点 $p$的辐射率，而无限小的入射光的立体角$\omega_i$可以看作入射光方向向量$\omega_i$，将用来衡量入射光与平面法线夹角对能量的影响的 $\cos \theta$分量移出辐射率方程，作为反射方程的单独项 $n \cdot \omega_i$ 。

反射方程计算了点 $p$在所有视线方向$\omega_0$上被反射出来的辐射率 $L_o(p,\omega_o)$的总和。换言之：$L_0$计算的是在$\omega_o$方向的眼睛观察到的 $p$点的总辐照度。

反射方程里面使用的辐照度，必须要包含所有以 $p$点为中心的半球$\Omega$内的入射光，而不单单只是某一个方向的入射光。这个半球指的是围绕面法线 $n$的那一个半球：  

![[1679148476772.png]]

笔者注：为什么只计算半球而不计算整个球体呢？

因为另外一边的半球因与视线方向相反，不能被观察，也就是辐射通量贡献量为 0，所以被忽略。

为了计算这个区域（半球）内的所有值，在反射方程中使用了一个称作为积分的数学符号 $\int$，来计算半球$\Omega$内所有的入射向量 $d\omega_i$。

积分计算面积的方法，有**解析 (analytically)** 和**渐近 (numerically)** 两种方法。目前尚没有可以满足渲染计算的解析法，所以只能选择离散渐近法来解决这个积分问题。

具体做法是在半球$\Omega$按一定的步长将反射方程离散地求解，然后再按照步长大小将所得到的结果平均化，这种方法被称为**黎曼和 (Riemann sum)**。下面是实现的伪代码：

```
int steps = 100; // 分段计算的数量，数量越多，计算结果越准确。
float dW  = 1.0f / steps;
vec3 P    = ...;
vec3 Wo   = ...;
vec3 N    = ...;
float sum = 0.0f;
for(int i = 0; i < steps; ++i) 
{
    vec3 Wi = getNextIncomingLightDir(i);
    sum += Fr(P, Wi, Wo) * L(P, Wi) * dot(N, Wi) * dW;
}
```

`dW`的值越小结果越接近正确的积分函数的面积或者说体积，衡量离散步长的`dW`可以看作反射方程中的 $d\omega_i$。积分计算中我们用到的 $d\omega_i$是线性连续的符号，跟代码中的`dW`并没有直接关系，但是这种方式有助于我们理解，而且这种离散渐近的计算方法总是可以得到一个很接近正确结果的值。值得一提的是，通过增加步骤数`steps`可以提高黎曼和的准确性，但计算量也会增大。

反射方程加了所有的，以各个方向$\omega_i$射入半球$\Omega$并打中点 $p$的入射光，经过反射函数 $f_r$进入观察者眼睛的所有反射光 $L_o$的辐射率之和。入射光辐射度可以由光源处获得，此外还可以利用一个环境贴图来测算所有入射方向上的辐射度。

至此，反射方程中，只剩下 $f_r$项未描述。$f_r$就是**双向反射分布函数** (Bidirectional Reflectance Distribution Function, BRDF)，它的作用是基于表面材质属性来对入射辐射度进行缩放或者加权。

### **3.1.4 双向反射分布函数（BRDF）**

**双向反射分布函数**（Bidirectional Reflectance Distribution Function，BRDF）是一个使用入射光方向$\omega_i$作为输入参数的函数，输出参数为出射光$\omega_o$，表面法线为 $n$，参数 $a$表示的是微平面的粗糙度。

BRDF 函数是近似的计算在一个给定了属性的不透明表面上每个单独的光线对最终的反射光的贡献量。假如表面是绝对光滑的（比如镜子），对于所有入射光$\omega_i$的 BRDF 函数都将会返回 0.0，除非出射光线$\omega_o$方向的角度跟入射光线$\omega_i$方向的角度以面法线为中轴线完全对称，则返回 1.0。

BRDF 对于材质的反射和折射属性的模拟基于之前讨论过的微平面理论，想要 BRDF 在物理上是合理的，就必须遵守能量守恒定律。比如反射光能量总和永远不应该超过入射光。技术上来说，Blinn-Phong 光照模型跟 BRDF 一样使用了$\omega_i$跟$\omega_o$作为输入参数，但是没有像基于物理的渲染这样严格地遵守能量守恒定律。

BRDF 有好几种模拟表面光照的算法，然而，基本上所有的**实时**渲染管线使用的都是 **Cook-Torrance BRDF**。

Cook-Torrance BRDF 分为漫反射和镜面反射两个部分：

$$f_r = k_d f_{lambert} + k_s f_{cook-torrance}$$

其中 $k_d$是入射光中被折射的比例，$k_s$是另外一部分被镜面反射的入射光。BRDF 等式左边的 $f_{lambert}$表示的是漫反射部分，这部分叫做伦勃朗漫反射（Lambertian Diffuse）。它类似于我们之前的漫反射着色，是一个恒定的算式：

$$f_{lambert} = \frac{c}{\pi}$$

其中 $c$代表的是 Albedo 或表面颜色，类似漫反射表面纹理。除以$\pi$是为了规格化漫反射光，为后期的 BRDF 积分做准备。

此处的伦勃朗漫反射跟以前用的漫反射之间的关系：以前的漫反射是用表面的漫反射颜色乘以法线与面法线的点积，这个点积依然存在，只不过是被移到了 BRDF 外面，写作 $n \cdot \omega_i$，放在反射方程 $L_o$靠后的位置。

BRDF 的高光（镜面反射）部分更复杂：

$$f_{cook-torrance} = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

Cook-Torrance 镜面反射 BRDF 由 3 个函数（$D$，$F$，$G$）和一个标准化因子构成。$D$，$F$，$G$符号各自近似模拟了特定部分的表面反射属性：

*   **$D$(Normal _D_istribution Function，NDF)**：法线分布函数，估算在受到表面粗糙度的影响下，取向方向与中间向量一致的微平面的数量。这是用来估算微平面的主要函数。
*   **$F$(_F_resnel equation)**：菲涅尔方程，描述的是在不同的表面角下表面反射的光线所占的比率。
*   **$G$(_G_eometry function)**：几何函数，描述了微平面自成阴影的属性。当一个平面相对比较粗糙的时候，平面表面上的微平面有可能挡住其他的微平面从而减少表面所反射的光线。

以上的每一种函数都是用来估算相应的物理参数的，而且你会发现用来实现相应物理机制的每种函数都有不止一种形式。它们有的非常真实，有的则性能高效。你可以按照自己的需求任意选择自己想要的函数的实现方法。

Epic Games 公司的 Brian Karis 对于这些函数的多种近似实现方式进行了大量的研究。这里将采用 Epic Games 在 Unreal Engine 4 中所使用的函数，其中 $D$使用 Trowbridge-Reitz GGX，$F$使用 Fresnel-Schlick 近似法 (Approximation)，而 $G$使用 Smith's Schlick-GGX。

#### **3.1.4.1 $D$(Normal _D_istribution Function，NDF)**

法线分布函数，从统计学上近似的表示了与某些（如中间）向量 $h$取向一致的微平面的比率。

目前有很多种 NDF 都可以从统计学上来估算微平面的总体取向度，只要给定一些粗糙度的参数以及一个我们马上将会要用到的参数 Trowbridge-Reitz GGX（GGXTR）：

$$NDF_{GGX TR}(n, h, \alpha) = \frac{\alpha^2}{\pi((n \cdot h)^2 (\alpha^2 - 1) + 1)^2}$$

这里的 $h$是用来测量微平面的半角向量，$\alpha$是表面的粗糙度，$n$是表面法线。 如果将 $h$放到表面法线和光线方向之间，并使用不同的粗糙度作为参数，可以得到下面的效果：  

![[1679148476814.png]]

当粗糙度很低（表面很光滑）时，与中间向量 $h$取向一致的微平面会高度集中在一个很小的半径范围内。由于这种集中性，NDF 最终会生成一个非常明亮的斑点。但是当表面比较粗糙的时候，微平面的取向方向会更加的随机，与向量 $h$取向一致的微平面分布在一个大得多的半径范围内，但是较低的集中性也会让最终效果显得更加灰暗。

Trowbridge-Reitz GGX 的 NDF 实现代码：

```
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

#### **3.1.4.2 $F$(_F_resnel equation)**

菲涅尔方程定义的是在不同观察方向上，表面上被反射的光除以被折射的光的比例。在一束光击中了表面的一瞬间，菲涅尔根据表面与观察方向之间的夹角，计算得到光被反射的百分比。根据这个比例和能量守恒定律我们可以直接知道剩余的能量就是会被折射的能量。

当我们垂直观察每个表面或者材质时都有一个基础反射率，当我们以任意一个角度观察表面时所有的反射现象都会变得更明显（反射率高于基础反射率）。你可以从你身边的任意一件物体上观察到这个现象，当你以 90 度角观察你的桌子你会法线反射现象将会变得更加的明显，理论上以完美的 90 度观察任意材质的表面都应该会出现全反射现象（所有物体、材质都有菲涅尔现象）。

菲涅尔方程同样是个复杂的方程，但是幸运的是菲涅尔方程可以使用 Fresnel-Schlick 来近似：

$$F_{Schlick}(h, v, F_0) = F_0 + (1 - F_0) ( 1 - (h \cdot v))^5$$

$F_0$表示的是表面基础反射率，这个我们可以使用一种叫做 **Indices of refraction(IOR)** 的方法计算得到。运用在球面上的效果就是你看到的那样，观察方向越是接近**掠射角**（grazing angle，又叫切线角，与正视角相差 90 度），菲涅尔现象导致的反射就越强：  

![[1679148476836.png]]

菲涅尔方程中有几个微妙的地方，一个是 Fresnel-Schlick 算法仅仅是为电介质（绝缘体）表面定义的算法。对于金属表面，使用电介质的折射率来计算基础反射率是不合适的，我们需要用别的菲涅尔方程来计算。对于这个问题，我们需要预先计算表面在正视角 (即以 0 度角正视表面) 下的反应（$F_0$），然后就可以跟之前的 Fresnel-Schlick 算法一样，根据观察角度来进行插值。这样我们就可以用一个方程同时计算金属和电介质了。

表面在正视角下的反映或者说基础反射率可以在这个数据库中找到，下面是 Naty Hoffman 的在 SIGGRAPH 公开课中列举的一些常见材质的值：  

![[1679148476872.png]]

这里可以观察到的一个有趣的现象，所有电介质材质表面的基础反射率都不会高于 0.17，这其实是例外而非普遍情况。导体材质表面的基础反射率起点更高一些并且（大多）在 0.5 和 1.0 之间变化。此外，对于导体或者金属表面而言基础反射率一般是带有色彩的，这也是为什么要用 RGB 三原色来表示的原因（法向入射的反射率可随波长不同而不同）。这种现象我们只能在金属表面观察的到。

金属表面这些和电介质表面相比所独有的特性引出了所谓的金属工作流的概念。也就是我们需要额外使用一个被称为金属度 (Metalness) 的参数来参与编写表面材质。金属度用来描述一个材质表面是金属还是非金属的。

通过预先计算电介质与导体的值，我们可以对两种类型的表面使用相同的 Fresnel-Schlick 近似，但是如果是金属表面的话就需要对基础反射率添加色彩。我们一般是按下面这个样子来实现的：

```
vec3 F0 = vec3(0.04);
F0      = mix(F0, surfaceColor.rgb, metalness);
```

我们为大多数电介质表面定义了一个近似的基础反射率。$F_0$取最常见的电解质表面的平均值，这又是一个近似值。不过对于大多数电介质表面而言使用 0.04 作为基础反射率已经足够好了，而且可以在不需要输入额外表面参数的情况下得到物理可信的结果。然后，基于金属表面特性，我们要么使用电介质的基础反射率要么就使用 $F_0$作来为表面颜色。因为金属表面会吸收所有折射光线而没有漫反射，所以我们可以直接使用表面颜色纹理来作为它们的基础反射率。

Fresnel Schlick 近似可以用 GLSL 代码实现：

```
vec3 fresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
```

其中`cosTheta`是表面法向量 $n$与观察方向 $v$的点乘的结果。

#### **3.1.4.3 $G$(_G_eometry function)**

几何函数模拟微平面相互遮挡导致光线的能量减少或丢失的现象。  

![[1679148476912.png]]

类似 NDF，几何函数也使用粗糙度作为输入参数，更粗糙意味着微平面产生自阴影的概率更高。几何函数使用由 GGX 和 Schlick-Beckmann 组合而成的模拟函数 Schlick-GGX：

$$G_{SchlickGGX}(n, v, k) = \frac{n \cdot v} {(n \cdot v)(1 - k) + k }$$

这里的 $k$是使用粗糙度$\alpha$计算而来的，用于直接光照和 IBL 光照的几何函数的参数：

$$\begin{eqnarray*} k_{direct} &=& \frac{(\alpha + 1)^2}{8} \\ k_{IBL} &=& \frac{\alpha^2}{2} \end{eqnarray*}$$

需要注意的是这里$\alpha$的值取决于你的引擎怎么将粗糙度转化成$\alpha$，在接下来的教程中我们将会进一步讨论如何和在什么地方进行这个转换。

为了有效地模拟几何体，我们需要同时考虑两个视角，视线方向（几何遮挡）跟光线方向（几何阴影），我们可以用 **Smith 函数**将两部分放到一起：

$$G(n, v, l, k) = G_{sub}(n, v, k) G_{sub}(n, l, k)$$

其中 $v$表示视线向量，$G_{sub}(n, v, k)$表示视线方向的几何遮挡；$l$表示光线向量，$G_{sub}(n, l, k)$表示光线方向的几何阴影。使用 Smith 函数与 Schlick-GGX 作为 $G_{sub}$可以得到如下所示不同粗糙度 R 的视觉效果：  

![[1679148477023.png]]

几何函数是一个值域为 [0.0, 1.0] 的乘数，其中白色 (1.0) 表示没有微平面阴影，而黑色 (0.0) 则表示微平面彻底被遮蔽。

使用 GLSL 编写的几何函数代码如下：

```
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

#### **3.1.4.4 Cook-Torrance 反射方程 (Cook-Torrance reflectance equation)**

Cook-Torrance 反射方程中的每一个部分我们我们都用基于物理的 BRDF 替换，可以得到最终的反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

上面的方程并非完全数学意义上的正确。前面提到菲涅尔项 $F$代表光在表面的反射比率，它直接影响 $k_s$因子，意味着反射方程的镜面反射部分已经隐含了因子 $k_s$。因此，最终的 Cook-Torrance 反射方程如下（去掉了 $k_s$）：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

这个方程完整地定义了一个基于物理的渲染模型，也就是我们一般所说的基于物理的渲染（PBR）。

### **3.1.5 制作 PBR 材质**

对 PBR 数学模型有了基本了解之后，我们最后要讨论的是美工应该生成怎样的材质属性，让我们可以直接用在 PBR 渲染管线里。PBR 管线中需要的所有材质参数都可以使用纹理来定义或者模拟，使用纹理我们可以逐像素控制制定的面如何跟光线交互：这个点是否是金属，粗糙度如何又或者表面对不同波长的光有什么反映。

下面是在 PBR 渲染管线中经常用到的纹理：  

![[1679148477062.png]]

下面的参数跟 [[#2-4-pbr在游戏引擎的应用|**2.4 PBR 在游戏引擎的应用**]]描述的很多参数基本一致。

*   **反射率**（Albedo）：反射率纹理指定了材质表面每个像素的颜色，如果材质是金属那纹理包含的就是基础反射率。这个跟我们之前用过的漫反射纹理非常的类似，但是不包含任何光照信息。漫反射纹理通常会有轻微的阴影和较暗的裂缝，这些在 Albedo 贴图里面都不应该出现，仅仅只包含材质的颜色（金属材质是基础反射率）。
    
*   **法线**（Normal）：法线纹理跟我们之前使用的是完全一样的。法线贴图可以逐像素指定表面法线，让平坦的表面也能渲染出凹凸不平的视觉效果。
    
*   **金属度**（Metallic）：金属度贴图逐像素的指定表面是金属还是电介质。根据 PBR 引擎各自的设定，金属程度即可以是 [0.0，1.0] 区间的浮点值也可以是非 0 即 1 的布尔值。
    
*   **粗糙度**（Roughness）：粗糙度贴图逐像素的指定了表面有多粗糙，粗糙度的值影响了材质表面的微平面的平均朝向，粗糙的表面上反射效果更大更模糊，光滑的表面更亮更清晰。有些 PBR 引擎用光滑度贴图替代粗糙度贴图，因为他们觉得光滑度贴图更直观，将采样出来的光滑度使用（1 - 光滑度）= 粗糙度 就能转换成粗糙度了。
    
*   **环境光遮挡**（Ambient Occlusion，AO）：AO 贴图为材质表面和几何体周边可能的位置，提供了额外的阴影效果。比如有一面砖墙，在两块砖之间的缝隙里 Albedo 贴图包含的应该是没有阴影的颜色信息，而让 AO 贴图来指定这一块需要更暗一些，这个地方光线更难照射到。AO 贴图在光照计算的最后一步使用可以显著的提高渲染效果，模型或者材质的 AO 贴图一般是在建模阶段手动生成的。
    

美术可以直接根据物体在真实世界里的物理属性，来设置和调整用于渲染的基于物理的材质。

基于物理的渲染管线最大的优势在于，材质的物理属性是不变的，无论环境光怎么样设置都能得到一个接近真实的渲染结果，这让美术的人生都变得美好了。

基于物理管线的材质可以很简单的移植到不同的渲染引擎，不管光照环境如何都能正确的渲染出一个自然的结果。

## **3.2 PBR 的光照实现**

3.1 章节阐述了 Cook-Torrance 反射方程的理论和公式意义。这节将探讨如何将前面讲到的理论转化成一个基于直接光照的渲染器：比如点光源，方向光和聚光灯。

### **3.2.1 辐照度计算**

3.1 章节解释了 Cook-Torrance 反射方程的大部分含义，但有一点未提及：具体要怎么处理场景中的辐照度（Irradiance，也就是辐射的总能量 $L$）？在计算机领域，场景的辐射率 $L$度量的是来自光源光线的辐射通量$\phi$穿过指定的立体角$\omega$，在这里我们假设立体角$\omega$无限小，小到辐射度衡量的是光源射出的一束经过指定方向向量的光线的通量。

有了这个假设，我们又要怎么将之融合到之前教程讲的光照计算里去呢？想象我们有一个辐射通量以 RGB 表示为（23.47, 21.31, 20.79）的点光源，这个光源的辐射强度等于辐射通量除以所有出射方向。当为平面上某个特定的点 $p$着色的时候，所有可能的入射光方向都会经过半球$\Omega$，但只有一个入射方向$\omega_i$是直接来自点光源的，又因为我们的场景中只包含有一个光源，且这个光源只是一个点，所以 $p$点所有其它的入射光方向的辐射率都应该是 0.  

![[1679148477087.png]]

如果我们暂时不考虑点光源的距离衰减问题，且无论光源放在什么地方入射光线的辐射率都一样大（忽略入射光角度 $\cos \theta$对辐射度的影响），又因为点光源朝各个方向的辐射强度都是一样的，那么有效的辐射强度就跟辐射通量完全一样：恒定值（23.47, 21.31, 20.79）。

然而，辐射率需要使用位置 $p$作为输入参数，因为现实中的灯光根据点 $p$和光源之间距离的不同，辐射强度多少都会有一定的衰减。另外，从原始的辐射方程中我们可以发现，面法线 $n$于入射光方向向量$\omega_i$的点积也会影响结果。

用更精炼的话来描述：在点光源直接光照的情况里，辐射率函数 $L$计算的是灯光颜色，经过到 $p$点距离的衰减之后，再经过 $n \cdot \omega_i$缩放。能击中点 $p$的光线方向$\omega_i$就是从 $p$点看向光源的方向。把这些写成代码：

```
vec3  lightColor  = vec3(23.47, 21.31, 20.79);
vec3  wi          = normalize(lightPos - fragPos);
float cosTheta    = max(dot(N, Wi), 0.0);
// 计算光源在点fragPos的衰减系数
float attenuation = calculateAttenuation(fragPos, lightPos); 
// 英文原版的radiance类型有误，将它改成了vec3
vec3 radiance  = lightColor * (attenuation * cosTheta);
```

你应该非常非常熟悉这段代码：这就是以前我们计算漫反射光的算法！在只有单光源直接光照的情况下，辐射率的计算方法跟我们以前的光照算法是类似的。

要注意我们这里假设点光源无限小，只是空间中的一个点。如果我们使用有体积的光源模型，那么就有很多的入射光方向的辐射率是非 0 的。

对那些基于点的其他类型光源我们可以用类似的方法计算辐射率，比如平行光源的入射角的恒定的且没有衰减因子，聚光灯没有一个固定的辐射强度，而是围绕一个正前方向量来进行缩放的。

这也将我们带回了在表面半球$\Omega$的积分$\int$。我们知道，多个单一位置的光源对同一个表面的同一个点进行光照着色并不需要用到积分，我们可以直接拿出这些数目已知的光源来，分别计算这些光源的辐照度后再加到一起，毕竟每个光源只有一束方向光能影响物体表面的辐射率。这样只需要通过相对简单的循环计算每个光源的贡献就能完成整个 PBR 光照计算。当我们需要使用 IBL 将环境光加入计算的时候我们才会需要用到积分，因为环境光可能来自任何方向。

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
vec3 fresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
```

Fresnel-Schlick 算法需要的`F0`参数就是我们之前说的基础反射率，即以 0 度角照射在表面上的光被反射的比例。不同材质的`F0`的值都不一样，可以根据材质到那张非常大的材质表里去找。在 PBR 金属度流水线中我们做了一个简单的假设，我们认为大部分的电介质表面的`F0`用 0.04 效果看起来很不错。而金属表面我们将`F0`放到 albedo 纹理内，这些可以写成代码如下：

```
vec3 F0 = vec3(0.04); 
F0      = mix(F0, albedo, metallic);
vec3 F  = fresnelSchlick(max(dot(H, V), 0.0), F0);
```

如上述代码所见，非金属的`F0`永远是 0.04，除非我们通过金属度属性在`F0`跟`albedo`之间进行线性插值，才能得到一个不同的非金属`F0`。

有了`F`，还剩下法线分布函数 $D$跟几何函数 $G$需要计算。

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

`denominator`项里的`0.001`是为了防止除 0 情况而特意加上的。

到这里，我们终于可以计算每个光源对反射方程的贡献了。因为菲涅尔值相当于 $k_S$，可用`F`代表任意光击中表面后被反射的部分，根据能量守恒定律我们可以用 $k_S$直接计算得到 $k_D$：

```
vec3 kS = F;
vec3 kD = vec3(1.0) - kS;
  
kD *= 1.0 - metallic; // 由于金属表面不折射光，没有漫反射颜色，通过归零kD来实现这个规则
```

$k_S$表示的是光能有多少被反射了，剩下的被折射的光能我们用 $k_D$来表示。此外，由于金属表面不折射光，因此没有漫反射颜色，我们通过归零 $k_D$来实现这个规则。

有了这些数据，我们终于可以算出每个光源的出射光了：

```
const float PI = 3.14159265359;
  
    float NdotL = max(dot(N, L), 0.0);        
    Lo += (kD * albedo / PI + specular) * radiance * NdotL;
}
```

最终结果`Lo`，或者说出射辐射度（Radiosity），实际上是反射方程在半球$\Omega$的积分$\int$的结果。这里要特别注意的是，我们将 $k_S$移除方程式，是因为我们已经在 BRDF 中乘过菲涅尔参数`F`了，此处不需要再乘一次。

我们没有真正的对所有可能的入射光方向进行积分，因为我们已经清楚的知道只有 4 个入射方向可以影响这个片元，所以我们只需要直接用循环处理这些入射光就行了。

剩下的就是要将 AO 运用到光照结果`Lo`上，我们就可以得到这个片元的最终颜色了：

```
vec3 ambient = vec3(0.03) * albedo * ao;
vec3 color   = ambient + Lo;
```

#### **3.2.2.2 线性和 HDR 渲染（ Linear and HDR rendering）**

以上我们假设所有计算都在线性空间，为了使用这个结果我们还需要在着色器的最后进行**伽马校正（Gamma Correct）**，在线性空间计算光照对于 PBR 是非常非常重要的，所有输入参数同样要求是线性的，不考虑这一点将会得到错误的光照结果。

另外，我们希望输入的灯光参数更贴近实际的物理参数，比如他们的辐射度或者颜色值可以是一个非常宽广的值域。这样作为结果输出的`Lo`也将变得很大，如果我们不做处理默认会直接 Clamp 到 0.0 至 1.0 之间以适配低动态范围（LDR）输出方式。

为了有效解决`Lo`的值域问题，我们可以使用色调映射（Tone Map）和曝光控制（Exposure Map），用它们将`Lo`的高动态范围（HDR）映射到 LDR 之后再做伽马校正：

```
color = color / (color + vec3(1.0)); // 色调映射
color = pow(color, vec3(1.0/2.2)); 	 // 伽马校正
```

这里我们使用的是莱因哈特算法（Reinhard operator）对 HDR 进行 Tone Map 操作，尽量在伽马矫正之后还保持高动态范围。我们并没有分开帧缓冲或者使用后处理，所以我们可以直接将 Tone Mapping 和伽马矫正放在前向片元着色器（forward fragment shader）。  

![[1679148477347.png]]

对于 PBR 渲染管线来说，线性空间跟高动态范围有着超乎寻常的重要性，没有这些就不可能绘制出不同灯光强度下的高光低光细节，错误的计算结果会产生难看的渲染效果。

#### **3.2.2.3 完整的 PBR 直接光照着色器**

现在唯一剩下的就是将最终的色调映射和伽玛校正的颜色传递给片元着色器的输出通道，我们就拥有了一个 PBR 直接光照着色器。基于完整性考虑，下面列出完整的`main`函数：

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
    vec3 F0 = vec3(0.04); 
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
        float G   = GeometrySmith(N, V, L, roughness);      
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

![[1679148477380.png]]

从下往上金属度的值从 0.0 到 1.0，粗糙度从左往右从 0.0 增加到 1.0。可以通过观察小球之间的区别理解金属度和粗糙度参数的作用。

示例的源码可以从 [LearnOpenGL 的网站](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/1.1.lighting/lighting.cpp)找到。

### **3.2.3 使用纹理的 PBR（Textured PBR）**

3.2.2.3 小节的 PBR 实现中，部分重要的表面材质属性是`float`类型：

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

![[1679148477400.png]]

可以在这里找到带纹理的 [Demo 源码](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/1.2.lighting_textured/lighting_textured.cpp)，所有用到的纹理在[这里](http://freepbr.com/materials/rusted-iron-pbr-metal-material-alt/)（用了白色的 AO 贴图）。记住金属表面在直接光照环境中更暗是因为他们没有漫反射。在环境使用环境高光进行光照计算的情况下看起来也是正常的，这个我们在下一个教程里再说。

这里没有其他 PBR 渲染示例中那样令人惊艳的效果，因为我们还没有加入基于图片的光照（Image Based Lighting）技术。尽管如此，这个 shader 任然算是一个基于物理的渲染，即使没有 IBL 你也可以法线光照看起来真实了很多。

## **3.3 基于图像的光照（Image Based Lighting，IBL）**

基于图像的光照（IBL）是对光源物体的技巧集合，与直接光照不同，它将周围环境当成一个大光源。IBL 通常结合 cubemap 环境贴图，cubemap 通常采集自真实的照片或从 3D 场景生成，这样可以将其用于光照方程：将 cubemap 的每个像素当成一个光源。这样可以更有效地捕获全局光照和常规感观，使得被渲染的物体更好地融入所处的环境中。

当基于图像的光照算法获得一些（全局的）环境光照时，它的输入被当成更加精密形式的环境光照，甚至是一种粗糙的全局光照的模拟。这使得 IBL 有助于 PBR 的渲染，使得物体渲染效果更真实。

在介绍 IBL 结合 PBR 之前，先回顾一下反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

如之前所述，我们的主目标是解决所有入射光 $w_i$通过半球$\Omega$的积分$\int$。与直接光照不同的是，在 IBL 中，**每一个**来自周围环境的入射光$\omega_i$都可能存在辐射，这些辐射对解决积分有着重要的作用。为解决积分有两个要求：

*   需要用某种方法获得给定任意方向向量$\omega_i$的场景辐射。
*   解决积分需尽可能快并实时。

对第一个要求，相对简单，采用环境 cubemap。给定一个 cubemap，可以假设它的每个像素是一个单独的发光光源。通过任意方向向量$\omega_i$采样 cubemap，可以获得场景在这个方向的辐射。

获取任意方向向量$\omega_i$的场景辐射很简单，如下：

```
vec3 radiance = texture(_cubemapEnvironment, w_i).rgb;
```

对要求二，解决积分能只考虑一个方向的辐射，要考虑环境贴图的半球$\Omega$的所有可能的方向$\omega_i$，但常规积分方法在片元着色器中开销非常大。为了有效解决积分问题，可采用预计算或预处理的方法。因此，需要深究一下反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

可将上述的 $k_d$和 $k_s$项拆分：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i+ \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

拆分后，可分开处理漫反射和镜面反射的积分。先从漫反射积分开始。

### **3.3.1 漫反射辐照度（Diffuse irradiance）**

仔细分析上面方程的漫反射积分部分，发现 Lambert 漫反射是个常量项（颜色 $c$，折射因子 $k_d$和$\pi$）并且不依赖积分变量。因此，可见常量部分移出漫反射积分：

$$L_o(p,\omega_o) = k_d\frac{c}{\pi} \int\limits_{\Omega} L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

因此，积分只依赖$\omega_i$（假设 $p$在环境贴图的中心）。据此，可以计算或预计算出一个新的 cubemap，这个 cubemap 存储了用**卷积**（convolution）计算出的每个采样方向（或像素）$\omega_o$的漫反射积分结果。

**卷积**（convolution）是对数据集的每个入口应用一些计算，假设其它所有的入口都在这个数据集里。此处的数据集就是场景辐射或环境图。因此，对 cubemap 的每个采样方向，我们可以顾及在半球$\Omega$的其它所有的采样方向。

为了卷积环境图，我们要解决每个输出$\omega_o$采样方向的积分，通过离散地采样大量的在半球$\Omega$的方向$\omega_i$并取它们辐射的平均值。采样方向$\omega_i$的半球是以点 $p$为中心以$\omega_o$为法平面的。  

![[1679148477451.png]]

这个预计算的为每个采样方向$\omega_o$存储了积分结果的 cubemap，可被当成是预计算的在场景中所有的击中平行于$\omega_o$表面的非直接漫反射的光照之和。这种 cubemap 被称为**辐照度图（Irradiance map）**。

辐射方程依赖于位置 $p$，假设它在辐照度图的中心。这意味着所有非直接漫反射光需来自于同一个环境图，它可能打破真实的幻觉（特别是室内）。渲染引擎用放置遍布场景的反射探头（reflection probe）来解决，每个反射探头计算其所处环境的独自的辐照度图。这样，点 p 的辐射率（和辐射）是与其最近的反射探头的辐照度插值。这里我们假设总是在环境图的中心采样。反射探头将在其它章节探讨。

下面是 cubemap 环境图 (下图左）和对应的辐照度图（下图右）：  

![[1679148477478.png]]

通过存储每个 cubemap 像素卷积的结果，辐照度图有点像环境的平均颜色或光照显示。从这个环境图采样任意方向，可获得这个方向的场景辐照度。

#### **3.3.1.1 球体图（Equirectangular map）**

球体图（Equirectangular map）有些文献翻译成全景图，它与 cubemap 不一样的是：cubemap 需要 6 张图，而球体图只需要一张，并且存储的贴图有一定形变：  

![[1679148477522.png]]

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

![[1679148477573.png]]

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
    // Gamma校正（只在颜色为线性空间的渲染管线才需要）
    envColor = pow(envColor, vec3(1.0/2.2)); 
  
    FragColor = vec4(envColor, 1.0);
}
```

上述代码中，要注意在输出最终的颜色之前，做了 HDR 到 LDR 的转换和 Gamma 校正。

渲染的效果如下图：  

![[1679148477687.png]]

#### **3.3.1.3 PBR 和非直接辐射度光照（indirect irradiance lighting）**

辐射度图提供了漫反射部分的积分，该积分表示来自非直接的所有方向的环境光辐射之和。由于辐射度图被当成是无方向性的光源，所以可以将漫反射镜面反射合成环境光。

首先，得声明预计算出的辐射度图的 sample：

```
uniform samplerCube irradianceMap;
```

通过表面的法线，获得环境光可以简化成下面的代码：

```
// vec3 ambient = vec3(0.03);
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

由于环境光来自在半球内所有围绕着法线`N`的方向，没有单一的半向量去决定菲涅尔因子。为了仍然能模拟菲涅尔，这里采用了法线和视线的夹角。之前的算法采用了受表面粗糙度影响的微平面半向量，作为菲涅尔方程的输入。这里，我们并不考虑粗糙度，表面的反射因子被视作相当大。

非直接光照将沿用直接光照的相同的属性，所以，期望越粗糙的表面镜面反射越少。由于不考虑表面粗糙度，非直接光照的菲涅尔方程强度被视作粗糙的非金属表面（下图）。  

![[1679148477738.png]]

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

![[1679148477776.png]]

本节所有代码可在[这里](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.1.2.ibl_irradiance/ibl_irradiance.cpp)找到。

### **3.3.2 镜面的 IBL（Specular IBL）**

3.3.1 描述的是 IBL 的漫反射部分，本节将讨论 IBL 的镜面反射部分先回顾一下反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

上述的镜面反射部分（被 $k_s$相乘）不是恒定的，并且依赖于入射光方向和视线入射方向，尝试实时地计算所有入射光和所有入射视线的积分是几乎不可能的。Epic Games 推荐折中地使用预卷积镜面反射部分的方法来解决实时渲染的性能问题，这就是**分裂和近似法（split sum approximation）**。

**分裂和近似法**将镜面反射部分从反射方程分离出两个部分，这样可以单独地对它们卷积，后面在 PBR 的 shader 中为镜面的非直接 IBL 将它们结合起来。跟预卷积辐射度图类似，分裂和近似法需要 HDR 环境图作为输入。为了更好地理解分裂和近似法，下面着重关注反射方程的镜面部分：

$$\begin{eqnarray*} L_o(p,\omega_o) & = & \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)} L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i \end{eqnarray*}$$

出于跟辐射度图相同的性能问题的考虑，我们要预计算类似镜面 IBL 图的积分，并且用片元的法线采样这个图。辐射度图的预计算只依赖于$\omega_i$，并且我们可以将漫反射项移出积分。但这次从 BRDF 可以看出，不仅仅是依赖于$\omega_i$：

$$f_r(p, w_i, w_o) = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

如上方程所示，还依赖$\omega_o$，并且我们不能用两个方向向量来采样预计算的 cubemap。预计算所有$\omega_i$和$\omega_o$的组合在实时渲染环境中不实际的。

Epic Games 的分裂和近似法将镜面反射部分从反射方程分离出两个部分，这样可以单独地对它们卷积，后面在 PBR 的 shader 中为镜面的非直接 IBL 将它们结合起来。分离后的方程如下：

$$\begin{eqnarray*} L_o(p,\omega_o) & = & \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)} L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} L_i(p,\omega_i) d\omega_i * \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d\omega_i \end{eqnarray*}$$

第一部分$\int\limits_{\Omega} L_i(p,\omega_i) d\omega_i$是**预过滤环境图（pre-filtered environment map）**，类似于辐射度图的预计算环境卷积图，但会加入粗糙度。随着粗糙度等级的增加，环境图使用更多的散射采样向量来卷积，创建出更模糊的反射。

对每个卷积的粗糙度等级，循环地在预过滤环境图的 mimap 等级存储更加模糊的结果。下图是 5 个不同粗糙度等级的预过滤环境图：  

![[1679148477798.png]]

生成采样向量和它们的散射强度，需要用到 Cook-Torrance BRDF 的法线分布图（NDF），而其带了两个输入：法线和视线向量。当卷积环境图时并不知道视线向量，Epic Games 用了更近一步的模拟法：假设视线向量（亦即镜面反射向量）总是等于输出采样向量$\omega_o$。所以代码变成如下所示：

```
vec3 N = normalize(w_o);
vec3 R = N;
vec3 V = R;
```

这种方式预过滤环境图卷积不需要关心视线方向。这就意味着当从某个角度看向下面这张图的镜面表面反射时，无法获得很好的掠射镜面反射（grazing specular reflections）。然而通常这被认为是一个较好的妥协：  

![[1679148477844.png]]

第二部分$\int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d\omega_i$是**镜面积分**。假设所有方向的入射辐射率是全白的（那样 $L(p, x) = 1.0$），那就可以用给定的粗糙度和一个法线 $n$和光源方向$\omega_i$之间的角度或 $n \cdot \omega_i$来预计算 BRDF 的值。Epic Games 存储了用变化的粗糙度来预计算每一个法线和光源方向组合的 BRDF 的值，该粗糙度存储于 2D 采样纹理（LUT）中，它被称为 **BRDF 积分图（BRDF integration map）**。

2D 采样纹理输出一个缩放（红色）和一个偏移值（绿色）给表面的菲涅尔方程式（Fresnel response），以便提供第二部分的镜面积分：  

![[1679148477893.png]]

  
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

接着用法线`N`和视线`-V`算出反射向量`R`，再结合`MAX_REFLECTION_LOD`和粗糙度等参数采样预过滤环境图：

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

此时可以算出由 IBL 的漫反射和镜面反射部分结合而成的环境光`ambient`，渲染效果如下：  

![[1679148477938.png]]

扩展一下，加入一些酷酷的[材质](https://freepbr.com/)：  

![[1679148477981.png]]

或者加载[这些极好又免费的 PBR 3D 模型](http://artisaverb.info/PBT.html)（by Andrew Maximov）：  

![[1679148478025.png]]

非常肯定地，加了 IBL 光照后，渲染效果更真实更加物理正确。下图展示了在未改变任何光照信息的情况下，在不同的预计算 HDR 图中的效果，它们看起来依然是物理正确的：  

![[1679148478048.png]]

IBL 的教程结束了，本节的代码可在[球体场景](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.2.1.ibl_specular/ibl_specular.cpp)和[纹理场景](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.2.2.ibl_specular_textured/ibl_specular_textured.cpp)中找到。
