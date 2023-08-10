在阅读这篇文章前，最好提前了解：

*   [游戏开发中的数学：傅里叶变换 (FT) 篇](https://zhuanlan.zhihu.com/p/619856201)
*   [离散傅里叶变换 (DFT)](https://zhuanlan.zhihu.com/p/620462217)
*   [FT 在图像处理中的应用](https://zhuanlan.zhihu.com/p/620559433)

### 5.1 FFT 海洋公式：二维 IDFT

[GPGPU_FFT_Ocean_Simulation.pdf](https://tore.tuhh.de/bitstream/11420/1439/1/GPGPU_FFT_Ocean_Simulation.pdf)

既然任意连续且收敛的函数 $f(x)$ 都可以分解为无数个不同频率、不同幅值的正、余弦信号，且这个性质可以扩展到二维，即任意一个二维图像都可以分解为无数个复平面波 $e^{-i 2 \pi(u x+v y)}$ 的叠加

$F(u, v)=\int_{-\infty}^{\infty} \int_{-\infty}^{\infty} f(x, y) e^{-i 2 \pi(u x+v y)} d x d y$

![[34a6579351486bbd55c58689800d8cf5_MD5.jpg]]

那么海浪作为 “波” 的典型，是否也可以将其拆成任意方向不同频率强度的基波的叠加？当然没有问题：令 $h(\vec{x}, t)$ 为 $t$ 时刻，位置 $\vec{x}=(x,z)$ 上海面的高度，其构建公式就为  
$h(\vec{x}, t)=\sum_k \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$

可以看出，**这正是二维离散傅里叶变换的逆变换 (IDFT) 版本**，只不过多了一个量度即时间，考虑到 DFT 形式下任意 $\vec{k}$ 将对应所有可能的复平面波 $e^{-i 2 \pi(n x+m z)}$ ，就能顺理成章的推出波矢量

$\vec{k}=\left(k, l\right)=\left(\frac{2 \pi k}{L}, \frac{2 \pi l}{L}\right)$

其中常量 $L$ 为海平面大小，采样点 $k, l$ 满足 $-N / 2 \leq k, l$ ，N 为采样点数量，当然也可以做个变换，即将范围映射到 $0<k, l<N-1$ ，此时

$\vec{k}=\left(\frac{2 \pi k-\pi N}{L}, \frac{2 \pi l-\pi N}{L}\right)$ 本质不变

代入上面的海洋方程，并将所有向量全部展开，为方便可以直接任 $N=L$ 就有

$h(x, z, t)=\frac{1}{N^2} \sum_{k=0}^{N-1} \sum_{l=0}^{N-1} \tilde{h}(k, l, t)e^{i \frac{2 \pi k x-\pi N x}{N}}e^{ i \frac{2 \pi l z-\pi N z}{N}} \\ =\frac{1}{N^2} \sum_{k=0}^{N-1} \sum_{l=0}^{N-1} \tilde{h}(k, l, t)(-1)^x e^{ i \frac{2 \pi kx}{N}}(-1)^z e^{i \frac{2 \pi lz}{N}} \\ \frac{1}{N^2}(-1)^x \sum_{k=0}^{N-1}\left[(-1)^z \sum_{l=0}^{N-1} \tilde{h}(k, l, t) e^{i \frac{2 \pi lz}{N}}\right] e^{i \frac{2 \pi kx}{N}}$

当然也可以任 $N\neq L$ ，这在计算方式上是相同的

### 5.1.1 二维 IFFT

前面已经详细介绍过了一维 FFT 的原理和计算过程，这里同样要扩展二维以得到最终的 $h(\vec{x}, t)$ ，其实看这个二维 IDFT 式子，你就会发现结论很明显了：

$h(x, z, t)=\frac{1}{N^2}(-1)^x \sum_{k=0}^{N-1}\left[(-1)^z \sum_{l=0}^{N-1} \tilde{h}(k, l, t) e^{i \frac{2 \pi lz}{N}}\right] e^{i \frac{2 \pi kx}{N}}$

它其实是可以拆成两个一维的 IDFT 的：

*   $h'(k,z,t)=\frac{(-1)^z}{N} \sum_{l=0}^{N-1} \tilde{h}(k, l, t) e^{i \frac{2 \pi lz}{N}}$
*   $h(x, z, t)=\frac{(-1)^x}{N} \sum_{k=0}^{N-1}h'(k,z,t)e^{i \frac{2 \pi kx}{N}}$

那么最终我们计算海洋公式的步骤就应如下：

1.  计算 $\tilde{h}(k, l, t)$ 生成频谱，如果是交给 GPU 计算，可以将结果存储在一张 NxN 的纹理中（关于 $\tilde{h}(k, l, t)$ 的具体内容及计算方式可以参考下一节 5.2 的内容），t 可以理解成第几帧，必然每一帧这个图像都会不同
2.  执行 N 次水平 FFT： $\sum_{l=0}^{N-1} \tilde{h}(k, l, t) e^{i \frac{2 \pi lz}{N}}$ ，上一步的纹理作为输入，输出结果作为下一步的输入
3.  执行 N 次纵向 FFT： $\sum_{k=0}^{N-1}h'(k,z,t)e^{i \frac{2 \pi kx}{N}}$ ，得到的结果再进行最后的符号计算：即乘上 $\frac{(-1)^x(-1)^y}{N^2}$ ，搞定

![[d11274d7e9b9e52f3287c324eb311473_MD5.jpg]]

到此，我们就拿到了一张高度图，可以用于后续的顶点动画及渲染着色了

### 5.2 海浪统计模型

事实上，这种基于 FFT 或者说基于频谱逆向推出海洋水面高度的方法，本质上是一种统计模型方法，这里面涉及不少其他领域内的知识，例如海洋物理学，又或是对现实海洋的实时观测统计与实验，我们当然不用深入了解这部分的内容，只要拿对应的统计结果来用就可以了

上面的内容还有一个很重要的东西还未知，就是用于表达海洋波的频率的集合、IDFT 中的频域函数 $\tilde{h}(\vec{k}, t)$ ，搞定了这个之后，我们才能通过 IDFT 算法将频谱转化为时域的值，从而得到海洋随时间变化的高度图

公式都给你写好了，照着抄就行

### 5.2.1 [2004] 菲利浦频谱 (Tenssendorf Phillips spectrum)

基于风运动的菲利浦频谱（也被称作 Tessendorf 频谱）来自海洋动力学的成果，是一个反馈了真实海洋的经验模型，也是这里一切的核心，当然这个模型并不唯一

对于菲利浦频谱 $P_h(\vec{k})=S(\omega)D(\omega, \theta)=A \frac{e^{-1 /(k L)^2}}{k^4}|\vec{k} \cdot \vec{\omega}|^2$ ，其中

*   $L=\frac{V^2}{ g}$ ， $V$ 是风速， $\vec{\omega}$ 为风向， $g$ 为引力常数 $9.8 \mathrm{~m} / \mathrm{sec}^2$
*   $k$ 为 $\vec{k}$ 的幅值，即频率大小

从论文得到的信息是：菲利浦频谱主要由两个单独部分组成，分别为

1.  非定向能量波谱 (Energy Spectrum) $S(\omega)=A\frac{e^{-1 /(k L)^2}}{k^4}$ ，由海洋统计分析得出，其描述了具有特定频率的波浪有多凶猛，与波的方向无关
2.  方向扩展函数 (Directional Spread) $D(\omega, \theta)=|\vec{k} \cdot \vec{\omega}|^2$ ，体现的是风向对波最终幅值的贡献，很明显：垂直于风向的波浪几乎不易察觉，而方向与风向一致的波浪往往异常凶猛

### 5.2.2 [2015] TMA JONSWAP 频谱与 Hasselman 方向扩展

[Empirical directional wave spectra for computer graphics](https://dl.acm.org/doi/10.1145/2791261.2791267)

然而菲利浦频谱仍存在一些缺陷，其非方向和方向谱在观察得到的海洋行为的经验拟合性较差，方向拓展在波长较大的情况下收敛性也不够，能调整的参数也有限

和 PBR 中的 BRDF 一样，海洋频谱的物理模型也必然不止一种，2015 年论文《Empirical directional wave spectra for computer graphics》中详细提到了波谱的构造方式，并提供了几种非方向波谱模型和方向拓展函数（方向谱），同时引入了一个 Swell 参数来调整方向拓展函数从而能制作更长更平缓的波浪

[皮尔森 - 莫斯科维茨波谱 (Pierson-Moskowitz Spectrum)](https://wikiwaves.org/Ocean-Wave_Spectra)

如果海风在一段很长的时间内都处于一个稳定的大小与状态，那么海浪也会风达成平衡，基于这个假设，Pierson 及 Moskowitz 使用了北大西洋英国气象船上的速度传感器对海浪进行的测量，并计算拟合了同风速下的海洋的非定向能量波谱： $S(\omega)=\frac{\alpha g^2}{\omega^5} e^{ -\beta\left(\frac{\omega_0}{\omega}\right)^4}$

其中 $\alpha=8.1 \times 10^{-3}$，$\beta=0.74$ ，$\omega_0=g / U_{19.5}$，$U_{19.5}$ 为距离海洋 19.5m 处高度的风速

![[9641c87aa682e88203977272e097e874_MD5.jpg]]

但还没有结束，经过他们的专业分析后（这部分析略过，有兴趣直接看论文）发现风力与海浪流速其实是一个非常复杂的非线性关系，并且几乎不可能去精确计算，因此后续去做了一些拟合，得出了改良的 JONSWAP 频谱：

$\begin{aligned} S_j(\omega) & =\frac{\alpha g^2}{\omega^5} \exp \left[-\frac{5}{4}\left(\frac{\omega_p}{\omega}\right)^4\right] \gamma^r \\ r & =\exp \left[-\frac{\left(\omega-\omega_p\right)^2}{2 \sigma^2 \omega_p^2}\right] \end{aligned}$

即原先的皮尔森 - 莫斯科维茨波谱 (Pierson-Moskowitz Spectrum) 乘上额外的峰值增强因子 $\gamma^r$  
其中经过详尽的测量， $F$ 为离背风岸的距离，即风场，可以理解为风以恒定速度吹过的距离，有：

$\begin{aligned} \alpha & =0.076\left(\frac{U_{10}^2}{ gF}\right)^{0.22} \\ \omega_p & =22\left(\frac{g^2}{U_{10} F}\right)^{1 / 3} \\ \gamma & =3.3 \\ \sigma & = \begin{cases}0.07 & \omega \leq \omega_p \\ 0.09 & \omega>\omega_p\end{cases} \end{aligned}$

JONSWAP 频谱的一个特点就是：波浪随着时间或距离的推移，其峰值强度也会越来越大，当然你也可以再乘上一个 TMA 矫正，这用于对深海海浪强度进行弱化调整：

$S_{\text {TMA }}(\omega)=S_{\text {JONSWAP }}(\omega) \Phi(\omega, h)$ ，其中 h 为水深

$\begin{aligned} \omega_h&=\sqrt{\frac{\omega^2 h}{g}} \\ \Phi(\omega, h) &\approx \begin{cases}\frac{1}{2} \omega_h^2 & \text { if } \omega_h \leq 1 \\ 1-\frac{1}{2}\left(2-\omega_h\right)^2 & \text { if } \omega_h>1\end{cases} \end{aligned}$

然后就是方向扩展函数 $D(\omega, \theta)$ 的公式选择，一个非常简单的经验模型如下：

$D_{\cos ^2}(\theta)= \begin{cases}\frac{2}{\pi} \cos ^2(\theta) & \text { if } \frac{-\pi}{2}<\theta<\frac{\pi}{2} \\ 0 & \text { otherwise }\end{cases}$ ，这个模型简单暴力，直接根据风向角度加大对于同向或者同逆风向的波浪强度，但由于其缺乏足够的数据支持，看上去也不美观，所以后面就很少用了

**哈塞尔曼方向扩展 (Hasselmann Directional Spreading)**

到目前为止，所有海浪的产生都和当地的风场有关，但其实还有一类型的浪容易被忽视，那就是**涌浪 (Swell)**：涌浪和一般的波浪不同，其是由产生风浪的海区传播到另一海区的波浪或风平息之后残留于海面的余浪，形状特点较风浪规则，呈对称波形，波顶平缓，周期长，除此之外，涌浪在传播过程中会出现波高逐渐降低，能量减少，波长变长的现象

为了模拟来自远处风事件的涌浪的影响，需要设计一个函数 $D_{\xi}(\omega, \theta)$ ，其随着波的波长增加而逐渐增大其推进速度及涌浪高度：

$\begin{aligned} D_{\xi}(\omega, \theta) & =Q_{\xi}\left(s_{\xi}\right)|\cos (\theta / 2)|^{2 s_{\xi}} \\ s_{\xi} & =16 \tanh \left(\frac{\omega_p}{\omega}\right) \xi^2 \end{aligned}$

其中 $s= \begin{cases}6.97\left(\omega / \omega_p\right)^{5} & \omega \leq \omega_p \\ 9.77\left(\omega / \omega_p\right)^{2.5} & \omega>\omega_p\end{cases}$

$\xi$ 为一个 0 到 1 之间的值，用于控制整个涌浪的强度，当然 Hasselmann Directional Spreading 也只不过是众多方向拓展函数的其中一种，更多的可以依旧可以去论文上找

这一节不再做过多的介绍了，对于这些公式只需要会用就可以

### 5.2.3 海洋初始状态

回到前面的 $\tilde{h}(\vec{k}, t)$ ，可以先考虑它的初始状态，也就是 t = 0 的时刻的振幅 $\tilde{h_0}(\vec{k_0})$

$\tilde{h}_0(\vec{k})=\frac{1}{\sqrt{2}}\left(\xi_r+i \xi_i\right) \sqrt{P_h(\vec{k})}$

$\xi_r$ 和 $\xi_i$ 为两个相互独立的服从均值为 0，标准差为 1 的高斯随机数，实部表初始幅度，虚部表初始相位，而 $P_h(\vec{k})$ 正是前面的方向波谱

这里再介绍另一个[案例](https://www.youtube.com/watch?v=kGEqaX4Y4bQ)，其实后面的 FFT 实现也正是借鉴的这篇视频，这个案例中额外考虑了频谱到波的离散转化：  
$\tilde{h}_0(\vec{k})=\frac{1}{\sqrt{2}}\left(\xi_1+i \xi_2\right) \sqrt{2 S(\omega) D(\theta, \omega) \frac{d \omega(k)}{d k} \frac{1}{k} \Delta k_x \Delta k_z}$

### 5.2.4 海洋公式最终形式

最后就是把时间参数加上：

$\tilde{h}(\vec{k}, t)=\tilde{h}_0(\vec{k}) e^{i \omega(k) t}+\tilde{h}_0^*(-\vec{k}) e^{-i \omega(k) t}$ ，这里又有

*   角频率与波长的频散关系 $\omega(k)$ 满足 $\omega(k)=\sqrt{g k}$ ，其考虑了引力风波和重力场的关系，当然如果额外考虑水深 h，其频散关系 $\omega(k)=\sqrt{g k \tanh (k h)}$ ，如果说作用在水面上的干扰力是风力，那么重力就是恢复力
*   $\tilde{h}_0$ 是 $\tilde{h}_0^*$ 的共轭复数

到此为止，整个 IDFT 公式就明朗了，然后就是套用上面 IFFT 的步骤

### 5.3 基于 GPGPU 的 Ocean IFFT 计算

通过上面的内容，可以预见的是生成高度图会有大量的数学计算，尽管这已经是算法加速后的结果了，因此大多数情况下都会把这部分计算任务交给更擅长并行计算的 GPU 而非 CPU  
在 Unity 中，往往使用 ComputeShader 来实现

### 5.3.1 初始频谱计算

先是计算菲利浦频谱 $P_h(\vec{k})=A \frac{e^{-1 /(k L)^2}}{k^4}|\vec{k} \cdot \vec{\omega}|^2$ 以生成对应纹理，看上去最陌生的公式其实直接抄作业就行了，其中 $\vec{k}=\left(k, l\right)$ 作为纹理坐标，其余除了风向 $\vec{\omega}$ 都为常量。很明显，游戏中我们不太可能每帧去改变风向，因此风向 $\vec{\omega}$ 也可以视作常量（由美术或策划去配置），**在这种情况下** $P_h(\vec{k})$ **的计算理论可以离线或者只在风向改变时做一次，不需要每帧去重复计算**

当然你也可以选择效果更好的，也更复杂的 TMA JONSWAP 频谱与 Hasselman 方向扩展：

*   JONSWAP spectrum： $\begin{aligned} S_j(\omega) & =\frac{\alpha g^2}{\omega^5} \exp \left[-\frac{5}{4}\left(\frac{\omega_p}{\omega}\right)^4\right] \gamma^r \\ r & =\exp \left[-\frac{\left(\omega-\omega_p\right)^2}{2 \sigma^2 \omega_p^2}\right] \end{aligned}$
*   TMA correct： $S_{\text {TMA }}(\omega)=S_{\text {JONSWAP }}(\omega) \Phi(\omega, h)$
*   Swell Longuet-Higgins form： $\begin{aligned} D_{\xi}(\omega, \theta) & =Q_{\xi}\left(s_{\xi}\right)|\cos (\theta / 2)|^{2 s_{\xi}} \\ s_{\xi} & =16 \tanh \left(\frac{\omega_p}{\omega}\right) \xi^2 \end{aligned}$
*   $s= \begin{cases}6.97\left(\omega / \omega_p\right)^{5} & \omega \leq \omega_p \\ 9.77\left(\omega / \omega_p\right)^{2.5} & \omega>\omega_p\end{cases}$

不管多复杂，就一个字：抄！**这一部分也可以离线计算**

```
//----------------------------------------------- Hasselmann Directional Spreading D(w,/theta) -----------------------------------------------
float NormalisationFactor(float s) {
    float s2 = s * s;
    float s3 = s2 * s;
    float s4 = s3 * s;
    if (s < 5)
        return -0.000564 * s4 + 0.00776 * s3 - 0.044 * s2 + 0.192 * s + 0.163;
    else
        return -4.80e-08 * s4 + 1.07e-05 * s3 - 9.53e-04 * s2 + 5.90e-02 * s + 3.93e-01;
}

float Cosine2s(float theta, float s) {
    return NormalisationFactor(s) * pow(abs(cos(0.5 * theta)), 2 * s);
}

float SpreadPower(float omega, float peakOmega) {
    if (omega > peakOmega)
        return 9.77 * pow(abs(omega / peakOmega), -2.5);
    else
        return 6.97 * pow(abs(omega / peakOmega), 5);
}

float DirectionSpectrum(float theta, float omega, SpectrumParameters pars) {
    float s = SpreadPower(omega, pars.peakOmega)
            + 16 * tanh(min(omega / pars.peakOmega, 20)) * pars.swell * pars.swell;
    return lerp(2 / 3.1415 * cos(theta) * cos(theta), Cosine2s(theta - pars.angle, s), pars.spreadBlend);
}
//----------------------------------------------- Hasselmann Directional Spreading End -----------------------------------------------

//----------------------------------------------- TMA JONSWAP spectrum -----------------------------------------------
float TMACorrection(float omega, float g, float depth) {
    float omegaH = omega * sqrt(depth / g);
    if (omegaH <= 1)
        return 0.5 * omegaH * omegaH;
    if (omegaH < 2)
        return 1.0 - 0.5 * (2.0 - omegaH) * (2.0 - omegaH);
    return 1;
}

float JONSWAP(float omega, float g, float depth, SpectrumParameters pars) {
    //pars.peakOmega = wp，pars.gamma = 3.3，pars.alpha = a
    float sigma;
    if (omega <= pars.peakOmega)
        sigma = 0.07;
    else
        sigma = 0.09;
    float r = exp(-(omega - pars.peakOmega) * (omega - pars.peakOmega)
            / 2 / sigma / sigma / pars.peakOmega / pars.peakOmega);

    float oneOverOmega = 1 / omega;
    float peakOmegaOverOmega = pars.peakOmega / omega;
    return pars.scale * TMACorrection(omega, g, depth) * pars.alpha * g * g
            * oneOverOmega * oneOverOmega * oneOverOmega * oneOverOmega * oneOverOmega
            * exp(-1.25 * peakOmegaOverOmega * peakOmegaOverOmega * peakOmegaOverOmega * peakOmegaOverOmega)
            * pow(abs(pars.gamma), r);
}
//----------------------------------------------- TMA JONSWAP spectrum End -----------------------------------------------
```

部分常量与参数，例如风速，风向、以及 TMA spectrum 中的增强因子、方向函数中的涌浪比重等等，都可以做成可配置的，用于给美术调节最终的效果（一般而言，只需要调整标红的参数）：

![[3b57fd2e53bfc6561a8bd3de13f37edc_MD5.jpg]]

1.  G：重力加速度，默认 9.8 即可
2.  Depth：水深，用于与风力相对应的恢复力计算，也参与 TMA 矫正，一般而言水深不用改
3.  Lambda：海洋水平方向偏移参数，如果 Lambda = 0，则海洋最终顶点动画只有高度变化，没有水平偏移，也就是不会形成挤压（此参数和初始海洋频谱计算无关，具体内容可参考下面 5.3.2 小节）
4.  Scale：控制最终的海浪表现：如果 Scale = 0，则没有海浪
5.  WindSpeed：风速，决定 TMA JOPNSWAP 频谱的 $\alpha$ 项，及 $\omega_p$ 项，理论风速越快，海浪越大
6.  WindDirection：风向，角度表示法
7.  Fetch：离背风岸的距离倍数，即风场，可以理解为风以恒定速度吹过的距离
8.  SpreadBlend：控制方向扩展函数对最终海浪效果的贡献，如果 SpreadBlend = 0，则方向扩展函数使用最基础的经验模型： $D_{\cos ^2}(\theta)= \begin{cases}\frac{2}{\pi} \cos ^2(\theta) & \text { if } \frac{-\pi}{2}<\theta<\frac{\pi}{2} \\ 0 & \text { otherwise }\end{cases}$ ，如果 SpreadBlend = 1，则方向扩展函数完全使用 Swell Longuet-Higgins form： $\begin{aligned} D_{\xi}(\omega, \theta) & =Q_{\xi}\left(s_{\xi}\right)|\cos (\theta / 2)|^{2 s_{\xi}} \end{aligned}$ ，否则最终结果在两者之间插值
9.  Swell：用于控制涌浪强度，关于涌浪的解释可以看 5.2.2 节中关于 Hasselmann Directional Spreading 模型的介绍
10.  Peak Enhancement：TMA JOPNSWAP 频谱的 $\gamma$ 项
11.  Short Waves Fade：短波过滤，效果上只保留了波长超过一定值的海浪

![[c49f0b1c51a9e0b2e5c479db9e9c69c6_MD5.jpg]]

然后就是拿计算得到的非定向波普和方向扩展函数，去最终计算

*   $\tilde{h}_0(\vec{k})=\frac{1}{\sqrt{2}}\left(\xi_1+i \xi_2\right) \sqrt{2 S(\omega) D(\theta, \omega) \frac{d \omega(k)}{d k} \frac{1}{k} \Delta k_x \Delta k_z}$
*   $\omega(k)=\sqrt{g k \tanh (k h)}$

高斯随机数 $\left(\xi_r+i \xi_i\right)$ 生成，只是为了得到不同的初始状态，因此同样可以离线计算：

[GPU Gems3:Chapter 37](https://developer.nvidia.com/gpugems/GPUGems3/gpugems3_ch37.html) 中提到过如何生成满足服从均值为 0，标准差为 1 的高斯随机数，公式如下：  
$\xi_r =\sin \left(2 \pi u\right) \sqrt{-2 \log \left(v\right)}$  
$\xi_i =\cos \left(2 \pi u\right) \sqrt{-2 \log \left(v\right)}$  
其中 $u,v$ 为均匀分布的随机数

$\vec{k}$ 则对应所有可能的复平面波 $e^{-i 2 \pi(n x+m z)}$ ，规定常量 $L$ 为海平面大小，采样点 $k, l$ 满足 $0<k, l<N-1$ 有 $\vec{k}=\left(\frac{2 \pi k-\pi N}{L}, \frac{2 \pi l-\pi N}{L}\right)$

```
[numthreads(8, 8, 1)]
void CalculateInitialSpectrum(uint3 id : SV_DispatchThreadID)
{
    float deltaK = 2 * PI / 500;
    int nx = id.x - Size / 2;
    int nz = id.y - Size / 2;
    //波矢量 k
    float2 k = float2(nx, nz) * deltaK;
    float kLength = length(k);

    if (kLength <= 10000 && kLength >= 0.001)
    {
        float kAngle = atan2(k.y, k.x);
        float omega = Frequency(kLength, GravityAcceleration, Depth);
        WavesData[id.xy] = float4(k.x, 1 / kLength, k.y, omega);

        //求 dw(k)，即频散关系函数的导数
        float dOmegadk = FrequencyDerivative(kLength, GravityAcceleration, Depth);
        //求 S(w) * D(w,/theta)
        float spectrum = JONSWAP(omega, GravityAcceleration, Depth, Spectrums[0])
                * DirectionSpectrum(kAngle, omega, Spectrums[0]) * ShortWavesFade(kLength, Spectrums[0]);
        if (Spectrums[1].scale > 0)
                spectrum += JONSWAP(omega, GravityAcceleration, Depth, Spectrums[1])
                * DirectionSpectrum(kAngle, omega, Spectrums[1]) * ShortWavesFade(kLength, Spectrums[1]);
        //计算最终的 H0
        H0K[id.xy] = float2(Noise[id.xy].x, Noise[id.xy].y)
                * sqrt(2 * spectrum * abs(dOmegadk) / kLength * deltaK * deltaK);
    }
    else
    {
        H0K[id.xy] = 0;
        WavesData[id.xy] = float4(k.x, 1, k.y, 0);
    }
}

[numthreads(8, 8, 1)]
void CalculateConjugatedSpectrum(uint3 id : SV_DispatchThreadID)
{
    float2 h0K = H0K[id.xy];
    float2 h0MinusK = H0K[uint2((Size - id.x) % Size, (Size - id.y) % Size)];
    H0[id.xy] = float4(h0K.x, h0K.y, h0MinusK.x, -h0MinusK.y);
}
```

这些计算量并不小，但是好在它只需要算一次，除非你的参数例如风速风向有所调整，因此可以离线先烘好保存，运行时只需要加载对应的 asset 资源就可以

![[605ab52eb6d44ef3a58a3ac2dd764be3_MD5.jpg]]

实时计算当然也没问题，特别是编辑器情况下，调整参数可以直接看到海浪的效果变化，就不能离线了，这一部分的计算由于没有像素之间相互依赖的关系，因此可以交给 GPU 并行去算：最终可以得到 2 张 RenderTexture：

*   initialSpectrum(ARGBFloat)： $({h}_0(\vec{k}), \ {h}_0^*(-\vec{k}))$
*   precomputedData(ARGBFloat)： $(k_x, \frac{1}{|k|}, k_z, w(k))$ ，用于后续计算水平偏移

你可能计算得到的 initialSpectrum Texture 看上去是全黑的，这可能由于生成的频率幅值过小，不一定出错，只是表现上看不出来

```
public void CalculateInitials(WavesSettings wavesSettings, float lengthScale) {
    //设置计算频谱的各项参数，例如风速、风向、重力加速度常量等
    wavesSettings.SetParametersToShader(initialSpectrumShader, KERNEL_INITIAL_SPECTRUM, paramsBuffer);

    //将计算的 H0k 保存到一张纹理中
    initialSpectrumShader.SetTexture(KERNEL_INITIAL_SPECTRUM, H0K_PROP, buffer);
    //计算角频率与波长的频散关系 w 保存到另一张纹理中
    initialSpectrumShader.SetTexture(KERNEL_INITIAL_SPECTRUM, PRECOMPUTED_DATA_PROP, precomputedData);
    //传入高斯随机数纹理，用于计算 H0K
    initialSpectrumShader.SetTexture(KERNEL_INITIAL_SPECTRUM, NOISE_PROP, gaussianNoise);
    //交给 GPGPU 开始计算
    initialSpectrumShader.Dispatch(KERNEL_INITIAL_SPECTRUM, size / LOCAL_WORK_GROUPS_X, size / LOCAL_WORK_GROUPS_Y, 1);

    //计算得到 H0k 后，把共轭结果 H*0K 也存下来
    initialSpectrumShader.SetTexture(KERNEL_CONJUGATE_SPECTRUM, H0_PROP, initialSpectrum);
    initialSpectrumShader.SetTexture(KERNEL_CONJUGATE_SPECTRUM, H0K_PROP, buffer);
    initialSpectrumShader.Dispatch(KERNEL_CONJUGATE_SPECTRUM, size / LOCAL_WORK_GROUPS_X, size / LOCAL_WORK_GROUPS_Y, 1);
}
```

### 5.3.2 任意时刻频谱计算

关于 $\tilde{h}(\vec{k}, t)=\tilde{h}_0(\vec{k}) e^{i \omega(k) t}+\tilde{h}_0^*(-\vec{k}) e^{-i \omega(k) t}$ 的代码实现更简单，前面已经得到了 $\tilde{h}_0(\vec{k})$ 以及 $\omega(k)$ ，组合一下就好：

![[33c99dccb923d3d3d02baaac77f43e9d_MD5.jpg]]

```
float4 wave = WavesData[id.xy];
float phase = wave.w * Time;
float2 exponent = float2(cos(phase), sin(phase));
float2 h = ComplexMult(H0[id.xy].xy, exponent)
    + ComplexMult(H0[id.xy].zw, float2(exponent.x, -exponent.y));
float2 ih = float2(-h.y, h.x);
```

不过考虑到海面并不只有高度变化，还有水平方向的流动，后续只有一张 y 轴的高度图还是不够的，还需要 x 轴和 z 轴的偏移图，这里可以先根据 $\tilde{h}(\vec{k}, t)$ 获取水平方向的频谱

*   $\tilde{d_x}(\vec{k}, t)=i \frac{k_x}{|k|} \tilde{h}(\vec{k}, t)$
*   $\tilde{d_z}(\vec{k}, t)=i \frac{k_z}{|k|} \tilde{h}(\vec{k}, t)$

Gerstner 波为了展现波峰尖角，也同样对 xz 平面做了变换

前面代码将 $(k_x, \frac{1}{|k|}, k_z, w(k))$ 一起塞到了一张 4 通道纹理中，这里继续拿来用：

```
float2 displacementX = ih * wave.x * wave.y;
float2 displacementY = h;
float2 displacementZ = ih * wave.z * wave.y;
```

### 5.3.3 海洋法线

其实到这这一步应该就可以准备 IFFT 计算了，不过中间还有一个东西没有考虑，那就是法线图的生成，因为你要计算光照，法线图是必须的，实时的海浪必然意味着法线图也需要实时计算

这里关于法线计算有两种方案

1.  得到高度图后，可以通过差分方法来求法线，即对变换后的新顶点 $M_0(x,y)$ ，求出与其相邻的两组顶点 $M_1(x+1,y)$ $M_1'(x-1,y)$ 和 $M_2(x,y-1)$ $M_2'(x,y+1)$，交叉连线后得到两个向量，叉乘得出法线，这个方案逻辑非常简单，可以直接在着色器中实时计算得到法线，缺点也很明显，那就是仅能得到近似法线，这个结果是不会准确的
2.  通过偏导计算求出曲面梯度，然后再根据梯度得到法线

梯度的本意是一个向量，它的方向与取得最大[方向导数](https://www.zhihu.com/question/36301367/answer/142096153)的方向一致，而它的模为 方向导数的最大值，也可以理解为函数在该点处沿着该方向变化最快，变化率最大  
设函数 $z=f(x, y)$ 在平面区域 D 内具有一阶连续偏导数，则对于每一点 $(x, y)\in D$ ，都可定出一个向量 $\frac{\partial f}{\partial x} i+\frac{\partial f}{\partial y} j$ ，该向量称为函数 $z=f(x, y)$ 在点 $P(x, y)$ 的梯度，记作 $\nabla f(x, y)$  
可以证明： $\nabla f(x, y)=\left\{\frac{\partial f}{\partial x}, \frac{\partial f}{\partial y}\right\}=f_x(x, y) \bar{i}+f_y(x, y) \bar{j}$

对于高度图 $h(\vec{x}, t)=\sum_k \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$ ，其梯度满足  
$\nabla h(\vec{x}, t)=\sum_{{k}} \tilde{h}(\vec{k}, t) \nabla e^{i \vec{k} \cdot \vec{x}} = \sum_{{k}} i\vec{k}\tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$ ，其中偏导

*   $\frac{\partial h(\vec{x}, t)}{\partial x}=\sum_{{k}} ik_x\tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$
*   $\frac{\partial h(\vec{x}, t)}{\partial z}=\sum_{{k}} ik_z\tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$

```
float2 displacementY_dx = ih * wave.x;
float2 displacementY_dz = ih * wave.z;
```

不过还不够，我们还有两张水平方向的偏移图：

*   $D_x(\vec{x}, t)=\sum_ki \frac{k_x}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$
*   $D_z(\vec{x}, t)=\sum_ki \frac{k_z}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$

此时不止高度会发生变化，延 x 方向和 z 方向也有一段偏移，这段偏移是  
$D_x'(\vec{x}, t)=x+ \vec{D_x}(\vec{x}, t)$ 和 $D_z'(\vec{x}, t)=z+ \vec{D_z}(\vec{z}, t)$ ，对应的偏导

*   $\frac{\partial D_x'(\vec{x}, t)}{\partial x}=1+ \sum_k- \frac{k_x^2}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$
*   $\frac{\partial D_z'(\vec{x}, t)}{\partial z}=1+ \sum_k- \frac{k_z^2}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}$

```
float2 displacementX_dx = -h * wave.x * wave.x * wave.y;
float2 displacementZ_dz = -h * wave.z * wave.z * wave.y;
```

因此，考虑了水平偏移后，真正的海浪梯度 $\nabla$ 就为：

$(\frac{\sum_{{k}} ik_x\tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}}{1+ \sum_k- \frac{k_x^2}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}} ,\frac{\sum_{{k}} ik_z\tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}} }{1+ \sum_k- \frac{k_z^2}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}})$

看上去这个公式很复杂，其实本质上就是 (高度图对 x 偏导 / 扭曲挤压后的新海面对 x 偏导, 高度图对 z 偏导 / 扭曲挤压后的新海面对 z 偏导)

高度图很好理解，但是平面延 x 和 z 方向的偏移过程可能不是特别好理解：这里可以沿用一张 wiki 上的图片来做参考：

![[62c86e5cebed129d65b58a9c25ceac26_MD5.jpg]]

在应用水平偏移后，整个海平面就不再是一个线性的平面了，其坐标会被扭曲，甚至会出现挤压（有向面元在变换后面积为负数），这样再看上面那个极其复杂的海浪梯度向量，其实分子部分就是

$\nabla h(\vec{x}, t)=(\frac{\partial h(\vec{x}, t)}{\partial x},\frac{\partial h(\vec{x}, t)}{\partial z})$

分母部分即是对上图右部分的新曲面求偏导，即 $(\frac{\partial D_x'(\vec{x}, t)}{\partial x},\frac{\partial D_z'(\vec{x}, t)}{\partial z})$

也可以以实际应用为例子，直接看最终海洋的顶点变换表现，当你没有应用水平偏移时最终顶点动画的效果如左：

![[a2ddd4f9a427cdc807badf34d439c192_MD5.gif]]

可以看出顶点网格水平方向上其实是静止的，只有高度在发生变化，而如果应用了水平方向的偏移，最终顶点动画效果如右

差别明显，如果我们把视角调成垂直往下，就可以直接观察到变换后海面的顶点水平分布情况：

![[f071645a0b8be9720d7f1a1ac8cd92cd_MD5.jpg]]

搞定梯度后，想要计算法线就很简单了，只需要拿上向量 $(0,1,0)$ 减去梯度向量，得到的就是法向量，这样下来对于空间中海面上的一点

$P(x+ \vec{D_x}(\vec{x}, t), h(\vec{x}, t), z+ \vec{D_z}(\vec{z}, t))$ ，即

$P(x+\sum_ki \frac{k_x}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}, \sum_k \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}, z+\sum_ki \frac{k_z}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}})$

其精确法线（未考虑归一化）就为

$(0,1,0)-(\frac{\frac{\partial h(\vec{x}, t)}{\partial x}}{1 + \frac{\partial D_x(\vec{x}, t)}{\partial x}}, 0, \frac{\frac{\partial h(\vec{x}, t)}{\partial z}}{1 + \frac{\partial D_z(\vec{x}, t)}{\partial z}}) \\ = P_{N}(-\frac{\sum_{{k}} ik_x\tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}}{1+ \sum_k- \frac{k_x^2}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}}, 1, -\frac{\sum_{{k}} ik_z\tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}} }{1+ \sum_k- \frac{k_z^2}{k} \tilde{h}(\vec{k}, t) e^{i \vec{k} \cdot \vec{x}}})$

