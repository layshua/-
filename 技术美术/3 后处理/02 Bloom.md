# 1  Bloom

Bloom 是游戏开发中最常用的一种全屏后处理特效，它可以模拟真实相机的一种图像效果，让画面中较亮的区域扩散到周围区域中，造成一种朦胧的效果。
## 实现原理

**思路：**
1. 根据一个设定阈值提取图像中较亮区域，把它们存储到一张 RT 中
2. 利用模糊算法对这张纹理进行模糊处理
3. 与原图混合/叠加
![[Pasted image 20221209110919.png]]

## 二、用 Unity 实现 Bloom 算法
![[1 4.gif]]
### 1、C #脚本部分
-   定义 shader 中的相关参数
-   高斯模糊迭代次数
-   高斯模糊范围（blurSpread）
-   下采样系数（downSample，downSample 控制渲染纹理大小）
-   阈值：`luminanceThreshold`

-   OnRenderImage 函数部分
-   OnRenderImage 是官方提供的函数，可以使用这个函数获取当前的屏幕图像（得到渲染纹理）
-   实现内容：
-   检查材质的可用性（valid）
-   将阈值传入材质
-   定义 rtW、rtH 变量，作为屏幕的实际宽度、高度
-   创建一块大小小于原屏幕分辨率的缓冲区 buffer0

-   将滤波模式改为双线性滤波

-   调用“Graphics. Blit”方法 pass1 提取图像中较亮的区域
-   用 for 循环对图像进行高斯模糊处理

-   shader 中传入高斯模糊范围
-   定义第二个缓冲区 buffer1
-   调用“Graphics. Blit”方法进行竖直方向的高斯模糊
-   调用“ReleaseTemporary”方法释放缓冲区 buffer0（为了让模糊后的结果存入 buffer1，再用 buffer1 覆盖 buffer0，重新分配 buffer1，这样一来每次模糊使用的都是上次模糊做完的结果）
-   用同样的思路完成水平方向的高斯模糊
-   迭代后的 buffer0 的结果就是我们需要的结果

-   将模糊后的 buffer0 作为纹理传入 shader
-   用“Graphics. Blit”方法调用最后一个 pass，将模糊后的图像和原图混合叠加，作为最终结果输出

```c fold
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DS_Bloom : PostEffectsBase
{
    //定义使用的shader和材质
    public Shader bloomShader;
    private Material bloomMaterial = null;

    public Material material
    {
        get
        {
            bloomMaterial = CheckShaderAndCreateMaterial(bloomShader,bloomMaterial); 
            //调用PostEffectsBase基类中的函数，检查shader并且创建材质
            return bloomMaterial;
        }
    }
    
    //定义shader中的参数
    [Range(0, 4)] public int iterations = 3;//高斯模糊迭代次数
    [Range(0.2f, 3.0f)] public float blurSpread = 0.6f;//高斯模糊范围
    [Range(1,8)] public int downSample = 2;//下采样，缩放系数
    [Range(0.0f, 4.0f)] public float luminanceThreshold = 0.6f;//阈值
    
    //调用OnRenderImage函数来实现Bloom
    private void OnRenderImage(RenderTexture src, RenderTexture dest)
    {
        if (material != null) 
        {
            material.SetFloat("_LuminanceThreshold", luminanceThreshold);//传入阈值
            
            //src.width和hight代表屏幕图像的宽度和高度
            int rtW = src.width / downSample;//得到渲染纹理的宽度
            int rtH = src.height / downSample;//得到渲染纹理的高度
            
            //创建一块分辨率小于原屏幕的缓冲区：buffer0
            RenderTexture buffer0 = RenderTexture.GetTemporary(rtW,rtH,0);
            buffer0.filterMode = FilterMode.Bilinear;//滤波模式为双线性
            
            //用Blit方法调用shader中的第一个pass，提取图像中较亮的区域
            Graphics.Blit(src, buffer0, material, 0);//结果存在buffer0
            
            //迭代进行高斯模糊
            for (int i = 0; i <iterations; i++)
            {
                material.SetFloat("_BlurSize", 1.0f + i * blurSpread);//传入模糊半径
                
                //定义第二个缓冲区：buffer1
                RenderTexture buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                
                //用Blit方法调用shader中的第二个pass，进行竖直方向的高斯模糊
                Graphics.Blit(buffer0, buffer1, material, 1);
                
                //释放缓冲区buffer0，将buffer1的赋值给buffer，并重新分配buffer1
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1;
                buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0);
                
                //用Blit方法调用shader中的第三个pass，进行水平方向的高斯模糊
                Graphics.Blit(buffer0, buffer1, material, 2);
                
                //原理同上次释放，故技重施
                RenderTexture.ReleaseTemporary(buffer0);
                buffer0 = buffer1;
                
                //迭代完成后，buffer0 的结果就是高斯模糊后的结果
            }
            
            // 将完成高斯模糊后的结果 buffer0 传递给材质中的_Bloom 纹理属性
            material.SetTexture("_Bloom", buffer0);
            
            //用Blit方法调用shader中的第四个pass，完成混合
            Graphics.Blit(src, dest, material, 3);//dest是最终输出
            
            //最后记得释放临时缓冲区
            RenderTexture.ReleaseTemporary(buffer0);
        }
        else
        {
            Graphics.Blit(src, dest);
        }
    }
}

```

