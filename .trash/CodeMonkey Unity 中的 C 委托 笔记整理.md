该笔记整理自 Youtube 上的 CodeMonkey 频道 C# 教程系列  
原教程地址：[https://youtu.be/3ZfwqWl-YI0](https://youtu.be/3ZfwqWl-YI0)

### 1 多播委托

```
using UnityEngine;

public class Testing : MonoBehaviour
{
    public delegate void TestDelegate();

    private TestDelegate testDelegateFunction;

    private void Start()
    {
        testDelegateFunction = MyTestDelegateFunction;
         // 与testDelegateFunction = new TestDelegate(MyTestDelegateFunction)相同;
        testDelegateFunction += MySecondDelegateFunction;

        testDelegateFunction();  // 调用了两个函数

        testDelegateFunction -= MySecondDelegateFunction;

        testDelegateFunction();  // 调用了一个函数
    }

    private void MyTestDelegateFunction()
    {
        Debug.Log("MyTestDelegateFunction");
    }

    private void MySecondDelegateFunction()
    {
        Debug.Log("MySecondDelegateFunction");
    }
}
```

![[0edb76da5ed43666674487f8208414e5_MD5.png]]

### 2 委托可以与匿名函数与 lambda 表达式结合使用

*   使用匿名函数与 lambda 表达式无法添加和删除该函数
*   使用匿名函数

```
using UnityEngine;

public class Testing : MonoBehaviour
{
    public delegate void TestDelegate();

    private TestDelegate testDelegateFunction;

     private void Start()  
     {  
         testDelegateFunction = delegate { Debug.Log("Anonymous method"); };  
         testDelegateFunction();  
     }
}
```

![[f7e42d075052de70d7b3431b2d7d8bbf_MD5.png]]

*   使用 lambda 表达式，括号中填参数

```
using UnityEngine;

public class Testing : MonoBehaviour
{
    public delegate bool TestDelegate(int i);

    private TestDelegate testDelegateFunction;

     private void Start()  
     {  
         testBoolDelegateFunction = (int i) => { return i < 5; };
         // 像上面这句只有一句话也可以写成下面这样
         // testBoolDelegateFunction = (int i) => i < 5;
         testBoolDelegateFunction();  
     }
}
```

![[c9a87fb3b04ba8a55c4980e9afbd68fc_MD5.png]]

### 3 内置的 Action 与 Func 委托

*   Action 类似于上面的 TestDelegate，是一个返回 void 的委托，可以和泛型一起使用，指定形参的类型

```
using System;
using UnityEngine;

public class Testing : MonoBehaviour
{
     private Action tesAction;
    private Action<int, float> testIntFloatAction;

     private void Start() {  
          testAction = () => { Debug.Log("Test Action"); };  
          testAction();

         testIntFloatAction = (int i, float f) => { Debug.Log("Test int float action!"); };  
         testIntFloatAction(1, 1.0f); 
     }
}
```

![[d4af5abd943e7b2834207e152371c01d_MD5.jpg]]

*   Func 是一个有返回值的委托，需要指定形参以及返回值的类型，尖括号中前面的是形参类型，最后一个是返回值类型

```
using System;
using UnityEngine;

public class Testing : MonoBehaviour
{
     private Func<bool> testFunc;  
     private Func<int, bool> testIntFunc;

     private void Start() {  
          testFunc = () => { return true; };  
          Debug.Log(testFunc());  

          testIntFunc = (int i) => { return i < 5; };
          Debug.Log(testIntFunc(1));
     }
}
```

![[49951deb5c7520b1fd633f49d1a43112_MD5.png]]

### 4 案例：Timer 的解耦

*   我们有一个 ActionOnTimer 类，可以设定倒计时然后判断是否完成

```
// ActionOnTimer.cs
using UnityEngine;

public class ActionOnTimer : MonoBehaviour
{
    private float timer;

    public void setTimer(float timer) {
        timer = timer;
    }

    private void Update() {
        timer -= Time.deltaTime;
    }

    public bool IsTimerDone() {
        return timer <= 0f;
    }
}
```

*   我们想用 Timer 来设定时间，然后倒计时走完时做一些事，这里把该脚本添加到一个物体上，再添加一个 Testing.cs 脚本，在这个脚本中的 Start() 中设定时间，在 Update() 中判断是否完成，在完成后我们还需要利用一个 bool 值 hasTimerElapsed 让它在之后的循环中不再输出 log 信息

```
// Testing.cs
using UnityEngine;

public class Testing : MonoBehaviour
{
    [SerializeField] private ActionOnTimer actionOnTimer;
    private bool hasTimerElapsed;

    private void Start()
    {
        actionOnTimer.setTimer(1f);
    }

    private void Update()
    {
        if (!hasTimerElapsed && actionOnTimer.IsTimerComplete())
        {
            Debug.Log("Timer complete!");
            hasTimerElapsed = true;
        }
    }
}
```

*   可以看到上面的代码耦合非常严重，关于 Timer 的语句到处都是，需要修改非常麻烦，这时可以使用 Action 委托解耦，将与 Timer 有关的实现都放到 ActionOnTimer 类中，在该类中完成 Timer 的所有逻辑

```
// ActionOnTimer.cs
using System;
using UnityEngine;

public class ActionOnTimer : MonoBehaviour
{
    private Action timerCallback;

    private float timer;

    public void setTimer(float timer, Action timerCallback) {
        this.timer = timer;
        this.timerCallback = timerCallback;
    }

    private void Update() {
        if (timer > 0f)
        {
            timer -= Time.deltaTime;
            if (IsTimerComplete())
            {
                timerCallback();
            }
        }
    }

    public bool IsTimerComplete() {
        return timer <= 0f;
    }
}
```

*   现在在 Testing.cs 就可以只在 Start() 中调用一次就完成了需求

```
using UnityEngine;

public class Testing : MonoBehaviour
{
    [SerializeField] private ActionOnTimer actionOnTimer;

    private void Start()
    {
        actionOnTimer.setTimer(1f, () => { Debug.Log("Timer completed!"); });
    }
}
```

### 5 案例：角色切换不同武器

*   我们拥有一个角色，角色可以在按下 0 键时用出拳击

```
using UnityEngine;

public class PlayerDelegates : MonoBehaviour
{
    #region Setup
    // something to setup
    #endregion

    private void Update()
    {
        switch (state)
        {
            default:
            case State.Idle:
                HandleMovement();
                HandleAttack();
                break;
            case State.Busy:
                HandleAttack();
                break;
        }
    }

    private void HandleAttack()
    {
        if (Input.GetMouseButtonDown(0))
        {
            PunchAttack();
        }
    }

    private void PunchAttack()
    {
        state = State.Busy;
        // do something
        state = State.Idle;
    }
}
```

*   现在我们希望游戏有一个升级系统，角色不光可以使用拳击，在按下 M 键升级之后也可以使用剑进行攻击，比较直接想到的是用一个 bool 值来判断角色是否处于持剑状态，如果是则使用剑进行攻击

```
using UnityEngine;

public class PlayerDelegates : MonoBehaviour
{
    #region Setup
    // something to setup
    #endregion

    private bool isUsingSword = flase;

    private void Update()
    {
        switch (state)
        {
            default:
            case State.Idle:
                HandleMovement();
                HandleAttack();
                break;
            case State.Busy:
                HandleAttack();
                break;
        }

        if (Input.GetKeyDown(KeyCode.M))
        {
            SetUseSword();
        }
    }

    private void SetUseSword()
    {
        isUsingSword = true;
    }

    private void HandleAttack()
    {
        if (Input.GetMouseButtonDown(0))
        {
            if (isUsingSword)
            {
                SwordAttack();
            }else
            {
                PunchAttack();
            }
        }
    }

    private void PunchAttack()
    {
        state = State.Busy;
        // do something
        state = State.Idle;
    }

    private void SwordAttack()
    {
        state = State.Busy;
        // do something
        state = State.Idle;
    }
}
```

*   这样代码耦合比较严重，我们可以使用 Action 委托来使代码更模块化

```
using System;
using UnityEngine;

public class PlayerDelegates : MonoBehaviour
{
    #region Setup
    // something to setup
    #endregion

    private Action attackFunction;

    private void Start()
    {
        // 默认攻击方式设置为拳击
        attackFunction = PunchAttack;
    }

    private void Update()
    {
        switch (state)
        {
            default:
            case State.Idle:
                HandleMovement();
                HandleAttack();
                break;
            case State.Busy:
                HandleAttack();
                break;
        }

        if (Input.GetKeyDown(KeyCode.M))
        {
            attackFunction = SwordAttack;
        }
    }

    private void SetUseSword()
    {
        attackFunction = SwordAttack;
    }

    private void HandleAttack()
    {
        if (Input.GetMouseButtonDown(0))
        {
            attackFunction();
        }
    }
    ...
}
```