# 环境搭建  

新建空项目，平台改成`` `x64` ``。 ![](/api/filetransfer/images?url=https%3A%2F%2Fddddyx.oss-cn-beijing.aliyuncs.com%2Fimg%2F202208201233252.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_15%252Ctext_dGhldHVz%252Ccolor_FFFFFF%252Cshadow_50%252Ct_80%252Cg_se%252Cx_10%252Cy_10&sign=c66511839f00b3ff26349ac042426749d5bf78d93457c18f1894d1fff7232702) 把标准改到C++17和C17。 ![](/api/filetransfer/images?url=https%3A%2F%2Fddddyx.oss-cn-beijing.aliyuncs.com%2Fimg%2F202208201233254.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_34%252Ctext_dGhldHVz%252Ccolor_FFFFFF%252Cshadow_50%252Ct_80%252Cg_se%252Cx_10%252Cy_10&sign=355f27c475cf0178cb0cd2961a9823212dba509f892ba1730092f720344319c7)  

在`` `VC++ Directors → Include Directors` ``中，添加随书资源的`` `Common` ``文件夹，方便我们直接inlcude。 ![](/api/filetransfer/images?url=https%3A%2F%2Fddddyx.oss-cn-beijing.aliyuncs.com%2Fimg%2F202208201233255.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_34%252Ctext_dGhldHVz%252Ccolor_FFFFFF%252Cshadow_50%252Ct_80%252Cg_se%252Cx_10%252Cy_10&sign=cace4538da8b683aadf4b7f6f397b5c15c6479c244e3ebef99b2a152ebe4b8b9) 在`` `Linker → Input → Additional Dependencies` ``中，填上关联的库，而不是龙书中`` `#pragma comment("xxx.lib")` ``的方式，这样有助于维护。 ![](/api/filetransfer/images?url=https%3A%2F%2Fddddyx.oss-cn-beijing.aliyuncs.com%2Fimg%2F202208201233256.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_34%252Ctext_dGhldHVz%252Ccolor_FFFFFF%252Cshadow_50%252Ct_80%252Cg_se%252Cx_10%252Cy_10&sign=f8821523f6cacb3e52a5387467f9375aac7046cce77ce1fa03d9f8daefbffa28) 最后把`` `Linker → System` ``改成`` `Windows` ``。  

现在直接运行书中代码会出现错误：`` `'&' requires l-value` `` 。 这是因为龙书喜欢把拿取函数返回值的指针的操作，类似： `` `SetValue(&GetValue());` `` 虽然函数返回值会在分号后才销毁，但编译器已经舍弃了对这种语法糖的支持。 有三种解决方法：  

*   编造语法糖，用新的语法糖代替`` `&` ``符号，详细可以看麦老师的专栏。  
    

*   使用变量，即`` `aotu t = GetValue; SetValue(&t);` ``。  
    

*   关闭符合模式：`` `Configuration Properties -> C/C++ -> Language -> Conformance mode` ``改为`` `No` ``。  
    

我用的第三种方法。  

# 创建Win32窗口  

凡是 Windows应用程序就要依从事件驱动编程模型( event-driven programming model )。一般来讲，应用程序总会“坐等”"某事的发生，即事件(event)的发生。生成事件的方式多种多样，常见的例子有键盘按键、点击鼠标,或者是窗口的创建、调整大小、移动、关闭、最小化、最大化乃至“隐身( visible,即窗口变为不可见的状态)”。当事件发生时，Windows会向发生事件的应用程序发送相应的消息(message)，随后，该消息会被添加至此应用程序的消息队列(message queue，简言之，这是一种为应用程序存储消息的优先级队列)之中。应用程序会在消息循环(message loop)中不断地检测队列中的消息，在接收到消息之后，它会将此消息分派到相应窗口的窗口过程(window procedure)。(一个应用程序可能附有若干个窗口)。每个窗口都有一个与之关联的名为窗口过程的函数。我们实现的窗口过程函数中写有处理特定消息的代码。 简而言之，用户或应用程序的某些行为会产生事件。操作系统会为响应此事件的应用程序发送相关的消息。随后，该消息会被添加到目标应用程序的消息队列之中。由于应用程序会不断地检测队列中的消息，在接收到消息后，应用程序就会将它分派到对应窗口的窗口过程。最后，窗口过程会针对此消息执行相应的系列指令。 下面这张图概括了事件驱动编程的模型。目前，我们只用关心一个窗口，所以只需要注意这棵树的中间即可。 ![](/api/filetransfer/images?url=https%3A%2F%2Fddddyx.oss-cn-beijing.aliyuncs.com%2Fimg%2F202208201233257.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_29%252Ctext_dGhldHVz%252Ccolor_FFFFFF%252Cshadow_50%252Ct_80%252Cg_se%252Cx_10%252Cy_10&sign=bb8234b739a78abca0b9570cff0973cf127b35c9a6608b7e541be600facadae0)  

