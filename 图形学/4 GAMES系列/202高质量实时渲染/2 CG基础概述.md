# Recap of CG Basis
本文是 games202 的 Lecture 2 的笔记, 如有任何错误之处请各位大佬指正.

[GAMES202 - 高质量实时渲染_哔哩哔哩 (゜ - ゜) つロ 干杯~-bilibili](https://www.bilibili.com/video/BV1YK4y1T7yY?p=2)

本课目录

• Recap of CG Basics

- Basic GPU hardware pipeline （复习 GPU 渲染管线）

- OpenGL and OpenGL Shading Language (opengl 和对应的着色语言 --GLSL)

- The Rendering Equation

- 相关微积分知识

**I -> Graphics (Hardware) Pipeline**

首先我们对渲染管线进行一个复习:

我们知道物体在最开始是 3D 的模型, 我们经过渲染后最终会得到一张图, 而这中间经过了一些系列的操作, 这个过程我们称其为渲染管线 (渲染流水线).

渲染管线的一系列操作是在现代 GPU 上完成的, 因为 GPU 完成或者说是计算的快, 因为 GPU 的并行度高.

![[a6b019049cea49696ea52cd28f0ffac2_MD5.jpg]]

3D 物体在空间中以**一堆点和点的连接关系**来表示, 每个点都会经过第一个步骤**顶点处理,** 也就是经过一系列的变换, 如 **MVP 变换**,**ViewPoint 变换**, 从而变换成屏幕上的一个点.

![[eb4772d42de58ce3f96ca0f3a53f0f3e_MD5.jpg]]

经过这一些系列的变换之后, 各点之间的连接关系是不会变换的, 只是点被投影到了屏幕上, 因此变换前的三个点所组成的三角形在变换后仍由这三个点组成三角形.

三角形投影到屏幕上后我们要进行光栅化操作来将三角形离散成一堆**像素 (或者是 fragments).**

![[516aeadd5544f1f9c7a5a805c47874f7_MD5.jpg]]

在打散成像素的过程中我们要利用深度缓存来处理遮挡问题.

![[edc46c81dddc8b1cb477b06b22be416a_MD5.jpg]]

接着我们进行着色处理, 也就是计算它应该长什么样子, 比如布林冯着色模型.

**布林冯是一个经验式的模型, 因此它并不是完美的, 它对全局或者间接光照是一个近似的做法, 它对全局的一些现象处理的不好, 比如阴影, 光线的多次弹射.**

![[32a4ee699d9b669b932099106cb14d26_MD5.jpg]]

在着色的过程中, 我们可以在三角形内部或者是整个物体上的任意一个地方得到一个纹理的坐标, 从而把一张图给贴在上面, 或者根据它的纹理坐标, 任一个位置我们都可以知道在纹理的哪个地方去查询.

同时我们还提到了一个叫**插值**的概念, 就是说我知道三角形三个顶点对应的纹理坐标, 我们可以通过插值来在三角形内部的任一位置得到一个平滑的纹理过渡.

最终我们得到一张成图.

**II -> OpenGL**

首先什么是 Opengl:

opengl 是一系列在 cpu 端上负责调动 gpu 的 api，也就是在 CPU 端写的, 我们可以知道 opengl 用什么语言写是没有关系。

我们更关心的是 GPU 怎么去执行, 而不是 CPU 如何让 GPU 去执行.

![[e8d28986444c1501ad494492bff55431_MD5.jpg]]

**OpenGl 的一个特点:**

可以跨平台。

**OpenGl 的缺点:**

1.  版本过于碎片化
2.  c 风格语言，不是很好用
3.  几年前不好 debug，现在方便很多

**OpenGL 就像画油画:**

我们要渲染一个 3D 的场景, 我们类比画油画的各个步骤:

1.  **摆好物体和模型**

![[7e23ad5873cbb57ae204d25e73d505a7_MD5.jpg]]

在摆好物体和模型之前我们会有两个问题:

**a) 我们需要告诉 GPU 我们要用什么样的模型**

在 OPENGL 种我们通过 VBO 我们可以把需要渲染的图元的顶点信息，直接上传存储在 GPU 的显存中.

VBO 就是 GPU 种一块用来存储你的模型的区域, 模型的存储方式与存储三角形的 OBJ 方式相似, 存储一堆 vertices,normals,texture coords 等以一定的格式组织了一个物体应该放在 GPU 的何处.

