# github桌面端
## 新建仓库
File->New repository
![[Pasted image 20230108234658.png]]
![[Pasted image 20230108235440.png]]

File->add local repository
![[Pasted image 20230108234752.png]]
## 内容更改
**克隆github仓库到本地进行内容更改：**
1. File->clone repository
2. Commit
![[Pasted image 20230108225909.png]]
3. publish
![[Pasted image 20230108230142.png]]
## 多人协作
进行多人协作时，如何更新项目？

Repository->pull

## 分支
Branch->New branch
![[Pasted image 20230108234447.png]]
切换分支，点击Fetch origin上传即可

# .gitignore忽略文件
## .gitignore文件
提交之前，在目录中创建.gitignore
```c
touch  .gitignore     #创建gitignore隱藏文件  
```

示例：这个文件每一行保存了一个匹配的规则
```c
# 此为注释 – 将被 Git 忽略
*.a       # 忽略所有 .a 结尾的文件
*a        # 忽略所有名字为a的文件
!lib.a    # 但 lib.a 除外
/TODO     # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/    # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```

文件.gitignore的格式规范：
A：#为注释   
B：可以使用shell所使用的正则表达式来进行模式匹配   
C：匹配模式最后跟"/"说明要忽略的是目录 
D：使用！取反（例如目录中包含  test.a，并且gitignore文件中包含  * .[oa]，如果在文件中加入 ！test.a   表明忽略除test.a文件以外的后缀名为.a或者.o的文件）
## git rm -r --cached
有时候在项目开发过程中，想把某些目录或文件加入忽略规则，按照上述方法定义后发现并未生效，原因是.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态）
```c++
git rm -r --cached 文件/文件夹名字
```
去掉已经托管的文件或者文件夹，然后提交即可。

## gitignoreglobal全局忽略文件
另外 git 提供了一个全局的 .gitignore，你可以在你的用户目录下创建 ~/.gitignoreglobal 文件，以同样的规则来划定哪些文件是不需要版本控制的。

需要执行 git config --global core.excludesfile ~/.gitignoreglobal来使得它生效。

