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

ssize_t write(int fd, const void *buffer, size_t count);

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



# 五、深入探究文件 I/O

## 1. 原子操作和竞争条件

所有系统调用都是以原子操作方式执行的。内核保证了某系统调用中的所有步骤会作为独立操作而一次性加以执行，其间不会为其他进程或线程所中断  

竞争状态：操作共享资源的两个进程（或线程），其结果取决于一个无法预期的顺序，即这些进程或线程获得 CPU 使用权的先后相对顺序  

**例一**：  

```c
// open 1: check if file exists
int fd = open(argv[1], O_WRONLY);
if (fd != -1) {
    printf(...);
    close(fd);
} else {
    if (errno != ENOENT) {
        errExit("open");
    } else {
        // open 2: create file
        fd = open(argv[1], O_WRONLY | O_CREAT, S_IRUSR | S_IWUSR);
        if (fd == -1) {
            errExit("open");
        }
        
        printf(...);
    }
}
```

当第一次调用 `open` 时，希望打开的文件还不存在，而当第二次调用 `open` 时，其他进程已经创建了该文件  

在这一场景下，进程 A 将得出错误的结论：目标文件是由自己创建的。因为无论目标文件存在与否，进程 A 对 `open` 的第二次调用都会成功  

由于第一个进程在检查文件是否存在和创建文件之间发生了中断，造成两个进程都声称自己是文件的创建者。结合 `O_CREAT` 和 `O_EXCL` 标志来一次性地调用 `open` 可以防止这种情况，因为这确保了检查文件和创建文件的步骤属于一个单一的原子（即不可中断的）操作  



**例二**：  

多个进程同时向同一个文件（例如，全局日志文件）尾部添加数据  

```c
if (lseek(fd, 0, SEEK_END) == -1) {
	errExit("lseek");
}
// ...
if (write(fd, buf, len) != len) {
    fatal(" write failed")
}
```

和上一个例子一样，第一个进程执行到 `lseek` 和 `write` 之间，转而执行第二个进程的相同代码，这两个进程会在写入数据前，将文件偏移量设置为相同的位置，而当第一个进程再次获得调度时，会覆盖第二个进程已写入的数据  

此时再次出现了竞争状态，因为执行的结果依赖于内核对两个进程的调度顺序  

要规避这一问题，需要将文件偏移量的移动与数据写操作纳入同一原子操作。在打开文件时加入 O_APPEND 标志就可以保证这一点  



## 2. fcntl

`fcntl` 根据参数 `cmd` 执行一些控制操作  

```c
#include <unistd.h>
#include <fcntl.h>

int fcntl(int fd, int cmd, ... /* arg */ );
```



接下来会对各种操作进行讨论  



## 3. 打开文件状态

`cmd` 为 `F_GETFL` 获取文件的访问模式和文件状态，`arg` 参数被忽略  

```c
int flags = fcntl(fd, F_GETFL);
if (flags == -1) {
    errExit("fcntl");
}

// test O_SYNC file status
if (flags & O_SYNC) {
    printf("writes are synchronized\n");
}

int accessMode = flags & O_ACCMODE;
if (accessMode == O_WRONLY || accessMode == O_RDWR) {
    printf("file is writable\n");
}
```

使用 `F_SETFL` 命令来修改打开文件的某些状态标志  

允许修改的有 `O_APPEND, O_NONBLOCK, O_NOATIME, O_ASYNC, O_DIRECT`，系统会忽略其他标志的修改(有些 UNIX 实现允许修改其他标志，具体查文档)    

```c
int flags = fcntl(fd, F_GETFL);
if (flags == -1) {
    errExit("fcntl");
}
flags |= O_APPEND;
if (fcntl(fd, F_SETFL, flags) == -1) {
    errExit("fcntl");
}
```



## 4. 文件描述符和打开文件的关系

多个文件描述符可以指向同一打开文件  

内核维护的3个数据结构：  

- 进程级的文件描述符表 
- 系统级的打开文件表  
- 文件系统的 i-node 表  



每个进程，内核为其维护打开文件的描述符（ open file descriptor）表。该表的每一条目都记录了单个文件描述符的相关信息  

- 控制文件描述符操作的一组标志(目前只有 close-on-exec 标志)
- 对打开文件句柄的引用



