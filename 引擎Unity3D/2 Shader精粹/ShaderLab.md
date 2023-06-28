[Unity小白的TA之路-Shader开发|图形渲染管线|URP|性能优化|图形渲染|PostProcessing (91maketop.github.io)](https://91maketop.github.io/ta/#/README)
# 约定
## Unity 中的坐标系


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

稍后在着色器的固定函数部分中，可使用括在方括号中的属性名称来访问属性值：`[_name]`。例如，可通过声明两个整数属性（例如`_SrcBlend` 和`_DstBlend`）来使混合模式由材质属性驱动，然后让 Blend 命令使用它们：`Blend [_SrcBlend] [_DstBlend]`

`Properties` 代码块中的着色器参数被序列化为[材质](https://91maketop.github.io/ta/#/Materials.html)数据。[着色器程序](https://91maketop.github.io/ta/#/SL-ShaderPrograms.html)实际上可以有更多参数（如矩阵、矢量和浮点数），这些参数在运行时从代码中在材质上设置，但如果它们不是 Properties 代码块的一部分，则不会保存它们的值。这对于完全由脚本代码驱动的值最有用（使用 [Material.SetFloat](https://91maketop.github.io/ta/#/../ScriptReference/Material.SetFloat.html) 和类似函数）。



- `[PerRendererData]` - indicates that a property will be coming from per-renderer data in the form of a [MaterialPropertyBlock](https://91maketop.github.io/ta/#/../ScriptReference/MaterialPropertyBlock.html). Material inspector shows these properties as read-only.
-  -

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
|`_WorldSpaceLightPos0` |float4|方向光：（世界空间方向，0）。其他光源：（世界空间位置，1）。 |
|unity_WorldToLight（在 AutoLight.cginc 中声明）|float4x4|世界/光源矩阵。用于对剪影和衰减纹理进行采样。 |
|unity_4LightPosX0、unity_4LightPosY0、unity_4LightPosZ0|float4|（仅限 ForwardBase 通道）前四个非重要点光源的世界空间位置。 |
|unity_4LightAtten0| float4      | （仅限 ForwardBase 通道）前四个非重要点光源的衰减因子。    |
|unity_LightColor|half4[4]| （仅限 ForwardBase 通道）前四个非重要点光源的颜色。        |
|unity_WorldToShadow| float4x4[4] | 世界/阴影矩阵。聚光灯的一个矩阵，方向光级联最多有四个矩阵。  |


延迟着色和延迟光照，在光照通道着色器中使用（全部在 UnityDeferredLibrary.cginc 中声明）：

|**名称**|**类型**|**值**|
|:--|:--|:--|
|`_LightColor`|float4|光源颜色。|
|unity_WorldToLight|float4x4|世界/光源矩阵。用于对剪影和衰减纹理进行采样。|
|unity_WorldToShadow|float4x4[4] |世界/阴影矩阵。聚光灯的一个矩阵，方向光级联最多有四个矩阵。|

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



# 变体
![[Pasted image 20230628192002.png]]
能否写一个 All in One 的 Shader？
有三种方式，根据具体需求选择：
1. 静态分支 `#if`
2. 动态分支 `if`
3. 着色器变体 `#pragma`

## 静态分支 `#if`
![[Pasted image 20230628192227.png|500]]

**原理**：着色器**编译时**选择代码分支

**选择**：编译时，能够确定 Shader 执行的条件

- 静态分支
    - 使用 `#define` 定义激活分支
    - 使用 `#if` 、 `#elif` 、 `#else` 和 `#endif` 预处理程序指令来创建静态分支
    - 让 shader 代码执行其中一个分支
    - 编译器会裁剪未激活代码分支，只会将执行的部分编译

注意：静态分支仅在手写代码可用，不能在 Shader Graph 中创建静念分文。

## 动态分支 `if`
![[Pasted image 20230628192654.png|300]]
原理：着色器**运行时**选择代码分支

选择：运行时，是否有可能动态选择分支? 

动态分支用法:
- 在手写代码中，使用 if 语句来执行分支
- 在 Shader Graph 中，使用 Branch 节点

动态分支的优点（相对于着色器变体方式)：
- 可以动态选择分支
- 不会造成代码膨胀

动态分支缺点（相对于着色器变体方式):·会导致运行时性能损失

## 着色器变体 `#pragma`
静态分支的加强版