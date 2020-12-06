# Linux/UNIX 系统编程手册 下

# 三十四、进程组和会话

## 1. 概述

进程组由一个或多个共享同一进程组标识符(`PGID`) 的**进程**组成  

会话是一组**进程组**的集合。进程的会话成员关系是由其 session 标识符(`SID`)确定的，会话标识符与 `PGID` 一样，是一个类型为 `pid_t` 的数字  

在任一时刻，会话中的其中一个进程组会成为终端的前台进程组，其他进程组会成为后台进程组。只有前台进程组中的进程才能从控制终端中读取输入。  

当控制终端的连接建立起来之后，会话首进程(shell，比如 bash)会成为该终端的控制进程。成为控制进程的主要标志是当断开与终端之间的连接时内核会向该进程发送一个 `SIGHUP` 信号。  

会话和进程组的主要用途是用于 shell 作业控制  



## 2. 进程组

```c
#include <unistd.h>

pid_t getpgrp(void);
// always successfully return process group ID of calling process
```

如果 `getpgrp()` 的返回值与调用进程的进程 ID 匹配的话就说明该调用进程是其进程组的首进程  

`setpgid()` 系统调用将进程 ID 为 `pid` 的进程的进程组 ID 修改为 `pgid`  

```c
#include <unistd.h>

int setpgid(pid_t pid, pid_t pgid);
// return 0 on success or -1 on error
```

如果传入 `pid` 的值设置为 0，那么调用进程的进程组 ID 就会被改变。如果传入 `pgid` 的值设置为 0，那么 ID 为 `pid` 的进程的进程组 ID 会被设置成 `pid` 的值  

`setpgid` 限制：

- `pid` 参数仅可以指定调用进程或其中一个子进程。违反这条规则会导致 `ESRCH` 错误  
- 在组之间移动进程时，调用进程、由 `pid` 指定的进程以及目标进程组必须要属于同一个会话。违反这条规则会导致 `EPERM` 错误。  
- `pid` 参数所指定的进程不能是会话首进程。违反这条规则会导致 `EPERM` 错误。  
- 一个进程在其子进程已经执行 `exec()` 后就无法修改该子进程的进程组 ID 了。 违反这条规则会导致 `EACCES` 错误  





## 3. 会话

```c
#include <unistd.h>

pid_t getsid(pid_t pid);
// return session ID of specified process, or (pid_t)-1 on error

pid_t setsid(void);
// return session ID of new session, or (pid_t)-1 on error
```

如果 `pid` 参数的值为 0，那么 `getsid()` 会返回调用进程的会话 ID  

如果调用进程不是进程组首进程，那么 `setsid()` 会创建一个新会话  

`setsid()` 系统调用会按照下列步骤创建一个新会话：  

- 调用进程成为新会话的首进程和该会话中新进程组的首进程。调用进程的进程组 ID 和会话 ID 会被设置成该进程的进程 ID  
- 调用进程没有控制终端。所有到之前控制终端的连接都会被断开  

如果调用进程是一个进程组首进程，那么 `setsid()` 调用会报出 `EPERM` 错误。避免这个错误发生的最简单的方式是执行一个 `fork()` 并让父进程终止以及让子进程调用 `setsid()`。由于子进程会继承其父进程的进程组 ID 并接收属于自己的唯一的进程 ID，因此它无法成为进程组首进程。  



## 4. 控制终端和控制进程

一个会话中的所有进程可能会拥有一个控制终端。会话在被创建出来的时候是没有控制终端的， 当会话首进程首次打开一个还没有成为某个会话的控制终端的终端时会建立控制终端， 除非在调用 `open()`时指定 `O_NOCTTY` 标记。一个终端至多只能成为一个会话的控制终端  

如果一个进程拥有一个控制终端，那么打开特殊文件`/dev/tty` 就能够获取该终端的文件描述符。  

如果使用 `ioctl(fd, TIOCNOTTY)` 删除进程与文件描述符 `fd` 指定的控制终端之间的关联关系，之后再试图打开 `/dev/tty` 文件就会失败  



`ctermid` 获取表示控制终端段路径名：

```c
#include <stdio.h>

char *ctermid(char *ttyname);
// The pointer to the pathname
```

通常会生成字符串 `/dev/tty`  



## 5. 前台和后台进程组

`tcgetpgrp()` 和 `tcsetpgrp()` 函数分别获取和修改一个终端的前台进程组。  

```c
#include <unistd.h>

pid_t tcgetpgrp(int fd);
// return process group ID of terminal's foreground process group or -1 on error

int tcsetpgrp(int fd, pid_t pgid);
// return 0 on success or -1 on error
```

`fd` 所指定的终端，该终端必须是调用进程的控制终端  



## 6. `SIGHUB`

当一个控制进程失去其终端连接之后，内核会向其发送一个 `SIGHUP` 信号  

`SIGHUP` 信号的默认处理方式是终止进程。如果控制进程处理了或忽略了这个信号，那么后续尝试从终端中读取数据的请求就会返回文件结束的错误  

向控制进程发送 `SIGHUP` 信号会引起一种链式反应，从而导致将 `SIGHUP` 信号发送给很多其他进程。这个过程可能会以下列两种方式发生：  

- 控制进程通常是一个 shell。 shell 建立了一个 `SIGHUP` 信号的处理器，这样在进程终止之前，它能够将 `SIGHUP` 信号发送给由它所创建的各个任务  
- 在终止终端的控制进程时，内核会解除会话中所有进程与该控制终端之间的关联关系以及控制终端与该会话的关联关系，并且通过向该终端的前台进程组的成员发送 `SIGHUP` 信号来通知它们控制终端的丢失  



# 三十五、进程优先级和调度

## 1. 进程优先级

Linux 与大多数其他 UNIX 实现一样，调度进程使用 CPU 的默认模型是循环时间共享。在这种模型中，每个进程轮流使用 CPU 一段时间，这段时间被称为时间片  

进程特性 nice 值（优先级）允许进程间接地影响内核的调度算法。每个进程都拥有一个 nice 值，其取值范围为 -20（高优先级）到 19（低优先级），默认值为 0  

非特权进程只能降低自己的优先级，这样做之后它们就对其他进程 "友好(nice)"了  

nice 值是一个权重因素，它导致内核调度器倾向于调度拥有高优先级的进程。  

```c
#include <sys/resource.h>

int getpriority(int which, id_t who);
// return nice value os specified process on success or -1 on error
int setpriority(int which, id_t who, int prio);
// return 0 on success or -1 on error
```

参数 `which`：

- `PRIO_PROCESS`：操作 `PID` 为 who 的进程，如果 who 为 0，那么使用调用者的进程
- `PRIO_PGRP`：操作进程组 ID 为 who 的进程组中的所有成员，如果 who 为 0，那么为调用者的进程组
- `PRIO_USER`：操作所有真实用户 ID 为 who 的进程，如果 who 为 0，那么使用调用者的真实用户 ID

试图将 nice 值设置为一个超出允许范围的值 `[-20, 19]` 时会直接将 nice 值设置为边界值  



## 2. 实时进程调度

标准的内核调度算法一般能够为这些进程提供足够的性能和响应度。但实时应用对调度器有更加严格的要求  

有对实时的调度进行支持，但是没有达到硬实时的要求，实时调度需要再进行讨论  



## 3. 实时进程调用 API

Linux 实时和和非实时的调度策略：  

实时：

- `SCHED_FIFO`：实时先入先出
- `SCHED_RR`：实时循环

非实时：

- `SCHED_OTHER`：标准的时间片
- `SCHED_BATCH`：与 `SCHED_OTHER` 类似，用于批量执行
- `SCHED_IDLE`：与 `SCHED_OTHER` 类似，但优先级比最大的 nice 值(+19) 还要低

其他同上节，之后需要再讨论



## 4. affinity

进程切换 CPU 时对性能会有一定的影响：如果在原来的 CPU 的高速缓冲器中存在进程的数据，那么为了将进程的一行数据加载进新 CPU 的高速缓冲器中，首先必须使这行数据失效  

Linux（ 2.6）内核尝试了给进程保证软 CPU affinity(亲和力)—在条件允许的情况下进程重新被调度到原来的 CPU 上运行  

```c
#include <sched.h>

int sched_setaffinity(pid_t pid, size_t len, cpu_set_t* set);
int sched_getaffinity(pid_t pid, size_t len, cpu_set_t* set);
// reutrn 0 on success or -1 on error

void CPU_ZERO(cpu_set *set);
void CPU_SET(int cpu, cpu_set_t *set);
void CPU_CLR(int cpu, cpu_set_t *set);

int CPU_ISSET(int cpu, cpu_set_t *set);
// return 1 if cpu is set, or 0 otherwise
```

`<sched.h>` 头文件定义了常量 `CPU_SETSIZE`，通常为 1024，`len` 为指定 set 参数中的字节数，即为 `sizeof(cpu_set_t)` 



# 三十六、进程资源

## 1. 资源统计

```c
#include <sys/resource.h>

struct rusage {
    struct timeval ru_utime; /* user CPU time used */
    struct timeval ru_stime; /* system CPU time used */
    long   ru_maxrss;        /* maximum resident set size */
    long   ru_ixrss;         /* integral shared memory size */
    long   ru_idrss;         /* integral unshared data size */
    long   ru_isrss;         /* integral unshared stack size */
    long   ru_minflt;        /* page reclaims (soft page faults) */
    long   ru_majflt;        /* page faults (hard page faults) */
    long   ru_nswap;         /* swaps */
    long   ru_inblock;       /* block input operations */
    long   ru_oublock;       /* block output operations */
    long   ru_msgsnd;        /* IPC messages sent */
    long   ru_msgrcv;        /* IPC messages received */
    long   ru_nsignals;      /* signals received */
    long   ru_nvcsw;         /* voluntary context switches */
    long   ru_nivcsw;        /* involuntary context switches */
};
int getrusage(int who, struct rusage *res_usage);
// return 0 on success or -1 on error
```

参数 `who`：

- `RUSAGE_SELF`：返回调用进程相关的信息
- `RUSAGE_CHILDREN`：返回调用进程所有被终止和处于等待状态的子进程相关信息
- `RUSAGE_THREAD`：Linux 特有，返回线程相关信息



## 2. 进程资源限制

使用 shell 的内置命令 `ulimit` 可以设置 shell 的资源限制  

`getrlimit()` 和 `setrlimit()` 系统调用允许一个进程读取和修改自己的资源限制  

```c
#include <sys/resource.h>

struct rlimit {
    rlim_t rlim_cur;  // soft limit
    rlim_t rlim_max;  // hard limit
};

int getrlimit(int resource, struct rlimit *rlim);
int setrlimit(int resource, const struct rlimit *rlim);
// return 0 on success or -1 on error
```

`rlim_cur` 和 `rlim_max` 为 `RLIM_INFINITY` 表示没有限制  

参数 `resource`：

|      resource       |                    限制                     |
| :-----------------: | :-----------------------------------------: |
|     `RLIMIT_AS`     |       进程虚拟内存限制大小（字节数）        |
|    `RLIMIT_CORE`    |           core 文件大小（字节数）           |
|    `RLIMIT_CPU`     |              CPU 时间（秒数）               |
|    `RLIMIT_DATA`    |            进程数据段（字节数）             |
|   `RLIMIT_FSIZE`    |             文件大小（字节数）              |
|  `RLIMIT_MEMLOCK`   |            锁住的内存（字节数）             |
|  `RLIMIT_MSGQUEUE`  | 为真实用户 ID 分配的 POSIX 消息队列的字节数 |
|    `RLIMIT_NICE`    |                   nice 值                   |
|   `RLIMIT_NOFILE`   |          最大的文件描述符数量加 1           |
|   `RLIMIT_NPROC`    |          真实用户 ID 下的进程数量           |
|    `RLIMIT_RSS`     |       驻留集大小（字节数；没有实现）        |
|   `RLIMIT_RTPRIO`   |                实时调度策略                 |
|   `RLIMIT_RTTIME`   |            实时 CPU 时间（微秒）            |
| `RLIMIT_SIGPENDING` |       真实用户 ID 信号队列中的信号数        |
|   `RLIMIT_STACK`    |            栈段的大小（字节数）             |



# 三十七、守护进程

## 1. 概述

守护进程(daemon) 特征：

- 它的生命周期很长。通常，一个 daemon 会在系统启动的时候被创建并一直运行直至系统被关闭  
- 它在后台运行并且不拥有控制终端。  

daemon 是用来执行特殊任务的，比如 `cron` 在规定时间执行命令、`sshd` 允许远程主机使用 ssh 登录等等  



## 2. 创建守护进程

执行一个 `fork()`，之后父进程退出，子进程继续执行。

子进程调用 `setsid()` 开启一个新会话并释放它与之前控制终端之间的所有关联关系  

如果 daemon 后面可能会打开一个终端设备，那么必须要采取措施来确保这个设备不会成为控制终端，在所有可能应用到一个终端设备上的 `open()` 调用中指定 `O_NOCTTY` 标记或者在 `setsid()` 调用之后执行第二个 `fork()`，然后再次让父进程退出并让孙子进程继续执行。 这样就确保了子进程不会成为会话组长  

清除进程的 `umask` 以确保当 daemon 创建文件和目录时拥有所需的权限  

修改进程的当前工作目录，通常会改为根目录 `/`。  

关闭 daemon 从其父进程继承而来的所有打开着的文件描述符  

在关闭了文件描述符 0、 1 和 2 之后， daemon 通常会打开 `/dev/null` 并使用 `dup2()` 使所有这些描述符指向这个设备。  

实际可以使用 `becomeDaemon()` 函数将调用者变为 daemon  

```c
#include <syslog.h>

#define BD_NO_CHDIR 01          // Don't chdir("/")
#define BD_NO_CLOSE_FILES 02    // Don't close all open files
#define BD_NO_REOPEN_STD_FDS 04 // Don't reopen stdin, stdout and stderr to /dev/null
#define BD_NO_UMASKO 010        // Don't do a umask(0)
#define BD_MAX_CLOSE 8192       // Maximum file descriptors to close if 
                                // sysconf(_SC_OPEN_MAX) is indeterminate


int becomeDaemon(int flags);
// return 0 on success or -1 on error
```



## 3. 编写 daemon

很多标准的 daemon 是通过在系统关闭时执行特定于应用程序的脚本来停止的。而那些不以这种方式终止的
daemon 会收到一个 `SIGTERM` 信号。如果 daemon 在终止之前需要做些清理工作，那么就需要为这个信号建立一个处理函数。而且 `init` 进程在发完 `SIGTERM` 信号的 5 秒之后会发送一个 `SIGKILL` 信号  

