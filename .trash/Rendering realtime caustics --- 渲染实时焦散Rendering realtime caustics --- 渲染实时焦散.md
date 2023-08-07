
我将介绍渲染实时焦散的方法。目标不是生成物理上精确的结果，而是实现实时、可控、好看的水焦散效果。


使用焦散的一个好方法是**创建焦散体积 caustics volumes**。从本质上讲，我们创建了一个 volumes**，并将其定位在场景中应显示焦散的位置。

##  World-space UVs  
跳转到标题#世界空间UV

深度缓冲重建世界坐标，将xz 坐标作为世界空间 uv 采样焦散纹理

### [Jump to heading ##](#1.-reading-the-depth-buffer) 1. Reading the depth buffer  
跳到标题#1。读取深度缓冲区

First we sample the depth buffer using screen-space coordinates.  
首先，我们使用屏幕空间坐标对深度缓冲区进行采样。

```
struct Attributes{    float4 positionOS : POSITION;};struct Varyings{    float4 positionCS : SV_POSITION;};Varyings vert(Attributes IN){    Varyings OUT;    OUT.positionCS = TransformObjectToHClip(IN.positionOS.xyz);    return OUT;}half4 frag(Varyings IN) : SV_Target{    // calculate position in screen-space coordinates    float2 positionNDC = IN.positionCS.xy / _ScaledScreenParams.xy;    // sample scene depth using screen-space coordinates    #if UNITY_REVERSED_Z    real depth = SampleSceneDepth(positionNDC);    #else        real depth = lerp(UNITY_NEAR_CLIP_VALUE, 1, SampleSceneDepth(UV));    #endif}
```

In the code above we simply sample the scene depth texture using normalized screen-space coordinates.  
在上面的代码中，我们简单地使用归一化的屏幕空间坐标对场景深度纹理进行采样。

💡 The UNITY_REVERSED_Z block is used to handle platform-specific differences related to the depth buffer.  
💡 UNITY_REVERSED_Z块用于处理与深度缓冲区相关的平台特定差异。

### [Jump to heading ##](#2.-reconstructing-world-position-from-depth) 2. Reconstructing world position from depth  
跳到标题#2。从深度重构世界地位

Now that we have access to the value of the depth buffer given a screen-space coordinate, we can use it to compute a position in world-space coordinates using the following code in the fragment shader.  
现在，我们可以访问给定屏幕空间坐标的深度缓冲区的值，我们可以使用它在片段着色器中使用以下代码计算世界空间坐标中的位置。

```
// calculate position in world-space coordinatesfloat3 positionWS = ComputeWorldSpacePosition(positionNDC, depth, UNITY_MATRIX_I_VP);
```

If we want, we can visualize this in the scene using the following code in the fragment shader, where we take the fractional part of the world-space position (you can leave it out in the final shader, it is just for debugging).  
如果需要，我们可以在片段着色器中使用以下代码在场景中可视化这一点，我们在其中获取世界空间位置的小数部分（您可以在最终着色器中忽略它，这只是为了调试）。

```
half4 color = half4(frac(positionWS), 1.0);#if UNITY_REVERSED_Z    if(depth < 0.0001) return half4(0,0,0,1);#else    if(depth > 0.9999) return half4(0,0,0,1);#endifreturn color;
```

As you can see, this gives us world-space positions to map our caustics texture to.  
如您所见，这为我们提供了将焦散纹理映射到的世界空间位置。

<video src="" control></video>

### [Jump to heading ##](#3.-caustics-volume) 3. Caustics volume

We want the caustics volume to act like some sort of decal, where caustics show up wherever the volume intersects with the scene geometry. In order to achieve this, we calculate a bounding box mask in object space to limit the output of our shader.  
我们希望焦散体积的作用类似于某种贴花，焦散显示在体积与场景几何体相交的任何位置。为了实现这一点，我们在对象空间中计算边界框遮罩，以限制着色器的输出。

```
// calculate position in object-space coordinatesfloat3 positionOS = TransformWorldToObject(positionWS);// create bounding box maskfloat boundingBoxMask = all(step(positionOS, 0.5) * (1 - step(positionOS, -0.5)));
```

