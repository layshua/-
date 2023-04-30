```c++
//将序列中的负数都改为正数
int main()
{
		vector<int> v{-1,-2,3,4,-5};
		//正确：无需指定返回类型，编译器推断返回int
		transform(v.begin(), v.end(), v.begin(),
			[](int i) {return i < 0 ? -i : i; });

		for (auto value : v)
		{
			cout << value << endl; //输出1 2 3 4 5
		}
}


//用if改写，看起来等价
//正确：return的表达式类型都是int
transform(v.begin(), v.end(), v.begin (),
		  [](int i) {if(i < 0) return -i; else return i;}) 

//使用尾置返回类型，指定lambda返回类型
transform(v.begin(), v.end(), v.begin (),
		  [](int i)->int {if(i < 0) return -i; else return i;}) 


//无return，返回类型为void
//错误，lambda返回了void，而vector元素是int类型，无法完成转换
transform(v.begin(), v.end(), v.begin(),
		[](int i) {if (i < 0)  i*=-1; });
	
```

```c++ 
test
```

```/
123
```





[aidenlx/obsidian-icon-shortcodes: Obsidian Plugin: Insert emoji and custom icons with shortcodes --- aidenlx/obsidian-icon-shortcodes：黑曜石插件：使用简码插入表情符号和自定义图标 (github.com)](https://github.com/aidenlx/obsidian-icon-shortcodes)
