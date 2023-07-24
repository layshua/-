![[44ba657a4c6bdb87fdf31f341e0a5c20_MD5.gif]]

原教程地址：[https://youtu.be/AmGSEH7QcDg](https://youtu.be/AmGSEH7QcDg)

B 站搬运教程地址：[https://www.bilibili.com/video/BV1gT411Z7bL/?share_source=copy_web&vd_source=10f4d7fd9e763a87da08cd00452bc8a4](https://www.bilibili.com/video/BV1gT411Z7bL/?share_source=copy_web&vd_source=10f4d7fd9e763a87da08cd00452bc8a4)

该笔记的语雀链接：

[https://www.yuque.com/wocaibuqinamochangdemingzi/akh7w4/ka1o8oxb7723ndg9?singleDoc#](https://www.yuque.com/wocaibuqinamochangdemingzi/akh7w4/ka1o8oxb7723ndg9?singleDoc#)

上篇：

[Ziur：Unity 制作类胡闹厨房游戏 KitchenChaos 笔记整理（上）](https://zhuanlan.zhihu.com/p/612934767)

## 14 创建送餐台 Delivery Counter

与上面创建其他 Counter 相同，从_BaseCounter.prefab 右键 ->Create->PrefabVariant、复制一个 Selected，改缩放，在 Scripts 文件夹下新建 DeliveryCounter.cs 继承自 BaseCounter，挂载好对应脚本，目前 DeliveryCounter 只需要销毁玩家装在盘子里送过来的菜即可

```
// DeliveryCounter.cs中
public class DeliveryCounter : BaseCounter
{
    public override void Interact(Player player)
    {
        if (player.HasKitchenObject())
        {
            if (player.GetKitchenObject().TryGetPlate( out PlateKitchenObject plateKitchenObject))
            {
                player.GetKitchenObject().DetroySelf();
            }
        }
    }
}
```

接下来使用 ShaderGraph 创建一个 shader 来实现送餐台上的箭头效果，新建文件夹 Shaders，右键 ->Create->Shader Graph->URP->Lit Shader Graph 命名为 MovingVisual.shadergraph（实际上 Unlit 在这里的效果更好，不用担心透明部分的反光问题），在_Assets/Material 文件夹下右键 ->Create->Material，新建一个材质，命名为 DeliveryArrow.mat，将 shader 文件拖动到材质文件上  
在 ShaderGraph 中，在 Graph Inspector 面板的 Graph Settings 中将 SurfaceType 改为 Transparent，用 Sample Transform 2D 采样纹理，通过将 uv 加上一个 Vector2 做出偏移的效果  

![[bed3c5519ea01fec33cf3c8f58242ed0_MD5.jpg]]

  
在 DeliveryCounter.prefab 中右键 ->Create->3D Object->Quad，将材质赋予该面片并将 BaseMap 设置为这张 Arrow 图片  

![[09b285fc3d10e97cb617fc6bdd32b595_MD5.png]]

  
运行游戏，送餐台可以正常显示效果并且删除物品  

![[3bdd0ce504dc3fb532426f1ea1a3d767_MD5.webp]]

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#deliveryCounterShaderGraph)

## 15 订单管理器与订单图标 Delivery Manager & Delivery Manager UI

## 15.1 Delivery Manager

接下来我们会实现检查通过送餐台送出的餐品是否在订单中的逻辑，首先我们要实现相应的逻辑在控制台输出的效果，下一小节会实现对应的 UI  
在 Scirpts 文件夹新建 DeliveryManager.cs，在场景中新建空物体 DeliveryManager，添加脚本  
为了检查送进去的菜品是否符合要求，我们需要保存一个菜品的 List，而每个菜品又是一个 KitchenObjectSO 的 List，所以我们需要一个 List<List<\KitchenObjectSO>>，但是我们希望保持代码尽可能整洁，所以我们先来新建一个新的 ScriptableObject 保存菜品所需的菜和菜品名称的对应关系  
在 Scripts/ScriptableObject 文件夹新建 RecipeSO.cs

```
// RecipeSO.cs中
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu()]
public class RecipeSO : ScriptableObject
{
    public List<KitchenObjectSO> kitchenObjectSOList;
    public string recipeName;
}
```

在 ScriptableObjects 文件夹下新建 RecipeSO 文件夹，在该文件夹下右键 ->Create->Recipe SO 创建 Burger.asset、Cheeseburger、MEGAburger、Salad，配置相应 List 和 Recipe Name  

![[a3befdbb637ebd53a519c300d1331c57_MD5.png]]

  
接下来我们可以直接在 DeliveryManager.cs 序列化一个 List 然后在编辑器中一个个添加来保存我们的所有菜品，但是这样做如果有其他脚本也要使用到这个拥有所有菜品的 List，就又要创建一次这个 List，所以为了不在游戏中重复创建这个 List 和之后使用的方便，我们可以用一个 ScriptableObject 来保存所有菜品，当需要用到这个有所有菜品的 List 时，只需要去添加这个 ScriptableObject 即可  
在 Scripts/ScriptableObjects 中新建 RecipeListSO.cs

```
// RecipeListSO.cs
using System.Collections.Generic;
using UnityEngine;

[CreateAssetMenu()]
public class RecipeListSO : ScriptableObject
{
    public List<RecipeSO> recipeSOList;
}
```

在 ScriptableObjects/RecipeSO 文件夹中右键 ->Create->Recipe List SO，命名为_RecipeListSO（加一个下划线是因为我们希望它显示在最前面，一会我们会将 RecipeListSO.cs 中的 [CreateAssetMenu()] 去掉，避免我们创建第二个菜品列表），在列表中添加所有菜品  

![[50da048cd6e6066738a4ccab4ee70e82_MD5.png]]

  
在 DeliveryManager.cs 中使用了单例模式，使得整个场景只能由一个 DeliveryManager，在 Update() 中每隔一段时间向等待列表随机添加菜品，并且将菜品名输出到控制台，这个脚本中还有一个 DeliverRecipe() 方法，该方法接受一个带盘子的对象，然后判断该对象是否与订单中的菜品有相同的，如果有则删除订单中的对应菜品，并在控制台输出成功信息，如果没有则输出失败信息

```
// DeliveryManager.cs
using System.Collections.Generic;
using UnityEngine;

public class DeliveryManager : MonoBehaviour
{
    public static DeliveryManager Instance { get; private set; }
    
    [SerializeField] private RecipeListSO recipeListSO;
    
    private List<RecipeSO> waitingRecipeSOList;
    private float spawnRecipeTimer;
    private float spawnRecipeTimerMax = 4f;
    private int waitingRecipesMax = 4;

    private void Awake()
    {
        Instance = this;
        waitingRecipeSOList = new List<RecipeSO>();
    }

    private void Update()
    {
        spawnRecipeTimer -= Time.deltaTime;
        if (spawnRecipeTimer <= 0f)
        {
            spawnRecipeTimer += spawnRecipeTimerMax;

            if (waitingRecipeSOList.Count < waitingRecipesMax)
            {
                RecipeSO waitingRecipeSO = recipeListSO.recipeSOList[Random.Range(0, recipeListSO.recipeSOList.Count)];
                Debug.Log(waitingRecipeSO.recipeName);
                waitingRecipeSOList.Add(waitingRecipeSO);
            }
        }
    }

    public void DeliverRecipe(PlateKitchenObject plateKitchenObject)
    {
        for (int i = 0; i < waitingRecipeSOList.Count; i++)
        {
            RecipeSO waitingRecipeSO = waitingRecipeSOList[i];
            if (waitingRecipeSO.kitchenObjectSOList.Count == plateKitchenObject.GetKitchenObjectSOList().Count)
            {
                // 订单中的菜品与送上去的菜品由同样数量的物品组成
                bool plateContentsMatchesRecipe = true;
                foreach (KitchenObjectSO recipeKitchenObjectSO in waitingRecipeSO.kitchenObjectSOList)
                {
                    // 遍历订单中的菜品所组成的物品
                    bool ingredientFound = false;
                    foreach (KitchenObjectSO plateKitchenObjectSO in plateKitchenObject.GetKitchenObjectSOList())
                    {
                        // 遍历送上去的菜品所组成的物品
                        if (recipeKitchenObjectSO == plateKitchenObjectSO)
                        {
                            // 订单中的菜品与送上去的菜品由同样的物品组成
                            ingredientFound = true;
                            break;
                        }
                    }
                    if (!ingredientFound)
                    {
                        // 订单中的菜品与送上去的菜品由不同的物品组成
                        plateContentsMatchesRecipe = false;
                    }
                }

                if (plateContentsMatchesRecipe)
                {
                    Debug.Log("Player delivered the correct recipe!");
                    waitingRecipeSOList.RemoveAt(i);
                    return;
                }
            }
        }
        // 遍历了所有订单，没有找到匹配的订单
        Debug.Log("Player did not deliver a correct recipe!");
    }
}
```

在 DeliveryCounter.cs 中与订单台交互时实例化 DeliveryManager 并调用 DeliverRecipe() 方法判断送餐与订单是否匹配  
设置好组件的变量，运行游戏，即可在控制台看到订单管理器输出的信息  

![[9cf47bd600203d019e5800536be4f941_MD5.gif]]

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#deliveryManager)

## 15.2 Delivery Manager UI

在场景中右键 ->UI->Canvas 新建一个 Canvas，将 Canvas Scaler 中的 UI Scale Mode 改为 Scale With Screen Size，将 Reference Resolution 改为 1920 1080，将 Match 改为 1，完全匹配高度  

![[f84f293ce4a8421cebbaca249a7ef2a3_MD5.png]]

  
这样设置意味着 Canvas 上的 UI 在游戏视口宽度改变时不会改变，在高度改变时会等比例缩小，选择这样的设置是因为这样我们就可以只关心 UI 的横向排布，而当纵向改变时该组件会自动帮我们缩放，如图，可以在 Canvas 下创建一个 Image，颜色改为红色进行测试  

![[18a3747fc8d2d49f1cbee9d28706a3bd_MD5.gif]]

  
在 Canvas 下新建空物体命名为 DeliveryManagerUI，在 AnchorPresets 中调整为随父级宽度高度都拉伸，将 Left、Top、Right、Bottom 都设为 0 以填充 Canvas 大小  

![[e96ada57805f35ec45e55a62562aaf26_MD5.png]]

  
在 DeliveryManagerUI 下右键 ->UI->Text 命名为 TitleText，内容写上”RECIPES WAITING...“，字体设置为加粗（Bold），将锚点设置到左上角，Width 和 Hight 设为 0，Wrapping 设为 Disabled，移动到合适位置，可以打开 Game 窗口查看位置  

![[2ded03d7ce73d6eb484133ec63ca80e5_MD5.png]]

  
在 DeliveryManagerUI 下新建空物体命名为 Container，锚点设置到左上角，调整位置到刚刚设置的文字的下方；Container 下新建空物体命名为 RecipeTemplate，锚点和位置设置到左上角（按住 Shift 设置锚点即可一块调整位置）；在 RecipeTemplate 下右键 ->UI->Image，同样调整为随父级宽度高度都拉伸，Left、Top、Right、Bottom 都设为 0，颜色调整为黑色，透明度降低一些；在 RecipeTemplate 下右键 ->UI->Text，锚点改为右上角，调整位置，字体大小改为 20，Width、Hight 改为 0，Wrapping 设为 Disabled  

![[fce3d39713f14d9e9a18cf8e5e85528b_MD5.png]]

  
在 Countainer 上添加 VerticalLayoutGroup 组件，Spacing（间距）增加一些，复制几个 Recipe Template，就能看到纵向排布的订单列表  

![[0c5bc74e073135510c1ada6810779e7e_MD5.png]]

  
在 Scripts 文件夹下新建 DeliveryManagerUI.cs，添加到 DeliveryManagerUI 上，在 DeliveryManager.cs 中，新添加两个 EventHandler 委托，分别在订单出现和订单完成时触发，这两个 EvnetHandler 委托调用的函数写在 DeliveryManagerUI.cs，主要是去更新 UI，现在我们更新 UI 的逻辑只需要处理一下订单的初始化然后遍历订单中的每一个菜品然后实例化 recipeTemplate（因为 DeliveryManager 用了单例模式，我们可以在 DeliveryManager.cs 中写一个 public 的 GetWaitingRecipeSOList() 方法在这里调用）。另外使用 EventHandler 需要使用 using System，而 System 库和 UnityEngine 库中都有 Random() 这个函数，所以这里使用 Random() 时使用 UnityEngine.Random 说明要使用 UnityEngine 库中的 Random()

```
// DeliveryManager.cs
...
public class DeliveryManager : MonoBehaviour
{
    public event EventHandler OnRecipeSpawned;
    public event EventHandler OnRecipeCompleted;  
    ...
    private void Update()
    {
        ...
            if (waitingRecipeSOList.Count < waitingRecipesMax)
            {
                // 生成订单
                RecipeSO waitingRecipeSO = recipeListSO.recipeSOList[UnityEngine.Random.Range(0, recipeListSO.recipeSOList.Count)];
                
                OnRecipeSpawned?.Invoke(this, EventArgs.Empty);
            }
        }
    }

    public void DeliverRecipe(PlateKitchenObject plateKitchenObject)
    {
        for (int i = 0; i < waitingRecipeSOList.Count; i++)
        {
            ...
            if (waitingRecipeSO.kitchenObjectSOList.Count == plateKitchenObject.GetKitchenObjectSOList().Count)
            {
                ...
                if (plateContentsMatchesRecipe)
                {
                    ...
                    OnRecipeCompleted?.Invoke(this, EventArgs.Empty);
                    return;
                }
            }
        }
        // 遍历了所有订单，没有找到匹配的订单
    }
    
    public List<RecipeSO> GetWaitingRecipeSOList()
    {
        return waitingRecipeSOList;
    } 
}
```

```
// DeliveryManagerUI.cs中
using System;
using UnityEngine;

public class DeliveryManagerUI : MonoBehaviour
{
    [SerializeField] private Transform container;
    [SerializeField] private Transform recipeTemplate;

    private void Awake()
    {
        recipeTemplate.gameObject.SetActive(false);
    }

    private void Start()
    {
        DeliveryManager.Instance.OnRecipeSpawned += DeliveryManager_OnRecipeSpawned;
        DeliveryManager.Instance.OnRecipeCompleted += DeliveryManager_OnRecipeCompleted;
        UpdateVisual();
    }
    
    private void DeliveryManager_OnRecipeSpawned(object sender, EventArgs e)
    {
        UpdateVisual();
    }
    
    private void DeliveryManager_OnRecipeCompleted(object sender, EventArgs e)
    {
        UpdateVisual();
    }

    private void UpdateVisual()
    {
        foreach (Transform child in container)
        {
            if (child == recipeTemplate) continue;
            Destroy(child.gameObject);
        }

        foreach (RecipeSO recipeSO in DeliveryManager.Instance.GetWaitingRecipeSOList())
        {
            Transform recipeTransform = Instantiate(recipeTemplate, container);
            recipeTransform.gameObject.SetActive(true);
        }
    }
}
```

运行游戏，可以看到每隔四秒生成一个新的 RecipeTemplate  

![[810761421549c182a4e1cdedee3d9703_MD5.gif]]

  
接下来根据生成订单的不同改变 Text 的内容，而不是都显示 Recipe，我们可以直接用 Find("RecipeNameText") 找到显示文字的子级对象，但我们并不希望使用字符串，所以这里依然选择新创建一个脚本实现获取该对象上 TextMeshPro 组件的功能  
在 Scripts 文件夹下新建 DeliveryManagerSingleUI.cs，将该脚本添加到 RecipeTemplate 上，由于我们已经有了很多关于 UI 的脚本，所以这里整理了一下，在 Scripts 文件夹下新建了一个 UI 文件夹，将与 UI 相关的脚本都放进去了  
在 DeliveryManagerSingleUI.cs 中，我们接收一个从 UI->Text 创建的对象然后改变内容，注意这里要写的类为 TextMeshProGUI，如果我们是从 3D Object->Text 创建的对象，才应该用 TextMeshPro 类

```
// DeliveryManagerSingleUI.cs中
using TMPro;
using UnityEngine;

public class DeliveryManagerSingleUI : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI recipeNameText;
    
    public void setRecipeSO(RecipeSO recipeSO)
    {
        recipeNameText.text = recipeSO.recipeName;
    }
}
```

在编辑器面板设置好变量，在 DeliveryManagerUI.cs 中更新 UI 的方法中调用 setRecipeSO 方法将 UI 上的文字换为从 DeliveryManager 接收到菜品的名称

```
// DeliveryManagerUI.cs中
...
public class DeliveryManagerUI : MonoBehaviour
{
    ...

    private void UpdateVisual()
    {
        ...
        foreach (RecipeSO recipeSO in DeliveryManager.Instance.GetWaitingRecipeSOList())
        {
            ...
            recipeTransform.GetComponent<DeliveryManagerSingleUI>().setRecipeSO(recipeSO);
        }
    }
}
```

运行游戏，就能看到生成的订单 UI 上显示了菜品名称  

![[b95e04de54b75d4388be48a19a2a62d6_MD5.png]]

  
接下来开始处理图标的显示，我们希望在每个 RecipeTemplate 中显示制作该菜品所需的物品  
在 RecipeTemplate 下新建空物体命名为 IconContainer，将 Width 和 Height 设置为 0，移到合适位置；在 IconContainer 下右键 ->UI->Image 命名为 IngredientImage，调整 Width 和 Height，暂时设置一张图标；在 IconContainer 上添加 Horizontal Layout Group 组件，多复制几个 IngredientImage 查看效果  

![[8d3f5bde08713f191153b505000f4322_MD5.png]]

  
接下来在 DeliveryManagerSingleUI.cs 中写对应的初始化和更新图标的逻辑

```
// DeliveryManagerSingleUI.cs中
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class DeliveryManagerSingleUI : MonoBehaviour
{
    ...
    [SerializeField] private Transform iconContainer;
    [SerializeField] private Transform iconTemplate;

    private void Awake()
    {
        iconTemplate.gameObject.SetActive(false);
    }
    
    public void setRecipeSO(RecipeSO recipeSO)
    {
        ...
        foreach (Transform child in iconContainer)
        {
            if (child == iconTemplate) continue;
            Destroy(child.gameObject);
        }

        foreach (KitchenObjectSO kitchenObjectSO in recipeSO.kitchenObjectSOList)
        {
            Transform iconTransform = Instantiate(iconTemplate, iconContainer);
            iconTransform.gameObject.SetActive(true);
            iconTransform.GetComponent<Image>().sprite = kitchenObjectSO.sprite;
        }
    }
}
```

在编辑器面板设置相应参数，运行游戏即可看到订单 UI 处显示了对应图标  

![[e670c1a47d43177dcb3dfb5862217fa5_MD5.webp]]

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#deliveryManagerUI)

## 16 背景音乐与音效 Music & Sound Effects

## 16.1 背景音乐

添加背景音乐很简单，在_Assets/Sounds/Music 下有一个 Music.wav 文件，该文件是一个做好的可以循环播放的背景音乐，该在场景中新建空物体命名为 MusicManager，添加一个 Audio Source 组件，将 AudioClip 设置为准备好的 Music.wav 文件，勾上 Loop 让音乐循环播放，设置 Priority 到 0，在 unity 中不能同时播放过多的音乐，Priority 的值越大在音频过多时就越可能不被播放，Volume 在后面的教程中会有在游戏中调整的方法，这里先设置为 0.5  

![[17ca31edcf18361d20ba2640120b4bbb_MD5.png]]

  
确保 Main Camera 上有 Audio Listener 组件，运行游戏即可听到声音。另外 unity 中还有 Audio mixer 可以进行简单的混音，这里游戏中没有使用  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#music)

## 16.2 音效

### 16.2.1 SoundManager & AudioClipRefsSO

除了使用 Audio Source 组件，在 unity 中还可以通过 AudioSource.playClipAtPoint(AudioClip clip, Vector3 position, float volume = 1.0F) 来播放音频（但是相比 AudioSource 很多东西都无法调整，如果想要调整 AudioSource 中的东西可以给每个音频都创建一个 prefab，然后想要使用音频时实例化该 prefab）  
在场景中创建一个空物体命名为 SoundManager，在 Scripts 文件夹新建 SoundManager.cs 添加到 SoundManager 上，与 15.1 中创建表示所有菜品的 List 的需求相同，这里我们也希望将所有音效通过 ScriptableObject 创建的对象保存起来，所以在写其他逻辑之前，在 Scripts/ScripatableObjects 文件夹下新建 AudioClipRefsSO.cs

```
// AudioClipRefsSO.cs中
using UnityEngine;

[CreateAssetMenu]
public class AudioClipRefsSO : ScriptableObject
{
    public AudioClip[] chop;
    public AudioClip[] deliveryFail;
    public AudioClip[] deliverySuccess;
    public AudioClip[] footstep;
    public AudioClip[] objectDrop;
    public AudioClip[] objectPickup;
    public AudioClip stoveSizzle;
    public AudioClip[] trash;
    public AudioClip[] warning;
}
```

在 ScriptableObjects 文件夹中右键 ->Create->Audio Clip Refs SO，根据不同种类将_Assets/Sounds/SFX 中的音频文件按分类添加好  

![[4b214975c0fd4950f3ab25f2450b331d_MD5.jpg]]

### 16.2.2 送餐台音效

首先处理送餐台的音效，在 DeliveryManager.cs 中，我们声明两个 EventHandler 委托，分别在菜品匹配订单成功和失败时触发，在 SoundManager.cs 中为这两个委托添加调用的函数，播放对应的的音效。由于我们的游戏中只有一个送餐台，所以我们也可以在 DeliveryCounter.cs 中使用单例模式，在 SoundManager.cs 获取该实例的位置，这样我们就可以在送餐台的位置播放对应音效

```
// DeliveryManager.cs中
...
public class DeliveryManager : MonoBehaviour
{
    ...
    public void DeliverRecipe(PlateKitchenObject plateKitchenObject) {
        for (int i = 0; i < waitingRecipeSOList.Count; i++)
        {
            ...
            if (waitingRecipeSO.kitchenObjectSOList.Count == plateKitchenObject.GetKitchenObjectSOList().Count)
            {
                ...
                if (plateContentsMatchesRecipe)
                {
                    // 找到了匹配的订单，送餐成功
                    OnRecipeSuccess?.Invoke(this, EventArgs.Empty);
                    ...
                }
            }
        }
        // 遍历了所有订单，没有找到匹配的订单
        OnRecipeFailed?.Invoke(this, EventArgs.Empty);
    }
    ...
}
```

```
// DeliveryCounter.cs中
public class DeliveryCounter : BaseCounter
{
    public static DeliveryCounter Instance { get; private set; }
    
    private void Awake()
    {
        Instance = this;
    }
    
    public override void Interact(Player player)
    {
        if (player.HasKitchenObject())
        {
            if (player.GetKitchenObject().TryGetPlate( out PlateKitchenObject plateKitchenObject))
            {
                DeliveryManager.Instance.DeliverRecipe(plateKitchenObject);
                player.GetKitchenObject().DetroySelf();
            }
        }
    }
}
```

```
// SoundManager.cs中
using System;
using UnityEngine;

public class SoundManager : MonoBehaviour
{
    [SerializeField] private AudioClipRefsSO audioClipRefsSO;
    
    private void Start()
    {
        DeliveryManager.Instance.OnRecipeSuccess += DeliveryManager_OnRecipeSuccess;
        DeliveryManager.Instance.OnRecipeFailed += DeliveryManager_OnRecipeFailed;
    }
    
    private void DeliveryManager_OnRecipeSuccess(object sender, EventArgs e)
    {
        DeliveryCounter deliveryCounter = DeliveryCounter.Instance;
        PlaySound(audioClipRefsSO.deliverySuccess, deliveryCounter.transform.position);
    }

    private void DeliveryManager_OnRecipeFailed(object sender, EventArgs e)
    {
        DeliveryCounter deliveryCounter = DeliveryCounter.Instance;
        PlaySound(audioClipRefsSO.deliveryFail, deliveryCounter.transform.position);
    }

    private void PlaySound(AudioClip[] audioClipArray, Vector3 position, float volume = 1f)
    {
        PlaySound(audioClipArray[UnityEngine.Random.Range(0, audioClipArray.Length)], position, volume);
    }
    
    private void PlaySound(AudioClip audioClip, Vector3 position, float volume = 1f)
    {
        AudioSource.PlayClipAtPoint(audioClip, position, volume);
    }
}
```

### 16.2.3 切菜台音效

由于我们已经在做切菜动画时在 CuttingCounter.cs 中定义了一个叫 OnCut 的 EventHandler 委托，该委托在每次按下 F 键时触发，在这里我们只需要为这个委托添加一个播放对应音效的函数即可  
与送餐台不同的是，场景中有多个切菜台，我们并不希望 SoundManager 订阅所有的切菜台，所以不能使用单例模式让场景中只存在一个 CuttingCounter 实例，然而我们可以声明一个静态的委托，让所有切菜台在触发 OnCut 委托时也触发这个委托，这样我们就可以只为该委托添加触发的函数

```
// CuttingCounter.cs中
...
public class CuttingCounter : BaseCounter, IHasProgress
{
    public static EventHandler OnAnyCut;
    ...  
    public override void InteractAlternate(Player player)
    {
        if (HasKitchenObject() && HasRecipeWithInput(GetKitchenObject().GetKitchenObjectSO()))
        {
            ...
            // 柜子上有物品，开始切菜
            OnAnyCut?.Invoke(this, EventArgs.Empty);
            ...
        }
    }
    ...
}
```

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    ...
    private void Start()
    {
        ...
        CuttingCounter.OnAnyCut += CuttingCounter_OnAnyCut;
    } 
    ...
    private void CuttingCounter_OnAnyCut(object sender, EventArgs e)
    {
        CuttingCounter cuttingCounter = sender as CuttingCounter;
        PlaySound(audioClipRefsSO.chop, cuttingCounter.transform.position);
    }
    ...
}
```

### 16.2.4 角色拿起物品音效

在 Player.cs 中我们写过一个 SetKitchenObject() 方法，每次在角色拿起物品时都会调用此方法，所以刚好适合我们用来做拿起物品的音效  
在 Player.cs 中，声明一个 EventHandler 委托，在调用 SetKitchenObject() 方法时触发，在 SoundManager.cs 中添加触发的函数，播放对应音频

```
// Player.cs
...
public class Player : MonoBehaviour, IKitchenObjectParent
{
    ...
    public event EventHandler OnPickUpSomething;
    ...
    public void SetKitchenObject(KitchenObject kitchenObject)
    {
        ...
        if (kitchenObject != null)
        {
            OnPickUpSomething?.Invoke(this, EventArgs.Empty);
        }
    }
}
```

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    ...
    private void Start()
    {
        ...
        Player.Instance.OnPickUpSomething += Player_OnPickUpSomething;
    }
    ... 
    private void Player_OnPickUpSomething(object sender, EventArgs e)
    {
        PlaySound(audioClipRefsSO.objectPickup, Player.Instance.transform.position);
    }
}
```

### 16.2.5 角色放下物品音效

与拿起物品类似，我们在 BaseCounter.cs 中也有一个 SetKitchenObject() 方法，在角色向柜台放物品时触发，所以同样在 BaseCounter.cs 中添加一个 EventHandler 委托，在 SetKitchenObject() 中触发，在 SoundManager.cs 中添加调用的函数，与切菜台类似，也声明一个静态的委托

```
// BaseCounter.cs中
...
public class BaseCounter : MonoBehaviour, IKitchenObjectParent
{
    ...
    public static event EventHandler OnAnyObjectPlacedHere;
    ...
    public void SetKitchenObject(KitchenObject kitchenObject)
    {
        this.kitchenObject = kitchenObject;
        
        if (kitchenObject != null)
        {
            OnAnyObjectPlacedHere?.Invoke(this, EventArgs.Empty);
        }
    }
}
```

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    ...
    private void Start()
    {
        ...
        BaseCounter.OnAnyObjectPlacedHere += BaseCounter_OnAnyObjectPlacedHere;
    }
    ... 
    private void Player_OnPickUpSomething(object sender, EventArgs e)
    {
        PlaySound(audioClipRefsSO.objectPickup, Player.Instance.transform.position);
    }
}
```

### 16.2.6 扔垃圾音效

我们的垃圾箱类也继承了 BaseCounter，但是并没有实现 GetKitchenObject() 方法，而是直接删除角色手上的物品，所以只在 Interact() 方法中触发委托即可

```
// TrashCounter.cs中
...
public class TrashCounter : BaseCounter
{
    public static event EventHandler OnAnyObjectTrashed;
    
    public override void Interact(Player player)
    {
        if (player.HasKitchenObject())
        {
            ...
            OnAnyObjectTrashed?.Invoke(this, EventArgs.Empty);
        }
    }
}
```

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    ...
    private void Start()
    {
        ...
        TrashCounter.OnAnyObjectTrashed += TrashCounter_OnAnyObjectTrashed;
    }
    ... 
    private void TrashCounter_OnAnyObjectTrashed(object sender, EventArgs e)
    {
        TrashCounter trashCounter = sender as TrashCounter;
        PlaySound(audioClipRefsSO.trash, trashCounter.transform.position);
    }
}
```

### 16.2.7 炉灶台音效

炉灶台音效有些不同，我们希望根据炉灶台循环播放音频，并根据不同的状态来播放和暂停，所以我们在 StoveCounter.prefab 中单独创建控制音频的物体用 AudioSouce 组件和单独写的脚本来控制音频  
在 StoveCounter.prefab 中新建空物体命名为 Sound，AudioClip 选择对应的炉灶台的声音，在该空物体上添加 Audio Source 组件，取消勾选 Play On Awake、勾选 Loop、Spatial Blend 改为 1 让它是一个 3d 音效  

![[1d8c2d0ffc3110bf1b2e3df7260f2b60_MD5.png]]

  
在 Scripts 文件夹新建 StoveCounterSound.cs 添加到 Sound 物体上，之前在 StoveCounter.cs 中我们有一个 OnStateChanged 的 EventHandler 委托，该委托在状态改变时触发，我们在 toveCounterSound.cs 中为该委托添加调用的函数，当当前炉灶台的状态为 Frying 或 Fried 的时候开始播放音频，其他状态时暂停

```
// StoveCounterSound中
using UnityEngine;

public class StoveCounterSound : MonoBehaviour
{
    [SerializeField] private StoveCounter stoveCounter;
    private AudioSource audioSource;

    private void Awake()
    {
        audioSource = GetComponent<AudioSource>();
    }

    private void Start()
    {
        stoveCounter.OnStateChanged += StoveCounter_OnStateChanged;
    }
    
    private void StoveCounter_OnStateChanged(object sender, StoveCounter.OnStateChangedEventArgs e)
    {
        bool playSound = e.state == StoveCounter.State.Frying || e.state == StoveCounter.State.Fried;
        if (playSound)
        {
            audioSource.Play();
        } else
        {
            audioSource.Pause();
        }
    }
}
```

### 16.2.8 角色脚步声

角色脚步声我们使用和炉灶台差不多的方式，同样用一个单独的脚本来控制音频，不过这次我们不再在 Player 下创建一个子物体，而是直接给 Player 本身添加脚本  
新建 PlayerSounds.cs，添加到 Player 上，我们的脚步声音频文件很短，我们可以在 PlayerSounds.cs 中每隔 `footstepTimerMax` 秒来检测角色是否在行走而判断是否播放音频，我们可以像其他对象一样在 PlayerSounds.cs 声明 EventHandler 委托，然后在 SoundManager.cs 中添加其调用的函数，或者我们也可以在 Player.cs 中直接调用 SoundManager 中的方法去播放音频，第二种做法意味着我们的 PlayerSounds.cs 和 SoundManager.cs 没有很好的解耦，然而我们 PlayerSounds 这个类就不是为了独立存在而设计的，而是必须要与 SoundManager 类一起使用，所以这种情况下我们可以直接去这样调用方法而不使用事件  
我们可以在 SoundManager.cs 中使用单例模式或者在 PlayerSounds 中用 SerializeField 接收该对象，这里我们使用单例模式

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    public static SoundManager Instance { get; private set; }
    
    private void Awake()
    {
        Instance = this;
    }
    ...
    public void PlayFootStepsSound(Vector3 position, float volume)
    {
        PlaySound(audioClipRefsSO.footstep, position, volume);
    }
}
```

```
// PlayerSounds.cs中
using UnityEngine;

public class PlayerSounds : MonoBehaviour
{
    private Player player;
    private float footstepTimer;
    private float footstepTimerMax = 0.1f;
    
    private void Awake() {
        player = GetComponent<Player>();
    }

    private void Update() {
        footstepTimer -= Time.deltaTime;
        if (footstepTimer < 0f)
        {
            footstepTimer = footstepTimerMax;

            if (player.IsWalking())
            {
                float volume = 1f;
                SoundManager.Instance.PlayFootStepsSound(player.transform.position, volume);    
            }
        }
    }
}
```

启动游戏，即可听见游戏内的各种音效，目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#soundEffects)

## 17 其他界面

## 17.1 Scene Manager

在场景中新建空物体命名为 KitchenObjectManager，Scripts 文件夹新建 KitchenObjectManager.cs 添加到该对象上  
我们同样用状态机来表示游戏的各个状态，在一开始我们先用计时器来控制各个状态间的改变，这里我们也使用了单例模式

```
// KitchenObjectManager.cs
using System;
using UnityEngine;

public class KitchenGameManager : MonoBehaviour {

    public static KitchenGameManager Instance { get; private set; }

    public event EventHandler OnStateChanged;

    private enum State {
        WaitingToStart,
        CountdownToStart,
        GamePlaying,
        GameOver,
    }
    
    private State state;
    private float waitingToStartTimer = 1f;
    private float countdownToStartTimer = 3f;
    private float gamePlayingTimer = 10f;
    
    private void Awake() {
        Instance = this;

        state = State.WaitingToStart;
    }

    private void Update() {
        switch (state) {
            case State.WaitingToStart:
                waitingToStartTimer -= Time.deltaTime;
                if (waitingToStartTimer < 0f) {
                    state = State.CountdownToStart;
                }
                break;
            case State.CountdownToStart:
                countdownToStartTimer -= Time.deltaTime;
                if (countdownToStartTimer < 0f) {
                    state = State.GamePlaying;
                }
                break;
            case State.GamePlaying:
                gamePlayingTimer -= Time.deltaTime;
                if (gamePlayingTimer < 0f) {
                    state = State.GameOver;
                }
                break;
            case State.GameOver:
                break;
        }
        Debug.Log(state);
    }
}
```

这时运行游戏，可以看到控制台可以根据计时器输出当前状态  

![[d1ac7ddc84eb4cdf5b149cc7d8798ea1_MD5.jpg]]

## 17.2 开始倒计时与结束界面

我们希望角色在除了状态为 GamePlaying 时可以让角色与物品交互，其他状态都不能交互，在 KitchenObjectManager.cs 中，添加一个方法来判断当前状态是否为 GamePlaying，在 Player.cs 中调用角色交互相关的方法前使用上面的方法

```
// KitchenObjectManager.cs中
...
public class KitchenGameManager : MonoBehaviour {
    ...
    public bool IsGamePlaying() {
        return state == State.GamePlaying;
    }
}
```

```
// Player.cs中
...
public class Player : MonoBehaviour, IKitchenObjectParent {
    ...
    private void GameInput_OnInteractAlternateAction(object sender, EventArgs e) {
        if (!KitchenGameManager.Instance.IsGamePlaying()) return;
        ...
    }

    private void GameInput_OnInteractAction(object sender, System.EventArgs e) {
        if (!KitchenGameManager.Instance.IsGamePlaying()) return;
        ...
    }
    ...
}
```

接下来我们在游戏在 CountdownToStart 状态时显示一个倒计时，在 Canvas 新建空物体命名为 GameStartCountdownUI，将位置宽高属性都改为 0，右键 ->UI->Text 新建文本对象命名为 CountdownText，将位置宽高属性都改为 0、Warpping Disabled、Alignment 改为 center 和 Middle、内容暂时写上 3，后面的文字内容基本上都类似设置的，如果没有太大区别笔记里不再写了  
TextMeshPro 默认的材质是全局的，所以如果我们直接在下面的材质面板更改颜色描边等属性，游戏中其他的文本的材质也会变化，为了单独更改该对象的材质，我们双击 Font Asset  

![[160ebd6333bc6a436576d7c97f1c2e32_MD5.png]]

  
在文件夹中找到这个文字文件，复制该文件下的材质文件 (.mat)，重命名为 LiberationSans SDF StartCountdown，注意名字的前面部分不能改变，否则无法在面板进行选择  

![[b9c48a9035d00f21893d60537fd630f7_MD5.png]]

  
改变刚刚的 Font Asset 下面的 Material Preset 为我们复制的 StartCountdown 材质，在调整文字属性、材质面板中设置 Outline 后即可在场景中看到效果，还可以在 Game 窗口看到游戏运行起来的大致效果  

![[2b7760e54d365dd11bb97144a7717b20_MD5.png]]

  
在 Scripts/UI 文件夹下新建 GameStartCountdown.cs，添加到场景的对象上，在 KitchenGameManager.cs 中添加两个方法，一个判断当前状态是否为 CountdownToStart 状态，另一个得到 CountdownToStart 状态的计时器，再添加一个 EventHandler 委托，状态改变时触发该委托；在 GameStartCountdown.cs 判断状态后改变倒计时文本的显示与隐藏，在 Update() 根据倒计时显示对应文本（我们可以在每帧都去发送事件传递当前计时器的秒数，但是这样做比较耗时，所以我们直接在 KitchenGameManager.cs 中添加了一个方法来得到当前计时器的值）

```
// KitchenGameManager.cs中
...
public class KitchenGameManager : MonoBehaviour {
    ...
    private void Update() {
        switch (state) {
            case State.WaitingToStart:
                ...
                if (waitingToStartTimer < 0f) {
                   ...
                    OnStateChanged?.Invoke(this, EventArgs.Empty);
                }
                break;
            case State.CountdownToStart:
                ...
                if (countdownToStartTimer < 0f) {
                    state = State.GamePlaying;
                    ...
                }
                break;
            case State.GamePlaying:
                ...
                if (gamePlayingTimer < 0f) {
                    ...
                    OnStateChanged?.Invoke(this, EventArgs.Empty);
                }
                break;
            case State.GameOver:
                break;
        }
        Debug.Log(state);
    }
    ...
    public bool IsCountdownToStartActive() {
        return state == State.CountdownToStart;
    }

    public float GetCountdownToStartTimer() {
        return countdownToStartTimer;
    }
}
```

```
// GameStartCountdown.cs中
using TMPro;
using UnityEngine;

public class GameStartCountdownUI : MonoBehaviour {

    [SerializeField] private TextMeshProUGUI countdownText;

    private void Start() {
        KitchenGameManager.Instance.OnStateChanged += KitchenGameManager_OnStateChanged;
        Hide();
    }

    private void KitchenGameManager_OnStateChanged(object sender, System.EventArgs e) {
        if (KitchenGameManager.Instance.IsCountdownToStartActive()) {
            Show();
        } else {
            Hide();
        }
    }

    private void Update() {
        countdownText.text = Mathf.Ceil(KitchenGameManager.Instance.GetCountdownToStartTimer()).ToString();
    }

    private void Show() {
        gameObject.SetActive(true);
    }

    private void Hide() {
        gameObject.SetActive(false);
    }
}
```

运行游戏，可以看见倒计时，目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#gameStart)  

![[02d40358ce8d70ddc53e3bcbf1026492_MD5.gif]]

  
接下来做一下结束画面，我们希望在结束画面中显示我们成功送餐的数量  
在 Cancas 文件夹下新建空物体命名为 GameOverUI，更改属性；新建 Image 命名为 Background，改为 stretch，更改属性，颜色调为黑色，透明度降低一些；新建 Text 分别命名为 GameOverText、LabelRecipesDeliveredText、RecipesDeliveredText，更改相应属性  

![[34645980a04dbb876b42604e1b862341_MD5.png]]

  
在 Scripts/UI 文件夹中新建 GameOverUI.cs 添加到对象上，我们需要在 KitchenGameManager.cs 中添加一个方法判断当前是否为 GameOver 状态，在 DeliveryManager.cs 中保存成功送餐的数量并添加相应方法获取该数量，在 GameOverUI.cs 中隐藏与显示 UI

```
// KitchenGameManager.cs中
...
public class KitchenGameManager : MonoBehaviour
{
    ...
    public bool IsGameOver() {
        return state == State.GameOver;
    }
    ...
}
```

```
// DeliveryManager.cs中
...
public class DeliveryManager : MonoBehaviour
{
    ...
    private int successfulRecipesAmount;
    ...
    public void DeliverRecipe(PlateKitchenObject plateKitchenObject) {
        ...
        for (int i = 0; i < waitingRecipeSOList.Count; i++)
        {
             ...
             // 找到了匹配的订单，送餐成功
             successfulRecipesAmount++;
             ...
        }
        ...
    }
    ... 
    public int GetSuccessfulRecipesAmount() {
        return successfulRecipesAmount;
    }
}
```

```
// GameOverUI.cs中
using TMPro;
using UnityEngine;

public class GameOverUI : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI recipesDeliveredText;
    
    private void Start()
    {
        KitchenGameManager.Instance.OnStateChanged += KitchenGameManager_OnGameStateChanged;
        
        Hide();
    }
    
    private void KitchenGameManager_OnGameStateChanged(object sender, System.EventArgs e)
    {
        if (KitchenGameManager.Instance.IsGameOver())
        {
            Show();
            recipesDeliveredText.text = DeliveryManager.Instance.GetSuccessfulRecipesAmount().ToString();
        } else
        { 
            Hide();  
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

运行游戏，当游戏结束时即可看到对应 UI  

![[c0714986d7fea41c053b4c8e93f23732_MD5.jpg]]

  
接下来在游戏运行场景的右上角做一个圆形的进度条来表示开始游戏后过去的时间  
在 Canvas 下新建空物体命名为 GamePlayingClockUI，更改属性调整锚点；在该物体下新建 Image 命名为 Background，SourceImage 选择 Circle，调整颜色，可以添加 Outline 和 Shadow 组件；复制 Background 改名为 TimerImage，注意顺序放到 Background 的后面，主要将 Image Type 改为 Fiiled，其他属性进行调整，添加组件（下图为 TimerImage 的 Inspector 面板）  

![[5ed0a9e0604a6c1681db6559697109db_MD5.png]]

![[6948b6e1f94c8702ae826bf28efab643_MD5.png]]

  
在 Scripts/UI 文件夹中新建 GamePlayingClockUI.cs，添加到对象上，在 KitchenGameManager.cs 中添加一个方法得到归一化后的计时器的值，在 GamePlayingClockUI.cs 中根据归一化的值填充图形

```
// KitchenGameManager.cs中
...
public class KitchenGameManager : MonoBehaviour
{
    ...
    private float gamePlayingTimerMax = 10f;
    ...
    public float GetGamePlayingTimerNormalized() {
        return 1 - gamePlayingTimer / gamePlayingTimerMax;
    }
    ...
}
```

运行游戏，即可看到圆形的进度条  

![[2f35412d5c26dde7439a75638deeab91_MD5.gif]]

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#gameOver)

## 17.3 主菜单页面与加载页面

在 Scenes 文件夹右键 ->Create->Scene 新建一个场景，命名为 MainMenuScene.unity；在场景中右键 ->UI->Canvas 新建一个 Canvas，将 Canvas Scaler 中的 UI Scale Mode 改为 Scale With Screen Size，将 Reference Resolution 改为 1920 1080，将 Match 改为 1，完全匹配高度；在 Canvas 下新建一个空物体，命名为 MainMenuUI，改为 stretch；在 MainMenuUI 下右键 ->UI->Button 新建 PlayButton，加组件调整属性，复制一个改名为 QuitButton，安排好后效果如下  

![[d7f728b0995bfd489dea942a8d298dda_MD5.png]]

  
我们可以装饰一下菜单场景，将 GameScene 中的 GlobalVolume 和 Plane 复制到 MainMenuScene，可以将_Assets/PrefabsVisuals 中的、PlayerViusal.prefab 拖动到场景，还可以改变其材质、使用 cineMachine 插件添加 Virtual Camera、在 MainMenuUI 下添加 Image，Source Image 选择 KitchenChaosLogo 添加游戏 logo，最终效果如下  

![[5337b87505c5428bbabbc19b7b017848_MD5.gif]]

  
在 File->Build Settings 中将 MainMenuScene 添加为第一个场景，对应的索引为 0  

![[0a7b960d0337121dfcd8bc71efe62561_MD5.png]]

  
在 Scripts/UI 文件夹下新建 MainMenuUI.cs，将脚本挂载到对象上，用匿名函数给两个按钮添加按下按钮调用的方法

```
// MainMenuUI.cs
using UnityEngine;
using UnityEngine.UI;

public class MainMenuUI : MonoBehaviour
{
    [SerializeField] private Button playButton;
    [SerializeField] private Button quitButton;

    private void Awake()
    {
        playButton.onClick.AddListener(() =>
        {
            SceneManager.LoadScene(1);
        });
        
        quitButton.onClick.AddListener(() => 
        {
            Application.Quit();
        });
    }
}
```

如果现在就运行游戏，我们会在主菜单场景和游戏场景中卡顿一下，这是因为我们的游戏场景加载需要一定时间，这样的 “冻结” 的画面会让玩家体验很不好，并且如果游戏内容很多，加载的时间会更长，所以我们来在中间加一个加载的场景  
在 Scenes 文件夹新建场景 LoadingScene.unity；在 MainCamera 中将 Background 改为黑色；新建 Canvas，属性修改同上；在 Canvas 下右键 ->UI->Text，调整属性与文字内容，打开 Game 窗口效果如下  

![[a55bfaebaec83c8187a7fe06d18ab768_MD5.png]]

  
我们要在主菜单的场景通过脚本来到这个加载场景，然而当场景变化时，一个场景中的物体都会被销毁，所以我们需要一个特殊的脚本能在场景之间传输数据，在主菜单场景告诉加载场景我们要去的场景是哪个（我们也可以用 [Object.DontDestroyOnLoad](https://docs.unity3d.com/ScriptReference/Object.DontDestroyOnLoad.html) 来让某个对象在场景切换时不被销毁，不过这里我们使用其他的方法）  
在 Scripts 文件夹新建 Loader.cs，我们需要一个 Static 的类，并且不继承自 MonoBehaviour，在这个脚本中，使用枚举类型定义一下各个场景，避免调用方法时使用 string 或索引。我们不能使用 SceneManager 调用 LoadScene 方法加载 LoadingScene 之后马上去加载 targetScene，这样我们还是会卡在 targetScene，我们必须等待 LoadingScene 加载出来（即 Update() 开始运行时）再去加载 targetScene，所以这里要使用一个回调函数。在 Scripts 文件夹新建 LoaderCallback.cs，场景中新建空物体命名为 LoaderCallback 并添加脚本，在 LoaderCallback.cs 中只需要在 Update() 的第一帧开始调用这个回调函数即可

```
// Loader.cs中
using UnityEngine.SceneManagement;

public static class Loader
{
    public enum Scene
    {
        MainMenuScene,
        GameScene,
        LoadingScene
    }

    private static Scene targetScene;

    public static void Load(Scene targetScene) {
        Loader.targetScene = targetScene;
        
        SceneManager.LoadScene(Scene.LoadingScene.ToString());
    }
    
    public static void LoaderCallback() {
        SceneManager.LoadScene(targetScene.ToString());
    }
}
```

```
// LoaderCallback.cs中
using UnityEngine;

public class LoaderCallback : MonoBehaviour
{
    private bool isFirstUpdate = true;

    private void Update() {
        if (isFirstUpdate)
        {
            isFirstUpdate = false;
            
            Loader.LoaderCallback();
        }
    }
}
```

最后在之前的 MainMenuUI.cs 中，不直接使用 SceneManager，而是使用 Loader.cs 中的 Load() 方法切换场景，就能达到先切换到加载场景的效果

```
// MainMeneUI.cs
...
public class MainMenuUI : MonoBehaviour
{
    ...
    private void Awake() {
        playButton.onClick.AddListener(() =>
        {
            Loader.Load(Loader.Scene.GameScene);
        });
        ...
    }
}
```

运行游戏，可以正常达到加载场景  

![[c3c1ef123e3ba86dd5a250b2fc58e0b5_MD5.gif]]

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#mainMenuLoading)

## 17.4 暂停游戏与返回主页

在 PlayerInputAction.inputactions 中，添加一个暂停 Action，绑定 Escape 键  

![[0c10432082e2182799ad3c771717a065_MD5.png]]

  
在 GameInput.cs 中，为该按键触发的委托添加会调用的函数，函数中触发了一个 EventHandler 委托，我们在 KitchenGameManager.cs 中为这个委托添加会调用的函数，让委托触发时将 Time.timeScale 在 0f 和 1f 之间切换，Time.timeScale = 0 时，游戏时间会暂停。这里教程中还在 GameInput.cs 中用了单例模式。

```
// GameInput.cs中
...
public class GameInput : MonoBehaviour
{
    public static GameInput Instance { get; private set; }
    ...
    public event EventHandler OnPauseAction;
    ...
    private void Awake()
    {
        Instance = this;
        ...
        playerInputActions.Player.Pause.performed += Pause_performed;
    }
    
    private void OnDestroy()
    {
        ...
        playerInputActions.Player.Pause.performed -= Pause_performed;
        
        playerInputActions.Disable();
    }
    ...
    private void Pause_performed(UnityEngine.InputSystem.InputAction.CallbackContext obj)
    {
        OnPauseAction?.Invoke(this, EventArgs.Empty);
    }
    ...
}
```

```
// KitchenGameManager.cs中
...
public class KitchenGameManager : MonoBehaviour
{
    ...
    public event EventHandler OnGameUnpaused;
    ...
    private void Start()
    {
        GameInput.Instance.OnPauseAction += GameInput_OnPauseAction;
    }
    ...
    private void GameInput_OnPauseAction(object sender, EventArgs e)
    {
        TogglePauseGame();
    }
    ...
    public void TogglePauseGame()
    {
        isGamePaused = !isGamePaused;
        if (isGamePaused)
        {
            Time.timeScale = 0f;
        } else
        {
            Time.timeScale = 1f;
        }
    }
}
```

现在在游戏中按下 esc 键，游戏就会暂停，再次按下，游戏就会恢复，接下来我们来做暂停的 UI  
在 Canvas 下新建空对象命名为 GamePauseUI，stretch；在 GamePauseUI 下右键 ->UI->Image，颜色调黑，透明度低一点，stretch；右键 ->UI->Text，命名 PauseText，调整属性；右键 ->UI->Button 新建两个 button，分别调整属性，一个是恢复暂停用的 Resume 按钮，一个是回到主菜单的 MainMenu 按钮，创建完后效果如下  

![[65643c6f8f25db8cea86208eff3f702b_MD5.png]]

  
在 Scripts/UI 文件夹新建 GamePauseUI.cs，在 KitchenGameManager.cs 中添加两个 EventHandler 委托，分别在游戏进入暂停和恢复时触发，在 GamePauseUI.cs 中进行 UI 的隐藏和显示和按钮触发的函数

```
// KitchenGameManager.cs中
...
public class KitchenGameManager : MonoBehaviour
{
    ...
    public event EventHandler OnGamePaused;
    public event EventHandler OnGameUnpaused;
    ...
    public void TogglePauseGame()
    {
        isGamePaused = !isGamePaused;
        if (isGamePaused)
        {
            ...
            OnGamePaused?.Invoke(this, EventArgs.Empty);
        } else
        {
            ...
            OnGameUnpaused?.Invoke(this, EventArgs.Empty);
        }
    }
}
```

```
// GamePauseUI.cs中
using System;
using UnityEngine;
using UnityEngine.UI;

public class GamePauseUI : MonoBehaviour
{
    [SerializeField] private Button resumeButton;
    [SerializeField] private Button mainMenuButton;
    
    private void Awake()
    {
        resumeButton.onClick.AddListener(() =>
        {
            KitchenGameManager.Instance.TogglePauseGame();
        });
        mainMenuButton.onClick.AddListener(() =>
        {
            Loader.Load(Loader.Scene.MainMenuScene);
        });
    }
    
    private void Start()
    {
        KitchenGameManager.Instance.OnGamePaused += KitchenGameManager_OnGamePaused;
        KitchenGameManager.Instance.OnGameUnpaused += KitchenGameManager_OnGameUnpaused;
        Hide();
    }
    
    private void KitchenGameManager_OnGamePaused(object sender, EventArgs e)
    {
        Show();
    }
    
    private void KitchenGameManager_OnGameUnpaused(object sender, EventArgs e)
    {
        Hide();
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

运行游戏，可以正常恢复暂停或回到主菜单，但是我们遇到了很多问题  
第一个问题：游戏在再次进入到主菜单和进入游戏时时间依然是暂停的，这是因为我们的 Time.timeScale 暂停后就一直是 0，没有恢复，为了恢复我们可以在 MainMenuUI.cs 中将 Time.timeScale 重置回 1

```
// MainMenuUI.cs中
using UnityEngine;
using UnityEngine.UI;

public class MainMenuUI : MonoBehaviour
{
    [SerializeField] private Button playButton;
    [SerializeField] private Button quitButton;

    private void Awake()
    {
        playButton.onClick.AddListener(() =>
        {
            Loader.Load(Loader.Scene.GameScene);
        });
        
        quitButton.onClick.AddListener(() => 
        {
            Application.Quit();
        });
        
        Time.timeScale = 1f;
    }
}
```

第二个问题：一般我们场景中的实例都会在场景销毁后被销毁，但是 PlayerInputActions 类，也就是我们用到的新的 input system 中使用到的一个类的实例不会自动销毁，我们需要手动在场景销毁时将它销毁，OnDestroy() 会在销毁时被调用，可以用它去取消事件的订阅，并且使用 Dispose() 销毁该实例

```
// GameInput.cs中
...
public class GameInput : MonoBehaviour
{
    ...
    private void OnDestroy() {
        playerInputActions.Player.Interact.performed -= Interact_performed;
        playerInputActions.Player.InteractAlternate.performed -= InteractAlternate_performed;
        playerInputActions.Player.Pause.performed -= Pause_performed;
        
        playerInputActions.Dispose();
    }
    ...
}
```

第三个问题：static 修饰的变量或方法不属于某一个实例，它不会被销毁。在 Loader 中使用 static 没有什么问题，但是有些地方使用 static 可能会导致第二次从主菜单进入游戏场景时受到来自上一次游戏的某些影响，这里我们的代码中有三处会收到影响，分别是 CuttingCounter.cs 中的 OnAnyCut、BaseCounter.cs 中的 OnAnyObjectPlacedHere、TrashCounter.cs 中的 OnAnyObjectTrashed，这三个都是我们不希望类每个实例都单独发送发送事件而设置成 static 的 EventHandler 委托，我们只需要在相应的脚本中加上重置的方法，然后在一个新的脚本中调用即可。  
三个类中重置 static 变量的方法，注意其他两个类都继承自 BaseCounter，所以都使用一个函数名可能会报错，我们需要加上 new 关键字

```
// CuttingCounter.cs中
...
public class CuttingCounter : BaseCounter, IHasProgress
{
    ...
    new public static void ResetStaticData()
    {
        OnAnyCut = null;
    }
    ...
}
```

```
// BaseCounter.cs中
...
public class BaseCounter : MonoBehaviour, IHasProgress
{
    ...
    public static void ResetStaticData()
    {
        OnAnyObjectPlacedHere = null;
    }
    ...
}
```

```
// TrashCounter.cs中
...
public class BaseCounter : BaseCounter, IHasProgress
{
    ...
    public static void ResetStaticData()
    {
        OnAnyObjectTrashed = null;
    }
    ...
}
```

新建 ResetStaticDataManager.cs 并在主菜单场景新建空物体命名为 ResetStaticDataManager，添加脚本

```
// ResetStaticDataManager.cs
using UnityEngine;

public class ResetStaticDataManager : MonoBehaviour
{
    private void Awake() {
        CuttingCounter.ResetStaticData();
        BaseCounter.ResetStaticData();
        TrashCounter.ResetStaticData();
    }
}
```

再次运行游戏后在多次进入场景后就不会出现上述问题，目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#pauseClearStatics)

## 17.5 音量设置

接下来来做一个设置界面，我们希望在游戏暂停时有一个 Option 按钮，按下可以显示一些设置选项，我们先做音量设置的选项，两个调整音量的设置选项都有十个级别，每次按下时增大一个等级  
在 GameScene 中的 Canvas 下新建空物体，命名为 OptionsUI，调整属性；在 OptionsUI 下新建 Image 作为背景，Stretch 填充背景、颜色改为黑色降低一点透明度；新建 Text 显示 OPTIONS；新建三个按钮 SoundEffectsButton、MusicButton 和 CloseButton，调整属性，最后效果如下  

![[e50c02f9f29c5e7884d55b8b93c0eb39_MD5.png]]

  
在 Scripts/UI 文件夹下新建 OptionsUI.cs；在 SoundManager.cs（管理音效的）中写改变音量的方法添加到对应对象上，并在处理声音播放的 PlaySound() 方法使用我们调整后的音量大小；新建 MusicManager.cs 添加到对应对象上，和 SoundManager 中差不多，但是是调整 AudioSource 组件的属性来改变的音量；在 OptionUI.cs 中进行订阅

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    ...
    private float volume = 1f;
    ...
    private void PlaySound(AudioClip audioClip, Vector3 position, float volumeMultiplier = 1f) {
        AudioSource.PlayClipAtPoint(audioClip, position, volumeMultiplier * volume);
    }
    ...
    public void ChangeVolume() {
        volume += 0.1f;
        if (volume > 1f)
        {
            volume = 0f;
        }
    }
}
```

```
// MusicManager.cs中
using UnityEngine;

public class MusicManager : MonoBehaviour
{
    public static MusicManager Instance { get; private set; }

    private AudioSource audioSource;
    private float volume = 0.3f;

    private void Awake()
    {
        Instance = this;
        
        audioSource = GetComponent<AudioSource>();
        
        audioSource.volume = volume;
    }
    
    public void ChangeVolume()
    {
        volume += 0.1f;
        if (volume > 1f)
        {
            volume = 0f;
        }
        audioSource.volume = volume;
    }
    
    public float GetVolume()
    {
        return volume;
    }
}
```

```
// OptionsUI.cs
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class OptionsUI : MonoBehaviour
{
    public static OptionsUI Instance { get; private set; }
    
    [SerializeField] private Button soundEffectsButton;
    [SerializeField] private Button musicButton;
    [SerializeField] private Button closeButton;
    [SerializeField] private TextMeshProUGUI soundEffectsText;
    [SerializeField] private TextMeshProUGUI musicText;
    
    private void Awake()
    {
        Instance = this;
        
        soundEffectsButton.onClick.AddListener(() =>
        {
            SoundManager.Instance.ChangeVolume();
            UpdateVisual();
        });
        musicButton.onClick.AddListener(() =>
        {
            MusicManager.Instance.ChangeVolume();
            UpdateVisual();
        });
    }

    private void Start()
    {
        UpdateVisual();
    }
    
    private void UpdateVisual()
    {
        soundEffectsText.text = "Sound Effects: " + Mathf.Round(SoundManager.Instance.GetVolume() * 10f);
        musicText.text = "Music: " + Mathf.Round(MusicManager.Instance.GetVolume() * 10f);
    }
}
```

这时运行游戏，按下暂停键时暂停界面和设置界面会同时出现，可以正常调整音量，接下来我们来写从暂停界面到设置界面的逻辑  
在 GamePauseUI 下添加一个按钮 OptionsButton，调整相应属性，调整完如下  

![[04efabc602a5706889792f6521f41c8a_MD5.png]]

  
在 OptionsUI.cs 中添加显示和隐藏显示隐藏 UI 相关的代码（注意由于在暂停状态下按 esc 可以退出该模式，所以还要订阅事件添加一个按下 esc 隐藏这些 UI 的函数），在 GamePauseUI.cs 中，给上面的 OptionsButton 添加触发的函数

```
// OptionsUI.cs中
...
public class OptionsUI : MonoBehaviour
{
    ...
    private void Awake()
    {
        ...
        closeButton.onClick.AddListener(() =>
        {
            Hide();
        });
    }

    private void Start()
    {
        ...
        Hide();
    }
    
    private void KitchenGameManager_OnGameUnpaused(object sender, System.EventArgs e)
    {
        Hide();
    }
    ...
    public void Show()
    {
        gameObject.SetActive(true);
    }
    
    public void Hide()
    {
        gameObject.SetActive(false);
    }
}
```

```
// GamePauseUI.cs中
...
public class GamePauseUI : MonoBehaviour
{
    ...
    private void Awake() {
        ...
        optionsButton.onClick.AddListener(() =>
        {
            OptionsUI.Instance.Show();
        });
    }
    ...
}
```

这时运行游戏我们可以正常进入和退出设置界面和正常调整音量，但是当我们完全退出游戏再次打开游戏时音量又被恢复为了我们初始化时设置的值，如果我们希望保存这个数据，可以使用 PlayerPrefs.SetFloat() 来保存变量、GetFloat() 来获取变量，这里又要用到字符串，我们不希望使用字符串，所以先将字符串存为一个变量，在 SoundManager.cs 和 MusicManager.cs 中保存变量

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    private const string PLAYER_PREFS_SOUND_EFFECTS_VOLUME = "SoundEffectsVolume";

    private void Awake() {
        ...
        volume = PlayerPrefs.GetFloat(PLAYER_PREFS_SOUND_EFFECTS_VOLUME, 1f);
    }
    ...
    public void ChangeVolume() {
        ...
        PlayerPrefs.SetFloat(PLAYER_PREFS_SOUND_EFFECTS_VOLUME, volume);
        PlayerPrefs.Save();
    }
}
```

```
// MusicMnanager.cs中
...
public class MusicManager : MonoBehaviour
{
    private const string PLAYER_PREFS_MUSIC_VOLUME = "MusicVolume";
    ...
    private void Awake() {
        ...
        volume = PlayerPrefs.GetFloat(PLAYER_PREFS_MUSIC_VOLUME, 0.3f);
        audioSource.volume = volume;
    }
    
    public void ChangeVolume() {
        ...
        PlayerPrefs.SetFloat(PLAYER_PREFS_MUSIC_VOLUME, volume);
        PlayerPrefs.Save();
    }
}
```

这样就可以在每次进入游戏后都加载已经保存的数据了，这里我们游戏单局事件比较短，所以没有保存其他游戏的数据，如果想要将游戏数据保存为 json 格式可以看 [CodeMonkey 的这期视频](https://youtu.be/AmGSEH7QcDg)  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#optionsAudioLevels)

## 17.6 按键绑定设置

在 OptionsUI 对象下新建 Text 和 Button，创建需要绑定的按键的文本和按钮，效果如下  

![[bd96be42c544346efa9f13deccd2d9d8_MD5.png]]

  
在 OptionsUI.cs 中获取按钮与按钮中的文字

```
// OptionsUI.cs中
public class OptionsUI : MonoBehaviour
{ 
    ...
    [SerializeField] private Button moveUpButton;
    [SerializeField] private Button moveDownButton;
    [SerializeField] private Button moveLeftButton;
    [SerializeField] private Button moveRightButton;
    [SerializeField] private Button interactButton;
    [SerializeField] private Button interactAlternateButton;
    [SerializeField] private Button pauseButton;
    [SerializeField] private TextMeshProUGUI soundEffectsText;
    [SerializeField] private TextMeshProUGUI musicText;
    [SerializeField] private TextMeshProUGUI moveUPText;
    [SerializeField] private TextMeshProUGUI moveDownText;
    [SerializeField] private TextMeshProUGUI moveLeftText;
    [SerializeField] private TextMeshProUGUI moveRightText;
    [SerializeField] private TextMeshProUGUI interactText;
    [SerializeField] private TextMeshProUGUI interactAlternateText;
    [SerializeField] private TextMeshProUGUI pauseText;
    ...
}
```

![[92496f6bee378fab22dd15cd1929ff40_MD5.png]]

  
首先将按键上的文字都换成对应的默认绑定的按键，我们不希望直接在 OptionsUI.cs 中直接通过 playerInputActions 去获取绑定的按键，我们希望我们可以通过调用一个方法在不知道用了底层用了什么系统的情况下就获取到了对应按键，因此我们需要在 GameInput.cs 中写一个 GetBindingText() 方法用于获取当前绑定的按键，在 OptionsUI.cs 中的 UpdateVisual() 中更新按钮上的文本

```
// GameInput.cs中
...
public class GameInput : MonoBehaviour
{
    public enum Binding
    {
        Move_Up,
        Move_Down,
        Move_Left,
        Move_Right,
        Interact,
        InteractAlternate,
        Pause
    }
    
    public string GetBindingText(Binding binding) {
        switch (binding)
        {
            default:
            case Binding.Move_Up:
                return playerInputActions.Player.Move.bindings[1].ToDisplayString();
            case Binding.Move_Down:
                return playerInputActions.Player.Move.bindings[2].ToDisplayString();
            case Binding.Move_Left:
                return playerInputActions.Player.Move.bindings[3].ToDisplayString();
            case Binding.Move_Right:
                return playerInputActions.Player.Move.bindings[4].ToDisplayString();
            case Binding.Interact:
                return playerInputActions.Player.Interact.bindings[0].ToDisplayString();
            case Binding.InteractAlternate:
                return playerInputActions.Player.InteractAlternate.bindings[0].ToDisplayString();
            case Binding.Pause:
                return playerInputActions.Player.Pause.bindings[0].ToDisplayString();
        }
    }
}
```

```
// OptionsUI.cs中
...
public class OptionsUI : MonoBehaviour
{ 
    ...
    private void UpdateVisual() {
        ...
        moveUPText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Move_Up);
        moveDownText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Move_Down);
        moveLeftText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Move_Left);
        moveRightText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Move_Right);
        interactText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Interact);
        interactAlternateText.text = GameInput.Instance.GetBindingText(GameInput.Binding.InteractAlternate);
        pauseText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Pause);
    }
}
```

接下来开始处理游戏中的按键绑定，首先先来测试一下，重新绑定上键。在 GameInput.cs 中添加一个进行交互式绑定的方法，在 OptionsUI.cs 中订阅上键按下的事件（这里教程视频中写的代码其实多让 RebindBinding() 方法接收了一个 Binding 类型的参数，这个参数在后面才会用到）

```
// GameInput.cs中
...
public class GameInput : MonoBehaviour
{
    ...
    public void RebindBinding() {
        playerInputActions.Player.Disable();
        playerInputActions.Player.Move.PerformInteractiveRebinding(1)
            .OnComplete(callback =>
            {
                Debug.Log(callback.action.bindings[1].path);
                Debug.Log(callback.action.bindings[1].overridePath);
                callback.Dispose();
                playerInputActions.Player.Enable();
            })
            .Start();
    }
}
```

```
// OptionsUI.cs中
...
public class OptionsUI : MonoBehaviour
{
    ...
    private void Awake() {
        ...
        moveUpButton.onClick.AddListener(() =>
        {
            GameInput.Instance.RebindBinding();
        });
        ...
    }
}
```

这里我还没看作者的讲新的 input system 的视频，找到了文档但还是每太看明白，问了一下 newbing 看懂了，注意这里链式调用，是先执行的 Start() 再执行的 OnComplete()  

![[406ef31173a2661cef3ec21929242d35_MD5.png]]

![[74d9e442469327f71a144203909adce2_MD5.png]]

  
在检查运行游戏确实可以更改上键的绑定，并且可以在控制台打印出更改前后的按键是哪个，接下来我们可以在绑定的过程中增加一个提示画面，让玩家按下一个按键来绑定，同时在绑定后更改原按钮上的文本。  
在 OptionsUI 对象上，新建空物体 PressToRebindingKey，修改属性；在空物体下新建 Image 对象和 Text 对象，调整属性，效果如下  

![[31828d122e7620bda331fd8b6b69e8cd_MD5.png]]

  
在 GameInput.cs 中，我们让 RebindBinding() 方法接收一个 Action 委托，在 OptionsUI.cs 中使用时再套一个函数为委托添加更新按键文本和显示关闭 UI 的方法

```
// GameInput.cs中
...
public class GameInput : MonoBehaviour
{
    ...
    public void RebindBinding(Action onActionRebind) {
        playerInputActions.Player.Disable();
        playerInputActions.Player.Move.PerformInteractiveRebinding(1)
            .OnComplete(callback =>
            {
                callback.Dispose();
                playerInputActions.Player.Enable();
                onActionRebind();
            })
            .Start();
    }
}
```

```
// OptionsUI.cs中
...
public class OptionsUI : MonoBehaviour
{
    ...
    private void Awake() {
        ...
        moveUpButton.onClick.AddListener(() =>
        {
            RebindBinding();
        });
        ...
    }
    
