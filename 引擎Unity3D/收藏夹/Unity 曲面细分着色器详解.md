Unity 版本：2022

URP 版本：14.0.0

项目链接：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry)

## 概述

**曲面细分阶段 (tessellation stages)** 是利用镶嵌化处理技术对网格中三角形进行**细分 (subdivide)**，以此来增加物体表面上的三角形数量。再将这些新的三角形偏移到适当的位置，使网格表现出更加细腻的细节。

曲面细分阶段是可选阶段，使用它能带来如下的好处：

*   实现**细节层次 (level-of-detail, LOD)** 机制。
*   在内存中仅维护**低模 (low-poly)** 网格，再根据需求为它动态地增添额外的三角形，以此节省内存资源。
*   处理动画和物理模拟时采用简单的低模网格，而仅在渲染过程中使用经镶嵌化处理地高模 (high-poly) 网格。

在本文中，我会从渲染管线入手，讨论 hlsl 中曲面细分着色器在 Unity 里的写法。并且给出几个例子帮助理解。

## 曲线与曲面

以便更好的理解曲面着色器，我们需要先了解一些几何相关的知识，这些概念和原理有助于理解曲面细分着色器是如何进行的。关于曲线与曲面，可以阅读 RTR4 第 17 章，或者参考我的文章：

