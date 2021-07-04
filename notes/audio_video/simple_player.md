使用 FFmpeg + SDL 制作简单的视频播放器，参考雷霄骅视频内容



# 一、环境搭建

## 1. SDL2

我这里直接使用 vcpkg 安装 SDL2 64-bit 库

```sh
vcpkg install SDL2:x64-Windows
```

后续使用需要编写类似的 CMake 脚本：

```cmake
find_package(SDL2 CONFIG REQUIRED)
target_link_libraries(main PRIVATE SDL2::SDL2 SDL2::SDL2main)
```



## 2. FFmpeg

我这里直接使用 vcpkg 安装 FFmpeg 64-bit 库

```sh
vcpkg install FFmpeg:x64-Windows
```

等待编译安装即可



第一个 FFmpeg 程序

```c++
#include <iostream>

extern "C" {
#include <libavcodec/avcodec.h>
}

using namespace std;

#pragma comment(lib, "bcrypt.lib")

int main(int argc, char* argv[]) {
    cout << "avcodec_configuration:" << endl;
	cout << avcodec_configuration() << endl;
	return 0;
}
```



vcpkg 会自动处理 FFmpeg 头文件和库，因为库使用了 `bcrypt.lib`，需要把这个库链接进来。编译运行，程序输出：

```txt
avcodec_configuration:
--toolchain=msvc --prefix=/d/software/vcpkg/packages/ffmpeg_x64-windows-static/debug --enable-asm --enable-yasm --disable-doc --enable-debug --enable-runtime-cpudetect --disable-openssl --disable-ffmpeg --disable-ffplay --disable-ffprobe --disable-libvpx --disable-libx264 --disable-opencl --disable-lzma --disable-bzlib --enable-avresample --disable-cuda --disable-nvenc --disable-cuvid --disable-libnpp --extra-cflags='-DHAVE_UNISTD_H=0' --debug --extra-cflags=-MTd --extra-cxxflags=-MTd
```



因为 Windows 环境下使用 vcpkg 安装的 SDL2 和 FFmpeg，之后需要指定 `CMAKE_TOOLCHAIN_FILE` 参数

可以使用命令行 `cmake -DCMAKE_TOOLCHAIN_FILE=path/to/vcpkg.cmake ...` 来生成工程文件。

使用 CMake GUI 先点击 `Add Entry` 添加 `CMAKE_TOOLCHAIN_FILE` 参数在进行 `Configure` 

也可以将这个参数变量写死到 CMake 脚本中，需要注意设置 `CMAKE_TOOLCHAIN_FILE` 变量必须在 `project` 命令之前。



这个小的视频播放器主要会用到的 FFmpeg 的库有：`avcodec, avformat, avutil, swscale`



# 二、整体流程

**一般的视频解码流程**：

视频码流一般存储到一定的封装格式中，例如 MP4、AVI 中，封装格式中包含视频码流和音频码流的信息。

对于封装格式的视频，需要解封装格式，将音频压缩数据和视频压缩数据分别进行解码出音频采样数据和视频像素数据，然后进行视音频同步。



**解码流程**：

```c
avformat_open_input(); // 打开封装格式
avformat_find_stream_info(); // 寻找视频基本信息
avcodec_find_decoder(); // 寻找解码器
avcodec_open2(); // 打开解码器

while (AVPacket package = av_read_frame()) {
    avcodec_decode_video2(); // 解码视频
    get AVFrame avframe;
    show on screen;
}

close(...);
```





