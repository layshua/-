## **声明**：

本文章的学习内容来源全部出自《UnityShader 入门精要》——冯乐乐

该文章只是本人我的学习笔记，里面对《UnityShader 入门精要》进行了些许概括且加了自己的些许理解

如果想更加具体地了解其内容，建议购买原著进行学习

## 概述：

1.  线性变换 (旋转、缩放、镜像、错切......)，可以用 3x3 矩阵表示, 平移变换是非线性变换，需要 4x4 的齐次坐标变换矩阵
2.  点的变换矩阵为 4x4，向量的变换矩阵为 3x3
3.  顶点的变换过程 “模型空间” $\to$ "世界空间" $\to$ “观察空间（右手系）” $\to$ “齐次裁剪空间” $\to$ (NDC 空间） $\to$ “屏幕空间 "
4.  法线的变换矩阵是顶点变换矩阵的逆转置矩阵，如果变换矩阵为正交矩阵，则逆转置矩阵等于该矩阵
5.  Unity 的矩阵构造是行优先

## **4.5 矩阵的几何意义：变换**

在三维渲染中，矩阵用来表示变换

游戏世界中的变换一般包含（旋转、缩放、平移）

### **4.5.1 什么是变换**

**变换 (transform),** 指的是我们把一些数据， 如点、 方向矢量甚至是颜色等， 通过某种方式进行转换的过程。

**几种变换类型：**

### **1. 线性变换 (Linear Transforms)**

线性变换 (Linear Transforms) 指的是那些可以保留**矢量相加**和**标量相乘**的变换

用数学公式表示这 2 个条件就是：

$\mathbf{f(x)}+\mathbf{f(y)=\mathbf{f(x+y)}}$

$k\mathbf{f(x)}=\mathbf{f}(k\mathbf{(x)})$

上面的式子比较难以理解，下面给出更加通俗易懂的解释：

在**二维平面**中满足下面变换的就是线性变换：(用一个 2x2 矩阵就可表示的变换)

$x'=m_{11}x+m_{12}y\\y'=m_{21}x+m_{22}y$ 其中 $x',y'$ 表示新的值， $x,y$ 表示旧的值

用矩阵表示： $\left[ \begin{array}{} {x’} \\ {y'} \end{array} \right]=\left[ \begin{array}{} {m_{11}} & {m_{12}}\\ {m_{21}} & {m_{22}}\ \end{array} \right]\left[ \begin{array}{} {x} \\ {y}\ \end{array} \right]$

即**二维向量** $\mathbf{v'}=\mathbf{M}_{2\times2}\mathbf{v}$

在**三维**中的线性变换为 $\mathbf{v'}=\mathbf{M}_{3\times3}\mathbf{v}$

**常见的线性变换：**

1. 缩放 (scale)

![](<images/1686794682704.png>)

2. 镜像 (relfection)

![](<images/1686794682955.png>)

3. 错切 (shear)

![](<images/1686794682999.png>)

4. 旋转 (rotation)

![](<images/1686794683036.png>)

### **2. 平移 (不是线性变换)**

不能用 3x3 的矩阵表示平移

$x'=x+a_1\\y'=y+a_2\\z'=z+a_3$

矩阵表示： $\left[ \begin{array}{} {x’} \\ {y'}\\ {z'}\end{array} \right]=\left[ \begin{array}{} {x} \\ {y}\\ {z}\end{array} \right]+\left[ \begin{array}{} {a_1} \\ {a_2}\\ {a_3}\end{array} \right]$

向量表示： $\mathbf{v'}=\mathbf{v}+\mathbf{a}$

### **3. 仿射变换 (affine transform)**

仿射变换是线性变换和平移变换的合并，即**仿射变换 = 线性变换 + 平移**

仿射变换可以**用 4x4 的矩阵**表示

为此，我们需要把矢量扩展到四维空间下，这就是**齐次坐标空间 (homogeneous space)**

### **几种变换的所属类型：**

![](<images/1686794683091.png>)

### **4.5.2 齐次坐标 (homogenerous coordinate)**

三维矢量转为四维矢量 $\mathbf{v}=(x,y,z,w)$

*   点：**把其** $w$ **分量设为 1**，这样平移、旋转、缩放都会施加于该点
*   向量：**把其** $w$ **分量设为 0**，这样只有旋转和缩放会施加于该向量，平移将被忽略

### **4.5.3 4x4 平移旋转缩放矩阵的表示**

$\left[ \begin{array}{} {\mathbf{M}_{3\times3}} & {\mathbf{t}_{3\times1}}\\ {\mathbf{0}_{1\times3}} & {1} \end{array} \right]$

其中左上角的矩阵 $\mathbf{M}_{3\times3}$ 表示**旋转和缩放**； $\mathbf{t}_{3\times1}$ 表示**平移**； $\mathbf{0}_{1\times3}=\left[ \begin{array}{} 0 &0 &0 \end{array} \right]$ 是零**矩阵**；右下角是**标量 1**

### **4.5.4 平移矩阵**

**点**：（会平移）

$\left[ \begin{array}{} 1 & 0 & 0 & t_x \\0 & 1 &0 & t_y\\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} x \\y\\z\\1\end{array} \right]=\left[ \begin{array}{} x+t_x \\y+t_y\\z+t_z \\1\end{array} \right]$

**向量**：(不会平移)

$\left[ \begin{array}{} 1 & 0 & 0 & t_x \\0 & 1 &0 & t_y\\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} x \\y\\z\\0\end{array} \right]=\left[ \begin{array}{} x \\y\\z \\0\end{array} \right]$

平移矩阵的**逆矩阵**

$\left[ \begin{array}{} 1 & 0 & 0 & t_x \\0 & 1 &0 & t_y\\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1\end{array} \right]^{-1}=\left[ \begin{array}{} 1 & 0 & 0 & -t_x \\0 & 1 &0 & -t_y\\ 0 & 0 & 1 & -t_z \\ 0 & 0 & 0 & 1\end{array} \right]$

可以看出，平移矩阵**不是正交矩阵**

### **4.5.5 缩放矩阵**

**点**：（会缩放）

$\left[ \begin{array}{} k_x & 0 & 0 & 0 \\0 & k_y &0 & 0\\ 0 & 0 & k_z & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} x \\y\\z\\1\end{array} \right]=\left[ \begin{array}{} k_xx \\k_yy\\k_zz \\1\end{array} \right]$

**向量**：（会缩放）

$\left[ \begin{array}{} k_x & 0 & 0 & 0 \\0 & k_y &0 & 0\\ 0 & 0 & k_z & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} x \\y\\z\\0\end{array} \right]=\left[ \begin{array}{} k_xx \\k_yy\\k_zz \\0\end{array} \right]$

统一缩放： $k_x=k_y=k_z$ ; 反之为非统一缩放

**统一缩放**不会改变模型的角度和比例信息；而**非统一缩放**会改变与模型相关的角度和比例

（注：非统一缩放后的模型在对法线进行变换时，会出先错误，需要事先乘这个非统一缩放的逆矩阵后再进行法线的变换）

缩放矩阵的**逆矩阵**：

$\left[ \begin{array}{} k_x & 0 & 0 & 0 \\0 & k_y &0 & 0\\ 0 & 0 & k_z & 0 \\ 0 & 0 & 0 & 1\end{array} \right]^{-1}=\left[ \begin{array}{} \frac{1}{k_x} & 0 & 0 & 0 \\0 & \frac{1}{k_y} &0 & 0\\ 0 & 0 & \frac{1}{k_z} & 0 \\ 0 & 0 & 0 & 1\end{array} \right]$

缩放矩阵一般不是正交矩阵，除非 $k_x=k_y=k_z=1$

上述的矩阵只适用于沿坐标轴方向进行缩放

如果要在任意方向进行缩放，需要进行复合变换，具体步骤：

1.  将缩放轴通过旋转、移动、缩放 (如果不是标准正交基的话) 等操作变换成标准坐标轴
2.  同时要进行缩放的物体也进行和上面缩放轴一样的变换
3.  物体沿着变换后的坐标轴缩放
4.  物体再进行 (第 2 步的) 逆变换回到原来的缩放轴朝向

### **4.5.6 旋转矩阵**

绕 $x$ 轴旋转 $\theta$ 角： $\mathbf{M}_{rotate_X}(\theta)=\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & cos\theta &-sin\theta & 0\\ 0 & sin\theta & cos\theta & 0 \\ 0 & 0 & 0 & 1\end{array} \right]$

绕 $y$ 轴旋转 $\theta$ 角 ： $\mathbf{M}_{rotate_Y}(\theta)=\left[ \begin{array}{} cos\theta & 0 & sin\theta & 0 \\0 & 1 &0 & 0\\ -sin\theta & 0 & cos\theta & 0 \\ 0 & 0 & 0 & 1\end{array} \right]$

绕 $z$ 轴旋转 $\theta$ 角 ：$\mathbf{M}_{rotate_Z}(\theta)=\left[ \begin{array}{} cos\theta & -sin\theta & 0 & 0 \\sin\theta & cos\theta &0 & 0\\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1\end{array} \right]$

任意旋转 == 绕三轴旋转的矩阵相乘

在 Unity，旋转顺序是 **zxy**，如给定旋转角度 $(\theta_x,\theta_y,\theta_z)$ 则**组合后的旋转变换矩阵**是： $\mathbf{M}_{rotate_Z}\mathbf{M}_{rotate_X}\mathbf{M}_{rotate_Y}$

为什么旋转顺序是 **zxy**，但矩阵却没有按照从右往左的顺序： 原因是，这里每次旋转的只有物体，坐标系是不旋转的，（如果是坐标系跟着旋转，则旋转矩阵组合顺序是 $\mathbf{M}_{rotate_Y}\mathbf{M}_{rotate_X}\mathbf{M}_{rotate_Z}$

旋转矩阵的**逆矩阵**为**旋转相反角度**的变换矩阵

旋转矩阵是**正交矩阵**，多个旋转矩阵串联相乘也是正交的

### **4.5.7 复合变换**

符合变换可以通过矩阵的串联来实现（从右往左顺序：**缩放、旋转、平移**）

（$\mathbf{p}$ 是列向量）

$\mathbf{p}_{new}=\mathbf{M}_{translation}\mathbf{M}_{rotation}\mathbf{M}_{scale}\mathbf{p}_{old}$

为什么是这个顺序：

因为物体的起始位置是 (0,0,0), 若先平移再缩放，那么缩放后会再次改变物体位置

**（注意！：若旋转中心不在原点，则在缩放前还需加上一步平移）**

## **4.6 坐标空间**

### **4.6.1 为什么要使用这么多不同的坐标空间**

我们需要在不同的情况下使用不同的坐标空间，因为一些概念只有在特定的坐标空间下才有意义，才更容易理解

### **4.6.2 坐标空间的变换**

在渲染流水线中，我们往往需要把一个点或方向矢量 **从一个坐标空间转换到另一个坐标空间**

每个坐标空间都是另一个坐标空间的子空间

例如下图：球的中心有个坐标轴 (子坐标空间)，世界原点也有个坐标轴 (父坐标空间)

![](<images/1686794683393.png>)

对坐标空间的变换实际上就是**在父空间和子空间对 点和矢量 进行变换**

现在假设 父空间 $W$ ，以及一个子空间 $O$ ，我们一般有 2 种需求：

*   把子空间下的点或矢量 $\mathbf{p}_O$ 转换到父坐标空间下的 $\mathbf{p}_W$
*   把父空间下的点或矢量 $\mathbf{p}_W$ 转换到子坐标空间下的 $\mathbf{p}_O$

我们可以使用下面的公式来表示这 2 种需求：

$\mathbf{p}_W=\mathbf{M}_{(O\to W)}\mathbf{p}_O\\\mathbf{p}_O=\mathbf{M}_{(W\to O)}\mathbf{p}_W$

其中 $\mathbf{M}_{(O\to W)}$ 表示**从子空间变换到父空间**的变换矩阵； $\mathbf{M}_{(W\to O)}$ 表示**从父空间变换到子空间**的变换矩阵；

$\mathbf{M}_{(O\to W)}=\mathbf{M}_{(W\to O)}^{-1}$ , 即 2 者互为逆矩阵，所以只需解出 2 者之一即可，另一个可以通过求逆矩阵得到

_**1. 对点：**_

**下面我们来求解** $\mathbf{M}_{(O\to W)}$ **的表达式**

已知**子坐标空间** $O$ 的原点在**父坐标空间** $W$ 的位置是 $\mathbf{o}_O=(x_{\mathbf{o}_O},y_{\mathbf{o}_O},z_{\mathbf{o}_O})$ , **其三个坐标轴向量在父坐标空间的下表示为** $\mathbf{x}_O=(x_{\mathbf{x}_O},y_{\mathbf{x}_O},z_{\mathbf{x}_O})、\mathbf{y}_O=(x_{\mathbf{y}_O},y_{\mathbf{y}_O},z_{\mathbf{y}_O})、\mathbf{z}_O=(x_{\mathbf{z}_O},y_{\mathbf{z}_O},z_{\mathbf{z}_O})$

**例**：给定**子坐标空间**下的一点 $\mathbf{q}_O=(a,b,c)$ , 求该点在**父空间**下的坐标 $\mathbf{q}_W$

**答案是** $\mathbf{q}_W=\mathbf{o}_O+a\mathbf{x}_O+b\mathbf{y}_O+c\mathbf{z}_O$

**用矩阵表示**：

$\mathbf{q}_W=\left[ \begin{array}{} x_{\mathbf{o}_O} \\ y_{\mathbf{o}_O}\\z_{\mathbf{o}_O} \end{array} \right]+\left[ \begin{array}{} x_{\mathbf{x}_O} & x_{\mathbf{y}_O} & x_{\mathbf{z}_O} \\ y_{\mathbf{x}_O} & y_{\mathbf{y}_O} & y_{\mathbf{z}_O} \\ z_{\mathbf{x}_O} & z_{\mathbf{y}_O} & z_{\mathbf{z}_O} \end{array} \right]\left[ \begin{array}{} {a} \\ {b}\\{c} \end{array} \right]$

$=\left[ \begin{array}{} x_{\mathbf{o}_O} \\ y_{\mathbf{o}_O}\\z_{\mathbf{o}_O} \end{array} \right]+\left[ \begin{array}{} \mathbf{x}_O & \mathbf{y}_O & \mathbf{z}_O \\| & | & | \\ | & | & | \end{array} \right]\left[ \begin{array}{} {a} \\ {b}\\{c} \end{array} \right]$ 其中 “|" 符号表示向量按列展开

扩展到齐次坐标空间，用 4x4 矩阵合并 平移变换（注意平移变换在最左边）

$\mathbf{q}_W=\left[ \begin{array}{} 1 & 0 & 0 & x_{\mathbf{o}_O} \\0 & 1 &0 & y_{\mathbf{o}_O}\\ 0 & 0 & 1 & z_{\mathbf{o}_O} \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} \mathbf{x}_O & \mathbf{y}_O & \mathbf{z}_O & 0\\| & | & | &0\\ | & | & |&0\\0 & 0& 0 &1\end{array} \right]\left[ \begin{array}{} {a} \\ {b}\\{c}\\1 \end{array} \right]$

