---
sort: 2
title: "2022.3.5"
---
# OOP
## 类
* 一个类拥有4个缺省函数
1. 构造函数
2. 析构函数
3. 拷贝构造函数
4. = 号重载函数(operator=)

* this 指针
1. 用于成员函数中, 表示实际调用函数的类实例
2. 用于链式调用, 函数返回 类引用(return * this)
c++ 中可以访问空指针的成员函数, 但不能访问其成员变量(一般在成员函数中加入空指针判断)
## 类的构造函数
用来初始化类中的成员, 分配资源
三种类型的构造函数:
1. 无参构造
2. 有参构造
3. 拷贝构造(一般使用 const 引用)
class Person()
{
	private:
	int age;
	Person(int a)
	{
		age = a;
	}
	Person(const &p);
}
拷贝构造中, 堆区资源浅拷贝在析构时会导致重复释放异常, 解决方法是在拷贝构造函数中重新申请资源赋值(深拷贝)

初始化列表
用于初始化 const/引用 型类成员(构造函数中是赋值操作), 类中的类成员带参初始化, 使用括号赋值法
```c++
class Person
{
public:
		int a;
		int b;
		int c;
		Person(int a,int b, int c)
		{
			a=a,b=b,c=c;
		}
		// 初始化列表
		Person(int x,int y, int z):a(x),b(y),c(z)
		{

		}
}
```

## 类的初始化
空对象占用1字节内存空间
1. 括号初始化
Person p1(10);
2. 显式初始化
Person p1=Person(10);
3. 隐式初始化
Person p1=10;
## 静态成员变量/函数
需要在类外初始化, 静态成员不属于任一实例, 只在全局空间中存在一份, 赋值和访问时使用类名而不是实例名
静态成员函数无法访问非静态成员变量(只能访问全局变量, 不能访问实例私有成员)
```c++
int p1.i=0;
class P
{
	static int i;
}
P p1;
```
## 常函数/常对象
在函数后加 const 为常函数, 作用是将 this 设置为 const, 函数体中后续操作无法修改成员变量(如果成员变量是 mutable 的则仍可以修改)
声明对象时加 const 为常对象, 只能调用常函数
```c++
class P
{
	int a;
	mutable int b;
	void const_fun() const
	{

	}
};
```
## 友元
成员函数/全局函数/类都可以作为友元, 用于访问类中的私有成员
```c++
class p
{
	friend void friend_meta();
}
```
## 运算符重载
用于编译器支持的数据类型之外的自定义操作, 如两个类相加
* operator+ 重载加号(+)
通过成员函数/全局函数重载运算符, 如果是成员函数可以只传一个参数, 另一个参数使用 this
```c++
P operator+ (P &p1, P &p2)
{
	Person tmp;
	tmp.a=p1->a+p2.a;
	return tmp;
}

class P
{
	P operator+(P &p)
	{
		Person tmp;
		tmp.a=this->a+p.a;
		return tmp;
	}
	int a;
	int b;
}

P p1(10), p2(20);
P p3=p1+p2;
```

* operator<< 重载左移运算符
可以用于输出自定义数据类型(cout << p), 通常成员函数无法实现重载左移运算符输出, 因为实现后的形式为 p << cout, 可以使用全局函数(必须仍然返回 ostream 来实现链式调用)
```c++
ostream & operator<< (ostream &cout, P &p)
{
	cout << p.a << p.b;
	return cout;
}
```

* operator++ 重载递增运算符
递增运算符分为前置和后置, 需要分别重载
void operator++(int) 用 int 代表占位参数, 表示后置递增
前置递增返回引用, 后置递增返回临时变量的值(自身递增在过程中)

* operator= 重载类的默认赋值运算符
类中包含动态内存数据, 则需要重载默认的赋值运算符以避免浅拷贝问题

* operator() 重载函数调用运算符(仿函数)
匿名函数对象 ret=MyAdd()(1,2);

## 继承
class <派生类>: <继承方式> <基类>
继承方式与权限一致, 包括 public/private/protected, 任何继承都不包括基类的 private
public	继承: 沿用基类的 public 和 protected 属性
private	继承: 私有继承基类的 public 和 protected
protected 继承: 保护继承基类的 public 和 protected
```c++
class COM
{
	int a;
	int b;
};

class C1 : public COM
{
	int c;
}
```
基类的私有数据其实也被子类继承了, 但无法访问(类似基因的选择表达)
