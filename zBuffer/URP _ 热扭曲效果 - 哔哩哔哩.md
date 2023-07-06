内容偏多，下图是目录

![[16a5d29809cc086fdecabbcb80f877cd_MD5.webp]]

*   URP 管线下怎么获取屏幕信息。
*   实现特效中使用的热扭曲效果。
*   认识摄像机输出的不同阶段效果图。
    

*   URP 管线环境下，GrabPass 是失效的。
*   在 URP 管线使用的 `OpaqueTexture` 获取不透明物体的绘制。

![[ce449645edc09d088c28c26147053ec3_MD5.webp]]

*   在 shader Graph 里对应的节点是  SceneColor

![[b8b79fba7b11fb3cbd53f648435c682c_MD5.webp]]

**注意： OpaqueTexture 只能渲染不透明物体。** 透明物体是抓取不到的。

![[a48c0e7ed60fca2461eb8029bc83778e_MD5.webp]]

*   设置贴图

```
SAMPLER(_CameraOpaqueTexture);            //注意名字
```

*   计算出模型显示 UV 坐标

![[1e67ee312f033073be8ec7213aafb63c_MD5.webp]]

*   全代码
    

```
Shader "URP/14_OpaqueTexture_01"
{
    Properties
    {

    }
    HLSLINCLUDE
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)
        CBUFFER_END


        struct appdata
        {
            float4 positionOS : POSITION;
            float2 texcoord : TEXCOORD0;
        };

        struct v2f
        {
            float2 uv : TEXCOORD0;
            float4 positionOS : SV_POSITION;

        };


        SAMPLER(_CameraOpaqueTexture);                   //定义贴图
    ENDHLSL

    SubShader
    {
        Tags { "RenderPipeline"="UniversalPipeline" "RenderType"="Transparent"  "Queue" = "Transparent" "IgnoreProjector" = " True"}
        LOD 100

        Pass
        {
            Tags{ "LightMode"="UniversalForward" }
            Blend SrcAlpha OneMinusSrcAlpha

            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            v2f vert (appdata v)
            {
                v2f o;
                o.positionOS = TransformObjectToHClip(v.positionOS.xyz);
                return o;
            }

            half4 frag (v2f i) : SV_Target
            {

                half2 screenUV = (i.positionOS.xy / _ScreenParams.xy);
                half4 col = tex2D(_CameraOpaqueTexture, screenUV);
                return col;
            }
            ENDHLSL
        }
    }
}
```

*   效果
    

![[36fb6ccc38c5b73f6e280a1421583973_MD5.webp]]

![[8ee97bf9bbc88681364c992db823a9c4_MD5.png]]

*   制作原理，就是我们获取到屏幕颜色以后，对屏幕颜色 UV 进行偏移达到扭曲的效果。
    
*   原来我们是扭曲纹理，就是对纹理坐标 UV 扭曲。
    
*   现在我们获取一张 Niose 纹理对屏幕 UV 扭曲。
    

*   输入需要的变量
    

![[2170f4d44ac5a21d23fdca8d12351b6a_MD5.webp]]

           niose 纹理，遮罩纹理，niose 纹理移动速度，扭曲力度

*   计算 UV 信息，输出
    

![[bf0d0c970d193d4fda54a2af6b3de03e_MD5.webp]]

*   片元着色器阶段
    
    前计算出不同的 UV 移动方向，
    
    生成俩个扭曲的纹理，计算出不同的 UV 扭曲程度，
    
    加到屏幕 UV 里。
    

![[4d06e5b1da9b2402bbf89f24088828cd_MD5.webp]]

*   计算遮罩，
    
    我们输出设置成遮罩，这样我们就可控扭曲范围大小。
    

![[d0d9a27cc668ea26d43188b335295a2c_MD5.webp]]

*   效果
    

GIF

![[7bc8afd6a2b519feb94dae12448e2162_MD5.webp]]

![[8ee97bf9bbc88681364c992db823a9c4_MD5.png]]

*   准备好特效刀光贴图
    

