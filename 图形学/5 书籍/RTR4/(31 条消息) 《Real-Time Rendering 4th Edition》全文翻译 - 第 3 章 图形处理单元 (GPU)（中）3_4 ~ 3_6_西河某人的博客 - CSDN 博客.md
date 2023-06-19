不闹情绪了…… 好好学习，拯救世界……

## **3.4 可编程着色和 API 的发展 The Evolution of Programmable Shading and APIs**

可编程着色框架的构想可以追溯到 1984 年，当时库克（Cook）的着色树（shade trees）**[287]**。一个简单的着色器及其相应的着色树如图 3.4 所示。RenderMan 着色语言（The RenderMan Shading Language）**[63，1804]** 是在 1980 年代后期从这个想法发展而来的。如今，它与其他不断发展的规范（例如，开放着色语言（OSL）项目 **[608]**）一起用于电影制作渲染。

![](<images/1685518884809.png>)

_图 3.4。一个简单的铜着色器的着色树及其相应的着色器语言程序。（在库克 **[287]** 之后。）_

消费者级图形硬件最早是 3dfx Interactive 于 1996 年 10 月 1 日成功推出的。今年的时间表请参见图 3.5。他们的 Voodoo 图形卡能够以高品质和高性能来渲染《Quake》游戏，因此很快就被采用。该硬件始终实现了固定功能的流水线。在 GPU 原生支持可编程着色器之前，曾有各种尝试通过多次渲染实时实现可编程着色操作。Quake III：Arena 脚本语言是 1999 年在该领域的首个广泛的商业成功案例。如本章开头所述，NVIDIA 的 GeForce256 是第一个被称为 GPU 的硬件，它不是可编程的（not programmable），但是它是可配置的（configurable）。

![](<images/1685518884833.png>)

_图 3.5。一些 API 和图形硬件版本的时间表。_

在 2001 年初，NVIDIA 的 GeForce 3 是第一个支持可编程顶点着色器 **[1049]** 的 GPU，该着色器通过 DirectX 8.0 和 OpenGL 扩展公开。这些着色器以一种类似于汇编的语言进行编程，该语言被驱动程序即时转换为微代码。像素着色器也包含在 DirectX 8.0 中，但是像素着色器没有达到实际的可编程性——驱动程序将受支持的有限 “程序” 转换为纹理混合状态，然后将其连接到硬件“寄存器组合器”（register combiners）。这些 “程序” 不仅限于长度（不超过 12 条指令），而且缺少重要的功能。通过对 RenderMan 的研究，Peercy 等人确定了相关的纹理读取和浮点数据 **[1363]** 对真正的可编程性是至关重要的，。

着色器此时不允许进行流控制（分支，branching），因此必须通过计算两个项以及在结果之间进行选择或内插（interpolating）来模拟条件选择。DirectX 定义了着色器模型（Shader Model，SM）的概念，以区分具有不同着色器功能的硬件。2002 年，包括 Shader Model 2.0 在内的 DirectX 9.0 发行了，该版本具有真正可编程的顶点和像素着色器。在 OpenGL 下，使用各种扩展功能也公开了类似的功能。添加了对任意依赖的纹理读取的支持以及对 16 位浮点值的存储的支持，最终完成了 Peercy 等人确定的一组要求。诸如指令，纹理和寄存器之类的着色器资源的限制增加了，因此着色器变得能够产生更复杂的效果。还增加了对流控制（flow control）的支持。着色器的长度和复杂性不断增长，使得汇编编程模型变得越来越麻烦。幸运的是，DirectX 9.0 还包含 HLSL 。此着色语言是由 Microsoft 与 NVIDIA 合作开发的。大约在同一时间，OpenGL ARB（架构审查委员会）发布了 GLSL，一种与 OpenGL 非常相似的语言 **[885]**。这些语言在很大程度上受到 C 编程语言的语法和设计理念的影响，其中包括来自 RenderMan 着色语言的元素。

