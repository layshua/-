在某些特定应用场景，比如说屏幕空间反射，会要求我们从深度缓冲中重建像素点的世界空间位置。本文介绍在 Unity 中如何从深度缓冲中重建世界空间位置。

## 深度缓冲

首先先来看看在 Unity 中怎么计算深度。

UnityCG.cginc

```c
#define COMPUTE_EYEDEPTH(o) o = -mul( UNITY_MATRIX_MV, v.vertex ).z
#define COMPUTE_DEPTH_01 -(mul( UNITY_MATRIX_MV, v.vertex ).z * _ProjectionParams.w)
```

其中，`_ProjectionParams.w` 是 $\frac{1}{far}$ 

符号取反的原因是在 Unity 的观察空间（View space）看向 $-z$，符号取反是为了获得距离正数。

**从上式可知，Unity 中的观察线性深度（Eye depth）就是顶点在观察空间（View space）中的 z 分量，而 01 线性深度（01 depth）就是观察线性深度通过除以摄像机远平面重新映射到 [0，1] 区间所得到的值。**

我们可以从深度缓冲中采样得到深度值，并使用 Unity 中内置的功能函数将原始数据转换成线性深度。


```cs
// zBufferParam = { (f-n)/n, 1, (f-n)/n*f, 1/f }  
float Linear01Depth(float depth, float4 zBufferParam)  
{  
    return 1.0 / (zBufferParam.x * depth + zBufferParam.y);  
}

// zBufferParam = { (f-n)/n, 1, (f-n)/n*f, 1/f }  
float LinearEyeDepth(float depth, float4 zBufferParam)  
{  
    return 1.0 / (zBufferParam.z * depth + zBufferParam.w);  
}
```

## 从 NDC 空间中重建

第一种方法是通过像素的屏幕坐标位置来计算。

![[6471c3e36c9050bb5ad67495a3088a7b_MD5.png]]

首先将屏幕空间坐标转换到 NDC 空间中。

```c
float4 ndcPos = (o.screenPos / o.screenPos.w) * 2 - 1;
```

然后将屏幕像素对应在摄像机远平面（Far plane）的点转换到剪裁空间（Clip space）。因为在 NDC 空间中远平面上的点的 z 分量为 1，所以可以直接乘以摄像机的 Far 值来将其转换到剪裁空间（实际就是反向透视除法）。

```c
float far = _ProjectionParams.z;
float3 clipVec = float3(ndcPos.x, ndcPos.y, 1.0) * far;
```

接着通过逆投影矩阵（Inverse Projection Matrix）将点转换到观察空间（View space）。

```c
float3 o.viewVec = mul(unity_CameraInvProjection, clipVec.xyzz).xyz;
```

已知在观察空间中摄像机的位置一定为（0，0，0），所以从摄像机指向远平面上的点的向量就是其在观察空间中的位置。

将向量乘以线性深度值，得到在深度缓冲中储存的值的观察空间位置。

```c
float depth = UNITY_SAMPLE_DEPTH(tex2Dproj(_CameraDepthTexture, i.screenPos));
float3 viewPos = i.viewVec * Linear01Depth(depth);
```

最后将观察空间中的位置变换到世界空间中。

```c
float3 worldPos = mul(UNITY_MATRIX_I_V, float4(viewPos, 1.0)).xyz;
```

附上在 Shader Graph 中的实现。这里 Unity 有 bug 导致如果使用 Transformation Matrix 节点的 Inverse Projection 会报错，所以这里使用了一个 Custom Function 节点输出一个 4x4 矩阵 unity_CameraInvProjection。理论上效果是一样的。

![[94472e125fcd056bf928bcf28d6ddd3e_MD5.png]]

## 在世界空间中重建

第二种方法是利用在世界空间中从摄像机指向屏幕像素点的向量来计算。

![[a2bb8d47bf027dec6e209f61cf7e0731_MD5.png]]

首先构造在世界空间中从摄像机指向屏幕像素点的向量。

```
o.worldSpaceDir = WorldSpaceViewDir(v.vertex);
```

将向量转换到观察空间，存储其 z 分量的值。注意向量和位置的空间转换是不同的，当 w 分量为 0 的时候 Unity 会将其视为向量，而当 w 分量为 1 的时候 Unity 将其视为位置。