[TheTus：计算机图形学：曲线与曲面](https://zhuanlan.zhihu.com/p/629202115)

我们简单提炼几个几何中的关键概念：

*   **控制点 (control point)**：控制点是定义曲面形状的关键元素。它们是二维或三维空间中的点，用于确定曲面的控制网格。一般来说，曲面通过关于控制点的方程计算生成。
*   **面片 (patch)**：对于一个参数曲面 $p(u,v)$ ，如果定义域 $(u,v)$ 为矩形，则称该曲面为 patch。
*   **镶嵌 (tessellation)**：在实时渲染中，我们需要计算并创建 (多个) 三角形对真实曲面进行拟合，这个过程称为镶嵌。在运行时，表面可以被镶嵌为多个小三角形。

## 渲染管线

建议在这之前先梳理一遍图形渲染管线，可以参考我的文章：

[TheTus：图形渲染管线详细梳理](https://zhuanlan.zhihu.com/p/627201581)

将渲染管线按功能性阶段划分，曲面细分着色器处于几何处理阶段的可选顶点处理阶段。

![[7fc976c905dd164642fb1259b29b677e_MD5.jpg]]

将渲染管线按 GPU 逻辑管线划分，几何着色器在顶点着色器阶段之后，光栅化 (和几何着色器) 阶段之前。

![[b2149ed3f4ef0620573e18407b4bb84b_MD5.jpg]]

在一般情况下，顶点着色器向下一个阶段输出三角形顶点数据。然而，在曲面细分着色器开启后，可以理解为 **输入装配阶段 (IA)** 向顶点着色器提交的是具有若干 **控制点 (control point)** 的**面片 (patch)**，而顶点着色器传递给曲面细分着色器的为这些控制点的数据。在曲面细分着色器中，一个 patch 通常会被 **镶嵌化 (tessellation)** 为更多的子 patch。这个细分过程中使用子 patch 的顶点带入控制点的曲面方程，以生成曲面上的顶点，进而拟合真实曲面。

也就是说，当开启曲面细分后，顶点着色器就变成了 “处理控制点的着色器”。正因如此，我们还能在曲面细分开展之前，对控制点进行一些调整。一般来讲，动画与物理模拟的计算工作都会在对几何体进行镶嵌化处理之前的顶点着色器中以较低的频次进行。

在光栅化阶段之前，顶点位置数据需要变换到 **齐次裁剪空间** 。如果开启几何着色器，这将在几何着色器中进行。否则曲面细分着色器将完成这个任务。

需要注意的是，顶点数据的插值在光栅化阶段前，曲面细分着色器 (几何着色器) 后进行。顶点着色器仅将控制点数据传递给曲面细分着色器。

![[3316e9b6682e5520c5e63386d9a527c7_MD5.jpg]]

## 曲面细分着色器

曲面细分着色器又分为 **外壳着色器 (Hull Shader)** ，**镶嵌器阶段（Tessellation Stage)** 和 **域着色器 (Domain Shader)** 三个子阶段。下面我们一一讨论。

![[5a01668b34084b4417a94bdb1a1b21bf_MD5.jpg]]

### 外壳着色器 (Hull Shader)

外壳着色器接收顶点着色器传递的控制点数据，向镶嵌器 (tessllator) 输出常量细分因子，向域着色器传递经过变换和增删后的控制点数据。

![[ac3f411e4ea2eee9cd83fa9dc4da417d_MD5.jpg]]

外壳着色器实际上由两种着色器组成：常量外壳着色器和控制点外壳着色器。注意，这两个 phase 在硬件上 **并行 (parallel)** 执行。

**常量外壳着色器**

**常量外壳着色器 (constant hull shader)** **逐 patch** 调用，即每处理一个 patch 就被调用一次。它的输入是顶点着色器传递的该 patch 的控制点数据，并向镶嵌器 (tessellator) 输出**曲面细分因子 (tessellation factor)**。曲面细分因子指示在曲面细分阶段中将 patch 镶嵌处理后的份数。

下面是一个具有 3 个控制点的三角形面片示例，我们将它从各个方面均匀地镶嵌细分为 3 份。

```
struct PatchTess{
    float edgeTess[3] : SV_TessFactor;
    float insideTess : SV_InsideTessFactor

    // 可以在下面为每个买牛排你附加所需的额外信息 例如额外的控制点等
};

PatchTess ConstantHS(InputPatch<VertexOut, 3> patch,// 处理3个控制点的面片
    unit patchID : SV_PrimitiveID){
    PatchTess pt;

    // 将该面片从各方面均匀地镶嵌处理为3等份
    pt.edgeTess[0] = 3;// 三角形面片的左侧边缘
    pt.edgeTess[1] = 3;// 三角形面片的上侧边缘
    pt.edgeTess[2] = 3;// 三角形面片的右侧边缘
    pt.edgeTess[3] = 3;// 三角形面片的下侧边缘

    pt.insideTess = 3;// 三角形内部细分的份数

    return pt;
}
```

常量外壳着色器以面片的所有控制点作为输入，在此用`InputPatch<VertexOut, 4>`对此进行定义。常量外壳着色器接受的是顶点着色器传递的控制点，因此它们的类型由顶点着色器输出类型`VertexOut`来确定。在此例中，我们的三角形面片拥有 3 个控制点，所以就将`InputPatch`模板的第二个参数指定为 3. 系统还通过`SV_PrimitiveID`语义提供了面片的 ID 之，此 ID 值唯一地标识了绘制调用过程中的各个面片。

常量外壳着色器必须输出曲面细分因子，该因子取决于面片的拓扑结构。常量外壳着色器阶段对所有输入和输出控制点具有**只读访问权限**。另外，还可以给输出结构体添加额外的信息，例如可以指定额外的控制点等。

对 **三角形面片 (triangle patch)** 执行镶嵌化处理的过程分为两部分：

*   3 个边缘曲面细分因子控制对应边上镶嵌后的份数
*   一个内部曲面细分因子指示着三角形面片内部的镶嵌份数

对 **四边形面片 (quad patch)** 进行镶嵌化处理的过程同样分为两部分：

*   4 个边缘曲面细分因子控制着对应边缘镶嵌后的份数
*   两个内部曲面细分因子指示如何来对该四边形面片内部进行镶嵌化处理（一个指示横向维度，一个指示纵向维度）

一般硬件支持的最大曲面细分因子为 64。若指定的曲面细分因子为 0，则该 patch 将被剔除，不会送到后面的阶段。这使得我们可以在常量外壳着色器执行 **视锥体剔除** 和 **背面剔除** 等优化手段。

另外，常量着色器中定义的曲面细分因子可以是动态的，以下是一些确定镶嵌次数的常用衡量标准：根据与摄像机之间的距离；根据占用屏幕的范围；根据三角形的朝向；根据粗糙程度。在 Unity URP 包中附带了前两种的算法，我们后面再讨论。

有关细分因子如何镶嵌化 patch，我们在镶嵌化阶段进行讨论。

**控制点外壳着色器**

**控制点外壳着色器 (control point hull shader)** **逐控制点** 调用，即顶点着色器每输出一个控制点，此着色器就会被调用一次。它用来对控制点进行进一步转换，并且可以增加或减少控制点的数量。例如在 **PN 三角形** 中，可以利用控制点外壳着色器将一个 3 个控制点的三角形网格转换为具有 10 个控制点的贝塞尔曲面片。

下面是一个控制点外壳着色器仅仅充当一个简单的 **传递着色器 (pass-through shader)** 的例子，它不会对控制点进行任何的修改。

```
struct HullOut{
    float3 positionOS : TEXCOORD0;
}

[domain("tri")]
[partitioning("interger")]
[outputtopology("triangle_cw")]
[outputcontrolpoints(4)]
[patchconstantfunc("ConstantHS")]
[maxtessfactor(64.0)]
HullOut HS(InputPatch<VertexOut, 3> input, // 处理3个控制点的面片
    uint controlPointId : SV_OutputControlPointID, uint patchId : SV_PrimitiveID){
    HullOut output;

    output.positionOS = input[i].positionOS;

    return output;
}
```

通过`InputPatch`参数可以将 patch 所有控制点都传至外壳着色器之中。系统值`SV_OutputControlPointID`索引的正是在被外壳着色器所处理的输出控制点。输入控制点和输出控制点数量**未必**相同。例如，可以输入 4 个控制点，输出 16 个控制点（话虽这么说，我并没找到如何在 HS 中真正意义上的衍生控制点，一般都是塞进 uniform 数据和顶点数据了）。

一个控制点外壳着色器还需要定义如下属性：

*   domain：patch 的类型。可选用的参数有 tri(三角形面片)、quad(四边形面片) 或 isoline(等值线)。
*   partitioning：曲面细分的模式。
*   interger：新顶点的添加或一处仅取决于曲面细分因子的整数部分，**向上取整**。这样一来，在网格随着曲面细分级别而改变时，会容易发生明显的突跃 (popping) 情况。
*   非整数曲面细分 (fractional_even/fractional_odd)。新顶点的添加或移除取决于曲面细分因子的整数部分，但是细微的“渐变” 调整就要根据因子的小数部分。具体见后面的图片。
*   outputpology：通过细分所创的三角形的绕序
*   triangle_cw：顺时针方向的绕序
*   triangle_ccw：逆时针方向的绕序
*   line：针对线段曲面细分
*   outputcontrolpoints：外壳着色器执行的次数，每次执行都输出 1 个控制点。系统值`SV_OutputControlPointID`给出的索引标明当前正在工作的外壳着色器所输出的控制点。
*   patchconstantfunc：指定常量外壳着色器函数名称的字符串。
*   maxtessfactor：告知驱动程序，用户在着色器所用的曲面细分因子的最大值。一般硬件最大值为 64。

下面进一步讨论不同的细分因子。

interger 表明边缘上的所有镶嵌段长度都相等，向上取整。

![[30fd17d818e177aba75fba36f7a7ca21_MD5.gif]]

fractional_even 表明向上取最近的偶数 n，并将整段分为 n-2 个相等长度的整数部分，和两端较短的小数部分。

![[1b9cc2e89c92b0014a8ba27a91e8dc02_MD5.gif]]

fractional_even 表明向上取最近的奇数 n，并将整段分为 n-2 个相等长度的整数部分，和两端较短的小数部分。

![[0742cd9f509d67a06a35ed4e42bd82df_MD5.gif]]

通常使用 fractional_odd 模式，因为它可以处理 1 的因数，而 fractional_even 模式则被迫使用最小级别 2。

### 镶嵌器阶段 (Tessellator Stage)

在实时渲染中，我们需要计算并创建 (多个) 三角形对真实曲面进行拟合，这个过程称为镶嵌。在运行时，表面可以被镶嵌为多个小三角形。

镶嵌器是一个**固定功能阶段**， 这意味我们无法对这一阶段进行任何控制，它全权交由给硬件处理。镶嵌器阶段接收常量外壳着色器输出的曲面细分因子，对面片进行镶嵌化处理。然后，它将镶嵌后生成的顶点传递给域着色器。

对于三角形面片，常量着色器的边缘细分因子分别指示右 / 下 / 左边的段数，内部细分因子指示三角形各边中线的段数。

![[006a25fd43a2e925bcbe5478c572a470_MD5.jpg]]

对于四边形面片，常量着色器的边缘细分因子分别指示左 / 上 / 右 / 下边的段数，内部细分因子分别指示横向和纵向中线的段数。

![[6c2ff1467762965e072debc6b158739f_MD5.jpg]]

### 域着色器 (Domain Shader)

域着色器接收镶嵌器阶段输出的所有顶点与三角形和控制点外壳着色器输出的经过变换后的控制点。在**镶嵌器阶段中创建的顶点**，都会逐一调用域着色器进行后续处理。随着曲面细分功能的开启，顶点着色器便化为 “处理每个控制点的顶点着色器”，而域着色器的本质实为 “针对已经过镶嵌化的面片进行处理的顶点着色器”。特别的是，我们可以在此将经镶嵌化处理的面片顶点变换到齐次裁剪空间（如果打开几何着色器，则在几何着色器进行）。

![[138f09a14aab6998abb1cd66517b2749_MD5.jpg]]

对于三角形面片来讲，域着色器以曲面细分因子、控制点外壳着色器所输出的所有面片控制点以及镶嵌化处理后的顶点位置重心插值坐标 (u,v,w) 作为输入。 是否利用这些参数坐标以及控制点来求取真正的 3D 顶点位置，完全取决于用户自己。 下面代码中，根据控制点外壳着色器输出 patch 的控制点和镶嵌器提供的重心插值坐标计算当前镶嵌器阶段创建顶点的真正 3D 坐标，并变换到齐次裁剪空间中，最后将数据传输给顶点着色器。

```
struct DomainOut {
    float4  positionCS : SV_POSITION;
};

[domain("tri")]      
DomainOut DS (PatchTess patchTess, float3 bary : SV_DomainLocation, 
const OutputPatch<HullOut, 3> patch) {// 处理3个控制点的三角面片
    DomainOut output;

    float3 positionOS = patch[0].positionOS * bary.x + patch[1].positionOS * bary.y + patch[2].positionOS * bary.z; 
    output.positionCS = TransformObjectToHClip(positionOS);

    return output; 
}
```

对于四边形面片，不同地是用镶嵌化处理后的顶点位置参数坐标 (u,v) 作为输入。它的用法类似于纹理线性过滤的双线性插值。

```
struct DomainOut {
    float4  positionCS : SV_POSITION;
};

[domain("quad")]      
DomainOut DS (PatchTess patchTess, float3 bary : SV_DomainLocation, 
const OutputPatch<HullOut, 4> patch) {// 处理4个控制点的四边形面片
    DomainOut output;

    float3 v1 = lerp(patch[0].positionOS, patch[1].positionOS, uv.x);
    float3 v2 = lerp(patch[2].positionOS, patch[3].positionOS, uv.x);
    float positionOS = lerp(v1, v2, uv.y);
    output.positionCS = TransformObjectToHClip(positionOS);

    return output; 
}
```

### 总结

![[5a01668b34084b4417a94bdb1a1b21bf_MD5.jpg]]

曲面细分着色器分为三个子阶段，hull shader，tessellator 和 domain shader。顶点着色器将控制点输出给 hull shader，它逐控制点将变换后的控制点传递给 domain shader，逐 patch 将细分因子传递给 tessellator。Tessellator 接收 hull shader 传递的 patch 类型和曲面细分因子，将 patch 镶嵌为更多的子 patch，并将生成的顶点传递给 domain shader。domain shader 接收 hull shader 传递的控制点和细分因子，以及 tessellator 生成的顶点。tessellator 每生成一个顶点，domain shader 调用一次。domain shader 对顶点进行最终的变换，并输出给下一个阶段。

## 例子

### Flat Tessellation

![[ac072547df6ee35fe438829e694870a5_MD5.gif]]

下面，使用最基础的曲面细分着色器为例子。它仅仅在曲面细分着色器中对 patch 进行镶嵌化，其他着色器多充当传递着色器的作用。另外，为了观察具体的细分，加入 Geometry Shader 进行三角形描边，具体算法参考 [catlikecoding](https://catlikecoding.com/unity/tutorials/advanced-rendering/flat-and-wireframe-shading/)。

主要代码如下：

```
// 顶点着色器：接收IA控制点数据，向Hull Shader传递控制点数据
VertexOut FlatTessellationPassVertex(Attributes input) {
    VertexOut output;

    output.positionOS = input.positionOS;

    return output;
}

// Constant Hull Shader：接收顶点着色器传递的patch，向Tessellator传递细分因子
PatchTess FlatTessellationPassConstantHull(InputPatch<VertexOut, 3> patch, uint patchID : SV_PrimitiveID) {
    PatchTess pt;

    pt.edgeTess[0] = _TessFactor.x;
    pt.edgeTess[1] = _TessFactor.y;
    pt.edgeTess[2] = _TessFactor.z;
    pt.insideFactor = _TessFactor.w;

    return pt;
}

[domain("tri")]
#if defined(_PARTITIONING_INTERGER)
[partitioning("integer")]// 整数分割 向上取整
#elif defined(_PARTITIONING_FRACTIONAL_ODD)
[partitioning("fractional_odd")]// 奇数分割
#elif defined(_PARTITIONING_FRACTIONAL_EVEN)
[partitioning("fractional_even")]// 偶数分割
#else
[partitioning("integer")]// 整数分割 向上取整
#endif
#if defined(_ORDER_CW)
[outputtopology("triangle_cw")]
#elif defined(_ORDER_CWW)
[outputtopology("triangle_ccw")]
#else
[outputtopology("triangle_cw")]
#endif
[patchconstantfunc("FlatTessellationPassConstantHull")]
[outputcontrolpoints(3)]
[maxtessfactor(64.0)]
// Control Point Hull Shader：接收顶点着色器传递的控制点，对控制点进行变换后输出给Domain Shader
HullOut FlatTessellationPassHull(InputPatch<VertexOut, 3> patch, uint id : SV_OutputControlPointID) {
    HullOut output;

    output.positionOS = patch[id].positionOS;

    return output;
}

// Domain Shader：接收Tessellator镶嵌化后的patch和常量外壳着色器输出的细分因子，将曲面顶点传递给下个阶段
[domain("tri")]
DomainOut FlatTessellationPassDomain(PatchTess patchTess, float3 bary : SV_DomainLocation, OutputPatch<HullOut, 3> patch) {
    DomainOut output;
    output.positionOS = patch[0].positionOS * bary.x + patch[1].positionOS * bary.y + patch[2].positionOS * bary.z;

    return output;
}
```

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/FlatTessellation)

Flat Tessellation 通常与位移贴图一起使用，具体教程可以参考 [catlikecoding](https://catlikecoding.com/unity/tutorials/advanced-rendering/surface-displacement/)。

![[9dd32c104b85ad147693b2457172acdd_MD5.jpg]]

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/Displacement)

### 三次贝塞尔曲面

![[398d8ae3f47d9b72c266039d4749e90e_MD5.gif]]

下面通过贝塞尔曲面的例子来理解控制点与曲面的作用，并为后面的 Phong Tessellation 铺垫。有关贝塞尔曲面的详细几何计算，可以参考我的文章：

[TheTus：计算机图形学：曲线与曲面](https://zhuanlan.zhihu.com/p/629202115)

首先，我们需要在 CPU 端通过 C# 脚本向 Shader 传递控制点数据。由于 Unity 支持的图元拓扑类型仅仅只有点、线、三角形和四边形，我们无法直接创建控制点类型的图元拓扑。所以这里采用创建一个 4 控制点 quad 的图元，然后将剩余控制点通过 uniform 数组的方式传递给 GPU。 C# 脚本：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/blob/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Scripts/BezierGenerator.cs)

在顶点着色器中，我们仅仅将 quad 的控制点传给 Hull Shader。

```
// Vertex Shader：将quad的4个control point传给Hull Shader
VertexOut QuadTessPassVertex(Attributes input) {
    VertexOut output;

    output.positionOS = input.positionOS;

    return output;
}
```

在 Hull Shader 中，Constant Hull Shader 定义细分因子，Control Point Hull Shader 仅仅传递 quad 的控制点。注意，这里并没有在 control point hull shader 里创建额外的控制点，因为它被存在全局数组里供 Domain Shader 使用。

```
// Constant Hull Shader: 定义quad的细分因子，传递给Tessellator
PatchTess QuadTessPassConstantHull(InputPatch<VertexOut, 4> patch, uint patchID : SV_PrimitiveID) {
    PatchTess pt;

    // Uniform Tessellation
    pt.edgeTess[0] = _EdgeTess;
    pt.edgeTess[1] = _EdgeTess;
    pt.edgeTess[2] = _EdgeTess;
    pt.edgeTess[3] = _EdgeTess;

    pt.insideTess[0] = _InsideTess;
    pt.insideTess[1] = _InsideTess;

    return pt;
}

// Control Point Hull Shader: 仅传递Vertex Shader传递的quad的control point给Tessellator
[domain("quad")]
[partitioning("fractional_even")]
[outputtopology("triangle_cw")]
[outputcontrolpoints(4)]
[patchconstantfunc("QuadTessPassConstantHull")]
[maxtessfactor(64.0f)]
HullOut QuadTessPassHull(InputPatch<VertexOut, 4> patch, uint id : SV_OutputControlPointID) {
    HullOut output;

    output.positionOS = patch[id].positionOS;

    return output;
}
```

在域着色器中，我们用矩阵形式表示三次伯恩斯坦系数，然后用这些系数和控制点的方程计算出 Hull Shader 传递的该 quad 顶点的在曲面上的坐标。偏导数同理，法线由两个偏导数叉乘得到。

```
// 计算伯恩斯坦基函数的4个系数（三阶）
float4 BernsteinBasis(float t) {
    float invT = 1.0f - t;

    return float4(invT * invT * invT,
                  3.0f * t * invT * invT,
                  3.0f * t * t * invT,
                  t * t * t);
}

// 通过伯恩斯坦系数计算控制点坐标
float3 CubicBezierSum(float4 basisU, float4 basisV) {
    float3 sum = float3(0.0f, 0.0f, 0.0f);
    sum = basisV.x * (basisU.x * _ControlPoints[0] + basisU.y * _ControlPoints[1] + basisU.z * _ControlPoints[2] + basisU.w * _ControlPoints[3]);
    sum += basisV.y * (basisU.x * _ControlPoints[4] + basisU.y * _ControlPoints[5] + basisU.z * _ControlPoints[6] + basisU.w * _ControlPoints[7]);
    sum += basisV.z * (basisU.x * _ControlPoints[8] + basisU.y * _ControlPoints[9] + basisU.z * _ControlPoints[10] + basisU.w * _ControlPoints[11]);
    sum += basisV.w * (basisU.x * _ControlPoints[12] + basisU.y * _ControlPoints[13] + basisU.z * _ControlPoints[14] + basisU.w * _ControlPoints[15]);

    return sum;
}

// 计算贝塞尔系数的导系数
float4 dBernsteinBasis(float t) {
    float invT = 1.0f - t;

    return float4(-3 * invT * invT,
                  3 * invT * invT - 6 * t * invT,
                  6 * t * invT - 3 * t * t,
                  3 * t * t);
}

// Domain Shader: 逐patch处理Tessllator传递的细分后的patch顶点
// 对于quad 输入的是uv坐标(顶点patch位置uv，而非纹理uv)
[domain("quad")]
DomainOut QuadTessPassDomain(PatchTess patchTess, float2 uv : SV_DomainLocation, const OutputPatch<HullOut, 4> patch) {
    DomainOut output;

    // 由于uv in [0, 1]线性，刚好对应Bezier Curve的t，所以直接将uv带进Bernstein公式得到两轴系数，然后再将系数组合为Bezier Curved Surfaces

    // 计算曲面点坐标
    float4 basisU = BernsteinBasis(uv.x);
    float4 basisV = BernsteinBasis(uv.y);
    float3 positionOS = CubicBezierSum(basisU, basisV);

    // 计算曲面点偏导
    float4 dBasisU = dBernsteinBasis(uv.x);
    float4 dBasisV = dBernsteinBasis(uv.y);
    float3 dPu = CubicBezierSum(dBasisU, basisV);
    float3 dPv = CubicBezierSum(basisU, dBasisV);

    output.positionOS = positionOS;
    output.normalOS = cross(dPu, dPv);

    return output;
}
```

由于打开几何着色器进行描边，所以顶点变换齐次裁剪空间在几何着色器中进行。片元着色器负责光照计算。

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/Bezier)

### Phong Tessellation

![[40a16f1e5fb8d7a945ab7487c9ec4d97_MD5.gif]]

下面通过 Phong Tessellation 的例子进一步理解如何在域着色器中应用控制点计算曲面顶点。

Phong Tessellation 用来平滑 Phong Shading 的着色结果，减少 artifaction。并且尽可能少的生成额外三角形，以减少消耗。

![[b27a0dffbe47447fcb52d15cc6f22337_MD5.jpg]]

有关 Phong Tessellation 的详细几何计算，可以参考我的文章：

[TheTus：计算机图形学：曲线与曲面](https://zhuanlan.zhihu.com/p/629202115)

顶点着色器依然充当传递着色器：

```
VertexOut PhongTessellationPassVertex(Attributes input) {
    VertexOut output;

    output.positionOS = input.positionOS;
    output.normalOS = input.normalOS;

    return output;
}
```

在 Hull Shader 中，Constant Hull Shader 定义细分因子，Control Point Hull Shader 仅仅传递 quad 的控制点。

```
PatchTess PhongTessellationPassConstantHull(InputPatch<VertexOut, 3> patch, uint patchID : SV_PrimitiveID) {
    PatchTess pt;

    pt.edgeTess[0] = _TessFactor.x;
    pt.edgeTess[1] = _TessFactor.y;
    pt.edgeTess[2] = _TessFactor.z;
    pt.insideFactor = _TessFactor.w;

    return pt;
}

[domain("tri")]
[partitioning("integer")]
[outputtopology("triangle_cw")]
[patchconstantfunc("PhongTessellationPassConstantHull")]
[outputcontrolpoints(3)]
[maxtessfactor(64.0)]
HullOut PhongTessellationPassHull(InputPatch<VertexOut, 3> patch, uint id : SV_OutputControlPointID) {
    HullOut output;

    output.positionOS = patch[id].positionOS;
    output.normalOS = patch[id].normalOS;

    return output;
}
```

在域着色器中，我们将三角形的三个控制点分别投影到对应法线定义的切线空间中，再对投影点进行重心坐标插值计算得到曲面上的点。

投影公式如下：

$\pi_{i}(\mathbf{p})=\mathbf{p}-\left((\mathbf{p}-\mathbf{v}_{i})^{T}\mathbf{n}_{i}\right)\mathbf{n}_{i}$

曲面坐标计算方程如下：

$p^{*}(u,v)=(1-\alpha)\mathbf{p}(u,v)+\alpha(u,v,w)\left( \begin{matrix} \pi_{i}(\mathbf{p}(u,v))\\ \pi_{j}(\mathbf{p}(u,v))\\ \pi_{k}(\mathbf{p}(u,v)) \end{matrix} \right)$

```
#define DOMAIN_PROGRAM_INTERPOLATE(fieldName) \
patch[0].fieldName * bary.x + \
patch[1].fieldName * bary.y + \
patch[2].fieldName * bary.z;

// 计算三角形点Q投影到顶点vi切线平面P的点Q'
float3 PhongProjectedPosition(float3 position, float3 triVertexPosition, float3 normal) {
    return position - dot(position - triVertexPosition, normal) * normal;
}

// 计算应用Phong Tessellation后点p的位置
// 所有点应在同一空间
float3 CalculatePhongPosition(float3 position, float3 p0Position, float3 p0Normal, float3 p1Position, float3 p1Normal, float3 p2Position, float3 p2Normal, float3 bary, float smoothing = 0.75) {
    float3 output = bary.x * PhongProjectedPosition(position, p0Position, p0Normal) +
        bary.y * PhongProjectedPosition(position, p1Position, p1Normal) +
        bary.z * PhongProjectedPosition(position, p2Position, p2Normal);
    return lerp(position, output, smoothing);
}

[domain("tri")]
DomainOut PhongTessellationPassDomain(PatchTess patchTess, float3 bary : SV_DomainLocation, OutputPatch<HullOut, 3> patch) {
    DomainOut output;

    float3 positionOS = DOMAIN_PROGRAM_INTERPOLATE(positionOS);

    output.positionOS = CalculatePhongPosition(positionOS, patch[0].positionOS, patch[0].normalOS, patch[1].positionOS, patch[1].normalOS, patch[2].positionOS, patch[2].normalOS, bary, _Smoothing);

    return output;
}
```

顶点转化到齐次裁剪空间同样在几何着色器中完成。

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/PhongTessellation)

在此基础上，加上光照。

![[f1770435e5721bbd3503b3c573dc40bb_MD5.gif]]

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/PhongTessellationLighting)