The way this bounding box mask works is by relying on the fact that the positions of the vertices of the box volume in object space, have a min/max of -0.5 and 0.5 respectively. We use a step function to mask out any pixels that are out of these bounds. The all function is used to make sure this happens in all of the x/y/z axes.  
此边界框遮罩的工作方式取决于对象空间中框体顶点的位置分别为-0.5和0.5的最小值/最大值。我们使用阶跃函数来屏蔽超出这些边界的任何像素。all函数用于确保在所有x/y/z轴上都发生这种情况。

By multiplying our output with this bounding box mask, we can limit the caustics to only be rendered where needed.  
通过将输出与此边界框遮罩相乘，我们可以将焦散限制为仅在需要的地方渲染。

![[0a98c174d45d0a7f3ca9070e765ed65f_MD5.jpg]]

Bounding box mask. 边界框遮罩。

## [Jump to heading #](#caustics) Caustics

We now have everything required to start displaying caustics. We will do this in multiple steps, each step improving on the effect.  
我们现在拥有了开始显示焦散所需的一切。我们将分多个步骤进行，每一步都会提高效果。

### [Jump to heading ##](#1.-mapping) 1. Mapping  
跳到标题#1。映射

We can map the caustics to the scene geometry by using the world-space coordinates as UVs. If you use the x and z components of the world-space position as UVs, the caustics will appear to be projected top-down onto the scene geometry.  
我们可以通过将世界空间坐标用作UV，将焦散映射到场景几何体。如果使用世界空间位置的x和z分量作为UV，则焦散将显示为自上而下投影到场景几何体上。  
However, nstead of using a fixed direction, it would be better to let the direction of the light play a role in how the caustics are oriented.  
但是，与其使用固定方向，不如让灯光的方向在焦散的定向方式中发挥作用。

```
half4x4 _MainLightDirection;TEXTURE2D(_CausticsTexture);SAMPLER(sampler_CausticsTexture);// calculate caustics texture UV coordinates (influenced by light direction)half2 uv = mul(positionWS, _MainLightDirection).xy;half4 caustics = SAMPLE_TEXTURE2D(_CausticsTexture, sampler_CausticsTexture, uv);
```

In the code above, we use the main light direction to influence the UVs that we use to sample the caustics texture. By doing this, the caustics follow the direction of light and appear to be projected onto the scene.  
在上面的代码中，我们使用主灯光方向来影响用于对焦散纹理进行采样的UV。通过执行此操作，焦散将跟随灯光的方向，并看起来像是投影到场景上。

![[ecbda491a6355fe79196937d259c518c_MD5.jpg]]

Mapping caustics to the scene geometry.  
将焦散映射到场景几何体。

To be able to access the light direction in our shader, a C# script is used with the following code.  
为了能够访问着色器中的光方向，使用C#脚本和以下代码。

```
var sunMatrix = RenderSettings.sun.transform.localToWorldMatrix;causticsMaterial.SetMatrix("_MainLightDirection", sunMatrix);
```

This code will simply write the direction of the light to the appropriate shader property so that it can be used in the shader.  
此代码只需将灯光的方向写入适当的着色器属性，即可在着色器中使用。

### [Jump to heading ##](#2.-scaling-and-movement) 2. Scaling and movement  
跳到标题#2。缩放和移动

Now that we have the caustics mapped to the scene geometry, let's get them moving. We can use a simple panner function to move the caustics texture. There is a parameter for controlling the speed, as well as the scale of the texture.  
现在我们已经将焦散映射到场景几何体，让我们移动它们。我们可以使用一个简单的平移函数来移动焦散纹理。有一个参数用于控制速度以及纹理的比例。

```
half2 Panner(half2 uv, half speed, half tiling){    return (half2(1, 0) * _Time.y * speed) + (uv * tiling);}half2 moving_uv = Panner(uv, _CausticsSpeed, 1 / _CausticsScale);
```

By using these modified UVs to sample the texture, we get moving caustics.  
通过使用这些修改的UV对纹理进行采样，我们可以获得移动焦散。

<video src="" control></video>

### [Jump to heading ##](#3.-multiple-textures) 3. Multiple textures  
跳到标题#3。多个纹理

