---
sort: 1
title: "2022.3.4"
---

# 数据类型

c++11 字符串/数组/结构体初始化可以省略等号

```c++
int arr[] {1,2,4,8};	// c++ style
char str1[]="abc";		// c style
char str2[] {"abc"};	// c++ style
string str3 {"abc"};	// c++ style
struct people
{
	int age;
	string name;
};
struct people p1{18,"LiMing"};
```

# iostream

- 标准输入输出流
  cin/cout/cerr/clog
  cout.setf(ios_base::bolalpha) 设置 cout 遇到 bool 类型数据时输出对应字符串(true/false)

cin 在接收标准输入时默认开启缓冲, 如果类型为 char 则自动忽略空格和换行符, 并且只有在按下回车后才接受缓冲中的所有内容
如果要逐字符的检查所有输入字符需要使用 cin.get(), 传入的参数无需像 scanf 那样取地址, 因为 get 函数已经声明其为一个引用
getline 接收一行输入, 默认添加字符串结尾 '\0'

```c++
char c;
char str[5];
cin.get(c);
cin.getline(str, 5);
```

- 文件 I/O
  头文件 <fstream>
  ofstream

# 循环和控制结构

c++11 中一种新特性是`基于范围的循环`(类似 python 等脚本), 常用于遍历数组/向量等模板类的成员

```c++
vector<int> arr(10);
for(int i:arr)
	cout << i << endl;
// 修改成员时需要传引用
for(int &x:arr)
	x=x*0.8
for(int a:{1,3,5,7})
	cout << a << endl;
```

# 引用

其本质是一个指针常量(指针是一个常量, 指向一个变量), 省略了解引用和取址过程

- 可以用于函数返回值, 返回类的引用, 然后就可以继续使用类的方法

- 返回引用的函数可以作为左值赋值(可用于修改局部静态变量)

```c++
int &internal_x(void)
{
	static int x=0;
	cout << "x=" << x << endl;

	return x;
}

// 将 X 修改为 50;
internal_x = 50;
```

- const 引用 引用不能直接引用常量, 但是 const 引用可以, 常用于限制函数形参不可修改(传引用与传指针一样)

# string 类

类似于 c 中的字符串, 支持自动调整大小, 避免了字符串溢出问题
可以直接运算 str = str1 + str2
raw 字符串中无需加转义反斜杠\

```c++
#include <string>

string str="abc";
R"(This is \ a book "His address")"
```

# vector/array 模板类

类似于 c 中的数组, 他们都包含在 std 空间中, 对应头文件 <vector> <array>
vector 为动态数组使用堆空间, array 为静态数组使用栈空间(与数组效率相同)
支持直接赋值, 添加删除元素等高级操作
访问成员可以使用传统的下标(arr[i]), 也可以使用 c++ 的类方法 arr.at(i)

```c++
// 基本语法
// vector<type> name(nums)
// array<type, nums> name
vector<int> arr1(10);
array<int,20> arr2 = {0};
```
