原教程地址：[https://youtu.be/AmGSEH7QcDg](https://youtu.be/AmGSEH7QcDg)

B 站搬运教程地址：[https://www.bilibili.com/video/BV1gT411Z7bL/?share_source=copy_web&vd_source=10f4d7fd9e763a87da08cd00452bc8a4](https://www.bilibili.com/video/BV1gT411Z7bL/?share_source=copy_web&vd_source=10f4d7fd9e763a87da08cd00452bc8a4)

该笔记的语雀链接：
[https://www.yuque.com/wocaibuqinamochangdemingzi/akh7w4/ka1o8oxb7723ndg9?singleDoc#](https://www.yuque.com/wocaibuqinamochangdemingzi/akh7w4/ka1o8oxb7723ndg9?singleDoc#)
下篇：
[undefined](https://zhuanlan.zhihu.com/p/612951943)


## 3 角色控制器与动画 Character Controller & Animations

### 3.1 旧的角色控制方法

如果想要组织好一个游戏，应该永远将视觉效果与逻辑分开，在创建角色时，我们不直接创建一个胶囊体然后修改缩放、偏移等属性，三轴不相等的缩放或一些位置的偏移可能会让原本的代码逻辑出现问题，所以这里我们创建一个空物体，之后会**在空物体上写逻辑**，然后在空物体之下再创建一个胶囊体作为我们的角色，之后会在这个子物体上做视觉的修改

![](<images/1689211705184.png>)

  
接下来开始代码的编写，创建 Scripts 文件夹，创建 Player.cs  
**目前 Unity 中有两种实现角色控制的方法，一个是旧的 input manager，一个是新的 input system，旧的 input manager 很易用，适合做原型开发，但是复杂的项目最好用新的 input system 来做，在这个项目中我们会先用旧的 input manager 做出原型，然后替换为新的 input system**

```cs
// Player.cs中
public class Player : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 7f;
    
    private void Update()
    {
        Vector2 inputVector = new Vector2(0, 0);

        // getkey会一直返回true，而getkeydown只会在按下的一帧返回true
        if (Input.GetKey(KeyCode.W))
        {
            inputVector.y += 1;
        }
        if (Input.GetKey(KeyCode.S))
        {
            inputVector.y -= 1;
        }
        if (Input.GetKey(KeyCode.A))
        {
            inputVector.x -= 1;
        }
        if (Input.GetKey(KeyCode.D))
        {
            inputVector.x += 1;
        }
        
        inputVector = inputVector.normalized;

        Vector3 moveDir = new Vector3(inputVector.x, 0f, inputVector.y);
        transform.position += moveDir * Time.deltaTime * moveSpeed;
    }
}
```

将素材文件中的人物模型 PlayerVisual 放到空物体 Player 下面，删除之前的胶囊体，这时再操控角色可以正常移动，但是角色的朝向不会变，只需要在上面的脚本中最后加上以下代码即可实现（使用 slerp 球形插值处理转向角度变化）

```cs 
float rotateSpeed = 10f;  
transform.forward = Vector3.Slerp(transform.forward, moveDir, Time.deltaTime * rotateSpeed);
```

目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#characterVisualRotation)

### 3.2 角色动画

接下来开始添加动画，创建 Animations 文件夹，创建一个 animator，挂载到 PlayerVisual 上  
首先创建 Idle.anim，拖入 Animator 面板，右击 Entry->Make Transition 指向 Idle；打开 Animation 面板，做出角色的头部上下移动的动画

![](<images/1689211705376.png>)

  
然后创建 Walk.anim，同样拖入 Animator 面板，右击 Idle->Make Transition 指向 Walk，取消勾选 Has Exit Time，Parameters 中添加 IsWalking 参数，同时 Conditions 中将 IsWalking 设为 True，同样地，右击 Walk->Make Transition 指向 Ilde，取消 Has Exit Time，oCnditions 中设置 IsWalking 为 False，Walk 动画在 Idle 的基础上添加身体的上下移动，同时加快速度

![](<images/1689211705518.png>)

  
接下来我们创建 PlayerAnimator.cs 脚本管理角色的动画，添加到 PlayerVisual 上；在 Player.cs 中设置 IsWalking 的值，当角色移动向量不为 0 时则为 true；在 Player.Animator.cs 中更新 Animator 中设置的参数

```
// Player.cs中
...
public class Player : MonoBehaviour
{
    private bool isWalking;
	 ...
    private void Update() {
	 	  ...
        isWalking = (inputVector != Vector2.zero);
	 	  ...
    }
    
    public bool IsWalking() {
        return isWalking;
    }
}
```

```
// PlayerAnimator.cs中
using UnityEngine;

public class PlayerAnimator : MonoBehaviour
{
    private const string IS_WALKING = "IsWalking";
    
    [SerializeField] private Player player;
    
    private Animator animator;

    private void Awake()
    {
        animator = GetComponent<Animator>();
    }
    private void Update()
    {
        animator.SetBool(IS_WALKING, player.IsWalking());
    }
}
```

目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#animations)

### 3.3 Cinemachine

接着我们使用 Cinemachine 在角色移动的时候添加一些简单的摄像机动画，首先在 Package Manager 里安装 Cinemachine 的包，然后 GameObject->Cinemachine->Virtual Camera 创建一个摄像机，这样创建一个 Virtual Camera 会在 Main Camera 上加上一个 CinemachineBrain 组件，我们需要在 Virtual Camera 中去控制相机  
在 Virtual Camera 的 Inspector 面板中，将 Body->Binding Mode 设为 World Space，将 Follow 和 Look At 都设为之前创建的 Player，调整 Follow Offset，就可以得到一个简单的跟随相机（还可以创建多个 Cinemachine，通过设置 priority 确定相机控制权，如果控制权从一个相机到了另一个相机，Cinemachine 还会自动对相机位置进行插值）

![](<images/1689211705603.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#cinemachine)

### 3.4 使用新的 input system 进行重构

首先将处理玩家输入得到移动向量的部分代码分离  
创建 GameInput.cs，并在 Hierachy 创建一个 GameInput 对象，将脚本挂载到对象上；将原先 Player.cs 中处理输入的部分拿出来，放到 GameInput.cs 中，调整完的代码如下

```
// Player.cs中
using UnityEngine;

public class Player : MonoBehaviour
{
    [SerializeField] private float moveSpeed = 7f;
    [SerializeField] private GameInput gameInput;

    private bool isWalking;
    
    private void Update()
    {
        Vector2 inputVector = gameInput.GetMovementVectorNormalized();
        Vector3 moveDir = new Vector3(inputVector.x, 0f, inputVector.y);
        transform.position += moveDir * Time.deltaTime * moveSpeed;

        isWalking = (inputVector != Vector2.zero);
        float rotateSpeed = 10f;
        transform.forward = Vector3.Slerp(transform.forward, moveDir, Time.deltaTime * rotateSpeed);
    }
    
    public bool IsWalking()
    {
        return isWalking;
    }
}
```

```
// GameInput.cs中
using UnityEngine;

public class GameInput : MonoBehaviour
{
    public Vector2 GetMovementVectorNormalized() {
        Vector2 inputVector = new Vector2(0, 0);
        
        if (Input.GetKey(KeyCode.W))
        {
            inputVector.y += 1;
        }
        if (Input.GetKey(KeyCode.S))
        {
            inputVector.y -= 1;
        }
        if (Input.GetKey(KeyCode.A))
        {
            inputVector.x -= 1;
        }
        if (Input.GetKey(KeyCode.D))
        {
            inputVector.x += 1;
        }
        
        inputVector = inputVector.normalized;
        return inputVector;
    }
}
```

接下来去 Package Manager 中安装 Input System，安装完后会提示我们激活 input system，我们可以选择 no 然后在 Edit->Project Setting->Player->Other Settings 手动激活，我们在下拉菜单中选择 Both

![](<images/1689211705688.png>)

![](<images/1689211705859.png>)

  
在 Settings 文件夹下右键 ->Create->Input Actions 创建 PlayerInputActions.inputactions  
双击打开该窗口，创建一个 Action Map，创建一个 Move Action，将 Action Type 改为 Value，Control Type 改为 Vector2，删除 Move 下面的 `<No Binding>`，点击右边的加号选择 Add Up\Down\Left\Right Composite

![](<images/1689211705930.png>)

  
依次修改下方的四个方向绑定的事件，可以选择 listen 然后按下对应按键

![](<images/1689211706115.png>)

  
Input System 可以通过 Add Component 添加对应的脚本，但是这里我们选择用代码的方式使用。选中 PlayerInputAction.inputactions，在 Inspector 面板中勾选 Generate C# Class，然后 Apply

![](<images/1689211706391.png>)

  
修改 GameInput.cs，替换为将原来的方法替换为使用 Input System（归一化的操作也可以在. inputactions 文件中添加 processor）

```
// GameInput.cs中
using UnityEngine;

public class GameInput : MonoBehaviour
{
    private PlayerInputActions playerInputActions;
    
    private void Awake() {
        playerInputActions = new PlayerInputActions();
        playerInputActions.Player.Enable();
    }

    public Vector2 GetMovementVectorNormalized() {
        // Vector2 inputVector = new Vector2(0, 0);

        // if (Input.GetKey(KeyCode.W))
        // {
        //     inputVector.y += 1;
        // }
        // if (Input.GetKey(KeyCode.S))
        // {
        //     inputVector.y -= 1;
        // }
        // if (Input.GetKey(KeyCode.A))
        // {
        //     inputVector.x -= 1;
        // }
        // if (Input.GetKey(KeyCode.D))
        // {
        //     inputVector.x += 1;
        // }
        //
        
        Vector2 inputVector = playerInputActions.Player.Move.ReadValue<Vector2>();
        
        inputVector = inputVector.normalized;
        return inputVector;
    }
}
```

想要添加其他的输入方式，可以在. inputactions 文件面板添加新的 Action，比如这里添加了一个用方向键控制移动的 Action

![](<images/1689211706663.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#inputSystemRefactor)

### 3.5 碰撞检测 Collision Detection

我们可以先使用 Physics.Raycast() 方法做一个简单的碰撞检测  
在场景中放置一个 Cube，确保这个 Cube 带有 Box Collider 组件。在控制角色位置发生变化的脚本 Player.cs 中，从角色的原点出发，向移动方向发出一条射线，射线长度大于角色的大小时才可以移动

```
// Player.cs中
...
public class Player : MonoBehaviour
{
    ...
    private void Update() {
        ...
        float playerRadius = .7f;
        bool canMove = !Physics.Raycast(transform.position, moveDir, playerRadius);

        if (canMove)
        {
            transform.position += moveDir * Time.deltaTime * moveSpeed;
        }
        ...
    }
    ...
}
```

这样在对着正方体移动时确实正确处理了碰撞，但是由于我们只从原点发射了一条射线，有且情况还是会发生穿模，所以我们需要使用 Physics.CapsuleCast() 方法

![](<images/1689211706871.png>)

  
Physics.CapsuleCast() 方法的六个参数分别定义了胶囊体的底部、顶部、半径、射线发射方向、射线最大距离

```
// Player.cs中
...
float moveDistance = moveSpeed * Time.deltaTime;  
float playerRadius = .7f;
float playerHeight = 2f;
bool canMove = !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDir, moveDistance);
...
```

使用 Physics.CapsuleCast() 方法后，即是在边缘也能发生碰撞了，但是如果在一个面前有墙的地方同时按住向上移动和向右移动的方向键，角色也不会移动，但通常在游戏中这种情况通常会让角色朝右移动

![](<images/1689211707035.png>)

  
我们可以多增加一些逻辑来让角色有其他方向的速度向量时仍然移动

```
// Player.cs中
...
float moveDistance = moveSpeed * Time.deltaTime;  
float playerRadius = .7f;
float playerHeight = 2f;
bool canMove = !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDir, moveDistance);

if (!canMove)
{
	// 当不能向moveDir方向移动时
	
	// 尝试沿x轴移动
	Vector3 moveDirX = new Vector3(moveDir.x, 0f, 0f).normalized; // 归一化让速度和直接左右移动相同
	canMove = !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirX, moveDistance);

	if (canMove)
	{
		 // 可以沿x轴移动
		 moveDir = moveDirX;
	} else
	{
		 // 不能向x轴方向移动，尝试向z轴方向移动
		 Vector3 moveDirZ = new Vector3(0f, 0f, moveDir.z).normalized; // 归一化让速度和直接左右移动相同
		 canMove = !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirZ, moveDistance);

		 if (canMove)
		 {
			  // 可以向z轴方向移动
			  moveDir = moveDirZ;
		 } else
		 {
			  // 不能朝任何方向移动
		 }
	}
}

if (canMove)
{
	transform.position += moveDir * Time.deltaTime * moveSpeed;
}
...
```

进行一些调整之后，我们就可以正常移动了

![](<images/1689211707667.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#collisionDetection)

  
在原视频 4:24:00 处作者进行了一些改进，现在如果我们垂直向墙体走动，由于当不能向 moveDir 方向移动时我们限制住了 moveDir，所以垂直向墙体走动的时候 moveDir 为 (0, 0, 0)，而我们的动画是当前角色朝向往 moveDir 方向插值的，所以我们的角色在垂直向墙体走动的时候不会向墙体转向，效果如下

![](<images/1689211707714.png>)

  
要解决这个问题，需要改动以下语句（注意这里`moveDir.x != 0`和`moveDir.z != 0`的加入也给更后面的手柄输入带来了一定问题，在最后的时候作者改为了`(moveDir.x < -0.5f || moveDir.x > 0.5f)`和 `(moveDir.z < -0.5f || moveDir.z> 0.5f)）

```
// Player.cs中
...
if (!canMove)
{
	...
	// canMove = !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirX, moveDistance);
	canMove = moveDir.x != 0 && !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirX, moveDistance);
	...
	if (canMove)
	{
		 ...
	} else
	{
		 ...
		 // canMove = !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirZ, moveDistance);
		 canMove = moveDir.z != 0 && !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirZ, moveDistance);
		 ...
		 if (canMove)
		 {
			  ...
		 } else
		 {
			  ...
		 }
	}
}
```

改动之后，就能够在垂直朝墙体走动时正常转向了

![](<images/1689211707859.png>)

## 4 创建空的柜台 Clear Counter

### 4.1 添加柜台

首先在场景中创建一个空物体，命名为 ClearCounter，在_Assets/PrefabsVisuals/CountersVisuals 下找到 ClearCounter_Visual 拖动到 ClearCounter 下，在 ClearCounter 上添加一个 Box Collider 组件，调整到合适大小，现在角色就可以和柜台发生碰撞了

![](<images/1689211708239.png>)

  
我们需要把设置好碰撞体积的柜台变为一个 prefab，这样每次只要使用这个 prefab 就可以了，新建文件夹，命名为 Prefabs，然后将需要制作 prefab 的物体从 Hierachy 窗口拖入文件夹

### 4.2 利用 Raycast 处理角色与柜台的交互

然后我们在 Player.cs 中开始写角色与柜台交互的代码，首先先将原代码进行整理，将处理移动的代码放入到一个函数中去

```
// Player.cs中
public class Player : MonoBehaviour
{
    ...
    private void Update() {
        HandleMovement();
        HandleInteractions()
    }
    
    public bool IsWalking() {
        return isWalking;
    }

    private void HandleInteractions() {
        
    }

    private void HandleMovement() {
        // 之前Update中的代码全放在这里
    }
    ...
}
```

Physics.Raycast()有一个构造函数可以填入一个参数用来返回被射线击中位置的属性，这里用 raycastHit.transform 来返回被击中的物体信息，我们可以在这里测试一下，当在可交互距离内击中则返回物体名称，未击中则返回 "-"，使用 lastInteractDir 保存上一次操作时移动的方向，防止当移动速度为 0 时 moveDir 为 (0, 0, 0) 而无法确定是否可以与柜台交互

```
// Player.cs中
 ...
 private Vector3 lastInteractDir;
 ...
 private void HandleInteractions()
 {
	  Vector2 inputVector = gameInput.GetMovementVectorNormalized();
	  
	  Vector3 moveDir = new Vector3(inputVector.x, 0f, inputVector.y);
	          
	  if (moveDir != Vector3.zero)
	  {
			lastInteractDir = moveDir;
	  }
	  
	  float interactDistance = 2f;
	  if (Physics.Raycast(transform.position, lastInteractDir, out RaycastHit raycastHit, interactDistance))
	  {
			Debug.Log(raycastHit.transform);
	  } else
	  {
			Debug.Log("-");
	  }        
 }
 ...
```

![](<images/1689211708304.png>)

  
接下来开始给柜台添加脚本，在 Scripts 文件夹新建 ClearCounter.cs，将脚本添加到 ClearCounter.prefab 上

```
// ClearCounter.cs中
using UnityEngine;

public class ClearCounter : MonoBehaviour
{
    public void Interact() {
        Debug.Log("Interact");
    }
}
```

为了处理角色与柜台间的交互，我们同样需要在 Player.cs 中添加代码

```
// Player.cs中
 ...
 private void HandleInteractions()
 {
	  Vector2 inputVector = gameInput.GetMovementVectorNormalized();
	  
	  Vector3 moveDir = new Vector3(inputVector.x, 0f, inputVector.y);
	  
	  if (moveDir != Vector3.zero)
	  {
			lastInteractDir = moveDir;
	  }
	  
	  float interactDistance = 2f;
	  if (Physics.Raycast(transform.position, lastInteractDir, out RaycastHit raycastHit,interactDistance))
	  {
			if (raycastHit.transform.TryGetComponent(out ClearCounter clearCounter))
			{
				 // 线击中的物体拥有ClearCounter.cs脚本
				 clearCounter.Interact();
			}
			
			// 使用TryGetComponent()方法和使用下面的代码相同，会检测到是否有ClearCounter.cs
			// ClearCounter clearCounter = raycastHit.transform.GetComponent<ClearCounter>();
			// if (clearCounter != null)
			// {
			//     // Has clearCounter.cs
			// }
	  } else
	  {
			Debug.Log("-");
	  }
 }
 ...
```

### 4.3 Layermask

现在我们的代码还有一个问题，如果有其他东西挡在了角色与柜台之间，射线就不会打到柜台，所以我们可以将 ClearCounter.prefab 单独设置在一个 Layer 中（要手动添加一个 Layer），然后在 Player.cs 使用 Physics.Raycast 的其中的一个构造函数，最后一个参数可以传入一个 layermask

![](<images/1689211708531.png>)

```
// Player.cs中
 ...
 [SerializeField] private LayerMask countersLayerMask;
 ...
 private void HandleInteractions()
 {
	  ...
	  if (Physics.Raycast(transform.position, lastInteractDir, out RaycastHit raycastHit,interactDistance, countersLayerMask))
	  {
			if (raycastHit.transform.TryGetComponent(out ClearCounter clearCounter))
			{
				 // Has ClearCounter.cs
				 clearCounter.Interact();
			}
	  } else
	  {
			Debug.Log("-");
	  }
 }
 ...
```

目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#clearCounter)

## 5 处理 Interact 输入的 C# 事件 Interact Action C# Events

### 5.1 添加 Interact Action

打开 PlayerInputActions.inputactions，添加一个 Action，命名为 Interact，绑定 E 键

![](<images/1689211709009.png>)

  
在 GameInput.cs 中，我们使用委托为这个 Interact Action 添加一个调用的函数 Interact_performed()，我们先来测试一下，使用 Debug.log(obj) 在控制台输出一下调用的函数本身

```
// GameInput.cs中
using UnityEngine;

public class GameInput : MonoBehaviour
{
    private PlayerInputActions playerInputActions;
    
    private void Awake() {
        playerInputActions = new PlayerInputActions();
        playerInputActions.Player.Enable();
        
        playerInputActions.Player.Interact.performed += Interact_performed;
    }
    
    private void Interact_performed(UnityEngine.InputSystem.InputAction.CallbackContext obj) {
        Debug.Log(obj);
    }

    public Vector2 GetMovementVectorNormalized() {
        Vector2 inputVector = playerInputActions.Player.Move.ReadValue<Vector2>();
        
        inputVector = inputVector.normalized;
        return inputVector;
    }
}
```

[Unity Input System 文档](https://docs.unity3d.com/Packages/com.unity.inputsystem@1.0/manual/Actions.html)

![](<images/1689211709384.png>)

启动游戏，按下 E 键，可以看到控制台输出了我们的按下对应按键的相关信息

![](<images/1689211709426.png>)

### 5.2 使用 EventHandler 委托将交互逻辑写在 Player.cs

接下来在 Interact Action 调用的函数 Interact_performed() 中添加 EventHandler 委托，在 Player.cs 中为该委托添加具体的交互行为

```
// GameInput.cs中
using System;
using UnityEngine;

public class GameInput : MonoBehaviour
{
    public event EventHandler OnInteractAction; // 新添加
    
    private PlayerInputActions playerInputActions;
    
    private void Awake()
    {
        playerInputActions = new PlayerInputActions();
        playerInputActions.Player.Enable();
        
        playerInputActions.Player.Interact.performed += Interact_performed;
    }
    
    private void Interact_performed(UnityEngine.InputSystem.InputAction.CallbackContext obj)
    {
        OnInteractAction?.Invoke(this, EventArgs.Empty); // 新添加
        // 这里使用了"?"运算符，与下面的代码相同
        // if (OnInteractAction != null)
        // {
        //     OnInteractAction(this, EventArgs.Empty);
        // }
    }

    public Vector2 GetMovementVectorNormalized()
    {
        Vector2 inputVector = playerInputActions.Player.Move.ReadValue<Vector2>();
        
        inputVector = inputVector.normalized;
        return inputVector;
    }
}
```

这里可以在 GameInput_OnInteractAction() 中先放上之前 HandleInteractions() 的代码测试一下

```
// Player.cs中 
...
 private void Start()
 {
	  gameInput.OnInteractAction += GameInput_OnInteractAction;
 }
 
 private void GameInput_OnInteractAction(object sender, EventArgs e)
 {
	  //将原先HandleInteractions()中的内容暂时放到了这里，并且暂时不再调用HandleInteractions()中的clearCounter.Interact()
	  Vector2 inputVector = gameInput.GetMovementVectorNormalized();
	  
	  Vector3 moveDir = new Vector3(inputVector.x, 0f, inputVector.y);
	  
	  if (moveDir != Vector3.zero)
	  {
			lastInteractDir = moveDir;
	  }
	  
	  float interactDistance = 2f;
	  if (Physics.Raycast(transform.position, lastInteractDir, out RaycastHit raycastHit,interactDistance, countersLayerMask))
	  {
			if (raycastHit.transform.TryGetComponent(out ClearCounter clearCounter))
			{
				 // Has ClearCounter.cs
				 clearCounter.Interact();
			}
	  }
 }
 ...
```

这时运行游戏按 E 进行交互，可以得到显示 Interact（我们先前在 clearCounter.Interact() 中定义了输出 "Interact"）

![](<images/1689211709502.png>)

个人解释：使用`playerInputActions.Player.Interact.performed += Interact_performed;`是为该控制器触发的委托添加了一个会调用的函数，在这个函数中，我们写具体的按下按键发生的事情，这里我们想把具体的交互行为写在 Player.cs 中，所以我们在这个函数中去触发一个叫`OnInteractAction`的 Eventhandler 委托，这个委托添加的会调用的函数写在了 Player.cs 中，使用`gameInput.OnInteractAction += GameInput_OnInteractAction;`添加了`GameInput_OnInteractAction()`这个具体处理输入逻辑函数  
关于委托和事件的讲解可以看[刘铁猛老师的 C# 课程](https://www.bilibili.com/video/BV13b411b7Ht?p=19&vd_source=c01a7b5440706d76efa61acaf26acff7)和 [CodeMonkey 讲 C# 的相关课程](https://www.youtube.com/playlist?list=PLzDRvYVwl53t2GGC4rV_AmH7vSvSqjVmz)

目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#interactActionCSEvents)

## 6 选中柜台的视觉效果与单例模式 Select Counter Visual Singleton Pattern

### 6.1 添加带有选中效果的模型

我们可以将柜台视觉效果的改变写在 Player.cs 中最后 clearCounter.Interact() 中，然而这样在写角色逻辑的代码中处理柜台的视觉效果，会使视觉效果与交互逻辑的代码耦合，所以这里我们要使用其他的方法去改变柜台的视觉效果  
在 ClearCounter.perfab 中，复制 ClearCounter_Visual 并重命名为 Selected，选择下面的模型，将 Material 换为 CounterSelected，在 Scripts 文件夹中新建 SelectCounterVisual.cs 并将该脚本添加到 Selected 上，这样当我们在打开 Selected 的显示时就能得到一个选中效果的柜台

![](<images/1689211709631.png>)

  
像这样两个网格重叠时，可能会形成闪烁的效果，为了避免这种情况，我们可以将选中效果的柜台稍微放大一些，例如这里将所有轴的缩放改为 1.01

![](<images/1689211709862.png>)

### 6.2 为柜台增加选中的效果

我们在 Player.cs 中增加一个 selectedCounter 变量，用于获取当前被选中的柜台，在 HandleInteractions() 中随着投射出的光线击中的物体改变而改变当前 selectedCounter 为哪一个柜台，然后在 GameInput_OnInteractAction() 中调用 selectedCounter.Interact() 实现交互

```
// Player.cs中
...
public class Player : MonoBehaviour
{
    ...
    private ClearCounter selectedCounter;

    private void Start()
    {
        gameInput.OnInteractAction += GameInput_OnInteractAction;
    }
    
    private void GameInput_OnInteractAction(object sender, EventArgs e)
    {
        if (selectedCounter != null)
        {
            selectedCounter.Interact();
        }
    }

    private void Update()
    {
        HandleMovement();
        HandleInteractions();
    }
    ...
    private void HandleInteractions()
    {
        ...
        
        if (Physics.Raycast(transform.position, lastInteractDir, out RaycastHit raycastHit,interactDistance, countersLayerMask))
        {
            if (raycastHit.transform.TryGetComponent(out ClearCounter clearCounter))
            {
                // 射线击中的物体拥有ClearCounter.cs脚本
                // 如果当前交互的柜台不是上一次选中的柜台，就把选中的clearCounter设置为当前的柜台
                if (clearCounter != selectedCounter)
                {
                    selectedCounter = clearCounter;
                } 
            } else
            {
                // 射线击中的物体没有ClearCounter.cs脚本
                selectedCounter = null;
            }
        } else
        {
            // 没有射线碰撞到任何东西
            selectedCounter = null;
        }
        
        Debug.Log(selectedCounter);
    }
    ...
}
```

现在运行游戏，我们就可以在靠近柜台时看到 Debug.Log(selectedCounter) 信息显示出我们当前选中的柜台

![](<images/1689211709977.png>)

  
接下来我们处理选中的视觉效果，这里我们有两种思路可以选择。一种是让当前 selectedCounter 发送选中的事件，在处理视觉表现的脚本中订阅该事件写选中时视觉效果的变化，也就是在 ClearCounter.cs 中发送事件，在 SeletedCounterVisual.cs 中订阅该事件；另一种思路是让当前操作的角色发送事件，在处理视觉表现的脚本中订阅该事件写选中时视觉效果的变化。  
第一种方式的好处是 SeletedCounterVisual.cs 只会订阅当前选中柜台的事件，并且如果我们需要选中时添加其他效果也可以很方便地添加，坏处是控制视觉效果的代码又经过了专门处理逻辑的 ClearCounter.cs 的脚本中，会造成代码的耦合  
第二种方式的好处是它更加方便，控制视觉效果的代码不会经过处理逻辑的脚本，坏处是可能会存在性能问题，因为所有的柜台都在订阅由角色发送的事件，但是我们的项目体量比较小，这样的方法并不会带来性能瓶颈，所以我们选择使用这一种方法。并且由于我们要完成的游戏只有一个角色，使用这一种方法还可以使用单例模式  
在 Player.cs 中，添加一个 [EventHandler](https://learn.microsoft.com/en-us/dotnet/api/system.eventargs?view=net-7.0) 委托，并使用 EventArgs 传递选中的柜台信息，在 Update() 中触发事件，这里由于要使用到多次 OnSelectedCounterChanged，所以这里把它单独放在了一个函数中

```
// Player.cs中
...
public class Player : MonoBehaviour
{
    ...
    public event EventHandler<OnSelectedCounterChangedEventArgs> OnSelectedCounterChanged;
    public class OnSelectedCounterChangedEventArgs : EventArgs
    {
        public ClearCounter selectedCounter;
    }
    ...
    private void Update()
    {
        ...
        HandleInteractions();
    }
    ...
    private void HandleInteractions()
    {
        ...
        if (Physics.Raycast(transform.position, lastInteractDir, out RaycastHit raycastHit,interactDistance, countersLayerMask))
        {
            if (raycastHit.transform.TryGetComponent(out ClearCounter clearCounter))
            {
                // 射线击中的物体拥有ClearCounter.cs脚本
                // 如果当前交互的柜台不是上一次选中的柜台，就把选中的clearCounter设置为当前的柜台
                if (clearCounter != selectedCounter)
                {
                    SetSelectedCounter(clearCounter);
                } 
            } else
            {
                // 射线击中的物体没有ClearCounter.cs脚本
                SetSelectedCounter(null);
            }
        } else
        {
            // 没有射线碰撞到任何东西
            SetSelectedCounter(null);
        }
    }
    ...
    private void SetSelectedCounter(ClearCounter selectedCounter)
    {
        this.selectedCounter = selectedCounter;
        
        OnSelectedCounterChanged?.Invoke(this, new OnSelectedCounterChangedEventArgs
        {
            selectedCounter = selectedCounter
        });
    }
}
```

我们需要在 SelectCounterVisual.cs 中订阅该事件，由于我们只有一个 Player，所以我们可以使用单例模式  
依然是在 Player.cs 中，我们定义一个 public 的、static 的 Player 的实例，并且设置为只读（public 是为了其他类能够访问到，static 是让该实例与类无关，让程序中只有始终只有一个 Player 实例），我们必须确保游戏中只有一个 Player，所以我们需要在 Awake() 中进行检查

```
// Player.cs中
...
public class Player : MonoBehaviour
{
    public static Player Instance { get; private set; }
    ...
    private void Awake()
    {
        if (Instance != null)
        {
            Debug.LogError("There is more than one Player instance");
        }
        Instance = this;
    }
    ...
}
...
```

在 SelectCounterVisual.cs 中，订阅这个实例上的事件控制选中效果模型的显示与隐藏，由于我们在 Player.cs 中使用了 Awake() 设置了单例模式中的实例，而 Awake() 会在 Start() 之前执行，所以程序可以正常运行，如果我们这里也使用 Awake()，则可能会导致单例设置前该脚本就已经执行（如果都使用 Awake()，还可以在 Edit->Project Settings->Script Execution Order 中规定脚本的执行顺序）

```
// SelectCounterVisual.cs中
using UnityEngine;

public class SelectCounterVisual : MonoBehaviour
{
    [SerializeField] private ClearCounter clearCounter;
    [SerializeField] private GameObject visualGameObject;
    
    private void Start()
    {
        Player.Instance.OnSelectedCounterChanged += Player_OnSelectedCounterChanged;
    }
    
    private void Player_OnSelectedCounterChanged(object sender, Player.OnSelectedCounterChangedEventArgs e)
    {
        if (e.selectedCounter == clearCounter)
        {
            show();
        }
        else
        {
            hide();
        }
    }

    private void show()
    {
        visualGameObject.SetActive(true);
    }
    
    private void hide()
    {
        visualGameObject.SetActive(false);
    }
}
```

现在，我们就实现了物品的选中效果

![](<images/1689211710034.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#selectedCounterVisual)

## 7 放置物品与 Scriptable Objects

### 7.1 按 E 在柜台上生成物品

我们先来实现按 E 在柜台上生成番茄的效果  
首先要创建番茄的 prefab，在场景中创建一个空物体命名为 Tomato，在_Assets/PrefabVisuals/KitchenObjects 下找到 Tomato_Visual，拖动到空物体下，将该带着番茄模型的空物体再拖回 Prefabs 中创建 prefab（为了之后分类方便，这里视频中在 Prefab 文件夹下分为了 Counters 文件夹和 KitchenObjects 文件夹，番茄放到了 KitchenObjects 中），创建完 prefab 后，删除场景中的番茄

![](<images/1689211710167.png>)

  
为了定位番茄生成的位置，我们需要在 ClearCounter.prefab 中创建一个空物体，重命名为 CounterTopPoint，移动到柜台的正上方

![](<images/1689211710937.png>)

  
接下来在 ClearCounter.cs 中，接收 tomatoPrefab 与 counterTopPoint，并在 Interact() 中实例化一个番茄

```
// ClearCounter.cs中
using UnityEngine;

public class ClearCounter : MonoBehaviour
{
    [SerializeField] private Transform tomatoPrefab;
    [SerializeField] private Transform counterTopPoint;

    public void Interact()
    {
        Debug.Log("Interact");
        Transform tomatoTransform = Instantiate(tomatoPrefab, counterTopPoint);
        tomatoTransform.localPosition = Vector3.zero;
    }
}
```

现在运行游戏，当柜子高亮，按下 E 即可放置番茄，现在我们测试一下其他物体，复制 Tomato.prefab，重命名为 Cheese.prefab，将其中的 Tomato_Visual 换为 CheeseBlock_Visual，拖动 Cheese.prefab 到其中一个柜台的脚本下

![](<images/1689211711258.png>)

  
运行游戏，按 E 即可放置番茄与奶酪

![](<images/1689211711302.png>)

### 7.2 Scriptable Objects

Scriptable Object 可以很方便地定义一个类的多种不同实例，如多种武器、多种装备、多种食物等等  
首先在 Scripts 文件夹新建 KitchenObjectSO.cs，注意这里不是继承自 MonoBehaviour，而是 ScriptableObject，要创建 Scriptable Object，还需要在类前面加上 Unity 为我们准备的 [CreateAssetMenu()]，这样我们就可以在 Unity 编辑器中使用这个脚本创建对象

```
// KitchenObjectSO.cs中
using UnityEngine;

[CreateAssetMenu()]
public class KitchenObjectSO : ScriptableObject
{
    public Transform prefab;
    public Sprite sprite;
    public string objectName;
}
```

创建文件夹 ScriptableObjects/KitchenObjectSO（截图里多加了个 s，后面我改了），右键 create，点击最上方的 Kitchen Object SO 新建文件，重命名为 Tomato.asset，并在 Inspector 面板中将 prefab、Sprite、Object Name 填好

![](<images/1689211711361.png>)

  
在 ClearCounter.cs 中，修改对应部分，使其通过 kitchenObjectSO 对象调用对应 prefab

```
// ClearCounter.cs中
using UnityEngine;

public class ClearCounter : MonoBehaviour
{
    // [SerializeField]private Transform tomatoPrefab;
    [SerializeField] private KitchenObjectSO kitchenObjectSO;
    [SerializeField] private Transform counterTopPoint;

    public void Interact()
    {
        // Transform tomatoTransform = Instantiate(tomatoPrefab, counterTopPoint);
        Transform kitchenObjectTransform = Instantiate(kitchenObjectSO.prefab, counterTopPoint);
        // tomatoTransform.localPosition = Vector3.zero;
        kitchenObjectTransform.localPosition = Vector3.zero;
    }
}
```

在场景中挂载了 ClearCounter.cs 脚本的 ClearCounter 上，将 Kitchen Object SO 设置为 Tomato，开始游戏，可以正常交互，同时可以通过 Script Object 创建另一个 CheeseBlock.asset，将另一个柜台的 Kitchen Object SO 设置为 CheeseBlock，开始游戏，得到和之前相同的效果

![](<images/1689211711490.png>)

  
我们需要让柜台知道在其上方放置的物品是什么，但是由于 Script Object 不是继承自 Monobehaviour 的类，所以不能作为一个组件添加到物体上，所以我们需要在 Scripts 文件夹下创建一个 KitchenObject.cs，将它添加到我们已经创建的两个 prefab 上并将对应的 Tomato.asset 和 CheeseBlock.asset 拖动到脚本上

```
// KitchenObject.cs中
using UnityEngine;

public class KitchenObject : MonoBehaviour
{
    [SerializeField] private KitchenObjectSO kitchenObjectSO;
    
    public KitchenObjectSO GetKitchenObjectSO()
    {
        return kitchenObjectSO;
    }
}
```

然后在 ClearCounter.cs 中获取放置物品信息

```
// ClearCounter.cs中
...
public class ClearCounter : MonoBehaviour
{
    ...
    public void Interact() {
        ...
        Debug.Log(kitchenObjectTransform.GetComponent<KitchenObject>().GetKitchenObjectSO());
    }
}
```

现在运行游戏，就能在控制台看到放置物品时的信息了

![](<images/1689211711656.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#kitchenObjectScriptableObjects)

### 7.3 Kitchen Object Parent

现在我们只能在特定的柜子上放特定的物品，而最后游戏应该是物品可以放置到其他柜子上的，所以我们需要一直改变 Kitchen Object 的父级，这意味着我们需要让 Kitchen Object 知道自己在哪个 Counter 上，Counter 也能知道哪个 Kitchen Object 在放置在了自己身上，以便接下来移动 Kitchen Object 到其他位置  
在 KitchenObject.cs 中，用变量 clearCounter 保存当前物品放置在的柜子，增加可以设置和获取当前放置在的柜子的方法

```
// KitchenObject.cs中
using UnityEngine;

public class KitchenObject : MonoBehaviour
{
    [SerializeField] private KitchenObjectSO kitchenObjectSO;
    
    private ClearCounter clearCounter;
    
    public KitchenObjectSO GetKitchenObjectSO()
    {
        return kitchenObjectSO;
    }
    
    public void SetClearCounter(ClearCounter clearCounter)
    {
        this.clearCounter = clearCounter;
    }
    
    public ClearCounter GetClearCounter()
    {
        return clearCounter;
    }
}
```

在 ClearCounter.cs 中，用变量 kitchenObject 来保存当前柜子上放置的物品，并用 if else 避免柜子上放置多个物体

```
// ClearCounter.cs中
using UnityEngine;

public class ClearCounter : MonoBehaviour
{
    [SerializeField] private KitchenObjectSO kitchenObjectSO;
    [SerializeField] private Transform counterTopPoint;
    
    private KitchenObject kitchenObject;

    public void Interact()
    {
        if (kitchenObject == null)
        {
            Transform kitchenObjectTransform = Instantiate(kitchenObjectSO.prefab, counterTopPoint);
            kitchenObjectTransform.localPosition = Vector3.zero;
            
            kitchenObject = kitchenObjectTransform.GetComponent<KitchenObject>();
            kitchenObject = SetClearCounter(this);
        } else  
        {  
            Debug.Log(kitchenObject.GetClearCounter());  
        }
    }
}
```

接下来我们先来测试一下如何改变物体的父级，最终我们要让物品可以成为成为任意柜子的子级，也可以成为角色的子级，我们先来实现按 T 键物品就可以从一个柜子的子级变为另一个柜子的子级，并且位置从一个柜子移动到另一个柜子上的逻辑  
在 ClearCounter.cs 中，增加 secondClearCounter 参数获取另一个柜子，增加叫 testing 的布尔值方便测试，在 Update() 中检测如果按下了 T 键则将当前柜子上的物品的父级设为另一个柜子，为了将位置也移动过去，我们新增了一个 GetKitchenObjectFollowTransform() 方法用来获取柜子上放置物品的位置，并在 KitchenObject.cs 中的 SetClearCounter() 中加上设置位置的代码

```
// ClearCounter.cs中
using UnityEngine;

public class ClearCounter : MonoBehaviour
{
    [SerializeField] private KitchenObjectSO kitchenObjectSO;
    [SerializeField] private Transform counterTopPoint;
    [SerializeField] private ClearCounter secondClearCounter;
    [SerializeField] private bool testing;

    private KitchenObject kitchenObject;

    private void Update()
    {
        if (testing && Input.GetKeyDown(KeyCode.T))
        {
            if (kitchenObject != null)
            {
                kitchenObject.SetClearCounter(secondClearCounter);
            }
        }
    }

    public void Interact()
    {
        if (kitchenObject == null)
        {
            Transform kitchenObjectTransform = Instantiate(kitchenObjectSO.prefab, counterTopPoint);
            kitchenObjectTransform.localPosition = Vector3.zero;
            
            kitchenObject = kitchenObjectTransform.GetComponent<KitchenObject>();
            kitchenObject.SetClearCounter(this);
        } else
        {
            Debug.Log(kitchenObject.GetClearCounter());
        }
    }
    
    public Transform GetKitchenObjectFollowTransform()
    {
        return counterTopPoint;
    }
}
```

```
// KitchenObject.cs中
...
public class KitchenObject : MonoBehaviour
{
    ...
    public KitchenObjectSO GetKitchenObjectSO(){...}
    
    public void SetClearCounter(ClearCounter clearCounter) {
        this.clearCounter = clearCounter;
        transform.parent = clearCounter.GetKitchenObjectFollowTransform();
        transform.localPosition = Vector3.zero;
    }
    
    public ClearCounter GetClearCounter(){...}
}
```

这时调整好相应参数，运行游戏，按下 T 键时发现在 Hierachy 中番茄确实从当前柜子跑到了另一个柜子上，并且位置也发生了改变

![](<images/1689211711943.png>)

  
然而这样操作时我们的两个 ClearCounter 物体中的 KitchenObject 变量并没有改变，第一个柜子的 KitchenObject 依然是 Tomato，第二个柜子的 KitchenObject 依然是 None（private 的变量可以通过右上角三个点打开 debug 模式查看）

![](<images/1689211712203.png>)

  
我们有两个途径可以解决这个问题，第一种是在 ClearCounter.cs 中设置 kitchenObject 的父级后让新的父级去更新一下对应变量，第二种是在 KitchenObject.cs 中当该物体被设置到新的父级上时自己去更新父级，这里教程中用了第二种，作者认为让物体去更新父级的代码写在物体对应的脚本中更合理一些  
为了在 KitchenObject.cs 中更新父级，还需要在 ClearCounter.cs 中增加获取、设置、清空当前柜子下物体和判断当前柜子上是否有物体的方法，然后在 KitchenObject.cs 的 SetClearCounter() 中更新父级，当我们把更新父级的逻辑写在 KitchenObject.cs 的 SetClearCounter() 后，原来 KitchenObject.cs 的 Interact() 中的逻辑就可以不用了，直接 SetKitchenObjectParent() 即可

```
// ClearCounter.cs中
public class ClearCounter : MonoBehaviour
{
    ...
    public void Interact()
    {
        if (kitchenObject == null)
        {
            Transform kitchenObjectTransform = Instantiate(kitchenObjectSO.prefab, counterTopPoint);
            // kitchenObjectTransform.localPosition = Vector3.zero;
            // kitchenObject = kitchenObjectTransform.GetComponent<KitchenObject>();
            // kitchenObject.SetKitchenObjectParent(this);
            kitchenObjectTransform.GetComponent<KitchenObject>().SetKitchenObjectParent(this);
        } else
        {
            Debug.Log(kitchenObject.GetClearCounter());
        }
    }
    ...
    public void SetKitchenObject(KitchenObject kitchenObject)
    {
        this.kitchenObject = kitchenObject;
    }

    public KitchenObject GetKitchenObject()
    {
        return kitchenObject;
    }
    
    public void ClearKitchenObject()
    {
        kitchenObject = null;
    }
    
    public bool HasKitchenObject()
    {
        return kitchenObject != null;
    }
}
```

```
// KitchenObject.cs中
...
public class KitchenObject : MonoBehaviour
{
    ...
    public void SetClearCounter(ClearCounter clearCounter)
    {
        if (this.clearCounter != null) // 这里的this.clearCounter指的是之前的clearCounter
        {
            this.clearCounter.ClearKitchenObject();
        }
        this.clearCounter = clearCounter;

        if (clearCounter.HasKitchenObject()) // 这里clearCounter指的是现在的、作为传入的clearCounter
        {  
            Debug.LogError("ClearCounter already has a KitchenObject!");  
        }
        clearCounter.SetKitchenObject(this);
        
        transform.parent = clearCounter.GetKitchenObjectFollowTransform();
        transform.localPosition = Vector3.zero;
    }
    ...
}
```

再次运行游戏，可以看到按下 T 键之后变量可以被正确更改了

![](<images/1689211712250.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#kitchenObjectParent)

## 8 角色拿取物品与 C# 接口 Player Pick Up & C# Interfaces

我们已经实现了物品从一个柜子上转移到另一个柜子上，同时切换了物品的父级，我们同样也可以将物品从一个柜子上转移到角色身上，同时切换物品的父级为角色，然而，我们现有的逻辑都是针对 ClearCounter 类写的，因此我们可以把这部分逻辑抽象成一个接口，让 Player 类与 ClearCounter 都继承自这个接口，这部分基本上是提取接口、修改变量名的过程，所以笔记就简单写了

我们可以抽象出来接口 IkitchenObjectParent，让所有可以成为物品父级的对象都继承自这个接口，这个接口中要实现的方法就和之前 ClearCounter.cs 中的获取、设置、删除物品等方法相同，在 Scripts 文件夹新建 c# 脚本，重命名为 IkitchenObjectParent.cs

```
// IkitchenObjectParent.cs中
using UnityEngine;

public interface IKitchenObjectParent
{
    public Transform GetKitchenObjectFollowTransform();

    public void SetKitchenObject(KitchenObject kitchenObject);

    public KitchenObject GetKitchenObject();

    public void ClearKitchenObject();

    public bool HasKitchenObject();
}
```

在 ClearCounter.cs 中，我们要继承接口实现方法，并且要实现让 Inertct()方法实现 “当按下交互键时，如果柜子上有物品，就将物品转移到角色上” 的逻辑，所以需要传入 Player 类的实例作为参数，在 else 部分调用 SetKitchenObjectParent(player)实现转移的逻辑，以下是实现了对应逻辑并且根据接口进行改名的脚本

```
// ClearCounter.cs中
using UnityEngine;

public class ClearCounter : MonoBehaviour, IKitchenObjectParent
{
    [SerializeField] private KitchenObjectSO kitchenObjectSO;
    [SerializeField] private Transform counterTopPoint;


    private KitchenObject kitchenObject;


    public void Interact(Player player)
    {
        if (kitchenObject == null)
        {
            Transform kitchenObjectTransform = Instantiate(kitchenObjectSO.prefab, counterTopPoint);
            kitchenObjectTransform.GetComponent<KitchenObject>().SetKitchenObjectParent(this);
        } else
        {
            // 将物品转到角色下，成为角色的子级
            kitchenObject.SetKitchenObjectParent(player);
        }
    }
    
    public Transform GetKitchenObjectFollowTransform()
    {
        return counterTopPoint;
    }
    
    public void SetKitchenObject(KitchenObject kitchenObject)
    {
        this.kitchenObject = kitchenObject;
    }

    public KitchenObject GetKitchenObject()
    {
        return kitchenObject;
    }
    
    public void ClearKitchenObject()
    {
        kitchenObject = null;
    }
    
    public bool HasKitchenObject()
    {
        return kitchenObject != null;
    }
}
```

在 Player.cs 中，同样需要继承接口实现方法，并且由于调用了 Interact()，经过上面的修改之后 Interact() 需要传入参数，所以调用的时候要传入 this

```
// Player.cs
...
public class Player : MonoBehaviour, IKitchenObjectParent
{
    ...
    private void GameInput_OnInteractAction(object sender, EventArgs e)
    {
        if (selectedCounter != null)
        {
            selectedCounter.Interact(this);
        }
    }
    ...    
    public Transform GetKitchenObjectFollowTransform()
    {
        return kitchenObjectHoldPoint;
    }
    
    public void SetKitchenObject(KitchenObject kitchenObject)
    {
        this.kitchenObject = kitchenObject;
    }

    public KitchenObject GetKitchenObject()
    {
        return kitchenObject;
    }
    
    public void ClearKitchenObject()
    {
        kitchenObject = null;
    }
    
    public bool HasKitchenObject()
    {
        return kitchenObject != null;
    }
}
```

在 KitchenObject.cs 中，需要将 ClearCounter 相关的全都改为 kitchenObjectParent 相关的名称

```
// KitchenObject.cs
using UnityEngine;

public class KitchenObject : MonoBehaviour
{
    [SerializeField] private KitchenObjectSO kitchenObjectSO;
    
    private IKitchenObjectParent kitchenObjectParent;
    
    public KitchenObjectSO GetKitchenObjectSO()
    {
        return kitchenObjectSO;
    }
    
    public void SetKitchenObjectParent(IKitchenObjectParent kitchenObjectParent)
    {
        if (this.kitchenObjectParent != null)
        {
            this.kitchenObjectParent.ClearKitchenObject();
        }
        this.kitchenObjectParent = kitchenObjectParent;

        if (kitchenObjectParent.HasKitchenObject())
        {
            Debug.LogError("IKitchenObjectParent already has a KitchenObject!");
        }
        kitchenObjectParent.SetKitchenObject(this);
        
        transform.parent = kitchenObjectParent.GetKitchenObjectFollowTransform();
        transform.localPosition = Vector3.zero;
    }
    
    public IKitchenObjectParent GetKitchenObjectParent()
    {
        return kitchenObjectParent;
    }
}
```

目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#playerPickUp)

## 9 创建容器柜台 Container Counter

### 9.1 柜台预制件变体 Prefab Viriant

容器柜台和空柜台都是柜台，因此我们可以直接复制 ClearCounter.prefab 然后更改一些东西，但是 unity 为我们提供了一个叫 [prefab variant](https://docs.unity3d.com/cn/current/Manual/PrefabVariants.html) 的东西，我们这里就创建一个基本柜台的 prefab，然后使用 prefab variant 来创建不同的柜台  
复制 ClearCounter.prefab，重命名为_BaseCounter.prefab，作为基类，该基类除了要保留 Box Collider 组件和空物体 CounterTopPoint，其余全部删除

![](<images/1689211712340.png>)

  
在 Project 窗口中右键_BaseCounter.prefab->Create->Prefab Variant 基于这个基类创建一个 prefab，将之前 ClearCounter.prefab 中的内容复制到这个新创建的 prefab 上，并把脚本都设置好，然后删除之前的 ClearCounter.prefab，将新的 prefab 命名为 ClearCounter.prefab

![](<images/1689211712451.png>)

  
再次右键_BaseCounter.prefab->Create->Prefab Variant 基于这个基类创建 ContainerCounter.prefab，将 ContainerCounter_Visual 拖到 ContainerCounter 下面作为它的子级，复制一份 ContainerCounter_Visual 重命名为 Selected，将 Selected 下的模型的材质都改为 CounterSelected 并默认隐藏，将 Selected 缩放均改为 1.01，添加 SelectedCounterVisual.cs 组件

![](<images/1689211712542.png>)

### 9.2 继承自柜台基类创建 ClearCounter 与 ContainerCounter

接下来我们在 Scripts 文件夹下创建 ContainerCounter.cs，我们可以让这个脚本中的内容和 ClearCounter.cs 中的内容一致来进行测试，原教程中先尝试了这么做，但是在写完之后容器柜台并不能正常与角色交互，这时因为在 Player.cs 中我们进行 Raycast 的时候检测的就是射线是否与 ClearCounter 类的实例碰撞，我们当然可以让它也检测射线与 ContainerCounter 类的实例是否碰撞，但是 ContainerCounter 与 ClearCounter 的行为很类似，我们可以写一个基类 BaseCounter，将所有柜台都需要实现的逻辑放在基类中，然后其他柜台类都继承自这个类，我们在检测射线与 BaseCounter 类的实例是否碰撞时，所有柜台就都能被检测到了，通过继承，我们也能够更好地组织代码  
在 Scripts 文件夹下新建 BaseCounter.cs，在这个基类中，我们可以写一个虚方法 public virtual void Interact(Player player)，所有继承自该基类的类都需要使用 override 重写这个方法，并且由于所有柜台类都有 counterTopPoint，都实现了 IKitchenObjectParent 接口，我们也把这两部分都放到这个基类中

```
// BaseCounter.cs中
using UnityEngine;

public class BaseCounter : MonoBehaviour, IKitchenObjectParent
{
    [SerializeField] private Transform counterTopPoint;
    
    private KitchenObject kitchenObject;
    
    public virtual void Interact(Player player)
    {
        Debug.Log("BaseCounter.Interact()");
    }

    public Transform GetKitchenObjectFollowTransform()
    {
        return counterTopPoint;
    }
    
    public void SetKitchenObject(KitchenObject kitchenObject)
    {
        this.kitchenObject = kitchenObject;
    }

    public KitchenObject GetKitchenObject()
    {
        return kitchenObject;
    }
    
    public void ClearKitchenObject()
    {
        kitchenObject = null;
    }
    
    public bool HasKitchenObject()
    {
        return kitchenObject != null;
    }
}
```

在 ClearCounter.cs 中，改为角色放置和拾取物品的逻辑

```
// ClearCounter.cs中
using UnityEngine;

public class ClearCounter : BaseCounter
{
    [SerializeField] private KitchenObjectSO kitchenObjectSO;

    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                player.GetKitchenObject().SetKitchenObjectParent(this);
            } else
            {
                // 角色没有物品
            }
        } else
        {
            // 柜子上有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品
            } else
            {
                // 角色没有物品，拾取物品
                GetKitchenObject().SetKitchenObjectParent(player);
            }
        }
    }
}
```

在 ContainerCounter.cs 中，只需要实现按下交互按键时该容器柜子对应的物品出现在角色手上的逻辑

```
// ContainerCounter.cs中
using UnityEngine;

public class ContainerCounter : BaseCounter
{
    [SerializeField] private KitchenObjectSO kitchenObjectSO;

    if (!player.HasKitchenObject())
	 {
		 // 角色没有物品，拾取物品
		 Transform kitchenObjectTransform = Instantiate(kitchenObjectSO.prefab);
		 kitchenObjectTransform.GetComponent<KitchenObject>().SetKitchenObjectParent(player);
	 }
}
```

在 SelectCounterVisual.cs 中，我们之前使用的是 ClearCounter 类的实例，这样 ContainerCounter 是无法被正确选中的，现在要换成 BaseCounter，同时我们的 ContainerCounter 模型有好几部分组成，因此在设置选中效果的模型时，我们需要传入一个数组，遍历这个数组将其中所有的对象设为显示或隐藏（注释部分是修改之前的代码）

```
// SelectCounterVisual.cs中
using UnityEngine;

public class SelectCounterVisual : MonoBehaviour
{
    // [SerializeField] private ClearCounter clearCounter;
    [SerializeField] private BaseCounter baseCounter;
    // [SerializeField] private GameObject visualGameObject;
    [SerializeField] private GameObject[] visualGameObjectArray;
    
    private void Start()
    {
        Player.Instance.OnSelectedCounterChanged += Player_OnSelectedCounterChanged;
    }
    
    private void Player_OnSelectedCounterChanged(object sender, Player.OnSelectedCounterChangedEventArgs e)
    {
        // if (e.selectedCounter == clearCounter)
        if (e.selectedCounter == baseCounter)
        {
            show();
        }
        else
        {
            hide();
        }
    }

    private void show()
    {
        // visualGameObject.SetActive(true);
        foreach (GameObject visualGameObject in visualGameObjectArray)
        {
            visualGameObject.SetActive(true);
        }
    }
    private void hide()
    {
        // visualGameObject.SetActive(false);
        foreach (GameObject visualGameObject in visualGameObjectArray)
        {
            visualGameObject.SetActive(false);
        }
    }
}
```

代码之外，在 ContainerCounter.prefab 上挂载上对应的脚本

![](<images/1689211712648.png>)

  
由于我们改变了 SelectCounterVisual.cs 这个脚本，原先的 ClearCounter 上的相应组件也要重新设置一下

![](<images/1689211712804.png>)

  
在场景中放置两个 ContainerCounter，其中一个将上面的 Sprite 改为番茄，将这个柜台的 CountainerCounter 组件中的 SkitchenObjectSO 也改为 Tomato，让我们能够从中拿取番茄

![](<images/1689211712903.png>)

  
现在运行游戏，我们就可以从对应的柜台中获取对应的物品了

![](<images/1689211712963.png>)

### 9.3 容器柜台的动画

我们的 ContainerCounter 素材实际上已经包含了一个打开柜子的动画，直接双击 ContainerCounter_Visual 上 Animator 组件的 Controller 部分，可以看到 Animator 面板中已经设置好的状态机，这些动画通过 OpenClose 这个参数来控制开关

![](<images/1689211713087.png>)

  
在 Scripts 文件夹新建 ContainerCounterVisual.cs，在 ContainerCounter.cs 中触发事件，在 ContainerCounterVisual.cs 中订阅事件

```
// ContainerCounter.cs中
using System;
using UnityEngine;

public class ContainerCounter : BaseCounter
{
    public event EventHandler OnPlayerGrabbedObject;
    
    [SerializeField] private KitchenObjectSO kitchenObjectSO;

    public override void Interact(Player player)
    {
        if (!player.HasKitchenObject())
        {
            // 角色没有物品，拾取物品
            Transform kitchenObjectTransform = Instantiate(kitchenObjectSO.prefab);
            kitchenObjectTransform.GetComponent<KitchenObject>().SetKitchenObjectParent(player);
            OnPlayerGrabbedObject?.Invoke(this, EventArgs.Empty);
        }
    }
}
```

```
// ContainerCounterVisual.cs中
using System;
using UnityEngine;

public class ContainerCounterVisual : MonoBehaviour
{
    private const string OPEN_CLOSE = "OpenClose";
    
    [SerializeField] private ContainerCounter containerCounter;
    
    private Animator animator;

    private void Awake()
    {
        animator = GetComponent<Animator>();
    }

    private void Start()
    {
        containerCounter.OnPlayerGrabbedObject += ContainerCounter_OnPlayerGrabbedObject;
    }
    
    private void ContainerCounter_OnPlayerGrabbedObject(object sender, EventArgs e)
    {
        animator.SetTrigger(OPEN_CLOSE);
    }
}
```

再次运行游戏，就可以看到对应的动画了

![](<images/1689211713297.png>)

### 9.4 容器柜台预制件变体

之前我们是通过复制容器柜台然后更改上面的 Sprite 创建新的容器柜台的，为了方便起见我们也应该以容器柜台为基类去创建 Prefab Viriant  
通过 Script Object 给其他食材物品创建对象（Bread、Cabbage、MeatPattyUncooked），复制现有的食材的 Prefab，替换组件中的 KitchenObjectSO 项，然后将模型更改为 KitchenObjectsVisuals 中对应的模型，如 Bread.prefab 和 Bread.asset 更改后如下

![](<images/1689211713340.png>)

  
所有食材物品创建好后应该如下所示

![](<images/1689211713467.png>)

  
然后右键 ContainerCounter.prefab->Create->Prefab Viriant 给对应食材创建对应容器柜台的 Prefab Viriant，更改每个 Prefab Viriant 组件中的 KitchenObjectSO 项与 Sprite

![](<images/1689211713652.png>)

![](<images/1689211713818.png>)

  
将各种柜台摆放在场景中，运行游戏，我们就能够从各个容器柜台中拿取对应的食材了

![](<images/1689211713958.png>)

目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#playerPickUp))

## 10 创建切菜台 Cutting Counter

### 10.1 继承自柜台基类创建 CuttingCounter

右键_BaseCounter.prefab->Create->Prefab Variant 创建一个新的 prefab，重命名为 CuttingCounter.prefab，从_Assets/PrefabsVisuals/CounterVisuals 找到 CuttingCounter_Visual，拖动到 CuttingCounter.prefab 下，复制一个重命名为 Selected，像之前一样给 Selected 加上 SelectedCounterVisual.cs 组件，将其中的模型材质换为 CounterSelected，将 Selected 的缩放改为 1.01

![](<images/1689211714061.png>)

  
在 Scripts 创建 CuttingCounter.cs，同样要继承自 BaseCounter 基类，暂时先放上与 ClearCounter.cs 相同的交互逻辑

```
// CuttingCounter.cs中
using UnityEngine;

public class CuttingCounter : BaseCounter
{
    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                player.GetKitchenObject().SetKitchenObjectParent(this);
            } else
            {
                // 角色没有物品
            }
        } else
        {
            // 柜子上有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品
            } else
            {
                // 角色没有物品，拾取物品
                GetKitchenObject().SetKitchenObjectParent(player);
            }
        }
    }
}
```

在 CuttingCounter 上添加 CuttingCounter.cs 组件，将 CuttingCounter 拖动到 Selected 的组件中的 BaseCounter 项上，开始游戏，就可以看到实现了和 ClearCounter 相同的交互效果

### 10.2 CutKitchenObject 与简单的切菜交互

要实现切菜交互的效果，我们需要删除放在切菜台上的食材，替换为切片的相应的食材的模型，然后播放切菜的循环动画，这里我们先来实现按下相应按键番茄替换为番茄切片的简单效果，为此我们需要创建相应的 ScriptableObject 与 prefab  
先来做一个番茄的切片素材的 ScriptableObject 与 prefab，在 ScriptableObjects/KitchenObjectsSO 下右键 ->Create->Kitchen Object SO，重命名为 TomatoSlices

![](<images/1689211714379.png>)

  
在 Setting/PlayInputActions.inputactions 中，添加一个 Action，绑定 F 按键

![](<images/1689211714523.png>)

  
要实现按下 F 键物体替换为物体切片的效果，先要实现按下 F 键删除原物体的效果，可以按照写上一个 Interact 的 Action 差不多的方式去完成。我们在 BaseCounter.cs 这个基类中写了一个 InteractAlternate() 的接口，在 CuttingCounter.cs 中实现了该接口。在 GameInput.cs 中添加按下 F 键将会触发的函数，这里这个函数中我们触发一个 EventHandler 委托，在 Player.cs 中在该委托上添加会调用的函数 GameInput_OnInteractAlternateAction()，在函数中使用 selectedCounter.InteractAlternate(this) 调用具体方法。

```
// BaseCounter.cs中
...
public class BaseCounter : MonoBehaviour, IKitchenObjectParent
{
    ...
    public virtual void InteractAlternate(Player player) {
        Debug.LogError("BaseCounter.InteractAlternate()"); // 不是所有的柜台都有这个交互，所以不应该输出Error信息，一开始作者是输出的，但是在后面改了
    }
    ...
}
```

```
// CuttingCounter.cs中
...
public class CuttingCounter : BaseCounter
{
    [SerializeField] private KitchenObjectSO cutKitchenObjectSO;
    
