## 前言

在使用 GPU Instancing 技术有一个约束，必须使用相同材质和相同 Mesh 的对象才能使用 GPU Instancing，那都显示一样的对象就显的很无趣，有没有办法能让 GPU Instancing 中每个 Instance 有不同的表现呢？那当然有，这一节我会带大家对不同 Instance 的个性化属性进行学习。

相关测试工程传送门如下：

链接:[https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0](https://pan.baidu.com/s/1qoiiHGRbe_skt6Nercub8Q?pwd=9hf0) 提取码: 9hf0

## GPU Instanceing 相关章节传送门

[梅川依福：GPU Instancing 深入浅出 - 基础篇（1）](https://zhuanlan.zhihu.com/p/523702434)

[梅川依福：GPU Instancing 深入浅出 - 基础篇（2）](https://zhuanlan.zhihu.com/p/523765931)

[梅川依福：GPU Instancing 深入浅出 - 基础篇（3）](https://zhuanlan.zhihu.com/p/523924945)

[梅川依福：GPU Instancing 深入浅出 - 中级篇（1）](https://zhuanlan.zhihu.com/p/524195324)

[梅川依福：GPU Instancing 深入浅出 - 中级篇（2）](https://zhuanlan.zhihu.com/p/524285662)

## 一、明确目的

上一节我们的通过 GPU Instancing 技术通过 4（其中渲染 GPU 对象的只有 2 个批次）个批次渲染了 512 个对象，现在我们要让这些对象变的更有趣起来

![[d341614014987e737d601eb331e78ba6_MD5.jpg]]

![[9423951cf0c7c3cc78fa2c00a1c07ce5_MD5.gif]]

从两者比对来看，我们需要让每个 Cube Instance 有自己独立的颜色，同时这些 Instance 还能有自己的运动方式

## 二、所要使用的技术

### 1、什么是 MaterialPropertyBlock

![[cffff41a57f9d9db5ccdc3d03a2fb089_MD5.jpg]]

其实就是可以给每个实例对象通过 Render.SetPropertyBlock 设置相应的 MaterialPropertyBlock

### 2、官方测试代码

![[0156988a0f0a11108f95b9e830443eaf_MD5.jpg]]

```
using UnityEngine;

public class MaterialPropertyBlockExample : MonoBehaviour
{
    public GameObject[] objects;

    void Start()
    {
        //创建MaterialPropertyBlock
        MaterialPropertyBlock props = new MaterialPropertyBlock();
        MeshRenderer renderer;

        foreach (GameObject obj in objects)
        {
            float r = Random.Range(0.0f, 1.0f);
            float g = Random.Range(0.0f, 1.0f);
            float b = Random.Range(0.0f, 1.0f);
            //设置MaterialPropertyBlock所使用的颜色
            props.SetColor("_Color", new Color(r, g, b));
            //得到MeshRenderer
            renderer = obj.GetComponent<MeshRenderer>();
            //设置PropertyBlock
            renderer.SetPropertyBlock(props);
        }
    }
}
```

通过以上测试官方用例我们知道了如何给 Render 设置颜色

## 三、让我们的 Instance 变的有意思

### 1、增加相应 C# 脚本

相应 CubeCreate 传送门：[梅川依福：GPU Instancing 深入浅出 - 基础篇（2）](https://zhuanlan.zhihu.com/p/523765931)

```
using UnityEngine;

public class FunnyGPUInstance : MonoBehaviour
{
    [SerializeField]
    private GameObject _instanceGo;//初实例化对你
    [SerializeField]
    private int _instanceCount;//实例化个数
    [SerializeField]
    private bool _bRandPos = false;
 
    private MaterialPropertyBlock _mpb = null;//与buffer交换数据
    // Start is called before the first frame update
    void Start()
    {
        for (int i = 0; i < _instanceCount; i++)
        {
            Vector3 pos = new Vector3(i * 1.5f, 0, 0);
            GameObject pGO = GameObject.Instantiate<GameObject>(_instanceGo);
            pGO.transform.SetParent(gameObject.transform);
            if (_bRandPos)
            {
                pGO.transform.localPosition = Random.insideUnitSphere * 10.0f;
            }
            else
            {
                pGO.transform.localPosition = pos;
            }
            //个性化显示
            SetPropertyBlockByGameObject(pGO);

        }
    }

    //修改每个实例的PropertyBlock
    private bool SetPropertyBlockByGameObject(GameObject pGameObject)
    {
        if(pGameObject == null)
        {
            return false;
        }
        if(_mpb == null)
        {
            _mpb = new MaterialPropertyBlock();
        }

        //随机每个对象的颜色
        _mpb.SetColor("_Color", new Color(Random.Range(0f, 1f), Random.Range(0f, 1f), Random.Range(0f, 1f), 1.0f));
        _mpb.SetFloat("_Phi", Random.Range(-40f, 40f));

        MeshRenderer meshRenderer = pGameObject.GetComponent<MeshRenderer>();
        if (meshRenderer == null)
        {
            return false;         
        }

        meshRenderer.SetPropertyBlock(_mpb);

        return true;
    }
}
```

把脚本挂到

![[be52b736ff58a80a842c6e65804faeac_MD5.jpg]]

在随机对象中我们给 “_Color” 与“_Phi”设置了两组随机数值

```
_mpb.SetColor("_Color", new Color(Random.Range(0f, 1f), Random.Range(0f, 1f), Random.Range(0f, 1f), 1.0f));      
_mpb.SetFloat("_Phi", Random.Range(-40f, 40f));
```

从代码中我们可以看到我们给相应的对象设置了 MaterialPropertyBlock

```
meshRenderer.SetPropertyBlock(_mpb);
```

以上的_Color 与_Phi 是给 Material 中的 Shader 设置的参数

### 2、定义 Shader 的属性

在上一节中的 MyGPUInstance.Shader 中增加代码如下

上一节传送门 [梅川依福：GPU Instancing 深入浅出 - 中级篇（1）](https://zhuanlan.zhihu.com/p/524195324)

定义 Shader 的属性代码，此代码和 C# 的代码中的_mpb.SetColor("_Color",X,X,X)，_mpb.SetFloat("_Phi", Random.Range(-40f, 40f)); 配对使用

```
UNITY_INSTANCING_BUFFER_START(Props)
    UNITY_DEFINE_INSTANCED_PROP(float4,_Color)
    UNITY_DEFINE_INSTANCED_PROP(float, _Phi)
UNITY_INSTANCING_BUFFER_END(Props)
```

以上代码定义了一个 Props 的常量缓冲区，并且定义了 float4 的_Color 的属性与 float 的_Phi 属性

官方手册的解释

![[418b3554e80da54a773890acdd964f92_MD5.jpg]]

翻译

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th>宏的名称</th><th>描述</th></tr><tr><td>UNITY_INSTANCING_BUFFER_Start(bufferName)</td><td>在每个实例的开始处声明名为 bufferName 的常量缓冲区。将此宏与 UNITY_NSTANCING_BUFFER_END 一起使用，可以包装要对每个实例唯一的属性声明。使用 UNITY_DEFINE_INSTANCED_PROP 声明缓冲区内的属性。</td></tr><tr><td>UNITY_INSTANCING_BUFFER_END(bufferName)</td><td>在每个实例的结尾处声明名为 bufferName 的常量缓冲区。将此宏与 UNITY_INSTANCING_BUFFER_START 一起使用，可以包装要对每个实例唯一的属性声明。使用 UNITY_DEFINE_INSTANCED_PROP 声明缓冲区内的属性。</td></tr><tr><td>UNITY)DEFINE_INSTANCED_PROP(type, propertyName)</td><td>使用指定的类型和名称定义每个实例着色器属性。在以下示例中，_Color 属性是唯一的。(可以上面的官方测试代码)</td></tr><tr><td>UNITY_ACCESS_INSTANCED_PROP</td><td></td></tr></tbody></table>

### 3、使用属性

C# 的脚本代码通过 Setcolor 或是 SetFloat 传递到 Shader 中，那在 Shader 中是如何使用的呢

在顶点着色器中通过 UNITY_ACCESS_INSTANCED_PROP(Props, _Phi); 进行属性访问

![[a7b2e289e960343a14fbaf6b14567ab1_MD5.png]]

翻译：在一个实例常量缓冲区中访问每个实例着色器属性。Unity 使用 Instance ID 索引实例数据数组。bufferName 必须与包含指定属性的常量缓冲区的名称匹配。此宏的编译方式对于 Instance_ON 和非 Instance 变体编译不同。

顶点着色器的代码如下，通过得到 _Phi 来让对象有不同的偏移值

```
v2f vert (appdata v) {
   v2f o;
   //第四步：instanceid在顶点的相关设置 
   UNITY_SETUP_INSTANCE_ID(v);
   //第五步：传递 instanceid 顶点到片元
   UNITY_TRANSFER_INSTANCE_ID(v, o);

   float phi = UNITY_ACCESS_INSTANCED_PROP(Props, _Phi);
   v.vertex = v.vertex + sin(_Time.y + phi);

   o.vertex = UnityObjectToClipPos(v.vertex);
   o.uv = TRANSFORM_TEX(v.uv, _MainTex);
   UNITY_TRANSFER_FOG(o,o.vertex);
   return o;
 }
```

片元着色器代码如下

得到 C# 的设置给 Shader 的 UNITY_ACCESS_INSTANCED_PROP(Props, _Color); 颜色，并让每个片元显示这个颜色值

```
fixed4 frag (v2f i) : SV_Target
{
   //第六步：instanceid在片元的相关设置
   UNITY_SETUP_INSTANCE_ID(i);

   //得到由CPU设置的颜色
   float4 col= UNITY_ACCESS_INSTANCED_PROP(Props, _Color);
   return col;   
}
```

与 FunnyGPUInstance 的代码

```
Shader "Unlit/FunnyGPUInstance"
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

            UNITY_INSTANCING_BUFFER_START(Props)
                UNITY_DEFINE_INSTANCED_PROP(float4,_Color)
	      	    UNITY_DEFINE_INSTANCED_PROP(float, _Phi)
            UNITY_INSTANCING_BUFFER_END(Props)

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

                float phi = UNITY_ACCESS_INSTANCED_PROP(Props, _Phi);
                v.vertex = v.vertex + sin(_Time.y + phi);

                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                UNITY_TRANSFER_FOG(o,o.vertex);

              
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //第六步：instanceid在片元的相关设置
                UNITY_SETUP_INSTANCE_ID(i);

                //得到由CPU设置的颜色
                float4 col= UNITY_ACCESS_INSTANCED_PROP(Props, _Color);
                return col;
            }
            ENDCG
        }
    }
}
```

### 4、制作 FunnyGPUInstanceCube 的 prefab

![[38b21a3c19c70dcccfe1a9a991c44efa_MD5.jpg]]

把相应的 Prefab 拖放到 FunnyGPUInstanceCube 中

![[2d4930f99c66f51d3a46981f5db38d8a_MD5.jpg]]

### 5、测试效果

![[5617d7d6022185f2adacfc566a9aa0bd_MD5.jpg]]

Batches 为 3，并且动起来了，目标达成

## 四、总结

通过 MatermialPropertyBlock 我们通过 C# 代码给每个实例对象进行了属性设置，把一个死寂沉沉的 GPUInstancing 用例变的有趣起来。当然所有的这一切还是归功于 Shader 与 C# 的代码的配合。