由于 Phong Tessellation 基于二次贝塞尔三角形，无法提供拐点，所以效果于 PN Triangles 较差。并且，它只保证 C0 连续性，对于硬边过渡可能产生裂缝。一种简单解决方法是利用顶点色等遮罩判断某顶点是否需要进行 Phong Tessellation。

### PN Triangles

![[37b91f2e2a14508f43f5d4fbfc7b1c2d_MD5.gif]]

在之前的例子中，control point hull shader 都仅仅充当传递着色器。下面，我们通过 PN Triangles 的例子说明在 hull shader 中计算并添加控制点。

PN Triangles 与 Phong Tessellation 类似，但它基于三次贝塞尔三角形，所以能提供更精细的曲面。

![[f709f22cd4bb8a1213101237d27f806d_MD5.jpg]]

关于 PN Triangles 详细的几何计算，可以参考我的文章：

[TheTus：计算机图形学：曲线与曲面](https://zhuanlan.zhihu.com/p/629202115)

顶点着色器依旧充当控制点的传递着色器。

```
VertexOut PNTrianglesPassVertex(Attributes input) {
    VertexOut output;

    output.positionOS = input.positionOS;
    output.normalOS = input.normalOS;

    return output;
}
```

Conostant Hull Shader 传递曲面细分因子。一种做法是在 constant hull shader 的输出结构体中存储额外控制点，但我们这里选择在 control point hull shader 里进行。

```
PatchTess PNTrianglesPassConstantHull(InputPatch<VertexOut, 3> patch, uint patchID : SV_PrimitiveID) {
    PatchTess pt;

    pt.edgeTess[0] = _TessFactor.x;
    pt.edgeTess[1] = _TessFactor.y;
    pt.edgeTess[2] = _TessFactor.z;
    pt.insideFactor = _TessFactor.w;

    return pt;
}
```

在 control point hull shader 中，我们需要计算额外的 6 个控制点坐标 (中心控制点可以由其他 9 个控制点计算出来)。对于每个 patch 的输出结构体，我们按照顺时针绕序添加两个控制点。将其带入投影公式即可。对于法线同理。（这部分原理描述起来有点麻烦，建议看我给出的链接）

![[55ef26192fffd9be03fa14cac486ef58_MD5.jpg]]

顶点投影公式如下：

$\mathbf{p}_{210}={\frac{1}{3}}(2{\bf p}_{300}+{\bf p_{030}}-({\bf n}_{200}\cdot({\bf p}_{030}-{\bf p}_{300})){\bf n}_{200}).$

法线投影公式如下：

${\bf n}_{110}={\bf n}_{200}+{\bf n}_{020}-2{\frac{({\bf p}_{030}-{\bf p}_{300})\cdot({\bf n}_{200}+{\bf n}_{020})}{({\bf p}_{030}-{\bf p}_{300})\cdot({\bf p}_{030}-{\bf p}_{300})}}({\bf p}_{030}-{\bf p}_{300}).$

```
// 计算cubic bezier (p0, p1)的第一个控制点
float3 CalculateCubicBezierControlPoint(float3 p0, float3 p1, float3 n0) {
    return (2.0 * p0 + p1 - dot(n0, p1 - p0) * n0) / 3.0;
}

// 计算quadratic bezier (p0, p1)的控制点
float3 CalculateQuadraticBezierControlNormal(float3 n0, float3 n1, float3 p0, float3 p1) {
    float3 d = p1 - p0;
    float v = 2.0 * dot(d, n0 + n1) / dot(d, d);
    return normalize(n0 + n1 - v * d);
}

[domain("tri")]
[partitioning("integer")] // 整数分割 向上取整
[outputtopology("triangle_cw")]
[patchconstantfunc("PNTrianglesPassConstantHull")]
[outputcontrolpoints(3)]
[maxtessfactor(64.0)]
HullOut PNTrianglesPassHull(InputPatch<VertexOut, 3> patch, uint id : SV_OutputControlPointID) {
    HullOut output;

    output.positionOS = patch[id].positionOS;
    output.normalOS = patch[id].normalOS;

    // 计算两控制点坐标 在Constant Hull Shader里进行也可以
    // 计算邻接顶点patchId
    const uint adjVertexId = id < 2 ? id + 1 : 0;
    output.positionOS0 = CalculateCubicBezierControlPoint(patch[id].positionOS, patch[adjVertexId].positionOS, patch[id].normalOS);
    output.positionOS1 = CalculateCubicBezierControlPoint(patch[adjVertexId].positionOS, patch[id].positionOS, patch[adjVertexId].normalOS);
    output.normalOS0 = CalculateQuadraticBezierControlNormal(patch[id].normalOS, patch[adjVertexId].normalOS, patch[id].positionOS, patch[adjVertexId].positionOS);

    return output;
}
```

在域着色器中，我们先计算出中心控制点的坐标，然后将这 10 个控制点带入曲面方程计算镶嵌器传递的该 patch 顶点的真正曲面位置。对于法线同理。

中心控制点坐标计算公式如下：

$\mathbf{p}_{111}=\frac{1}{4}(\mathbf{p}_{210}+\mathbf{p}_{120}+\mathbf{p}_{102}+\mathbf{p}_{201}+\mathbf{p}_{021}+\mathbf{p}_{012})-\frac{1}{6}(\mathbf{p}_{300}+\mathbf{p}_{030}+\mathbf{p}_{003})$

三次 (cubic) 贝塞尔三角曲面公式如下：

$\begin{align} {\bf p}(u,v)&=\sum_{i+j+k=3}B_{i j k}^{3}(u,v){\bf p}_{i j k}\\ &=u^{3}{\bf p}_{300}+v^{3}{\bf p}_{030}+w^{3}{\bf p}_{003}+3u^{2}v{\bf p}_{210}+3u^{2}w{\bf p}_{201} \\ &+3u v^{2}{\bf p}_{120}+3v^{2}w{\bf p}_{021}+3v w^{2}{\bf p}_{012}+3u w^{2}{\bf p}_{102}+6u v w{\bf p}_{111}. \end{align}$

二次 (quadratic) 贝塞尔曲面公式如下：

$\begin{align} \mathbf{n}(u,v)&=\sum_{i+j+k=2}B_{i j k}^{2}(u,v)\mathbf{n}_{i j k}\\ &=u^{2}{\bf n}_{200}+v^{2}{\bf n}_{020}+w^{2}{\bf n}_{002}+2(u v{\bf n}_{110}+u w{\bf n}_{101}+v w{\bf n}_{011}). \end{align}$

```
// 根据重心坐标计算 三次贝塞尔三角patch顶点 注意向量应在同一空间
float3 CalculateCubicBezierPosition(float3 controlPoints[10], float3 bary, float smoothing = 0.75) {
    float3 flatSurfacePosition = bary.x * controlPoints[0] + bary.y * controlPoints[3] + bary.z * controlPoints[6];
    float3 bezierSurfacePosition =
        bary.x * bary.x * bary.x * controlPoints[0] + 3.0 * bary.x * bary.x * bary.y * controlPoints[1] + 3.0 * bary.x * bary.y * bary.y * controlPoints[2] +
        bary.y * bary.y * bary.y * controlPoints[3] + 3.0 * bary.y * bary.y * bary.z * controlPoints[4] + 3.0 * bary.y * bary.z * bary.z * controlPoints[5] +
        bary.z * bary.z * bary.z * controlPoints[6] + 3.0 * bary.x * bary.z * bary.z * controlPoints[7] + 3.0 * bary.x * bary.x * bary.z * controlPoints[8] +
        6.0 * bary.x * bary.y * bary.z * controlPoints[9];

    return lerp(flatSurfacePosition, bezierSurfacePosition, smoothing);
}

// 根据重心坐标计算 二次贝塞尔三角patch法线 注意向量应在同一空间
float3 CalculateQuadraticBezierNormal(float3 controlPoints[6], float3 bary, float smoothing = 0.75) {
    float3 flatSurfaceNormal = bary.x * controlPoints[0] + bary.y * controlPoints[2] + bary.z * controlPoints[4];
    float3 bezierSurfaceNormal =
        bary.x * bary.x * controlPoints[0] + 2.0 * bary.x * bary.y * controlPoints[1] +
        bary.y * bary.y * controlPoints[2] + 2.0 * bary.y * bary.z * controlPoints[3] +
        bary.z * bary.z * controlPoints[4] + 2.0 * bary.x * bary.z * controlPoints[5];
    return normalize(lerp(flatSurfaceNormal, bezierSurfaceNormal, smoothing));
}

// 计算二次贝塞尔插值后的切线
float3 CalculateTangentAfterQuadraticBezier(float3 t0, float3 t1, float3 t2, float3 flatSurfaceNormal, float3 bezierSurfaceNormal, float3 bary) {
    float3 flatSurfaceTangent = bary.x * t0 + bary.y * t1 + bary.z * t2;
    float3 flatBitangent = cross(flatSurfaceNormal, flatSurfaceTangent);
    return normalize(cross(flatBitangent, bezierSurfaceNormal));
}

[domain("tri")]
DomainOut PNTrianglesPassDomain(PatchTess patchTess, float3 bary : SV_DomainLocation, OutputPatch<HullOut, 3> patch) {
    DomainOut output;

    // 计算cubic triangle bezier patch position
    // 计算控制点位置
    float3 controlPoints[10];
    float3 avgBezier = 0.0, avgVertices = 0.0;
    // 让编译器将循环展开
    UNITY_UNROLL
    for (int i = 0; i < 3; i++) {
        controlPoints[i * 3] = patch[i].positionOS;
        controlPoints[i * 3 + 1] = patch[i].positionOS0;
        controlPoints[i * 3 + 2] = patch[i].positionOS1;
        avgBezier += patch[i].positionOS0 + patch[i].positionOS1;
        avgVertices += patch[i].positionOS;
    }
    avgBezier /= 6.0;
    avgVertices /= 3.0;
    controlPoints[9] = avgBezier + (avgBezier - avgVertices) * 0.5;

    // 计算quadratic bezier patch normal
    // 计算控制点位置
    float3 controlNormals[6];
    UNITY_UNROLL
    for (int i = 0; i < 3; i ++) {
        controlNormals[i * 2] = patch[i].normalOS;
        controlNormals[i * 2 + 1] = patch[i].normalOS0;
    }

    // 计算贝塞尔点
    output.positionOS = CalculateCubicBezierPosition(controlPoints, bary, _Smoothing);
    output.normalOS = CalculateQuadraticBezierNormal(controlNormals, bary, _Smoothing);

    return output;
}
```

顶点变换到齐次裁剪空间同样在几何着色器里进行。

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/PN%20Triangles)