搞定，这些内容同样交给 GPU 去计算：

```
//计算 h(k,t) 即海浪任意时刻的频谱
public void CalculateWavesAtTime(float time, RenderTexture initialSpectrum, RenderTexture precomputedData) {
    //x 方向及 z 方向水平偏移频谱
    timeDependentSpectrumShader.SetTexture(KERNEL_TIME_DEPENDENT_SPECTRUMS, JSeaLib.SC.Dx_Dz_PROP, DxDz);
    //y 方向海洋高度频谱，另一个频谱用于计算雅可比方程，以得到海浪泡沫效果
    timeDependentSpectrumShader.SetTexture(KERNEL_TIME_DEPENDENT_SPECTRUMS, JSeaLib.SC.Dy_Dxz_PROP, DyDxz);
    //x 和 z 对海洋高度谱 y 求偏导，这一项用于计算海洋法线
    timeDependentSpectrumShader.SetTexture(KERNEL_TIME_DEPENDENT_SPECTRUMS, JSeaLib.SC.Dyx_Dyz_PROP, DyxDyz);
    //x 对水平偏移频谱、z 对其对于水平偏移频谱求偏导，这一项用于计算海洋法线
    timeDependentSpectrumShader.SetTexture(KERNEL_TIME_DEPENDENT_SPECTRUMS, JSeaLib.SC.Dxx_Dzz_PROP, DxxDzz);

    //传入之前（离线）计算好的 H0k 和共轭结果 H*0K
    timeDependentSpectrumShader.SetTexture(KERNEL_TIME_DEPENDENT_SPECTRUMS, JSeaLib.SC.H0_PROP, initialSpectrum);
    //传入之前（离线）计算好的 (ARGB: Kx, 1/length(k), Kz, w(k))
    timeDependentSpectrumShader.SetTexture(KERNEL_TIME_DEPENDENT_SPECTRUMS, JSeaLib.SC.PRECOMPUTED_DATA_PROP, precomputedData);

    //传入时间 t
    timeDependentSpectrumShader.SetFloat(JSeaLib.SC.TIME_PROP, time);
    //开始计算
    timeDependentSpectrumShader.Dispatch(KERNEL_TIME_DEPENDENT_SPECTRUMS, size / JSeaLib.LOCAL_WORK_GROUPS_X, size / JSeaLib.LOCAL_WORK_GROUPS_Y, 1);
}
```

