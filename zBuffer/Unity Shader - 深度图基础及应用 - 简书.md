# 相交高亮

思路是判断当前物体的深度值与深度图中对应的深度值是否在一定范围内，如果是则判定为相交。  
首先访问当前物体的深度值：

```c
//vertex
COMPUTE_EYEDEPTH(o.eyeZ);
```

然后访问深度图。由于此时不是 Post Process，因此需要利用投影纹理采样来访问深度图：

```c
//vertex
o.screenPos = ComputeScreenPos(o.vertex);
//fragment
float screenZ = LinearEyeDepth(SAMPLE_DEPTH_TEXTURE_PROJ(_CameraDepthTexture, UNITY_PROJ_COORD(i.screenPos)));
```

最后就是进行相交判断：

```c
float halfWidth = _IntersectionWidth / 2;
float diff = saturate(abs(i.eyeZ - screenZ) / halfWidth); //除以halfWidth来控制相交宽度为_IntersectionWidth

fixed4 finalColor = lerp(_IntersectionColor, col, diff);
return finalColor;
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FIntersectionHighlight.shader)

![[345bdadf2be3da62d8a6d331495c14c0_MD5.webp]]

IntersectionHighlight 场景

# 能量场

在相交高亮效果的基础上，加上**半透明**和**边缘高亮**，就能制造出一个简单的能量场效果：

```
float3 worldNormal = normalize(i.worldNormal);
float3 worldViewDir = normalize(i.worldViewDir);
float rim = 1 - saturate(dot(worldNormal, worldViewDir)) * _RimPower;

float screenZ = LinearEyeDepth(SAMPLE_DEPTH_TEXTURE_PROJ(_CameraDepthTexture, UNITY_PROJ_COORD(i.screenPos)));  
float intersect = (1 - (screenZ - i.eyeZ)) * _IntersectionPower;
float v = max (rim, intersect);

return _MainColor * v;
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FForceField.shader)

![[c9caaa1b8ea54e3b1463c0bfe39b2df8_MD5.webp]]

ForceField 场景

# 全局雾效

思路是让雾的浓度随着深度值的增大而增大，然后进行的原图颜色和雾颜色的插值：

```
fixed4 frag (v2f i) : SV_Target
{
    fixed4 col = tex2D(_MainTex, i.uv.xy);
    float depth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv.zw));
    float linearDepth = Linear01Depth(depth);
    float fogDensity = saturate(linearDepth * _FogDensity);
    fixed4 finalColor = lerp(col, _FogColor, fogDensity);
    return finalColor;
}
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FFog.shader)

![[aa8f778ba81f5d48c2e06800891e7134_MD5.webp]]

Fog 场景

# 扫描线

思路与相交高亮效果类似，只是这里是 Post Process。自定义一个 [0,1] 变化的值_CurValue，根据_CurValue 与深度值的差进行颜色的插值：

```
fixed4 frag (v2f i) : SV_Target
{
    fixed4 originColor = tex2D(_MainTex, i.uv.xy);
    float depth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv.zw));
    float linear01Depth = Linear01Depth(depth);
    float halfWidth = _LineWidth / 2;
    float v = saturate(abs(_CurValue - linear01Depth) / halfWidth); //线内返回(0, 1)，线外返回1
    return lerp(_LineColor, originColor, v);
}
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FScanLine.shader)

![[8a5f6b329c528a4a855e12d23f41a880_MD5.webp]]

ScanLine 场景

# 水淹

利用上面提到的第二种重建世界空间坐标的方法得到世界空间坐标，判断该坐标的 Y 值是否在给定阈值下，如果是则混合原图颜色和水的颜色：

```
fixed4 frag (v2f i) : SV_Target
{
    fixed4 col = tex2D(_MainTex, i.uv.xy);
    float depth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv.zw));
    float linearEyeDepth = LinearEyeDepth(depth);
    float3 worldPos = _WorldSpaceCameraPos.xyz + i.frustumDir * linearEyeDepth;
                
    if(worldPos.y < _WaterHeight)
        return lerp(col, _WaterColor, _WaterColor.a); //半透明

    return col;
}
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FWaterFlooded.shader)

![[f72d713427cce38f7905b4f065e792dc_MD5.webp]]

WaterFlooded 场景

# 垂直雾效

利用上面提到的第二种重建世界空间坐标的方法得到世界空间坐标，让雾的浓度随着 Y 值变化：

```
fixed4 frag (v2f i) : SV_Target
{
    fixed4 col = tex2D(_MainTex, i.uv.xy);
    float depth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv.zw));
    float linearEyeDepth = LinearEyeDepth(depth);
    float3 worldPos = _WorldSpaceCameraPos + linearEyeDepth * i.frustumDir.xyz;

    float fogDensity = (worldPos.y - _StartY) / (_EndY - _StartY);
    fogDensity = saturate(fogDensity * _FogDensity);
                
    fixed3 finalColor = lerp(_FogColor, col, fogDensity).xyz;
    return fixed4(finalColor, 1.0);
}
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FVerticalFog.shader)

