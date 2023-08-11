
本文主要介绍 URP 下 Tessellation 的实现。曲面细分位于渲染管线几何阶段，VertexShader 之后，GeometryShader 之前，主要分为三个阶段：Hull,Tessellation, 和 Domain。其中 Hull 和 Domain 是可编程的。其详细的渲染流程如下：

![[136b9af1a24ccc1c359f187dfd31c172_MD5.jpg]]

下面开始一步步实现 Tessellation 的 URP Shader。

1. 首先，需要一个 URP 下的基础 Shader，如下：

[https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/writing-shaders-urp-basic-unlit-structure.html](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@8.2/manual/writing-shaders-urp-basic-unlit-structure.html)

2. 使用细分时的最低着色器目标级别为 4.6。如果我们不手动设置，Unity 将发出警告并自动使用该级别

```
#pragma target 4.6
```

3. 使用 #pragma 声明相关的着色器方法

```
#pragma vertex BeforeTessVertProgram
#pragma hull HullProgram
#pragma domain DomainProgram
#pragma fragment FragmentProgram
```

4. 顶点着色器

顶点着色器不再像以前那样负责把顶点坐标从 ObjectSpace 转换到 ClipSpace，或是贴图 UV 转换等工作，此处只是简单得将 Attributes 中的数据传递给曲面细分阶段

```
// 顶点着色器，此时只是将Attributes里的数据递交给曲面细分阶段
ControlPoint BeforeTessVertProgram(Attributes v) {
     ControlPoint p;
        
     p.vertex = v.vertex;
     p.uv = v.uv;
     p.normal = v.normal;
     p.color = v.color;
        
     return p;
}
```

ControlPoint 的结构如下：

```
struct ControlPoint
{
     float4 vertex : INTERNALTESSPOS;
     float2 uv : TEXCOORD0;
     float4 color : COLOR;
     float3 normal : NORMAL;
};
```

可以和 Attributes 的结构比较一下

```
struct Attributes
{
     float4 vertex : POSITION;
     float3 normal : NORMAL;
     float2 uv : TEXCOORD0;
     float4 color : COLOR;
};
```

可见两个结构几乎相同的，只是 ControlPoint 中的 vertex 使用 INTERNALTESSPOS 代替 POSITION 语意，否则编译器会报位置语义的重用

5.Hull 着色器

细分阶段非常灵活，可以处理三角形，四边形或等值线。我们必须告诉它必须使用什么表面并提供必要的数据。这是 hull 程序的工作。

```
ControlPoint HullProgram(InputPatch<ControlPoint, 3> patch, uint id : SV_OutputControlPointID)
{
     return patch[id];
}
```

InputPatch 参数是向 Hull 程序传递曲面补丁的参数。Patch 是网格顶点的集合。必须指定顶点的数据格式。现在，我们将使用 ControlPoint 结构。在处理三角形时，每个补丁将包含三个顶点，此数量必须指定为 InputPatch 的第二个模板参数，所以第二个参数设置为 3。

Hull 程序的工作是将所需的顶点数据传递到细分阶段。尽管向其提供了整个补丁，但该函数一次仅应输出一个顶点。补丁中的每个顶点都会调用一次它，并带有一个附加参数，该参数指定应该使用哪个控制点（顶点）。该参数是具有 SV_OutputControlPointID 语义的无符号整数。

仅仅是这样的函数声明是不行的，编译器会报错，要求我们指定详细的参数，具体如下：

```
[domain("tri")]
```

domain: 指定 patch 的类型，可选的有：tri(三角形)、quad（四边形）、isoline（线段，苹果的 metal api 不支持：2018/8/21）。不同的 patch 类型，细分的方式也有差别，此处指定为三角形类型

```
[outputcontrolpoints(3)]
```

outputcontrolpoints：输出的控制点的数量（每个图元），不一定与输入数量相同，也可以新增控制点。此处设置为 3，是明确地告诉编译器每个补丁输出三个控制点

