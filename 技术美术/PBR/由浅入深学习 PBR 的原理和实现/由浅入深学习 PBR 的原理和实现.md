目录

*   [**一. 前言**](#一-前言)
    *   [**1.1 本文动机**](#11-本文动机)
    *   [**1.2 PBR 知识体系**](#12-pbr知识体系)
    *   [**1.3 本文内容及特点**](#13-本文内容及特点)
*   [**二. 初阶：PBR 基本认知和应用**](#二-初阶pbr基本认知和应用)
    *   [**2.1 PBR 的基本介绍**](#21-pbr的基本介绍)
        *   [**2.1.1 PBR 概念**](#211-pbr概念)
        *   [**2.1.2 与物理渲染的差别**](#212-与物理渲染的差别)
        *   [**2.1.3 PBR 的特征**](#213-pbr的特征)
    *   [**2.2 PBR 的衍变历史**](#22-pbr的衍变历史)
        *   [**2.2.1 Lambert（1760 年）**](#221-lambert1760年)
        *   [**2.2.2 Smith（1967 年）**](#222-smith1967年)
        *   [**2.2.3 Phong（1973 年）**](#223-phong1973年)
        *   [**2.2.4 Cook-Torrance（1982 年）**](#224-cook-torrance1982年)
        *   [**2.2.5 Oren Nayarh（1994 年）**](#225-oren-nayarh1994年)
        *   [**2.2.6 Schlick（1994 年）**](#226-schlick1994年)
        *   [**2.2.7 GGX（2007 年）**](#227-ggx2007年)
        *   [**2.2.8 迪斯尼原则的 BRDF（Disney principled BRDF, 2012 年)**](#228-迪斯尼原则的brdfdisney-principled-brdf-2012年)
        *   [**2.2.9 现阶段的 BxDF（2019 年）**](#229-现阶段的bxdf2019年)
    *   [**2.3 PBR 的应用领域**](#23-pbr的应用领域)
    *   [**2.4 PBR 在游戏引擎的应用**](#24-pbr在游戏引擎的应用)
        *   [**2.4.1 Unreal Engine 4 的 PBR**](#241-unreal-engine-4的pbr)
        *   [**2.4.2 Unity 的 PBR**](#242-unity的pbr)
*   [**三. 中阶：PBR 基本原理和实现**](#三-中阶pbr基本原理和实现)
    *   [**3.1 PBR 基础理论和推导**](#31-pbr基础理论和推导)
        *   [**3.1.1 微平面（Microfacet）**](#311-微平面microfacet)
        *   [**3.1.2 能量守恒（Energy Conservation）**](#312-能量守恒energy-conservation)
        *   [**3.1.3 反射方程（Reflectance Equation）**](#313-反射方程reflectance-equation)
        *   [**3.1.4 双向反射分布函数（BRDF）**](#314-双向反射分布函数brdf)
            *   [**3.1.4.1 $D$(Normal _D_istribution Function，NDF)**](#3141-normal-distribution-functionndf)
            *   [**3.1.4.2 $F$(_F_resnel equation)**](#3142-fresnel-equation)
            *   [**3.1.4.3 $G$(_G_eometry function)**](#3143-geometry-function)
            *   [**3.1.4.4 Cook-Torrance 反射方程 (Cook-Torrance reflectance equation)**](#3144-cook-torrance反射方程cook-torrance-reflectance-equation)
        *   [**3.1.5 制作 PBR 材质**](#315-制作pbr材质)
    *   [**3.2 PBR 的光照实现**](#32-pbr的光照实现)
        *   [**3.2.1 辐照度计算**](#321-辐照度计算)
        *   [**3.2.2 PBR 表面模型（ PBR surface model）**](#322-pbr表面模型-pbr-surface-model)
            *   [**3.2.2.1 直接光照（Direct lighting）**](#3221-直接光照direct-lighting)
            *   [**3.2.2.2 线性和 HDR 渲染（ Linear and HDR rendering）**](#3222-线性和hdr渲染-linear-and-hdr-rendering)
            *   [**3.2.2.3 完整的 PBR 直接光照着色器**](#3223-完整的pbr直接光照着色器)
        *   [**3.2.3 使用纹理的 PBR（Textured PBR）**](#323-使用纹理的pbrtextured-pbr)
    *   [**3.3 基于图像的光照（Image Based Lighting，IBL）**](#33-基于图像的光照image-based-lightingibl)
        *   [**3.3.1 漫反射辐照度（Diffuse irradiance）**](#331-漫反射辐照度diffuse-irradiance)
            *   [**3.3.1.1 球体图（Equirectangular map）**](#3311-球体图equirectangular-map)
            *   [**3.3.1.2 从球体图到立方体图**](#3312-从球体图到立方体图)
            *   [**3.3.1.3 PBR 和非直接辐射度光照（indirect irradiance lighting）**](#3313-pbr和非直接辐射度光照indirect-irradiance-lighting)
        *   [**3.3.2 镜面的 IBL（Specular IBL）**](#332-镜面的iblspecular-ibl)
        *   [**3.3.3 完整的 IBL**](#333-完整的ibl)
*   [**四. 进阶：PBR 核心理论和原理**](#四-进阶pbr核心理论和原理)
    *   [**4.1 再论 PBR 核心理论**](#41-再论pbr核心理论)
    *   [**4.2 光的性质**](#42-光的性质)
        *   [**4.2.1 光是什么？**](#421-光是什么)
        *   [**4.2.2 电磁频谱和可见光（Electromagnetic spectrum and visible light）**](#422-电磁频谱和可见光electromagnetic-spectrum-and-visible-light)
        *   [**4.2.3 人眼感知可见光原理**](#423-人眼感知可见光原理)
        *   [**4.2.4 光的来源**](#424-光的来源)
        *   [**4.2.5 光的理论**](#425-光的理论)
            *   [**4.2.5.1 光的粒子理论 (Particle theory)**](#4251-光的粒子理论particle-theory)
            *   [**4.2.5.2 光的波动理论 (Wave theory)**](#4252-光的波动理论wave-theory)
            *   [**4.2.5.3 光的电磁理论 (Electromagnetic theory)**](#4253-光的电磁理论electromagnetic-theory)
            *   [**4.2.5.4 光的量子理论 (Quantum theory)**](#4254-光的量子理论quantum-theory)
            *   [**4.2.5.5 光的波粒二象性 (Wave-particle duality)**](#4255-光的波粒二象性wave-particle-duality)
        *   [**4.2.6 光的能量**](#426-光的能量)
    *   [**4.3 光学原理（Optics theory）**](#43-光学原理optics-theory)
        *   [**4.3.1 光的反射（Reflection）**](#431-光的反射reflection)
        *   [**4.3.2 光的折射（Refraction）**](#432-光的折射refraction)
        *   [**4.3.3 光的散射（Scattering）**](#433-光的散射scattering)
        *   [**4.3.4 光的色散（Dispersion）**](#434-光的色散dispersion)
        *   [**4.3.5 光的吸收（Absorption）**](#435-光的吸收absorption)
        *   [**4.3.6 光的衍射（Diffraction）**](#436-光的衍射diffraction)
        *   [**4.3.7 光的叠加和干涉（Superposition and interference）**](#437-光的叠加和干涉superposition-and-interference)
        *   [**4.3.8 光的偏振（Polarization）**](#438-光的偏振polarization)
    *   [**4.4 几何光学（Geometry optics）**](#44-几何光学geometry-optics)
        *   [**4.4.1 反射定律（Law of Reflection）**](#441-反射定律law-of-reflection)
        *   [**4.4.2 折射定律（Law of Refraction）**](#442-折射定律law-of-refraction)
        *   [**4.4.3 几何光学的其它定律**](#443-几何光学的其它定律)
    *   [**4.5 物质理论**](#45-物质理论)
        *   [**4.5.1 物质是什么？**](#451-物质是什么)
        *   [**4.5.2 物质结构**](#452-物质结构)
        *   [**4.5.3 物质形态**](#453-物质形态)
        *   [**4.5.4 物质属性**](#454-物质属性)
            *   [**4.5.4.1 导电性**](#4541-导电性)
            *   [**4.5.4.2 粗糙度**](#4542-粗糙度)
            *   [**4.5.4.3 透光性**](#4543-透光性)
            *   [**4.5.4.4 各向性**](#4544-各向性)
    *   [**4.6 能量理论**](#46-能量理论)
        *   [**4.6.1 能量是什么？**](#461-能量是什么)
        *   [**4.6.2 能量类型**](#462-能量类型)
        *   [**4.6.3 能量转化**](#463-能量转化)
        *   [**4.6.4 质能等值**](#464-质能等值)
        *   [**4.6.5 能量守恒 (Conservation of energy)**](#465-能量守恒conservation-of-energy)
    *   [**4.7 PBR 与光学**](#47-pbr与光学)
        *   [**4.7.1 光谱能量分布（SPD）**](#471-光谱能量分布spd)
        *   [**4.7.2 物质与光交互**](#472-物质与光交互)
        *   [**4.7.3 BxDF**](#473-bxdf)
*   [**五. 高阶：PBR 关联理论和推导**](#五-高阶pbr关联理论和推导)
    *   [**5.1 微积分（Calculus）**](#51-微积分calculus)
        *   [**5.1.1 微分（Differentiation）**](#511-微分differentiation)
            *   [**5.1.1.1 函数极限（Functional Limit）**](#5111-函数极限functional-limit)
            *   [**5.1.1.2 导数（Derivative）**](#5112-导数derivative)
            *   [**5.1.1.3 微分（Differentiation）**](#5113-微分differentiation)
        *   [**5.1.2 积分（Integration）**](#512-积分integration)
            *   [**5.1.2.1 不定积分（Indefinite integration）**](#5121-不定积分indefinite-integration)
            *   [**5.1.2.2 定积分（Definite integration）**](#5122-定积分definite-integration)
    *   [**5.2 辐射度量（Radiometry，Radiation Measure）**](#52-辐射度量radiometryradiation-measure)
        *   [**5.2.1 立体角（Solid Angle）**](#521-立体角solid-angle)
        *   [**5.2.2 辐射强度（Radiant Intensity）**](#522-辐射强度radiant-intensity)
        *   [**5.2.3 辐射率（Radiance）**](#523-辐射率radiance)
    *   [**5.3 公式推导**](#53-公式推导)
        *   [**5.3.1 麦克斯韦方程组（Maxwell's equations）**](#531-麦克斯韦方程组maxwells-equations)
            *   [**5.3.1.1 高斯定律（Gauss' law）**](#5311-高斯定律gauss-law)
            *   [**5.3.1.2 高斯磁定律（Gauss's law for magnetism）**](#5312-高斯磁定律gausss-law-for-magnetism)
            *   [**5.3.1.3 法拉第定律（Faraday's law）**](#5313-法拉第定律faradays-law)
            *   [**5.3.1.4 **麦克斯韦 - 安培定律**（Ampère's law with Maxwell's addition）**](#5314-麦克斯韦-安培定律ampères-law-with-maxwells-addition)
        *   [**5.3.2 几何光学基本定律的推导**](#532-几何光学基本定律的推导)
        *   [**5.3.3 Cook-Torrance BRDF 推导**](#533-cook-torrance-brdf推导)
    *   [**5.4 预计算技术**](#54-预计算技术)
        *   [**5.4.1 立方体图卷积（Cubemap convolution）**](#541-立方体图卷积cubemap-convolution)
        *   [**5.4.2 预过滤 HDR 环境图（Pre-filtering HDR environment map）**](#542-预过滤hdr环境图pre-filtering-hdr-environment-map)
            *   [**5.4.2.1 蒙特卡洛（Monte Carlo）积分和重要性采样（Importance sampling）**](#5421-蒙特卡洛monte-carlo积分和重要性采样importance-sampling)
            *   [**5.4.2.2 低差异序列（Low-discrepancy sequence）**](#5422-低差异序列low-discrepancy-sequence)
            *   [**5.4.2.3 GGX 重要性采样（GGX Importance sampling）**](#5423-ggx重要性采样ggx-importance-sampling)
            *   [**5.4.2.3 预过滤卷积瑕疵**](#5423-预过滤卷积瑕疵)
        *   [**5.4.3 预计算 BRDF**](#543-预计算brdf)
    *   [**5.5 PBR 的优化**](#55-pbr的优化)
        *   [**5.5.1 离线渲染优化**](#551-离线渲染优化)
            *   [**5.5.1.1 积分公式优化**](#5511-积分公式优化)
            *   [**5.5.1.2 硬件集成**](#5512-硬件集成)
            *   [**5.5.1.3 并行渲染**](#5513-并行渲染)
            *   [**5.5.1.4 分布式渲染**](#5514-分布式渲染)
        *   [**5.5.2 实时渲染优化**](#552-实时渲染优化)
            *   [**5.5.2.1 光照模型优化**](#5521-光照模型优化)
            *   [**5.5.2.2 资源优化**](#5522-资源优化)
            *   [**5.5.2.3 其它实时优化**](#5523-其它实时优化)
        *   [**5.5.3 移动端优化**](#553-移动端优化)
    *   [**5.6 PBR 的未来**](#56-pbr的未来)
        *   [**5.6.1 基于纳米级别原理**](#561-基于纳米级别原理)
        *   [**5.6.2 更精确的光照模型**](#562-更精确的光照模型)
        *   [**5.6.3 离线技术实时化**](#563-离线技术实时化)
        *   [**5.6.4 新兴理论和技术**](#564-新兴理论和技术)
        *   [**5.6.5 更多应用领域**](#565-更多应用领域)
*   [**六. 后记**](#六--后记)
    *   [**6.1 辅助工具**](#61-辅助工具)
        *   [**6.1.1 曲线工具**](#611-曲线工具)
        *   [**6.1.2 作图工具**](#612-作图工具)
        *   [**6.1.3 写作工具**](#613-写作工具)
        *   [**6.1.4 BRDF Explorer**](#614-brdf-explorer)
    *   [**6.2 更多资料**](#62-更多资料)
*   [**特别说明**](#特别说明)
*   [**参考文献**](#参考文献)

# **一. 前言**

## **1.1 本文动机**

**PBR** 全称 **Physically Based Rendering**，译成中文是**基于物理的渲染**，是当今非常流行的一种拟真渲染技术。

国内外研究它的人数不胜数，由此出现的书籍、论文、文章等资料也非常多，其中最富盛名的非《Physically Based Rendering From Theory to Implementation》莫属，它非常系统全面深入地介绍了 PBR 的底层原理、渲染实现、公式推导、进阶主题等内容。

![](1679148471306.png)

  
_上图：《Physically Based Rendering From Theory to Implementation, 3rd Edition》的封面，**总页数 1270**。_

但是，这些资料大多存在一些问题，它们要么太简单太笼统，不能系统全面地介绍 PBR 的原理和实现；要么太全面太复杂，动辄上千页，对刚踏入 PBR 技术领域的人不够友好，令人望而却步。

## **1.2 PBR 知识体系**

PBR 无疑是一种涉及综合性交叉学科的技术，它涉及的技术有：

*   **数学**：
    *   基础数学
    *   空间几何
    *   线性代数
    *   积分和微积分
    *   统计学
    *   概率论
    *   离散数学
    *   数学建模
    *   ...
*   **物理**：
    *   光学
    *   能量理论
    *   颜色理论
    *   成相理论
    *   物理材料学
    *   原子理论
    *   电磁理论
    *   ...
*   **化学**：
    *   材料学
    *   分子学
*   **生物**：
    *   人眼构造
    *   视觉系统
    *   大脑感光原理
*   **计算机图形学**：
    *   渲染管线
    *   Shader 编码
    *   图形 API
    *   GPU 硬件架构
    *   图像处理
    *   色域空间
    *   ...

![](1679148471360.png)

  
_知乎牛人**毛星云**总结出的 PBR 较完整的知识体系架构图。如果看不清，请点[这里](https://raw.githubusercontent.com/QianMo/PBR-White-Paper/master/media/PBR-White-Paper-Knowledge-Architecture-1.0.png)。_

对于一个刚接触 PBR 技术的新人，一旦接触到如此庞大的知识体系和如此大量陌生的名词、理论、公式、实现，多半会在 PBR 的知识体系中迷失，成为难以逾越的鸿沟，产生畏惧心理，从而放弃 PBR 的学习。

恰好笔者在近期研习了大量 PBR 相关的资料，有些资料反复观摩了几遍，对于 PBR 的渐进式学习有了一定的认知。于是萌生了撰写此篇文章的念头，以便让从未接触过 PBR 的人能循序渐进地学习它、应用它、掌握它、实现它。

## **1.3 本文内容及特点**

本文有以下**特点**：

*   章节分层，层层递进，承上启下。
*   引入大量精美配图，使 PBR 教学生动有趣。
*   采用 Markdown 语法，图文编排更精美和统一。
*   采用 $L^AT_EX$表达数学公式。
*   语句精炼，浅显易懂。
*   丰富多样的举例。
*   紧紧围绕 PBR 的核心原理，避免节外生枝。

本篇文章主要有以下章节内容，分别从不同层次不同侧重点描述了 PBR 的原理、实现、应用，面向的群体也不一样：

*   **初阶：PBR 基本认知和应用**
    *   内容：
        *   介绍 PBR 的概念、历史、应用。
    *   面向：
        *   初级程序员
        *   美术
        *   初级 TA（技术美术）
        *   想初步了解 PBR 基本认知和应用的人
*   **中阶：PBR 基本原理和实现**
    *   内容：
        *   介绍 PBR 的基本原理和商业引擎的实现。
    *   面向：
        *   中级程序员
        *   TA
        *   想了解 PBR 实现的人
*   **进阶：PBR 核心理论和原理**
    *   内容：
        *   深入介绍 PBR 的核心理论和渲染原理。
    *   面向：
        *   进阶程序员
        *   对 PBR 底层原理感兴趣的人
*   **高阶：PBR 关联理论和推导**
    *   内容：
        *   介绍跟 PBR 核心理论相关的理论原理及推导。
    *   面向：
        *   高阶程序员
        *   想全面且透彻了解 PBR 底层原理及理论的人

从上面可以看出，每个章节都是承上启下，层层递进地剖析 PBR 原理及实现，从而达到由浅入深介绍 PBR 的目的。读者可以根据目前的水平，以及想要了解的程度，有针对性有选择性地阅读不同的章节。其实本文标题更适合取为《分层渐进式学习 PBR 的原理和实现》，但，还是现在的标题浅显易懂。

需要注意的是，本文将围绕 **PBR 的核心理论**做阐述，以实时渲染领域的 Cook-Torrance 的 BRDF 光照模型为实现案例。其它旁系理论不在本文探讨的范围，可以查阅其它文献。

# **二. 初阶：PBR 基本认知和应用**

本章内容主要介绍 PBR 的基本概念和衍变历史，以及其在主流商业引擎的应用。  
面向的群体：

*   初级程序员
*   美术
*   初级 TA（技术美术）
*   想初步了解 PBR 基本认知和应用的人

## **2.1 PBR 的基本介绍**

### **2.1.1 PBR 概念**

**PBR**（**Physically Based Rendering**）译成中文是基于物理的渲染。它是利用真实世界的原理和理论，通过各种数学方法推导或简化或模拟出一系列渲染方程，并依赖计算机硬件和图形 API 渲染出拟真画面的技术。

### **2.1.2 与物理渲染的差别**

那它为什么不叫**物理渲染**（**Physical Rendering**）呢？

物理渲染（Physical Rendering）是指跟真实世界完全一致的计算机渲染效果。

为了回答这个问题，先了解一下真实世界的成相原理。

真实世界的物体有着各自的材质属性和表面特征，它们受到各种局部灯光和全局环境光的影响，而且它们之间又相互影响，最终这些信息通过光波的形式进入复杂的人眼构造，刺激视神经形成生物信号进入大脑感光皮层，最终让人产生视觉认知。（下图）  

![](1679148473225.png)

有论文指出，绝大多数人的眼睛可以接收相当于 **5 亿 10 亿 ** 个像素的信息量。目前主流的分辨率才百万千万级别，加上显示器亮度范围和屏幕像素间距的限制，远远达不到亿级像素的渲染和亮度表示范围。

基于现阶段的知识水平和硬件水平，还不能渲染跟真实世界完全一致的效果，只能一定程序上模拟接近真实世界的渲染画面，故而叫**基于物理的渲染**（**Physically Based Rendering**），而非**物理渲染**（**Physical Rendering**）。

### **2.1.3 PBR 的特征**

这节阐述的是 PBR 呈现的效果特征，而非底层物理原理的特征。  
相比传统的 Lambert 着色和 Phong 着色，PBR 着色在效果上有着质的提升，可以表示更多更复杂的材质特征：

*   表面细节
*   物体粗糙度
*   区别明显的金属和绝缘体
*   物体的浑浊程度
*   菲涅尔现象：不同角度有不同强度的反射光
*   半透明物体
*   多层混合材质
*   清漆效果
*   其它更复杂的表面特征

![](1679148473257.png)

  
_Phong 模型着色效果，只能简单地表现理想模型的漫反射和高光，渲染出的效果跟真实世界相差甚远。_

![](1679148473288.png)

  
_PBR 材质效果球，它们真实地渲染出各类材质的粗糙、纹理、高光、清漆、边缘光等等表面细节特征。PBR 对渲染效果真实感的提升可见一斑。_

## **2.2 PBR 的衍变历史**

PBR 从最初传统型的 Lambert 光照发展至今，已经历经 200 多年，期间发生多次迭代衍变和改进，主流光照模型和分支光照模型也遍地开花。下面按照时间顺序着重对 PBR 衍变的**关键技术节点**做阐述。

### **2.2.1 Lambert（1760 年）**

Lambert 模型是 Johann Heinrich Lambert 在 1760 年提出的光照模型。是传统的光照模型。

它计算的是漫反射。漫反射是光源照射到物体表面后，向四面八方反射，产生的反射效果。这是一种理想的漫反射光照模型。

漫反射光的强度近似地服从于 Lambert 定律，即漫反射光的光强仅与入射光的方向和反射点处表面法向夹角的余弦成正比。

![](1679148473333.png)

  
_Lambert 模型着色效果，模拟了理想环境下的漫反射效果。_

### **2.2.2 Smith（1967 年）**

Smith 将 Cook-Torrance 的 DFG 部分的 G 几何项有效地结合起来，使得几何函数的近似法得到了有效地提升，后面章节将会阐述更多细节。

### **2.2.3 Phong（1973 年）**

Phong 模型由美国越南裔学者裴祥风（Bùi Tường Phong）发明，于 1973 年的博士论文首度发表。它也是一种传统的理想的光照模型。

相较 Lambert，Phong 增加了镜面反射部分，使得物体渲染效果更接近真实世界（下图）。  

![](1679148473408.png)

### **2.2.4 Cook-Torrance（1982 年）**

Cook-Torrance 是 Cook 和 Torrance 于 1982 年联合提出的光反射模型。

该模型考虑在同一景物中不同材料和不同光源的相对亮度。它描述反射光线在方向上的分布和当反射随入射角而改变时颜色的变化，并能求得从具体的实际材料制成的物体反射出来的光线的光谱能量分布，并根据这种光谱能量分布精确地再现颜色。

简而言之，Cook-Torrance 增加了几何项 G、Fresnel 项、粗糙度项 D 等信息。利用该模型渲染出的图像真实感有了较大跨度的提升。  

![](1679148473445.png)

  
_Cook-Torrance 光照模型渲染效果。它较好地渲染出模型的表面特征和光照效果。_

### **2.2.5 Oren Nayarh（1994 年）**

Lambert 模型由于是理想环境下的光照模拟，不能正确体现物体（特别是粗糙物体）表面的光照效果。

Oren Nayarh 模型对此做出了改进，主要对粗糙表面的物体建模，比如石膏、沙石、陶瓷等。用了一系列的 Lambert 微平面，考虑了微小平面之间的相互遮挡（shadowing and masking）和互相反射照明。它能一定程度上模拟真实物体的表面粗糙度，使物体更有质感。  

![](1679148473469.png)

  
_左：真实照片，中：Lambert 模型效果，右：Oren Nayarh 模型效果_

### **2.2.6 Schlick（1994 年）**

Schlick 模型简化了 Phong 模型的镜面反射中的指数运算。采用以下公式替代：

$$F = F_0+(1-F_o)(1-cos(\theta))^5$$

$$F_0 = (\frac{n_1-n_2}{n_1+n_2})^2$$

它模拟的高光反射效果跟 Pow 运算基本一致，且效率比 Pow 运算高。

### **2.2.7 GGX（2007 年）**

GGX 模型所解决的问题是，如何将微平面反射模型推广到表面粗糙的**半透明**材质，从而能够模拟类似于毛玻璃的粗糙表面的**透射**效果。同时，它也提出了一种新的微平面分布函数 。  

![](1679148473500.png)

  
_上图：GGX 非常逼真地模拟**半透明**物体的效果。_

虽然它提出时被用于半透明物体的模拟，但它作为一种描述微平面**法线**方向分布的函数，同样适用于渲染表面粗糙的**不透明**物体。  

![](1679148473524.png)

  
_GGX 同样可以非常逼真地模拟**不透明**物体的效果_

GGX 已经广泛应用于各种主流游戏引擎中，同时也是效果最好的。

### **2.2.8 迪斯尼原则的 BRDF（Disney principled BRDF, 2012 年)**

在 SIGGRAPH 2012 会议上，工作于迪斯尼动画工作室的 **Brent Burly** 演讲了著名的主题：**《Physically Based Shading at Disney》**。  

![](1679148473565.png)

  
_Brent Burly 在 SIGGRAPH 2012 演讲迪斯尼原则的 PBR。_

他提出了**迪斯尼原则的 BRDF（Disney Principled BRDF）**，**奠定了后续游戏行业和电影行业 PBR 的方向和标准**。后续的主流游戏引擎，3D 渲染器及动画制作软件大多基于此方案或变种实现的。  

![](1679148473589.png)

  
_迪斯尼原则的 PBR 渲染出的《无敌破坏王》画面。_

迪斯尼原则的 BRDF 用少量简单易懂的参数和高度完善的美术工作流程，大大简化了此前复杂的 PBR 的参数和制作流程。它是**艺术导向（Art Directable）**的着色模型，而不完全是**物理正确（Physically Correct）**。  

![](1679148473681.png)

  
_迪斯尼原则的 BRDF 抽象出的参数。_

### **2.2.9 现阶段的 BxDF（2019 年）**

基于物理的光照模型已经发展了数十年，期间衍生的关键技术和变种技术非常多，它们各有适用场景或解决的各个具体应用场景的问题。

近今年，PBR 的技术主要朝着更逼真、更复杂、效能更好的方向，或是结合若干种模型的综合性技术迈进。代表性技术有：

*   PBR Diffuse for GGX + Smith (2017)
*   MultiScattering Diffuse (2018)
*   Layers Material（分层材质）
*   Mixed Material（混合材质）
*   Mixed BxDF（混合 BxDF）
*   Advanced Rendering（进阶渲染）

![](1679148473935.png)

  
_UE4 渲染出的虚拟人 Siren。综合了分层材质、混合材质、混合 BxDF、眼球毛发和皮肤渲染等新兴技术。_

![](1679148473965.png)

  
_虚拟人 Siren 的皮肤细节。与数码相机摄制的相片如出一辙，逼真程度令人咂舌。如果不特意提醒，很难相信这是游戏引擎实时渲染出来的画面。_

## **2.3 PBR 的应用领域**

PBR 经过长时间的发展，技术上和渲染的效果突飞猛进，是计算机图形学的下一代渲染技术。它在实时渲染和离线渲染领域都有着非常广泛且深入的应用，主要有：

*   **电影和动漫**。使用 PBR 技术渲染的真人电影，拟真电影，以及各类动漫电影数量非常多，比如早些年的《阿凡达》《飞屋环游记》，近期的《战斗天使》《流浪地球》《驯龙高手 3》等。  
    
    ![](1679148474070.png)
    
      
    _电影《阿凡达》的人物画面。_  
    
    ![](1679148474125.png)
    
      
    _电影《战斗天使》的画面。主角阿丽塔是计算机通过 PBR 技术渲染出来的虚拟角色，她与真人演员和真实环境无缝地融合在了一起。_  
    
    ![](1679148474175.png)
    
      
    _电影《流浪地球》的虚拟场景。特效制作公司利用 PBR 技术模拟出恐怖的身临其境的画面。_
    
*   **实时游戏**。PBR 的身影流传于 PC 游戏，在线游戏，移动游戏，主机游戏等游戏细分领域。相信接触过游戏的人大多体验过次世代效果的魅力。  
    
    ![](1679148474228.png)
    
      
    _PC 网游《逆水寒》的角色次世代效果。_  
    
    ![](1679148474285.png)
    
      
    _移动游戏《绝地求生 · 刺激战场》的次世代场景。_  
    
    ![](1679148474387.png)
    
      
    _单机游戏《极品飞车 20》的动感瞬间。_
    
*   **计算机辅助设计与制造（CAD/CAM）**。计算机图形学刚起步时，便应用于此领域，PBR 的引入，更加真实地帮助设计人员设计出与实物相差无几的产品。  
    
    ![](1679148474408.png)
    
      
    _电路板设计预渲染效果图。_  
    
    ![](1679148474680.png)
    
      
    _跑车概念设计效果图。_  
    
    ![](1679148474715.png)
    
      
    _室内家装设计效果图。_
    
*   **计算机辅助教学 (CAI)**。通过逼真的 PBR 技术，渲染出教学内容所需的虚拟场景，佐以动画技术，使得教学更加形象生动有趣。
    
*   **虚拟现实 (VR/AR/MR)**。虚拟技术通常需要佩戴眼镜或头盔等显示设备，较多地用于军事，教学，模拟训练，医学等领域。而 VR 引入 PBR 技术，能更逼真地模拟现实世界，让参与者身临其境。  
    
    ![](1679148474753.png)
    
      
    _Magic Leap 制作的 VR 概念图。_
    
*   **科学计算可视化**。气象、地震、天体物理、分子生物学、医学等科学领域采用 PBR 技术将更真实地模拟自然规律，有助于科学家新发现，有助于高校师生教学。  
    
    ![](1679148474790.png)
    
      
    _计算机模拟出的 DNA 双螺旋结构图。_
    

## **2.4 PBR 在游戏引擎的应用**

迪斯尼自 2012 年提出迪斯尼原则的 PBR 理论后，在游戏和电影界引起轰动，随后各大主流游戏引擎和渲染器及建模软件纷纷实现基于斯尼原则的 PBR 技术。

下面是主流游戏引擎支持迪斯尼原则的 PBR 时间表：

*   **Unreal Engine 4**：《Real Shading in Unreal Engine 4》，SIGGRAPH 2013
*   **Unity 5**：《Physically Based Shading in Unity》，GDC 2014
*   **Frostbite（寒霜）**： 《Moving Frostbite to PBR》，SIGGRAPH 2014
*   **Cry Engine 3.6**：《Physically Based Shading in Cry Engine》，2015

UE4 和 Unity 在算法上的实现略有差别，但本章先不讨论算法的实现问题，主要阐述材质上的参数。

### **2.4.1 Unreal Engine 4 的 PBR**

UE4 的 PBR 相对其它迪斯尼原则的 PBR 实现，在参数方面做了精简，涉及的参数主要有：

*   **基础色（Base Color）**：为材质提供基础纹理色，是 Vector3（RGB），它们的值都限定在 0~1 之间。  
    
    ![](1679148474816.png)
    
      
    _利用 UE4 的材质编辑器处理 Base Color。_

下表是经过测量后得出的**非金属**材质的基础色强度（非金属材质只有**单色**，即强度）：

<table><thead><tr><th>材质 (Material)</th><th style="text-align: center">基础色强度 (BaseColor Intensity)</th></tr></thead><tbody><tr><td>木炭 (Charcoal)</td><td style="text-align: center">0.02</td></tr><tr><td>新沥青 (Fresh asphalt)</td><td style="text-align: center">0.02</td></tr><tr><td>旧沥青 (Worn asphalt)</td><td style="text-align: center">0.08</td></tr><tr><td>土壤 (Bare soil)</td><td style="text-align: center">0.13</td></tr><tr><td>绿草 (Green Grass)</td><td style="text-align: center">0.21</td></tr><tr><td>沙漠沙 (desert sand)</td><td style="text-align: center">0.36</td></tr><tr><td>新混泥土 (Fresh concrete)</td><td style="text-align: center">0.51</td></tr><tr><td>海洋冰 (Ocean Ice)</td><td style="text-align: center">0.56</td></tr><tr><td>鲜雪 (Fresh snow)</td><td style="text-align: center">0.81</td></tr><tr><td>下表是经过测量后得出的<strong>金属</strong>材质的基础色（R, G, B），是在 Linear 色域空间的值：</td><td style="text-align: center"></td></tr></tbody></table>

<table><thead><tr><th>材质 (Material)</th><th style="text-align: center">基础色 (BaseColor)</th></tr></thead><tbody><tr><td>铁 (Iron)</td><td style="text-align: center">(0.560, 0.570, 0.580)</td></tr><tr><td>银 (Silver)</td><td style="text-align: center">(0.972, 0.960, 0.915)</td></tr><tr><td>铝 (Aluminum)</td><td style="text-align: center">(0.913, 0.921, 0.925)</td></tr><tr><td>金 (Gold)</td><td style="text-align: center">(1.000, 0.766, 0.336)</td></tr><tr><td>铜 (Copper)</td><td style="text-align: center">(0.955, 0.637, 0.538)</td></tr><tr><td>铬 (Chromium)</td><td style="text-align: center">(0.550, 0.556, 0.554)</td></tr><tr><td>镍 (Nickel)</td><td style="text-align: center">(0.660, 0.609, 0.526)</td></tr><tr><td>钛 (Titanium)</td><td style="text-align: center">(0.542, 0.497, 0.449)</td></tr><tr><td>钴 (Cobalt)</td><td style="text-align: center">(0.662, 0.655, 0.634)</td></tr><tr><td>铂 (Platinum)</td><td style="text-align: center">(0.672, 0.637, 0.585)</td></tr></tbody></table>

*   **粗糙度（Roughness）**：表示材质表面的粗糙程度，值限定在 0~1 之间。越粗糙材质高光反射越不明显，金属和非金属的粗糙度有所区别。  
    
    ![](1679148474839.png)
    
      
    
    ![](1679148474873.png)
    
      
    _上：非金属材质随粗造度从 0-1 变化而渐变的图，下：金属材质随粗造度从 0-1 变化而渐变的图。_
    
*   **金属度（Metallic）**：表示材质像金属的程度，0 是电介质（绝缘体），1 是金属。金属没有漫反射，只有镜面反射。  
    
    ![](1679148474922.png)
    
      
    _金属度从 0~1 的变化图。_
    
*   **镜面度（Specular）**：表示材质的镜面反射强度，从 0（完全无镜面反射）~1（完全镜面反射。UE4 的默认值是 0.5。万物皆有光泽（镜面反射），对于强漫反射的材质，可通过调节粗糙度，而不应该将镜面度调成 0。  
    
    ![](1679148474982.png)
    
      
    _镜面度从 0~1 的变化图。_
    

下表是 UE4 给出的部分材质镜面度参考值：

<table><thead><tr><th>材质 (Material)</th><th style="text-align: center">镜面度 (Specular)</th></tr></thead><tbody><tr><td>草 (Glass)</td><td style="text-align: center">0.500</td></tr><tr><td>塑料 (Plastic)</td><td style="text-align: center">0.500</td></tr><tr><td>石英 (Quartz)</td><td style="text-align: center">0.570</td></tr><tr><td>冰 (Ice)</td><td style="text-align: center">0.224</td></tr><tr><td>水 (Water)</td><td style="text-align: center">0.255</td></tr><tr><td>牛奶 (Milk)</td><td style="text-align: center">0.277</td></tr><tr><td>皮肤 (Skin)</td><td style="text-align: center">0.350</td></tr></tbody></table>

UE4 模拟的部分材质效果见下图。  

![](1679148475030.png)

  
_上排从左到右：木炭、生混凝土、旧沥青；下排从左到右：铜、铁、金、铝、银、镍、钛。_

### **2.4.2 Unity 的 PBR**

Unity 的 PBR 已经纳入内建的标准着色器（Standard Shader），它的实现准则是用户友好的（user-friendly），故而在材质编辑器里呈现给用户是有限的参数，而且跟传统的各类贴图信息统一在了一起。

Unity 内部实现机制遵循了 PBR 的基本准则，支持金属度，表面粗糙度，能量守恒，菲涅尔反射，表面阴影遮蔽等特性。  

![](1679148475056.png)

  
_Unity 的 Standard Shader 编辑界面。_

其中跟 PBR 相关的参数：

*   **Albedo**：基础色，相当于 UE4 的 Base Color。可用纹理贴图指定，也可用一个颜色值代替。
*   **Metallic**：金属度，意义跟 UE4 的一致。但它可以用金属贴图代替，此时 Smoothness 参数会消失。  
    
    ![](1679148475320.png)
    
      
    _Unity 指定了 Metallic 贴图后的效果，Smoothness 参数消失。_
*   **Smoothness**：光滑度，跟 UE 的粗糙度取值刚好相反，但都是表示材质表面的粗糙程度。  
    
    ![](1679148475708.png)
    
      
    _Unity 的 Smoothness 参数从 0~1 的变化。_
    *   **Smoothness Source**：指定存储光滑度数据的纹理通道，可选择金属度、镜面贴图的 Alpha 通道或基础色贴图的 Alpha 通道。
*   **Occlusion**：遮蔽图。用于指定材质接受间接光（如环境光）的光照强度和反射强度。  
    
    ![](1679148475741.png)
    
      
    _Unity 中使用遮蔽图为人物阴暗面（脸部，脖子）屏蔽环境光的影响。_
*   **Fresnel**：随着物体表面法线与视线的角度增大，物体的反射能力增大，这种现象称之为菲涅尔效应。在 Unity 中，无法直接调节菲涅尔效应的参数，但内部实现机制会自动处理。越光滑的表面具有越强的菲涅尔效应，相反，越粗糙的表面具有越弱的菲涅尔效应。  
    
    ![](1679148476133.png)
    
      
    _上图展示了菲涅尔效应从弱到强的渐变。_

# **三. 中阶：PBR 基本原理和实现**

上章主要介绍了 PBR 的历史和逼真的效果特征。这章将重点介绍 PBR 的核心部分的基本原理及主流的实现方案，使读者对 PBR 的核心理论有一定了解，并能掌握相关的编码。

主要面向：

*   中级程序员
*   TA
*   想了解 PBR 基本原理和实现的人

在分析比较了大量资料之后，本章选取了 [LearnOpenGL 的 PBR 教程](https://learnopengl.com/PBR/Theory)作为依托，阐述 PBR 的基本原理和实现。

## **3.1 PBR 基础理论和推导**

本节的理论和推导尽量简化和精简，更深入的原理和理论将在下一章阐述。

满足以下条件的光照模型才能称之为 PBR 光照模型：

*   基于微平面模型（Be based on the microfacet surface model）。
*   能量守恒（Be energy conserving）。
*   使用基于物理的 BRDF（Use a physically based BRDF）。

### **3.1.1 微平面（Microfacet）**

大多数 PBR 技术都是基于微平面理论。在此理论下，认为在微观上所有材质表面都是由很多朝向不一的微小平面组成，有的材质表面光滑一些，有的粗糙一些。

真实世界的物体表面不一定是很多微小平面组成，也可能是带有弧度或者坑坑洼洼。但对于我们肉眼能观察到的维度，PBR 的微观近似模拟方法产生的结果跟实际差别甚微。

![](1679148476503.png)

  
_所有材质表面由粗糙度不同的微小平面组成。左边材质更粗糙，右边的平滑一些。_

当光线射入这些微平面后，通常会产生镜面反射。对于越粗糙的表面，由于其朝向更无序，反射的光线更杂乱，反之，平滑的微平面，反射的光线更平齐。  

![](1679148476534.png)

  
_上图左边材质表面更粗糙，反射的光线更杂乱；图右的平滑许多，反射的光线更有规律。_

从微观角度来说，没有任何表面是完全光滑的。由于这些微平面已经微小到无法逐像素地继续对其进行细分，因此我们只有假设一个粗糙度 (Roughness，即 2.4.1 中提到的粗糙度) 参数，然后用统计学的方法来概略的估算微平面的粗糙程度。

我们可以基于一个平面的粗糙度来计算出某个向量的方向与微平面平均取向方向一致的概率。这个向量便是位于光线向量 $l$和视线向量 $v$之间的中间向量，被称为**半角向量 (Halfway Vector)**。  

![](1679148476577.png)

  
_半角向量 $h$是视线 $v$和入射光 $l$的中间单位向量。_

半角向量计算公式如下：

$$h = \frac{l + v}{\|l + v\|}$$

半角向量计算 GLSL 实现:

```
// lightPos是光源位置，viewPos是摄像机位置，FragPos是像素位置
vec3 lightDir   = normalize(lightPos - FragPos);
vec3 viewDir    = normalize(viewPos - FragPos);
vec3 halfwayDir = normalize(lightDir + viewDir);
```

越多的微平面取向与其半角向量一致，材质镜面反射越强越锐利。加上引入取值 0~1 的粗糙度，可以大致模拟微平面的整体取向。  

![](1679148476601.png)

  
_粗糙度从 0.1~1.0 的变化图。粗糙度越小，镜面反射越亮范围越小；粗糙度越大，镜面反射越弱。_

### **3.1.2 能量守恒（Energy Conservation）**

在微平面理论中，采用近似的能量守恒：出射光的总能量不超过入射光的总能量（自发光材质除外）。3.1.1 的粗糙度变化图可以看出，材质粗糙度越大，反射的范围越大，但整体亮度变暗。

那么 PBR 是如何实现近似的能量守恒呢？

为了回答这个问题，先弄清楚**镜面反射**（specular）和**漫反射**（diffuse）的区别。

一束光照到材质表面上，通常会分成**反射**（reflection）部分和**折射**（refraction）部分。反射部分直接从表面反射出去，而不进入物体内部，由此产生了镜面反射光。折射部分会进入物体内部，被吸收或者散射产生漫反射。

折射进物体内部的光如果没有被立即吸收，将会持续前进，与物体内部的微粒产生碰撞，每次碰撞有一部分能量损耗转化成热能，直至光线能量全部消耗。有些折射光线在跟微粒发生若干次碰撞之后，从物体表面射出，便会形成漫反射光。  

![](1679148476642.png)

  
_照射在平面的光被分成镜面反射和折射光，折射光在跟物体微粒发生若干次碰撞之后，有可能发射出表面，成为漫反射。_

通常情况下，PBR 会简化折射光，将平面上所有折射光都视为被完全吸收而不会散开。而有一些被称为次表面散射 (Subsurface Scattering) 技术的着色器技术会计算折射光散开后的模拟，它们可以显著提升一些材质（如皮肤、大理石或蜡质）的视觉效果，不过性能也会随着下降。

金属 (Metallic) 材质会立即吸收所有折射光，故而金属只有镜面反射，而没有折射光引起的漫反射。

回到能量守恒话题。反射光与折射光它们二者之间是互斥的，被表面反射出去的光无法再被材质吸收。故而，进入材质内部的折射光就是入射光减去反射光后余下的能量。

根据上面的能量守恒关系，可以先计算镜面反射部分，此部分等于入射光线被反射的能量所占的百分比。而折射部分可以由镜面反射部分计算得出。

```c
float kS = calculateSpecularComponent(...); // 反射/镜面部分
float kD = 1.0 - kS;                        // 折射/漫反射部分
```

通过以上代码可以看出，镜面反射部分与漫反射部分的和肯定不会超过 1.0，从而近似达到能量守恒的目的。

### **3.1.3 反射方程（Reflectance Equation）**

**渲染方程** (Render Equation) 是用来模拟光的视觉效果最好的模型。而 PBR 的渲染方程是用以抽象地描述 PBR 光照计算过程的特化版本的渲染方程，被称为**反射方程**。

PBR 的反射方程可抽象成下面的形式：

$$L_o(p,\omega_o) = \int\limits_{\Omega} f_r(p,\omega_i,\omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

反射方程看似很复杂，但如果拆分各个部分加以解析，就可以揭开其神秘的面纱。

为了更好地理解反射方程，先了解**辐射度量学 (Radiometry)**。辐射度量学是一种用来度量电磁场辐射（包括可见光）的手段。有很多种辐射度量 (radiometric quantities) 可以用来测量曲面或者某个方向上的光，此处只讨论和反射方程有关的一种量，它就是**辐射率 (Radiance)**，用 $L$来表示。

先用一个表展示辐射度量学涉及的概念、名词、公式等信息，后面会更加详细地介绍。

<table><thead><tr><th>名称</th><th>符号</th><th>单位</th><th>公式</th><th>解析</th></tr></thead><tbody><tr><td><strong>辐射能量</strong> (Radiant energy)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-18-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>Q</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-166" style="width: 1.031em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.803em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.75em, 2.631em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-167"><span class="mi" id="MathJax-Span-168" style="font-family: MathJax_Math-italic;">Q</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.282em; border-left: 0px solid; width: 0px; height: 1.289em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>Q</mi></math></span></span><script type="math/tex" id="MathJax-Element-18">Q</script></span></td><td>焦耳 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-19-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>J</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-169" style="width: 0.803em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.63em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-170"><span class="mi" id="MathJax-Span-171" style="font-family: MathJax_Math-italic;">J<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>J</mi></math></span></span><script type="math/tex" id="MathJax-Element-19">J</script></span>)</td><td>-</td><td>电磁辐射能量</td></tr><tr><td><strong>辐射通量</strong> (Radiant Flux)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-20-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-172" style="width: 0.974em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.69em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-173"><span class="mi" id="MathJax-Span-174" style="font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi></math></span></span><script type="math/tex" id="MathJax-Element-20">\Phi</script></span></td><td>瓦 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-21-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>W</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-175" style="width: 1.374em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.089em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1001.09em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-176"><span class="mi" id="MathJax-Span-177" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>W</mi></math></span></span><script type="math/tex" id="MathJax-Element-21">W</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-22-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi>Q</mi></mrow><mrow><mi>d</mi><mi>t</mi></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-178" style="width: 4.174em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.317em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.089em, 1003.32em, 2.86em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-179"><span class="mi" id="MathJax-Span-180" style="font-family: MathJax_Main;">Φ</span><span class="mo" id="MathJax-Span-181" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-182" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.92em, 4.289em, -999.997em); top: -4.569em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-183"><span class="mi" id="MathJax-Span-184" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-185" style="font-size: 70.7%; font-family: MathJax_Math-italic;">Q</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.63em, 4.174em, -999.997em); top: -3.597em; left: 50%; margin-left: -0.283em;"><span class="mrow" id="MathJax-Span-186"><span class="mi" id="MathJax-Span-187" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-188" style="font-size: 70.7%; font-family: MathJax_Math-italic;">t</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.03em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.031em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.568em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi>Q</mi></mrow><mrow><mi>d</mi><mi>t</mi></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-22">\Phi = \frac{dQ}{dt}</script></span></td><td>单位时间辐射的能量，也叫辐射功率 (Radiant Power) 或通量(Flux)</td></tr><tr><td><strong>辐照度</strong> (Irradiance)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-23-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>E</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-189" style="width: 0.974em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.75em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-190"><span class="mi" id="MathJax-Span-191" style="font-family: MathJax_Math-italic;">E<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>E</mi></math></span></span><script type="math/tex" id="MathJax-Element-23">E</script></span></td><td>瓦 / 平方米 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-24-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><msup><mi>m</mi><mn>2</mn></msup></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-192" style="width: 3.603em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.86em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1002.86em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-193"><span class="texatom" id="MathJax-Span-194"><span class="mrow" id="MathJax-Span-195"><span class="mi" id="MathJax-Span-196" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-197"><span class="mrow" id="MathJax-Span-198"><span class="mo" id="MathJax-Span-199" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-200"><span class="mrow" id="MathJax-Span-201"><span class="msubsup" id="MathJax-Span-202"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-203" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-204" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><msup><mi>m</mi><mn>2</mn></msup></mrow></math></span></span><script type="math/tex" id="MathJax-Element-24">{W}/{m^2}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-25-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-205" style="width: 4.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.774em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1003.77em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-206"><span class="mi" id="MathJax-Span-207" style="font-family: MathJax_Main;">Φ</span><span class="mo" id="MathJax-Span-208" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-209" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.489em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-210"><span class="mi" id="MathJax-Span-211" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-212" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1001.32em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -0.683em;"><span class="mrow" id="MathJax-Span-213"><span class="mi" id="MathJax-Span-214" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-215"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-216" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-217" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.49em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.489em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-25">\Phi = \frac{d\Phi}{dA^\perp}</script></span></td><td><em>到达</em>单位面积的辐射通量</td></tr><tr><td><strong>辐射度</strong> (Radiosity)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-26-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>M</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-218" style="width: 1.317em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1001.03em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-219"><span class="mi" id="MathJax-Span-220" style="font-family: MathJax_Math-italic;">M<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>M</mi></math></span></span><script type="math/tex" id="MathJax-Element-26">M</script></span></td><td>瓦 / 平方米 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-27-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><msup><mi>m</mi><mn>2</mn></msup></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-221" style="width: 3.603em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.86em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1002.86em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-222"><span class="texatom" id="MathJax-Span-223"><span class="mrow" id="MathJax-Span-224"><span class="mi" id="MathJax-Span-225" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-226"><span class="mrow" id="MathJax-Span-227"><span class="mo" id="MathJax-Span-228" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-229"><span class="mrow" id="MathJax-Span-230"><span class="msubsup" id="MathJax-Span-231"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-232" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-233" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><msup><mi>m</mi><mn>2</mn></msup></mrow></math></span></span><script type="math/tex" id="MathJax-Element-27">{W}/{m^2}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-28-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>M</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-234" style="width: 5.146em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.117em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1004.12em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-235"><span class="mi" id="MathJax-Span-236" style="font-family: MathJax_Math-italic;">M<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mo" id="MathJax-Span-237" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-238" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.489em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-239"><span class="mi" id="MathJax-Span-240" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-241" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1001.32em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -0.683em;"><span class="mrow" id="MathJax-Span-242"><span class="mi" id="MathJax-Span-243" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-244"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-245" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-246" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.49em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.489em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>M</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-28">M = \frac{d\Phi}{dA^\perp}</script></span></td><td><em>离开</em>单位面积的辐射通量，也叫辐出度、辐射出射度（Radiant Existance）</td></tr><tr><td><strong>辐射强度</strong> (Radiant Intensity)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-29-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>I</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-247" style="width: 0.689em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.517em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.52em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-248"><span class="mi" id="MathJax-Span-249" style="font-family: MathJax_Math-italic;">I<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>I</mi></math></span></span><script type="math/tex" id="MathJax-Element-29">I</script></span></td><td>瓦 / 立体弧度 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-30-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>s</mi><mi>r</mi></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-250" style="width: 3.089em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.46em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1002.46em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-251"><span class="texatom" id="MathJax-Span-252"><span class="mrow" id="MathJax-Span-253"><span class="mi" id="MathJax-Span-254" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-255"><span class="mrow" id="MathJax-Span-256"><span class="mo" id="MathJax-Span-257" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-258"><span class="mrow" id="MathJax-Span-259"><span class="mi" id="MathJax-Span-260" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-261" style="font-family: MathJax_Math-italic;">r</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><mi>s</mi><mi>r</mi></mrow></math></span></span><script type="math/tex" id="MathJax-Element-30">{W}/{sr}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-31-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>I</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><mi>&amp;#x03C9;</mi></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-262" style="width: 3.946em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.146em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1003.15em, 2.86em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-263"><span class="mi" id="MathJax-Span-264" style="font-family: MathJax_Math-italic;">I<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mo" id="MathJax-Span-265" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-266" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-267"><span class="mi" id="MathJax-Span-268" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-269" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.8em, 4.174em, -999.997em); top: -3.597em; left: 50%; margin-left: -0.397em;"><span class="mrow" id="MathJax-Span-270"><span class="mi" id="MathJax-Span-271" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-272" style="font-size: 70.7%; font-family: MathJax_Math-italic;">ω</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.03em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.031em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.568em; border-left: 0px solid; width: 0px; height: 1.789em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>I</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><mi>ω</mi></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-31">I = \frac{d\Phi}{d\omega}</script></span></td><td>通过单位立体角的辐射通量</td></tr><tr><td><strong>辐射率</strong> (Radiance)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-32-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>L</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-273" style="width: 0.86em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.63em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-274"><span class="mi" id="MathJax-Span-275" style="font-family: MathJax_Math-italic;">L</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>L</mi></math></span></span><script type="math/tex" id="MathJax-Element-32">L</script></span></td><td>瓦 / 平方米立体弧度 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-33-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><msup><mi>m</mi><mn>2</mn></msup><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>s</mi><mi>r</mi></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-276" style="width: 4.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.774em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1003.77em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-277"><span class="texatom" id="MathJax-Span-278"><span class="mrow" id="MathJax-Span-279"><span class="mi" id="MathJax-Span-280" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-281"><span class="mrow" id="MathJax-Span-282"><span class="mo" id="MathJax-Span-283" style="font-family: MathJax_Main;">/</span></span></span><span class="msubsup" id="MathJax-Span-284"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-285" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-286" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="texatom" id="MathJax-Span-287"><span class="mrow" id="MathJax-Span-288"><span class="mi" id="MathJax-Span-289" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-290" style="font-family: MathJax_Math-italic;">r</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><msup><mi>m</mi><mn>2</mn></msup><mrow class="MJX-TeXAtom-ORD"><mi>s</mi><mi>r</mi></mrow></math></span></span><script type="math/tex" id="MathJax-Element-33">{W}/m^2{sr}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-34-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>L</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><mi>&amp;#x03C9;</mi><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-291" style="width: 5.717em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.574em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1004.57em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-292"><span class="mi" id="MathJax-Span-293" style="font-family: MathJax_Math-italic;">L</span><span class="mo" id="MathJax-Span-294" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-295" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 2.289em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-296"><span class="mi" id="MathJax-Span-297" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-298" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1002.17em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -1.083em;"><span class="mrow" id="MathJax-Span-299"><span class="mi" id="MathJax-Span-300" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-301" style="font-size: 70.7%; font-family: MathJax_Math-italic;">ω</span><span class="mi" id="MathJax-Span-302" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-303"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-304" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-305" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.29em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.289em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>L</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><mi>ω</mi><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-34">L = \frac{d\Phi}{d\omega dA^\perp}</script></span></td><td>通过单位面积单位立体角的辐射通量</td></tr><tr><td><strong>立体角</strong> (Solid Angle)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-35-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>&amp;#x03C9;</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-306" style="width: 0.803em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.66em, 1000.63em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-307"><span class="mi" id="MathJax-Span-308" style="font-family: MathJax_Math-italic;">ω</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 0.718em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>ω</mi></math></span></span><script type="math/tex" id="MathJax-Element-35">\omega</script></span></td><td>立体弧度，球面度（<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-36-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>s</mi><mi>r</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-309" style="width: 1.146em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.917em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.66em, 1000.92em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-310"><span class="mi" id="MathJax-Span-311" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-312" style="font-family: MathJax_Math-italic;">r</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 0.718em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>s</mi><mi>r</mi></math></span></span><script type="math/tex" id="MathJax-Element-36">sr</script></span>）</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-37-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>&amp;#x03C9;</mi><mo>=</mo><mfrac><mi>S</mi><msup><mi>r</mi><mn>2</mn></msup></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-313" style="width: 3.717em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.974em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1002.97em, 2.917em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-314"><span class="mi" id="MathJax-Span-315" style="font-family: MathJax_Math-italic;">ω</span><span class="mo" id="MathJax-Span-316" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-317" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.46em, 4.174em, -999.997em); top: -4.454em; left: 50%; margin-left: -0.226em;"><span class="mi" id="MathJax-Span-318" style="font-size: 70.7%; font-family: MathJax_Math-italic;">S<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.63em, 4.174em, -999.997em); top: -3.54em; left: 50%; margin-left: -0.283em;"><span class="msubsup" id="MathJax-Span-319"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.29em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-320" style="font-size: 70.7%; font-family: MathJax_Math-italic;">r</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.346em;"><span class="mn" id="MathJax-Span-321" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1000.75em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 0.746em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.639em; border-left: 0px solid; width: 0px; height: 1.861em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>ω</mi><mo>=</mo><mfrac><mi>S</mi><msup><mi>r</mi><mn>2</mn></msup></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-37">\omega=\frac{S}{r^2}</script></span></td><td>是二维弧度在三维的扩展，1 球面度等于单位球体的表面面积</td></tr></tbody></table>

辐射率被用来量化单一方向上发射来的光线的大小或者强度。辐射率是由多个物理变量集合而成的，它涉及的物理变量有以下几种：

*   **辐射通量 (Radiant Flux)**：辐射通量用符号$\Phi$表示，表示一个光源输出的能量，以瓦特为单位。光是由多种不同波长的能量集合而成，每种波长与一种特定的（可见的）颜色相关。因此一个光源所放射出来的能量可以被视作这个光源包含的所有各种波长的一个函数。波长介于 390nm（纳米）到 700nm 的光被认为是处于可见光光谱中，也就是说它们是人眼可见的波长。
    
    ![](1679148476662.png)
    
      
    _上图展示了太阳光中不同波长的光所具有的能量。_  
    传统物理学上的辐射通量将会计算这个由不同波长构成的函数的总面积，这种计算很复杂，耗费大量性能。在 PBR 技术中，不直接使用波长的强度，而是使用三原色编码（RGB）来简化辐射通量的计算。虽然这种简化会带来一些信息上的损失，但是这对于视觉效果上的影响基本可以忽略。
    
*   **立体角 (Solid Angle)**：用符号$\omega$表示，它描述投射到单位球体上的一个截面的大小或者面积。可以把立体角想象成为一个带有体积的方向：  
    
    ![](1679148476688.png)
    
      
    更加形象地描述：观察者站在单位球面的中心，向着投影的方向看，在单位球体面上的投影轮廓的大小就是立体角。
    
*   **辐射强度 (Radiant Intensity)**：用符号 $I$表示，它描述的是在单位球面上，一个光源向每单位立体角所投送的辐射通量。举个例子，假设一个点光源向所有方向均匀地辐射能量，辐射强度就能计算出它在一个单位面积（立体角）内的能量大小：  
    
    ![](1679148476720.png)
    
      
    计算辐射强度的公式：
    

$$I = \frac{d\Phi}{d\omega}$$

其中 $I$表示辐射通量$\Phi$除以立体角$\omega$的辐射强度。

理解以上物理变量后，可以继续讨论辐射率方程了。下面方程代表的意义是：一个辐射强度为$\Phi$的光通过立体角$\omega$辐射在区域 $A$的可被观察到的总能量。

$$L=\frac{I}{dA^\perp}=\frac{\frac{d\Phi}{d\omega}}{dA\cos\theta}=\frac{d\Phi}{ dA d\omega \cos\theta}$$

笔者注：原文的公式是 $L = \frac{d^2\Phi}{ dA d\omega \cos\theta}$，经推导之后，并没有平方。​

![](1679148476742.png)

辐射率是一个区域内光照量的辐射学度量，按照光的入射（或者来源）角与平面法线的夹角$\theta$计算 $\cos \theta$。越是斜着照射在平面上光越弱，反之越是垂直照射在表面上的光越强，类似基础光照中的漫反射颜色计算，$\cos \theta$直接等于光的方向和表面法线的点积。

```
float cosTheta = dot(lightDir, N);
```

上面的物理符号似乎和 PBR 的反射方程没有直接的关系。但是，如果将立体角$\omega$跟区域 $A$都看作无限小，就可以使用辐射率来分析一束光线打在空间上一个点的通量，也就是说能够计算单束光线对单个（片元）点的辐射率影响。进一步地，将立体角$\omega$转化为方向向量$\omega$，将区域 $A$转化成点 $p$，因此在 shader 中直接使用辐射率来计算单束光线对每个片元的贡献。

实际上，当谈及光的辐射率时，通常只关注的是所有射入点 $p$的光线，这些光的辐射度总和称为**辐照度 (Irradiance)**。理解了辐射率和辐照度，回到反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} f_r(p,\omega_i,\omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

渲染方程式中 $L$代表某个点 $p$的辐射率，而无限小的入射光的立体角$\omega_i$可以看作入射光方向向量$\omega_i$，将用来衡量入射光与平面法线夹角对能量的影响的 $\cos \theta$分量移出辐射率方程，作为反射方程的单独项 $n \cdot \omega_i$ 。

反射方程计算了点 $p$在所有视线方向$\omega_0$上被反射出来的辐射率 $L_o(p,\omega_o)$的总和。换言之：$L_0$计算的是在$\omega_o$方向的眼睛观察到的 $p$点的总辐照度。

反射方程里面使用的辐照度，必须要包含所有以 $p$点为中心的半球$\Omega$内的入射光，而不单单只是某一个方向的入射光。这个半球指的是围绕面法线 $n$的那一个半球：  

![](1679148476772.png)

笔者注：为什么只计算半球而不计算整个球体呢？

因为另外一边的半球因与视线方向相反，不能被观察，也就是辐射通量贡献量为 0，所以被忽略。

为了计算这个区域（半球）内的所有值，在反射方程中使用了一个称作为积分的数学符号 $\int$，来计算半球$\Omega$内所有的入射向量 $d\omega_i$。

积分计算面积的方法，有**解析 (analytically)** 和**渐近 (numerically)** 两种方法。目前尚没有可以满足渲染计算的解析法，所以只能选择离散渐近法来解决这个积分问题。

具体做法是在半球$\Omega$按一定的步长将反射方程离散地求解，然后再按照步长大小将所得到的结果平均化，这种方法被称为**黎曼和 (Riemann sum)**。下面是实现的伪代码：

```
int steps = 100; // 分段计算的数量，数量越多，计算结果越准确。
float dW  = 1.0f / steps;
vec3 P    = ...;
vec3 Wo   = ...;
vec3 N    = ...;
float sum = 0.0f;
for(int i = 0; i < steps; ++i) 
{
    vec3 Wi = getNextIncomingLightDir(i);
    sum += Fr(P, Wi, Wo) * L(P, Wi) * dot(N, Wi) * dW;
}
```

`dW`的值越小结果越接近正确的积分函数的面积或者说体积，衡量离散步长的`dW`可以看作反射方程中的 $d\omega_i$。积分计算中我们用到的 $d\omega_i$是线性连续的符号，跟代码中的`dW`并没有直接关系，但是这种方式有助于我们理解，而且这种离散渐近的计算方法总是可以得到一个很接近正确结果的值。值得一提的是，通过增加步骤数`steps`可以提高黎曼和的准确性，但计算量也会增大。

反射方程加了所有的，以各个方向$\omega_i$射入半球$\Omega$并打中点 $p$的入射光，经过反射函数 $f_r$进入观察者眼睛的所有反射光 $L_o$的辐射率之和。入射光辐射度可以由光源处获得，此外还可以利用一个环境贴图来测算所有入射方向上的辐射度。

至此，反射方程中，只剩下 $f_r$项未描述。$f_r$就是**双向反射分布函数** (Bidirectional Reflectance Distribution Function, BRDF)，它的作用是基于表面材质属性来对入射辐射度进行缩放或者加权。

### **3.1.4 双向反射分布函数（BRDF）**

**双向反射分布函数**（Bidirectional Reflectance Distribution Function，BRDF）是一个使用入射光方向$\omega_i$作为输入参数的函数，输出参数为出射光$\omega_o$，表面法线为 $n$，参数 $a$表示的是微平面的粗糙度。

BRDF 函数是近似的计算在一个给定了属性的不透明表面上每个单独的光线对最终的反射光的贡献量。假如表面是绝对光滑的（比如镜子），对于所有入射光$\omega_i$的 BRDF 函数都将会返回 0.0，除非出射光线$\omega_o$方向的角度跟入射光线$\omega_i$方向的角度以面法线为中轴线完全对称，则返回 1.0。

BRDF 对于材质的反射和折射属性的模拟基于之前讨论过的微平面理论，想要 BRDF 在物理上是合理的，就必须遵守能量守恒定律。比如反射光能量总和永远不应该超过入射光。技术上来说，Blinn-Phong 光照模型跟 BRDF 一样使用了$\omega_i$跟$\omega_o$作为输入参数，但是没有像基于物理的渲染这样严格地遵守能量守恒定律。

BRDF 有好几种模拟表面光照的算法，然而，基本上所有的**实时**渲染管线使用的都是 **Cook-Torrance BRDF**。

Cook-Torrance BRDF 分为漫反射和镜面反射两个部分：

$$f_r = k_d f_{lambert} + k_s f_{cook-torrance}$$

其中 $k_d$是入射光中被折射的比例，$k_s$是另外一部分被镜面反射的入射光。BRDF 等式左边的 $f_{lambert}$表示的是漫反射部分，这部分叫做伦勃朗漫反射（Lambertian Diffuse）。它类似于我们之前的漫反射着色，是一个恒定的算式：

$$f_{lambert} = \frac{c}{\pi}$$

其中 $c$代表的是 Albedo 或表面颜色，类似漫反射表面纹理。除以$\pi$是为了规格化漫反射光，为后期的 BRDF 积分做准备。

此处的伦勃朗漫反射跟以前用的漫反射之间的关系：以前的漫反射是用表面的漫反射颜色乘以法线与面法线的点积，这个点积依然存在，只不过是被移到了 BRDF 外面，写作 $n \cdot \omega_i$，放在反射方程 $L_o$靠后的位置。

BRDF 的高光（镜面反射）部分更复杂：

$$f_{cook-torrance} = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

Cook-Torrance 镜面反射 BRDF 由 3 个函数（$D$，$F$，$G$）和一个标准化因子构成。$D$，$F$，$G$符号各自近似模拟了特定部分的表面反射属性：

*   **$D$(Normal _D_istribution Function，NDF)**：法线分布函数，估算在受到表面粗糙度的影响下，取向方向与中间向量一致的微平面的数量。这是用来估算微平面的主要函数。
*   **$F$(_F_resnel equation)**：菲涅尔方程，描述的是在不同的表面角下表面反射的光线所占的比率。
*   **$G$(_G_eometry function)**：几何函数，描述了微平面自成阴影的属性。当一个平面相对比较粗糙的时候，平面表面上的微平面有可能挡住其他的微平面从而减少表面所反射的光线。

以上的每一种函数都是用来估算相应的物理参数的，而且你会发现用来实现相应物理机制的每种函数都有不止一种形式。它们有的非常真实，有的则性能高效。你可以按照自己的需求任意选择自己想要的函数的实现方法。

Epic Games 公司的 Brian Karis 对于这些函数的多种近似实现方式进行了大量的研究。这里将采用 Epic Games 在 Unreal Engine 4 中所使用的函数，其中 $D$使用 Trowbridge-Reitz GGX，$F$使用 Fresnel-Schlick 近似法 (Approximation)，而 $G$使用 Smith's Schlick-GGX。

#### **3.1.4.1 $D$(Normal _D_istribution Function，NDF)**

法线分布函数，从统计学上近似的表示了与某些（如中间）向量 $h$取向一致的微平面的比率。

目前有很多种 NDF 都可以从统计学上来估算微平面的总体取向度，只要给定一些粗糙度的参数以及一个我们马上将会要用到的参数 Trowbridge-Reitz GGX（GGXTR）：

$$NDF_{GGX TR}(n, h, \alpha) = \frac{\alpha^2}{\pi((n \cdot h)^2 (\alpha^2 - 1) + 1)^2}$$

这里的 $h$是用来测量微平面的半角向量，$\alpha$是表面的粗糙度，$n$是表面法线。 如果将 $h$放到表面法线和光线方向之间，并使用不同的粗糙度作为参数，可以得到下面的效果：  

![](1679148476814.png)

当粗糙度很低（表面很光滑）时，与中间向量 $h$取向一致的微平面会高度集中在一个很小的半径范围内。由于这种集中性，NDF 最终会生成一个非常明亮的斑点。但是当表面比较粗糙的时候，微平面的取向方向会更加的随机，与向量 $h$取向一致的微平面分布在一个大得多的半径范围内，但是较低的集中性也会让最终效果显得更加灰暗。

Trowbridge-Reitz GGX 的 NDF 实现代码：

```
float DistributionGGX(vec3 N, vec3 H, float a) {
    float a2     = a*a;
    float NdotH  = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;
	
    float nom    = a2;
    float denom  = (NdotH2 * (a2 - 1.0) + 1.0);
    denom        = PI * denom * denom;
	
    return nom / denom;
}
```

#### **3.1.4.2 $F$(_F_resnel equation)**

菲涅尔方程定义的是在不同观察方向上，表面上被反射的光除以被折射的光的比例。在一束光击中了表面的一瞬间，菲涅尔根据表面与观察方向之间的夹角，计算得到光被反射的百分比。根据这个比例和能量守恒定律我们可以直接知道剩余的能量就是会被折射的能量。

当我们垂直观察每个表面或者材质时都有一个基础反射率，当我们以任意一个角度观察表面时所有的反射现象都会变得更明显（反射率高于基础反射率）。你可以从你身边的任意一件物体上观察到这个现象，当你以 90 度角观察你的桌子你会法线反射现象将会变得更加的明显，理论上以完美的 90 度观察任意材质的表面都应该会出现全反射现象（所有物体、材质都有菲涅尔现象）。

菲涅尔方程同样是个复杂的方程，但是幸运的是菲涅尔方程可以使用 Fresnel-Schlick 来近似：

$$F_{Schlick}(h, v, F_0) = F_0 + (1 - F_0) ( 1 - (h \cdot v))^5$$

$F_0$表示的是表面基础反射率，这个我们可以使用一种叫做 **Indices of refraction(IOR)** 的方法计算得到。运用在球面上的效果就是你看到的那样，观察方向越是接近**掠射角**（grazing angle，又叫切线角，与正视角相差 90 度），菲涅尔现象导致的反射就越强：  

![](1679148476836.png)

菲涅尔方程中有几个微妙的地方，一个是 Fresnel-Schlick 算法仅仅是为电介质（绝缘体）表面定义的算法。对于金属表面，使用电介质的折射率来计算基础反射率是不合适的，我们需要用别的菲涅尔方程来计算。对于这个问题，我们需要预先计算表面在正视角 (即以 0 度角正视表面) 下的反应（$F_0$），然后就可以跟之前的 Fresnel-Schlick 算法一样，根据观察角度来进行插值。这样我们就可以用一个方程同时计算金属和电介质了。

表面在正视角下的反映或者说基础反射率可以在这个数据库中找到，下面是 Naty Hoffman 的在 SIGGRAPH 公开课中列举的一些常见材质的值：  

![](1679148476872.png)

这里可以观察到的一个有趣的现象，所有电介质材质表面的基础反射率都不会高于 0.17，这其实是例外而非普遍情况。导体材质表面的基础反射率起点更高一些并且（大多）在 0.5 和 1.0 之间变化。此外，对于导体或者金属表面而言基础反射率一般是带有色彩的，这也是为什么要用 RGB 三原色来表示的原因（法向入射的反射率可随波长不同而不同）。这种现象我们只能在金属表面观察的到。

金属表面这些和电介质表面相比所独有的特性引出了所谓的金属工作流的概念。也就是我们需要额外使用一个被称为金属度 (Metalness) 的参数来参与编写表面材质。金属度用来描述一个材质表面是金属还是非金属的。

通过预先计算电介质与导体的值，我们可以对两种类型的表面使用相同的 Fresnel-Schlick 近似，但是如果是金属表面的话就需要对基础反射率添加色彩。我们一般是按下面这个样子来实现的：

```
vec3 F0 = vec3(0.04);
F0      = mix(F0, surfaceColor.rgb, metalness);
```

我们为大多数电介质表面定义了一个近似的基础反射率。$F_0$取最常见的电解质表面的平均值，这又是一个近似值。不过对于大多数电介质表面而言使用 0.04 作为基础反射率已经足够好了，而且可以在不需要输入额外表面参数的情况下得到物理可信的结果。然后，基于金属表面特性，我们要么使用电介质的基础反射率要么就使用 $F_0$作来为表面颜色。因为金属表面会吸收所有折射光线而没有漫反射，所以我们可以直接使用表面颜色纹理来作为它们的基础反射率。

Fresnel Schlick 近似可以用 GLSL 代码实现：

```
vec3 fresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
```

其中`cosTheta`是表面法向量 $n$与观察方向 $v$的点乘的结果。

#### **3.1.4.3 $G$(_G_eometry function)**

几何函数模拟微平面相互遮挡导致光线的能量减少或丢失的现象。  

![](1679148476912.png)

类似 NDF，几何函数也使用粗糙度作为输入参数，更粗糙意味着微平面产生自阴影的概率更高。几何函数使用由 GGX 和 Schlick-Beckmann 组合而成的模拟函数 Schlick-GGX：

$$G_{SchlickGGX}(n, v, k) = \frac{n \cdot v} {(n \cdot v)(1 - k) + k }$$

这里的 $k$是使用粗糙度$\alpha$计算而来的，用于直接光照和 IBL 光照的几何函数的参数：

$$\begin{eqnarray*} k_{direct} &=& \frac{(\alpha + 1)^2}{8} \\ k_{IBL} &=& \frac{\alpha^2}{2} \end{eqnarray*}$$

需要注意的是这里$\alpha$的值取决于你的引擎怎么将粗糙度转化成$\alpha$，在接下来的教程中我们将会进一步讨论如何和在什么地方进行这个转换。

为了有效地模拟几何体，我们需要同时考虑两个视角，视线方向（几何遮挡）跟光线方向（几何阴影），我们可以用 **Smith 函数**将两部分放到一起：

$$G(n, v, l, k) = G_{sub}(n, v, k) G_{sub}(n, l, k)$$

其中 $v$表示视线向量，$G_{sub}(n, v, k)$表示视线方向的几何遮挡；$l$表示光线向量，$G_{sub}(n, l, k)$表示光线方向的几何阴影。使用 Smith 函数与 Schlick-GGX 作为 $G_{sub}$可以得到如下所示不同粗糙度 R 的视觉效果：  

![](1679148477023.png)

几何函数是一个值域为 [0.0, 1.0] 的乘数，其中白色 (1.0) 表示没有微平面阴影，而黑色 (0.0) 则表示微平面彻底被遮蔽。

使用 GLSL 编写的几何函数代码如下：

```
float GeometrySchlickGGX(float NdotV, float k) {
    float nom   = NdotV;
    float denom = NdotV * (1.0 - k) + k;
	
    return nom / denom;
}
  
float GeometrySmith(vec3 N, vec3 V, vec3 L, float k) {
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx1 = GeometrySchlickGGX(NdotV, k); // 视线方向的几何遮挡
    float ggx2 = GeometrySchlickGGX(NdotL, k); // 光线方向的几何阴影
	
    return ggx1 * ggx2;
}
```

#### **3.1.4.4 Cook-Torrance 反射方程 (Cook-Torrance reflectance equation)**

Cook-Torrance 反射方程中的每一个部分我们我们都用基于物理的 BRDF 替换，可以得到最终的反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

上面的方程并非完全数学意义上的正确。前面提到菲涅尔项 $F$代表光在表面的反射比率，它直接影响 $k_s$因子，意味着反射方程的镜面反射部分已经隐含了因子 $k_s$。因此，最终的 Cook-Torrance 反射方程如下（去掉了 $k_s$）：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

这个方程完整地定义了一个基于物理的渲染模型，也就是我们一般所说的基于物理的渲染（PBR）。

### **3.1.5 制作 PBR 材质**

对 PBR 数学模型有了基本了解之后，我们最后要讨论的是美工应该生成怎样的材质属性，让我们可以直接用在 PBR 渲染管线里。PBR 管线中需要的所有材质参数都可以使用纹理来定义或者模拟，使用纹理我们可以逐像素控制制定的面如何跟光线交互：这个点是否是金属，粗糙度如何又或者表面对不同波长的光有什么反映。

下面是在 PBR 渲染管线中经常用到的纹理：  

![](1679148477062.png)

下面的参数跟 [**2.4 PBR 在游戏引擎的应用**](#2-4-pbr%E5%9C%A8%E6%B8%B8%E6%88%8F%E5%BC%95%E6%93%8E%E7%9A%84%E5%BA%94%E7%94%A8)描述的很多参数基本一致。

*   **反射率**（Albedo）：反射率纹理指定了材质表面每个像素的颜色，如果材质是金属那纹理包含的就是基础反射率。这个跟我们之前用过的漫反射纹理非常的类似，但是不包含任何光照信息。漫反射纹理通常会有轻微的阴影和较暗的裂缝，这些在 Albedo 贴图里面都不应该出现，仅仅只包含材质的颜色（金属材质是基础反射率）。
    
*   **法线**（Normal）：法线纹理跟我们之前使用的是完全一样的。法线贴图可以逐像素指定表面法线，让平坦的表面也能渲染出凹凸不平的视觉效果。
    
*   **金属度**（Metallic）：金属度贴图逐像素的指定表面是金属还是电介质。根据 PBR 引擎各自的设定，金属程度即可以是 [0.0，1.0] 区间的浮点值也可以是非 0 即 1 的布尔值。
    
*   **粗糙度**（Roughness）：粗糙度贴图逐像素的指定了表面有多粗糙，粗糙度的值影响了材质表面的微平面的平均朝向，粗糙的表面上反射效果更大更模糊，光滑的表面更亮更清晰。有些 PBR 引擎用光滑度贴图替代粗糙度贴图，因为他们觉得光滑度贴图更直观，将采样出来的光滑度使用（1 - 光滑度）= 粗糙度 就能转换成粗糙度了。
    
*   **环境光遮挡**（Ambient Occlusion，AO）：AO 贴图为材质表面和几何体周边可能的位置，提供了额外的阴影效果。比如有一面砖墙，在两块砖之间的缝隙里 Albedo 贴图包含的应该是没有阴影的颜色信息，而让 AO 贴图来指定这一块需要更暗一些，这个地方光线更难照射到。AO 贴图在光照计算的最后一步使用可以显著的提高渲染效果，模型或者材质的 AO 贴图一般是在建模阶段手动生成的。
    

美术可以直接根据物体在真实世界里的物理属性，来设置和调整用于渲染的基于物理的材质。

基于物理的渲染管线最大的优势在于，材质的物理属性是不变的，无论环境光怎么样设置都能得到一个接近真实的渲染结果，这让美术的人生都变得美好了。

基于物理管线的材质可以很简单的移植到不同的渲染引擎，不管光照环境如何都能正确的渲染出一个自然的结果。

## **3.2 PBR 的光照实现**

3.1 章节阐述了 Cook-Torrance 反射方程的理论和公式意义。这节将探讨如何将前面讲到的理论转化成一个基于直接光照的渲染器：比如点光源，方向光和聚光灯。

### **3.2.1 辐照度计算**

3.1 章节解释了 Cook-Torrance 反射方程的大部分含义，但有一点未提及：具体要怎么处理场景中的辐照度（Irradiance，也就是辐射的总能量 $L$）？在计算机领域，场景的辐射率 $L$度量的是来自光源光线的辐射通量$\phi$穿过指定的立体角$\omega$，在这里我们假设立体角$\omega$无限小，小到辐射度衡量的是光源射出的一束经过指定方向向量的光线的通量。

有了这个假设，我们又要怎么将之融合到之前教程讲的光照计算里去呢？想象我们有一个辐射通量以 RGB 表示为（23.47, 21.31, 20.79）的点光源，这个光源的辐射强度等于辐射通量除以所有出射方向。当为平面上某个特定的点 $p$着色的时候，所有可能的入射光方向都会经过半球$\Omega$，但只有一个入射方向$\omega_i$是直接来自点光源的，又因为我们的场景中只包含有一个光源，且这个光源只是一个点，所以 $p$点所有其它的入射光方向的辐射率都应该是 0.  

![](1679148477087.png)

如果我们暂时不考虑点光源的距离衰减问题，且无论光源放在什么地方入射光线的辐射率都一样大（忽略入射光角度 $\cos \theta$对辐射度的影响），又因为点光源朝各个方向的辐射强度都是一样的，那么有效的辐射强度就跟辐射通量完全一样：恒定值（23.47, 21.31, 20.79）。

然而，辐射率需要使用位置 $p$作为输入参数，因为现实中的灯光根据点 $p$和光源之间距离的不同，辐射强度多少都会有一定的衰减。另外，从原始的辐射方程中我们可以发现，面法线 $n$于入射光方向向量$\omega_i$的点积也会影响结果。

用更精炼的话来描述：在点光源直接光照的情况里，辐射率函数 $L$计算的是灯光颜色，经过到 $p$点距离的衰减之后，再经过 $n \cdot \omega_i$缩放。能击中点 $p$的光线方向$\omega_i$就是从 $p$点看向光源的方向。把这些写成代码：

```
vec3  lightColor  = vec3(23.47, 21.31, 20.79);
vec3  wi          = normalize(lightPos - fragPos);
float cosTheta    = max(dot(N, Wi), 0.0);
// 计算光源在点fragPos的衰减系数
float attenuation = calculateAttenuation(fragPos, lightPos); 
// 英文原版的radiance类型有误，将它改成了vec3
vec3 radiance  = lightColor * (attenuation * cosTheta);
```

你应该非常非常熟悉这段代码：这就是以前我们计算漫反射光的算法！在只有单光源直接光照的情况下，辐射率的计算方法跟我们以前的光照算法是类似的。

要注意我们这里假设点光源无限小，只是空间中的一个点。如果我们使用有体积的光源模型，那么就有很多的入射光方向的辐射率是非 0 的。

对那些基于点的其他类型光源我们可以用类似的方法计算辐射率，比如平行光源的入射角的恒定的且没有衰减因子，聚光灯没有一个固定的辐射强度，而是围绕一个正前方向量来进行缩放的。

这也将我们带回了在表面半球$\Omega$的积分$\int$。我们知道，多个单一位置的光源对同一个表面的同一个点进行光照着色并不需要用到积分，我们可以直接拿出这些数目已知的光源来，分别计算这些光源的辐照度后再加到一起，毕竟每个光源只有一束方向光能影响物体表面的辐射率。这样只需要通过相对简单的循环计算每个光源的贡献就能完成整个 PBR 光照计算。当我们需要使用 IBL 将环境光加入计算的时候我们才会需要用到积分，因为环境光可能来自任何方向。

### **3.2.2 PBR 表面模型（ PBR surface model）**

我们先从写一个能满足前面讲到的 PBR 模型的片源着色器开始。首先，我们需要将表面的 PBR 相关属性输入着色器：

```
#version 330 core
out vec4 FragColor;
in vec2 TexCoords;
in vec3 WorldPos;
in vec3 Normal;
  
uniform vec3 camPos;
  
uniform vec3  albedo;
uniform float metallic;
uniform float roughness;
uniform float ao;
```

我们能从顶点着色器拿到常见的输入，另外一些是物体表面的材质属性。

在片源着色器开始的时候，我们先要做一些所有光照算法都需要做的计算：

```
void main()
{
    vec3 N = normalize(Normal); 
    vec3 V = normalize(camPos - WorldPos);
    [...]
}
```

#### **3.2.2.1 直接光照（Direct lighting）**

在这个教程的示例中，我们将会有 4 个点光源作为场景辐照度来源。为了满足反射方程我们循环处理每一个光源，计算它独自的辐射率，然后加总经过 BRDF 跟入射角缩放的结果。我们可以把这个循环当作是积分运算的一种实现方案。首先，计算每个光源各自相关参数：

```
vec3 Lo = vec3(0.0);
for(int i = 0; i < 4; ++i) 
{
    vec3 L = normalize(lightPositions[i] - WorldPos);
    vec3 H = normalize(V + L);
  
    float distance    = length(lightPositions[i] - WorldPos);
    float attenuation = 1.0 / (distance * distance);
    vec3 radiance     = lightColors[i] * attenuation; 
    [...] // 还有逻辑放在后面继续探讨，所以故意在for循环缺了‘}’。
```

由于我们是在线性空间进行的计算（在最后阶段处理 Gamma 校正），所以光源的衰减会更符合物理上的反平方律（inverse-square law）。

反平方律虽然物理学正确，但我们可能还会使用常量、线性、二次方程式来更好地控制光照衰减，即便这些衰减不是物理学正确的。

然后，我们对每个光源计算所有的 Cook-Torrance BRDF 分量：

$$\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

我们要做的第一件事是计算高光跟漫反射之间的比例，有多少光被反射出去了又有多少产生了折射。前面的教程我们讲到过这个菲涅尔方程：

```
vec3 fresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
```

Fresnel-Schlick 算法需要的`F0`参数就是我们之前说的基础反射率，即以 0 度角照射在表面上的光被反射的比例。不同材质的`F0`的值都不一样，可以根据材质到那张非常大的材质表里去找。在 PBR 金属度流水线中我们做了一个简单的假设，我们认为大部分的电介质表面的`F0`用 0.04 效果看起来很不错。而金属表面我们将`F0`放到 albedo 纹理内，这些可以写成代码如下：

```
vec3 F0 = vec3(0.04); 
F0      = mix(F0, albedo, metallic);
vec3 F  = fresnelSchlick(max(dot(H, V), 0.0), F0);
```

如上述代码所见，非金属的`F0`永远是 0.04，除非我们通过金属度属性在`F0`跟`albedo`之间进行线性插值，才能得到一个不同的非金属`F0`。

有了`F`，还剩下法线分布函数 $D$跟几何函数 $G$需要计算。

在直接光照的 PBR 光照着色器中它们等价于如下代码：

```
float DistributionGGX(vec3 N, vec3 H, float roughness) {
    float a      = roughness*roughness;
    float a2     = a*a;
    float NdotH  = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;
	
    float num   = a2;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    denom = PI * denom * denom;
	
    return num / denom;
}

float GeometrySchlickGGX(float NdotV, float roughness) {
    float r = (roughness + 1.0);
    float k = (r*r) / 8.0;

    float num   = NdotV;
    float denom = NdotV * (1.0 - k) + k;
	
    return num / denom;
}

float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness) {
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2  = GeometrySchlickGGX(NdotV, roughness);
    float ggx1  = GeometrySchlickGGX(NdotL, roughness);
	
    return ggx1 * ggx2;
}
```

这里值得注意的是，相较于 3.1 理论篇教程，我们直接传入了粗糙度参数进函数。这样我们就可以对原始粗糙度做一些特殊操作。根据迪斯尼的原则和 Epic Games 的用法，在法线分布函数跟几何函数中使用粗糙度的平方替代原始粗糙度进行计算光照效果会更正确一些。

当这些都定义好了之后，在计算 NDF 和 G 分量就是很简单的事情了：

```
float NDF = DistributionGGX(N, H, roughness);       
float G   = GeometrySmith(N, V, L, roughness);
```

然后就可以计算 Cook-Torrance BRDF 了：

```
vec3 numerator    = NDF * G * F;
float denominator = 4.0 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0);
vec3 specular     = numerator / max(denominator, 0.001);
```

`denominator`项里的`0.001`是为了防止除 0 情况而特意加上的。

到这里，我们终于可以计算每个光源对反射方程的贡献了。因为菲涅尔值相当于 $k_S$，可用`F`代表任意光击中表面后被反射的部分，根据能量守恒定律我们可以用 $k_S$直接计算得到 $k_D$：

```
vec3 kS = F;
vec3 kD = vec3(1.0) - kS;
  
kD *= 1.0 - metallic; // 由于金属表面不折射光，没有漫反射颜色，通过归零kD来实现这个规则
```

$k_S$表示的是光能有多少被反射了，剩下的被折射的光能我们用 $k_D$来表示。此外，由于金属表面不折射光，因此没有漫反射颜色，我们通过归零 $k_D$来实现这个规则。

有了这些数据，我们终于可以算出每个光源的出射光了：

```
const float PI = 3.14159265359;
  
    float NdotL = max(dot(N, L), 0.0);        
    Lo += (kD * albedo / PI + specular) * radiance * NdotL;
}
```

最终结果`Lo`，或者说出射辐射度（Radiosity），实际上是反射方程在半球$\Omega$的积分$\int$的结果。这里要特别注意的是，我们将 $k_S$移除方程式，是因为我们已经在 BRDF 中乘过菲涅尔参数`F`了，此处不需要再乘一次。

我们没有真正的对所有可能的入射光方向进行积分，因为我们已经清楚的知道只有 4 个入射方向可以影响这个片元，所以我们只需要直接用循环处理这些入射光就行了。

剩下的就是要将 AO 运用到光照结果`Lo`上，我们就可以得到这个片元的最终颜色了：

```
vec3 ambient = vec3(0.03) * albedo * ao;
vec3 color   = ambient + Lo;
```

#### **3.2.2.2 线性和 HDR 渲染（ Linear and HDR rendering）**

以上我们假设所有计算都在线性空间，为了使用这个结果我们还需要在着色器的最后进行**伽马校正（Gamma Correct）**，在线性空间计算光照对于 PBR 是非常非常重要的，所有输入参数同样要求是线性的，不考虑这一点将会得到错误的光照结果。

另外，我们希望输入的灯光参数更贴近实际的物理参数，比如他们的辐射度或者颜色值可以是一个非常宽广的值域。这样作为结果输出的`Lo`也将变得很大，如果我们不做处理默认会直接 Clamp 到 0.0 至 1.0 之间以适配低动态范围（LDR）输出方式。

为了有效解决`Lo`的值域问题，我们可以使用色调映射（Tone Map）和曝光控制（Exposure Map），用它们将`Lo`的高动态范围（HDR）映射到 LDR 之后再做伽马校正：

```
color = color / (color + vec3(1.0)); // 色调映射
color = pow(color, vec3(1.0/2.2)); 	 // 伽马校正
```

这里我们使用的是莱因哈特算法（Reinhard operator）对 HDR 进行 Tone Map 操作，尽量在伽马矫正之后还保持高动态范围。我们并没有分开帧缓冲或者使用后处理，所以我们可以直接将 Tone Mapping 和伽马矫正放在前向片元着色器（forward fragment shader）。  

![](1679148477347.png)

对于 PBR 渲染管线来说，线性空间跟高动态范围有着超乎寻常的重要性，没有这些就不可能绘制出不同灯光强度下的高光低光细节，错误的计算结果会产生难看的渲染效果。

#### **3.2.2.3 完整的 PBR 直接光照着色器**

现在唯一剩下的就是将最终的色调映射和伽玛校正的颜色传递给片元着色器的输出通道，我们就拥有了一个 PBR 直接光照着色器。基于完整性考虑，下面列出完整的`main`函数：

```
#version 330 core
out vec4 FragColor;
in vec2 TexCoords;
in vec3 WorldPos;
in vec3 Normal;

// material parameters
uniform vec3 albedo;
uniform float metallic;
uniform float roughness;
uniform float ao;

// lights
uniform vec3 lightPositions[4];
uniform vec3 lightColors[4];

uniform vec3 camPos;

const float PI = 3.14159265359;
// --------------------------------------------------------------------
float DistributionGGX(vec3 N, vec3 H, float roughness)
{
    float a = roughness*roughness;
    float a2 = a*a;
    float NdotH = max(dot(N, H), 0.0);
    float NdotH2 = NdotH*NdotH;

    float nom   = a2;
    float denom = (NdotH2 * (a2 - 1.0) + 1.0);
    denom = PI * denom * denom;

    return nom / max(denom, 0.001); // prevent divide by zero for roughness=0.0 and NdotH=1.0
}
// --------------------------------------------------------------------
float GeometrySchlickGGX(float NdotV, float roughness)
{
    float r = (roughness + 1.0);
    float k = (r*r) / 8.0;

    float nom   = NdotV;
    float denom = NdotV * (1.0 - k) + k;

    return nom / denom;
}
// --------------------------------------------------------------------
float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness)
{
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2 = GeometrySchlickGGX(NdotV, roughness);
    float ggx1 = GeometrySchlickGGX(NdotL, roughness);

    return ggx1 * ggx2;
}
// --------------------------------------------------------------------
vec3 fresnelSchlick(float cosTheta, vec3 F0)
{
    return F0 + (1.0 - F0) * pow(1.0 - cosTheta, 5.0);
}
// --------------------------------------------------------------------
void main()
{		
    vec3 N = normalize(Normal);
    vec3 V = normalize(camPos - WorldPos);

    // calculate reflectance at normal incidence; if dia-electric (like plastic) use F0 
    // of 0.04 and if it's a metal, use the albedo color as F0 (metallic workflow) 
    vec3 F0 = vec3(0.04); 
    F0 = mix(F0, albedo, metallic);

    // reflectance equation
    vec3 Lo = vec3(0.0);
    for(int i = 0; i < 4; ++i) 
    {
        // calculate per-light radiance
        vec3 L = normalize(lightPositions[i] - WorldPos);
        vec3 H = normalize(V + L);
        float distance = length(lightPositions[i] - WorldPos);
        float attenuation = 1.0 / (distance * distance);
        vec3 radiance = lightColors[i] * attenuation;

        // Cook-Torrance BRDF
        float NDF = DistributionGGX(N, H, roughness);   
        float G   = GeometrySmith(N, V, L, roughness);      
        vec3 F    = fresnelSchlick(clamp(dot(H, V), 0.0, 1.0), F0);
           
        vec3 nominator    = NDF * G * F; 
        float denominator = 4 * max(dot(N, V), 0.0) * max(dot(N, L), 0.0);
        vec3 specular = nominator / max(denominator, 0.001); // prevent divide by zero for NdotV=0.0 or NdotL=0.0
        
        // kS is equal to Fresnel
        vec3 kS = F;
        // for energy conservation, the diffuse and specular light can't
        // be above 1.0 (unless the surface emits light); to preserve this
        // relationship the diffuse component (kD) should equal 1.0 - kS.
        vec3 kD = vec3(1.0) - kS;
        // multiply kD by the inverse metalness such that only non-metals 
        // have diffuse lighting, or a linear blend if partly metal (pure metals
        // have no diffuse light).
        kD *= 1.0 - metallic;	  

        // scale light by NdotL
        float NdotL = max(dot(N, L), 0.0);        

        // add to outgoing radiance Lo
        Lo += (kD * albedo / PI + specular) * radiance * NdotL;  // note that we already multiplied the BRDF by the Fresnel (kS) so we won't multiply by kS again
    }   
    
    // ambient lighting (note that the next IBL tutorial will replace 
    // this ambient lighting with environment lighting).
    vec3 ambient = vec3(0.03) * albedo * ao;

    vec3 color = ambient + Lo;

    // HDR tonemapping
    color = color / (color + vec3(1.0));
    // gamma correct
    color = pow(color, vec3(1.0/2.2)); 

    FragColor = vec4(color, 1.0);
}
```

希望在学习了前面教程的反射方程的理论知识之后，这个 shader 不再会让大家苦恼。使用这个 shader，4 个点光源照射在金属度和粗糙度不同的球上的效果大概类似这样：  

![](1679148477380.png)

从下往上金属度的值从 0.0 到 1.0，粗糙度从左往右从 0.0 增加到 1.0。可以通过观察小球之间的区别理解金属度和粗糙度参数的作用。

示例的源码可以从 [LearnOpenGL 的网站](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/1.1.lighting/lighting.cpp)找到。

### **3.2.3 使用纹理的 PBR（Textured PBR）**

3.2.2.3 小节的 PBR 实现中，部分重要的表面材质属性是`float`类型：

```
uniform float metallic;
uniform float roughness;
uniform float ao;
```

实际上，可以将它们用纹理代替，使用纹理的 PBR 可以更加精确地控制表面材质的细节，使得渲染效果更佳。Unity 支持这种方法。

为了实现逐像素的控制材质表面的属性我们必须使用纹理替代单个的材质参数：

```
[...]
uniform sampler2D albedoMap;
uniform sampler2D normalMap;
uniform sampler2D metallicMap;
uniform sampler2D roughnessMap;
uniform sampler2D aoMap;
  
void main()
{
    vec3 albedo     = pow(texture(albedoMap, TexCoords).rgb, 2.2);
    vec3 normal     = getNormalFromNormalMap();
    float metallic  = texture(metallicMap, TexCoords).r;
    float roughness = texture(roughnessMap, TexCoords).r;
    float ao        = texture(aoMap, TexCoords).r;
    [...]
}
```

要注意美术制作的 albedo 纹理一般都是 sRGB 空间的，因此我们要先转换到线性空间再进行后面的计算。根据美术资源的不同，AO 纹理也许同样需要从 sRGB 转换到线性空间。

将前面那些小球的材质属性替换成纹理之后，对比以前用的光照算法，PBR 有了一个质的提升：  

![](1679148477400.png)

可以在这里找到带纹理的 [Demo 源码](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/1.2.lighting_textured/lighting_textured.cpp)，所有用到的纹理在[这里](http://freepbr.com/materials/rusted-iron-pbr-metal-material-alt/)（用了白色的 AO 贴图）。记住金属表面在直接光照环境中更暗是因为他们没有漫反射。在环境使用环境高光进行光照计算的情况下看起来也是正常的，这个我们在下一个教程里再说。

这里没有其他 PBR 渲染示例中那样令人惊艳的效果，因为我们还没有加入基于图片的光照（Image Based Lighting）技术。尽管如此，这个 shader 任然算是一个基于物理的渲染，即使没有 IBL 你也可以法线光照看起来真实了很多。

## **3.3 基于图像的光照（Image Based Lighting，IBL）**

基于图像的光照（IBL）是对光源物体的技巧集合，与直接光照不同，它将周围环境当成一个大光源。IBL 通常结合 cubemap 环境贴图，cubemap 通常采集自真实的照片或从 3D 场景生成，这样可以将其用于光照方程：将 cubemap 的每个像素当成一个光源。这样可以更有效地捕获全局光照和常规感观，使得被渲染的物体更好地融入所处的环境中。

当基于图像的光照算法获得一些（全局的）环境光照时，它的输入被当成更加精密形式的环境光照，甚至是一种粗糙的全局光照的模拟。这使得 IBL 有助于 PBR 的渲染，使得物体渲染效果更真实。

在介绍 IBL 结合 PBR 之前，先回顾一下反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

如之前所述，我们的主目标是解决所有入射光 $w_i$通过半球$\Omega$的积分$\int$。与直接光照不同的是，在 IBL 中，**每一个**来自周围环境的入射光$\omega_i$都可能存在辐射，这些辐射对解决积分有着重要的作用。为解决积分有两个要求：

*   需要用某种方法获得给定任意方向向量$\omega_i$的场景辐射。
*   解决积分需尽可能快并实时。

对第一个要求，相对简单，采用环境 cubemap。给定一个 cubemap，可以假设它的每个像素是一个单独的发光光源。通过任意方向向量$\omega_i$采样 cubemap，可以获得场景在这个方向的辐射。

获取任意方向向量$\omega_i$的场景辐射很简单，如下：

```
vec3 radiance = texture(_cubemapEnvironment, w_i).rgb;
```

对要求二，解决积分能只考虑一个方向的辐射，要考虑环境贴图的半球$\Omega$的所有可能的方向$\omega_i$，但常规积分方法在片元着色器中开销非常大。为了有效解决积分问题，可采用预计算或预处理的方法。因此，需要深究一下反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

可将上述的 $k_d$和 $k_s$项拆分：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i+ \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

拆分后，可分开处理漫反射和镜面反射的积分。先从漫反射积分开始。

### **3.3.1 漫反射辐照度（Diffuse irradiance）**

仔细分析上面方程的漫反射积分部分，发现 Lambert 漫反射是个常量项（颜色 $c$，折射因子 $k_d$和$\pi$）并且不依赖积分变量。因此，可见常量部分移出漫反射积分：

$$L_o(p,\omega_o) = k_d\frac{c}{\pi} \int\limits_{\Omega} L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

因此，积分只依赖$\omega_i$（假设 $p$在环境贴图的中心）。据此，可以计算或预计算出一个新的 cubemap，这个 cubemap 存储了用**卷积**（convolution）计算出的每个采样方向（或像素）$\omega_o$的漫反射积分结果。

**卷积**（convolution）是对数据集的每个入口应用一些计算，假设其它所有的入口都在这个数据集里。此处的数据集就是场景辐射或环境图。因此，对 cubemap 的每个采样方向，我们可以顾及在半球$\Omega$的其它所有的采样方向。

为了卷积环境图，我们要解决每个输出$\omega_o$采样方向的积分，通过离散地采样大量的在半球$\Omega$的方向$\omega_i$并取它们辐射的平均值。采样方向$\omega_i$的半球是以点 $p$为中心以$\omega_o$为法平面的。  

![](1679148477451.png)

这个预计算的为每个采样方向$\omega_o$存储了积分结果的 cubemap，可被当成是预计算的在场景中所有的击中平行于$\omega_o$表面的非直接漫反射的光照之和。这种 cubemap 被称为**辐照度图（Irradiance map）**。

辐射方程依赖于位置 $p$，假设它在辐照度图的中心。这意味着所有非直接漫反射光需来自于同一个环境图，它可能打破真实的幻觉（特别是室内）。渲染引擎用放置遍布场景的反射探头（reflection probe）来解决，每个反射探头计算其所处环境的独自的辐照度图。这样，点 p 的辐射率（和辐射）是与其最近的反射探头的辐照度插值。这里我们假设总是在环境图的中心采样。反射探头将在其它章节探讨。

下面是 cubemap 环境图 (下图左）和对应的辐照度图（下图右）：  

![](1679148477478.png)

通过存储每个 cubemap 像素卷积的结果，辐照度图有点像环境的平均颜色或光照显示。从这个环境图采样任意方向，可获得这个方向的场景辐照度。

#### **3.3.1.1 球体图（Equirectangular map）**

球体图（Equirectangular map）有些文献翻译成全景图，它与 cubemap 不一样的是：cubemap 需要 6 张图，而球体图只需要一张，并且存储的贴图有一定形变：  

![](1679148477522.png)

cubemap 是可以通过一定算法转成球体图的，详见[这里](https://stackoverflow.com/questions/34250742/converting-a-cubemap-into-equirectangular-panorama)。

#### **3.3.1.2 从球体图到立方体图**

直接从球体图采样出环境光照信息是可能的，但它的开销远大于直接采样立方体图（cubemap）。因此，需要将球体图先转成立方体图，以便更好地实现后面的逻辑。当然，这里也会阐述如何从作为 3D 环境图的球体图采样，以便大家有更多的选择权。

为了将球体图映射到立方体图，首先需要构建一个立方体模型，渲染这个立方体模型的顶点着色器如下：

```
#version 330 core
layout (location = 0) in vec3 aPos;

out vec3 localPos;

uniform mat4 projection;
uniform mat4 view;

void main()
{
    localPos = aPos;  
    gl_Position =  projection * view * vec4(localPos, 1.0);
}
```

在像素着色器中，将会对变形的球体图的每个部位映射到立方体的每一边，具体实现如下：

```
#version 330 core
out vec4 FragColor;
in vec3 localPos;

uniform sampler2D equirectangularMap;

const vec2 invAtan = vec2(0.1591, 0.3183);
vec2 SampleSphericalMap(vec3 v)
{
    vec2 uv = vec2(atan(v.z, v.x), asin(v.y));
    uv *= invAtan;
    uv += 0.5;
    return uv;
}

void main()
{
    // make sure to normalize localPos
    vec2 uv = SampleSphericalMap(normalize(localPos)); 
    vec3 color = texture(equirectangularMap, uv).rgb;
    
    FragColor = vec4(color, 1.0);
}
```

渲染出来的立方体效果如下：  

![](1679148477573.png)

对于立方体图的采样，顶点着色器如下：

```
#version 330 core
layout (location = 0) in vec3 aPos;

uniform mat4 projection;
uniform mat4 view;

out vec3 localPos;

void main()
{
    localPos = aPos;

    // remove translation from the view matrix
    mat4 rotView = mat4(mat3(view)); 
    vec4 clipPos = projection * rotView * vec4(localPos, 1.0);

    gl_Position = clipPos.xyww;  // 注意这里的分量是`xyww`！！
}
```

对于立方体图的采样，像素着色器如下：

```
#version 330 core
out vec4 FragColor;

in vec3 localPos;
  
uniform samplerCube environmentMap;
  
void main()
{
    // 从cubemap采样颜色
    vec3 envColor = texture(environmentMap, localPos).rgb;
    
    // HDR -> LDR
    envColor = envColor / (envColor + vec3(1.0));
    // Gamma校正（只在颜色为线性空间的渲染管线才需要）
    envColor = pow(envColor, vec3(1.0/2.2)); 
  
    FragColor = vec4(envColor, 1.0);
}
```

上述代码中，要注意在输出最终的颜色之前，做了 HDR 到 LDR 的转换和 Gamma 校正。

渲染的效果如下图：  

![](1679148477687.png)

#### **3.3.1.3 PBR 和非直接辐射度光照（indirect irradiance lighting）**

辐射度图提供了漫反射部分的积分，该积分表示来自非直接的所有方向的环境光辐射之和。由于辐射度图被当成是无方向性的光源，所以可以将漫反射镜面反射合成环境光。

首先，得声明预计算出的辐射度图的 sample：

```
uniform samplerCube irradianceMap;
```

通过表面的法线，获得环境光可以简化成下面的代码：

```
// vec3 ambient = vec3(0.03);
vec3 ambient = texture(irradianceMap, N).rgb;
```

尽管如此，在之前所述的反射方程中，非直接光依旧包含了漫反射和镜面反射两个部分，所以我们需要加个权重给漫反射。下面采用了菲涅尔方程来计算漫反射因子：

```
vec3 kS = fresnelSchlick(max(dot(N, V), 0.0), F0);
vec3 kD = 1.0 - kS;
vec3 irradiance = texture(irradianceMap, N).rgb;
vec3 diffuse    = irradiance * albedo;
vec3 ambient    = (kD * diffuse) * ao;
```

由于环境光来自在半球内所有围绕着法线`N`的方向，没有单一的半向量去决定菲涅尔因子。为了仍然能模拟菲涅尔，这里采用了法线和视线的夹角。之前的算法采用了受表面粗糙度影响的微平面半向量，作为菲涅尔方程的输入。这里，我们并不考虑粗糙度，表面的反射因子被视作相当大。

非直接光照将沿用直接光照的相同的属性，所以，期望越粗糙的表面镜面反射越少。由于不考虑表面粗糙度，非直接光照的菲涅尔方程强度被视作粗糙的非金属表面（下图）。  

![](1679148477738.png)

为了缓解这个问题，可在 Fresnel-Schlick 方程注入粗糙度项（该方程的[来源](https://seblagarde.wordpress.com/2011/08/17/hello-world/)）：

```
vec3 fresnelSchlickRoughness(float cosTheta, vec3 F0, float roughness) {
    return F0 + (max(vec3(1.0 - roughness), F0) - F0) * pow(1.0 - cosTheta, 5.0);
}
```

考虑了表面粗糙度后，菲涅尔相关计算最终如下：

```
vec3 kS = fresnelSchlickRoughness(max(dot(N, V), 0.0), F0, roughness); 
vec3 kD = 1.0 - kS;
vec3 irradiance = texture(irradianceMap, N).rgb;
vec3 diffuse    = irradiance * albedo;
vec3 ambient    = (kD * diffuse) * ao;
```

如上所述，实际上，基于图片的光照计算非常简单，只需要单一的 cubemap 纹理采样。大多数的工作在于预计算或卷积环境图到辐射度图。

加入了 IBL 的渲染效果如下（竖向是金属度增加，水平是粗糙度增加）：  

![](1679148477776.png)

本节所有代码可在[这里](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.1.2.ibl_irradiance/ibl_irradiance.cpp)找到。

### **3.3.2 镜面的 IBL（Specular IBL）**

3.3.1 描述的是 IBL 的漫反射部分，本节将讨论 IBL 的镜面反射部分先回顾一下反射方程：

$$L_o(p,\omega_o) = \int\limits_{\Omega} (k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

上述的镜面反射部分（被 $k_s$相乘）不是恒定的，并且依赖于入射光方向和视线入射方向，尝试实时地计算所有入射光和所有入射视线的积分是几乎不可能的。Epic Games 推荐折中地使用预卷积镜面反射部分的方法来解决实时渲染的性能问题，这就是**分裂和近似法（split sum approximation）**。

**分裂和近似法**将镜面反射部分从反射方程分离出两个部分，这样可以单独地对它们卷积，后面在 PBR 的 shader 中为镜面的非直接 IBL 将它们结合起来。跟预卷积辐射度图类似，分裂和近似法需要 HDR 环境图作为输入。为了更好地理解分裂和近似法，下面着重关注反射方程的镜面部分：

$$\begin{eqnarray*} L_o(p,\omega_o) & = & \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)} L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i \end{eqnarray*}$$

出于跟辐射度图相同的性能问题的考虑，我们要预计算类似镜面 IBL 图的积分，并且用片元的法线采样这个图。辐射度图的预计算只依赖于$\omega_i$，并且我们可以将漫反射项移出积分。但这次从 BRDF 可以看出，不仅仅是依赖于$\omega_i$：

$$f_r(p, w_i, w_o) = \frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)}$$

如上方程所示，还依赖$\omega_o$，并且我们不能用两个方向向量来采样预计算的 cubemap。预计算所有$\omega_i$和$\omega_o$的组合在实时渲染环境中不实际的。

Epic Games 的分裂和近似法将镜面反射部分从反射方程分离出两个部分，这样可以单独地对它们卷积，后面在 PBR 的 shader 中为镜面的非直接 IBL 将它们结合起来。分离后的方程如下：

$$\begin{eqnarray*} L_o(p,\omega_o) & = & \int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)} L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i \\ & = & \int\limits_{\Omega} L_i(p,\omega_i) d\omega_i * \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d\omega_i \end{eqnarray*}$$

第一部分$\int\limits_{\Omega} L_i(p,\omega_i) d\omega_i$是**预过滤环境图（pre-filtered environment map）**，类似于辐射度图的预计算环境卷积图，但会加入粗糙度。随着粗糙度等级的增加，环境图使用更多的散射采样向量来卷积，创建出更模糊的反射。

对每个卷积的粗糙度等级，循环地在预过滤环境图的 mimap 等级存储更加模糊的结果。下图是 5 个不同粗糙度等级的预过滤环境图：  

![](1679148477798.png)

生成采样向量和它们的散射强度，需要用到 Cook-Torrance BRDF 的法线分布图（NDF），而其带了两个输入：法线和视线向量。当卷积环境图时并不知道视线向量，Epic Games 用了更近一步的模拟法：假设视线向量（亦即镜面反射向量）总是等于输出采样向量$\omega_o$。所以代码变成如下所示：

```
vec3 N = normalize(w_o);
vec3 R = N;
vec3 V = R;
```

这种方式预过滤环境图卷积不需要关心视线方向。这就意味着当从某个角度看向下面这张图的镜面表面反射时，无法获得很好的掠射镜面反射（grazing specular reflections）。然而通常这被认为是一个较好的妥协：  

![](1679148477844.png)

第二部分$\int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d\omega_i$是**镜面积分**。假设所有方向的入射辐射率是全白的（那样 $L(p, x) = 1.0$），那就可以用给定的粗糙度和一个法线 $n$和光源方向$\omega_i$之间的角度或 $n \cdot \omega_i$来预计算 BRDF 的值。Epic Games 存储了用变化的粗糙度来预计算每一个法线和光源方向组合的 BRDF 的值，该粗糙度存储于 2D 采样纹理（LUT）中，它被称为 **BRDF 积分图（BRDF integration map）**。

2D 采样纹理输出一个缩放（红色）和一个偏移值（绿色）给表面的菲涅尔方程式（Fresnel response），以便提供第二部分的镜面积分：  

![](1679148477893.png)

  
上图水平表示 BRDF 的输入 $n \cdot \omega_i$，竖向表示输入的粗糙度。

有了预过滤环境图和 BRDF 积分图，可以在 shader 中将它们结合起来：

```
float lod             = getMipLevelFromRoughness(roughness);
vec3 prefilteredColor = textureCubeLod(PrefilteredEnvMap, refVec, lod);
vec2 envBRDF          = texture2D(BRDFIntegrationMap, vec2(NdotV, roughness)).xy;
vec3 indirectSpecular = prefilteredColor * (F * envBRDF.x + envBRDF.y)
```

### **3.3.3 完整的 IBL**

首先是声明 IBL 镜面部分的两个纹理采样器：

```
uniform samplerCube prefilterMap;
uniform sampler2D   brdfLUT;
```

接着用法线`N`和视线`-V`算出反射向量`R`，再结合`MAX_REFLECTION_LOD`和粗糙度等参数采样预过滤环境图：

```
void main()
{
    [...]
    vec3 R = reflect(-V, N);   

    const float MAX_REFLECTION_LOD = 4.0;
    vec3 prefilteredColor = textureLod(prefilterMap, R,  roughness * MAX_REFLECTION_LOD).rgb;    
    [...]
}
```

然后用视线、法线的夹角及粗糙度采样 BRDF 查找纹理，结合预过滤环境图的颜色算出 IBL 的镜面部分：

```
vec3 F        = FresnelSchlickRoughness(max(dot(N, V), 0.0), F0, roughness);
vec2 envBRDF  = texture(brdfLUT, vec2(max(dot(N, V), 0.0), roughness)).rg;
vec3 specular = prefilteredColor * (F * envBRDF.x + envBRDF.y);
```

自此，反射方程的非直接的镜面部分已经算出来了。可以将它和上一小节的 IBL 的漫反射部分结合起来：

```
vec3 F = FresnelSchlickRoughness(max(dot(N, V), 0.0), F0, roughness);

vec3 kS = F;
vec3 kD = 1.0 - kS;
kD *= 1.0 - metallic;	  
  
vec3 irradiance = texture(irradianceMap, N).rgb;
vec3 diffuse    = irradiance * albedo;
  
const float MAX_REFLECTION_LOD = 4.0;
vec3 prefilteredColor = textureLod(prefilterMap, R,  roughness * MAX_REFLECTION_LOD).rgb;   
vec2 envBRDF  = texture(brdfLUT, vec2(max(dot(N, V), 0.0), roughness)).rg;
vec3 specular = prefilteredColor * (F * envBRDF.x + envBRDF.y);
  
vec3 ambient = (kD * diffuse + specular) * ao;
```

此时可以算出由 IBL 的漫反射和镜面反射部分结合而成的环境光`ambient`，渲染效果如下：  

![](1679148477938.png)

扩展一下，加入一些酷酷的[材质](https://freepbr.com/)：  

![](1679148477981.png)

或者加载[这些极好又免费的 PBR 3D 模型](http://artisaverb.info/PBT.html)（by Andrew Maximov）：  

![](1679148478025.png)

非常肯定地，加了 IBL 光照后，渲染效果更真实更加物理正确。下图展示了在未改变任何光照信息的情况下，在不同的预计算 HDR 图中的效果，它们看起来依然是物理正确的：  

![](1679148478048.png)

IBL 的教程结束了，本节的代码可在[球体场景](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.2.1.ibl_specular/ibl_specular.cpp)和[纹理场景](https://learnopengl.com/code_viewer_gh.php?code=src/6.pbr/2.2.2.ibl_specular_textured/ibl_specular_textured.cpp)中找到。

# **四. 进阶：PBR 核心理论和原理**

上章主要介绍了 PBR 中 Cook-Torrance 的 BRDF 的两个部分：直接光照和 IBL。

这章将深入介绍 PBR 核心部分的底层理论和原理，使读者对 PBR 的底层原理有更彻底的理解。本章部分内容在上一章已经有所涉及，但会更加深入。

主要面向：

*   进阶程序员
*   对 PBR 底层原理感兴趣的人
$$
\lim_{x\to 0}\frac{\sin x}{x}=1
$$
## **4.1 再论 PBR 核心理论**

上章讲述了符合 PBR 必须满足以下 3 个条件：

*   **基于微平面模型（Be based on the microfacet surface model）**。该模型将物体表面建模成无数微观尺度上有随机朝向的**理想镜面反射**的小平面（microfacet）。微观几何（microgeometry）是在不同微表面改变其法线，从而改变反射和折射光的方向。常用统计方法处理微观几何现象，将表面视为具有微观结构法线的随机分布，在宏观表面视为在每个点处多个方向上反射（和折射）光的总和。
*   **能量守恒 （Energy Conservation）**。出射光线的能量永远不能大于入射光线的能量。随着表面粗糙度的增加，镜面反射区域的面积会增加，但平均亮度则会下降。
*   **使用基于物理的 BRDF（Use a physically based BRDF）**。Cook-Torance 的 BRDF 是实时渲染领域最普遍的 PBR 光照模型，上章详述了其原理和实现。它是数学和物理领域里诸多知识的综合体。

若是将上面 3 点进一步详细论述，将涉及以下知识点：

*   **光学**
    *   光的性质
    *   光的理论
    *   光的能量
    *   几何光学
        *   反射
            *   镜面反射
            *   菲涅尔反射
        *   折射
            *   散射
            *   吸收
*   **物质**
    *   微观表面
    *   宏观表面
    *   分类：金属、电介质、半导体
    *   与光学的交互
*   **能量**
    *   光能
    *   电能
    *   动能
    *   热能
    *   能量守恒
    *   能量转化
*   **PBR 综合**
    *   材质分类
    *   属性模拟
    *   光照模型

总结起来，PBR 就是光学原理和物体结构交互作用的抽象和模拟。下面先从光的性质说起。

## **4.2 光的性质**

### **4.2.1 光是什么？**

有人说光是粒子，有人说光是电磁，有人说光是一种波，有人说光是一种能量，还有人说光是量子，那么光到底是什么？

狭义上说，光是电磁辐射的某一部分内人眼可见的电磁频谱，即可见光，它是人眼可感知的可见光谱，是造成视觉的原因。

可见光通常被定义为具有波长在 400-700 纳米（nm）的范围内，不可见的有红外线（具有更长的波长）和紫外线（具有更短的波长）。

广义上说，光指的是任何波长的电磁辐射，无论是否可见。包括伽马射线、X 射线、微波和无线电波。而可见光（400-700 纳米）只是所有波长区域的一小部分：  

![](1679148478105.png)

### **4.2.2 电磁频谱和可见光（Electromagnetic spectrum and visible light）**

电磁辐射（Electromagnetic Radiation，EMR）按波长从长到短分为：无线电波、微波、红外线、可见光、紫外线、X 射线和伽玛射线。

EMR 的行为取决于其波长。较高频率具有较短波长，较低频率具有较长波长。不同波长的电磁辐射携带着不同的能量。当 EMR 与单个原子和分子相互作用时，其行为取决于它携带的每个量子的能量。

不同波长的可见光代表着不同的颜色。太阳光、日光灯等可见光是一组不同波长的电磁辐射的集合，在三棱镜下可以被分离出不同的颜色：  

![](1679148478354.png)

不同来源对可见光的定义略有不同，有的将可见光定义为狭窄的 420-680nm，有的宽达 380-800nm。在理想的实验室条件下，人们可以看到至少 1050 纳米的红外线; 儿童和年轻人可能会感知波长低至约 310-313 纳米的紫外线。  

![](1679148478410.png)

### **4.2.3 人眼感知可见光原理**

上节阐述了可见光的范围和简单的感知理论，本小节将深入阐述人类为什么会感知并且只感知波长为 380-800 纳米的可见光。

首先要了解人眼的结构和视觉的分子机制。

人眼的结构类似于一架高精度的照相机，光线穿过透明的角膜（cornea）和虹膜（iris）包围的瞳孔（pupil），经过晶状体（lens）的折射在视网膜（retina）上形成空间分布的像。而视网膜上则分布着主要检测**光强度**的视杆细胞（rod cell）和主要检测**颜色**的视锥细胞（cone cell），它们是视觉形成的细胞基础。  

![](1679148478475.png)

视杆细胞与视锥细胞对光的响应程度虽然略有差异，但它们发生光响应的机制都是类似的。以视杆细胞上的视紫红质（rhodopsin）为例，它由一个细胞膜上的七次跨膜蛋白（视蛋白，opsin）和视黄醛（retinal）辅基组成。视蛋白是 G 蛋白偶联受体（GPCR）的一种，视黄醛辅基以共价键结合在其第七个跨膜$\alpha$螺旋片段的赖氨酸残基上。

视黄醛分子是由维生素 A 氧化而来的，一个维生素 A 分子氧化得到一个视黄醛。视黄醛具有两种构型：11 位顺式（11-cis）和 全反式（All-trans），正常与视蛋白结合的是 11 位顺式构型。恰巧在可见光（对视紫红质而言是波长 500 nm 左右的电磁波）照射下，11 位顺式构型可以转变为全反式构型，从而导致视黄醛辅基从视蛋白上脱离。辅基的脱离造成视紫红质**构象变化**，经过信号转导导致细胞膜内外**离子电位发生变化，产生神经电信号**。这一信号经过视神经传入大脑，就使得我们产生了视觉。  

![](1679148478739.png)

  
_(a) 视紫红质的结构；(b) 视黄醛分子的光敏异构反应。_

因此，从视觉的分子机制出发，可以这么回答：正由于视黄醛分子的构型转变反应恰好响应了可见光波段的电磁波，这才导致这一波段的电磁波能被人类 “看见”。

此外，视黄醛分子是维生素 A 的部分氧化产物，又可由植物中广泛存在的天然色素——β- 胡萝卜素氧化得到，来源和代谢路径明确，被大多数生物进化选中作为光敏分子也在情理之中。

**那为什么高度进化的人类不能感知可见光谱之外的电磁辐射呢？为什么视黄醛分子刚好只对可见波段的电磁波产生反应？**

这个可以从各个波段的电磁辐射的性质来回答。

波长最短的伽马射线（Gamma ray）和高能 X 射线（X ray）由于携带的能量（光子）太高，很快就会导致分子电离、分解甚至激发原子核（导致原子核爆发）。首先被排除。

波长较短的深紫外（deep Ultraviolet）和软 X 射线激发的电子能级一般是内层电子或高能电子，这种激发得到的分子高能态很不稳定，在常温下的水溶液或空气中都难以保证信息的有效传递。也被排除。

波长较长的红外（Infrared）与微波（Microwave）频段的电磁波主要与分子的振动、转动和平动相耦合，而这些运动主要以随机热运动形式存在，很难实现信息的准确表达。

波长更长的中波、长波（Radio）的运动尺度超过了单个分子能够接收的尺度，更不适合以细胞为基础的生物选择。

这样考察的结果，如果细胞一定要采取分子层面上的光敏机制对电磁波进行响应，那么最合适的波段可能就是现在的可见光波段。**这一波段在分子运动中相当于电子光谱的外层电子激发能量，与分子中化学键的能量高低大致相当而略低，既不至于损伤一般较为稳定的化学键（尤其是作为生命体基础的 C-C、C-H、C=O、C-N 等化学键），又可以使得一些 “动态” 化学键（例如视黄醛中具有顺反异构的 11 位双键）发生光响应，并实现信息的有效传递。**

所以，人类感知当前波段的可见光，是亿万年不断进化的结果。换个角度说，感知其它波段的人类祖先已经被淘汰了，他们的基因无法遗传传承下来。

可谓：**物竞天择，适者生存**。

### **4.2.4 光的来源**

众所皆知，光是电磁波，而物质是由原子组成，原子是由原子核与核外运转着的电子组成。那么，物质原子中的电磁波是哪里来的？电磁波难道会无中生有？

奥斯特实验发现了直流导线的周围产生磁场，因为电子的运动伴生着磁场。电子的运动分为**线性运动**和**振动**：

*   **线性运动**：电子的线性运动是核外电子的绕核运动及在导电时电子的流动，它所伴生电磁波的宏观表现是**磁场**。电子的线性运动不是产生光的原因。
*   **振动**：电子的振动与发光息息相关，它会使电磁脱离场源形成电磁波，也就是产生了光，而不是所谓的光子。引起电子振动有两种原因：
    *   一是高温物质核外电子的跃迁引发的振动，这种振动需要物质的温度大大高于环境温度，运转速率很高的核外电子跃迁辐射才能达到可见光的频率。这种高温物质核外电子的跃迁辐射所形成发光的光源叫**热光源**。岩浆、铁水、火焰、灯丝等高温物质的发光属于热光源。
    *   二是电子在磁场或电场的作用下引发的受激振动，这样的电子振动与温度无关、与核外电子运转速率无关。这种不需要高温而使电子振动所形成辐射的光源叫**冷光源**。日光灯、节能灯、极光、萤火虫的发光、半导体发光（LED）等属于冷光源。

本小节开头的问题有了答案：光源中的光来自于电子的振动，电子振动所伴生的电磁波辐射形成了光波，电子振动的频率构成了光波的频率，大量电子振动所伴生的电磁波辐射形成了光源。

### **4.2.5 光的理论**

光的研究和理论经过数百年的发展，至今出了很多理论学说，每种理论都是为了解释部分光的物理现象。

目前，光存在的理论主要有：粒子理论、波动理论、电磁理论、量子理论及波粒二象性等。

#### **4.2.5.1 光的粒子理论 (Particle theory)**

光的粒子说又称光的微粒说，这种理论认为光的本质与通过它反射而可见的实体物质一样，是一种粒子（下图）。  

![](1679148478775.png)

  
_光的粒子性示意图_

法国数学家皮埃尔 · 加森迪（Pierre Gassendi）于 1660 年提出了一种光的粒子理论。 Isaac Newton 在 Gassendi 的理论基础上做了扩展：光是由来自各个方向或从各个方向发射的微粒（物质粒子）组成的。

牛顿随后对于加森迪的这种观点进行研究，他根据光的直线传播规律、光的偏振现象，最终于 1675 年提出假设，认为光是从光源发出的一种物质微粒，在均匀媒质中以一定的速度传播。

微粒说很容易解释光的直进性，也很容易解释光的反射，因为粒子与光滑平面发生碰撞的反射定律与光的反射定律相同。

然而微粒说在解释一束光射到两种介质分界面处会同时发生反射和折射，以及几束光交叉相遇后彼此毫不妨碍的继续向前传播等现象时，却发生了很大困难。

#### **4.2.5.2 光的波动理论 (Wave theory)**

在 1660 年代，胡克（Robert Hooke）发表了他的光波动理论。他认为光线在一个名为光以太（Luminiferous ether）的介质中以波的形式四射，并且由于波并不受重力影响，光在进入高密度介质时会减速。

光的波理论预言了干涉现象以及光的偏振性。

欧拉是波动学说的支持者之一，他认为波理论更容易解释衍射现象。

菲涅耳也支持并独立完成了他的波动理论。在 1821 年，菲涅尔使用数学方法使光的偏振在波动理论上得到了唯一解释。  

![](1679148478828.png)

  
_上图：光的偏振现象，回旋光波先后经过四分一波偏振板和线性偏振板的情形。_

但是，波动理论的弱点在于，波类似于声波，传播需要介质。虽然曾有过光以太介质的假想，但因为 19 世纪迈克耳孙 - 莫雷实验陷入了强烈的质疑。

牛顿推测光速在高密度下变高，惠更斯和其他人觉得正相反，但当时并没有准确测量光速的条件。直到 1850 年，莱昂 · 傅科（Léon Foucault）的实验得到了和波动理论同样的结果。之后，经典粒子理论才真正被抛弃。

#### **4.2.5.3 光的电磁理论 (Electromagnetic theory)**

光的电磁理论是关于光的本性的一种现代学说，19 世纪 60 年代由麦克斯韦提出。把光看成是频率在某一范围的电磁波。能解释光的传播、干涉、衍射、散射、偏振等现象，以及光与物质相互作用的规律。

电磁理论还认为，电磁波具有互相垂直的电场与磁场，电场与磁场的频率、振幅、波长、传播方向是一致的。  

![](1679148478925.png)

但由于光还具有粒子性，所以它不能解释光电效应、康普顿效应等物理现象。

#### **4.2.5.4 光的量子理论 (Quantum theory)**

光的量子理论是以辐射的量子理论研究光的产生、传输、检测及光与物质相互作用的学科。  

![](1679148478962.png)

  
_量子光学示意图_

1900 年，普朗克在研究黑体辐射时，为了从理论上推导出得到的与实际相符甚好的经验公式，他大胆地提出了与经典概念迥然不同的假设，即 “组成黑体的振子的能量不能连续变化，只能取一份份的分立值”。

1905 年，爱因斯坦在研究光电效应时推广了普朗克的上述量子论，进而提出了光子的概念。他认为光能并不像电磁波理论所描述的那样分布在波阵面上，而是集中在所谓光子的微粒上。在光电效应中，当光子照射到金属表面时，一次为金属中的电子全部吸收，而无需电磁理论所预计的那种累积能量的时间，电子把这能量的一部分用于克服金属表面对它的吸力即作逸出功，余下的就变成电子离开金属表面后的动能。

1923 年，亚瑟 · 霍利康普顿表明，当从电子散射的低强度 X 射线（所谓的康普顿散射）中看到的波长漂移可以通过 X 射线的粒子理论来解释，而不是波动理论。

1926 年 Gilbert N. Lewis 将这些光量子粒子命名为**光子**。

2018 年 2 月，科学家首次报道了一种可能涉及极化子的新型光的发现，这可能对量子计算机的发展有用。

量子力学作为一门 “很数学” 化的物理体系，已经像经典力学那样成熟了，并成为洞悉微观世界的重要工具。

但量子力学也给留下了许多物理上的困惑，如粒子运动的波粒二象性问题、几率波问题、粒子纠缠问题、波函数崩塌问题等等。

#### **4.2.5.5 光的波粒二象性 (Wave-particle duality)**

历史上关于光是粒子还是波动的争论，已有两千多年（下图）。  

![](1679148478986.png)

光的种种现象和性质表明它既有粒子的特征又有波动的特征，处于两个派别立场的研究者各执一词，互不相让。

直到 1905 年，爱因斯坦在德国《物理年报》上发表了题为《关于光的产生和转化的一个推测性观点》的论文。他认为对于时间的平均值，光表现为波动；对于时间的瞬间值，光表现为粒子性。这是历史上第一次揭示微观客体波动性和粒子性的统一，即**波粒二象性**。这一科学理论最终得到了学术界的广泛接受。

在新的事实与理论面前，光的波动说与粒子说之争以 “光具有波粒二象性” 而落下了帷幕。

即：**光粒子的运动轨迹是呈周期性的波**。

Wikipedia 提供了[一个视频](https://upload.wikimedia.org/wikipedia/commons/transcoded/e/e4/Wave-particle_duality.ogv/Wave-particle_duality.ogv.720p.webm)，形象地描述了光在各种理论下的特征。

### **4.2.6 光的能量**

光是能量的一种传播方式。光能量也被称为光子能量（按粒子性）或电磁辐射（按波动性）。每个光子都具有一定量的能量，频率越高，能量也越高。

光的度量跟能量或辐射测量类似，常被用于太阳能、加热、照明、电信、计算机图形学等领域。

光能量作为能量，可被测量，单位是焦耳（J）。可以通过将辐射通量（或功率）相对于时间、面积、空间积分来计算辐射能量的量。

测量辐射能量的概念和符号非常多，[完整的表](https://en.wikipedia.org/wiki/Radiant_energy)有数十个。下面只列出跟 PBR 相关的概念：

<table><thead><tr><th>名称</th><th>符号</th><th>单位</th><th>公式</th><th>解析</th></tr></thead><tbody><tr><td><strong>辐射能量</strong> (Radiant energy)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-240-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>Q</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2402" style="width: 1.031em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.803em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.75em, 2.631em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2403"><span class="mi" id="MathJax-Span-2404" style="font-family: MathJax_Math-italic;">Q</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.282em; border-left: 0px solid; width: 0px; height: 1.289em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>Q</mi></math></span></span><script type="math/tex" id="MathJax-Element-240">Q</script></span></td><td>焦耳 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-241-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>J</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2405" style="width: 0.803em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.63em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2406"><span class="mi" id="MathJax-Span-2407" style="font-family: MathJax_Math-italic;">J<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>J</mi></math></span></span><script type="math/tex" id="MathJax-Element-241">J</script></span>)</td><td>-</td><td>电磁辐射能量</td></tr><tr><td><strong>辐射通量</strong> (Radiant Flux)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-242-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2408" style="width: 0.974em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.69em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2409"><span class="mi" id="MathJax-Span-2410" style="font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi></math></span></span><script type="math/tex" id="MathJax-Element-242">\Phi</script></span></td><td>瓦 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-243-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>W</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2411" style="width: 1.374em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.089em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1001.09em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2412"><span class="mi" id="MathJax-Span-2413" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>W</mi></math></span></span><script type="math/tex" id="MathJax-Element-243">W</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-244-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi>Q</mi></mrow><mrow><mi>d</mi><mi>t</mi></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2414" style="width: 4.174em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.317em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.089em, 1003.32em, 2.86em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2415"><span class="mi" id="MathJax-Span-2416" style="font-family: MathJax_Main;">Φ</span><span class="mo" id="MathJax-Span-2417" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-2418" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.92em, 4.289em, -999.997em); top: -4.569em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-2419"><span class="mi" id="MathJax-Span-2420" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-2421" style="font-size: 70.7%; font-family: MathJax_Math-italic;">Q</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.63em, 4.174em, -999.997em); top: -3.597em; left: 50%; margin-left: -0.283em;"><span class="mrow" id="MathJax-Span-2422"><span class="mi" id="MathJax-Span-2423" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-2424" style="font-size: 70.7%; font-family: MathJax_Math-italic;">t</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.03em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.031em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.568em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi>Q</mi></mrow><mrow><mi>d</mi><mi>t</mi></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-244">\Phi = \frac{dQ}{dt}</script></span></td><td>单位时间辐射的能量，也叫辐射功率 (Radiant Power) 或通量(Flux)</td></tr><tr><td><strong>辐照度</strong> (Irradiance)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-245-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>E</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2425" style="width: 0.974em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.75em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2426"><span class="mi" id="MathJax-Span-2427" style="font-family: MathJax_Math-italic;">E<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>E</mi></math></span></span><script type="math/tex" id="MathJax-Element-245">E</script></span></td><td>瓦 / 平方米 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-246-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><msup><mi>m</mi><mn>2</mn></msup></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2428" style="width: 3.603em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.86em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1002.86em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2429"><span class="texatom" id="MathJax-Span-2430"><span class="mrow" id="MathJax-Span-2431"><span class="mi" id="MathJax-Span-2432" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-2433"><span class="mrow" id="MathJax-Span-2434"><span class="mo" id="MathJax-Span-2435" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-2436"><span class="mrow" id="MathJax-Span-2437"><span class="msubsup" id="MathJax-Span-2438"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-2439" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-2440" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><msup><mi>m</mi><mn>2</mn></msup></mrow></math></span></span><script type="math/tex" id="MathJax-Element-246">{W}/{m^2}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-247-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2441" style="width: 4.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.774em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1003.77em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2442"><span class="mi" id="MathJax-Span-2443" style="font-family: MathJax_Main;">Φ</span><span class="mo" id="MathJax-Span-2444" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-2445" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.489em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-2446"><span class="mi" id="MathJax-Span-2447" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-2448" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1001.32em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -0.683em;"><span class="mrow" id="MathJax-Span-2449"><span class="mi" id="MathJax-Span-2450" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-2451"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-2452" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-2453" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.49em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.489em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi mathvariant="normal">Φ</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-247">\Phi = \frac{d\Phi}{dA^\perp}</script></span></td><td><em>到达</em>单位面积的辐射通量</td></tr><tr><td><strong>辐射度</strong> (Radiosity)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-248-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>M</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2454" style="width: 1.317em; display: inline-block;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1001.03em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2455"><span class="mi" id="MathJax-Span-2456" style="font-family: MathJax_Math-italic;">M<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>M</mi></math></span></span><script type="math/tex" id="MathJax-Element-248">M</script></span></td><td>瓦 / 平方米 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-249-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><msup><mi>m</mi><mn>2</mn></msup></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2457" style="width: 3.603em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.86em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1002.86em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2458"><span class="texatom" id="MathJax-Span-2459"><span class="mrow" id="MathJax-Span-2460"><span class="mi" id="MathJax-Span-2461" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-2462"><span class="mrow" id="MathJax-Span-2463"><span class="mo" id="MathJax-Span-2464" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-2465"><span class="mrow" id="MathJax-Span-2466"><span class="msubsup" id="MathJax-Span-2467"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-2468" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-2469" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><msup><mi>m</mi><mn>2</mn></msup></mrow></math></span></span><script type="math/tex" id="MathJax-Element-249">{W}/{m^2}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-250-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>M</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2470" style="width: 5.146em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.117em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1004.12em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2471"><span class="mi" id="MathJax-Span-2472" style="font-family: MathJax_Math-italic;">M<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mo" id="MathJax-Span-2473" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-2474" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.489em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-2475"><span class="mi" id="MathJax-Span-2476" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-2477" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1001.32em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -0.683em;"><span class="mrow" id="MathJax-Span-2478"><span class="mi" id="MathJax-Span-2479" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-2480"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-2481" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-2482" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.49em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.489em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>M</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-250">M = \frac{d\Phi}{dA^\perp}</script></span></td><td><em>离开</em>单位面积的辐射通量，也叫辐出度、辐射出射度（Radiant Existance）</td></tr><tr><td><strong>辐射强度</strong> (Radiant Intensity)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-251-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>I</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2483" style="width: 0.689em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.517em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.52em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2484"><span class="mi" id="MathJax-Span-2485" style="font-family: MathJax_Math-italic;">I<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>I</mi></math></span></span><script type="math/tex" id="MathJax-Element-251">I</script></span></td><td>瓦 / 立体弧度 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-252-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>s</mi><mi>r</mi></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2486" style="width: 3.089em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.46em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1002.46em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2487"><span class="texatom" id="MathJax-Span-2488"><span class="mrow" id="MathJax-Span-2489"><span class="mi" id="MathJax-Span-2490" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-2491"><span class="mrow" id="MathJax-Span-2492"><span class="mo" id="MathJax-Span-2493" style="font-family: MathJax_Main;">/</span></span></span><span class="texatom" id="MathJax-Span-2494"><span class="mrow" id="MathJax-Span-2495"><span class="mi" id="MathJax-Span-2496" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-2497" style="font-family: MathJax_Math-italic;">r</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><mrow class="MJX-TeXAtom-ORD"><mi>s</mi><mi>r</mi></mrow></math></span></span><script type="math/tex" id="MathJax-Element-252">{W}/{sr}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-253-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>I</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><mi>&amp;#x03C9;</mi></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2498" style="width: 3.946em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.146em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1003.15em, 2.86em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2499"><span class="mi" id="MathJax-Span-2500" style="font-family: MathJax_Math-italic;">I<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mo" id="MathJax-Span-2501" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-2502" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-2503"><span class="mi" id="MathJax-Span-2504" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-2505" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.8em, 4.174em, -999.997em); top: -3.597em; left: 50%; margin-left: -0.397em;"><span class="mrow" id="MathJax-Span-2506"><span class="mi" id="MathJax-Span-2507" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-2508" style="font-size: 70.7%; font-family: MathJax_Math-italic;">ω</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.03em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.031em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.568em; border-left: 0px solid; width: 0px; height: 1.789em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>I</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><mi>ω</mi></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-253">I = \frac{d\Phi}{d\omega}</script></span></td><td>通过单位立体角的辐射通量</td></tr><tr><td><strong>辐射率</strong> (Radiance)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-254-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>L</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2509" style="width: 0.86em; display: inline-block;"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1000.63em, 2.46em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2510"><span class="mi" id="MathJax-Span-2511" style="font-family: MathJax_Math-italic;">L</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.068em; border-left: 0px solid; width: 0px; height: 1.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>L</mi></math></span></span><script type="math/tex" id="MathJax-Element-254">L</script></span></td><td>瓦 / 平方米立体弧度 (<span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-255-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>W</mi></mrow><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mo>/</mo></mrow><msup><mi>m</mi><mn>2</mn></msup><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>s</mi><mi>r</mi></mrow></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2512" style="width: 4.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.774em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1003.77em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2513"><span class="texatom" id="MathJax-Span-2514"><span class="mrow" id="MathJax-Span-2515"><span class="mi" id="MathJax-Span-2516" style="font-family: MathJax_Math-italic;">W<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.117em;"></span></span></span></span><span class="texatom" id="MathJax-Span-2517"><span class="mrow" id="MathJax-Span-2518"><span class="mo" id="MathJax-Span-2519" style="font-family: MathJax_Main;">/</span></span></span><span class="msubsup" id="MathJax-Span-2520"><span style="display: inline-block; position: relative; width: 1.317em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.86em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-2521" style="font-family: MathJax_Math-italic;">m</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.86em;"><span class="mn" id="MathJax-Span-2522" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="texatom" id="MathJax-Span-2523"><span class="mrow" id="MathJax-Span-2524"><span class="mi" id="MathJax-Span-2525" style="font-family: MathJax_Math-italic;">s</span><span class="mi" id="MathJax-Span-2526" style="font-family: MathJax_Math-italic;">r</span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mrow class="MJX-TeXAtom-ORD"><mi>W</mi></mrow><mrow class="MJX-TeXAtom-ORD"><mo>/</mo></mrow><msup><mi>m</mi><mn>2</mn></msup><mrow class="MJX-TeXAtom-ORD"><mi>s</mi><mi>r</mi></mrow></math></span></span><script type="math/tex" id="MathJax-Element-255">{W}/m^2{sr}</script></span>)</td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-256-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>L</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant=&quot;normal&quot;>&amp;#x03A6;</mi></mrow><mrow><mi>d</mi><mi>&amp;#x03C9;</mi><mi>d</mi><msup><mi>A</mi><mo>&amp;#x22A5;</mo></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-2527" style="width: 5.717em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.574em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1004.57em, 3.031em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-2528"><span class="mi" id="MathJax-Span-2529" style="font-family: MathJax_Math-italic;">L</span><span class="mo" id="MathJax-Span-2530" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-2531" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 2.289em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.86em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.454em;"><span class="mrow" id="MathJax-Span-2532"><span class="mi" id="MathJax-Span-2533" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-2534" style="font-size: 70.7%; font-family: MathJax_Main;">Φ</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1002.17em, 4.174em, -999.997em); top: -3.426em; left: 50%; margin-left: -1.083em;"><span class="mrow" id="MathJax-Span-2535"><span class="mi" id="MathJax-Span-2536" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-2537" style="font-size: 70.7%; font-family: MathJax_Math-italic;">ω</span><span class="mi" id="MathJax-Span-2538" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="msubsup" id="MathJax-Span-2539"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-2540" style="font-size: 70.7%; font-family: MathJax_Math-italic;">A</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.283em; left: 0.517em;"><span class="mo" id="MathJax-Span-2541" style="font-size: 50%; font-family: MathJax_Main;">⊥</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.29em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.289em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.782em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>L</mi><mo>=</mo><mfrac><mrow><mi>d</mi><mi mathvariant="normal">Φ</mi></mrow><mrow><mi>d</mi><mi>ω</mi><mi>d</mi><msup><mi>A</mi><mo>⊥</mo></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-256">L = \frac{d\Phi}{d\omega dA^\perp}</script></span></td><td>通过单位面积单位立体角的辐射通量</td></tr></tbody></table>

## **4.3 光学原理（Optics theory）**

**光学**（Optics）是物理学的一个分支，研究光的行为和性质，包括它与物质的相互作用以及使用或检测它的仪器的结构。

光学通常描述可见光、紫外光和红外光的行为。由于光是电磁波，其它波段的电磁辐射（如 X 射线、微波和无线电波）表现出类似的特性。

光学按照不同角度、不同粒度和不同侧重点大致可以分为以下几类：

*   **电磁光学**。将光分为大多数光学现象可以使用光的经典电磁描述来解释。然而，光的完整电磁描述通常难以应用于实践中，需要借助其它光学类型。
*   **几何光学**。几何光学系统将光线视为一组光线，它们以直线传播，并在通过或从表面反射时弯曲。是物理应用中简化的一种模型。由于 PBR 的 BRDF 几乎都是基于几何光学，后面章节会侧重地介绍几何光学。
*   **物理光学**。物理光学是一种更全面的光模型，包括衍射和干涉等波效应几何光学中无法解释的。历史上，首先开发基于射线的光模型，然后是波的光模型。19 世纪电磁理论的进步才发现光波实际上是电磁辐射。
*   **运动物理光学**。主要研究天体运动的光速差、光漂移、多普勒效应等。当前已经发展成一支庞大的独立的物理分支。
*   **量子光学**。一些现象取决于光具有波状和粒子状特性的事实。这些效应的解释需要量子力学。当考虑光的粒子特性时，光被建模为称为 “光子” 的粒子集合。量子光学涉及量子力学在光学系统中的应用。

光学与许多相关学科联合进行研究，包括天文学、工程领域、摄影、计算机和医学等等。光学的应用存在于各种日常物品中，包括镜子、透镜、望远镜、显微镜、激光器和光纤等等。

### **4.3.1 光的反射（Reflection）**

光的反射是当光在两种物质分界面上改变传播方向又返回原来物质中的现象。  

![](1679148479241.png)

  
_上图：光的反射现象。_

产生反射的原理：光是电磁波，射在物体上的光波引起单个原子中的极化振荡（或电子在金属中的振荡），致使每个粒子在各个方向上辐射小的二次波，如偶极天线（ dipole antenna）。根据惠更斯 - 菲涅耳原理，所有这些波加起来就产生反射和折射。

光的反射细分为以下几种：

*   **镜面反射**（Specular reflection）：平行光线射到光滑表面上时反射光线也是平行的现象。表面平滑的物体，易形成光的镜面反射，形成刺目的强光，反而看不清楚物体。
*   **漫反射**（Diffuse reflection）：平行光线射到凹凸不平的表面上，反射光线射向各个方向的现象。
*   **方向反射**（Directional reflection）：是介于漫反射和镜面反射之间反射，也称非朗伯反射。其表现为各向都有反射，且各向反射强度不均一。
*   **回射**（Retroreflection）：入射光射在介质表面上，反射光的方向与入射光一致的现象。飞机在飞越被阳光照射的云层时，在飞机阴影周围看到的区域将显得更亮，从草地上的露水可以看到类似的效果。

### **4.3.2 光的折射（Refraction）**

光的折射是指光从一种介质斜射入另一种介质时，传播方向发生改变，从而使光线在不同介交界处发生的偏折。  

![](1679148479282.png)

  
_上图：光的折射现象。_

折射的原理与反射类似：光波是一种特定频段的电磁波，光在传播过程中有两个垂直于传播方向的分量：电场分量和磁场分量。当电场分量与介质中的原子发生相互作用，引起电子极化，形成电子云和原子荷重心发生相对位移。导致光的一部分能量被吸收，同时光在介质中的速度被减慢，方向发生变化，引发了折射。  

![](1679148479351.png)

  
_上图：光从一种物质进入另一种物质后发生了折射，波长、方向、速率都发生了改变。_

近代物理学指出，光是一种没有静质量、体积非常小、运动速度比较高的物质。光和其它物质有相同的性质。光和物质间的相互作用力使光的运动方向发生改变即折射。

### **4.3.3 光的散射（Scattering）**

光通过不均匀媒质时，部分光束将偏离原来方向而分散传播，从侧向也可以看到光的现象。

散射发生的原理：当光子与分子或原子相互接近时，由于双方具有很强的相互斥力，迫使它们在接触前就偏离了原来的运动方向而分开。

散射是观察和辨别物体的主要现象，是自然中最普遍存在的现象。漫反射其实也是散射的一种。

### **4.3.4 光的色散（Dispersion）**

光的色散指的是复色光分解为单色光的现象。  

![](1679148479609.png)

色散现象说明光在介质中的速度 $v=\frac{c}{n}$（$n$为介质的折射率）随光的频率 $f$而变。光的色散可以用三棱镜、衍射光栅、干涉仪等来实现。

光的色散说明了光具有波动性。因为色散是光的成分（不同色光）折射率不同引起的，而折射率由波的频率决定。

对同一种介质，光的频率越高，介质对这种光的折射率就越大。在可见光中，紫光的频率最高，红光频率最小。当白光通过三棱镜时，棱镜对紫光的折射率最大，光通过棱镜后，紫光的偏折程度最大，红光偏折程度最小。这样，三棱镜将不同频率的光分开，就产生了光的色散。

**为什么在同一介质中，不同波长的光，其速度和折射率会不同呢？**

由于光有粒子性，与介质的原子、分子有相互作用力。对于波长越短的光，其携带的能量越大、运动越强，与介质的原子、分子的相互作用力越大，致使其速度越小、折射率越大。_（这个回答是笔者根据经典物理学结合波动论的推测，未找到确切的依据和论据，有待考证！）_

### **4.3.5 光的吸收（Absorption）**

电磁理论认为，光的吸收是光 (电磁辐射) 通过材料时，与材料发生相互作用，电磁辐射能量被部分地转化为其他能量形式的物理过程。

当被吸收的光能量以热能的形式被释放，即形成了光热转化；当未被吸收的光能量被物体反射、散射或透射，便影响着我们看到的物体的色彩。

量子理论认为，光的吸收是指分子或原子在光波辐射场（光照）下，会吸收光子的能量由低能态跃迁到高能态的现象。这种跃迁也等效于一个具有一定固有频率的振子。

电磁理论证明，当物体对某种频率光的吸收系数很大时，它对该频率光的反射率也大。若干电介质具有很强的吸收带，故它们对于吸收带附近频率的光也有很强的反射，这称为**选择反射**。

半导体材料在不同的程度上具备电介质和金属材料的全部光学特性。当半导体材料从外界以某种形式（如光、电等）吸收能量，则其电子将从基态被激发到激发态，即光吸收。而处于激发态的电子会自发或受激再从激发态跃迁到基态，并将吸收的能量以光的形式辐射出来（辐射复合），即发光；当然也可以无辐射的形式如发热将吸收的能量发散出来（无辐射复合）

金属的光吸收要同时考虑束缚电子与自由电子的作用。对于红外线或更低频率的辐射，自由电子起主要作用；而对于紫外线及更高频率的辐射，则束缚电子的作用比较显著，这时金属实际上表现出与电介质相似的光学性质。

### **4.3.6 光的衍射（Diffraction）**

衍射是指当光波遇到障碍物或狭缝时发生的各种现象。它被定义为围绕障碍物或孔的角落的波浪弯曲到障碍物的几何阴影区域中。

由于光具有波动性，所以也会产生衍射。  

![](1679148479671.png)

  
_光波穿过单波长的缝隙后发生了衍射现象。_

当光波穿过具有变化的折射率的介质时，也会发生衍射的效果。所有波都会发生衍射，包括声波、水波和电磁波（可见光、X 射线和无线电波）。

光的衍射产生的原理可以从两方面解释：

*   波动光学：根据惠更斯 - 菲涅耳原理和波叠加原理，衍射是由于波传播的方式而产生的。通过将波前传输介质的每个粒子视为二次球面波的点源，可以可视化波的传播，任何后续点的波位移是这些二次波的总和。当这些波被加在一起时，它们的总和由相对相位以及各个波的幅度确定，使得波的总和幅度可以具有零和各个幅度之和之间的任何值
*   量子光学：在通过狭缝传播光时，每个光子具有波函数，波函数描述了从发射器通过狭缝到屏幕的路径。波函数（光子将采取的路径）由物理环境决定，例如狭缝形状、屏幕距离和光子创建时的初始条件。

### **4.3.7 光的叠加和干涉（Superposition and interference）**

若干个光波组合在一起形成的复合效果便是光的叠加。

更准确地说，在没有非线性效应的情况下，叠加原理可用于通过简单地添加干涉来预测相互作用波形的形状。产生光波的复合图案的相互作用通常被称为干涉，干涉可能导致不同的结果。  

![](1679148479725.png)

  
_光波在不同相位产生的叠加效果。_

光产生叠加和干涉现象的原理与衍射类型，便不再累述。

### **4.3.8 光的偏振（Polarization）**

偏振是波的一般属性，描述了它们的振荡方向。由于光具有波动性，所以也会偏振。

偏振为横向波（如许多电磁波）描述了垂直于行进波方向的平面中的振荡方向。振荡可以在单个方向上（线性偏振），或者随着波行进方向而旋转（圆形或椭圆形偏振）。  

![](1679148479961.png)

  
_左：线性偏振；中：圆形偏振；右：椭圆偏振。_

## **4.4 几何光学（Geometry optics）**

**几何光学**是将光的波长视作无限小，以致可以将光当成直线来研究的一门物理分支。它以光线为基础，研究光的传播和成像规律。

在几何光学中，把组成物体的物点看作是几何点，把它所发出的光束看作是无数几何光线的集合，光线的方向代表光能的传播方向。

**几何光学中光线的概念与光的波动性质相违背**。因为从能量和光的波动性现象（如衍射）来看，这种几何光线都是不可能存在的。**几何光学只是波动光学的近似**，把所有光当成波长极小的情况处理，小到光线被当成了直线。但是，简化后的几何光学可以不涉及光的物理本性，而能以其简便的方法解决光学仪器中的光学技术问题和计算机图形渲染的复杂度问题。

在几何光学中，特别是在计算机图形学中，光线处理做了以下简化或遵循以下基本定律：

*   **直线传播**：光在均匀介质中沿直线传播。直线意味着将光波长视作极小。
*   **独立传播**：两束光在传播途中相遇时互不干扰，仍按各自的途径继续传播。而当两束光会聚于同一点时，在该点上的光能量是简单的相加。
*   **反射和折射**：光在传播途中遇到两种不同介质的光滑分界面时，一部分光线反射，它们的传播方向遵循反射定律；另外另一部分折射，它们的传播方向遵循折射定律。
*   **路径可逆性**：一束光线从一点出发经过无论多少次反射和折射，如在最后遇到与光束成直角的界面反射，光束必然准确地循原路返回出发点。
*   **介质各向同性**：将光线传播的介质简化成均匀的，即各向同性（isotropic）。所谓各向同性是指介质任意一点的所有方向的物理性质（如密度、弹性、摩擦系数等等）和化学性质是一样的。
*   **分界面简化**：将光线经过两个介质的分界面处的点当做平面上的点处理。
*   **单色光源**：真实的发光源基本是复合色光源，但在几何光学中，简化成了单色光，避开色散、傅里叶积分、能量积分等复杂问题。
*   **麦克斯韦方程的特化**：由于很多参数作了特例化和简化，所以几何光学的光照计算实际上使用的是麦克斯韦方程简化和特化后的版本。

由于 PBR 的相关技术及诸多理论跟几何光学相关，所以本节将深入地探讨几何光学的内容。

### **4.4.1 反射定律（Law of Reflection）**

反射定律描述了反射光的角度：入射光的角度与反射光的角度相同。  

![](1679148480248.png)

如上图所示，入射光线 P 射在介质点 O 上，反射光线是 Q，点 O 的法线是 normal，则根据反射定律：

$$\theta_i = \theta_r$$

即入射角$\theta_i$和反射角$\theta_r$相同。它们的另外一种等效表达形式：

$$\theta_r = \pi - \theta_i$$

需要注意的是，反射定律描述的是反射角问题，并不涉及能量分配。

反射还涉及到反射率的问题。**反射率**是反射波的功率与入射波的功率之比。每种材料的反射率不一样，并且跟入射光与介质的夹角有关，这种现象叫**菲涅尔反射效应**，与之相关的方程是[菲涅尔方程（Fresnel equations）](https://en.wikipedia.org/wiki/Fresnel_equations)。  

![](1679148480311.png)

  
_上图：菲涅尔反射效应，球体的反射率从中心到边缘以某种曲线提升。_

实际上，当光照射到介质表面时，光可能产生的结果：

*   被吸收：部分光被介质吸收，转化其它形式的能力。
*   被折射：部分折射的光如果透过介质进入另外的介质，就会形成透射效果。
*   被反射：部分光被反射，其中反射分成了镜面反射和漫反射。

### **4.4.2 折射定律（Law of Refraction）**

**折射定律**也叫**斯涅尔定律**（Snell's law，Snell–Descartes law），描述了光在两种介质之间折射后的角度、折射率、光速的关系。  

![](1679148480364.png)

  
_上图：折射定律动画示意图。_

![](1679148480438.png)

  
如上图所示，光在介质之间发生了折射，介质 1 的入射角、折射率和光速分别是$\theta_1$、$n_1$、$v_1$，介质 2 的入射角、折射率和光速分别是$\theta_2$、$n_2$、$v_2$，则根据折射定律，它们有以下的关系：

$$\frac{\text{sin}\theta_2}{\text{sin}\theta_1} = \frac{v_2}{v_1} = \frac{n_2}{n_1}$$

用折射定律会出现一种**异常**情况：当光从较**高折射率**的介质传播到较**低折射率**的介质时，在入射角足够大的情况下，折射定律似乎要求折射角的正弦大于 1。

这当然是不可能的。实际上，在这种情况下，光线完全被边界反射，这种现象称为**全内反射（Total internal reflection）**。仍能导致折射的最大入射角称为**临界角**，此时折射角是 $90^\circ$。

![](1679148480491.png)

如上图所示，光线从较高折射率的水射到较低折射率的空气中，当入射角大于临界角时，会出现图右的全内反射。

其中，临界角$\theta_\text{crit}$可由折射定律推导出来：

$$\theta_\text{crit} = \text{arcsin} \left( \frac{n_2}{n_1}\text{sin}\theta_2 \right)$$

由于水的折射率 $n_1 = 1.333$，空气的折射率 $n_2 = 1.0$，折射角$\theta_2 = 90^\circ$，则根据上面的公式可以算出水相对空气折射的临界角：

$$\theta_\text{crit_water2air} = \text{arcsin} \left( \frac{1.0}{1.333}\text{sin}90^\circ \right) = 48.6^\circ$$

### **4.4.3 几何光学的其它定律**

在几何光学中，除了反射定律和折射定律之外，还有以下定律：

*   拉格朗日积分不变式。
*   费马原理。
*   马吕斯和杜平定律。

由于这些定律跟 PBR 技术关联不大，本文不详述，有兴趣的可以另外找资料。

## **4.5 物质理论**

### **4.5.1 物质是什么？**

从经典物理学上，物质是任何具有质量并且有体积占据空间的东西。

从现代物理学上，物质是构成宇宙间一切物体的实物和能量场（光、电场、磁场、声等），还包括反物质和暗物质。

从宏观上，物质是所有看得见摸得着感受得到的东西。

从微观上，物质是由原子构成的所有东西，而原子又是由相互作用的亚原子粒子构成。实体粒子包含但不限于：原子、中子、质子、电子、夸克、轻子、重子、费米子等等；能量粒子包含但不限于：光量子、声波等。

从哲学上，物质除了客观实体，还有包含了主观意识。

虽然物质的概念和构成非常复杂，但计算机图形学的 PBR 领域，只要研究经典物理学的物质和能量场的光波即可。

### **4.5.2 物质结构**

物质的构成千姿百态，结构也形态各异。

从不同的微观尺寸观察，物体的结构描述如下：

*   $10^0m$：1 米维度，这个维度就是人类最常接触也最熟悉的尺度，人体、动物、生物、静态物体大多数是这个维度。
*   $10^{-3}m$：1 毫米维度，仔细观察或借助普通的放大镜，人类还是能清晰看到很多东西，如毛孔、微型动物、微型昆虫、巨型细菌、血管、毛发、皮肤细节等。
*   $10^{-6}m$：1 微米维度，在此维度下的物质有细菌、病毒、DNA、血小板、红细胞、白细胞、淋巴细胞等等。
*   $10^{-9}m$：1 纳米维度，这个维度已经到达了原子级别，可以分析氢原子、X 射线的特性。
*   $10^{-12}m$：1 皮米维度，这个维度已经深入到原子核级别，可描述的原子、质子、中子的尺寸。
*   $10^{-15}m$：1 飞米维度，这个维度只能描述电荷、强子、费米子的尺寸。
*   $10^{-18}m$：1 阿米维度，这个维度只能描述电子、夸克的尺寸。
*   $10^{-35}m$：1 普朗克长度，这个维度只能描述量子泡沫、量子弦。  
    
    ![](1679148480511.png)
    
      
    _不同尺寸维度下的物质构成。从上到下：$10^{-35}m$到 $10^{27}m$。_

### **4.5.3 物质形态**

物质的**形态（Phase，也叫相态）**常见的有：固态、气态、液态、非晶态、液晶态，另外还有奇特的形态：等离子态、超固态、中子态、超导态、超流态、玻色–爱因斯坦凝聚、费米子凝聚态。

这些状态都是物质在不同的密度、温度、压强、辐射下的形态，当所处的环境发生改变时，会从一种形态转成另外一种形态。  

![](1679148480563.png)

  
_物质在固态、液态、气态、等离子态的转化图。_

### **4.5.4 物质属性**

物质的属性有很多，从不同角度有不同的属性，但常见的物理和化学属性有：体积、尺寸、质量、密度、硬度、导电性、导磁性、磁性、范性（可塑性）、透光性（透明度）、比热容、弹性、可燃性、助燃性、酸碱性等等。

下面只阐述跟 PBR 相关的物质属性。

#### **4.5.4.1 导电性**

**导电性**是指物质内载电荷粒子的运动。含有电荷的粒子称为**电荷载子**，它们的运动形成了**电流**。

电流产生的原因有两种：

*   受到电场的作用。
*   电荷载子分布不均匀引发的扩散机制。

根据导电性，可以将物质分为：

*   **绝缘体**：也称电介质，常见的绝缘体包含干燥的木材、塑料、橡胶、纸张等。
*   **半导体**：处于导体和绝缘体之间，半导体的电传导是由电场作用和扩散这两种物理机制共同引发。常见的半导体有硅、锗、砷化镓等。
*   **导体**：导电性强，常见的导体有金属、电解质、液体等。
*   **超导体**：导电性非常强，没有电阻，通常在绝对零度左右的极限条件下才能出现。

#### **4.5.4.2 粗糙度**

**粗糙度**是反映物质微观表面的轮廓紊乱的程度。

根据粗糙程度可以将物体分成以下几类：

*   **光滑物体**：粗糙度非常小，几乎为 0，摩擦系数很小；微观表面轮廓几乎是一条直线，易产生镜面反射。
*   **半光滑物体**：粗糙度较小，摩擦系数较小；微观表面轮廓是较平整，产生介于镜面和漫反射之间的反射。
*   **粗糙物体**：粗糙度很大，接近 1.0，摩擦系数很大；微观表面轮廓非常不平坦，产生漫反射。

![](1679148480615.png)

  
有很多估算物体粗糙度的方法：

![](1679148480644.png)

*   $R_a$：算术平均偏差法，公式：

$$R_a = \frac{1}{n}\sum_{i=1}^n|y_i|$$

*   $R_q$：均方根法，公式：

$$R_q = \sqrt{\frac{1}{n}\sum_{i=1}^n y_i^2}$$

*   $R_{sk}$：偏态分布法，公式：

$$R_{sk} = \frac{1}{nR_{q^3}}\sum_{i=1}^n y_i^3$$

*   $R_{ku}$：峰态法，公式：

$$R_{ku} = \frac{1}{nR_{q^4}}\sum_{i=1}^n y_i^4$$

还有 $R_zDIN$、$R_zJIS$法，但最常用的是 $R_a$。

#### **4.5.4.3 透光性**

**透光性**亦即透明度，描述物质透过光线的程度。

物体的透光性主要取决于物质内部结构对外来光子的吸收和散射。

金属物质在可见光波段的电子轨道密集（有能带），能强烈地吸收对应能量的光子并发射（相同能量或较小能量的）光子，即其表面可以强烈地反射光，是不透明的。

物质内部对光的散射，主要取决于其内部缺陷的多少，缺陷多的物质，散射率高。普通陶瓷材料，其内部充斥着大量的微气孔等缺陷，气孔会对经过的光产生强烈的散射，所以普通陶瓷是不透明的；而气孔率保持极低的陶瓷材料是可以像玻璃一样透明的。单晶体（如天然水晶）和液体（如水）由于内部排列规则，缺陷极少，是透明的。

#### **4.5.4.4 各向性**

各向性是描述物质任意一点的物理和化学等属性跟方向是否相关，如果与方向无关叫**各向同性（isortropy）**，否则叫**各向异性（anisortropy）**。

比如清澈平静的水是各向同性，因为水的每个部分的属性（密度、压力、温度、折射率质量等等）都与方向无关；而飘忽不定的烟或雾则是各向异性，很明显同个部位不同方向有着不一样的密度或外力。

在计算机图形学，特别是实时渲染领域，通常将物体简化成均匀的，即各向同性的。几何光学也通常将光线传播的介质看成各向同性。

## **4.6 能量理论**

### **4.6.1 能量是什么？**

**能量**是必须转移到物体以便对物体进行影响或加热的定量属性。能量是物质运动转换的量度，是表征物理系统做功本领的量度。

能量的单位与功的单位相同，在国际单位制中是焦耳（J）；在微观研究领域，常用电子伏（eV）作为单位。

### **4.6.2 能量类型**

能量存在的形式多种多样，在不同学科不同分支不同角度有着不同的类别，主要有：

*   **机械能**：物体运动（平移、旋转）宏观的能量（动能、势能）的总和。
*   **电能**：由电场引起或存储的势能。
*   **磁能**：由磁场引起或存储的势能。
*   **重力势能**：由重力场引起或存储的势能。
*   **化学能**：由化学键引起的势能。
*   **电离能**：由电子与其原子或分子结合引起的势能。
*   **核能**：结合原子形成原子核和核反应的势能。
*   **胶子结合能**：结合夸克形成强子的势能。
*   **弹性能**：由于材料（或其容器）的变形而出现恢复力的势能。
*   **机械波能**：由于机械波的传播引发的弹性材料中的动能和势能。
*   **声波能**：声波是机械波的一种。
*   **辐射能**：存储在电磁辐射传播的场中的势能，包括光。
*   **静止能**：由物体的静止质量引发的势能。
*   **热能**：粒子微观运动的动能，是一种无序等效的机械能。

### **4.6.3 能量转化**

能量的类型多钟多样，而且它们之间是可以相互转化的。

能量的转化在生活中随处可见。比如，“利用太阳能发的电煮开了水，水蒸气一直往上冒”，蕴含了很多能量转化的过程：

*   太阳内部的核能通过核反应转变成太阳光的辐射能；
*   太阳发电站的太阳板吸收辐射能形成了电能；
*   利用煮水电器将电能转化成了水的热能；
*   水蒸气向上冒的过程转化成了重力势能。

### **4.6.4 质能等值**

爱因斯坦在二十世纪初提出了著名的质能转化公式：

$$E = Pc = mc\cdot c = mc^2$$

其中 $E$是能量，$m$是物质的静止质量，$c$是真空的光速，约 $3\times10^8m/s$。

质能等值公式揭示了任意有静止质量的物质都可以转化成能量，而且很少一部分的物质转成能量后是非常巨量的。

比如，1kg 的物质转成能量后：

$$E_{1kg} \approx 1.0 \times (3\times10^8)^2 = 9\times10^{16} J$$

其能量相当于：

*   250 亿千瓦时（$2000GW\cdot h$）；
*   三峡水电站（全球发电量最大）一整个季度的发电量；
*   21.5 万亿千卡（$21Pcal$）；
*   21 万吨 TNT；
*   26.3 亿升汽油。

由此可见，物质转化成能量后，是非常恐怖的。一块小石头，足以毁掉一个小星球。

幸好，目前尚没有很便捷地将物质转化成能量的方法。然而，虽然核爆（原子弹、中子弹、氢弹）不是利用质能转化定理，但是核能表现出的威力已经足够令人望而生畏了。

### **4.6.5 能量守恒 (Conservation of energy)**

传统物理学上，**能量守恒定律**表明在封闭的系统中，能量不会凭空出现或消失，只会从一种形式转成另外一种形式，能量的总量保持不变。

现代物理学上，由于质量和能量可以相互转化，能量守恒扩展到能量和质量的总和保持不变。

诺埃德定理（Noether's theorem）可以严格证明能量守恒，它也表明了永动机是不可行的。

![](1679148481033.png)

比如，上图所示，光线照到物质表面上时，光能可能一部分被镜面反射（黄色），一部分被散射（深蓝色），一部分被吸收（褐色），还有一部分被透射（图中未标识）。与之对应的能量分别是反射部分依旧是光能，吸收部分转化成热能、电能等形式，透射部分依旧还是光能，并且：

$$E_{in} = E_{specular} + E_{diffuse} + E_{absorb} + E_{transmit}$$

亦即入射光能与反射（包含镜面反射和散射）、吸收、透射部分的总光能相等，遵循了能量守恒定律。

PBR 的 BRDF 也遵循了这一守恒定律，引入粗糙度、反射率等概念，使得原理上更加物理正确，渲染效果上更加真实。

## **4.7 PBR 与光学**

本节将阐述本章前几节涉及的光学理论与 PBR 结合的理论，特别是物质和光的交互原理及理论。主要参考了 **Naty Hoffman** 在 2013~2015 年（特别是 2015 年）的 SIGGRAPH 公开课中演讲的主题：**《Physics and Math of Shading》**。

### **4.7.1 光谱能量分布（SPD）**

回顾一下 [4.2.5.3 光的电磁理论(Electromagnetic theory)](#4.2.5.3 光的电磁理论(Electromagnetic theory)) 描述的电磁理论：

光是电磁波，在介质中作为能量以特定波长（Wavelength）和频率振荡着向前方传播。电磁波可被分解成电场（Electric）和磁场（Magnetic），并且它们相互垂直（如下图）。

![](1679148481071.png)

人类可见的光波波长分布在 400nm~780nm 之间，可见光的波长跟蜘蛛丝宽度相仿，但远小于人类头发丝的宽度（下图）。

![](1679148481121.png)

_上图：可见光波长（图左）与蜘蛛丝（图右斜灰线）、人类头发丝（图右黄色区域）宽度比较。_

实际上，绝大多数光包含了很多个波段的电磁场，每个波段的电磁场包含了不同的能量。比如下图的位于 500~550nm 波段的电磁波，在左上角的**光谱能量分布（Spectral Power Distribution，SPD）**中显示出了绿色光的能量。激光可以发出单色的光。

![](1679148481171.png)

下图左 R、G、B 的光谱能量分布乘以各自的缩放因子，将它们的结果相加之后就成了下图右的摸样。

![](1679148481224.png)

这个原理与已投入影院使用的 R、G、B 激光投影系统类似。

上图所示的是能量分布图，如果将它们用波形图表示，会显得更加复杂，因为涉及到光的叠加和干涉，见下图：

![](1679148481270.png)

但是，由于大多数光源都不是单一波长的光波，而是有一定宽度的连续的复合光波，于是它们的最终叠加的波形更加复杂。举个例子，以标准白光光源 D65 为例，它的光谱能量分布图和复合波形如下：

![](1679148481319.png)

有趣的是，下图的两种 SPD 虽然不一样（上部分是连续的分布，下部分是离散的 RGB 分布），但是刚好跟人类视觉成像原理（详见 [4.2.3 人眼感知可见光原理](#4.2.3 人眼感知可见光原理)）匹配，所以人类并没有发现它们之间的差别。也就是说，人类的视觉是有损的，将无限维度的 SPD 简化成了三维视觉空间。

![](1679148481353.png)

### **4.7.2 物质与光交互**

在纳米级别，当一束电磁光波和原子或分子相遇后，会发生什么呢？

![](1679148481398.png)

答案是，光波会引发原子、分子偏振，并且使它们的正负电荷分离，形成偶极子（dipole）。这就意味着进入的光波被吸收了。

![](1679148481430.png)

吸收了能量并且极化后的原子、分子会迅速恢复，重新向外辐射，形成二次光波。当然被吸收的部分能量可能转化成了热运动，即热能。

![](1679148481454.png)

对均匀介质（Homogeneous Medium）来说，光线是沿着直线传播。由于所有物质在原子维度上看是不可能完全均匀的，所以均匀介质是在实践中抽象出来的概念。

在渲染技术中，会使用宏观的统计和组合为材质提供参数。这个参数就是**折射率（Index of refraction，IOR）**，它由两个部分组成：

*   一部分描述光在介质中的速率。
*   另外一部分描述光在介质被吸收的比例。

对于介质中局部不均匀的部分将被建模成粒子（原子核等），折射率不连续的物质会散射入射光，散射出的光方向向着四面八方。

![](1679148481498.png)

虽然这跟前面讨论的单一分子或原子的极化类似，但可以将这些微粒组合起来，形成宏观模型。

下图是吸收系数和散射系数组合成的宏观维度的材质表现图。

![](1679148481523.png)

**横轴**是**散射**系数，从左到右可以看出物质从清澈到混浊的程度；**纵轴**是**吸收**系数，从下到上表示物质从透明（光被全透射而表现出跟入射光几乎一样的颜色）到不透明（光被全吸收而表现出与入射光不一样的颜色）的程度。比如牛奶，它是吸收系数低而散射系数高的物质，所以它表现出入射光一致的白色，并且是混浊的。有色液体能很轻易吸某些波段的光，而其它波段的就没那么容易。

从光学角度上看，最重要的事情是所有材质的表面都是粗糙的。没有表面是完全平坦的，至少在原子维度上是不规则的。原子间排布距离如果跟光波差不多或更小，就会引起一种现象，它就是之前章节涉及过的**衍射**。

**惠根斯 - 菲涅尔原理（Huygens-Fresnel Principle）**可以解释这个现象。当光波遇到跟它波长相仿的障碍时，会 “绕弯” 传播，绕到了障碍后面：

![](1679148481572.png)

在纳米维度，当光波传播到光学平坦的表面时，惠根斯 - 菲涅尔原理同样适用。当与入射光碰撞后，每个粒子都会发射球面波，有些强有些弱。这些不平行的球面波结合起来，就会形成复杂的波形，在很多方向发射不一样的光：

![](1679148481612.png)

纳米几何体（Namogeometry）越小，越少光波被衍射。入射光与单个原子碰撞后，少部分会被衍射。

现在聊回几何和射线光学，它们都是更简化的并且广泛应用于计算机图形领域。其中一种简化方法是忽略纳米级别的不规则和衍射，将光学平坦的表面当成完全平坦。由此可以应用几何光学的反射和折射定律。

很多材质表面看起来是平滑的，但实际并非如此，对于微观几何体（Microgeometry），它们一样凹凸不平：

![](1679148481843.png)

上图的上部分由于物体更加平滑，所以在微观的表面看起来更加规则，反射的光也相对规则，宏观表现就是被反射的光更加清晰，反射的画面更容易表达出被反射物体的轮廓；而下部分由于微观更加不规则，反射的光线取向更紊乱，所以宏观表现就是高光变模糊了，被反射的物体看不清楚。

**对于折射进入介质的光，会发生什么呢？**

对这个问题，需要对物质的导电性进行分类，然后分开探讨：

*   **导体（Conductor）**：导体是金属、电解质等导电性强的物质，由于它们的微粒组合特性，会立即吸收折射光形成热运动。
    
    ![](1679148481889.png)
    
*   **绝缘体（Dielectric）**：即电介质，指没有导电性的物质。如前面章节描述的一样，折射光进入除了透明的电介质后，小部分被吸收，相当一部分在介质内部被多次散射并重新射出表面形成漫反射。
    
    ![](1679148481927.png)
    
    重新被散射出介质表面的光线形成了不同的散射距离。散射距离的分布取决于散射微粒的密度和属性。
    
    ![](1679148482178.png)
    
    如下图所示，如果像素尺寸（绿色标识区域）大于散射距离（黄色线段），就可以无视次表面散射效果。
    
    ![](1679148482435.png)
    
    由于忽略次表面散射的效果，所以入射光附近的区域可以当成一个点来处理（下图），采用经典的关照计算方式，比如 Lambert 或 Phong 光照模型。
    
    ![](1679148482457.png)
    
    当成单点处理后，便可以将光照分成两个部分：镜面反射和漫反射（包含了折射、吸收、散射和重新折射回表面的光），见下图：
    
    ![](1679148482521.png)
    
    对于下图中所示的，散射范围（黄色线段）大于像素尺寸（绿色小圆区域），就不能采用上面的简化模型，而需要采取次表面散射（Subsurface scattering）渲染技术。
    
    ![](1679148482560.png)
    
*   **半导体（Semiconductor）**：由于半导体与光交互的特性介于金属和非金属之间，在实际渲染中，常用迪尼斯原则的金属度系数来模拟半导体特性。
    

### **4.7.3 BxDF**

上节讲述了物质和光的不同情况的交互原理，本节将讲述 BxDF 的主要类型。

目前计算机图形渲染领域，基于物理的渲染方式主要有：

*   **辐射度（Radiance）**：计算光源的镜面反射和漫反射占总的辐射能量的比例，从而算出颜色。在实时渲染领域，是最主流的渲染方式。BRDF 大多数都是基于此种方式，包括 Cook-Torrance。
    
*   **光线追踪（Ray Tracing）**：即光线追踪技术，它的做法是将摄像机的位置与渲染纹理的每个像素构造一条光线，从屏幕射出到虚拟世界，每遇到几何体就计算一次光照，同时损耗一定比例的能量，继续分拆成反射光线和折射光线，如此递归地计算，直到初始光线及其所有分拆的光线能量耗尽为止。
    
    ![](1679148482590.png)
    
    由于这种方式开销非常大，特别是场景复杂度高的情况，所以常用于离线渲染，如影视制作、动漫制作、设计行业等。
    
    近年来，随着 NVIDIA 的 RTX 系列和 AMD 的 RX 系列显卡问世，它们的共同特点是硬件级别支持光线追踪，从而将高大上的光线追踪技术带入了实时渲染领域。
    
*   **路径追踪（Path Tracing）**：实际上路径追踪是光线追踪的一种改进方法。它与光线追踪不同的是，引入了蒙特卡洛方法，利用 BRDF 随机跟踪多条反射光线，随后根据这些光线的贡献计算该点的颜色值。
    
    这种方法更加真实（下图），但同时也更加耗时，通常用于离线渲染领域。
    
    ![](1679148482649.png)
    

上章已经详细描述了基于辐射度的 Cook-Torrance 的 BRDF 模型的理论和实现。实际上，Cook-Torrance 模型在整个渲染体系中，只是冰山一角。下面是 BRDF 光照模型体系：

![](1679148482675.png)

限于篇幅和本文主题，下面将介绍基于辐射度方式的 BxDF 光照模型。

BxDF 可细分为以下几类：

*   **BRDF**（双向反射分布函数，Bidirectional Reflectance Distribution Function）：用于非透明材质的光照计算。Cook-Torrance 就是 BRDF 的一种实现方式，上章详述过，不多说。
    
*   **BTDF**（双向透射分布函数，Bidirectional Transmission Distribution Function）：用于透明材质的光照计算。折射光穿透介质进入另外一种介质时的光照计算模型，只对有透明度的介质适用。
    
*   **BSDF**（双向散射分布函数，Bidirectional Scattering Distribution Function）：实际上是 BRDF 和 BTDF 的综合体：
    
    ![](1679148482726.png)
    
    简单地用公式表达：**BSDF = BRDF + BTDF**。
    
*   **SVBRDF**（空间变化双向反射分布函数，Spatially Varying Bidirectional Reflectance Distribution Function）：将含有双参数的柯西分布替代常规高斯分布引入微面元双向反射分布函数 (BRDF) 模型，同时考虑了目标自身辐射强度的方向依赖性，在此基础上推导了长波红外偏振的数学模型，并在合理范围内对模型做简化与修正使之适用于仿真渲染。
    
*   **BTF**（双向纹理函数，Bidirectional Texture Function）：主要用于模拟非平坦表面，参数跟 SVBRDF 一致。但是，BTF 包含了非局部的散射效果，比如阴影、遮挡、相互反射、次表面散射等。用 BTF 给表面的每个点建模的方法被成为 **Apparent BRDFs**（表面双向反射分布函数）。
    
*   **SSS**（次表面散射，也称 3S，Subsurface Scattering）：它是模拟光进入半透明或者有一定透明深度的材质（皮肤、玉石、大理石、蜡烛等）后，在内部散射开来，然后又通过表面反射出来的光照模拟技术。下面是用 SSS 模拟的玉石效果图：
    
    ![](1679148482775.png)
    
    关于次表面散射方面的研究，比较好的是 Jensen 的文章《A Practical Model for Subsurface Light Transport》，该文提出了一个较为全面的 SSS 模型，将它建模成一个双向表面散射反射分布函数 (BSSRDF)。
    
*   **BSSRDF**（双向表面散射分布函数，Bidirectional Surface Scattering Reflectance Distribution Function）：它常用于模拟透明材质，目前是主流技术。它和 BRDF 的不同之处在于，BSSRDF 可以再现光线透射材质的效果，还可以指定不同的光线入射位置和出射位置：
    
    ![](1679148482799.png)
    

从上面可以看出，BxDF 的形式多种多样，但由于它们都是基于辐射度的光照模型，所以最终可以用以下公式抽象出来：

$$L_o(p,\omega_o) = \int\limits_{\Omega} f_r(p,\omega_i,\omega_o) L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

用更简洁的方式描述，入射光$\omega_i$在 $p$点的颜色的计算公式：

$$\begin{eqnarray*} p点颜色 & = & 光源颜色 \times 材质颜色 \times 反射系数 \times 光照函数 \\ 光照函数 & = & f(n_{法线}, \omega_{光源方向}, v_{视点方向}) \end{eqnarray*}$$

由于篇幅问题，本文不会对 BTDF、BSDF、SSS、BSSRDF 进行详细讨论，有兴趣的可以另外找资料了解。笔者以后也可能另外开辟专题探讨。

本章末，值得一提的是，BRDF 最终的光照计算结果是几何函数和油墨算法（ink-selection）结合的结果。

![](1679148483056.png)

其中油墨算法描述了如何计算各颜色分量的反射率，可参看论文[《A Multi-Ink Color-Separation Algorithm Maximizing Color Constancy》](https://pdfs.semanticscholar.org/9e56/8b13ea51ca3c669186624566f672eb547857.pdf)。

# **五. 高阶：PBR 关联理论和推导**

上章详细介绍了 PBR 的核心原理：光学理论和物质交互的原理。

这章将深入介绍 PBR 核心原理相关的理论，并对其中一些公式做推导或详细阐述，使读者对 PBR 的关联技术和原理有更彻底的理解。

主要面向：

*   高阶程序员
*   想全面且透彻了解 PBR 底层原理及理论的人

**数学物理（ Mathematical Physics）**是一门数学和物理相结合的学科，意图将物理现象和理论用数学公式或理论表达出来。本章主要是跟数学物理相关的内容。

## **5.1 微积分（Calculus）**

由于微积分可以解决很多物理学上的现象或问题，比如光照辐射度量、电磁场、量场等等，所以很有必要重温一下微积分的基础知识。

微积分是数学的一门基础学科，是高等数学中研究函数的一个分支。它包含的主要内容有：

*   **微分学**（Differentiation）：它是一套描述函数变化率的理论及求导数的运算，将函数、速度、加速度和曲线斜率归集起来，以便用一套统一通用的符号进行讨论和计算。包含的内容：
    *   极限理论（Limit Theory）
    *   导数（Derivative）
    *   微分（Differentiation）
*   **积分学**（Integration）：为定义和计算面积、体积等提供一套通用的方法。包含的内容：
    *   定积分（Definite integration）
    *   不定积分（Indefinite integration）

### **5.1.1 微分（Differentiation）**

#### **5.1.1.1 函数极限（Functional Limit）**

*   **极限的概念**：设 $y=f(x)$是给定函数，如果自变量 $x$在定义域内按照某种趋势变化时，若 $x\to a$时，函数值与某个常数 $A$可无限接近（甚至相等），则称 $f(x)$在此变化过程中**有极限**，$A$为其**极限**，记做 $\lim_{x \to a}f(x) = A$，否则称 $f(x)$在此过程中**无极限**。
    
    例如，函数 $y=xf(x)$，它的曲线如下图，从中可以观察到，当 $x$无限趋近于 0 时，$y$也趋近于 0，用公式表达就是：$\lim_{x \to 0}xf(x) = 0$。
    

![](1679148483091.png)

*   **极限的性质**（假设 $\lim{f(x)}$和 $\lim{g(x)}$均存在，$C$为常数）：
    
    *   $\lim C = C$
        
    *   $\lim_{x \to a}x = a$
        
    *   $\lim Cf(x) = Cf(x)$
        
    *   $\lim[ f(x)\pm g(x)] = \lim f(x) \pm \lim g(x)$
        
    *   $\lim[ f(x) g(x)] = \lim f(x) \lim g(x)$
        
    *   $\lim\frac{f(x)}{ g(x)} = \frac{\lim f(x)}{ \lim g(x)}$（$\lim g(x) \ne 0$）
        
    *   $\lim_{x \to a}f(x) = A \iff f(a+0) = f(a-0) = A$（极限与左、右极限的关系）
        
*   **重要极限**：
    
    *   重要极限 1，跟三角函数相关：
    
    $$\lim_{x \to 0} \frac{\sin{x}}{x} = 1$$
    
    ​ 其中 $\frac{\sin{x}}{x}$曲线图如下：
    
    ![](1679148483127.png)
    
    ​ 可以将 $x$扩展到 $f(x)$：
    
    $$\lim_{x \to 0} \frac{\sin{f(x)}}{f(x)} = 1$$
    
    ​ 利用这一重要极限，可以求得一系列涉及三角函数的极限。
    
    *   重要极限 2，跟自然常数 $e$相关，有两种表达形式：
    
    $$\lim_{x \to \infty} (1+\frac{1}{x})^x = e$$
    
    $$\lim_{x \to 0} (1+x)^\frac{1}{x} = e$$
    
    ​ 其中 $e$是无理数，$e=2.718281828459045…$。
    
    ​ 可以将 $x$扩展到 $f(x)$：

    $$\begin{eqnarray*} \lim_{x \to \infty} (1+\frac{1}{f(x)})^{f(x)} &=& e \; (\lim_{x \to a}f(x) = \infty) \\
    
    \end{eqnarray*}$$
    

#### **5.1.1.2 导数（Derivative）**

*   **导数的定义**：
    
    设 $y=f(x)$在 $x_0$和 $x_0+\triangle x$有定义，则函数有增量$\triangle y = f(x_0 + \triangle x)-f(x_0)$。如果极限
    $$\lim_{\triangle x\to 0}\frac{\triangle y}{\triangle x} = \lim_{\triangle x\to 0}\frac{f(x_0+\triangle x)-f(x)}{\triangle x}$$
    
    存在，则称 $y=f(x)$在 $x_0$可导，称极限值为 $f(x)$在 $x_0$处的**导数**（其实就是斜率、变化率），可记为以下几种形式：
    
    $$\begin{eqnarray*} f'(x_0) \\ \frac{df}{dx}|_{x=x_0}\\ y'|_{x=x_0}\\ \frac{dy}{dx}|_{x=x_0} \end{eqnarray*}$$
    
*   **求导步骤**：
    
    1.  对给定的$\triangle x$，求出$\triangle y = f(x_0 + \triangle x)-f(x_0)$。
    2.  计算$\frac{\triangle y}{\triangle x}$并化简。
    3.  求 $\lim_{\triangle x\to 0}\frac{\triangle y}{\triangle x}$。
    
    比如，求 $y=f(x)=x^2$的导数，分为以下步骤：
    
    $$\begin{eqnarray*} \triangle y &=& (x+\triangle x)^2 - \triangle x = 2x\triangle x + (\triangle)^2 \\ \frac{\triangle y}{\triangle x} &=& 2x + \triangle x \\ \lim_{\triangle x\to 0}\frac{\triangle y}{\triangle x}&=& 2x + \triangle x = 2x \end{eqnarray*}$$
    
    更抽象地，对幂函数 $f(x)=x^\alpha$的导数：
    
    $$(x^\alpha)' = \alpha \cdot x^{\alpha - 1}$$
    
*   **求导法则**：
    
    *   **基本运算**求导法则：设 $u=u(x)$和 $v=v(x)$是 $x$的可导函数，且 $v(x)\ne 0$，则：
        
        $$\begin{eqnarray*} (u\pm v)' &=& u' \pm v' \\ (uv)' &=& u'v + uv' \\ (\frac{u}{v})' &=& \frac{u'v-uv'}{v^2} \end{eqnarray*}$$
        
        设 $C$为常数，则 $[Cv(x)]' = Cv'(x)$。
        
        其它常见函数的导数：
        
        $$\begin{eqnarray*} (\ln x)' &=& \frac{1}{x} \\ (\log_ax)' &=& \frac{1}{x\ln a} \\ (\sin x)' &=& \cos x \\ (\cos x)' &=& -\sin x \\ (\tan x)' &=& \frac{1}{cos^2x} = \sec^2 x \\ (\cot x)' &=& -\csc^2 x \\ (\csc x)' &=& -\csc x \cot x \end{eqnarray*}$$
        
    *   **复合函数**求导法则：设 $y=f[h(x)]$是由 $y=f(u)$和 $u=h(x)$组成的复合函数，并且设 $u=h(x)$可导，$y=f(u)$也可导，则复合函数 $y=f[h(x)]$的导数为：
        
        $$\frac{dy}{dx} = \frac{dy}{du}\cdot \frac{du}{dx}$$
        
        还有其它几种等价表示形式：
        
        $$\begin{eqnarray*} y'_x &=& y'_u \cdot u'_x \\ (f[h(x)])' &=& f'(u)|_{u=h(x)}h'(x) \end{eqnarray*}$$
        
        以上法则也叫**链式求导法则**，它表明：复合函数的导数等于函数对中间变量的导数乘以中间变量  
        对自变量的导数。
        
    *   **反函数**的导数：反函数的导数等于原来函数导数的倒数。
        
        更具体地，设单调函数 $y=f(x)$，则它的反函数是 $x=f^{-1}(y)$，则有：
        
        $$f'(x) = \frac{1}{f^{-1}(y)}或\frac{dx}{dy} = \frac{1}{\frac{dy}{dx}}$$
        
    *   **隐函数**的导数：如果方程 $F(x,y)=0$确定了 $y$是 $x$的函数，那么，这样的函数叫做**隐函数**。
        
        若隐函数 $y$关于 $x$可导，则可根据复合函数求导法则求出函数 $y$对 $x$的导数。
        
*   **高阶导数**：
    
    一般情况下，函数 $y=f(x)$的导数 $y'=f'(x)$仍然是 $x$的函数。如果 $f'(x)$仍然可导，则把 $f'(x)$的导数称为函数 $y=f(x)$的**二阶导数**，记作 $f''(x)$或 $y''$，也可用以下表达式：
    
    $$y'' = (y')^{'} 或 f''(x) = (f'(x))'$$
    
    若二阶导数仍然可导，则二阶导数的导数称为函数 $y=f(x)$的**三阶导数**，记作 $f'''(x)$或 $y'''$。
    
    二阶及以上的导数统称为**高阶导数**。
    

#### **5.1.1.3 微分（Differentiation）**

*   **微分的定义**：
    
    设函数 $y=f(x)$在点 $x$处可导，则把函数 $y=f(x)$在 $x$处的导数 $f'(x)$与自变量在 $x$处的增量$\triangle x$之积 $f'(x) \triangle x$称为函数 $y=f(x)$在点 $x$处的**微分**，记做 $dy$，即 $dy=f'(x)\triangle x$，这时称函数 $y=f(x)$在点 $x$处**可微**。
    
    对自变量 $x$的微分，可以认为是对函数 $y=x$的微分，有以下等式：
    
    $$dy = dx = x'\triangle x = \triangle x$$
    
    故而，$y=f(x)$的微分又可记为 $dy=f'(x)dx$，推导出：$\frac{dy}{dx} = f'(x)$。
    
    也就是说，函数 $y=f(x)$的微分 $dy$与自变量的微分 $dx$之商是函数 $y=f(x)$的导数。因此，函数的导数也叫**微商**（注意，跟朋友圈的微商概念不一样 o_o!!）。
    
    也就是说，函数 $y=f(x)$ 在 $x$处**可微与可导等价**。
    
*   **微分的几何意义**：
    
    如下图所示，$MT$是曲线 $y=f(x)$上的点 $M$处的切线，设它与 $x$轴正向的夹角为$\alpha$，则 $QP = MQ \cdot \tan \alpha = \triangle x \cdot f'(x_0)$，所以 $dy = QP$，即函数 $y=f(x)$在 $x$_0 处相对于$\triangle x$的微分 $dy=f'(x)\triangle x$。
    
    也就是说，微分的几何意义是曲线上点的切线的纵坐标的改变量。
    
    ![](1679148483151.png)
    
*   **微分公式和法则**：
    
    由于微分 $dy=f'(x)dx$，也就是说，只要求出函数的导数，即可求得对应的微分。
    
    因而求导数和求微分的方法统称为**微分法**。
    
    导数和微分之间的公式和法则**高度一致**：
    
    <table><thead><tr><th>导数公式</th><th>微分公式</th></tr></thead><tbody><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-443-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>C</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mn>0</mn></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-4950" style="width: 4.689em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.717em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1003.66em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-4951"><span class="mo" id="MathJax-Span-4952" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-4953" style="font-family: MathJax_Math-italic;">C<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="msup" id="MathJax-Span-4954"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-4955" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-4956" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-4957" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mn" id="MathJax-Span-4958" style="font-family: MathJax_Main; padding-left: 0.289em;">0</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>C</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mn>0</mn></math></span></span><script type="math/tex" id="MathJax-Element-443">(C)' = 0</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-444-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>C</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mn>0</mn></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-4959" style="width: 4.974em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.946em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1003.89em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-4960"><span class="mi" id="MathJax-Span-4961" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-4962" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-4963" style="font-family: MathJax_Math-italic;">C<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.06em;"></span></span><span class="mo" id="MathJax-Span-4964" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-4965" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mn" id="MathJax-Span-4966" style="font-family: MathJax_Main; padding-left: 0.289em;">0</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>C</mi><mo stretchy="false">)</mo><mo>=</mo><mn>0</mn></math></span></span><script type="math/tex" id="MathJax-Element-444">d(C) = 0</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-445-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><msup><mi>x</mi><mi>u</mi></msup><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mi>u</mi><msup><mi>x</mi><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>u</mi><mo>&amp;#x2212;</mo><mn>1</mn></mrow></msup></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-4967" style="width: 7.546em; display: inline-block;"><span style="display: inline-block; position: relative; width: 6.003em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1006em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-4968"><span class="mo" id="MathJax-Span-4969" style="font-family: MathJax_Main;">(</span><span class="msubsup" id="MathJax-Span-4970"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-4971" style="font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.574em;"><span class="mi" id="MathJax-Span-4972" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="msup" id="MathJax-Span-4973"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-4974" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-4975" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-4976" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mi" id="MathJax-Span-4977" style="font-family: MathJax_Math-italic; padding-left: 0.289em;">u</span><span class="msubsup" id="MathJax-Span-4978"><span style="display: inline-block; position: relative; width: 1.946em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-4979" style="font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.574em;"><span class="texatom" id="MathJax-Span-4980"><span class="mrow" id="MathJax-Span-4981"><span class="mi" id="MathJax-Span-4982" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span class="mo" id="MathJax-Span-4983" style="font-size: 70.7%; font-family: MathJax_Main;">−</span><span class="mn" id="MathJax-Span-4984" style="font-size: 70.7%; font-family: MathJax_Main;">1</span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><msup><mi>x</mi><mi>u</mi></msup><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mi>u</mi><msup><mi>x</mi><mrow class="MJX-TeXAtom-ORD"><mi>u</mi><mo>−</mo><mn>1</mn></mrow></msup></math></span></span><script type="math/tex" id="MathJax-Element-445">(x^u)' = ux^{u-1}</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-446-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><msup><mi>x</mi><mi>u</mi></msup><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mi>u</mi><msup><mi>x</mi><mrow class=&quot;MJX-TeXAtom-ORD&quot;><mi>u</mi><mo>&amp;#x2212;</mo><mn>1</mn></mrow></msup><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-4985" style="width: 9.146em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.317em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1007.26em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-4986"><span class="mi" id="MathJax-Span-4987" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-4988" style="font-family: MathJax_Main;">(</span><span class="msubsup" id="MathJax-Span-4989"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-4990" style="font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.574em;"><span class="mi" id="MathJax-Span-4991" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-4992" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-4993" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mi" id="MathJax-Span-4994" style="font-family: MathJax_Math-italic; padding-left: 0.289em;">u</span><span class="msubsup" id="MathJax-Span-4995"><span style="display: inline-block; position: relative; width: 1.946em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-4996" style="font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.574em;"><span class="texatom" id="MathJax-Span-4997"><span class="mrow" id="MathJax-Span-4998"><span class="mi" id="MathJax-Span-4999" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span class="mo" id="MathJax-Span-5000" style="font-size: 70.7%; font-family: MathJax_Main;">−</span><span class="mn" id="MathJax-Span-5001" style="font-size: 70.7%; font-family: MathJax_Main;">1</span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mi" id="MathJax-Span-5002" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5003" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><msup><mi>x</mi><mi>u</mi></msup><mo stretchy="false">)</mo><mo>=</mo><mi>u</mi><msup><mi>x</mi><mrow class="MJX-TeXAtom-ORD"><mi>u</mi><mo>−</mo><mn>1</mn></mrow></msup><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-446">d(x^u) = ux^{u-1}dx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-447-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><msub><mi>log</mi><mi>a</mi></msub><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mfrac><mn>1</mn><mrow><mi>x</mi><mi>ln</mi><mo>&amp;#x2061;</mo><mi>a</mi></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5004" style="width: 8.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 6.974em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1006.97em, 2.86em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5005"><span class="mo" id="MathJax-Span-5006" style="font-family: MathJax_Main;">(</span><span class="msubsup" id="MathJax-Span-5007"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px;"><span style="position: absolute; clip: rect(3.146em, 1001.26em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5008" style="font-family: MathJax_Main;">log</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -3.769em; left: 1.26em;"><span class="mi" id="MathJax-Span-5009" style="font-size: 70.7%; font-family: MathJax_Math-italic;">a</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5010"></span><span class="mi" id="MathJax-Span-5011" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5012"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5013" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5014" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5015" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5016" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.831em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5017" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1001.66em, 4.174em, -999.997em); top: -3.597em; left: 50%; margin-left: -0.854em;"><span class="mrow" id="MathJax-Span-5018"><span class="mi" id="MathJax-Span-5019" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span class="mi" id="MathJax-Span-5020" style="font-size: 70.7%; font-family: MathJax_Main; padding-left: 0.231em;">ln</span><span class="mo" id="MathJax-Span-5021" style="font-size: 70.7%;"></span><span class="mi" id="MathJax-Span-5022" style="font-size: 70.7%; font-family: MathJax_Math-italic; padding-left: 0.231em;">a</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.83em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.831em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.568em; border-left: 0px solid; width: 0px; height: 1.789em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><msub><mi>log</mi><mi>a</mi></msub><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mfrac><mn>1</mn><mrow><mi>x</mi><mi>ln</mi><mo>⁡</mo><mi>a</mi></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-447">(\log_a x)' = \frac{1}{x\ln a}</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-448-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><msub><mi>log</mi><mi>a</mi></msub><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mfrac><mrow><mi>d</mi><mi>x</mi></mrow><mrow><mi>x</mi><mi>ln</mi><mo>&amp;#x2061;</mo><mi>a</mi></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5023" style="width: 9.031em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.203em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1007.2em, 2.86em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5024"><span class="mi" id="MathJax-Span-5025" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5026" style="font-family: MathJax_Main;">(</span><span class="msubsup" id="MathJax-Span-5027"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px;"><span style="position: absolute; clip: rect(3.146em, 1001.26em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5028" style="font-family: MathJax_Main;">log</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -3.769em; left: 1.26em;"><span class="mi" id="MathJax-Span-5029" style="font-size: 70.7%; font-family: MathJax_Math-italic;">a</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5030"></span><span class="mi" id="MathJax-Span-5031" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5032" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5033" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5034" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.831em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1000.75em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.397em;"><span class="mrow" id="MathJax-Span-5035"><span class="mi" id="MathJax-Span-5036" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5037" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1001.66em, 4.174em, -999.997em); top: -3.597em; left: 50%; margin-left: -0.854em;"><span class="mrow" id="MathJax-Span-5038"><span class="mi" id="MathJax-Span-5039" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span class="mi" id="MathJax-Span-5040" style="font-size: 70.7%; font-family: MathJax_Main; padding-left: 0.231em;">ln</span><span class="mo" id="MathJax-Span-5041" style="font-size: 70.7%;"></span><span class="mi" id="MathJax-Span-5042" style="font-size: 70.7%; font-family: MathJax_Math-italic; padding-left: 0.231em;">a</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.83em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.831em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.568em; border-left: 0px solid; width: 0px; height: 1.789em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><msub><mi>log</mi><mi>a</mi></msub><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mfrac><mrow><mi>d</mi><mi>x</mi></mrow><mrow><mi>x</mi><mi>ln</mi><mo>⁡</mo><mi>a</mi></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-448">d(\log_a x) = \frac{dx}{x\ln a}</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-449-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>ln</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mfrac><mn>1</mn><mi>x</mi></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5043" style="width: 5.946em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.746em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1004.75em, 2.803em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5044"><span class="mo" id="MathJax-Span-5045" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5046" style="font-family: MathJax_Main;">ln</span><span class="mo" id="MathJax-Span-5047"></span><span class="mi" id="MathJax-Span-5048" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5049"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5050" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5051" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5052" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5053" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 0.517em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5054" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.654em; left: 50%; margin-left: -0.226em;"><span class="mi" id="MathJax-Span-5055" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1000.52em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 0.517em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.496em; border-left: 0px solid; width: 0px; height: 1.718em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>ln</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mfrac><mn>1</mn><mi>x</mi></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-449">(\ln x)' = \frac{1}{x}</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-450-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>ln</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mfrac><mn>1</mn><mi>x</mi></mfrac><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5056" style="width: 7.603em; display: inline-block;"><span style="display: inline-block; position: relative; width: 6.06em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1006em, 2.803em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5057"><span class="mi" id="MathJax-Span-5058" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5059" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5060" style="font-family: MathJax_Main;">ln</span><span class="mo" id="MathJax-Span-5061"></span><span class="mi" id="MathJax-Span-5062" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5063" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5064" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5065" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 0.517em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5066" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.654em; left: 50%; margin-left: -0.226em;"><span class="mi" id="MathJax-Span-5067" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1000.52em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 0.517em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span><span class="mi" id="MathJax-Span-5068" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5069" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.496em; border-left: 0px solid; width: 0px; height: 1.718em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>ln</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mfrac><mn>1</mn><mi>x</mi></mfrac><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-450">d(\ln x) = \frac{1}{x}dx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-451-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><msup><mi>a</mi><mi>x</mi></msup><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><msup><mi>a</mi><mi>x</mi></msup><mi>ln</mi><mo>&amp;#x2061;</mo><mi>a</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5070" style="width: 7.831em; display: inline-block;"><span style="display: inline-block; position: relative; width: 6.231em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1006.23em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5071"><span class="mo" id="MathJax-Span-5072" style="font-family: MathJax_Main;">(</span><span class="msubsup" id="MathJax-Span-5073"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5074" style="font-family: MathJax_Math-italic;">a</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.517em;"><span class="mi" id="MathJax-Span-5075" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="msup" id="MathJax-Span-5076"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5077" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5078" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5079" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="msubsup" id="MathJax-Span-5080" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5081" style="font-family: MathJax_Math-italic;">a</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.517em;"><span class="mi" id="MathJax-Span-5082" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mi" id="MathJax-Span-5083" style="font-family: MathJax_Main; padding-left: 0.174em;">ln</span><span class="mo" id="MathJax-Span-5084"></span><span class="mi" id="MathJax-Span-5085" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">a</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><msup><mi>a</mi><mi>x</mi></msup><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><msup><mi>a</mi><mi>x</mi></msup><mi>ln</mi><mo>⁡</mo><mi>a</mi></math></span></span><script type="math/tex" id="MathJax-Element-451">(a^x)' = a^x\ln a</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-452-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><msup><mi>a</mi><mi>x</mi></msup><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><msup><mi>a</mi><mi>x</mi></msup><mi>ln</mi><mo>&amp;#x2061;</mo><mi>a</mi><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5086" style="width: 9.431em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.546em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1007.49em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5087"><span class="mi" id="MathJax-Span-5088" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5089" style="font-family: MathJax_Main;">(</span><span class="msubsup" id="MathJax-Span-5090"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5091" style="font-family: MathJax_Math-italic;">a</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.517em;"><span class="mi" id="MathJax-Span-5092" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5093" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5094" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="msubsup" id="MathJax-Span-5095" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5096" style="font-family: MathJax_Math-italic;">a</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.517em;"><span class="mi" id="MathJax-Span-5097" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mi" id="MathJax-Span-5098" style="font-family: MathJax_Main; padding-left: 0.174em;">ln</span><span class="mo" id="MathJax-Span-5099"></span><span class="mi" id="MathJax-Span-5100" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">a</span><span class="mi" id="MathJax-Span-5101" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5102" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><msup><mi>a</mi><mi>x</mi></msup><mo stretchy="false">)</mo><mo>=</mo><msup><mi>a</mi><mi>x</mi></msup><mi>ln</mi><mo>⁡</mo><mi>a</mi><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-452">d(a^x) = a^x\ln a dx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-453-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>sin</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mi>cos</mi><mo>&amp;#x2061;</mo><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5103" style="width: 8.174em; display: inline-block;"><span style="display: inline-block; position: relative; width: 6.517em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1006.46em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5104"><span class="mo" id="MathJax-Span-5105" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5106" style="font-family: MathJax_Main;">sin</span><span class="mo" id="MathJax-Span-5107"></span><span class="mi" id="MathJax-Span-5108" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5109"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5110" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5111" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5112" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mi" id="MathJax-Span-5113" style="font-family: MathJax_Main; padding-left: 0.289em;">cos</span><span class="mo" id="MathJax-Span-5114"></span><span class="mi" id="MathJax-Span-5115" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>sin</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mi>cos</mi><mo>⁡</mo><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-453">(\sin x)' = \cos x</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-454-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>sin</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mi>cos</mi><mo>&amp;#x2061;</mo><mi>x</mi><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5116" style="width: 9.831em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.831em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1007.77em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5117"><span class="mi" id="MathJax-Span-5118" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5119" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5120" style="font-family: MathJax_Main;">sin</span><span class="mo" id="MathJax-Span-5121"></span><span class="mi" id="MathJax-Span-5122" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5123" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5124" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mi" id="MathJax-Span-5125" style="font-family: MathJax_Main; padding-left: 0.289em;">cos</span><span class="mo" id="MathJax-Span-5126"></span><span class="mi" id="MathJax-Span-5127" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5128" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5129" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>sin</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mi>cos</mi><mo>⁡</mo><mi>x</mi><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-454">d(\sin x) = \cos xdx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-455-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>cos</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mo>&amp;#x2212;</mo><mi>sin</mi><mo>&amp;#x2061;</mo><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5130" style="width: 9.317em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.431em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1007.37em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5131"><span class="mo" id="MathJax-Span-5132" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5133" style="font-family: MathJax_Main;">cos</span><span class="mo" id="MathJax-Span-5134"></span><span class="mi" id="MathJax-Span-5135" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5136"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5137" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5138" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5139" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5140" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="mi" id="MathJax-Span-5141" style="font-family: MathJax_Main; padding-left: 0.174em;">sin</span><span class="mo" id="MathJax-Span-5142"></span><span class="mi" id="MathJax-Span-5143" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>cos</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mo>−</mo><mi>sin</mi><mo>⁡</mo><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-455">(\cos x)' = -\sin x</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-456-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>cos</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mo>&amp;#x2212;</mo><mi>sin</mi><mo>&amp;#x2061;</mo><mi>x</mi><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5144" style="width: 10.974em; display: inline-block;"><span style="display: inline-block; position: relative; width: 8.746em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1008.69em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5145"><span class="mi" id="MathJax-Span-5146" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5147" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5148" style="font-family: MathJax_Main;">cos</span><span class="mo" id="MathJax-Span-5149"></span><span class="mi" id="MathJax-Span-5150" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5151" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5152" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5153" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="mi" id="MathJax-Span-5154" style="font-family: MathJax_Main; padding-left: 0.174em;">sin</span><span class="mo" id="MathJax-Span-5155"></span><span class="mi" id="MathJax-Span-5156" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5157" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5158" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>cos</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mo>−</mo><mi>sin</mi><mo>⁡</mo><mi>x</mi><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-456">d(\cos x) = -\sin xdx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-457-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>tan</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><msup><mi>sec</mi><mn>2</mn></msup><mo>&amp;#x2061;</mo><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5159" style="width: 8.86em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.089em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1007.03em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5160"><span class="mo" id="MathJax-Span-5161" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5162" style="font-family: MathJax_Main;">tan</span><span class="mo" id="MathJax-Span-5163"></span><span class="mi" id="MathJax-Span-5164" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5165"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5166" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5167" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5168" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="msubsup" id="MathJax-Span-5169" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1001.26em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5170" style="font-family: MathJax_Main;">sec</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 1.26em;"><span class="mn" id="MathJax-Span-5171" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5172"></span><span class="mi" id="MathJax-Span-5173" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>tan</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><msup><mi>sec</mi><mn>2</mn></msup><mo>⁡</mo><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-457">(\tan x)' = \sec^2 x</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-458-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mo stretchy=&quot;false&quot;>(</mo><mi>tan</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><msup><mi>sec</mi><mn>2</mn></msup><mo>&amp;#x2061;</mo><mi>x</mi><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5174" style="width: 11.031em; display: inline-block;"><span style="display: inline-block; position: relative; width: 8.803em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1008.75em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5175"><span class="mi" id="MathJax-Span-5176" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5177" style="font-family: MathJax_Main;">(</span><span class="mo" id="MathJax-Span-5178" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5179" style="font-family: MathJax_Main;">tan</span><span class="mo" id="MathJax-Span-5180"></span><span class="mi" id="MathJax-Span-5181" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5182" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5183" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="msubsup" id="MathJax-Span-5184" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1001.26em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5185" style="font-family: MathJax_Main;">sec</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 1.26em;"><span class="mn" id="MathJax-Span-5186" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5187"></span><span class="mi" id="MathJax-Span-5188" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5189" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5190" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mo stretchy="false">(</mo><mi>tan</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><msup><mi>sec</mi><mn>2</mn></msup><mo>⁡</mo><mi>x</mi><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-458">d((\tan x) = \sec^2 xdx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-459-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>cot</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mo>&amp;#x2212;</mo><msup><mi>csc</mi><mn>2</mn></msup><mo>&amp;#x2061;</mo><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5191" style="width: 9.946em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.946em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1007.89em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5192"><span class="mo" id="MathJax-Span-5193" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5194" style="font-family: MathJax_Main;">cot</span><span class="mo" id="MathJax-Span-5195"></span><span class="mi" id="MathJax-Span-5196" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5197"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5198" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5199" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5200" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5201" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="msubsup" id="MathJax-Span-5202" style="padding-left: 0.174em;"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1001.26em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5203" style="font-family: MathJax_Main;">csc</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 1.26em;"><span class="mn" id="MathJax-Span-5204" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5205"></span><span class="mi" id="MathJax-Span-5206" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>cot</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mo>−</mo><msup><mi>csc</mi><mn>2</mn></msup><mo>⁡</mo><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-459">(\cot x)' = - \csc^2 x</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-460-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>cot</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mo>&amp;#x2212;</mo><msup><mi>csc</mi><mn>2</mn></msup><mo>&amp;#x2061;</mo><mi>x</mi><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5207" style="width: 11.603em; display: inline-block;"><span style="display: inline-block; position: relative; width: 9.26em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.26em, 1009.2em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5208"><span class="mi" id="MathJax-Span-5209" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5210" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5211" style="font-family: MathJax_Main;">cot</span><span class="mo" id="MathJax-Span-5212"></span><span class="mi" id="MathJax-Span-5213" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5214" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5215" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5216" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="msubsup" id="MathJax-Span-5217" style="padding-left: 0.174em;"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1001.26em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5218" style="font-family: MathJax_Main;">csc</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 1.26em;"><span class="mn" id="MathJax-Span-5219" style="font-size: 70.7%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5220"></span><span class="mi" id="MathJax-Span-5221" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5222" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5223" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.504em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>cot</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mo>−</mo><msup><mi>csc</mi><mn>2</mn></msup><mo>⁡</mo><mi>x</mi><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-460">d(\cot x) = - \csc^2 xdx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-461-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>sec</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mi>sec</mi><mo>&amp;#x2061;</mo><mi>x</mi><mi>tan</mi><mo>&amp;#x2061;</mo><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5224" style="width: 11.089em; display: inline-block;"><span style="display: inline-block; position: relative; width: 8.86em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1008.8em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5225"><span class="mo" id="MathJax-Span-5226" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5227" style="font-family: MathJax_Main;">sec</span><span class="mo" id="MathJax-Span-5228"></span><span class="mi" id="MathJax-Span-5229" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5230"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5231" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5232" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5233" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mi" id="MathJax-Span-5234" style="font-family: MathJax_Main; padding-left: 0.289em;">sec</span><span class="mo" id="MathJax-Span-5235"></span><span class="mi" id="MathJax-Span-5236" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5237" style="font-family: MathJax_Main; padding-left: 0.174em;">tan</span><span class="mo" id="MathJax-Span-5238"></span><span class="mi" id="MathJax-Span-5239" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>sec</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mi>sec</mi><mo>⁡</mo><mi>x</mi><mi>tan</mi><mo>⁡</mo><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-461">(\sec x)'=\sec x \tan x</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-462-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>sec</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mi>sec</mi><mo>&amp;#x2061;</mo><mi>x</mi><mi>tan</mi><mo>&amp;#x2061;</mo><mi>x</mi><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5240" style="width: 12.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 10.174em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1010.12em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5241"><span class="mi" id="MathJax-Span-5242" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5243" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5244" style="font-family: MathJax_Main;">sec</span><span class="mo" id="MathJax-Span-5245"></span><span class="mi" id="MathJax-Span-5246" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5247" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5248" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mi" id="MathJax-Span-5249" style="font-family: MathJax_Main; padding-left: 0.289em;">sec</span><span class="mo" id="MathJax-Span-5250"></span><span class="mi" id="MathJax-Span-5251" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5252" style="font-family: MathJax_Main; padding-left: 0.174em;">tan</span><span class="mo" id="MathJax-Span-5253"></span><span class="mi" id="MathJax-Span-5254" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5255" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5256" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>sec</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mi>sec</mi><mo>⁡</mo><mi>x</mi><mi>tan</mi><mo>⁡</mo><mi>x</mi><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-462">d(\sec x)=\sec x \tan xdx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-463-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>csc</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mo>&amp;#x2212;</mo><mi>csc</mi><mo>&amp;#x2061;</mo><mi>x</mi><mi>cot</mi><mo>&amp;#x2061;</mo><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5257" style="width: 12.174em; display: inline-block;"><span style="display: inline-block; position: relative; width: 9.717em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1009.66em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5258"><span class="mo" id="MathJax-Span-5259" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5260" style="font-family: MathJax_Main;">csc</span><span class="mo" id="MathJax-Span-5261"></span><span class="mi" id="MathJax-Span-5262" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5263"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5264" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5265" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5266" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5267" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="mi" id="MathJax-Span-5268" style="font-family: MathJax_Main; padding-left: 0.174em;">csc</span><span class="mo" id="MathJax-Span-5269"></span><span class="mi" id="MathJax-Span-5270" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5271" style="font-family: MathJax_Main; padding-left: 0.174em;">cot</span><span class="mo" id="MathJax-Span-5272"></span><span class="mi" id="MathJax-Span-5273" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>csc</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mo>−</mo><mi>csc</mi><mo>⁡</mo><mi>x</mi><mi>cot</mi><mo>⁡</mo><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-463">(\csc x)' = -\csc x \cot x</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-464-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>csc</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mo>&amp;#x2212;</mo><mi>csc</mi><mo>&amp;#x2061;</mo><mi>x</mi><mi>cot</mi><mo>&amp;#x2061;</mo><mi>x</mi><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5274" style="width: 13.831em; display: inline-block;"><span style="display: inline-block; position: relative; width: 11.031em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1010.97em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5275"><span class="mi" id="MathJax-Span-5276" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5277" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5278" style="font-family: MathJax_Main;">csc</span><span class="mo" id="MathJax-Span-5279"></span><span class="mi" id="MathJax-Span-5280" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5281" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5282" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5283" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="mi" id="MathJax-Span-5284" style="font-family: MathJax_Main; padding-left: 0.174em;">csc</span><span class="mo" id="MathJax-Span-5285"></span><span class="mi" id="MathJax-Span-5286" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5287" style="font-family: MathJax_Main; padding-left: 0.174em;">cot</span><span class="mo" id="MathJax-Span-5288"></span><span class="mi" id="MathJax-Span-5289" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mi" id="MathJax-Span-5290" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5291" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>csc</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mo>−</mo><mi>csc</mi><mo>⁡</mo><mi>x</mi><mi>cot</mi><mo>⁡</mo><mi>x</mi><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-464">d(\csc x) = -\csc x \cot xdx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-465-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>arcsin</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mfrac><mn>1</mn><msqrt><mn>1</mn><mo>&amp;#x2212;</mo><msup><mi>x</mi><mn>2</mn></msup></msqrt></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5292" style="width: 10.574em; display: inline-block;"><span style="display: inline-block; position: relative; width: 8.46em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1008.46em, 3.203em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5293"><span class="mo" id="MathJax-Span-5294" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5295" style="font-family: MathJax_Main;">arcsin</span><span class="mo" id="MathJax-Span-5296"></span><span class="mi" id="MathJax-Span-5297" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5298"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5299" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5300" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5301" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5302" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 2.46em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5303" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1002.35em, 4.346em, -999.997em); top: -3.426em; left: 50%; margin-left: -1.14em;"><span class="msqrt" id="MathJax-Span-5304"><span style="display: inline-block; position: relative; width: 2.346em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1001.6em, 4.231em, -999.997em); top: -3.997em; left: 0.689em;"><span class="mrow" id="MathJax-Span-5305"><span class="mn" id="MathJax-Span-5306" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span class="mo" id="MathJax-Span-5307" style="font-size: 70.7%; font-family: MathJax_Main;">−</span><span class="msubsup" id="MathJax-Span-5308"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5309" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mn" id="MathJax-Span-5310" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.974em, 1001.6em, 1.317em, -999.997em); top: -1.769em; left: 0.689em;"><span style="display: inline-block; overflow: hidden; vertical-align: -0.054em; border-top: 1.3px solid; width: 1.603em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1000.75em, 4.403em, -999.997em); top: -4.054em; left: 0em;"><span><span style="font-size: 70.7%; font-family: MathJax_Size1;">√</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.46em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.46em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.996em; border-left: 0px solid; width: 0px; height: 2.218em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>arcsin</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mfrac><mn>1</mn><msqrt><mn>1</mn><mo>−</mo><msup><mi>x</mi><mn>2</mn></msup></msqrt></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-465">(\arcsin x)' = \frac{1}{\sqrt{1-x^2}}</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-466-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>arcsin</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mfrac><mn>1</mn><msqrt><mn>1</mn><mo>&amp;#x2212;</mo><msup><mi>x</mi><mn>2</mn></msup></msqrt></mfrac><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5311" style="width: 12.231em; display: inline-block;"><span style="display: inline-block; position: relative; width: 9.774em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1009.72em, 3.203em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5312"><span class="mi" id="MathJax-Span-5313" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5314" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5315" style="font-family: MathJax_Main;">arcsin</span><span class="mo" id="MathJax-Span-5316"></span><span class="mi" id="MathJax-Span-5317" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5318" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5319" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5320" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 2.46em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5321" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1002.35em, 4.346em, -999.997em); top: -3.426em; left: 50%; margin-left: -1.14em;"><span class="msqrt" id="MathJax-Span-5322"><span style="display: inline-block; position: relative; width: 2.346em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1001.6em, 4.231em, -999.997em); top: -3.997em; left: 0.689em;"><span class="mrow" id="MathJax-Span-5323"><span class="mn" id="MathJax-Span-5324" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span class="mo" id="MathJax-Span-5325" style="font-size: 70.7%; font-family: MathJax_Main;">−</span><span class="msubsup" id="MathJax-Span-5326"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5327" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mn" id="MathJax-Span-5328" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.974em, 1001.6em, 1.317em, -999.997em); top: -1.769em; left: 0.689em;"><span style="display: inline-block; overflow: hidden; vertical-align: -0.054em; border-top: 1.3px solid; width: 1.603em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1000.75em, 4.403em, -999.997em); top: -4.054em; left: 0em;"><span><span style="font-size: 70.7%; font-family: MathJax_Size1;">√</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.46em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.46em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span><span class="mi" id="MathJax-Span-5329" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5330" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.996em; border-left: 0px solid; width: 0px; height: 2.218em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>arcsin</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mfrac><mn>1</mn><msqrt><mn>1</mn><mo>−</mo><msup><mi>x</mi><mn>2</mn></msup></msqrt></mfrac><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-466">d(\arcsin x) = \frac{1}{\sqrt{1-x^2}}dx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-467-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>arccos</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mo>&amp;#x2212;</mo><mfrac><mn>1</mn><msqrt><mn>1</mn><mo>&amp;#x2212;</mo><msup><mi>x</mi><mn>2</mn></msup></msqrt></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5331" style="width: 11.66em; display: inline-block;"><span style="display: inline-block; position: relative; width: 9.317em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1009.32em, 3.203em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5332"><span class="mo" id="MathJax-Span-5333" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5334" style="font-family: MathJax_Main;">arccos</span><span class="mo" id="MathJax-Span-5335"></span><span class="mi" id="MathJax-Span-5336" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5337"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5338" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5339" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5340" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5341" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="mfrac" id="MathJax-Span-5342"><span style="display: inline-block; position: relative; width: 2.46em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5343" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1002.35em, 4.346em, -999.997em); top: -3.426em; left: 50%; margin-left: -1.14em;"><span class="msqrt" id="MathJax-Span-5344"><span style="display: inline-block; position: relative; width: 2.346em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1001.6em, 4.231em, -999.997em); top: -3.997em; left: 0.689em;"><span class="mrow" id="MathJax-Span-5345"><span class="mn" id="MathJax-Span-5346" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span class="mo" id="MathJax-Span-5347" style="font-size: 70.7%; font-family: MathJax_Main;">−</span><span class="msubsup" id="MathJax-Span-5348"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5349" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mn" id="MathJax-Span-5350" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.974em, 1001.6em, 1.317em, -999.997em); top: -1.769em; left: 0.689em;"><span style="display: inline-block; overflow: hidden; vertical-align: -0.054em; border-top: 1.3px solid; width: 1.603em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1000.75em, 4.403em, -999.997em); top: -4.054em; left: 0em;"><span><span style="font-size: 70.7%; font-family: MathJax_Size1;">√</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.46em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.46em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.996em; border-left: 0px solid; width: 0px; height: 2.218em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>arccos</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mo>−</mo><mfrac><mn>1</mn><msqrt><mn>1</mn><mo>−</mo><msup><mi>x</mi><mn>2</mn></msup></msqrt></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-467">(\arccos x)' = -\frac{1}{\sqrt{1-x^2}}</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-468-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>arccos</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mo>&amp;#x2212;</mo><mfrac><mn>1</mn><msqrt><mn>1</mn><mo>&amp;#x2212;</mo><msup><mi>x</mi><mn>2</mn></msup></msqrt></mfrac><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5351" style="width: 13.317em; display: inline-block;"><span style="display: inline-block; position: relative; width: 10.631em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1010.57em, 3.203em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5352"><span class="mi" id="MathJax-Span-5353" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5354" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5355" style="font-family: MathJax_Main;">arccos</span><span class="mo" id="MathJax-Span-5356"></span><span class="mi" id="MathJax-Span-5357" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5358" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5359" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5360" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="mfrac" id="MathJax-Span-5361"><span style="display: inline-block; position: relative; width: 2.46em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5362" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1002.35em, 4.346em, -999.997em); top: -3.426em; left: 50%; margin-left: -1.14em;"><span class="msqrt" id="MathJax-Span-5363"><span style="display: inline-block; position: relative; width: 2.346em; height: 0px;"><span style="position: absolute; clip: rect(3.317em, 1001.6em, 4.231em, -999.997em); top: -3.997em; left: 0.689em;"><span class="mrow" id="MathJax-Span-5364"><span class="mn" id="MathJax-Span-5365" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span class="mo" id="MathJax-Span-5366" style="font-size: 70.7%; font-family: MathJax_Main;">−</span><span class="msubsup" id="MathJax-Span-5367"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5368" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mn" id="MathJax-Span-5369" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.974em, 1001.6em, 1.317em, -999.997em); top: -1.769em; left: 0.689em;"><span style="display: inline-block; overflow: hidden; vertical-align: -0.054em; border-top: 1.3px solid; width: 1.603em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span><span style="position: absolute; clip: rect(3.203em, 1000.75em, 4.403em, -999.997em); top: -4.054em; left: 0em;"><span><span style="font-size: 70.7%; font-family: MathJax_Size1;">√</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.46em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.46em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span><span class="mi" id="MathJax-Span-5370" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5371" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.996em; border-left: 0px solid; width: 0px; height: 2.218em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>arccos</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mo>−</mo><mfrac><mn>1</mn><msqrt><mn>1</mn><mo>−</mo><msup><mi>x</mi><mn>2</mn></msup></msqrt></mfrac><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-468">d(\arccos x) = -\frac{1}{\sqrt{1-x^2}}dx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-469-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>arctan</mi><mo>&amp;#x2061;</mo><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mfrac><mn>1</mn><mrow><mn>1</mn><mo>+</mo><msup><mi>x</mi><mn>2</mn></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5372" style="width: 9.889em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.889em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1007.89em, 2.974em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5373"><span class="mo" id="MathJax-Span-5374" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5375" style="font-family: MathJax_Main;">arctan</span><span class="mo" id="MathJax-Span-5376"></span><span class="mi" id="MathJax-Span-5377" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="msup" id="MathJax-Span-5378"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5379" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5380" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5381" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5382" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5383" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1001.6em, 4.231em, -999.997em); top: -3.54em; left: 50%; margin-left: -0.797em;"><span class="mrow" id="MathJax-Span-5384"><span class="mn" id="MathJax-Span-5385" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span class="mo" id="MathJax-Span-5386" style="font-size: 70.7%; font-family: MathJax_Main;">+</span><span class="msubsup" id="MathJax-Span-5387"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5388" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mn" id="MathJax-Span-5389" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.72em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.717em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.711em; border-left: 0px solid; width: 0px; height: 1.932em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>arctan</mi><mo>⁡</mo><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mfrac><mn>1</mn><mrow><mn>1</mn><mo>+</mo><msup><mi>x</mi><mn>2</mn></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-469">(\arctan x)' = \frac{1}{1+x^2}</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-470-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>arctan</mi><mo>&amp;#x2061;</mo><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mfrac><mn>1</mn><mrow><mn>1</mn><mo>+</mo><msup><mi>x</mi><mn>2</mn></msup></mrow></mfrac><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5390" style="width: 11.546em; display: inline-block;"><span style="display: inline-block; position: relative; width: 9.203em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1009.15em, 2.974em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5391"><span class="mi" id="MathJax-Span-5392" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5393" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5394" style="font-family: MathJax_Main;">arctan</span><span class="mo" id="MathJax-Span-5395"></span><span class="mi" id="MathJax-Span-5396" style="font-family: MathJax_Math-italic; padding-left: 0.174em;">x</span><span class="mo" id="MathJax-Span-5397" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5398" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5399" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5400" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1001.6em, 4.231em, -999.997em); top: -3.54em; left: 50%; margin-left: -0.797em;"><span class="mrow" id="MathJax-Span-5401"><span class="mn" id="MathJax-Span-5402" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span class="mo" id="MathJax-Span-5403" style="font-size: 70.7%; font-family: MathJax_Main;">+</span><span class="msubsup" id="MathJax-Span-5404"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5405" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mn" id="MathJax-Span-5406" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.72em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.717em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span><span class="mi" id="MathJax-Span-5407" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5408" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.711em; border-left: 0px solid; width: 0px; height: 1.932em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>arctan</mi><mo>⁡</mo><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mfrac><mn>1</mn><mrow><mn>1</mn><mo>+</mo><msup><mi>x</mi><mn>2</mn></msup></mrow></mfrac><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-470">d(\arctan x) = \frac{1}{1+x^2}dx</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-471-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mtext>arccot</mtext><mi>x</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mo>&amp;#x2212;</mo><mfrac><mn>1</mn><mrow><mn>1</mn><mo>+</mo><msup><mi>x</mi><mn>2</mn></msup></mrow></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5409" style="width: 10.517em; display: inline-block;"><span style="display: inline-block; position: relative; width: 8.403em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1008.4em, 2.974em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5410"><span class="mo" id="MathJax-Span-5411" style="font-family: MathJax_Main;">(</span><span class="mtext" id="MathJax-Span-5412" style="font-family: MathJax_Main;">arccot</span><span class="mi" id="MathJax-Span-5413" style="font-family: MathJax_Math-italic;">x</span><span class="msup" id="MathJax-Span-5414"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5415" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5416" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5417" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5418" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="mfrac" id="MathJax-Span-5419"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5420" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1001.6em, 4.231em, -999.997em); top: -3.54em; left: 50%; margin-left: -0.797em;"><span class="mrow" id="MathJax-Span-5421"><span class="mn" id="MathJax-Span-5422" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span class="mo" id="MathJax-Span-5423" style="font-size: 70.7%; font-family: MathJax_Main;">+</span><span class="msubsup" id="MathJax-Span-5424"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5425" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mn" id="MathJax-Span-5426" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.72em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.717em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.711em; border-left: 0px solid; width: 0px; height: 1.932em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mtext>arccot</mtext><mi>x</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mo>−</mo><mfrac><mn>1</mn><mrow><mn>1</mn><mo>+</mo><msup><mi>x</mi><mn>2</mn></msup></mrow></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-471">(\text{arccot} x)' = -\frac{1}{1+x^2}</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-472-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mtext>arccot</mtext><mi>x</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mo>&amp;#x2212;</mo><mfrac><mn>1</mn><mrow><mn>1</mn><mo>+</mo><msup><mi>x</mi><mn>2</mn></msup></mrow></mfrac><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5427" style="width: 12.174em; display: inline-block;"><span style="display: inline-block; position: relative; width: 9.717em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.203em, 1009.66em, 2.974em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5428"><span class="mi" id="MathJax-Span-5429" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5430" style="font-family: MathJax_Main;">(</span><span class="mtext" id="MathJax-Span-5431" style="font-family: MathJax_Main;">arccot</span><span class="mi" id="MathJax-Span-5432" style="font-family: MathJax_Math-italic;">x</span><span class="mo" id="MathJax-Span-5433" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5434" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mo" id="MathJax-Span-5435" style="font-family: MathJax_Main; padding-left: 0.289em;">−</span><span class="mfrac" id="MathJax-Span-5436"><span style="display: inline-block; position: relative; width: 1.717em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.374em, 1000.29em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.169em;"><span class="mn" id="MathJax-Span-5437" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1001.6em, 4.231em, -999.997em); top: -3.54em; left: 50%; margin-left: -0.797em;"><span class="mrow" id="MathJax-Span-5438"><span class="mn" id="MathJax-Span-5439" style="font-size: 70.7%; font-family: MathJax_Main;">1</span><span class="mo" id="MathJax-Span-5440" style="font-size: 70.7%; font-family: MathJax_Main;">+</span><span class="msubsup" id="MathJax-Span-5441"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5442" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mn" id="MathJax-Span-5443" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1001.72em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 1.717em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span><span class="mi" id="MathJax-Span-5444" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5445" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.711em; border-left: 0px solid; width: 0px; height: 1.932em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mtext>arccot</mtext><mi>x</mi><mo stretchy="false">)</mo><mo>=</mo><mo>−</mo><mfrac><mn>1</mn><mrow><mn>1</mn><mo>+</mo><msup><mi>x</mi><mn>2</mn></msup></mrow></mfrac><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-472">d(\text{arccot} x) = -\frac{1}{1+x^2}dx</script></span></td></tr></tbody></table>
    
    <table><thead><tr><th>导数法则</th><th>微分法则</th></tr></thead><tbody><tr><td><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-473-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>u</mi><mo>&amp;#x00B1;</mo><mi>v</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><msup><mi>u</mi><mo>&amp;#x2032;</mo></msup><mo>&amp;#x00B1;</mo><msup><mi>v</mi><mo>&amp;#x2032;</mo></msup></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5446" style="width: 4.231em; display: inline-block;"><span style="display: inline-block; position: relative; width: 3.374em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(3.489em, 1003.37em, 7.203em, -999.997em); top: -4.397em; left: 0em;"><span class="mrow" id="MathJax-Span-5447"><span style="display: inline-block; position: relative; width: 3.374em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1003.37em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5448" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5449" style="font-family: MathJax_Math-italic;">u</span><span class="mo" id="MathJax-Span-5450" style="font-family: MathJax_Main; padding-left: 0.231em;">±</span><span class="mi" id="MathJax-Span-5451" style="font-family: MathJax_Math-italic; padding-left: 0.231em;">v</span><span class="msup" id="MathJax-Span-5452"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5453" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5454" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.089em, 1001.89em, 4.174em, -999.997em); top: -2.569em; left: 0em;"><span class="mo" id="MathJax-Span-5455" style="font-family: MathJax_Main;">=</span><span class="msup" id="MathJax-Span-5456" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 0.86em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.57em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5457" style="font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.574em;"><span class="mo" id="MathJax-Span-5458" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.089em, 1001.77em, 4.174em, -999.997em); top: -1.369em; left: 0em;"><span class="mo" id="MathJax-Span-5459" style="font-family: MathJax_Main;">±</span><span class="msup" id="MathJax-Span-5460" style="padding-left: 0.231em;"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.46em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5461" style="font-family: MathJax_Math-italic;">v</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.46em;"><span class="mo" id="MathJax-Span-5462" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.403em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -3.354em; border-left: 0px solid; width: 0px; height: 4.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>u</mi><mo>±</mo><mi>v</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><msup><mi>u</mi><mo>′</mo></msup><mo>±</mo><msup><mi>v</mi><mo>′</mo></msup></math></span></span><script type="math/tex" id="MathJax-Element-473">(u\pm v)' = u' \pm v'</script></td><td><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-474-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>u</mi><mo>&amp;#x00B1;</mo><mi>v</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mi>d</mi><mi>u</mi><mo>&amp;#x00B1;</mo><mi>d</mi><mi>v</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5463" style="width: 2.746em; display: inline-block;"><span style="display: inline-block; position: relative; width: 2.174em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(4.86em, 1002.12em, 9.946em, -999.997em); top: -5.769em; left: 0em;"><span class="mrow" id="MathJax-Span-5464"><span style="display: inline-block; position: relative; width: 2.174em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1001.49em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5465" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5466" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5467" style="font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.089em, 1001.77em, 4.403em, -999.997em); top: -2.569em; left: 0em;"><span class="mo" id="MathJax-Span-5468" style="font-family: MathJax_Main;">±</span><span class="mi" id="MathJax-Span-5469" style="font-family: MathJax_Math-italic; padding-left: 0.231em;">v</span><span class="mo" id="MathJax-Span-5470" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.146em, 1002.12em, 4.174em, -999.997em); top: -1.197em; left: 0em;"><span class="mo" id="MathJax-Span-5471" style="font-family: MathJax_Main;">=</span><span class="mi" id="MathJax-Span-5472" style="font-family: MathJax_Math-italic; padding-left: 0.289em;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5473" style="font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.146em, 1002em, 4.174em, -999.997em); top: 0.003em; left: 0em;"><span class="mo" id="MathJax-Span-5474" style="font-family: MathJax_Main;">±</span><span class="mi" id="MathJax-Span-5475" style="font-family: MathJax_Math-italic; padding-left: 0.231em;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5476" style="font-family: MathJax_Math-italic;">v</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 5.774em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -5.068em; border-left: 0px solid; width: 0px; height: 6.075em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>u</mi><mo>±</mo><mi>v</mi><mo stretchy="false">)</mo><mo>=</mo><mi>d</mi><mi>u</mi><mo>±</mo><mi>d</mi><mi>v</mi></math></span></span><script type="math/tex" id="MathJax-Element-474">d(u\pm v) = du \pm dv</script></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-475-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mi>u</mi><mi>v</mi><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><msup><mi>u</mi><mo>&amp;#x2032;</mo></msup><mi>v</mi><mo>+</mo><mi>u</mi><msup><mi>v</mi><mo>&amp;#x2032;</mo></msup></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5477" style="width: 9.26em; display: inline-block;"><span style="display: inline-block; position: relative; width: 7.374em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1007.37em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5478"><span class="mo" id="MathJax-Span-5479" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5480" style="font-family: MathJax_Math-italic;">u</span><span class="mi" id="MathJax-Span-5481" style="font-family: MathJax_Math-italic;">v</span><span class="msup" id="MathJax-Span-5482"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5483" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5484" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5485" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="msup" id="MathJax-Span-5486" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 0.86em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.57em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5487" style="font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.574em;"><span class="mo" id="MathJax-Span-5488" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mi" id="MathJax-Span-5489" style="font-family: MathJax_Math-italic;">v</span><span class="mo" id="MathJax-Span-5490" style="font-family: MathJax_Main; padding-left: 0.231em;">+</span><span class="mi" id="MathJax-Span-5491" style="font-family: MathJax_Math-italic; padding-left: 0.231em;">u</span><span class="msup" id="MathJax-Span-5492"><span style="display: inline-block; position: relative; width: 0.746em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.46em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5493" style="font-family: MathJax_Math-italic;">v</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.46em;"><span class="mo" id="MathJax-Span-5494" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mi>u</mi><mi>v</mi><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><msup><mi>u</mi><mo>′</mo></msup><mi>v</mi><mo>+</mo><mi>u</mi><msup><mi>v</mi><mo>′</mo></msup></math></span></span><script type="math/tex" id="MathJax-Element-475">(uv)' = u'v + uv'</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-476-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mi>u</mi><mi>v</mi><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mi>v</mi><mi>d</mi><mi>u</mi><mo>+</mo><mi>u</mi><mi>d</mi><mi>v</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5495" style="width: 10.174em; display: inline-block;"><span style="display: inline-block; position: relative; width: 8.117em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.374em, 1008.12em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5496"><span class="mi" id="MathJax-Span-5497" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5498" style="font-family: MathJax_Main;">(</span><span class="mi" id="MathJax-Span-5499" style="font-family: MathJax_Math-italic;">u</span><span class="mi" id="MathJax-Span-5500" style="font-family: MathJax_Math-italic;">v</span><span class="mo" id="MathJax-Span-5501" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5502" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mi" id="MathJax-Span-5503" style="font-family: MathJax_Math-italic; padding-left: 0.289em;">v</span><span class="mi" id="MathJax-Span-5504" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5505" style="font-family: MathJax_Math-italic;">u</span><span class="mo" id="MathJax-Span-5506" style="font-family: MathJax_Main; padding-left: 0.231em;">+</span><span class="mi" id="MathJax-Span-5507" style="font-family: MathJax_Math-italic; padding-left: 0.231em;">u</span><span class="mi" id="MathJax-Span-5508" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5509" style="font-family: MathJax_Math-italic;">v</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.432em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mi>u</mi><mi>v</mi><mo stretchy="false">)</mo><mo>=</mo><mi>v</mi><mi>d</mi><mi>u</mi><mo>+</mo><mi>u</mi><mi>d</mi><mi>v</mi></math></span></span><script type="math/tex" id="MathJax-Element-476">d(uv) = vdu + udv</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-477-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mo stretchy=&quot;false&quot;>(</mo><mfrac><mi>u</mi><mi>v</mi></mfrac><msup><mo stretchy=&quot;false&quot;>)</mo><mo>&amp;#x2032;</mo></msup><mo>=</mo><mfrac><mrow><msup><mi>u</mi><mo>&amp;#x2032;</mo></msup><mi>v</mi><mo>&amp;#x2212;</mo><mi>u</mi><msup><mi>v</mi><mo>&amp;#x2032;</mo></msup></mrow><msup><mi>v</mi><mn>2</mn></msup></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5510" style="width: 7.546em; display: inline-block;"><span style="display: inline-block; position: relative; width: 6.003em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.089em, 1006em, 2.917em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5511"><span class="mo" id="MathJax-Span-5512" style="font-family: MathJax_Main;">(</span><span class="mfrac" id="MathJax-Span-5513"><span style="display: inline-block; position: relative; width: 0.517em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.546em, 1000.4em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.226em;"><span class="mi" id="MathJax-Span-5514" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.654em; left: 50%; margin-left: -0.169em;"><span class="mi" id="MathJax-Span-5515" style="font-size: 70.7%; font-family: MathJax_Math-italic;">v</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1000.52em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 0.517em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span><span class="msup" id="MathJax-Span-5516"><span style="display: inline-block; position: relative; width: 0.689em; height: 0px;"><span style="position: absolute; clip: rect(3.089em, 1000.29em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mo" id="MathJax-Span-5517" style="font-family: MathJax_Main;">)</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.34em; left: 0.403em;"><span class="mo" id="MathJax-Span-5518" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5519" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5520" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 2.574em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1002.4em, 4.231em, -999.997em); top: -4.454em; left: 50%; margin-left: -1.197em;"><span class="mrow" id="MathJax-Span-5521"><span class="msup" id="MathJax-Span-5522"><span style="display: inline-block; position: relative; width: 0.574em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.4em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5523" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.403em;"><span class="mo" id="MathJax-Span-5524" style="font-size: 50%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mi" id="MathJax-Span-5525" style="font-size: 70.7%; font-family: MathJax_Math-italic;">v</span><span class="mo" id="MathJax-Span-5526" style="font-size: 70.7%; font-family: MathJax_Main;">−</span><span class="mi" id="MathJax-Span-5527" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span class="msup" id="MathJax-Span-5528"><span style="display: inline-block; position: relative; width: 0.517em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5529" style="font-size: 70.7%; font-family: MathJax_Math-italic;">v</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.346em;"><span class="mo" id="MathJax-Span-5530" style="font-size: 50%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.63em, 4.174em, -999.997em); top: -3.54em; left: 50%; margin-left: -0.34em;"><span class="msubsup" id="MathJax-Span-5531"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5532" style="font-size: 70.7%; font-family: MathJax_Math-italic;">v</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.346em;"><span class="mn" id="MathJax-Span-5533" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.57em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.574em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.639em; border-left: 0px solid; width: 0px; height: 2.004em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mo stretchy="false">(</mo><mfrac><mi>u</mi><mi>v</mi></mfrac><msup><mo stretchy="false">)</mo><mo>′</mo></msup><mo>=</mo><mfrac><mrow><msup><mi>u</mi><mo>′</mo></msup><mi>v</mi><mo>−</mo><mi>u</mi><msup><mi>v</mi><mo>′</mo></msup></mrow><msup><mi>v</mi><mn>2</mn></msup></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-477">(\frac{u}{v})' = \frac{u'v-uv'}{v^2}</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-478-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mo stretchy=&quot;false&quot;>(</mo><mfrac><mi>u</mi><mi>v</mi></mfrac><mo stretchy=&quot;false&quot;>)</mo><mo>=</mo><mfrac><mrow><mi>v</mi><mi>d</mi><mi>u</mi><mo>&amp;#x2212;</mo><mi>u</mi><mi>d</mi><mi>v</mi></mrow><msup><mi>v</mi><mn>2</mn></msup></mfrac></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5534" style="width: 8.231em; display: inline-block;"><span style="display: inline-block; position: relative; width: 6.574em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.146em, 1006.57em, 2.917em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5535"><span class="mi" id="MathJax-Span-5536" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5537" style="font-family: MathJax_Main;">(</span><span class="mfrac" id="MathJax-Span-5538"><span style="display: inline-block; position: relative; width: 0.517em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.546em, 1000.4em, 4.174em, -999.997em); top: -4.397em; left: 50%; margin-left: -0.226em;"><span class="mi" id="MathJax-Span-5539" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.654em; left: 50%; margin-left: -0.169em;"><span class="mi" id="MathJax-Span-5540" style="font-size: 70.7%; font-family: MathJax_Math-italic;">v</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1000.52em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 0.517em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span><span class="mo" id="MathJax-Span-5541" style="font-family: MathJax_Main;">)</span><span class="mo" id="MathJax-Span-5542" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="mfrac" id="MathJax-Span-5543" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 2.917em; height: 0px; margin-right: 0.117em; margin-left: 0.117em;"><span style="position: absolute; clip: rect(3.317em, 1002.8em, 4.231em, -999.997em); top: -4.454em; left: 50%; margin-left: -1.369em;"><span class="mrow" id="MathJax-Span-5544"><span class="mi" id="MathJax-Span-5545" style="font-size: 70.7%; font-family: MathJax_Math-italic;">v</span><span class="mi" id="MathJax-Span-5546" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5547" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span class="mo" id="MathJax-Span-5548" style="font-size: 70.7%; font-family: MathJax_Main;">−</span><span class="mi" id="MathJax-Span-5549" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span class="mi" id="MathJax-Span-5550" style="font-size: 70.7%; font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5551" style="font-size: 70.7%; font-family: MathJax_Math-italic;">v</span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.317em, 1000.63em, 4.174em, -999.997em); top: -3.54em; left: 50%; margin-left: -0.34em;"><span class="msubsup" id="MathJax-Span-5552"><span style="display: inline-block; position: relative; width: 0.631em; height: 0px;"><span style="position: absolute; clip: rect(3.546em, 1000.35em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5553" style="font-size: 70.7%; font-family: MathJax_Math-italic;">v</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; top: -4.226em; left: 0.346em;"><span class="mn" id="MathJax-Span-5554" style="font-size: 50%; font-family: MathJax_Main;">2</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(0.86em, 1002.92em, 1.26em, -999.997em); top: -1.311em; left: 0em;"><span style="display: inline-block; overflow: hidden; vertical-align: 0em; border-top: 1.3px solid; width: 2.917em; height: 0px;"></span><span style="display: inline-block; width: 0px; height: 1.089em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.639em; border-left: 0px solid; width: 0px; height: 1.932em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mo stretchy="false">(</mo><mfrac><mi>u</mi><mi>v</mi></mfrac><mo stretchy="false">)</mo><mo>=</mo><mfrac><mrow><mi>v</mi><mi>d</mi><mi>u</mi><mo>−</mo><mi>u</mi><mi>d</mi><mi>v</mi></mrow><msup><mi>v</mi><mn>2</mn></msup></mfrac></math></span></span><script type="math/tex" id="MathJax-Element-478">d(\frac{u}{v}) = \frac{vdu-udv}{v^2}</script></span></td></tr><tr><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-479-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><msubsup><mi>y</mi><mi>x</mi><mo>&amp;#x2032;</mo></msubsup><mo>=</mo><msubsup><mi>y</mi><mi>u</mi><mo>&amp;#x2032;</mo></msubsup><msubsup><mi>u</mi><mi>x</mi><mo>&amp;#x2032;</mo></msubsup></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5555" style="width: 5.431em; display: inline-block;"><span style="display: inline-block; position: relative; width: 4.346em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1004.35em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5556"><span class="msubsup" id="MathJax-Span-5557"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5558" style="font-family: MathJax_Math-italic;">y<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.431em, 1000.29em, 4.117em, -999.997em); top: -4.283em; left: 0.574em;"><span class="mo" id="MathJax-Span-5559" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.46em, 4.174em, -999.997em); top: -3.826em; left: 0.517em;"><span class="mi" id="MathJax-Span-5560" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mo" id="MathJax-Span-5561" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="msubsup" id="MathJax-Span-5562" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5563" style="font-family: MathJax_Math-italic;">y<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.431em, 1000.29em, 4.117em, -999.997em); top: -4.283em; left: 0.574em;"><span class="mo" id="MathJax-Span-5564" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.46em, 4.174em, -999.997em); top: -3.826em; left: 0.517em;"><span class="mi" id="MathJax-Span-5565" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="msubsup" id="MathJax-Span-5566"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.57em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5567" style="font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.431em, 1000.29em, 4.117em, -999.997em); top: -4.283em; left: 0.574em;"><span class="mo" id="MathJax-Span-5568" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.46em, 4.174em, -999.997em); top: -3.826em; left: 0.574em;"><span class="mi" id="MathJax-Span-5569" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.289em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><msubsup><mi>y</mi><mi>x</mi><mo>′</mo></msubsup><mo>=</mo><msubsup><mi>y</mi><mi>u</mi><mo>′</mo></msubsup><msubsup><mi>u</mi><mi>x</mi><mo>′</mo></msubsup></math></span></span><script type="math/tex" id="MathJax-Element-479">y_x' = y_u'u_x'</script></span></td><td><span class="math inline"><span class="MathJax_Preview" style="color: inherit;"></span><span class="MathJax" id="MathJax-Element-480-Frame" tabindex="0" style="position: relative;" data-mathml="<math xmlns=&quot;http://www.w3.org/1998/Math/MathML&quot;><mi>d</mi><mi>y</mi><mo>=</mo><msubsup><mi>y</mi><mi>u</mi><mo>&amp;#x2032;</mo></msubsup><msubsup><mi>u</mi><mi>x</mi><mo>&amp;#x2032;</mo></msubsup><mi>d</mi><mi>x</mi></math>" role="presentation"><nobr aria-hidden="true"><span class="math" id="MathJax-Span-5570" style="width: 6.86em; display: inline-block;"><span style="display: inline-block; position: relative; width: 5.489em; height: 0px; font-size: 125%;"><span style="position: absolute; clip: rect(1.431em, 1005.43em, 2.689em, -999.997em); top: -2.283em; left: 0em;"><span class="mrow" id="MathJax-Span-5571"><span class="mi" id="MathJax-Span-5572" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5573" style="font-family: MathJax_Math-italic;">y<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mo" id="MathJax-Span-5574" style="font-family: MathJax_Main; padding-left: 0.289em;">=</span><span class="msubsup" id="MathJax-Span-5575" style="padding-left: 0.289em;"><span style="display: inline-block; position: relative; width: 0.974em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.52em, 4.403em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5576" style="font-family: MathJax_Math-italic;">y<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.431em, 1000.29em, 4.117em, -999.997em); top: -4.283em; left: 0.574em;"><span class="mo" id="MathJax-Span-5577" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.46em, 4.174em, -999.997em); top: -3.826em; left: 0.517em;"><span class="mi" id="MathJax-Span-5578" style="font-size: 70.7%; font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="msubsup" id="MathJax-Span-5579"><span style="display: inline-block; position: relative; width: 1.031em; height: 0px;"><span style="position: absolute; clip: rect(3.374em, 1000.57em, 4.174em, -999.997em); top: -3.997em; left: 0em;"><span class="mi" id="MathJax-Span-5580" style="font-family: MathJax_Math-italic;">u</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.431em, 1000.29em, 4.117em, -999.997em); top: -4.283em; left: 0.574em;"><span class="mo" id="MathJax-Span-5581" style="font-size: 70.7%; font-family: MathJax_Main;">′</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span><span style="position: absolute; clip: rect(3.546em, 1000.46em, 4.174em, -999.997em); top: -3.826em; left: 0.574em;"><span class="mi" id="MathJax-Span-5582" style="font-size: 70.7%; font-family: MathJax_Math-italic;">x</span><span style="display: inline-block; width: 0px; height: 4.003em;"></span></span></span></span><span class="mi" id="MathJax-Span-5583" style="font-family: MathJax_Math-italic;">d<span style="display: inline-block; overflow: hidden; height: 1px; width: 0.003em;"></span></span><span class="mi" id="MathJax-Span-5584" style="font-family: MathJax_Math-italic;">x</span></span><span style="display: inline-block; width: 0px; height: 2.289em;"></span></span></span><span style="display: inline-block; overflow: hidden; vertical-align: -0.354em; border-left: 0px solid; width: 0px; height: 1.289em;"></span></span></nobr><span class="MJX_Assistive_MathML" role="presentation"><math xmlns="http://www.w3.org/1998/Math/MathML"><mi>d</mi><mi>y</mi><mo>=</mo><msubsup><mi>y</mi><mi>u</mi><mo>′</mo></msubsup><msubsup><mi>u</mi><mi>x</mi><mo>′</mo></msubsup><mi>d</mi><mi>x</mi></math></span></span><script type="math/tex" id="MathJax-Element-480">dy = y_u'u_x'dx</script></span></td></tr></tbody></table>
    
*   **一阶微分形式不变性**：
    
    不论是自变量还是中间变量，函数的微分形式总是：
    
    $$dy = f'(u)du$$
    
    此性质就是**一阶微分形式不变性**。
    

### **5.1.2 积分（Integration）**

#### **5.1.2.1 不定积分（Indefinite integration）**

*   **原函数**：如果在区间 $I$上，可导函数 $F(x)$的导函数为 $f(x)$，即当 $x\in I$时，即
    
    $$F'(x)=f(x) \; \text{或} \; d(F(x))=f(x)dx$$
    
    则 $F(x)$是 $f(x)$在区间 $I$上的**原函数**。
    
*   **原函数存在定理**：连续函数一定有原函数。
    
*   **不定积分定义**：
    
    在区间 $I$上，函数 $f(x)$的带有任意常数项的原函数称为 $f(x)$(或 $f(x)dx$) 在区间 $I$上的**不定积分**，记做
    

$$\int f(x)dx$$

​ 其中符号$\int$称为**积分号**，$f(x)$为**被积函数**，$f(x)dx$为**被积表达式**，$x$为**积分变量**。

​ 求一个函数的不定积分实际上只需求出它的一个原函数，再加上一个任意常数：

$$\int f(x)dx = f(x) + C$$

*   **不定积分的性质**：
    
    *   $\big[\int f(x)dx\big]' = f(x)$，或 $d\big[\int f(x)dx\big] = f(x)dx$
        
    *   $\int F'(x)dx = F(x) + C$，或$\int dF(x) = F(x) + C$
        
    
    由上面两个性质可得出：微分运算 $d$与不定积分运算$\int$互为**逆运算**，当它们的符号连在一起时，可抵消，抵消后可能相差一个常数。
    
    速记口诀：**先积后微，形式不变；先微后积，差一常数**。
    
    *   $\int \big[f(x)\pm g(x)\big]dx = \int f(x)dx \pm \int g(x)dx$
        
    *   $\int k f(x)dx = k \int f(x) dx$（$k$是常数且 $k\ne 0$）
        
    
    与求积分变量无关的常数 $k$，可以提出积分号。
    
*   **常见积分表**：
    
    *   $\int 0 dx = C$
        
    *   $\int x^n dx = \frac{1}{n+1}x^{n+1} + C \; (n \ne -1)$
        
    *   $\int \frac{1}{x}dx = \ln |x| + C$
        
    *   $\int \frac{1}{1+x^2}dx = arctan x + C$
        
    *   $\int \frac{1}{\sqrt{1-x^2}}dx = \arcsin x + C$
        
    *   $\int \sin x dx = -\cos x + C$
        
    *   $\int \cos x dx = \sin x + C$
        
    *   $\int \sec^2 x dx = \ tan x + C$
        
    *   $\int \csc^2 xdx = -\cot x + C$
        
    *   $\int \tan x \sec x dx = \sec x + C$
        
    *   $\int \cot x \csc x dx = -\csc x + C$
        
    *   $\int e^x dx = e^x + C$
        
    *   $\int a^x dx = \frac{1}{\ln a} a^x + C$
        
    *   $\int \tan x dx = - \ln |\cos x| + C$
        
    *   $\int \cot x dx = \ln|\sin x| + C$
        
    *   $\int \sec x dx = \ln |\sec x + \tan x| + C$
        
    *   $\int \csc x dx = \ln |\csc x - \cot x| + C$
        
    *   $\int \frac{dx}{a^2 + x^2} = \frac{1}{a} \arctan|\frac{x}{a} |+ C$
        
    *   $\int \frac{dx}{\sqrt{a^2 - x^2}} = \arcsin\frac{x}{a} + C$
        
    *   $\int \frac{dx}{x^2 - a^2} = \frac{1}{2a}\ln\big|\frac{x-a}{x+a}\big| + C$
        
    *   $\int \frac{dx}{\sqrt{x^2 - a^2}} = \ln|x+\sqrt{x^2-a^2}| + C$
        
    *   $\int \frac{dx}{\sqrt{a^2 + x^2}} = \ln|x+\sqrt{x^2+a^2}| + C$
        
*   **积分的方法**：
    
    *   **直接积分法**：利用不定积分的 4 个性质求不定积分。
        
        例如，求$\int \cos x dx$。
        
        因为 $(\sin x)' = \cos x$，所以$\int \cos x dx = \sin x + C$。
        
    *   **换元积分法**：
        
        *   设 $f(u)$具有原函数 $F(u)$，$u=h(x)$可导，则有换元公式：
            
            $$\begin{eqnarray} \int f(h(x))d(h(x)) &=& \big[\int f(u)du\big]_{u=h(x)} \\ &=& (F(u)+C)_{u=h(x)} \\ &=& F(h(x)) + C \end{eqnarray}$$
            
        *   设 $f(x)$连续，$x=h(t)$的导数 $h'(t)$也连续，且 $h'(t)\ne 0$，假若
            
            $$\int f(h(t))h'(t)dt = G(t) + C，$$
            
            则有换元公式：
            
            $$\int f(x)dx = \big[\int f[h(t)] h'(t) dt \big] = (G(t)+C)_{t=h^{-1}(x)} = G(h^{-1}(x) + C)$$
            
            其中 $t=h^{-1}(x)$为 $x=h(t)$的反函数。
            
    *   **分部积分法**：
        
        分部积分法由两个函数乘积的导数公式推导而来，最终形式：
        
        $$\int u dv= uv - \int v du$$
        
        若是分部的一部分有困难时，可以尝试另外一部分可能相对容易，这就是分部积分法的作用。
        

#### **5.1.2.2 定积分（Definite integration）**

定积分是积分学的另一个重要概念，自然科学与生产实践中的许多问题，如平面图形的面积、曲线的弧长、水压力、变力所做的功等都可以归结为定积分问题。

计算机图形学的很多计算问题也归结于定积分问题，如辐射度量、采样、卷积、预计算等等。

*   **定积分的定义**：
    
    ![](1679148483181.png)
    
    设 $f(x)$在区间 $[a,b]$上有界，在 $[a,b]$中插入若干个分点
    
    $$a = x_0 < x_1 < \;...\; < x_n = b$$
    
    把区间 $[a,b]$分成 $n$个小区间
    
    $$[x_0,x_1], \;[x_1,x_2], \; \; ..., \; [x_{n-1},x_n]$$
    
    各个小区间的长度依次为
    
    $$\triangle x_1 = x_1 - x_0, \; \triangle x_2 = x_2 - x_1, \; ..., \; \triangle x_{n-1} = x_n - x_{n-1}$$
    
    在每个小区间 $[x_{i-1},x_i]$上任取一点$\xi_i(x_{i-1} \leqslant \xi_i \leqslant x_i)$，取函数值 $f(\xi_i)$与小区间长度$\triangle x_i$的乘积 $f(\xi_i)\triangle x_i(i=1,2,...,n)$，再求和
    
    $$S = \sum_{i=1}^nf(\xi_i)\triangle x_i$$
    
    记$\lambda = \max\{{\triangle x_1, \triangle x_2, \; ..., \; \triangle x_n}\}$，当$\lambda \to 0$时，和 $S$的极限 $I$（有限）存在且 $[a,b]$的划分和$\lambda_i$无关，则成函数 $f(x)$在 $[a,b]$上可积，且称这个极限 $I$为函数 $f(x)$在区间 $[a,b]$上的**定积分**（简称**积分**），记作
    
    $$\int_a^bf(x)dx，即\int_a^bf(x)dx = \lim_{\lambda \to 0}\sum_{i=1}^nf(\xi_i)\triangle x_i = I$$
    
    其中称 $f(x)$为**被积函数**，$f(x)dx$为**被积表达式**，$x$为**积分变量**，$a$为**积分下限**，$b$为**积分上限**，$[a,b]$为**积分区间**。
    
*   **定积分性质**：
    
    *   设 $f(x)$在区间 $[a,b]$上连续，则 $f(x)$在 $[a,b]$上可积。
        
    *   设 $f(x)$在区间 $[a,b]$上有界，且只有有限个间断点，则 $f(x)$在 $[a,b]$上可积。
        
    *   当 $a=b$时，$\int_a^bf(x)dx，即\int_a^bf(x)dx=0$
        
    *   当 $a>b$时，$\int_a^bf(x)dx，即\int_a^bf(x)dx=-\int_b^af(x)dx$
        
    *   $\int_a^b[f(x)\pm g(x)]dx = \int_a^bf(x)dx\pm\int_a^bg(x)dx$
        
    *    $\int_a^b kf(x)dx = k\int_a^bf(x)dx$
        
    *   设 $a<c<b$，则
        
        $$\int_a^bf(x)dx = \int_a^cf(x)dx+\int_c^bf(x)dx$$
        
    *   设 $f(x) \equiv 1$，$x\in[a,b]$，则
        
        $$\int_a^bf(x)dx = b - a$$
        
    *   设 $f(x) \geqslant 0$，$x\in[a,b]$且 $a<b$，则
        
        $$\int_a^bf(x)dx \geqslant 0$$
        
    *   设 $f(x) \leqslant g(x)$，$x\in[a,b]$且 $a<b$，则
        
        $$\int_a^bf(x)dx \leqslant \int_a^bg(x)dx$$
        
    *   定积分估值定理：如果 $m\leqslant f(x) \leqslant M，x\in [a,b]$，即 $m$、$M$分别是 $f(x)$在区间 $[a,b]$的最小、最大值，那么
        
        $$m(b-a) \leqslant \int_a^b f(x)dx \leqslant M(b-a)$$
        
    *   定积分中值定理：如果函数 $f(x)$在 $[a,b]$上连续，则至少存在一点$\xi\in[a,b]$，使得
        
        $$\int_a^bf(x)dx = f(\xi)(b-a) \; (a\leqslant\xi\leqslant b)$$
        
    *   如果函数 $f(x)$在区间 $[a,b]$上连续，则积分上限的函数
        
        $$\Phi(x) = \int_a^bf(t)dt$$
        
        在 $[a,b]$上可导，并且它的导数
        
        $$\Phi'(x) = \frac{d}{dx}\int_a^xf(t)dt = f(x)\;(a\leqslant x \leqslant b)$$
        
    *   如果函数 $f(x)$在区间 $[a,b]$上连续，则函数
        
        $$\Phi(x) = \int_a^xf(t)dt$$
        
        就是 $f(x)$在区间 $[a,b]$上的一个原函数。
        
    *   **牛顿 - 莱布尼兹公式（Newton-Leibniz Formula）**：如果函数 $F(x)$是连续的，是 $f(x)$在区间 $[a,b]$上的一个原函数，则
        
        $$\int_a^bf(x)dx = F(x)\bigg|_a^b = F(b) - F(a)$$
        
        也叫**微积分基本公式**。它揭示了被积函数与原函数之间的联系，说明一个连续函数在区间 $[a,b]$上的定积分等于它的任意一个原函数在区间 $[a,b]$上的增量，它为定积分的计算提供了一个简单而有效的方法。
        
*   **定积分的方法**：
    
    有**换元法**、**分部积分法**，跟不定积分类似，唯一的区别是加了区间限制，不再累述。
    

微积分就介绍到这里了，其它更多高级的概念和性质，如多元微积分、多重微积分、无穷级数、幂级数等等，可以另外找资料，也可以在参考文献里寻得。

## **5.2 辐射度量（Radiometry，Radiation Measure）**

章节 [3.1.3 反射方程](#3.1.3 反射方程（Reflectance Equation）) 和[4.2.6 光的能量](#4.2.6 光的能量)已经列出了辐射度量的基本概念、符合、公式，本小节将做一些补充。

### **5.2.1 立体角（Solid Angle）**

在**二维**平面几何中，**弧度（Radian，rad）**是测量角度的标准单位，表示了与圆心角与其对应的圆弧长度的关系，见下图：

![](1679148483293.png)

_上图动态地描述了圆半径如何转化成圆弧，以及圆心角与弧度的对应关系。_

可以明显看出，弧度只是衡量角度大小，跟半径无关，所以弧度的计算方式是圆弧的长度除以圆半径：

$$弧度 = 圆心角 = \frac{圆心角对应的弧长度}{圆半径}$$

如果对角度和弧长进行微分，就可用微分的方式表达（$ds$表示微分的弧长）：

$$d\theta = \frac{ds}{r}$$

同样地，在**三维**立体几何中，也有跟弧度类似的概念，用来衡量三维球体的圆心角，它就是**立体角**（宏观符号$\Omega$，微分符号$\omega$）。

立体角的定义，用公式表达就是：

$$立体角 = \frac{立体角对应的球表面面积}{半径的影响因子}$$

![](1679148483323.png)

从上面可以看出，立体角与球体半径无关，由于是三维立体空间，且 1 单位立体角的球表面面积为 $r^2$（上图），所以半径的影响因子就是 $r^2$，用宏观符号公式表达：

$$\Omega = \frac {A}{r^{2} } sr$$

其中 $sr$（Steradian）是立体角的单位，叫**立体弧度**或**球面度**。

若对立体角和球表面面积微分，可得到微分形式的公式：

$$d\omega = \frac {dA}{r^{2}}$$

利用 [Spherical Cap 的面积公式](https://en.wikipedia.org/wiki/Spherical_cap)，可求得半个球体的立体角：

$$\Omega_{hemisphere} = \frac {2\pi r\cdot r}{r^{2} } sr = 2\pi \ sr$$

也就是说半个球体的立体角为 $2\pi \ sr$，整个球体的立体角为 $4\pi \ sr$：

$$\Omega_{sphere} = 4\pi \ sr$$

### **5.2.2 辐射强度（Radiant Intensity）**

![](1679148483346.png)

辐射强度指通过单位立体角的辐射通量。用符号 $I$表示，单位 W$/sr$，微分公式：

$$I = \frac{d\Phi}{d\omega}$$

**既然已有了辐照度和辐射度，为什么还要引入辐射强度呢？**

原因是在计算辐射时，有时会考虑某个点的通量的密度，但一个点的面积是 0，无法用辐照度和辐射度的公式，故而引入跟面积无关的辐射强度。而辐射强度之所以跟面积无关，是因为立体角只跟角度相关，跟球体的半径、距离、面积无关。

也就是说，由于立体角不会随距离变化而变化，辐射强度不会随距离变化而变化，不像点光源的辐照度会随距离增大而衰减。

### **5.2.3 辐射率（Radiance）**

辐射率是测量微小方向照到微小表面的通量，即每单位面积每单位立体角的辐射通量密度。用公式表达：

$$L = \frac{d\Phi}{d\omega dA^{\bot}}$$

![](1679148483382.png)

辐射率实际上就是材质的颜色，在基于物理着色时，计算表面一点的颜色就是计算它的辐射率。

辐射率不会随距离变化而衰减，这和真实世界的物理原理一致：在没有空气干扰的情况下，我们看到的物体颜色并不会随距离变化而变化。

## **5.3 公式推导**

### **5.3.1 麦克斯韦方程组（Maxwell's equations）**

既然光的很多现象，包括反射、折射定律都可以用麦克斯韦方程组解释，那我们就有必要揭开它的神秘面纱。

**麦克斯韦方程组**是描述了电场、磁场与电荷密度、电流密度之间关系的偏微分方程。利用麦克斯韦方程组，可以推论出电磁波在真空中以光速传播，并进而做出光是电磁波的猜想。

实际上，麦克斯韦方程组虽然由英国物理数学家詹姆斯 · 克拉克 · 麦克斯韦（James Clerk Maxwell）提出，但最初提出来时有 20 个方程和 20 个变量。后来由英国物理学家奥利弗 · 赫维赛德（Oliver Heaviside）和美国物理数学家约西亚 · 威拉德 · 吉布斯（Josiah Willard Gibbs）以矢量分析的形式重新表达，也就是现在我们使用的形式。

麦克斯韦方程组和洛伦兹力方程是经典电磁学的基础方程。从这些基础方程的相关理论，发展出现代的电力科技与电子科技。

它由 4 个方程组成：

*   **高斯定律**（Gauss' law）：描述电场与空间中电荷分布的关系。
*   **高斯磁定律**（Gauss's law for magnetism）：表明磁场的散度等于零，也就是说进入任何区域的磁场线，必需从该区域离开。
*   **法拉第定律**（Faraday's law）：描述时变磁场怎样感应出电场。
*   **麦克斯韦 - 安培定律**（Ampère's law with Maxwell's addition）：阐明磁场的产生方式有两种：一种是靠传导电流；另一种是靠时变电场，或称位移电流。

#### **5.3.1.1 高斯定律（Gauss' law）**

高斯定律表明在静电场中，穿过任一封闭曲面的电场通量只与封闭曲面内的电荷的代数和有关，且等于封闭曲面的电荷的代数和除以真空中的电容率。用积分形式表达的公式：

$$\Phi_{\boldsymbol E} = \frac{Q}{\varepsilon_0}$$

其中，$\Phi_E$是穿过封闭曲面的电场通量，$Q$是封闭曲面内的总电荷，$\varepsilon_0$是真空中的电容率。

若是更严谨一些，引入封闭曲面 $S$和及封闭曲面包含的体积 $V$，则有积分形式的公式：

$$\Phi_{\boldsymbol E} = \oint_{\boldsymbol S} \boldsymbol E\cdot dA$$

其中 $\boldsymbol E$是电场，$dA$是是封闭曲面的一块极小的面积，$\oint_S$表示封闭曲面面积，有些文献会写成$\iiint_S$、$\iint_S$、$\int_S$，表达的都是曲面面积。

还可以用微分的形式表达：

$$\triangle \cdot \boldsymbol E = \frac{\rho}{\varepsilon_0}$$

其中$\triangle \cdot \boldsymbol E$是电场散度，$\rho$是电荷密度。

由于微分形式是从微观层面描述的，从宏观维度上，可表达成：

$$D = \varepsilon \boldsymbol E$$

其中 $\boldsymbol D$表示电场散度，$\varepsilon$表示材质电容率，$\boldsymbol E$表示电场。此公式只适应与均匀、各向同性、非分散的线性物质。

#### **5.3.1.2 高斯磁定律（Gauss's law for magnetism）**

高斯磁定律表明磁场的散度等于零，因此磁场是一个螺线矢量场，还可以推断磁单极子不存在（下图）。磁的基本实体是磁偶极子，而不是磁荷。

![](1679148483414.png)

用微分形式的公式：

$$\triangle \cdot \boldsymbol B = 0$$

$\triangle \cdot \boldsymbol B$表示磁场的散度。

同样地，可以用积分形式：

$$\newcommand{\oiint}{\bigcirc \hspace{-1.3em}\int \hspace{-0.8em}\int} \oiint_{\delta\Omega}\boldsymbol E \cdot d\boldsymbol S = \frac{1}{\varepsilon_0}\iiint_\Omega \rho dV$$

#### **5.3.1.3 法拉第定律（Faraday's law）**

法拉第定律描述时变磁场怎样感应出电场。在积分形式，它表明在闭环移动电荷所需的每单位电荷的工作量等于通过封闭表面的磁通量的减少速率。微分形式：

$$\Delta \times \boldsymbol E = -\frac{\delta \boldsymbol B }{\delta t }$$

积分形式：

$$\oint_{\delta \sum} \boldsymbol E \cdot d\boldsymbol l = -\frac{d}{dt}\iint_{\sum}\boldsymbol B \cdot d \boldsymbol S$$

#### **5.3.1.4 **麦克斯韦 - 安培定律**（Ampère's law with Maxwell's addition）**

麦克斯韦 - 安培定律表明，磁场的产生有两种：

*   靠传导电流，也就是原本的安培定律；
*   靠时变电场，也被称作位移电流，这点是麦克斯韦修正项所提出。

微分形式的公式：

$$\Delta \times \boldsymbol B = \mu_0\bigg(\boldsymbol J + \varepsilon_0\frac{\delta\boldsymbol E}{\delta t} \bigg)$$

积分形式复杂一些：

$$\oint_{\delta \sum} \boldsymbol B \cdot d\boldsymbol l = \mu_0\bigg(\iint_{\sum} \boldsymbol J \cdot d \boldsymbol S + \varepsilon_0\frac{d}{dt}\iint_{\sum}\boldsymbol E\cdot d \boldsymbol S \bigg)$$

由于标准的 $L_AT^EX$无法表示闭合曲面的符号，可能跟维基百科的有些出入，具体参看维基百科的麦克斯韦方程组。

### **5.3.2 几何光学基本定律的推导**

几何光学有三条基本定律：

*   第一定律：入射波、反射波、折射波的波矢，与界面的法线共同包含于**入射平面**（下图）。
*   第二定律：反射角等于入射角。这定律称为**反射定律**。
*   第三定律：$n_{1}\sin \theta _{1}=n_{2}\sin \theta _{2}$，也叫**斯涅尔定律**或**折射定律**。

它们可以由麦克斯韦方程组推导出来。

![](1679148483452.png)

光波是电磁辐射，必须满足麦克斯韦方程组与伴随的边界条件，其中一条边界条件为，在边界的临近区域，电场平行于边界的分量必须具有连续性。假设边界为 xy - 平面，则在边界，有：

$$E_{{||,i}}(x,y,0)+E_{{||,r}}(x,y,0)=E_{{||,t}}(x,y,0)$$

其中，$E_{{||,i}}$、 $E_{{||,r}}$、$E_{{||,t}}$分别为在入射波、反射波、折射波（透射波）的电场平行于边界的分量。

假设入射波是频率为 $\omega$ 的单色平面波，则为了在任意时间满足边界条件，反射波、折射波的频率必定为  $\omega$ 。设定 $E_{{||,i}}$、$E_{{||,r}}$、$E_{{||,t}}$的形式分别为

$E_{{||,i}}=E_{{||,i0}}\ e^{{i{\mathbf {k}}_{i}\cdot {\mathbf {r}}-\omega t}}$、

$E_{{||,r}}=E_{{||,r0}}\ e^{{i{\mathbf {k}}_{r}\cdot {\mathbf {r}}-\omega t}}$、

$E_{{||,t}}=E_{{||,t0}}\ e^{{i{\mathbf {k}}_{t}\cdot {\mathbf {r}}-\omega t}}$，

其中，$\mathbf{k}_i$、 ${\mathbf {k}}_{r}$、${\mathbf {k}}_{t}$分别是入射波、反射波、折射波的波矢，$E_{{||,i0}}$、$E_{{||,r0}}$、$E_{{||,t0}}$分别是入射波、反射波、折射波的波幅（可能是复值）。

为了在边界任意位置 $(x,y,0)$满足边界条件，相位变化必须一样，必须设定  
$k_{{ix}}x+k_{{iy}}y=k_{{rx}}x+k_{{ry}}y=k_{{tx}}x+k_{{ty}}y$。  
因此，

$k_{{ix}}=k_{{rx}}=k_{{tx}}$、  
$k_{{iy}}=k_{{ry}}=k_{{ty}}$。  
不失一般性，假设 $k_{{iy}}=k_{{ry}}=k_{{ty}}=0$，则立刻可以推断**第一定律成立**，入射波、反射波、折射波的波矢，与界面的法线共同包含于入射平面。

从波矢 x - 分量的相等式，可以得到

$k_{{i}}\sin \theta _{i}=k_{{r}}\sin \theta _{r}$。  
而在同一介质里，$k_{{i}}=k_{{r}}$。所以，**第二定律成立**，入射角$\theta _{i}$等于反射角$\theta _{r}$。

应用折射率 $n$的定义式：

$n\ {\stackrel {def}{=}}\ {\frac {c}{v}}={\frac {ck}{\omega }}$，  
可以推断**第三定律成立**：

$n_{i}\sin \theta _{i}=n_{t}\sin \theta _{t}$；  
其中，$n_{t}$、$\theta _{t}$分别是折射介质的折射率与折射角。

从入射波、反射波、折射波之间的相位关系，就可以推导出几何光学的三条基础定律。

此外，还可以用费马原理、惠更斯原理、平移对称性推导出来，更多参看维基百科的 Snell's Law。

### **5.3.3 Cook-Torrance BRDF 推导**

本节参考了[基于物理着色：BRDF](https://zhuanlan.zhihu.com/p/21376124) 的公式推导部分。

假设有一束光照射到微表面上，入射光方向$\omega_i$，视线方向$\omega_o$，对反射到$\omega_o$方向的反射光有贡献的微表面法线为半角向量$\omega_h$，则这束光的微分通量是：

$$d \Phi_h = L_i(\omega_i) d \omega_i dA^{\bot}(\omega_h) = L_i(\omega_i) d \omega_i cos \theta_h dA(\omega_h)$$

其中 $dA(\omega_h)$是法线为半角向量$\omega_h$的微分微表面面积，$dA^{\bot}(\omega_h)$为 $dA(\omega_h)$在入射光线方向的投影，$\theta_h$为入射光线$\omega_i$和微表面法线$\omega_h$的夹角。

Torrance-Sparrow 将微分微表面面积 $dA(\omega_h)$定义为 $dA(\omega_h) = D(\omega_h) d \omega_h dA$，Torrance-Sparrow 将前两项解释为单位面积微平面中朝向为$\omega_h$的微分面积。

要从一组微表面面积 dA 中得到朝向为$\omega_h$的微表面面积 $dA(\omega_h)$，只需要将 $D(\omega_h)$定义为 $dA$中朝向为$\omega_h$的比例，取值范围在 $[0, 1]$就可以了。这里引入 $d \omega_h$的实际用途稍后再讨论。

由上两式可得：

$$d \Phi_h = L_i(\omega_i) d \omega_i cos \theta_h D(\omega_h) d \omega_h dA$$

设定微表面反射光线遵循菲涅尔定理，则反射通量：

$$d \Phi_o = F_r(\omega_o) d \Phi_h$$

由上两式可得反射辐射率：

$$dL_o(\omega_o) = \frac{d \Phi_o}{d \omega_o cos \theta_o dA} = \frac{F_r(\omega_o) L_i(\omega_i) d \omega_i cos \theta_h D(\omega_h) d \omega_h dA}{d \omega_o cos \theta_o dA}$$

由 BRDF 的定义可得：

$$f_r(\omega_i, \omega_o) = \frac{d L_o(\omega_o)}{d E_i(\omega_i)} = \frac{d L_o(\omega_o)}{L_i(\omega_i) cos \theta_i d \omega_i} = \frac{F_r(\omega_o) cos \theta_h D(\omega_h) d \omega_h}{cos \theta_o cos \theta_i d \omega_o}$$

这里需要特别强调几个夹角：

*   $\theta_h$是入射光线$\omega_i$与朝向为$\omega_h$的微表面法线的夹角
*   $\theta_i$是入射光线$\omega_i$与宏观表面法线的夹角
*   $\theta_o$是反射光线$\omega_o$与宏观表面法线的夹角

回到反射方程：

$$L_o(v) = \int_{\Omega }^{} f(l, v) \otimes L_i(l) cos \theta_i d\omega_i$$

它是对 $d \omega_i$积分，而上式分母包含 $d \omega_o$，可以通过找到 $\frac{d \omega_h}{d \omega_o}$的关系，把 $d \omega_o$消掉。塞入 $d \omega_h$并不会影响方程的合理性，因为 $D(\omega_h)$是可以调整的，现在 $D(\omega_h)$是一个有单位的量，单位为 $1/sr$。

继续 $d\omega_h$和 $d\omega_o$关系的推导：

![](1679148483497.png)

如上图，入射光线照射到一个微表面上，与微表面的单位上半球相交于点 $I$，与微表面相交于点 $O$，反射光线与单位上半球相交于点 $R$，反射光束立体角 $d \omega_o$（图中是 $d \omega_r$）等于光束与单位上半球相交区域面积 $dA_r$，法线立体角 $d \omega_h$（图中是 $d \omega^\prime$）等于法线立体角与单位上半球相交区域面积 $dA^\prime$，因此求 $\frac{d \omega_h}{d \omega_o}$等价于求 $\frac{dA^\prime}{dA_r}$。

连线 $IR$与法线 $n^\prime$相交于点 $P$，则 $IR = 2IP$，由于 $dA_r$与 $dA^{\prime \prime \prime}$半径的比值等于 $\frac{IR}{IP}$，而面积为$\pi r^2$，与半径的平方成正比，所以 $dA_r = 4 dA^{\prime \prime \prime}$

连线 $OQ$长度为 1，$OP$长度为 $cos \theta_i ^ \prime$，所以

$$\frac{dA^{\prime \prime}}{dA^{\prime \prime \prime}} = \frac{1}{cos ^ 2 \theta_i ^ \prime}$$

而 $dA^{\prime \prime} = \frac{dA^{\prime}}{cos \theta_i^{\prime}}$

由以上几式可得 $\frac{dA^\prime}{dA_r} = \frac{1}{4 cos \theta_i ^ \prime}$

需要注意的是，上图中的$\theta_i ^ \prime$实际上是微表面的半角$\theta_h$，所以 $\frac{d \omega_h}{d \omega_o} = \frac{1}{4 cos \theta_h}$

因此

$$f_r(\omega_i, \omega_o) = \frac{F_r(\omega_o) D(\omega_h)}{4 cos \theta_o cos \theta_i}$$

前面讲到过并非所有朝向为$\omega_h$的微表面都能接受到光照（Shadowing），也并非所有反射光照都能到达观察者（Masking），考虑几何衰减因子 G 的影响，最终得出 Cook-Torrance 公式：

$$f_r(\omega_i, \omega_o) = \frac{F_r(\omega_o) D(\omega_h) G(\omega_i, \omega_o)}{4 cos \theta_o cos \theta_i}$$

## **5.4 预计算技术**

在第三章阐述 PBR 的 Cook-Torrance 原理和实现的时候，提及过很多预渲染技术，诸如：Cubemap、HDR 环境光等。本章节主要是讲解这些预计算或预卷积的技术，为将耗时的部分提前渲染，以便减轻实时光照时的渲染消耗。

### **5.4.1 立方体图卷积（Cubemap convolution）**

立方体图卷积是以离线的方式预先为场景的辐照度求解所有漫反射间接光照的积分。为了解决积分问题，必须对每个片元在半球$\Omega$内的所有可能方向对场景的辐射进行采样。

然而，代码实现上不可能在半球$\Omega$从每个可能的方向采样环境的照明，可能的方向数量在理论上是无限的。但可以通过采用有限数量的方向或样本来近似方向的数量，均匀间隔或从半球内随机取得，以获得相当精确的辐照度近似，从而有效地用离散的方法求解积分$\int$。

即便采用离散的近似方法，对于每个片元实时执行此操作仍然太昂贵，因为样本数量仍然需要非常大才能获得不错的结果，因此通常采用预计算解决实时的消耗问题。由于半球$\Omega$的朝向决定所需捕获辐照度的位置，可以预先计算每个可能的半球方向的辐照度，所有采样的半球环绕着所有传出的方向 $w_o$：

$$L_o(p,\omega_o) = k_d\frac{c}{\pi} \int\limits_{\Omega} L_i(p,\omega_i) n \cdot \omega_i d\omega_i$$

给定任意方向向量 $w_i$后，就可以预计算的辐照度图进行采样。为了确定小块表面的间接漫射（辐照）光的数量，可以从半球的整个辐照度中采样出围绕其表面法线的总辐照度。取得场景的辐照度的代码很简单：

```
vec3 irradiance = texture(irradianceMap, N);
```

为了生成辐照度图，需要将环境的光照卷积转换为立方体图。鉴于对于每个片段，表面的半球沿着法向量定向 $N$，对立方体图进行卷积等于计算沿着法线 $N$的半球$\Omega$内的每个方向的总平均辐射度 $w_i$。

![](1679148483521.png)

[3.3.1.2 从球体图到立方体图](#3.3.1.2 从球体图到立方体图) 描述了如何从球体图转换成立方体贴图，这样就可以直接获取转换后的立方体贴图，以便在片段着色器中对其进行卷积，并使用朝向所有 6 个面部方向呈现的帧缓冲区将其计算结果放到新的立方体贴图中。由于已经描述了将球体图转换为立方体图，可以采用类似的方法和代码：

```
#version 330 core
out vec4 FragColor;
in vec3 localPos;

uniform samplerCube environmentMap;

const float PI = 3.14159265359;

void main()
{		
    // the sample direction equals the hemisphere's orientation 
    vec3 normal = normalize(localPos);
  
    vec3 irradiance = vec3(0.0);
  
    [...] // convolution code
  
    FragColor = vec4(irradiance, 1.0);
}
```

用`environmentMap`从球体 HDR 环境图转换到 HDR 立方体图。

卷积环境贴图有很多种方法，此处将为半球上的每个立方体贴图像素生成固定数量的样本方向向量围绕半球$\Omega$并平均结果。固定量的样本向量将均匀地分布在半球内部。注意，积分是连续函数，并且在给定固定量的样本向量的情况下离散地采样积分函数只是近似值。如果使用的样本向量越多，就越接近积分实际值，但同时预计算过程越慢。

围绕着立体角 $dw$的反射方程的积分$\int$很难处理，所以用其等效的球面坐标$\theta$和$\phi$。

![](1679148483542.png)

我们使用极面方位角$\phi$在半球环之间采样，其角度范围是 $0$和 $2\pi$，并使用仰角$\theta$，其角度范围是 $0$和 $\frac{1}{2}\pi$，这样可方便地对半球进行采样。采用球面角度后的反射公式：

$$L_o(p,\phi_o, \theta_o) = k_d\frac{c}{\pi} \int_{\phi = 0}^{2\pi} \int_{\theta = 0}^{\frac{1}{2}\pi} L_i(p,\phi_i, \theta_i) \cos(\theta) \sin(\theta) d\phi d\theta$$

用黎曼和的方法以及给定的 $n_1$、$n_2$球面坐标采样数量，可将积分转换为以下离散版本：

$$L_o(p,\phi_o, \theta_o) = k_d\frac{c}{\pi} \frac{1}{n_1 n_2} \sum_{\phi = 0}^{n_1} \sum_{\theta = 0}^{n_2} L_i(p,\phi_i, \theta_i) \cos(\theta) \sin(\theta) d\phi d\theta$$

当离散地对两个球面值进行采样时，仰角越高$\theta$，面积越小，如上图所示。如果不对面积差进行处理，就会出现累积误差。为了弥补较小的区域，可以增加额外的 $\sin$值来缩放 $\sin \theta$的权重。

给定每个片段调用的积分球面坐标对半球进行离散采样转换为以下代码：

```
vec3 irradiance = vec3(0.0);  

vec3 up    = vec3(0.0, 1.0, 0.0);
vec3 right = cross(up, normal);
up         = cross(normal, right);

float sampleDelta = 0.025;
float nrSamples = 0.0; 
for(float phi = 0.0; phi < 2.0 * PI; phi += sampleDelta)
{
    for(float theta = 0.0; theta < 0.5 * PI; theta += sampleDelta)
    {
        // spherical to cartesian (in tangent space)
        vec3 tangentSample = vec3(sin(theta) * cos(phi),  sin(theta) * sin(phi), cos(theta));
        // tangent space to world
        vec3 sampleVec = tangentSample.x * right + tangentSample.y * up + tangentSample.z * normal;

        irradiance += texture(environmentMap, sampleVec).rgb * cos(theta) * sin(theta);
        nrSamples++;
    }
}
irradiance = PI * irradiance * (1.0 / float(nrSamples));
```

通过指定一个固定的`sampleDelta`值来遍历半球，减小或增加样本增量将分别增加或减少准确度。

在两个`for`循环内，采用球面坐标将它们转换为 3D 笛卡尔样本向量，将样本从切线空间转换为世界空间，并使用此样本向量直接对 HDR 环境贴图进行采样。循环的最后将每个样本结果添加到`irradiance`，并除以采样的总数，得到平均采样辐照度。请注意，缩放采样的颜色值是`cos(theta)`，因为光线在较大的角度处较弱，并且`sin(theta)`是为了弥补较高仰角的半球区域中面积较小的样本区域。

### **5.4.2 预过滤 HDR 环境图（Pre-filtering HDR environment map）**

预过滤环境图与预卷积辐照图非常相似。不同之处在于，需要考虑粗糙度并在预过滤环境图的不同 mip 级别中按顺序地存储更粗糙的反射。

通过使用球面坐标生成均匀分布在半球$\Omega$上的样本向量来对环境贴图进行复杂处理的方法，虽然这个方法适用于辐照度，但对于镜面反射效果较差。当涉及镜面反射时，基于表面的粗糙度，光在通过法线 $n$附近的反射就越粗糙，范围越大：

![](1679148483587.png)

光线反射后所有可能的出射光形成的形状被称为**镜面波瓣**。随着粗糙度的增加，镜面波瓣的大小增加; 并且镜面波瓣的形状在变化的入射光方向上变化。因此，镜面波瓣高度取决于材质。

当谈到微表面模型时，可以将镜面波瓣想象为给定一些入射光方向的微平面中间向量的反射方向。当看到的大多数光线最终反射在微平面中间矢量周围的镜面波瓣中，这样的方法生成的样本向量才是有意义的，这个处理过程就是**重要性采样（Importance sampling）**。

#### **5.4.2.1 蒙特卡洛（Monte Carlo）积分和重要性采样（Importance sampling）**

为了充分掌握重要性采样的重要性，需要先深入研究已知的数学方法：蒙特卡洛积分。

**蒙特卡洛积分**主要围绕着统计和概率理论的组合。它帮我们离散地解决了一个群体统计或重要性的问题，而不必考虑**所有**群体。

例如，假设想要计算一个国家所有公民的平均身高。为了得到结果，可以测量**每个**公民并平均他们的身高，这将提供确切 ** 的答案。但是，由于大多数国家人口众多，这不是一个现实的方法：需要花费太多的精力和时间。

另一种方法是选择一个小得多的**完全随机**（无偏差）的人口子集，测量他们的身高并平均结果。这个人口可能只有 100 人。虽然不如确切的答案准确，但也会得到一个相对接近真相的答案，它被称为**大数定律（Law of large numbers）**。这个方法是，如果测量一个较小数量的子集 $N$，它从总人口中得到真正随机的样本，结果将与真实答案相对接近，并且随着样本数量 $N$的增加而变得更接近实际结果。

蒙特卡罗积分建立在这个大数定律的基础上，并采用相同的方法来求解积分。从总人口和平均值中随机抽取的方式简单地生成样本值 $N$，而不是为所有可能的（理论上无限的）样本值 $x$求解积分。如 $N$增加得到的结果更接近积分的确切答案：

$$O = \int\limits_{a}^{b} f(x) dx = \frac{1}{N} \sum_{i=0}^{N-1} \frac{f(x)}{pdf(x)}$$

为了解决积分，我们采取用 $N$从人口 $a$到 $b$中随机抽样，将它们加在一起并除以样本总数以平均它们。该 $pdf$ 代表着概率密度函数（Probability density function），它表明特定样本在整个样本集上发生的概率。例如，人口高度的 $pdf$看起来有点像这样：

![](1679148483630.png)

从该图中可以看出，如果我们采用任意随机样本的人口，那么挑选高度为 1.70 的人的样本的可能性更高，而样本高度为 1.50 的概率较低。

当涉及蒙特卡罗积分时，一些样本可能比其他样本具有更高的生成概率。这就是为什么对于任何一般的蒙特卡罗估计，我们根据 $pdf$将采样值除以采样概率。到目前为止，在估算积分的每个例子中，生成的样本是均匀的，具有完全相同的生成几率。到目前为止我们的估计是不偏不倚，这意味着，鉴于样本数量不断增加，我们最终将会收敛到积分的**精确**解。

但是，蒙特卡罗的一些样本是有偏倚的，意味着生成的样本不是完全随机的，而是聚焦于特定的值或方向。这些有偏倚的蒙特卡罗估计有一个更快的收敛速度，这意味着它们可以以更快的速度收敛到精确值。但是，由于此方法的偏向性质，它们可能永远不会收敛到精确值。这通常是可接受的平衡，特别是在计算机图形学中，因为只要结果在视觉上可接受，精确的解决方案就不太重要。正如我们很快就会看到重要性采样（使用偏置估计器）所生成的样本偏向于特定方向，在这种情况下，我们通过将每个样本乘以或除以其对应的 $pdf$来达到这一点。

蒙特卡罗积分在计算机图形学中非常普遍，因为它是以离散和有效的方式近似连续积分的一种相当直观的方式：取任何面积或体积进行采样（如半球$\Omega$），生成 $N$区域 / 体积内的随机样本量和总和，并权衡每个样本对最终结果的权重。

蒙特卡洛积分是一个广泛的数学主题，这里不会深入研究具体细节，但会提到有多种方法可以生成**随机样本**。默认情况下，每个样本都是完全随机（伪随机）的，因为我们习惯了，但是通过利用半随机序列的某些属性，我们可以生成仍然是随机的但具有有趣属性的样本向量。例如，我们可以用**低差异序列（Low-discrepancy sequences）**对蒙特卡洛进行积分，以生成随机样本，且每个样本分布更均匀：

![](1679148483730.png)

_图左：完全伪随机序列生成的采用点；图右：低差异序列生成的采样点。可以看出右边的更均匀。_

当使用低差异序列生成蒙特卡罗样本向量时，该过程称为**准蒙特卡罗积分（Quasi-Monte Carlo integration）**。准蒙特卡罗方法有更快的收敛速度，这使它们对性能繁重的应用程序感兴趣。

鉴于新获得的蒙特卡罗和准蒙特卡罗积分的知识，我们可以使用一个有趣的属性来实现更快的收敛速度，它就是**重要性采样（ Importance sampling）**。当涉及光的镜面反射时，反射光向量被约束在镜面波瓣中，其尺寸由表面的粗糙度决定。看到镜面外的任何（准）随机生成的样本与镜面积分无关，将样本生成集中在镜面波瓣内是有意义的，代价是蒙特卡罗估计有偏差。

重要性采样是这样的：在一些区域内生成样本向量，该区域受到围绕微平面中间向量的粗糙度的约束。通过将准蒙特卡罗采样与低差异序列相结合并使用重要性采样偏置采样向量，可以获得高收敛率。因为以更快的速度到达解决方案，所以只需要更少的样本来达到足够的近似值。因此，该组合甚至允许图形应用程序实时解决镜面反射积分，尽管它仍然比预先计算结果慢得多。

#### **5.4.2.2 低差异序列（Low-discrepancy sequence）**

这里，将通过基于准蒙特卡罗方法的随机低差异序列，使用重要性采样预先计算间接反射方程的镜面反射部分。本小节使用的序列称为**哈默斯利序列（Hammersley Sequence）**。哈默斯利序列序列基于**范德科皮特（Van Der Corpus）序列**，它将以基数 $b$表示的自然数列反转可得结果。

鉴于一些巧妙的技巧，我们可以非常有效地产生，我们将用它来获得一个序列哈默斯利样品着色器程序范德语料库序列，`N`是总样本：

```
// 反转的范德科皮特序列
float RadicalInverse_VdC(uint bits) 
{
    bits = (bits << 16u) | (bits >> 16u);
    bits = ((bits & 0x55555555u) << 1u) | ((bits & 0xAAAAAAAAu) >> 1u);
    bits = ((bits & 0x33333333u) << 2u) | ((bits & 0xCCCCCCCCu) >> 2u);
    bits = ((bits & 0x0F0F0F0Fu) << 4u) | ((bits & 0xF0F0F0F0u) >> 4u);
    bits = ((bits & 0x00FF00FFu) << 8u) | ((bits & 0xFF00FF00u) >> 8u);
    return float(bits) * 2.3283064365386963e-10; // / 0x100000000
}
// ----------------------------------------------------------------------------
// 哈默斯利序列
vec2 Hammersley(uint i, uint N)
{
    return vec2(float(i)/float(N), RadicalInverse_VdC(i));
}
```

GLSL 代码`Hammersley`函数给出了总样本集为 $N$的低差异样本 $i$。

并非所有与 OpenGL 的驱动程序都支持位运算符（例如 WebGL 和 OpenGL ES 2.0），在这种情况下，我们可能希望使用不依赖于位运算符的替代版 Van Der Corpus Sequence：

```
float VanDerCorpus(uint n, uint base)
{
    float invBase = 1.0 / float(base);
    float denom   = 1.0;
    float result  = 0.0;

    for(uint i = 0u; i < 32u; ++i)
    {
        if(n > 0u)
        {
            denom   = mod(float(n), 2.0);
            result += denom * invBase;
            invBase = invBase / 2.0;
            n       = uint(float(n) / 2.0);
        }
    }

    return result;
}
// ----------------------------------------------------------------------------
vec2 HammersleyNoBitOps(uint i, uint N)
{
    return vec2(float(i)/float(N), VanDerCorpus(i, 2u));
}
```

请注意，由于旧硬件中的 GLSL 循环限制，序列会循环遍历`32`位能表示的所有数。这个版本性能较差，但可以在所有硬件上运行。

值得一提的是，生成低差异序列的方法还有很多：

*   [Random numbers](https://en.wikipedia.org/wiki/Low-discrepancy_sequence#Random_numbers)
*   [Additive recurrence](https://en.wikipedia.org/wiki/Low-discrepancy_sequence#Additive_recurrence)
*   [van der Corput sequence](https://en.wikipedia.org/wiki/Low-discrepancy_sequence#van_der_Corput_sequence)
*   [Halton sequence](https://en.wikipedia.org/wiki/Low-discrepancy_sequence#Halton_sequence)
*   [Hammersley set](https://en.wikipedia.org/wiki/Low-discrepancy_sequence#Hammersley_set)
*   [Sobol sequence](https://en.wikipedia.org/wiki/Low-discrepancy_sequence#Sobol_sequence)
*   [Poisson disk sampling](https://en.wikipedia.org/wiki/Low-discrepancy_sequence#Poisson_disk_sampling)

详细请参看 [Low-discrepancy sequence](https://en.wikipedia.org/wiki/Low-discrepancy_sequence)。

#### **5.4.2.3 GGX 重要性采样（GGX Importance sampling）**

我们将基于表面粗糙度生成偏向于微表面中间矢量的一般反射方向的样本矢量来取代统一或随机（蒙特卡罗）地在积分半球$\Omega$上生成样本向量。采样过程将类似于之前的过程：开始一个大循环，生成一个随机（低差异）序列值，取序列值在切线空间中生成一个样本向量，转换到世界空间并采样场景的辐射。不同的是，我们现在使用低差异序列值作为输入来生成样本向量：

```
const uint SAMPLE_COUNT = 4096u;
for(uint i = 0u; i < SAMPLE_COUNT; ++i)
{
	// 使用Hammersley序列
    vec2 Xi = Hammersley(i, SAMPLE_COUNT);
```

另外，为了构建样本向量，我们需要一些方法来定向和偏置原本朝向某些表面粗糙度的镜面波瓣的样本向量。我们可以按照章节 [3.1.4 双向反射分布函数（BRDF）](#3.1.4 双向反射分布函数（BRDF）) 中的描述获取 NDF ，并将 GGX NDF 结合在 Epic Games 所描述的那样球形采样向量：

```
vec3 ImportanceSampleGGX(vec2 Xi, vec3 N, float roughness) {
    float a = roughness*roughness;
	
    float phi = 2.0 * PI * Xi.x;
    float cosTheta = sqrt((1.0 - Xi.y) / (1.0 + (a*a - 1.0) * Xi.y));
    float sinTheta = sqrt(1.0 - cosTheta*cosTheta);
	
    // from spherical coordinates to cartesian coordinates
    vec3 H;
    H.x = cos(phi) * sinTheta;
    H.y = sin(phi) * sinTheta;
    H.z = cosTheta;
	
    // from tangent-space vector to world-space sample vector
    vec3 up        = abs(N.z) < 0.999 ? vec3(0.0, 0.0, 1.0) : vec3(1.0, 0.0, 0.0);
    vec3 tangent   = normalize(cross(up, N));
    vec3 bitangent = cross(N, tangent);
	
    vec3 sampleVec = tangent * H.x + bitangent * H.y + N * H.z;
    return normalize(sampleVec);
}
```

这给了我们一个样本向量，它基于一些输入粗糙度和低差异序列值 $X_i$，并且在预期的微表面中间向量的周围。请注意，根据迪斯尼原则的 PBR 研究，Epic Games 使用平方粗糙度来获得更好的视觉效果。

用低差异序列的 Hammersley 序列和样本生成为我们提供了最终确定预过滤卷积着色器：

```
#version 330 core
out vec4 FragColor;
in vec3 localPos;

uniform samplerCube environmentMap;
uniform float roughness;

const float PI = 3.14159265359;

float RadicalInverse_VdC(uint bits);
vec2 Hammersley(uint i, uint N);
vec3 ImportanceSampleGGX(vec2 Xi, vec3 N, float roughness);
  
void main()
{		
    vec3 N = normalize(localPos); 
    vec3 R = N;
    vec3 V = R;

    const uint SAMPLE_COUNT = 1024u;
    float totalWeight = 0.0; 
    vec3 prefilteredColor = vec3(0.0); 
    for(uint i = 0u; i < SAMPLE_COUNT; ++i)
    {
        vec2 Xi = Hammersley(i, SAMPLE_COUNT);
        vec3 H  = ImportanceSampleGGX(Xi, N, roughness);
        vec3 L  = normalize(2.0 * dot(V, H) * H - V);

        float NdotL = max(dot(N, L), 0.0);
        if(NdotL > 0.0)
        {
            prefilteredColor += texture(environmentMap, L).rgb * NdotL;
            totalWeight      += NdotL;
        }
    }
    prefilteredColor = prefilteredColor / totalWeight;

    FragColor = vec4(prefilteredColor, 1.0);
}
```

根据输入的粗糙度预先过滤环境，这些粗糙度在预过滤器立方体贴图的每个 mipmap 级别（从`0.0`到`1.0`）中变化，并将结果存储在`prefilteredColor`中。得到的预过滤颜色除以总样品权重，其中对最终结果影响较小的样品（对于小 NdotL）对最终重量的权重较小。

#### **5.4.2.3 预过滤卷积瑕疵**

虽然上述的预过滤图在大多数情况下都能正常，但总会遇到一些瑕疵。下面列出最常见的，包括如何解决它们。

*   **Cubemap 高粗糙度的接缝**

在具有粗糙表面的表面上对预滤镜图进行采样意味着在其一些较低的 mip 级别上对预滤镜图进行采样。对立方体贴图进行采样时，默认情况下，OpenGL 不会在立方体贴图面上进行线性插值。由于较低的 mip 级别都具有较低的分辨率，并且预滤波器映射与较大的样本波瓣进行了卷积，因此立方体面之间的滤波的瑕疵变得非常明显：

![](1679148483753.png)

幸运的是，OpenGL 为我们提供了通过启用 GL_TEXTURE_CUBE_MAP_SEAMLESS 来正确过滤立方体贴图面的选项：

```
glEnable(GL_TEXTURE_CUBE_MAP_SEAMLESS);
```

只需在应用程序启动时的某个位置启用此属性，接缝就会消失。

*   **预过滤卷积中的亮点**

由于镜面反射中的高频细节和剧烈变化的光强度，使镜面反射卷积需要大量样本以适当地解析 HDR 环境反射的广泛变化的性质。我们已经采集了大量样本，但在某些环境中，在某些较粗糙的 mip 级别上可能仍然不够，在这种情况下，将开始看到明亮区域周围出现点状图案：

![](1679148484012.png)

一种选择是进一步增加样本数，但这对所有环境都还不足够。可以通过（在预过滤卷积期间）不直接对环境贴图进行采样来减少这种伪影，而是基于积分的 PDF 和粗糙度对环境贴图的 mip 级别进行采样：

```
float D   = DistributionGGX(NdotH, roughness);
float pdf = (D * NdotH / (4.0 * HdotV)) + 0.0001; 

float resolution = 512.0; // resolution of source cubemap (per face)
float saTexel  = 4.0 * PI / (6.0 * resolution * resolution);
float saSample = 1.0 / (float(SAMPLE_COUNT) * pdf + 0.0001);

float mipLevel = roughness == 0.0 ? 0.0 : 0.5 * log2(saSample / saTexel);
```

不要忘记在环境贴图上启用三线性过滤，以便从以下位置对其 mip 级别进行采样：

```
glBindTexture(GL_TEXTURE_CUBE_MAP, envCubemap);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
```

然后让 OpenGL 在设置立方体贴图的基本纹理后生成 mipmap ：

```
// convert HDR equirectangular environment map to cubemap equivalent
[...]
// then generate mipmaps
glBindTexture(GL_TEXTURE_CUBE_MAP, envCubemap);
glGenerateMipmap(GL_TEXTURE_CUBE_MAP);
```

这种效果非常好，并且可以在粗糙表面上的预过滤图中删除大多数点。

### **5.4.3 预计算 BRDF**

在预过滤环境启动和运行的情况下，我们可以关注分裂和近似的第二部分：BRDF。让我们再次简要回顾一下镜面分裂和近似：

$$L_o(p,\omega_o) = \int\limits_{\Omega} L_i(p,\omega_i) d\omega_i * \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d\omega_i$$

我们已经在不同粗糙度级别的预过滤图中预先计算了分裂和近似的左侧部分。右侧要求我们在角度上收集 BRDF 方程 $n \cdot \omega_o$、表面粗糙度和菲涅耳的 $F_0$。这类似于将镜面 BRDF 与纯白环境或`1.0`的恒定辐射 $L_i$进行积分。将 BRDF 压缩为 3 个变量有点多，但我们可以将 $F_0$移出镜面 BRDF 方程式：

$$\int\limits_{\Omega} f_r(p, \omega_i, \omega_o) n \cdot \omega_i d\omega_i = \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) \frac{F(\omega_o, h)}{F(\omega_o, h)} n \cdot \omega_i d\omega_i$$

$F$是菲涅耳方程。将菲涅耳分母移动到 BRDF 给出了以下等效方程：

$$\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} F(\omega_o, h) n \cdot \omega_i d\omega_i$$

用 Fresnel-Schlick 近似法代替最右边的 $F$可得到：

$$\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 + (1 - F_0){(1 - \omega_o \cdot h)}^5) n \cdot \omega_i d\omega_i$$

再进一步地，用$\alpha$替换 ${(1 - \omega_o \cdot h)}^5$，将更容易解决 $F_0$：

$$\begin{eqnarray*} &&\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 + (1 - F_0)\alpha) n \cdot \omega_i d\omega_i \\ &=& \int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 + 1*\alpha - F_0*\alpha) n \cdot \omega_i d\omega_i \\ &=& \int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 * (1 - \alpha) + \alpha) n \cdot \omega_i d\omega_i \end{eqnarray*}$$

然后拆分菲涅耳函数 $F$成两个积分：

$$\int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (F_0 * (1 - \alpha)) n \cdot \omega_i d\omega_i + \int\limits_{\Omega} \frac{f_r(p, \omega_i, \omega_o)}{F(\omega_o, h)} (\alpha) n \cdot \omega_i d\omega_i$$

由于 $F_0$是常量，可以从积分号内移出。接下来，我们替换$\alpha$回原来的形式，得到最终的 BRDF 方程：

$$F_0 \int\limits_{\Omega} f_r(p, \omega_i, \omega_o)(1 - {(1 - \omega_o \cdot h)}^5) n \cdot \omega_i d\omega_i + \int\limits_{\Omega} f_r(p, \omega_i, \omega_o) {(1 - \omega_o \cdot h)}^5 n \cdot \omega_i d\omega_i$$

两个得到的积分分别代表了 $F_0$的缩放和偏移。请注意，作为 $f(p, \omega_i, \omega_o)$已包含一个 $F$项，所以 $F$项都从 f$ 中删除了！

以类似于早期卷积环境图的方式，我们可以在其输入上卷积 BRDF 方程：$n$和$\omega_o$之间的角度和粗糙度，并将卷积的结果存储在 2D 查找纹理（LUT）中。

BRDF 卷积着色器在 2D 平面上运行，使用其 2D 纹理坐标直接作为 BRDF 卷积的输入（`NdotV`和`roughness`）。卷积代码很大程度上类似于预过滤卷积，不同之处在于它现在根据我们的 BRDF 几何函数和 Fresnel-Schlick 的近似值处理样本向量：

```
vec2 IntegrateBRDF(float NdotV, float roughness)
{
    vec3 V;
    V.x = sqrt(1.0 - NdotV*NdotV);
    V.y = 0.0;
    V.z = NdotV;

    float A = 0.0;
    float B = 0.0;

    vec3 N = vec3(0.0, 0.0, 1.0);

    const uint SAMPLE_COUNT = 1024u;
    for(uint i = 0u; i < SAMPLE_COUNT; ++i)
    {
        vec2 Xi = Hammersley(i, SAMPLE_COUNT);
        vec3 H  = ImportanceSampleGGX(Xi, N, roughness);
        vec3 L  = normalize(2.0 * dot(V, H) * H - V);

        float NdotL = max(L.z, 0.0);
        float NdotH = max(H.z, 0.0);
        float VdotH = max(dot(V, H), 0.0);

        if(NdotL > 0.0)
        {
            float G = GeometrySmith(N, V, L, roughness);
            float G_Vis = (G * VdotH) / (NdotH * NdotV);
            float Fc = pow(1.0 - VdotH, 5.0);

            A += (1.0 - Fc) * G_Vis;
            B += Fc * G_Vis;
        }
    }
    A /= float(SAMPLE_COUNT);
    B /= float(SAMPLE_COUNT);
    return vec2(A, B);
}
// ----------------------------------------------------------------------------
void main() 
{
    vec2 integratedBRDF = IntegrateBRDF(TexCoords.x, TexCoords.y);
    FragColor = integratedBRDF;
}
```

从上面可看到，BRDF 卷积是从数学到代码的直接转换。采取角度$\theta$和粗糙度作为输入，生成具有重要性采样的样本向量，在几何体上处理它并且导出 BRDF 的菲涅耳项，并输出对于每个样本的 $F_0$的缩放和偏移，最后将它们平均化。

当与 IBL 一起使用时，BRDF 的几何项略有不同，亦即变量 $k$的解释略有不同：

$$\begin{eqnarray*} k_{direct} &=& \frac{(\alpha + 1)^2}{8} \\ k_{IBL} &=& \frac{\alpha^2}{2} \end{eqnarray*}$$

由于 BRDF 卷积是我们将使用的镜面 IBL 积分的一部分，所以用 $k_{IBL}$作为 Schlick-GGX 几何函数的参数：

```
float GeometrySchlickGGX(float NdotV, float roughness) {
    float a = roughness;
    float k = (a * a) / 2.0; // k_IBL

    float nom   = NdotV;
    float denom = NdotV * (1.0 - k) + k;

    return nom / denom;
}
// ----------------------------------------------------------------------------
float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness) {
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx2 = GeometrySchlickGGX(NdotV, roughness);
    float ggx1 = GeometrySchlickGGX(NdotL, roughness);

    return ggx1 * ggx2;
}
```

分裂和积分卷积的 BRDF 部分渲染结果如下：

![](1679148484034.png)

利用预滤环境图和 BRDF 2D LUT，我们可以根据分裂和近似计算间接光照镜面部分的积分。然后，组合间接或环境镜面反射光，最终算出 IBL 光照结果。

## **5.5 PBR 的优化**

### **5.5.1 离线渲染优化**

[5.4 预计算技术](#5.4 预计算技术) 章节提到了一些离线渲染的加速技术，除此之外，常见的离线技术还有：

*   局部静态光照烘焙
*   全局光照烘焙

还可以从以下小节中阐述的方法加速离线渲染部分。

#### **5.5.1.1 积分公式优化**

主要是利用 [5.1 微积分（Calculus）](#5.1 微积分（Calculus）) 描述的性质和定理对渲染公式进行优化：

*   常量移出积分项外
*   增加等效积分项
*   分离积分项
*   利用近似法替代复杂项

具体例子可以参看 [5.4 预计算技术](#5.4 预计算技术)。

#### **5.5.1.2 硬件集成**

将渲染通用的逻辑集成硬件指令或内建接口，可以充分利用硬件的性能，从而为渲染加速。

例如，将光线追踪算法集成进 GPU 显卡，而 nVidia 新一代 RTX20 系显卡已经集成了光线追踪技术，使得渲染效率更上一层楼。Unreal Engine 4.22 的版本也集成了这一特性。

![](1679148484069.png)

#### **5.5.1.3 并行渲染**

通过多线程、多进程、多设备的架构分摊消耗的帧渲染，使得每帧的渲染时间大大降低。这种技术在实时渲染领域也逐渐被普及。

#### **5.5.1.4 分布式渲染**

不同于并行渲染的小规模架构，分布式渲染通常以图形工作站、集群式渲染簇等中大型硬件架构为依托，以满足电影级别的离线渲染加速需求。

下图是[《A MultiAgent System for Physically based Rendering Optimization》](http://www.weiss-gerhard.info/publications/D02.pdf)提出的一种多代理的加速渲染架构：

![](1679148484105.png)

### **5.5.2 实时渲染优化**

#### **5.5.2.1 光照模型优化**

*   GGX 伦勃朗光照计算
*   Schlick 的 $F_0$近似法
*   Smith 几何遮蔽函数混合
*   迪斯尼原则的金属度线性插值

以上都是本文前面章节描述过的加速算法，这对于性能敏感的实时渲染领域是非常有必要的。

#### **5.5.2.2 资源优化**

*   若干贴图合成一张蒙板图。将若干独立的 PBR 属性蒙板贴图合成一张：
    
    ![](1679148484125.png)
    
    _使用同一张蒙板贴图同时控制 PBR 的颜色、金属度、粗糙度等属性。_
    
*   减少 PBR 标准参数的使用。例如，金属材质的漫反射大部分是黑色，所以无需额外的漫反射贴图。
    
*   其它资源优化：材质、模型、渲染参数、纹理、PBR 参数等等几乎都有优化的余地。
    

#### **5.5.2.3 其它实时优化**

实时渲染领域还有很多优化方法值得尝试和应用，比如：

*   [《Moving Frostbite to PBR》](https://seblagarde.files.wordpress.com/2015/07/course_notes_moving_frostbite_to_pbr_v32.pdf)提出的 IES 光照模拟。
    
    ![](1679148484162.png)
    
*   [《Applying Visual Analytics to Physically-Based Rendering》](http://cg.ivd.kit.edu/publications/2018/visual_analytics_pbr/preprint.pdf)提出的可视化分析优化。
    
    ![](1679148484238.png)
    

### **5.5.3 移动端优化**

由于移动设备普遍的性能与 PC 机有一定的差距，所以要将 PBR 应用到移动端，性能优化的需求更加迫切。

上一小节提到的实时渲染优化同样适用于移动端，此外，还可针对移动端做一些特殊的优化：

*   简化光照模型。采用更少的样本采样数量，更简化的光照计算公式。
*   简化 shader。通过少量的 shader 指令或简化的数学运算可达到优化的目的。
*   启用引擎 Mobile 版本的资源和设置。Unity 和 Unreal Engine 都提供了移动版本的材质库和特殊的配置，在无特别需求下，尽量使用它们。
*   分级策略。针对不同分级的设备启用不同复杂度的材质和资源，可以有效解决高中低画质的兼容问题。

更多请参看[《Optimizing PBR》](https://community.arm.com/cfs-file/__key/communityserver-blogs-components-weblogfiles/00-00-00-20-66/siggraph2015_2D00_mmg_2D00_renaldas_2D00_slides.pdf)，还可参看笔者的另外一篇原创技术文章：[**《移动游戏性能优化通用技法》**](https://www.cnblogs.com/timlly/p/10463467.html)

## **5.6 PBR 的未来**

当今阶段，由于硬件、技术、理论等种种原因的限制，PBR 技术在很多时候只能是采取近似模拟的方法，特别是在实时渲染领域，甚至在很多中低端 PC 或移动端设备还无法运行 PBR 技术。

但是，这不妨碍我们想象 PBR 技术未来的趋势和前景。

### **5.6.1 基于纳米级别原理**

目前大多数 PBR 都是基于微平面（microfacet）的光照模型，micro 即微米（$10^{-6}m$），并且将反射模型简化成了几何光学，忽略了衍射、干扰、色散、光谱能量分布等等精确物理模型。

近两年，有人提出了基于纳米（nano，$10^{-9}m$）级别的光照模型，可以先看看它和微米级别的区别：

<table><thead><tr><th>微米几何（Microgeometry）</th><th>纳米几何（Nanogeometry）</th></tr></thead><tbody><tr><td>波瓣形状取决于表面统计（微米级别的 NDF）</td><td>波瓣形状取决于表面统计（纳米级别的光谱能量分布）</td></tr><tr><td>忽略光波波长</td><td>强依赖于光波波长</td></tr><tr><td>入射角通过可见性概率影响表面</td><td>入射角通过透视收缩（foreshortening）概率影响表面</td></tr></tbody></table>

也就是说纳米级别理论引入了 SPD、光波波长，先进的表面统计，使得光照渲染更加物理正确真实了。

![](1679148484291.png)

_纳米几何将引入光波长、SPD 等，会考虑光的衍射、干扰、色散等现象，显得更加物理真实。_

当然，这种技术目前只能用于电影级别的离线渲染，未来还有很长一段时间才能进入实时渲染领域的视野。

### **5.6.2 更精确的光照模型**

当前的 PBR 光照模型，包括 Cook-Torrance 及 BSSRDF，大多是基于一维的曲线拟合。

![](1679148484315.png)

从上图可以看出，由于是一维曲线，所以它们都是从中心向周边散开的圆形形状，只是圆的过渡稍有不同。

若是引入考虑光波波长的纳米级别的 SPD（光谱能量分布），则可以引入更加复杂的二维光照模型曲线图：

![](1679148484377.png)

上图可以看出，光的分布曲线不再是圆形形状，而是变成复杂的类似棉花絮状的二维图。这种才是更接近真实世界的光照曲线拟合。

虽然目前这种技术开销非常昂贵，但相信未来不久，这种技术会逐渐成为主流。

### **5.6.3 离线技术实时化**

[5.4 预计算技术](#5.4 预计算技术) 中提到了很多预渲染、预卷积技术，这些都是为了减轻实时渲染的负担。而且实时渲染部分采用大量近似、简化的手段，使得光照不那么物理正确和真实。

未来若干年，离线渲染技术将会逐渐引入到实时渲染领域，使得实时渲染能够获得更加真实的渲染效果。

这里所指的离线技术包括但不限于：光照追踪、路径追踪、全局光照、环境光、局部静态光、烘焙光。

若是能将这些技术引入到实时渲染领域，那将是振奋人心的。近期发布的 Unreal Engine 4.22 的版本已经支持实时光线追踪技术：

![](1679148484408.png)

_短片《Troll》展现虚幻引擎 4.22 的全新光线追踪功能，能够实时渲染电影级别的画质。_

相信这只是开始，未来离线技术实时化的步伐将会越来越快。

### **5.6.4 新兴理论和技术**

近年来图形学的技术蓬勃发展，围绕着 PBR 为中心的新兴技术和理论百花齐放，相信未来也是如此，而且新的理论和技术会成倍加速发展。

这些新兴技术涵盖了基础物理学、光学原理、数学模型、算法和数据结构、计算机语言、渲染技术、图形 API、GPU 架构等等软件和硬件领域。

### **5.6.5 更多应用领域**

未来随着基础理论、软件和硬件的发展，PBR 的发展迅猛，将会得到更广泛地应用。横向维度将涵盖各行各业，纵向维度覆盖高中端层次的设备。

例如，沉浸式 4D 影院，虚拟与现实混合的游戏和教学互动（下图），投影真实人像的幻影会议，电影画质的移动端游戏，甚至是电影画质的街机游戏。

![](1679148484465.png)

虽然目前看还有一段差距，但相信在不久的未来，借助 PBR 技术，这些预言将或多或少地呈现在大家面前。

总之，PBR 技术在未来的发展和前景非常值得期待。

# **六. 后记**

## **6.1 辅助工具**

一些辅助工具可以提升我们在工作或者研发新技术的效率。

利用数学曲线拟合工具，可以直观地看到函数在参数具体化后的结果，并协助我们得到想要的数据。

![](1679148484508.png)

_利用曲线拟合工具直观地展现数据曲线变化。_

### **6.1.1 曲线工具**

*   **MathLab**：MathLab 是老牌数学分析和建模的工具，功能强大，用户受众广，是理工科必备的软件之一。同样地可以用于图形渲染的曲线拟合。
    
*   **Mathematica**：跟 MathLab 类似，功能虽然不如 MathLab 多，但更轻巧，各有侧重，非常适合曲线拟合。
    
    ![](1679148484560.png)
    
*   **Excel**：你没看错，Excel 同样可以用于简单曲线的拟合，参考文献里有不少图就出自 Excel。
    

### **6.1.2 作图工具**

*   **GeoGebra**：功能强大，基本可以画出我们所需的图例，也可用以曲线拟合。它还有[在线版](https://www.geogebra.org/graphing)。
    
    ![](1679148484597.png)
    
*   **几何画板**：老牌数学绘图软件，数学教师的必备工具。
    

### **6.1.3 写作工具**

*   **Markdown**：一种无需我们关注格式的标准语法，可用简单的标记即可表达复杂多变的图文混合编排，使得写作者可以从繁琐的格式编排中抽身出来，更加专注于内容创作。本文就是基于 Markdown 格式撰写。
*   **$L^AT_EX$**：其多变复杂的编排可以满足各种变态的混排需求，特别适合用于数学公式的格式表达。本文的公式都是基于 _$L^AT_EX$_实现的。
*   **Typora**：一种支持 Markdown 编写的文本 IDE，小巧、灵活、功能较全，支持巨量文字撰写，性能较好，值得推荐。

### **6.1.4 BRDF Explorer**

迪斯尼原则发布的时候，随同发布了 [BRDF Explorer](https://www.disneyanimation.com/technology/brdf.html)，用于查看 BRDF 的各种公式参数和材质属性。

![](1679148484634.png)

## **6.2 更多资料**

什么？看完本文还觉得不过瘾？

那么，参考文献的众多资料等着你发掘。

骚年，走你~~

![](1679148484660.png)

# **特别说明**

*   大部分图片来自参考文献及网络，侵删。
    
*   感谢并致敬所有参考文献的作者。
    
*   由于时间仓促、水平有限，纰漏在所难免，恳请指正。
    
*   欢迎分享本文链接，但**未经允许，禁止转载**！
    

# **参考文献**

*   **书籍**
    *   [《Physically Based Rendering（ Third Edition）》](http://www.pbr-book.org/3ed-2018/contents.html)
    *   《Real-Time Rendering（4th Edition）》
    *   《Physically Based Shader Development for Unity》
    *   《Principles of Optics, 6th Edition》 (by MAX BORN and EMIL WOLF)
    *   《数学物理学百科全书 (卷 04)：规范场论》
    *   《数学物理学百科全书 (卷 05)：广义相对论》
    *   《Ray Tracing Gems》
    *   [《GPU Gems》](https://developer.nvidia.com/gpugems/GPUGems/gpugems_pref01.html)
    *   [《GPU Gems 2》](https://developer.nvidia.com/gpugems/GPUGems2/gpugems2_inside_front_cover.html)
    *   [《GPU Gems 3》](https://developer.nvidia.com/gpugems/GPUGems3/gpugems3_pref01.html)
    *   《高等数学微积分 (北大版)》
*   **专题、论文**
    *   [Physically Based Shading at Disney](https://disney-animation.s3.amazonaws.com/library/s2012_pbs_disney_brdf_notes_v2.pdf)
    *   [Real Shading in Unreal Engine 4](https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf)
    *   [Moving Frostbite to PBR](https://seblagarde.files.wordpress.com/2015/07/course_notes_moving_frostbite_to_pbr_v32.pdf)
    *   [Rendering the world of Far Cry 4](http://www.gdcvault.com/play/1022235/Rendering-the-World-of-Far)
    *   [【SIGGRAPH 2010 Course】Physically-Based Shading Models in Film and Game Production](http://renderwonk.com/publications/s2010-shading-course/)
    *   [【SIGGRAPH 2011 Course】Physically-Based Shading Models in Film and Game Production](http://renderwonk.com/publications/s2011-shading-course/)
    *   [【SIGGRAPH 2012 Course】Physically-Based Shading Models in Film and Game Production](http://renderwonk.com/publications/s2012-shading-course/)
    *   [【SIGGRAPH 2013 Course】Physically-Based Shading Models in Film and Game Production](http://renderwonk.com/publications/s2013-shading-course/)
    *   [【SIGGRAPH 2014 Course】Physically-Based Shading Models in Film and Game Production](http://renderwonk.com/publications/s2014-shading-course/)
    *   [【SIGGRAPH 2015 Course】Physically-Based Shading Models in Film and Game Production](http://renderwonk.com/publications/s2015-shading-course/)
    *   [【SIGGRAPH 2016 Course】Physically-Based Shading Models in Film and Game Production](http://renderwonk.com/publications/s2016-shading-course/)
    *   [【SIGGRAPH 2017 Course】Physically-Based Shading Models in Film and Game Production](http://renderwonk.com/publications/s2017-shading-course/)
    *   [Physically Based Shading and Image Based Lighting](https://www.trentreed.net/blog/physically-based-shading-and-image-based-lighting/)
    *   [disney animation papers](https://www.disneyanimation.com/technology/publications/#papers)
    *   [Physically Based Shading in Cry Engine](https://docs.cryengine.com/display/SDKDOC2/Physically+Based+Shading)
    *   Reflection Model Design for WALL-E and Up
    *   Background-Physics and Math of Shading (2013~2015)
    *   PBS Approx Models
    *   A Microfacet Based Coupled Specular-Matte BRDF Model
    *   A Multi-Ink Color-Separation Algorithm
    *   Arbitrarily Layered Micro-Facet Surfaces
    *   Microfacet Models for Refraction through Rough Surface
    *   Pre-Filter Antialiasing
    *   Printing Spatially-Varying Reflectance
    *   Volumetric Skin and Fabric Shading at Framestore
    *   [The Mathematics of Shading](http://www.scratchapixel.com/lessons/mathematics-physics-for-computer-graphics/mathematics-of-shading)
    *   [PHYSICALLY-BASED RENDERING REVOLUTIONIZES PRODUCT DEVELOPMENT](https://pny.com/File%20Library/Unassigned/Moor-Whitepaper-Download.pdf)
    *   [A MultiAgent System for Physically based Rendering Optimization](http://www.weiss-gerhard.info/publications/D02.pdf)
    *   [Physically Based Shading on Mobile](https://medium.com/spaceapetech/physically-based-shading-on-mobile-d7d4e90bb4bd)
    *   [Applying Visual Analytics to Physically-Based Rendering](http://cg.ivd.kit.edu/publications/2018/visual_analytics_pbr/preprint.pdf)
    *   [An Inexpensive BRDF Model for Physically based Rendering](http://mathinfo.univ-reims.fr/IMG/pdf/An_inexpensive_BRDF_model_for_Physically-based_rendering_-_Schlick.pdf)
    *   [Optimizing PBR](https://community.arm.com/cfs-file/__key/communityserver-blogs-components-weblogfiles/00-00-00-20-66/siggraph2015_2D00_mmg_2D00_renaldas_2D00_slides.pdf)
    *   [移动游戏性能优化通用技法](https://www.cnblogs.com/timlly/p/10463467.html)
*   **Wikipedia**
    *   [Optics](https://en.wikipedia.org/wiki/Optics)
    *   [Light](https://en.wikipedia.org/wiki/Light)
    *   [Polarization (waves)](https://en.wikipedia.org/wiki/Polarization_(waves))
    *   [Spectroscopy](https://en.wikipedia.org/wiki/Spectroscopy)
    *   [Visible spectrum](https://en.wikipedia.org/wiki/Visible_spectrum)
    *   [Wave–particle duality](https://en.wikipedia.org/wiki/Wave%E2%80%93particle_duality#)
    *   [Radiant energy](https://en.wikipedia.org/wiki/Radiant_energy)
    *   [Radiometry](https://en.wikipedia.org/wiki/Radiometry)
    *   [BRDF](https://en.wikipedia.org/wiki/Bidirectional_reflectance_distribution_function)
    *   [Surface roughness](https://en.wikipedia.org/wiki/Surface_roughness)
    *   [Maxwell's equations](https://en.wikipedia.org/wiki/Maxwell%27s_equations)
    *   [Solid angle](https://en.wikipedia.org/wiki/Solid_angle)
    *   [Normal distribution](https://en.wikipedia.org/wiki/Normal_distribution)
    *   [Low-discrepancy sequence](https://en.wikipedia.org/wiki/Low-discrepancy_sequence)
    *   [Rendering equation](https://en.wikipedia.org/wiki/Rendering_equation)
    *   [Mathematical physics](https://en.wikipedia.org/wiki/Mathematical_physics)
    *   [Oren–Nayar reflectance model](https://en.wikipedia.org/wiki/Oren%E2%80%93Nayar_reflectance_model)
    *   [Schlick's approximation](https://en.wikipedia.org/wiki/Schlick%27s_approximation)
*   **百度百科**
    *   [几何光学](https://baike.baidu.com/item/%E5%87%A0%E4%BD%95%E5%85%89%E5%AD%A6/869075)
    *   [波粒二象性](https://baike.baidu.com/item/%E6%B3%A2%E7%B2%92%E4%BA%8C%E8%B1%A1%E6%80%A7)
    *   [光能量](https://baike.baidu.com/item/%E5%85%89%E8%83%BD%E9%87%8F)
    *   [BRDF](https://baike.baidu.com/item/BRDF/2843176)
    *   [微积分公式大全](https://wenku.baidu.com/view/bfa2b645be1e650e52ea99a3.html)
    *   [大学积分入门](https://wenku.baidu.com/view/5a9412ca87c24028905fc307.html)
    *   [麦克斯韦方程组](https://baike.baidu.com/item/%E9%BA%A6%E5%85%8B%E6%96%AF%E9%9F%A6%E6%96%B9%E7%A8%8B%E7%BB%84/2717563)
    *   [数学物理](https://baike.baidu.com/item/%E6%95%B0%E5%AD%A6%E7%89%A9%E7%90%86/5363054)
*   **知乎、简书**
    *   [辐射度学基本量](https://www.zhihu.com/question/20286038/answer/64282761)
    *   [光的度量](https://zhuanlan.zhihu.com/p/21376124)
    *   [基于物理着色：BRDF](https://zhuanlan.zhihu.com/p/21376124)
    *   [如何深入浅出地讲解麦克斯韦方程组？](https://www.zhihu.com/question/36766702/answer/70426471)
    *   [PBR（物理渲染技术）在实时渲染中的概念与限制](https://zhuanlan.zhihu.com/p/32951655)
    *   [【基于物理的渲染（PBR）白皮书】系列](https://zhuanlan.zhihu.com/p/53086060)
    *   [基于物理着色系列](https://zhuanlan.zhihu.com/p/20091064)
    *   [PBR 渲染介绍](https://www.jianshu.com/p/d2c97d0646d5)
    *   [基于物理的渲染学习心得——面向使用的 PBR 理论](https://www.jianshu.com/p/7c9b666c3fe1)
    *   [PBR 渲染介绍](https://www.jianshu.com/p/d2c97d0646d5?open_source=weibo_search)
*   **博客园、CSDN**
    *   [基于物理的渲染技术（PBR）系列](%E5%9F%BA%E4%BA%8E%E7%89%A9%E7%90%86%E7%9A%84%E6%B8%B2%E6%9F%93%E6%8A%80%E6%9C%AF%EF%BC%88PBR%EF%BC%89%E7%B3%BB%E5%88%97)
    *   [基于物理渲染的基础理论](https://www.cnblogs.com/TracePlus/p/4056893.html)
    *   [基于物理的渲染](https://blog.csdn.net/coldkaweh/article/details/70187399)
    *   [Physically Based Rendering,PBRT 笔记](https://blog.csdn.net/pizi0475/article/details/48393933)
    *   [伽马空间与线性空间](https://blog.csdn.net/bill2ccssddnn/article/details/53423410)
    *   [线性渲染（Linear Rendering）和 Gamma Correction](https://blog.csdn.net/k46023/article/details/52489363/)
    *   [人眼到底等于多少像素](https://blog.csdn.net/github_38885296/article/details/77914436)
    *   [PBR Step by Step（一）立体角](https://www.cnblogs.com/jerrycg/p/4924761.html)
    *   [基于物理渲染的基础理论（译）](https://www.cnblogs.com/jim-game-dev/p/5425731.html)
    *   [基于物理的渲染 – 理论篇](https://blog.csdn.net/coldkaweh/article/details/70187399)
    *   [Lambert （兰伯特）光照模型](https://www.cnblogs.com/jqm304775992/p/4887779.html)
    *   [Unity3d 基于物理渲染 Physically-Based Rendering 之 specular BRDF](https://blog.csdn.net/wolf96/article/details/44172243)
    *   [Cook-Torrance lighting model](https://www.cnblogs.com/jqm304775992/p/5202973.html)
    *   [光照模型](https://www.cnblogs.com/mengdd/archive/2013/08/05/3238123.html)
    *   [Specular BRDF Reference](https://blog.csdn.net/aidlife/article/details/48972661)
    *   [Physically Based Rendering—BRDF 中 D 函数 NDF 的中文资料](https://blog.csdn.net/xingzhe2001/article/details/83897914)
    *   [手推系列——直观理解推导 Physically Based Rendering 的 BRDF 公式之微表面法线分布函数 NDF](https://blog.csdn.net/xingzhe2001/article/details/83829705)
    *   [Equirectangular Projection(ERP)](https://blog.csdn.net/lin453701006/article/details/71173090)
*   **Unreal、Unity**
    *   [SubSurface Profile Shading Model](https://docs.unrealengine.com/en-us/Engine/Rendering/Materials/LightingModels/SubSurfaceProfile)
    *   [Physically Based Materials in Unreal Engine 4](https://docs.unrealengine.com/en-us/Engine/Rendering/Materials/PhysicallyBased)
    *   [PBR: 应用于虚幻引擎 4 贴图和材质创建的启示](https://www.unrealengine.com/zh-CN/blog/gdcc2015-pbr)
    *   [Unity Standard Shader](https://docs.unity3d.com/Manual/shader-StandardShader.html)
    *   [Using Texture Masks](https://docs.unrealengine.com/en-us/Engine/Rendering/Materials/HowTo/Masking)
*   **LearnOpenGL**
    *   [PBR Tutorial - Theory](https://learnopengl.com/PBR/Theory)
    *   [PBR Tutorial - Lighting](https://learnopengl.com/PBR/Lighting)
    *   [PBR Tutorial - Diffuse-irradiance](https://learnopengl.com/PBR/IBL/Diffuse-irradiance)
    *   [PBR Tutorial - Specular-IBL](https://learnopengl.com/PBR/IBL/Specular-IBL)
    *   [PBR Tutorial - Advanced Lighting](https://learnopengl.com/Advanced-Lighting/Advanced-Lighting)
    *   [Cubemap convolution](https://learnopengl.com/PBR/IBL/Diffuse-irradiance)
    *   [Pre-computing the BRDF](https://learnopengl.com/PBR/IBL/Specular-IBL)
*   **其它**
    *   [Physically Based Rendering](http://www.codinglabs.net/article_physically_based_rendering.aspx)
    *   [Physically Based Rendering - Cook–Torrance](http://www.codinglabs.net/article_physically_based_rendering_cook_torrance.aspx)
    *   [How Your Eyes Work](https://www.aoa.org/patients-and-public/resources-for-teachers/how-your-eyes-work)
    *   [How Does The Human Eye Work?](https://www.nkcf.org/about-keratoconus/how-the-human-eye-works/)
    *   [为什么可见光是 “可见” 光](http://blog.sciencenet.cn/blog-711486-1052519.html)
    *   [光学](https://baike.sogou.com/v175274.htm;jsessionid=5DB12EA04CF58C2AE24C846AC00C3C5B.n2)
    *   [“波粒二象性” 本质是粒子磁矩与物质空间磁场相互作用的结果](http://blog.sina.com.cn/s/blog_449e3b970102xa1n.html)
    *   [A Multi-Ink Color-Separation Algorithm Maximizing Color Constancy](https://pdfs.semanticscholar.org/9e56/8b13ea51ca3c669186624566f672eb547857.pdf)
    *   [Unidirectional Reflectance of Imperfectly Diffuse Surfaces](https://www.onacademic.com/detail/journal_1000035238254910_7744.html#)
    *   [Adopting a physically based shading model](https://seblagarde.wordpress.com/2011/08/17/hello-world/)
    *   [SaschaWillems / Vulkan-glTF-PBR](https://juejin.im/repo/5a8127a4f265da02d800abba)
    *   [基于物理的渲染－用真实的环境光照亮物体](https://blog.uwa4d.com/archives/Study_IBL.html)
    *   [基于物理的渲染—更精确的微表面分布函数 GGX](https://blog.uwa4d.com/archives/1582.html)
    *   [LaTeX Math document](https://www.latex-project.org/help/documentation/amsldoc.pdf)
    *   [常用数学符号的 LaTeX 表示方法](http://www.mohu.org/info/symbols/symbols.htm)
    *   [LaTeX/Mathematics](https://en.wikibooks.org/wiki/LaTeX/Mathematics)
    *   一份不太简短的 LATEX 2ε 介绍
    *   [The Beginner’s Guide to Physically Based Rendering in Unity](https://blog.teamtreehouse.com/beginners-guide-physically-based-rendering-unity)
    *   [Image Based Lighting](https://chetanjags.wordpress.com/2015/08/26/image-based-lighting/)
    *   [Using Image Based Lighting (IBL)](https://www.indiedb.com/features/using-image-based-lighting-ibl)
    *   [Converting a Cubemap into Equirectangular Panorama](https://stackoverflow.com/questions/34250742/converting-a-cubemap-into-equirectangular-panorama)
    *   [使用基于物理规则的渲染，你也可以做到！](http://gad.qq.com/program/translateview/7196156)
    *   [Does PBR incur a performance penalty by design?](https://computergraphics.stackexchange.com/questions/1568/does-pbr-incur-a-performance-penalty-by-design)
    *   [Lec 2: Shading Models](http://www.cs.cornell.edu/courses/cs5625/2013sp/lectures/Lec2ShadingModelsWeb.pdf)