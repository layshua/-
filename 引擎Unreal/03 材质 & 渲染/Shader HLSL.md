[# [Ue5.1]Shader入门教程(HLSL)](https://www.bilibili.com/video/BV1hg411W7hn?p=1&vd_source=9d1c0e05a6ea12167d6e82752c7bc22a)
[(16条消息) HLSL内置函数，及HLSL与GLSL的对应函数_Eloudy的博客-CSDN博客_hlsl和glsl](https://blog.csdn.net/eloudy/article/details/71258367)
[虚幻5.1内置函数关键词整理by乃冬橘](https://www.bilibili.com/read/cv20247033?spm_id_from=333.999.0.)
# 内置函数关键词
## 数学运算
```cpp
max
min
fmod //取余
round //四舍五入
pow(a,b) //指数幂
sqrt //平方根
rsqrt //平方根的倒数
degrees //弧度转换成角度
redians //角度转换成弧度
noise //噪声函数，传入二维坐标范围[0,1]随机值
log //幂指对函数
cos sin tan atan atan2 //三角函数
sinh cosh tanh //双曲线函数
ceil
floor
smoothstep
clamp
saturate
step
lerp
frac
length //向量到原点的距离
distance //两点距离
normalize
mul
dot
sign //返回x的正负，x<0返回-1，x=0返回0，x>0返回1
any //传入矢量，任何一个分量为非0返回true，否则false
all //传入矢量，所有分量均为非0返回true，否则false



```
## 纹理采样
```c
float4 h = Texture2DSample(tex,texSampler,uv);
return h;

float4 h = Texture3DSample(tex,texSampler,uv);
return h;

float4 cube = TextureCubeSample(tex,texSampler,uv);
return cube;
```
![[Pasted image 20230103112156.png|300]]

## 基础参数
```cpp
//调用方法：
float3 normal = Parameters.WorldNormal;
return normal;
```

```cpp
Parameters.VertexColor   //获取方向颜色
Parameters.WorldTangent  //获取世界切线
Parameters.ReflectionVector  //获取反射向量
Parameters.CameraVector  //获取相机向量
Parameters.LightVector  //获取灯光向量(用于灯光函数)
Parameters.SvPosition   //获取全局位置
Parameters.ScreenPosition  //获取场景位置
Parameters.WorldNormal //获取世界法线
Parameters.ViewBufferUV  //
Parameters.AbsoluteWorldPosition //获取世界绝对位置
Parameters.WorldPosition_CamRelative  //世界位置 相机相对位置
Parameters.WorldPosition_NoOffsets_CamRelative  //世界相机(没有编译)
Parameters.LightingPositionOffset //灯光位置偏移
```
## SceneTexture
```cpp
float4 h = SceneTextureLookup(GetDefaultSceneTextureUV(Parameters, 1), 1, false);
return h;
```

```cpp

对应参数数字(第二个数字);
SceneColor 0    //场景颜色
SceneDepth 1    //场景深度
DiffuseColor 2     //漫反射颜色
SpecularColor 3   //高光颜色
SubsurfaceColor 4   //次表面颜色
BaseColor 5   //底色(针对光照)
Specular 6  //高光(针对光照)
Metallic 7   //金属
WorldNormal 8   //场景法线
SeparateTranslucency 9   //分隔半透明
Opacity 10  //不透明度
Roughness 11   //粗糙度
MaterialAO 12   //材质AO
CustomDepth 13  // 自定义深度
PostProcessInput0 14  //后期处理输入0
PostProcessInput1 15  //后期处理输入1
PostProcessInput2 16  //后期处理输入2
PostProcessInput3 17  //后期处理输入3
PostProcessInput4 18  ////后期处理输入4
PostProcessInput5 19 //后期处理输入5
PostProcessInput6 20  //后期处理输入6
DecalMask 21   //贴花遮罩
ShadingModelColor 22   //着色器颜色
ShadingModelID 23  // 着色器ID
AmbientOcclusion 24   //环境光遮蔽
CustomStencil 25  //自定义模板值
StoredBaseColor 26  //底色
StoredSpecular 27  //高光
Velocity 28  //速度
WorldTangent 29  //场景切线
Anisotropy 30//各向异性

```

## 空间变换
```cpp
ClipToWorld      //裁剪空间到世界空间
ClipToView      //裁剪空间到摄像机空间
ClipToTranslatedWorld     //裁剪空间到变换空间
ClipToRelativeWorld     //裁剪空间到相对空间
CameraViewToTranslatedWorld     //摄像机空间到变换空间
TranslatedWorldCameraOrigin     //变换空间相机原点
TranslatedWorldToView     //变换空间到摄像机空间
TranslatedWorldToClip     //变换空间到裁剪空间
TranslatedWorldToCameraView     //变换空间到摄像机空间
ViewToClip     //摄像机空间到裁剪空间
ViewToTranslatedWorld     //摄像机空间到变换空间
ViewToClipNoAA     //摄像机空间到没有A的裁剪空间
ViewForward     //摄像机向前空间
ViewUp         //摄像机向上空间
ViewRight     //摄像机向右空间
ViewTilePosition     //摄像机空间平铺位置
HMDViewNoRollUp  //无向上的HMD视口
HMDViewNoRollRight //无右的HMD视口
InvDeviceZToWorldZTransform
NumSceneColorMSAASamples
RelativeWorldToClip //相对空间到裁剪空间
RelativeWorldViewOrigin //相对空间视口原点
RelativeWorldCameraOrigin //相对空间相机原点
RelativePreViewTranslation//相对空间预览变换
WorldToClip //世界空间到裁剪空间
WorldViewOrigin //世界视口原点
WorldCameraOrigin //世界相机原点
MatrixTilePosition
SVPositionToTranslatedWorld //SV位置到变换空间
ScreenToWorld //屏幕空间到世界空间
ScreenToTranslatedWorld //屏幕空间到变换空间

```

## View调用
```cpp
//调用方法：
float time = View.RealTime;
return time;
```

```cpp
GlobalClippingPlane  //全局裁剪屏幕
ViewWideAngles  //查看广角
PrevFieldOfViewWideAngles //上一个视野广角
ViewSizeAndInvSize //视图大小和inv大小
ViewRectMinAndSize //视图矩形最小值和大小
LightProbeSizeRatioAndInvSizeRatio //灯光测试
 BufferSizeAndInvSize //缓冲区大小
 BufferBilinearUVMinMax //缓冲线UV最大最小
 ScreenToViewSpace //屏幕到视图空间
BufferToSceneTextureScale //缓冲区到场景文本比例
ResolutionFractionAndInv //分辨率分数和库存
NumSceneColorMSAASamples //NumSceneColorMSAA示例
SeparateWaterMainDirLightLuminance //分隔水主指示灯亮度
PreExposure //预暴露
OneOverPreExposure //一次过度预暴露
PrevFrameGameTime //上一帧游戏时间
 PrevFrameRealTime //上一帧实时
WorldCameraMovementSinceLastFrame //自上一帧以来的世界相机移动
CullingSign //消隐符号
GameTime //游戏时间
RealTime //时间
DeltaTime //增量时间
MaterialTextureMipBias //材料纹理MipBias
MaterialTextureDerivativeMultiply //材料纹理衍生乘法
Random //随机值
FrameNumber //帧数
StateFrameIndexMod8 //帧数状态索引mod8
StateFrameIndex //帧数状态索引
DebugViewModeMask //调试视图模式掩码
 TemporalAAParams //临时AA参数
 CircleDOFParams //圆形DOF参数
DepthOfFieldSensorWidth //景深传感器宽度
DepthOfFieldFocalDistance //景深焦距深度
DepthOfFieldScale //字段比例深度
DepthOfFieldFocalLength //字段焦点长度深度
DepthOfFieldFocalRegion //字段焦点区域深度
DepthOfFieldNearTransitionRegion //字段最近转换区域的深度
DepthOfFieldFarTransitionRegion //字段深度转换区域
MotionBlurNormalizedToPixel //单一化bulr到像素
GeneralPurposeTweak //一般目的调整
GeneralPurposeTweak2 //一般目的调整2
DecalDepthBias //贴花深度偏移
 IndirectLightingColorScale //间接照明颜色比例
PrecomputedIndirectLightingColorScale //预计算间接照明色阶
PrecomputedIndirectSpecularColorScale //预计算间接规格颜色比例
AtmosphereLightDirection//大气照明方向
AtmosphereLightIlluminanceOnGroundPostTransmittance//大气光照度地面后透射率
AtmosphereLightIlluminanceOuterSpace//大气照明室外空间
AtmosphereLightDiscLuminance//大气发光二极管亮度
AtmosphereLightDiscCosHalfApexAngle_PPTrans//大气光光盘
SkyViewLutSizeAndInvSize //天空视图
SkyCameraTranslatedWorldOrigin //天空相机变换世界原点
SkyPlanetTranslatedWorldCenterAndViewHeight //开始深度Km
SkyViewLutReferential //透视体积深度分辨率
SkyAtmosphereSkyLuminanceFactor //透视体积深度切片长度Km
SkyAtmospherePresentInScene //透视体积深度切片长度KmInv
SkyAtmosphereHeightFogContribution //天空大气应用相机空中透视体积
SkyAtmosphereBottomRadiusKm //正常曲率到粗糙度刻度偏差
SkyAtmosphereTopRadiusKm //渲染反射捕获遮罩
```
## 其他
```cpp
SkyAtmosphereCameraAerialPerspectiveVolumeSizeAndInvSize
SkyAtmosphereAerialPerspectiveStartDepthKm
SkyAtmosphereCameraAerialPerspectiveVolumeDepthResolution
SkyAtmosphereCameraAerialPerspectiveVolumeDepthResolutionInv
SkyAtmosphereCameraAerialPerspectiveVolumeDepthSliceLengthKm
SkyAtmosphereCameraAerialPerspectiveVolumeDepthSliceLengthKmInv
SkyAtmosphereApplyCameraAerialPerspectiveVolume
NormalCurvatureToRoughnessScaleBias
RenderingReflectionCaptureMask
RealTimeReflectionCapture
RealTimeReflectionCapturePreExposure
AmbientCubemapTint
AmbientCubemapIntensity
SkyLightApplyPrecomputedBentNormalShadowingFlag
SkyLightAffectReflectionFlag
SkyLightAffectGlobalIlluminationFlag
FLinearColor, SkyLightColor
MobilePreviewMode
HMDEyePaddingOffset
ShowDecalsMask
uDistanceFieldAOSpecularOcclusionMode
IndirectCapsuleSelfShadowingIntensity
ReflectionEnvironmentRoughnessMixingScaleBiasAndLargestWeight
StereoPassIndex
GlobalVolumeCenterAndExtent
GlobalVolumeWorldToUVAddAndMul
GlobalDistanceFieldMipWorldToUVScale
GlobalDistanceFieldMipWorldToUVBias
GlobalDistanceFieldMipFactor
GlobalDistanceFieldMipTransition
GlobalDistanceFieldClipmapSizeInPages
GlobalDistanceFieldInvPageAtlasSize
GlobalDistanceFieldInvCoverageAtlasSize
GlobalVolumeDimension
GlobalVolumeTexelSize
MaxGlobalDFAOConeDistance
uNumGlobalSDFClipmaps
CoveredExpandSurfaceScale
NotCoveredExpandSurfaceScale
NotCoveredMinStepScale
DitheredTransparencyStepThreshold
DitheredTransparencyTraceThreshold
FIntPo CursorPosition
bCheckerboardSubsurfaceProfileRendering
VolumetricFogInvGridSize
VolumetricFogGridZParams
VolumetricFogSVPosToVolumeUV
VolumetricFogMaxDistance
VolumetricLightmapWorldToUVScale
VolumetricLightmapWorldToUVAdd
VolumetricLightmapIndirectionTextureSize
VolumetricLightmapBrickSize
VolumetricLightmapBrickTexelSize
IndirectLightingCacheShowFlag
EyeToPixelSpreadAngle
GlobalVirtualTextureMipBias
uVirtualTextureFeedbackShift
uVirtualTextureFeedbackMask
uVirtualTextureFeedbackStride
uVirtualTextureFeedbackJitterOffset
uVirtualTextureFeedbackSampleOffset
 RuntimeVirtualTextureMipLevel
FVector2f, RuntimeVirtualTexturePackHeight
 RuntimeVirtualTextureDebugParams
OverrideLandscapeLOD
FarShadowStaticMeshLODBias
MinRoughness
 HairRenderInfo
uEnableSkyLight
uHairRenderInfoBits
uHairComponents
bSubsurfacePostprocessEnabled
SSProfilesTextureSizeAndInvSize
SSProfilesPreIntegratedTextureSizeAndInvSize
PhysicsFieldClipmapCenter
PhysicsFieldClipmapDistance
PhysicsFieldClipmapResolution
PhysicsFieldClipmapExponent
PhysicsFieldClipmapCount
PhysicsFieldTargetCount
PhysicsFieldTargets
uInstanceSceneDataSOAStride
uGPUSceneViewId
ViewResolutionFraction
SubSurfaceColorAsTransmittanceAtDistanceInMeters 
```

# 步骤
VScode新建HLSL文件，后缀名为.ush。保存到项目目录下，新建一个Shader文件夹。

在Custom节点里，包含ush文件路径
![[Pasted image 20230103105044.png]]
ush文件切换语言模式为hlsl：
![[Pasted image 20230103113224.png]]
在自定义节点中设置输入名称
![[Pasted image 20230103105439.png]]

# 用结构体创建局部函数
```c
struct function
{
    float add(float a, float b)
    {
        float c=a+b;
        return c;
    }
};

function f;
float y = f.add(a,b);
return y;
```

# 创建全局函数
引擎目录下打开：
![[Pasted image 20230103113021.png]]
尝试写一个全局函数：
![[Pasted image 20230103113144.png]]
这样就可以在自己的shader中调用：
![[Pasted image 20230103113420.png]]

# shadertoy转换
[番外篇-简单将shadertoy转换到虚幻上_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hg411W7hn?p=38&spm_id_from=pageDriver&vd_source=9d1c0e05a6ea12167d6e82752c7bc22a)