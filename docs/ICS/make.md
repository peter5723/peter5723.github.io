# GNU Make

昨天调整了字体，但是不知道为什么搜索框的字体不会改变，然后又废了很大的功夫加了一个 css 样式，改了搜索框的字体和其余部分的字体相一致。主要是因为对前端的部分我还很不熟悉，都是摸黑操作。巴别塔里的战士都喜欢音乐，没有人不喜欢美。难得有机会做成一个网站，想做的好看一点。

说回 make。什么是 make？简单的说，make 用来构建 C 语言的项目。首先你要下载 make。在项目文件夹里必须要有 `Makefile` 文件，make 才能运行。`Makefile` 文件里面有什么呢？

## rule
`Makefile`里面必须包含 `rules`。`rules` 由这三部分构成：`target`, `prerequisite`, `recipe`。

- `target` 是最终生成的目标文件名称
- `prerequisite` 是生成 `target` 所需要的文件
- `recipe` 是需要执行的操作

差不多是这种形式：

```Makefile
target ... : prerequisites ...
recipe
...
...
```

下面是一个示例 `Makefile`，它包含了好几个 `rule`，并最终生成名为 `edit` 的文件。注意，`clean` 是一个比较特殊的 `target`，它不需要 `prerequisites`。运行 `make`，执行前面的所有操作；运行 `make clean`，是完成 `clean` 这个 `target`，一般是用来清楚所有编译结果。

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

## variable

像上面这样写出所有文件将会非常冗长，所以引入了 `variable` 这个概念，就是变量。
变量可以在文件开头先定义好，后面要用的时候使用 `$(var)` 就可以引用我们想要的变量了，看下面的例子。
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

