
本课目录

• Recap of CG Basics

- Basic GPU hardware pipeline （复习 GPU 渲染管线）

- OpenGL and OpenGL Shading Language (opengl 和对应的着色语言 --GLSL)

- The Rendering Equation

- 相关微积分知识


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

gl_FragColor(输出颜色）= ......

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