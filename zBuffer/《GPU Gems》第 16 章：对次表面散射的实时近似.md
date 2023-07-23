本文是对《GPU Gems》[Chapter 16. Real-Time Approximations to Subsurface Scattering](https://developer.nvidia.com/gpugems/gpugems/part-iii-materials/chapter-16-real-time-approximations-subsurface-scattering) 的翻译和总结，其间可能补充了一些自己的见解和其他来源的资料（[使用 Unity Shader 实现，模型太大就不上传了大家自己找吧](https://gitee.com/luoxiaoc/tutorial-source-code/raw/master/GPU%20Gems%20Chapter%2016.unitypackage)），欢迎大家阅读并指出不足和错误。

## 概述

今天，实时渲染中的大多数着色模型只考虑光在物体表面的相互作用。但是，在真实的世界中许多物体是有点半透明的 (或者说透光吧)：光射进表面，在材质里散射，然后从与射入点不同的地方射出表面。对有效而正确的次表面光传输的模型已经有了很多研究。虽然在目前的图形硬件上还达不到完全正确的物理模拟，但有可能得到实时的近似视觉效果。这章介绍几种方法，用可编程的图形硬件，近似表皮和大理石等半透明材料。

## 16.1 次表面散射的视觉效果

要再生任何视觉效果，经常的做法是考查这种效果的图像，并把可视外表分解为它的组成要素。

![[2ac665c2eac7b2c7634d0f5b1ec307d5_MD5.png]]

**在观察透光物体的相片和图像时能够注意到 (后面的方案都是来实现下面内容)：**

*   首先，次表面散射往往使照明的整体效果变得柔和。
*   一个区域的光线往往渗透到表面的周围区域，而小的表面细节变得看不清了。
*   光线穿入物体越深，就衰减和散射得越严重。
*   对于皮肤来说，在照亮区到阴影区的衔接处，散射也往往引起微弱的倾向红色的颜色偏移。这是由于光线照亮表皮并进入皮肤，接着被皮下血管和组织散射和吸收，然后从阴影部分离开。散射效果在皮肤薄的部位更加明显，比如鼻孔和耳朵周围。

## 16.2 简单的散射近似

近似散射的简单技巧是环绕光照 (wrap lighting)。正常情况下，当表面的法线对于光源方向垂直的时候，漫反射函数 (Lambert）提供的照明度是 0。环绕光照修改 Lambert，使得光照环绕在物体的周围，越过那些正常时会变黑暗的点。这减少漫反射光照明的对比度，从而减少了环境光和所要求的填充光的量。环绕光照是对 Oren-Nayar 照明模型的一个粗糙的近似，那个模型力图更精确地模拟粗糙的不光滑表面（[Nayar and Oren 1995](https://en.wikipedia.org/wiki/Oren%E2%80%93Nayar_reflectance_model))。

显示在下面的代码和图 16-1，说明了如何改变漫反射光照函数，使它包含环绕效果。环绕值是范围在 0 和 1 之间的浮点数，范围值控制光照环绕物体周围的距离：

```
float diffuse = max(0, dot(L, N));
float wrap_diffuse = max(0, (dot(L, N) + wrap) / (1 + wrap));
```

![[456e2b4f207552e095aa76ddcaf1415f_MD5.png]]

在照明度接近 0 时，可以表达那种倾向红色的微小颜色漂移，这是模拟皮肤散射的一种廉价方法。下面代码示范了如何使用这种技术，见图 16-2 的例子（为了表达效果我把值调的比较夸张）。

```
half4 frag (v2f i) : SV_Target
{
    half3 N = normalize(i.worldNormal);
    half3 L = normalize(UnityWorldSpaceLightDir(i.worldPos));
    half3 V = normalize(UnityWorldSpaceViewDir(i.worldPos));
    half3 H = normalize(L + V);

    half NdotL = saturate(dot(N, L));
    half NdotH = saturate(dot(N, H));

    half diffuse = NdotL;
    half specular = pow(NdotH, _Shininess);
    half scatter = 0;
#ifdef _SUBSURFACESCATTERING
    half NdotL_wrap = (NdotL + _Wrap) / (1 + _Wrap);
    diffuse = max(0, NdotL_wrap);
    half range1 = smoothstep(0, _ScatterWidth, NdotL_wrap);
    half range2 = smoothstep(_ScatterWidth * 2, _ScatterWidth, NdotL_wrap);
    scatter = range1 * range2;
#endif

    half4 col = tex2D(_MainTex, i.uv);
    half4 lit = diffuse * _LightColor0 * col + specular * _SpecularColor + scatter * _ScatterColor;

    return lit;
}
```

![[b54c0f47f70ee693b7c033b2546c4a77_MD5.png]]

我还找到了一种更平滑的实现：[Fast Subsurface Scattering for the Unity URP — John Austin](https://johnaustin.io/articles/2020/fast-subsurface-scattering-for-the-unity-urp)，其中 Wrap 的方式：[http://www.cim.mcgill.ca/~derek/files/jgt_wrap.pdf](http://www.cim.mcgill.ca/~derek/files/jgt_wrap.pdf)，为了让效果更好，我使用了表面着色器：

```
inline half4 LightingStandardTranslucent(SurfaceOutputStandard s, half3 viewDir, UnityGI gi) {
    // PBR
    half4 pbr = LightingStandard(s, viewDir, gi);

    // Subsurface Scattering
#ifdef _SUBSURFACESCATTERING
    float3 L = gi.light.dir;
    float3 N = s.Normal;
    half NdotL = dot(N, L);
    half alpha = _SSS_Radius;
    half theta_m = acos(-alpha); // boundary of the lighting function
    half theta = max(0, NdotL + alpha) - alpha;
    half normalization_jgt = (2 + alpha) / (2 * (1 + alpha));
    half wrapped_jgt = (pow(((theta + alpha) / (1 + alpha)), 1 + alpha)) * normalization_jgt;
    //half wrapped_valve = 0.25 * (NdotL + 1) * (NdotL + 1);
    //half wrapped_simple = (NdotL + alpha) / (1 + alpha);
    half3 subsurface_radiance = _SSS_Color * wrapped_jgt * pow((1 - NdotL), 3);
    pbr.rgb = pbr.rgb * (1 - _SSS_Value * _SSS_Value) + gi.light.color * subsurface_radiance * _SSS_Value;
#endif

    return pbr;
}
```

![[2efb3427214a022355608cc5499d0ba5_MD5.png]]

## 16.3 使用深度映射模拟吸收

吸收是模拟半透明材质的最重要特性之一。光线在物质中传播得越远，它被散射和吸收得就越厉害。为了模拟这种效果，我们需要测量光在物质中传播的距离。而估算传播距离可以使用深度映射技术，此技术非常类似于阴影映射，而且可用于实时渲染：如下图 16-4 ，在深度遍中，我们以光源为摄像机渲染场景，存储从光源到当前片元的距离。在渲染遍中，把当前片元坐标转换到光源空间，采样深度贴图得到当前片元对应入射点到光源的距离 (di)，再计算出当前片元(出射点) 到光源的距离(do)，后者减去前者，即光线在物体中传播的距离(s)。

![[5cee1887924e433081eba659ab805c1f_MD5.png]]

*   这个技术的问题是它只对凸的物体有效，不能正确描绘物体上的洞（问题不大）。
*   对于静态的物体，可以绘制或预计算一个映射，以表现表面每个点的大约厚度。
*   使用深度映射的好处在于考虑到了光的入射方向，并且可用于动画模型（实时）。

### 16.3.1 实现细节

原文的参考代码：

```
// 深度Pass：渲染从光源到片元的距离。假定通过应用程序为光源视点建立了modelView和modelViewProj矩阵。
struct a2v 
{
   float4 pos    : POSITION;
   float3 normal : NORMAL; 
}; 
struct v2f 
{
   float4 hpos : POSITION;
   float  dist : TEXCOORD0; // distance from light 
}; 
// 顶点着色器
v2f main(a2v IN, uniform float4x4 modelViewProj, uniform float4x4 modelView, uniform float grow) 
{
   v2f OUT;
   float4 P = IN.pos;
   P.xyz += IN.normal * grow;  // 稍稍缩放以避免类似阴影映射中出现的自阴影走样现象
   OUT.hpos = mul(modelViewProj, P);
   OUT.dist = length(mul(modelView, IN.pos));
   return OUT; 
} 
// 片元着色器
float4 main(float dist : TEX0) : COLOR 
{
   return dist;  // return distance 
} 

// 查询光线到片元的距离以计算深度
float trace(float3 P, // 世界坐标
             uniform float4x4  lightTexMatrix, // 到光源屏幕空间的矩阵
             uniform float4x4  lightMatrix,    // 到光源空间的矩阵
             uniform sampler2D lightDepthTex,) 
{   
    // 转换到光源屏幕空间
    float4 texCoord = mul(lightTexMatrix, float4(P, 1.0));
    // 得到入射点到光源距离
    float d_i = tex2Dproj(lightDepthTex, texCoord.xyw);
    // 转换到光源空间
    float4 Plight = mul(lightMatrix, float4(P, 1.0));
    // 出射点到光源距离
    float d_o = length(Plight);
    // 光在物体内部传播距离
    float s = d_o - d_i;   
    return s; 
}
```

可以说上面的代码很简单明了了，不过我并没有在 Unity 中实现，· 因为我觉得现在的做法和阴影映射完全一样，何必再浪费一轮渲染呢，所以我们直接用 Command Buffer 抓取阴影贴图即可：

```
RenderTargetIdentifier shadowmap = BuiltinRenderTextureType.CurrentActive;
cb.SetShadowSamplingMode(shadowmap, ShadowSamplingMode.RawDepth);
cb.Blit(shadowmap, new RenderTargetIdentifier(m_ShadowmapCopy));
cb.SetGlobalTexture("_CopyShadowMap", m_ShadowmapCopy);
lightC.AddCommandBuffer(LightEvent.AfterShadowMap, cb);
```

我这里用的是聚光灯（平行光和点光源要麻烦一些），所以采样方法为：

```
float GetThicknessForSpot(float4 positionWS) {
    float4 shadowCoord = mul(unity_WorldToShadow[0], positionWS);
    float In = tex2D(_CopyShadowMap, shadowCoord.xy / shadowCoord.w);//入射点
    float Out = shadowCoord.z / shadowCoord.w;//出射点
    return In - Out;
}
```

直接返回厚度（或者说是光在材质内传播距离）的结果：可以看到深浅的变化

![[6bfe7a13e4dbe212d288efee7f67d902_MD5.png]]

得到了光在材质内传播距离怎么用：用它索引 RampTex(颜色应该随着距离呈指数衰减) 或者自己写一个指数函数 y=exp(-depth......)......，并与其他光照明模型相结合，就可以丰富材质类型。

为了更准确地模拟，需要知道光在物体上入射点的法线及潜在的表面颜色。可以通过在另外的遍渲染额外的信息到其他纹理的方法来办到这点，然后以类似于深度纹理的方法查询这些纹理。在支持多重渲染目标的系统上，可以将深度、法线和其他的遍整合进一个多值输出遍。

![[acb04d4a45d99f5b74ddd95baea592c4_MD5.png]]

### 16.3.2 更精密的散射模型

更精密的模型试图精确地模拟在介质里散射的累积效应：一种模型是单次散射的近似，它假设光在材质中只反弹一次，沿着材质内的折射光线，可以计算有多少光子会朝向摄像机散射，当光击中一个粒子的时候，光散射方向的分布用相位函数来描述，入射点和出射点的 Fresnel 效应也很重要；另一个模型，近似漫反射，模拟高散射介质 (例如皮肤) 的多次散射效果。但是这些技术超出了本章范围。SIGGRAPH 2003 RenderMan 课程 (Hery 2003) 的 Christophe Hery 章节, 讨论了皮肤 shader 的单次和漫反射散射的细节。

## 16.4 纹理空间的漫反散

次表面散射最明显的视觉特征之一是模糊的光照效果。其实，3D 美术时常在屏幕空间中效仿这个现象，通过在 Photoshop 中执行 Gaussian 模糊，然后把模糊图像少量地覆盖在原始图像上，这种 Bloom 技术使光照变得柔和。我们可以在纹理空间中计算漫反射来实现实时的模糊：我们可以用顶点程序展开物体的网格（使用纹理坐标 UV 作为顶点的屏幕位置，把 [0，1] 范围的纹理坐标重映射为 [-1，1] 的规范化的坐标，当然前提是物体必须有好的 UV 映射：在纹理上的每个点必须映射为物体上的惟一点，不能重叠），通过在光源空间对这个展开的网格进行光照计算（可以想想是光在照着 UV 展开图），我们就获得了代表物体光照明的 2D 图像。然后可以用 Bloom 的方式处理这个图像，并且再度把它像正常的纹理一样用于原本 3D 模型。

这个技术可以用于其他应用，因为它能减轻屏幕分辨率造成的着色复杂性：现在的着色完全在 UV 展开图的每个像素上执行，而不是在物体上的每个像素上执行。比如卷积运算，在图像空间比在 3D 表面上执行起来更有效。因为在世界空间中靠近的点，映射到纹理空间中也是靠近的点，所以如果表面的 UV 参数化越均匀，近似效果就越好。

展开物体的网格并在光源空间计算漫反射：上一个 Part 我们用的是聚光灯，现在我们用平行光（都展示一遍让大家看看怎么自己采样），光源矩阵还是利用阴影的（当然可以自己构建）：

![[e223658843dc34d35a933f9a2cb3f6e5_MD5.png]]

```
float4 GetCascadeWeights_SplitSpheres(float3 wpos)
{
    //unity_ShadowSplitSpheres:用于构建层叠式阴影贴图时子视锥体用到的包围球(x、y、z、w分量存储包围球的球心坐标和半径)
    float3 fromCenter0 = wpos.xyz - unity_ShadowSplitSpheres[0].xyz;
    float3 fromCenter1 = wpos.xyz - unity_ShadowSplitSpheres[1].xyz;
    float3 fromCenter2 = wpos.xyz - unity_ShadowSplitSpheres[2].xyz;
    float3 fromCenter3 = wpos.xyz - unity_ShadowSplitSpheres[3].xyz;
    float4 distances2 = float4(
        dot(fromCenter0,fromCenter0), 
        dot(fromCenter1,fromCenter1), 
        dot(fromCenter2,fromCenter2), 
        dot(fromCenter3,fromCenter3));
    //unity_ShadowSplitSpheres中4个包围球半径的平方
    float4 weights = float4(distances2 < unity_ShadowSplitSqRadii); 
    weights.yzw = saturate(weights.yzw - weights.xyz);
    return weights;
}

// Returns the shadow-space coordinate for the given world-space position.
float3 GetLightSpaceDir(float3 dirWS, float4 cascadeWeights)
{
    float3 sc0 = mul((float3x3)unity_WorldToShadow[0], dirWS);
    float3 sc1 = mul((float3x3)unity_WorldToShadow[1], dirWS);
    float3 sc2 = mul((float3x3)unity_WorldToShadow[2], dirWS);
    float3 sc3 = mul((float3x3)unity_WorldToShadow[3], dirWS);
    float3 dirLS = float3(
        sc0 * cascadeWeights[0] + 
        sc1 * cascadeWeights[1] + 
        sc2 * cascadeWeights[2] + 
        sc3 * cascadeWeights[3]);
#if defined(UNITY_REVERSED_Z)
    float  noCascadeWeights = 1 - dot(cascadeWeights, float4(1, 1, 1, 1));
    dirLS.z += noCascadeWeights;
#endif
    return normalize(dirLS);
}

v2f vert (appdata v)
{
    v2f o;
    float4 uv = float4(0, 0, 0, 1);
    uv.xy = float2(1, _ProjectionParams.x) * (v.uv.xy * 2 - 1);
    o.vertex = uv; 
    o.uv = v.uv;
    o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
    o.worldNormal = UnityObjectToWorldNormal(v.normal);
    return o;
}

half4 frag (v2f i) : SV_Target
{  
    float4 cascadeWeights = GetCascadeWeights_SplitSpheres(i.worldPos);
    half3 N = GetLightSpaceDir(i.worldNormal, cascadeWeights);
    half3 L = GetLightSpaceDir(UnityWorldSpaceLightDir(i.worldPos), cascadeWeights);
    return half4(saturate(dot(N, L)) * _LightColor0.rgb, 1);
}
```

为了模拟图像空间的漫反射，可以简单地模糊光照映射纹理。我们可以利用一切通常的 GPU 图像处理技巧，例如使用可分离的滤波器，以及双线性过滤硬件，或者对相对低分辨率的纹理渲染照明就已经提供了某种程度的模糊。下图 16-7 显示了展开的头部网格，以及模糊光照映射纹理的结果。

![[d32bad603647122d535d1f6ff4d6a813_MD5.png]]

值得注意的是：

*   如果我们使用带色彩的纹理包含在光照映射纹理中，色彩映射的细节也会被淡化；
*   如果阴影包含在光照映射纹理中，将造成模糊的阴影。

为了模拟皮肤的吸收和散射与波长相关的事实，我们对每个彩色通道分别地改变滤波权重。在下面代码中显示了 shader 样本，它是有 Gaussian 权重的 7 个纹理样本。红色通道的滤波器宽度比绿色和蓝色通道的更宽，所以红色比其他通道漫反射更强。

```
struct appdata
{
    float4 vertex : POSITION;
    float2 uv : TEXCOORD0;
};

struct v2f
{
    float4 vertex : SV_POSITION;
    float2 uv0 : TEXCOORD0;
    float2 uv1 : TEXCOORD1;
    float2 uv2 : TEXCOORD2;
    float2 uv3 : TEXCOORD3;
    float2 uv4 : TEXCOORD4;
    float2 uv5 : TEXCOORD5;
    float2 uv6 : TEXCOORD6;
};

sampler2D _MainTex;
float4 _MainTex_TexelSize;

v2f vert (appdata v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);
    float2 unit = float2(_MainTex_TexelSize.x, _MainTex_TexelSize.y);
    o.uv0 = v.uv + 2 * float2(unit.x * -5.5, unit.y * 0);
    o.uv1 = v.uv + 2 * float2(unit.x * -3.5, unit.y * 0);
    o.uv2 = v.uv + 2 * float2(unit.x * -1.5, unit.y * 0);
    o.uv3 = v.uv + 2 * float2(unit.x * 0, unit.y * 0);
    o.uv4 = v.uv + 2 * float2(unit.x * -1.5, unit.y * 0);
    o.uv5 = v.uv + 2 * float2(unit.x * -3.5, unit.y * 0);
    o.uv6 = v.uv + 2 * float2(unit.x * -5.5, unit.y * 0);
    return o;
}

half4 frag (v2f i) : SV_Target
{  
    const float4 weight[7] =
    {
        { 0.006, 0.00, 0.00, 0 },
        { 0.061, 0.00, 0.00, 0 },
        { 0.242, 0.25, 0.25, 0 },
        { 0.383, 0.50, 0.50, 0 },
        { 0.242, 0.25, 0.20, 0 },
        { 0.061, 0.00, 0.00, 0 },
        { 0.006, 0.00, 0.00, 0 },
    };

    half4 a = 0;
    a += tex2D(_MainTex, i.uv0) * weight[0];
    a += tex2D(_MainTex, i.uv1) * weight[1];
    a += tex2D(_MainTex, i.uv2) * weight[2];
    a += tex2D(_MainTex, i.uv3) * weight[3];
    a += tex2D(_MainTex, i.uv4) * weight[4];
    a += tex2D(_MainTex, i.uv5) * weight[5];
    a += tex2D(_MainTex, i.uv6) * weight[6];
    return a;
}
```

为了得到更大的模糊，可以多次重复应用模糊 shader，或写一个 shader，通过在片元程序中计算样本位置取更多样本。图 16-8 显示了把模糊光照映射纹理应用到 3D 头部模型。最后的 shader 用最初的高分辨率彩色映射，混合漫反射的照明纹理，获得最终的效果，如图 16-9 所示。

![[987164fb9a813b1f7a99a76ffc60837a_MD5.png]]

![[033b9430a336883d39bfde12d74d41a6_MD5.png]]

为描绘物体中更密集的物体 (例如身体里的骨头），渲染额外的深度遍是深度映射技术可能的扩展。但是使用基于表面的方法去考虑体效果，会出现问题。体渲染没有这个限制，而且当物体的密度变化时，它能产生更加正确的渲染。关于更多的体渲染，见本书的第 39 章 “体渲染技术”。这个技术另外可能的扩展是提供几个彩色贴图，每个代表皮肤的不同层次的颜色。举例来说，可以给表皮的颜色提供一个贴图，而给皮肤下的血管和毛细管提供另一个贴图。Greg James(2003）介绍了一种技术：处理任意多边形的物体时，首先加上所有背表面的距离，然后减去所有前表面的距离。应用程序计算屏幕空间的距离，用于产生体雾效果，还有可能扩展到更一般的情形。

未来研究的有趣方向，是结合深度贴图和纹理空间技术，以获得两者最好的效果。

## 16.5 小结

要产生使人信服的皮肤和其他半透明材质的图像，次表面散射的效果是十分重要的因素。使用几个不同的近似，我们已经得到实时的次表面散射的效果。随着图形硬件变得更为强大，可能会出现更为精确的次表面光传播模型。我们希望这章中介绍的技术有助于改善实时游戏中人物的逼真度。但是要记住的是，要有好的艺术、好的着色法才能给予帮助！

感慨：这是 2004 年的书啊！！！