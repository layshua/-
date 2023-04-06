### PyCharm 代码格式化插件

*   [前言](#_1)
*   [1. 安装 autopep8](#1autopep8_3)
*   [2. 设置 external tools](#2external_tools_6)
*   [3. 快捷键设置](#3_24)

# 前言

作为新手，可能对代码格式不会太注重，但其对工作而言又是必不可少的，再此，向各位看官推荐 PyCharm 代码格式化工具 aotupep8，其主要步骤如下：

# 1. 安装 autopep8

本机已安装，如下图  

![](<assets/1680783205849.png>)

# 2. 设置 external tools

按下列顺序点击  
【File】→【Settings】→【Tools】→【External Tools】→【+】，如下图

![](<assets/1680783205920.png>)

  
点击完成后，会得到如下图界面  

![](<assets/1680783205961.png>)

  
填写如下：  
【Name】：aotupep8  
【Program】：autopep8.exe 存放路径  
【Arguments】：–in-place --aggressive --aggressive F i l e P a t h FilePath FilePath  
【Working directory】： P r o j e c t F i l e D i r ProjectFileDir ProjectFileDir  
【Output filters】： F I L E P A T H FILE_PATH FILEP​ATH: L I N E LINE LINE: C O L U M N COLUMN COLUMN:.*  
填写完后如下图  

![](<assets/1680783206017.png>)

  
填写完后，点击【OK】→【Apply】→【OK】  
此时，设置已经完毕，已经可以在编辑 py 文件时，点击鼠标右键，选择【External Tools】→【autopep8】，如下图  

![](<assets/1680783206100.png>)

  
点击之后，就会变成如下格式：  

![](<assets/1680783206149.png>)

# 3. 快捷键设置

为了更方便调用这个外部工具，可以设置快捷键，点击【Files】→【Settings】→【Keymap】→【External Tools】，会出现如下图

![](<assets/1680783206197.png>)

  
然后双击【aotupep8】，会出现如下图

![](<assets/1680783206236.png>)

  
输入自己想设置的快捷键，但注意不能与其他快捷键重复。设置完毕后，在敲代码时，便可通过快捷键优化代码格式。