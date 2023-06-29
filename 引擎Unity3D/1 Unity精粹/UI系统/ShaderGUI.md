**ShaderGUI 分为两种，一种是 Drawer 一种是 GUI，当然可以把 GUI 集成后用 Drawer 的形式写。**
这里主要说的是  GUI，因为 Drawer 有很多限制，例如修改 RenderType 的时候就很麻烦。
回到 GUI。大部分 GUI 继承自两个类，**`ShaderGUI & BaseShaderGUI`**（可以去 Unity 里翻翻，记得改成 all）。前者是自己造轮子，后者是根据前者的基础上造好了一些轮子（如果是在 Lit 基础上魔改的，用 BaseShaderGUI 方便一些）这里主要是抄作业

另外一点就是参考默认 lit. shader 的时候会发现**CustomEditor**里面是一个**namespace**，所以其实也可以写成**CustomEditor “namespace. name”**的形式

**再有一点，Drawer 和 GUI 是分开的独立的，互不影响。如果需要影响的话，有一个 base. OnGUI 可以达到 Drawer 和 GUI 混用**

## 一、Drawer
Unity 为用户提供了基础类：`MaterialPropertyDrawer`，专门**用于快速实现自定义材质面板**的目的。


### 常用的属性特性

`[Space]` 单行空格
`[Space (5)]` 五行空格
`[Header (name)]`  标题名
`[HideInInSpector]`：在 InSpector 面板隐藏
`[NoScaleOffset]`：隐藏纹理的 Tiling 和 Offset
`[Normal]`：检测是否为 NormalMap
`[HDR]`：开启高动态范围，颜色值可以超过 1
`[Gamma]`
`[MainTexture]` ：将纹理设置为主纹理，默认情况 Unity 会将名为 `_MainTexture` 的纹理设置为主纹理。如果 Shader 中有多个该命令，只有第一个命令会生效 
```cs file:脚本访问主纹理
public Texture texture;
void Start()
{
    Material.mat = GetConponent<Renderer>().material;
    mat.material = texture;
}
```

-  `[MainColor]`：将属性设置为主颜色，默认情况 Unity 会将名为 `_BaseColor` 的纹理设置为主纹理。如果 Shader 中有多个该命令，只有第一个命令会生效 
```cs file:脚本访问主颜色
public Texture texture;
void Start()
{
    Material.mat = GetConponent<Renderer>().material;
    mat.color = color.red;
}
```



### 不同类型的 DrawerClass
![[Pasted image 20230615172023.png]]
在编写 Shader 的时候，DrawerClass 需要写在对应属性之前的“`[]`”中，类别的后缀名称“Drawer”不需要添加，因为 Unity 在编辑的时候会自动添加。


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
KeywordEnum 和 Enum 使用上有些不同，区别在于 KeywordEnum 类似于 if-else，同时在 shader 代码中需要处理


#### Toggle 和 ToggleOff
**将 float 类型的数据以开关的形式在材质属性面板上显示，数值只能设置为 0 或 1，0 为关闭，1 为开启。**
当 Toggle 开启，Shader 关键词会被 Unity 默认设置为 `property name_ON`
当 ToggleOff 开启，Shader 关键词会被 Unity 默认设置为 `property name_OFF`

注意：关键词的所有字母必须大写。

```c file:使用方法
//1、声明Property，格式为：[ToggleOff] VarName("Display", Int) = 0/1
Properties
{
    [Toggle] _EnableColor_Attr("_EnableColorAttr", Int) = 1
    //[ToggleOff] _EnableColor_Attr("_EnableColorAttr", Int) = 1
}


// 2、声明关键词ShaderFeature，格式为: #progma shader_feature VARNAME_ON/OFF
#pragma shader_feature _ENABLECOLOR_ATTR_On
//#pragma shader_feature _ENABLECOLOR_ATTR_OFF


// 3、使用，直接用 #if defined(VARNAME_ON/OFF)
#if defined(_ENABLECOLOR_ATTR_ON)
//#if defined(_ENABLECOLOR_ATTR_OFF)
...
#endif
```

除了使用 Unity 默认的关键词，也可以自定义一个特殊的关键词，例如：
```c
[Toggle (ENABLE_FANCY)] _Fancy ( "Fancy? " ,Float) = 0
```
括号内的名称 ENABLE_FANCY 即为自定义的 Shader 关键词。

#### Enum
枚举（Enum）将 float 类型的数据以下拉列表的形式在材质属性面板上显示，Unity 为用户提供了一些内置的枚举类，例如 BlendMode、CillMode、CompareFunction，举个例子：
![[Pasted image 20221020200242.png|400]]
```c
Properties  
{  
    ......
    [Enum(UnityEngine.Rendering.BlendMode)]  
    _BlendSrc("混合源乘子",int) = 0  
    [Enum(UnityEngine.Rendering.BlendMode)]  
    _BlendOst("混合目标乘子",int) = 0  
    [Enum(UnityEngine.Rendering.BlendOp)]  
    _BlendOp("混合算符",int) = 0  
}
Pass  
{
......
BlendOp [_BlendOp]        //可自定义混合运算符  
Blend [_BlendSrc] [_BlendOst]   //可自定义混合模式
}
```