### 2、shader 部分

-   **基本思路**
-   使用 4 个 pass 完成 bloom 效果，对应 bloom 的实现步骤
-   pass1：提取亮部区域
-   pass2：实现竖直方向的高斯模糊
-   pass3：实现水平方向的高斯模糊
-   pass4：模糊后的高亮区域的RT作为纹理属性传给 shader，叠加到原图

-   **整理一下其中需要注意的几点**
-   **亮度如何获取？**
    -   亮度计算公式
-   **较亮区域如何提取？**
    -   采样后获取亮度值，再减去阈值，最后用 clamp 截取
-   **如何对竖直/水平方向进行高斯模糊？**
    -   在顶点着色器中计算 uv
-   **在 vertex shader 中计算的好处：**
    -   计算量 (次数)少（一般情况下顶点数量<像素数量）
    -   在顶点着色器中计算纹理坐标可以减少运算提高性能
    -   而且由于顶点到片元的插值是线性的，因此不会影响纹理坐标的计算结果

## 三、 Bloom 算法的应用

### 1、配合自发光贴图
![[Pasted image 20221209114223.png]]
### 2、配合特效
![[Pasted image 20221209114240.png]]
### 3、配合 ToneMapping
-   bloom 效果和 ToneMapping 结合可以较好的保留暗部和亮部的细节
![[Pasted image 20221209114250.png]]
### 4、GoodRay (体积光)效果
-   使用**径向模糊**代替高斯模糊，模拟光线往某个方向扩散的效果
![[Pasted image 20221209114329.png]]
-   GoodRay 配合 ToneMapping
![[Pasted image 20221209114348.png]]
-   可以看到 ToneMapping 解决了**过曝**的问题
#### ①径向模糊介绍及原理
-   径向模糊（Radial Blur）可以给画面带来很好的速度感，是各类游戏中后处理的常客，也常用于 Sun Shaft 等后处理特效中作为光线投射的模拟。
![[Pasted image 20221209114607.png]]
**径向模糊的原理：**
-   首先选取一个**径向轴心**（Radial Center）
-   然后**将每一个采样点的 uv 基于此径向轴心进行偏移**（offset）
-   并进行一定次数的**迭代采样**
-   最终将采样得到的 RGB 值累加，并除以迭代次数。
#### ②实现过程
相比 Bloom 效果的高斯模糊，这里我们使用径向模糊来代替它
![[1 5.gif]]
##### C #脚本

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class GodRay : PostEffectsBase
{
    // 声明GodRay效果需要的Shader，并创建相应的材质
    public Shader godRayShader;
    private Material godRayMaterial = null;
    public Material material
    {
        get
        {
            // 调用PostEffectsBase基类中检查Shader和创建材质的函数
            godRayMaterial = CheckShaderAndCreateMaterial(godRayShader, godRayMaterial);
            return godRayMaterial;
        }
    }

    // 高亮部分提取阈值
    public Color colorThreshold = Color.gray;
    // 光颜色
    public Color lightColor = Color.white;
    // 光强度
    [Range(0.0f, 20.0f)]
    public float lightFactor = 0.5f;
    // 径向模糊uv采样偏移值
    [Range(0.0f, 10.0f)]
    public float samplerScale = 1;
    // 迭代次数
    [Range(1, 5)]
    public int blurIteration = 2;
    // 分辨率缩放系数
    [Range(1, 5)]
    public int downSample = 1;
    // 光源位置
    public Transform lightTransform;
    // 光源范围
    [Range(0.0f, 5.0f)]
    public float lightRadius = 2.0f;
    // 提取高亮结果Pow系数，用于适当降低颜色过亮的情况
    [Range(1.0f, 4.0f)]
    public float lightPowFactor = 3.0f;

    private Camera targetCamera = null;

    void Awake()
    {
        targetCamera = GetComponent<Camera>();
    }

    void OnRenderImage(RenderTexture src, RenderTexture dest)
    {
        if (material && targetCamera)
        {
            int rtW = src.width / downSample;
            int rtH = src.height / downSample;
            // 创建一块大小小于原屏幕分辨率的缓冲区buffer0
            RenderTexture buffer0 = RenderTexture.GetTemporary(rtW, rtH, 0, src.format);

            //计算光源位置从世界空间转化到视口空间
            Vector3 viewPortLightPos = lightTransform == null ? new Vector3(.5f, .5f, 0) : targetCamera.WorldToViewportPoint(lightTransform.position);

            // 参数传给材质
            material.SetVector("_ColorThreshold", colorThreshold);
            material.SetVector("_ViewPortLightPos", new Vector4(viewPortLightPos.x, viewPortLightPos.y, viewPortLightPos.z, 0));
            material.SetFloat("_LightRadius", lightRadius);
            material.SetFloat("_PowFactor", lightPowFactor);
            Graphics.Blit(src, buffer0, material, 0);// 根据阈值提取高亮部分,使用pass0进行高亮提取，比Bloom多一步计算光源距离剔除光源范围外的部分
            material.SetVector("_ViewPortLightPos", new Vector4(viewPortLightPos.x, viewPortLightPos.y, viewPortLightPos.z, 0));
            material.SetFloat("_LightRadius", lightRadius);
            
            // 径向模糊的采样 uv 偏移值
            float samplerOffset = samplerScale / src.width;
            
            // 通过循环迭代径向模糊
            for (int i = 0; i < blurIteration; i++)
            {
                RenderTexture buffer1 = RenderTexture.GetTemporary (rtW, rtH, 0, src. format);
                float offset = samplerOffset * (i * 2 + 1);
                material.SetVector("_offsets", new Vector4(offset, offset, 0, 0));
                Graphics.Blit(buffer0, buffer1, material, 1);

                offset = samplerOffset * (i * 2 + 2);
                material.SetVector("_offsets", new Vector4(offset, offset, 0, 0));
                Graphics.Blit(buffer1, buffer0, material, 1);
                RenderTexture.ReleaseTemporary(buffer1);
            }
            
            //将完成模糊的结果传递给材质中的属性

            material.SetTexture("_BlurTex", buffer0);
            material.SetVector("_LightColor", lightColor);
            material.SetFloat("_LightFactor", lightFactor);
            
            // 将径向模糊结果与原图进行混合
            Graphics.Blit(src, dest, material, 2);
            
            //最后释放临时缓冲
            RenderTexture.ReleaseTemporary(buffer0);
        } else {
            Graphics.Blit(src, dest);
        }
    }

}

