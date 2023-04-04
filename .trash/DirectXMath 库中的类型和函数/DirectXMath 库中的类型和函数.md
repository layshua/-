第一部分主要讲的是一些数学知识，这些都是线性代数的内容不不必赘述了。  

主要记录一下DirectXMath库中提供的变量类型和函数当作字典，其中包括SIMD架构的好处。  

# SIMD架构  

[https://zhuanlan.zhihu.com/p/514966360](https://zhuanlan.zhihu.com/p/514966360) ![](/api/filetransfer/images?url=https%3A%2F%2Fddddyx.oss-cn-beijing.aliyuncs.com%2Fimg%2F202208171924539.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_28%252Ctext_dGhldHVz%252Ccolor_FFFFFF%252Cshadow_50%252Ct_80%252Cg_se%252Cx_10%252Cy_10&sign=5191a58539095a3a8824482ee3e5c2cc67b8d34370ecc12876fec5bad33c9017)  

# 向量  

DirectXMath是D3D使用的数学库，它借助128位宽的SIMD寄存器，利用一条SIMD指令即可对4个32位浮点数或整数进行运算。也就是说，我们可以用1条SIMD加法指令取代4D向量中4条普通的标量指令，从而直接算出4D向量的加法结果。3D和2D向量用不到的部分置0并忽略。  

DirectXMath关联的头文件：  

*   `` `DirectXMath.h` ``，命名空间`` `DirectX` ``  
    

*   `` `DirectXPackedVector.h` ``，包含相关的数据结构，命名空间`` `DirectX::PackedVector` ``  
    

注意在x86平台，需要打开vs的SSE2指令集。  

注： 将`` `XM_CALLCONV` ``调用约定注解加在非构造函数名之前时，它会根据编译器的版本确定出对应的调用约定属性。  

## 向量类型  

向量类型定义：`` `typedef __m128 XMVECTOR` ``，这里的`` `__m128` ``是一种特殊的SIMD类型。  

`` `XMVECTOR` ``类数据需要按16位字节对齐，这对于局部变量和全局变量都是自动实现的。 对于类中的数据结构，分别使用`` `XMFLOAT2、XMFLOAT3、XMFLOAT4` ``类型来代替，它们分别对应2D、3D、4D结构体，其中声明了各个维度的变量和构造函数，重载了`` `=` ``运算符。  

为了发挥SIMD的高效特性，计算时需要把结构体实例转换成`` `XMVECTOR` ``类型——加载函数(loading function)，存储时需要把`` `XMVECTOR` ``转换成`` `XMFLOATn` ``——存储函数(storage function)。  

## 加载方法和存储方法  

加载方法： `` `XMFLOATn` ``→`` `XMVECTOR` ``： `` `XMVECTOR XM_CALLCONV XMLoadFloat[n](const XMFLOAT[n] *pSource)` `` 其中`` `[n]` ``为`` `2` ``, `` `3` ``, `` `4` `` 加载某一个分量[I]： `` `float XM_CALLCONV XMVectorGet[I](FXMVECTOR V)` `` 其中`` `[I]` ``为`` `X` ``, `` `Y` ``, `` `Z` ``, `` `W` ``  

存储方法： `` `XMVECTOR` ``→`` `XMFLOATn` ``： `` `void XM_CALLCONV XMStoreFloat[n](XMFLOAT[n] *pDestination, FXMVECTOR V` ``。 其中`` `[n]` ``为`` `2` ``, `` `3` ``, `` `4` `` 存储某一个分量： `` `XMVECTOR XM_CALLCONV XMVectorSet[I](FXMVECTOR V, float [i])` `` 其中`` `[I]` ``为`` `X` ``, `` `Y` ``, `` `Z` ``, `` `W` ``，`` `[i]` ``为`` `2` ``, `` `3` ``, `` `4` ``  

## 参数的传递  

为了提高效率，可以将XMVECTOR类型的值作为函数参数，直接传送至SSE/SSE2寄存器（register）里，而不存于栈（stack）内。  

传递XMVECTOR参数的规则如下：  

*   前3个XMVECTOR参数应当用类型 FXMVECTOR；  
    

*   第4个XMVECTOR参数应当用类型 GXMVECTOR；  
    

*   第5、6个XMVECTOR参数应当用类型HXMVECTOR；  
    

*   其余的XMVECTOR参数应当用类型 CXMVECTOR；  
    

```
//在32位的windows系统上，编译器将根据_fastcall调用约定将前3个// XMVECTOR参数传递到寄存器中，而把其余参数都存在栈上
typedef const XMVECTOR FXMVECTOR;
typedef const XMVECTOR& GXMVECTOR;
typedef const XMVECTOR& HXMVECTOR;
typedef const XMVECTOR& CXMVECTOR;

//在32位的 windows系统上，编译器将通过_vectorcall调用约定将前6个/ /XMVECTOR参数传递到寄存器中,而把其余参数均存在栈上
typedef const XMVECTOR FXMVECTOR;
typedef const XMVECTOR GXMVECTOR;
typedef const XMVECTOR HXMVECTOR;
typedef const XMVECTOR& CXMVECTOR;

``` 

以上规则仅适用于“输入”参数。  

## 常向量  

`` `XMVECTOR` ``类型的常量实例用`` `XMVECTOR32` ``类型表示，`` `XMVECTORF32` ``是一种按16字节对齐的结构体。  

## 重载运算符  

XMVECTOR类型针对向量的加法运算、减法运算和标量乘法运算，都分别提供了对应的重载运算符。  
```c++ nums

```
```
XMVECTOR  XM_CALLCONV   operator+ (FXMVECTOR V);
XMVECTOR  XM_CALLCONV   operator- (FXMVECTOR V);

XMVECTOR&  XM_CALLCONV   operator+= (XMVECTOR& V1, FXMVECTOR V2);
XMVECTOR&  XM_CALLCONV   operator-= (XMVECTOR& V1, FXMVECTOR V2);
XMVECTOR&  XM_CALLCONV   operator*= (XMVECTOR& V1, FXMVECTOR V2);
XMVECTOR&  XM_CALLCONV   operator/= (XMVECTOR& V1, FXMVECTOR V2);

XMVECTOR&  operator*= (XMVECTOR& V, float S);
XMVECTOR&  operator/= (XMVECTOR& V, float S);

XMVECTOR  XM_CALLCONV   operator+ (FXMVECTOR V1, FXMVECTOR V2);
XMVECTOR  XM_CALLCONV   operator- (FXMVECTOR V1, FXMVECTOR V2);
XMVECTOR  XM_CALLCONV   operator* (FXMVECTOR V1, FXMVECTOR V2);
XMVECTOR  XM_CALLCONV   operator/ (FXMVECTOR V1, FXMVECTOR V2);
XMVECTOR  XM_CALLCONV   operator* (FXMVECTOR V, float S);
XMVECTOR  XM_CALLCONV   operator* (float S, FXMVECTOR V);
XMVECTOR  XM_CALLCONV   operator/ (FXMVECTOR V, float S);

``` 

## 杂项  

DirectXMath库定义了一组与 π 有关的常用数学常量近似值：  

```
const float XM_PI    =   3.141592654f;
const float XM_2PI   =   6.283185307f;
const float XM_1DIVPI    =  0.318309886f;
const float XM_1DIV2PI   =  0.159154943f;
const float XM_PIDIV2    =  1.570796327f;
const float XM_PIDIV4    =  0.785398163f;

``` 

下列内联函数实现了弧度和角度间的互相转化：  

```
inline float XMConvertToRadians(float fDegrees)
{ return fDegrees * (XM_PI / 180.0f); }
inline float XMConvertToDegrees(float fRadians)
{ return fRadians * (180.0f / XM_PI); }

``` 

DirectXMath库还定义了求出两个数间较大值及较小值的函数：  

```
template inline T XMMin(T a, T b) { return (a < b) ? a : b; }
template inline T XMMax(T a, T b) { return (a > b) ? a : b; } 
``` 

## Setter函数  

DirectXMath库提供了下列函数，以设置 XMVECTOR 类型中的数据：  

```
// 返回零向量0
XMVECTOR XM_CALLCONV XMVectorZero();

// 返回向量(1, 1, 1, 1)
XMVECTOR XM_CALLCONV XMVectorSplatOne();

// 返回向量(x, y, z, w)
XMVECTOR XM_CALLCONV XMVectorSet(float x, float y, float z, float w);

// 返回向量(Value, Value, Value, Value)
XMVECTOR XM_CALLCONV XMVectorReplicate(float Value);

// 返回向量(vx, vx, vx, vx) 
XMVECTOR XM_CALLCONV XMVectorSplatX(FXMVECTOR V);

// 返回向量(vy, vy, vy, vy) 
XMVECTOR XM_CALLCONV XMVectorSplatY(FXMVECTOR V);

// 返回向量(vz, vz, vz, vz) 
XMVECTOR XM_CALLCONV XMVectorSplatZ(FXMVECTOR V);

``` 

## 向量函数  

DirectXMath库提供了下面的函数来执行各种向量运算。我们主要围绕3D向量的运算函数进行讲解，类似的运算还有2D和4D版本。除了表示维度的数字不同以外，这几种版本的函数名皆同。  

```
XMVECTOR XM_CALLCONV XMVector3Length(      // 返回||v||
  FXMVECTOR V);                            // 输入向量v

XMVECTOR XM_CALLCONV XMVector3LengthSq(    //返回||v||2
  FXMVECTOR V);                            // 输入向量v

XMVECTOR XM_CALLCONV XMVector3Dot(         // 返回v1·v２
  FXMVECTOR V1,                            // 输入向量v1
  FXMVECTOR V2);                           // 输入向量v2

XMVECTOR XM_CALLCONV XMVector3Cross(       // 返回v1×v2
  FXMVECTOR V1,                            // 输入向量v1
  FXMVECTOR V2);                           // 输入向量v2

XMVECTOR XM_CALLCONV XMVector3Normalize(   // 返回v/||v||
  FXMVECTOR V);                            // 输入向量v

XMVECTOR XM_CALLCONV XMVector3Orthogonal(  // 返回一个正交于v的向量
  FXMVECTOR V);                            // 输入向量v

XMVECTOR XM_CALLCONV XMVector3AngleBetweenVectors( // 返回v1和v2之间的夹角
  FXMVECTOR V1,                           // 输入向量v1
  FXMVECTOR V2);                          // 输入向量v2

void XM_CALLCONV XMVector3ComponentsFromNormal(
  XMVECTOR* pParallel,                   // 返回projn(v)
  XMVECTOR* pPerpendicular,              // 返回perpn(v)
  FXMVECTOR V,                           // 输入向量v
  FXMVECTOR Normal);                     // 输入规范化向量n

bool XM_CALLCONV XMVector3Equal(     // 返回v1 == v2？
  FXMVECTOR V1,                      // 输入向量v1
  FXMVECTOR V2);                     // 输入向量v2

bool XM_CALLCONV XMVector3NotEqual(  // 返回v1≠v２
  FXMVECTOR V1,                      // 输入向量v1
  FXMVECTOR V2);                     // 输入向量v2

``` 

注意： 可以看到，即使在数学上计算的结果是标量，函数所返回的类型依旧是`` `XMVECTOR` ``，而得到的标量结果则被复制到`` `XMVECTOR` ``中的各个分量之中。（如点积函数 XMVector3Dot ，此函数返回的向量为（v1 v2 , v1 v2 , v1 v2 , v1 v2）仍然是贯彻上面说的，将标量和SIMD向量的混合运算次数降到最低，使用户除了自定义的计算之外全程都使用SIMD技术，以提升计算效率。  

DirectXMath库也提供了一些估算方法，精度低但速度快。如果愿意为了速度而牺牲一些精度，则可以使用它们。下面是两个估算方法的例子：  

```
XMVECTOR XM_CALLCONV XMVector3LengthEst(     // 返回估算值||v||
  FXMVECTOR V);                              // 输入v
　
XMVECTOR XM_CALLCONV XMVector3NormalizeEst(  // 返回估算值v/||v||
  FXMVECTOR V);                             // 输入v

``` 

## 浮点数误差  

DirectXMath库提供了XMVector3NearEqual函数，用于以Epsilon作为容差，测试比较的向量是否相等：  

```
// 返回 
//  abs(U.x – V.x) <= Epsilon.x && 
//  abs(U.y – V.y) <= Epsilon.y &&
//  abs(U.z – V.z) <= Epsilon.z
XMFINLINE bool XM_CALLCONV XMVector3NearEqual(
  FXMVECTOR U, 
  FXMVECTOR V, 
  FXMVECTOR Epsilon);

``` 

# 矩阵  

## 矩阵类型  

```
#if (defined(_M_IX86) || defined (_M _X64) l| defined(_M_ ARM)) && defined &&(_XM_NO_INTRINSICS_)
struct XMMATRIX
#else
_declspec (align (16))struct XMMATRIX
#endif
{
	//利用4个XMVECTOR来表示矩阵，借此使用SIMD技术
	XMVECTOR r[ 4];
	
	XMMATRIX(){}
	
	//通过指定4个行向量来初始化矩阵
	XMMATRIX （FXMVECTOR RO，FXMVECTOR Rl, FXMVECTOR-R2，CXMVECTOR R3){ r[0]= R0; r[1] =Rl; r[2] =R2; r[3]=R3;}
	
	//通过指定16个矩阵元素来初始化矩阵
	XMMATRIX(float m00,float m01, float m02， float m03,
	float ml0,float m11,float m12,float m13,
	float m20,float m21，float m22,float m23,float m30,float m31,float m32,float m33);
	
	//通过含有16个浮点数元素的数组来初始化矩阵
	explicit XMMATRIX(_In_reads_(16) const float *pArray);
	XMMATRIX&operator= ( const XMMATRIX& M)
	{ r[0] = M.r[0]; r[1] = M.r[1]; r[2] = M.r [2]; r[3] = M.r [ 3] ;return *this; }
	XMMATRIXoperator+ () const { return *this; }
	XMMATRIXoperator- () const;
	
	XMMATRIX& XM CALLCONVoperator+= (FXMMATRIX M);
	XMMATRIX& XM CALLCONVoperator-= (FXMMATRIX M);
	XMMATRIX& XM CALLCONVoperator*= (FXMMATRIX M);
	XMMATRIX&operator*=(float s);
	XMMATRIX&operator/=(float s);
	
	XMMATRIX XM CALLCONV operator+ (FXMMATRIX M) const;
	XMMATRIX XM CALLCONV operator- (FXMMATRIX M) const;
	XMMATRIX XM CALLCONVoperator*(FXMMATRIX M) const;
	XMMATRIXoperator*(float s) const;
	XMMATRIXoperator/(float s) const;
	
	friend XMMATRIX XM CALLCONVoperator* (float S，FXMMATRIX M);
};

``` 

除了各种构造方法之外，还可以使用XMMatrixSet函数来创建XMMATRIX实例：  

```
XMMATRIX XM_CALLCONV XMMatrixSet(
  float m00, float m01, float m02, float m03,
  float m10, float m11, float m12, float m13,
  float m20, float m21, float m22, float m23,
  float m30, float m31, float m32, float m33);

``` 

就像通过`` `XMFLOAT2` ``(2D)，`` `XMFLOAT3` ``(3D)和`` `XMFLOAT4` ``(4D)来存储类中不同维度的向量一样，DirectXMath文档也建议我们用`` `XMFLOAT4X4` ``来存储类中的矩阵类型数据成员。  

```
struct XMFLOAT4X4
{
  union
  {
    struct
    {
      float _11, _12, _13, _14;
      float _21, _22, _23, _24;
      float _31, _32, _33, _34;
      float _41, _42, _43, _44;
    };
    float m[4][4];
  };
  XMFLOAT4X4() {}
  XMFLOAT4X4(float m00, float m01, float m02, float m03,
             float m10, float m11, float m12, float m13,
             float m20, float m21, float m22, float m23,
             float m30, float m31, float m32, float m33);
  explicit XMFLOAT4X4(_In_reads_(16) const float *pArray);

  float    operator() (size_t Row, size_t Column) const { return m[Row][Column]; }
  float&   operator() (size_t Row, size_t Column) { return m[Row][Column]; }

  XMFLOAT4X4& operator=(const XMFLOAT4X4& Float4x4);
};

``` 

通过下列方法将数据从`` `XMFLOAT4X4` ``内加载到`` `XMMATRIX` ``中：  

```
inline XMMATRIX XM_CALLCONV 
XMLoadFloat4x4(const XMFLOAT4X4* pSource);

``` 

通过下列方法将数据从`` `XMMATRIX` ``内存储到`` `XMFLOAT4X4` ``中：  

```
inline void XM_CALLCONV 
XMStoreFloat4x4(XMFLOAT4X4* pDestination, FXMMATRIX M);

``` 

## 矩阵函数  

```
XMMATRIX XM_CALLCONV XMMatrixIdentity();   // 返回单位矩阵I

bool XM_CALLCONV XMMatrixIsIdentity(       // 如果M是单位矩阵则返回true
    FXMMATRIX M);                          // 输入矩阵M

XMMATRIX XM_CALLCONV XMMatrixMultiply(     // 返回矩阵乘积AB
    FXMMATRIX A,                           // 输入矩阵A
    CXMMATRIX B);                          // 输入矩阵B

XMMATRIX XM_CALLCONV XMMatrixTranspose(    // 返回MT
    FXMMATRIX M);                          // 输入矩阵M

XMVECTOR XM_CALLCONV XMMatrixDeterminant(  // 返回(det M, det M, det M, det M)
    FXMMATRIX M);                          // 输入矩阵M

XMMATRIX XM_CALLCONV XMMatrixInverse(      // 返回M-1
    XMVECTOR* pDeterminant,                // 输入(det M, det M, det M, det M)
    FXMMATRIX M);                          // 输入矩阵M

``` 

在声明具有XMMATRIX参数的函数时，除了要注意1个XMMATRIX应计作4个XMVECTOR参数这一点之外，其他的规则与传入XMVECTOR类型的参数时相一致。假设传入函数的FXMVECTOR参数不超过两个，则第一个XMMATRIX参数应当为FXMMATRIX类型，其余的XMMATRIX参数均应为CXMMATRIX类型。  

```
// 在32位的Windows系统上，__fastcall调用约定通过寄存器传递前3个XMVECTOR参数，其余的
//参数则存在堆栈上
typedef const XMMATRIX& FXMMATRIX;
typedef const XMMATRIX& CXMMATRIX;

// 在32位的Windows系统上，__vectorcall调用约定通过寄存器传递前 6个 XMVECTOR实参，其余的
// 参数则存在堆栈上
typedef const XMMATRIX FXMMATRIX;
typedef const XMMATRIX& CXMMATRIX;

``` 

可以看出，在32位Windows操作系统上的__fastcall调用约定中，XMMATRIX类型的参数是不能传至SSE/SSE2寄存器的，因为这些寄存器此时只支持3个XMVECTOR参数传入。而XMMATRIX参数却是由4个XMVECTOR构成，所以矩阵类型的数据只能通过堆栈来加以引用。  

# 变换  

## 变换函数  

```
// 构建一个缩放矩阵:
XMMATRIX XM_CALLCONV XMMatrixScaling(
float ScaleX, 
float ScaleY, 
float ScaleZ);                        // 缩放系数

// 用一个3D向量中的分量来构建缩放矩阵:
XMMATRIX XM_CALLCONV XMMatrixScalingFromVector(
FXMVECTOR Scale);                     // 缩放系数(sx,sy, sz)

// 构建一个绕x轴旋转的矩阵Rx:
XMMATRIX XM_CALLCONV XMMatrixRotationX(
    float Angle);                    // 以顺时针方向按弧度θ进行旋转

// 构建一个绕y轴旋转的矩阵Ry:
XMMATRIX XM_CALLCONV XMMatrixRotationY(
    float Angle);                    // 以顺时针方向按弧度θ进行旋转

// 构建一个绕z轴旋转的矩阵Rz:
XMMATRIX XM_CALLCONV XMMatrixRotationZ(
    float Angle);                    // 以顺时针方向按弧度θ进行旋转

// 构建一个绕任意轴旋转的矩阵Rn: 
XMMATRIX XM_CALLCONV XMMatrixRotationAxis(
 FXMVECTOR Axis,                      // 旋转轴n    
 float Angle);                        // 沿n轴正方向看，以顺时针方向按弧度θ进行旋转
// 构建一个平移矩阵:
XMMATRIX XM_CALLCONV XMMatrixTranslation(
float OffsetX, 
float OffsetY, 
float OffsetZ);                       // 平移系数

// 用一个3D向量中的分量来构建平移矩阵:
XMMATRIX XM_CALLCONV XMMatrixTranslationFromVector(
FXMVECTOR Offset);                    // 平移系数(tx, ty, tz)

// 计算向量与矩阵的乘积vM，此函数为针对点的变换，即总是默认令vw = 1:
XMVECTOR XM_CALLCONV XMVector3TransformCoord(
FXMVECTOR V,      // 输入向量v
CXMMATRIX M);     // 输入矩阵M

// 计算向量与矩阵的乘积vM，此函数为针对向量的变换，即总是默认令vw = 0:
XMVECTOR XM_CALLCONV XMVector3TransformNormal(
FXMVECTOR V,       // 输入向量v 
CXMMATRIX M);      // 输入矩阵M

``` 

其中，缩放、平移和旋转对应的变换矩阵如下：  

![](/api/filetransfer/images?url=https%3A%2F%2Fddddyx.oss-cn-beijing.aliyuncs.com%2Fimg%2F202208171924540.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_19%252Ctext_dGhldHVz%252Ccolor_FFFFFF%252Cshadow_50%252Ct_80%252Cg_se%252Cx_10%252Cy_10&sign=a50595fb21254e439688f08aa86bff29035865228bedab94847affc478ed0d7c)