内核对所有打开的文件维护有一个系统级的描述表格(open file description table)，并将表中各条目称为打开文件句柄（open file handle）。一个打开文件句柄存储了与一个打开文件相关的全部信息：  

- 当前文件偏移量  
- 打开文件时所使用的状态标志(`open` 的 `flags` 参数)  
- 文件访问模式(`O_RDONLY, O_WRONLY, O_RDWR`)  
- 与信号驱动 I/O 相关的设置  
- 对该文件 i-node 对象的引用  



每个文件系统都会为驻留其上的所有文件建立一个 i-node 表，每个文件的 i-node 信息    

- 文件类型和访问权限  
- 指向该文件所持有的锁的列表的指针  
- 文件的各种属性，包括文件大小以及与不同类型操作相关的时间戳  



上述可以推出：

- 两个不同的文件描述符，若指向同一打开文件句柄(不是打开同一文件，而是指向同一打开文件句柄，比如使用 dup 复制)，将共享同一文件偏移量。因此，如果通过其中一个文件描述符来修改文件偏移量，那么从另一文件描述符中也会观察到这一变化。无论这两个文件描述符分属于不同进程，还是同属于一个进程，情况都是如此  
- 要获取和修改打开的文件标志（例如，`O_APPEND`、 `O_NONBLOCK` 和 `O_ASYNC`），可执行 `fcntl` 的 `F_GETFL` 和 `F_SETFL` 操作，其对作用域的约束与上一条类似  
- 文件描述符标志（亦即， close-on-exec 标志）为进程和文件描述符所私有  



## 5. dup

`dup` 复制一个打开的文件描述符  `oldfd`，并返回一个新描述符，二者都指向同一打开的文件句柄  

系统会保证新描述符一定是编号值最低的未用文件描述符  

`dup2` 为 `oldfd` 参数所指定的文件描述符创建副本，其编号由 `newfd` 参数指定，如果由 `newfd` 参数所指定编号的文件描述符之前已经打开，那么 `dup2` 会首先将其关闭，不过 `dup2` 会忽略关闭 `newfd` 出现的任何错误  

`dup3` (始于 Linux 2.6.27)系统调用完成的工作与 `dup2` 相同，只是新增了一个附加参数 `flags`，这是一个可以修改系统调用行为的位掩码，目前只支持 `O_CLOEXEC`  

```c
#include <unistd.h>

int dup(int oldfd);
int dup2(int oldfd, int newfd);

#define _GNU_SOURCE             /* See feature_test_macros(7) */
#include <fcntl.h>              /* Obtain O_* constant definitions */
#include <unistd.h>

int dup3(int oldfd, int newfd, int flags);

// On success, return the new file descriptor
// On error, -1 is returned, and errno is set appropriately
```

`dup` 系列调用可以实现 shell 中 I/O 重定向  

如果 `oldfd` 并非有效的文件描述符，那么 `dup2` 调用将失败并返回错误 `EBADF`，且不关闭 `newfd`。如果 `oldfd` 有效，且与 `newfd` 值相等，那么 `dup2` 将什么也不做，不关闭 `newfd`，并将其作为调用结果返回  



`fcntl` 的 `F_DUPFD` 操作也可以复制文件描述符，且更具有灵活性  

```c
newfd = fcntl(oldfd, F_DUPFD, startfd);
```

该调用为 `oldfd` 创建一个副本，且将使用大于等于 `startfd` 的最小未用值作为描述符编号。该调用还能保证新描述符（`newfd`）编号落在特定的区间范围内。总是能将 `dup` 和 `dup2` 调用改写为对 `close` 和 `fcntl` 的调用，虽然前者更为简洁，还有 `dup` 和 `fcntl` 返回值存在差异  

新文件描述符有其自己的一套文件描述符标志，且其 close-on-exec 标志（`FD_CLOEXEC`）总是处于关闭状态  



## 6. pread/pwrite

`pread` 和 `pwrite` 完成与 `read` 和 `write` 相类似的工作，只是前两者会在 `offset `参数所指定的位置进行文件 I/O 操作，而非始于文件的当前偏移量处，且它们不会改变文件的当前偏移量  