在这个基础上，加上光照。可以发现效果还是挺明显的。

![[325340e8ad9bd806fc9572db6f5db12b_MD5.gif]]

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/PNTrianglesLighting)

## 优化手段

### 动态细分因子

曲面细分阶段的一个重要应用就是进行 LOD，从而减少远处物体不必要的细节渲染。

下面将讨论两种动态细分因子的方法，一种基于相机距离，一种基于屏幕占用范围。它们的代码可以在 URP 库的 [Tessellation.hlsl](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.core/ShaderLibrary/Tessellation.hlsl) 中找到。

**基于相机距离**

![[e98e2e9917477d4bac1b0da1d5ed5369_MD5.gif]]

基于相机距离即离相机仅细分因子大，离相机远细分因子小。在`Tessellation.hlsl`中，提供了一个`GetDistanceBasedTessFactor`用来根据相机距离输出细分因子，具体代码如下：

```
real3 GetDistanceBasedTessFactor(real3 p0, real3 p1, real3 p2, real3 cameraPosWS, real tessMinDist, real tessMaxDist) {
    real3 edgePosition0 = 0.5 * (p1 + p2);
    real3 edgePosition1 = 0.5 * (p0 + p2);
    real3 edgePosition2 = 0.5 * (p0 + p1);

    // In case camera-relative rendering is enabled, 'cameraPosWS' is statically known to be 0,
    // so the compiler will be able to optimize distance() to length().
    real dist0 = distance(edgePosition0, cameraPosWS);
    real dist1 = distance(edgePosition1, cameraPosWS);
    real dist2 = distance(edgePosition2, cameraPosWS);

    // The saturate will handle the produced NaN in case min == max
    real fadeDist = tessMaxDist - tessMinDist;
    real3 tessFactor;
    tessFactor.x = saturate(1.0 - (dist0 - tessMinDist) / fadeDist);
    tessFactor.y = saturate(1.0 - (dist1 - tessMinDist) / fadeDist);
    tessFactor.z = saturate(1.0 - (dist2 - tessMinDist) / fadeDist);

    return tessFactor;
}
```

