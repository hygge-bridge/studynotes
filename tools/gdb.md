ps：其实很多gdb命令用法都是通用的，可以自行测试自己想要写的语法，看看gdb是否支持

帮助命令

```
help 命令
```

继续执行上一条命令

```
直接按回车就行
```

# 1-1LinuxCC++调试准备工作

# 1-2启动调试

section1只用于测试传入参数

section2

## 用法

启动调试并传入参数

```
gdb --args 可执行文件 参数

gdb 可执行文件
set args 参数

gdb 可执行文件
r 参数
```

附加到进程：适用于程序已经启动了，如果使用gdb启动那么就会运行在一个新程序上而不是在原有程序上调试

```
查看pid：ps -aux | grep 名字
gdb attach <pid>
gdb --pid <pid>
注意：以上代码大概率会报错，切换为sudo用户就可以了。一般不要用附加到进程
原因：gdb允许攻击者从用户环境的对等进程中窃取或更改私钥等操作，因此现在操作系统通常会限制gdb可以访问的内容
```

逐过程执行：有函数也会跳过

```
next(n)
```

逐语句执行：有函数会进入

```
step(s)
```

退出当前函数：回到函数的调用点

```
finish
```

退出调试：调式进程会被杀死

```
quit(q)
```

分离程序：只是把这个进程从调试中分离出去，进程任然在运行，还可以附加回去

```
detach
```

终止调试进程而非分离

```
kill
```

# 1-3调试断点管理

section3

## 用法

设置断点

```
为代码行设置断点：如果不写文件名就默认为当前文件
b 文件:行号

为函数设置断点：只要函数名相同就会被设置断点
b 函数名

通过正则表达式设置断点：把带有这个名字的函数都设置断点
rb 名字

条件断点
b 文件:行号 if 条件

临时断点：命中一次后就会被自动删除
tb 文件:行号
```

查看断点

```
查看所有断点信息
info breakpoints(i b)

只查看某一个断点信息
i b 断点编号
```

删除断点

```
删除所有断点
delete

删除某一个断点
delete 断点编号
举例：delete 1-3 删除断点1 2 3
```

开启/禁用断点

```
启动断点
enable 断点编号

禁用断点
disable 断点编号
```

# 1-4变量查看与修改

section4

## 用法

查看函数参数

```
info args
```

查看局部变量

```
info locals
```

查看变量

```
p 变量名
如果变量是字符串，会把整个字符数组的内容都显示出来，包括填充的数据\000
如果变量是结构体，会把所有成员变量都显示在一行，不会显示成员方法
如果变量是数组，也会把所有数据显示在一行
```

设置显示模式

```
设置显示字符串时，遇到结束符就不再显示
set print null-stop

设置显示结构体时，一个字段显示一行
set print pretty

设置显示数组时，按照结构体方式显示
set print array on
```

使用p+gdb内嵌函数

```
p gdb内嵌函数
举例：p sizeof(int)，都是c函数
```

修改变量值

```
p 赋值操作
举例：p i=3
```

# 1-5内存查看与修改 

section5

## 作用

```
做网络通信中，往往会出现接收到的数据不是发送的数据，这个很可能是CPU大小端的问题。就需要去查看这个数据在内存中的分布
12 34 56 78(高位--->低位) //数字是高位在前，内存是低位在前
小端：78 56 34 12 低位存储低位地址，高位存储高位地址
大端：12 34 56 78 和小端相反
```

## 用法

无法查看非法地址

```
举例：int itest = 0x12345678;
查看0x12345678的内存布局，这个内存不一定可以访问
x itest

但是可以查看itest的内存布局
x &itest
```

查看内存

```
如果要查看一个变量内存布局，需要对其解引用
x 地址

查看变量的4个字节
x /4b 地址

显示字符串,字符串本质就是一个指针，指针本身存的就是地址
x /s 地址
```

修改内存（不怎么用）

```
set {类型}取变量地址 赋值
举例：set {int}&test.gender=110（不是很会）
```

## 结构体字节对齐（补充内容）

https://blog.csdn.net/lanzhihui_10086/article/details/44353381

自然对齐：一个变量的内存地址刚好是它长度的整数倍

需要字节对齐的原因：

字节对齐的作用不仅是便于cpu快速访问，同时合理的利用字节对齐可以有效地节省存储空间。。假如没有对齐，可能cpu需要多次才能取到数据

X86 gcc默认按照4字节对齐

# 1-6寄存器查看与修改

section6：去掉-g再编译，不要调试符号

