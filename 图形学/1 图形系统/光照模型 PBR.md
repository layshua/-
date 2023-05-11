
## **目录**

*   **约定**
*   **Lambert Lighting**
*   **Half-Lambert Lighting**
*   **Phong**
*   **Blin-Phong Lighting**
*   **Wrap Lighting**
*   **Gouraud Lighting**
*   **Banded Lighting**
*   **Minnaert Lighting**
*   **Oren-Nayar Lighting**
*   **BackLight or CheapSSS 背光 | 简易次表面散射**
*   **Anisotropic Lighting**
*   **PBR**
*   **PBR Anistropy 基于物理的各项异性高光**
*   **ClearCoat 清漆**
*   **Cloth 布料**

**约定**

L 光照方向

N 法线方向

V 视角方向

T 切线方向

H 半角向量

这些向量 为世界空间下的向量 且是 单位向量


**Banded Lighting**

这个方法更像一个 Trick 而不是一个光照模型。基本原理就是对 NdotL 做条状化处理，卡通渲染效果的二分卡边原理与此类似。

```
//Banded Lighting
float NL = saturate(dot(N,L));
float BandedStep = 6;
float BandedNL = floor(NL*BandedStep)/BandedStep;
float4 Diffuse = BandedNL;
FinalColor = Diffuse;
```

![](<images/1683732877586.png>)

整点颜色渐变

```
//Banded Lighting
float NL = (dot(N,L))*0.5+0.5;//[-1,1] => [0,1]
// float _BandedStep = 6;
float BandedNL = floor(NL*_BandedStep)/_BandedStep;
float4 Diffuse = lerp(_ColorA,_ColorB,BandedNL);
// float4 Diffuse = smoothstep(_ColorA,_ColorB,BandedNL);
FinalColor = Diffuse;
```

![](<images/1683732877909.png>)

三个颜色渐变

```
//Banded Lighting
float NL = (dot(N,L))*0.5+0.5;
// float _BandedStep = 6;
float BandedNL = floor(NL*_BandedStep)/_BandedStep;
float4 C1 = lerp(_ColorA,_ColorB,BandedNL);
float4 C2 = lerp(_ColorB,_ColorC,BandedNL);
float4 Diffuse = lerp(C1,C2,BandedNL);
// float4 Diffuse = smoothstep(_ColorA,_ColorB,BandedNL);
FinalColor = Diffuse;
```

![](<images/1683732877955.png>)

![](<images/1683732878012.png>)


**Oren-Nayar Lighting**

