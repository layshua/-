


## 纹理环绕（Texture Wrap）

纹理环绕的作用是为了处理超出 0.0~1.0 范围的纹理坐标，例如采用重复（REPEAT）的环绕方式，采样（4.5,-4.5）的纹理坐标，实际采样的纹理坐标应是（0.5，0.5）。对于负数纹理坐标，采样的实际位置应是 1-uv，例如（-0.2,-0.6）应该采样（0.8,0.4）

![](<images/1684499682357.png>)

请注意，纹理坐标的取值范围是 [0.0,1.0]，两端都是**闭区间**。许多人（包括我自己）之前在写软件渲染器时都是简单地将纹理坐标减去了小数部分。

```
float u = texCoord.x - floor(texCoord.x);
float v = texCoord.y - floor(texCoord.y);
int x = u * (width - 1);
int y = v * (height - 1);
```

这样操作会丢失 uv = 1.0 的边界纹理。虽然在绝大部分情况下并无大碍，但在绘制天空盒等较大物体时会出现纹理接缝。

![](<images/1684499682424.png>)

此问题的原因在于，我们在映射纹理坐标时像素的坐标是（0,0）到（width-1,height-1），但这两个坐标对应的是像素中心的坐标。

![](<images/1684499682480.png>)

真实的纹理像素边界是（-0.5,-0.5）到（width-0.5,height-0.5）。例如上图中从（0,0）到（4,4）画一个正方形，理论上应该覆盖这个范围

![](<images/1684499682524.png>)

但由于一个像素只能填充一种颜色，实际光栅化的区域是下图所示

![](<images/1684499682604.png>)

我们将这样一个纹理贴上去

![](<images/1684499682643.png>)

结果如下：

![](<images/1684499682708.png>)

可以看到，对应 UV=1 的行列颜色都错了。这是因为，我们采样使用的是在像素中心处插值得到的 UV 值。像下面这个图上，第二行右侧的点，插值得到的像素中心 UV 为（0.75,0.25），直接计算得到的采样坐标为（0.75*3,0.25*3）=（2,0）, 但实际它应该采样（3,1）处。

![](<images/1684499682764.png>)

解决方法是在进行纹理采样时，对纹理坐标加上 - 0.5 的偏移

```
int x = (int)(texCoord.x * width - 0.5f) % width;
int y = (int)(texCoord.y * height - 0.5f) % height;
x = x < 0 ? width + x : x;
y = y < 0 ? height + y : y;
```

![](<images/1684499682800.png>)

