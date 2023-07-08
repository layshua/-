### [GAMES101](https://so.csdn.net/so/search?q=GAMES101&spm=1001.2101.3001.7020) 投影矩阵推导详解和分析

*   [前言](#_1)
*   [GAMES101 投影矩阵相关坐标系的约定](#GAMES101_3)
*   [正交投影矩阵的推导](#_12)
*   *   [和 OpenGL 正交投影矩阵的比较](#OpenGL_31)
    *   [正交投影 clip space 中顶点的 w 值](#clip_spacew_60)
*   [透视投影矩阵的推导](#_63)
*   *   [推导透视投影 frustum 挤压到正交投影视景体的矩阵](#frustum_67)
    *   [最终的 GAMES101 透视投影矩阵](#GAMES101_102)
    *   [和 OpenGL 透视投影矩阵的比较](#OpenGL_127)
    *   [透视投影 clip space 中顶点的 w 值](#clip_spacew_177)
    *   [GAMES101 思考题：视景体挤压后 z 值为 (n+f)/2 的点会挤向 n 还是 f](#GAMES101znf2nf_179)

# 前言

之前推导过 [OpenGL](https://so.csdn.net/so/search?q=OpenGL&spm=1001.2101.3001.7020) 的投影矩阵，学了 GAMES101 之后，发现老师的推导方式很有意思，且 GAMES101 的坐标系约定和 OpenGL 不一样。最近在填新坑 [URasterizer](https://gitcode.net/n5/urasterizer) 的过程中，发现了一些问题，比如透视投影在 clip space 做裁剪时为啥 w 必须取反，以及之前 GAMES101 作业中做深度测试时为啥 z 值要取反的问题，因此重新推导 GAMES101 的[投影矩阵](https://so.csdn.net/so/search?q=%E6%8A%95%E5%BD%B1%E7%9F%A9%E9%98%B5&spm=1001.2101.3001.7020)并分析一下。由于 GAMES101 的推导思路很有趣，因此整个过程会轻松愉快很多。

# GAMES101 投影矩阵相关坐标系的约定

GAMES101 使用右手坐标系，包括 View space 和 NDC space(Clip space)。

*   对于 View space，camera 的位置为原点，看向负 Z 轴，这和 OpenGL 一致。GAMES101 推导投影矩阵时，近裁面坐标 n 和远裁面坐标 f 都是使用的是**坐标值**，因此有 f < n < 0。
*   对于 NDC space, GAMES101 的 NDC 中，x,y,z 的坐标范围都是 [-1,1]。虽然 OpenGL 也是 [-1,1]，但是需要注意的是，GAMES101 中，由于 View space 和 NDC space 都是右手系，所以变换后 z 轴方向并没有变化，因此 n 被映射到了 1，而 f 被映射到了 - 1，仍然符合 f < n，且。GAMES101 讲义上也说了，_**near and far not intuitive (n>f)**_ ，而 OpenGL 使用左手系的 NDC 更加直觉一些。
*   下图是右手系的 view space（以正交投影视景体为例），由于 camera 从原点看向负 Z 轴，因此 n 更靠近正 Z，f 更靠近负 Z。  
    
    ![](https://img-blog.csdnimg.cn/0484bb682d2340e687cff42f1dc307c2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)
    
*   当变换到 NDC 后，由于还是右手系，因此 n 映射到了 +1，f 映射到了 -1  
    
    ![](https://img-blog.csdnimg.cn/7045bcbeaabd486c8fb6185c76237331.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_19,color_FFFFFF,t_70,g_se,x_16)
    
      
    而对于 OpenGL，NDC 是左手坐标系，因此 n 被映射到 - 1，f 被映射到 1。（图就免了，上图反转 Z 轴）

# 正交投影矩阵的推导

由于正交投影只是把一个长方体变换到一个中心位于原点，坐标范围 [-1,1] 的立方体，因此通过移动加缩放即可完成变换。借用讲义中的图：  

![](https://img-blog.csdnimg.cn/712ecb105b8f44f1bb210f53fb79ca0b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
首先把长方体的原点移动到坐标系原点，由于 view space 中长方体的中心点为 P o = [ ( r + l ) / 2 , ( t + b ) / 2 , ( n + f ) / 2 ] Po = [(r+l)/2, (t+b)/2, (n+f)/2] Po=[(r+l)/2,(t+b)/2,(n+f)/2], 因此移动到原点只要移动 − P o -Po −Po 即可，写成平移矩阵就是：  

![](https://img-blog.csdnimg.cn/010320a9898147978d84fd05e0d487a8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_19,color_FFFFFF,t_70,g_se,x_16)

  
缩放也很简单，视景体在 x,y,z 三个轴上的长度分别是 ( r − l ) (r-l) (r−l), ( t − b ) (t-b) (t−b) 和 ( n − f ) (n-f) (n−f), 注意这儿都是大数减小数，上面说过 n>f，虽然它们都是小于 0 的负数，但 ( n − f ) (n-f) (n−f) 仍然是正数。而 NDC 的标准立方体的长宽高都是 2。因此 x,y,z 轴的缩放系数分别是 2 / ( r − l ) 2/(r-l) 2/(r−l), 2 / ( t − b ) 2/(t-b) 2/(t−b) 和 2 / ( n − f ) 2/(n-f) 2/(n−f)，写成缩放矩阵就是：  

![](https://img-blog.csdnimg.cn/44eb49f4deab42bba06aa775dbde3d94.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
将上面两个矩阵相乘，注意 GAMES101 和 OpenGL 一样，都是矩阵在左，向量在右进行向量变换的，因此矩阵连乘时，先起作用的矩阵在右边（更靠近向量），所以最终的正交投影矩阵就是：  

![](https://img-blog.csdnimg.cn/43f2731b90d3482cbc42e6fdc4f9e914.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
手动乘了一下，结果是：  
M o r t h o = [ 2 r − l 0 0 − r + l r − l 0 2 t − b 0 − t + b t − b 0 0 2 n − f − f + n n − f 0 0 0 1 ] M_{ortho} = \left[

$$\begin{matrix} \frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\ 0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\ 0 & 0 & \frac{2}{n-f} & -\frac{f+n}{n-f} \\ 0 & 0 & 0 & 1 \end{matrix}$$

\right]

Mortho​=⎣⎢⎢⎡​r−l2​000​0t−b2​00​00n−f2​0​−r−lr+l​−t−bt+b​−n−ff+n​1​⎦⎥⎥⎤​

## 和 OpenGL 正交投影矩阵的比较

如上所述，GAMES101 的投影坐标系约定和 OpenGL 基本一致，区别就在于 NDC 手向性不同。在参数方面，经典的 OpenGL 函数 g l O r t h o ( l e f t , r i g h t , t o p , b o t t o m . n e a r , f a r ) glOrtho(left, right, top, bottom. near, far) glOrtho(left,right,top,bottom.near,far), 前四个参数是左右上下剪裁面的坐标值，这和 GAMES101 一致。但 near, far 并不是坐标值（负数），而是距离 near/far plane 的距离值：

Specify the distances to the nearer and farther depth clipping planes. These distances are negative if the plane is to be behind the viewer.

既然是距离值，正常情况就是正数了。但是 glOrtho 中也可使用负数 near,far，但其含义并不是坐标值，而是表示平面在视点后面（这有什么意义？）。  
综合以上两点差异，对于 NDC 的差异，就是 z 轴方向反了，那么只要将 GAMES101 的变换矩阵再乘一个 Z 轴缩放 - 1 的矩阵就可以右手系变左手系，而 n,f 参数意义的差异，只要 n 和 f 各自取负就行。那么：  
M g l o r t h o = [ 1 0 0 0 0 1 0 0 0 0 − 1 0 0 0 0 1 ] ∗ [ 2 r − l 0 0 − r + l r − l 0 2 t − b 0 − t + b t − b 0 0 2 f − n f + n f − n 0 0 0 1 ] = [ 2 r − l 0 0 − r + l r − l 0 2 t − b 0 − t + b t − b 0 0 − 2 f − n − f + n f − n 0 0 0 1 ] M_{glortho} = \left [

$$\begin{matrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & -1 & 0 \\ 0 & 0 & 0 & 1 \end{matrix}$$

\right] * \left[

$$\begin{matrix} \frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\ 0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\ 0 & 0 & \frac{2}{f-n} & \frac{f+n}{f-n} \\ 0 & 0 & 0 & 1 \end{matrix}$$

\right] = \left[

$$\begin{matrix} \frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\ 0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\ 0 & 0 & \frac{-2}{f-n} & -\frac{f+n}{f-n} \\ 0 & 0 & 0 & 1 \end{matrix}$$

\right]

Mglortho​=⎣⎢⎢⎡​1000​0100​00−10​0001​⎦⎥⎥⎤​∗⎣⎢⎢⎡​r−l2​000​0t−b2​00​00f−n2​0​−r−lr+l​−t−bt+b​f−nf+n​1​⎦⎥⎥⎤​=⎣⎢⎢⎡​r−l2​000​0t−b2​00​00f−n−2​0​−r−lr+l​−t−bt+b​−f−nf+n​1​⎦⎥⎥⎤​

乘式右边的矩阵是 GAMES101 正交投影矩阵 n 和 f 分别取负的结果，最终得到了 OpenGL 的正交投影矩阵。

## 正交投影 clip space 中顶点的 w 值

我们知道投影矩阵的作用是将顶点从 view space 变换到 clip space（而不是 NDC）,NDC 是由 clip space 经过透视除法 ( x / w , y / w , z / w ) (x/w,y/w,z/w) (x/w,y/w,z/w) 得到的。对于正交投影，其实不存在透视除法，但是为了流水线的统一，还是需要经过一个除以 w 的过程。而我们推导的正交投影矩阵将顶点从 view space 变换到 clip space 后，其坐标的 w 值是 1（因为矩阵最后一行是 ( 0 , 0 , 0 , 1 ) (0,0,0,1) (0,0,0,1)）, 因此可兼容于流水线。  
为啥要强调一下 w 为 1 呢？我们思考一下这个问题，我们经常说 NDC 中的坐标范围为 [-1,1]，而 clip space 是[-w,w]。所以如果我们在 clip space 中判断一个点是否在视景体内，只要判断 -w <= p <= w 是否成立。但是以上式子成立的条件是 w > 0，因为如果 w < 0, 那么 clip space 的坐标范围就是[w, -w] 了，比较也要反过来。而正交投影时，w 为 1，是个正数，所以天然满足 -w <= p <= w，但是下面推导透视投影矩阵后会发现，在 GAMES101 的约定下，透视投影后，clip space 的 w 值是一个负数。

# 透视投影矩阵的推导

GAMES101 的推导方法很有趣，首先将透视投影的 Frustum 挤压成一个正交投影那样的长方体视景体，然后对长方体进行正交投影，得到 NDC 立方体：  

![](https://img-blog.csdnimg.cn/98d98e4c80a94f6dadf187ac10c77f4f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
所以主要的工作就是推导这个挤压矩阵。

## 推导透视投影 frustum 挤压到正交投影视景体的矩阵

看上面的图，需要分别沿着 x 轴和 y 轴进行挤压。先看 y 轴的情况，x 轴可以类比。  

![](https://img-blog.csdnimg.cn/e7bcc5236d2b46f2a54e5e3b289e488e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
这是讲义上的示例图，但要注意并不是说把 ( x , y , z ) (x,y,z) (x,y,z) 点挤压到 ( x ′ , y ′ , z ′ ) (x',y',z') (x′,y′,z′)，如果点在远裁面上，那么挤压后应该是在绿色点的位置，如果点在近裁面上，那么不用挤压了，就是 ( x ′ , y ′ , z ′ ) (x',y',z') (x′,y′,z′)。那么中间的点呢？显然挤压后是在绿色的虚线上，但是其 z 坐标如何变化，是向 n n n 移动，还是向 f f f 移动？这是 GAMES101 的思考题，下面再说。  
虽然示例图标识的不是挤压后点的位置，但是挤压后，y 坐标确实变成了 y’，利用相似三角形是可以得到 y’和 y 的关系：  
y ′ = n z y y' = \frac{n}{z}y y′=zn​y  
同样类比可得，挤压后 x’和 x 的关系：  
x ′ = n z x x' = \frac{n}{z}x x′=zn​x  
这里出现了除 z，而我们使用的是齐次坐标，因此可以将变换后的点乘以 z，得到表示同一个点的齐次坐标：  

![](https://img-blog.csdnimg.cn/a9ba84745400418590776e1da92679b8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
这样，我们需要推导的 Persp to ortho 矩阵的作用就是将 view space 的顶点变换到这样的齐次坐标：  

![](https://img-blog.csdnimg.cn/09192c7f701345a5a5cd11b8168ff874.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
从 x,y 坐标的对应关系: x = > n x , y = > n y x => nx, y => ny x=>nx,y=>ny, 可以填入矩阵的前两行：  

![](https://img-blog.csdnimg.cn/e9b0ccd9d28843a5a60a70d3e1a3653c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
为了解出最后一行，需要用到 z 坐标的映射关系，当 z 为 n 时，挤压后的点的 z 也是 n，因此齐次坐标的 z 就是 n 2 n^2 n2:  

![](https://img-blog.csdnimg.cn/e15f3e1bc909439fa126d202434abecc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
设第三行为 ( 0 , 0 , A , B ) (0, 0, A, B) (0,0,A,B)，有：  

![](https://img-blog.csdnimg.cn/e4f4599a07364a5d88cec6235c25b44c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
同样，位于远裁面的点，z 值为 f，挤压后仍然为 f，代入方程，可得：  

![](https://img-blog.csdnimg.cn/a661703f9d354ecca984f5db4eddc328.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAbjU=,size_20,color_FFFFFF,t_70,g_se,x_16)

  
利用上面两个关于 A 和 B 的方程式，可计算出 A 和 B:  

![](https://img-blog.csdnimg.cn/cfd0db7e68814d2b9d74c9e9b8c642f7.png)

  
这就得到了从透视投影变换到正交投影的矩阵：  
M p e r s p − > o r t h o = [ n 0 0 0 0 n 0 0 0 0 n + f − n f 0 0 1 0 ] M_{persp->ortho} = \left[

$$\begin{matrix} n & 0 & 0 & 0 \\ 0 & n & 0 & 0 \\ 0 & 0 & n+f & -nf \\ 0 & 0 & 1 & 0 \end{matrix}$$

\right]

Mpersp−>ortho​=⎣⎢⎢⎡​n000​0n00​00n+f1​00−nf0​⎦⎥⎥⎤​

## 最终的 GAMES101 透视投影矩阵

使用上面推导的正交投影矩阵乘上这个矩阵就得到最终的透视投影矩阵了：  
M p e r s p = M o r t h o M p e r s p − > o r t h o = [ 2 r − l 0 0 − r + l r − l 0 2 t − b 0 − t + b t − b 0 0 2 n − f − f + n n − f 0 0 0 1 ] [ n 0 0 0 0 n 0 0 0 0 n + f − n f 0 0 1 0 ] = M_{persp} = M_{ortho}M_{persp->ortho} = \left[

$$\begin{matrix} \frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\ 0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\ 0 & 0 & \frac{2}{n-f} & -\frac{f+n}{n-f} \\ 0 & 0 & 0 & 1 \end{matrix}$$

\right] \left[

$$\begin{matrix} n & 0 & 0 & 0 \\ 0 & n & 0 & 0 \\ 0 & 0 & n+f & -nf \\ 0 & 0 & 1 & 0 \end{matrix}$$

\right] =

Mpersp​=Mortho​Mpersp−>ortho​=⎣⎢⎢⎡​r−l2​000​0t−b2​00​00n−f2​0​−r−lr+l​−t−bt+b​−n−ff+n​1​⎦⎥⎥⎤​⎣⎢⎢⎡​n000​0n00​00n+f1​00−nf0​⎦⎥⎥⎤​=

[ 2 n r − l 0 − r + l r − l 0 0 2 n t − b − t + b t − b 0 0 0 n + f n − f − 2 n f n − f 0 0 1 0 ] \left[

$$\begin{matrix} \frac{2n}{r-l} & 0 & -\frac{r+l}{r-l} & 0 \\ 0 & \frac{2n}{t-b} & -\frac{t+b}{t-b} & 0 \\ 0 & 0 & \frac{n+f}{n-f} & -\frac{2nf}{n-f} \\ 0 & 0 & 1 & 0 \end{matrix}$$

\right] ⎣⎢⎢⎡​r−l2n​000​0t−b2n​00​−r−lr+l​−t−bt+b​n−fn+f​1​00−n−f2nf​0​⎦⎥⎥⎤​

## 和 OpenGL 透视投影矩阵的比较

如前所述，区别首先在于 NDC 的手向性，另外就是 OpenGL 的经典函数 g l F r u s t u m ( l e f t , r i g h t , b o t t o m , t o p , n e a r V a l , f a r V a l ) glFrustum(left, right, bottom, top, nearVal, farVal) glFrustum(left,right,bottom,top,nearVal,farVal), 前 4 个参数都是坐标值，而后两个是距离值（正值）。那么我们像正交投影一样操作，将 n 和 f 取负，并且左乘一个翻转 z 轴的缩放矩阵：  
M g l P e r s p = [ 1 0 0 0 0 1 0 0 0 0 − 1 0 0 0 0 1 ] ∗ [ − 2 n r − l 0 − r + l r − l 0 0 − 2 n t − b − t + b t − b 0 0 0 − n + f f − n − 2 n f f − n 0 0 1 0 ] = M_{glPersp} = \left [

$$\begin{matrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & -1 & 0 \\ 0 & 0 & 0 & 1 \end{matrix}$$

\right] * \left[

$$\begin{matrix} \frac{-2n}{r-l} & 0 & -\frac{r+l}{r-l} & 0 \\ 0 & \frac{-2n}{t-b} & -\frac{t+b}{t-b} & 0 \\ 0 & 0 & -\frac{n+f}{f-n} & -\frac{2nf}{f-n} \\ 0 & 0 & 1 & 0 \end{matrix}$$

\right]=

MglPersp​=⎣⎢⎢⎡​1000​0100​00−10​0001​⎦⎥⎥⎤​∗⎣⎢⎢⎡​r−l−2n​000​0t−b−2n​00​−r−lr+l​−t−bt+b​−f−nn+f​1​00−f−n2nf​0​⎦⎥⎥⎤​=

[ − 2 n r − l 0 − r + l r − l 0 0 − 2 n t − b − t + b t − b 0 0 0 f + n f − n 2 n f f − n 0 0 1 0 ] \left[

$$\begin{matrix} \frac{-2n}{r-l} & 0 & -\frac{r+l}{r-l} & 0 \\ 0 & \frac{-2n}{t-b} & -\frac{t+b}{t-b} & 0 \\ 0 & 0 & \frac{f+n}{f-n} & \frac{2nf}{f-n} \\ 0 & 0 & 1 & 0 \end{matrix}$$

\right] ⎣⎢⎢⎡​r−l−2n​000​0t−b−2n​00​−r−lr+l​−t−bt+b​f−nf+n​1​00f−n2nf​0​⎦⎥⎥⎤​

但是 OpenGL 的透视投影矩阵并不是这样的啊？别的不说，至少最后一行明明是

( 0 , 0 , − 1 , 0 ) (0,0,-1,0) (0,0,−1,0)

。这是因为 OpenGL 设计出来的矩阵，clip space 的 w 坐标值为

− Z v i e w -Z_{view} −Zview​

。而 GAMES101 的 w 值是

Z v i e w Z_{view} Zview​

。这是上面没有说的第 3 个差异。所以我们需要 w 值取反，对于齐次坐标来说，所有元素乘以同一个系数和原来的坐标是一样的，因此为了让 w 取反，只要所有坐标同时乘 - 1 就行。那么我们在前面再乘一个 x,y,z,w 都缩放 - 1 的矩阵：

M g l P e r s p = [ − 1 0 0 0 0 − 1 0 0 0 0 − 1 0 0 0 0 − 1 ] ∗ [ − 2 n r − l 0 − r + l r − l 0 0 − 2 n t − b − t + b t − b 0 0 0 f + n f − n 2 n f f − n 0 0 1 0 ] = M_{glPersp} = \left [

$$\begin{matrix} -1 & 0 & 0 & 0 \\ 0 & -1 & 0 & 0 \\ 0 & 0 & -1 & 0 \\ 0 & 0 & 0 & -1 \end{matrix}$$

\right] * \left[

$$\begin{matrix} \frac{-2n}{r-l} & 0 & -\frac{r+l}{r-l} & 0 \\ 0 & \frac{-2n}{t-b} & -\frac{t+b}{t-b} & 0 \\ 0 & 0 & \frac{f+n}{f-n} & \frac{2nf}{f-n} \\ 0 & 0 & 1 & 0 \end{matrix}$$

\right]= MglPersp​=⎣⎢⎢⎡​−1000​0−100​00−10​000−1​⎦⎥⎥⎤​∗⎣⎢⎢⎡​r−l−2n​000​0t−b−2n​00​−r−lr+l​−t−bt+b​f−nf+n​1​00f−n2nf​0​⎦⎥⎥⎤​=

[ 2 n r − l 0 r + l r − l 0 0 2 n t − b t + b t − b 0 0 0 − f + n f − n − 2 n f f − n 0 0 − 1 0 ] \left[

$$\begin{matrix} \frac{2n}{r-l} & 0 & \frac{r+l}{r-l} & 0 \\ 0 & \frac{2n}{t-b} & \frac{t+b}{t-b} & 0 \\ 0 & 0 & -\frac{f+n}{f-n} & -\frac{2nf}{f-n} \\ 0 & 0 & -1 & 0 \end{matrix}$$

\right] ⎣⎢⎢⎡​r−l2n​000​0t−b2n​00​r−lr+l​t−bt+b​−f−nf+n​−1​00−f−n2nf​0​⎦⎥⎥⎤​

这就对了。所以约定真的是对投影矩阵的影响很大，随便搞错一个地方，得到的矩阵就不对，算出来的坐标就需要在哪儿莫名其妙的取个负。

## 透视投影 clip space 中顶点的 w 值

如前所述，在 GAMES101 的约定下，最终 clip space 的 w 值为 Z v i e w Z_{view} Zview​，由于 Camera 从原点看向 - Z 轴，所以 view space 中，在 Frustum 内的顶点的 Z v i e w Z_{view} Zview​应该是一个负数，既 w 是负数。因此在 clip space 中使用 w 值判断点是否在 Frustum 内时，需要判断的区间为 [w,-w]。是不是很不直觉？我是在实现 URasterizer 的 clip 功能时发现的这个问题，一开始我就是直接判断 -w <= P <= w，结果是所有的点都被裁剪掉了，什么都没剩下。上面也说了，OpenGL 设计的 w 值为 − Z v i e w -Z_{view} −Zview​，由于 OpenGL 的约定下，符合条件的点的 Z v i e w Z_{view} Zview​也是负数，因此 w 就是正数，这样就科学多了。

## GAMES101 思考题：视景体挤压后 z 值为 (n+f)/2 的点会挤向 n 还是 f

在 GAMES101 课上，闫老师提出一个思考题，在挤压之后，近裁面和远裁面上的点的 z 坐标保持不变，那么中间的点的 z 坐标如何变化呢，是向 n n n 移动，还是向 f f f 移动，以 z 值为 ( n + f ) / 2 (n+f)/2 (n+f)/2 的点为例。  
这个问题很难用脑子想，反正我是想不通，所以直接算一下。设原来的点为 ( x , y , ( n + f ) / 2 ) (x,y,(n+f)/2) (x,y,(n+f)/2)，那么挤压后的点为：  
P = [ n 0 0 0 0 n 0 0 0 0 n + f − n f 0 0 1 0 ] ∗ [ x y n + f 2 1 ] = [ n x n y n 2 + f 2 2 n + f 2 ] = [ − − − − n 2 + f 2 n + f 1 ] P = \left[

$$\begin{matrix} n & 0 & 0 & 0 \\ 0 & n & 0 & 0 \\ 0 & 0 & n+f & -nf \\ 0 & 0 & 1 & 0 \end{matrix}$$

\right] * \left[

$$\begin{matrix} x\\ y \\ \frac{n+f}{2} \\ 1 \end{matrix}$$

\right] = \left[

$$\begin{matrix} nx\\ ny \\ \frac{n^2+f^2}{2} \\ \frac{n+f}{2} \end{matrix}$$

\right] = \left[

$$\begin{matrix} --\\ -- \\ \frac{n^2+f^2}{n+f} \\ 1 \end{matrix}$$

\right]

P=⎣⎢⎢⎡​n000​0n00​00n+f1​00−nf0​⎦⎥⎥⎤​∗⎣⎢⎢⎡​xy2n+f​1​⎦⎥⎥⎤​=⎣⎢⎢⎡​nxny2n2+f2​2n+f​​⎦⎥⎥⎤​=⎣⎢⎢⎡​−−−−n+fn2+f2​1​⎦⎥⎥⎤​

所以 z 值从

( n + f ) / 2 (n+f)/2 (n+f)/2

挤压到了

( n 2 + f 2 ) / ( n + f ) (n^2+f^2)/(n+f) (n2+f2)/(n+f)

, 用前值减去后值，化解之后得到：

− ( n − f ) 2 / 2 ( n + f ) -(n-f)^2/2(n+f) −(n−f)2/2(n+f)

由于 n 和 f 是负数，所以整个结果是正数，这说明原来的 z 坐标值大于挤压后的 z 坐标，因此中间点向近裁面移动了。