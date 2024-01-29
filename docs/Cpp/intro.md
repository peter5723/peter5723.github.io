# C++
 
我会试图跟着 CS106 这门课的节奏走。课程链接：<https://web.stanford.edu/class/cs106l>

## Lecture 1

课程的第一课，不多说。

## Lecture 2: Types and Structs

### 数据基本类型

和 C 语言类似，也就是 int、char、double 这类的。

有趣的是，C++ 可以使用 `auto` 关键字来让编译器自动指定变量的类型

```cpp
auto a = 3;
auto b = 4.3;
auto c = 'x';
auto d = "Hello";
```

### namespace

参考链接：

<https://zhuanlan.zhihu.com/p/126481010>

<https://www.runoob.com/cplusplus/cpp-namespaces.html>

在 C++ 中，名称（name）可以是符号常量、变量、函数、结构、枚举、类和对象等等。工程越大，名称互相冲突性的可能性越大。另外使用多个厂商的类库时，也可能导致名称冲突。于是 C++ 就引入了 namespace 名称空间，我们这里就简单的介绍一下 这个概念。

名称空间的定义就是：
```cpp
namespace name {
   // 代码声明
   // 函数、变量、类等等
}
```

定义好后，要使用这个名称空间的变量或函数，这样即可：
```cpp
name::code;  // code 可以是变量或函数
```

看下面这个例子吧：
```cpp
#include <iostream>
using namespace std;
 
// 第一个命名空间
namespace first_space{
   void func(){
      cout << "Inside first_space" << endl;
   }
}
// 第二个命名空间
namespace second_space{
   void func(){
      cout << "Inside second_space" << endl;
   }
}
int main ()
{
 
   // 调用第一个命名空间中的函数
   first_space::func();
   
   // 调用第二个命名空间中的函数
   second_space::func(); 
 
   return 0;
}
```

输出：
```
Inside first_space
Inside second_space
```

很好理解，对吧。

然后可以使用 `using` 命令来偷懒，`using namespace name`，使用命名空间 `name`时，就可以不用在前面加上命名空间的名称，编译器会帮忙搞定。

```cpp
#include <iostream>
using namespace std;
 
// 第一个命名空间
namespace first_space{
   void func(){
      cout << "Inside first_space" << endl;
   }
}
// 第二个命名空间
namespace second_space{
   void func(){
      cout << "Inside second_space" << endl;
   }
}
using namespace first_space;
int main ()
{
 
   // 调用第一个命名空间中的函数
   func();
   
   return 0;
}
```
输出：
```
Inside first_space
```

最常用的名字空间就是 std 了，它是 STL 标准模版库的名字空间。C++ 的输入输出函数 `cin` 和 `cout` 就是 STL 库的。STL 库很强大，后面会多次用到。

下面输出一下 hello world 看看
```cpp
#include <iostram>

int main() {
    std::cout << "Hello, world!" << std::endl;
    return 0; 
}
```

相当于：
```cpp
#include <iostram>
using namespace std;
int main() {
    cout << "Hello, world!" << endl;
    return 0; 
}
```

### overload

学习 Java 的时候已经使用过函数重载 overload 了，C++ 的也是类似的。函数可以重名，只要保证输入参数的类型或数量不同即可，例子如下：
```cpp
#include <iostream>
using namespace std;

int half(int x) {
    return x/2;
}

double half(double x, int divisor=2) {
    // C++ 和 python 一样，支持默认参数！
    return x/divisor;
}

int main() {
	cout << half(3) << endl;
	cout << half(3.0) << endl;
	cout << half(3.0, 3) << endl;
	return 0;
} 
```

输出：
```
1
1.5
1
```

### struct and pairs

C++ 中 struct 的使用和 C 相比更加方便了。例子如下：
```cpp
struct Student {
    std::string name;
    std::string state;
    int age;
}

Student s; //C 语言还要 typedef 一下
s.name = "Haven";
s.state = "AR";
s.age = 32;

// 初始化也可以写成：
Student s = {"Haven","AR",32};
```

STL 库中有一个 pair 模版，可以用来快速使用构造结构体。
```cpp
std::pair<int, string> numSuffix = {1, "st"};
numSuffix.first; //1
numSuffix.second; //"st"
```

`<>` 里的内容用来指定两个数据的类型，非常灵活。
此外，可以用 `make_pair` 函数快速生成 pair，连定义都省了。
```cpp
make_pair(true, "love");
```

## Lecture 3: Initialization and References

### Initialization

C++ 初始化的方式有两种。第一种是和 C 语言一样，使用等号。

```cpp
int a = 12;
int b = 12.5; // 不会报错
double c = 56.8;
```

