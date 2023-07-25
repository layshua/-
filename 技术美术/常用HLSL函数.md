```cpp
max
min
fmod //取余
round //四舍五入
pow(a,b) //指数幂
sqrt //平方根
rsqrt //平方根的倒数
degrees //弧度转换成角度
redians //角度转换成弧度
noise //噪声函数，传入二维坐标范围[0,1]随机值
log //幂指对函数
cos sin tan atan atan2 //三角函数
sinh cosh tanh //双曲线函数
ceil
floor
smoothstep
clamp
saturate
step
lerp
frac
length //向量到原点的距离
distance //两点距离
normalize
mul
dot
sign //返回x的正负，x<0返回-1，x=0返回0，x>0返回1
any //传入矢量，任何一个分量为非0返回true，否则false
all //传入矢量，所有分量均为非0返回true，否则false
```
HLSL 提供了一些内置全局函数，它通常直接映射到指定的着色器汇编指令集。这里只列出一些比较常用的函数：

<table><thead><tr><th>函数名</th><th>描述</th><th>最小支持着色器模型</th></tr></thead><tbody><tr><td>abs</td><td>每个分量求绝对值</td><td>1.1</td></tr><tr><td>acos</td><td>求 x 分量的反余弦值</td><td>1.1</td></tr><tr><td>all</td><td>测试 x 分量是否按位全为 1</td><td>1.1</td></tr><tr><td>any</td><td>测试 x 分量是否按位存在 1</td><td>1.1</td></tr><tr><td>asdouble</td><td>将值按位重新解释成 double 类型</td><td>5.0</td></tr><tr><td>asfloat</td><td>将值按位重新解释成 float 类型</td><td>4.0</td></tr><tr><td>asin</td><td>求 x 分量的反正弦值</td><td>1.1</td></tr><tr><td>asint</td><td>将值按位重新解释成 int 类型</td><td>4.0</td></tr><tr><td>asuint</td><td>将值按位重新解释成 uint 类型</td><td>4.0</td></tr><tr><td>atan</td><td>求 x 分量的反正切值值</td><td>1.1</td></tr><tr><td>atan2</td><td>求 (x, y) 分量的反正切值</td><td>1.1</td></tr><tr><td>ceil</td><td>求不小于 x 分量的最小整数</td><td>1.1</td></tr><tr><td>clamp</td><td>将 x 分量的值限定在 [min, max]</td><td>1.1</td></tr><tr><td>clip</td><td>丢弃当前像素，如果 x 分量的值小于 0</td><td>1.1</td></tr><tr><td>cos</td><td>求 x 分量的余弦值</td><td>1.1</td></tr><tr><td>cosh</td><td>求 x 分量的双曲余弦值</td><td>1.1</td></tr><tr><td>countbits</td><td>计算输入整数的位 1 个数 (对每个分量)</td><td>5.0</td></tr><tr><td>cross</td><td>计算两个 3D 向量的叉乘</td><td>1.1</td></tr><tr><td>ddx</td><td>估算屏幕空间中的偏导数<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-1-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp; #x2202 ;</mi><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi mathvariant=&quot;bold&quot;>p</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mi mathvariant=&quot;normal&quot;>&amp; #x2202 ;</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-1" style="width: 3.66em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.917em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1002.86em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2"><span class="mi" id="MathJax-Span-3" style="font-family: MathJax_Main;">∂<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="texatom" id="MathJax-Span-4"><span class="mrow" id="MathJax-Span-5"><span class="mi" id="MathJax-Span-6" style="font-family: MathJax_Main-bold;">p</span></span></span><span class="texatom" id="MathJax-Span-7"><span class="mrow" id="MathJax-Span-8"><span class="mo" id="MathJax-Span-9" style="font-family: MathJax_Main;">/</span></span></span><span class="mi" id="MathJax-Span-10" style="font-family: MathJax_Main;">∂<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mi" id="MathJax-Span-11" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">∂</mi><mrow class="MJX-TeXAtom-ORD"><mi mathvariant="bold">p</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mi mathvariant="normal">∂</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-1">\partial \mathbf{p} / \partial x</script></span>。这使我们可以确定在屏幕空间的 x 轴方向上，相邻像素间某属性值<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-2-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi mathvariant=&quot;bold&quot;>p</mi></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-12" style="width: 0.803em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.66em, 1000.57em, 2.631em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-13"><span class="texatom" id="MathJax-Span-14"><span class="mrow" id="MathJax-Span-15"><span class="mi" id="MathJax-Span-16" style="font-family: MathJax_Main-bold;"> p</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.282em; border-left: 0px solid; width: 0px; height: 0.932em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi mathvariant="bold">p</mi></mrow></math></span></span><script type="math/tex" id="MathJax-Element-2">\mathbf{p}</script></span>的变化量</td><td>2.1</td></tr><tr><td>ddy</td><td>估算屏幕空间中的偏导数<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-3-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp; #x2202 ;</mi><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi mathvariant=&quot;bold&quot;>p</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mi mathvariant=&quot;normal&quot;>&amp; #x2202 ;</mi><mi>y</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-17" style="width: 3.546em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.803em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1002.8em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-18"><span class="mi" id="MathJax-Span-19" style="font-family: MathJax_Main;">∂<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="texatom" id="MathJax-Span-20"><span class="mrow" id="MathJax-Span-21"><span class="mi" id="MathJax-Span-22" style="font-family: MathJax_Main-bold;">p</span></span></span><span class="texatom" id="MathJax-Span-23"><span class="mrow" id="MathJax-Span-24"><span class="mo" id="MathJax-Span-25" style="font-family: MathJax_Main;">/</span></span></span><span class="mi" id="MathJax-Span-26" style="font-family: MathJax_Main;">∂<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mi" id="MathJax-Span-27" style="font-family: MathJax_Math-italic;">y<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">∂</mi><mrow class="MJX-TeXAtom-ORD"><mi mathvariant="bold">p</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mi mathvariant="normal">∂</mi><mi>y</mi></math></span></span><script type="math/tex" id="MathJax-Element-3">\partial \mathbf{p} / \partial y</script></span>。这使我们可以确定在屏幕空间的 y 轴方向上，相邻像素间某属性值<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-4-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi mathvariant=&quot;bold&quot;>p</mi></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-28" style="width: 0.803em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.66em, 1000.57em, 2.631em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-29"><span class="texatom" id="MathJax-Span-30"><span class="mrow" id="MathJax-Span-31"><span class="mi" id="MathJax-Span-32" style="font-family: MathJax_Main-bold;"> p</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.282em; border-left: 0px solid; width: 0px; height: 0.932em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi mathvariant="bold">p</mi></mrow></math></span></span><script type="math/tex" id="MathJax-Element-4">\mathbf{p}</script></span>的变化量</td><td>2.1</td></tr><tr><td>degrees</td><td>将 x 分量从弧度转换为角度制</td><td>1.1</td></tr><tr><td>determinant</td><td>返回方阵的行列式</td><td>1.1</td></tr><tr><td>distance</td><td>返回两个点的距离值</td><td>1.1</td></tr><tr><td>dot</td><td>返回两个向量的点乘</td><td>1.1</td></tr><tr><td>dst</td><td>计算距离向量</td><td>5.0</td></tr><tr><td>exp</td><td>计算 e^x</td><td>1.1</td></tr><tr><td>exp2</td><td>计算 2^x</td><td>1.1</td></tr><tr><td>floor</td><td>求不大于 x 分量的最大整数</td><td>1.1</td></tr><tr><td>fmod</td><td>求 x/y 的余数</td><td>1.1</td></tr><tr><td>frac</td><td>返回 x 分量的小数部分</td><td>1.1</td></tr><tr><td>isfinite</td><td>返回 x 分量是否为有限的布尔值</td><td>1.1</td></tr><tr><td>isinf</td><td>返回 x 分量是否为无穷大的布尔值</td><td>1.1</td></tr><tr><td>isnan</td><td>返回 x 分量是否为 nan 的布尔值</td><td>1.1</td></tr><tr><td>length</td><td>计算向量的长度</td><td>1.1</td></tr><tr><td>lerp</td><td>求 x + s (y - x)</td><td>1.1</td></tr><tr><td>lit</td><td>返回一个光照系数向量 (环境光亮度, 漫反射光亮度, 镜面光亮度, 1.0f)</td><td>1.1</td></tr><tr><td>log</td><td>返回以 e 为底，x 分量的对数</td><td>1.1</td></tr><tr><td>log10</td><td>返回以 10 为底，x 分量的对数</td><td>1.1</td></tr><tr><td>log2</td><td>返回以 2 为底，x 分量的自然对数</td><td>1.1</td></tr><tr><td>mad</td><td>返回 mvalue * avalue + bvalue</td><td>1.1</td></tr><tr><td>max</td><td>返回 x 分量和 y 分量的最大值</td><td>1.1</td></tr><tr><td>min</td><td>返回 x 分量和 y 分量的最小值</td><td>1.1</td></tr><tr><td>modf</td><td>将值 x 分开成整数部分和小数部分</td><td>1.1</td></tr><tr><td>mul</td><td>矩阵乘法运算</td><td>1</td></tr><tr><td>normalize</td><td>计算规格化的向量</td><td>1.1</td></tr><tr><td>pow</td><td>返回 x^y</td><td>1.1</td></tr><tr><td>radians</td><td>将 x 分量从角度值转换成弧度值</td><td>1</td></tr><tr><td>rcp</td><td>对每个分量求倒数</td><td>5</td></tr><tr><td>reflect</td><td>返回反射向量</td><td>1</td></tr><tr><td>refract</td><td>返回折射向量</td><td>1.1</td></tr><tr><td>reversebits</td><td>对每个分量进行位的倒置</td><td>5</td></tr><tr><td>round</td><td>x 分量进行四舍五入</td><td>1.1</td></tr><tr><td>rsqrt</td><td>返回 1/sqrt (x)</td><td>1.1</td></tr><tr><td>saturate</td><td>对 x 分量限制在 [0,1] 范围</td><td>1</td></tr><tr><td>sign</td><td>计算符号函数的值，x 大于 0 为 1，x 小于 0 为 - 1，x 等于 0 则为 0</td><td>1.1</td></tr><tr><td>sin</td><td>计算 x 的正弦</td><td>1.1</td></tr><tr><td>sincos</td><td>返回 x 的正弦和余弦</td><td>1.1</td></tr><tr><td>sinh</td><td>返回 x 的双曲正弦</td><td>1.1</td></tr><tr><td>smoothstep</td><td>给定范围 [min, max]，映射到值 [0, 1]。小于 min 的值取 0，大于 max 的值取 1</td><td>1.1</td></tr><tr><td>step</td><td>返回 (x&gt;= a) ? 1 : 0</td><td>1.1</td></tr><tr><td>tan</td><td>返回 x 的正切值</td><td>1.1</td></tr><tr><td>tanh</td><td>返回 x 的双曲正切值</td><td>1.1</td></tr><tr><td>transpose</td><td>返回矩阵 m 的转置</td><td>1</td></tr><tr><td>trunc</td><td>去掉 x 的小数部分并返回</td><td>1</td></tr></tbody></table>


