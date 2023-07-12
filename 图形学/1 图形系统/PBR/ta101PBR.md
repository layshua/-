
# 【第五章】猴子的 PBR 笔记
## PBR

**理论**

PBR (Physically Based Shading) 是基于与现实世界的物理原理所创建的一种渲染技术，相比于传统的基于经验的模型（Phong Blin-Phong 等）更具有物理准确性。基于物理的光照模型必须满足一下三个条件:
1. 基于微平面 (Microfacet) 的表面模型。
2. 能量守恒。
3. 基于物理的 BRDF。

**微表面模型**

微观尺度上，物体表面有无数的为表面构成。用粗糙度去衡量一个表面的粗糙程度, 越粗糙则说明微表面的法线方向越不一致，反射的光越分散，反之则越集中。

![](<images/1683732881208.png>)

较高的粗糙度值显示出来的镜面反射的轮廓要更大一些。与之相反地，较小的粗糙值显示出的镜面反射轮廓则更小更锐利。

**能量守恒**

能量守恒定律是自然界普遍的基本定律之一。一般表述为：能量既不会凭空产生，也不会凭空消失，它只会从一种形式转化为另一种形式，或者从一个物体转移到其它物体，而能量的总量保持不变。

将能量守恒定律应用到光照模型中: 出射光线的能量永远不能超过入射光线的能量。可以用渲染方程来描述:

![](<images/1683732881380.png>)

在 PBR 中考虑简化的形式即反射方程:

![](<images/1683732881797.png>)

再看这张图:

![](<images/1683732881906.png>)

从上图可以看出随着粗糙度的上升镜面反射区域的面积会增加，作为平衡，镜面反射区域的平均亮度则会下降。

![](<images/1683732882106.png>)

按照能量守恒的关系，首先计算镜面反射部分，它的值等于入射光线被反射的能量所占的百分比。然后漫反射部分就可以直接由镜面反射部分计算得出：

```
float kS = calculateSpecularComponent(...); // 镜面反射部分
float kD = 1.0 - ks;                        // 漫反射 部分
```

而在实时渲染中，会有两种光对物体表面造成影响，分别是直接光和间接光，那么与两种反射组合，会有四中光照反射情况。

![](<images/1683732882430.png>)

**基于物理的 BRDF**

双向反射分布函数（Bidirectional Reflectance Distribution Function）描述的是入射光和反射光关系。在各种 brdf 模型中，Cook-Torrance 效果更真实应用最广泛。

![](<images/1683732882588.png>)

其中 Ks 是镜面反射所占百分比，Kd 是漫反射所占百分比 Kd=1-Ks

将 Cook-Torrance 带入反射方程中可得:

![](<images/1683732882697.png>)

其中 F 菲涅尔描述了光被反射的比例，代表了反射方程的 ks，两者可以合并，所以最终的反射方程为:

![](<images/1683732883054.png>)

**直接光漫反射 + 镜面反射**

由于直接光通常是平行光或者点光源都是有限的数量，因此直接将它们的光照结果相加即是最终的积分结构。我们目前只考虑一个平行光，因此直接光的计算结构即使积分结果。而间接光从四面八方射到物体表面上来，数量是无限的，因此要用积分计算出所有间接光的计算结果。

直接光的漫反射比较简单:

```
ks = F;//F的计算参考后面
kd = 1 - ks;
Diffuse = kd*BaseColor/PI;
```

间接光需要考虑 DFG 三项

**D Distribution of normal function**

法线分布函数描述的是微表面的法线方向与半角向量对齐的概率，如果对齐那么认为该反射光可以看到，否则没有。

![](<images/1683732883298.png>)

D 的 Trowbridge-Reitz GGX 公式:

![](<images/1683732883394.png>)

```
//D
float D_DistributionGGX(float3 N,float3 H,float Roughness) {
    float a             = Roughness*Roughness;
    float a2            = a*a;
    float NH            = saturate(dot(N,H));
    float NH2           = NH*NH;
    float nominator     = a2;
    float denominator   = (NH2*(a2-1.0)+1.0);
    denominator         = PI * denominator*denominator;
    
    return              nominator/ max(denominator,0.001) ;//防止分母为0
}
```

仅有 D 项

```
float Roughness = 0.356;
FinalColor = D_DistributionGGX(N,H,Roughness);
```

![](<images/1683732883468.png>)

**F Frenel function**

如果你站在湖边，低头看脚下的水，你会发现水是透明的，反射不是特别强烈；如果你看远处的湖面，你会发现水并不是透明的，但反射非常强烈。这就是 “菲涅尔效应”。简单的讲，就是视线垂直于表面时，反射较弱，而当视线非垂直表面时，夹角越小，反射越明显。如果你看向一个圆球，那圆球中心的反射较弱，靠近边缘较强。

![](<images/1683732883538.png>)

F 的 Schlick 公式:

![](<images/1683732883633.png>)

n 代表介质的折射率, 1 为空气的折射率，以下为真实材质测量的 F0 数值表

![](<images/1683732883722.png>)

非金属的 F0 数值较小，金属 F0 的数值较大，出于简化计算的原因，通过金属度在一个预设的 F0 和自身颜色之间经行插值。

```
float3 F0 = 0.04;
F0 = lerp(F0, BaseColor, Metallic);
```

```
//F
float3 F_FrenelSchlick(float HV,float3 F0) {
    return F0 +(1 - F0)*pow(1-HV,5);
    //return lerp(pow(1-HV,5),1,F0);
}
```

仅有 F 项:

```
float3 BaseColor = float3(1,0.782,0.344);
 float MEtallic = 0.874;
 float3 F0 = 0.04;
 F0 = lerp(F0, BaseColor, Metallic);
 FinalColor = F_FrenelSchlick(HV,F0);
```

从背面观察:

![](<images/1683732883789.png>)

**G Geometry function**

几何项体现了微表面的自我遮蔽现象，即入射光线或者反射光线会被自身凹凸不平的表面遮蔽。一般而言，这一项也跟材质的粗糙程度有关，可以理解为越粗超的材质表面越有可能发生自我遮蔽现象。

![](<images/1683732883956.png>)

G 的 Schlick-GGX 公式:

![](<images/1683732884254.png>)

k 在直接光与间接光中分别为:

![](<images/1683732884314.png>)

如前面的示意图所示 Smith 认为 G 项分应该分为为两部分: 第一部分是 Geometry Obstruction 第二部分是 Geometry Shadowing 即:

