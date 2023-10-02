# 案例
## 毛玻璃
不适用2D管线
![[screen0.gif]]

### 原理

使用高斯模糊来表达不清晰的半透明效果。

先渲染除毛玻璃外的不透明和半透明物体，然后做全屏高斯模糊，将结果保存到RT。最后渲染毛玻璃，在vertex阶段计算毛玻璃顶点在屏幕空间的位置，fragment阶段根据上述屏幕空间位置采样高斯模糊RT，将毛玻璃范围内的RT画在毛玻璃上。

### 实现

上面原理部分描述了最基础效果。实际开发中，用URP的方式照搬了开源工程[[1 前言]](https://zhuanlan.zhihu.com/p/437305443#ref_1) ，完成了一些花活：lerp不同分辨率的高斯模糊，使模糊效果更佳；毛玻璃多一张贴图，控制lerp数值，来呈现不同的毛玻璃效果。

#### RenderFeature

在场景中放置一Plane作为毛玻璃，设置layer为Glass
![[Pasted image 20221013143858.png|300]]
>                      毛玻璃物体

在默认ForwardRenderer中添加两个RenderFeature：

-   GrabScreenBlur：自定义的RendererFeature。使用MyBlur shader的材质，完成除毛玻璃外的全屏高斯模糊，时机为AfterRenderingTransparents；
-   FrostedGlass：内置的RenderObjects。指定渲染Glass layer的毛玻璃，时机为AfterRenderingTransparents。
![[Pasted image 20221013143942.png|300]]
>                 ForwardRenderer

**GrabScreenBlurRendererFeature源码**：

执行MyBlur shader，产出4张RT：_BluredTexture0~3，供MyFrostedGlass shader使用。

```csharp
using System;  
using UnityEngine;  
using UnityEngine.Rendering;  
using UnityEngine.Rendering.Universal;  
  
public class GrabScreenBlurRendererFeature : ScriptableRendererFeature  
{  
   [Serializable]  
   public class Config  
   {  
      public float blurAmount;  
      public Material blurMaterial;  
   }  
  
   [SerializeField]  
   private Config config;  
  
        private GrabScreenBlurPass grabScreenBlurPass;  
  
        public override void Create()  
        {  
      grabScreenBlurPass = new GrabScreenBlurPass(config);  
      grabScreenBlurPass.renderPassEvent = RenderPassEvent.AfterRenderingTransparents;  
        }  
  
        public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)  
        {  
      grabScreenBlurPass.SetUpColorRT(renderer.cameraColorTarget);  
      renderer.EnqueuePass(grabScreenBlurPass);  
        }  
  
   // render pass  
   class GrabScreenBlurPass : ScriptableRenderPass  
   {  
      private Material blurMat;  
      private float blurAmount;  
  
      private RenderTextureDescriptor rtDesc;  
      private RenderTargetIdentifier colorRT;  
  
      private int[] sizes = { 1, 2, 4, 8 };  
  
      public GrabScreenBlurPass(Config config)  
      {  
         blurMat = CoreUtils.CreateEngineMaterial(Shader.Find("Unlit/Blur"));  
         blurAmount = config.blurAmount;  
  
         profilingSampler = new ProfilingSampler(nameof(GrabScreenBlurPass));  
      }  
  
      public void SetUpColorRT(RenderTargetIdentifier rt)  
      {  
         colorRT = rt;  
      }  
  
      public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)  
      {  
         rtDesc = cameraTextureDescriptor;  
      }  
  
      public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)  
      {  
         CommandBuffer cmd = CommandBufferPool.Get();  
         using (new ProfilingScope(cmd, profilingSampler))  
         {  
            for (int i = 0; i < sizes.Length; ++i)  
            {  
               //downsample  
               int size = sizes[i];  
               rtDesc.width = Screen.width / size;  
               rtDesc.height = Screen.height / size;  
               //申请临时RT  
               int blurRT1 = Shader.PropertyToID("_BlurRT1_" + i);  
               int blurRT2 = Shader.PropertyToID("_BlurRT2_" + i);  
               cmd.GetTemporaryRT(blurRT1, rtDesc);  
               cmd.GetTemporaryRT(blurRT2, rtDesc);  
  
               //Blur  
               cmd.SetGlobalVector("_BlurAmount", new Vector4(blurAmount / rtDesc.width, 0, 0, 0));  
               cmd.Blit(colorRT, blurRT1, blurMat);  
               cmd.SetGlobalVector("_BlurAmount", new Vector4(0, blurAmount / rtDesc.height, 0, 0));  
               cmd.Blit(blurRT1, blurRT2, blurMat);  
               cmd.SetGlobalVector("_BlurAmount", new Vector4(blurAmount * 2 / rtDesc.width, 0, 0, 0));  
               cmd.Blit(blurRT2, blurRT1, blurMat);  
               cmd.SetGlobalVector("_BlurAmount", new Vector4(0, blurAmount * 2 / rtDesc.height, 0, 0));  
               cmd.Blit(blurRT1, blurRT2, blurMat);  
  
               cmd.SetGlobalTexture("_BluredTexture" + i, blurRT2);  
            }  
  
            cmd.SetRenderTarget(colorRT);  
         }  
         //schedule command buffer  
         context.ExecuteCommandBuffer(cmd);  
         CommandBufferPool.Release(cmd);  
      }  
   }  
}
```

#### Shader

**Blur**：完成全屏高斯模糊。

```C
Shader "Unlit/Blur"  
{  
    Properties  
    {  
        _MainTex ("Texture", 2D) = "white" {}  
    }  
    SubShader  
    {  
        Tags {"RenderPipeline" = "UniversalPipeLine" }  
        
       HLSLINCLUDE    
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"    
       CBUFFER_START(UnityPerMaterial)    
        float4 _MainTex_ST;  
       float4 _BlurAmount;  
         
        CBUFFER_END    
       ENDHLSL    
        Pass  
        {  
           Tags{"LightMode" = "UniversalForward"}  
            HLSLPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
  
            struct Attributes  
            {  
            float4 positionOS : POSITION;  
            float2 uv : TEXCOORD;  
         };  
  
         struct Varings  
         {  
            float4 positionCS : SV_POSITION;  
            float2 uv : TEXCOORD;  
            float4 uv01 : TEXCOORD1;  
            float4 uv23 : TEXCOORD2;  
            float4 uv45 : TEXCOORD3;  
         };  
  
            TEXTURE2D(_MainTex);       
            SAMPLER(sampler_MainTex);    
           
         Varings vert(Attributes i)  
         {  
            Varings o;  
            VertexPositionInputs posInputs = GetVertexPositionInputs(i.positionOS.xyz);  
            o.positionCS = posInputs.positionCS;  
           
            o.uv = TRANSFORM_TEX(i.uv, _MainTex);  
            o.uv01 =  i.uv.xyxy + _BlurAmount.xyxy * float4(1, 1, -1, -1);  
            o.uv23 =  i.uv.xyxy + _BlurAmount.xyxy * float4(1, 1, -1, -1) * 2.0;  
            o.uv45 =  i.uv.xyxy + _BlurAmount.xyxy * float4(1, 1, -1, -1) * 3.0;  
           
            return o;  
         }  
           
         float4 frag(Varings i) : SV_Target   
         {  
            float4 color = float4(0, 0, 0, 0);  
            color += 0.40 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv);  
            color += 0.15 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.xy);   
            color += 0.15 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.zw);   
            color += 0.10 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.xy);   
            color += 0.10 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.zw);   
            color += 0.05 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv45.xy);   
            color += 0.05 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv45.zw);   
           
            return color;  
         }  
            ENDHLSL   
}    
   }  
}
```

**MyFrostedGlass**：完成毛玻璃渲染。

_FrostTexture为毛玻璃纹理，实际只需要一个通道。资源来自参考工程[[1 前言]](https://zhuanlan.zhihu.com/p/437305443#ref_1) ，出彩的效果主要靠它。

![[f0fb5a7ecdf204c340c649a9ebaa65cc_MD5 1.webp]]
>                                              不同的毛玻璃纹理

_FrostTexture与_FrostIntensity一起控制_BluredTexture0~3四张RT的lerp效果。

```c
Shader "Unlit/forostedGlass"  
{  
    Properties  
    {  
        _FrostTexture ("FrostTexture", 2D) = "white" {}  
       _FrostIntensity ("FrostIntensity",float) = 1   
_GlassColor("GlassColor", COLOR) = (1,1,1,1)  
         
    }  
    SubShader  
    {  
        Tags { "RenderType"="Transparent" "Queue" = "Transparent" "RenderPipeline" = "UniversalPipeLine"   }  
        LOD 100  
      HLSLINCLUDE    
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"    
       CBUFFER_START(UnityPerMaterial)    
        float4 _FrostTexture_ST;  
       float4 _BluredTexture0_ST;  
       float _FrostIntensity;  
       float4 _GlassColor;  
        CBUFFER_END    
       ENDHLSL    
        Pass  
        {  
            HLSLPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
  
            struct Attributes  
         {  
            float4 positionOS : POSITION;  
            float2 uv : TEXCOORD0;  
         };  
           
         struct Varings  
         {  
            float4 positionCS : SV_POSITION;  
            float2 uv : TEXCOORD0;  
            float4 uvBluredTex : TEXCOORD1;  
         };  
  
         TEXTURE2D(_FrostTexture);  
         SAMPLER(sampler_FrostTexture);  
           
         TEXTURE2D(_BluredTexture0);  
         SAMPLER(sampler_BluredTexture0);  
           
         TEXTURE2D(_BluredTexture1);  
         TEXTURE2D(_BluredTexture2);  
         TEXTURE2D(_BluredTexture3);  
              
         Varings vert(Attributes i)  
         {  
            Varings o;  
            VertexPositionInputs posInputs = GetVertexPositionInputs(i.positionOS.xyz);  
            o.positionCS = posInputs.positionCS;  
            o.uv = TRANSFORM_TEX(i.uv, _FrostTexture);  
            o.uvBluredTex = ComputeScreenPos(o.positionCS);  
  
            return o;  
         }  
           
         float4 frag(Varings i) : SV_Target   
         {  
            float surfSmooth = 1 - SAMPLE_TEXTURE2D(_FrostTexture, sampler_FrostTexture, i.uv).x * _FrostIntensity;  
            surfSmooth = clamp(0, 1, surfSmooth);  
           
            half4 ref00 = SAMPLE_TEXTURE2D(_BluredTexture0, sampler_BluredTexture0, i.uvBluredTex.xy / i.uvBluredTex.w);  
            half4 ref01 = SAMPLE_TEXTURE2D(_BluredTexture1, sampler_BluredTexture0, i.uvBluredTex.xy / i.uvBluredTex.w);  
            half4 ref02 = SAMPLE_TEXTURE2D(_BluredTexture2, sampler_BluredTexture0, i.uvBluredTex.xy / i.uvBluredTex.w);  
            half4 ref03 = SAMPLE_TEXTURE2D(_BluredTexture3, sampler_BluredTexture0, i.uvBluredTex.xy / i.uvBluredTex.w);  
           
            float step00 = smoothstep(0.75, 1.00, surfSmooth);  
            float step01 = smoothstep(0.5, 0.75, surfSmooth);  
            float step02 = smoothstep(0.05, 0.5, surfSmooth);  
            float step03 = smoothstep(0.00, 0.05, surfSmooth);  
           
            float4 refraction = lerp(ref03, lerp(lerp(lerp(ref03, ref02, step02), ref01, step01), ref00, step00), step03);  
              
            return refraction * _GlassColor;  
         }  
            ENDHLSL   
}  
    }  
}
```

blur次数、downsample数值、毛玻璃贴图、颜色差值方法的不同组合能呈现出不同的效果，挺有意思。
### 参考
1.  ^[a](https://zhuanlan.zhihu.com/p/437305443#ref_1_0)[b](https://zhuanlan.zhihu.com/p/437305443#ref_1_1)[https://github.com/andydbc/unity-frosted-glass](https://github.com/andydbc/unity-frosted-glass)

## 2D Sprite 影子
![[Pasted image 20221016213657.png]]
### 流程
打开unity新建一个2D场景，导入一张2D人物图片和一张透明的图片（用来接收影子），修改图片Texture Type为 Sprite 类型，
1.拖拽透明图到物体精灵节点下，命名为shadow（把A通道设置为0），用来接收影子
2.材质球，拖拽到 shadow 上
3.最后把c#脚本拖拽到物体精灵节点上，把shadow拖拽到脚本的Shadow变量上，点击运行即可。：
![[Pasted image 20221015095547.png]]
![[Pasted image 20221015095611.png]]
### 实现思路：

其实很简单，就是把人物的纹理传递给shadow shader, 把alpha值大于0的像素的rgb变为黑色即可。

```C#
using System.Collections;  
using System.Collections.Generic;  
using UnityEngine;  
  
public class ShadowScript : MonoBehaviour  
{  
    public GameObject shadow;  
    void Start () {  
        if (!shadow) {  
            return;  
        }  
        // 获取纹理并传递到shader  
        var shadowMat = shadow.GetComponent<SpriteRenderer> ().material;  
        //后两行也可以传入void Update()，实现每帧刷新
        var playerTex = GetComponent<SpriteRenderer> ().sprite.texture;  
        shadowMat.SetTexture ("_ShadowTex", playerTex);  
    }  
}
```

```c
Shader "Shadow/material"  
{  
    Properties  
    {  
        _ShadowColor("ShadowColor",Color)=(1,1,1,1)  
    }  
    // ---------------------------【子着色器】---------------------------  
    SubShader  
    {  
        Cull Off ZWrite Off ZTest Always  
        Blend SrcAlpha OneMinusSrcAlpha  
        Tags { "Queue"="Transparent" "RenderType"="Transparent" }  
          
        HLSLINCLUDE  
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"  
        CBUFFER_START(UnityPerMaterial)  
        float4 _ShadowTex_ST;  
        float4 _ShadowColor;  
        CBUFFER_END  
        ENDHLSL  
                Pass  
        {  
            Tags{"LightMode" = "Universal2D"}  
            HLSLPROGRAM  
            #pragma vertex vert  
            #pragma fragment frag  
  
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
              
            v2f vert (appdata v)  
            {  
                v2f o;  
                o.vertex = TransformObjectToHClip(v.vertex);  
                o.uv = v.uv;  
                o.uv.y = 1 - o.uv.y;  
                return o;  
            }  
  
            //通过c#获取到人物纹理传递过来  
            TEXTURE2D(_ShadowTex);  
            SAMPLER(sampler_ShadowTex);  
  
            float4 frag (v2f i) : SV_Target  
            {  
                // 采样传过来的纹理  
                float4 col = SAMPLE_TEXTURE2D(_ShadowTex, sampler_ShadowTex, i.uv);  
                // 这里用step代替if  
                // 当 透明度值大于1时, 就呈现黑色(即影子)  
                col.rgb = (1 - step(0,col.a)) + _ShadowColor;  
                return col;  
            }  
            ENDHLSL  
        }  
    }  
}
```
## 气泡破裂
![[bubble.gif|300]]
### URP PBR材质
![[Pasted image 20221016211318.png]]
### 粒子系统
![[Pasted image 20221016212821.png]]
![[Pasted image 20221016212918.png]]
![[Pasted image 20221016213124.png]]

## 可交互雪地（内置管线URP）
### 雪球材质
![[Pasted image 20221016221307.png|300]]
>（参数为了适配雪地，造成外观不好QAQ，实际很漂亮）
#### 基本颜色 + 轨迹颜色
![[Pasted image 20221016220652.png]]
#### 高光
![[Pasted image 20221016220126.png]]
#### 点状高光
![[Pasted image 20221016220322.png]]
#### 顶点偏移
![[Pasted image 20221016220835.png]]
#### 结果
![[Pasted image 20221016221222.png]]
### 交互
1. 新建plane，赋予雪球材质，新建camera，作为plane子节点
![[Pasted image 20221016221812.png]]
2. 相机设置
![[Pasted image 20221016221905.png]]
背景颜色为黑色，正交相机，新建StepTex材质，作为Target Texture。
3. 建立粒子系统，单独设置一个Layer，指定摄像机只渲染该Layer
![[Pasted image 20221016223007.png]]
![[Pasted image 20221016223145.png]]
![[Pasted image 20221016223236.png]]
摄像机只渲染粒子。可以采集粒子系统的移动轨迹，反映到StepTex中。
4. 设置粒子系统，下面这个参数可以控制脚印的渐变，其他属性按情况调整。如果轨迹与实际脚印位置不相符，可以旋转粒子系统到正确的角度。
![[Pasted image 20221016223330.png]]
## 粒子闪电
unity 粒子系统制作闪电
制作闪电效果的方法有以下几种：

2d动画方式（适合2d游戏的背景或范围技能，性能最好，效果看设计师水平）
LineRenderer划1线的方法（可以动态设定起点和终点，适合需要指定放电目标的场景）
trail拖尾的方法（同2）
粒子系统的方法（性能最差，效果最好，花样最多最省事）
本文描述粒子系统的构建方式
### 创建粒子对象
在Hierarchy窗口中点击右键 => particle system场景中即可出现粒子对象
![[20200621234537979.gif]]

### 设置起点和粒子发射形状
在Inspector创空中 找到Particle System组件，勾选Shape栏
设置Shape为Cone，并设置Radius为0.0001，这样发射位置就变成一个点了。
![[20200621234946355.gif]]
### 设置粒子拖尾
勾选trials栏
![[20200622001046538.gif]]
### 让粒子随机移动，产生闪电的曲折效果
勾选noise栏，按图设置参数

有点意思了吧。可是闪电是紫色的，这是因为没有设置相应待material，一般情况下闪电都是高亮的，所以我们的做个合适的材质

### 制作闪电材质
在Renderer中设置Trail Materil为刚刚新建的材质

### 设置闪电材质
勾选Renderer栏，并设置Trail Material为上一步新建的材质。感人的画面就出现了
![[20200622003510479.gif]]
**让闪电随着延伸而变细**
自然界里待闪电和树根差不多，随着时间待推移，末端会越来越细。只要勾选 Size oiver life time，并设置曲线为1-0即可实现
![[20200622010310870.gif]]
**添加闪电的末端光亮**
闪电的末端有亮球效果
Renderer 》Min particle Size
最终效果：
![[20200622010310870 1.gif]]
### 闪电相关设置
放电速度：Simulation Speed
放电数目：Max Particles
每次释放几个粒子：Emission > Rate over Time
每根闪电的节点数，Tails 》minimum vertex distance
闪电长度 ：Start life time
闪电折角圆滑度：noise》frequency
闪电折角的角度抖动强度：noise》strength
放电范围角度：Shape 》 angle

### 几个效果截图
**单次放电效果**
当设置Max Particles = Emission > Rate over Time = 3 时，就变成了技能放电效果：
![[20200622012746401.gif]]
**闪电球**
将shape设置为spher，并设置 noise strength为2 就是闪电球。
![[20200622015307137.gif]]

## 传送门
参考：[[Unity]简易传送门效果 - 简书 (jianshu.com)](https://www.jianshu.com/p/25636d97861d)
![[Portal.gif|300]]
主要是利用Twirl和Voronoi节点
![[Pasted image 20221018111002.png]]