## 作用

```
调试过程中，很多时候这个程序可能没有调试符号，比如是发行版；或者是程序崩溃了，内存遭到了破坏。就很难看到程序中运行的状态，也很很难看到内存内容，gdb很多查看命令都没法用了。
```

```
有些内容是放到寄存器的，比如函数参数少于6个是通过寄存器来传递的，多于6个就会放到函数栈里面
```

| **寄存器** | **函数参数**（按照左到右的顺序） |
| ---------- | -------------------------------- |
| **rdi**    | **第一个参数**                   |
| **rsi**    | **第二个参数**                   |
| **rdx**    | **第三个参数**                   |
| **rcx**    | **第四个参数**                   |
| **r8**     | **第五个参数**                   |
| **r9**     | **第六个参数**                   |

## 用法

查看寄存器

```
查看通用寄存器
info registers

查看所有寄存器
info all-regisers

查看的那个寄存器
i r 寄存器

如果查看到的是地址，可以使用x查看，或者使用p (强制转换)地址,比如
i r rsi //假设查看到存放的是0x1234
//已经知道是字符串的前提下，可以这么查看
p (char*)0x1234 
x /s 0x1234
```

查看某一行的汇编代码地址

```
info line 行号
```

反汇编代码

```
disassemble
```

修改寄存器

```
pc/rip (program counter)寄存器,保存程序下一条要执行的指令地址，通过修改pc寄存器来改变程序执行的流程

p $rip=某一行地址
set var $rip=某一行地址
```

# 1-7源代码管理查看

view_source

## 用法

查看源代码

```
显示默认10行，当前行的前后5行
list(l)

往前显示
l -

往后显示
l

查看函数：如果有多个同名函数，都会显示出来
l 函数名

显示类中的函数
l 类:函数

指定行号/函数，文件名可以省略
l 文件名：行号/函数

设置默认显示行数
set listsize 数量
```

# 1-8源代码管理搜索

## 作用

```
代码量比较大时，一直list查看代码找我想要找的函数等就很麻烦
```

## 用法

搜索源代码

```
正则表达式，当前代码往后查找
search 名字
forward-search 名字

当前代码往前查找
reverse-search 名字
```

查看gdb查找源代码的路径

```
show directories
Source directories searched: $cdir:$cwd
默认显示两个目录---当前目录:工作目录
```

设置源文件搜索目录

```
directory 路径（比如源文件在上一级目录，直接给文件名居然也可以，逆天！）
```

# 1-9函数调用栈管理

backtrace

## 概念

```
栈帧是指函数在被调用时，所拥有的一块独立的用于存放函数所使用的状态和变量的栈空间
所有栈帧组成的信息称为调用栈
```

## 用法

查看栈回溯信息

```
backtrace(bt)
```

切换栈帧

```
frame(f) 编号
```

查看栈帧信息

```
info frame 编号
```

# 2-1观察点使用

watch-section

## 概念

```
观察点是一个特殊的断点，当表达式的值发生变化时，它将中断下来。表达式可以是一个变量的值，也可以包含由运算符组合的一个或多个变量的值，例如' a + b '。有时被称为数据断点（VC里面就称之为数据断点）
观察点类型：硬件观察点-几乎不影响性能 软件观察点-比较影响性能
```

## 用法

设置观察点

```
写观察点，变量被写就会中断
watch 变量

读观察点，变量被读就会中断
rwatch 变量

读写观察点，读写都会中断
awatch 变量

指定线程设置观察点
watch 变量 thread id
```

查看观察点

```
i b也可以查看
info watch就是只查看观察点
```

断点的相关操作大部分也使用（比如disable）

# 2-2捕获点使用

catch-section

## 作用

```
捕获点是一个特殊的断点，当捕获到event这个事件时，程序就会中断下来。比如捕获throw，那么只要运行到throw都会中断。
```

## 用法

```
catch event
```

```
event比如：
catch assert -- Catch failed Ada assertions, when raised.
catch catch -- Catch an exception, when caught.
catch exception -- Catch Ada exceptions, when raised.
catch exec -- Catch calls to exec.
catch fork -- Catch calls to fork.
catch handlers -- Catch Ada exceptions, when handled.
catch load -- Catch loads of shared libraries.
catch rethrow -- Catch an exception, when rethrown.
catch signal -- Catch signals by their names and/or numbers.
catch syscall -- Catch system calls by their names, groups and/or numbers.
catch throw -- Catch an exception, when thrown.
catch unload -- Catch unloads of shared libraries.
catch vfork -- Catch calls to vfork.
```

