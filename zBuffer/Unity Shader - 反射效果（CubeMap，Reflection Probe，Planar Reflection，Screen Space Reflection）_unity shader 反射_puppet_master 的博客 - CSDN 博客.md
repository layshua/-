
## 反射的相关基本内容


## Planar Reflection

Planar Reflection，平面反射。顾名思义，就是在平面上运用的反射，名字也正是这种反射方式的限制，只能用于平面，而且是高度一致的平面，多个高度不一的平面效果也不正确（除非针对每个平面单独计算，消耗嘛，你懂得）。一般情况下，一个平面整体反射效果基本可以满足需求，而且对于实时渲染反射效果，需要将要反射的物体多渲染一次，控制好层级数量的话，性能至少是在可以接受的范围，并且相对于其他几种反射效果来说，平面反射的效果是最好的，所以 Planar Reflection 目前是实时反射效果中使用得比较多的一个方案。

### 平面方程以及平面反射相关推导

既然说到平面反射，自然，我们得先找到一个方法表示一个平面。常见的两种平面表示方法：

一般式：可以表示为 Ax + By + Cz + D = 0（其中 A,B,C,D 为已知常数，并且 A,B,C 不同时为零）。

点法式：平面可以由平面上任意一点和垂直于平面的任意一个向量确定，这个垂直于平面的向量称之为平面的法向量。假设平面上一个确定点 P0（x0，y0，z0），法向量为 N（a，b，c），那么平面上任意点 P（x，y，z），满足 PP0 · N = 0，即（x - x0， y - y0，z - z0） · （a，b，c） = 0。

两种方式各有优点，一般式变量数量少，但是没有什么有用信息，点法式包含面法线信息，但是需要变量数较多。那么把二者结合一下，或者说变形一下即可。比如点法式点乘展开后得到 ax + by + cz + (-ax0 - by0 - cz0) = 0，即为一般式。其中 - ax0 - by0 - cz0 为常数，表示为 d。那么点法式方程就可以表示为 N·P + d = 0 的形式，其中 P 为平面上任意一点，N 为法向量。那么，d 表示神魔恋？要是硬说几何意义的话，可以表示为原点到平面上任意点在法线上的投影长度。所以要表示一个面，我们知道面的法向量 N，然后再知道面上任意一点 P0，就可以求得 d = -Dot(N，P0)。

知道了平面方程的表示，我们还需要再温故一下反射的基本原理，开头我们推导 Reflect 函数时有过一个示意图，这里面我们再画一张针对平面的：

![](<images/1692014244368.png>)

我们要想求得一个点 A 相对于平面的反射点 A‘，根据反射定律可知，|AB| = |A'B|，BA 与 N 同向，已知 A 点坐标的话，我们就可以求得 A’ = A - 2*|AB| * N。

所以，问题就变成了求空间中任意一点 A，到平面 N·P + d = 0 的距离 | AB|，再来一张图，推导一下 AB 距离：

![](<images/1692014244425.png>)

P 为平面上任意一点，A 为空间中一点，B 为点 A 在平面上的投影点，N 为平面法线（为简化问题，N 视为单位向量）。PAB 构成三角形，BAP 夹角θ。从三角形余弦定理易得 | BA| = |PA|cosθ；再通过向量点乘的公式，BA 与 N 同向，PA 与 N 夹角也为θ，所以 dot（PA，N）= |PA||N|cosθ => cosθ = dot（PA，N）/ （|PA||N|），代入 | BA| = |PA|cosθ，|BA| = dot（PA，N）/ |N|，由于 N 为单位向量，故 | BA| = dot（PA，N）。将 PA 拆分，|BA| = dot（A - P， N） = dot（A，N） - dot（P，N）。这时候就需要我们的平面方程推导公式了，根据点法式的结果我们知道，对于平面上任意一点 P，可以表示为 P·N + d = 0 的形式，即 dot（P，N） = -d. 所以最终 | BA| = dot（A，N）+ d。进而得到 A 对于平面的对称点 A’ = A - 2（dot（A，N） + d）N。

我们已知 N（nx，ny，nz），d 的值，A（x，y，z）点作为我们要变换的点，我们需要将 A’（x’，y’，z‘）的计算公式表示为矩阵的形式，需要将各分量拆分出来：

x’ = x - 2（x * nx + y * ny + z * nz + d）* nx = （1 - 2nx * nx）x +（ -2nx * ny）y + （-2nx * nz）z + （-2dnx）

y’ = y - 2（x * nx + y * ny + z * nz + d） * ny = （-2nx * ny）x + （1 - 2ny * ny）y + （-2ny * nz）z + （-2dny）

z’ = z - 2（x * nx + y * ny + z * nz + d） * nz = （-2nx * nz）x  + （-2ny * nz）y + （1 - 2nz * nz）z + （-2dnz）

如此复杂的计算公式，我们已经把 x，y，z 对应的系数和常数抽取出来了，是时候看一下矩阵的威力了，把上述计算公式改为矩阵的形式，Unity 是 OpenGL 风格的矩阵，即矩阵 * 列向量的形式：

![](<images/1692014244537.png>)

![](<images/1692014245238.png>)

至此，我们就得到了用于变换空间中任意一点 A 相对于平面 P·N + d = 0 的反射变换矩阵，我们假设其为 R。

