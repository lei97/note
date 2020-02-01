#  C++ 语法糖

[TOC]

链接：https://changkun.gitbooks.io/cpp1x-tutorial/content/

##  nullptr

**nullptr** 目的是为了替代 **NULL** 。

```c++
int *num = nullptr;
```
在函数重载时，使用 **NULL** 会导致重载混乱。

```c++
#define NULL 0
int *ch = NULL;
void foo (char *);  //函数1
void foo (int );	//函数2

foo(NULL);		//编译失败;
foo(nullptr);   //调用函数2
```
- 建议：使用 **nullptr** 替代 **NULL** 。

## constexpr

C++ 具备常数表达式的概念，为了提高性能，编译时把表达式直接优化并植入到程序运行中。

```c++
#define LEN 10

int len_foo()
{
	return 5;
}

int main ()
{
    int len = 5;
	char arr_1[len];	 // 非法
    const int len_2 = 5;
    char arr_2[len_2];   //合法
    constexpr int len_3 =  len_2 + 1;
    char arr_3[len_3];	 //合法
}
```

`constexpr` 函数

```c++
constexpr int fibonacci(const int n) 
{
	return n == 1 || n == 2 ? 1 : fibonacci(n - 1) + fibonacci(n - 2);
}
```

C++14 标准下 `constexpr` 函数支持 局部变量、循环和分支等简单语句 。

```c++
constexpr int fibonacci_14(const int n)
{
	if (n == 1) return 1;
	if (n == 2) return 1;
	return fibonacci(n - 1) + fibonacci(n - 2);
}
```

## auto 

减少变量类型操作 ，利用 `auto`  进行类型推导。

```c++
std::vector<int> vec{ 1,2,3,4,5 };

for (std::vector<int>::const_iterator itr = vec.cbegin(); itr != vec.cend(); ++itr)
{
	std::cout << *itr << std::endl;
}

// 等价
for (auto itr = vec.cbegin(); itr != vec.cend(); ++itr)
{
	std::cout << *itr << std::endl;
}
```

常见用法 ：

```c++
auto i = 5;
auto arr = new auto(10);  //arr 被推导为 int*
```

> 注意事项：`auto`  不能用于函数传参，应该使用模板。

```c++
int add(auto x, auto y)  //编译出错
```

> `auto` 不能用于推导数组类型。

```c++
int arr[10] = {};
auto auto_arr = arr;  //类型为int 数据为 arr[0]
```

###  decltype 

`decltype` 关键字为了解决 `auto` 关键字只能对变量进行类型推导的缺陷。 用法类似于 `sizeof` 。

```c++
auto x = 1;
auto y = 1;
decltype (x + y) z;
```

`auto` 与 `decltype` 配合

 ```c++
// 写法 1 
template<typename R, typename T, typename U>
R add(T x, U y) {
	return x + y;
}

// 写法 2 C++11 尾类型返回
template<typename T, typename U>
auto add(T x, U y) -> decltype(x + y) {
	return x + y;
}  

// 写法 3 C++14
template<typename T, typename U>
auto add(T x, U y) {
	return x + y;
}
 ```

###  基于范围的 `for` 循环

```c++
std::vector<int> vec{ 1,2,3,4,5 };
// 写法 1 之前
for (std::vector<int>::const_iterator itr = vec.cbegin(); itr != vec.cend(); ++itr)
{
	std::cout << *itr << std::endl;
}

// 写法 2 C++11
for (auto& i : vec)
{
	std::cout << i << std::endl;
}
```

###  初始化列表

```C++
int arr[3] = {1, 2, 3};

std::vector<int> v = {1, 2, 3, 4, 5};

void foo(std::initializer_list<int> list);
foo({1,2,3});

struct A 
{
	int a;
    float b;
}

struct B {
	B(int _a, float _b) : a(_a), b(_b) {}
private:
	int a;
	float b;
};

A a {1, 1.1};
B b {2, 2.2};
```

###  模板增强

####  外部模板

```c++
// TODO 待整理
template class std::vector<Foo>;         // 强行实例化
extern template class std::vector<Foo>;  // 不在该编译文件中实例化模板
```

####  尖括号

嵌套模板代码：

```c++
std::vector<std::vector<int>> num;
```

####  类型别名模板

> 模板是用来产生类型的。	**typedef**  定义与一个新的名称，不能为模板定义为一个新的名称。

```c++
template<class T,class U>
class SuckType;
// 分别在 QT，visual studio 能通过编译
typedef SuckType<std::vector<int>,std::string> NewType;
```

C++11 使用 `using`，并且对传统 `typedef` 相同的功效

```c++
typedef int (*process)(void*);  // 定义了一个返回类型为 int，参数为 void* 的函数指针类型，名字叫做 process
using process = int(*)(void*); // 同上, 更加直观
using NewType1 = SuckType<std::vector<int>, std::string>;
```

####  默认模板参数

```c++
// 原函数
template<typename T, typename U>
auto add(T x, U y) -> decltype(x + y) 
{
	return x + y;
}
```