### 5.3.4 IFFT 蝶形图生成

[Fast Computation of General Fourier Transforms on GPUs](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-2008-62.pdf)

第三章已经对 FFT 蝶形算法做过了详细的解析，一个 N = 8 的蝶形算法如下图：

![[826abc5ae9fe0f8b099a342adfe26bbe_MD5.jpg]]

对于 IDFT $x[n]=\frac{1}{N} \sum_{k=0}^{N-1} X[k] e^{i \frac{2 \pi}{N} kn}$ ， $n(0{\leq}n{<}N)$ ，令 $W_N=e^{-i\frac{2{\pi}}{N}}$ ，其满足性质：

*   $X(n)=G(n)+W_N^{-n} H(n) , \ 0{\leq}n{<}\frac{N}{2}$
*   $X(n)=G(n-\frac{N}{2})-W_N^{-(n-\frac{N}{2})} H(n-\frac{N}{2})$ ， $\frac{N}{2}{\leq}n{<}N$

其中 $G(n)=\sum_{r=0}^{\frac{N}{ 2}-1} x(2 r) W_{\frac{N}{ 2}}^{-r n}$ ， $H(n)=\sum_{r=0}^{\frac{N}{ 2}-1} x(2 r+1) W_{\frac{N}{ 2}}^{-r n}$