由于 daemon 是长时间运行的，因此要特别小心潜在的内存泄露问题  



## 4. 重新初始化 daemon

daemon 需要持续运行，因此在设计 daemon 程序时需要克服一些障碍：

- 通常 daemon 会在启动时从相关的配置文件中读取操作参数，但有些时候需要在不重启 daemon 的情况下快速修改这些参数  
- 一些 daemon 会产生日志文件。如果 daemon 永远不关闭日志文件的话，那么日志文件就会无限制地增长，最终会阻塞文件系统  

解决这两个问题的方案是让 daemon 为 `SIGHUP` 建立一个处理器， 并在收到这个信号时采取所需的措施  



## 5. `syslog`

在编写 daemon 时碰到的一个问题是如何显示错误消息  

`syslog` 需要再进行讨论，暂跳过  



# 三十八、安全特权级程序

**1. Set ID**  

尽量避免使用 Set UID 和 Set GID，可以将需要权限才能完成的功能拆分到一个只执行单个任务的程序中，然后在需要的时候在子进程中执行这个程序。  

即使有时候需要 Set UID 或 Set GID 权限，对于一个 Set UID 程序来讲也并不总是需要赋给进程 root 身份  



**2. 以最小权限操作**

程序应该总是使用完成当前所执行的任务所需的最小权限来操作， saved 的 Set UID 工具就是为此而设计的  

按需拥有权限，在无需权限时永久删除权限  



**3. 小心执行程序**  

在执行另一个程序(`exec/system`)之前永久地删除权限  

避免执行一个拥有权限的 shell  

在 `exec()` 之前关闭所有用不到的文件描述符  



**4. 避免暴露敏感信息**  

当一个程序读取密码或其他敏感信息时应该在执行完所需的处理之后立即从内存中删除这些信息  

因为包含这些数据的虚拟内存页可能被换出，交换区域的数据可能被特权或者如果进程接收到了一个能导致它产生一个核心 dump 文件的信号，那么就有可能会从该文件中获取这类信息  



**5. 确定进程边界**  

考虑使用 capacity、考虑使用 `chroot`  



**6. 小心信号和竞争条件**  

当信号在程序执行过程中的任意时刻发送时需要考虑可能出现的竞争条件。在程序中合适的地方应该捕获、阻塞或忽略信号以防止可能存在的安全性问题。  



**7. 执行文件操作和文件 I/O**   

如果一个特权进程需要创建一个文件， 那么必须要小心处理那个文件的所有权和权限以确保文件不存在被恶意操作攻击的风险点  



**8. 不要完全相信输入和环境**  

**9. 小心缓冲区溢出**  

当输入值或复制的字符串超出分配的缓冲区空间时就需要小心缓冲区溢出了。永远不要使用 `gets()`，在使用诸如 `scanf()`、`sprintf()`、`strcpy()` 以及 `strcat()` 时需要谨慎  

对于其中的大多数函数来讲，如果到达了指定的最大值，那么源字符串的截断部分会被放到目标缓冲区中。由于这样的截断字符串对于程序来讲可能是毫无意义的，因此调用者必须要检查字符串是否发生了截断  



**10. 小心拒绝访问攻击**  

服务器应该执行负载控制，当负载超过预先设定的限制之后就丢弃请求  

服务器应该为与客户端的通信设置超时时间，这样如果客户端不响应（可能是故意的），那么服务器也不会永远地等待客户端。  

在发生超负荷时，服务器应该记录下合适的信息以便系统管理员得知这个问题  

服务器程序在碰到预期之外的负载时不应该崩溃。如应该严格进行边界检查以确保过多的请求不会造成数据结构溢出  

设计的数据结构应该能够避免算法复杂度攻击。  



**11. 检查返回状态和安全处理失败的情况**  

特权程序应该总是检查系统调用和库函数调用是否成功以及它们是否返回了预期的值  



# 三十九、capacity

跳过



# 四十、登录记账

跳过



# 四十一、动态库基础

动态库是一种将库函数打包成一个单元使之能够在运行时被多个进程共享的技术  

比较熟悉，该章挑重点写  



## 1. 概述

动态库的优点：  

- 由于整个程序的大小变得更小了， 因此在一些情况下， 程序可以完全被加载进内存中，从而能够更快地启动程序。这一点只有在大型共享库正在被其他程序使用的情况下才成立。第一个加载共享库的程序实际上在启动时会花费更长的时间，因为必须要先找到共享库并将其加载到内存中  
- 由于目标模块没有被复制进可执行文件中，而是在共享库中集中维护的，因此在修改目标模块时无需重新链接程序就能够看到变更  

新增开销：  

- 在概念上以及创建共享库和构建使用共享库的程序的实践上，共享库比静态库更复杂  
- 共享库在编译时必须要使用位置独立的代码，这在大多数架构上都会带来性能开销，因为它需要使用额外的一个寄存器  
- 在运行时必须要执行符号重定位。在符号重定位期间，需要将对共享库中每个符号（变量或函数）的引用修改成符号在虚拟内存中的实际运行时位置。由于存在这个重定位的过程，与静态链接程序相比， 一个使用共享库的程序或多或少需要花费一些时间来执行这个过程  



## 2. 位置独立的代码

`gcc -fPIC` 选项指定编译器应该生成位置独立的代码， 这会改变编译器生成执行特定操作的代码的方式，包括访问全局、静态和外部变量，访问字符串常量，以及获取函数的地址。这些变更使得代码可以在运行时被放置在任意一个虚拟地址处。这一点对于共享库来讲是必需的，因为在链接的时候是无法知道共享库代码位于内存的何处的。  

在 Linux/x86 上，可以使用不加 `–fPIC` 选项编译的模块来创建共享库。但这样做的话会丢失共享库的一些优点，因为包含依赖于位置的内存引用的程序文本页面不会在进程间共享  



## 3. 相关工具

```shell
ldd prog # 列出一个程序所需要的共享库

# 获取目标文件（或库、ELF 可执行程序）的各种信息
objdump xxx.so  
readelf xxx.so

# 获取目标库或可执行程序中定义的符号
nm xxx
strings xxx
```



## 4. 安装动态库

- 设置 `LD_LIBRARY_PATH` 环境变量
- 放入默认的目录 `/usr/lib, /lib, /usr/local/lib`
- 在 `/etc/ld.so.conf` 列出的目录

可以使用 `ldconfig` 可以查看或设置动态库的相关信息  





## 5. 版本兼容

随着时间的流逝，可能需要修改共享库的代码。这种修改会导致产生一个新版本的库，这个新版本可以与之前的版本兼容，也可能与之前的版本不兼容。如果是兼容的话则意味着只需要修改库的真实名称的次要版本标识符即可，如果是不兼容的话则意味着必须要定义一个库的新主要版本  

共享库的优点之一是当一个运行着的程序正在使用共享库的一个既有版本时也能够安装库的新主要版本或次要版本。在安装的过程中需要做的事情包括创建新的库版本、将其安装在恰当的目录中以及根据需要更新 `soname` 和链接器名称符号链接（通常使用 `ldconfig` 完成）  

创建 `/usr/lib/libdemo.so.1.0.1` 的动态库

```shell
gcc -c -fPIC xxx.c yyy.c zzz.c
gcc -shared -Wl,-soname,libdemo.so.1 -o libdemo.so.1.0.1 xxx.o yyy.o zzz.o

ln -s libdemo.so.1.0.1 libdemo.so.1
ln -s libdemo.so.1 libdemo.so
```





如果更新 `/usr/lib/libdemo.so.1.0.1` 为 `1.0.2`，需要完成下面的步骤  

```shell
gcc -c -fPIC xxx.c yyy.c zzz.c
gcc -shared -Wl,-soname,libdemo.so.1 -o libdemo.so.1.0.2 xxx.o yyy.o zzz.o

mv libdemo.so.1.0.2 /usr/local/lib
```

升级到 `2.0.0` 需要  

```shell
gcc -c -fPIC xxx.c yyy.c zzz.c
gcc -shared -Wl,-soname,libdemo.so.2 -o libdemo.so.2.0.0 xxx.o yyy.o zzz.o

mv libdemo.so.1.0.2 /usr/local/lib
```





## 6. 指定库搜索目录

除了把动态库放到系统库目录下，使用 `LD_LIBRARY_PATH` 之外，还可以在可执行文件中插入一个在运行时搜索共享库的目录列表，需要 `-rpath` 编译选项  

比如 `gcc -Wl,-rpath,/home/mtk/pdir -o prog prog.c libdemo.so` 会将字符串 `/home/mtk/pdir` 复制到可执行文件 `prog` 的运行时库路径(`rpath`) 列表中，因此当运行这个程序时，动态链接器在解析共享库引用时还会搜索这个目录  



## 7. 运行时找到动态库

在解析库依赖时，动态链接器首先会检查各个依赖字符串以确定它是否包含 `/`，因为在链接可执行文件时如果指定了一个显式的库路径名的话就会发生这种情况。如果找到了一个 `/`，那么依赖字符串就会被解释成一个路径名（绝对路径名或相对路径名），并且会使用该路径名加载库。否则动态链接器会使用下面的规则来搜索共享库。  

- 如果可执行文件的运行时库路径列表 `rpath` 中包含目录并且不包含 `DT_RUNPATH` 列表，那么就搜索这些目录  
- 如果定义了 `LD_LIBRARY_PATH` 环境变量，那么就会轮流搜索该变量值中以冒号分隔的各个目录。如果可执行文件是一个 Set `UID` 或 Set `GID` 程序，那么就会忽略 `LD_LIBRARY_PATH` 变量  
- 如果可执行文件 `DT_RUNPATH` 运行时库路径列表中包含目录，那么就会搜索这些目录  
- 检查 `/etc/ld.so.cache` 文件以确认它是否包含了与库相关的条目  
- 搜索 `/lib` 和 `/usr/lib` 目录  



## 8. 运行时符号解析

假设现在有一个主程序和一个共享库，它们两个都定义了一个全局函数 `xyz()`，并且共享库中的另一个函数调用了 `xyz()`  

```c
// prog.c
void xyz() {
    printf("main-xyz\n");
}

int main() {
    func();
}

// libfoo.so
void xyz() {
    printf("foo-xyz\n");
}

void func() {
    xyz();
}
```

输出为 `main-xyz`，主程序中的 `xyz()` 定义覆盖（隐藏）了共享库中的定义  

符号解析规则：

- 主程序中全局符号的定义覆盖库中相应的定义  
- 如果一个全局符号在多个库中进行了定义，那么对该符号的引用会被绑定到在扫描库时找到的第一个定义，其中扫描顺序是按照这些库在静态链接命令行中列出时从左至右的顺序  

如果想要确保在共享库中对 `xyz()` 的调用确实调用了库中定义的相应函数，那么在构建共享库的时候就需要使用 `–Bsymbolic` 链接器选项  



# 四十二、动态库高级特性

## 1. 加载动态库

当一个可执行文件开始运行之后，动态链接器会加载程序的动态依赖列表中的所有动态库，但有些时候延迟加载库是比较有用的，如只在需要的时候再加载一个插件。动态链接器的这项功能是通过一组 API 来实现的。这组 API 通常被称为 `dlopen` API  

`dlopen` API 使得程序能够在运行时打开一个共享库，根据名字在库中搜索一个函数，然后调用这个函数。在运行时采用这种方式加载的共享库通常被称为动态加载的库，它的创建方式与其他共享库的创建方式完全一样  

```c
#include <dlfcn.h>

void *dlopen(const char *libfilename, int flags);
// return library handle on success or NULL on error
```

如果 `libfilename` 指定的共享库依赖于其他共享库，那么 `dlopen()` 会自动加载那些库。  

同一个库文件中可以多次调用 `dlopen()`，但将库加载进内存的操作只会发生一次， 所有的调用都返回同样的句柄值。使用引用计数来维护库资源   

`flags`：

- `RTLD_LAZY`：只有当代码被执行的时候才解析库中未定义的函数符号。延迟解析只适用于函数引用，对变量的引用会被立即解析。  
- `RTLD_NOW`：在 `dlopen()` 结束之前立即加载库中所有的未定义符号，不管是否需要用到这些符号，这种做法的结果是打开库变得更慢了，但能够立即检测到任何潜在的未定义函数符号错误，而不是在后面某个时刻才检测到这种错误。  
- `RTLD_GLOBAL`：这个库及其依赖树中的符号在解析由这个进程加载的其他库中的引用和通过 `dlsym()` 查找时可用
- `RTLD_LOCAL`，大多数实现该项为默认值，它规定在解析后续加载的库中的引用时这个库及其依赖树中的符号不可用  
- `RTLD_NODELETE`：在 `dlclose()` 调用中不要卸载库，即使其引用计数已经变成 0 了。这意味着在后面重新通过 `dlopen()` 加载库时不会重新初始化库中的静态变量  
- `RTLD_NOLOAD`：不加载库。可以使用这个标记来检查某个特定的库是否已经被加载到了进程的地址空间中。如果已经加载了，那么 `dlopen()` 会返回库的句柄，如果没有加载，那么 `dlopen()` 会返回 `NULL`。第二，可以使用这个标记来更新已加载的库的标记  
- `RTLD_DEEPBIND`：在解析这个库中的符号引用时先搜索库中的定义，然后再搜索已加载的库中的定义。这个标记使得一个库能够实现自包含，即优先使用自己的符号定义，而不是在已加载的其他库中定义的同名全局符号(类似于 `-Bsymbolic` )



```c
#include <dlfcn.h>

const char *dlerror(void);
// return pointer to error-diagnostic or NULL if 
// no error has occurred since previous call to dlerror
```

```c
#include <dlfcn.h>

void *dlsym(void *handle, char *symbol);
// return address of symbol or NULL if symbol is not found
```

handle 参数还可以使用以下的参数：

- `RTLD_DEFAULT`：从主程序中开始查找 symbol，接着按序在所有已加载的共享库中查找，包括那些通过使用了 `RTLD_GLOBAL` 标记的 `dlopen()` 调用动态加载的库，这个标记对应于动态链接器所采用的默认搜索模型  
- `RTLD_NEXT`：在调用 `dlsym()` 之后加载的共享库中搜索 symbol，这个标记适用于需要创建与在其他地方定义的函数同名的包装函数的情况  



```c
#include <dlfcn.h>

int dlclose(void *handle);
// return 0 on success or -1 on error
```

