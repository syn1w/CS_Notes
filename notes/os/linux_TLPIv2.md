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