    public override void Interact(Player player)
    {
        ...
    }
    
    public override void InteractAlternate(Player player)
    {
        if (HasKitchenObject())
        {
            // 柜子上有物品，开始切菜
            GetKitchenObject().DetroySelf();
        }
    }
}
```

```
// GameInput.cs中
public class GameInput : MonoBehaviour
{
    ...
    public event EventHandler OnInteractAlternateAction;
    ...
    private void Awake()
    {
        ...
        playerInputActions.Player.InteractAlternate.performed += InteractAlternate_performed;
    }
    ... 
    private void InteractAlternate_performed(UnityEngine.InputSystem.InputAction.CallbackContext obj)
    {
        OnInteractAlternateAction?.Invoke(this, EventArgs.Empty);
    }
    ...
}
```

```
// Player.cs中
...
public class Player : MonoBehaviour, IKitchenObjectParent
{
    ...

    private void Start()
    {
        ...
        gameInput.OnInteractAlternateAction += GameInput_OnInteractAlternateAction;
    }
    
    ...
    
    private void GameInput_OnInteractAlternateAction(object sender, EventArgs e)
    {
        if (selectedCounter != null)
        {
            selectedCounter.InteractAlternate(this);
        }
    }
    ...
}
```

接下来我们还要实现对应的物品切片模型出现的效果，我们在 KitchenObject.cs 中写一个 SpawnKitchenObject() 方法，然后在 CuttingCounter.cs 的 InteractAlternate() 方法中调用

```
// KitchenObject.cs中
...
public class KitchenObject : MonoBehaviour
{
    ...
    public static KitchenObject SpawnKitchenObject(KitchenObjectSO kitchenObjectSO, IKitchenObjectParent kitchenObjectParent) {
        Transform kitchenObjectTransform = Instantiate(kitchenObjectSO.prefab);
        KitchenObject kitchenObject = kitchenObjectTransform.GetComponent<KitchenObject>();
        kitchenObject.SetKitchenObjectParent(kitchenObjectParent);
        return kitchenObject;
    }
}
```

```
// CuttingCounter.cs中
...
public class CuttingCounter : BaseCounter
{
    [SerializeField] private KitchenObjectSO cutKitchenObjectSO;
    
