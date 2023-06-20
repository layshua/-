# Unity渲染顺序
**在 Unity 中，渲染顺序是根据以下参数依次按条件先后顺序进行排序渲染处理**。先按上层条件排序，如上层条件相同，则进入下层条件牌序，最终分出先后

**Camera Depth > Render Queue 大于还是小于等于 2500 > Sorting Layer > Order in Layer > Render Queue > Camera order algorithm

1.  Camera Depth：值越小越优先渲染，优先渲染可能会被覆盖
2.  Render Queue <= 2500：
    1.  按照 Sorting Layer / Order in Layer 设置的值，越小越优先。若无此属性，等同于 Sorting Layer = default，Order in Layer = 0 参与排序
    3.  Render Queue 越小越优先
    4.  Render Queue 相等，由**近到远排序**
3. Render Queue >= 2500
    1.  按照 Sorting Layer / Order in Layer 设置的值，越小越优先
    2.  无此属性，等同于 Sorting Layer = default，Order in Layer = 0 参与排序
    3.  Render Queue 越小越优先
    4.  Render Queue 相等，由**远到近排序**

## Camera Depth

![[c41cf813488837339d527b343fd9840f_MD5.jpg]]

*   通过无法搭配 Clear Flags: Don't Clear，因为不会清除 depth buffer（z-buffer）

## Material Type

[https://docs.unity3d.com/2018.4/Documentation/Manual/SL-SubShaderTags.html](https://docs.unity3d.com/2018.4/Documentation/Manual/SL-SubShaderTags.html)

![[ba852a52d51849f9158cf443cded5061_MD5.jpg]]

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><td>2500 是关键值，它是透明和不透明的分界点，因此我们考虑层级的时候要注意：renderqueue &gt; 2500 的物体绝对会在 renderqueue &lt;= 2500 的物体前面，即渲染时 renderqueue 大的会挡住 renderqueue 小的。</td></tr></tbody></table>

*   先渲染不透明物体（opaque），再渲染透明物体（transparent）
*   根据 Render Queue 来决定，数值小于等于 2500 为不透明物体，数值大于 2500 为透明物体
*   **skyboxes 渲染发生在 opaque 和 transparent 之间**

## Sorting Layer

![[29f5c88b31d2d14474e6a32292a876a7_MD5.jpg]]

![[8e7a22d963d86de96cb7d3b3f06399b9_MD5.jpg]]

## Order in Layer

![[ca37a2ad867ee77bcfa77d3da7caae06_MD5.jpg]]

## Render Queue

![[9fa4b63e50b7a4612d5d0a08591529f7_MD5.jpg]]

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th>Properties</th><th>Value</th><th>渲染队列描述</th><th>说明</th></tr><tr><td>Background</td><td>1000</td><td>This Render Queue is rendered before any others</td><td>这个队列通常被最先渲染（比如天空盒）。</td></tr><tr><td>Geometry</td><td>2000</td><td>Opaque geometry uses this queue</td><td>这个默认的渲染队列。它被用于绝大数对象。不透明几何体使用该队列。</td></tr><tr><td>AlphaTest</td><td>2450</td><td>Alpha tested geometry uses this queue</td><td>需要开启透明度测试的物体。Unity5 以后从 Geometry 队列中拆出来，因为在所有不透明物体渲染完之后再渲染会比较高效。</td></tr><tr><td>GeometryLast</td><td>2500</td><td>Last Render Queue that is considered "opaque"</td><td>所有 Geometry 和 AlphaTest 队列的物体渲染完后。</td></tr><tr><td>Transparent</td><td>3000</td><td>This Render Queue is rendered after Geometry and AlphaTest,in back-to-front order</td><td>所有 Geometry 和 AlphaTest 队列渲染完后，再按照从后往前的顺序进行渲染，任何使用了透明度混合的物体都应该使用该队列（例如玻璃和粒子效果）。</td></tr><tr><td>Overlay</td><td>4000</td><td>This Render Queue is meat for overlay effects</td><td>该队列用于实现一些叠加效果，适合最后渲染的物体（如镜头光晕）。</td></tr></tbody></table>

## Camera render algorithm

### 非透明物体排序算法

camera.opaqueSortMode

*   Default ：在 Unity 2018.1 预设值 FrontToBack
*   FrontToBack ：从近到远排序绘制，由于 z-buffering 机制，能使得 GPU rendering 时有更好的性能
*   NoDistanceSort：关闭排序绘制，能减低 CPU 的使用量

### 透明物体排序算法

camera.transparencySortMode

*   Default：根据 camera projection mode 调整
*   Perspective：根据 camera 位置到物体中心（object center）的距离排序
*   Orthographic：根据 view plane 到物体中心（object center）的距离排序
*   CustomAxis：指定 axis 排序，专门用于 2D 游戏制作

## UGUI's rendering order

CanvasRenderer 可视为画在画布 Canvas 的元件，之后该画布再画在最终的画面上（eg：render target）

### Canvas

Screen Space - Overlay

![[55a299e7de41c82f4c3575de0fdde378_MD5.jpg]]

*   该 Canvas 由隐藏的 camera 处理，其 depth = 101（最后才处理，用户自建的 Camera Depth 最多 100）
*   多个相同的 canvas 使用 Sort order 来决定绘制顺序

Screen Space - Camera & World Space

![[bcef7b0b7bb75a0d0bd756cd891f833f_MD5.jpg]]

*   存在世界场景的平面
*   多个相同的 canvas 使用 Sorting Layer 以及 Order in Layer 来决定绘制顺序

### CanvasRenderer

关于同一个 canvas 下，其 CanvasRenderer 之间的 rendering order：**Render Queue > Transform order**

![[13689268295ba6d6c9a4925ed171ae8a_MD5.jpg]]

*   Render Queue：材质球的 Render Queue
*   Transform order：依照 Transform Hierarchy 关系，采用 Pre-order 方式排序

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><td>当所属 Canvas 的 render mode 为 Screen Space - Overlay，则无视 Render Queue。</td></tr></tbody></table>

## 最佳实践

*   3D

*   不透明物体 & 半透明物体（例如草、铁丝网等）依照场景摆放

*   不需要特别设定 rendering order
*   一切交给 z-buffering 机制

*   透明物体或者粒子特效可透过 Sorting Layer & Order in Layer 机制调整 rendering order

*   透明物体 shader 通常不会写 z-buffer（ZWrite Off）
*   可 hack inspector 来设定 renderer.sortingLayerID 以及 renderer.sortingOrder

*   2D

*   Sprite renderer 使用 Sorting Layer & Order in Layer 机制来调整 rendering order，以控制 depth

*   UGUI

*   利用 transform hierarchy 来建立 rendering order，根据性能优化可以拆成多个 canvas
*   若采用 Canvas render mode：World Space，想让 UI 与 3D 场景物体的结合，可将 canvas 视为 3D 物体去设计场景架构