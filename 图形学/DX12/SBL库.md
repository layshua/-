# 0 API
```c++ nums
#include "simple_library/public/simple_library.h"
```

```c++ nums
void init_def_c_paths(def_c_paths *c_paths);
void init_def_c_paths_w(def_c_paths_w* c_paths);

// 拷贝文件
int copy_file(char *Src, char *Dest);

// 查找文件（第3个参数确定是否递归查找）
void find_files(char const *in_path, def_c_paths *str,bool b_recursion);

bool is_file_exists(char const* filename);

// 创建文件夹
bool create_file(char const *filename);

// 创建路径
bool create_file_directory(char const *in_path);

// 打开地址
bool open_url(const char* url);

// 通过参数打开url
bool open_url_by_param(const char* url, const char* param);

// 通过操作打开某个东西
bool open_by_operation(const char* in_operation, const char* url, const char* param);

// 打开一个文件夹
bool open_explore(const char* url);

// 使用以下接口 一定要初始化buf
char buf[1024] = { 0 };

bool get_file_buf(const char *path,char *buf);

bool save_file_buff(const char* path, char* buf);

bool add_file_buf(const char *path, char *buf);

// 这个函数是以字符串的方式存储，如果数据中有0 自动截断，建议用二进制存储
bool add_new_file_buf(const char *path, char *buf);

// 这个是以二进制方式读取
bool load_data_from_disk(const char* path, char* buf);
// 获取文件大小
unsigned int get_file_size_by_filename(const char *filename);

unsigned int get_file_size(FILE *file_handle);

// 这个是以二进制方式存储，不会遇到像0自动截断的情况
bool save_data_to_disk(const char* path, char* buf, int buf_size);


```

```c++ nums
// 宽字符和窄字符
// 
// 宽字符转窄字符
_number_of_successful_conversions(size_t) wchar_t_to_char(
	_out_pram(char*) dst_char,
	size_t char_size,
	_in_pram(wchar_t const*) _Src);

// 窄字符转宽字符
_number_of_successful_conversions(size_t) char_to_wchar_t(
	_out_pram(wchar_t*) dst_wchar_t,
	size_t wchar_t_size,
	_in_pram(char const*) _Src);
```

```c++ nums

// 宽字符
//////////////////////////////////////////////
// 这个函数是以字符串的方式存储，如果数据中有0 自动截断，建议用二进制存储
bool add_new_file_buf_w(const wchar_t* path, char* buf);

bool get_file_buf_w(const wchar_t* path, char* buf);

// 这个是以二进制方式存储，不会遇到像0自动截断的情况
bool save_data_to_disk_w(const wchar_t* path, char* buf,int buf_size);

// 这个是以二进制方式读取 buf的大小要比实际大小+1 因为最后一位留给/0
bool load_data_from_disk_w(const wchar_t* path, char* buf);

bool is_file_exists_w(const wchar_t *filename);

// 打开地址
bool open_url_w(const wchar_t* url);

bool open_url_by_param_w(const wchar_t* url,const wchar_t *param);

bool open_by_operation_w(const wchar_t *in_operation, const wchar_t* url, const wchar_t* param);

// 打开一个文件夹
bool open_explore_w(const wchar_t* url);

unsigned int get_file_size_by_filename_w(const wchar_t* filename);
```
# 1 打开文件操作
`_open_url`地址可以是网站，也可以是本地文件
```c++ nums
open_url("http://renzhai.net/");
open_url("C:/Test/hello.bmp");
//open_url_w(L"http://renzhai.net/"); //宽字符
```

# 2 读取磁盘二进制文件

```c++ nums
char path[] = "C:/Test/hello.bmp";
char bmp[1024] = { 0 };
load_data_from_disk(path, bmp);
```

# 3 获取文件大小
```c++ nums
int size = get_file_size_by_filename("C:/Test/hello.bmp");
```

# 4 存储数据到磁盘
```c++ nums
char path[] = "../test.txt";
char buf[1024] = { 0 };
save_data_to_disk(path, buf, strlen(buf));
```

C 库函数 `size_t strlen(const char *str)` 计算字符串 **str** 的长度，直到空结束字符，但不包括空结束字符。
# 5 窄字符宽字符抓换
```c++ nums
// 宽字符转窄字符
wchar_t path_w[] =L"../test.txt";
char path[1024] = { 0 };
wchar_t_to_char(path, sizeof(path), path_w);
cout << path << endl;

// 窄字符转宽字符
char path[] = "../test.txt";
wchar_t path_w[1024] = { 0 };
char_to_wchar_t(path_w, 1024,path);
cout << path_w << endl;
```

# 6 创建文件和文件夹
**创建文件**
```c++ nums
char path[] = "C:/Users/22625/Desktop/LearnSBL/test.txt";
create_file(path);
```

**创建文件夹**
在地址 `C:/Users/22625/Desktop/LearnSBL/LearnSBL` 后创建文件夹 `TestCreatFiles` 及其子文件夹 ` FileA `
```c++ nums
char path[] = "C:/Users/22625/Desktop/LearnSBL/LearnSBL/TestCreatFiles/FileA";
create_file_directory(path);
```

# 7 递归查找文件