在这之前，我建议先把附录A的给出的程序完成的打(抄)一遍明白大概流程。  

# 初始化D3D  

D3D的初始化过程可以分为以下几个步骤：  

*   开启D3D12调试层。  
    

*   创建设备。  
    

*   创建围栏，同步CPU和GPU。  
    

*   获取描述符大小。  
    

*   设置MSAA抗锯齿属性。  
    

*   创建命令队列、命令列表、命令分配器。  
    

*   创建交换链。  
    

*   创建描述符堆。  
    

*   创建描述符。  
    

*   资源转换。  
    

*   设置视口和裁剪矩形。  
    

*   设置围栏刷新命令队列。  
    

*   将命令从列表传至队列。  
    

首先我们打开D3D调试层，在`` `WinMain` ``中添加：  

```
#if defined(DEBUG) | defined(_DEBUG)
	_CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);
#endif

``` 

下面我们将各个模块封装成函数。  

## 创建设备  

设备代表着显示适配器，一般来说显示适配器是一种3D图形硬件（如显卡）。但是，一个系统也能用软件显示适配器来模拟3D图形硬件的功能（如WARP适配器）。  

```
// 创建设备
ComPtr dxgiFactory;
ComPtr d3dDevice;
void CreateDevice() {
	ThrowIfFailed(CreateDXGIFactory1(IID_PPV_ARGS(&dxgiFactory)));
	ThrowIfFailed(D3D12CreateDevice(nullptr, // 此参数如果设置为nullptr，则使用主适配器
		D3D_FEATURE_LEVEL_12_0,		// 应用程序需要硬件所支持的最低功能级别
		IID_PPV_ARGS(&d3dDevice)));	// 返回所建设备
} 
``` 

DX中很重要的一个概念是COM接口，它可以提高ABI兼容性。它像是一个智能指针有引用计数，可以指定各种类型，且可以跨语言。COM接口解释建议看麦老师专栏。  

```
ComPtr < A > a;	//定义一个ComPtr智能指针
&a  //A** 类型，并增加引用，写入用
a.Get();  //得到A*
a.GetAddressOf();  //得到A**，只读，不改引用
a.GetAddressOfAndRelease();  //得到A**并减少引用
a.Reset();  //释放与之相关的所有引用
IID_PPV_ARGS(&a)// 获取a的COM ID

``` 

另一个概念是DXGI API，D3D12将设备对象概念进行了扩展显示器，适配器，3D设备等对象进行了分离，而分离的标志就是使用IDXGIFactory来代表整个图形子系统，它主要的功用之一就是让我们创建适配器、3D设备等对象接口用的，因此它的名字就多了个Factory，这估计也是暗指Factory设计模式之故。接口和函数名后面的数字可以理解它们为对应接口或函数的版本号，默认为0，也就是第一个原始版本，不用写出来，2就表示升级的第三个版本，依此类推。  

```
ComPtr dxgiFactory;
ThrowIfFailed(CreateDXGIFactory1(IID_PPV_ARGS(&dxgiFactory))); 
``` 

有了IDXGIFactory接口，我们就可以创建D3D12设备了：  

*   `` `pAdpter` ``：指定创建设备时所用的显示适配器。空指针代表主显示适配器。  
    

*   `` `MinimunFeatureLevel` ``：硬件所支持的最低功能级别，如果适配器不支持此功能级别，则设备创建失败。  
    

*   `` `riid` ``：所建`` `ID3D12Device` ``接口的COM ID。  
    

*   `` `ppDevice` ``：返回所创建的Direct3D 12设备。  
    

## 创建围栏并获取描述符大小  

我们围栏实现刷新命令队列，强制CPU等待，直到GPU完成所有命令的处理。  

创建一个围栏对象的方法原型如下：  

*   `` `InitialValue` ``：围栏的初始值。每当需要标记一个新的围栏点时就将它加1。  
    

*   `` `D3D12_FENCE_FALGS` ``：用按位或运算指定`` `D3D11_FENCE_FLAG` ``，可以设置围栏是否被共享等属性。`` `D3D11_FENCE_FLAG_NONE` ``表示没有指定选项。  
    

*   `` `rrid` ``：所建`` `ID3D12Fence` ``接口的COM ID。  
    

*   `` `ppFence` ``：访问围栏的`` `ID3D11FENCE` ``接口的指针。  
    

## 检测对4X MSAA质量级别的支持  

在D3D12中，我们无须检验4X MSAA是否正常开启，但是还是需要对质量级别进行检测。  

我们通过设置`` `D3D12_FEATURE_DATA_MULTISAMPLE_QUALITY_LEVELS` ``结构体成员，并将它传递给`` `ID3D12Device::CheckFeatureSupport` ``方法查询到对应的质量级别。 其中，`` `D3D12_FEATURE_DATA_MULTISAMPLE_QUALITY_LEVELS` ``结构体定义如下：  

