# Game Objects and Scripts

Creating a Clock

*   Build a clock with simple objects.
*   Write a C# script.
*   Rotate the clock's arms to show the time.
*   Animate the arms.

sr-annote { all: unset; }

This is the first tutorial in a series about learning the [basics](https://catlikecoding.com/unity/tutorials/basics/) of working with Unity. In it we will create a simple clock and program a component to have it display the current time. You don't need to have any experience with the Unity editor yet, but you're assumed to have some experience with multi-window editor applications in general.

At the bottom of all my recent tutorials you'll find links to the tutorial license, a repository containing the finished tutorial project, and a PDF version of the tutorial page.

This tutorial is made with Unity 2020.3.6f1.

![](<images/1686829292768.png>)

It is time to create a clock.

## Creating a Project

Before we can start working with the Unity editor we must first create a project.

### New Project

When you open Unity you will be presented with the Unity Hub. This is a launcher and installer app from where you can create or open projects, install Unity versions, and do some other things. If you don't have Unity 2020.3 or higher installed add it now.

### Which Unity versions are appropriate?

Unity releases multiple new versions per year. There are two parallel release schedules. The most stable and safe are the LTS releases. LTS stands for long term support, which is two years in Unity's case. I stick to LTS versions for my tutorials. This tutorial uses 2020.3.6 specifically. The third portion of the version number indicates the patch release. Patch releases contain bug fixes and only rarely new functionality. A further f1 suffix indicates an official final release. Any 2020.3 version will do for this tutorial.

The highest Unity version is of the development branch, which introduces new features and possibly removes old functionality. These versions aren't as reliable as LTS versions and only remain supported for a few months each.

Occasionally my tutorials contain little questions and their answers, always in a gray box, like the one above. On a web page the answer is hidden by default. This can be toggled by clicking or tapping the question.

When you create a new project you get to pick its Unity version and a template. We'll use the standard 3D template. Once it's created it gets added to the list of projects and gets opened in the appropriate version of the Unity editor.

### Can I create a project with a different render pipeline?

Yes, the only difference is that the project will have more things in its default scene and your materials will look different. Your project will also contain the appropriate packages.

### Editor Layout

If you haven't customized the editor yet, you will end up with its default window layout.

![](<images/1686829293343.png>)

Default editor layout.

The default layout contains all the windows that we need, but you can customize it as you like, by reordering and grouping windows. You can also open and close windows, like the one of the asset store. Each window also has its own configuration options, accessible via the triple-dot button in their top right corner. Besides that most also have a toolbar with more options. If your window doesn't look the same as in the tutorials—for example the scene window has a uniform background instead of a skybox—then one of its options is different.

You can switch to a preconfigured layout via the dropdown menu at the top right of the Unity editor. You can also save your current layout there so you can revert to it later.

### Packages

Unity's functionality is modular. Besides the core functionality there are extra packages that can be downloaded and included in your project. The default 3D project currently includes a few packages by default, which you can see in the project window under _Packages_.

![](<images/1686829293873.png>)

Default packages.

These packages can be hidden, by toggling the button at the top right of the project window that looks like an eye with a dash through it. This is purely to reduce visual clutter in the editor, the packages are still part of the project. The button also displays how many such packages there are.

You can control which packages are included in your project via the package manager, which can be opened via the _Window / Package Manager_ menu item.

![](<images/1686829294413.png>)

Package manager, only showing packages in project.

The packages add extra functionality to Unity. For example, _Visual Studio Editor_ adds integration for the Visual Studio editor, used to write code. This tutorial doesn't use the functionality of the included packages, so I removed them all. The only exception is _Visual Studio Editor_ because that's the editor that I use for writing code. If you use a different editor you'd want to include its integration package, if it exists.

### Don't you also need the _Visual Studio Code Editor_ package?

Despite the similar names, _Visual Studio_ and _Visual Studio Code_ are two different editors. You only need one of the packages, depending on which editor you use.

The easiest way to remove packages is by first using the toolbar to limit the package list to _In Project_ only. Then select the packages one at a time and use the _Remove_ button at the bottom right of the window. Unity will recompile after each removal, so it takes a few seconds before the process is finished.

After removing everything except _Visual Studio Editor_ I an left with three packages visible in the project window: _Custom NUnit_, _Test Framework_, and _Visual Studio Editor_. The other two are still there because _Visual Studio Editor_ depends on them.

You can make dependencies and implicitly imported packages visible in the package manager via the project settings window, opened via _Edit / Project Settings..._ Select its _Package Manager_ category and then enable _Show Dependencies_ under _Advanced Settings_.

![](<images/1686829295006.png>)

Package manager project settings; _Show Dependencies_ enabled.

### Color Space

Nowadays rendering is usually done in linear color space, but Unity still configured to use gamma color space by default. For best visual results select the _Player_ category of the project settings window, open the _Other Settings_ panel, and scroll down to its _Rendering_ section. Make sure that _Color Space_ is set to _Linear_. Unity will show warning that this might take a long time, but this won't be the case for a nearly-empty project. Confirm to switch.

![](<images/1686829295571.png>)

Color space set to linear.

### Is there a reason to ever use gamma color space?

Only when you're targeting old hardware or old graphics APIs. OpenGL ES 2.0 and WebGL 1.0 don't support linear space, besides that gamma can be faster than linear on old mobile devices.

### Sample Scene

The new project contains a sample scene named _SampleScene_, which is opened by default. You can find its asset under _Assets / Scenes_ in the project window.

![](<images/1686829296313.png>)

Sample scene in project window.

By default the project window uses a two-column layout. You can switch to a one-column layout via its triple-dot configuration menu option.

![](<images/1686829296874.png>)

One-column layout.

The sample scene contains a main camera and a directional light. These are game objects. They are listed in the hierarchy window, under the scene.

![](<images/1686829297439.png>)

Object hierarchy within scene.

You can select a game object either via the hierarchy window or the scene window. The camera has a scene icon that looks like an oldfashioned film camera while the directional light's icon looks like a sun.

![](<images/1686829297967.png>)

Icons in scene window.

### How do I navigate the scene window?

You can use the alt or option key in combination with the cursor to rotate the view. You can also use the arrow keys to move the point of view, and zoom by scrolling. Also, pressing the F key focuses the view on the game object that is currently selected. There are more possibilities, but these are enough to find your way around the scene.

When an object is selected details about it will be shown in the inspector window, but we'll cover those when we need them. We won't need to modify the camera nor the light, so we can hide them in the scene by clicking the eye icon to the left of them in the hierarchy window. This icon is invisible by default but will appear when we hover the cursor there. This is purely to reduce visual clutter in the scene window.

![](<images/1686829298497.png>)

Hidden objects.

### What does the hand-like icon next to the eye do?

Next to the column that contains the eye icons is another column that contains hand-like icons. These icons are also invisible by default. When a game object's hand icon is active it is impossible to select the object via the scene window. This way you can control which objects respond to selection via the scene window.

## Building a Simple Clock

Now that our project is set up correctly we can start creating our clock.

### Creating a Game Object

We need a game object to represent the clock. We'll start with the simplest possible game object, which is an empty one. It can be created via the _GameObject / Create Empty_ menu option. Alternatively, you can use the _Create Empty_ option in the context menu of the hierarchy window, which you can open with an alternative click, usually a right-click or a two-finger tap. This will add the game object to the scene. It's visible and immediately selected in the hierarchy window under _SampleScene_, which is now marked with an asterisk to indicate that it has unsaved changes. You can also immediately change its name or leave that for later.

![](<images/1686829299027.png>)

Hierarchy with new game object selected.

The inspector window shows the details of the game object as long as it is selected. At its top is a header with the object's name plus a few configuration options. By default, the object is enabled, is not static, is untagged, and sits on the default layer. These settings are fine, except its name. Rename it to _Clock_.

![](<images/1686829299594.png>)

Inspector window with clock selected.

Below the header is a list of all the components of the game object. The list always has a `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component at the top, which is all our clock currently has. It controls the position, rotation, and scale of the game object. Make sure that all the clock's position and rotation values are set to 0. Its scale should be uniformly 1.

### What about 2D objects?

When working in 2D instead of 3D, you can ignore one of the three dimensions. Objects specifically meant for 2D—like UI elements—typically have a `[RectTransform](http://docs.unity3d.com/Documentation/ScriptReference/RectTransform.html)` instead, which is a specialized `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component.

Because the game object is empty it isn't visible in the scene window itself. However, a manipulation tool is visible at the game object's location, which is at the center of the world.

![](<images/1686829300153.png>)

Selected with move tool.

### Why don't I see a manipulation tool after selecting the clock?

The manipulation tool exists in the scene window. Make sure that you're looking at the scene window, not the game window.

Which manipulation tool is active can be controlled via the buttons at the top left of the editor toolbar. The modes can also be activated via the Q, W, E, R, T, and Y keys. The rightmost button in the group is for enabling custom editor tools, which we don't have. The move tool is active by default.

![](<images/1686829300713.png>)

Manipulation mode toolbar.

Next to the mode buttons are three more buttons to control the placement, orientation, and snapping of manipulation tools.

### Creating the Face of the Clock

Although we have a clock object, we don't see anything yet. We'll have to add 3D models to it so something gets rendered. Unity contains a few primitive objects that we can use to build a simple clock. Let's begin by adding a cylinder to the scene via _GameObject / 3D Object / Cylinder_. Make sure that it has the same `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` values as our clock.

![](<images/1686829301326.png>)

![](<images/1686829301878.png>)

Game object representing a cylinder.

The new object has three more components than an empty game object. First, it has a `[MeshFilter](http://docs.unity3d.com/Documentation/ScriptReference/MeshFilter.html)`, which contains a reference to the built-in cylinder mesh.

![](<images/1686829302445.png>)

`[MeshFilter](http://docs.unity3d.com/Documentation/ScriptReference/MeshFilter.html)` component, set to cylinder.

Second is a `[MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html)`. This component's purpose is to ensure that the object's mesh gets rendered. It also determines what material is used for rendering, which is the default material. This material is also shown in the inspector, below the component list.

![](<images/1686829302988.png>)

`[MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html)` component, set to default material.

Third is a `[CapsuleCollider](http://docs.unity3d.com/Documentation/ScriptReference/CapsuleCollider.html)`, which is for 3D physics. The object represents a cylinder, but it has a capsule collider because Unity doesn't have a primitive cylinder collider. We don't need it, so we can remove this component. If you'd like to use physics with your clock, you're better off using a `[MeshCollider](http://docs.unity3d.com/Documentation/ScriptReference/MeshCollider.html)` component. Components can be removed via the triple-dot dropdown menu in their top right corner.

![](<images/1686829303583.png>)

Cylinder without collider.

We'll turn the cylinder into the clock's face, by flattening it. This is done by decreasing the Y component of its scale. Reduce it to 0.2. As the cylinder mesh is two units high, its effective height becomes 0.4 units. Let's also make a big clock, so increase the X and Z components of its scale to 10.

![](<images/1686829304112.png>)

  

![](<images/1686829304672.png>)

Scaled cylinder.

Our clock is supposed to stand or hang on a wall, but its face is currently laying flat. We can fix this by rotating the cylinder a quarter turn. In Unity the X axis points right, the Y axis points up, and the Z axis points forward. So let's design our clock with the same orientation in mind, meaning that we see its front while we're looking at it along the Z axis. Set the cylinder's X rotation to 90 and adjust the scene view so the clock's front is visible, so the blue Z arrow of the move tool points away from you, into the screen.

![](<images/1686829305213.png>)

  

![](<images/1686829305767.png>)

Rotated cylinder.

Change the name of the cylinder object to _Face_, as it represents the face of the clock. It is only one part of the clock, so we make it a child of the _Clock_ object. We do this by dragging the face onto the clock in the hierarchy window.

![](<images/1686829306350.png>)

Face child object.

Child objects are subject to the transformation of their parent object. This means that when _Clock_ changes position, _Face_ does as well. It's as if they are a single entity. The same goes for rotation and scale. You can use this to make complex object hierarchies.

### Creating the Clock Periphery

The outer ring of a clock's face usually has markings that help indicate what time it is displaying. This is known as the clock periphery. Let's use blocks to indicate the hours of a 12-hour clock.

Add a cube object to the scene via _GameObject / 3D Object / Cube_, name it _Hour Indicator 12_, and also make it a child of _Clock_. The order of the child objects in the hierarchy doesn't matter, you could place it either above or below the face.

![](<images/1686829306917.png>)

Hour indicator child object.

Set its X scale to 0.5, Y scale to 1, and Z scale to 0.1 so it becomes a narrow flat long block. Then set its X position to 0, Y position to 4, and Z position to −0.25. That places it on top of the face to indicate hour 12. Also remove its `[BoxCollider](http://docs.unity3d.com/Documentation/ScriptReference/BoxCollider.html)` component.

![](<images/1686829307464.png>)

![](<images/1686829308033.png>)

Indicator for hour 12.

The indicator is hard to see, because it has the same color as the face. Let's create a separate material for it, via _Assets / Create / Material_, or via the plus button or context menu of the project window. This gives us a material asset that is a duplicate of the default material. Change its name to _Hour Indicator_.

![](<images/1686829308600.png>)

  

![](<images/1686829309194.png>)

Hour indicator in project window, one and two column layout.

Select the material and change its _Albedo_ to something else, by clicking its color field. That opens a color popup window which offers various ways to pick a color. I chose dark gray, corresponding to hexadecimal 494949, which is the same as uniform 73 for RGB 0–255 mode. We don't use the alpha channel so its value is irrelevant. We can also leave all other material properties as they are.

![](<images/1686829309746.png>)

Dark gray albedo.

### What is albedo?

Albedo is a Latin word which means whiteness. It's the color of something when illuminated by white light.

Make the hour indicator use this material. You can do this by dragging the material onto the object in either the scene or hierarchy window. You can also drag it to the bottom of the inspector window when the indicator game object is selected, or change _Element 0_ of the _Materials_ array of its `[MeshRenderer](http://docs.unity3d.com/Documentation/ScriptReference/MeshRenderer.html)`.

![](<images/1686829310351.png>)

![](<images/1686829310919.png>)

Dark hour indicator.

### Twelve Hour Indicators

We could make do with a single indicator for hour 12, but let's include one for every hour. Begin by orienting the scene view camera so we look straight down the Z axis. You can do this by clicking on the axis cones of the view camera gizmo at the top right of the scene view. You can also change the axis of the scene grid to Z via the grid toolbar button.

![](<images/1686829311487.png>)

Looking straight at clock, along Z axis.

Duplicate the _Hour Indicator 12_ game object. You can do this via _Edit / Duplicate_, via the indicated keyboard shortcut, or via the its context menu in the hierarchy window. The duplicate will appear underneath the original in the hierarchy window, also a child of _Clock_. Its name is set to _Hour Indicator 12 (1)_. Rename it to _Hour Indicator 6_ and negate the Y component of its position so it indicates hour 6.

![](<images/1686829312017.png>)

  

![](<images/1686829312547.png>)

Indicators for hours 6 and 12.

Create indicators for hours 3 and 9 in the same way. In this case their X positions should be 4 and −4 while their Y positions should be zero. Also, set their Z rotation to 90 so they're turned a quarter circle.

![](<images/1686829313111.png>)

Four hour indicators.

Then create another duplicate of _Hour Indicator 12_, this time for hour 1. Set its X position to 2, its Y position to 3.464, and its Z rotation to −30. Then duplicate that one for hour 2, swap its X and Y positions, and double its Z rotation to −60.

![](<images/1686829313649.png>)

Indicators for hours 1 and 2.

### Where do those numbers come from?

Each hour covers a 30° clockwise rotation along the Z axis. In this case we use negative rotations because Unity's rotation is counterclockwise. We can find the position for hour 1 via trigonometry. The sine of 30° is ½ and its cosine is √32. We scale those by the distance at which we position the hours indicator from the center, which is 4. So we end up with X 2 and Y 2√3≈3.464. For hour 2 the rotation is 60°, for which we can simply swap the sine and cosine.

Duplicate these two indicators and negate their Y positions and their rotations to create the indicators for hours 4 and 5. Then use the same trick on hours 1, 2, 4, and 5 to create the remaining indicators, this time negating their X positions and again their rotations.

![](<images/1686829314216.png>)

All hour indicators.

### Creating the Arms

The next step is to create the arms of the clock. We start with the hour arm. Again duplicate _Hour Indicator 12_ and name it _Hours Arm_. Then create a _Clock Arm_ material and make the arm use it. In this case I made it solid black, hexadecimal 000000. Decrease the arm's X scale to 0.3 and increase its Y scale to 2.5. Then change its Y position to 0.75 so it points towards hour 12, but also a bit in the opposite direction. That makes it look as if the arm has a little counterweight when it rotates.

![](<images/1686829314783.png>)

![](<images/1686829315310.png>)

Hours arm.

The arm must rotate around the center of the clock, but changing its Z rotation makes it rotate around its own center.

 <video src="" control></video>

 

Clock arm rotates around its center.

This happens because rotation is relative to the local position of the game object. To create the appropriate rotation we have to introduce a pivot object and rotate that one instead. So create a new empty game object and make it a child of _Clock_. You can do this directly by creating the object via the context menu of _Clock_ in the hierarchy window. Name it _Hours Arm Pivot_ and make sure that its position and rotation are zero and its scale is uniformly 1. Then make _Hours Arm_ a child of the pivot.

![](<images/1686829315878.png>)

Hours arm with pivot.

Now try rotating the pivot. If you do this via the scene view make sure that the tool handle position mode is set to _Pivot_ instead of _Center_.

 <video src="" control></video>

 

Clock arm rotates around the pivot.

Duplicate _Hours Arm Pivot_ twice to create a _Minutes Arm Pivot_ and a _Seconds Arm Pivot_. Rename them accordingly, including the duplicated arm child objects.

![](<images/1686829316437.png>)

All arm hierarchies.

_Minutes Arm_ should be narrower and longer than _Hours Arm_, so set its X scale to 0.2 and Y scale to 4, then increase its Y position to 1. Also change its Z position to −0.35 so it sits on top of the hours arm. Note that this applies to the arm, not its pivot.

![](<images/1686829316979.png>)

`[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` of _Minutes Arm_.

Adjust _Seconds Arm_ as well. This time use 0.1 and 5 for the XY scale and 1.25 and −0.45 for the YZ position.

![](<images/1686829317542.png>)

`[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` of _Seconds Arm_.

Let's make the seconds arm stand out by creating a separate material for it. I gave it a dark red color, hexadecimal B30000. Also, I turned off the grid in the scene window as we finished building our clock.

![](<images/1686829318072.png>)

Clock with three arms.

If you haven't done so already, this is a good moment to save the scene, via _File / Save_ or the indicated keyboard shortcut.

It is also a good idea to keep the project's assets organized. As we have three materials let's put them in a _Materials_ folder that we create via _Assets / Create / Folder_ or via the project window. You can then drag the materials there.

![](<images/1686829318600.png>)

  

![](<images/1686829319169.png>)

Material folder in project window, one and two column layout.

## Animating the Clock

Our clock currently does not tell the time, it's always stuck at twelve o'clock. To animate it we have to add a custom behavior to it. We do this by creating a custom component type, which is defined via a script.

### C# Script Asset

Add a new script asset to the project via _Assets / Create / C# Script_ and name it _Clock_. C# is the programming language used for Unity scripts and is pronounced as _C-sharp_. Let's also immediately put it in a new _Scripts_ folder to keep the project tidy.

![](<images/1686829319703.png>)

  

![](<images/1686829320244.png>)

Scripts folder with `**Clock**` script, one and two column layout.

When the script is selected the inspector will show its contents. But to edit the code we'll have to use a code editor. You can open the script for editing by pressing the _Open..._ button in its inspector or by double-clicking it in the hierarchy window. Which program gets opened can be configured via Unity's preferences.

![](<images/1686829320813.png>)

Inspector of C# `**Clock**` asset.

### Defining a Component Type

Once the script is loaded in your code editor begin by deleting the standard template code, as we'll create the component type from scratch.

An empty file defines nothing. It must contain the definition of our clock component. What we're going to define isn't a single instance of a component. Instead, we define the general class or type known as `**Clock**`. Once that's established, we could create multiple such components in Unity, even though we'll limit ourselves to a single clock in this tutorial.

In C#, we define the `**Clock**` type by first stating that we're defining a class, followed by its name. In the code fragments below, changed code has a yellow background, or dark red if you're using the dark web page theme to view this tutorial. As we start with an empty file, the contents of it should literally become `**class** **Clock**` and nothing else, though you could add spaces and newlines between words as you like.

```
class Clock
```

### What's a class, technically?

You can think of a class as a blueprint that can be used to create objects that reside in a computer's memory. The blueprint defines what data these objects contain and what functionality they have.

Classes can also define data and functionality that don't belong to object instances, but to the class itself. This is often used to provide globally-available functionality. We'll use some of that, but `**Clock**` won't have it.

Because we don't want to restrict which code has access to our `**Clock**` type, it is good form to prefix it with the `**public**` access modifier.

```
public class Clock
```

### What is the default access modifier for classes?

Without the access modifier, it would be as if we had written `**internal** **class** **Clock**`. That would restrict access to code from the same assembly, which becomes relevant when you use code packaged in separate assemblies. To make sure it always works, make classes public by default.

At this point we don't have valid C# syntax yet. If you were to save the file and go back to the Unity editor then compilation errors will get logged in its console window.

We indicated that we're defining a type, so we must actually define what it is like. That's done by a block of code that follows the declaration. The boundaries of a code block are indicated with curly brackets. We're leaving it empty for now, so just write `{}`.

```
public class Clock {}
```

Our code is now valid. Save the file and switch back to Unity. The Unity editor will detect that the script asset has changed and triggers a recompilation. After that is done, select our script. The inspector will inform us that the asset does not contain a `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` script.

![](<images/1686829321379.png>)

Non-component script.

What this means is that we cannot use this script to create components in Unity. At this point, our `**Clock**` defines a basic C# object type. Our custom component type must extend Unity's `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` type, inheriting its data and functionality.

### What does mono-behavior mean?

The idea is that we can program our own components to add custom behavior to game objects. That's what the behavior part refers to. It just happens to use the British spelling, which is an oddity. The mono part refers to the way in which support for custom code was added to Unity. It used the Mono project, which is a multi-platform implementation of the .NET framework. Hence, `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)`. It's an old name that we're stuck with due to backwards-compatibility.

To turn `**Clock**` into a subtype of `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)`, we have to change our type declaration so that it extends that type, which is done with a colon after our type name, followed by what it extends. This makes `**Clock**` inherit everything of the `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` class type.

```
public class Clock : MonoBehaviour {}
```

However, this will result in an error after compilation. The compiler complains that it cannot find the `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` type. This happens because the type is contained in a namespace, which is `UnityEngine`. To access it, we have to use its fully-qualified name, `UnityEngine.[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)`.

```
public class Clock : UnityEngine.MonoBehaviour {}
```

### What's a namespace?

A namespace is like a website domain, but for code. Just like domains can have subdomains, namespaces can have sub-namespaces. The big difference is that it's written the other way around. So instead of forum.unity.com it would be com.unity.forum. Namespaces are used to organize code and prevent name clashes.

The assembly containing the `UnityEngine` code comes with Unity, you don't have to go online to fetch it separately. The project file used by the code editor should be set up automatically to recognize it, if you imported the appropriate editor integration package.

  
访问Unity类型时，总是必须包含 `UnityEngine` 前缀是不方便的。幸运的是，我们可以声明应该自动搜索命名空间以完成C#文件中的类型名称。这是通过在文件顶部添加 `**using** UnityEngine;` 来完成的。分号是用来标记语句结尾的。

```
using UnityEngine;

public class Clock : MonoBehaviour {}
```

  
现在我们可以将我们的自定义组件添加到Unity中的游戏对象中。这可以通过将脚本资源拖动到对象上，也可以通过对象检查器底部的按钮来完成。

![](<images/1686829321908.png>)

  
时钟游戏对象与我们的 `**Clock**` 组件。

  
请注意，我的教程中的大多数代码类型都链接到在线文档。例如， `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 是一个链接，可以带您访问Unity针对该类型的在线脚本API页面。

###   
抓住一只手臂

  
若要旋转手臂， `**Clock**` 对象需要了解它们。让我们从计时臂开始。像所有游戏对象一样，它可以通过调整其 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件来旋转。因此，我们必须将臂枢轴的#2组件的知识添加到#3中。这可以通过在其代码块中添加一个数据字段来实现，该字段定义为名称后跟分号。

  
名称 `hours pivot` 将适用于该字段。但是，名称必须是单个单词。惯例是使字段名称的第一个单词小写，并将所有其他单词大写，然后将它们粘在一起。所以我们将其命名为 `hoursPivot` 。

```
public class Clock : MonoBehaviour {

	hoursPivot;
}
```

###   
`**using**` 语句去了哪里？

  
它仍然存在，我只是没有显示它。代码片段将包含足够多的现有代码，以便您了解更改的上下文。

  
我们还必须声明字段的类型，在本例中为 `UnityEngine.[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 。它必须写在字段名称的前面。

```
Transform hoursPivot;
```

  
我们的类现在定义了一个字段，该字段可以保存对另一个对象的引用，该对象的类型必须是 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 。我们必须确保它包含对小时臂枢轴的 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件的引用。

  
默认情况下，字段是私有的，这意味着它们只能由属于 `**Clock**` 的代码访问。但是这个类不知道我们的Unity场景，所以没有直接的方法将字段与正确的对象关联起来。我们可以通过将字段声明为可序列化来更改这一点。这意味着，当Unity保存场景时，它应该包含在场景的数据中，这是通过将所有数据放在一个序列中进行序列化并将其写入文件来实现的。

  
将字段标记为可序列化是通过向其附加一个属性来完成的，在本例中为 `[SerializeField](http://docs.unity3d.com/Documentation/ScriptReference/SerializeField.html)` 。它写在方括号之间的字段声明前面，通常在它上面的行上，但也可以放在同一行上。

```
[SerializeField]
	Transform hoursPivot;
```

###   
我们就不能做到 `**public**` 吗？

  
是的，但是公开类字段通常是不好的。经验法则是，只有当其他类型的C#代码需要访问类内容时，才公开类内容，然后更喜欢方法或属性而不是字段。可访问性越低的东西就越容易维护，因为直接依赖它的代码就越少。在本教程中，我们唯一的C#代码是 `**Clock**` ，所以没有理由公开其内容。

  
一旦字段可序列化，Unity将检测到这一点，并将其显示在我们游戏对象的 `**Clock**` 组件的检查器窗口中。

![](<images/1686829322470.png>)

 小时数据透视字段。

  
若要进行正确的连接，请将从层次结构拖动到字段上。或者，使用字段右侧的圆形按钮，在弹出的列表中搜索轴心。在这两种情况下，Unity编辑器都会获取的 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件，并将其引用到我们的字段中。

![](<images/1686829323042.png>)

 小时枢轴已连接。

###   
无所不知

  
我们必须对分秒臂枢轴做同样的事情。因此，在 `**Clock**` 中再添加两个具有适当名称的可序列化 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 字段。

```
[SerializeField]
	Transform hoursPivot;

	[SerializeField]
	Transform minutesPivot;

	[SerializeField]
	Transform secondsPivot;
```

  
可以使这些字段声明更加简洁，因为它们共享相同的属性、访问修饰符和类型。它们可以合并为一个逗号分隔的字段名称列表，该列表位于属性和类型声明之后。

```
[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	//[SerializeField]
	//Transform minutesPivot;

	//[SerializeField]
	//Transform secondsPivot;
```

###  `//` 做什么？

  
双斜杠表示注释。编译器将忽略它们之后直到行尾的所有文本。如果需要，它用于添加文本以澄清代码。我还用它来表示已删除的代码。除此之外，被删除的代码还有一行代码。

  
把编辑的另外两只手臂也勾起来。

![](<images/1686829323620.png>)

  
三个枢轴全部连接。

###  醒来

  
现在我们可以使用臂枢轴，下一步是旋转它们。要做到这一点，我们需要告诉 `**Clock**` 执行一些代码。这是通过向类中添加一个称为方法的代码块来完成的。块必须以名称为前缀，该名称按惯例大写。我们将其命名为 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` ，建议在组件唤醒时执行代码。

```
public class Clock : MonoBehaviour {

	[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	Awake {}
}
```

  
例如，方法有点像数学函数f(x)=2x+3  
.该函数采用由变量参数表示的数字x  
-加倍，然后加三。它对单个数字进行运算，其结果也是单个数字。在方法的情况下，它更像f(p)=c 哪里p  
表示输入参数，并且c  
表示它执行的任何代码。

  
就像数学函数一样，一个方法可以产生一个结果，但这不是必需的。我们必须像声明字段一样声明结果的类型，或者写 `**void**` 来表示没有结果。在我们的例子中，我们只想在不提供结果值的情况下执行一些代码，所以我们使用 `**void**` 。

```
void Awake {}
```

  
我们也不需要任何输入数据。然而，我们仍然需要将方法的参数定义为圆括号之间的逗号分隔列表。在我们的案例中，这只是一个空列表。

```
void Awake () {}
```

  
我们现在有了一个有效的方法，尽管它还没有做任何事情。就像Unity检测到我们的字段一样，它也检测到这个 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 方法。当一个组件有 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 方法时，Unity将在组件唤醒时调用该方法。这是在游戏模式下创建或加载后发生的。我们目前处于编辑模式，所以这还没有发生。

###   
难道 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 不一定是 `**public**` 吗？

  
`[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 和其他方法的集合被认为是特殊的Unity事件方法。Unity引擎将找到它们，并在适当的时候调用它们，无论我们如何声明它们。这种情况发生在托管.NET环境之外。

  
注意， `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 和其他特殊的Unity事件方法在我的教程中有粗体文本，并链接到它们的在线Unity脚本API页面。

###  通过代码旋转

  
要旋转手臂，我们必须创建一个新的旋转。我们可以通过为 `localRotation` 属性指定一个新的来更改 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 的旋转。

###  什么是财产？

  
属性是一种伪装成字段的方法。它可能是只读的或只写的。C#的约定是将属性大写，但Unity的代码没有做到这一点。

  
尽管 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件的旋转在检查器中是用每轴的欧拉角（以度为单位）定义的，但在代码中，我们必须使用四元数。

###  什么是四元数？

  
四元数基于复数，用于表示三维旋转。虽然比单独的X、Y和Z旋转角度的组合更难理解，但它们有一些有用的特性。例如，它们不会受到万向节锁定的影响。

  
我们可以通过调用 `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` 方法来创建基于欧拉角的四元数。将它写在 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 中，然后用分号结束语句。

```
void Awake () {
		Quaternion.Euler;
	}
```

  
该方法具有用于描述所需旋转的参数。在这种情况下，我们将提供一个逗号分隔的列表，其中包含三个参数，全部位于方法名称后面的圆括号之间。我们为X、Y和Z旋转提供了三个数字。前两个使用零，Z旋转使用−30。

```
Quaternion.Euler(0, 0, -30);
```

  
此调用的结果是一个 `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html)` 结构值，包含围绕Z轴的30°顺时针旋转，与时钟上的1小时相匹配。

###  什么是结构？

  
结构的缩写是蓝图，就像类一样。不同之处在于，无论它创建什么，都被视为一个简单的值，比如整数或颜色，而不是对象。它没有身份感。定义您自己的结构与定义类的工作原理相同，只是您编写 `**struct**` 而不是 `**class**` 。

  
若要将此旋转应用于时针臂，请使用 `=` 赋值语句将 `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` 的结果指定给 `hoursPivots.localRotation` 。

```
hoursPivot.localRotation = Quaternion.Euler(0, 0, -30);
```

###   
`localRotation` 和 `rotation` 之间有什么区别？

  
`localRotation` 属性表示 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件单独描述的旋转，因此它是相对于其父组件的旋转。这是你在它的检查员身上看到的轮换。相反， `rotation` 属性表示世界空间中的最终旋转，同时考虑整个对象层次。如果我们将时钟作为一个整体旋转，设置该属性会产生奇怪的结果，因为当该属性补偿时钟的旋转时，臂会忽略这一点。

###   
难道不应该有一个警告说 `hoursPivot` 从未初始化过吗？

  
编译器可以检测到没有任何代码为字段分配任何内容，并且确实可能发出这样的警告，因为它不知道我们是通过Unity的检查器设置的。但是，默认情况下会抑制此警告。抑制可以通过项目设置进行控制。在播放器/其他设置/脚本编译下有一个抑制常见警告切换。它禁止对未初始化和未使用的专用字段发出警告。

  
现在在编辑器中进入播放模式。您可以通过指示的键盘快捷键或按下编辑器窗口顶部中央的播放按钮来执行此操作。Unity将焦点切换到游戏窗口，该窗口将渲染场景中的内容。时钟组件将唤醒，时钟将设置为1点钟。

![](<images/1686829324171.png>)

  
在播放模式下总是一点钟。

  
如果相机没有聚焦在时钟上，您可以移动它，使时钟可见，但请记住，退出播放模式时会重置场景，因此在播放模式下对场景所做的任何更改都不会持续。然而，资产的情况并非如此，它们的变化总是持续存在的。在播放模式下，您还可以打开场景窗口，甚至打开多个场景和游戏窗口。退出播放模式，然后继续。

###   
获取当前时间

  
下一步是找出我们醒来的当前时间。我们可以使用 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 结构来访问我们正在运行的设备的系统时间。 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 不是Unity类型，它位于 `System` 命名空间中。它是.NET框架核心功能的一部分，Unity使用它来支持脚本编写。

  
`[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 具有 `Now` 属性，该属性生成包含当前系统日期和时间的 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 值。为了检查它是否正确，我们将在#3开始时将其记录到控制台。我们可以通过将它传递给#4方法来实现这一点。

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

  
现在，我们每次进入播放模式时都会记录一个时间戳。您可以在控制台窗口和编辑器窗口底部的状态栏中看到它。

###  旋转手臂

  
我们离工作时钟越来越近了。让我们再次从小时开始 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 有一个 `Hour` 属性，它为我们获取 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 值的hours部分。在当前时间戳上调用它将为我们提供一天中的时间。

```
Debug.Log(DateTime.Now.Hour);
```

  
因此，为了让小时臂显示当前小时，我们必须将−30°旋转乘以当前小时。使用星号 `*` 字符进行乘法运算。我们也不再需要记录当前时间，这样就可以删除该语句。

```
//Debug.Log(DateTime.Now.Hour);
		hoursPivot.localRotation = Quaternion.Euler(0, 0, -30 * DateTime.Now.Hour);
```

![](<images/1686829324734.png>)

  
目前四点钟处于播放模式。

  
为了明确我们正在从小时转换为度，我们可以定义一个包含转换因子的 `hoursToDegrees` 字段。 `[Quaternion](http://docs.unity3d.com/Documentation/ScriptReference/Quaternion.html).Euler` 的角度被定义为浮点值，所以我们将使用 `**float**` 类型。因为我们已经知道这个数字，所以我们可以立即将其作为字段声明的一部分进行赋值。然后与字段相乘，而不是与#4中的文字#3相乘。

```
float hoursToDegrees = -30;

	[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	void Awake () {
		hoursPivot.localRotation =
			Quaternion.Euler(0, 0, hoursToDegrees * DateTime.Now.Hour);
	}
```

###  什么是 `**float**` ？

  
计算机不能存储所有的数字，它们必须在由0或1组成的二进制存储器中表示。例如，这使得不可能在有限的内存大小内准确地存储许多数字⅓, 就像我们不能用十进制记数法精确地写出那个数字一样。我们能做的最好的事情就是写0.3333333，然后在某个点停止。

  
假设我们决定在点后面最多写三位数字，而在点前面只写一位⅓ 近似为0.333。如果我们分开⅓ 到100，我们将被迫写0.003，这意味着我们失去了两位数的精度。为了提高小值的精度，让我们添加一个单独的指数，指示数字的数量级。然后0.333×10−2  
可以表示⅓ 除以100，而不会丢失有意义的数字。我们可以使用0.333×102  
以表示乘以100，同时只在点前面保留一个数字。因此，点可以被认为是浮动的，因为它没有指定固定的数量级。这使得我们只能用几个数字来表示大量的数字。

  
浮点数字在计算机上的工作方式与此相同，只是它们使用二进制数字而不是十进制数字，并且还必须表示特殊值，如无穷大和非数字。 `**float**` 是这样一个存储在四个字节中的值，这意味着它有32位。

  
如果我们声明一个没有后缀的整数，那么它被认为是一个整数，这是一个不同的值类型。尽管编译器会自动转换它们，但让我们通过添加f后缀来明确我们所有的数字都是 `**float**` 类型的。

```
float hoursToDegrees = -30f;

	[SerializeField]
	Transform hoursPivot, minutesPivot, secondsPivot;

	void Awake () {
		hoursPivot.localRotation =
			Quaternion.Euler(0f, 0f, hoursToDegrees * DateTime.Now.Hour);
	}
```

  
每小时的度数总是相同的。我们可以通过在 `hoursToDegrees` 的声明中添加 `**const**` 前缀来实现这一点。这就把它变成了一个常数，而不是一个场。

```
const float hoursToDegrees = -30f;
```

###   
`**const**` 值有什么特别之处？

  
`**const**` 关键字表示值永远不会更改，并且不需要是字段。相反，它的值将在编译过程中计算，并被替换为常量的所有用法。这只适用于像数字这样的基元类型。

  
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

![](<images/1686829325329.png>)

 目前为5:16:31。

  
我们使用 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime).Now` 三次，以检索小时、分钟和秒。每次我们再次浏览该属性时，都需要进行一些工作，理论上可能会导致不同的时间值。为了确保这种情况不会发生，我们应该只检索一次时间。我们可以通过在方法内部声明一个变量并将时间分配给它，然后再使用这个值来实现这一点。让我们把它命名为 `time` 。

###  什么是变量？

  
变量的作用类似于字段，只是它仅在执行方法时存在。它属于方法，而不是类。

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

  
对于变量，可以省略类型声明，用 `**var**` 关键字替换它。这可以缩短代码，但只有当变量的类型可以从声明时分配给它的内容中推断出来时才有可能。此外，我更喜欢只在语句中明确提到类型时才这样做，这里就是这种情况。

```
var time = DateTime.Now;
```

###  设置手臂动画

  
当进入播放模式时，我们会得到当前时间，但在那之后，时钟仍然静止不动。要使时钟与当前时间保持同步，请将 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 方法的名称更改为 `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` 。这是另一种特殊的事件方法，只要我们处于播放模式，Unity就会每帧调用一次，而不是只调用一次。

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

 

 正在更新时钟。

###  框架是什么？

  
在播放模式下，Unity从主摄影机的角度不断渲染场景。一旦完成渲染，就将结果呈现给显示器。然后，显示器将显示该帧，直到获得下一帧为止。在渲染新帧之前，所有内容都会更新。因此，Unity会经历一系列更新、渲染、更新、渲染等过程。一个更新步骤然后渲染一次场景通常被视为一帧，但实际上时间更复杂。

  
请注意，我们的 `**Clock**` 组件在检查器中的名称前面有一个切换。这允许我们禁用它，从而阻止Unity调用其 `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` 方法。

![](<images/1686829325932.png>)

  
#可以禁用的0#组件。

###  连续旋转

  
时钟的指针准确地表示当前的小时、分钟或秒。它的行为就像一个数字时钟，离散但有臂。时钟通常具有缓慢旋转的臂，该臂提供时间的模拟表示。让我们改变我们的方法，让我们的时钟变成模拟的。

  
`[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime)` 不包含小数数据。幸运的是，它确实有一个 `TimeOfDay` 属性。这为我们提供了一个 `[TimeSpan](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=TimeSpan)` 值，该值通过其 `TotalHours` 、#4和 `TotalSeconds` 属性以我们需要的格式包含数据。

  
首先从 `[DateTime](https://learn.microsoft.com/en-us/search/?category=Reference&scope=.NET&terms=DateTime).Now` 中获取 `TimeOfDay` 结构值，然后将其存储在变量中。由于本语句中没有提到#2类型，我将使变量的类型显式。然后调整用于旋转手臂的属性。

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

  
这将导致编译器错误，抱怨我们无法从 `**double**` 转换为 `**float**` 。之所以会发生这种情况，是因为#2属性生成的值具有双精度浮点类型，即#3。这些值提供了比#4值更高的精度，但Unity的代码仅适用于单精度浮点值。

###   
单精度足够吗？

  
对于大多数游戏来说，是的。当使用非常大的距离或比例差异时，这会成为一个问题。然后，你必须应用隐形传送或相机相对渲染等技巧，使活动区域靠近世界原点。虽然使用双精度可以解决这个问题，但它也会使所涉及的数字的内存大小翻倍，这会导致其他性能问题。游戏引擎通常使用单精度浮点值，GPU也是如此。

  
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

 

 模拟时钟。

  
现在您了解了在Unity中创建对象和编写代码的基本原理。下一个教程是构建图形。

[license](https://catlikecoding.com/unity/tutorials/license/) [repository](https://bitbucket.org/catlikecodingunitytutorials/basics-01-game-objects-and-scripts/) [PDF](Game-Objects-and-Scripts.pdf)