这是 Unity 内置的所有混合系数的枚举类，默认值为 0 表示选择第一个混合系数，默认值为 1 表示选择第二个混合系数，
以此类推。最终在材质面板上的显示效果如图 11-1 所示，这些选项就是 Shader 中可以使用的所有混合系数。
![[Pasted image 20230615172846.png|450]]

当然，用户也可以自己定义枚举的名称／数值对，但是一个枚举最多只能自定义 7 个名称／数值对。举个例子：
```cs
[Enum (Off, 0，On, 1)] _Zwrite ( "ZWrite", Float) = 0
```
上述例子定义的枚举为“是否深度写入”，括号内为定义的名称／数值对，序号 0 对应 Off，序号 1 对应 On，中间用符号“，”间隔开。默认为序号 0，也就是 Off。

#### KeywordEnum
关键词枚举（KeywordEnum）跟普通的枚举类似，也是将 float 类型的数据以下拉列表的形式在材质属性面板上显示，**只不过关键词枚举会有与之对应的 Shader 关键词**，在 Shader 中通过 `#pragma shader_feature` 或 `#pragma multi_compile ` 指令可以开启或者关闭某一部分 Shader 代码。

Shader 关键词格式为：`property name_enum name`，属性名称+“下画线”+枚举名称，所有英文必须大写，并且最多支持 9 个关键词。举个例子：
```c
[KeywordEnum(None,Add,Multiply)] _Overlay("Overlay mode", Float) = 0
```
括号内的 None，Add，Multiply 是定义的 3 个枚举名称，中间用逗号隔开。默认值为 0，表示默认使用 None。这三个选项所对应的 Shader 关键词分别为：`_OVERLAY_NONE`、`_OVERLAY_ADD` 和 `_OVERLAY_MULTIPLY`。

定义如下：
![[ff2ec5c9a828a57966b774f66e3c89b4_MD5.png]]

使用如下：
![[26594b2b6e750ca51a90aa2b193aeb79_MD5.png]]

#### 在编译指令中定义关键词
定义了 `ToggleDrawer` 或者 `KeywordEnumDrawer` 之后，如果想要正常使用，还需要在编译指令中声明 Shader 关键词。例如，上面定义的 None、Add、Multiply 关键词枚举，在编译指令中的代码如下：

```c
#pragma shader_feature _OVERLAY_NONE _OVERLAY_ADD _OVERLAY_MULTIPLY
```

不同关键词之间需要用空格间隔开。
另外，也可以使用另一种编译指令定义关键词，代码如下：

```c
#pragma multi_compile _OVERLAY_NONE _OVERLAY_ADD _OVERLAY_MULTIPLY
```

**虽然表面上看似通过一个 Shader 文件实现了不同种情况，但是 Unity 会自动将不同情况编译成不同版本的 Shader 文件，这些不同版本的 Shader 文件被称为 Shader 变体（Variants），上述编译指令中包含三个 Shader 变体**。

假设再添加一个指令：
```c
#pragma shader_feature _INVERT_ON
```
本指令包含 Toggle 的关闭与开启两种情况，所以 Unity 最终会编译出 2×3=6 个 Shader 变体。因此在使用大量 shader feature 或 multi compile 指令的时候，无形之中会产生大量的 Shader 变体文件。

**两种不同编译指令之间的区别如下：**
（1）shader_feature：只会为材质使用到的关键词生成变体，没有使用到的关键词不会生成变体，**因此无法在运行的时候通过脚本切换效果**。
（2）multi_compile：会为所有关键词生成变体，因此**可以在运行的时候通过脚本切换效果**。

在 Shader 文件的属性设置面板中可以查看到本 Shader 生成的变体数量，如图 11-2 所示，通过开启“Skip unused shader_features”选项可以只查看使用关键词的变体数量，也可以关闭“Skip unused shader_features”选项查看所有关键词的变体数量。如果需要确定具体的关键词是哪些，可以单击“Show”查看。
![[Pasted image 20230615175748.png]]

#### PowerSlider
指数滑动条（PowerSlider）会将范围型数值的属性显示为非线性对应的滑动条。滑动条上的数值不再按照线性关系进行对应，而是以指数的方式。
这是一个以 3 为指数对应关系的滑动条，其中，括号内的数值为指数
```c
[ PowerSlider (3.0) ]_Brightness ("Brightness", Range (0.01,1)) = 0.1
```
#### IntRange

以滑动条的形式在材质属性面板上显示，Int 类型

### 内置枚举 UI 汇总
ZWriteMode 是没有内置的，实际上也只有 on 和 off 两个状态，所以用 Toogle 其实也可以，这里是直接用 `[Enum(Off, 0, On, 1)]` 这样的写法声明了个新的自定义 Enum 

