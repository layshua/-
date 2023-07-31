

## 简介

深度是个好东西哇，很多效果都需要深度，比如[景深](https://blog.csdn.net/puppet_master/article/details/52819874)，屏幕空间扫描效果，软粒子，阴影，SSAO，近似次表面散射（更确切的说是透射），对于延迟渲染来说，还可以用深度反推世界空间位置降低带宽消耗，还可以用深度做运动模糊，屏幕空间高度雾，距离雾，部分 [Ray-Marching 效果](https://blog.csdn.net/puppet_master/article/details/79859678)也都需要深度，可以说，深度是一些渲染高级效果必要的条件。另一方面，光栅化渲染本身可以得到正确的效果，就与深度（Z Buffer）有着密不可分的关系。

深度对于实时渲染的意义十分重大，OpenGL，DX，Unity 为我们封装好了很多深度相关的内容，如 ZTest，ZWrite，CameraDepthMode，Linear01Depth 等等。今天我来整理一下与学习过程中遇到的深度相关的一些问题，主要是渲染中深度的一些问题以及 Unity 中深度图生成，深度图的使用，深度的精度，Reverse-Z 等等问题，然后再用深度图，实现一些好玩的效果。本人才疏学浅，如果有不正确的地方，还望各位高手批评指正。

## 画家算法 & ZBuffer 算法

在渲染中为了保证渲染的正确，其实主要得益于两个最常用的算法，第一个是画家算法。所谓画家算法，就是按照画画的顺序，先画远处的内容，再画近处的内容叠加上去，近处的会覆盖掉远处的内容。即，在绘制之前，需要先按照远近排序。但是画家算法有一个很严重的问题，对于自身遮挡关系比较复杂的对象，没有办法保证绘制的正确；无法进行检测，overdraw 比较严重；再者对对象排序的操作，不适合硬件实现。

而另一种保证深度正确的算法就是 ZBuffer 算法，申请一块和 FrameBuffer 大小一样的缓冲区，如果开启了深度测试，那么在最终写入 FrameBuffer 之前（Early-Z 实现类似，只是时机效果不同），就需要进行测试，比如 ZTest LEqual 的话，如果深度小于该值，那么通过深度测试，如果开启了深度写入，还需要顺便更新一下当前点的深度值，如果不通过，就不会写入 FrameBuffer。ZBuffer 保证像素级别的深度正确，并且实现简单，比起靠三角形排序这种不确定性的功能更加容易硬件化，所以目前的光栅化渲染中大部分都使用的是 ZBuffer 算法。ZBuffer 算法也有坏处，第一就是需要一块和颜色缓冲区一样大小的 Buffer，精度还要比较高，所以比较费内存，再者需要逐像素计算 Z 值，但是为了渲染的正确，也就是透视校正纹理映射，Z 值的计算是不可避免的，所以总体来看，ZBuffer 的优势还是比较明显的。关于 ZBuffer 的实现以及透视投影纹理映射，可以参照[软渲染实现](https://blog.csdn.net/puppet_master/article/details/80317178)。

对于不透明物体来说，ZBuffer 算法是非常好的，可以保证遮挡关系没有问题。但是透明物体的渲染，由于一般是不写深度的，所以经常会出现问题，对于透明物体，一般还是采用画家算法，即由远及近进行排序渲染。还有一种方案是关闭颜色写入，先渲染一遍 Z 深度，然后再渲染半透，就可以避免半透明对象内部也被显示出来的问题，可以参考之前的遮挡处理这篇文章中[遮挡半透](https://blog.csdn.net/puppet_master/article/details/73478905)的做法。

## 透视投影与光栅化过程

透视投影的主要知识点在于三角形相似以及小孔呈像，透视投影实现的就是一种 “近大远小” 的效果，其实投影后的大小（x，y 坐标）也刚好就和 1/Z 呈线性关系。看下面一张图：

![[a9712b5b3bc24be6da79cac80505dcb5_MD5.png]]

上图是一个视锥体的截面图（只看 x，z 方向），P 为空间中一点（x，y，z），那么它在近裁剪面处的投影坐标假设为 P’(x'，y'，z’)，理论上来说，呈像的面应该在眼睛后方才更符合真正的小孔呈像原理，但是那样会增加复杂度，没必要额外引入一个负号（此处有一个裁剪的注意要点，下文再说），只考虑三角形相似即可。即三角形 EAP’相似于三角形 EGP，我们可以得到两个等式：

x’/ x = z’/ z => x’= xz’/ z  

y’/ y = z’/ z => y’= yz’/ z

由于投影面就是近裁剪面，那么近裁剪面是我们可以定义的，我们设其为 N，远裁剪面为 F，那么实际上最终的投影坐标就是：

（Nx/z，Ny/z，N）。

投影后的 Z 坐标，实际上已经失去作用了，只用 N 表示就可以了，但是这个每个顶点都一样，每个顶点带一个的话简直是暴殄天物，浪费了一个珍贵的维度，所以这个 Z 会被存储一个用于后续深度测试，透视校正纹理映射的变换后的 Z 值。

但是还有一个问题，这里我们得到是只是顶点的 Z 值，也就是我们在 vertex [shader](https://so.csdn.net/so/search?q=shader&spm=1001.2101.3001.7020) 中计算的结果，只有顶点，但是实际上，我们在屏幕上会看到无数的像素，换句话说，这些顶点的信息都是离散的，但是最终显示在屏幕上的模型却是连续的，这个那么每个像素点的值是怎么得到的呢？其实就是插值。一个三角形光栅化到屏幕空间上时，我们仅有的就是在三角形三个顶点所包含的各种数据，其中顶点已经是被变换过的了（Unity 中常用的 MVP 变换），在绘制三角形的过程中，根据屏幕空间位置对上述数据进行插值计算，来获得顶点之间对应屏幕上像素点上的颜色或其他数据信息。

这个 Z 值，还是比较有说道的。在透视投影变换之前，我们的 Z 实际上是相机空间的 Z 值，直接把这个 Z 存下来也无可厚非，但是后续计算会比较麻烦，毕竟没有一个统一的标准。既然我们有了远近裁剪面，有了 Z 值的上下限，我们就可以把这个 Z 值映射到 [0,1] 区间，即当在近裁剪面时，Z 值为 0，远裁剪面时，Z 值为 1（暂时不考虑 reverse-z 的情况）。

首先，能想到的最简单的映射方法就是 depth = （Z（eye） - N）/ F - N。直接线性映射到（0,1）区间，但是这种方案是不正确的，看下面一张图：

![[65434bdfd0ce54eb8547b994c146ba3b_MD5.png]]

右侧的三角形，在 AB 近裁剪面投影的大小一致，而实际上 C1F1 和 F1E1 相差的距离甚远，换句话说，经过投影变换的透视除法后，我们在屏幕空间插值的数据（根据屏幕空间距离插值），并不能保证其对应点在投影前的空间是线性变换的。关于透视投影和光栅化，可以参照上一篇文章中[软渲染透视投影和光栅化](https://blog.csdn.net/puppet_master/article/details/80317178)的内容。

透视投影变换之后，在屏幕空间进行插值的数据，与 Z 值不成正比，而是与 1/Z 成正比。所以，我们需要一个表达式，可以使 Z = N 时，depth = 0，Z = F 时，depth = 1，并且需要有一个 z 作为分母，可以写成（az + b）/z，带入上述两个条件：

（N * a  + b） / N = 0   =>  b = -an

（F * a  +  b） / F = 0   =>   aF + b = F => aF - aN = F

进而得到： 

a = F / （F - N）

b = NF / （N - F）

最终 depth（屏幕空间） = （aZ + b）/ Z （Z 为视空间深度）。

通过透视投影，在屏幕空间 X，Y 值都除以了 Z（视空间深度），当一个值的 Z 趋近于无穷远时，那么 X，Y 值就趋近于 0 了，也就是类似近大远小的效果了。而对于深度值的映射，从上面看也是除以了 Z 的，这个现象其实也比较好理解，比如一个人在离相机 200 米的地方前进了 1 米，我们基本看不出来距离的变化，但是如果在相机面前 2 米处前进了 1 米，那么这个距离变化是非常明显的，这也是近大远小的一种体现。

## Unity 中生成深度图

先来考古一下，我找到了一个上古时代的 Unity 版本，4.3，在 4.X 的时代，Unity 生成深度图使用的还是 Hidden/Camera-DepthTexture 这个函数，机制就是使用 Replacement Shader，在渲染时将 shader 统一换成 Hidden/Camera-DepthTexture，不同类型的 RenderType 对应不同的 SubShader，比如带有 Alpha Test 就可以在 fragment 阶段 discard 掉不需要的部分，防止在深度图中有不该出现的内容。那时候，也许还有些设备不支持原生的 DepthTexture RT 格式（SM2.0 以上，DepthTexture 支持）还有一个 UNITY_MIGHT_NOT_HAVE_DEPTH_TEXTURE 的宏，针对某些不支持深度格式的 RT，使用普通的 RGBA 格式编码深度图进行输出，采样时再将 RGBA 解码变回深度信息，使用编码的好处主要在于可以充分利用颜色的四个通道（32 位）获得更高的精度，否则就只有一个通道（8 位），编码和解码的函数如下：

```
// Encoding/decoding [0..1) floats into 8 bit/channel RGBA. Note that 1.0 will not be encoded properly.
inline float4 EncodeFloatRGBA( float v ) {
	float4 kEncodeMul = float4(1.0, 255.0, 65025.0, 160581375.0);
	float kEncodeBit = 1.0/255.0;
	float4 enc = kEncodeMul * v;
	enc = frac (enc);
	enc -= enc.yzww * kEncodeBit;
	return enc;
}
inline float DecodeFloatRGBA( float4 enc ) {
	float4 kDecodeDot = float4(1.0, 1/255.0, 1/65025.0, 1/160581375.0);
	return dot( enc, kDecodeDot );
}
```

深度图生成的函数如下，其实那时绝大多数情况都已经支持 DepthFormat 格式了，所以直接使用了空实现，颜色返回为 0：

```
#if defined(UNITY_MIGHT_NOT_HAVE_DEPTH_TEXTURE)
	#define UNITY_TRANSFER_DEPTH(oo) oo = o.pos.zw
	#if SHADER_API_FLASH
	#define UNITY_OUTPUT_DEPTH(i) return EncodeFloatRGBA(i.x/i.y)
	#else
	#define UNITY_OUTPUT_DEPTH(i) return i.x/i.y
	#endif
#else
	#define UNITY_TRANSFER_DEPTH(oo) 
	#define UNITY_OUTPUT_DEPTH(i) return 0
#endif
```

在 Unity5.X 版本后，实际上深度的 pass 就变为了 ShadowCaster 这个 pass，而不需要再进行 Shader Raplacement 的操作了（但是 DepthNormalMap 仍然需要），所谓 ShadowCaster 这个 pass，其实就是用于投影的 Pass，Unity 的所有自带 shader 都带这个 pass，而且只要我们 fallback 了 Unity 内置的 shader，也会增加 ShadowCaster 这个 pass。我们应该也可以自己定义 ShadowCaster 这个 pass，防止类似 AlphaTest 等造成深度图中内容与实际渲染内容不符的情况。

ShadowCaster 这个 pass 实际上是有两个用处，第一个是屏幕空间的深度使用该 pass 进行渲染，另一方面就是 ShadowMap 中光方向的深度也是使用该 pass 进行渲染的，区别主要在与 VP 矩阵的不同，阴影的 pass 是相对于光空间的深度，而屏幕空间深度是相对于摄像机的。新版的 Unity 使用了 ScreenSpaceShadowMap，屏幕空间的深度也是必要的（先生成 DpehtTexture，再生成 ShadowMap，然后生成 ScreenSpaceShadowMap，再正常渲染物体采样 ScreenSpaceShadowMap）。所以，如果我们开了屏幕空间阴影，再使用 DepthTexture，就相当于免费赠送，不用白不用喽。

新版本的 Unity，本人目前使用的是 Unity2017.3 版本，VS 和 PS 阶段的宏直接全部改为了空实现：

```
// Legacy; used to do something on platforms that had to emulate depth textures manually. Now all platforms have native depth textures.
#define UNITY_TRANSFER_DEPTH(oo)
// Legacy; used to do something on platforms that had to emulate depth textures manually. Now all platforms have native depth textures.
#define UNITY_OUTPUT_DEPTH(i) return 0
```

而 ShadowCaster 的实现也是颇为简单，VS 阶段不考虑 ShadowBias 的情况下其实就是 MVP 变换，而 PS 也直接是空实现：

```
/********************************************************************
 FileName: DepthTextureTest.cs
 Description:显示深度贴图
 Created: 2018/05/27
 history: 27:5:2018 1:25 by puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class DepthTextureTest : MonoBehaviour
{
 private Material postEffectMat = null;
 private Camera currentCamera = null;
 
 void Awake()
 {
 currentCamera = GetComponent<Camera>();
 }
 
 void OnEnable()
 {
 if (postEffectMat == null)
 postEffectMat = new Material(Shader.Find("DepthTexture/DepthTextureTest"));
 currentCamera.depthTextureMode |= DepthTextureMode.Depth;
 }
 
 void OnDisable()
 {
 currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
 }
 
 void OnRenderImage(RenderTexture source, RenderTexture destination)
 {
 if (postEffectMat == null)
 {
 Graphics.Blit(source, destination);
 }
 else
 {
 Graphics.Blit(source, destination, postEffectMat);
 }
 }
}
```

为何 Unity 会如此肆无忌惮，直接空实现我们就可以得到一张深度图呢？我们可以用 framedebugger 看到我们使用的深度图的格式实际上是 DepthFormat：

![[f2648fb9b166796554dff633af075f8f_MD5.png]]

正如上文中 Untiy 新版本 shader 注释中所说的，“现在所有平台都支持原生的深度图了 "，所以也就没有必要再 RGBA 格式编码深度然后在进行解码这种费劲的方法，直接申请 DepthFormat 格式的 RT 即可，也就是在采样时，只将 DepthAttachment 的内容作为 BindTexture 的 id。ColorBuffer 输出的颜色是什么都无所谓了，我们要的只是 DepthBuffer 的输出，而这个输出的结果就是正常我们渲染时的深度，也就是 ZBuffer 中的深度值，只是这个值系统自动帮我们处理了，类似固定管线 Native 方式。

至于 Unity 为何不直接用 FrameBuffer 中的 Z，而是使用全场景渲染一遍的方式，个人猜测是为了更好的兼容性吧（如果有大佬知道，还望不吝赐教），再者本身一张 RT 同时读写在某些平台就是未定义的操作，可能出现问题（本人之前测试是移动平台上大部分都挂了，这也是为什么很多后处理，比如[高斯模糊](https://blog.csdn.net/puppet_master/article/details/52783179)等在申请 RT 的时候要申请两块，在两块 RT 之间互相 Blit 的原因）。倒是之前了解过一个黑科技，直接 bind 一张 RT 的 DepthAttachment 到 depth 上，然后读这张 RT 就是深度了，然而没有大面积真机测试过，真是不太敢用 。

## 深度图的使用

大概了解了一下 Unity 中深度图的由来，下面准备使用深度图啦。虽然前面说了这么多，然而实际上在 Unity 中使用深度图，却是一个简单到不能再简单的操作了，通过 Camera 的 depthTextureMode 即可设置 DepthTexture。我们来用一个后处理效果把当前的深度图绘制到屏幕上：

```
//puppet_master 
//2018.5.27 
//显示深度贴图
Shader "DepthTexture/DepthTextureTest" 
{
	CGINCLUDE
	#include "UnityCG.cginc"
	sampler2D _CameraDepthTexture;
	
	fixed4 frag_depth(v2f_img i) : SV_Target
	{
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		//float linear01EyeDepth = LinearEyeDepth(depthTextureValue) * _ProjectionParams.w;
		float linear01EyeDepth = Linear01Depth(depthTextureValue);
		return fixed4(linear01EyeDepth, linear01EyeDepth, linear01EyeDepth, 1.0);
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
			#pragma vertex vert_img
			#pragma fragment frag_depth
			ENDCG
		}
	}
}
```

Shader 部分：

```
float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
return 1 - depthTextureValue;
```

依然是我最常用的测试场景，哇咔咔，场景原始效果如下：

![[8f8fc6eb9136e037efed1c8240faeb89_MD5.png]]

显示深度效果如下：

![[9c57d8fb4f255c787ef674096b12bfaf_MD5.png]]

上面的 Shader 中我们使用了 SAMPLE_DEPTH_TEXTURE 这个宏进行了深度图的采样，其实这个宏就是采样了 DepthTexuter 的 r 通道作为深度（除在 PSP2 平台不一样），其余平台的定义都是下面的：

```
// Z buffer to linear depth
inline float LinearEyeDepth( float z ) {
    return 1.0 / (_ZBufferParams.z * z + _ZBufferParams.w);
}
 
// Values used to linearize the Z buffer (http://www.humus.name/temp/Linearize%20depth.txt)
// x = 1-far/near
// y = far/near
// z = x/far
// w = y/far
float4 _ZBufferParams;
```

## LinearEyeDepth&Linear01Depth

在上面的 Shader 中，我们使用了 LinearEyeDepth 和 LinearDepth 对深度进行了一个变换之后才输出到屏幕，那么实际上的 Z 值应该是啥样的呢，我放置了四个距离相等的模型，来看一下常规的 Z 值直接输出的情况（由于目前开启了 Reverse-Z，所以用 1-z 作为输出），即：

```
// Z buffer to linear 0..1 depth
inline float Linear01Depth( float z ) {
    return 1.0 / (_ZBufferParams.x * z + _ZBufferParams.y);
}
 
// Values used to linearize the Z buffer (http://www.humus.name/temp/Linearize%20depth.txt)
// x = 1-far/near
// y = far/near
// z = x/far
// w = y/far
float4 _ZBufferParams;
```

效果：

![[90c02dd0fe389bcc1b1794c5b05c1214_MD5.png]]

经过 Linear01Depth 变换后的效果：

![[b951af733abdba2ccee84a76edbe9ef1_MD5.png]]

对比两张图我们应该也就比较清楚效果了，没有经过处理的深度，在视空间上不是线性变化的，近处深度变化较明显，而远处几乎全白了，而经过处理的深度，在视空间是线性变化的。

为什么会这样呢，还是得从透视投影和光栅化说起，在视空间，每个顶点的原始的 Z 值是视空间的深度，但是经过透视投影变换以及透视投影，转化到屏幕空间后，需要保证在屏幕空间的深度与 1/z 成正比才可以在屏幕空间逐像素地进行插值进而获得屏幕上任意一点像素的屏幕空间深度值，简单来说，这个转化的过程主要是为了从顶点数据获得屏幕空间任意一点的逐像素数据。而得到屏幕空间深度之后，我们要使用时，经过变换的这个屏幕空间的东西，又不是很直观，最直观的还是视空间的深度，所以我们要进行一步变换，把屏幕空间的深度再转换回原始的视空间深度。

上文中，我们推导过从视空间深度转化到屏幕空间深度的公式如下：

a = F / （F - N）

b = NF / （N - F）

depth（屏幕空间） = （aZ + b）/ Z （Z 为视空间深度）。

那么，反推回 Z（视空间） = b /（depth - a），进一步地，Z（视空间） = 1 / （depth / b - a / b），然后将上述 a 和 b 的值代入：

Z（视空间） = 1 / （（depth / （NF / （N - F）） - （F /（F - N）） / （NF / （N - F）））

化简： Z（视空间） = 1 / (（（N - F）/ NF） * depth + 1 / N）

Z（视空间） = 1 / （param1 * depth + param2），param1 = （N - F）/ NF，param2 = 1 / N。

下面让我们来看看 Unity 自带 Shader 中关于深度值 LinearEyeDepth 的处理：

```
v2f vert( appdata_base v ) 
{
    v2f o;
    o.pos = UnityObjectToClipPos(v.vertex);
    o.nz.xyz = COMPUTE_VIEW_NORMAL;
    o.nz.w = COMPUTE_DEPTH_01;
    return o;
}
 
fixed4 frag(v2f i) : SV_Target 
{
    return EncodeDepthNormal (i.nz.w, i.nz.xyz);
}
 
#define COMPUTE_DEPTH_01 -(UnityObjectToViewPos( v.vertex ).z * _ProjectionParams.w)
 
// x = 1 or -1 (-1 if projection is flipped)
// y = near plane
// z = far plane
// w = 1/far plane
uniform vec4 _ProjectionParams;
```

_ZBufferParams.z = _ZBufferParams.x / far = （1 - far / near）/ far = （near - far） / near * far

_ZBufferParams.w = _ZBufferParams.y / far = （far / near） / far = 1 / near

我们推导的 param1 = _ZBufferParams.z，param2 = _ZBufferParams.w，实际上 Unity 中 LinearEyeDepth 就是将透视投影变换的公式反过来，用 zbuffer 图中的屏幕空间 depth 反推回当前像素点的相机空间深度值。

下面再来看一下 Linear01Depth 函数，所谓 01，其实也比较好理解，我们上面得到的深度值实际上是真正的视空间 Z 值，但是这个值没有一个统一的比较标准，所以这个时候依然秉承着映射大法好的理念，把这个值转化到 01 区间即可。由于相机实际上可以看到的最远区间就是 F（远裁剪面），所以这个 Z 值直接除以 F 即可得到映射到（0,1）区间的 Z 值了：

Z（视空间 01） = Z（视空间） / F = 1 / (（（N - F）/ N） * depth + F / N）

Z（视空间 01） = 1 / （param1 * depth + param2），param1 = （N - F）/ N = 1 - F/N，param2 = F / N。

再来看一下 Unity 中关于 Linear01Depth 的处理：

```
// Depth/normal sampling functions
float SampleDepth(float2 uv) {
#if defined(SOURCE_GBUFFER) || defined(SOURCE_DEPTH)
    float d = LinearizeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, uv));
#else
    float4 cdn = tex2D(_CameraDepthNormalsTexture, uv);
    float d = DecodeFloatRG(cdn.zw);
#endif
    return d * _ProjectionParams.z + CheckBounds(uv, d);
}
```

可以看出我们推导的 param1 = _ZBufferParams.x，param2 = _ZBufferParams.y。也就是说，Unity 中 Linear01Depth 的操作值将屏幕空间的深度值还原为视空间的深度值后再除以远裁剪面的大小，将视空间深度映射到（0,1）区间。

Unity 应该是 OpenGL 风格（矩阵，NDC 等），上面的推导上是基于 DX 风格的 DNC 进行的，不过，如果是深度图的话，不管怎么样都会映射到（0，1）区间的，相当于 OpenGL 风格的深度再进行一步映射，就与 DX 风格的一致了。个人感觉 OpenGL 风格的 NDC 在某些情况下并不是很方便（见下文 Reverse-Z 相关内容）。

了解了这两个 Unity 为我们提供的 API 具体是干什么的了之后，我们就可以放心大胆的使用了，因为实际上绝大多数情况下，我们都是需要相机空间的深度值或者映射到 01 区间的相机空间深度值。

## Z&1/Z

通过上面的深度图具体的使用，我们发现，实际上真正使用的深度，是从顶点的视空间 Z，经过投影变成一个 1/Z 成正比的值（屏幕空间 Depth），然后在使用时，再通过投影变换时的计算公式反推回对应视空间像素位置的 Z。可见，这个操作还是非常折腾的。那为何要如此费劲地进行上面的操作，而不是直接存一个视空间的值作为真正的深度呢？

其实前辈们也想过这个问题，原来的显卡，甚至是不用我们当今的 Z Buffer（存储的是屏幕空间的 Depth，也就是与 1/Zview 成正比的一个值）的，而是用了一个所谓的 W Buffer（存储的是视空间的 Z）。W Buffer 的计算表面上看起来应该是很简单的，即在顶点计算时，直接将当前顶点的 z 值进行进行 01 映射，类似 W = Zview / Far，就可以了把视空间的值映射到一个（0,1）区间的深度值。然后我们在 Pixel 阶段要使用的时候，就需要通过光栅化阶段顶点数据插值得到当前屏幕空间这一点的 Z 值，但是这又回到了一个问题，Z 值是视空间的，经过了透视投影变换之后变成了屏幕空间，我们插值的系数是屏幕空间位置，这个位置是与 1/Z 成正比的，换句话说，在屏幕空间插值时，必须要进行透视投影校正，类似透视投影校正纹理采样，针对的是 uv 坐标进行了插值，大致思路是计算时对顶点数据先除以 Z，然后屏幕空间逐像素插值，之后再乘回该像素真正的 Z 值。可见，如果要使用这样的 W Buffer，虽然我们使用起来简单了，但是硬件实现上，还是比较麻烦的，毕竟需要多做一次乘除映射。

所以，实际上，现在的 Z Buffer 使用的仍然是屏幕空间的 Depth，也就是在透视投影变换时，使用透视投影矩阵直接相乘把顶点坐标 xyz 变换到齐次裁剪空间，然后统一透视除法除以 w，就得到了一个在屏幕空间是线性的 Depth 值。这个值可以在屏幕上直接根据像素位置进行简单线性插值，无需再进行透视校正，这样的话，对于硬件实现上来说是最容易的。

其实在 Unity 中也是分为两种 DepthTexture 的，一种是 DepthTexture，存储的是屏幕空间线性深度，也是最常见的深度的格式，上面已经推导过了。而另一种是 DepthNormalTexture（不仅仅是它除了 Depth 还包含 Normal），存的就是相机空间的深度值，这个就是最基本的线性映射，把这个值作为顶点数据走透视投影校正后传递给 Fragment 阶段，那么这个值其实直接就是在视空间是线性变换的了，不需要再进行类似普通 DepthTexture 的 Linear 操作。

DepthNormalTexture 的生成用到的相关内容（只看 Depth 部分）：

```
// Values used to linearize the Z buffer (http://www.humus.name/temp/Linearize%20depth.txt)
// x = 1-far/near
// y = far/near
// z = x/far
// w = y/far
// or in case of a reversed depth buffer (UNITY_REVERSED_Z is 1)
// x = -1+far/near
// y = 1
// z = x/far
// w = 1/far
float4 _ZBufferParams;
```

新版本后处理包中对于深度的采样大概是这个样子的：

```
#if defined(SHADER_API_D3D11) || defined(SHADER_API_PSSL) || defined(SHADER_API_XBOXONE) || defined(SHADER_API_METAL) || defined(SHADER_API_VULKAN) || defined(SHADER_API_SWITCH)
// D3D style platforms where clip space z is [0, 1].
#define UNITY_REVERSED_Z 1
#endi
```

可见，对于两种方式的深度，进行生成和采样的方式是不同的，DepthNormal 类型的深度直接就可以乘以远裁剪面还原到视空间深度，而深度图的需要进行 Linearize 变换。可以根据需要定制自己的 DepthTexture。主流一些的方式还是原生的 DepthTextue 方式，但是这种方式也有很一个很严重的问题，就是精度问题。

## ZBuffer 的精度问题

既然说了 1/Z 的好处，那再来看看 1/Z 的坏处。ZBuffer 的这种设计可能会导致远处深度精度不够，进而会出现 ZFighting 的现象，前辈们一直在用各种方式来与 ZBuffer 的精度做着斗争，我们也来看看这个问题。由于使用 1/Z 作为深度，深度的分布是不均匀的，以一个 4Bit 的深度缓存来看的话，Z 值和 Depth 的关系如下图：

![[4963f943ac89f69e6fd99729ba9591a5_MD5.png]]

4Bit 的深度的精度可以表示 2^4 也就是 1/16 的精度，但是由上图可以看出，在 Z（相机空间）从 near 到 far 变化时，在 near 处精度很密集，而在 Z 超过 1/2(far - near) 这一段，几乎只有几个格子来表示这一段的精度了，也就是说，即使两个对象在远处离得很远，可能在深度 Buffer 里面二者也是归为深度相同的，那么在进行深度测试时，两个物体深度相同，两者的像素就都可能出现在前面，概率性地遮挡和不遮挡，就形成了 ZFighting 的现象。

要想缓解 ZFighting，首先看一下深度表示的公式，以 n 位精度的深度来说，每一位的精度表示如下：

D（perBit） = （1<<n） * （aZ + b ） / Z = （1 <<n） * （a + b  / z）

让 D（preBit）更小就是精度更高， 如果是线性深度，理论上我们的 depth 应该是 d = （Z - N）/ （F - N）插值，由于一般而言 F 远远大于 N，所以实际上影响因子主要在于 F。但是与 1/Z 作为插值的话，大概可以这样理解 d = （1/Z - 1/N）/ （1/F - 1/N），在这种情况下，实际上就是倒数影响了，那主要影响的因子实际上是 N。

我们看一下近裁剪面对 ZFighting 的影响，看来 Unity 为了缓解 ZFighting，近裁剪面最近只能设置为 0.01（本人 Unity 版本 2017.3），没办法看，所以这里用了之前的软渲染做测试了。

正常渲染的情况下（近裁剪面 0.1）：

![[7d1b3e09b95b3db0759373172f67808d_MD5.gif]]

出现 ZFighting 的情况（近裁剪面 0.00001f），在立方体的棱角位置出现了 ZFighting：

![[a4d266298d3d13b985d02e5011b8b4ff_MD5.gif]]

一个有效缓解 ZFighting 的方案就是尽可能远地放置近裁剪面（保证面前内容表现效果的情况下，太远会裁掉面前的东西）。将近裁剪面推远后的深度分布曲线如下， 可见，深度的分布曲线在远处被 “提起来”，也就是远处获得了更大一些的精度分布：

![[7a5cf32ed47a9a61e28affed43bb165a_MD5.png]]

目前正常 ZBuffer 的方式，简单来说就是近处对象的深度精度极高，远处对象的深度精度极地，差了 N 个数量级。其实正常来说，这样的深度分布也是有好处的，因为我们在近处的精度高一些，远处精度低点，感觉也比较符合正常思维。如果只是为了保证近处渲染的效果，那么直接用正常的 ZBuffer 就是最好的选择了。但是，主要就在于超大视距，类似超大地图这种，既需要保证远处的精度，又希望保证近处的精度，远处精度衰减太厉害，所以 ZFighting 现象就出现了。

本人所了解过的缓解 ZFighting 的方案主要是下面几种（如果还有好玩的方案还望不吝赐教）：

1）提高深度 Buffer 的精度，精度高了，自然表现效果就好了。在渲染到 RT 上时，经常出现 ZFighting 的现象，16Bit 满足不了效果的情况下选择 24Bit 深度。

2）尽可能远地放置相机的近裁剪面。

3）对于特别近的两个对象，适当考虑把二者之间的距离拉开一点，比如地面上的贴片，适当抬起来一点点（很无脑，但是最有效）。

4）对于实在有问题的情况，可以考虑 Offset 操作。本人曾经遇到在魅族 MX4 机型上渲染半透 Prepass 之后，半透的 Pass 和 Prepass 深度冲突，后来无奈给半透的 Pass 增加了一个 Offset 解决问题（比较特殊情况，只有这个机型很多效果都不对，简直给我带来了无尽的烦恼）。

5）动态切换远近裁剪面，即先设置很远的近裁剪面和远裁剪面，渲染远景物体，然后 ClearDepthBuffer 保留 ColorBuffer，修改近裁剪面到很低的值，远裁剪面到刚才设置的近裁剪面值，再渲染近处物体。这个方案在分界处交叉的物体可能有问题，不过这个问题影响不大，主要是这样会导致 Early-Z 失效，先渲后面的再渲染前面的，成了画家算法，至少一遍 overdarw。一般来说对于不透明物体的渲染顺序应该还是先渲染近处的，再渲染远处的物体（比如 Unity）<个人感觉不太实用，目前 Unity 内不太好实现，这招是 Ogre 里面一个哥们分享的，但是方案比较好玩，可能是本渣渣没做过什么大世界，没有什么超大视距和近处细节同时兼顾的需求，所以没有被精度逼到这种程度吧，万一逼急了，谁管他 overdraw 呢？>。

6）不写硬件深度，直接写视空间深度，换句话说就是正常的线性深度，类似生成 DepthNormalMap 的方式。

7）Logarithmic Depth Buffer，对数深度。与上一条类似，都是自己生成深度图。貌似 GTA5 中延迟渲染中生成深度的流程，就是自己算了一个对数深度，极大地提高了深度的精度，真是一切为了精度啊。比较复杂的玩法，没有玩过，Unity 目前应该也不支持，需要 pixel shader 里面对深度进行校正再写回，应该也会导致 Early-Z 失效，只能使用最终的 ZCheck，不过延迟渲染会好很多。

8）最后还有一种能够有效缓解 ZFighting 的方法，就是 Reverse-Z，这个 Unity 目前在一部分平台 <OpenGL ES 木有，已哭瞎> 已经自带了。

## Reverse-Z

这一部分先贴出一篇 [Nvidia 的关于 Reverse-Z 的文章](https://developer.nvidia.com/content/depth-precision-visualized)（本文中深度精度分布图来自该文章），里面讲的很详细。

所谓 Reverse-Z，直接翻译过来的意思是反转 Z。顾名思义的话，ZBuffer（深度图）中存储的值是反过来的，也就是近裁剪面的深度值实际上是 1，而远裁剪面的深度值是 0。那么，我们的 ZTest LEqual 就得当做 ZTest GEqual 来处理，采样的深度值都需要用 1 - depth 喽，为何会做出如此反人道的设计呢？还是得从 ZBuffer 本身的存储 1/Z 的设定以及浮点数精度的问题着手。

上面我们看到了用定点精度表示深度的分布曲线，那么，如果改为浮点数，理论上浮点数可以表示的范围要远大于定点数，是否会对其有所缓解呢？额，关于具体原因，还是看[这位大佬分析的浮点数精度相关的 blog](https://blog.csdn.net/dreamer2020/article/details/24158303) 吧（当年上计算机组成原理的时候，貌似我还在沉迷 COC，不过好在没挂科，233）。简单总结一下就是：浮点虽然表示的范围广，但是有精度损失，一个浮点数表示的其实是其周围的一个有理数区间，这个区间在 0 点处精度很高，而当浮点数本身很大时，根据科学计数法，小数部分乘以阶码表示最终的这个值，阶码越大，最终结果里面可以表示的真正小数的位数就变少，甚至没有了，所以浮点数的精度分布大概是酱紫的：

![[14d415a46d99bcac79c0c825f4c12e54_MD5.png]]

如果我们用浮点数表示精度的话，精度的曲线如下：

![[2dd146982ba6139a10b47cfc3df4f1dd_MD5.png]]

尴尬，浮点精度虽然高了，但是还是都集中在了近裁剪面，本身这个地方精度已经够高了，再高的话就是浪费了。于是乎前辈们就想到了一个非常巧妙的方法，既然浮点精度和 ZBuffer 精度都是在近裁剪面精度高，浮点精度我们没办法控制（IEEE 标准就这样的），那就只能在 ZBuffer 的生成上做文章了。固定流水线的话，不好控制，但是目前基本都是可编程流水线，矩阵是自己传给 shader 的，那么只要把上面的投影矩阵改一下，让近裁剪面的深度置为 1，远裁剪面的深度置为 0，这样这个 d = a + b/z 的变换执行了一个相当于反向映射的操作，也就成了所谓的 Reverse-Z。通过这个操作，把浮点数在 0 附近精度高抵消了深度远裁剪面精度低问题，使整体的深度 Buffer 精度有了较大的提高，使用了 Reverse-Z 的深度分布如下图：

![[f1d0231b93c954f67599be603c3750d0_MD5.png]]

Reverse-Z 的好处是提升了深度精度，坏处的话。。个人感觉应该就是不太好理解咯。主要的操作在于替换投影矩阵，深度映射提取时需要反向，ZTest 全部反过来看，DepthClear 需要修改。D3D 的话，NDC 是的 Z 是在 01 区间比较好实现，但是 OpenGL 的话，NDC 的 Z 是在 - 1,1 区间，这个值需要映射到 01 区间，需要有 glClipControl 强行设置远近裁剪面倒置。相当于多折腾一步映射，这个设定是在需要用 Reverse-Z 的情况还有写入深度图的时候都需要进行 01 映射，貌似在网上也看到不少人吐槽 OpenGL 强迫症地设计一个正方体的 NDC。

Unity 在 5.5 之后的版本里面，开始使用了 Reverse-Z。不过，Unity 封装得比较好，以至于一般情况下我们是不会发现问题的，Unity 大法好啊。在上面推导了 LinearEyeDepth 和 Linear01Depth 两个函数的实现，但是还是建议使用 Unity 的 API 来进行这个变换，因为 Unity 不仅为我们封装了上面的变换，可以很方便地使用，还有一个更重要的问题，就是 Unity 帮我们处理了 Reverse-Z 的情况，我们自己如果不处理的话，得到的深度实际上是反向的，因为 DepthBufferParam 这个值在是否开启 Reverse-Z 的情况下，从引擎传过来的值是不一样的，完整版本的如下：

```
// Unity built-in shader source. Copyright (c) 2016 Unity Technologies. MIT license (see license.txt)
 
Shader "Particles/Additive (Soft)" {
Properties {
    _MainTex ("Particle Texture", 2D) = "white" {}
    _InvFade ("Soft Particles Factor", Range(0.01,3.0)) = 1.0
}
 
Category {
    Tags { "Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent" "PreviewType"="Plane" }
    Blend One OneMinusSrcColor
    ColorMask RGB
    Cull Off Lighting Off ZWrite Off
 
    SubShader {
        Pass {
 
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma target 2.0
            #pragma multi_compile_particles
            #pragma multi_compile_fog
 
            #include "UnityCG.cginc"
 
            sampler2D _MainTex;
            fixed4 _TintColor;
 
            struct appdata_t {
                float4 vertex : POSITION;
                fixed4 color : COLOR;
                float2 texcoord : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };
 
            struct v2f {
                float4 vertex : SV_POSITION;
                fixed4 color : COLOR;
                float2 texcoord : TEXCOORD0;
                UNITY_FOG_COORDS(1)
                #ifdef SOFTPARTICLES_ON
                float4 projPos : TEXCOORD2;
                #endif
                UNITY_VERTEX_OUTPUT_STEREO
            };
 
            float4 _MainTex_ST;
 
            v2f vert (appdata_t v)
            {
                v2f o;
                UNITY_SETUP_INSTANCE_ID(v);
                UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(o);
                o.vertex = UnityObjectToClipPos(v.vertex);
                #ifdef SOFTPARTICLES_ON
				//计算顶点在屏幕空间的位置（没有进行透视除法）
                o.projPos = ComputeScreenPos (o.vertex);
				//计算顶点距离相机的距离
                COMPUTE_EYEDEPTH(o.projPos.z);
                #endif
                o.color = v.color;
                o.texcoord = TRANSFORM_TEX(v.texcoord,_MainTex);
                UNITY_TRANSFER_FOG(o,o.vertex);
                return o;
            }
 
            UNITY_DECLARE_DEPTH_TEXTURE(_CameraDepthTexture);
            float _InvFade;
 
            fixed4 frag (v2f i) : SV_Target
            {
                #ifdef SOFTPARTICLES_ON
				//根据上面的屏幕空间位置，进行透视采样深度图（tex2dproj，即带有透视除法的采样，相当于tex2d（xy/w）），
				//得到当前像素对应在屏幕深度图的深度，并转化到视空间，线性化（深度图中已有的不透明对象的深度）
                float sceneZ = LinearEyeDepth (SAMPLE_DEPTH_TEXTURE_PROJ(_CameraDepthTexture, UNITY_PROJ_COORD(i.projPos)));
                //本像素点在视空间真正的距离（粒子本身的深度）
				float partZ = i.projPos.z;
				//计算二者的深度差，该值越小，说明越近穿插
                float fade = saturate (_InvFade * (sceneZ-partZ));
				//上面的深度差调整粒子的alpha值
                i.color.a *= fade;
                #endif
 
                half4 col = i.color * tex2D(_MainTex, i.texcoord);
                col.rgb *= col.a;
                UNITY_APPLY_FOG_COLOR(i.fogCoord, col, fixed4(0,0,0,0)); // fog towards black due to our blend mode
                return col;
            }
            ENDCG
        }
    }
}
}
```

通过修改 x，y 的值，LinearEyeDepth 和 Linear01Depth 最终对于是否 Reverse-Z 都能得到正确的深度结果。

Unity 关于 Reverse-Z 的其他部分主要在于 MotionVector 的生成，阴影的计算等地方有区别，如果自己用深度计算的时候，可能也需要考虑一下这个问题。最后再来看看哪些平台开了这个宏：

```
//puppet_master
//https://blog.csdn.net/puppet_master
//2018.5.27 
//基于深度的扫描效果
Shader "DepthTexture/ScreenDepthScan" 
{
	Properties
	{
		_MainTex("Base (RGB)", 2D) = "white" {}
	}
	
	CGINCLUDE
	#include "UnityCG.cginc"
	sampler2D _CameraDepthTexture;
	sampler2D _MainTex;
	fixed4 _ScanLineColor;
	float _ScanValue;
	float _ScanLineWidth;
	float _ScanLightStrength;
	
	float4 frag_depth(v2f_img i) : SV_Target
	{
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		float linear01EyeDepth = Linear01Depth(depthTextureValue);
		fixed4 screenTexture = tex2D(_MainTex, i.uv);
		
		if (linear01EyeDepth > _ScanValue && linear01EyeDepth < _ScanValue + _ScanLineWidth)
		{
			return screenTexture * _ScanLightStrength * _ScanLineColor;
		}
		return screenTexture;
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
			#pragma vertex vert_img
			#pragma fragment frag_depth
			ENDCG
		}
	}
}
```

很遗憾，GLES 没开（GL Core4.5 原生支持，老版本 GL 的需要用扩展，ES 压根没提），不过 Mental 和 DX11 开了。所以设备上出现某些深度表现和 DX11 表现不同相，可以往这个方向考虑一下（老版本插件升级到 5.5 之后有可能会出现这个问题）。

所以这个问题告诉我们一个道理：能用官方 API，就用官方 API，即使知道 API 的实现，也尽量别自己造轮子，造轮子神马的是学习的时候用的，除非没官方轮子，工程里如果不用官方 API，纯属为后续升版本挖坑，逆着 Unity 干，一般没啥好结果（恩，说的就是我这个渣渣，之前没少干这种坏事，皮了一下很开心，升级时候改到死。不过，如果您是图形学或者 Unity 大佬的话，那还是想怎么玩就怎么玩）。

基于 Reverse-Z，后续又有人发现了一些减少深度计算误差的方法，比如用无穷远的远裁剪面以及把投影矩阵单独拆开来与顶点相乘（个人感觉会略微损失一点性能吧），可以参考[这篇论文](http://www.geometry.caltech.edu/pubs/UD12.pdf)。

## 软粒子效果

上面看过了深度相关的基本知识，下面就到了基本的效果实践了。第一个，也是一个比较常见的深度的应用就是软粒子效果。何谓软，何谓硬，看一下下面的一张截图：

![[33e01f352d91078361eca85ce521b81a_MD5.png]]

左侧为普通的粒子效果，而右侧为开启了软粒子的粒子效果。普通的粒子效果，和非透明的地面穿插时，是直接硬插进地面了，而右侧的软粒子效果，越靠近地面，粒子的 alpha 权重越低，到地面的时候就透明了，可见，软粒子相比于普通粒子能够更好地做到和非半透对象平滑过渡，不至于有明显的穿插。

下面看一下软粒子的实现，由于这个 Unity 是内置了这个效果，所以我就直接找到软粒子的 shader 源码添加点注释喽：

```
/********************************************************************
 FileName: ScreenDepthScan.cs
 Description:深度扫描线效果
 Created: 2018/05/27
 history: 27:5:2018 1:25 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class ScreenDepthScan : MonoBehaviour
{
    private Material postEffectMat = null;
    private Camera currentCamera = null;
 
    [Range(0.0f, 1.0f)]
    public float scanValue = 0.05f;
    [Range(0.0f, 0.5f)]
    public float scanLineWidth = 0.02f;
    [Range(0.0f, 10.0f)]
    public float scanLightStrength = 10.0f;
    public Color scanLineColor = Color.white;
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/ScreenDepthScan"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            //限制一下最大值，最小值
            float lerpValue = Mathf.Min(0.95f, 1 - scanValue);
            if (lerpValue < 0.0005f)
                lerpValue = 1;
            
            //此处可以一个vec4传进去优化
            postEffectMat.SetFloat("_ScanValue", lerpValue);
            postEffectMat.SetFloat("_ScanLineWidth", scanLineWidth);
            postEffectMat.SetFloat("_ScanLightStrength", scanLightStrength);
            postEffectMat.SetColor("_ScanLineColor", scanLineColor);
            Graphics.Blit(source, destination, postEffectMat);
        }
        
    }
}
```

基本思想就是，在渲染粒子效果时，先取当前屏幕空间深度图对应该像素点的深度值，然后计算该粒子对应该像素点位置的深度值（二者都转化到了视空间），然后用两个深度差作为一个系数调制粒子的 alpha 值，最终达到让粒子接近不透明物体的部分渐变淡出的效果。

上面的函数中有使用了一个这样的宏，通过该宏直接把顶点转化到视空间，取 z 值的负数就是真正的视空间距离了。

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.6.10  
//基于深度的扫描效果，附带扭曲
Shader "DepthTexture/ScreenDepthScanWithDistort" 
{
	Properties
	{
		_MainTex("Base (RGB)", 2D) = "white" {}
	}
	
	CGINCLUDE
 #include "UnityCG.cginc"
	sampler2D _CameraDepthTexture;
	sampler2D _MainTex;
	fixed4 _ScanLineColor;
	float _ScanValue;
	float _ScanLineWidth;
	float _ScanLightStrength;
	float _DistortFactor;
	float _DistortValue;
	
	float4 frag_depth(v2f_img i) : SV_Target
	{
		
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		float linear01EyeDepth = Linear01Depth(depthTextureValue);
		
		float2 dir = i.uv - float2(0.5, 0.5); 
		float2 offset = _DistortFactor * normalize(dir) * (1 - length(dir)); 
		float2 uv = i.uv - offset * _DistortValue * linear01EyeDepth; 
		fixed4 screenTexture = tex2D(_MainTex, uv); 
		if (linear01EyeDepth > _ScanValue && linear01EyeDepth < _ScanValue + _ScanLineWidth)
		{
			return screenTexture * _ScanLightStrength * _ScanLineColor;
		}
		return screenTexture;
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vert_img
 #pragma fragment frag_depth
			ENDCG
		}
	}
}
```

软粒子效果，虽然使用了深度图，但是比较麻烦，需要各种坐标转换，因为要在对象空间使用屏幕空间的深度，所以不得不 ComputeScreenPos，并 tex2Dproj，非常折腾，这也足以见得软粒子有多费，不仅仅在于渲染深度图本身的消耗，自身计算也是非常费的，再加上粒子一般都是半透，不写深度，没办法在粒子之间通过 early-z 优化，导致 overdraw 非常高，逐像素计算爆炸。看着为了这个渐变导致的这个计算量，我感觉移动上，粒子穿插还是忍了吧，万一有美术同学问我，我就假装不知道 -_-。

其实 Unity 的软粒子这套写法，可以用在不少其他效果中，比如水面，海边等根据深度渐变的效果（刷顶点色或许更省一些），我不只在一个水插件中看到上面的这套写法了，变量名都一样，今天才算是找到 “始作俑者”，哈哈哈（额，这好像是个贬义词，我特意百度了一下，实在没找到啥别的词儿，我没有贬义的意思哈。。。看来我的语文是百度老师教的）。

## 基于深度的扫描波效果

下面来搞个很简单，但是很好玩的效果，这个效果没有乱七八糟的 ComputeScreenPos 之类的，直接就是在屏幕空间进行的，恩，也就是我最爱的后处理啦，开心！

这个效果即 blog 开头《恶灵附身》截图的第一张图的类似效果。先观察一下，基本效果就是一个高亮的区域，按照深度由远及近地运动，直到略过摄像机。恩，我直接简单粗暴地在 shader 里判断了一下：

```
/********************************************************************
 FileName: ScreenDepthScan.cs
 Description:深度扫描线效果，附带扭曲
 Created: 2018/06/10
 history: 10:6:2018 10:25 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class ScreenDepthScanWithDistort : MonoBehaviour
{
    private Material postEffectMat = null;
    private Camera currentCamera = null;
 
    [Range(0.0f, 1.0f)]
    public float scanValue = 0.05f;
    [Range(0.0f, 0.5f)]
    public float scanLineWidth = 0.02f;
    [Range(0.0f, 10.0f)]
    public float scanLightStrength = 10.0f;
    [Range(0.0f, 0.04f)]
    public float distortFactor = 0.02f;
    public Color scanLineColor = Color.white;
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/ScreenDepthScanWithDistort"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            //限制一下最大值，最小值
            float lerpValue = Mathf.Min(0.95f, 1 - scanValue);
            if (lerpValue < 0.0005f)
                lerpValue = 1;
 
            //此处可以一个vec4传进去优化
            postEffectMat.SetFloat("_ScanValue", lerpValue);
            postEffectMat.SetFloat("_ScanLineWidth", scanLineWidth);
            postEffectMat.SetFloat("_ScanLightStrength", scanLightStrength);
            postEffectMat.SetFloat("_DistortFactor", distortFactor);
            postEffectMat.SetFloat("_DistortValue", 1 - scanValue);
            postEffectMat.SetColor("_ScanLineColor", scanLineColor);
            Graphics.Blit(source, destination, postEffectMat);
        }
 
    }
}
```

C# 代码如下，一些边界条件的判断放在 c# 里面，要比 shader 全屏计算效率好得多：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.6.10 
//打印对象在世界空间位置
Shader "DepthTexture/WorldPosPrint"
{
	SubShader
	{
		Tags { "RenderType"="Opaque" }
		LOD 100
 
		Pass
		{
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
				float3 worldPos : TEXCOORD0;
				float4 vertex : SV_POSITION;
			};
			
			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.worldPos = mul(unity_ObjectToWorld, v.vertex);
				return o;
			}
			
			fixed4 frag (v2f i) : SV_Target
			{
				return fixed4(i.worldPos, 1.0);
			}
			ENDCG
		}
	}
	//fallback使之有shadow caster的pass
	FallBack "Legacy Shaders/Diffuse"
}
```

效果如下：

![[d23522377f23fbab42ac6593ddb5c375_MD5.gif]]

当然，如果为了好玩的话，可以再加点别的效果烘托一下分为，比如结合一下[时空扭曲效果](https://blog.csdn.net/puppet_master/article/details/71437031)：

shader 代码：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.6.10 
//通过逆矩阵的方式从深度图构建世界坐标
Shader "DepthTexture/ReconstructPositionInvMatrix" 
{
	CGINCLUDE
	#include "UnityCG.cginc"
	sampler2D _CameraDepthTexture;
	float4x4 _InverseVPMatrix;
	
	fixed4 frag_depth(v2f_img i) : SV_Target
	{
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		//自己操作深度的时候，需要注意Reverse_Z的情况
		#if defined(UNITY_REVERSED_Z)
		depthTextureValue = 1 - depthTextureValue;
		#endif
		float4 ndc = float4(i.uv.x * 2 - 1, i.uv.y * 2 - 1, depthTextureValue * 2 - 1, 1);
		
		float4 worldPos = mul(_InverseVPMatrix, ndc);
		worldPos /= worldPos.w;
		return worldPos;
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
			#pragma vertex vert_img
			#pragma fragment frag_depth
			ENDCG
		}
	}
}
```

C# 代码：

```
/********************************************************************
 FileName: ReconstructPositionInvMatrix.cs
 Description:从深度图构建世界坐标，逆矩阵方式
 Created: 2018/06/10
 history: 10:6:2018 13:09 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class ReconstructPositionInvMatrix : MonoBehaviour {
 
    private Material postEffectMat = null;
    private Camera currentCamera = null;
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/ReconstructPositionInvMatrix"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            var vpMatrix = currentCamera.projectionMatrix * currentCamera.worldToCameraMatrix;
            postEffectMat.SetMatrix("_InverseVPMatrix", vpMatrix.inverse);
            Graphics.Blit(source, destination, postEffectMat);
        }
    }
}
```

效果：

![[ee508c0391b6894eccbc9eff71b991e1_MD5.gif]]

个人感觉扭曲和深度重建某些情况下是冲突的，如果仔细观察其实可能会发现图片有重影，但是，鉴于扫描速度很快，这点穿帮其实应该还是可以接受的。这不由得让我想起了做剧情的时候，给剧情做了不少效果，但是策划妹纸特别爱用震屏，景深，径向模糊这几个效果，我十分不解，后来我才知道其中缘由：“我们有好多地方都有穿帮，震一下，或者模糊一下，玩家就不注意了”，正所谓天下武功唯快不破，哇咔咔。

## 根据深度重建世界坐标

下面打算再用深度做几个更好玩的效果。但是这几个效果略微有些复杂，主要就在于不仅仅需要的是深度信息，还需要得到世界坐标的信息，也就是说我需要根据深度图反推当前世界坐标位置。

### 证明世界坐标重建正确的方法

首先，得先找到一种证明反推回世界空间位置正确的方法。这里，我在相机前摆放几个物体，尽量使之在世界坐标下的位置小于 1，方便判定颜色如下图：

![[c16943bf91525b8c359b3f35b7b80884_MD5.png]]

然后将几个物体的 shader 换成如下的一个打印世界空间位置的 shader：

```
float depth;
float3 normal;
 
float4 cdn = tex2D(_CameraDepthNormalsTexture, i.uv);
DecodeDepthNormal(cdn, depth, normal);
 
//float depth = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
//逆矩阵的方式使用的是1/z非线性深度，而_CameraDepthNormalsTexture中的是线性的，进行一步Linear01Depth的逆运算
depth = (1.0/depth - _ZBufferParams.y) /_ZBufferParams.x ;
//自己操作深度的时候，需要注意Reverse_Z的情况
#if defined(UNITY_REVERSED_Z)
depth = 1 - depth;
#endif
 
float4 ndc = float4(i.uv.x * 2 - 1, i.uv.y * 2 - 1, depth * 2 - 1, 1);
float4 worldPos = mul(_InverseVPMatrix, ndc);
worldPos /= worldPos.w;
```

然后挂上上面的重建世界坐标位置的脚本，在开启和关闭脚本前后，屏幕输出完全无变化，说明通过后处理重建世界坐标位置与直接用 shader 输出世界坐标位置效果一致：

![[51abec5f071cfe1f281b12f6b291cba1_MD5.png]]

### 逆矩阵方式重建

深度重建有几种方式，先来看一个最简单粗暴，但是看起来最容易理解的方法：

我们得到的屏幕空间深度图的坐标，xyz 都是在（0,1）区间的，需要经过一步变换，变换到 NDC 空间，OpenGL 风格的话就都是（-1,1）区间，所以需要首先对 xy 以及 xy 对应的深度 z 进行 * 2 - 1 映射。然后再将结果进行 VP 的逆变换，就得到了世界坐标。

shader 代码如下：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.6.16  
//通过深度图重建世界坐标，视口射线插值方式
Shader "DepthTexture/ReconstructPositionViewPortRay" 
{
	CGINCLUDE
 #include "UnityCG.cginc"
	sampler2D _CameraDepthTexture;
	float4x4 _ViewPortRay;
	
	struct v2f
	{
		float4 pos : SV_POSITION;
		float2 uv : TEXCOORD0;
		float4 rayDir : TEXCOORD1;
	};
	
	v2f vertex_depth(appdata_base v)
	{
		v2f o;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
		
		//用texcoord区分四个角，就四个点，if无所谓吧
		int index = 0;
		if (v.texcoord.x < 0.5 && v.texcoord.y > 0.5)
			index = 0;
		else if (v.texcoord.x > 0.5 && v.texcoord.y > 0.5)
			index = 1;
		else if (v.texcoord.x < 0.5 && v.texcoord.y < 0.5)
			index = 2;
		else
			index = 3;
		
		o.rayDir = _ViewPortRay[index];
		return o;
		
	}
	
	fixed4 frag_depth(v2f i) : SV_Target
	{
		
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		float linear01Depth = Linear01Depth(depthTextureValue);
		//worldpos = campos + 射线方向 * depth
		float3 worldPos = _WorldSpaceCameraPos + linear01Depth * i.rayDir.xyz;
		return fixed4(worldPos, 1.0);
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vertex_depth
 #pragma fragment frag_depth
			ENDCG
		}
	}
}
```

C# 部分：

```
/********************************************************************
 FileName: ReconstructPositionViewPortRay.cs
 Description:通过深度图重建世界坐标，视口射线插值方式
 Created: 2018/06/16
 history: 16:6:2018 16:17 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class ReconstructPositionViewPortRay : MonoBehaviour {
 
    private Material postEffectMat = null;
    private Camera currentCamera = null;
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/ReconstructPositionViewPortRay"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            var aspect = currentCamera.aspect;
            var far = currentCamera.farClipPlane;
            var right = transform.right;
            var up = transform.up;
            var forward = transform.forward;
            var halfFovTan = Mathf.Tan(currentCamera.fieldOfView * 0.5f * Mathf.Deg2Rad);
 
            //计算相机在远裁剪面处的xyz三方向向量
            var rightVec = right * far * halfFovTan * aspect;
            var upVec = up * far * halfFovTan;
            var forwardVec = forward * far;
 
            //构建四个角的方向向量
            var topLeft = (forwardVec - rightVec + upVec);
            var topRight = (forwardVec + rightVec + upVec);
            var bottomLeft = (forwardVec - rightVec - upVec);
            var bottomRight = (forwardVec + rightVec - upVec);
 
            var viewPortRay = Matrix4x4.identity;
            viewPortRay.SetRow(0, topLeft);
            viewPortRay.SetRow(1, topRight);
            viewPortRay.SetRow(2, bottomLeft);
            viewPortRay.SetRow(3, bottomRight);
 
            postEffectMat.SetMatrix("_ViewPortRay", viewPortRay);
            Graphics.Blit(source, destination, postEffectMat);
        }
    }
}
```

效果如下，重建 ok：

![[f67cca779e9af6f80e0a8999a27985f1_MD5.png]]

看起来比较简单，但是其中有一个 / w 的操作，如果按照正常思维来算，应该是先乘以 w，然后进行逆变换，最后再把 world 中的 w 抛弃，即是最终的世界坐标，不过实际上投影变换是一个损失维度的变换，我们并不知道应该乘以哪个 w，所以实际上上面的计算，并非按照理想的情况进行的计算，而是根据计算推导而来（更加详细推导请参考[这篇文章](http://feepingcreature.github.io/math.html)，不过我感觉这个推导有点绕）。

已知条件（M 为 VP 矩阵，M^-1 即为其逆矩阵，Clip 为裁剪空间，ndc 为标准设备空间，world 为世界空间）：

ndc = Clip.xyzw / Clip.w = Clip / Clip.w

world = M^-1 * Clip

二者结合得：

world = M ^-1 * ndc * Clip.w

我们已知 M 和 ndc，然而还是不知道 Clip.w，但是有一个特殊情况，是 world 的 w 坐标，经过变换后应该是 1，即

1 = world.w = （M^-1 * ndc）.w * Clip.w

进而得到 Clip.w = 1 / （M^ -1 * ndc）.w

带入上面等式得到：

world = （M ^ -1 * ndc） / （M ^ -1 * ndc）.w

所以，世界坐标就等于 ndc 进行 VP 逆变换之后再除以自身的 w。

不过这种方式重建世界坐标，性能比较差，一般来说，我们都是逐顶点地进行矩阵运算，毕竟定点数一般还是比较少的，但是全屏幕逐像素进行矩阵运算，这个计算量就不是一般的大了，性能肯定是吃不消的。

补充：如果是 DepthNormalTexture 中的 depth 通过逆矩阵方式重建，计算方式略有不同：

```
v2f vert (appdata v)
{
	float4 clipPos = float4(v.uv * 2 - 1.0, 1.0, 1.0);
	float4 viewRay = mul(_InverseProjectionMatrix, clipPos);
	o.viewRay = viewRay.xyz / viewRay.w;
	return o;
}
 
fixed4 frag (v2f i) : SV_Target
{
	float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
	float linear01Depth = Linear01Depth(depthTextureValue);
	float3 viewPos = _WorldSpaceCameraPos.xyz + linear01Depth * i.viewRay;
	
}
```

### 屏幕射线插值方式重建

这种方式的重建，可以参考 Secrets of CryENGINE 3 Graphics Technology 这个 CryTech 2011 年的 PPT。借用一张图：

![[0f6bb116d0e435570f5c683519427b97_MD5.png]]

然后偶再画个平面的图：

![[bc5b55cd3d24d0ca9b0ead43b1a8c48c_MD5.png]]

上图中，A 为相机位置，G 为空间中我们要重建的一点，那么该点的世界坐标为 A（worldPos） + 向量 AG，我们要做的就是求得向量 AG 即可。根据三角形相似的原理，三角形 AGH 相似于三角形 AFC，则得到 AH / AC = AG / AF。由于三角形相似就是比例关系，所以我们可以把 AH / AC 看做 01 区间的比值，那么 AC 就相当于远裁剪面距离，即为 1，AH 就是我们深度图采样后变换到 01 区间的深度值，即 Linear01Depth 的结果 d。那么，AG = AF * d。所以下一步就是求 AF，即求出相机到屏幕空间每个像素点对应的射线方向。看到上面的立体图，其实我们可以根据相机的各种参数，求得视锥体对应四个边界射线的值，这个操作在 vertex 阶段进行，由于我们的后处理实际上就是渲染了一个 Quad，上下左右四个顶点，把这个射线传递给 pixel 阶段时，就会自动进行插值计算，也就是说在顶点阶段的方向值到 pixel 阶段就变成了逐像素的射线方向。

那么我们要求的其实就相当于 AB 这条向量的值，以上下平面为例，三维向量只比二维多一个维度，我们已知远裁剪面距离 F，相机的三个方向（相机 transform.forward，.right，.up），AB = AC + CB，|BC| = tan(0.5fov) * |AC|，|AC| = Far，AC = transorm.forward * Far，CB = transform.up * tan(0.5fov) * Far。

我直接使用了远裁剪面对应的位置计算了三个方向向量，进而组合得到最终四个角的向量。用远裁剪面的计算代码比较简单（恩，我懒），不过《ShaderLab 入门精要》中使用的是近裁剪面 + 比例计算，不确定是否有什么考虑（比如精度，没有测出来，如果有大佬知道，还望不吝赐教）。

shader 代码如下：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.6.16  
//屏幕空间高度雾效
Shader "DepthTexture/ScreenSpaceHeightFog" 
{
	Properties
	{
		_MainTex("Base (RGB)", 2D) = "white" {}
	}
	
	CGINCLUDE
 #include "UnityCG.cginc"
	sampler2D _MainTex;
	sampler2D _CameraDepthTexture;
	float4x4 _ViewPortRay;
	float _FogHeight;
	float _WorldFogHeight;
	fixed4 _FogColor;
	
	struct v2f
	{
		float4 pos : SV_POSITION;
		float2 uv : TEXCOORD0;
		float4 rayDir : TEXCOORD1;
	};
	
	v2f vertex_depth(appdata_base v)
	{
		v2f o;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
		
		//用texcoord区分四个角
		int index = 0;
		if (v.texcoord.x < 0.5 && v.texcoord.y > 0.5)
			index = 0;
		else if (v.texcoord.x > 0.5 && v.texcoord.y > 0.5)
			index = 1;
		else if (v.texcoord.x < 0.5 && v.texcoord.y < 0.5)
			index = 2;
		else
			index = 3;
		
		o.rayDir = _ViewPortRay[index];
		return o;
		
	}
	
	fixed4 frag_depth(v2f i) : SV_Target
	{
		fixed4 screenTex = tex2D(_MainTex, i.uv);
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		float linear01Depth = Linear01Depth(depthTextureValue);
		
		float3 worldPos = _WorldSpaceCameraPos + linear01Depth * i.rayDir.xyz;
		
		float fogInensity = saturate((_WorldFogHeight - worldPos.y) / _FogHeight);
		return lerp(screenTex, _FogColor, fogInensity);
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vertex_depth
 #pragma fragment frag_depth
			ENDCG
		}
	}
}
```

C# 代码如下：

```
/********************************************************************
 FileName: ScreenSpaceHeightFog.cs
 Description:屏幕空间高度雾效
 Created: 2018/06/16
 history: 16:6:2018 21:23 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class ScreenSpaceHeightFog : MonoBehaviour {
 
    [Range(0.0f, 10.0f)]
    public float fogHeight = 0.1f;
    public Color fogColor = Color.white;
    public float horizontalPlane = 0.0f;
 
    private Material postEffectMat = null;
    private Camera currentCamera = null;
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/ScreenSpaceHeightFog"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            var aspect = currentCamera.aspect;
            var far = currentCamera.farClipPlane;
            var right = transform.right;
            var up = transform.up;
            var forward = transform.forward;
            var halfFovTan = Mathf.Tan(currentCamera.fieldOfView * 0.5f * Mathf.Deg2Rad);
 
            //计算相机在远裁剪面处的xyz三方向向量
            var rightVec = right * far * halfFovTan * aspect;
            var upVec = up * far * halfFovTan;
            var forwardVec = forward * far;
 
            //构建四个角的方向向量
            var topLeft = (forwardVec - rightVec + upVec);
            var topRight = (forwardVec + rightVec + upVec);
            var bottomLeft = (forwardVec - rightVec - upVec);
            var bottomRight = (forwardVec + rightVec - upVec);
 
            var viewPortRay = Matrix4x4.identity;
            viewPortRay.SetRow(0, topLeft);
            viewPortRay.SetRow(1, topRight);
            viewPortRay.SetRow(2, bottomLeft);
            viewPortRay.SetRow(3, bottomRight);
 
            postEffectMat.SetMatrix("_ViewPortRay", viewPortRay);
            postEffectMat.SetFloat("_WorldFogHeight", horizontalPlane + fogHeight);
            postEffectMat.SetFloat("_FogHeight", fogHeight);
            postEffectMat.SetColor("_FogColor", fogColor);
            Graphics.Blit(source, destination, postEffectMat);
        }
    }
}
```

开关后处理前后效果仍然不变：

![[f67cca779e9af6f80e0a8999a27985f1_MD5.png]]

这里我用了默认非线性的深度图进行的深度计算，需要先进行 Linear01Depth 计算，如果用了线性深度，比如 DepthNormalTexture，那么就进行一步简单的线性映射即可。整体的射线计算，我用了 Linear01Depth * 外围计算好的距离。也可以用 LinearEyeDepth * 外围计算好的方向。总之，方案还是蛮多的，变种也很多，还有自己重写 Graphic.Blit 自己设置 Quad 的值把 index 设置在顶点的 z 值中。

### 屏幕射线插值方式重建视空间坐标

补充一条屏幕空间深度重建坐标的 Tips。如果我们要求视空间的位置的话，有一种更简便并且性能更好的方式。这种方式与上面的屏幕射线插值的方式重建世界坐标的原理一致。只需要输入一个投影矩阵的逆矩阵，即在 vertex 阶段，从 NDC 坐标系的四个远裁剪面边界（+-1，+-1，1，1）乘以逆投影矩阵，得到视空间的四个远裁剪面坐标位置，然后除以齐次坐标转化到普通坐标下。这样的四个点的位置也就是视空间下从相机到该点的射线方向，经过插值到 fragment 阶段直接乘以 01 区间深度就得到了该像素点的视空间位置了。

那么就只有一个问题没有解决，在于应该如何获得 NDC 坐标系下的边界点。上面推导中提到过，在后处理阶段，实际上就是绘制了一个 Quad，对应整个屏幕。这个 Quad 的四个边界点刚好对应屏幕的四个边界点，uv 是（0,1）区间的，刚好对应屏幕空间，我们通过 * 2 - 1 将其转化到（-1,1）区间就可以得到四个边界对应 NDC 坐标系下的 xy 坐标了。

```
float fogInensity = (_WorldFogHeight - worldPos.y) / _FogHeight;
fogInensity = max(linear01Depth * _FogHeight, fogInensity);
return lerp(screenTex, _FogColor, saturate(fogInensity));
```

此处的 InverseProjectionMatrix 与上文中一样，也需要自己传入，因为在后处理阶段，内置矩阵已经被替换了。

## 屏幕空间高度或距离雾效果

在后处理阶段拿到世界空间位置，我们就可以做一些更加好玩的效果啦。屏幕空间高度或者距离雾就是其中之一。正常 Unity 中的雾效，实际上是在 shader 计算结束之后和雾效颜色根据世界空间距离计算的指数或者线性雾，对于一般的表现已经很好啦。而这个效果主要是可以模拟一些 “体积雾” 的感觉，让雾效更加明显，变成一个可以看得到的雾效，而不是仅仅附着在物体表面。

上面我们重建世界坐标后，我们就使用世界空间的高度作为雾效强度的判断条件，在最终计算颜色时，根据雾效高度差将屏幕原始颜色与雾效进行插值计算，即可得到屏幕空间高度雾效的效果。我直接使用了线性插值，也可以使用 exp 之类的。

shader 代码如下：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.6.17  
//通过深度图重建世界坐标方式运动模糊效果
Shader "DepthTexture/MotionBlurByDepth" 
{
	Properties
	{
		_MainTex("Base (RGB)", 2D) = "white" {}
	}
	
	CGINCLUDE
 #include "UnityCG.cginc"
	sampler2D _MainTex;
	sampler2D _CameraDepthTexture;
	float4x4 _CurrentInverseVPMatrix;
	float4x4 _PreviousInverseVPMatrix;
	float4 	 _BlurWeight;
	float 	 _BlurStrength;
	
	fixed4 frag_depth(v2f_img i) : SV_Target
	{
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		//自己操作深度的时候，需要注意Reverse_Z的情况
 #if defined(UNITY_REVERSED_Z)
		depthTextureValue = 1 - depthTextureValue;
 #endif
		float4 ndc = float4(i.uv.x * 2 - 1, i.uv.y * 2 - 1, depthTextureValue * 2 - 1, 1);
 
		float4 currentWorldPos = mul(_CurrentInverseVPMatrix, ndc);
		currentWorldPos /= currentWorldPos.w;
		
		float4 previousWorldPos = mul(_PreviousInverseVPMatrix, ndc);
		previousWorldPos /= previousWorldPos.w;
		
		float2 velocity = (currentWorldPos - previousWorldPos).xy * _BlurStrength;
		
		fixed4 screenTex = tex2D(_MainTex, i.uv);
		screenTex += tex2D(_MainTex, i.uv + velocity * 1.0) * _BlurWeight.x;
		screenTex += tex2D(_MainTex, i.uv + velocity * 2.0) * _BlurWeight.y;
		screenTex += tex2D(_MainTex, i.uv + velocity * 3.0) * _BlurWeight.z;
		
		screenTex /= (1.0 + _BlurWeight.x + _BlurWeight.y + _BlurWeight.z);
		
		return screenTex;
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vert_img
 #pragma fragment frag_depth
			ENDCG
		}
	}
}
```

C# 代码如下：

```
/********************************************************************
 FileName: MotionBlurByDepth.cs
 Description:通过深度图重建世界坐标方式运动模糊效果
 Created: 2018/06/17
 history: 17:6:2018 1:47 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class MotionBlurByDepth : MonoBehaviour {
 
    private Material postEffectMat = null;
    private Camera currentCamera = null;
    private Matrix4x4 previouscurrentVPMatrix;
    [Range(0.0f, 0.02f)]
    public float blurStrength = 0.5f;
    public Vector3 blurWeight = new Vector3(0.6f, 0.3f, 0.1f);
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/MotionBlurByDepth"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            postEffectMat.SetMatrix("_PreviousInverseVPMatrix", previouscurrentVPMatrix);
            var currentVPMatrix = (currentCamera.projectionMatrix * currentCamera.worldToCameraMatrix).inverse;
            postEffectMat.SetMatrix("_CurrentInverseVPMatrix", currentVPMatrix);
            postEffectMat.SetFloat("_BlurStrength", blurStrength);
            postEffectMat.SetVector("_BlurWeight", blurWeight);
            Graphics.Blit(source, destination, postEffectMat);
        }
    }
}
```

效果如下：

![[1305ee80657137c89fbaefbb6d9fcb38_MD5.png]]

雾效的判断条件也可以修改为高度 + 距离：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.6.18  
//扩散波动效果
Shader "DepthTexture/SpreadWaveByDepth" 
{
	Properties
	{
		_MainTex("Base (RGB)", 2D) = "white" {}
	}
	
	CGINCLUDE
 #include "UnityCG.cginc"
	sampler2D _MainTex;
	sampler2D _CameraDepthTexture;
	float4x4 _ViewPortRay;
	fixed4 _ScanColor;
	float _ScanValue;
	float4 _ScanCenterPos;
	float _ScanCircleWidth;
	
	struct v2f
	{
		float4 pos : SV_POSITION;
		float2 uv : TEXCOORD0;
		float4 rayDir : TEXCOORD1;
	};
	
	v2f vertex_depth(appdata_base v)
	{
		v2f o;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
		
		//用texcoord区分四个角
		int index = 0;
		if (v.texcoord.x < 0.5 && v.texcoord.y > 0.5)
			index = 0;
		else if (v.texcoord.x > 0.5 && v.texcoord.y > 0.5)
			index = 1;
		else if (v.texcoord.x < 0.5 && v.texcoord.y < 0.5)
			index = 2;
		else
			index = 3;
		
		o.rayDir = _ViewPortRay[index];
		return o;
		
	}
	
	fixed4 frag_depth(v2f i) : SV_Target
	{
		fixed4 screenTex = tex2D(_MainTex, i.uv);
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		float linear01Depth = Linear01Depth(depthTextureValue);
		
		float3 worldPos = _WorldSpaceCameraPos + linear01Depth * i.rayDir.xyz;
		
		float dist = distance(worldPos, _ScanCenterPos.xyz);
		
		if (dist > _ScanValue && dist < _ScanCircleWidth + _ScanValue)
			return screenTex * _ScanColor;
		return screenTex;
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vertex_depth
 #pragma fragment frag_depth
			ENDCG
		}
	}
}
```

效果如下：

![[ee1d1c38a47aff51ae0e2230649ad884_MD5.gif]]

## 运动模糊效果

运动模糊效果还是有很多种其他的方式去做的，比如渲染速度图，不过本篇只考虑了深度重建世界空间位置的做法进行模糊处理。参考[《GPU Gems 3-Motion Blur as a Post-Processing Effect》](https://developer.nvidia.com/gpugems/GPUGems3/gpugems3_ch27.html)这篇文章。渲染速度图的方式，需要额外的渲染批次，没有 MRT 的话批次比较难搞。用深度图进行运动模糊的话，无需额外的批次（不考虑深度本身的批次，毕竟开了深度是一个好多效果都可以用，性价比更高），不过也有一个问题，就是这种方式的运动模糊只能模糊相机本身的运动。

上面我们已经过通过逆矩阵进行重建世界空间位置，那么视线运动模糊就好实现啦。相比于存储一张上一帧的贴图，这里我们直接存储一下上一阵的（相机 * 投影矩阵）的逆矩阵，然后重建世界坐标，用其中差作为 uv 采样的偏移值进行采样，然后按权重进行模糊计算，模拟一个运动物体拖尾的效果。

shader 代码如下：

```
/********************************************************************
 FileName: SpreadWaveByDepth.cs
 Description:扩散波动效果
 Created: 2018/06/18
 history: 18:6:2018 15:56 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class SpreadWaveByDepth : MonoBehaviour
{
    public Color scanColor = Color.white;
    public float scanValue = 0.0f;
    public float scanCircleWidth = 1.0f;
    public Vector3 scanCenterPos = Vector3.zero;
 
 
    private Material postEffectMat = null;
    private Camera currentCamera = null;
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/SpreadWaveByDepth"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            var aspect = currentCamera.aspect;
            var far = currentCamera.farClipPlane;
            var right = transform.right;
            var up = transform.up;
            var forward = transform.forward;
            var halfFovTan = Mathf.Tan(currentCamera.fieldOfView * 0.5f * Mathf.Deg2Rad);
 
            //计算相机在远裁剪面处的xyz三方向向量
            var rightVec = right * far * halfFovTan * aspect;
            var upVec = up * far * halfFovTan;
            var forwardVec = forward * far;
 
            //构建四个角的方向向量
            var topLeft = (forwardVec - rightVec + upVec);
            var topRight = (forwardVec + rightVec + upVec);
            var bottomLeft = (forwardVec - rightVec - upVec);
            var bottomRight = (forwardVec + rightVec - upVec);
 
            var viewPortRay = Matrix4x4.identity;
            viewPortRay.SetRow(0, topLeft);
            viewPortRay.SetRow(1, topRight);
            viewPortRay.SetRow(2, bottomLeft);
            viewPortRay.SetRow(3, bottomRight);
 
            postEffectMat.SetMatrix("_ViewPortRay", viewPortRay);
            postEffectMat.SetColor("_ScanColor", scanColor);
            postEffectMat.SetVector("_ScanCenterPos", scanCenterPos);
            postEffectMat.SetFloat("_ScanValue", scanValue);
            postEffectMat.SetFloat("_ScanCircleWidth", scanCircleWidth);
            Graphics.Blit(source, destination, postEffectMat);
        }
    }
}
```

C# 代码如下：

```
/********************************************************************
 FileName: MotionBlurByDepth.cs
 Description:通过深度图重建世界坐标方式运动模糊效果
 Created: 2018/06/17
 history: 17:6:2018 1:47 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class MotionBlurByDepth : MonoBehaviour {
 
    private Material postEffectMat = null;
    private Camera currentCamera = null;
    private Matrix4x4 previouscurrentVPMatrix;
    [Range(0.0f, 0.02f)]
    public float blurStrength = 0.5f;
    public Vector3 blurWeight = new Vector3(0.6f, 0.3f, 0.1f);
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/MotionBlurByDepth"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            postEffectMat.SetMatrix("_PreviousInverseVPMatrix", previouscurrentVPMatrix);
            var currentVPMatrix = (currentCamera.projectionMatrix * currentCamera.worldToCameraMatrix).inverse;
            postEffectMat.SetMatrix("_CurrentInverseVPMatrix", currentVPMatrix);
            postEffectMat.SetFloat("_BlurStrength", blurStrength);
            postEffectMat.SetVector("_BlurWeight", blurWeight);
            Graphics.Blit(source, destination, postEffectMat);
        }
    }
}
```

效果如下：

![[e0d21e096207e9bddbe499f78df6287e_MD5.gif]]

据我观察，实际上有些游戏里面的运动模糊并不是全屏幕的，而是只模糊边缘，相机中间不模糊，可以加个根据距离屏幕中心距离计算权重与原图 lerp 的操作，不过如果不是很纠结效果的话，用[径向模糊](https://blog.csdn.net/puppet_master/article/details/54566397)来代替运动模糊也是个不错的选择。

真正的对高速运动物体进行运动模糊，肯定要比这种 Trick 的方式要复杂一点，也更真实一些，不过这个不在本文讨论范围了。

## 扩散扫描效果

接下来来实现一个更好玩一点的效果。类似开头《恶灵附身》截图中第二个的效果，在空间一点实现扩散扫描的效果，其实与屏幕空间扫描线的效果实现思路是一样的，只不过实现了世界空间重建后，我们的判断条件就可以更加复杂，用世界空间位置进行计算可以实现一些复杂一些的形状的扫描线。比如判断与一点的距离实现的环形扩散扫描：

Shader 代码如下：

```
//puppet_master
//https://blog.csdn.net/puppet_master 
//2018.6.18  
//扩散波动效果
Shader "DepthTexture/SpreadWaveByDepth" 
{
	Properties
	{
		_MainTex("Base (RGB)", 2D) = "white" {}
	}
	
	CGINCLUDE
 #include "UnityCG.cginc"
	sampler2D _MainTex;
	sampler2D _CameraDepthTexture;
	float4x4 _ViewPortRay;
	fixed4 _ScanColor;
	float _ScanValue;
	float4 _ScanCenterPos;
	float _ScanCircleWidth;
	
	struct v2f
	{
		float4 pos : SV_POSITION;
		float2 uv : TEXCOORD0;
		float4 rayDir : TEXCOORD1;
	};
	
	v2f vertex_depth(appdata_base v)
	{
		v2f o;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
		
		//用texcoord区分四个角
		int index = 0;
		if (v.texcoord.x < 0.5 && v.texcoord.y > 0.5)
			index = 0;
		else if (v.texcoord.x > 0.5 && v.texcoord.y > 0.5)
			index = 1;
		else if (v.texcoord.x < 0.5 && v.texcoord.y < 0.5)
			index = 2;
		else
			index = 3;
		
		o.rayDir = _ViewPortRay[index];
		return o;
		
	}
	
	fixed4 frag_depth(v2f i) : SV_Target
	{
		fixed4 screenTex = tex2D(_MainTex, i.uv);
		float depthTextureValue = SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture, i.uv);
		float linear01Depth = Linear01Depth(depthTextureValue);
		
		float3 worldPos = _WorldSpaceCameraPos + linear01Depth * i.rayDir.xyz;
		
		float dist = distance(worldPos, _ScanCenterPos.xyz);
		
		if (dist > _ScanValue && dist < _ScanCircleWidth + _ScanValue)
			return screenTex * _ScanColor;
		return screenTex;
	}
	ENDCG
 
	SubShader
	{
		Pass
		{
			ZTest Off
			Cull Off
			ZWrite Off
			Fog{ Mode Off }
 
			CGPROGRAM
 #pragma vertex vertex_depth
 #pragma fragment frag_depth
			ENDCG
		}
	}
}
```

C# 代码如下：

```
/********************************************************************
 FileName: SpreadWaveByDepth.cs
 Description:扩散波动效果
 Created: 2018/06/18
 history: 18:6:2018 15:56 by puppet_master
 https://blog.csdn.net/puppet_master
*********************************************************************/
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
[ExecuteInEditMode]
public class SpreadWaveByDepth : MonoBehaviour
{
    public Color scanColor = Color.white;
    public float scanValue = 0.0f;
    public float scanCircleWidth = 1.0f;
    public Vector3 scanCenterPos = Vector3.zero;
 
 
    private Material postEffectMat = null;
    private Camera currentCamera = null;
 
    void Awake()
    {
        currentCamera = GetComponent<Camera>();
    }
 
    void OnEnable()
    {
        if (postEffectMat == null)
            postEffectMat = new Material(Shader.Find("DepthTexture/SpreadWaveByDepth"));
        currentCamera.depthTextureMode |= DepthTextureMode.Depth;
    }
 
    void OnDisable()
    {
        currentCamera.depthTextureMode &= ~DepthTextureMode.Depth;
    }
 
    void OnRenderImage(RenderTexture source, RenderTexture destination)
    {
        if (postEffectMat == null)
        {
            Graphics.Blit(source, destination);
        }
        else
        {
            var aspect = currentCamera.aspect;
            var far = currentCamera.farClipPlane;
            var right = transform.right;
            var up = transform.up;
            var forward = transform.forward;
            var halfFovTan = Mathf.Tan(currentCamera.fieldOfView * 0.5f * Mathf.Deg2Rad);
 
            //计算相机在远裁剪面处的xyz三方向向量
            var rightVec = right * far * halfFovTan * aspect;
            var upVec = up * far * halfFovTan;
            var forwardVec = forward * far;
 
            //构建四个角的方向向量
            var topLeft = (forwardVec - rightVec + upVec);
            var topRight = (forwardVec + rightVec + upVec);
            var bottomLeft = (forwardVec - rightVec - upVec);
            var bottomRight = (forwardVec + rightVec - upVec);
 
            var viewPortRay = Matrix4x4.identity;
            viewPortRay.SetRow(0, topLeft);
            viewPortRay.SetRow(1, topRight);
            viewPortRay.SetRow(2, bottomLeft);
            viewPortRay.SetRow(3, bottomRight);
 
            postEffectMat.SetMatrix("_ViewPortRay", viewPortRay);
            postEffectMat.SetColor("_ScanColor", scanColor);
            postEffectMat.SetVector("_ScanCenterPos", scanCenterPos);
            postEffectMat.SetFloat("_ScanValue", scanValue);
            postEffectMat.SetFloat("_ScanCircleWidth", scanCircleWidth);
            Graphics.Blit(source, destination, postEffectMat);
        }
    }
}
```

效果（开了 Bloom 后处理，有了 Bloom，再挫的画面也能加不少分，哈哈哈）：

![[38fecbc590a045564dd8c09f34b01c2b_MD5.gif]]

## 总结

本篇主要是总结了一下实时渲染当中关于深度（图）相关的一些内容，主要是透视投影，ZBuffer 算法，1/Z 问题，深度图的基本使用，Linear01Depth，LinearEyeDepth，ZBuffer 精度，Reverse-Z，根据深度重建世界坐标等内容。另外，实现了软粒子，屏幕空间扫描波，扩散波，屏幕空间高度雾，运动模糊等常见的使用深度图的效果。效果的实现其实都比较基本，但是扩展性比较强，用深度可以做很多很多好玩的东西。文中的一些特殊效果实现已经给出了参考链接，另外还参考了乐乐大佬的《shader lab 入门精要》关于运动模糊以及高度雾效的部分。不过我还是低估了深度的内容，精简过后还是这么多，可见，深度对于渲染的重要性。更加复杂的渲染效果，如 SSAO，阴影，基于深度和法线的描边效果之类的，还是等之后在写啦。

注：文中的 shader 目前都没有考虑 #if UNITY_UV_STARTS_AT_TOP 的情况，在 5.5 版本以前（我只有 4.3，5.3，5.5，2017，2018 这几个版本，具体哪个版本开始不需要考虑这个问题，不太确定），PC 平台，开启 AA 的情况下会出现 RT 采样翻转的情况。以前的一些 blog 是都加了这个宏判断的，不过目前测试的新版本 2017.3 貌似木有了这个问题（也可能触发条件变了），所以我就愉快地偷了个懒喽。