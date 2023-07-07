# 基本原理与实现

主要使用**噪声**和**透明度测试**，从噪声图中读取某个通道的值，然后使用该值进行透明度测试。  
主要代码如下：

```
fixed cutout = tex2D(_NoiseTex, i.uvNoiseTex).r;
clip(cutout - _Threshold);
```

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/Basic.shader)

![[61b51a6ca46395c8504ac4b803f65a4d_MD5.webp]]

Basic 场景

# 边缘颜色

如果纯粹这样镂空，则效果太朴素了，因此通常要在镂空边缘上弄点颜色来模拟火化、融化等效果。

## 1. 纯颜色

第一种实现很简单，首先定义_EdgeLength 和_EdgeColor 两个属性来决定边缘多长范围要显示边缘颜色；然后在代码中找到合适的范围来显示边缘颜色。  
主要代码如下：

```
//Properties
_EdgeLength("Edge Length", Range(0.0, 0.2)) = 0.1
_EdgeColor("Border Color", Color) = (1,1,1,1)
...
//Fragment
if(cutout - _Threshold < _EdgeLength)
    return _EdgeColor;
```

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/EdgeColor.shader)

![[07f4dad6e67a495606cb14e883ab0760_MD5.webp]]

EdgeColor 场景

## 2. 两种颜色混合

第一种纯颜色的效果并不太好，更好的效果是混合两种颜色，来实现一种更加自然的过渡效果。  
主要代码如下：

```
if(cutout - _Threshold < _EdgeLength)
{
    float degree = (cutout - _Threshold) / _EdgeLength;
    return lerp(_EdgeFirstColor, _EdgeSecondColor, degree);
}
```

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/TwoEdgeColor.shader)

![[c72ee998168cd6357e62977b4a3bfbd2_MD5.webp]]

TwoEdgeColor 场景

## 3. 边缘颜色混合物体颜色

为了让过渡更加自然，我们可以进一步混合边缘颜色和物体原本的颜色。  
主要代码如下：

```
float degree = saturate((cutout - _Threshold) / _EdgeLength); //需要保证在[0,1]以免后面插值时颜色过亮
fixed4 edgeColor = lerp(_EdgeFirstColor, _EdgeSecondColor, degree);

fixed4 col = tex2D(_MainTex, i.uvMainTex);

fixed4 finalColor = lerp(edgeColor, col, degree);
return fixed4(finalColor.rgb, 1);
```

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/BlendOriginColor.shader)

![[247a0d68c980e65392720b3081b3d7a8_MD5.webp]]

BlendOriginColor 场景

## 4. 使用渐变纹理

为了让边缘颜色更加丰富，我们可以进而使用渐变纹理：

![[dad9a491408d89e7ad4c029cfbea609b_MD5.webp]]

然后我们就可以利用 degree 来对这条渐变纹理采样作为我们的边缘颜色：

```
float degree = saturate((cutout - _Threshold) / _EdgeLength);
fixed4 edgeColor = tex2D(_RampTex, float2(degree, degree));

fixed4 col = tex2D(_MainTex, i.uvMainTex);

fixed4 finalColor = lerp(edgeColor, col, degree);
return fixed4(finalColor.rgb, 1);
```

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/Ramp.shader)

![[90cf1ec144e4e05117961db9a2858384_MD5.webp]]

Ramp 场景

# 从特定点开始消融

![[69a74153809c2d6850cb8a5014db7799_MD5.webp]]

DissolveFromPoint 场景

为了从特定点开始消融，我们需要把片元到特定点的距离考虑进 clip 中。  
第一步需要先定义消融开始点，然后求出各个片元到该点的距离（本例子是在模型空间中进行）：

```
//Properties
_StartPoint("Start Point", Vector) = (0, 0, 0, 0) //消融开始点
...
//Vert
//把点都转到模型空间
o.objPos = v.vertex;
o.objStartPos = mul(unity_WorldToObject, _StartPoint); 
...
//Fragment
float dist = length(i.objPos.xyz - i.objStartPos.xyz); //求出片元到开始点距离
```

第二步是求出网格内两点的最大距离，用来对第一步求出的距离进行归一化。这一步需要在 C# 脚本中进行，思路就是遍历任意两点，然后找出最大距离：

```
public class Dissolve : MonoBehaviour {
    void Start () {
        Material mat = GetComponent<MeshRenderer>().material;
        mat.SetFloat("_MaxDistance", CalculateMaxDistance());
    }
    
    float CalculateMaxDistance() {
        float maxDistance = 0;
        Vector3[] vertices = GetComponent<MeshFilter>().mesh.vertices;
        for(int i = 0; i < vertices.Length; i++)
        {
            Vector3 v1 = vertices[i];
            for(int k = 0; k < vertices.Length; k++)
            {
                if (i == k) continue;

                Vector3 v2 = vertices[k];
                float mag = (v1 - v2).magnitude;
                if (maxDistance < mag) maxDistance = mag;
            }
        }

        return maxDistance;
    }
}
```