想要知道 unity 还有哪些 shader 里可以用的 Attributes 可以看看 MaterialPropertyDrawer. cs 这个文件，或者继承 MaterialPropertyDrawer 后自己写一个。

![[Pasted image 20230622155106.jpg]]

```cs file:CustomEnum
public enum CustomEnum
{
    Enum1 = 0,
    Enum2 = 1,
    Enum3 = 2
}
```

```cs
Shader "Mya/EnumTest"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        
        [Header(Custom)]
        [Enum(CustomEnum)]  _CustomEnum ("CustomEnum", Float) = 1

        [Header(Option)]
        [Enum(UnityEngine.Rendering.BlendOp)]  _BlendOp  ("BlendOp", Float) = 0
        [Enum(UnityEngine.Rendering.BlendMode)] _SrcBlend ("SrcBlend", Float) = 1
        [Enum(UnityEngine.Rendering.BlendMode)] _DstBlend ("DstBlend", Float) = 0
        [Enum(Off, 0, On, 1)]_ZWriteMode ("ZWriteMode", float) = 1
        [Enum(UnityEngine.Rendering.CullMode)]_CullMode ("CullMode", float) = 2
        [Enum(UnityEngine.Rendering.CompareFunction)]_ZTestMode ("ZTestMode", Float) = 4
        [Enum(UnityEngine.Rendering.ColorWriteMask)]_ColorMask ("ColorMask", Float) = 15

        [Header(Stencil)]
        [Enum(UnityEngine.Rendering.CompareFunction)]_StencilComp ("Stencil Comparison", Float) = 8
        [IntRange]_StencilWriteMask ("Stencil Write Mask", Range(0,255)) = 255
        [IntRange]_StencilReadMask ("Stencil Read Mask", Range(0,255)) = 255
        [IntRange]_Stencil ("Stencil ID", Range(0,255)) = 0
        [Enum(UnityEngine.Rendering.StencilOp)]_StencilPass ("Stencil Pass", Float) = 0
        [Enum(UnityEngine.Rendering.StencilOp)]_StencilFail ("Stencil Fail", Float) = 0
        [Enum(UnityEngine.Rendering.StencilOp)]_StencilZFail ("Stencil ZFail", Float) = 0

    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            BlendOp [_BlendOp]
            Blend [_SrcBlend] [_DstBlend]
            ZWrite [_ZWriteMode]
            ZTest [_ZTestMode]
            Cull [_CullMode]
            ColorMask [_ColorMask]

            Stencil
            {
                Ref [_Stencil]
                Comp [_StencilComp]
                ReadMask [_StencilReadMask]
                WriteMask [_StencilWriteMask]
                Pass [_StencilPass]
                Fail [_StencilFail]
                ZFail [_StencilZFail]
            }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG. cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos (v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                // sample the texture
                fixed4 col = tex2D(_MainTex, i.uv);
                return col;
            }
            ENDCG
        }
    }
}

```


```cs
using System;
using UnityEngine;
using UnityEditor;

public class XXX : ShaderGUI
{
    // OnGuI 接收的两个参数 ：
    MaterialEditor materialEditor;//当前材质面板
    MaterialProperty[] materialProperty;//当前shader的properties
    Material targetMat;//绘制对象材质球

    // 折叠栏
    private bool m_GUITest = true;

    // 主要实现逻辑
    public override void OnGUI(MaterialEditor materialEditor, MaterialProperty[] properties)
    {
        this.materialEditor = materialEditor; // 当前编辑器
        this.materialProperty = properties;   // 用到的变量
        this.targetMat = materialEditor.target as Material; // 当前材质球

        show(); // 使用下面这个 show函数
    }
    void show()
    {
        #region Shader属性
        // Shader里面的属性，FindProperty 就是从shader里找这个属性
        MaterialProperty _MainTex = FindProperty("_MainTex", materialProperty);
        MaterialProperty _MainColor = FindProperty("_MainColor", materialProperty);
        MaterialProperty _Range = FindProperty("_Range", materialProperty);
        MaterialProperty _Float = FindProperty("_Float", materialProperty);
        MaterialProperty _Red = FindProperty("_Red", materialProperty);
        #endregion

        #region GUI名称
        // GUI名称
        GUIContent mainTex = new GUIContent("主贴图");
        GUIContent mainColor = new GUIContent("主贴图染色");
        GUIContent range = new GUIContent ("测试用 Range");
        GUIContent float1 = new GUIContent("测试用Float");
        GUIContent red = new GUIContent("红色");
        #endregion

        #region GUI折叠
        // 供折叠使用
        m_GUITest = EditorGUILayout.BeginFoldoutHeaderGroup(m_GUITest, "GUI折叠");
        if (m_GUITest)
        {
            // 显示图片用
            materialEditor.TexturePropertySingleLine(mainTex, _MainTex, _MainColor);
                EditorGUI.indentLevel++;
                materialEditor.ShaderProperty(_Range, range);
                materialEditor.ShaderProperty(_Float, float1);
                EditorGUI.indentLevel--;
        }
        #endregion

        // 开关
        EditorGUI.BeginChangeCheck ();
        EditorGUI.showMixedValue = _Red.hasMixedValue;
        var _RED_ON = EditorGUILayout.Toggle(red, _Red.floatValue == 1);
        if (EditorGUI.EndChangeCheck())
            _Red.floatValue = _RED_ON ? 1 : 0;
        EditorGUI.showMixedValue = false;
        // 打开开关之后的效果
        if (_Red.floatValue == 1)
        {
            targetMat.EnableKeyword("_RED_ON");
                EditorGUI.indentLevel++;
                GUILayout.Label("已启用变体 _RED_ON");
                EditorGUI.indentLevel--;
        }
        else
        {
            targetMat.DisableKeyword("_RED_ON");
        }
        
        EditorGUILayout.Space(20);
        // Render Queue
        materialEditor.RenderQueueField(); 
    }
}
```
## 二、CustomEditor GUI

