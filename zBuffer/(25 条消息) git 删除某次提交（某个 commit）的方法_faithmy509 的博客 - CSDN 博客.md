例如我的提交历史如下

```
commit 58211e7a5da5e74171e90d8b90b2f00881a48d3a
Author: test <test@36nu.com>
Date:   Fri Sep 22 20:55:38 2017 +0800
 
    add d.txt
 
commit 0fb295fe0e0276f0c81df61c4fd853b7a000bb5c
Author: test <test@36nu.com>
Date:   Fri Sep 22 20:32:45 2017 +0800
 
    add c.txt
 
commit 7753f40d892a8e0d14176a42f6e12ae0179a3210
Author: test <test@36nu.com>
Date:   Fri Sep 22 20:31:39 2017 +0800
 
    init
```

假如要删除备注为`add c.txt` [commit](https://so.csdn.net/so/search?q=commit&spm=1001.2101.3001.7020) 为`0fb295fe0e0276f0c81df61c4fd853b7a000bb5c`的这次提交

1.  首先找到此次提交之前的一次提交的 commit`7753f40d892a8e0d14176a42f6e12ae0179a3210`
2.  执行如下命令
    
    ```
    git rebase -i  7753f40
    ```
    
    弹出如下界面 (原图丢失，下图类似)  
    
    ![](<images/1683810396258.png>)
    
3.  将`0fb295f`这一行前面的 pick 改为 drop，然后按照提示保存退出
4.  至此已经删除了指定的 commit，可以使用`git log`查看下