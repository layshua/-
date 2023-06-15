[Unity小白的TA之路-Shader开发|图形渲染管线|URP|性能优化|图形渲染|PostProcessing (91maketop.github.io)](https://91maketop.github.io/ta/#/README)
# 约定
## Unity 中的坐标系

![image-20220702145045963](image-20220702145045963.png)
>注意观察空间是右手坐标系

模型和世界空间中，前向是 $Z$ 轴
观察空间中，摄像机的前向为 $-Z$ 轴，深度越大，Z 轴坐标越小

## 矩阵运算
变换顶点时，使用左乘变换矩阵，因为 Untiy 提供的内置矩阵都是列向量形式。

CG 是按行优先存储的，声明时，从左到右从上到下按顺序填就可以，
注意: 
1. 行优先只会影响声明时的顺序，不等于计算时是行向量形式。
2. Unity 的矩阵类型 Matrix4x4 是列优先存储!

## Unity 中的屏幕坐标 ComputeScreenPos/VPOS/WPOS


在写 Shader 的过程中，我们可以获得片元在屏幕上的像素位置

在顶点 / 片元着色器中有两种方式获得片元的屏幕坐标

**第一种**：在片元着色器的输入声明 **VPOS** 或 **WPOS** 语义, VPOS 是 HLSL 中对屏幕坐标的语义，WPOS 是 Cg 中对屏幕坐标的语义

我们可以在**片元着色器**中这样写：

```c
fixed4 frag (float4 sp :VPOS) : SV_Target
{
    //_ScreenParams.xy是屏幕分辨率,.xy表示取前2个值构成2维
    //用屏幕坐标除以屏幕分辨率，得到视口空间中的坐标
    return fixed4 (sp.xy/_ScreenParams.xy,0.0,1.0);
}
```

**VPOS/WPOS** 语义定义的输入是个 float4 类型的变量，它的 xy 值代表**屏幕空间**中的像素坐标。

如果屏幕分辨率为 400x300，那么 x 的范围就是 $[0.5,400.5]$ ,y 的范围是 $[0.5,300.5]$

**注意**：这里的像素坐标并不是整数值，这是因为 OpenGL 和 DirectX 10 以后的版本认为像素重心对应的是浮点值中的 0.5

**VPOS/WPOS** 的 z 分量范围是 $[0,1]$ , 在相机的近裁剪平面处 z 为 0，远裁剪平面处 z 为 1

**VPOS/WPOS** 的 w 分量范围是 $[\frac{1}{Near},\frac{1}{Far}]$ , $Near$ 和 $Far$ 对应 Camera 组件中设置的近裁切平面和远裁切平面距离相机的远近，若为正交投影，则 $w$ 恒为 1

代码最后：用屏幕坐标除以屏幕分辨率，得到**视口空间**中的坐标，**视口空间 (viewport space)** 就是把屏幕坐标归一化，屏幕左下角为 $(0,0)$ , 右上交为 $(1,1)$


**第二种**：通过 Unity 提供的 `ComputeScreenPos` 函数, 这个函数在 UnityCG. cginc 里被定义。

1. ComputeScreenPos 函数在顶点着色器中使用，输入的是 posCS (裁剪空间顶点坐标)

2. ComputeScreenPos 函数的输出结果传给片元着色器, 在片元着色器进行一个除法：自己的 xy 除以 w

```c
struct VertOut
 {
     float4 posCS : SV_POSITION;
     float4 scrpos : TEXCOORD0;
 };
  VertOut vert (appdata v)
 {
     VertOut o;
     o.posCS = UnityObjectToClipPos(v.vertex);
     //第一步：把ComputeScreenPos的结果保存到scrPos中
     o.scrpos = ComputeScreenPos(o.posCS);
     return o;
 }
 ​
 fixed4 frag (VertOut i ) : SV_Target
 {
     //第二步：用scrPos.xy除以scrpos.w得到视口空间的坐标
     float2 wcoord = i.scrpos.xy/i.scrpos.w;
     return fixed4 (wcoord,0.0,1.0);
 }
```

**注：为什么不直接在 ComputeScreenPos 函数中直接除以 $w$ ，还需要在片元着色器中进行除？**

答案是：投影空间不是线性的，不能在投影空间中插值。我们的 pos 是在顶点着色器得到的，如果直接在顶点着色器就除以 $w$ ，将裁剪空间转为 NDC 空间（正方体），再进行插值，此时是对 $\frac{x}{w}$ 和 $\frac{y}{w}$ 会得到错误的结果。所以必须先插值，再在片元着色器除以 $w$

在片元着色器进行除法后，就得到了视口空间的坐标了，即 $x,y$ 的范围都在 $[0,1]$

$z,w$ 是直接在顶点着色器输出的裁剪空间的 z, w 值 (经过插值后)。如果是透视投影，z 的范围是 $[-Near,Far]$ , $w$ 的范围是 $[Near,Far]$ ; 如果是正交投影， $z$ 的范围是 $[-1,1]$ , $w$ 的值恒为 1



# ShaderLab 语法基础
## 1 组织结构
Shader 中可以编写多个子着色器（SubShader），但至少需要一个。

在应用程序运行过程中，GPU 会先检测第一个子着色器能否正常运行，如果不能正常运行就会再检测第二个，以此类推。
假如当前 GPU 的硬件版本太旧，以至于所有的子着色器都无法正常运行时，则执行最后的回退（Fallback）命令，运行指定的一个基础着色器。

如果编写的是顶点-片段着色器（Vertex-Fragment Shader），每个子着色器中还会包含一个甚至多个 Pass。在运行的过程中，如果某个子着色器能够在当前 GPU 上运行，那么该子着色器内的所有 Pass 会依次执行，每个 Pass 的输出的结果会以指定的方式与上一步的结果进行混合，最终输出。

如果编写的是表面着色器（Surface Shader），着色器的代码也是包含在子着色器中，但是与顶点-片段着色器不同的是，表面着色器不会再嵌套 Pass。系统在编译表面着色器的时候会自动生成多个对应的 Pass，最终编译出来的 Shader 本质上就是顶点-片段着色器。


## 2 名称

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

## 3 Properties

![[Pasted image 20230615092149.png]]

![[Pasted image 20230615102705.png]]

```c file:所有类型属性汇总
Properties
{
    _MyInt ( "Int Property",Int) = 1                       //浮点类型
    _MyFloat ( "Float Property",Float) = 1.5               //浮点类型
    
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
int _MyInt;             //整数类型
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

## 4 SubShader
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
**每个 SubShader 都可以设置一个或者多个标签（Tags）和渲染状态（States），然后定义至少一个 Pass**。在 SubShader 中设置的渲染状态会影响到该 SubShader 中所有的 Pass，如果想要某些状态不影响其他 Pass，**可以针对某个 Pass 单独设置渲染状态。但是需要注意的是，部分渲染状态在 Pass 中并不支持。**
### （1）Tags

![[Pasted image 20230615092421.png]]
>以上标签仅可以在 SubShader 中声明，不可以在 Pass 块中声明。


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

|类型名称|描述|
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
### （2）渲染状态
如果想某些 Pass 的渲染状态不影响到其他的 Pass，**可以在该 Pass 中单独设置渲染状态**。**这些渲染状态在 SubShader 中同样被允许使用**，需要特别注意的是，在 SubShader 中使用会影响到该 SubShader 中的所有 Pass。

|渲染状态|数值|作用|
|:--|:--|:--|
|Cull|Cull Back/Front/ Off|设置多边形的剔除方式, 有背面剔除、正面剔除、不剔除﹐默认为 Back|
|ZTest|ZTest (Less/Greater/LEqual/GEqual/Equal/NotEquall /Always)|设置深度测试的对比方式, 默认为 LEqual |
|ZWrite|ZWrite On/ Off|设置是否写入深度缓存, 默认为 On |
|Blend|Blend  sourceBlendMode  destBlendMode|设置渲染图像的混合方式|
|ColorMask|ColorMask RGB/A/0/或者 R、G、B、A 的任意组合|设置颜色通道的写入蒙版﹐默认蒙版为 RGBA, 当设置为 0 时, 则无法写入任何颜色|

### （3）Pass
#### 自定义名称
```cs
pass
{
    Name  "mypass"
}
```

通过这个名称，我们可以使用 ShaderLab 的 `UsePass` 命令来直接**使用其他 Unity Shader 中的 Pass**，提高代码复用性。例如:
`UsePass "MyShader/MYPASSNAME"`

> [!warning] Title
> Unity 内部会把所有 Pass 的名称转换成大写字母，所以使用 UsePass 时要使用大写字母的名字

#### Pass 专用 Tags
Pass 也可以设置标签，单核 SubShader 不同
![[Pasted image 20230615093350.png]]

其他 Tags：
`UsePass`: 如我们之前提到的一样，可以使用该命令来复用其他 Unity Shader 中的 Pass; 
`GrabPass`: 该 Pass 负责抓取屏幕并将结果存储在一张纹理中, 以用于后续的 Pass 处理 。
##### LightMode
`LightMode`（灯光模式）标签定义了 Pass 在光照渲染流水线中的渲染规则
![[Pasted image 20230615142758.png]]

##### PassFlags
PassFlags 标签用于更改渲染流水线传递数据给 Pass 的方式。目前仅可以使用的值为 `OnlyDirectional`。
当使用前向渲染的时候，这个标签使得只有主要平行光、环境光或灯光探针、光照贴图的数据才能传递给 Shader，SH 和逐顶点灯光不能传递数据。

### （4）Fallback
Fallback 在所有 SubShader 之后进行定义。当所有的 SubShader 都不能在当前显卡上运行的时候，就会运行 Fallback 定义的 Shader。它的语法如下：
`Fallback "name"`
最常用于 Fallback 的 Shader 为 Unity 内置的 Diffuse。
如果觉得某些 Shader 肯定可以在目标显卡上运行，没有指定 Fallback 的必要，可以使用 Fallback Off 关闭 Fallback 功能，或者直接什么都不写。

### （5）LOD
LOD：Level of Detail  
**shader 的 LOD 和模型的 LOD 作用不同！shader 的 LOD 只是用来选择 SubShader 的**
作用：unity 引擎会根据不同的 LOD 值在使用不同的 SubShader  
Unity 选择对应的 Subshader 会从上往下寻找第一个小于等于  `shader.maximumLOD`  值的 SubShader。

```c
SubShader
{
    Tags { "RenderType" = "Opaque" }
    LOD 600 // LOD这里设置为600

    CGPROGRAM
    ...
    ENDCG
}
        
SubShader
{
    Tags { "RenderType" = "Opaque" }
    LOD 500 // LOD这里设置为500

    CGPROGRAM
    ...
    ENDCG
}
SubShader
{
    Tags { "RenderType" = "Opaque" }
    LOD 400 // LOD这里设置为400
    
    CGPROGRAM
    ...
    ENDCG
}

```

通过脚本调整 `shader.maximumLOD`

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class LODCtrl : MonoBehaviour
{
    public Shader shader;//将Shader拖进来即可
    
    void Start() {
        Debug.Log(this.shader.maximumLOD);
    }
    // Update is called once per frame
    void Update() {
        if (Input.GetKeyDown(KeyCode.A))
        {
            // 当前这个shader最大的LOD_value;
            this.shader.maximumLOD = 600;
        }
        if (Input.GetKeyDown(KeyCode.B))
        {
            this.shader.maximumLOD = 500;
        }
        if (Input.GetKeyDown(KeyCode.C))
        {
            this.shader.maximumLOD = 300;
        }
 
    }
}
```

（4）运行，查看效果，通过按 ABC 按键，修改 maximumLOD 的值。查看 Cube 颜色的变化。
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

## 数据类型
顶点函数和片段函数中支持的数据类型
![[Pasted image 20230614194727.png]]


> [!NOTE] uniform 关键字
> uniform 关键词和图形 API 的 unitform 作用不一样，只是一个修饰词。在 UntiyShader 中，uniform 关键词是可以省略的。

### 整数数据类型

整数（`int` 数据类型）通常用作循环计数器或数组索引。为此，它们通常可以在各种平台上正常工作。

根据平台的不同，GPU 可能不支持整数类型。例如，Direct3D 9 和 OpenGL ES 2.0 GPU 仅对浮点数据进行运算，并且可以使用相当复杂的浮点数学指令来模拟简单的整数表达式（涉及位运算或逻辑运算）。

Direct3D 11、OpenGL ES 3、Metal 和其他现代平台都对整数数据类型有适当的支持，因此使用位移位和位屏蔽可以按预期工作。

### 复合矢量/矩阵类型

HLSL 具有从基本类型创建的内置矢量和矩阵类型。例如，`float3` 是一个 3D 矢量，具有分量 .x、. y 和 .z，而 `half4` 是一个中等精度 4D 矢量，具有分量 .x、. y、. z 和 .w。或者，可使用 .r、. g、. b 和 .a 分量来对矢量编制索引，这在处理颜色时很有用。

矩阵类型以类似的方式构建；例如 `float4x4` 是一个 4x4 变换矩阵。请注意，某些平台仅支持方形矩阵，最主要的是 OpenGL ES 2.0。

### 纹理/采样器类型

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

## 语义
参数后被冒号隔开并且全部大写的关键词就是语义。

当使用 CG 语言编写着色器函数的时候，函数的输入参数和输出参数都需要填充一个语义（Semantic）来表示它们要传递的数据信息。语义可以执行大量烦琐的操作，使用户能够避免直接与 GPU 底层进行交流。

### 顶点着色器输入语义
![[Pasted image 20230614195139.png]]
当顶点信息包含的元素少于顶点着色器输入所需要的元素时，**缺少的部分会被0填充，而 w 分量会被1填充**。例如：顶点的 UV 坐标通常是二维向量，只包含 x 和 y 元素。如果输入的语义 TEXCOORD0被声明为 float4类型，那么顶点着色器最终获取到的数据将变成（x，y，0，1）。


> [!NOTE] 数据来源：MeshRender
> 填充到这些语义中的数据由使用该材质的 MeshRender 组件提供，每帧调用 DrawCall 的时候，MeshRender 组件会把它负责渲染的模型数据发送给 UnityShader。

### 顶点着色器输出和片段着色器输入语义
在整个渲染流水线中，顶点着色器最重要的一项任务就是需要输出顶点在裁切空间中的坐标，这样 GPU 就可以知道顶点在屏幕上的栅格化位置以及深度值。在顶点函数中，这个输出参数需要使用 float4类型的 `SV_POSITION` 语义进行填充。

顶点着色器产生的输出值将会在三角形遍历阶段经过插值计算，最终作为像素值输入到片段着色器。换句话说，顶点着色器的输入即为片段着色器的输入。

![[Pasted image 20230614195313.png]]

片段着色器会自动获取顶点着色器输出的裁切空间顶点坐标，**所以片段函数输入的 SV_POSITION 可以省略**。这也解释了为什么有些 Shader 的片段函数中只有输出参数，但是没有输入参数。
**需要特别注意的是，与顶点函数的输入语义不同，`TEXCOORDn` 不再特指模型的 UV 坐标，`COLORn` 也不再特指顶点颜色。它们的使用范围更广，可以用于声明任何符合要求的数据，所以在使用过程中不要被语义的名称欺骗了。**
### 片段着色器输出语义
片段着色器通常只会输出一个 fixed4 类型的颜色信息，输出的值会存储到渲染目标（Render Target）中，输出参数使用 `SV_TARGET` 语义进行填充。

## CG 标准库函数
![[Pasted image 20230615103657.png]]
### lerp 插值

lerp 函数是针对**CG/HLSL**（一种 Shader 语法）中的 lerp 函数
我们先看一下它的函数签名和定义

![image-20220706155641339](image-20220706155641339.png)

从定义来看还是挺简单的，我们主要是理解它有什么作用。另外我们规定，weight 是一个在区间[0,1]的实数，倒不是因为取更大的值之后，这个函数就无定义了，而是因为取更大的值，这个函数就失去了我们构造它的理由，另外，CG 会限制 weight 的值在 0-1 的范围内，超过这个范围会被留在边界 0 或者边界1

这里的 y1 被称为起点，而 y2 被称为终点，lerp 函数就是取值 y1 到 y2 中间的一个值。取多少呢？就由 weight 来控制，当 weight 为 0.5 时，它正好落在起点和终点的中间。为了更加方便理解，我们可以把这个公式换成这种格式

![image-20220706155700754](image-20220706155700754.png)

简单来说，lerp 函数就是在 y1 和 y2 之间过渡，唯一不同的地方就是，y1 和 y2 可以是一个值，也可以是一个函数。比如，我们可以在正弦函数和线性函数之前做过渡，我们先看一下正弦函数

![img](v2-911d01df3d20519bd07b2880d8d91fb2_1440w.jpg)

再看一下最简单的线性函数（恒等映射函数 y=x）

![img](v2-a2374819e0d93f70242bb91131883a47_1440w.jpg)

在它俩之间做过渡，我们只需要写出 lerp (sin (x), x, 0.5)即可。当然，可以调整 weight 参数观察不同的结果。

**weight 为 0.5 时：**

![img](v2-770e7772c6e7a8d99962450eb7672745_1440w.jpg)

**weight 为 0.8 时：**

![img](v2-000da5aa65555a40a9c08537824e69db_1440w.jpg)

**weight 为 0.2 时：**

![img](v2-aa8c4ca591e6dd6f54393eb8b67c634b_1440w.jpg)

是不是非常的直观，所以这就扯到了 Lerp 函数的两种不同的作用。

- 构造新的函数
- 在两个值之间进行过度

### frac 向下取整

frac 函数通过如下代码实现：

```cpp
float frac(float v)
{
  // floor函数返回值会向下取整
  return v - floor(v);
}
```

该函数只有一个参数 v，v 参数的类型不仅可以是 float 类型，也可以是其它 float 向量。通过该函数的实现可知，它**返回标量或每个矢量中各分量的小数部分。**

### clamp 夹具函数

~~~less
float clamp（float x, float a, float b）;
~~~

将 x 固定在[a, b]范围内，

如下所示:

1. 如果 x 小于 a，返回 a;

2. 如果 x 大于 b，返回 b;

3. 否则返回 x。

### clip 剔除

如果给定向量的任何分量或给定标量为负，则终止当前像素输出

对像素剔除：当采样结果小于 0，该像素就会被剔除不会显示到屏幕上

### smoothstep 平滑阶梯

smoothstep 可以用来生成 0 到 1 的平滑过渡值，它也叫平滑阶梯函数。

```cpp
float smoothstep(float t1, float t2, float x)
{
  x = clamp((x - t1) / (t2 - t1), 0.0, 1.0); 
  return x * x * (3 - 2 * x);
}
```

**当 x 等于 t1 时，结果值为 0；**

**当 x 等于 t2 时，结果值为 1；**

**(ps: 值限制在 0~1 之间的原因是因为 clamp 函数的限制)**



### step 函数

step (a, b) 

b >= a 时输出 1

b < a 时输出 0


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

常用的 include 文件包括：
- `HLSLSupport.cginc` **（自动包含）** 用于跨平台着色器编译的 helper 宏和定义。
- `UnityShaderVariables.cginc` **（自动包含）** 常用的全局变量。
- `UnityCG.cginc` 常用的 [helper 函数](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinFunctions.html)。

其他 include 文件：
![[Pasted image 20230614232035.png]]

## HLSLSupport.cginc
**（自动包含）** 用于跨平台着色器编译的 helper 宏和定义。


## UnityShaderVariables.cginc
**（自动包含）** 常用的全局变量。

### 空间变换矩阵
Unity 提供了很多空间变换矩阵，可以直接使用 CG 函数 `mul(Matrix，Vertex)` 将顶点在不同空间之间进行变换。
![[Pasted image 20230614231825.png]]

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


### 摄像机和屏幕

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

### 时间

时间以秒为单位，并由项目 [Time 设置](https://docs.unity3d.com/cn/2021.1/Manual/class-TimeManager.html)中的时间乘数 (Time multiplier) 进行缩放。没有内置变量可用于访问未缩放的时间。


|**名称**|**类型**|**值**|
|:--|:--|:--|
|`_Time`|float4|自关卡加载以来的时间 (t/20, t, t_2, t_3)，用于将着色器中的内容动画化。|
|`_SinTime`|float4|时间正弦：(t/8, t/4, t/2, t)。|
|`_CosTime`|float4|时间余弦：(t/8, t/4, t/2, t)。|
|unity_DeltaTime|float4|增量时间：(dt, 1/dt, smoothDt, 1/smoothDt)。|

### 光照

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


### 光照贴图

|**名称**|**类型**|**值**|
|:--|:--|:--|
|unity_Lightmap|Texture2D|包含光照贴图信息。|
|unity_LightmapST|float4[8]|缩放 UV 信息并转换到正确的范围以对光照贴图纹理进行采样。|

### 雾效和环境光

|**名称**|**类型**|**值**|
|:--|:--|:--|
|unity_AmbientSky|fixed4|梯度环境光照情况下的天空环境光照颜色。|
|unity_AmbientEquator|fixed4|梯度环境光照情况下的赤道环境光照颜色。|
|unity_AmbientGround|fixed4|梯度环境光照情况下的地面环境光照颜色。|
|UNITY_LIGHTMODEL_AMBIENT|fixed4|环境光照颜色（梯度环境情况下的天空颜色）。旧版变量。|
|unity_FogColor|fixed4|雾效颜色。|
|unity_FogParams|float4|用于雾效计算的参数：(density / sqrt(ln(2))、density / ln(2)、–1/(end-start) 和 end/(end-start))。_x_ 对于 Exp2 雾模式很有用；_y_ 对于 Exp 模式很有用，_z_ 和 _w_ 对于 Linear 模式很有用。|

### 其他


|**名称**|**类型**|**值**|
|:--|:--|:--|
|unity_LODFade|float4|使用 [LODGroup](https://docs.unity3d.com/cn/2021.1/Manual/class-LODGroup.html) 时的细节级别淡入淡出。_x_ 为淡入淡出（0 到 1），_y_ 为量化为 16 级的淡入淡出，_z_ 和 _w_ 未使用。|
|`_TextureSampleAdd`|float4|根据所使用的纹理是 Alpha8 格式（值设置为 (1,1,1,0)）还是不是该格式（值设置为 (0,0,0,0)）由 Unity **仅针对 UI** 自动设置。|

## UnityCG.cginc

Unity 着色器中通常会包含此文件。此文件声明大量[内置 helper 函数](https://docs.unity3d.com/cn/2021.1/Manual/SL-BuiltinFunctions.html) （实用函数）和数据结构。

![[Pasted image 20230614222148.png]]

### 顶点变换函数
![[Pasted image 20230614223858.png]]

### 向量变换函数
![[Pasted image 20230614223913.png]]

### 灯光辅助函数
以下函数仅适用于前向渲染路径（ForwardBase 或 ForwardAdd Pass 类型）。
![[Pasted image 20230614224010.png]]
### 视角向量函数
![[Pasted image 20230614224036.png]]

### 其他辅助函数和宏
![[Pasted image 20230614224113.png]]



####### UnityCG.cginc 中的顶点光照 helper 函数

仅当使用每顶点光照着色器（“Vertex”通道类型）时，这些函数才有用。

|**功能：**|**描述：**|
|:--|:--|
|`float3 ShadeVertexLights (float4 vertex, float3 normal)`|根据给定的对象空间位置和法线计算四个每顶点光源和环境光的光照。|

# 灯光阴影
## multi_compile
**在默认状态下，前向渲染只支持一个投射阴影的平行光，如果想要修改默认状态，就需要添加多重编译指令。**
Unity 提供了一系列多重编译指令以编译出不同的 Shader 变体，这些编译指令主要用于处理不同类型的灯光、阴影和灯光贴图，可以使用的编译指令如下：
（1）`multi_compile_fwdbase`：编译 Forward BasePass 中的所有变体，用于处理不同类型的光照贴图，并为主要平行光开启或者关闭阴影。
（2）`multi_compile_fwdadd`：编译 Forward Additional Pass 中的所有变体，用于处理平行光、聚光灯和点光源，以及它们的 cookie 纹理。
（3）`multi_compile_fwdadd_fullshadows`：与 multi_compile_fwdadd 类似，但是增加了灯光投射实时阴影的效果。


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