```
public void IFFT2D(RenderTexture input, RenderTexture buffer) {
    int logSize = (int)Mathf.Log(size, 2);
    bool pingPong = false;
    fftShader.SetTexture(KERNEL_HORIZONTAL_STEP_IFFT, PROP_ID_PRECOMPUTED_DATA, precomputedData);
    fftShader.SetTexture(KERNEL_HORIZONTAL_STEP_IFFT, PROP_ID_BUFFER0, input);
    fftShader.SetTexture(KERNEL_HORIZONTAL_STEP_IFFT, PROP_ID_BUFFER1, buffer);
    for (int i = 0; i < logSize; i++)
    {
        pingPong = !pingPong;
        fftShader.SetInt(PROP_ID_STEP, i);
        fftShader.SetBool(PROP_ID_PINGPONG, pingPong);
        fftShader.Dispatch(KERNEL_HORIZONTAL_STEP_IFFT, size / LOCAL_WORK_GROUPS_X, size / LOCAL_WORK_GROUPS_Y, 1);
    }
    fftShader.SetTexture(KERNEL_VERTICAL_STEP_IFFT, PROP_ID_PRECOMPUTED_DATA, precomputedData);
    fftShader.SetTexture(KERNEL_VERTICAL_STEP_IFFT, PROP_ID_BUFFER0, input);
    fftShader.SetTexture(KERNEL_VERTICAL_STEP_IFFT, PROP_ID_BUFFER1, buffer);
    for (int i = 0; i < logSize; i++)
    {
        pingPong = !pingPong;
        fftShader.SetInt(PROP_ID_STEP, i);
        fftShader.SetBool(PROP_ID_PINGPONG, pingPong);
        fftShader.Dispatch(KERNEL_VERTICAL_STEP_IFFT, size / LOCAL_WORK_GROUPS_X, size / LOCAL_WORK_GROUPS_Y, 1);
    }
    if (pingPong)
    {
        Graphics.Blit(buffer, input);
    }
}
```