```c++
// 函数
template<typename T = int, typename U = int>
auto add(T x, U y) -> decltype(x+y) 
{
    return x+y
}

add(100,100);
```

####  变长参数模板

> C++11 引入了 多个模板参数表示方法，允许任意个数，任意类别的模板参数。

```c++
/******************************************************
* 变长参数模板
******************************************************/
template<typename...Ts> 
class Magic {};    // 必须添加 {} 否则编译报错

// 多个模板参数
Magic<int,
	std::vector<int>,
	std::map<std::string,
	std::vector<int>>> darkMagic;

// 0 个模板参数
Magic<> nothing;

// 至少 1 个模板参数
template<typename Require, typename... Args> class Magic_1;

//变长参数函数
template<typename... Args>
void printf(const std::string& str, Args... args);

template<typename... Args>
void magic(Args... args) {
    std::cout << sizeof...(args) << std::endl;
}
```

```c++
magic();		// 输出 0
magic(1);     	 // 输出 1
```

#####  解包方式

* **递归模板参数**

> 需要定义一个终止递归函数。

``` c++
template<typename T>
void printf1(T value) {
    std::cout << value << std::endl;
}
template<typename T, typename... Args>
void printf1(T value, Args... args) {
    std::cout << value << std::endl;
    printf1(args...);
}

printf1(1, 3, 5.5, 23);
// 输出结果：1
//		   3
//          5.5
//          23
```

*  **初始化列表展开** 

>  C++11 中提供的初始化列表以及 Lambda 表达式的特性（下一节中将提到），而 std::initializer_list 也是 C++11 新引入的容器 

```c++
template<typename T, typename... Args>
auto printf2(T value, Args... args) 
{
    std::cout << value << std::endl;
    return std::initializer_list<T>{([&] {
        std::cout << args << std::endl;
    }(), value)...};
}
```

>  事实上，有时候我们虽然使用了变参模板，却不一定需要对参数做逐个遍历，我们可以利用 `std::bind` 及完美转发等特性实现对函数和参数的绑定，从而达到成功调用的目的。 

###  面对对象增强

####  委托构造

> 在同一个类中一个构造函数调用另一个构造函数，简化代码。

```c++ 
class Base {
    public:
    int m_value1;
    int m_value2;
    Base() {
        m_value1 = 1;
    }
    Base(int value) : Base() {  // 委托 Base() 构造函数
        m_value2 = value;
    }
};

// 使用
Base base(2);
```

####  继承构造

> 在传统 C++ 中，构造函数需要参数一一传递，导致效率低下。C++11 利用关键字 `using` 引进继承构造函数。

```c++
class Subclass : public Base {
    public:
    using Base::Base;          // 继承构造 取消这一行造成下面使用编译错误
};

// 使用 
Subclass s(3);
```

####  显式虚函数重载

> 在传统 C++ 中，经常发生意外重载虚函数。

```c++ 
	struct BaseStruct {
		virtual void foo()
		{
			cout << "BaseStruct" << endl;
		}
	};
	struct SubStruct : BaseStruct {
		void foo()
		{
			cout << "SubStruct" << endl;
		}
	};
```

>  `SubClass::foo` 可能并不是程序员尝试重载虚函数，只是恰好加入了一个具有相同名字的函数。另一个可能的情形是，当基类的虚函数被删除后，子类拥有旧的函数就不再重载该虚拟函数并摇身一变成为了一个普通的类方法，这将造成灾难性的后果 。

 C++11 引入了 `override` 和 `final` 这两个关键字来防止上述情形的发生。

#####  `override`

```c++ 
	struct BaseStruct {
		virtual void foo(float);
	};

	struct SubStruct : BaseStruct {
		void foo(float) override;
         void foo(double) override;  // 不合法
	};
```

#####  `final`

 ```c++
	struct BaseStruct {
		virtual void foo1() final;
	};
	struct SubStruct1 : BaseStruct {
		void foo1(); 	// 不合法
	};
	struct SubClass1 final : BaseStruct {		//合法
	};
	struct SubClass2 : SubClass1 {
	};                  // 非法, SubClass1 已 final
 ```

####  显式禁用默认函数

>  允许显式的声明采用或拒绝编译器自带的函数，构造函数

```c++
	class Magic2 {
	public:
		Magic2() = default;  // 显式声明使用编译器生成的构造
		Magic2& operator=(const Magic2&) = delete; // 显式声明拒绝编译器生成构造
		Magic2(int magic_number);
	};
```

###  强枚举类型

```c++
	enum class new_enum : unsigned int {
		value1,
		value2,
		value3 = 100,
		value4 = 100
	};
```

```c++
	if (new_enum::value3 == new_enum::value4) {
		// 会输出
		std::cout << "new_enum::value3 == new_enum::value4" << std::endl;
	}
```

```c++
template<typename T>
std::ostream& operator<<(typename std::enable_if<std::is_enum<T>::value, std::ostream>::type& stream, const T& e)
{
	return stream << static_cast<typename std::underlying_type<T>::type>(e);
}

std::cout << new_enum::value3 << std::endl;
```

###  Lambda 表达式

> 类似于匿名函数的特性。不需要命名一个函数的情况下使用。