**b) 模型该如何摆放**

在 101 中我们需要自己手写各种的矩阵来进行物体的运动, 而在 OPENGL 中内置了这些函数, 我们只需要调动函数写入参数即可, 不用再自己去写函数.

**2. 架好画架**

![[d5491109a8e3a5590744ac3d11475c69_MD5.jpg]]

架好画架也会遇到两件事:

**a) view transformation(视图变换)**

首先放置相机，视图变换就是相机选用的是透视投影还是正交投影, 在 101 中仍是我们自己去推导和写的矩阵, 而 opengl 的 api 简化了摄像机的视图变化, 让一切变得简单了许多.

我们只需要规定相机的一些属性就可以了, 比如说:

fov(可视角度),aspect(长宽比),zNear(近平面),zFar(远平面).

**b) creat / use a framebuffer(建立画架)**

要建立 opengl 的画架，就是 framebuffer, 为了要使用一个画架, 我们一定要在上面贴一块画布的, 因此到第三步.

**3. 在画架上贴上一块画布**

![[4174c6c7e920e1513681c749fb13594a_MD5.jpg]]

frambuffer 对场景渲染一次，可以用渲染出好几张不同性质的纹理,（也解释了第 5 点画好一张后换张画布可以继续画）, 最后由 fragment shader 告诉你要写到哪一个纹理上去。

也就是你从一个角度看过去渲染一次场景可以输出好几张不同的图, 可能有一张是 shading 的结果, 有一张是深度.

例如这几张图就是通过 MRT 在对场景渲染一次后得到的四张不同性质的图:

![[51ccb14d3e06ae838b43d230205272f1_MD5.jpg]]

![[a59dc913fa1015af3bc0fff47d1e864b_MD5.jpg]]

我们也是可以直接把 framebuffer 渲染的结果显示到屏幕上的, 但是直接把 frambuffer 的渲染结果放在屏幕上会造成屏幕撕裂，因为你这一帧还渲染一半, 下一帧就开始渲染覆盖你这一帧所看到的内容, 从而导致了屏幕撕裂.

在游戏中时常有一个叫做垂直同步的设置能解决这个问题。或者先把渲染好的结果存储在一个纹理或者缓冲区里，确定没问题在放在屏幕上，这种方法叫做双重缓冲，更复杂的还有三重缓冲。

**4. 在画布上绘画**

我们要把场景给画在画布上, 那么无论如何我们肯定要进行 shading, 那么:

如何做 shading 呢?(我们在 101 中已经讲的很清楚了)

课程中只会用到顶点着色和像素着色。

顶点着色的话, 每个顶点的要在顶点着色器操作，首先要进行 mvp 的投影, 之后我们将需要做插值的值进行插值得到结果后, 再将结果传递到像素着色器。

那么像素着色器得到的输入就是顶点着色器的输出在一个顶点上的属性, 接着就会被插值到任何一个像素上的属性.

opengl 会将三角形打成一堆像素，然后对每个像素着色。

这门课程最重要的就是我们如何去写顶点着色器和片段着色器.

我们来做一个简单的总结:

![[f95d75d1787e6cd5a455657f79869f5f_MD5.jpg]]

Opengl 就是告诉 GPU 在每一趟渲染中需要:

**- 指定物体，相机，mvp 矩阵等**

**- 指定 Framebuffer 和输 / 出纹理**

**- 指定顶点、fragment 着色器**

**-（当在 gpu 里确定了一切）渲染**

**6. 在画新图时可以采用或参考旧画上的内容**

之前渲染的纹理可以给之后渲染参考（多 pass 渲染).

那么为什么场景要渲染很多次?

我们以 shadow map 为例, 他是一个典型的两趟做法, 从 light 和 camera 分别去看场景, 因此在 light 得到的深度这么一个纹理, 我们可以在 camera 时候再去用它.

**III -> OpenGL Shading Language (GLSL)**

顾名思义就, 描述着色器怎么操作的语言就是着色语言，着色语言风格偏 c 语言。

**怎么使用 shader**

首先要写 shader, 然后让 opengl 去编译 shader，但是你可能会比编译 fragment shader 或者是 vertice shader, 因此可以建立 program 集合了你写的所有自定义 shader，再做一个链接，最后使用链接好的 program 渲染。

写 shader 可以近似于在 cpu 上编程, 二者十分相似。

![[926ded9c5547536081b1893c260596e2_MD5.jpg]]