如果递归计算上面按所有的 X, G, H，总体复杂度为 O(NlogN)，例如 N = 8 时，从上图可以看出其递归深度为 3，每次计算量都为 8，即总共 3x8=24 个蝶形计算单元：

![[32e953f53e2085a72701bbaee64233fb_MD5.jpg]]

一次蝶形计算单元如上，由于我们可能需要进行多次 IFFT，但事实上由于每次 IFFT 的大小（采样次数）N 是固定的，因此可以**预计算所有的** $W_N^k$ **，并将其复数结果放入一张 logN x N 大小的纹理的 R 和 G 通道中（纹理坐标 x, y 取值从 0 开始），令** $d = \frac{N}{2^{x+1}}$ **，则对于纹理** $(x,y)$ **位置的值** $W_N^k$ **满足：**

$k=\lfloor\frac{y }{d} \rfloor\cdot d$

**还有两个通道 B 和 A，用于放入图中的上一层 IFFT 的索引 a 和 b**，a 和 b 的计算公式如下：

*   $a=(2k+y\bmod d)\bmod{N}$
*   $b=a+d$

对于 N = 8 的情况下，a, b, k 和 d 的取值即如下，纵轴对应 y = 0~8，横轴对应 x = 0~2

*   x = 0：a = 0 1 2 3 0 1 2 3 b = 4 5 6 7 4 5 6 7 k = 0 0 0 0 4 4 4 4 d = 4
*   x = 1：a = 0 1 4 5 0 1 4 5 b = 2 3 6 7 2 3 6 7 k = 0 0 2 2 4 4 8 8 d = 2
*   x = 2：a = 0 2 4 6 0 2 4 6 b = 1 3 5 7 1 3 5 7 k = 0 1 2 3 4 5 6 7 d = 1

