越来越熟练了，也可能是上一节内容太繁杂了，对比下觉得比较轻松。

业余翻译，不足之处请多多指教！

![](<images/1685518938360.png>)

## **4.3 四元数 Quaternions**

四元数是复数的扩展，尽管它是威廉 · 罗恩 · 汉密尔顿爵士（Sir William Rowan Hamilton）于 1843 年发明的，但直到 1985 年，Shoemake **[1633]** 才将它们引入[计算机图形学](https://so.csdn.net/so/search?q=%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9B%BE%E5%BD%A2%E5%AD%A6&spm=1001.2101.3001.7020)领域。**(1)** 四元数用于表示旋转和方向。它们在几种方面都优于欧拉角和矩阵。任何三维定向都可以表示为围绕特定轴的单个旋转。当给出了该轴和角度表示后，四元数的平移很简单，而在任一方向上的欧拉角转换都十分有挑战性。四元数可用于稳定方向和恒定插值，而欧拉角无法很好地完成这些操作。

复数（complex number）具有实部和虚部。每个复数都由两个实数表示，其中第二个实数乘以 

![](<images/1685518938411.png>)

。同样，四元数有四个部分。前三个值与旋转轴密切相关，旋转角影响所有四个部分（有关更多信息，请参见第 4.3.2 节）。每个四元数由四个实数表示，每个实数与一个不同的部分相关联。由于四元数具有四个分量，因此我们选择将它们表示为向量，但是为了区分它们，我们在它们上加了一个帽子：

![](<images/1685518939356.png>)

。我们从四元数的一些数学背景开始，然后将其用于构建各种有用的变换。

**作者注：**

**(1)** 公正地说，Robinson **[1502]** 在 1958 年就将四元数用于刚体模拟。

**引用：**

****[1633]**** Shoemake, Ken, “Animating Rotation with Quaternion Curves,” Computer Graphics (SIGGRAPH ’85 Proceedings), vol. 19, no. 3, pp. 245–254, July 1985. Cited on p. 73, 76, 80, 82

****[1502]**** Robinson, Alfred C., “On the Use of Quaternions in Simulation of Rigid-Body Motion,”Technical Report 58-17, Wright Air Development Center, Dec. 1958. Cited on p. 76

### 4.3.1 数学背景 Mathematical Background

我们从四元数的定义开始。

**定义** 四元数 

![](<images/1685518940283.png>)

可以用以下所有等效的方式定义。

![](<images/1685518941502.png>)

变量  

![](<images/1685518942250.png>)

  被称为四元数 

![](<images/1685518943036.png>)

 的实部（real  part）。虚部（ imaginary part）为

![](<images/1685518943852.png>)

，

![](<images/1685518944596.png>)

，

![](<images/1685518946413.png>)

 和 

![](<images/1685518947149.png>)

 称为虚单位（ imaginary units）。

对于虚部

![](<images/1685518947979.png>)

，我们可以使用所有法向向量运算，例如加法，缩放，点积，叉积等等。使用四元数的定义，得出两个四元数 

![](<images/1685518948749.png>)

和 

![](<images/1685518949560.png>)

之间的乘法运算，如下所示。注意，虚部的乘法是不可交换的。

**乘法（Multiplication）：**

![](<images/1685518950301.png>)

从该方程式可以看出，我们使用叉积和点积来计算两个四元数的乘法。  
除了四元数的定义外，还需要加法，共轭，范数和恒等式的定义：

**加法（Addition）：**

![](<images/1685518951183.png>)

**共轭（Conjugate）：**

**![](<images/1685518951933.png>)**

**归一化（Norm）：**

**![](<images/1685518952765.png>)**

**恒等式（Identity）：**

**![](<images/1685518953572.png>)**

当简化

![](<images/1685518954331.png>)

 时（如上所示），虚部将抵消，仅保留实部。归一化有时表示为

![](<images/1685518955076.png>)

  **[1105]**。上面的结果是，可以导出由

![](<images/1685518955917.png>)

 表示的乘法逆。方程 

![](<images/1685518956835.png>)

 对于逆必须成立（对于乘法逆来说是常见的）。我们从归一化的定义中得出一个公式：

![](<images/1685518957750.png>)

这给出了乘法逆，如下所示：

**逆（Inverse）：**

![](<images/1685518958487.png>)

逆公式使用标量乘法，它是从公式 4.3.1 中的乘法得出的运算：

![](<images/1685518959435.png>)

，

![](<images/1685518960233.png>)

，这意味着标量乘法是可交换的：

![](<images/1685518961899.png>)

我们可以从定义中轻松得出以下规则集合：

**共轭规则（Conjugate rules）：**

**![](<images/1685518962634.png>)**

**归一化规则（Norm rules）：**

![](<images/1685518963568.png>)

**乘法定律（Laws of Multiplication）(包含以下两个)：**

**线性度（Linearity）：**

**![](<images/1685518964508.png>)**

**![](<images/1685518965771.png>)**

单位四元数

![](<images/1685518966590.png>)

 使得

![](<images/1685518967333.png>)

。由此可以得出 

![](<images/1685518968389.png>)

可写为

![](<images/1685518969205.png>)

对于某些三维向量

![](<images/1685518970212.png>)

，例如 

![](<images/1685518971003.png>)

，因为

![](<images/1685518971745.png>)

当且仅当  

![](<images/1685518972668.png>)

 。如在下一节中将看到的，单位四元数非常适合以最有效的方式创建旋转和方向。但在此之前，将为单元四元数引入一些额外的操作。

对于复数，可以将二维单位向量写为

![](<images/1685518973587.png>)

 。四元数的等效项是

![](<images/1685518974429.png>)

单位四元数的对数和幂函数来自公式 4.41：

**对数：**

**![](<images/1685518975399.png>)**

![](<images/1685518976141.png>)

**幂：**

**![](<images/1685518977058.png>)**

* * *

**引用：**

****[1105]**** Maillot, Patrick-Giles, “Using Quaternions for Coding 3D Transformations,” in Andrew S.Glassner, ed., Graphics Gems, Academic Press, pp. 498–515, 1990. Cited on p. 77

### 4.3.2 四元数变换 Quaternion Transforms

![](<images/1685518978004.png>)

_图 4.9。由单位四元数_ 

_![](<images/1685518978036.png>)_

 _表示的旋转变换的图示。变换围绕轴 

![](<images/1685518979281.png>)

旋转 

![](<images/1685518980023.png>)

 弧度。_

现在，我们将研究四元数集的子类，即单位长度的子类，称为单位四元数（ unit quaternions）。关于单元四元数的最重要事实是，它们可以表示任何三维旋转，并且这种表示极其紧凑和简单。

现在，我们将说明使单元四元数对旋转和定向如此有用的原因。首先，将一个点或向量的四个坐标

![](<images/1685518981014.png>)

 放入四元数

![](<images/1685518981753.png>)

![](<images/1685518983004.png>)

 的分量中，并假设我们有一个单位四元数

![](<images/1685518984109.png>)

 。可以证明：

![](<images/1685518984859.png>)

绕轴 

![](<images/1685518985774.png>)

 旋转

![](<images/1685518986517.png>)

（并因此旋转点

![](<images/1685518987442.png>)

），旋转角度为 

![](<images/1685518988231.png>)

。注意，由于

![](<images/1685518989782.png>)

 是单位四元数，因此

![](<images/1685518991566.png>)

。见图 4.9。

![](<images/1685518992307.png>)

 的任何非零实数倍也表示相同的变换，这意味着 

![](<images/1685518993128.png>)

和

![](<images/1685518993925.png>)

 表示相同的旋转。也就是说，取反轴

![](<images/1685518994668.png>)

 和实部

![](<images/1685518995413.png>)

，将生成一个四元数，该四元数的旋转与原始四元数的旋转完全相同。这也意味着从矩阵中提取四元数可以返回 

![](<images/1685518996182.png>)

或

![](<images/1685518997065.png>)

。

给定两个单位四元数 

![](<images/1685518997985.png>)

和

![](<images/1685518998733.png>)

，首先将 

![](<images/1685518999652.png>)

然后应用于四元数 

![](<images/1685519000397.png>)

（可以解释为点 

![](<images/1685519001139.png>)

）的连接由式 4.44 给出：

![](<images/1685519001886.png>)

在此，

![](<images/1685519002839.png>)

 是表示单元四元数 

![](<images/1685519003755.png>)

和 

![](<images/1685519004498.png>)

的级联的单元四元数。

**矩阵转换（Matrix Conversion）**

由于通常需要组合几个不同的变换，并且大多数变换都是矩阵形式，因此需要一种将公式 4.43 转换为矩阵的方法。四元数 

![](<images/1685519005278.png>)

可以转换成矩阵

![](<images/1685519006029.png>)

，如公式 4.45 **[1633，1634]** 所示：

![](<images/1685519006941.png>)

在此，标量为

![](<images/1685519008681.png>)

。对于单元四元数，这简化为

![](<images/1685519009689.png>)

一旦构建了四元数，就无需计算三角函数（trigonometric functions），因此转换过程实际上是有效的。

从正交矩阵 

![](<images/1685519010611.png>)

到单位四元数 

![](<images/1685519011724.png>)

的反向转换要复杂得多。此过程的关键是公式 4.46 中的矩阵具有以下差异：

![](<images/1685519012788.png>)

这些等式的含义是，如果 

![](<images/1685519013705.png>)

是已知的，则可以计算向量 

![](<images/1685519014513.png>)

的值，从而得出

![](<images/1685519015270.png>)

。的踪迹由下式计算

![](<images/1685519016011.png>)

此结果将对单位四元数产生以下转换：

![](<images/1685519016750.png>)

      

![](<images/1685519017660.png>)

![](<images/1685519018579.png>)

      

![](<images/1685519019519.png>)

为了具有数值稳定的例程 **[1634]**，应该避免小数除法。因此，首先设置  

![](<images/1685519020532.png>)

，由此得出

![](<images/1685519021547.png>)

这又意味着

![](<images/1685519022886.png>)

 和 

![](<images/1685519025020.png>)

中的最大值确定 

![](<images/1685519025764.png>)

和 

![](<images/1685519026506.png>)

 中的哪个最大。如果 

![](<images/1685519027242.png>)

最大，则使用公式 4.49 得出四元数。否则，我们注意到以下情况成立：

![](<images/1685519028044.png>)

然后，使用上述方程式的适当公式来计算

![](<images/1685519029384.png>)

 和 

![](<images/1685519030259.png>)

的最大值，然后使用公式 4.47 计算 

![](<images/1685519031174.png>)

 剩下的部分。Sch¨uler**[1588]** 提出了一种无分支但使用四个平方根的变体。

* * *

**引用：**

****[1633********]**** Shoemake, Ken, “Animating Rotation with Quaternion Curves,” Computer Graphics (SIGGRAPH ’85 Proceedings), vol. 19, no. 3, pp. 245–254, July 1985. Cited on p. 73, 76, 80, 82

****[********1634]**** Shoemake, Ken, “Quaternions and 4 × 4 Matrices,” in James Arvo, ed., Graphics Gems II, Academic Press, pp. 351–354, 1991. Cited on p. 80

**球面线性插值（Spherical Linear Interpolation）**

球面线性插值是一种操作，即在给定两个单元四元数 

![](<images/1685519031912.png>)

和 

![](<images/1685519032739.png>)

以及参数

![](<images/1685519033481.png>)

的情况下，计算插值四元数。例如，这对于动画对象很有用。插值相机的方向没有用，因为插值期间相机的 “向上” 向量可能会倾斜，通常是一种干扰效果。

该运算的代数形式由下面的复合四元数 

![](<images/1685519034225.png>)

表示：

![](<images/1685519034965.png>)

但是，对于软件实现，以下形式更合适：（下边的 slerp 代表球面线性插值  spherical linear interpolation）

![](<images/1685519035881.png>)

为了计算该方程式所需的 

![](<images/1685519036661.png>)

，可以使用以下事实：

![](<images/1685519037404.png>)

  **[325]**。对于

![](<images/1685519038147.png>)

，slerp 函数计算（唯一 **(2)**）内插四元数，它们共同构成从

![](<images/1685519039058.png>)

 到 

![](<images/1685519040069.png>)

的二维单位球面上的最短弧。圆弧位于由

![](<images/1685519040810.png>)

，

![](<images/1685519041545.png>)

  和原点之间的平面与三维单位球面的交点形成的圆上。如图 4.10 所示。计算出的旋转四元数以固定速度绕固定轴旋转。这样的曲线具有恒定速度，因此加速度为零，称为测地曲线（ geodesic  curve）  **[229]**。球体上的大圆是通过原点和球体的平面相交而生成的，这种圆的一部分称为大圆弧（ great arc）。

![](<images/1685519042455.png>)

_图 4.10。单位四元数表示为单位球面上的点。slerp 函数用于在四元数之间进行插值，并且插值路径是球面上的大弧。请注意，从_ 

_![](<images/1685519042490.png>)_

 _到 

![](<images/1685519043475.png>)

 进行插值，以及从 

![](<images/1685519044218.png>)

到 

![](<images/1685519044957.png>)

 到 

![](<images/1685519045718.png>)

 进行插值，即使它们到达相同的方向，也是不一样的。_

slerp 函数非常适合在两个方向之间进行插值，并且表现良好（固定轴，恒定速度）。使用多个欧拉角插值时则不是这种情况。

实际上，直接计算一个 slerp 是昂贵的操作，因为它涉及调用三角函数（trigonometric functions）。Malyshau **[1114]** 讨论了将四元数集成到渲染管线中的问题。他指出，如果不使用 slerp 而是简单地在像素着色器中对四元数进行归一化，则 90 度角的三角形方向误差最大为 4 度。栅格化三角形时，此错误率可以接受。Li **[1039，1040]** 提供了更快的增量方法（incremental methods）来计算不牺牲任何准确性的 slerps。Eberly **[406]** 提出了一种仅使用加法和乘法来计算视力的快速技术。

当方向大于两个时，我们称  

![](<images/1685519046459.png>)

  可用，我们想从

![](<images/1685519047377.png>)

 到 

![](<images/1685519048308.png>)

 再到 

![](<images/1685519049254.png>)

进行插值，依此类推，直到 

![](<images/1685519049996.png>)

 为止，此时 slerp 可以直接使用。现在，当我们称

![](<images/1685519050986.png>)

 时，我们将使用

![](<images/1685519051967.png>)

 和 

![](<images/1685519052858.png>)

 作为 slerp 的参数。通过 

![](<images/1685519053764.png>)

 之后，我们将使用 

![](<images/1685519054682.png>)

和 

![](<images/1685519055424.png>)

作为 slerp 的参数。这将导致突然的抽搐现象出现在方向插值中，如图 4.10 所示。这类似于线性插补点时发生的情况。请参阅第 720 页的图 17.3 的右上部分。某些读者在阅读了第 17 章中的样条线（splines）后，可能希望重新阅读以下段落。

一种更好的插值方法是使用某种样条线。我们在 qˆi 和 qˆi + 1 之间引入四元数 aˆi 和 aˆi + 1。可以在四元数 qˆi，aˆi，aˆi + 1 和 qˆi + 1 的集合内定义球形三次插值。令人惊讶的是，这些额外的四元数的计算如下 **[404]**  **(3)** 所示：

![](<images/1685519056168.png>)

![](<images/1685519057113.png>)

 和 

![](<images/1685519057857.png>)

将用于使用光滑三次样条球面插值四元数，如公式 4.55 所示：

![](<images/1685519059006.png>)

从上面可以看出，squad 函数是通过使用 slerp 的重复球面插值法构建的（有关点的重复线性插值的信息，请参见第 17.1.1 节）。插值将通过初始方向 

![](<images/1685519059945.png>)

，但不是通过 

![](<images/1685519061171.png>)

来表示初始位置处的切线方向。

* * *

**作者注：**

**(2)** 当且仅当 

![](<images/1685519062811.png>)

和 

![](<images/1685519064069.png>)

不相反时。

**(3)** Shoemake **[1633]** 给出了另一个推导。

**引用：**

****[325]**** Dam, Erik B., Martin Koch, and Martin Lillholm, “Quaternions, Interpolation and Animation,” Technical Report DIKU-TR-98/5, Department of Computer Science, University of Copenhagen, July 1998. Cited on p. 81

****[229]**** do Carmo, Manfred P., Differential Geometry of Curves and Surfaces, Prentice-Hall, Inc., 1976. Cited on p. 81

****[1114]**** Malyshau, Dzmitry, “A Quaternion-Based Rendering Pipeline,” in Wolfgang Engel, ed., GPU Pro3 , CRC Press, pp. 265–273, 2012. Cited on p. 82, 210, 715

****[1039********]**** Li, Xin, “To Slerp, or Not to Slerp,” Game Developer, vol. 13, no. 7, pp. 17–23, Aug. 2006. Cited on p. 82

****[********1040]****  Li, Xin, “iSlerp: An Incremental Approach of Slerp,” journal of graphics tools, vol. 12, no. 1, pp. 1–6, 2007. Cited on p. 82

****[406]**** Eberly, David, “A Fast and Accurate Algorithm for Computing SLERP,” Journal of Graphics, GPU, and Game Tools, vol. 15, no. 3, pp. 161–176, 2011. Cited on p. 82

****[404]**** Eberly, David, 3D Game Engine Design: A Practical Approach to Real-Time Computer Graphics, Second Edition, Morgan Kaufmann, 2006. Cited on p. 82, 772, 829, 950, 951, 959, 976, 990

**从一个向量旋转到另一个向量（Rotation from One Vector to Another）**

常见的操作是通过最短路径从一个方向 s 转换到另一个方向 t。四元数的数学极大地简化了此过程，并显示了四元数与该表示形式的密切关系。首先，将

![](<images/1685519065027.png>)

和

![](<images/1685519065837.png>)

 归一化。然后计算称为

![](<images/1685519066582.png>)

 的单位旋转轴，其计算公式为

![](<images/1685519067489.png>)

 。接下来，

![](<images/1685519068230.png>)

 和

![](<images/1685519069145.png>)

 ，其中

![](<images/1685519070298.png>)

 是

![](<images/1685519071047.png>)

 和 

![](<images/1685519071957.png>)

 之间的角度。那么表示从

![](<images/1685519072703.png>)

到 

![](<images/1685519073466.png>)

旋转的四元数为

![](<images/1685519074369.png>)

 。实际上，使用半角关系和三角恒等式简化 

![](<images/1685519075170.png>)

 ，得到 **[1197]**

**![](<images/1685519075911.png>)**

当

![](<images/1685519076857.png>)

和

![](<images/1685519077895.png>)

指向几乎相同的方向时，以这种方式直接生成四元数（相对于叉积

![](<images/1685519078641.png>)

 的归一化）避免了数值不稳定 **[1197]**。当

![](<images/1685519080598.png>)

和

![](<images/1685519081706.png>)

指向相反的方向时，这两种方法都会出现稳定性问题，因为它们会被零除。当检测到这种特殊情况时，可以使用任何垂直于

![](<images/1685519082452.png>)

的旋转轴旋转到

![](<images/1685519083313.png>)

。

有时我们需要从 

![](<images/1685519084058.png>)

 到 

![](<images/1685519084796.png>)

旋转的矩阵表示。在对公式 4.46 进行一些代数和三角简化之后，旋转矩阵变为 **[1233]**

**![](<images/1685519085539.png>)**

在此等式中，我们使用了以下中间计算（intermediate calculations）：

![](<images/1685519086288.png>)

可以看出，由于简化，所有平方根和三角函数都消失了，因此这是创建矩阵的有效方法。请注意，公式 4.57 的结构类似于公式 4.30 的结构，并请注意，后一种形式是怎样做到不需要三角函数。

请注意，当 

![](<images/1685519087548.png>)

 和 

![](<images/1685519088457.png>)

平行或接近平行时必须小心，因为 

![](<images/1685519089199.png>)

。如果

![](<images/1685519089944.png>)

，那么我们可以返回单位矩阵。但是，如果

![](<images/1685519090872.png>)

，那么我们可以绕任何轴旋转 

![](<images/1685519091968.png>)

弧度。该轴可以作为 

![](<images/1685519092702.png>)

 与不平行于

![](<images/1685519093839.png>)

的任何其他向量之间的叉积找到（第 4.2.4 节）。莫勒和休斯使用 Householder 矩阵以不同的方式处理这种特殊情况 **[1233]**。

* * *

**引用：**

****[1197]**** Melax, Stan, “The Shortest Arc Quaternion,” in Mark DeLoura, ed., Game Programming Gems, Charles River Media, pp. 214–218, 2000. Cited on p. 83

****[1233]**** M¨oller, Tomas, and John F. Hughes, “Efficiently Building a Matrix to Rotate One Vector to Another,” journal of graphics tools, vol. 4, no. 4, pp. 1–4, 1999. Also collected in [112]. Cited on p. 83, 84

## **4.4 顶点混合 Vertex Blending**

想象一下，数字角色的手臂使用前臂和上臂两部分进行动画处理，如图 4.11 左侧所示。该模型可以使用刚体变换进行动画处理（第 4.1.6 节）。但是，这两个部分之间的关节将不会像真正的肘部。这是因为使用了两个单独的对象，因此，关节由来自这两个单独的对象的重叠部分组成。显然，最好只使用一个对象。但是，静态模型零件无法解决使接头具有柔性的问题。

顶点混合（Vertex blending）是解决此问题的一种流行解决方案 **[1037，1903]**。该技术还有其他几个名称，例如线性混合蒙皮（linear-blend skinning），包络（enveloping）或骨架子空间变形（skeleton-subspace deformation）。虽然此处介绍的算法的确切来源尚不清楚，但定义骨骼并使蒙皮对变化做出反应是计算机动画中的一个古老概念 **[1100]**。在最简单的形式中，前臂和上臂像以前一样分别进行动画处理，但是在关节处，两个部分通过有弹性的 “蒙皮” 相连。因此，该弹性部件将具有一组由前臂矩阵转换的顶点和另一组由上臂矩阵转换的顶点。与每个三角形使用单个矩阵相反，这导致三角形的顶点可以通过不同的矩阵进行变换。见图 4.11。

![](<images/1685519094821.png>)

_图 4.11。手臂由前臂和上臂组成，使用左侧的两个独立对象的刚体变换进行动画处理。肘部看起来不现实。在右侧，对一个对象使用顶点混合。最右边的手臂说明了当简单的蒙皮__将两个部分直接覆盖以覆盖肘部时发生的情况。最右边的手臂说明了使用顶点混合时发生的情况，并且某些顶点以不同的权重进行了混合：（2/3，1/3）表示顶点对上臂的变换的权重为 2/3，对前臂的变换的权重为。1/3。该图在最右边的插图中还显示了顶点混合的缺点。在这里，可以看到肘部内部的折叠。使用更多的骨骼和更精心选择的权重可以达到更好的效果。_

通过进一步执行这一步骤，可以使单个顶点可以通过几种不同的矩阵进行变换，并将得到的位置加权并混合到一起。这是通过为动画对象设置骨骼来完成的，其中每个骨骼的变换可能会通过用户定义的权重影响每个顶点。由于整个手臂可能是 “弹性的”，即所有顶点可能受到多个矩阵的影响，因此整个网格（mesh）通常称为（骨骼上的）蒙皮（skin）。见图 4.12。许多商业建模系统都具有相同的骨架骨骼建模功能。尽管名称如此，骨骼并不一定必须是刚性的。例如，Mohr 和 Gleicher **[1230]** 提出了添加附加关节以实现诸如肌肉隆起等效果的想法。James 和 Twigg **[813]** 讨论了使用可以挤压和拉伸的骨骼的动画蒙皮。

在数学上，这用公式 4.59 表示，其中 

![](<images/1685519094853.png>)

是原始顶点，而

![](<images/1685519095588.png>)

 是变换后的顶点，其位置取决于时间

![](<images/1685519096531.png>)

：

![](<images/1685519097401.png>)

有 

![](<images/1685519098184.png>)

个骨骼影响 

![](<images/1685519098925.png>)

的位置，这在世界坐标中表示出来。值 

![](<images/1685519099793.png>)

是顶点 

![](<images/1685519100534.png>)

的骨骼 

![](<images/1685519101274.png>)

的权重。

![](<images/1685519102101.png>)

矩阵从初始骨骼的坐标系转换为世界坐标。通常，骨骼的控制关节位于其坐标系的原点。例如，前臂骨骼将其肘关节移动到原点，而动画旋转矩阵将手臂的这一部分绕关节移动。

![](<images/1685519103030.png>)

 矩阵是第 

![](<images/1685519103772.png>)

个骨骼的世界变换，会随着时间变化以对对象进行动画处理，并且通常是多个矩阵的串联，例如以前的骨骼变换的层次结构和局部动画矩阵。

Woodland **[1903]** 深入讨论了一种维护和更新

![](<images/1685519104623.png>)

 矩阵动画函数的方法。每个骨骼都将一个顶点转换到相对于其自己的参照系的位置，并从一组计算点中插值最终位置。在一些蒙皮讨论中未明确显示矩阵

![](<images/1685519105392.png>)

，而是将其视为 

![](<images/1685519106127.png>)

的一部分。我们在这里介绍它是因为它是有用的矩阵，并且几乎总是矩阵级联过程的一部分。

在实践中，对于动画的每一帧，为每个骨骼连接矩阵

![](<images/1685519107055.png>)

 和 

![](<images/1685519108394.png>)

，并且每个结果矩阵都用于变换顶点。顶点 

![](<images/1685519109136.png>)

由不同骨骼的级联矩阵转换，然后使用权重

![](<images/1685519110237.png>)

进行混合，因此称为顶点混合（vertex blending）。权重是非负的，并且总和为 1，因此发生的事情是将顶点转换到几个位置，然后在其中进行插值。这样，对于所有的 

![](<images/1685519111066.png>)

，变换后的点

![](<images/1685519112005.png>)

 将位于点集 

![](<images/1685519112837.png>)

 （固定的

![](<images/1685519113577.png>)

）的凸包中。通常也可以使用公式 4.59 转换法线。根据所使用的变换（例如，如果骨骼被拉伸或压扁了很多），可能需要对

![](<images/1685519114321.png>)

 的逆进行转置，如第 4.1.7 节中所述。

顶点混合非常适合在 GPU 上使用。网格中的顶点集可以放置在静态缓冲区中，该缓冲区会一次发送到 GPU 并重新使用。在每个框架中，只有骨骼矩阵会发生变化，而顶点着色器会计算它们对存储的网格的影响。这样，可以最大程度地减少在 CPU 上处理和从 CPU 传输的数据量，从而使 GPU 可以有效地渲染网格。如果可以将模型的整个骨矩阵一起使用，则是最简单的。否则，必须拆分模型并复制一些骨骼。或者，可以将骨骼变换存储在顶点访问的纹理中，从而避免达到寄存器存储限制。通过使用四元数表示旋转，每个变换可以仅存储在两个纹理中 **[1639]**。如果可用，无序访问视图存储将允许重新使用蒙皮结果 **[146]**。

我们可以指定超出

![](<images/1685519115252.png>)

范围或不等于 1 的权重集。但是，这仅在使用某些其他混合算法（例如变形目标，morph targets）（第 4.5 节）时才有意义。

基本顶点融合的一个缺点是可能发生不必要的折叠，扭曲和自相交 **[1037]**。见图 4.13。更好的解决方案是使用双季铵盐 **[872，873]**。这种执行蒙皮的技术有助于保持原始变换的刚度，因此避免了四肢的 “糖果包裹” 扭曲。计算量不到线性蒙皮混合的成本的 1.5 倍，并且效果很好，这导致该技术的快速采用。然而，双四元数蒙皮会导致鼓起效果，Le 和 Hodgins **[1001]** 提出了旋转中心蒙皮作为更好的选择。他们基于这样的假设：局部变换应该是刚体，并且具有相似权重

![](<images/1685519116010.png>)

 的顶点应该具有相似的变换形式。预先为每个顶点计算旋转中心，同时施加正交（刚体）约束以防止肘关节塌陷和糖果包装纸扭曲伪像（candy wrapper twist artifacts）。在运行时，该算法类似于线性混合蒙皮，因为 GPU 在旋转中心执行线性混合蒙皮，随后执行四元数混合步骤。

![](<images/1685519117058.png>)

_图 4.13。左侧显示了使用线性混合蒙皮时关节处的问题。在右侧，使用双四元数混合可以改善外观。（图片由 Ladislav Kavan 等人提供，Paul Steed **[1693]** 提供模型。）_

* * *

**引用：**

****[1037********]**** Lewis, J. P., Matt Cordner, and Nickson Fong, “Pose Space Deformation: A Unified Approach to Shape Interpolation and Skeleton-Driven Deformation,” in SIGGRAPH ’00: Proceedings of the 27th Annual Conference on Computer Graphics and Interactive Techniques, ACM Press/Addison-Wesley Publishing Co., pp. 165–172, July 2000. Cited on p. 84, 87, 90, 102

****[********1903]**** Woodland, Ryan, “Filling the Gaps—Advanced Animation Using Stitching and Skinning,” in Mark DeLoura, ed., Game Programming Gems, Charles River Media, pp. 476–483, 2000. Cited on p. 84, 85

****[1100]**** Magnenat-Thalmann, Nadia, Richard Laperri`ere, and Daniel Thalmann, “Joint-Dependent Local Deformations for Hand Animation and Object Grasping,” in Graphics Interface ’88, Canadian Human-Computer Communications Society, pp. 26–33, June 1988. Cited on p. 85

****[1230]****  Mohr, Alex, and Michael Gleicher, “Building Efficient, Accurate Character Skins from Examples,” ACM Transactions on Graphics (SIGGRAPH 2003), vol. 22, no. 3, pp. 562–568, 2003. Cited on p. 85

****[813]**** James, Doug L., and Christopher D. Twigg, “Skinning Mesh Animations,” ACM Transactions on Graphics, vol. 23, no. 3, pp. 399–407, Aug. 2004. Cited on p. 85

****[1903]**** Woodland, Ryan, “Filling the Gaps—Advanced Animation Using Stitching and Skinning,” in Mark DeLoura, ed., Game Programming Gems, Charles River Media, pp. 476–483, 2000. Cited on p. 84, 85

****[1639]**** Sikachev, Peter, Vladimir Egorov, and Sergey Makeev, “Quaternions Revisited,” in Wolfgang Engel, ed., GPU Pro5, CRC Press, pp. 361–374, 2014. Cited on p. 87, 210, 715

****[146]**** Bilodeau, Bill, “Vertex Shader Tricks: New Ways to Use the Vertex Shader to Improve Performance,” Game Developers Conference, Mar. 2014. Cited on p. 51, 87, 514, 568, 571, 798

****[1037]**** Lewis, J. P., Matt Cordner, and Nickson Fong, “Pose Space Deformation: A Unified Approach to Shape Interpolation and Skeleton-Driven Deformation,” in SIGGRAPH ’00: Proceedings of the 27th Annual Conference on Computer Graphics and Interactive Techniques, ACM Press/Addison-Wesley Publishing Co., pp. 165–172, July 2000. Cited on p. 84, 87, 90, 102

****[872********]**** Kavan, Ladislav, Steven Collins, Jiˇr´ı Z´ara, and Carol O’Sullivan, “Skinning with Dual Quaternions,” in Proceedings of the 2007 Symposium on Interactive 3D Graphics and Games, ACM, pp. 39–46, Apr.–May 2007. Cited on p. 87

****[********873]**** Kavan, Ladislav, Steven Collins, Jiˇr´ı Z´ara, and Carol O’Sullivan, “Geometric Skinning with Approximate Dual Quaternion Blending,” ACM Transactions on Graphics, vol. 27, no. 4, pp. 105:1–105:23, 2008. Cited on p. 87

****[1001]**** Le, Binh Huy, and Jessica K. Hodgins, “Real-Time Skeletal Skinning with Optimized Centers of Rotation,” ACM Transactions on Graphics, vol. 35, no. 4, pp. 37:1–37:10, 2016. Cited on p. 87