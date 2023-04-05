# Win32
## WinMain 批注不一致
![[Pasted image 20230403225746.png]]
解决办法一：解决方案->右键属性，将子系统改为窗口
![[Pasted image 20230403225818.png|500]]
方法二：添加批注
```c++ nums
int WINAPI
WinMain(_In_ HINSTANCE  hInstance, _In_opt_ HINSTANCE hPrevInstance, _In_ PSTR pCmdLine, _In_ int nCmdShow)
```