对于实际应用的例子中 N = 256 的情况，生成的蝶形纹理如下：

![[b7b4727b585a35d35cb81c11a0028ce6_MD5.jpg]]

```
void PrecomputeTwiddleFactorsAndInputIndices(uint3 id : SV_DispatchThreadID)
{
    uint b = Size >> (id.x + 1);
    float2 mult = 2 * PI * float2(0, 1) / Size;
    uint i = (2 * b * (id.y / b) + id.y % b) % Size;
    float2 twiddle = ComplexExp(-mult * ((id.y / b) * b));
    PrecomputeBuffer[id.xy] = float4(twiddle.x, twiddle.y, i, i + b);
    PrecomputeBuffer[uint2(id.x, id.y + Size / 2)] = float4(-twiddle.x, -twiddle.y, i, i + b);
}
```

这个无论是 k 的计算，还是最后索引 a 的计算，可能单看数字不是特别明白，但是没有关系，只要代入到后面的计算场景就会立刻清晰：

如果采样递归的手段计算 FFT，那么逻辑写起来会相对简单一点，因为它是顺着思路的，之前就有举过一道多项式乘法的例子，并且给出了代码，然而由于我们想要 GPU 帮我们并行计算，就不好递归去做，这个时候只能逆过来算，也就是按照蝶形图从左至右（stage 1~3 的顺序）的计算每一个单元：这个时候再看我们关于索引 a, b 以及 k 的计算就好理解多了