*   `` `Format` ``：指定关于`` `DXGI-FOMRMAT` ``类型的返回值。我们指定的`` `DXGI_FORMAT_R8G8B8A8_UNORM` ``是归一化处理的无符号整数。  
    

*   `` `SampleCount` ``：每个像素的子采样点的数量。  
    

*   `` `Flags` ``：`` `D3D12_MULTISAMPLE_QUALITY_LEVEL_FLAGS` ``类型的采样质量级别。我们指定的`` `D3D12_MULTISAMPLE_QUALITY_LEVELS_FLAG_NONE` ``表示没有任何选项支持。  
    

*   `` `NumQualityLevels` ``：质量水平。我们在后面通过`` `ID3D12Device::CheckFeatureSupport` ``方法，根据前3个成员填充这个成员，如果是0，则这个结构体成员设置不被支持。  
    

`` `ID3D12Device::CheckFeatureSupport` ``方法原型如下：  

*   `` `Feature` ``：想要查询支持特性的类型。我们指定的enum`` `D3D12_FEATURE_MULTISAMPLE_QUALITY_LEVELS` ``表示MSAA特性。  
    

*   `` `[in,out] pFeatureSupportData` ``：指向与特征参数值相对应的数据结构的指针。  
    

*   `` `FeatureSupportDataSize` ``：`` `pFeatureSupportData` ``指针的size。  
    

## 创建命令队列和命令列表  

CPU通过命令列表对GPU进行通信。它们三者的关系是：首先CPU创建命令列表，然后将关联在命令分配器上的命令传入命令列表，最后将命令列表传入命令队列给GPU处理。 下面我们进行三者的创建工作：  

在D3D12中，命令队列被抽象为`` `ID3D12CommandQueue` ``接口来表示，要通过填写`` `D3D12_COMMAND_QUEUE_DESC` ``结构体来描述队列：  

*   `` `Type` ``：指定的`` `D3D12_COMMAND_LIST_TYPE` ``。我们指定的`` `D3D12_COMMAND_LIST_TYPE_DIRECT` ``表示指定GPU可以执行的命令缓冲区，直接命令列表未继承任何GPU状态。  
    

*   `` `Priority` ``：命令列表的优先级。可以选择enum`` `D3D12_COMMAND_LIST_TYPE` ``。  
    

*   `` `Flags` ``：指定enum`` `D3D12_COMMAND_QUEUE_FLAGS` ``，可以选择GPU特性(?)。我们指定的`` `D3D12_COMMAND_QUEUE_FLAG_NONE` ``表示指示默认命令队列。  
    

*   `` `NodeMask` ``：对于单个GPU操作，将其设置为零。如果有多个GPU节点，请设置一些以识别命令队列应用的节点（设备的物理适配器）。掩模中的每个位对应于一个节点，必须设置1位。  
    

再调用`` `ID3D12Device::CreateCommandQueue` ``方法创建队列：  

*   `` `pDesc` ``：指定描述命令队列的`` `D3D12_COMMAND_QUEUE_DESC` ``地址。  
    

*   `` `rrid` ``：命令队列的COM ID  
    

*   `` `ppCommandQueue` ``：一个指向内存块的指针，该指针接收指向命令队列的ID3D12Commandqueue接口的指针。  
    

记录在命令列表内的命令，实际上是存储在与之关联的命令分配器(command allocator)上的。`` `ID3D12CommandAllocator` ``是与命令有关的内存类接口。而命令分配器则由`` `ID3D12Device::CreateCommandAllocator` ``方法来创建，一个命令分配器可以关联多个命令列表，但是只能同时记录一个命令列表：  

*   `` `type` ``：D3D12_COMMAND_LIST_TYPE型值，该值指定要创建的命令分配器的类型。命令分配器的类型可以是记录直接命令列表或捆绑包的类型。我们指定的`` `D3D12_COMMAND_LIST_TYPE_DIRECT` ``表示指定GPU可以执行的命令缓冲区，直接命令列表未继承任何GPU状态。  
    

*   `` `rrid` ``：`` `ID3D12CommandAllocator` ``的COM ID。  
    

*   `` `ppCommandAllocator` ``：指向内存块的指针，该指针接收指向命令分配器的`` `ID3D12CommandAllocator` ``接口的指针。  
    

命令列表则由`` `ID3D12Device::CreateCommandList` ``接口创建：  

*   `` `nodeMask` ``：对于单GPU操作，将其设置为零。如果有多个GPU节点，请设置一些以识别用于创建命令列表的节点（设备的物理适配器）。掩模中的每个位对应于一个节点。必须设置一位。  
    

*   `` `type` ``：指定要创建的命令列表的类型。我们指定的enum`` `D3D12_COMMAND_LIST_TYPE_DIRECT` ``表示指定GPU可以执行的命令缓冲区，直接命令列表未继承任何GPU状态。  
    