```c
#include <unistd.h>

ssize_t pread(int fd, void *buf, size_t count, off_t offset);
ssize_t pwrite(int fd, const void *buf, size_t count, off_t offset);

// On success, pread() returns the number of bytes read 
// (a return of zero indicates end of file) and 
// pwrite() returns the number of bytes written
// On error, -1 is returned and errno is set to indicate 
// the cause of the error.
```

对 `pread` 和 `pwrite` 而言，必须允许对 `fd` 指向 `lseek` 调用  

多线程应用为这些系统调用提供了用武之地  



## 7. readv/writev

`readv` 和 `writev` 系统调用分别实现了分散输入和集中输出的功能  

```c
#include <sys/uio.h>

ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
ssize_t preadv(int fd, const struct iovec *iov, int iovcnt,
               off_t offset);
ssize_t preadv2(int fd, const struct iovec *iov, int iovcnt,
                off_t offset, int flags);
// On success, return the number of bytes read
// On error, -1 is returned, and errno is set appropriately

ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
ssize_t pwritev(int fd, const struct iovec *iov, int iovcnt,
                off_t offset);
ssize_t pwritev2(int fd, const struct iovec *iov, int iovcnt,
                 off_t offset, int flags);

// On success, return the number of bytes written
// On error, -1 is returned, and errno is set appropriately
```

这些系统调用并非只对单个缓冲区进行读写操作，而是一次即可传输多个缓冲区的数据  

数组 `iov` 定义了一组用来传输数据的缓冲区。整型数 `iovcnt` 则指定了 `iov` 的成员个数。 `iov` 中的每个成员都是如下形式的数据结构  

```c
struct iovec {
    void  *iov_base;
    size_t iov_len;
};
```



SUSv3 标准允许系统实现对 `iov` 中的成员个数加以限制(不少于 16)  

在 `limits.h` 文件中，用 `IOV_MAX` 来确定最大个数  

Linux 把 `IOV_MAX` 设置为 1024  

glibc 对 `readv` 和 `writev` 的封装函数还悄悄做了些额外工作。若系统调用因 `iovcnt` 参数值过大而失败，wrapper 函数将临时分配一块缓冲区，其大小足以容纳 `iov` 参数所有成员所描述的数据缓冲区，随后再执行 `read` 或 `write` 调用  

原子性是 `readv` 的重要属性  



## 8. 截断文件

`truncate` 和 `ftruncate` 系统调用将文件大小设置为 `length` 参数指定的值  

```c
#include <unistd.h>
#include <sys/types.h>

int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
```

若文件当前长度大于参数 length，调用将丢弃超出部分，若小于参数 length，调用将在文
件尾部添加一系列空字节或是一个文件空洞  

两个系统调用之间的差别在于如何指定操作文件。`truncate` 以路径名字符串来指定文件，并要求可访问该文件，且对文件拥有写权限。若文件名为符号链接，那么调用将对其进行解引用。而调用`ftruncate` 之前，需以可写方式打开操作文件，获取其文件描述符以指代该文件，该系统调用不会修改文件偏移量  

若 `ftruncate` 的 `length` 参数值超出文件的当前大小， SUSv3 允许两种行为：要么扩展该文件（如 Linux），要么返回错误。而符合 XSI 标准的系统则必须采取前一种行为。相同的情况，对于 `truncate` 系统调用， SUSv3 则要求总是能扩展文件  



## 9. 非阻塞 I/O

非阻塞 I/O 在打开文件时指定 O_NONBLOCK 标志  

- 若 `open` 调用未能立即打开文件，则返回错误，而非陷入阻塞。有一种情况属于例外，调用 `open`操作 FIFO 可能会陷入阻塞  
- 调用 `open` 成功后，后续的 I/O 操作也是非阻塞的。若 I/O 系统调用未能立即完成，则可能会只传输部分数据，或者系统调用失败，并返回 EAGAIN 或 EWOULDBLOCK 错误。具体返回何种错误将依赖于系统调用  

pipe、FIFO、 套接字、 设备（比如终端、 伪终端） 都支持非阻塞模式  

由于内核缓冲区保证了普通文件 I/O 不会陷入阻塞，故而打开普通文件时一般会忽略 `O_NONBLOCK` 标志。 然而， 当使用强制文件锁时， `O_NONBLOCK` 标志对普通文件也是起作用的  