待补充...


我们使用 `CustomEditor` 来扩展材质面板，声明在 Shader 最下方。
> `CustomEditor`：可为着色器定义一个 CustomEditor。如果执行了此操作，Unity 将查找具有此名称并能扩展 ShaderGUI 的类。如果找到，则使用此着色器的所有材质都将使用此 ShaderGUI

我们要在 `Editor` 文件夹创建一个指定的 `CustomShaderGUI.cs` 脚本
```c
CustomEditor "CustomShaderGUI.cs"
```

该类**继承 `ShaderGUI` 并重载 `OnGUI` 方法来扩展材质编辑器**。

ShaderGUI 通过 shader 中的 CustonEditor 关联 UI 脚本，Unity 会调用 OnGUI 来绘制面板， **UI 脚本必须放入 Editor 文件夹中.**

```cs
public class TestGUI : ShaderGUI
{
	//OnGUI接收两个参数：
    MaterialEditor materialEditor;//当前材质面板
    MaterialProperty[] properties;//当前shader的properties

    Material targetMat;//绘制对象材质球
    string[] keyWords;//当前shader keywords
    
    Public override void OnGUI(MaterialEditor materialEditor, MaterialProperty[] properties)
    {
        this.MaterialEditor = materialEditor;
        this.MaterialProperty = properties;
        this.targetMat = materialEditor.target as Material;
        this.keyWords = targetMat.shaderKeywords;
        //关键字是否存在可以判断分支的开启状态
        show();
    }
    
    void show(){
        GUILayout.Label(“Hello Word”);
    }

}
```

![[7c9deaa843b7d9b2b2c3270779012a2c_MD5.png]]

_我们开启自定义 UI 显示以后，默认 UI 将会失效。_  

### 贴图单行显示

`FindProperty` 会根据属性名称 ID 去查找 material Properties 中包含的相应属性，Content 是一个显示 Lable, 它包含了属性名称，属性数值和属性 Tips。最后用 materialEditor 绘制单行贴图 UI

**参数一**：propertyname 材质属性的名称
**参数二**：properties 可用材质属性的数组
**参数三**：propertylsmandatory 如果为 true，则如果没有找到 propertyName 属性，此方法将抛出异常。

```cs
//显示贴图
MaterialProperty _CubeMap= FindProperty(“_CubeMap”, materialProperties, true);

GUIContent content = new GUIContent(_CubeMap.displayName, _CubeMap.textureValue, “cube Map”);//tips 是说明文字，鼠标悬停属性名称时显示
materialEditor.TexturePropertySingleLine(content, _CubeMap);
```

![[2936ec0ef4505833aa6d51546d970848_MD5.png]]

  
添加调色给这张图, 在原有属性下方查找到颜色，GUI 容器还是使用 cubemap 的, 这时 color 就会出现在 cubemap 之后单行显示。

```cs
MaterialProperty tint = FindProperty (“_Color”, materialProperties, true);
//modification
materialEditor.TexturePropertySingleLine(content, _CubeMap, tint);//重载方法
//添加缩放偏移属性显示
//EditorGUI.indentLevel是将绘制的元素进行头部位置偏移
EditorGUI.indentLevel++;
//添加贴图缩放
materialEditor.TextureScaleOffsetProperty(_CubeMap);
EditorGUI.indentLevel--;
//偏移后须将头部位置归位，即便在属性列表末端也需要。
```

![[38d98d0bcb87e266ed15ad6b913f20a0_MD5.png]]

### 法线单行显示，无贴图隐藏滑竿