```
o.viewSpaceZ = mul(UNITY_MATRIX_V, float4(o.worldSpaceDir, 0.0)).z;
```

在深度缓冲中采样。这里使用 tex2Dproj 而不是 tex2D 的原因是 screenPos 是用 ComputeScreenPos 来计算得到的 [[3]](https://zhuanlan.zhihu.com/p/92315967#ref_3)，用 tex2Dproj 可以帮我们做透视除法。

```
float eyeDepth = UNITY_SAMPLE_DEPTH(tex2Dproj(_CameraDepthTexture, i.screenPos));
eyeDepth = LinearEyeDepth(eyeDepth);
```

因为像素点的观察线性深度就是其在观察空间中的 z 分量，所以根据向量的 z 分量计算其缩放因子，将向量缩放到实际的长度。

```
i.worldSpaceDir *= -eyeDepth / i.viewSpaceZ;
```

最后以摄像机为起点，缩放后的向量为指向向量，得到像素点在世界空间中位置。

```
float3 worldPos = _WorldSpaceCameraPos + i.worldSpaceDir;
```

附上在 Shader Graph 中的实现。

![[053339cd6c7e8e3217cf385dd760a206_MD5.png]]

## 正交摄像机的情况

如果摄像机不是透视而是正交的，做法上就有些不同。

计算观察空间（View space）中顶点的 xy 分量。

```
float4 ndcPos = (o.screenPos / o.screenPos.w) * 2 - 1;
o.viewVec = float3(unity_OrthoParams.xy * ndcPos.xy, 0);
```

正交摄像机的深度缓冲是线性的。根据深度值在远近平面之间作线性插值，就得到顶点在观察空间中的 z 分量。

```
float depth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.screenPos));
float z = -lerp(near, far, depth);
```

最后将观察空间中的位置转换到世界空间。

```
float3 worldPos = mul(UNITY_MATRIX_I_V, float4(viewPos, 1)).xyz;
```

这里不知道为什么 Unity 对正交摄像机的深度缓冲无法采样出正确的深度值，需要重新变换才能得到正确的结果 [[4]](https://zhuanlan.zhihu.com/p/92315967#ref_4)。

```
// Wrong depth
// float depth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.screenPos));

// Correct depth
float rawDepth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.screenPos));
float ortho = (far - near) * (1 - rawDepth) + near;
float depth = lerp(LinearEyeDepth(rawDepth), ortho, unity_OrthoParams.w) / far;
```

附上在 Shader Graph 中的实现。其中 Custom Function 节点为：

```
float ortho = (_ProjectionParams.z - _ProjectionParams.y) * (1 - rawDepth) 
              + _ProjectionParams.y;
depth =  lerp(eyeDepth, ortho, unity_OrthoParams.w) / _ProjectionParams.z;
```

![[e69e9c0c0b080a15c527572d6751f125_MD5.png]]

以上方法可以根据实际渲染的对象是在世界中物体（贴花）还是屏幕大小的四边形（后处理）来灵活使用。最后附上大神写的代码作为参考，这个项目通过后处理将深度缓冲转换成世界空间位置并可视化。

[keijiro/DepthInverseProjection](https://github.com/keijiro/DepthInverseProjection)

**代码**

*   NDC 方法

```
#pragma vertex vert
#pragma fragment frag

#include "UnityCG.cginc"

struct v2f
{
    float4 vertex : SV_POSITION;
    float4 screenPos : TEXCOORD0;
    float3 viewVec : TEXCOORD1;
};

v2f vert(appdata_base v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);

    // Compute texture coordinate
    o.screenPos = ComputeScreenPos(o.vertex);

    // NDC position
    float4 ndcPos = (o.screenPos / o.screenPos.w) * 2 - 1;

    // Camera parameter
    float far = _ProjectionParams.z;

    // View space vector pointing to the far plane
    float3 clipVec = float3(ndcPos.x, ndcPos.y, 1.0) * far;
    o.viewVec = mul(unity_CameraInvProjection, clipVec.xyzz).xyz;

    return o;
}

sampler2D _CameraDepthTexture;

half4 frag(v2f i) : SV_Target
{
    // Sample the depth texture to get the linear 01 depth
    float depth = UNITY_SAMPLE_DEPTH(tex2Dproj(_CameraDepthTexture, i.screenPos));
    depth = Linear01Depth(depth);

    // View space position
    float3 viewPos = i.viewVec * depth;

    // Pixel world position
    float3 worldPos = mul(UNITY_MATRIX_I_V, float4(viewPos, 1)).xyz;

    return float4(worldPos, 1.0);
}
```

*   世界空间方法

```
#pragma vertex vert
#pragma fragment frag

#include "UnityCG.cginc"

struct v2f
{
    float4 vertex : SV_POSITION;
    float4 screenPos : TEXCOORD0;
    float3 worldSpaceDir : TEXCOORD1;
    float viewSpaceZ : TEXCOORD2;
};

v2f vert(appdata_base v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);

    // World space vector from camera to the vertex position
    o.worldSpaceDir = WorldSpaceViewDir(v.vertex);

    // Z value of the vector in view space
    o.viewSpaceZ = mul(UNITY_MATRIX_V, float4(o.worldSpaceDir, 0.0)).z;

    // Compute texture coordinate
    o.screenPos = ComputeScreenPos(o.vertex);
    return o;
}

sampler2D _CameraDepthTexture;

half4 frag(v2f i) : SV_Target
{
    // Sample the depth texture to get the linear eye depth
    float eyeDepth = UNITY_SAMPLE_DEPTH(tex2Dproj(_CameraDepthTexture, i.screenPos));
    eyeDepth = LinearEyeDepth(eyeDepth);

    // Rescale the vector
    i.worldSpaceDir *= -eyeDepth / i.viewSpaceZ;

    // Pixel world position
    float3 worldPos = _WorldSpaceCameraPos + i.worldSpaceDir;

    return float4(worldPos, 1.0);
}
```

*   正交摄像机的情况

```
#pragma vertex vert
#pragma fragment frag

#include "UnityCG.cginc"

struct v2f
{
    float4 vertex : SV_POSITION;
    float4 screenPos : TEXCOORD0;
    float3 viewVec : TEXCOORD1;
};

v2f vert(appdata_base v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);

    // Compute texture coordinate
    o.screenPos = ComputeScreenPos(o.vertex);

    // NDC position
    float4 ndcPos = (o.screenPos / o.screenPos.w) * 2 - 1;

    // View space vector from near plane pointing to far plane
    o.viewVec = float3(unity_OrthoParams.xy * ndcPos.xy, 0);

    return o;
}

sampler2D _CameraDepthTexture;

half4 frag(v2f i) : SV_Target
{
    // Camera parameters
    float near = _ProjectionParams.y;
    float far = _ProjectionParams.z;

    // Sample the depth texture to get the linear depth
    float rawDepth = UNITY_SAMPLE_DEPTH(tex2D(_CameraDepthTexture, i.screenPos));
    float ortho = (far - near) * (1 - rawDepth) + near;
    float depth = lerp(LinearEyeDepth(rawDepth), ortho, unity_OrthoParams.w) / far;

    // Linear interpolate between near plane and far plane by depth value
    float z = -lerp(near, far, depth);

    // View space position
    float3 viewPos = float3(i.viewVec.xy, z);

    // Pixel world position
    float3 worldPos = mul(UNITY_MATRIX_I_V, float4(viewPos, 1)).xyz;

    return float4(worldPos, 1.0);
}
```

## 参考

1.  [^](https://zhuanlan.zhihu.com/p/92315967#ref_1_0) w is 1/FarPlane. [https://docs.unity3d.com/Manual/SL-UnityShaderVariables.html](https://docs.unity3d.com/Manual/SL-UnityShaderVariables.html)
2.  [^](https://zhuanlan.zhihu.com/p/92315967#ref_2_0)Unity's convention, where forward is the positive Z axis. [https://docs.unity3d.com/ScriptReference/Camera-worldToCameraMatrix.html](https://docs.unity3d.com/ScriptReference/Camera-worldToCameraMatrix.html)
3.  [^](https://zhuanlan.zhihu.com/p/92315967#ref_3_0)[https://www.jianshu.com/p/df878a386bec](https://www.jianshu.com/p/df878a386bec)
4.  [^](https://zhuanlan.zhihu.com/p/92315967#ref_4_0)[https://forum.unity.com/threads/getting-scene-depth-z-buffer-of-the-orthographic-camera.601825/#post-4966334](https://forum.unity.com/threads/getting-scene-depth-z-buffer-of-the-orthographic-camera.601825/#post-4966334)