```
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {
    // 函数体
}
```

####  值捕获

> 与参数传值类似，值捕获前期变量可以拷贝，不同之处在于，被捕获的变量在 `lambda` 表达式==**被创建时拷贝**==，而非调用时拷贝。

```c++
	void learn_lambda_func_1() {
		int value_1 = 1;
		auto copy_value_1 = [value_1] {
			return value_1;
		};
		value_1 = 100;
		auto stored_value_1 = copy_value_1();
		cout << "stored_value_1:" << stored_value_1 << endl; 
		// 这时, stored_value_1 == 1, 而 value_1 == 100.
		// 因为 copy_value_1 在创建时就保存了一份 value_1 的拷贝
	}
```

####  引用捕获

```c++
	void learn_lambda_func_2() {
		int value_2 = 1;
		auto copy_value_2 = [&value_2] {
			return value_2;
		};
		value_2 = 100;
		auto stored_value_2 = copy_value_2();
		cout << "stored_value_2:" << stored_value_2 << endl;
		// 这时, stored_value_2 == 100, value_1 == 100.
		// 因为 copy_value_2 保存的是引用
	}
```

#### 隐式捕获

>  可以在捕获列表中写一个 `&` 或 `=` 向编译器声明采用 引用捕获或者值捕获 

*  [] 空捕获列表 
* [name1, name2, ...] 捕获一系列变量
* [&] 引用捕获，让编译器自行推导捕获列表
* [=] 值捕获，让编译器执行推导应用列表

####  表达式捕获（ C++14 ）

> 捕获方式均为左值，不能捕获右值。

```c++
	void learn_lambda_func_3() {
		auto important = std::make_unique<int>(1);
		auto add = [v1 = 1, v2 = std::move(important)](int x, int y) -> int {
			return x + y + v1 + (*v2);
		};
		auto test = add(3, 4);
		std::cout << add(3, 4) << std::endl;
	}
```

> `important` 是一个独占指针，不能捕获，转移为右值，在表达式初始化。

####  泛型 Lambda ( C++14 )

```c++ 
	auto add = [](auto x, auto y) {
		return x + y;
	};

	auto b = add(1, 2);
	auto a = add(1.1, 2.2);
```

### 函数对象包装器

####  `std::function`

> `Lambda` 表达式的本质是一个函数对象，当 `Lambda` 表达式捕获列表为空时，`Lambda` 表达式还能够作为一个函数指针进行传递。

```c++
	using foo2 = void(int);  // 定义函数指针, using 的使用见上一节中的别名语法
	void functional(foo2 f) {
		f(2);
	}

	void test_foo2()
	{
		auto f = [](int value) {
			std::cout << value << std::endl;
		};
		functional(f);  // 函数指针调用
		f(1);		   // 直接调用 Lambda 表达式
	}
```

####  通用、多态的函数封装

> C++中现有的可调用实体的一种类型安全的包裹（相对来说，函数指针的调用不是类型安全的），换句话说，就是函数的容器 。

```c++
	int foo_3(int para) {
		return para;
	}

	void test_foo3() 
	{
		// std::function 包装了一个返回值为 int, 参数为 int 的函数
		std::function<int(int)> func = foo_3;

		int important = 10;
		std::function<int(int)> func2 = [&](int value) -> int {
			return 1 + value + important;
		};
		std::cout << func(10) << std::endl;
		std::cout << func2(10) << std::endl;
	}
```

#### `std::bind/std::placehorder`

>  `std::bind` 则是用来绑定函数调用的参数，解决不一定获取调用函数取得全部参数。

```c++
	int foo4(int a, int b, int c)
	{
		return a + b + c;  // a = 3, b = 1, c = 2
	}

	void test_foo4()
	{
         // TODO 了解更加深入
		auto bindFoo4 = std::bind(foo4, std::placeholders::_1, 1, 2);
		//调用BinFoo4，只需一个参数
		bindFoo4(3);	
	}
```

###  右值引用

> 右值引用是 C++11 引入的与 Lambda 表达式齐名的重要特性之一。

####  左值、右值的纯右值、将亡值、右值

**左值(lvalue, left value)**，顾名思义就是赋值符号左边的值。准确来说，左值是表达式（不一定是赋值表达式）后依然存在的持久对象。

**右值(rvalue, right value)**，右边的值，是指表达式结束后就不再存在的临时对象。 在 C++11 中引入了强大的右值引用，进行进一步划分：纯右值、将亡值。 

**纯右值(prvalue, pure rvalue)**，纯粹的右值，要么是纯粹的字面量，例如 `1+2`。非引用返回的临时变量、运算表达式产生的临时变量、原始字面量、Lambda 表达式都属于纯右值。 

**将亡值(xvalue, expiring value)**，是 C++11 为了引入右值引用而提出的概念（因此在传统 C++中，纯右值和右值是统一个概念），也就是即将被销毁、却能够被移动的值。 

**将亡值** 定义了这样一种行为：临时的值能够被识别、同时又能够被移动 。