在代码中，首先计算每条边中点与相机距离，然后根据相机距离，将细分因子从 tessMinDist 的 1 逐渐缩减到 tessMaxDist 的 0。

并且提供了一个`CalcTriTessFactorsFromEdgeTessFactors`函数用来根据边缘细分因子平均加权得到内部细分因子。

```
real4 CalcTriTessFactorsFromEdgeTessFactors(real3 triVertexFactors) {
    real4 tess;
    tess.x = triVertexFactors.x;
    tess.y = triVertexFactors.y;
    tess.z = triVertexFactors.z;
    tess.w = (triVertexFactors.x + triVertexFactors.y + triVertexFactors.z) / 3.0;

    return tess;
}
```

在我们的 Shader 中，应用这些函数。

```
PatchTess DynamicTessellationFactorsPassConstantHull(InputPatch<VertexOut, 3> patch, uint patchID : SV_PrimitiveID) {
    PatchTess pt;
    float4 tessFactors = _TessFactor;
 #if defined(_DYNAMIC_CAMERA)
    float3 cameraPositionWS = GetCameraPositionWS();
    real3 distanceBasedTessFactor = GetDistanceBasedTessFactor(patch[0].positionWS, patch[1].positionWS, patch[2].positionWS, cameraPositionWS, _TessMinDist, _TessMinDist + _TessFadeDist);
    tessFactors = _TessFactor * CalcTriTessFactorsFromEdgeTessFactors(distanceBasedTessFactor);
    pt.edgeTess[0] = max(1.0, tessFactors.x);
    pt.edgeTess[1] = max(1.0, tessFactors.y);
    pt.edgeTess[2] = max(1.0, tessFactors.z);
    pt.insideFactor = max(1.0, tessFactors.w);
 #endif

    pt.edgeTess[0] = max(1.0, tessFactors.x);
    pt.edgeTess[1] = max(1.0, tessFactors.y);
    pt.edgeTess[2] = max(1.0, tessFactors.z);
    pt.insideFactor = max(1.0, tessFactors.w);

    return pt;
}
```