*   `` `pCommandAllocator` ``：设备创建命令列表的命令分配器对象的指针。  
    

*   `` `pInitialState` ``：渲染管线的初始状态。空指针表示不进行绘制。  
    

*   `` `rrid` ``：`` `ID3D12GraphicsCommandList` ``的COM ID  
    

*   `` `ppCommandList` ``：指向`` `ID3D12 CommandList` ``或`` `ID3D12GraphicsCommandList` ``接口的指针。  
    

## 描述并创建交换链  

交换链由前台缓冲区和后台缓冲区构成，它们两种缓冲区互换进行呈现操作，避免了动画中出现画面闪烁的现象。   

首先，我们用`` `DXGI_SWAP_CHAIN_DESC` ``结构体描述欲创建交换链的特性：  

*   `` `BufferDesc` ``：这个结构体描述了待创建后台缓冲区的属性。  
    

*   `` `SampleDesc` ``：多重采样的质量级别以及对每个像素的采样次数。  
    

*   `` `BufferUsage` ``：`` `DXGI_USAGE` ``列举类型的成员，该类型描述了后缓冲区的表面使用情况和CPU访问选项。后台缓冲区可用于着色器输入或渲染目标输出。由于我们要将数据渲染至后台缓冲区（即用它作为渲染目标），因此将此参数指定为`` `DXGI_USAGE_RENDER_TARGET_OUTPUT` ``。  
    

*   `` `BufferCount` ``：交换链中所用的缓冲区数量。  
    

*   `` `OutputWindow` ``：渲染窗口的句柄。  
    

*   `` `Windowed` ``：若指定为true，程序将在窗口模式下运行；如果指定为false，则采用全屏模式。  
    

*   `` `SwapEffect` ``：`` `DXGI_SWAP_EFFECT` ``枚举类型的成员，该类型描述了在介绍表面后处理演示缓冲区内容的选项。看文档说好像D3D12只能指定为`` `DXGI_SWAP_EFFECT_FLIP_DISCARD` ``。  
    

*   `` `Flags` ``：`` `DXGI_SWAP_CHAIN_FLAG` ``枚举类型的成员，描述了交换链行为的选项。我们指定的`` `DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH` ``表示自适应窗口模式（自动选择最适于当前窗口尺寸的显示模式）。  
    

其中，`` `DXGI_MODE_DESC` ``结构体定义如下：  

*   `` `Width` ``：缓冲区分辨率宽度。  
    

*   `` `Height` ``：缓冲区分辨率高度。  
    

*   `` `RefreshRate` ``：刷新率。`` `DXGI_RATIONAL` ``是一个表示分数的结构体。  
    

*   `` `Format` ``：缓冲区的显示格式。我们指定的`` `DXGI_FORMAT_R8G8B8A8_UNORM` ``表示A four-component, 32-bit unsigned-normalized-integer format that supports 8 bits per channel including alpha。  
    

*   `` `ScanlineOrdering` ``：`` `DXGI_MODE_SCANLINE_ORDER` ``类型的值描述了扫描行绘图模式。我们指定的`` `DXGI_MODE_SCANLINE_ORDER_UNSPECIFIED` ``表示未指定拉伸。  
    

*   `` `Scaling` ``：图像如何相对于屏幕进行拉伸。我们指定的`` `DXGI_MODE_SCALING_UNSPECIFIED` ``表示未指定拉伸。  
    

*   `` `Stereo` ``：指定全屏显示模式是否为立体声。  
    

描述完交换链后，我们用`` `IDXGIFactory::CreateSwapChain` ``的方法来创建交换链：  

*   `` `pDevice` ``：指向`` `ID3D12CommandQueue` ``接口的指针。  
    

*   `` `pDesc` ``：指向描述交换链结构体的指针。  
    

*   `` `ppSwapChain` ``：返回所创建的交换链接口。  
    

## 创建描述符堆  

描述符是一种对送往GPU的资源进行描述的轻量级结构。CPU通过绑定描述符告诉GPU这个资源是干什么的，而且我们可借助描述符来指定欲绑定资源中的局部数据。每个描述符都有一种具体类型，此类型指明了资源的具体作用，我们可以用多个描述符来引用同一个资源，本书常用描述符如下：  

*   CBV/SRV/UAV描述符分别表示的是常量缓冲区视图(constant buffer view)，着色器资源视图(shader resource view)和无序访问视图(unordered access view)这3种资源。  
    

*   常量缓冲区：一种GPU资源(`` `ID3D12Resource` ``)，其数据内容可供着色器程序所引用。通常由CPU每帧更新一次。  
    

*   常量缓冲区视图：需要存放在以`` `D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV` ``类型所建的描述符堆里。这种堆内可以混合常量缓冲区描述符、着色器资源描述符和无序访问描述符。非数值数据是不能添加到常量缓冲区的，如Texture2D、SamplerState（这些可以绑在register上）  
    

