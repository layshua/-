Unity 版本：2022

URP 版本：14.0.0

项目链接：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry)

## 概述

在顶点着色器和片元着色器之间有一个可选的**几何着色器 (geometry shader)** 阶段。 几何着色器可以将输入图元转换为其他图元，这是曲面细分等阶段无法左到的。几何着色器在输入的几何图元 (如点、线段、三角形) 级别上进行操作，并根据需要生成零个、一个或多个输出的新的几何图元。这使得在几何着色器阶段可以实现诸如几何图元的细分、几何的放大缩小、草地生成、粒子系统等各种复杂的效果。

几何着色器的输入是完整的图元，输出是新的图元。例如，如果要绘制一个三角形列表 (triangle list)，则几何着色器程序实际将对列表中的每个三角形 T 执行下列操作：

```
for(UINT i = 0; i < numTriagnles; ++i){
    OutputPrimitiveList = GeometryShader(T[i].vertexList);
}
```

几何着色器保证按照输入的相同顺序输出图元的结果，这恨影响性能，因为结果必须保存和排序。同时，它又是完全可编程的。所以通常很少使用，因为它不能很好体现 GPU 优势。在移动设备上，甚至会被阻止。

不同 API 的几何着色器写法各不同，Unity 与 DirectX 的类似，所以这里更多讨论 hlsl 几何着色器在 Unity 里的写法。并且简单给出几个 Unity 几何着色器的例子，当然不会涉及到什么更炫酷的效果，这些在网上也能搜到。

## 渲染管线

建议在这之前先梳理一遍渲染管线，可以参考我的文章：

