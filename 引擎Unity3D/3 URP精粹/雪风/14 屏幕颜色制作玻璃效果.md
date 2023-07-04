      这两天研究了屏幕图像相关的内容，有一些心得记录下来。在 build in 管线里，我们通过 grab pass/RT 获取屏幕图像去制作玻璃效果，在 URP 管线下，我们则可以使用 urp 提供的 copy color 去得到一张同等分辨率屏幕的图像，它的名字叫_CameraColorTexture，它是在所有的 opaque 模型和 skybox 渲染完成后抓取的一张图像。

![[dc281627e7b24df78c3de918cf40c41b_MD5.webp]]

copy color

    要获取这张图像，得在 URP 的配置文件里开启 opaque texture 才可以获取到它。

![[6fcbf7809888823c81cc5f89c7cbeeca_MD5.webp]]

管线配置文件

        然后在 shader 里，利用 SAMPLER() 函数采样这张贴图。得到了这张贴图后，下一步就是获取屏幕的 uv，本人总结了三种方式获取屏幕 uv。

        1. 使用 computegrabscreenpos（）函数去获取，这是《入门精要》的操作，这里不多赘述。

        2. 自己手动在顶点着色器里，从 HClip 空间计算到屏幕空间，经我多次研究，总结了顶点着色器的 HClip 的数据特点，并通过它计算得到屏幕 uv 的。

![[1962aa908eeff5cf4eb0603069cd3fc8_MD5.webp]]

