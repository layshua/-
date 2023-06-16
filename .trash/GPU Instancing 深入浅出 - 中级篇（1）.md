在初期篇中我们对 GPU Instancing 的使用有了一个基础的了解，知道了其简单的原理，以及简单的应用，甚至是一些在使用过程中的约束。

进入中级篇，之前是写给小白们看的，当前一些老司机可以通过中级篇开始上路了。

在中级篇中我们将会看

1、创建带有 Enable GPU Instancing 的 Shader  
1、创建带有 Enable GPU Instancing 的 Shader

2、使用 MaterialPropertyBlock 让 GPU Instancing 变的更有趣  
2、使用 MaterialPropertyBlock 让 GPU Instancing 变的更有趣

3、使用 Graphic.DrawMeshInstanced 进行无 GameObject 的 Instance 对象创建

相关测试工程传送门如下：

链接:[https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0](https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0) 提取码: 9hf0  
链接: https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0 提取码: 9hf0

跟着文章由浅入深，相信只要看完整个系列的文章就能轻松搞定在大规模的程序开发中使用 GPU Instancing 的问题，本文主要来源一个小众《元宇宙》项目。

## GPU Instanceing 相关章节传送门

[梅川依福：GPU Instancing 深入浅出 - 基础篇（1）](https://zhuanlan.zhihu.com/p/523702434)

[梅川依福：GPU Instancing 深入浅出 - 基础篇（2）](https://zhuanlan.zhihu.com/p/523765931)

[梅川依福：GPU Instancing 深入浅出 - 基础篇（3）](https://zhuanlan.zhihu.com/p/523924945)

[梅川依福：GPU Instancing 深入浅出 - 中级篇（1）](https://zhuanlan.zhihu.com/p/524195324)

[梅川依福：GPU Instancing 深入浅出 - 中级篇（2）](https://zhuanlan.zhihu.com/p/524285662)

## 一、目的

### 1、Unity 默认支持

其实在 Unity 的默认渲染管线中的 Standard Shader 中默认都支持了 Enable GPU Instancing

![[4adf22ae58d521cfb658cfcb27b5ba6f_MD5.jpg]]

那为什么我们需要还需要自己在 Shader 中重新支持 Enable GPU Instancing 呢？其实在 SRP 中或是 URP 中或是自己定义的 Shader 中其实默认都不支持 Enable GPU Instancing，所以在写 Shader 的时候我们都会自己加入 GPU Instacing 的支持。对于 GPU Instancing 的支持 Unity 提供了一系列的宏，让用户自定义的 Shader 可以轻松支持 GPU Instancing。

### 2、Unity 手册描述

[Creating shaders that support GPU instancing - Unity 手册  
创建支持 GPU 实例化的着色器 - Unity 手册](https://docs.unity.cn/cn/current/Manual/gpu-instancing-shader.html)

![[18ae2d020b1278183775742e50973598_MD5.jpg]]

翻译

创建支持 GPU Instancing 的 Shader

本节内容包括了如何给用户自定义的 Shader 增加支持 GPU Instancing 的功能。本文首先介绍了自定义 Unity 着色器支持 GPU 实例化所需的 Shader 关键字、变量和函数。然后，本文内容还包括如何向曲面着色器和顶点 / 片段着色器添加逐实例数据的示例。

所以建议感兴趣的也可以先看看官方手册。

## 一、创建 Unlit 的自定义 Shader 材质球

下载测试工程的话可以在 3_MyInstanceShader 中进行学习

![[8f9e8cfda7dac0b5ca094f7dc9031757_MD5.jpg]]

### 1、创建 Unlit Shader 1、创建无光照着色器

![[ae6d234d704c7c0e61bd808c84b52b2d_MD5.jpg]]

然后左键 NewUnlitShader

![[d46f0c510d12936e2101b2514bd11995_MD5.jpg]]

### 2、创建材质 Unlit 材质球

如下创建使用 NewUnlitShader 右键创建 Material

![[42a8c3b78e1280a24ba50ee88f2bbfc1_MD5.jpg]]

创建出的 Material

![[ceae30fdb516a7a4f8008451a4b86681_MD5.jpg]]

没有相应的 Enable GPU Instancing 的选项。

那要如何才能快速支持 Enable GPU Instancing 呢？

## 二、让材质球支持 GPU Instancing

### 1、给我们的 Shader 命个名

需要给自定义的 Shader 定义一个自己的名字，双击 Shader 后对开始 Shader 的编写

![[66f4f289aa465ee44f3492943cad1f0c_MD5.png]]

```
Shader "Unlit/MyGPUInstance"
```

### 2、第一步：增加变体让 Shader 支持 instance

增加变体使用 Shader 可以支持 Instance

![[1ade084cd809ceeae4aa897287e3a151_MD5.jpg]]

```
//第一步： sharder 增加变体使用shader可以支持instance 
#pragma multi_compile_instancing
```

以上代码将使 Unity 生成着色器的两个变体，一个具有 GPU 实例化支持，一个不具有 GPU 实例化支持。

到我们的材质球上看看有什么变化

![[2465998150d1c7987ab8b3ace241afb9_MD5.jpg]]

是不是很神奇

官方手册说明

![[543d64922e959a11a6dc7941ffe75ce1_MD5.png]]

翻译：生成 instance 变体。这对于片段和顶点着色器是必需增加的。对于曲面着色器，它是可选的。

### 3、第二步 - 添加顶点着色器输入宏

instancID 加入顶点着色器输入结构

![[cea41d10d6d45be9791a5061a71d4ba2_MD5.jpg]]

```
//第二步：instancID 加入顶点着色器输入结构 
  UNITY_VERTEX_INPUT_INSTANCE_ID
```

宏翻译后如下其实就是增加了一个 SV_InstanceID 语义的 instanceID 变量

#define UNITY_VERTEX_INPUT_INSTANCE_ID unit instanceID : SV_InstanceID  
#define UNITY_VERTEX_INPUT_INSTANCE_ID 单位实例 ID ： SV_InstanceID

instanceID 主要作用是使用 GPU 实例化时，用作顶点属性的索引。

官方手册说明

![[c0cadec7e7369ea1a3abb2b941554673_MD5.jpg]]

翻译：在顶点着色器输入 / 输出结构体中定义 instance ID。要使用此宏，请启用 IINSTANCING_ON 关键字。否则，Unity 不会设置 instance ID。要访问 instance ID，请使用 #ifdef INSTANCING_ON 中的 vertexInput.instanceID 。如果不使用此块，变体将无法编译。

### 4、第三步 - 添加顶点着色器输出宏

instancID 加入顶点着色器输出结构

![[0ed25b5dddb2d2bd3a0c4e48171b071d_MD5.jpg]]

如第二步一样目的是增加一个 SV_InstanceID 语义的 nstanceID 变量，用作顶点属性的索引。

```
//第三步：instancID 加入顶点着色器输出结构 
  UNITY_VERTEX_INPUT_INSTANCE_ID
```

### 5、第四步 - 得到 instanceid 顶点的相关设置

![[d72f0899fa30002e98815ec680bea960_MD5.jpg]]

```
//第四步：instanceid在顶点的相关设置 
UNITY_SETUP_INSTANCE_ID(v);
```

#define UNITY_SETUP_INSTANCE_ID(input) \  
#define UNITY_SETUP_INSTANCE_ID（输入） \

unity_InstanceID = input.instanceID + unity_BaseInstanceID;  
unity_InstanceID = input.instanceID + unity_BaseInstanceID;

官方文档

![[407c58a4f1c43cc28bb0587a02f10cf1_MD5.jpg]]

翻译：允许着色器函数访问实例 ID。对于顶点着色器，开始时需要此宏。对于片段着色器，此添加是可选的。有关示例，请参见顶点和片段着色器。

### 6、第五步 - 传递 instanceID 顶点到片元角色器

![[5816b11bf52c294fff6957000cc6bb8b_MD5.jpg]]

```
//第五步：传递 instanceid 顶点到片元
UNITY_TRANSFER_INSTANCE_ID(v, o);
```

官方手册

![[c3a50e88e117c33b1db5879150767e63_MD5.png]]

翻译：在顶点着色器中将 InstanceID 从输入结构复制到输出结构。如果需要访问片段着色器中的每个实例数据，请使用此宏。

### 7、第六步 instanceID 在片元的相关设置

![[95d95fd746889e9716b84071b86f5372_MD5.jpg]]

```
//第六步：instanceid在片元的相关设置
UNITY_SETUP_INSTANCE_ID(i);
```

### 8、代码全展示

经过以上六个步骤后我们写成了自己带 GPU Instancing 的 Shader(MyGPUInstance)

```
Shader "Unlit/MyGPUInstance"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            CGPROGRAM
            //第一步： sharder 增加变体使用shader可以支持instance 
            #pragma multi_compile_instancing

            #pragma vertex vert
            #pragma fragment frag
            // make fog work
            #pragma multi_compile_fog

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;

                //第二步：instancID 加入顶点着色器输入结构 
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                UNITY_FOG_COORDS(1)
                float4 vertex : SV_POSITION;
                //第三步：instancID加入顶点着色器输出结构
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            v2f vert (appdata v)
            {
                v2f o;
                //第四步：instanceid在顶点的相关设置 
                UNITY_SETUP_INSTANCE_ID(v);
                //第五步：传递 instanceid 顶点到片元
                UNITY_TRANSFER_INSTANCE_ID(v, o);

                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                UNITY_TRANSFER_FOG(o,o.vertex);
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //第六步：instanceid在片元的相关设置
                UNITY_SETUP_INSTANCE_ID(i);
                // sample the texture
                fixed4 col = tex2D(_MainTex, i.uv);
                // apply fog
                UNITY_APPLY_FOG(i.fogCoord, col);
                return col;
            }
            ENDCG
        }
    }
}
```

## 三、一个带有 GPU instancing 的测试

### 1、创建 Prefab 并使用相应材质

命名：Prefba 为 MyGPUInstanceCube  
命名：Prefba 为 MyGPUInstanceCube

![[bd875511a3e414eae49d9677767cd2d6_MD5.jpg]]

按以上截图创建相应的 Prefab

### 2、挂上之前的 CreateCube 脚本

并在 Instance GO 中使用 MyGPUInstanceCube 的 Prefba  
并在 Instance GO 中使用 MyGPUInstanceCube 的 Prefba

![[041f582d981e9a6b970d330e74758af9_MD5.jpg]]

CreatCube 代码传送门：[梅川依福：GPU Instancing 深入浅出 - 基础篇（2）](https://zhuanlan.zhihu.com/p/523765931)

### 3、来测试一下

![[0a72535d8bfcc0c12efd82eec8a8b159_MD5.jpg]]

达成目标，批次为 4 个批次，优化了 510 个 Batcing.

## 四、总结

本节我们使用 Unity 的 Ulit 的 Shader 创建了我们自定义的 MyGPUInstance.shader，并通过六步依次添加了宏完成了自己定义的 Shadr 支持 GPU instancing 的制作。然后通过创建一个材质并在 Cube 中使用了本材质，通过 createCube 批量创建了 512 个 Cube 只使用了 2 个批次渲染了 512 个 Cube 对象。

但是我们测试用例真的比较丑，那我们是不是需要让这些白盒看起来好看一些呢？下一节我们会让我们的白盒子变的更有趣一些，不然我自己都看不下去了。