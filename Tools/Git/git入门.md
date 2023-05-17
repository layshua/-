# 基本操作
查看git版本
```c++
git --version
```
更新git
```c++
git update-git-for-windows
```

```c
//git ssh流程
ssh-keygen -t rsa -C "226256089@qq.com" //三次回车生成公钥，注意这行不同平台有差异
cat ~/.ssh/id_rsa.pub //复制公钥并设置
```

```c
git init 
git add --all
git commit -m "first commit"
git remote add origin git@codeup.aliyun.com:63a5d66f8d9a873a30aae912/TA101.git
git push -u origin "master"

//关于git remote add origin 后面跟的是仓库ssh名称
git remote   //查看有哪些remote，一般就一个，叫做origin
git remote remove origin   //删除，一般不用到，改变仓库ssh时要使用
git remote add origin xxx //新增，一般不用到。xxx 请用最开始的 git clone xxx 中的 xxx 替代


```

```c
//查看当前用户（global）配置
git config --global  --list

//查看当前用户名、邮箱、密码
git config user.name
git config user.email
git config user.password

//修改用户名、邮箱、密码方式：
git config --global user.name "xxx(新的用户名)"
git config --global user.email "123456@163.com(新的邮箱)"
git config --global user.password "123456(新的密码)"

```
# 分支操作
**分支覆盖 master**
比如有一个 dev 分支进行了多次迭代，但是 master 没有及时更新，需要使用 dev 分支来取代 master。
```c++ nums
//切换到 dev 分支: 
git checkout dev

//删除 master 分支: 
git branch -D master

//将 dev 分支复制并创建为 master: 
git checkout -b master

//推送到远程: 
git push -u origin master --force
```

**把远程分支拉到本地**
```c++ nums
git fetch origin xxx（xxx为远程仓库的分支名）
```

**在本地创建分支 dev 并切换到该分支**
```c++ nums
git checkout -b xxx(本地分支名称) origin/xxx(远程分支名称)
```

**远程分支上的内容都拉取到本地**
```c++ nums
git pull origin xxx(远程分支名称)
```

**1、git checkout <分支名>**

切换到目标分支，比如切换到 test 分支：git checkout test

同时可以用 git checkout . 直接回到当前分支的最新 commit 代码，同时放弃未提交更改，但是这条命令在合并分支的时候请慎用。

**2、git checkout -b <分支名>**

从当前分支创建并切换到一个新的分支，比如创建并切换到 test 分支：

git checkout -b test

**3、git push --up-stream  origin <分支名>**

把一个远程没有的分支推送到远程仓库，比如新创建的 test 分支

git push --up-stream origin test

**4、git branch -D <分支名>**

删除本地分支，例如删除本地 test 分支

git branch -D test

**5、git push origin --delete <分支名>**

删除远程分支，仍然以删除远程 test 分支为例
git push origin --delete test
# commit 操作

## 删除本地commit
1. 首先使用 git log 命令查看提交记录，找到出错的那一笔提交的 commit_id (黄色)  

![](<images/1683810435617.png>)