全部函数： [https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions](https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions)
## clip，discard
通常，我们会在片元着色器中使用 clip 函数来进行透明度测试。
参数： 裁剪时使用的标量或矢量条件。
描述： 如果给定参数的任何一个分量是负数，就会舍弃当前像素的输出颜色。

```c
//定义
void clip(x)
{
	if(any(x < 0))
		discard;
}


//discard：丢弃像素
//在片段着色器中
if(input.uv > 0.5)
	dicard;

```

透明度测试中的常会添加一个阈值， `clip(texColor.a - _Cutoff)`，如果小于阈值，就裁剪掉。

**sin cos tan**：![[Pasted image 20221003190129.png]] ![[Pasted image 20221003190311.png]]
以 sin 函数为例，根据函数图像，小于 0 的值被截为 0，所以显示成黑色。白色区域由黑到白再到黑，对应着 sin 函数值的变化。

---
**Fract**：返回标量或矢量的小数
**Floor**：对输入参数向下取整。
**divide**：除以
![[Pasted image 20221003191651.png]]
uv 乘 4 后，fract 图 x 分量视为 0~4，取小数部分，即实现分成四块的0~1
floor 函数向下取整，0~1 部分取为 0，显示为黑色。1~4 取为1


**exp (x)**
**length (v)**
**distance (v1, v2)**：两点之间的欧几里德距离 ![[Pasted image 20221003192605.png]]