*   无序访问视图：允许多个GPU线程读取和写入的buffer类型。  
    

*   采样器(sampler)描述符表示的是采样器资源（用于纹理贴图）。  
    

*   RTV描述符表示的是渲染目标视图资源(render target view)。  
    

*   DSV描述符表示的是深度/模板视图资源(depth/stencil view)。  
    

描述符堆(descriptor heap)中存有一系列描述符（可将其看作是描述符数组），本质上是存放用户程序中某种特定类型描述符的一块内存。我们需要为每一种类型的而描述符都创建出单独的描述符堆。另外，也可以为同一种描述符类型创建出多个描述符堆。  

我们用`` `ID3D12DescriptorHeap` ``接口来表示描述符堆，首先使用`` `D3D12_DESCRIPTOR_HEAP_DESC` ``结构体描述我们要创建堆的基本信息：  

*   `` `Type` ``：指出堆的类型。  
    

*   `` `NumDescriptors` ``：堆中的描述符数。  
    

*   `` `Flags` ``：通过按位或运算指定堆的用法。我们指定的`` `D3D12_DESCRIPTOR_HEAP_FLAG_NONE` ``表示默认用法。  
    

*   `` `NodeMask` ``：对于单个显示适配器，把它置于0。 If there are multiple adapter nodes, set a bit to identify the node (one of the device's physical adapters) to which the descriptor heap applies. Each bit in the mask corresponds to a single node. Only one bit must be set.  
    

然后用`` `ID3D12Device::CreateDescriptorHeap` ``方法来创建它：  

*   `` `pDescriptorHeapDesc` ``：描述堆信息的`` `D3D12_DESCRIPTOR_HEAP_DESC` ``的地址。  
    

*   `` `rrid` ``：描述符堆`` `ID3D12DescriptorHeap` ``接口的COM ID。  
    

*   `` `ppvHeap` ``：指向内存块的指针，该指针接收到描述符堆的指针。`` `ppVHeap` ``可以NULL，以实现功能测试。当`` `PPVheap` ``为NULL时，将不会创建对象，并且当`` `Pdescriptorheapdesc` ``有效时，将返回S_FALSE。  
    

## 获取描述符大小  

然后我们获取描述符大小，这个大小可以让我们知道描述符堆中每个元素的大小（描述符在不同的GPU平台上的大小各异），方便我们之后在地址中做偏移来找到堆中的描述符元素。这里我们获取三个描述符大小，分别是RTV（渲染目标缓冲区描述符）、DSV（深度模板缓冲区描述符）、CBV_SRV_UAV（常量缓冲区描述符、着色器资源缓冲描述符和随机访问缓冲描述符）。  

## 创建渲染目标视图  

有了描述符堆，我们就可以创建堆中的描述符了。资源不能与渲染流水线的阶段直接绑定，所以我们必须先为资源创建视图（描述符），并将其绑定到流水线阶段。例如，为了将后台缓冲区绑定到流水线的输出合并阶段(output merge stage，这样D3D才能向其渲染)，便需要为该后台缓冲区创建一个渲染目标视图。  

具体过程是，先从RTV堆中拿到首个RTV句柄，然后获得存于交换链中的RT资源，最后创建RTV将RT资源和RTV句柄联系起来，并在最后根据RTV大小做了在堆中的地址偏移。  

我们首先定义`` `ID3D12Resource` ``COM接口数组，它们用于操作缓冲区。`` `ID3D12Resource` ``接口将物理内存与堆资源抽象组织为可处理的数据数组与多维数组，从而使CPU和GPU可以对这些资源进行读写。  

在程序中，我们是通过句柄来引用描述符的，并以`` `ID3D12DescriptorHeap::GetCPUDescriptorHandleForHeapStart` ``方法来获得描述符堆中第一个 描述符的句柄，然后用`` `d3dx12.h` ``中扩展的类`` `CD3DX12_CPU_DESCRIPTOR_HANDLE` ``来管理描述堆的当前元素指针位置，概念上可以将这个对象理解为一个数组元素迭代器。它的构造函数初始化了`` `D3D12_CPU_DESCRIPTOR_HANDLE` ``结构体中的元素。  

因为是双后台缓冲，所以我们要创建两个RTV。对于每个RTV，我们先用`` `IDXGISwapChain::GetBuffer` ``方法获取存于交换链中的后台缓冲区资源：  

*   `` `Buffer` ``：希望获得的特定后台缓冲区的索引。注意这个索引要和对应描述符的数组元素序号保持一致。  
    

*   `` `rrid` ``：用于操纵缓冲区的`` `ID3D12Resource` ``接口类型COM ID。  
    

然后，我们用`` `ID3D12Device::CreateRenderTargetView` ``方法来为获取的后台缓冲区创建渲染目标视图资源：  

*   `` `pResource` ``：指定用于用作渲染目标的资源。在上面的例子中是后台缓冲区（即为后台缓冲区创建了一个渲染目标视图）。  
    

*   `` `pDesc` ``：指向`` `D3D12_RENDER_TARGET_VIEW_DESC` ``数据结构实例的指针，该结构体迷哦奥数了资源中元素的树蕨类型（格式）。如果该资源在创建时已经指定了具体格式（即此资源不是无类型格式），那么就可以把这个参数设为空指针，表示采用该资源创建时的格，为它的第一个`` `mipmap` ``层级（后台缓冲区只有一种`` `mipmap` ``层级，有关`` `mipmap` ``的内存将在第9章展开讨论）创建一个视图。由于已经指定了后台缓冲区的格式，因此就将这个参数设置为空指针。  
    

*   `` `DestDescriptor` ``：引用所创建渲染目标视图的描述符句柄。  
    

循环最后一行`` `Offset` ``则暴漏了这里其实是在操作数组的本质。  

## 创建深度/模板缓冲区  

深度缓冲区其实就是一种2D纹理，它存储着离管擦和这最近的可视对象的深度信息（如果使用了模板，还会附有模板信息）。  

纹理是一种GPU资源，因此我们要通过填写`` `D3D12_RESOURCE_DESC` ``结构体来描述纹理资源。`` `D3D12_RESOURCE_DESC` ``结构体的定义如下：  

*   `` `Dimension` ``：资源的维度。我们指定的`` `D3D12_RESOURCE_DIMENSION_TEXTURE2D` ``表示资源是2D纹理。  
    

*   `` `Alignment` ``：指定对齐。如果将对齐设置为0，则运行时将使用4MB用于MSAA纹理，而其他所有内容都将使用64KB。当纹理较小时，该应用程序可能会选择比这些违约较小的对齐方式。可以使用64KB对齐来创建具有未知布局和MSAA的纹理（如果通过下面详述的小尺寸限制）。  
    

*   `` `Width` ``：以纹素为单位来表示的纹理宽度。对于缓冲区资源来说，此项是缓冲区占用的字节数。  
    

*   `` `Height` ``：以纹素为单位来表示的纹理高度。  
    

*   `` `DepthOrArraySize` ``：以纹素为单位来表示的纹素深度，或者（对于1D纹理和2D纹理来说）是纹理数组的大小。注意，D3D中并不存在3D纹理数组的概念。  
    

*   `` `MipLevels` ``：`` `mipmap` ``层级的数量（第九章介绍`` `mipmap` ``）。对于深度/模板缓冲区而言，只有一个`` `mipmap` ``级别。  
    

*   `` `Format` ``：`` `DXGI_FORMAT` ``枚举类型中的成员之一，用于指定纹素的格式。对于深度/模板缓冲区来说，此格式需要从以下几种格式中选：  
    

*   `` `DXGI_FORMAT_D32_FLOAT_S8X24_UINT` ``：该格式共用64位，取其中的32位指定一个浮点型深度缓冲区，另有8位（无符号整数）分配给模板缓冲区(stencil buffer)，并将该元素映射到[0, 255]区间，剩下的24位仅用于填充对齐(padding)不做他用。  
    

*   `` `DXGI_FORMT_D32_FLOAT` ``：指定一个32位浮点型深度缓冲区。  
    

*   `` `DXGI_FORMAT_D24_UNORM_S8_UINT` ``：指定一个无符号24位深度缓冲区，并将该元素映射到[0, 1]区间。另有8位（无符号整数）分配给模板缓冲区，将此元素映射到[0, 255]区间。  
    

*   `` `DXGI_FORMAT_D16_UNORM` ``：指定一个无符号16位深度缓冲区，把该元素映射到[0, 1]区间。  
    

*   `` `SampleDesc` ``：多重采样的质量级别以及对每个像素的采样次数。为了存储每个子像素的颜色深度/模板信息，所用后台缓冲区和深度缓冲区的大小要4倍于屏幕分辨率。因此，深度/模板缓冲区与徐娜然目标的多重采样设置一定要相匹配。  
    

*   `` `Layout` ``：`` `D3D12_TEXTURE_LAYOUT` ``枚举类型的成员之一，用于指定纹理的布局。我们暂时还不用考虑这个问题，在此将它指定为`` `D3D12_TEXTURE_LAYOUT_UNKOWN` ``即可。它表示表明布局未知，并且可能依赖于适配器。在创建过程中，驱动程序选择了基于其他资源属性，尤其是资源大小和标志的最有效的布局。除非其他纹理布局需要某些功能，否则更喜欢此选择。  
    

*   `` `Flags` ``：与资源有关的杂项标志。对于一个深度/模板缓冲区资源来说，要将此项指定为`` `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` ``。它表示允许为资源创建深度模具视图，并使资源能够过渡到D3D12_RESOURCE_STATE_DEPTH_WRITE和/或`` `D3D12_RESOURCE_STATE_STATE_DEPTH_READ` ``。大多数适配器体系结构为纹理分配额外的内存，以减少有效的带宽并最大程度地提高对早期深度测试的优化。当永远不会发生深度操作时，应用程序应避免设置此标志。  
    

GPU资源都存于堆(heap)中，其本质是具有特定属性的GPU显存快。`` `ID3D12Device::CreateCommittedResource` ``方法将根据我们所提供的属性创建一个资源与一个堆，并把资源提交到这个堆中：  

*   `` `pHeapProperties` ``：（资源欲提交至的）堆所具有的属性结构体。有一些属性是针对高级用法而设。目前只关心第一个成员代表的属性即`` `D3D12_HEAP_TYPE` ``枚举成员，它的成员列表如下：  
    

*   `` `D3D12_HEAP_TYPE_DEFAULT` ``：默认堆(default heap)。向这堆离提交的资源，唯独GPU可以访问。 举一个有关深度/模板缓冲区的例子：GPU会读写深度/模板缓冲区，而CPU从不需要访问它，所以深度/模板缓冲区应被放入默认堆中。  
    

*   `` `D3D12_HEAP_TYPE_READBACK` ``：回读堆(read-back heap)。向这种堆里提交的都是需要由CPU读取的资源。  
    

*   `` `D3D12_HEAP_TYPE_CUSTOM` ``：此成员应用于高级场景。  
    

*   `` `HeapFlags` ``：与（资源欲提交至的）堆有关的额外选项标志。通常将它设为`` `D3D12_HEAP_FLAG_NONE` ``。它表示没有指定选项。  
    

*   `` `pDesc` ``：指向一个`` `D3D12_RESOURCE_DESC` ``实例的指针，用它迷哦奥数代建的资源。  
    

*   `` `InitialResourceState` ``：在资源创建时，需要用此参数来设置它的初始状态。对于深度/模板缓冲区来说，通常将其初始状态设置为`` `D3D12_RESOURCE_STATE_COMMON` ``。  
    

*   `` `pOptimizedClearValue` ``：指向一个`` `D3D12_CLEAR_VALUE` ``对象的指针，它描述了一个用于清除资源的优化值。选择适当的优化清除值，可提高清除操作的执行速度。若不希望指定优化清除值，可把此参数设为`` `nullptr` ``：  
    

*   `` `Format` ``：一个`` `DXGI_FORMAT` ``枚举的成员。我们指定的`` `DXGI_FORMAT_D24_UNORM_S8_UINT` ``表示24位深度，8位模板。  
    

*   `` `Color` ``：指定浮点值的四个输入阵列，确定RGBA值。RGBA的顺序与`` `ClearRenderTargetView` ``一起使用的顺序匹配。  
    

*   `` `DepthStencil` ``：指定`` `D3D12_DEPTH_STENCEL_VALUE` ``的一个成员。这些值与`` `cleardepthstencilview` ``中的深度语义和模具相匹配：  
    

在使用深度/模板缓冲区之前，一定要创建相关的深度/模板视图 ，并将它们绑定到渲染流水线上。与创建渲染目标视图不同的是，我们用的是`` `D3D12_DEPTH_STENCIL_VIEW_DESC` ``结构体描述深度/模板缓视图（而RTV使用的是从交换链中的后台缓冲区资源中获取的句柄）。我们用`` `ID3D12Device::CreateDepthStencilView` ``方法创建深度/模型视图：  

*   `` `pResource` ``：代表深度/模板视图的`` `ID3D12Resource` ``对象的指针。  
    

*   `` `pDesc` ``：描述深度/模板视图的`` `D3D12_DEPTH_STENCEL_VIEW_DESC` ``结构体的地址。  
    

*   `` `DestDecriptor` ``：描述代表具有深度/模板视图的堆开始的CPU描述符，即DSV句柄。  
    

## 标记深度/模板资源的状态  

创建完描述符后，我们需要标记深度/模板资源的状态。为什么要标记呢？因为资源在不同的时间段有着不同的作用，比如，有时候它是只读，有时候又是可写入的。而标记DS状态需要创建一个资源屏障，它的核心思想就是追踪资源权限的变换，从而同不GPU上前后执行命令堆访问资源的操作。  

我们利用`` `ResourceBarrier` ``方法将深度/模板资源状态从`` `D3D12_RESOURCE_STATE_COMMON` ``转换到`` `D3D12_RESOURCE_STATE_DEPTH_WRITE` ``状态，将其转为可以绑定在渲染流水线上的深度/模板缓冲区。 我们首先用`` `ID3D12GraphicsCommandList::ResourceBarrier` ``方法来创建一个资源屏障：  

*   `` `NumBarriers` ``：资源屏障的数量。  
    

*   `` `pBarries` ``：描述资源屏障的地址。  
    

我们用`` `CD3DX12_RESOURCE_BARRIER::Transition` ``方法填充`` `pBarries` ``参数，它是`` `d3dx12.h` ``中的帮助结构体中的一个静态内联函数：  

*   `` `pResourceBefore` ``：视图的COM接口(?)。这里我们指定之前的深度/模板缓冲区的接口。  
    

*   `` `stateBefore` ``：转换前的状态。[状态枚举API](https://docs.microsoft.com/en-us/windows/win32/api/d3d12/d3d12-d3d12_resource_states)  
    

*   `` `stateAfter` ``：转换后的状态。[状态枚举API](https://docs.microsoft.com/en-us/windows/win32/api/d3d12/d3d12-d3d12_resource_states)  
    

等所有命令都进入`` `ID3D12GraphicsCommandList` ``COM后，还需要用`` `ID3D12CommandQueue::ExecuteCommandLists` ``方法将命令从命令列表传入命令队列，也就是从CPU传入GPU的过程。 注意：在传入命令队列前必须关闭命令列表。 其中`` `ID3D12CommandQueue::ExecuteCommandLists` ``定义如下：  

*   `` `NumCommandLists` ``：要执行的命令列表数量。  
    

*   `` `ppCommandLists` ``：执行的`` `ID3D12CommandList` ``COM接口。  
    

## 实现围栏  

为了使CPU和GPU同步，我们之前已经创建了围栏接口，现在需要实现围栏代码。  

实现思想是，围栏初始值为0（可理解为GPU端的围栏值），现我们定义一个当前围栏值也为0（可理解为CPU端的围栏值），当CPU将命令传递至GPU后，当前围栏值++（CPU围栏++），而当GPU处理完CPU传来的命令后，围栏值++（GPU围栏++），然后判定围栏值（CPU围栏值）和当前围栏值（GPU围栏值）的大小，来确定GPU是否命中围栏点，如果没有命中，则等待命中后触发事件。 我们使用`` `ID3D12CommandQuque::Signal` ``方法更新围栏为指定的值：  

*   `` `pFence` ``：指向`` `ID3D12Fence` ``的指针。  
    

*   `` `Value` ``：更新后围栏的值  
    

当GPU没有处理完围栏+1的任务时，我们需要等待直到这个任务完成。具体过程是先用`` `CreateEvent` ``来创建一个事件：  

*   `` `lpEventAtrributes` ``：指向`` `SECURITY_ATTRIBUTES` ``结构体的指针，描述句柄是否可以继承。如果此参数为null，则无法通过子进程继承句柄。  
    

*   `` `bManualReset` ``：与多线程有关，我们置于`` `false` ``。If this parameter is TRUE, the function creates a manual-reset event object, which requires the use of the [ResetEvent](https://docs.microsoft.com/en-us/windows/desktop/api/synchapi/nf-synchapi-resetevent) function to set the event state to nonsignaled（未置位）. If this parameter is FALSE, the function creates an auto-reset event object, and system automatically resets the event state to nonsignaled（未置位） after a single waiting thread has been released.  
    

*   `` `bInitialState` ``：与多线程有关，我们置于`` `false` ``。If this parameter is TRUE, the initial state of the event object is signaled（置位）; otherwise, it is nonsignaled（未置位）。  
    

*   `` `lPName` ``：事件对象的名称。  
    

然后将这个事件用`` `ID3D12Fence::SetEventOnCompletion` ``方法将其设置为围栏值达到设定值后触发：  

*   `` `Value` ``：当事件发生时，围栏的值。  
    

*   `` `hEvent` ``：触发的事件的句柄。  
    

最后用`` `WaitForSingleObject` ``方法来等待（阻塞线程）直到这个事件触发或超时：  

*   `` `hHandle` ``：等待的事件的句柄。  
    

*   `` `dwMilliseconds` ``：最大等待事件(ms)。  
    

当线程阻塞结束即围栏值更新完成，我们用`` `CloseHandle` ``方法关闭这个事件。  

## 设置视口和裁剪矩阵  

# 绘制命令  

这个过程相对好理解，可以看参考里卡卡写的专栏。  

最终，我们运行可以得到如下结果：   

# 参考  

DirectX 12 3D游戏开发实战 第四章 [MSDN](https://docs.microsoft.com/zh-cn/) [卡卡-DX12初始化篇-初始化基础模块](https://zhuanlan.zhihu.com/p/129665235) [MaxwellGeng-Windows渲染引擎开发入门教学（1）: HelloWorld](https://zhuanlan.zhihu.com/p/457348124) [GamebabyRockSun_QQ-DirectX12（D3D12）基础教程（一）——基础教程](https://gamebaby.blog.csdn.net/article/details/82730776?spm=1001.2014.3001.5502) [bebh-Effective D3D12——龙书中常见术语、API与11个优化建议](https://zhuanlan.zhihu.com/p/419643738)  

[本篇笔记代码](https://github.com/dyxdyxdyx/MyDragonBook)