Having just a single caustics texture moving around looks kind of weird. To fix this, we will put a second caustics texture on top of the first one. The trick here is to move the textures with a different strength and scale.  
只有一个焦散纹理四处移动看起来有点奇怪。若要修复此问题，我们将在第一个焦散纹理上放置第二个焦散贴图。这里的技巧是用不同的强度和比例移动纹理。

```
// create panning UVs for the causticshalf2 uv1 = Panner(uv, 0.75 * _CausticsSpeed, 1 / _CausticsScale);half2 uv2 = Panner(uv, 1 * _CausticsSpeed, -1 / _CausticsScale);half4 tex1 = SAMPLE_TEXTURE2D(_CausticsTexture, sampler_CausticsTexture, uv1);half4 tex2 = SAMPLE_TEXTURE2D(_CausticsTexture, sampler_CausticsTexture, uv2);half3 caustics = min(tex1, tex2) * _CausticsStrength;
```

We combine the 2 moving textures using a min operation which will return the minimum of the inputs. We multiply the result with a strength parameter to control how bright the caustics appear.  
我们使用最小操作来组合两个移动纹理，该操作将返回最小的输入。我们将结果与强度参数相乘，以控制焦散显示的亮度。

<video src="" control></video>

In the code above, I use 1 single speed/scale parameter and use it for both textures by multiplying by some magic numbers, but you can of course expose the parameters so you have full control over it.  
在上面的代码中，我使用了一个单独的速度/比例参数，并通过乘以一些幻数将其用于两个纹理，但您当然可以公开这些参数，这样您就可以完全控制它。  
The idea is just that the textures should have a different scale/speed and then the min operation will blend them.  
其想法只是纹理应该具有不同的比例/速度，然后最小操作将混合它们。

### [Jump to heading ##](#4.-chromatic-aberration) 4. Chromatic aberration  
跳到标题#4。色差

In real life, the light rays passing through the water surface get refracted because a change of medium occurs from air to water.  
在现实生活中，穿过水面的光线会发生折射，因为介质从空气变为水。  
During this process, different wavelength components of the light get refracted under different angles, causing the light ray to fall out into a rainbow pattern. This looks really beautiful and we will try to mimic it in our caustics effect.  
在这个过程中，不同波长的光在不同的角度下发生折射，导致光线形成彩虹图案。这看起来真的很漂亮，我们将尝试在焦散效果中模仿它。

The idea is to create function that will sample the caustics texture 3 times, each time adding an offset to the UVs. We then use these 3 samples as our r/g/b components of the final result.  
其想法是创建将对焦散纹理采样3次的函数，每次都向UV添加偏移。然后，我们使用这3个样本作为最终结果的r/g/b分量。

```
half3 SampleCaustics(half2 uv, half split){    half2 uv1 = uv + half2(split, split);    half2 uv2 = uv + half2(split, -split);    half2 uv3 = uv + half2(-split, -split);    half r = SAMPLE_TEXTURE2D(_CausticsTexture, sampler_CausticsTexture, uv1).r;    half g = SAMPLE_TEXTURE2D(_CausticsTexture, sampler_CausticsTexture, uv2).r;    half b = SAMPLE_TEXTURE2D(_CausticsTexture, sampler_CausticsTexture, uv3).r;    return half3(r, g, b);}// sample the causticshalf3 tex1 = SampleCaustics(uv1, _CausticsSplit);half3 tex2 = SampleCaustics(uv2, _CausticsSplit);
```