**Vertex shader**

attribute（顶点属性关键字只在 vertex shader 出现） vec3（3 维向量） aVertexPosition(获取顶点位置)

vec4(aVertexPosition,1.0) 定于一个 4 维向量只要 3 维向量后面跟一个数

varying（需要传递到 fragment shader) high（定义计算精度） vec2 vNormol;

uniform(全局变量）

![[b2571db892679331742f2bee9422b6b1_MD5.jpg]]

**Fragment Shader**

color =pow(texture2D(uSampler,vTextureCoord).rgb,vec3(2.2)(伽马矫正))

gl_FragColor(输出颜色）=......

![[19db01f9e7cf4997600b70fd9306962b_MD5.jpg]]

![[e8355faf7dac797a86a32f74f982b00d_MD5.jpg]]

**Debug**

早年 debug 只有 NVIDA Nsight 能调试 glsl，hlsl 只能在软件上运行来 debug；

Now

-Nsight Graphics，跨平台，但只支持 NVIDA。

-RenderDoc，对显卡品牌没要求。

RGB 调试法

**The Rendering Equation**

在 games101 中, 我们整个 path tracing 的体系是建立在 rendering equation 上的, 因为它是一个正确的用来描述光线传播的一个方法.

![[44aac5a615536e795c4e91d9a4b368f0_MD5.jpg]]

Rendering Equation 是一个正确的用来描述光线传播的等式，讲的是:

看到的任何一个点 p 反射到眼中的 radiance = 这个点 p 本身发出的 radiance + 其他打到这个点的 radiance * brdf * cos

![[d172fdd13479f3554c65e05b8541e3ba_MD5.jpg]]

我们在指 brdf 时候可能指原始 brdf, 也可能指 cosine-weighted brdf

Visibility 是在实时渲染中我们需要考虑到物体会不会被光源找到，比如在一点 p 往一方向去看, 我们知道这一方向有光源发出光线的, 但是光线能否打到 p 上, 因此 引入 Visibility 的概念。

real time rendering 会显式的考虑 visibility, 他与原先的理解是等价的, 之所以这么理解是为了能够更好理解环境光照, 要比不写 visilibity 项更加直观.

![[6c4f7ba802600766848f36dd74f781a0_MD5.jpg]]

任何一方向过来的光的强度由一张图来决定, 可以一个是 cube box 也可以是定义在球上的一张图, 这两种表示都有不同的问题, 在本课中会介绍一种新的表示, 以八面体来表示.

此时只需要考虑说, 从任何一个 shading point 往那个方向去, 看这个 visility 是否可见, 等于将实际上的光源和能否看到他, 把这两项拆开考虑了, 就很方便.

**间接光照**

光线会进行弹射，形成间接光照，将间接光照加加直接光照上就是全局光照，实现全局光照就是要解决间接光照。

![[1f9ed9d0a960f1c97ebf64f72cee4c92_MD5.jpg]]

![[58740e183fc8e01ae09830b32499e9a6_MD5.jpg]]

![[13ce5bb50e94e8cd30c6ea0b73a669ba_MD5.jpg]]

我们可以看出, 图像越亮越 NB.

课堂答疑整理 (此处是复制的祭曹大佬的整理)

[祭曹：GAMES202 Real-Time High Quality Rendrting （高质量实时渲染）课程笔记 Lecture 2: Recap of CG Basics](https://zhuanlan.zhihu.com/p/358615084)

1. 问：怎么定义哪些点连接成三角形? 答：比如 obj 格式是先编号一系列点，再定义面，每个面都带着三个点的编号的下标。

2. 问：有无适合全局光照的管线？答：无，但光线追踪渲染管线很完善。

3. 问：纹理坐标怎么参数化的？答：可以理解成物体外面有个盒子，把盒子挤压到物体上，盒子上的 uv 是好确定的。

4. 问：几个光源就需要几个 shadowmap 吗？光源也需要深度测试吗？答：是

5. 问：opengl 支持 optix 吗？答：可以，但不要指望 shader 能调用一个光线一个场景怎么运作。

6. 问：纹理是不是是个 buffer？答：opengl 里两者定义不一样，但也可以这样理解，都是显存里的一块缓存区域。

7. 问：一个 pass 就是一个 frambuffer 渲染一次吗？答：一个场景渲染一次

8. 问：不同 shader 里定义里的变量会通用吗？答：不是