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