$=\left[ \begin{array}{} \mathbf{x}_O & \mathbf{y}_O & \mathbf{z}_O & x_{\mathbf{o}_O}\\| & | & | &y_{\mathbf{o}_O}\\ | & | & |&z_{\mathbf{o}_O}\\0 & 0& 0 &1\end{array} \right]\left[ \begin{array}{} {a} \\ {b}\\{c}\\1 \end{array} \right]$

$=\left[ \begin{array}{} \mathbf{x}_O & \mathbf{y}_O & \mathbf{z}_O & \mathbf{o}_O\\| & | & | &|\\ | & | & |&|\\0 & 0& 0 &1\end{array} \right]\left[ \begin{array}{} {a} \\ {b}\\{c}\\1 \end{array} \right]$

可以看出变换矩阵 $\mathbf{M}_{(O\to W)}$ 只实际上可以通过坐标空间 $O$ 在坐标空间 $W$ 中的原点和坐标轴的矢量表示来构建出来：**只需把 3 个坐标轴依次放入矩阵的前 3 列，把原点矢放到最后一列，再用 0 和 1 填充最后一行**即可

**注意**：并没有要求 $\mathbf{x}_O,\mathbf{y}_O,\mathbf{z}_O$ 是单位向量，如果存在缩放的话， $\mathbf{x}_O,\mathbf{y}_O,\mathbf{z}_O$ 很可能不是单位向量

