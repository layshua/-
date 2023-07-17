工作中，会有很多美术同学提出疑问，高光流和金属流如何选择，他们的异同点到底是什么，遵循 PBR 的美术制作流程又到底是怎样的。在进行 ClearCoatPBR 的剖析前，我们先聊一聊 PBR 的这两个工作流，并且结合上一篇文章的 InitializeBRDFData 函数，根据不同工作流的选择，分析其中的参数赋值。

## 一、PBR 工作流

**1、金属和非金属的光照特性**

首先我们知道一束光打在物体上，一部分直接被反射（高光反射），一部分被吸收，一部分被折射到内部后又从物体表面射出（漫反射，包括次表面反射），一部分折射到内部后从物体另一个表面射出（透射）。一般材质，我们忽略透射，只考虑前三者，即：

总光照 = 漫反射 + 高光反射 + 吸收

自然界的物质根据光照特性大体可以分为金属和非金属。

金属的光照特性是：漫反射率基本为 0，所以漫反射颜色也为 0（黑色），所以总光照 = 高光反射 + 吸收，那么高光反射到底占总光照的多少呢，我们使用 reflctivity（高光反射率）* 总光照来获得，reflctivity 在 [70%，100%]。而高光颜色总是偏金属本身的颜色，例如黄金的高光颜色是金黄色，白银的高光颜色是灰色，黄铜的高光颜色是黄色。

**金属：漫反射率 = 0，漫反射颜色 = 黑，高光反射率 = reflctivity，高光颜色 = 自身颜色**

非金属的光照特性是：高光反射率在 4% 左右（高光颜色几乎为黑色 0），而漫反射很强，漫反射颜色 =（1-reflctivity）*albedo，其中 1-reflctivity 等于 “漫反射 + 吸收” 的光照比例，再乘以 diffuse 后就是漫反射颜色。

**非金属：漫反射率 = 1-reflctivity，漫反射颜色 = 自身颜色，高光反射率 = 0.04，高光颜色 = 灰黑**

**2、PBR 工作流分析**

然后来看 PBR，PBR 的核心也是将物体分成了金属和非金属两类，除了漫反射和高光反射外，还增加了 roughness 属性，它可以控制高光的扩散程度，roughness 越大，高光越散。

我们都知道 PBR 流程分为：

“Metal-Roughness”（金属度 - 粗糙度）流程

“Specular-Glossiness”（高光反射 - 光泽度）流程

接下来我们分别说明两个流程里用到的贴图。（除了公用的 AO、Normal、Emission 等）

![[f2cb97c58a08caa135b771b4a8b2066a_MD5.jpg]]

**（1）、“Metal-Roughness”（金属度 - 粗糙度）流程**

**（a）BaseColor(基础色贴图)**

包含了 “金属的高光反射颜色” 和“非金属的漫反射颜色”，相当于将金属和非金属的可见色杂糅在一起了。

**（b）Roughness(粗糙度贴图)**

粗糙度是一张灰度图，取值范围 [0 ,1]，越靠近 1，越粗糙，它会影响高光的扩散程度。

**（c）Metallic(金属度贴图)**

金属度也是一张灰度图，取值范围 [0 ,1]，越靠近 1，金属属性越强，反之越弱。使用 Metallic 换算成 Reflectivity（Metallic 越强，Reflectivity 就越强，这两者可以认为是等价的，算法中 Metallic = Reflectivity）。

**（2）、“Specular-Glossiness”（高光反射 - 光泽度）流程**

**（a）Diffuse(漫反射贴图)**

包含了金属和非金属的漫反射颜色，其中金属的漫反射颜色为 0，所以是黑色的。

**（b）glossiness(光滑度贴图)**

就是粗糙度图的取反。

**（c）Specular(高光反射贴图)**

包含了金属和非金属的高光颜色，其中非金属的高光颜色很弱，所以是灰黑色的。使用 Specular 换算成 Reflectivity（Specular 越强，Reflectivity 就越强，这两者也可以认为是等价的，算法中 Reflectivity 取了 Specular 三通道中的最大值）。

**总结：这两种工作流实质上是将光照信息以不同的方式整合，存储在不同形式的贴图里**。

**在 Metal-Roughness 流程里，最明显的优势就是它的贴图非常省，总共只使用了五个通道。但它的问题在于，为了节约通道，它默认非金属的高光反射率等于 0.04，这从一定程度上限制了我们的设计发挥。**

**在 Specular-Glossiness 流程里，它还能单独调节非金属的高光颜色和反射率，但这导致它不得不使用两张 RGB 图，造成了一定程度上的存储压力。**

**最终将 Metallic 和 Specular 都转换成 Reflctivity，统一了两个工作流。**

**3、InitializeBRDFData 函数分析**

然后我们回顾上篇中的 InitializeBRDFData 函数，看下在不同工作流中的参数初始化。

如果选择高光流：

