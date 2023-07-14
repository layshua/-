蒙特 · 卡罗（Monte Carlo）法是一种统计模拟方法，通常是利用随机数来解决一些数值计算问题，本文要讲的就是利用蒙特 · 卡罗方法来求解数值积分。

## 基本思路

首先我们知道定积分其实就是一个面积，将其设为 $I$ ，现在我们就是要求出这个 $I$ 。我们的想法是通过在包含定积分的面积为 $S$ 的区域（通常为矩形）内随机产生一些随机数，其数量为 $N$ ，再统计在积分区域内的随机数，其数量为 $i$ ，则产生的随机数在积分区域内的概率为 $\frac{i}{N}$ ，这与积分区域与总区域面积的比值 $\frac{I}{S}$ 应该是近似相等的，我们利用的就是这个关系，即

$$\frac{I}{S}\approx \frac{i}{N} $$

最后即得所求定积分算式为：

$$I=\frac{i}{N}\circledS$$

## 代码部分

有了上面的铺垫，我们就可以来写 MATLAB 代码了。

我们要求的定积分为

$$\int_0^\pi \sin x \mathrm{d}x. \\$$

对于上述积分我们很容易可以得到其解析解为 $2$ ，下面我们来看用蒙特 · 卡罗方法得到的结果，输入代码

```
% Monte Carlo
% 蒙特卡洛法求定积分
clear
N = 1e4;
x_min = 0; x_max = pi;
f = @(x) sin(x);
xx = x_min:0.01:x_max;
x = x_min + (x_max-x_min)*rand(N,1);
y_min = min(f(xx)); y_max = max(f(xx));
y = y_min + (y_max-y_min)*rand(N,1);
i = y < f(x);
I = sum(i)/N*(x_max-x_min)*(y_max-y_min);

% 画图
plot(x,y,'go',x(i),y(i),'bo')
axis([x_min x_max y_min y_max])
hold on
plot(xx,f(xx),'r-','LineWidth',2)
```

执行上述代码即得

![](https://pic1.zhimg.com/v2-7858b683aaac94c61cf9db4e5a92ab30_r.jpg)

![](https://pic3.zhimg.com/v2-3b1d8905fb47970620b5144964be134e_r.jpg)

从上述结果我们可以看到，所得积分结果为 $I=2.0075$ ，可见其与真值 $2$ 还是十分接近的。如果我们将 $N$ 取得更大一些，所得结果可以更加逼近真值，比如我们取 $N=10^6$ 将得到

```
I =

   2.000408487907250
```

此时的图为

![](https://pic3.zhimg.com/v2-abadfed63f8fdf6ed1cf3b1c8c1cdd1a_r.jpg)

## 总结

其实我们很容易发现上述蒙特 · 卡罗方法的弊端，即他只能求在求积区间函数值恒非负或恒非正的函数的定积分，如果在积分区间上函数有正有负，那么我们只能将区间拆分为多个子区间分别计算子积分再相加得到。传统的数值积分计算方法可以查看我的另一篇文章：

[清清清平：数值积分常用方法](https://zhuanlan.zhihu.com/p/166272592)

## 不足

对于有些函数用蒙特 · 卡罗方法得到的结果与真值总存在很大的误差，比如求此积分

$$\int_{0.2}^1 \frac 1{x^2}\mathrm{d}x\\$$

我们容易知道这个定积分的值为 $4$ 。但是采用蒙特 · 卡罗方法将得到

```
% Monte Carlo
% 蒙特卡洛法求定积分
clear
N = 1e5;
x_min = 0.2; x_max = 1;
f = @(x) 1./x.^2;
xx = x_min:0.01:x_max;
x = x_min + (x_max-x_min)*rand(N,1);
y_min = min(f(xx)); y_max = max(f(xx));
y = y_min + (y_max-y_min)*rand(N,1);
i = y < f(x);
I = sum(i)/N*(x_max-x_min)*(y_max-y_min)

% 画图
plot(x,y,'go',x(i),y(i),'bo')
axis([x_min x_max y_min y_max])
hold on
plot(xx,f(xx),'r-','LineWidth',2)
```

执行结果为

```
I =

   3.216960000000000
```

![](https://pic3.zhimg.com/v2-1d07d221b5f61f75a4297410d34959ae_r.jpg)

此时的随机数数量已经取为 $N=10^5$ 了，所得结果与真值仍然存在很大的差距，检查代码并结合上述图形发现计算过程应该没有问题，我也不知道问题出在哪里，希望广大知友能过指出错误之处，谢谢！