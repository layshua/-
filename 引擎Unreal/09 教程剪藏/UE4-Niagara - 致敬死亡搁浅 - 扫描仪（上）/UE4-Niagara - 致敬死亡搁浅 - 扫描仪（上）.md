## 概述

话不多说，开冲！（多图杀 cat，建议在 WiFi 下观看）

![](1677315621012.png)

## **目录**

*   正交相机**重建粒子位置**
*   **网格发射器**模块
*   Niagara modules **调用另一个模块属性**（Current modules attribute specifying as another modules parameter.）
*   蓝图**传递参数**到 **Niagara System**
*   过程化的 Niagara 操作

**制作思路**：scenecapture 可以在目标头顶拍摄到一张图片，这张图片包含了场景 XY 两个轴向上的位置排列，但是缺失高度轴 Z。**放置网格状的粒子于场景中、缩放到相机捕获范围相同的大小即可还原整个 XY Plane。**

**对于 Z 轴可以使用 capture 中的 “scene depth” 捕获。随后赋予粒子 UV 属性来采样捕获内容的贴图 depth 值，最终每个粒子得到对应场景位置的 Z 轴位置。加在之前的 XY Plane 上就还原了场景。**

![](1677315621105.png)

XY Plane：先生成一个 size 为 1 单位的网格平面粒子，把 position 赋予 UV，缩放粒子网格大小适配场景大小，完成 XY 轴向的还原。 Z Axis：使用之前的 UV 属性采样 scene depth，加在 z 轴分量上完成 Z 轴还原。

**资源准备**：创建一个新的 render target 来储存 scene capture 2d 捕获的场景深度。在第三人称人物蓝图中添加 scenecapture 2d，更改相机模式为 orth 正交模式，捕获内容的 A 通道改为 scene depth。

![](1677315621183.png)

![](1677315621258.png)

建立一个 xy 平面的网格粒子，并且使用 position 赋予 particle uv 属性。同时把 uv 作为颜色显出来 debug 一下信息是否正确。

![](1677315621316.png)

Tip&Note：创建新的变量和修改，都可以点击 + 号在跳出来的搜索框内找到 "Set new or existing parameter directly" 来修改 / 创建你的变量。这里的 UV 属性就是在这个 module 中创建的。

## **正交相机重建粒子位置**

在人物上方放置一个正交 **Scenecapture 2D 捕获场景深度作为粒子的 Z 轴数据**，网格粒子采样 RT 贴图信息**添加 Z 轴向分量到 Position.z** 完成映射。

网格粒子采样贴图时有了现成的 xy plane 排列间隔了，只需要将标准大小（0-1）的 xy plane 向量放置到坐标正中心再乘以 scenecapture 捕获区域的 width size 就可以还原平面。

（因为摄像机拍摄是在中心拍摄的出来 RT 的，采样贴图的 UV 还原出来是再右上角区间）  
Tip：正常采样出来的是方向不对，需要旋转一下贴图，所以重新修正 UV 方向选择 90°

![](1677315621388.png)

XY Plane

```
float2 xyPlane = float2(1.0-UV.y,UV.x);
xyPlane -=0.5;
xyPlane *= WIDTH;

OV = float3(xyPlane.x, xyPlane.y, 0.0);
```

Z Axis

```
float3 projectorDir = normalize(PRODIR);

projectorDir *= (DEPTH + OFFSET);

OV = projectorDir;
```

![](1677315621547.png)

ok 这样看起来好像我们已经成功了，下面把粒子放入蓝图中。

![](1677315621603.png)

Running！

![](1677315621660.png)

现在和我们脑海中想的是不一样的。我想问题应该是出现在 grid location 上，我选择自己建立一个自己的发射模块，但是这需要用到一些**两个 modules 互相绑定参数的技巧**。

## **网格发射器**模块

![](1677315621915.png)

**Niagara modules 调用另一个模块属性**

![](1677315621953.png)

![](1677315622064.png)

```
float xPos = fmod((float)ID,XC) / float(XC - 1.0) * XS;
float yPos = floor(float(ID) / YC) / float(YC - 1.0) * YS;
OP = float3(xPos,yPos,0);
```

![](1677315622141.png)

运行结果

![](1677315622250.png)

现在完成了大体的框架，我们继续做一些设置来优化

## **蓝图传递参数到 Niagara System**

在 system 中创建一个 float parameter 连接到 Orth width 端口，从蓝图中来赋值这个 parameter 保证我们的粒子网格大小与 scene capture 捕获的大小一样大。