**反向思维**：当变换矩阵 $\mathbf{M}_{(O\to W)}$ 已知时，可以通过变换矩阵 $\mathbf{M}_{(O\to W)}$ 反推模型空间在世界空间下的单位向量和位置，如变换矩阵 $\mathbf{M}_{(O\to W)}$ 的第一列归一化后 (消除缩放影响) 就是 $\mathbf{x}_O$ ，同理可以得到 $\mathbf{y}_O,\mathbf{z}_O$ ，最后 1 列就是模型空间坐标中心在世界空间坐标的位置

_**2. 对矢量**_

因为用不到平移，所以只需要提取**左上角 3x3 矩阵**即可

$\mathbf{M}_{(O\to W)}=\left[ \begin{array}{} \mathbf{x}_O & \mathbf{y}_O & \mathbf{z}_O \\| & | & | \\ | & | & | \end{array} \right]$

因此在 Shader 中，常常截取变换矩阵的前 3 行和前 3 列来对法线方向、光照方向来进行空间变换

当变换矩阵 $\mathbf{M}_{(O\to W)}$ 是**正交矩阵**时，可以通过**转置得到逆矩阵**，即 $\mathbf{M}_{(W\to O)}$

$\mathbf{M}_{(W\to O)}=\left[ \begin{array}{} \mathbf{x}_W & \mathbf{y}_W & \mathbf{z}_W \\| & | & | \\ | & | & | \end{array} \right]=\mathbf{M}_{(W\to O)}^{-1}=\mathbf{M}_{(W\to O)}^T=\left[ \begin{array}{} \mathbf{x}_O & - & - \\\mathbf{y}_O & - & - \\ \mathbf{z}_O & - & - \end{array} \right]$

可以发现，**在变换矩阵为正交矩阵的前提下**，我们已知任意一个空间关于另一个空间的关系，就能快速得到对应的变换矩阵

**例**：空间 $A\to B$ 的变换矩阵 $\mathbf{M}_{(A\to B)}=\left[ \begin{array}{} \mathbf{x}_A & \mathbf{y}_A& \mathbf{z}_A \\| & | & | \\ | & | & | \end{array} \right]=\left[ \begin{array}{} \mathbf{x}_B & - & - \\\mathbf{y}_B & - & - \\ \mathbf{z}_B & - & - \end{array} \right]$

（其中， $\mathbf{x}_A,\mathbf{y}_A,\mathbf{z}_A$ 是 $A$ 空间的 3 个坐标轴在 $B$ 空间的指向； $\mathbf{x}_B,\mathbf{y}_B,\mathbf{z}_B$ 是 $B$ 空间的 3 个坐标轴在 $A$ 空间的指向）

### **4.6.3 顶点的坐标空间变换过程**

