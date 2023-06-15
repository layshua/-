LOD：Level of Detail  
作用：unity 引擎会根据不同的 LOD 值在使用不同的 SubShader  
Unity 选择对应的 Subshader 会从上往下寻找第一个小于等于 LOD 值的子[着色器](https://so.csdn.net/so/search?q=%E7%9D%80%E8%89%B2%E5%99%A8&spm=1001.2101.3001.7020)。一个着色器中会有一到多个 SubShader，但是系统每次只会执行一个子着色器，选择子着色器的标准就是根据子着色器所设置的 LOD 的值来进行选择。

如何设置 [Shader](https://so.csdn.net/so/search?q=Shader&spm=1001.2101.3001.7020) 的 LOD 的值：通过 Shader 的下面属性进行设置：shader.maximumLOD=500；

（1）新建 Shader 脚本如下：

```
Shader "Custom/LODShader" {
	Properties{
		_Color("Color", Color) = (1,1,1,1)
		_MainTex("Albedo (RGB)", 2D) = "white" {}
		_Glossiness("Smoothness", Range(0,1)) = 0.5
		_Metallic("Metallic", Range(0,1)) = 0.0
	}
	// 每次只会根据情况来选择一个可执行的SubShader
	// 找到第一个<= Shader.maximumLOD 这个subShader执行;
	
	SubShader{
				Tags { "RenderType" = "Opaque" }
				LOD 600 // LOD-----------------这里设置为600
 
				CGPROGRAM
				// Physically based Standard lighting model, and enable shadows on all light types
				#pragma surface surf Standard fullforwardshadows
 
				// Use shader model 3.0 target, to get nicer looking lighting
				#pragma target 3.0
 
				sampler2D _MainTex;
 
				struct Input {
					float2 uv_MainTex;
				};
 
				half _Glossiness;
				half _Metallic;
				fixed4 _Color;
 
				void surf(Input IN, inout SurfaceOutputStandard o) {
					o.Albedo = fixed3(1.0, 0.0, 0.0);
				}
				ENDCG
			}
	SubShader{
					Tags { "RenderType" = "Opaque" }
					LOD 500 // LOD-----------------这里设置为500
 
					CGPROGRAM
					// Physically based Standard lighting model, and enable shadows on all light types
					#pragma surface surf Standard fullforwardshadows
 
					// Use shader model 3.0 target, to get nicer looking lighting
					#pragma target 3.0
 
					sampler2D _MainTex;
 
					struct Input {
						float2 uv_MainTex;
					};
 
					half _Glossiness;
					half _Metallic;
					fixed4 _Color;
 
					void surf(Input IN, inout SurfaceOutputStandard o) {
						o.Albedo = fixed3(0.0, 1.0, 0.0);
					}
					ENDCG
				}
	SubShader{
				Tags { "RenderType" = "Opaque" }
				LOD 400 // LOD-----------------这里设置为400
				CGPROGRAM
						// Physically based Standard lighting model, and enable shadows on all light types
						#pragma surface surf Standard fullforwardshadows
 
						// Use shader model 3.0 target, to get nicer looking lighting
						#pragma target 3.0
						sampler2D _MainTex;
						struct Input {
							float2 uv_MainTex;
						};
						half _Glossiness;
						half _Metallic;
						fixed4 _Color;
						void surf(Input IN, inout SurfaceOutputStandard o) {
							o.Albedo = fixed3(0.0, 0.0, 1.0);
						}
						ENDCG
					}
	FallBack "Diffuse"
}
```

描述如下：该 Shader 中存在三个 SubShader，LOD 分别设置为 600,500,400.。其对应的颜色分别设置为红色、绿色、蓝色

（2）创建一个立方体，使用该 Shader 对应的材质

（3）创建脚本 LODCtrl，通过键盘的 ABC 控制 Shader 内部的 LOD，该脚本挂载给相机。代码如下所示

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class LODCtrl : MonoBehaviour
{
    public Shader shader;//将Shader拖进来即可
    void Start() {
        Debug.Log(this.shader.maximumLOD);
    }
    // Update is called once per frame
    void Update() {
        if (Input.GetKeyDown(KeyCode.A))
        {
            // 当前这个shader最大的LOD_value;
            this.shader.maximumLOD = 600;
        }
        if (Input.GetKeyDown(KeyCode.B))
        {
            this.shader.maximumLOD = 500;
        }
        if (Input.GetKeyDown(KeyCode.C))
        {
            this.shader.maximumLOD = 300;
        }
 
    }
}
```

（4）运行，查看效果，通过按 ABC 按键，修改 maximumLOD 的值。查看 Cube 颜色的变化。