`dlclose()` 函数会减小 handle 所引用的库的打开引用的系统计数。如果这个引用计数变成了 0 并且其他库已经不需要用到该库中的符号了，那么就会卸载这个库  



```c
#include <dlfcn.h>

typedef struct {
    const char *dli_fname;  /* Pathname of shared object that
                               contains address */
    void       *dli_fbase;  /* Base address at which shared
                               object is loaded */
    const char *dli_sname;  /* Name of symbol whose definition
                               overlaps addr */
    void       *dli_saddr;  /* Exact address of symbol named
                               in dli_sname */
} Dl_info;

int dladdr(const void *addr, Dl_info *info);
// return nonzero value if addr was found in a shared library otherwise 0
```



## 2. 控制符号可见性

设计良好的共享库应该只公开那些构成其声明的 API  

下面的方法可以控制符号的 export：

- `static` 声明的符号
- GNU C 编译器提供的 `__attribute__ ((visibility("hidden")))` 属性声明
- version-script 精确控制符号的可见性以及选择将一个引用绑定到符号的哪个版本  
- 当动态加载一个共享库时， `dlopen()` 的 `RTLD_GLOBAL` 标记可以用来指定这个库中定义的符号应该用于后续加载的库中的绑定操作， `––export–dynamic` 链接器选项可以用来使主程序的全局符号对动态加载的库可用。  



## 3. version-script

version-script 是一个包含链接器 `ld` 执行的指令的文本文件。要使用必须要指定 `––version–script ` 链接器选项  

```sh
gcc -Wl,--version-script,myscriptfile.map
```

版本脚本的一个用途是控制那些可能会在无意中变成全局可见（即对与该库进行链接的应用程序可见）的符号的可见性  

比如有三个编译模块 `vis_comm.c, vis_f1.c, vis_f2.c` 构建成一个动态库，这三个源文件分别定义了函数 `vis_comm()`、 `vis_f1()` 以及 `vis_f2()`。`vis_comm()` 函数由 `vis_f1()` 和 `vis_f2()` 调用，但不想被与该库进行链接的应用程序直接使用。  



可以通过以下的方法来实现上面的需求：

```map
# vis.map
VER_1 {
	global:
		vis_f1;
		vis_f2;
	local:
		*;
};
```

```sh
gcc -shared -o vis.so xxx.o -Wl,--version-script,vis.map
readelf --syms --use-dynamic vis.so | grep vis
```



符号版本化允许一个共享库提供同一个函数的多个版本  

比如下面的例子  

```map
# sv_v1.map
VER_1 {
	global: xyz;
	local: *;
};
```

接着创建一个程序 `p1` 来使用这个库  

现在假设需要修改库中 `xyz()` 的定义， 但同时仍然需要确保程序 pl继续使用老版本的函数。为完成这个任务，必须要在库中定义两个版本的 `xyz()`。  

```c
// lib_v2.c
__asm__(".symver xyz_old, xyz@VER_1");
__asm__(".symver xyz_new, xyz@@VER_2");

// old xyz
void xyz_old() {
    // ...
}

void xyz_new() {
    // ...
}

void foo() {
    // ...
}
```

第二个 `.symver` 指令使用 `@@`（不是@）来指示当应用程序与这个共享库进行静态链接时应该使用的 `xyz()` 的默认定义。一个符号的 `.symver` 指令中应该只有一个指令使用 `@@` 标记。  

```map
# v2.map
VER_1 {
	global: xyz;
	local: *;
};

VER_2 {
	global foo;
} VER_1;
```

新版本标签 `VER_2`，它依赖于标签 `VER_1`  

Linux 上的版本标签依赖的唯一效果是版本节点可以从它所依赖的版本节点中继承 global 和 local 规范  



## 4. 初始化和终止函数

在动态库加载和卸载时自动执行的初始化和终止函数  

```c
void __attribute__ ((constructor)) some_load(void) {
    // ...
}

void __attribute__ ((destructor)) some_unload(void) {
    // ...
}
```



## 5. 预加载动态库

出于测试的目的，有些时候可以有选择地（因为查找动态库的顺序）覆盖一些正常情况下会被动态链接器找出的函数（或其他符号） 。要完成这个任务可以定义一个环境变量 `LD_PRELOAD`，其值由在加载其他共享库之前需加载的共享库名称构成  

由于首先会加载这些共享库，因此可执行文件自动会使用这些库中定义的函数，从而覆盖那些动态链接器在其他情况下会搜索的同名函数  

出于安全原因，Set ID 的程序忽略了 `LD_PRELOAD`  



## 6. `LD_DEBUG`

有些时候需要监控动态链接器的操作以弄清楚它在搜索哪些库，这可以通过 `LD_DEBUG` 环境变量来完成。通过将这个变量设置为一个（或多个）标准关键词可以从动态链接器中得到各种跟踪信息  

具体的值如下  

```txt
libs        display library search paths
reloc       display relocation processing
files       display progress for input file
symbols     display symbol table processing
bindings    display information about symbol binding
versions    display version dependencies
scopes      display scope information
all         all previous options combined
statistics  display relocation statistics
unused      determined unused DSOs
help        display this help message and exit
```



# 四十三、进程间通信

对 IPC 比较熟悉，该章只进行总结  

## 1. 分类

![IPC classify](../../imgs/linux/TLPI/43_1.png)  



# 四十四、pipe 和 FIFO

对 pipe 比较熟悉，FIFO 用的不太多，同样该章只进行总结  

## 1. 概述

pipe 是单向的，可以确保写入不超过 `PIPE_BUF` 字节(Linux 4096)的操作是原子的，pipe 的存储能力是有限的  



## 2. 使用

```c
#include <unistd.h>

int pipe(int fds[2]);
// return 0 on success or -1 on error
```

`fds[1]` 写，`fds[0]` 读，一般用于父子进程  



管道的一个常见用途是执行 shell 命令并读取其输出或向其发送一些输入。 `popen()` 和 `pclose()` 函数简化了这个任务。`popen()` 函数创建了一个管道，然后创建了一个子进程来执行 shell，而 shell 又创建了一个
子进程来执行 command 字符串  

```c
#include <stdio.h>

FILE *popen(const char *command, const char *mode);
// return file stream, or NULL on error

int pclose(FILE *stream);
// return termination status of child process or -1 on error
```

`mode` 参数是 `"r"` 或 `"w"`，如果是 `r`，命令的标准输出将通过管道传入调用进程，如果是 `w`，则调用进程通过管道写入到命令的标准输入  

由于 `popen()` 调用返回的文件流指针没有引用一个终端， 因此 stdio 库会对这种文件流应用块缓冲。这意味着当将 mode 的值设置为 w 来调用 `popen()` 时，在默认情况下只有当 stdio 缓冲器被充满或使用 `pclose()` 关闭了管道之后输出才会被发送到管道另一端的子进程。  

如果需要确保子进程能够立即从管道中接收数据，那么就需要定期调用 `fflush()` 或使用 `setbuf(fp, NULL)` 调用禁用 stdio 缓冲  



FIFO 暂时跳过



# 四十五、System V IPC

|     接 口     |     消 息 队 列      |         信 号 量          |     共 享 内 存      |
| :-----------: | :------------------: | :-----------------------: | :------------------: |
|    头文件     |    `<sys/msg.h>`     |       `<sys/sem.h>`       |    `<sys/shm.h>`     |
| 关联数据结构  |      `msqid_ds`      |        `semid_ds`         |      `shmid_ds`      |
| 创建/打开对象 |      `msgget()`      |        `semget()`         | `shmget() + shmat()` |
|   关闭对象    |        （无）        |          （无）           |      `shmdt()`       |
|   控制操作    |      `msgctl()`      |        `semctl()`         |      `shmctl()`      |
|   执行 IPC    | `msgsnd(), msgrcv()` | `semop()` 测试/调整信号量 | 访问共享区域中的内存 |



# 四十六、System V 消息队列

不太常用，简单看看，用到再详细了解

消息队列允许进程以消息的形式交换数据。尽管消息队列在某些方面与管道和 FIFO 类似，但它们之间仍然存在显著的差别  

- 用来引用消息队列的句柄是一个由 `msgget()` 调用返回的标识符。这些标识符与 UNIX 系统上大多数其他形式的 I/O 所使用的文件描述符是不同的  

- 通过消息队列进行的通信是面向消息的，即读者接收到由写者写入的整条消息。读取一条消息的一部分而让剩余部分遗留在队列中或一次读取多条消息都是不可能的。这一点与管道不通，管道提供的是一个无法进行区分的字节流（即使用管道时读者一次可以读取任意数量的字节数，不管写者写入的数据块的大小是什么）  

- 除了包含数据之外，每条消息还有一个用整数表示的类型。从消息队列中读取消息既可以按照先入先出的顺序，也可以根据类型来读取消息  



# 四十七、System V 信号量

有可替代 API，简单看看，用到再详细了解



# 四十八、System V 共享内存

共享内存允许两个或多个进程共享物理内存的同一块区域（通常被称为段）。由于一个共享内存段会成为一个进程用户空间内存的一部分，因此这种 IPC 机制无需内核介入。所有需要做的就是让一个进程将数据复制进共享内存中，并且这部分数据会对其他所有共享同一个段的进程可用。与管道或消息队列要求发送进程将数据从用户空间的缓冲区复制进内核内存和接收进程将数据从内核内存复制进用户空间的缓冲区的做法相比，这种 IPC 技术的速度更快  

共享内存这种 IPC 机制不由内核控制意味着通常需要通过某些同步方法使得进程不会出现同时访问共享内存的情况  



# 四十九、内存映射

适用于需要高性能的 IPC 需求

## 1. 概述

`mmap()` 系统调用在调用进程的虚拟地址空间中创建一个新内存映射。映射分为两种。

- 文件映射：文件映射将一个文件的一部分直接映射到调用进程的虚拟内存中。一旦一个文件被映射之后就可以通过在相应的内存区域中操作字节来访问文件内容了。映射的分页会在需要的时候从文件中（自动）加载。这种映射也被称为基于文件的映射或内存映射文件。
- 匿名映射：一个匿名映射没有对应的文件。相反，这种映射的分页会被初始化为 0  

一个进程的映射中的内存可以与其他进程中的映射共享（各个进程的页表条目指向RAM 中相同分页）  

- 当两个进程映射了一个文件的同一个区域时它们会共享物理内存的相同分页。  
- 通过 fork()创建的子进程会继承其父进程的映射的副本，并且这些映射所引用的物理内存分页与父进程中相应映射所引用的分页相同  

映射分类：

- 私有映射（`MAP_PRIVATE`）：在映射内容上发生的变更对其他进程不可见，对于文件映射来讲，变更将不会在底层文件上进行。尽管一个私有映射的分页在上面介绍的情况中初始时是共享的，但对映射内容所做出的变更对各个进程来讲则是私有的。使用 COW 进行优化  
- 共享映射（`MAP_SHARED`）：在映射内容上发生的变更对所有共享同一个映射的其他进程都可见，对于文件映射来讲，变更将会发生在底层的文件上  



各种内存映射的作用

- 私有文件映射：根据文件内容初始化内存  

- 私有匿名映射：内存分配  
- 共享文件映射：内存映射 I/O；进程间共享内存（IPC）
- 共享匿名映射：进程间共享内存（IPC）

一个进程在执行 `exec()` 时映射会丢失，但通过 `fork()` 创建的子进程会继承映射  



## 2. `mmap`

`mmap()` 系统调用在调用进程的虚拟地址空间中创建一个新映射。  

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
// return starting address of mapping on success or MAP_FILED on error
```

`addr` 参数指定了映射被放置的虚拟地址。如果将 `addr` 指定为 NULL，那么内核会为映射选择一个合适的地址。这是创建映射的首选做法。或者在 `addr` 中指定一个非 NULL 值时，内核会在选择将映射放置在何处时将这个参数值作为一个提示信息来处理。  

`length` 参数指定了映射的字节数。尽管 length 无需是一个系统分页大小的倍数，但内核会以分页大小为单位来创建映射  

`prot` 参数是一个位掩码，指定了映射的保护信息

- `PROT_NONE` 区域无法访问
- `PROT_READ` 区域内容可读取
- `PROT_WRITE` 区域内容可修改
- `PROT_EXEC` 区域内容可执行

`flags` 参数：

- `MAP_PRIVATE` 
- `MAP_SHARED`
- `MAP_FIXED`：并且 `addr` 为非零值，那么 `addr` 和 offset 除以系统分页大小所得的余数应该相等  
- `MAP_ANONYMOUS`：匿名映射，即没有底层文件对应的映射  
- `MAP_LOCKED`：按照 `mlock()` 的方式预加载映射分页并将映射分页锁进内存  
- `MAP_HUGETLB`：创建一个使用巨大页的映射
- `MAP_NORESERVE`：控制交换空间的预留
- `MAP_POPULATE`：填充一个映射的分页。对于文件映射来讲，这将会在文件上执行一个超前读取。这意味着后续对映射内容的访问不会因分页故障而发生阻塞  
- `MAP_UNINITIALIZED`：指定这个标记会防止一个匿名映射被清零。它能够带来性能上的提升，但同时也带来了安全风险，因为已分配的分页中可能会包含上一个进程留下来的敏感信息。因此这个标记一般只供嵌入式系统使用，因为在这种系统中性能是一个至关重要的因素，并且整个系统都处于嵌入式应用程序的控制之下  

参数 `fd` 和 `offset` 是用于文件映射的（匿名映射将忽略它们）。 `fd` 参数是一个标识被映射的文件的文件描述符。 `offset` 参数指定了映射在文件中的起点，它必须是系统分页大小的倍数。要映射整个文件就需要将 `offset` 指定为 0 并且将 `length` 指定为文件大小。`offset` 参数必须要与分页对齐  



## 3. `munmap`

```c
#include <sys/mman.h>