    private void Start() {
        ...
        HidePressToRebindKey();
        ...
    }
    ...
    private void ShowPressToRebindKey() {
        pressToRebindKeyTransform.gameObject.SetActive(true);
    }
    
    private void HidePressToRebindKey() {
        pressToRebindKeyTransform.gameObject.SetActive(false);
    }
    ...
    private void RebindBinding(GameInput.Binding binding) {
        ShowPressToRebindKey();
        GameInput.Instance.RebindBinding(() =>
        {
            HidePressToRebindKey();
            UpdateVisual();
        });
    }
}
```

运行游戏，即可在绑定按键时看到绑定提示和绑定后的变化了，接下来，为所有其他的按键添加绑定的方法，这次我们在 GameInput.cs 的 RebindBinding() 方法中还需要接收一个 Binding 对象，用于确定绑定的具体是哪个按键，由于绑定的按键很多，调用各个函数的步骤基本相同，所以用 switch 语句对不同案件所需的不同变量进行改变，然后再调用完成交互式按键绑定的函数

```
// GameInput.cs中
...
public class GameInput : MonoBehaviour
{
    ...
    public void RebindBinding(Binding binding, Action onActionRebind)
    {
        playerInputActions.Player.Disable();

        InputAction inputAction;
        int bindingIndex;
        
        switch (binding)
        {
            default:
            case Binding.Move_Up:
                inputAction = playerInputActions.Player.Move;
                bindingIndex = 1;
                break;
            case Binding.Move_Down:
                inputAction = playerInputActions.Player.Move;
                bindingIndex = 2;
                break;
            case Binding.Move_Left:
                inputAction = playerInputActions.Player.Move;
                bindingIndex = 3;
                break;
            case Binding.Move_Right:
                inputAction = playerInputActions.Player.Move;
                bindingIndex = 4;
                break;
            case Binding.Interact:
                inputAction = playerInputActions.Player.Interact;
                bindingIndex = 0;
                break;
            case Binding.InteractAlternate:
                inputAction = playerInputActions.Player.InteractAlternate;
                bindingIndex = 0;
                break;
            case Binding.Pause:
                inputAction = playerInputActions.Player.Pause;
                bindingIndex = 0;
                break;
        }
        
        inputAction.PerformInteractiveRebinding(bindingIndex)
            .OnComplete(callback =>
            {
                callback.Dispose();
                playerInputActions.Player.Enable();
                onActionRebind();
            })
            .Start();
    }
}
```

```
// OptionsUI.cs中
...
public class OptionsUI : MonoBehaviour
{
    ...
    private void Awake() {
        ...
        moveUpButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Move_Up); });
        moveDownButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Move_Down); });
        moveLeftButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Move_Left); });
        moveRightButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Move_Right); });
        interactButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Interact); });
        interactAlternateButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.InteractAlternate); });
        pauseButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Pause); });
        ...
    }
    ...
    private void RebindBinding(GameInput.Binding binding) {
        ShowPressToRebindKey();
        GameInput.Instance.RebindBinding(binding, () =>
        {
            HidePressToRebindKey();
            UpdateVisual();
        });
    }
}
```

运行游戏，即可在游戏中绑定所有按键  

![[c457ee0c90b77c963b1b765a143f2a85_MD5.gif]]

  
最后，我们可以在 GameInput.cs 中使用新的 input system 中的 SaveBindingOverridesAsJson() 方法来保存设置，然后使用 Playerprefs.setString() 将设置保存为字符串，注意在 Awake() 中读取设置时需要在 PlayerInputActions 实例化之后，playerInputActions.Player.Enable() 之前

```
// GameInput.cs中
...
public class GameInput : MonoBehaviour
{
    private const string PLAYER_PREFS_BINDINGS = "InputBindings";
    ...
    private void Awake() {
        ...
        if (PlayerPrefs.HasKey(PLAYER_PREFS_BINDINGS))
        {
            playerInputActions.LoadBindingOverridesFromJson(PlayerPrefs.GetString(PLAYER_PREFS_BINDINGS));
        }
        ...
    }
    ...
    public void RebindBinding(Binding binding, Action onActionRebind) {
        ...
        inputAction.PerformInteractiveRebinding(bindingIndex)
            .OnComplete(callback =>
            {
                ...
                PlayerPrefs.SetString(PLAYER_PREFS_BINDINGS, playerInputActions.SaveBindingOverridesAsJson());
            })
            .Start();
    }
}
```

运行游戏，即可在完全退出游戏后仍然保存之前的按键设置，目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#optionsKeyRebinding)

## 17.7 手柄输入与手柄菜单导航

打开 PlayerInputActions.inputactions，首先在用新的 input system 重构角色移动的代码时作者已经说过，Move 只要再绑定一个 Left Stick [Gamepad] 即可完成通用的手柄左摇杆的绑定，但是这样做没有处理 deadzone，轻轻推一下摇杆角色就会移动我们需要在 Processors 中添加一个 Stick Deadzone 调整参数；然后在 Interact 添加一个 Binding，绑定 Button South[Gamepad]；InteractAlternate 绑定 Button West[Gamepad]；Pause 绑定 Start[Gamepad]  

![[3cda7bb4b47b5f71cec6536db5659ae6_MD5.png]]

  
现在运行游戏可以发现一个问题，当我们朝向柜子移动时，总会不自觉地向左向右转，这是因为我们的键盘只有四个方向键，我们为了让顶着柜子时按左右移动顺畅加了一些逻辑，但是手柄的摇杆很难走正前正后方向，稍微偏一些就会被识别为其他方向的移动，这里为了解决这个问题在 Player.cs 中更改了之前写的`moveDir.x != 0`和`moveDir.z != 0`

```
// Player.cs中
...
if (!canMove)
{
	...
	// canMove = moveDir.x != 0 && !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirX, moveDistance);
	(moveDir.x < -0.5f || moveDir.x > 0.5f) && !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirX, moveDistance);
	...
	if (canMove)
	{
		 ...
	} else
	{
		 ...
		 // canMove = moveDir.z != 0 && !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirZ, moveDistance);
		 (moveDir.z < -0.5f || moveDir.z > 0.5f) && !Physics.CapsuleCast(transform.position, transform.position + Vector3.up * playerHeight, playerRadius, moveDirZ, moveDistance);
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

接下来让我们支持在游戏中绑定手柄按键，在 GameInput.cs 中补上相应的枚举类型以及相应的方法

```
// GameInput.cs
...
public class GameInput : MonoBehaviour
{
    ...
    public enum Binding
    {
        ...
        Gamepad_Interact,
        Gamepad_InteractAlternate,
        Gamepad_Pause
    }
    ...
    public string GetBindingText(Binding binding) {
        switch (binding)
        {
            default:
            ...
            case Binding.Gamepad_Interact:
                return playerInputActions.Player.Interact.bindings[1].ToDisplayString();
            case Binding.Gamepad_InteractAlternate:
                return playerInputActions.Player.InteractAlternate.bindings[1].ToDisplayString();
            case Binding.Gamepad_Pause:
                return playerInputActions.Player.Pause.bindings[1].ToDisplayString();
        }
    }

    public void RebindBinding(Binding binding, Action onActionRebind) {
        ...
        switch (binding)
        {
            ...
            case Binding.Gamepad_Interact:
                inputAction = playerInputActions.Player.Interact;
                bindingIndex = 1;
                break;
            case Binding.Gamepad_InteractAlternate:
                inputAction = playerInputActions.Player.InteractAlternate;
                bindingIndex = 1;
                break;
            case Binding.Gamepad_Pause:
                inputAction = playerInputActions.Player.Pause;
                bindingIndex = 1;
                break;
        }
        ...
    }
}
```

在选项界面加上对应的 UI  

![[5c46c7d1fff16f6dfef5cbeb0d2c3d5f_MD5.png]]

  
在 OptionsUI.cs 中添加相应逻辑

```
// OptionsUI.cs中
...
public class OptionsUI : MonoBehaviour
{
    ...
    [SerializeField] private TextMeshProUGUI gamepadInteractText;
    [SerializeField] private TextMeshProUGUI gamepadInteractAlternateText;
    [SerializeField] private TextMeshProUGUI gamepadPauseText;
    [SerializeField] private Transform pressToRebindKeyTransform;
    
    private void Awake()
    {
        ...
        gamepadInteractButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Gamepad_Interact); });
        gamepadInteractAlternateButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Gamepad_InteractAlternate); });
        gamepadPauseButton.onClick.AddListener(() => { RebindBinding(GameInput.Binding.Gamepad_Pause); });
    }
    ...
    private void UpdateVisual()
    {
        ...
        gamepadInteractText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Gamepad_Interact);
        gamepadInteractAlternateText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Gamepad_InteractAlternate);
        gamepadPauseText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Gamepad_Pause);
    }
    ...
}
```

现在我们可以用手柄进入菜单、选项界面，但是在这些界面时只能使用鼠标操作，我们希望使用手柄也可以上下移动看到当前选择的是哪个选项然后按 A 确认  
首先在场景中的 EventSystem 中，点击 Replace with InputSystemUIInputModule  

![[984ed1b27389a7d8309edd89fa9af7e4_MD5.png]]

  
将所有按钮的 SelectedColor 改为一个显眼的颜色  

![[462e10fc4580c99874760b7b712d16fc_MD5.png]]

  
这时候运行游戏，如果我们先用鼠标按下一个按键然后在按键外松开，就可以发现这个按键带上了了 Selected Color，这时就能用键盘上下键或者手柄去控制了，因此我们只要确保暂停或其他界面开启时有一个按钮已经是被选择状态了即可

```
// GamePauseUI.cs中
...
public class GamePauseUI : MonoBehaviour
{
    ...
    private void Show() {
        ...
        resumeButton.Select();
    }
}
```

```
// OptionsUI.cs中
...
public class OptionsUI : MonoBehaviour
{
    ...
    public void Show() {
        ...
        soundEffectsButton.Select();
    }
    ...
}
```

运行游戏，进入暂停和选项界面时都有按钮可以选，但是在设置里的按钮时会选到后面的暂停界面的选项  

![[6e828e307e44b18c26e134174573e118_MD5.gif]]

  
这是因为我们的选项和暂停其实都在一个界面上，全都是激活状态，unity 会在这个界面上自动寻找按钮然后自动导航，选择一个 Button，在 Button 组件的 Navigation 处有一个 Visualize，点击即可可视化地显示 unity 自动生成的导航  

![[93f63a12299c3b90c8b333b9d957f6ff_MD5.png]]

  
一个解决方案是，在一些导航不正确的 Button 上，我们可以将 Navigation 处的 Automatic 改为 Explicit 手动设置上下左右会导航到哪个按钮上；第二个解决方案是，我们在开启设置页面时隐藏掉暂停页面相关的对象  
在 OptionsUI.cs 中，给 Show() 方法添加一个参数，这个参数是一个 Action 委托，该 Action 委托在设置界面关闭时被调用，我们希望调用 GamePauseUI.cs 中的 Show() 方法显示暂停界面，同时在 GamePauseUI.cs 中让玩家在暂停界面按下设置按钮显示设置界面时隐藏暂停界面

```
// OptionsUI.cs中
...
public class OptionsUI : MonoBehaviour
{
    ...
    private Action onCloseButtonAction;
    
    private void Awake() {
        ...
        closeButton.onClick.AddListener(() =>
        {
            ...
            onCloseButtonAction();
        });
        ...
    }
    ...
    public void Show(Action onCloseButtonAction) {
        this.onCloseButtonAction = onCloseButtonAction;
        ...
    }
    ...
}
```

```
// GameInput.cs中
...
public class GamePauseUI : MonoBehaviour
{
    ...
    private void Awake() {
        ...
        optionsButton.onClick.AddListener(() =>
        {
            ...
            OptionsUI.Instance.Show(Show);
        });
    }
}
```

再次运行游戏，即可正常导航  

![[adb3bcdf2a85a264c7d6aadf5a472a7d_MD5.gif]]

  
另外主菜单还有两个按键，去 MainMenuScene.unity，这里我们只有两个按钮，所以可以直接把 PlayButton 拖到 EventSystem 的 First Selected 上即可  

![[43608fb949ee329ae1a0a134d6a37529_MD5.png]]

  
目前为止的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#controllerInputMenuNavigation)

## 18 打磨细节 Polish

## 18.1 给游戏场景添加墙壁

在场景中添加一些 Cube，一些做成墙壁，添加上_Assets/Materials 中的 Wall.mat 材质，一些做成外围的黑色，添加上 Black.mat 材质  

![[a7b6f6431d8093a16665c6efcca3a7b0_MD5.png]]

  
运行游戏，即可看到效果  

![[dfd432f21810ff3b254c04d9ce8f016d_MD5.png]]

## 18.2 角色移动粒子特效

在_Assets/PrefabsVisuals 中已经做好了一个 PlayerMovingParticles.prefab，从 Inspector 面板可以看到，这个效果主要在 Particle System 中调整了 Emission 这个属性，将 Rate over Time 调为 0，Rate over Distance 调为了 4，另外将 Simulation Space 调为了 World，让该粒子效果能随着位置的变化而发射粒子  

![[d047a8388632a0e3d3afa88d411392dd_MD5.png]]

  
将该物体拖入场景中，手动移动可以即可看到效果  

![[dcc1cb0cdbfa230c92cff73a893a6270_MD5.gif]]

  
将该对象拖到 Player 下作为子物品，运行游戏，当角色移动时即可看到效果  

![[15450adcd61f86a07db2b150b67ac2bf_MD5.gif]]

## 18.3 教学页面

在 Canvas 下新建空物体命名为 TutorialUI；在下面新建 Image 命名为 BackGround，给一个有点透明度的白色背景 ；再来一个 Image，调整大小 Source Image 选择一个叫 Tutorial 的图片；这张图片上没有按钮的图片，我们自己新建 Image 命名为各个键的名字，然后下面新建背景 Source Image 选 Circle，新建 Text，复制出来把名字都改为对应的按键；我们希望这个页面上的按键是跟随我们在游戏中的按键绑定的，所以再 Scripts/UI 新建 TutorialUI.cs，获取对应文字对象  

![[eb77f9b110c45bd6fe3a660828943c22_MD5.jpg]]

  
在 GameInput.cs 中添加一个 EventHandler 委托，在游戏中绑定按键后触发；在 TutorialUI.cs 中，为这个委托添加函数来更新文字

```
// GameInput.cs中
...
public class GameInput : MonoBehaviour
{
    ...
    public event EventHandler OnBindingRebind;
    ...
    public void RebindBinding(Binding binding, Action onActionRebind)
    {
        ...
        inputAction.PerformInteractiveRebinding(bindingIndex)
            .OnComplete(callback =>
            {
                OnBindingRebind?.Invoke(this, EventArgs.Empty);
            })
            .Start();
    }
}
```

```
// TutorialUI.cs中
using System;
using UnityEngine;
using TMPro;

public class TutorialUI : MonoBehaviour
{
    [SerializeField] private TextMeshProUGUI keyMoveUpText;
    [SerializeField] private TextMeshProUGUI keyMoveDownText;
    [SerializeField] private TextMeshProUGUI keyMoveLeftText;
    [SerializeField] private TextMeshProUGUI keyMoveRightText;
    [SerializeField] private TextMeshProUGUI keyInteractText;
    [SerializeField] private TextMeshProUGUI keyInteractAlternateText;
    [SerializeField] private TextMeshProUGUI keyPauseText;
    [SerializeField] private TextMeshProUGUI keyGamepadInteractText;
    [SerializeField] private TextMeshProUGUI keyGamepadInteractAlternateText;
    [SerializeField] private TextMeshProUGUI keyGamepadPauseText;

    private void Start()
    {
        GameInput.Instance.OnBindingRebind += GameInput_OnBindingRebind;
        
        UpdateVisual();
    }
    
    private void GameInput_OnBindingRebind(object sender, EventArgs e)
    {
        UpdateVisual();
    }
    
    private void UpdateVisual()
    {
        keyMoveUpText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Move_Up);
        keyMoveDownText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Move_Down);
        keyMoveLeftText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Move_Left);
        keyMoveRightText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Move_Right);
        keyInteractText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Interact);
        keyInteractAlternateText.text = GameInput.Instance.GetBindingText(GameInput.Binding.InteractAlternate);
        keyPauseText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Pause);
        keyGamepadInteractText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Gamepad_Interact);
        keyGamepadInteractAlternateText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Gamepad_InteractAlternate);
        keyGamepadPauseText.text = GameInput.Instance.GetBindingText(GameInput.Binding.Gamepad_Pause);
    }
}
```

接下来处理该页面的显示与隐藏，之前我们在 KitchenGameManager.cs 中设置了游戏开始倒计时前有 1 秒，这里我们不再使用这个一秒的计时器，而是什么时候按下交互键什么时候开始进入倒计时状态  
在 KitchenGameManager.cs 中，不再使用之前的 1 秒的计时器，为在 GameInput 中按下交互键触发的委托添加调用函数，将游戏状态由 WaitingToStart 变为 ountdownToStart，并触发 OnStateChanged 的委托；在 TutorialUI.cs 中，为该委托添加调用函数，处理教学页面的隐藏和显示

```
// KitchenGameManager.cs中
public class KitchenGameManager : MonoBehaviour
{
    ...
    // private float waitingToStartTimer = 1f;
    ...
    private void Start()
    {
        ...
        GameInput.Instance.OnInteractAction += GameInput_OnInteractAction;
    }
    ...
    private void GameInput_OnInteractAction(object sender, EventArgs e)
    {
        if (state == State.WaitingToStart)
        {
            state = State.CountdownToStart;
            OnStateChanged?.Invoke(this, EventArgs.Empty);
        }
    }
    ...
}
```

```
// TutorialUI.cs中
...
public class TutorialUI : MonoBehaviour
{
    ...
    private void Start()
    {
        ...
        KitchenGameManager.Instance.OnStateChanged += Instance_OnStateChanged;
        ...
    }
    ...
    private void Instance_OnStateChanged(object sender, EventArgs e)
    {
        if (KitchenGameManager.Instance.IsCountdownToStartActive())
        {
            Hide();
        }
    }
    ...
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

## 18.4 倒计时文本动画

给场景中的 GameStartCountdownUI 添加一个 Animator 组件，在_Assets/Animations 中右键 ->Create->Animator Controller，新建 CountdownUI.controller，拖动到组件对应位置；打开 Animation 窗口，点击 Create 新建 CountdownUI_NumberPopup.animation；给 GameStartCountdownUI 添加一个 CanvasGroup 组件，这个组件可以方便地调整 Alpha 值；在 Animation 窗口中打关键帧 k 一个时常为 1 秒地动画，效果如下  

![[899faa6a114bb5bf105c1ee30c4f89a2_MD5.gif]]

  
在 Animator 窗口中，Any State 右键 ->Make Transition 到 CountdownUI_NumberPopup，过渡时间 Transition Duration 改为 0，在 Parameter 新建一个 Trigger NumberPopup，在这个 Transition 中添加 Conditions 为 NumberPopup  

![[2d60a7705c9e2a68a74833032fe9b19b_MD5.png]]

  
接下来我们要添加相应的逻辑，对于 UI 的视觉和逻辑的代码我们不一定要分开，因为他们经常是紧密相关的，所以这里我们在 GameStartCountdownUI.cs 中写对应的逻辑，同时添加声音，为此还要在 SoundManager.cs 中添加一个播放倒计时声音的方法  
在 GameStartCountdownUI.cs 中在 Update() 中通过对比当前显示的 countdownNumber 和上一帧保存的 previousCountdownNumber 是否相同来判断是否触发动画中的 trigger 和播放音效

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    ...
    public void PlayCountdownSound() {
        PlaySound(audioClipRefsSO.warning, Vector3.zero);
    }
    ...
}
```

```
// GameStartCountDownUI.cs中
...
public class GameStartCountDownUI : MonoBehaviour
{
    private const string NUMBER_POPUP = "NumberPopup";
    ...
    private Animator animator;
    private int previousCountdownNumber;
    ...
    private void Awake() {
        animator = GetComponent<Animator>();
    }
    ...
    private void Update() {
        int countdownNumber = Mathf.CeilToInt(KitchenGameManager.Instance.GetCountdownToStartTimer());
        countdownText.text = countdownNumber.ToString();

        if (previousCountdownNumber != countdownNumber)
        {
            previousCountdownNumber = countdownNumber;
            animator.SetTrigger(NUMBER_POPUP);
            SoundManager.Instance.PlayCountdownSound();
        }
    }
}
```

运行游戏，游戏倒计时动画以及声音正常播放  

![[531837dc613f5a1111ee14b010d12173_MD5.webp]]

## 18.5 炉灶台提示 UI 与音效

首先先实现在炉灶台在煎肉时第二个进度条走到一般出现警告标志的逻辑  
在 StoveCounter.prefab 中新建 Canvas 命名为 StoveBurnWarningUI，调整 Render Mode 为 World Space，添加之前做的 Look At Camera 组件，移动到进度条上面的位置；调整该物体下面新建一个 Image，Image Source 选择 Warning，效果如下  

![[d5a758bd044901994e5a14baa8312b88_MD5.png]]

  
在 Scripts/UI 文件夹新建 StoveBurnWarningUI.cs，在 StoveCounter.cs 中添加一个方法判断当前是否处于 Fried 状态，在 StoveBurnWarningUI.cs 中写隐藏显示该警告标志的逻辑

```
// StoveCounter.cs中
...
public class StoveCounter : BaseCounter, IHasProgress
{
    ...
    public bool IsFried() {
        return state == State.Fried;
    }
}
```

```
// StoveBurnWarningUI.cs中
using UnityEngine;

public class StoveBurnWarningUI : MonoBehaviour
{
    [SerializeField] private StoveCounter stoveCounter;

    private void Start()
    {
        stoveCounter.OnProgressChanged += StoveCounter_OnProgressChanged;
        
        Hide();
    }
    
    private void StoveCounter_OnProgressChanged(object sender, IHasProgress.OnProgressChangedEventArgs e)
    {
        float burnShowProgressAmount = 0.5f;
        bool show = stoveCounter.IsFried() && e.progressNormalized >= burnShowProgressAmount;

        if (show)
        {
            Show();
        } else
        {
            Hide();
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

现在运行游戏即可看到进度条第二条走到一半时标志出现，现在添加标志出现时进度条同时开始闪烁的动画  
给 StoveBurnWarningUI 添加 Animator 组件和 Canvas Grounp 组件；在_Assets/Animations 创建 StoveBurnWarningUI，拖到组件处；在 Animator 面板点击 Create 创建 StoveBurnWarningUI_Flash.anim，给 Canvas Group 组件上的 Alpha 值打关键帧  

![[34d9651ec928618bd91f6ea8f7a7a12a_MD5.gif]]

  
我们只需要让图标在显示的时候一直播放该动画，所以不需要再在 Animator 中去调整。我们还有一个警告的音效，在 SoundManager.cs 中，添加一个 PlayWarningSound() 方法，在 StoveCounterSound.cs 中判断状态每隔 0.2 秒播放一次

```
// SoundManager.cs中
...
public class SoundManager : MonoBehaviour
{
    ...
    public void PlayCountdownSound() {
        PlaySound(audioClipRefsSO.warning, Vector3.zero);
    }
    ...
}
```

```
// StoveCounterSound.cs中
...
public class StoveCounterSound : MonoBehaviour
{
    ...
    private bool playWarningSound;
    ...
    private void Start()
    {
        ...
        stoveCounter.OnProgressChanged += StoveCounter_OnProgressChanged;
    }
    ...
    private void StoveCounter_OnProgressChanged(object sender, IHasProgress.OnProgressChangedEventArgs e)
    {
        float burnShowProgressAmount = 0.5f; 
        playWarningSound = stoveCounter.IsFried() && e.progressNormalized >= burnShowProgressAmount;
    }
    ...
    private void Update()
    {
        if (playWarningSound)
        {
            warningSoundTimer -= Time.deltaTime;
            if (warningSoundTimer <= 0f)
            {
                float warningSoundMax = 0.2f;
                warningSoundTimer = warningSoundMax;
                
                SoundManager.Instance.PlayWarningSound(stoveCounter.transform.position);
            }   
        }
    }
}
```

接下来为进度条添加红黄色的闪烁动画  
给 ProgressBarUI 添加 Animator 组件；在_Assets/Animations 创建 StoveBurnFlashBar，拖到组件处；在 Animator 面板点击 Create 创建 StoveBurnFlashBar_Idle.anim，k 一个和之前颜色一样的帧，复制一个 anim 文件重命名为 StoveBurnFlashBar_Flash，拖到 animator 窗口中，k 颜色黄红闪烁的动画，效果如下  

![[5bf498936a0816d20ce25ffce9c428c7_MD5.gif]]

  
在 Animator 窗口，让 Idle 和 Flashing 间互相可以 Transition，添加一个 IsFlashing 参数，设置两个 Transition 加上 conditions  

![[1a62521196ea81e28ec9a4ee3efb6f92_MD5.png]]

  
在 Scripts/UI 文件夹新建 StoveBurnFlashingBarUI.cs，判断当前是否处于第二条进度条的一半时间之后然后设置 Animator 中设置的布尔值

```
// StoveBurnFlashingBarUI.cs中
using UnityEngine;

public class StoveBurnFlashingBarUI : MonoBehaviour
{
    private const string IS_FLASHING = "IsFlashing";
    [SerializeField] private StoveCounter stoveCounter;

    private Animator animator;

    private void Awake()
    {
        animator = GetComponent<Animator>();
    }

    private void Start()
    {
        stoveCounter.OnProgressChanged += StoveCounter_OnProgressChanged;
        
        animator.SetBool(IS_FLASHING, false);
    }
    
    private void StoveCounter_OnProgressChanged(object sender, IHasProgress.OnProgressChangedEventArgs e)
    {
        float burnShowProgressAmount = 0.5f;
        bool show = stoveCounter.IsFried() && e.progressNormalized >= burnShowProgressAmount;

        animator.SetBool(IS_FLASHING, show);
    }
}
```

运行游戏，可以播放闪烁动画  

![[90a57b9214a50517c922aeddcb18834b_MD5.gif]]

## 18.6 送餐提示 UI

在 DeliveryCounter.orefab 中新建 Canvas 命名为 DeliveryResultUI，Render Mode 改为 World Space，将位置放到里相机较近的位置；在 DeliveryResultUI 下新建 Image 命名为 Background，调整属性；新建 Text 命名为 MessageText，调整属性，新建 Image 命名为 IconImage，调整属性，最后效果如下  

![[83115eaa86521cb1478f5c9a151a84c7_MD5.png]]

  
我们想要做和倒计时一样的动画，但是我们又想让它最后加上 Look At Camera 组件避免左右颠倒，这个动画与该组件有冲突，所以我们这里新创建一个空物体命名为 DeliveryResultUI_LookAtCamera，让该空物体和 DeliveryResultUI 在同一位置，然后拖动 DeliveryResultUI 作为这个空物体的子级，将位移信息改为 0，现在我们可以旋转 DeliveryResultUI_LookAtCamera 来做旋转动画了  

![[92c00e34771fd578cd2d39ccfe299614_MD5.png]]

  
在 DeliveryResultUI 上添加 Canvas Grounp 组件用于调整透明度，添加 Animator 组件在_Assets/Animations 文件夹创建 DeliveryResultUI.controller，在 Animation 面板点击 Create 新建 DeliveryResultUI_Popup.anim，k 缩放旋转和透明度，效果如下  

![[7b929a51996178fa841681879e545a7e_MD5.gif]]

  
在 Animator 面板从 Any State 右键 ->Make Transition 指向 Popup 动画，设置属性，添加一个 trigger 叫 Popup，Transition 的 Condisions 设置为 Popup，默认该动画是循环播放的，我们需要找到. anim 文件然后取消勾选 Loop Time  

![[823480f3c701640c9045369909ba8b0f_MD5.png]]

  
在 Scripts/UI 文件夹新建 DeliveryResultUI.cs，写隐藏显示 UI 与播放动画的代码

```
// DeliveryResultUI.cs中
using TMPro;
using UnityEngine;
using UnityEngine.UI;

public class DeliveryResultUI : MonoBehaviour
{
    private const string POPUP = "Popup";
    
    [SerializeField] private Image backgroundImage;
    [SerializeField] private Image iconImage;
    [SerializeField] private TextMeshProUGUI messageText;
    [SerializeField] private Color successColor;
    [SerializeField] private Color failedColor;
    [SerializeField] private Sprite successSprite;
    [SerializeField] private Sprite failedSprite;

    private Animator animator;
    
    private void Awake()
    {
        animator = GetComponent<Animator>();
    }
    
    private void Start()
    {
        DeliveryManager.Instance.OnRecipeSuccess += DeliveryManager_OnRecipeSuccess;
        DeliveryManager.Instance.OnRecipeFailed += DeliveryManager_OnRecipeFailed;

        gameObject.SetActive(false);
    }
    
    private void DeliveryManager_OnRecipeSuccess(object sender, System.EventArgs e)
    {
        gameObject.SetActive(true);
        animator.SetTrigger(POPUP);
        backgroundImage.color = successColor;
        iconImage.sprite = successSprite;
        messageText.text = "DELIVERY\nSUCCESS";
    }
    
    private void DeliveryManager_OnRecipeFailed(object sender, System.EventArgs e)
    {
        gameObject.SetActive(true);
        animator.SetTrigger(POPUP);
        backgroundImage.color = failedColor;
        iconImage.sprite = failedSprite;
        messageText.text = "DELIVERY\nFAILED";
    }
}
```

在组件处设置好相应参数  

![[85a23a198bdd6aa6147334fd61da7e25_MD5.png]]

  
运行游戏，即可看到正确弹出的送餐提示  

![[7b84901200d6d8f78a4af00460917efc_MD5.gif]]

  
最终的[工程文件](https://unitycodemonkey.com/kitchenchaoscourse.php#polish)