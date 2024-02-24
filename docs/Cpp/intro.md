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


## Lecture 4: Streams

参考链接：<https://www.runoob.com/cplusplus/cpp-basic-input-output.html>

### 介绍
Streams 输入输出流这个概念是 C++ 中的重要概念。知道了它就可以使用高贵的 `cin` 和 `cout`，抛弃可怜的 `scanf` 和 `printf` 了。

那么到底什么是 stream 呢？stream 就是 C++ 为输入输出数据提供的一种抽象。抽象的作用就是提供高层的接口，避免直接和底层打交道。stream 的本质是字节序列，从 IO 设备读出或者写入 IO 设备。

在 C++ 中，使用 `iostream` 库来实现流，提供输入输出机制。`iostream` 中有两个基础类，`istream` 和 `ostream`，分别表示输入流和输出流。在标准库中，又定义了 4 个对象：`cin`，`cout`，`cerr`，`clog`。`cin` 是 `istream` 类型的实例，`cout` 和后面两个是 `ostream` 类型的实例。

### 输出流
还是来看一个例子吧：
```cpp
#include<iostream>
int main() 
{
    std::cout << "hello, world" << std::endl;
    return 0;
}
```

上面的例子，很简单，就是打印 hello, world 到屏幕上。但是有一些细节需要说明。`std::cout << "hello, world" << std::endl` 本质上是一个表达式，`<<` 是一个输出运算符。（从现在开始要认识到 `cout` 不是一个函数了。） 

`<<` 运算符接受两个运算对象，左侧的是一个 `ostream` 对象，右侧的运算对象是要打印的值。这个运算符将给定的值（右侧）写入到给定的 `ostream` 对象（左侧）中，就是说，计算结果就是写入给定值的左侧的 `ostream` 对象。 

所以我们可以把表达式写成
```cpp
(std::cout << "hello, world") << std::endl;
```

或者是
```cpp
std::cout << "hello, world";
std::cout << std::endl;
```

得到的输出都是相同的。

那么这个 `std::endl` 又是怎么回事呢？`endl` 是一个被称为操纵符的特殊值，它的效果是结束当前行，并将缓冲区的内容全部刷新到设备中，清空缓冲区。

### 输入流
上面简单介绍了一下输出流，其实对于输入流也类似的。

```cpp
int v1 = 0, v2 = 0;
std::cin >> v1 >> v2;
```

`>>` 是输入运算符，接受 istream 类为左侧运算对象，将数据输入到右侧对象中，返回左侧对象作为运算结果。`>>` 读取 istream 的内容时，读到空格停止（和 `scanf` 一样）。

stream 提供了一种 universal 的方式来处理输入输出。

### IO 类

除 `istream` 和 `ostream` 之外，标准库还定义了其他的一些 IO 类型，定义在三个独立的头文件中：`iostream` 定义了用于读写流的基本类型，`fstream` 定义了读写文件的类型，`sstream` 定义了读写字符串对象的类型。实际上，这三种 IO 类型是通过继承机制实现的，关系如下图所示：

