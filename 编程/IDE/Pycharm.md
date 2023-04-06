# 代码格式化插件 autopep8
1. 安装 autopep8
```c++ nums
// cmd输入
pip install autopep8
```

2. 设置 external tools
【File】→【Settings】→【Tools】→【External Tools】→【+】，如下图

![](1680783205920.png)

点击完成后，会得到如下图界面  

![](1680783205961.png)

填写如下：  
```text
Name: AutoPep8
Description: autopep8 your code
Program: autopep8
Arguments: --in-place --aggressive --aggressive $FilePath$
Working directory: $ProjectFileDir$
Output filters: $FILE_PATH$\:$LINE$\:$COLUMN$\:.*
```

填写完后如下图  

![](1680783206017.png)

填写完后，点击【OK】→【Apply】→【OK】  
此时，设置已经完毕，已经可以在编辑 py 文件时，点击鼠标右键，选择【External Tools】→【autopep8】，如下图  

![](1680783206100.png)

3. 快捷键设置

为了更方便调用这个外部工具，可以设置快捷键，点击【Files】→【Settings】→【Keymap】→【External Tools】，会出现如下图

![](1680783206197.png)

  
然后双击【aotupep8】，会出现如下图

![](1680783206236.png)

输入自己想设置的快捷键，但注意不能与其他快捷键重复。设置完毕后，在敲代码时，便可通过快捷键优化代码格式。

# 自动换行
![[Pasted image 20230406202829.png]]