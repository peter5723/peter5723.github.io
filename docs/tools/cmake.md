# cmake

cmake 的官方教程居然全是练习，太搞了。

## Step1

### Exercise 1 - Building a Basic Project

The most basic CMake project is an executable built from a single source code file. For simple projects like this, a CMakeLists.txt file with three commands is all that is required.

Any project's top most CMakeLists.txt must start by specifying a minimum CMake version using the `cmake_minimum_required()` command. **This establishes policy settings and ensures that the following CMake functions are run with a compatible version of CMake.**

To start a project, **we use the `project()` command to set the project name.** This call is required with every project and should be called soon after `cmake_minimum_required()`. **As we will see later, this command can also be used to specify other project level information such as the language or version number.**

Finally, the `add_executable()` command tells CMake to **create an executable using the specified source code files.**


所以，一份最简单的 Cmakelist.txt 就是这样的：

```cmake
cmake_minimum_required(VERSION 3.10)

project(Tutorial)

add_executable(Tutorial tutorial.cxx)
```

在 windows 系统上总是要处理路径的问题，所以我推荐在 linux 系统上运行。

在 build 文件夹下运行 `cmake ../source`，就可以生成一个 Makefile 文件。

然后运行 `cmake --build .` 或者 `make`，就可以编译链接，生成可执行文件。之后如果更新了 Cmakelist，`cmake --build .`即可


### Exercise 2 - Specifying the C++ Standard

CMake 的特殊变量都是以 `CMAKE_` 打头的。这个练习中我们要用的变量是 `CMAKE_CXX_STANDARD` 和 `CMAKE_CXX_STANDARD_REQUIRED`。

用 `set()` 可以指定变量的值。

我们指定 C++ 标准为 C++11：

```
cmake_minimum_required(VERSION 3.10)

project(Tutorial)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

add_executable(Tutorial tutorial.cxx)
```

### Exercise 3 - Adding a Version Number and Configured Header File

```cmake
project(Tutorial VERSION 1.0)
```

就可以指定 target 的变量 VERSION.


```
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

利用 `configure_file(input output)` 指令，我们可以实现 cmake 和源代码文件的联动。

output path is to the output file or directory. A relative path is treated with respect to the value of CMAKE_CURRENT_BINARY_DIR. 即输出的头文件是在 build 文件夹中。


输入 TutorialConfig.h.in 类似这样，cmake 中的变量形如 `@var@`。
```c
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

输出的 .h 文件就是
```c
#define Tutorial_VERSION_MAJOR 1
#define Tutorial_VERSION_MINOR 0
```

最后添加头文件路径——不然默认在源代码路径找，找不到我们新生成的 .h 文件。

```cmake
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
```

整个 Cmakelist 如下所示：

```cmake
cmake_minimum_required(VERSION 3.10)
project(Tutorial VERSION 1.0)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
configure_file(TutorialConfig.h.in TutorialConfig.h)
add_executable(Tutorial tutorial.cxx)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
```


## Step2

学一下更大些项目的组织。

首先，如果我们的项目有多个子目录，那么用 `add_subdirectory` 就可以在构建过程中包含另一个目录的 Cmakelist。不同的子目录下，我们可以用 `add_library()` 构建静态库：

```
add_library(MathFunctions MathFunctions.cxx)
```

注意，target 是 `add_executable` 或者 `add_library` 生成的结果名字。


然后用 `target_link_libraries()` 进行库的链接，将库链接到 target 上：

```
target_link_libraries(Tutorial PUBLIC MathFunctions)
```

更新头文件搜索路径：
```
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           "${PROJECT_SOURCE_DIR}/MathFunctions"
                           )
```


然后看看下面的语句：

```
if(USE_MYMATH)
target_compile_definitions(MathFunctions PUBLIC USE_MYMATH)
endif()
```

首先注意一下 if 语句的写法，然后 `target_compile_definitions()` 用于在源代码中添加宏定义。


### 例子

这是一个最简单的例子。我们的项目结构是这样：

```
project_root/
├── CMakeLists.txt
├── src/
│   ├── CMakeLists.txt
│   └── main.cpp
└── lib/
    ├── CMakeLists.txt
    └── mylib.cpp
```


`project_root/CMakeLists.txt` 文件内容：
```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# 添加子目录
add_subdirectory(src)
add_subdirectory(lib)
```

`project_root/src/CMakeLists.txt` 文件内容：
```cmake
add_executable(MyExecutable main.cpp)

# 链接库
target_link_libraries(MyExecutable MyLibrary)
```

`project_root/lib/CMakeLists.txt` 文件内容：
```cmake
add_library(MyLibrary mylib.cpp)
```

这样：可以将项目分成多个子目录，每个子目录都有自己的构建规则和依赖关系。
