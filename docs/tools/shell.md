# Shell

## 常用指令

这里整理 shell 的指令脚本等。


find 使用:
```bash
find . -name "<filename>" # 在当前文件夹下查找指定文件名
```

grep 真的太常用了：
```bash
grep -r "<you want>" . # 在当前文件夹下搜索指定字符串输出的位置
```

## shell 脚本

参考资料:

[鸟哥私房菜第十二章](https://linux.vbird.org/linux_basic/centos7/0340bashshell-scripts.php#script_be)

### 1. 第一个脚本
先来看第一个 shell 脚本(script) 吧.
```bash
#!/bin/bash
# Program:
#       This program shows "Hello World!" in your screen.
# History:
# 2015/07/16	VBird	First release
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
echo -e "Hello World! \a \n"
exit $?
```

介绍一下脚本的结构:

1. 第一行 `#!/bin/bash` 在宣告这个脚本使用的 shell 名称. 后面的 # 是注释用途.
2. 设定好变量的值, 如 PATH 与 LANG.
3. 主要程式: 这个例子, 就是 echo 那一行
4. 执行结果: $? 是执行结果, 若成功执行, 则返回 0.

### 2. 一些例子

#### 2.1 输入输出
```bash
#!/bin/bash
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

read -p "Please input your first name: " firstname      
read -p "Please input your last name:  " lastname       
echo -e "\nYour full name is: ${firstname} ${lastname}" 
```

注意: read 用法, 读取用户输入, 并存储到变量中. 可选择 -p 选项可跟提示字符串. `read [-p] var`

#### 2.2 利用 date
建立几个空文件, 文件名以用户输入为开头, 以今天/昨天/前天的名称结尾.

```bash
#!/bin/bash
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

echo -e "I will use 'touch' command to create 3 files." # 純粹顯示資訊
read -p "Please input your filename: " fileuser         # 提示使用者輸入

filename=${fileuser:-"filename"}

date1=$(date --date='2 days ago' +%Y%m%d)  # 前兩天的日期
date2=$(date --date='1 days ago' +%Y%m%d)  # 前一天的日期
date3=$(date +%Y%m%d)                      # 今天的日期
file1=${filename}${date1}                  # 底下三行在設定檔名
file2=${filename}${date2}
file3=${filename}${date3}

touch "${file1}"                           # 底下三行在建立檔案
touch "${file2}"
touch "${file3}"
```

上面的脚本有几个要注意的点:
1. `filename=${fileuser:-"filename"}`: `:-` 符号为变量提供默认值. 如果前面的变量未设定或者是空字符串, 那么就用后面的内容来赋值.
2. `date` 指令. 很简单, 直接使用获得当前时间, 也可以添加选项设置特定日期等,  `+%Y%m%d` 设置日期格式. 用法: `date [OPTION]... [+FORMAT]`
3. `touch` 指令: 如果文件不存在, 就创建一个新的以这个文件名命名的空文件; 反之则用当前的时间更新这个文件的访问和修改时间.

#### 2.3 运算 pi
```bash
#!/bin/bash

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH

echo -e "This program will calculate pi value. \n"
echo -e "You should input a float number to calculate pi value.\n"
read -p "The scale number (10~10000) ? " checking
num=${checking:-"10"} 

echo -e "Starting calculate pi value.  Be patient."
time echo "scale=${num}; 4*a(1)" | bc -lq
```

这里的新知识是运算. bash 默认支持整数的运算, `var=$(expression)`, 比如 `echo $((13%3))`. 如果是小数运算, 就要用 bc 指令来运算, 比如 `echo "123.123*55.9" | bc`. 上面的脚本就是利用 bc 指令来计算 pi 的后 scale 位小数.

最后, time 指令会显示后面指令的运行时间.

### 3. 几种执行方式

shell 脚本有好几种执行方式, 一种是像前面那样, 直接运行脚本, `bash script.sh` 或 `chmod u+x script.sh; ./script.sh`; 还有一种就是使用 `source` 命令来执行脚本, `source script.sh`.

两种方式的差别是, 直接运行的脚本, 是在一个新的子 shell 中执行的, 脚本中定义的变量和函数只有在脚本执行的函数存在, 执行完后, 子 shell 关闭, 这些变量, 函数和脚本中的环境改变都不存在了. 而使用 `source` 命令, 则是在当前 shell 环境中执行, 这意味着脚本中定义的变量和函数会在脚本执行完后仍然存在, 可以在当前shell中使用. 脚本中有改变环境的命令(如cd, export等), 这些改变也会影响到当前shell.

因此, 如果脚本中的变量是要持续使用的话, 就要使用 `source` 命令来加载.

### 4. 特殊参数

当然可以向脚本传递命令行参数了, 并且在脚本运行的过程中, 这些命令行参数都会被储存起来. (前面的例子里用 read 来输入参数还是笨笨的).

看下面的例子: 
```bash
#!/bin/bash
# add two numbers
total=$(($1 + $2)) # 赋值等号不要空格, 不然变量会被当做指令运行
echo "a: $1"
echo "b: $2"
echo "a+b: ${total}"
```

在脚本中, `${num}` 代表第 num 个命令行参数. num 大于 10 则必须用大括号, 否则可省略. $0 代表第 0 个变量, 即脚本的路径名. 后面就是 $1, $2, ... 依次类推了.

除此之外, 在 bash 中有很多特殊变量. 

- `$#`: 命令行参数的数量
- `$*`: 将所有命令行变量储存成一个字符串, 各个变量用空格分隔.
- `$@`: 将所有命令行变量储存成一个数组

看下面的例子: 
```bash
#!/bin/bash
echo "The script name is: $0"
echo "Total parameter number is: $#"
[ "$#" -lt 2 ] && echo "The number of parameter is less than 2.  Stop here." && exit 0
echo "The 1st parameter: ${1}"
echo "The 2nd parameter: ${2}"
echo "print \$@: $@"
echo "print \$*: $*"

count=1
for param in "$@"
do
    echo "\$@ Parameter #${count} = ${param}"
    count=$((${count}+1))
done

count=1
for param in "$*"
do
    echo "\$* Parameter #${count} = ${param}"
    count=$((${count}+1))
done
```

运行上面的程序 `bash param.sh 1x a2 sss`

输出:
```
The script name is: param.sh
Total parameter number is: 3
The 1st parameter: 1x
The 2nd parameter: a2
print $@: 1x a2 sss
print $*: 1x a2 sss
$@ Parameter #1 = 1x
$@ Parameter #2 = a2
$@ Parameter #3 = sss
$* Parameter #1 = 1x a2 sss
```

有几个注意点:

1. `$var` 和 `${var}` 都表示获取变量 `var` 的值. 有时需要用 `${var}` 来明确变量的边界. `${var}` 形式还支持变量扩展, 如前面用到的 `${var:-default}`, 以及字符串替换 `${var/old/new}`
2. `$(command)` 语法用于命令替换, 执行括号中的命令, 并将输出替换到原位置, 如 `var="Hello world"; length=$(echo "$var" | wc -c);echo $length` 将输出变量 `var` 的长度
3. `$((expression))` 用于算术运算(只支持整数), 如 `a=5;b=10;sum=$(($a+$b));echo $sum` 将输出 `a+b` 的值. `$[expression]` 是它的过时形式.
4. `${var}` 和 `"${var}"` 的区别: 大多数情况等价, 但是如果 `${var}` 包含空格, 那 `${var}` 会自动将这个参数分成多个参数, 而 `"${var}"` 则保留原来的整个参数. 不妨将上面的 for 循环中的 `$@` 换成 `$*`, 会发现得到一样的结果.

TODO: 结构化语句
TODO: 函数
TODO: 调试