![](1677315622348.png)

![](1677315622415.png)

## **过程化的 Niagara 操作**

现在完成了大体的构架，剩下的就是优化这个粒子的动态效果。这个时候不建议创建太多的 modules，所以可以使用 "Scratch Pad Module" 来做过程化的一些连接。

1.  粒子扩散出现
2.  粒子在坡度不同处颜色与图形不同

**1. 粒子扩散出现**

![](1677315622518.png)

这里为了让粒子不直接发射在物体表面被遮挡住，在之前的 "NM_OrthCam___Project" 上 depth offset 调整到了 - 5，把粒子 z 轴高度整体上移一点。

蓝图中使用 "timeline" 添加一个变化的增量值，作为圆形的半径，使用 "Radius" 与 position 做 distance 运算即可算出圆球范围。

稍后将 timeline 的数据赋予粒子系统的用户自定义参数中。

![](1677315622811.png)

添加一个速写板制作一个修改粒子 Alpha 信息的功能，使只有再圆形半径内的粒子显示，并且开始增加生命。

![](1677315622917.png)

Tip&Note：这里为了偷懒少截图，没有从蓝图里面传递角色按下扫描按钮时的一个固定位置，直接用了动态的 Engine.Owner.Positon，要制作正确结果的请自己多动手加一个参数链接进来！同时还要把 Radius 的最大值也 bind 进来删除不会出现的那些粒子，但是这里因为制作截图就会太多我都选择了省略。 这里也可以只用一个 custom hlsl，但是为了方便基础弱的我使用了两个来分开这个功能，写的时候写一个里面就好了。

Dist Start

```
PointPos = float3(PointPos.x, PointPos.y, 0.0f);
StartPos = float3(StartPos.x, StartPos.y, 0.0f);

float dist = length(PointPos - StartPos);

float alpha = 0.0;
if(dist <= DynamicRadius){
alpha = 1.0;
}

OF = alpha;
```

ReAge

```
if(Start == 1){
OAGE = AGE;
}else{
OAGE = 0.0f;
}
```

![](1677315623071.png)

**2. 粒子在坡度不同处颜色与图形不同**

粒子动态制作完成后就是材质部分，在 substance designer 中创建基础图形放置在一张贴图中，使用 subUVIndex 来做选择最后出现的图案。

![](1677315623129.png)

创建一个简单的材质链接，赋予粒子材质。

![](1677315623209.png)

使用之前抓取的 scene Depth 计算一下 slope（这里我也不确切的知道死亡搁浅中如何计算，随意计算了一个用 depth 采样周围的 depth 多次计算出周围的高度差）。

![](1677315623269.png)

```
float4 oriTex = Texture2DSample(Tex, GetMaterialSharedSampler(TexSampler,View.MaterialTextureBilinearClampedSampler),UV.xy);

float4 anthorTex;
float d, diff, c, s, ramp;
float2 offset;

for(int i=0; i<NUM;i++){
    ramp = float(i)/(NUM-1) * 6.28;
    c = cos(ramp);
    s = sin(ramp);
    offset = float2(c, s) * Radius;
    anthorTex = Texture2DSample(Tex, GetMaterialSharedSampler(TexSampler,View.MaterialTextureBilinearClampedSampler), UV.xy + offset);
    diff = oriTex.a - anthorTex.a;
    d = max(d, diff);
}

return saturate(d/Scale);
```

新建一个 "RT_Slope"，绘制坡度材质信息到 RT 中，传递至粒子系统读取坡度信息。

![](1677315623317.png)

![](1677315623374.png)

在粒子系统中新机哪一个速写板，使用 UV 属性读取 slope rt，根据颜色的范围做选择来切换颜色与图案。

![](1677315623424.png)

这里为什么不用 hlsl 非常方便的写一个呢... 因为我使用 if(){}else if(){} 这样的结构失败了，使用 if(){}else{ if(){}} 这样的书写也失败了... 最后不得不节点连接。

![](1677315623472.png)

![](1677315623646.png)

现在完成了基础部分的粒子设置，后续的后处理部分推荐大家看黑哥的文章。如果有空我应该也会在加一篇 ue 的实现。（估计没太大必要）

[https://mp.weixin.qq.com/s/uohr3b7Ah8hInqvVGX3QFA](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/uohr3b7Ah8hInqvVGX3QFA)

The end.

这篇文章拖太久了，暂时先发布了，后续空了会写后篇。