## 10. 大文件 I/O

通常将存放文件偏移量的数据类型 off_t 实现为一个有符号的长整型(`long`)  

在 32 位体系架构中（比如 x86-32），这将文件大小置于 $2^{31}-1$ 个字节（即 2GB）的限制之下  

32 位 UNIX 实现有处理超过 2GB 大小文件的需求  

始于内核版本 2.4， 32 位 Linux 系统开始提供对 LFS 的支持（glibc 版本必须为 2.2 或更高），相应的文件系统也必须支持大文件操作    

应用程序可使用如下两种方式之一以获得 LFS 功能：  

- 使用支持大文件操作的备选 API  
- 在编译应用程序时，将宏 `_FILE_OFFSET_BITS` 的值定义为 64  

不过在 64 位架构，LFS 增强特性所要突破的限制对其而言并不是问题  



LFS API：

```c
fd = open64(name, O_CREAT | O_RDWR, mode);
off64_t;
struct stat64;
```



## 11. /dev/fd

对每个进程，内核都提供了一个特殊的虚拟目录 `/dev/fd`  

该目录中包含 `/dev/fd/n` 形式的文件名，其中 `n` 是与进程中的打开文件描述符相对应的编号  

SUSv3 对 `/dev/fd` 特性未做规定，但有些其他的 UNIX 实现也提供了这一特性  

打开 `/dev/fd` 目录中文件等同于复制相应的文件描述符，下列两行代码是等价的：  

```c
fd = open("/dev/fd/1", O_WRONLY);
fd = dup(1);
```

在为 `open` 调用设置 `flags` 参数时，需要注意将其设置为与原描述符相同的访问模式  

程序中很少会使用/dev/fd 目录中的文件。其主要用途在 shell 中  



## 12. 临时文件

有些程序需要创建一些临时文件， 仅供其在运行期间使用， 程序终止后即行删除  

GNU C 语言函数库为此而提供了一系列库函数  

介绍其中的两个函数： `mkstemp` 和 `tmpfile`  



基于调用者提供的模板， `mkstemp` 函数生成一个唯一文件名并打开该文件，返回一个可用于 I/O 调用的文件描述符  

```c
#include <stdlib.h>

int mkstemp(char *template);

// On success, return the file descriptor of the temporary file
// On error, -1 is returned, and errno is set appropriately
```

模板参数采用路径名形式，其中最后 6 个字符必须为 `XXXXXX`。这 6 个字符将被替换，以保证文件名的唯一性， 且修改后的字符串将通过 `template` 参数传回。 因为会对传入的 `template` 参数进行修改，所以必须将其指定为字符数组，而非字符串常量  

文件拥有者对 `mkstemp` 函数建立的文件拥有读写权限（其他用户则没有任何操作权限），且打开文件时使用了 `O_EXCL` 标志，以保证调用者以独占方式访问文件  

通常，打开临时文件不久，程序就会使用 unlink 系统调用将其删除，示例程序  

```c
char template[] = "/tmp/someStringXXXXXX";
int fd = mkstemp(template);
if (fd == -1) {
    errExit("mkstemp");
}

printf("tmp file: %s\n", template);
unlink(template);
if (close(fd) == -1) {
    errExit("close");
}
```

`tmpnam`、 `tempnam` 和 `mktemp` 函数也能生成唯一的文件名。然而，由于这会导致应用程序出现安全漏洞，应当避免使用这些函数  



```c
#include <stdio.h>

FILE *tmpfile(void);

// returns a stream descriptor, or NULL if a unique filename 
// cannot be generated or the unique file cannot be opened
// In the latter case, errno is set to indicate the error
```



# 六、进程

进程(process)是一个可执行程序(program)的实例  

程序是包含了一系列信息的文件， 这些信息描述了如何在运行时创建一个进程， 所包括的内容  

