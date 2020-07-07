# Linux/UNIX 系统编程手册

# 一、历史和标准  

## 1. Unix

1969 ~ 1979 年，UNIX 经历了前六版  

1973 年，UNIX 用 C 语言对其进行了重写  

1979 年，UNIX 第七版发布，从该版本起，UNIX 分裂成 BSD 和 System V  

1983 年，加州大学伯克利分校计算机系统研究组发布 4.2 BSD，该版本包含了完整的 TCP/IP 实现，4.2 BSD 及其前身 4.1 BSD 在世界上多所大学开始广为流传。以这两者为基础，还形成了 SunOS 操作系统。其他重要的 BSD 版本还有发布于 1986 年的 4.3BSD，以及发布于 1993 年的最终版本 4.4BSD  

与此同时，1981 年 System III  发布，System III 由 AT&T 所属的 (UNIX Support Group， USG) 研发，1983 年， System V 的首个发布版发布，1989 年，USG 推出了 System V Release 4(SVR4)，此时的 System V 纳入了 BSD 的诸多特性  

除此之外，其他发布版有 SUN 的 SunOS 和 Solaris、Digital 公司的 Ultrix 和 OSF/1(现称为 HP Tru64 UNIX)、IBM 公司的 AIX、HP 公司的 HP-UX、NeXT 公司的 NeXTStep、在 Apple Macintosh 机上的 A/UX 以及 Microsoft 和 SCO 公司联合为 Intel x86-32 架构开发的 XENIX  



## 2. Linux

1991 年， Linus Torvalds  为自己的 Intel 80386 PC 开发操作系统，1991 年 Linux 0.01 版发布，1994 年 3 月，发布了 Linux 1.0 版本 ，Linux 2.0 发布于 1996 年 6 月，  Linux 2.4 发布于 2001  年 1 月，Linux 2.6 发布于 2003 年 12 月  



**内核版本号**  

在 Linux1.0 版本之后，内核版本编号方案为 x.y.z，x 表示主版本号， y 为附属于主版本号的次版本号， z 是从属于次版本号的修订版本号  

采用这一发布模式，内核的两个版本会一直处于开发之中，一个是用于生产系统的 stable 分支，其次版本号为偶数；另一个是经常变动的 development 分支，其次版本号为奇数（当前稳定版次版本号+1），比如 开发分支 2.3.z 的 stable 分支版本为 2.4  

随着 2.6 内核的发布，内核开发模式再次发生改变，稳定内核版本之间发布间隔过长  

之后改变为：

- 不再有稳定内核和开发内核的概念，每个 2.6.z 发布版都可以包含新特性
- 有时候需要为某个稳定的 2.6.z 发布版打补丁，可以直接发布 2.6.z.r 版本，r 作为内核版本的次修订版序号



## 3. POSIX

可移植操作系统 Portable Operating System Interface  

POSIX.1 于 1989 年成为 IEEE 标准，并在稍作修订后于 1990 年被正式采纳为 ISO 标准（ISO/
IEC 9945-1:1990）  

POSIX.1 基于 UNIX 系统调用和 C 语言库函数，但无需与任何特殊实现相关。这意味着任
何操作系统都可以实现该接口，而不一定要是 UNIX 操作系统  

POSIX.2（ 1992， ISO/IEC 9945-2:1993）这一与 POSIX.1 相关的标准，对 shell 和包括 C
编译器命令行接口在内的各种 UNIX 工具进行了标准化  

POSIX 1003.1-2001也将该标准称为 Single Unix Specification 版本 3，本书在后续内容中将称其为 SUSv3，包括了：基本定义 、系统接口、Shell 和实用工具、基本原理  

SUSv4 和 POSIX.1-2008，这一修订版本也称为 SUSv4，相比 SUSv3 修改不大  



# 二、基本概念

## 1. 内核

内核的主要任务：

- 进程调度：Linux 属于抢占式多任务操作系统。”多任务”意指多个进程（即运行中的程序）可同时驻留于内存，且每个进程都能获得对 CPU 的使用权。“抢占”则是指一组规则。这组规则控制着哪些进程获得对 CPU 的使用，以及每个进程能使用多长时间，这两者都由内核进程调度程序（而非进程本身）决定  
- 内存管理：内核必须以公平、高效地方式在进程间共享这一资源  
- 提供了文件系统：内核在磁盘之上提供有文件系统，允许对文件执行创建、获取、更新以及删除等操作  
- 创建和终止进程：内核可将新程序载入内存，为其提供运行所需的资源。一旦进程执行完毕，内核还要确保释放其占用资源  
- 对设备的访问：外部输入输出设备的交互
- 网络协议栈
- 提供系统调用应用编程接口(API)：进程可利用系统调用请求内核执行各种任务



## 2. 文件 I/O 模型

UNIX 系统 I/O 模型最为显著的特性之一是其 I/O 通用性概念，同一套系统调用所执行的 I/O 操作，可以对所有的文件类型使用，包括设备文件等等  

就本质而言，内核只提供了一种文件类型：字节流序列  

**文件描述符 fd**  

I/O 系统调用使用文件描述符(小数值的非负整数)来指代打开的文件  