int munmap(void *addr, size_t length);
// return 0 on success or -1 on error
```

`addr` 参数是待解除映射的地址范围的起始地址，它必须与一个分页边界对齐。`length` 参数是一个非负整数，它指定了待解除映射区域的大小（字节数）  

当一个进程终止或执行了一个 `exec()` 之后进程中所有的映射会自动被解除  





## 4. 文件映射

`mmap()` 将打开的文件的内容映射到调用进程的地址空间中，被调用之后就能够关闭文件描述符了，而不会对映射产生任何影响  

**私有文件映射**  

私有文件映射最常见的两个用途：

- 允许多个执行同一个程序或使用同一个共享库的进程共享同样的（只读的）文本段，它是从底层可执行文件或库文件的相应部分映射而来的  
- 映射一个可执行文件或共享库的初始化数据段。这种映射会被处理成私有使得对映射数据段内容的变更不会发生在底层文件上  



**共享文件映射**  

当多个进程创建了同一个文件区域的共享映射时，它们会共享同样的内存物理分页。对映射内容的变更将会反应到文件上。实际上，这个文件被当成了该块内存区域的分页存储  

共享文件映射可以**内存映射 I/O** 和 IPC



**内存映射 I/O**  

由于共享文件映射中的内容是从文件初始化而来的，并且对映射内容所做出的变更都会自动反应到文件上，因此可以简单地通过访问内存中的字节来执行文件 I/O，而依靠内核来确保对内存的变更会被传递到映射文件中，比起传统的 `read` 和 `write` 有更好的性能（在大型文件中执行重复随机访问）  



当访问文件未被映射的的区域会导致 `SIGSEVG` 信号，默认动作是 core dump  



## 5. 同步映射区域

内核会自动将发生在 `MAP_SHARED` 映射内容上的变更写入到底层文件中，但在默认情况下，内核不保证这种同步（写回）操作会在何时发生。   

`msync()` 系统调用让应用程序能够显式地控制何时完成共享映射与映射文件之间的同步。  

```c
#include <sys/mman.h>

int msyc(void *addr, size_t length, int flags);
// return 0 on success or -1 on error
```

`flags` ：

- `MS_SYNC`：执行一个同步的文件写入。这个调用会阻塞直到内存区域中所有被修改过的分页被写入到文件为止。  
- `MS_ASYNC`：执行一个异步的文件写入。内存区域中被修改过的分页会在后面某个时刻被写入磁盘并立即对在相应文件区域中执行 `read()` 的其他进程可见
- `MS_INVALIDATE`：使映射数据的缓存副本失效。当内存区域中所有被修改过的分页被同步到文件中之后，内存区域中所有与底层文件不一致的分页会被标记为无效。当下次引用这些分页时会从文件的相应位置处复制相应的分页内容，其结果是其他进程对文件做出的所有更新将会在内存区  





## 6. 匿名映射

创建匿名映射的方法：

- 在 `flags` 中指定 `MAP_ANONYMOUS` 并将 `fd` 指定为 -1。Linux 上会忽略 `fd`  
- 打开 `/dev/zero` 设备文件并将得到的文件描述符传递给 `mmap()`。  



## 7. 重新映射

Linux 提供了 `mremap()` 系统调用改变映射的位置和大小  

```c
#include <sys/mman.h>

void *mremap(void *oddaddress, size_t oldsize, size_t newsize, int flags, ...);
// return starting address of remapping region on success or MAP_FILED on error
```

在执行重映射的过程中内核可能会为映射在进程的虚拟地址空间中重新指定一个位置，而是否允许这种行为则是由 `flags` 参数来控制的。它是一个位掩码，其值要么是 0，要么包含下列几个值  

- `MREMAP_MAYMOVE`：如果指定了这个标记，那么根据空间要求的指令，内核可能会为映射在进程的虚拟地址空间中重新指定一个位置。如果没有指定这个标记，并且在当前位置处没有足够的空间来扩展这个映射，那么就返回 `ENOMEM` 错误  
- `MREMAP_FIXED`：这个标记只能与 `MREMAP_MAYMOVE` 一起使用。它在 `mremap()` 中所起的作用与  `MAP_FIXED` 在 `mmap()` 中所起的作用类似  



## 8. `MAP_NORESERVE`

一些应用程序会创建大（通常是私有匿名的）映射，但只使用映射区域中的一小部分  

如果内核总是为此类映射分配（或预留）足够的交换空间，那么很多交换空间可能会被浪费。相反，内核可以只在需要用到映射分页的时候（即当应用程序访问分页时）为它们预留交换空间。这种方法被称为 lazy swap reservation  

lazy swap reservation 允许交换空间被过度利用。但是如果尝试访问整个映射，那么 RAM 和 swap 空间就会被耗尽，之后会执行 OOM  

Linux 特有的 `/proc/sys/vm/overcommit_memory` 文件包含了一个整数值，它控制着内核对交换空间过度利用的处理  

| `overcommit_memory` | 指定 `MAP_NORESERVE` | 没有指定 `MAP_NORESERVE` |
| :-----------------: | :------------------: | :----------------------: |
|          0          |     允许过度利用     |    拒绝明显的过度利用    |
|          1          |     允许过度利用     |       允许过度利用       |
|          2          |    严格的过度利用    |      严格的过度利用      |

过度利用监控只适用于私有可写映射和共享匿名映射  



## 9. 非线性映射

非线性映射——文件分页的顺序与它们在连续内存中出现的顺序不同的映射  

```c
#include <sys/mman.h>

int remap_file_pages(void *addr, size_t size, int prot, size_t pgoff, int flags);
// reutrn 0 on success or -1 on error
```





# 五十、虚拟内存操作

## 1. 改变内存保护

```c
#include <sys/mman.h>

int mprotect(void *addr, size_t length, int prot);
// return 0 on success or -1 on error
```

`prot`：`PROT_NONE/PROT_READ/PROT_WRITE/PROT_EXEC`  

如果一个进程在访问一块内存区域时违背了内存保护，那么内核就会向该进程发送一个`SIGSEGV` 信号  



## 2. 内存锁

在一些应用程序中将一个进程的虚拟内存的部分或全部锁进内存以确保它们总是位于物理内存中是非常有用的。之所以需要这样做的一个原因是它可以提高性能。对被锁住的分页的访问可以确保永远不会因为分页故障而发生延迟。  

给内存加锁的另一个原因是安全。如果一个包含敏感数据的虚拟内存分页永远不会被交换出去，那么该分页的副本就不会被写入到磁盘  

`RLIMIT_MEMLOCK` 为一个进程能够锁进内存的字节数设定了一个上限。Linux 2.6.9 开始，内存加锁模型发生了变化，允许非特权进程给一小段内存进行加锁  

- 特权进程能够锁住的内存数量是没有限制的  
- 非特权进程能够锁住的内存数量上限由软限制 `RLIMIT_MEMLOCK` 定义，软和硬 `RLIMIT_MEMLOCK` 限制的默认值都是 8 个分页  

`mlock()/mlockall()`，`mmap` 的 `MAP_LOCKED` 标记可以在映射被创建时将内存映射锁住，`shmctl` 的 `SHM_LOCK` 操作，用来给 System V 共享内存段加锁  

对于 `mlock()`、`mlockall()` 以及 `mmap() MAP_LOCKED` 操作来讲， `RLIMIT_ MEMLOCK` 定义了一个进程级别的限制， 它限制了一个进程的虚拟地址空间中能够被锁进内存的字节数  

对于 `shmctl() SHM_LOCK` 操作来讲，`RLIMIT_MEMLOCK` 定义了一个用户级别的限制，它限制了这个进程的真实用户 ID 在共享内存段中能够锁住的字节数  

```c
#include <sys/mman.h>

int mlock(void *addr, size_t length);
int munlock(void *addr, size_t length);
// return 0 on success or -1 on error
```

`mlock()` 系统调用会锁住调用进程的虚拟地址空间中起始地址为 `addr` 长度为 `length` 字节的区域中的所有分页。内核会从 `addr` 下面的下一个分页边界开始锁住分页，为了移植性需要是分页大小的整数倍  

内存锁不会被通过 `fork()` 创建的子进程继承，也不会在 `exec()` 执行期间被保留  



**给一个进程占据的所有内存加锁和解锁**  

```c
#include <sys/mman.h>

int mlockall(int flags);
int munlockall(void);
// return 0 on success or -1 on error
```

`flags`：  

- `MCL_CURRENT`：将调用进程的虚拟地址空间中当前所有映射的分页锁进内存，包括当前为程序文本段、数据段、内存映射以及栈分配的所有分页  
- `MCL_FUTURE`：将后续映射进调用进程的虚拟地址空间的所有分页锁进内存。  



## 3. 内存驻留

`mincore()` 系统调用是内存加锁系统调用的补充，它报告在一个虚拟地址范围中哪些分页当前驻留在 RAM 中，因此在访问这些分页时也不会导致分页故障。  

```c
#include <sys/mman.h>

int mincore(void *addr, size_t length, unsigned char *vec);
// return 0 on success or -1 on error
```

`mincore()` 系统调用返回起始地址为 `addr` 长度为 `length` 字节的虚拟地址范围中分页的内存驻留信息。   

内存驻留相关的信息会通过 `vec` 返回，它是一个数组，其大小为`(length + PAGE_SIZE – 1) / PAGE_SIZE`  

  

## 4. 建议后续使用的内存模式

`madvise()` 系统调用通过通知内核调用进程对起始地址为 `addr` 长度为 `length` 字节的范围之内分页的可能的使用情况来提升应用程序的性能  

```c
#include <sys/mman.h>

int madvise(void *addr, size_t length, int advice);
// return 0 on success or -1 on error
```

`advice`：

- `MADV_NORMAL`：默认行为。分页是以簇的形式（较小的一个系统分页大小的整数倍）传输的。这个值会导致一些预先读和事后读  
- `MADV_RANDOM`：这个区域中的分页会被随机访问，这样预先读将不会带来任何好处，因此内核在每次读取时所取出的数据量应该尽可能少  
- `MADV_SEQUENTIAL`：在这个范围中的分页只会被访问一次，并且是顺序访问，因此内核可以激进地预先读，并且分页在被访问之后就可以将其释放了  
- `MADV_WILLNEED`：预先读取这个区域中的分页以备将来的访问之需  
- `MADV_DONTNEED`：调用进程不再要求这个区域中的分页驻留在内存中  



# 五十一、POSIX IPC

|    接口     |       消息队列       |              信号量              |      共享内存       |
| :---------: | :------------------: | :------------------------------: | :-----------------: |
|   头文件    |     `<mqueue.h>`     |         `<semaphore.h>`          |   `<sys/mman.h>`    |
| 对象 handle |       `mqd_t`        |            `sem_t *`             |      `int fd`       |
|  创建/打开  |      `mq_open`       |            `sem_open`            | `shm_open` + `mmap` |
|    关闭     |      `mq_close`      |           `sem_close`            |      `munmap`       |
|  断开链接   |     `mq_unlink`      |           `sem_unlink`           |    `shm_unlink`     |
|  执行 IPC   | `mq_send/mq_receive` | `sem_post/sem_wait/sem_getvalue` |                     |
|  其他操作   |     `mq_setattr`     |            `sem_init`            |                     |
|             |     `mq_getattr`     |          `sem_destroy`           |                     |
|             |     `mq_notify`      |                                  |                     |



与 System V IPC 对比：

- POSIX IPC 的接口比 System V IPC 接口简单  
- POSIX IPC 模型——使用名字替代键、使用 open、 close 以及 unlink 函数——与传统的 UNIX 文件模型更加一致  
- POSIX IPC 对象是引用计数的。简化了对象删除  
- POSIX IPC 的移植性不如 System V IPC



# 五十二、POSIX 消息队列

POSIX 消息队列，它允许进程之间以消息的形式交换数据。 POSIX 消息队列与 System V 消息队列的相似之处在于数据的交换单位是整个消息  

其他用到再来补坑  



# 五十三、POSIX 信号量

POSIX 信号量，它允许进程和线程同步对共享资源的访问  

其他用到再来补坑  



# 五十四、POSIX 共享内存

POSIX 共享内存能够让无关进程共享一个映射区域而无需创建一个相应的映射文件  

Linux 使用挂载于 `/dev/shm` 目录下的专用 `tmpfs` 文件系统  

要使用 POSIX 共享内存对象需要完成下列任务  

- 使用 `shm_open()` 函数打开一个与指定的名字对应的对象，返回一个引用该对象的文件描述符  
- 将上一步中获得的文件描述符传入 `mmap()` 调用并在其 `flags` 参数中指定 `MAP_SHARED`  

其他用到再来补坑  



# 五十五、文件加锁

## 1. 概述

应用程序的一个常见需求是从一个文件中读取一些数据，修改这些数据，然后将这些数据写回文件。只要在一个时刻只有一个进程以这种方式使用文件就不会存在问题，但当多个进程同时更新一个文件时就会出问题  

`flock()` 对整个文件加锁，`fcntl()` 对一个文件区域加锁  

尽管文件加锁通常会与文件 I/O 一起使用， 但也可以将其作为一项更通用的同步技术来使用。 协作进程可以约定一个进程对整个文件或一个文件区域进行加锁表示对一些共享资源（如一个共享内存区域）而非文件本身的访问  

由于 stdio 库会在用户空间进行缓冲，因此在混合使用 stdio 函数与加锁技术时需要特别小心。这里的问题是一个输入缓冲器在被加锁之前可能会被填满或者一个输出缓冲器在锁被删除之后可能会被刷新。需要使用下面的策略：   

- 使用 `read()` 和 `write()` 取代 stdio 来执行文件 I/O
- 对文件加锁之后立即刷新 stdio 流，并在释放锁之前再次刷新
- 使用 `setbuff()` 来禁用 stdio 缓冲，会牺牲掉一些效率

锁分成**劝告式**和**强制式**两种。劝告式表示一个进程可以简单地忽略另一个进程在文件上放置的锁，劝告式加锁模型能够正常工作，所有访问文件的进程都必须要配合，即在执行文件 I/O 之前首先需要在文件上放置一把锁，比如一组进程约定好一个规则，在执行 I/O 之前先判断是否符合规则，但是也可以不按照约定规则进行 I/O；强制式加锁系统会强制一个进程在执行 I/O 时需要遵从其他进程持有的锁  



## 2. `flock`

```c
#include <sys/file.h>

