# 变量类型
## 标量/向量

**标量类型**

`bool ,int ,float,double` 基本上可 C++ 差不多，所以不再这里讲解；  
`half`:16 位的浮点数

**向量类型**

`float2` ,2D 向量，内部存储着两个浮点类型 float  
`float3` ,3D 向量，内部存储着三个浮点类型 float  
`float4` ,4D 向量，内部存储着四个浮点类型 float  
除此之外还有：  
`int2` ,2D 整数，内部存储着两个类型 int  
`int3` ,3D 整数，内部存储着三个类型 int  
`int4` ,4D 整数，内部存储着四个类型 int  
除此之外还有：  
`bool2` ,2Dbool，内部存储着两个类型 bool  
`bool3` ,3Dbool，内部存储着三个类型 bool  
`bool4`,4Dbool，内部存储着四个类型 bool  
...  
TypeN (2<=N<=4)  
...

**向量初始化**
```c++ nums
float3 v = {1.0f, 2.0f, 3.0f};  
float2 w = float2(x, y);// 内部支持联等  
float4 u = float4(w, 3.0f, 4.0f); // u= (w.x, w.y, 3.0f, 4.0f)  
```
向量的访问  
用数组语法：  
如：`pos[i] = 3.14f; ` 
用字母语法，xyzw，rgba。  
如：  
```c++ nums
pos.x = pos.r = 1.0f;  
pos.y = pos.g = 2.0f;  
pos.z = pos.b = 3.0f;  
pos.w = pos.a = 4.0f;
```

**向量的替换调配**
```c++ nums
float2 pos = {x,y};  
float3 pos = {x,y,z};  
float4 pos = {x,y,z,w};  
//例如  
float4 UV1 = {0.1f, 0.2f, 0.3f, 0.4f};  
float4 UV2 = {0.0f, 0.0f, 0.0f, 0.0f};  
UV2 = UV1 .wyyx; // v = {0.4f, 0.2f, 0.2f,0.10=f}  
UV2 = UV1 .wzyx; // v = {0.4f, 0.3f, 0.2f,0.1f}  
UV2.xy = u; // v = {0.1f,0.2f, 0.0f,0.0f}
```


## **矩阵类型**

**矩阵定义**

`float2x2`:2x2 的矩阵，类型是 float;  
`half3x3`:3x3 矩阵类型是 half  
`int4x4`: 4 × 4 矩阵类型是 int  
`bool3x4`: 3 × 4 矩阵类型是 bool

**矩阵访问**

矩阵的数组语法：`M[i][j] = value`;
除此之外，可以依照访问结构体成员的方式获取元素：
```c++ nums
// 以1作为起始值的索引
M._11 = M._12 = M._13 = M._14 = 0.1f;  
M._21 = M._22 = M._23 = M._24 = 0.2f;  
M._31 = M._32 = M._33 = M._34 = 0.3f;  
M._41 = M._42 = M._43 = M._44 = 0.4f;  

// 以0作为基准值的索引
M._m00 = M._m01 = M._m02 = M._m03 =0.5f;  
M._m10 = M._m11 = M._m12 = M._m13 =0.6f;  
M._m20 = M._m21 = M._m22 = M._m23 =0.7f;  
M._m30 = M._m31 = M._m32 = M._m33 =0.8f;
```

**通过数组单下标获取行向量**

**矩阵初始化**
```c++ nums
float2x2 Mat =float2x2(1.0f,2.0f,3.0f,4.0f);  
int2x2 Mat ={1,2,3,4};
```

**矩阵赋值**
```c++ nums
float3 Nor = normalize(pIn.normalW);  
float3 Tan = normalize(pIn.tangentW -dot(pIn.tangentW, N)*N);// 固定写法  
float3 B = cross(Nor ,Tan);  
float3x3 NTB;  
NTB[0] = Nor ; // float3  
NTB[1] =Tan ; // float3  
NTB[2] = B; // float3

```

