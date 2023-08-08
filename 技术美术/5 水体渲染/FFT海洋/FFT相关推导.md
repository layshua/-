# 生成服从标准生成分布的随机数
在 fft 海面模拟中，需要生成服从标准生成分布的随机数（见：[杨超：fft 海面模拟 (一)](https://zhuanlan.zhihu.com/p/64414956)）。

我们知道数构造服从目标分布的随机数发生器，一般是采用反函数法：对目标分布的 CDF 求反函数，则当其输入服从 U (0,1) 的随机数时，就返回服从目标分布的随机数。（参考 pbrt 第三版第 13 章第 3 节）。

不过由于正态分布的 CDF 无法用初等函数表达，所以上面方法走不通，需另辟蹊径。

通常使用称为 Box-Muller 转换（Box-Muller transform）的方法来生成服从标准正态分布的随机数（对儿）。

## **一，Box-Muller 转换的基本形式（basic form）**

设 U1, U2 相互独立，均服从分布 U (0,1)

X= $Rcos\Theta=\sqrt{-2lnU_1}cos(2\pi U_2)$

Y= $Rsin\Theta=\sqrt{-2lnU_1}sin(2\pi U_2)$

则 X、Y 相互独立且均服从标准正态分布。

证明：

U1, U2~U (0,1) 所以 $p_{U_1}(u_1)=p_{U_2}(u_2)=1$。

因为 $\Theta=2\pi U_2$ ，根据一元随机变量函数分布（见：[杨超：pbrt 注解：transforming between distributions](https://zhuanlan.zhihu.com/p/67446317)）有：

$p_\Theta(\theta)=|\frac{d\theta}{du_2}|^{-1}p_{U_2}(u_2)$

$=|\frac{d(2 \pi u_2)}{du_2}|^{-1}p_{U_2}(u_2)$

$=\frac{1}{2 \pi}$

因为 $R=\sqrt{-2lnU_1}$ ，同样，根据一元随机变量函数分布，有：

$p_R(r)=|\frac{dr}{du_1}|^{-1}p_{U_1}(u_1)$

$=|\frac{d\sqrt{-2lnu_1}}{du_1}|^{-1}*1$

$=u_1\sqrt{-2lnu_1}$

(因为 $r=\sqrt{-2lnu_1}$ ，所以 $u_1=e^{-\frac{r^2}{2}}$，所以 )

$=r*e^{-\frac{r^2}{2}}$

因为 U1、U2 互相独立，而 $\Theta$ 只取决于 U2，R 只取决于 U1，所以 $\Theta$ 、R 互相独立。而独立随机变量联合分布等于边缘分布之积。所以

$p_{\Theta,R}(\theta,r)=p_{\Theta}(\theta)*p_R(r)=\frac{1}{2 \pi}r*e^{-\frac{r^2}{2}}$

因为

$X=Rcos\Theta$

$Y=Rsin\Theta$

根据二元随机变量函数分布（见：[杨超：pbrt 注解：transforming between distributions](https://zhuanlan.zhihu.com/p/67446317)），有：

$p_{X,Y}(x,y)=|J|^{-1}p_{\Theta,R}(\theta,r)$

其中

$J=\begin{vmatrix} \frac{\partial x}{\partial \theta} &\frac{\partial x}{\partial r} \\ \frac{\partial y}{\partial \theta} &\frac{\partial y}{\partial r} \end{vmatrix}$

$=\begin{vmatrix} \frac{\partial (r*cos\theta)}{\partial \theta} &\frac{\partial (r*cos\theta)}{\partial r} \\ \frac{\partial (r*sin\theta)}{\partial \theta} &\frac{\partial (r*sin\theta)}{\partial r} \end{vmatrix}$

$=\begin{vmatrix} -r*sin\theta & cos\theta \\ r*cos\theta&sin\theta \end{vmatrix}$

$=-r*sin^2\theta-r*cos^2\theta$

$=-r$

所以

$p_{X,Y}(x,y)=|-r|^{-1}*\frac{1}{2 \pi}r*e^{-\frac{r^2}{2}}$

$=\frac{1}{2 \pi}e^{-\frac{r^2}{2}}$

(因为 $r^2=x^2+y^2$ ，所以)

$=\frac{1}{2 \pi}e^{-\frac{x^2+y^2}{2}}$

所以

$p_X(x)=\int_{-\infty}^{\infty}p_{X,Y}(x,y)dy$

$=\int_{-\infty}^{\infty}\frac{1}{2\pi}e^{-\frac{x^2+y^2}{2}}dy$

$=\frac{1}{2\pi}e^{-\frac{x^2}{2}}\int_{-\infty}^{\infty}e^{-\frac{y^2}{2}}dy$

$=\frac{1}{2\pi}e^{-\frac{x^2}{2}}*\sqrt{2\pi}*\int_{-\infty}^{\infty}\frac{1}{\sqrt{2\pi}}e^{-\frac{y^2}{2}}dy$

(注意其中积分正好是标准正态分布密度函数的积分，故积分结果为 1)

$=\frac{1}{2\pi}e^{-\frac{x^2}{2}}*\sqrt{2\pi}*1$

$=\frac{1}{\sqrt{2\pi}}e^{-\frac{x^2}{2}}$

同理可得：

$p_Y(y)=\int_{-\infty}^{\infty}p_{X,Y}(x,y)dx=\frac{1}{\sqrt{2\pi}}e^{-\frac{y^2}{2}}$

可见 X 和 Y 均服从标准正态分布。又因可验证 $p_{X,Y}(x,y)=p_X(x)*p_Y(y)$ ，所以 X, Y 互相独立。

证毕。

## **二，Box-Muller 转换的极坐标形式（polar form）**

设 u, v 相互独立，均服从分布 U (-1,1)，但拒绝掉 $u^2+v^2=0$ 和 $u^2+v^2\geq1$ 的 (u, v) 对儿。

X= $u*\sqrt{\frac{-2lns}{s}}$

Y= $v*\sqrt{\frac{-2lns}{s}}$

其中 $s=u^2+v^2$

则 X、Y 相互独立且均服从标准正态分布。

证明：

实际上在 Box-Muller 转换的标准形式

X= $\sqrt{-2lnU_1}cos(2\pi U_2)$

Y= $\sqrt{-2lnU_1}sin(2\pi U_2)$

中，作变换 (*)：

$U_1=s$

$cos(2\pi U_2)=\frac{u}{\sqrt s}$

$sin(2\pi U_2)=\frac{v}{\sqrt s}$

即得

X= $u*\sqrt{\frac{-2lns}{s}}$

Y= $v*\sqrt{\frac{-2lns}{s}}$

但问题是上述变换 (*) 能满足 “U1, U2 相互独立，均服从分布 U (0,1)” 吗？

验证如下：

（1）首先确认 U1, U2 的取值范围均为 (0,1)：

![](https://pic2.zhimg.com/v2-f7a6610eebff35f3742aea5629912ed5_r.jpg)

如图，因为 (u, v) 均匀分布在单位圆内，而

$U_1=s=r^2$

故 U1 取值范围为 (0,1)。

又由图中看出 $2\pi U_2$ 即为辐角θ，因为θ取值范围为 $(0,2\pi)$ ，故 U2 取值范围为 (0,1)。

（2）再看 U1, U2 的分布：

因为 (u, v) 均匀分布在单位圆内，故 $p_{u,v}(u,v)=\frac{1}{\pi}$ 。

根据二元随机变量函数分布，有：

$p_{U_1,U_2}(u_1,u_2)=|J|^{-1}p_{u,v}(u,v)$

其中

$J=\begin{vmatrix} \frac{\partial u_1}{\partial u} &\frac{\partial u_1}{\partial v} \\ \frac{\partial u_2}{\partial u} &\frac{\partial u_2}{\partial v} \end{vmatrix}$

$\frac{\partial u_1}{\partial u}$ 和 $\frac{\partial u_1}{\partial v}$ 好求：

$\frac{\partial u_1}{\partial u}=\frac{\partial s}{\partial u}=\frac{\partial (u^2+v^2)}{\partial u}=2u$

$\frac{\partial u_1}{\partial v}=\frac{\partial s}{\partial v}=\frac{\partial (u^2+v^2)}{\partial v}=2v$

$\frac{\partial u_2}{\partial u}$ 和 $\frac{\partial u_2}{\partial v}$ 则需隐函数求导：

将 $cos(2\pi u_2)=\frac{u}{\sqrt s}$ 两边对 v 求导，得：

$-sin(2\pi u_2)*2\pi*\frac{\partial u_2}{\partial v}=-\frac{uv}{s\sqrt s}$

解得：

$\frac{\partial u_2}{\partial v}=\frac{uv}{s\sqrt s *sin(2\pi u_2)*2\pi}$

(因为 $sin(2\pi u_2)=\frac{v}{\sqrt s}$ ，所以)

$=\frac{uv}{s\sqrt s *(v/\sqrt s)*2\pi}$

$=\frac{u}{2\pi s}$

同理，将 $sin(2\pi u_2)=\frac{v}{\sqrt s}$ 两边对 u 求导，可解得：

$\frac{\partial u_2}{\partial u}=\frac{-v}{2\pi s}$

所以：

$J=\begin{vmatrix} \frac{\partial u_1}{\partial u} &\frac{\partial u_1}{\partial v} \\ \frac{\partial u_2}{\partial u} &\frac{\partial u_2}{\partial v} \end{vmatrix}$

$=\begin{vmatrix}2u &2v \\ \frac{-v}{2\pi s} &\frac{u}{2\pi s} \end{vmatrix}$

$=\frac{u^2+v^2}{s\pi}$

(因为 $u^2+v^2=s$ ，所以)

$=\frac{1}{\pi}$

所以

$p_{U_1,U_2}(u_1,u_2)=|J|^{-1}p_{u,v}(u,v)$

$=|\frac{1}{\pi}|^{-1}*\frac{1}{\pi}$

$=1$

所以

$p_{U_1}(u_1)=\int_{-\infty}^{\infty}p_{U_1,U_2}(u_1,u_2)du_2$

(因为 U2 取值范围为 (0,1)，所以)

$=\int_{0}^{1}1du_2$

$=1$

同理

$p_{U_2}(u_2)=\int_{-\infty}^{\infty}p_{U_1,U_2}(u_1,u_2)du_1=\int_{0}^{1}1du_1=1$

故 U1，U2 均服从 U (0,1)。

又 $p_{U_1,U_2}(u_1,u_2)=p_{U_1}(u_1)*p_{U_2}(u_2)$ ，故 U1, U2 相互独立。

证毕。

## **三，代码**

采用极坐标形式：

```
public Vector2 gaussianRandomVariablePair() {
		float x1, x2, w;
		do {
			x1 = 2f * Random.Range(0f,1f) - 1f;
			x2 = 2f * Random.Range(0f,1f) - 1f;
			w = x1 * x1 + x2 * x2;
		} while ( w >= 1f );
		w = Mathf.Sqrt((-2f * Mathf.Log(w)) / w);
		return new Vector2(x1 * w, x2 * w);
	}
```

# 法线的解析式推导
用差分就可以求法线，但那样求出来的不是很精确。最精确的方法是直接推出法线的解析式。

差分方法：假设我们想要计算点 $M_0$ 处的法线，我们需要计算两个切向量 $T_x$ 和 $T_y$ ，然后两者叉积就可以得到我们的法线。**对于求切向量就是对曲面方程求偏导就可以得到，因为我们这都是离散的点，可以直接取 $M_0$ 两侧的点，做差就可以求到。
![[a0c4e435a42580c72c32e743c3526dd2_MD5.jpg|450]]


**推解析式：**
因为高度是：

$h(\vec{x},t)=\sum_{\vec{k}}^{}{\tilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}}$

其空间梯度为：

$\triangledown h(\vec{x},t)=\sum_{\vec{k}}^{}{\tilde{h}(\vec{k},t)\triangledown e^{i\vec{k}\cdot\vec{x}}}$ （因为 $\tilde{h}(\vec{k},t)$ 并不包含空间变元，故可从梯度符号内移出）

而其中

$\triangledown e^{i\vec{k}\cdot\vec{x}}=(\frac{\partial e^{i(k_x*x+k_z*z)}}{\partial x},\frac{\partial e^{i(k_x*x+k_z*z)}}{\partial z})$

$=(e^{i(k_x*x+k_z*z)}*ik_x,e^{i(k_x*x+k_z*z)}*ik_z)$

$=e^{i(k_x*x+k_z*z)}*i(k_x,k_z)$

$=i\vec{k}e^{i\vec{k}\cdot\vec{x}}$

所以

$\triangledown h(\vec{x},t)=\sum_{\vec{k}}^{}{i\vec{k}\tilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}}$

又梯度向量、up 向量、法向量三者具有下图所示几何关系：

![[83cffe3c57011f127bc4444be5b1d538_MD5.jpg]]

所以有（注意，normalize 是必要的）：

$\vec{N}=normalize((0,1,0)-(\triangledown {h}_x(\vec{x},t),0,\triangledown{h}_z(\vec{x},t)))$

$=normalize(-\triangledown {h}_x(\vec{x},t),1,-\triangledown {h}_z(\vec{x},t))$


# 浪尖泡沫推导
在 gerstner wave 中，为了表现波峰尖角，使用下面公式在 xz 平面内进行挤压（红框中部分）：

![[46e4202e2b55209453431d647a793bbb_MD5.jpg]]

此处 IDFT 海面，同样需要类似挤压操作，公式为：

$\vec{D}(\vec{x},t)=\sum_{\vec{k}}^{}{-i\frac{\vec{k}}{k}\tilde{h}(\vec{k},t)e^{i\vec{k}\cdot\vec{x}}}$

$\vec{x}^{,}=\vec{x}+\lambda \vec{D}(\vec{x},t)$

不难看出，两者虽然写法不同，含义是一样的：**即对 sin 波进行 cos 挤压，对 cos 波进行 sin 挤压。**

**当 xz 平面内挤压过头时，就会出现刺穿（如图所示）。恰好对应浪尖破碎形成泡沫的区域。**

![[544cbe481f20abfeeedfbcd9777dea24_MD5.jpg]]

当发生刺穿时，局部发生翻转，表现在数学上，即面元有向面积变为负值。

**那么如何求面元有向面积呢？** 同济高数下册里学过：二重积分换元法，雅可比行列式。[[FFT相关推导#二重积分换元法、雅可比行列式]]

因为 x'和 z'均为以 x, z 为变元的二元函数，即 x'=x' (x, z), z'=z' (x, z)，由二重积分换元法知变换后面积元为：

$dA=\vec{dx'}\times \vec{dz'}= \begin{vmatrix} \frac{\partial x'}{\partial x} & \frac{\partial x'}{\partial z}\\ \frac{\partial z'}{\partial x}& \frac{\partial z'}{\partial z} \end{vmatrix}dxdz$

由于 dxdz 必定为正数，所以 dA 的正负就取决于雅可比行列式

$J(\vec{x})=\begin{vmatrix} J_{xx} &J_{xz} \\ J_{zx} & J_{zz} \end{vmatrix}= \begin{vmatrix} \frac{\partial x'}{\partial x} & \frac{\partial x'}{\partial z}\\ \frac{\partial z'}{\partial x}& \frac{\partial z'}{\partial z} \end{vmatrix}$

的正负。

由于 $\vec{x}^{,}=\vec{x}+\lambda \vec{D}(\vec{x},t)$ ，所以：

$J_{xx}=\frac{\partial x'}{\partial x}=1+\lambda\frac{\partial D_x(\vec{x},t)}{\partial x}$

$J_{zz}=\frac{\partial z'}{\partial z}=1+\lambda\frac{\partial D_z(\vec{x},t)}{\partial z}$

$J_{zx}=\frac{\partial z'}{\partial x}=\lambda\frac{\partial D_z(\vec{x},t)}{\partial x}$

$J_{xz}=\frac{\partial x'}{\partial z}=\lambda\frac{\partial D_x(\vec{x},t)}{\partial z}$

由于我们有 $\vec{D}(\vec{x},t)$ 的表达式，所以其实上面各偏导数都是可以求出来的。

如果真的去求，我们会发现 $J_{xz}=J_{zx}$，亦即 $\frac{\partial D_z(\vec{x},t)}{\partial x}=\frac{\partial D_x(\vec{x},t)}{\partial z}$ ，验证如下：

![[290e778049f1613a582f1da2442ca948_MD5.jpg]]


# 二重积分换元法、雅可比行列式

二重积分换元法同济高数下册有讲，当时没细看证明，近来用到搜了一下，感觉下面这样推比较直观：

![[dcfa47b288ff779b3e0f287006004b2b_MD5.jpg]]

$\vec{dx}=\vec{P'Q'}=Q'-P'$

$=f(Q)-f(P)$

$=f(u+du,v)-f(u,v)$  
$=\frac{f(u+du,v)-f(u,v)}{du}du$

$=\binom{\frac{x(u+du,v)-x(u,v)}{du}}{\frac{y(u+du,v)-y(u,v)}{du}}du$  
$=\binom{\frac{\partial x}{\partial u}du}{\frac{\partial y}{\partial u}du}$  
同理可得：

$\vec{dy}=\binom{\frac{\partial x}{\partial v}dv}{\frac{\partial y}{\partial v}dv}$

所以变换后面积元

$dA=\vec{dx}\times \vec{dy}$  
$=\binom{\frac{\partial x}{\partial u}du}{\frac{\partial y}{\partial u}du}\times \binom{\frac{\partial x}{\partial v}dv}{\frac{\partial y}{\partial v}dv}$

$=\begin{vmatrix} \frac{\partial x}{\partial u} & \frac{\partial x}{\partial v}\\ \frac{\partial y}{\partial u}& \frac{\partial y}{\partial v} \end{vmatrix}dudv$