顶点从最开始的 **模型空间** 变换到最终的 **屏幕空间**，得到真正的屏幕像素坐标

### **4.6.4 模型空间 (model space)**

**模型空间 (model space)** 也被称为对象空间 (object space) 或局部空间 (local space)

模型空间会跟着模型移动和旋转，可以理解为自己的” 前后左右上下 “

在 Unity 中模型空间使用的时左手坐标系，（所以对于某些用右手坐标系的建模软件导出模型时需要进行变换，通常是 z 轴反向）

一般模型坐标系的原点在模型的重心 (在建模时定义)

例：奶牛鼻子在模型空间的坐标,（扩展到齐次坐标系下）如下图

![](<images/1686794683430.png>)

我们可以在顶点着色器访问模型的顶点信息，其中就包含了每个顶点的模型空间坐标

### **4.6.5 世界空间 (world space)**

**世界空间 (world space)** 在 Unity 中同样使用左手坐标系，但它的 x,y,z 轴固定不变，在 Unity 中，可以通过调节模型的 Transform 组件中的 **Postion(位置)、Rotation(旋转)、Scale(缩放)** 来改变模型的参数。

Transform 组件下的值是相对于 **父级 (Parent)** 的 Transform 组件值为原值定义的；**如果没有父级**，则相对于世界坐标系的值, 即 **Postion(0,0,0),Rotation(0,0,0),Scale(1,1,1)**

**模型变换：**将**模型空间**的顶点坐标 变换到 **世界空间**的顶点坐标

根据 Transform 组件下的 3 个参数值，能够得到**模型空间**相对于**世界空间 (如果有父级，就是父级的模型空间)** 的关系, 再应用 **4.5.7 章节** 的知识，求变换矩阵

$\mathbf{M}_{(O\to W)}=\mathbf{M}_{translation}\mathbf{M}_{rotation}\mathbf{M}_{scale}$

**例子：**(已知奶牛的鼻子在模型空间中的坐标 $\mathbf{p}_O=(0,2,4,1)_O$ ，以及 Transform 的值，求奶牛鼻子在世界空间的坐标 \ mathbf{p}_W)

![](<images/1686794683754.png>)

解：1. 求变换矩阵（注意从右往左计算顺序为：缩放、旋转、平移)

$\mathbf{M}_{(O\to W)}=\mathbf{M}_{translation}\mathbf{M}_{rotation}\mathbf{M}_{scale}$

$=\left[ \begin{array}{} 1 & 0 & 0 & t_x \\0 & 1 &0 & t_y\\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} cos\theta & 0 & sin\theta & 0 \\0 & 1 &0 & 0\\ -sin\theta & 0 & cos\theta & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} k_x & 0 & 0 & 0 \\0 & k_y &0 & 0\\ 0 & 0 & k_z & 0 \\ 0 & 0 & 0 & 1\end{array} \right]$

$=\left[ \begin{array}{} 1 & 0 & 0 & 5 \\0 & 1 &0 & 0\\ 0 & 0 & 1 & 25 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} -0.866 & 0 & 0.5 & 0 \\0 & 1 &0 & 0\\ -0.5 & 0 & -0.866 & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} 2 & 0 & 0 & 0 \\0 & 2 &0 & 0\\ 0 & 0 & 2 & 0 \\ 0 & 0 & 0 & 1\end{array} \right]$

$=\left[ \begin{array}{} -1.732 & 0 & 1 & 5 \\0 & 2 &0 & 0\\ -1 & 0 & -1.732 & 25 \\ 0 & 0 & 0 & 1\end{array} \right]$

2. 对模型顶点的坐标进行变换

$\mathbf{p}_W=\mathbf{M}_{(O\to W)}\mathbf{p}_O=\left[ \begin{array}{} -1.732 & 0 & 1 & 5 \\0 & 2 &0 & 0\\ -1 & 0 & -1.732 & 25 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} {0} \\ {2}\\{4}\\1 \end{array} \right]=\left[ \begin{array}{} {9} \\ {4}\\{18.072}\\1 \end{array} \right]$

解得奶牛鼻子的世界坐标是 $(9,4,18.072)$ , 注意这里的浮点数都是近似值，这里近似到小数点后 3 位，这取决于 Unity 采用的浮点值精度

### **4.6.6 观察空间 (view space)**

**观察空间 (view space)** 也叫摄像机空间 (camera space)，观察空间可以理解为模型空间的特例，它的模型是摄像机

在观察中间中，摄像机位于坐标原点，在 Unity 中，观察空间区别于世界空间和物体空间，它是**_右手坐标系_**

**+x 轴指向 (相机镜头) 右方**，**+y 轴指向 (相机镜头) 上方**，**+z 轴指向相机后方**，即相机的正前方指向 - z 轴方向

（注：观察空间不是屏幕空间，观察空间是 3 维的，屏幕空间是 2 维的，从观察空间到屏幕空间需要进行投影变换）

**观察变换**：将顶点坐标从**世界空间**变换到**观察空间**中

获得观察变换矩阵：

法一：计算观察空间的三个坐标轴在世界空间下的表示，根据 4.6.2 节的方法，观察空间变换到世界空间的变换矩阵 $\mathbf{M}_{(V\to W)}$ , 再求逆矩阵得到 $\mathbf{M}_{(W\to V)}$

法二：将相机通过逆变换，让观察空间和世界空间重合，即让相机到原点，观察空间坐标轴与世界坐标轴重合，即直接求 $\mathbf{M}_{(W\to V)}$

**例子：**已知奶牛的鼻子在世界空间中的坐标 $\mathbf{p}_W=(9,4,18.02,1)_W$ ，以及相机的 Transform 的值，求奶牛鼻子在观察空间的坐标 $\mathbf{p}_V$

![](<images/1686794683792.png>)

解 (法二)：

**1. 已知了相机的 Transform 值，构造逆变换矩阵 (逆变换矩阵的顺序和原先相反，平移在最右边，即先平移到原点，再做旋转和缩放)**

$\mathbf{M}_{(W\to V)}=\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 1 &0 & 0\\ 0 & 0 & -1 & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & cos\theta &-sin\theta & 0\\ 0 & sin\theta & cos\theta & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} 1 & 0 & 0 & t_x \\0 & 1 &0 & t_y\\ 0 & 0 & 1 & t_z \\ 0 & 0 & 0 & 1\end{array} \right]$

$=\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 1 &0 & 0\\ 0 & 0 & -1 & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 0.866 &0.5 & 0\\ 0 & -0.5 & 0.866 & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 1 &0 & -10\\ 0 & 0 & 1 & 10 \\ 0 & 0 & 0 & 1\end{array} \right]$

