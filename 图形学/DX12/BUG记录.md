# 1 引擎开发环境部署
## 子系统设置
正常显示窗口需要设置子系统为窗口：
![[Pasted image 20230404110549.png]]

## WinMain 批注不一致
![[Pasted image 20230403225746.png]]
解决办法一：解决方案->右键属性，将子系统改为窗口
![[Pasted image 20230403225818.png|500]]
方法二：添加批注（成功）
```c++ nums
int WINAPI
WinMain(_In_ HINSTANCE  hInstance, _In_opt_ HINSTANCE hPrevInstance, _In_ PSTR pCmdLine, _In_ int nCmdShow)
```
## 日志的堆栈保留设置
由于使用日志系统导致内存不足
![[Pasted image 20230405103536.png]]
解决方法：将堆栈保留大小调高：
![[Pasted image 20230405103842.png]]