这里贴上 D3D9 的文档地址：[从像素到纹素](https://docs.microsoft.com/en-us/windows/win32/direct3d9/directly-mapping-texels-to-pixels)

## 纹理过滤（Texture Filtering）

纹理过滤的作用是将浮点型纹理坐标转换为整数的像素坐标，并对采样结果进行处理。简单地说，由于我们用于显示纹理的图形与纹理图像存在大小、形状的区别，我们需要在采样过程中进行一定的处理来进行滤波，否则会显示为伪像，包括重叠、错位等。

![](<images/1684499682835.png>)

在 OpenGL 中有两种基础过滤方式，邻近点（Nearest）和双线性（Bilinear）

![](<images/1684499682872.png>)

顾名思义，邻近点就是选取与纹理坐标最接近的像素点颜色，其操作伪代码如下：

```
vec4 Sample2D(vec2 texCoord){
        int x = (int)(texCoord.x * width - 0.5f) % width;
        int y = (int)(texCoord.y * height - 0.5f) % height;
        x = x < 0 ? width + x : x;
        y = y < 0 ? height + y : y;
        return GetColor(x,y);
}
```

这样操作简单粗暴，但是像素之间会呈现明显的马赛克现象，尤其是在纹理分辨率与图像大小不一致时。

双线性过滤能够很好地解决上述现象，像素之间的过渡更加平滑，但代价是对于图形的每一个像素点，我们需要在纹理上采样 4 个像素颜色进行插值。关于双线性过滤的具体操作过程，网上的资料大多说得比较模糊，都说是采样最近的四个像素，却并没有说明如何操作。经过一番查证，我发现在 OpenGL 中，双线性过滤采样的是像素本身以及往上、往右、往右上分别移动一格的像素点。获得的四个像素并不是简单平均，而是根据整数纹理坐标的小数值进行插值。举个列子，一张 512x512 的图片，双线性过滤采样（0.6,0.6）的位置，实际步骤如下：

1.  换算纹理像素坐标为（307.2,307.2），实际采样的基准像素点为 s1 =（307,307）
2.  继续采样 s2 =（308，307）、s3 =（307，308）和 s4 =（308，308）三个点
3.  小数部分为（0.2,0.2），以 x 部分 0.2 分别在 s1,s2 和 s3,s4 间插值
4.  以 y 部分 0.2 插值上一步中的两个结果

```
vec4 Sample2D(vec2 texCoord) {
        texCoord = texCoord * vec2(width,height) - vec2(0.5f);
        float f = fract(texCoord);
        int x = (int)(texCoord.x) % width;
        int y = (int)(texCoord.y) % height;
        x = x < 0 ? width + x : x;
        y = y < 0 ? height + y : y;
	vec4 s1 = GetColor(x,y);
	vec4 s2 = GetColor(x+1,y);
	vec4 s3 = GetColor(x,y+1);
	vec4 s4 = GetColor(x+1,y+1);
	return lerp(lerp(s1, s2, f.x), lerp(s3, s4, f.x), f.y);
}
```

上述代码中并没有考虑边界问题，这是因为边界颜色跟环绕设置有关，如果设置为 Clamp to border 模式，超出边界的部分会采样背景色；在 Repeat 模式下，采样 (width-1, y）和（x,height-1) 位置时，会从（0,y）和（x,0）获取颜色。

双线性过滤每次会采样四个像素点，但采样过程由硬件实现，性能消耗与临近点过滤基本是一样的。

## Mipmap

我们已经知道了在采样纹理时，纹理大小跟图形大小接近才会有好的显示效果，因此便有了 Mipmap 技术。Mipmap 的原理是预先生成一系列以 2 为倍数缩小的纹理序列，在采样纹理时根据图形的大小自动选择相近等级的 Mipmap 进行采样。

![](<images/1684499682994.png>)

使用 Mipmap（通常结合使用双线性过滤）可以有效消除远处物体出现的纹理重叠现象

![](<images/1684499683031.png>)

那么问题来了，我们知道像素着色器是以像素为单位运行的，采样时该如何得知图形的大小呢？实际上在 GPU 中像素着色器并不是逐个像素运行，而是同时处理 2x2 的像素块，并提供了（唯一）一组获取相邻像素信息的函数——偏导函数 dFdx 和 dFdy。偏导数代表了函数在某一方向的变化率，那么如果相邻两个像素间纹理坐标变化很大，不就能说明绘制的图形很小了吗？

事实上确实是这么做的，例如 OpenGL 就是通过计算出纹理坐标在纵向和横向的偏导数（并取最大值）来计算 Mipmap 级别

```
float MipmapLevel(vec2 uv, vec2 texSize) {
    // The OpenGL Graphics System: A Specification 4.2
    //  - chapter 3.9.11, equation 3.21

    vec2 s = dFdx(uv) * texSize;
    vec2 t = dFdy(uv) * texSize;
    float delta = max(dot(s, s), dot(t, t));
    return 0.5 * log2(delta);
}
```

假设我们有一个 512x512 的图片，贴在屏幕空间上呈 300x300 大小的正方形上。可知相邻像素的 UV 坐标差值为 $\frac{1}{300}$ ，乘上纹理尺寸后偏导数约为 1.706，delta 为 2.91271，计算得到 MipmapLevel 为 0.7712，因此最接近的级别为 1，也就是 256x256 这一级。

值得注意的是，这么做的前提条件是在屏幕空间中纹理坐标是连续变化的 [[1]](https://zhuanlan.zhihu.com/p/143377682#ref_1)。该结论很好证明，因为 UV 坐标呈线性变化，一次方程求导之后为常数。

Mipmap 除了能消除采样率过低带来的失真问题，还有一个重要的优点是**节约显存带宽**，注意是带宽而**不是容量**。Mipmap 实际消耗的显存大约增加了 1/3，但每次仅从需要的 Mipmap 级别进行读取，而不必每次都访问原始大小的纹理，因此可以节约带宽。

## 三线性过滤

在同时启用双线性过滤和 Mipmap 之后，我们解决了远处物体的显示失真问题，却又引入了新的问题——Mipmap 跳变。因为每个 Mipmap 级别的分别率相差四倍，当图形一段区域的 Mipmap 级别恰好处于两个整数之间时，就会发生跳变。

![](<images/1684499683083.png>)

在镜头不动时还不明显，一旦移动起来就可以看到一条明显的由清晰到模糊的分界线。为解决这一问题，又出现了三线性过滤方法。三线性过滤的实现就很简单了，对于浮点值 MipmapLevel，分别在其**前后两个整数级别**的 Mipmap 上进行双线性过滤，然后将两个结果再根据 MipmapLevel 到两个级别的距离进行平均。

现代 GPU 上，三线性过滤同样由硬件提供，每次会采样 8 个像素，但通常来说性能消耗不大。

## 各向异性过滤

终于写到本文的重点了。使用三线性过滤与 Mipmap 之后，对于在屏幕上呈现（近似）正方形的图形，我们已经能够取得很好的效果。但是对于倾斜或者长条状的图形，显示效果依然不够好

![](<images/1684499683132.png>)

究其原因，在于我们是取得纹理坐标在 xy 方向上较大的那个变化率计算得到的 MipmapLevel，而倾斜或是长条形状的物体，在 xy 方向上的纹理坐标变化率可以差距很大。例如下图中，左图在同样的距离上 du 与 dv 基本相等，而右图中 dv 则大约是 du 的两倍。若在这种情况下开启 Mipmap，右边的图形就会被贴上更低一级的 Mipmap，导致模糊。

![](<images/1684499683197.png>)

解决上述问题的终极方案便是各向异性过滤 [[2]](https://zhuanlan.zhihu.com/p/143377682#ref_2)。各项异性过滤的实现比较复杂，各位可以查阅原始的论文获取最准确的方案，这里仅说说我个人的理解步骤：

（1）计算得到 UV 在 xy 方向上的偏导数 $du_{x},du_{y},dv_{x},dv_{y}$

（2）计算得到纹理坐标偏导数 $s=du*width,t=dv*height$

（3）计算绘制图形在纹理空间的投影向量 **r1 r2 d1 d2**

$r1 = (s_{x},t_{x}),r2=(s_{y},t_{y})$

![](<images/1684499683255.png>)

其中，为了后续计算简便，将向量的模进行近似

$|r1| \approx max(|s_{x}|,|t_{x}|)$

$|r2| \approx max(|s_{y}|,|t_{y}|)$

$|d1| \approx max(|s_{x}+s_{y}|,|t_{x}+t_{y}|)$

$|d1| \approx max(|s_{x}-s_{y}|,|t_{x}-t_{y}|)$

（3）计算使用的 MipmapLevel

$j=min(|r1|,|r2|,|d1|,|d2|)$

$l = log_{2}j$ ，此为采样使用的 MipmapLevel

$f=\frac{j}{2^l}-1$ ，此为三线性过滤使用的插值系数

（4）计算各向异性比例

$N = min(2^{[log_{2}\frac{max(|r1|,|r2|)}{j}]}, maxAniso)$ 这里的 maxAniso 就是设置开启的各向异性过滤级别，方括号代表取整

（4）在上面计算出的 MipmapLevel 下进行多次三线性采样，为此需要生成一系列采样坐标

$m = max(r1,r2)$ (这里使用的是向量 r1 r2)

$du = \frac{m.x}{2^{l}N},dv = \frac{m.y}{2^{l}N}$

$u_{n} = u + (\frac{n}{2} du),v_{n} = v+(\frac{n}{2}dv)$

其中 $n = -N+1,-N+3......-3,-1,1,3......N-3,N-1$

（5）将采样获得的颜色进行平均

一般来说，我们在游戏中最高可以开启 16x 的各向异性过滤，但是实际运算时会将我们设置的各向异性级别与计算得的 N 取较小值，因此并不是开启了 16x 就一定会执行 16 次采样。各向异性过滤相比双线性的性能消耗成倍增长，好在现代 GPU 上纹理采样已不是瓶颈，在平时游戏时我们可以放心开启此选项，帧数不足时优先关闭阴影、抗锯齿等特效。

## 总结

在现代 GPU 中，纹理的采样与过滤方式皆已通过硬件实现，因此我们现在已经难以见到对相关技术细节的讲解了。在本文的撰写过程中，参考了几十年前发表的原始论文，以及 OpenGL 规范的文档。如果你认为本文对你有所帮助，那么就请点赞收藏支持一下吧~

## 参考

1.  [^](https://zhuanlan.zhihu.com/p/143377682#ref_1_0) Ewins JP, Waller MD, White M, Lister PF. MIP-map level selection for texture mapping. IEEE Transactions on Visualization and Computer Graphics 1998;4(4):317}29. [https://ieeexplore.ieee.org/abstract/document/765326](https://ieeexplore.ieee.org/abstract/document/765326)
2.  [^](https://zhuanlan.zhihu.com/p/143377682#ref_2_0)Implementing an anisotropic texture filter [https://www.sciencedirect.com/science/article/abs/pii/S0097849399001594](https://www.sciencedirect.com/science/article/abs/pii/S0097849399001594)