**除了利用 float4 与 float4x4 类型来表示 4D 向量与 4x4 矩阵外，我们还可以使用 vector 与 matrix 类型来加以代替：**
```c++ nums
vector u = {1.0f, 2.0f, 3.0f, 4.0f};  
matrix M; // 4x4 矩阵
```

## **数组**
和 C++一样
```c++ nums
float C[8][7];  
half t[n];  
int3 v[42];//42 个 int3
```

## **结构体**

类似 C++ 中的结构体，区别在于不具有成员函数
```c++ nums
struct TastShader  
{  
float3 normal;  
float3 postion;  
float4 diffuse;  
float4 spec;  
};  
TastShader T;  
litColor += T.diffuse;  
dot(lightVec, T.normal);  
float specPower = max(T.spec.a, 1.0f)；
```

## 变量的修饰符


# 关键字与运算符
## 关键字

# 控制流
和 C++相同

# 函数
## 自定义函数


而且，HLSL 还专为函数添加了一些额外的关键字。例如，考虑下列以 HLSL 编写的函数。
```c++ nums
bool foo (in const bool b, // 输入的bool类型参数
          out int r1,      // 输出的int类型参数
          inout float r2)  // 同时兼具输入/输出属性的float类型参数
{
    if( b )   // 测试输入值
    {
        r1 = 5;   // 通过r1输出一个值
    }
    else
        r1 = 1;   // 通过r1输出一个值
    }
    // 因为r2的类型为inout，所以既可把它作为输入值，也能通过它来输出值
    r2 = r2 * r2 * r2 ;
    
    return true;
}
```

除了 `in`、 `out` 与 `inout` 关键字以外，HLSL 中函数的语法基本与 C++中的函数相一致。

1. `in`: 在目标函数执行之前, 此修饰符所指定的参数应从调用此函数的程序中复制有输入的数据。我们其实不必显式地为参数指定此修饰符，**在默认的情况下参数都为 in 类型**。
2. `out`: 在目标函数返回时，此修饰符所指定的参数应当已经复制了该函数中的最后计算结果。借此即可方便地返回数值。关键字 out 的存在是很有必要的，因为 **HLSL 既不允许按引用的方式传递数据，也不允许使用指针**。需要注意的是，如果一个参数被标记为 out, 那么此参数在目标函数执行之前不能被复制任何值。换言之，out 参数只能用于输出数据而不能用于输入数据。
3. `inout`: 此修饰符表示参数兼有 in 与 out 的属性。如果希望某个参数既可输入又可输出就可用 inout。
## 内置函数
[内部函数 - Win32 apps | Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/direct3dhlsl/dx-graphics-hlsl-intrinsic-functions)

## 缓冲区的封装规则

# **着色器关联**

顶点和像素着色器提供的数据供其后的图形管线使用. 输出关联用于指定着色器产生的数据如何链接给下一阶段的输入. 举例来说, 顶点着色器的输出关联将顶点着色程序运算得到的结果链接到像素着色器的输入关联上.

顶点着色器输出关联用于将着色器连接到像素着色器和光栅阶段. POSITION 输出是每个顶点必须输出给光栅器而不暴露给像素着色器的数据,

TEXCOORDn 和 COLORn 表示输出用于像素着色器进行插值.

像素着色器输出关联将其输出颜色绑定给正确的渲染目标. 颜色输出被连接到 alpha 混合阶段. DEPTH 输出关联用于改变当前光栅化位置的目标深度值. 输出关联语义同输入关联的声明相同.

**顶点着色器的输入关联**

**关联符** **描述**  
POSITIONn 位置  
BLENDWEIGHTn 混合宽  
BLENDINDICESn 混合索引  
NORMALn 标准向量  
PSIZEn 点大小  
COLORn 颜色  
TEXCOORDn 纹理坐标  
TANGENTn 切线  
BINORMALn 副法线  
TESSFACTORn 细分因子

**像素着色器输入关联**

**关联符** **描述**  
COLORn 颜色  
TEXCOORDn 纹理坐标  
注意：n 是一个可选的整形变量