![](<images/1683732884517.png>)

```
//G
float GeometrySchlickGGX(float NV,float Roughness) {
    float r = Roughness +1.0;
    float k = r*r / 8.0;      //直接光
    float nominator = NV;
    float denominator = k + (1.0-k) * NV;
    return nominator/ max (denominator, 0.001) ;//防止分母为 0
}

float G_GeometrySmith(float3 N,float3 V,float3 L,float Roughness) {
    float NV = saturate(dot(N,V));
    float NL = saturate(dot(N,L));

    float ggx1 = GeometrySchlickGGX(NV,Roughness);
    float ggx2 = GeometrySchlickGGX(NL,Roughness);

    return ggx1*ggx2;

}
```

仅有 G 项:

```
float Roughness = 0.356;
FinalColor = G_GeometrySmith(N,V,L,Roughness);
```

![](<images/1683732884564.png>)

直接光的最后效果:

```
//================== PBR  ============================================== //
// float3 BaseColor = float3(0.5,0.3,0.2);
float3 BaseColor = _BaseColor;
float Roughness = _Roughness;
float Metallic = _Metallic;
float3 F0 = lerp(0.04,BaseColor,Metallic);
float3 Radiance = _LightColor0.xyz;

//================== Direct Light  ============================================== //
//Specular
//Cook-Torrance BRDF
float HV = saturate(dot(H,V));
float NV = saturate(dot(N,V));
float NL = saturate(dot(N,L));

float D = D_DistributionGGX(N,H,Roughness);
float3 F = F_FrenelSchlick(HV,F0);
float G = G_GeometrySmith(N,V,L,Roughness);

float3 KS = F;
float3 KD = 1-KS;
KD*=1-Metallic;
float3 nominator = D*F*G;
float denominator = max(4*NV*NL,0.001);
float3 Specular = nominator/denominator;

//Diffuse
float3 Diffuse = KD * BaseColor / PI;
float3 DirectLight = (Diffuse + Specular)*NL *Radiance;
FinalColor.rgb = DirectLight.rgb;
```

![](<images/1683732884614.png>)

**间接光漫反射 + 镜面反射**

间接光是用 IBL (Image Based Lighting) 来模拟的。与直接光相比，IBL 把周围环境整体视为一个大光源，IBL 通常使用（取自现实世界或从 3D 场景生成的）环境立方体贴图 (Cubemap) ，我们可以将立方体贴图的每个像素视为光源，在渲染方程中直接使用它。这种方式可以有效地捕捉环境的全局光照和氛围，使物体**更好地融入**其环境。

表示环境或场景辐照度的一种方式是（预处理过的）环境立方体贴图，给定这样的立方体贴图，可以将立方体贴图的每个纹素视为一个光源。使用一个方向向量 wi 对此立方体贴图进行采样，就可以获取该方向上的场景辐照度。

给定方向向量 wi ，获取此方向上场景辐射度的方法就简化为：

```
float3 radiance = tex2D(_CubemapEnvironment, lightDirection).rgb;
```

虽然这么简化了，但是如果要求解反射方程的话，我们依然需要计算球面积分，这对于实时渲染来说太昂贵了。解决这个问题，我们可以通过预计算一些结果并存储起来，然后在运行时利用这些中间结果计算最终值。首先将反射方程拆分一下。

![](<images/1683732884676.png>)

通过将积分分成两部分，可以分开研究漫反射和镜面反射部分。

间接光漫反射部分：

![](<images/1683732884791.png>)

间接光镜面反射部分：

![](<images/1683732884844.png>)

**漫反射部分**

颜色 c 、折射率 kd 和 π 在整个积分是常数，不依赖于任何积分变量。基于此，可以将常数项移出漫反射积分：

![](<images/1683732884942.png>)

这给了我们一个只依赖于 wi 的积分（假设 p 位于环境贴图的中心）。有了这些知识，我们就可以计算或预计算一个新的立方体贴图，它在每个采样方向——也就是纹素——中存储漫反射积分的结果，这些结果是通过卷积计算出来的。

卷积的特性是，对数据集中的一个条目做一些计算时，要考虑到数据集中的所有其他条目。这里的数据集就是场景的辐射度或环境贴图。因此，要对立方体贴图中的每个采样方向做计算，我们都会考虑半球 Ω 上的所有其他采样方向。

为了对环境贴图进行卷积，我们通过对半球 Ω 上的大量方向进行离散采样并对其辐射度取平均值，来计算每个输出采样方向 wo 的积分。以 wo 方向所在的半球，采样方向 wi 。

![](<images/1683732884990.png>)

这个预计算的立方体贴图，在每个采样方向 wo 上存储其积分结果，可以理解为场景中所有能够击中面向 wo 的表面的间接漫反射光的预计算总和。这样的立方体贴图被称为**辐照度贴图**，因为经过卷积计算的立方体贴图能让我们从任何方向有效地直接采样场景（预计算好的）辐照度。

![](<images/1683732885056.png>)

对于一点 P 它所受间接光来自整个 CubeMap (以 CubeMap 中的每个像素为一个光源) ，假设 P 点的法线 N 为光线的出射方向，那么可以离线计算出 P 点所受环境光照的影响。给定任何方向向量 wi ，我们可以对预计算的辐照度图采样以获取方向 wi 的总漫反射辐照度。为了确定片段上间接漫反射光的数量（辐照度），我们获取以表面法线为中心的半球的总辐照度。获取场景辐照度的方法就简化为：

```
float3 irradiance = tex3D(irradianceMap, N);
```

**生成辐照度贴图**

实时进行积分计算太消耗，可以先离线计算出来。然而，理论上在半球上采样方向是无限的 , 因此不可能从半球上的所有方向采样环境光照。不过我们可以对有限数量的方向采样以近似求解，在半球内均匀间隔或随机取方向可以获得一个近似精确的辐照度值，从而离散地计算积分结果。

伪代码:

```
//Normal为当前点P的 采样方向
以Normal方向为半球，获取一组随机的方向 且将其定义为Normals 
float3 avg =0
foreach( N in Normals)
{
    avg = SampleCube(CubeMap,N)
}
avg /= Normals.Count
//avg 则是 所有环境光 对当前点P的最终影响结果
IrradianceMap.SetValue(avg,Normal) //将结果储存在辐照度贴图中
```

**LightProbe 与球协函数**

