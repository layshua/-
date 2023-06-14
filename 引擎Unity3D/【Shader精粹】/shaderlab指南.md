[Unity小白的TA之路-Shader开发|图形渲染管线|URP|性能优化|图形渲染|PostProcessing (91maketop.github.io)](https://91maketop.github.io/ta/#/README)
# ShaderLab 语法基础
## ShaderLab 组织结构
Shader 中可以编写多个子着色器（SubShader），但至少需要一个。

在应用程序运行过程中，GPU 会先检测第一个子着色器能否正常运行，如果不能正常运行就会再检测第二个，以此类推。
假如当前 GPU 的硬件版本太旧，以至于所有的子着色器都无法正常运行时，则执行最后的回退（Fallback）命令，运行指定的一个基础着色器。

如果编写的是顶点-片段着色器（Vertex-Fragment Shader），每个子着色器中还会包含一个甚至多个 Pass。在运行的过程中，如果某个子着色器能够在当前 GPU 上运行，那么该子着色器内的所有 Pass 会依次执行，每个 Pass 的输出的结果会以指定的方式与上一步的结果进行混合，最终输出。

如果编写的是表面着色器（Surface Shader），着色器的代码也是包含在子着色器中，但是与顶点-片段着色器不同的是，表面着色器不会再嵌套 Pass。系统在编译表面着色器的时候会自动生成多个对应的 Pass，最终编译出来的 Shader 本质上就是顶点-片段着色器。


## 名称

Shader 程序的第一行代码用来声明该 Shader 的名称以及所在路径。

```
Shader "Unlit/NewUnlitShader"
```

这一行代码的意思是：这个 Shader 位于 Unlit 路径里，名称为 NewUnlitShader。最终在材质设置面板中选择 Shader 的下拉菜单，如图
![[Pasted image 20230614182320.png|450]]

当然也可以多加几级路径，例如：
```
Shader "Unlit/Path_1/Path_2/NewUnlitShader"
```

## Properties
```c file:所有类型属性汇总
Properties
{
    _MyFloat ( "Float Property",Float) = 1                 //浮点类型
    _MyRange ( "Range Property" , Range(0, 1)) = 0.1       //范围类型
    _MyColor ( "Color Property" ,Color) = (1, 1, 1, 1)     //颜色类型
    _MyVector ( "Vector Property" , Vector) = (0, 1, 0, 0) //向量类型
    _MyTex ( "Texture Property", 2D) = "white"{}           //2D 贴图类型
    _MyCube ( "Cube Property" ,Cube) = ""{}                //立方体贴图类型
    _My3D ( "3D Property", 3D)= ""{}                       //3D 贴图类型

}
```

Unity Shader 的属性主要分为三大类：数值、颜色和向量、纹理贴图，每一条属性都是按照以下语法进行定义的：
`_Name ("Display Name", type) = defaultValue ［{options}］`
（1）`_Name`：属性的名字。为了方便获取，通常在名字的最前加一个下画线，后续在整个 Shader 中都将使用这个名称来获取该属性。
（2）`Display Name`：在材质面板中显示出来的名称。
（3）`type`：属性的类型。
（4）`defaultValue`：将 Shader 指定给材质的时候初始化的默认值。

对应的声明：
```cs file:在CG中声明属性变量
float _MyFloat;         //浮点类型
float _MyRange;         //范围类型
fixed4 _MyColor;        //颜色类型
float4 _MyVector;       //向量类型

sampler2D _MyTex;       //2D贴图类型
float4 _MyTex_ST;       //声明纹理变量的Tiling和Offset

samplerCUBE _MyCube;    //立方体贴图类型

sampler3D _My3D;        //3D贴图类型
```

#### 数值属性
**Unity Shader 的数值类属性基本都是浮点型（Float）数据，虽然 Unity 提供了整数型（Int）数据，但是在编译的时候最终都会转化为浮点型数据。**

```
name ("display name", Float) = number
name ("display name", Int) = number
name ("display name", Range (min, max)) = number
```

Float 是任意数值的浮点型数据，在材质面板上作为数字输入框显示。
Range 是一个介于最大值和最小值之间的浮点型数据，在材质面板作为滑动条显示。

#### 颜色和向量属性

```
name ("display name", Color) = (number,number,number,number)
name ("display name", Vector) = (number,number,number,number)
```

使用给定 RGBA 分量的默认值定义颜色属性，或使用默认值定义 4D 矢量属性。颜色属性会显示拾色器，并根据颜色空间按需进行调整。矢量属性显示为四个数字字段。

有一点需要注意的是：用 Photoshop 处理图片一般会使用8位深度图，每个通道的亮度最大值为 $2^8=256$，由于从 $0$ 开始计算，因此数值范围是 $[0，255]$。
而**在 Shader 中，每个分量的数值范围是 $[0,1]$**
![[Pasted image 20230614183329.png|500]]
#### 纹理贴图属性

```
name ("display name", 2D) = "defaulttexture" {}
name ("display name", Cube) = "defaulttexture" {}
name ("display name", 3D) = "defaulttexture" {}
```

（1）2D 属性是纹理类属性中最常使用的，漫反射贴图、法线贴图等都属于 2D 类型。
（2）Cube 全称 Cube map texture（立方体纹理），是由前、后、左、右、上、下 6 张有联系的 2D 贴图拼成的立方体，主要用作反射，例如 Skybox 和 Reflection Prob。
（3）3D 纹理只能被脚本创建

2D 类型的属性，默认值可以为空字符串，也可以是内置的表示颜色的字符串：`“white”（RGBA: 1，1，1，1）`，`“black”（RGBA：0，0，0，0）`，`“gray”（RGBA：0.5，0.5，0.5，0.5）`，`“bump”（RGBA：0.5，0.5，1，0.5）` 和 `“red”（RGBA：1，0，0，0）`。
至于非2D 类型的属性（Cube，3D，2DArray），默认值为空字符串。当材质没有指定 Cubemap 或者3D 或者2DArray 纹理的时候，会默认使用 `gray（RGBA：0.5，0.5，0.5，0.5）`。

> [!info] 纹理贴图类的属性最后都有一对空的花括号
这是因为在 Unity 5.0 之前的版本，纹理属性可以在花括号内添加选项，用于控制固定函数纹理坐标的生成。但是**该功能在 Unity 5.0 及以后的版本中已经被移除，所以无须考虑这个问题，直接加上一对空的花括号即可。**

### 详细信息

着色器中的每个属性均通过 **name** 引用（在 Unity 中，着色器属性名称通常以下划线开头）。属性在材质检视面板中将显示为 **display name**。每个属性都在等号后给出默认值：

- 对于 _Range_ 和 _Float_ 属性，默认值仅仅是单个数字，例如“13.37”。
- 对于 _Color_ 和 _Vector_ 属性，默认值是括在圆括号中的四个数字，例如“(1,0.5,0.2,1)”。
- 对于 2D 纹理，默认值为空字符串或内置默认纹理之一：“white”（RGBA：1,1,1,1）、“black”（RGBA：0,0,0,0）、“gray”（RGBA：0.5,0.5,0.5,0.5）、“bump”（RGBA：0.5,0.5,1,0.5）或“red”（RGBA：1,0,0,0）。其中“bump”通常用于法线体贴图的默认值。
- 对于非 2D 纹理（立方体、3D 或 2D 数组），默认值为空字符串。如果材质未指定立方体贴图/3D/数组纹理，则使用灰色（RGBA：0.5,0.5,0.5,0.5）。

稍后在着色器的固定函数部分中，可使用括在方括号中的属性名称来访问属性值：*_[name]**。例如，可通过声明两个整数属性（例如“_SrcBlend“和”*DstBlend”）来使混合模式由材质属性驱动，然后让 [Blend 命令](https://91maketop.github.io/ta/#/SL-Blend.html)使用它们：`Blend [_SrcBlend] [_DstBlend]`。

`Properties` 代码块中的着色器参数被序列化为[材质](https://91maketop.github.io/ta/#/Materials.html)数据。[着色器程序](https://91maketop.github.io/ta/#/SL-ShaderPrograms.html)实际上可以有更多参数（如矩阵、矢量和浮点数），这些参数在运行时从代码中在材质上设置，但如果它们不是 Properties 代码块的一部分，则不会保存它们的值。这对于完全由脚本代码驱动的值最有用（使用 [Material.SetFloat](https://91maketop.github.io/ta/#/../ScriptReference/Material.SetFloat.html) 和类似函数）。

### 属性特性和绘制器

在属性前面，可指定可选的特性（用方括号括起）。这些是 Unity 可以识别的特性，或者它们可以指示您自己的 [MaterialPropertyDrawer 类](https://91maketop.github.io/ta/#/../ScriptReference/MaterialPropertyDrawer.html) 来控制它们在[材质检视面板](https://91maketop.github.io/ta/#/class-Material.html)中的呈现方式。Unity 可以识别的特性包括：

- `[HideInInspector]` - does not show the property value in the Material inspector.
- `[NoScaleOffset]` - material inspector will not show Texture tiling/offset fields for Texture properties with this attribute.
- `[Normal]` - indicates that a Texture property expects a normal-map.
- `[HDR]` - indicates that a Texture property expects a high-dynamic range (HDR) Texture.
- `[Gamma]` - 表示在 UI 中将浮点/矢量属性指定为 sRGB 值（就像颜色一样），并且可能需要根据使用的颜色空间进行转换。请参阅[着色器程序中的属性](https://91maketop.github.io/ta/#/SL-PropertiesInPrograms.html)。
- `[PerRendererData]` - indicates that a property will be coming from per-renderer data in the form of a [MaterialPropertyBlock](https://91maketop.github.io/ta/#/../ScriptReference/MaterialPropertyBlock.html). Material inspector shows these properties as read-only.
- `[MainTexture]` - indicates that a property is the [main texture for a Material](https://91maketop.github.io/ta/#/../ScriptReference/Material-mainTexture.html). By default, Unity considers a texture with the property name name `_MainTex` as the main texture. Use this attribute if your texture has a different property name, but you want Unity to consider it the main texture. If you use this attribute more than once, Unity uses the first property and ignores subsequent ones.
- `[MainColor]` - indicates that a property is the [main color for a Material](https://91maketop.github.io/ta/#/../ScriptReference/Material-color.html). By default, Unity considers a color with the property name name `_Color` as the main color. Use this attribute if your color has a different property name, but you want Unity to consider it the main color. If you use this attribute more than once, Unity uses the first property and ignores subsequent ones.

### 属性的常用特性
1. NoScaleOffset（隐藏 Tiling 和 Offset）

`[NoScaleOffset]_MainTex ("Texture", 2D) = "white" {}`

如果不想在InSpector面板上被人修改，可以用这个特性隐藏掉

2. Normal（法线纹理）

`[Normal]_NormalTexture("Normal Texture",2D) = "white" {}`

3. HDR

`[HDR]_MainColor("Main Color",Color) = (1,1,1,1)`

4. HideInInSpector（InSpector面板隐藏）

`[HideInInSpector]_FloatValue("Float Value",Float) = 0`

5. Toggle

`[Toggle]_IsFloat("Is Float",Float) = 0`

6. IntRange（整数滑动条）

`[IntRange]_Alpha("Alpha",Range(0,255)) = 0`

7. Space（垂直间隔）

`[Space]_Prop1("Prop1",Float) = 0`

也可以加数字增大间隔

`[Space(50)]_Prop2("Prop2",Float) = 0`

8. Header（标题头）

`[Header(Title)]_Title("Title",Float) = 0`

9. PowerSlider（指数式的滑动条）

`[PowerSlider(3.0)]_Shininess("Shininess",Range(0,1)) = 0`

10. Enum（枚举）

`[Enum(Zero,0,One,1,Two,2,Three,3)] _Number ("Number", Float) = 0`

11. KeywordEnum（枚举）

`[KeywordEnum(None,Add,Multiply)]_Overlay("OverLay Mode",Float) = 0`

KeywordEnum和Enum使用上有些不同，区别在于KeywordEnum类似于if-else，同时在shader代码中需要处理

定义如下：

![img](https://91maketop.github.io/ta/ShaderLab%E7%AE%80%E6%98%8E%E6%89%8B%E5%86%8C%EF%BC%88%E5%86%85%E7%BD%AE%E7%AE%A1%E7%BA%BF%EF%BC%89/Shader%E5%B1%9E%E6%80%A7%E7%9A%84%E5%B8%B8%E7%94%A8%E7%89%B9%E6%80%A7.assets/20200421113212796.png)

使用如下：

![img](https://91maketop.github.io/ta/ShaderLab%E7%AE%80%E6%98%8E%E6%89%8B%E5%86%8C%EF%BC%88%E5%86%85%E7%BD%AE%E7%AE%A1%E7%BA%BF%EF%BC%89/Shader%E5%B1%9E%E6%80%A7%E7%9A%84%E5%B8%B8%E7%94%A8%E7%89%B9%E6%80%A7.assets/20200421113507377.png)

## SubShader
```cs file:SubShader的大致结构
SubShader
{
    //标签
    Tags { "TagName1" = "valuel" "TagName2" = "value2" ...}
    //渲染状态
    Cull Back

    Pass
    {
        //第一个 Pass
    }
    
    Pass
    {
        //第二个 Pass
    }
    ...
}
```

在 Unity 中，每一个 Shader 都会包含至少一个 SubShader。当 Unity 想要显示一个物体的时候，它就会去检测这些 SubShader，然后选择第一个能够在当前显卡运行的 SubShader。
**每个 SubShader 都可以设置一个或者多个标签（Tags）和渲染状态（States），然后定义至少一个 Pass**。在 SubShader 中设置的渲染状态会影响到该 SubShader 中所有的 Pass，如果想要某些状态不影响其他 Pass，可以针对某个 Pass 单独设置渲染状态。但是需要注意的是，部分渲染状态在 Pass 中并不支持。
### Tags

SubShader 通过标签来**确定什么时候以及如何对物体进行渲染。**

标签通过**键值对**的形式进行声明，并且没有使用数量的限制。如果有需要，可以使用任意多个标签。
```
Tags { "TagName1" = "Value1" "TagName2" = "Value2" }
```

#### Queue

在 SubShader 中可以使用 Queue（队列）标签**确定物体的渲染顺序**

|队列名称|**功能** |队列号|
| :---------- | :------------------------ | :------------------------ |
| Background  |指定背景渲染队列。最先执行渲染﹐一般用来渲染天空盒 (Skybox)或者背景|1000|
|Geometry|指定几何体渲染队列。非透明的几何体通常使用这个队列, 当没有声明渲染队列的时候，Unity 会默认使用这个队列|2000|
|AlphaTest|Alpha 测试的几何体会使用这个队列, 之所以从 Geometry 队列单独拆分出来，是因为当所有实体都绘制完之后再绘制 Alpha 测试会更高效 |2450|
| Transparent |在这个队列的几何体按由远及近的顺序进行绘制, 所有进行 Alpha 混合的几何体都应该使用这个队列, 例如玻璃材质、粒子特效等|3000|
| Overlay     |用来叠加渲染的效果，例如镜头光晕等, 放在最后渲染|4000|

除了使用 Unity 预定义的渲染队列，使用者也可以自己指定一个队列，例如：
`Tags { "Queue" = "Geometry+1" }`
这个队列的队列号其实就是 2001，表示在所有的非透明几何体绘制完成之后再进行绘制。
使用自定义的渲染队列在某些情况下非常有用，例如：透明的水应该在所有不透明几何体之后，透明几何体之前被绘制，所以透明水的渲染队列一般会使用`"Queue"="Transparent-1"`。

#### RenderType
RenderType（渲染类型）标签可以将 Shader 划分为不同的类别，用于后期进行 Shader 替换或者产生摄像机的深度纹理

|类型名称| 描述 |
|:--|:--|
|Opaque |用于普通 Shader, 例如: 不透明、自发光、反射、地形 Shader|
|Transparent|用于半透明 Shader, 例如: 透明﹑粒子|
|TransparentCutout|用于透明测试 Shader, 例如: 植物叶子|
|Background|用于 Skybox Shader|
|Overlay|用于 GUI 纹理、Halo、 Flare Shader |
|TreeOpaque|用于地形系统中的干|
|TreeTransparentCutout|用于地形系统中的树叶|
|TreeBillboard|用于地形系统中的 Billboarded 树|
|Grass|用于地形系统中的草|
|GrassBillboard|用于地形系统中的 Billboarded 草|
|  |  |

#### 禁用批处理
**当使用批处理（Batching）的时候，几何体会被变换到世界空间，模型空间会被丢弃。** 这会导致某些使用模型空间顶点数据的 Shader 最终无法实现所希望的效果。而开启 `DisableBatching`（禁用批处理）可以解决这个问题。
禁用批处理标签有三个数值可以使用：
（1）"DisableBatching"="True"：总是禁用批处理。
（2）"DisableBatching"="False"：不禁用批处理，这是默认数值。
（3）"DisableBatching"="LODFading"：当 LOD 效果激活的时候才会禁用批处理，主要用于地形系统上的树。

#### 禁止阴影投射
在游戏中，有很多特效类的物体并不需要对其他物体产生投影，这个时候可以使用`"ForceNoShadowCasting"`（禁止阴影投射）标签来达到需要实现的效果。只要将这个标签的数值设置为 true，那么使用这个 Shader 的物体就不会对其他物体产生投射阴影了。

#### 忽略 Projector
如果不希望物体受到 Projector（投影机）的投射，可以在 Shader 中添加 `IgnoreProjector` 标签。它有两个数值可以使用："True"和"False"，分别为忽略投射机和不忽略投射机。一般半透明的 Shader 都会开启这个标签。

- ! 除此之外，Unity 还提供了其他可以设置的 Tags，待补充...
### Pass 的渲染状态
如果想某些 Pass 的渲染状态不影响到其他的 Pass，可以在该 Pass 中单独设置渲染状态。这些渲染状态在 SubShader 中同样被允许使用，需要特别注意的是，在 SubShader 中使用会影响到该 SubShader 中的所有 Pass。

|渲染状态|数值|作用|
|:--|:--|:--|
|Cull|Cull Back/Front/ Off|设置多边形的剔除方式, 有背面剔除、正面剔除、不剔除﹐默认为 Back|
|ZTest|ZTest (Less/Greater/LEqual/GEqual/Equal/NotEquall /Always)|设置深度测试的对比方式, 默认为 LEqual |
|ZWrite|ZWrite On/ Off|设置是否写入深度缓存, 默认为 On |
|Blend|Blend  sourceBlendMode  destBlendMode|设置渲染图像的混合方式|
|ColorMask|ColorMask RGB/A/0/或者 R、G、B、A 的任意组合|设置颜色通道的写入蒙版﹐默认蒙版为 RGBA, 当设置为 0 时, 则无法写入任何颜色|

- ! 除此之外，Unity 还提供了其他可以设置的渲染状态，待补充...

### Fallback
Fallback 在所有 SubShader 之后进行定义。当所有的 SubShader 都不能在当前显卡上运行的时候，就会运行 Fallback 定义的 Shader。它的语法如下：
`Fallback "name"`
最常用于 Fallback 的 Shader 为 Unity 内置的 Diffuse。
如果觉得某些 Shader 肯定可以在目标显卡上运行，没有指定 Fallback 的必要，可以使用 Fallback Off 关闭 Fallback 功能，或者直接什么都不写。

# CG 语法基础
在 Unity Shader 中，ShaderLab 语言只是起到组织代码结构的作用，而真正实现渲染效果的部分是用 CG 语言编写的。
CG 程序片段通过指令嵌入在 Pass 中，夹在 Pass 中的指令 CGPROGRAM 和 ENDCG 之间

在 CG 程序片段之前，通常需要先使用 `#pragma`声明编译指令

## 编译指令
### 编译目标等级

|编译指令 |作用|
|:--|:--|
|`#pragma vertex name`|定义顶点着色器的名称, 通常会使用 vert|
|`#pragma fragment name`|定义片段着色器的名称, 通常会使用 frag|
|`#pragma target name`|定义 Shader 要编译的目标级别﹐默认 2.5|

当编写完 Shader 程序之后，其中的 CG 代码可以被编译到不同的 Shader Models（简称 SM）中，为了能够使用更高级的 GPU 功能，需要对应使用更高等级的编译目标。同时，高等级的编译目标可能会导致 Shader 无法在旧的 GPU 上运行。

声明编译目标的级别可以使用 `#pragma target name` 指令，或者也可以使用 `#pragma require feature` 指令直接声明某个特定的功能，

```c
#pragma target 3.5  //目标等级3.5
#pragma require geometry tessellation //需要儿何体细分功能
```

### 渲染平台
默认情况下，Unity 会为所有支持的平台编译一份 Shader 程序，不过可以通过编译指令 `#pragma only_renderers PlatformName` 或者 ` #pragma exclude_renderers PlatformName` 指定编译某些平台或不编译某些平台

![[Pasted image 20230614194552.png]]

## 着色器函数
顶点函数和片段函数中支持的数据类型
![[Pasted image 20230614194727.png]]

## 语义
参数后被冒号隔开并且全部大写的关键词就是语义。

当使用 CG 语言编写着色器函数的时候，函数的输入参数和输出参数都需要填充一个语义（Semantic）来表示它们要传递的数据信息。语义可以执行大量烦琐的操作，使用户能够避免直接与 GPU 底层进行交流。

### 顶点着色器输入语义
![[Pasted image 20230614195139.png]]
当顶点信息包含的元素少于顶点着色器输入所需要的元素时，**缺少的部分会被0填充，而 w 分量会被1填充**。例如：顶点的 UV 坐标通常是二维向量，只包含 x 和 y 元素。如果输入的语义 TEXCOORD0被声明为 float4类型，那么顶点着色器最终获取到的数据将变成（x，y，0，1）。

### 顶点着色器输出和片段着色器输入语义
在整个渲染流水线中，顶点着色器最重要的一项任务就是需要输出顶点在裁切空间中的坐标，这样 GPU 就可以知道顶点在屏幕上的栅格化位置以及深度值。在顶点函数中，这个输出参数需要使用 float4类型的 `SV_POSITION` 语义进行填充。

顶点着色器产生的输出值将会在三角形遍历阶段经过插值计算，最终作为像素值输入到片段着色器。换句话说，顶点着色器的输入即为片段着色器的输入。

![[Pasted image 20230614195313.png]]

片段着色器会自动获取顶点着色器输出的裁切空间顶点坐标，**所以片段函数输入的 SV_POSITION 可以省略**。这也解释了为什么有些 Shader 的片段函数中只有输出参数，但是没有输入参数。
**需要特别注意的是，与顶点函数的输入语义不同，`TEXCOORDn` 不再特指模型的 UV 坐标，`COLORn` 也不再特指顶点颜色。它们的使用范围更广，可以用于声明任何符合要求的数据，所以在使用过程中不要被语义的名称欺骗了。**
### 片段着色器输出语义
片段着色器通常只会输出一个 fixed4 类型的颜色信息，输出的值会存储到渲染目标（Render Target）中，输出参数使用 `SV_TARGET` 语义进行填充。



# Unity 的 include 文件

为了提高代码的重复使用率以及 Shader 的编写速度，Unity 提供了一系列的包含文件，其中有预先定义的变量、各种辅助函数和空间变换矩阵等。

在编写 Shader 时，只需要使用编译指令提前把对应的文件包含进 Shader，就可以直接使用了。与其他编译指令一样，包含文件的声明也要写在 CG 代码块内，它的语法结构如下所示：
```
CGPROGRAM
// ...
#include"UnityCG.cginc"
// ...
ENDCG
```

Unity 中的着色器 include 文件采用 `.cginc` 扩展名，如果使用的是 Windows 系统，Unity 所有的包含文件存放在安装目录`\Editor\Data\CGIncludes`\路径下。

内置的着色器 include 文件包括：

- `HLSLSupport.cginc` **（自动包含）** 用于跨平台着色器编译的 helper 宏和定义。

- `UnityShaderVariables.cginc` **（自动包含）** 常用的全局变量。

- `UnityCG.cginc` 常用的 [helper 函数](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinFunctions.html)。

- `AutoLight.cginc` 光照和阴影功能，例如[表面着色器](https://docs.unity3d.com/cn/2021.1/Manual/SL-SurfaceShaders.html)在内部使用此文件。

- `Lighting.cginc` 标准[表面着色器](https://docs.unity3d.com/cn/2021.1/Manual/SL-SurfaceShaders.html)光照模型；当您编写表面着色器时会自动包含。

- `TerrainEngine.cginc` 地形和植被着色器的 helper 函数。

## HLSLSupport.cginc



## UnityShaderVariables.cginc



## UnityCG.cginc

Unity 着色器中通常会包含此文件。此文件声明大量[内置 helper 函数](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinFunctions.html) （实用函数）和数据结构。

![[Pasted image 20230614222148.png]]

### 顶点变换函数

|**功能：**|**描述：**|
|:--|:--|
| `float4 UnityObjectToClipPos(float3 pos)` |将对象空间中的点变换到齐次坐标中的摄像机裁剪空间。等效于 `mul(UNITY_MATRIX_MVP, float4(pos, 1.0))` |
|`float3 UnityObjectToViewPos(float3 pos)`|将对象空间中的点变换到视图空间。这等效于 __mul(UNITY_MATRIX_MV, float4(pos, 1.0)).xyz__，应该在适当的位置使用。|


####### UnityCG.cginc 中的通用 helper 函数

|**功能：**|**描述：**|
|:--|:--|
|`float3 WorldSpaceViewDir (float4 v)`|返回从给定对象空间顶点位置朝向摄像机的世界空间方向（未标准化）。|
|`float3 ObjSpaceViewDir (float4 v)`|返回从给定对象空间顶点位置朝向摄像机的对象空间方向（未标准化）。|
|`float2 ParallaxOffset (half h, half height, half3 viewDir)`|计算视差法线贴图的 UV 偏移。|
|`fixed Luminance (fixed3 c)`|将颜色转换为亮度（灰阶）。|
|`fixed3 DecodeLightmap (fixed4 color)`|从 Unity 光照贴图（RGBM 或 dLDR，具体取决于平台）解码颜色。|
|`float4 EncodeFloatRGBA (float v)`|将 `[0..1)` 范围浮点数编码为 RGBA 颜色，用于存储在低精度渲染目标中。 |
|`float DecodeFloatRGBA (float4 enc)`|将 RGBA 颜色解码为浮点数。|
|`float2 EncodeFloatRG (float v)`|将` [0.1) `范围浮点数编码为 float2。|
|`float DecodeFloatRG (float2 enc)`|解码先前编码的 RG 浮点数。|
|`float2 EncodeViewNormalStereo (float3 n)`|将视图空间法线编码为 0 到 1 范围内的两个数字。|
|`float3 DecodeViewNormalStereo (float4 enc4)`|从 enc4.xy 解码视图空间法线。|

####### UnityCG.cginc 中的前向渲染 helper 函数

仅当使用前向渲染（ForwardBase 或 ForwardAdd 通道类型）时，这些函数才有用。

|**功能：**|**描述：**|
|:--|:--|
|`float3 WorldSpaceLightDir (float4 v)`|根据给定的对象空间顶点位置计算朝向光源的世界空间方向（未标准化）。|
|`float3 ObjSpaceLightDir (float4 v)`|根据给定对象空间顶点位置计算朝向光源的对象空间方向（未标准化）。|
|`float3 Shade4PointLights (...)`|计算四个点光源的光照，将光源数据紧密打包到矢量中。前向渲染使用它来计算每顶点光照。|

####### UnityCG.cginc 中的屏幕空间 helper 函数

以下 helper 函数可计算用于采样屏幕空间纹理的坐标。它们返回 `float4`，其中用于纹理采样的最终坐标可以通过透视除法（例如 `xy/w`）计算得出。

这些函数还处理渲染纹理坐标中的[平台差异](https://docs.unity3d.com/cn/2021.1/Manual/SL-PlatformDifferences.html)。

|**功能：**|**描述：**|
|:--|:--|
|`float4 ComputeScreenPos (float4 clipPos)`|计算用于执行屏幕空间贴图纹理采样的纹理坐标。输入是裁剪空间位置。|
|`float4 ComputeGrabScreenPos (float4 clipPos)`|计算用于 [GrabPass](https://docs.unity3d.com/cn/2021.1/Manual/SL-GrabPass.html) 纹理采样的纹理坐标。输入是裁剪空间位置。|

####### UnityCG.cginc 中的顶点光照 helper 函数

仅当使用每顶点光照着色器（“Vertex”通道类型）时，这些函数才有用。

|**功能：**|**描述：**|
|:--|:--|
|`float3 ShadeVertexLights (float4 vertex, float3 normal)`|根据给定的对象空间位置和法线计算四个每顶点光源和环境光的光照。|


# 变体和关键字


可以编写着色器代码片段来共享通用代码，但在启用或禁用给定关键字时具有不同功能。Unity 编译这些着色器代码片段时，它将为已启用和已禁用关键字的不同组合创建单独的着色器程序。这些各个着色器程序被称为着色器变体。

由于项目工作流程的原因，着色器变体可能会很有用；可以将同一着色器分配给不同材质，但要为每种材质配置不同关键字。这意味着可以在同一个地方编写和维护着色器代码，并减少项目中的着色器资源。还可以使用着色器变体，通过启用或禁用关键字在运行时更改着色器行为。

具有大量变体的着色器被称为“大型着色器”或“超级着色器”。Unity 的标准着色器就是此类着色器的一个示例。

## 使用着色器变体和关键字

### 创建着色器变体

使用下列[pragma 指令](https://docs.unity3d.com/cn/2021.1/Manual/SL-PragmaDirectives.html)之一:

- `#pragma multi_compile`
- `#pragma multi_compile_local`
- `#pragma shader_feature`
- `#pragma shader_feature_local`

You can use these directives in Unity shaders (including [surface shaders](https://docs.unity3d.com/cn/2021.1/Manual/SL-SurfaceShaders.html)) and [compute shaders](https://docs.unity3d.com/cn/2021.1/Manual/class-ComputeShader.html).

If a keyword affects only a single shader stage, you can add a suffix to these directives to reduce redundant shader compilation work. For more information, see [Stage-specific keyword directives](https://docs.unity3d.com/cn/2021.1/Manual/SL-MultipleProgramVariants.html#stage-specific-keyword-directives).

然后，Unity 使用不同的预处理器指令来多次编译同一着色器代码。

### 启用和禁用着色器关键字

要启用和禁用着色器关键字，请使用以下 API：

- [Shader.EnableKeyword](https://docs.unity3d.com/cn/2021.1/ScriptReference/Shader.EnableKeyword.html)：启用全局关键字
- [Shader.DisableKeyword](https://docs.unity3d.com/cn/2021.1/ScriptReference/Shader.DisableKeyword.html)：禁用全局关键字
- [CommandBuffer.EnableShaderKeyword](https://docs.unity3d.com/cn/2021.1/ScriptReference/Rendering.CommandBuffer.EnableShaderKeyword.html)：使用 `CommandBuffer` 来启用全局关键字
- [CommandBuffer.DisableShaderKeyword](https://docs.unity3d.com/cn/2021.1/ScriptReference/Rendering.CommandBuffer.DisableShaderKeyword.html)：使用 `CommandBuffer` 来禁用全局关键字
- [Material.EnableKeyword](https://docs.unity3d.com/cn/2021.1/ScriptReference/Material.EnableKeyword.html)：为常规着色器启用本地关键字
- [Material.DisableKeyword](https://docs.unity3d.com/cn/2021.1/ScriptReference/Material.DisableKeyword.html)：为常规着色器禁用本地关键字
- [ComputeShader.EnableKeyword](https://docs.unity3d.com/cn/2021.1/ScriptReference/ComputeShader.EnableKeyword.html): enable a local keyword for a compute shader
- [ComputeShader.DisableKeyword](https://docs.unity3d.com/cn/2021.1/ScriptReference/ComputeShader.DisableKeyword.html): disable a local keyword for a compute shader

启用或禁用关键字时，Unity 会使用相应变体。

### 从打包中裁剪Shader变体

可以通过下列API阻止Shader变体被打包，从而节省打包时间和文件体积:

- [IPreprocessShaders.OnProcessShader](https://docs.unity3d.com/cn/2021.1/ScriptReference/Build.IPreprocessShaders.OnProcessShader.html): receive a callback before Unity compiles a regular shader into a build ): receive a callback before Unity compiles a compute shader into a build
- [IPreprocessComputeShaders.OnProcessComputeShader](scriptref:Build.IPreprocessComputeShaders.OnProcessComputeShader): receive a callback before Unity compiles a compute shader into a build

For more information on this subject, see the Unity blog post [Stripping scriptable shader variants](https://blogs.unity3d.com/2018/05/14/stripping-scriptable-shader-variants/) .

## multi_compile 的工作方式

指令示例：

```
# pragma multi_compile FANCY_STUFF_OFF FANCY_STUFF_ON
```

此指令示例生成两个着色器变体：一个定义了 `FANCY_STUFF_OFF`，另一个定义了 `FANCY_STUFF_ON`。在运行时，Unity 根据材质或全局着色器关键字来激活其中一个变体。如果这两个关键字均未启用，则 Unity 使用第一个关键字（在此示例中为 `FANCY_STUFF_OFF`）。

可以在 multi_compile 行中添加两个以上的关键字。例如：

```
# pragma multi_compile SIMPLE_SHADING BETTER_SHADING GOOD_SHADING BEST_SHADING
```

此指令示例生成四个着色器变体：`SIMPLE_SHADING`、`BETTER_SHADING`、`GOOD_SHADING` 和 `BEST_SHADING`。

为了生成未定义预处理器宏的着色器变体，请添加一个只有下划线 (`__`) 的名称。这是避免用完两个关键字的常用方法，因为在一个项目中可以使用的关键字数量有限（请参阅后面的[关键字限制](https://docs.unity3d.com/cn/2021.1/Manual/SL-MultipleProgramVariants.html#KeywordLimits)部分）。例如：

```
# pragma multi_compile __ FOO_ON
```

此指令生成两个着色器变体：一个未定义任何关键字 (`__`)，另一个定义了 `FOO_ON`。

## shader_feature 与 multi_compile 之间的区别

`shader_feature` 与 `multi_compile` 非常相似。唯一的区别是 Unity 没有将 `shader_feature` 着色器的未用变体包含在最终构建中。因此，应该将 `shader_feature` 用于材质中设置的关键字，而 `multi_compile` 更适合通过代码来全局设置的关键字。

此外，有一个只包含一个关键字的速记符号：

```
# pragma shader_feature FANCY_STUFF
```

这只是 `#pragma shader_feature _ FANCY_STUFF` 的快捷方式。它会扩展为两个着色器变体（第一个没有定义；第二个有定义）。

## 合并多个 multi_compile 行

如果提供 `multi_compile` 行，Unity 将会针对所有可能的行组合来编译生成的着色器。例如：

```
# pragma multi_compile A B C
# pragma multi_compile D E
```

这会为第一行生成三个变体，为第二行生成两个变体。总共生成六个着色器变体（A+D、B+D、C+D、A+E、B+E、C+E）。

每个 `multi_compile` 行可以视为用于控制单个着色器“功能”。请记住，着色器变体的总数会以这种方式急速增长。例如，十个各有两个选项的 `multi_compile` 功能总共生成 1024 个着色器变体。

## 关键字限制

When using Shader variants, there is a limit of 384 keywords in Unity, and Unity uses around 60 of them internally (therefore lowering the available limit). The keywords are enabled globally across a Unity project, so be careful not to exceed the limit when you define multiple keywords in several different Shaders.

### 本地关键字

The main disadvantage of **shader_feature** and **multi_compile** is that all keywords defined in them contribute towards Unity’s global keyword count limit (384 global keywords, plus 64 local keywords). To avoid this issue, you can use different shader variant directives: **shader_feature_local** and **multi_compile_local**.

- **shader_feature_local：__类似于** shader_feature__，但是枚举的关键字为本地关键字。
- **multi_compile_local：__类似于** multi_compile__，但是枚举的关键字为本地关键字。

本地指令将已定义的关键字保留在特定于该着色器的这些指令之下，而不是将这些关键字应用于整个项目。因此，应该使用本地关键字而不是全局关键字，除非计划通过全局 API 来启用这些特定关键字。

开始使用本地关键字时，您可能会发现性能有变化，但是此差异取决于项目的设置方式。每个着色器的本地和全局关键字总数会影响性能：在理想设置中，多用本地关键字和少用全局关键字可以减少每个着色器的关键字总数。

如果全局关键字和本地关键字同名，Unity 会优先考虑本地关键字。

#### 限制

- 不能将本地关键字与进行全局关键字更改的 API 一起使用（例如 Shader.EnableKeyword 或 CommandBuffer.EnableShaderKeyword）。
- 每个着色器最多有 64 个唯一性的本地关键字。
- 如果材质启用了本地关键字，并且其着色器变为不再声明的着色器，Unity 将创建新的全局关键字。

#### 示例

```
# pragma multi_compile_local __ FOO_ON
```

此指令生成两个着色器变体：一个未定义任何关键字 (`__`)，另一个定义了 `FOO_ON`（本地关键字）。

启用本地关键字的过程与启用全局关键字的过程相同：

```
public Material mat;
Private void Start()
{
    mat.EnableKeyword("FOO_ON");
}
```

## Stage-specific keyword directives

When you create shader variants, the Unity Editor’s default behavior is to generate every stage of the shader program in every variant. For example, if your shader program contains a vertex stage and a fragment stage, Unity generates a vertex stage and a fragment stage for every keyword combination.

If a keyword does not affect all stages, this default behavior results in redundant work. For example, if a keyword affects only the fragment stage, the Editor generates an identical vertex stage for each variant. Unity identifies and removes duplicates afterwards, so this redundant work does not affect build sizes or runtime performance; however, if you have a lot of stages and/or variants, the time wasted during shader compilation can be significant.

To avoid this problem, you can use stage-specific keyword directives. These are suffixes that you apply to regular keyword directives. They tell the Editor which shader stage a given keyword affects, so it can skip the redundant work when building shaders for supported graphics APIs.

### Supported graphics APIs

Unity does not fully support the use of stage-specific keyword directives with all graphics APIs.

- When compiling shaders for OpenGL and Vulkan, the Editor automatically reverts any stage-specific keyword directives to regular keyword directives.
- When compiling shaders for Metal, any keyword targeting vertex stages will also affect tessellation stages, and vice versa.

### Using stage-specific keyword directives

The available suffixes are `_vertex`, `_fragment`, `_hull`, `_domain`, `_geometry`, and `_raytracing`. You apply the suffix at the end of a keyword directive, for example: `multi_compile_fragment`, or `shader_feature_local_vertex`. To target multiple shader stages, you use multiple stage-specific keyword directives declaring the same keywords.

**Note:** you are responsible for ensuring that the keywords are only used in the specified shader stages.

## 内置 multi_compile 快捷方式

在内置管线中，有几个“快捷方式”可以用于常用变体组合。这些组合常用于处理不同的灯光、阴影和光照贴图类型。详情请参见[rendering paths and shaders](https://docs.unity3d.com/cn/2021.1/Manual/SL-RenderPipeline.html)。

- `multi_compile_fwdbase` 编译 [PassType.ForwardBase](https://docs.unity3d.com/cn/2021.1/ScriptReference/Rendering.PassType.ForwardBase.html) 所需的所有变体。这些变体处理不同的光照贴图类型以及启用或禁用的方向光主要阴影，它相当于这些变体的组合：
    - DIRECTIONAL
    - LIGHTMAP_ON
    - DIRLIGHTMAP_COMBINED
    - DYNAMICLIGHTMAP_ON
    - SHADOWS_SCREEN
    - SHADOWS_SHADOWMASK
    - LIGHTMAP_SHADOW_MIXING
    - LIGHTPROBE_SH
    - VERTEXLIGHT_ON
- `multi_compile_fwdadd` 编译 [PassType.ForwardAdd](https://docs.unity3d.com/cn/2021.1/ScriptReference/Rendering.PassType.ForwardAdd.html) 的变体。这将编译变体来处理方向光、聚光灯或点光源类型，以及它们带有剪影纹理的变体，它相当于这些变体的组合：
    - POINT
    - DIRECTIONAL
    - SPOT
    - POINT_COOKIE
    - DIRECTIONAL_COOKIE
- `multi_compile_fwdadd_fullshadows` - 与 `multi_compile_fwdadd` 相同，但还能够让光源具有实时阴影，它相当于==multi_compile_fwdadd==变体的组合加上：
    - SHADOWS_DEPTH
    - SHADOWS_SCREEN
    - SHADOWS_CUBE
    - SHADOWS_SOFT
    - SHADOWS_SHADOWMASK
    - LIGHTMAP_SHADOW_MIXING
- `multi_compile_fog` 扩展为多个变体以处理不同的雾效类型 (off/linear/exp/exp2)。

大多数内置快捷方式会产生许多着色器变体。如果知道项目不需要这些变体，可以使用 `#pragma skip_variants` 来跳过对其中一些变体的编译。例如：

```
# pragma multi_compile_fwdadd
# pragma skip_variants POINT POINT_COOKIE
```

该指令会跳过包含 `POINT` 或 `POINT_COOKIE` 的所有变体。

## 图形层和着色器变体

在运行时，Unity 会检查 GPU 的功能并确定其对应的[图形层](https://docs.unity3d.com/cn/2021.1/ScriptReference/Rendering.GraphicsTier.html)。在内置渲染管线中，可以为每个图形层自动创建一组着色器变体；为此，请使用 `#pragma hardware_tier_variants` 指令。

该功能仅与内置渲染管线兼容。它不兼容通用渲染管线 (URP)、高清渲染管线 (HDRP) 或自定义的可编程渲染管线。

To enable this feature, add `#pragma hardware_tier_variants renderer`, where `renderer` is a valid [graphics API](https://docs.unity3d.com/cn/2021.1/Manual/SL-ShaderCompilationAPIs.html), like this:

```
# pragma hardware_tier_variants gles3
```

除了其他所有关键字，Unity 还为每个着色器生成三个着色器变体。每个生成的变体都有以下定义命令之一，它们对应于 [GraphicsTier](https://docs.unity3d.com/cn/2021.1/ScriptReference/Rendering.GraphicsTier.html) 枚举的相同编号值：

```
UNITY_HARDWARE_TIER1
UNITY_HARDWARE_TIER2
UNITY_HARDWARE_TIER3
```

您可以使用它们为更低端或更高端硬件编写条件性回退或额外功能。

Unity 首次加载应用程序时，它会检测到 `GraphicsTier` 并将结果存储在 [Graphics.activeTier](https://docs.unity3d.com/cn/2021.1/ScriptReference/Graphics.activeTier.html) 中。要覆盖 `Graphics.activeTier` 的值，请直接设置该值。请注意，必须在 Unity 加载您要更改的任何着色器之前执行此操作。一个非常适合设置此值的位置是在加载主场景之前的预加载场景中。

为了帮助尽可能降低这些变体的影响，Unity 在播放器中只加载一组着色器。相同的着色器（例如，如果您只为 `TIER1` 编写了专用版本而其他所有版本都相同）将不占用磁盘上的任何额外空间。

要在 Unity Editor 中测试图形层，请导航至 **Edit > Graphics tier**，然后选择您希望 Unity Editor 使用的层。

请注意，图形层与[质量设置](https://docs.unity3d.com/cn/2021.1/Manual/class-QualitySettings.html)无关。它们是此设置的补充。

### 每平台着色器定义设置和图形层变体

在内置渲染管线中，可以使用 [EditorGraphicsSettings.SetShaderSettingsForPlatform](https://docs.unity3d.com/cn/2021.1/ScriptReference/EditorGraphicsSettings.SetTierSettings.html) API 针对给定的 [BuildTarget](https://docs.unity3d.com/cn/2021.1/ScriptReference/BuildTarget.html) 和 [GraphicsTier](https://docs.unity3d.com/cn/2021.1/ScriptReference/Rendering.GraphicsTier.html) 来覆盖 Unity 的内部 `#define`。

该功能仅与内置渲染管线兼容。它不兼容通用渲染管线 (URP)、高清渲染管线 (HDRP) 或自定义的可编程渲染管线。

请注意，如果您为给定的 `BuildTarget` 的不同 `GraphicsTier` 提供不同的 `TierSettings` 值，即使您未向着色器代码添加 `#pragma hardware_tier_variants`，Unity 也会为着色器生成层变体。

# 变量

Unity 的内置文件包含着色器的全局变量：当前对象的变换矩阵、光源参数、当前时间等等。就像任何其他变量一样，可在[着色器程序](https://docs.unity3d.com/cn/2021.1/Manual/SL-ShaderPrograms.html)中使用这些变量，但如果已经包含相关的 include 文件，则不必声明这些变量。

有关 include 文件更多信息，请参阅[内置 include 文件](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinIncludes.html)。

## 变换

所有这些矩阵都是 `float4x4` 类型，并且是列主序的。

| 名称                  | 值                 |
|---------------------|-------------------|
| UNITY_MATRIX_MVP    | 当前模型 * 视图 * 投影矩阵。 |
| UNITY_MATRIX_MV     | 当前模型 * 视图矩阵。      |
| UNITY_MATRIX_V      | 当前视图矩阵。           |
|UNITY_MATRIX_P| 当前投影矩阵。           |
| UNITY_MATRIX_VP     | 当前视图 * 投影矩阵。      |
| UNITY_MATRIX_T_MV   | 模型转置 * 视图矩阵。      |
| UNITY_MATRIX_IT_MV  | 模型逆转置 * 视图矩阵。     |
| unity_ObjectToWorld | 当前模型矩阵。           |
| unity_WorldToObject | 当前世界矩阵的逆矩阵。       |


## 摄像机和屏幕

这些变量将对应于正在渲染的[摄像机](https://docs.unity3d.com/cn/2021.1/Manual/class-Camera.html)。例如，在阴影贴图渲染中，它们仍将引用摄像机组件值，而不是用于阴影贴图投影的“虚拟摄像机”。

| 名称                       | 类型     | 值                              |
| ------------------------------ | -------- | ------------------------------------------------------------ |
|` _WorldSpaceCameraPos`           | float3   | 摄像机的世界空间位置。                                       |
| `_ProjectionParams`              | float4   | `x` 是 1.0（如果当前使用[翻转投影矩阵](https://docs.unity3d.com/cn/2021.1/Manual/SL-PlatformDifferences.html)进行渲染，则为 –1.0），`y` 是摄像机的近平面，`z` 是摄像机的远平面，`w` 是远平面的倒数。 |
| `_ScreenParams`                  | float4   | `x` 是摄像机目标纹理的宽度（以像素为单位），`y` 是摄像机目标纹理的高度（以像素为单位），`z` 是 1.0 + 1.0/宽度，`w` 为 1.0 + 1.0/高度。 |
| `_ZBufferParams`                 | float4   | 用于线性化 Z 缓冲区值。`x` 是 (1-远/近)，`y` 是 (远/近)，`z` 是 (x/远)，`w` 是 (y/远)。 |
| unity_OrthoParams              | float4   | `x` 是正交摄像机的宽度，`y` 是正交摄像机的高度，`z` 未使用，`w` 在摄像机为正交模式时是 1.0，而在摄像机为透视模式时是 0.0。 |
| unity_CameraProjection         | float4x4 | 摄像机的投影矩阵。                                           |
| unity_CameraInvProjection      | float4x4 | 摄像机投影矩阵的逆矩阵。                                     |
| unity_CameraWorldClipPlanes[6] | float4   | 摄像机视锥体平面世界空间方程，按以下顺序：左、右、底、顶、近、远。 |

## 时间

时间以秒为单位，并由项目 [Time 设置](https://docs.unity3d.com/cn/2021.1/Manual/class-TimeManager.html)中的时间乘数 (Time multiplier) 进行缩放。没有内置变量可用于访问未缩放的时间。


|**名称**|**类型**|**值**|
|:--|:--|:--|
|`_Time`|float4|自关卡加载以来的时间 (t/20, t, t_2, t_3)，用于将着色器中的内容动画化。|
|`_SinTime`|float4|时间正弦：(t/8, t/4, t/2, t)。|
|`_CosTime`|float4|时间余弦：(t/8, t/4, t/2, t)。|
|unity_DeltaTime|float4|增量时间：(dt, 1/dt, smoothDt, 1/smoothDt)。|

## 光照

光源参数以不同的方式传递给着色器，具体取决于使用哪个[渲染路径](https://docs.unity3d.com/cn/2021.1/Manual/RenderingPaths.html)， 以及着色器中使用哪种光源模式[通道标签](https://docs.unity3d.com/cn/2021.1/Manual/SL-PassTags.html)。

[前向渲染](https://docs.unity3d.com/cn/2021.1/Manual/RenderTech-ForwardRendering.html)（`ForwardBase` 和 `ForwardAdd` 通道类型）：


|**名称**|**类型**|**值**|
|:--|:--|:--|
|`_LightColor0`（在 UnityLightingCommon.cginc 中声明）|fixed4|光源颜色。|
|`_WorldSpaceLightPos0`|float4|方向光：（世界空间方向，0）。其他光源：（世界空间位置，1）。|
|unity_WorldToLight（在 AutoLight.cginc 中声明）|float4x4|世界/光源矩阵。用于对剪影和衰减纹理进行采样。|
|unity_4LightPosX0、unity_4LightPosY0、unity_4LightPosZ0|float4|（仅限 ForwardBase 通道）前四个非重要点光源的世界空间位置。|
|unity_4LightAtten0| float4      | （仅限 ForwardBase 通道）前四个非重要点光源的衰减因子。    |
|unity_LightColor|half4[4]| （仅限 ForwardBase 通道）前四个非重要点光源的颜色。        |
|unity_WorldToShadow| float4x4[4] | 世界/阴影矩阵。聚光灯的一个矩阵，方向光级联最多有四个矩阵。  |



延迟着色和延迟光照，在光照通道着色器中使用（全部在 UnityDeferredLibrary.cginc 中声明）：

|**名称**|**类型**|**值**|
|:--|:--|:--|
|`_LightColor`|float4|光源颜色。|
|unity_WorldToLight|float4x4|世界/光源矩阵。用于对剪影和衰减纹理进行采样。|
|unity_WorldToShadow|float4x4[4]|世界/阴影矩阵。聚光灯的一个矩阵，方向光级联最多有四个矩阵。|

为 `ForwardBase`、`PrePassFinal` 和 `Deferred` 通道类型设置了球谐函数系数 （由环境光和光照探针使用）。这些系数包含由世界空间法线求值的三阶 SH 函数（请参阅 [UnityCG.cginc](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinIncludes.html) 中的 `ShadeSH9`）。 这些变量都是 half4 类型、`unity_SHAr` 和类似名称。

[顶点光照渲染](https://docs.unity3d.com/cn/2021.1/Manual/RenderTech-VertexLit.html)（`Vertex` 通道类型）：

最多可为 `Vertex` 通道类型设置 8 个光源；始终从最亮的光源开始排序。因此，如果您希望 一次渲染受两个光源影响的对象，可直接采用数组中前两个条目。如果影响对象 的光源数量少于 8，则其余光源的颜色将设置为黑色。


|**名称**|**类型**|**值**|
|:--|:--|:--|
|unity_LightColor|half4[8]|光源颜色。|
|unity_LightPosition|float4[8]|视图空间光源位置。方向光为 (-direction,0)；点光源/聚光灯为 (position,1)。|
|unity_LightAtten|half4[8]|光源衰减因子。_x_ 是 cos(spotAngle/2) 或 –1（非聚光灯）；_y_ 是1/cos(spotAngle/4) 或 1（非聚光灯）；_z_ 是二次衰减；_w_ 是平方光源范围。|
|unity_SpotDirection|float4[8]|视图空间聚光灯位置；非聚光灯为 (0,0,1,0)。|

## 光照贴图

|**名称**|**类型**|**值**|
|:--|:--|:--|
|unity_Lightmap|Texture2D|包含光照贴图信息。|
|unity_LightmapST|float4[8]|缩放 UV 信息并转换到正确的范围以对光照贴图纹理进行采样。|

## 雾效和环境光

|**名称**|**类型**|**值**|
|:--|:--|:--|
|unity_AmbientSky|fixed4|梯度环境光照情况下的天空环境光照颜色。|
|unity_AmbientEquator|fixed4|梯度环境光照情况下的赤道环境光照颜色。|
|unity_AmbientGround|fixed4|梯度环境光照情况下的地面环境光照颜色。|
|UNITY_LIGHTMODEL_AMBIENT|fixed4|环境光照颜色（梯度环境情况下的天空颜色）。旧版变量。|
|unity_FogColor|fixed4|雾效颜色。|
|unity_FogParams|float4|用于雾效计算的参数：(density / sqrt(ln(2))、density / ln(2)、–1/(end-start) 和 end/(end-start))。_x_ 对于 Exp2 雾模式很有用；_y_ 对于 Exp 模式很有用，_z_ 和 _w_ 对于 Linear 模式很有用。|

## 其他


|**名称**|**类型**|**值**|
|:--|:--|:--|
|unity_LODFade|float4|使用 [LODGroup](https://docs.unity3d.com/cn/2021.1/Manual/class-LODGroup.html) 时的细节级别淡入淡出。_x_ 为淡入淡出（0 到 1），_y_ 为量化为 16 级的淡入淡出，_z_ 和 _w_ 未使用。|
|`_TextureSampleAdd`|float4|根据所使用的纹理是 Alpha8 格式（值设置为 (1,1,1,0)）还是不是该格式（值设置为 (0,0,0,0)）由 Unity **仅针对 UI** 自动设置。|

# 预定义的着色器预处理器宏

Unity 在编译[着色器程序](https://docs.unity3d.com/cn/2021.1/Manual/SL-ShaderPrograms.html)时会定义几个预处理器宏。

## 目标平台

|**宏：**|**目标平台：**|
|:--|:--|
|`SHADER_API_D3D11`|Direct3D 11|
|`SHADER_API_GLCORE`|桌面端 OpenGL“核心”(GL 3/4)|
|`SHADER_API_GLES`|OpenGL ES 2.0|
|`SHADER_API_GLES3`|OpenGL ES 3.0/3.1|
|`SHADER_API_METAL`|iOS/Mac Metal|
|`SHADER_API_VULKAN`|Vulkan|
|`SHADER_API_D3D11_9X`|适用于通用 Windows 平台的 Direct3D 11“功能级别 9.x”目标|
|`SHADER_API_PS4`|PlayStation 4。也定义了 `SHADER_API_PSSL`。|
|`SHADER_API_XBOXONE`|Xbox One|

`SHADER_API_MOBILE` 是针对所有常规移动平台（GLES、GLES3、METAL）定义的。

此外，当目标着色语言为 GLSL 时，还会定义 `SHADER_TARGET_GLSL`（对于 OpenGL/GLES 平台来说始终会定义）。

## 着色器目标模型

`SHADER_TARGET` 被定义为与着色器目标编译模型匹配的数值（即匹配 `#pragma target` 指令）。例如，当编译到着色器模型 3.0 时，`SHADER_TARGET` 为 `30`。您可以在着色器代码中使用此宏来进行条件检查。例如：

```
# if SHADER_TARGET < 30
    // 低于着色器模型 3.0：
    // 着色器功能非常有限，执行近似操作
# else
    // 不错的功能，执行更高级的任务
# endif
```

## Unity 版本

`UNITY_VERSION` 包含 Unity 版本的数值。例如，对于 Unity 5.0.1，`UNITY_VERSION` 为 `501`。如果您需要编写使用不同着色器内置功能的着色器，则可以将其用于版本比较。例如，`#if UNITY_VERSION >= 500` 预处理器检查仅在版本为 5.0.0 或更高时可以通过。

## 编译的着色器阶段

编译每个着色器阶段时会定义预处理器宏 `SHADER_STAGE_VERTEX`、`SHADER_STAGE_FRAGMENT`、`SHADER_STAGE_DOMAIN`、`SHADER_STAGE_HULL`、`SHADER_STAGE_GEOMETRY` 或 `SHADER_STAGE_COMPUTE`。通常，在像素着色器和计算着色器之间共享着色器代码时，这些宏非常有用，可以解决某些工作必须以略有不同的方式来完成的情况。

## 平台差异 helper

不鼓励直接使用这些平台宏，因为它们并非始终有助于代码的未来验证。例如，如果您正在编写一个检查 D3D11 的着色器，您可能希望确保在将来将这项检查扩展为包含 Vulkan。应改用 Unity 定义的几个 helper 宏（在 [`HLSLSupport.cginc`](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinIncludes.html) 中）：

|宏|用途：**     |
|:--|:--|
|`UNITY_BRANCH` |在条件语句之前添加此宏，告知编译器应将其编译为实际分支。在 HLSL 平台上扩展为 `[branch]`。|
|`UNITY_FLATTEN`|在条件语句之前添加此宏，告知编译器应该将其展平以避免实际的分支指令。在 HLSL 平台上扩展为 `[flatten]`。|
|`UNITY_NO_SCREENSPACE_SHADOWS`|在不使用级联屏幕空间阴影贴图的平台（移动平台）上定义。|
|`UNITY_NO_LINEAR_COLORSPACE`|在不支持线性颜色空间的平台（移动平台）上定义。|
|`UNITY_NO_RGBM`|在不使用光照贴图 RGBM 压缩的平台（移动平台）上定义。|
|`UNITY_NO_DXT5nm`|在不使用 DXT5nm 法线贴图压缩的平台（移动平台）上定义。|
|`UNITY_FRAMEBUFFER_FETCH_AVAILABLE`|在可使用“帧缓冲颜色提取”功能的平台（通常为 iOS 平台 - OpenGL ES 2.0、3.0 和 Metal）上定义。|
|`UNITY_USE_RGBA_FOR_POINT_SHADOWS`|在点光源阴影贴图使用具有编码深度的 RGBA 纹理的平台（其他平台使用单通道浮点纹理）上定义。|
|`UNITY_ATTEN_CHANNEL`|定义光源衰减纹理的哪个通道包含数据；用于每像素光照代码。定义为“r”或“a”。|
|`UNITY_HALF_TEXEL_OFFSET`|在将纹理像素映射到像素时需要进行半纹素偏移调整的平台（例如 Direct3D 9）上定义。|
|`UNITY_UV_STARTS_AT_TOP`|始终定义值为 1 或 0。值为 1 表示在平台上的纹理之上的纹理 V 坐标为 0。Direct3D 类平台使用值 1；OpenGL 类平台使用值 0。|
|`UNITY_MIGHT_NOT_HAVE_DEPTH_Texture`|如果平台可以通过手动将深度渲染到纹理中来模拟阴影贴图或深度纹理，则定义此宏。|
|`UNITY_PROJ_COORD(a)`|给定一个 4 分量矢量，此宏返回一个适合投影纹理读取的纹理坐标。在大多数平台上，它直接返回给定值。|
|`UNITY_NEAR_CLIP_VALUE`|定义为近裁剪面的值。Direct3D 类平台使用 0.0，而 OpenGL 类平台使用 –1.0。|
|`UNITY_VPOS_TYPE` |定义像素位置输入 (VPOS) 所需的数据类型：D3D9 上为 `float2`，其他为 `float4`。|
|`UNITY_CAN_COMPILE_TESSELLATION`|在着色器编译器“理解”曲面细分着色器 HLSL 语法时定义（当前仅限 D3D11）。|
|`UNITY_INITIALIZE_OUTPUT(type,name)`|将给定_类型_的变量_名称_初始化为零。|
|`UNITY_COMPILER_HLSL`, `UNITY_COMPILER_HLSL2GLSL`, `UNITY_COMPILER_CG` |Indicates which Shader compiler is being used to compile Shaders. See documentation on [Shader compilation](https://docs.unity3d.com/cn/2021.1/Manual/shaders-compilation.html) for more details. Use this if you run into very specific Shader syntax handling differences between the compilers, and want to write different code for each compiler. |

- `UNITY_REVERSED_Z` - 在使用反转 Z 缓冲区的平台上定义。存储的 Z 值的范围是 1 到 0，而不是 0 到 1。

## 阴影贴图宏

根据平台的不同，声明和采样阴影贴图可能会有很大差异。Unity 有几个宏可帮助解决这个问题：

|**宏：**|**用途：**|
|:--|:--|
|`UNITY_DECLARE_SHADOWMAP(tex)`|声明一个名为“tex”的阴影贴图纹理变量。|
|`UNITY_SAMPLE_SHADOW(tex,uv)`|在给定的“uv”坐标处采样阴影贴图纹理“tex”（XY 分量是纹理位置，Z 分量是要比较的深度）。返回单个浮点值，阴影项的范围在 0 到 1 之间。|
|`UNITY_SAMPLE_SHADOW_PROJ(tex,uv)`|与上面类似，但是会读取投影阴影贴图。“uv”是一个 float4，所有其他分量除以 .w 来执行查找。|

__注意：__并非所有显卡都支持阴影贴图。请使用 [SystemInfo.SupportsRenderTextureFormat](https://docs.unity3d.com/cn/2021.1/ScriptReference/SystemInfo.SupportsRenderTextureFormat.html) 检查是否支持。

## 常量缓冲区宏

Direct3D 11 将所有着色器变量分组为“常量缓冲区”。Unity 的大多数内置变量已经分组，但对于您自己的着色器中的变量，更加理想的做法是，根据预期的更新频率将它们放入单独的常量缓冲区。

对此，请使用 `CBUFFER_START(name)` 和 `CBUFFER_END` 宏：

```
CBUFFER_START(MyRarelyUpdatedVariables)
    float4 _SomeGlobalValue;
CBUFFER_END
```

## 纹理/采样器声明宏

通常，在着色器代码中使用 `texture2D` 来声明纹理和采样器对。 但是在某些平台（例如 DX11）上，纹理和采样器是单独的对象， 并且可能的采样器最大数量非常有限。Unity 有一些宏来声明 没有采样器的纹理，并使用另一个纹理中的采样器对纹理进行采样。 如果您遇到采样器限制，并且知道几个纹理实际上可以共享同一个采样器 （采样器定义纹理过滤和包裹模式），请使用这些宏。

|**宏：**|**用途：**|
|:--|:--|
|`UNITY_DECLARE_TEX2D(name)`|声明纹理和采样器对。|
|`UNITY_DECLARE_TEX2D_NOSAMPLER(name)`|声明不含采样器的纹理。|
|`UNITY_DECLARE_TEX2DARRAY(name)`|声明纹理数组采样器变量。|
|`UNITY_SAMPLE_TEX2D(name,uv)`|使用给定的纹理坐标从纹理和采样器对中采样。|
|`UNITY_SAMPLE_TEX2D_SAMPLER( name,samplername,uv)`|使用另一个纹理中的采样器 (samplername)，从纹理 (name) 中采样。|
|`UNITY_SAMPLE_TEX2DARRAY(name,uv)`|从具有 float3 UV 的纹理数组中采样；坐标的 z 分量是数组元素索引。|
|`UNITY_SAMPLE_TEX2DARRAY_LOD(name,uv,lod)`|从具有显式 Mipmap 级别的纹理数组中采样。|

有关更多信息，请参阅[采样器状态](https://docs.unity3d.com/cn/2021.1/Manual/SL-SamplerStates.html)文档。

## 表面着色器通道指示符

编译[表面着色器](https://docs.unity3d.com/cn/2021.1/Manual/SL-SurfaceShaders.html)时，表面着色器会为各种通道生成大量代码以产生光照。编译每个通道时，将定义以下宏之一：

|**宏：**|**用途：**|
|:--|:--|
|`UNITY_PASS_FORWARDBASE`|前向渲染基础通道（主方向光、光照贴图和 SH）。|
|`UNITY_PASS_FORWARDADD`|前向渲染附加通道（每个通道一个光源）。|
|`UNITY_PASS_DEFERRED`|延迟着色通道（渲染 G 缓冲区）。|
|`UNITY_PASS_SHADOWCASTER`|阴影投射物和深度纹理渲染通道。|
|`UNITY_PASS_PREPASSBASE`|旧版延迟光照基础通道（渲染法线和镜面反射指数）。|
|`UNITY_PASS_PREPASSFINAL`|旧版延迟光照最终通道（应用光照和纹理）。|

## 禁用自动升级

`UNITY_SHADER_NO_UPGRADE` 允许您禁止 Unity 自动升级或修改着色器文件。

# 平台特定的渲染差异

Unity 在如下各种图形库平台上运行：[Open GL](https://www.opengl.org/)、[Direct3D](https://msdn.microsoft.com/en-us/library/windows/desktop/hh309466.aspx)、[Metal](https://developer.apple.com/metal/) 和游戏主机。在某些情况下，平台与着色器语言语义之间的图形渲染行为方式存在差异。大多数情况下，Unity Editor 会隐藏这些差异，但在某些情况下，Editor 无法为您执行此操作。在这些情况下，您需要确保消除平台之间的差异。下面列出了这些情况以及发生这些情况时需要采取的操作。

## 渲染纹理坐标

垂直纹理坐标约定在两种类型的平台之间有所不同，分别是 Direct3D 类和 OpenGL 类平台。

- **Direct3D 类**：顶部坐标为 0 并向下增加。此类型适用于 Direct3D、Metal 和游戏主机。
- **OpenGL 类**：底部坐标为 0 并向上增加。此类适用于 OpenGL 和 OpenGL ES。

除了渲染到[渲染纹理](https://docs.unity3d.com/cn/2021.1/Manual/class-RenderTexture.html)的情况下，这种差异不会对您的项目产生任何影响。在 Direct3D 类平台上渲染到纹理时，Unity 会在内部上下翻转渲染。这样就会使坐标约定在平台之间匹配，并以 OpenGL 类平台约定作为标准。

在着色器中，有两种常见情况需要您采取操作确保不同的坐标约定不会在项目中产生问题，这两种情况就是图像效果和 UV 空间中的渲染。

### 图像效果

使用[图像效果](https://docs.unity3d.com/Packages/com.unity.postprocessing@latest)和抗锯齿时，系统不会翻转为图像效果生成的源纹理来匹配 OpenGL 类平台约定。在这种情况下，Unity 渲染到屏幕以获得抗锯齿效果，然后将渲染解析为渲染纹理，以便通过图像效果进行进一步处理。

如果您的图像效果是一次处理一个渲染纹理的简单图像效果，则 [Graphics.Blit](https://docs.unity3d.com/cn/2021.1/ScriptReference/Graphics.Blit.html) 会处理不一致的坐标。但是，如果您在[图像效果](https://docs.unity3d.com/Packages/com.unity.postprocessing@latest)中一起处理多个[渲染纹理](https://docs.unity3d.com/cn/2021.1/Manual/class-RenderTexture.html)，则在 Direct3D 类平台中以及在您使用抗锯齿时，渲染纹理很可能以不同的垂直方向出现。要标准化坐标，必须在顶点着色器中手动上下“翻转”屏幕纹理，使其与 OpenGL 类坐标标准匹配。

以下代码示例演示了如何执行此操作：

```
// 翻转纹理的采样：
// 主纹理的
// 纹理像素大小将具有负 Y。

# if UNITY_UV_STARTS_AT_TOP
if (_MainTex_TexelSize.y < 0)
        uv.y = 1-uv.y;
# endif
```

[GrabPass](https://docs.unity3d.com/cn/2021.1/Manual/SL-GrabPass.html) 也出现了类似的情况。生成的渲染纹理实际上可能不会在 Direct3D 类（非 OpenGL 类）平台上进行上下翻转。如果着色器代码对 GrabPass 纹理进行采样，请使用 [UnityCG include](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinFunctions.html) 文件中的 `ComputeGrabScreenPos` 函数。

### 在 UV 空间中渲染

在纹理坐标 (UV) 空间中渲染特殊效果或工具时，您可能需要调整着色器，以便在 Direct3D 类和 OpenGL 类系统之间进行一致渲染。您还可能需要在渲染到屏幕和渲染到纹理之间进行渲染调整。为进行此类调整，应上下翻转 Direct3D 类投影，使其坐标与 OpenGL 类投影坐标相匹配。

[内置变量](https://docs.unity3d.com/cn/2021.1/Manual/SL-UnityShaderVariables.html) `ProjectionParams.x` 包含值 `+1` 或 `–1`。`-1` 表示投影已上下翻转以匹配 OpenGL 类投影坐标，而 `+1` 表示尚未翻转。 您可以在着色器中检查此值，然后执行不同的操作。下面的示例将检查是否已翻转投影，如果已翻转，则再次进行翻转，然后返回 UV 坐标以便匹配。

```
float4 vert(float2 uv : TEXCOORD0) : SV_POSITION
{
    float4 pos;
    pos.xy = uv;
    // 此示例使用上下翻转的投影进行渲染，
    // 因此也翻转垂直 UV 坐标
    if (_ProjectionParams.x < 0)
        pos.y = 1 - pos.y;
    pos.z = 0;
    pos.w = 1;
    return pos;
}
```

## 裁剪空间坐标

与纹理坐标类似，裁剪空间坐标（也称为投影后空间坐标）在 Direct3D 类和 OpenGL 类平台之间有所不同：

- **Direct3D 类**：裁剪空间深度从近平面的 +1.0 到远平面的 0.0。此类型适用于 Direct3D、Metal 和游戏主机。
- **OpenGL 类**：裁剪空间深度从近平面的 –1.0 到远平面的 +1.0。此类适用于 OpenGL 和 OpenGL ES。

在着色器代码内，可使用[内置宏](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinMacros.html) `UNITY_NEAR_CLIP_VALUE` 来获取基于平台的近平面值。

在脚本代码内，使用 [GL.GetGPUProjectionMatrix](https://docs.unity3d.com/cn/2021.1/ScriptReference/GL.GetGPUProjectionMatrix.html) 将 Unity 的坐标系（遵循 OpenGL 类约定）转换为 Direct3D 类坐标（如果这是平台所期望的）。

## 着色器计算的精度

要避免精度问题，请确保在目标平台上测试着色器。移动设备和 PC 中的 GPU 在处理浮点类型方面有所不同。PC GPU 将所有浮点类型（浮点精度、半精度和固定精度）视为相同；PC GPU 使用完整 32 位精度进行所有计算，而许多移动设备 GPU 并不是这样做。

有关详细信息，请参阅[数据类型和精度](https://docs.unity3d.com/cn/2021.1/Manual/SL-DataTypesAndPrecision.html)的文档。

## 着色器中的 const 声明

`const` 的使用在 Microsoft HSL（请参阅 [msdn.microsoft.com](http://msdn.microsoft.com/)）和 OpenGL 的 GLSL（请参阅 [Wikipedia](https://en.wikipedia.org/wiki/OpenGL_Shading_Language)）着色器语言之间有所不同。

- Microsoft 的 HLSL `const` 与 C# 和 C++ 中的含义大致相同：声明的变量在其作用域内是只读的，但可按任何方式初始化。
- OpenGL 的 GLSL `const` 表示变量实际上是编译时常量，因此必须使用编译时约束（文字值或其他对于 `const` 的计算）进行初始化。

最好是遵循 OpenGL 的 GLSL 语义，并且只有当变量真正不变时才将变量声明为 `const`。避免使用其他一些可变值初始化 `const` 变量（例如，作为函数中的局部变量）。这一原则也适用于 Microsoft 的 HLSL，因此以这种方式使用 `const` 可以避免在某些平台上混淆错误。

## 着色器使用的语义

要让着色器在所有平台上运行，一些着色器值应该使用以下语义：

- __顶点着色器输出（裁剪空间）位置__：`SV_POSITION`。有时，着色器使用 POSITION 语义来使着色器在所有平台上运行。请注意，这不适用于 Sony PS4 或有曲面细分的情况。
- __片元着色器输出颜色__：`SV_Target`。有时，着色器使用 `COLOR` 或 `COLOR0` 来使着色器在所有平台上运行。请注意，这不适用于 Sony PS4。

将网格渲染为点时，从顶点着色器输出 `PSIZE` 语义（例如，将其设置为 1）。某些平台（如 OpenGL ES 或 Metal）在未从着色器写入点大小时会将点大小视为“未定义”。

有关更多详细信息，请参阅有关[着色器语义](https://docs.unity3d.com/cn/2021.1/Manual/SL-ShaderSemantics.html)的文档。

## Direct3D 着色器编译器语法

Direct3D platforms use Microsoft’s [HLSL Shader compiler](https://docs.unity3d.com/cn/2021.1/Manual/shaders-compilation.html). The HLSL compiler is stricter than other compilers about various subtle Shader errors. For example, it doesn’t accept function output values that aren’t initialized properly.

使用此编译器时，您可能遇到的最常见情况是：

- 具有 `out` 参数的[表面着色器](https://docs.unity3d.com/cn/2021.1/Manual/SL-SurfaceShaders.html)顶点修改器。按如下方式初始化输出：

```
  void vert (inout appdata_full v, out Input o) 
      {
        **UNITY_INITIALIZE_OUTPUT(Input,o);**
        // ...
      }
```

- 部分初始化的值。例如，函数返回 `float4`，但代码只设置它的 `.xyz` 值。如果只需要三个值，请设置所有值或更改为 `float3`。
- 在顶点着色器中使用 `tex2D`。这是无效的，因为顶点着色器中不存在 UV 导数。这种情况下，您需要采样显式 Mip 级别；例如，使用 `tex2Dlod` (`tex, float4(uv,0,0)`)。此外，还需要添加 `#pragma target 3.0`，因为 `tex2Dlod` 是着色器模型 3.0 的功能。

## 着色器中的 DirectX 11 (DX11) HLSL 语法

[表面着色器](https://docs.unity3d.com/cn/2021.1/Manual/SL-SurfaceShaders.html)编译管线的某些部分不能理解特定于 DirectX 11 的 HLSL（Microsoft 的着色器语言）语法。

如果您正在使用 HLSL 功能（比如 `StructuredBuffers`、`RWTextures` 和其他非 DirectX 9 语法），请将它们包裹在 DirectX X11 专用的预处理器宏中，如下例所示。

```
# ifdef SHADER_API_D3D11
// DirectX11 专用代码，例如
StructuredBuffer<float4> myColors;
RWTexture2D<float4> myRandomWriteTexture;
# endif
```

## 使用着色器帧缓冲提取

一些 GPU（最明显的是 iOS 上基于 PowerVR 的 GPU）允许您通过提供当前片元颜色作为片元着色器的输入来进行某种可编程混合（请参阅 [khronos.org](https://www.khronos.org/registry/gles/extensions/EXT/EXT_shader_framebuffer_fetch.txt) 上的 `EXT_shader_framebuffer_fetch`）。

可在 Unity 中编写使用帧缓冲提取功能的着色器。要执行此操作，请在使用 HLSL（Microsoft 的着色语言，请参阅 [msdn.microsoft.com](http://msdn.microsoft.com/)）或 Cg（Nvidia 的着色语言，请参阅 [nvidia.co.uk](http://www.nvidia.co.uk/)）编写片元着色器时使用 `inout` 颜色参数。

以下示例采用的是 Cg 语言。

```
CGPROGRAM
// 只为可能支持该功能的平台（目前是 gles、gles3 和 metal）
// 编译着色器
# pragma only_renderers framebufferfetch

void frag (v2f i, inout half4 ocol : SV_Target)
{
    // ocol 可以被读取（当前帧缓冲区颜色）
    // 并且可以被写入（将颜色更改为该颜色）
    // ...
}   
ENDCG
```

## 着色器中的深度 (Z) 方向

深度 (Z) 方向在不同的着色器平台上不同。

**DirectX 11、DirectX 12、PS4、Xbox One、Metal：反转方向**

- 深度 (Z) 缓冲区在近平面处为 1.0，在远平面处减小到 0.0。
- 裁剪空间范围是 [near,0]（表示近平面处的近平面距离，在远平面处减小到 0.0）。

**其他平台：传统方向**

- 深度 (Z) 缓冲区值在近平面处为 0.0，在远平面处为 1.0。
- 裁剪空间取决于具体平台：
    - 在 Direct3D 类平台上，范围是 [0,far]（表示在近平面处为 0.0，在远平面处增加到远平面距离）。
    - 在 OpenGL 类平台上，范围是 [-near,far]（表示在近平面处为负的近平面距离，在远平面处增加到远平面距离）。

请注意，使反转方向深度 (Z) 与浮点深度缓冲区相结合，可显著提高相对于传统方向的深度缓冲区精度。这样做的优点是降低 Z 坐标的冲突并改善阴影，特别是在使用小的近平面和大的远平面时。

因此，在使用深度 (Z) 发生反转的平台上的着色器时：

- 定义了 UNITY_REVERSED_Z。
- `_CameraDepth` 纹理的纹理范围是 1（近平面）到 0（远平面）。
- 裁剪空间范围是“near”（近平面）到 0（远平面）。

但是，以下宏和函数会自动计算出深度 (Z) 方向的任何差异：

- `Linear01Depth(float z)`
- `LinearEyeDepth(float z)`
- UNITY_CALC_FOG_FACTOR(coord)

### 提取深度缓冲区

如果要手动提取深度 (Z) 缓冲区值，则可能需要检查缓冲区方向。以下是执行此操作的示例：

```
float z = tex2D(_CameraDepthTexture, uv);
# if defined(UNITY_REVERSED_Z)
    z = 1.0f - z;
# endif
```

### 使用裁剪空间

如果要手动使用裁剪空间 (Z) 深度，则可能还需要使用以下宏来抽象化平台差异：

```
float clipSpaceRange01 = UNITY_Z_0_FAR_FROM_CLIPSPACE(rawClipSpace);
```

**注意**：此宏不会改变 OpenGL 或 OpenGL ES 平台上的裁剪空间，因此在这些平台上，此宏返回“-near”1（近平面）到 far（远平面）之间的值。

### 投影矩阵

如果处于深度 (Z) 发生反转的平台上，则 [GL.GetGPUProjectionMatrix()](https://docs.unity3d.com/cn/2021.1/ScriptReference/GL.GetGPUProjectionMatrix.html) 返回一个还原了 z 的矩阵。 但是，如果要手动从投影矩阵中进行合成（例如，对于自定义阴影或深度渲染），您需要通过脚本按需自行还原深度 (Z) 方向。

以下是执行此操作的示例：

```
var shadowProjection = Matrix4x4.Ortho(...); //阴影摄像机投影矩阵
var shadowViewMat = ...     //阴影摄像机视图矩阵
var shadowSpaceMatrix = ... //从裁剪空间到阴影贴图纹理空间
    
//当引擎通过摄像机投影计算设备投影矩阵时，
//"m_shadowCamera.projectionMatrix"被隐式反转
m_shadowCamera.projectionMatrix = shadowProjection; 

//"shadowProjection"在连接到"m_shadowMatrix"之前被手动翻转，
//因为它被视为着色器的其他矩阵。
if(SystemInfo.usesReversedZBuffer) 
{
    shadowProjection[2, 0] = -shadowProjection[2, 0];
    shadowProjection[2, 1] = -shadowProjection[2, 1];
    shadowProjection[2, 2] = -shadowProjection[2, 2];
    shadowProjection[2, 3] = -shadowProjection[2, 3];
}
    m_shadowMatrix = shadowSpaceMatrix * shadowProjection * shadowViewMat;
```

### 深度 (Z) 偏差

Unity 自动处理深度 (Z) 偏差，以确保其与 Unity 的深度 (Z) 方向匹配。但是，如果要使用本机代码渲染插件，则需要在 C 或 C++ 代码中消除（反转）深度 (Z) 偏差。

#### 深度 (Z) 方向检查工具

- 使用 [SystemInfo.usesReversedZBuffer](https://docs.unity3d.com/cn/2021.1/ScriptReference/SystemInfo-usesReversedZBuffer.html) 可确认所在平台是否使用反转深度 (Z)。

# 数据类型和精度

Unity 中的标准着色器语言为 [HLSL](https://docs.unity3d.com/cn/2021.1/Manual/SL-ShadingLanguage.html)，支持一般 HLSL 数据类型。但是，Unity 对 HLSL 类型有一些补充，特别是为了在移动平台上提供更好的支持。

## 基本数据类型

着色器中的大多数计算是对浮点数（在 C# 等常规编程语言中为 `float`）进行的。浮点类型有几种变体：`float`、`half` 和 `fixed`（以及它们的矢量/矩阵变体，比如 `half3` 和 `float4x4`）。这些类型的精度不同（因此性能或功耗也不同）：

#### 高精度：`float`

最高精度浮点值；一般是 32 位（就像常规编程语言中的 `float`）。

完整的 `float` 精度通常用于世界空间位置、纹理坐标或涉及复杂函数（如三角函数或幂/取幂）的标量计算。

#### 中等精度：`half`

中等精度浮点值；通常为 16 位（范围为 –60000 至 +60000，精度约为 3 位小数）。

半精度对于短矢量、方向、对象空间位置、高动态范围颜色非常有用。

#### 低精度：`fixed`

最低精度的定点值。通常是 11 位，范围从 –2.0 到 +2.0，精度为 1/256。

固定精度对于常规颜色（通常存储在常规纹理中）以及对它们执行简单运算非常有用。

#### 整数数据类型

整数（`int` 数据类型）通常用作循环计数器或数组索引。为此，它们通常可以在各种平台上正常工作。

根据平台的不同，GPU 可能不支持整数类型。例如，Direct3D 9 和 OpenGL ES 2.0 GPU 仅对浮点数据进行运算，并且可以使用相当复杂的浮点数学指令来模拟简单的整数表达式（涉及位运算或逻辑运算）。

Direct3D 11、OpenGL ES 3、Metal 和其他现代平台都对整数数据类型有适当的支持，因此使用位移位和位屏蔽可以按预期工作。

## 复合矢量/矩阵类型

HLSL 具有从基本类型创建的内置矢量和矩阵类型。例如，`float3` 是一个 3D 矢量，具有分量 .x、.y 和 .z，而 `half4` 是一个中等精度 4D 矢量，具有分量 .x、.y、.z 和 .w。或者，可使用 .r、.g、.b 和 .a 分量来对矢量编制索引，这在处理颜色时很有用。

矩阵类型以类似的方式构建；例如 `float4x4` 是一个 4x4 变换矩阵。请注意，某些平台仅支持方形矩阵，最主要的是 OpenGL ES 2.0。

## 纹理/采样器类型

通常按照如下方式在 HLSL 代码中声明纹理：

```
sampler2D _MainTex;
samplerCUBE _Cubemap;
```

对于移动平台，这些将转换为“低精度采样器”，即预期纹理应具有低精度数据。如果您知道纹理包含 HDR 颜色，则可能需要使用半精度采样器：

```
sampler2D_half _MainTex;
samplerCUBE_half _Cubemap;
```

或者，如果纹理包含完整浮点精度数据（例如[深度纹理](https://docs.unity3d.com/cn/2021.1/Manual/SL-DepthTextures.html)），请使用完整精度采样器：

```
sampler2D_float _MainTex;
samplerCUBE_float _Cubemap;
```

## 精度、硬件支持和性能

使用 `float`/`half`/`fixed` 数据类型的一个难题是：PC GPU **始终**为高精度。也就是说，对于所有 PC (Windows/Mac/Linux) GPU，在着色器中编写 `float`、`half` 还是 `fixed` 数据类型都无关紧要。这些 GPU 将始终以 32 位浮点精度来计算所有数据。

仅当目标平台是移动端 GPU 时，`half` 和 `fixed` 类型才变得重要，在这种情况下，这些类型主要面临功耗（有时候是性能）约束。请记住，要确认是否遇到精度/数值问题，必须在移动设备上测试着色器。

即使在移动端 GPU 上，不同的精度支持也会因 GPU 产品系列而异。下面概述了个每个移动端 GPU 产品系列如何处理每个浮点类型（以用于该产品系列的位数来表示）：

|GPU 产品系列|浮点精度|半精度|固定精度|
|:--|:--|:--|:--|
|PowerVR 系列 6/7|32|16||
|PowerVR SGX 5xx|32|16|11|
|Qualcomm Adreno 4xx/3xx|32|16||
|Qualcomm Adreno 2xx|32 顶点，24 片元|||
|ARM Mali T6xx/7xx|32|16||
|ARM Mali 400/450|32 顶点，16 片元|||
|NVIDIA X1|32|16||
|NVIDIA K1|32|||
|NVIDIA Tegra 3/4|32|16||

大多数现代移动端 GPU 实际上只支持 32 位数字（用于 `float` 类型）或 16 位数字（用于 `half` 和 `fixed` 类型）。一些较旧的 GPU 对顶点着色器和片元着色器计算具有不同的精度。

使用较低的精度通常可以更快，这可能是由于改进的 GPU 寄存器分配，或是由于某些低精度数学运算的特殊“快速路径”执行单元。即使没有原始性能优势，使用较低的精度通常也会降低 GPU 的功耗，从而延长电池续航时间。

一般的经验法则是全部都从半精度开始（但位置和纹理坐标除外）。仅当半精度对于计算的某些部分不足时，才增加精度。

#### 支持无穷大、非数字和其他特殊浮点值

对特殊浮点值的支持可能会有所不同，具体取决于运行的 GPU 产品系列（主要是移动端）。

支持 Direct3D 10 的所有 PC GPU 都支持非常明确的 IEEE 754 浮点标准。这意味着，在 CPU 上，浮点数的行为与常规编程语言完全相同。

移动端 GPU 的支持程度可能稍有不同。在某些移动端 GPU 中，将零除以零可能会导致 NaN（“非数字”）；在其他移动端 GPU 上，它可能会导致无穷大、零或任何其他不明值。务必在目标设备上测试着色器以检查着色器是否受支持。




# 升级 URP
本文是基于7.3版本的 URP 编写的，有些暂时还不支持的内容可能在后续版本更新迭代。

## 结构

首先要在SubShader的Tags中添加"RenderPipeline" = "UniversalPipeline"，并且使用HLSL的宏代替旧版的CG语言宏。

|Built-in|URP|
|:--|:--|
|CGPROGRAM / HLSLPROGRAM|HLSLPROGRAM|
|ENDCG / ENDHLSL|ENDHLSL|
|CGINCLUDE / HLSLINCLUDE|HLSLINCLUDE|

## Include文件的改动

|Content|Built-in|URP|
|:--|:--|:--|
|Core|Unity.cginc|Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl|
|Light|AutoLight.cginc|Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl|
|Shadows|AutoLight.cginc|Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl|
|Surface shaders|Lighting.cginc|无|

其他常用的include文件:

- Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl
- Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl
- Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl
- Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl
- Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl
- Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl
- Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareOpaqueTextue.hlsl

## 光照模式

|Built-in|URP|
|:--|:--|
|ForwardBase|UniversalForward|
|ForwardAdd|无|
|Deferred and related|UniversalGBuffer seems to have just been added to URP|
|Vertex and related|无|
|ShadowCaster|ShadowCaster|
|MotionVectors|暂不支持|

URP其他支持的光照模式：

- DepthOnly
- Meta (用于烘焙光照贴图)
- Universal2D

## 变体(Variants)

URP支持着色器的变体，可以使用#pragma multi_compile宏实现编译不同需求下的着色器，常见的内置关键字有：

- _MAIN_LIGHT_SHADOWS
- _MAIN_LIGHT_SHADOWS_CASCADE
- _ADDITIONAL_LIGHTS_VERTEX
- _ADDITIONAL_LIGHTS
- _ADDITIONAL_LIGHT_SHADOWS
- _SHADOWS_SOFT
- _MIXED_LIGHTING_SUBTRACTIVE

## 预定义的着色器预处理宏

### 辅助宏(Helpers)

|Built-in|URP|
|:--|:--|
|UNITY_PROJ_COORD(a)|无，使用 a.xy/a.w 来代替|
|UNITY_INITIALIZE_OUTPUT(type, name)|ZERO_INITIALIZE(type, name)|

### 阴影贴图

需要包含 Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl

|Built-in|URP|
|:--|:--|
|UNITY_DECLARE_SHADOWMAP(tex)|TEXTURE2D_SHADOW_PARAM(textureName, samplerName)|
|UNITY_SAMPLE_SHADOW(tex, uv)|SAMPLE_TEXTURE2D_SHADOW(textureName, samplerName, coord3)|
|UNITY_SAMPLE_SHADOW_PROJ(tex, uv)|SAMPLE_TEXTURE2D_SHADOW(textureName, samplerName, coord4.xyz/coord4.w)|

### 纹理/采样器的声明宏

|Built-in|URP|
|:--|:--|
|UNITY_DECLARE_TEX2D(name)|TEXTURE2D(textureName); SAMPLER(samplerName);|
|UNITY_DECLARE_TEX2D_NOSAMPLER(name)|TEXTURE2D(textureName);|
|UNITY_DECLARE_TEX2DARRAY(name)|TEXTURE2D_ARRAY(textureName); SAMPLER(samplerName);|
|UNITY_SAMPLE_TEX2D(name, uv)|SAMPLE_TEXTURE2D(textureName, samplerName, coord2)|
|UNITY_SAMPLE_TEX2D_SAMPLER(name, samplername, uv)|SAMPLE_TEXTURE2D(textureName, samplerName, coord2)|
|UNITY_SAMPLE_TEX2DARRAY(name, uv)|SAMPLE_TEXTURE2D_ARRAY(textureName, samplerName, coord2, index)|
|UNITY_SAMPLE_TEX2DARRAY_LOD(name, uv, lod)|SAMPLE_TEXTURE2D_ARRAY_LOD(textureName, samplerName, coord2, index, lod)|

## 内置的着色器辅助函数

可以在 Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl 看到下方的所有函数

### 顶点变换函数

|Built-in|URP|
|:--|:--|
|float4 UnityObjectToClipPos(float3 pos)|float4 TransformObjectToHClip(float3 positionOS)|
|float3 UnityObjectToViewPos(float3 pos)|TransformWorldToView(TransformObjectToWorld(positionOS))|

### 泛用的辅助函数

|Built-in|URP|Include|
|:--|:--|:--|
|float3 WorldSpaceViewDir (float4 v)|float3 GetWorldSpaceViewDir(float3 positionWS)|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl"|
|float3 ObjSpaceViewDir (float4 v)|无，使用 TransformWorldToObject(GetCameraPositionWS()) - objectSpacePosition;||
|float2 ParallaxOffset (half h, half height, half3 viewDir)|可能没有，从 UnityCG.cginc 复制||
|fixed Luminance (fixed3 c)|real Luminance(real3 linearRgb)|Include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"|
|fixed3 DecodeLightmap (fixed4 color)|real3 DecodeLightmap(real4 encodedIlluminance, real4 decodeInstructions)|Include "Packages/com.unity.render-pipelines.core/ShaderLibrary/EntityLighting.hlsl" URP中decodeInstructions 为 half4(LIGHTMAP_HDR_MULTIPLIER, LIGHTMAP_HDR_EXPONENT, 0.0h, 0.0h)|
|float4 EncodeFloatRGBA (float v)|可能没有， 从 UnityCG.cginc 复制||
|float DecodeFloatRGBA (float4 enc)|可能没有， 从 UnityCG.cginc 复制||
|float2 EncodeFloatRG (float v)|可能没有， 从 UnityCG.cginc 复制||
|float DecodeFloatRG (float2 enc)|可能没有， 从 UnityCG.cginc 复制||
|float2 EncodeViewNormalStereo (float3 n)|可能没有， 从 UnityCG.cginc 复制||
|float3 DecodeViewNormalStereo (float4 enc4)|可能没有， 从 UnityCG.cginc 复制||

### 前向渲染辅助函数

|Built-in|URP|Include|
|:--|:--|:--|
|float3 WorldSpaceLightDir (float4 v)|_MainLightPosition.xyz - TransformObjectToWorld(objectSpacePosition)|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"|
|float3 ObjSpaceLightDir (float4 v)|TransformWorldToObject(_MainLightPosition.xyz) - objectSpacePosition|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"|
|float3 Shade4PointLights (…)|无，可尝试用half3 VertexLighting(float3 positionWS, half3 normalWS)|include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"|

### 屏幕空间辅助函数

|Built-in|URP|Include|
|:--|:--|:--|
|float4 ComputeScreenPos (float4 clipPos)|float4 ComputeScreenPos(float4 positionCS)|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl"|
|float4 ComputeGrabScreenPos (float4 clipPos)|无||

### 顶点光照的辅助函数

|Built-in|URP|Include|
|:--|:--|:--|
|float3 ShadeVertexLights (float4 vertex, float3 normal)|无，可尝试用 UNITY_LIGHTMODEL_AMBIENT.xyz + VertexLighting(…)|include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"|

可以在 Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl 中找到一些通用函数

## 内置的着色器变量

|Built-in|URP|Include|
|:--|:--|:--|
|_LightColor0|_MainLightColor|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"|
|_WorldSpaceLightPos0|_MainLightPosition|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Input.hlsl"|
|_LightMatrix0|可能还不支持||
|unity_4LightPosX0, unity_4LightPosY0, unity_4LightPosZ0|URP中，额外的灯光存储在一个数组或缓冲中(取决于平台),使用Light GetAdditionalLight(uint i, float3 positionWS)获取光照信息|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"|
|unity_4LightAtten0|URP中，额外的灯光存储在一个数组或缓冲中(取决于平台),使用Light GetAdditionalLight(uint i, float3 positionWS)获取光照信息|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"|
|unity_LightColor|URP中，额外的灯光存储在一个数组或缓冲中(取决于平台),使用Light GetAdditionalLight(uint i, float3 positionWS)获取光照信息|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"|
|unity_WorldToShadow|float4x4 _MainLightWorldToShadow[MAX_SHADOW_CASCADES + 1] or _AdditionalLightsWorldToShadow[MAX_VISIBLE_LIGHTS]|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"|

可以使用GetAdditionalLight(…)获取额外的光源，也可以使用GetAdditionalLightsCount()查询额外的光源数量。

## 其他方法

### 阴影

更多阴影相关函数可以查看 Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl

|Built-in|URP|
|:--|:--|
|UNITY_SHADOW_COORDS(x)|可能没有，可以写作float4 shadowCoord : TEXCOORD0;|
|TRANSFER_SHADOW(a)|a.shadowCoord = TransformWorldToShadowCoord(worldSpacePosition)|
|SHADOWS_SCREEN|暂不支持|

### 雾

更多雾相关的函数可以查看 Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl

|Built-in|URP|
|:--|:--|
|UNITY_FOG_COORDS(x)|可能没有，可以写作float fogCoord : TEXCOORD0;|
|UNITY_TRANSFER_FOG(o, outpos)|o.fogCoord = ComputeFogFactor(clipSpacePosition.z);|
|UNITY_APPLY_FOG(coord, col)|color = MixFog(color, i.fogCoord);|

### 深度

可以包含 "Packages/com.unity.render-pipelines.universal/ShaderLibrary/DeclareDepthTexture.hlsl" 并使用 _CameraDepthTexture来调用深度纹理。也可以使用SampleSceneDepth(…) 和 LoadSceneDepth(…)。

|Built-in|URP|Include|
|:--|:--|:--|
|LinearEyeDepth(sceneZ)|LinearEyeDepth(sceneZ, _ZBufferParams)|Include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"|
|Linear01Depth(sceneZ)|Linear01Depth(sceneZ, _ZBufferParams)|Include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"|

### 其他

|Built-in|URP|Include|
|:--|:--|:--|
|ShadeSH9(normal)|SampleSH(normal)|Include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"|
|unity_ColorSpaceLuminance|无，使用Luminance()|Include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"|

## 后期/特效

URP不支持OnPreCull, OnPreRender, OnPostRender 和 OnRenderImage. 支持 OnRenderObject 和 OnWillRenderObject。RenderPipelineManager提供了渲染管线中注入的位置：

- beginCameraRendering(ScriptableRenderContext context, Camera camera)
- endCameraRendering(ScriptableRenderContext context, Camera camera)
- beginFrameRendering(ScriptableRenderContext context,Camera[] cameras)
- endFrameRendering(ScriptableRenderContext context,Camera[] cameras)

例如：

```
void OnEnable()
{
    RenderPipelineManager.beginCameraRendering += MyCameraRendering;
}

void OnDisable()
{
    RenderPipelineManager.beginCameraRendering -= MyCameraRendering;
}

void MyCameraRendering(ScriptableRenderContext context, Camera camera)
{
    ...
    if(camera == myEffectCamera)
    {
    ...
        UniversalRenderPipeline.RenderSingleCamera(context, camera);
    }
    ...
}
```

另外，可以创建ScriptableRendererFeature来实现后期处理效果。可以在管线的不同阶段注入ScriptableRenderPasses：

- BeforeRendering
- BeforeRenderingShadows
- AfterRenderingShadows
- BeforeRenderingPrepasses
- AfterRenderingPrePasses
- BeforeRenderingOpaques
- AfterRenderingOpaques
- BeforeRenderingSkybox
- AfterRenderingSkybox
- BeforeRenderingTransparents
- AfterRenderingTransparents
- BeforeRenderingPostProcessing
- AfterRenderingPostProcessing
- AfterRendering

下面是一个示例：

```
public class CustomRenderPassFeature : ScriptableRendererFeature
{
    class CustomRenderPass : ScriptableRenderPass
    {
        CustomRPSettings _CustomRPSettings;
        RenderTargetHandle _TemporaryColorTexture;

        private RenderTargetIdentifier _Source;
        private RenderTargetHandle _Destination;

        public CustomRenderPass(CustomRPSettings settings)
        {
            _CustomRPSettings = settings;
        }

        public void Setup(RenderTargetIdentifier source, RenderTargetHandle destination)
        {
            _Source = source;
            _Destination = destination;
        }

        public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)
        {
            _TemporaryColorTexture.Init("_TemporaryColorTexture");
        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
            CommandBuffer cmd = CommandBufferPool.Get("My Pass");

            if (_Destination == RenderTargetHandle.CameraTarget)
            {
                cmd.GetTemporaryRT(_TemporaryColorTexture.id, renderingData.cameraData.cameraTargetDescriptor, FilterMode.Point);
                cmd.Blit(_Source, _TemporaryColorTexture.Identifier());
                cmd.Blit(_TemporaryColorTexture.Identifier(), _Source, _CustomRPSettings.m_Material);
            }
            else
            {
                cmd.Blit(_Source, _Destination.Identifier(), _CustomRPSettings.m_Material, 0);
            }

            context.ExecuteCommandBuffer(cmd);
            CommandBufferPool.Release(cmd);
        }

        public override void FrameCleanup(CommandBuffer cmd)
        {
            if (_Destination == RenderTargetHandle.CameraTarget)
            {
                cmd.ReleaseTemporaryRT(_TemporaryColorTexture.id);
            }
        }
    }

    [System.Serializable]
    public class CustomRPSettings
    {
        public Material m_Material;
    }

    public CustomRPSettings m_CustomRPSettings = new CustomRPSettings();
    CustomRenderPass _ScriptablePass;

    public override void Create()
    {
        _ScriptablePass = new CustomRenderPass(m_CustomRPSettings);

        _ScriptablePass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        _ScriptablePass.Setup(renderer.cameraColorTarget, RenderTargetHandle.CameraTarget);
        renderer.EnqueuePass(_ScriptablePass);
    }
}
```

# HLSL 函数参考
下表列出了 HLSL 中可用的内部函数。每个函数都有简短说明，并提供指向有关输入参数和返回类型的更多详细信息的参考页的链接。

|名称|说明|
|:--|:--|
|[**abort**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/abort) |终止正在执行的当前绘图或调度调用。|
|[**abs**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-abs)|每个组件) (绝对值。|
|[**acos**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-acos)|返回 x 的每个分量的反余弦。|
|[**all**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-all)|测试 x 的所有组件是否为非零。|
|[**AllMemoryBarrier**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/allmemorybarrier)|阻止组中所有线程的执行，直到所有内存访问都完成。|
|[**AllMemoryBarrierWithGroupSync**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/allmemorybarrierwithgroupsync)|阻止组中所有线程的执行，直到所有内存访问都完成并且组中的所有线程都达到此调用。|
|[**any**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-any)|测试 x 的任何组件是否为非零。|
|[**asdouble**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/asdouble)|重新解释将转换值转换为双精度值。|
|[**asfloat**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-asfloat)|将输入类型转换为 float。|
|[**asin**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-asin)|返回 x 的每个分量的反正弦。|
|[**asint**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-asint)|将输入类型转换为整数。|
|[**asuint**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/asuint)|将64位类型的位模式重新解释为 uint。|
|[**asuint**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-asuint)|将输入类型转换为无符号整数。|
|[**atan**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-atan)|返回 x 的反正切值。|
|[**atan2**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-atan2)|返回两个值的反正切值 (x，y) 。|
|[**ceil**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-ceil)|返回大于或等于 x 的最小整数。|
|[**CheckAccessFullyMapped**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/checkaccessfullymapped)|确定 **示例** 或 **加载** 操作中的所有值是否都访问了 [平铺资源](https://docs.microsoft.com/zh-cn/windows/desktop/direct3d11/direct3d-11-2-features)中的映射磁贴。|
|[**clamp**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-clamp)|夹紧 x 到最 [ 小值，最大值 ] 。|
|[**clip**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-clip)|如果 x 的任何分量小于零，则放弃当前像素。|
|[**cos**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-cos)|返回 x 的余弦值。|
|[**cosh**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-cosh)|返回 x 的双曲余弦值。|
|[**countbits**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/countbits)|计算输入整数中每个组件) (的位数。|
|[**cross**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-cross)|返回两个三维向量的叉积。|
|[**D3DCOLORtoUBYTE4**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-d3dcolortoubyte4)|Swizzles 和缩放4D 向量 xto 的组件补偿某些硬件中是否缺少 UBYTE4 支持。|
|[**ddx**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-ddx)|返回 x 的部分导数，相对于屏幕空间 x 坐标。|
|[**ddy**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-ddy)|返回 x 的部分导数，与屏幕空间的 y 坐标有关。|
|[**degrees**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-degrees)|将 x 从弧度转换为度。|
|[**diterminant**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-determinant)|返回正方形 matrix m 的行列式。|
|[**DeviceMemoryBarrier**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/devicememorybarrier)|阻止组中所有线程的执行，直到所有设备内存访问都完成。|
|[**DeviceMemoryBarrierWithGroupSync**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/devicememorybarrierwithgroupsync)|阻止组中所有线程的执行，直到所有设备内存访问完成并且组中的所有线程都达到此调用。|
|[**distance**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-distance)|返回两个点之间的距离。|
|[**dot**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-dot)|返回两个向量的点积。|
|[**dst**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dst)|计算距离向量。|
|[**errorf**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/errorf)|向信息队列提交一条错误消息。|
|[**EvaluateAttributeAtCentroid**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/evaluateattributeatcentroid)|计算像素质心。|
|[**EvaluateAttributeAtSample**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/evaluateattributeatsample)|在索引样本位置计算。|
|[**EvaluateAttributeSnapped**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/evaluateattributesnapped)|计算带有偏移量的质心的像素。|
|[**.exp**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-exp)|返回以 e 为底的指数。|
|[**exp2**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-exp2)|每个组件) 以2为底的指数 (。|
|[**f16tof32**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/f16tof32)|将在 uint 的下半部分中存储的 float16 转换为 float。|
|[**f32tof16**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/f32tof16)|将输入转换为 float16 类型。|
|[**faceforward**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-faceforward)|返回-n * 符号 (点 (i，ng) ) 。|
|[**firstbithigh**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/firstbithigh)|获取第一组位的位置，从最高顺序位开始，按组件向下移动。|
|[**firstbitlow**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/firstbitlow)|返回第一组位的位置（从最低顺序位开始，按组件向上处理）。|
|[**floor**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-floor)|返回小于或等于 x 的最大整数。|
|[**fma**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-fma)|返回一个与 b + c 的双精度的带外乘法 * 。|
|[**fmod**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-fmod)|返回 x/y 的浮点余数。|
|[**frac**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-frac)|返回 x 的小数部分。|
|[**frexp**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-frexp)|返回 x 的尾数和指数。|
|[**fwidth**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-fwidth)|返回 abs (ddx (x) ) + abs (ddy (x) )|
|[**GetRenderTargetSampleCount**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-getrendertargetsamplecount)|返回呈现器目标样本的数目。|
|[**GetRenderTargetSamplePosition**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-getrendertargetsampleposition)|返回给定示例索引 (x，y) 的位置。|
|[**GroupMemoryBarrier**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/groupmemorybarrier)|阻止组中所有线程的执行，直到所有组共享访问都完成。|
|[**GroupMemoryBarrierWithGroupSync**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/groupmemorybarrierwithgroupsync)|阻止组中所有线程的执行，直到所有组共享访问完成并且组中的所有线程都达到此调用。|
|[**InterlockedAdd**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedadd)|执行保证的原子值向目标资源变量的原子性。|
|[**InterlockedAnd**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedand)|执行保证的原子和。|
|[**InterlockedCompareExchange**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedcompareexchange)|以原子方式将输入与比较值进行比较并交换结果。|
|[**InterlockedCompareStore**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedcomparestore)|以原子方式将输入与比较值进行比较。|
|[**InterlockedExchange**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedexchange)|将值分配给 dest 并返回原始值。|
|[**InterlockedMax**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedmax)|执行保证的原子最大值。|
|[**InterlockedMin**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedmin)|执行保证的原子最小值。|
|[**InterlockedOr**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedor)|执行保证的原子或。|
|[**InterlockedXor**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/interlockedxor)|执行保证的原子 xor。|
|[**isfinite**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-isfinite)|如果 x 是有限的，则返回 true，否则返回 false。|
|[**isinf**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-isinf)|如果 x 为 + INF 或-INF，则返回 true，否则返回 false。|
|[**isnan**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-isnan)|如果 x 为 NAN 或 QNAN，则返回 true，否则返回 false。|
|[**ldexp**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-ldexp)|返回 x * 2exp|
|[**length**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-length)|返回向量 v 的长度。|
|[**lerp**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-lerp)|返回 (y-x) 的 x + s。|
|[**lit**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-lit)|返回光源向量 (环境、漫射、镜面、1)|
|[**log**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-log)|返回 x 的以 e 为底的对数。|
|[**log10**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-log10)|返回 x 的以10为底的对数。|
|[**log2**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-log2)|返回 x 的以2为底的对数。|
|[**mad**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/mad)|对三个值执行算术乘/加法运算。|
|[**max**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-max)|选择 x 和 y 的较大。|
|[**min**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-min)|选择 x 和 y 中的较小者。|
|[**modf**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-modf)|将值 x 拆分为小数部分和整数部分。|
|[**msad4**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-msad4)|比较4个字节的引用值和8个字节的源值，并累计4个和的向量。|
|[**mul**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-mul)|使用 x 和 y 执行矩阵乘法。|
|[**noise**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-noise)|使用 Perlin-干扰算法生成随机值。|
|[**normalize**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-normalize)|返回规范化向量。|
|[**pow**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-pow)|返回 xy。|
|[**printf**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/printf)|将自定义着色器消息提交到信息队列。|
|[**Process2DQuadTessFactorsAvg**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/process2dquadtessfactorsavg)|为四个修补程序生成已更正的分割系数。|
|[**Process2DQuadTessFactorsMax**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/process2dquadtessfactorsmax)|为四个修补程序生成已更正的分割系数。|
|[**Process2DQuadTessFactorsMin**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/process2dquadtessfactorsmin)|为四个修补程序生成已更正的分割系数。|
|[**ProcessIsolineTessFactors**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/processisolinetessfactors)|生成等值线的舍入分割系数。|
|[**ProcessQuadTessFactorsAvg**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/processquadtessfactorsavg)|为四个修补程序生成已更正的分割系数。|
|[**ProcessQuadTessFactorsMax**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/processquadtessfactorsmax)|为四个修补程序生成已更正的分割系数。|
|[**ProcessQuadTessFactorsMin**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/processquadtessfactorsmin)|为四个修补程序生成已更正的分割系数。|
|[**ProcessTriTessFactorsAvg**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/processtritessfactorsavg)|生成适用于三个修补程序的已更正分割系数。|
|[**ProcessTriTessFactorsMax**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/processtritessfactorsmax)|生成适用于三个修补程序的已更正分割系数。|
|[**ProcessTriTessFactorsMin**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/processtritessfactorsmin)|生成适用于三个修补程序的已更正分割系数。|
|[**radians**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-radians)|将 x 转换为弧度。|
|[**rcp**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/rcp)|计算快速、近似、按分量的倒数。|
|[**reflect**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-reflect)|返回反射向量。|
|[**refract**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-refract)|返回折射向量。|
|[**reversebits**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/reversebits)|为每个分量反转位的顺序。|
|[**round**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-round)|将 x 舍入到最接近的整数|
|[**rsqrt**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-rsqrt)|返回 1/sqrt (x)|
|[**saturate**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-saturate)|夹紧 x 到 [ 0，1]|
|[**sign**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-sign)|计算 x 的符号。|
|[**sin**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-sin)|返回 x 的正弦值|
|[**sincos**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-sincos)|返回 x 的正弦和余弦值。|
|[**sinh**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-sinh)|返回 x 的双曲正弦值|
|[**smoothstep**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-smoothstep)|返回介于0和1之间的平滑 Hermite 内插。|
|[**sqrt**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-sqrt)|每个组件 (平方根)|
|[**step**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-step)|返回 (x >=) ？ 1 : 0|
|[**tan**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tan)|返回 x 的正切值|
|[**tanh**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tanh)|返回 x 的双曲正切值|
|[**tex1D (s，t)**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex1d)|1D 纹理查找。|
|[**tex1D (s、t、ddx、ddy)**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex1d-s-t-ddx-ddy)|1D 纹理查找。|
|[**tex1Dbias**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex1dbias)|带有偏移的一维纹理查找。|
|[**tex1Dgrad**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex1dgrad)|使用渐变的一维纹理查找。|
|[**tex1Dlod**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex1dlod)|用 LOD 的一维纹理查找。|
|[**tex1Dproj**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex1dproj)|用 projective 相除的1D 纹理查找。|
|[**tex2D (s，t)**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2d)|2D 纹理查找。|
|[**tex2D (s、t、ddx、ddy)**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2d-s-t-ddx-ddy)|2D 纹理查找。|
|[**tex2Dbias**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2dbias)|带有偏移的2D 纹理查找。|
|[**tex2Dgrad**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2dgrad)|使用渐变的2D 纹理查找。|
|[**tex2Dlod**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2dlod)|2D 纹理查找和 LOD。|
|[**tex2Dproj**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex2dproj)|projective 相除的2D 纹理查找。|
|[**tex3D (s，t)**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex3d)|3D 纹理查找。|
|[**tex3D (s、t、ddx、ddy)**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex3d-s-t-ddx-ddy)|3D 纹理查找。|
|[**tex3Dbias**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex3dbias)|带有偏置的3D 纹理查找。|
|[**tex3Dgrad**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex3dgrad)|使用渐变的3D 纹理查找。|
|[**tex3Dlod**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex3dlod)|用 LOD 的3D 纹理查找。|
|[**tex3Dproj**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-tex3dproj)|带有 projective 除法的3D 纹理查找。|
|[**texCUBE (s，t)**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-texcube)|多维数据集纹理查找。|
|[**texCUBE (s、t、ddx、ddy)**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-texcube-s-t-ddx-ddy)|多维数据集纹理查找。|
|[**texCUBEbias**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-texcubebias)|带有偏移的多维数据集纹理查找。|
|[**texCUBEgrad**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-texcubegrad)|使用渐变的多维数据集纹理查找。|
|[**texCUBElod**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-texcubelod)|LOD 的多维数据集纹理查找。|
|[**texCUBEproj**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-texcubeproj)|Projective 相除的多维数据集纹理查找。|
|[**transpose**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-transpose)|返回矩阵 m 的转置。|
| [**trunc**](https://docs.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-trunc) |将) (的浮点值截断到整数值 (s|下表列出了 HLSL 中可用的内部函数。每个函数都有简短说明，并提供指向有关输入参数和返回类型的更多详细信息的参考页的链接。