通常由 shell 启动的进程会继承 3 个已经打开的文件描述符：描述符 0 为标准输入(stdin)；描述符 1 为标准输出(stdout)；描述符 2 为标准错误(stderr)  



## 3. 进程

进程是正在执行的程序实例  

执行程序时，内核会将程序代码载入虚拟内存，为程序变量分配空间，记录与进程有关的各种信
息（比如，进程 ID、用户 ID、组 ID 以及终止状态等）  



**进程的内存布局**：

- 代码段：程序的指令
- 数据段：程序使用的静态或全局变量
- 堆：程序可以从该区域动态分配内存
- 栈：随函数调用、返回而增减的一片内存，用于为局部变量和函数调用链接信息分配存储空间 



**进程的用户和组标识符**：  

每个进程都有一组与之相关的 UID 和 GID  

- 真实 UID 和 GID：用来标识进程所属的用户和组，新进程从其父进程处继承这些 ID  
- 有效 UID 和 GID：进程在访问受保护资源时，会使用这两个 ID (结合下述的补充 GID)来确定访问权限
- 补充 GID：用来标识进程所属的额外组。新进程从其父进程处继承补充组 ID  



**init 进程**  

man init(8)  

系统引导时，内核会创建一个名为 init 的特殊进程，其他进程都由或间接由 init 派生，init 进程 PID 总为 1，且总是以超级用户权限运行，谁都不能杀死 init 进程，只有关闭系统才能终止该进程，init 的主要任务是创建并监控系统运行所需的一系列进程  



**资源限制**  

每个进程都会消耗诸如打开文件、内存以及 CPU 时间之类的资源。使用系统调用 setrlimit()，进程可为自己消耗的各类资源设定一个上限。  

此类资源限制的每一项均有两个相关值：软限制（soft limit）限制了进程可以消耗的资源总量，硬限制（ hard limit）软限制的调整上限。非特权进程在针对特定资源调整软限制值时，可将其设置为 0 到相应硬限制值之间的任意值，但硬限制值则只能调低，不能调高  

由 fork()创建的新进程，会继承其父进程对资源限制的设置  



## 4. 进程间通信及同步

Linux 进程间通信(IPC)机制：

- 信号（ signal），用来表示事件的发生  
- pipe 和 FIFO，用于在进程间传递数据
- socket，供同一台主机或是联网的不同主机上所运行的进程之间传递数据  
- 文件锁：为防止其他进程读取或更新文件内容，允许某进程对文件的部分区域加以锁定  
- 消息队列：用于在进程间交换消息（数据包）  
- 信号量：用来同步进程动作  
- 共享内存：允许两个及两个以上进程共享一块内存  





## 5. 信号

虽然信号是 IPC 的方法之一，但是信号在其他方面的广泛应用则更为普遍  

信号也称为软件中断，进程收到信号，就意味着某一事件或异常情况的发生  

采用不同的整数来标识各种信号类型，并以 SIGXXX 形式的符号名加以定义  

发生某些情况时，比如用户键入 Ctrl C、进程的子进程之一终止、定时器到期、进程访问无效内存等等，内核会向进程发送信号  

进程收到信号，会根据信号采取如下的动作之一：

- 忽略信号
- 被信号终止
- 先挂起，之后再被专用信号唤醒



# 三、系统编程概念

## 1. 系统调用

man syscalls(2)

系统调用是受控的内核入口，借助于这一机制，进程可以请求内核以自己的名义去执行某些动作  

以应用程序编程接口（API）的形式，内核提供有一系列服务供程序访问  

关于系统调用：  

- 系统调用将处理器从用户态切换到核心态，以便 CPU 访问受到保护的内核内存  
- 系统调用的组成是固定的，每个系统调用都由一个唯一的数字来标识  
- 每个系统调用可以有一套参数，对用户空间（亦即进程的虚拟地址空间）与内核空
  间之间（相互）传递的信息加以规范  



以 x86 平台为例来分析系统调用的步骤  

1. 应用程序通过 C 语言库函数中的 wrapper 函数，来发起系统调用  
2. 参数传入到 wrapper 函数，wrapper 函数将这些参数置入特定的寄存器中，来实现系统调用的参数传递  
3. 将系统调用编号赋值到 eax 寄存器中
4. wrapper 函数执行 `int 0x80`  中断指令，处理器从用户态切换到内核态，并执行中断向量所指向的代码(较新的 x86 硬件平台实现了 sysenter 指令，2.6 内核及 glibc 2.3.2 之后的版本支持 sysenter 指令)  
5. 内核调用 `system_call` 函数，来处理中断  
   - 在内核栈中保存寄存器值
   - 校验系统调用编号的有效性
   - 以系统调用编号对存放所有调用服务例程的列表（内核变量 sys_call_table）进行索引，发现并调用相应的系统调用服务例程 。若系统调用服务例程带有参数，那么将首先检查参数的有效性。  随后，该服务例程会执行必要的任务。该服务例程会将结果状态返回给 system_call()例程  
   - 从内核栈中恢复各寄存器值，并将系统调用返回值置于栈中  
   - 返回至  wrapper 函数，同时将处理器切换回用户态  