此模型用来描述光在粗糙表面的反射情况，相比于 Lambert 模型，它考虑了 粗糙度参数，常用来模拟比较粗糙的表面。比如游戏风之旅人的沙漠就是在 Oren-Nayar 的模型上做的改进。([https://www.gdcvault.com/play/1017742/Sand-Rendering-in](https://www.gdcvault.com/play/1017742/Sand-Rendering-in))

![](<images/1683732878487.png>)

原 Oren-Nayar 模型公式, 公式中光线方向与反射方向是用球坐标系表示的，

![](<images/1683732878544.png>)

这是一个转化后便于写代码的版本:

![](<images/1683732878600.png>)

注意 arccos 是 arc cos，即反余弦函数

参考:

[https://en.wikipedia.org/wiki/Oren%E2%80%93Nayar_reflectance_model](https://en.wikipedia.org/wiki/Oren%E2%80%93Nayar_reflectance_model)

[https://lonalwah.wordpress.com/2013/11/29/oren-nayar-reflectance-diffuse-getting-rough/](https://lonalwah.wordpress.com/2013/11/29/oren-nayar-reflectance-diffuse-getting-rough/)

```
float NL = saturate(dot(N,L));
float NV = saturate(dot(N,V));
float theta2 = _Roughness*_Roughness;
float A = 1 - 0.5*(theta2/(theta2 +0.33));
float B = 0.45 *(theta2/(theta2+0.09));
float acosNV = acos(NV);
float acosNL = acos(NL);
float alpha = max(acosNV,acosNL);
float beta =  min(acosNV,acosNL);
float gamma = length(V - N*NV) * length(L - N*NL);
float Diffuse = 1;
float OrenNayer = Diffuse * NL *(A+ B*max(0,gamma)*sin(alpha)*tan(beta));
FinalColor = OrenNayer;
```

狂调参数

![](<images/1683732878655.png>)

加一张 Roughness 贴图

![](<images/1683732878775.png>)

```
//Oren-Nayer
float roughness = tex2D(_RoughnessTex,i.uv).r *_Roughness;
float NL = saturate(dot(N,L));
float NV = saturate(dot(N,V));
float theta2 = roughness*roughness;
float A = 1 - 0.5*(theta2/(theta2 +0.33));
float B = 0.45 *(theta2/(theta2+0.09));
float acosNV = acos(NV);
float acosNL = acos(NL);
float alpha = max(acosNV,acosNL);
float beta =  min(acosNV,acosNL);
float gamma = length(V - N*NV) * length(L - N*NL);
float Diffuse = 1;
float OrenNayer = Diffuse * NL *(A+ B*max(0,gamma)*sin(alpha)*tan(beta));
FinalColor = OrenNayer;
```

hh:D

![](<images/1683732879195.png>)

**BackLight**

先看图片:

![](<images/1683732879249.png>)

![](<images/1683732879303.png>)

概念:

反射: 入射光与反射光在表面的同一侧，且入射点与反射点相同

![](<images/1683732879355.png>)

次表面散射: 入射光与反射光在表面的同一侧，且入射点与反射点不同

![](<images/1683732879437.png>)

透射: 入射光与反射光在表面的不同侧，即光线投过了物体

![](<images/1683732879511.png>)

时常有人把**透射**与**次表面散射**弄混，因为这两种现象看起来那么像，且真实光照总是包括这三种 "射" 的情况。

为了实现这种背面透光的效果，我们可以假定，在主光的反方向还有一个 补光，用这个补光去做一些运算的骚操作即可模拟 背面透光的效果，比如说 沿补光 偏移 反法线 ，等价于，沿主光偏移法线 最后在取反。

```
//两者是等价的
N_Shift= -N*x + (-L)
N_Shift = -(N*x + L)
```

![](<images/1683732879594.png>)

([https://www.alanzucconi.com/2017/08/30/fast-subsurface-scattering-2/](https://www.alanzucconi.com/2017/08/30/fast-subsurface-scattering-2/))

将 N_Shift 当作 N 去进行光照模型的运算，比如带入 Phong 模型中。

```
/模拟透射现象
float _SSSValue =0.6;
float3 N_Shift = -normalize(N*_SSSValue+L);//沿着光线方向上偏移法线，最后在取反
float BackLight = saturate(pow(saturate( dot(N_Shift,V)) ,_PowerValue)*_ScaleValue);
FinalColor = BackLight;
```

![](<images/1683732879671.png>)

将这个背光效果与前面介绍的 Phong 和 Wrap 模型结合一下：

```
//WrapLight
float WrapLight = pow(dot(N,L)*_WrapValue+(1-_WrapValue),2);
//Blin-Phong
float3 R = reflect(-L,N);
float3 H = normalize(V+L);
// float3 R = normalize(  -L + 2* N* dot(N,L) );
float VR = saturate(dot(V,R));
float NH = saturate(dot(N,H));
float NL = saturate(dot(N,L));
float4 Specular = pow(NH,_SpecularPowerValue)*_SpecularScaleValue;
float4 Diffuse = WrapLight;
 //模拟透射现象
 float _SSSValue =0.6;
 float3 N_Shift = -normalize(N*_SSSValue+L);//沿着光线方向上偏移法线，最后在取反
 float BackLight = saturate(pow(saturate( dot(N_Shift,V)) ,_PowerValue)*_ScaleValue);
 FinalColor =Diffuse + Specular + BackLight;
```

![](<images/1683732879752.png>)

**Anisotropic Lighting**

各向异性，Anisotropy，通俗上讲就是在各个方向上所体现出来的性质都不一样。比如在光照下的头发。

![](<images/1683732879857.png>)

可以明显看到一条 "高光带"，而不是 "点状" 高光。**为了模拟这种 带状高光，我们在光照方程中使用发丝切线 T 替代法线 N，进行光照计算。为了使高光可以沿着头发移动，需要沿法线的方向偏移切线。为了使高光有点上下偏移使用了一个拉升的噪音贴图。**

![](<images/1683732879965.png>)

![](<images/1683732880068.png>)

参考:

[http://web.engr.oregonstate.edu/~mjb/cs519/Projects/Papers/HairRendering.pdf](http://web.engr.oregonstate.edu/~mjb/cs519/Projects/Papers/HairRendering.pdf)

先计算高光:

```
//_StretchedNoiseTex 拉升的噪音贴图
float shift = tex2D(_StretchedNoiseTex,i.uv*10).r + _ShiftTangent;
float3 T_Shift = normalize( T+ N*shift);
float3 H = normalize(V+L);
//因为 sin^2+cos^2 =1 所以 sin = sqrt(1-cos^2)
float dotTH = dot(T_Shift,H);
float sinTH = sqrt(1- dotTH*dotTH);

float dirAtten = smoothstep(-1,0,dotTH);
float Specular= dirAtten * pow(sinTH,_AnisotropicPowerValue)*_AnisotropicPowerScale;
FinalColor = Specular;
```

![](<images/1683732880201.png>)

用真实的物理公式去模拟这种各项异性是非常复杂的，所以都只是视觉上近似模拟。

产生带状的高光是因为使用了 sin 函数，而 sin 函数里的变量范围是 [0,1]，所以只会产生一条高光带

可以看一下 sin 函数的效果：

```
//设置sin函数的值大于 PI，可以看到多条高光带
FinalColor =sin(length(uv-float2(0.5,0.5)) * 10 *3.1415)
```

![](<images/1683732880359.png>)

既然 sin 函数可以，那么 cos 函数也一定行，因为它们只是相位差了一个 PI/2:

```
float shift = tex2D(_StretchedNoiseTex,i.uv*4).r + _ShiftTangent;
float3 T_Shift = normalize( T+ N*shift);
float3 H = normalize(V+L);

float dotTH = dot(T_Shift,H);
float cosTH = cos(dot(T_Shift,H));

float dirAtten = smoothstep(-1,0,dotTH);
float Specular = dirAtten * pow(cosTH,_AnisotropicPowerValue)*_AnisotropicPowerScale;
```

使用 cos 函数，效果一样，且比使用 sin 更省性能 (因为要开方)

![](<images/1683732880776.png>)

如果想要多条高光带，就像这位飘逸的姐姐

![](<images/1683732880913.png>)

再加几个高光带, 可以直接这么写:

```
//高光带数量
float NumberOfStrip =6;
float cosTh =cos(dot(T_Shift,H)*NumberOfStrip *3.141592654);
```

在调这个效果的时候，_AnisotropicPowerValue _AnisotropicPowerScale 以及 _StretchedNoise 的 uv 也要一起调，

![](<images/1683732880959.png>)

Diffuse 用前面的 Wraplight

```
float shift = tex2D(_StretchedNoiseTex,i.uv*4).r + _ShiftTangent;
                
// shift += sin(uv.y*5*3.14);

float3 T_Shift = normalize( T+ N*shift);
float3 H = normalize(V+L);

float dotTH = dot(T_Shift,H);
// float sinTH = sqrt(1- dotTH*dotTH);

// sinTH = sin( acos(dot(T_Shift,H)));
float NumberOfStrip =1;
float cosTH =cos(dot(T_Shift,H)*NumberOfStrip );
// float cosTh =cos(dot(T_Shift,H));
// sinTH = cosTh;

float dirAtten = smoothstep(-1,0,dotTH);
float Specular = dirAtten * pow(cosTH,_AnisotropicPowerValue)*_AnisotropicPowerScale;

float WrapLight = dot(N,L)*0.5+0.5;
float Diffuse = WrapLight;
FinalColor = Diffuse*float4(0.7,0.2,0.4,0) +Specular;

return FinalColor;
```

高光带有断裂是因为我的_StretchedNoiseTex 贴图不连续，整个连续的就行,

调个猛男色，:D

![](<images/1683732881102.png>)

## PBR

**理论**

PBR(Physically Based Shading) 是基于与现实世界的物理原理所创建的一种渲染技术，相比于传统的基于经验的模型（Phong Blin-Phong 等）更具有物理准确性。基于物理的光照模型必须满足一下三个条件:

**1. 基于微平面 (Microfacet) 的表面模型。**

**2. 能量守恒。**

**3. 基于物理的 BRDF。**

**微表面模型**

微观尺度上，物体表面有无数的为表面构成。用粗糙度去衡量一个表面的粗糙程度, 越粗糙则说明微表面的法线方向越不一致，反射的光越分散，反之则越集中。

![](<images/1683732881208.png>)

较高的粗糙度值显示出来的镜面反射的轮廓要更大一些。与之相反地，较小的粗糙值显示出的镜面反射轮廓则更小更锐利。

**能量守恒**

能量守恒定律是自然界普遍的基本定律之一。一般表述为：能量既不会凭空产生，也不会凭空消失，它只会从一种形式转化为另一种形式，或者从一个物体转移到其它物体，而能量的总量保持不变。

将能量守恒定律应用到光照模型中: 出射光线的能量永远不能超过入射光线的能量。可以用渲染方程来描述:

![](<images/1683732881380.png>)

在 PBR 中考虑简化的形式 即反射方程:

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

**直接光 漫反射 + 镜面反射**

由于直接光通常是 平行光 或者 点光源 都是有限的数量，因此直接将它们的光照结果相加 即是最终的积分结构。我们目前只考虑一个平行光，因此直接光的计算结构 即使积分结果。而间接光从四面八方射到物体表面上来，数量是无限的，因此要用积分计算出所有间接光的计算结果。

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

如前面的示意图所示 Smith 认为 G 项分应该分为为两部分: 第一部分 是 Geometry Obstruction 第二部分是 Geometry Shadowing 即:

![](<images/1683732884517.png>)

```
//G
float GeometrySchlickGGX(float NV,float Roughness) {
    float r = Roughness +1.0;
    float k = r*r / 8.0;      //直接光
    float nominator = NV;
    float denominator = k + (1.0-k) * NV;
    return nominator/ max(denominator,0.001) ;//防止分母为0
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

**间接光 漫反射 + 镜面反射**

间接光是用 IBL(Image Based Lighting) 来模拟的。与直接光相比，IBL 把周围环境整体视为一个大光源，IBL 通常使用（取自现实世界或从 3D 场景生成的）环境立方体贴图 (Cubemap) ，我们可以将立方体贴图的每个像素视为光源，在渲染方程中直接使用它。这种方式可以有效地捕捉环境的全局光照和氛围，使物体**更好地融入**其环境。

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

实时进行积分计算太消耗，可以先离线计算出来。然而，理论上在半球上采样方向是无限的 , 因此不可能从 半球上 的所有方向采样环境光照。不过我们可以对有限数量的方向采样以近似求解，在半球内均匀间隔或随机取方向可以获得一个近似精确的辐照度值，从而离散地计算积分结果。

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

**LightProbe 与 球协函数**

辐照度贴图是固定的，如果游戏中有多个场景，比如室内室外 那么肯定是不能为每一个场景都单独弄一个辐照度贴图的，会消耗大量内存。游戏引擎中使用 LightProbe 放置在场景中，去近似采样并存储以该 LightProbe 为中心的 "辐照度贴图"，但不是单独存一张贴图，而是用球协函数存储相应的信息，最终存储的是球协函数的参数，通常是 9 个 float，这比单独存一张贴图节省多了。

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

和球协的有点区别，但是明显这张图比球协的更准确地反映了 环境光 的 "黄"

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
//UNITY_SPECCUBE_LOD_STEPS 在  "UnityStandardConfig.cginc"中 
// #define UNITY_SPECCUBE_LOD_STEPS 6
float mip = Roughness*(1.7 - 0.7*Roughness) * UNITY_SPECCUBE_LOD_STEPS ; 
float4 rgb_mip = UNITY_SAMPLE_TEXCUBE_LOD(unity_SpecCube0,R,mip);
//间接光镜面反射采样的预过滤环境贴图
//unity_SpecCube0_HDR储存的是 最近的ReflectionProbe
float3 EnvSpecularPrefilted = DecodeHDR(rgb_mip, unity_SpecCube0_HDR);
FinalColor.rgb = EnvSpecularPrefilted.rgb;
```

仅显示 ReflectionProbe 信息，即 间接光镜面积分第一部分 结果

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

计算这个积分有两种办法，第一种是 BRDF 积分图 第二种是 函数拟合

**BrdfLut**

假设每个方向的入射光都是白色的（L(p,x)=1.0 ），就可以在给定粗糙度、光线 ωi 法线 n 夹角 n⋅ωi 的情况下，预计算 BRDF 的响应结果。以 X 轴为法线与入射光的夹角 (dot(N,L))，以 Y 轴为粗糙度，将计算的结果存储在一张 2D 贴图上 (Lut)，该贴图称为 BRDF 积分贴图。积分的结果分别储存在 贴图的 RG 通道中。使用的时候可以直接采样该贴图即可。

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

用函数拟合的效果比 BrdfLut 的 效果更好，且更节省性能。

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

与 Unity 默认的对比 BaseColor=(255,199,88) Metallic=0.95 Roughness=0.21 参数一致 用屏幕后处理统一做 ToneMapping

![](<images/1683732887029.png>)

可以看出 Unity 的效果偏亮一点，因为 Unity 在实现 PBR 算法的时候 直接光漫反射 与 间接光漫反射 都没有除以 PI(为了与老版本效果兼容)，而我们的算法是标准的 PBR 算法，在表现金属上会有更细腻的亚光效果。

完整的 PBR 效果:(BaseColor + Metallic +Roughness + Normal + AO)

![](<images/1683732887165.png>)

完整的 PBR 代码链接:

[https://github.com/ipud2/Unity-Basic-Shader/blob/master/PBR_BetterThanUnity.shader](https://github.com/ipud2/Unity-Basic-Shader/blob/master/PBR_BetterThanUnity.shader)

  
**HDR 与 Gamma**

**HDR**

在 PBR 中做光照运算时，最终输出的颜色值时常超过 1，而超过 1 的部分在显示器中显示就会 “泛白过爆”，为了解决这个问题，将 HDR(High Dynamic Range) 的颜色值转换到 LDR(Low Dynamic Range)

的算法叫做 ToneMapping(色调映射)。在各种 ToneMapping 算法中 ACESTonemapping 效果与性能兼优。

```
float3 ACESToneMapping(float3 x) {
    float a = 2.51f;
    float b = 0.03f;
    float c = 2.43f;
    float d = 0.59f;
    float e = 0.14f;
    return saturate((x*(a*x+b))/(x*(c*x+d)+e));
}
```

**Gamma**

1.  人眼对 低亮度 的颜色变化 感知强，对 高亮度 的颜色变化 感知弱
2.  为了提高图片的显示辨识度，将自然界线性的颜色储存在非线性的空间 (Gamma)
3.  在渲染中先将图片转化为 线性空间 (Linear)，再进行颜色加减乘除操作才是正确的，最后再转为非线性(Gamma) 的颜色输出

```
//Gamma => Linear
float3 Color_Linear = pow(Color_Gamma,2.2);
//Linear => Gamma
float3 Color_Gamma= pow(Color_Linear,1/2.2);
```

在 Unity 中如果**设置颜色空间为 Gamma**，那么在进行 PBR 计算之前需要用代码**手动转到 Linear**, 并且最终输出到显示器的颜色时候 需用代码**手动转到 Gamma 空间**

在 Unity 中如果**设置颜色空间为 Linear**，那么 Unity 会帮我们把图片**自动转到 Linear 空间**，并且最终输出到显示器的颜色时候 **自动转到 Gamma 空间**

**总结**

![](<images/1683732887297.png>)

## PBR Anistropy 基于物理的各项异性高光

**TODO 原理：**

[Physically Based Rendering in Filament](https://google.github.io/filament/Filament.html#materialsystem/anisotropicmodel/anisotropicspecularbrdf)

**结果**

![](<images/1683732887357.png>)

**代码:**

[ipud2/Unity-Basic-Shader](https://github.com/ipud2/Unity-Basic-Shader/blob/master/%E5%9F%BA%E4%BA%8E%E7%89%A9%E7%90%86%E7%9A%84%E9%AB%98%E5%85%89Brdf/PBRAnistropicSpecular.shader)

## **ClearCoat 清漆**

传统的 PBR 可以很好地模拟出各项同性的单层表面的光照模型，但是对于多层材质就很难模拟出。比如常见的清漆效果 (ClearCoat)。

![](<images/1683732887501.png>)

以下图片是一个对比:

![](<images/1683732887582.png>)

为了模拟这种清漆效果，可以再原 PBR 的基础上加一个二级高光，但要保证能量守恒，那么能量守恒公式可以这么描述:

![](<images/1683732887926.png>)

对于清漆材质，物体的表面分为两部分，上面的一部分是 ClearCoatLayer 下面的部分是 BaseLayer，一束光入射到表面，一部分光直接反射出去 (ClearCoat 高光，且忽略这部分光的漫反射), 另一部分进入 BaseLayer，再分为漫反射与 GGX 高光反射出去。

![](<images/1683732888003.png>)

那么现在要解决的问题就是: 有多少光进入了 BaseLayer？ 为了解决这个问题，引入了两个参数，ClearCoatRoughness 用来描述清漆表面的粗糙度以及 ClearCoatIntensity 用来描述清漆反射的强度。清漆表面的菲尼尔反射部分属于 ClearCoatLayer 的能量，剩下的进入 BaseLayer。用公式描述:

![](<images/1683732888106.png>)

ClearCoat 高光用 GGX 来描述，因为 GGX 中有 Fresnel 部分就不用额外在额外乘以菲尼尔项了。公式可以简化为:

![](<images/1683732888268.png>)

对于直接光部分的代码

```
//Diffuse为直接光漫反射,Specular为直接光高光
float3 DirectLight = (Diffuse + Specular)*NL *Radiance;
float F_ClearCoat = F_FrenelSchlick(HV,0.04)*_ClearCoat;
float3 Specular_ClearCoat = Specular_GGX(N,L,H,V,NV,NL,HV,_RoughnessClearCoat,0.04)*_ClearCoat;
//保证能量守恒
DirectLight = DirectLight * (1-F_ClearCoat) + Specular_ClearCoat;
```

对于间接光部分的代码

```
//Diffuse_Indirect为间接光漫反射,Specular_Indirect为间接光高光
IndirectLight = (Diffuse_Indirect + Specular_Indirect);
float3 Specular_Indirect_ClearCoat = SpecularIndirect(N,V,_RoughnessClearCoat,0.04)*_ClearCoat;
float3 F_IndirectLight_ClearCoat = FresnelSchlickRoughness(NV,0.04,_RoughnessClearCoat)*_ClearCoat;
//保证能量守恒
IndirectLight = IndirectLight*(1-F_IndirectLight_ClearCoat) +Specular_Indirect_ClearCoat;
```

最后结果:

![](<images/1683732888347.png>)

完整代码连接:

[https://github.com/ipud2/Unity-Basic-Shader/blob/master/%E5%85%89%E7%85%A7%E6%A8%A1%E5%9E%8B/ClearCoat.shader](https://github.com/ipud2/Unity-Basic-Shader/blob/master/%E5%85%89%E7%85%A7%E6%A8%A1%E5%9E%8B/ClearCoat.shader)

## Cloth 布料

**TODO 原理:**

**结果:**

![](<images/1683732888408.png>)

完整代码连接:

[https://github.com/ipud2/Unity-Basic-Shader/blob/master/%E5%85%89%E7%85%A7%E6%A8%A1%E5%9E%8B/Cloth.shader](https://github.com/ipud2/Unity-Basic-Shader/blob/master/%E5%85%89%E7%85%A7%E6%A8%A1%E5%9E%8B/Cloth.shader)

## 总结

![](<images/1683732888463.png>)

TODO: NPR Skin Hair Eye

**最后**

可以加一下本人 wx：JoeyTA101 （备注：知乎）

拉你进微信 TA 学习交流群，群里有 100 多份 TA 学习资料哦