同时 Shader 里面也要同时定义_MaxDistance 来存放最大距离的值：

```
//Properties
_MaxDistance("Max Distance", Float) = 0
//Pass
float _MaxDistance;
```

第三步就是归一化距离值

```
//Fragment
float normalizedDist = saturate(dist / _MaxDistance);
```

第四步要加入一个_DistanceEffect 属性来控制距离值对整个消融的影响程度：

```
//Properties
_DistanceEffect("Distance Effect", Range(0.0, 1.0)) = 0.5
...
//Pass
float _DistanceEffect;
...
//Fragment
fixed cutout = tex2D(_NoiseTex, i.uvNoiseTex).r * (1 - _DistanceEffect) + normalizedDist * _DistanceEffect;
clip(cutout - _Threshold);
```

上面已经看到一个合适_DistanceEffect 的效果了，下面贴出_DistanceEffect 为 1 的效果图：

![[6793512789b8d95f9d8ea8cf139fa415_MD5.webp]]

_DistanceEffect = 1

这就完成了从特定点开始消融的效果了，不过有一点要注意，消融开始点最好是在网格上面，这样效果会好点。

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/FromPoint.shader)

### 应用：场景切换

利用这个从特定点消融的原理，我们可以实现场景切换。  
假设我们要实现如下效果：

![[00bfce6e7a584fe3953f5096df7d93c2_MD5.webp]]

来自 Trifox 的图

因为我们原来的 Shader 是从中间开始镂空的，和图中从四周开始镂空有点不同，因此我们需要稍微修改一下计算距离的方式：

```
//Fragment
float normalizedDist = 1 - saturate(dist / _MaxDistance);
```

这时候我们的 Shader 就能从四周开始消融了。  
第二步就是需要修改计算距离的坐标空间，原来我们是在模型空间下计算的，而现在很明显多个不同的物体会同时受消融值的影响，因此我们改为世界空间下计算距离：

```
//Vert
o.worldPos = mul(unity_ObjectToWorld, v.vertex);
//Fragment
float dist = length(i.worldPos.xyz - _StartPoint.xyz);
```

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/ToPoint.shader)  
为了让 Shader 应用到场景物体上好看点，我加了点漫反射代码。

第三步为了计算所有场景的物体的顶点到消融开始点的最大距离，我定义了下面这个脚本：

```
public class DissolveEnvironment : MonoBehaviour {
    public Vector3 dissolveStartPoint;
    [Range(0, 1)]
    public float dissolveThreshold = 0;
    [Range(0, 1)]
    public float distanceEffect = 0.6f;

    void Start () {
        //计算所有子物体到消融开始点的最大距离
        MeshFilter[] meshFilters = GetComponentsInChildren<MeshFilter>();
        float maxDistance = 0;
        for(int i = 0; i < meshFilters.Length; i++)
        {
            float distance = CalculateMaxDistance(meshFilters[i].mesh.vertices);
            if (distance > maxDistance)
                maxDistance = distance;
        }
        //传值到Shader
        MeshRenderer[] meshRenderers = GetComponentsInChildren<MeshRenderer>();
        for(int i = 0; i < meshRenderers.Length; i++)
        {
            meshRenderers[i].material.SetVector("_StartPoint", dissolveStartPoint);
            meshRenderers[i].material.SetFloat("_MaxDistance", maxDistance);
        }
    }
    
    void Update () {
        //传值到Shader，为了方便控制所有子物体Material的值
        MeshRenderer[] meshRenderers = GetComponentsInChildren<MeshRenderer>();
        for (int i = 0; i < meshRenderers.Length; i++)
        {
            meshRenderers[i].material.SetFloat("_Threshold", dissolveThreshold);
            meshRenderers[i].material.SetFloat("_DistanceEffect", distanceEffect);
        }
    }

    //计算给定顶点集到消融开始点的最大距离
    float CalculateMaxDistance(Vector3[] vertices)
    {
        float maxDistance = 0;
        for(int i = 0; i < vertices.Length; i++)
        {
            Vector3 vert = vertices[i];
            float distance = (vert - dissolveStartPoint).magnitude;
            if (distance > maxDistance)
                maxDistance = distance;
        }
        return maxDistance;
    }
}
```

这个脚本同时还提供了一些值来方便控制所有场景的物体。

![[ed3d96e893b43d2ec618422f6ceee286_MD5.webp]]

像这样把场景的物体放到 Environment 物体下面，然后把脚本挂到 Environment，就能实现如下结果了：

![[dae88d424f6a4eb58634b1e451b549d5_MD5.webp]]

DissolveEnvironment 场景

[具体的场景文件点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Scenes/DissolveEnvironment.unity)

# 从特定方向开始消融

![[af9dc9b6f88651963e54f903681c490b_MD5.webp]]