int flock(int fd, int operation);
// return 0 on success or -1 on error
```

`flock()` 系统调用在整个文件上放置一个锁  



| `operation` |             描述             |
| :---------: | :--------------------------: |
|  `LOCK_SH`  | 在 `fd` 引用的文件上加共享锁 |
|  `LOCK_EX`  | 在 `fd` 引用的文件上加互斥锁 |
|  `LOCK_UN`  |             解锁             |
|  `LOCK_NB`  |    发起一个非阻塞的锁请求    |

可以将一个既有共享锁转换成一个互斥锁（反之亦然）。将一个共享锁转换成一个互斥锁，在另一个进程持有了文件上的共享锁时会阻塞，除非同时指定了 `LOCK_NB` 标记  

锁转换的过程不一定是原子的。在转换过程中首先会删除既有的锁，然后创建一个新锁。  

当一个文件描述符被复制时（`dup*`），新文件描述符会引用同一个文件锁。  

当使用 `fork()` 创建一个子进程时，这个子进程会复制其父进程的文件描述符，并且与使用 `dup()` 调用之类的函数复制的描述符一样，这些描述符会引用同一个打开的文件描述，进而会引用同一个锁。  

有时候可以利用这些语义来将一个文件锁从父进程（原子地）传输到子进程：在 `fork()` 之后，父进程关闭其文件描述符，然后锁就只在子进程的控制之下了  



`flock` 的限制：

- 只能对整个文件加锁  
- 通过 `flock` 只能放置劝告式锁  
- 很多 NFS 实现不识别 `flock()` 放置的锁  



## 3. `fcntl()` 加锁

使用 `fcntl()` 能够在一个文件的任意部分上放置一把锁，这个文件部分既可以是一个字节，也可以是整个文件  

```c
#include <unistd.h>
#include <fcntl.h>

int fcntl(int fd, int cmd, ... /* arg */ );

struct flock {
 	short l_type;   // F_RDLCK, F_WRLCK, F_UNLCK
    short l_whence; // SEEK_SET, SEEK_CUR, SEEK_END
    
    off_t l_start;  // offset where the lock begins
    off_t l_len;    // number of bytes to lock; 0 means util EOF
    pid_t l_pid;
};

// fcntl locks file
int fcntl(int fd, int cmd, struct flock *flockstr);
```

`cmd` 参数：

- `F_SETLK`：获取（ `l_type` 是 `F_RDLCK` 或 `F_WRLCK`）或释放（`l_type` 是 `F_UNLCK`）由 `flockstr` 指定的字节上的锁，如果另一个进程持有了一把待加锁的区域中任意部分上的不兼容的锁时， fcntl()
  就会失败并返回 `EAGAIN` 或 `EACCESS` 错误（实现定义）
- `F_SETLKW`：与 `F_SETLK` 是一样的，除了在有另一个进程持有一把待加锁的区域中任意部分上的不兼容的锁时，调用就会阻塞直到锁的请求得到满足。如果正在处理一个信号并且没有指定 `SA_RESTART`，那么 `F_SETLKW` 操作就可能会被中断（即失败并返回 `EINTR` 错误）。开发人员可以利用这种行为来使用`alarm()` 或 `setitimer()` 为一个加锁请求设置一个超时时间  
- `F_GETLK`：检测是否能够获取 `flockstr` 指定的区域上的锁，但实际不获取这把锁。`l_type` 字段的值必须为 `F_RDLCK` 或 `F_WRLCK`。 `flockstr` 结构是一个值-结果参数，在返回时它包含了有关是否能够放置指定的锁的信息。如果允许加锁（即在指定的文件区域上不存在不兼容的锁），那么在 `l_type` 字段中会返回 `F_UNLCK`，并且剩余的字段会保持不变。如果在区域上存在一个或多个不兼容的锁，那么`flockstr` 会返回与那些锁中其中一把锁（无法确定是哪把锁）相关的信息，包括其类型（`l_type`）、字节范围（`l_start` 和 `l_len`；`l_whence` 总是返回为 `SEEK_SET`）以及持有这把锁的进程的进程 ID（`l_pid`）  



获取锁和释放锁的细节：

- 解锁一块文件区域总是会立即成功。即使当前并不持有一块区域上的锁，对这块区域解锁也不是一个错误  
- 在任何一个时刻，一个进程只能持有一个文件的某个特定区域上的一种锁。在之前已经锁住的区域上放置一把新锁会导致不发生任何事情（新锁和旧锁的类型一样）或原子地将既有锁转换成新模式。将一个读锁转换成写锁时需要为调用返回一个错误（`F_SETLK`）或阻塞（`F_SETLKW`）做好准备  
- 一个进程永远都无法将自己锁在一个文件区域之外，即使通过多个引用同一文件的文件描述符放置锁也是如此。  



`fcntl()` 记录锁继承和释放：

- 由 `fork()` 创建的子进程不会继承记录锁，记录锁在 exec()中会得到保留。  
- 一个进程中的所有线程会共享同一组记录锁  
- 记录锁同时与一个进程和一个 i-node 关联。  



Linux 上进程获取锁的优先级:

- 排队的锁请求被准予的顺序是不确定的。如果多个进程正在等待加锁，那么它们被满足的顺序取决于进程的调度  
- 写者并不比读者拥有更高的优先权，反之亦然  



## 4. 强制加锁

到目前为止介绍的锁都是劝告式锁。 这意味着一个进程可以自由地忽略 `fcntl()`或 `flock()` 的使用或简单地在文件上执行 I/O。内核不会阻止进程的这种行为  

Linux 也允许 `fcntl()` 记录锁是强制式的。这表示需对每个文件 I/O 操作进行检查以判断其他进程在执行 I/O 所在的文件区域上是否持有任何不兼容的锁  

为了在 Linux 上使用强制式加锁就必须要在包含待加锁的文件的文件系统以及每个待加锁的文件上启用这一项功能。通过在挂载文件系统时使用（Linux 特有的） `–o mand` 选项能够在该文件系统上启用强制式加锁  



## 5. `/proc/locks`

通过检查 Linux 特有的 `/proc/locks` 文件中的内容能够查看系统中当前存在的锁  





## 6. “单例”进程

一些程序——特别是很多 daemon——需要确保同一时刻只有一个程序实例在系统中运行。 完成这项任务的一个常见方法是让 daemon 在一个标准目录中创建一个文件并在该文件上放置一把写锁。 daemon 在其执行期间一直持有这个文件锁并在即将终止之前删除这个文件。/var/run 目录通常是存放此类锁文件的位置。或者也可以在 daemon 的配置文件中加一行来指定文件的位置  





## 7. 老式加锁

在较早的不支持文件加锁的 UNIX 实现上可以使用一些特别的加锁技术  

`open(file, 0_CREAT | 0_EXCL,...)` 和 `unlink(file)`  

使用了 `O_CREAT` 和 `O_EXCL` 标记的 `open()` 调用有原子地执行检查文件的存在性以及创建文件两个步骤  

果两个进程尝试在创建一个文件时指定这些标记，那么就保证只有其中一个进程能够成功。（另一个进程会从 `open()` 中收到 `EEXIST` 错误。）这种调用与 `unlink()` 系统调用组合起来就构成了一种加锁机制的基础。获取锁可通过成功地使用 `O_CREAT` 和 `O_EXCL` 标记打开文件后，立即跟着一个 `close()` 来完成。释放锁则可以通过使用 `unlink()` 来完成。尽管这项技术能够正常工作，但它存在一些局限：

- 如果 `open()` 失败了， 即表示其他进程拥有了锁， 那么就必须要在某种循环中重试 `open()` 操作  
- 使用 `open()` 和 `unlink()` 获取和释放锁涉及到文件系统的操作， 这比记录锁要慢很多  
- 如果一个进程意外终止并且没有删除锁文件，那么锁就不会被释放。  
- 如果放置多把锁（即使用多个锁文件），那么就无法检测出死锁。如果发生了死锁，那么造成死锁的进程就会永远保持阻塞  
- 第二版的 NFS 不支持 `O_EXCL` 语义。 Linux 2.4 NFS 客户端也没有正确地实现 `O_EXCL`，即使是第三版的 NFS 以及之后的版本也没能完成这个任务  



# 五十六、socket

socket 是一种 IPC 方法，它允许位于同一主机或使用网络连接起来的不同主机上的应用程序之间交换数据  

## 1. 概述

domain：

- UNIX(`AF_UNIX, AF_LOCAL`) 允许在同一主机上的应用程序之间进行通信，地址是路径名，地址结构 `sockaddr_un`   
- IPv4(`AF_INET`)，地址结构 `sockaddr_in`
- IPv6(`AF_INET6`)，地址结构 `sockaddr_in6`  
- ...



socket 流和数据报区别：

- 流是可靠(可以保证发送者传输的数据会完整无缺地到达接收应用程序  )的传送，数据报是不可靠的
- 流没有保留边界，数据报有边界
- 流是面向连接的，数据报不面向连接



## 2. `socket()`

```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
// return fd on success or -1 on error
```

`type`一般会指定为 `SOCK_STREAM`(TCP 流)、`SOCK_DGRAM`(UDP 数据报)，还可以和 `SOCK_CLOEXEC`、`SOCK_NONBLOCK` 进行按位或操作  

`protocol` 一般指定为 0，在 `type` 为 `SOCK_RAW` 时会将 `protocol` 指定为 `IPPROTO_RAW`  



## 3. `bind()`

`bind()` 系统调用将一个 socket 绑定到一个地址上  

一般来讲，会将一个服务器的 socket bind 到一个地址  

```c
#include <sys/socket.h>

// 通用的 socket 地址数据结构，因为 AF_UNIX, IPv4, IPv6 的地址格式是不同的
struct sockaddr {
    sa_family_t sa_family;    // AF_*
    char        sa_data[14];  // socket address
};

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// return 0 on success or -1 on error
```



## 4. 流 socket

流 socket 类似于打电话  

A，B 进程都创建 socket，A 进程（通常是服务器）调用 `bind()` 将 socket 绑定到一个地址上（电话号码），然后调用 `listen()` 通知内核表示 A 进程接收 socket 连接（类似于有一个被别人知道且已注册的电话号码）。B 进程通过 `connect()` 到 A 的地址（拨打 A 的电话号码），之后 A 会调用 `accept()` 接受 B 进程的连接（接电话），在 A，B 进程建立连接之后，A，B 进程就可以进行双向通信（`read/write` 等）  

**listen**  

```c
#include <sys/socket.h>

int listen(int sockfd, int backlog);
// return 0 on success or -1 on error
```

无法在一个已连接的 socket 上执行 `listen()`  

`backlog` 定义了最大未决（排队的客户端）连接的数量，一般指定 `SOMAXCONN` 常量，Linux 上为 128，开源通过 `/proc/sys/net/core/somaxconn` 来调整这个限制  



**accept**  

`accept()` 系统调用在文件描述符 `sockfd` 引用的监听流 socket 上接受一个接入连接。如果在调用 `accept()` 时不存在未决的连接，那么调用就会阻塞直到有连接请求到达为止  

```c
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
// return fd on success or -1 on error
```

创建一个新 socket，这个新 socket 会与执行 `connect()` 的对等 socket 进行连接  



**connect**  

`connect()` 系统调用将文件描述符 `sockfd` 引用的主动 socket 连接到地址通过 `addr `和 `addrlen` 指定的监听 socket 上  

```c
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// return 0 on success or -1 on error
```



**close**  

终止一个流 socket 连接的常见方式是调用 `close()`  



## 5. 数据报 socket

数据报 socket 类似于邮政系统  

进程 A，B 使用 `socket()` 系统调用，A 程序（通常是服务器）使用 `bind()` 将其 socket 绑定到一个地址上，之后 B 进程（通常是客户端）不需要建立连接，可以直接使用 `sendto()` 到 A 绑定的地址（填好收件人地址把信件投递出去），如果要接收数据报，使用 `recvfrom()`，在没有数据时阻塞，`recvfrom()` 可以获取发送者地址  

当从一个地址向另一个地址发送多个数据报时是无法保证它们按照被发送的顺序到达的，甚至还无法保证它们都能够到达。由于底层的联网协议有时候会重新传输一个数据包，因此同样的数据包可能会多次到达  

```c
#include <sys/socket.h>

ssize_t recvfrom(int sockfd, void *buffer, size_t length, int flags,
                 struct sockaddr *srcaddr, socklen_t *addrlen);
// return number of bytes received, 0 on EOF or -1 on error

ssize_t sendto(int sockfd, const void *buffer, size_t length, int flags,
               const struct sockaddr *dstaddr, socklen_t *addrlen);
// return number of bytes sent, or -1 on error
```

不管 `length` 的参数值是什么，`recvfrom()` 只会从一个数据报 socket 中读取一条消息。如果消息的大小超过了 `length` 字节，那么消息会被静默地截断为 `length` 字节  

还可以使用 `recvmsg()` 系统调用来获取数据报  



尽管数据报 socket 是无连接的，但在数据报 socket 上应用 `connect()` 系统调用仍然是起作用的。在数据报 socket 上调用 `connect()` 会导致内核记录这个 socket 的对等 socket 的地址。叫做已连接的数据报 socket  

在数据报 socket 已连接后：

- 数据报的发送可在 socket 上使用 `write()`（或 `send()` ）来完成并且会自动被发送到同样的对等 socket 上。与 `sendto()` 一样，每个 write()调用会发送一个独立的数据报  
- 在这个 socket 上只能读取由对等 socket 发送的数据报  



上面的论断只适用于调用了 `connect()` 数据报 socket，并不适用于它连接的远程 socket  

通过再发起一个 `connect()` 调用可以修改一个已连接的数据报 socket 的对等 socket。此外，通过指定一个地址族（如 UNIX domain 中的 `sun_family` 字段）为 `AF_UNSPEC` 的地址结构还可以解除对等关联关系  

为一个数据报 socket 设置一个对等 socket，这种做法的一个明显优势是在该 socket 上传输数据时可以使用更简单的 I/O 系统调用，即无需使用指定了 `dstaddr` 和 `addrlen` 参数的 `sendto()`，而只需要使用 write()即可。  



# 五十七、UNIX Domain

UNIX 域不太常用，大致介绍，其他暂时跳过  

## 1. 地址表示

```c
struct sockaddr_un {
    sa_family_t sum_family; // AF_UNIX
    char sum_path[108];     // null-terminated socket pathname
};
```



## 2. 流 socket



## 3. 数据报 socket



## 4. socket 权限



## 5. `socketpair()`



## 6. 抽象路径名空间





# 五十八、TCP/IP 网络基础

见计算机网络 notes，该章节跳过  



# 五十九、Internet Domain

## 1. 概述

Internet domain 流 socket 是基于 TCP 之上的，它们提供了可靠的双向字节流通信信道。
Internet domain 数据报 socket 是基于 UDP 之上的。UNIX domain 数据报 socket 是可靠的， 但 UDP socket 则是不可靠的，在一个 UNIX domain 数据报 socket 上发送数据会在接收 socket 的数据队列为满时阻塞。与之不同的是，使用 UDP 时如果进入的数据报会使接收者的队列溢出，那么数据报就会静默地被丢弃  



## 2. 网络字节序

因为有大端小端序，所以发送到网络的字节序需要统一，网络字节序是大端字节序  

相关的转换函数：

```c
#include <arpa/inet.h>