```
[outputtopology("triangle_cw")]
```

outputtopology：输出拓扑结构。当 GPU 创建新三角形时，它需要知道我们是否要按顺时针或逆时针定义它们。有三种：triangle_cw（顺时针环绕三角形）、triangle_ccw（逆时针环绕三角形）、line（线段）。

```
[partitioning("fractional_odd")]
```

partitioning：分割模式，起到告知 GPU 应该如何分割补丁的作用呢，共有三种：integer，fractional_even，fractional_odd。

```
[patchconstantfunc("MyPatchConstantFunction")]
```

patchconstantfunc：指定补丁常数函数。GPU 必须知道应将补丁切成多少部分。这不是一个恒定值，每个补丁可能有所不同。必须提供一个评估此值的函数，称为补丁常数函数（Patch Constant Functions）

6.Patch Constant Function(补丁常数函数)

Patch Constant Function 决定 Patch 的属性是如何细分的。这意味着它每个 Patch 仅被调用一次，而不是每个控制点被调用一次。这就是为什么它被称为常量函数，在整个 Patch 中都是常量的原因。

```
TessellationFactors MyPatchConstantFunction(InputPatch<ControlPoint, 3> patch) {
                float minDist = _MinTessDistance;
                float maxDist = _MaxTessDistance;
            
                TessellationFactors f;
            
                float edge0 = CalcDistanceTessFactor(patch[0].vertex, minDist, maxDist, _Tess);
                float edge1 = CalcDistanceTessFactor(patch[1].vertex, minDist, maxDist, _Tess);
                float edge2 = CalcDistanceTessFactor(patch[2].vertex, minDist, maxDist, _Tess);
            
                // make sure there are no gaps between different tessellated distances, by averaging the edges out.
                f.edge[0] = (edge1 + edge2) / 2;
                f.edge[1] = (edge2 + edge0) / 2;
                f.edge[2] = (edge0 + edge1) / 2;
                f.inside = (edge0 + edge1 + edge2) / 3;
                return f;
}
```

为了确定如何细分三角形，GPU 使用了四个细分因子。三角形面片的每个边缘都有一个因数。三角形的内部也有一个因素。三个边缘向量必须作为具有 SV_TessFactor 语义的 float 数组传递。内部因素使用 SV_InsideTessFactor 语义

```
struct TessellationFactors
{
     float edge[3] : SV_TessFactor;
     float inside : SV_InsideTessFactor;
};
```

实际上，此功能是与 HullProgram 并行运行的子阶段。

![[957ebed4a6a6f7dfabae62a878af584f_MD5.jpg]]

3.Domain 着色器

HUll 着色器只是使曲面细分工作所需的一部分。一旦细分阶段确定了应如何细分补丁，则由 Domain 着色器来评估结果并生成最终三角形的顶点。对于每个顶点，都会调用一次 Domain 着色器。一般来讲，这里会涉及到大量的计算，所有的顶点信息都会在这里重新计算，最后会将顶点坐标转换到投影空间。

