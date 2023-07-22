# 4.1 Bloom  

课程链接：[【技术美术百人计划】图形 4.1 Bloom算法 游戏中的辉光效果实现哔哩哔哩bilibili](https://www.bilibili.com/video/BV1a3411z7LC?p=2)  

项目地址：[logic-three-body/Unity_Bloom: Unity Post procession about bloom (github.com)](https://github.com/logic-three-body/Unity_Bloom)  

## 4.1.1 前置知识  

### 什么是bloom  

辉光效果，模拟摄像机图像效果。让舞台有真实明亮效果  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F17%2FwsSWulrIYaVEFUH.png&sign=c8d517a508eac7dde54d63b5789357b554ec49ef64b284d23ea90e9a12571567)  

bloom算法：  

1.提取原图较量区域（阈值设定亮度）  

2.模糊（提取后）图像  

3.与原图混合  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F17%2F9WjgN1BI26Dd7Uq.png&sign=a66dca0efe57c0a0a08989e68ff6af82f2803c7862f073b7fb71fb3c367969a9)  

wiki部分总结：  

该效果会在高亮度物体周围产生条纹或羽毛状的光芒，以模糊图像细节。如果物体背光，从第三人称观察，光线会表现得更加真实，并在某种程度上与遮挡物体产生交叠。  

