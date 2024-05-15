[shell & bash 区别](https://www.cnblogs.com/conefirst/articles/15232323.html)

sh（Bourne Shell）是一个早期的重要shell，1978年由史蒂夫·伯恩编写，并同Version 7 Unix一起发布。

bash（Bourne-Again Shell）是一个为GNU计划编写的Unix shell。1987年由布莱恩·福克斯创造。主要目标是与POSIX标准保持一致，同时兼顾对sh的兼容，是各种Linux发行版标准配置的Shell，在Linux系统上/bin/sh往往是指向/bin/bash的符号链接。

dash （Debian Almquist shell）一种 Unix shell。它比 Bash 小，只需要较少的磁盘空间，但是它的对话性功能也较少。它由 NetBSD版本的Almquist shell (ash)发展而来，于1997年由赫伯特·许（Herbert Xu）移植到Linux上，于2002年改名为 dash。





docker 下载centos环境

```shell
# docker拉取centos:7镜像
docker pull centos:7

# 运行镜像
docker run -d -it -p 2222:22 -p 8888:80 --name centos centos:7  /bin/bash

# 进入镜像
docker exec -it centos /bin/bash

# 检查是否安装vim
rpm -qa|grep vim

[root@localhost usr]# rpm -qa|grep vim
vim-minimal-7.4.629-6.el7.x86_64
vim-filesystem-7.4.629-6.el7.x86_64
vim-enhanced-7.4.629-6.el7.x86_64
vim-common-7.4.629-6.el7.x86_64
vim-X11-7.4.629-6.el7.x86_64

# 没有则安装安装vim命令
yum -y install vim* 

# 编码格式修改，解决中文乱码问题
# 查看语言环境
locale -a |grep CN
# 如果没有中文语言环境 下载
yum install kde-l10n-Chinese
yum groupinstall "fonts" -y
# 修改环境变量
vim /etc/locale.conf
# 修改 en_US 为 GB2312
soure /etc/locale.conf
```

> 编写shell步骤

```shell
# 创建shell文件
vim test.sh
# 编写完shell命令后
chmod +x ./test.sh
# 执行shell脚本
./test.sh
```

注意，一定要写成 **./test.sh**，而不是 **test.sh**，运行其它二进制的程序也一样，直接写 test.sh，linux 系统会去 PATH 里寻找有没有叫 test.sh 的，而只有 /bin, /sbin, /usr/bin，/usr/sbin 等在 PATH 里，你的当前目录通常不在 PATH 里，所以写成 test.sh 是会找不到命令的，要用 ./test.sh 告诉系统说，就在当前目录找。

## shell变量

定义变量，变量名不加美元符号

```shell
your_name="zhangsan"
```

- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
- 中间不能有空格，可以使用下划线 **_**。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用help命令查看保留关键字）。

使用变量

使用一个定义过得变量，只要在变量前加美元符号即可

```shell
echo $your_name
echo ${your_name}
```

变量名外面的花括号是可选的(推荐给所有变量加上花括号)，加不加都行，加花括号是为了帮助解释器识别变量的边界

```shell
your_name="zhangsan"
# 只读变量
readonly your_name
# 删除变量
unset your_name
```

### shell字符串

字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。

单引号限制

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单独一个的单引号（对单引号使用转义符后也不行），但可成对出现，作为字符串拼接使用。

双引号限制

- 双引号里可以有变量
- 双引号里可以出现转义字符

举例

```shell
your_name="runoob"
# 使用双引号拼接
# 转义
greeting="hello, \"$your_name\" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1

# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3

# 输出
hello, "runoob" ! hello, runoob !
hello, runoob ! hello, ${your_name} !
```

获取字符串长度

```shell
string="abc"
echo ${#string}
# 等价于
echo ${#string[0]}

# 提取子字符串
string="runoob is a great site"
echo ${string:0:6}

# 查找子字符串
string="runoob is a great site"
echo `expr index "${string}" io`
```

### shell注释

```shell
# 单行注释

:<<EOF
多行注释
多行注释
EOF

:<<’
多行注释
多行注释
‘
```



## shell参数传递

```shell
#!/bin/bash
echo "Shell parameter"
echo "file_name:$0";
echo "first parameter:$1";
echo "second parameter:$2";
echo "thired parameter:$3";

# 运行结果
[root@0c3b412b7218 paramter]# ./test01.sh 1 2 3
Shell parameter
file_name:./test01.sh
first parameter:1
second parameter:2
thired parameter:3
```

特殊字符处理参数

| 参数处理 | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| $#       | 传递到脚本的参数个数                                         |
| $*       | 以一个单字符串显示所有向脚本传递的参数。 如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 |
| $$       | 脚本运行的当前进程ID号                                       |
| $!       | 后台运行的最后一个进程的ID号                                 |
| $@       | 与$*相同，但是使用时加引号，并在引号中返回每个参数。 如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 |
| $-       | 显示Shell使用的当前选项，与[set命令](https://www.runoob.com/linux/linux-comm-set.html)功能相同。 |
| $?       | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

```shell
#!/bin/bash
echo "first parameter:$1";
echo "length of parameters:$#";
echo "parameters as string:$*";
echo "parameters as string:$@"
echo "parameters as string:"$@""
echo "process ID:$$";
echo "The last process runs in the background:$!";

# 运行结果
[root@0c3b412b7218 paramter]# ./test02.sh 1 2 3 4
first parameter:1
length of parameters:4
parameters as string:1 2 3 4
parameters as string:1 2 3 4
parameters as string:1 2 3 4
process ID:807
The last process runs in the background:
```

$* 与 $@区别：

- 相同点：都是引用所有参数
- 不同点：不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。

```shell
# 举例
echo "----\$* Display----"
for i in $*; do
  echo $i
done
echo "----\$@ Display----"
for i in $@; do
  echo $i
done

# 输出结果
[root@0c3b412b7218 paramter]# ./test03.sh 1 2 3 4
----$* Display----
1
2
3
4
----$@ Display----
1
2
3
4

# 带双引号
#!/bin/bash
echo "----\$* Display----"
for i in "$*"; do
  echo $i
done
echo "----\$@ Display----"
for i in "$@"; do
  echo $i
done

# 输出结果
[root@0c3b412b7218 paramter]# ./test03.sh 1 2 3 4
----$* Display----
1 2 3 4
----$@ Display----
1
2
3
4
```

## shell数组

### 读取数组

bash支持一维数组，不支持多维数组，并且没有限定数组的大小。

```shell
# 声明数组
array_name=(val0 val1 val2 val3 val4)

# 或者
array_name=(
val0
val1
val2
val3
val4
)

# 单独定义 对下标没有限制
array_name[0]=val0
array_name[n]=valn

# 读取数组中某个值
$array_name[index]

# 获取数组长度
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}

# 输出所有元素
echo ${array_name[@]}

# 循环数组
for v in ${array_name[@]}; do
  echo "${v}"
done
```

### 关联数组

Bash 支持关联数组，可以使用任意的字符串、或者整数作为下标来访问数组元素。

关联数组使用 `declare` 命令来声明，语法格式如下：

```shell
declare -A array_name

# -A 选项就是用于声明一个关联数组
# 关联数组的键是唯一的

# 举例
declare -A site=(["google"]="www.google.com" ["baidu"]="www.baidu.com" ["jingdong"]="www.jd.com")
# or
declare -A site
site["google"]="www.google.com"
site["baidu"]="www.baidu.com" 
site["jingdong"]="www.jd.com"

# 输出关联数组内容
echo ${site["google"]}

# 使用@或者*获取所有元素
echo ${site[@]}
echo ${site[*]}

# 获取所有键值
echo ${!site[@]}

# 获取数组长度
echo ${#site[@]}
echo ${#site[*]}
```

## shell运算符

shell和其他编程语言一样，支持多种运算符，包括：

- 算数运算符
- 关系运算符
- 布尔运算符
- 字符串运算符
- 文件测试运算符

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

expr 是一款表达式计算工具，使用它能完成表达式的求值操作。

```shell
# 举例
val=`expr 2 + 2`
echo "two num sum:$val"
```

- 表达式和运算符之间要有空格
- 表达式要用``包含

### 算术运算符

假设a=10 b=20

| 运算符 | 说明                                          | 举例                          |
| :----- | :-------------------------------------------- | :---------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 30。    |
| -      | 减法                                          | `expr $a - $b` 结果为 -10。   |
| *      | 乘法                                          | `expr $a \* $b` 结果为  200。 |
| /      | 除法                                          | `expr $b / $a` 结果为 2。     |
| %      | 取余                                          | `expr $b % $a` 结果为 0。     |
| =      | 赋值                                          | a=$b 把变量 b 的值赋给 a。    |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | [ $a == $b ] 返回 false。     |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | [ $a != $b ] 返回 true。      |

**注意：**条件表达式要放在方括号之间，并且要有空格，例如: **[$a==$b]** 是错误的，必须写成 **[ $a == $b ]**。

```shell
val=`expr $a + $b`
echo "a + b = $val"

val=`expr $a - $b`
echo "a - b = $val"

# 注意乘法需要加\转义符
val=`expr $a \* $b`
echo "a * b = $val"

val=`expr $a / $b`
echo "a / b = $val"

val=`expr $a % $b`
echo "a % b = $val"

if [ $a == $b ]
then
  echo "a == b"
fi
if [ $a != $b ]
then
  echo "a != b"
fi
```

### 关系运算法

假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                                  | 举例                       |
| :----- | :---------------------------------------------------- | :------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true。               | [ $a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |



```shell
if [ $a -gt $b]
then
  echo "a > b"
else
  echo "a < b"
fi

if [ $a -le $b ]
then
  echo "a <= b"
else
  echo "a >= b"
fi

if [ $a -eq $b ]
then
  echo "a == b"
else
  echo "a != b"
fi
```

### 布尔运算法

假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                                | 举例                                     |
| :----- | :-------------------------------------------------- | :--------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [ $a -lt 20 -o $b -gt 100 ] 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |



```shell
a=$1
b=$2

echo "a = $a; b = $b"

if [ $a -lt 100 -a $b -ge 200 ]
then
  echo "a < 100 and b >= 20"
fi

if [ $a -le 100 -o $b -ge 200 ]
then
  echo "a <= 100 or b >= 200"
fi

if [ ! $a -le 100 ]
then
  echo "not a <= 100"
```

### 逻辑运算符

假定变量 a 为 10，变量 b 为 20:

| 运算符 | 说明       | 举例                                       |
| :----- | :--------- | :----------------------------------------- |
| &&     | 逻辑的 AND | [[ $a -lt 100 && $b -gt 100 ]] 返回 false  |
| \|\|   | 逻辑的 OR  | [[ $a -lt 100 \|\| $b -gt 100 ]] 返回 true |

> 逻辑与逻辑或与短路与短路或的区别

如果第一个表达式为true，会发生短路或；如果第一个表达式为false。会发生短路与

```shell
a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "返回 true"
else
   echo "返回 false"
fi
```

### 字符串运算符

假定变量 a 为 "abc"，变量 b 为 "efg"：

| 运算符 | 说明                                         | 举例                     |
| :----- | :------------------------------------------- | :----------------------- |
| =      | 检测两个字符串是否相等，相等返回 true。      | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否不相等，不相等返回 true。  | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。        | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否不为 0，不为 0 返回 true。 | [ -n "$a" ] 返回 true。  |
| $      | 检测字符串是否不为空，不为空返回 true。      | [ $a ] 返回 true。       |



```shell
a=$1
b=$2

echo "a = $a; b=$b"

if [ $a = $b ]
then
  echo "a = b"
fi

if [ $a != $b ]
then
  echo "a != b"
fi

if [ -z $a ]
then
  echo "the length of a is 0"
fi

if [ -n $a ]
then
  echo "the length of a is not 0"
fi

if [ $a ]
then
  echo "a is not null"
fi
```

### 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。

属性检测描述如下：

| 操作符  | 说明                                                         | 举例                      |
| :------ | :----------------------------------------------------------- | :------------------------ |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

其他检查符：

- **-S**: 判断某文件是否 socket。
- **-L**: 检测文件是否存在并且是一个符号链接。

```shell
file="/learn_shell/operator/test01.sh"
if [ -r $file ]
then
   echo "文件可读"
else
   echo "文件不可读"
fi
if [ -w $file ]
then
   echo "文件可写"
else
   echo "文件不可写"
fi
if [ -x $file ]
then
   echo "文件可执行"
else
   echo "文件不可执行"
fi
if [ -f $file ]
then
   echo "文件为普通文件"
else
   echo "文件为特殊文件"
fi
if [ -d $file ]
then
   echo "文件是个目录"
else
   echo "文件不是个目录"
fi
if [ -s $file ]
then
   echo "文件不为空"
else
   echo "文件为空"
fi
if [ -e $file ]
then
   echo "文件存在"
else
   echo "文件不存在"
fi
```

## echo命令

```shell
# 显示普通字符串
echo "string"
# or
echo string

# 显示转义字符串
echo "\"string\""

# 显示变量
# read 从标准输入中读取一行，并把输入行的每个字段的值指定给shell变量
read name
echo "$name test"

# 显示换行
echo -e "OK! \n" # -e开启转义
echo "test"

# 显示不换行
echo -e "OK! \c" # \c 不换行
echo "test"

# 显示结果定向至文件
echo "this is a test" > myfile

# 原样输出字符串，不进行转义或者取变量（使用单引号）
echo '$name\"'

# 显示命令执行结果
echo `date`
```

## printf命令

printf 命令模仿 C 程序库（library）里的 printf() 程序。

printf 由 POSIX 标准所定义，因此使用 printf 的脚本比使用 echo 移植性好。

printf 使用引用文本或空格分隔的参数，外面可以在 **printf** 中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等。默认的 printf 不会像 **echo** 自动添加换行符，我们可以手动添加 **\n**。

参数说明：

- format-string：为格式控制字符串
- arguments：为参数列表

```shell
echo "Hello, Shell"

printf "Hello, Shell\n"

printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876
```

**%s %c %d %f** 都是格式替代符，**％s** 输出一个字符串，**％d** 整型输出，**％c** 输出一个字符，**％f** 输出实数，以小数形式输出。

**%-10s** 指一个宽度为 10 个字符（**-** 表示左对齐，没有则表示右对齐），任何字符都会被显示在 10 个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。

**%-4.2f** 指格式化为小数，其中 **.2** 指保留2位小数。

### 转义序列

| 序列  | 说明                                                         |
| :---- | :----------------------------------------------------------- |
| \a    | 警告字符，通常为ASCII的BEL字符                               |
| \b    | 后退                                                         |
| \c    | 抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效），而且，任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略 |
| \f    | 换页（formfeed）                                             |
| \n    | 换行                                                         |
| \r    | 回车（Carriage return）                                      |
| \t    | 水平制表符                                                   |
| \v    | 垂直制表符                                                   |
| \\    | 一个字面上的反斜杠字符                                       |
| \ddd  | 表示1到3位数八进制值的字符。仅在格式字符串中有效             |
| \0ddd | 表示1到3位的八进制值字符                                     |

## test命令

shell中test命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试

### 数值测试

| 参数 | 说明           |
| :--- | :------------- |
| -eq  | 等于则为真     |
| -ne  | 不等于则为真   |
| -gt  | 大于则为真     |
| -ge  | 大于等于则为真 |
| -lt  | 小于则为真     |
| -le  | 小于等于则为真 |

```shell
#!/bin/bash
num1=100
num2=200
if test $num1 -eq $num2
then
  echo "$num1 == $num2"
else
  echo "$num1 != $num2"
fi
```

### 字符串测试

| 参数      | 说明                     |
| :-------- | :----------------------- |
| =         | 等于则为真               |
| !=        | 不相等则为真             |
| -z 字符串 | 字符串的长度为零则为真   |
| -n 字符串 | 字符串的长度不为零则为真 |

```shell
str1=$1
str2=$2

if [ $str1 = $str2 ]
then
  echo "$str1 == $str2"
else
  echo "$str1 != $str2"
fi
```

### 文件测试

| 参数      | 说明                                 |
| :-------- | :----------------------------------- |
| -e 文件名 | 如果文件存在则为真                   |
| -r 文件名 | 如果文件存在且可读则为真             |
| -w 文件名 | 如果文件存在且可写则为真             |
| -x 文件名 | 如果文件存在且可执行则为真           |
| -s 文件名 | 如果文件存在且至少有一个字符则为真   |
| -d 文件名 | 如果文件存在且为目录则为真           |
| -f 文件名 | 如果文件存在且为普通文件则为真       |
| -c 文件名 | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名 | 如果文件存在且为块特殊文件则为真     |

```shell
if test -x ./test02.sh
then
  echo 'file executable'
else
  echo 'file unenforceable'
fi
```

## shell流程控制函数

### if else

```shell
#!/bin/bash
a=$1
b=$2

printf "a=%d, b=%d\n" $a $b

if [ $a == $b ]
then
  echo "$a == $b"
elif [ $a -gt $b ]
then
  echo "$a > $b"
else
  echo "$a < $b"
fi



a=10
b=20
if (( $a == $b ))
then
   echo "a 等于 b"
elif (( $a > $b ))
then
   echo "a 大于 b"
elif (( $a < $b ))
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```

### for 循环











```bash
ps aux | grep main.py | awk '{print $2}' | xargs kill
```

1. `ps aux`：这个命令用于列出当前正在运行的所有进程。`ps`是进程状态的缩写，`aux`选项用于显示所有进程的详细信息。
2. `grep main.py`：通过管道将`ps aux`的输出传递给`grep`命令。`grep`命令用于在文本中搜索指定的模式。在这里，我们使用它来过滤出包含"main.py"的进程。
3. `awk '{print $2}'`：通过管道将`grep`命令的输出传递给`awk`命令。`awk`是一个强大的文本处理工具，可以按列或字段对文本进行处理。在这里，我们使用`awk`命令来提取每行的第二列，即进程ID。
4. `xargs kill`：通过管道将`awk`命令的输出传递给`xargs`命令。`xargs`命令用于将输入作为参数传递给其他命令。在这里，我们将进程ID作为参数传递给`kill`命令，以终止这些进程。