    public override void Interact(Player player)
    {
        ...
    }
    
    public override void InteractAlternate(Player player)
    {
        if (HasKitchenObject())
        {
            // 柜子上有物品，开始切菜
            GetKitchenObject().DetroySelf();
            KitchenObject.SpawnKitchenObject(cutKitchenObjectSO, this);
        }
    }
}
```

至于要出现的切片模型我们先来往场景拖一个 CuttingCounter.prefab 然后规定死测试一下，就用上面做好的番茄切片

![](<images/1689211714795.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#cuttingCounterInteractAlternate)

### 10.3 CuttingRecipeSO

为了使我们的代码更易扩展，我们可以给把这些表示物品对象到对应物体切片对象的信息用 ScriptableObject 储存  
首先先把几个物品切片的 ScriptbaleObject 创建好，在 KitchenObjectSO 下右键 ->Create->Kitchen Object SO 创建 CabbageSlices.asset、CheeseSlices.asset，并在 Prefab/KitchenObjects 中创建相应的 Prefab 填好信息

![](<images/1689211715004.png>)

  
我们还需要知道正常物品与切片物品的对应关系，在 Scripts 文件夹下新建 CuttingRecipeSO.cs，其中我们需要两个 ScriptableObject，一个表示正常的物品对象，一个表示该物品对应的物品切片对象

```
// CuttingRecipeSO.cs
using UnityEngine;

