透明度混合中有个绕不开的话题就是 premultiplied alpha（预乘 alpha）。本文不是关于 premultiplied alpha 的教程，旨在给理解 premultiplied alpha 知识点遇到障碍的朋友一些提示。虽说是一些 tips，但还是强烈建议按顺序阅读，内容有很强相关性。
## Tip1: 理解 Alpha 混合

最常见的混合是 "over" 混合，假设已经有一张 RenderTexture，RT 上像素的 RGB 我们称其为 $RGB_{dst}$ ，alpha 为 $A_{dst}$ 。现在有一个像素 $(RGB_{src}, A_{src})$ 要和 RT 上的像素混合，那正确的混合会这样进行：

$RGB'_{result} = A_{src} *RGB_{src}+ (A_{dst}*RGB_{dst} ) *(1-A_{src})$

$A_{result} = A_{src} + A_{dst} * (1-A_{src})$

最终混合出来的颜色由两部组成：

*   $A_{src} *RGB_{src}$ 代表 $RGB{src}$ 对最终颜色的贡献，它受 alpha 影响，如果 alpha 为 0 则对最终像素没有影响，如果 alpha 为 1 则贡献 100% 的 $RGB{src}$
*    $A_{dst} * RGB_{dst}$ 是 RT 中像素原本没有其它像素覆盖时的贡献值，但是现在被一个新来的像素遮挡了，被遮挡了 $(1-A_{src})$ ，因此 RT 中像素最终贡献了 $(A_{dst}*RGB_{dst}) * (1-A_{src})$

**由此可见，无论对于 src 还是 dst，** $A * RGB$ **才是实际的有效颜色，称其为 premultiplied alpha。**

我们令 $RGB' = A * RGB$ ，则颜色混合可以改写为

$RGB'_{result} = RGB'_{src}+ RGB'_{dst} * (1-A_{src})$

这么看就更清楚了：

**最终的输出 = 新叠加像素的有效颜色 + RT 中原有像素的有效颜色 * 新像素的遮挡**

## **Tip2: SrcAlpha, OneMinusSrcAlpha 颜色混合正确是有前提的**

例如 Unity 中的透明混合，默认采用 SrcAlpha, OneMinusSrcAlpha 这种方式，如果用符号表示出来是这样的：

$RGB_{result} = A_{src} * RGB_{src} + RGB_{dst} * (1-A_{src})$

对比之前给出的计算

$RGB'_{result} = A_{src} *RGB_{src}+ (A_{dst}*RGB_{dst} ) *(1-A_{src})$

加号右侧少乘了 $A_{dst}$ ，这是为什么呢？

**SrcAlpha, OneMinusSrcAlpha 正确的前提是混合目标是不透明的，即** $A_{dst}$ **为 1。**

平时渲染时，常见的情况是先渲染不透明物体，再渲染不透明的天空盒，最后再渲染半透明物体做 alpha 混合，在这种情况下渲染目标是不透明的，**不透明物体的有效颜色即其颜色本身**。

在满足这个前提下

$RGB_{result} = A_{src} * RGB_{src} + RGB_{dst} * (1-A_{src})$

才会成立，其本质为

$RGB’_{result} = A_{src} * RGB_{src} + (1.0 * RGB_{dst}) * (1-A_{src})$

依然是符合 tip1 中给出的结论

$RGB'_{result} = RGB'_{src}+ RGB'_{dst} * (1-A_{src})$

只不过此时的 $RGB'_{dst}$ 等于  $RGB_{dst}$ 。

可能有人会产生疑问，不透明背景上混合半透明后，怎么看待混合后的透明度？

我们回过头去看 tip1 中 alpha 的计算

$A_{result} = A_{src} + A_{dst} * (1-A_{src})$ ，

$= 1.0 * A_{src} + A_{dst} * (1-A_{src})$

$= lerp(A_{dst}, 1.0, A_{src})$