- 二进制格式标识，UNIX 可执行文件曾有两种广泛使用的格式，分别为最初的 a.out（汇编程序输出）和更加复杂的 COFF（通用对象文件格式）。现在，大多数 UNIX 实现（包括 Linux）采用可执行连接格式(ELF)
- 机器语言指令  
- 程序入口地址  
- 数据：程序文件包含的变量初始值和程序使用的字面常量（literal constant）值  
- 符号表及重定位表：描述程序中函数和变量的位置及名称。这些表格有多种用途，其中包括调试和运行时的符号解析（动态链接）  
- 共享库和动态链接信息：程序文件所包含的一些字段，列出了程序运行时需要使用的共享库，以及加载共享库的动态链接器的路径名  
- 其他信息：程序文件还包含许多其他信息，用以描述如何创建进程  



进程是由内核定义的抽象的实体，并为该实体分配用以执行程序的各项系统资源  

从内核角度看，进程由用户内存空间(user-space memory)和一系列内核数据结构组成，其中用户内存空间包含了程序代码及代码所使用的变量， 而内核数据结构则用于维护进程状态信息  
记录在内核数据结构中的信息包括许多与进程相关的标识号(ID)、虚拟内存表、打开文件的描述符表、信号传递及处理的有关信息、进程资源使用及限制、当前工作目录和大量的其他信息   



## 1. 进程 ID

每个进程都有一个进程 ID(PID)，PID 是一个正数，用以唯一标识系统中的某个进程。对各种系统调用而言，进程号有时可以作为传入参数，有时可以作为返回值，比如 `kill` 允许调用者向拥有特定进程号的进程发送一个信号    

```c
#include <unistd.h>
 #include <sys/types.h>

// returns the process ID (PID) of the calling process
pid_t getpid(void);

// returns the process ID of the parent of the calling process
pid_t getppid(void);
```



Linux 内核限制进程号需小于等于 32767。新进程创建时，内核会按顺序将下一个可用的进程号分配给其使用。每当进程号达到 32767 的限制时，内核将重置进程号计数器，以便从小整数开始分配  

旦进程号达到 32767，会将进程号计数器重置为 300，而不是 1。之所以如此，是因为低数值的进程号为系统进程和守护进程所长期占用，在此范围内搜索尚未使用的进程号只会是浪费时间  



每个进程的父进程号属性反映了系统上所有进程间的树状关系。每个进程的父进程又有自己的父进程，以此类推，回溯到 1 号进程(init 进程)，即所有进程的始祖  

`pstree` 命令可以查看到这一“家族树”  



## 2. 进程内存布局

每个进程所分配的内存由很多部分组成，通常称之为段(segment，此段非内存硬件分段混淆，此处段主要是 UNIX 系统中进程虚拟内存的逻辑划分，有时候，此处的段也使用术语 section 来代替)    

- text: 文本段(只读)，也叫代码段，包含了进程运行的程序机器语言指令  
- data: 初始化数据段包含显式初始化的全局变量和静态变量  
- bbs: 未初始化数据段包含了未进行显式初始化的全局变量和静态变量  
- statck: 栈是一个动态增长和收缩的段。系统会为每个当前调用的函数分配一个栈帧。栈帧中存储了函数的局部变量（所谓自动变量）、实参和返回值  
- heap: 堆是可在运行时（为变量） 动态进行内存分配的一块区域  



`size` 命令可以显示二进制可执行文件的 text、data、bss 的段大小  

虽然 SUSv3 未作规定，但在大多数 UNIX 实现（包括 Linux）中 C 语言编程环境提供了 3 个全局符号（ symbol）： etext、 edata 和 end，可在程序内使用这些符号以获取相应程序文本段、初始化数据段和非初始化数据段结尾处下一字节的地址  

一个测试验证程序  

```c
#include <stdio.h>

int main() {
    extern char etext, edata, end;
    printf("etext = %p, edata = %p, end = %p\n", &etext, &edata, &end);
    
    return 0;
}

// example output: 
// etext = 0x7f6a82e006fd, edata = 0x7f6a83001010, end = 0x7f6a83001018
```



![6_1](../../imgs/linux/TLPI/6_1.png)



x64 的 Linux 类似，不过内存扩展到了更大的内存空间  

 

## 3. 虚拟内存管理

上述关于进程内存布局存在于虚拟内存中  

Linux，像多数现代内核一样，采用了虚拟内存管理技术。该技术利用了大多数程序的一个典型特征，即访问局部性（ locality of reference），以求高效使用 CPU 和 RAM（物理内存）资源。大多数程序都展现了两种类型的局部性  

- 空间局部性：程序倾向于访问在最近访问过的内存地址附近的内存（由于指令是顺序执行的，且有时会按顺序处理数据结构）  
- 时间局部性（ Temporal locality）：是指程序倾向于在不久的将来再次访问最近刚访问过的内存地址  

虚拟内存的规划之一是将每个程序使用的内存切割成小型的、固定大小的 page 单元。相应地，将 RAM 划分成一系列与虚存页尺寸相同的页帧。任一时刻，每个程序仅有部分页需要驻留在物理内存页帧中。这些页构成了所谓驻留集(resident set)。程序未使用的页拷贝保存在交换区(swap area)内—这是磁盘空间中的保留区域，作为计算机 RAM 的补充—仅在需要时才会载入物理内存。若进程欲访问的页面目前并未驻留在物理内存中，将会发生页面错误(page fault)，内核即刻挂起进程的执行，同时从磁盘中将该页面载入内存  



Linux 系统上，交换分区的使用和内核参数 `swappiness` 的值有关  

- 当 `swappiness == 0`，表示尽最大可能的使用物理内存以避免换入到 swap
- 当 `swappiness == 100` 表示最大限度使用 swap 分区，并且把内存上的数据及时的换出到 swap空间里面  

大多数 Linux 发行版默认值为 60，可以使用 `cat /proc/sys/vm/swappiness` 来查看  

当内存在使用率到 40%(100%-60%) 的时候，系统就会开始出现有交换分区的使用  



为了支持分页，内核为每个进程维护了一张页表(page table)，该页表描述了每页在进程虚拟地址空间（virtual address space)中的位置（可为进程所用的所有虚拟内存页面的集合）。页表中的每个条目要么指出一个虚拟页面在 RAM 中的所在位置，要么表明其当前驻留在磁盘上  



在进程虚拟地址空间中，并非所有的地址范围都需要页表条目。通常情况下，由于可能存在大段的虚拟地址空间并未投入使用，故而也无必要为其维护相应的页表条目。若进程试图访问的地址并无页表条目与之对应，那么进程将收到一个 `SIGSEGV` 信号  

由于内核能够为进程分配和释放页（和页表条目），所以进程的有效虚拟地址范围在其生命周期中可以发生变化，这可能会发生于如下场景：  

- 由于栈向下增长超出之前曾达到的位置  
- 当在堆中分配或释放内存时，通过调用 brk()、 sbrk()或 malloc 函数族来提升 program break 的位置  
- 当调用 `shmat` 连接 System V 共享内存区时， 或者当调用 `shmdt` 脱离共享内存区时  
- 当调用 `mmap` 创建内存映射时，或者当调用 `munmap` 解除内存映射时  



## 4. 栈和栈帧

函数的调用和返回使栈的增长和收缩呈线性。x86 体系架构之上的 Linux，栈驻留在内存的高端并向下增长（朝堆的方向，栈的实际增长方向是实现细节，存在向上增长的体系结构）。专用寄存器—栈指针(stack pointer)，用于跟踪当前栈顶。每次调用函数时，会在栈上新分配一帧，每当函数返回时，再从栈上将此帧移去  

会用用户栈(user stack)来表示此处所讨论的栈，以便与内核栈区分开来。内核栈是每个进程保留在内核内存中的内存区域，在执行系统调用的过程中供（内核）内部函数调用使用  

每个（用户）栈帧包括如下信息:  

- 函数实参和局部变量：由于这些变量都是在调用函数时自动创建的，因此在 C 语言中
  称其为自动变量  
- 调用的链接信息：每个函数都会用到一些 CPU 寄存器，比如程序计数器，其指向下一条将要执行的机器语言指令。 每当一函数调用另一函数时，会在被调用函数的栈帧中保存这些寄存器的副本，以便函数返回时能为函数调用者将寄存器恢复原状  



## 5. 环境变量列表

每一个进程都有与其相关的称之为环境变量列表(environment list)的字符串数组  

其中每个字符串都以 `name=value` 形式定义  

新进程在创建之时，会继承其父进程的环境副本  

可以通过设置环境变量来改变一些库函数的行为。正因如此，用户无需修改程序代码或者重新链接相关库，就能控制调用该函数的应用程序行为  