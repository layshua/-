# 快速次表面散射 Fast Subsurface Scattering 

> [!NOTE] 公式总结
> $$I_{back} = (saturate(V\cdot{-H}))^{p}\cdot s*thicknessMap.r$$
> 其中 $H=L+N\delta$
> $thicknessMap$ 为局部厚度贴图

## 原理
Subsurface Scatting
次表面散射可以使用**光线追踪**实现实时，游戏中更多的是对次表面散射的“快速模拟”。

![[02 PBR理论#^c36ln4]]
![[02 PBR理论#^lxo1ef]]

BSSRDF：
![[Pasted image 20230723134543.png|500]]


高质量的次表面散射使用光线追踪来实现，对于实时渲染来说计算成本太高。大多数现代引擎依赖于大规模的简化，尽管无法再现真实感，但可以产生可信的近似值。**本教程介绍了一种快速、廉价且令人信服的解决方案，可用于模拟表现出次表面散射的半透明材质。**

基于 Alpha Blend 实现的**透明（Transparent）材质**光线是没有散射的（下图左），只影响通过的光量。真实的**半透明（Translucent）材质**会发生散射，改变光线的路径。（下图右）
![[Pasted image 20230723144138.png]]

**次表面散射**：当光照射到半透明材质的表面时，一部分在内部传播，在分子之间反弹，直到找到出路。这通常会导致在特定点被吸收的光在其他地方再次射出。
次表面散射会产生漫射辉光，这种辉光可以在皮肤、大理石和牛奶等材质中看到。

**半透明昂贵的原因：**
1. 它需要模拟光线在材质内部的散射。每条光线可以分成多条，在一种材质内反射数百甚至数千次。
2. 在一个点上接收到的光在其他地方重新发射。这是不容易的。在实时渲染领域，GPU 希望着色器能够简单地使用局部属性来计算材质的最终颜色。每个顶点可以读取自己的属性，但不能读取临近顶点的。大多数实时解决方案必须绕过这些限制，并找到一种在不依赖非局部信息的情况下伪造光在材质内传播的方法。

本教程中描述的方法基于 Colin Barré-Brisebois 和 Marc Bouchard 在 2011 年 GDC 发表的 [Approximating Translucency for a Fast, Cheap and Convincing Subsurface Scattering Look](https://colinbarrebrisebois.com/2011/03/07/gdc-2011-approximating-translucency-for-a-fast-cheap-and-convincing-subsurface-scattering-look/) 用于 DICE 的《战地 3》寒霜 2 引擎

他们的解决方案背后的想法非常简单。在不透明材质（opaque materials）中，光贡献直接来自光源。而半透明材质具有与之相关的额外光贡献。在几何学上，可以看出一些光实际上是穿过材质并使其到达另一侧（右下图）。

![[Pasted image 20230723144233.png]]

**现在每个光都有两个不同的反射贡献：正面和背面照明**。
正面照明（front illumination）：使用 Unity 的标准 PBR 光照模型
背面照明：找到一种方法来描述来自 $- L$ 的贡献，并以某种方式模拟材质内部可能发生的扩散过程。

### 背光
![[Pasted image 20230723135132.png]]
> $dot (N,+L)$ 是正光，$dot (V,-L)$ 是背光

如前所述，我们像素的最终颜色取决于两个组件的总和。第一个是 “传统”(traditional) 照明。第二个是虚拟光源照亮 (virtual light source illuminating) 我们模型背面的光线贡献。这给人的感觉是来自原始光源的光实际上穿过了材质。

数学建模：红点在 “黑暗” 一侧，它应该被 $- L$ 照亮。
![[Pasted image 20230723145227.png]]
作为第一个近似值 (approximation)，我们可以说由半透明度引起的背光照明量 $I_{back}$ 与 $(V\cdot-L)$ 成正比。在传统的漫反射着色器中，这将是 $(N\cdot L)$ 。我们可以看到，**我们没有在计算中包括法线，因为光线只是从材质中出来，而不是反射在材质上。**
![[Pasted image 20230723162619.png]]
>光线从背后照射时，效果很平，因为法线没有参与运算
### 次表面扰动/扭曲

**法线应该对光离开材质的角度有微小的影响**。该技术的作者引入了一个称为**次表面扰动的参数** $\delta$ ，该参数强制向量 $- L$ 趋向 $N$。从物理上讲， **$\delta$ 控制了法线偏转出射背光的强度**。根据提出的解决方案，背面半透明部分的强度变为：
 $$I_{back} = V\cdot-(L+N\delta)=V\cdot -H$$ .
当 $\delta$ = 0，我们得到 $(V\cdot-L)$ 和前边推导的一样。但是，当 $\delta=1$ 时，我们计算的是观察方向 $V$ 和 $-(L+N)$ 之间的点积。

$L+N$ 即半程向量 $H$（注意，和 BlinnPhong 的半程向量定义不同，BlinnPhong 中为 $H=\frac{L+V}{{|L+V|}}$），不需要单位化
![[Pasted image 20230723145357.png]]

**从几何学上讲， $\delta$ 从 $0$ 到 $1$ 导致光 $L$ 的感知方向的变化。粉色阴影区域显示背光源的方向范围。在下图中您可以看到， $\delta=0$ 对象似乎是从紫色光源照亮的。当 $\delta$ 朝向 1 移动时，感知的光源方向向紫色方向移动。**
![[Pasted image 20230723145824.png]]
> $\delta$ 越大，$-H$ 就越远离 $-L$，散射越多

$\delta$ **目的是模拟某些半透明材质以不同强度散射背光的趋势。较高的 $\delta$ 值将导致背光散射更多。**

### 背光扩散

我们已经有了一个方程式模拟半透明材质。 $I_{back}$ 不能用于计算最终的光贡献。

有两种主要方法可以使用。第一个依赖于纹理。如果你想对光在材质中漫射的方式进行全面的艺术控制，你应该 clamp $I_{back}$ 在 0 和 1 之间，并用它来采样背光的最终强度。不同的 ramp textures 将模拟不同材质内的光传输。我们将在本教程的下一部分中看到如何使用它来显著更改此着色器的结果。

然而，**该技术的作者使用的方法不依赖于纹理。它仅使用 Cg 代码创建曲线**：增加两个新参数 p (power) 和 s (scale) 用于更改曲线的属性。

**我们得出了一个依赖于观察方向的方程来模拟背光的反射率：** $$I_{back} = (saturate(V\cdot-(L+N\delta)))^{p}\cdot s$$
* $L$ 是光源方向
* $V$ 是观察方向
* $N$ 是法线
* $\delta$ ，改变背光的感知方向，使其更符合法线
 - $p$ 和 $s$（代表 power 和 scale）确定背光如何传播，类似高光反射中的参数

### 效果增强：局部厚度

很明显，背光量很大程度上取决于材质的密度和厚度。理想情况下，我们需要知道光在材质内部传播的距离，并相应地对其进行衰减。在下图中看到具有相同入射角的三种不同光线在材质中穿过不同的长度（厚度）。

![[Pasted image 20230723152354.png]]

但是，从着色器的角度来看，我们无法访问局部几何体或光线的历史。最佳解决方法是依赖**外部局部厚度图（local thickness map）**。这是一个纹理，映射到我们的表面，表明该部分材质的 “厚度”。“厚度”的概念使用不严格，因为实际厚度实际上取决于光线的角度。

![[Pasted image 20230723152400.png]]

>光线在材质中穿过的长度（厚度）取决于光源角度 $L$。

话虽如此，我们必须记住，这种半透明的方法并不是要在物理上准确，而是要足够现实以愚弄玩家的眼睛。您可以看到在雕像模型上可视化的局部厚度图。白色对应于模型中半透明效果更强的部分，近似于厚度的概念。
![[Pasted image 20230723152815.png]]

> [!question]  如何生成局部厚度图？
> 该技术的作者提出了一种从任何模型自动创建局部厚度图的方法。步骤如下：  
> 1. 翻转模型的面
> 2. 将 AO 渲染到纹理贴图
> 3. 反转纹理的颜色  
> 
> 这个过程背后的基本原理是，通过在背面上渲染 AO，可以大致“平均形状内部发生的所有光传输”。
> 
> 厚度也可以直接存储在顶点中，而不是纹理

我们现在知道我们需要考虑材质的局部厚度。最简单的方法是提供我们可以采样的纹理贴图。虽然在物理上不准确，但它可以产生可信的结果。此外，局部厚度的编码方式允许艺术家保持对效果的完全控制。

## 代码实现
```cs
Shader "Custom/SSS"
{
    Properties
    {
        _MainTex ("MainTex", 2D) = "white" {}
        _BaseColor("BaseColor", Color) = (1,1,1,1)
        [Normal] _NormalMap("NormalMap", 2D) = "bump" {}
        _NormalScale("NormalScale", Range(0, 10)) = 1

        [Header(Specular)]
        _SpecularPower("SpecularPower", Range(1, 128)) = 32
        _SpecularScale("SpecularScale", Range(0, 10)) = 1
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        
        [Header(SSS)]
        _Distortion("Distortion", Range(0, 1)) = 0
        //次表面扰动
        _BackLightPower("BackLightPower", Range(0, 5)) = 1
        //背光扩散
        _BackLightScale("BackLightScale", Range(0, 5)) = 1
        _BackLightColor("BackLightColor", Color) = (1,1,1,1)
        //局部厚度
        _ThicknessMap("ThicknessMap", 2D) = "white" {}
        
    }
    
    HLSLINCLUDE

    #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
    #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/lighting.hlsl"

    CBUFFER_START(UnityPerMaterial)
    float4 _MainTex_ST;
    float4 _BaseColor;
    float _NormalScale;
    float _SpecularPower;
    float _SpecularScale;
    float4 _SpecularColor;
    float _Distortion;
    float _BackLightPower;
    float _BackLightScale;
    float4 _BackLightColor;
    CBUFFER_END

    TEXTURE2D(_MainTex);
    SAMPLER(sampler_MainTex);
    TEXTURE2D(_NormalMap);
    SAMPLER(sampler_NormalMap);
    TEXTURE2D(_ThicknessMap);
    SAMPLER(sampler_ThicknessMap);
    
    struct Attributes
    {
        float4 positionOS : POSITION;
        float4 color : COLOR;
        float3 normalOS : NORMAL;
        float4 tangentOS : TANGENT;
        float2 uv : TEXCOORD0;
    };

    struct Varyings
    {
        float4 positionCS : SV_POSITION;
        float4 color : COLOR0;
        float2 uv : TEXCOORD0;
        float3 positionWS: TEXCOORD1;
        float3 normalWS : TEXCOORD2;
        float4 tangentWS : TEXCOORD3;
        float3 bitangentWS : TEXCOORD4;
        float3 viewDirWS : TEXCOORD5;
        float3 lightDirWS : TEXCOORD6;
    };
    ENDHLSL
    
    SubShader
    {
        Tags
        {
            "RenderPipeline" = "UniversalPipeline"
            "RenderType"="Opaque" 
        }

        Pass
        {
            Tags
            {
                "LightMode"="UniversalForward"
            }
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            
            Varyings vert(Attributes i)
            {
                Varyings o = (Varyings)0;
        
                VertexPositionInputs vertexInput = GetVertexPositionInputs(i.positionOS.xyz);
                VertexNormalInputs normalInput = GetVertexNormalInputs(i.normalOS, i.tangentOS);

                o.uv = TRANSFORM_TEX(i.uv, _MainTex);
                o.positionCS = vertexInput.positionCS;
                o.positionWS = vertexInput.positionWS;

                //TBN
                o.normalWS = normalInput.normalWS;
                real sign = i.tangentOS.w * GetOddNegativeScale();
                o.tangentWS = float4(normalInput.tangentWS.xyz, sign);
                o.bitangentWS = normalInput.bitangentWS;
                
                o.viewDirWS = GetWorldSpaceNormalizeViewDir(o.positionWS);
                
                return o;
            }

            float4 frag(Varyings i) : SV_Target
            {
                //纹理采样
                float4 MainTex = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv);
                float3 normalMap = UnpackNormalScale(
                    SAMPLE_TEXTURE2D(_NormalMap, sampler_NormalMap, i.uv), _NormalScale);
                float thickness = SAMPLE_TEXTURE2D(_ThicknessMap, sampler_ThicknessMap, i.uv).r;

                //向量计算
                float3x3 TBN = float3x3(i.tangentWS.xyz, i.bitangentWS.xyz, i.normalWS.xyz);
                float3 N = TransformTangentToWorld(normalMap, TBN, true);
                float3 L = normalize(_MainLightPosition.xyz);
                float3 V = normalize(i.viewDirWS);
                float3 H = normalize(L + V); //正面光照的半角向量
                float3 H_back = L + N*_Distortion;  //背面光照的半角向量
                float NdotL = dot(N, L);
                float NdotH = dot(N, H);
                float VdotNegativeH = dot(V, -H_back);
                
                //正面光照
                float3 diffuse = (0.5 * NdotL + 0.5) * _BaseColor.rgb * _MainLightColor.rgb;
                float3 specular = pow(max(0, NdotH), _SpecularPower) * _SpecularScale * _SpecularColor.rgb * _MainLightColor.rgb;
                float4 frontColor = MainTex * float4((diffuse + _GlossyEnvironmentColor.rgb) + specular, 1);
                

                //背面光照
                float4 backColor = float4(pow(saturate(VdotNegativeH), _BackLightPower) * _BackLightScale * _BackLightColor.rgb * thickness,1);

                return frontColor+backColor;
            }
            ENDHLSL
        }
    }
    FallBack "Packages/com.unity.render-pipelines.universal/FallbackError"
}
```
## 思考
![[Pasted image 20230723153544.png]]