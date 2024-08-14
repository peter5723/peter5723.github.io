# GNU Make

昨天调整了字体，但是不知道为什么搜索框的字体不会改变，然后又费了很大的功夫加了一个 css 样式，改了搜索框的字体和其余部分的字体相一致。主要是因为对前端的部分我还很不熟悉，都是摸黑操作。巴别塔里的战士都喜欢音乐，没有人不喜欢美。难得有机会做成一个网站，想做的好看一点。

说回 make。什么是 make？简单的说，make 用来构建 C 语言的项目。首先你要下载 make。在项目文件夹里必须要有 `Makefile` 文件，make 才能运行。`Makefile` 文件里面有什么呢？

## Makefile 介绍
摘自官方文档：Makefiles contain five kinds of things: explicit rules, implicit rules, variable definitions,
directives, and comments.

### rule
`Makefile`里面必须包含 `rules`。`rules` 由这三部分构成：`target`, `prerequisite`, `recipe`。

- `target` 是最终生成的目标文件名称
- `prerequisite` 是生成 `target` 所需要的文件，称为前置条件
- `recipe` 是需要执行的操作。由于 make 就是在 shell 中执行，所以 `recipe` 就是一系列的 shell 语句（当然后期写复杂了也会包含一些结构，比如分支、循环、函数等）

差不多是这种形式：

```Makefile
target ... : prerequisites ...
recipe
...
...
```

下面是一个示例 `Makefile`，它包含了好几个 `rule`，并最终生成名为 `edit` 的文件。注意，`clean` 是一个比较特殊的 `target`，它不需要 `prerequisites`。运行 `make`，执行前面的所有操作；运行 `make clean`，是完成 `clean` 这个 `target`，一般是用来清除所有编译结果。

```Makefile
edit : main.o kbd.o command.o display.o \
insert.o search.o files.o utils.o
cc -o edit main.o kbd.o command.o display.o \
insert.o search.o files.o utils.o
main.o : main.c defs.h
cc -c main.c
kbd.o : kbd.c defs.h command.h
cc -c kbd.c
command.o : command.c defs.h command.h
cc -c command.c
display.o : display.c defs.h buffer.h
cc -c display.c
insert.o : insert.c defs.h buffer.h
cc -c insert.c
search.o : search.c defs.h buffer.h
cc -c search.c
files.o : files.c defs.h buffer.h command.h
cc -c files.c
utils.o : utils.c defs.h
cc -c utils.c
clean :
rm edit main.o kbd.o command.o display.o \
insert.o search.o files.o utils.o
```

### variable

像上面这样写出所有文件将会非常冗长，所以引入了 `variable` 这个概念，就是变量。
变量可以在文件开头先定义好，后面要用的时候使用 `$(var)` 就可以引用我们想要的变量了，看下面的例子。这个变量，和 c 语言中的宏非常相似。

另外，下面的例子用到的省略：如果 `target` 是二进制文件，就只需要给出 `prerequisites`,
gcc 会找到同名的 c 文件进行编译。

```Makefile
CC = gcc
CFLAGS = -Wall
objects = main.o kbd.o command.o display.o \
insert.o search.o files.o utils.o
edit : $(objects)
$(CC) $(CFLAGS) -o edit $(objects)
main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h
.PHONY : clean
clean :
rm edit $(objects)
```

### comments
comment 就是注释，注释很简单，用 `#` 即可注释。

### directive
directive 就是指令，实现特殊的事情，比如 `include`.

#### include
`include` 让 make 暂停，引用外部的文件。比如：
`include foo *.mk $(bar)` 引用 `foo`, 所有 `.mk` 为后缀名的文件
和变量 `$(bar)` 对应的文件。


### make的工作方式
1. 读入所有的Makefile。
2. 读入被include的其它Makefile。
3. 初始化文件中的变量。
4. 推导隐式规则，并分析所有规则。
5. 为所有的目标文件创建依赖关系链。
6. 根据依赖关系，决定哪些目标要重新生成。
7. 执行生成命令。


## Makefile 编写

前面已经初步了解了 `make`，下面可以更详细地看看。

一个 `Makefile` 文件里面会包含好几个 rules。除了默认第一个是最终的项目文件，顺序并不重要。

怎么运行？如果指令指定了某一个 target：`make sometarget`，那么就生成那个 target。如果指令是 `make`，就生成 `Makefile` 中的第一个 target。

