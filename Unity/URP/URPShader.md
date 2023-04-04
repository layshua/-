[Acshy：【Unity Shader】在URP里写Shader（一）：介绍-从Built-In到URP](https://zhuanlan.zhihu.com/p/336428407)
[Acshy：【Unity Shader】在URP里写Shader（二）：从一个Unlit Shader开始](https://zhuanlan.zhihu.com/p/336508199)
[Acshy：【Unity Shader】在URP里写Shader（三）：URP简单光照Shader](https://zhuanlan.zhihu.com/p/336670858)
# 代码框架
```c
Shader "Unlit/NewUnlitShader"  
{  
    Properties  
    {  
        _BaseMap ("Base Texture", 2D) = "white" {}  
        _BaseColor ("Base Color", Color) = (1,1,1,1)  
          
        [Header(Specular)]  
        _SpecularExp ("SpecularExp", float) = 1  
        _SpecularScale ("SpecularScale", float) = 1  
    }  
    SubShader  
    {  
        Tags   
{  
            "RenderType"="Opaque"  
            "Queue" = "Geometry"  
            "RenderPipeline" = "UniversalPipeLine"    
}  
          
        HLSLINCLUDE  
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"  
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"  
        CBUFFER_START(UnityPerMaterial)  
        float4 _BaseMap_ST;  
        float4 _BaseColor;  
          
        float _SpecularExp;  
        float _SpecularScale;  
        CBUFFER_END  
          
        ENDHLSL  
                Pass  
        {  
            Tags{"LightMode" = "UniversalForward"}    
              
            HLSLPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
              
            struct appdata  
            {  
                float4 vertex : POSITION;  
                float2 uv : TEXCOORD0;  
                float3 normal : NORMAL;  
                float4 tangent : TANGENT;  
                float4 vertexColor  : COLOR;  
            };  
  
            struct v2f  
            {  
                float4 pos           : SV_POSITION;  
                float2 uv            : TEXCOORD0;  
                float3 normal        : TEXCOORD1;  
                float3 tangent       : TEXCOORD2;  
                float3 bitangent     : TEXCOORD3;  
                float3 worldPosition : TEXCOORD4;  
                float3 localPosition : TEXCOORD5;  
                float3 localNormal   : TEXCOORD6;  
                float4 vertexColor   : TEXCOORD7;  
            };  
              
            TEXTURE2D(_BaseMap);     
            SAMPLER(sampler_BaseMap);  
              
            v2f vert (appdata v)  
            {  
                v2f o;  
                // 局部空间的顶点坐标转换到裁剪空间、世界空间  
                VertexPositionInputs positionInputs = GetVertexPositionInputs(v.vertex);  
                o.pos = positionInputs.positionCS;  
                o.worldPosition = positionInputs.positionWS;  
                o.localPosition = v.vertex.xyz;  
                  
                o.uv = TRANSFORM_TEX(v.uv, _BaseMap);  
                  
                // 局部空间的法线、切线、副切线转到世界空间，并return tbn  
                VertexNormalInputs normalInputs =GetVertexNormalInputs(v.normal, v.tangent);  
                o.normal = normalInputs.normalWS;  
                o.tangent = normalInputs.tangentWS;  
                o.bitangent = normalInputs.bitangentWS;  
  
                o.localPosition = v.vertex.xyz;  
                o.localNormal = v.normal;  
                o.vertexColor = v.vertexColor;  
  
                return o;  
  
            }  
              
            float4 frag (v2f i) : SV_Target  
            {  
                float3 world_normal = normalize(i.normal);  
                  
                Light light = GetMainLight();  
                float3 world_lightDir = normalize(TransformObjectToWorldDir(light.direction));;  
                float3 world_viewPos = normalize(GetCameraPositionWS() - i.worldPosition);  
  
                float4 albedo= SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, i.uv);  
                  
                //Ambient  
                //float4 Ambient                                //Diffuse  
                float Diffuse = dot(world_normal,world_lightDir) * 0.5 + 0.5;  
                  
                //Specular  
                float3 Half_Vector = normalize(world_viewPos + world_lightDir);  
                float4 Specular =  pow(saturate(dot(world_normal, Half_Vector)),_SpecularExp) * _SpecularScale ;  
                  
                float4 FinalColor = Diffuse * albedo + Specular;  
                  
                return FinalColor;  
            }  
            ENDHLSL  
        }  
    }  
}
```
# 从Built-In到URP
## Unity的渲染管线

如果你和我一样从冯乐乐《Unity Shader入门精要》开始接触Unity Shader相关的知识，那么最为熟悉的就是Unity的Built-In渲染管线了。

在Unity 2018中，引进了SRP（Scriptable Render Pipeline），让开发者可以依照自己的需求配置渲染管线的某些环节。并在SRP的基础上，提供了两个构建好渲染管线模板，URP（Universal Render Pipeline）和HDRP（High Definition Render Pipeline）。

[几个管线的关系](https://link.zhihu.com/?target=https%3A//blog.csdn.net/weixin_41622043/article/details/107623694)如下图：

![[67837baa4690c91f27abf7fe7df4556f_MD5.webp]]

Build-in,SRP,URP与HDRP

## URP的组成

[URP的官方文档](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Packages/com.unity.render-pipelines.universal%407.4/manual/index.html)把URP的组成分成了URP Asset，Renderer Asset，Shaders和相机几个部分。

URP Asset和Render Asset都是用于配置相关参数的Scriptable Object。配置包括是否启用深度缓存、模板缓存、图像质量、渲染器、光照、阴影质量等参数。其中在Forward Renderer Asset中能够启用Render Feature功能，实现之前built-in管线中一些多Pass的效果，有兴趣同学的可以参照这个[外描边实现的样例](https://link.zhihu.com/?target=https%3A//learn.unity.com/tutorial/custom-render-passes-with-urp%235ddc3e79edbc2a002063f672)看看。

URP Shaders默认提供PBR光照（Lit）、简单兰伯特光照（SimpleLit）和无光照（Unlit）等常用Shader，最重要的要属引入了[Shader Graph这一图形化节点材质编辑器](https://link.zhihu.com/?target=https%3A//unity.com/shader-graph)，如果之前使用过Shader Forge 或者ASE的话，很快就能上手。如果这些还不能满足你的需求，需要在URP管线中写ShaderLab，那么可以参考下一篇文章。

## URP的渲染流程

URP采用的是前向渲染管线，对每一帧，URP会执行如下图的[Rendering Loop](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Packages/com.unity.render-pipelines.universal%407.3/manual/rendering-in-universalrp.html)。一个Rendering Loo中对场景中的每个相机执行相机的Camera Loop。

![[1e9e6087b9fe80db83c3dfeb5caa31f2_MD5.webp]]

>                                                 URP的渲染循环

URP会对场景中所有的相机执行剔除、构建渲染数据、执行渲染操作三个步骤。

-   剔除（Culling）

-   Setup Culling Parameters：可重载函数，设置剔除相关参数
-   Culling：内部函数，按照剔除参数执行剔除操作

-   构建渲染数据（Building Rendering Data）

-   Build Rendering Data：内部函数，从URP Asset中读取相关的渲染参数，为当前平台构建渲染数据

-   渲染（Rendering）

-   Setup Renderer：设置一个Pass列表，并对各Pass的渲染部分进行排序
-   Execute Renderer：执行渲染器，将结果写入帧缓存

在渲染过程中，管线还提供了BeginFrameRendering，BeginCameraRendering，EndCameraRendering，EndFrameRendering几个回调函数，允许在对应阶段[调用相应的回调函数](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/ScriptReference/Rendering.RenderPipelineManager-beginCameraRendering.html)。

## 从Built-In到URP

URP和之前的Built-In管线有[不少区别](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Packages/com.unity.render-pipelines.universal%407.4/manual/universalrp-builtin-feature-comparison.html)，对于熟悉了Built-In管线中工作的同学有不少需要适应的地方，SRP Batcher和可使用Pass数量的变化是URP重要的特性。

### SRP Batcher

熟悉Batcher的同学都知道，合批操作能减少Draw Call，提升渲染效率。

而[SRP Batcher](https://link.zhihu.com/?target=https%3A//blogs.unity3d.com/2019/02/28/srp-batcher-speed-up-your-rendering/)，虽然也叫Batcher，但是工作原理和静态合批与动态合批都不太一样。SRP Batcher通过ConstantBuffer将对象的基本信息存储在GPU上，从而避免了重复提交数据带来的性能损失。

要想使用SRP Batcher，需要在URP Asset中勾选启用，并且要有对应支持的Shder（默认提供的URP Shader和使用Shader Graph生成的Shader都行）。如果是自定义的ShderLab，则需要将需缓存到CBuffer的变量写到CBUFFER_START（UnityPerMaterial）和CBUFFER_END之间，具体可以看下一篇。

```c
CBUFFER_START(UnityPerMaterial)
    float4 _BaseMap_ST;
    float4 _BaseColor;
CBUFFER_END
```

## 单Pass

URP管线不再支持多个Pass，**准确来说，它不再支持多个渲染Pass**。传统的前向渲染管线采用的是ForwardBase Pass计算主光，ForwardAdd Pass计算其他光照叠加的实现方式。光源越多物体越多调用的Pass数量也就越多。而在URP中采用的是单Pass的光照计算，在一个Pass中遍历光源进行关照计算。

**URP提供的几个默认Shader中包含多个Pass，但是只能有一个渲染Pass**

-   渲染Pass

-   "LightMode"="UniversalForward"
-   最终输出到帧缓存中，只能有一个，也就是我们说的URP只支持一个Pass

-   投影Pass

-   "LightMode"="ShadowCaster"
-   用于计算投影

-   深度Pass

-   “LightMode”=”DepthOnly”
-   如果管线设置了生成深度图，会通过这个Pass渲染

-   其他Pass

-   专门用于烘焙的Pass，专门用于2D渲染的Pass等

在Built-In管线里面，我们可以使用多Pass实现描边、毛发等等效果，但是在URP中就需要改变实现方式了。一般来说，Render Feature可以解决大部分问题，可以通过它实现大部分多Pass的效果。

对于毛发这种Pass数量多的，就可以考虑使用[多物体单Pass的方式实现](https://link.zhihu.com/?target=https%3A//github.com/Acshy/FurShaderUnity)。

### 其他变化

-   少了几个buit-in管线中的效果组件

-   Halo组件，Lens Flare组件，Projector组件

-   不支持Surface Shader
-   默认Shader一些参数有变化

-   半透明材质的混合模式增加mutiply 和addtive选项
-   材质增加了双面渲染选项

-   ……

上面只是比较重要的变化，详细的改变可以[直接看文档](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Packages/com.unity.render-pipelines.universal%407.4/manual/universalrp-builtin-feature-comparison.html)。

## 参考文档

大部分的内容在URP的官方文档里都能看到，我这只是写了一些我觉得比较重要，方便从Buit-in管线中过来的同学参考的内容。

[https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@7.4/manual/index.html](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Packages/com.unity.render-pipelines.universal%407.4/manual/index.html)

这一篇文章以对比Built-In，介绍URP管线为主。没有太多具体实践的操作细节。在接下来的文章中，会讲讲在URP管线中编写ShaderLab和Built-In管线中的区别。

  # 从一个Unlit Shader开始

上一篇我们介绍了URP渲染管线的组成和一些特性。

在刚入门Shader的时候，我们都从一个Unlit Shader开始。所以，这次我们也从一个最简单的Unlit Shader开始，比较一下URP中的Shader和Buit-In管线的Shader有什么区别。

这个示例Shader只实现了一个模型贴图的效果。如果之前有写过Built-In管线的Shader，那下面内容对你来说一定非常容易！

## 最简单的URP Unlit Shader

在Unity的Project中右键创建的Unlit Shader还是一个Built-in管线的Shader（尽管在URP中也能用）。而如果你需要一个同样功能的URP Unlit Shader，它应该长这个样子。

```c
Shader "URPCustom/Unlit"
{
    Properties
    {
        _BaseMap ("Base Texture",2D) = "white"{}
        _BaseColor("Base Color",Color)=(1,1,1,1)
    }
    SubShader
    {
        Tags
        {
            "RenderPipeline"="UniversalPipeline"//这是一个URP Shader！
            "Queue"="Geometry"
            "RenderType"="Opaque"
        }
	    //HLSLINCLUDE~ENDHLSL 要共享的 HLSL 代码
        HLSLINCLUDE
         //CG中核心代码库 #include "UnityCG.cginc"
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
        //除了贴图外，要暴露在Inspector面板上的变量都需要缓存到CBUFFER中
        CBUFFER_START(UnityPerMaterial)
        float4 _BaseMap_ST;
        half4 _BaseColor;
        CBUFFER_END
        ENDHLSL

        Pass
        {
            Tags{"LightMode"="UniversalForward"}//这个Pass最终会输出到颜色缓冲里
			// 此 HLSL 着色器程序自动包含上面的 HLSLINCLUDE 块的内容
            HLSLPROGRAM //CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            struct Attributes//这就是a2v
            {
                float4 positionOS : POSITION;
                float2 uv : TEXCOORD;
            };
            struct Varings//这就是v2f
            {
                float4 positionCS : SV_POSITION;
                float2 uv : TEXCOORD;
            };

            TEXTURE2D(_BaseMap);//在CG中会写成sampler2D _MainTex;
            SAMPLER(sampler_BaseMap);

            Varings vert(Attributes IN)
            {
                Varings OUT;
                //在CG里面，我们这样转换空间坐标 o.vertex = UnityObjectToClipPos(v.vertex);
                //HLSL里可以这样写：o.vertex = TransformObjectToHClip(v.vertex);或者如下：
                VertexPositionInputs positionInputs = GetVertexPositionInputs(IN.positionOS.xyz);
                OUT.positionCS = positionInputs.positionCS;
				
                OUT.uv=TRANSFORM_TEX(IN.uv,_BaseMap);
                return OUT;
            }

            float4 frag(Varings IN):SV_Target
            {
                //在CG里，我们这样对贴图采样 fixed4 col = tex2D(_MainTex, i.uv);
                half4 baseMap = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv);                
                return baseMap * _BaseColor;
            }
            ENDHLSL  //ENDCG          
        }
    }
}
```

为了方便对比，我在注释里把URP中写Shader和Buit-In管线里的写Shader不一样的部分都做了简单的标注。这里十分建议把这段代码和在Unity中新建的Unlit Shader对照着看，能够更清楚二者之前有什么变化。

那么接下来，我们具体来讲一讲这些区别吧！

## **Tag打上URP的印记**

虽然有一些Bulit-In的自定义Shader在URP中也能使用。但是如果是专门为URP写Shader的话，就需要打上对应的Tags让渲染管线识别到。

在SubShader中需要声明这个Shader使用的是URP。

```c
SubShader
{
 Tags {
     "RenderPipeline"="UniversalPipeline"
    ……
  }
}
```

在上一章提到，**URP中实际会包含多个Pass，但是只有一个渲染Pass，要让管线能区分这些不同，就需要给这些Pass给Tags进行标记**。

```c
Pass
{
    Tags{"LightMode"="UniversalForward"}//说明这个Pass是一个渲染Pass
    ……
}
```

## **CG VS HlSL**

[ShaderLab是一个包含Shader语言的代码声明块](https://link.zhihu.com/?target=https%3A//forum.unity.com/threads/is-shaderlab-a-specific-language-for-unity-shaders.146930/)。在Built-In管线中我们常使用CG语言来写Shader，但是在URP中，一般使用HLSL。CG语言和HLSL语言都是C风格的语言，写法上没有太大差别。

由于使用语言不同，用于包含CG语言的**CGPROGRAM**和**ENDCG**需要改成包含HLSL的**HLSLPROGRAM**和**ENDHLSL**。

```c
HLSLPROGRAM
    ……
ENDHLSL
```

## **头文件**

使用的底层代码也发生了变化，默认包含的头文件需要从CG的 **#include "UnityCG.cginc"改为**
```c
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
```
除了核心的工具库外，其他如光照、阴影等要包含的头文件也需要修改。
## 常用函数
### 空间位置
使用了不同的头文件后，自然一些使用各种工具函数也不一样，比如顶点着色器中转换空间的方法UnityObjectToClipPos(v.vertex);在URP中就使用`GetVertexPositionInputs(IN.positionOS.xyz)`;来获取一个存储了各个空间Position坐标的结构体，再从中获取裁剪空间中的位置坐标。

```c
VertexPositionInputs positionInputs = GetVertexPositionInputs(IN.positionOS.xyz);
OUT.positionCS = positionInputs.positionCS;
```

源码：
```c
VertexPositionInputs GetVertexPositionInputs(float3 positionOS)
{
    VertexPositionInputs input;
    input.positionWS = TransformObjectToWorld(positionOS);
    input.positionVS = TransformWorldToView(input.positionWS);
    input.positionCS = TransformWorldToHClip(input.positionWS);

    float4 ndc = input.positionCS * 0.5f;
    input.positionNDC.xy = float2(ndc.x, ndc.y * _ProjectionParams.x) + ndc.w;
    input.positionNDC.zw = input.positionCS.zw;

    return input;
}
```
实际上它在函数体内部帮我们做了很多转换，我们直接使用结构体中的对应参数名（positionInputs.positionWS）就能访问到其中的数据，这点实在方便许多。

1.  positionWS，在世界空间中的位置
2.  positionVS，视图空间中的位置
3.  positionCS，裁剪空间中的位置
4.  positionNDC，标准化设备坐标中的位置
### 法线
**处理Normal的函数**可能也是我们比较关心的，同样的Core.hlsl库中为我们准备了一个函数GetVertexNormalInputs

```csharp
VertexNormalInputs GetVertexNormalInputs(float3 normalOS, float4 tangentOS)
{
    VertexNormalInputs tbn;

    // mikkts space compliant. only normalize when extracting normal at frag.
    real sign = tangentOS.w * GetOddNegativeScale();
    tbn.normalWS = TransformObjectToWorldNormal(normalOS);
    tbn.tangentWS = TransformObjectToWorldDir(tangentOS.xyz);
    tbn.bitangentWS = cross(tbn.normalWS, tbn.tangentWS) * sign;
    return tbn;
}
```

用于将对象空间的法线、切线转到世界空间：

1.  normalWS，在世界空间中的法线向量
2.  tangentWS，在世界空间中的切线向量
3.  bitangentWS，在世界空间中的副切线向量

**法线贴图的处理函数** 这也是我们常用的，这里我找到一个比较好用的函数SampleNormal（） 贴过来给大家看一下：

```csharp
half3 SampleNormal(float2 uv, TEXTURE2D_PARAM(bumpMap, sampler_bumpMap), half scale = 1.0h)
{
#ifdef _NORMALMAP
    half4 n = SAMPLE_TEXTURE2D(bumpMap, sampler_bumpMap, uv);
    #if BUMP_SCALE_NOT_SUPPORTED
        return UnpackNormal(n);
    #else
        return UnpackNormalScale(n, scale);
    #endif
#else
    return half3(0.0h, 0.0h, 1.0h);
#endif
}
```
### 纹理贴图
在CG中我们一般使用sampler2D _Texture来采样贴图。在HLSL中，贴图采样需要声明为Sampler。具体的采样方法也从原来的tex2D(Texture tex,float uv)；变为`SAMPLE_TEXTURE2D(Texture tex, Sampler sampler,float uv)`;

```c
TEXTURE2D(_BaseMap);
SAMPLER(sampler_BaseMap);
……
float4 frag(Varings IN):SV_Target
{
    ……
    half4 baseMap = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv);     
    ……
}
```

```csharp
TEXTURE2D(textureName);
SAMPLER(sampler_textureName);
```

```csharp
half4 baseMap = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv);
```

## **CBUFFER**

为了支持SRP Batcher（具体看前一篇），Shader中要将所有暴露出的参数（贴图除外）给包含到**CBUFFER_START(UnityPerMaterial)**与**CBUFFER_END**之间。并且为了保证之后的每个Pass都能拥有一样的CBUFFER，这一段代码需要写在SubShader之内，其它Pass之前。

```c
CBUFFER_START(UnityPerMaterial)
    float4 _BaseMap_ST;
    float _Color;
CBUFFER_END
```

## **命名习惯上的差别**

这个只是个习惯问题，但是以免大家在看其它HLSL Shader的时候浪费时间这里提一下。

在《UnityShader 入门精要》中一般对从应用传入顶点着色器的输出结构体和从顶点传入片元着色器的输出结构体命名为a2v和v2f。在HLSL中这两个结构体一般被命名成Attributes和Varying。

对于一些三维坐标、向量的命名，在HLSL中一般在命名结尾使用空间名字缩写表示其所属的空间。比如positionOS就是对象坐标戏（Object Space）下的位置坐标，NormalWS就是世界坐标系下的法线坐标。

## 参考文档

[https://cyangamedev.wordpress.com/2020/06/05/urp-shader-code/](https://link.zhihu.com/?target=https%3A//cyangamedev.wordpress.com/2020/06/05/urp-shader-code/)

[urp管线的自学hlsl之路 第二篇 从写一个unlit shader开始](https://link.zhihu.com/?target=https%3A//www.bilibili.com/read/cv6383390)

# URP简单光照Shader
在上一篇我们通过一个Unlit Shader熟悉了URP里的Shader怎么写。

那么在这一篇，我们写一个简单的Lambert+BlinPhong的简单光照Shader，来熟悉一下URP中和光照相关的API。

## Lighiting库函数

在URP中所有和光照相关的函数都在Lighting.hlsl这个文件中，包含了获取光照信息计算简单光照乃至计算PBR的相关功能函数。

我们需要在Shader中引入这个库：

```less
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
```

## 相关变量获取

从公式中可以知道，要完成光照的计算，需要在同一坐标空间下的法线、光线方向、视线方向（用于求半角向量）以及光照颜色、高光颜色、表面平滑度等。

我们先只计算主光源的光照，在世界坐标系下计算，需要获取世界坐标系下的相关向量。

视线方向（ViewDir）,下面的GetCameraPositionWS()是Lighting.hlsl中获取相机位置的方法：

```less
VertexPositionInputs positionInputs = GetVertexPositionInputs(IN.positionOS.xyz);

OUT.viewDirWS = GetCameraPositionWS() - positionInputs.positionWS; 
```

法线（NormalDir）：

```less
VertexNormalInputs normalInputs = GetVertexNormalInputs(IN.normalOS.xyz);

OUT.normalWS = normalInputs.normalWS;
```

光线方向（LightDir），GetMainLight()方法能获取主光源的颜色、方向、衰减等信息，非常方便：

```c
 Light light = GetMainLight();
 float3 lightDirWS = light.direction;
```

## 计算光照

当一切准备就绪，我们就能进行计算了。虽然公式本身就非常简单，但是我们能通过调用Lighting.hlsl的相关函数让这一切变得更加方便！

```c
half3 diffuse = baseMap.xyz*_BaseColor*LightingLambert(light.color, light.direction, IN.normalWS);
//等同于：
//half3 diffuse = lightColor*saturate(dot(normal, lightDir));

half3 specular = LightingSpecular(light.color, light.direction, normalize(IN.normalWS), normalize(IN.viewDirWS), _SpecularColor, _Smoothness);
// 等同于：
//float3 halfVec = SafeNormalize(float3(lightDir) + float3(viewDir));
//half NdotH = saturate(dot(normal, halfVec));
//half3 specular = lightColor * specular.rgb * pow(NdotH, smoothness);

half3 color=diffuse+specular;
```

渲染效果：

![[7e4016abe7a252395a55cbaee8e1cbed_MD5.webp]]

Lambert+BlinPhong光照

## 处理多个光源

上面我们只处理了主光源，现在我们来对其他光源进行处理。

在[第一篇文章](https://zhuanlan.zhihu.com/p/336428407)中我们提到过，URP使用的单Pass的方式计算多个光源。所以我们需要在一个Pass中遍历所有光源，对每个光源进行上文中的计算。

**方案一：逐顶点计算附加光源**

**处于性能考虑，Unity URP的默认材质中一般对主光源进行逐像素光照，对于其他光源进行逐顶点光照**（我们可以在URP Asset中更改这一设置）。URP封装了在逐顶点漫反射的方法，我们只需要在逐顶点着色器中调用就能计算附加光照的漫反射。该方法将所有附加光的漫反射计算结果进行累加，返回附加光的累加颜色。

```c
half3 vertexLight = VertexLighting(vertexInput.positionWS, normalInput.normalWS);
```

**方案二：逐像素计算每个附加光源**

如果希望每个光源都能够进行逐像素的漫反射和高光反射计算，那么我们就需要在片元着色器中遍历每一个光源，并进行和上面一样的光照计算。

```c
uint pixelLightCount = GetAdditionalLightsCount();
for (uint lightIndex = 0; lightIndex < pixelLightCount; ++lightIndex)
{
    Light light = GetAdditionalLight(lightIndex, IN.positionWS);
    diffuse += LightingLambert(light.color, light.direction, IN.normalWS);
    specular += LightingSpecular(light.color, light.direction, normalize(IN.normalWS), normalize(IN.viewDirWS), _SpecularColor, _Smoothness);
}
```

GetAdditionalLightsCount()能够获取到影响这个片段的附加光源数量，但是如果数量超过了URP中设定的附加光照上限，就会返回附加光照上限的数量。

GetAdditionalLight(lightIndex, IN.positionWS);方法会按照index去找到对应的光源，并根据提供的片段世界坐标位置计算光照和阴影衰减，并存储在返回的Light结构体内。

![[a01314f82fb05a7697f65039d08e9fa9_MD5.webp]]

逐像素多光源光照

## 阴影Pass

我们目前只写了一个渲染Pass，并没有处理阴影相关的内容。

一般在Buit-In管线里，我们只需要最后FallBack返回到系统的Diffuse Shader，管线就会去里面找到他处理阴影的Pass。但是在URP中，一个Shader中的所有Pass需要有一致的CBuffer，否则便会打破SRP Batcher，影响效率。

而系统默认SimpleLit的Shader中的CBuffer内容和我的写的并不一致，所以我们需要把它阴影处理的Pass复制一份，并且删掉其中引用的SimpleLitInput.hlsl（相关CBuffer的声明在这里面）。

```c
Pass
        {
            Name "ShadowCaster"
            Tags{"LightMode" = "ShadowCaster"}

            ZWrite On
            ZTest LEqual
            Cull[_Cull]

            HLSLPROGRAM
            // Required to compile gles 2.0 with standard srp library
            #pragma prefer_hlslcc gles
            #pragma exclude_renderers d3d11_9x
            #pragma target 2.0

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature _ALPHATEST_ON
            #pragma shader_feature _GLOSSINESS_FROM_BASE_ALPHA

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing

            #pragma vertex ShadowPassVertex
            #pragma fragment ShadowPassFragment


            //由于这段代码中声明了自己的CBUFFER，与我们需要的不一样，所以我们注释掉他
            //#include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            //它还引入了下面2个hlsl文件
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/SurfaceInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/ShadowCasterPass.hlsl"
            ENDHLSL
        }
```

## 完整Shader

到此为止，我们实现了一个能够处理多光源情况的简单漫反射和镜面反射的Shader。

下面是完整的参考代码：

```c
Shader "Custom/URPSimpleLit"
{
    Properties
    {
        _BaseMap ("Base Texture",2D) = "white"{}
        _BaseColor("Base Color",Color) = (1,1,1,1)
        _SpecularColor("SpecularColor",Color)=(1,1,1,1)
        _Smoothness("Smoothness",float)=10
        _Cutoff("Cutoff",float)=0.5
    }
    SubShader
    {
        Tags
        {
            "RenderPipeline"="UniversalPipeline"
            "Queue"="Geometry"
            "RenderType"="Opaque"
        }

        HLSLINCLUDE
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
        CBUFFER_START(UnityPerMaterial)
        float4 _BaseMap_ST;
        float4 _BaseColor;
        float4 _SpecularColor;
        float _Smoothness;
        float _Cutoff;
        CBUFFER_END
        
        ENDHLSL
    

        Pass
        {
            Name "URPSimpleLit" 
            Tags{"LightMode"="UniversalForward"}

            HLSLPROGRAM            
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

            #pragma vertex vert
            #pragma fragment frag

            struct Attributes
            {
                float4 positionOS : POSITION;
                float4 normalOS : NORMAL;
                float2 uv : TEXCOORD0;
            };
            struct Varings
            {
                float4 positionCS : SV_POSITION;
                float2 uv : TEXCOORD0;
                float3 positionWS : TEXCOORD1;
                float3 viewDirWS : TEXCOORD2;
                float3 normalWS : TEXCOORD3;
            };
            
            TEXTURE2D(_BaseMap);
            SAMPLER(sampler_BaseMap);    

            Varings vert(Attributes IN)
            {
                Varings OUT;
                VertexPositionInputs positionInputs = GetVertexPositionInputs(IN.positionOS.xyz);
                VertexNormalInputs normalInputs = GetVertexNormalInputs(IN.normalOS.xyz);
                OUT.positionCS = positionInputs.positionCS;
                OUT.positionWS = positionInputs.positionWS;
                OUT.viewDirWS = GetCameraPositionWS() - positionInputs.positionWS;
                OUT.normalWS = normalInputs.normalWS;
                OUT.uv=TRANSFORM_TEX(IN.uv,_BaseMap);
                return OUT;
            }
            
            float4 frag(Varings IN):SV_Target
            {
                
                half4 baseMap = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv);      
                //计算主光
                Light light = GetMainLight();
                half3 diffuse = LightingLambert(light.color, light.direction, IN.normalWS);
                half3 specular = LightingSpecular(light.color, light.direction, normalize(IN.normalWS), normalize(IN.viewDirWS), _SpecularColor, _Smoothness);
                //计算附加光照
                uint pixelLightCount = GetAdditionalLightsCount();
                for (uint lightIndex = 0; lightIndex < pixelLightCount; ++lightIndex)
                {
                    Light light = GetAdditionalLight(lightIndex, IN.positionWS);
                    diffuse += LightingLambert(light.color, light.direction, IN.normalWS);
                    specular += LightingSpecular(light.color, light.direction, normalize(IN.normalWS), normalize(IN.viewDirWS), _SpecularColor, _Smoothness);
                }

                half3 color=baseMap.xyz*diffuse*_BaseColor+specular;
                clip(baseMap.a-_Cutoff);
                return float4(color,1);
            }
            ENDHLSL            
        }

        
          Pass
        {
            Name "ShadowCaster"
            Tags{"LightMode" = "ShadowCaster"}

            ZWrite On
            ZTest LEqual
            Cull[_Cull]

            HLSLPROGRAM
            // Required to compile gles 2.0 with standard srp library
            #pragma prefer_hlslcc gles
            #pragma exclude_renderers d3d11_9x
            #pragma target 2.0

            // -------------------------------------
            // Material Keywords
            #pragma shader_feature _ALPHATEST_ON
            #pragma shader_feature _GLOSSINESS_FROM_BASE_ALPHA

            //--------------------------------------
            // GPU Instancing
            #pragma multi_compile_instancing

            #pragma vertex ShadowPassVertex
            #pragma fragment ShadowPassFragment


            //由于这段代码中声明了自己的CBUFFER，与我们需要的不一样，所以我们注释掉他
            //#include "Packages/com.unity.render-pipelines.universal/Shaders/SimpleLitInput.hlsl"
            //它还引入了下面2个hlsl文件
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/SurfaceInput.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/Shaders/ShadowCasterPass.hlsl"
            ENDHLSL
        }

    }
}
```

  

## 参考文章

[Writing Shader Code for the Universal RP](https://link.zhihu.com/?target=https%3A//cyangamedev.wordpress.com/2020/06/05/urp-shader-code/8/)

最好的参考其实就是直接看URP默认提供的Shader和相关工具库，在Unity的Project窗口中，可以在Packages/com.unity.render-pipelines.universal/ShaderLibrary 目录下面找到他们。

[https://github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary](https://link.zhihu.com/?target=https%3A//github.com/Unity-Technologies/Graphics/tree/master/com.unity.render-pipelines.universal/ShaderLibrary)

# URP后处理

今天我们先讲URP的Renderer Feature，**Renderer Feature可让我们向URP Renderer添加额外的渲染通道**，支持我们进行Asset资产配置来重写从而可以自定义渲染的顺序、渲染的对象、材质等等。

## **新建 Renderer Feature**

我们先来新建一个Renderer Feature ，首先你得有一个或者新建一个URP工程，创建Universal Render Pipeline Asset同时会生成一个Universal Render Pipeline Asset_Renderer, 创建完成后我们把Universal Render Pipeline Asset导入到Graphics Settings中。

![[4eb770fec8c0c267b639f45b44e6c403_MD5.webp]]

接着我们点击生成的Renderer，然后Add Renderer Feature点Render Objects，我们就创建好一个Renderer Feature了。

![[f7d886011650d2db06324be865dfc6e3_MD5.webp]]

我们在 Project 窗口中也可以查看，点击渲染器旁边的小三角，可以看到新建的Renderer Feature 已经是我们Renderer渲染器的子集（这里Renderer Feature我们已经重命名为Toon Outline）。

![[6239752624bb6c5dfc9728f2e1c0ecd7_MD5.webp]]

## **Renderer Feature 属性简介**

我们可以看到 Render Objects 的 Renderer Feature 包含一些属性；我们一个个过一下。

![[ca9528223ed01fa6d5366601c5e2e641_MD5.webp]]

**_01_ Name：首先是这个Feature的名字；

_**02**_ **Event (事件)：当Unity执行这个Renderer Feature 的时候，这个事件Event在通用渲染管线中的执行顺序；

**_03_ Filters：这个设置允许我们给这个Renderer Feature 去配置要渲染哪些对象；这里面有两个参数，一个是Queue，一个是Layer Mask。

**Queue：这个Feature选择渲染透明物体还是不透明物体；

**Layer Mask：这个Feature选择渲染哪个图层中的对象；

**_04_ Shader Passes Name：如果shader中的一个pass具有 LightMode Pass 这个标记的话，那我们的Renderer Feature仅处理 LightMode Pass Tag 等于这个Shader Passes Name的Pass。

_**05**_ **Overrides：使用这个Renderer Feature 进行渲染时，这部分的设置可让我们配置某些属性进行重写覆盖。

**Material：渲染对象时，Unity会使用该材质替换分配给它的材质。

**Depth：选择此选项可以指定这个Renderer Feature如何影响或使用深度缓冲区。此选项包含以下各项：

_Write Depth：_写入深度，此选项定义渲染对象时这个Renderer Feature是否更新深度缓冲区。

_Depth Test：_深度测试，决定renderer feature是否渲染object的片元。

**Stencil：选中此复选框后，Renderer将处理模板缓冲区值。

**Camera：选择此选项可让您覆盖以下“摄像机”属性：

_Field of View：_渲染对象时，渲染器功能使用此Field of View（视场），而不是在相机上指定的值。

_Position Offset：_渲染对象时，Renderer Feature将其移动此偏移量。_Restore：_选中此选项，在此Renderer Feature中执行渲染过程后，Renderer Feature将还原原始相机矩阵。

## **实战一：使用Renderer Feature来进行描边**

![[c41b548f333e93c720ba5d2236d6386a_MD5.webp]]

实战是加深理论的最好途径，所以接下来要进行实战演练，我们来看下这些属性的使用情况，做一个简单的游戏场景角色的描边效果来展示下。

当然描边的方法有很多种，**这里我们介绍的方法是**先画模型的背面，然后使其根据法线的方向产生一定的偏移量，这些偏移量就显示为模型周围的轮廓，然后我们对其进行用户自定义的着色。

![[1cb74e055e592e293fdd62d2498632b9_MD5.webp]]

首先，我们是仅要为游戏角色Character来进行描边的操作，所以我们需要在Layer Masks上设置为Character Layer层；

![[94a72e323f0bd696380908df954730e2_MD5.webp]]

然后我们创建Renderer Feature ，并将Name改为Toon Outline。

接着我们来看看Event这个属性，它是决定我们渲染哪个阶段，这里我们选择After Rendering Opaques，在渲染完不透明物体之后。

然后我们来看下**Filters的两个参数**一个是Queue,一个是Layer Mask。**Queue：我们创建的这个Feature选择是渲染透明物体还是不透明物体，这里我们选择Opaques；Layer Mask：我们创建的这个Feature选择是渲染哪个图层中的对象；这里当然是选择我们的Character Layer。

最后我们看下**Overrides中的Material参数**将材质进行替换，以定义我们如何在Character Layer上渲染对象。（关于这里的材质创建我使用的是Universal Rendering Examples 中的OutlineHull，Shader使用的是ToonBasicOutline，具体的链接见文末，这里算是个伏笔，后面会带大家使用URP 已经支持的Shader Graph来进行制作 ）

![[83f44f79e76742f714602c6255b2a218_MD5.webp]]

到这里的话，使用Renderer Feature 制作的描边效果的流程就已经结束了。为了更进一步说明Renderer Feature中的Event 属性对整个渲染的影响，我们尝试将描边的顺序改为渲染的最后一步：后处理之后进行描边处理，从而得到如下图的效果。

![[d30608247b302d9aa7633e13eeea5154_MD5.webp]]

## 使用Scriptable Renderer Feature自定义后处理效果
我们已经学习 URP Renderer Feature 的使用以及如何在 URP 使用 Volume 框架进行后期处理效果。我们知道可编程的渲染管线（SRP）是通用渲染管线（URP）和高清渲染管线（HDRP）的基础。而 Unity 可编程的渲染管道（SRP）的核心功能之一就是可以编写自己的 Feature 并将其添加到渲染中，而无需从头开始构建整个渲染系统。

另外 URP 已经为我们提供了很多的后处理效果，但是我们如果想得到更多的效果的话，这里就要去扩展 URP 的 Volume。这就要用到 URP 提供的 RendererFeature 的功能，我们可以通过这个自行添加一个 pass 管理我们的自定义 Volume。

但是为了更好的展示我们制作的后期效果这里我们引入了自定义的 Timline 轨道 PostProcess Timline Track。

今天我们就一起使用 Scriptable Renderer Feature 与 Custom Timline Track 来**实现自定义后处理效果 Zoom Blur**。喜欢的话，就点个赞吧~

![[f50b5496a7893ecf8bf97fb8bb94c6ae_MD5.webp]]

URP目前提供的后处理效果

本节内容主要讲对 Scriptable Renderer Feature 的解析以及如何使用 Scriptable Renderer Feature，来进行扩展 URP 的 Volume 从而实现我们自定义的后处理效果,并涉及到如何实现自定义的 PostProcess Timline Track。小伙伴们，快来一起学习吧~

### **Scriptable Renderer Feature脚本代码解析**

我们先自己创建一个 Renderer Feature 的脚本来观察下。具体创建步骤如下：

![[45009c52055db690334a70cc722e74c3_MD5.webp]]

在 Project 窗口点击 Create>Rendering>Universal Render Pipeline>Render Feature，这样我们就可以在 Assets 中找到这个 CustomRenderPassFeature 脚本。

可以看到我们的 **Scriptable Renderer Feature 由两个类 CustomRenderPassFeature 与 CustomRenderPass 组成，CustomRenderPassFeature 类继承自 ScriptableRendererFeature，CustomRenderPass 类继承自 ScriptableRenderPass。

**“CustomRenderPassFeature”负责把这个 Render Pass 加到 Renderer 里面。Render Feature 可以在渲染管线的某个时间点增加一个 Pass 或者多个 Pass。

 **Create() 是用来初始化这个 Feature 的资源**。
 
 **AddRenderPasses() 在 Renderer 中插入一个或多个 ScriptableRenderPass，对这个 Renderer 每个摄像机都设置一次。**

![[b91ad09989e13494893f6613f69a28d7_MD5.webp]]
>                             Scriptable Renderer Feature脚本

“CustomRenderPass”是实际的渲染工作，CustomRenderPass 这个类重要的是要有 RenderPassEvent，它里面有 Pass 执行的时间点，用来控制每个 Pass 的执行顺序。在 ScriptableRenderPass 里，可以在构造函数里面去指定了 Event 在什么时间点执行（默认的话Event是在不透明物体之后画，即 AfterRenderingOpaque）。

![[62b594509f0c748935be542101696b5b_MD5.webp]]
>                                         RenderPassEvent

Configure() 在执行渲染过程之前，Renderer 将调用此方法。如果需要配置渲染目标及其清除状态，并创建临时渲染目标纹理，那就要重写这个方法。如果渲染过程未重写这个方法，则该渲染过程将渲染到激活状态下 Camera 的渲染目标。

Execute() 是这个类的核心方法，定义我们的执行规则；包含渲染逻辑，设置渲染状态，绘制渲染器或绘制程序网格，调度计算等等。

FrameCleanup() 可用于释放通过此过程创建的分配资源。完成渲染相机后调用。就可以使用此回调释放此渲染过程创建的所有资源。

![[c6d536b51e53679c19b42aa9a6458f34_MD5.webp]]
>                                         CustomRenderPass

那么接下来我就手把手带大家写Scriptable Renderer Feature来实现一个自定义的后处理效果。

### **创建 ZoomBlur 的 VolumeComponent**

接下来，带大家一起来使用Scriptable Renderer Feature实现一个ZoomBlur的后处理效果（如下图）：

![[afbf57ff77cfa68d678616f489815a14_MD5.webp]]

为了在我们的 Inspector 面板上就会示出我们已经定义好的参数。从而也方便我们通过修改参数对效果进行调整；我们创建一个名字为 ZoomBlur 的 VolumeComponent 脚本，让它继承 VolumeComponent 类,并且要实现 IPostProcessComponent 接口。

为了在我们的 Inspector 面板上就会示出我们已经定义好的参数。从而也方便我们通过修改参数对效果进行调整；我们创建一个名字为 ZoomBlur 的 VolumeComponent 脚本，继承自 IPostProcessComponent。

![[92269b7fdefd5c766577545f5cc7b25f_MD5.webp]]

代码如下：

![[829a4ce38f6d2c52a17a2c1211583909_MD5.webp]]

### **创建 ZoomBlurRenderFeature**

我们先创建一个名字为 ZoomBlurRenderFeature 的脚本，接下来我们来实现两个类和 Shader，以便我们可以插入自己的绘制 pass；我们创建 ZoomBlurRenderFeature 类让他继承自 ScriptableRendererFeature，我们创建类的成员变量 ZoomBlurPass，在 Create() 中创建 ZoomBlurRenderFeature 的资源，实例化 ScriptableRenderPass。

![[51c835f5d0a0e9d2f56162cf98795d92_MD5.webp]]

注意：zoomBlurPass = new ZoomBlurPass(RenderPassEvent.BeforeRenderingPostProcessing)；这里的参数RenderPassEvent我们设置为BeforeRenderingPostProcessing，也就是在渲染后处理效果之前进行执行。在ScriptableRenderPass里，我们可以在构造函数里面去指定了Event 在什么时间点执行，这个我们在后面会进行介绍。

![[10e315a56466f0bd2d75ae52d54b8214_MD5.webp]]

![[41b993b11d440af0877580c0866d3d94_MD5.webp]]

接着我们来看下 AddRenderPasses() 这个函数，它在 Renderer 中插入我们自定义的 ZoomBlurPass。

我们在 zoomBlurPass.Setup() 中对自定义的 Pass 进行初始化,然后通过 renderer.EnqueuePass 将 Pass 添加到 Render 中。

### **创建 ZoomBlurPass**

首先我们创建 ZoomBlurPass 类让它继承自 ScriptableRenderPass，接着我们需要创建一个 k_RenderTag 值为"Render ZoomBlur Effects"的标记，因为我们后续需要在 CommandBufferPool 中去获取到它，这样的话我们在 FrameDebugger 中也可以找到它。

![[5b92c24f26eb6f53afbfb3548e67e893_MD5.webp]]

![[a71066fe929684668c7bc9ace0e08aec_MD5.webp]]

紧接着我们需要创建对应的变量 给这些参数创建缓冲区，这些是预先计算好的属性 id，后面给 Shader 赋值的时候用 ID 会比用字符串快。

然后我们创建成员变量 ZoomBlur zoomBlur、Material zoomBlurMaterial、RenderTargetIdentifier currentTarget。

![[87c4abcd9e09695a17bff4f5dfda04d6_MD5.webp]]

接着就是完成 ZoomBlurPass 的构造函数，这里的 renderPassEvent 必须正确赋值才能保证该 ZoomBlurPass 类在正确的 RenderPassEvent 的顺序下渲染（这里 RenderPassEvent 的值为 BeforeRenderingPostProcessing）。

![[eab31e556f97d1e50baec5e1454eaf1c_MD5.webp]]

接着我们来写一个接口，将 currentTarget 传进去。

![[e1556ca8d6f6c135deac1af5b88d5477_MD5.webp]]

然后就来实现 ZoomBlurPass 这个类的核心方法 Execute() 来定义我们的执行规则，也就是在 Override 的 Execute 方法里做我们的具体的后处理。我们来简单的看下：首先是环境的准备，是否创建材质，后效是否生效。

![[bfa152cda3b58d6bb92385b61d527f72_MD5.webp]]

使用 VolumeManager.instance.stack 的 GetComponent 方法来获得我们的自定义 Volume 类的实例；并获取里面的属性变量来做具体的后处理。

![[37c6a739cbdb31fb2d9edc53b0d2f1fd_MD5.webp]]

这个IsActive返回的值，是我们在继承的VolumeComponent类里定义的，并非是VolumeComponent组件的启用和禁用决定的。

![[4d4b9a30a9f7e2944e8f4f831092578f_MD5.webp]]

然后从命令缓存池中获取一个gl命令缓存，CommandBuffer主要用于收集一系列gl指令，然后之后执行。

![[a9ba37bfde73ffffff149d74306e19c4_MD5.webp]]

我们要在Render中实现渲染逻辑，这里用到了两次 Blit，另外我们使用camera buffer的CommandBuffer.GetTemporaryRT方法来申请这样一张texture。传入着色器属性 ID 以及与相机像素尺寸相匹配的纹理宽高。FilterMode、RenderTextureFormat（大家可以对应修改下FilterMode，RenderTextureFormat，我们来看下效果）。

![[c099ded33155e8aa15f39e38ac17eb5d_MD5.webp]]

![[a5a81829f6bb7a47d32222e9c9adde59_MD5.webp]]

然后就是执行、回收。

![[8f880298c9c620e97d6fcf0a9141cb43_MD5.webp]]

完整的代码可以在这个本教程资料上进行获取。

### **写后处理所需的 ZoomBlur Shader**

接下来就是写上后处理所需的 ZoomBlur Shader。

![[7d6d8d7682e6ceafc1b2f9bb62e56a85_MD5.webp]]

Shader的完整代码

### **编辑器中的设置**

接下来就是具体的一些设置了。

首先，记得勾选上主摄像机里面的 PostProcessing。

![[e9ba0d9dc90332d5639251efe06ebc5e_MD5.webp]]

并在 Hierarchy 面板下添加一个空的GameObject，这里我命名为PostProcessTimeline，并在其下添加/Volume/Global Volume。

![[6f4c517a9e4e2ab511743b51106573cf_MD5.webp]]

点击 New，创建 Volume Profile 文件，并命名为 ZoomBlur。

在 Volume 组件下点击 Add Override，选择我们自定义的后处理 ZoomBlur 组件。

![[16481d184decc0b68b1fcd141a7828cf_MD5.webp]]

在 Project 目录下点击Create/Rendering/UniversalRenderPipeline/PipelineAsset(ForwardRenderer)。

![[9930e9ccc649e64e22e227d84e45ab16_MD5.webp]]

创建完成后就会多出两个 Asset 文件。

![[63104594ba65a8779218eaa9a53768d5_MD5.webp]]

在UniversalRenderPipelineAsset_Renderer下点击Add Render Feature，添加已经创建好的Zoom Blur Render Feature。

![[998e1f2b68f1aaa24fa733b5524a85e1_MD5.webp]]

在ProjectSettings/Graphics/Scriptable Render Pipeline Settings为 UniversalRenderPipelineAsset。

![[12cb5d50e2951e49a8108cf9a9facebb_MD5.webp]]

### **自定义 PostProcess Timeline 轨道之 PostProcessTrack**

为了更好的去控制与展示后处理效果，这里引入了 Timeline，但是 Timeline 中并没有 PostProcess 的 Timeline 轨道，所以这里我们需要自定义 PostProcess Track，如果我们要实现自定义的 PostProcess Track 至少要创建 4 个脚本——PostProcessTrack,PostProcessClip,PostProcessMixerBehaviour和PostProcessBehaviour。

首先我们来看下 PostProcessTrack 这个脚本：它继承自 TrackAsset，这个类代表轨道。

![[3bce148cebe389998cd19308c56bfae7_MD5.webp]]

PostProcessTrack脚本

如脚本代码所示：这里我们用到了两个Attribute，其中第一个是 [TrackColor(0.98f, 0.27f, 0.42f)]，表示在编辑器轨道前端的标识颜色。

![[0cd7c88b4dd869be58fe21694759a3b2_MD5.webp]]

轨道标识颜色

第二个是[TrackClipType(typeof(PostProcessClip))]，表示轨道添加那种Clip，这里我们设置为PostProcessClip。接下来我们要重写这个CreateTrackMixer 的方法。当我们完成这个类的时候，我们就可以在Timeline Add 窗口下添加这个轨道如下所示。

![[520c46bfd77b7f9736d1a87b84f335ee_MD5.webp]]

![[86d47c866eff851199fb0088b4eef81b_MD5.webp]]

### **自定义 PostProcess Timeline 轨道之 PostProcessClip**

接下来我们要完成 PostProcessClip 这个脚本，它继承自 PlayableAsset，并且要实现 ITimelineClipAsset 这个接口；我先介绍下这个脚本的作用， PostProcessClip 代表的是后处理片段资源本身，用来定义 Clip 支持哪些功能。比如我们脚本中定义的混合 Blend；这里的 Blending 代表如果我们将两段 Clip 拖到同时间段的话，两段 Clip 会进行融合。

![[c82e75c143e0f0e54dc9dba6b63f438d_MD5.webp]]

接下来就是在编辑器中对 PostProcessClip 进行处理的对应参数接口的实现。

![[17a8e7c946f662e2e78f605fb8e01330_MD5.webp]]

![[fdc5b0c3599a97b821b3a8d271eee390_MD5.webp]]

### **自定义 PostProcess Timeline 轨道之 PostProcessMixerBehaviour**

PostProcessMixerBehaviour 继承自 PlayableBehaviour。

![[4482add793077edeae5114878d23d291_MD5.webp]]

从上述代码，我们可以看出Mixer中的ProcessFrame方法可以获取当前帧轨道上所有轨道和权重。所以我们可以根据clip的权重在此方法下写相应的逻辑。

### **自定义 PostProcess Timeline 轨道之 PostProcessBehaviour**

同样 PostProcessBehaviour 也继承自 PlayableBehaviour，这个类的主要作用其实就是负责声明每个 PostProcessClip 在运行时所需的字段属性。

![[052fca0d84ccc11ca999de9ffc498f4b_MD5.webp]]

OK，到这里的话，我们就已经实现了自定义的PostProcess Timeline轨道，接下来我们就可以愉快的使用了。

![[bfec8e271eff12ca6f87f0531223a46b_MD5.webp]]

**参考文献：**

[1] URP 官方文档：

[https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@10.2/manual/index.html](https://link.zhihu.com/?target=https%3A//docs.unity3d.com/Packages/com.unity.render-pipelines.universal%4010.2/manual/index.html)

[2] URP 后期处理效果案例：

[https://qiita.com/t-matsunaga/items/09343ae7c683269374c4](https://link.zhihu.com/?target=https%3A//qiita.com/t-matsunaga/items/09343ae7c683269374c4)

[3] URP 自定义后处理效果：

[https://github.com/YuSiangLai/URPCustomPostProcessing](https://link.zhihu.com/?target=https%3A//github.com/YuSiangLai/URPCustomPostProcessing)

[https://github.com/yahiaetman/U](https://link.zhihu.com/?target=https%3A//github.com/yahiaetman/URPCustomPostProcessingStack)