`saturate(v)`: 将 v 夹取到 $[0,1]$ 区间.
`clamp(v,min,max)`: 将 v 夹取到 $[min, max]$ 区间


# fmod，frac
fmod (x, y): 返回 x / y 的小数部分. 如: x = i * y + f
frac (x): 返回 x 的小数部分.
# step，lerp，smoothstep 

`step(x,y):` 如果x<=y，则输出 1，否则输出 0
`lerp(x,y,a)` ：插值 `x*(1-a) + y*a`
`smoothstep (min, max, x)`: 用来生成 0 到 1 的平滑过渡值，也叫平滑阶梯函数.

公式定义:
```c
float smoothstep (float min, float max, float x)
{
    x = clamp ((x - min) / (max - min), 0, 1);
    return x * x * (3 - 2 *x);
}
```
函数曲线:

1.  当 min < max

![[08ff83d421adfc01c4b17008f463ebe0_MD5.jpg]]

1.  当 min > max

![[df365600ecaefb116a098f55c9aebeea_MD5.png]]

应用举例:

可以通过多个 smoothstep 叠加 / 相减，构造一些波形曲线.

如 smoothstep (0,1, x) -smoothstep (1,2, x) 的波形

![[0ace704966b5b4ba1fb1a8a52a0a8717_MD5.png]]

