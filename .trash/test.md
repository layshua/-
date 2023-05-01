
# DX12管线梳理

## 整条线总结

### Pre Init

-   命令解析
    
-   日志系统
    

### Init

-   InitDirect3D
    
    -   D3D12GetDebugInterface
        
    -   CreateDXGIFactory1
        
    -   D3D12CreateDevice
        
    -   CreateFence
        
    -   CreateCommandQueue
        
    -   CreateCommandAllocator
        
    -   CreateCommandList
        
    -   CheckFeatureSupport
        
    -   CreateSwapChain
        
    -   CreateDescriptorHeap
        
        -   RTVHeap
            
        -   DSVHeap
            
-   PostInitDirect3D
    
    -   WaitGPUCommandQueueComplete
        
    -   CommandAllocator
        
        -   Reset
            
    -   SwapChainBuffer
        
        -   Reset
            
    -   ResizeBuffers
        
        -   自适应屏幕变大
            
    -   CreateRenderTargetView
        
        -   SwapChainBuffer
            
    -   CreateCommittedResource
        
    -   CreateDepthStencilView
        
    -   ExecuteCommandLists
        
    -   ViewprotInfo
        

### Post Init

-   模型前期数据注册
    
-   根签名
    
-   Shader输入布局
    
-   构建模型数据
    
    -   将数据存储在上传堆上
        
-   构建堆
    
-   构建缓冲区
    
    -   描述模型状态
        
-   构建摄像机常量缓冲区
    
-   属性和参数绑定PSO
    

### Tick

-   自由主题
    
-   UpdateCalculations
    
    -   视口状态和位置
        
    -   模型状态和位置
        
-   Pre Draw
    
    -   Reset PSO
        
-   Tick
    
    -   绑定矩形框
        
    -   清除画布
        
    -   清除深度模板缓冲区
        
    -   输出的合并阶段
        
-   Draw
    
    -   SetDescriptorHeaps
        
        -   CBVHeap
            
    -   SetGraphicsRootSignature
        
        -   RootSignature
            
    -   MeshDraw
        
        -   DrawViewport
            
            -   根据堆内存偏移绑定到CommandList
                
        -   DrawMesh
            
            -   根据堆内存偏移绑定到CommandList
                
    -   DirectXPipelineState
        
        -   CaptureKeyboardKeys
            
-   Post Draw
    

## 常量缓冲区

### 常量缓冲区

-   ConstantBuffer
    
    -   b0-bN
        

### 着色器资源

-   ShaderResourceViews
    
    -   t0-tN
        

### 无序访问

-   UnorderedAccessViews
    
    -   u0-uN
        

### 纹理采样器

-   Texture Samplers
    
    -   s0-sN
        

## DX12 基础结构

### 初始化

-   DXAPI初始化
    
-   绑定渲染管线
    
    -   PSO
        

### 渲染

-   模型和视口世界空间状态更新
    
-   绑定命令列表
    
    -   提交
        

## SLB

## InitWindows