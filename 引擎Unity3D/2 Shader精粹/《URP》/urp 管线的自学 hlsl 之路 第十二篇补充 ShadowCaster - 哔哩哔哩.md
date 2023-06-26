      在补充下 ShadowCaster 的 pass 里可以成功获取额外灯光的数据了。 

      翻了下官方的源码，找到在 ShadowCasterPass.hlsl 的，里面使用的光照是_LightDirection，这个是官方自己定义的特定的 float3，它可以获得主光源和额外光源的方向，替换掉 MainLight.direction 后，就可以正常获取 addlight 的灯光方向了，效果如下图所示（并且保持 SRP Batcher）。

![[a8cad8c92c63065f4d51e3c214dc8400_MD5.webp]]

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

}