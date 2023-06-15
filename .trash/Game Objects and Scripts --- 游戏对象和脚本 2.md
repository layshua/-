# Game Objects and Scripts

 1. 用简单的物体制作一个时钟。
 2. 编写一个 C# 脚本。
 3. 旋转时钟的指针来显示时间。
 4. 为手臂制作动画。

![](<images/1686829464193.png>)

### Creating a Game Object  
创建游戏对象

We need a game object to represent the clock. We'll start with the simplest possible game object, which is an empty one. It can be created via the _GameObject / Create Empty_ menu option. Alternatively, you can use the _Create Empty_ option in the context menu of the hierarchy window, which you can open with an alternative click, usually a right-click or a two-finger tap. This will add the game object to the scene. It's visible and immediately selected in the hierarchy window under _SampleScene_, which is now marked with an asterisk to indicate that it has unsaved changes. You can also immediately change its name or leave that for later.  
我们需要一个游戏对象来表示时钟。我们将从最简单的游戏对象开始，它是一个空的。它可以通过菜单选项创建。或者，您可以使用层次结构窗口的上下文菜单中的选项，您可以通过交替单击打开该窗口，通常是右键单击或用两个手指点击。这将把游戏对象添加到场景中。它是可见的，并立即在下的层次结构窗口中被选中，现在用星号标记，表示它有未保存的更改。您也可以立即更改其名称，或将其留待以后使用。

![](<images/1686829470293.png>)

Hierarchy with new game object selected.  
选择了新游戏对象的层次结构。

The inspector window shows the details of the game object as long as it is selected. At its top is a header with the object's name plus a few configuration options. By default, the object is enabled, is not static, is untagged, and sits on the default layer. These settings are fine, except its name. Rename it to _Clock_.  
只要选择了游戏对象，检查器窗口就会显示其详细信息。它的顶部是一个标题，其中包含对象的名称和一些配置选项。默认情况下，对象处于启用状态，不是静态的，未标记，并且位于默认图层上。除了名称之外，这些设置都很好。将其重命名为。

![](<images/1686829470859.png>)

Inspector window with clock selected.  
选中时钟的检查器窗口。

Below the header is a list of all the components of the game object. The list always has a [Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html) component at the top, which is all our clock currently has. It controls the position, rotation, and scale of the game object. Make sure that all the clock's position and rotation values are set to 0. Its scale should be uniformly 1.  
标题下方是游戏对象的所有组件的列表。列表的顶部总是有一个 [Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html) 组件，这是我们的时钟目前所拥有的全部。它控制游戏对象的位置、旋转和缩放。确保所有时钟的位置和旋转值都设置为 0。其比例应一致为 1。

### What about 2D objects?  
二维对象呢？