[CreateAssetMenu()]
public class CuttingRecipeSO : ScriptableObject
{
    public KitchenObjectSO input;
    public KitchenObjectSO output;
}
```

在 ScriptableObjects 文件夹下新建 CuttingRecipeSO，在该文件夹下右键 ->Create->Cutting Recipe SO，创建 Tomato-TomatoSilices.asset、Cabbage-CabbageSlices.asset、CheeseBlock-CheeseSlices.asset 并设置好相应的变量

![](<images/1689211715318.png>)

  
在 CuttingCounter.cs 中，我们使用一个数组来保存 CuttingRecipeSO 对象的实例，然后写一个 GetOutputForInput() 方法用 foreach 来获取对应输入物体的切片版本

```
// CuttingCounter.cs中
...
public class CuttingCounter : BaseCounter
{
    // [SerializeField] private KitchenObjectSO cutKitchenObjectSO;
    [SerializeField] private CuttingRecipeSO[] cuttingRecipeSOArray;
    ...
    public override void InteractAlternate(Player player)
    {
        if (HasKitchenObject())
        {
            // 柜子上有物品，开始切菜
            KitchenObjectSO outputKitchenObjectSO = GetOutputForInput(GetKitchenObject().GetKitchenObjectSO());
            GetKitchenObject().DetroySelf();
            KitchenObject.SpawnKitchenObject(outputKitchenObjectSO, this);
        }
    }