**基于屏幕占用范围**

![[4169546c7a873339697b8e767f3c5789_MD5.gif]]

基于屏幕占用范围的想法是获取屏幕上每边的长度，如果长度小于设定的 triangleSize，则细分因子进行衰减。

在`Tessellation.hlsl`中，提供了一个`GetScreenSpaceTessFactor`用于计算基于屏幕占用范围的细分因子。

```
// Compute both screen and distance based adaptation - return factor between 0 and 1
real3 GetScreenSpaceTessFactor(real3 p0, real3 p1, real3 p2, real4x4 viewProjectionMatrix, real4 screenSize, real triangleSize)
{
    // Get screen space adaptive scale factor
    real2 edgeScreenPosition0 = ComputeNormalizedDeviceCoordinates(p0, viewProjectionMatrix) * screenSize.xy;
    real2 edgeScreenPosition1 = ComputeNormalizedDeviceCoordinates(p1, viewProjectionMatrix) * screenSize.xy;
    real2 edgeScreenPosition2 = ComputeNormalizedDeviceCoordinates(p2, viewProjectionMatrix) * screenSize.xy;

    real EdgeScale = 1.0 / triangleSize; // Edge size in reality, but name is simpler
    real3 tessFactor;
    tessFactor.x = saturate(distance(edgeScreenPosition1, edgeScreenPosition2) * EdgeScale);
    tessFactor.y = saturate(distance(edgeScreenPosition0, edgeScreenPosition2) * EdgeScale);
    tessFactor.z = saturate(distance(edgeScreenPosition0, edgeScreenPosition1) * EdgeScale);

    return tessFactor;
}
```

