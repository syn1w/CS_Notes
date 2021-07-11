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

### **(1) 二进制 targets**

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



# 四、高级命令





# 附录A 参考资料

https://cmake.org/documentation/

https://cliutils.gitlab.io/modern-cmake/



# 附录 B CMake 相关变量

## 1. 环境变量



## 2. CMake 变量