6. 若系统调用服务例程的返回值表明调用有误， wrapper 函数会使用该值来设置全局变量 errno  
7. 最后，从 wrapper 返回到用户代码



# 四、文件 I/O：通用的 I/O 模型

## 1. 概述

UNIX 的一个核心思想是一切皆文件：所有执行 I/O 操作的系统调用都以文件描述符，一个非负整数（通常是小整数），来指代打开的文件，文件描述符用以表示所有类型的已打开文件，包括 pipe、FIFO、 socket、终端、设备和普通文件。针对每个进程，文件描述符都自成一套  



## 2. open

```c
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *pathname, int flags, ... /* mode_t mode */);

// return the new file descriptor(a nonnegative integer)
// or -1 if an error occurred (in which case, errno is set appropriately)
```

|     flags     |                             用途                             |
| :-----------: | :----------------------------------------------------------: |
|  `O_RDONLY`   |                         只读方式打开                         |
|  `O_WRONLY`   |                         只写方式打开                         |
|   `O_RDWR`    |                         读写方式打开                         |
|  `O_CLOEXEC`  |     设置 close-on-exec 标志（自 Linux 2.6.23 版本开始）      |
|   `O_CREAT`   |                     若文件不存在则创建之                     |
|  `O_DIRECT`   |                      无缓冲的输入/输出                       |
| `O_DIRECTORY` |               如果 `pathname` 不是目录，则失败               |
|   `O_EXCL`    | 确保调用创建文件，如果和 `O_CREAT` 一起使用，如果目录存在，`open` 失败且 `errno` 为 `EEXIST` |
| `O_LARGEFILE` |              在 32 位系统中使用此标志打开大文件              |
|  `O_NOATIME`  |          调用 `read()` 时， 不修改文件最近访问时间           |
|  `O_NOCTTY`   |      不要让 `pathname`（所指向的终端设备）成为控制终端       |
| `O_NOFOLLOW`  |                     对符号链接不予解引用                     |
|   `O_TRUNC`   |                  截断已有文件，使其长度为零                  |
|  `O_APPEND`   |                     总在文件尾部追加数据                     |
|   `O_ASYNC`   |        当 I/O 操作可行时，产生信号（ signal）通知进程        |
|   `O_DSYNC`   |    提供同步的 I/O 数据完整性（自 Linux 2.6.33 版本开始）     |
| `O_NONBLOCK`  |                       以非阻塞方式打开                       |
|   `O_SYNC`    |                      以同步方式写入文件                      |



## 3. creat

```c
#include <fcntl.h>

int creat(const char *pathname, mode_t mode);

// return the new file descriptor(a nonnegative integer)
// or -1 if an error occurred (in which case, errno is set appropriately)
```



## 4. read

```c
#include <unistd.h>

ssize_t read(int fd, void *buffer, size_t count);

// On success, the number of bytes read is returned (zero 
// indicates end of file)
// On error, -1 is returned, and errno is set appropriately
```



## 5. write

```c
#include <unistd.h>

ssize_t write(int fd, void *buffer, size_t count);

// On success, the number of bytes written is returned
// On error, -1 is returned, and 
// errno is set to indicate the cause of the error
```

`read` 和 `write` 的返回值，即读取或写入的字节数可能小于 `count` 值



## 6. close

`close` 系统调用关闭一个打开的文件描述符，并将其释放回调用进程，供该进程继续使用  

当一进程终止时，将自动关闭其已打开的所有文件描述符  

```c
#include <unistd.h>

int close(int fd);

// returns zero on success
// On error, -1 is returned, and errno is set appropriately.
```



## 7. lseek

对于每个打开的文件，系统内核会记录其文件偏移量，有时也将文件偏移量称为读写偏移量或指针。文件偏移量是指执行下一个 read()或 write()操作的文件起始位置，会以相对于文件头部起始点的文件当前位置来表示。文件第一个字节的偏移量为 0  

```c
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);

// returns the resulting offset location as measured in 
// bytes from the beginning of the file
// On error, the value (off_t) -1 is returned and 
// errno is set to indicate the error.
```

`lseek` 调用依照 `offset` 和 `whence` 参数值调整该文件的偏移量  

|  `whence`  |                             含义                             |
| :--------: | :----------------------------------------------------------: |
| `SEEK_SET` |   将文件偏移量设置为从文件头部起始点开始的 `offset` 个字节   |
| `SEEK_CUR` |              将当前文件偏移量 + `offset` 个字节              |
| `SEEK_END` | 将文件偏移量设置为起始于文件尾部的 `offset `个字节，文件尾之后未写数据的字节 |



**文件空洞**  

如果文件的偏移量已经跨越了文件结尾，然后执行I/O 操作，如果是 `read` 将会返回 0；但是 `write` 函数可以在文件结尾的任意位置写入数据  

从文件结尾后到新写入数据间的这段空间被称为文件空洞  

文件空洞不占用任何磁盘空间。直到后续某个时点，在文件空洞中写入了数据，文件系统才会为之分配磁盘块。核心转储文件（ core dump）是包含空洞文件的常见例子  



