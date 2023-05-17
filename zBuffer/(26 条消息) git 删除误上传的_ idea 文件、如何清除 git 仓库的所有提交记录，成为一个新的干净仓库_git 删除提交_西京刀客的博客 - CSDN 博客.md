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

注意这里强推 报错： Failed to connect to github.com port 443: Timed out  
使用 git bash 窗口试试，亲测可以！