下面也不知道按什么顺序介绍，就随便写吧。

### 回声
正常情况下，make 会打印每条命令（包括注释），然后再执行，这就叫做回声（echoing）。

可以使用 `@` 来忽略打印的输出。一般是用于注释和显示的 `echo` 命令。

```Makefile
test:
    @# 这是测试
    @echo TODO
```


### 自动变量
看 `Makefile` 文件的时候，经常会看到一些奇怪的符号，比如 `$@` 和 `$<` 等等，它们都是自动变量（automatic variables），它们的值与当前规则有关。哈哈，实际上 `$` 打头的就是变量，下面详细介绍一下。

1. `$@`：指代当前目标，就是 Make 命令当前构建的那个目标。比如
    ```Makefile
    a.txt b.txt:
    touch $@
    ```
    和
    ```
    a.txt:
        touch a.txt
    b.txt:
        touch b.txt
    ```
    等价

2. `$<`：指代第一个前置条件。比如
    ```Makefile
    a.txt: b.txt c.txt
    cp $< $@
    ```
    和
    ```Makefile
    a.txt: b.txt c.txt
    cp b.txt a.txt
    ```
    等价
3. `$?`：指代比目标更新的所有前置条件，（默认）之间以空格分隔。
4. `$^`： 指代所有前置条件。比如，规则为 `t: p1 p2`，那么 `$^` 就指代 `p1 p2` 。
### 匹配（%）
`%` 匹配任意个字符，是 make 特有的匹配，正则表达式里没有。