在现实世界中，透镜无法完美聚焦是高光的物理成因；理想透镜也会在成像时由于衍射而产生[艾里斑](https://zh.wikipedia.org/wiki/%E8%89%BE%E9%87%8C%E6%96%91)通常情况下难以察觉这些不完美的瑕疵，除非有强烈亮光源存在：这时，图像中的亮光部分会渗出其真实边界。  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F17%2FdBcI9YPSHLZjRyD.jpg&sign=0311af6eba22a29801c640678cac4a1d3c3b564f20bda1b36ae7a45c63a61e94)  

阅读链接：[here](https://zh.wikipedia.org/wiki/%E9%AB%98%E5%85%89)  

### HDR  

LDR（Low Dynamic Range,低动态范围） RGB range in [0,1]  

JPG PNG等格式图片  

  

HDR（High Dynamic Range,高动态范围） RGB range 可超过 [0,1]  

这样可以提取更高亮度（超过1）的区域产生bloom效果  

HDR、EXR格式图片  

  

wiki部分总结：  

现实中，当人由黑暗的地方走到光亮的地方，会眯起眼睛。人在黑暗的地方，为了看清楚对象，瞳孔会放大，以吸收更多光线；当突然走到光亮的地方，瞳孔来不及收缩，所以眯起眼睛，保护视网膜上的视神经。  

而电脑无法判断光线明暗，唯有靠HDRR技术模拟这效果——人眼自动适应光线变化的能力。方法是快速将光线渲染得非常光亮，然后将亮度逐渐降低。而HDRR的最终效果是亮处的效果是鲜亮，而黑暗处的效果是能分辨物体的轮廓和深度，而不是以往的一团黑。  

HDRR技术的使用场景举例如下：  

例一场景： 阳光普照下，水旁有一道墙壁。当阳光由水面反射到墙上，晴朗而明亮的天空会稍微暗一些，这样能有助表现出水面的反光效果。当人们低头看水面，阳光会反射到人眼中，整个画面会非常光亮，并逐渐减弱，因为人眼适应了从水面反射的光。  

例二场景： 阳光直射到一块光亮的石头。若你紧盯着它，石头表面的泛光会逐渐淡出，显示出更多细节。  

例三场景： 枪支的反射效果。  

阅读链接：[here](https://zh.wikipedia.org/wiki/%E9%AB%98%E5%8A%A8%E6%80%81%E5%85%89%E7%85%A7%E6%B8%B2%E6%9F%93)  

推荐阅读：[Bloom是什么 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/76505536) ：本文部分从生物（光子）角度阐述  

### 卷积  

数学运算如下：  

（请注意Kernel卷积核在运算时实际已经水平旋转180度，比如第一行第二列的“2”两侧的1实际已经颠倒位置，但因为值一样所以没有表现出来，PS：如果仅从获得运算结果的角度下，可以暂时理解为一种步骤）  

  

动图演示：  

  

再来一张加深理解：  

  

辅助阅读：[卷积究竟卷了啥？——17分钟了解什么是卷积哔哩哔哩bilibili](https://www.bilibili.com/video/av713651125)  

[什么！卷积要旋转180度？！ - 简书 (jianshu.com)](https://www.jianshu.com/p/8dfe02b61686)  

减少图像噪声、降低细节层次的方法。  

  

高斯核  

  

二维高斯函数特点：可分离性，将二维高斯函数拆成两个一维高斯函数以降低计算量  

二维高斯核运算：N N W * H 次纹理采样  

两次一维高斯核运算：2 N W * H 次纹理采样  

  

《unity shader入门精要》对应页数：P253-254  

## 4.1.2 算法演示  

C#脚本部分（详情请见注释，结合老师的注释加了些总结，建议配合视频或《入门精要》P259-260阅读）  

shader部分（详情请见注释，建议配合视频或《入门精要》阅读P261-262阅读）：  

bloom shader挂到材质上是这样，但是我们并不将它挂到具体材质，而是利用脚本生成材质并给材质传递数据（所用的纹理是渲染纹理，是渲染管线生成的也不是具体的纹理图片）  

  

downsample：  

  

pass 0 threshold截取阈值：  

<video src="" control></video>

pass0+pass1+pass2+pass3(仅输出处理后Bloom)  

<video src="" control></video>

pass0+pass1+pass2+pass3(仅输出处理后Bloom+原图)  

<video src="" control></video>

## 4.1.3 Demo演示  

### PS中的Bloom  

基础图层，叠加图层  

![[4ec7182d4e13d30140a571acf0373540_MD5.png]] 

![[4a757ffdf5b5f2224cb22b58855a2f56_MD5.png]]![[41a508fcf65a44d8d73590d218a19f55_MD5.png]]  

![[665e5ad22b8293048bdec90a5ab2fba7_MD5.png]] 

合并色阶1和叠加图层图层后，将模式改为线性叠加  

![[021597864dc62e07463825715b0e878c_MD5.png]] 

![[69afe2ec384f638015678eace834ace8_MD5.png]] 

模糊处理BloomAdd图层，这里使用盒型滤波（模糊），设置滤波核半径  

![[4da993124e619c2d9b94637b727414a3_MD5.png]] 

![[62e253e47107394b32aa57e11172ec99_MD5.png]] 

现在我们便有了bloom效果  

![[dfe466374c75b2a9e94a92d8a5c0526d_MD5.png]] 

增大半径使模糊效果更强，同时泛光效果也变强  

![[7307ef972790e7989db9544390955826_MD5.png]] 

![[65d4cdface87dbafa584827740a59d4a_MD5.png]] + ![[b27529756d9cc28f7bf9a0cf6af0c42c_MD5.png]]= ![[0ab6cfbd5beff837aa7b7cd79f74e3c0_MD5.png]] 

refer：[here](https://www.youtube.com/watch?v=tI70-HIc5ro)  

### bloom mask  

思路：利用alpha值（0或1）来选取bloom的区域  

在原shader的基础上，增加这个mask函数，并修改pass 3即最终叠加图像的片元着色器：  

 ```
		//src为原图颜色 color为叠加后颜色        
		//for bloom mask
        fixed4 mask(fixed4 src,fixed4 color)
        {
            return lerp(src,color,1.0-src.a);
        }

		fixed4 fragBloom(v2fBloom i) : SV_Target {
            //return tex2D(_Bloom, i.uv.zw);//for debug 仅输出处理后图像
            fixed4 orgin_img = tex2D(_MainTex, i.uv.xy); 
            fixed4 blur_img = tex2D(_Bloom, i.uv.zw);
            fixed4 result=orgin_img+blur_img;
			return mask(orgin_img,result);//原图与模糊图叠加
		}

``` 

lerp(a,b,w) 根据w返回a到b之间的插值相当于 fixed4 lerp(fixed4 a, fixed4 b, fixed4 w) { return a + w*(b-a); } 由此可见 当 w=0时返回a.当w = 1时 返回b.  

即渲染纹理中，alpha=1的部分将输出原图像，而alpha=0的部分将输出叠加后的bloom图像  

下图为测试场景（红胶囊alpha=1，蓝胶囊alpha=0）  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F17%2FC2rZuMwKgnSp9vR.gif&sign=240205723283474ce268072fe0e5a1bf37a7c6e115e84908718e22f80ad6f2e6)  

下图为一幅图片，设置alpha蒙版，用笔刷将天空部分alpha修改至半透明  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F18%2Fd4ycUIoSKphnvNJ.png&sign=a0a635d239cd87fdcb09aab86a83b283d4a087d5019011d603b1b9348005e7d6)  

无mask bloom（整体均受bloom效果影响）：  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F17%2F7qH98ZhUWfJ6yFO.png&sign=d1b87cff700cfd916ff431366b238be70754088c1d26af2447477276bf34996b)  

针对半透明部分，设置一个1-alpha的阈值（这里设为1e-1即0.1），具体请见下方  

 ```
	   fixed4 mask_chose1(fixed4 src,fixed4 color)//alpha=1时bloon有效
        {
            if(1e-1>1.0-src.a)
            {                
                return color;  
            }
            else
            {
                return src;
                         
            }
        }

        fixed4 mask_chose0(fixed4 src,fixed4 color)//alpha<1.0时bloom有效
        {
            if(1e-1>1.0-src.a)
            {                
                return src;  
            }
            else
            {
                return color;
                         
            }
        }

		fixed4 fragBloom(v2fBloom i) : SV_Target {
            //return tex2D(_Bloom, i.uv.zw);//for debug 仅输出处理后图像
            fixed4 orgin_img = tex2D(_MainTex, i.uv.xy); 
            fixed4 blur_img = tex2D(_Bloom, i.uv.zw);
            fixed4 result=orgin_img+blur_img;
            //return result;
			//return mask(orgin_img,result);//原图与模糊图叠加 lerp
			return mask_chose1(orgin_img,result);//原图与模糊图叠加 带alpha阈值判断
			//return mask_chose0(orgin_img,result);//原图与模糊图叠加 带alpha阈值判断
		}

``` 

使用mask_chose0，让半透明部分bloom  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F18%2F4zYMRB9qIwQ3pa1.gif&sign=2af5d3e7ce4b97c1f978e3b8da777956f3d21fd986930a4ea3c3198fab5f9e56)  

使用mask_chose1,让不透明部分bloom  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F18%2F2OCfeRLrUpSGPWu.gif&sign=9fbd1ab2b8681d01f4b02f6128c431aba7570ce6e7ae00d4a4e98aa801196581)  

（项目中的路径：[Assets/Packages/MaskBloom/Demo](https://github.com/logic-three-body/Unity_Bloom/tree/master/Assets/Packages/MaskBloom/Demo)）  

参考：[mattatz/unity-mask-bloom: Mask by alpha channel bloom effect for Unity. (github.com)](https://github.com/mattatz/unity-mask-bloom)  

[Unity3D Shader 之 lerp 函 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/73487722)  

### Gold Ray  

![](/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2021%2F08%2F18%2F7dVmRAbEajUh9uS.png&sign=942817a8175edb4bb455fa17c7f2424d3d0ec4a707b49944cc82218f1139396c)  

算法步骤：  

1.提取原图较量区域（阈值设定亮度）  

2.模糊（提取后）图像【径向模糊 ： 模拟光线往某方向的扩散效果】  

径向模糊的原理比较直接，首先选取一个径向轴心（Radial Center），然后将每一个采样点的uv基于此径向轴心进行偏移（offset），并进行一定次数的迭代采样，最终将采样得到的RGB值累加，并除以迭代次数。  

3.与原图混合  

演示：  

调节Light Transform 改变散射光线方向  

<video src="" control></video>

其他参数的调节  

<video src="" control></video>

cs部分：  

```
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
        targetCamera = GetComponent();
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
            // 根据阈值提取高亮部分,使用pass0进行高亮提取，比Bloom多一步计算光源距离剔除光源范围外的部分
            Graphics.Blit(src, buffer0, material, 0);

            material.SetVector("_ViewPortLightPos", new Vector4(viewPortLightPos.x, viewPortLightPos.y, viewPortLightPos.z, 0));
            material.SetFloat("_LightRadius", lightRadius);
            // 径向模糊的采样uv偏移值
            float samplerOffset = samplerScale / src.width;
            // 通过循环迭代径向模糊
            for (int i = 0; i < blurIteration; i++)
            {
                RenderTexture buffer1 = RenderTexture.GetTemporary(rtW, rtH, 0, src.format);
                float offset = samplerOffset * (i * 2 + 1);
                material.SetVector("_offsets", new Vector4(offset, offset, 0, 0));
                Graphics.Blit(buffer0, buffer1, material, 1);

                offset = samplerOffset * (i * 2 + 2);
                material.SetVector("_offsets", new Vector4(offset, offset, 0, 0));
                Graphics.Blit(buffer1, buffer0, material, 1);
                RenderTexture.ReleaseTemporary(buffer1);
            }

            material.SetTexture("_BlurTex", buffer0);
            material.SetVector("_LightColor", lightColor);
            material.SetFloat("_LightFactor", lightFactor);
            // 将径向模糊结果与原图进行混合
            Graphics.Blit(src, dest, material, 2);
            RenderTexture.ReleaseTemporary(buffer0);
        } else {
            Graphics.Blit(src, dest);
        }
    }

} 
``` 

shader部分：  

```
Shader "Unlit/GodRay"
{
    Properties
	{
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
 
	sampler2D _MainTex;
	float4 _MainTex_TexelSize;
	sampler2D _BlurTex;
	float4 _BlurTex_TexelSize;
	float4 _ViewPortLightPos;
	
	float4 _offsets;
	float4 _ColorThreshold;
	float4 _LightColor;
	float _LightFactor;
	float _PowFactor;
	float _LightRadius;
 
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
		// 仅当color大于设置的阈值的时候才输出
		float4 thresholdColor = saturate(color - _ColorThreshold) * distanceControl;
		float luminanceColor = Luminance(thresholdColor.rgb);
		luminanceColor = pow(luminanceColor, _PowFactor);
		return fixed4(luminanceColor, luminanceColor, luminanceColor, 1);
	}
 
	// 径向模糊VS
	v2fRadialBlur vertRadialBlur(appdata_img v)
	{
		v2fRadialBlur o;
		o.pos = UnityObjectToClipPos(v.vertex);
		o.uv = v.texcoord.xy;
		// 径向模糊采样偏移值*沿光的方向权重
		o.blurOffset = _offsets * (_ViewPortLightPos.xy - o.uv);
		return o;
	}
 

```