      序列帧和广告牌技术是特效两兄弟，在 C# 里有内置函数让 z 轴始终朝向相机；有时候我们也直接使用粒子系统来达到广告牌的效果，但是借助外来工具处理并不是完美的，在我们这里在顶点着色器里进行处理达到最完美状态。

![[724d3639a1c8e047a6778d07942a4740_MD5.webp]]

P 粒子系统自带的广告牌

![[37f859c77aa2a89d9bb203a92ca1642a_MD5.webp]]

V 粒子自带的广告牌

      广告牌的实现，我们直接在上一篇的序列帧的 shader 上进行拓展修改。在 dcc 软件做一个片，使它的 z 为片的法线方向，导出给 unity 使用。

    顶点着色器里是我们的重点，先构建一个新的 Z 轴朝向相机的坐标系，这时我们需要在模型空间下计算新的坐标系的 3 个坐标基（三个新的坐标基在图中分别为 ox‘   oy’   oz‘表示）由于三个坐标基两两垂直，故只需要计算 2 个叉乘即可得到第三个坐标基，先计算新坐标系的 Z 轴。新坐标系的 Z 轴就是把世界空间的相机到（0，0，0）的朝向转换到模型空间中，代码里令它为 newZ，即图中的 oz‘。

![[95d09df90c37c2b35c67530e8b2bb091_MD5.webp]]

新坐标系的 Z 轴

然后在通过 shader_feature_local _Z_STAGE_LOCAL_Z 去控制是否在锁定 Z 轴和释放 Z 轴之间切换，这样就得到 2 种状态之前的切换: 1.Free 状态，即面片始终朝向摄像机；2.LOCK 状态，面片的 Z 轴始终锁定在水平面内，该状态下就和相机投影在水平面的分量作为新的 Z 轴方向，它可以通过令 Free 状态下的 Z 轴的 y 分量为 0 得到。

![[65ef51afe1f8a26942fac63529ebaa34_MD5.webp]]

锁定状态和自由状态

![[4d247c127289b316b95acb41444a6a3e_MD5.webp]]

Lock 状态下令 Z 为 0  通过 shader 变体去控制

计算到新的 Z 轴后，随后对新的 X 轴，即 ox’进行计算。

![[34139e6dbcb0a95664a3869c39e90f09_MD5.webp]]

三个轴的计算

    观察新的 X 轴，我们发现新 X 轴是始终处于原模型空间的 ZOX 平面内的，该平面内的任意向量都是始终和老的 y 轴垂直，也就是 oy 和 ox’ 一定垂直；而我们所知 ox‘和 oz’垂直，那么通过 oz‘和 oy 的向量外积，在右手法则下，可以得到 ox’，而 oy=float3（0，1，0），则 ox‘的表达式可以写成 cross(float3(0,1,0),z’)。

![[094338afeefea43211523cb0741fc58c_MD5.webp]]

      貌似我们得到了正确的新的 X 轴朝向了，但是思考另外一种情况，新的 X 轴是 float3(0,1,0) 和 oz’进行叉乘得到，若 oz‘和 float3(0,1,0) 重合了（也就是我们的相机朝向完全垂直了），得到的新的 X 轴朝向为 XOZ 平面内的任意向量（XOZ 平面内的任意向量都满足与这 2 向量垂直），这不是我们想要的，所以我们需要加一个逻辑判断，在 oz’的 y 分量大于 0.99 时，我们就认为相机的朝向（oz’）和 oy 方向重合了，计算方式得换令一种计算方式。

![[1152e3db7bfaa5aaa9cee64c7a299406_MD5.webp]]

      当 oz‘的 y 分量大于 0.99 后，我们就粗略认为 oz’完全和 oy 重合了，这时候在用 oz‘和 oy 的叉乘去计算 ox’就不合理了，我们需要额外找到一个与 ox‘垂直的向量才行；这时候 oz 向量就有作用了，仔细观察这种极端条件下，ox’的特性，它一定和 oz 垂直，oz 我们所知为 float3（0，0，1），故在该条件下，计算 ox‘的方式是 oz’和 oz 的向量外积，再次使用右手法则，则可以得到 ox‘的表达式 cross(newZ,float3(0,0,1))。在 shader 里，我们通过 条件？A:B 来计算我们想要的 ox’，计算后在正交规范化一下。

![[4200c3497f7cb3a275bb7780371cd3fe_MD5.webp]]

计算 ox‘ 即 newX 轴

         得到 ox’，oz‘后，它两叉乘，就得到最后一个轴 oy’，在正交规范化一下即可。

![[d53a7d7f5238e7c947a19c208fed8269_MD5.webp]]

计算最后一个新的 OY 轴

    然后构建一个变换矩阵 matrix，本来三个轴应该以列向量的形式构建该矩阵，但是在 unity 中矩阵的填充方式是按行填充，也就是 float3x3 Matrix={newX,newY,newZ} 得到的矩阵还需要转置一下，这里我们未进行转置，而是在下一个计算新的顶点坐标时，使用 mul 函数把矩阵放在了右边，mul 函数就会自动帮我们进行转置并左乘，这是一个 trick 说明一下。

![[e725b06ca98980c90e0c4ecd83ac4260_MD5.webp]]

矩阵右乘

    然后把新的顶点坐标用 mvp 转一下，这样顶点着色器就完成了，而片元着色器直接使用上一篇的序列帧的算法，至此整个 shader 就完成了。

    广告牌的难点在于新的 3 个轴是如何计算出来的，尤其是前面 2 个轴的计算方式，然后 mul 函数也注意一下坑，左乘和右乘的区别，别掉进去即可。下面附一下 shader 源码

Shader "WX/URP / 序列帧"

{

    Properties

    {

        _MainTex("MainTex",2D)="white"{}

        _BaseColor("BaseColor",Color)=(1,1,1,1)

        _Sheet("Sheet",Vector)=(1,1,1,1)

        _FrameRate("FrameRate",float)=25

        [KeywordEnum(LOCK_Z,FREE_Z)]_Z_STAGE("Z_Stage",float)=1// 定义一个是否锁定 Z 轴

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

        "Queue"="Transparent"

        "RenderType"="Transparent"

        }

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        CBUFFER_START(UnityPerMaterial)

        float4 _MainTex_ST;

        half4 _BaseColor;

        half4 _Sheet;

        float _FrameRate;

        CBUFFER_END

        TEXTURE2D(_MainTex);

        SAMPLER(sampler_MainTex);

         struct a2v

         {

             float4 positionOS:POSITION;

             float4 normalOS:NORMAL;

             float2 texcoord:TEXCOORD;

         };

         struct v2f

         {

             float4 positionCS:SV_POSITION;

             float2 texcoord:TEXCOORD;

         };

        ENDHLSL

        pass

        {

        Tags{

         "LightMode"="UniversalForward"

        }

        ZWrite off

        Blend SrcAlpha OneMinusSrcAlpha  

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            #pragma  shader_feature_local _Z_STAGE_LOCK_Z 

            v2f VERT(a2v i)

            {

                v2f o;

                // 先构建一个新的 Z 轴朝向相机的坐标系，这时我们需要在模型空间下计算新的坐标系的 3 个坐标基

                // 由于三个坐标基两两垂直，故只需要计算 2 个即可叉乘得到第三个坐标基

                // 先计算新坐标系的 Z 轴

                float3 newZ=TransformWorldToObject(_WorldSpaceCameraPos);// 获得模型空间的相机坐标作为新坐标的 z 轴

                // 判断是否开启了锁定 Z 轴 

                #ifdef _Z_STAGE_LOCK_Z

                newZ.y=0;

                #endif

                newZ=normalize(newZ);

                // 根据 Z 的位置去判断 x 的方向

                float3 newX= abs(newZ.y)<0.99?cross(float3(0,1,0),newZ):cross(newZ,float3(0,0,1));

                newX=normalize(newX);

                float3 newY=cross(newZ,newX);

                newY=normalize(newY);

                float3x3 Matrix={newX,newY,newZ};// 这里应该取矩阵的逆 但是 hlsl 没有取逆矩阵的函数 

                float3 newpos=mul(i.positionOS.xyz,Matrix);// 故在 mul 函数里进行右乘 等同于左乘矩阵的逆（正交阵的转置等于逆）

                o.positionCS=TransformObjectToHClip(newpos);

                o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

            float2 uv;// 小方块的 uv

            uv.x=i.texcoord.x/_Sheet.x+frac(floor(_Time.y*_FrameRate)/_Sheet.x);

            uv.y=i.texcoord.y/_Sheet.y+1-frac(floor(_Time.y*_FrameRate/_Sheet.x)/_Sheet.y);

                return SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,uv);

            }

            ENDHLSL

        }

    }

}

![[2a72d2ad7ea63bbdb8d770e55516f399_MD5.webp]]