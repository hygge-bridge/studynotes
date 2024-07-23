# 3 GCC编译器

## 3.1 编译过程

1、预处理，生成.i文件

```
g++ -E test.cpp -o test.i
```

2、编译，生成.s文件

```
g++ -S test.i -o test.s
```

3、汇编，生成.o文件

```
g++ -c test.s -o test.o
```

4、链接，生成可执行文件

```
g++ test.o -o test
```

## 3.2 g++重要编译参数

1、编译产生带调试信息的可执行文件

```
g++ -g test.cpp -o test //此时test包含调试信息
```

2、优化源代码：省去代码中未使用的变量，把常量表达式用结果值替换等

```
-O[n]
g++ -O2 test.cpp //用O2优化
n表示的数字含义：

- 0：不做优化
- 1：默认优化
- 2：完成O1优化外还会进行额外调整比如指令调整
- 3：循环展开和处理特性相关的优化
- 不带数字：减少代码长度和执行时间，效果等价于-O1
```

3、指定库文件 | 指定库文件路径

```
在/lib和/usr/lib和/usr/local/lib的库直接用-l链接（小写l）
g++ -lglog test.cpp //链接glog库（glog在上面三个目录之一里面）

如果库没在上面三个目录中，需要-L指定库文件所在目录(大写L)
g++ -L/home/hygge/mytestlibdir -lmytest test.cpp
```

4、指定头文件搜索目录

```
在/usr/include是不需要指定的，gcc默认会去这里找
如果不在上面目录中，就要-I指定，相对路径、绝对路径都可以
g++ -Imyinclude test.cpp
```

5、打印警告信息

```
g++ -Wall test.cpp
```

6、关闭警告信息

```
g++ -w test.cpp
```

7、设置编译标准

```
g++ -std=c++11 test.cpp
```

8、指定输出文件

```
g++ test.cpp -o test
```

9、定义宏：在gcc编译时定义宏，默认定义的字符串为1

```
g++ -DDEBUG test.cpp //定义了DEBUG宏
```

10、查看gcc使用手册

```
man gcc
```

## 3.3 g++命令行实战

1、查看目录结构

```
tree . //查看当前目录的目录结构
```

2、头文件和源文件不在相同目录的编译

```
假如当前目录下有include目录，main.cpp，src目录
include目录包含swap.h头文件
src目录包含swap.cpp源文件
main.cpp包含了swap.h

g++ main.cpp /src/swap.cpp -Iinclude
```

3、生成静态库并生成可执行文件：程序编译器时直接把静态库编译进去了

```
生成静态库必须先生成.o二进制文件，静态库本质是个归档，通过ar命令把二进制文件归档为.a的静态库文件

生成swap.o二进制文件
g++ swap.cpp -c -I../include 

把.o归档为静态库文件，静态库文件在当前目录，注意文件名必须lib开头.a结束
ar rs libswap.a swap.o 

cd.. //回到main所在目录

链接库时不需要写libswap.a,把中间的名字写了就行
g++ main.cpp -lswap -Lsrc -Iinclude -o static_main
```

4、生成动态库并生成可执行文件：程序运行的时候才会被加载

```
cd src //回到src目录（因为库文件其实就是我的源文件编译成的.o的归档）

g++ swap.cpp -I../include -fPIC --shared -o libswap.so //fPIC代表与路径无关，shared代表生产动态库文件
上一条命令可以分解成：
g++ swap.cpp -I../include -c -fPIC
g++ --shared -o libswap.so swap.o

cd .. //回到main所在目录

g++ main.cpp -Iinclude -lswap -Lsrc -o dynamic_main
```

5、运行动态库的程序

```
如果直接运行，会报错，显示找不到动态库。因为动态库不在默认搜索的三个目录里面
./dynamic_main 

必须指定加载库路径才可以
LD_LIBRARY_PATH=src ./dynamic_main
```

# 6 CMake

## 6.1 CMake跨平台介绍

CMake支持跨平台，用简单语句编译

底层逻辑：

- 写一个CMakeLists.txt，然后使用cmake命令，生成不同平台的项目工程。
- 在linux就是构建Makefile，在windows的vs就是构建Visual Studio的项目工程。然后执行不同工程的命令，生成可执行文件

假如此时增加了一个文件，那么只需要把新添加文件添加到CMakeLists.txt文件就可以了。

## 6.2 语法特性介绍

基本语法格式：指令（参数1, 参数2。。。）

- 参数使用**括弧**括起来
- 参数之间使用**空格或者分号**分开
- **指令不区分大小写**，参数和变量必须区分大小写
- **变量使用${}**方式取值，在**IF语句中直接使用变量名**

## 6.3 重要指令和CMake常用变量

1、指定cmake最小版本要求