reflectivity 和 specular 成正比（高光颜色 3 通道中取最大值，保证反射最强烈）。

漫反射颜色使用 1-specular 得到非金属的漫反射率，乘以 albedo（实为 diffuse 颜色），得到非金属的漫反射颜色（金属的 diffuse 为 0，最终相乘还是 0）。

高光颜色由 specular 外部控制。

```
// 如果设置高光流
    #ifdef _SPECULAR_SETUP
        // 反射率
        half reflectivity = ReflectivitySpecular1(specular);
        // 反射率取反
        half oneMinusReflectivity = 1.0 - reflectivity;
        // 漫反射颜色
        half3 brdfDiffuse = albedo * (half3(1.0h, 1.0h, 1.0h) - specular);
        // 高光颜色
        half3 brdfSpecular = specular;// 高光颜色
```

如果选择金属流：

reflectivity 和 metallic 成正比，只不过取值范围在（0.04, 1），所以使用 metallic 在其间插值即可。

漫反射率和 metallic 成反比，所以使用 metallic 在（1， 0.04）间插值，注意取值范围是反过来的，所以使用 oneMinusReflectivity * albedo（baseColor）

高光反射率和 metallic 成正比，所以使用 metallic 代替高光反射率去插值（非金属 specular 和金属 specular 间插值）

```
// 如果设置金属流
    #else
        half oneMinusReflectivity = OneMinusReflectivityMetallic1(metallic);// 1 - 反射率
        half reflectivity = 1.0 - oneMinusReflectivity;// 反射率（金属度和反射率成正比）
        // metallic越弱，Reflectivity越小，oneMinusReflectivity就越大
        // 漫反射颜色越接近albedo本身，反之越暗
        half3 brdfDiffuse = albedo * oneMinusReflectivity;
        // 高光颜色
        // kDieletricSpec = half4(0.04, 0.04, 0.04, 1.0 - 0.04)
        // 非金属的反射系数普遍为0.04，可以理解为高光颜色是0.04
        // 金属的反射颜色为金属自身颜色
        // 所以metallic越强，反射颜色为自身颜色，反之为0.04
        half3 brdfSpecular = lerp(kDieletricSpec.rgb, albedo, metallic);
```

## 二、ClearCoatPBR

ClearCoat（清漆）材质，可以实现基础层 + 清漆层的双层材质效果，比如车漆、打蜡木地板、镀膜材料等等。基础层由通用 PBR 算法实现，清漆层由 ClearCoatPBR 算法实现，最后将两个层混合在一起，就可以实现双层材质的效果。UE4 在很早就内置了清漆材质（至少 3 年了），Unity 直到 2020 版本才有这个功能，需要使用 complexLit 着色器才能开启。接下来我们看下代码内部的实现。

Unity 使用 InitializeBRDFDataClearCoat 函数实现，分 3 步走：

1、ClearCoatBRDF 参数初始化

2、GI 间接光计算

3、直接光计算

**1、ClearCoatBRDF 参数初始化**

ClearCoatBRD 中的粗糙度（perceptualRoughness）需要重新赋值，根据平台不同，算法也不同，下面分移动端和非移动端来说明。

**（1）移动端**

移动端的参数和基础层的 BRDF 的参数算法是一样的。由于清漆是非金属，所以高光反射率为 0.04，高光颜色也是 0.04

```
// 清漆层的属性
    outBRDFData.diffuse             = kDielectricSpec.aaa; // 清漆的漫反射颜色 0.96
    outBRDFData.specular            = kDielectricSpec.rgb;//清漆的高光反射颜色 0.04
    outBRDFData.reflectivity        = kDielectricSpec.r;// 清漆的高光反射率 0.04

    outBRDFData.perceptualRoughness = PerceptualSmoothnessToPerceptualRoughness(clearCoatSmoothness);// 1 - clearCoatSmoothness
    outBRDFData.roughness           = max(PerceptualRoughnessToRoughness(outBRDFData.perceptualRoughness), HALF_MIN_SQRT);// perceptualRoughness^2
    outBRDFData.roughness2          = max(outBRDFData.roughness * outBRDFData.roughness, HALF_MIN);// roughness^2
    outBRDFData.normalizationTerm   = outBRDFData.roughness * 4.0h + 2.0h;
    outBRDFData.roughness2MinusOne  = outBRDFData.roughness2 - 1.0h;
    outBRDFData.grazingTerm         = saturate(clearCoatSmoothness + kDielectricSpec.x);// 掠射颜色
```

**（2）非移动端**

而在非移动端下，重新计算了 perceptualRoughness，具体算法见注释。并重新赋值了其他属性。

