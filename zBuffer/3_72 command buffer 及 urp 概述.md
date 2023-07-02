项目地址 ： [here](https://github.com/logic-three-body/CommandBufferDemo)  

老师的项目地址： [here](https://github.com/Thousandyearsofwar/CommandBufferDemo)  

ppt:[here](https://docs.qq.com/slide/DUUxtdE5GcEtwZmNm?u=09b32b4448404afa92a380647f4970da)  

# RenderObjectFeature  


## Scriptable RenderObjectsFeature

### RenderObjectsFeature.cs  

完整代码见：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/SetRenderTarget/RenderObjectsFeature.cs)  

重载函数[ScriptableRendererFeature]：Create,AddRenderPasses  

在AddRenderPasses中传入ScriptableRenderPass  

renderfeature设置：  

```c
public enum RenderQueueType
{
    Opaque,
    Transparent,
}
[System.Serializable]
public class CustomCameraSettings
{
    public bool overrideCamera = false;
    public bool restoreCamera = true;
    public Vector4 offset;
    public float cameraFieldOfView = 60.0f;
}
//@@@RenderFeature 设置
//Event
public RenderPassEvent Event;

//-------Filter Setting-------
public RenderQueueType m_RenderQueueType;
public LayerMask m_LayerMask;
public LayerMask m_LayerMask1;
public string[] PassNames;
//----------------------------

//-------Render State Block-------
public Material override_Material;
public int overrideMaterialPassIndex = 0;
//Depth
public bool overriderDepthState = false;
public bool enableWrite = true;
public CompareFunction depthCompareFunction = CompareFunction.LessEqual;
//Stencil
public StencilStateData stencilSettings = new StencilStateData();
//Camera
public CustomCameraSettings cameraSettings = new CustomCameraSettings();

//--------------------------------

``` 

将对应设置传入DrawRenderersPass[继承自ScriptableRenderPass ] 构造函数:  

```c
//in Create():
m_DrawRendererPass = new DrawRenderersPass(this.name, Event, m_RenderQueueType, m_LayerMask, PassNames,
                                           override_Material, overrideMaterialPassIndex,
                                           overriderDepthState, enableWrite, depthCompareFunction,
                                           this.stencilSettings,
                                           cameraSettings);

``` 

### DrawRenderPass.cs  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/SetRenderTarget/Passes/DrawRenderersPass.cs)  

重载函数[ScriptableRenderPass ]：OnCameraSetup[准备阶段],Execute[执行阶段],OnCameraCleanup[结束/清理阶段]  

DrawRenderersPass构造函数：  

```
public DrawRenderersPass(string profilerTag, RenderPassEvent renderPassEvent, RenderQueueType renderQueueType, int layerMask, string[] shaderTags,
                         Material overrideMaterial, int overrideMaterialPassIndex,
                         bool overrideDepthState, bool enableWrite, CompareFunction depthCompareFunction,
                         StencilStateData stencilSettings,
                         CustomCameraSettings cameraSettings)
{
    this.m_ProfilingSampler = new ProfilingSampler(profilerTag);
    this.m_ProfilerTag = profilerTag;

    this.renderPassEvent = renderPassEvent;
    this.m_RenderQueueType = renderQueueType;

    //shaderTags数组转成 List if (shaderTags != null && shaderTags.Length > 0)
    {
        foreach (var passName in shaderTags)
        {
            m_ShaderTagIdList.Add(new ShaderTagId(passName));
        }
    }
    else
    {
        m_ShaderTagIdList.Add(new ShaderTagId("SRPDefaultUnlit"));
    }

    RenderQueueRange renderQueueRange = (renderQueueType == RenderQueueType.Transparent) ? RenderQueueRange.transparent : RenderQueueRange.opaque;
    //@@@Filtering Setting  
    //1.renderQueueRange渲染队列：不透明队列还是透明队列过滤  
    //2.layerMask Layer层级过滤
    this.m_FilteringSetting = new FilteringSettings(renderQueueRange, layerMask);

    this.overrideMaterial = overrideMaterial;
    this.overrideMaterialPassIndex = overrideMaterialPassIndex;


    //@@@RenderBlockState:Depth
    this.overrideDepthState = overrideDepthState;
    this.enableWrite = enableWrite;
    this.depthCompareFunction = depthCompareFunction;

    //@@@RenderBlockState:Stencil
    this.stencilSettings = stencilSettings;

    m_CameraSettings = cameraSettings;

    m_RenderStateBlock = new RenderStateBlock(RenderStateMask.Nothing);
} 
``` 

excute函数：  

```
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
{
    Test0(context,ref renderingData);
    //Test3(context, ref renderingData);
}

``` 

### Test0:CullingResults+DrawingSettings+FilteringSettings  

 ```
									   //使用哪个pass绘制
m_DrawingSetting = new DrawingSettings(m_ShaderTagIdList[0], sortingSettings)
{
    perObjectData = renderingData.perObjectData,
    mainLightIndex = renderingData.lightData.mainLightIndex,
    enableDynamicBatching = renderingData.supportsDynamicBatching,

    enableInstancing = camera.cameraType == CameraType.Preview ? false : true,
};
for (int i = 1; i < m_ShaderTagIdList.Count; ++i)
{
    m_DrawingSetting.SetShaderPassName(i, m_ShaderTagIdList[i]);
}

``` ```
Debug.Log(((string)m_DrawingSetting.GetShaderPassName(0)));

``` 

着色器中的lightmode：  

```
Shader "Unlit/UnlitTest _LightModeTest"
{
    Properties
    {
        _TestColor ("TestColor", Color) = (1, 1, 1, 1)
    }
    SubShader
    {
        Tags { "RenderPipeline" = "UniversalRenderPipeline" }
        LOD 100
        HLSLINCLUDE
		//...code...
        ENDHLSL

        Pass
        {
            Tags { "LightMode" = "m_CustomLightMode" }
            Cull Off
            ZWrite Off
            HLSLPROGRAM
			//...code...
            ENDHLSL

        }
    }
}


``` 

![[7e76c9f100074b8b1834be3cd0c9e5dc_MD5.png]] 

重写对应pass,使用重写材质的shader pass:  

```
m_DrawingSetting.overrideMaterial = overrideMaterial;
m_DrawingSetting.overrideMaterialPassIndex = overrideMaterialPassIndex;

``` <table class="table" style="width: 500px;"><colgroup><col width="250"><col width="249"></colgroup><tbody class="table-inner"><tr class="tr" style="height: 35px;"><td class="td" data-col="0"><div class="td-content"><p id="u496ee263" data-lake-id="u496ee263"><text id="uf67a910f">未重写（注释上述语句）</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="ub8f1893c" data-lake-id="ub8f1893c"><text id="u2e5db491">重写</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr><tr class="tr"><td class="td" data-col="0"><div class="td-content"><p id="u923840b5" data-lake-id="u923840b5" class="sr-rd-content-center"><img width="960" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635216602402-dfe928e3-deed-42df-98a5-a8fd83549e4b.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="ud46b3e94" data-lake-id="ud46b3e94" class="sr-rd-content-center"><img width="960" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635216313612-c100d952-b0af-423a-b3b4-4798f8263435.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr></tbody></table>

这时可以解决我们之前的疑惑，就是在[material pass index](#giMAX)时，为什么不同的material index会有不同的效果，因为我们重写的材质shader是Lit_RenderStateBlockTest，里面有多个pass，对应着不同的index，下面是Lit_RenderStateBlockTest简略代码（完整代码见:[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Shader/DrawMesh/Lit_RenderStateBlockTest.shader)）：  

```
Shader "Universal Render Pipeline/Lit_RenderStateBlockTest"
{
    Properties
    {
 		//...properties...
    }

    SubShader
    {
        Tags { "RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" "UniversalMaterialType" = "Lit" "IgnoreProjector" = "True" }
        LOD 300

        // pass 0
        //  Forward pass. Shades all light in a single pass. GI + emission + Fog
        Pass
        {
            // Lightmode matches the ShaderPassName set in UniversalRenderPipeline.cs. SRPDefaultUnlit and passes with
            // no LightMode tag are also rendered by Universal Render Pipeline
            Name "ForwardLit"
            Tags { "LightMode" = "UniversalForward" }
            //...code...      

        }
        
        // pass 1

        Pass
        {
            Name "ShadowCaster"
            Tags { "LightMode" = "ShadowCaster" }

		//...code...

        }
        
        //pass 2

        Pass
        {
            // Lightmode matches the ShaderPassName set in UniversalRenderPipeline.cs. SRPDefaultUnlit and passes with
            // no LightMode tag are also rendered by Universal Render Pipeline
            Name "Unlit"
            Tags { "LightMode" = "UniversalForward" }

 		//...code...

        }
        
        //pass 3

        Pass
        {
            Name "DepthOnly"
            Tags { "LightMode" = "DepthOnly" }

 		//...code...
        }
        
        //pass 4

        // This pass is used when drawing to a _CameraNormalsTexture texture
        Pass
        {
            Name "DepthNormals"
            Tags { "LightMode" = "DepthNormals" }

		//...code...

        }
        
        //pass 5

        // This pass it not used during regular rendering, only for lightmap baking.
        Pass
        {
            Name "Meta"
            Tags { "LightMode" = "Meta" }

		//...code...

        }
        
        //pass 6
        
        Pass
        {
            Name "Universal2D"
            Tags { "LightMode" = "Universal2D" }

		//...code...

        }
    }

    FallBack "Hidden/Universal Render Pipeline/FallbackError"
    CustomEditor "UnityEditor.Rendering.Universal.ShaderGUI.LitShader"
}


``` 

Test0完整函数：  

```
public void Test0(ScriptableRenderContext context, ref RenderingData renderingData)
{

    Camera camera = renderingData.cameraData.camera;
    ref CameraData cameraData = ref renderingData.cameraData;
    Rect pixelRect = camera.pixelRect;
    float cameraAspect = (float)pixelRect.width / (float)pixelRect.height;

    //@@@Drawing setting 渲染排序顺序
    SortingCriteria sortingCriteria = (m_RenderQueueType == RenderQueueType.Transparent) ?
        SortingCriteria.CommonTransparent :
    renderingData.cameraData.defaultOpaqueSortFlags;
    SortingSettings sortingSettings = new SortingSettings(camera)
    {
        criteria = sortingCriteria
        };

    //
    //等价于
    //m_DrawingSetting = CreateDrawingSettings(m_ShaderTagIdList, ref renderingData, sortingCriteria);
    m_DrawingSetting = new DrawingSettings(m_ShaderTagIdList[0], sortingSettings)
    {
        perObjectData = renderingData.perObjectData,
        mainLightIndex = renderingData.lightData.mainLightIndex,
        enableDynamicBatching = renderingData.supportsDynamicBatching,

        enableInstancing = camera.cameraType == CameraType.Preview ? false : true,
    };
    for (int i = 1; i < m_ShaderTagIdList.Count; ++i)
    {
        m_DrawingSetting.SetShaderPassName(i, m_ShaderTagIdList[i]);
    }
    //Debug.Log(((string)m_DrawingSetting.GetShaderPassName(0)));
    m_DrawingSetting.overrideMaterial = overrideMaterial;
    m_DrawingSetting.overrideMaterialPassIndex = overrideMaterialPassIndex;

    CommandBuffer commandBuffer = CommandBufferPool.Get();
    using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
    {
        if (m_CameraSettings.overrideCamera)
        {
            Matrix4x4 projectionMat = Matrix4x4.Perspective(
                m_CameraSettings.cameraFieldOfView, cameraAspect,
                camera.nearClipPlane, camera.farClipPlane);
            projectionMat = GL.GetGPUProjectionMatrix(projectionMat, cameraData.IsCameraProjectionMatrixFlipped());

            Matrix4x4 viewMat = cameraData.GetViewMatrix();
            Vector4 cameraTranslation = viewMat.GetColumn(3);

            viewMat.SetColumn(3, cameraTranslation + m_CameraSettings.offset);

            RenderingUtils.SetViewAndProjectionMatrices(commandBuffer, viewMat, projectionMat, false);
        }

        context.ExecuteCommandBuffer(commandBuffer);
        commandBuffer.Clear();

        context.DrawRenderers(renderingData.cullResults, ref m_DrawingSetting, ref m_FilteringSetting, ref m_RenderStateBlock);

        if (m_CameraSettings.overrideCamera && m_CameraSettings.restoreCamera && !cameraData.xrRendering)
        {
            RenderingUtils.SetViewAndProjectionMatrices(commandBuffer, cameraData.GetViewMatrix(), cameraData.GetGPUProjectionMatrix(), false);
        }
    }
    context.ExecuteCommandBuffer(commandBuffer);
    CommandBufferPool.Release(commandBuffer);

}

``` 

### Test1:Override RenderStateBlock  

重写深度写入：  

```
//@@@RenderStateBlock
if (overrideDepthState)
{
    m_RenderStateBlock.mask |= RenderStateMask.Depth;
    m_RenderStateBlock.depthState = new DepthState(this.enableWrite, this.depthCompareFunction);
}

``` <table class="table" style="width: 1000px;"><colgroup><col width="250"><col width="250"><col width="250"><col width="249"></colgroup><tbody class="table-inner"><tr class="tr"><td class="td" data-col="0"><div class="td-content"><p id="u40556ee5" data-lake-id="u40556ee5"><text id="u246a84db">Less Equal</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="u030ca1cd" data-lake-id="u030ca1cd"><text id="ub8b94f70">Always</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="2"><div class="td-content"><p id="u5e7a4c28" data-lake-id="u5e7a4c28"><text id="u158d3057">Greater</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="3"><div class="td-content"><p id="uc4309718" data-lake-id="uc4309718"><text id="u7e3449ff">Equal</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr><tr class="tr"><td class="td" data-col="0"><div class="td-content"><p id="u95a1c6ff" data-lake-id="u95a1c6ff" class="sr-rd-content-center"><img width="960" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635321012556-730d0f57-4120-437c-929c-6c06209d7889.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="u168cfdbb" data-lake-id="u168cfdbb" class="sr-rd-content-center"><img width="960" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635321123997-30b2b1a4-0e88-4061-821a-d183a1057677.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="2"><div class="td-content"><p id="u0ab392f2" data-lake-id="u0ab392f2" class="sr-rd-content-center"><img width="960" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635321059840-412ddd33-10d2-4eab-8d2e-a451400dd01c.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="3"><div class="td-content"><p id="ufcab037b" data-lake-id="ufcab037b" class="sr-rd-content-center"><img width="960" class="image image-preview image-hide" alt="image.png" draggable="true"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr></tbody></table>

完整函数：  

```
//Test1:Override RenderStateBlock
public void Test1(ScriptableRenderContext context, ref RenderingData renderingData)
{
    SortingCriteria sortingCriteria = (m_RenderQueueType == RenderQueueType.Transparent) ?
        SortingCriteria.CommonTransparent :
    renderingData.cameraData.defaultOpaqueSortFlags;

    //m_DrawingSetting = CreateDrawingSettings(m_ShaderTagIdList, ref renderingData, sortingCriteria);
    Camera camera = renderingData.cameraData.camera;
    ref CameraData cameraData = ref renderingData.cameraData;
    Rect pixelRect = camera.pixelRect;
    float cameraAspect = (float)pixelRect.width / (float)pixelRect.height;

    //@@@Drawing setting
    SortingSettings sortingSettings = new SortingSettings(camera)
    {
        criteria = sortingCriteria
        };

    m_DrawingSetting = new DrawingSettings(m_ShaderTagIdList[0], sortingSettings)
    {
        perObjectData = renderingData.perObjectData,
        mainLightIndex = renderingData.lightData.mainLightIndex,
        enableDynamicBatching = renderingData.supportsDynamicBatching,

        enableInstancing = camera.cameraType == CameraType.Preview ? false : true,
    };
    for (int i = 0; i < m_ShaderTagIdList.Count; ++i)
    {
        m_DrawingSetting.SetShaderPassName(i, m_ShaderTagIdList[i]);
    }

    m_DrawingSetting.overrideMaterial = overrideMaterial;
    m_DrawingSetting.overrideMaterialPassIndex = overrideMaterialPassIndex;

    //@@@RenderStateBlock
    if (overrideDepthState)
    {
        m_RenderStateBlock.mask |= RenderStateMask.Depth;
        m_RenderStateBlock.depthState = new DepthState(this.enableWrite, this.depthCompareFunction);
    }

    if (stencilSettings.overrideStencilState)
    {
        StencilState stencilState = StencilState.defaultValue;
        stencilState.enabled = true;
        stencilState.SetCompareFunction(stencilSettings.stencilCompareFunction);
        stencilState.SetPassOperation(stencilSettings.passOperation);
        stencilState.SetFailOperation(stencilSettings.failOperation);
        stencilState.SetZFailOperation(stencilSettings.zFailOperation);

        m_RenderStateBlock.mask |= RenderStateMask.Stencil;
        m_RenderStateBlock.stencilReference = stencilSettings.stencilReference;
        m_RenderStateBlock.stencilState = stencilState;
    }


    CommandBuffer commandBuffer = CommandBufferPool.Get();
    using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
    {
        if (m_CameraSettings.overrideCamera)
        {
            Matrix4x4 projectionMat = Matrix4x4.Perspective(
                m_CameraSettings.cameraFieldOfView, cameraAspect,
                camera.nearClipPlane, camera.farClipPlane);
            projectionMat = GL.GetGPUProjectionMatrix(projectionMat, cameraData.IsCameraProjectionMatrixFlipped());

            Matrix4x4 viewMat = cameraData.GetViewMatrix();
            Vector4 cameraTranslation = viewMat.GetColumn(3);

            viewMat.SetColumn(3, cameraTranslation + m_CameraSettings.offset);

            RenderingUtils.SetViewAndProjectionMatrices(commandBuffer, viewMat, projectionMat, false);
        }


        // commandBuffer.SetRenderTarget(renderTextureID,
        // RenderBufferLoadAction.DontCare, RenderBufferStoreAction.Store,
        // RenderBufferLoadAction.DontCare, RenderBufferStoreAction.DontCare);

        context.ExecuteCommandBuffer(commandBuffer);
        commandBuffer.Clear();

        context.DrawRenderers(renderingData.cullResults, ref m_DrawingSetting, ref m_FilteringSetting, ref m_RenderStateBlock);

        //commandBuffer.SetRenderTarget(BuiltinRenderTextureType.CameraTarget,BuiltinRenderTextureType.CameraTarget);
        if (m_CameraSettings.overrideCamera && m_CameraSettings.restoreCamera && !cameraData.xrRendering)
        {
            RenderingUtils.SetViewAndProjectionMatrices(commandBuffer, cameraData.GetViewMatrix(), cameraData.GetGPUProjectionMatrix(), false);
        }
    }
    context.ExecuteCommandBuffer(commandBuffer);
    CommandBufferPool.Release(commandBuffer);
}

``` 

### Test2: Match Subshader RenderType Tags  

重写RenderType：  

如果找不到新的shaderTagId，则返回默认的。  

这里默认的深度比较是Always  

```
//@@@Default RenderStateBlock
RenderStateBlock defaultRenderStateBlock = new RenderStateBlock(RenderStateMask.Depth);
defaultRenderStateBlock.depthState = new DepthState(true, CompareFunction.Always);

m_RenderStateBlocks = new NativeArray(2, Allocator.Temp);
m_RenderStateBlocks[0] = m_RenderStateBlock;
m_RenderStateBlocks[1] = defaultRenderStateBlock;


//@@@ShaderTagId
ShaderTagId renderType = new ShaderTagId("Opaque");
ShaderTagId fallBack = new ShaderTagId();//Catch all
NativeArray renderTypes = new NativeArray(2, Allocator.Temp);
renderTypes[0] = renderType;
renderTypes[1] = fallBack;
//测试内容:
//更改Shader中的RenderType，改为"Opaque0",再改回去，说明renderType=Opaque时,使用m_RenderStateBlock
//所以我们可以根据RenderType去批量指定RenderStateBlock

//...

//@@@Only work for RenderType Tag
context.DrawRenderers(renderingData.cullResults, ref m_DrawingSetting, ref m_FilteringSetting, renderTypes, m_RenderStateBlocks); 
``` 

正常情况：("RenderType" = "Opaque")  

```
Shader "Unlit/UnlitTest _LightModeTest"
{
    Properties
    {
        _TestColor ("TestColor", Color) = (1, 1, 1, 1)
    }
    SubShader
    {
        Tags { "RenderType" = "Opaque" "RenderPipeline" = "UniversalPipeline" "UniversalMaterialType" = "Lit" "IgnoreProjector" = "True" }
		//...
    }
}

``` 

![[622b658f714a31e0bdf2458ac5e9835c_MD5.png]] 

异常情况：("RenderType" = "Opaque0")  

```
Shader "Unlit/UnlitTest _LightModeTest"
{
    Properties
    {
        _TestColor ("TestColor", Color) = (1, 1, 1, 1)
    }
    SubShader
    {
        Tags { "RenderType" = "Opaque0" "RenderPipeline" = "UniversalPipeline" "UniversalMaterialType" = "Lit" "IgnoreProjector" = "True" }
		//...
    }
}

``` 

![[e90fac47167c89d90c41d37b19d36283_MD5.png]] 

完整函数：  

```
//Test2: Match Subshader RenderType Tags
public void Test2(ScriptableRenderContext context, ref RenderingData renderingData)
{
    SortingCriteria sortingCriteria = (m_RenderQueueType == RenderQueueType.Transparent) ?
        SortingCriteria.CommonTransparent :
    renderingData.cameraData.defaultOpaqueSortFlags;

    //m_DrawingSetting = CreateDrawingSettings(m_ShaderTagIdList, ref renderingData, sortingCriteria);
    Camera camera = renderingData.cameraData.camera;
    ref CameraData cameraData = ref renderingData.cameraData;
    Rect pixelRect = camera.pixelRect;
    float cameraAspect = (float)pixelRect.width / (float)pixelRect.height;

    //@@@Drawing setting
    SortingSettings sortingSettings = new SortingSettings(camera)
    {
        criteria = sortingCriteria
        };

    m_DrawingSetting = new DrawingSettings(m_ShaderTagIdList[0], sortingSettings)
    {
        perObjectData = renderingData.perObjectData,
        mainLightIndex = renderingData.lightData.mainLightIndex,
        enableDynamicBatching = renderingData.supportsDynamicBatching,

        enableInstancing = camera.cameraType == CameraType.Preview ? false : true,
    };
    for (int i = 0; i < m_ShaderTagIdList.Count; ++i)
    {
        m_DrawingSetting.SetShaderPassName(i, m_ShaderTagIdList[i]);
    }

    m_DrawingSetting.overrideMaterial = overrideMaterial;
    m_DrawingSetting.overrideMaterialPassIndex = overrideMaterialPassIndex;

    //@@@Custom RenderStateBlock
    if (overrideDepthState)
    {
        m_RenderStateBlock.mask |= RenderStateMask.Depth;
        m_RenderStateBlock.depthState = new DepthState(this.enableWrite, this.depthCompareFunction);
    }

    if (stencilSettings.overrideStencilState)
    {
        StencilState stencilState = StencilState.defaultValue;
        stencilState.enabled = true;
        stencilState.SetCompareFunction(stencilSettings.stencilCompareFunction);
        stencilState.SetPassOperation(stencilSettings.passOperation);
        stencilState.SetFailOperation(stencilSettings.failOperation);
        stencilState.SetZFailOperation(stencilSettings.zFailOperation);

        m_RenderStateBlock.mask |= RenderStateMask.Stencil;
        m_RenderStateBlock.stencilReference = stencilSettings.stencilReference;
        m_RenderStateBlock.stencilState = stencilState;
    }

    //@@@Default RenderStateBlock
    RenderStateBlock defaultRenderStateBlock = new RenderStateBlock(RenderStateMask.Depth);
    defaultRenderStateBlock.depthState = new DepthState(true, CompareFunction.Always);

    m_RenderStateBlocks = new NativeArray(2, Allocator.Temp);
    m_RenderStateBlocks[0] = m_RenderStateBlock;
    m_RenderStateBlocks[1] = defaultRenderStateBlock;


    //@@@ShaderTagId
    ShaderTagId renderType = new ShaderTagId("Opaque");
    ShaderTagId fallBack = new ShaderTagId();//Catch all
    NativeArray renderTypes = new NativeArray(2, Allocator.Temp);
    renderTypes[0] = renderType;
    renderTypes[1] = fallBack;
    //测试内容:
    //更改Shader中的RenderType，改为"Opaque0",再改回去，说明renderType=Opaque时,使用m_RenderStateBlock
    //所以我们可以根据RenderType去批量指定RenderStateBlock

    CommandBuffer commandBuffer = CommandBufferPool.Get();
    using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
    {
        if (m_CameraSettings.overrideCamera)
        {
            Matrix4x4 projectionMat = Matrix4x4.Perspective(
                m_CameraSettings.cameraFieldOfView, cameraAspect,
                camera.nearClipPlane, camera.farClipPlane);
            projectionMat = GL.GetGPUProjectionMatrix(projectionMat, cameraData.IsCameraProjectionMatrixFlipped());

            Matrix4x4 viewMat = cameraData.GetViewMatrix();
            Vector4 cameraTranslation = viewMat.GetColumn(3);

            viewMat.SetColumn(3, cameraTranslation + m_CameraSettings.offset);

            RenderingUtils.SetViewAndProjectionMatrices(commandBuffer, viewMat, projectionMat, false);
        }

        context.ExecuteCommandBuffer(commandBuffer);
        commandBuffer.Clear();

        //@@@Only work for RenderType Tag
        context.DrawRenderers(renderingData.cullResults, ref m_DrawingSetting, ref m_FilteringSetting, renderTypes, m_RenderStateBlocks);

        if (m_CameraSettings.overrideCamera && m_CameraSettings.restoreCamera && !cameraData.xrRendering)
        {
            RenderingUtils.SetViewAndProjectionMatrices(commandBuffer, cameraData.GetViewMatrix(), cameraData.GetGPUProjectionMatrix(), false);
        }
    }

    context.ExecuteCommandBuffer(commandBuffer);
    CommandBufferPool.Release(commandBuffer);
    m_RenderStateBlocks.Dispose();
    renderTypes.Dispose();
} 
``` 

### Test3: Match Subshader or Pass’s TagName+TagVlaues  

重写pass的tagname：UniversalForward  

```
//@@@ShaderTagId
ShaderTagId tagName = new ShaderTagId("LightMode");
bool isPassTagName = true;

ShaderTagId renderType = new ShaderTagId("UniversalForward");
ShaderTagId fallBack = new ShaderTagId();
NativeArray tagValues = new NativeArray(2, Allocator.Temp);
tagValues[0] = renderType;
tagValues[1] = fallBack;
//...
//@@@Not Only work for RenderType Tag
context.DrawRenderers(renderingData.cullResults, ref m_DrawingSetting, ref m_FilteringSetting, tagName, isPassTagName, tagValues, m_RenderStateBlocks); 
``` 

![[78e987a13660e7e3d797dd8b196e9bbe_MD5.png]] 

重写pass的tagname：DepthNormals  

注意此时要把重写材质的index改为4，因为这对应着对应的pass  

```
string lightmode = "DepthNormals";
ShaderTagId renderType = new ShaderTagId(lightmode);
ShaderTagId fallBack = new ShaderTagId();
NativeArray tagValues = new NativeArray(2, Allocator.Temp);
tagValues[0] = renderType;
tagValues[1] = fallBack; 
``` 

![[1c0cbca00e78c44f6bd99211083ce9a5_MD5.png]] 

完整函数：  

```
//Test3: Match Subshader or Pass’s TagName+TagVlaues 
public void Test3(ScriptableRenderContext context, ref RenderingData renderingData)
{
    SortingCriteria sortingCriteria = (m_RenderQueueType == RenderQueueType.Transparent) ?
        SortingCriteria.CommonTransparent :
    renderingData.cameraData.defaultOpaqueSortFlags;

    //m_DrawingSetting = CreateDrawingSettings(m_ShaderTagIdList, ref renderingData, sortingCriteria);
    Camera camera = renderingData.cameraData.camera;
    ref CameraData cameraData = ref renderingData.cameraData;
    Rect pixelRect = camera.pixelRect;
    float cameraAspect = (float)pixelRect.width / (float)pixelRect.height;

    //@@@Drawing setting
    SortingSettings sortingSettings = new SortingSettings(camera)
    {
        criteria = sortingCriteria
        };

    m_DrawingSetting = new DrawingSettings(m_ShaderTagIdList[0], sortingSettings)
    {
        perObjectData = renderingData.perObjectData,
        mainLightIndex = renderingData.lightData.mainLightIndex,
        enableDynamicBatching = renderingData.supportsDynamicBatching,

        enableInstancing = camera.cameraType == CameraType.Preview ? false : true,
    };
    for (int i = 0; i < m_ShaderTagIdList.Count; ++i)
    {
        m_DrawingSetting.SetShaderPassName(i, m_ShaderTagIdList[i]);
    }

    m_DrawingSetting.overrideMaterial = overrideMaterial;
    m_DrawingSetting.overrideMaterialPassIndex = overrideMaterialPassIndex;

    //@@@Custom RenderStateBlock
    if (overrideDepthState)
    {
        m_RenderStateBlock.mask |= RenderStateMask.Depth;
        m_RenderStateBlock.depthState = new DepthState(this.enableWrite, this.depthCompareFunction);
    }

    if (stencilSettings.overrideStencilState)
    {
        StencilState stencilState = StencilState.defaultValue;
        stencilState.enabled = true;
        stencilState.SetCompareFunction(stencilSettings.stencilCompareFunction);
        stencilState.SetPassOperation(stencilSettings.passOperation);
        stencilState.SetFailOperation(stencilSettings.failOperation);
        stencilState.SetZFailOperation(stencilSettings.zFailOperation);

        m_RenderStateBlock.mask |= RenderStateMask.Stencil;
        m_RenderStateBlock.stencilReference = stencilSettings.stencilReference;
        m_RenderStateBlock.stencilState = stencilState;
    }

    //@@@Default RenderStateBlock
    RenderStateBlock defaultRenderStateBlock = new RenderStateBlock(RenderStateMask.Depth);
    defaultRenderStateBlock.depthState = new DepthState(true, CompareFunction.Always);

    m_RenderStateBlocks = new NativeArray(2, Allocator.Temp);
    m_RenderStateBlocks[0] = m_RenderStateBlock;
    m_RenderStateBlocks[1] = defaultRenderStateBlock;


    //@@@ShaderTagId
    ShaderTagId tagName = new ShaderTagId("LightMode");
    bool isPassTagName = true;

    ShaderTagId renderType = new ShaderTagId("UniversalForward");
    ShaderTagId fallBack = new ShaderTagId();
    NativeArray tagValues = new NativeArray(2, Allocator.Temp);
    tagValues[0] = renderType;
    tagValues[1] = fallBack;


    CommandBuffer commandBuffer = CommandBufferPool.Get();
    using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
    {
        if (m_CameraSettings.overrideCamera)
        {
            Matrix4x4 projectionMat = Matrix4x4.Perspective(
                m_CameraSettings.cameraFieldOfView, cameraAspect,
                camera.nearClipPlane, camera.farClipPlane);
            projectionMat = GL.GetGPUProjectionMatrix(projectionMat, cameraData.IsCameraProjectionMatrixFlipped());

            Matrix4x4 viewMat = cameraData.GetViewMatrix();
            Vector4 cameraTranslation = viewMat.GetColumn(3);

            viewMat.SetColumn(3, cameraTranslation + m_CameraSettings.offset);

            RenderingUtils.SetViewAndProjectionMatrices(commandBuffer, viewMat, projectionMat, false);
        }

        context.ExecuteCommandBuffer(commandBuffer);
        commandBuffer.Clear();

        //@@@Not Only work for RenderType Tag
        context.DrawRenderers(renderingData.cullResults, ref m_DrawingSetting, ref m_FilteringSetting, tagName, isPassTagName, tagValues, m_RenderStateBlocks);

        if (m_CameraSettings.overrideCamera && m_CameraSettings.restoreCamera && !cameraData.xrRendering)
        {
            RenderingUtils.SetViewAndProjectionMatrices(commandBuffer, cameraData.GetViewMatrix(), cameraData.GetGPUProjectionMatrix(), false);
        }
    }

    context.ExecuteCommandBuffer(commandBuffer);
    CommandBufferPool.Release(commandBuffer);
    m_RenderStateBlocks.Dispose();
    tagValues.Dispose();
} 
``` 

# Insert Command Buffer [lifetime]  

<table class="table" style="width: 500px;"><colgroup><col width="250"><col width="249"></colgroup><tbody class="table-inner"><tr class="tr"><td class="td" data-col="0"><div class="td-content"><p id="ub272e6a2" data-lake-id="ub272e6a2"><text id="u615c2d14">Before SkyBox</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="u7e97642f" data-lake-id="u7e97642f"><text id="u342ecc05">Afte SkyBox</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr><tr class="tr"><td class="td" data-col="0"><div class="td-content"><p id="ucec9e3a7" data-lake-id="ucec9e3a7" class="sr-rd-content-center"><img width="960" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635332035896-079210d8-6379-4c9b-a328-04a7bd320887.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="u2656517a" data-lake-id="u2656517a" class="sr-rd-content-center"><img width="960" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635332089251-0601db73-dda5-4610-96bf-ee6b16b5794f.png?x-oss-process=image%2Fresize%2Cw_1125%2Climit_0"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr></tbody></table>

urp这里和build-in管线有些区别，只有drawskybox的事件有效。  

from urp.drawskybox:  

```
context.DrawSkybox(renderingData.cameraData.camera);

``` 

command buffer可插入蓝色部分（build-in）  

![[dfcb5c0892d52f228e774efbab53547a_MD5.png]] 

```
public class InsertCommandBufferTest : MonoBehaviour
{
    public RenderTexture renderTexture;
    public Material m_Material;
    public CameraEvent m_CameraEvent;

    private void OnEnable()
    {
        if (m_Material != null)
        {
            var renderer = (Renderer)GetComponents().GetValue(0);
            if (renderTexture == null)
            {
                renderTexture = RenderTexture.GetTemporary(Camera.main.pixelHeight, Camera.main.pixelWidth, 16, RenderTextureFormat.R8);
            }
            else
            {
                var commandBuffer = new CommandBuffer();
                commandBuffer.SetRenderTarget(renderTexture);
                commandBuffer.ClearRenderTarget(true, true, Color.black);
                commandBuffer.DrawRenderer(renderer, m_Material, 0, 0);
                Camera.main.AddCommandBuffer(m_CameraEvent, commandBuffer);

                commandBuffer.Release();
            }
        }
    }
    private void OnDisable()
    {
        Camera.main.RemoveAllCommandBuffers();
    }
} 
``` 

# RenderTexture Test  

完整代码: [RenderTextureRequestPass](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/SetRenderTarget/Passes/RenderTextureRequestPass.cs) [RenderTextureRequestRenderFeature](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/SetRenderTarget/RenderTextureRequestRenderFeature.cs)  

## Test0 : new RenderTexture(cameraDescriptor)  

![[88d175a3d33cc4a22a2e0694dffeb49a_MD5.png]] 

完整函数：  

```
void RTRequestTest0_0(CommandBuffer commandBuffer, ref RenderingData renderingData)
{
    RenderTextureDescriptor cameraDescriptor = renderingData.cameraData.cameraTargetDescriptor;
    //1.renderTexture == null必要性 只创建一张RenderTexture
    //2.IsCreated 防止未被创建
    //3.CoreUtils.Destroy 安全Destory RenderTexture
    //4.编辑器上面的bug 
    //从哪里参考:ShadowsMidtonesHighlightsEditor.cs
    //5.renderTexture.Create();
    if (renderTexture == null || !renderTexture.IsCreated())
    {
        CoreUtils.Destroy(renderTexture);
        renderTexture = new RenderTexture(cameraDescriptor);
        renderTexture.name = "RequestRT";
    }
    //配置RenderTarget[渲染目标]
    ConfigureTarget(renderTexture);
}

``` 

## Test1:RenderTexture.GetTemporary(cameraDescriptor)  

（效果同Test0）  

```
void RTRequestTest0_1(CommandBuffer commandBuffer, ref RenderingData renderingData)
{
    RenderTextureDescriptor cameraDescriptor = renderingData.cameraData.cameraTargetDescriptor;
    //1.申请临时的RenderTexture 必须手动Release掉
    //需要时刻关心生命周期！！
    //2.释放资源的区别:RenderTexture.Release和Destroy。https://zhuanlan.zhihu.com/p/41251356 
    //RenderTexture.Release释放显存，内存不释放
    //Destroy会把Object销毁的同时连带显存释放掉
    //所以出于性能考虑，频繁使用Destory会加重申请内存的负担
    //从哪里参考:ShadowUtils.cs
    renderTexture = RenderTexture.GetTemporary(cameraDescriptor);
    renderTexture.name = "RequestRT";
    ConfigureTarget(renderTexture);
}

``` 

## Test2:Shader.PropertyToID  

```
renderTextureID = Shader.PropertyToID("Request_ID");

``` 

MRT shader:  

(完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Shader/SetRenderTarget/MRT.shader))  

 ```
 TEXTURE2D(Request_ID);	SAMPLER(sampler_Request_ID);

``` 

![[7ada9f0508811e99ea38477b41c9010e_MD5.png]] 

![[5c435e9c665cb6c56368416569d57a24_MD5.png]] 

![[a5d092a1854937fac9ed654cf1d6a863_MD5.png]] 

完整函数：  

```
void RTRequestTest0_2(CommandBuffer commandBuffer, ref RenderingData renderingData)
{
  RenderTextureDescriptor cameraDescriptor = renderingData.cameraData.cameraTargetDescriptor;
  //1.Shader.PropertyToID的作用：
  //用字符串获取一个唯一的HashID
  //相当于一个全局的id，需要保证其他Material着色使用这张RenderTexture时，这张RenderTexture渲染完成，没有被Release掉
  //就不用手动setTexture了
  renderTextureID = Shader.PropertyToID("Request_ID");
  //2.RenderTexture.GetTemporary和commandBuffer.GetTemporaryRT[申请RenderTexture的区别]
  //1.RenderTexture.GetTemporary需要手动释放掉
  //2.使用CommandBuffer申请的临时的RenderTexture如果都没有显式地使用(Release)TemporaryRT，在相机完成渲染后或Graphics.ExecuteCommandBuffer完成后被移除(Remove)(Destory?)
  commandBuffer.GetTemporaryRT(renderTextureID, cameraDescriptor, FilterMode.Bilinear);

  ConfigureTarget(renderTextureID);
}

``` 

## Test3:renderTargetHandle  

（效果同Test0）  

完整函数：  

```
//使用URP的renderTargetHandle获取PropertyToID
void RTRequestTest0_3(CommandBuffer commandBuffer, ref RenderingData renderingData)
{
  RenderTextureDescriptor cameraDescriptor = renderingData.cameraData.cameraTargetDescriptor;

  renderTargetHandle.Init("Request_Handle");
  //commandBuffer.GetTemporaryRT(renderTargetHandle.Identifier(), cameraDescriptor, FilterMode.Bilinear);//错误示范
  commandBuffer.GetTemporaryRT(renderTargetHandle.id, cameraDescriptor, FilterMode.Bilinear);

  ConfigureTarget(renderTargetHandle.Identifier());
}

``` 

RenderTargetHandle.cs(部分代码):  

```
using UnityEngine.Scripting.APIUpdating;

namespace UnityEngine.Rendering.Universal
{
    // RenderTargetHandle can be thought of as a kind of ShaderProperty string hash
    [MovedFrom("UnityEngine.Rendering.LWRP")] public struct RenderTargetHandle
    {
        public int id { set; get; }
        private RenderTargetIdentifier rtid { set; get; }

        public static readonly RenderTargetHandle CameraTarget = new RenderTargetHandle {id = -1 };

        public RenderTargetHandle(RenderTargetIdentifier renderTargetIdentifier)
        {
            id = -2;
            rtid = renderTargetIdentifier;
        }

			//...

        public void Init(string shaderProperty)
        {
            // Shader.PropertyToID returns what is internally referred to as a "ShaderLab::FastPropertyName".
            // It is a value coming from an internal global std::map that converts shader property strings into unique integer handles (that are faster to work with).
            id = Shader.PropertyToID(shaderProperty);
        }

        public void Init(RenderTargetIdentifier renderTargetIdentifier)
        {
            id = -2;
            rtid = renderTargetIdentifier;
        }

        public RenderTargetIdentifier Identifier()
        {
            if (id == -1)
            {
                return BuiltinRenderTextureType.CameraTarget;
            }
            if (id == -2)
            {
                return rtid;
            }
            return new RenderTargetIdentifier(id);
        }

				//...
    }
} 
``` 

## [Wrong]Test4:commandBuffer.SetRenderTarget(renderTextureID)  

skybox绘制到了错误的render target。  

![[cd539e8f925e63e14b7f356f6ac90325_MD5.png]]![[b64e4602617fb3fe613dc8610d7cd0c8_MD5.png]]  

问题：部分pass缺少configure()重写的函数：  

```
//in ScriptableRenderPass class
public virtual void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)
{}

``` 

drawskybox未重载configure()：  

```
namespace UnityEngine.Rendering.Universal
{
    public class DrawSkyboxPass : ScriptableRenderPass
    {
        public DrawSkyboxPass(RenderPassEvent evt)
        {
            base.profilingSampler = new ProfilingSampler(nameof(DrawSkyboxPass));

            renderPassEvent = evt;
        }

        public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
        {
           context.DrawSkybox(renderingData.cameraData.camera);
        }
    }
}


``` 

像上三个测试均自主调用ConfigureTarget函数，判断是否切换回应有的render target。  

Test4完整函数：  

```
//错误对比用放置于Excute中
void RTRequestTest0_4(CommandBuffer commandBuffer, ScriptableRenderContext context, ref RenderingData renderingData)
{
  RenderTextureDescriptor cameraDescriptor = renderingData.cameraData.cameraTargetDescriptor;

  renderTextureID = Shader.PropertyToID("Request_ID");
  commandBuffer.GetTemporaryRT(renderTextureID, cameraDescriptor, FilterMode.Bilinear);
  // commandBuffer.GetTemporaryRT(renderTextureID, cameraDescriptor.width, cameraDescriptor.height, 16, FilterMode.Bilinear,
  // UnityEngine.Experimental.Rendering.GraphicsFormat.A2B10G10R10_UNormPack32, 1, false, RenderTextureMemoryless.None, false);//Full of parameters version

  commandBuffer.SetRenderTarget(renderTextureID);

  context.ExecuteCommandBuffer(commandBuffer);
  commandBuffer.Clear();
}

public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
{
  CommandBuffer commandBuffer = CommandBufferPool.Get();

  RTRequestTest0_4(commandBuffer, context, ref renderingData);

  //Do something at here...
  using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
  {
    commandBuffer.ClearRenderTarget(true, true, Color.blue, 1);
  }
  context.ExecuteCommandBuffer(commandBuffer);
  commandBuffer.Clear();
  CommandBufferPool.Release(commandBuffer);
}

``` 

# Draw Mesh Feature  

完整代码：[DrawMeshFeature.cs](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/DrawMesh/DrawMeshFeature.cs) [DrawMeshPass.cs](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/DrawMesh/Passes/DrawMeshPass.cs)  

## Draw Mesh Test  

```
//DrawMeshInstanced_Setup
for (int i = 0; i < matrices.Length; i++)
{
    Vector3 position = Random.onUnitSphere * 10f;
    matrices[i] = Matrix4x4.Translate(position);
    if (i < 1023)//超过1023个颜色后会报错
        colors[i] = new Vector4(position.x / 7, position.y / 7, position.z / 15, 1.0f);
}

```
//DrawMeshInstanced
MaterialPropertyBlock m_MatBlock = new MaterialPropertyBlock();
m_MatBlock.SetColor("_TestColor", Color.red);
commandBuffer.DrawMesh(passSetting.m_Mesh, Matrix4x4.identity, passSetting.UnlitInstancedMaterial, 0, 0, m_MatBlock);


``` 

![[50c20a370ce0e1f7ce2537f6af65ffc7_MD5.png]] 

完整函数：  

```
public DrawMeshPass(DrawMeshPassSetting m_DrawMeshPassSetting)
{
    this.passSetting = m_DrawMeshPassSetting;
    // Configures where the render pass should be injected.
    m_ProfilingSampler = new ProfilingSampler(m_ProfilerTag);
    this.renderPassEvent = m_DrawMeshPassSetting.passEvent;
}
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
{
    DrawMeshTest(context, ref renderingData);
}
void DrawMeshTest(ScriptableRenderContext context, ref RenderingData renderingData)
{
    CommandBuffer commandBuffer = CommandBufferPool.Get();
    using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
    {
        if (passSetting.UnlitInstancedMaterial != null)
        {
            MaterialPropertyBlock m_MatBlock = new MaterialPropertyBlock();
            m_MatBlock.SetColor("_TestColor", Color.red);
            commandBuffer.DrawMesh(passSetting.m_Mesh, Matrix4x4.identity, passSetting.UnlitInstancedMaterial, 0, 0, m_MatBlock);
        }
    }
    context.ExecuteCommandBuffer(commandBuffer);
    commandBuffer.Clear();
    CommandBufferPool.Release(commandBuffer);
}

``` 

## DrawMeshInstanced  

```
MaterialPropertyBlock m_MatBlock = new MaterialPropertyBlock();
m_MatBlock.SetVectorArray("_TestColor", colors);
//一个DrawMeshInstance指令绘制的Instance数量最多1023个
commandBuffer.DrawMeshInstanced(passSetting.m_Mesh, 0, passSetting.UnlitInstancedMaterial, 0, matrices, 1023, m_MatBlock);

``` 

![[5aafa79fad25e3d536c75d0e1a4a0af7_MD5.png]] 

![[273d966033646f2796416d8e99e2977f_MD5.png]]![[34e841d3ce92a8148da26f8d1bc7e1dc_MD5.png]]  

![[ef6d0f8242bdc0c7d9e969d166dba14b_MD5.png]] 

完整函数：  

```
public DrawMeshPass(DrawMeshPassSetting m_DrawMeshPassSetting)
{
    this.passSetting = m_DrawMeshPassSetting;
    // Configures where the render pass should be injected.
    m_ProfilingSampler = new ProfilingSampler(m_ProfilerTag);
    this.renderPassEvent = m_DrawMeshPassSetting.passEvent;
    //注意调用drawmeshinstance前先调用这个函数设置mesh instance
    DrawMeshInstanced_Setup();
}
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
{
    DrawMeshInstanced(context, ref renderingData);
}
void DrawMeshInstanced(ScriptableRenderContext context, ref RenderingData renderingData)
{
    CommandBuffer commandBuffer = CommandBufferPool.Get();
    using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
    {
        if (passSetting.UnlitInstancedMaterial != null)
        {
            MaterialPropertyBlock m_MatBlock = new MaterialPropertyBlock();

            m_MatBlock.SetVectorArray("_TestColor", colors);
            //一个DrawMeshInstance指令绘制的Instance数量最多1023个
            commandBuffer.DrawMeshInstanced(passSetting.m_Mesh, 0, passSetting.UnlitInstancedMaterial, 0, matrices, 1023, m_MatBlock);
        }
    }
    context.ExecuteCommandBuffer(commandBuffer);
    commandBuffer.Clear();
    CommandBufferPool.Release(commandBuffer);
}

``` 

### InstanceTest shader  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Shader/DrawMesh/Instance/InstanceTest.shader)  

其中实例化宏指令可在[UnityInstancing.hlsl](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Packages/com.unity.render-pipelines.core%4010.4.0/ShaderLibrary/UnityInstancing.hlsl)中查阅  

```
UNITY_INSTANCING_BUFFER_START(Props)  #UNITY_INSTANCING_CBUFFER_SCOPE_BEGIN(UnityInstancing_##buf) struct {
UNITY_DEFINE_INSTANCED_PROP(float4, _TestColor) #type var;
UNITY_DEFINE_INSTANCED_PROP(float, _Value)
UNITY_INSTANCING_BUFFER_END(Props) #arr##Array[UNITY_INSTANCED_ARRAY_SIZE]; UNITY_INSTANCING_CBUFFER_SCOPE_END
UNITY_VERTEX_INPUT_INSTANCE_ID  #...->uint instanceID;
UNITY_SETUP_INSTANCE_ID(input); #...->unity_InstanceID = inputInstanceID + unity_BaseInstanceID;
UNITY_TRANSFER_INSTANCE_ID(input, output); #output.instanceID = UNITY_GET_INSTANCE_ID(input)

``` 
#支持实例化指令：
#pragma multi_compile_instancing

``` ```
#没有M矩阵变换
#pragma instancing_options nomatrices

``` 

![[02d5726667b9b6c13f455de99a221be7_MD5.png]] 

```
#移动平台,可以打包到手机测试
#pragma instancing_options force_same_maxcount_for_gl

``` ```
#pragma instancing_options maxcount:512
#pragma instancing_options forcemaxcount:512
#MaxCount值最大被定义为500但是你可以绘制512个[翻译:https://mp.weixin.qq.com/s/qDPfrn2Vtw4qLUpiOBCa8g][原文:https://catlikecoding.com/unity/tutorials/rendering/part-19/]
#CBuffer只能存4096个16 btye(float)

``` 

在大多数平台上，Unity 通过以下方式自动计算实例化数据数组大小：将目标设备上的最大常量缓冲区大小除以包含所有每个实例的属性的结构的大小。通常您不必担心批次大小。但是，在某些平台（Vulkan、Xbox One 和 Switch）上，仍然需要固定的数组大小。您可以使用 maxcount 选项来为这些平台指定批次大小。在其他平台上，将完全忽略此选项。如果确实希望强制设定所有平台的批次大小，请使用 forcemaxcount（例如，当您知道自己只会通过 DrawMeshInstanced 发出包含 256 个实例化精灵的绘制时）。这两个选项的默认值为 500。  

引用自：[https://docs.unity3d.com/Manual/GPUInstancing.html](https://docs.unity3d.com/Manual/GPUInstancing.html)  

所以，maxcount指令会被忽略，控制批次的指令实际上只有forcemaxcount指令，要绘制1023个方块，我们需要两个批次绘制完成（一次512，一次511），如果我们减小forcemaxcount值，则需要更多批次完成。  

![[18c439e67a7312bdf4967884dbb0979a_MD5.png]]![[5d56a45e1381d383c3918b8a0353952b_MD5.png]]  

forcemaxcount:50：  

![[25833c7bafd2110d3e58cdd8b1bbf56a_MD5.png]]forcemaxcount:513：  

【Perdraw0报错】  

![[348d9c10816fba79d59fb697ce88a596_MD5.png]] 

包含PPConstantBuffers指定的缓冲区中的常量数的数组。每个数字指定着色器使用的常量缓冲区中包含的常量数。每个数量的常量都从其在pfirstConstant数组中指定的各自的偏移开始。每个常量必须是16个常量的倍数，在范围内[0..4096]。  

引用自：[https://docs.microsoft.com/en-us/windows/win32/api/d3d11_1/nf-d3d11_1-id3d11devicecontext1-vssetconstantbuffers1](https://docs.microsoft.com/en-us/windows/win32/api/d3d11_1/nf-d3d11_1-id3d11devicecontext1-vssetconstantbuffers1)  

```
#改变CBuffer结构达到扩容效果
#pragma instancing_options assumeuniformscaling

```
//in UnityInstancing.hlsl
//Put worldToObject array to a separate CB if UNITY_ASSUME_UNIFORM_SCALING is defined. Most of the time it will not be used.
#ifdef UNITY_ASSUME_UNIFORM_SCALING
	#define UNITY_WORLDTOOBJECTARRAY_CB 1
#else
	#define UNITY_WORLDTOOBJECTARRAY_CB 0
#endif
//...
UNITY_INSTANCING_BUFFER_START(PerDraw0)
//...
UNITY_INSTANCING_BUFFER_END(unity_Builtins0)

UNITY_INSTANCING_BUFFER_START(PerDraw1)
//...
UNITY_INSTANCING_BUFFER_END(unity_Builtins1)

UNITY_INSTANCING_BUFFER_START(PerDraw2)
//...
UNITY_INSTANCING_BUFFER_END(unity_Builtins2)


``` 

forcemaxcount:585，现在forcemaxcount最多可达到585  

![[8b7fea79ac6721ff04a7697791d9b985_MD5.png]]![[64d60dea540a407204407c57e6dd2ee1_MD5.png]]  

forcemaxcount:586  

【Perdraw2报错，超过容量】  

![[f4578774a481dd7877ab09ca910674b0_MD5.png]] 

## DrawLitMeshInstance  

![[544f6be4613cd1fc70376f34f87a5e3b_MD5.png]] 

完整函数：  

```
void DrawLitMeshInstanced(ScriptableRenderContext context, ref RenderingData renderingData)
{
  CommandBuffer commandBuffer = CommandBufferPool.Get();
using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
{
  if (passSetting.LitInstancedMaterial != null)
  {
    LightProbes.CalculateInterpolatedLightAndOcclusionProbes(passSetting.lightMapData.lightMapUVs.m_Position.ToArray(), lightProbesSH, OcclusionProbes);

    passSetting.LitInstancedMaterial.EnableKeyword("LIGHTMAP_ON");//可以自己测试LightMap
    //passSetting.LitInstancedMaterial.DisableKeyword("LIGHTMAP_ON");//可以自己测试LightProbe
    //passSetting.LitInstancedMaterial.EnableKeyword("LOD_FADE_CROSSFADE");
    MaterialPropertyBlock m_MatBlock = new MaterialPropertyBlock();
    m_MatBlock.SetTexture("unity_Lightmap", passSetting.lightMap);
    m_MatBlock.SetVectorArray("unity_LightmapST", lightMapST);
    m_MatBlock.SetVectorArray("unity_LODFade", LODFade);
    m_MatBlock.CopySHCoefficientArraysFrom(lightProbesSH);
    m_MatBlock.CopyProbeOcclusionArrayFrom(OcclusionProbes);

    commandBuffer.DrawMeshInstanced(passSetting.m_Mesh, 0, passSetting.LitInstancedMaterial, 0, matrices, 4, m_MatBlock);
  }
}
context.ExecuteCommandBuffer(commandBuffer);
commandBuffer.Clear();
CommandBufferPool.Release(commandBuffer);
}
public DrawMeshPass(DrawMeshPassSetting m_DrawMeshPassSetting)
{
    this.passSetting = m_DrawMeshPassSetting;
    // Configures where the render pass should be injected.
    m_ProfilingSampler = new ProfilingSampler(m_ProfilerTag);
  this.renderPassEvent = m_DrawMeshPassSetting.passEvent;

  // DrawMeshInstanced_Setup();
  DrawLitMeshInstanced_Setup();
}
public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
{
  DrawLitMeshInstanced(context, ref renderingData);
}
public void DrawLitMeshInstanced_Setup()
{
  int Count = passSetting.lightMapData.lightMapUVs.m_Position.Count;
  positions = new Vector4[Count];
  for (int i = 0; i < Count; i++)
  {
    matrices[i] = Matrix4x4.Translate(passSetting.lightMapData.lightMapUVs.m_Position[i]);
    lightMapST[i] = passSetting.lightMapData.lightMapUVs.m_LightMapUV[i];

    positions[i] = new Vector4(passSetting.lightMapData.lightMapUVs.m_Position[i].x, passSetting.lightMapData.lightMapUVs.m_Position[i].y, passSetting.lightMapData.lightMapUVs.m_Position[i].z, 1.0f);
  }
  lightProbesSH = new SphericalHarmonicsL2[Count];
  OcclusionProbes = new Vector4[Count];

  //有点为了使用而使用这个Option了，实际上我感觉是用Graphics.DrawInstance会好一点,毕竟能直接用Renderer https://my.oschina.net/u/4589313/blog/4447463
  LODFade = new Vector4[Count];
  for (int i = 0; i < Count; i++)
  	LODFade[i] = new Vector4(0.3f * i, 0.0f, 0.0f, 0.0f);
}

``` 

### LitInstanceTest shader  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Shader/DrawMesh/Instance/LitInstanceTest.shader)  

```
#pragma instancing_options nolightmap
//使用实例化时获取不了LightmapST[Scale/Offset].

``` 

![[abc55407a0059ffb7deb93ab4154241c_MD5.png]] 

```
passSetting.LitInstancedMaterial.DisableKeyword("LIGHTMAP_ON");//可以自己测试LightProbe

``` 

![[2db9b0a72a88e61d5a4a5ba3d7efe32c_MD5.png]] 

```
#pragma instancing_options nolightprobe
//使用实例化时获取不了Light Probe values(包括occlusion data).

``` 

![[706beb5e93881b96e4779be605a56ef4_MD5.png]] 

```
LODFade = new Vector4[Count];
for (int i = 0; i < Count; i++)
	LODFade[i] = new Vector4(0.3f * i, 0.0f, 0.0f, 0.0f);//第一个会是黑色
//...
m_MatBlock.SetVectorArray("unity_LODFade", LODFade);

``` ```
float4 LitPassFragment(Varyings input) : SV_TARGET
{
  UNITY_SETUP_INSTANCE_ID(input);
	return float4(unity_LODFade.xxx * ColorGI, 1.0);
}

``` 

![[9c4029b2b27984ceba860d2b9a88d436_MD5.png]] 

```
#pragma instancing_options nolodfade 
//默认lodfade

``` 

![[0008d1e41736a780ce3149cb25aac8e6_MD5.png]] 

## DrawMeshInstancedProcedural  

![[04cb2a8e331d4cf3b58591da35c9bd3e_MD5.png]] 

完整函数：  

```
void DrawMeshInstancedProcedural(ScriptableRenderContext context, ref RenderingData renderingData)
{
  //Procedural只能用全局
  CommandBuffer commandBuffer = CommandBufferPool.Get();
  using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
  {
    if (passSetting.LitInstancedProceduralMaterial != null)
    {
      passSetting.LitInstancedProceduralMaterial.EnableKeyword("LIGHTMAP_ON");

      MaterialPropertyBlock m_MatBlock = new MaterialPropertyBlock();

      m_MatBlock.SetVectorArray("_Position", positions);
      m_MatBlock.SetTexture("unity_Lightmap", passSetting.lightMap);

      m_MatBlock.SetVectorArray("_LightmapST", lightMapST);
      m_MatBlock.SetVectorArray("unity_LODFade", LODFade);

      // m_MatBlock.CopySHCoefficientArraysFrom(lightProbesSH);//LightProbe实现需要手动设置数组
      // m_MatBlock.CopyProbeOcclusionArrayFrom(OcclusionProbes);

      commandBuffer.DrawMeshInstancedProcedural(passSetting.m_Mesh, 0, passSetting.LitInstancedProceduralMaterial, 0, 4, m_MatBlock);
    }
  }
  context.ExecuteCommandBuffer(commandBuffer);
  commandBuffer.Clear();
  CommandBufferPool.Release(commandBuffer);
}

``` 

### LitInstanceProceduralTest shader  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Shader/DrawMesh/Instance/LitInstanceProceduralTest.shader)  

```
#pragma instancing_options procedural:ConfigureProcedural

void ConfigureProcedural()
{
   #if defined(UNITY_PROCEDURAL_INSTANCING_ENABLED)
    float3 position = _Position[unity_InstanceID].xyz;

    unity_ObjectToWorld = 0.0;
    unity_ObjectToWorld._m03_m13_m23_m33 = float4(position, 1.0);#平移

    unity_ObjectToWorld._m00_m11_m22 = float3(1, 1, 1);#缩放
  #endif
}

``` ```
//UnityInstancing.hlsl
#ifdef UNITY_PROCEDURAL_INSTANCING_ENABLED
  #ifndef UNITY_INSTANCING_PROCEDURAL_FUNC
  	#error "UNITY_INSTANCING_PROCEDURAL_FUNC must be defined."
  #else
  	void UNITY_INSTANCING_PROCEDURAL_FUNC(); // forward declaration of the procedural function
  	#define DEFAULT_UNITY_SETUP_INSTANCE_ID(input)      { UnitySetupInstanceID(UNITY_GET_INSTANCE_ID(input)); UNITY_INSTANCING_PROCEDURAL_FUNC();}
  #endif
#else
	//...
// in procedural mode we don't need cbuffer, and properties are not uniforms
#ifdef UNITY_PROCEDURAL_INSTANCING_ENABLED
  #define UNITY_INSTANCING_BUFFER_START(buf)
  #define UNITY_INSTANCING_BUFFER_END(arr)
  #define UNITY_DEFINE_INSTANCED_PROP(type, var)      static type var;
#else
  #define UNITY_INSTANCING_BUFFER_START(buf)          CBUFFER_START(buf)
  #define UNITY_INSTANCING_BUFFER_END(arr)            CBUFFER_END
  #define UNITY_DEFINE_INSTANCED_PROP(type, var)      type var;
#endif


``` 

更改 :  

```
unity_ObjectToWorld._m00_m11_m22 = float3(2, 1, 1)

``` 

![[2ff1d9b5d87bdec35ef050681b2ffe16_MD5.png]] 

# RenderInstancesIndirectPassFeature  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/DrawMesh/RenderInstancesIndirectPassFeature.cs)  

```
public override void Create()
{
  resolution = (uint)(resolution / 16) * 16;//限制为16倍数
	SetUpIndirectPass();//申请ompute buffer
	SetUpProceduralPass();
}

void SetUpIndirectPass()
{
  if (isActive)
  {
    if (positionBuffer == null)                                         //3*float(4 bytes)            
      positionBuffer = new ComputeBuffer(MaxResolution * MaxResolution, 3 * 4);
    if (bufferWithArgs_Indirect == null)
      bufferWithArgs_Indirect = new ComputeBuffer(5, 4, ComputeBufferType.IndirectArguments);
    else
      if (InstanceMesh != null)                           
        bufferWithArgs_Indirect.SetData(new uint[] { InstanceMesh.GetIndexCount(0), resolution * resolution, 0, 0, 0 });
	}

	m_IndirectPass = new RenderInstancesIndirectPass(GPUComputeShader, InstanceMaterial, InstanceMesh, positionBuffer, bufferWithArgs_Indirect, resolution);
	m_IndirectPass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
}

void SetUpProceduralPass()
{

  if (isActive)
  {
    if (particleBuffer == null)
      particleBuffer = new ComputeBuffer(MaxResolution * MaxResolution, 4 * 4);
    if (bufferWithArgs_Procedural == null)
      bufferWithArgs_Procedural = new ComputeBuffer(5, 4, ComputeBufferType.IndirectArguments);
    else
      if (InstanceMesh != null)                         
        bufferWithArgs_Procedural.SetData(new uint[] { resolution * resolution * 4, 5, 0, 0, 0 });
	}

	m_ProceduralPass = new RenderInstancesProceduralPass(GPUProceduralCS, ProceduralMaterial, InstanceMesh, particleBuffer, bufferWithArgs_Procedural, resolution);
	m_ProceduralPass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
}

``` 

## RenderInstancesIndirectPass  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/DrawMesh/Passes/RenderInstancesIndirectPass.cs)  

![[b8f1d59d1e54bb06ab262590f92544d8_MD5.png]] 

```
void UpdateFunctionOnGPU(ScriptableRenderContext context, ref RenderingData renderingData)
{
    float step = 2f / resolution;
    GPUComputeShader.SetInt(resolutionId, ((int)resolution));
    GPUComputeShader.SetFloat(stepId, step);

    #if UNITY_EDITOR
        float time = Application.isPlaying ? Time.time : Time.realtimeSinceStartup;
    	GPUComputeShader.SetFloat(timeId, time);
    #else
        float time = Time.time;
    GPUComputeShader.SetFloat(timeId, time);
    #endif

        GPUComputeShader.SetBuffer(0, positionId, positionBuffer);
    int groups = Mathf.CeilToInt(resolution / 8f);
    GPUComputeShader.Dispatch(0, groups, groups, 1);

    CommandBuffer commandBuffer = CommandBufferPool.Get();
    using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
    {
        MaterialPropertyBlock m_MatBlock = new MaterialPropertyBlock();
        m_MatBlock.SetBuffer(positionId, positionBuffer);
        m_MatBlock.SetFloat(stepId, step);

        commandBuffer.DrawMeshInstancedIndirect(instanceMesh, 0, instanceMaterial, 0, bufferWithArgs, 0, m_MatBlock);
    }

    context.ExecuteCommandBuffer(commandBuffer);
    commandBuffer.Clear();
    CommandBufferPool.Release(commandBuffer);
}

``` 

### GPUParticleCS.compute  

```
// Each #kernel tells which function to compile; you can have many kernels
#pragma kernel FunctionKernel
// Create a RenderTexture with enableRandomWrite flag and set it
// with cs.SetTexture
RWStructuredBuffer _Positions;
///计算出一个位置数组
//Parameter
uint _Resolution;
float _Time;
float _Step;
//Function
float2 GetUV(uint3 id)
{
    return(id.xy + 0.5) * _Step - 1.0;
}
#define PI 3.14159265358979323846
float3 Wave(float u, float v, float t)
{
    float3 p;
    p.x = u;
    p.y = sin(PI * (u + v + t));
    p.z = v;
    //position放大*5
    return p ;
}
void SetPosition(uint3 id, float3 position)
{
    //_Resolution=8*groups[Dispatch]=groupSizeX
    if (id.x < _Resolution && id.y < _Resolution)
        _Positions[id.x + id.y * _Resolution] = position;
}
[numthreads(8, 8, 1)]
void FunctionKernel(uint3 id : SV_DispatchThreadID)
{
    float2 uv = GetUV(id);
    // TODO: insert actual code here!
    SetPosition(id, Wave(uv.x, uv.y, _Time));
} 
``` 

### GPUParticleShader  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Shader/DrawMesh/GPGPU/GPUParticleShader.shader)  

compute shader计算的位置数组最终会传入这里计算。  

```
StructuredBuffer _Positions;//调用[GPUParticleCS]compute shader计算的数组
float _Step;
void ConfigureProcedural(Attributes input)
{
    //#if defined(UNITY_INSTANCING_ENABLED)
    float3 position = _Positions[input.instanceID];
    unity_ObjectToWorld = 0.0;
    unity_ObjectToWorld._m03_m13_m23_m33 = float4(position * 5, 1.0);
    unity_ObjectToWorld._m00_m11_m22 = _Step * 5;
    //#endif
} 
``` 

## RenderInstancesProceduralPass  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Scripts/RenderFeatures/DrawMesh/Passes/RenderInstancesProceduralPass.cs)  

```
commandBuffer.DrawProcedural(Matrix4x4.identity, instanceMaterial, 0, MeshTopology.Quads, ((int)resolution) * ((int)resolution) * 4, 1, m_MatBlock);

``` 

![[948f730cec598c415a71988998889da9_MD5.png]] 

<table class="table" style="width: 1000px;"><colgroup><col width="250"><col width="250"><col width="250"><col width="249"></colgroup><tbody class="table-inner"><tr class="tr" style="height: 33px;"><td class="td" data-col="0"><div class="td-content"><p id="191f588d025db55e90282c95db7d6f7e" data-lake-id="191f588d025db55e90282c95db7d6f7e"><text id="ua1d019d9">Points</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="11f0346d92d4a982d7ca0c19e09e9ffb" data-lake-id="11f0346d92d4a982d7ca0c19e09e9ffb"><text id="u73bddab4">Lines</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="2"><div class="td-content"><p id="99217a506454984cd391f6fadb3c22a9" data-lake-id="99217a506454984cd391f6fadb3c22a9"><text id="u170f131e">LineStrip</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="3" style="background-color: rgb(255, 255, 255); vertical-align: top;"><div class="td-content"><p id="a20d224c568e48b9d67847a2c66a8c01_p_0" data-lake-id="a20d224c568e48b9d67847a2c66a8c01_p_0"><text id="u25c69e92">Triangles</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr><tr class="tr" style="height: 33px;"><td class="td" data-col="0"><div class="td-content"><p id="ua0bfe8cf" data-lake-id="ua0bfe8cf" class="sr-rd-content-center"><img width="454.5" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635602511690-61f2903a-978d-4a4d-9664-bb40c2043447.png"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="f8b944ac94b24859dcd3c5113c9c146b" data-lake-id="f8b944ac94b24859dcd3c5113c9c146b" class="sr-rd-content-center"><img width="460.5" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635602632035-0340c5a1-6174-447f-a49c-efd43730e333.png"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="2"><div class="td-content"><p id="f6c436239374512893808781213ed431" data-lake-id="f6c436239374512893808781213ed431" class="sr-rd-content-center"><img width="452.5" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635602675913-bc788fc6-af63-43f8-9d33-0636eae62d79.png"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="3"><div class="td-content"><p id="u1d4fa6d3" data-lake-id="u1d4fa6d3" class="sr-rd-content-center"><img width="457.5" class="image image-preview image-hide" alt="image.png" draggable="true"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr></tbody></table>```
commandBuffer.DrawProceduralIndirect(Matrix4x4.identity, instanceMaterial, 0, MeshTopology.Quads, bufferWithArgs, 0, m_MatBlock);//Quad会报错,Quad不支持使用BufferArgs控制

``` 

![[b03ca19111b0168009eddcfe45789a75_MD5.png]]  

<table class="table" style="width: 1000px;"><colgroup><col width="250"><col width="250"><col width="250"><col width="249"></colgroup><tbody class="table-inner"><tr class="tr"><td class="td" data-col="0"><div class="td-content"><p id="u88c5fe2e" data-lake-id="u88c5fe2e"><text id="u558ab697">Points</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="ua21d47df" data-lake-id="ua21d47df"><text id="u78e68726">line</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="2"><div class="td-content"><p id="u2d1071ca" data-lake-id="u2d1071ca"><text id="u714d9d83">LineStrip</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="3"><div class="td-content"><p id="u38509ec9" data-lake-id="u38509ec9"><text id="u1ada2874">Triangles</text><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr><tr class="tr" style="height: 36px;"><td class="td" data-col="0"><div class="td-content"><p id="ufe4b0d54" data-lake-id="ufe4b0d54" class="sr-rd-content-center"><img width="498.5" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635608457866-7b448326-4724-4603-8da6-89cc64fde13d.png"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="1"><div class="td-content"><p id="u1058ea6b" data-lake-id="u1058ea6b" class="sr-rd-content-center"><img width="497.5" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635608465077-e30a7dc8-e1f8-40bc-8420-558efa6177a3.png"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="2"><div class="td-content"><p id="u804109dc" data-lake-id="u804109dc" class="sr-rd-content-center"><img width="493.5" class="image image-preview" alt="image.png" draggable="true" src="https://cdn.nlark.com/yuque/0/2021/png/22095277/1635608460136-2b1dfa10-3ad4-4a02-98d6-0d64584591b9.png?x-oss-process=image%2Fresize%2Cw_741%2Climit_0"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td><td class="td" data-col="3"><div class="td-content"><p id="ub9e1e1c6" data-lake-id="ub9e1e1c6" class="sr-rd-content-center"><img width="504" class="image image-preview image-hide" alt="image.png" draggable="true"><span class="viewer-b-filler" filler="block"><br></span></p></div><div class="td-break" contenteditable="false" style="user-select: text !important;-webkit-user-select: text !important;-webkit-touch-callout: text !important;"></div></td></tr></tbody></table>

完整函数：  

```
void SetUpProceduralPass()
{

    if (isActive)
    {
        if (particleBuffer == null)
            particleBuffer = new ComputeBuffer(MaxResolution * MaxResolution, 4 * 4);
        if (bufferWithArgs_Procedural == null)
            bufferWithArgs_Procedural = new ComputeBuffer(5, 4, ComputeBufferType.IndirectArguments);
        else
            if (InstanceMesh != null)                         
                bufferWithArgs_Procedural.SetData(new uint[] { resolution * resolution * 4, 5, 0, 0, 0 });
    }

    m_ProceduralPass = new RenderInstancesProceduralPass(GPUProceduralCS, ProceduralMaterial, InstanceMesh, particleBuffer, bufferWithArgs_Procedural, resolution);
    m_ProceduralPass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
}
void UpdateFunctionOnGPU(ScriptableRenderContext context, ref RenderingData renderingData)
{
    float step = 2f / resolution;
    GPUProceduralCS.SetInt(resolutionId, ((int)resolution));
    GPUProceduralCS.SetFloat(stepId, step);

    #if UNITY_EDITOR
        float time = Application.isPlaying ? Time.time : Time.realtimeSinceStartup;
    GPUProceduralCS.SetFloat(timeId, time);
    #else
        float time = Time.time;
    GPUProceduralCS.SetFloat(timeId, time);
    #endif

        GPUProceduralCS.SetBuffer(0, positionId, particleBuffer);
    int groups = Mathf.CeilToInt(resolution / 8f);
    GPUProceduralCS.Dispatch(0, groups, groups, 1);

    CommandBuffer commandBuffer = CommandBufferPool.Get();
    using (new ProfilingScope(commandBuffer, m_ProfilingSampler))
    {
        MaterialPropertyBlock m_MatBlock = new MaterialPropertyBlock();
        m_MatBlock.SetBuffer(positionId, particleBuffer);
        m_MatBlock.SetFloat(stepId, step);
        //Instance绘制
        //commandBuffer.DrawProcedural(Matrix4x4.identity, instanceMaterial, 0, MeshTopology.Points,((int)resolution) * ((int)resolution) * 4, 1, m_MatBlock);
        //commandBuffer.DrawProcedural(Matrix4x4.identity, instanceMaterial, 0, MeshTopology.Lines, ((int)resolution) * ((int)resolution) * 4, 1, m_MatBlock);
        //commandBuffer.DrawProcedural(Matrix4x4.identity, instanceMaterial, 0, MeshTopology.LineStrip, ((int)resolution) * ((int)resolution)* 4, 1, m_MatBlock);
        //commandBuffer.DrawProcedural(Matrix4x4.identity, instanceMaterial, 0, MeshTopology.Triangles, ((int)resolution) * ((int)resolution)* 4, 1, m_MatBlock);
        //commandBuffer.DrawProcedural(Matrix4x4.identity, instanceMaterial, 0, MeshTopology.Quads, ((int)resolution) * ((int)resolution) * 4, 1, m_MatBlock);

        commandBuffer.DrawProceduralIndirect(Matrix4x4.identity, instanceMaterial, 0, MeshTopology.Lines, bufferWithArgs, 0, m_MatBlock);//Quad会报错,Quad不支持使用BufferArgs控制
    }

    context.ExecuteCommandBuffer(commandBuffer);
    commandBuffer.Clear();
    CommandBufferPool.Release(commandBuffer);
}

``` 

### GPUParticleProceduralCS.compute  

```
// Each #kernel tells which function to compile; you can have many kernels
#pragma kernel FunctionKernel


struct ParticleData
{
    float4 position;
};

RWStructuredBuffer _Positions;

uint _Resolution;
float _Time;
float _Step;

float2 GetUV(uint3 id)
{
    float2 uv;
    //_Step:2f/_Resolution
    uv.x = (id.x) * _Step * 10 ;
    uv.y = (id.y) * _Step * 10;
    //这里的作用是将id[0-DispatchIndex]映射到(-1,1)
    //所以映射到(-1,1)之后每个点的距离为2/_Resolution即_Step的大小
    return uv;
}

[numthreads(8, 8, 1)]
void FunctionKernel(uint3 id : SV_DispatchThreadID)
{
    ParticleData m_ParticleData = (ParticleData)0;
    
    //_Resolution=8*ThreadGroups[Dispatch了多少个线程组]=groupSizeX
    uint DispatchIndex = id.x + (id.y * _Resolution);
    float2 uv = GetUV(id);

    // simple version
    // float3 basicPosition = float3(DispatchIndex, 0, 0);
    // m_ParticleData.position = float4(basicPosition, 1.0);

    // complex version
    float3 basicPosition = float3(id.x * 1.5, id.x * 0.1 + sin(id.x + _Time), id.y * 1.5);
    m_ParticleData.position = float4(basicPosition, 1.0);

    _Positions[DispatchIndex] = m_ParticleData;
} 
``` ```
// simple version
float3 basicPosition = float3(DispatchIndex, 0, 0);
m_ParticleData.position = float4(basicPosition, 1.0);

``` 

![[e21187ddfd721259189c0a665f7cbbd4_MD5.png]] 

### GPUParticleProceduralShader  

完整代码：[here](https://github.com/logic-three-body/CommandBufferDemo/blob/main/Assets/Shader/DrawMesh/GPGPU/GPUParticleProceduralShader.shader)  

```
Varyings LitPassVertex(Attributes input)
{
    Varyings output = (Varyings)0;
    float2 texcoord;
    texcoord.x = float(((input.vid + 1) & 2) >> 1);
    texcoord.y = float((input.vid & 2) >> 1);

    float3 objectSpace = float3(texcoord.xy - 0.5, 0.0);

    float3 worldSpace = _Positions[input.vid / 4].position.xyz;

    unity_ObjectToWorld = 0.0;
    //
    unity_ObjectToWorld._m03_m13_m23_m33 = float4(worldSpace, 1.0);
    unity_ObjectToWorld._m03_m13_m23_m33 = float4(worldSpace + float3(input.instanceID.x, input.instanceID.x, 0.0), 1.0);
    unity_ObjectToWorld._m00_m11_m22 = 1.0;

    float4 pivotWS = mul(UNITY_MATRIX_M, float4(0, 0, 0, 1));
    float4 pivotVS = mul(UNITY_MATRIX_V, pivotWS);
    float4 positionVS = pivotVS + float4(objectSpace.xy, 0, 1);

    output.positionCS = TransformWViewToHClip(positionVS.xyz);

    //output.positionCS = TransformObjectToHClip(objectSpace.xyz);

    output.positionWS = (objectSpace.xyz);
    output.viewWS = GetWorldSpaceViewDir(output.positionWS);

    output.texcoord = texcoord;
    return output;
}

``` 

# 应用  

## Render Features的模板测试  

Learn From: [here](https://www.youtube.com/watch?v=eiyJh242FLI)  

![[6b5310762d5deee08080354486148e58_MD5.gif]]  

### 设置layer  

![[9634dce2a9d1863fecdbf1fee5326be7_MD5.png]]![[bdc060e5f3b75fb914adfe5b1e614cf2_MD5.png]]  

### 设置render feature  

最开始的两个挡板即mask1和mask2模板测试都会失败，所以它们不会被渲染，渲染失败后，他们的模板值会写入缓冲区（stencil[Mask1]=1,stencil[Mask2]=2）,之后两个模型片元对应的模板值就会和缓冲区里的相等，就会被渲染。  

![[61b0892852e47caa090fb049da529132_MD5.png]]![[feb19da4325abe6f9e6bca96c4bb4e88_MD5.png]]  

### 渲染过程  

![[bc472386da9d2f21f601a5a4ca2e7928_MD5.png]]![[d9c91321f7359c48bc561c12a455ce1b_MD5.png]]  

## 传送门效果  

![[1999807f773d94dee678e644772d4c4f_MD5.gif]]  

# Refference&RoadMap:  

[Portal Stencil Shader - VRChat/Unity Advanced Tutorial](https://www.youtube.com/watch?v=SySnHIhiVgM) : [unitypack](https://www.dropbox.com/s/e25f2du9z5mv7hv/stencil%20package.unitypackage?dl=0)  

[Stencil-based Portals](https://danielilett.com/2019-12-14-tut4-2-portal-rendering/) ： [github](https://github.com/daniel-ilett/shaders-portal)  

[Unity 3D - Portal Room Cubes via Stencil Buffer](https://www.youtube.com/watch?v=5DKIP9N-OB4) : [Github](https://github.com/noisecrime/Unity-StencilPortalRoomCube)  

[Coding Adventure: Portals](https://www.youtube.com/watch?v=cWpFZbjtSQg&list=PLFt_AvWsXl0ehjAfLFsp1PGaatzAwo0uK&index=6) : [Github](https://github.com/SebLague/Portals/tree/master)  

[Unity URP et les Render Features ! : Reprenez le contrôle de vos graphismes](https://www.youtube.com/watch?v=eiyJh242FLI)