注：关于平面方程的表示，可以参考[这篇文章](http://www.360doc.com/content/17/0601/23/42479807_659140224.shtml)。

### 平面反射效果实现

下面看一下要怎样用这个矩阵来实现平面反射的效果。首先，也是最重要的，我们需要一个相机，与当前正常相机关于平面对称，也就是说，我们把正常相机变换到这个对称的位置，然后将这个相机的渲染结果输出到一张 RT 上，就可以得到对称位置的图像了。我们得到了平面反射的矩阵 R，下面我们需要考虑的就是在哪个阶段使用这个反射矩阵 R。我们知道，渲染物体需要通过 MVP 变换（可以参考本人之前关于软渲染的 blog），物体首先通过 M 矩阵，从物体空间变换到世界空间，然后通过 V 矩阵，从世界空间变换到视空间，最后通过投影矩阵 P 变换到裁剪空间。我们把 R 矩阵插在 V 之后，在一个物体在进行 MV 变换后，变到正常相机坐标系下，然后再进行一次反射变换，就相当于变换到了相对于平面对称的相机坐标系下，然后再进行正常的投影变换，就可以得到反射贴图了。代码如下：

```
/********************************************************************
 FileName: PlanarReflection.cs
 Description: 平面反射效果
 history: 15:8:2018 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using UnityEngine;
 
[ExecuteInEditMode]
public class PlanarReflection : MonoBehaviour
{
    private Camera reflectionCamera = null;
    private RenderTexture reflectionRT = null;
    private static bool isReflectionCameraRendering = false;
    private Material reflectionMaterial = null;
 
    private void OnWillRenderObject()
    {
        if (isReflectionCameraRendering)
            return;
 
        isReflectionCameraRendering = true;
       
        if (reflectionCamera == null)
        {
            var go = new GameObject("Reflection Camera");
            reflectionCamera = go.AddComponent<Camera>();
            reflectionCamera.CopyFrom(Camera.current);
        }
        if (reflectionRT == null)
        {
            reflectionRT = RenderTexture.GetTemporary(1024, 1024, 24);
        }
        //需要实时同步相机的参数，比如编辑器下滚动滚轮，Editor相机的远近裁剪面就会变化
        UpdateCamearaParams(Camera.current, reflectionCamera);
        reflectionCamera.targetTexture = reflectionRT;
        reflectionCamera.enabled = false;
 
        //根据上文平面定义，需要平面法向量和平面上任意点，此处使用transform.up为法向量，transform.position为平面上的点
        //即需要保证平面模型的原点在平面上，否则可以尝试增加offset偏移
        var reflectM = CaculateReflectMatrix(transform.up, transform.position);
 
        reflectionCamera.worldToCameraMatrix = Camera.current.worldToCameraMatrix * reflectM;
        
        //需要将背面裁剪反过来，因为仅改变了顶点，没有改变法向量，绕序反向，裁剪会不对
        GL.invertCulling = true;
        reflectionCamera.Render();
        GL.invertCulling = false;
 
        if (reflectionMaterial == null)
        {
            var renderer = GetComponent<Renderer>();
            reflectionMaterial = renderer.sharedMaterial;
        }
        reflectionMaterial.SetTexture("_ReflectionTex", reflectionRT);
 
        isReflectionCameraRendering = false;
    }
 
    Matrix4x4 CaculateReflectMatrix(Vector3 normal, Vector3 positionOnPlane)
    {
        var d = -Vector3.Dot(normal, positionOnPlane);
        var reflectM = new Matrix4x4();
        reflectM.m00 = 1 - 2 * normal.x * normal.x;
        reflectM.m01 = -2 * normal.x * normal.y;
        reflectM.m02 = -2 * normal.x * normal.z;
        reflectM.m03 = -2 * d * normal.x;
 
        reflectM.m10 = -2 * normal.x * normal.y;
        reflectM.m11 = 1 - 2 * normal.y * normal.y;
        reflectM.m12 = -2 * normal.y * normal.z;
        reflectM.m13 = -2 * d * normal.y;
 
        reflectM.m20 = -2 * normal.x * normal.z;
        reflectM.m21 = -2 * normal.y * normal.z;
        reflectM.m22 = 1 - 2 * normal.z * normal.z;
        reflectM.m23 = -2 * d * normal.z;
 
        reflectM.m30 = 0;
        reflectM.m31 = 0;
        reflectM.m32 = 0;
        reflectM.m33 = 1;
        return reflectM;
    }
 
    private void UpdateCamearaParams(Camera srcCamera, Camera destCamera)
    {
        if (destCamera == null || srcCamera == null)
            return;
 
        destCamera.clearFlags = srcCamera.clearFlags;
        destCamera.backgroundColor = srcCamera.backgroundColor;
        destCamera.farClipPlane = srcCamera.farClipPlane;
        destCamera.nearClipPlane = srcCamera.nearClipPlane;
        destCamera.orthographic = srcCamera.orthographic;
        destCamera.fieldOfView = srcCamera.fieldOfView;
        destCamera.aspect = srcCamera.aspect;
        destCamera.orthographicSize = srcCamera.orthographicSize;
    }
}
```

要使用反射贴图，我们在 shader 中增加相应的 Texture 变量即可。不过这个贴图的采样并非使用正常的 uv 坐标，因为我们的贴图是反射相机的输出的 RT，假设这个 RT 我们输出在屏幕上，反射平面上当前像素点对应位置我们屏幕位置上的位置作为 uv 坐标才能找到这一点对应 RT 上的位置。类似之前热扭曲 blog 中 GrabPass 的采样操作。需要在 vertex shader 中通过 ComputeScreenPos 计算屏幕坐标，fragment shader 采样时进行透视校正纹理采样：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.8.15 
//Planar Reflection效果
Shader "Reflection/PlanarReflection"
{	
	SubShader
	{
		Tags { "RenderType"="Opaque" }
 
		Pass
		{
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			
			#include "UnityCG.cginc"
 
			struct appdata
			{
				float4 vertex : POSITION;
			};
 
			struct v2f
			{
				float4 screenPos : TEXCOORD0;
				float4 vertex : SV_POSITION;
			};
 
			sampler2D _ReflectionTex;
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.screenPos = ComputeScreenPos(o.vertex);
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 col = tex2D(_ReflectionTex, i.screenPos.xy / i.screenPos.w);
				//或者
				//fixed4 col = tex2Dproj(_ReflectionTex, i.screenPos);
				return col;
			}
			ENDCG
		}
	}
}
```

反射效果如下：

![](<images/1692014245935.png>)

近处的人物和球体以及远处的栅栏，反射严丝合缝，可以说，Planar Reflection 是几种反射中效果最好的一种啦。

### 斜视锥体裁剪

上面的反射看似完美，但是却有一个非常致命的问题，看下面一幅图，我们把其中一个模型向下移动，让其逐渐到平面以下：

![](<images/1692014245994.png>)

似乎不太对。。。虚像倒着升了起来，恩，不太符合常理。其实仔细考虑一下 Planar Reflection 的原理，应该就能想明白啦。我们把相机在相对于平面对称的位置进行渲染，物体在平面上没有问题，但是物体在平面下的时候，平面并没有挡住虚像的渲染，在反射图中还会会存在反射图像，在采样的时候，就会得到错误的效果。如下图：

![](<images/1692014246067.png>)

C 为正常相机，AB 为平面，D 为反射相机，GH 为相机 D 的近裁剪面，EF 为相机 D 的远裁剪面。在平面 AB 上方的物体 I 渲染正常，但是在 AB 下方的物体，并没有在 D 的近裁剪面内，所以仍然会渲染，就导致了错误的结果。

那么核心需就是，怎样用反射平面进行裁剪，把在反射平面以下的内容全部裁剪掉。也就是说上图中反射相机 D 的近裁剪面不再是 GH，而是替换为 AB 平面。这个技术也就是所谓的斜视锥体裁剪 - Oblique View Frustum Clippling，可以参考[《Oblique View Frustum   Depth Projection and Clipping》](http://www.terathon.com/lengyel/Lengyel-Oblique.pdf)这篇论文。

Unity 已经为我们提供了一个接口，直接可以对相机和一个平面计算出斜视锥体裁剪投影矩阵，代码如下：

```
/********************************************************************
 FileName: PlanarReflection.cs
 Description: 平面反射效果
 history: 15:8:2018 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using UnityEngine;
 
[ExecuteInEditMode]
public class PlanarReflection : MonoBehaviour
{
    private Camera reflectionCamera = null;
    private RenderTexture reflectionRT = null;
    private bool isReflectionCameraRendering = false;
    private Material reflectionMaterial = null;
 
    private void OnWillRenderObject()
    {
        if (isReflectionCameraRendering)
            return;
 
        isReflectionCameraRendering = true;
       
        if (reflectionCamera == null)
        {
            var go = new GameObject("Reflection Camera");
            reflectionCamera = go.AddComponent<Camera>();
            reflectionCamera.CopyFrom(Camera.current);
        }
        if (reflectionRT == null)
        {
            reflectionRT = RenderTexture.GetTemporary(1024, 1024, 24);
        }
        //需要实时同步相机的参数，比如编辑器下滚动滚轮，Editor相机的远近裁剪面就会变化
        UpdateCamearaParams(Camera.current, reflectionCamera);
        reflectionCamera.targetTexture = reflectionRT;
        reflectionCamera.enabled = false;
 
        var reflectM = CaculateReflectMatrix();
        reflectionCamera.worldToCameraMatrix = Camera.current.worldToCameraMatrix * reflectM;
 
        var normal = transform.up;
        var d = -Vector3.Dot(normal, transform.position);
        var plane = new Vector4(normal.x, normal.y, normal.z, d);
        //用逆转置矩阵将平面从世界空间变换到反射相机空间
        var viewSpacePlane = reflectionCamera.worldToCameraMatrix.inverse.transpose * plane;
        var clipMatrix = reflectionCamera.CalculateObliqueMatrix(viewSpacePlane);
        reflectionCamera.projectionMatrix = clipMatrix;
        
        GL.invertCulling = true;
        reflectionCamera.Render();
        GL.invertCulling = false;
 
        if (reflectionMaterial == null)
        {
            var renderer = GetComponent<Renderer>();
            reflectionMaterial = renderer.sharedMaterial;
        }
        reflectionMaterial.SetTexture("_ReflectionTex", reflectionRT);
 
        isReflectionCameraRendering = false;
    }
 
    Matrix4x4 CaculateReflectMatrix()
    {
        var normal = transform.up;
        var d = -Vector3.Dot(normal, transform.position);
        var reflectM = new Matrix4x4();
        reflectM.m00 = 1 - 2 * normal.x * normal.x;
        reflectM.m01 = -2 * normal.x * normal.y;
        reflectM.m02 = -2 * normal.x * normal.z;
        reflectM.m03 = -2 * d * normal.x;
 
        reflectM.m10 = -2 * normal.x * normal.y;
        reflectM.m11 = 1 - 2 * normal.y * normal.y;
        reflectM.m12 = -2 * normal.y * normal.z;
        reflectM.m13 = -2 * d * normal.y;
 
        reflectM.m20 = -2 * normal.x * normal.z;
        reflectM.m21 = -2 * normal.y * normal.z;
        reflectM.m22 = 1 - 2 * normal.z * normal.z;
        reflectM.m23 = -2 * d * normal.z;
 
        reflectM.m30 = 0;
        reflectM.m31 = 0;
        reflectM.m32 = 0;
        reflectM.m33 = 1;
        return reflectM;
    }
 
    private void UpdateCamearaParams(Camera srcCamera, Camera destCamera)
    {
        if (destCamera == null || srcCamera == null)
            return;
 
        destCamera.clearFlags = srcCamera.clearFlags;
        destCamera.backgroundColor = srcCamera.backgroundColor;
        destCamera.farClipPlane = srcCamera.farClipPlane;
        destCamera.nearClipPlane = srcCamera.nearClipPlane;
        destCamera.orthographic = srcCamera.orthographic;
        destCamera.fieldOfView = srcCamera.fieldOfView;
        destCamera.aspect = srcCamera.aspect;
        destCamera.orthographicSize = srcCamera.orthographicSize;
    }
 
    private Matrix4x4 CaculateObliqueViewFrustumMatrix(Vector4 plane, Camera camera)
    {
        var viewSpacePlane = camera.worldToCameraMatrix.inverse.transpose * plane;
        return camera.CalculateObliqueMatrix(viewSpacePlane);
    }
}
```

这样，通过这样一个 API，我们很容易可以求得被视空间的一个平面裁剪过的投影矩阵，再应用回摄像机，这样就不会出现穿帮啦：

![](<images/1692014246200.png>)

API 虽然简单，但是 API 背后的原理还是需要一番推导的，下面看一下斜视锥体裁剪的推导过程。

首先需要几个预备的知识点。第一点，既然需要平面裁剪，就免不了对平面进行一些坐标空间的变换，上面我们推导过平面的表示，可以用平面法向量和平面上一点与法向量点积的相反数。平面的变换与法线变换类似，不能直接进行变换，对于非 uniform 类型可能导致法线不垂直于平面，所以平面的变换也采用矩阵逆转置的方式进行（关于法线的变换，可以参考之前[描边效果的 blog](https://blog.csdn.net/puppet_master/article/details/54000951)）。即，如果我们已知一个 View 空间的平面 Pv，要想将其转化到裁剪空间 Pc，就需要投影矩阵 M 的逆转置矩阵：

![](<images/1692014246256.png>)

![](<images/1692014247024.png>)

这里就需要线性代数里面的一个性质啦，如果一个矩阵可逆，那么这个矩阵的转置的逆等于逆的转置，对于上面公式来说：

![](<images/1692014247720.png>)

进而可以将 Pv 和 Pc 的变换公式进一步化简为：

![](<images/1692014248417.png>)

下面是第二个预备知识点，关于裁剪空间的。我们知道，经过投影矩阵变换后，会被变换到裁剪空间（实际上此时还没有经过透视除法，属于用齐次坐标系表示的坐标，此处我们为了方便表示最终的立方体，假设进行了透视除法，各个分量除以 w 分量，将变换的结果置为一个标准的立方体，二者的表示结果实际上是等价的），视锥体这个平头截体会被变换成一个立方体（OpenGL 是正方体，区间（-1,1），DX 是普通立方体，xy 区间（-1,1），z 区间（0,1）），我们以 Unity 用的 OpenGL 风格变换为例，最终一个视锥体的前后左右上下六个面在裁剪空间就都会被变换到标准立方体的六个面上。

通过我们之前推导的平面表示的方程，我们很容易地可以表示出裁剪空间下视锥体六个面的平面方程，然后根据

![](<images/1692014249113.png>)

，我们就可以求得在视空间下视锥体六个面的平面方程，如下图（该图片来自上文中提到的论文）：

![](<images/1692014249813.png>)

根据上图中的变换结果，N = M4 + M3，F = M4 - M3，我们需要用一个自定义的平面 P 来代替默认的近裁剪平面 Near，也就是说新的 P 裁剪面也需要满足 P = M4 + M3 这个条件。要想修改近裁剪面 N 使之变成 P，我们就需要调整 M4 或者 M3 这两个向量中的值。M4 是投影矩阵的最后一行，包含了 z 值透视投影等信息，是后续透视除法必须的，所以我们就只能改动 M3 这一行。

M3'= P - M4，F = M4 - M3，F' = M4 - M3'=> F' = 2M4 - P

似乎我们只需要求出 M3'然后带入就大功告成了。不过这样有一个问题，在于本身 P 可能不平行于 XY 平面，得到的远裁剪面也可能不平行，保证了近裁剪面正确，远裁剪面的位置是一个未知的值，可能截断了原来的视锥体，这样可能会导致一些不该被裁剪掉的物体也被裁减掉了；也可能偏移出视锥体很远，进而对深度值造成影响。因此，我们需要考虑让远裁剪面也位于一个合适的位置。

由于公式 F'= 2M4 - P，M4 不能动，P 平面也不能动，那么我们可以考虑给 P 乘以一个系数，一个平面方程，整体乘以一个系数后，表示的仍然是原来的平面，即 F' = 2M4 - uP，这样 M4，uP 都没有变化，但是最终的 F'就会变化了。我们可以求得一个合适的 u，使远裁剪面不截断原来的视锥体同时又与 P 平面夹角最小。那么这个平面就是过视锥体原始边界的一个顶点即可，如下图所示：

![](<images/1692014249944.png>)

HG 为原始的近裁剪面，FE 为原始的远裁剪面，AB 为新的近裁剪面（也就是上文的 P 平面），在裁剪空间（此时应该没进行透视除法，但是为了方便，我们假设 w = 1）下的坐标边界为 E（+-1，+-1,1,1），那么我们要求的远裁剪面的边界点就是 AB 平面面对的裁剪面的边界点即可。也就是说，E 点 xy 坐标的正负取决于 AB 平面的朝向，我们可以用 AB 平面（P 平面）的法线进行表示，由于我们目前可以得到视空间的 P 平面方程，而 E 点坐标目前是裁剪空间的，不过投影变换不会再去改变 xy 的符号，所以我们直接取视空间 P 平面 xy 值的符号即可。

那么，裁剪空间下 E 点坐标为 E（Sign（P.x）， Sign（P.y），1，1），我们可以将其乘以投影矩阵的逆矩阵变换回视空间的 E 点，即 E = （Sign（P.x）， Sign（P.y），1，1） *  ProjectionMatrix.Inverse。此时，我们知道了新的远裁剪面方程 F' = 2M4 - P，又知道新的远裁剪面过 E 点，根据平面公式，F’·E = 0 可得：

（2M4 - uP）·E = 0 => u= 2*（M4 · E） /  （P · E）

我们就可以求得 u 值，进而根据 M3' = uP - M4 得到最终的 M3 值，对投影矩阵进行修改。

不过，此处有一个小优化。首先，看一下 OpenGL 版本的投影矩阵：

![](<images/1692014250075.png>)

投影矩阵的逆矩阵：

![](<images/1692014250140.png>)

关于投影矩阵的推导，可以参考之前[软渲染](https://blog.csdn.net/puppet_master/article/details/80317178)这篇 blog，不过本人之前推导的是 DX 风格的矩阵，GL 风格原理也是一样的。

M4（0，0，-1，0），E = （+-1，+-1，1，1） * Projection.Inverse，看似比较复杂，但是实际上 M4·E ，由于 M4 仅 z 项非零，我们只看 E 点的 z 项即可。那么其实只需要考虑的是 Porjection.Inverse 的第三行与 E 点乘的结果，而 Porjection.InverseM3  = （0，0，0，-1），即 E.z = （0，0，0，-1）· （+-1， +-1， 1， 1） = -1，最终得到 M4·E 为定值 1，因此可以直接省去公式 u= 2*（M4 · E） /  （P · E）中 M4·E 的计算：

u = 2 / （P · E）

完整 C# 代码如下：

```
/********************************************************************
 FileName: PlanarReflection.cs
 Description: 平面反射效果
 history: 15:8:2018 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using UnityEngine;
 
[ExecuteInEditMode]
public class PlanarReflection : MonoBehaviour
{
    private Camera reflectionCamera = null;
    private RenderTexture reflectionRT = null;
    private bool isReflectionCameraRendering = false;
    private Material reflectionMaterial = null;
 
    private void OnWillRenderObject()
    {
        if (isReflectionCameraRendering)
            return;
 
        isReflectionCameraRendering = true;
       
        if (reflectionCamera == null)
        {
            var go = new GameObject("Reflection Camera");
            reflectionCamera = go.AddComponent<Camera>();
            reflectionCamera.CopyFrom(Camera.current);
            reflectionCamera.hideFlags = HideFlags.HideAndDontSave;
        }
        if (reflectionRT == null)
        {
            reflectionRT = RenderTexture.GetTemporary(1024, 1024, 24);
        }
        //需要实时同步相机的参数，比如编辑器下滚动滚轮，Editor相机的远近裁剪面就会变化
        UpdateCamearaParams(Camera.current, reflectionCamera);
        reflectionCamera.targetTexture = reflectionRT;
        reflectionCamera.enabled = false;
 
        var reflectM = CaculateReflectMatrix();
        reflectionCamera.worldToCameraMatrix = Camera.current.worldToCameraMatrix * reflectM;
 
        var normal = transform.up;
        var d = -Vector3.Dot(normal, transform.position);
        var plane = new Vector4(normal.x, normal.y, normal.z, d);
        //用逆转置矩阵将平面从世界空间变换到反射相机空间
        var clipMatrix = CalculateObliqueMatrix(plane, reflectionCamera);
        reflectionCamera.projectionMatrix = clipMatrix;
        
        GL.invertCulling = true;
        reflectionCamera.Render();
        GL.invertCulling = false;
 
        if (reflectionMaterial == null)
        {
            var renderer = GetComponent<Renderer>();
            reflectionMaterial = renderer.sharedMaterial;
        }
        reflectionMaterial.SetTexture("_ReflectionTex", reflectionRT);
 
        isReflectionCameraRendering = false;
    }
 
    Matrix4x4 CaculateReflectMatrix()
    {
        var normal = transform.up;
        var d = -Vector3.Dot(normal, transform.position);
        var reflectM = new Matrix4x4();
        reflectM.m00 = 1 - 2 * normal.x * normal.x;
        reflectM.m01 = -2 * normal.x * normal.y;
        reflectM.m02 = -2 * normal.x * normal.z;
        reflectM.m03 = -2 * d * normal.x;
 
        reflectM.m10 = -2 * normal.x * normal.y;
        reflectM.m11 = 1 - 2 * normal.y * normal.y;
        reflectM.m12 = -2 * normal.y * normal.z;
        reflectM.m13 = -2 * d * normal.y;
 
        reflectM.m20 = -2 * normal.x * normal.z;
        reflectM.m21 = -2 * normal.y * normal.z;
        reflectM.m22 = 1 - 2 * normal.z * normal.z;
        reflectM.m23 = -2 * d * normal.z;
 
        reflectM.m30 = 0;
        reflectM.m31 = 0;
        reflectM.m32 = 0;
        reflectM.m33 = 1;
        return reflectM;
    }
 
    private void OnDisable()
    {
        DestroyImmediate(reflectionCamera.gameObject);
        RenderTexture.ReleaseTemporary(reflectionRT);
        reflectionCamera = null;
        reflectionRT = null;
    }
 
    private void UpdateCamearaParams(Camera srcCamera, Camera destCamera)
    {
        if (destCamera == null || srcCamera == null)
            return;
 
        destCamera.clearFlags = srcCamera.clearFlags;
        destCamera.backgroundColor = srcCamera.backgroundColor;
        destCamera.farClipPlane = srcCamera.farClipPlane;
        destCamera.nearClipPlane = srcCamera.nearClipPlane;
        destCamera.orthographic = srcCamera.orthographic;
        destCamera.fieldOfView = srcCamera.fieldOfView;
        destCamera.aspect = srcCamera.aspect;
        destCamera.orthographicSize = srcCamera.orthographicSize;
    }
 
    private Matrix4x4 CalculateObliqueMatrix(Vector4 plane, Camera camera)
    {
        var viewSpacePlane = camera.worldToCameraMatrix.inverse.transpose * plane;
        var projectionMatrix = camera.projectionMatrix;
 
        var clipSpaceFarPanelBoundPoint = new Vector4(Mathf.Sign(viewSpacePlane.x), Mathf.Sign(viewSpacePlane.y), 1, 1);
        var viewSpaceFarPanelBoundPoint = camera.projectionMatrix.inverse * clipSpaceFarPanelBoundPoint;
        
        var m4 = new Vector4(projectionMatrix.m30, projectionMatrix.m31, projectionMatrix.m32, projectionMatrix.m33);
        //u = 2 * (M4·E)/(E·P)，而M4·E == 1，化简得
        //var u = 2.0f * Vector4.Dot(m4, viewSpaceFarPanelBoundPoint) / Vector4.Dot(viewSpaceFarPanelBoundPoint, viewSpacePlane);
        var u = 2.0f / Vector4.Dot(viewSpaceFarPanelBoundPoint, viewSpacePlane);
        var newViewSpaceNearPlane = u * viewSpacePlane;
 
        //M3' = P - M4
        var m3 = newViewSpaceNearPlane - m4;
 
        projectionMatrix.m20 = m3.x;
        projectionMatrix.m21 = m3.y;
        projectionMatrix.m22 = m3.z;
        projectionMatrix.m23 = m3.w;
 
        return projectionMatrix;
    }
}
```

效果与 API 版本的一致，均可以裁减掉位于原始近裁剪面和平面之间的内容：

![](<images/1692014250183.png>)

## Screen Space Reflection

Screen Space Reflection - 屏幕空间反射（SSR），是一个逼格绝对是 SSR 级别的技术，但是效果有些情况下有硬伤，而且性能堪忧，在前向渲染的情况下性价比较低（需要额外的全屏深度 + 法线），移动平台就更不知道有没有哪位勇者尝试过了。不过这个技术自从 Crytek（Local Reflection）提出之后，各种大作争相把这个技术集成了进来，并且衍生出了很多变种实现，直到最近仍然还在发展，铺天盖地的 paper 和 ppt 看得我眼花缭乱。为何 SSR 会如此受追捧，还是需要看一下 SSR 的原理，我们就会了解其优缺点了。

### SSR 的基本原理与优缺点

SSR，凡是带 SS（屏幕空间的）技术，最近这几年发展的很多，如屏幕空间环境光遮蔽，屏幕空间阴影，屏幕空间次表面散射等等技术，一方面是屏幕空间计算降低了一些消耗，降低不必要的重复计算，再者主要因为延迟渲染，一些操作不方便在直接渲染时进行，所以 SS 系列的技术随着延迟渲染技术发扬光大了，其实延迟渲染本身就是 SS 技术。而 SSR 就是最明显的例子之一，Unity 官方后处理包的 SSR 也只支持延迟渲染。

根据上面的几种反射，我们已经知道，要想计算反射，我们需要求出反射向量。这就需要该点的法线方向，以及该点的视线方向，相机位置已知的话，我们只需要求得该点的位置就可以求得视线方向了。即我们需要物体在视空间的位置以及法线（假设我们在视空间计算反射的话），但是 SS 阶段，一听就是个后处理，这个阶段我们已经没有每个物体的输入了，所以我们就需要从几个特殊的 RT 下手，找到所需要的信息。在深度相关 blog 中，我们推导过通过深度图重建世界空间坐标或视空间坐标，以此可以求得物体在视空间的位置。另外，视空间的法线可以通过 DepthNormalTexture（前向）或者 GBuffer（延迟，需转换到视空间）得到。即该效果需要全屏深度图，全屏法线图。我们在前向渲染可以通过 DepthNormalTexture 得到这两者，而在延迟渲染阶段，GBuffer 中就包含了法线等信息，我们可以直接使用。

求得了全屏幕各个像素点的反射向量后，我们也知道了每个像素点在视空间的位置，要想求得反射值，要怎么办呢？其实如果不看变种的话，基本 SSR 的原理是各种反射里面最简单的。那就是直接从当前点出发，沿着反射方向进行步进，直到碰到东西位为止，碰到的位置对应的颜色值就是该点的反射颜色。哇，这不就是 RayMarching 嘛，在体积光的 blog 中我们也使用过类似的方式。那么接下来就只剩下一个问题了，就是怎样确定碰到了东西。还是 Depth，因为我们要反射的所有物体都在屏幕上，不可能出现屏幕外的东西，而一旦沿着反射方向的光线已经超过了当前这一点的屏幕空间深度，那么就认为光线已经有了碰撞，直接采样该点对应的屏幕空间 Frame Buffer 值就得到了反射值。

当然，这只是最基本的原理，如果完全按照这个原理不进行进一步处理的话，效果不是没法看就是性能极差或者穿帮太多。不过，以上已经足够我们判断 SSR 技术的优缺点了。

优点：

1. 可以实现真 · 实时反射，并且可以用于任意面，无需平面。

2. 无需额外 DrawCall，没有 Planar Reflection 那种翻倍 DC 的问题，计算都在 GPU，解放 CPU，尤其在被反射对象 shader 及其复杂的情况下，更能节省大幅反射渲染的消耗。

3. 一个后处理，无需大规模改动材质系统，容易集成。

4. 最关键的，延迟渲染实现实时反射（静态的还 reflection probe 还是可以用的），貌似也没啥别的好办法啊！！！总不能再来个前向的 Planar Reflection 吧？

缺点：

1. 光线追踪！！！一听这词儿，就不是个省的效果，尤其手机那 GPU。虽然省了 CPU，但是对 GPU 来说负载很大。（后续优化可以大幅度降低消耗，但是相比于普通的后处理，还是很费很费，而且有大量分支计算）。

2. 需要全屏深度和全屏法线。延迟渲染的话可以免费拿到，然而前向渲染的话，单说渲染一遍 DepthNormalMap 的消耗，恐怕就和 Planar Reflection 翻倍的 DC 差不多了，更不要说后续的计算（当然渲染深度图的 shader 比较简单，不过 Planar Reflection 也可以用 lod shader 渲染嘛，而且不管 shader 复杂度，DC 在 cpu 的消耗是移动设备上消耗最大的一点）。当然 DepthNormal 可能还有别的用处，毕竟这个东西当年 Crytek 定义为一个 Mini G Buffer，还是很好很好用的，一些好玩的效果都可以使用 DepthNormal 实现。

3. 效果硬伤，这是这个技术本身的瓶颈，原理上可能就解决不了。只能反射屏幕上的出现过的像素。如果不在屏幕内的，就完全不会反射。比如，角色正对着镜子，背后是摄像机，那么，镜子里面是没有角色的正脸反射的。类似的，还有在视口边界经常容易出现反射丢失的情况。

个人感觉，SSR 对于延迟渲染下是一个比较不错的实时反射的方案。

### 基本 SSR 效果实现

上面大概描述了一遍 SSR 的基本原理，我们按照这个原理实现一版代码。此处我就先使用前向渲染 + DepthNormalTexture 的方式进行 SSR 渲染，暂时没有切换到延迟渲染。

shader 代码如下：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.8.30  
//Screen Space Reflection效果
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			
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
				float4 vertex : SV_POSITION;
				float2 uv : TEXCOORD0; 
				float3 viewRay : TEXCOORD1;
			};
 
			sampler2D _MainTex;
			float4 _MainTex_ST;//(1 / width, 1 / height, width, height)
			sampler2D _CameraDepthTexture;
			//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
			//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
			float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
			float4x4 _CameraProjectionMatrix;//unity_CameraProjection
			float _maxRayMarchingDistance;
			float _maxRayMarchingStep;
			float _rayMarchingStepSize;
			float _depthThickness;
			
			sampler2D _CameraDepthNormalsTexture;
			
			bool checkDepthCollision(float3 viewPos, out float2 screenPos)
			{
				float4 clipPos = mul(_CameraProjectionMatrix, float4(viewPos, 1.0));
 
				clipPos = clipPos / clipPos.w;
				screenPos = float2(clipPos.x, clipPos.y) * 0.5 + 0.5;
				float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, screenPos);
				float depth = DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
				//判断当前反射点是否在屏幕外，或者超过了当前深度值
				return screenPos.x > 0 && screenPos.y > 0 && screenPos.x < 1.0 && screenPos.y < 1.0 && depth < -viewPos.z;
			}
			
			bool viewSpaceRayMarching(float3 rayOri, float3 rayDir, out float2 hitScreenPos)
			{
				int maxStep = _maxRayMarchingStep;
				UNITY_LOOP
				for(int i = 0; i < maxStep; i++)
				{
					float3 currentPos = rayOri + rayDir * _rayMarchingStepSize * i;
					if (length(rayOri - currentPos) > _maxRayMarchingDistance)
						return false;
					if (checkDepthCollision(currentPos, hitScreenPos))
					{
						return true;
					}
				}
				return false;
			}
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.uv = TRANSFORM_TEX(v.uv, _MainTex);
				
				float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
				float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
				o.viewRay = viewRay.xyz / viewRay.w;
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 mainTex = tex2D(_MainTex, i.uv);
				float linear01Depth;
				float3 viewNormal;
				
				float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
				DecodeDepthNormal(cdn, linear01Depth, viewNormal);
				//重建视空间坐标
				float3 viewPos = linear01Depth * i.viewRay;
				float3 viewDir = normalize(viewPos);
				viewNormal = normalize(viewNormal);
				//视空间方向反射方向
				float3 reflectDir = reflect(viewDir, viewNormal);
				float2 hitScreenPos = float2(0,0);
				//从该点开始RayMarching
				if (viewSpaceRayMarching(viewPos, reflectDir, hitScreenPos))
				{
					float4 reflectTex = tex2D(_MainTex, hitScreenPos);
					mainTex.rgb += reflectTex.rgb;
				}
				return mainTex;
			}
			
			ENDCG
		}
	}
}
```

C# 代码如下：

```
/********************************************************************
 FileName: ScreenSpaceReflection.cs
 Description: 屏幕空间反射SSR
 history: 30:8:2018 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{
    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 256)]
    public int maxRayMarchingStep = 64;
    [Range(0, 2.0f)]
    public float rayMarchingStepSize = 0.05f;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;
    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
    }
 
    private void OnEnable()
    {
        currentCamera.depthTextureMode |= DepthTextureMode.DepthNormals;    
    }
 
    private void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.DepthNormals;
    }
 
    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }
 
        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        Graphics.Blit(source, destination, reflectionMaterial, 0);
    }
 
}
```

这里，我们使用了 DepthNormalTexture，其中的 Normal 取出来就是视空间的法线值，可以直接使用，深度是视空间的 01 区间深度，我们需要使用这个深度进行视空间坐标重建的操作，可以参考本人之前的深度相关内容的 blog，此处不再赘述。然后根据反射计算出视空间的反射方向，进而每像素进行 RayMarching 操作，每次步进一定步长，查询是否与超过了当前视深度，超过则认为已经碰撞，对该点采样就是对应的反射颜色。然后将这个颜色直接叠加到屏幕颜色上。

效果如下：

![](<images/1692014250263.png>)

### 逆着视线方向的光线拖影的问题

效果嘛，似乎不太对，每个反射下面都有一个长长的尾巴，感觉看上去好像是已经超过了反射的深度值，我们没有及时制止反射的计算，其实并不是，我们在光线碰撞到之后，就会直接返回，因此这个尾巴另有原因。再看我们判断碰撞的条件，我们只考虑了反射光线在深度之后就认为是碰撞了，但是我们没考虑这个反射光线是从哪里来的。看下面一张示意图：

![](<images/1692014250313.png>)

理想情况下，我们的反射光线是 KH，沿 KH 方向步进，直到深度达到视空间深度后，认为已经进入了圆 IEG 中，停止步进，采样该点作为颜色值。但是如果类似有 CD 反射平面，MF 反射方向并非沿着视线方向，而是与视线方向相反，而 MF 方向的点一定满足大于视方向深度，所以在 MF 方向上步进就会马上返回采样成功。但是这个位置采样并不正确，更不用说这个点本身就没有渲染信息，多个类似的光方向采样都不正确造成了上面的穿帮效果。

要解决这个问题，我们先往简单了想，实际上我们希望的反射都是在屏幕空间内有信息的才需要反射，而类似 F 点的，即使采样正确，这个反射信息也没有，采样出来也是错误的结果。那么，索性，就直接让我们只取距离视空间深度附近的采样点即可。也就是说我们需要一个阈值进行判断，当深度超过 H 所在的深度，并且不超过太多的情况下才认为是正确的深度碰撞，其他部分都舍弃掉。

我们稍微修改一下碰撞判断函数，增加一个深度阈值进行判断：

```
bool checkDepthCollision(float3 viewPos, out float2 screenPos)
{
	float4 clipPos = mul(_CameraProjectionMatrix, float4(viewPos, 1.0));
 
	clipPos = clipPos / clipPos.w;
	screenPos = float2(clipPos.x, clipPos.y) * 0.5 + 0.5;
	float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, screenPos);
	float depth = DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
	//判断当前反射点是否在屏幕外，或者超过了当前深度值并且不超过太多的情况下
	return screenPos.x > 0 && screenPos.y > 0 && screenPos.x < 1.0 && screenPos.y < 1.0 && depth < -viewPos.z && depth + _depthThickness > -viewPos.z;
}
```

效果如下：

![](<images/1692014250363.png>)

这一次，看起来稍微好了一些，没有每个反射下面的拖影了，反射也比较清晰。至于左下角和右下角缺角的情况，这就是所谓的 SSR 的效果硬伤，反射对象在屏幕外，没有信息，反射不到。所以大部分 SSR 的效果都是封闭室内，或者向下视角，尽可能包含被反射物体。

还有一种方法可以得到更加逼真的结果，就是渲染一张背面的深度图，进而得到物体的厚度，但是开销比较大，这里就不再实现了。

### 二分搜索优化

似乎可以结束这篇长长的 blog 了吗？非也！目前的效果，RayMarching 过程中步长很小，迭代次数很多，所以超级费！这和之前体积光的 raymarching 还略有不同，体积光中没有采样，这里的每次步进都需要采样深度图。好在我们加了 UNITY_LOOP，让 shader 不在编译时展开，否则可能指令数就直接超限制了。步长大一些之后就会出现断层的问题，而迭代次数小的话又会出现可能稍微远一点的内容就反射不到的问题。如下图，略微加大步长：

![](<images/1692014250506.png>)

所以下一个问题就是怎样用较低的步进次数和较大的步长进行 RayMarching。

既然说到循环步进，最简单的优化就是二分法了。也就是说，我们开始的时候用一个比较大的步长，步进，如果遇到碰撞，那么缩回去一次，改二分之一步长再步进，再碰，再退缩，再碰，再退缩，以此类推，直到达到阈值或者超过缩步长的次数。

重新修改后的 C# 代码：

```
/********************************************************************
 FileName: ScreenSpaceReflection.cs
 Description: 屏幕空间反射SSR
 history: 30:8:2018 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{
    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 256)]
    public int maxRayMarchingStep = 64;
    [Range(0, 32)]
    public int maxRayMarchingBinarySearchCount = 8;
    [Range(0, 8.0f)]
    public float rayMarchingStepSize = 0.05f;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;
    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
    }
 
    private void OnEnable()
    {
        currentCamera.depthTextureMode |= DepthTextureMode.DepthNormals;    
    }
 
    private void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.DepthNormals;
    }
 
    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }
 
        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);
 
        reflectionMaterial.SetFloat("_maxRayMarchingBinarySearchCount", maxRayMarchingBinarySearchCount);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        Graphics.Blit(source, destination, reflectionMaterial, 0);
    }
 
}
```

Shader 代码：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.8.30  
//Screen Space Reflection效果,Binary Search
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			
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
				float4 vertex : SV_POSITION;
				float2 uv : TEXCOORD0; 
				float3 viewRay : TEXCOORD1;
			};
 
			sampler2D _MainTex;
			float4 _MainTex_ST;//(1 / width, 1 / height, width, height)
			sampler2D _CameraDepthTexture;
			//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
			//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
			float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
			float4x4 _CameraProjectionMatrix;//unity_CameraProjection
			float _maxRayMarchingDistance;
			float _maxRayMarchingStep;
			float _maxRayMarchingBinarySearchCount;
			float _rayMarchingStepSize;
			float _depthThickness;
			
			sampler2D _CameraDepthNormalsTexture;
			
			bool checkDepthCollision(float3 viewPos, out float2 screenPos, inout float depthDistance)
			{
				float4 clipPos = mul(_CameraProjectionMatrix, float4(viewPos, 1.0));
 
				clipPos = clipPos / clipPos.w;
				screenPos = float2(clipPos.x, clipPos.y) * 0.5 + 0.5;
				float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, screenPos);
				float depth = DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
				//判断当前反射点是否在屏幕外，或者超过了当前深度值
				depthDistance = abs(depth + viewPos.z);
				return screenPos.x > 0 && screenPos.y > 0 && screenPos.x < 1.0 && screenPos.y < 1.0 && depth < -viewPos.z;
			}
			
			bool viewSpaceRayMarching(float3 rayOri, float3 rayDir, float currentRayMarchingStepSize, inout float depthDistance, inout float3 currentViewPos, inout float2 hitScreenPos)
			{
				int maxStep = _maxRayMarchingStep; 
				UNITY_LOOP
				for(int i = 0; i < maxStep; i++)
				{
					float3 currentPos = rayOri + rayDir * currentRayMarchingStepSize * i;
					if (length(rayOri - currentPos) > _maxRayMarchingDistance)
						return false;
					if (checkDepthCollision(currentPos, hitScreenPos, depthDistance))
					{
						currentViewPos = currentPos;
						return true;
					}
				}
				return false;
			}
			
			bool binarySearchRayMarching(float3 rayOri, float3 rayDir, inout float2 hitScreenPos)
			{
				float currentStepSize = _rayMarchingStepSize;
				float3 currentPos = rayOri;
				float depthDistance = 0;
				UNITY_LOOP
				for(int i = 0; i < _maxRayMarchingBinarySearchCount; i++)
				{
					if(viewSpaceRayMarching(rayOri, rayDir, currentStepSize, depthDistance, currentPos, hitScreenPos))
					{
						if (depthDistance < _depthThickness)
						{
							return true;
						}
						rayOri = currentPos - rayDir * currentStepSize;
						currentStepSize *= 0.5;
					}
					else
					{
						return false;
					}
				}
				return false;
			}
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.uv = TRANSFORM_TEX(v.uv, _MainTex);
				
				float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
				float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
				o.viewRay = viewRay.xyz / viewRay.w;
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 mainTex = tex2D(_MainTex, i.uv);
				float linear01Depth;
				float3 viewNormal;
				
				float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
				DecodeDepthNormal(cdn, linear01Depth, viewNormal);
				//重建视空间坐标
				float3 viewPos = linear01Depth * i.viewRay;
				float3 viewDir = normalize(viewPos);
				viewNormal = normalize(viewNormal);
				//视空间方向反射方向
				float3 reflectDir = reflect(viewDir, viewNormal);
				float2 hitScreenPos = float2(0,0);
				//从该点开始RayMarching
				if (binarySearchRayMarching(viewPos, reflectDir, hitScreenPos))
				{
					float4 reflectTex = tex2D(_MainTex, hitScreenPos);
					mainTex.rgb += reflectTex.rgb;
				}
				return mainTex;
			}
			
			ENDCG
		}
	}
}
```

这样，在步长可以设置为 2 左右，每次步进 10 次左右，二分搜索 6 次就可以达到之前 0.2 步长，256 次步进的效果：

![](<images/1692014250623.png>)

不过从上图我们也看出，反射信息缺失是多么蛋疼，人挡住的栅栏反射是看不到的，人的小腿不在画面中，因而反射也是缺失的。官方的 Post Processing Stack 中也是一样，但是似乎做了一些模糊，并且反射效果较弱，但是看官方的拖影很严重：

![](<images/1692014250686.png>)

### 屏幕空间光栅化 RayMarching

基本的 SSR 到此就结束了。然而不断有对 SSR 进行的优化，我们还是简单看一下。最著名的一个优化就是按照屏幕空间光栅化的方式进行 RayMarching。这个优化在[这篇论文](http://jcgt.org/published/0003/04/04/paper.pdf)中提出，也是后续各方 SSR 参考的重点。

我们上文是通过视空间进行 RayMarching 的，三维空间有一个很大的性质就是近大远小，就是所谓的透视效果，那么我们在视空间步进一步，光栅化后对应到屏幕空间，可能就不一定是一个像素了。即在远处可能我们步进一步或者几步结果只对应到一个像素，这就浪费了计算；而到近处时，可能我们在视空间步进一个单位，就已经跨过好多个像素了，这又涉及到了采样不足，可能效果不好。如下图（图片来自上述论文）。

![](<images/1692014250850.png>)

为了解决以上的问题，屏幕空间进行 RayMarching 的方案就应运而生了，我们希望在屏幕空间可以保证沿着反射光线方向进行步进，不漏掉一个像素，也不重复采样一个像素。其实也就是光栅化的方式，之前在本人的软渲染实现过 Bresenham 算法，不过那是为使用整型并减少除法使之在 CPU 模拟计算时尽可能快速，实际上在 shader 上使用 GPU 再去实现，就没必要考虑整型的问题了，浮点型更快，因此可以直接就使用更加容易理解一些的画线算法 - DDA 算法，直接用斜率计算。

我们可以用最大步进距离，步进出发点和方向得到结束点。另外，我们需要剔除近裁剪面后面的点，并且可以直接剔除与视线方向一致的反射光线。既然涉及到屏幕空间计算了，首先需要将光线的出发点和结束点转化到屏幕空间（注意，并非 shader 中使用的 01 空间，而是真正的屏幕空间 <或者渲染 RT>，即乘以过屏幕 < 或者 RT > 长宽的屏幕空间）。可以将视空间的顶点用投影矩阵变换到裁剪空间，然后需要做透视除法，转换到 NDC 空间，再进行 * 0.5 + 0.5，转换到（0,1）屏幕区间，最后再乘以屏幕长宽，转换为真正的屏幕空间。我们可以暂时不考虑透视除法，使之仍然用齐次坐标表示，然后将着一些列变换封装到一个矩阵中，然后将矩阵与投影矩阵相乘，得到的结果矩阵就可以直接将视空间的顶点转换到屏幕空间了。直接用点乘以矩阵得到的还是齐次空间下的坐标。这个齐次坐标的 w 值，其实也就是视空间的 z 值非常重要。直接对屏幕空间齐次坐标除以 w，就得到了透视除法后的屏幕坐标。

得到了光线的起始点和结束点，我们就可以在屏幕空间进行光线的步进。这个过程与光栅化过程有些类似，即需要沿着光线的方向绘制一条直线，并且需要将一些信息进行插值计算。为何需要插值呢，很明显我们将计算转化到屏幕空间，是为了控制步进的距离，虽然这样做步进控制得比较好，但是我们需要这一点的深度信息来和当前深度图（存储的是视空间深度 <或 1/z>，取决于是 DepthNormalTexture 还是普通 DepthTexture）该像素的深度进行比较来判断是否发生了碰撞。每次步进再将其反推回视空间肯定不现实，那么就可以考虑类似光栅化中，我们在 vertex 阶段的输出值传递给 fragment 阶段时进行插值的做法，将起点和终点的视空间深度进行插值。不过有一点需要注意，这个深度并非直接与屏幕空间的步进值直接成正比，而是与屏幕空间步进值 * 1/z 成正比，也就是所谓的透视校正插值。而这个 z 也就是我们上文得到的 w 值。这样，得到每次步进前后的深度值，再与深度图中的值进行比较，就得到了是否碰撞的结果。

关于画线算法，坐标变换，透视投影校正插值等内容，可以参考本人之前的 [SoftRenderer 的 blog](https://blog.csdn.net/puppet_master/article/details/80317178)。

基本原理说到这里，上代码。

C# 部分如下，增加了从视空间直接转化到屏幕空间的变换矩阵：

```
/********************************************************************
 FileName: ScreenSpaceReflection.cs
 Description: 屏幕空间反射SSR，屏幕空间步进方式
 history: 10:9:2018 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{
    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 1024)]
    public int maxRayMarchingStep = 64;
    [Range(0, 32)]
    public int maxRayMarchingBinarySearchCount = 8;
    [Range(1, 10)]
    public int rayMarchingStepSize = 2;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;
    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
    }
 
    private void OnEnable()
    {
        currentCamera.depthTextureMode |= DepthTextureMode.DepthNormals;    
    }
 
    private void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.DepthNormals;
    }
 
    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }
 
        var width = source.width;
        var height = source.height;
        var screenSize = new Vector4(1.0f / width, 1.0f / height, width, height);
        var clipToScreenMatrix = new Matrix4x4();
        // (clip * 0.5 + 0.5)变换到screenspace，*width或height，得到真正的像素位置
        clipToScreenMatrix.SetRow(0, new Vector4(width * 0.5f, 0, 0, width * 0.5f));
        clipToScreenMatrix.SetRow(1, new Vector4(0, height * 0.5f, 0, height * 0.5f));
        clipToScreenMatrix.SetRow(2, new Vector4(0, 0, 1.0f, 0));
        clipToScreenMatrix.SetRow(3, new Vector4(0, 0, 0, 1.0f));
        var projectionMatrix = GL.GetGPUProjectionMatrix(currentCamera.projectionMatrix, false);
        var viewToScreenMatrix = clipToScreenMatrix * projectionMatrix;
        reflectionMaterial.SetMatrix("_ViewToScreenMatrix", viewToScreenMatrix); 
        reflectionMaterial.SetVector("_ScreenSize", screenSize);
 
        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);
 
        reflectionMaterial.SetFloat("_maxRayMarchingBinarySearchCount", maxRayMarchingBinarySearchCount);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        Graphics.Blit(source, destination, reflectionMaterial, 0);
    }
 
}
```

Shader 代码如下，改用屏幕空间步进的方式：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.9.10  
//Screen Space Reflection效果，屏幕空间步进方式
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			
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
				float4 vertex : SV_POSITION;
				float2 uv : TEXCOORD0; 
				float3 viewRay : TEXCOORD1;
			};
 
			sampler2D _MainTex;
			float4 _MainTex_ST;
			float4 _ScreenSize;//(1 / width, 1 / height, width, height)
			sampler2D _CameraDepthTexture;
			//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
			//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
			float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
			float4x4 _CameraProjectionMatrix;//unity_CameraProjection
			float4x4 _ViewToScreenMatrix;
			
			float _maxRayMarchingDistance;
			float _maxRayMarchingStep;
			float _maxRayMarchingBinarySearchCount;
			float _rayMarchingStepSize;
			float _depthThickness;
			
			sampler2D _CameraDepthNormalsTexture;
			
			void swap(inout float v0, inout float v1)
			{
				float temp = v0;
				v0 = v1;
				v1 = temp;
			}
			
			float distanceSquared(float2 A, float2 B)
			{
				A -= B;
				return dot(A, A);
			}
			
			bool screenSpaceRayMarching(float3 rayOri, float3 rayDir, inout float2 hitScreenPos)
			{
				//反方向反射的，本身也看不见，索性直接干掉
				if (rayDir.z > 0.0)
					return false;
				//首先求得视空间终点位置，不超过最大距离
				float magnitude = _maxRayMarchingDistance;
				float end = rayOri.z + rayDir.z * magnitude;
				//如果光线反过来超过了近裁剪面，需要截取到近裁剪面
				if (end > -_ProjectionParams.y)
					magnitude = (-_ProjectionParams.y - rayOri.z) / rayDir.z;
				float3 rayEnd = rayOri + rayDir * magnitude;
				//直接把cliptoscreen与projection矩阵结合，得到齐次坐标系下屏幕位置
				float4 homoRayOri = mul(_ViewToScreenMatrix, float4(rayOri, 1.0));
				float4 homoRayEnd = mul(_ViewToScreenMatrix, float4(rayEnd, 1.0));
				//w
				float kOri = 1.0 / homoRayOri.w;
				float kEnd = 1.0 / homoRayEnd.w;
				//屏幕空间位置
				float2 screenRayOri = homoRayOri.xy * kOri;
				float2 screenRayEnd = homoRayEnd.xy * kEnd;
				screenRayEnd = (distanceSquared(screenRayEnd, screenRayOri) < 0.0001) ? screenRayOri + float2(0.01, 0.01) : screenRayEnd;
				
				float3 QOri = rayOri * kOri;
				float3 QEnd = rayEnd * kEnd;
				
				float2 displacement = screenRayEnd - screenRayOri;
				bool permute = false;
				if (abs(displacement.x) < abs(displacement.y))
				{
					permute = true;
					
					displacement = displacement.yx;
					screenRayOri.xy = screenRayOri.yx;
					screenRayEnd.xy = screenRayEnd.yx;
				}
				float dir = sign(displacement.x);
				float invdx = dir / displacement.x;
				float2 dp = float2(dir, invdx * displacement.y);
				float3 dq = (QEnd - QOri) * invdx;
				float  dk = (kEnd - kOri) * invdx;
				float rayZmin = rayOri.z;
				float rayZmax = rayOri.z;
				float preZ = rayOri.z;
				
				float2 screenPoint = screenRayOri;
				float3 Q = QOri;
				float k = kOri;
				
				UNITY_LOOP
				for(int i = 0; i < _maxRayMarchingStep; i++)
				{
					//向前步进一个单位
					screenPoint += dp;
					Q.z += dq.z;
					k += dk;
					
					//得到步进前后两点的深度
					rayZmin = preZ;
					rayZmax = (dq.z * 0.5 + Q.z) / (dk * 0.5 + k);
					preZ = rayZmax;
					if (rayZmin > rayZmax)
					{
						swap(rayZmin, rayZmax);
					}
					
					//得到当前屏幕空间位置，交换过的xy换回来，并且根据像素宽度还原回（0,1）区间而不是屏幕区间
					hitScreenPos = permute ? screenPoint.yx : screenPoint;
					hitScreenPos *= _ScreenSize.xy;
					
					//转换回屏幕（0,1）区间，剔除出屏幕的反射
					if (any(hitScreenPos.xy < 0.0) || any(hitScreenPos.xy > 1.0))
						return false;
					
					//采样当前点深度图，转化为视空间的深度（负值）
					float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, hitScreenPos);
					float depth = -DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
					
					bool isBehand = (rayZmin <= depth);
					bool intersecting = isBehand && (rayZmax >= depth - _depthThickness);
					
					if (intersecting)
						return true;
				}
				return false;
			}
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.uv = TRANSFORM_TEX(v.uv, _MainTex);
				
				float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
				float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
				o.viewRay = viewRay.xyz / viewRay.w;
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				fixed4 mainTex = tex2D(_MainTex, i.uv);
				float linear01Depth;
				float3 viewNormal;
				
				float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
				DecodeDepthNormal(cdn, linear01Depth, viewNormal);
				//重建视空间坐标
				float3 viewPos = linear01Depth * i.viewRay;
				float3 viewDir = normalize(viewPos);
				viewNormal = normalize(viewNormal);
				//视空间方向反射方向
				float3 reflectDir = reflect(viewDir, viewNormal);
				float2 hitScreenPos = float2(-1,-1);
				//从该点开始RayMarching
				if (screenSpaceRayMarching(viewPos, reflectDir, hitScreenPos))
				{
					float4 reflectTex = tex2D(_MainTex, hitScreenPos);
					mainTex.rgb += reflectTex.rgb;
				}
				return mainTex;
			}
			
			ENDCG
		}
	}
}
```

没有 Binary，最大步进次数 450 效果如下：

![](<images/1692014250899.png>)

在屏幕空间步进的效果几乎没有重影，效果也能保证，然而最大步进次数着实让人有点害怕。我们可以尝试添加一些步长，比如我们在步进距离上乘以一个系数（1,10）正数，使之每次步进超过一个像素，加快碰撞过程：

```
float2 dp = float2(dir, invdx * displacement.y) * _rayMarchingStepSize;
float3 dq = (QEnd - QOri) * invdx * _rayMarchingStepSize;
float  dk = (kEnd - kOri) * invdx * _rayMarchingStepSize;
```

这样，我们步进 100 次，步长 4 的情况下效果也与之前差不多：

![](<images/1692014250964.png>)

但是如果再提高步长，效果就差了很多，尤其是在近处会出现断带，步进 20 次，步长 10 效果：

![](<images/1692014251015.png>)

远处效果勉强可以忍，近处基本忍不了了。下一步我们还是考虑可变步长，Binary Search 可以，也可以考虑在近处减少步长，在远处加大步长，不过我们此处选择一个更加好玩的方式，并且可以进一步减少步进次数。

### Dither RayMarching

这是一个老套路了，在 [GodRay](https://blog.csdn.net/puppet_master/article/details/79859678) 的 blog 里面就使用过这个套路，对于 RayMarching 类型的效果来说，Dither 确实是一个神级优化了。简单来说，Dither（Jitter）就是通过增加随机噪声来增加一些随机性，进而大幅度减少真正步进的次数。依然使用的是 KillZone 分享的那个 DitherMap，然后在 RayMarching 时，第一步增加一个随机的值。

![](<images/1692014251161.png>)

DitherMap 生成代码：

```
private Texture2D GenerateDitherMap()
    {
        int texSize = 4;
        var ditherMap = new Texture2D(texSize, texSize, TextureFormat.Alpha8, false, true);
        ditherMap.filterMode = FilterMode.Point;
        Color32[] colors = new Color32[texSize * texSize];
 
        colors[0] = GetDitherColor(0.0f);
        colors[1] = GetDitherColor(8.0f);
        colors[2] = GetDitherColor(2.0f);
        colors[3] = GetDitherColor(10.0f);
 
        colors[4] = GetDitherColor(12.0f);
        colors[5] = GetDitherColor(4.0f);
        colors[6] = GetDitherColor(14.0f);
        colors[7] = GetDitherColor(6.0f);
 
        colors[8] = GetDitherColor(3.0f);
        colors[9] = GetDitherColor(11.0f);
        colors[10] = GetDitherColor(1.0f);
        colors[11] = GetDitherColor(9.0f);
 
        colors[12] = GetDitherColor(15.0f);
        colors[13] = GetDitherColor(7.0f);
        colors[14] = GetDitherColor(13.0f);
        colors[15] = GetDitherColor(5.0f);
 
        ditherMap.SetPixels32(colors);
        ditherMap.Apply();
        return ditherMap;
    }
 
    private Color32 GetDitherColor(float value)
    {
        byte byteValue = (byte)(value / 16.0f * 255);
        return new Color32(byteValue, byteValue, byteValue, byteValue);
    }
```

Shader 中 RayMarching 第一步时增加 Dither 值：

```
//dither
float2 offsetUV = (fmod(floor(screenRayOri), 4.0));
float ditherValue = tex2D(_ditherMap, offsetUV / 4.0).a;
screenPoint += dp * ditherValue;
Q.z += dq.z * ditherValue;
k += dk * ditherValue;
```

依然是步进 20 次，步长为 10 的情况下，效果会好很多：

![](<images/1692014251208.png>)

但是使用 Dither 之后，会出现小格子。不过没有关系，Dither 可以和模糊配套使用，使用模糊去噪。简单来说就是 Jitter 工作流（只不过本人的这套比较简陋喽）。既然需要模糊了，我们就不能直接将结果输出了，我们可以申请两块中间 RT，用于将结果渲染到 RT 上，并且使用高斯模糊，然后再将结果与原始 RT 叠加得到最终结果。此外，如果不直接输出，我们在申请 RT 的时候就可以申请一张二分之一或者更小的 RT，来降低 RayMarching 的消耗，这也是各家 SSR 经常使用的一个优化（如 [KlayGE](http://www.klayge.org/2015/07/20/%E5%8A%A0%E9%80%9F%E5%8F%8D%E5%B0%84%E7%9A%84%E6%B8%B2%E6%9F%93/)），毕竟反射效果无需特别清晰，而且这个效果的逐像素计算消耗实在太大。

在 C# 中增加中间 RT，模糊操作，叠加操作：

```
/********************************************************************
 FileName: ScreenSpaceReflection.cs
 Description: 屏幕空间反射SSR，DitherRayMarching，延迟渲染下
 history: 30:8:2018 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{
    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 1024)]
    public int maxRayMarchingStep = 64;
    [Range(0, 32)]
    public int maxRayMarchingBinarySearchCount = 8;
    [Range(1, 10)]
    public int rayMarchingStepSize = 2;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;
 
    [Range(0, 3)]
    public int downSample = 1;
 
    [Range(0,3)]
    public int samplerScale = 1;
 
    private Texture2D ditherMap = null;
 
 
    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
    }
 
    private void OnEnable()
    {
        if (ditherMap == null)
            ditherMap = GenerateDitherMap();
        currentCamera.depthTextureMode |= DepthTextureMode.DepthNormals;
    }
 
    private void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.DepthNormals;
    }
 
    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }
 
        var width = source.width >> downSample;
        var height = source.height >> downSample;
        var screenSize = new Vector4(1.0f / width, 1.0f / height, width, height);
        var clipToScreenMatrix = new Matrix4x4();
        // (clip * 0.5 + 0.5)变换到screenspace，*width或height，得到真正的像素位置
        clipToScreenMatrix.SetRow(0, new Vector4(width * 0.5f, 0, 0, width * 0.5f));
        clipToScreenMatrix.SetRow(1, new Vector4(0, height * 0.5f, 0, height * 0.5f));
        clipToScreenMatrix.SetRow(2, new Vector4(0, 0, 1.0f, 0));
        clipToScreenMatrix.SetRow(3, new Vector4(0, 0, 0, 1.0f));
        var projectionMatrix = GL.GetGPUProjectionMatrix(currentCamera.projectionMatrix, false);
        var viewToScreenMatrix = clipToScreenMatrix * projectionMatrix;
        reflectionMaterial.SetMatrix("_ViewToScreenMatrix", viewToScreenMatrix); 
        reflectionMaterial.SetVector("_ScreenSize", screenSize);
 
        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);
        reflectionMaterial.SetMatrix("_WorldToCameraMatrix", currentCamera.worldToCameraMatrix);
 
        reflectionMaterial.SetFloat("_maxRayMarchingBinarySearchCount", maxRayMarchingBinarySearchCount);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        reflectionMaterial.SetTexture("_ditherMap", ditherMap);
 
        var reflectRT = RenderTexture.GetTemporary(width, height, 0, source.format);
        var tempBlurRT = RenderTexture.GetTemporary(width, height, 0, source.format);
        Graphics.Blit(source, reflectRT, reflectionMaterial, 0);
 
        //高斯模糊，两次模糊，横向纵向，使用pass1进行高斯模糊
        reflectionMaterial.SetVector("_offsets", new Vector4(0, samplerScale, 0, 0));
        Graphics.Blit(reflectRT, tempBlurRT, reflectionMaterial, 1);
        reflectionMaterial.SetVector("_offsets", new Vector4(samplerScale, 0, 0, 0));
        Graphics.Blit(tempBlurRT, reflectRT, reflectionMaterial, 1);
 
        //将反射贴图叠加到原图
        reflectionMaterial.SetTexture("_ReflectTex", reflectRT);
        Graphics.Blit(source, destination, reflectionMaterial, 2);
 
        RenderTexture.ReleaseTemporary(reflectRT);
        RenderTexture.ReleaseTemporary(tempBlurRT);
    }
 
 
    private Texture2D GenerateDitherMap()
    {
        int texSize = 4;
        var ditherMap = new Texture2D(texSize, texSize, TextureFormat.Alpha8, false, true);
        ditherMap.filterMode = FilterMode.Point;
        Color32[] colors = new Color32[texSize * texSize];
 
        colors[0] = GetDitherColor(0.0f);
        colors[1] = GetDitherColor(8.0f);
        colors[2] = GetDitherColor(2.0f);
        colors[3] = GetDitherColor(10.0f);
 
        colors[4] = GetDitherColor(12.0f);
        colors[5] = GetDitherColor(4.0f);
        colors[6] = GetDitherColor(14.0f);
        colors[7] = GetDitherColor(6.0f);
 
        colors[8] = GetDitherColor(3.0f);
        colors[9] = GetDitherColor(11.0f);
        colors[10] = GetDitherColor(1.0f);
        colors[11] = GetDitherColor(9.0f);
 
        colors[12] = GetDitherColor(15.0f);
        colors[13] = GetDitherColor(7.0f);
        colors[14] = GetDitherColor(13.0f);
        colors[15] = GetDitherColor(5.0f);
 
        ditherMap.SetPixels32(colors);
        ditherMap.Apply();
        return ditherMap;
    }
 
    private Color32 GetDitherColor(float value)
    {
        byte byteValue = (byte)(value / 16.0f * 255);
        return new Color32(byteValue, byteValue, byteValue, byteValue);
    }
 
}
```

增加了模糊 Pass 和叠加 Pass 的 shader：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.9.10  
//Screen Space Reflection效果，屏幕空间步进方式,DitherRayMarching
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	CGINCLUDE
 #include "UnityCG.cginc"
	
	struct appdata
	{
		float4 vertex : POSITION;
		float2 uv : TEXCOORD0;
	};
	
	struct v2f
	{
		float4 vertex : SV_POSITION;
		float2 uv : TEXCOORD0; 
		float3 viewRay : TEXCOORD1;
	};
	
	sampler2D _MainTex;
	float4 _MainTex_ST;
	float4 _ScreenSize;//(1 / width, 1 / height, width, height)
	sampler2D _CameraDepthTexture;
	sampler2D _ditherMap;
	//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
	//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
	float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
	float4x4 _CameraProjectionMatrix;//unity_CameraProjection
	float4x4 _ViewToScreenMatrix;
	
	float _maxRayMarchingDistance;
	float _maxRayMarchingStep;
	float _maxRayMarchingBinarySearchCount;
	float _rayMarchingStepSize;
	float _depthThickness;
	
	sampler2D _CameraDepthNormalsTexture;
	
	void swap(inout float v0, inout float v1)
	{
		float temp = v0;
		v0 = v1;
		v1 = temp;
	}
	
	float distanceSquared(float2 A, float2 B)
	{
		A -= B;
		return dot(A, A);
	}
	
	bool screenSpaceRayMarching(float3 rayOri, float3 rayDir, inout float2 hitScreenPos)
	{
		//反方向反射的，本身也看不见，索性直接干掉
		if (rayDir.z > 0.0)
			return false;
		//首先求得视空间终点位置，不超过最大距离
		float magnitude = _maxRayMarchingDistance;
		float end = rayOri.z + rayDir.z * magnitude;
		//如果光线反过来超过了近裁剪面，需要截取到近裁剪面
		if (end > -_ProjectionParams.y)
			magnitude = (-_ProjectionParams.y - rayOri.z) / rayDir.z;
		float3 rayEnd = rayOri + rayDir * magnitude;
		//直接把cliptoscreen与projection矩阵结合，得到齐次坐标系下屏幕位置
		float4 homoRayOri = mul(_ViewToScreenMatrix, float4(rayOri, 1.0));
		float4 homoRayEnd = mul(_ViewToScreenMatrix, float4(rayEnd, 1.0));
		//w
		float kOri = 1.0 / homoRayOri.w;
		float kEnd = 1.0 / homoRayEnd.w;
		//屏幕空间位置
		float2 screenRayOri = homoRayOri.xy * kOri;
		float2 screenRayEnd = homoRayEnd.xy * kEnd;
		screenRayEnd = (distanceSquared(screenRayEnd, screenRayOri) < 0.0001) ? screenRayOri + float2(0.01, 0.01) : screenRayEnd;
		
		float3 QOri = rayOri * kOri;
		float3 QEnd = rayEnd * kEnd;
		
		float2 displacement = screenRayEnd - screenRayOri;
		bool permute = false;
		if (abs(displacement.x) < abs(displacement.y))
		{
			permute = true;
			
			displacement = displacement.yx;
			screenRayOri.xy = screenRayOri.yx;
			screenRayEnd.xy = screenRayEnd.yx;
		}
		float dir = sign(displacement.x);
		float invdx = dir / displacement.x;
		//float stride = 2.0 - min(1.0, -rayOri * 0.01);
		float stride = _rayMarchingStepSize;
		
		float2 dp = float2(dir, invdx * displacement.y) * stride;
		float3 dq = (QEnd - QOri) * invdx * stride;
		float  dk = (kEnd - kOri) * invdx * stride;
		float rayZmin = rayOri.z;
		float rayZmax = rayOri.z;
		float preZ = rayOri.z;
		
		float2 screenPoint = screenRayOri;
		float3 Q = QOri;
		float k = kOri;
		
		float2 offsetUV = (fmod(floor(screenRayOri), 4.0));
		float ditherValue = tex2D(_ditherMap, offsetUV / 4.0).a;
		
		screenPoint += dp * ditherValue;
		Q.z += dq.z * ditherValue;
		k += dk * ditherValue;
		
		UNITY_LOOP
		for(int i = 0; i < _maxRayMarchingStep; i++)
		{
			//向前步进一个单位
			screenPoint += dp;
			Q.z += dq.z;
			k += dk;
			
			//得到步进前后两点的深度
			rayZmin = preZ;
			rayZmax = (dq.z * 0.5 + Q.z) / (dk * 0.5 + k);
			preZ = rayZmax;
			if (rayZmin > rayZmax)
			{
				swap(rayZmin, rayZmax);
			}
			
			//得到当前屏幕空间位置，交换过的xy换回来，并且根据像素宽度还原回（0,1）区间而不是屏幕区间
			hitScreenPos = permute ? screenPoint.yx : screenPoint;
			hitScreenPos *= _ScreenSize.xy;
			
			//转换回屏幕（0,1）区间，剔除出屏幕的反射
			if (any(hitScreenPos.xy < 0.0) || any(hitScreenPos.xy > 1.0))
				return false;
			
			//采样当前点深度图，转化为视空间的深度（负值）
			float4 depthnormalTex = tex2D(_CameraDepthNormalsTexture, hitScreenPos);
			float depth = -DecodeFloatRG(depthnormalTex.zw) * _ProjectionParams.z;
			
			bool isBehand = (rayZmin <= depth);
			bool intersecting = isBehand && (rayZmax >= depth - _depthThickness);
			
			if (intersecting)
				return true;
		}
		return false;
	}
	
	v2f vert_raymarching (appdata v)
	{
		v2f o;
		o.vertex = UnityObjectToClipPos(v.vertex);
		o.uv = TRANSFORM_TEX(v.uv, _MainTex);
		
		float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
		float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
		o.viewRay = viewRay.xyz / viewRay.w;
		return o;
	}
	
	fixed4 frag_raymarching (v2f i) : SV_Target
	{
		float linear01Depth;
		float3 viewNormal;
		
		float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
		DecodeDepthNormal(cdn, linear01Depth, viewNormal);
		//重建视空间坐标
		float3 viewPos = linear01Depth * i.viewRay;
		float3 viewDir = normalize(viewPos);
		viewNormal = normalize(viewNormal);
		//视空间方向反射方向
		float3 reflectDir = reflect(viewDir, viewNormal);
		float2 hitScreenPos = float2(-1,-1);
		fixed4 color = fixed4(0,0,0,1);
		
		//从该点开始RayMarching
		if (screenSpaceRayMarching(viewPos, reflectDir, hitScreenPos))
		{
			float4 reflectTex = tex2D(_MainTex, hitScreenPos);
			color.rgb += reflectTex.rgb;
		}
		return color;
	}
	
	
	//用于blur
	struct v2f_blur
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float4 uv01 : TEXCOORD1;
		float4 uv23 : TEXCOORD2;
		float4 uv45 : TEXCOORD3;
	};
 
	//用于叠加
	struct v2f_add
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float2 uv1 : TEXCOORD1;
	};
 
	float4 _MainTex_TexelSize;
	float4 _offsets;
 
	//高斯模糊 vert shader
	v2f_blur vert_blur(appdata_img v)
	{
		v2f_blur o;
		_offsets *= _MainTex_TexelSize.xyxy;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
 
		o.uv01 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1);
		o.uv23 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1) * 2.0;
		o.uv45 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1) * 3.0;
 
		return o;
	}
 
	//高斯模糊 pixel shader
	fixed4 frag_blur(v2f_blur i) : SV_Target
	{
		fixed4 color = fixed4(0,0,0,0);
		color += 0.40 * tex2D(_MainTex, i.uv);
		color += 0.15 * tex2D(_MainTex, i.uv01.xy);
		color += 0.15 * tex2D(_MainTex, i.uv01.zw);
		color += 0.10 * tex2D(_MainTex, i.uv23.xy);
		color += 0.10 * tex2D(_MainTex, i.uv23.zw);
		color += 0.05 * tex2D(_MainTex, i.uv45.xy);
		color += 0.05 * tex2D(_MainTex, i.uv45.zw);
		return color;
	}
	
	sampler2D _ReflectTex;
 
	fixed4 frag_add(v2f_add i) : SV_Target
	{
		fixed4 ori = tex2D(_MainTex, i.uv);
		fixed4 reflect = tex2D(_ReflectTex, i.uv);
		return ori + reflect;
	}
 
	
	ENDCG
	
	SubShader
	{
		//Pass0 : RayMarching
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
			CGPROGRAM
 #pragma vertex vert_raymarching
 #pragma fragment frag_raymarching
			ENDCG
			
		}
		
		//Pass1 : Blur
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vert_blur
 #pragma fragment frag_blur
			ENDCG
		}
 
		//pass 1: Add
		Pass
		{
 
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vert_img
 #pragma fragment frag_add
			ENDCG
		}
 
	}
}
```

20 次步进 + 10 步长 + Dither + Gaussian Blur 效果，反射不是很清晰，但是已经看不到噪点了：

![](<images/1692014251264.png>)

### 延迟渲染下的 SSR

前面的各种 SSR 都是在前向渲染的情况下实现的。虽然也可以凑合使用，然而也就到这里了。有一个很要命的问题，SSR 是一个全屏的后处理效果，我们没有办法区分哪里需要反射，哪里不需要反射。当然可以再去渲染一个 Mask 或者放到 Alpha 通道里面，但是毕竟不是一个很正统的做法。所以，还是在延迟下使用 SSR 会更加合适一些。

要改回延迟渲染，其实修改并不是很大。延迟渲染下默认就有 CameraDepthTexture，不过这个深度是非线性的，我们需要将其改为线性，才能正常使用；另外的一个必要条件是法线，在延迟渲染下可以通过_CameraGBufferTexture2 中得到法线，不过这个法线是世界空间的，我们需要通过世界到相机空间的逆矩阵将法线变换回法线空间。最后融合时，我们可以通过_CameraGBufferTexture1.a 通道，即 roughness 值与界面叠加融合。关于 Unity 延迟渲染的 GBuffer，可以参考[官方文档](https://docs.unity3d.com/Manual/RenderTech-DeferredShading.html)。

延迟渲染下 SSR 的 C# 代码：

```
/********************************************************************
 FileName: ScreenSpaceReflection.cs
 Description: 屏幕空间反射SSR，DitherRayMarching，延迟渲染下
 history: 30:8:2018 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using UnityEngine;
[ExecuteInEditMode]
public class ScreenSpaceReflection : MonoBehaviour
{
    Material reflectionMaterial = null;
    Camera currentCamera = null;
    [Range(0, 1000.0f)]
    public float maxRayMarchingDistance = 500.0f;
    [Range(0, 1024)]
    public int maxRayMarchingStep = 64;
    [Range(0, 32)]
    public int maxRayMarchingBinarySearchCount = 8;
    [Range(1, 50)]
    public int rayMarchingStepSize = 2;
    [Range(0, 2.0f)]
    public float depthThickness = 0.01f;
 
    [Range(0, 3)]
    public int downSample = 1;
 
    [Range(0,3)]
    public int samplerScale = 1;
 
    private Texture2D ditherMap = null;
 
 
    private void Awake()
    {
        var shader = Shader.Find("Reflection/ScreenSpaceReflection");
        reflectionMaterial = new Material(shader);
        currentCamera = GetComponent<Camera>();
        if (ditherMap == null)
            ditherMap = GenerateDitherMap();
    }
 
    private void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (reflectionMaterial == null)
        {
            Graphics.Blit(source, destination);
            return;
        }
 
        var width = source.width >> downSample;
        var height = source.height >> downSample;
        var screenSize = new Vector4(1.0f / width, 1.0f / height, width, height);
        var clipToScreenMatrix = new Matrix4x4();
        // (clip * 0.5 + 0.5)变换到screenspace，*width或height，得到真正的像素位置
        clipToScreenMatrix.SetRow(0, new Vector4(width * 0.5f, 0, 0, width * 0.5f));
        clipToScreenMatrix.SetRow(1, new Vector4(0, height * 0.5f, 0, height * 0.5f));
        clipToScreenMatrix.SetRow(2, new Vector4(0, 0, 1.0f, 0));
        clipToScreenMatrix.SetRow(3, new Vector4(0, 0, 0, 1.0f));
        var projectionMatrix = GL.GetGPUProjectionMatrix(currentCamera.projectionMatrix, false);
        var viewToScreenMatrix = clipToScreenMatrix * projectionMatrix;
        reflectionMaterial.SetMatrix("_ViewToScreenMatrix", viewToScreenMatrix); 
        reflectionMaterial.SetVector("_ScreenSize", screenSize);
 
        reflectionMaterial.SetMatrix("_InverseProjectionMatrix", currentCamera.projectionMatrix.inverse);
        reflectionMaterial.SetMatrix("_CameraProjectionMatrix", currentCamera.projectionMatrix);
        reflectionMaterial.SetMatrix("_WorldToCameraMatrix", currentCamera.worldToCameraMatrix);
 
        reflectionMaterial.SetFloat("_maxRayMarchingBinarySearchCount", maxRayMarchingBinarySearchCount);
        reflectionMaterial.SetFloat("_maxRayMarchingDistance", maxRayMarchingDistance);
        reflectionMaterial.SetFloat("_maxRayMarchingStep", maxRayMarchingStep);
        reflectionMaterial.SetFloat("_rayMarchingStepSize", rayMarchingStepSize);
        reflectionMaterial.SetFloat("_depthThickness", depthThickness);
        reflectionMaterial.SetTexture("_ditherMap", ditherMap);
 
        var reflectRT = RenderTexture.GetTemporary(width, height, 0, source.format);
        var tempBlurRT = RenderTexture.GetTemporary(width, height, 0, source.format);
        Graphics.Blit(source, reflectRT, reflectionMaterial, 0);
 
        //高斯模糊，两次模糊，横向纵向，使用pass1进行高斯模糊
        reflectionMaterial.SetVector("_offsets", new Vector4(0, samplerScale, 0, 0));
        Graphics.Blit(reflectRT, tempBlurRT, reflectionMaterial, 1);
        reflectionMaterial.SetVector("_offsets", new Vector4(samplerScale, 0, 0, 0));
        Graphics.Blit(tempBlurRT, reflectRT, reflectionMaterial, 1);
 
        //将反射贴图叠加到原图
        reflectionMaterial.SetTexture("_ReflectTex", reflectRT);
        Graphics.Blit(source, destination, reflectionMaterial, 2);
 
        RenderTexture.ReleaseTemporary(reflectRT);
        RenderTexture.ReleaseTemporary(tempBlurRT);
    }
 
 
    private Texture2D GenerateDitherMap()
    {
        int texSize = 4;
        var ditherMap = new Texture2D(texSize, texSize, TextureFormat.Alpha8, false, true);
        ditherMap.filterMode = FilterMode.Point;
        Color32[] colors = new Color32[texSize * texSize];
 
        colors[0] = GetDitherColor(0.0f);
        colors[1] = GetDitherColor(8.0f);
        colors[2] = GetDitherColor(2.0f);
        colors[3] = GetDitherColor(10.0f);
 
        colors[4] = GetDitherColor(12.0f);
        colors[5] = GetDitherColor(4.0f);
        colors[6] = GetDitherColor(14.0f);
        colors[7] = GetDitherColor(6.0f);
 
        colors[8] = GetDitherColor(3.0f);
        colors[9] = GetDitherColor(11.0f);
        colors[10] = GetDitherColor(1.0f);
        colors[11] = GetDitherColor(9.0f);
 
        colors[12] = GetDitherColor(15.0f);
        colors[13] = GetDitherColor(7.0f);
        colors[14] = GetDitherColor(13.0f);
        colors[15] = GetDitherColor(5.0f);
 
        ditherMap.SetPixels32(colors);
        ditherMap.Apply();
        return ditherMap;
    }
 
    private Color32 GetDitherColor(float value)
    {
        byte byteValue = (byte)(value / 16.0f * 255);
        return new Color32(byteValue, byteValue, byteValue, byteValue);
    }
 
}
```

延迟渲染下 SSR 的 Shader 代码：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.9.10  
//Screen Space Reflection效果，屏幕空间步进方式,DitherRayMarching
Shader "Reflection/ScreenSpaceReflection"
{
	Properties
	{
		_MainTex ("Texture", 2D) = "white" {}
	}
	
	CGINCLUDE
 #include "UnityCG.cginc"
	
	struct appdata
	{
		float4 vertex : POSITION;
		float2 uv : TEXCOORD0;
	};
	
	struct v2f
	{
		float4 vertex : SV_POSITION;
		float2 uv : TEXCOORD0; 
		float3 viewRay : TEXCOORD1;
	};
	
	sampler2D _MainTex;
	float4 _MainTex_ST;
	float4 _ScreenSize;//(1 / width, 1 / height, width, height)
	sampler2D _CameraDepthTexture;
	sampler2D _CameraGBufferTexture0;
    sampler2D _CameraGBufferTexture1;
    sampler2D _CameraGBufferTexture2;
    sampler2D _CameraGBufferTexture3;
	sampler2D _ditherMap;
	//使用外部传入的matrix，实际上等同于下面两个unity内置matrix
	//似乎在老版本unity中在pss阶段使用正交相机绘制Quad，矩阵会被替换，2017.3版本测试使用内置矩阵也可以
	float4x4 _InverseProjectionMatrix;//unity_CameraInvProjection
	float4x4 _CameraProjectionMatrix;//unity_CameraProjection
	float4x4 _WorldToCameraMatrix;
	float4x4 _ViewToScreenMatrix;
	
	float _maxRayMarchingDistance;
	float _maxRayMarchingStep;
	float _maxRayMarchingBinarySearchCount;
	float _rayMarchingStepSize;
	float _depthThickness;
	
	sampler2D _CameraDepthNormalsTexture;
	
	void swap(inout float v0, inout float v1)
	{
		float temp = v0;
		v0 = v1;
		v1 = temp;
	}
	
	float distanceSquared(float2 A, float2 B)
	{
		A -= B;
		return dot(A, A);
	}
	
	bool screenSpaceRayMarching(float3 rayOri, float3 rayDir, inout float2 hitScreenPos)
	{
		//反方向反射的，本身也看不见，索性直接干掉
		if (rayDir.z > 0.0)
			return false;
		//首先求得视空间终点位置，不超过最大距离
		float magnitude = _maxRayMarchingDistance;
		float end = rayOri.z + rayDir.z * magnitude;
		//如果光线反过来超过了近裁剪面，需要截取到近裁剪面
		if (end > -_ProjectionParams.y)
			magnitude = (-_ProjectionParams.y - rayOri.z) / rayDir.z;
		float3 rayEnd = rayOri + rayDir * magnitude;
		//直接把cliptoscreen与projection矩阵结合，得到齐次坐标系下屏幕位置
		float4 homoRayOri = mul(_ViewToScreenMatrix, float4(rayOri, 1.0));
		float4 homoRayEnd = mul(_ViewToScreenMatrix, float4(rayEnd, 1.0));
		//w
		float kOri = 1.0 / homoRayOri.w;
		float kEnd = 1.0 / homoRayEnd.w;
		//屏幕空间位置
		float2 screenRayOri = homoRayOri.xy * kOri;
		float2 screenRayEnd = homoRayEnd.xy * kEnd;
		screenRayEnd = (distanceSquared(screenRayEnd, screenRayOri) < 0.0001) ? screenRayOri + float2(0.01, 0.01) : screenRayEnd;
		
		float3 QOri = rayOri * kOri;
		float3 QEnd = rayEnd * kEnd;
		
		float2 displacement = screenRayEnd - screenRayOri;
		bool permute = false;
		if (abs(displacement.x) < abs(displacement.y))
		{
			permute = true;
			
			displacement = displacement.yx;
			screenRayOri.xy = screenRayOri.yx;
			screenRayEnd.xy = screenRayEnd.yx;
		}
		float dir = sign(displacement.x);
		float invdx = dir / displacement.x;
		//float stride = 2.0 - min(1.0, -rayOri * 0.01);
		float stride = _rayMarchingStepSize;
		
		float2 dp = float2(dir, invdx * displacement.y) * stride;
		float3 dq = (QEnd - QOri) * invdx * stride;
		float  dk = (kEnd - kOri) * invdx * stride;
		float rayZmin = rayOri.z;
		float rayZmax = rayOri.z;
		float preZ = rayOri.z;
		
		float2 screenPoint = screenRayOri;
		float3 Q = QOri;
		float k = kOri;
		
		float2 offsetUV = (fmod(floor(screenRayOri), 4.0));
		float ditherValue = tex2D(_ditherMap, offsetUV / 4.0).a;
		
		screenPoint += dp * ditherValue;
		Q.z += dq.z * ditherValue;
		k += dk * ditherValue;
		
		UNITY_LOOP
		for(int i = 0; i < _maxRayMarchingStep; i++)
		{
			//向前步进一个单位
			screenPoint += dp;
			Q.z += dq.z;
			k += dk;
			
			//得到步进前后两点的深度
			rayZmin = preZ;
			rayZmax = (dq.z * 0.5 + Q.z) / (dk * 0.5 + k);
			preZ = rayZmax;
			if (rayZmin > rayZmax)
			{
				swap(rayZmin, rayZmax);
			}
			
			//得到当前屏幕空间位置，交换过的xy换回来，并且根据像素宽度还原回（0,1）区间而不是屏幕区间
			hitScreenPos = permute ? screenPoint.yx : screenPoint;
			hitScreenPos *= _ScreenSize.xy;
			
			//转换回屏幕（0,1）区间，剔除出屏幕的反射
			if (any(hitScreenPos.xy < 0.0) || any(hitScreenPos.xy > 1.0))
				return false;
			
			//采样当前点深度图，转化为视空间的深度（负值）
			float depth = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, hitScreenPos);
			depth = -LinearEyeDepth(depth);
			
			bool isBehand = (rayZmin <= depth);
			bool intersecting = isBehand && (rayZmax >= depth - _depthThickness);
			
			if (intersecting)
				return true;
		}
		return false;
	}
	
	v2f vert_raymarching (appdata v)
	{
		v2f o;
		o.vertex = UnityObjectToClipPos(v.vertex);
		o.uv = TRANSFORM_TEX(v.uv, _MainTex);
		
		float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
		float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
		o.viewRay = viewRay.xyz / viewRay.w;
		return o;
	}
	
	fixed4 frag_raymarching (v2f i) : SV_Target
	{
		float4 z = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		float linear01Depth = Linear01Depth(z);
		
		float3 worldNormal = tex2D(_CameraGBufferTexture2, i.uv).rgb * 2.0 - 1.0;
		float3 viewNormal = mul((float3x3)(_WorldToCameraMatrix), worldNormal);
		
		//重建视空间坐标
		float3 viewPos = linear01Depth * i.viewRay;
		float3 viewDir = normalize(viewPos);
		viewNormal = normalize(viewNormal);
		//视空间方向反射方向
		float3 reflectDir = reflect(viewDir, viewNormal);
		float2 hitScreenPos = float2(-1,-1);
		fixed4 color = fixed4(0,0,0,1);
		
		//从该点开始RayMarching
		if (screenSpaceRayMarching(viewPos, reflectDir, hitScreenPos))
		{
			float4 reflectTex = tex2D(_MainTex, hitScreenPos);
			color.rgb += reflectTex.rgb;
		}
		return color;
	}
	
	
	//用于blur
	struct v2f_blur
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float4 uv01 : TEXCOORD1;
		float4 uv23 : TEXCOORD2;
		float4 uv45 : TEXCOORD3;
	};
 
	//用于叠加
	struct v2f_add
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float2 uv1 : TEXCOORD1;
	};
 
	float4 _MainTex_TexelSize;
	float4 _offsets;
 
	//高斯模糊 vert shader
	v2f_blur vert_blur(appdata_img v)
	{
		v2f_blur o;
		_offsets *= _MainTex_TexelSize.xyxy;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
 
		o.uv01 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1);
		o.uv23 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1) * 2.0;
		o.uv45 = v.texcoord.xyxy + _offsets.xyxy * float4(1, 1, -1, -1) * 3.0;
 
		return o;
	}
 
	//高斯模糊 pixel shader
	fixed4 frag_blur(v2f_blur i) : SV_Target
	{
		fixed4 color = fixed4(0,0,0,0);
		color += 0.40 * tex2D(_MainTex, i.uv);
		color += 0.15 * tex2D(_MainTex, i.uv01.xy);
		color += 0.15 * tex2D(_MainTex, i.uv01.zw);
		color += 0.10 * tex2D(_MainTex, i.uv23.xy);
		color += 0.10 * tex2D(_MainTex, i.uv23.zw);
		color += 0.05 * tex2D(_MainTex, i.uv45.xy);
		color += 0.05 * tex2D(_MainTex, i.uv45.zw);
		return color;
	}
	
	sampler2D _ReflectTex;
 
	fixed4 frag_add(v2f_add i) : SV_Target
	{
		fixed4 ori = tex2D(_MainTex, i.uv);
		fixed4 reflect = tex2D(_ReflectTex, i.uv);
		float s = tex2D(_CameraGBufferTexture1, i.uv).a;
		return ori + reflect * s;
	}
 
	
	ENDCG
	
	SubShader
	{
		//Pass0 : RayMarching
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
			CGPROGRAM
 #pragma vertex vert_raymarching
 #pragma fragment frag_raymarching
			ENDCG
			
		}
		
		//Pass1 : Blur
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vert_blur
 #pragma fragment frag_blur
			ENDCG
		}
 
		//pass 1: Add
		Pass
		{
 
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vert_img
 #pragma fragment frag_add
			ENDCG
		}
 
	}
}
```

最终效果来张动图：

![](<images/1692014251391.png>)

目前仅仅实现了最基本的 SSR，真正的 SSR 的进阶版有很多很多，比如保存采样点而非采样贴图，进行根据反射距离淡出的效果；增加模糊反射模拟真正 PBR 的效果，将不同级别的模糊图存在 Mip 中模拟粗糙度的问题；支持法线贴图；Hi-Z 优化等等。如果有精力以后再加啦。

PS：如果真想用 SSR 的话，Unity 官方提供的后处理包 PostProcessingStack 中已经带啦，而且也有[知乎上的大佬实现了更好的 SSR 效果](https://zhuanlan.zhihu.com/p/38303394)以及目前[最新的 SSR 方案](https://zhuanlan.zhihu.com/p/38528391)。膜拜一波这些大佬们。

## 总结

本篇 blog 主要是梳理了一下目前实时渲染中常见的一些反射技术，包括 CubeMap（Reflection Probe），Box Projected Cube Map，Planar Reflection，Screen Space Reflection。每种技术都有其优缺点。很多情况下，这些反射技术都会被结合使用，比如上面图中，动态的人物是使用 SSR 渲染的，而地面上的天空盒反射仍然是使用 Reflection Probe 进行渲染的。比如《Thief》当年也分享过一个关于反射的 PPT<很大，慎重>，主讲反射系统的构建，结合了很多很多种反射技术。