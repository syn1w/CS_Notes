<!-- TOC -->

- [一、基础命令](#一基础命令) 
- [二、静态库和动态库](#二静态库和动态库)
	- [1 创建库](#1-创建库) 
	- [2 使用库](#2-使用库)
- [三、变量和环境变量](#三变量和环境变量)
	- [1. 常用变量](#1-常用变量)
	- [2. 环境变量](#2-环境变量)
	- [3. 系统信息](#3-系统信息)
	-  [4. 开关选项](#4-开关选项)
- [四、常用指令](#四常用指令)   
	- [1. 基本指令](#1-基本指令)
	- [2. install](#2-install)
	- [3. find_xxx](#3-find_xxx)
	- [4. 控制指令](#4-控制指令)
	- [5. cmd](#5-cmd)
- [五、例子](#五例子)
<!-- /TOC -->

# 一、基础命令

```cmake
PROJECT(projectname [CXX] [C]) # 指定工程名称，可选的语言列表
# 而且隐式包含了两个变量 projectname_BINARY_DIR 和 projectname_SOURCE_DIR
# 还有这两个变量的别名变量 PROJECT_BINARY_DIR 和 PROJECT_SOURCE_DIR

# example
PROJECT(Hello CXX)
```

```cmake
SET(varname [value] [CACHE TYPE DOCSTRING [FORCE]]) # TODO: 可选参数介绍

# example 1
SET(main_SRCS main.cpp test.cpp)

# example 2
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin) # EXECUTABLE_OUTPUT_PATH 为原有变量
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
```

```cmake
MESSAGE([SEND_ERROR | STATUS | FATAL_ERROR] "message to display")
```

```cmake
ADD_EXECUTABLE(output srcs)        # output 为输出的可执行文件名，srcs 是源文件
                                   # 如果源文件名包含空格，必须使用引号 ""

# example
ADD_EXECUTABLE(hello ${main_SRCS}) # ${main_SRCS} 引用变量名
```

```cmake
# 添加存放源文件的子目录，并将生成的中间二进制和目标文件放到 `binarydir`
# EXECLUDE_FROM_ALL 参数是这个目录从编译过程排除
ADD_SUBDIRECTORY(sourcedir [binarydir] [EXECLUDE_FROM_ALL])
```



```cmake
# 目标文件安装
INSTALL(TARGETS targets 
	[[ARCHIVE|LIBRARY|RUNTIME]
		[DESTINATION <dir>]
		[PERMISSIONS] permissions)
		[CONFIGURATIONS [Debug|Release|...]]
		[COMPONENT <component>]
		[OPTIONAL]] [...])
# targets 为目标名
# ARCHIVE 指 static library, LIBRARY 指 dynamic library, RUMTIME 指可执行二进制文件
# DESTINATION 定义了安装路径，如果以 '/' 开头，指的是绝对路径，CMAKE_INSTALL_PREFIX 失效
# 默认 CMAKE_INSTALL_PREFIX 为 /usr/local 

# 普通文件安装
INSTALL(FILES files... DESTINATION <dir>
	[PERMISSIONS permissions...]
	[CONFIGURATIONS [Debug|Release|...]]
	[COMPONENT <component>]
	[RENAME <name>] [OPTIONAL]))
# 默认权限是 644 

# 非目标文件的可执行程序指向，比如脚本
INSTALL(PROGRAMS files... DESTINATION <dir>
	[PERMISSIONS permissions...]
	[CONFIGURATIONS [Debug|Release|...]]
	[COMPONENT <component>]
	[RENAME <name>] [OPTIONAL])
# 默认权限是 755


# 目录的安装
INSTALL(DIRECTORY dirs... DESTINATION <dir>
	[FILE_PERMISSIONS permissions...]
	[DIRECTORY_PERMISSIONS permissions...]
	[USE_SOURCE_PERMISSIONS]
	[CONFIGURATIONS [Debug|Release|...]]
	[COMPONENT <component>]
	[[PATTERN <pattern> | REGEX <regex>]
	[EXCLUDE] [PERMISSIONS permissions...]] [...])
# dirs 是相对路径 
# abc 和 abc/ 有很大区别，abc 是将 abc 目录放入到目标位置，abc/ 是将目录中的内容安装到目标位置
# PATTERN 用正则表达式进行过滤
```

```sh
cmake -DCMAKE_INSTALL_PREFIX=/usr # 设置 CMAKE_INSTALL_PREFIX 宏
```

```sh
# cmake 之后进行实际安装
make install
```



# 二、静态库和动态库

## 1 创建库

```cmake
ADD_LIBRARY(libname [SHARED|STATIC|MODULE]  # 默认为 STATIC
	[EXCLUDE_FROM_ALL] 
	src1 src2 src2 ... srcn)
	
# 默认的生成目录和源文件的目录相同，也可以修改内置变量来修改库文件的路径
SET(LIBRARY_OUTPUT_PATH <dir>) 

# example
ADD_LIBRARY(hello SHARED hello.cpp)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib) 
# 在 xxx/build/lib 下生成 libhello.so
# 如果是 STATIC 则生成 libhello.a
```

如果想要生成相同名称的静态库和动态库，比如 `libhello.so` 和 `libhello.a`，不能直接使用两个 `ADD_LIBRARY`。需要用到另外一个指令。

```cmake
SET_TARGET_PROPERTIES(target1 target2 ...
	PROPERTIES prop1 value1
	prop2 value2 ...)
	
# example 
ADD_LIBRARY(hello_static STATIC ${srcs})
SET_TARGET_PROPERTIES(hello_static PROPERTIES OUTPUT_NAME "hello")

# 与之相对的指令是 
GET_TARGET_PROPERTY(VAR target property)
GET_TARGET_PROPERTY(OUTPUT_VALUE hello_static OUTPUT_NAME) # example
```

动态库版本号

```cmake
SET_TARGET_PROPERTIES(hello PROPERTIES VERSION 1.2 SOVERSION 1)
# VERSION 表示动态库版本，SOVERSION 表示 API 版本

# 之后在 xxx/build/lib 下生成，其中 libhello.so.1 和 libhello.so 是符号链接文件
# libhello.so.1.2
# libhello.so.1 -> libhello.so.1.2
# libhello.so   -> libhello.so.1
```

之后还需要把库和头文件用 `INSTALL` 指令安装到正确的位置，进行后续的使用。

```cmake
INSTALL(TARGETS hello hello_static 
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib)
INSTALL(TARGETS hello hello.hpp DESTINATION include/hello) 
# 头文件安装到 <prefix>/include/hello 中
```



## 2 使用库

```cmake
# 为了找到 hello.hpp 文件，需要
INCLUDE_DIRECTORIES([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...) 
# AFTER 和 BEFORE 控制新增路径放在已存在路径的前面还是后面

# 为 target 添加库
LINK_DIRECTORIES(dir1 dir2)  # 不过不要使用这个指令
TARGET_LINK_LIBRARIES(target lib1 [debug | optimized]
                             lib2 ...)

# 经过试验，上面的 LINK_DIRECTORIES 方法无效。应该按照下面 example 中的方法
# 首先找到库的绝对路径，然后链接绝对路径的库
# 假设 libhello.so 放在 /home/vczn/tmp/lib 中
FIND_LIBRARY(hellolib hello HINTS /home/vczn/tmp/lib) # libhello 必须预先存在
TARGET_LINK_LIBRARY(main ${hellolib})
```



```cmake
# 特殊的环境变量 CMAKE_INCLUDE_PATH 和 CMAKE_LIBRARY_PATH
# 是环境变量，不是cmake变量，可以使用 export 设置
export CMAKE_INCLUDE_PATH=/usr/include/hello

FIND_PATH(header hello.hpp)
IF(header)
INCLUDE_DIRECTORIES(${header})
ENDIF(header)

# FIND_PATH 搜索在指定路径搜索
FIND_PATH(header NAMES hello.hpp PATHS /usr/include /usr/include/hello)

# CMAKE_INCLUDE_PATH 在 FIND_PATH 中使用
# CMKAE_LIBRARY_PATH 在 FIND_LIBRARY 中使用
```



# 三、变量和环境变量

cmake 引用变量的方式 `${var}`，但是 `IF` 等语句中，直接使用变量名而不是通过 `${}` 取值

cmake 有隐式定义和显式定义，比如 `projectname_BINRARY_DIR` 就是隐式定义的变量；显式定义使用 `SET` 指令。

## 1. 常用变量

```cmake
CMAKE_BINARY_DIR 
PROJECT_BINARY_DIR
projectname_BINARY_DIR
# 这三个变量是相同的，如果是 in-source 编译，指工程的顶层目录；
# 如果是 out-source 编译，指编译的目录，一般命名为为 build

CMAKE_SOURCE_DIR
PROJECT_SOURCE_DIR
projectname_SOURCE_DIR
# 这三个变量是相同的，都是工程的顶层目录。

CMAKE_CURRENT_SOURCE_DIR
# 指当前处理 CMakeLists.txt 文件的位置

CMAKE_CURRENT_BINARY_DIR
# 如果是 in-source 编译，和 SOURCE 一样；如果是 out-source 编译，指 target 的编译目录
## ADD_SUBDIRECTORY(src bin) 可以改变这个值
## SET_EXECUTABLE_OUTPUT_PATH dir 不会改变这个变量

CMAKE_CURRENT_LIST_FILE 
# 输出使用这个变量的 CMakeLists.txt 的完整路径

CMAKE_CURRENT_LIST_NAME
# 输出这个变量所在的行

CMAKE_MOUDLE_PATH
# 自定义的 cmake 模块的所在的路径

EXECUTABLE_OUTPUT_PATH 
LIBRARY_OUTPUT_PATH
# 重新定义可执行文件和库文件的目录

PROJECT_NAME
# 项目名称 
```



## 2. 环境变量

```cmake 
# cmake 使用环境变量的方式
$ENV{name}
SET(ENV{name} value)

# e.g.
MESSAGE(STATUS "HOME dir: $ENV{HOME}")
```

```sh
# 常用环境变量 

CMAKE_INCLUDE_CURRENT_DIR 
# 相当于在每个 CMakeLists.txt 中添加了
# INCLUDE_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})

CMAKE_INCLUDE_DIRECTORIES_PROJECT_BEFORE
# 将工程提供的头文件目录始终置于系统头文件目录的前面

CMAKE_INCLUDE_PATH/CMAKE_LIBRARY_PATH 
# 见 2.2 节
```



## 3. 系统信息

```cmake
CMAKE_MAJOR_VERSION     # CMAKE 主版本号，比如 2.4.6 中的 2
CMAKE_MINOR_VERSION     # CMAKE 次版本号，比如 2.4.6 中的 4
CMAKE_PATCH_VERSION     # CMAKE 补丁等级，比如 2.4.6 中的 6
CMAKE_SYSTEM            # 系统名称，比如 Linux-2.6.22
CMAKE_SYSTEM_NAME       # 不包含版本的系统名，比如 Linux
CMAKE_SYSTEM_VERSION    # 系统版本，比如 2.6.22
CMAKE_SYSTEM_PROCESSOR  # 处理器名称，比如 i686.
UNIX                    # 在所有的类 UNIX 平台为 TRUE，包括 OS X 和 cygwin
WIN32                   # 在所有的 win32 平台为 TRUE，包括 cygwin
```



## 4. 开关选项

```cmake
CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS # 用来控制 IF ELSE 语句的书写方式

BUILD_SHARED_LIBS  # 控制默认的库编译方式，如果不进行设置，
# 使用 ADD_LIBRARY 并没有指定库类型的情况下，默认编译生成的库都是静态库
SET(BUILD_SHARED_LIBS ON) # 默认生成动态库 

CMAKE_C_FLAGS/CMAKE_CXX_FLAGS
# 可以 SET 或 ADD_DEFINITIONS 或 ADD_COMPILE_OPTIONS
```



# 四、常用指令

## 1. 基本指令

```cmake
ADD_DEFINITIONS  # 向 C/C++ 编译器添加 -D 宏定义
ADD_DEFINITIONS(-DENABLE_DEBUG -DABC=42)

ADD_DEPENDENCIES # 定义 target 依赖的其他 target
ADD_DEPENDENCIES(target_name depend_target1 depend_target2 ...)

ADD_EXECUTABLE/ADD_LIBRARY/ADD_SUBDIRECTORY

ADD_TEST/ENABLE_TESTING
ADD_TEST(testname Exename arg1 arg2 ...)
ENABLE_TESTING()

ADD_TEST(mytest ${PROJECT_BINARY_DIR}/bin/main)

AUX_SOURCE_DIRECTORY(dir var)
# 作用是发现一个目录下所有的源代码文件并将列表存储到一个变量中
AUX_SOURCE_DIRECTORY(. srcs)

CMAKE_MINIMUM_REQUIRED(VERSION versionNum [FATAL_ERROR])
CMAKE_MINIMUM_REQUIRED(VERSION 2.5 FATAL_ERROR)

EXEC_PROGRAM(executable [dir] 
	[ARGS args] 
	[OUTPUT_VARIABLE <var>] 
	[RETURN_VARIABLE <var>])
	
# 文件相关	
FILE(WRITE filename "message to write" ...)
FILE(APPEND filename "message to write" ...)
FILE(READ filename var)
# 还有其他移动，删除，具体用到再查


INCLUDE(file [OPTIONAL])  # OPTIONAL 参数的作用是文件不存在也不会产生错误
INCLUDE(module [OPTIONAL])
```

## 2. install

```cmake
INSTALL # 见第一章 INSTALL
```

## 3. find_xxx

```cmake
# FIND_
FIND_FILE(<var> name1 path1 path2 ...)    # var 代表找到的文件全路径, name1 表示文件名
FIND_LIBRARY(<var> name1 path1 path2 ...) # 找到的库全路径
FIND_PATH(<var> name1 path1 path2 ...)    # 包含这个文件的全路径
FIND_PROGRAM(<var> name1 path1 path2 ...) # 包含这个程序的全路径
FIND_PACKAGE(<name> [major.minor] [QUIET] [NO_MODULE]
	[[REQUIRED|COMPONENTS] [componets...]])
```

## 4. 控制指令

```cmake
IF(expression1)
	command1(args ...)
	command2(args ...)
	...
ELSE(expression2)
	command1(args ...)
	command2(args ...)
	...
ENDIF(expression3)
# 另外一个是 ELSEIF
# 关于表达式 expression 的判断具体查
# e.g. 判断平台差异
IF(WIN32)
	MESSAGE(STATUS "This is Windows.")
ELSE(WIN32)
	MESSAGE(STATUS "This is not Windows.")
ENDIF(WIN32)

# 这样可能不利于阅读，可以使用之前提到过的开关选项 CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS
## SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON) 之后可以写成
IF(WIN32)
ELSE()
ENDIF()

# e.g. ELSEIF
IF(WIN32)
	...
ELSEIF(UNIX)
	...
ELSEIF(APPLE)
	...
ENDIF(WIN32)
```

```cmake
WHILE(condition)
	command1(args ...)
	command2(args ...)
	...
ENDWHILE(condition)
```

```cmake
# foreach list
FOREACH(loopvar arg1 arg2 ...)
	command1(args ...)
	comaand2(args ...)
	...
ENDFOREACH(loopvar)

# e.g.
AUX_SOURCE_DIRECTORY(. srcs)
FOREACH(src ${srcs})
	MESSAGE(${src})
ENDFOREACH(src)

# foreach range
FOREACH(loopvar RANGE total) # [0, total] 步长为 1
ENDFOREACH(loopvar)

# foreach range and step
FOREACH(loopvar RANGE start stop [step])  # [start, stop]
ENDFOREACH(loopvar)
```



## 5. cmd

```sh
cmake -DCMAKE_BUILD_TYPE=RELEASE|DEBUG
```





# 五、例子

todo: 优化

```cmake
# util
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

SET(util_srcs
	config.cpp
	log_stream.cpp
	logger.cpp
	time_stamp.cpp
	util.cpp
)

SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

ADD_LIBRARY(asuka_util ${util_srcs})
TARGET_LINK_LIBRARIES(asuka_util pthread rt)

INSTALL(TARGETS asuka_util DESTINATION lib)

FILE(GLOB headers "*.hpp")
INSTALL(FILES ${headers} DESTINATION include/util)
```

```cmake
# net
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

SET(net_srcs
	acceptor.cpp
	buffer.cpp
	channel.cpp
	connector.cpp
	default_poller.cpp
	epoller.cpp
	event_loop.cpp
	event_loop_thread.cpp
	event_loop_thread_pool.cpp
	ip_port.cpp
	poller.cpp
	poller_base.cpp
	socket.cpp
	tcp_client.cpp
	tcp_connection.cpp
	tcp_server.cpp
	timer.cpp
	timer_queue.cpp
)

SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

ADD_LIBRARY(asuka_net ${net_srcs})
TARGET_LINK_LIBRARIES(asuka_net asuka_util)

INSTALL(TARGETS asuka_net DESTINATION lib)
FILE(GLOB headers "*.hpp")
INSTALL (FILES ${headers} DESTINATION include/net)
```

```cmake
# main
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(Asuka CXX)

ADD_COMPILE_OPTIONS(-g)
ADD_COMPILE_OPTIONS(-Wall)
ADD_COMPILE_OPTIONS(-Werror)
ADD_COMPILE_OPTIONS(-std=c++14)

SET(CMAKE_CXX_COMPILER "g++")
SET(CMAKE_CXX_FLAGS_DEBUG "-O0")
SET(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNODEBUG")
SET(CMAKE_INSTALL_PREFIX ".")
SET(EXECUTABLE_OUTPUT_DIR ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_DIR ${PROJECT_BINARY_DIR}/lib)

INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR})

ADD_SUBDIRECTORY(src/util)
ADD_SUBDIRECTORY(src/net)

ADD_EXECUTABLE(unit_test unit_test.cpp)
TARGET_LINK_LIBRARIES(unit_test asuka_util asuka_net)

INSTALL(TARGETS unit_test DESTINATION bin)
```