```cs
MaterialProperty _Normal = FindProperty(“Normal”, materialProperties,true);
MaterialProperty _NormalStrength= null;
//如果有贴图让容器包括强度绘制，没贴图不绘制强度
If(_Normal.textureValue != null)
{
	_NormalStrength = FindProperty(“_NormalStrength”, materialproperties, true);
}

materialEditor.TexturePropertySingleLine(MakeGUIContent(_Normal), _Normal, _NormalStrength );

//自定义绘制 content 方法
GUIContent MakeGUICOntent(MaterialProperty m){
	GUIContent content = new GUIContent(m.displayName, m.textureValue, “”);
Return content;
}
```

![[c15abbbdf66858de8afe9ab275868012_MD5.png]]

![[fa217d21b9cc784271e34dacd5b3197e_MD5.png]]

### 贴图特殊设置提示

在默认 attribute 中我们使用法线贴图时会提示我们当前传入图片是否是法线，我们可以借鉴这一功能定义我们自己需要设置的内容作为提示显示出来。

```cs
MaterialProperty _Tex2 = FindProperty("_Tex2", properties);
materialEditor.TextureProperty(_Tex2, "Tex 2");

if (_Tex2 != null && _Tex2.textureValue.wrapMode != TextureWrapMode.Clamp)
{
	setClamp = materialEditor.HelpBoxWithButton(new GUIContent("贴图需要clamp模式"), new GUIContent("设置")); //setClamp : bool
}
```

![[222151b3539e6faf843b96590fd115a5_MD5.gif]]

```cs
//当我们修改状态以后可以对离线资源进行同步设置
if(setClamp){
    setClamp = false;
    string path = AssetDatabase.GetAssetPath(_Tex2.textureValue);
    TextureImporter textureImporter = AssetImporter.GetAtPath(path) as TextureImporter;
    textureImporter.warpMode = TexturWrapMode.Clamp;
    textureImporter.SaveAndReimport();
}
```

![[4365c9eb40c7e9564164559c72ebf64b_MD5.gif]]

### UI 界面变更检查

现在是在 OnGUI 中每帧重复执行所有方法 (并不是每帧重绘制)，我们应当是 material 属性改变以后在执行内部方法赋值 shader。

```cs
void SetKeyWord(string keyword, bool enable)
    {
        if(enable){
            targetMat.EnableKeyword(keyword);
        }
        else
        {
            targetMat.DisableKeyword(keyword);

        }
    }

EditorGUI.BeginChangeCheck();//需要检查的位置之前放置
materialEditor.TexturePropertySingleLine(MakeGUIContent(_Metallic), _Metallic, _Metal);
If(EditorGUI.EndChangeCheck())//有修改返回ture
{
	SetKeyWord(_Metallic.name.ToUpper() + "_ON", _Metallic.textureValue != null);
}
```

![[2318ba91681f552da97bc6736e230faf_MD5.png]]

![[8b085b56ec15cbeeea62fe93ae15f434_MD5.png]]

```cs
//也可以根据 debug 来查看 keyword 是否成功
targetMat.IsKeywordEnabled(string)
```

### 根据条件隐藏显示所属 UI 控件

```cs
//设置列表显示关键字
public enum LAYER_COUNT
{
    _LAYERCOUNT_ONE, _LAYERCOUNT_TWO, _LAYERCOUNT_THREE
}

public LAYER_COUNT LC;
public void SetKeyWorld(LAYER_COUNT settings) {
           switch (settings)
        {
            case LAYER_COUNT._LAYERCOUNT_ONE:
                targetMat.DisableKeyword(LAYER_COUNT._LAYERCOUNT_THREE.ToString());
                targetMat.DisableKeyword(LAYER_COUNT._LAYERCOUNT_TWO.ToString());
                targetMat.EnableKeyword(LAYER_COUNT._LAYERCOUNT_ONE.ToString());
                break;
            case LAYER_COUNT._LAYERCOUNT_TWO:
                targetMat.DisableKeyword (LAYER_COUNT._LAYERCOUNT_ONE.ToString ());
                targetMat.DisableKeyword(LAYER_COUNT._LAYERCOUNT_THREE.ToString());
                targetMat.EnableKeyword(LAYER_COUNT._LAYERCOUNT_TWO.ToString());
                break;
            case LAYER_COUNT._LAYERCOUNT_THREE:
                targetMat.DisableKeyword(LAYER_COUNT._LAYERCOUNT_ONE.ToString());
                targetMat.DisableKeyword(LAYER_COUNT._LAYERCOUNT_TWO.ToString());
                targetMat.EnableKeyword(LAYER_COUNT._LAYERCOUNT_THREE.ToString());
                break;
        }

    }
```

我们将 enum 控件绘制在最顶端，根据修改去赋值 shader

```cs
EditorGUI.BeginChangeCheck();
LC = (LAYER_COUNT)EditorGUILayout.EnumPopup("LayerCount", LC);
if (EditorGUI.EndChangeCheck()) { 
        SetKeyWorld(LC);	
}
```

![[3173a3dd4a1e65e8ddc95c18a3952730_MD5.png]]

![[36a37e9485c5e538c2970bf3d6e26e39_MD5.png]]