![[9f21be08c43cbbcda925da3f7b4df461_MD5.webp]]

VerticalFog 场景

# 边缘检测

思路是取当前像素的附近 4 个角，分别计算出两个对角的深度值差异，将这两个差异值相乘就得到我们判断边缘的值。  
首先是得到 4 个角：

```
//vertex
//Robers算子
o.uv[1] = uv + _MainTex_TexelSize.xy * float2(-1, -1);
o.uv[2] = uv + _MainTex_TexelSize.xy * float2(-1, 1);
o.uv[3] = uv + _MainTex_TexelSize.xy * float2(1, -1);
o.uv[4] = uv + _MainTex_TexelSize.xy * float2(1, 1);
```

然后是得到这 4 个角的深度值：

```
float sample1 = Linear01Depth(UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv[1])));
float sample2 = Linear01Depth(UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv[2])));
float sample3 = Linear01Depth(UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv[3])));
float sample4 = Linear01Depth(UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv[4])));
```

最后就是根据对角差异来得到判断边缘的值：

```
float edge = 1.0;
//对角线的差异相乘
edge *= abs(sample1 - sample4) < _EdgeThreshold ? 1.0 : 0.0;
edge *= abs(sample2 - sample3) < _EdgeThreshold ? 1.0 : 0.0;

return edge;
// return lerp(0, col, edge); //描边
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FEdgeDetection.shader)

![[53076353ed3980d076d80e1a29017c3c_MD5.webp]]

EdgeDetection 场景

PS：上面这种只用深度值来检测边缘的效果并不太好，最好结合法线图来判断，原理都是一样的。

# 运动模糊 (Motion Blur)

![[7aadb194b02b3f359dca989ea1950907_MD5.webp]]

MotionBlur 场景

运动模糊主要用在竞速类游戏中用来体现出速度感。这里介绍的运动模糊只能用于**周围物体不动，摄像机动**的情景。  
思路是利用上面提到的重建世界坐标方法得到世界坐标，由于该世界坐标在摄像机运动过程中都是不动的，因此可以将该世界空间坐标分别转到摄像机运动前和运动后的坐标系中，从而得到两个 NDC 坐标，利用这两个 NDC 坐标就能得到该像素运动的轨迹，在该轨迹上多次取样进行模糊即可。  
首先是得到世界坐标（这里使用提到的第一种重建方法）：

```
float depth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv.zw));
float4 H = float4(i.uv.x * 2 - 1, i.uv.y * 2 - 1, depth * 2 - 1, 1); //NDC坐标
float4 D = mul(_CurrentInverseVP, H);
float4 W = D / D.w; //将齐次坐标w分量变1得到世界坐标
```

然后是计算出运算前后的 NDC 坐标：

```
float4 currentPos = H;
float4 lastPos = mul(_LastVP, W);
lastPos /= lastPos.w;
```

最后就是在轨迹上多次取样进行模糊：

```
//采样两点所在直线上的点，进行模糊
fixed4 col = tex2D(_MainTex, i.uv.xy);
float2 velocity = (currentPos - lastPos) / 2.0;
float2 uv = i.uv;
uv += velocity;
int numSamples = 3;
for(int index = 1; index < numSamples; index++, uv += velocity)
{
    col += tex2D(_MainTex, uv);
}
col /= numSamples;
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FMotionBlur.shader)

# 景深 (Depth Of Field)

![[2dc7222d3d4f3469ba8fb883fb63086d_MD5.webp]]

DepthOfField 场景

景深是一种聚焦处清晰，其他地方模糊的效果，在摄影中很常见。  
思路是首先渲染一张模糊的图，然后在深度图中找到聚焦点对应的深度，该深度附近用原图，其他地方渐变至模糊图。  
第一步是使用 SimpleBlur Shader 渲染模糊的图，这里我只是简单地采样当前像素附近的 9 个点然后平均，你可以选择更好的模糊方式：