它通过`ComputeNormalizedDeviceCoordinates`计算得到顶点 screen space 坐标，然后乘以屏幕尺寸得到 window space 坐标。最后，根据三角形长度对屏幕顶点长度进行缩放得到细分因子。

在我们的 Shader 中，应用这些函数。

```
PatchTess DynamicTessellationFactorsPassConstantHull(InputPatch<VertexOut, 3> patch, uint patchID : SV_PrimitiveID) {
    PatchTess pt;
    float4 tessFactors = _TessFactor;
 #if defined(_DYNAMIC_SCREEN)
    real3 screenSpaceTessFactor = GetScreenSpaceTessFactor(patch[0].positionWS, patch[1].positionWS, patch[2].positionWS, GetWorldToHClipMatrix(), _ScreenParams, _TriangleSize);
    tessFactors = _TessFactor * CalcTriTessFactorsFromEdgeTessFactors(screenSpaceTessFactor);
 #endif

    pt.edgeTess[0] = max(1.0, tessFactors.x);
    pt.edgeTess[1] = max(1.0, tessFactors.y);
    pt.edgeTess[2] = max(1.0, tessFactors.z);
    pt.insideFactor = max(1.0, tessFactors.w);

    return pt;
}
```

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/DynamicTessellationFactors)

### patch 剔除

