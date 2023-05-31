 

这篇鸽了，真的不好意思……（对不起评论区的小伙伴）

原定是上个月译完发的，没想到业务繁忙，没有什么精力与时间做这事了。

另外，第四章的第一节篇幅是真的长，而且有各种数学符号与公式，光输入就花了不少时间。

不过每天这么一段一段的往前推进，最终还是完成了！业余翻译，若有不周到之处，还请多多指教。

![](<images/1685518920043.png>)

# **实时渲染（第四版）Real-Time Rendering (Fourth Edition)**

# **第 4 章 变换** **Chapter 4 Transforms**

_“What if angry vectors veer_

_Round your sleeping head, and form._

_There’s never need to fear_

_Violence of the poor world’s abstract storm.”_

_—Robert Penn Warren_

_要是愤怒的航船（vectors）改变了方向  
围绕着你沉睡的脑袋，和身体  
那就永远不必去害怕  
穷苦世界的抽象风暴之_暴行

_——罗伯特 · 佩 · 华伦_

_（注：此处航船可换为向量，与此章联系起来，一语双关。）_

变换（transform）是一种操作，它接受点（points），向量（vectors）或颜色（colors）之类的实体（entities），并且以某种方式转换它们。对于计算机图形从业者来说，掌握变换是非常重要的。有了它们，你可以定位（position）、重塑（reshape）和移动（animate）物体、灯光和照相机。你还可以确保所有计算都在同一坐标系中进行，并且以不同的方式将对象投影到平面上。这里只是少数可以通过变换执行的操作，但是这已经充分说明了变换（transform）在实时图形或任何类型计算机图形中的重要性。

线性变换（linear transform）是保留向量加法和标量乘法的变换。具体来说：

$\\ \textbf{f}(\textbf{x})+ \textbf{f}(\textbf{y}) = \textbf{f}(\textbf{x+y}),\;\;\;\;(4.1)\\ k\textbf{f}(\textbf{x}) = \textbf{f}(k\textbf{x}).\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;(4.2)$

举个例子，

![](<images/1685518920092.png>)

 是一个使向量每个元素乘以 5 的变换。为了证明这是线性的，需要满足两个条件（公式 4.1 与 公式 4.2）。第一个条件成立，因为任何两个向量乘以 5 然后相加就等于将向量相加然后再相乘。第二个标量乘法条件（公式 4.2）已经明显是满足的。此函数我们称为缩放变换（scaling transform），因为它可以更改对象的缩放比例（大小）。旋转变换（rotation transform）是另一个线性变换，它可使向量绕原点旋转。包括缩放和旋转变换，实际上三维向量的所有线性变换，都可以使用 3 × 3 矩阵表示。

然而，矩阵的大小通常不够大。三维向量

![](<images/1685518920955.png>)

 的函数，例如  

![](<images/1685518921729.png>)

  是非线性的。在两个单独的向量上执行此函数将把

![](<images/1685518922474.png>)

$(7,3,2)$ 的每个值相加两次。这种将定值向量加到另一个向量上的操作称为平移（translation），它会将位置移动相应的值。平移是一种很有用的变换类型，接下来我们将组合各种变换，例如，将对象缩放到一半大小，然后将其移动到其他位置。此时我们会发现，将函数保持在简单形式是很难轻松地进行组合的。

我们可以使用仿射变换（affine transform）将线性变换（linear transforms）和平移（translations）结合起来，仿射变换通常存储为 4 × 4 矩阵。仿射变换是先执行线性变换然后执行平移变换。为了表示四维向量，我们使用齐次表示法（homogeneous notation），以相同的方式表示点和方向（使用粗体小写字母）。方向向量（direction vector）表示为 $\textbf{v} = (v_{x}\;v_{y}\;v_{z}\;0)^{T}$，点（point）表示为 $\textbf{v} = (v_{x}\;v_{y}\;v_{z}\;1)^{T}$。在本章中，我们将广泛使用在 realtimerendering.com 上可下载的线性代数附录中解释的术语和运算。

所有平移（translation），旋转（rotation），缩放（scaling），反射（reflflection）和剪切矩阵（shearing matrices）都是仿射（affine）。仿射矩阵（affine matrix）的主要特征就是它保留了线的平行性，但不一定保留长度和角度。仿射变换（affine transform）也可以是各个仿射变换级联（concatenations）的任何序列。

本章将从最根本的基本仿射变换（basic affine transforms）开始。本部分可以看作是简单变换的 “参考手册”。之后，我们会描述一些专用的矩阵，随后对四元数（quaternions）——一种强大的变换工具，进行讨论和描述。然后是顶点融合（blending）和变形（morphing），这是表达网格动画的两种简单但有效的方法。最后，描述了投影矩阵。这些变换中大多数，它们的符号，功能和特性都总结在 表 4.1 中，其中正交矩阵的逆矩阵为其转置矩阵。

变换（Transforms）是用于操纵几何体（geometry）的基本工具。大多数图形应用程序编程接口允许用户设置任意矩阵，有时库（library）可能与实现本章讨论的许多变换的矩阵运算一起使用。但是，仍然有必要了解函数调用背后的实际矩阵及其相互作用。知道调用函数后矩阵做了什么是一个开始，但是了解矩阵本身的属性将使你的理解更进一步。例如，这种理解可以使你辨别何时处理正交矩阵（正交是其转置），从而可以更快地进行矩阵求逆。这样的知识可以让你的代码更加高效。

## **4.1 基本变换 Basic Transforms**

本节介绍最基本的变换，例如平移（translation），旋转（rotation），缩放（scaling），剪切（shearing），变换级联（transform concatenation），刚体变换（rigid-body transform），法线变换（normal transform）（然而并不是很正常 原文：which is not so normal 双关语笑话）和逆计算（computation of inverses）。对于有经验的读者，可以将其用作简单变换的参考手册，对于新手，则可以作为该主题的入门。本材料是本章其余部分和本书其他各章的必要背景。我们从最简单的变换——“平移” 开始。