Shader Model 3.0 于 2004 年推出，并添加了动态流控制（Dynamic Flow Control），使着色器功能更加强大。它还将可选功能转变为需求，进一步增加了资源限制，并增加了对顶点着色器中纹理读取的有限支持。当在 2005 年下半年（Microsoft 的 Xbox 360）和 2006 年下半年（Sony Computer Entertainment 的 PLAYSTATION 3 系统）推出新一代游戏机时，它们配备了 Shader Model 3.0 级 GPU。任天堂的 Wii console 是最后一批著名的固定功能 GPU 之一，该 GPU 最初于 2006 年底交付。纯固定功能产品线在这一点上早已消失了。着色器语言已经发展到可以使用各种工具来创建和管理它们的地步。图 3.6 显示了使用库克（Cook）的着色树（Shade Tree）概念的一种此类工具的屏幕截图。

可编程性的下一个重大进步也是在 2006 年底左右。DirectX 10.0 **[175]** 中包含的 Shader Model 4.0 引入了几个主要功能，例如几何体着色器（geometry shader）和流输出（stream output）。Shader Model 4.0 包括适用于所有着色器（顶点，像素和几何图形）的统一编程模型，这是先前描述的统一着色器设计。资源限制进一步增加，并增加了对整数数据类型（包括按位运算）的支持。OpenGL 3.3 中 GLSL 3.30 的引入提供了类似的着色器模型。

![](<images/1685518884874.png>)

_图 3.6。用于着色器设计的可视着色器图形系统。各种操作封装在功能框中，可在左侧选择。选中后，每个功能框都有可调参数，如右图所示。每个功能框的输入和输出相互链接以形成最终结果，如中间框架的右下方所示。（摘自 “心理磨坊”，mental images inc。）_

2009 年发布了 DirectX 11 和 Shader Model 5.0，添加了细分阶段着色器（tessellation stage）和计算着色器（compute shader），也称为 DirectCompute。该版本还专注于更有效地支持 CPU 多处理，这是第 18.5 节中讨论的主题。OpenGL 在 4.0 版中添加了细分，在 4.3 版中添加了计算着色器。DirectX 和 OpenGL 的发展不同。两者都设置了特定版本发行所需的一定级别的硬件支持。Microsoft 控制 DirectX API，因此直接与独立硬件供应商（IHV）（例如 AMD，NVIDIA 和 Intel）以及游戏开发商和计算机辅助设计软件公司合作，以确定要公开的功能。OpenGL 由非营利组织 Khronos Group 管理的硬件和软件供应商联盟开发。由于涉及的公司数量众多，API 功能通常在 DirectX 中引入 OpenGL 之后的一段时间内就会出现。但是，OpenGL 允许特定于供应商的或更广泛的扩展（extensions），这些扩展允许在发行版正式支持之前使用最新的 GPU 功能。

API 的下一个重大变化是由 AMD 在 2013 年推出了 MantleAPI。Mantle 与视频游戏开发商 DICE 合作开发的，其目的是消除大部分图形驱动程序的开销，并将此控件直接交给开发人员。除了这种重构之外，还进一步支持有效的 CPU 多处理。这类新的 API 专注于大大减少 CPU 在驱动程序中花费的时间，以及更有效的 CPU 多处理器支持（第 18 章）。在 Mantle 中开创的创意被 Microsoft 采纳，并在 2015 年以 DirectX 12 的形式发布。请注意，DirectX 12 并不专注于公开新的 GPU 功能 - DirectX 11.3 公开了相同的硬件功能。这两个 API 均可用于将图形发送到虚拟现实系统，例如 Oculus Rift 和 HTC Vive。但是，DirectX 12 是对 API 的彻底重新设计，可以更好地映射到现代 GPU 架构。低开销的驱动程序对于以下应用程序很有用：CPU 驱动程序成本引起瓶颈，或者使用更多 CPU 处理器进行图形处理可能会提高性能 **[946]**。从较早的 API 移植可能很困难，并且天真的实现可能会导致性能降低 **[249、699、1438]**。

