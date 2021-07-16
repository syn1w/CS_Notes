# CMake 详解

# 一、介绍

CMake 是一种跨平台工具，用来自动化构建、测试和打包软件。使用 CMake 的流程是编写 CMake 脚本，使用脚本自动生成对应平台的工程文件(比如 Windows 平台可以生成 VS 工程，macOS 平台可以生成 Xcode 或 Makefile 文件，Linux 平台可以生成 Makefile 文件等等)，再进行只会的构建、测试、打包。

以下内容对 CMake v3.16 进行说明。



# 二、基本概念

## 1. CMake

通过 CMake 语言([CMake language](https://cmake.org/cmake/help/v3.16/manual/cmake-language.7.html#manual:cmake-language(7))) 指定特定平台的构建系统(buildsystem)。

**Source Tree**：在工程中，包含源文件的顶级目录。从顶级目录的叫做 `CMakeLists.txt` 的文件开始，这个文件指定了 buildsystem 、构建的目标及其依赖项。

**Build Tree**：用于存储 buildsystem 文件和构建输出文件(比如可执行文件和库文件)的顶级目录。CMake 在 Build Tree 目录下写入 `CMakeCache.txt` 文件，用来标识为 Build Tree 目录并且存储一些持久的配置信息。

有两种 build 方式，一种是 *in-source*，Source Tree 和 Build Tree 为同一目录，但是不建议使用 in-source 构建；另一种是 *out-of-source*，Source Tree 和 Build Tree 为不同目录，进行构建更加方便。

**Generator**：选择要生成 buildsystem 的种类。目前(v3.16)可以选择的 Generator 见 [cmake generator](https://cmake.org/cmake/help/v3.16/manual/cmake-generators.7.html#manual:cmake-generators(7))。可以使用 `-G` 选项来指定生成器，或者让 CMake 为当前平台选择默认的生成器。比如 `-G Unix Makefiles`，`Visual Studio 16 2019`，`Ninja` 等。



运行 `cmake` 命令或使用 cmake-gui 生成 buildsystem。

```sh
cmake [option] <source_dir>
cmake -S src -B build

# option
-S <src>                # Source tree
-B <build>              # Build tree
-C <init-cache>         # Pre-load a script to populate the cache
-D <var>:<type>=<value> # Create or update CMake CACHE entry
-U <var>                # Remove matching entries from CMake CACHE
-G <genor>              # Specify a build system generator
-T <toolset>            # Toolset specification for the generator, if supported
-A <platform>           # Specify platform name if supported by generator
```



**运行构建系统**

可以使用构建系统的原生方法进行构建，比如 Makefile 使用 `make` 命令，Visual Studio 解决方案使用 Visual Studio 进行生成等等。

还可以直接使用 CMake 命令进行构建

```sh
cmake --build <build_dir> [--config Debug|Release|...] [--target target] ...
```



## 2. buildsystem

基于 CMake 的构建系统被构建为一系列高级逻辑目标(targets)，每个 target 对应一个可执行文件或库，或者是包含自定义 commands 的自定义 target。在两个 targets 之间的依赖决定了构建顺序和修改之后重新生成的规则。

### (1) targets

可执行程序和库使用 [`add_executable()`](https://cmake.org/cmake/help/v3.16/command/add_executable.html#command:add_executable) 和 [`add_library()`](https://cmake.org/cmake/help/v3.16/command/add_library.html#command:add_library)  命令。生成的二进制文件在目标平台具有合适的[`PREFIX`](https://cmake.org/cmake/help/v3.16/prop_tgt/PREFIX.html#prop_tgt:PREFIX), [`SUFFIX`](https://cmake.org/cmake/help/v3.16/prop_tgt/SUFFIX.html#prop_tgt:SUFFIX) and extensions。二进制之间的依赖使用 [`target_link_libraries()`](https://cmake.org/cmake/help/v3.16/command/target_link_libraries.html#command:target_link_libraries) 命令。

**二进制可执行程序**

```cmake
add_executable(mytool mytool.cpp)
```

**二进制库**

默认情况下定义静态库 `STATIC`，除非指定类型。[`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/v3.16/variable/BUILD_SHARED_LIBS.html#variable:BUILD_SHARED_LIBS) 变量可以让 `add_library` 命令默认构建动态库。

`MODULE` 库是不同的，通常不会链接，也就不能放到 `target_link_libraries()` 命令的右边。它是一种作为运行时技术作为插件被装载的类型。

如果库没有导出任何非托管符号(比如 Windows 资源 DLL，C++/CLI DLL)，该库不能是 `SHARED` 库，因为 CMake 期待 `SHARED` 库至少导出一个符号。

`OBJECT` 库定义了从给定源代码编译出的目标文件，而不是归档的静态库。

```cmake
# normal libraries
add_library(archive SHARED archive.cpp zip.cpp lzma.cpp)
add_library(archive STATIC archive.cpp zip.cpp lzma.cpp)
```



### (2) build specification

使用 [`target_include_directories()`](https://cmake.org/cmake/help/v3.16/command/target_include_directories.html#command:target_include_directories), [`target_compile_definitions()`](https://cmake.org/cmake/help/v3.16/command/target_compile_definitions.html#command:target_compile_definitions) 和 [`target_compile_options()`](https://cmake.org/cmake/help/v3.16/command/target_compile_options.html#command:target_compile_options) 命令分别指定 [`INCLUDE_DIRECTORIES`](https://cmake.org/cmake/help/v3.16/prop_tgt/INCLUDE_DIRECTORIES.html#prop_tgt:INCLUDE_DIRECTORIES), [`COMPILE_DEFINITIONS`](https://cmake.org/cmake/help/v3.16/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS) 和 [`COMPILE_OPTIONS`](https://cmake.org/cmake/help/v3.16/prop_tgt/COMPILE_OPTIONS.html#prop_tgt:COMPILE_OPTIONS) 或者 [`INTERFACE_INCLUDE_DIRECTORIES`](https://cmake.org/cmake/help/v3.16/prop_tgt/INTERFACE_INCLUDE_DIRECTORIES.html#prop_tgt:INTERFACE_INCLUDE_DIRECTORIES), [`INTERFACE_COMPILE_DEFINITIONS`](https://cmake.org/cmake/help/v3.16/prop_tgt/INTERFACE_COMPILE_DEFINITIONS.html#prop_tgt:INTERFACE_COMPILE_DEFINITIONS) 和 [`INTERFACE_COMPILE_OPTIONS`](https://cmake.org/cmake/help/v3.16/prop_tgt/INTERFACE_COMPILE_OPTIONS.html#prop_tgt:INTERFACE_COMPILE_OPTIONS)。

每个命令都有 `PRIVATE, PUBLIC, INTERFACE` 三种模式，`PRIVATE` 模式仅填充非 interface 变量，`INTERFACE` 模式仅填充 interface 变量，`PUBLIC` 变量填充两种变量。

**目标属性**

在 [`INCLUDE_DIRECTORIES`](https://cmake.org/cmake/help/v3.16/prop_tgt/INCLUDE_DIRECTORIES.html#prop_tgt:INCLUDE_DIRECTORIES) 中的条目在编译命令中添加 `-I` 或 `-isystem` 前缀并且在属性值以适当的顺序。

在 [`COMPILE_DEFINITIONS`](https://cmake.org/cmake/help/v3.16/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS) 中的条目在编译命令中添加 `-D` 或 `/D` 前缀，没有特定的顺序。[`DEFINE_SYMBOL`](https://cmake.org/cmake/help/v3.16/prop_tgt/DEFINE_SYMBOL.html#prop_tgt:DEFINE_SYMBOL) 目标属性也作为编译定义添加，作为 `SHARED` 和 `MODULE` 库目标特殊方便的情况。

在 [`COMPILE_OPTIONS`](https://cmake.org/cmake/help/v3.16/prop_tgt/COMPILE_OPTIONS.html#prop_tgt:COMPILE_OPTIONS) 中的条目对于 shell 会被转义并按照适当的顺序添加，一些选项也会单独处理，比如 [`POSITION_INDEPENDENT_CODE`](https://cmake.org/cmake/help/v3.16/prop_tgt/POSITION_INDEPENDENT_CODE.html#prop_tgt:POSITION_INDEPENDENT_CODE)

NOTE: 关于带有 `INTERFACE` 和不带的区别见[这里](https://stackoverflow.com/questions/52059777/what-is-the-difference-between-include-directories-and-interface-include-directo)，文档没有提到，不是100%确定，大意带有 `INTERFACE` 的 必须是 library target 或者是 interaface library target。





# 三、基本命令

CMake 具有一个相对简单的解释型命令式脚本语言(CMake script language) 。一般放到 Source Tree 目录级别下的 `CMakeLists.txt`，还有一些 `script.cmake` 和 `module.cmake`。

本章，我们对 CMake 基本命令进行比较详细的说明。



## 0. HelloWorld

既然 CMake script 是脚本语言，按照传统，我们从 Hello World 的例子开始。

创建以下目录树：

```txt
├── CMakeLists.txt
└── build
```

然后在 `CMakeLists.txt` 中写入以下内容，

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.8)
project(HelloWorld VERSION 1.0.0 LANGUAGES C)

message(STATUS "Hello World!")
```

之后运行 CMake：

```sh
cd build   # Build Tree Directory
cmake ..

# Possible Output:
# -- The C compiler identification is ...
# ...
# -- Hello World!
# -- Configuring done
# -- Generating done
# -- Build files have been written to: path/to/hello/build
```

可以看到我们使用 [`message`](https://cmake.org/cmake/help/v3.16/command/message.html) 成功输出了 Hello World。`cmake_minimum_required` 和 `project` 之后会详细说明。



`message` 命令会打印一些信息。

```cmake
message([<mode>] "message to display" ...)
```

可选的 `mode` 可以是以下类型:

- `FATAL_ERROR`，CMake Error，停止脚本的处理和生成
- `SEND_ERROR`，CMake Error，继续处理脚本，但是跳过生成过程
- `WARNING`，CMake Warning，继续处理
- `AUTHOR_WARNING`，CMake Warning (dev)，继续处理
- `DEPRECATION`，CMake Deprecation Warning [^注1]
- `NOTICE`，默认的类型(没有 mode 则为 `NOTICE`)，打印到 stderr 上的重要的信息
- `STATUS`，用户可能感兴趣的信息，信息应该尽量简洁，不超过一行，`message` 一般会使用 `STATUS`
- `VERBOSE`，详细的信息，一般不会去看，但是出了某些问题，可能需要看的信息
- `DEBUG`，debug 信息，仅被开发者用于调试目的
- `TRACE`，trace 信息，非常底层实现细节，只是临时的信息，在发布的项目中会删除掉。



[^ 注1]: [文档](https://cmake.org/cmake/help/v3.16/command/message.html)上写取决于 [`CMAKE_ERROR_DEPRECATION`](https://cmake.org/cmake/help/v3.16/variable/CMAKE_ERROR_DEPRECATED.html#variable:CMAKE_ERROR_DEPRECATED) 和 [`CMAKE_WARN_DEPRECATION`](https://cmake.org/cmake/help/v3.16/variable/CMAKE_WARN_DEPRECATED.html#variable:CMAKE_WARN_DEPRECATED)，但是实际测试并不会如此。在 `CMAKE_ERROR_DEPRECATION`  的文档中写道，如果为 `TRUE`，使用了被弃用的功能将报告一个 Fatal Error，关于 `DEPRECATION` 仍有一些疑问。  





## 1. cmake_minimum_required

设置项目需要 CMake 的最低版本

```cmake
cmake_minimum_required(VERSION <min>[...<max>] [FATAL_ERROR])
```

其中 `...` 是字面量，`<max>` 指定最大版本，版本用 `major.minor[.patch[.tweak]]` 格式指定。

`...<max>` 在 CMake 3.12 版本及以上可用。

`FATAL_ERROR` 被 CMake 2.6 及更高版本忽略。

<br>

例子：

```cmake
cmake_minimum_required(VERSION 3.14)
cmake_minimum_required(VERSION 3.12...3.16)
```

<br>

其他说明：

cmake_minimum_required 会隐式调用 `cmake_policy(VERSION)` 命令指定 CMake 版本范围。当 `<min>` 高于 2.4 版本时，隐式调用 `cmake_policy(VERSION <min>[...<max>])`；否则，隐式调用 `cmake_policy(VERSION 2.4[...<max>])`。



## 2. project

```cmake
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```

设置项目的名称，名称只能使由字母、数字、下划线、连字符(`-`) 构成，，并将其存储到 `PROJECT_NAME` 命令中，从顶层 `CMakeLists.txt` 调用时，还会将项目名称存储到 `CMAKE_PROJECT_NAME` 中。

还会设置以下的变量：

- [`PROJECT_SOURCE_DIR`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_SOURCE_DIR.html#variable:PROJECT_SOURCE_DIR), [`<PROJECT-NAME>_SOURCE_DIR`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_SOURCE_DIR.html#variable:_SOURCE_DIR)
- [`PROJECT_BINARY_DIR`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_BINARY_DIR.html#variable:PROJECT_BINARY_DIR), [`<PROJECT-NAME>_BINARY_DIR`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_BINARY_DIR.html#variable:_BINARY_DIR)

以下的变量由可选的参数设置，如果未设置参数，则对应的变量被设置为空字符串。

- `VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]`

    - [`PROJECT_VERSION`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_VERSION.html#variable:PROJECT_VERSION), [`<PROJECT-NAME>_VERSION`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_VERSION.html#variable:_VERSION)
    - [`PROJECT_VERSION_MAJOR`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_VERSION_MAJOR.html#variable:PROJECT_VERSION_MAJOR), [`<PROJECT-NAME>_VERSION_MAJOR`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_VERSION_MAJOR.html#variable:_VERSION_MAJOR)
    - [`PROJECT_VERSION_MINOR`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_VERSION_MINOR.html#variable:PROJECT_VERSION_MINOR), [`<PROJECT-NAME>_VERSION_MINOR`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_VERSION_MINOR.html#variable:_VERSION_MINOR)
    - [`PROJECT_VERSION_PATCH`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_VERSION_PATCH.html#variable:PROJECT_VERSION_PATCH), [`<PROJECT-NAME>_VERSION_PATCH`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_VERSION_PATCH.html#variable:_VERSION_PATCH)
    - [`PROJECT_VERSION_TWEAK`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_VERSION_TWEAK.html#variable:PROJECT_VERSION_TWEAK), [`<PROJECT-NAME>_VERSION_TWEAK`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_VERSION_TWEAK.html#variable:_VERSION_TWEAK).

- `DESCRIPTION <project-description-string>`

    [`PROJECT_DESCRIPTION`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_DESCRIPTION.html#variable:PROJECT_DESCRIPTION), [`_DESCRIPTION`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_DESCRIPTION.html#variable:_DESCRIPTION)

- `HOMEPAGE_URL <url-string>`

    [`PROJECT_HOMEPAGE_URL`](https://cmake.org/cmake/help/v3.16/variable/PROJECT_HOMEPAGE_URL.html#variable:PROJECT_HOMEPAGE_URL), [`_HOMEPAGE_URL`](https://cmake.org/cmake/help/v3.16/variable/PROJECT-NAME_HOMEPAGE_URL.html#variable:_HOMEPAGE_URL)

- `LANGUAGES <language-name>...`

    支持的语言：`C, CXX(i.e. C++), CUDA, OBJC(i.e. Objective-C), OBJCXX, Fortran, ASM`

项目的顶层必须包含对 project 的直接调用(不能 include)，如果不存在会假设有 `project(Project)` 命令并设置默认语言 (C/C++)。



例子：

```cmake
project(MyProject VERSION 1.0.0)
project(MyProject VERSION "1.0.0")
project(MyProject VERSION 1.0.0 LANGUAGES CXX)
project(MyProject VERSION 1.0.0 LANGUAGES C CXX ASM)
```



## 3. add_executable

使用指定的源代码添加 executable target

 ```cmake
 add_executable(<name> [WIN32] [MACOSX_BUNDLE]
                [EXCLUDE_FROM_ALL]
                [source1] [source2 ...])
 ```

`WIN32` 选项表示将构建 Win32 GUI 应用程序，程序入口为 `WinMain()`，而且增加链接选项 `/SUBSYSTEM:WINDOWS`，其他平台会自动忽略 `WIN32` 选项

`MACOSX_BUNDLE`，个人对 macOS 不了解，有需要可以自己查看 [MACOSX_BUNDLE](https://cmake.org/cmake/help/v3.16/prop_tgt/MACOSX_BUNDLE.html#prop_tgt:MACOSX_BUNDLE) 文档

`EXCLUDE_FROM_ALL`，项目定义了许多 targets，当没有指定 target 就对项目进行构建时，会对 `ALL` 这个特殊的 target 进行构建，比如在生成的 Visual Studio 解决方案可以看到一个 `ALL_BUILD` 的项目。使用 `add_executable` 指定的 target 默认在 `ALL` target 中。

sources 可以指定该 target 需要的源代码文件，如果是 C/C++ 项目，除了 C/C++ 源码(`.c, .cpp, .cc` 等)，建议把所需本项目的头文件也添加进去，虽然对构建没有影响，添加头文件可以在 Visual Studio 的工程中更方便的找到头文件

例子：

```cmake
add_executable(hello hello.c)
add_executable(foo EXCLUDE_FROM_ALL foo.h foo.c main.c)
```



## 4. add_library

使用指定的源代码添加 library target

```cmake
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [<source>...])
```

`STATIC` 指定构建为静态库。在 Windows 上生成 `name.lib`，Unix 上生成 `libname.a`

`SHARED` 指定构建为动态库。在 Windows 上生成 `name.dll`，macOS 上为 `libname.dylib`，其他 Unix 上一般为 `libname.so`

`MODULE` 指定构建为模块(module)。类似于动态库，不同的是 module 只是作为插件不会被其他 targets 链接进去，只会在运行时使用类似于 `dlopen` 的方法动态加载模块。

如果没有显式注明构建库类型，会使用 `BUILD_SHARED_LIBS` 变量决定构建类型。如果 `BUILD_SHARED_LIBS` 设置为 `true`，将会构建动态库；否则，构建静态库（默认）。

```sh
cmake -DBUILD_SHARED_LIBS=true source
```



一些好的习惯：

- 命名库时，尽量不要以 `lib` 作为库的开头，因为在大部分非 Windows 平台，构建库会自动添加 `lib` 前缀，如果以 `lib` 开头，则生成的库以 `liblibxxx.xx` 命名。
- 除非有硬性需求，可以避免使用指定 `STATIC` 或 `SHARED`，在构建时，通过 `BUILD_SHARED_LIBS` 变量来选择构建静态库还是动态库。



## 5. target_link_libraries

为给定的 target 指定其需要链接的依赖库

```cmake
target_link_libraries(<target> <PRIVATE|PUBLIC|INTERFACE> items1...
                               [<PRIVATE|PUBLIC|INTERFACE> items2...]...)
```

target 必须是使用 `add_executable` 或 `add_library` 创建的可执行文件或库 target，不能是 [ALIAS target](https://cmake.org/cmake/help/v3.16/manual/cmake-buildsystem.7.html#alias-targets)



关于库依赖传递可以见 [transitive-usage-requirements](https://cmake.org/cmake/help/v3.16/manual/cmake-buildsystem.7.html#transitive-usage-requirements)

`PRIVATE`：在 `PRIVATE` 之后的库和目标被链接到 target，但是不会成为 target 链接接口的一部分。如果依赖项仅使用库的实现，即 target 的源文件需要依赖 A 库，而 target 的头文件不需要依赖 A 库，通常会采用 `PRIVATE A`。

`PUBLIC`：在 `PUBLIC` 之后的库和目标会被链接到 target，而且会成为 target 链接接口的一部分。如果 target 的头文件依赖了 A 库(比如继承 A 库的类)，则应该将其指定为 `PUBLIC A`。

`INTERFACE`：如果库的实现不使用库而仅头文件使用依赖项应该指定为 `INTERFACE`，即 target 的头文件包含了 A 库的头文件，但是 target 的源文件没有使用 A 库，可以使用 `INTERFACE A`。

如果不显式注明链接类型(`PRIVATE|PUBLIC|INTERFACE`)，写成 `target_link_libraries(<target> <item>...)` 这样的形式，默认一般是可传递的(`PUBLIC`)，如果和新式链接形式混合使用，则一般是 `PRIVATE`，而且还有其他复杂的情况，具体见[这里](https://cmake.org/cmake/help/v3.16/command/target_link_libraries.html#id4)。在新项目中尽量显式注明链接类型。



items 一般为库名，CMake 可以是以下内容：

- CMake target name，将包含与目标关联的可链接库文件的完整路径。如果库文件更改，构建系统将依赖于重新链接。
- 库完整的路径，通常会保留库的完整路径，如果库文件修改，构建系统则会重新进行链接。
- 非 CMake target name，将要求链接器搜索库。比如 `-lpthread`
- link flag，以 `-` 开始，但是不能是 `-l` 或 `-framework`，link flags 指定链接标志插入到链接库命令中，具体依赖于链接器（不同的链接器 flags 不一样）。也可以使用 `LINK_OPTIONS` 目标属性或 `target_link_options()` 命令显式添加链接标志。比如在 Unix 上，静态库的依赖顺序可能会造成链接错误，可以使用 `-Wl,--start-group lib1 lib2 ... -Wl,--end-group` 来解决静态库的依赖顺序导致的链接错误。
- 生成器表达式(generator expression)，`$<...>` 生成器表达式可以计算为上述任何项或以分号分隔的列表，比如 `${libs}` 等等，生成器表达式之后详细讨论。

<br>



# 四、CMake 脚本语言

CMake 脚本支持 CMake 命令、变量、字符串操作、数组、函数、宏和模块包含等等。

本章主要说明 CMake 变量、控制语句(`if`、`foreach`、`while`、`break`、`continue`)、工程目录结构、函数(`function`)、宏(`macro`)、表达式等

## 1. Variable

CMake script 和其他的编程语言类似，也有变量的概念。

`set` 用来设置 [CMake 变量](https://cmake.org/cmake/help/v3.16/manual/cmake-language.7.html#cmake-language-variables)的值，`unset` 用来取消设置 CMake 变量。除了 `set/unset`，也有其他的命令可以修改变量值的语义。

变量名是大小写敏感的，变量名几乎可以由大多数字符构成，但是建议使用类似其他编程语言的变量名，仅使用字母、数字、下划线。

变量有动态作用域，CMake 变量的作用域：

- 函数作用域。在由 `function()` 命令创建的函数内，使用 `set/unset` 设置/移除变量绑定在此作用域，并对当前函数和其中的任何嵌套调用可见，但在函数返回后不可见。
- 目录作用域。在 Source Tree 中，每个目录有自己的变量绑定。在处理当前目录的 `CMakeLists.txt` 之前，CMake 会拷贝父目录所有的变量绑定到该目录。
- 持久缓存作用域。CMake 存储一组单独的缓存变量("cache" variable)，会被存储到 `CMakeCache.txt`，其值在项目 Build Tree 中的多次运行中持续存在，除非显式移除或删除缓存。



接下来从简单到复杂对变量的各种情况进行说明。

**普通变量**

```cmake
set(<variable> <value>... [PARENT_SCOPE])
unset(<variable> [PARENT_SCOPE])
```

在当前目录或函数作用域内设置/取消普通变量。所有的变量值的类型都是字符串类型，虽然可能有些命令将字符串解释为其他类型。当设置一个值时，不要求使用引号引起来，除非值中包含空格。如果给定多个值，值会被多个分号连接在一起，例如：

```cmake
set(var "")            # var = ""
set(var value)         # var = "value"
set(var "value")       # var = "value"
set(var "a b")         # var = "a b"
set(var a b)           # var = "a;b"
set(var a;b)           # var = "a;b"
set(var a b;c)         # var = "a;b;c"
set(var a "b c")       # var = "a;b c"
```

[变量引用](https://cmake.org/cmake/help/v3.16/manual/cmake-language.7.html#variable-references)可以通过 `${variable}` 来获得，可以在任何需要的地方使用，CMake 可以递归使用变量来取得另一个变量值。例如：

```cmake
set(var1 ab)         # var1 = "ab"
set(var2 ${var1}cd)  # var2 = "abcd"
set(var3 ${var1} cd) # var3 = "ab;cd"

set(bavar ba)      # bavar = "ba"
set(bar ab)        # bar = "ab"
set(foo "${${bavar}r}cd") # foo = "${bar}cd" = "abcd"
set(${foo} xxx)    # abcd = "xxx"
```

CMake 变量值也可以是多行字符串，还可以包含引号。

```cmake
set(var1 "First Line
Second Line with \"quoted\"")

set(var1 "First Line\nSecond Line with \"quoted\"")

set(var2 [[
First Line
Second Line with "quoted"
]])

set(command [=[
#!/bin/bash
[[ -n ${PATH} ]] && echo "PATH: ${PATH}"
]=])
```



移除变量的设置：

```cmake
set(var)
unset(var)
```

上述两者是等价的，都是移除变量的设置，如果 `var` 不存在，也不会报错。移除之后变量会变成未定义。

注意：CMake 变量引用的规则是首先搜索函数作用域的普通变量，如果不存在，搜索目录作用域的普通变量。如果都不存在，则搜索具有该名称的缓存变量。由于 unset 普通变量，可能导致被隐藏的缓存变量暴露出来，如果强制 `${var}` 返回空字符串，可以使用 `set(var "")` 。

<br>

**环境变量**

```cmake
set(ENV{<variable>} [<value>])
unset(ENV{<variable>})
$ENV{variable}                 # env variable reference
```

对环境变量的修改只对当前项目有效，不会影响全局的环境变量。CMake 允许结束，对环境变量的更改就会就会丢失，同样，在构建项目时更改也不可见。

<br>

**缓存变量**

```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
unset(<variable> CACHE)
option(<variable> "<help_text>" [ON | OFF])
```

缓存变量比普通变量附带更多的信息，包括类型和文档字符串，设置缓存变量时必须提供这两个参数。类型和文档变量都不会影响 CMake 的处理，只是为了更好地展示给用户。

类型可以是以下之一：

- `BOOL`：变量值底层仍然是字符串，`BOOL` 类型的值可以是 `ON/OFF, TRUE/FALSE, 1/0`  
- `FILEPATH`，表示某个文件的路径
- `PATH`，和 `FILEPATH` 类似
- `STRING`，直接作为字符串处理
- `INTERNAL`，对用户不可见，内部缓存变量有时用于持久地记录项目的内部信息，不会显示给用户。这个类型也会带有隐式 `FORCE` 参数

如果在 `set` 调用之前缓存变量不存在或给定了 `FORCE` 参数，则缓存条目将设置为给定值。此外，当前范围内的任何普通变量绑定都将被删除，以将新缓存的值暴露给任何紧随其后的变量引用。

使用 `set` 缓存 `BOOL` 变量可能会比较繁琐，可以直接使用 `option` 命令来缓存 `BOOL`。`option` 默认值是 `OFF`，如果 `<variable>` 已被设置为普通变量，则 `option` 不执行任何操作。

例子：

```cmake
set(TESTING FALSE CACHE BOOL "enable testing")
option(TESTING "enable testing" OFF)
set(CMAKE_CXX_STANDARD 14 CACHE STRING "C++ standard to conform to")
```

为了造成不必要的麻烦，建议谨慎对变量进行命名。

<br>

**CMake 工具设置缓存变量**

CMake 命令行工具可以使用 `cmake -Dvar:type=value` 的方式来设置缓存变量，使用  `-U var` 从缓存中删除变量，支持 `*` 和 `?` 通配符。

CMake GUI 可以点击 "Add Entry" 或 "Remove Entry" 添加或移除缓存变量。

<br>



## 2. operations

前面介绍了变量，CMake 变量本质都是字符串，有些操作会字符串解释为其他类型。我们这一节主要介绍对变量值(字符串)的操作。

### (1) string

CMake 有 `string` 操作，提供了对字符串的处理功能。主要功能有查找和替换、正则表达式、操纵（附加、大小写转换、获取长度、字串等）、比较、计算哈希、生成（比如生成随机字符串、时间戳、UUID 等）。在 3.19 版本新增了 JSON 相关操作。

**查找**

```cmake
string(FIND <string> <substring> <output_variable> [REVERSE])
```

查找操作的第一个参数 `<string>` 为输入字符串，第二个参数 `substring` 为期待查找的子串，`output_variable` 为查找结果被存储到输出变量中。处理所有的字符为 ASCII 值，所以输出的索引以字节为单位计算。如果找到子串，则输出 `string` 中首个匹配字串的位置，从 0 开始；否则输出 `-1`。`[REVERSE]` 表示从后往前查找。

例如：

```cmake
set(s1 "This is a string")
set(s2 "is")
string(FIND ${s1} ${s2} pos1)            # pos1 = "2"
string(FIND ${s1} ${s2} pos2 REVERSE)    # pos2 = "5"
string(FIND ${s1} "this" pos3)           # pos3 = "-1"
```



**替换**

```cmake
string(REPLACE <match_string> <replace_string>
       <output_variable> <input> [<input>...])
```

用 `replace_string` 把在 `input` 中的所有匹配到 `match_string` 字符串替换到，替换的结果存储到 `output_variable`。当然替换操作不会修改原字符串

当给定多个 `input` 时，首先将所有的 `input` 连接起来（类似于 std::string 的 append 操作），两个字符串之间没有任何分隔符。

例如：

```cmake
set(s1 "abcd")
string(REPLACE "bc" "yz" out1 s1)        # out1 = "ayzd"
string(REPLACE "da" "wx" out2 s1 s1 s1)  # out2 = "abcwxbcwxbcd"
```



**正则表达式**

```cmake
string(REGEX MATCH <regular_expression> <output_variable> <input> [<input>...])
string(REGEX MATCHALL <regular_expression> <output_variable> <input> [<input>...])
string(REGEX REPLACE <regular_expression> <replacement_expression>
       <output_variable> <input> [<input>...])
```

`<input>` 和 `string(REPLACE ...)` 的处理方式相同

`MATCH` 正则表达式匹配一次，将匹配到的结果字符串写到 `output_variable`

`MATCH_ALL` 正则表达式匹配多次，将可能匹配到的结果字符串作为 `list` 写入到 `output_variable`

`REPLACE`  正则表达式匹配多次，使用 `replacement_expression` 将所有可能匹配到的字符串替换

CMake 3.16 支持的正则表达式语法见[这里](https://cmake.org/cmake/help/v3.16/command/string.html#regex-specification)



**操纵字符串**

```cmake
string(APPEND <string_variable> [<input>...])
string(PREPEND <string_variable> [<input>...])

string(CONCAT <output_variable> [<input>...])
string(JOIN <glue> <output_variable> [<input>...])
string(TOLOWER <string> <output_variable>)
string(TOUPPER <string> <output_variable>)
string(LENGTH <string> <output_variable>)
string(SUBSTRING <string> <begin> <length> <output_variable>)
string(STRIP <string> <output_variable>)
string(REPEAT <string> <count> <output_variable>)
```

`APPEND` 修改 `string_variable>`，在其后 append 所有的 `input`，`PREPEND` 和 `APPEND` 类似，不过插入到其开始位置，而不是末尾。

`CONCAT` 连接操作。`JOIN` 使用 `glue` 将字符串拼接起来，比如 `glue` 是 `, `，所有的 `input` 是 `"abc" "123" "def"`，则结果是 `abc, 123, def`

`TOLOWER` 和  `TOUPPER` 将 `string` 分别转换为小写和大写，`LENGTH` 计算字符串长度，`SUBSTRING` 是输出 `string` 的子字符串到 `output_variable`

`STRIP` 将删除了前导空格和末尾空格的 `string` 存储到 `output_variable`

`REPEAT` 生成 `string` 重复 `count` 次的结果写入 `output_variable`



**比较操作**

```cmake
string(COMPARE LESS <string1> <string2> <output_variable>)
string(COMPARE GREATER <string1> <string2> <output_variable>)
string(COMPARE EQUAL <string1> <string2> <output_variable>)
string(COMPARE NOTEQUAL <string1> <string2> <output_variable>)
string(COMPARE LESS_EQUAL <string1> <string2> <output_variable>)
string(COMPARE GREATER_EQUAL <string1> <string2> <output_variable>)
```

将比较的结果 `true` 或 `false` 写入到 `output_variable`



**哈希操作**

```cmake
string(<HASH> <output_variable> <input>)
```

可选的 `HASH` 有：`MD5, SHA1, SHA224, SHA256, SHA384, SHA512, SHA3_224, SHA3_256, SHA3_384, SHA3_512`





# 五、高级主题





# 附录A 参考资料

https://cmake.org/documentation/

https://cliutils.gitlab.io/modern-cmake/



# 附录 B CMake 相关变量

## 1. 环境变量



## 2. CMake 变量