When working in 2D instead of 3D, you can ignore one of the three dimensions. Objects specifically meant for 2D—like UI elements—typically have a `[RectTransform](http://docs.unity3d.com/Documentation/ScriptReference/RectTransform.html)` instead, which is a specialized `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component.

Because the game object is empty it isn't visible in the scene window itself. However, a manipulation tool is visible at the game object's location, which is at the center of the world.  
因为游戏对象是空的，所以它在场景窗口本身中不可见。但是，操纵工具在游戏对象位于世界中心的位置是可见的。

![](<images/1686829471436.png>)

Selected with move tool.  
使用移动工具选择。

### Why don't I see a manipulation tool after selecting the clock?  
为什么我在选择时钟后看不到操作工具？

The manipulation tool exists in the scene window. Make sure that you're looking at the scene window, not the game window.

Which manipulation tool is active can be controlled via the buttons at the top left of the editor toolbar. The modes can also be activated via the Q, W, E, R, T, and Y keys. The rightmost button in the group is for enabling custom editor tools, which we don't have. The move tool is active by default.  
哪个操作工具处于活动状态可以通过编辑器工具栏左上角的按钮进行控制。也可以通过 Q、W、E、R、T 和 Y 键激活这些模式。该组中最右边的按钮用于启用自定义编辑器工具，而我们没有这些工具。默认情况下，移动工具处于活动状态。

![](<images/1686829472050.png>)

Manipulation mode toolbar.  
操纵模式工具栏。

Next to the mode buttons are three more buttons to control the placement, orientation, and snapping of manipulation tools.  
模式按钮旁边还有三个按钮，用于控制操纵工具的放置、方向和捕捉。

### Creating the Face of the Clock  
创造钟面

Although we have a clock object, we don't see anything yet. We'll have to add 3D models to it so something gets rendered. Unity contains a few primitive objects that we can use to build a simple clock. Let's begin by adding a cylinder to the scene via _GameObject / 3D Object / Cylinder_. Make sure that it has the same `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` values as our clock.  
虽然我们有一个时钟对象，但我们还没有看到任何东西。我们将不得不添加 3D 模型到其中，以便对某些内容进行渲染。Unity 包含一些原始对象，我们可以使用这些对象来构建一个简单的时钟。让我们首先通过将圆柱体添加到场景中。请确保它与我们的时钟具有相同的 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 值。

![](<images/1686829472620.png>)

![](<images/1686829473162.png>)

Game object representing a cylinder.  
代表圆柱体的游戏物体。

The new object has three more components than an empty game object. First, it has a `[MeshFilter](http://docs.unity3d.com/Documentation/ScriptReference/MeshFilter.html)`, which contains a reference to the built-in cylinder mesh.  
新的对象比空的游戏对象多了三个组件。首先，它有一个[MeshFilter](http://docs.unity3d.com/Documentation/ScriptReference/MeshFilter.html)，其中包含对内置圆柱体网格的引用。

![](<images/1686829473714.png>)

`[MeshFilter](http://docs.unity3d.com/Documentation/ScriptReference/MeshFilter.html)` component, set to cylinder.  
[MeshFilter](http://docs.unity3d.com/Documentation/ScriptReference/MeshFilter.html) 组件，设置为圆柱体。

Second is a `[MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html)`. This component's purpose is to ensure that the object's mesh gets rendered. It also determines what material is used for rendering, which is the default material. This material is also shown in the inspector, below the component list.  
第二个是 [MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html) 。该组件的目的是确保渲染对象的网格。它还确定用于渲染的材质，即默认材质。该材料也显示在部件列表下方的检查器中。

![](<images/1686829474281.png>)

`[MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html)` component, set to default material.  
`[MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html)` 组件，设置为默认材质。

Third is a `[CapsuleCollider](http://docs.unity3d.com/Documentation/ScriptReference/CapsuleCollider.html)`, which is for 3D physics. The object represents a cylinder, but it has a capsule collider because Unity doesn't have a primitive cylinder collider. We don't need it, so we can remove this component. If you'd like to use physics with your clock, you're better off using a `[MeshCollider](http://docs.unity3d.com/Documentation/ScriptReference/MeshCollider.html)` component. Components can be removed via the triple-dot dropdown menu in their top right corner.  
第三个是一个 `[CapsuleCollider](http://docs.unity3d.com/Documentation/ScriptReference/CapsuleCollider.html)` ，用于 3D 物理。该对象表示一个圆柱体，但它有一个胶囊碰撞器，因为 Unity 没有基本圆柱体碰撞器。我们不需要它，所以我们可以删除此组件。如果你想在你的时钟上使用物理学，你最好使用 `[MeshCollider](http://docs.unity3d.com/Documentation/ScriptReference/MeshCollider.html)` 组件。组件可以通过右上角的三点下拉菜单删除。

![](<images/1686829474874.png>)

Cylinder without collider.  
没有碰撞器的气缸。

We'll turn the cylinder into the clock's face, by flattening it. This is done by decreasing the Y component of its scale. Reduce it to 0.2. As the cylinder mesh is two units high, its effective height becomes 0.4 units. Let's also make a big clock, so increase the X and Z components of its scale to 10.  
我们将通过使圆柱体变平来将其变成钟面。这是通过减小其比例的 Y 分量来实现的。将其降低到 0.2。由于圆柱体网格的高度为两个单位，因此其有效高度变为 0.4 个单位。让我们制作一个大时钟，将其比例的 X 和 Z 分量增加到 10。

![](<images/1686829475441.png>)

  

![](<images/1686829476008.png>)

Scaled cylinder. 缩放圆柱体。

Our clock is supposed to stand or hang on a wall, but its face is currently laying flat. We can fix this by rotating the cylinder a quarter turn. In Unity the X axis points right, the Y axis points up, and the Z axis points forward. So let's design our clock with the same orientation in mind, meaning that we see its front while we're looking at it along the Z axis. Set the cylinder's X rotation to 90 and adjust the scene view so the clock's front is visible, so the blue Z arrow of the move tool points away from you, into the screen.  
我们的钟本应立在墙上或挂在墙上，但它的表面目前是平的。我们可以把圆柱体旋转四分之一圈来解决这个问题。在 Unity 中，X 轴指向右侧，Y 轴指向上方，Z 轴指向前方。因此，让我们在设计时钟时考虑到相同的方向，这意味着我们在沿着 Z 轴观察时钟时可以看到它的正面。将圆柱体的 X 旋转设置为 90，并调整场景视图，使时钟的前部可见，使移动工具的蓝色 Z 箭头指向远离您的屏幕。

![](<images/1686829476556.png>)

  

![](<images/1686829477089.png>)

Rotated cylinder. 旋转气缸。

Change the name of the cylinder object to _Face_, as it represents the face of the clock. It is only one part of the clock, so we make it a child of the _Clock_ object. We do this by dragging the face onto the clock in the hierarchy window.  
将圆柱体对象的名称更改为，因为它表示时钟的面。它只是时钟的一部分，所以我们把它作为物体的一个子。我们通过将面拖动到层次结构窗口中的时钟上来完成此操作。

![](<images/1686829477650.png>)

Face child object. 面向子对象。

Child objects are subject to the transformation of their parent object. This means that when _Clock_ changes position, _Face_ does as well. It's as if they are a single entity. The same goes for rotation and scale. You can use this to make complex object hierarchies.  
子对象受其父对象变换的约束。这意味着当改变位置时，也会这样做。就好像它们是一个单一的实体。旋转和缩放也是如此。您可以使用它来创建复杂的对象层次结构。

### Creating the Clock Periphery  
创建时钟外围设备

The outer ring of a clock's face usually has markings that help indicate what time it is displaying. This is known as the clock periphery. Let's use blocks to indicate the hours of a 12-hour clock.  
时钟表面的外圈通常有标记，有助于指示它显示的时间。这被称为时钟外围。让我们使用块来指示 12 小时时钟的小时数。

Add a cube object to the scene via _GameObject / 3D Object / Cube_, name it _Hour Indicator 12_, and also make it a child of _Clock_. The order of the child objects in the hierarchy doesn't matter, you could place it either above or below the face.  
通过将立方体对象添加到场景中，为其命名，并使其成为的子对象。子对象在层次中的顺序无关紧要，可以将其放置在面的上方或下方。

![](<images/1686829478179.png>)

Hour indicator child object.  
小时指示器子对象。

Set its X scale to 0.5, Y scale to 1, and Z scale to 0.1 so it becomes a narrow flat long block. Then set its X position to 0, Y position to 4, and Z position to −0.25. That places it on top of the face to indicate hour 12. Also remove its `[BoxCollider](http://docs.unity3d.com/Documentation/ScriptReference/BoxCollider.html)` component.  
将其 X 比例设定为 0.5，Y 比例设定为 1，Z 比例设定为 0.1，使其成为一个窄而平的长块。然后将其 X 位置设置为 0，将 Y 位置设置为 4，将 Z 位置设置为 - 0.25。这就把它放在脸的顶部，表示 12 小时。同时删除其 `[BoxCollider](http://docs.unity3d.com/Documentation/ScriptReference/BoxCollider.html)` 组件。

![](<images/1686829478721.png>)

![](<images/1686829479289.png>)

Indicator for hour 12.  
12 小时指示灯。

The indicator is hard to see, because it has the same color as the face. Let's create a separate material for it, via _Assets / Create / Material_, or via the plus button or context menu of the project window. This gives us a material asset that is a duplicate of the default material. Change its name to _Hour Indicator_.  
指示器很难看到，因为它和脸的颜色相同。让我们通过或通过项目窗口的加号按钮或上下文菜单为其创建一个单独的材质。这为我们提供了一个与默认材质重复的材质资产。将其名称更改为。

![](<images/1686829479823.png>)

  

![](<images/1686829480390.png>)

Hour indicator in project window, one and two column layout.  
项目窗口中的小时指示器，一列和两列布局。

Select the material and change its _Albedo_ to something else, by clicking its color field. That opens a color popup window which offers various ways to pick a color. I chose dark gray, corresponding to hexadecimal 494949, which is the same as uniform 73 for RGB 0–255 mode. We don't use the alpha channel so its value is irrelevant. We can also leave all other material properties as they are.  
选择材质并通过单击其颜色字段将其更改为其他材质。这会打开一个颜色弹出窗口，提供各种选择颜色的方法。我选择了深灰色，对应于十六进制 494949，这与 RGB 0–255 模式的统一 73 相同。我们不使用 alpha 通道，因此它的值无关紧要。我们也可以保留所有其他材料特性。

![](<images/1686829481003.png>)

Dark gray albedo. 深灰色反照率。

### What is albedo? 什么是反照率？

Albedo is a Latin word which means whiteness. It's the color of something when illuminated by white light.
反 rr
Make the hour indicator use this material. You can do this by dragging the material onto the object in either the scene or hierarchy window. You can also drag it to the bottom of the inspector window when the indicator game object is selected, or change _Element 0_ of the _Materials_ array of its `[MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html)`.  
使小时指示器使用这种材料。可以通过在场景或层次窗口中将材质拖动到对象上来执行此操作。当指示器游戏对象被选中时，您也可以将其拖动到检查器窗口的底部，或者更改其 `[MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html)` 的数组。

![](<images/1686829481596.png>)

![](<images/1686829482160.png>)

Dark hour indicator. 暗小时指示器。

### Twelve Hour Indicators 十二小时指示器

We could make do with a single indicator for hour 12, but let's include one for every hour. Begin by orienting the scene view camera so we look straight down the Z axis. You can do this by clicking on the axis cones of the view camera gizmo at the top right of the scene view. You can also change the axis of the scene grid to Z via the grid toolbar button.  
我们可以在第 12 小时使用一个指示器，但让我们每小时包含一个指示器。首先确定场景视图摄影机的方向，使我们沿 Z 轴向下看。可以通过单击场景视图右上角的视图摄影机 gizmo 的轴锥来执行此操作。也可以通过栅格工具栏按钮将场景栅格的轴更改为 Z。

![](<images/1686829482722.png>)

Looking straight at clock, along Z axis.  
沿着 Z 轴直视时钟。

Duplicate the _Hour Indicator 12_ game object. You can do this via _Edit / Duplicate_, via the indicated keyboard shortcut, or via the its context menu in the hierarchy window. The duplicate will appear underneath the original in the hierarchy window, also a child of _Clock_. Its name is set to _Hour Indicator 12 (1)_. Rename it to _Hour Indicator 6_ and negate the Y component of its position so it indicates hour 6.  
复制游戏对象。您可以通过、指示的键盘快捷键或层次结构窗口中的上下文菜单来执行此操作。重复项将出现在层次结构窗口中的原始项下面，也是的子项。其名称设置为。将其重命名为，并否定其位置的 Y 分量，使其指示小时 6。

![](<images/1686829483289.png>)

  

![](<images/1686829483862.png>)

Indicators for hours 6 and 12.  
第 6 小时和第 12 小时的指示灯。

Create indicators for hours 3 and 9 in the same way. In this case their X positions should be 4 and −4 while their Y positions should be zero. Also, set their Z rotation to 90 so they're turned a quarter circle.  
以相同的方式创建第 3 小时和第 9 小时的指标。在这种情况下，它们的 X 位置应该是 4 和−4，而它们的 Y 位置应该是零。此外，将它们的 Z 旋转设置为 90，使它们旋转四分之一圈。

![](<images/1686829484394.png>)

Four hour indicators. 四小时指示器。

Then create another duplicate of _Hour Indicator 12_, this time for hour 1. Set its X position to 2, its Y position to 3.464, and its Z rotation to −30. Then duplicate that one for hour 2, swap its X and Y positions, and double its Z rotation to −60.  
然后创建的另一个副本，这次是 1 小时。将其 X 位置设定为 2，将其 Y 位置设定为 3.464，将其 Z 旋转设定为−30。然后将其复制第 2 小时，交换其 X 和 Y 位置，并将其 Z 旋转加倍至−60。

![](<images/1686829484984.png>)

Indicators for hours 1 and 2.  
第 1 小时和第 2 小时的指示灯。

### Where do those numbers come from?  
这些数字是从哪里来的？

Each hour covers a 30° clockwise rotation along the Z axis. In this case we use negative rotations because Unity's rotation is counterclockwise. We can find the position for hour 1 via trigonometry. The sine of 30° is ½ and its cosine is √32. We scale those by the distance at which we position the hours indicator from the center, which is 4. So we end up with X 2 and Y 2√3≈3.464. For hour 2 the rotation is 60°, for which we can simply swap the sine and cosine.

Duplicate these two indicators and negate their Y positions and their rotations to create the indicators for hours 4 and 5. Then use the same trick on hours 1, 2, 4, and 5 to create the remaining indicators, this time negating their X positions and again their rotations.  
复制这两个指标，并否定它们的 Y 位置和旋转，以创建第 4 和第 5 小时的指标。然后在第 1、2、4 和 5 小时使用相同的技巧来创建剩余的指示器，这一次否定了它们的 X 位置和旋转。

![](<images/1686829485553.png>)

All hour indicators. 所有小时指示器。

### Creating the Arms 创造武器

The next step is to create the arms of the clock. We start with the hour arm. Again duplicate _Hour Indicator 12_ and name it _Hours Arm_. Then create a _Clock Arm_ material and make the arm use it. In this case I made it solid black, hexadecimal 000000. Decrease the arm's X scale to 0.3 and increase its Y scale to 2.5. Then change its Y position to 0.75 so it points towards hour 12, but also a bit in the opposite direction. That makes it look as if the arm has a little counterweight when it rotates.  
下一步是创建时钟臂。我们从时针开始。再次复制并命名。然后创建一个材料，让手臂使用它。在这个例子中，我把它做成了纯黑，十六进制 000000。将手臂的 X 比例减小到 0.3，将其 Y 比例增大到 2.5。然后将其 Y 位置更改为 0.75，使其指向 12 小时，但也指向相反的方向。这使得手臂在旋转时看起来好像有一点配重。

![](<images/1686829486085.png>)

![](<images/1686829486651.png>)

Hours arm. 小时武装。

The arm must rotate around the center of the clock, but changing its Z rotation makes it rotate around its own center.  
手臂必须绕时钟中心旋转，但更改其 Z 轴旋转会使其绕自己的中心旋转。

 <video src="" control></video>

 

Clock arm rotates around its center.

This happens because rotation is relative to the local position of the game object. To create the appropriate rotation we have to introduce a pivot object and rotate that one instead. So create a new empty game object and make it a child of _Clock_. You can do this directly by creating the object via the context menu of _Clock_ in the hierarchy window. Name it _Hours Arm Pivot_ and make sure that its position and rotation are zero and its scale is uniformly 1. Then make _Hours Arm_ a child of the pivot.  
之所以会发生这种情况，是因为旋转相对于游戏对象的局部位置。为了创建适当的旋转，我们必须引入一个枢轴对象并旋转该对象。因此，创建一个新的空游戏对象并使其成为的子对象。通过层次窗口中的上下文菜单创建对象，可以直接执行此操作。命名它，并确保它的位置和旋转为零，并且其比例一致为 1。然后制作枢轴的子对象。

![](<images/1686829487220.png>)

Hours arm with pivot.  
带枢轴的时针臂。

Now try rotating the pivot. If you do this via the scene view make sure that the tool handle position mode is set to _Pivot_ instead of _Center_.  
现在尝试旋转枢轴。如果通过场景视图执行此操作，请确保将工具控制柄位置模式设置为而不是。

 <video src="" control></video>

 

Clock arm rotates around the pivot.  
时钟臂绕枢轴旋转。

Duplicate _Hours Arm Pivot_ twice to create a _Minutes Arm Pivot_ and a _Seconds Arm Pivot_. Rename them accordingly, including the duplicated arm child objects.  
重复两次以创建和。相应地重命名它们，包括复制的手臂子对象。

![](<images/1686829487773.png>)

All arm hierarchies. 所有手臂层次结构。

_Minutes Arm_ should be narrower and longer than _Hours Arm_, so set its X scale to 0.2 and Y scale to 4, then increase its Y position to 1. Also change its Z position to −0.35 so it sits on top of the hours arm. Note that this applies to the arm, not its pivot.  
应比窄且长，因此将其 X 比例设置为 0.2，将 Y 比例设置为 4，然后将其 Y 位置增加到 1。同时将其 Z 位置更改为 - 0.35，使其位于时针臂的顶部。请注意，这适用于手臂，而不是其枢轴。

![](<images/1686829488336.png>)

`[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` of _Minutes Arm_. #第 0 个，共 0 个。

Adjust _Seconds Arm_ as well. This time use 0.1 and 5 for the XY scale and 1.25 and −0.45 for the YZ position.  
同时进行调整。这一次，XY 比例使用 0.1 和 5，YZ 位置使用 1.25 和−0.45。

![](<images/1686829488929.png>)

`[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` of _Seconds Arm_. #第 0 个，共 0 个。

Let's make the seconds arm stand out by creating a separate material for it. I gave it a dark red color, hexadecimal B30000. Also, I turned off the grid in the scene window as we finished building our clock.  
让我们通过创建一个单独的材料来突出秒针。我给它一个深红色，十六进制 B30000。此外，当我们完成时钟的构建时，我关闭了场景窗口中的网格。

![](<images/1686829489459.png>)

Clock with three arms.  
三臂钟。

If you haven't done so already, this is a good moment to save the scene, via _File / Save_ or the indicated keyboard shortcut.  
如果您还没有这样做，现在是通过或指示的键盘快捷键保存场景的好时机。

It is also a good idea to keep the project's assets organized. As we have three materials let's put them in a _Materials_ folder that we create via _Assets / Create / Folder_ or via the project window. You can then drag the materials there.  
保持项目资产的有序性也是一个好主意。由于我们有三种材料，让我们把它们放在一个文件夹中，我们通过或通过项目窗口创建。然后可以将材质拖动到那里。

![](<images/1686829490026.png>)

  

![](<images/1686829490557.png>)

Material folder in project window, one and two column layout.  
项目窗口中的材质文件夹，一列和两列布局。

## Animating the Clock 设置时钟动画

Our clock currently does not tell the time, it's always stuck at twelve o'clock. To animate it we have to add a custom behavior to it. We do this by creating a custom component type, which is defined via a script.  
我们的钟现在不报时，总是卡在十二点钟。要使其动画化，我们必须添加一个自定义行为。我们通过创建一个自定义组件类型来实现这一点，该类型是通过脚本定义的。

### C# Script Asset C# 脚本资源

Add a new script asset to the project via _Assets / Create / C# Script_ and name it _Clock_. C# is the programming language used for Unity scripts and is pronounced as _C-sharp_. Let's also immediately put it in a new _Scripts_ folder to keep the project tidy.  
通过将新的脚本资源添加到项目中并命名。C# 是用于 Unity 脚本的编程语言，发音为。让我们也立即把它放在一个新的文件夹中，以保持项目的整洁。

![](<images/1686829491152.png>)

  

![](<images/1686829491694.png>)

Scripts folder with `**Clock**` script, one and two column layout.  
脚本文件夹包含 `**Clock**` 脚本，一列和两列布局。

When the script is selected the inspector will show its contents. But to edit the code we'll have to use a code editor. You can open the script for editing by pressing the _Open..._ button in its inspector or by double-clicking it in the hierarchy window. Which program gets opened can be configured via Unity's preferences.  
选择脚本后，检查器将显示其内容。但是要编辑代码，我们必须使用代码编辑器。您可以通过按脚本检查器中的按钮或在层次结构窗口中双击打开脚本进行编辑。打开哪个程序可以通过 Unity 的首选项进行配置。

![](<images/1686829492289.png>)

Inspector of C# `**Clock**` asset.  
C# `**Clock**` 资产的检查员。

### Defining a Component Type  
定义零部件类型

Once the script is loaded in your code editor begin by deleting the standard template code, as we'll create the component type from scratch.  
在代码编辑器中加载脚本后，首先删除标准模板代码，因为我们将从头开始创建组件类型。

An empty file defines nothing. It must contain the definition of our clock component. What we're going to define isn't a single instance of a component. Instead, we define the general class or type known as `**Clock**`. Once that's established, we could create multiple such components in Unity, even though we'll limit ourselves to a single clock in this tutorial.  
空文件没有定义任何内容。它必须包含我们的时钟组件的定义。我们要定义的不是一个组件的单个实例。相反，我们定义了称为 `**Clock**` 的通用类或类型。一旦建立了这一点，我们就可以在 Unity 中创建多个这样的组件，尽管在本教程中我们将仅限于一个时钟。

In C#, we define the `**Clock**` type by first stating that we're defining a class, followed by its name. In the code fragments below, changed code has a yellow background, or dark red if you're using the dark web page theme to view this tutorial. As we start with an empty file, the contents of it should literally become `**class** **Clock**` and nothing else, though you could add spaces and newlines between words as you like.  
在 C# 中，我们定义 `**Clock**` 类型，首先声明我们定义的是一个类，然后是它的名称。在下面的代码片段中，更改后的代码背景为黄色，如果您使用深色网页主题来查看本教程，则为深红色。当我们从一个空文件开始时，它的内容应该变成 `**class** **Clock**` ，而不是其他内容，尽管你可以根据自己的喜好在单词之间添加空格和换行符。

```
class Clock
```

### What's a class, technically?  
从技术上讲，什么课？

You can think of a class as a blueprint that can be used to create objects that reside in a computer's memory. The blueprint defines what data these objects contain and what functionality they have.

Classes can also define data and functionality that don't belong to object instances, but to the class itself. This is often used to provide globally-available functionality. We'll use some of that, but `**Clock**` won't have it.

Because we don't want to restrict which code has access to our `**Clock**` type, it is good form to prefix it with the `**public**` access modifier.  
因为我们不想限制哪些代码可以访问我们的 `**Clock**` 类型，所以最好在它前面加上 `**public**` 访问修饰符。

```
public class Clock
```

### What is the default access modifier for classes?  
类的默认访问修饰符是什么？

Without the access modifier, it would be as if we had written `**internal** **class** **Clock**`. That would restrict access to code from the same assembly, which becomes relevant when you use code packaged in separate assemblies. To make sure it always works, make classes public by default.

At this point we don't have valid C# syntax yet. If you were to save the file and go back to the Unity editor then compilation errors will get logged in its console window.  
在这一点上，我们还没有有效的 C# 语法。如果您保存文件并返回 Unity 编辑器，则编译错误将记录在其控制台窗口中。

We indicated that we're defining a type, so we must actually define what it is like. That's done by a block of code that follows the declaration. The boundaries of a code block are indicated with curly brackets. We're leaving it empty for now, so just write `{}`.  
我们指出我们正在定义一个类型，所以我们必须实际定义它是什么样的。这是由声明后面的一块代码完成的。代码块的边界用花括号表示。我们暂时把它留空，所以只写 `{}` 。

```
public class Clock {}
```

Our code is now valid. Save the file and switch back to Unity. The Unity editor will detect that the script asset has changed and triggers a recompilation. After that is done, select our script. The inspector will inform us that the asset does not contain a `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` script.  
我们的代码现在有效。保存文件并切换回 Unity。Unity 编辑器将检测到脚本资产已更改，并触发重新编译。完成后，选择我们的脚本。检查员将通知我们资产不包含 `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 脚本。

![](<images/1686829492902.png>)

Non-component script. 非组件脚本。

What this means is that we cannot use this script to create components in Unity. At this point, our `**Clock**` defines a basic C# object type. Our custom component type must extend Unity's `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` type, inheriting its data and functionality.  
这意味着我们不能使用这个脚本在 Unity 中创建组件。在这一点上，我们的 `**Clock**` 定义了一个基本的 C# 对象类型。我们的自定义组件类型必须扩展 Unity 的 `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 类型，继承其数据和功能。

### What does mono-behavior mean?  
单声道行为是什么意思？

The idea is that we can program our own components to add custom behavior to game objects. That's what the behavior part refers to. It just happens to use the British spelling, which is an oddity. The mono part refers to the way in which support for custom code was added to Unity. It used the Mono project, which is a multi-platform implementation of the .NET framework. Hence, `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)`. It's an old name that we're stuck with due to backwards-compatibility.

To turn `**Clock**` into a subtype of `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)`, we have to change our type declaration so that it extends that type, which is done with a colon after our type name, followed by what it extends. This makes `**Clock**` inherit everything of the `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` class type.  
要将 `**Clock**` 转换为 `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 的子类型，我们必须更改我们的类型声明，以便它扩展该类型，这是在我们的类型名称后面加一个冒号，然后加上它扩展的内容。这使得 #2 继承了 #3 类类型的所有内容。

```
public class Clock : MonoBehaviour {}
```

However, this will result in an error after compilation. The compiler complains that it cannot find the `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` type. This happens because the type is contained in a namespace, which is `UnityEngine`. To access it, we have to use its fully-qualified name, `UnityEngine.[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)`.  
但是，这将导致编译后出现错误。编译器抱怨找不到 `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 类型。之所以会发生这种情况，是因为该类型包含在一个名称空间中，即 `UnityEngine` 。要访问它，我们必须使用它的完全限定名称 `UnityEngine.[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 。

```
public class Clock : UnityEngine.MonoBehaviour {}
```

### What's a namespace? 什么是命名空间？

A namespace is like a website domain, but for code. Just like domains can have subdomains, namespaces can have sub-namespaces. The big difference is that it's written the other way around. So instead of forum.unity.com it would be com.unity.forum. Namespaces are used to organize code and prevent name clashes.

The assembly containing the `UnityEngine` code comes with Unity, you don't have to go online to fetch it separately. The project file used by the code editor should be set up automatically to recognize it, if you imported the appropriate editor integration package.

It is inconvenient to always have to include the `UnityEngine` prefix when accessing Unity types. Fortunately we can declare that the namespace should be searched automatically to complete type names in the C# file. This is done by adding `**using** UnityEngine;` at the top of the file. The semicolon is required to mark the end of the statement.  
访问 Unity 类型时，总是必须包含 `UnityEngine` 前缀是不方便的。幸运的是，我们可以声明应该自动搜索命名空间以完成 C# 文件中的类型名称。这是通过在文件顶部添加 `**using** UnityEngine;` 来完成的。分号是用来标记语句结尾的。

```
using UnityEngine;

public class Clock : MonoBehaviour {}
```

Now we can add our custom component to the _Clock_ game object in Unity. This can be done either by dragging the script asset onto the object, or via the _Add Component_ button at the bottom of the object's inspector.  
现在我们可以将我们的自定义组件添加到 Unity 中的游戏对象中。这可以通过将脚本资源拖动到对象上，也可以通过对象检查器底部的按钮来完成。

![](<images/1686829493430.png>)

Clock game object with our `**Clock**` component.  
时钟游戏对象与我们的 `**Clock**` 组件。

Note that most code types in my tutorials are linked to online documentation. For example, `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` is a link that takes you to Unity's online scripting API page for that type.  
请注意，我的教程中的大多数代码类型都链接到在线文档。例如， `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 是一个链接，可以带您访问 Unity 针对该类型的在线脚本 API 页面。

### Getting Hold of an Arm  
抓住一只手臂

To rotate the arms, `**Clock**` objects need to know about them. Let's start with the hours arm. Like all game objects, it can be rotated by adjusting its `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component. So we have to add knowledge of the arm pivot's `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component to `**Clock**`. This can be done by adding a data field inside its code block, defined as a name followed by a semicolon.  
若要旋转手臂， `**Clock**` 对象需要了解它们。让我们从计时臂开始。像所有游戏对象一样，它可以通过调整其 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件来旋转。因此，我们必须将臂枢轴的 #2 组件的知识添加到 #3 中。这可以通过在其代码块中添加一个数据字段来实现，该字段定义为名称后跟分号。

The name `hours pivot` would be appropriate for the field. However, names have to be single words. The convention is to make the first word of a field name lowercase and capitalize all other words, then stick them together. So we'll name it `hoursPivot`.  
名称 `hours pivot` 将适用于该字段。但是，名称必须是单个单词。惯例是使字段名称的第一个单词小写，并将所有其他单词大写，然后将它们粘在一起。所以我们将其命名为 `hoursPivot` 。

```
public class Clock : MonoBehaviour {

	hoursPivot;
}
```

### Where did the `**using**` statement go?  
`**using**` 语句去了哪里？

It's still there, I just didn't show it. The code fragments will contain enough of the existing code so you know the context of the changes.

We also have to declare the type of the field, which in this case is `UnityEngine.[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)`. It has to be written in front of the field's name.  
我们还必须声明字段的类型，在本例中为 `UnityEngine.[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 。它必须写在字段名称的前面。

```
Transform hoursPivot;
```

Our class now defines a field that can hold a reference to another object, whose type has to be `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)`. We have to make sure that it holds a reference to the `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component of the hours arm pivot.  
我们的类现在定义了一个字段，该字段可以保存对另一个对象的引用，该对象的类型必须是 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 。我们必须确保它包含对小时臂枢轴的 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件的引用。

Fields are private by default, which means that they can only be accessed by the code belonging to `**Clock**`. But the class doesn't know about our Unity scene, so there's no direct way to associate the field with the correct object. We can change that by declaring the field as serializable. This means that it should be included in the scene's data when Unity saves the scene, which it does by putting all data in a sequence—serializing it—and writing it to a file.  
默认情况下，字段是私有的，这意味着它们只能由属于 `**Clock**` 的代码访问。但是这个类不知道我们的 Unity 场景，所以没有直接的方法将字段与正确的对象关联起来。我们可以通过将字段声明为可序列化来更改这一点。这意味着，当 Unity 保存场景时，它应该包含在场景的数据中，这是通过将所有数据放在一个序列中进行序列化并将其写入文件来实现的。

Marking a field as serializable is done by attaching an attribute to it, in this case `[SerializeField](http://docs.unity3d.com/Documentation/ScriptReference/SerializeField.html)`. It's written in front of the field declaration between square brackets, typically on the line above it but can also be placed on the same line.  
将字段标记为可序列化是通过向其附加一个属性来完成的，在本例中为 `[SerializeField](http://docs.unity3d.com/Documentation/ScriptReference/SerializeField.html)` 。它写在方括号之间的字段声明前面，通常在它上面的行上，但也可以放在同一行上。

```
[SerializeField]
	Transform hoursPivot;
```

### Can't we just make it `**public**`?  
我们就不能做到 `**public**` 吗？

Yes, but it is generally bad form to make class fields publicly accessible. The rule of thumb is to only make class contents public if C# code from other types need access to it, and then prefer methods or properties over fields. The less accessible something is the easier it is to maintain, because there's less code that could directly depend on it. In this tutorial our only C# code is `**Clock**` so there's no reason to make its contents public.

Once the field is serializable Unity will detect this and display it in the inspector window of the `**Clock**` component of our _Clock_ game object.  
一旦字段可序列化，Unity 将检测到这一点，并将其显示在我们游戏对象的 `**Clock**` 组件的检查器窗口中。

![](<images/1686829493971.png>)

Hours pivot field. 小时数据透视字段。

To make the proper connection, drag the _Hours Arm Pivot_ from the hierarchy onto the _Hours Pivot_ field. Alternatively, use the circular button at the right of the field and search for the pivot in the list that pops up. In both cases the Unity editor grabs the `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component of _Hours Arm Pivot_ and puts a reference to it in our field.  
若要进行正确的连接，请将从层次结构拖动到字段上。或者，使用字段右侧的圆形按钮，在弹出的列表中搜索轴心。在这两种情况下，Unity 编辑器都会获取的 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件，并将其引用到我们的字段中。

![](<images/1686829494532.png>)

Hours pivot connected. 小时枢轴已连接。

### Knowing all Three Arms  
无所不知

We have to do the same for the minutes and seconds arm pivots. So add two more serializable `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` fields to `**Clock**` with appropriate names.  
我们必须对分秒臂枢轴做同样的事情。因此，在 `**Clock**` 中再添加两个具有适当名称的可序列化 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 字段。

```
[SerializeField]
	Transform hoursPivot;

	[SerializeField]
	Transform minutesPivot;

	[SerializeField]
	Transform secondsPivot;
```

It is possible to make these field declarations more concise, because they share the same attribute, access modifier, and type. They can be consolidated into a comma-separated list of field names following the attribute and type declaration.  
可以使这些字段声明更加简洁，因为它们共享相同的属性、访问修饰符和类型。它们可以合并为一个逗号分隔的字段名称列表，该列表位于属性和类型声明之后。

```
[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	//[SerializeField]
	//Transform minutesPivot;

	//[SerializeField]
	//Transform secondsPivot;
```

### What does `//` do? `//` 做什么？

Double slashes indicate a comment. All text after them until the end of the line is ignored by the compiler. It is used to add text to clarify code, if needed. I also use it to indicate code that has been removed. Besides that, deleted code has a line through it.

Hook up the other two arms in the editor as well.  
把编辑的另外两只手臂也勾起来。

![](<images/1686829495084.png>)

All three pivots connected.  
三个枢轴全部连接。

### Waking Up 醒来

Now that we have access to the arm pivots the next step is to rotate them. To do this, we need to tell `**Clock**` to execute some code. This is done by adding a code block to the class, known as a method. The block has to be prefixed by a name, which is capitalized by convention. We'll name it `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)`, suggesting that the code should be executed when the component awakens.  
现在我们可以使用臂枢轴，下一步是旋转它们。要做到这一点，我们需要告诉 `**Clock**` 执行一些代码。这是通过向类中添加一个称为方法的代码块来完成的。块必须以名称为前缀，该名称按惯例大写。我们将其命名为 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` ，建议在组件唤醒时执行代码。

```
public class Clock : MonoBehaviour {

	[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	Awake {}
}
```

Methods are somewhat like mathematical functions, for example  
例如，方法有点像数学函数 f(x)=2x+3. That function takes a number—represented by the variable parameter  
. 该函数采用由变量参数表示的数字 x—doubles it, then adds three. It operates on a single number, and its result is a single number as well. In the case of a method, it's more like  
- 加倍，然后加三。它对单个数字进行运算，其结果也是单个数字。在方法的情况下，它更像 f(p)=c where  哪里 p represents input parameters and  
表示输入参数，并且 c represents whatever code it executes.  
表示它执行的任何代码。

Like a mathematical function a method can produce a result, but this isn't required. We have to declare the type of the result—as if it were a field—or write `**void**` to indicate that there is no result. In our case, we just want to execute some code without providing a resulting value, so we use `**void**`.  
就像数学函数一样，一个方法可以产生一个结果，但这不是必需的。我们必须像声明字段一样声明结果的类型，或者写 `**void**` 来表示没有结果。在我们的例子中，我们只想在不提供结果值的情况下执行一些代码，所以我们使用 `**void**` 。

```
void Awake {}
```

We also don't need any input data. However, we still have to define the method's parameters, as a comma-separated list between round brackets. It's just an empty list in our case.  
我们也不需要任何输入数据。然而，我们仍然需要将方法的参数定义为圆括号之间的逗号分隔列表。在我们的案例中，这只是一个空列表。

```
void Awake () {}
```

We now have a valid method, although it doesn't do anything yet. Just like Unity detected our fields, it also detects this `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` method. When a component has an `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` method, Unity will invoke that method on the component when it awakens. This happens after it's been created or loaded while in play mode. We're currently in edit mode, so this doesn't happen yet.  
我们现在有了一个有效的方法，尽管它还没有做任何事情。就像 Unity 检测到我们的字段一样，它也检测到这个 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 方法。当一个组件有 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 方法时，Unity 将在组件唤醒时调用该方法。这是在游戏模式下创建或加载后发生的。我们目前处于编辑模式，所以这还没有发生。

### Doesn't `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` have to be `**public**`?  
难道 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 不一定是 `**public**` 吗？

`[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` and a collection of other methods are considered special Unity event methods. The Unity engine will find them and invoke them when appropriate, no matter how we declare them. This happens from outside the managed .NET environment.

Note that `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` and other special Unity event methods have bold text in my tutorials and link to their online Unity scripting API page.  
注意， `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 和其他特殊的 Unity 事件方法在我的教程中有粗体文本，并链接到它们的在线 Unity 脚本 API 页面。

### Rotating via Code 通过代码旋转

To rotate the arms we have to create a new rotation. We can change the rotation of a `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` by assigning a new one to its `localRotation` property.  
要旋转手臂，我们必须创建一个新的旋转。我们可以通过为 `localRotation` 属性指定一个新的来更改 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 的旋转。

### What's a property? 什么是财产？

A property is a method that pretends to be a field. It might be read-only or write-only. The C# convention is to capitalize properties, but Unity's code doesn't do this.

Although the rotation of a `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component is defined with Euler angles in degrees per axis in the inspector, in code we have to do it with a quaternion.  
尽管 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件的旋转在检查器中是用每轴的欧拉角（以度为单位）定义的，但在代码中，我们必须使用四元数。

### What's a quaternion? 什么是四元数？

Quaternions are based on complex numbers and are used to represent 3D rotations. Although harder to understand than a combination of separate X, Y, and Z rotation angles, they have some useful characteristics. For example, they don't suffer from gimbal lock.

We can create a quaternion based on Euler angles by invoking the `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` method. Do this by writing it in `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)`, followed by a semicolon to end the statement.  
我们可以通过调用 `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` 方法来创建基于欧拉角的四元数。将它写在 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 中，然后用分号结束语句。

```
void Awake () {
		Quaternion.Euler;
	}
```

The method has parameters used to describe the desired rotation. In this case we'll provide a comma-separated list of containing three arguments, all between round brackets, after the method name. We supply three numbers for the X, Y, and Z rotations. Use zero for the first two and −30 for the Z rotation.  
该方法具有用于描述所需旋转的参数。在这种情况下，我们将提供一个逗号分隔的列表，其中包含三个参数，全部位于方法名称后面的圆括号之间。我们为 X、Y 和 Z 旋转提供了三个数字。前两个使用零，Z 旋转使用−30。

```
Quaternion.Euler(0, 0, -30);
```

The result of this invocation is a `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html)` struct value containing a 30° clockwise rotation around the Z axis, matching hour 1 on our clock.  
此调用的结果是一个 `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html)` 结构值，包含围绕 Z 轴的 30° 顺时针旋转，与时钟上的 1 小时相匹配。

### What's a struct? 什么是结构？

A struct—short for structure—is a blueprint, just like a class. The difference is that whatever it creates is treated as a simple value, like an integer or color, instead of an object. It has no sense of identity. Defining your own structure works the same as defining a class, except you write `**struct**` instead of `**class**`.

To apply this rotation to the hour arm assign the result of `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` to `hoursPivots.localRotation`, using the `=` assignment statement.  
若要将此旋转应用于时针臂，请使用 `=` 赋值语句将 `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` 的结果指定给 `hoursPivots.localRotation` 。

```
hoursPivot.localRotation = Quaternion.Euler(0, 0, -30);
```

### What's the difference between `localRotation` and `rotation`?  
`localRotation` 和 `rotation` 之间有什么区别？

The `localRotation` property represents the rotation described by the `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component in isolation, thus it is a rotation relative to its parent. It's the rotation that you see in its inspector. In contrast, the `rotation` property represents the final rotation in world space, taking the entire object hierarchy into account. Setting that property would produce weird results if we rotate the clock as a whole, because the arm would ignore that as the property compensates for the rotation of the clock.

### Shouldn't there be a warning that `hoursPivot` is never initialized?  
难道不应该有一个警告说 `hoursPivot` 从未初始化过吗？

The compiler can detect that no code assigns anything to the field and could indeed issue such a warning, because it is unaware that we set it up via Unity's inspector. However, this warning is suppressed by default. The suppression can be controlled via the project settings. There's a _Suppress Common Warnings_ toggle under _Player / Other Settings / Script Compilation_. It suppresses warnings about both uninitialized and unused private fields.

Now enter play mode in the editor. You can do this via _Edit / Play_, the indicated keyboard shortcut, or by pressing the play button at top center of the editor window. Unity will switch focus to the game window, which renders what the _Main Camera_ in the scene sees. The clock component will awaken, and the clock will be set to one o'clock.  
现在在编辑器中进入播放模式。您可以通过指示的键盘快捷键或按下编辑器窗口顶部中央的播放按钮来执行此操作。Unity 将焦点切换到游戏窗口，该窗口将渲染场景中的内容。时钟组件将唤醒，时钟将设置为 1 点钟。

![](<images/1686829495679.png>)

Always one o'clock in play mode.  
在播放模式下总是一点钟。

If the camera isn't focused on the clock you can move it so the clock becomes visible, but keep in mind that the scene is reset when exiting play mode, so any changes that you make to the scene while in play mode will not persist. This isn't true for assets though, changes to them always persist. You can also have the scene window—or even multiple scene and game windows—open while in play mode. Exit play mode before continuing.  
如果相机没有聚焦在时钟上，您可以移动它，使时钟可见，但请记住，退出播放模式时会重置场景，因此在播放模式下对场景所做的任何更改都不会持续。然而，资产的情况并非如此，它们的变化总是持续存在的。在播放模式下，您还可以打开场景窗口，甚至打开多个场景和游戏窗口。退出播放模式，然后继续。

### Getting the Current Time  
获取当前时间

The next step is to figure out the current time when we awaken. We can use the `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` struct to access the system time of the device we're running on. `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` isn't a Unity type, it is found in the `System` namespace. It is part of the core functionality of the .NET framework, which is what Unity uses to support scripting.  
下一步是找出我们醒来的当前时间。我们可以使用 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 结构来访问我们正在运行的设备的系统时间。 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 不是 Unity 类型，它位于 `System` 命名空间中。它是. NET 框架核心功能的一部分，Unity 使用它来支持脚本编写。

`[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` has a `Now` property that produces a `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` value containing the current system date and time. To check whether it's correct we'll log it to the console at the start of `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)`. We can do that by passing it to the `[Debug](http://docs.unity3d.com/Documentation/ScriptReference/Debug.html).Log` method.  
`[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 具有 `Now` 属性，该属性生成包含当前系统日期和时间的 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 值。为了检查它是否正确，我们将在 #3 开始时将其记录到控制台。我们可以通过将它传递给 #4 方法来实现这一点。

```
using System;
using UnityEngine;

public class Clock : MonoBehaviour {

	[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	void Awake () {
		Debug.Log(DateTime.Now);
		hoursPivot.localRotation = Quaternion.Euler(0, 0, -30);
	}
}
```

Now we get a timestamp logged each time we enter play mode. You can see it both in the console window and in the status bar at the bottom of the editor window.  
现在，我们每次进入播放模式时都会记录一个时间戳。您可以在控制台窗口和编辑器窗口底部的状态栏中看到它。

### Rotating the Arms 旋转手臂

We're getting close to a working clock. Let's again start with the hours. `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` has an `Hour` property that gets us the hours portion of a `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` value. Invoking it on the current timestamp will give us the hour of the day.  
我们离工作时钟越来越近了。让我们再次从小时开始 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 有一个 `Hour` 属性，它为我们获取 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 值的 hours 部分。在当前时间戳上调用它将为我们提供一天中的时间。

```
Debug.Log(DateTime.Now.Hour);
```

So to have the hours arm show the current hour we have to multiply the −30° rotation by the current hour. Multiplication is done with the asterisk `*` character. We also no longer need to log the current time so can get rid of that statement.  
因此，为了让小时臂显示当前小时，我们必须将−30° 旋转乘以当前小时。使用星号 `*` 字符进行乘法运算。我们也不再需要记录当前时间，这样就可以删除该语句。

```
//Debug.Log(DateTime.Now.Hour);
		hoursPivot.localRotation = Quaternion.Euler(0, 0, -30 * DateTime.Now.Hour);
```

![](<images/1686829496242.png>)

Currently four o'clock in play mode.  
目前四点钟处于播放模式。

To make it clear that we're converting from hours to degrees, we can define an `hoursToDegrees` field containing the conversion factor. The angles of `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` are defined as floating-point values, so we'll use the `**float**` type. Because we already know the number, we can immediately assign it as part of the field declaration. Then multiply with the field instead of the literal `-30` in `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)`.  
为了明确我们正在从小时转换为度，我们可以定义一个包含转换因子的 `hoursToDegrees` 字段。 `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` 的角度被定义为浮点值，所以我们将使用 `**float**` 类型。因为我们已经知道这个数字，所以我们可以立即将其作为字段声明的一部分进行赋值。然后与字段相乘，而不是与 #4 中的文字 #3 相乘。

```
float hoursToDegrees = -30;

	[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	void Awake () {
		hoursPivot.localRotation =
			Quaternion.Euler(0, 0, hoursToDegrees * DateTime.Now.Hour);
	}
```

### What's a `**float**`? 什么是 `**float**` ？

Computers cannot store all numbers, they have to be representable in their binary memory which consists of bits that are either 0 or 1. This makes it impossible to store many numbers exactly within a finite memory size, for example ⅓, just like how we cannot exactly write that number in decimal notation. The best we can do is write 0.3333333 and stop at some point.

Suppose that we decide to write at most three digits after the dot and only one in front of it. Then ⅓ is approximated by 0.333. If we were to divide ⅓ by 100 then we'd be forced to write 0.003, which means that we lost two digits of precision. To improve precision of small values let's add a separate exponent that indicates the order of magnitude of our number. Then 0.333×10−2 can represent ⅓ divided by 100, without losing meaningful digits. And we can use 0.333×102 to represent multiplication by 100 as well, while keeping only a single digit in front of the dot. Thus the dot can be considered to float, as it doesn't designate a fixed order of magnitude. This allows us to use only a few digits to represents a large amount of numbers.

Floating-point numbers work the same way for computers, except that they use binary instead of decimal digits and also have to represent special values like infinities and not-a-number. A `**float**` is such a value stored in four bytes, which means that it has 32 bits.

If we declare a whole number without a suffix then it's assumed to be an integer, which is a different value type. Although the compiler converts them automatically, let's make explicit that all our numbers are of type `**float**`, by adding the f suffix to them.  
如果我们声明一个没有后缀的整数，那么它被认为是一个整数，这是一个不同的值类型。尽管编译器会自动转换它们，但让我们通过添加 f 后缀来明确我们所有的数字都是 `**float**` 类型的。

```
float hoursToDegrees = -30f;

	[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	void Awake () {
		hoursPivot.localRotation =
			Quaternion.Euler(0f, 0f, hoursToDegrees * DateTime.Now.Hour);
	}
```

The amount of degrees per hour is always the same. We can enforce this by adding the `**const**` prefix to the declaration of `hoursToDegrees`. This turns it into a constant instead of a field.  
每小时的度数总是相同的。我们可以通过在 `hoursToDegrees` 的声明中添加 `**const**` 前缀来实现这一点。这就把它变成了一个常数，而不是一个场。

```
const float hoursToDegrees = -30f;
```

### What's special about `**const**` values?  
`**const**` 值有什么特别之处？

The `**const**` keyword indicates that a value will never change and doesn't need to be a field. Instead, its value will be computed during compilation and is substituted for all usage of the constant. This is only possible for primitive types like numbers.

Let's give the same treatment to the other two arms, using the appropriate properties of `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)`. Both a minute and a second are represented by a rotation of negative six degrees.  
让我们使用 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 的适当属性对其他两个臂进行相同的处理。一分钟和一秒都用负六度的旋转来表示。

```
const float hoursToDegrees = -30f, minutesToDegrees = -6f, secondsToDegrees = -6f;

	[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	void Awake () {
		hoursPivot.localRotation =
			Quaternion.Euler(0f, 0f, hoursToDegrees * DateTime.Now.Hour);
		minutesPivot.localRotation =
			Quaternion.Euler(0f, 0f, minutesToDegrees * DateTime.Now.Minute);
		secondsPivot.localRotation =
			Quaternion.Euler(0f, 0f, secondsToDegrees * DateTime.Now.Second);
	}
```

![](<images/1686829496783.png>)

Currently 5:16:31. 目前为 5:16:31。

We're using `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime).Now` three times, to retrieve the hour, minute, and second. Each time we go through the property again, which requires some work, which could theoretically result in different time values. To make sure that this doesn't happen, we should retrieve the time only once. We can do this by declaring a variable inside the method and assign the time to it, then use this value afterwards. Let's name it `time`.  
我们使用 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime).Now` 三次，以检索小时、分钟和秒。每次我们再次浏览该属性时，都需要进行一些工作，理论上可能会导致不同的时间值。为了确保这种情况不会发生，我们应该只检索一次时间。我们可以通过在方法内部声明一个变量并将时间分配给它，然后再使用这个值来实现这一点。让我们把它命名为 `time` 。

### What's a variable? 什么是变量？

A variable acts like a field, except that it exists only while a method is being executed. It belongs to the method, not the class.

```
void Awake () {
		DateTime time = DateTime.Now;
		hoursPivot.localRotation =
			Quaternion.Euler(0f, 0f, hoursToDegrees * time.Hour);
		minutesPivot.localRotation =
			Quaternion.Euler(0f, 0f, minutesToDegrees * time.Minute);
		secondsPivot.localRotation =
			Quaternion.Euler(0f, 0f, secondsToDegrees * time.Second);
	}
```

In case of a variable it's possible to omit the type declaration, replacing it with the `**var**` keyword. This can shorten the code but is only possible when the variable's type can be inferred from what is assigned to it when it is declared. Also, I prefer to only do this when the type is explicitly mentioned in the statement, which is the case here.  
对于变量，可以省略类型声明，用 `**var**` 关键字替换它。这可以缩短代码，但只有当变量的类型可以从声明时分配给它的内容中推断出来时才有可能。此外，我更喜欢只在语句中明确提到类型时才这样做，这里就是这种情况。

```
var time = DateTime.Now;
```

### Animating the Arms 设置手臂动画

We get the current time when entering play mode, but after that the clock remains motionless. To keep the clock synchronized with the current time, change the name of our `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` method to `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)`. This is another special event method that gets invoked by Unity every frame instead of just once, as long as we stay in play mode.  
当进入播放模式时，我们会得到当前时间，但在那之后，时钟仍然静止不动。要使时钟与当前时间保持同步，请将 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 方法的名称更改为 `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` 。这是另一种特殊的事件方法，只要我们处于播放模式，Unity 就会每帧调用一次，而不是只调用一次。

```
void Update () {
		var time = DateTime.Now;
		hoursPivot.localRotation =
			Quaternion.Euler(0f, 0f, hoursToDegrees * time.Hour);
		minutesPivot.localRotation =
			Quaternion.Euler(0f, 0f, minutesToDegrees * time.Minute);
		secondsPivot.localRotation =
			Quaternion.Euler(0f, 0f, secondsToDegrees * time.Second);
	}
```

 <video src="" control></video>

 

Updating clock. 正在更新时钟。

### What's a frame? 框架是什么？

While in play mode Unity continually renders the scene from the point of view of the main camera. Once rendering is done the result is presented to the display. The display will then show that frame until it gets the next one. Before rendering a new frame everything gets updated. So Unity goes through a sequence of update, render, update, render, and so on. A single update step followed by rendering the scene once is typically considered a single frame, though in reality the timing is more complicated.

Note that our `**Clock**` component has gained a toggle in front of its name in the inspector. This allows us to disable it, which prevents Unity from invoking its `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` method.  
请注意，我们的 `**Clock**` 组件在检查器中的名称前面有一个切换。这允许我们禁用它，从而阻止 Unity 调用其 `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` 方法。

![](<images/1686829497351.png>)

`**Clock**` component that can be disabled.  
#可以禁用的 0# 组件。

### Continuously Rotating 连续旋转

The arms of our clock indicate exactly the current hour, minute, or second. It behaves like a digital clock, discrete but with arms. Typically clocks have slowly-rotating arms that provide an analog representation of time. Let's change our approach so our clock becomes analog.  
时钟的指针准确地表示当前的小时、分钟或秒。它的行为就像一个数字时钟，离散但有臂。时钟通常具有缓慢旋转的臂，该臂提供时间的模拟表示。让我们改变我们的方法，让我们的时钟变成模拟的。

`[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` doesn't contain fractional data. Fortunately, it does have a `TimeOfDay` property. This gives us a `[TimeSpan](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=TimeSpan)` value that contains the data in the format that we need, via its `TotalHours`, `TotalMinutes`, and `TotalSeconds` properties.  
`[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 不包含小数数据。幸运的是，它确实有一个 `TimeOfDay` 属性。这为我们提供了一个 `[TimeSpan](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=TimeSpan)` 值，该值通过其 `TotalHours` 、#4 和 `TotalSeconds` 属性以我们需要的格式包含数据。

Begin by getting the `TimeOfDay` struct value from `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime).Now` and store that in the variable instead. As the `[TimeSpan](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=TimeSpan)` type isn't mentioned in this statement I'll make the variable's type explicit. Then adjust the properties that we use to rotate the arms.  
首先从 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime).Now` 中获取 `TimeOfDay` 结构值，然后将其存储在变量中。由于本语句中没有提到 #2 类型，我将使变量的类型显式。然后调整用于旋转手臂的属性。

```
void Update () {
		TimeSpan time = DateTime.Now.TimeOfDay;
		hoursPivot.localRotation =
			Quaternion.Euler(0f, 0f, hoursToDegrees * time.TotalHours);
		minutesPivot.localRotation =
			Quaternion.Euler(0f, 0f, minutesToDegrees * time.TotalMinutes);
		secondsPivot.localRotation =
			Quaternion.Euler(0f, 0f, secondsToDegrees * time.TotalSeconds);
	}
```

This will result in compiler errors, complaining that we cannot convert from `**double**` to `**float**`. This happens because the `[TimeSpan](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=TimeSpan)` properties produce values with the double-precision floating point type, known as `**double**`. These values provide higher precision than `**float**` values, but Unity's code only works with single-precision floating point values.  
这将导致编译器错误，抱怨我们无法从 `**double**` 转换为 `**float**` 。之所以会发生这种情况，是因为 #2 属性生成的值具有双精度浮点类型，即 #3。这些值提供了比 #4 值更高的精度，但 Unity 的代码仅适用于单精度浮点值。

### Is single precision enough?  
单精度足够吗？

For most games, yes. It becomes a problem when working with very large distances or scale differences. Then you'll have to apply tricks like teleportation or camera-relative rendering to keep the active area near the world origin. While using double precision would solve this problem, it would also double the memory size of the numbers involved, which leads to other performance problems. Game engines typically use single-precision floating-point values, and so do GPUs.

We can solve this problem by explicitly converting from `**double**` to `**float**`. This process is known as casting and is done by writing the new type within round brackets in front of the value to be converted.  
我们可以通过显式地从 `**double**` 转换为 `**float**` 来解决这个问题。这个过程被称为铸造，通过在要转换的值前面的圆括号中写入新类型来完成。

```
hoursPivot.localRotation =
			Quaternion.Euler(0f, 0f, hoursToDegrees * (float)time.TotalHours);
		minutesPivot.localRotation =
			Quaternion.Euler(0f, 0f, minutesToDegrees * (float)time.TotalMinutes);
		secondsPivot.localRotation =
			Quaternion.Euler(0f, 0f, secondsToDegrees * (float)time.TotalSeconds);
```

 <video src="" control></video>

 

Analog clock. 模拟时钟。

Now you know the fundamentals of object creation and writing code in Unity. The next tutorial is [Building a Graph](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/).  
现在您了解了在 Unity 中创建对象和编写代码的基本原理。下一个教程是构建图形。

[license 许可证](https://catlikecoding.com/unity/tutorials/license/) [repository](https://bitbucket.org/catlikecodingunitytutorials/basics-01-game-objects-and-scripts/) [PDF](Game-Objects-and-Scripts.pdf)