```c++ 
	std::vector<int> foo5() {
		std::vector<int> temp = { 1, 2, 3, 4 };
		return temp;
	}

	void test_foo5()
	{
        // v 是左值，foo5() 返回值是右值
        // foo5() 产生的返回作为一个临时变量，被 V 复制，立刻被销毁。
		std::vector<int> v = foo5();
	}
```

####  右值引用和左值引用

> 需要拿到一个将亡值，需要用到右值引用的申明：`T &&`。`T` 是类型。右值引用让临时生命周期得以延长。 
>
> `std::move` 将左值转换为右值。

```c++
	void reference(std::string& str) 
	{
		std::cout << "左值" << std::endl;
	}

	void reference(std::string&& str) 
	{
		std::cout << "右值" << std::endl;
	}

	void test_reference()
	{
		std::string  lv1 = "string,";       // lv1 是一个左值

		#ifdef ERR_CODE
		std::string&& r1 = lv1;             // 非法, 右值引用不能引用左值
		#endif

		std::string&& rv1 = std::move(lv1); // 合法, std::move可以将左值转移为右值
		std::cout << rv1 << std::endl;      // string,

		const std::string& lv2 = lv1 + lv1; // 合法, 常量左值引用能够延长临时变量的申明周期

		#ifdef  ERR_CODE
		lv2 += "Test";                      // 非法, 引用的右值无法被修改
		#endif //  ERR_CODE

		std::cout << lv2 << std::endl;      // string,string

		std::string&& rv2 = lv1 + lv2;      // 合法, 右值引用延长临时对象声明周期
		rv2 += "Test";                      // 合法, 非常量引用能够修改临时变量
		std::cout << rv2 << std::endl;      // string,string,string,Test

		// rv2 虽然引用了一个右值，但由于它是一个引用，所以 rv2 依然是一个左值。
		reference(rv2);                     // 输出左值
	}
```

#### 移动语义

`TODO:`  这一部分待整理。

```c++
	class A {
	public:
		int* pointer;
		A() :pointer(new int(1)) { std::cout << "构造" << pointer << std::endl; }
		A(A& a) :pointer(new int(*a.pointer)) { 
			std::cout << "拷贝" << pointer << std::endl; 
		}    // 无意义的对象拷贝
		A(A&& a) :pointer(a.pointer) { 
            a.pointer = nullptr; std::cout << "移动" << pointer << std::endl; 
         } 
		~A() { 
            std::cout << "析构" << pointer << std::endl; delete pointer; 
         }
	};

	A return_rvalue(bool test) {
		A a, b;
		return test ? a : b;
	}

	void test_return_rvalue()
	{
        /*
        	1. 会在 return_rvalue 内部构造两个对象 A 对象，获取两个构造函数的输出
        	2. 函数返回产生对象拷贝
        	3. 会在 return_rvalue 产生两次析构
        */
		A obj = return_rvalue(false);	
		std::cout << "obj:" << std::endl;
		std::cout << obj.pointer << std::endl;
		std::cout << *obj.pointer << std::endl;
	}
```

```c++
	void test_return_rvalue1()
	{
		std::string str = "Hello world.";
		std::vector<std::string> v;

		// 将使用 push_back(const T&), 即产生拷贝行为
		v.push_back(str);
		// 将输出 "str: Hello world."
		std::cout << "str: " << str << std::endl;

		// 将使用 push_back(const T&&), 不会出现拷贝行为
		// 而整个字符串会被移动到 vector 中，所以有时候 std::move 会用来减少拷贝出现的开销
		// 这步操作后, str 中的值会变为空
		v.push_back(std::move(str));
		// 将输出 "str: "
		std::cout << "str: " << str << std::endl;
	}
```

####  完美转发

```c++
	void reference(int& v) {
		std::cout << "左值" << std::endl;
	}
	void reference(int&& v) {
		std::cout << "右值" << std::endl;
	}

	template <typename T>
	void pass(T&& v) {
		std::cout << "普通传参:";
		reference(v);   // 始终调用 reference(int& )
	}

	void test_pass() 
	{
		std::cout << "传递右值:" << std::endl;
         // 由于 v 是一个引用，输出是一个左值
		pass(1);        // 1是右值, 但输出左值

		std::cout << "传递左值:" << std::endl;
		int v = 1;
		pass(v);        // r是左引用, 输出左值
	}
```

* **坍塌规则**： **无论模板参数是什么类型的引用，当且仅当实参类型为右引用时，模板参数才能被推导为右引用类型** 

| 函数形参类型 | 实参参数类型 | 推导函数形参类型 |
| :----------: | :----------: | :--------------: |
|     T &      |    左引用    |       T &        |
|     T &      |    右引用    |       T &        |
|     T &&     |    左引用    |       T &        |
|     T &&     |    右引用    |       T &&       |

```c++
	template <typename T>
	void pass1(T&& v) 
	{
		std::cout << "普通传参:";
		reference(v);
		std::cout << "std::move 传参:";
		reference(std::move(v));
		std::cout << "std::forward 传参:";
		reference(std::forward<T>(v));
	}

	void test_pass1()
	{
		std::cout << "传递右值:" << std::endl;
		pass1(1);

		std::cout << "传递左值:" << std::endl;
		int v = 1;
		pass1(v);
	}

	/*
        传递右值:
        普通传参:左值
        std::move 传参:右值
        std::forward 传参:右值
        传递左值:
        普通传参:左值
        std::move 传参:右值
        std::forward 传参:左值
	*/
```

