赶在 2019 结束之前把第三章结束，提前祝大家新年快乐！

![](<images/1685518897107.png>)

## 3.7 几何[着色器](https://so.csdn.net/so/search?q=%E7%9D%80%E8%89%B2%E5%99%A8&spm=1001.2101.3001.7020) The Geometry Shader

几何着色器可以将图元转换为其他图元，而这在细分阶段是无法完成的。例如，可以通过让每个三角形创建线边缘，将三角形网格转换为线框视图。或者，可以将这些线替换为面向观察者的四边形，从而使线框渲染的边缘更粗 **[1492]**。几何着色器是在 2006 年底随 DirectX 10 发行版添加到硬件加速的图形管道中的。它位于管道中的细分着色器之后，并且可以选择使用。虽然是 [Shader](https://so.csdn.net/so/search?q=Shader&spm=1001.2101.3001.7020) Model 4.0 的必需部分，但在较早的着色器模型中未使用它。OpenGL 3.2 和 OpenGL ES 3.2 也支持这种类型的着色器。

几何着色器的输入是单个对象及其关联的顶点。对象通常由带状（strip），线段（line segment）或点（point）构成的三角形所组成。扩展的图元可以由几何着色器定义和处理。特别是，可以传入三角形外部的三个附加顶点，并且可以使用折线上的两个相邻顶点。参见图 3.12。使用 DirectX 11 和 Shader Model 5.0，你可以传入多达 32 个控制点的更精细的补丁程序。也就是说，细分阶段对于补丁生成更有效 **[175]**。

几何着色器处理该图元并输出零个或多个顶点，这些顶点被视为点（points），折线（polylines）或三角形带（triangles）。请注意，几何着色器根本无法生成任何输出。通过这种方式，可以通过编辑顶点，添加新图元以及删除其他图元来选择性地修改网格。

几何着色器设计用于修改传入的数据或制作有限数量的副本（copies）。例如，一种用途是生成六个转换后的数据副本，以同时渲染立方体贴图的六个面； 参见第 10.4.3 节。它也可以用来有效地创建级联的阴影贴图（cascaded shadow maps），以生成高质量的阴影。利用几何着色器的其他算法包括从点数据创建尺寸可变的粒子，沿着轮廓拉伸鳍（fins）以进行毛发渲染以及为着色算法找到对象边缘。有关更多示例，请参见图 3.13。这些和其他用途将在本书的其余部分中讨论。

![](<images/1685518897154.png>)

_图 3.13。几何着色器（GS）的某些用途。左侧图，使用 GS 快速进行元球等值面细分。中间图，使用 GS 完成线段的分形细分并将其输出，而 GS 生成广告牌以显示闪电效果。右侧图，通过使用流输出的顶点和几何着色器执行布料模拟。（图片来自 NVIDIA SDK 10 [1300] 示例，由 NVIDIA Corporation 提供。）_

DirectX 11 增加了几何着色器使用实例化的功能，其中几何着色器可以在任何给定的图元上运行设定的次数 **[530，1971]**。在 OpenGL 4.0 中，这是通过调用计数指定的。几何着色器最多也可以输出四个流。可以在渲染管道上发送一个流以进行进一步处理。所有这些流都可以选择发送到流输出渲染目标。

保证几何着色器以与输入相同的顺序从图元输出结果。这会影响性能，因为如果多个着色器内核并行运行，则必须保存和排序结果。此因素和其他因素不利于在单个调用中用于复制或创建大量几何图形的几何着色器 **[175，530]**。

发出绘制调用后，管线中只有三个位置可以在 GPU 上创建工作：光栅化，细分阶段和几何体着色器。其中，考虑到所需的资源和内存，几何着色器的行为是最不可预测的，因为它是完全可编程的。实际上，几何着色器通常用得很少，因为它无法很好地映射到 GPU 的优势。在某些移动设备上，它是通过软件实现的，因此在此强烈建议不要使用它 **[69]**。

**引用：**

**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040

**[1971]** Zink, Jason, Matt Pettineo, and Jack Hoxley, Practical Rendering & Computation with Direct3D 11, CRC Press, 2011. Cited on p. 47, 54, 90, 518, 519, 520, 568, 795, 813, 814, 914

**[175]** Blythe, David, “The Direct3D 10 System,” ACM Transactions on Graphics, vol. 25, no. 3, pp. 724–734, July 2006. Cited on p. 29, 39, 42, 47, 48, 50, 249

**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040

**[69]** ARM Limited, “ARM R MaliTMApplication Developer Best Practices, Version 1.0,” ARM documentation, Feb. 27, 2017. Cited on p. 48, 798, 1029

### **3.7.1 流输出 Stream Output**

GPU 管线的标准使用方法是通过顶点着色器发送数据，然后光栅化生成三角形并在像素着色器中进行处理。在以前，数据总是通过管线传递，而中间结果无法访问。流输出（stream output）的想法是在 Shader Model 4.0 中引入的。在顶点着色器（以及可选的细分和几何着色器）处理了顶点之后，除了可以发送到光栅化阶段之外，还可以将它们输出到流（即有序数组）中。实际上，光栅化可以完全关闭，然后将流水线纯粹用作非图形流处理器。可以将通过这种方式处理的数据通过管线发送回去，从而允许进行迭代处理。如第 13.8 节所述，这种类型的操作可用于模拟流水或其他粒子效果。它也可以用于为模型蒙皮，然后使这些顶点可重复使用（第 4.4 节）。

流输出仅以浮点数的形式返回数据，因此可能会产生明显的内存开销。流输出在图元上起作用，而不是在顶点上起作用。如果沿管线发送网格，则每个三角形将生成自己的三个输出顶点集。原始网格中共享的所有顶点都将丢失。因此，更典型的用法是仅通过管线将顶点发送为点集图元（point set primitive）。在 OpenGL 中，流输出阶段称为变换反馈（transform feedback），因为它的大部分使用重点是变换顶点并将其返回以进行进一步处理。保证按输入顺序将基元发送到流输出目标，这意味着将保持顶点顺序 **[530]**。

**引用：**

**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040

## **3.8 像素着色器 The Pixel Shader**

顶点，曲面细分和几何体着色器执行完操作后，便会裁剪并设置图元以进行光栅化，如上一章所述。流水线的这一部分在其处理步骤中是相对固定的，即，不是可编程的，而是有些可配置的。遍历每个三角形以确定其覆盖哪些像素。光栅化器还可以粗略计算出三角形覆盖每个像素的像元（pixel’s cell）区域的数量（第 5.4.2 节）。部分或完全重叠像素的三角形称为片元（fragment）。

三角形顶点的值（包括 z 缓冲区中使用的 z 值）将在每个像素的三角形表面上插值。这些值将传递到像素着色器，然后由该着色器处理片元。在 OpenGL 中，像素着色器称为片元着色器，这也许是一个更好的名称。为了保证一致性，我们在本书中始终使用 “像素着色器”。沿管线发送的点和线图元也会为所覆盖的像素创建片元。

跨整个三角形执行的插值类型由像素着色器程序指定。通常，我们使用透视校正内插法（perspective-correct interpolation），以便像素表面位置之间的世界空间距离随着对象后退距离的增加而增加。一个示例是渲染延伸到地平线的铁轨。铁轨在铁轨较远的地方间距更近，因为每个接近地平线的连续像素行进的距离都更大。其他插值选项也可用，例如屏幕空间插值，其中不考虑透视投影。DirectX 11 进一步控制何时以及如何执行插值 **[530]**。

![](<images/1685518897248.png>)

_图 3.14。用户定义的裁剪平面。左侧图为单个水平裁剪平面将对象切片。中间图为嵌套球被三个平面修剪。右侧图为仅当球体的曲面在所有三个剪切平面的外部时，才对其进行裁剪。（来自 Three.js 示例中的 webgl 裁剪和 webgl 裁剪交集 **[218]**。）_

用编程术语来说，顶点着色器程序的输出（插在三角形（或线）上）实际上成为像素着色器程序的输入。随着 GPU 的发展，其他输入也已公开。例如，片元的屏幕位置可用于 Shader Model 3.0 及更高版本中的像素着色器。同样，三角形的哪一侧可见是输入标志。该知识对于一次通过每个三角形的正面和背面渲染不同的材质非常重要。

有了输入，通常像素着色器会计算并输出片元的颜色。它还可能会产生不透明度值（opacity value），并可以选择修改其 z 深度。在合并期间，这些值用于修改存储在像素处的内容。光栅化阶段生成的深度值也可以由像素着色器修改。模板缓冲区值通常是不可修改的，而是传递到合并阶段。DirectX 11.3 允许着色器更改此值。雾计算和 alpha 测试等操作已从 SM 4.0 中的合并操作变为像素着色器计算 **[175]**。

像素着色器还具有丢弃传入片元（即不生成任何输出）的独特功能。图 3.14 显示了如何使用片元丢弃的一个示例。裁剪平面功能以前是固定功能管道中的可配置元素，后来在顶点着色器中指定。有了片元丢弃功能之后，就可以用像素着色器中所需的任何方式来实现此功能，例如确定裁剪量应进行 “与” 运算还是 “或” 运算。

最初，像素着色器只能输出到合并阶段，以进行最终显示。随着时间的推移，像素着色器可以执行的指令数量已大大增加。这种增加引起了多个渲染目标（multiple render targets，MRT）的想法。不仅可以将像素着色器程序的结果仅发送到颜色和 z 缓冲区，还可以为每个片元生成多组值并将其保存到不同的缓冲区，每个缓冲区称为渲染目标（render target）。渲染目标通常具有相同的 x 和 y 维度； 一些 API 允许使用不同的大小，但是渲染区域将是其中最小的。一些架构要求渲染目标必须具有相同的位深，甚至可能具有相同的数据格式。取决于 GPU，可用的渲染目标数量为四个或八个。

即使有这些限制，MRT 功能还是更有效地执行渲染算法的有力辅助。一次渲染过程可以在一个目标中生成彩色图像，在另一个目标中生成对象标识符，在第三个目标中生成世界空间距离。此功能还引起了另一种类型的渲染管线，称为延迟着色（deferred shading），其中可见性和着色是在单独的通道（passes）中完成的。第一遍存储有关每个像素处对象位置和材质的数据。然后，连续通过可以有效地施加照明和其他效果。此类渲染方法在第 20.1 节中进行了描述。

像素着色器的局限性在于，它通常只能在传递给目标的片元位置上写入渲染目标，而不能从相邻像素读取当前结果。也就是说，执行像素着色器程序时，它无法将其输出直接发送到相邻像素，也无法访问其他人的最新更改。而是，它计算仅影响其自身像素的结果。但是，此限制并不像听起来那样严重。一次通过创建的输出图像可以让像素着色器在后续通过中访问其任何数据。相邻像素可使用第 12.1 节中所述的图像处理技术进行处理。

像素着色器无法知道或影响相邻像素的结果的规则是有例外的。一种是像素着色器可以在计算梯度或导数信息时立即访问相邻片元的信息（尽管是间接的）。像素着色器具有沿 x 和 y 屏幕轴每像素内插值变化的量。这些值可用于各种计算和纹理寻址。这些梯度对于诸如纹理过滤（第 6.2.2 节）之类的操作尤为重要，因为我们想知道多少图像覆盖了一个像素。所有现代 GPU 都通过以 2×2 为一组处理片元（称为四边形）来实现此功能。当像素着色器请求梯度值时，将返回相邻片元之间的差异。参见图 3.15。统一核心具有访问相邻数据（保留在同一 warp 中的不同线程中）的功能，因此可以计算用于像素着色器的渐变。此实现的一个结果是，无法在受动态流控制影响的着色器的部分中访问渐变信息，即，“if” 语句或具有可变迭代次数的循环。一组中的所有片元都必须使用相同的指令集进行处理，以便所有四个像素的结果对于计算梯度都是有意义的。这是一个基本限制，即使在脱机渲染系统中也存在 **[64]**。

DirectX 11 引入了一种缓冲区类型，该类型允许对任何位置（无序访问视图（unordered access view，UAV））的写访问。最初仅适用于像素和计算着色器，对 UAV 的访问已扩展到 DirectX 11.1 中的所有着色器 **[146]**。OpenGL 4.3 将此称为着色器存储缓冲区对象（shader storage buffffer object，SSBO）。这两个名称以其自己的方式进行描述。像素着色器以任意顺序并行运行，并且此存储缓冲区在它们之间共享。

![](<images/1685518897278.png>)

_图 3.15。在左侧，将三角形栅格化为四边形，每组 2 × 2 像素。然后，在右侧显示了带有黑点标记的像素的梯度计算。针对四边形中四个像素位置的每一个，显示了 v 的值。请注意，三角形中没有覆盖三个像素，但是 GPU 仍对其进行处理，以便可以找到渐变。通过使用左下像素的两个四边形邻居，可以计算出 x 和 y 屏幕方向上的渐变。_

通常需要某种机制来避免数据争用情况（也称为数据危险（data hazard）），在这种情况下，两个着色器程序都在 “竞相” 以影响相同的值，从而可能导致任意结果。例如，如果两次调用像素着色器试图在大约同一时间将其添加到相同的检索值中，则可能会发生错误。两者都将检索原始值，都将在本地对其进行修改，但是，无论哪个调用最后写入其结果，都将抹去另一个调用的作用，只会发生一次添加。GPU 通过使用着色器可以访问的专用原子单元（atomic units）来避免此问题 **[530]**。但是，原子意味着某些着色器可能在等待访问另一个着色器进行读 / 修改 / 写操作的存储位置时停滞。

尽管原子避免了数据危害，但许多算法都需要特定的执行顺序。例如，你可能需要绘制一个更远的透明蓝色三角形，然后再用红色透明三角形覆盖它，将红色混合在蓝色上面。一个像素可能对一个像素进行两次像素着色器调用，每个三角形调用一次，以这样一种方式执行：红色三角形的着色器先于蓝色着色器完成。在标准管线中，片段结果将在合并阶段进行排序，然后再进行处理。在 DirectX 11.3 中引入了光栅化程序顺序视图（Rasterizer order views，ROV）以强制执行顺序。这些就像无人机。着色器可以以相同的方式读取和写入它们。关键区别在于 ROV 保证以正确的顺序访问数据。这大大增加了这些着色器可访问缓冲区的有用性 **[327、328]**。例如，ROV 使像素着色器可以编写自己的混合方法，因为它可以直接访问和写入 ROV 中的任何位置，因此不需要合并阶段 **[176]**。代价是，如果检测到乱序访问，像素着色器调用可能会停顿，直到处理了先前绘制的三角形。

**引用：**

**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040  
**[218]** Cabello, Ricardo, et al., Three.js source code, Release r89, Dec. 2017. Cited on p. 41, 50, 115, 189, 201, 407, 485, 552, 628  
**[175]** Blythe, David, “The Direct3D 10 System,” ACM Transactions on Graphics, vol. 25, no. 3, pp. 724–734, July 2006. Cited on p. 29, 39, 42, 47, 48, 50, 249  
**[64]** Apodaca, Anthony A., “How PhotoRealistic RenderMan Works,” in Advanced RenderMan: Creating CGI for Motion Pictures, Morgan Kaufmann, Chapter 6, 1999. Also in SIGGRAPH Advanced RenderMan 2: To RI INFINITY and Beyond course, July 2000. Cited on p. 51  
**[146]** Bilodeau, Bill, “Vertex Shader Tricks: New Ways to Use the Vertex Shader to Improve Performance,” Game Developers Conference, Mar. 2014. Cited on p. 51, 87, 514, 568, 571, 798  
**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040  
**[327]** Davies, Leigh, “OIT to Volumetric Shadow Mapping, 101 Uses for Raster-Ordered Views Using DirectX 12,” Intel Developer Zone blog, Mar. 5, 2015. Cited on p. 52, 139, 156  
**[328]** Davies, Leigh, “Rasterizer Order Views 101: A Primer,” Intel Developer Zone blog, Aug. 5, 2015. Cited on p. 52, 156  
**[176]** Bookout, David, “Programmable Blend with Pixel Shader Ordering,” Intel Developer Zone blog, Oct. 13, 2015. Cited on p. 52

## **3.9 合并阶段 The Merging Stage**

如第 2.5.2 节所述，合并阶段是将各个片段（在像素着色器中生成）的深度和颜色与帧缓冲区组合在一起的阶段。DirectX 将此阶段称为输出合并（output merger）； OpenGL 将其称为逐样本操作（per-sample operations）。在大多数传统管线图（包括我们自己的管线图）上，此阶段是模板缓冲区（stencil-buffer）和 z 缓冲区（z-buffer）操作发生的地方。如果片元可见，则此阶段中发生的另一种操作是颜色混合。对于不透明的表面，不涉及真正的混合，因为片段的颜色会简单地替换以前存储的颜色。片元和所存储颜色的实际混合通常用于透明度和合成操作（第 5.5 节）。

想象一下，通过光栅化生成的片元通过像素着色器运行，然后在应用 z 缓冲区时被某些先前渲染的片元隐藏。这样就不需要在像素着色器中进行所有处理。为了避免这种浪费，许多 GPU 在执行像素着色器之前执行一些合并测试 **[530]**。片元的 z 深度（以及其他正在使用的东西，例如模板缓冲区或剪刀）用于测试可见性。如果隐藏该片元，则将其剔除。此功能称为 Early-z **[1220，1542]**。像素着色器具有更改片元的 z 深度或完全丢弃片元的能力。如果发现像素着色器程序中存在这两种类型的操作，则通常无法使用 Early-Z ，然后通常将其关闭，这通常会使管线效率降低。DirectX 11 和 OpenGL 4.2 允许像素着色器强制进行 Early-Z 测试，尽管有很多限制 **[530]**。有关早期 z 和其他 z 缓冲区优化的更多信息，请参见第 23.7 节。有效使用 Early-z 会对性能产生很大影响，这将在 18.4.5 节中详细讨论。

合并阶段占据了固定功能阶段（例如三角形设置）和完全可编程着色器阶段之间的中间地带。尽管它不是可编程的，但它的操作是高度可配置的。可以将颜色混合设置为执行大量不同的操作。最常见的是涉及颜色和 Alpha 值的乘法，加法和减法的组合，但是其他操作（例如最小值和最大值）以及按位逻辑运算也是可能的。DirectX 10 添加了将像素着色器中的两种颜色与帧缓冲区颜色混合的功能。此功能称为双源颜色混合（dual source-color），不能与多个渲染目标一起使用。MRT 否则支持混合，DirectX 10.1 引入了在每个单独的缓冲区上执行不同混合操作的功能。

如上一节末尾所述，DirectX 11.3 提供了一种通过 ROV 进行混合编程的方法，尽管这是以性能为代价的。ROV 和合并阶段都保证了绘制顺序，也就是输出不变性。不管生成像素着色器结果的顺序如何，API 要求都按照输入结果的顺序（对象，对象和三角形，以及三角形）对结果进行排序并将其发送到合并阶段。

**引用：**

**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040  
**[1220]** Mitchell, Jason L., and Pedro V. Sander, “Applications of Explicit Early-Z Culling,” SIGGRAPH Real-Time Shading course, Aug. 2004. Cited on p. 53, 1016  
**[1542]** Sander, Pedro V., Natalya Tatarchuk, and Jason L. Mitchell, “Explicit Early-Z Culling for Efficient Fluid Flow Simulation,” in Wolfgang Engel, ed., ShaderX5 , Charles River Media, pp. 553–564, 2006. Cited on p. 53, 1016  
**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040

## **3.10 计算着色器 The Compute Shader**

除了实现传统的图形管线外，GPU 还可以用于更多用途。在计算领域，有许多非图形用途，例如计算股票期权的估计价值和训练用于深度学习的神经网络。以这种方式使用硬件称为 GPU 计算。诸如 CUDA 和 OpenCL 之类的平台可作为大型并行处理器来控制 GPU，而无需真正的需求或访问特定于图形的功能。这些框架通常使用带有扩展功能的 C 或 C++ 等语言以及为 GPU 制作的库。

DirectX 11 中引入了计算着色器，它是 GPU 计算的一种形式，因为它是未锁定在图形管线中某个位置的着色器。它与渲染过程紧密相关，因为它由图形 API 调用。它与顶点，像素和其他着色器一起使用。它使用与管道中使用的统一着色器处理器池相同的池。与其他着色器一样，它是着色器，因为它具有一组输入数据，并且可以访问缓冲区（例如纹理）以进行输入和输出。扭曲和线程在计算着色器中更明显。例如，每个调用都会获取一个可以访问的线程索引。还有一个线程组的概念，它由 DirectX 11 中的 1 到 1024 个线程组成。这些线程组由 x，y 和 z 坐标指定，主要是为了简化在着色器代码中的使用。每个线程组都有少量的内存，这些内存在线程之间共享。在 DirectX 11 中，这等于 32 kB。计算着色器由线程组执行，因此保证该组中的所有线程可以同时运行 **[1971]**。

计算着色器的一个重要优点是它们可以访问在 GPU 上生成的数据。从 GPU 向 CPU 发送数据会产生延迟，因此如果可以将处理和结果保留在 GPU 上，则可以提高性能 **[1403]**。在后期处理中，以某种方式修改了渲染的图像，这是计算着色器的常见用法。共享内存意味着来自采样图像像素的中间结果可以与相邻线程共享。例如，已经发现使用计算着色器确定图像的分布或平均亮度的运行速度是在像素着色器上执行此操作的两倍 **[530]**。

计算着色器还可用于粒子系统，网格处理（例如面部动画 **[134]**，剔除 **[1883、1884]**，图像过滤 **[1102、1710]**，提高深度精度 **[991]**，阴影 **[865]**，景深 **[764]**，以及可以承担一组 GPU 处理器的任何其他任务。Wihlidal **[1884]** 讨论了计算着色器如何比曲面细分外壳着色器更有效。其他用途请参见图 3.16。

至此，我们对 GPU 的渲染管线实现的审查结束了。有多种方法可以使用和组合 GPU 功能来执行各种与渲染相关的过程。调整以利用这些功能的相关理论和算法是本书的重点。现在，我们将重点放在变换（transforms）和着色（shading）上。

![](<images/1685518897522.png>)

_图 3.16。计算着色器示例。左侧是计算着色器，用于模拟受风影响的头发，并使用细分阶段渲染头发本身。在中间，计算着色器执行快速模糊操作。在右侧，模拟了海浪。（图像来自 NVIDIA SDK 11 **[1301]** 示例，由 NVIDIA Corporation 提供。）_

**引用：**

**[1971]** Zink, Jason, Matt Pettineo, and Jack Hoxley, Practical Rendering & Computation with Direct3D 11, CRC Press, 2011. Cited on p. 47, 54, 90, 518, 519, 520, 568, 795, 813, 814, 914  
**[1403]** Pettineo, Matt, “A Sampling of Shadow Techniques,” The Danger Zone blog, Sept. 10, 2013. Cited on p. 54, 238, 245, 250, 265  
**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040  
**[134]**  Bentley, Adrian, “inFAMOUS Second Son Engine Postmortem,” Game Developers Conference, Mar. 2014. Cited on p. 54, 490, 871, 884, 904  
**[1883]** Wihlidal, Graham, “Optimizing the Graphics Pipeline with Compute,” Game DevelopersConference, Mar. 2016. Cited on p. 54, 798, 834, 837, 840, 848, 849, 851, 908, 986  
**[1884]** Wihlidal, Graham, “Optimizing the Graphics Pipeline with Compute,” in Wolfgang Engel, ed., GPU Zen, Black Cat Publishing, pp. 277–320, 2017. Cited on p. 54, 702, 784, 798, 812, 834, 837, 840, 848, 850, 851, 908, 986  
**[1102]** Mah, Layla, and Stephan Hodes, “DirectCompute for Gaming: Supercharge Your Engine with Compute Shaders,” Game Developers Conference, Mar. 2013. Cited on p. 54, 518, 535  
**[1710]** Story, Jon, “DirectCompute Accelerated Separable Filtering,” Game Developers Conference, Mar. 2011. Cited on p. 54, 518  
**[991]** Lauritzen, Andrew, Marco Salvi, and Aaron Lefohn, “Sample Distribution Shadow Maps,” in Symposium on Interactive 3D Graphics and Games, ACM, pp. 97–102, Feb. 2011. Cited on p. 54, 101, 244, 245  
**[865]** Kasyan, Nikolas, “Playing with Real-Time Shadows,” SIGGRAPH Efficient Real-Time Shadows course, July 2013. Cited on p. 54, 234, 245, 251, 264, 585  
**[764]** Hoobler, Nathan, “High Performance Post-[Processing](https://so.csdn.net/so/search?q=Processing&spm=1001.2101.3001.7020),” Game Developers Conference, Mar. 2011. Cited on p. 54, 536  
**[1884]** Wihlidal, Graham, “Optimizing the Graphics Pipeline with Compute,” in Wolfgang Engel, ed., GPU Zen, Black Cat Publishing, pp. 277–320, 2017. Cited on p. 54, 702, 784, 798, 812, 834, 837, 840, 848, 850, 851, 908, 986  
**[1301]** NVIDIA SDK 11, https://developer.nvidia.com/dx11-samples. Cited on p. 46, 55, 150

##  **进一步阅读和资源**

吉森（Giesen）的图形管道之旅 **[530]** 详细讨论了 GPU 的许多方面，并解释了元素为何按其方式工作。Fatahalian 和 Bryant 的课程 **[462]** 在一系列详细的讲义幻灯片集中讨论了 GPU 并行性。尽管着眼于使用 CUDA 进行 GPU 计算，但 Kirk 和 Hwa 的书 **[903]** 的介绍部分讨论了 GPU 的发展和设计理念。

要学习着色器编程的形式方面，需要花费一些工作。诸如 OpenGL Superbible **[1606]** 和 OpenGL Programming Guide **[885]** 之类的书籍都包含有关着色器编程的材料。旧书 OpenGL Shading Language **[1512]** 没有涵盖较新的着色器阶段，例如几何和细分着色器，但确实专注于与着色器相关的算法。有关最新和推荐的图书，请参见本书的网站 realtimerendering.com。

**引用：**

**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040  
**[462]** Fatahalian, Kayvon, and Randy Bryant, Parallel Computer Architecture and Programming course, Carnegie Mellon University, Spring 2017. Cited on p. 30, 55  
**[903]** Kirk, David B., and Wen-mei W. Hwu, Programming Massively Parallel Processors: A Handson Approach, Third Edition, Morgan Kaufmann, 2016. Cited on p. 55, 1040  
**[1606]** Sellers, Graham, Richard S. Wright Jr., and Nicholas Haemel, OpenGL Superbible: Comprehensive Tutorial and Reference, Seventh Edition, Addison-Wesley, 2015. Cited on p. 55  
**[885]** Kessenich, John, Graham Sellers, and Dave Shreiner, OpenGL Programming Guide: The Of- ficial Guide to Learning OpenGL, Version 4.5 with SPIR-V, Ninth Edition, Addison-Wesley, 2016. Cited on p. 27, 39, 41, 55, 96, 173, 174  
**[1512]**  Rost, Randi J., Bill Licea-Kane, Dan Ginsburg, John Kessenich, Barthold Lichtenbelt, Hugh Malan, and Mike Weiblen, OpenGL Shading Language, Third Edition, Addison-Wesley, 2009. Cited on p. 55, 200