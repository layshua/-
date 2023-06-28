*    HLSL 和 CG 的大部分语法的内容是一致的，对于重复的内容，本人不在重述。
    
*    管线说明 tags 标签，URP 的 shader 都需标明使用的渲染管线的标签，即 "RenderPipeline"="UniversalRenderPipeline"。
    
*    CG 的引入变成了 HLSL 的引入，即 CGINCLUDE ENDCG 变成了 HLSLINCLUDE  ENDHLSL。CG 的编码，从 CGPROGRAM ENDCG 变成了 HLSL 的 HLSLPROGRAM ENDHLSL.
    
*    CBUFFER_START 和 CBUFFER_END, 对于变量是单个材质独有的时候建议放在这里面，以提高性能。CBUFFER(常量缓冲区) 的空间较小，不适合存放纹理贴图这种大量数据的数据类型，适合存放 float，half 之类的不占空间的数据，关于它的官方文档在下有详细说明。https://blogs.unity3d.com/2019/02/28/srp-batcher-speed-up-your-rendering/
    

    