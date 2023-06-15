# Building a Graph 构建图表

Visualizing Math 数学可视化

*   Create a prefab. 创建预制件。
*   Instantiate multiple cubes.  
    实例化多个多维数据集。
*   Show a mathematical function.  
    显示一个数学函数。
*   Create a surface shader and shader graph.  
    创建曲面着色器和着色器图形。
*   Animate the graph. 为图形制作动画。

sr-annote { all: unset; }

This is the second tutorial in a series about learning the [basics](https://catlikecoding.com/unity/tutorials/basics/) of working with Unity. This time we'll use game objects to build a graph, so we can show mathematical formulas. We'll also make the function time-dependent, creating an animating graph.  
这是关于学习使用Unity的基础知识系列的第二个教程。这次我们将使用游戏对象来构建一个图，这样我们就可以显示数学公式。我们还将使函数与时间相关，创建一个动画图形。

This tutorial is made with Unity 2020.3.6f1.  
本教程是用Unity 2020.3.6f1制作的。

![](<images/1686829710414.png>)

Using cubes to show a sine wave.  
使用立方体显示正弦波。

## Creating a Line of Cubes  
创建立方体线

A good understanding of mathematics is essential when programming. At its most fundamental level math is the manipulation of symbols that represent numbers. Solving an equation boils down to rewriting one set of symbols so it becomes another—usually shorter—set of symbols. The rules of mathematics dictate how this rewriting can be done.  
编程时，对数学的良好理解是必不可少的。在最基本的层面上，数学是对代表数字的符号的处理。求解一个方程可以归结为重写一组符号，使其成为另一组通常较短的符号。数学规则决定了如何重写。

For example, we have the function  
例如，我们有f(x)=x+1. We can substitute a number for its  
.我们可以用一个数字来代替它x parameter, say 3. That leads to  
参数，比如3。这导致f(3)=3+1=4. We provided 3 as an input argument and ended up with 4 as the output. We can say that the function maps 3 to 4. A shorter way to write this would be as an input-output pair, like (3,4). We can create many pairs of the form  
。我们提供了3作为输入参数，最后得到了4作为输出。我们可以说，函数映射3到4。写这篇文章的一种较短的方法是将其作为输入输出对，如（3，4）。我们可以创建许多对形式(x,f(x)), for example (5,6) and (8,9) and (1,2) and (6,7). But it is easier to understand the function when we order the pairs by the input number. (1,2) and (2,3) and (3,4) and so on.  
，例如（5，6）和（8，9）以及（1，2）和（6，7）。但是，当我们根据输入数字对配对进行排序时，更容易理解函数。（1,2）和（2,3）以及（3,4）等等。

The function  功能f(x)=x+1 is easy to understand.  
很容易理解。f(x)=(x−1)4+5x3−8x2+3x is harder. We could write down a few input-output pairs, but that likely won't give us a good grasp of the mapping it represents. We're going to need many points, close together. That will end up as a sea of numbers, which are hard to parse. Instead, we could interpret the pairs as two-dimensional coordinates of the form  
更难。我们可以写下一些输入输出对，但这可能不会让我们很好地掌握它所代表的映射。我们需要很多点，紧密地联系在一起。这最终会变成一个数字的海洋，很难解析。相反，我们可以将这些对解释为形式的二维坐标[xf(x)]. This is a 2D vector where the top number represents the horizontal coordinate, on the X axis, and the bottom number represents the vertical coordinate, on the Y axis. In other words,  
这是2D矢量，其中顶部数字表示X轴上的水平坐标，底部数字表示Y轴上的垂直坐标。换句话说，y=f(x). We can plot these points on a surface. If we use enough points that are very close together we end up with a line. The result is a graph.  
.我们可以在曲面上绘制这些点。如果我们使用足够多的非常接近的点，我们最终会得到一条线。结果是一个图形。

![](<images/1686829711012.png>)

Graph with  使用绘制图形x between −2 and 2, made with [Desmos](https://www.desmos.com/calculator/di84egsf7a).  
介于−2和2之间，由Desmos制成。

Looking at a graph can quickly give us an idea of how a function behaves. It's a handy tool, so let's create one in Unity. We'll start with a new project, as described in the first section of the [previous tutorial](https://catlikecoding.com/unity/tutorials/basics/game-objects-and-scripts/).  
看一张图可以很快让我们了解函数的行为。这是一个方便的工具，所以让我们在Unity中创建一个。我们将从一个新项目开始，如前一教程的第一部分所述。

### Prefabs 前言

Graphs are created by placing points at the appropriate coordinates. To do this, we need a 3D visualization of a point. We'll simply use Unity's default cube game object for this. Add one to the scene and name it _Point_. Remove its `[BoxCollider](http://docs.unity3d.com/Documentation/ScriptReference/BoxCollider.html)` component, as we won't use physics.  
图形是通过将点放置在适当的坐标上来创建的。要做到这一点，我们需要一个点的三维可视化。我们将简单地使用Unity的默认立方体游戏对象。将一个添加到场景中并命名。删除它的 `[BoxCollider](http://docs.unity3d.com/Documentation/ScriptReference/BoxCollider.html)` 组件，因为我们不会使用物理。

### Are cubes the best way to visualize graphs?  
立方体是可视化图形的最佳方式吗？

You could also use a particle system or line segments, but individual cubes are the simplest to use.  
您也可以使用粒子系统或线段，但单独的立方体最容易使用。

We will be using a custom component to create many instances of this cube and position them correctly. In order to do this we'll turn the cube into a game object template. Drag the cube from the hierarchy window into the project window. This will create a new asset, known as a prefab. It is a pre-fabricated game object that exists in the project, not in a scene.  
我们将使用自定义组件来创建此多维数据集的许多实例并正确定位它们。为了做到这一点，我们将把立方体变成一个游戏对象模板。将多维数据集从层次结构窗口拖动到项目窗口中。这将创建一个新的资产，称为预制件。它是一个预先制作的游戏对象，存在于项目中，而不是场景中。

![](<images/1686829711579.png>)

  

![](<images/1686829712140.png>)

Point prefab asset, one and two column layout.  
点预制资产，一列和两列布局。

The game object that we used to create the prefab still exists in the scene, but is now a prefab instance. It has a blue icon in the hierarchy window and an arrow to its right side. The header of its inspector also indicates that it is a prefab and displays a few more controls. The position and rotation are now displayed with bold text, which indicates that the values of the instance override the prefab's. Any other changes that you make to the instance will also be indicated this way.  
我们用来创建预制件的游戏对象仍然存在于场景中，但现在是预制件实例。它在层次结构窗口中有一个蓝色图标，右侧有一个箭头。其检查器的标题也表示它是预制的，并显示了更多的控件。位置和旋转现在以粗体文本显示，这表示实例的值将覆盖预制件的值。您对实例所做的任何其他更改也将以这种方式指示。

![](<images/1686829712670.png>)

  

![](<images/1686829713203.png>)

Point prefab instance. 点预制实例。

When selecting the prefab asset its inspector will show its root game object and a big button to open the prefab.  
当选择预制资产时，其检查器将显示其根游戏对象和打开预制的大按钮。

![](<images/1686829713745.png>)

Prefab asset inspector. 预置资产检查员。

Clicking the _Open Prefab_ button will make the scene window show a scene that contains nothing but the prefab's object hierarchy. You can also get there via the _Open_ button of an instance, the right arrow next to an instance in the hierarchy window, or by double-clicking the asset in the project window. This is useful when a prefab has a complex hierarchy, but this isn't the case for our simple point prefab.  
单击该按钮将使场景窗口显示一个只包含预制对象层次结构的场景。您也可以通过实例的按钮、层次结构窗口中实例旁边的右箭头或双击项目窗口中的资源来到达该位置。当预制件具有复杂的层次结构时，这很有用，但对于我们的简单点预制件来说，情况并非如此。

![](<images/1686829714280.png>)

Hierarchy window for our prefab.  
我们预制件的层次结构窗口。

You can exit the prefab's scene via the arrow to the left of its name in the hierarchy window.  
可以通过层次窗口中预制件名称左侧的箭头退出预制件的场景。

### Why is the background of the prefab scene uniform dark blue?  
为什么预制场景的背景是统一的深蓝色？

If you open a prefab instance that's part of a scene then the scene window will display its surroundings depending on the _Context_ settings shown at the top of the window. If you open the prefab asset then there is no context. In the case of assets the skybox is disabled by default in the prefab scene, along with some other things. You can configure this via the scene window's toolbar, just like you can for the regular scene window. The skybox can be toggled via the dropdown menu that looks like a stack with a star on top of it. Notice how the scene toolbar settings change when you jump in and out of prefab asset mode.  
如果打开作为场景一部分的预制实例，则场景窗口将根据窗口顶部显示的上下文设置显示其周围环境。如果打开预制资产，则没有上下文。在资源的情况下，预制场景中的skybox以及其他一些功能在默认情况下处于禁用状态。您可以通过场景窗口的工具栏对此进行配置，就像您可以对常规场景窗口进行配置一样。skybox可以通过下拉菜单进行切换，下拉菜单看起来像一个顶部有星星的堆栈。请注意，当您进入和退出预制资产模式时，场景工具栏设置是如何更改的。

Prefabs are a handy way to configure game objects. If you change the prefab asset all instances of it in any scene are changed in the same way. For example, changing the prefab's scale will also change the scale of the cube that's still in the scene. However, each instance uses its own position and rotation. Also, game object instances can be modified, which overrides the prefab's values. Note that the relationship between prefab and instance is broken while in play mode.  
前言是配置游戏对象的一种方便方法。如果更改预制资源，则任何场景中的所有预制资源实例都将以相同的方式更改。例如，更改预制件的比例也会更改仍在场景中的立方体的比例。但是，每个实例都使用自己的位置和旋转。此外，可以修改游戏对象实例，从而覆盖预制件的值。请注意，在播放模式下，预制和实例之间的关系已断开。

We're going to use a script to create instances of the prefab, which means that we no longer need the prefab instance that is currently in the scene. So delete it, either via _Edit / Delete_, the indicated keyboard shortcut, or its context menu in the hierarchy window.  
我们将使用脚本来创建预制实例，这意味着我们不再需要当前场景中的预制实例。因此，通过指示的键盘快捷键或层次结构窗口中的上下文菜单删除它。

### Graph Component 图形组件

We need a C# script to generate a graph with our point prefab. Create one and name it `**Graph**`.  
我们需要一个C#脚本来生成一个带有点预制的图。创建一个并将其命名为 `**Graph**` 。

![](<images/1686829714851.png>)

Graph C# asset in Scripts folder.  
Scripts文件夹中的Graph C#资产。

We begin with a simple class that extends `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` so it can be used as a component for game objects. Give it a serializable field to hold a reference to a prefab for instantiating points, named `pointPrefab`. We'll need access to the `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component to position the points, so make that the field's type.  
我们从一个扩展 `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 的简单类开始，这样它就可以用作游戏对象的组件。给它一个可串行化的字段来保存对实例化点的预制块的引用，该预制块名为 `pointPrefab` 。我们需要访问#2组件来定位点，所以要确保字段的类型。

```
using UnityEngine;

public class Graph : MonoBehaviour {

	[SerializeField]
	Transform pointPrefab;
}
```

Add an empty game object to the scene and name it _Graph_. Make sure that its position and rotation are zero, and that its scale is 1. Add our `**Graph**` component to this object. Then drag our prefab asset onto the _Point Prefab_ field of the graph. It now holds a reference to the prefab's `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component.  
将一个空的游戏对象添加到场景中并命名。确保其位置和旋转为零，并且其比例为1。将我们的 `**Graph**` 组件添加到此对象。然后将我们的预制资产拖到图的字段上。现在，它包含了对预制件#1组件的引用。

![](<images/1686829715399.png>)

Graph game object with reference to prefab.  
图形游戏对象参考预制。

### Instantiating Prefabs 实例化前言

Instantiating a game object is done via the `[Object](http://docs.unity3d.com/Documentation/ScriptReference/Object.html).[Instantiate](http://docs.unity3d.com/Documentation/ScriptReference/Object.Instantiate.html)` method. This is a publicly available method of Unity's `[Object](http://docs.unity3d.com/Documentation/ScriptReference/Object.html)` type, which `**Graph**` indirectly inherited by extending `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)`. The `[Instantiate](http://docs.unity3d.com/Documentation/ScriptReference/Object.Instantiate.html)` method clones whatever Unity object is passed to it as an argument. In the case of a prefab, it will result in an instance being added to the current scene. Let's do this when our `**Graph**` component awakens.  
实例化游戏对象是通过 `[Object](http://docs.unity3d.com/Documentation/ScriptReference/Object.html).[Instantiate](http://docs.unity3d.com/Documentation/ScriptReference/Object.Instantiate.html)` 方法完成的。这是Unity的 `[Object](http://docs.unity3d.com/Documentation/ScriptReference/Object.html)` 类型的一个公开可用的方法， `**Graph**` 通过扩展 `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 间接继承了该方法。#4方法克隆作为参数传递给它的任何Unity对象。在预制的情况下，它将导致一个实例被添加到当前场景中。让我们在 `**Graph**` 组件唤醒时执行此操作。

```
public class Graph : MonoBehaviour {

	[SerializeField]
	Transform pointPrefab;
	
	void Awake () {
		Instantiate(pointPrefab);
	}
}
```

### What is the full inheritance chain of `[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)`?  
`[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 的完整继承链是什么？

`[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` extends `[Behaviour](http://docs.unity3d.com/Documentation/ScriptReference/Behaviour.html)`, which extends `[Component](http://docs.unity3d.com/Documentation/ScriptReference/Component.html)`, which extends `[Object](http://docs.unity3d.com/Documentation/ScriptReference/Object.html)`.  
`[MonoBehaviour](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.html)` 扩展 `[Behaviour](http://docs.unity3d.com/Documentation/ScriptReference/Behaviour.html)` ，扩展 `[Component](http://docs.unity3d.com/Documentation/ScriptReference/Component.html)` ，扩展 `[Object](http://docs.unity3d.com/Documentation/ScriptReference/Object.html)` 。

If we enter play mode now a single instance of the _Point_ prefab will be spawned at the world origin. Its name is the same as the prefab's, with _(Clone)_ appended to it.  
如果我们现在进入播放模式，预制件的单个实例将在世界原点产生。它的名称与预制件的名称相同，并附在其后面。

![](<images/1686829715971.png>)

Instantiated prefab, looking down the Z axis in the scene window.  
实例化预制，在场景窗口中沿Z轴向下看。

### Can you have the scene window open while in play mode?  
你能在播放模式下打开场景窗口吗？

Yes, but Unity always forces the game window to the foreground when play mode is entered. If the game window shares a panel with a scene window then that scene window will be hidden. But you can switch back to the scene window while still in play mode. Also, you can configure the editor layout so one or more game and scene windows are visible at the same time. Keep in mind that Unity has to render all these windows, so the more you have open the slower things get.  
是的，但当进入游戏模式时，Unity总是强制将游戏窗口移到前台。如果游戏窗口与场景窗口共享一个面板，则该场景窗口将被隐藏。但是，您可以在播放模式下切换回场景窗口。此外，您可以配置编辑器布局，使一个或多个游戏和场景窗口同时可见。请记住，Unity必须渲染所有这些窗口，所以你打开的越多，事情就越慢。

To place the point somewhere else we need to adjust the position of the instance. The `[Instantiate](http://docs.unity3d.com/Documentation/ScriptReference/Object.Instantiate.html)` method gives us a reference to whatever it created. Because we gave it a reference to a `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component, that's what we get in return. Let's keep track of it with a variable.  
要将点放置在其他位置，我们需要调整实例的位置。 `[Instantiate](http://docs.unity3d.com/Documentation/ScriptReference/Object.Instantiate.html)` 方法为我们提供了对它所创建的任何内容的引用。因为我们给了它一个 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件的引用，这就是我们得到的回报。让我们用一个变量来跟踪它。

```
void Awake () {
		Transform point = Instantiate(pointPrefab);
	}
```

In the [previous tutorial](https://catlikecoding.com/unity/tutorials/basics/game-objects-and-scripts/) we rotated the clock arms by assigned a quaternion to the `localRotation` property of the pivot's `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)`. Changing the position works the same way, except that we have to assign a 3D vector to the `localPosition` property instead.  
在上一个教程中，我们通过将四元数指定给枢轴的 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 的 `localRotation` 属性来旋转时钟臂。更改位置的方式相同，只是我们必须将3D矢量指定给 `localPosition` 属性。

3D vectors are created with the `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` struct type. For example, let's set the X coordinate of our point to 1, leaving its Y and Z coordinates at zero. `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` has a `right` property that gives us such a vector. Use it to set the point's position.  
3D矢量是使用 `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` 结构类型创建的。例如，让我们将点的X坐标设置为1，使其Y和Z坐标为零 `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` 有一个#2属性，它给了我们这样一个向量。使用它可以设置点的位置。

```
Transform point = Instantiate(pointPrefab);
		point.localPosition = Vector3.right;
```

![](<images/1686829716568.png>)

Cube one unit to the right.  
将一个单元向右立方排列。

When entering play mode now we still get one cube, just at a slightly different position. Let's instantiate a second one and place it an additional step to the right. This can be done by multiplying the right vector by 2. Repeat the instantiation and positioning, then add the multiplication to the new code.  
现在进入播放模式时，我们仍然会得到一个立方体，只是位置略有不同。让我们实例化第二个，并将其放置在右侧的附加步骤中。这可以通过将右向量乘以2来实现。重复实例化和定位，然后将乘法添加到新代码中。

```
void Awake () {
		Transform point = Instantiate(pointPrefab);
		point.localPosition = Vector3.right;

		Transform point = Instantiate(pointPrefab);
		point.localPosition = Vector3.right * 2f;
	}
```

### Can we multiply structs and numbers?  
我们能把结构和数字相乘吗？

Normally you cannot, but it is possible to define such functionality. This is done by creating a method with a special syntax, so it can be invoked as if it were a multiplication. In this case, what appears to be a simple multiplication is actually a method invocation, something like `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).Multiply([Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).right, 2f)` The result is a vector equal to the `right` vector with all its components doubled.  
通常情况下不能，但可以定义这样的功能。这是通过创建一个具有特殊语法的方法来完成的，因此可以像调用乘法一样调用它。在这种情况下，看似简单的乘法实际上是一个方法调用，类似于 `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).Multiply([Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).right, 2f)` 。结果是一个向量等于 `right` 向量，其所有分量都加倍。

Being able to use methods as if they were simple operations makes writing code faster and easier to read. It is not essential, but nice to have, just like being able to implicitly use namespaces. Such convenient syntax is known as syntactic sugar.  
能够像使用简单操作一样使用方法，可以使编写代码更快、更容易阅读。它不是必不可少的，但很好，就像能够隐式使用名称空间一样。这种方便的语法被称为句法糖。

Having said that, methods should only be used as operators if they strictly match the original meaning of that operator. In the case of vectors some mathematical operators are well-defined, so it's fine for those.  
话虽如此，只有当方法与运算符的原意严格匹配时，才应将其用作运算符。在向量的情况下，一些数学运算符是定义良好的，所以对它们来说很好。

This code will produce a compiler error, because we attempt to define the `point` variable twice. If we want to use another variable we have to give it a different name. Alternatively, we reuse the variable that we already have. We don't need to hold on to a reference to the first point once we're done with it, so assign the new point to the same variable.  
此代码将产生编译器错误，因为我们试图定义 `point` 变量两次。如果我们想使用另一个变量，我们必须给它一个不同的名称。或者，我们重用已经存在的变量。完成后，我们不需要保留对第一个点的引用，所以将新点分配给同一个变量。

```
Transform point = Instantiate(pointPrefab);
		point.localPosition = Vector3.right;

//		Transform point = Instantiate(pointPrefab);
		point = Instantiate(pointPrefab);
		point.localPosition = Vector3.right * 2f;
```

![](<images/1686829717242.png>)

Two instances, with X coordinates 1 and 2.  
两个实例，X坐标为1和2。

### Code Loops 代码循环

Let's create more points, until we have ten. We could repeat the same code eight more times, but that would be very inefficient programming. Ideally, we only write the code for one point and instruct the program to execute it multiple times, with slight variation.  
让我们创造更多的点，直到我们有十个。我们可以再重复八次相同的代码，但这将是非常低效的编程。理想情况下，我们只为一点编写代码，并指示程序多次执行，略有变化。

The `**while**` statement can be used to cause a block of code to repeat. Apply it to the first two statements of our method and remove the other statements.  
`**while**` 语句可用于使代码块重复。将它应用于我们方法的前两个语句，并删除其他语句。

```
void Awake () {
		while {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right;
		}
//		point = Instantiate(pointPrefab);
//		point.localPosition = Vector3.right * 2f;
	}
```

The `**while**` keyword must be followed by an expression within round brackets. The code block following `**while**` will only get executed if the expression evaluates as true. Afterwards, the program will loop back to the `**while**` statement. If at that point the expression again evaluates as true, the code block will be executed again. This repeats until the expression evaluates as false. Then the program skips the code block following the `**while**` statement and continues below it.  
`**while**` 关键字后面必须跟一个位于圆括号内的表达式。只有当表达式的计算结果为true时，才会执行 `**while**` 后面的代码块。然后，程序将循环回到#2语句。如果此时表达式再次计算为true，则将再次执行代码块。重复此操作，直到表达式的计算结果为false。然后程序跳过#3语句后面的代码块，并在它下面继续。

So we have to add an expression after `**while**`. We must be careful to make sure that the loop doesn't repeat forever. Infinite loops cause programs to get stuck, requiring manual termination by the user. The safest possible expression that compiles is simply `**false**`.  
所以我们必须在 `**while**` 之后添加一个表达式。我们必须小心，确保循环不会永远重复。无限循环会导致程序陷入停滞，需要用户手动终止。编译的最安全的表达式就是 `**false**` 。

```
while (false) {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right;
		}
```

### Can we define `point` inside the loop?  
我们可以在循环中定义 `point` 吗？

Yes. Although the code gets repeated, we've defined the variable only once. It gets reused each iteration of the loop, like we manually did earlier.  
对尽管代码会重复，但我们只定义了一次变量。它在循环的每次迭代中都会被重用，就像我们之前手动做的那样。

You could also define `point` before the loop. That allows you to use the variable outside the loop as well. Otherwise, its scope is limited to the block of the `**while**` loop.  
您也可以在循环之前定义 `point` 。这也允许您在循环之外使用变量。否则，其范围仅限于 `**while**` 循环的块。

Limiting the loop can be done by keeping track of how many times we've repeated the code. We can use an integer variable to keep track of this. It's type is `**int**`. It will contain the iteration number of the loop, so let's name it `i`. It's initial value is zero. To be able to use it in the `**while**` expression it must be defined above it.  
可以通过跟踪我们重复代码的次数来限制循环。我们可以使用一个整数变量来跟踪这一点。它的类型是 `**int**` 。它将包含循环的迭代次数，所以让我们将其命名为 `i` 。它的初始值为零。为了能够在#2表达式中使用它，必须在它上面定义它。

```
int i = 0;
		while (false) {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right;
		}
```

Each iteration, increase the number by one, by setting it to itself plus 1.  
每次迭代，通过将数字设置为自身加1，将数字增加一。

```
int i = 0;
		while (false) {
			i = i + 1;
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right;
		}
```

Now `i` becomes 1 at the start of the first iteration, 2 at the start of the second iteration, and so on. But the `**while**` expression is evaluated before each iteration. So right before the first iteration `i` is zero, it's 1 before the second, and so on. So after the tenth iteration `i` is ten. At this point we want to stop the loop, so its expression should evaluate as false. In other words, we should continue as long as `i` is less than ten. Mathematically, that's expressed as  
现在 `i` 在第一次迭代开始时变为1，在第二次迭代开始处变为2，依此类推。但是 `**while**` 表达式在每次迭代之前都会求值。所以在第一次迭代之前#2是0，在第二次迭代之前是1，依此类推。所以在第十次迭代之后#3是10。在这一点上，我们想要停止循环，所以它的表达式应该计算为false。换言之，只要 `i` 少于10，我们就应该继续。从数学上讲，这表示为i<10. It is written the same in code, with the `<` less-than operator.  
。它在代码中写得一样，使用 `<` 小于运算符。

```
int i = 0;
		while (i < 10) {
			i = i + 1;
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right;
		}
```

Now we'll get ten cubes after entering play mode. But they all end up at the same position. To put them in a row along the X axis multiply the `right` vector by `i`.  
现在，进入播放模式后，我们将获得十个立方体。但他们最终都处于相同的位置。若要将它们沿X轴排成一行，请将 `right` 向量乘以 `i` 。

```
point.localPosition = Vector3.right * i;
```

![](<images/1686829717838.png>)

Ten cubes in a row along the X axis.  
沿着X轴排成十个立方体。

Note that currently the first cube ends up with an X coordinate of 1 and the last cube ends up with 10. Let's change this so we begin at zero, positioning the first cube at the origin. We can shift all points one unit to the left by multiplying `right` by `(i - 1)` instead of `i`. However, we could skip that extra subtraction by increasing `i` at the end of the block, after the multiplication, instead of at the beginning.  
请注意，当前第一个立方体的X坐标为1，最后一个立方体的坐标为10。让我们改变这个，从零开始，将第一个立方体定位在原点。我们可以通过将 `right` 乘以 `(i - 1)` 而不是#2，将所有点向左移动一个单位。然而，我们可以跳过额外的减法，在块的末尾，在乘法之后，而不是在开始时增加#3。

```
while (i < 10) {
//			i = i + 1;
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right * i;
			i = i + 1;
		}
```

### Concise Syntax 简明语法

Because looping a certain amount of times is so common, it is convenient to keep the code for a loop concise. Some syntactic sugar can help us with that.  
因为循环一定次数是很常见的，所以保持循环的代码简洁是很方便的。一些句法糖可以帮助我们做到这一点。

First, let's consider incrementing the iteration number. When an operation of the form `x = x * y` is performed, it can be shortened to `x *= y`. This works for all operators that act on two operands.  
首先，让我们考虑递增迭代次数。当执行形式 `x = x * y` 的操作时，可以将其缩短为 `x *= y` 。这适用于作用于两个操作数的所有运算符。

```
//			i = i + 1;
			i += 1;
```

Going even further, when incrementing or decrementing a number by 1, this can be shortened to `++x` or `--x`.  
更进一步，当将一个数字递增或递减1时，可以将其缩短为 `++x` 或 `--x` 。

```
//			i += 1;
			++i;
```

One property of assignment statements is that they can also be used as expressions. This means that you could write something like `y = (x += 3)`. That would increase `x` by three and assign the result of that to `y` as well. This suggests that we could increment `i` inside the `**while**` expression, shortening the code block.  
赋值语句的一个特性是它们也可以用作表达式。这意味着您可以编写类似 `y = (x += 3)` 的内容。这将使 `x` 增加三，并将其结果分配给 `y` 。这表明我们可以在#4表达式中增加#3，从而缩短代码块。

```
while (++i < 10) {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right * i;
//			++i;
		}
```

However, now we're incrementing `i` before the comparison, instead of afterwards, which would lead to one less iteration. Specifically for situations like this, the increment and decrement operators can also be placed after a variable, instead of before it. The result of that expression is the original value, before it was changed.  
然而，现在我们在比较之前而不是之后递增 `i` ，这将导致少一次迭代。特别是在这种情况下，递增和递减运算符也可以放在变量之后，而不是放在变量之前。该表达式的结果是更改之前的原始值。

```
//		while (++i < 10) {
		while (i++ < 10) {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right * i;
		}
```

Although the `**while**` statement works for all kinds of loops, there is an alternative syntax particularly suited for iterating over ranges. It is the `**for**` loop. It works like `**while**`, except that both the iterator variable declaration and its comparison are contained within round brackets, separated by a semicolon.  
尽管 `**while**` 语句适用于所有类型的循环，但有一种替代语法特别适合在范围内迭代。这是 `**for**` 循环。它的工作原理与#2类似，只是迭代器变量声明及其比较都包含在圆括号中，用分号分隔。

```
//		int i = 0;
//		while (i++ < 10) {
		for (int i = 0; i++ < 10) {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right * i;
		}
```

That would produce a compiler error, because there is also a third part for incrementing the iterator, after another semicolon, keeping it separate from the comparison. This part gets performed at the end of each iteration.  
这将产生编译器错误，因为在另一个分号之后还有第三部分用于递增迭代器，使其与比较分离。该部分在每次迭代结束时执行。

```
//		for (int i = 0; i++ < 10) {
		for (int i = 0; i < 10; i++) {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right * i;
		}
```

### Why use `i++` and not `++i` in the `**for**` loop?  
为什么在#2循环中使用 `i++` 而不使用 `++i` ？

As the increment expression is not used for anything else, it doesn't matter which version we use. We could've also used `i += 1` or `i = i + 1`.  
由于增量表达式不用于其他任何内容，所以我们使用哪个版本并不重要。我们也可以使用 `i += 1` 或 `i = i + 1` 。

The classical for loop has the form `**for** (**int** i = 0; i < someLimit; i++)`. You will encounter that code fragment in a lot of programs and scripts.  
经典的for循环的形式为 `**for** (**int** i = 0; i < someLimit; i++)` 。您将在许多程序和脚本中遇到该代码片段。

### Changing the Domain 更改域

Currently, our points are given X coordinates 0 through 9. This isn't a convenient range when working with functions. Often, a range of 0–1 is used for X. Or when working with functions that are centered around zero, a range of −1–1. Let's reposition our points accordingly.  
目前，我们的点的X坐标为0到9。使用函数时，这不是一个方便的范围。通常，0–1的范围用于X。或者，当使用以零为中心的函数时，范围为−1–1。让我们相应地重新定位我们的观点。

Positioning ten cubes along a line segment two units long will cause them to overlap. To prevent this, we're going to reduce their scale. Each cube has size 1 in each dimension by default, so to make them fit we have to reduce their scale to  
沿着两个单位长的线段放置十个立方体将导致它们重叠。为了防止这种情况，我们将减少他们的规模。默认情况下，每个多维数据集的每个维度的大小都为1，因此为了使其适合，我们必须将其比例缩小到210=15. We can do this by setting each point's local scale to the `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).one` property divided by five. Division is done with the `/` slash operator.  
。我们可以通过将每个点的局部比例设置为 `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).one` 属性除以五来实现这一点。除法是用 `/` 斜杠运算符完成的。

```
for (int i = 0; i < 10; i++) {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right * i;
			point.localScale = Vector3.one / 5f;
		}
```

You can get a better view of the relative position of the cubes by switching the scene window to orthographic projection, which ignores perspective. Clicking the label under the axis widget at the top right of the scene window toggles between orthographic and perspective mode. The white cubes are also easier to see if you turn off the skybox via the scene window toolbar.  
通过将场景窗口切换到正交投影（忽略透视），可以更好地查看立方体的相对位置。单击场景窗口右上角的轴小部件下的标签可在正交模式和透视模式之间切换。如果通过场景窗口工具栏关闭skybox，白色立方体也更容易看到。

![](<images/1686829718372.png>)

Small cubes, seen in orthographic scene window without skybox.  
小立方体，在没有skybox的正交场景窗口中可以看到。

To bring the cubes back together again, divide their positions by five as well.  
要将立方体重新组合在一起，也可以将它们的位置除以五。

```
point.localPosition = Vector3.right * i / 5f;
```

This makes them cover the 0–2 range. To turn that into the −1–1 range, subtract 1 before scaling the vector. Use round brackets to indicate the operation order of the math expression.  
这使得它们覆盖了0–2的范围。要将其转换为−1–1范围，请在缩放矢量之前减去1。使用圆括号表示数学表达式的运算顺序。

```
point.localPosition = Vector3.right * (i / 5f - 1f);
```

![](<images/1686829718938.png>)

From −1 to 0.8.  
从−1到0.8。

Now the first cube has X coordinate −1, while the last has X coordinate 0.8. However, the cube size is 0.2. As the cube is centered on its position, the left side of the first cube is at −1.1, while the right side of the last cube is at 0.9. To neatly fill the −1–1 range with our cubes we have to shift them half a cube to the right. This can be done by adding 0.5 to `i` before dividing it.  
现在，第一个立方体的X坐标为−1，而最后一个立方体的X坐标为0.8。但是，立方体大小为0.2。由于立方体以其位置为中心，第一个立方体的左侧位于-1.1，而最后一个立方体的右侧位于0.9。为了用我们的立方体整齐地填充−1–1范围，我们必须将它们向右移动半个立方体。这可以通过在对 `i` 进行除法之前将0.5添加到 `i` 来实现。

```
point.localPosition = Vector3.right * ((i + 0.5f) / 5f - 1f);
```

![](<images/1686829719481.png>)

Filling the −1–1 range.  
填充−1–1范围。

### Hoisting the Vectors out of the Loop  
将矢量吊出回路

Although all the cubes have the same scale we calculate it again in every iteration of the loop. We don't have to do this, the scale is invariant. Instead, we could calculate it once before the loop, store it in a `scale` variable, and use that in the loop.  
尽管所有的立方体都有相同的尺度，但我们在循环的每次迭代中都会再次计算。我们不必这么做，规模是不变的。相反，我们可以在循环之前计算一次，将其存储在 `scale` 变量中，并在循环中使用。

```
void Awake () {
		var scale = Vector3.one / 5f;
		for (int i = 0; i < 10; i++) {
			Transform point = Instantiate(pointPrefab);
			point.localPosition = Vector3.right * ((i + 0.5f) / 5f - 1f);
			point.localScale = scale;
		}
	}
```

We could also define a variable for the position before the loop. As we're creating a line along the X axis, we only need to adjust the X coordinate of the position inside the loop. So we no longer have to multiply by `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).right`.  
我们还可以为循环之前的位置定义一个变量。当我们沿着X轴创建一条线时，我们只需要调整循环内位置的X坐标。因此，我们不再需要乘以 `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).right` 。

```
Vector3 position;
		var scale = Vector3.one / 5f;
		for (int i = 0; i < 10; i++) {
			Transform point = Instantiate(pointPrefab);
			//point.localPosition = Vector3.right * ((i + 0.5f) / 5f - 1f);
			position.x = (i + 0.5f) / 5f - 1f;
			point.localPosition = position;
			point.localScale = scale;
		}
```

### Can we change a vector's components individually?  
我们可以单独更改矢量的分量吗？

The `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` struct has three floating-point fields: `x`, `y`, and `z`. These fields are public, so we can change them.  
`[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` 结构有三个浮点字段： `x` 、 `y` 和 `z` 。这些字段是公开的，所以我们可以更改它们。

Because structs behave like simple values, the idea is that they should be immutable. Once constructed, they should't change. If you want to use a different value, assign a new struct to the field or variable, like we do with numbers. If we say that  
因为结构的行为类似于简单的值，所以其思想是它们应该是不可变的。一旦建成，它们就不应该改变。如果要使用不同的值，请像处理数字一样，为字段或变量分配一个新的结构。如果我们这么说x=3 and later that  
后来x=5, we've assigned a different number to  
，我们给分配了一个不同的号码x. We didn't modify the number 3 itself to become a 5. However, the vector types of Unity are mutable. This is done both for convenience and performance, because individual vector components are often manipulated independently.  
。我们没有将数字3本身修改为5。然而，Unity的向量类型是可变的。这样做是为了方便和性能，因为单独的矢量分量通常是独立操作的。

To get an idea of how to work with mutable vectors, you can consider the use of `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` a convenient substitute for using three separate `**float**` values. You can access them independently, yet also copy and assign them as a group.  
为了了解如何使用可变向量，可以考虑使用 `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` 来方便地替代使用三个单独的 `**float**` 值。您可以独立访问它们，也可以将它们作为一个组进行复制和分配。

This will result in a compiler error, complaining about the use of an unassigned variable. This happens because we're assigning `position` to something while we haven't set its Y and Z coordinates yet. We can fix this by initially setting `position` to a zero vector, by assigning `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).zero` to it.  
这将导致编译器错误，抱怨使用了未分配的变量。之所以会发生这种情况，是因为我们正在将 `position` 分配给某个对象，而我们尚未设置其Y和Z坐标。我们可以通过最初将 `position` 设置为零向量，并将 `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html).zero` 分配给它来解决这个问题。

```
//Vector3 position;
		var position = Vector3.zero;
		var scale = Vector3.one / 5f;
```

### Using X to Define Y  
使用X定义Y

The idea is that the positions of our cubes are defined as  
这个想法是，我们的立方体的位置被定义为⎡⎢⎣xf(x)0⎤⎥⎦, so we can use them to display a function. At this point the Y coordinates are always zero, which represents the trivial function  
，所以我们可以使用它们来显示一个函数。此时，Y坐标始终为零，这表示平凡函数f(x)=0. To show a different function we have to determine the Y coordinate inside the loop, instead of before it. Let's begin by making Y equal to X, representing the function  
.为了显示不同的函数，我们必须确定循环内部的Y坐标，而不是循环之前的坐标。让我们从使Y等于X开始，表示函数f(x)=x.

```
for (int i = 0; i < 10; i++) {
			Transform point = Instantiate(pointPrefab);
			position.x = (i + 0.5f) / 5f - 1f;
			position.y = position.x;
			point.localPosition = position;
			point.localScale = scale;
		}
```

![](<images/1686829720032.png>)

Y equals X. Y等于X。

A slightly less obvious function would be  
一个稍微不那么明显的功能是f(x)=x2, which defines a parabola with its minimum at zero.  
，定义了一个最小值为零的抛物线。

```
position.y = position.x * position.x;
```

![](<images/1686829720562.png>)

Y equals X squared.  
Y等于X的平方。

## Creating More Cubes 创建更多立方体

Although we have a functional graph at this point, it is ugly. Because we're only using ten cubes the suggested line looks very blocky and discrete. It would look better if we used more and smaller cubes.  
尽管我们在这一点上有一个函数图，但它是丑陋的。因为我们只使用了十个立方体，所以建议的线看起来非常块状和离散。如果我们用更多更小的立方体会看起来更好。

### Variable Resolution 可变分辨率

Instead of using a fixed amount of cubes we can make it configurable. To make this possible add a serializable integer field for the resolution to `**Graph**`. Give it a default of 10, which is what we're using now.  
我们可以配置它，而不是使用固定数量的多维数据集。为了实现这一点，请为 `**Graph**` 添加一个可序列化的整数字段。默认值为10，这就是我们现在使用的值。

```
[SerializeField]
	Transform pointPrefab;

	[SerializeField]
	int resolution = 10;
```

![](<images/1686829721114.png>)

Configurable resolution.  
可配置的分辨率。

Now we can adjust the graph's resolution by changing it via the inspector. However, not all integers are valid resolutions. At minimum they have to be positive. We can instruct the inspector to enforce a range for our resolution. This is done by attaching the `[Range](http://docs.unity3d.com/Documentation/ScriptReference/RangeAttribute.html)` attribute to it. We could either put both attributes of `resolution` between their own square brackets or combine then in a single comma-separated attribute list. Let's do the latter.  
现在，我们可以通过检查器更改图形的分辨率来调整它。但是，并非所有整数都是有效的分辨率。至少他们必须是积极的。我们可以指示检查员强制执行我们决议的范围。这是通过将 `[Range](http://docs.unity3d.com/Documentation/ScriptReference/RangeAttribute.html)` 属性附加到它上来完成的。我们可以将 `resolution` 的两个属性放在它们自己的方括号之间，也可以将它们组合在一个逗号分隔的属性列表中。让我们做后者。

```
[SerializeField, Range]
	int resolution = 10;
```

The inspector checks whether a field has a `[Range](http://docs.unity3d.com/Documentation/ScriptReference/RangeAttribute.html)` attribute attached to it. If so, it will constrain the value and also show a slider. However, to do this it needs to know the allowed range. So `[Range](http://docs.unity3d.com/Documentation/ScriptReference/RangeAttribute.html)` requires two arguments—like a method—for the minimum and maximum value. Let's use 10 and 100.  
检查器检查字段是否附加了 `[Range](http://docs.unity3d.com/Documentation/ScriptReference/RangeAttribute.html)` 属性。如果是，它将约束值并显示滑块。然而，要做到这一点，它需要知道允许的范围。所以 `[Range](http://docs.unity3d.com/Documentation/ScriptReference/RangeAttribute.html)` 需要两个参数，比如最小值和最大值的方法。让我们用10和100。

```
[SerializeField, Range(10, 100)]
	int resolution = 10;
```

![](<images/1686829721655.png>)

Resolution slider set to 50.  
分辨率滑块设置为50。

### Does this guarantee that `resolution` is constrained to 10–100?  
这是否保证 `resolution` 被限制在10–100之间？

All the `[Range](http://docs.unity3d.com/Documentation/ScriptReference/RangeAttribute.html)` attribute does is instruct the inspector to use a slider with that range. It doesn't affect `resolution` in any other way. So we could write code to assign it a value that is out of range, but we won't do that.  
`[Range](http://docs.unity3d.com/Documentation/ScriptReference/RangeAttribute.html)` 属性所做的只是指示检查器使用具有该范围的滑块。它不会以任何其他方式影响#1。所以我们可以编写代码给它分配一个超出范围的值，但我们不会这么做。

### Variable Instantiation 变量实例化

To make use of the configured resolution we have to change how many cubes we instantiate. Instead of looping a fixed amount of times in `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)`, the amount of iterations is now constrained by `resolution` instead of always `10`. So if the resolution is set to 50 we'll get 50 cubes after entering play mode.  
为了使用配置的分辨率，我们必须更改实例化的多维数据集的数量。在 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 中循环固定次数，迭代次数现在由 `resolution` 约束，而不是始终由 `10` 约束。因此，如果分辨率设置为50，我们将在进入播放模式后获得50个立方体。

```
for (int i = 0; i < resolution; i++) {
			…
		}
```

### What does `…` mean? `…` 是什么意思？

It's an indication that I omitted some code that didn't change.  
这表明我省略了一些没有更改的代码。

We also have to adjust the scale and positions of the cubes to keep them inside the −1–1 domain. The size of each step that we have to make per iteration is now two divided by the resolution. Store this value in a variable and use it to calculate the scale of the cubes and their X coordinates.  
我们还必须调整立方体的比例和位置，使其保持在−1–1域内。我们每次迭代必须执行的每个步骤的大小现在是分辨率除以2。将此值存储在变量中，并使用它来计算立方体的比例及其X坐标。

```
float step = 2f / resolution;
		var position = Vector3.zero;
		var scale = Vector3.one * step;
		for (int i = 0; i < resolution; i++) {
			Transform point = Instantiate(pointPrefab);
			position.x = (i + 0.5f) * step - 1f;
			…
		}
```

![](<images/1686829722191.png>)

Using resolution 50. 使用决议50。

### Setting the Parent 设置父项

After entering play mode with resolution 50 a lot of instantiated cubes show up in the scene, and thus also in the project window.  
在以分辨率50进入播放模式后，许多实例化的立方体出现在场景中，因此也出现在项目窗口中。

![](<images/1686829722726.png>)

Points are root objects.  
点是根对象。

These points are currently root objects, but it makes sense for them to be children of the graph object. We can set up this relationship after instantiating a point, by invoking the `SetParent` method of its `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component, passing it the desired parent `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)`. We can get the graph object's `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` component via the `transform` property of `**Graph**`, which it inherited from `[Component](http://docs.unity3d.com/Documentation/ScriptReference/Component.html)`. Do this at the end of the loop's block.  
这些点当前是根对象，但它们是图形对象的子对象是有意义的。我们可以在实例化一个点后建立这种关系，方法是调用其 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件的 `SetParent` 方法，并将所需的父#2传递给它。我们可以通过 `**Graph**` 的 `transform` 属性获得图形对象的 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 组件，它继承自 `[Component](http://docs.unity3d.com/Documentation/ScriptReference/Component.html)` 。在循环块的末尾执行此操作。

```
for (int i = 0; i < resolution; i++) {
			…
			point.SetParent(transform);
		}
```

![](<images/1686829723257.png>)

Points are children of the graph.  
点是图的子项。

When a new parent is set Unity will attempt to keep the object at its original world position, rotation, and scale. We don't need this in our case. We can signal this by passing `**false**` as a second argument to `SetParent`.  
当设置新的父对象时，Unity将尝试将对象保持在其原始世界位置、旋转和缩放。我们的案子不需要这个。我们可以通过将 `**false**` 作为第二个参数传递给 `SetParent` 来表示这一点。

```
point.SetParent(transform, false);
```

## Coloring the Graph 为图形着色

A white graph isn't pretty to look at. We could use another solid color, but that isn't very interesting either. It's more interesting to use a point's position to determine its color.  
白色的图形看起来不好看。我们可以用另一种纯色，但这也不是很有趣。使用点的位置来确定其颜色更有趣。

A straightforward way to adjust the color of each cube would be to set the color property of its material. We can do this in the loop. As each cube will get a different color, this means that we would end up with one unique material instance per object. And when we animate the graph later we'd have to adjust these materials all the time as well. While this works, it isn't very efficient. It would be much better if we could use a single material that directly uses the position as its color. Unfortunately, Unity doesn't have such a material. So let's make our own.  
调整每个立方体颜色的一种简单方法是设置其材质的颜色特性。我们可以在循环中做到这一点。由于每个立方体将获得不同的颜色，这意味着我们最终会为每个对象提供一个唯一的材质实例。当我们稍后为图形设置动画时，我们也必须始终调整这些材质。虽然这是有效的，但效率不是很高。如果我们可以使用一种直接使用位置作为颜色的单一材料，那会更好。不幸的是，Unity没有这样的材料。所以让我们自己做吧。

### Creating a Surface Shader  
创建曲面着色器

The GPU runs shader programs to render 3D objects. Unity's material assets determine which shader is used and allows its properties to be configured. We need to create a custom shader to get the functionality that we want. Create one via _Assets / Create / Shader / Standard Surface Shader_ and name it _Point Surface_.  
GPU运行着色器程序来渲染3D对象。Unity的材质资源决定使用哪个着色器，并允许配置其属性。我们需要创建一个自定义着色器来获得我们想要的功能。创建一个via并命名。

![](<images/1686829723793.png>)

  

![](<images/1686829724388.png>)

Shader grouped with prefab in Point folder, one and two column layout.  
着色器分组，带有点文件夹中的预制，一列和两列布局。

We now have a shader asset, which you can open like a script. Our shader file contains code to define a surface shader, which uses different syntax than C#. It contains a surface shader template, but we'll delete everything and start from scratch to create a minimal shader.  
我们现在有了一个着色器资源，您可以像脚本一样打开它。我们的着色器文件包含定义曲面着色器的代码，该着色器使用与C#不同的语法。它包含一个曲面着色器模板，但我们将删除所有内容，并从头开始创建一个最小的着色器。

### How do surface shaders work?  
曲面着色器是如何工作的？

Unity provides a framework to quickly generate shaders that perform default lighting calculations, which you can influence by adjusting certain values. Such shaders are known as surface shaders. Unfortunately they only work for the default render pipeline. We'll cover the Universal render pipeline later.  
Unity提供了一个框架来快速生成执行默认照明计算的着色器，您可以通过调整某些值来影响这些计算。此类着色器称为曲面着色器。不幸的是，它们只适用于默认的渲染管道。我们稍后将介绍通用渲染管道。

Unity has its own syntax for shader assets, which is overall roughly like C# but it's a mix of different languages. It begins with the `**Shader**` keyword followed by a string defining a menu item for the shader. Strings are written inside double quotes. We'll use _Graph/Point Surface_. After that comes a code block for the shader's contents.  
Unity有自己的着色器资源语法，总体上大致类似于C#，但它是不同语言的混合。它以 `**Shader**` 关键字开头，后跟一个定义着色器菜单项的字符串。字符串写在双引号内。我们将使用。之后是着色器内容的代码块。

```
Shader "Graph/Point Surface" {}
```

Shaders can have multiple sub-shaders, each defined by the `**SubShader**` keyword followed by a code block. We only need one.  
着色器可以有多个子着色器，每个子着色器由 `**SubShader**` 关键字和一个代码块定义。我们只需要一个。

```
Shader "Graph/Point Surface" {

	SubShader {}
}
```

Below the sub-shader we also want to add a fallback to the standard diffuse shader, by writing `**FallBack** "Diffuse"`.  
在子着色器下面，我们还想通过写入 `**FallBack** "Diffuse"` 来向标准漫反射着色器添加回退。

```
Shader "Graph/Point Surface" {
	
	SubShader {}
	
	FallBack "Diffuse"
}
```

The sub-shader of a surface shader needs a code section written in a hybrid of CG and HLSL, two shader languages. This code must be enclosed by the `CGPROGRAM` and `ENDCG` keywords.  
曲面着色器的子着色器需要用CG和HLSL这两种着色器语言混合编写的代码部分。此代码必须用 `CGPROGRAM` 和 `ENDCG` 关键字括起来。

```
SubShader {
		CGPROGRAM
		ENDCG
	}
```

The first needed statement is a compiler directive, known as a pragma. It's written as `#pragma` followed by a directive. In this case we need `#pragma surface ConfigureSurface Standard fullforwardshadows`, which instructs the shader compiler to generate a surface shader with standard lighting and full support for shadows. `ConfigureSurface` refers to a method used to configure the shader, which we'll have to create.  
第一个需要的语句是编译器指令，称为杂注。它被写成 `#pragma` ，后面跟着一个指令。在这种情况下，我们需要 `#pragma surface ConfigureSurface Standard fullforwardshadows` ，它指示着色器编译器生成一个具有标准照明和完全支持阴影的曲面着色器 `ConfigureSurface` 指的是一种用于配置着色器的方法，我们必须创建它。

```
CGPROGRAM
		#pragma surface ConfigureSurface Standard fullforwardshadows
		ENDCG
```

### What does pragma mean?  
实用主义是什么意思？

The word pragma comes from Greek and refers to an action, or something that needs to be done. It's used in many programming languages to issue special compiler directives.  
pragma一词来自希腊语，指的是一种行动或需要做的事情。它在许多编程语言中被用来发布特殊的编译器指令。

We follow that with the `#pragma target 3.0` directive, which sets a minimum for the shader's target level and quality.  
我们遵循 `#pragma target 3.0` 指令，该指令为着色器的目标级别和质量设置了最小值。

```
CGPROGRAM
		#pragma surface ConfigureSurface Standard fullforwardshadows
		#pragma target 3.0
		ENDCG
```

We're going to color our points based on their world position. To make this work in a surface shader we have to define the input structure for our configuration function. It has to be written as `**struct** **Input**` followed by a code block and then a semicolon. Inside the block we declare a single struct field, specifically `**float3** worldPos`. It will contain the world position of what gets rendered. The `**float3**` type is the shader equivalent of the `[Vector3](http://docs.unity3d.com/Documentation/ScriptReference/Vector3.html)` struct.  
我们将根据他们在世界上的位置来给我们的分数上色。为了在曲面着色器中实现这一点，我们必须为配置函数定义输入结构。它必须写成 `**struct** **Input**` ，后面跟着一个代码块，然后是一个分号。在块内部，我们声明一个结构字段，特别是 `**float3** worldPos` 。它将包含渲染内容的世界位置。#2类型是等价于#3结构的着色器。

```
CGPROGRAM
		#pragma surface ConfigureSurface Standard fullforwardshadows
		#pragma target 3.0

		struct Input {
			float3 worldPos;
		};
		ENDCG
```

### Does this mean that moving the graph would affect its color?  
这是否意味着移动图形会影响其颜色？

Yes. With this approach the coloring will only be correct as long as we leave the _Graph_ object where it is: at the world origin, with no rotation, and scale 1.  
对使用这种方法，只有当我们将对象留在原地时，着色才会正确：在世界原点，没有旋转，比例为1。

Also note that this position is determined per vertex. In our case, that is for each corner of a cube. The color will be interpolated across the cube's faces. The larger the cubes are, the more obvious this color transition will be.  
还要注意，此位置是按顶点确定的。在我们的例子中，这是针对立方体的每个角。颜色将在立方体的面上进行插值。立方体越大，这种颜色过渡就越明显。

Below that we define our `ConfigureSurface` method, although in the case of shaders it's always referred to as a function, not as a method. It is a `**void**` function with two parameters. First is an input parameter that has the `**Input**` type that we just defined. The second parameter is the surface configuration data, with the type `**SurfaceOutputStandard**`.  
下面我们定义了 `ConfigureSurface` 方法，尽管在着色器的情况下，它总是被称为函数，而不是方法。它是一个带有两个参数的 `**void**` 函数。首先是一个输入参数，它具有我们刚刚定义的#2类型。第二个参数是曲面配置数据，类型为 `**SurfaceOutputStandard**` 。

```
struct Input {
			float3 worldPos;
		};

		void ConfigureSurface (Input input, SurfaceOutputStandard surface) {}
```

The second parameter must have the `**inout**` keyword written in front of its type, which indicates that it's both passed to the function and used for the result of the function.  
第二个参数的类型前面必须写有 `**inout**` 关键字，这表明它既被传递给函数，又被用于函数的结果。

```
void ConfigureSurface (Input input, inout SurfaceOutputStandard surface) {}
```

Now that we have a functioning shader create a material for it, named _Point Surface_. Set it to use our shader, by selecting _Graph / Point Surface_ via the _Shader_ dropdown list in the header of its inspector.  
现在我们有了一个正常工作的着色器，为其创建一个材质，名为。通过其检查器标题中的下拉列表进行选择，将其设置为使用我们的着色器。

![](<images/1686829725045.png>)

Point surface material. 点曲面材质。

The material is currently solid matte black. We can make it look more like the default material by setting `surface.Smoothness` to 0.5 in our configuration function. When writing shader code we do not have to add the f suffix to `**float**` values.  
该材质当前为实心无光黑色。我们可以通过在配置功能中将 `surface.Smoothness` 设置为0.5，使其看起来更像默认材质。在编写着色器代码时，我们不必将f后缀添加到 `**float**` 值。

```
void ConfigureSurface (Input input, inout SurfaceOutputStandard surface) {
			surface.Smoothness = 0.5;
		}
```

Now the material is no longer perfectly matte. You can see this in the small material preview in the inspector's header, or in the resizable preview at its bottom.  
现在，材质不再是完全无光的。您可以在检查器标题中的小型材质预览或底部可调整大小的预览中看到这一点。

![](<images/1686829725605.png>)

Material preview with average smoothness.  
具有平均平滑度的材质预览。

We can also make smoothness configurable, as if adding a field for it and using that in the function. The default style is to prefix shader configuration options with an underscore and capitalize the next letter, so we'll use `_Smoothness`.  
我们还可以配置平滑度，就好像为它添加一个字段并在函数中使用它一样。默认样式是在着色器配置选项前面加下划线，并将下一个字母大写，因此我们将使用 `_Smoothness` 。

```
float _Smoothness;

		void ConfigureSurface (Input input, inout SurfaceOutputStandard surface) {
			surface.Smoothness = _Smoothness;
		}
```

To make this configuration option appear in the editor we have to add a `**Properties**` block at the top of the shader, above the sub-shader. Write `_Smoothness` in there, followed by `("Smoothness", **Range**(0,1)) = 0.5`. This gives it the _Smoothness_ label, exposes it as a slider with the 0–1 range, and sets its default to 0.5.  
要使此配置选项显示在编辑器中，我们必须在着色器顶部、子着色器上方添加 `**Properties**` 块。在那里写 `_Smoothness` ，然后写 `("Smoothness", **Range**(0,1)) = 0.5` 。这将为其提供标签，将其显示为0–1范围的滑块，并将其默认值设置为0.5。

```
Shader "Graph/Point Surface" {

	Properties {
		_Smoothness ("Smoothness", Range(0,1)) = 0.5
	}
	
	SubShader {
		…
	}
}
```

![](<images/1686829726200.png>)

Configurable smoothness.  
可配置的平滑度。

Make our _Cube_ prefab asset use this material instead of the default one. That will turn the points black.  
使我们的预制资产使用此材料，而不是默认材料。这将使要点变黑。

![](<images/1686829726754.png>)

Black points. 黑点。

### Coloring Based on World Position  
基于世界位置的着色

To adjust the color of our points we have to modify `surface.Albedo`. As both albedo and the world position have three components we can directly use the position for albedo.  
为了调整我们点的颜色，我们必须修改 `surface.Albedo` 。由于反照率和世界位置都有三个组成部分，我们可以直接使用反照率的位置。

```
void ConfigureSurface (Input input, inout SurfaceOutputStandard surface) {
			surface.Albedo = input.worldPos;
			surface.Smoothness = _Smoothness;
		}
```

![](<images/1686829727291.png>)

Colored points. 彩色点。

### What does albedo mean?  
反照率是什么意思？

Albedo means whiteness in Latin. It's a measure of how much light is diffusely reflected by a surface. If albedo isn't fully white then part of the light energy gets absorbed instead of reflected.  
Albedo在拉丁语中的意思是白色。这是一个表面漫反射光的度量。如果反照率不是完全白色的，那么部分光能会被吸收而不是反射。

Now the world X position controls the point's red color component, the Y position controls the green color component, and Z controls blue. But our graph's X domain is −1–1, and negative color components make no sense. So we have to halve the position and then add ½ to make the colors fit the domain. We can do this for all three dimension at once.  
现在，世界X位置控制点的红色分量，Y位置控制绿色分量，Z位置控制蓝色分量。但我们的图的X域是−1–1，负颜色分量没有意义。因此，我们必须将位置减半，然后添加½，以使颜色适合域。我们可以同时对所有三维进行此操作。

```
surface.Albedo = input.worldPos * 0.5 + 0.5;
```

To get a better sense of whether the colors are correct let's change `**Graph**.[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` so we display the function  
为了更好地了解颜色是否正确，让我们更改 `**Graph**.[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` ，以便显示函数f(x)=x3 which makes Y go from −1 to 1 as well.  
这使得Y也从−1变为1。

```
position.y = position.x * position.x * position.x;
```

![](<images/1686829727820.png>)

X cubed, bluish. X立方体，蓝色。

The result is bluish because all cube faces have Z coordinates close to zero, which sets their blue color component close to 0.5. We can eliminate blue by only including the red and green channels when setting the albedo. This can be done in shaders by only assigning to `surface.Albedo.**rg**` and only using `input.worldPos.**xy**`. That way the blue component stays zero.  
结果是蓝色的，因为所有立方体面的Z坐标都接近于零，这将使其蓝色分量接近于0.5。在设置反照率时，我们可以通过仅包括红色和绿色通道来消除蓝色。这可以在着色器中通过仅指定给 `surface.Albedo.**rg**` 和仅使用 `input.worldPos.**xy**` 来完成。这样，蓝色分量保持为零。

```
surface.Albedo.rg = input.worldPos.xy * 0.5 + 0.5;
```

As red plus green results in yellow this will make the points start near black at the bottom left, turn green as Y initially increases quicker than X, turn yellow as X catches up, turn slightly orange as X increases faster, and finally end near bright yellow at the top right.  
由于红色加上绿色导致黄色，这将使点从左下角的黑色附近开始，随着Y最初比X增加得更快而变为绿色，随着X赶上而变为黄色，随着X增加得快而变为轻微的橙色，最后在右上角的亮黄色附近结束。

![](<images/1686829728430.png>)

X cubed, from green to yellow.  
X立方，从绿色到黄色。

### Universal Render Pipeline  
通用渲染管道

Besides the default render pipeline Unity also has the Universal and High-Definition render pipelines, URP and HDRP for short. Both render pipelines have different features and limitations. The current default render pipeline is still functional, but its feature set is frozen. In a few years URP will likely become the default. So let's make our graph also work with URP.  
除了默认渲染管道外，Unity还拥有通用和高清渲染管道，简称URP和HDRP。两个渲染管道都有不同的功能和限制。当前默认渲染管道仍然可用，但其功能集已冻结。再过几年，URP可能会成为默认。因此，让我们让我们的图也与URP一起工作。

If you aren't using URP yet go to the package manager and install the latest _Universal RP_ package verified for your Unity version. In my case that's 10.4.0.  
如果您还没有使用URP，请转到软件包管理器，安装为您的Unity版本验证的最新软件包。就我而言，这是10.4.0。

![](<images/1686829728961.png>)

URP package installed. 已安装URP程序包。

### Where can I find URP in the package manager?  
在包管理器中的哪里可以找到URP？

Make sure that you've set the package filter to _Unity Registry_ and not _In Project_. Then search for _universal_ or scroll down the list until you find it.  
请确保已将包筛选器设置为，而不是。然后搜索或向下滚动列表，直到找到为止。

This doesn't automatically make Unity use the URP. We first have to create an asset for it, via _Assets / Create / Rendering / Universal Render Pipeline / Pipeline Asset (Forward Renderer)_. I named it _URP_. This will also automatically create another asset for a renderer, in my case named _URP_Renderer_.  
这不会自动使Unity使用URP。我们首先必须通过为它创建一个资产。我给它起了名字。这也将自动为渲染器创建另一个资源，在我的例子中名为。

![](<images/1686829729513.png>)

  

![](<images/1686829730043.png>)

URP assets in separate folder, one and two column layout.  
URP资产在单独的文件夹中，单列和单列布局。

Next, go to the _Graphics_ section of the project settings and assign the URP asset to the _Scriptable Renderer Pipeline Settings_ field.  
接下来，转到项目设置的部分，并将URP资源指定给字段。

![](<images/1686829730577.png>)

Using the URP. 使用URP。

To switch back to the default render pipeline later simply set _Scriptable Renderer Pipeline Settings_ to _None_. This can only be done in the editor, the render pipeline cannot be changed in a built stand-alone app.  
若要稍后切换回默认渲染管道，只需将设置为即可。这只能在编辑器中完成，渲染管道不能在构建的独立应用程序中更改。

### What about HDRP? HDRP呢？

HDRP is a much more complex render pipeline. I won't cover it in my tutorials.  
HDRP是一个复杂得多的渲染管道。我不会在教程中介绍它。

### Creating a Shader Graph  
创建着色器图形

Our current material only works with the default render pipeline, not URP. So when URP is used it is replaced with Unity's error material, which is solid magenta.  
我们当前的材质仅适用于默认渲染管道，而不适用于URP。因此，当使用URP时，它会被Unity的误差材料取代，该材料是实心品红色。

![](<images/1686829731120.png>)

Cubes have become magenta.  
立方体变成了品红色。

We have to create a separate shader for the URP. We could write one ourselves, but that's currently very hard and likely to break when upgrading to a newer URP version. The best approach is to use Unity's shader graph package to visually design a shader. URP depends on this package so it was automatically installed along with the URP package.  
我们必须为URP创建一个单独的着色器。我们可以自己写一个，但目前这很难，升级到新的URP版本时可能会中断。最好的方法是使用Unity的着色器图形包来可视化设计着色器。URP依赖于此程序包，因此它是与URP程序包一起自动安装的。

Create a new shader graph via _Assets / Create / Shader / Universal Render Pipeline / Lit Shader Graph_ and name it _Point URP_.  
通过创建新的着色器图形并命名。

![](<images/1686829731685.png>)

  

![](<images/1686829732252.png>)

Point URP shader graph asset, one and two column layout.  
点URP着色器图形资源，单列和单列布局。

The graph can be opened by double-clicking its asset in the project window or by pressing the _Open Shader Editor_ button in its inspector. This opens a shader graph window for it, which might be cluttered by multiple nodes and panels. These are the blackboard, graph inspector, and main preview panels, which can be resized and can also be hidden via toolbar buttons. There are also two linked nodes: a _Vertex_ node and a _Fragment_ node. These two are used to configure the output of the shader graph.  
通过双击项目窗口中的资源或按下其检查器中的按钮，可以打开图形。这将为其打开一个着色器图形窗口，该窗口可能被多个节点和面板弄得杂乱无章。这些是黑板、图形检查器和主预览面板，它们可以调整大小，也可以通过工具栏按钮隐藏。还有两个链接节点：“顶点”节点和“片段”节点。这两个用于配置着色器图形的输出。

![](<images/1686829732847.png>)

Default lit shader graph with everything visible.  
所有内容都可见的默认照明着色器图形。

A shader graph consists of nodes that represent data or operations. Currently, the _Smoothness_ value of the _Fragment_ node is set to 0.5. To make it a configurable shader property press the plus button on the _Point URP_ backboard panel, choose _Float_, and name the new entry _Smoothness_. That adds a rounded button to the blackboard that represents the property. Select it and switch the graph inspector to its _Node Settings_ tab to see the configuration of this property.  
着色器图形由表示数据或操作的节点组成。当前，“片段”节点的值设置为0.5。若要使其成为可配置的着色器属性，请按背板面板上的加号按钮，选择，然后将新条目命名为“平滑度”。这会在黑板上添加一个圆形按钮来表示属性。选择它并将图形检查器切换到其“节点设置”选项卡以查看此属性的配置。

![](<images/1686829733414.png>)

Smoothness property with default settings.  
具有默认设置的“平滑度”属性。

_Reference_ is the name by which the property is known internally. This corresponds to how we named the property field __Smoothness_ in our surface shader code, so let's use the same internal name here as well. Then set the default value below it to 0.5. Make sure that its _Exposed_ toggle option is enabled, as this controls whether materials gets a shader property for it. Finally, to make it appear as a slider change its _Mode_ to _Slider_.  
引用是内部已知属性的名称。这与我们在曲面着色器代码中命名特性字段_Smoothness的方式相对应，因此让我们在这里也使用相同的内部名称。然后将其下方的默认值设置为0.5。确保其切换选项已启用，因为这将控制材质是否为其获取着色器属性。最后，要使其显示为滑块，请将其更改为。

![](<images/1686829733968.png>)

Smoothness property configured.  
已配置“平滑度”属性。

Next, drag the rounded _Smoothness_ button from the blackboard onto an open space in the graph. That will add a smoothness node to the graph. Connect it to the _Smoothness_ input of the _PRB Master_ node by dragging from one of their dots to the other. This creates a link between them.  
接下来，将圆形按钮从黑板拖动到图形中的空白处。这将为图形添加一个平滑度节点。通过从其中一个点拖动到另一个点，将其连接到节点的输入。这就在它们之间建立了联系。

![](<images/1686829734562.png>)

Smoothness connected. 连接的平滑度。

Now you can save the graph via the _Save Asset_ toolbar button and create a material named _Point URP_ that uses it. The shader's menu item is _Shader Graphs / Point URP_. Then make the _Point_ prefab use that material instead of _Point Surface_.  
现在，您可以通过工具栏按钮保存图形，并创建一个名为的材质来使用它。着色器的菜单项为。然后使预制件使用这种材料而不是。

![](<images/1686829735094.png>)

Material for URP using our shader graph.  
使用我们的着色器图的URP材质。

### Programming with Nodes 使用节点编程

To color the points we have to start with a position node. Create one by opening a context menu on an empty part of the graph and choosing _New Node_ from it. Select _Input / Geometry / Position_ or just search for _Position_.  
为了给点上色，我们必须从一个位置节点开始。通过在图形的空白部分打开上下文菜单并从中进行选择来创建一个。选择或仅搜索。

![](<images/1686829735668.png>)

World position node. 世界位置节点。

We now have a position node, which is set to world space by default. You can collapse its preview visualization by pressing the upward arrow that appears when you hover the cursor over it.  
我们现在有一个位置节点，默认情况下设置为世界空间。通过将光标悬停在其上时显示的向上箭头，可以折叠其预览可视化效果。

Use the same approach to create a _Multiply_ and an _Add_ node. Use these to scale the position's XY components by 0.5 and then add 0.5, while setting Z to zero. These nodes adapt their input types depending on what they're connected to. So first connect the nodes and then fill in their constant inputs. Then connect the result to the _Base Color_ input of _Fragment_.  
使用相同的方法创建和节点。使用这些可以将位置的XY分量缩放0.5，然后添加0.5，同时将Z设置为零。这些节点根据它们所连接的内容来调整它们的输入类型。因此，首先连接节点，然后填充它们的常量输入。然后将结果连接到Fragment的输入。

![](<images/1686829736223.png>)

Colored shader graph. 着色着色器图形。

You can compact the visual size of the _Multiply_ and _Add_ nodes by pressing the arrow that appear in their top right corner if you hover over them. That hides all their inputs and outputs that aren't connected to anther node. This removes a lot of clutter. You can also delete components of the _Vertex_ and _Fragment_ nodes via their context menu. This way you can hide everything that keeps its default value.  
如果将鼠标悬停在“乘法”和“加法”节点上，则可以通过按下右上角出现的箭头来缩小它们的视觉大小。这隐藏了所有未连接到另一个节点的输入和输出。这样可以去除很多杂物。也可以通过“顶点”和“片段”节点的关联菜单删除它们的组件。通过这种方式，您可以隐藏所有保持其默认值的内容。

![](<images/1686829736789.png>)

Compacted shader graph. 压缩着色器图形。

After saving the shader asset we now get the same colored points in play mode that we got when using the default render pipeline. Besides that, a debug updater appears in a separate _DontDestroyOnLoad_ scene in play mode. This is for debugging URP and can be ignored.  
保存着色器资源后，我们现在可以在播放模式下获得与使用默认渲染管道时相同的颜色点。除此之外，在播放模式下，调试更新程序会出现在单独的场景中。这是为了调试URP，可以忽略。

![](<images/1686829737329.png>)

URP debug updater in play mode.  
URP调试更新程序处于播放模式。

From this point you can use either the default render pipeline or the URP. After switching from one to the other you'll also have to change the material of the _Point_ prefab otherwise it will be magenta. If you're curious about the shader code that is generated from the graph you can get to it via the _View Generated Shader_ button of the graph's inspector.  
从这一点开始，您可以使用默认渲染管道或URP。在从一个切换到另一个之后，您还必须更改预制件的材料，否则它将是品红色的。如果您对从图形生成的着色器代码感到好奇，可以通过图形检查器的按钮获取该代码。

## Animating the Graph 设置图形动画

Displaying a static graph is useful, but a moving graph is more interesting to look at. So let's add support for animating functions. This is done by including time as an additional function parameter, using functions of the form  
显示静态图形很有用，但移动图形更有趣。因此，让我们添加对动画函数的支持。这是通过使用以下形式的函数将时间作为附加函数参数来实现的f(x,t) instead of just  
而不是f(x), where  哪里t is the time. 时间到了。

### Keeping Track of the Points  
跟踪点

To animate the graph we'll have to adjust its points as time progresses. We could do this by deleting all points and creating new ones each update, but that's an inefficient way to do this. It's much better to keep using the same points, adjusting their positions each update. To make this possible we're going to use a field to keep a reference to our points. Add a `points` field to `**Graph**` of type `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)`.  
要设置图形的动画，我们必须随着时间的推移调整其点。我们可以通过删除所有点并在每次更新时创建新点来做到这一点，但这是一种效率低下的方法。最好保持使用相同的点，每次更新都调整它们的位置。为了实现这一点，我们将使用一个字段来保持对我们的点的引用。将 `points` 字段添加到类型为 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 的 `**Graph**` 中。

```
[SerializeField, Range(10, 100)]
	int resolution = 10;
	
	Transform points;
```

This field allows us to reference a single point, but we need access to all of them. We can turn our field into an array by putting empty square brackets behind its type.  
这个字段允许我们引用一个点，但我们需要访问所有这些点。我们可以通过在字段类型后面放上空的方括号来将字段转换为数组。

```
Transform[] points;
```

The `points` field is now a reference to an array, whose elements are of type `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)`. Arrays are objects, not simple values. We have to explicitly create such an object and make our field reference it. This is done by writing `**new**` followed by the array type, so `**new** [Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)[]` in our case. Create the array in `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)`, before our loop, and assign it to `points`.  
`points` 字段现在是对数组的引用，该数组的元素类型为 `[Transform](http://docs.unity3d.com/Documentation/ScriptReference/Transform.html)` 。数组是对象，而不是简单的值。我们必须显式地创建这样一个对象，并使我们的字段引用它。这是通过写#2后面跟着数组类型来完成的，所以在我们的例子中是#3。在循环之前的#4中创建数组，并将其分配给 `points` 。

```
points = new Transform[];
		for (int i = 0; i < resolution; i++) {
			…
		}
```

When creating an array we have to specify its length. This defines how many elements it has, which cannot be changed after it has been created. The length is written inside the square brackets when constructing the array. Make it equal to the resolution of the graph.  
创建数组时，我们必须指定其长度。这定义了它有多少元素，这些元素在创建后无法更改。构造数组时，长度写在方括号内。使其等于图形的分辨率。

```
points = new Transform[resolution];
```

Now we can fill the array with references to our points. Accessing an array element is done by writing its index between square brackets behind the array reference. Array indices start at zero for the first element, just like the iteration counter of our loop. So we can use that to assign to the appropriate array element.  
现在我们可以用对我们的点的引用来填充数组。访问数组元素是通过在数组引用后面的方括号之间写入其索引来完成的。第一个元素的数组索引从零开始，就像我们循环的迭代计数器一样。因此，我们可以使用它来分配给适当的数组元素。

```
points = new Transform[resolution];
		for (int i = 0; i < resolution; i++) {
			Transform point = Instantiate(pointPrefab);
			points[i] = point;
			…
		}
```

If we're assigning the same thing multiple times in a row, we can chain these assignments together because the result of an assignment expression is what was assigned, as explained in the previous tutorial.  
如果我们在一行中多次指定同一个对象，我们可以将这些指定链接在一起，因为指定表达式的结果就是指定的结果，如前一教程中所述。

```
Transform point = points[i] = Instantiate(pointPrefab);
			//points[i] = point;
```

We're now looping through our array of points. Because the array's length is the same as the resolution, we could also use that to constrain our loop. Each array has a `Length` property for this purpose, so let's use that.  
我们现在循环浏览我们的点数组。因为数组的长度与分辨率相同，所以我们也可以使用它来约束循环。每个数组都有一个用于此目的的 `Length` 属性，所以让我们使用它。

```
points = new Transform[resolution];
		for (int i = 0; i < points.Length; i++) {
			…
		}
```

### Updating the Points 更新积分

To adjust the graph each frame we need to set the Y coordinates of the points in an `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` method. So we no longer need to calculate them in `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)`. We can still set the X coordinates here because we won't change them.  
为了调整每个帧的图形，我们需要在 `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` 方法中设置点的Y坐标。因此，我们不再需要在 `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` 中计算它们。我们仍然可以在这里设置X坐标，因为我们不会更改它们。

```
for (int i = 0; i < points.Length; i++) {
			Transform point = points[i] = Instantiate(pointPrefab);
			position.x = (i + 0.5f) * step - 1f;
//			position.y = position.x * position.x * position.x;
			…
		}
```

Add an `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` method with a `**for**` loop just like `[Awake](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Awake.html)` has, but without any code in its block yet.  
添加一个带有 `**for**` 循环的 `[Update](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.Update.html)` 方法，就像#2一样，但其块中还没有任何代码。

```
void Awake () {
		…
	}
	
	void Update () {
		for (int i = 0; i < points.Length; i++) {}
	}
```

We'll begin each iteration of the loop by getting a reference to the current array element and storing it in a variable.  
我们将通过获取对当前数组元素的引用并将其存储在变量中来开始循环的每次迭代。

```
for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
		}
```

After that we retrieve the point's local position and store it in a variable as well.  
之后，我们检索点的局部位置，并将其存储在变量中。

```
for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
			Vector3 position = point.localPosition;
		}
```

Now we can set the position's Y coordinate, based on X, as we did earlier.  
现在我们可以根据X设置位置的Y坐标，就像前面所做的那样。

```
for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
			Vector3 position = point.localPosition;
			position.y = position.x * position.x * position.x;
		}
```

Because the position is a struct we only adjusted the local variable's value. To apply it to the point we have to set its position again.  
因为位置是一个结构，所以我们只调整了局部变量的值。要把它应用到这一点上，我们必须重新确定它的位置。

```
for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
			Vector3 position = point.localPosition;
			position.y = position.x * position.x * position.x;
			point.localPosition = position;
		}
```

### Couldn't we directly set `point.localPosition.y`?  
我们不能直接设置 `point.localPosition.y` 吗？

If `localPosition` were a public field then we could directly set the Y coordinate of the point's position. However, `localPosition` is a property. It passes a copy of the vector value to us, or copies what we assign to it. So we'd end up adjusting a local vector value, which doesn't affect the point's position at all. As we haven't explicitly stored it in a variable first, the operation would be meaningless and will produce a compiler error.  
如果 `localPosition` 是一个公共字段，那么我们可以直接设置点位置的Y坐标。但是， `localPosition` 是一个属性。它将向量值的副本传递给我们，或者复制我们分配给它的值。所以我们最终会调整局部向量值，这根本不会影响点的位置。由于我们没有先将其显式存储在变量中，因此该操作将毫无意义，并将产生编译器错误。

### Showing a Sine Wave  
显示正弦波

From now on, while in play mode, the points of our graph get positioned every frame. We don't notice this yet because they always end up at the same positions. We have to incorporate the time into the function in order to make it change. However, simply adding the time will cause the function to rise and quickly disappear out of view. To prevent this from happening we have to use a function that changes but remains within a fixed range. The sine function is ideal for this, so we'll use  
从现在起，当处于播放模式时，我们的图中的点会在每帧中定位。我们还没有注意到这一点，因为他们最终总是处于相同的位置。我们必须将时间纳入函数中，以便使其发生变化。然而，简单地添加时间会导致函数上升并迅速消失在视野之外。为了防止这种情况发生，我们必须使用一个变化但保持在固定范围内的函数。正弦函数是理想的，所以我们将使用f(x)=sin(x). We can use the `[Mathf](http://docs.unity3d.com/Documentation/ScriptReference/Mathf.html).Sin` method to compute it.  
。我们可以使用 `[Mathf](http://docs.unity3d.com/Documentation/ScriptReference/Mathf.html).Sin` 方法来计算它。

```
position.y = Mathf.Sin(position.x);
```

![](<images/1686829737884.png>)

The sine of X, from −1 to 1.  
X的正弦，从−1到1。

### What's `[Mathf](http://docs.unity3d.com/Documentation/ScriptReference/Mathf.html)`? 什么是 `[Mathf](http://docs.unity3d.com/Documentation/ScriptReference/Mathf.html)` ？

It is a struct in the `UnityEngine` namespace that contains a collection of mathematical functions and constants. As it works with floating-point numbers its type name was given the _f_ suffix.  
它是 `UnityEngine` 命名空间中的一个结构，包含数学函数和常量的集合。当它使用浮点数时，它的类型名称被赋予了f后缀。

The sine wave oscillates between −1 and 1. It repeats every 2π—pronounced as two pie—units, which means that it has a period of roughly 6.28. As our graph's X coordinates are between −1 and 1 we currently see less than a third of the repeating pattern. To see it in its entirety scale X by π so we end up with  
正弦波在−1和1之间振荡。它每2π重复一次，发音为两个馅饼单位，这意味着它的周期大约为6.28。由于我们的图形的X坐标在-1到1之间，我们目前看到的重复模式不到三分之一。从X乘π的整个尺度来看，我们得出f(x)=sin(πx). We can use the `[Mathf](http://docs.unity3d.com/Documentation/ScriptReference/Mathf.html).PI` constant as an approximation of π.  
.我们可以使用 `[Mathf](http://docs.unity3d.com/Documentation/ScriptReference/Mathf.html).PI` 常数作为π的近似值。

```
position.y = Mathf.Sin(Mathf.PI * position.x);
```

![](<images/1686829738435.png>)

The sine of πX.  
πX的正弦。

### What's a sine wave and π?  
正弦波和π是什么？

The sine is a trigonometric function, operating on an angle. In our case, the most useful example is a circle with radius 1, the unit circle. Each point on the circle has an angle  
正弦是一个三角函数，对一个角度进行运算。在我们的例子中，最有用的例子是半径为1的圆，即单位圆。圆上的每个点都有一个角度θ—theta—associated with it, as well as a 2D position. One way to define the coordinates of those positions is  
-θ以及2D位置。定义这些位置坐标的一种方法是[sin(θ)sin(θ+π2)]. This represents starting at the top of the circle and going around it in a clockwise direction. Instead of  
。这表示从圆的顶部开始，沿顺时针方向绕圆一周。而不是sin(θ+π2) you can also use the cosine, leading to  
你也可以使用余弦，导致[sin(θ)cos(θ)].

![](<images/1686829738972.png>)

Sine and cosine of πX.  
πX的正弦和余弦。

The angle  角度θ is expressed in radians, which corresponds to the distance traveled along the circumference of the unit circle. At the halfway point the traveled distance is equal to π, which is roughly 3.14. So the entire circumference has a length of 2π. In other words, π is the ratio between a circle's circumference and its diameter.  
以弧度表示，弧度对应于沿单位圆圆周行进的距离。在中点，行进距离等于π，大约为3.14。所以整个圆周的长度是2π。换句话说，π是圆的周长和直径之间的比率。

To animate this function, add the current game time to X before calculating the sine function. It's found via `[Time](http://docs.unity3d.com/Documentation/ScriptReference/Time.html).time`. If we scale the time by π as well the function will repeat every two seconds. So use  
要设置此函数的动画，请在计算正弦函数之前将当前游戏时间添加到X。它是通过 `[Time](http://docs.unity3d.com/Documentation/ScriptReference/Time.html).time` 找到的。如果我们也用π来缩放时间，函数将每两秒重复一次。所以使用f(x,t)=sin(π(x+t)), where  哪里t is the elapsed game time. This will advance the sine wave as time progresses, shifting it in the negative X direction.  
是经过的游戏时间。这将使正弦波随着时间的推移而前进，使其向负X方向移动。

```
position.y = Mathf.Sin(Mathf.PI * (position.x + Time.time));
```

 <video src="" control></video>

 

Animated sine wave. 动画正弦波。

Because the value of `[Time](http://docs.unity3d.com/Documentation/ScriptReference/Time.html).time` is the same for each iteration of the loop we can hoist the property invocation outside of it.  
因为 `[Time](http://docs.unity3d.com/Documentation/ScriptReference/Time.html).time` 的值对于循环的每次迭代都是相同的，所以我们可以将属性调用提升到它之外。

```
float time = Time.time;
		for (int i = 0; i < points.Length; i++) {
			Transform point = points[i];
			Vector3 position = point.localPosition;
			position.y = Mathf.Sin(Mathf.PI * (position.x + time));
			point.localPosition = position;
		}
```

### Clamping the Colors 夹紧颜色

The sine wave's amplitude is 1, which means that the lowest and highest positions that our points attain are −1 and 1. However, because the points are cubes with a size they extend a bit beyond this range. Thus we can get colors with green components that are negative or greater than 1. While this isn't noticeable let's be correct and clamp the colors to ensure that they remain in the 0–1 range.  
正弦波的振幅是1，这意味着我们的点达到的最低和最高位置是−1和1。然而，因为这些点是具有一定大小的立方体，所以它们稍微超出了这个范围。因此，我们可以得到具有负分量或大于1的绿色分量的颜色。虽然这并不明显，但让我们纠正并限制颜色，以确保它们保持在0–1范围内。

We can do this for our surface shader by passing the generated color through the `[saturate](http://developer.download.nvidia.com/cg/saturate.html)` function. This is a special function that clamps all components to 0–1. It is a common operation in shaders known as saturation, hence its name.  
我们可以通过 `[saturate](http://developer.download.nvidia.com/cg/saturate.html)` 函数传递生成的颜色来为曲面着色器执行此操作。这是一个特殊的功能，可将所有组件夹紧到0–1。它是着色器中的一种常见操作，称为饱和度，因此得名。

```
surface.Albedo.rg = saturate(input.worldPos.xy * 0.5 + 0.5);
```

The same can be done in the shader graph with the _Saturate_ node.  
在具有节点的着色器图形中也可以执行同样的操作。

![](<images/1686829739550.png>)

Saturated color in shader graph.  
着色器图形中的饱和颜色。

The next tutorial is [Mathematical Surfaces](https://catlikecoding.com/unity/tutorials/basics/mathematical-surfaces/).  
下一个教程是“数学曲面”。

[license 许可证](https://catlikecoding.com/unity/tutorials/license/) [repository](https://bitbucket.org/catlikecodingunitytutorials/basics-02-building-a-graph/) [PDF](Building-a-Graph.pdf)