uint16_t htons(uint16_t host_uint16); // host to net short(uint16)
uint32_t htonl(uint16_t host_uint32); // host to net long(uint32)

uint16_t ntohs(uint16_t host_uint16); // net to host short(uint16)
uint32_t ntohl(uint16_t host_uint32); // net to host long(uint32)
```



## 3. IP 地址

```c
// ipv4
struct in_addr {
    in_addr_t  s_addr; // uint32
};

struct sockaddr_in {
    sa_family_t    sin_family;  // AF_INET
    in_port_t      sin_port;
    struct in_addr sin_addr; 
    unsigned char  __pad[X];    // pad to size of sockaddr(16 bytes)
};
```



```c
// ipv6
struct in6_addr {
    uint8_t s6_addr[16]; // 128 bits
};

struct sockaddr_in6 {
    sa_family_t     sin6_family; // AF_INET6
    in_port_t       sin_port;
    uint32_t        sin6_flowinfo;  // flow information
    struct in6_addr sin6_addr;
    uint32_t        sin6_scope_id;  // scope id
};
```

IPv6 环回地址可以用 `IN6_ADDR_ANY_INIT` 来初始化  

```c
const struct in6_addr_any = IN6ADDR_ANY_INIT;
```



```c
// storage
#define __ss_aligntype uint32_t; // uint32 on 32-bit or uint64 on 64-bit

struct sockaddr_storage {
    sa_family_t    ss_family;
    __sa_aligntype __ss_align;
    char           __ss_pading[SS_PADSIZE]; // 128 bits
};
```





## 4. 地址转换

**二进制和可读形式转换(IPv4，旧 API)**  

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int inet_aton(const char *cp, struct in_addr *inp);
// returns 1 if the supplied string was successfully interpreted,
// or 0 if the string is invalid

char *inet_ntoa(struct in_addr in);
```



**二进制和可读形式转换(IPv4 和 IPv6)**  

```c
#include <arpa/inet.h>

int inet_pton(int domain, const char *srcstr, void *addrptr);
// return 1 on successful conversion, 0 if srcstr is not in 
// presentation format, or -1 on error

const char *inet_ntop(int domain, const void *addrptr, char *dststr, size_t len);
// return pointer to dststr on success or NULL on error
```



## 5. 域名系统

`getaddrinfo` 获取一个 host 对应的 IP 地址，`getnameinfo` 获取一个 IP 地址对应的 host  

过时的 API 有 `gethostbyname()`，`getservbyname()` 和 `getservbyport`  

```c
#include <sys/socket.h>
#include <netdb.h>

struct addrinfo {
    int ai_flags;       // AI_*: input flags
    int ai_family;
    int ai_socktype;    // SOCK_STREAM|SOCK_DGRAM
    size_t ai_protocol;
    char *ai_canonname; // canonical name of host
    struct sockaddr *ai_addr;
    struct sockaddr *ai_next;
};

int getaddrinfo(const char *host, const char *service,
                const struct *addrinfo *hints, struct addrinfo **result);
// return 0 on success or nonzero on error
```

`host`：主机名或 IPv4 点分十进制或 IPv6 十六进制地址字符串  

`service`：服务名或一个端口号  

`hints`：规定了通过 `result` 返回地址结构的标准  

`ai_flags`：

- `AI_ADDRCONFIG`：在本地系统上至少配置了一个 IPv4 地址时返回 IPv4 地址，在本地系统上至少配置了一个 IPv6 系统时返回 IPv6 地址，都是非环回地址
- `AI_ALL`
- `AI_CANONNAME`：如果 host 不为 NULL，那么返回一个以 null 结尾的字符串，包含了主机的规范名  
- `AI_NUMERICHOST`：强制将 host 解释成一个数值地址字符串  
- `AI_NUMERICSERV  `：将 service 解释成一个数值端口号  
- `AI_PASSIVE`：返回一个适合进行被动式打开（即一个监听 socket）的 socket 地址结构  
- `AI_V4MAPPED`：如果在 hints 的 `ai_family` 字段中指定了 `AF_INET6`，那么在没有找到匹配的 IPv6 地址时应该在 result 返回 IPv4 映射的 IPv6 地址结构，如果指定了 `AI_ALL` 和 `AI_V4MAPPED`，那么在 result 中会同时返回 IPv4 和 IPv6 地址，其中 IPv4 地址会被返回成 IPv4 映射的 IPv6 地址结构  



对 `addrinfo` 链表进行释放：

```c
#include <sys/socket.h>
#include <netdb.h>

void freeaddrinfo(struct addrinfo *result);
```



在 `getaddrinfo()` 发生错误时，可以使用 `gai_strerror()` 来通过错误码获取错误信息  

```c
#include <netdb.h>

const char *gai_strerror(int errcode);
// return pointer to string containing error msg
```



`getnameinfo`  

```c
#include <sys/socket.h>
#include <netdb.h>

int getnameinfo(const struct sockaddr *addr, socklen_t addrlen, char *host,
                size_t hostlen, char *service, size_t servlen, int flags);
// return 0 on success or nonzero on error
```

可以使用 `NI_MAXHOST` 和 `NI_MAXSERV` 分别传入到 `hostlen` 和 `servlen`  

`flags`: 

- `NI_DGRAM`：在默认情况下，`getnameinfo()` 返回与流 socket（即 TCP）服务对应的名字，NI_DGRAM 标记会强制返回数据报 socket（即 UDP）服务的名字  
- `NI_NAMEREQD`：如果无法解析主机名，那么在 `host` 中会返回一个数值地址字符串。如果指定了 `NI_NAMEREQD`，那么就会返回一个错误（`EAI_NONAME`）。  
- `NI_NOFQDN`：默认情况下会返回主机的完全限定域名。指定 `NI_NOFQDN` 标记会导致当主机位于局域网中时只返回名字的第一部分（即主机名）  
- `NI_NUMERICHOST`：强制在 host 中返回一个数值地址字符串  
- `NI_NUMERICSERV`：强制在 service 中返回一个十进制端口号字符串  



## 6. `/etc/services`

一些公共的服务和端口号是固定的，会把服务名和端口号记录到 `/etc/services` 文件中，`getaddrinfo()` 和 `getnameinfo()` 会使用这个文件的信息在服务名和端口号之间转换  



# 六十、服务器设计

## 1. 迭代和并发

迭代：服务器每次只处理一个客户端，只有当完全处理完一个客户端的请求后才去处理下一个客户端。  

并发：这种类型的服务器被设计为能够同时处理多个客户端的请求  



## 2. I/O 模型

阻塞式 I/O：用户进程进行 I/O 系统调用，陷入到内核，阻塞直到数据准备好并复制到用户空间，最后成功返回  

非阻塞式 I/O：用户进程不断进行 I/O 系统调用，如果无数据，立即返回并设置 `errno` 为 `EWOULDBLOCK`；否则，当数据准备好并复制到用户空间，成功返回  

I/O 多路复用（事件驱动 I/O）：可以使用 `select/poll/epoll/kqueue` 在一个线程中监听多个文件描述符，当其中一个数据准备好，就通知进程，之后进行 I/O 系统调用，把数据从内核空间拷贝到用户空间，之后成功返回，其中进行 I/O 的文件描述符为非阻塞的  

信号驱动式 I/O：不常用，让内核在描述符就绪时发送 `SIGIO` 信号通知用户进程  

异步 I/O：用户进程发出系统调用后立即返回，内核等待数据准备完成，然后将数据拷贝到用户进程缓冲区，然后发送信号告诉用户进程 I/O 操作执行完毕



信号驱动和异步的区别在于信号驱动是在数据准备好之后就通知用户进程，用户进程通过 I/O 系统调用进行处理，异步 I/O 通过异步 I/O 系统调用（比如 `aio_read`），当数据准备好之后还不会发送信号，直到 I/O 执行结束（把内核空间的数据拷贝到用户空间），才会发送信号到用户进程  



## 3. 设计模型

**每个请求一个线程（多线程或线程池）+ 阻塞 I/O**  

简单易行，不过是长连接而且请求多时会出现性能问题  



**Reactor 模型**  

利用事件驱动机制，应用程序需要提供相应的接口并注册到 Reactor 上，如果相应的事件发生，将会调用之前注册的回调函数

Reactor 是被动的事件和分发模型，等待请求的到来，再分别对请求做出相应的处理，事件串行化避免多线程的编程复杂性

实现较为简单，可以模块化，由于回调机制导致调试比较困难  

也可以将多线程来使用 Reactor  

- 单线程 Reactor，acceptor 和事件处理函数 handler 都在一个线程中（接受新连接和处理请求事件都放到一个线程中）
- 多线程 Reactor，将非 I/O 的业务逻辑剥离出主 Reactor，放到一个线程池中，让工作线程进行处理，保证 `Reactor` 线程不会阻塞，而 `Reactor` 仍为单个线程仍然要处理连接和 I/O 操作
- 主从 Reactor，主 Reactor 线程也就是 `Acceptor` 只负责建立连接，建立之后将其注册到 从 Reactor 线程 中，一个连接注册一个 `Reactor` 线程上，非 I/O 请求（具体逻辑处理）的任务则会交由工作线程池处理

案例：`libevent, Redis, ACE` 等等



**Proactor 模型**

应用程序初始化一个异步 I/O 操作，然后注册相应的事件处理函数 handler，这个 handler 关注 I/O 操作执行完成的的事件，而不是 Reactor 那样关注可以读取的事件。之后切换处理其他的事情，等待 I/O 操作处理完成，当等待的 I/O 操作完成（内核写会到用户空间）时，事件分离器捕获到事件完成的信号后，激活事件处理器，执行之前注册的回调函数  

Proactor 是主动的事件分离和分发模型，允许多个任务并发执行  

Proactor 性能更高，能够处理耗时长的并发场景，实现的逻辑复杂，依赖操作系统对异步的支持  



**半同步半异步(Half-Sync/Half-Async)**  

单独一个 I/O 线程来异步处理网络 I/O，使用线程池来同步处理请求，Leader-Follower 模型属于该模型变体  



**全异步**  

比如 C++ 的 `future/promise`，Go 的 goroutine，还有一些语言的 `async/await`，还有 Python 的 `yield`，高效，但是用某种语言编写业务逻辑可能困难    





## 4. `inetd`

守护进程 `inetd` 被设计为用来消除运行大量非常用服务器进程的需要  

- 与其为每个服务运行一个单独的守护进程，现在只用一个进程 `inetd` 守护进程—就可以监视一组指定的套接字端口，并按照需要启动其他的服务，降低系统上运行的进程数量  
- `inetd` 简化了启动其他服务的编程工作  





# 六十一、socket 高级主题

## 1. 流部分读写

如果套接字上可用的数据比在 `read()` 调用中请求的数据要少，那就可能会出现部分读的现象。在这种情况下，`read()` 简单地返回可用的字节数  

出现以下其中一条可能会发生部分写现象：

- 在 `write()` 调用传输了部分请求的字节后被信号处理例程中断  
- 套接字工作在非阻塞模式下，可能当前只能传输一部分请求的字节  
- 在部分请求的字节已经完成传输后出现了一个异步错误，比如由于 TCP 连接出现问题  

可以实现 `readn` 和 `writen` 函数来循环调用 `read()` 和 `write`，确保请求的字节数全部得到传输  



## 2. `shutdown`

```c
#include <sys/socket.h>

int shutdown(int sockfd, int how);
// return 0 on success or -1 on error
```

`how`：

- `SHUT_RD` 关闭读端，之后的读操作返回 `EOF`，对 TCP socket 来说意义不大  
- `SHUT_WR` 关闭写端，在 shutdown()中最常用到的操作就是 `SHUT_WR`，有时候也被称为半关闭套接字，关闭写端，对端会收到 `EOF`  
- `SHUT_RDWR` 关闭读写端，这等同于先执行 `SHUT_RD`，跟着再执行一次 `SHUT_WR` 操作  

`shutdown()` 并不会关闭文件描述符，之后仍然需要 `close()` 来关闭文件描述符  



## 3. `recv/send`

`recv()` 和 `send()` 系统调用专用于已连接的套接字上执行 I/O 操作  

```c
#include <sys/socket.h>

ssize_t recv(int sockfd, void *buffer, size_t length, int flags);
// return number of bytes received, 0 on EOF, or -1 on error

ssize_t send(int sockfd, const void *buffer, size_t length, int flags);
// return number of bytes sent, or -1 on error
```

`recv flags`：

- `MSG_DONTWAIT`：让 `recv()` 以非阻塞方式执行。如果没有数据可用，错误码为 `EAGAIN`。`O_NONBLOCK` 的区别在于 `MSG_DONTWAIT` 允许我们在每次调用中控制非阻塞行为  
- `MSG_OOB`：在套接字上接收带外数据  
- `MSG_PEEK`：从套接字缓冲区中获取一份请求字节的副本，但不会将请求的字节从缓冲区中实际移除。这份数据稍后可以由其他的 `recv()` 或 `read()` 调用重新读取  
- `MSG_WAITALL`：指定了 `MSG_WAITALL` 标记后将导致系统调用阻塞，直到成功接收到 `length` 个字节。但是当 (a) 捕获到一个信号；(b) 流 socket 对端终止了连接；(c) 遇到了带外数据字节； (d) 数据报 socket 接收的数据报长度小于 `length` 字节； (e) socket 上出错。仍然会导致接收到的字节数小于 `length`  



`send flags`：

- `MSG_DONTWAIT`：让 `send()` 以非阻塞方式执行。如果数据不能立刻传输（因为套接字发送缓冲区已满），那么该调用不会阻塞，而是调用失败，伴随的错误码为 `EAGAIN`  
- `MSG_MORE`：在 TCP 套接字上，这个标记实现的效果同套接字选项 `TCP_CORK`；如果用于 UDP 数据报，在连续的 `send` 或 `sendto` 调用中传输的数据，会打包成一个单独的数据报，仅当下一次调用没有 `MSG_MORE` 才会传输出去（类似于 Linux 的 UDP_CORK）  
- `MSG_NOSIGNAL`：当在已连接的流式套接字上发送数据时，如果连接的另一端已经关闭了，指定该标记后将
  不会产生 `SIGPIPE` 信号。相反， `send()` 调用会失败，伴随的错误码为 `EPIPE`  
- `MSG_OOB`：在流式套接字上发送带外数据  



## 4. `sendfile()`