```

##### shader
```c
Shader "Unlit/GodRay"
{
    Properties
	{
		//对应原图和提取出来的部分
		_MainTex("Base (RGB)", 2D) = "white" {}
		_BlurTex("Blur", 2D) = "white"{}
	}
 
	CGINCLUDE
	#define RADIAL_SAMPLE_COUNT 6
	#include "UnityCG.cginc"
	
	// 提取亮部图像
	struct v2fExtractBright
	{
		float4 pos : SV_POSITION;
		float2 uv : TEXCOORD0;
	};
 
	// 径向模糊
	struct v2fRadialBlur
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float2 blurOffset : TEXCOORD1;
	};
 
	// 混合
	struct v2fGodRay
	{
		float4 pos : SV_POSITION;
		float2 uv  : TEXCOORD0;
		float2 uv1 : TEXCOORD1;
	};

	//声明属性和C#脚本中用到的变量
	sampler2D _MainTex;
	float4 _MainTex_TexelSize;
	sampler2D _BlurTex;
	float4 _BlurTex_TexelSize;
	float4 _ViewPortLightPos;
	
	float4 _offsets;
	float4 _ColorThreshold; //高亮部分阈值
	float4 _LightColor; //光颜色
	float _LightFactor; //光强度
	float _PowFactor; //提取高亮结果Pow系数，用于适当降低颜色过亮的情况
	float _LightRadius; //光源范围
 
	// 提取亮部图像VS
	v2fExtractBright vertExtractBright(appdata_img v)
	{
		v2fExtractBright o;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
		// 平台差异化处理
		#if UNITY_UV_STARTS_AT_TOP
		if (_MainTex_TexelSize.y < 0)
			o.uv.y = 1 - o.uv.y;
		#endif	
		return o;
	}
 
	// 提取亮部图像PS
	fixed4 fragExtractBright(v2fExtractBright i) : SV_Target
	{
		fixed4 color = tex2D(_MainTex, i.uv);
		float distFromLight = length(_ViewPortLightPos.xy - i.uv);
		float distanceControl = saturate(_LightRadius - distFromLight);

		// 仅当 color 大于设置的阈值的时候才输出
		float4 thresholdColor = saturate(color - _ColorThreshold) * distanceControl;
		float luminanceColor = Luminance(thresholdColor.rgb);
		luminanceColor = pow (luminanceColor, _PowFactor);
		return fixed4(luminanceColor, luminanceColor, luminanceColor, 1);
	}
 
	// 径向模糊VS
	v2fRadialBlur vertRadialBlur(appdata_img v)
	{
		v2fRadialBlur o;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;

		// 径向模糊采样偏移值 * 沿光的方向权重
		o.blurOffset = _offsets * (_ViewPortLightPos.xy - o.uv);

		return o;
	}
 
	// 径向模拟PS
	fixed4 fragRadialBlur(v2fRadialBlur i) : SV_Target
	{
		half4 color = half4(0,0,0,0);
		//通过迭代，将采样得到的RGB值累加
		for(int j = 0; j < RADIAL_SAMPLE_COUNT; j++)   
		{	
			color += tex2D(_MainTex, i.uv.xy);
			i.uv.xy += i.blurOffset; 	
		}
		//最后除以迭代次数
		return color / RADIAL_SAMPLE_COUNT;
	}
 
	// 混合VS
	v2fGodRay vertGodRay(appdata_img v)
	{
		v2fGodRay o;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv.xy = v.texcoord.xy;
		o.uv1.xy = o.uv.xy;
		#if UNITY_UV_STARTS_AT_TOP
		if (_MainTex_TexelSize.y < 0)
			o.uv.y = 1 - o.uv.y;
		#endif	
		return o;
	}

 	// 混合PS
	fixed4 fragGodRay(v2fGodRay i) : SV_Target
	{
		fixed4 ori = tex2D(_MainTex, i.uv1);
		fixed4 blur = tex2D(_BlurTex, i.uv);
		return ori + _LightFactor * blur * _LightColor;
	}
 
	ENDCG
 
	SubShader
	{
		ZTest Always Cull Off ZWrite Off

		// 提取高亮部分
		Pass
		{
			CGPROGRAM
			#pragma vertex vertExtractBright
			#pragma fragment fragExtractBright
			ENDCG
		}
 
		// 径向模糊
		Pass
		{
			CGPROGRAM
			#pragma vertex vertRadialBlur
			#pragma fragment fragRadialBlur
			ENDCG
		}
 
		// 将亮部图像与原图进行混合得到最终的GodRay效果
		Pass
		{
			CGPROGRAM
			#pragma vertex vertGodRay
			#pragma fragment fragGodRay
			ENDCG
		}
	}
}

```
## 四、扩展/课后作业
### 如何实现 bloom 的 mask 功能

①用 Alpha 通道
-   参考：[https://blog.csdn.net/SnoopyNa2Co3/article/details/88075047](https://blog.csdn.net/SnoopyNa2Co3/article/details/88075047)
②用 SRP 渲染一张 Mask 图
③用 Command-Buffer
④用模板测试
⑤ 直接用 Mask 图（简单情况下）