发现没有，**其本质是以 $A_{src}$ 作为参数的 $A_{dst}$ 到 1.0 线性插值**。当 $A_{dst}$ 为 1.0 时，无论 $A_{src}$ 是何值，最终输出都是 1.0。回到现实中，这很好理解，砖墙前放一块玻璃，当我们将玻璃和墙看作一个整体时，它们是不透明的。

## Tip3: SrcAlpha, OneMinusSrcAlpha 混合出来的 Alpha 值是无意义的

这种常见混合方式根据 tip2，其已默认渲染目标的 alpha 为 1，因此它不关心 alpha 结果的正确性。

根据其表达式

$RGB_{result} = A_{src} * RGB_{src} + RGB_{dst} * (1-A_{src})$

我们可以清晰的看到，这里没有出现 $A_{dst}$ ，得出正确的 RGB 与 RT 中的 alpha 存什么没有任何关联。

通过 SrcAlpha, OneMinusSrcAlpha 方式我们会计算得到

$A_{result} = A_{src} * A_{src} + A_{dst} * (1-A_{src})$

这个结果没有意义。**有些情况下，可以利用这种性质，将 RT 中没有被用到的 alpha 通道利用起来**，例如存储 bloom 系数。

**正因如此，想要单独渲染含半透的角色、特效到 RT，再混合到界面必须修改 Alpha 的计算方式，请看 tip4。**

## Tip4: 如何正渲染半透 RenderTexture