像 Web 服务器和文件服务器这样的应用程序常常需要将磁盘上的文件内容不做修改地通过（已连接）套接字传输出去。可以循环写到 `sockfd` 上  

还可以调用 `sendfile()` 时，文件内容会直接传送到套接字上，而不会经过用户空间。被称为 zero-copy transfer  



```c
#include <sys/sendfile.h>

ssize_t sendfile(int outfd, int infd, off_t *offset, size_t count);
// return number of bytes transferred, or -1 on error
```

参数 `infd` 指向的文件必须是可以进行 `mmap()` 操作的，我们可以使用 `sendfile()` 将数据从文件传递到 socket 上， 但反过来不行  



`TCP_CORK` socket 选项  

要进一步提高 TCP 应用使用 `sendfile()` 时的性能，采用 Linux 专有的套接字选项 `TCP_CORK` 常常会很有帮助  

当在 TCP 套接字上启用了 TCP_CORK 选项后，之后所有的输出都会缓冲到一个单独的 TCP报文段中，直到满足以下条件为止：已达到报文段的大小上限、取消了 `TCP_CORK` 选项、套接字被关闭。或者当启用 `TCP_CORK` 后，从写入第一个字节开始已经经历了 200 毫秒  



## 5. 获取 socket 地址

`getsockname()` 和 `getpeername()` 这两个系统调用分别返回本地套接字地址以及对端套接字地址  

```c
#include <sys/socket.h>

int getsockname(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
int getpeername(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
// return 0 on success or -1 on error
```

当隐式绑定到一个 Internet 域套接字上时，如果我们想获取内核分配给套接字的临时端口号，那么调用 `getsockname()` 也是有用的  

系统调用 `getpeername()` 返回流式套接字连接中对端套接字的地址。需要客户端的地址时需要使用  



## 6. 深入探讨 TCP

见计算机网络  



## 7. `netstat`

`netstat` 命令可以显示系统中 Internet 和 UNIX 域套接字的状态  

|   选项   |                      描述                      |
| :------: | :--------------------------------------------: |
|    -a    |      显示所有套接字的信息，包括监听套接字      |
|    -e    |   显示出扩展信息（包括套接字属主的用户 ID）    |
|    -c    |   连续重新显示套接字信息（每秒刷新显示一次）   |
|    -l    |             只显示监听套接字的信息             |
|    -n    | 显示 IP 地址、端口号并以数字形式显示出用户名称 |
|    -p    |    显示进程 ID 号以及套接字所归属的程序名称    |
| `--inet` |          显示 Internet 域套接字的信息          |
| `--tcp`  |     显示 Internet 域 TCP（流）套接字的信息     |
| `--udp`  |   显示 Internet 域 UDP（数据报）套接字的信息   |
| `--unix` |            显示 UNIX 域套接字的信息            |



## 8. `tcpdump`

TCP 输出形式：

```txt
timestamp src > dst: flags seqno ack window urg options

flags: S(SYN), F(FIN), P(PSH), R(RST), E(ECE), C(CWR)
```



## 9. socket 选项

```c
#include <sys/socket.h>

int getsockopt(int sockfd, int level, int optname, void *optval,
               socklen_t *optlen);
int setsockopt(int sockfd, int level, int optname, const void *optval,
               socklen_t *optlen);
// return 0 on success or -1 on error
```

`SOL_SOCKET`，这表示选项作用于套接字 API 层  

`optname` 标识了我们希望设定或得到的套接字选项  

socket 选项有 `SO_TYPE` 获取 socket 类型(`SOCK_STREAM/SOCK_DGRAM`)，`SO_REUSEADDR` 避免当 TCP 服务器重启时，尝试将套接字绑定到当前已经同 TCP 结点相关联的端口上时出现的 `EADDRINUSE`（地址已使用）错误  

以下情况会出现 `EADDRINUSE`：

- 之前连接到客户端的服务器要么通过 `close()`，要么是因为崩溃（例如被信号杀死）而执行了一个主动关闭。 这就使得 TCP 结点将处于 `TIME_WAIT` 状态， 直到 2 倍的 `MSL` 超时过期为止  
- 服务器先创建一个子进程来处理客户端的连接。稍后，服务器终止，而子进程继续服务客户端，因而使得维护的 TCP 结点使用了服务器的 well-known port  



在监听套接字的文件描述符上设置的标记或选项，在 Linux 上，以下属性不会被 `accept()` 返回的连接文件描述符所继承，需要重新设置：

- 打开文件描述符相关的状态标记，可以通过 `fcntl()` 的 `F_SETFL` 所修改的标记，包括 `O_NONBLOCK` 和 `O_ASYNC`
- 文件描述符标记，可以通过 `fcntl()` 的 `F_SETFD` 来修改的标记，包括 `FD_CLOEXEC`  
- 信号驱动 I/O 相关联的文件描述符选项 `fcntl()` 的 `F_SETOWN` 已经 `F_SETSIG`  



## 10. 带外数据

带外数据是流式套接字的一种特性，允许发送端将传送的数据标记为高优先级。带外数据的发送和接收需要在 `send()` 和 `recv()` 中指定 `MSG_OOB` 标记。  当一个套接字接收到带外数据可用的通知时，内核为套接字的属主（通常是使用该套接字的进程）生成 `SIGURG` 信号  

当采用 TCP 套接字时，任意时刻最多只有 1 字节数据可被标记为带外数据。  

不提倡使用带外数据的，  



## 11. `*msg()`

`sendmsg()` 和 `recvmsg()` 是套接字 I/O 系统调用中最为通用的两种。`sendmsg()` 系统调用能做到所有 `write()`、`send()` 以及 `sendto()` 能做到的事；`recvmsg()` 系统调用能做到所有 `read()`、`recv()` 以
及 `recvfrom()` 能做到的事，还有以下功能：

- 同 `readv()` 和 `writev()` 一样， 我们可以执行分散-聚合 I/O  
- 我们可以传送包含特定于域的辅助数据  



# 六十二、终端

## 1. 概述

传统型终端和终端模拟器都需要同终端驱动程序相关联，由驱动程序负责处理设备上的输入和输出  

当执行输入时，驱动程序可以工作在以下两种模式下:

- 规范模式：在这种模式下，终端的输入是按行来处理的，而且可进行行编辑操作  
- 非规范模式：终端输入不会被装配成行。像 vi、 more 和 less 这样的程序会将终端置于非规范模式，这样不需要用户按下回车键它们就能读取到单个的字符了  

终端驱动程序会对两个队列做操作：一个用于从终端设备将输入字符传送到读取进程上，另一个用于将输出字符从进程传送到终端上。  



## 2. 获取和设置终端

```c
#include <termios.h>

struct termios {
  	tcflag_t c_iflag; // input flags
    tcflag_t c_oflag; // output flags
    tcflag_t c_cflag; // control flags
    tcflag_t c_lflag; // local modes
    cc_t     c_cc[NCCS];  // speical characters
};

int tcgetattr(int fd, struct termios *ptermios);
int tcsetattr(int fd, int optionalactions, const struct )
```

`optionalactions`：

- `TCSANOW`：修改立刻得到生效  
- `TCSADRAIN`：当所有当前处于排队中的输出已经传送到终端之后，修改得到生效  
- `TCSAFLUSH`：该标志的产生的效果同 `TCSADRAIN`，但是除此之外，当标志生效时那些仍然等待处理的输入数据都会被丢弃。比如，当读取一个密码时，此时我们希望关闭终端回显功能，并防止用户提前输入  



## 3. `stty` 命令

`stty` 命令是以命令行的形式来模拟函数 `tcgetattr()` 和 `tcsetattr()` 的功能  



## 4. 终端特殊字符

|  字符   | `c_cc` 下标 |       描述        | 默认设定 |
| :-----: | :---------: | :---------------: | :------: |
|   CR    |   （无）    |       回车        |    ^M    |
| DISCARD |  VDISCARD   |     丢弃输出      |    ^O    |
|   EOF   |    VEOF     |     文件结尾      |    ^D    |
|   EOL   |    VEOL     |      行结尾       |          |
|  EOL2   |    VEOL2    |   另一种行结尾    |          |
|  ERASE  |   VERASE    |     擦除字符      |    ^?    |
|  INTR   |    VINTR    |  中断（ SIGINT）  |    ^C    |
|  KILL   |    VKILL    |     擦除一行      |    ^U    |
|  LNEXT  |   VLNEXT    |  字面化下个字符   |          |
|   NL    |   （无）    |       换行        |    ^J    |
|  QUIT   |    VQUIT    | 退出（ SIGQUIT）  |    ^\    |
| REPRINT |  VREPRINT   |  重新打印输入行   |    ^R    |
|  START  |   VSTART    |     开始输出      |    ^Q    |
|  STOP   |    VSTOP    |     停止输出      |    ^S    |
|  SUSP   |    VSUSP    | 暂停（`SIGTSTP`） |    ^Z    |
| WERASE  |   VWERASE   |   擦除一个 word   |    ^W    |



## 5. 终端的 I/O 模式

**规范模式**  

可通过设定 `ICANON` 标志来打开规范模式输入，规范模式：

- 输入被装配成行，通过如下几种行结束符来终结：`NL`、`EOL`、`EOL2`(如果启用 `IEXTEN`)、`EOF`、`CR`(如果启用 `ICRNL`)  
- 打开了行编辑功能， 这样可以修改当前行中的输入。 因此， 下列字符是可用的： `ERASE`、
  `KILL`、`WERASE`(如果设定了 `IEXTEN` 标志)  
- 如果设定了 `IEXTEN` 标志，则 `REPRINT` 和 `LNEXT` 字符也都是可用的  



**非规范模式**  

在非规范模式下（关闭 `ICANON` 标志）不会处理特殊的输入。特别的一点是：输入不再装配成行，相反会立刻对应用程序可见  



# 六十三、其他 I/O

主要介绍 I/O 多路复用、信号驱动 I/O、Linux 专有的 `epoll`  

## 1. 概述

**非阻塞式 I/O** 可以让我们周期性地检查（“轮询”） 某个文件描述符上是否可执行 I/O 操作，但是在一个紧凑的循环中做轮询就是在浪费 CPU。  

**每个请求一个进程/线程**：我们可以创建一个新的进程来执行 I/O。此时父进程就可以去处理其他的任务了，而子进程将阻塞直到 I/O 操作完成。  

可以使用 **I/O 多路复用的方式**：允许进程同时检查多个文件描述符以找出它们中的任何一个是否可执行 I/O 操作(`select` 和 `poll` 复杂度是 O(n)，n 为监听文件描述符的数量，可以使用 Linux 特有的 `epoll`，复杂度是 O(log n))  



**水平触发**：如果文件描述符上可以非阻塞地执行 I/O 系统调用，此时认为它已经就绪  

**边缘触发**：如果文件描述符自上次状态检查以来有了新的 I/O 活动，此时需要触发通知  

`select` 和 `poll` 只能水平触发，信号驱动 I/O 只能边缘触发，`epoll` 两种都可以  



由于水平触发模式允许我们在任意时刻重复检查 I/O 状态，没有必要每次当文件描述符就绪后需要尽可能多地执行 I/O（也就是尽可能多地读取字节，亦或是根本不去执行任何 I/O）  

当我们采用边缘触发时，只有当 I/O 事件发生时我们才会收到通知。在另一个 I/O 事件到来前我们不会收到任何新的通知。另外，当文件描述符收到 I/O 事件通知时，通常我们并不知道要处理多少 I/O（例如有多少字节可读）。因此，采用边缘触发通知的程序通常要按照如下规则来设计：

- 在接收到一个 I/O 事件通知后，程序在某个时刻应该在相应的文件描述符上尽可能多地执行 I/O（比如尽可能多地读取字节）。  
- 每个被检查的文件描述符通常都应该置为非阻塞模式，在得到 I/O 事件通知后重复执行 I/O 操作，直到相应的系统调用以错误码 `EAGAIN` 或 `EWOULDBLOCK` 的形式失败。  



## 2. I/O 多路复用

**select**  

```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

void FD_CLR(int fd, fd_set *set);
int  FD_ISSET(int fd, fd_set *set);
void FD_SET(int fd, fd_set *set);
void FD_ZERO(fd_set *set);

int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
// return number of ready file descriptors, 0 on timeout, or -1 on error
```

`readfds` 是用来检测输入是否就绪的文件描述符集合  

`writefds` 是用来检测输出是否就绪的文件描述符集合  

`exceptfds` 是用来检测异常情况是否发生的文件描述符集合  



Linux 上，异常情况只会在下面两种情况下发生：

- 连接到处于信包模式下的伪终端主设备上的从设备状态发生了改变  
- 流式套接字上接收到了带外数据  



`fd_set` 用 `FD_*` 四个宏操作，`FD_ZERO` 清空，`FD_SET` 将 `fd` 加入到 `fd_set` 中，`FD_CLR` 将 `fd` 从 `fd_set` 中移除  

文件描述符集合有一个最大容量限制，由常量 `FD_SETSIZE` 来决定。在 Linux 上，该常量的值为 1024。



**poll**  

系统调用 `poll()` 执行的任务同 `select()` 很相似。 两者间主要的区别在于我们要如何指定待检查的文件描述符。  

```c
#include <poll.h>

struct pollfd {
    int fd;
    short events;  // requested events bit mask
    short revents; // returned events bit mask
};

int poll(struct pollfd fds[], nfds_t nfds, int timeout);
// return number of ready fd, 0 on timeout or -1 on error
```



`events` 和 `revents` 掩码值  

|    位掩码    | `events` 输入 | 返回到 `revents` |                描述                |
| :----------: | :-----------: | :--------------: | :--------------------------------: |
|  *`POLLIN`   |       v       |        v         |        可读取的非优先级数据        |
| `POLLRDNORM` |       v       |        v         |          等同于 `POLLIN`           |
| `POLLRDBAND` |       v       |        v         | 可读取的优先级数据(Linux 中不使用) |
|  *`POLLPRI`  |       v       |        v         |         可读取高优先级数据         |
| *`POLLRDHUP` |       v       |        v         |           对端套接字关闭           |
|  *`POLLOUT`  |       v       |        v         |            普通数据可写            |
| `POLLWRNORM` |       v       |        v         |          等同于 `POLLOUT`          |
| `POLLWRBAND` |       v       |        v         |          优先级数据可写入          |
|  *`POLLERR`  |               |        v         |             有错误发生             |
|  *`POLLHUP`  |               |        v         |              出现挂断              |
|  `POLLNVAL`  |               |        v         |          文件描述符未打开          |
|  `POLLMSG`   |               |                  |           Linux 中不使用           |