你知道的，C++ 中的类型非常多，每种类型的初始化方式都不同，所以 C++11 又引入了新的初始化方式：Uniform initialization（统一初始化），它使用 `{}` 来初始化。

```cpp
int a{12};
int b{12.5}; // Uniform initialization 非常关心变量类型，这里就会报错

std::map<std::string, int> ages{
    {"Alice", 25},
    {"Bob", 30},
    {"Charlie",35}
}; // 对 map 初始化

std::vector<int> numbers{1,2,3,4,5}; // 对 vector 初始化

struct Student {
    string name;
    string state;
    int age;
};

Student s{"Haven", "AR", 32}; // 对 struct 初始化
Student s = {"Haven","AR",32}; // 用 = 来初始化
```

### Structured Binding

Structured Binding，中文名所谓的“结构化绑定”，是什么意思呢？这是 C++17 引入的特性。

先前我们的函数要返回多个参数的时候，应该怎么办呢？定义一个结构体即可：（例子来自<https://zhuanlan.zhihu.com/p/139140185>）
```c++
struct Person
{
    std::string name;
    int age;
};

Person CreatePerson()
{
    return {"小岛秀夫赛高", 56};
};

int main()
{
    auto person = CreatePerson();
    std::cout << person.name << " " << person.age << '\n';
}
```

这个函数就可以返回人的姓名和年龄。但是有时候这个结构体只用一次，还专门去定义，就浪费了。那么我们看看下面的方式：
```c++
std::tuple<std::string, int> CreatePerson()
{
    return { "小岛秀夫赛高", 56 };
} 

int main()
{
    auto[name, age] = CreatePerson();
    // 和下面两种写法等价，但是下面要对 name、age 初始化，auto[name, age] 把初始化的工作也做了。

    // std::tie(name, age) = CreatePerson();

    // std::string &name = std::get<0>(person);    
    // int age           = std::get<1>(person);
    std::cout << name << " " << age << "\n";
}
```

利用 C++11 引入的新模版 tuple 来写这个函数，`<>` 里的内容是类型，`auto[name, age]` 就会自动根据括号里的类型来初始化变量。

### Reference

reference 引用，是 C++ 中重要的概念之一。使用 `&` 运算符来进行。 

可以把引用看成是指针的方便版本。

```cpp
int num = 5;
int& ref = num;
ref = 10;
cout << num << endl;
```

上面这个例子会输出 10。在 C 语言中，你可以这么实现：

```c
int num = 5;
int *p = &num;
*p = 10;
printf("%d", *p);
```

如果在 C++ 中想要函数改变变量本身而不是传入形参，也就不用传入指针了，使用对应的引用类型即可：

```cpp
void squareN(int& n) {
    n = n * n;
}

int main() {
    int num = 2;
    squareN(num);
    cout << num << endl;
}
```

引用值和原始值指向同一个地址，所以改变任何一个都可以。

下面看一个比较难的例子：
```cpp
void shift(vector<pair<int, int>> &nums) {
    for (auto [num1, num2]: nums) {
        num1++;
        num2++;
    }
}
```
虽然上面这个函数的本意是希望将数组中的所有值都自增，但是有一个较难发觉的地方是，for 循环中的 `:` 运算符本质上也是一个函数，它是返回一个新值，即为解包为单个元素的结果，而并非数组中的元素本身（我想起来和 Java 中的 Iterator 函数类似）。所以上面的函数不会产生任何结果，正确的做法是在 `auto` 后面增加 `&` 表示引用类型。

另外，实际上上面也用到了 structure binding 这个性质，还记得吗？

```cpp
void shift(vector<pair<int, int>> &nums) {
    for (auto& [num1, num2]: nums) {
        num1++;
        num2++;
    }
}
```

### l-values and r-values

左值和右值。左值可以在赋值运算符 = 的左边或者右边，比如一个变量。而右值只能在 = 的右边，比如常量。

会想到 C 语言中一个经典的问题：如何快速避免在 if 表达式中将`x==0` 写成 `x=0` 这种问题？ 答案很简单，就是写成 `0==x` 的形式，如果写成 `0=x`，编译器自然就会报错。这就是利用了右值不能被赋值的原理了。

然后就是在传参数的时候，右值是不可以传到引用类型的变量里去的，比如对于：
```c
void squareN(int& n) {
    n = n * n;
}
```
当然不能写成 `squareN(2)` 的形式。还是比较好理解的。


### 常量

常量 const。

```cpp
const int a = 1;
```

只有一点是新的，常量不能被非常量引用引用，所以 `int& b = a` 会导致错误，而 `const int &b = a` 则语法正确。