苹果于 2014 年发布了自己的低开销 API（称为 Metal）。Metal 首次在 iPhone 5S 和 iPad Air 等移动设备上可用，一年后，可通过 OS X El Capitan 访问较新的 Macin 代码。除效率外，减少 CPU 使用率还可以节省功耗，这是移动设备上的重要因素。该 API 具有自己的着色语言，适用于图形和 GPU 计算程序。

AMD 将其 Mantle 工作捐赠给了 Khronos Group，后者于 2016 年初发布了自己的新 API，名为 Vulkan。与 OpenGL 一样，Vulkan 可在多个操作系统上工作。Vulkan 使用一种称为 SPIR V 的新的高级中间语言，该语言既用于着色器表示又用于常规 GPU 计算。预编译的着色器是可移植的，因此可以在支持所需功能的任何 GPU 上使用 **[885]**。Vulkan 也可以用于非图形 GPU 计算，因为它不需要显示窗口 **[946]**。Vulkan 与其他低开销驱动程序的显着区别是，它旨在与多种系统一起使用，从工作站到移动设备。

在移动设备上，规范是使用 OpenGL ES。“ES” 代表嵌入式系统（Embedded Systems），因为此 API 是为移动设备而开发的。当时的标准 OpenGL 在其某些调用结构中相当庞大且缓慢，并且需要支持很少使用的功能。OpenGL ES 1.0 于 2003 年发布，是 OpenGL 1.3 的简化版本，描述了固定功能的管线。虽然 DirectX 的发布与支持它们的图形硬件的发布是同步的，但是开发针对移动设备的图形支持的方式却并不相同。例如，2010 年发布的第一台 iPad 实施了 OpenGL ES 1.1。OpenGL ES 2.0 规范于 2007 年发布，提供了可编程的着色（programmable shading）。它基于 OpenGL 2.0，但没有固定功能组件，因此与 OpenGL ES 1.1 不向后兼容。OpenGL ES 3.0 于 2012 年发布，提供了多个渲染目标，纹理压缩，变换反馈，实例化以及更广泛的纹理格式和模式等功能，并改进了着色器语言。OpenGL ES 3.1 添加了计算着色器，而 3.2 添加了几何和曲面细分着色器，以及其他功能。第 23 章将更详细地讨论移动设备架构。

OpenGL ES 的一个分支是基于浏览器的 API WebGL，可通过 JavaScript 调用。该 API 的第一版发布于 2011 年，可在大多数移动设备上使用，因为它的功能等效于 OpenGL ES 2.0。与 OpenGL 一样，扩展允许访问更高级的 GPU 功能。WebGL 2 假定支持 OpenGL ES 3.0。

WebGL 特别适合在教室中试用功能或使用：  
• 它是跨平台的，可在所有个人计算机和几乎所有移动设备上使用。  
• 驱动程序批准由浏览器处理。即使一个浏览器不支持特定的 GPU 或扩展，通常另一个浏览器也支持。  
• 代码被解释而不是编译，并且仅需要文本编辑器即可进行开发。  
• 大多数浏览器都内置了调试器，可以检查在任何网站上运行的代码。  
• 可以通过将程序上传到网站或 Github 来进行部署。