> 无论传递参数为左值还是右值，普通传参都将参数进行左值进行转发，`std::move` 接到一个左值，调用了 `reference(int &&)` 输出右值引用。

> `std::forward` 即没有造成任何多余的拷贝，同时**完美转发**(传递)了函数的实参给了内部调用的其他函数。

>  `std::forward` 和 `std::move` 一样，没有做任何事情，`std::move` 单纯的将左值转化为右值，`std::forward` 也只是单纯的将参数做了一个类型的转换，从是线上来看，`std::forward(v)` 和 `static_cast(v)` 是完全一样的。 

###  `std::array` 和 `std::forward_list`

####  `std::array`

* `std::array` 编译时创建一个固定大小的数组。
* 不能被隐式转化为指针。

```c++ 
	void foo_array(int* p, int len) {
		return;
	}

	void test_array()
	{
		std::array<int, 5> arr = { 1,2,3,4,5 };

		#ifdef ERR_CODE
		int len = 4;
		std::array<int, len> arr1 = { 1,2,3,4 }; // 非法, 数组大小参数必须是常量表达式
		#endif
		#ifdef ERR_CODE
		foo_array(arr, arr.size());
		#endif

		foo_array(&arr[0], arr.size());
		foo_array(arr.data(), arr.size());

		std::sort(arr.begin(), arr.end());
	}
```

####   `std::forward_list`

> `std::forward_list` 是一个列表容器，使用方法和 `std::list` 基本类似。
>
> 需要知道的是，和 `std::list` 的双向链表的实现不同，`std::forward_list` 使用单向链表进行实现，提供了 `O(1)` 复杂度的元素插入，不支持快速随机访问（这也是链表的特点），也是标准库容器中唯一一个不提供 `size()` 方法的容器。当不需要双向迭代时，具有比 `std::list` 更高的空间利用率。

###  无序容器 ( C++11 )

```c++
	// TODO: 有序容器 std::map / std::set 学习
```

 C++11 引入了两组无序容器：`std::unordered_map`/`std::unordered_multimap` 和 `std::unordered_set`/`std::unordered_multiset` 

```c++
	void test_UnorderedContainers()
	{
		std::unordered_map<int, std::string> u = {
			{1, "1"},
			{3, "3"},
			{2, "2"}
		};
		std::map<int, std::string> v = {
			{1, "1"},
			{3, "3"},
			{2, "2"}
		};

		// 分别对两组结构进行遍历
		std::cout << "std::unordered_map" << std::endl;
		for (const auto& n : u)
			std::cout << "Key:[" << n.first << "] Value:[" << n.second << "]\n";

		std::cout << std::endl;
		std::cout << "std::map" << std::endl;
		for (const auto& n : v)
			std::cout << "Key:[" << n.first << "] Value:[" << n.second << "]\n";
	}

    /*
    	输出结果：
            std::unordered_map
            Key:[1] Value:[1]
            Key:[3] Value:[3]
            Key:[2] Value:[2]

            std::map
            Key:[1] Value:[1]
            Key:[2] Value:[2]
            Key:[3] Value:[3]
    */
```

###  元组 `std::tuple`

>  除了 `std::pair` 外，似乎没有现成的结构能够用来存放不同类型的数据（通常我们会自己定义结构）, `std::pair` 只能保存两个元素。

####  基本操作

```c++
	auto get_student(int id)
	{
		// 返回类型被推断为 std::tuple<double, char, std::string>

		if (id == 0)
			return std::make_tuple(3.8, 'A', "张三");
		if (id == 1)
			return std::make_tuple(2.9, 'C', "李四");
		if (id == 2)
			return std::make_tuple(1.7, 'D', "王五");
		return std::make_tuple(0.0, 'D', "null");
		// 如果只写 0 会出现推断错误, 编译失败
	}

	void test_tuple()
	{
		auto student = get_student(0);
		std::cout << "ID: 0, "
			<< "GPA: " << std::get<0>(student) << ", "
			<< "成绩: " << std::get<1>(student) << ", "
			<< "姓名: " << std::get<2>(student) << '\n';

		double gpa;
		char grade;
		std::string name;

		// 元组进行拆包
		std::tie(gpa, grade, name) = get_student(1);
		std::cout << "ID: 1, "
			<< "GPA: " << gpa << ", "
			<< "成绩: " << grade << ", "
			<< "姓名: " << name << '\n';
	}
```

`C++14` 增加使用类型来获取元组的对象：

```c++
		std::tuple<std::string, double, double, int> t("123", 4.5, 6.7, 8);
		std::cout << std::get<std::string>(t) << std::endl;
		//std::cout << std::get<double>(t) << std::endl;   // 非法, 引发编译期错误
		std::cout << std::get<3>(t) << std::endl;
```

####  运行期索引 （待测试）