    private KitchenObjectSO GetOutputForInput(KitchenObjectSO inputKitchenObjectSO)
    {
        foreach (CuttingRecipeSO cuttingRecipeSO in cuttingRecipeSOArray)
        {
            if (cuttingRecipeSO.input == inputKitchenObjectSO)
            {
                return cuttingRecipeSO.output;
            }
        }
        return null;
    }
}
```

设置 CuttingCounter 组件对应变量

![](<images/1689211715395.png>)

  
运行游戏，可以看到当按下 F 键时，我们可以将对应物品切片了

![](<images/1689211715507.png>)

  
然而，并不是所有的物品都可以被切片，如果遇到不能被切片的物品或者已经被切过的切片我们按下 F 键就会报错，我们在 CuttingCounter.cs 中添加一个 HasRecipeInput() 方法来检查对应物品是否有切片形式，在 Interact() 和 InteractAlternate() 中角色放置物品时进行检查

```
// CuttingCounter.cs
using UnityEngine;

public class CuttingCounter : BaseCounter
{
    [SerializeField] private CuttingRecipeSO[] cuttingRecipeSOArray;
    
    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                if (HasRecipeWithInput(player.GetKitchenObject().GetKitchenObjectSO()))
                {
                    // 角色拿着的物品可以被切片
                    player.GetKitchenObject().SetKitchenObjectParent(this);
                }
            } else
            {
                // 角色没有物品
            }
        } else
        {
            // 柜子上有物品
            ...
        }
    }
    
    public override void InteractAlternate(Player player)
    {
        // if (HasKitchenObject())
        if (HasKitchenObject() && HasRecipeWithInput(GetKitchenObject().GetKitchenObjectSO()))
        {
            // 柜子上有物品，开始切菜
            ...
        }
    }
    
    private bool HasRecipeWithInput(KitchenObjectSO inputKitchenObjectSO)
    {
        foreach (CuttingRecipeSO cuttingRecipeSO in cuttingRecipeSOArray)
        {
            if (cuttingRecipeSO.input == inputKitchenObjectSO)
            {
                return true;
            }
        }
        return false;
    }
    
    ...
}
```

目前为止，我们就不能在切菜台上放置不能切的物品，并且不能切已经切过的物品了  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#cuttingRecipeSO)

### 10.4 使用 Canvas 绘制切菜进度条

现在我们的菜一下就切好了，我们希望不同的食材需要按不同次 F 键才能被切成片，在 CuttingRecipeSO.cs 中，增加一个变量 cuttingProgressMax 保存这种食材应该切的次数

```
// CuttingRecipeSO.cs中
using UnityEngine;

[CreateAssetMenu()]
public class CuttingRecipeSO : ScriptableObject
{
    public KitchenObjectSO input;
    public KitchenObjectSO output;
    public int cuttingProgressMax;
}
```

在 CuttingCounter.cs 中，我们用 cuttingProgress 保存切菜的按键数量，该数量在每次在 Interact() 方法被调用时重置为 0，在每次调用 InteractAlternate() 方法时 + 1，由于我们需要频繁获取每样物品的对应的 ScriptObject，这里将 GetCuttingRecipeSOWithInput() 这个方法抽象了出来

```
// CuttingCounter.cs中
using UnityEngine;

public class CuttingCounter : BaseCounter
{
    [SerializeField] private CuttingRecipeSO[] cuttingRecipeSOArray;
    
    private int cuttingProgress;
    
    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                if (HasRecipeWithInput(player.GetKitchenObject().GetKitchenObjectSO()))
                {
                    // 角色拿着的物品可以被切片
                    ...
                    cuttingProgress = 0;
                    ...
                }
            } else
            {
                ...
            }
        } else
        {
            ...
        }
    }
    
    public override void InteractAlternate(Player player)
    {
        if (HasKitchenObject() && HasRecipeWithInput(GetKitchenObject().GetKitchenObjectSO()))
        {
            // 柜子上有物品，开始切菜
            cuttingProgress++;
            
            if (cuttingProgress >= GetCuttingRecipeSOWithInput(GetKitchenObject().GetKitchenObjectSO()).cuttingProgressMax)
            {
                KitchenObjectSO cutKitchenObjectSO = GetOutputForInput(GetKitchenObject().GetKitchenObjectSO());
                GetKitchenObject().DetroySelf();
                KitchenObject.SpawnKitchenObject(cutKitchenObjectSO, this);
            }
        }
    }
    
    private bool HasRecipeWithInput(KitchenObjectSO inputKitchenObjectSO)
    {
        CuttingRecipeSO cuttingRecipeSO = GetCuttingRecipeSOWithInput(inputKitchenObjectSO);
        return cuttingRecipeSO != null;
    }
    
    private KitchenObjectSO GetOutputForInput(KitchenObjectSO inputKitchenObjectSO)
    {
        CuttingRecipeSO cuttingRecipeSO = GetCuttingRecipeSOWithInput(inputKitchenObjectSO);
        if (cuttingRecipeSO != null)
        {
            return cuttingRecipeSO.output;
        } else
        {
            return null;
        }
    }
    
    private CuttingRecipeSO GetCuttingRecipeSOWithInput(KitchenObjectSO inputKitchenObjectSO)
    {
        foreach (CuttingRecipeSO cuttingRecipeSO in cuttingRecipeSOArray)
        {
            if (cuttingRecipeSO.input == inputKitchenObjectSO)
            {
                return cuttingRecipeSO;
            }
        }
        return null;
    }
}
```

在编辑器中，为三个 CuttingRecipeSO 的 CuttingProgressMax 赋予不同的值，比如这里将番茄与奶酪设置为 5，卷心菜设置为 3

![](<images/1689211715563.png>)

  
加上一点控制台输出，现在启动游戏就能看到不同食材需要切不同次数了

![](<images/1689211715769.png>)

  
接下来来使用 World Canvas 将切菜的进度用进度条绘制出来  
进入 CuttingCounter.prefab 的编辑界面，Hierachy 窗口右键 ->UI->Canvas 创建一个 Canvas 对象，重命名为 ProgressBarUI，将 Canvas 组件下的 Render Mode 改为 World Space，然后调整位置到柜台上方并将长宽设置为 0 用于定位；在 ProgressBarUI 下新建 Cavans，重命名为 Bar，在 Bar 中调整大小、设置 Image 组件中的 Color、Source Image、Fill Method，滑动调整 Fill Amount 的值即可看到场景中进度条的变化；复制 Bar，重命名为 Background，放在 Bar 上方以确保渲染顺序正确，添加 Outline 组件并设置 Effect Color 的 Alpha 值为 0，调整 Effect Distance 到合适值，将 Image 组件中的 Color 改为深灰色。

![](<images/1689211716101.png>)

  
新建完 Canvas，我们需要一个写一个脚本来根据切菜的进度来控制进度条。在 Scripts 文件夹下新建 ProgressBarUI.cs，挂载到 Prefab 中的 ProgressBarUI 上，在 CuttingCounter.cs 中添加 EventHandler 委托，在每一次切菜时触发事件并传递归一化后的进度值，在 ProgressBarUI.cs 中写用这个进度值控制 fillAmount 值的逻辑与进度条隐藏显示的逻辑

```
// CuttingCounter.cs中
...
public class CuttingCounter : BaseCounter
{
    public event EventHandler<OnProgressChangedEventArgs> OnProgressChanged;
    public class OnProgressChangedEventArgs : EventArgs
    {
        public float progressNormalized;
    }
    ...
    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                if (HasRecipeWithInput(player.GetKitchenObject().GetKitchenObjectSO()))
                {
                    // 角色拿着的物品可以被切片
                    ...
                    
                    CuttingRecipeSO cuttingRecipeSO = GetCuttingRecipeSOWithInput(GetKitchenObject().GetKitchenObjectSO());
                    OnProgressChanged?.Invoke(this, new OnProgressChangedEventArgs
                    {
                        progressNormalized = (float)cuttingProgress / cuttingRecipeSO.cuttingProgressMax
                    });
                }
            } else
            {
                ...
            }
        } else
        {
            ...
        }
    }
    
    public override void InteractAlternate(Player player)
    {
        if (HasKitchenObject() && HasRecipeWithInput(GetKitchenObject().GetKitchenObjectSO()))
        {
            // 柜子上有物品，开始切菜
            ...
            OnProgressChanged?.Invoke(this, new OnProgressChangedEventArgs
            {
                CuttingRecipeSO cuttingRecipeSO = GetCuttingRecipeSOWithInput(GetKitchenObject().GetKitchenObjectSO());
                progressNormalized = (float)cuttingProgress / cuttingRecipeSO.cuttingProgressMax
            });
            ...
        }
    }
    ...
}
```

```
// ProgressBarUI.cs中
using UnityEngine;
using UnityEngine.UI;