更高级别的场景图形和效果库（例如 three.js **[218]**）使您可以轻松访问代码，以获取各种更复杂的效果，例如着色算法（shadow algorithms），后处理效果（post-[processing](https://so.csdn.net/so/search?q=processing&spm=1001.2101.3001.7020) effects），基于物理的着色（physically based shading）和延迟渲染（deferred rendering）。

**引用：**

**[287]** Cook, Robert L., “Shade Trees,” Computer Graphics (SIGGRAPH ’84 Proceedings), vol. 18, no. 3, pp. 223–231, July 1984. Cited on p. 37, 765  
**[63]** Apodaca, Anthony A., and Larry Gritz, Advanced RenderMan: Creating CGI for Motion Pictures, Morgan Kaufmann, 1999. Cited on p. 37, 909  
**[1804]** Upstill, S., The RenderMan Companion: A Programmer’s Guide to Realistic Computer Graphics, Addison-Wesley, 1990. Cited on p. 37  
**[608]** Gritz, Larry, ed., “Open Shading Language 1.9: Language Specification,” Sony Pictures Imageworks Inc., 2017. Cited on p. 37  
**[1049]** Lindholm, Erik, Mark Kilgard, and Henry Moreton, “A User-Programmable Vertex Engine,” in SIGGRAPH ’01 Proceedings of the 28th Annual Conference on Computer Graphics and Interactive Techniques, ACM, pp. 149–158, Aug. 2001. Cited on p. 15, 38  
**[1363]** Peercy, Mark S., Marc Olano, John Airey, and P. Jeffrey Ungar, “Interactive Multi-Pass Programmable Shading,” in SIGGRAPH ’00: Proceedings of the 27th Annual Conference on Computer Graphics and Interactive Techniques, ACM Press/Addison-Wesley Publishing Co., pp. 425–432, July 2000. Cited on p. 38  
**[885]** Kessenich, John, Graham Sellers, and Dave Shreiner, OpenGL Programming Guide: The Of-ficial Guide to Learning OpenGL, Version 4.5 with SPIR-V, Ninth Edition, Addison-Wesley, 2016. Cited on p. 27, 39, 41, 55, 96, 173, 174  
**[175]** Blythe, David, “The Direct3D 10 System,” ACM Transactions on Graphics, vol. 25, no. 3, pp. 724–734, July 2006. Cited on p. 29, 39, 42, 47, 48, 50, 249  
**[946]** Kubisch, Christoph, “Transitioning from OpenGL to Vulkan,” NVIDIA GameWorks blog, Feb. 11, 2016. Cited on p. 40, 41, 796, 814  
**[249]** Chajdas, Matth¨aus G., “D3D12 and Vulkan: Lessons Learned,” Game Developers Conference, Mar. 2016. Cited on p. 40, 806, 814  
**[699]** Hector, Tobias, “Vulkan: High Efficiency on Mobile,” Imagination Blog, Nov. 5, 2015. Cited on p. 40, 794, 814  
**[1438]** Pranckeviˇcius, Aras, “Porting Unity to New APIs,” SIGGRAPH An Overview of Next Generation APIs course, Aug. 2015. Cited on p. 40, 806, 814  
**[885]** Kessenich, John, Graham Sellers, and Dave Shreiner, OpenGL Programming Guide: The Of-ficial Guide to Learning OpenGL, Version 4.5 with SPIR-V, Ninth Edition, Addison-Wesley, 2016. Cited on p. 27, 39, 41, 55, 96, 173, 174  
**[946]** Kubisch, Christoph, “Transitioning from OpenGL to Vulkan,” NVIDIA GameWorks blog, Feb. 11, 2016. Cited on p. 40, 41, 796, 814  
**[218]** Cabello, Ricardo, et al., Three.js source code, Release r89, Dec. 2017. Cited on p. 41, 50, 115, 189, 201, 407, 485, 552, 628  
**[175]** Blythe, David, “The Direct3D 10 System,” ACM Transactions on Graphics, vol. 25, no. 3, pp. 724–734, July 2006. Cited on p. 29, 39, 42, 47, 48, 50, 249  
**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040  
**[1208]** Microsoft, “Direct3D 11 Graphics,” Windows Dev Center. Cited on p. 42, 233, 525

## **3.5 顶点着色器 The Vertex Shader**

顶点着色器是图 3.2 所示功能管线中的第一阶段。虽然这是直接在程序员控制下的第一阶段，但值得注意的是，一些数据操作在此阶段之前发生。在 DirectX 所谓的输入汇编器 **[175、530、1208]** 中，可以将几个数据流编织在一起，以形成沿管线发送的一组顶点和图元。例如，一个对象可以由一个位置阵列和一个颜色阵列表示。输入汇编器将通过创建具有位置和颜色的顶点来创建此对象的三角形（或直线或点）。第二个对象可以使用相同的位置数组（以及不同的模型转换矩阵）和不同的颜色数组表示。数据表示将在 16.4.5 节中详细讨论。输入汇编器中也支持执行实例化。这允许一个对象被绘制多次，每个实例具有一些不同的数据，所有这些都可以通过一个绘制调用进行。第 18.4.2 节介绍了实例化的使用。

三角形网格由一组顶点表示，每个顶点与模型表面上的特定位置相关联。除了位置之外，每个顶点还有其他可选属性，例如颜色或纹理坐标。曲面法线也定义在网格顶点上，这似乎是一个奇怪的选择。从数学上讲，每个三角形都有一个定义明确的表面法线，直接将三角形的法线用于着色似乎更有意义。但是，渲染时，通常使用三角形网格来表示基础曲面，而使用顶点法线来表示该表面的方向，而不是三角形网格本身的方向。16.3.4 节将讨论计算顶点法线的方法。图 3.7 显示了两个三角形网格的侧视图，这些三角形网格代表曲面，一个是平滑的，另一个是带有锐利折痕的三角形。

![](<images/1685518884921.png>)

_图 3.7。三角形曲面（黑色，具有顶点法线）的侧视图，代表曲面（红色）。在左侧，平滑的顶点法线用于表示平滑表面。在右侧，中间顶点已被复制并指定了两个法线，表示折痕。_

顶点着色器是处理三角形网格的第一阶段。描述顶点形成什么的数据对于顶点着色器是不可用的。顾名思义，它专门处理传入的顶点。顶点着色器提供了一种修改、创建或忽略与每个三角形的顶点关联的值的方法，例如其颜色、法线、纹理坐标和位置。通常，顶点着色器程序会将顶点从模型空间转换为齐次裁剪空间（homogeneous clip space）（第 4.7 节）。顶点着色器至少必须始终输出此位置。

顶点着色器与前面描述的统一着色器（unifified shader）几乎相同。传入的每个顶点都由顶点着色器程序处理，该程序然后输出在三角形或直线上内插的多个值。顶点着色器既不能创建也不能破坏顶点，并且一个顶点生成的结果不能传递到另一个顶点。由于每个顶点都是独立处理的，因此可以将 GPU 上任意数量的着色器处理器并行应用于传入的顶点流。

输入汇编通常表示为在执行顶点着色器之前发生的过程。这是一个物理模型通常与逻辑模型不同的例子。从物理上讲，获取数据以创建顶点的操作可能发生在顶点着色器中，并且驱动程序将悄悄地为每个着色器添加适当的指令，这些指令对于程序员是不可见的。

接下来的章节介绍了几种顶点着色器效果，例如用于动画关节的顶点混合和轮廓渲染。顶点着色器的其他用途包括：

• 对象生成，通过仅创建一次网格并使其由顶点着色器变形即可。  
• 使用蒙皮和变形技术对角色的身体和面部进行动画处理。  
• 程序变形，例如旗帜，布料或水的移动 **[802、943]**。  
• 通过发送退化的（degenerate）（无区域）网格沿管线生成粒子，并根据需要为其分配区域。  
• 通过将整个帧缓冲区的内容用作屏幕对齐的网格上的纹理，镜头变形，热雾，水波纹，页面卷曲和其他效果会发生程序变形。  
• 通过使用顶点纹理获取 **[40，1227]** 应用地形高度场（terrain height fields）。

使用顶点着色器完成的一些变形如图 3.8 所示。

顶点着色器的输出可以通过几种不同的方式使用。然后为每个实例的图元（例如三角形）生成常用路径，并对其进行光栅化，生成的各个像素片元将发送到像素着色器程序以进行继续处理。在某些 GPU 上，数据也可以发送到细分阶段或几何着色器，或存储在内存中。以下各节将讨论这些可选阶段。

![](<images/1685518884945.png>)

_图 3.8。左边是一个普通的茶壶。由顶点着色器程序执行的简单剪切操作将生成中间图像。在右侧，噪声函数会创建一个使模型失真的字段。（图像由 FX Composer 2 制作，由 NVIDIA Corporation 提供。）_

**引用：**

**[802]** Isidoro, John, Alex Vlachos, and Chris Brennan, “Rendering Ocean Water,” in Wolfgang Engel, ed., Direct3D ShaderX: Vertex & Pixel Shader Tips and Techniques, Wordware, pp. 347– 356, May 2002. Cited on p. 43  
**[943]** Kryachko, Yuri, “Using Vertex Texture Displacement for Realistic Water Rendering,” in Matt Pharr, ed., GPU Gems 2, Addison-Wesley, pp. 283–294, 2005. Cited on p. 43  
**[40]** Andersson, Johan, “Terrain Rendering in Frostbite Using Procedural Shader Splatting,” SIGGRAPH Advanced Real-Time Rendering in 3D Graphics and Games course, Aug. 2007. Cited on p. 43, 175, 218, 877, 878  
**[1227]** Mittring, Martin, “Finding Next Gen—CryEngine 2,” SIGGRAPH Advanced Real-Time Rendering in 3D Graphics and Games course, Aug. 2007. Cited on p. 43, 195, 239, 242, 255, 457, 476, 559, 856, 860, 861

## **3.6 曲面细分阶段 The Tessellation Stage**

细分阶段允许我们渲染曲面。GPU 的任务是获取每个表面描述，并将其变成一组代表性的三角形。此阶段是可选的 GPU 功能，该功能首先在 DirectX 11 中可用（并且是 DirectX 11 所必需的）。OpenGL4.0 和 OpenGL ES 3.2 也支持该功能。

使用细分阶段有几个优点。曲面描述通常比提供相应的三角形本身更紧凑。除了节省内存外，此功能还可以防止 CPU 和 GPU 之间的总线成为形状变化的动画角色或对象的瓶颈。通过为给定视图生成适当数量的三角形，可以有效地渲染表面。例如，如果一个球远离相机，则仅需要几个三角形。近距离观察时，最好用数千个三角形来表示。这种控制细节水平的能力还可以使应用程序控制其性能，例如，在较弱的 GPU 上使用较低质量的网格以保持帧速率。通常用平坦表面表示的模型可以转换为三角形的细网格，然后根据需要进行变形 **[1493]**，或者可以对其进行细分，以便更不频繁地执行昂贵的着色计算 **[225]**。

细分阶段始终由三个元素组成。使用 DirectX 的术语，它们是外壳着色器（hull shader），细分（tessellator）和域着色器（domain shader）。在 OpenGL 中，外壳着色器是曲面细分控制着色器（the tessellation control shader），而域着色器是曲面细分评估着色器（tessellation evaluation shader），虽然详细，但更具描述性。固定功能细分器（fixed-function tessellator）在 OpenGL 中称为原始生成器（primitive generator），并且可以看到，确实是它的功能。

在第 17 章中详细讨论了如何指定和细分曲面和曲线。在这里，我们简要概述了每个细分阶段的目的。首先，外壳着色器的输入是一个特殊的补丁图元（patch primitive）。它由几个控制点组成，这些控制点定义了细分曲面，B'ezier 面片或其他类型的弯曲元素。外壳着色器具有两个功能。首先，它告诉细分器应生成多少个三角形以及采用哪种配置。其次，它对每个控制点执行处理。同样，可选地，外壳着色器可以修改传入的补丁描述，根据需要添加或删除控制点。外壳着色器将其控制点集以及细分控制数据放到域着色器中。参见图 3.9。

![](<images/1685518884991.png>)

_图 3.9。细分阶段。外壳着色器采用由控制点定义的补丁。它将细分因子（tessellation factors, TFs）和类型发送给固定功能细分器。控制点集由外壳着色器根据需要进行转换，并与 TF 和相关的修补程序常量一起发送到域着色器。曲面细分对象将创建一组顶点及其重心坐标。然后由域着色器对其进行处理，从而生成三角形网格（显示控制点以供参考）。_

曲面细分是管线中的固定功能阶段，仅与曲面细分着色器一起使用。它的任务是为域着色器添加多个新顶点以进行处理。外壳着色器向细分器发送有关所需细分曲面类型的信息：三角形，四边形或等值线（isoline）。等值线是线带（line strips）的集合，有时用于毛发渲染 **[1954]**。外壳着色器发送的其他重要值是细分因子（OpenGL 中的细分级别，tessellation levels）。它们有两种类型：内边缘和外边缘。这两个内部因素决定了三角形或四边形内部发生了多少细分。外部因素决定每个外部边缘被分割多少（第 17.6 节）。图 3.10 显示了增加细分因子的示例。通过允许使用单独的控件，我们可以使相邻曲面的边缘在细分中匹配，而无论内部如何细分。匹配的边缘可避免在补丁相遇之处出现裂缝或其他着色瑕疵。顶点被分配了重心坐标（barycentric coordinates）（第 22.8 节），这些值指定了所需表面上每个点的相对位置。

外壳着色器始终输出补丁，一组控制点位置。但是，它可以通过向细分器发送零或更低（或非数字，NaN）的外部细分级别（outer tessellation level）来发出信号，表示将要丢弃补丁。否则，细分器将生成网格并将其发送到域着色器。域着色器的每次调用都使用来自外壳着色器的曲面的控制点，以计算每个顶点的输出值。域着色器具有类似于顶点着色器的数据流模式，来自细分细分器的每个输入顶点都经过处理并生成相应的输出顶点。然后将形成的三角形沿管道向下传递。

尽管此系统听起来很复杂，但我们是为提高效率而采用这种结构的，并且这样的话，每个着色器可能都非常简单。传递到外壳着色器中的补丁程序通常很少或根本不做修改。该着色器还可以使用补丁的估计距离或屏幕大小来动态计算细分因子，就像地形渲染一样 **[466]**。或者，外壳着色器可以简单地为应用程序计算和提供的所有补丁程序传递一组固定的值。细分器执行一个涉及但固定功能的过程，生成顶点，为其指定位置并指定它们形成的三角形或直线。此数据放大步骤是在着色器外部执行的，以提高计算效率 **[530]**。域着色器采用为每个点生成的重心坐标，并在补丁的评估方程式中使用这些坐标，以生成位置，法线，纹理坐标以及所需的其他顶点信息。有关示例，请参见图 3.11。

![](<images/1685518885029.png>)

_图 3.11。左边是大约 6000 个三角形的基础网格。在右侧，使用 PN 三角形细分 来细分每个三角形并进行置换。（图像由 NVIDIA 公司提供，来自 NVIDIA SDK 11 **[1301]** 的示例，由 4A Games 提供的 Metro 2033 型号。）_

![](<images/1685518885071.png>)

_图 3.12。几何着色器程序的几何着色器输入为某些单一类型：点，线段，三角形。最右边的两个图元包括与直线和三角形对象相邻的顶点。另外，更精细的补丁类型是可做到的。_

**引用：**

**[225]** Cantlay, Iain, and Andrei Tatarinov, “From Terrain to Godrays: Better Use of DX11,” Game Developers Conference, Mar. 2014. Cited on p. 44, 569  
**[1954]** Yuksel, Cem, and Sara Tariq, SIGGRAPH Advanced Techniques in Real-Time Hair Rendering and Simulation course, July 2010. Cited on p. 45, 642, 646, 649  
**[466]** Fernandes, Ant´onio Ramires, and Bruno Oliveira, “GPU Tessellation: We Still Have a LOD of Terrain to Cover,” in Patrick Cozzi & Christophe Riccio, eds., OpenGL Insights, CRC Press, pp. 145–161, 2012. Cited on p. 46, 879  
**[530]** Giesen, Fabian, “A Trip through the Graphics Pipeline 2011,” The ryg blog, July 9, 2011. Cited on p. 32, 42, 46, 47, 48, 49, 52, 53, 54, 55, 141, 247, 684, 701, 784, 1040  
**[1301]** NVIDIA SDK 11, https://developer.nvidia.com/dx11-samples. Cited on p. 46, 55, 150