>  `std::get<>` 依赖一个编译期的常量 。

```c++
int index = 1;
std::get<index>(t);	 // 不合法
```

>  使用 `boost::variant` 配合变长模板参数的黑魔法 。

```c++
#include <boost/variant.hpp>
template <size_t n, typename... T>
boost::variant<T...> _tuple_index(size_t i, const std::tuple<T...>& tpl) {
    if (i == n)
        return std::get<n>(tpl);
    else if (n == sizeof...(T) - 1)
        throw std::out_of_range("越界.");
    else
        return _tuple_index<(n < sizeof...(T)-1 ? n+1 : 0)>(i, tpl);
}
template <typename... T>
boost::variant<T...> tuple_index(size_t i, const std::tuple<T...>& tpl) {
    return _tuple_index<0>(i, tpl);
}
```

```c++
int i = 1;
std::cout << tuple_index(i, t) << std::endl;
```

####  元组合并与遍历（待测试）

合并元组可以通过 `std::tuple_cat` 实现：

```c++
auto new_tuple = std::tuple_cat(get_student(1), std::move(t));
```

```c++
template <typename T>
auto tuple_len(T &tpl) {
    return std::tuple_size<T>::value;
}

for(int i = 0; i != tuple_len(new_tuple); ++i)
    // 运行期索引
    std::cout << tuple_index(i, new_tuple) << std::endl;
```

###  RAII 与引用计数

 传统 C++ 里我们只好使用 `new` 和 `delete` 去『记得』对资源进行释放。而 C++11 引入了智能指针的概念，使用了引用计数的想法 。

> 注意：引用计数不是垃圾回收，引用技术能够尽快收回不再被使用的对象，同时在回收的过程中也不会造成长时间的等待，更能够清晰明确的表明资源的生命周期。 

####  `std::shared_ptr`

```c++
	void foo_shared(std::shared_ptr<int> i)
	{
		(*i)++;
	}
	
	void test_shared()
	{
		auto pointer = new int(10);  //编译通过
		// 构造了一个 std::shared_ptr
		auto pointer1 = std::make_shared<int>(10);
		foo_shared(pointer1);
		std::cout << *pointer1 << std::endl; // 11
		// 离开作用域前，shared_ptr 会被析构，从而释放内存
	}
```

```c++
	void test_shared1()
	{
		auto pointer = std::make_shared<int>(10);
		auto pointer2 = pointer;			// 引用计数+1
		auto pointer3 = pointer;			// 引用计数+1
		int* p = pointer.get();             // 这样不会增加引用计数

		std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;      // 3
		std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl;    // 3
		std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl;    // 3

		pointer2.reset();
		std::cout << "reset pointer2:" << std::endl;
		std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;      // 2
		std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl;    // 0, pointer2 已 reset
		std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl;    // 2

		pointer3.reset();
		std::cout << "reset pointer3:" << std::endl;
		std::cout << "pointer.use_count() = " << pointer.use_count() << std::endl;      // 1
		std::cout << "pointer2.use_count() = " << pointer2.use_count() << std::endl;    // 0
		std::cout << "pointer3.use_count() = " << pointer3.use_count() << std::endl;    // 0, pointer3 已 reset
	}
```

####  `std::unique_ptr`

是一种独占智能指针，禁止其他智能指针共享一个对象。

```c++
	struct Foo_unique {
		Foo_unique() { std::cout << "Foo_unique::Foo_unique" << std::endl; }
		~Foo_unique() { std::cout << "Foo_unique::~Foo_unique" << std::endl; }
		void foo_unique() { std::cout << "foo_unique::foo_unique" << std::endl; }
	};

	void f_unique(const Foo_unique&) {
		std::cout << "f(const Foo_unique&)" << std::endl;
	}
	
	void test_f_unique()
	{
		std::unique_ptr<Foo_unique> p1(std::make_unique<Foo_unique>());

		// p1 不空, 输出
		if (p1) p1->foo_unique();

		{
			std::unique_ptr<Foo_unique> p2(std::move(p1));

			// p2 不空, 输出
			f_unique(*p2);

			// p2 不空, 输出
			if (p2) p2->foo_unique();

			// p1 为空, 无输出
			if (p1) p1->foo_unique();

			p1 = std::move(p2);

			// p2 为空, 无输出
			if (p2) p2->foo_unique();
			std::cout << "p2 被销毁" << std::endl;
		}

		// p1 不空, 输出
		if (p1) p1->foo_unique();

		// Foo 的实例会在离开作用域时被销毁
	}
```

####  `std::weak_ptr`

```c++
// 内存泄漏
	struct A_sh;
	struct B_sh;

	struct A_sh {
		std::shared_ptr<B_sh> pointer1;
		~A_sh() {
			std::cout << "A 被销毁" << std::endl;
			//std::cout << "" << std::end;
		}
	};

	struct B_sh {
		std::shared_ptr<A_sh> pointer1;
		~B_sh() {
			std::cout << "B 被销毁" << std::endl;
		}
	};

	void test_shared_struct()
	{
		auto a = std::make_shared<A_sh>();
		auto b = std::make_shared<B_sh>();
		a->pointer1 = b;
		b->pointer1 = a;
	}
```

