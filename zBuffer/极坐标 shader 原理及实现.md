我们常用笛卡尔二维坐标系统，极坐标是一种坐标系统，极坐标是**用距离和角度**来表示二维坐标中的一个点。

ShaderGraph  Polar Coordinates 节点：[极坐标官方结点文档](https://docs.unity3d.com/Packages/com.unity.shadergraph@12.0/manual/Polar-Coordinates-Node.html)

今天说的这个极坐标结点是 Unity 内部实现，通过 UV 坐标（其实就是笛卡尔二维坐标）转换为极坐标表示。  

![[213a69bbdbedbc30c75fb2c008ab54a4_MD5.jpg|500]]

上图就是 Unity 实现的极坐标：R 通道表示距离，G 通道表示角度。

官方数学实现：

```c
void Unity_PolarCoordinates_float(float2 UV, float2 Center, float RadialScale, float LengthScale, out float2 Out) 
{
    float2 delta = UV - Center;
    float radius = length(delta) * 2 * RadialScale;
    float angle = atan2(delta.x, delta.y) * 1.0/6.28 * LengthScale;
    Out = float2(radius, angle);
}
```


相比于 UV 坐标，极坐标则是直接将坐标系改了，由默认 UV 两轴坐标定义的方形笛卡尔直角坐标系，转换成了由弧度、半径两个坐标定义的圆形极坐标系，在这基础上进行动态偏移的效果。

首先看下面的图

![|500](https://www.yumefx.com/wp-content/uploads/2021/10/PolarCoord_02.jpg)

对于图中的 A 点，在传统的笛卡尔直角坐标系中表示为 (x,y)，其中 x 值为 A 点在 x 轴的投影长度，y 值为 A 点在 y 轴的投影长度，A 的位置范围为矩形。

而如果使用极坐标系，则 A 点表示为 (ρ,θ)，其中ρ值为 A 点距离坐标原点 O 的距离，θ值为 AO 与 x 轴正方向的夹角，A 的位置范围为圆形。

两者的转化方式图上已经给出，很容易看懂就不赘述了。

那么我们来看一下，在 Shader 中使用默认 UV 和极坐标时，效果的区别。

![](https://www.yumefx.com/wp-content/uploads/2021/10/PolarCoord_01.jpg)

这个感觉就像拽着图片的两侧边向下绕了一个圈直到两边重合，将方形抻成一个圆形，再把重合处旋转到左侧。

当然实际制作中用的贴图肯定是可以无缝拼接的贴图，重合处就不会出现这样的硬边。

极坐标的优点就是模型只需要很低的面数就能够轻易制作出完美的圆形效果，同时极坐标的偏移轴是径向和角度这两个轴向，可以制作出从圆心处收缩或扩散以及旋转的效果，很适合制作魔法阵、Buff 等圆形动态特效。

缺点是图片中间偏下的部分像素会堆积，而中间偏上的部分像素会拉伸，可以配合渐入渐出效果进行优化。

下面说一下在 Unity 中实现极坐标效果的代码。

```c
// shader的路径和名称
Shader "Shader Forge/PolarCoord" {
    // 暴露属性到面板
    Properties {
        _MainTex("RGB: Color A: Alpha", 2d) = "gray"{}
        _Opacity("Opacity", range(0.0, 1.0)) = 1.0
    }
    SubShader {
        Tags {
            "Queue"="Transparent"                //调整渲染顺序
            "RenderType"="Transparent"           //改为对应的
            "ForceNoShadowCasting"="True"        //关闭阴影投射
            "IgnoreProjector"="True"             //不响应投射器
        }
        Pass {
            Name "FORWARD"
            Tags {
                "LightMode"="ForwardBase"
            }
            Blend One OneMinusSrcAlpha          //修改混合模式One One
            
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #pragma multi_compile_fwdbase_fullshadows
            #pragma target 3.0
            
            // 声明变量
            uniform sampler2D _MainTex; uniform float4 _MainTex_ST; //支持UV TillingOffset
            uniform half _Opacity;
            
            // 输入结构
            struct VertexInput {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
                float4 color : COLOR;
            };
            
            // 输出结构
            struct VertexOutput {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
                float4 color : COLOR;
            };
            
            // 顶点shader
            VertexOutput vert (VertexInput v) {
                VertexOutput o = (VertexOutput)0;
                o.pos = UnityObjectToClipPos( v.vertex );
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                o.color = v.color;
                return o;
            }
            
            // 像素shader
            float4 frag(VertexOutput i) : COLOR {
                i.uv = i.uv - float2(0.5, 0.5);  //坐标中心移到物体中心
                float theta = atan2(i.uv.y, i.uv.x); //获取夹角，值域为-Π到Π
                theta = theta / 3.1415926  * 0.5 + 0.5; //将夹角值域转为-1到1
                float r = length(i.uv) + frac(_Time.x*3); //获取半径，加上了时间偏移，得到向圆心收缩的动态效果
                i.uv = float2(theta, r);


                half4 var_MainTex = tex2D(_MainTex, i.uv);   //使用极坐标采样贴图
                half finalOpacity = var_MainTex.a * _Opacity;  
                return half4(var_MainTex.rgb * finalOpacity,finalOpacity);
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```

效果如下

![](https://www.yumefx.com/wp-content/uploads/2021/10/PolarCoord_03.gif)

如果做的复杂点，可以将径向速度、旋转速度等开放成属性，自由地控制动态效果。

本文同样参考自 [B 站庄懂的技术美术入门课](https://space.bilibili.com/6373917/video)，感谢。

永远努力在你的生活之上，  
保留一片天空。

——马塞尔 · 普鲁斯特