DissolveFromDirectionX 场景

理解了上面的从特定点开始消融，那么理解从特定方向开始消融就很简单了。  
下面实现 X 方向消融的效果。  
第一步求出 X 方向的边界，然后传给 Shader：

```
using UnityEngine;
using System.Collections;

public class DissolveDirection : MonoBehaviour {

    void Start () {
        Material mat = GetComponent<Renderer>().material;
        float minX, maxX;
        CalculateMinMaxX(out minX, out maxX);
        mat.SetFloat("_MinBorderX", minX);
        mat.SetFloat("_MaxBorderX", maxX);
    }
    
    void CalculateMinMaxX(out float minX, out float maxX)
    {
        Vector3[] vertices = GetComponent<MeshFilter>().mesh.vertices;
        minX = maxX = vertices[0].x;
        for(int i = 1; i < vertices.Length; i++)
        {
            float x = vertices[i].x;
            if (x < minX)
                minX = x;
            if (x > maxX)
                maxX = x;
        }
    }
}
```

第二步定义是从 X 正方向还是负方向开始消融，然后求出各个片元在 X 分量上与边界的距离：

```
//Properties
_Direction("Direction", Int) = 1 //1表示从X正方向开始，其他值则从负方向
_MinBorderX("Min Border X", Float) = -0.5 //从程序传入
_MaxBorderX("Max Border X", Float) = 0.5  //从程序传入
...
//Vert
o.objPosX = v.vertex.x;
...
//Fragment
float range = _MaxBorderX - _MinBorderX;
float border = _MinBorderX;
if(_Direction == 1) //1表示从X正方向开始，其他值则从负方向
    border = _MaxBorderX;
```

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/FromDirection.shader)

# 灰烬飞散效果

![[57f269bc58db0d7a46c35ba31347b1b7_MD5.webp]]

DirectionAsh 场景

主要效果就是上面的从特定方向消融加上灰烬向特定方向飞散。  
首先我们需要生成灰烬，我们可以延迟 clip 的时机：

```
float edgeCutout = cutout - _Threshold;
clip(edgeCutout + _AshWidth); //延至灰烬宽度处才剔除掉
```

这样可以在消融边缘上面留下一大片的颜色，而我们需要的是细碎的灰烬，因此我们还需要用白噪声图对这片颜色再进行一次 Dissolve：

```
float degree = saturate(edgeCutout / _EdgeWidth);
fixed4 edgeColor = tex2D(_RampTex, float2(degree, degree));
fixed4 finalColor = fixed4(lerp(edgeColor, albedo, degree).rgb, 1);
if(degree < 0.001) //粗略表明这是灰烬部分
{
    clip(whiteNoise * _AshDensity + normalizedDist * _DistanceEffect - _Threshold); //灰烬处用白噪声来进行碎片化
    finalColor = _AshColor;
}
```

下一步就是让灰烬能够向特定方向飞散，实际上就是操作顶点，让顶点进行偏移，因此这一步在顶点着色器中进行：

```
float cutout = GetNormalizedDist(o.worldPos.y);
float3 localFlyDirection = normalize(mul(unity_WorldToObject, _FlyDirection.xyz));
float flyDegree = (_Threshold - cutout)/_EdgeWidth;
float val = max(0, flyDegree * _FlyIntensity);
v.vertex.xyz += localFlyDirection * val;
```

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/DirectionAsh.shader)

# Trifox 的镜头遮挡消融

![[296372eb07c4c0f77893acdf4c91417e_MD5.webp]]

Trifox 场景

具体原理参考 [Unity 案例介绍: Trifox 里的遮挡处理和溶解着色器 (一)](https://link.jianshu.com?t=http://gad.qq.com/program/translateview/7187984)

[完整代码点这里](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/blob/master/UnityShaderProject/Assets/Dissolve/Shaders/Trifox.shader) 我这里的实现是简化版。

# 项目代码

项目代码在 Github 上，[点这里查看](https://link.jianshu.com?t=https://github.com/KaimaChen/Unity-Shader-Demo/tree/master/UnityShaderProject/Assets/Dissolve)

# 参考

《Unity Shader 入门精要》  
[Tutorial - Burning Edges Dissolve Shader in Unity](https://link.jianshu.com?t=http://www.codeavarice.com/dev-blog/tutorial-burning-edges-dissolve-shader-in-unity)  
[A Burning Paper Shader](https://link.jianshu.com?t=http://kylehalladay.com/blog/tutorial/2015/11/10/Dissolve-Shader-Redux.html)  
[Unity 案例介绍: Trifox 里的遮挡处理和溶解着色器 (一)](https://link.jianshu.com?t=http://gad.qq.com/program/translateview/7187984)  
[《Trifox》中的遮挡处理和溶解着色器技术（下）](https://link.jianshu.com?t=http://www.gad.qq.com/article/detail/25821)