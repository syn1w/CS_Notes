<!-- TOC -->
- [一、概述](#一概述)
- [二、常用命令](#二常用命令)
    - [1. 帮助](#1-帮助)    
    - [2. 文件和目录](#2-文件和目录)   
    - [3. 管理进程](#3-管理进程)    
    - [4. 外部存储](#4-外部存储)    
    - [5. shell](#5-shell)    
    - [6. 重定向和管道](#6-重定向和管道)    
    - [7. 环境变量](#7-环境变量)    
    - [8. 文件权限](#8-文件权限)   
    - [9. 管理文件系统](#9-管理文件系统)   
    - [10. 安装程序](#10-安装程序) 
    - [11. 编辑器](#11-编辑器) 
    - [12. 编译相关](#12-编译相关)
    - [13. 网络相关](#13-网络相关)    
    - [14. GDB](#14-gdb)    
    - [15. 其他](#15-其他)
- [三、shell 脚本](#三shell-脚本)    
    - [1. 基本脚本](#1-基本脚本)   
    - [2. 结构化命令](#2-结构化命令)  
    - [3. 处理用户输入](#3-处理用户输入)    
    - [4. 呈现数据](#4-呈现数据)    
    - [5. 控制脚本](#5-控制脚本)
- [四、高级 shell 脚本](#四高级-shell-脚本)    
    - [1. 函数](#1-函数)    
    - [2. 图形化中的脚本](#2-图形化中的脚本)    
    - [3. sed 编辑器](#3-sed-编辑器)   
    - [4. gawk 编辑器](#4-gawk-编辑器)  
    - [5. 正则表达式](#5-正则表达式)
- [五、有用的实例](#五有用的实例)    
    - [1. 归档](#1-归档)    
    - [2. 代码行数](#2-代码行数)
    - [3. 递归删除文件](#3-递归删除文件)
- [六、其他实用命令](#其他实用命令)  
    - [1. zsh](#1-zsh)
    - [2. ag](#2-ag)
    - [3. ccache](#3-ccache)
    - [4. tldr](#4-tldr)
    - [5. axel](#5-axel)

<!-- /TOC -->

# 一、概述

**内存管理**：虚拟内存映射到物理内存，交换空间

**进程管理**：自启动进程通常在 `/etc/inittab` 或者 `/etc/rcX.d` 下，X 表示运行级，1 为基本的系统进程和一个控制台终端进程，单用户模式。3 运行大多数命令行软件。5 运行 X Window 系统。

**硬件设备管理**：设备驱动程序可以编译进内核或者可插入内核的设备驱动模块。Linux 系统将硬件设备当成特殊的文件，有字符设备文件、块设备文件、网络设备文件。

**文件系统**：常见的有 `ext/ext2/ext3*/ext4*/msdos/nfs*/ntfs*/XFS(高性能64位日志文件系统)`。Linux 内核使用虚拟文件系统 VFS 作为和每个文件系统交互的接口。`df -T`

**桌面环境**：X Window、KDE(类似 Windows)、GNOME、Unity(Ubuntu)等。

**发行版**：

- RedHat：服务器和商业付费发行版=>CentOS(免费发行版)，Fedora(家用发行版)，openSUSE(商用和家用的发行版)
- Debian：Linux 专家和商用 Linux 产品中流行=>Ubuntu(学校和个人免费发行版)，Mint(家庭娱乐)，Deepin(桌面非常漂亮，国内开发)
- 其他小众：gentoo(高级用户)、Arch(KISS、滚动更新) 等

**文件类型**：目录(d)，常规文件(-)，字符设备(c)，块设备(b)，套接字文件(s)，管道 FIFO 文件(p)，符号链接文件(l)



# 二、常用命令

## 1. 帮助

```shell
man [section] xxx [-k keyword(不记得命令)]
# section: 
# 1可执行程序或者shell命令、2系统调用、3库调用、4特殊文件、5文件格式和约定
# 6游戏，7概览约定及杂项，8超级用户和系统管理员命令，9内核例程
# 命令和系统调用重名可以使用 section，比如
man kill     # KILL(1) cmd
man 2 kill   # KILL(2) syscall
```



## 2. 文件和目录

**文件和目录列表**

```shell
cd path
pwd                 # print working directory
ls  -[ailR] [path]  # path 可以使用 *[!]? 模式
                    # i: inode
                    # l: 类型、权限、硬链接数、user、group、size、time、name
```

**处理文件**

```shell
mkdir dirname     # 创建目录，新目录权限由 umask 决定，比如 umask == 0022，新目录为 rwxr-xr-x
touch filename    # 创建文件，新文件权限由 umask 决定，普通文件没有 x
                  # touch 原本的功能是 change file timestamps
                  # 当文件不存在 && 没有 -c 或者 -h 时，创建文件

cp [-iR] src dst  # 深拷贝 i(询问是否覆盖) R(recursive)，也可以使用 * 通配符
mv [] src dst     # move，不改变 inode 和时间戳，相当于改变了一下指针，类似于编程语言的中移动语义
```

```shell
ln file linkfile     # 硬链接，可以使用 ls -l 查看硬连接数，相当于引用计数，指向相同文件
# 硬链接不能给目录创建，因为会形成环。而且只有同一文件系统的文件之间才能创建硬链接。
# 现代 Unix 系统可能包含多种文件系统，这些文件系统位于不同的磁盘和 / 或 分区

ln -s file linkfile  # 软链接，也叫符号链接(symbol link)
# 软链接没有硬链接的那种限制，实质上是查找名称，类似于 Windows 上的快捷方式
```

```shell
rmdir dirname   # 删除空目录
rm [-fir] path  # 删除文件 force, 询问, recursive
rm -rf dirname  # 删除目录的终极大法
```

**文件内容**

```shell
file filename       # 显示文件类型
cat [-nb] filename  # 显示文本文件 n: 添加行号，b: 非空行加行号
more filename
less filename       # less is more, more 的升级版
tail [-n x] file    # 打印倒数 x 行
head [-x] file      # 打印开头 x 行
sort [-n]           # 对数据排序输出，默认为字符串字典序，n 为按数字序，其他方式见手册
wc [-cmlw]          # wordcount, c 字节，m 字符，l 行，w 单词
```

```shell
grep [options] pattern file  # 模式匹配
# options:
#   -v: 输出不匹配的行
#   -n: 输出匹配的行号和行
#   -c: 匹配行的数量
#   -e p1 -e p2: 匹配多个模式
# pattern: 可以使用 Unix 风格的正则表达式，egrep 是 POSIX 扩展正则表达式, fgrep 在大型文件中使用
```



## 3. 管理进程

**查看进程**

```shell
ps -ef             # Unix 风格
ps axu             # BSD  风格，显示项更多
# VSZ: 进程在内存中的大小，以 KB 为单位
# RSS: 进程在未换出时占用的物理内存
# STAT: 状态。< 高优先级，N 低优先级，L有页面所在内存中，s 控制进程，l多线程，+前台

top                # 实时监控
# PR: 进程优先级
# NI: 进程的谦让度值
# VIRT：进程占用的虚拟内存总量
# RES：进程占用的物理内存总量
# SHR：进程占用的物理内存总量。
# S：进程的状态。D 可中断的休眠， R 运行， S 休眠， T 跟踪状态或停止， Z 僵死
```

**结束进程**

```shell
kill pid             # 发送 TERM 信号
kill -s signal pid   # 发送 signal，可以为名称或信号值
killall pname        # 可以使用进程名来结束进程

# 使用信号进行 IPC，发送信号来结束进程
# Linux 信号: 
# 1 HUP 挂起，2 INT，3 QUIT，9 KILL，11 SEGV 段错误，15 TERM 尽可能终止
# 17 STOP 无条件停止运行，18 TSTP 后台运行，19 CONT 在 STOP 或者 TSTP 后恢复
```

关于上述 `INT, QUIT, KILL, TERM, STOP, HUP` 的区别：

> https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html
>
> https://www.quora.com/What-is-the-difference-between-the-SIGINT-and-SIGTERM-signals-in-Linux-What%E2%80%99s-the-difference-between-the-SIGKILL-and-SIGSTOP-signals

- `INT`， 按下 `Ctrl+C` 发给前台进程，默认行为是 terminate 进程，可以被捕获或忽略，意图是提供一种有序优雅 shutdown 的机制
- `QUIT`， dump core，按下 `Ctrl+\` 发送给前台进程，默认 terminate 进程，可以被捕获或忽略，意图是提供一种用户 abort 机制，也就是用户认为该进程运行出错
- `KILL`，立即杀死进程，不能被捕获，忽略，阻塞，因此总是在 fatal 的情况下执行。
- `TERM`，默认行为是 terminate 进程，可以被捕获或者忽略，比较礼貌地终止，先让进程有机会清理。
- `STOP`，pause，不能被捕获或者忽略，使用 `CONT` 进行恢复
- `HUP`，hang up，报告用户终端断开连接，比如关闭终端， SSH 网络连接中断等原因。在关闭终端后，给终端对应的控制进程发送 SIGHUP，bash 收到 SIGHUP 之后，会给各个 Job(前后台)发送 SIGHUP，各个 Job 收到 bash 的 SIGHUP 之后默认退出(不捕获或者忽略)。



## 4. 外部存储

```shell
# mount 将一个设备挂载到文件树上，才能读写设备中的文件
mount -t type device dir             # type 为存储设备被格式化的文件系统类型
mount -t vfat /dev/sdb1 /media/disk  # 将 U 盘 /dev/sdb1 挂载到 /media/disk

umount dir | device 
```

```shell
df -Th      # 打印文件系统使用空间 -h 输出格式更友好一些，-T 打印文件系统类型

du -ch dir  # 显示 dir 目录下所有的文件、目录、子目录的磁盘使用情况
            # -c --total, -h --human-readable
```

```shell
# 压缩文件
# 压缩工具 bzip2: .bz2, compress: .Z, gzip: .gz, zip: .zip
# 最常用的工具是 tar
tar -cvf test.tar pathname   # 打包，不压缩，c: --create, v: --verbose, f: 指定归档文件
tar -tf test.tar             # 不提取文件，查看内容, t: --list
tar -xvf test.tar            # x: --eXtract

tar -zcvf test.tar.gz pathname  # 以 gzip 压缩
tar -jcvf test.tar.gz pathname  # 以 bzip2 压缩

# zip 命令
zip -r test.zip pathname        # zip 压缩, recursive
unzip [-d dir] test.zip         # 不指定路径直接解压到当前路径
```



## 5. shell

```shell
bash     # 使用 bash，其他类型的 shell 也类似，可以嵌套使用，互为父子关系
exit     # 退出 bash

# 用户默认 shell 类型，可以在 /etc/passwd 中查看，可以看到使用的是 bash
root:x:0:0:root:/root:/bin/bash
vczn:x:1000:1000::/home/vczn:/bin/bash
```

登录虚拟终端或者 GUI 中运行的终端仿真器时所启动的默认的交互 shell，是一个父 shell。当输入 `bash` 后，会创建一个新的 shell 程序，子 shell 也拥有 CLI 提示符，等待命令输入。可以使用 `ps -f` 命令查看。

```txt
UID        PID  PPID  C STIME TTY          TIME CMD
vczn      6439  6438  0 09:28 pts/0    00:00:00 -bash
vczn      6453  6439  0 09:28 pts/0    00:00:00 bash
vczn      6465  6453  0 09:28 pts/0    00:00:00 ps -f
```

可以看到 第二个 bash 的 PPID(父进程ID) 为 6439，正是第一个 bash 的 PID。

使用 `ps --forest` 可以更清晰地看出进程树关系。

```bash
pwd; ls; pwd    # 一行多个命令可以使用 ; 分隔
(pwd; ls; )     # 用 () 变成进程列表，生成一个子 shell 来执行对应地命令
                # 可以使用 (cmd; ps --forest) 来实验 或者 (cmd; echo $BASH_SUBSHELL)
```



Shell 有前台和后台任务(Job)。

```bash
sleep x         # 休眠 x 秒
sleep x &       # 加上 & 之后，就变成后台模式，

jobs [-l]       # 查看后台任务，l 显示更多内容(PID)
```

**协程**：一个协程在子 shell 中被异步执行，就类似于使用 `&` 操作符置入后台。在执行 shell 和 协程之间建立双向管道

> https://lujun9972.github.io/blog/2018/04/26/%E5%B0%8F%E8%AE%AEbash%E4%B8%AD%E7%9A%84coproc/

```shell
 # 基本语法，适用于只有一个协程的情况
coproc cmd [redirctions]    
# 输入输出管道相连的文件描述符保存在 $COPROC 数组中，分别为 ${COPROC[1]}, ${COPROC[0]}
# ??? 之后填坑

# 扩展语法，可以适用于多个协程的情况
coproc NAME { cmds; } [redirections]  # 在 { 和命令之间必须有空格，; 和 } 之间同样
# 输入输出管道相连的文件描述符保存在 $NAME 数组中
```

**外部命令和内建命令**

```shell
# 外部命令一般位于 /bin, /usr/bin, /sbin, /usr/sbin 中。
# 比如 ps 就是外部命令。当执行外部命令时，会 fork 出一个子进程，之后 exec ps 进程。
which ps    # 找到 ps 的位置
/bin/ps

type -a ps 
ps is /bin/ps

# 内建命令(built-in) 和 shell 编译为一体，不需要外部程序来运行。
type cd 
cd is a shell builtin

# 有一些命令有多种实现
type -a echo
echo is a shell builtin
echo is /bin/echo
```

```shell
# 一些内建命令
history                   # 显示输入命令的历史记录
alias [-p] newcmd='cmd'   # 命令别名，p 为打印出设置的别名
alias ll='ls -l'
ll
```



## 6. 重定向和管道

**重定向** 

```shell
cmd > outfile          # 将输出重定向到 outfile
cmd >> outfile         # 将输出以追加的方式重定向到 outfile
cmd n > file           # 将文件描述符为 n 的文件重定向到 file
cmd n >> file          # 将文件描述符为 n 的文件以追加的方式重定向到 file
cmd < infile           # 将输入重定向到 infile
cmd < infile > outfile # 两者合并使用
cmd > file n>&m        # 将输出文件文件描述符 n 和 m 合并重定向到 file
cmd >& n               # 将输出重定向到文件描述符为 n 的文件
cmd < file n<&m        # 将输入文件描述符 n 和 m 合并，重定向到 file
cmd << tag

./test > file 2>&1    # 将 stdout 和 stderr 重定向到 file， 2>&1 顺序不能反!!!

# cmd << tag
wc -l << end
aaa
bb
c
end
3        # 行数
```

**管道** 


```shell
command1 | command2  # 把 comamnd1 的 stdout 和 stderr 作为 command2 的 stdin
```



## 7. 环境变量
**局部环境变量**：只对创建它们的 shell 可见

```shell
set                    # 会输出全局变量、局部变量、用户定义的值
variableName=value     # 设置局部变量 variableName，不能有空格
                       # 如果要赋值一个含有空格的字符串，需要使用 '' 来界定字符串的首尾
echo $variableName
```

**全局环境变量**：对 shell 会话和所有生成的子 shell 都是可见的

```shell
printenv [SOME_ENV]    # 打印所有全局变量，也可以打印某个全局变量
env                    # 打印所有全局变量

printenv HOME
echo $HOME             # $HOME 为 HOME 变量的值，可以用在其他命令中

my_variable="I am Global now"
export my_variable
export variable2="xxx"

# Tip：系统全局环境变量使用大写字母，自己创建的局部变量或者 shell 可以使用小写字母。
#      能够避免重新定义系统环境环境所带来的灾难。
```

**删除环境变量**

```shell
unset my_variable      # 删除环境变量
```

**设置 `PATH` 环境变量**：执行外部命令时，依次在环境变量目录搜索命令。用 `:` 分隔

```shell
echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
PATH=$PATH:add_some_path_variable
```

**环境变量持久化**：

当登录 Linux 系统时，bash 会作为登录 shell 启动。会从下列的启动文件内读取命令，运行第一个被找到的文件，余下的被忽略

- `/etc/profile`
- `$HOME/.bash_profile`
- `$HOME/.bash_login`, 
- `$HOME/.profile`

上述文件没有 `$HOME/.bashrc`，因为该文件通过其他文件运行。对于所有用户的设置，可以直接修改 `/etc/profile`，但是由于升级的原因可能会改变。可以在 `/etc/profile.d` 目录建立一个 `.sh` 脚本文件，用于修改 `/etc/profile` 设置

对于个人用户的设置，大多数发行版可以修改 `$HOME/.bashrc` 文件

**数组变量**

```shell
myarray=(one two three four five)
echo $myarray 
one
echo ${myarray[2]}
three
echo ${myarray[*]}
one two three four five

myarray[2]=seven   # 修改某个索引位置的值
```



## 8. 文件权限

**用户权限**是通过创建用户时分配的 UID 来跟踪的。`/etc/passwd` 文件存储了用户相关信息。root 是 Linux 上的管理员，UID 为 0。Linux 系统为各种各样的功能创建不同的用户账户，但是这些不是真的用户，这些叫做系统账户，系统上运行各种服务进程访问资源用的特殊账户。防止被攻陷一个就能作为 root 用户进入系统。

Linux 的密码保存到另一个叫做 `/etc/shadow` 的文件中。只有特定的程序才能访问。直接修改 `/etc/passwd` 内容是极其危险的，可以用 Linux 的用户管理工具。

```shell
useradd [-m] username     # m 为创建同名目录
userdel [-r] username     # recursive
usermod [-alLUG]          # append, l 为修改用户账户的登录名，L 锁定无法登录，U 解锁，G groups

# 修改密码最好使用 passwd
passwd username
chapasswd < users.txt     # 批量修改密码，users.txt 中为若干 userid:passwd 行

chsh -s /bin/csh username # 修改登录 shell，-s: shell

finger username           # 查看某个用户信息
chfn username             # 修改用户备注信息，之后根据提示填写，使用 finger 可以看到修改的内容

chage                     # 管理用户账户有效期
```

**组权限**允许多个用户对系统中的对象共享一组共用的权限。同样，也有 `/etc/group` 文件。不要直接修改该文件，使用 `usermod` 来修改用户所属组。

```shell
groupadd gname            # 添加新组
usermod -G gname uname    # 将 uname 用户添加到 gname 组中，注销再登录组关系才能生效

groupmod -n newgname oldgname  # 修改组名
groupmod -g newgid oldgid      # 修改 gid

groupdel group
```

**默认文件权限**

```shell
umask
0222
```

`umask` 的第一位代表粘滞位(*sticky bit*)。后面三位为八进制数。`r` 代表 4，`w` 代表 2，`x` 代表 1，相加得到。

**改变权限**

```shell
chmod options mode file 
```

`chmod` 有两种风格，一种是字母，一种是数字。

字母：`mode: [ugoa][[+-=][rwxXstugo]]`

- `X`：如果对象是目录或者它已有执行权限，赋予执行权限 
- `s`：运行时重新设置UID或GID。设置 Set UID 和 Set GID 位。
- `t`：保留文件或目录。 
- `u/g/o`：将权限设置为跟属主/组/其他一样 



---

**关于 Set UID、Set GID、sticky bit 的详细介绍：**

**Set UID 和 Set GID**

```shell
chmod u+s exefile  # 设置 Set UID
chmod g+s exefile  # 设置 Set GID
```

如果一个文件被设置了 Set UID 或 Set GID 位，会分别表现在所有者或同组用户的权限的可执行位上。

如果设置了 Set UID 或者 Set GID 的话，进程获得了该文件的 UID 或者 GID。

这儿有一个实例使用 `passwd` 就是修改用户密码。`passwd` 需要修改 root 权限下的 `/etc/shadow` 和 `/etc/passwd` 文件，可是使用 `ls -l ...`  可以看到，他们的 user 为 root。那么如何在非 root 权限下修改密码呢？这时候就是 Set UID 起作用了。可以使用 `ls -l /usr/passwd` 可以看到设置了 Set UID 位。

```txt
-rwsr-xr-x 1 root root 59680 May 17  2017 /usr/bin/passwd
```
我们可以做一个实验来验证一下。

```cpp
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

#include <cerrno>
#include <cstring>
#include <iostream>

int main()
{
    std::cout << "uid = " << getuid() << std::endl;
    std::cout << "euid = " << geteuid() << std::endl;
    int fd = open("a.txt", O_WRONLY);
    const char hello[] = "hello world";
    int n = write(fd, hello, sizeof(hello));
    if (n != sizeof(hello))
    {
        std::cout << "write error: " << std::strerror(errno) << std::endl;
    }
  
    return 0;
}
```

编译好上述代码，假设生成 `a.out`，在 root 权限下，创建 `a.txt` 文件，其权限为 `-rw-r--r--`。

先不设置 Set UID，在非 root 用户权限下，执行 `a.out`，这时候 `write` 调用会发送错误：

```txt
uid = 1000
euid = 1000
write error: Bad file descriptor
```

现在在 `a.out` 上加上 Set UID 之后，成功写入文件并输出：

```txt
uid = 1000
euid = 0
```

**粘着位 sticky**

如果设置了 sticky 位的可执行程序向内核发送请求，当程序结束后，仍保留在内存。基于代码页共享有其他更好的方法。

还有就是使用在目录上，目录也是可执行文件。只有目录内文件的所有者或者 root 才可以删除或移动该文件。如果不为目录设置粘滞位，任何具有该目录写和执行权限的用户都可以删除和移动其中的文件。在我们系统中，粘滞位一般用于 `/tmp` 目录，以防止普通用户删除或移动其他用户的文件。

 一个目录具有粘滞位，则在 other 的 X 位会表现为 t/T。大小写的区别在于，原来 x 位上有 x 权限，有了粘滞位则表现为 t。否则，表现为 T。

  ```shell
  chmod +t filename
  ```

---



**改变所属关系**

```shell
chown [options] owner[.group] filename 
chgrp newGroupName file    # 当前用户必须是旧属组和新属组的成员
```



## 9. 管理文件系统

文件系统为硬盘中存储的 0,1 和应用中使用的文件与目录之间搭起一座桥梁。

**Linux 文件系统的演进**

- ext(*extended filesystem*)。类 Unix，虚拟目录操作设备，物理设备按定长块存储数据。inode 存放虚拟目录中所存储的文件信息。而且会跟踪每个文件额外的(extended)信息。
- ext2。扩展了索引节点表的格式来保存更多文件信息，最大文件增加到 2TB(后期 32 TB)。改变了文件在数据块中的存储方式，避免碎片化。
- ext3。每个存储设备增加了一个日志文件，默认使用有序模式[^1]，可以使用命令来改变模式。
- ext4。支持数据压缩和加密，支持区段(*extent*)的特性，区段在存储设备上按块分配空间，在 inode 表中只保存起始块的位置。还引入了块预分配技术。
- 写时复制(*COW*)利用快照兼顾安全和性能。比如 ZFS、Btrf
- 其他日志文件系统比如 Reiser、JFS、XFS 不加赘述。

---

[^1]: Linux 文件系统日志方法。

|   方法   |                             描述                             |
| :------: | :----------------------------------------------------------: |
| 数据模式 |    索引节点和文件都会被写入日志，丢失数据风险低，性能差。    |
| 有序模式 | 只有索引节点被写入日志，但只有数据写入成功后才删除，性能和安全折中。 |
| 回写模式 | 只有索引节点被写入日志，但不控制文件数据何时写入，丢失数据风险高。 |

---



**创建分区**：创建分区来容纳文件系统

```shell
# 创建分区
sudo fdisk [options] /dev/sdb    # 需要超级用户权限，无特殊 option 进入命令模式
sudo fdisk -l [device]           # 为某个 device 显示分区表，默认为 /proc/partitions

# 命令(m 查看帮助)：d 删除分区，n 添加新分区，p 查看当前分区表，w 将分区表写入磁盘
# 比如添加分区
sudo fdisk /dev/sdb
Command (m for help): n
Command action
   e extended
   p primary partition (1-4)
p                                      # 选择主分区
Partition number (1-4): 1              # 分区号
First cylinder (1-652, default 1): 1   # 柱面
Last cylinder, +cylinders or +size{K,M,G} (1-652, default 652): +2G # 容量

# 之后查看分区，看到多了 sdb1
Device Boot Start End Blocks Id System
/dev/sdb1 1 262 2104483+ 83 Linux

# 改变分区表
Command (m for help): w  
The partition table has been altered!
```

主分区(最大 4 个)可以被文件系统直接格式化，而扩展分区容纳逻辑分区。



**创建文件系统**：在数据存储到分区之前，必须用某种文件系统对其进行格式化，这样 Linux 才能使用它。

```txt
mkefs: ext
mke2fs: ext2
mkfs.ext3: ext3
mkfs.ext4: ext4
```

上述命令不一定默认安装。

```shell
sudo mkfs.ext4 /dev/sdb1   # 在 sdb1 分区创建 ext4 文件系统

# 挂载到虚拟目录的某个挂载点就可以使用啦
sudo mkdir /mnt/my_partition   # 创建自定义分区目录
sudo mount -t ext4 /dev/sdb1 /mnt/my_partition  # 挂临时载
# 如果需要添加到 /etc/fstab
```

**文件系统的检测和修复**

```shell
fsck [-as] [-r [fd]]  # a:自动修复文件系统，s:依次检测多个文件系统，r:出现错误提示
                      # 只能在为挂载的系统运行 fsck 
```

**逻辑卷管理 LVM**：将物理卷抽象到更高的层次，方便管理硬盘。

基本概念：硬盘分区被作为**物理卷(PV)**，多个物理卷组成一个**卷组(VG)**，逻辑卷管理系统将卷组视为一个物理硬盘。最上一层是**逻辑层(LV)**。逻辑卷为 Linux 提供了创建文件系统的分区环境。

```txt
+-----------------------------+----------------------------------+
|       logic volume 1        |          logic volume 2          |
+-----------------------------+----------------------------------+
|                           volume group                         |
+----------------------------------------------------------------+
|   phy 1    |   phy 2    |   phy 3    |   phy 4    |   phy 5    |
+------------+------------+------------+------------+------------+------------+
|   part 1   |   part 2   |   part 3   |   part 4   |   part 5   |   unused   |
+------------+------------+------------+------------+------------+------------+
|         disk 1          |         disk 2          |        disk 3           |
+-------------------------+-------------------------+-------------------------+
```

**快照**允许在逻辑卷在线的情况下，将其复制到另一个设备。

**条带化**可跨多个物理设备创建逻辑卷。可以提高磁盘性能，同时读多写个磁盘。

**镜像**功能可以随时创建逻辑卷的副本，当创建镜像逻辑卷时，LVM 将原始逻辑卷同步到镜像副本中。



```shell
## 使用 Linux LVM
# 创建物理卷, 带 ** 为用户输入
sudo fdisk /dev/sdb1
Command (m for help): *t*
Hex code (type L to list codes): *8e*  # 8e 为 Linux LVM
Changed system type of partition 1 to 8e (Linux LVM)
Command (m for help): *w*   
The partition table has been altered!

sudo pvcreate /dev/sdb1   # 创建实际的物理卷
sudo pvdisplay /dev/sdb1  # 显示创建的物理卷

# 创建卷组
sudo vgcreate Vol1 /dev/sdb1
sudo vgdisplay Vol1 

# 创建逻辑卷，Linux 可以使用逻辑卷来模拟物理卷，并在其中保存文件系统
sudo lvcreate -l 100%FREE -n lvtest Vol1  # l 区段数或百分比，n 命名
sudo lvdisplay Vol1

# 运行文件系统
sudo mkfs.ext4 /dev/Vol1/lvtest
sudo mount /dev/Vol1/lvtest /mnt/my_partition

# 修改 LVM
vgchange/vgremove/vgextend/vgreduce
lvextend/lvreduce
```



## 10. 安装程序

### (1) Debian 包

```shell
sudo apt-get install xxx
sudo apt-get update
sudo apt-get remove --purge xxx
sudo apt-get remove xxx

sudo apt-get -f install   # 解决依赖问题

sudo dpkg -i xxx.deb      # install
sudo dpkg -L xxx.deb      # 列出 xxx 软件包所安装的全部软件

sudo aptitude
sudo aptitude show packageName
sudo aptitude search packageName  # 前面 i 表示已安装，p/v 表示包可用，但没安装
sudo aptitude install packageName
sudo aptitude safe-upgrade        # full-upgrade/dist-upgrade 不管依赖可能出问题
sudo aptitude purge xxx

# 软件源配置文件在 /etc/apt/sources.list
```

### (2) RedHat 包

```shell
yum list installed      # 列出已经安装包
yum list xxx            # 列出 xxx 包的详细信息
yum list installed xxx  # 查看 xxx 是否被安装
yum provides fileName   # 查看某个文件属于哪个软件包

yum install xxx
yum localinstall xxx.rpm

yum [list] updates [xxx]   # list 为列出

yum remove xxx             # 只删除软件，保留配置文件和数据文件
yum erase xxx              # 删除软件和它的所有文件

yum clean all              # 解决包依赖问题，之后 update。不行的话见下一行
yum deplist xxx            # 列出 xxx 的依赖包
yum update --skip-broken   # 跳过依赖关系损坏的包

yum repolist               # 查看使用的软件仓库
# 软件源配置文件在 /etc/yum.repos.d
```

### (3) 编译安装

下载源码，按照文档进行编译安装。



## 11. 编辑器

### (1) Vim

```vim
# 常用命令
h: left; j: right; k: up; l: down;
PageDown(Ctrl+F): 下翻; PageUp(Ctrl+B): 上翻;
G: 移动到行尾; num G 移动到第 num 行; gg 移动到缓冲区第一行;
x: 删除光标处字符; 
[num] dd: 从当前行开始删除 num 行，默认 1; dw: 删除光标位置单词; d$: 删除光标至行尾; # 其实是剪切
J: 拼接下一行; 
u: 撤销;
a: 在当前光标后追加数据; A 在当前光标所在行行尾追加数据;
r char: 替代单个字符; R text: 替代光标位置数据，直至按下 ESC 键
[num]yy: 从当前行开始开始复制 num 行，默认为 1; yw; y$;
p: 粘贴;
v: 进入 visual 模式，可以使用 y 键复制高亮文本;
/: 前向查找; n 查找 next;
:[n,m/]s/old/new[/g]: 使用 new 替换 old，[n,m]为替换行 n 和 m 之间，[/g] 为替换所有
```

```vim
# 分屏
vim -On file1 file2 ... # 垂直分屏, n 是数字，O 为大写字母
vim -on file1 file2 ... # 水平分屏 

ctrl+w c                # 关闭当前分屏
ctrl+W q                # 关闭当前分屏；如果是最后一个分屏，退出

ctrl+w s                # 上下分割当前打开的文件
ctrl+w v                # 左右分割当前打开的文件

:sp filename            # 上下分割打开新文件
:vsp filename           # 左右分割打开新文件

ctrl+w h                # 光标移动到左边的分屏
ctrl+w j                # 光标移动到下边的分屏
ctrl+W k                # 光标移动到上边的分屏
ctrl+w l                # 光标移动到右边的分屏
ctrl+w w                # 光标移动到下一个分屏

ctrl+w H/J/K/L          # 分屏向左/下/上/右移动

ctrl+w =                # 所有分屏相同高度
ctrl+w +/-              # 增加减少高度

ctrl+hjkl
```



### (2) nano 

nano 比较简单，说明在主窗口下。

### (3) 其他

emacs 和图形编辑器(gedit, atom, sublime, vscode)就不再介绍



## 12. 编译相关

C/C++ 编译，这里主要写 C++ 简单的编译命令

```sh
g++/clang++
-c            # 生成 object 文件
-C            # 预处理，但删除注释信息
-Dmacro       # define macro
-Dmacro=defn 
-E            # 预处理(cpp)
-g            # debug 信息，在 gdb, objdump 时用到这些信息
-Idir         # 头文件目录
-l`lib`       # link 名为 lib 的 lib 文件，一般为 `*.a`
-Ldir         # 库文件目录
-o outfile    # 输出文件名
-O?           # ? 为优化等级
-shared       # 生成共享 .so 文件，常和 -fPIC 一起使用 
-static       # 默认优先使用动态链接库，强制使用静态链接库
-S            # 生成汇编代码
-Wall         # 所有 warning
-Werror       # warning 视为 error
```



## 13. 网络相关

```shell
# 网络查看及配置
ifconfig                 # 查询网卡配置信息
ifconfig eth0 up         # 启动网卡 eth0
ifconfig eth0 down       # 关闭网卡 eth0，远程登录需要注意

ifconfig eth0 192.168.1.110 [netmask 255.255.255.0]  # 暂时修改
# 其他还能配置和删除 ipv6，修改 MAC 地址(修改 arp 缓存表的 MAC 地址，网卡上的 MAC 写死在硬件)
# 启用或关闭 arp 协议，设置 MTU 等

ifup eth0
ifdown eth0

route                    # 显示路由表
# flags: U(Up)，H(Host)，G(Gateway)，!(Closed)
# R(Reinstate 使用动态路由重新初始化的路由), D(Dynamically 此路由是动态地写入)
# M(Modified 此路由是路由守护进程或导向器动态修改)
route add/del [-net/-host/default gw] someIP [netmask someMask]

netstat [-autxlpnv]  # all, udp, tcp, unix, listening, program, numberic, verbose
                     # 一般和 grep 等命令配合使用
                     
# 永久修改可以修改 /etc/network/interface 文件，之后重启网络功能
sudo /etc/init.d/networking restart
sudo service network restart

# hostname 文件在 /etc/hostname，Debian 发行版
hostname # 显示 hostname
```

```shell
ping host/ip
traceroute 
```

```shell
wget url       # network download，简单

curl url       # 主要在模拟 web，支持上传，下载，POST，GET，cookie
# curl 常用 options
# -G/default   # GET
# -o file/-O   # 保存到文件而不是 stdout，-O 为默认名称
# -d data      # POST
# -b cookie    # cookie
# -I           # --head
# -C           # continue，断点续传
```

```shell
tcpdump options filter       # 抓包
# 常用参数
# -c: count, -F file 使用文件作为输入过滤器, -i: interface, -ttt 打印时间间隔(微秒分辨率)
# -w: 输出到文件，方便之后用 wireshark 等工具分析

# 抓取的信息格式一般为 
# timestamp IP/IP6 src > dst: Flags [.] ack win options length
# 参考文章：https://linuxwiki.github.io/NetTools/tcpdump.html
```



## 14. GDB

```gd
help
quit(q)
run(r)
kill(k)

break(b) function_name 
break(b) *address

delete(d) 1      # delete breakpoint 1
delete(d)        # delete all breakpoints

stepi(si)
stepi(si) n      # 执行 n 条指令
nexti(ni)        # 以函数调用为单位
continue(c)      # 继续执行
finish           # 运行到当前函数结束
disassemble(disas)    
disassemble(disas) function_name 
disassemble(disas) address
disassemble(disas) [begin, end]

print(p) /x $RIP   # 16 进制打印 RIP
print(p) $RAX      # 10 进制打印 RAX
print(p) var/*addr # 打印值

x/nfu addr         # print memory value
                   # n instructions
                   # f format, similar to printf format
                   # u unit(b/h/w/g)
                   
info(i) frame(f) 
info(i) registers(r)
```





## 15. 其他

```shell
who                          # 查看登录用户
mesg [y/n]                   # 查看/启用/禁用 向其他登录用户发生消息
write username ttyName       # 双方启用，发送消息
```

```sh
uname -r         # 查看内核版本
```





# 三、shell 脚本

## 1. 基本脚本

```sh
#!/bin/bash
# 显示消息
echo hello world
echo "\"xxx\"'yyy'"   # "xxx"'yyy'，如果只出现一种可以使用另外一种引号

# 使用变量
echo UID: $UID        # 想输出 '$' 符号，需要 \$ 转义
var1=42               # = 前后不能出现空格
var2=testing          # 变量定义的生命周期为脚本运行到脚本结束
var3="less is more"
echo $var1

# 命令替换，将命令输出赋给变量
var4=`date`
var4=$(date)

# --- log file per day ---
today=$(date +%y%m%d)
touch log.$today
# --- log file per day ---
```

```shell
chmod u+x test.sh
./tesh.sh
```



**数学运算** 

```shell
# expr 数值运算，|,&,<,<=,=,!=,>=,>,+,-,*(使用需要 \ 转义),/,%
expr 1 + 5    # 必须加空格，不加空格判断为字符串
var1=$[1+5]   # 使用方括号来计算数学表达式，而且不用担心 * 通配符，只支持整型

# expr 命令对字符串的操作
expr length STR                # 返回 STRING 长度，""''，需要转义，空格也可以转义，也可以引号
expr substr STR [POS] [LENGTH] # 从 pos(下标 1 开始) 开始，长度为 length 的子串
expr index STR CHARS           # 返回找到找到 CHARS 第一个位置的 index
expr STR : REGEXP              # 如果 REGEXP 匹配到 STRING 中的某个模式匹配，返回该模式匹配
expr match STR REGEXP          # 同 : 
```

```shell
# bc: bash calculator，可以输入浮点表达式
# 可以识别整型和浮点型、变量、注释、表达式、编程语句(if-then, ...)，函数
# 浮点数结果，部分运算(除法、取余、幂)由变量 scale 控制，默认为 0
$ bc
3.44 / 5      # 0
scale=3       
3.44 / 5      # .688
quit

# 在 shell 脚本中使用 
var1=$(echo "scale=4; 3.44/5" | bc)
var1=`echo "scale=4; 3.44/5" | bc`
```



**退出 shell 脚本**

```shell
# Linux 脚本提供了 $? 来保存上个命令的退出状态码
# 0 成功，1 一般性未知错误，2 不适合 shell 命令
# 126 命令不可执行，127 没找到命令，128 无效地退出参数
# 128+x，Linux 信号 x 相关地严重错误，比如 130 Ctrl+C，255 正常范围之外的退出状态码

exit num      # 写在 script 脚本中，可以显式指定退出码，退出码会 % 256
./scricpt
echo $?       # 显示 script 的状态码
```



## 2. 结构化命令

**if**

```shell
# if-elif-then
if command1        # 如果退出状态码为 0，执行 then
then 
	commands1
elif command2      # 可以放置多个 elif
then
	commands2
else
	commands3
fi

# 另外一种形式
if command; then
	commands
fi
```

```shell
if test condition or [ condition ]  # [ xxx ] xxx 前后的空格不能省略
# 关于 condition
## int: n1 -op n2, op 可以为 eq, ge, gt, le, lt, ne

## string: [str1] op str2, op 可以为 ==, !=, <, >, -n(一元，长度非0), -z(一元，长度为0)
##     !!! 需要注意 > 和 < 在使用时需要转义，会被认为成重定向符号，比较根据 ASCII 码顺序

## file: 
## -d(directory)/-e(exist)/-f(file)/-s(space i.e. not empty)/
## -r(readable)/-w(writable)/-x(execute)/
## -O(own same as current user)/-G(group same as current user)
## file1 -nt/-ot file2  检查 file1 是否比 file2 新/旧

# 复合条件
[ condition1 ] &&/|| [ condition2 ]
              and/or
              -o/-a

# 高级特性
(( expersion ))  # 类似于 C，val++,val--,++val,--val,!,~,**,<<,>>,&,|,&&,||,comp
[[ expersion ]]  # 字符串的正则表达式模式匹配，比较符号都无需转义
```

**case**

```shell
case variable in 
pattern1 | pattern2) commands1;;  # 两个条件都是这个分支
pattern3) commands2;;             # 这个分支只有一个条件
*) default commands;;             # 默认分支
esac
```

**for**

```shell
# foreach
for var in list   # list 例子: one two three four, var 作用域不仅在 for
do                # 空格分隔，引号需要转义，如果包含空格数据值，需要引号
	commands
done

list="one two three four"
list=$list" five"      # 连接字符串
for var in $list       # 从变量读写列表
for car in $(cat file) # 从命令读写列表

## 更改列表分隔符，默认为 空格、\t、\n
IFS=$'\n'              # 更改列表分隔符为 \n，忽略空格和 \t，更改之前建议备份

## 通配符
for file in /path/*
do
	if [ -d "$file" ]  # 需要 "$file"，虽然 Linux 不允许有空格的文件，以防其他文件
	...
	fi
done

# for
for (( i = 1; i < 10; i++ ))  # assign 可以有空格，多个用 `,` 隔开
                              # 变量不以 $ 开头，迭代过程未用 expr
```

**while**

```shell
while test command 
	[ condition ]    # 可以使用多个测试命令，每个占一行，最后一个状态码被作为测试条件，可以省略
do
	...
done
```

**until**

```shell
until test command
do 
	commands
done
```

**break/continue**

```shell
break [n]     # 默认跳出一层循环，可以加 n，n 表示跳出 n 层循环
continue [n]  # 类似 break
```



**实例** 

```shell
# 查找可执行文件
IFS=:
for dir in $PATH
do
    echo "$dir:"
    for file in $dir/*
    do
        if [ -x $file ]
        then
            echo "  $file"
        fi
    done
done
```

```sh
# 创建多个用户账户
# 账户文本文件 uid, username 
input="user.csv"
while IFS=',' read -r userid name
do 
	echo "adding $userid"
	useradd -c "$name" -m $userid
done < "$input"
```



## 3. 处理用户输入

**命令行参数**

```shell
$0                           # 脚本名
$1, $2, ..., ${10}, ...      # 命令行参数
$#                           # 命令行参数数量
${!#}                        # 最后一个参数的值，在 {} 不能使用 $
$*                           # 一个包含所有变量的单个变量
$@                           # 所有参数当作同一字符串中多个独立的单词，也就可以用 foreach

# tip: 在使用命令行参数可以先判断是否存在
if [ -n "$1" ]
then 
	...
fi

# 命令行参数左移
shift    # $0 不变， $1 被抛弃，$2 移到 $1，$3 移到 $2，...

# 处理简单选项
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option";;
        -b) param="$2"
            echo "Found the -b option with parameter value $param"
            shift ;;
        --) shift
            break ;;
         *) echo "$1 is not an option"
     esac
     shift
 done

# 使用 getopt 处理选项
getopt optstring parameters # optstring 为单个字符，如果有值，需要后加 :
                            # 比如 ab:cd -a -b test -cd test2 test3
# 在脚本中使用 getopt 生成格式化后的版本替换已有的命令行参数
set -- $(getopt ab:cd "$@") # 解析命令行参数，替换原先的 $@，之后处理简单的命令行选项就可以了
./script.sh -ab test -c -d -e test2
getopt: invalid option -- 'e'
-a -b test -c -d -- test2   # echo $@

# getopts 注意加了 s，一次处理一个参数，处理完所有参数，会返回大于 0 的退出码
while getopts ab:c opt
do
    case "$opt" in
        a) echo "Found the -a option";;
        b) echo "Found the -b option with index:value $OPTIND:$OPTARG";;
        *) echo "Unknown option: $opt"
    esac
done
```

**用户 stdin**

```shell
read name                            # read 命令将数据放入 name 中
read -t 5 -p "enter your name: "     # -t 5s -p 提示，过期会返回一个非 0 状态码
read -n1 -p "continue? [y/s]" answer # 读一个字符
read -s -p "enter your password: "   # 隐藏输入字符
cat file | while read line           # 从文件读取
```



## 4. 呈现数据

**在脚本中重定向**

```shell
echo "This is an error message" >&2  # 临时将输出信息重定向到 STDERR
                                     # 在数字前加 & 表示 fd，而不是 filename
exec 1>testout                       # exec 在脚本执行期间文件描述符重定向到某个文件
exec 2>testerr

exec 0<testfile                      # 在脚本中重定向输入
```

**创建自己的重定向**

```shell
# shell 脚本中，最多有 9 个打开的文件描述符
exec 3> test3out        # 使用 exec 3>>test3out 将会 append
echo "fd3" >&3          # 重定向到 fd 3

exec 3>&1               # 恢复，将已重定向的 fd 重定向到 stdout 即可

exec 6<&0               # 输入
exec 3<>testfile        # 读写
exec 3>&-               # 关闭 fd3，关闭之后再写入 shell 会产生错误信息
```

**列出打开的文件描述符**

```shell
lsof                    # 列出所有进程打开的文件描述符
lsof -a -p $$ -d 0,1,2  # a 对其他选项结果进行 and 操作，p 表示进程号，$$ 是当前进程，d: fd
```

**阻止命令输出**

```shell
cmd > /dev/null    
```

**创建临时文件**

```shell
mktemp testing.XXXXXX   # XXXXXX 类似于模板，会随机自动填充，保证唯一
mktemp -t testing.XXX   # 在 /tmp/ 目录下，创建临时文件
mktemp -d testdir.XXX   # 创建临时目录而不是文件

tempfile=$(mktemp testing.XXXXXX)
exec 3>$tempfile        # 在 shell 可以这样使用
```

**记录消息**

```shell
cmd | tee testfile  # 相当于一个 T 型接口，将从 STDIN 过来的数据发向 STDOUT 和 指定的文件
tee -a              # append
```

**实例**

```shell
# 从 csv 文件读取数据，输出为 sql 语句文件，之后直接执行 sql 文件
outfile="members.sql"
IFS=','
while read lname fname address city state zip
do 
	cat >> $outfile << EOF    # 输入碰到 EOF 结束，之后将 cat 输出重定向到 $outfile
	INSERT INTO members(lname,fname,address,city,state,zip) VALUES
		('$lname','$fname','$address','$city','$state','$zip')
	EOF
done < ${1}  #${1} 为读取数据的 csv 文件名
```



## 5. 控制脚本

**处理信号**

```shell
# bash 生成信号
# Ctrl+C: SIGINT
# Ctrl+Z: SIGSTOP
kill -SIGNO PID
killall -SIGNO PName
```

```shell
# shell 捕获信号
trap commands signals
trap "echo ' Sorry! I hava trapped Ctrl+C'" SIGINT
trap "echo 'Googbye...'" EXIT

# 修改已设置的捕获直接覆盖就行
trap -- SIGINT      # 移除已设置的捕获
```

**后台运行**

后台运行的脚本仍然会在终端显示 STDOUT 和 STDERR 消息，最后进行重定向，避免杂乱的输出。

```shell
# 可以在非控制台下运行脚本，使用 nohup，忽视 SIGHUB 
nohup ./script.sh &    
```

**作业控制**

```shell
jobs [-lnrs]   # l 列出 PID 和 JobID，n 只列出状态改变的作业，r: running, s: stoping
bg [jobNum]    # 将 job 置为后台模式运行，如果多个 job 需要指定作业号
```

**调度谦让度**

最低值 -20 是最高优先级，最高值 19 是最低优先级。默认情况，以优先级 0 来启动进程。

```shell
nice -n 10 cmd         # 让命令以更低的优先级运行，负数为提高优先度
ps -o pid,ppid,ni,cmd  # 查看谦让度 NI 值

renice -n 10 -p pid    # 设置 pid 进程谦让度为 10 
```

**定时运行作业**

```shell
# at 命令将 Job 提交到队列中，指定 shell 如何运行该作业。at 的 守护进程 atd 以后台模式运行。
at [-M -f shellFileName] time   # 默认为 STDIN，可以指定 -f 来读取命令文件; M 屏蔽输出
# time 支持多种时间格式
# 10:15, 10:15 PM(加不加空格都行), now,noon,midnight,teatime(4PM)
# MMDDYY,MM/DD/YY,DD.MM.YY
# now+25min, tomorrow, 10:15+7day

atq         # 列出正在等待的作业
atrm num    # 删除正在等待的作业
```

```shell
# cron 定期执行作业
#                                 可以用 mon 之类
# 时间表 min hour dayofmonth month dayofweek command
        15  10   *          *     *         command  # 每天 10:15 
        15  16   *          *     1         command  # 每周一 16:15
        
# command 必须是命令或者脚本的绝对路径

crontab -l     # 显示已有的 cron 时间表
```

```shell
# 当关机后，cron 的任务不再执行，可以使用 anacron，开机后会尽快执行错过的任务
period delay id command
```



# 四、高级 shell 脚本

## 1. 函数

```shell
# way 1
function name {
    commands
}

# way 2
name() {
    commands
}

# call
name
```

**返回值** 

```shell
# function 会被当成一个小型脚本，结束会返回一个退出状态码
func1() {
    ls -l badfile
}

echo "The exit status is: $?"

# 使用 return，退出状态码必须是 0~255
function func {
    read -p "Enter a value: " value
    echo "double the value "
    return $[ $value * 2 ]
}

func
echo "The new value is $?"  

# 使用函数输出
function func {
    read -p "Enter a value: " value
    echo $[ $value * 2 ]
}

result=$(func)
echo "The new value is $result"
```

**传递参数**

```shell
function add {
    if [ $# -ne 2 ]
    then
        echo "input error"
    else
        echo $[ $1 + $2 ]   
    fi
}

value=$(add $1 $2)    # call，传参
echo $value
```

**在函数中处理变量**

```shell
# 在函数中有 全局变量 和 局部变量
# 全局变量
function double1 {
    value=$[ $value * 2]
}
read -p "Enter a value: " value
double1
echo "The new value is: $value"

# 局部变量
function func1 {
    local tmp=$[ $value + 5 ]
    result=$[ $tmp * 2 ]
}

# 向函数传递数组
function testArray {
    local newarray
    newarray=(`echo "$@"`)
    echo "The new array value is: ${newarray[*]}"
}

myarray=(1 2 3 4 5)
echo "The origin array is: ${myarray[*]}"
testArray ${myarray[*]}

# 函数返回数组
function func1 {
    ...
    echo "${newarray[*]}"
}

result=$(func1 ${myarray[*]})
```

**函数递归**

```shell
# factorial.sh
function factorial {
    if [ $1 -eq 1 ]
    then
        echo 1
    else
        local tmp=$[ $1 - 1 ]
        local result=`factorial $tmp`
        echo $[ $result * $1 ]
    fi
}

result=`factorial $1`
echo "The factorial of $value is: $result"
```

**创建库**

```shell
# 在使用库的文件中添加需要包含的函数库，需要 source 命令
. ./factorial.sh        # source 的 dot operator，该脚本和 factorial.sh 位于同一目录
                        # 使用之后可以在命令行使用函数
```

可以在命令行创建函数，而且可以在 bashrc 中定义函数，bashrc 从文件读取文件

**实例**

```shell
# shtool 提供了一些简单的开源脚本函数，安装之后使用
shtool [options] [function [options] [args]]
```



## 2. 图形化中的脚本

**文本菜单**

```shell
function diskspace {
    clear
    df -k
}

function whoseon {
    clear
    who
}

function memusage {
    clear
    cat /proc/meminfo
}

function menu {
    clear
    echo
    echo -e "\t\t\tSys Admin Menu\n"
    echo -e "\t1. Display disk space"
    echo -e "\t2. Display logged on users"
    echo -e "\t3. Display memory usage"
    echo -e "\t0. Exit\n\n"
    echo -en "\t\tEnter option: "
    read -n 1 option
}

while [ 1 ]
do
    menu
    case $option in
    0)
        break ;;
    1)
        diskspace ;;
    2)
        whoseon ;;
    3)
        memusage ;;
    *)
        clear
        echo "Wrong selection" ;;
    esac
    echo -en "\n\n\t\t\tPress any key to continue"
    read -n 1 line
done
clear
```

**使用 `select `**

```shell
function diskspace {
    clear
    df -k
}

function whoseon {
    clear
    who
}

function memusage {
    clear
    cat /proc/meminfo
}

select option in "Display disk space" "Display logged on users" "Display memory usage" "Exit"
do
    case $option in
    "Exit")
        break ;;
    "Display disk space")
        diskspace ;;
    "Display logged on users")
        whoseon ;;
    "Display memory usage")
        memusage ;;
    *)
        clear
        echo "wrong selection" ;;
    esac
done
clear
```



**制作窗口**

```shell
# 在纯文本下可用，包含许多控件
dialog --widget parameters
dialog --msgbox text height width
dialog --title titleText
dialog --yesno text height width  # yes $? 为 0
dialog --inputbox text height width # 通过 stderr 输出，需要将 stderr 重定向到文件
dialog --textbox filename height width # 可滚动文本窗口
dialog --menu title height width numShowMenu tag1 tag1msg tag2 tag2msg ... # stderr
dialog --fselect path height width # stderr
```

**图形**

KDE 可以使用 kdialog

Gnome 可以使用 gdialog，zenity



## 3. sed 编辑器

sed 是流编辑器(*stream editor*) ，流编辑器会在编辑器处理之前预先定义一组规则来编辑数据流。

sed 编辑器能够执行下列操作：

- 一次从输入中读取一行数据；
- 根据所提供的编辑器命令匹配数据； 
- 根据命令修改流中的数据
- 将新的数据输出到 `stdout`

```shell
sed [-e script -f file -n] script inputfile   
# -e -f 会将 script 和 file 产生的命令添加到已有命令中
# -n 不产生命令输出，使用 print 命令来完成输出

sed -e 's/brown/green/; s/dog/cat/'        # 多个命令使用 -e
sed -f script1.sed data1.txt               # 从文件读取编辑命令
```

```shell
# 替换
sed 's/pattern/replacement/flags' # flags 可以是 n，第几个；g 全部；p 打印已替换；w file 写
sed 's/dog/cat/' data1.txt         # 不改变原文件，仅仅 replace 每行第一个
sed 's!/bin/bash!/bin/csh!' /etc/passwd  # ! 作为字符串分隔符

## 行寻址，将命令作用于特定行
[address] command
address {
    commands
}
### 数字寻址
sed '2s/dog/cat/' data.txt
sed '2,3s/dog/cat/' data.txt         # 从 2 ~ 3 行
sed '2,$s/dog/cat/' data.txt         # 从 2 ~ last line($)
### 文本模式过滤器
sed '/pattern/command'               # 采用正则表达式模式匹配
sed '/vczn/s/bash/csh' /etc/passwd   # 匹配 vczn 的行

# 删除，同样能用行寻址
sed 'd' data.txt                     # empty
sed '3d' data.txt                    # delete the line 3
sed '3,$d' data.txt                  # delete the third line to the last line
sed 'test 3' data.txt                # delete the lines of matching the 'test 3' 

# insert，在指定行前增加新行。append，在指定行之后附加新行
echo "Test Line 2" | sed 'i\Test Line 1'
echo "Test Line 2" | sed 'a\Test Line 1'
sed '3a\This is a new line of text.' data.txt

# 修改行
sed '3c\This is changed line of text.' data.txt  
sed '2,3c\This is changed line of text.' data.txt   # 2~3 替换为一行

# 转换命令 y，唯一可以处理单个字符 [address]/inchars/outchars
sed 'y/123/789' data.txt      # 转换所有 1->7，2->8, 3->9

# 打印，最常用的是模式匹配
sed -n '/pattern/p' data.txt
sed '=' data.txt              # 打印行号
sed -n 'l' data.txt           # 会打印不可打印的 ASCII，比如 制表符打印为 \t

# 读写
sed '[address]w filename'
sed '[address]r filename' data.txt  # filename 的数据被插入到 data.txt 匹配行之后
```



## 4. gawk 编辑器

gawk 不只是编辑器命令，提供了一种编程语言。可以做到：

- 定义变量来保存数据
- 使用算术和字符串操作来处理数据
- 使用结构化编程(if,loop)
- 提取数据文件中的数据元素，重排或者格式化，生成格式化报告

```shell
gawk [-F sep -f file -v var=value -mf maxField -mr maxrow -W warnLevel] program_file
gawk '{print "hello"}'

# 特殊数据字段 $0 代表整行，$1 代表文本行第一个数据字段，$2 ...
gawk '{print $1}' data.txt                 # 输出每行第一个数据字段
gawk '{$4="testing"; print $0}' data.txt   # 不改变原文件
gawk 'BEGIN {s1} {s2} END {s3}' data       # 先执行 s1，之后为 s2 处理数据，最后 s3
```



## 5. 正则表达式

Linux 有两种流行的正则表达式引擎：POSIX basic regular expression(BRE) 和 extended regular express(ERE)

不详细写每个符号，具体见 [正则表达式(zh)](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)，[正则表达式(En)](https://en.wikipedia.org/wiki/Regular_expression)

[正则表达式测试](https://www.regexpal.com/)

**BRE**：

纯文本，`^ $ . * [] [^] [m-n]`，特殊的字符组 比如 `[[:alpha:]]`

**ERE**

`? + {m,n} | ()group`



```shell
grep -Eo 'pattern' data.txt               # Extended, output
sed -n -r '/pattern/p' data.txt           # regular
gawk --re-interval '/pattern/{print $0}'
```



**实战**

```shell
# 匹配电话号码，美国电话格式 (123)456-7890 (123) 456-7890 123-456-7890 123.456.7890
# 区号 以2~9开头-
^\(?[2-9][0-9]{2}\)?    # 匹配区号
(| |-|\.)               # 后分组，有无空格，空格，- 和 . 四种情况
[0-9]{3}                # 三位交换机
( |-|\.)
[0-9]{4}$               # 匹配四位本机号码

gawk --re-interval '/^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}( |-|\.)[0-9]{4}/{print $0}'

# 个人认为有个问题是能够匹配到 (321)-456-7890，一个一个匹配尽量合并比较好
'^\([2-9][0-9]{2}\)[0-9]{3}-\[0-9]{4}|\([2-9][0-9]{2}\) [0-9]{3}-[0-9]{4}|([2-9][0-9]{2}(-|\.)){2}[0-9]{4}$'

# 需要注意 Linux 上不能使用 \d 代替 [0-9]
```

```shell
# 匹配 Email username@hostname
# username 可以是 .-+_alnum, hostname 可以是 ._alnum
# 顶级域名只能是字母，2~5个字符
^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\.]+)\.([a-zA-Z]{2,5})
```



# 五、有用的实例

## 1. 归档

```shell
tar -zcf archive.tar.gz /home/vczn/test/
```

**创建逐日归档文件**

```shell
# 需要手动创建归档目录和修改权限配置
sudo mkdir /archive
sudo groupadd Archives
sudo chgrp Archivers /archive
sudo usermod -aG Archivers vczn    # 添加用户 vczn 到 Archivers 组
sudo chmod 775 /archives           # group rwx
mv FileToBackup /archive
```

```shell
#!/bin/bash
DATE=$(date +%y%m%d)               # 181105
FILE=archive$DATE.tar.gz           # 拼接文件名
CONFIG_FILE=/archive/FilesToBackup
DESTINATION=/archive/$FILE
# 检查文件是否存在
if [ -f $CONFIG_FILE ]
then 
	echo 
else 
	echo 
	echo "$CONFIG_FILE does not exist."
	echo "Backup not completed due to missing Configuration File"
	echo
	exit
fi

FILE_NO=1                          # 配置文件开始行
exec < $CONFIG_FILE                # 重定向 stdin
read FILE_NAME                     # 从配置文件读取 FILENAME

while [ $? -eq 0 ]
do 
	if [ -f $FILE_NAME || -d $FILE_NAME ]
	then 
		FILE_LIST="$FILE_LIST $FILE_NAME"
	else
		echo 
		echo "$FILE_NAME, does not exist."
		echo "Obviously, I will not include it in this archive."
		echo "It is listed on line $FILE_NO of the config file."
		echo "Continuing to build archive list..."
		echo
	fi
	FILE_NO=$[$FILE_NO+1]
	read FILE_NAME
done
echo "Starting archive..."
echo
tar -zcf $DESTINATION $FILE_LIST 2 > /dev/null
echo "Archive completed"
echo "Resulting archive file is: $DESTINATION"
echo 
exit
```

**层级备份**

```shell
# 比如按小时来归档，月，日作为目录，小时作为文件名
mkdir -p $BASEDEST/$MONTH/$DAY          # 可以创建层级目录，目录即便存在，不报错
TIME=$(date +%k0%M)
DESTINATION=$BASEDST/$MONTH/$DAY/archive$TIME.tar.gz
```



## 2. 代码行数

```sh
find . -name "*.xxx" | xargs wc -l   # xargs 把标准输入传递给 wc -l

cloc file 
```



## 3. 递归删除文件

```sh
find . -name "*.txt" | xargs rm -rf
# 在当前目录下递归删除所以的 *.txt 文件
```





# 六、其他实用命令

## 1. zsh

比 bash 更强大的 shell，可以使用 `oh my zsh` 进行简单地配置



## 2. ag

类似于 `grep` 和 `ack`，可以更快速更简单地递归搜索文件内容或文件

```sh
ag [FILE-TYPE] [OPTIONS] PATTERN [PATH]
-g         # 递归查找文件名
-c         # count，仅打印在每个文件中匹配数量
-f         # following symlink
-F|-Q      # 不解析 PATTERN作为正则表达式，只作为字符串字面量
```



## 3. ccache

编译器缓存，使用缓存来加快之后的编译速度。

```sh
ccache gcc foo.cpp
```

使用 `make` 之类可以

```sh
export CC="ccache gcc"
```



## 4. tldr

man 命令输出太长懒得看，可以使用 `tldr`

```sh
sudo npm install -g tldr

tldr xxx
```



## 5. axel

多线程下载工具，替代 `curl`、`wget`

```sh
axel url
axel url -o filename
axel -n numConnections url
```