顶点着色器里的 HClip 的顶点特点

      在编辑器 DX11 下，近裁剪面的坐标最大值在右下角（near，near），最小值在左上角（-near，-near），对应的深度 Z 为 near；远裁剪面的坐标最大值在右下角（far，far），最小值在左上角（-far，-far），对应的深度 Z 为 0。很明显，这个已经和我们在《入门精要》的坐标不一样了：1. 因为入门精要的内容是 openGL 下的，Y 轴已经反向了；2.Z 轴为了平衡近裁剪面和远裁剪面的精度也做了 Z 反向（利用了浮点数靠近 0 处的精度很高来达到让远裁剪面的精度提高；利用了反比例函数曲线的特性让靠近近裁剪面的精度也提高）。3.Z 轴的比例也进行了缩放，整个视锥体的长度应该是（0，far-near），这里被缩放成了（0，near)。

        这里我们只关注屏幕 uv 不关注深度 z，屏幕 uv 它应该就是（xy/w*0.5+0.5），但是我们不希望在顶点着色器里进行透视除法，所以结果应该是 w*（xy/w*0.5+0.5）=xy*0.5+0.5*w，随后在片元着色器里进行透视除法（其实是可以在顶点着色器里进行透视除法，因为视锥体的非线性只影响了 Z，我们这里不会用到 Z(至少暂时如此），但是这种骚操作可能导致一些后续的问题（万一后面要用 z，搞混淆了不好），还是推荐不在顶点着色器里进行透除。

![[239cf4c88da957c95f6162ab3cc12a11_MD5.webp]]

在顶点着色器里获取未经透除的屏幕 uv

     在片元着色器里，我们对拿到的屏幕 uv 进行透除，考虑到 openGL 和 DX11 的 Y 轴反向问题，还得判断一下当前的平台是否对 y 进行 oneminus，最终就能得到正常的屏幕 uv。

![[545dada3dec0d56708f4b0649e0a1973_MD5.webp]]

片元着色器里得到正常的屏幕 uv

    3. 如果你被第二种获取屏幕 uv 搞晕了，不用担心，还有第三种超级简单的方法，直接在片元着色器里获取 HClip 的值，通过它得到 uv，经我测试，HClip.xy 就是屏幕空间的物理像素大小，利用它在除以_ScreenParams.xy，就是我们的屏幕 UV 了，惊不惊喜，意不意外。

![[c938359a193d00d14590937bf11536e5_MD5.webp]]

只需一部，获取屏幕 uv

      前面大费精力得到了屏幕图像和屏幕 uv 了，是时候展示真正的技术了。通过第五篇专栏的法线贴图的计算方式，我们得到了世界空间的法线，计算过程中也得到了切线空间的法线。这 2 个法线去分别偏移屏幕 uv 有什么区别呢？本人通过 shader_feather 去做了 2 个 shader 变体来观察 2 种不同的偏移方式带来的不同的效果。

![[e93afaaae26c1ebf8538b3258080cc1d_MD5.webp]]

shader 变体切换不同的效果

![[e9ac434d37ae8dd96785da44a9a7269c_MD5.webp]]

如果是世界空间切线，则构建 TBN 矩阵转换；否则直接拿去计算

        计算完成后，我们把颜色输出：如果使用的是世界空间法线，则扭曲的效果除了扭曲之外，还会朝着一个方向进行偏移；而切线空间下只会扭曲，并不会偏移。如果提高强度，世界空间法线下的效果会偏移到不知道什么地方的图像，而切线空间下还是只会加剧扭曲程度，原来的颜色还是停留在原地。

![[af4481f6f6caea13e070a351c9b4121e_MD5.webp]]

强度设为 1000，世界坐标下，已经把图像偏移到不知是什么区域的图像了

![[3a55c837afaf9378f71389d8bdd352c2_MD5.webp]]

强度为 1000 下，切线空间里，虽然扭曲剧烈，但是图像还是原来区域的图像

        两者的差距会如此巨大，原因是世界空间的法线它是取决于整个世界坐标系；而切线空间下的法线是取决与模型本身，与世界无关。如果我们旋转模型，观察某一个面的法线朝向，世界空间法线会随着旋转而变化；而该面的切线空间的法线始终不会改变。正是因为参考系的不同，才导致了 2 种截然不同的效果，感兴趣的读者可以自己把 2 种坐标系下的法线的 x 分量，y 分量分别返回到片元着色器输出，旋转模型看看效果。

      说了这么多，该附上最后的代码了，如果代码复制过去报错，那是空格复制的错误，把每行代码里最前面的空格替换成 ide 里的空格即可。

Shader "WX/URP/grass"

{

    Properties

    {

        _NormalTex("Normal",2D)="bump"{}

        _NormalScale("NormalScale",Range(0,1))=1

        _BaseColor("BaseColor",Color)=(1,1,1,1)

        _Amount("amount",float)=100

        [KeywordEnum(WS_N,TS_N)]_NORMAL_STAGE("NormalStage",float)=1

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

        "RenderType"="Transparent"

        "Queue"="Transparent"

        }

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/lighting.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float4 _NormalTex_ST;

        half4 _BaseColor;

        float _NormalScale;

        float _Amount;

        CBUFFER_END

        float4 _CameraColorTexture_TexelSize;// 该向量是非本 shader 独有，不能放在常量缓冲区

        TEXTURE2D(_NormalTex);

        SAMPLER(sampler_NormalTex);

        SAMPLER(_CameraColorTexture);

         struct a2v

         {

             float4 positionOS:POSITION;

             float4 normalOS:NORMAL;

             float2 texcoord:TEXCOORD;

             float4 tangentOS:TANGENT;

         };

         struct v2f

         {

             float4 positionCS:SV_POSITION;

             float2 texcoord:TEXCOORD;

             float4 normalWS:NORMAL;

             float4 tangentWS:TANGENT;

             float4 BtangentWS:TEXCOORD1;  

         };

        ENDHLSL

        pass

        {

        Tags{

        "LightMode"="UniversalForward"

        }

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            #pragma  shader_feature_local _NORMAL_STAGE_WS_N 

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=TRANSFORM_TEX(i.texcoord,_NormalTex);

                o.normalWS.xyz=normalize(TransformObjectToWorldNormal(i.normalOS.xyz));

                o.tangentWS.xyz=normalize(TransformObjectToWorldDir(i.tangentOS.xyz));

                o.BtangentWS.xyz=cross(o.normalWS.xyz,o.tangentWS.xyz)*i.tangentOS.w*unity_WorldTransformParams.w;

                float3 positionWS=TransformObjectToWorld(i.positionOS.xyz);

                o.tangentWS.w=positionWS.x;

                o.BtangentWS.w=positionWS.y;

                o.normalWS.w=positionWS.z;

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 normalTex=SAMPLE_TEXTURE2D(_NormalTex,sampler_NormalTex,i.texcoord)*_BaseColor;// 获取法线贴图

                float3 normalTS=UnpackNormalScale(normalTex,_NormalScale);// 得到我们想要对比的切线空间法线

                float2 SS_texcoord=i.positionCS.xy/_ScreenParams.xy;// 获取屏幕 UV

                #ifdef _NORMAL_STAGE_WS_N// 计算偏移的 2 张方式

                float3x3 matrix_T2W={i.tangentWS.xyz,i.BtangentWS.xyz,i.normalWS.xyz};// 构建 tbn 矩阵

                float3 WSnor=mul(normalTS,matrix_T2W);// 得到我们想要的世界空间法线

                //return real4(WSnor,1);// 测试世界空间法线

                float2 SS_bias=WSnor.xy*_Amount*_CameraColorTexture_TexelSize;// 如果取的世界空间的法线则执行它计算偏移，但是世界空间的法线由世界空间确定，会随着模型的旋转而变化；

                #else

                float2 SS_bias=normalTS.xy*_Amount*_CameraColorTexture_TexelSize;// 如果取的是切线空间的法线则执行它计算偏移，但是切线空间的法线不随着模型的旋转而变换；

                #endif

                float4 glassColor=tex2D(_CameraColorTexture,SS_texcoord+SS_bias);// 把最终的颜色输出到屏幕即可

                return real4(glassColor.xyz,1);

            }

            ENDHLSL

        }

    }

}