![[e2c0b16135ca31bae9685f9bc1fee250_MD5.webp]]

*   在结构体输入，输出，如果不清楚可以参考上面的文档。
    

```
struct appdata
        {
            float4 positionOS : POSITION;
            float2 texcoord : TEXCOORD0;
            float4 vertexColor : COLOR;
        };

        struct v2f
        {
            float2 uv : TEXCOORD0;
            float4 positionOS : SV_POSITION;
            float4 vertexColor : COLOR;
        };
```

*   顶点着色器阶段输出
    

![[2c5d1dd689e4301b456a9edeac2b063e_MD5.webp]]

*   计算出俩个方向，使用顶点 Alpha 控制扭曲强度。
    

```
half4 frag (v2f i) : SV_Target
            {

                half4 mainTex = SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.uv);

                half2 screenUV = (i.positionOS.xy / _ScreenParams.xy) + half2(mainTex.rg * _Force * i.vertexColor.a);
                half4 col = tex2D(_CameraOpaqueTexture, screenUV);
                return col;
            }
```

*   效果
    

GIF

![[2c15b430b7e6c24fb3f112166910b3ab_MD5.webp]]

能看到扭曲越来越弱。

![[4de8ef8484ce8568ac8514d37f0379c8_MD5.webp]]

*   准备贴图
    

![[a284b2348dce7bb509d6f9fa405fbdbf_MD5.webp]]

*   修改变量
    

```
Properties
    {
        _NormalTex("_NormalTex",2D) = "bump" {}
        _NormalScale("_NormalScale",range(0,0.05)) = 0.01
    }
```

**注意：法线贴图使用 **"bump" {}  法线标签 ****

*   在片元着色器阶段我们处理法线的方法。
    

```
half4 frag (v2f i) : SV_Target
            {

                half3 NormalTex = UnpackNormalScale(SAMPLE_TEXTURE2D(_NormalTex,sampler_NormalTex,i.uv),_NormalScale);

                half2 screenUV = (i.positionOS.xy / _ScreenParams.xy) + half2(NormalTex.rg  * i.vertexColor.a);
                half4 col = tex2D(_CameraOpaqueTexture, screenUV);
                return col;
            }
```

这里和普通贴图不一样，使用一个内置函数，UnpackNormalScale 这函数是控制法线强度。

*   效果
    

GIF

![[33e991364ba89259a77f8a81b24b51bd_MD5.webp]]

*   代码
    

```
Shader "URP/14_OpaqueTexture_Normal"
{
    Properties
    {
        _NormalTex("_NormalTex",2D) = "bump" {}
        _NormalScale("_NormalScale",range(0,0.05)) = 0.01
    }
    HLSLINCLUDE
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)
            float4 _NormalTex_ST;
            float _NormalScale;
        CBUFFER_END


        struct appdata
        {
            float4 positionOS : POSITION;
            float2 texcoord : TEXCOORD0;
            float4 vertexColor : COLOR;
        };

        struct v2f
        {
            float2 uv : TEXCOORD0;
            float4 positionOS : SV_POSITION;
            float4 vertexColor : COLOR;
        };

        TEXTURE2D (_NormalTex);
        SAMPLER(sampler_NormalTex);

        SAMPLER(_CameraOpaqueTexture);                   //定义贴图
    ENDHLSL

    SubShader
    {
        Tags { "RenderPipeline"="UniversalPipeline" "RenderType"="Transparent"  "Queue" = "Transparent" "IgnoreProjector" = " True"}
        LOD 100

        Pass
        {
            Tags{ "LightMode"="UniversalForward" }
            Blend SrcAlpha OneMinusSrcAlpha

            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            v2f vert (appdata v)
            {
                v2f o;
                o.positionOS = TransformObjectToHClip(v.positionOS.xyz);
                o.uv = TRANSFORM_TEX(v.texcoord, _NormalTex);
                o.vertexColor = v.vertexColor;
                return o;
            }

            half4 frag (v2f i) : SV_Target
            {

                half3 NormalTex = UnpackNormalScale(SAMPLE_TEXTURE2D(_NormalTex,sampler_NormalTex,i.uv),_NormalScale);

                half2 screenUV = (i.positionOS.xy / _ScreenParams.xy) + half2(NormalTex.rg  * i.vertexColor.a);
                half4 col = tex2D(_CameraOpaqueTexture, screenUV);
                return col;
            }
            ENDHLSL
        }
    }
}
```

