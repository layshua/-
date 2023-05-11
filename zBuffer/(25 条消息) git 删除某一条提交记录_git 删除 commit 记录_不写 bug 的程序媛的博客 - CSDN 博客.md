1. 首先使用 git log 命令查看提交记录，找到出错的那一笔提交的 commit_id(黄色)  

![](<images/1683810435617.png>)

2，用命令 git [rebase](https://so.csdn.net/so/search?q=rebase&spm=1001.2101.3001.7020 "rebase") -i commit_id , 查找提交记录  
 

```
git rebase -i 15774a44d46bcd0c055b07c63bd0ecbe35a9660b
```

  
3. 执行 (2) 命令后出现 如下界面：  

![](<images/1683810435660.png>)

 4. 输入 i 进入编辑模式，在要删除的 commitid 前，将 pick 修改成 drop。看到的结果如下图：

![](<images/1683810435699.png>)

5. 编辑完成，按键盘 Esc，退出编辑模式，然后按 Shift+; 再输入 wq!（保存文件的写入修改）退出。（q! 是不保存修改）

![](<images/1683810435789.png>)

  

6. 这时候本地删除已经完成了，然后推送到远端 git push --force