$=\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 1 &0 & 0\\ 0 & 0 & -1 & 0 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 0.866 &0.5 & -3.66\\ 0 & -0.5 & 0.866 & 13.66 \\ 0 & 0 & 0 & 1\end{array} \right]$

$=\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 0.866 &0.5 & -3.66\\ 0 & 0.5 & -0.866 & -13.66 \\ 0 & 0 & 0 & 1\end{array} \right]$

**注意**：因为**观察空间使用的是右手坐标系**，所以在最前面的乘上矩阵 $\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 1 &0 & 0\\ 0 & 0 & -1 & 0 \\ 0 & 0 & 0 & 1\end{array} \right]$

**2. 对顶点进行变换：**

$\mathbf{p}_V=\mathbf{M}_{(W\to V)}\mathbf{p}_W=\left[ \begin{array}{} 1 & 0 & 0 & 0 \\0 & 0.866 &0.5 & -3.66\\ 0 & 0.5 & -0.866 & -13.66 \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} {9} \\ {4}\\{18.02}\\1 \end{array} \right]=\left[ \begin{array}{} {9} \\ {8.84}\\{-27.31}\\1 \end{array} \right]$

解得奶牛鼻子的观察空间坐标是 $(9 , 8.84 , -27.31)$ ,

### **4.6.7 (齐次) 裁剪空间 (clip space)**

**裁剪空间 (clip space，也被称为齐次裁剪空间)**，左手坐标系

顶点接下来要从观察空间转换到裁剪空间，这个变换矩阵叫做**裁剪矩阵 (clip matrix), 也叫投影矩阵 (projection matrix)**

裁剪空间的作用是保留看的到的图元，剔除看不到的图元，裁剪与 视锥体 边界相交的图元

**视锥体指的是空间中的一块区域，决定了相机可以看到的空间，此区域由六个平面包围，这些平面叫做裁剪平面**

视锥体有 2 种类型：“透视投影”、“正交投影”

**1. 透视投影**：近大远小，模拟人眼看世界（3D 游戏中往往使用）

**2. 正交投影**：完全保留物体的距离和角度，平行线会已知平行，所有网格一样大，（一般用于 2D 游戏或小地图等其他 HUD 元素）

![](<images/1686794683830.png>)

![](<images/1686794684199.png>)

视锥体的 6 块平面中，**近裁剪平面 (near clip plane)** 和**远裁剪平面 (far clip plane)** 决定了摄像机可以看到的_深度范围_

对于透视投影，想要知道一个顶点是否在视锥体内比较麻烦，因此我们通过一个投影矩阵把顶点转换到裁剪空间中

**关于投影矩阵：**

*   这里的投影不是真正意义上的将三维的点投影到二维平面，真正的投影发生在后面的**齐次除法**过程中
*   经过投影矩阵变换后，**顶点的** $w$ **值会改变**，新的 $w$ 值就表示可视范围，如果 $x,y,z$ 都位于这个范围内，则说明这个顶点位于裁剪空间内，即可视

### **1. 透视投影**

透视投影的视锥体由 **Camera 组件的参数**和 **Game 视图的横纵比**共同决定

*   Camera 组件的 **Field of View(简称 FOV)**——控制视锥体的张开角度
*   Clipping Planes 中的 **Near** 和 **Far**——控制近裁切平面和远裁切平面距离相机的远近

![](<images/1686794684236.png>)

通过上面的参数可求近裁切平面和远裁切平面的高度：

$nearClipPlaneHeight=2\cdot Near\cdot tan\frac{FOV}{2}\\farClipPlaneHeight=2\cdot Far\cdot tan\frac{FOV}{2}$

**视锥体的横纵比 (近 / 远的横纵比)** 由 Game 视图的横纵比和 Viewport Rect 中的 W 和 H 属性共同决定, 横纵比用 **Aspect** 表示

$Aspect=\frac{nearClipPlaneWidth}{nearClipPlaneHeight}=\frac{farClipPlaneWidth}{farClipPlaneHeight}$

**透视投影的投影矩阵**：现在，我们可以根据已知的 Near、Far、FOV、Aspect 的值来确定透视投影的**投影矩阵**,（推导略）

$\mathbf{M}_{frustum}=\left[ \begin{array}{} \frac{cot\frac{FOV}{2}}{Aspect} & 0 & 0 & 0 \\0 & cot\frac{FOV}{2} &0 & 0\\ 0 & 0 & -\frac{Far+Near}{Far-Near} & -\frac{2\cdot Far\cdot Near}{Far-Near} \\ 0 & 0 & -1 & 0\end{array} \right]$

**注意**：这里得到的投影矩阵是建立在 Unity 使用的观察空间为**右手坐标系**，且变换后 $z$ 分量的范围在 $[-w,w]$ 之间。但在 DirectX 图形接口中，我们要得到变换后 $z$ 分量的范围在 $[0,w]$ 之间，就需要对上面的投影矩阵进行一些更改

一个顶点乘投影矩阵，可由**观察空间 (view space) 变换到裁剪空间(clip space)** 中，结果如下：

$\mathbf{p}_{C}=\mathbf{M}_{frustum}\mathbf{p}_{V}$

$=\left[ \begin{array}{} \frac{cot\frac{FOV}{2}}{Aspect} & 0 & 0 & 0 \\0 & cot\frac{FOV}{2} &0 & 0\\ 0 & 0 & -\frac{Far+Near}{Far-Near} & -\frac{2\cdot Far\cdot Near}{Far-Near} \\ 0 & 0 & -1 & 0\end{array} \right]\left[ \begin{array}{} {x} \\ {y}\\{z}\\1 \end{array} \right]$

$=\left[ \begin{array}{} {x\frac{cot\frac{FOV}{2}}{Aspect}} \\ {ycot\frac{FOV}{2}}\\{-z\frac{Far+Near}{Far-Near}-\frac{2\cdot Far\cdot Near}{Far-Near}}\\-z \end{array} \right]$

从结果可以看出，这个投影矩阵本质就是对 $x,y,z$ 分量进行了不同程度的缩放（当然， $z$ 分量还做了一个平移），缩放的目的是为了方便裁剪

而且 $w$ 分量不再是 1，而是**原先** $z$ **的取反**结果。

现在我们就可以很容易判断一个顶点是否在视锥体内，如果一个顶点在视锥体内，那么它**变换后的坐标**必满足：

$-w \leqslant x \leqslant w\\-w \leqslant y \leqslant w\\-w \leqslant z \leqslant w$