public class ProgressBarUI : MonoBehaviour
{
    [SerializeField] private CuttingCounter cuttingCounter;
    [SerializeField] private Image barImage;

    private void Start()
    {
        cuttingCounter.OnProgressChanged += CuttingCounter_OnProgressChanged;
        
        barImage.fillAmount = 0;
        Hide();
    }
    
    private void CuttingCounter_OnProgressChanged(object sender, CuttingCounter.OnProgressChangedEventArgs e)
    {
        barImage.fillAmount = e.progressNormalized;
        
        if (e.progressNormalized == 0f || e.progressNormalized == 1)
        {
            Hide();
        } else
        {
            Show();
        }
    }

    private void Show()
    {
        gameObject.SetActive(true);
    }
    
    private void Hide()
    {
        gameObject.SetActive(false);
    }
}
```

目前为止运行游戏就可以看到带有进度条的切菜台了

![](<images/1689211716490.png>)

### 10.5 切菜台的动画

目前为止运行游戏我们已经能够在将物品放到切菜台上按下 F 键时看到进度条了，另外素材中还有一个切菜的动画，其中 CuttingCouterCut 为刀切菜的动画，CuttingCounterIdle 为静止状态

![](<images/1689211716529.png>)

  
打开 Animations/CuttingCounter.controller，在 Animator 面板可以看到，动画由 AnyState 通过一个 Trigger 进入 CuttingCounterCut 状态，然后执行完该动画后就会再次回到 Idle 状态

![](<images/1689211716756.png>)

  
在 Scripts 文件夹新建 CuttingCounterVisual.cs，在 CuttingCounter.cs 中添加 EventHandler 委托，该委托在开始切菜时触发，委托执行的函数 CuttingCounter_OnCut() 写在 CuttingCounterVisual.cs 中，该函数将触发控制动画的 Trigger

```
// CuttingCounter.cs
...
public class CuttingCounter : BaseCounter
{
    ...
    public event EventHandler OnCut;
    ...   
    public override void InteractAlternate(Player player)
    {
        if (HasKitchenObject() && HasRecipeWithInput(GetKitchenObject().GetKitchenObjectSO()))
        {
            // 柜子上有物品，开始切菜
            ...
            OnCut?.Invoke(this, EventArgs.Empty);
            ...
        }
    }
    ...
}
```

```
// CuttingCounterVisual.cs中
using System;
using UnityEngine;

public class CuttingCounterVisual : MonoBehaviour
{
    private const string CUT = "Cut";

    [SerializeField] private CuttingCounter cuttingCounter;
    
    private Animator animator;

    private void Awake()
    {
        animator = GetComponent<Animator>();
    }

    private void Start()
    {
        cuttingCounter.OnCut += CuttingCounter_OnCut;
    }

    private void CuttingCounter_OnCut(object sender, EventArgs e)
    {
        animator.SetTrigger(CUT);
    }
}
```

运行游戏，就可以看到带有动画和进度条的切菜台了

![](<images/1689211717051.png>)

  
[目前为止的工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#cuttingProgressWorldCanvas)

### 10.6 切菜进度条朝向摄像机

目前我们的切菜台的进度条还有一个问题，如果我们 180 度旋转该柜台，我们的 UI 也会跟着旋转，我们会从背面看到进度条，进度条会呈现从右向左的状态

![](<images/1689211717246.png>)

  
为了解决这个问题我们可以创建一个脚本，让进度条始终面朝摄像机。在 Scripts 文件夹新建 LookAtCamera.cs，添加到 CuttingCounter.prefab 的 ProgressBarUI 上，其中 LateUpdate() 会在所有 Update() 调用后被调用

```
// LookAtCamera.cs中
using UnityEngine;

public class LookAtCamera : MonoBehaviour
{
    private void LateUpdate() {
        transform.LookAt(Camera.main.transform);
    }
}
```

运行游戏，发现进度条确实朝向了摄像机，但是有两个问题，一个是我们的 Canvas 其实默认是背面朝摄像机的，使用 LookAt 函数会导致进度条变为从右向左的样式，另一个问题是，由于它朝向了摄像机，所以在画面中并不是完全水平的，而是会随着摄像机的左右移动而左右倾斜

![](<images/1689211717319.png>)

  
上面的效果具体要哪种是个人偏好问题，我们可以使用枚举类型和 switch 语句设置一些选项供我们在编辑器中选择，其中进度条左右翻转的问题通过看向完全相反的位置修复，左右倾斜的问题通过看向摄像机朝向方向修复

```
// LookAtCamera.cs中
using UnityEngine;

public class LookAtCamera : MonoBehaviour
{
    private enum Mode
    {
        LookAt,
        LookAtInverted,
        CameraForward,
        CameraForwardInverted
    }

    [SerializeField] private Mode mode;
    
    private void LateUpdate()
    {
        switch (mode)
        {
            case Mode.LookAt:
                transform.LookAt(Camera.main.transform);
                break;
            case Mode.LookAtInverted:
                Vector3 dirFromCamera = transform.position - Camera.main.transform.position;
                transform.LookAt(transform.position + dirFromCamera);
                break;
            case Mode.CameraForward:
                transform.forward = Camera.main.transform.forward;
                break;
            case Mode.CameraForwardInverted:
                transform.forward = -Camera.main.transform.forward;
                break;
        }
    }
}
```

这里选择 CameraForward 模式来得到从左到右的、水平的进度条

![](<images/1689211717604.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#lookAtCamera)

## 11 创建垃圾箱 Trash Counter

垃圾箱的逻辑很简单，只需要按下交互按键时销毁角色手上的物品即可  
右键_BaseCounter.prefab->Create->Prefab Variant 创建一个新的 prefab，重命名为 TrashCounter.prefab，从_Assets/PrefabsVisuals/CounterVisuals 找到 TrashCounter_Visual，拖动到 TrashCounter.prefab 下，复制一个重命名为 Selected，像之前一样给 Selected 加上 SelectedCounterVisual.cs 组件，将其中的模型材质换为 CounterSelected，将 Selected 的缩放改为 1.01

![](<images/1689211717748.png>)

  
在 Scripts 创建 TrashCounter.cs，同样要继承自 BaseCounter 基类，当按下交互键时检查如果角色手上有物品则销毁

```
// CuttingCounter.cs中
public class TrashCounter : BaseCounter
{
    public override void Interact(Player player)
    {
        if (player.HasKitchenObject())
        {
            player.GetKitchenObject().DetroySelf();
        }
    }
}
```

顺便可以把 Scripts 文件夹下的文件整理一下，创建 Counters 文件夹和 ScriptableObjets 文件夹，将相应的文件分类放好

![](<images/1689211718189.png>)

  
在 TrashCounter 上添加 TrashCounter.cs 组件，将 TrashCounter 拖动到 Selected 的组件中的 BaseCounter 项上，开始游戏，就可以看到实现了垃圾箱的效果

![](<images/1689211718318.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#trashCounter)

## 12 创建炉灶台 Stove Counter

### 12.1 继承自柜台基类创建 StoveCounter 与简单的计时器

与上面创建其他 Counter 相同，从_BaseCounter.prefab 创建 PrefabVariant、复制一个 Selected，改缩放，将 CounterTopPoint 移动到炉灶台上锅的中心处，在 Scripts 文件夹下新建 StoveCounter.cs 继承自 BaseCounter，挂载好对应脚本

![](<images/1689211718488.png>)

  
我们需要将要经过炉灶台的生的食材变成熟的，这个转换关系也可以用 Scriptable Object 来存储。在 Scripts 文件夹下新建 FryingRecipeSO.cs

```
// FryingRecipeSO.cs
using UnityEngine;

[CreateAssetMenu()]
public class FryingRecipeSO : ScriptableObject
{
    public KitchenObjectSO input;
    public KitchenObjectSO output;
    public float fryingTimerMax;
}
```

在 Scriptbale Object 文件夹下新建 FryingObjectSO 文件夹，在这个文件夹下右键 ->Create->Frying Recipe SO，新建 MeatPattyUncooked-MeatPattyCooked.asset，将 MeatPattyCooked 和 MeatPattyBurned 的 Prefab 和 KitchenObjectSO 都创建好，将所有要设置的参数都设置好

![](<images/1689211718893.png>)

  
在 StoveCounter.cs 中，我们仿照 CuttingCouter.cs 写出 Interact()、HasRecipeWithInput()、GetOutputForInput()、GetFryingRecipeSOWithInput() 几个函数，然后声明一个 fryingTimer，在 Update() 中随时间增加，一旦到达了设定的值就会销毁原物品生成被煎好的版本（这里计时器也可以使用协程写，但是作者更喜欢直接声明变量）

```
// StoveCounter.cs中
using UnityEngine;

public class StoveCounter : BaseCounter
{
    [SerializeField] private FryingRecipeSO[] fryingRecipeSOArray;
    
    private float fryingTimer;
    private FryingRecipeSO fryingRecipeSO;

    private void Update()
    {
        if (HasKitchenObject())
        {
            fryingTimer += Time.deltaTime;
            if (fryingTimer > fryingRecipeSO.fryingTimerMax)
            {
                // 煎好了
                fryingTimer = 0f;
                Debug.Log("Fried!");
                GetKitchenObject().DetroySelf();
                KitchenObject.SpawnKitchenObject(fryingRecipeSO.output, this);
            }
            Debug.Log(fryingTimer);
        }
    }

    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                if (HasRecipeWithInput(player.GetKitchenObject().GetKitchenObjectSO()))
                {
                    // 角色拿着的物品可以被煎
                    player.GetKitchenObject().SetKitchenObjectParent(this);
                    fryingRecipeSO = GetFryingRecipeSOWithInput(GetKitchenObject().GetKitchenObjectSO());
                }
            } else
            {
                // 角色没有物品
            }
        } else
        {
            // 柜子上有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品
            } else
            {
                // 角色没有物品，拾取物品
                GetKitchenObject().SetKitchenObjectParent(player);
            }
        }
    }
    
    private bool HasRecipeWithInput(KitchenObjectSO inputKitchenObjectSO)
    {
        FryingRecipeSO fryingRecipeSO = GetFryingRecipeSOWithInput(inputKitchenObjectSO);
        return fryingRecipeSO != null;
    }
    
    private KitchenObjectSO GetOutputForInput(KitchenObjectSO inputKitchenObjectSO)
    {
        FryingRecipeSO fryingRecipeSO = GetFryingRecipeSOWithInput(inputKitchenObjectSO);
        if (fryingRecipeSO != null)
        {
            return fryingRecipeSO.output;
        } else
        {
            return null;
        }
    }
    
    private FryingRecipeSO GetFryingRecipeSOWithInput(KitchenObjectSO inputKitchenObjectSO)
    {
        foreach (FryingRecipeSO fryingRecipeSO in fryingRecipeSOArray)
        {
            if (fryingRecipeSO.input == inputKitchenObjectSO)
            {
                return fryingRecipeSO;
            }
        }
        return null;
    }
}
```

这样运行游戏还有一些 bug，比如我们在计时器到了之后便会重新开始计时，等到第二次计时器到了后会生成第二个被煎好的食材，不过我们已经能看到被煎熟的效果，下面会通过一个状态机来实现正确的效果

![](<images/1689211719532.png>)

### 12.2 使用状态机控制食物状态

为了实现煎糊的效果，我们还需要创建一个 ScriptableObject 用来保存煎熟的和煎糊的食材间的对应关系，在 Scripts/ScriptableObjects 中，新建 BurningRecipeSO.cs

```
// BurningRecipeSO.cs中
using UnityEngine;

[CreateAssetMenu()]
public class BurningRecipeSO : ScriptableObject
{
    public KitchenObjectSO input;
    public KitchenObjectSO output;
    public float burningTimerMax;
}
```

在 ScriptableObjects 下新建 BurningRecipeSO 文件夹，在该文件夹下新建 MeatPattyCooked-MeatPattyBurned.asset，设置对应变量

![](<images/1689211719699.png>)

  
接下来我们要用状态机来实现食物在一开始被放到炉灶台上时间到了会被煎熟，等再过了一段时间后会被煎糊的效果，使用一个枚举类型的变量 State 保存各个状态，在 Start() 中初始化状态为 Idle，在 Update() 中用一个 switch 语句写下各个状态的行为，在 Interact() 中写下当角色把食材放到炉灶台上重置计时器与设置状态为 Frying、当角色拿取物品设置状态为 Idle 的逻辑

```
// StoveCounter.cs中
...
public class StoveCounter : BaseCounter
{
    public enum State
    {
        Idle,
        Frying,
        Fried,
        Burned
    }
    
    [SerializeField] private FryingRecipeSO[] fryingRecipeSOArray;
    [SerializeField] private BurningRecipeSO[] burningRecipeSOArray;
    
    private State state;
    private float fryingTimer;
    private FryingRecipeSO fryingRecipeSO;
    private float burningTimer;
    private BurningRecipeSO burningRecipeSO;

    private void Start()
    {
        state = State.Idle;
    }
    