![[e653b3e95445034f72f7c41195909849_MD5.jpg]]

当然每一层计算完毕后，索引都会按照该层的计算顺序更新排列

不仅如此，由于我们计算每一个蝶形单元时，只依赖上一个层 (stage) 的结果，因此每一层的运算都可以并行处理，效率极高：

```
void HorizontalStepInverseFFT(uint3 id : SV_DispatchThreadID)
{
    float4 data = PrecomputedData[uint2(Step, id.x)];
    uint2 inputsIndices = (uint2)data.ba;
    if (PingPong)
    {
        Buffer1[id.xy] = Buffer0[uint2(inputsIndices.x, id.y)]
            + ComplexMult(float2(data.r, -data.g), Buffer0[uint2(inputsIndices.y, id.y)]);
    }
    else
    {
        Buffer0[id.xy] = Buffer1[uint2(inputsIndices.x, id.y)]
            + ComplexMult(float2(data.r, -data.g), Buffer1[uint2(inputsIndices.y, id.y)]);
    }
}
```

对所有行来一次横向 FFT 后，就是所有列的纵向 FFT，对应的计算方式几乎完全一致，只不过别忘了最后得到的结果还要即乘上系数 $\frac{(-1)^x(-1)^y}{N^2}$ ：分子部分符号位的纠正理论上是必要的，但是分母 N² 可以不用去除，因为你可以把其当成一个海浪整体幅度的放缩倍数

