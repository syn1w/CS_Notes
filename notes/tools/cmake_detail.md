# CMake

# 一、介绍

CMake 是一种跨平台工具，用来自动化构建、测试和打包软件。使用 CMake 的流程是先生成对应平台的工程文件(比如 Windows 平台可以生成 VS 工程，macOS 平台可以生成 Xcode 或 Makefile 文件，Linux 平台可以生成 Makefile 文件等等)，之后再进行对应的操作。

以下内容对 CMake v3.16 进行说明。



# 二、基本概念

## 1. cmake

通过 CMake 语言([CMake language](https://cmake.org/cmake/help/v3.16/manual/cmake-language.7.html#manual:cmake-language(7))) 指定特定平台的构建系统(buildsystem)。

**Source Tree：**在工程中，包含源文件的顶级目录。从顶级目录的叫做 `CMakeLists.txt` 的文件开始，这个文件指定了 buildsystem 、构建的目标及其依赖项。

**Build Tree：**用于存储 buildsystem 文件和构建输出文件(比如可执行文件和库文件)的顶级目录。CMake 在 Build Tree 目录下写入 `CMakeCache.txt` 文件，用来标识为 Build Tree 目录并且存储一些持久的配置信息。

有两种 build 方式，一种是 *in-source*，Source Tree 和 Build Tree 为同一目录，但是不建议使用 in-source 构建；另一种是 *out-of-source*，Source Tree 和 Build Tree 为不同目录，进行构建更加方便。

**Generator**：选择要生成 buildsystem 的种类。目前(v3.16)可以选择的 Generator 见 [cmake generator](https://cmake.org/cmake/help/v3.16/manual/cmake-generators.7.html#manual:cmake-generators(7))。可以使用 `-G` 选项来指定生成器，或者让 CMake 为当前平台选择默认的生成器。



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



## 2. buildsystem

基于 CMake 的构建系统被构建为一系列高级逻辑目标(targets)，每个 target 对应一个可执行文件或库，或者是包含自定义 commands 的自定义 target。在两个 targets 之间的依赖决定了构建顺序和修改之后重新生成的规则。





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



##2. project

```cmake
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```

设置项目的名称，并将其存储到 `PROJECT_NAME` 命令中，从顶层 `CMakeLists.txt` 调用时，还会将项目名称存储到 `CMAKE_PROJECT_NAME` 中。

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

项目的顶层必须包含对 project 的直接调用(不能 include)，如果不存在会发出警告，并且假设有 `project(Project)` 命令并设置默认语言 (C/C++)。



# 四、高级命令





# 附录A 参考资料

https://cmake.org/documentation/

https://cliutils.gitlab.io/modern-cmake/



# 附录 B CMake 相关变量

## 1. 环境变量



## 2. CMake 变量

`PROJECT_NAME`:

`CMAKE_PROJECT_NAME`: