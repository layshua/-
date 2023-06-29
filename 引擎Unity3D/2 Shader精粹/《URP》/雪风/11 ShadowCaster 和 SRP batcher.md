

![[Pasted image 20230629155628.png]]

自己写阴影的投射 pass，这样就可以继续开启 SRP Batcher，并支持 alpha test 的透明阴影。

![[545500cae1b94172d5194bc310746549_MD5.webp]]

多光影和 alpha test

这次，我使用 shader_feature_local 去开启或者关闭 alphatest 的计算，不使用 shader_feature 是为了减少全局的关键词占用；同样，我也对是否计算额外灯光是否参与计算做了关键字，额外灯光的计算耗费性能，根据需求酌情开启关闭。

![[d916b2e4af0ce4130c639a9ced583505_MD5.webp]]

关键字定义

 第一个 pass，计算所有光照和阴影衰减。顶点着色器中和之前的篇幅内容一样，这里我额外增加判断是否开启了_MAIN_LIGHT_SHADOWS 关键字判断，如果没有定义，则不计算灯光空间的阴影坐标；然后把世界空间的视图方向，法线，都传递给片元。

![[8190b014464b5e603993b91a07956f27_MD5.webp]]

第一个 pass 的顶点着色器

   第一个 pass 的片元着色器，也是根据是否定义 _MAIN_LIGHT_SHADOWS 关键字去计算主光源的阴影衰减；然后使用半兰伯特布林风高光模型计算得到主光源的最终结果。

   而额外灯光的结算结果是根据是否定义了_ADD_LIGHT_ON 来决定是否计算，这个在材质面板来控制开启关闭；额外灯光的计算我只计算了半兰伯特部分，高光未参与（觉得比较费性能而且效果不明显。

  本人还根据是否定义了_CUT_ON，来开启或者关闭 alphatest。

![[8076245b0e4b0a03e590001456564793_MD5.webp]]

  
第一个 pass 的片元着色器  

最终把主光源和额外光影的计算结果加在一起就是我们想要的最终结果，这样第一个 pass 就完成。

第二个 pass 的目的是把当前灯光空间下的模型坐标写到 shadowmap 里，去投射到其他的模型上形成阴影，它的光照模式标签应该改成 "ShadowCaster"。

![[9f94c1ec7d668e1af603043072cc22d0_MD5.webp]]

第二个 pass 的标签

第二个 pass 的顶点着色器里，我们要得到一个特殊的裁剪空间的坐标，为了得到这坐标，我们需要模型的世界坐标，模型的世界法线，灯光方向，然后使用函数 float3 ApplyShadowBias(float3 positionWS, float3 normalWS, float3 lightDirection)，来得到特殊的世界坐标，然后转换到裁剪空间得到阴影投射专用的裁剪空间坐标。其中这个新函数的是定义在 Shadow.hlsl 下的，它的定义如下。

![[e072a8b725f10cfa286d1aebd7a17cb4_MD5.webp]]

ApplyShadowBias 函数定义

得到新的特殊的裁剪空间的坐标后，这里不急着输出给片元，它的 z 值还需进一步处理；根据是否进行了 Z 反向（比如 unity 编辑器下是 DX11，是有 Z 反向的），来取 z 值和 w 值 * 近裁剪面两者之间取最小值；若未 Z 反向则取最大值。这样得到的 Z 值在传递给 GPU 流水线下一个工位。

![[3d99ff4403e5b2cf21d92383d94d7652_MD5.webp]]

第二个 pass 的顶点着色器

   计算它的片元部分，其实这一步是没有做任何计算的，我们只根据是否定义了_CUT_ON 来是否进行 alphatest，采样一下贴图的 a 通道去测试，并不输出任何颜色（return 0）。

![[bf6431b799b654300bf6ba9ef00c6d41_MD5.webp]]

第二个 pass 的片元部分

   至此整体 shader 就完成了，他也兼容了 SRP Batcher，这是我们在官方明明有现成的 pass 可以使用时还偏偏自己写 ShadowCaster 的目的，但是他也有缺陷，在我们写的这个 shadowcaster 的顶点着色器里获取的光照方向是主光影的方向，也就是只支持主光源的阴影投射，若有大佬明白怎么把额外灯光的计算也加进去希望告知我，谢谢。

![[039639cd9cd7e05cc579f7fc16d5226b_MD5.webp]]

满足 SRP Batcher

最后老板来一串 82 年的 shader 源码：
```c
Shader "WX/URP/shadowCaster"

{

    Properties

    {

        _MainTex("MainTex",2D)="White"{}

        _BaseColor("BaseColor",Color)=(1,1,1,1)

        _Gloss("Gloss",Range(10,300))=50

        _SpecularColor("SpecularColor",Color)=(1,1,1,1)

        [KeywordEnum(ON,OFF)]_CUT("CUT",float)=1

        _Cutoff("cutoff",Range(0,1))=1

        [KeywordEnum(ON,OFF)]_ADD_LIGHT("AddLight",float)=1

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

        }

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"

        #pragma  shader_feature_local _CUT_ON 

        #pragma  shader_feature_local _ADD_LIGHT_ON 

        CBUFFER_START(UnityPerMaterial)

        float4 _MainTex_ST;

        half4 _BaseColor;

        float _Cutoff;

        float _Gloss;

        real4 _SpecularColor;

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

             #ifdef _MAIN_LIGHT_SHADOWS

             float4 shadowcoord:TEXCOORD1;

             #endif

             float3 WS_P:TEXCOORD2 ;

             float3 WS_N:TEXCOORD4 ; 

             float3 WS_V:TEXCOORD3 ;

         };

        ENDHLSL

        pass

        {

        Tags{

        "LightMode"="UniversalForward"

        "RenderType"="TransparentCutout"

        "Queue"="AlphaTest"

        }

        Cull off

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS

            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE

            #pragma multi_compile _ _ADDITIONAL_LIGHTS_VERTEX _ADDITIONAL_LIGHTS

            #pragma multi_compile _ _ADDITIONAL_LIGHT_SHADOWS

            #pragma multi_compile _ _SHADOWS_SOFT

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);

                o.WS_P=TransformObjectToWorld(i.positionOS.xyz);

                #ifdef _MAIN_LIGHT_SHADOWS

                o.shadowcoord=TransformWorldToShadowCoord(o.WS_P);

                #endif

                o.WS_V=normalize(_WorldSpaceCameraPos-o.WS_P.xyz);

                o.WS_N=normalize(TransformObjectToWorldNormal(i.normalOS.xyz));

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord)*_BaseColor;

                #ifdef _CUT_ON

                clip(tex.a-_Cutoff);

                #endif

                float3 NormalWS=i.WS_N;

                float3 PositionWS=i.WS_P;

                float3 viewDir=i.WS_V;

                //main light

                #ifdef _MAIN_LIGHT_SHADOWS

                Light mylight=GetMainLight(i.shadowcoord);

                #else

                Light mylight=GetMainLight();

                #endif

                half4 MainColor=(dot(normalize(mylight.direction.xyz),NormalWS)*0.5+0.5)*half4(mylight.color,1);

                MainColor+= pow(max(dot(normalize(viewDir +normalize(mylight.direction.xyz)),NormalWS),0),_Gloss);

                MainColor*=mylight.shadowAttenuation;

                //addlights

                half4 AddColor=half4(0,0,0,1);

                #ifdef _ADD_LIGHT_ON

                int addlightCount=GetAdditionalLightsCount();

                for(int t=0;t<addlightCount;t++)

                {

                Light addlight=GetAdditionalLight(t,PositionWS);

                // 额外灯光就只计算一下半兰伯特模型（高光没计算，性能考虑）

                AddColor+=(dot(normalize(addlight.direction),NormalWS)*0.5+0.5)*half4(addlight.color,1)*addlight.shadowAttenuation*addlight.distanceAttenuation;

                }

                #endif

                return tex*(MainColor+AddColor);

            }

            ENDHLSL

        }

      // UsePass "Universal Render Pipeline/Lit/ShadowCaster"

      pass

      {// 该 pass 只把主灯光空间的深度图写到了 shadowmap 里  addlight 灯光空间目前没有写进去 导致模型无法投射 addlight 的阴影 但是整 shader 可以接受 addlight 的阴影

      // 官方的

      Tags

      {

      "LightMode"="ShadowCaster"

      }

      HLSLPROGRAM

      #pragma vertex vertshadow

      #pragma fragment fragshadow

      v2f vertshadow(a2v i)

      {

      v2f o;

      o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);

      float3 WSpos=TransformObjectToWorld(i.positionOS.xyz);

      Light MainLight=GetMainLight();

      float3 WSnor=TransformObjectToWorldNormal(i.normalOS.xyz);

      o.positionCS=TransformWorldToHClip(ApplyShadowBias(WSpos,WSnor,MainLight.direction));

      #if UNITY_REVERSED_Z

      o.positionCS.z=min(o.positionCS.z,o.positionCS.w*UNITY_NEAR_CLIP_VALUE);

      #else

      o.positionCS.z=max(o.positionCS.z,o.positionCS.w*UNITY_NEAR_CLIP_VALUE);

      #endif

      return o;

         }

      half4 fragshadow(v2f i):SV_TARGET

      {

      #ifdef _CUT_ON 

      float alpha=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord).a;

      clip(alpha-_Cutoff);

      #endif

      return 0;

      }

      ENDHLSL

      }

    }

}
```

      在补充下 ShadowCaster 的 pass 里可以成功获取额外灯光的数据了。 

      翻了下官方的源码，找到在 ShadowCasterPass.hlsl 的，里面使用的光照是_LightDirection，这个是官方自己定义的特定的 float3，它可以获得主光源和额外光源的方向，替换掉 MainLight.direction 后，就可以正常获取 addlight 的灯光方向了，效果如下图所示（并且保持 SRP Batcher）。

![[a8cad8c92c63065f4d51e3c214dc8400_MD5.webp]]
```c
shader 源码如下

Shader "WX/URP/Caste Shadow"

{

    Properties

    {

        _MainTex("MainTex",2D)="white"{}

        _BaseColor("BaseColor",Color)=(1,1,1,1)

        [HDR]_SpecularColor("Specular",Color)=(1,1,1,1)

        _Gloss("Gloss",Range(10,300))=50 

        [KeywordEnum(OFF,ON)]_ADDLIGHT("Addlight",float)=1

        [KeywordEnum(OFF,ON)]_CUT("Cut",float)=1

        _Cutoff("Cutoff",Range(0,1))=0.5

    }

    SubShader

    {

        Tags{

        "RenderPipeline"="UniversalRenderPipeline"

        }

        HLSLINCLUDE

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Shadows.hlsl"

        #pragma  shader_feature_local _CUT_ON

        #pragma  shader_feature_local _ADDLIGHT_ON 

        #pragma multi_compile _ _MAIN_LIGHT_SHADOWS

        #pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE

        #pragma multi_compile _ _ADDITIONAL_LIGHT_SHADOWS

        #pragma multi_compile _ _SHADOWS_SOFT

        CBUFFER_START(UnityPerMaterial)

        float4 _MainTex_ST;

        half4 _BaseColor;

        real4 _SpecularColor;

        half _Gloss;

        float _Cutoff;

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

             float3 WS_N:NORMAL;

             float3 WS_P:TEXCOORD1;

             float3 WS_V:TEXCOORD2;  

             #ifdef _MAIN_LIGHT_SHADOWS

             float4 shadowcoord:TEXCOORD5;

             #endif

         };

        ENDHLSL

        pass

        {

        Tags{

         "LightMode"="UniversalForward"

        }

        cull off

            HLSLPROGRAM

            #pragma vertex VERT

            #pragma fragment FRAG

            v2f VERT(a2v i)

            {

                v2f o;

                o.positionCS=TransformObjectToHClip(i.positionOS.xyz);

                o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);

                o.WS_P=TransformObjectToWorld(i.positionOS.xyz);

                o.WS_N=normalize(TransformObjectToWorldNormal(i.normalOS.xyz));

                o.WS_V=normalize(_WorldSpaceCameraPos-o.WS_P);

                #ifdef _MAIN_LIGHT_SHADOWS

                o.shadowcoord=TransformWorldToShadowCoord(o.WS_P);

                #endif

                return o;

            }

            half4 FRAG(v2f i):SV_TARGET

            {

                half4 tex=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord)*_BaseColor;

                #ifdef _CUT_ON

                clip(tex.a-_Cutoff);

                #endif

                #ifdef _MAIN_LIGHT_SHADOWS

                Light mylight=GetMainLight(i.shadowcoord);

                #else

                Light mylight=GetMainLight();

                #endif

                real4 maincolor=(dot(i.WS_N,normalize(mylight.direction))*0.5+0.5)*real4(mylight.color,1)*mylight.shadowAttenuation;

                real4 addcolor=real4(0,0,0,1);

                #ifdef _ADDLIGHT_ON

                int addcount=GetAdditionalLightsCount();

                for(int t=0;t<addcount;t++)

                {

                Light addlight=GetAdditionalLight(t,i.WS_P);

                addcolor+=(dot(i.WS_N,normalize(addlight.direction))*0.5+0.5)*real4(addlight.color,1)*addlight.shadowAttenuation*addlight.distanceAttenuation;

                }

                #endif

                return tex*(maincolor+addcolor);

            }

            ENDHLSL

        }

        //UsePass "Universal Render Pipeline/Lit/ShadowCaster"

        pass

        {

        Tags

        {

        "LightMode"="ShadowCaster"

        }

        HLSLPROGRAM

        #pragma vertex vert

        #pragma fragment frag

        half3 _LightDirection;

        v2f vert(a2v i)

        {

        v2f o;

        float3 WS_P=TransformObjectToWorld(i.positionOS.xyz);

        float3 WS_N=normalize(TransformObjectToWorldNormal(i.normalOS.xyz));

        o.positionCS=TransformWorldToHClip(ApplyShadowBias(WS_P,WS_N,_LightDirection));

        o.texcoord=TRANSFORM_TEX(i.texcoord,_MainTex);

        #if UNITY_REVERSE_Z

        o.positionCS.z=min(o.positionCS.z,o.positionCS.w*UNITY_NEAR_CLIP_VALUE);

        #else

        o.positionCS.z=max(o.positionCS.z,o.positionCS.w*UNITY_NEAR_CLIP_VALUE);

        #endif

        return o;

        }

        real4 frag(v2f i):SV_TARGET

        {

        #ifdef _CUT_ON

        float alpha=SAMPLE_TEXTURE2D(_MainTex,sampler_MainTex,i.texcoord).a*_BaseColor.a;

        clip(alpha-_Cutoff);

        #endif

        return 0;

        }

        ENDHLSL

        }

    }

}```