2，用命令 git [rebase]( https://so.csdn.net/so/search?q=rebase&spm=1001.2101.3001.7020 "rebase") -i commit_id , 查找提交记录  
 

```
git rebase -i 15774a44d46bcd0c055b07c63bd0ecbe35a9660b
```

  
3. 执行 (2) 命令后出现如下界面：  

![](<images/1683810435660.png>)

 4. 输入 i 进入编辑模式，在要删除的 commitid 前，将 pick 修改成 drop。看到的结果如下图：

![](<images/1683810435699.png>)

5. 编辑完成，按键盘 Esc，退出编辑模式，然后按 Shift+; 再输入 wq!（保存文件的写入修改）退出。（q! 是不保存修改）

![](<images/1683810435789.png>)
  
6. 这时候本地删除已经完成了，然后推送到远端 git push --force
## 回退到之前的 commit

如果只误提交了一个 commit，直接执行

```
git reset --hard HEAD^
```

这一条命令就 OK 了（但是这条命令会把最近一次 commit 的代码删除哦，如果要保留代码，请看本文最后面部分）

如果你要回滚多个 commit 可以使用下面的方法：

**第一步：通过 git log 可以查看我们之前提交的 commit_id：**

![](<images/1683810912550.png>)

**第二步：复制你需要回滚的 commit_id。不过 windows 下的命令行是不能复制文本的，为此你可以到 github 上查看仓库的 commit 历史，上面是可以直接复制的：**

![](<images/1683810912595.png>)

复制好后在命令行执行：git [reset](https://so.csdn.net/so/search?q=reset&spm=1001.2101.3001.7020) --hard <commit_id>

```
git reset --hard 4458e09
```

**第三步：执行: git push origin HEAD --force 提交当前 HEAD**

这样就能够将错误的提交删除，回滚至其之前的代码

## 误删 commit 后的恢复方法

**第一步：执行 git reflog**

   你能在命令行中看到你的历史操作，复制你要恢复操作最前面的 hash 值

![](<images/1683810912627.png>)

**第二步：执行 `git reset --hard <hash>` 

   命令中最后要输的 hash 就是你要恢复的历史记录最前面黄色部分的 hash 值

   比如：
    
```
git reset --hard 4458e09
```

这样就成功解决问题了。

**但是上面的操作仍然有缺陷，我们经常遇到的情况是：在错误的分支写了大量代码，并且已经 push 到远程仓库，这个时候我们要的不仅是回退 commit，并且要保留自己新写的代码并且合并到正确的分支上。**

接下来不要眨眼，我们来演示如何解决问题：

**第一步：找到你提交之前的最新 commit id，执行如下命令：**

```
git reset --soft 4458e09
```

这条命令会把你本地的当前分支回退到之前的 commit，并且保留你在这条 commit 之后写的代码。

**第二步：执行 git stash 将新写的代码放到缓存中**

```
git stash
```

**第三步：把当前分支强行覆盖远程分支**

```
git push --force
```

**第四步：切换到正确的开发分支，这里假设是 dev 分支**

```
git checkout dev
```

**第五步：把缓存的代码从内存中 pop 出来**

```
git stash pop
```

执行完这五步，当前分支的错误提交就清理了，并且你的代码也回到了正确的 dev 分支。

但是，需要注意，这样的操作必须保证在你的错误提交之后别人没有提交代码，否则这样操作将会把别人正常提交的代码回退掉。

## git reset 的相关参数

**--mixed**

意思是不删除工作空间改动代码，撤销 commit ，并且撤销 git add . 操作

这个是 reset 命令的默认参数，即 git reset —mixed HEAD^ 和 git reset HEAD^ 效果是一样的

**--soft**

不删除工作空间改动代码，撤销 commit， 撤销 git add .

**--hard**

删除工作空间代码，撤销 commit，撤销 git add .

注意，添加该参数后执行后，你最新提交的代码会从内存中删除

当然如果误删了可以通过上面的方法找回来。


# git 删除误上传的. idea 文件

提交项目的时候忘记添加. gitignore 文件，误上传了文件夹 (如. idea) 如何解决？

1.  pycharm 安装 .ignore 插件
    
2.  项目跟目录下创建 .ignore 文件  
    以. idea 文件夹举例，如下排除 .idea/
    
    ```
    # Project exclude paths
    /venv/
    .idea/
    ```
    
3.  打开项目终端执行命令  
    删除 [github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020) 上的误上传文件. idea
    
    ```
    git rm -rf --cached .idea
    ```
    
4.  提交. gitignore 文件
    
    ```
    git add .gitignore
    ```
    
5.  上传
    
    ```
    git commit -m '忽略idea'
    git push -u origin master
    ```
    

# 如何清除 git 仓库的所有提交记录，成为一个新的干净仓库

需求：  
提交代码的时候在 Log 中上传了一些比较敏感的信息，这些信息都可以在 Github 上面搜索到，想把这些 Log 信息清除掉，使其变成一个没有提交记录的 “新仓库”。

解决方案：  
使用 git –orphan 清理 git 历史

操作步骤：  
1） 创建新分支  
git checkout --orphan newBranch

2.  添加所有文件 (除了. gitignore 中声明排除的)  
    git add -A
    
3.  提交跟踪过的文件（Commit the changes）  
    git commit -am “init commit”
    

4) 删除 master 分支  
git branch -D master

5.  重命名当前分支为 master  
    git branch -m master
    
6.  强制提交到远程 master 分支  
    git push -f origin master
    

通过以上几步就可以简单地把一个 Git 仓库的历史提交记录清除掉了

注意这里强推报错： Failed to connect to github. com port 443: Timed out  
使用 git bash 窗口试试，亲测可以！