```
// 非移动端
#if !defined(SHADER_API_MOBILE)
    // Modify Roughness of base layer using coat IOR
    // CLEAR_COAT_IOR = 1.5
    // CLEAR_COAT_IETA = 1.0 / CLEAR_COAT_IOR  // IETA是eta的倒数
    half ieta                        = lerp(1.0h, CLEAR_COAT_IETA, clearCoatMask);
    half coatRoughnessScale          = Sq(ieta);// ieta^2
    // real RoughnessToVariance(real roughness)
    // {
    //     return 2.0 / Sq(roughness) - 2.0;
    // }
    // sigma = 2.0 / perceptualRoughness^4 - 2.0
    half sigma                       = RoughnessToVariance(PerceptualRoughnessToRoughness(baseBRDFData.perceptualRoughness));

    // perceptualRoughness = sqrt(sqrt(2.0 / sigma * coatRoughnessScale + 2.0));
    baseBRDFData.perceptualRoughness = RoughnessToPerceptualRoughness(VarianceToRoughness(sigma * coatRoughnessScale));

    // 重新计算其他属性
    baseBRDFData.roughness          = max(PerceptualRoughnessToRoughness(baseBRDFData.perceptualRoughness), HALF_MIN_SQRT);
    baseBRDFData.roughness2         = max(baseBRDFData.roughness * baseBRDFData.roughness, HALF_MIN);
    baseBRDFData.normalizationTerm  = baseBRDFData.roughness * 4.0h + 2.0h;
    baseBRDFData.roughness2MinusOne = baseBRDFData.roughness2 - 1.0h;
#endif
```

然后初始化 specular，使用 clearCoatMask 插值计算清漆的最终高光颜色。

```
baseBRDFData.specular = lerp(baseBRDFData.specular, ConvertF0ForClearCoat151(baseBRDFData.specular), clearCoatMask);
```

其中 ConvertF0ForClearCoat15 函数如下，根据不同平台，计算出的清漆高光颜色。具体算法见注释。

```
half3 ConvertF0ForClearCoat151(half3 f0) {
#if defined(SHADER_API_MOBILE)
    //return saturate(f0 * (f0 * 0.526868 + 0.529324) - 0.0482256)
    return ConvertF0ForAirInterfaceToF0ForClearCoat15Fast(f0);
#else
    //return saturate(-0.0256868 + f0 * (0.326846 + (0.978946 - 0.283835 * f0) * f0))
    return ConvertF0ForAirInterfaceToF0ForClearCoat15(f0);
#endif
}
```

**2、GI 间接光计算**

ClearCoat 的 GI 间接光只有 “GI 高光反射”，没有 “GI 漫反射”，漫反射使用基础层的。

GI 高光反射和基础层是一样的，通过采样 CubeMap，并根据其 MipLevel 来控制粗糙度，所以还是使用 GlossyEnvironmentReflection 函数实现。

```
// 清漆层的IBL（GI高光反射）
half3 coatIndirectSpecular = GlossyEnvironmentReflection(reflectVector, brdfDataClearCoat.perceptualRoughness, occlusion);
```

高光衰减和遮罩使用 EnvironmentBRDFClearCoat 函数，算法也和基础层一样。最后将高光颜色 * 衰减，得到最终高光颜色。（函数中第一句代码不用写，写了也没地引用，这串代码是在 EnvironmentBRDFSpecular 函数中计算的，官方也不是十全十美啊，这么明显的代码冗余）。

```
// 清漆层GI漫反射
half3 EnvironmentBRDFClearCoat(BRDFData brdfData, half clearCoatMask, half3 indirectSpecular, half fresnelTerm) {
    float surfaceReduction = 1.0 / (brdfData.roughness2 + 1.0);
    return indirectSpecular * EnvironmentBRDFSpecular(brdfData, fresnelTerm) * clearCoatMask;
}
```

接着计算清漆层的菲涅尔反射。

```
// 清漆层的菲涅尔反射
half coatFresnel = kDielectricSpec.x + kDielectricSpec.a * fresnelTerm;
```

最终混合基础层的 GI

```
// 混合基础层和清漆层的GI
return color * (1.0 - coatFresnel * clearCoatMask) + coatColor;
```

**3、直接光计算**

直接光也只有 “直接光高光反射”，漫反射使用基础层的。

和基础层的高光项算法一致，使用 DirectBRDFSpecular 函数实现。

```
half brdfCoat = kDielectricSpec.r * DirectBRDFSpecular1(brdfDataClearCoat, normalWS, lightDirectionWS, viewDirectionWS);
```

然后计算菲涅尔反射。

```
half NoV = saturate(dot(normalWS, viewDirectionWS));
// 清漆层的F项
half coatFresnel = kDielectricSpec.x + kDielectricSpec.a * Pow4(1.0 - NoV);
```

最后混合基础层的高光。

```
// 混合清漆层和基础层的高光
brdf = brdf * (1.0 - clearCoatMask * coatFresnel) + brdfCoat * clearCoatMask;
```

至此，2020 版本的 PBR 代码剖析完毕。欢迎大家留言区讨论并指正错误。