辐照度贴图是固定的，如果游戏中有多个场景，比如室内室外那么肯定是不能为每一个场景都单独弄一个辐照度贴图的，会消耗大量内存。游戏引擎中使用 LightProbe 放置在场景中，去近似采样并存储以该 LightProbe 为中心的 "辐照度贴图"，但不是单独存一张贴图，而是用球协函数存储相应的信息，最终存储的是球协函数的参数，通常是 9 个 float，这比单独存一张贴图节省多了。

在 Unity 中放置 LightProbe，并且烘培后，使用 ShadeSH9 函数即可获取当前的点的 "辐照度"。

![](<images/1683732885110.png>)

仅显示球协函数的光照:

```
//注意需要设置 Tags{ "LightMode"="ForwardBase"} 才能正确得到球鞋函数值
//否则ShadeSh9返回值为0 
float3 irradianceSH = ShadeSH9(float4(N,1)));
FinalColor.rgb = irradianceSH;
```

![](<images/1683732885172.png>)

HDR 图可以在这里下载: [https://hdrihaven.com/hdris/](https://hdrihaven.com/hdris/)

![](<images/1683732885293.png>)

仅显示间接光漫反射:

```
float3 irradianceSH = ShadeSH9(float4(N,1));
return irradianceSH.rgbb;
float3 Diffuse_Indirect = irradianceSH * BaseColor / PI *KD_IndirectLight;
FinalColor.rgb = Diffuse_Indirect;
```

![](<images/1683732885374.png>)

骚操作一下:

将上面的 HDR 图直接在 PS 里做一个高斯模糊，然后再去采样

![](<images/1683732885432.png>)

```
float3 EnvCubeMap = texCUBE(_EnvCubeMap,N).xyz;
return (EnvCubeMap.xyzz);
```

得到下面结果:

![](<images/1683732885488.png>)

会比较暗，这是因为在 PS 做模糊的时候，将 HDR 从 32 位转为了 16 位，因此手动把精度加回来 (不准确，但是可以理解这个过程)

```
float3 EnvCubeMap = texCUBE(_EnvCubeMap,N).xyz;
//用ToneMaping HDR=>LDR
//x3.75 是为了补救损失的精度
return ACESToneMapping(EnvCubeMap.xyzz*3.75);
```

和球协的有点区别，但是明显这张图比球协的更准确地反映了环境光的 "黄"

![](<images/1683732885551.png>)

球协函数原理参考：

[https://zhuanlan.zhihu.com/p/50208005](https://zhuanlan.zhihu.com/p/50208005)

[https://pdfs.semanticscholar.org/83d9/28031e78f15d9813061b53d25a4e0274c751.pdf](https://pdfs.semanticscholar.org/83d9/28031e78f15d9813061b53d25a4e0274c751.pdf)

[https://basesandframes.files.wordpress.com/2016/05/spherical-harmonic-lighting-gdc-2003.pdf](https://basesandframes.files.wordpress.com/2016/05/spherical-harmonic-lighting-gdc-2003.pdf)

[http://web.cs.wpi.edu/~emmanuel/courses/cs563/S05/talks/mark_w5_spherical_harmonic_lighting.pdf](http://web.cs.wpi.edu/~emmanuel/courses/cs563/S05/talks/mark_w5_spherical_harmonic_lighting.pdf)

**镜面反射部分**

![](<images/1683732885654.png>)

  
将镜面反射的公式近似地拆成两部分（EpicGames SplitSum）

![](<images/1683732885769.png>)

**间接光镜面积分第一部分**

![](<images/1683732885871.png>)

对于高光部分而言，如果材质表面越光滑，则该点出射光大部分贡献来源于入射光的镜面反射，很小一部分来源于各个方向的漫反射。所以，这次我们在计算积分的时候，需要把粗糙度也考虑进去。如果粗糙度越低，那么积分的贡献范围就越集中，相当于卷积核的面积越小，大权重集中在核中心，卷积结果尺寸越大，产生越清晰的反射效果。反之亦然。

![](<images/1683732885919.png>)

于是可以根据材质的粗糙度，映射到某个粗糙度区间，然后把计算结果保存到不同的 LOD 等级中。这张具有不同 LOD 等级的立方体贴图也称为 pre-filtered environment map。

![](<images/1683732885974.png>)

后续在运行时只需要根据粗糙等级做一次采样就可以得到高光积分的第一部分。

```
//UNITY_SPECCUBE_LOD_STEPS 在  "UnityStandardConfig. cginc"中 
// #define UNITY_SPECCUBE_LOD_STEPS 6
float mip = Roughness*(1.7 - 0.7*Roughness) * UNITY_SPECCUBE_LOD_STEPS ; 
float4 rgb_mip = UNITY_SAMPLE_TEXCUBE_LOD(unity_SpecCube0,R,mip);
//间接光镜面反射采样的预过滤环境贴图
//unity_SpecCube0_HDR储存的是 最近的ReflectionProbe
float3 EnvSpecularPrefilted = DecodeHDR(rgb_mip, unity_SpecCube0_HDR);
FinalColor.rgb = EnvSpecularPrefilted.rgb;
```

仅显示 ReflectionProbe 信息，即间接光镜面积分第一部分结果

![](<images/1683732886038.png>)

**ReflectionProbe**

Reflection Probe 是六个相机分别捕捉它所在位置的 6 个方向的场景信息，最后储存在一张 Cubemap 贴图里。

![](<images/1683732886093.png>)

![](<images/1683732886446.png>)

在场景中放置 ReflectionProbe Bake 之后才会有信息。

![](<images/1683732886502.png>)

当用一个未烘培过的 ReflectionProbe 去靠近物体 (仅显示 ReflectionProbe 信息)，明显发现物体变黑了，因为这个未烘培过的 ReflectionProbe 没有信息，而 unity_SpecCube0 是最近的一个 ReflectionProbe。

![](<images/1683732886578.png>)

**间接光镜面积分第二部分**

![](<images/1683732886666.png>)

经过推导后可得如下：

![](<images/1683732886718.png>)

计算这个积分有两种办法，第一种是 BRDF 积分图第二种是函数拟合

**BrdfLut**

假设每个方向的入射光都是白色的（L (p, x)=1.0 ），就可以在给定粗糙度、光线 ωi 法线 n 夹角 n⋅ωi 的情况下，预计算 BRDF 的响应结果。以 X 轴为法线与入射光的夹角 (dot (N, L))，以 Y 轴为粗糙度，将计算的结果存储在一张 2D 贴图上 (Lut)，该贴图称为 BRDF 积分贴图。积分的结果分别储存在贴图的 RG 通道中。使用的时候可以直接采样该贴图即可。

![](<images/1683732886768.png>)

```
float3 F_IndirectLight = FresnelSchlickRoughness(NV,F0,Roughness);
float2 env_brdf = tex2D(_BRDFLUTTex, float2(NV, Roughness)).rg;
float3 IndirectLightSpecularPartTwo = F_IndirectLight * env_brdf.r + env_brdf.g;
FinalColor.rgb = IndirectLightSpecularPartTwo ;
```

仅显示 Specular PartTwo

![](<images/1683732886829.png>)

上面代码中的菲涅尔系数计算和前面的有两点不同，第一点是这里没有用于计算微片元朝向的 D 函数，计算菲涅尔系数使用的是真正的 nv 而不是 vh，第二点是考虑了粗糙度。使用 nv 是由于环境光来自半球内围绕法线 N 的所有方向，因此无法和直接光照中的法线分布函数 D 一样使用单个半角向量来确定微平面分布，所以在此我们只能使用法线和视线的夹角（即 nv）来计算菲涅尔效果。

```
float3 FresnelSchlickRoughness(float NV,float3 F0,float Roughness) {
    return F0 + (max(float3(1.0 - Roughness, 1.0 - Roughness, 1.0 - Roughness), F0) - F0) * pow(1.0 - NV, 5.0);
}
```

**数值拟合**

PartTwo 的结果出了可以预计算出来外，还可以用函数拟合来实时计算。

```
// 这是使命召唤黑色行动2的函数拟合 Black Ops II
float2 EnvBRDFApprox_BlackOp2(float Roughness, float NV) {
    float g = 1 -Roughness;
    float4 t = float4(1/0.96, 0.475, (0.0275 - 0.25*0.04)/0.96, 0.25);
    t *= float4(g, g, g, g);
    t += float4(0, 0, (0.015 - 0.75*0.04)/0.96, 0.75);
    float A = t.x * min(t.y, exp2(-9.28 * NV)) + t.z;
    float B = t.w;
    return float2 ( t.w-A,A);
}
```

```
//UE4 在 黑色行动2 上的修改版本
float2 EnvBRDFApprox_UE4(float Roughness, float NoV ) {
    // [ Lazarov 2013, "Getting More Physical in Call of Duty: Black Ops II" ]
    // Adaptation to fit our G term.
    const float4 c0 = { -1, -0.0275, -0.572, 0.022 };
    const float4 c1 = { 1, 0.0425, 1.04, -0.04 };
    float4 r = Roughness * c0 + c1;
    float a004 = min( r.x * r.x, exp2( -9.28 * NoV ) ) * r.x + r.y;
    float2 AB = float2( -1.04, 1.04 ) * a004 + r.zw;
    return AB;
}
```

```
float3 F_IndirectLight = FresnelSchlickRoughness(NV,F0,Roughness);
float2 env_brdf = EnvBRDFApprox_UE4(Roughness,NV);
float3 IndirectLightSpecularPartTwo = F_IndirectLight * env_brdf.r + env_brdf.g;
FinalColor.rgb = IndirectLightSpecularPartTwo ;
```

用函数拟合的效果比 BrdfLut 的效果更好，且更节省性能。

![](<images/1683732886879.png>)

关于 BrdfLut 的生产参考:

[https://learnopengl.com/PBR/IBL/Specular-IBL](https://learnopengl.com/PBR/IBL/Specular-IBL)

数值拟合的参考:

[https://blog.selfshadow.com/publications/s2013-shading-course/lazarov/s2013_pbs_black_ops_2_slides_v2.pptx](https://blog.selfshadow.com/publications/s2013-shading-course/lazarov/s2013_pbs_black_ops_2_slides_v2.pptx)

公式推导参考：

[https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf](https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf)

最终效果:

```
//================== Direct Light  ============================================== //
//Specular
//Cook-Torrance BRDF
float HV = saturate (dot (H, V));
float NV = saturate(dot(N,V));
float NL = saturate(dot(N,L));

float D = D_DistributionGGX(N,H,Roughness);
float3 F = F_FrenelSchlick(HV,F0);
float G = G_GeometrySmith(N,V,L,Roughness);

float3 KS = F;
float3 KD = 1-KS;
KD*=1-Metallic;
float3 nominator = D*F*G;
float denominator = max(4*NV*NL,0.001);
float3 Specular = nominator/denominator;
//Diffuse
float3 Diffuse = KD * BaseColor / PI;

float3 DirectLight = (Diffuse + Specular)*NL *Radiance;
//================== Indirect Light  ============================================== //
float3 IndirectLight = 0;
//Specular
float3 R = reflect(-V,N);
float3 F_IndirectLight = FresnelSchlickRoughness(NV,F0,Roughness);
// float3 F_IndirectLight = F_FrenelSchlick(NV,F0);
float mip = Roughness*(1.7 - 0.7*Roughness) * UNITY_SPECCUBE_LOD_STEPS ;
float4 rgb_mip = UNITY_SAMPLE_TEXCUBE_LOD(unity_SpecCube0,R,mip);

//间接光镜面反射采样的预过滤环境贴图
float3 EnvSpecularPrefilted = DecodeHDR(rgb_mip, unity_SpecCube0_HDR);

//LUT采样
// float2 env_brdf = tex2D(_BRDFLUTTex, float2(NV, Roughness)).rg; //0.356

//数值近似
float2 env_brdf = EnvBRDFApprox(Roughness,NV);
float3 Specular_Indirect = EnvSpecularPrefilted  * (F_IndirectLight * env_brdf.r + env_brdf.g);

//Diffuse           
float3 KD_IndirectLight = 1 - F_IndirectLight;
KD_IndirectLight *= 1 - Metallic;

float3 irradianceSH = ShadeSH9(float4(N,1));
float3 Diffuse_Indirect = irradianceSH * BaseColor / PI *KD_IndirectLight;

//float3 EnvCubeMap = texCUBE(_EnvCubeMap,N).xyz;

IndirectLight = Diffuse_Indirect + Specular_Indirect;

float4 FinalColor =0;
FinalColor.rgb = DirectLight + IndirectLight;
//HDR => LDR aka ToneMapping
FinalColor.rgb = ACESToneMapping(FinalColor.rgb);

//Linear => Gamma
// FinalColor = pow(FinalColor,1/2.2);
return FinalColor
```

![](<images/1683732886958.png>)

与 Unity 默认的对比 BaseColor=(255,199,88) Metallic=0.95 Roughness=0.21 参数一致用屏幕后处理统一做 ToneMapping

![](<images/1683732887029.png>)

可以看出 Unity 的效果偏亮一点，因为 Unity 在实现 PBR 算法的时候直接光漫反射与间接光漫反射都没有除以 PI (为了与老版本效果兼容)，而我们的算法是标准的 PBR 算法，在表现金属上会有更细腻的亚光效果。

完整的 PBR 效果: (BaseColor + Metallic +Roughness + Normal + AO)

![](<images/1683732887165.png>)

完整的 PBR 代码链接:

[https://github.com/ipud2/Unity-Basic-Shader/blob/master/PBR_BetterThanUnity.shader](https://github.com/ipud2/Unity-Basic-Shader/blob/master/PBR_BetterThanUnity.shader)

# 5.1 PBR
[[1 .PBR总览]]
![[TA101_PBR_6.jpg]]
![[TA101_PBR_7.jpg]]
![[TA101_PBR_9.jpg]]
![[TA101_PBR_10.jpg]]
![[TA101_PBR_11.jpg]]


![[Pasted image 20221029160325.png]]
![[Pasted image 20221029160335.png]]
![[Pasted image 20221029160340.png]]

![[Pasted image 20221029160350.png]]

BRDF就是为了解决上述问题
![[Pasted image 20221029160441.png]]
### 5.1.1 经验模型
![[Pasted image 20221029160631.png]]
![[Pasted image 20221029160641.png]]

## 5.1.1 基于物理的材质（BRDF计算）
**ks+kd = 1**
![[Pasted image 20221101211702.png]]
![[Pasted image 20221101211713.png]]
![[Pasted image 20221102144938.png]]
### 直接光（主光）
#### BRDF
![[Pasted image 20221101211851.png]]
![[Pasted image 20221029161107.png]]
![[Pasted image 20221101211935.png]]
**注意：
直接光数量有限，计算直接光将光照结果相加
间接光数量无限，计算间接光要用积分算所有的**
![[Pasted image 20221102172101.png]]
```c
/*  直接光（主光） */
// Cook-Torrance BRDF  
// 漫反射部分  
float3 Ks = F_FrenelSchlick(VH, F0); //菲涅尔描述了光被反射的比例  
float3 Kd = (1-Ks) * (1 - Metallic);  
//float3 Diffuse = Kd * BaseColor / PI;  
float3 Diffuse = Kd * BaseColor; //没有除以 PI, 颜色亮一些  
  
// 高光反射部分  
float D = D_DistributionGGX(N, H, Roughness);  
float3 F = Ks;   
float G = G_Smith(N, V, L, Roughness);  
float3 Specular = D * F * G / max(0.0001, 4 * NV * NL);
```
#### D法线分布函数
法线分布函数描述的是微表面的法线方向与半角向量对齐的概率，如果对齐那么认为该反射光可以被看到，否则没有。

GGX：
![[Pasted image 20221029161311.png]]

```c
float D_DistributionGGX(float3 N, float3 H, float Roughness)
{
		float a = Roughness * Roughness;
		float a2 = a * a;  //为什么取a的四次方，这里是参考的ue4，也可以使用a的二次方进行计算
		float NH = max(0, dot(N,H));
		float NH2 = NH * NH;
		float nominator = a2; //分子
		float denominator = (NH2*(a2-1.0)+1.0);  //分母
		denominator = PI * denominator * denominator;  
		
		return nominator / max(0.00001, denominator);  //防止分母为0
}
```

![[Pasted image 20221101212846.png]]

#### G几何（遮蔽）函数
![[Pasted image 20221029161443.png]]
![[Pasted image 20221101214734.png]]
![[Pasted image 20221101214916.png]]

```c
//G_SchlickGGX  
float G_SchlickGGX(float NV, float Roughness)  
{  
    float a = Roughness + 1.0;  
    float k = a * a / 8.0;  //直接光  
    float nominator = NV;  
    float denominator = NV * (1.0 - k) + k;  
    
    return nominator / max(0.00001, denominator); //防止分母为0    
 }  
 
//G_Smith  
float G_Smith(float3 N, float3 V, float3 L, float Roughness)  
{  
    float NV = max(0, dot(N,V));  
    float NL = max(0, dot(N,L));  
    
    float GGX1 = G_SchlickGGX(NV, Roughness);  
    float GGX2 = G_SchlickGGX(NL,Roughness);  
  
    return GGX1 * GGX2;  
}
```

![[Pasted image 20221101215517.png]]
#### F菲涅尔方程
我不是科学家，没必要公式都会推，会用就可以！
![[Pasted image 20221101213118.png]]
![[Pasted image 20221029161549.png]]
![[Pasted image 20221101213507.png]]
n代表介质的折射率，1为空气的折射率
**v·h或v·n都可以，有两种形式，这里我才用毛星云白皮书里提到的v·h的方法。**
**非金属的FO数值较小，金属FO的数值较大，出于简化计算的原因，通过金属度在一个预设的FO和自身颜色之间经行插值。**

```c
//F
float3 F0 = lerp(0.04, BaseColor, Metallic);

float3 F_FresnelSchlick(float VH, float3 F0)
{
		return F0 + (1 - F0) * pow(1 - VH, 5);
		 
}
```

![[Pasted image 20221101214427.png]]

**真实材质F0数值表参考：**![[技术美术/PBR/PBR白皮书/bonus/PBR-Material-F0-Quick-Reference-Chart.png]]
#### 代码
![[TA101_PBR_36.jpg]]
### 间接光（环境光）
#### 漫反射
前面提到：
直接光数量有限，计算直接光将光照结果相加
间接光数量无限，计算间接光要用积分算所有的

由于实时渲染中机器算力不足，我们通常需要对间接光进行预处理，这里使用到了**IBL**，近似出间接光。
![[Pasted image 20221101220611.png]]
![[Pasted image 20221101220920.png]]
![[Pasted image 20221101221123.png]]
![[Pasted image 20221101221141.png]]
##### 辐照度贴图 irradiance

![[Pasted image 20221101221323.png]]
1. 为什么选择法线所在半球？
因为法线只朝上，不会对下面区域造成影响。
2. 自定义采样数量，并不是采样所有点，开销太大。
![[Pasted image 20221101221538.png]]
##### 球谐函数 Spherical Harmonics

辐照度贴图是固定的，如果游戏中有多个场景，比如室内室外那么肯定是不能为每一个场景都单独弄一个辐照度贴图的，会消耗大量内存。**游戏引擎中使用LightProbe放置在场景中，去近似采样并存储以该LightProbe为中心的“辐照度贴图"，但不是单独存一张贴图，而是用球协函数存储相应的信息，最终存储的是球协函数的参数,通常是9个float**,这比单独存一张贴图节省多了。
在Unity中放置LightProbe，并且烘培后，使用`ShadeSH9`函数即可获取当前的点的"辐照度"。
[Unity中Light Probe详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/38550884)
![[Pasted image 20221101223730.png]]

![[Pasted image 20221101222402.png]]
![[Pasted image 20221101222827.png]]
**HDR图下载**：[HDRIs • Poly Haven](https://polyhaven.com/hdris)

#### 镜面反射
![[Pasted image 20221101223143.png]]
##### PartOne
对于高光部分而言，如果材质表面越光滑，则该点出射光大部分贡献来源于入射光的镜面反射，很小一部分来源于各个方向的漫反射。所以，这次**我们在计算积分的时候，需要把<font color="#ff0000">粗糙度</font>也考虑进去**。
**如果粗糙度越低，那么积分的贡献范围就越集中，相当于卷积核的面积越小，大权重集中在核中心，卷积结果尺寸越大，产生越清晰的反射效果。反之亦然。**
![[Pasted image 20221101223443.png]]
可以根据材质的粗糙度，映射到某个粗糙度区间，然后把计算结果保存到不同的LOD等级中(Unity默认6级)。这张具有不同LOD等级的立方体贴图也称为**pre-filtered environment map**。
![[Pasted image 20221101223523.png]]

![[Pasted image 20221101224230.png]]
###### 反射探针 ReflectionProbe
![[Pasted image 20221101224526.png]]
各轴互相垂直，为什么垂直：4 x 90 = 360
![[Pasted image 20221101224717.png]]
##### PartTwo
![[Pasted image 20221101225517.png]]
同样受硬件限制，我们需要进行预处，有两种方法。
- **BRDF LUT**，将原公式结果离线生成出来。
- **数值拟合**
###### BRDF LUT

LUT：Look Up Table  查找表
假设每个方向的入射光都是白色的【L(p,x)= 1.0】，就可以在给定粗糙度，光线wi法线n夹角n·wi的情况，预计算BRDF的响应结果。以x轴的法线与入射光的夹角（NL01），以Y轴为粗糙度，将计算的结果存储在一张2D贴图上（lut），该帖图称为为**BRDF积分贴图**。积分的结果分别储存在贴图的**RG通道**中。使用的时候直接采样该帖图即可。
![[Pasted image 20221101234002.png|300]]


![[Pasted image 20221101233948.png]]
>参考： https://www.gamedevs.org/uploads/real-shading-in-unreal-engine-4.pdf
###### F的计算 与 引入粗糙度的菲涅尔
- **间接光代码中的菲涅尔系数计算和直接光的有两点不同**:
- 第一点是这里没有用于计算微片元朝向的D函数，计算菲涅尔系数使用的是真正的**NV**而不是HV
- 第二点是**考虑了粗糙度**。使用NV是由于环境光来自半球内围绕法线N的所有方向，因此无法和直接光照中的法线分布函数D一样使用单个半角向量来确定微平面分布，所以在此我们只能使用法线和视线的夹角（即NV)来计算菲涅尔效果。
![[Pasted image 20221101234203.png]]

```c
// 直接光部分 NV或VH均可  
float3 F_FrenelSchlick(float VH,float3 F0)  
{  
    return F0 +(1.0 - F0)*pow(1.0-VH,5);  
}  
//间接光部分 只能使用NV并引入粗糙度  
float3 FresnelSchlickRoughness(float NV,float3 F0,float Roughness)  
{  
    float smoothness = 1.0 - Roughness;  
    return F0 + (max(smoothness.xxx, F0) - F0) * pow(1.0 - NV, 5.0);  
}
```

![[Pasted image 20221101234222.png]]
###### 数值拟合
**使命召唤黑色行动2的函数拟合**
如果输出该float2值，会发现和Lut贴图很相似。

```c
float2 BRDF_Ami = AmiBRDFApprox(i.uv.y, i.uv.x);  
return pow(float4(BRDF_Ami,0,0),2.2);
```

![[Pasted image 20221102201016.png|300]]
![[Pasted image 20221102143359.png]]
参考：https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf

![[Pasted image 20221102143419.png]]
![[Pasted image 20221102143439.png]]
### HDR与LDR 色调映射
在PBR中做光照运算时，最终输出的颜色值时常超过1，而超过1的部分在显示器中显示就会“泛白过爆"，为了解决这个问题，将HDR(High Dynamic Range)的颜色值转换到LDR(Low Dynamic Range)的算法叫做
ToneMapping(色调映射)。在各种ToneMapping算法中**ACESTonemapping效果与性能兼优**。
ToneMapping: HDR->LDR

可以使用unity自带的后处理，也可以手写

ToneMapping.cs **挂载到相机**
```C#
using System.Collections;  
using System.Collections.Generic;  
using UnityEngine;  
  
// [ExecuteInEditorMode]  
[ExecuteInEditMode]  
public class ToneMapping : MonoBehaviour  
{  
    Material material;  
    // Start is called before the first frame update  
    void Start()  
    {  
        var shader = Shader.Find("Ulit/ToneMapping");  
        material = new Material(shader);  
    }  
  
    // Update is called once per frame  
    void Update()  
    {            }  
  
    private void OnRenderImage(RenderTexture src, RenderTexture dest)   
    {  
        Graphics.Blit(src,dest,material);  
    }  
  
}
```

ToneMapping.shader
```c
Shader "Ulit/ToneMapping"  
{  
    Properties  
    {  
        _MainTex ("Texture", 2D) = "white" {}  
    }  
    SubShader  
    {  
        // No culling or depth  
        Cull Off ZWrite Off ZTest Always  
  
        Pass        {  
            CGPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
  
            #include "UnityCG.cginc"  
  
            struct appdata  
            {  
                float4 vertex : POSITION;  
                float2 uv : TEXCOORD0;  
            };  
  
            struct v2f  
            {  
                float2 uv : TEXCOORD0;  
                float4 vertex : SV_POSITION;  
            };  
  
            float3 ACESToneMapping(float3 x)  
            {  
                float a = 2.51f;  
                float b = 0.03f;  
                float c = 2.43f;  
                float d = 0.59f;  
                float e = 0.14f;  
                return saturate((x*(a*x+b))/(x*(c*x+d)+e));  
            }  
  
            v2f vert (appdata v)  
            {  
                v2f o;  
                o.vertex = UnityObjectToClipPos(v.vertex);  
                o.uv = v.uv;  
                return o;  
            }  
  
            sampler2D_float _MainTex;  
              
            float4 frag (v2f i) : SV_Target  
            {  
                float4 col = tex2D(_MainTex, i.uv);  
                  
                col.rgb = ACESToneMapping(col.rgb);  
  
                return col;  
            }  
            ENDCG  
        }  
    }  
}
```
![[2 光照基础#2. ACES曲线#]]
```c
float3 ACESToneMapping(float3 x)
{
		float a = 2.51f;
		float b = 0.03f;
		float c = 2.43f;
		float d = 0.59f;
		float e = 0.14f;
		return saturate((x * (a * x + b))/(x * (c * x + d) + e));
}
```
### Gamma与Linear
[[2 光照基础#2.6 伽马矫正]]
人眼对低亮度的颜色变化感知强，对高亮度的颜色变化感知弱
为了提高图片的显示辨识度，将自然界线性的颜色储存在非线性的空间(Gamma)
在渲染中先将图片转化为线性空间(Linear)，再进行颜色加减乘除操作才是正确的，最后再转为非线性(Gamma)的颜色输出
在Unity中如果设置颜色空间为Gamma，那么在进行PBR计算之前需要用代码手动转到Linear,并且最终输出到显示器的颜色时候需用代码手动转到Gamma空间
在Unity中如果设置颜色空间为Linear，那么Unity会帮我们把图片自动转到Linear空间，并且最终输出到显示器的颜色时候自动转到Gamma空间

### FinalColor
![[Pasted image 20221102143626.png]]
![[Pasted image 20221102143845.png]]
unity没有的basecolor没有乘NL
### 总结
![[Pasted image 20221029161818.png]]
### 拓展
![[Pasted image 20221029163621.png]]
![[Pasted image 20221029163736.png]]
### 案例：PBR头盔
![[Pasted image 20221102205521.png]]
```less
Shader "Unlit/MyHelmet"
{
    Properties
    {
        _BRDFLUTTex ("BRDFLUT", 2D) = "white" {}
        _BaseColorTex ("BaseColor", 2D) = "white" {}
        _MetallicTex ("Metallic", 2D) = "white" {}
        _RoughnessTex ("Roughness", 2D) = "white" {}
        _EmissionTex ("Emission", 2D) = "white" {}
        [HDR]_EmissionColor("Emission Color",Color)=(1,1,1,1)
        _NormalTex ("Normal", 2D) = "black" {}
        _AOTex ("AO", 2D) = "white" {}
    }
    SubShader
    {
        //LightMode 设置为ForwardBase，否则ShadeSH9()会出错。
        Tags { "RenderType"="Opaque" "LightMode"="ForwardBase"}

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            #include "UnityGlobalIllumination.cginc" //ShadeSH9()头文件
            
            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;
                
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
                float3 normal : TEXCOORD1;
                float3 tangent : TEXCOORD2;
                float3 bitangent : TEXCOORD3;
                float3 worldPos : TEXCOORD4;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            float  _Value, _RangeValue;
            float4 _Color, _BaseColor;

            float _Metallic, _Roughness;
            sampler2D _BRDFLUTTex;
            // samplerCUBE _EnvCubeMap;

            sampler2D _BaseColorTex, _MetallicTex, _RoughnessTex;
            sampler2D _EmissionTex, _AOTex, _NormalTex;
            float4 _EmissionColor;
            
            #define PI 3.14159265358979323846

            /* D法线分布函数：GGX */
            float D_DistributionGGX(float3 N, float3 H, float Roughness)
            {
                float a = Roughness * Roughness;
                float a2 = a * a;  //为什么取a的四次方，这里是参考的ue4，也可以使用a的二次方进行计算
                float NH =  max(0, dot(N,H));
                float NH2 = NH * NH;
                float nominator = a2; //分子
                float denominator = (NH2*(a2-1.0)+1.0);  //分母
                denominator = PI * denominator * denominator;  
                return nominator / max(0.00001, denominator);  //防止分母为0
            }
            
            /* G几何（遮蔽）函数 ：Schlick-GGX + Smith */
            // G_SchlickGGX  
            float G_SchlickGGX(float NV, float Roughness)  
            {  
                float a = Roughness + 1.0;  
                float k = a * a / 8.0;  //直接光  
                float nominator = NV;  
                float denominator = NV * (1.0 - k) + k;  
                
                return nominator / max(0.00001, denominator); 
             }
            
            // G_Smith  
            float G_Smith(float3 N, float3 V, float3 L, float Roughness)  
            {  
                float NV = max(0, dot(N,V));  
                float NL = max(0, dot(N,L)); 
                
                float GGX1 = G_SchlickGGX(NV, Roughness);  
                float GGX2 = G_SchlickGGX(NL,Roughness);  
              
                return GGX1 * GGX2;  
            }
            
            /* F菲涅尔方程：Schlick近似  */
            // 直接光部分 NV或VH均可
            float3 F_Schlick(float VH,float3 F0)
            {
                return F0 +(1.0 - F0)*pow(1.0-VH,5);
            }
            
            //间接光部分 只能使用NV并引入粗糙度
            float3 F_SchlickRoughness(float NV,float3 F0,float Roughness)
            {
                float smoothness = 1.0 - Roughness;
                return F0 + (max(smoothness.xxx, F0) - F0) * pow(1.0 - NV, 5.0);
            }

            /* 数值拟合 */
            // 使命召唤黑色行动2 的函数拟合
            // float2 AmiBRDFApprox(float Roughness, float NV)
            // {
            //     float g = 1 -Roughness;
            //     float4 t = float4(1/0.96, 0.475, (0.0275 - 0.25*0.04)/0.96, 0.25);
            //     t *= float4(g, g, g, g);
            //     t += float4(0, 0, (0.015 - 0.75*0.04)/0.96, 0.75);
            //     float A = t.x * min(t.y, exp2(-9.28 * NV)) + t.z;
            //     float B = t.w;
            //     return float2 ( t.w-A,A);
            // }
            
            // UE4 在黑色行动2 上的修改版本
            float2 AmiBRDFApprox(float Roughness, float NoV )
            {
                // [ Lazarov 2013, "Getting More Physical in Call of Duty: Black Ops II" ]
                // Adaptation to fit our G term.
                const float4 c0 = { -1, -0.0275, -0.572, 0.022 };
                const float4 c1 = { 1, 0.0425, 1.04, -0.04 };
                float4 r = Roughness * c0 + c1;//mad:multiply add
                float a004 = min( r.x * r.x, exp2( -9.28 * NoV ) ) * r.x + r.y;//mad
                float2 AB = float2( -1.04, 1.04 ) * a004 + r.zw;//mad
                return AB;
            }

            /* 色调映射 ToneMapping */
            float3 ACESToneMapping(float3 x)
            {
                float a = 2.51f;
                float b = 0.03f;
                float c = 2.43f;
                float d = 0.59f;
                float e = 0.14f;
                return saturate((x*(a*x+b))/(x*(c*x+d)+e));
            }
            
            float4 ACESToneMapping(float4 x)
            {
                float a = 2.51f;
                float b = 0.03f;
                float c = 2.43f;
                float d = 0.59f;
                float e = 0.14f;
                return saturate((x*(a*x+b))/(x*(c*x+d)+e));
            }
            
            v2f vert (appdata v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = v.uv;
                o.normal = UnityObjectToWorldNormal(v.normal);
                o.tangent = UnityObjectToWorldDir(v.tangent);
                o.bitangent = normalize(cross(o.normal, o.tangent) * v.tangent.w);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex);
                
                return o;
            }

            
            fixed4 frag (v2f i) : SV_Target
            {
                // 纹理采样
                float3 BaseColor = tex2D(_BaseColorTex, i.uv);
                float3 NormalMap = UnpackNormal(tex2D(_NormalTex,i.uv));
                float Roughness = tex2D(_RoughnessTex, i.uv).r;
                float Metallic = tex2D(_MetallicTex, i.uv).r;
                float3 Emission = tex2D(_EmissionTex, i.uv);
                float3 AO = tex2D(_AOTex, i.uv);
                
                // 变量准备
                float3 L = normalize(UnityWorldSpaceLightDir(i.worldPos));
                float3 V = normalize(UnityWorldSpaceViewDir(i.worldPos));
                float3 H = normalize(L + V);
                float3x3 TBN = float3x3(i.tangent, i.bitangent, i.normal);
                float3 N = normalize(mul(NormalMap, TBN));
                
                float VH = max(0, dot(V, H));
                float NV = max(0, dot(N, V));
                float NL = max(0,dot(N,L));
                
                float3 F0 = lerp(0.04, BaseColor, Metallic); //Fresnel F0：插值区分非金属和金属不同的F0值，非金属的FO数值较小，金属FO的数值较大
                
                /*  直接光（主光） */
                // Cook-Torrance BRDF
                // 漫反射部分
                float3 Ks = F_Schlick(VH, F0); //菲涅尔描述了光被反射的比例
                float3 Kd = (1-Ks) * (1 - Metallic);
                float3 Diffuse = Kd * BaseColor / PI; 
                //float3 Diffuse = Kd * BaseColor; //unity内置的PBR没有除以 PI, 颜色亮一些
                
                // 高光反射部分
                float D = D_DistributionGGX(N, H, Roughness);
                float3 F = Ks; 
                float G = G_Smith(N, V, L, Roughness);
                float3 Specular = D * F * G / max(0.0001, 4 * NV * NL);

                float3 DirectLightColor = (Diffuse + Specular) * NL * _LightColor0.rgb; //NL在这里起到了阴影贴图的作用，背光处变暗
                
                /*  间接光（环境光ambient） */
                // 漫反射部分
                float3 Ks_Ami = F_SchlickRoughness(NV, F0, Roughness);
                float3 Kd_Ami = (1 - Ks_Ami) * (1 - Metallic);
                float3 irradiance =  ShadeSH9(float4(N, 1));  // 球谐函数
                float3 Diffuse_Ami = irradiance * BaseColor * Kd_Ami / PI;
                //float3 Diffuse_Ami = irradiance * BaseColor * Kd_Ami; //没有除以 PI
                
                // 高光反射部分
                float3 F_Ami = Ks_Ami;
                // PartOne
                float3 R = reflect(-V, N);
                //UNITY_SPECCUBE_LOD_STEPS在"UnityStandardConfig.cginc"中// #define UNITY_SPECCUBE_LOD_STEPS (6)
                //根据材质的粗糙度，映射到某个粗糙度区间，然后把计算结果保存到不同的LOD等级中(Unity默认6级)
                float mip = Roughness * (1.7 - 0.7 * Roughness) * UNITY_SPECCUBE_LOD_STEPS;
                // 得到预过滤环境贴图 pre-filtered environment map
                float4 rgb_mip = UNITY_SAMPLE_TEXCUBE_LOD(unity_SpecCube0, R, mip);
                // 采样：光滑的地方采样清晰，粗造的地方采样模糊
                float3 preFilteredEnvironmentMap = DecodeHDR(rgb_mip,unity_SpecCube0_HDR);

                // PartTwo
                //LUT采样
                //float2 env_brdf = tex2D(_BRDFLUTTex, float2(NV, Roughness)).rg; //0.356
                //float2 env_brdf = tex2D(_BRDFLUTTex, float2(lerp(0, 0.99, NV), lerp(0, 0.99, Roughness))).rg;
                
                // 数值拟合
                float2 BRDF_Ami = AmiBRDFApprox(Roughness, NV);
                // float2 BRDF_Ami = AmiBRDFApprox(i.uv.y,i.uv.x);
                //  return pow(float4(BRDF_Ami,0,0),2.2);
                
                float3 Specular_Ami = preFilteredEnvironmentMap  * (F_Ami * BRDF_Ami.r + BRDF_Ami.g);

                float3 AmbientLightColor = (Diffuse_Ami + Specular_Ami) * AO;

                /*  颜色混合 */
                float3 FinalColor = DirectLightColor + AmbientLightColor + (Emission * _EmissionColor);
                
                return float4(FinalColor,1);
            }
            ENDCG
        }
    }
}
```
## 5.1.2 基于物理的相机（鸽）

## 5.1.3 基于物理的灯光（鸽）

## 5.1.4 IBL基于图像照明（鸽）
基于图像照明（Image-Based Lighting, IBL）
[深入理解 PBR/基于图像照明 (IBL) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/66518450)