[undefined](https://zhuanlan.zhihu.com/p/627201581)

将渲染管线按功能性阶段划分，几何着色器处于几何处理阶段的可选顶点处理阶段。

![[7fc976c905dd164642fb1259b29b677e_MD5.jpg]]

将渲染管线按 GPU 逻辑管线划分，几何着色器在顶点着色器 (和曲面细分着色器) 阶段之后，光栅化阶段之前。

![[b2149ed3f4ef0620573e18407b4bb84b_MD5.jpg]]

顶点着色器以顶点数据作为输入数据，而几何着色器的输入是上一个阶段生成的顶点组成的图元。与顶点着色器不同，几何着色器可以创建或销毁几何图元。然后，几何着色器将新的几何图元传递给光栅化阶段。经过光栅化处理后，最终会生成像素片段，供像素着色器阶段进行处理。  
几何着色器所输出的图元由顶点列表定义而成，必须将顶点的位置变换到**齐次裁剪空间**。换言之，经过几何着色器阶段的处理后，我们就得到了位于齐次裁剪空间中由一系列顶点所定义的多个图元。

需要注意的是，顶点数据的插值发生在光栅化阶段前，即几何着色器后，顶点着色器仅仅将数据传输给几何着色器。

![[cd43dd6dd3bac332f2ad070516fc55bc_MD5.jpg]]

几何着色器的输出图元类型不一定与输入图元的类型相同。例如，我们可以将顶点着色器传入的一个点扩展为一个四边形。

## 图元拓扑类型

在应用阶段，CPU 需要向 GPU 提交一系列数据和命令供其渲染。  
应用阶段最重要的任务是**输入装配 (input assembler)**。输入装配阶段会从显存中读取几何数据 (顶点和索引)，再将它装配为**几何图元 (geometry primitive)**。  
可是，单凭顶点和索引数据，GPU 无法知道顶点究竟如何组成几何图元。例如，我们应将顶点 2 个一组解释成线段，还是 3 个一组解释为三角形呢？对此，我们需要通过指定**图元拓扑 (primitive topology)** 来告诉 GPU 如何利用顶点数据来表示几何图元。

在 DirectX 中，基础图元拓扑类型有以下五种：**点列表 (point list)**，**线条带 (line strip)**，**线列表 (line list)**，**三角形带 (triangle strip)**，**三角形列表 (triangle list)**。  
在 DirectX 10 以后的版本中，加入了一种**邻接图元 (primitive adjacency)** 类型。任何基础图元类型都可以扩展成邻接图元版本。  
各种图元拓扑类型如下图，可以先只关注实线顶点，并且注意所有三角类型的顶点都是相同的。

![[c9142dfe823e9050ce5108e9c46866ad_MD5.jpg]]

上图中各符号意义如下表所示。

![[1666bcb6d044ca030bfdb440a218ef6d_MD5.jpg]]

我们可以将拓扑类型简单分类。  
根据图元的类型，我们可以将其分为点、线和三角形。而根据连接方式，我们可以将其分为列表 (list) 和带(strip)。  
列表类型指的是孤立图元类型的列表。对于三角形列表而言，我们可以将输入顶点序列每三个一组组成一个三角形。例如，对于输入顶点序列 012345，我们可以组成两个三角形：[0, 1, 2] 和 [3, 4, 5]。线列表的处理方式类似。  
带类型指的是一系列相连的图元类型。对于三角形带而言，当我们输入一个三角形后，在绘制完第一个三角形后，每个后续顶点都将会与上一个三角形的边相连，生成另一个三角形。我们可以使用一个长度为 3 的窗口在输入顶点序列中滑动，观察每个生成的三角形。例如，对于输入序列 012345，生成的三角形序列如下：[0, 1, 2], 3, 4, 5; 0, [1, 2, 3], 4, 5; 0, 1, [2, 3, 4], 5; 0, 1, 2, [3, 4, 5]。线条带的处理方式类似。

大多数场景中物体的图元拓扑类型都是三角形带，因为 N 个顶点序列就可以表示 N-2 个三角形。可以用来指定复杂的物体，并且有效地利用内存和处理时间。下面进一步讨论三角形带。  
在几何处理阶段，光栅化阶段之前，三角形的顶点会执行背面剔除，忽略那些看不见的三角形面片。背面剔除算法通常使用三角形的顶点顺序和观察者的视点方向来确定面片的朝向。也就是说，三角形的**绕序 (winding order)** 也是拓扑类型的一个重要因素。  
以微软 DirectX(Unity 也是) 文档中配图可以看出，三角形列表的绕序为顺时针。但三角形带无法保证每个三角形的绕序相同。经过观察发现，三角形带中，奇次序的三角形绕序为顺时针，偶次序三角形的绕序为逆时针。为了解决这个问题，**GPU 内部会对偶数三角形的后两个顶点顺序进行调换**，以此使它们与奇数三角形的绕序都保持**顺时针**。

![[85156af39fe4c9468793d859d676906e_MD5.jpg]]

以上图为例，GPU 实际处理的绕序应该是：123 243 345 465 567。

邻接图元可以在任何基础拓扑类型衍生出来。注意，邻接图元你的顶点只能用作几何着色器的输入数据，却并**不会被绘制出来**。即便程序没有用到几何着色器，但依旧不会绘制邻接图元。

注意，以上的讨论都是针对应用阶段输入装配阶段，CPU 告诉 GPU 如何将顶点组成图元类型的图元拓扑类型，而并非向几何着色器输入的图元类型。

## 编写几何着色器

几何着色器的一般编写格式如下：

```
[maxcertexcount(N)]
void ShaderName (PrimitiveType InputVertexType InputName[NumElements],
                inout StreamOutputObjectVertexType) OutputName){
    // 几何着色器具体实现
}
```

### 最大顶点数量

`[maxvertexcount(N)]`用来指定几何着色器单词调用所输出的顶点数量最大值。其中，N 是几何着色器**单次调用**所输出的顶点数量最大值。几何着色器每次输出的顶点个数都可能不同，但是这个数量却不能超过之前定义的最大值。  
出于对性能方面的考虑，我们应当令 maxvertexcount 的值尽可能小。线管资料显示，GS 每次输出的标量数量在 1-20 时，它将发挥出最佳的性能；而当 27-40 时，它的性能将下降到峰值性能的 50%。

每次调用几何着色器所输出的标量个数为：maxvertexcount 与输出顶点类型结构体中标量个数的乘积。例如，如果顶点结构体定义了`float3 pos : POSITION`与`float2 tex : COORD0`，即顶点元素中含有 5 个标量。假设此时将 maxvertexcount 设置为 4，则几何着色器每次输出 20 个标量，以峰值性能执行。

### 输入输出

几何着色器的输入参数必须是一个定义有特定图元的顶点数组，点应输入一个顶点，线条列表 / 带应输入两个顶点，三角形列表 / 带应输入 3 个顶点，线及邻接图元为 4 个顶点，三角形及其邻接图元则为 6 个顶点。  
输入参数以图元类型作为前缀，用以描述输入到几何着色器的具体图元类型。并且注意，**输入图元类型必须对应输入装配阶段的图元拓扑类型**，否则会出现顶点不匹配的现象。该前缀可以是下列类型之一：

*   `point`：输入图元拓扑类型为点列表
*   `line`：输入图元拓扑类型为线列表或线条带
*   `triangle`：输入的图元拓扑类型为三角形列表或三角形带
*   `lineadj`：输入的图元拓扑类型为线条列表 / 带及其邻接图元
*   `triangleadj`：输入的图元为三角形列表 / 带及其邻接图元

几何着色器的输出参数是标有`inout`修饰符的**流类型 (stream type)** 。流类型存有一系列顶点，它们定义了几何着色器输出的几何图形。  
流类型的本质是一种模板类型 (template type)，其模板参数用以指定输出顶点的具体类型。流类型有如下 3 种：

*   `PointStream<OutputVertexType>`：一系列顶点所定义的点列表
*   `LineStream<OutputVertexType>`：一系列顶点所定义的线条带
*   `TriangleStream<OutputVertexType>`：一系列顶点所定义的三角形带

几何着色器输出的多个顶点会够成图元，图元的输出类型由流类型来指定。对于线条与三角形来说，几何着色器输出的对应图元拓扑类型必须是**线条带与三角形带**。而线条列表与三角形列表可以借助内置函数`RestarStrip`输出。

由于大多数模型图元拓扑类型都是三角形带，所以其实输入一般都是`triangle`。

### Append

`Append`函数用来将几何着色器的输出数据追加到一个现有的流中。

```
[maxcertexcount(N)]
void ShaderName (PrimitiveType InputVertexType InputName[NumElements],
                inout StreamOutputObjectVertexType) OutputName){
    // 几何着色器具体实现
    StreamOutputObjectVertexType gout;
    OutputName.Append(gout);
}
```

### RestartStrip

`RestartStrip`函数用来结束当前的基元条带，开始一个新的条带。如果当前的条带没有足够的顶点被追加出来以填满基元拓扑结构，那么末端的不完整基元将被丢弃。  
前面提到，几何着色器输出的图元拓扑类型只能是线条带或三角形带，但总有带状结构无法表示的情况 (或者说过载了想不出来)，这时候就可以用`RestartStrip`来重置输出流，采用类似三角形列表的方式追加几个三角形。

```
[maxcertexcount(N)]
void ShaderName (PrimitiveType InputVertexType InputName[NumElements],
                inout StreamOutputObjectVertexType) OutputName){
    // 几何着色器具体实现
    StreamOutputObjectVertexType gout;
    OutputName.Append(gout);
    OutputName.RestartStrip();
    OutputName.Append(gout);
}
```

## 广告牌 (Billboard)

使用几何着色器将输入点图元扩展为三角形带。

![[922cbce45f88fe282575f6c858222a4b_MD5.gif]]

广告牌（Billboard）技术是一种常用的图形渲染技术，用于在三维场景中渲染面向摄像机的平面对象，通常用于表示树木、草地、粒子效果等。  
在广告牌技术中，每个平面对象（如树木）实际上是一个只有一个面的矩形，该面始终面向摄像机。这样可以通过简化几何模型来提高性能，并在保持物体外观的同时实现高效渲染。

我们可以通过几何着色器来实现多个广告牌的渲染。这使得我们在 CPU 阶段只需要发送由顶点组成的图元，GPU 再将它扩展成四边形面片，并让它朝向相机。

在这个例子中，我们用 C# 脚本绘制随机位置点列表组成的 mesh，然后用几何着色器将它扩展为一个房子 (LearnOpenGL 的例子)，并朝向摄像机。

在 C# 脚本中，只需要创建由随机位置点拓扑组成的 Mesh 即可。代码：[github](https://github.com/dyxdyxdyx/TA-100/tree/master/Assets/3.3%20TESS%26GS/Scripts)  
在 Shader 中，首先定义各个阶段的输入与输出。

```
struct Attributes {
    float4 positionOS : POSITION;
};

struct VertexOut {
    float4 positionOS : TEXCOORD0;
    float3 positionWS : TEXCOORD1;
};

struct Varyings {
    float4 positionCS : SV_POSITION;
    float2 uv : TEXCOORD0;
    float3 positionWS : TEXCOORD1;
    uint primID: SV_PrimitiveID;
};
```

顶点着色器负责将顶点数据传给几何着色器。

```
VertexOut HouseBuildingPassVertex(Attributes input) {
    VertexOut output;

    VertexPositionInputs vertexInputs = GetVertexPositionInputs(input.positionOS);
    output.positionOS = input.positionOS;
    output.positionWS = vertexInputs.positionWS;

    return output;
}
```

在几何着色器中，将顶点扩展为三角形带。计算面朝向相机的 up 和 right 向量，然后用它计算出其他四边形的位置。

![[09a0bb578071251b22dbb32dff52a6ad_MD5.jpg]]

```
// 输出最大顶点数量：5
[maxvertexcount(5)]
void HouseBuildingPassGeometry(point VertexOut input[1],// 输入图元类型：点
                               uint primID : SV_PrimitiveID,// 当前处理图元的ID
                               inout TriangleStream<Varyings> triStream) {// 输出流
    // 计算朝向摄像机的上方向和右方向向量
    float3 up = float3(0.0, 1.0, 0.0);
    float3 look = -GetWorldSpaceViewDir(input[0].positionWS.xyz);
    look.y = 0.0;
    look = normalize(look);
    float3 right = cross(up, look);

    float4 v[5];
    float width = _Width * 0.5, height = _Height * 0.5;
    // 顺时针
    v[0] = float4(input[0].positionWS.xyz + 1.7 * up * height, 1.0); // 顶部
    v[1] = float4(input[0].positionWS.xyz + width * right + height * up, 1.0); // 右上
    v[2] = float4(input[0].positionWS.xyz - width * right + height * up, 1.0); // 左上
    v[3] = float4(input[0].positionWS.xyz + width * right - height * up, 1.0); // 右下
    v[4] = float4(input[0].positionWS.xyz - width * right - height * up, 1.0); // 左下

    float2 texcoords[5];
    texcoords[0] = float2(0.0, 0.0);
    texcoords[1] = float2(1.0, 1.0);
    texcoords[2] = float2(0.0, 1.0);
    texcoords[3] = float2(1.0, 0.0);
    texcoords[4] = float2(0.0, 0.0);

    //  static loop，把循环展开
    Varyings gout;
    UNITY_UNROLL
    for (uint i = 0; i < 5; ++i) {
        VertexPositionInputs positionInputs = GetVertexPositionInputs(v[i]);
        gout.positionCS = TransformWorldToHClip(v[i]);
        gout.positionWS = v[i];
        gout.primID = primID;
        gout.uv = texcoords[i];

        triStream.Append(gout);
    }
}
```

几何着色器中，对纹理数组采样，用 primID 取模作为索引。

```
half4 HouseBuildingPassFragment(Varyings input) : SV_Target {
    half4 var_MainTex = SAMPLE_TEXTURE2D_ARRAY(_Textures, sampler_Textures, input.uv, input.primID % _TextureCount);
    half3 finalCol = var_MainTex.rgb;

    return half4(finalCol, 1.0);
}
```

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/GeometryShaders/HouseBuilding)