# reflect，refract

`reflect(i,n)`: 计算反射向量 (i 和 n 必须是归一化的)，使用公式 i - 2*n*dot (i, n)
`refract (i,n,eta)`: 计算折射向量 (i 和 n 必须是归一化的)，eta 为折射系数

# ddx，ddy，fwidth
## 定义

**GPU 在光栅化的时候一般以 2x2 的像素块为单位并行执行的。** 在这个 2x2 像素块当中，右侧的像素对应的 fragment 的 x 坐标减去左侧的像素对应的 fragment 的 x 坐标就是 ddx；下侧像素对应的 fragment 的坐标 y 减去上侧像素对应的 fragment 的坐标 y 就是 ddy。  

ddx , ddy 的计算规则如下图（注: dFdx, dFdy 是 GLSL 里的叫法）

![[842872b87359c92a610ee9f99bba8127_MD5.png|450]]
1. **`ddx，ddy` 反映了相邻像素在屏幕空间 x 和 y 方向上的距离（变化率）**
注：ddx (常量) = 0; 因为没有差值变化
- ddx (v) = 该像素点右边的 v 值 - 该像素点的 v 值  
- ddy (v) = 该像素点下面的 v 值 - 该像素点的 v 值
1.  `fwidth(v) = abs(ddx(v) + ddy(v))`：**fwidth 反映了相邻像素在屏幕空间上的距离差值**.
    - `fwidth (pos)`：相邻像素位置的差值
    - `fwidth (normal)`：相邻像素法线的差值
    - `fwidth (uv)`：相邻像素 uv 的差值

## 应用举例：
### 计算 Mipmap Level

当相机离目标物体比较远时，当目标物体只占屏幕很小一块区域 (pixels), 这时如果还使用正常尺寸的 texture 对目标物体进行渲染，由于此时一个 pixel 对应多个 texel，导致采样不足出现走样。使用 Mipmap 就能解决这个问题，整个 texture 大小为原来的 1.33 倍，选择合适的 level，这样尽可能让 pixel 和 texel 一一对应 (同时也能减少 Cache Miss)。

![[8e9605977f310c6b72db592098f3445f_MD5.jpg]]

**Level 计算公式：**

$$
\rho=\max\left\{\sqrt{\left(\frac{\partial u}{\partial x}\right)^2+\left(\frac{\partial v}{\partial x}\right)^2+\left(\frac{\partial u}{\partial x}\right)^2},\sqrt{\left(\frac{\partial u}{\partial y}\right)^2+\left(\frac{\partial v}{\partial y}\right)^2+\left(\frac{\partial w}{\partial y}\right)^2}\right\}
$$

$$Mipmap Level = log_2 (ρ)$$

注：$w$ 是 3D 贴图的第三个坐标轴，对于 2D 贴图，$∂w/∂x$ 和$∂w/∂y$ 为 $0$;

$∂u/∂x$ 是 $u$ 对 $x$ 的偏微分 (也就是 $u$ 沿 $x$ 轴的变化率), 即 $ddx (u)$, 可以理解成当屏幕像素沿 $x$ 轴变换一个单位时，贴图沿 $u$ 方向变化了几个单位.

![[b5f733798149019e66cd179c37beec6d_MD5.png]]



### 边缘处理

让颜色边缘突出:

finalColor += (ddx (finalColor) + ddy (finalColor )) * _Intensity;

或 finalColor += fwidth (finalColor) * _Intensity;

3. **计算表面法线**

normal = normalize (cross (ddy (IN. worldPos), ddx (IN. worldPos)));

![[189df4a6706aa9f8085c04bbfb6df2c6_MD5.png]]


# ddx, ddy

**常见使用：**  
**边缘突出**  
ddx (v) = 该像素点右边的 v 值 - 该像素点的 v 值  
ddy (v) = 该像素点下面的 v 值 - 该像素点的 v 值
![[Pasted image 20230722233358.png]]
![[Pasted image 20230722233401.png]]
**边缘亮化**  

![[Pasted image 20230722233410.png]]
![[Pasted image 20230722233412.png]]