常见使用技巧

```
捕获点中断后，常用bt查看栈帧，然后f切换到对应的栈帧，从而查看问题所在
如果是设置的系统函数捕获点，可以先在在main设置断点并运行后，再来设置捕获点，这样子就可以避免初始化过程中调用的系统函数被中断
```

# 2-3为断点执行命令 

breaks-section：链表排序

## 作用

```
当执行到断点时，自动执行命令
```

## 用法

设置自动执行的命令

```
commands 断点编号
命令
end
ps：如果不加编号，就代表执行最后一个断点编号；如果命令有c函数，不需要加括号，比如printf"good\n"，不加\n就显示不出来结果（估计会被覆盖吧）
```

取消自动执行命令，就是里面啥也不写

```
commands 断点编号
end
```

保存断点信息到文件

```
save breakpoints 文件
ps：而且这个文件是可以修改的
```

加载文件信息到调试

```
source 文件
```

# 2-4gdb多窗口管理

viewsource

## 作用

```
提供一个tui界面(text user interface)
```

## 用法

打开源码/汇编/寄存器窗口

```
layout src/asm/reg
断点显示：窗口上显示B表示命中过，b表示还没有命中，+表示断点可用(enable)，-表示断点不可用(disable),h表示硬件断点
```

切分窗口

```
layout split
```

切换窗口焦点

```
focus(fs) cmd/src/asm/reg
```

查看当前拥有焦点的窗口

```
info win
```

退出窗口模式

```
ctrl+x+a
```

# 2-5查看对象类型

ptype-section

## 作用

```
print-查看变量的值，计算表达式的值，设置变量的值（局限性了）
查看变量是什么类，是什么结构体，大小是多少，内存布局怎么样
```

## 用法

查看类型

```
whatis 变量/函数(类中的也可以)
```

查看变量

```
查看变量的类型，如果是类/结构体会显示其成员变量和函数
ptype 变量

只显示类的成员变量
ptype /m 变量

让类显示自己的真实类型，比如T1* t2 = new T2（），真实类型为T2，但是如果不设置就会显示类型为T1
set print object on 

不显示typedef
ptype /t 变量 

显示偏移量，每个成员多大，结构体总共多大
ptype /o 变量 

显示包含有这个名字的变量定义位置，只能查看全局/静态变量
i variables 名字
```

# 2-6多线程基础

multhread

## 概念

```
进程：通常被定义为一个正在运行的程序中的实例。程序在被实例化之前，只是一个二进制集合，没有实际意义。程序执行后，os才会为这个进程分配资源。类似c++的类，只有实例化后才会真正分配资源。

线程：是系统调度的单元，os调度的最小单位，os不会调度进程，进程只是线程的容器，故线程的生命周期受制于进程，进程结束线程也就结束
```

## 用法

创建线程

```
pthread_create()
```

等待线程结束

```
pthread_join()
```

# 2-7C++跨平台多线程知识

multhread2

## 概念

```
linux原生创建线程方式只支持全局函数或静态函数
c++线程类支持全局函数、类的静态成员函数、类的普通成员函数
```

## 用法

创建线程

```
thread t()
```

等待线程结束

```
t.join()
```

# 2-8多线程调试管理

multhread3

## 用法

查看所有线程信息

```
info threads
线程信息解释：Id-线程序号，Target Id-线程地址（LWP-轻量级线程）Frame-线程所用的栈帧（名字+函数+函数参数）
```

切换线程

```
thread num
```

查看指定线程的栈帧

```
bt只针对当前线程，想查看指定线程的栈帧使用情况，需要先去切换线程再bt
```

# 2-9线程查找线程断点

multhread3

## 用法

查找线程

```
使用正则表达式
thread find 名字/lwp/地址/通配符（不可以用id找，你都知道id了还找啥。。）
```

设置线程名字

```
thread name 名字
```

为线程设置断点，线程退出断点自动被删除

```
b 行号/函数名 thread id
举例：b 28 thread 5
一定要在线程5启动后才可以设置这种断点
```

# 2-10为线程执行命令

multhread4

## 作用

```
不需要切换线程也可以让线程执行对应操作，也可以让多个线程都执行对应操作
```

## 用法

为线程执行命令

```
为单个线程执行操作
thread apply id 命令

为多个线程执行操作
thread apply 1 2 3 5 命令 OR thread apply 1-3 5 命令

为所有线程执行操作
thread apply all 命令

参数-s：就算输错了命令也不会报错 -q：多个线程会显示到一起并不会分行 数字：显示帧数（用于信息太多的情况）
thread apply -s all bt 2
```