```
v2f vert (appdata v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);

    o.uv[0] = v.uv + _MainTex_TexelSize.xy * float2(-1, -1) * _BlurLevel;
    o.uv[1] = v.uv + _MainTex_TexelSize.xy * float2(-1, 0) * _BlurLevel;
    o.uv[2] = v.uv + _MainTex_TexelSize.xy * float2(-1, 1) * _BlurLevel;
    o.uv[3] = v.uv + _MainTex_TexelSize.xy * float2(0, -1) * _BlurLevel;
    o.uv[4] = v.uv + _MainTex_TexelSize.xy * float2(0, 0) * _BlurLevel;
    o.uv[5] = v.uv + _MainTex_TexelSize.xy * float2(0, 1) * _BlurLevel;
    o.uv[6] = v.uv + _MainTex_TexelSize.xy * float2(1, -1) * _BlurLevel;
    o.uv[7] = v.uv + _MainTex_TexelSize.xy * float2(1, 0) * _BlurLevel;
    o.uv[8] = v.uv + _MainTex_TexelSize.xy * float2(1, 1) * _BlurLevel;

    return o;
}
            
fixed4 frag (v2f i) : SV_Target
{
    fixed4 col = tex2D(_MainTex, i.uv[0]);
    col += tex2D(_MainTex, i.uv[1]);
    col += tex2D(_MainTex, i.uv[2]);
    col += tex2D(_MainTex, i.uv[3]);
    col += tex2D(_MainTex, i.uv[4]);
    col += tex2D(_MainTex, i.uv[5]);
    col += tex2D(_MainTex, i.uv[6]);
    col += tex2D(_MainTex, i.uv[7]);
    col += tex2D(_MainTex, i.uv[8]);
    col /= 9;
    return col;
}
```

第二步就是传递该模糊的图给 DepthOfField Shader：

```
RenderTexture blurTex = RenderTexture.GetTemporary(source.width, source.height, 16);
Graphics.Blit(source, blurTex, blurMat);
dofMat.SetTexture("_BlurTex", blurTex);
Graphics.Blit(source, destination, dofMat);
```

第三步就是在 DepthOfField Shader 中根据焦点来混合原图颜色和模糊图颜色：

```
fixed4 col = tex2D(_MainTex, i.uv.xy);
fixed4 blurCol = tex2D(_BlurTex, i.uv.zw);
float depth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.uv.zw));
float linearDepth = Linear01Depth(depth);
float v = saturate(abs(linearDepth - _FocusDistance) * _FocusLevel);
return lerp(col, blurCol, v);
```

[完整代码点这里](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Fblob%2Fmaster%2FUnityShaderProject%2FAssets%2FDepth%2FShaders%2FDepthOfField.shader)

# 完整项目地址

[https://github.com/KaimaChen/Unity-Shader-Demo/tree/master/UnityShaderProject](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2FKaimaChen%2FUnity-Shader-Demo%2Ftree%2Fmaster%2FUnityShaderProject)

# 参考

[Unity Docs - Camera’s Depth Texture](https://link.jianshu.com?t=https%3A%2F%2Fdocs.unity3d.com%2FManual%2FSL-CameraDepthTexture.html)  
[Unity Docs - Platform-specific rendering differences](https://link.jianshu.com?t=https%3A%2F%2Fdocs.unity3d.com%2FManual%2FSL-PlatformDifferences.html)  
[神奇的深度图：复杂的效果，不复杂的原理](https://link.jianshu.com?t=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F27547127%3Frefer%3Dchenjiadong)  
[SPECIAL EFFECTS WITH DEPTH](https://link.jianshu.com?t=https%3A%2F%2Fwww.google.com%2Furl%3Fsa%3Dt%26rct%3Dj%26q%3D%26esrc%3Ds%26source%3Dweb%26cd%3D2%26ved%3D0ahUKEwiM1Zic6efWAhXLrVQKHXLyCywQFggsMAE%26url%3Dhttps%253A%252F%252Fhalckemy.s3.amazonaws.com%252Fuploads%252Fpdf_file%252Ffile%252F91281%252FSiggraph2011_SpecialEffectsWithDepth_WithNotes.pdf%26usg%3DAOvVaw2AoGZitmnmb76btIOG0YWB)  
[GPU Gems - Chapter 27. Motion Blur as a Post-Processing Effect](https://link.jianshu.com?t=https%3A%2F%2Fdeveloper.nvidia.com%2Fgpugems%2FGPUGems3%2Fgpugems3_ch27.html)  
《Unity Shader 入门精要》  
《Unity 3D ShaderLab 开发实战详解》