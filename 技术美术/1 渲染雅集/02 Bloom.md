# 4.1 Bloom
## 一、什么是 Bloom 算法
### 1、首先看一下 Bloom 效果长什么样
![[Pasted image 20221209110858.png]]
![[Pasted image 20221209110901.png]]
![[Pasted image 20221209110903.png]]
### 2、什么是 Bloom

-   Bloom，也称辉光，是一种常见的屏幕效果
-   模拟摄像机的一种图像效果，让画面中较亮的区域“扩散”到周围的区域中，造成一种朦胧的效果
-   可以让物体具有真实的明亮效果
-   可以实现光晕效果

### 3、Bloom 的实现原理
#### ①Bloom 实现原理

-   实现思路：

-   1. 提取原图较亮区域（利用阈值）
-   2. 模糊该图像
-   3. 与原图混合/叠加
- ![[Pasted image 20221209110919.png]]
- //在 HDR 和 LDR 的那节课中也提到过 bloom，我直接把当时的作的一张流程图摘过来以供参考
 ![[Pasted image 20221209111030.png]]
#### ②前置知识 1：HDR 和 LDR
[[2 光照基础#2.7 LDR和HDR]]
-   HDR 和 LDR 分别是是高动态范围和低动态范围的缩写
-   **LDR**
-   jpg、png 格式图片
-   RGB 范围在[0,1]

-   **HDR**
-   HDR、EXR 格式图片
-   可以超过 1

-   因为自然界中的亮度差异是很大的（比如蜡烛的光强度约为 15，而太阳的强度约为 10w），只用 LDR 的话很多效果完全表现不出来
-   //其他具体细节参考 2.7 节

#### ③前置知识 2：高斯模糊

-   实现图像模糊的一种方式
-   **高斯模糊：**
-   <font color="#ff0000">利用高斯核进行卷积运算，得到模糊的图像</font>
![[Pasted image 20221209111419.png]]
-   高斯核：通过高斯函数定义的**卷积核**
-   核中心：(0,0)
-   核大小：3x3
-   标准方差σ（sei ge ma）：1.5

-   计算步骤：
	- 将（x，y）带入公式中，计算出权重值，**（权重值代表当前处理像素的影响程度，离中心越近权重越大，影响程度越大）**
	- 为了保证卷积后图像不变暗，需要对高斯核进行归一化处理（每个权重除以所有权重的和）
	- ![[Pasted image 20221209111535.png]]

**二维高斯核的特点**
-   计算量大，N×N 的高斯核需要 N * N * W * H 次纹理采样（图像的宽度和高度分别为 W、H）。

**二维高斯核的可分离性**
-   二维高斯核可以拆成两个一维高斯核
-   利用可分离性，我们就可以优化算法
	-   我们可以用两个一维高斯核先后对图像进行两次卷积操作
	-   这样一来，结果一样，采样次数变为了 2 * N * W * H
-   再进一步
	-   一维高斯核中包括了很多重复的权重，即对称性（下例中 0.0545，0.02442）
	-   下例中大小为 5 的高斯核，<font color="#ff0000">实际上只需要记录三个权重值即可（0.0545、0.2442、0.4026）</font>

![[Pasted image 20221209111755.png]]

#### 卷积

-   **课程内容**
-   是一种图像操作
-   利用“卷积核”对图像的每个像素进行一系列操作

-   **卷积核**：
	-   通常是由四方形网格结构，该区域内每个放个都有一个权重值
	-   当我们对图像中的像素进行卷积时：
	-   会把卷积核的中心放置在该像素上
	-   翻转核之后再依次计算核中每个元素和其覆盖的图像像素值的乘积并求和
	-   得到的结果就是该位置的新像素值

-   一个例子：
![[卷积.gif]]
-   计算步骤：先水平反转卷积核，再将位置一一对应求和

-   **补充：**
-   参考：GAMES101-L6 内容
-   **滤波（Filtering）**
	-   滤波就是抹掉特殊频率的东西
	-   不同滤波的效果：
		-   高通滤波 = 边界
		-   低通滤波 = 模糊
![[Pasted image 20221209112711.png]]
-   **滤波（Filtering）=卷积（Convolution）=平均（Averaging）**
-   **卷积操作的定义**
-   ①原始信号的任意一个位置，取其周围的平均
-   ②作用在一个信号上，用一种滤波操作，得到一个结果 Result

![[Pasted image 20221209112739.png]]
-   **卷积定理**
-   时域上的卷积就是频域上的乘积（时域上的乘积=频域上的卷积）

-   根据卷积定理，我们实现一个卷积操作可以有两种方法：
-   方法 1：图和滤波器直接在时域上做卷积操作
-   方法 2：先把图傅里叶变换，变换到频域上，把滤波器变到频域上，两者相乘；乘完之后再逆傅里叶变换到时域上
- ![[Pasted image 20221209113030.png]]
### 4、总结

①我们想用 bloom 实现什么效果？
-   自然界中亮度差异较大的效果，LDR 不能表现的，光晕的效果

②实现 bloom 效果的步骤？
-   先利用阈值提取图像中较亮的区域
-   对这个区域做模糊
-   将模糊后的高亮部分叠加回原图

## 二、用 Unity 实现 Bloom 算法
![[1 4.gif]]
### 1、C #脚本部分
-   **基本思路**：
-   调用 OnRenderImage 函数，得到渲染纹理
-   将纹理和相关参数传入 shader

-   **具体内容**：
-   将基类改为 PostEffectsBase（为我们提供了检查 shader 材质的方法，用来检查 shader 材质是否可用（其实就是 Valid））
-   声明这个脚本使用的 shader、材质

-   定义 shader 中的相关参数
-   高斯模糊迭代次数
-   高斯模糊范围（blurSpread）
-   下采样系数（downSample，downSample 控制渲染纹理大小）
-   阈值

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

```c
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
-   pass4：模糊后的高亮区域叠加到原图
-   //只是为了更理解原理，结合脚本用一个 pass 同样可以实现

-   **具体内容**

-   声明相关参数
-   渲染状态：

-   开启深度测试，关闭剔除，关闭深度写入

-   定义四个 pass 相关函数名
-   引入相关头文件
-   声明相关变量
-   剩下就是各个 pass 需要的输入输出结构体和顶点像素着色器
-   **整理一下其中需要注意的几点**

-   **亮度如何获取？**

-   亮度计算公式

-   **较亮区域如何提取？**

-   采样后获取亮度值，再减去阈值，最后用 clamp 截取

-   **如何对竖直/水平方向进行高斯模糊？**

-   **在顶点着色器中计算 uv**

-   在 vertex shader 中计算的好处：

-   计算量 (次数)少（一般情况下顶点数量<像素数量）
-   在顶点着色器中计算纹理坐标可以减少运算提高性能
-   而且由于顶点到片元的插值是线性的，因此不会影响纹理坐标的计算结果

-   用 5 维数组存 5 个 uv，其中**uv0 位当前纹理坐标，其他四个是高斯模糊对邻域采样时用到的纹理坐标**
-   uv0 就是纹理的坐标，直接从输入结果传过来即可
-   邻域 uv 的计算：

-   以 uv1 为例，假设 uv1 是（0,1），也就是在基础 uv 向上移动
-   对于移动的距离，我们要利用到模糊范围
-   同理，uv2 向下就是-1，uv3 就是向上两个单位，uv4 是向下两个单位
-   相同的思路，将竖直的 y 改为 x 就能给水平方向用

-   **在像素着色器中进行模糊**

-   用数组存需要的三个权重值
-   定义 sum 变量用来存模糊后的像素值，并通过纹理采样获取像素值
-   模糊操作：

-   用 for 循环进行卷积运算

-   **混合时有哪些注意点？**

-   顶点着色器：

-   uv 的 xy 是渲染纹理坐标，zw 是模糊后的纹理坐标
-   考虑平台差异处理

-   如果 y 是负的，就进行反转操作

-   片元着色器：

-   直接返回 uv 采样的 xy、zw 之和即可

-   还有一点要注意

-   bloom 模拟的是相机的一种图像处理效果，所以 C #脚本给的是camera
```c
Shader "Unlit/DS_Bloom"
{
    Properties
    {
        // _MainTex为渲染纹理，变量名固定不能改变
        //模糊结果、阈值、模糊半径的变量名与C#脚本中的对应
        _MainTex ("Texture", 2D) = "white" {}
        _Bloom ("Bloom (RGB)", 2D) = "black" {} //高斯模糊后的结果
		_LuminanceThreshold ("Luminance Threshold", Float) = 0.5 //阈值
		_BlurSize ("Blur Size", Float) = 1.0 //模糊半径
    }
    SubShader
    {
        //用CGINCLUDE和ENDCG
        //Unity会把它们之间的代码插入到每一个pass中，已达到声明一遍，多次使用的目的。
        CGINCLUDE
        #include "UnityCG.cginc"

        //声明属性和C#脚本中用到的变量
        sampler2D _MainTex;
		half4 _MainTex_TexelSize;//纹素大小
		sampler2D _Bloom;
		float _LuminanceThreshold;
		float _BlurSize;

        //########第1个pass使用########
        //输出结构
        struct v2fExtractBright {
			float4 pos : SV_POSITION; 
			half2 uv : TEXCOORD0;
		};
        
        //顶点着色器
        v2fExtractBright vertExtractBright(appdata_img v) {
        	//appdata_img是官方提供的输入结构，只包含图像处理时必须的顶点坐标和uv等变量
			v2fExtractBright o;
			o.pos = UnityObjectToClipPos(v.vertex);
			o.uv = v.texcoord;	 
			return o;
		}
        
        // 明亮度公式
		fixed luminance(fixed4 color) {
        	//计算得到像素的亮度值
			return  0.2125 * color.r + 0.7154 * color.g + 0.0721 * color.b; 
		}
        
        //片元着色器->提取高亮区域
        fixed4 fragExtractBright(v2fExtractBright i) : SV_Target {
        	fixed4 c = tex2D(_MainTex, i.uv);// 贴图采样
			
			fixed val = clamp(luminance(c) - _LuminanceThreshold, 0.0, 1.0);
        	// 调用luminance得到采样后像素的亮度值，再减去阈值
			// 使用clamp函数将结果截取在[0,1]范围内
        	
			return c * val;
        	// 将val与原贴图采样得到的像素值相乘，得到提取后的亮部区域
		}

        
        //########第2、3个pass使用########
        //输出结构
        struct v2fBlur {
			float4 pos : SV_POSITION;
        	half2 uv[5]: TEXCOORD0;
			// 此处定义5维数组用来计算5个纹理坐标
        	// 由于卷积核大小为5x5的二维高斯核可以拆分两个大小为5的一维高斯核
        	// uv[0]存储了当前的采样纹理
        	// uv[1][2][3][4]为高斯模糊中对邻域采样时使用的纹理坐标
		};
        
        //顶点着色器->计算竖直方向进行高斯模糊的uv
        v2fBlur vertBlurVertical(appdata_img v) {
			
			v2fBlur o;
			o.pos = UnityObjectToClipPos(v.vertex);//将顶点从模型空间变换到裁剪空间下
			half2 uv = v.texcoord;
			o.uv[0] = uv;
        	//uv[0]就是（0,0）
        	//对竖直方向进行模糊
			//对应到邻域就是下边的情况
        	//uv[1]，向上挪动1个单位(0, 1)
        	//uv[2]，向下挪动1个单位(0, -1)
        	//uv[3]，向上挪动2个单位(0, 2)
        	//uv[3]，向下挪动2个单位(0, -2)
        	//最后乘上模糊半径作为参数控制
			o.uv[1] = uv + float2 (0.0, _MainTex_TexelSize. y * 1.0) * _BlurSize;
			o.uv[2] = uv - float2 (0.0, _MainTex_TexelSize. y * 1.0) * _BlurSize;
			o.uv[3] = uv + float2(0.0, _MainTex_TexelSize.y * 2.0) * _BlurSize;
			o.uv[4] = uv - float2(0.0, _MainTex_TexelSize.y * 2.0) * _BlurSize;
					 
			return o;
		}
        
        //顶点着色器->计算水平方向进行高斯模糊的uv
        v2fBlur vertBlurHorizontal(appdata_img v) {
			v2fBlur o;
			o.pos = UnityObjectToClipPos(v.vertex);
			half2 uv = v.texcoord;
        	o.uv[0] = uv;
        	//uv[0]就是（0,0）
        	//对水平方向进行模糊
			//同理，uv[1]到[4]分别对应(1, 0)、(-1, 0)、(2, 0)、(-2, 0)
			o.uv[1] = uv + float2(_MainTex_TexelSize.x * 1.0, 0.0) * _BlurSize;
			o.uv[2] = uv - float2(_MainTex_TexelSize.x * 1.0, 0.0) * _BlurSize;
			o.uv[3] = uv + float2(_MainTex_TexelSize.x * 2.0, 0.0) * _BlurSize;
			o.uv[4] = uv - float2(_MainTex_TexelSize.x * 2.0, 0.0) * _BlurSize;
					 
			return o;
		}
        //片元着色器->进行高斯模糊
        fixed4 fragBlur(v2fBlur i) : SV_Target {
			
			float weight[3] = {0.4026, 0.2442, 0.0545};
        	// 因为二维高斯核具有可分离性，而分离得到的一维高斯核具有对称性
			// 所以只需要在数组存放三个高斯权重即可
			
			fixed3 sum = tex2D(_MainTex, i.uv[0]).rgb * weight[0];
        	// 结果值sum初始化为当前的像素值乘以它对应的权重值

        	// 进行卷积运算，根据对称性完成两次循环
				// 第一次循环计算第二个和第三个格子内的结果
				// 第二次循环计算第四个和第五个格子内的结果
			for (int it = 1; it < 3; it++) {
				sum += tex2D(_MainTex, i.uv[it*2-1]).rgb * weight[it];
				sum += tex2D(_MainTex, i.uv[it*2]).rgb * weight[it];
			}
			
			return fixed4(sum, 1.0);// 返回滤波后的结果
		}

        
        //########第4个pass使用########
        //输出结构
        struct v2fBloom {
			float4 pos : SV_POSITION; 
			half4 uv : TEXCOORD0;
		};

        //顶点着色器
        v2fBloom vertBloom(appdata_img v) {
			v2fBloom o;
			o.pos = UnityObjectToClipPos (v.vertex);
			
			o.uv.xy = v.texcoord; //xy分量为_MainTex的纹理坐标		
			o.uv.zw = v.texcoord; //zw分量为_Bloom的纹理坐标
			
			// 平台差异化处理
        	//判断y是否小于0，如果是就进行翻转处理
			#if UNITY_UV_STARTS_AT_TOP			
			if (_MainTex_TexelSize.y < 0.0)
				o.uv.w = 1.0 - o.uv.w;
			#endif
			return o; 
		}

        //片元着色器->混合亮部和原图
        fixed4 fragBloom(v2fBloom i) : SV_Target {
		    // 把这两张纹理的采样结果相加即可得到最终效果
			return tex2D(_MainTex, i.uv.xy) + tex2D(_Bloom, i.uv.zw);
		}
    	ENDCG
    	
        // 开启深度测试，关闭剔除和深度写入
        ZTest Always 
    	Cull Off 
    	ZWrite Off
        
    	//第一个pass，提取较亮区域
        Pass{
            CGPROGRAM
			#pragma vertex vertExtractBright
			#pragma fragment fragExtractBright
            ENDCG
        }
	    
    	//第二个 pass，进行竖直方向高斯模糊
    	Pass{
            CGPROGRAM
			#pragma vertex vertBlurVertical
			#pragma fragment fragBlur
            ENDCG
        }
    	
    	//第三个 pass，进行水平方向高斯模糊
    	Pass{
            CGPROGRAM
			#pragma vertex vertBlurHorizontal
			#pragma fragment fragBlur
            ENDCG
        }
    	
    	//第四个pass，混合高亮区域和原图
    	Pass{
            CGPROGRAM
			#pragma vertex vertBloom
			#pragma fragment fragBloom
            ENDCG
        }
    }
	FallBack Off
}
```

## 三、 Bloom 算法的应用

### 1、配合自发光贴图
![[Pasted image 20221209114223.png]]
### 2、配合特效
![[Pasted image 20221209114240.png]]
### 3、配合 ToneMapping
-   bloom 效果和 ToneMapping 结合可以较好的保留暗部和亮部的细节
![[Pasted image 20221209114250.png]]
### 4、GoodRay 效果
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

### 其他资料扩展/同学笔记
-   十种图像模糊像算法的总结与实现-------毛星云
-   [https://zhuanlan.zhihu.com/p/125744132](https://zhuanlan.zhihu.com/p/125744132)

-   SRP 做 Mask---by 丷葉丷
-   [https://quaint-author-3ce.notion.site/Bloom-1b2f32f3502c471a867a4ca07d27306d](https://quaint-author-3ce.notion.site/Bloom-1b2f32f3502c471a867a4ca07d27306d)

-   CommandBufffer 做 Mask-------by 清盐
-   [https://www.yuque.com/qingyan-bng85/zagg8x/epl3qs](https://www.yuque.com/qingyan-bng85/zagg8x/epl3qs)
## 五、Reference

-   图片参考：· [https://unsplash.com/·](https://unsplash.com/·)
-   自行拍摄项目参考：· [https://github.com/keijiro/KinoBloom·](https://github.com/keijiro/KinoBloom·) [https://github.com/MarcusXie3D/FastBloomForMobiles](https://github.com/MarcusXie3D/FastBloomForMobiles)
-   资料参考：

-   learnopengl：[https://learnopengl.com/Advanced-Lighting/Bloom·](https://learnopengl.com/Advanced-Lighting/Bloom·)
-   Unity Shader 入门精要：12.5 Bloom 效果·
-   [ https://en.wikipedia.org/wiki/Bloom_ (shader_effect)]( https://en.wikipedia.org/wiki/Bloom_ (shader_effect)·)
-   [https://en.wikipedia.org/wiki/High_dynamic_range](https://en.wikipedia.org/wiki/High_dynamic_range·)
-   [https://zhuanlan.zhihu.com/p/76505536](https://zhuanlan.zhihu.com/p/76505536)