# 2-11线程日志信息控制

multhread4

## 概念

线程日志信息包括线程打开、关闭提示等

## 用法

查看线程启动和退出时是否打印线程日志信息

```
show print thread-events
```

设置是否打印线程日志信息

```
set print thread-events on/off
```

# 2-12执行外部命令以及保存命令及输出

## 用法

执行shell命令

```
shell shell命令
! shell命令
```

使用管道

```
pipe 命令
| 命令
举例：pipe i locals | grep test 或者 | i locals | grep test
```

启动/禁用结果输出，默认结果存到gdb.txt

```
set logging enabled on/off
```

设置输出文件，将调试信息存到文件

```
set logging file 文件名
```

覆盖输出文件，默认为追加

```
set logging overwrite
```

# 3-1跳转执行-任意执行代码穿越到过去和未来

jump-section

## 作用

```
跳转执行，跳转到指定位置恢复执行。比如错过了某个很重要的部分代码，使用jump就可以避免重新调试，可以节省很多调试的时间，提高效率

jump即在指定位置恢复执行，如果存在断点，执行到指定位置时将中断下来。如果没有断点，则不会停下来，因此，我们通常会在指定位置设置一个断点。
跳转命令不会更改当前堆栈帧、堆栈指针、任何内存内容、程序计数器以外的任何寄存器，可以暂时认为只改程序计数器。
```

## 作用

跳转代码的执行位置

```
跳转到指定行
jump 行号

如果有标签，也可以跳转到标签位置，现在的代码应该几乎都没有用标签的吧。。
jump 标签

跳转到别的函数，不推荐使用，大概率会出问题
jump 函数名
可以跳转到函数调用点前面的行数，比如赋值实参的代码附近，然后运行到函数再进入函数
```

# 3-2反向执行-调试中的undo（之后的还没测试）

runback-section

## 作用

```
反向执行就是把刚才的操作撤销掉：主要针对内存的数据和寄存器的数据，因为就这个能改。如果是直接修改文件，一般是没法撤销的
```

## 用法

记录操作：每执行一步都需要记录，有记录才可以回退

```
record
```

反向执行

```
reverse-next(rn)
所有反向执行的命令都是reverse-命令
举例：reverse-finish回到调用点，假如回到调用点后再n，那么就会直接运行到上次没有执行完的函数代码处
```

和jump的区别

```
jump不会修改内存内容，就算jump回去值还是改变后的
rn是方向执行，值会变成原来的值
```

停止记录操作：此时反向执行就没用了

```
record stop
```

# 3-3调试子进程

fork-section

## 作用

```
创建子进程就是创建一个和父进程差不多的进程，大部分内存数据是一样的，pid和锁是自己的特有数据，父子进程是运行在自己不同的进程空间里面的。

创建子进程作用：创建多个进程并行处理任务，互不干扰
fork：写时拷贝

如果看到子进程的父进程pid是1，这个是init process
```

## 用法

查看当前调试进程模式

```
show follow-fork-mode
```

选择调试父/子进程：默认是parent，所以如果只是在子进程代码设置断点，是不会中断的

```
set follow-fork-mode parent/child
```

查看进程pid，调用函数都可以采取类似方法

```
call (int)getpid()
不强制转换会提示找不到返回类型，因为gdb不知道是什么类型
```

同时调试父子进程：告知gdb不要剥离未跟踪的进程

```
set detach-on-fork off
默认值为on，调试一个进程，另外一个进程就会被剥离，不知道什么时候就运行完了，那么此时另外一个进程就没法调试了
```

显示gdb子进程：gdb调试的进程或者attach的进程都是gdb的子进程

```
info inferiors
```

切换gdb子进程：切换了进程才可以调试

```
inferior 编号
```

# 3-4多进程调试

patch-section

attach的程序：

release-section

remote-section

## 作用

```
一个gdb session调用多个进程

gdb用inferior来表示一个被调试进程的状态，通常情况下，一个inferior代表一个进程。有时候一个inferior也不一定有进程和它绑定。gdb调试一个程序时默认会创建一个inferior

inferior类似一个房间，可以放一个进程，也可以没有那就是空的

所有gdb命令只针对当前inferior
```

## 用法

添加inferior

```
add-inferior
```

切换inferior

```
inferior 编号
```

多个进程同时执行：默认只有一个进程可以执行

```
set schedule-multiple on
```