上面代码运行结果是 A，B 都不会被销毁，因为 a，b 的引用计数变为 2，离开作用域时，a，b 智能指针被析构，区域的引用计数减一，造成 a，b 对象的内存区域不为 0 ，造成内存泄漏。

![img](https://changkun.gitbooks.io/cpp1x-tutorial/content/pointers1.png) 

`std::weak_ptr`，`std::weak_ptr`是一种弱引用（相比较而言 `std::shared_ptr` 就是一种强引用）。引用计数不会增加计数。

![img](https://changkun.gitbooks.io/cpp1x-tutorial/content/pointers2.png) 

 `std::weak_ptr` 没有 `*` 运算符和 `->` 运算符，所以不能够对资源进行操作，它的唯一作用就是用于检查 `std::shared_ptr` 是否存在，`expired()` 方法在资源未被释放时，会返回 `true`，否则返回 `false`。 

###  正则表达式

####  简介

描述了一种字符串匹配模式。

* 检查一个串是否包含某中形式的子串。
* 将匹配的子串替换。
* 从某个串取出符合条件的字串。

学习链接： https://github.com/ziishaned/learn-regex/blob/master/translations/README-cn.md 

####  `std::regex` 及其相关

```C++
	void test_regex()
	{
		std::string fnames[] = { "foo.txt", "bar.txt", "test", "a0.txt", "AAA.txt" };
		// 在 C++ 中 `\` 会被作为字符串内的转义符，为使 `\.` 作为正则表达式传递进去生效，需要对 `\` 进行二次转义，从而有 `\\.`
		std::regex txt_regex("[a-z]+\\.txt");
		for (const auto& fname : fnames)
			std::cout << fname << ": " << std::regex_match(fname, txt_regex) << std::endl;

		std::regex base_regex("([a-z]+)\\.txt");
		std::smatch base_match;
		for (const auto& fname : fnames) {
			if (std::regex_match(fname, base_match, base_regex)) {
				// sub_match 的第一个元素匹配整个字符串
				// sub_match 的第二个元素匹配了第一个括号表达式
				if (base_match.size() == 2) {
					std::string base = base_match[1].str();
					std::cout << "sub-match[0]: " << base_match[0].str() << std::endl;
					std::cout << fname << " sub-match[1]: " << base << std::endl;
				}
			}
		}
	}
```

###  线程支持

####  `std::thread`

```c++
		void foo()
		{
			std::cout << "hello world" << std::endl;
		}

		void test_thread()
		{
			std::thread t(foo);
			t.join();
		}
```

#### `std::mutex` 和 `std::qnique_lock`

`std::mutex` 是 C++11 中最基本的 `mutex` 类，通过实例化 `std::mutex` 可以创建互斥量，而通过其成员函数 `lock()` 可以仅此能上锁，`unlock()` 可以进行解锁 。

 C++11 还为互斥量提供了一个 RAII 语法的模板类`std::lock_gurad`。

```C++
		void test_lock_guard()
		{
			static std::mutex mutex;
			std::lock_guard<std::mutex> lock(mutex);

			// ...操作

			// 当离开这个作用域的时候，互斥锁会被析构，同时unlock互斥锁
			// 因此这个函数内部的可以认为是临界区
		}
```

`std::unique_lock` 则相对于 `std::lock_guard` 出现的，`std::unique_lock` 更加灵活，`std::unique_lock` 的对象会以独占所有权（没有其他的 `unique_lock` 对象同时拥有某个 `mutex` 对象的所有权）的方式管理 `mutex` 对象上的上锁和解锁的操作。 

```c++
		std::mutex mtx;

		void block_area() {
			std::unique_lock<std::mutex> lock(mtx);
			//...临界区
		}
		
		void test_unique_lock()
		{
			std::thread thd1(block_area);
			thd1.join();
		}
```

####   `std::future` 和 `std::packaged_task`

```c++
        void test_future()
        {
            // 将一个返回值为7的 lambda 表达式封装到 task 中
            // std::packaged_task 的模板参数为要封装函数的类型
            std::packaged_task<int()> task([]() {return 7; });
            // 获得 task 的 future
            std::future<int> result = task.get_future();    // 在一个线程中执行 task
            std::thread(std::move(task)).detach();    
            std::cout << "Waiting...";
            result.wait();
            // 输出执行结果
            std::cout << "Done!" << std::endl << "Result is " << result.get() << '\n';
        }
```

#### `std::condition_variable`

```C++
void test_condition_variable()
{
    // 生产者数量
    std::queue<int> produced_nums;
    // 互斥锁
    std::mutex m;
    // 条件变量
    std::condition_variable cond_var;
    // 结束标志
    bool done = false;
    // 通知标志
    bool notified = false;

    // 生产者线程
    std::thread producer([&]() {
        for (int i = 0; i < 5; ++i) {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            // 创建互斥锁
            std::unique_lock<std::mutex> lock(m);
            std::cout << "producing " << i << '\n';
            produced_nums.push(i);
            notified = true;
            // 通知一个线程
            cond_var.notify_one();
        }
        done = true;
        cond_var.notify_one();
    });

    // 消费者线程
    std::thread consumer([&]() {
        std::unique_lock<std::mutex> lock(m);
        while (!done) {
            while (!notified) {  // 循环避免虚假唤醒
                cond_var.wait(lock);
            }
            while (!produced_nums.empty()) {
                std::cout << "consuming " << produced_nums.front() << '\n';
                produced_nums.pop();
            }
            notified = false;
        }
    });
	
    producer.join();
    consumer.join();
    
    /*
    	输出结果：
    		producing 0
    		consuming 0
    		...
    */
}
```

###  `long long int`

> C++ 11 纳入标准库，64 位。

###  `noexcept`修饰符

> 异常处理机制

```c++
    void may_throw();           // 可能抛出异常
    void no_throw() noexcept;   // 不可能抛出异常
```

```c++ 
void may_throw() 
{
    throw true;
}

auto non_block_throw = [] {
    may_throw();
};

void no_throw() noexcept {
    return;
}

auto block_throw = []() noexcept {
    no_throw();
};

void test_noexcept()
{
    std::cout << std::boolalpha
        << "may_throw() noexcept? " << noexcept(may_throw()) << std::endl
        << "no_throw() noexcept? " << noexcept(no_throw()) << std::endl
        << "lmay_throw() noexcept? " << noexcept(non_block_throw()) << std::endl
        << "lno_throw() noexcept? " << noexcept(block_throw()) << std::endl;

    // 捕获异常, 来自 my_throw()
    // 捕获异常, 来自 non_block_throw()
    try {
        may_throw();
    }
    catch (...) {
        std::cout << "捕获异常, 来自 may_throw()" << std::endl;
    }

    try {
        no_throw();
    }
    catch (...) {
        std::cout << "捕获异常，来自 no_throw" << std::endl;
    }

    try {
        non_block_throw();
    }
    catch (...) {
        std::cout << "捕获异常, 来自 non_block_throw()" << std::endl;
    }

    try {
        block_throw();
    }
    catch (...) {
        std::cout << "捕获异常, 来自 block_throw()" << std::endl;
    }
}
```

###  原始字符串字面量

```c++
void test_font()
{
	std::string str = R"(C:\\wate\\the\\f)";
	std::cout << str << std::endl;
}
```

###  自定义字面量

```c++
//字符串字面量自定义必须设置如下的参数列表
std::string operator"" _wow1(const char* wow1, size_t len) {
	return std::string(wow1) + "woooooooooow, amazing";
}

std::string operator"" _wow2(unsigned long long i) {
	return std::to_string(i) + "woooooooooow, amazing";
}

void test_font1()
{
    auto str = "abc"_wow1;
    auto num = 1_wow2;
    std::cout << str << std::endl;
    std::cout << num << std::endl;
}
```

* 整型字面量：重载时必须使用 `unsigned long long`、`const char *`、模板字面量算符参数，在上面的代码中使用的是前者；
* 浮点型字面量：重载时必须使用 `long double`、`const char *`、模板字面量算符；
* 字符串字面量：必须使用 `(const char *, size_t)` 形式的参数表
* 字符字面量：参数只能是 `char`, `wchar_t`, `char16_t`, `char32_t` 这几种类型。

###  非类型模板参数的 `auto `（ c++17）

####  类型参数模板

```c++
template <typename T, typename U>
	auto add(T t, U u) {
	return t + u;
}
```

```c++ 
        template <typename T, int BufSize>
        class buffer_t {
        public:
            T& alloc();
            void free(T& item);
        private:
            T data[BufSize];
        };

        buffer_t<int, 100> buf; // 100 作为模板参数
```

```c++
// c++17
template <auto value> void foo() {
	return;
}

void test_auto() {
	foo<10>();     // value 被推导为 int 类型
};
```

###  `std::variant<>` ( c++17 )

```c++
        template <size_t n, typename... Args>
        std::variant<Args...> _tuple_index(size_t i, const std::tuple<Args...>& tpl) {
            if (i == n)
                return std::get<n>(tpl);
            else if (n == sizeof...(Args) - 1)
                throw std::out_of_range("越界.");
            else
                return _tuple_index<(n < sizeof...(Args) - 1 ? n + 1 : 0)>(i, tpl);
        }
        template <typename... Args>
        std::variant<Args...> tuple_index(size_t i, const std::tuple<Args...>& tpl) {
            return _tuple_index<0>(i, tpl);
        }
```

###  结构化绑定 ( c++17 )

```c++
std::tuple<int, double, std::string> f() {
    return std::make_tuple(1, 2.3, "456");
}

void test_struct() {
    auto [x, y, z] = f(); // x,y,z 分别被推导为int,double,std::string
}
```

### 变量声明的强化

在 `if` 或者 `switch` 语句声明一个临时变量。

```c++
void test_s()
{
	if (auto p = 10; p == 10) {
		//...
	}
	else {
		//...
	}
}
```

##  TODO: c++17 整理

学习链接：https://chenxiaowei.gitbook.io/c-17-stl-cook-book/

