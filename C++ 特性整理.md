##  C++ 特性整理

###  nullptr

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

###  constexpr

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

###  auto 

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

常见用法

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

`decltype` 关键字为了解决 `auto` 关键字只能对变量进行类型推导的缺陷。 用法类似于 `sizeof` 

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

