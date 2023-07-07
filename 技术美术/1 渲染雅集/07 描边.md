## Unity 2022 URP 自定义后处理之基于颜色使用 Sobel 算子进行描边

本文基于冯乐乐《shader 入门精要》并基于 Unity 2022.3.0 lts，URP14 改写

URP 14 带来了一个新 Feature，FullScreenPassRendererFeature，可以很方便的写各种自定义的后处理特效，本文使用 Sobel 算子进行边缘检测并描边。

Sobel 算子的思想是在某个像素的临近 9 宫格内采样，计算 x 和 y 方向的梯度，如果梯度越大，表示这个地方颜色突变越强，越接近于一个边缘点。

在 Renderer 中添加 FullScreenPassRendererFeature，新建一个材质，命名为 SobelOutline，拖入 Pass Material 即可

![](https://pic2.zhimg.com/v2-8bdf3034f0f046238159d81ab2a11075_r.jpg)

新建一个 Shader，命名为 SobelOutline，拖入 Material 中，Shader 写法如下：

```
Shader "Outline"
​
{
    Properties
    {
        _EdgeColor("EdgeColor",Color)=(0,0,0,1)
    }
    SubShader
    {
        Tags
        {
​
            "RenderType" = "Transparent" "RenderPipeline" = "UniversalPipeline"
        }
        Cull Off
        Blend Off
        ZTest Off
        ZWrite Off
        Pass
        {
            Name "Outline"
            HLSLPROGRAM
 #pragma vertex Vert
 #pragma fragment Frag
            //这两个头文件包括了大多数需要用到的变量
 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
 #include "Packages/com.unity.render-pipelines.universal/Shaders/PostProcessing/Common.hlsl"
            //这个需要自己声明，xy表示纹素的长宽，zw表示整个BlitTexture的长宽，BlitTexture就是当前摄像机的颜色缓冲区
            float4 _BlitTexture_TexelSize;
​
            float2 uvs[9];
            //常规CBUFFER，上面自定义的属性要写在这里
            CBUFFER_START(UnityPerMaterial)
            half4 _EdgeColor;
            CBUFFER_END
​
            half Sobel()
            {
                const half Gx[9] = {-1, -2, -1, 0, 0, 0, 1, 2, 1};
                const half Gy[9] = {-1, 0, 1, -2, 0, 2, -1, 0, 1};
                half texColor;
                half edgeX = 0, edgeY = 0;
                for (int it = 0; it < 9; it++)
                {
                    //RGB转亮度
                    texColor = Luminance(SAMPLE_TEXTURE2D_X(_BlitTexture, sampler_LinearClamp, uvs[it]));
                    //计算亮度在XY方向的导数，如果导数越大，越接近一个边缘点
                    edgeX += texColor * Gx[it];
                    edgeY += texColor * Gy[it];
                }
                //edge越小，越可能是个边缘点
                half edge = 1 - abs(edgeX) - abs(edgeY);
                return edge;
            }
​
            half4 Frag(Varyings i) : SV_TARGET
            {
                half2 uv = i.texcoord;
                uvs[0] = uv + _BlitTexture_TexelSize.xy * half2(-1, -1);
                uvs[1] = uv + _BlitTexture_TexelSize.xy * half2(0, -1);
                uvs[2] = uv + _BlitTexture_TexelSize.xy * half2(1, -1);
                uvs[3] = uv + _BlitTexture_TexelSize.xy * half2(-1, 0);
                uvs[4] = uv + _BlitTexture_TexelSize.xy * half2(0, 0);
                uvs[5] = uv + _BlitTexture_TexelSize.xy * half2(1, 0);
                uvs[6] = uv + _BlitTexture_TexelSize.xy * half2(-1, 1);
                uvs[7] = uv + _BlitTexture_TexelSize.xy * half2(0, 1);
                uvs[8] = uv + _BlitTexture_TexelSize.xy * half2(1, -1);
                half edge = Sobel();
                //根据edge的大小，在边缘颜色和原本颜色之间插值，edge为0时，完全是边缘，edge为1时，完全是原始颜色
                half4 withEdgeColor = lerp(_EdgeColor,SAMPLE_TEXTURE2D_X(_BlitTexture, sampler_LinearClamp, uv), edge);
                return withEdgeColor;
            }
            ENDHLSL
        }
    }
    //后处理不需要Fallback，不满足的时候不显示即可
    Fallback off
}
```

效果：可见这种方法下如果两个物体颜色接近，就检测不出来了。

![](https://pic2.zhimg.com/v2-91813942b14226f72936b98783a03749_r.jpg)