```
[domain("tri")]//Hull着色器和Domain着色器都作用于相同的域，即三角形。我们通过domain属性再次发出信号
Varyings DomainProgram(TessellationFactors factors, OutputPatch<ControlPoint, 3> patch, float3 barycentricCoordinates : SV_DomainLocation)
{
    Attributes v;
        
    //为了找到该顶点的位置，我们必须使用重心坐标在原始三角形范围内进行插值。
    //X，Y和Z坐标确定第一，第二和第三控制点的权重。
    //以相同的方式插值所有顶点数据。让我们为此定义一个方便的宏，该宏可用于所有矢量大小。
     #define DomainInterpolate(fieldName) v.fieldName = \
                        patch[0].fieldName * barycentricCoordinates.x + \
                        patch[1].fieldName * barycentricCoordinates.y + \
                        patch[2].fieldName * barycentricCoordinates.z;
    
     //对位置、颜色、UV、法线等进行插值
     DomainInterpolate(vertex)
     DomainInterpolate(uv)
     DomainInterpolate(color)
     DomainInterpolate(normal)
                    
     //现在，我们有了一个新的顶点，该顶点将在此阶段之后发送到几何程序或插值器。
     //但是这些程序需要Varyings数据，而不是Attributes。为了解决这个问题，
     //我们让域着色器接管了原始顶点程序的职责。
     //这是通过调用其中的AfterTessVertProgram（与其他任何函数一样）并返回其结果来完成的。
     return AfterTessVertProgram(v);
}

Varyings AfterTessVertProgram (Attributes v)
{
    Varyings o;
    o.vertex = TransformObjectToHClip(v.vertex);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    o.posWS = TransformObjectToWorld(v.vertex);

    return o;
}
```

[domain("tri")]->Hull 着色器和 Domain 着色器都作用于相同的域，即三角形。我们通过 domain 属性再次发出信号。

参数说明：

1. TessellationFactors ：由 Patch Constant Function(补丁常数函数) 输入，细分参数。

2. OutputPatch：由 Hull 着色器传入的 patch 数据，尖括号的第二个参数与 Hull 着色器中的 InputPatch 对应。

3. SV_DomainLocation：由曲面细分阶段阶段传入的顶点位置信息。

Hull 着色器确定补丁的细分方式时，不会产生任何新的顶点。相反，它会为这些顶点提供重心坐标。使用这些坐标来导出最终顶点取决于 Domain 着色器。为了使之成为可能，每个顶点都会调用一次域函数，并为其提供重心坐标。

现在，我们有了一个新的顶点，该顶点将在此阶段之后发送到几何程序或插值器。但是这些程序需要 Varyings 数据，而不是 Attributes。为了解决这个问题，我们让 Domain 着色器接管了原始顶点程序的职责。这是通过调用其中的 AfterTessVertProgram 并返回其结果来完成的。

最后，完整的 Shader 代码如下：

