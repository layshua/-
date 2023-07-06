“_A_ **blit** _operation is a process of copying a source texture to a destination texture._”

## Overview

![[36a84effd6ac3b5a1f3198ede7445de0_MD5.jpg]]

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="striped"><tbody><tr><td>Graphics.Blit；<br>GrabPass；</td><td>你可能已经在冯乐乐的《UnityShader 入门精要》第 12 章屏幕后处理里用过 Graphics.Blit 了，但是只能在 BRP 里用，SRP 不支持 Graphics.Blit</td></tr><tr><td>CommandBuffer.Blit</td><td>是已过时的 API，且不支持 Single-Pass Instanced VR；<br>在 URP XR 项目中，请避免使用 CommandBuffer.Blit 及其它依赖于它的 API(比如 RenderingUtils.Blit)；</td></tr><tr><td>CommandBuffer.DrawMesh</td><td>&lt;How to perform a full screen blit in Single Pass Instanced rendering in XR (V13.1.9)&gt;<br>通过把画面绘制在一个占据全屏的 mesh(quad) 上来实现全屏后处理，支持 Single-Pass Instanced VR</td></tr><tr><td>ScriptableRenderPass 的 Blit() 方法</td><td>&lt;github.com/Cyanilux/URP_BlitRenderFeature&gt;<br>通过 ScriptableRenderPass 的 Blit() 方法来实现的全屏后处理，不支持 VR 显示；如果你需要在 VR 里显示，贴心的 Cyanilux 也提供了一个 cmd-DrawMesh 的版本<br>---------------------------------------<br>老版本：本质是 cmd.Blit()；<br>V14.0.6 以后：本质是 Blitter.BlitCameraTexture()；所以也是可以支持 Single-Pass VR 显示的；</td></tr><tr><td>Class Blitter</td><td>&lt;Perform a full screen blit in URP(V14.0.6)&gt;<br>在 URP V13.1 之后引入了 RTHandle(替换 RenderTargetIdentifier)，且不再使用 cmd.DrawMesh，而是用 Blitter.BlitCameraTexture() 实现 "Blit"</td></tr><tr><td>Full Screen Pass Renderer Feature</td><td>&lt;How to create a custom post-processing effect&gt;<br>在 unity 2022.2 (URP v14.0.6) 之后的版本，提供了现成的 Full Screen Pass Renderer Feature；<br>shader graph 也提供了新的节点： Create Node -&gt; Input -&gt; Universal -&gt; URP Sample Buffer</td></tr></tbody></table>

(URP 在不断升级，技术更迭… 这篇笔记里的东西过不了多久可能也会变成过时的东西，希望能帮到现在的你)  
附一个可能有用可能没用的相关讨论：

