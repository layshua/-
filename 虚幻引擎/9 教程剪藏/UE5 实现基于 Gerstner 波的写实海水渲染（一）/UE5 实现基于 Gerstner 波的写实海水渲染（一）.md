## 前言
[UE5实现基于Gerstner波的写实海水渲染（一） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/609316112)
废话不多说，效果放前头：

[UE5 Gerstner 海洋测试_哔哩哔哩_bilibili](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1Hy4y1o7JB/%3Fspm_id_from%3D333.999.0.0%26vd_source%3D1144cd70fd26d7a4d02397189f9ad785)

前段时间因为做作品集场景需要，开始研究了一下写实海水渲染。

海水渲染和常见的风格化插片水不同，海水必须做顶点位移才能有比较好的效果，下图是我开始随便写的一个插片海水，效果其实一般般，尤其是和建筑连接的海水部分没有波浪。写了一半就去找波形的资料去了。

![](1677303700494.png)

看了不少文章，感觉 FFT 算是一个逼格、效果和性能占用都拉满的方案。考虑到今年春招开的很早，FFT 实现不太来得及，就先做了一套 Gerstner 波的方案，感觉效果也算满意的。

[毛星云：真实感水体渲染技术总结](https://zhuanlan.zhihu.com/p/95917609)

## Gerstner 波介绍

Gerstner 波的本质是，水表面的每个点都绕着一个固定的锚点作圆周动画。整个水面的效果就像是水在波峰中聚集，在波谷中扩散。

![](1677303700897.png)

可以看下面这篇文章，用 Houdini 展示的 Gerstner 波的生成方式，讲的非常清楚了。

[Justin：Gerstner 水波详解](https://zhuanlan.zhihu.com/p/157752100)

![](1677303700949.png)

首先给一个 100*100 细分的 gird，然后 vex 代码如下：

```
//声明强度、波长、速度
float Amplitude = chf("Amplitude");
float Wavelength = chf("Wavelength");
float Speed = chf("Speed");
//计算频率
float frequency = 2*PI/Wavelength;
//核心代码，在Y轴上计算正弦波并根据时间来偏移
@P.y = Amplitude * sin(frequency *(@P.x - Speed *@Frame));
//使用偏导计算法线
vector tangent = set(
    1,
    Amplitude * frequency * cos(frequency *(@P.x - Speed *@Frame)),
    );
tangent = normalize(tangent);
vector normal = set(-tangent.y,tangent.x,0);
@N = normal;
```

理解了这段 vex 代码的含义，我们就可以继续下面的工作：

## 一维 G 波

首先导入一个带细分的 mesh 网格平面。

![](1677303701249.png)

然后新建材质，shader 使用默认的不透明即可。

![](1677303701296.png)

![](1677303701548.png)

基础色直接采样，我这里以一个网格纹理先演示 Gerstner 波的生成。

![](1677303701917.png)

核心是输出法线和世界坐标偏移，也就是顶点动画。总体思路和上面 Houdini 的那篇文章是类似的。

然后是核心的材质函数

![](1677303702154.png)

这里可以使用 Custom HLSL 节点直接写，比蓝图连成蜘蛛网看起来清爽很多。整体算法和上面的 vex 算法是类似的。

HLSL 代码：

```
float2 xy = float2(0.0f, 0.0f);
float z = 0.0f;

float2 nxy = float2(0.0f, 0.0f);
float nz = 0.0f;

// Gerstner波的生成
float dispersion = 2.f * PI / wavelength;
float wavespeed = sqrt(dispersion * 981.f);
float2 wavevector = direction * dispersion;
float wavetime = wavespeed * time;

float wavepos = dot(float2(position.x, position.y), wavevector) - wavetime;

float wavesin = sin(wavepos);
float wavecos = cos(wavepos);

float wKA = amplitude * dispersion;

float q = steepness / wKA;

// normal (这里由于编码限制，需要限制法线的Z分量)
nxy = wavesin * wKA * direction;
nz = wavecos * steepness * saturate((amplitude * 50.f) / wavelength);

xy = -q * wavesin * direction * amplitude;
z = wavecos * amplitude;

Normal = float3(nxy.x, nxy.y, nz);
Height = z;
return float3(xy.x, xy.y, z);
```

OK，现在就可以看到一个基础的一维 Gerster 波的运动效果了。

![](1677303702533.png)

## 二维 G 波

这边实现应该就很容易了，无非就是换写输入叠加一下。我简单写一下自己的操作。

新建一个材质函数，新添加几个波。不同的波给不同的参数，使用一个材质参数集统一控制：

![](1677303702738.png)

有了参数集，在新的材质函数中，分别计算每个要添加的波：

![](1677303702944.png)

最后统一计算，输出即可。

![](1677303703228.png)

switch 节点用于开关这个波，也可以不写。

二维叠加的效果：

![](1677303703463.png)

这一篇先写到这。下一篇会写使用 Niagara 采样浪尖白沫。