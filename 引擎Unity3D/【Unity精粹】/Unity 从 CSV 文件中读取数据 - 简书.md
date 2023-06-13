# CSV
### 程序在表格文件中读取数据。

*   首选创建 excel，并保存为 csv 格式。用文本打开，可以看到一行行的字符串，以逗号分隔。

![](<images/1686645065163.png>)

![](<images/1686645065208.png>)

*   为 csv 文件编写一个脚本，用于对此文件进行操作，打开读取 csv 文件，将 csv 中数据存储到一个 list 中，对 list 的数据读取的操作，脚本为 csvController ：

```
using UnityEngine;
using System.Collections;
using System.IO;
using System.Collections.Generic;

public class csvController  {

    static csvController csv;
    public List<string[]> arrayData;  

    private csvController()   //单例，构造方法为私有
    {
        arrayData = new List<string[]>();
    }

    public static csvController GetInstance()   //单例方法获取对象
    {
        if(csv == null)
        {
            csv = new csvController();
        }
        return csv;
    }

    public void loadFile(string path,string fileName)
    {
        arrayData.Clear();
        StreamReader sr = null;
        try
        {
            string file_url = path + "//" + fileName;    //根据路径打开文件
            sr = File.OpenText(file_url);
            Debug.Log("File Find in " + file_url);
        }
        catch
        {
            Debug.Log("File cannot find ! ");
            return; 
        }

        string line;
        while((line = sr.ReadLine()) != null)   //按行读取
        {
            arrayData.Add(line.Split(','));   //每行逗号分隔,split()方法返回 string[]
        }
        sr.Close();
        sr.Dispose();
    }

    public string getString(int row,int col)
    {
        return arrayData[row][col];
    }
    public int getInt(int row,int col)
    {
        return int.Parse(arrayData[row][col]);
    }
}
```

*   在其他脚本中调用

```
void Start () {
        //csvController加载csv文件，单例模式，这个类只有一个对象，这个对象只能加载一个csv文件
        csvController.GetInstance().loadFile(Application.dataPath + "/Res", "csvTest.csv");
        //根据索引读取csvController中的list（csv文件的内容）数据
        speed = csvController.GetInstance().getInt(1, 2);
        Debug.Log("Player speed is " + csvController.GetInstance().getInt(1,2));
    }
```

参考：[http://www.jianshu.com/p/ffda934b5e15#](https://www.jianshu.com/p/ffda934b5e15#)

程序读取文件信息，这个感觉和以前 C++ 读取 text/dat 文件差不多的。  
unity 存储数据是如何实现的？参考 [http://www.wtoutiao.com/a/2139080.html](https://link.jianshu.com?t=http://www.wtoutiao.com/a/2139080.html)