下图：可以发现相比于变换之前，投影变换后更容易判断点是否在视锥体内

![](<images/1686794684277.png>)

从图中还能看出，裁剪 (投影) 矩阵会**改变空间的旋向性**：即空间从右手坐标系变换成了左手坐标系。这就意味着，离相机越远，z 值越大

### **2. 正交投影**

和透视投影类似，正交投影的视锥体也由 **Camera 组件的参数**和 **Game 视图的横纵比**共同决定

正交投影的**视锥体是长方体**

*   Camera 组件的 **Size** 属性控制视锥体的高度 (一般也会控制长度，若横纵比不变的话)
*   Clipping Planes 中的 **Near** 和 **Far**——控制近裁切平面和远裁切平面距离相机的远近

![](<images/1686794684566.png>)

通过上面的参数可求近裁切平面和远裁切平面的高度：

$nearClipPlaneHeight=farClipPlaneHeight=2\cdot Size$

相机的横纵比：Aspect

$Aspect=\frac{nearClipPlaneWidth}{nearClipPlaneHeight}=\frac{farClipPlaneWidth}{farClipPlaneHeight}$

**正交投影的投影 (裁剪) 矩阵**：根据已知的 Near、Far、Size、Aspect 的值来确定正交投影的**裁剪 (投影) 矩阵**,（推导略）

$\mathbf{M}_{ortho}=\left[ \begin{array}{} \frac{1}{Aspect\cdot Size} & 0 & 0 & 0 \\0 & \frac{1}{Size} &0 & 0\\ 0 & 0 & -\frac{2}{Far-Near} & -\frac{ Far+ Near}{Far-Near} \\ 0 & 0 & 0 & 1\end{array} \right]$

一个顶点和上述裁剪矩阵相乘：

$\mathbf{p}_{C}=\mathbf{M}_{ortho}\mathbf{p}_{V}$

$=\left[ \begin{array}{} \frac{1}{Aspect\cdot Size} & 0 & 0 & 0 \\0 & \frac{1}{Size} &0 & 0\\ 0 & 0 & -\frac{2}{Far-Near} & -\frac{ Far+ Near}{Far-Near} \\ 0 & 0 & 0 & 1\end{array} \right]\left[ \begin{array}{} {x} \\ {y}\\{z}\\1 \end{array} \right]$

$=\left[ \begin{array}{} {\frac{x}{Aspect\cdot Size}} \\ {\frac{y}{Size}}\\{-\frac{2z}{Far-Near}}-\frac{ Far+ Near}{Far-Near}\\1 \end{array} \right]$

和透视投影一样， $w$ 表示可视范围, 一个点在视锥体的内部满足：

$-1=-w \leqslant x \leqslant w=1\\-1=-w \leqslant y \leqslant w=1\\-1=-w \leqslant z \leqslant w=1$

![](<images/1686794684603.png>)

同样，裁剪 (投影) 矩阵改变了空间的旋向性(右手坐标系 $\to$ 左手坐标系），可以发现变换后，可视范围在一个立方体内

**例子：**在**透视投影**相机下，已知奶牛的鼻子在观察空间的坐标 $\mathbf{p}_V=(9,8.84,-27.31)_V$ , 以及相机的参数和 Game 视图的横纵比

![](<images/1686794684674.png>)

由图可知 FOV=60°，Near=5，Far=40,Aspect=4:3(左上角)=1.333

**1. 求投影矩阵：**

$\mathbf{M}_{frustum}=\left[ \begin{array}{} \frac{cot\frac{FOV}{2}}{Aspect} & 0 & 0 & 0 \\0 & cot\frac{FOV}{2} &0 & 0\\ 0 & 0 & -\frac{Far+Near}{Far-Near} & -\frac{2\cdot Far\cdot Near}{Far-Near} \\ 0 & 0 & -1 & 0\end{array} \right]$

$=\left[ \begin{array}{} 1.299 & 0 & 0 & 0 \\0 & 1.732 &0 & 0\\ 0 & 0 & -1.286 & -11.429\\ 0 & 0 & -1 & 0\end{array} \right]$

**2. 对顶点进行变换：**

$\mathbf{p}_{C}=\mathbf{M}_{frustum}\mathbf{p}_{V}$

$=\left[ \begin{array}{} \frac{cot\frac{FOV}{2}}{Aspect} & 0 & 0 & 0 \\0 & cot\frac{FOV}{2} &0 & 0\\ 0 & 0 & -\frac{Far+Near}{Far-Near} & -\frac{2\cdot Far\cdot Near}{Far-Near} \\ 0 & 0 & -1 & 0\end{array} \right]\left[ \begin{array}{} {x} \\ {y}\\{z}\\1 \end{array} \right]$

$=\left[ \begin{array}{} 1.299 & 0 & 0 & 0 \\0 & 1.732 &0 & 0\\ 0 & 0 & -1.286 & -11.429\\ 0 & 0 & -1 & 0\end{array} \right]\left[ \begin{array}{} {9} \\ {8.84}\\{-27.31}\\1 \end{array} \right]=\left[ \begin{array}{} {11.691} \\ {15.311}\\{23.692}\\27.31 \end{array} \right]$

解得顶点在**裁剪空间 (clip space)** 的位置—— $(11.691 , 15.311 , 23.692 , 27.31)$

**3. 判断是否在视锥体内**

$-w \leqslant x \leqslant w\to -27.31 \leqslant 11.691 \leqslant 27.31\\-w \leqslant y \leqslant w\to -27.31 \leqslant 15.311 \leqslant 27.31\\-w \leqslant z \leqslant w\to -27.31 \leqslant 23.692 \leqslant 27.31$

满足条件，所以该顶点在视锥体内，不需要被裁剪

### **4.6.8 屏幕空间 (screen space)**

将视锥体投影到**屏幕空间 (screen space)**，得到真正的 2D 像素位置

步骤：

1. 进行**标准齐次除法 (homogeneous division)**，也被称为**透视除法 (perspective)**，就是**用齐次坐标系 (裁剪空间) 的 w,y,z 分别除以 w 分量（即原观察空间的 - z 分量)。**

在 OpenGL 中，我们把这一步得到的坐标叫做**归一化的设备坐标 (Normalized Device Coordinates,NDC)**, 可以发现齐次裁剪空间转换到 **NDC 空间**中，透视投影的视锥体会变换到**一个立方体内**，OpenGL 的 $x,y,z$ 范围是 $[-1,1]$ ,DirectX 的 $z$ 范围是