我们可以通过在 constant hull shader 中指定细分因子为 0，将该 patch 剔除，进而不送到后面的阶段。这使得我们可以在常量外壳着色器执行 **视锥体剔除** 和 **背面剔除** 等优化手段。

我们首先进行视锥体剔除，这意味着我们不会看见视锥体外的三角形。为了实现这一点，我们可以使用三角形顶点的齐次裁剪空间位置。

如果某点向量与视锥体某平面的点积为负数，则说明它在这个平面的外部。如果该点向量在所有平面外部，则说明它在视锥体外部。引入 bias 调整容忍度。

```
bool TriangleIsBelowClipPlane(float3 p0, float3 p1, float3 p2, int planeIndex, float bias) {
    float4 plane = unity_CameraWorldClipPlanes[planeIndex];
    // 结果为负，则在平面外部
    return dot(float4(p0, 1), plane) < bias && dot(float4(p1, 1), plane) < bias && dot(float4(p2, 1), plane) < bias;
}
```

我们只需要对所有视锥体平面进行判断并取或即可。

然后进行背面剔除，它是根据三角形 **绕序 (winding order)** 进行判断的。假定顺时针为正方向，那么绕序为逆时针的就是背面。这可以通过三角形两边向量的叉乘进行判断。引入 bias 调整容忍度。

```
bool TriangleIsBackFace(float4 p0CS, float4 p1CS, float4 p2CS, float bias) {
    float3 p0 = p0CS.xyz / p0CS.w;
    float3 p1 = p1CS.xyz / p1CS.w;
    float3 p2 = p2CS.xyz / p2CS.w;
    #if UNITY_REVERSED_Z
    return cross(p1 - p0, p2 - p0).z < bias;
    #else
        return cross(p1 - p0, p2-p0).z > -bias;
    #endif
}
```

将这两个判断杂交在一起，判断某三角形是否需要进行剔除。

```
// 根据视锥平面进行剔除
bool TriangleIsCulled(float3 p0WS, float3 p1WS, float3 p2WS, float bias) {
    float4 p0CS = TransformWorldToHClip(p0WS);
    float4 p1CS = TransformWorldToHClip(p1WS);
    float4 p2CS = TransformWorldToHClip(p2WS);
    return TriangleIsBackFace(p0CS, p1CS, p2CS, bias) ||
        TriangleIsBelowClipPlane(p0WS, p1WS, p2WS, 0, bias) ||
        TriangleIsBelowClipPlane(p0WS, p1WS, p2WS, 1, bias) ||
        TriangleIsBelowClipPlane(p0WS, p1WS, p2WS, 2, bias) ||
        TriangleIsBelowClipPlane(p0WS, p1WS, p2WS, 3, bias);
}
```

最后，在 constant hull shader 中判断是否需要剔除三角形，如果需要剔除则将细分因子设为 0，否则进行正常的细分因子传递。

```
// 常量外壳着色器，传递细分因子
PatchTess DisplacementPassConstantHull(InputPatch<VertexOut, 3> patch, uint patchID : SV_PrimitiveID) {
    PatchTess pt;

    float3 p0 = TransformObjectToWorld(patch[0].positionOS);
    float3 p1 = TransformObjectToWorld(patch[1].positionOS);
    float3 p2 = TransformObjectToWorld(patch[2].positionOS);
    float bias = -0.5 * _DisStrength;
    // 判断是否要剔除三角形
    if (TriangleIsCulled(p0, p1, p2, bias)) {
        pt.edgeTess[0] = pt.edgeTess[1] = pt.edgeTess[2] = pt.insideTess = 0;
    }
    else {
        pt.edgeTess[0] = TessellationEdgeFactor(p1, p2);
        pt.edgeTess[1] = TessellationEdgeFactor(p2, p0);
        pt.edgeTess[2] = TessellationEdgeFactor(p0, p1);
        // 编译器优化
        pt.insideTess = (TessellationEdgeFactor(p1, p2) + TessellationEdgeFactor(p2, p0) + TessellationEdgeFactor(p0, p1)) * 0.333;
    }

    return pt;
}
```

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/TessellationShaders/Displacement)

## 参考

《DirectX12 3D 游戏开发实战》

《Real-time Rendering 4th》

[MSDN: Tessellation Stage](https://learn.microsoft.com/en-us/windows/win32/direct3d11/direct3d-11-advanced-stages-tessellation) [OpenGL Wiki: Tessellation](https://www.khronos.org/opengl/wiki/Tessellation)

[Paper: Curved PN Triangles](http://alex.vlachos.com/graphics/CurvedPNTriangles.pdf)

[Paper: Phong Tessellation](https://perso.telecom-paristech.fr/boubek/papers/PhongTessellation/PhongTessellation.pdf)

[Unity-Technologies: Tessellation.hlsl](https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.core/ShaderLibrary/Tessellation.hlsl)

[Direct3D 11 Tutorial: Tessellation](https://ubm-twvideo01.s3.amazonaws.com/o1/vault/gdc10/slides/Bilodeau_Bill_Direct3D11TutorialTessellation.pdf)

[Mastering Teseellation Shaders and Their Many Uses in Unity](https://nedmakesgames.medium.com/mastering-tessellation-shaders-and-their-many-uses-in-unity-9caeb760150e)

[CatlikeCoding: Tessellation](https://catlikecoding.com/unity/tutorials/advanced-rendering/tessellation/)