![pF1JSmt.jpg](https://s11.ax1x.com/2024/02/06/pF1JSmt.jpg)

`stringstream`和 `fstream` 都是继承自 `iostream` 的。


先看一个 `stringstream` 的例子：
```cpp
std::string initial_quote = "Bjarne Stroustrup C makes it easy to shoot yourself in the foot"; 

std::stringstream ss;
ss << initial_quote;
// stringstream 也可以像 cout 那样用输出运算符来赋值
std::string first;
std::string last;
std::string language, extracted_quote;

ss >> first >> last >> language;
// 可以用输入运算符对字符串赋值，就像 cin 那样，每一次赋值到空格结束
std::getline(ss, extracted_quote);
std::cout << first << " " << last <<" said this: " << language << " " << extracted_quote << std::endl;
```

如果直接用输入运算符，那么到空格就截止了。有时我们需要读入一整行，就像上面的 `extracted_quote` 那样，这时候就要使用 `getline` 函数。来看一下 `getline` 函数的原型：

```cpp
istream& getline(istream& is, string& str, char delim)
```

`getline()` 函数读取一个输入流对象 `is`，直到遇到 `delim`  字符，并将读取结果储存在 `str` 中。默认的 `delim` 就是换行符 `\n`。注意：`getline()`  函数会消耗换行符。

然后我们再看一下文件流的例子。

```cpp
int main() {
    std::ofstream ofs("hello.txt");
    if (ofs.is_open()) {
        ofs << "Heloo CS106L!" << '\n';
    }
    ofs.close();
    ofs << "this won't get written";

    ofs.open("hello.txt");
    ofs << "this will though! It’s open again";
    return 0;
}
```

文件流的输入输出和 IO 也是类似的，就好像 `fprintf` 和 `printf` 之间的关系一样。


### 格式化输出

例子抄自这个链接：<https://zhuanlan.zhihu.com/p/583649384>

C++ 怎么用 `cout` 做到类似于 `printf` 的格式化输出呢？C++ 的 `ostream` 库中定义了大量的“流操纵算子”来控制输出。下面介绍一下：

#### 整数不同进制输出

输出整数时用的流操纵算子有 `dec`，`oct`，`hex` 等等。在更改进制显示方式之后，系统默认后面使用该方式显示数据，如果有显示为其他进制形式的需要重新设置进制显示方式。

```cpp
cout << "cout以不同进制显示整数***********************************************************" << endl;
/*cout以不同进制显示整数*/
i = 90;
cout << "以十进制显示i：" << dec << i << endl;//以十进制显示
cout << "以八进制显示i：" << oct << i << endl;//以八进制显示
cout << "以16进制显示i：" << hex << i << endl;//以16进制显示
//如需更改为十进制显示方式，则可以使用以下方式
cout.setf(ios_base::dec);
cout << "以十进制显示i：" << i << endl << endl;
/*bool数据类型显示*/
cout << "bool数据类型显示" << endl;
bool is_true = true;
cout.setf(ios_base::boolalpha);//可以显示为true或false
cout << "is_true = " << is_true << endl;
is_true = false;
cout << "is_true = " << is_true << endl << endl;
```

输出：
```
cout以不同进制显示整数***********************************************************
以十进制显示i：90
以八进制显示i：132
以16进制显示i：5a
以十进制显示i：90

bool数据类型显示
is_true = true
is_true = false
```

#### 设置宽度

`cout` 使用 `width` 方法可以设置宽度。`width()` 只影响下一次的cout，然后字段宽度将恢复默认值。C++ 不会截断数据，如果显示字段宽度小于数字宽度，C++ 将增宽字段，以容纳该数据，如最后一个例子。

```cpp
cout << "width()***************************************************************************" << endl;
int w = cout.width(30);
cout << "default field width = " << w << ":\n";
cout.width(5);
cout << "N" << ':';
cout.width(8);
cout << "N * N" << ":\n";
for (long i = 1; i <= 100; i *= 10)
{
    cout.width(5);
    cout << i << ':';
    cout.width(8);
    cout << i * i << ":\n";
}
cout.width(5);
cout << 9.888889999 << endl;//将不会截断
```

运行结果

```
width()***************************************************************************
        default field width = 0:
    N:   N * N:
    1:       1:
   10:     100:
  100:   10000:
9.88889
```

#### 更改填充字符

`fill()` 方法更改空位填充的字符（默认是空格），也是全局设定，更改后系统都用这个字符填充。

```cpp
cout << "更改填充字符****************************************************************************" << endl;
cout.fill('*');
const char* staff[2] = { "Waldo Whipsnade", "Wilmarie Wooper" };
long bonus[2] = { 900, 1350 };
for (int i = 0; i < 2; i++)
{
    cout << staff[i] << ": $";
    cout.width(7);
    cout << bonus[i] << "\n";
}
```

输出：
```
更改填充字符****************************************************************************
Waldo Whipsnade: $****900
Wilmarie Wooper: $***1350
```

#### 更改显示精度
`precision` 方法可以更改显示精度，也是全局设置。

```cpp
cout << "设置小数显示精度：precision()***********************************************************" << endl;
double j = 3333.1415926;
/*设置显示有效数字位数*/
cout << "默认情况显示(6位)：" << j << endl;//输出浮点数时，默认情况下保留六位有效数字
cout.precision(9);//可以使用cout.precision(n)设置输出浮点数时保留n位有效数字
cout << "设置有效数字位数为9位时：" << j << endl;
cout.precision(3);//当有效数字位数小于整数有效数字位数时，使用科学计数法显示
cout << "设置有效数字位数为3位时：" << j << endl << endl;
```

输出
```
设置小数显示精度：precision()***********************************************************
默认情况显示(6位)：3333.14
设置有效数字位数为9位时：3333.14159
设置有效数字位数为3位时：3.33e+03
```

总的来说感觉还是 `printf()` 好用？但是人要学会接受新事物。