删除一个referior：不可以删除当前referior

```
remove-referior 编号
```

附加一个进程到referior

```
attach pid
```

剥离referior中的进程

```
直接在当前referior里detach 或者 detach referior 编号
```

# 3-5调试时调用内部外部函数

call-section

## 作用

```
在调试过程中直接调用函数（自定义函数，c函数等），也可以起到单元测试的作用，程序中没有调用的函数，可以直接在调试中调用。

没有调试符号一样可以调用，不过需要强制类型转换才可以调用成功
```

## 用法

调用函数/表达式

```
p 函数/表达式
求表达式的值并显示结果值。表达式可以包括对正在调试的程序中的函数的调用，即使函数返回值是void，也会显示。

call 函数/表达式
求表达式的值并显示结果值，如果是函数调用，返回值是void的话，不显示void返回值。
```

# 3-6调试时跳过指定函数

skip-section

## 作用

```
跳过某些函数，这里指的是使用s的时候不进入某些函数中，函数任然会正常执行
```

## 用法

跳过函数名

```
skip 函数名
效果等同于s进入函数，然后finish或者return，但是这种做法太麻烦了
```

跳过文件中的所有函数：需要跳过的函数多又大多在同一文件时使用

```
skip file 文件名
```

通配符方式

```
skip -gfi 通配符形式
举例：跳过当前目录下common的所有文件 skip -gfi common/*.*
```

# 3-7制作调试发行版

release-section

## 作用

```
release版或者发布给客户的程序，一般都没有调试信息。如果客户那里出了问题或者程序崩溃了，即使生成了dump文件，我们没有调试信息，也不方便分析问题。

本章学习如何分析不带调试信息的程序以及生成发行版程序
```

## 用法

制作发行版（release）

```
取消-g参数make一个release版本，然后再添加-g参数make一个debug版本

make一个debug版本，利用strip去掉调试信息并输出到另外一个文件
strip -g debug文件 -o release文件
```

调试发行版：调试无调试符号的文件时指定一个符号文件

```
gdb --symbol=release文件 -exec=debug文件

利用objcopy提取调试信息到一个文件
objcopy --only-keep-debug debug文件 纯调试信息文件（一般叫*.sym）
gdb --symbol=纯调试信息文件 -exec=debug文件
```

生成dump文件

```
generate-core-file 文件名(*.core)
```

调试release版dump文件

```
gdb debug文件 *.core
举例：此时具有一个rel.core的dump文件，它的debug文件叫dubug-section那么调试代码如下gdb debug-section rel.core（可以用。sym文件嘛）
```

# 3-8软件补丁制作-直接编辑二进制程序

patch-section

## 作用

```
利用gdb直接修改可执行文件。给软件打补丁，也是可以直接修改可执行文件然后保存。

可执行文件存放的二进制文件，所以修改时需要直接修改二进制代码
```

## 用法

设置可写方式打开二进制文件：默认gdb是只读方式打开文件的

```
gdb --write 可执行文件
```

显示反汇编代码：同时显示机器码和源码

```
disassemble /mr 函数名
举例：修改其中一个字节p {unsigned char}地址=值，set也可以
unsigned char只占用了一个字节
```

查看有哪些函数：没有调式信息的情况下，使用这个命令查看函数

```
info functions
```

# 4-1内存泄漏检测

memleak-section

## 作用

```
valgrind会出现内存泄漏误报的问题，信息太复杂。如果只是关注一些具体函数是否内存泄露，那使用gdb就够了。这种用法只是可以判断有没有内存泄漏，不方便检查内存泄漏在那个位置。
```

## 用法

查看内存泄漏，调用函数来确认是否内存泄漏

```
call malloc_stats()
通过调用此函数，在当前函数执行前后比较系统内存的差值，有差值就代表有泄露，主要看in use bytes

call malloc_info(0,stdout)
第一个参数始终为0，第二个参数是个输出文件
显示的是xml文件格式，主要看<total type="rest" count="" size=""/>
```

# 4-2检测各种内存问题检查泄漏栈溢出野指针等

memcheck-section

## 作用

```
gdb检查内存泄漏功能太有限了，现在用gcc本身的特性来检查内存问题，比如内存泄漏、堆栈溢出。用gcc的优点：不需要修改源程序，只需要添加编译选项或者link选项就可以，也不用调试，也不会影响性能，而且自动汇报程序的内存相关问题

gcc选项 -fsanitize=address
检查内存泄漏：出现内存泄漏马上汇报是哪一行出现的问题
检查堆溢出：一般堆溢出不是直接crash，而是在某些特定情况下崩溃，使用选项后一旦溢出就会崩溃且汇报时哪一行出现问题
检查栈溢出：同上
检查全局内存溢出
检查释放后再使用：野指针问题
ps：不用选项这些错误几乎都能编译通过
```

