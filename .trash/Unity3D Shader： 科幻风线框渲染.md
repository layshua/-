之前看了一本小说叫《副本》，里面有个描写就是拷问犯人的时候进入虚拟空间，进入的时候显示基础的线条，然后在变得越来越精细，最后女主角为男主角递上一根烟，正好全部加载完成，之后一直在脑中想象了这个效果，当然最终的实现还是和想象的有点差距，下面我们先来看看最终的效果：

![[a87c99ff41f9cd0d77c5f8e06e1aad09_MD5.gif]]

第一步首先先根据模型创建线框，对于这一步我一开始联想到了 Unity 的 warframe，但是却不知道如何绘制两点之间的直线，

![[ff2428b09d1ecfbd884d7e10d187ee2d_MD5.jpg]]

然后去找它的类似效果的实现方式，却意外的发现了 Geometry Shader, 它介于顶点和片元之间，可以在这一阶段通过图元来修改顶点，下面的实现参考：

[Unity GeometryShader(从一个线框渲染的例子开始)](https://www.bbsmax.com/A/1O5EO1paz7/)

下面来开始编写我们的 shader：

```
#pragma vertex vert
#pragma geometry geom
#pragma fragment frag
```

现在我们从原先的顶点 -》像素 变成 顶点 --》几何 --》像素的方式 同时顶点阶段也要变为输出数据给几何阶段

```
struct appdata
            {
                float4 vertex: POSITION;
                float2 uv: TEXCOORD0;
            };

            struct v2g
            {
                float2 uv: TEXCOORD0;
                float4 vertex: SV_POSITION;
            };

            struct g2f
            {
                float2 uv: TEXCOORD0;
                float4 vertex: SV_POSITION;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            v2g vert(appdata v)
            {
                v2g o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            [maxvertexcount(3)]
            void geom(triangle v2g IN[3], inout TriangleStream < g2f > triStream)
            {
                g2f OUT;
                
                OUT.vertex = IN[0].vertex;
                OUT.uv = IN[0].uv;
                triStream.Append(OUT);

                OUT.vertex = IN[1].vertex;
                OUT.uv = IN[1].uv;
                triStream.Append(OUT);

                OUT.vertex = IN[2].vertex;
                OUT.uv = IN[2].uv;
                triStream.Append(OUT);
            }

            fixed4 frag(g2f i): SV_Target
            {
                // sample the texture
                fixed4 col = tex2D(_MainTex, i.uv);
                return col;
            }
```

下面绘制线框，绘制线框的原理是这样的，在几何阶段，我们得到一个三角形，那么对于这个三角形内部的点来说，该点距离其他三条边的距离是 0 的话，说明这个点在这个三角形的边上，我们可以使用一个变量来控制最小的距离来实现控制线框的宽度。

```
[maxvertexcount(3)]
            void geom(triangle v2g IN[3], inout TriangleStream < g2f > triStream)
            {
                float2 p0 = IN[0].vertex.xy / IN[0].vertex.w;
                float2 p1 = IN[1].vertex.xy / IN[1].vertex.w;
                float2 p2 = IN[2].vertex.xy / IN[2].vertex.w;

                float2 v0 = p2 - p1;
                float2 v1 = p2 - p0;
                float2 v2 = p1 - p0;
                //triangles area
                float area = abs(v1.x * v2.y - v1.y * v2.x);

                // //到三条边的最短距离
                g2f OUT;
                OUT.vertex = IN[0].vertex;
                OUT.uv = IN[0].uv;
                OUT.dist = float3(area / length(v0), 0, 0);
                triStream.Append(OUT);

                OUT.vertex = IN[1].vertex;
                OUT.uv = IN[1].uv;
                OUT.dist = float3(0, area / length(v1), 0);
                triStream.Append(OUT);

                OUT.vertex = IN[2].vertex;
                OUT.uv = IN[2].uv;
                OUT.dist = float3(0, 0, area / length(v2));
                triStream.Append(OUT);
            }
```

首先 P0,P1,P2 三个点根据世界坐标除以 w 得到视口坐标, v0,v1,v2 则是对应的三个边的方向，area 则利用叉积的几何意义得到三角形的面积（应该是平行四边形），那么对于每个点而言，这个点到对着边的距离根据面积公式反推可以得到 平行四边形的面积除以对边的长度，我们将距离记录下来。

```
fixed4 frag(g2f i): SV_Target
            {

                fixed4 col_Wire;
                float d = min(i.dist.x, min(i.dist.y, i.dist.z));
                col_Wire.rgb = d < _WireWidth?_WireColor: _FillColor;
                col_Wire.a = 1;
                return col_Wire;
}
```

到了像素阶段，我们就可以比较这个点到三条边的最短距离，得到一个最短距离，然后就可以通过这个最短距离来决定如何渲染，这里我们添加了一个变量_WireWidth 来判断距离，另外添加了两种颜色，分别取渲染线和其他区域

![[d67e4c9d54d127b7a4096c9869d80fc5_MD5.jpg]]

到这里线框处理好了，下面我们再来实现断层的效果

```
struct v2g
            {
                float3 uv: TEXCOORD0;
                float4 vertex: SV_POSITION;
            };

            struct g2f
            {
                float3 uv: TEXCOORD0;
                float4 vertex: SV_POSITION;
                float3 dist: TEXCOORD1;
            };

            .......

            v2g vert(appdata v)
            {
                v2g o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv.xy = TRANSFORM_TEX(v.uv, _MainTex);
                o.uv.z = _Clip + v.vertex.x;
                return o;
            }
....
 fixed4 frag(g2f i): SV_Target
            {
                clip(i.uv.z);
....
```

我们新加入了一个变量_Clip 来对模型进行 clip，将 z 的高度信息暂时保存在 uv.z 中，然后进行 clip

![[7c187a7cb51e6d853e3c773c29f126d7_MD5.jpg]]

模型已经被裁剪掉了，下面再来实现对原有材质的渐变

```
half blendValue = smoothstep(_Lerp - _WireLerpWidth, _Lerp + _WireLerpWidth, i.uv.z);
fixed4 col_Tex = tex2D(_MainTex, i.uv);
                //return blendValue * col_Tex;
                return lerp(col_Wire, col_Tex, blendValue);
```

_Lerp 用于控制离线框的远近，_WireLerpWidth 用于控制边缘宽度

![[5dc2507edf116aa5e7e22f31f16d4f67_MD5.jpg]]

![[f57dba1c99c0cf1c04d4db0ce922dc06_MD5.jpg]]

完成！

完整代码如下

```
Shader "Unlit/Wireframe2"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" { }
        _WireColor ("WireColor", Color) = (1, 0, 0, 1)
        _FillColor ("FillColor", Color) = (1, 1, 1, 1)
        _WireWidth ("WireWidth", Range(0, 0.005)) = 1

        _Clip ("Clip", Range(-2, 2.5)) = 1
        _Lerp ("Lerp", Range(0, 1)) = 0.5
        _WireLerpWidth ("WireLerpWidth", Range(0, 1)) = 0.1
    }
    SubShader
    {
        Tags { "RenderType" = "Transparent" "Queue" = "Transparent" }
        LOD 100

        Pass
        {
            Blend SrcAlpha OneMinusSrcAlpha
            Cull Off
            CGPROGRAM
 #pragma vertex vert
 #pragma geometry geom
 #pragma fragment frag
 #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex: POSITION;
                float2 uv: TEXCOORD0;
            };

            struct v2g
            {
                float3 uv: TEXCOORD0;
                float4 vertex: SV_POSITION;
            };

            struct g2f
            {
                float3 uv: TEXCOORD0;
                float4 vertex: SV_POSITION;
                float3 dist: TEXCOORD1;
            };

            sampler2D _MainTex;
            float4 _MainTex_ST;

            float4 _FillColor, _WireColor;
            float _WireWidth, _Clip, _Lerp, _WireLerpWidth;

            v2g vert(appdata v)
            {
                v2g o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv.xy = TRANSFORM_TEX(v.uv, _MainTex);
                o.uv.z = _Clip + v.vertex.x;
                return o;
            }

            [maxvertexcount(3)]
            void geom(triangle v2g IN[3], inout TriangleStream < g2f > triStream)
            {
                float2 p0 = IN[0].vertex.xy / IN[0].vertex.w;
                float2 p1 = IN[1].vertex.xy / IN[1].vertex.w;
                float2 p2 = IN[2].vertex.xy / IN[2].vertex.w;

                float2 v0 = p2 - p1;
                float2 v1 = p2 - p0;
                float2 v2 = p1 - p0;
                //triangles area
                float area = abs(v1.x * v2.y - v1.y * v2.x);

                // //到三条边的最短距离
                g2f OUT;
                OUT.vertex = IN[0].vertex;
                OUT.uv = IN[0].uv;
                OUT.dist = float3(area / length(v0), 0, 0);
                triStream.Append(OUT);

                OUT.vertex = IN[1].vertex;
                OUT.uv = IN[1].uv;
                OUT.dist = float3(0, area / length(v1), 0);
                triStream.Append(OUT);

                OUT.vertex = IN[2].vertex;
                OUT.uv = IN[2].uv;
                OUT.dist = float3(0, 0, area / length(v2));
                triStream.Append(OUT);
            }

            fixed4 frag(g2f i): SV_Target
            {
                clip(i.uv.z);
                fixed4 col_Wire;
                float d = min(i.dist.x, min(i.dist.y, i.dist.z));
                col_Wire = d < _WireWidth?_WireColor: _FillColor;
                //return col_Wire;
                //颜色差值
                half blendValue = smoothstep(_Lerp - _WireLerpWidth, _Lerp + _WireLerpWidth, i.uv.z);
                
                
                fixed4 col_Tex = tex2D(_MainTex, i.uv);
                //return blendValue * col_Tex;
                return lerp(col_Wire, col_Tex, blendValue);
            }
            ENDCG
            
        }
    }
}
```