    private void Update()
    {
        if (HasKitchenObject())
        {
            switch (state)
            {
                case State.Idle:
                    break;
                case State.Frying:
                    fryingTimer += Time.deltaTime;
                    if (fryingTimer > fryingRecipeSO.fryingTimerMax)
                    {
                        // 煎好了
                        fryingTimer = 0f;
                        GetKitchenObject().DetroySelf();
                        KitchenObject.SpawnKitchenObject(fryingRecipeSO.output, this);
                    
                        Debug.Log("Object fried!");
                        state = State.Fried;
                        burningTimer = 0f;
                        burningRecipeSO = GetBurningRecipeSOWithInput(GetKitchenObject().GetKitchenObjectSO());
                    }
                    break;
                case State.Fried:
                    burningTimer += Time.deltaTime;
                    if (burningTimer > burningRecipeSO.burningTimerMax)
                    {
                        // 煎好了
                        fryingTimer = 0f;
                        GetKitchenObject().DetroySelf();
                        KitchenObject.SpawnKitchenObject(burningRecipeSO.output, this);
                    
                        Debug.Log("Object burned!");
                        state = State.Burned;
                    }
                    break;
                case State.Burned:
                    break;
            }
        Debug.Log(state);
        }
    }

    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                if (HasRecipeWithInput(player.GetKitchenObject().GetKitchenObjectSO()))
                {
                    // 角色拿着的物品可以被煎
                    ...
                    state = State.Frying;
                    fryingTimer = 0f;
                }
            } else
            {
                // 角色没有物品
            }
        } else
        {
            // 柜子上有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品
            } else
            {
                // 角色没有物品，拾取物品
                ... 
                state = State.Idle;
            }
        }
    }
    ...
    private BurningRecipeSO GetBurningRecipeSOWithInput(KitchenObjectSO inputKitchenObjectSO)
    {
        foreach (BurningRecipeSO burningRecipeSO in burningRecipeSOArray)
        {
            if (burningRecipeSO.input == inputKitchenObjectSO)
            {
                return burningRecipeSO;
            }
        }
        return null;
    }
}
```

挂载好相应的组件并设置变量

![](<images/1689211721342.png>)

  
运行游戏，就能看到将肉饼放到炉灶台上由生变熟变糊的过程

![](<images/1689211721381.png>)

### 12.3 简单的特效

在所给的 StoveCounter_Visual 素材中有两个简单的效果，一个是 SizzlingParticles 像火花的粒子效果，一个是 StoveOnVisual 红色的自发光面片，这两个加上之后就有一种烹饪的感觉，我们希望在 Frying 和 Fried 状态出现这个效果

![](<images/1689211721457.png>)

  
粒子效果主要调整了 Gravity Source、Size over Lifetime 的曲线让粒子发射出来逐渐受力落下变小

![](<images/1689211721585.png>)

  
在 Scripts/Counters 文件夹新建 StoveCounterVisual.cs，在 StoveCounter.cs 中添加 EventHandler 委托，在每一处状态改变的代码后触发该委托并传递当前状态作为参数，在 StoveCounterVisual.cs 中写在 Frying 和 Fried 状态才会出现上面两个视觉效果的逻辑

```
// StoveCounter.cs中
...
state = State.Fried;
OnStateChanged?.Invoke(this, new OnStateChangedEventArgs
{
	state = state
});
...
state = State.Burned;
OnStateChanged?.Invoke(this, new OnStateChangedEventArgs
{
	state = state
});
...
state = State.Frying;
OnStateChanged?.Invoke(this, new OnStateChangedEventArgs
{
	state = state
});
...
state = State.Idle;
OnStateChanged?.Invoke(this, new OnStateChangedEventArgs
{
	state = state
});
...
```

```
// StoveCounterVisual.cs中
using UnityEngine;

public class StoveCounterVisual : MonoBehaviour
{
    [SerializeField] private StoveCounter stoveCounter;
    [SerializeField] private GameObject stoveGameObject;
    [SerializeField] private GameObject particlesGameObject;
    
    private void Start()
    {
        stoveCounter.OnStateChanged += StoveCounter_OnStateChanged;
    }
    
    private void StoveCounter_OnStateChanged(object sender, StoveCounter.OnStateChangedEventArgs e)
    {
        bool showVisual = e.state == StoveCounter.State.Frying || e.state == StoveCounter.State.Fried;
        stoveGameObject.SetActive(showVisual);
        particlesGameObject.SetActive(showVisual);
    }
}
```

将 StoveCounterVisual.cs 挂载到 StoveCounter_Visual 上，设置好变量，运行游戏，即可看到煎肉的特效

![](<images/1689211722940.png>)

### 12.4 使用接口重构进度条组件用在炉灶台上

我们在切菜台上使用的进度条组件 ProgressBarUI.cs 只接受 CuttingCounter 类的实例，但是我们想要实现一个更通用的进度条组件，之后其他需要表示进度的物品都可以用上，所以我们需要修改一下 ProgressBarUI.cs，使它接受一个实现了 IHasProgress 接口的类的实例  
将 CuttingCounter.prefab 下的 ProgressBarUI 对象拖动到到 Prefabs 文件夹，在 Scripts 文件夹新建 IHasProgress.cs，在这个脚本中我们定义一个接口，这个接口拥有一个可以传递已经归一化了的进度的 EventHandler 委托

```
// IHasProgress.cs中
using System;

public interface IHasProgress
{
    public event EventHandler<OnProgressChangedEventArgs> OnProgressChanged;
    public class OnProgressChangedEventArgs : EventArgs
    {
        public float progressNormalized;
    }
}
```

在 ProgressBarUI.cs 中，我们将原来使用 CuttingCounter 类的实例的方法与其定义的委托的地方替换为实现了 IHasProgress 接口的类的实例的方法与其定义的委托，但是 Unity 并不支持我们在编辑器面板显示 Interface，所以我们先声明了一个 GameObject 对象，在 Start() 中再确定实现了该接口的对象

```
// ProgressBarUI.cs中
...
public class ProgressBarUI : MonoBehaviour
{
    // [SerializeField] private CuttingCounter cuttingCounter;
    [SerializeField] private GameObject hasProgressGameObject;
    ...
    private IHasProgress hasProgress;

    private void Start()
    {
        hasProgress = hasProgressGameObject.GetComponent<IHasProgress>();
        if (hasProgress == null)
        {
            Debug.LogError("Game Object " + hasProgressGameObject + " does not have a component that implements IHasProgress!");
        }
        
        // cuttingCounter.OnProgressChanged += CuttingCounter_OnProgressChanged;
        hasProgress.OnProgressChanged += HasProgress_OnProgressChanged;
        ...
    }
    
    // private void CuttingCounter_OnProgressChanged(object sender, CuttingCounter.OnProgressChangedEventArgs e)
    private void HasProgress_OnProgressChanged(object sender, IHasProgress.OnProgressChangedEventArgs e)
    {
        ...
    }
    ...
}
```

我们先来修复 CuttingCounter.cs，我们需要让该类继承自 IHasProgress 接口，然后改动所有使用 OnProgressChanged 这个委托的地方，注释处是改动前

```
// CuttingCounter.cs中
...
public class CuttingCounter : BaseCounter, IHasProgress
{
// public event EventHandler<OnProgressChangedEventArgs> OnProgressChanged;
// public class OnProgressChangedEventArgs : EventArgs
// {
//     public float progressNormalized;
// }
	public event EventHandler<IHasProgress.OnProgressChangedEventArgs> OnProgressChanged;
	...
	// OnProgressChanged?.Invoke(this, new OnProgressChangedEventArgs
	OnProgressChanged?.Invoke(this, new IHasProgress.OnProgressChangedEventArgs
	{
		...
	});
	...
	// OnProgressChanged?.Invoke(this, new OnProgressChangedEventArgs
	OnProgressChanged?.Invoke(this, new IHasProgress.OnProgressChangedEventArgs
	{
		 ...
	});
}
...
```

运行游戏，可以正常切菜

![](<images/1689211723087.png>)

  
然后来处理在 StoveCounter 上的进度条显示，首先进到 StoveCounter，将 Prefab 文件夹下的 ProgressBarUI.prefab 拖动到 StoveCounter 下，设置好组件中相应变量，在 StoveCounter.cs 中，在 Interact() 中角色将物品放至炉灶台上时、Frying 状态中、Fried 状态中触发 OnProgressChanged 委托设置归一化进度值，在计时器跑完 Fried 变为 Burned 状态时和角色中途拿起物品时同样触发 OnProgressChanged 委托，将归一化进度值设置为 0

```
// StoveCounter.cs中
...
public class StoveCounter : BaseCounter, IHasProgress
{
    public event EventHandler<IHasProgress.OnProgressChangedEventArgs> OnProgressChanged;
    ...
    private void Update()
    {
        switch (state)
        {
            if (HasKitchenObject())
            {
                case State.Idle:
                    break;
                case State.Frying:
                    ...
                    OnProgressChanged?.Invoke(this, new IHasProgress.OnProgressChangedEventArgs
                    {
                        progressNormalized = fryingTimer / fryingRecipeSO.fryingTimerMax
                    });
                
                    if (fryingTimer > fryingRecipeSO.fryingTimerMax)
                    {
                        // 煎好了
                        ...
                    }
                    break;
                case State.Fried:
                    ...
                    OnProgressChanged?.Invoke(this, new IHasProgress.OnProgressChangedEventArgs
                    {
                        progressNormalized = burningTimer / burningRecipeSO.burningTimerMax
                    });
                    
                    if (burningTimer > burningRecipeSO.burningTimerMax)
                    {
                        // 煎好了
                        ...
                        OnProgressChanged?.Invoke(this, new IHasProgress.OnProgressChangedEventArgs
                        {
                            progressNormalized = 0f
                        });
                    }
                    break;
                case State.Burned:
                    break;
            }
        }
    }

    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                if (HasRecipeWithInput(player.GetKitchenObject().GetKitchenObjectSO()))
                {
                    // 角色拿着的物品可以被煎
                    ...
                    OnProgressChanged?.Invoke(this, new IHasProgress.OnProgressChangedEventArgs
                    {
                        progressNormalized = fryingTimer / fryingRecipeSO.fryingTimerMax
                    });
                }
            } else
            {
                // 角色没有物品
            }
        } else
        {
            // 柜子上有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品
            } else
            {
                // 角色没有物品，拾取物品
                ...
                OnProgressChanged?.Invoke(this, new IHasProgress.OnProgressChangedEventArgs {
                    progressNormalized = 0f
                });
            }
        }
    }
}
```

运行游戏，可以看到炉灶台的进度条正常运行

![](<images/1689211723309.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#stoveCounterStateMachine)

## 13 创建盘子存放台 Plates Counter

### 13.1 继承自柜台基类创建 PlatesCounter 与自动生成盘子

与上面创建其他 Counter 相同，从_BaseCounter.prefab 右键 ->Create->PrefabVariant、复制一个 Selected，改缩放，在 Scripts 文件夹下新建 PlatesCounter.cs 继承自 BaseCounter，挂载好对应脚本  
在 ScriptableObjects/KitchenObjectSO 下右键 ->Create->KitchenObjectSO 新建 Plate.asset，在 Prefabs/KitchenObjects 文件夹下复制一个 prefab 改为 Plate.prefab 并放进去对应模型，将各个中的组件的相应变量填好  
我们希望盘子存放台可以每隔几秒生成一个盘子，当存放台上的盘子到达一定数量时不再生成盘子，然而，我们的_BaseCounter 在设计之初并没有考虑过一个 Counter 上放置多个 KitchenObject 实例的情况，所以，我们并不会真的去生成 KitchenObject 实例，而是生成盘子的模型 KitchenObject_Visual，当角色想要取盘子时我们再调用 SpawnKitchenObject() 方法去生成 KitchenObject 实例交给角色  
在 PlatesCounter.cs 中，规定生成盘子所需的时间、最大盘子数，在计时器到时时触发 EventHandler 委托，在 PlatesCounterVisual.cs 中我们用一个 List 保存生成的 KitchenObject_Visual，并实现堆在柜子上的效果

```
// PlatesCounter.cs
using UnityEngine;
using System;

public class PlatesCounter : BaseCounter
{
    public event EventHandler OnPlateSpawned;

    private float spawnPlateTimer;
    private float spawnPlateTimerMax = 4f;
    private int platesSpawnedAmount;
    private int platesSpawnedAmountMax = 4;

    private void Update()
    {
        spawnPlateTimer += Time.deltaTime;
        if (spawnPlateTimer > spawnPlateTimerMax)
        {
            spawnPlateTimer = 0f;
            if (platesSpawnedAmount < platesSpawnedAmountMax)
            {
                platesSpawnedAmount++;
                
                OnPlateSpawned?.Invoke(this, EventArgs.Empty);
            }
        }
    }
}
```

```
// PlatesCounterVisual.cs
using System;
using System.Collections.Generic;
using UnityEngine;

public class PlatesCounterVisual : MonoBehaviour
{
    [SerializeField] private PlatesCounter platesCounter;
    [SerializeField] private Transform counterTopPoint;
    [SerializeField] private Transform plateVisualPrefab;

    private List<GameObject> plateVisualGameObjectsList;

    private void Awake()
    {
        plateVisualGameObjectsList = new List<GameObject>();
    }
    
    private void Start()
    {
        platesCounter.OnPlateSpawned += PlatesCounter_OnPlateSpawned;
    }
    
    private void PlatesCounter_OnPlateSpawned(object sender, EventArgs e)
    {
        Transform plateVisualTransform = Instantiate(plateVisualPrefab, counterTopPoint);
        
        float plateOffsetY = 0.1f;
        plateVisualTransform.localPosition = new Vector3(0f, plateOffsetY * plateVisualGameObjectsList.Count, 0f);
        
        plateVisualGameObjectsList.Add(plateVisualTransform.gameObject);
    }
}
```

运行游戏，我们就能看到每隔我们设置的 4 秒，就会出现一个盘子，且最多能达到 4 个盘子

![](<images/1689211723545.png>)

  
>接下来让角色可以拿起来盘子，我们需要在 PlatesCounter.cs 中实现一下 Interact() 接口，当角色拿起盘子符合条件时将盘子 Spawn 给角色，同时触发 EventHandler 委托，该委托调用的函数写在 PlateCounterVisual.cs 中，该函数移除掉存放 Plater_Visual 的 List 的最后一个对象

```
// PlatesCounter.cs
...
public class PlatesCounter : BaseCounter
{
    ...
    public event EventHandler OnPlateRemoved;
    ...
    [SerializeField] private KitchenObjectSO plateKitchenObjectSO;
    ...
    public override void Interact(Player player)
    {
        if (!player.HasKitchenObject())
        {
            // 角色手上没有东西
            if (platesSpawnedAmount > 0)
            {
                // 盘子存放台上有盘子
                platesSpawnedAmount--;
                
                KitchenObject.SpawnKitchenObject(plateKitchenObjectSO, player);
                
                OnPlateRemoved?.Invoke(this, EventArgs.Empty);
            }
        }
    }
}
```

```
// PlateCounterVisual.cs中
...
public class PlatesCounterVisual : MonoBehaviour
{
    ...
    private void Start()
    {
        ...
        platesCounter.OnPlateRemoved += PlatesCounter_OnPlateRemoved;
    }
    ...
    private void PlatesCounter_OnPlateRemoved(object sender, EventArgs e)
    {
        if (plateVisualGameObjectsList.Count > 0)
        {
            GameObject plateGameObject = plateVisualGameObjectsList[plateVisualGameObjectsList.Count - 1];
            plateVisualGameObjectsList.Remove(plateGameObject);
            Destroy(plateGameObject);
        }
    }
}
```

运行游戏，我们就可以拿起盘子了

![](<images/1689211724087.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#platesCounter)

### 13.2 使用盘子装物品

首先我们先在 ClearCounter 上实现装物品的效果，在之前 ClearCounter.cs 的 Interact() 中，当柜子上有物品而角色也有物品时，我们没有写任何逻辑，这里我们可以在角色拿的物品是盘子的时候将桌子上的物品装到盘子里，为了保存盘子里装的东西已经方便实现对应方法，我们在 Script 文件夹新建 PlateKitchenObject.cs，这个类继承自 KitchenObject，使用一个 List 保存装的东西

```
// PlateKitchenObject.cs中
using System.Collections.Generic;