因为我们设置了分支关键字，shader 内会根据关键字走相应流程。我们可以将不被使用的流程属性隐藏。

![[be95405ef6aeca6b9db35edddcf0c5a7_MD5.gif]]

![[4a7d3a24518cf0e8840b72e5960bc410_MD5.gif]]

### 折叠组

和上一步的实现类似区别在于使用一个带有折叠判断的控件绘制, 可以使用 FoldoutHeaderGroup 或 Foldut

```cs
isFoldut = EditorGUILayout.BeginFoldoutHeaderGroup(isFoldut, "Group 01");
if (isFoldut)
{
    //TODO
}
EditorGUILayout.EndFoldoutHeaderGroup();
```

![[7449c1a5fcb835adaf7c7845c5dfcb27_MD5.gif]]

### 可调节 min max 的滑动条

节省控件位置或者更直观的表达时使用，中间以 0 为例，左区间是 [minLimit, 0] 右[0, maxLimit]

左右区间是可以被动态修改的

```cs
EditorGUILayout.MinMaxSlider(ref minVal, ref maxVal, minLimit, maxLimit);
```

![[377de902a4d3ad548a715b0159317bd0_MD5.gif]]

### 控件容器 Rect

在界面中每一个控件都可以定制长宽。x，y 0 点在左上角

![[f9b6c3c6b01d719e3f4e0372c2fd727d_MD5.png]]

  

![[328032c1a6abcc378000b8efa977042b_MD5.png]]

如果我们手动设置 rect，那样以后排板将会很痛苦。

```cs
//获取上一个Rect
//Rect eST = GUILayoutUtility.GetLastRect();
Rect e;

if (Event. current. type == EventType. Repaint){
    e = GUILayoutUtility.GetLastRect();
}
```

判断执行事件是为了避免获取失效。

我们用一个 silder 来控制一个 box 的长短，使其 100% 时填充满 inspector 宽。

```cs
slider =  EditorGUILayout.Slider(slider, 0, 1);
GUI.backgroundColor =  Color.green;
GUILayout.Box(new GUIContent(), GUILayout.Width(slider*e.width));
```

![[e8eb2a7285e194b61e01ab9116042ab4_MD5.gif]]


# SimpleShaderGUI（插件）

最近抽空学习并弄了一个通用的 Shader GUI，你可以使用他轻松的组织你的 shader 属性。他非常的方便，并且兼容 Unity 内置的属性样式例如 [Header ()]、[Space]、[Toggle]、[Enum] 等

在介绍如何使用前，先感谢那些大佬无私的奉献，让我少走了很多弯路，参考已放在文章最后。

## 目录

1.  URP Shader 模板
2.  使用 SimpleShaderGUI
3.  折叠属性
4.  切换属性
5.  纹理属性
6.  向量属性
7.  范围属性
8.  兼容于扩展
9.  工程
10.  参考

## 1. URP Shader 模板

众所周知，目前 Unity 没有提供创建 URP Shader 的模板，在编写 URP shader 时需要建一个 UnlitShader，然后在对其进行修改。

我在这里编写了一个 URP Shader 模板，你可以通过右键 Create->Shader->URP Shader 来创建他，该模板具有基础的 URP 格式，并使用了我自定义的 ShaderGUI，你可以通过修改 CustomShaderGUI/Editor/Template/URPShader 来修改这么模板