可以使用 《GPU Gems 3》 [Chapter 23. High-Speed, Off-Screen Particles](https://developer.nvidia.com/gpugems/gpugems3/part-iv-image-effects/chapter-23-high-speed-screen-particles) 中提到的反转 alpha 的方法，和本文的主题无关不展开讲了。另外可以从前往后排序用 "under" 混合也是可以的。

**最直接了当的方法就是采用 premultiplied alpha 的混合方式。**  

## Tip5: 理解预乘 Alpha 混合公式的颜色部分

Premultiplied alpha 混合采用 One, OneMinusSrcAlpha，其实我们在 tip1 中就已经看到了，

$RGB'_{result} = RGB'_{src}+ RGB'_{dst} * (1-A_{src})$

即

$RGB'_{result} = RGB'_{src} * 1.0 + RGB'_{dst} * (1-A_{src})$

One 就是这里的 1.0 而 OneMinusSrcAlpha 就是 $(1-A_{src})$ 。

$RGB'_{dst}$ 来自于混合的结果，真正留给我们的问题是 $RGB'_{src}$ 如何获得。

**最简单方式就是纹理中的 RGB 预乘好 alpha，那我们采样得到的颜色直接就是有效 RGB。**

**另外需要注意的是这里输出的 RGB 是预乘 alpha 的 RGB，详见 tip8。**

## Tip6: 纹理预乘 Alpha 实践上可能有潜在问题

在实践中，我们的纹理的数据源大多是 RGBA32，即单通道 8 比特，只能表示 0-255 的整数，同时游戏资产还会根据目标平台做纹理压缩。

**由于精度问题，原本相近的颜色在预乘后会存储为更相近，甚至相同的颜色，经压缩后很容易产生大量 artifacts。要使用预乘 alpha 的纹理，一般会建议采用单通道 16 位的存储。**

**由于这种情况，即使预乘有很好的纹理过滤特性，也没有被广泛采用**，我所了解 WebGL 由于网页对于 alpha composition 的天然需求，做了这方面的支持。  

## Tip7: 即使不纹理预乘，采用预乘 Alpha 的混合公式也有好处

**采用 One, OneMinusSrcAlpha 混合有个很好的特性，可以统一 Blend 和 Additive，减少 BlendState 切换，还能增加效果。**

推荐阅读 [https://amindforeverprogramming.blogspot.com/2013/07/why-alpha-premultiplied-colour-blending.html](https://amindforeverprogramming.blogspot.com/2013/07/why-alpha-premultiplied-colour-blending.html) ，这里简单介绍下思路：

*   把非预乘纹理的采样到的 RGBA，我们在 shader 中输出 (RGB*A, A) 就是 Blend 模式。
*   把非预乘纹理的采样到的 RGBA，我们在 shader 中输出 (RGB*A, 0) 就是 Additive 模式。

输出的 Alpha 可以定义一个 uniform t 控制，输出 (RGB*A, A*t)，这样通过 t 就是控制 Blend 和 Additive 模式之间的过渡。

如果再定义一个 uniform s，输出 (RGB*A*s, A*t*s)，我们还可以通过 s 控制其整体透明度，用于淡入淡出！简直就是特效的救星。

众所周知，采用 Additive 模式的特效，在亮的场景中几乎看不到效果，而 Blend 模式的特效在暗的场景中提不亮。采用 One OneMinusSrcAlpha 就可以使用中间态来做出适配比较好的特效，而且不需要 framebuffer fetch。

## Tip8: Premultiplied Alpha 运算是封闭的

换人话来讲，就是**预乘 alpha 混合得到的颜色也是预乘 alpha 的**。

细心的同学会注意到，在 tip1 中

$RGB'_{result} = RGB'_{src}+ RGB'_{dst} * (1-A_{src})$

作为运算结果的 $RGB'_{result}$ 是有 prime 符号的，正是想提示这一点。

最终输出的有效颜色来自两部分，

*   叠加上去的 src 像素贡献的有效颜色
*   背景 dst 像素贡献的有效颜色，它被 src 遮挡掉一部分，遮挡的量是 $(1-A_{arc})$

我们观察 tip1 中给出的两式

$$RGB'_{result} = A_{src} *RGB_{src}+ (A_{dst}*RGB_{dst} ) *(1-A_{src}) \tag{1}$$

$$RGB'_{result} = RGB'_{src}+ RGB'_{dst} * (1-A_{src}) \tag{2}$$

(1)(2) 的计算过程是一样的，这就不禁会产生疑问：(1) 式混合两个未预乘 alpha 的 RGB，结果是预乘 alpha 的 RGB？

这没错，**未预乘 alpha 的颜色经混合得到的是预乘 alpha 的颜色。**

**那平时用 SrcAlpha, OneMinusSrcAlpha 为什么能得到未预乘的结果呢？**

**原因正是 tip2。**

由于 SrcAlpha, OneMinusSrcAlpha 混合隐含了一个假设，渲染目标是不透明的。在这个前提下，用正确的混合公式计算，我们可以得到：

*   预乘了 alpha 的 $RGB’_{result}$
*   $A_{result} =1.0$

我们在 tip2 中已经讲过，与不透明目标混合得到的 alpha 恒为 1。

显而易见，当 alpha 为 1 时， $RGB’_{result}$ 等于 $RGB_{result}$ 。

因此 (1) 式在当渲染目标是不透明时，改成下式是成立的

$RGB_{result} = A_{src} *RGB_{src}+ (A_{dst}*RGB_{dst} ) *(1-A_{src})$

小结一下

*   预乘 alpha 的颜色经透明混合得到的颜色也是预乘 alpha 的
*   未预乘 alpha 的颜色经透明混合得到的是预乘 alpha 的颜色
*   SrcAlpha, OneMinusSrcAlpha 假设了渲染目标不透明，因此结果你可以说是预乘的，也可以说是没预乘的，因为两个值此时是一样的
*   如果渲染目标是半透明的，未预乘混合后需要 ”unmultiply” 才能恢复未预乘的颜色，详见 tip10

## Tip9: 理解预乘 Alpha 混合公式的 Alpha 部分

**预乘 Alpha 混合时，颜色分量和 Alpha 分量的运算是一致的。**对比一下

$RGB'_{result} = RGB'_{src}+ RGB'_{dst} * (1-A_{src})$

$A_{result} = A_{src} + A_{dst} * (1-A_{src})$

都是 $Result = Src + Dst * (1-A_{src})$ 。

因此，不需要额外指定 alpha 分量的混合公式，就能得到有意义的 alpha 值，而且**无论渲染目标是透明还是不透明，结果都是正确的。**（Tip4）

Tip2 和 tip3 中讲到过 SrcAlpha, OneMinusSrcAlpha 混合正确是有条件的，必须满足渲染目标是不透明的，而且得到的 alpha 也是无意义的。相比之下用 premultiplied alpha 做混合优势就太明显了。

**如果提前知道会往不透明渲染目标混合，那预乘 alpha 混合也可以借鉴 tip3 中的做法把 RT 的 alpha 通道利用起来。**

## Tip10: 仅当必要时 Unmultiply

凡是讲 premultiplied alpha 都会告诉你，可以通过

$RGB = \frac{RGB'}{A}$ 还原未预乘的颜色值，常见的、未预乘的颜色值也叫 straight alpha 或 unassociated alpha，而预乘好的叫 premultiplied alpha 或 associated alpha。这种还原操作在渲染自己可控的环境下几乎用不到。

根据 tip8，预乘 alpha 混合时运算封闭，可以多次混合不需要还原 straight alpha。

但如果用未预乘 alpha 混合时，如果渲染目标是半透明的，每次混合完成都要 unmultiply 回 straight alpha 才能继续混合，而且当一个网格有多层透明叠加时结果是错误的。

**从实践上讲，预乘 alpha 混合的结果需要 unmultiply 主要就这种情况：三方组件只接受 straight alpha 表示的纹理。**

Framebuffer 显示到屏幕上输出时 RT 最终总是不透明的，不透明的 alpha 为 1，预乘和未预乘没有区别，也不用特殊处理。

## Tip11：Bleed Alpha 与预乘 Alpha 原理不同，目的相同，结果略有不同

Alpha bleeding 的原理以前已经写过了

[undefined](https://zhuanlan.zhihu.com/p/340754532)

预乘 alpha 和 bleed alpha 目的都是减少半透明纹理过滤产生的瑕疵，但它们有一些比较显著的区别：

*   Bleed alpha 不需要修改混合公式
*   Bleed alpha 只能优化完全透明和非完全透明像素边缘的过滤瑕疵
*   预乘 alpha 不仅可以达到 bleed alpha 的结果，半透像素之间的过滤效果也能得到优化
*   预乘 alpha 需要修改混合公式，可能产生 tip6 中提到的情况

**当不使用 premultiplied alpha 时，预处理贴图 bleed alpha 是一个 “免费” 替代品。**虽然效果上会有折扣，但性价比极高。

## Tip12: 纹理预乘 Alpha 可以减少纹理过滤带来的 artifacts

介绍 premultiplied alpha 的文章大多都会从纹理过滤的角度讲，个人认为没有充分理解 alpha 混合的情况下，如果还结合着 SrcAlpha, OneMinusSrcAlpha 混合的简单认知去看 premultiplied alpha 很容易一头雾水。

本文已经够长了，应该没也几个人会耐心看完，

本文不再探讨纹理过滤的内容了，这里推荐两篇不错的文章

[Alpha Blending: To Pre or Not To Pre](https://developer.nvidia.com/content/alpha-blending-pre-or-not-pre)[Beware of Transparent Pixels - Adrian Courrèges](http://www.adriancourreges.com/blog/2017/05/09/beware-of-transparent-pixels/)

Nvidia 的这篇从 mipmap 的角度而 Adrian 的文章从双线性过滤的角度介绍了 premultiplied alpha 是如何减少半透纹理过滤带来的 artifacts。

小结一下，**纹理预乘 alpha 可以减少 downsampling、upsampling、非 pixel perfect 各种情况下半透纹理过滤产生的 artifacts。**

喜欢我的文章请点赞、收藏加关注，转发给有需要的朋友。