$[0,1]$ ,Unity 选择 OpenGL 这样的齐次裁剪空间。

![](<images/1686794684709.png>)

因为**正交投影**的视锥体已经在立方体内，且 $w$ 分量已经为 1，所以变换后和变换前一样

![](<images/1686794684912.png>)

2.，映射至二维屏幕上，根据变换后的 $x,y$ 来映射输出窗口的对应像素坐标

在 Unity 中，屏幕空间的左下角的像素坐标是 $(0,0)$ ，右上角的像素坐标是 $(pixelWidh,pixelHeight)$ , 由于当前 $x$ 和 $y$ 的范围是 $[-1,1]$ , 所以需要进行一半的缩放，具体公式如下：（**大家可以想想这个公式的推导过程，其实很简单，想想为什么除以 2，为什么加 1/2, 因为范围要从 [-1,1] 变成[0,1]**）

$screen_x=\frac{clip_x\cdot pixelWidth}{2\cdot clip_w}+\frac{pixelWidth}{2}\\screen_y=\frac{clip_y\cdot pixelHeight}{2\cdot clip_w}+\frac{pixelHeight}{2}$

关于 $z$ 分量，通常会被用于深度缓冲，进行前后物体的深度比较，传统方式是把 $\frac{clip_z} {clip_w}$ , 即立方体内 (NDC 空间内) 的$z$ 值

（一般在计算完 NDC 后，会抛弃 $clip_w$ ，但有时候不会，它仍然会在后续的一些工作中起到重要的作用，例如进行透视校正插值。）

**在 Unity 中，其实我们只需要在顶点着色器完成顶点转换到上一步的裁剪空间即可，这一步从裁剪空间到屏幕空间的转换是由底层帮我们完成的**

**例子：**

紧接上一步，根据裁剪空间中奶牛的鼻子位置—— $(11.691 , 15.311 , 23.692 , 27.31)$ ，确定鼻子在像素上的位置 (有没被挡住都一样), 已知屏幕分辨率 (400x300)

$screen_x=\frac{clip_x\cdot pixelWidth}{2\cdot clip_w}+\frac{pixelWidth}{2}\\=\frac{11.691\cdot 400}{2\cdot 27.31}+\frac{400}{2}=285.617$

$screen_y=\frac{clip_y\cdot pixelHeight}{2\cdot clip_w}+\frac{pixelHeight}{2}\\=\frac{15.311\cdot 300}{2\cdot 27.31}+\frac{300}{2}=234.096$

解得奶牛鼻子的屏幕位置—— $(285.617,234.096)$

**4.6.8 小总结:** 这一步涉及到的空间有点多，我再次做了个小总结：

$裁剪空间(CS)\stackrel{标准齐次除法(透视除法)}{\Longrightarrow} NDC空间([-1,1]的正方体空间)\stackrel{映射}{\Longrightarrow}屏幕空间(SS)$

### **4.6.9 总结：**