public class PlateKitchenObject : KitchenObject
{
    private List<KitchenObjectSO> kitchenObjectSOList;

    private void Awake() {
        kitchenObjectSOList = new List<KitchenObjectSO>();
    }

    public void AddIngredient(KitchenObjectSO kitchenObjectSO) {
        kitchenObjectSOList.Add(kitchenObjectSO);
    }
}
```

在 Plate.prefab 上添加 PlateKitchenObject 组件，移除 KitchenObject 组件

![](<images/1689211724446.png>)

  
在 ClearCounter.cs 中写上在角色拿的物品是盘子的时候将桌子上的物品添加到 List 中的逻辑

```
// ClearCounter.cs中
...
public class ClearCounter : BaseCounter
{
    ...
    public override void Interact(Player player)
    {
        if (!HasKitchenObject())
        {
            // 柜子上没有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品，放置物品
                ...
            } else
            {
                // 角色没有物品
            }
        } else
        {
            // 柜子上有物品
            if (player.HasKitchenObject())
            {
                // 角色有物品
                if (player.GetKitchenObject() is PlateKitchenObject)
                {
                    // 角色拿的是盘子
                    PlateKitchenObject plateKitchenObject = player.GetKitchenObject() as PlateKitchenObject;
                    plateKitchenObject.AddIngredient(GetKitchenObject().GetKitchenObjectSO());
                    GetKitchenObject().DetroySelf();
                }
            } else
            {
                // 角色没有物品，拾取物品
                ...
            }
        }
    }
}
```

运行游戏，将物品放到空的柜台上，在 Inspector 面板右上角三个点切换为 Debug 模式，就能在角色进行交互时将物品添加到我们想要加入的 List 中了

![](<images/1689211724485.png>)

  
在最终游戏中，我们的盘子中并不能装一切东西，并且盘子中也不会装两种相同种类的物品，我们通过一个 List 存放可以被装在盘子的物品，每次装盘时检查该物品是否在 List 中，并且判断是否盘子中已经有了相同种类的物品

```
// PlateKitchenObject.cs中
...
public class PlateKitchenObject : KitchenObject
{
    [SerializeField] private List<KitchenObjectSO> validKitchenObjectSOList;
    
    private List<KitchenObjectSO> kitchenObjectSOList;
    ...
    public bool TryAddIngredient(KitchenObjectSO kitchenObjectSO)
    {
        if (!validKitchenObjectSOList.Contains(kitchenObjectSO))
        {
            // 盘子中不能放这种物品
            return false;
        }
        if (kitchenObjectSOList.Contains(kitchenObjectSO))
        {
            // 盘子中已经有了这种物品
            return false;
        } else
        {
            kitchenObjectSOList.Add(kitchenObjectSO);
            return true;
        }
    }
}
```

在 Plate.prefab 上设置可以被装盘的物品列表

![](<images/1689211724526.png>)

  
为了以后检查一个 KitchenObject 是否为盘子更加方便，我们在 KitchenObject.cs 中写一个 TryGetPlate() 判断是否为盘子，如果是则传回 PlateKitchenObject 类的版本

```
// KitchenObject.cs中
...
public class KitchenObject : MonoBehaviour
{
    ...
    public bool TryGetPlate(out PlateKitchenObject plateKitchenObject)
    {
        if (this is PlateKitchenObject)
        {
            plateKitchenObject = this as PlateKitchenObject;
            return true;
        } else
        {
            plateKitchenObject = null;
            return false;
        }
    }
    ...
}
```

在 ClearCounter.cs 中替换相关的实现，在 CuttingCounter.cs、StoveCounter.cs 中相同位置添加上用盘子装物品的相关代码，由于 StoveCounter.cs 中有状态设置的代码，所以在写了用盘子装上物品的逻辑后还要设置相关状态

```
// ClearCounter.cs与CuttingCounter.cs中的相同位置
...
// 柜子上有物品
if (player.HasKitchenObject())
{
	 // 角色有物品
	 // if (player.GetKitchenObject() is PlateKitchenObject)
	 if (player.GetKitchenObject().TryGetPlate(out PlateKitchenObject plateKitchenObject))
	 {
		  // 角色拿的是盘子
		  // PlateKitchenObject plateKitchenObject = player.GetKitchenObject() as PlateKitchenObject;
		  if (plateKitchenObject.TryAddIngredient(GetKitchenObject().GetKitchenObjectSO()))
		  {
				GetKitchenObject().DetroySelf();
		  }
	 }
}
...
```

```
// StoveCounter.cs中
...
// 柜子上有物品
if (player.HasKitchenObject())
{
	 // 角色有物品
	 if (player.GetKitchenObject().TryGetPlate(out PlateKitchenObject plateKitchenObject))
	 {
		  // 角色拿的是盘子
		  if (plateKitchenObject.TryAddIngredient(GetKitchenObject().GetKitchenObjectSO()))
		  {
				GetKitchenObject().DetroySelf();
				
				state = State.Idle;
	 
				OnStateChanged?.Invoke(this, new OnStateChangedEventArgs
				{
					 state = state
				});
	 
				OnProgressChanged?.Invoke(this, new IHasProgress.OnProgressChangedEventArgs {
					 progressNormalized = 0f
				});
		  }
	 }
}
...
```

启动游戏，在 Inspector 面板打开 Debug 模式，就能看到被装到盘子的物品的 List 了

![](<images/1689211724765.png>)

接下来我们要实现相反的效果，我们希望角色拿着相应的食材来装到盘子了，由于只有在空的柜子上才能放盘子，我们只用在 ClearCounter.cs 中实现相关逻辑

```
// ClearCounter.cs中
...
// 柜子上有物品
if (player.HasKitchenObject())
{
	 // 角色有物品
	 if (player.GetKitchenObject().TryGetPlate(out PlateKitchenObject plateKitchenObject))
	 {
		  ...
	 } else {
		  // 角色拿的不是盘子，而是别的东西
		  if (GetKitchenObject().TryGetPlate(out plateKitchenObject))
		  {
				if (plateKitchenObject.TryAddIngredient(player.GetKitchenObject().GetKitchenObjectSO()) )
				{
					 player.GetKitchenObject().DetroySelf();
				}
		  }
	 }
}
...
```

运行游戏，可以将物品向空柜台上的盘子里装东西

![](<images/1689211724829.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#platePickUpObjects)

### 13.3 装盘效果

我们要实现的装盘效果用了比较简单的方式，可以看到在_Assets/PrefabsVisuals 下有一个 PlateCompleteVisual.prefab 的汉堡模型，由六个不同的部分组成，要做成不同搭配的菜只需要让对应物品显示，其余物品隐藏即可

![](<images/1689211725043.png>)

  
在上一个小节我们已经可以将放进盘子的物品保存到一个 List 了，只要根据 List 中有哪些物品去显示和隐藏物品即可，我们可以通过 tranform.Find() 通过名字来找到对应的物品，但是直接使用 string 并不是一个很好的方式，因此我们需要建立 KitchenObjectSO 类的实例与 GameObject 类的实例之间的对应关系，要找到这个对应关系，我们可以定义一个包含这两个变量的 struct，用一个 List 保存所有的 struct 的实例，也就是所有的对应关系  
在 Scripts 文件夹新建 PlateCompleteVisual.cs，添加到 PlateCompleteVisual.prefab 上，在 PlateKitchenObject.cs 中添加一个 EventHandler 委托，在将物品放置到盘子中时触发该委托并传递，委托调用的函数写在 PlateCompleteVisual.cs 中，遍历盘子中物体的 List，显示和隐藏对应物品

```
// PlateKitchenObject.cs中
...
public class PlateKitchenObject : KitchenObject
{
    public event EventHandler<OnIngredientAddedEventArgs> OnIngredientAdded;
    public class OnIngredientAddedEventArgs : EventArgs
    {
        public KitchenObjectSO kitchenObjectSO;
    }
    ...
    public bool TryAddIngredient(KitchenObjectSO kitchenObjectSO)
    {
        if (!validKitchenObjectSOList.Contains(kitchenObjectSO))
        {
            // 盘子中不能放这种物品
            return false;
        }
        if (kitchenObjectSOList.Contains(kitchenObjectSO))
        {
            // 盘子中已经有了这种物品
            return false;
        } else
        {
            kitchenObjectSOList.Add(kitchenObjectSO);
            
            OnIngredientAdded?.Invoke(this, new OnIngredientAddedEventArgs
            {
                kitchenObjectSO = kitchenObjectSO
            });
            
            return true;
        }
    }
}
```

在 unity 序列化 class 或一个 struct 的实例需要在这个 struct 或类前加上 [Serializable]

```
// PlateCompleteVisual.cs中
using System;
using System.Collections.Generic;
using UnityEngine;

public class PlateCompleteVisual : MonoBehaviour
{
    [Serializable]
    public struct KitchenObjectSO_GameObject
    {
        public KitchenObjectSO kitchenObjectSO;
        public GameObject gameObject;    
    }

    [SerializeField] private PlateKitchenObject plateKitchenObject;
    [SerializeField] private List<KitchenObjectSO_GameObject> kitchenObjectSOGameObjectList;

    private void Start()
    {
        plateKitchenObject.OnIngredientAdded += PlateKitchenObject_OnIngredientAdded;
        
        foreach (KitchenObjectSO_GameObject kitchenObjectSOGameObject in kitchenObjectSOGameObjectList)
        {
            kitchenObjectSOGameObject.gameObject.SetActive(false);
        }
    }
    
    private void PlateKitchenObject_OnIngredientAdded(object sender, PlateKitchenObject.OnIngredientAddedEventArgs e)
    {
        foreach (KitchenObjectSO_GameObject kitchenObjectSOGameObject in kitchenObjectSOGameObjectList)
        {
            if (kitchenObjectSOGameObject.kitchenObjectSO == e.kitchenObjectSO)
            {
                kitchenObjectSOGameObject.gameObject.SetActive(true);
            }
        }
    }
}
```

将 PlateCompleteVisual.prefab 添加到 Plate.prefab 下，设置好相应的变量

![](<images/1689211725275.png>)

  
现在运行游戏，就能够正确装盘了

![](<images/1689211725322.png>)

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#plateCompleteVisual)

### 13.4 显示盘子中物品的 UI

接下来我们做一个显示一个盘子中都放了哪些物品的 UI  
首先在 Package Manager 中下载 2D Sprite

![](<images/1689211725447.png>)

  
在 Plate.prefab 中右键 ->UI->Canvas 新建一个 Canvas 重命名为 PlateIconsUI，调整位置大小添加一个 GridLayoutGroup 组件；在 PlateIconsUI 下新建空物体重命名为 Icon Template，在这个 Icon Template 下右键 ->UI->Image，重命名为 Background，Source Image 设置为下载的包里的 Circle 素材（记得点开 Source Image 右上角显示隐藏项）；复制一份重命名为 Icon，Source Image 放一个 Bread；我们可以多复制几个 Icon Template 看看效果

![](<images/1689211725665.png>)

  
我们可以给不同物品的 UI 都做成 prefab，但是作者认为这样做会有很多 prefab，文件会很乱，所以我们接下来写的脚本就挂载到 PlateIconsUI 对象上然后复制和删除下面的 IconTemplate 就行了  
我们要给 PlateIconsUI 添加一个新的脚本 PlateIconsUI.cs，在这个脚本中写生成 IconTemplate 的方法并在物品放上去时调用，我们希望不同的物品放到盘子时会有不同的 Icon，为此我们可以用比较 "脏" 的方式，生成 IconTemplate 后 Find("Icon") 找到 Icon 对象然后改变 Image 组件的 Sprite 属性，但是代码里最好不要用 string 来找一个对象，为此我们又在 IconTemplate 上添加了一个新的脚本 PlateIconsSingleUI.cs 用来获取 Icon 对象更改 Sprite 属性

```
// PlateIconsSingleUI.cs中
using UnityEngine;
using UnityEngine.UI;

public class PlateIconsSingleUI : MonoBehaviour
{
    [SerializeField] private Image image;
    
    public void SetKitchenObjectSO(KitchenObjectSO kitchenObjectSO)
    {
        image.sprite = kitchenObjectSO.sprite;
    }
}
```

在上面的教程中，我们已经在 PlateKitchenObjectSO.cs 中添加过了一个叫做 OnIngredientAdded 的 EventHandler 委托，该委托在物品放入盘子时触发，并且当前放入盘中的物品的 ScriptableObject 存在了一个 List 中，我们需要在 PlateKitchenObjectSO.cs 添加一个方法获取这个 List，然后在 PlateIconsUI.cs 中为这个委托添加调用的函数和一些刷新 UI 的逻辑即可

```
// PlateKitchenObjectSO.cs中
...
public class PlateKitchenObject : KitchenObject
{
    ...
    public List<KitchenObjectSO> GetKitchenObjectSOList()
    {
        return kitchenObjectSOList;
    }
}
```

```
// PlateIconsUI.cs中
using UnityEngine;

public class PlateIconsUI : MonoBehaviour
{
    [SerializeField] private PlateKitchenObject plateKitchenObject;
    [SerializeField] private Transform iconTemplate;

    private void Awake()
    {
        iconTemplate.gameObject.SetActive(false);
    }

    private void Start()
    {
        plateKitchenObject.OnIngredientAdded += PlateKitchenObject_OnIngredientAdded;
    }

    private void PlateKitchenObject_OnIngredientAdded(object sender, PlateKitchenObject.OnIngredientAddedEventArgs e)
    {
        UpdateVisual();
    }
    
    private void UpdateVisual()
    {
        foreach (Transform child in transform)
        {
            if (child == iconTemplate) continue;
            Destroy(child.gameObject);
        }
        
        foreach (KitchenObjectSO kitchenObjectSO in plateKitchenObject.GetKitchenObjectSOList())
        {
            Transform iconTransform = Instantiate(iconTemplate, transform);
            iconTransform.gameObject.SetActive(true);
            iconTransform.GetComponent<PlateIconsSingleUI>().SetKitchenObjectSO(kitchenObjectSO);
        }
    }
}
```

这个 UI 也用到了 Canvas，所以同样也有根据视角不同会看到相反方向的 UI 的情况，不过还好我们在做进度条 UI 时已经写了一个 LookAtCamera.cs 组件解决这个问题，只需要把这个组件放到 PlateIconsUI 上即可

![](<images/1689211725835.png>)

运行游戏，即可看到盘子的 UI 正常运行

![](<images/1689211726192.png>)

目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#plateWorldUIIcons)

下篇：

[undefined](https://zhuanlan.zhihu.com/p/612951943)