```
Shader "V/URP/Tessellation"
{
    Properties
    {
        _Color("Color(RGB)",Color) = (1,1,1,1)
        _MainTex("MainTex",2D) = "gary"{}
        _Tess("Tessellation", Range(1, 32)) = 20
        _MaxTessDistance("Max Tess Distance", Range(1, 32)) = 20
        _MinTessDistance("Min Tess Distance", Range(1, 32)) = 1
    }
    SubShader
    {
        Tags
        {
            "RenderPipeline"="UniversalPipeline"
            "RenderType"="Opaque"
            "Queue"="Geometry+0"
        }
        
        Pass
        {
            Name "Pass"
            Tags 
            { 
                
            }
            
            // Render State
            Blend One Zero, One Zero
            Cull Back
            ZTest LEqual
            ZWrite On

            HLSLPROGRAM

            #pragma require tessellation
            #pragma require geometry
            
            #pragma vertex BeforeTessVertProgram
            #pragma hull HullProgram
            #pragma domain DomainProgram
            #pragma fragment FragmentProgram

            #pragma prefer_hlslcc gles
            #pragma exclude_renderers d3d11_9x
            #pragma target 4.6

            // Includes
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            CBUFFER_START(UnityPerMaterial)
            half4 _Color;
            float _Tess;
            float _MaxTessDistance;
            float _MinTessDistance;
            CBUFFER_END
            
			Texture2D _MainTex;
            float4 _MainTex_ST;
            
            //为了方便操作 定义预定义
            #define smp SamplerState_Point_Repeat
            // SAMPLER(sampler_MainTex); 默认采样器
            SAMPLER(smp);

            // 顶点着色器的输入
            struct Attributes
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float2 uv : TEXCOORD0;
                float4 color : COLOR;
            };

            // 片段着色器的输入
            struct Varyings
            {
                float4 color : COLOR;
                float3 normal : NORMAL;
                float4 vertex : SV_POSITION;
                float2 uv : TEXCOORD0;
                float3 posWS:TEXCOORD1;
            };

            // 为了确定如何细分三角形，GPU使用了四个细分因子。三角形面片的每个边缘都有一个因数。
            // 三角形的内部也有一个因素。三个边缘向量必须作为具有SV_TessFactor语义的float数组传递。
            // 内部因素使用SV_InsideTessFactor语义
            struct TessellationFactors
            {
                float edge[3] : SV_TessFactor;
                float inside : SV_InsideTessFactor;
            };

            // 该结构的其余部分与Attributes相同，只是使用INTERNALTESSPOS代替POSITION语意，否则编译器会报位置语义的重用
            struct ControlPoint
            {
                float4 vertex : INTERNALTESSPOS;
                float2 uv : TEXCOORD0;
                float4 color : COLOR;
                float3 normal : NORMAL;
            };

            // 顶点着色器，此时只是将Attributes里的数据递交给曲面细分阶段
            ControlPoint BeforeTessVertProgram(Attributes v)
            {
                ControlPoint p;
        
                p.vertex = v.vertex;
                p.uv = v.uv;
                p.normal = v.normal;
                p.color = v.color;
        
                return p;
            }

            // 随着距相机的距离减少细分数
            float CalcDistanceTessFactor(float4 vertex, float minDist, float maxDist, float tess)
            {
                float3 worldPosition = TransformObjectToWorld(vertex.xyz);
                float dist = distance(worldPosition,  GetCameraPositionWS());
                float f = clamp(1.0 - (dist - minDist) / (maxDist - minDist), 0.01, 1.0) * tess;
                return (f);
            }
            
            // Patch Constant Function决定Patch的属性是如何细分的。这意味着它每个Patch仅被调用一次，
            // 而不是每个控制点被调用一次。这就是为什么它被称为常量函数，在整个Patch中都是常量的原因。
            // 实际上，此功能是与HullProgram并行运行的子阶段。
            // 三角形面片的细分方式由其细分因子控制。我们在MyPatchConstantFunction中确定这些因素。
            // 当前，我们根据其距离相机的位置来设置细分因子
            TessellationFactors MyPatchConstantFunction(InputPatch<ControlPoint, 3> patch)
            {
                float minDist = _MinTessDistance;
                float maxDist = _MaxTessDistance;
            
                TessellationFactors f;
            
                float edge0 = CalcDistanceTessFactor(patch[0].vertex, minDist, maxDist, _Tess);
                float edge1 = CalcDistanceTessFactor(patch[1].vertex, minDist, maxDist, _Tess);
                float edge2 = CalcDistanceTessFactor(patch[2].vertex, minDist, maxDist, _Tess);
            
                // make sure there are no gaps between different tessellated distances, by averaging the edges out.
                f.edge[0] = (edge1 + edge2) / 2;
                f.edge[1] = (edge2 + edge0) / 2;
                f.edge[2] = (edge0 + edge1) / 2;
                f.inside = (edge0 + edge1 + edge2) / 3;
                return f;
            }

            //细分阶段非常灵活，可以处理三角形，四边形或等值线。我们必须告诉它必须使用什么表面并提供必要的数据。
            //这是 hull 程序的工作。Hull 程序在曲面补丁上运行，该曲面补丁作为参数传递给它。
            //我们必须添加一个InputPatch参数才能实现这一点。Patch是网格顶点的集合。必须指定顶点的数据格式。
            //现在，我们将使用ControlPoint结构。在处理三角形时，每个补丁将包含三个顶点。此数量必须指定为InputPatch的第二个模板参数
            //Hull程序的工作是将所需的顶点数据传递到细分阶段。尽管向其提供了整个补丁，
            //但该函数一次仅应输出一个顶点。补丁中的每个顶点都会调用一次它，并带有一个附加参数，
            //该参数指定应该使用哪个控制点（顶点）。该参数是具有SV_OutputControlPointID语义的无符号整数。
            [domain("tri")]//明确地告诉编译器正在处理三角形，其他选项：
            [outputcontrolpoints(3)]//明确地告诉编译器每个补丁输出三个控制点
            [outputtopology("triangle_cw")]//当GPU创建新三角形时，它需要知道我们是否要按顺时针或逆时针定义它们
            [partitioning("fractional_odd")]//告知GPU应该如何分割补丁，现在，仅使用整数模式
            [patchconstantfunc("MyPatchConstantFunction")]//GPU还必须知道应将补丁切成多少部分。这不是一个恒定值，每个补丁可能有所不同。必须提供一个评估此值的函数，称为补丁常数函数（Patch Constant Functions）
            ControlPoint HullProgram(InputPatch<ControlPoint, 3> patch, uint id : SV_OutputControlPointID)
            {
                return patch[id];
            }

			Varyings AfterTessVertProgram (Attributes v)
			{
				Varyings o;
				o.vertex = TransformObjectToHClip(v.vertex);
				o.uv = TRANSFORM_TEX(v.uv, _MainTex);
				o.posWS = TransformObjectToWorld(v.vertex);

                return o;
			}

            //HUll着色器只是使曲面细分工作所需的一部分。一旦细分阶段确定了应如何细分补丁，
            //则由Domain着色器来评估结果并生成最终三角形的顶点。
            //Domain程序将获得使用的细分因子以及原始补丁的信息，原始补丁在这种情况下为OutputPatch类型。
            //细分阶段确定补丁的细分方式时，不会产生任何新的顶点。相反，它会为这些顶点提供重心坐标。
            //使用这些坐标来导出最终顶点取决于域着色器。为了使之成为可能，每个顶点都会调用一次域函数，并为其提供重心坐标。
            //它们具有SV_DomainLocation语义。
            //在Demain函数里面，我们必须生成最终的顶点数据。
            [domain("tri")]//Hull着色器和Domain着色器都作用于相同的域，即三角形。我们通过domain属性再次发出信号
            Varyings DomainProgram(TessellationFactors factors, OutputPatch<ControlPoint, 3> patch, float3 barycentricCoordinates : SV_DomainLocation)
            {
                Attributes v;
        
                //为了找到该顶点的位置，我们必须使用重心坐标在原始三角形范围内进行插值。
                //X，Y和Z坐标确定第一，第二和第三控制点的权重。
                //以相同的方式插值所有顶点数据。让我们为此定义一个方便的宏，该宏可用于所有矢量大小。
                #define DomainInterpolate(fieldName) v.fieldName = \
                        patch[0].fieldName * barycentricCoordinates.x + \
                        patch[1].fieldName * barycentricCoordinates.y + \
                        patch[2].fieldName * barycentricCoordinates.z;
    
                    //对位置、颜色、UV、法线等进行插值
                    DomainInterpolate(vertex)
                    DomainInterpolate(uv)
                    DomainInterpolate(color)
                    DomainInterpolate(normal)
                    
                    //现在，我们有了一个新的顶点，该顶点将在此阶段之后发送到几何程序或插值器。
                    //但是这些程序需要Varyings数据，而不是Attributes。为了解决这个问题，
                    //我们让域着色器接管了原始顶点程序的职责。
                    //这是通过调用其中的AfterTessVertProgram（与其他任何函数一样）并返回其结果来完成的。
                    return AfterTessVertProgram(v);
            }
            
            // 片段着色器
            half4 FragmentProgram(Varyings i) : SV_TARGET 
            {    
                half4 mainTex = _MainTex.Sample(smp,i.uv);
                half4 c = _Color * mainTex;

                return c;
            }

            ENDHLSL
        }
    }
}
```