![[8ee97bf9bbc88681364c992db823a9c4_MD5.png]]

*   上面方法是不支持半透明物体。但是我们特效中都是半透明物体。
    
*   怎么让支持半透明物体扭曲。
    

*   实际上 URP 管线中给我提供了很多 COPY 的图，我们这里使用 **_AfterPostProcessTexture** 这张贴图
    

![[96f9c96feacd0335a9634ae2cd45050b_MD5.webp]]

*   这张图是后处理完保存的一张图。
    

*   然后利用 RendererFeatures 新建一个渲染时机 ，并新建一种 LightMode Tags 类型.
    
*   这样所有 Tags 是 Grab 的 shader 都会在后期处理完成之后在渲染。
    

简单理解：我们创建一个状态，让这个状态是在摄像机渲染完以后在渲染。

*   找到 URP 管线设置——设置后处理资源
    

![[fee2d0691bb9247444fdc93dd7846ad7_MD5.webp]]

![[5ff0ddabe7228357b0efb5ad8fe946ce_MD5.webp]]

![[0a6905c7dd006e1aa684403570f69baf_MD5.webp]]

*   设置一个渲染状态
    

![[8593345b90c1c495ea32f444b1682e5c_MD5.webp]]

*   获取屏幕渲染出的一张图。
    

![[5a612bb8fdc2bc954b9239a566b0ead4_MD5.webp]]

*   设置叠加模式和渲染所有层，设置标签  
    

![[8d03dc25740243a80da08b5cda2a7b4b_MD5.webp]]

*   我们前面设置渲染状态了，我们现在还需要创建一个摄像机。
    
    这个摄像机是获取主相机渲染的画面。
    

![[4cc5666a212399686c43ef269abc9161_MD5.webp]]

*   设置管线，关闭所有渲染图层。
    

![[2893a658cc8f78c68b45e28bcfe4b16d_MD5.webp]]

扩展 **Overlay Camera 是将其视图呈现在另一个摄影机的输出之上的摄影机.**

*   主相机里绑定 这个子相机
    

![[fad041b9f2f24b9ec82e258a33227f0c_MD5.webp]]

*   在 FrameDebug 里我们可以看到俩个摄像机，第二个摄像机就是看到前面渲染效果。
    

![[cad5f40eece7d0e59a6c0adb9eecbcf4_MD5.webp]]

*   定义渲染后处理图
    

![[67d65afa1c15471ab58b860bd20ba7b3_MD5.webp]]

**注意：这里是和上面不一样的图。**

*   标签里修改渲染方式。
    

![[14763e6c1a02c73b06ef18f044f47cee_MD5.webp]]

*   片元着色器阶段, 替换后处理素材。
    

```
half4 frag (v2f i) : SV_Target
            {

                half3 normal = UnpackNormalScale(SAMPLE_TEXTURE2D(_NormalTex,sampler_NormalTex,i.uv),_NormalScale);
                normal.z = lerp(1,normal.z,_NormalScale);
                half2 screenUV = (i.positionOS.xy / _ScreenParams.xy) + half2(normal.rg  * i.vertexColor.a);
                half4 col = tex2D(_AfterPostProcessTexture, screenUV);
                return col;
            }
```

*   效果
    
    现在半透明物体也可以支持扭曲效果。
    

GIF

![[ea31844b7b0326eef02876745e6655bf_MD5.webp]]

**代码**

