1 C++ 中 #(一个 #号) 和 ##(两个 #号) 的用法和作用

1 关于一个 #号：  
在 C 语言的宏中，# 的功能是将其后面的宏参数进行[字符串](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1001.2101.3001.7020)化操作， 简单说就是对他所引用的宏变量通过替换后再其左右各加上一个双引用。  
例子：

```
# define WARNIF(EXP) \
do{ \
    if (EXP) \
    {    \ 
        fprintf(stderr, "warning:" #EXP "\n"); \
    }   \    
}while(0)    
在实际使用中会出现下面所示的替换过程：
WARN_IF(div == 0); 被替换成以下代码
do{
    if (div == 0) 
    {        
        fprintf(stderr, "warning:" "div == 0" "\n"); 
    }       
}while(0)
```

2 关于两个 #号：  
## 被称为连接符，用来将两个 Token 链接成一个 Token. 注意这里的连接的对象时 Token 就行， 而不一定是宏的变量。比如你要做一个菜单项命令名和函数指针组成的结构体的数组，并且希望在函数和菜单项命令名直接有直观的名字上的关系， 那么下面的代码就非常实用

```
struct command
{
    char *name；
    void (*function)(void);
};
 
#define COMMAND(NAME){NAME, NAME##_command}
然后就用一些预定义好的命令来方便的初始化一个command结构的数据了：
struct command commands[] = {
    COMMAND(quit),
    COMMAND(help),
    ...
}
COMMAND宏在这里充当一个代码生成器的作用，这样可以在一定程度上减少代码的密度， 间接的也可以减少粗心所造成的错误。
我们还可以n个##符号链接n+1个Token，这个特性也是#符号所不具备的。
例如： #define LINK_MULTIPLE(a, b, c, d) a##_##b##_##c##_###d
typedef struct_record_type LINK_MULTIPLE(name, company, position, salary);
//展开内容为 ： typedef struct_record_type name_company_position_salary;
```

3 关于... 的使用  
... 在 C 语言中被称为变参宏   
#define myprintf(templt, ...)  fprintf(stderr, templt, ##__VA_ARGS__)  
这是 ## 这个连接符充当的作用就是当__VA_ARGS__为空的时候， 消除前面的那个逗号。