I got the idea to generate chromatic aberration like this from [this article](https://www.alanzucconi.com/2019/09/13/believable-caustics-reflections/) by Alan Zucconi.  
我从Alan Zucconi的这篇文章中得到了产生这样色差的想法。

<video src="" control></video>

### [Jump to heading ##](#5.-luminance-mask) 5. Luminance mask  
跳到标题#5。亮度遮罩

Next, we will improve our caustics effect by masking the caustics based on the luminance in the scene. This will make it so that the caustics appear less prominent in shadowed areas.  
接下来，我们将通过基于场景中的亮度遮罩焦散来改进焦散效果。这将使焦散在阴影区域中显得不那么突出。

```
// luminance maskhalf3 sceneColor = SampleSceneColor(positionNDC);half sceneLuminance = Luminance(sceneColor);half luminanceMask = lerp(1, sceneLuminance, _CausticsLuminanceMaskStrength);
```

The steps are simple. We sample the scene color, calculate the luminance, and then create a mask using a masking strength parameter.  
步骤很简单。我们对场景颜色进行采样，计算亮度，然后使用遮罩强度参数创建遮罩。

<video src="" control></video>

Since the luminance value will almost never be zero, the caustics will still show up in shadowed areas, just less bright.  
由于亮度值几乎永远不会为零，焦散仍将显示在阴影区域，只是亮度较低。  
If you really do not want caustics to show up there, you could work with a threshold value where caustics only show up if the luminance is over a certain value.  
如果确实不希望焦散显示在那里，可以使用一个阈值，其中只有当亮度超过某个值时才会显示焦散。

```
half luminanceMask = smoothstep(_CausticsLuminanceMaskStrength, _CausticsLuminanceMaskStrength + 0.1, sceneLuminance);
```

Another option is to sample the shadow map of the scene, and mask the caustics that way. This will result in hard cutoffs where the shadows are.  
另一个选项是对场景的阴影贴图进行采样，并以此方式遮罩焦散。这将导致阴影所在的位置出现硬剪切。

### [Jump to heading ##](#6.-edge-fade) 6. Edge fade  
跳到标题##6。边缘渐变

Currently the caustics have a hard cutoff at the edge of the caustics volume. To make the transition a bit softer, we introduce a soft edge fade mask.  
当前，焦散在焦散体积的边缘具有硬截断。为了使过渡更加柔和，我们引入了一种软边渐变遮罩。

```
half edgeFadeMask = 1 - saturate((distance(positionOS, 0) - _CausticsFadeRadius) / (1 - _CausticsFadeStrength));
```

We can control the radius and the strength of the mask. Using this, the edges of the caustics volume look a bit less harsh.  
我们可以控制遮罩的半径和强度。使用此选项，焦散体积的边看起来不那么粗糙。

<video src="" control></video>

### [Jump to heading ##](#7.-underwater-camera) 7. Underwater camera  
跳到标题##7。水下摄像机

To allow the camera to go inside of the caustics volume, we apply a cool trick where we use Cull Front to not render the front-facing polygons of the volume mesh, but only the back-facing (inside) polygons. We also use ZTest Always to always render those back-faces, even if other scene geometry is blocking it.  
为了允许摄影机进入焦散体积内部，我们应用了一个很酷的技巧，即使用Cull Front不渲染体积网格的前向多边形，而只渲染后向（内部）多边形。我们还使用ZTest Always始终渲染这些背面，即使其他场景几何体正在阻挡它。

```
Cull Front
ZTest Always
```

This simple change allows the camera to enter the caustics volume without any problem.  
这种简单的更改使摄影机可以毫无问题地进入焦散体积。

<video src="" control></video>

This approach was kindly explained to me by [A Beginner's Dev Blog](https://twitter.com/ABeginnersDevB1), [Harry Heath](https://twitter.com/harryh___h), [Mike V](https://twitter.com/RealtimeVFXMike) and [Anton Kudin](https://twitter.com/antonkudin) on Twitter, thanks!  
初学者开发博客、Harry Heath、Mike V和Anton Kudin在推特上友好地向我解释了这种方法，谢谢！

## [Jump to heading #](#conclusion) Conclusion  
跳转到标题#结论

And that's it! A nice looking caustics effect, as a result of many little techniques coming together.  
就这样！一个好看的焦散效果，这是许多小技巧结合在一起的结果。

<video src="" control></video>

You can get this shader including an additional volume system for easy placement on the Unity asset store. Any support would be greatly appreciated!  
您可以获得包含额外体积系统的着色器，以便轻松放置在Unity资源存储中。如有任何支持，我们将不胜感激！

## [Jump to heading #](#additional-resources) Additional resources  
跳转到标题#其他资源

[https://www.alanzucconi.com/2019/09/13/believable-caustics-reflections/](https://www.alanzucconi.com/2019/09/13/believable-caustics-reflections/)

[https://twitter.com/flogelz/status/1165251296720576512?ref_src=twsrc%5Etfw](https://twitter.com/flogelz/status/1165251296720576512?ref_src=twsrc%5Etfw)

Published March 2022 2022年3月出版