## 法线渲染

使用几何着色器将三角形带扩展为线条带。

![[efa98efca409b1f8e3db2ded52e34ca9_MD5.jpg]]

其中涉及到线条顶点位置的计算，法线起点应该在三角形重心位置。  
代码如下：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/GeometryShaders/NormalShading)

把材质放到 RendererFeature 中，以便该 Layer 的物体可以用这个 Pass 渲染法线。

## Flat Wireframe

使用几何着色器将三角形带转换为三角形带，计算并传递额外信息。

![[aabcd31cd821041794b6470b4f44c586_MD5.jpg]]

这个例子的详细教程在 [catlikecoding](https://catlikecoding.com/unity/tutorials/advanced-rendering/flat-and-wireframe-shading/)，这里仅仅给出代码。[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/GeometryShaders/FlatWireframe)

## 细分

这是龙书的一个练习，我的做法也不一定是最好的 (项目里肯定不能这么用)，放这里主要想说明一下`RestartStrip`的用法。

![[3dad522c2efbac125374581e75db736b_MD5.gif]]

题目描述如下：

![[14bf7e51298becfde119cc7962e78c8c_MD5.jpg]]

正二十面体模型就不用代码生成了，直接导进来。

下面先讨论一次细分的写法。

![[e976bd8840fdb0e6c3990ef1a62de3b7_MD5.jpg]]

由于是正三角形，我们只需要在每条边找到一个中点，然后将中点连起来组成新三角形即可。一个三角形可以生成 4 个子三角形。

![[39eb83f1a0a53461f8d17a78083ff9fa_MD5.jpg]]

如上图三角形，v0-v1-v2，将中点 m0-m1-m2 连接，一共生成 4 个子三角形。我们就需要在几何着色器中先计算出三个中点，然后依次追加到输出三角形带里。  
但是，上述子三角形是无法一次性用三角形带输出的。我们换一种思路，将下面一层当作三角形带，上面一层当作三角形列表，中间用`RestartStrip`分开。  
也就是说，添加给输出流的顺序如下：(v0,m0,m2,m1,v2) `RestartStrip` (m0,v1,m1)。

```
[maxvertexcount(9)]
void LODPassGeometry(triangle VertexOut gin[3], uint primID : SV_PrimitiveID, inout TriangleStream<Varyings> triStream) {
    // 计算中点坐标
    VertexOut m[3];
    UNITY_UNROLL
    for (int i = 0; i < 3; i++) {
        // 计算中点坐标
        m[i].positionOS = (gin[i].positionOS + gin[(i + 1) % 3].positionOS) * 0.5;
        // 中点法线即模型空间位置(球)
        m[i].normalOS = normalize(m[i].positionOS);
        // 将顶点投影到半径为1的球面上
        m[i].positionOS = m[i].normalOS;
    }

    // 组装输出顶点
    VertexOut vout[6];
    vout[0] = gin[0];
    vout[1] = m[0];
    vout[2] = m[2];
    vout[3] = m[1];
    vout[4] = gin[2];
    vout[5] = gin[1];

    Varyings gout[6];
    VertexPositionInputs vertexInputs;
    VertexNormalInputs normalInputs;

    float3 coordsX = {1.0, 0.0, 0.0};
    float3 coordsY = {0.0, 1.0, 0.0};

    // 计算输出图元顶点序列
    UNITY_UNROLL
    for (uint i = 0; i < 6; ++i) {
        vertexInputs = GetVertexPositionInputs(vout[i].positionOS);
        normalInputs = GetVertexNormalInputs(normalize(vout[i].positionOS));
        gout[i].positionCS = vertexInputs.positionCS;
        gout[i].normalWS = normalInputs.normalWS;
        gout[i].baryCoord = float2(coordsX[i % 3], coordsY[i % 3]);
    }

    // 输出下方三角形带
    UNITY_UNROLL
    for (uint i = 0; i < 5; ++i)
        triStream.Append(gout[i]);

    // 输出顶三角形
    triStream.RestartStrip();
    triStream.Append(gout[1]);
    triStream.Append(gout[5]);
    triStream.Append(gout[3]);
}
```

下面讨论多次细分的写法。  
明显可以感觉到，这是一个递归的过程。一个三角形分成 4 个子三角形，1 个子三角形又细分成 4 个子子三角形。但是在 Shader 里面递归还是有点抽象的，还是寻找一种线性的解法。

![[e15e33f51df5c210928f45ab8cfd684f_MD5.jpg]]

观察二次细分，我们可以得到一个规律：当细分 n 次时，一共有 $depth=2^{n}$ 行三角形，且每个小三角形的边长为 $len=distance(v_{0},v_{1})/depth$ 。底行线性顶点数为 $vcnt=depth*2+1$ ，往上一行顶点数少 2。然后，我们还需要计算两个基向量 $\hat{x}=\hat{v_{2}v_{0}},~\hat{y}=\hat{v_{1}v_{0}}$ ，这样就可以根据 $v_{0}$ 算出所有点的坐标了。  
将两行看作一层，从最底层开始处理到最顶层。对于每一层，三角形带添加的顺序应该是下 - 上 - 下... 输出完一层后，使用`RestartStrip`添加新的一层。

```
// 根据模型空间位置计算Varyings，并输出到流
void AppendVertex(float3 positionOS, float2 baryCoord, inout TriangleStream<Varyings> triStream) {
    VertexOut vout;

    vout.positionOS = positionOS;
    vout.normalOS = normalize(vout.positionOS);
    vout.positionOS = normalize(vout.normalOS);

    Varyings gout;
    VertexPositionInputs vertexInputs = GetVertexPositionInputs(vout.positionOS);
    VertexNormalInputs normalInputs = GetVertexNormalInputs(vout.normalOS);
    gout.positionCS = vertexInputs.positionCS;
    gout.normalWS = normalInputs.normalWS;
    gout.baryCoord = baryCoord;

    triStream.Append(gout);
}

// 进行cnt次细分
void Subdivide(int cnt, VertexOut gin[3], inout TriangleStream<Varyings> triStream) {
    float depth = pow(2, cnt);// 层数
    uint vcnt = depth * 2 + 1; // 每层下行顶点数
    float len = distance(gin[0].positionOS, gin[1].positionOS) / depth; // 每个小三角形边长
    float3 rightup = normalize(gin[1].positionOS - gin[0].positionOS); // 右上向量
    float3 right = normalize(gin[2].positionOS - gin[0].positionOS); // 右向量

    float3 coordsX = {1.0, 0.0, 0.0};
    float3 coordsY = {0.0, 1.0, 0.0};

    // 用来当前层每行三角形的基准位置
    float3 down = gin[0].positionOS;
    float3 up = down + rightup * len;

    UNITY_UNROLL
    for (uint i = 0; i < depth; ++i) {
        for (uint j = 0; j < vcnt; ++j) {
            // 下行
            if (j % 2 == 0)
                AppendVertex(down + right * len * (j / 2), float2(coordsX[j % 3], coordsY[j % 3]), triStream);
            // 上行
            else
                AppendVertex(up + right * len * (j / 2), float2(coordsX[j % 3], coordsY[j % 3]), triStream);
        }
        // 迭代下一层
        vcnt -= 2;
        down = up;
        up += rightup * len;
        triStream.RestartStrip();
    }
}

[maxvertexcount(80)]
void LODPassGeometry(triangle VertexOut gin[3], uint primID : SV_PrimitiveID, inout TriangleStream<Varyings> triStream) {
    // 计算世界空间中心点与相机距离
    float3 centerW = mul(unity_ObjectToWorld, float3(0.0, 0.0, 0.0));
    float3 vDir = GetWorldSpaceViewDir(centerW);
    float dis = length(vDir);

    // 根据距离设置细分次数
    uint subCnt = 0;
    if (dis < 4)
        subCnt = 3;
    else if (dis < 8)
        subCnt = 2;
    else if (dis < 12)
        subCnt = 1;

    Subdivide(subCnt, gin, triStream);
}
```

这里的最大顶点数量限制了细分次数，再往上写就报错了，理论来说是可以进行无数次细分的。

完整代码：[github](https://github.com/dyxdyxdyx/TheTus-Unity-Projects/tree/master/TheTus-Unity-Projects/Assets/Tessellation%26Geometry/Shaders/GeometryShaders/LOD)

## 参考

《Real-Time Rendering 4th》

《DirectX 12 3D 游戏开发实战》

[《MSDN: Geometry Shader Stage》](https://learn.microsoft.com/en-us/windows-hardware/drivers/display/geometry-shader-stage)

[MSDN: Geometry-Shader Object](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-geometry-shader)

[MSDN: Primitive Topologies](https://learn.microsoft.com/en-us/windows/win32/direct3d11/d3d10-graphics-programming-guide-primitive-topologies)

[LearnOpenGL-CN: 几何着色器](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/09%20Geometry%20Shader)

[CatlikeCoding: Flat and Wireframe Shading](https://catlikecoding.com/unity/tutorials/advanced-rendering/flat-and-wireframe-shading/)