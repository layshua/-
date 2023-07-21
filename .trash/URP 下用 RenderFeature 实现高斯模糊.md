
### **高斯模糊**

高斯模糊就不细说了，相关文章有很多。稍微讲一下原理：用高斯核对原图像进行卷积。由于高斯函数是能拆成 G(x)G(y) 形式的，所以核也能拆成一维横向和一维竖向的核，而且这两个核是完全一致的。因此横着的 Pass 来一次，竖着的 Pass 来一次，就完成高斯模糊了。

先写刚才遗留的 **Render** 函数。其实大部分和入门精要都差不多，稍微有点不同的就是用 **cmd** 来做 **Blit**、**GetTemporaryRT**。

```c
void Render(CommandBuffer cmd, ref RenderingData renderingData)
{
    var source = currentTarget;
    int destination0 = TempTargetId0;
    int destination1 = TempTargetId1;
​
    ref var cameraData = ref renderingData.cameraData;
​
    var data = renderingData.cameraData.cameraTargetDescriptor;
​
    var width = data.width/ parameter.downSample.value;
    var height = data.height / parameter.downSample.value;
​
    // 先存到临时的地方
    cmd.GetTemporaryRT(destination0, width, height, 0, FilterMode.Trilinear, RenderTextureFormat.ARGB32);
    cmd.Blit(source, destination0);
​
    for (int i = 0; i < parameter.iterations.value; ++i)
    {
        cmd.SetGlobalFloat("_BlurSize", 1.0f + i * parameter.blurSpread.value);
​
        // 第一轮
        cmd.GetTemporaryRT(destination1, width, height, 0, FilterMode.Bilinear, RenderTextureFormat.ARGB32);
        cmd.Blit(destination0, destination1, postMaterial, 0);
        cmd.ReleaseTemporaryRT(destination0);
​
        // 第二轮
        cmd.GetTemporaryRT(destination0, width, height, 0, FilterMode.Bilinear, RenderTextureFormat.Default);
        cmd.Blit(destination1, destination0, postMaterial, 1);
        cmd.ReleaseTemporaryRT(destination1);
    }
​
    cmd.Blit(destination0, source);
​
    cmd.ReleaseTemporaryRT(TempTargetId0);
}
```

然后是 Shader。这里有一点**非常非常重要！！！非常非常重要！！！非常非常重要！！！**

那就是 Properties 里一定要写**_MainTex**，不能只在 HLSL 里声明！

URP 不知道抽的什么风，不在 Properties 写**_MainTex** 不会直接黑，只会导致高斯模糊的结果出问题！比如增加 iterations 会让画面出现残影；downSample 调成 1，提高 iterations 后 RT 变黑。说实话要不是突然想起来要在 Properties 里写 **_MainTex**，我估计这辈子都搞不定 unity 的后处理了...

```c
Shader "URP/PostProcessing/GaussianBlur"
{
    Properties
    {
         _MainTex ("Texture", 2D) = "white" {}
    }
​
    SubShader
    {
        Tags
        {
           "RenderPipeline" = "UniversalRenderPipeline"
        }
        LOD 100
​
        HLSLINCLUDE
 #pragma prefer_hlslcc gles
 #pragma exclude_renderers d3d11_9x
 #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Color.hlsl"
 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
 #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
 #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/UnityInstancing.hlsl"
​
​
        struct Attributes
        {
            float4 positionOS : POSITION;
            float2 uv : TEXCOORD0;
        };
​
        struct Varyings
        {
            float2 uv[5] : TEXCOORD0;
            float4 positionCS : SV_POSITION;
        };
​
        float _BlurSize;
        sampler2D _MainTex;
        float4 _MainTex_TexelSize;
​
        Varyings VertBlurVertical(Attributes v)
        {
            Varyings o;
            o.positionCS = TransformObjectToHClip(v.positionOS.xyz);
            float2 uv = v.uv;
​
            o.uv[0] = uv;
            o.uv[1] = uv + float2(0.0, _MainTex_TexelSize.y * 1.0) * _BlurSize;
            o.uv[2] = uv - float2(0.0, _MainTex_TexelSize.y * 1.0) * _BlurSize;
            o.uv[3] = uv + float2(0.0, _MainTex_TexelSize.y * 2.0) * _BlurSize;
            o.uv[4] = uv - float2(0.0, _MainTex_TexelSize.y * 2.0) * _BlurSize;
​
            return o;
        }
​
        Varyings VertBlurHorizontal(Attributes v)
        {
            Varyings o = (Varyings)0;
            o.positionCS = TransformObjectToHClip(v.positionOS.xyz);
            float2 uv = v.uv;
​
            o.uv[0] = uv;
            o.uv[1] = uv + float2(_MainTex_TexelSize.x * 1.0, 0.0) * _BlurSize;
            o.uv[2] = uv - float2(_MainTex_TexelSize.x * 1.0, 0.0) * _BlurSize;
            o.uv[3] = uv + float2(_MainTex_TexelSize.x * 2.0, 0.0) * _BlurSize;
            o.uv[4] = uv - float2(_MainTex_TexelSize.x * 2.0, 0.0) * _BlurSize;
            return o;
        }
​
        float4 fragBlur(Varyings i) : SV_Target
        {
            float weight[3] = {0.4026, 0.2442, 0.0545};
​
            //中心像素值
            // float3 sum = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[0]).rgb * weight[0];
            float3 sum = tex2D(_MainTex, i.uv[0]).rgb * weight[0];
​
            for (int it = 1; it < 3; it++)
            {
                // sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[it * 2 - 1]).rgb * weight[it];
                // sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv[it * 2]).rgb * weight[it];
                sum += tex2D(_MainTex, i.uv[it * 2 - 1]).rgb * weight[it];
                sum += tex2D(_MainTex, i.uv[it * 2]).rgb * weight[it];
            }
​
            return float4(sum, 1.0);
        }
        ENDHLSL
​
​
        ZTest Always
        Cull Off
        ZWrite Off
​
        Pass
        {
            Name "GaussianPass0"
            HLSLPROGRAM            
 #pragma vertex VertBlurVertical
 #pragma fragment fragBlur
            ENDHLSL
        }
​
        Pass
        {
            Name "GaussianPass1"
            HLSLPROGRAM            
 #pragma vertex VertBlurHorizontal
 #pragma fragment fragBlur
            ENDHLSL
        }
    }
    Fallback Off
}
```