<table border="1" cellpadding="1" cellspacing="1" style="width:700px;"><caption>表 4.1 本章讨论到的大多数变换的小结</caption><thead><tr><th>符号 Notation</th><th>名称 Name</th><th>特点 Characteristics</th></tr></thead><tbody><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-5-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>T</mtext></mrow><mo stretchy=&quot;false&quot;>(</mo><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>t</mtext></mrow><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-150" style="width: 2.443em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.027em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.313em, 1001.91em, 2.682em, -999.997em); top: -2.259em; left: 0em;"><span class="mrow" id="MathJax-Span-151"><span class="texatom" id="MathJax-Span-152"><span class="mrow" id="MathJax-Span-153"><span class="mtext" id="MathJax-Span-154" style="font-family: MathJax_Main-bold;">T</span></span></span><span class="mo" id="MathJax-Span-155" style="font-family: MathJax_Main;">(</span><span class="texatom" id="MathJax-Span-156"><span class="mrow" id="MathJax-Span-157"><span class="mtext" id="MathJax-Span-158" style="font-family: MathJax_Main-bold;">t</span></span></span><span class="mo" id="MathJax-Span-159" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.265em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.361em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">T</mtext></mrow><mo stretchy="false">(</mo><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">t</mtext></mrow><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-5">\textbf{T}(\textbf{t})</script></span></td><td>平移矩阵&nbsp;<div><span style="color:#231f20;">translation matrix</span></div></td><td>移动一个点。仿射。</td></tr><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-6-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><msub><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>R</mtext></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>x</mi></mrow></msub><mo stretchy=&quot;false&quot;>(</mo><mi>&amp;#x03C1;</mi><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-160" style="width: 3.217em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.682em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.253em, 1002.56em, 2.622em, -999.997em); top: -2.199em; left: 0em;"><span class="mrow" id="MathJax-Span-161"><span class="msubsup" id="MathJax-Span-162"><span style="display: inline-block; position: relative; width: 1.372em; height: 0px;"><span style="position: absolute; clip: rect(3.098em, 1000.84em, 4.17em, -999.997em); top: -3.985em; left: 0em;"><span class="texatom" id="MathJax-Span-163"><span class="mrow" id="MathJax-Span-164"><span class="mtext" id="MathJax-Span-165" style="font-family: MathJax_Main-bold;">R</span></span></span><span style="display: inline-block; width: 0px; height: 3.991em;"></span></span><span style="position: absolute; top: -3.807em; left: 0.836em;"><span class="texatom" id="MathJax-Span-166"><span class="mrow" id="MathJax-Span-167"><span class="mi" id="MathJax-Span-168" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span></span></span><span style="display: inline-block; width: 0px; height: 3.991em;"></span></span></span></span><span class="mo" id="MathJax-Span-169" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-170" style="font-family: MathJax_Math-italic;">ρ</span><span class="mo" id="MathJax-Span-171" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.205em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.361em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">R</mtext></mrow><mrow class="MJX-TeXAtom-ORD"><mi>x</mi></mrow></msub><mo stretchy="false">(</mo><mi>ρ</mi><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-6">\textbf{R}_{x}(\rho )</script></span></td><td><p>旋转矩阵</p><div><span style="color:#231f20;">rotation matrix</span></div></td><td><p>绕 x 轴 旋转&nbsp;<span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-7-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>&amp;#x03C1;</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-172" style="width: 0.628em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.523em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.565em, 1000.52em, 2.503em, -999.997em); top: -2.133em; left: 0em;"><span class="mrow" id="MathJax-Span-173"><span class="mi" id="MathJax-Span-174" style="font-family: MathJax_Math-italic;">ρ</span></span><span style="display: inline-block; width: 0px; height: 2.138em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.309em; border-left: 0px solid; width: 0px; height: 0.941em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>ρ</mi></math></span></span><script type="math/tex" id="MathJax-Element-7">\rho</script></span>&nbsp;弧度角。</p><p>y 轴 与 z 轴也使用此标记。</p><p>正交且仿射。</p></td></tr><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-8-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>R</mtext></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-175" style="width: 1.015em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.836em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.372em, 1000.84em, 2.443em, -999.997em); top: -2.259em; left: 0em;"><span class="mrow" id="MathJax-Span-176"><span class="texatom" id="MathJax-Span-177"><span class="mrow" id="MathJax-Span-178"><span class="mtext" id="MathJax-Span-179" style="font-family: MathJax_Main-bold;">R</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.265em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">R</mtext></mrow></math></span></span><script type="math/tex" id="MathJax-Element-8">\textbf{R}</script></span></td><td><div>旋转矩阵</div><div><span style="color:#231f20;">rotation matrix</span></div></td><td><p>任意的旋转矩阵。</p><p>正交且仿射。</p></td></tr><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-9-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>S</mtext></mrow><mo stretchy=&quot;false&quot;>(</mo><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>s</mtext></mrow><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-180" style="width: 2.265em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.848em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.313em, 1001.73em, 2.682em, -999.997em); top: -2.259em; left: 0em;"><span class="mrow" id="MathJax-Span-181"><span class="texatom" id="MathJax-Span-182"><span class="mrow" id="MathJax-Span-183"><span class="mtext" id="MathJax-Span-184" style="font-family: MathJax_Main-bold;">S</span></span></span><span class="mo" id="MathJax-Span-185" style="font-family: MathJax_Main;">(</span><span class="texatom" id="MathJax-Span-186"><span class="mrow" id="MathJax-Span-187"><span class="mtext" id="MathJax-Span-188" style="font-family: MathJax_Main-bold;">s</span></span></span><span class="mo" id="MathJax-Span-189" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.265em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.361em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">S</mtext></mrow><mo stretchy="false">(</mo><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">s</mtext></mrow><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-9">\textbf{S}(\textbf{s})</script></span></td><td><p>缩放矩阵</p><p>scaling matrix</p></td><td><p>根据&nbsp;<span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-10-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>s</mtext></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-190" style="width: 0.576em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.471em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.565em, 1000.42em, 2.346em, -999.997em); top: -2.185em; left: 0em;"><span class="mrow" id="MathJax-Span-191"><span class="texatom" id="MathJax-Span-192"><span class="mrow" id="MathJax-Span-193"><span class="mtext" id="MathJax-Span-194" style="font-family: MathJax_Main-bold;">s</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.19em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.059em; border-left: 0px solid; width: 0px; height: 0.691em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">s</mtext></mrow></math></span></span><script type="math/tex" id="MathJax-Element-10">\textbf{s}</script></span>&nbsp;值来缩放 x，y，z 轴。</p><p>仿射。</p></td></tr><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-11-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><msub><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>H</mtext></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>i</mi><mi>j</mi></mrow></msub><mo stretchy=&quot;false&quot;>(</mo><mi>s</mi><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-195" style="width: 3.336em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.741em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.253em, 1002.62em, 2.682em, -999.997em); top: -2.199em; left: 0em;"><span class="mrow" id="MathJax-Span-196"><span class="msubsup" id="MathJax-Span-197"><span style="display: inline-block; position: relative; width: 1.491em; height: 0px;"><span style="position: absolute; clip: rect(3.098em, 1000.84em, 4.17em, -999.997em); top: -3.985em; left: 0em;"><span class="texatom" id="MathJax-Span-198"><span class="mrow" id="MathJax-Span-199"><span class="mtext" id="MathJax-Span-200" style="font-family: MathJax_Main-bold;">H</span></span></span><span style="display: inline-block; width: 0px; height: 3.991em;"></span></span><span style="position: absolute; top: -3.807em; left: 0.896em;"><span class="texatom" id="MathJax-Span-201"><span class="mrow" id="MathJax-Span-202"><span class="mi" id="MathJax-Span-203" style="font-size: 70.7%; font-family: MathJax_Math-italic;">i</span><span class="mi" id="MathJax-Span-204" style="font-size: 70.7%; font-family: MathJax_Math-italic;">j</span></span></span><span style="display: inline-block; width: 0px; height: 3.991em;"></span></span></span></span><span class="mo" id="MathJax-Span-205" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-206" style="font-family: MathJax_Math-italic;">s</span><span class="mo" id="MathJax-Span-207" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.205em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.425em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">H</mtext></mrow><mrow class="MJX-TeXAtom-ORD"><mi>i</mi><mi>j</mi></mrow></msub><mo stretchy="false">(</mo><mi>s</mi><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-11">\textbf{H}_{ij}(s)</script></span></td><td><p>剪切矩阵</p><p>shear matrix</p></td><td><p>相对于分量&nbsp;<span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-12-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>j</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-208" style="width: 0.523em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.419em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.305em, 1000.42em, 2.503em, -999.997em); top: -2.133em; left: 0em;"><span class="mrow" id="MathJax-Span-209"><span class="mi" id="MathJax-Span-210" style="font-family: MathJax_Math-italic;">j</span></span><span style="display: inline-block; width: 0px; height: 2.138em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.309em; border-left: 0px solid; width: 0px; height: 1.191em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>j</mi></math></span></span><script type="math/tex" id="MathJax-Element-12">j</script></span>&nbsp;，用因子&nbsp;<span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-13-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>s</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-211" style="width: 0.576em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.471em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.565em, 1000.42em, 2.294em, -999.997em); top: -2.133em; left: 0em;"><span class="mrow" id="MathJax-Span-212"><span class="mi" id="MathJax-Span-213" style="font-family: MathJax_Math-italic;">s</span></span><span style="display: inline-block; width: 0px; height: 2.138em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.059em; border-left: 0px solid; width: 0px; height: 0.691em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>s</mi></math></span></span><script type="math/tex" id="MathJax-Element-13">s</script></span>&nbsp;剪切分量 <span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-14-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>i</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-214" style="width: 0.471em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.367em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.305em, 1000.32em, 2.294em, -999.997em); top: -2.133em; left: 0em;"><span class="mrow" id="MathJax-Span-215"><span class="mi" id="MathJax-Span-216" style="font-family: MathJax_Math-italic;">i</span></span><span style="display: inline-block; width: 0px; height: 2.138em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.059em; border-left: 0px solid; width: 0px; height: 0.941em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>i</mi></math></span></span><script type="math/tex" id="MathJax-Element-14">i</script></span>&nbsp;。</p><p><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-15-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>i</mi><mo>,</mo><mi>j</mi><mi>&amp;#x03F5;</mi><mo stretchy=&quot;false&quot;>(</mo><mi>x</mi><mo>,</mo><mi>y</mi><mo>,</mo><mi>z</mi><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-217" style="width: 5.784em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.794em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.305em, 1004.69em, 2.607em, -999.997em); top: -2.185em; left: 0em;"><span class="mrow" id="MathJax-Span-218"><span class="mi" id="MathJax-Span-219" style="font-family: MathJax_Math-italic;">i</span><span class="mo" id="MathJax-Span-220" style="font-family: MathJax_Main;">,</span><span class="mi" id="MathJax-Span-221" style="font-family: MathJax_Math-italic; padding-left: 0.159em;">j</span><span class="mi" id="MathJax-Span-222" style="font-family: MathJax_Math-italic;">ϵ</span><span class="mo" id="MathJax-Span-223" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-224" style="font-family: MathJax_Math-italic;">x</span><span class="mo" id="MathJax-Span-225" style="font-family: MathJax_Main;">,</span><span class="mi" id="MathJax-Span-226" style="font-family: MathJax_Math-italic; padding-left: 0.159em;">y<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-227" style="font-family: MathJax_Main;">,</span><span class="mi" id="MathJax-Span-228" style="font-family: MathJax_Math-italic; padding-left: 0.159em;">z<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-229" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.19em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.372em; border-left: 0px solid; width: 0px; height: 1.316em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>i</mi><mo>,</mo><mi>j</mi><mi>ϵ</mi><mo stretchy="false">(</mo><mi>x</mi><mo>,</mo><mi>y</mi><mo>,</mo><mi>z</mi><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-15">i,j\epsilon (x,y,z)</script></span>。仿射。</p></td></tr><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-16-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>E</mtext></mrow><mo stretchy=&quot;false&quot;>(</mo><mi>h</mi><mo>,</mo><mi>p</mi><mo>,</mo><mi>r</mi><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-230" style="width: 4.824em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.991em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.313em, 1003.87em, 2.682em, -999.997em); top: -2.259em; left: 0em;"><span class="mrow" id="MathJax-Span-231"><span class="texatom" id="MathJax-Span-232"><span class="mrow" id="MathJax-Span-233"><span class="mtext" id="MathJax-Span-234" style="font-family: MathJax_Main-bold;">E</span></span></span><span class="mo" id="MathJax-Span-235" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-236" style="font-family: MathJax_Math-italic;">h</span><span class="mo" id="MathJax-Span-237" style="font-family: MathJax_Main;">,</span><span class="mi" id="MathJax-Span-238" style="font-family: MathJax_Math-italic; padding-left: 0.182em;">p</span><span class="mo" id="MathJax-Span-239" style="font-family: MathJax_Main;">,</span><span class="mi" id="MathJax-Span-240" style="font-family: MathJax_Math-italic; padding-left: 0.182em;">r</span><span class="mo" id="MathJax-Span-241" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.265em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.361em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">E</mtext></mrow><mo stretchy="false">(</mo><mi>h</mi><mo>,</mo><mi>p</mi><mo>,</mo><mi>r</mi><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-16">\textbf{E}(h,p,r)</script></span></td><td><p>欧拉变换</p><p>Euler transform</p></td><td><p>根据 head(yaw)，pitch，roll</p><p>三个方向的欧拉角给出的方向矩阵。</p><p>正交 &amp; 仿射。</p></td></tr><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-17-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><msub><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>P</mtext></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>o</mi></mrow></msub><mo stretchy=&quot;false&quot;>(</mo><mi>s</mi><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-242" style="width: 2.979em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.443em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.253em, 1002.32em, 2.622em, -999.997em); top: -2.199em; left: 0em;"><span class="mrow" id="MathJax-Span-243"><span class="msubsup" id="MathJax-Span-244"><span style="display: inline-block; position: relative; width: 1.193em; height: 0px;"><span style="position: absolute; clip: rect(3.098em, 1000.72em, 4.17em, -999.997em); top: -3.985em; left: 0em;"><span class="texatom" id="MathJax-Span-245"><span class="mrow" id="MathJax-Span-246"><span class="mtext" id="MathJax-Span-247" style="font-family: MathJax_Main-bold;">P</span></span></span><span style="display: inline-block; width: 0px; height: 3.991em;"></span></span><span style="position: absolute; top: -3.807em; left: 0.777em;"><span class="texatom" id="MathJax-Span-248"><span class="mrow" id="MathJax-Span-249"><span class="mi" id="MathJax-Span-250" style="font-size: 70.7%; font-family: MathJax_Math-italic;">o</span></span></span><span style="display: inline-block; width: 0px; height: 3.991em;"></span></span></span></span><span class="mo" id="MathJax-Span-251" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-252" style="font-family: MathJax_Math-italic;">s</span><span class="mo" id="MathJax-Span-253" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.205em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.361em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">P</mtext></mrow><mrow class="MJX-TeXAtom-ORD"><mi>o</mi></mrow></msub><mo stretchy="false">(</mo><mi>s</mi><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-17">\textbf{P}_{o}(s)</script></span></td><td><div>正交投影</div><div><span style="color:#231f20;">orthographic projection</span></div></td><td><p>平行投影到某个平面或某个体积上。</p><p>仿射。</p></td></tr><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-18-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><msub><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>P</mtext></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>p</mi></mrow></msub><mo stretchy=&quot;false&quot;>(</mo><mi>s</mi><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-254" style="width: 2.979em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.443em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.253em, 1002.32em, 2.682em, -999.997em); top: -2.199em; left: 0em;"><span class="mrow" id="MathJax-Span-255"><span class="msubsup" id="MathJax-Span-256"><span style="display: inline-block; position: relative; width: 1.193em; height: 0px;"><span style="position: absolute; clip: rect(3.098em, 1000.72em, 4.17em, -999.997em); top: -3.985em; left: 0em;"><span class="texatom" id="MathJax-Span-257"><span class="mrow" id="MathJax-Span-258"><span class="mtext" id="MathJax-Span-259" style="font-family: MathJax_Main-bold;">P</span></span></span><span style="display: inline-block; width: 0px; height: 3.991em;"></span></span><span style="position: absolute; top: -3.807em; left: 0.777em;"><span class="texatom" id="MathJax-Span-260"><span class="mrow" id="MathJax-Span-261"><span class="mi" id="MathJax-Span-262" style="font-size: 70.7%; font-family: MathJax_Math-italic;">p</span></span></span><span style="display: inline-block; width: 0px; height: 3.991em;"></span></span></span></span><span class="mo" id="MathJax-Span-263" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-264" style="font-family: MathJax_Math-italic;">s</span><span class="mo" id="MathJax-Span-265" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.205em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.425em; border-left: 0px solid; width: 0px; height: 1.361em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><msub><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">P</mtext></mrow><mrow class="MJX-TeXAtom-ORD"><mi>p</mi></mrow></msub><mo stretchy="false">(</mo><mi>s</mi><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-18">\textbf{P}_{p}(s)</script></span></td><td><div>透视投影</div><div><span style="color:#231f20;">perspective projection</span></div></td><td>以透视图投影到平面或体积上。</td></tr><tr><td><span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-19-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>s</mi><mi>l</mi><mi>e</mi><mi>r</mi><mi>p</mi><mo stretchy=&quot;false&quot;>(</mo><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>\^q</mtext></mrow><mo>,</mo><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>\^r</mtext></mrow><mo>,</mo><mi>t</mi><mo stretchy=&quot;false&quot;>)</mo></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-266" style="width: 9.17em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.622em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.313em, 1007.5em, 2.682em, -999.997em); top: -2.259em; left: 0em;"><span class="mrow" id="MathJax-Span-267"><span class="mi" id="MathJax-Span-268" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-269" style="font-family: MathJax_Math-italic;">l</span><span class="mi" id="MathJax-Span-270" style="font-family: MathJax_Math-italic;">e</span><span class="mi" id="MathJax-Span-271" style="font-family: MathJax_Math-italic;">r</span><span class="mi" id="MathJax-Span-272" style="font-family: MathJax_Math-italic;">p</span><span class="mo" id="MathJax-Span-273" style="font-family: MathJax_Main;">(</span><span class="texatom" id="MathJax-Span-274"><span class="mrow" id="MathJax-Span-275"><span class="mtext" id="MathJax-Span-276" style="font-family: MathJax_Main-bold;">\^q</span></span></span><span class="mo" id="MathJax-Span-277" style="font-family: MathJax_Main;">,</span><span class="texatom" id="MathJax-Span-278" style="padding-left: 0.182em;"><span class="mrow" id="MathJax-Span-279"><span class="mtext" id="MathJax-Span-280" style="font-family: MathJax_Main-bold;">\^r</span></span></span><span class="mo" id="MathJax-Span-281" style="font-family: MathJax_Main;">,</span><span class="mi" id="MathJax-Span-282" style="font-family: MathJax_Math-italic; padding-left: 0.182em;">t</span><span class="mo" id="MathJax-Span-283" style="font-family: MathJax_Main;">)</span></span><span style="display: inline-block; width: 0px; height: 2.265em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.361em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>s</mi><mi>l</mi><mi>e</mi><mi>r</mi><mi>p</mi><mo stretchy="false">(</mo><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">\^q</mtext></mrow><mo>,</mo><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">\^r</mtext></mrow><mo>,</mo><mi>t</mi><mo stretchy="false">)</mo></math></span></span><script type="math/tex" id="MathJax-Element-19">slerp(\textbf{\^q},\textbf{\^r},t )</script></span></td><td><div>斯勒普变换</div><div><span style="color:#231f20;">slerp transform</span></div><p>（注：全称为球面线性插值变换</p><p>Spherical Linear Interpolation Transform）</p></td><td><p>生成关于四元数&nbsp;<span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-20-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>\^q</mtext></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-284" style="width: 2.138em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.773em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.305em, 1001.77em, 2.607em, -999.997em); top: -2.185em; left: 0em;"><span class="mrow" id="MathJax-Span-285"><span class="texatom" id="MathJax-Span-286"><span class="mrow" id="MathJax-Span-287"><span class="mtext" id="MathJax-Span-288" style="font-family: MathJax_Main-bold;">\^q</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.19em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.372em; border-left: 0px solid; width: 0px; height: 1.316em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">\^q</mtext></mrow></math></span></span><script type="math/tex" id="MathJax-Element-20">\textbf{\^q}</script></span>&nbsp;和&nbsp;<span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-21-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mtext mathvariant=&quot;bold&quot;>\^r</mtext></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-289" style="width: 1.982em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.617em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.305em, 1001.57em, 2.607em, -999.997em); top: -2.185em; left: 0em;"><span class="mrow" id="MathJax-Span-290"><span class="texatom" id="MathJax-Span-291"><span class="mrow" id="MathJax-Span-292"><span class="mtext" id="MathJax-Span-293" style="font-family: MathJax_Main-bold;">\^r</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.19em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.372em; border-left: 0px solid; width: 0px; height: 1.316em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mtext mathvariant="bold">\^r</mtext></mrow></math></span></span><script type="math/tex" id="MathJax-Element-21">\textbf{\^r}</script></span> 以及参数&nbsp;<span class="img-codecogs"><span class="MathJax_Preview" style="color: inherit; display: none;"></span><span class="MathJax" id="MathJax-Element-22-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>t</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-294" style="width: 0.471em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.367em; height: 0px; font-size: 120%;"><span style="position: absolute; clip: rect(1.357em, 1000.32em, 2.294em, -999.997em); top: -2.133em; left: 0em;"><span class="mrow" id="MathJax-Span-295"><span class="mi" id="MathJax-Span-296" style="font-family: MathJax_Math-italic;">t</span></span><span style="display: inline-block; width: 0px; height: 2.138em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.059em; border-left: 0px solid; width: 0px; height: 0.878em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>t</mi></math></span></span><script type="math/tex" id="MathJax-Element-22">t</script></span>&nbsp;的插值四元数。</p></td></tr></tbody></table>

### **4.1.1 平移 Translation**

从一个位置到另一个位置的变化由平移矩阵 $\textbf{T}$ 表示。此矩阵通过向量  去平移实体。$\textbf{T}$ 由下面的 公式 4.3 给出：

$\textbf{T}(\textbf{t})=\textbf{T}(t_{x},t_{y},t_{z})=\begin{pmatrix} 1 & 0 & 0 & t_{x}\\ 0 & 1 & 0 & t_{y}\\ 0 & 0 & 1 & t_{z}\\ 0 & 0 & 0 & 1 \end{pmatrix}\;\;\;\;(4.3)$

图 4.1 显示了平移变换效果的示例。容易证明，将点 $\textbf{p}=(p_{x},p_{y},p_{z},1)$ 与 $\textbf{T}(\textbf{t})$ 相乘会产生一个新的点 ${\textbf{p}}'=(p_{x}+t_{x},p_{y}+t_{y},p_{z}+t_{z},1)$， 这显然是一个平移。请注意，向量 $\textbf{v}=(v_{x},v_{y},v_{z},0)$ 不受 $\textbf{T}$ 乘法的影响，因为方向向量无法平移。相反，点和向量都受其余仿射变换（affine transforms）的影响。平移矩阵的逆是 $\textbf{T}^{-1}(\textbf{t})=\textbf{T}(\textbf{-t})$，即向量 $\textbf{t}$ 取反（negated）。

![](<images/1685518923217.png>)

_图 4.1 左边的方形进行了平移变换，平移矩阵为 $\textbf{T}(5,2,0)$，方形向右移动了 5 个单位距离，向上移动了 2 个单位距离。_

在这一点上，我们应该提到的是，有时在计算机图形学中看到的另一种有效的符号方案，它使用的矩阵的底标是平移向量。例如 DirectX 就是使用这种形式。在该方案中，矩阵的顺序将颠倒，即矩阵应用的顺序将从左至右读取。由于向量是行向量，因此可以将这种表示形式的向量和矩阵称为行优先形式（row-major form）。在本书中，我们将使用列优先形式（column-major form）。无论使用哪种方式，这纯粹是一种符号上的差异。当矩阵存储在内存中时，十六进制的最后四个值为三个平移值加后跟的一个值。

### **4.1.2 旋转 Rotation**

旋转变换将一个向量（位置或方向）绕经过原点的给定轴旋转指定的角度。像平移矩阵一样，它是一个刚体变换（rigid-body transform），换句话说，它保留了变换后的点之间的距离，并保留了惯用性（handedness）（即从不导致左右两侧互换）。在计算机图形学中，这两种类型的变换对于定位和定向对象显然很有用。方向矩阵（orientation matrix）是与摄像机视图（camera view）或对象相关联的旋转矩阵，它定义了其在空间中的方向，即其向上和向前的方向。

在二维中，旋转矩阵很容易得出。假设我们有一个向量 $\textbf{v}=(v_{x},v_{y})$，我们将其参数化为 $\textbf{v}=(v_{x},v_{y})=(r\;cos\theta,r\;sin\theta)$。如果我们将向量旋转 $\phi$ 弧度（逆时针），则将得到 $\textbf{u}=(r cos(\theta +\phi ),r sin(\theta +\phi ))$。这可以重写为

$\textbf{u}=\begin{pmatrix} r\;cos(\theta +\phi )\\ r\;sin(\theta +\phi ) \end{pmatrix}= \begin{pmatrix} r(cos\theta cos\phi - sin\theta sin\phi)\\ r(sin\theta cos\phi + cos\theta sin\phi) \end{pmatrix}$

$=\begin{pmatrix} cos\phi&-sin\phi \\ sin\phi&cos\phi \end{pmatrix} \begin{pmatrix} rcos\theta\\ rsin\theta \end{pmatrix} =\textbf{R}(\phi)\textbf{v}\;\;\;\;\;\;\;\;(4.4)$

![](<images/1685518923252.png>)

在这里我们使用角度和关系来展开 $cos(\theta +\phi)$ 和  $sin(\theta+\phi)$。在三维中，常用的旋转矩阵是 $\textbf{R}_{x}(\phi)$，$\textbf{R}_{y}(\phi)$ 和 $\textbf{R}_{z}(\phi)$，它们分别绕 x，y 和 z 轴旋转实体 $\phi$ 弧度。它们由公式 4.5 – 4.7 给出：

$\textbf{R}_{x}(\phi)=\begin{pmatrix} 1 & 0 & 0 & 0\\ 0 & cos\phi & -sin\phi & 0\\ 0 & sin\phi & cos\phi & 0\\ 0 & 0 & 0 & 1 \end{pmatrix}, \;\;\;\;\;\;\;\;(4.5)$

$\textbf{R}_{y}(\phi)=\begin{pmatrix} cos\phi & 0 & sin\phi & 0\\ 0 & 1 & 0 & 0\\ -sin\phi & 0 & cos\phi & 0\\ 0 & 0 & 0 & 1 \end{pmatrix}, \;\;\;\;\;\;\;\;(4.6)$

$\textbf{R}_{z}(\phi)=\begin{pmatrix} cos\phi & -sin\phi & 0 & 0\\ sin\phi & cos\phi & 0 & 0\\ 0 & 0 & 1 & 0\\ 0 & 0 & 0 & 1 \end{pmatrix}. \;\;\;\;\;\;\;\;\;(4.7)$

如果从 4 × 4 矩阵中删除最底行和最右列，则将获得 3 × 3 矩阵。对于每个绕任意轴旋转 $\phi$ 弧度的 3 × 3 矩阵 $\textbf{R}$，它的迹（trace，矩阵中对角元素的总和）与轴无关，是恒定的，计算公式为 **[997]**：

$tr(\textbf{R})=1+2cos\phi\;\;\;\;\;\;\;\;(4.8)$

旋转矩阵的效果可以在第 65 页的 图 4.4 中看到。使旋转矩阵 $\textbf{R}_{i}(\phi)$ 起作用的原因是，它绕着轴 $i$ 旋转了 $\phi$ 弧度，但它保留了旋转中的所有点。轴， $i$，不变。注意，$\textbf{R}$ 也将用于表示围绕任何轴的旋转矩阵。上面给出的轴旋转矩阵可以在一系列的三个变换中使用，以执行任意轴旋转。该过程在 第 4.2.1 节 中讨论。第 4.2.4 节 介绍了直接绕任意轴的旋转。

所有旋转矩阵的行列式（determinant）均为 1，并且是正交的（orthogonal）。这对于任意数量的这些变换的级联（concatenations）也成立。还有另一种求逆的方法：$\textbf{R}_{i}^{-1}(\phi) = \textbf{R}_{i}(-\phi)$，即绕同一轴沿相反方向旋转。

**示例：绕点旋转**

假设我们要围绕 z 轴旋转一个 $\phi$ 弧度的对象，旋转中心为某个点 $\textbf{p}$。那么该如何进行变换？ 图 4.2 中描述了这种情况。由于绕点旋转的特征在于该点本身不受旋转的影响，因此变换首先通过平移对象使 $\textbf{p}$ 与原点重合开始，这是通过 $\textbf{T}(-\textbf{p})$ 完成的。此后跟随实际旋转：$\textbf{R}_{z}(\phi)$。最后，必须使用 $\textbf{T}(\textbf{p})$  将对象平移回其原始位置。然后，得到的变换 $\textbf{X}$ 由下式给出：

$\textbf{X}=\textbf{T}(\textbf{p})\textbf{R}_{z}(\phi)\textbf{T}(-\textbf{p})\;\;\;\;\;\;\;\;(4.9)$

注意上面矩阵的顺序。

![](<images/1685518923285.png>)

_图 4.2 围绕特定点 $\textbf{p}$ 旋转的例子。_

* * *

**引用：**

****[997]**** Lax, Peter D., Linear Algebra and Its Applications, Second Edition, John Wiley & Sons, Inc., 2007. Cited on p. 61

### **4.1.3 缩放 Scaling**

缩放矩阵 $\textbf{S}(\textbf{s})=\textbf{S}(s_{x},s_{y},s_{z})$，分别沿 x，y 和 z 方向按 $s_{x}$，$s_{y}$ 和 $s_{z}$ 的缩放因子去缩放实体。这意味着可以使用缩放矩阵来放大或缩小对象。$s_{i}$（$i\epsilon \left \{ x,y,z \right \}$）越大，则按比例缩放的实体在该方向上越大。将 s 的任何分量设置为 1 自然可以避免在该方向上缩放比例发生变化。公式 4.10 展示了 $\textbf{S}$：

$\textbf{S}(\textbf{s})=\begin{pmatrix} s_{x} & 0 & 0 & 0\\ 0 & s_{y} & 0 & 0\\ 0 & 0 & s_{z} & 0\\ 0 & 0 & 0 & 1 \end{pmatrix}. \;\;\;\;\;\;\;\;(4.10)$

第 65 页的 图 4.4 说明了缩放矩阵的作用。如果 $s_{x}=s_{y}=s_{z}$，则缩放操作称为统一操作（uniform），否则称为非统一操作（nonuniform）。有时，使用等向性（isotropic）和各向异性（anisotropic）缩放来代替统一和非统一性。倒数是

$\textbf{S}^{-1}(\textbf{s})=\textbf{S}(1/s_{x},1/s_{y},1/s_{z}).$

使用齐次坐标，创建统一缩放矩阵的另一种有效方法是通过操作位置（3，3）处的矩阵元素，即右下角的元素。该值会影响齐次坐标的 w 分量，因此会缩放矩阵变换后的点（而非方向向量）的每个坐标。例如，要均匀地缩放 5 倍，可以将缩放矩阵中（0，0），（1、1）和（2，2）的元素设置为 5，或将（3， 3）可以设置为 1/5。执行此操作的两个不同矩阵如下所示：

$\textbf{S}=\begin{pmatrix} 5 & 0 & 0 & 0\\ 0 & 5 & 0 & 0\\ 0 & 0 & 5 & 0\\ 0 & 0 & 0 & 1 \end{pmatrix} ,\;\;\;\;\;\;\;\; {\textbf{S}}'=\begin{pmatrix} 1 & 0 & 0 & 0\\ 0 & 1 & 0 & 0\\ 0 & 0 & 1 & 0\\ 0 & 0 & 0 & 1/5 \end{pmatrix}.\;\;\;\;\;\;\;\; (4.11)$

与使用 $\textbf{S}$ 进行均匀缩放相反，必须始终在使用 ${\textbf{S}}'$ 之后进行齐次化（homogenization）。这可能是低效的，因为它涉及齐次化过程中的除法。如果右下角的元素（位置（3，3））为 1，则不需要除法。当然，如果系统总是在不检测是否为 1 的情况下进行除法，则不会产生任何额外消耗。

$\textbf{s}$ 的一个或三个分量上的负值给出一种反射矩阵（reflection matrix），也称为镜像矩阵（mirror matrix）。如果只有两个比例因子为 -1，那么我们将旋转 $\pi$ 弧度。应当注意的是，与反射矩阵连接的旋转矩阵也是反射矩阵。因此，以下是反射矩阵：

$\begin{pmatrix} cos(\pi /2) & sin(\pi /2) \\ -sin(\pi /2) & cos(\pi /2) \end{pmatrix} \begin{pmatrix} 1 & 0 \\ 0 & -1 \end{pmatrix} = \begin{pmatrix} 0 & -1 \\ -1 & 0 \end{pmatrix}. \;\;\;\;\;\;\;\;(4.12)$

![](<images/1685518923310.png>)

当检测到反射矩阵时，通常需要进行特殊处理。例如，当顶点由反射矩阵变换时，其顶点具有逆时针顺序的三角形将获得顺时针顺序。此顺序更改可能导致不正确的照明和背面剔除（backface culling）。要检测给定的矩阵是否以某种方式反射，请计算矩阵左上  3×3 个元素的行列式。如果该值为负，则矩阵是反射的（reflflective）。例如，公式 4.12 中矩阵的行列式（determinant）为 $0\cdot 0-(-1)\cdot (-1)=-1$。

**示例：按特定比例缩放**

缩放矩阵 $\textbf{S}$ 仅沿 x，y 和 z 轴缩放。如果应在其他方向执行缩放，则需要复合变换。假设缩放应沿着正交轴右向向量 $\textbf{f}^{\;x}$，$\textbf{f}^{\;y}$ 和  $\textbf{f}^{\;z}$ 的轴进行。那么首先，构造矩阵 $\textbf{F}$，以更改基底（basis），如下所示：

$F=\begin{pmatrix} \textbf{f}^{\;x} & \textbf{f}^{\;y} & \textbf{f}^{\;z} & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}. \;\;\;\;\;\;\;\;(4.13)$

这个想法是使三个轴给定的坐标系与标准轴（standard axes）重合，然后使用标准缩放矩阵，然后变换回去。其第一步是通过乘以 $\textbf{F}$ 的转置即它的逆来进行的。然后完成实际的缩放，然后再变换回去。变换如公式 4.14 所示：

$\textbf{X\;=\;FS}(\textbf{s})\textbf{F}^{T}.\;\;\;\;\;\;\;\;(4.14)$

### **4.1.4 剪切 Shearing**

另一类变换是剪切矩阵集。这些矩阵可以，例如，用于扭曲游戏中整个场景，以产生迷幻效果或扭曲模型的外观。有六个基本剪切矩阵，它们分别表示为 $\textbf{H}_{xy}(s)$，$\textbf{H}_{xz}(s)$，$\textbf{H}_{yx}(s)$，$\textbf{H}_{yz}(s)$，$\textbf{H}_{zx}(s)$ 和 $\textbf{H}_{zy}(s)$。第一个下标用于表示剪切矩阵正在更改哪个坐标，而第二个下标表示进行剪切的坐标。剪切矩阵 $\textbf{H}_{xz}(s)$ 的示例如 公式 4.15 所示。注意，下标可用于在下面的矩阵中找到参数 $s$ 的位置； $x$（其数字索引为 0）标识第零行，而 $z$（其数字索引为 2 ）标识第二列，因此 $s$ 位于此处：

$\textbf{H}_{xz}(s) = \begin{pmatrix} 1 & 0 & s & 0\\ 0 & 1 & 0 & 0\\ 0 & 0 & 1 & 0\\ 0 & 0 & 0 & 1 \end{pmatrix}. \;\;\;\;\;\;\;\;(4.15)$

![](<images/1685518923340.png>)

_图 4.3。用 $\textbf{H}_{xz}(s)$ 剪切单位平方的效果。y 值和 z 值都不受转换的影响，而 x 值是旧 x 值和 s 乘以 z 值的总和，导致平方倾斜。这种变换是保留区域的，这可以通过虚线区域相同来看出。_

将此矩阵与点 $\textbf{p}$ 相乘的结果是一个点： $(p_{x}+sp_{z} \;\;p_{y}\;\;p_{z})^T$ 。以图形方式显示，如图 4.3 所示。$\textbf{H}_{ij}(s)$ 的倒数（相对于第 $j$ 个坐标剪切第 $i$ 个坐标，其中 $i\neq j$ ）是通过在相反方向上剪切产生的，即 $\textbf{H}_{ij}^{-1}(s) = \textbf{H}_{ij}(-s)$ 。

你还可以使用略有不同的剪切矩阵：

$\textbf{H}_{xy}^{'}(s,t) = \begin{pmatrix} 1 & 0 & s & 0\\ 0 & 1 & t & 0\\ 0 & 0 & 1 & 0\\ 0 & 0 & 0 & 1 \end{pmatrix}. \;\;\;\;\;\;\;\;(4.16)$

然而，在此，两个下标都用于表示这些坐标将被第三坐标剪切。这两种不同类型的描述之间的联系是 $\textbf{H}_{ij}^{-1}(s,t)=\textbf{H}_{ik}(s)\textbf{H}_{jk}(t)$，其中 $k$ 用作第三坐标的索引。该使用什么矩阵取决于个人偏好。最后，应该注意的是，由于任何剪切矩阵的行列式 $\begin{vmatrix} H \end{vmatrix}=1$，这是一个保留体积的变换，如图 4.3 所示。

### **4.1.5 变换的级联 Concatenation of Transforms**

（注：Concatenation 指一系列相关联的事物 (或事件); 这边意思是把多个矩阵结合起来。）

由于矩阵上乘法运算的不可交换性，因此矩阵出现的顺序很重要。因此，变换的级联被认为是顺序相关的。

作为顺序依赖的示例，请考虑两个矩阵 $\textbf{S}$ 和 $\textbf{R}$。$\textbf{S}(2,0.5,1)$  将 $x$ 分量缩放为两倍，将 y 分量缩放为 0.5。$\textbf{R}_{z}(\pi /6)$ 绕 $z$ 轴逆时针旋转 $\pi /6$ 弧度（在右手坐标系中从本书的页面向外指向）。这些矩阵可以用两种方法进行混合，其结果是完全不同的。这两种情况如图 4.4 所示。

![](<images/1685518923369.png>)

_图 4.4。这说明了矩阵相乘时的顺序依赖性。在第一行图片中，应用旋转矩阵  $\textbf{R}_{z}(\pi /6)$，然后缩放  $\textbf{S}(\textbf{s})$，其中 $\textbf{s}=(2,0.5,1)$。这样，合成后的矩阵为  $\textbf{S}(\textbf{s})\textbf{R}_{z}(\pi /6)$ 。在第二行图片中，以相反的顺序应用矩阵，从而得出  $\textbf{R}_{z}(\pi /6)\textbf{S}(\textbf{s})$。结果明显不同。对于任意矩阵  $\textbf{M}$ 和 $\textbf{N}$，通常认为  $\textbf{MN}\neq \textbf{NM}$。_

将一系列矩阵的连接转换为单个矩阵的明显原因是为了提高效率。例如，假设你的游戏场景具有数百万个顶点，并且场景中的所有对象都必须缩放，旋转并最终平移。现在，不是将所有顶点与这三个矩阵中的每一个相乘，而是将这三个矩阵连接到一个矩阵中。然后将此单个矩阵应用于顶点。该复合矩阵为 $\textbf{C\;=\;TRS}$。注意这里的顺序。比例矩阵 $\textbf{S}$ 应该首先应用于顶点，因此在合成中显示在右侧。该排序意味着 $\textbf{TRSp}=\textbf{(T(R(Sp)))}$，其中 $\textbf{p}$ 是要变换的点。顺便说一句，$\textbf{TRS}$ 是场景图系统（scene graph systems）常用的顺序。

值得注意的是，尽管矩阵级联是依赖于顺序的，但是矩阵可以根据需要进行分组。例如，假设你要使用 $\textbf{TRSp}$ 计算一次刚体运动变换 $\textbf{TR}$。将这两个矩阵 $\textbf{(TR)(Sp)}$ 分组在一起并用中间结果替换是有效的。因此，矩阵级联是关联的（associative）。

### **4.1.6 刚体变换 The Rigid-Body Transform**

当一个人抓住一个坚固的物体时，例如从桌子上用笔将其移动到另一个位置，也许移动到衬衫的口袋里，只有物体的方向和位置会发生变化，而物体的形状通常不会受到影响。这种仅由平移和旋转的串联组成的变换称为刚体变换。它具有保留长度，角度和惯用性的特性。

可以将任何刚体矩阵 $\textbf{X}$ 表示为平移矩阵 $\textbf{T(t)}$ 和旋转矩阵 $\textbf{R}$ 的串联。因此，$\textbf{X}$ 在公式 4.17 中具有矩阵的外观：

$\textbf{X}=\textbf{T}(t)\textbf{R}=\begin{pmatrix} r_{00} & r_{01} & r_{02} & t_{x} \\ r_{10} & r_{11} & r_{12} & t_{y} \\ r_{20} & r_{21} & r_{22} & t_{z} \\ 0 & 0 & 0 & 1 \end{pmatrix}.\;\;\;\;\;\;\;\;(4.17)$

$\textbf{X}$ 的倒数计算为 $\textbf{X}^{-1}=(\textbf{T}(\textbf{t})\textbf{R})^{-1}=\textbf{R}^{-1}\textbf{T}(\textbf{t})^{-1}$。因此，为了计算逆，对 $\textbf{R}$ 的左上 3×3 矩阵进行转置，并且 $\textbf{T}$ 的平移值改变符号。将这两个新矩阵以相反的顺序相乘以获得逆。计算 $\textbf{X}$ 的逆的另一种方法是考虑 $\textbf{R}$（使 $\textbf{R}$ 出现为 3×3 矩阵）和 $\textbf{X}$ 的符号如下（符号在第 6 页的公式 1.2 中描述）：

$\bar{\textbf{R}}=(\textbf{r}_{,0}\;\;\textbf{r}_{,1}\;\;\textbf{r}_{,2})=\begin{pmatrix} \textbf{r}_{0,}^{T}\\ \textbf{r}_{1,}^{T}\\ \textbf{r}_{2,}^{T} \end{pmatrix},\;\;\;\;\;\;\;\;(4.18)$

$\Rightarrow$

$\textbf{X}=\begin{pmatrix} \bar{\textbf{R}} & \textbf{t}\\ \textbf{0}^{T} & 1 \end{pmatrix},$

其中，$\textbf{r}_{,0}$ 表示旋转矩阵的第一列（即逗号表示从 0 到 2 的任何值，而第二个下标为 0），而 $\textbf{r}_{0}^{T}$ 是列矩阵的第一行。请注意，$\textbf{0}$ 是填充有零的 3×1 列向量。一些计算得出公式 4.19 中所示表达式的反函数：

$\textbf{X}^{-1}=\begin{pmatrix} \textbf{r}_{0} & \textbf{r}_{1} & \textbf{r}_{2} & -\bar{\textbf{R}}^{T}\textbf{t}\\ 0 & 0 & 0 & 1 \end{pmatrix}.\;\;\;\;\;\;\;\;(4.19)$

![](<images/1685518923392.png>)

_图 4.5。计算几何体变换，该变换使相机对准  $\textbf{c}$ 处，向量为 $\textbf{u}'$上看点  $\textbf{l}$。为此，我们需要计算 $\textbf{r}$， $\textbf{u}$和 $\textbf{v}$。_

**示例：调整相机的方向**

计算机图形中的常见任务是调整相机的方向，使其对准特定位置。在这里，我们将介绍 gluLookAt（）（来自 OpenGL Utility Library，简称 GLU）的作用。即使现在很少使用此函数调用，该任务仍然很常见。假设照相机位于 $\textbf{c}$ 处，我们希望照相机看着目标   $\textbf{l}$，并且照相机的给定方向为 ${\textbf{u}}'$，如图 4.5 所示。我们要计算一个由三个向量 $\left \{ \textbf{r,\;u,\;v} \right \}$ 组成的基数。我们从计算视点向量为 $\textbf{v}=\textbf{(c-1)}/\left \| \textbf{c-1} \right \|$ 开始，即从目标到摄像机位置的归一化向量。然后可以将向右看的向量计算为 $\textbf{r} = -(\textbf{v }\times\textbf{u}')/\left \| \textbf{v }\times\textbf{u}' \right \|$。通常不能保证 $\textbf{u}'$ 向量指向正上方，因此最终的向上向量是另一个叉积 $\textbf{u}=\textbf{v}\times \textbf{r}$，由于 $\textbf{v}$ 和 $\textbf{r}$ 都通过构造进行了归一化和垂直处理，因此可以保证对其进行归一化。在我们将构建的相机变换矩阵 $\textbf{M}$ 中，其思想是首先平移所有内容，使相机位置位于原点 $\left ( 0,0,0 \right )$，然后更改基数，以使 $\textbf{r}$ 与 $\left ( 1,0,0 \right )$，$\textbf{u}$ 与 $\left ( 0,1,0 \right )$ 以及 $\textbf{v}$ 与 $\left ( 0,0,1 \right )$ 。这是通过以下公式实现：

 $\textbf{ M}=\begin{pmatrix} r_{x} & r_{y} & r_{z} & 0\\ u_{x} & u_{y} & u_{z} & 0\\ v_{x} & v_{y} & v_{z} & 0\\ 0 & 0 & 0 & 1 \end{pmatrix} \begin{pmatrix} 1 & 0 & 0 & -t_{x}\\ 0 & 1 & 0 & -t_{y}\\ 0 & 0 & 1 & -t_{z}\\ 0 & 0 & 0 & 1 \end{pmatrix} = \begin{pmatrix} r_{x} & r_{y} & r_{z} & -\textbf{t}\cdot -\textbf{r}\\ u_{x} & u_{y} & u_{z} & -\textbf{t}\cdot -\textbf{u}\\ 0 & 0 & 1 & -\textbf{t}\cdot -\textbf{v}\\ 0 & 0 & 0 & 1 \end{pmatrix}.\;\;\;\;\;\;\;\;(4.20)$

![](<images/1685518923415.png>)

注意，当将平移矩阵与基本矩阵的变化连接在一起时，平移 $-\textbf{t}$ 在右边，因为它应该首先应用。记住将 $\textbf{r}$，$\textbf{u}$ 和 $\textbf{v}$ 的分量放在何处的一种方法如下。我们希望 $\textbf{r}$ 变成 $\left ( 1,0,0 \right )$，所以当将基础矩阵的变化乘以 $\left ( 1,0,0 \right )$ 时，我们可以看到矩阵的第一行必须是 $\textbf{r}$ 的元素，因为 $\textbf{r}\cdot\textbf{r}=1$。此外，第二行和第三行必须由垂直于 $\textbf{r}$ 的向量组成，即 $\textbf{r}\cdot \textbf{x}=0$。当对 $\textbf{u}$ 和 $\textbf{v}$ 应用相同的思维时，我们得出以上基础矩阵的变化。

### **4.1.7 法线变换 Normal Transform**

单个矩阵可用于一致地变换点，线，三角形和其他几何形状。矩阵还可以沿这些线或在三角形的曲面上变换切向量。但是，矩阵不能始终用于变换一个重要的几何特性，即表面法线（和顶点照明法线）。图 4.6 显示了如果使用相同的矩阵会发生什么。

![](<images/1685518923448.png>)

_图 4.6。左侧是原始几何图形，三角形以及从侧面显示的法线。中间的插图显示了如果模型沿 x 轴缩放 0.5，法线使用相同的矩阵会发生什么。右图显示了法线的正确变换。_

适当的方法不是使用矩阵本身相乘，而是使用矩阵的伴随项的转置相乘 **[227]**。伴随矩阵（adjoint）的计算在我们的在线线性代数附录中进行了描述。伴随矩阵关系始终保证存在，但法线不能保证在变换后仍是单位长度，因此通常需要将其归一化（normalized）。

变换法线的传统解法是计算逆的转置（the transpose of the inverse） **[1794]**。此方法通常有效。但是，完整的逆不是必需的，并且有时无法创建。逆是伴随数除以原始矩阵的行列式。如果该行列式为零，则矩阵是奇异的（singular）并且不存在逆。

即使只计算一个完整的 4×4 矩阵的伴随，也可能很昂贵，并且通常没有必要。由于法线是向量，因此平移不会对其产生影响。此外，大多数模型变换都是仿射的（affine）。它们不会更改传入的齐次坐标的 w 分量，也就是说，它们不会执行投影（projection）。在这些（常见）情况下，正常变换所需的仅是计算左上 3×3 分量的伴随矩阵。

通常甚至不需要这种伴随计算。假设我们知道变换矩阵完全由平移，旋转和统一缩放操作（无拉伸或压扁）的串联组成。可知平移不影响法线。统一的缩放因子仅改变法线的长度。剩下的就是一系列旋转，因此总是产生某种形式的最终的旋转值，仅此而已。逆的转置可用于变换法线。旋转矩阵是通过其转置矩阵为逆来定义的。代替以获得法线变换，两个转置（或两个逆）给出原始旋转矩阵。综上所述，原始变换本身也可以在这些情况下直接用于变换法线。

最后，并不总是需要完全重新归一化（renormalizing）生成的法线。如果仅平移和旋转连接在一起，则法线在通过矩阵进行变换时不会更改长度，因此不需要重新归一化。如果还连接了统一的缩放比例，则总缩放比例因子（如果已知或已提取，请参见第 4.2.3 节）可用于直接归一化所生成的法线。举个例子，如果我们知道应用了一系列缩放，使对象变大了 5.2 倍，则通过将此矩阵直接变换的法线除以 5.2 就会对其进行重新归一化。另外，为了创建一个产生归一化结果的法线变换矩阵，原始矩阵左上角的 3×3 可以除以该比例因子。

请注意，在变换之后，表面法线是从三角形得出的系统中，法线变换不是问题（例如，使用三角形边的叉积）。另外，切向量在本质上与法线不同，并且总是直接由原始矩阵变换而成。

* * *

**引用：**

****[227]**** Carling, Richard, “Matrix Inversion,” in Andrew S. Glassner, ed., Graphics Gems, Academic Press, pp. 470–471, 1990. Cited on p. 68

****[1794]**** Turkowski, Ken, “Properties of Surface-Normal Transformations,” in Andrew S. Glassner, ed., Graphics Gems, Academic Press, pp. 539–547, 1990. Cited on p. 68

### **4.1.8 逆的计算 Computation of Inverses**

在许多情况下都需要逆（inverses）。例如在坐标系之间来回切换时， 根据有关变换的可用信息，我们可以使用以下三种方法之一来计算矩阵的逆：

*   如果矩阵是单个变换或具有给定参数的简单变换序列，则该矩阵可以通过 “反转参数” 和矩阵顺序轻松地计算。举个例子，如果 $\textbf{M}=\textbf{T}(\textbf{t})\textbf{R}(\phi)$，则 $\textbf{M}^{-1}=\textbf{R}(-\phi)\textbf{T}(-\textbf{t})$。这很简单，并且保留了变换的准确性，这在渲染巨大世界时很重要 **[1381]**。
*   如果已知矩阵是正交的，则  $\textbf{M}^{-1}=\textbf{M}^{T}$，即转置为逆。旋转的任何序列都是旋转，因此是正交的。
*   如果没有任何已知条件，则可以使用伴随方法（the adjoint method），克莱姆法则（Cramer’s rule），LU 分解（LU decomposition）或高斯消除法（Gaussian elimination）来计算逆。通常最好使用克莱姆法则和伴随方法，因为它们的分支操作较少； 在现代体系结构上最好避免使用 “if” 测试。请参阅第 4.1.7 节，了解如何使用伴随来反转变换法线。
    

优化时也可以考虑逆计算的目的。例如，如果将逆函数用于向量变换，则通常只需要对矩阵左上角的 3×3 部分进行反转（请参见上一节）。

* * *

**引用：**

****[1381]**** Persson, Emil, “Creating Vast Game Worlds: Experiences from Avalanche Studios,” in ACM SIGGRAPH 2012 Talks, ACM, article no. 32, Aug. 2012. Cited on p. 69, 210, 245, 714, 715, 796, 797

## **4.2 特殊矩阵变换与运算 Special Matrix Transforms and Operations**

在本节中，将介绍和推导一些对实时图形来说必不可少的几种矩阵变换和运算。首先，我们介绍欧拉变换（the Euler transform）及其参数提取，欧拉变换是描述方向的一种直观的方法。然后，我们会谈到从单个矩阵中提取出一组基本变换。最终，推导出一种绕任意轴旋转实体的方法。

### **4.2.1 欧拉变换 The Euler Transform**

这种变换是构造矩阵以将自己（即相机）或任何其他实体定向到某个方向的一种直观方法。它的名字来自伟大的瑞士数学家莱昂哈德 · 欧拉（Leonhard Euler，1707–1783 年）。

首先，必须建立某种默认的视图方向。如图 4.7 所示，它通常沿负 z 轴放置，head 沿 y 轴放置。欧拉变换是三个矩阵的乘积，即图中所示的旋转。更正式地说，表示为 E 的变换由公式 4.21 给出：

$\textbf{E}(h,p,r)=\textbf{R}_{z}(r)\textbf{R}_{x}(p)\textbf{R}_{y}(h)\;\;\;\;\;\;\;\;(4.21)$

矩阵的顺序可以以 24 种不同的方式选择 **[1636]**；这可以通过选择矩阵的顺序来实现。我们介绍这个是因为它是常用的。由于 $\textbf{E}$ 是旋转的串联，因此它也显然是正交的。因此，它的逆可以表示为  $\textbf{E}^{-1}=\textbf{E}^{T}=(\textbf{R}_{z}\textbf{R}_{x}\textbf{R}_{y})^{T}=\textbf{R}_{x}^{T}\textbf{R}_{y}^{T}\textbf{R}_{z}^{T}$，当然，直接使用 $\textbf{E}$ 的转置会更容易。

欧拉角 $h$，$p$ 和 $r$ 分别表示 head，pitch 和 roll 应按其顺序旋转以及绕其各自的轴旋转多少。有时，所有角度都称为 “rolls”，例如，我们的 “head” 为 “y-roll”，而我们的 “pitch”为 “x-roll”。另外，“head” 有时也称为“yaw”，例如在飞行模拟中。

这种变换很直观，因此很容易用外行的语言进行讨论。例如，改变 head 角度会使观看者摇头 “no”，改变 pitch 会使他们点头，而 rolling 会使他们的 head 向侧面倾斜。我们不是在谈论围绕 x ， y 和 z 轴 的旋转，而是谈论改变 head，pitch 和 roll。请注意，此变换不仅可以定向相机，还可以定向任何对象或实体。可以使用世界空间的全局轴（the global axes of the world space）或相对于局部参照系执行这些变换。

重要的是要注意，一些欧拉角的表示将 z 轴作为初始向上方向。这种差异纯粹是一种符号上的变化，尽管可能会造成混淆。在计算机图形学中，如何看待世界以及如何形成内容存在分歧：y 向上 或 z 向上。大多数制造工艺（包括 3D 打印）都认为 z 方向在世界空间中向上。航空和海上交通工具认为 -z 为向上。建筑和 GIS（地理信息系统，Geographic Information System 或 Geo－Information system，缩写为 GIS） 通常使用 z-up，因为建筑平面图或地图是二维的 x 和 y 。与媒体相关的建模系统通常将 y 方向视为世界坐标上的向上方向，以匹配我们始终在计算机图形学中描述相机的屏幕向上方向的方式。这两个世界向量选择之间的差异仅相差 90° 旋转（并且可能是反射），但不知道假定哪个会导致问题。在本卷中，除非另有说明，否则我们使用 y 向上的世界方向。

![](<images/1685518923480.png>)

_图 4.7。欧拉变换及其与更改 head，pitch 和 roll 的方式之间的关系。显示默认视图方向，沿负 z 轴看，向上方向沿 y 轴看。_

我们还想指出，相机在其视野中的向上方向与世界的向上方向没有特别关系。转动 head，视野就会倾斜，其世界空间向上方向与世界方向不同。再举一个例子，假设世界使用 y-up，而我们的相机则直视下方的地形，鸟瞰。此方向表示相机已向前倾斜 90°，因此其在世界空间中的向上方向为  $(0,0,-1)$。在这种方向上，相机没有 y 分量，而是认为 -z 在世界空间中向上，但根据定义，“y 在上方” 在视图空间（view space）中仍然适用。

欧拉角虽然适用于较小的角度变化或观看者方向，但还有其他一些严重的限制——很难将两组欧拉角组合使用。例如，在一组和另一组之间进行插值并不是对每个角度进行插值的简单问题。实际上，两组不同的欧拉角可以给出相同的方向，因此任何插值都不应旋转对象。这些是使用本章稍后讨论的替代方向表示形式（例如四元数）值得追求的一些原因。使用欧拉角，你还会遇到被称为万向节锁定（gimbal lock）的问题，这将在第 4.2.2 节中介绍。

* * *

**引用：**

****[1636]**** Shoemake, Ken, “Euler Angle Conversion,” in Paul S. Heckbert, ed., Graphics Gems IV, Academic Press, pp. 222–229, 1994. Cited on p. 70, 73

### **4.2.2 从欧拉变换中提取参数 Extracting Parameters from the Euler Transform**

在某些情况下，使用从正交矩阵中提取欧拉参数 h，p 和 r 的过程很有用。此过程如公式 4.22 所示：

$\textbf{E}(h,p,r)=\begin{pmatrix} e_{00} & e_{01} & e_{02}\\ e_{10} & e_{11} & e_{12}\\ e_{20} & e_{21} & e_{22} \end{pmatrix}= \textbf{R}_{z}(r)\textbf{R}_{x}(p)\textbf{R}_{y}(h).\;\;\;\;\;\;\;\;(4.22)$

在这里，我们放弃了 3×3 矩阵的 4×4 矩阵，因为后者提供了旋转矩阵的所有必要信息。也就是说，等效的 4×4 矩阵的其余部分始终在右下位置包含 0 和一个 1。

将公式 4.22 中的三个旋转矩阵串联起来：

$\textbf{E}=\begin{pmatrix} cos\;r\;cos\;h -sin\;r\;sin\;p\;sin\;h\;& -sin\;r\;cos\;p & cos\;r\;sin\;h -sin\;r\;sin\;p\;cos\;h\; \\ sin\;r\;cos\;h -cos\;r\;sin\;p\;sin\;h\; & cos\;r\;cos\;p & sin\;r\;sin\;h-cos\;r\;sin\;p\;cos\;h\\ -cos\;p\;sin\;h & sin\;p & cos\;p\;cos\;h \end{pmatrix}.\;\;\;\;\;\;\;\;(4.23)$

由此可见，pitch 参数由 $sin\;p=e_{21}$ 给出。同样，将 $e_{01}$ 除以 $e_{11}$，并类似地将 $e_{20}$ 除以 $e_{22}$，会产生以下用于 head 和 roll 参数的提取公式：

$\frac{e_{01}}{e_{11}}=\frac{-sin\;r}{cos\;r}=-tan\;r\;\;\;\;and\;\;\;\;\frac{e_{20}}{e_{22}}=\frac{-sin\;h}{cos\;h}=-tan\;h\;\;\;\;\;\;\;\;(4.24)$

因此，如公式 4.25 所示，使用函数 atan2（y，x）（请参阅第 1 章第 8 页）从矩阵 E 提取欧拉参数 h（head），p（pitch）和 r（roll）：

$\\h = atan2(-e_{20},e_{22}),\\ p=arcsin(e_{21}),\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;\;(4.25)\\ r=atan2(-e_{01},e_{11}).\\$

但是，有一种特殊情况需要处理。如果 $cos p = 0$，则具有万向节锁定（第 4.2.2 节），旋转角度 r 和 h 将绕同一轴旋转（尽管可能沿不同的方向旋转，具体取决于 $\textbf{p}$ 旋转角度是 $-\pi /2$ 还是 $\pi /2$ ），因此只需导出一个角度。如果我们任意设置 $h=0$ **[1769]**，我们得到

$\textbf{E}=\begin{pmatrix} cos\;r & sin\;r\;cos\;p & sin\;r\;sin\;p\\ sin\;r & cos\;r\;cos\;p & -cos\;r\;sin\;p\\ 0 & sin\;p & cos\;p \end{pmatrix}.\;\;\;\;\;\;\;\;\;\;(4.26)$

由于 $\textbf{p}$ 不影响第一列中的值，因此当 $cos\;p=0$ 时，我们可以使用 $sin\;r/cos\;r=tan\;r=e_{10}/e_{00}$，得出  $r=atan2(e_{10},e_{00})$。

请注意，根据反正弦的定义，$-\pi/2\leqslant p\leqslant \pi/2$，这意味着如果使用该间隔之外的 $\textbf{p}$ 值创建 $\textbf{E}$，则无法提取原始参数。$h$，$p$ 和 $r$ 不是唯一的，意味着可以使用一组以上的欧拉参数来产生相同的变换。有关欧拉角转换的更多信息，请参见 Shoemake 的 1994 年文章 **[1636]**。上面概述的简单方法可能会导致数值不稳定的问题，这在速度方面会付出一定的代价来避免 **[1362]**。

当您使用欧拉变换时，可能会发生万向节锁定（gimbal lock）**[499，1633]**。这发生在旋转时，会因此失去一个自由度。举个例子，变换的顺序是 $x/y/z$。假设第二次旋转我们绕 y 轴旋转 $\pi/2$。这样做会旋转局部 z 轴以使其与原始 x 轴对齐，因此围绕 z 的最终旋转是不正确的。

在数学上，我们已经在公式 4.26 中看到了万向节锁定，其中假设 $cos\;p = 0$，即 $p=\pm \pi /2+2\pi k$，其中 $k$ 是整数。有了这样的 $p$ 值，我们失去了一个自由度，因为矩阵仅取决于一个角度 $r+h$ 或 $r-h$（但不是同时取决于两个角度）。

尽管在模型系统中通常以 $x/y/z$ 顺序表示欧拉角，但绕每个局部轴旋转时，其他顺序也是可行的。例如，动画中使用 $z/x/y$，动画和物理学中都使用 $z/x/z$。所有都是指定三个单独旋转的有效方法。最后的 $z/x/z$ 顺序在某些应用中可能会更好，因为只有当绕 x 旋转 $\pi$ 弧度（半旋转）时，才会发生万向节锁定。没有完美的序列可以避免万向节锁定。尽管如此，欧拉角还是很常用的，因为动画师更喜欢曲线编辑器来指定角度如何随时间变化 **[499]**。

**示例：约束变换**

想象您拿着一把（虚拟的）扳手紧紧地抓住螺栓。为了将螺栓固定到位，您必须围绕 x 轴旋转扳手。现在假设您的输入设备（鼠标，VR 手套，太空球等）为扳手的运动提供了一个旋转矩阵，即一个旋转矩阵。问题在于，将这种变换应用于扳手，扳手应该只绕 x 轴旋转，这可能是错误的。要将输入变换（称为 $\textbf{P}$）限制为绕 x 轴旋转，只需使用本节中介绍的方法提取欧拉角 $h$，$p$ 和 $r$，然后创建一个新矩阵 $\textbf{R}_{x}(p)$ 。然后，这是一种受欢迎的变换，它将使扳手绕 x 轴旋转（如果 $\textbf{P}$ 现在包含这样的运动）。

* * *

**引用：**

****[1769]**** Thomas, Spencer W., “Decomposing a Matrix into Simple Transformations,” in James Arvo, ed., Graphics Gems II, Academic Press, pp. 320–323, 1991. Cited on p. 72, 74

****[1636]**** Shoemake, Ken, “Euler Angle Conversion,” in Paul S. Heckbert, ed., Graphics Gems IV, Academic Press, pp. 222–229, 1994. Cited on p. 70, 73

****[1362]**** Paul, Richard P. C., Robot Manipulators: Mathematics, Programming, and Control, MIT Press, 1981. Cited on p. 73

****[499********]**** Frykholm, Niklas, “What Is Gimbal Lock and Why Do We Still Have to Worry about It?” Autodesk Stingray blog, Mar. 15, 2013. Cited on p. 73

****[1633]**** Shoemake, Ken, “Animating Rotation with Quaternion Curves,” Computer Graphics (SIGGRAPH ’85 Proceedings), vol. 19, no. 3, pp. 245–254, July 1985. Cited on p. 73, 76, 80, 82

### **4.2.3 矩阵分解 Matrix Decomposition**

到目前为止，我们一直在假设我们知道所使用的变换矩阵的初始状态和历史记录的情况下进行工作。通常情况下并非如此。例如，仅连接的矩阵可以与某个变换后的对象相关联。从级联矩阵中提取各种变换的任务称为矩阵分解（matrix decomposition）。

提取变换的原因有很多。用途包括：

*   为对象提取比例因子。
*   查找特定系统所需的变换。（例如，某些系统可能不允许使用任意 4×4 矩阵。）
*   确定模型是否仅经历了刚体变换。
*   在动画中的关键帧之间进行插值，其中仅对象的矩阵可用。
*   从旋转矩阵中删除剪切。

我们已经提出了两种分解方法，分别是为刚体变换导出平移和旋转矩阵（第 4.1.6 节）以及从正交矩阵导出欧拉角（第 4.2.2 节）。

正如我们所看到的，提取变换矩阵很简单，因为我们只需要 4×4 矩阵的最后一列中的元素。我们还可以通过检查矩阵的行列式是否为负来确定是否发生了反射。要分离出旋转，缩放和剪切，需要花费更多的精力。

幸运的是，有几篇关于该主题的文章以及在线提供的代码。Thomas **[1769]** 和 Goldman **[552，553]** 各自针对各种类型的变换提出了一些不同的方法。Shoemake **[1635]** 改进了他们的仿射矩阵技术，因为他的算法与参考系无关，并尝试分解矩阵以获得刚体变换。

* * *

**引用：**

****[1769]**** Thomas, Spencer W., “Decomposing a Matrix into Simple Transformations,” in James Arvo, ed., Graphics Gems II, Academic Press, pp. 320–323, 1991. Cited on p. 72, 74

****[552********]**** Goldman, Ronald, “Recovering the Data from the Transformation Matrix,” in James Arvo, ed., Graphics Gems II, Academic Press, pp. 324–331, 1991. Cited on p. 74

****[********553]**** Goldman, Ronald, “Decomposing Linear and Affine Transformations,” in David Kirk, ed., Graphics Gems III, Academic Press, pp. 108–116, 1992. Cited on p. 74

****[1635]**** Shoemake, Ken, “Polar Matrix Decomposition,” in Paul S. Heckbert, ed., Graphics Gems IV, Academic Press, pp. 207–221, 1994. Cited on p. 74

### 4.2.4 绕任意轴旋转 Rotation about an Arbitrary Axis

有时使用使实体绕任意轴旋转某个角度的过程会很方便。假设旋转轴 $\textbf{r}$ 已归一化，并且应该创建一个围绕 $\textbf{r}$ 旋转 $\alpha$ 弧度的变换。

为此，我们首先变换到一个空间，这个空间里我们要围绕其旋转的轴是 x 轴。这是通过一个称为 $\textbf{M}$ 的旋转矩阵完成的。然后执行实际的旋转，之后使用 $\textbf{M}^{-1}$ 进行变换 **[314]**。此过程如图 4.8 所示。

为了计算 $\textbf{M}$，我们需要找到两个对 $\textbf{r}$ 来说正交的轴。我们专注于找到第二根轴 s，知道第三根轴 t 将是第一根轴和第二根轴的叉积，$\textbf{t}=\textbf{r}\times \textbf{s}$ 。一种数字稳定的方法是找到 $\textbf{r}$ 的最小成分（绝对值），并将其设置为 0。交换剩余的两个成分，然后对它们中的第一个取反（实际上，可以否定非零分量中的任何一个）。在数学上，这表示为 **[784]**：

$\\ \bar{\textbf{s}}=\left\{\begin{matrix} (0,-r_{z},r_{y}),\;\;\textbf{if}\;\;\left | r_{x} \right |\leqslant \left | r_{y} \right |\;and\;\left | r_{x} \right |\leqslant \left | r_{z} \right |,\\ (-r_{z},0,r_{x}),\;\;\textbf{if}\;\;\left | r_{y} \right |\leqslant \left | r_{x} \right |\;and\;\left | r_{y} \right |\leqslant \left | r_{z} \right |,\\ (-r_{y},r_{x},0),\;\;\textbf{if}\;\;\left | r_{z} \right |\leqslant \left | r_{x} \right |\;and\;\left | r_{z} \right |\leqslant \left | r_{y} \right |, \end{matrix}\right.\;\;\;\;\;\;\;\;\;\;(4.27) \\ \textbf{s}=\bar{\textbf{s}}/\left \| \bar{\textbf{s}} \right \|, \\ \textbf{t}=\textbf{r}\times \textbf{s}.$

这保证了 $\bar{\textbf{s}}$ 与 $\textbf{r}$ 正交（垂直），并且 $(\textbf{r},\textbf{s},\textbf{t})$ 是正交的基础。Frisvad **[496]** 提出了一种在代码中没有任何分支的方法，该方法速度更快，但准确性较低。Max **[1147]** 和 Duff 等。**[388]** 提高了 Frisvad 方法的准确性。无论采用哪种技术，都会使用这三个向量来创建旋转矩阵：

$\textbf{M}=\begin{pmatrix} \textbf{r}^{T}\\ \textbf{s}^{T}\\ \textbf{t}^{T} \end{pmatrix}.\;\;\;\;\;\;\;\;\;\;(4.28)$

该矩阵将向量 $\textbf{r}$ 变换为 x 轴，将 $\textbf{s}$ 变换为 y 轴，将 $\textbf{t}$ 变换为 z 轴。因此，然后使围绕标准化向量 $\textbf{r}$ 旋转 $\alpha$ 弧度的最终变换为：

$\textbf{X}=\textbf{M}^{T}\textbf{R}_{x}(\alpha )\textbf{M}.\;\;\;\;\;\;\;\;\;\;(4.29)$

换句话说，这意味着首先我们进行变换，使 $\textbf{r}$ 为 x 轴（使用 $\textbf{M}$），然后围绕该 x 轴旋转 $\alpha$ 弧度（使用 $\textbf{R}_{x}(\alpha )$），然后使用 $\textbf{M}$ 的逆函数进行变换 ，在这种情况下为 $\textbf{M}^{T}$，因为 $\textbf{M}$ 是正交的。

Goldman **[550]** 提出了另一种通过 $\phi$ 弧度绕任意归一化轴 $\textbf{r}$ 旋转的方法。在这里，我们只介绍他的变换：

$\\\textbf{R}= \begin{pmatrix} cos\;\phi+(1-cos\;\phi)r_{x}^{2} & (1-cos\;\phi)r_{x}r_{y}-r_{z}sin\;\phi & (1-cos\;\phi)r_{x}r_{z}+r_{y}sin\;\phi\\ (1-cos\;\phi)r_{x}r_{y} +r_{z}sin\;\phi & cos\;\phi+(1-cos\;\phi)r_{y}^{2} & (1-cos\;\phi)r_{y}r_{z}-r_{x}sin\;\phi\\ (1-cos\;\phi)r_{x}r_{z}-r_{y}sin\;\phi & (1-cos\;\phi)r_{y}r_{z}+r_{x}sin\;\phi & cos\;\phi+(1-cos\;\phi)r_{z}^{2} \end{pmatrix}\\$$.\;\;\;\;\;\;\;\;\;\;(4.30)$

在第 4.3.2 节中，我们提出了解决此问题的另一种方法，使用四元数（Quaternions）。在那个部分中，还有针对相关问题（例如从一个向量到另一个向量的旋转）的更高效的算法。

* * *

**引用：**

****[314]**** Cunningham, Steve, “3D Viewing and Rotation Using Orthonormal Bases,” in Andrew S. Glassner, ed., Graphics Gems, Academic Press, pp. 516–521, 1990. Cited on p. 74

****[********784********]**** Hughes, John F., and Tomas M¨oller, “Building an Orthonormal Basis from a Unit [Vector](https://so.csdn.net/so/search?q=Vector&spm=1001.2101.3001.7020),” journal of graphics tools, vol. 4, no. 4, pp. 33–35, 1999. Also collected in [112]. Cited on p. 75, 552

****[********496********]**** Frisvad, Jeppe Revall, “Building an Orthonormal Basis from a 3D Unit Vector Without Normalization,” journal of graphics tools, vol. 16, no. 3, pp. 151–159, 2012. Cited on p. 75

****[********1147********]**** Max, Nelson, “Improved Accuracy When Building an Orthonormal Basis,” Journal of Computer Graphics Techniques, vol. 6, no. 1, pp. 9–16, 2017. Cited on p. 75

****[********388********]**** Duff, Tom, James Burgess, Per Christensen, Christophe Hery, Andrew Kensler, Max Liani, and Ryusuke Villemin, “Building an Orthonormal Basis, Revisited,” Journal of Computer Graphics Techniques, vol. 6, no. 1, pp. 1–8, 2017. Cited on p. 75

****[550]**** Goldman, Ronald, “Matrices and Transformations,” in Andrew S. Glassner, ed., Graphics Gems, Academic Press, pp. 472–475, 1990. Cited on p. 75