[https://forum.unity.com/threads/how-to-blit-in-urp-documentation-and-or-a-unity-blog-post-on-what-every-blit-function-does.1211508/](https://forum.unity.com/threads/how-to-blit-in-urp-documentation-and-or-a-unity-blog-post-on-what-every-blit-function-does.1211508/)


## **Class Blitter**

*   找不到教程，就只有 <[Perform a full screen blit in URP(V14.0.6)](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@16.0/manual/renderer-features/how-to-fullscreen-blit.html)>
*   Blitter 需要用到 [RTHandle](https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@13.1/api/UnityEngine.Rendering.RTHandle.html)，只有 2022.1 以上版本的 unity(URP v13.1.9) 才能使用；来自 <[link](https://forum.unity.com/threads/how-to-blit-in-urp-documentation-and-or-a-unity-blog-post-on-what-every-blit-function-does.1211508/#post-7740675)>
*   相关讨论：< [https://forum.unity.com/threads/urp-13-1-8-proper-rthandle-usage-in-a-renderer-feature.1341164/#post-8498552](https://forum.unity.com/threads/urp-13-1-8-proper-rthandle-usage-in-a-renderer-feature.1341164/#post-8498552)> 很有用诶!!!
*   看源码 “做实验” ：[https://github.com/Unity-Technologies/Graphics/blob/d4bc651868e95163e4aa464f65efa6c806d99dfc/Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blitter.cs#L260](https://github.com/Unity-Technologies/Graphics/blob/d4bc651868e95163e4aa464f65efa6c806d99dfc/Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blitter.cs#L260)

**先上实验成果**：使用 Class Blitter 实现的毛玻璃效果

[acnestis：URP 毛玻璃（升级至 URP_v14.0.6）](https://zhuanlan.zhihu.com/p/618752003)

下面是踩坑记录：（新手的踩坑记录可能没什么食用价值，请多包涵）

**public static void BlitTexture(){} 有 5 种重载：**

![[5373a34467cbee99eaddb4affd3c3ea9_MD5.png]]

①②③的 source 都是 RenderTargetIdentifier，所以它们都用【cmd.SetGlobalTexture("_BlitTexture", source)】来传入 source；  
而④⑤的 source 是 RTHandle，它们用【s_PropertyBlock.SetTexture("_BlitTexture", source)】传入 source；——我感觉这是个 bug——

**先**看为什么它们调用的方法不同：  
[MaterialPropertyBlock.SetTexture](https://docs.unity3d.com/2022.2/Documentation/ScriptReference/MaterialPropertyBlock.SetTexture.html)(int nameID, Texture value)；  
[CommandBuffer.SetGlobalTexture](https://docs.unity3d.com/2022.2/Documentation/ScriptReference/Rendering.CommandBuffer.SetGlobalTexture.html)(int nameID, RenderTargetIdentifier value)；  
RTHandle 类型可以隐式转换为 Texture 类型，所以 RTHandle 类型的 source 可以使用【MaterialPropertyBlock.SetTexture()】来传入 material，而 RenderTargetIdentifier 类型无法使用这个方法，它就只能用【CommandBuffer.SetGlobalTexture()】来传入；

**再**看 DrawTriangle(cmd, material, pass) 的源码，它最后用的是 s_PropertyBlock

```
static private void DrawTriangle(CommandBuffer cmd, Material material, int shaderPass){
    if (SystemInfo.graphicsShaderLevel < 30)
        cmd.DrawMesh(s_TriangleMesh, Matrix4x4.identity, material, 0, shaderPass, s_PropertyBlock);
    else
        cmd.DrawProcedural(Matrix4x4.identity, material, shaderPass, MeshTopology.Triangles, 3, 1, s_PropertyBlock);
}
```

而【cmd.SetGlobalTexture("_BlitTexture", source)】这句话并不能影响 s_PropertyBlock；那①②③这不离谱吗，先用 cmd.SetGlobalTexture() 重设了_BlitTexture，结果最后还是被 s_PropertyBlock 给覆盖了；

于是你就会遇到下面的 bug：

**Bug01**：如果你的代码里先执行 1 次方法④或⑤(即你在这一步改动了 s_PropertyBlock)，然后你再试图执行方法①或②或③，你会发现，不管你指定什么 source，unity 只会把你之前执行方法④或⑤时使用的 source 传入 material

**Bug02**：假设你之前从没有执行过方法④或⑤，现在你试图执行方法①或②或③，你会发现，不管你指定什么 source，unity 只会把_CameraColorAttachmentA 作为 source 传入 material，可能因为这个是 MaterialPropertyBlock 的默认值？

——但是我这么菜，这可能不是什么 BUG 是我用的方法不对 ？？？  
那先假设这是个 bug 的话，下面是解决方法：

**解决方法 01**：如果你的 source 是 RenderTargetIdentifier 类型的，那不要用这些封装好的 Blitter 方法了，直接搬源码吧 但是只用下面的源码的话并不能支持 Single-Pass VR，应该还需要一些其他设置 谁能教教我

```
MaterialPropertyBlock s_PropertyBlock = new MaterialPropertyBlock();
cmd.SetGlobalTexture("_BlitTexture", rti_tempTex);
cmd.SetRenderTarget(rti_Blur01);
cmd.DrawProcedural(Matrix4x4.identity, m_blurMat, 0, MeshTopology.Triangles, 3, 1, s_PropertyBlock);
// 天？为什么源码可以用，用封装好的函数就不行？？？
// 因为你现在new了一个空的MaterialPropertyBlock
```

**解决方法 02**：不要用 RenderTargetIdentifier，全用 RTHandle+【[RenderingUtils.ReAllocateIfNeeded()](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/api/UnityEngine.Rendering.Universal.RenderingUtils.html#UnityEngine_Rendering_Universal_RenderingUtils_ReAllocateIfNeeded_UnityEngine_Rendering_RTHandle__UnityEngine_RenderTextureDescriptor__UnityEngine_FilterMode_UnityEngine_TextureWrapMode_System_Boolean_System_Int32_System_Single_System_String_)】

*   之前我一直在用【RTHandles.Alloc(RenderTargetIdentifier)】来试图定义 RTHandle，结果是：执行第 1 个【Blitter.BlitCameraTexture(cmd,source,rth_tempA);】方法时可以将 scene color 传给 tempA，但是执行第 2 个【Blitter.BlitCameraTexture(cmd,rth_tempA,rth_tempB);】方法的时候会报错说 tempA 是 null，tempA 不能传给 tempB；我猜是因为执行第 2 个方法的时候 tempA 被清空了？？？为什么被清空，因为 RenderTargetIdentifier 是个短命的吗？
*   其实我不知道【RenderingUtils.ReAllocateIfNeeded】这函数到底是干嘛的，我猜是因为这个方法给它固定了 size（只在每次 OnCameraSetup 的时候重新分配一次），使 rth_tempA 成为了一个不会被覆盖 / 清空的 RTHandle；所以成功的关键在于【固定 size 的 RTHandle】吗？
*   最后一定记得要 Release！不然电脑会炸
*   然后我试着用【RTHandles.Alloc(in opaqueDesc, name: "_TempTex")】来取代【RenderingUtils.ReAllocateIfNeeded()】，也可以诶~~ 就是千万记得要在 OnCameraCleanup() 里执行 Release；

——总之就是我其实根本没理解 RTHandle 的原理，就在瞎试验…… 试验成功了也不知道为什么

**public static void BlitCameraTexture(){} 有 5 种重载：**

![[e06474d147ded16efad00671ec3ea2fe_MD5.png]]

你可以发现吧，BlitCameraTexture() 的本质是 BlitTexture()

一头雾水几个问题：（2023.3 URP 版本 v14.0.6)

✅What kind of shader/material can be used in a Blitter method?  
——使用 [_BlitTexture] 接受 Blitter 函数输入的 source，不需要手动定义[_BlitTexture]，可直接 include Blit.hlsl；

```
#include "Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl"
```

Blit.hlsl 可以提供 vertex shader (Vert), input structure (Attributes) and output strucutre (Varyings)

✅What is the difference between Blitter.BlitCameraTexture() and Blitter.BlitTexture()?  
——看源码可以发现吧，BlitCameraTexture() 的本质是 BlitTexture()

✅How could some Blitter.BlitTexture functions omit 【destination render target】 in its input parameters?  
——你可以在执行 Blitter 前手动添加一行 SetRenderTarget

```c
// Vector4 m_scaleBias = new Vector4(1,1,0,0);
// RenderTextureDescriptor opaqueDesc = renderingData.cameraData.cameraTargetDescriptor;
RTHandle source;// = renderer.cameraColorTargetHandle
int id_tempTex = Shader.PropertyToID("_TempTex");
RenderTargetIdentifier rti_tempTex = new RenderTargetIdentifier(id_tempTex);
cmd.GetTemporaryRT(id_tempTex, opaqueDesc, FilterMode.Bilinear);
cmd.SetRenderTarget(rti_tempTex);// 下面的方法不提供dest输入，所以这里SetRenderTarget手动设置dest
Blitter.BlitTexture(cmd,source,m_scaleBias,m_blurMat,0);//表格里的方法|4|
// 上面：把_CameraColorAttachmentA传入m_blurMat，最后输出给_TempTex
```

⬜I don't need material to do extra operations during blit, but some Blitter.BlitTexture functions require a Material as an input, How should I deal with them？

✅No matter what source parameter I set for the Blitter.BlitTexture(), it always gives 【_CameraColorAttachmentA】to【_BlitTexture】，why？(Actually it's not all the BlitTexture() methods that ignore the source input, only those with RenderTargetIdentifier source)  
——见上面的踩坑记录；

⬜I read the document but am still confused about the concept of RTHandle. I don't know which method I should pick and how to use it correctly. And most importantly, how do RTHandle, Blitter and RendererFeature work together? ([How to allocate and release RTHandle in RendereFeature](https://forum.unity.com/threads/general-guidance-on-rthandle-allocation-and-releasing-when-using-scriptablerenderfeature.1352801/);)  
——RTHandle 的优势在于 dynamic scaling support，但是我现在做毛玻璃不需要这种优势，我只需要最普通的 temporary RT；  
——这个链接 < [Upgrading to version 2022.1 of URP](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@13.1/manual/upgrade-guide-2022-1.html) > 挺有用；  
——<[link](https://forum.unity.com/threads/urp-13-1-8-proper-rthandle-usage-in-a-renderer-feature.1341164/#post-8801368)> 官方说了，“相关教程要等正式推出 RenderGraph 之后会有的”；