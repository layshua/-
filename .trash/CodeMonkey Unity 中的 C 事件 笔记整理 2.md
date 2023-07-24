该笔记整理自 Youtube 上的 CodeMonkey 频道 C# 教程系列

原教程地址：[https://youtu.be/OuZrhykVytg](https://youtu.be/OuZrhykVytg)

### 1 介绍
首先我们需要将我们的事件设为 public，然后使用 event 关键字，接下来我们需要定义类型，.net 中有一个多播委托类型 [EventHandler](https://learn.microsoft.com/en-us/dotnet/api/system.eventargs?view=net-7.0)，要使用这个类型，我们需要加上 using System 引入 System 命名空间，这个类型有两个字段，一个是事件源 sender，一个是 EventArgs

![[9b2db9a0ded1bf88fd14248d5bdf2bb7_MD5.png]]

### 3 EventHandler 参数：object sender

现在我们创建一个名为 OnSpacePressed 的 EventHandler 对象，使用事件应该尽量使用 "On" 开头的命名，如 OnEnemyKilled、OnPlayWin 等等，创建完成之后我们在 Update() 中触发这个事件，事件源 sender 我们传入 this，EventArgs 我们这里不需要传递任何额外信息，所以使用 EventArgs.Empty

```cs
using System;
using UnityEngine;

public class TestingEvents : MonoBehaviour
{
    public event EventHandler OnSpacePressed;
    private void Start()
    {

    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            OnSpacePressed(this, EventArgs.Empty);
        }
    }
}
```


```cs
using System;
using UnityEngine;

public class TestingEvents : MonoBehaviour
{
    public event EventHandler OnSpacePressed;
    private void Start()
    {

    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
           
            OnSpacePressed?.Invoke(this, EventArgs.Empty);
        }
    }
}
```

接下来我们写一个函数用来订阅事件，这个函数的形参需要与我们的委托相同，在 Start() 中使用 `+=` 订阅事件

```cs
using System;
using UnityEngine;

public class TestingEvents : MonoBehaviour
{
    public event EventHandler OnSpacePressed;
    private void Start()
    {
        OnSpacePressed += Testing_OnSpacePressed;
    }

    private void Testing_OnSpacePressed(object sender, EventArgs e)
    {
        Debug.Log("Space Pressed");
    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            //若没有订阅，OnSpacePressed 的值是 null
            //EventArgs 我们这里不需要传递任何额外信息，所以使用 EventArgs.Empty
            OnSpacePressed?.Invoke(this, EventArgs.Empty);
        }
    }
}
```

运行游戏，按下空格，我们就可以看到该事件触发的函数

![[d5e96c1660250a39150d8484ed75fde2_MD5.gif]]

现在我们都是在同一个脚本、同一个类中去触发和监听事件，但使用事件模型的好处是我们可以从其他地方去监听，所以接下来我们新创建一个脚本 TestingEventSubscriber.cs，将上面的监听事件的过程放到这个脚本中

```cs
// TestingEventSubscriber.cs中
using System;
using UnityEngine;

public class TestingEventSubscriber : MonoBehaviour
{
    private void Start()
    {
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed += TestingEvents_OnSpacePressed;
    }

    private void TestingEvents_OnSpacePressed(object sender, EventArgs e)
    {
        Debug.Log("Space Pressed");
    }
}
// TestingEvents.cs中
using System;
using UnityEngine;

public class TestingEvents : MonoBehaviour
{
    public event EventHandler OnSpacePressed;
    private void Start()
    {

    }
    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            OnSpacePressed?.Invoke(this, EventArgs.Empty);
        }
    }
}
```

将脚本挂载到同一个物体上，运行游戏，按下空格，和之前的效果相同

![[d5e96c1660250a39150d8484ed75fde2_MD5.gif]]

除了可以使用 += 订阅事件，我们也可以使用 -= 取消订阅

```
// TestingEventSubscriber.cs中
using System;
using UnityEngine;

public class TestingEventSubscriber : MonoBehaviour
{
    private void Start()
    {
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed += TestingEvents_OnSpacePressed;
    }

    private void TestingEvents_OnSpacePressed(object sender, EventArgs e)
    {
        Debug.Log("Space Pressed");
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed -= TestingEvents_OnSpacePressed;
    }
}
```

再次运行游戏，我们只能够触发一次事件

![[2fb9124ba6da68bf3bfc86d07bdb667b_MD5.gif]]

### 4 EventHandler 参数：EventArgs e

EventHandler 的另一个参数 EventArgs e 可以通过事件传递更多信息，**要使用 EventArgs，我们首先需要使用泛型，然后定义一个派生自 EventArgs 的类**，比如这里我们想要传递一个 int 类型的 spaceCount 记录按下空格的次数，在调整了 EventArgs 之后两个脚本如下

```cs
// TestingEventSubscriber.cs中
using System;
using UnityEngine;

public class TestingEventSubscriber : MonoBehaviour
{
    private void Start()
    {
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed += TestingEvents_OnSpacePressed;
    }

    private void TestingEvents_OnSpacePressed(object sender, TestingEvents.OnSpacePressEventArgs e)
    {
        Debug.Log("Space Pressed" + e.spaceCount);
    }
}
// TestingEvents.cs中
using System;
using UnityEngine;

public class TestingEvents : MonoBehaviour
{
    public event EventHandler<OnSpacePressEventArgs> OnSpacePressed;
    public class OnSpacePressEventArgs : EventArgs
    {
        public int spaceCount;
    }
    private int _spaceCount;

    private void Start()
    {

    }
    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            spaceCount++;
            OnSpacePressed?.Invoke(this, new OnSpacePressEventArgs { spaceCount = _spaceCount });
        }
    }
}
```

运行游戏，可以看到控制台显示出了按下空格的次数

![[fc01cca4aa4deea225a506d26679262f_MD5.gif]]

### 5 自定义委托

EventHandler 只是. NET 的标准，我们也可以定义我们自己的委托类型，比如这里我们定义一个有一个浮点数参数的委托

```
// TestingEvents.cs中
...

public class TestingEvents : MonoBehaviour
{
    ...

    public event TestEventDelegate OnFloatEvent;
    public delegate void TestEventDelegate(float f);

   ...

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            ...

            OnFloatEvent?.Invoke(5.5f);
        }
    }
}
// TestingEventSubscriber.cs中
...

public class TestingEventSubscriber : MonoBehaviour
{
    private void Start()
    {
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed += TestingEvents_OnSpacePressed;
        testingEvents.OnFloatEvent += TestingEvents_OnFloatEvent;
    }

    private void TestingEvents_OnSpacePressed(object sender, TestingEvents.OnSpacePressEventArgs e)
    {
        Debug.Log("Space Pressed " + e.spaceCount);
    }

    private void TestingEvents_OnFloatEvent(float f)
    {
        Debug.Log("Float Event " + f);
    }
}
```

运行游戏，按下空格，就可以看到我们新定义的事件

![[0376a5c7cd6b7b1dc7e791c12abcfe72_MD5.gif]]

既然可以使用自定义的委托，那也可以使用 c# 自带的 Action 委托

```cs
// TestingEvents.cs中
...

public class TestingEvents : MonoBehaviour
{
    ...

    public event Action<bool, int> OnActionEvent;

   ...

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            ...

            OnActionEvent?.Invoke(true, 56);
        }
    }
}
// TestingEventSubscriber.cs中
...

public class TestingEventSubscriber : MonoBehaviour
{
    private void Start()
    {
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed += TestingEvents_OnSpacePressed;
        testingEvents.OnFloatEvent += TestingEvents_OnFloatEvent;
        testingEvents.OnActionEvent += TestingEvents_OnActionEvent;
    }

    private void TestingEvents_OnSpacePressed(object sender, TestingEvents.OnSpacePressEventArgs e)
    {
        Debug.Log("Space Pressed " + e.spaceCount);
    }

    private void TestingEvents_OnFloatEvent(float f)
    {
        Debug.Log("Float Event " + f);
    }

    private void TestingEvents_OnActionEvent(bool b, int i)  
    {  
        Debug.Log("Action Event " + b + " " + i);  
    }
}
```

### 6 UnityEvent

Unity 中也有一个 UnityEvent，我们也可以使用 UnityEvent，使用 UnityEvent 需要 using UnityEngine.Events

```
// TestingEvents.cs中
...

public class TestingEvents : MonoBehaviour
{
    ...

    public UnityEvent OnUnityEvent;

   ...

    private void Update() {
        if (Input.GetKeyDown(KeyCode.Space))
        {
            ...

            OnUnityEvent?.Invoke();
        }
    }
}
```

这里我们没有在 TestingEventSubscriber.cs 中使用 += 进行订阅，并且将需要订阅事件的函数设为了 public

```
// TestingEventSubscriber.cs中
...

public class TestingEventSubscriber : MonoBehaviour
{
    private void Start()
    {
        TestingEvents testingEvents = GetComponent<TestingEvents>();
        testingEvents.OnSpacePressed += TestingEvents_OnSpacePressed;
        testingEvents.OnFloatEvent += TestingEvents_OnFloatEvent;
        testingEvents.OnActionEvent += TestingEvents_OnActionEvent;
    }

    private void TestingEvents_OnSpacePressed(object sender, TestingEvents.OnSpacePressEventArgs e)
    {
        Debug.Log("Space Pressed " + e.spaceCount);
    }

    private void TestingEvents_OnFloatEvent(float f)
    {
        Debug.Log("Float Event " + f);
    }

    private void TestingEvents_OnActionEvent(bool b, int i)  
    {  
        Debug.Log("Action Event " + b + " " + i);  
    }

    public void TestingEvents_OnUnityEvent()
    {
        Debug.Log("Unity Event");
    }
}
```

使用 UnityEvent 可以在编辑器面板选择订阅该事件的函数

![[ae0ad7859ce428c6a0a281edd7bb1534_MD5.png]]

选择对应的函数，然后运行游戏，按下空格，就能看到我们所有订阅该事件的函数都被正确触发了

![[1bf19dd1b13793277684b6f0e700e1ee_MD5.gif]]