```
Shader "URP/14_OpaqueTexture_01"
{
    Properties
    {
        _NormalTex("_NormalTex",2D) = "bump" {}
        _NormalScale("_NormalScale",range(0,0.05)) = 0.01
    }
    HLSLINCLUDE
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)
            float4 _NormalTex_ST;
            float _NormalScale;
        CBUFFER_END


        struct appdata
        {
            float4 positionOS : POSITION;
            float2 texcoord : TEXCOORD0;
            float4 vertexColor : COLOR;
        };

        struct v2f
        {
            float2 uv : TEXCOORD0;
            float4 positionOS : SV_POSITION;
            float4 vertexColor : COLOR;
        };

        TEXTURE2D (_NormalTex);
        SAMPLER(sampler_NormalTex);

        SAMPLER(_AfterPostProcessTexture);                   //定义贴图

    ENDHLSL

    SubShader
    {
        Tags { "RenderPipeline"="UniversalPipeline" "RenderType"="Transparent"  "Queue" = "Transparent" "IgnoreProjector" = " True"}
        LOD 100

        Pass
        {
            Tags{ "LightMode"= "Grab" }
            Blend SrcAlpha OneMinusSrcAlpha

            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            v2f vert (appdata v)
            {
                v2f o;
                o.positionOS = TransformObjectToHClip(v.positionOS.xyz);
                o.uv = TRANSFORM_TEX(v.texcoord, _NormalTex);
                o.vertexColor = v.vertexColor;
                return o;
            }

            half4 frag (v2f i) : SV_Target
            {
                half3 normal = UnpackNormalScale(SAMPLE_TEXTURE2D(_NormalTex,sampler_NormalTex,i.uv),_NormalScale);

                half2 screenUV = (i.positionOS.xy / _ScreenParams.xy) + half2(normal.rg  * i.vertexColor.a);
                half4 col = tex2D(_AfterPostProcessTexture, screenUV);
                return col;
            }
            ENDHLSL
        }
    }
}
```

**参考资料**

1. 人人有功练 Unity URP 管线透明折射扭曲材质的一系列实践（一）两种渲染时机的扭曲材质 - 知乎 (zhihu.com)

2. URP 系列教程 | 多相机玩法攻略 - 知乎 (zhihu.com)

*   在 Debug 状态下查看这俩个
    

![[5b28b9c97b0e6cec5645154ea69331fc_MD5.webp]]

![[24fa1584e94c59763be6657c9fc7f4b2_MD5.webp]]

*   _CameraColorTexture   是场景渲染后生成的纹理截图，
    
*   _CameraOpaqueTexture  是在不透明物体渲染后截图，所以截取不到透明物体。
    

*   _AfterPostProcessTexture 是后处理渲染结果输出的图。
    

![[6ab5e30d4885c8032b361c9235038015_MD5.webp]]

*   第一，我们可以看到最前渲染的是 CameraColorTexture
    

![[8e9d8145bc0127303f5fa0a6b5f40d37_MD5.webp]]

*   第二是 _CameraOpaqueTexture   不透明物体渲染后截图
    

![[689cbbc3db8aba21b1a1c580af41c9d9_MD5.webp]]

*   最后是后处理渲染出结果在输出的图。
    

![[b6c943eeeb7835f95cefcc1dfb6ba055_MD5.webp]]

![[8ee97bf9bbc88681364c992db823a9c4_MD5.png]]

*   URP 管线下怎么抓取渲染图，默认 URP 管线提供了设置，这里有俩个一个是获取渲染图，一个是渲染深度。
    

![[038a42a90ec9dd2fc697b270485c49b0_MD5.webp]]

*   渲染结果输出图的认识，会产生 CameraColorTexture ，_CameraOpaqueTexture   ，AfterPostProcessTexture ，他们的前后顺序，具体什么情况使用上面图。
    

![[ba27e61f3d96a1f3c2cea1f149724a30_MD5.webp]]

*   变透明实现扭曲，是在管线里，增加一种状态，使用第二个摄像机渲染输出 AfterPostProcessTexture 图。在进行扭曲处理。
    
    在编辑窗口是有问题的，需要运行状态查看效果。
    

![[df6d341f2ba2aedb69a8e2dd4f6de8a7_MD5.webp]]