Linux 上，`poll` 关心的位掩码只有上表带 `*` 的  



**文件描述符何时就绪**  

如果 I/O 函数调用不会阻塞，而不论该函数是否能够实际传输数据，此时文件描述符（未指定 `O_NONBLOCK` 标志）被认为是就绪的，`select()` 和 `poll()` 只会告诉我们 I/O操作是否会阻塞，而不是告诉我们到底能否成功传输数据  

详细分析 `select` 和 `poll` 在不同类型的文件上发生的情况，`select`  用 `r/w/x` 分别表示文件描述符是否被标记为可读、可写、异常，`poll` 分别使用`POLLIN/POLLOUT/POLLHUP/POLLERR` 来表示  



*普通文件*

普通文件的文件描述符总是被 `select()` 标记为可读和可写。对于 `poll()` 来说，则会在 `revents` 字段中返回 `POLLIN` 和 `POLLOUT` 标志  



*终端和伪终端*  

|                     条件或事件                     | `select` |   `poll`   |
| :------------------------------------------------: | :------: | :--------: |
|                       有输入                       |    r     |  `POLLIN`  |
|                       可输出                       |    w     | `POLLOUT`  |
|             伪终端对端调用 `close` 后              |   `rw`   | 取决于实现 |
| 处于信包模式下的伪终端主设备检测到从设备端状态改变 |    x     | `POLLPRI`  |



*pipe 和 FIFO*  

读端通知：

| 管道中是否有数据？ | 写端是否打开？ | `select` |       `poll`       |
| :----------------: | :------------: | :------: | :----------------: |
|         否         |       否       |    r     |     `POLLHUP`      |
|         是         |       是       |    r     |      `POLLIN`      |
|         是         |       否       |    r     | `POLLIN | POLLHUP` |

写端通知：

| 是否有 `PIPE_BUF` 个字节的空间？ | 读端是否打开？ | `select` |       `poll`        |
| :------------------------------: | :------------: | :------: | :-----------------: |
|                否                |       否       |    w     |      `POLLERR`      |
|                是                |       是       |    w     |      `POLLOUT`      |
|                是                |       否       |    w     | `POLLOUT | POLLERR` |



*socket*  

|                     条件或事件                     | `select` |           `poll`           |
| :------------------------------------------------: | :------: | :------------------------: |
|                       有输入                       |    r     |          `POLLIN`          |
|                       可输出                       |    w     |         `POLLOUT`          |
|               在监听套接字上建立连接               |    r     |          `POLLIN`          |
|             接收到带外数据（只限 TCP）             |    x     |         `POLLPRI`          |
| 流套接字的对端关闭连接或执行了 shutdown(`SHUT_WR`) |   `rw`   | `POLLIN|POLLOUT|POLLRDHUP` |



**select 和 poll 的比较：**

- 实现细节：都使用了内核 poll 
- API：`select` 监听描述上限是 `FD_SETSIZE`，一般是 1024，poll 理论上没有上限（文件描述符上限）；`select()` 提供的超时精度（微秒）比 `poll()` 提供的超时精度（毫秒）高；如果其中一个被检查的文件描述符关闭了，通过在对应的 `revents` 字段中设定 `POLLNVAL` 标记， `poll()` 会准确告诉我们是哪一个文件描述符关闭了。 与之相反， `select()` 只会返回 -1，并设错误码为 `EBADF`  
- 可移植性：`select` 更好，不过区别不大
- 性能：都是`O(nfds)`， `select` 适用于待检查的文件描述符范围较小；如果被检查的文件描述符集合很稀疏的话，使用 `poll` 更好一些  



**存在的问题**  

- 每次调用 `select()` 或 `poll()`，内核都必须检查所有被指定的文件描述符，看它们是否处于就绪态  
- 每次调用 `select()` 或 `poll()` 时，程序都必须传递一个表示所有需要被检查的文件描述符的数据结构到内核，内核检查过描述符后，修改这个数据结构并返回给程序。对 `poll` 来说，传递给内核的数据结构大小随着监听文件描述符的增加而增大，对 `select` 而言，数据结构为固定的 `FD_SETSIZE`  
- `select()` 或 `poll()` 调用完成后，程序必须检查返回的数据结构中的每个元素，以此查明哪个文件描述符处于就绪态了  



## 3. 信号驱动 I/O

使用信号驱动 I/O 步骤：

- 为内核发送的通知信号安装一个信号处理例程。默认情况下，这个通知信号为 `SIGIO`  
- 设定文件描述符的属主，也就是当文件描述符上可执行 I/O 时会接收到通知信号的进程或进程组  
- 通过设定 `O_NONBLOCK` 标志使能非阻塞 I/O  
- 通过打开 `O_ASYNC` 标志使能信号驱动 I/O  
- 调用进程现在可以执行其他的任务了  
- 信号驱动 I/O 提供的是边缘触发通知。这表示一旦进程被通知 I/O 就绪，它就应该尽可能多地执行 I/O（例如尽可能多地读取字节）。 假设文件描述符是非阻塞式的，这表示需要在循环中执行 I/O 系统调用直到失败为止，此时错误码为 `EAGAIN` 或 `EWOULDBLOCK`  





## 4. `epoll`

`epoll` 和 `poll` 类似，当检测大量文件描述符时，`epoll` 性能比 `poll` 高很多。而且支持边缘触发和水平触发两种  

`epoll` API 包括 `create/ctl/wait` 3 个系统调用  

```c
// create

#include <sys/epoll.h>

int epoll_create(int size);
int epoll_create1(int flags);
// return fd on success or -1 on error
```

参数 size 指定了我们想要通过 `epoll` 实例来监听的文件描述符个数，现在已经被忽略  

`epoll_create1` 去掉了过时的 `size` 参数，支持 `EPOLL_CLOEXEC` flags  



控制 `epoll` 感兴趣的文件描述符列表：`epoll_ctl`

```c
#include <sys/epoll.h>

typedef union epoll_data {
    void *ptr;   // pointer to user-defined data
    int fd;
    uint32_t u32;
    uint64_t u64;
}

struct epoll_event {
    uint32_t     events;  // epoll events(bit mask)
    epoll_data_t data;    // user data
};

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *ev);
// return 0 on success or -1 on error
```

`op`：

- `EPOLL_CTL_ADD`：添加 `fd`，事件在 `ev` 指向的结构体中，如果兴趣列表中添加一个已存在的文件描述符， `epoll_ctl()` 将出现 `EEXIST` 错误  
- `EPOLL_CTL_MOD`：修改描述符 `fd` 上设定的事件，需要用到由 `ev` 所指向的结构体中的信息。如果我们试图修改不在兴趣列表中的文件描述符，`epoll_ctl()` 将出现 `ENOENT` 错误  
- `EPOLL_CTL_DEL`：删除 `fd`，忽略 `ev` 参数，如果移除一个不在 `epfd` 的兴趣列表中的文件描述符， `epoll_ctl()` 将出现 `ENOENT` 错误  



`events` 指定了我们为待检查的描述符 `fd` 上所感兴趣的事件集合，感兴趣的事件和 `poll` 类似，有 `EPOLL_IN/EPOLL_HUP/EPOLL_PRI/EPOLL_OUT/EPOLL_RDHUP/EPOLL_ERR`  



因为每个注册到 `epoll` 实例上的文件描述符需要占用一小段不能被交换的内核内存空间，因此内核提供了一个接口用来定义每个用户可以注册到 `epoll` 实例上的文件描述符总数。这个上限值可以通过 `max_user_watches` 来查看和修改。 max_user_watches 是专属于 Linux 系统的 `/proc/sys/fd/epoll` 目录下的一个文件。  



事件等待：`epoll_wait`，单个 `epoll_wait()` 调用能返回多个就绪态文件描述符的信息  

```c
#include <sys/epoll.h>

int epoll_wait(int epfd, struct epoll_event *evlist, int maxevents, int timeout);
// return number of ready fd, 0 on timeout or -1 on error
```

参数 `evlist` 所指向的结构体数组中返回的是有关就绪态文件描述符的信息  

数组 `evlist` 的空间由调用者负责申请，所包含的元素个数在参数 `maxevents` 中指定  

在数组 `evlist` 中，每个元素返回的都是单个就绪态文件描述符的信息。 `events` 字段返回了在该描述符上已经发生的事件掩码。 `data` 字段返回的是我们在描述符上使用 `cpoll_ctl()` 注册感兴趣的事件时在 `ev.data` 中所指定的值  

调用 `epoll_ctl()` 将文件描述符添加到兴趣列表中时，应该要么将 `ev.data.fd` 设为文件描述符号，要么将 `ev.data.ptr` 设为指向包含文件描述符号的结构体  



`timeout`：

- -1：调用将一直阻塞，直到兴趣列表中的文件描述符上有事件产生，或者直到捕获到一个信号为止。  
- 0：执行一次非阻塞式的检查，看兴趣列表中的文件描述符上产生了哪个事件。  
- 大于 0：调用将阻塞至多 timeout 毫秒，直到文件描述符上有事件发生，或者直到捕获到一个信号为止  



在多线程程序中，可以在一个线程中使用 `epoll_ctl()` 将文件描述符添加到另一个线程由 `epoll_wait()` 所监视的 `epoll` 实例的兴趣列表中去  



**`EPOLLONESHOT` 标志**  

一旦通过 `epoll_ctl()` 的 `EPOLL_CTL_ADD` 操作将文件描述符添加到 `epoll` 实例的兴趣列表中后，它会保持激活状态直到我们显式地通过 `epoll_ctl()` 的 `EPOLL_CTL_DEL` 操作将其从列表中移除  

如果我们希望在某个特定的文件描述符上只得到一次通知，那么可以在传给 `epoll_ctl()` 的 `ev.events` 中指定 `EPOLLONESHOT` 标志。如果指定了这个标志，那么在下一个 `epoll_wait()` 调用通知我们对应的文件描述符处于就绪态之后，这个描述符就会在兴趣列表中被标记为非激活态，之后的 `epoll_wait()` 调用都不会再通知我们有关这个描述符的状态了  



**使用 `epoll` 的步骤**： 

- 创建 `epoll` 实例

- 将感兴趣的 `fd` 和事件使用 `epoll_ctl` 添加到兴趣列表中，一般需要检查的事件为 `EPOLLIN`  

- 执行一个循环调用 `epoll_wait`，来监听 `epoll` 感兴趣列表中的文件描述符  

- 当 `epoll_wait` 调用之后，需要检查是否返回了 `EINTR` 错误码，如果在 `epoll_wait` 调用执行期间被信号打断，之后通过 `SIGCONT` 信号恢复执行，此时就可能出现这个错误，这种情况应用程序需要重新继续执行 `epoll_wait`调用

-  如果 `epoll_wait()` 调用成功，程序就再执行一个内层循环检查 `evlist` 中每个已就绪的元素，不止检查调用 `epoll_ctl` 时所添加感兴趣的事件，也要检查 `EPOLLHUP` 和 `EPOLLERR` 等标记

  ```c
  int ready = epoll_wait(epfd, evlist, MAX_EVENTS, -1);
  // check ready...
  // successfuly call epoll_wait
  for (int i = 0; i < ready; ++i) {
      if (evlist[i].events & EPOLLIN) {
          // read syscall
      } else if (evlist[i].events & (EPOLLHUP | EPOLLERR))
      
      
  }
  ```

  

`epoll` API 的应用场景就是需要同时处理许多客户端的服务器：需要监视大量的文件描述符，但大部分处于空闲状态，只有少数文件描述符处于就绪态  



**`epoll` 的触发机制**  

默认情况下 `epoll` 提供的是水平触发通知。这表示 `epoll` 会告诉我们何时能在文件描述符上以非阻塞的方式执行 I/O 操作  

`epoll` API 还能以边缘触发方式进行通知—也就是说，会告诉我们自从上一次调用 `epoll_wait()` 以来文件描述符上是否已经有 I/O 活动了（或者由于描述符被打开了，如果之前没有调用的话）。使用 `epoll` 的边缘触发通知在语义上类似于信号驱动 I/O，只是如果有多个 I/O 事件发生的话， `epoll` 会将它们合并成一次单独的通知  

使用边缘触发通知，我们在调用 `epoll_ctl()` 时在 `ev.events` 字段中指定 `EPOLLET` 标志  

```c
ev.events = EPOLLIN | EPOLLET;
```



`epoll` 水平触发和边缘触发区别的例子，使用 `epoll` 来监视一个套接字上的输入（ `EPOLLIN`）  

- socket 有数据到来
- 调用一次 `epoll_wait`，无论采用水平触发还是边缘触发，都会通知 socket 处于就绪态
- 再次调用 `epoll_wait` 

如果我们采用的是水平触发通知，那么第二个 `epoll_wait()` 调用将告诉我们套接字处于就绪态。  

而如果我们采用边缘触发通知，那么第二个 `epoll_wait()` 调用将阻塞，因为自从上一次调用 `epoll_wait()` 以来并没有新的输入到来  



`epoll` 边缘触发基本步骤：

- 让所有待监视的文件描述符都成为非阻塞的  
- 通过 `epoll_ctl()` 构建 `epoll` 的兴趣列表  
- 通过 `epoll_wait()` 取得处于就绪态的描述符列表  
- 针对每一个处于就绪态的文件描述符， 不断进行 I/O 处理直到相关的系统调用返回 `EAGAIN` 或 `EWOULDBLOCK` 错误  

采用边缘触发可能会导致文件描述符饥饿现象，该问题的一种解决方案是让应用程序维护一个列表，列表中存放着已经被通知为就绪态的文件描述符。通过一个循环按照如下方式不断处理：  

- 调用 `epoll_wait()` 监视文件描述符，并将处于就绪态的描述符添加到应用程序维护的列表中。如果这个文件描述符已经注册到应用程序维护的列表中了，那么这次监视操作的超时时间应该设为较小的值或者是 0。这样如果没有新的文件描述符成为就绪态，应用程序就可以迅速进行到下一步，去处理那些已经处于就绪态的文件描述符了  
- 在应用程序维护的列表中，只在那些已经注册为就绪态的文件描述符上进行一定限度的 I/O 操作（可能是以轮转调度方式循环处理，而不是每次 `epoll_wait()` 调用后都从列表头开始处理 ）。当相关的非阻塞 I/O 系 统调用出现 `EAGAIN` 或 `EWOULDBLOCK` 错误时，文件描述符就可以从应用程序维护的列表中移除了  