```
设置cmake最小版本要求2.8.3
cmake_minimum_required(VERSION 2.8.3)
```

2、定义工程名称

```
设置执行工程名称为HELLOWORLD
project(HELLOWORLD)
```

3、显式定义变量

```
定义SRC变量，值为sayhello.cpp hello.cpp
set(SRC sayhello.cpp hello.cpp) 
```

4、向工程添加多个指定的头文件搜索路径，类似g++ -I（大写i）

```
将/usr/include/myincludefolder 和 ./include 添加到头文件搜索路径
include_directories(/usr/include/myincludefolder ./include)
```

5、向工程添加多个指定库文件搜索路径，类似g++ -L

```
将/usr/lib/mylibfolder 和 ./lib 添加到库文件搜索路径
link_directories(/usr/lib/mylibfolder ./lib)
```

6、生成库文件，联想g++怎么生成的

```
通过SRC变量生成libhello.so动态库
add_library(hello SHARED ${SRC})

通过SRC变量生成libhello.so静态库
add_library(hello STATIC ${SRC})
```

7、添加编译选项

```
添加编译选项 -Wall -std=c++11 -O2
add_compile_options(-Wall -std=c++11 -O2)
```

8、生成可执行文件

```
编译main.cpp生成可执行文件main
add_executable(main main.cpp)
```

9、链接共享库，类似g++ -l（小写l）

```
把hello动态库，链接到main可执行文件中
target_link_libraries(main hello)
```

10、向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置（不太理解）

```
添加src子目录，此时必须确保src子目录中有CMakeLists.txt
add_subdirectory(src)
```

11、发现一个目录下所有源代码文件，并且列表存储在一个变量中，这个指令临时被用来自动构建源文件列表

```
定义SRC变量，值为当前目录下所有的源代码文件
aux_source_directory(. SRC)

编译SRC变量代表的所有源代码文件，生成main可执行文件
add_executable(main ${SRC})
```

11、gcc/g++编译选项

```
CMAKE_C_FLAGS
CMAKE_CXX_FLAGS

在g++编译选项后面追加-std=c++11
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
```

12、设定编译类型debug/release

```
设定编译类型为debug模式，需要调试选debug
set(CMAKE_BUILD_TYPE Debug)

设定编译类型为release模式，需要发布选release
set(CMAKE_BUILD_TYPE Release)
```

其他看课件学习

## 6.4 cmake编译工程

cmake目录结构：项目主目录存在CMakeLists.txt文件中，项目顶层目录存在这里

这个时候有两种方式设置编译规则：

- 如果包含源文件的子文件夹包含CMakeLists文件，那么主目录的CMakeLists.txt可以通过add_subdirecroty添加子目录
- 如果包含源文件的子文件夹没有包含CMakeLists文件，那么子目录的编译规则就需要体现在主目录的CMakeLists.txt中

cmake编译流程（linux下）：

1. 编写CMakeLists.txt
2. 执行cmake path生成Makefile（path就是CMakeLists.txt所在的目录)
3. 执行make进行编译

cmake工程一般会有两种构建方法：

1. 内部构建（不推荐）：内部构建会在同级目录下产生一大堆中间文件，这些中间文件是我不需要的，和源代码文件放在一起就显得很乱

   ```
   在当前目录下，编译本目录的CMakeLists.txt，生成Makefile和其他文件。此时中间文件就在当前目录下
   cmake .
   
   执行make，生成可执行文件
   make
   ```

2. 外部构建（推荐）：将编译输出文件和源代码放到不同目录

   ```
   在当前目录下，创建build文件夹(程序界公认的，其实取其他名字也可以)
   mkdir build
   
   进入build文件夹
   cd build
   
   编译上一级的CMakeLists.txt，生成Makefile和其他文件。此时中间文件都会放在build文件夹中。
   cmake ..
   
   执行make，生成可执行文件
   make
   ```


## 6.5 CMake代码实践

文件目录树：

![image-20240326131145037](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240326131145037.png)

一般CMakeLists.txt都在顶层目录，先写个简单的CMakeLists.txt

```
设置cmake最低版本要求
cmake_minimum_required(VERSION 3.0)

设置项目名称
project(HELLOWORLD)

生成可执行文件
add_executable(helloWorld_cmake helloworld.cpp)
```

采用内部构建：生成很多中间文件，看着太不舒服了

```
cmake .
make
```

所以采用外部构建：文件全部在build里面了，爽！

```
mkdir build
cd build
cmake ..
make
```

再来个复杂的项目：

![image-20240326132305533](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240326132305533.png)

写个CMakeLists.txt

```
设置cmake最低版本，以及设置项目名
cmake_minimum_required(VERSION 3.0)
project(SWAP)

指定头文件路径
include_directory(include)

生成可执行文件
add_executable(main_cmake main.cpp src/swap.cpp)
```