有很多博客会单独拎出来 `%` 这个符号，讲它用来匹配任意个字符。然后就很容易和正则表达式的 `*` 搞混，搞不清楚它们的区别是什么。可以看[这个](https://blog.csdn.net/qq_44139306/article/details/130395123)来辨析。

按照手册，不妨把 `%` 看成一个语法结构的组成部分，也就是 **static pattern rules**，静态匹配模式。语法如下：
```makefile
targets ...: target-pattern: prereq-patterns ...
recipe
```

先来看个例子吧：
```makefile
objects = foo.o bar.o

all: $(objects)
$(objects): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
```

规则就是首先 target-pattern（即 `%.o`） 在 target（即 `$(objects)`） 里面找到对应的模式，存储到 `%` 中，然后将这个模式替换 prereq-patterns（即 `%.c`） 中的 `%`。这个语法一下子可以处理很多文件，故而备受欢迎。

实际也很常用的是用 `%` 编写 implicit rules（隐式规则）中的模式规则。
下面是一个例子，编译文件夹里的所有 c 文件
```Makefile
all:$(subst .c,.o,$(wildcard *.c))
%.o:%.c
    gcc -o $@ $<
```
注意，此时 `%` 是从上下文寻找，也就是一定要上下文出现过才能匹配。这也就是为什么删除 `all:$(subst .c,.o,$(wildcard *.c))` 会导致 targets not found 这个编译错误的原因。

其实总结起来很简单, 就是从 Makefile 文件的展开之后的上下文进行寻找.


### 伪目标 （prony targets）
伪目标同样也是 target，最大的不同是它不生成新的文件，比如一开始提到的 `clean`。

伪目标有以下几种情况：

1. 没有依赖（prerequisites），比如经典的 clean 例子：
    ```makefile
    .PHONY : clean
    clean :
        rm *.o temp
    ```

    此时目标就是单纯的执行 shell 语句。注意，`.PHONY : clean` 显式声明某一个目标是伪目标。不声明也可以，一般 make 能够自动推导出来。

2. 依赖也是 targets，看下面的例子：
    ```makefile
    all : prog1 prog2 prog3
    .PHONY : all

    prog1 : prog1.o utils.o
        cc -o prog1 prog1.o utils.o

    prog2 : prog2.o
        cc -o prog2 prog2.o

    prog3 : prog3.o sort.o utils.o
        cc -o prog3 prog3.o sort.o utils.o
    ```

    此时，`all` 就是一个伪目标，make 会实现后面作为依赖的多个目标。实际上，介绍 `%` 的那一节，举的例子中的 `all` 都是类似的伪目标。

### 函数 （functions）
函数也像是一种变量，格式如下
```makefile
$(<function> <arguments>)
```


下面介绍一些常见的函数

```makefile
$(subst from, to, text)
# 将 text 中的 from 都替换成 to
$(subst ee,EE,feet on the street)
# fEEt on the strEEt
```

```makefile
$(patsubst pattern,replacement,text)
# 搜索 text 中以空格分开的单词, 将符合 pantern 的子串替换为 replacement. 常用模式通配符 % 来匹配

$(patsubst %.c,%.o,x.c.c bar.c)
# x.c.o bar.o
```

```makefile
$(filter pattern...,text)
# 只保留 text 中符合 pattern 的子串.

sources := foo.c bar.c baz.s ugh.h
foo: $(sources)
cc $(filter %.c %.s,$(sources)) -o foo
# $(filter %.c %.s,$(sources)) =  foo.c bar.c baz.s
```

```makefile
$(wildcard pattern)
# 在当前文件夹下匹配所有符合模式的文件名并返回使用空格分开的字符串

objects := $(patsubst %.c,%.o,$(wildcard *.c))
foo : $(objects)
cc -o foo $(objects)
# 利用 make 的隐含规则来编译 C 的源文件
```

```makefile
# 一系列文件名操作

$(dir names...)
# 返回文件的目录名
$(dir src/foo.c hacks)
# src/ ./

$(notdir names...)
# 返回文件去掉目录部分后的名称. 若文件是目录, 返回空字符串.
$(notdir src/foo.c hacks ./lib)
# foo.c hacks


$(suffix names...)
# 返回文件的后缀名. 若没有后缀, 返回空字符串.
$(suffix src/foo.c src-1.0/bar.c hacks)
# .c .c

$(basename names...)
# 返回文件除后缀名以外的部分.
$(basename src/foo.c src-1.0/bar hacks)
# src/foo src-1.0/bar hacks
```

### 分支语句
很像 c 语言的条件宏语句，也是读取 `makefile` 文件就做了判断。
下面是一个例子：
```makefile
libs_for_gcc = -lgnu
normal_libs =

ifeq ($(CC),gcc)
    libs=$(libs_for_gcc)
else
    libs=$(normal_libs)
endif

foo: $(objects)
    $(CC) -o foo $(objects) $(libs)
```
### 等号辨析与追加

### 命令行

在运行 make 的时候，可以添加命令行参数。

-n：只打印所有要执行的命令，却并不执行。这个在查看 Makefile 在做什么的时候特别有用

-B：无条件执行所有命令
## 参考

别人的博客：<https://www.ruanyifeng.com/blog/2015/02/make.html>

官方文档：<https://www.gnu.org/software/make/manual/>

一份中文参考教程：<https://seisman.github.io/how-to-write-makefile/index.html>

另外，实际上你可以发现，很多教程都是大量参考官方文档的，所以能看官方文档就看官方文档吧。这就叫做 RTFM 吧（:rofl:）

## Emoji in Markdown
markdown 怎么使用 emoji 呢？

教程：

<https://markdown.com.cn/extended-syntax/emoji.html>

所有的 emoji 集合：

<https://gist.github.com/rxaviers/7360908>

<https://awes0mem4n.github.io/emojis-github.html>


因为网站弄得很好看（好看吗，缺陷挺多的），所以想把文档也弄得好看一点；但是很麻烦，比如说英文与数字和中文要空格；文档最好只使用二三四级标题，因为五级标题比正文还小；熟练使用 markdown，文档的结构怎么安排，什么时候用代码符号，文章多了以后怎么排网站的结构，等等，现在就很晚了，只是大致了解了 make，还有一些内容没补全，强迫症犯了就要命。我以后要放飞自我了，不会遵守规范了，哈哈。



为什么要做一个网站呢，不仅是整理知识，也希望有一个地方能够“表达”。但是，我其实并没有什么勇气，展示自己的感受，哪怕是仅仅是给自己看，哪怕仅仅是写出来。所以大部分的笔记本，肯定是以“说明”为主，否则逃不掉被我扔掉的命运:rofl:。建一个公众号，有人在看，我怎么写想法呢？建一个网站总没有人看了吧！我也需要表达，不然我会窒息。我的表达不会多么有逻辑，我也不会像写议论文或者说明文那样构建逻辑，就是乱写一通。希望以后这个网站不会变成纯“说明”的博客吧，哈哈。