```
[numthreads(8, 8, 1)]
void Scale(uint3 id : SV_DispatchThreadID)
{
    Buffer0[id.xy] = Buffer0[id.xy] / Size / Size;
}

[numthreads(8, 8, 1)]
void Permute(uint3 id : SV_DispatchThreadID)
{
    Buffer0[id.xy] = Buffer0[id.xy] * (1.0 - 2.0 * ((id.x + id.y) % 2));
}
```

搞定！

### 5.3.5 海洋高度场、梯度与水平偏移量

根据 5.3.2 及 5.3.3 的内容，可知需要对以下频谱求得对应的时域图像：

*   高度场及水平偏移量： $h(\vec{x}, t), \tilde{d_x}(\vec{k}, t), \tilde{d_z}(\vec{k}, t)$
*   用于求梯度法线的偏导： $\frac{\partial h(\vec{x}, t)}{\partial x}, \frac{\partial h(\vec{x}, t)}{\partial z}, \frac{\partial D_x'(\vec{x}, t)}{\partial x}, \frac{\partial D_z'(\vec{x}, t)}{\partial z}$
*   用于求海洋泡沫的偏导： $\frac{\partial D_z'(\vec{x}, t)}{\partial x}$

看上去需要进行 8 次 FFT，最终得到 8 张单通道高精度的纹理，用于存储时域结果

但事实上，只需要进行 4 次的 FFT：

```
Dx_Dz[id.xy] = float2(displacementX.x - displacementZ.y, displacementX.y + displacementZ.x);                //X + iZ
Dy_Dxz[id.xy] = float2(displacementY.x - displacementZ_dx.y, displacementY.y + displacementZ_dx.x);                //Y + idxZ
Dyx_Dyz[id.xy] = float2(displacementY_dx.x - displacementY_dz.y, displacementY_dx.y + displacementY_dz.x);                //dxY + idzY
Dxx_Dzz[id.xy] = float2(displacementX_dx.x - displacementZ_dz.y, displacementX_dx.y + displacementZ_dz.x);                //dxX + idzZ
```

考虑到整个 FT 的输入输出，我们的输入内容（频域图像）是一个复数：实部为频率，虚部为相位，但是输出的时域结果只有实部有意义，例如生成的海洋高度场，其实部结果就对应的海浪高度

利用这个特点，我们就可以将两个不同的频域函数成对打包：即其中一个不变，另一个频域函数实部和虚部颠倒，从而做到一次 IFFT 计算得到两个时域结果

![[d6e3a50de38f88fca18c1b7d2610ece9_MD5.jpg]]

到此整个 FFT Ocean 的物理计算流程就要到尾声了，经过 IFFT 计算后，我们就能通过后续纹理合并得到两张 RGBAFloat 四通道纹理，分别对应

1.  一张海浪高度 + 偏移图：如果不做海洋泡沫，只需要前 3 个通道
2.  一张海浪梯度图：4 通道偏导，用于计算梯度，以及法线

![[43a1f2f1adeff9f9ea7624280edf7d49_MD5.jpg]]

其中第一张纹理，运用到海洋着色器的顶点变换：

```
float3 displacement = 0;
displacement += tex2Dlod(_Displacement_c0, worldUV / LengthScale0) * lod_c0;
v.vertex.xyz += mul(unity_WorldToObject, displacement);
```

同着色器中，法线图的计算则如下

```
float4 derivatives = tex2D(_Derivatives_c0, IN.worldUV / LengthScale0);
float2 slope = float2(derivatives.x / (1 + derivatives.z),
    derivatives.y / (1 + derivatives.w));
float3 worldNormal = normalize(float3(-slope.x, 1, -slope.y));
```

搞定，效果放在了文章最前面，代码的算法部分实际上是直接借鉴的 [Ocean waves simulation with Fast Fourier transform](https://www.youtube.com/watch?v=kGEqaX4Y4bQ)，当然为了能让高配手机上也能运行，以及要做非真实感的海洋，还是要做不少工作的，例如着色，水体交互、海浪泡沫、性能优化、离线 FT 等等。考虑到本篇的原意只是介绍 FT 及其在游戏开发领域的应用，就先在这画上句号吧

剩下相关的两节，也将在后面更新

*   基于 LOD 的无限大海洋的频谱填充
*   移动平台 FFT 方案：离线 FFT + 性能优化