以上就是一个顶点从**模型空间**变换到**屏幕空间**坐标的过程：“模型空间” $\to$ "世界空间" $\to$ “观察空间（右手系）” $\to$ “齐次裁剪空间”（非线性） $\to$ (NDC 空间）（非线性） $\to$ “屏幕空间 "

在顶点着色器中，我们是把**模型空间**变换到**齐次裁剪空间**即可，我们把需要进行的 **“模型变换”、“观察变换”、“投影变换” 串联成一个矩阵，即 MVP 矩阵**

Unity 会在后台计算片元在屏幕空间的像素位置

![](<images/1686794684972.png>)

其实除了这些空间，还有其他空间，如**切线空间 (tangent space)**, 通常用于法线映射

## **4.7 法线变换**

**法线 (normal)**，也叫法矢量 (normal vector)

模型的顶点往往携带额外的信息，而顶点法线就是其中一种信息，我们在变换模型的时候，不仅需要变换它的顶点，还需要变换顶点法线，一般在后续处理中计算光照

一般来说，绝大部分方向矢量可以通过 3x3 的变换矩阵 $\mathbf{M}_{(A\to B)}$ 把其从坐标空间 $A$ 变换到坐标空间 $B$ ，但在变换法线是，如果使用和顶点使用同一 $\mathbf{M}_{(A\to B)}$ ，那就可能无法确保维持变换后法线的垂直性 (于表面垂直)

求法线变换矩阵之前，先了解**切线 (tangent)**，也称为**切矢量 (tangent vector)**

切线和法线类似，也是模型顶点携带的信息，它通常与纹理空间 (UV 空间) 对齐，且与法线方向垂直，切线是由 2 个顶点之间的**插值** (梯度) 计算得到的

（具体有关切线的内容，可以看我的另一篇文章的 TBN 矩阵那块内容）

[TecrayC：凹凸贴图、法线贴图、TBN 矩阵讲解](https://zhuanlan.zhihu.com/p/412555049)

![](<images/1686794685258.png>)

由于是切线是由 2 个顶点插值得到的，因此我们可以直接使用顶点的变换矩阵 $\mathbf{M}_{(A\to B)}$ (这里因为是向量，不考虑平移变换，所以用 3x3 矩阵) 变换切线）

得到 $B$ 空间的切线方向：$\mathbf{t}_B=\mathbf{M}_{(A\to B)}\mathbf{t}_A$

我们使用 $\mathbf{G}_{(A\to B)}$ 来变换法线，会得到 $\mathbf{n}_B=\mathbf{G}_{(A\to B)}\mathbf{n}_A$

因为必须满足法线与切线垂直，即 $\mathbf{t}_B\cdot \mathbf{n}_B=(\mathbf{M}_{(A\to B)}\mathbf{t}_A)\cdot (\mathbf{G}_{(A\to B)}\mathbf{n}_A)=0$ (注：这里是向量表示)

用矩阵推导：

矩阵表示 $(\mathbf{M}_{(A\to B)}\mathbf{t}_A)\cdot (\mathbf{G}_{(A\to B)}\mathbf{n}_A)\stackrel{矩阵表示}{=}(\mathbf{M}_{(A\to B)}\mathbf{t}_A)^T (\mathbf{G}_{(A\to B)}\mathbf{n}_A)$

$=\mathbf{t}_A^T\mathbf{M}_{(A\to B)}^T \mathbf{G}_{(A\to B)}\mathbf{n}_A=\mathbf{t}_A^T(\mathbf{M}_{(A\to B)}^T \mathbf{G}_{(A\to B)})\mathbf{n}_A=0$

因为 矩阵表示 $\mathbf{t}_A\cdot \mathbf{n}_A\stackrel{矩阵表示}{=}\mathbf{t}_A^T\mathbf{n}_A=0$ , 所以必须 $\mathbf{M}_{(A\to B)}^T \mathbf{G}_{(A\to B)}=\mathbf{I}$ , 法线才可以与切线垂直

即 $\mathbf{G}_{(A\to B)}=(\mathbf{M}_{(A\to B)}^T)^{-1}=(\mathbf{M}_{(A\to B)}^{-1})^{T}$ ，**法线变换使用_原变换矩阵的逆转置矩阵_来变化法线**

**1**. 如果顶点变换**只包含旋转变换和平移变换**，那么这个 **3x3** 的变换矩阵 $\mathbf{M}_{(A\to B)}$ 就是**正交矩阵**（因为是对向量，这里特地强调一下 3x3，如果是对顶点，那么 4x4 的矩阵，就不能有平移变换）, 根据**正交矩阵的性质，转置矩阵 = 逆矩阵**，所以此时 $\mathbf{G}_{(A\to B)}=\mathbf{M}_{(A\to B)}$

**2**. 如果**有统一缩放**，那么就需要统一乘个系数（或者直接归一化）， $\mathbf{G}_{(A\to B)}=(\mathbf{M}_{(A\to B)}^{-1})^{T}=\frac{1}{k}\mathbf{M}_{(A\to B)}$

**3**. 如果**有非统一缩放**，那么就必须求逆矩阵了 $\mathbf{G}_{(A\to B)}=(\mathbf{M}_{(A\to B)}^{-1})^{T}$

补充：在 Unity 内置文件 "UnityCG.cginc" 中，法线变换只分了 2 种情况：有无非统一缩放

```
// Transforms direction from world to object space 
inline float3 UnityWorldToObjectDir( in float3 dir )      //向量空间变换OS->WS {
    return normalize(mul((float3x3)unity_WorldToObject, dir));
}

// Transforms normal from object to world space
inline float3 UnityObjectToWorldNormal( in float3 norm ) {
#ifdef UNITY_ASSUME_UNIFORM_SCALING   //假设全为统一缩放
    return UnityObjectToWorldDir(norm);
#else                                 //存在非统一缩放
    // mul(IT_M, norm) => mul(norm, I_M) => {dot(norm, I_M.col0), dot(norm, I_M.col1), dot(norm, I_M.col2)}
    return normalize(mul(norm, (float3x3)unity_WorldToObject));    //用逆矩阵，转置用行向量表示
#endif
}
```

## **4.8 Unity Shader 的内置变量（数学篇）**

Unity 内置了很多参数，本节将给出 Unity 内置的用于空间变换、摄像机、屏幕参数的内置变量，这些内置变量可以在 UnityShaderVariables.cginc 文件中找到定义和说明。

### **4.8.1 变换矩阵**

Unity 内置的变换矩阵 (Unity5.2 版本)

![](<images/1686794685304.png>)

补充：`//transpose()函数是将矩阵转置`

把顶点或方向矢量从观察空间变换到模型空间，我们可以使用下面的类似代码

```
//方法一：使用transpose函数对UNITY_MATRIX_IT_MV进行转置
 //得到UNITY_MATRIX_IT_MV的逆矩阵
 //然后列乘向量即可
 float4 posOS = mul(transpose(UNITY_MATRIX_IT_MV),posVS);
 ​
 //方法二：不进行矩阵的转置，直接使用 行向量相乘   ，这里就涉及到mul()函数的特点了
 float4 posOS = mul(posVS,UNITY_MATRIX_IT_MV);
```

### **4.8.2 摄像机和屏幕参数**

Unity(5,2 版本) 内置的相机 Camera 组件中的属性值：

![](<images/1686794685360.png>)

![](<images/1686794685404.png>)

## **4.9 答疑解惑**

### **4.9.1 使用 3x3 还是 4x4 的变换矩阵**

*   对于**线性变换** (旋转、缩放、镜像、错切...)，可以使用 3x3 变换矩阵
*   对于**顶点**变换，通常使用 4x4 变换矩阵
*   对于**方向矢量**，使用 3x3 变换矩阵即可

### **4.9.2 Cg 的矢量和矩阵类型**

矩阵的类型定义和声明, 以 float 为例：

```
//float3x3,float4x4     三阶矩阵，四阶矩阵
 //float3,float4         可以当成矢量、行矩阵、列矩阵
 ​
 //定义向量
 float4 a = float4(1.0 , 2.0 , 3.0 , 4.0);
 float4 b = float4(5.0 , 6.0 , 7.0 , 8.0);
 //定义矩阵（先行再列）
 float4x4 M= float4x4(a , b ,
                     10.0 , 11.0 , 12.0 , 13.0 ,
                     14.0 , 15.0 , 16.0 , 17.0);
                     //最终结果(1.0 , 2.0 , 3.0 ，4.0
                              5.0 ，6.0 ， 7.0 ，8.0
                              10.0 , 11.0 , 12.0 , 13.0
                              14.0 , 15.0 , 16.0 , 17.0)
 //向量点积操作
 float result = dot(a,b);
 //把a当成列矩阵和矩阵M右乘
 float4 column_mul_result = mul(M,a);
 //把a当成行矩阵和矩阵M左乘
 float4 row_mul_result = mul(a,M);
 ​
 //注意：矩阵不会自动转置，转置函数是tranpose
 //column_mul_result 不等于 row_mul_result,而是：
 //mul(M,a) == mul(a,tranpose(M))
 //mul(a,M) == mul(tranpose(M),a)
```

**mul 函数中参数的位置会影响结果**。在变换顶点时，我们通常使用右乘的方式按列向量进行乘法

Cg 定义矩阵使用**行优先**原则：(不论构造还是索引)

```
float3x3 M = float3x3(1.0,2.0,3.0,
 4.0,5.0,6.0,
 7.0,8.0,9.0);
 //得到M的第一行，即(1.0,2.0,3.0)
 float3 row = M[0];
 //得到M的第2行第1列元素,即4.0
 float ele = M[1][0];
```

（注：Unity 在脚本中提供了一种矩阵类型——Matrix4x4，脚本中的这个矩阵式按列优先方式的）


**另外参考资料：Games101**

## **再次声明**：

本文章的学习内容来源全部出自《UnityShader 入门精要》——冯乐乐

该文章只是本人我的学习笔记，里面对《UnityShader 入门精要》进行了些许概括且加了自己的些许理解

如果想更加具体地了解其内容，建议购买原著进行学习