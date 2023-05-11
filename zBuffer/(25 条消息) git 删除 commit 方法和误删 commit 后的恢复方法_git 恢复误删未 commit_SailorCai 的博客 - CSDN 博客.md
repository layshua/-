### 一、如果你在 git 上提交了错误的 [commit](https://so.csdn.net/so/search?q=commit&spm=1001.2101.3001.7020)，不要慌，通过下面的方法可以回退到之前的 commit

如果只误提交了一个 commit， 直接执行

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

**但是如果当你回滚代码以后发现 commit_id 复制错了或者回滚错了怎么办呢？不要慌！！！按下面的方法做：**

**第一步：执行 git reflog**

    你能在命令行中看到你的历史操作，复制你要恢复操作最前面的 hash 值

![](<images/1683810912627.png>)

**第二步：执行 git reset --hard <hash>**

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

### 二、再补一下 git reset 的相关参数：

**--mixed**

意思是不删除工作空间改动代码，撤销 commit ，并且撤销 git add . 操作

这个是 reset 命令的默认参数，即 git reset —mixed HEAD^ 和 git reset HEAD^ 效果是一样的

**--soft**

不删除工作空间改动代码，撤销 commit， 撤销 git add .

**--hard**

删除工作空间代码，撤销 commit，撤销 git add .

注意，添加该参数后执行后，你最新提交的代码会从内存中删除

当然如果误删了可以通过上面的方法找回来。

### **三、最后再补充一些常用的 git 命令**

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