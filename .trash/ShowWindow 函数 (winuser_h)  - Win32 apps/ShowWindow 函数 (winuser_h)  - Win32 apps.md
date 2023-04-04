设置指定窗口的显示状态。

[](#syntax)

## 语法

```
BOOL ShowWindow( [in] HWND hWnd,
  [in] int  nCmdShow );
```

[](#parameters)

## 参数

`[in] hWnd`

类型：**HWND**

窗口的句柄。

`[in] nCmdShow`

类型： **int**

控制窗口的显示方式。 如果启动应用程序的程序提供 [STARTUPINFO](/zh-cn/windows/desktop/api/processthreadsapi/ns-processthreadsapi-startupinfoa) 结构，则首次调用 **ShowWindow** 时忽略此参数。 否则，首次调用 **ShowWindow** 时，该值应该是 [WinMain](/zh-cn/windows/desktop/api/winbase/nf-winbase-winmain) 函数在其 _nCmdShow_ 参数中获取的值。 在后续调用中，此参数可以是以下值之一。


## 返回值

类型： **BOOL**

如果窗口以前可见，则返回值为非零。

如果窗口之前已隐藏，则返回值为零。

[](#remarks)

## 注解

若要在显示或隐藏窗口时执行某些特殊效果，请使用 [AnimateWindow](/zh-cn/windows/desktop/api/winuser/nf-winuser-animatewindow)。

应用程序首次调用 **ShowWindow** 时，应使用 [WinMain](/zh-cn/windows/desktop/api/winbase/nf-winbase-winmain) 函数的 _nCmdShow_ 参数作为其 _nCmdShow_ 参数。 对 **ShowWindow** 的后续调用必须使用给定列表中的某个值，而不是 **WinMain** 函数的 _nCmdShow_ 参数指定的值。

如 _nCmdShow_ 参数的讨论中所述，如果启动应用程序的程序在结构中指定启动信息的程序，则**对 ShowWindow** 的第一次调用中忽略 _nCmdShow_ 值。 在这种情况下， **ShowWindow** 使用 [STARTUPINFO](/zh-cn/windows/desktop/api/processthreadsapi/ns-processthreadsapi-startupinfoa) 结构中指定的信息来显示窗口。 在后续调用中，应用程序必须调用 **ShowWindow** ，并将 _nCmdShow_ 设置为 **SW_SHOWDEFAULT** 才能使用启动应用程序的程序提供的启动信息。 此行为适用于以下情况：

*   应用程序通过调用具有 **WS_VISIBLE** 标志集的 [CreateWindow](/zh-cn/windows/desktop/api/winuser/nf-winuser-createwindowa) 来创建主窗口。
*   应用程序通过调用已清除 **WS_VISIBLE** 标志的 [CreateWindow](/zh-cn/windows/desktop/api/winuser/nf-winuser-createwindowa) 来创建主窗口，稍后使用设置为 **SW_SHOW 标志来**调用 **ShowWindow**，使其可见。

[](#examples)

#### 示例

有关示例，请参阅 [“创建主窗口](/zh-cn/windows/desktop/winmsg/using-windows)”。

[](#requirements)

## 要求

<table aria-label="表 2"><thead><tr><th>&nbsp;</th><th>&nbsp;</th></tr></thead><tbody><tr><td><strong>最低受支持的客户端</strong></td><td>Windows 2000 Professional [仅限桌面应用]</td></tr><tr><td><strong>最低受支持的服务器</strong></td><td>Windows 2000 Server [仅限桌面应用]</td></tr><tr><td><strong>目标平台</strong></td><td>Windows</td></tr><tr><td><strong>标头</strong></td><td>winuser.h (包括 Windows.h)</td></tr><tr><td><strong>Library</strong></td><td>User32.lib</td></tr><tr><td><strong>DLL</strong></td><td>User32.dll</td></tr><tr><td><strong>API 集</strong></td><td>在 Windows 8) 中引入的 ext-ms-win-ntuser-window-l1-1-0 (</td></tr></tbody></table>

[](#see-also)

## 另请参阅

[AnimateWindow](/zh-cn/windows/desktop/api/winuser/nf-winuser-animatewindow)

**概念性**

[CreateProcess](/zh-cn/windows/desktop/api/processthreadsapi/nf-processthreadsapi-createprocessa)

[CreateWindow](/zh-cn/windows/desktop/api/winuser/nf-winuser-createwindowa)

**其他资源**

**参考**

[STARTUPINFO](/zh-cn/windows/desktop/api/processthreadsapi/ns-processthreadsapi-startupinfoa)

[ShowOwnedPopups](/zh-cn/windows/desktop/api/winuser/nf-winuser-showownedpopups)

[ShowWindowAsync](/zh-cn/windows/desktop/api/winuser/nf-winuser-showwindowasync)

[WinMain](/zh-cn/windows/desktop/api/winbase/nf-winbase-winmain)

[Windows](/zh-cn/windows/desktop/winmsg/windows)