## 用法

![image-20240310213202270](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240310213202270.png)

# 4-3远程调试

remote-section

## 作用

```
本地程序运行起来很正常，但是放到测试环境就出问题了，主要是环境不同了。使用远程调试，源代码还是在本地，对远端的测试环境程序进行源码级别的调试

远程调试和本地基本上一样，只是不支持r命令，用c替换就行
```

## 用法

服务器端/被调试机

```
安装gdbserver
apt install gdbserver

启动gdbserver
gdbserver 本机ip地址:端口（端口可以自己取） 可执行文件

如果服务端程序已经启动了，那就使用附加形式来调试
gdbserver 本机ip地址:端口（端口可以自己取） --attach pid

建议关闭防火墙，或者把gdbserver的端口添加到防火墙
```

客户端/调试机

```
gdb远程连接并进行调试
target remote 服务器端ip地址:端口

中断程序：不想调试了就这样子操作，服务端也会结束吗？？？
ctrl+c

退出gdb
quit：远程调试中不会导致进程结束，本地调试会
detach：将进程脱离出来
```

# 4-4多线程死锁调试

deadlock-section

## 用法

研究死锁方法

```
逐个查看线程的堆栈情况
info threads
bt
切换到本线程的和自己相关的函数堆栈中去，看看停在那里
frame

查看锁的使用情况，一般c++的锁里面会有相关字段的
p
```

# 4-5核心转储coredump

coredump1-section

## 用法

为正在运行的进程生成coredump文件

```
采用attach的方式附加进gdb
gcore 文件名
此时就算detach了，也完全不影响进程的执行
```

让程序崩溃后自动产生core dump文件

（不同系统稍微有点不同）

```
如果发现值为0的话，表示不会自动生成core dump文件
ulimit -c


设置值为无限，此时会进程崩溃会自动生成名为core的core dump文件
ulimit -c unlimited

修改默认coredump文件的名字
默认是在/proc/sys/kernel/core_pattern路径下
注意：系统文件vi是改不了的，使用echo
echo -e "%e-%p-%t" > /proc/sys/kernel/core_pattern(进程名字-pid-时间戳)
```

```
当ulimit -c为0，但是/proc/sys/kernel/core_pattern不一样也可以生成coredump文件
每次生成后都会把前面的覆盖
```

![](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240314151458332.png)

# 4-6栈溢出核心转储coredump分析

coredump2-section

## 作用

```
通过gdb定位程序崩溃的问题

linux C有个alloca函数，可以动态分配内存，不用自己释放内存。alloc性能也比malloc好一点
```

## 用法

gdb调试coredump文件

```
gdb 可执行文件的当前路径 coredump文件
可执行文件才有调试信息，目的是为了和coredump文件匹配起来
```

常见分析步骤

```
先bt查看函数堆栈
f切换到指定栈帧，一般是根据main一步一步往下看函数
看到分配内存函数，可以想想是不是分配内存失败，计算一下。
!free -h可以查看内存使用情况

因为只有alloca是分配内存的函数，直接看看这个函数
!man alloca
惊奇发现这个alloca是在栈上分配的内存

! ulimit -a
查看系统中栈的大小等系统信息

最后总结分配内存分配多了，是栈溢出问题。
```

# 4-7无调试符号coredump分析

coredump3-section

## 作用

```
没有调试信息，很多命令都用不了，比如查看参数等
```

## 用法

查看堆栈

```
bt
where
```

查看代码

```
gdb里面没法看了
使用shell命令查看
比如：!head main.cpp -n 15
```

此时发现只是调用了memcpy，没有调用memmove

```
!man memcpy 可以看note
memcpy无法处理内存有重叠的字符串，比如：
const char* str = "12345";
memcpy(src+2, src, 3)
memmove可以处理上诉问题，所以memcpy后面就是memmove的别名了
```

看反汇编

```
disassemble
发现是给某个值赋值的时候崩溃了
```

可以查看一下寄存器，看看是不是函数参数出了问题

```
自己回忆函数参数是通过那些寄存器传递的
i r edi
x /s $rdi 发现第一个参数没问题

i r rdx 发现第三个参数也没问题

i r rsi 发现第二个参数有点问题
```