模板的创建使用了[雨松大佬的方法](https://www.xuanyusong.com/archives/3732)

![[4ee2ec6f75f2474aaadb4e98bdd530d7_MD5.gif]]

## 2. 使用 SimpleShaderGUI

使用时只需要在 Shader 最后添加 ShaderGUI 的引用 Scarecrow. SimpleShaderGUI，之后像使用 Unity 内置的属性绘制一样就可以，下面将说明目前的属性有哪些

![[966717cc548efc2f7b7562bd9fb1dd3b_MD5.png]]

Unity 自带的属性绘制可以参考以下文章

[【Unity Shader】自定义材质面板的小技巧](https://blog.csdn.net/candycat1992/article/details/51417965) [喵喵 Mya：Shader 面板上常用的一些内置 Enum](https://zhuanlan.zhihu.com/p/93194054)

在使用该 ShaderGUI 时，如果你的属性中包含以下俩个属性_SrcBlend、_DstBlend。将会自动在材质顶部生成不透明和半透明的切换按钮

![[94cc68aa35c11c82d749ca6a7e5b802e_MD5.png]]

## 3. 折叠属性

折叠页使用了 World 标签的形式，使用这种方式可以轻松的制作和管理嵌套折叠页 (之前还考虑使用标签语言的方式，但发现太麻烦了就弃用了...)

![[861d61308f18ef896f6cf942f61f3bea_MD5.jpg]]

```
//foldoutLevel      折叠页等级，折叠页最低等级为1级(默认1级折叠页)
        //foldoutStyle      折叠页外观样式(默认第一种)，目前有3种 1 大折叠页样式， 2 中折叠页样式, 3 小折叠页样式
        //foldoutToggleDraw 折叠页 复选框是否绘制， 0 不绘制 , 1绘制 
        //foldoutOpen       折叠页初始展开状态，    0 折叠， 1展开
        //showList          填写任意数量的选项，当其中一个选项被选中时，该控件会被渲染
        public FoldoutDrawer(float foldoutLevel = 1, float foldoutStyle = 1, float foldoutToggleDraw = 0, float foldoutOpen = 1, params string[] showList)
```

1.  **foldoutLevel 折叠页等级:** 就像 World 一样，你可以选择折叠页的等级，级别低 (数字大) 的就会被嵌套在级别高的折叠页中
2.  **foldoutStyle 折叠页外观样式:** 目前一共有三种样式，通过 1~3 进行选择
3.  **foldoutToggleDraw 是否绘制复选框:** 控制复选框是否被绘制 0 不绘制， 1 绘制
4.  **foldoutOpen 折叠页初始展开状态:** 0 折叠， 1 展开
5.  **showList 显示项列表:** 该属性配合切换属性进行使用，当列表中任意一个选项被选中时该折叠页 (以及折叠页里的属性) 将会被绘制，否则将不绘制

![[08a4d7a0bd3222e123ebb856cc6a6809_MD5.jpg]]

**特别需要注意的是折叠页显示的名字必须以_Foldout 结尾，他只起表示作用，属性为 float。属性的初始值 0 为禁用折叠页，1 为启用折叠页**

当勾选复选框时，将会对材质设置关键字 **大写属性名_ON**，你可以使用他进行一些操作，例如

![[5e80ffa77b2beccde834b0980d821620_MD5.webp]]

showList 将会和切换属性一起说明

**3.1 跳出折叠页**

如果你想将内容跳出当前折叠页，你可以使用 [Foldout_Out]

```
public Foldout_Out(float foldoutLevel = 1)
```

foldoutLevel 跳出折叠页等级: 比如你的属性在 3 级折叠页中，你可以选择跳到 2 级或者 1 级，例如将颜色属性跳出 2 级折叠页

![[bec36c29d34892780d70ed0b0415aeca_MD5.gif]]

**4. 切换属性**

切换属性它可以控制你指定的属性显示或隐藏，他与折叠页不同，折叠页只是把属性折叠起来。在介绍切换属性前，先了解一下他的工作原理

切换属性一共有两种控件，一个是复选框 [Toggle_Switch]、另一个是菜单栏 [Enum_Switch]。我们有一个选项池，用来存储被选中的选项。当复选框被勾选、或者菜单栏某个选项被选中都会向选项池里添加该选项。其他属性需要输入他显示的选项列表，当他显示列表中的选项至少有一个存在选项池里，该属性将会被显示出来。如下图

![[022bd850241f5576f169c45ec1a125bd_MD5.jpg]]

如上图，颜色 1 属性会不显示、颜色 2 属性将会显示出来，接下来来了解下切换控件吧

**4.1 复选框切换**

他和 Unity 的 [Toggle] 一样，只是名称需要换成[Toggle_Switch]

![[f400abc347da3eeed1f7287d0abfa507_MD5.jpg]]

![[58a55d8637634ddee7f5895bd735bd80_MD5.jpg]]

选中时会设置 **大写属性名_ON** 的关键字，如上就会设置 TOGGLE1_ON

**4.2 菜单栏切换**

他和 Unity 的 [Enum] 一样 (只是目前不支持直接输入枚举), 需要名称换成[Enum_Switch]

你需要对他进行传参 (任意多)，该参数表示菜单栏中的选项

![[2397a234baa7ddba9b513f3ce0ff4ee6_MD5.jpg]]

![[0c52f1ddc30f18653044fc7f38a173b6_MD5.jpg]]

选中会设置 **大写属性名_大写选项名** 的关键字，如上就会设置 **_ENUM1_ENUM2**

**4.3 显示控件**

对于一般的属性使用 [Switch] 来进行属性的显示切换，但对于已经使用绘制的属性来说 (例如折叠页、纹理、向量等) 在属性最后面的 showList 就是他的显示列表

下面来看下 [Switch] 的使用

![[2588712904f9b7dde277f69bcdac2f11_MD5.jpg]]

![[f5723f5a37a5868d4809865a6cec3e94_MD5.gif]]

但是对于折叠页那样的属性，已经使用了 [Foldout] 绘制，所以他不能使用 [Switch] 来进行切换。不过我在参数的最后面留有了显示列表的接口 (showList) 提供使用，其他属性也是一样的。当折叠页不显示时，他里面的所有东西也会不显示。当参数为空时他将一直显示

![[003dbfe6e18e6ec002893d23e93cefdf_MD5.jpg]]

![[3bb56a0296d6ac690eadedde37ae03d6_MD5.webp]]

他的自由度非常高，你甚至可以通过以下方式来控制使用纹理的数量

![[194e41d1ba0786f1d6b98c521bb51f42_MD5.jpg]]

![[863ea8cc16362472b5dbfe3bfff01a55_MD5.gif]]

**4.4 注意!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!**

**在使用该切换控件时有两点需要特别注意，当然你可能不知道意味着什么，当你出了问题时会想起来的**

1.  **切换控件一定要在使用该选项的属性前面 (在属性列表中把他放在前面)，因为只有记录了该控件的选项你才能使用该选项**
2.  **切换控件一定要比使用该选项的属性优先显示，出于某些骚操作，你可能会将切换控件和显示属性放在两个不同的折叠页中，当切换控件被折叠时，依然不会记录选中的选项**

**一般是不会有问题的，这是考虑某些极端的骚操作所留下来的问题，当你遇到问题时，看看是不是犯了这两个错误**

**5. 纹理属性**

这里提供了一个单行纹理的显示方式，并且纹理后面可以选择跟一个属性

```
//addProName    要在纹理后面绘制属性的名字
//showList      填写任意数量的选项，当其中一个选项被选中时，该控件会被渲染
public TexDrawer(string addProName, params string[] showList)
```

*   addProName 纹理后面要显示属性的名字
*   showList 显示选项列表，当任意一个被选中时，该属性将会显示

![[38abc1f9fd524893e995b07175ce09dc_MD5.jpg]]

![[02164397a33b7d0125089faadd98b224_MD5.jpg]]

使用 [NoScaleOffset] 就可以不显示缩放属性

**6. 向量属性**

这里把[喵爷控制方向的属性整合 (搬(抄)) 了过来](https://zhuanlan.zhihu.com/p/97256929)

```
//showList      填写任意数量的选项，当其中一个选项被选中时，该控件会被渲染
public Vector3Drawer (params string[] showList)
```

![[b19a96b37f21a1096242011f17827f3e_MD5.png]]

![[01ae48d06ea268ec35ef44958e421614_MD5.gif]]

使用时你需要选中一个物体，然后再点击 Set。他将会设置一个世界空间下的向量

需要注意的是如果你想使用该向量来计算光照，你应该使用该向量的相反数

![[917748b8a51dea55179177c0ad0419a9_MD5.jpg]]

**7. 范围属性**

该属性会生成一个范围的滑块控件，在你向指定某一个范围时会很有用

```
//showList      填写任意数量的选项，当其中一个选项被选中时，该控件会被渲染
public RangeDrawer(params string[] showList)
```

![[610a7492671d8d8035e68a1c2abbdfc0_MD5.png]]

![[cd5e00cc0c3ee6b03037267428f5e995_MD5.gif]]

**8. 兼容于扩展**

该 ShaderGUI 使用的是 Unity 2020.2.3f1c1 制作的，其他版本没有测试，在制作时发现 SceneView. onSceneGUIDelegate 在新版将被弃用，所以使用的是 SceneView. duringSceneGui。如果你在旧版的 unity 中使用报这个错误，你可以将其替换回来。

如果你想要拓展自己的属性绘制方法，直接继承 MaterialPropertyDrawer 就好。并不会造成冲突。如果想使用切换控件来控制自己属性的显示，你可以参考 PropertyGUI_Texture. cs 来看如何使用他

如果你想要学习这方面的知识，你可以直接查看代码，里面的注释我已经写的非常详细

**9. 工程**

*   暂时放在了网盘里，注意这里不会及时更新，建议去 github 下载最新版本

链接：[https://pan.baidu.com/s/17-vVMD4tY8x554T8NumquQ](https://pan.baidu.com/s/17-vVMD4tY8x554T8NumquQ)

提取码：7cyz

*   工程更新到了我的 Github

[https://github.com/Straw1997/UnityCustomShaderGUI](https://github.com/Straw1997/UnityCustomShaderGUI)

*   unitypackage 下载地址

[Releases · Straw1997/UnityCustomShaderGUI](https://github.com/Straw1997/UnityCustomShaderGUI/releases)

**10. 参考**

*   [喵刀 Hime：LWGUI：不写一行 GUI 自定义 Unity ShaderGUI](https://zhuanlan.zhihu.com/p/129289103)
*   [unity3d-jp/UnityChanToonShaderVer2_Project](https://github.com/unity3d-jp/UnityChanToonShaderVer2_Project)
*   [喵喵 Mya：[自定义 shader 面板] 在 SceneView 中绘制一个控制灯光方向的操纵杆]( https://zhuanlan.zhihu.com/p/97256929 )
*   [喵喵 Mya：Shader 面板上常用的一些内置 Enum](https://zhuanlan.zhihu.com/p/93194054)
*   [Unity Shader GUI 学习](https://blog.csdn.net/enk_2/article/details/109236874)
*   [Unity3D 研究院编辑器之创建 Lua 脚本模板（十六） | 雨松 MOMO 程序研究院](https://www.xuanyusong.com/archives/3732)