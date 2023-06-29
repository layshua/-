



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


  在补充下 ShadowCaster 的 pass 里可以成功获取额外灯光的数据了。 

  翻了下官方的源码，找到在 ShadowCasterPass.hlsl 的，里面使用的光照是_LightDirection，这个是官方自己定义的特定的 float3，它可以获得主光源和额外光源的方向，替换掉 MainLight.direction 后，就可以正常获取 addlight 的灯光方向了，效果如下图所示（并且保持 SRP Batcher）。
- ? 疑问？为什么可以获取额外光源??
![[Pasted image 20230629172200.png]]

![[a8cad8c92c63065f4d51e3c214dc8400_MD5.webp]]
