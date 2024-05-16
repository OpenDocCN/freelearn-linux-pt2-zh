# 第九章：创建、监视、终止和重新启动进程

在上一章中，我们详细讨论了文件管理。然后，我们介绍了如何在 shell 中创建文件。此外，我们还看到了文件的各种权限，并学习了如何更改它们。接着，我们转向 Linux 环境中的目录。最后，我们在 shell 环境的上下文中使用了管道和重定向。此外，我们还了解了另一个强大的命令——`tee`命令。

在本章中，我们将探讨用于管理各种进程的各种技术。首先，我们将使用非常流行的命令——`ps`命令实时调查进程。这个`ps`命令在第二章中简要介绍了一下，*启动系统*，在*解释引导过程*部分。在本章中，我们更加强调`ps`命令，探索可以传递的更多选项，从而暴露重要信息。之后，我们进入管理守护进程的方法；首先，我们从非常流行的`top`命令开始。这种处理进程的方法在整个 Linux 社区广泛使用。这主要是因为`top`命令提供各种守护进程的实时统计信息。除此之外，我们还可以控制守护进程的行为。接着，我们转向另一种管理进程的常见方法：`service`命令。最后，我们介绍了管理守护进程的最新方法；即`systemctl`命令。这在第二章中简要介绍了，*启动系统*，在*解释引导过程*部分。在本章中，我们更深入地探讨了使用`systemctl`命令进行守护进程管理的常见做法。

在本章中，我们将涵盖以下主题：

+   `ps`命令

+   使用`top`命令查看和管理进程

+   使用`service`命令管理进程

+   使用`systemctl`命令管理进程

# ps 命令

`ps`命令代表**进程状态**，是当今环境中最流行的命令之一。它显示系统中正在运行的当前进程；当我们在 Linux 环境中工作时，我们经常忽视使一切成为可能的底层进程。`ps`命令显示的所有信息都来自一个非常流行的目录；即`/proc`文件系统。`/proc`文件系统实际上并不是一个真正的文件系统；它实际上是一个虚拟文件系统。它在启动时加载，几乎可以在今天的每个 Linux 发行版中找到`/proc`文件系统。让我们深入了解`ps`命令。

首先，我们可以显示在当前 shell 中启动的任何进程：

```
[philip@localhost ~]$ ps
 PID         TTY        TIME        CMD
 2220        pts/0      00:00:00    bash
 95677       pts/0      00:00:00    ps
[philip@localhost ~]$
```

根据前面的输出，我们在当前 shell 中没有启动任何其他进程，除了`ps`命令本身和 Bash shell。我们还可以使用`ps`命令列出当前系统中的所有进程；我们将传递`-A`参数：

```
[philip@localhost ~]$ ps -A
 PID    TTY      TIME       CMD
 1   ?        00:00:31   systemd
 2   ?        00:00:00   kthreadd
 3   ?        00:00:02   ksoftirqd/0
 5   ?        00:00:00   kworker/0:0H
 7   ?        00:00:00   migration/0
 8   ?        00:00:00   rcu_bh
 9   ?        00:00:12   rcu_sched
 10  ?        00:00:11   watchdog/0
 12  ?        00:00:00   kdevtmpfs
 13  ?        00:00:00   netns
 14  ?        00:00:00   khungtaskd
 15  ?        00:00:00   writeback
 95730  ?        00:00:00   kworker/0:3
 95747  ?        00:00:00   sleep
 95748  pts/0    00:00:00   ps
[philip@localhost ~]$
```

当我们使用`-A`或`-e`参数运行`ps`命令时，它只会打印出每个进程的进程 ID 和名称。但是，我们可以进一步扩展这个输出。我们可以传递`-a`和`-u`。这将打印出当前用户在终端中打开的进程：

```
[philip@localhost ~]$ ps -au
```

![](img/00111.jpeg)

要查看系统中当前用户拥有的所有进程，我们传递`-x`选项：

```
[philip@localhost ~]$ ps -x
PID  TTY  STAT  TIME COMMAND
1487  ?    Sl   0:00 /usr/bin/gnome-keyring-daemon --daemonize --login
1491  ?    Ssl  0:01 /usr/libexec/gnome-session-binary --session gnome-classic
1498  ?    S    0:00 dbus-launch --sh-syntax --exit-with-session
1499  ?    Ssl  0:00 /bin/dbus-daemon --fork --print-pid 4 --print-address 6 --session
1567  ?    Sl   0:00 /usr/libexec/gvfsd
1572  ?    Sl   0:00 /usr/libexec/gvfsd-fuse /run/user/1000/gvfs -f -o big_writes
1664  ?    Ss   0:00 /usr/bin/ssh-agent /bin/sh -c exec -l /bin/bash -c "env
```

```
GNOME_SHELL_SESSION_MODE=classic gnome-session --session gnome-cla
1683  ?    Sl   0:00 /usr/libexec/at-spi-bus-launcher
1688  ?    Sl   0:00 /bin/dbus-daemon --config-file=/usr/share/defaults/at-spi2/accessibility.conf --nofork --print-address 3
```

我们还可以将用户作为参数的一部分指定为`-u`：

```
[philip@localhost ~]$ ps -au root
 PID  TTY   TIME      CMD
 1   ?     00:00:31  systemd
 2   ?     00:00:00  kthreadd
 3   ?     00:00:02  ksoftirqd/0
 5   ?     00:00:00  kworker/0:0H
```

我们还可以看到所有用户以及每个守护进程的可执行文件的路径；我们传递`-aux`或`aux-`；这是**伯克利软件发行**（**BSD**）语法。BSD 是 Unix 的另一种风味。以下是 Linux 语法的示例：

```
[philip@localhost ~]$ ps -aux
USER    PID %CPU %MEM  VSZ     RSS   TTY  STAT  START   TIME 
COMMAND
root      1    0.0  0.4  193700  4216  ?    Ss    Aug08  0:31 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root      2    0.0  0.0   0      0     ?    S     Aug08  0:00 
[kthreadd]
root      3    0.0  0.0   0      0     ?    S     Aug08  0:02 [ksoftirqd/0]
root      5    0.0  0.0   0      0     ?    S     Aug08  0:00 [kworker/0:0H]
dbus      570  0.0  0.2  36524  2236   ?    Ssl   Aug08  0:14 /bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activati
chrony    571  0.0  0.0  115640 672    ?     S    Aug08  0:00 /usr/sbin/chronyd
avahi     585  0.0  0.0  30072  28     ?     S    Aug08  0:00 avahi-daemon: chroot helper
philip    2209 0.0  0.0 313472  644    ?     Sl   Aug08  0:00 /usr/libexec/gvfsd-metadata
philip    2213 0.0  1.0 720692 10608   ?     Sl   Aug08   0:05 /usr/libexec/gnome-terminal-server
```

太棒了！根据前面的输出，我们可以看到各种用户帐户。一些帐户是实际的系统帐户，例如`dbus`帐户。我们还可以指定用户帐户 ID：

```
[philip@localhost ~]$ ps -ux 1000
USER PID  %CPU  %MEM    VSZ    RSS  TTY  STAT START   TIME 
COMMAND
philip   1487  0.0   0.0    462496 996   ?    Sl   Aug08   0:00 /usr/bin/gnome-keyring-daemon --daemonize --login
philip  1491   0.0   0.1    761348 1512  ?    Ssl  Aug08   0:01 /usr/libexec/gnome-session-binary --session gnome-classic
philip  1498   0.0   0.0    13976   0    ?    S    Aug08   0:00 
dbus-launch --sh-syntax --exit-with-session
philip  1499   0.0   0.1    36284   1276 ?    Ssl  Aug08   0:00 /bin/dbus-daemon --fork --print-pid 4 --print-address 6 --session
philip  1567   0.0  0.0     386352   0   ?    Sl   Aug08   0:00 /usr/libexec/gvfsd
philip  1572   0.0  0.0     415548   52  ?    Sl   Aug08   0:00 /usr/libexec/gvfsd-fuse /run/user/1000/gvfs -f -o big_writes
```

除此之外，还可以显示由特定组拥有的进程。是的！通过传递组名或 ID 来实现。如果我们传递组名，那么我们使用`-g`：

```
[philip@localhost ~]$ ps -fg postfix
UID        PID    PPID  C  STIME  TTY      TIME      CMD
postfix    1110   1108  0  Aug08   ?    00:00:00  qmgr -l -t unix -u
postfix    95714  1108  0  06:12   ?    00:00:00  pickup -l -t unix -u
[philip@localhost ~]$
```

要传递组 ID，我们传递`-G`选项：

```
[philip@localhost ~]$ ps -fg 89
UID         PID    PPID   C  STIME  TTY         TIME   CMD
[philip@localhost ~]$
[philip@localhost ~]$ ps -fG 89
UID         PID    PPID  C   STIME  TTY   TIME       CMD
postfix     1110   1108  0   Aug08  ?     00:00:00   qmgr -l -t unix -u
postfix     95714  1108  0   06:12  ?     00:00:00   pickup -l -t unix -u
[philip@localhost ~]$
```

干得好！我们还可以通过指定**进程 ID**（**PID**）来搜索进程。我们传递`-f`，它将打印一个长列表，以及`-p`选项，它期望一个数值：

```
[philip@localhost ~]$ ps -fp 1982
UID       PID   PPID  C  STIME   TTY  TIME      CMD
philip    1982  1     0  Aug08   ?    00:00:00  /usr/libexec/tracker-store
[philip@localhost ~]$
```

有趣的是，我们甚至可以在同一行上指定多个进程；我们用逗号分隔进程：

```
[philip@localhost ~]$ ps -fp 1982,2001,2219
UID         PID   PPID  C  STIME   TTY  TIME     CMD
philip     1982   1     0  Aug08   ?    00:00:00 /usr/libexec/tracker-store
```

```
philip     2001   1730  0  Aug08   ?    00:00:00 /usr/libexec/ibus-engine-simple
philip     2219   2213  0  Aug08    ?   00:00:00 gnome-pty-helper
[philip@localhost ~]$
```

干得好！还可以通过传递`-o`选项来查找命令指定的进程 ID：

```
[philip@localhost ~]$ ps -fp 955 -o comm=sshd
[philip@localhost ~]$
```

根据前面的输出，仅显示了相应 PID 的实际可执行文件。

还可以使用`ps`命令获取内存和 CPU 信息；我们传递`-e`选项以及`-o`选项。然后，我们需要传递我们感兴趣的列名称。以下是我们如何完成这个任务：

```
[philip@localhost ~]$ ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head     -14
PID    PPID    CMD                           %MEM   %CPU
1710   1491    /usr/bin/gnome-shell           17.9   0.0
1926   1491    /usr/bin/gnome-software --g    7.1    0.0
1042   989     /usr/bin/X :0 -background n    2.5    0.0
95581  633     /sbin/dhclient -d -q -sf /u    1.3    0.0
2213   1       /usr/libexec/gnome-terminal    1.1    0.0
605    1       /usr/lib/polkit-1/polkitd -    1.1    0.0
1872   1491    /usr/libexec/gnome-settings    0.8    0.0
633    1       /usr/sbin/NetworkManager --    0.8    0.0
1890   1491    nautilus-desktop --force       0.7    0.0
2050   1915    /usr/libexec/evolution-cale    0.6    0.0
1291   1       /usr/libexec/packagekitd       0.6    0.0
632    1       /usr/bin/python -Es /usr/sb    0.4    0.0
1990   1915    /usr/libexec/evolution-cale    0.4    0.0
[philip@localhost ~]$
```

太棒了！根据前面的输出，我们指定了`pid,ppid,cmd,%mem,%cpu`。除此之外，还添加了`--sort`选项。这将查找使用最多系统 RAM 的进程，并从最高到最低显示这些进程。此外，我们添加了`head`命令；这将只显示内容的顶部部分。

我们指定只想看到前 14 行。但是，`ps`命令的输出不是实时刷新的；我们可以使用另一个流行的命令来查看输出实时刷新，而不是我们必须重新运行命令。我们使用`watch`命令来完成这个任务：

```
[philip@localhost ~]$ watch -n 1 'ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head'
```

运行前面的命令的输出如下：

![](img/00112.jpeg)

根据前面的截图，我们已经对输出进行了排序，以查看在系统中占用大部分 CPU 的进程。我们可以以分层视图查看`ps`命令的输出；我们将添加`-f`和`--forest`选项：

```
[philip@localhost ~]$ ps -af --forest
UID         PID   PPID  C  STIME  TTY     TIME      CMD
philip      99053 2220  0  07:29  pts/0   00:00:00  ps -af --forest
[philip@localhost ~]$ ps -axf --forest
PID TTY      STAT   TIME COMMAND
 2 ?        S      0:00 [kthreadd]
 3 ?        S      0:02  \_ [ksoftirqd/0]
 517?        S<sl   0:00 /sbin/auditd
519 ?        S<sl   0:01  \_ /sbin/audispd
521 ?        S<     0:00   \_ /usr/sbin/sedispatch
543 ?        SNsl   0:04     /usr/libexec/rtkit-daemon
1664 ?        Ss    0:00    \_ /usr/bin/ssh-agent /bin/sh -c exec -l /bin/bash -c "env GNOME_SHELL_SESSION_MODE=classic gnome-session --sessi
1710 ?        Sl     5:01    \_ /usr/bin/gnome-shell
1730 ?        Sl     0:00    |  \_ ibus-daemon --xim --panel disable
1743 ?        Sl     0:00    |     \_ /usr/libexec/ibus-dconf
2001 ?        Sl     0:00    |       \_ /usr/libexec/ibus-engine-simple
1872 ?        Sl     0:18     \_ /usr/libexec/gnome-settings-daemon
```

# 杀死命令

`kill`命令用于终止进程。我们可以利用刚刚介绍的`ps`命令来识别进程，然后调用`kill`命令来结束进程。以下是我们如何使用`kill`命令停止进程的方法：

```
[philip@localhost ~]$ ps -p 1788
PID   TTY       TIME     CMD
1788   ?        00:00:00 goa-daemon
[philip@localhost ~]$
[philip@localhost ~]$ kill -9 1788
[philip@localhost ~]$ ps -fp 1788
UID         PID   PPID  C STIME TTY        TIME CMD
[philip@localhost ~]$
```

太棒了！我们使用了`9`数字，这意味着发送`SIGKILL`。要查看我们可以传递的各种信号，我们可以使用`kill`命令的`-l`选项：

```
[philip@localhost ~]$ kill -l
1) SIGHUP      2) SIGINT       3) SIGQUIT    4) SIGILL    5) SIGTRAP         6) SIGABRT     7) SIGBUS       8) SIGFPE     9) SIGKILL  10) SIGUSR111) SIGSEGV       12) SIGUSR2      13) SIGPIPE   14) SIGALRM 15) SIGTERM 
16) SIGSTKFLT 17) SIGCHLD      18) SIGCONT   19) SIGSTOP 20) SIGTSTP
21) SIGTTIN   22) SIGTTOU      23) SIGURG    24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM 27) SIGPROF      28) SIGWINCH  29) SIGIO   30) SIGPWR 
31) SIGSYS     34) SIGRTMIN    35) SIGRTMIN+1 36) SIGRTMIN+2 
37) SIGRTMIN+3 38) SIGRTMIN+4  39) SIGRTMIN+5 40) SIGRTMIN+6 
41) SIGRTMIN+7 42) SIGRTMIN+8  43) SIGRTMIN+9 44) SIGRTMIN+10            45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13 48) SIGRTMIN+14            49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12            53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9 56) SIGRTMAX-8             57) SIGRTMAX-7  58) SIGRTMAX-6 59) SIGRTMAX-5 60) SIGRTMAX-4               61) SIGRTMAX-3  62) SIGRTMAX-2 63) SIGRTMAX-1 64) SIGRTMAX 
[philip@localhost ~]$
```

要使用信号名称停止进程，我们传递`-s`选项：

```
[philip@localhost ~]$ ps -fp 1990
UID     PID   PPID  C STIME  TTY   TIME      CMD
philip  1990  1915  0 Aug08  ?     00:00:00  /usr/libexec/evolution-calendar-factory-subprocess --factory contacts --bus-name org.gnome.evolution
[philip@localhost ~]$ kill -s SIGKILL 1915
[philip@localhost ~]$ ps -fp 1915
UID       PID   PPID  C STIME TTY          TIME CMD
[philip@localhost ~]$
```

在调用`kill`命令时，停止使用`SIGTERM`时应该小心。

# pstree 命令

还有另一种`ps`命令的变体，可用于查看系统中的进程——`pstree`命令。这将以分层布局呈现所有进程。它看起来是这样的：

![](img/00113.jpeg)

根据前面的截图，一些进程是父进程：它们有子进程。我们还可以通过传递`-h`选项来突出显示特定进程：

```
[root@localhost Desktop]# pstree -h 1735
rsyslogd───3*[{rsyslogd}]
[root@localhost Desktop]#The Process Grep commonly known as pgrep is another popular method
```

我们还可以仅显示特定于用户的进程；我们传递`用户名`：

![](img/00114.jpeg)

根据前面的截图，我们可以看到用户父进程是`gdm-x-session`；然后有子进程，从`Xorg`开始，向下移动树。

# pgrep 命令

**进程 Grep**，通常称为`pgrep`，是另一种在 shell 中查找进程 ID 的流行方法。如果我们知道进程名称，那么我们可以使用`pgrep`命令指定它：

```
[root@localhost Desktop]# pgrep rsyslogd
545
[root@localhost Desktop]#
```

根据前面的命令，我们可以看到`rsyslogd`的 PID。我们还可以找到特定用户的进程。为此，我们传递`-u`选项：

```
[root@localhost Desktop]# pgrep -u root rsyslogd
545
[root@localhost Desktop]#
```

干得好！

# pkill 命令

`pkill`命令是另一种用于终止进程的已知方法。它使我们能够在终止给定进程时使用进程名称。在其最简单的形式中，它如下：

```
[philip@localhost ~]$ pgrep rsyslogd
545
[philip@localhost ~]$ pkill rsyslogd
pkill: killing pid 545 failed: Operation not permitted
[philip@localhost ~]$ su
Password:
[root@localhost philip]# pkill rsyslogd
[root@localhost philip]# pgrep rsyslogd
[root@localhost philip]#
```

厉害了！根据前面的代码输出，我们可以看到`pkill`命令的有效性。

# 使用 top 命令查看和管理进程

`top`命令，意思是*进程表*，在性质上类似于 Windows 任务管理器。您会发现许多 Linux 发行版支持`top`命令。`top`命令主要用于获取系统的 CPU 和内存利用率。输出是通过创建一个由用户指定标准选择的运行进程列表来构造的；输出是实时的。每个进程的 PID 都列在第一列中。让我们开始吧：

```
[philip@localhost ~]$ top
top - 12:50:44 up 5 days, 11:44,  2 users,  load average: 0.01, 0.02, 0.05
Tasks: 165 total,   1 running, 164 sleeping,   0 stopped,   0 zombie
%Cpu(s): 12.1 us,  1.4 sy,  0.0 ni, 86.1 id,  0.0 wa,  0.0 hi,  0.4 si,  0.0 st
KiB Mem :   999696 total,    95804 free,   633636 used,   270256 buff/cache
KiB Swap:  2097148 total,  1852900 free,   244248 used.   137728 avail Mem
PID   USER    PR    NI VIRT    RES  SHR S %CPU %MEM   TIME+  COMMAND                                                                          1710 philip   20    0  1943720 175920  15680 S  9.3 17.6 5:32.99 gnome-shell 
1042 root     20    0  306324  26188  1864 S  5.6 2.6 1:13.99 X                                                                                 2213 philip   20    0  721204  11976  5992 S  2.7 1.2 0:15.04 gnome-terminal-                                                                  1934 philip   20   0  389192  6308  1952 S  0.3  0.6   5:25.10 vmtoolsd
```

```
103282 philip 20   0  157716   2260   1540 R  0.3  0.2   0:00.28 top                                                                              1  root       20   0  193700   4248   2484 S  0.0  0.4   0:33.67 systemd                                                                          2 root        20   0       0      0      0 S  0.0  0.0   0:00.21 kthreadd                                                                          
```

在最右边，有一个`COMMAND`列；这显示了可执行文件。我们可以过滤我们想要显示的用户及其相应的进程；我们在`top`中使用`-u`选项：

```
[philip@localhost ~]$ top -u philip
top - 12:55:24 up 5 days, 11:49,  2 users,  load average: 0.25, 0.08, 0.06
Tasks: 164 total,   2 running, 162 sleeping,   0 stopped,   0 zombie
%Cpu(s): 55.4 us,  6.8 sy,  0.0 ni, 36.5 id,  1.4 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :   999696 total,    73184 free,   641804 used,   284708 buff/cache
KiB Swap:  2097148 total,  1856364 free,   240784 used.   128952 avail Mem
PID   USER  PR NI VIRT    RES    SHR   S %CPU %MEM  TIME+  COMMAND 
1710 philip 20 0 1943720 177568  16612 S 42.7 17.8 5:39.32 gnome-shell 
2213 philip 20 0 721204  12256   6228  S  2.6  1.2  0:15.61 gnome-terminal-                                                                  1934 philip 20 0 389192  6308   1952  S  0.3  0.6   5:25.37 vmtoolsd 
103360 philip  20   0  157716   2260   1544 R  0.3  0.2   0:00.06 top                                                                               1487 philip  20  0 462496   1504  1004 S  0.0  0.2 0:00.08 gnome-keyring-d                                                                  1491 philip  20 0 761348 2140 1220 S 0.0 0.2  0:01.77 gnome-session-b                                                                  1498 philip  20 0 13976   0     0  S  0.0  0.0 0:00.00 dbus-launch                                                                      1499 philip  20   0  36284 160 600 S  0.0  0.2   0:00.72 dbus-daemon                                                                      1567 philip  20   0  386352    864    592 S0.0 0.1 0:00.02 gvfsd                                                                             
```

根据前面的输出，只显示了用户`philip`的进程。我们可以通过在`top`命令中按*C*来查看所有进程的绝对路径。这是按下*C*时得到的截图：

![](img/00115.jpeg)

太棒了！现在我们可以看到每个进程的位置。我们还可以更改输出的刷新频率；默认值是每三秒。我们从`top`命令中按下*D*键：

![](img/00116.jpeg)

根据前面的屏幕截图，当按下*D*键时，会出现一行新的内容：`Change delay from 3.0 to`。这提示我们要指定一个数字。我在这里输入`2`，这样更新将每两秒刷新一次。现在，当我再次按下*D*键时，我们会注意到提示中的差异：

![](img/00117.jpeg)

干得好！要查看`top`的帮助，我们可以按*H*：

![](img/00118.jpeg)

我们可以更改`top`实用程序中内存的显示方式；根据当前的内存输出，当我们按下*M*时，显示将切换：

```
top - 13:09:50 up 5 days, 12:03,  2 users, load average: 0.00, 0.04, 0.05
Tasks: 164 total,   1 running, 163 sleeping,   0 stopped,   0 zombie
%Cpu(s):  4.1 us,  0.7 sy,  0.0 ni, 95.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

根据前面的屏幕截图，内存部分被隐藏了。当我们再次按下*M*键时，这将改变：

![](img/00119.jpeg)

太棒了！如果我们再次按下*M*键，我们会看到一种图形设计：

![](img/00120.jpeg)

干得好！现在我们有漂亮的条形图，指示了 RAM 和交换的内存使用情况。同样，我们可以通过 CPU 更改输出的显示；为此，我们按*T*：

```
top - 13:19:23 up 5 days, 12:13,  2 users,  load average: 0.30, 0.11, 0.07
Tasks: 163 total,   3 running, 160 sleeping,   0 stopped,   0 zombie
2 sleeping,   0 stopped,   0 zombie
%Cpu(s):   9.6/1.4    11[|||||||||||                                                                                         ]
KiB Mem :   999696 total,    73524 free,   641328 used,   284844 buff/cache
KiB Swap:  2097148 total,  1856532 free,   240616 used.   129444 avail Mem
```

太棒了！当我们按*T*时，它会将条形图变成阴影输出：

![](img/00121.gif)

除此之外，进程还可以以分层输出的方式显示；我们按*Shift* + *V*：

![](img/00122.jpeg)

要关闭层次视图，我们只需再次切换*Shift* + *V*：

![](img/00123.jpeg)

我们还可以使用`top`命令停止一个进程；我们按*K*，这是在`top`命令中杀死进程的快捷键：

![](img/00124.jpeg)

基于前面的命令，会出现一行新的内容：`PID to signal/kill [default pid = 1710]`，我们需要指定一个进程 ID：

```
KiB Swap:2097148 total, 1856800 free, 240348 used. 129084 avail Mem
Send pid 1718 signal [15/sigterm]
PID  USER   PR NI VIRT   RES    SHR   S %CPU %MEM  TIME+   COMMAND                                                                          1710 philip 20 0 1944552 176788 16840 S  1.5 17.7  6:36.40 gnome-shell                                                                       2213 philip 20 0 721724  16020   8740 S  0.5 1.6   0:22.60 gnome-terminal-
```

现在我们需要指定要发送给进程的信号；默认是`15/sigterm`。我们将接受默认值；这将在不必退出`top`实用程序的情况下终止进程。

# 使用`service`命令管理进程

`service`命令最初用于在`systemd`之前的早期 Linux 发行版上运行 SysVinit 脚本。根据您要完成的任务，您用于启动、停止或重新启动服务的方法将取决于您的发行版是使用`systemd`还是`init`。大多数 Linux 工程师更喜欢使用`service`命令而不是在系统环境中处理进程的较新方法。因此，在大多数较新的发行版中支持`service`命令。`service`命令的语法是：

```
service <process> <status>
```

要查看运行 SysV 脚本的系统上的所有服务，我们将使用 CentOS 6.5 系统：

```
[philip@localhost Desktop]$ service --status-all
abrt-ccpp hook is installed
abrtd (pid  2254) is running...
abrt-dump-oops is stopped
acpid (pid  1964) is running...
atd (pid  2273) is running...
auditd (pid  1710) is running...
Usage: /etc/init.d/bluetooth {start|stop}
cpuspeed is stopped
crond (pid  2262) is running...
cupsd (pid  1874) is running...
dnsmasq (pid  2087) is running...
firstboot is not scheduled to run
hald (pid  1975) is running...
htcacheclean is stopped
httpd is stopped
winbindd is stopped
wpa_supplicant (pid  1875) is running...
[philip@localhost Desktop]$
```

`service`命令读取的脚本以`rc`开头。我们可以快速查看所有相关脚本：

```
[philip@localhost Desktop]$ ls -l /etc | grep rc.
lrwxrwxrwx.  1 root root     11 Jun 20 01:37 init.d -> rc.d/init.d
lrwxrwxrwx.  1 root root      7 Jun 20 01:40 rc -> rc.d/rc
lrwxrwxrwx.  1 root root     10 Jun 20 01:40 rc0.d -> rc.d/rc0.d
lrwxrwxrwx.  1 root root     10 Jun 20 01:40 rc1.d -> rc.d/rc1.d
lrwxrwxrwx.  1 root root     10 Jun 20 01:40 rc2.d -> rc.d/rc2.d
lrwxrwxrwx.  1 root root     10 Jun 20 01:40 rc3.d -> rc.d/rc3.d
lrwxrwxrwx.  1 root root     10 Jun 20 01:40 rc4.d -> rc.d/rc4.d
lrwxrwxrwx.  1 root root     10 Jun 20 01:40 rc5.d -> rc.d/rc5.d
lrwxrwxrwx.  1 root root     10 Jun 20 01:40 rc6.d -> rc.d/rc6.d
drwxr-xr-x. 10 root root   4096 Jun 20 05:50 rc.d
lrwxrwxrwx.  1 root root     13 Jun 20 01:40 rc.local -> rc.d/rc.local
lrwxrwxrwx.  1 root root     15 Jun 20 01:40 rc.sysinit -> rc.d/rc.sysinit
[philip@localhost Desktop]$
```

要控制进程的状态，我们可以这样做：

```
[philip@localhost Desktop]$ service crond status
crond (pid  4457) is running...
[philip@localhost Desktop]$
```

根据前面的命令，这个特定进程目前正在运行。我们可以更改这一点；假设我们想要停止`crond`进程。我们只需用`stop`替换`status`：

```
[philip@localhost Desktop]$ service crond stop
User has insufficient privilege.
[philip@localhost Desktop]$
```

根据前面的输出，我们遇到了一个障碍；这可以很容易地通过成为 root 用户来解决：

```
[root@localhost Desktop]# service crond stop
Stopping crond:                                            [  OK  ]
[root@localhost Desktop]#
```

太棒了！现在我们可以重新运行`service`命令；这次使用`status`选项：

```
[root@localhost Desktop]# service crond status
crond is stopped
[root@localhost Desktop]#
```

然后我们完成了。服务已经停止。要开始备份这个进程，我们只需用`start`替换`stop`：

```
[root@localhost Desktop]# service crond start
Starting crond:                                            [  OK  ]
[root@localhost Desktop]#
```

现在让我们再次尝试启动这个进程：

```
[root@localhost Desktop]# service crond status
crond (pid  6606) is running...
[root@localhost Desktop]#
```

干得好！如果出于某种原因，我们对进程进行了更改并需要重新启动进程，那么我们可以通过多种方式来做到。我们可以停止进程，然后再次启动它：

```
[root@localhost Desktop]# service crond stop
Stopping crond:                                            [  OK  ]
[root@localhost Desktop]# service crond start
Starting crond:                                            [  OK  ]
[root@localhost Desktop]#
```

此外，我们可以使用`restart`选项：

```
[root@localhost Desktop]# service crond restart
Stopping crond:                                            [  OK  ]
Starting crond:                                            [  OK  ]
[root@localhost Desktop]#
```

最后，我们可以使用`reload`选项；这个选项将重新读取已经进行的任何更改的配置文件：

```
[root@localhost Desktop]# service crond reload
Reloading crond:                                           [  OK  ]
[root@localhost Desktop]# service crond status
crond (pid  6703) is running...
[root@localhost Desktop]#
```

太棒了！

# 使用 systemctl 命令管理进程

在大多数使用 system 的新发行版上，我们将使用`systemctl`命令来管理进程。Linux 开发人员也留下了对`service`命令的支持；如果我们尝试使用`service`命令终止一个进程，那么我们会发现它实际上会将我们的请求重定向到`systemctl`命令。让我们试试看：

```
[root@localhost philip]# service crond status
Redirecting to /bin/systemctl status crond.service
crond.service - Command Scheduler
Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
Active: active (running) since Thu 2018-08-02 07:13:38 PDT; 1 weeks 5 days ago
 Main PID: 991 (crond)
CGroup: /system.slice/crond.service
 └─991 /usr/sbin/crond -n
Aug 02 07:13:38 localhost.localdomain systemd[1]: Started Command Scheduler.
Aug 02 07:13:38 localhost.localdomain systemd[1]: Starting Command Scheduler...
Aug 02 07:13:38 localhost.localdomain crond[991]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 15% if used.)
Aug 02 07:13:43 localhost.localdomain crond[991]: (CRON) INFO (running with inotify support)
[root@localhost philip]#
```

太棒了！根据输出，我们可以看到`service`命令实际上正在被重定向：

```
[root@localhost philip]# service crond status
Redirecting to /bin/systemctl status crond.service
crond.service - Command Scheduler
```

现在让我们尝试使用管理进程的新方法；我们将使用`systemctl`命令。格式如下：

```
systemctl <action><process>
```

我们可以在 shell 中使用这个：

```
[root@localhost philip]# systemctl status atd
atd.service - Job spooling tools
Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; vendor preset: enabled)
Active: active (running) since Thu 2018-08-02 07:13:38 PDT; 1 weeks 5 days ago
Main PID: 993 (atd)
CGroup: /system.slice/atd.service
 └─993 /usr/sbin/atd -f
Aug 02 07:13:38 localhost.localdomain systemd[1]: Started Job spooling tools.
Aug 02 07:13:38 localhost.localdomain systemd[1]: Starting Job spooling tools...
[root@localhost philip]#
```

使用`systemctl`启动一个进程，我们传递`start`选项：

```
[root@localhost philip]# systemctl start rsyslog.service
[root@localhost philip]#
```

我们可以通过传递`status`选项来检查进程的状态：

```
[root@localhost philip]# systemctl status rsyslog.service
rsyslog.service - System Logging Service
Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
 Active: active (running) since Tue 2018-08-14 08:29:22 PDT; 5s ago
 Docs:
man:rsyslogd(8)
 http://www.rsyslog.com/doc/
Main PID: 117499 (rsyslogd)
 CGroup: /system.slice/rsyslog.service
 └─117499 /usr/sbin/rsyslogd -n
Aug 14 08:29:22 localhost.localdomain systemd[1]: Starting System Logging Service...
Aug 14 08:29:22 localhost.localdomain rsyslogd[117499]:  [origin software="rsyslogd" swVersion="8.24.0" x-pid="117499" x-info="http://www.rs...] start
Aug 14 08:29:22 localhost.localdomain systemd[1]: Started System Logging Service.
Hint: Some lines were ellipsized, use -l to show in full.
[root@localhost philip]#
```

您会注意到，与早期 Linux 发行版中的旧 service 命令相比，`systemctl`命令的输出要直观得多。我们也可以使用`systemctl`命令停止一个进程；我们传递`stop`选项：

```
[root@localhost philip]# systemctl stop rsyslog.service
[root@localhost philip]# systemctl status rsyslog.service
rsyslog.service - System Logging Service
Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
Active: inactive (dead) since Tue 2018-08-14 08:38:38 PDT; 8s ago
Docs: man:rsyslogd(8)
http://www.rsyslog.com/doc/
Process: 117499 ExecStart=/usr/sbin/rsyslogd -n $SYSLOGD_OPTIONS (code=exited, status=0/SUCCESS)
Main PID: 117499 (code=exited, status=0/SUCCESS)
Aug 14 08:29:22 localhost.localdomain systemd[1]: Starting System Logging Service...
Aug 14 08:29:22 localhost.localdomain rsyslogd[117499]:  [origin software="rsyslogd" swVersion="8.24.0" x-pid="117499" x-info="http://www.rs...] start
Aug 14 08:29:22 localhost.localdomain systemd[1]: Started System Logging Service.
Aug 14 08:38:38 localhost.localdomain rsyslogd[117499]:  [origin software="rsyslogd" swVersion="8.24.0" x-pid="117499" x-info="http://www.rs...nal 15.
Aug 14 08:38:38 localhost.localdomain systemd[1]: Stopping System Logging Service...
Aug 14 08:38:38 localhost.localdomain systemd[1]: Stopped System Logging Service.
Hint: Some lines were ellipsized, use -l to show in full.
[root@localhost philip]#
```

此外，我们可以重新启动或重新加载一个进程：

```
[root@localhost philip]# systemctl restart rsyslog.service
[root@localhost philip]# systemctl status rsyslog.service
rsyslog.service -
System Logging Service
Loaded: loaded (/usr/lib/systemd/system/rsyslog.service; enabled; vendor preset: enabled)
 Active: active (running) since Tue 2018-08-14 08:39:37 PDT; 2s ago
 Docs: man:rsyslogd(8)
 http://www.rsyslog.com/doc/
Main PID: 117730 (rsyslogd)
CGroup: /system.slice/rsyslog.service
 └─117730 /usr/sbin/rsyslogd -n
Aug 14 08:39:37 localhost.localdomain systemd[1]: Starting System Logging Service...
Aug 14 08:39:37 localhost.localdomain rsyslogd[117730]:  [origin software="rsyslogd" swVersion="8.24.0" x-pid="117730" x-info="http://www.rs...] start
Aug 14 08:39:37 localhost.localdomain systemd[1]: Started System Logging Service.
Hint: Some lines were ellipsized, use -l to show in full.
[root@localhost philip]#
```

根据前面的输出，当我们传递`restart`选项时，它只是启动了进程。使用`systemctl`命令处理的进程在使用`systemctl`命令时被视为单元。我们可以通过传递`list-units`文件来查看这些单元：

```
[root@localhost philip]# systemctl list-units --all --state=active
UNIT    LOAD   ACTIVE SUB       DESCRIPTION
proc-sys-fs-binfmt_misc.automount                              loaded active waiting   Arbitrary Executable File Formats File System Automount Point
dev-cdrom.device                                               loaded active plugged   VMware_Virtual_IDE_CDROM_Drive
dev-disk-by\x2did-ata\x2dVMware_Virtual_IDE_CDROM_Drive_10000000000000000001.device loaded active plugged   VMware_Virtual_IDE_CDROM_Drive
dev-disk-by\x2dpath-pci\x2d0000:00:07.1\x2data\x2d2.0.device   loaded active plugged   VMware_Virtual_IDE_CDROM_Drive
dev-disk-by\x2dpath-pci\x2d0000:00:10.0\x2dscsi\x2d0:0:0:0.device loaded active plugged   VMware_Virtual_S
dev-disk-by\x2dpath-pci\x2d0000:00:10.0\x2dscsi\x2d0:0:0:0\x2dpart1.device loaded active plugged   VMware_Virtual_S 1
dev-disk-by\x2dpath-pci\x2d0000:00:10.0\x2dscsi\x2d0:0:0:0\x2dpart2.device loaded active plugged   VMware_Virtual_S 2
dev-disk-by\x2dpath-pci\x2d0000:00:10.0\x2dscsi\x2d0:0:0:0\x2dpart3.device loaded active plugged   VMware_Virtual_S 3
dev-disk-by\x2duuid-16e2de7b\x2db679\x2d4a12\x2d888e\x2d55081af4dad8.device loaded active plugged   VMware_Virtual_S 3
sys-devices-virtual-net-virbr0\x2dnic.device                   loaded active plugged   /sys/devices/virtual/net/virbr0-nic
[root@localhost philip]#
```

各种进程存储在`/usr/lib/systemd/system`中：

```
[root@localhost philip]# ls /usr/lib/systemd/system
abrt-ccpp.service       iscsiuio.socket          shutdown.target
abrtd.service           kdump.service            shutdown.target.wants
abrt-oops.service       kexec.target             sigpwr.target
abrt-pstoreoops.service kexec.target.wants       sleep.target
abrt-vmcore.service     kmod-static-nodes.service  -.slice
abrt-xorg.service       kpatch.service            slices.target
accounts-daemon.service ksm.service               smartcard.target
alsa-restore.service    ksmtuned.service          smartd.service
alsa-state.service      libstoragemgmt.service    sockets.target
alsa-store.service      libvirtd.service          sockets.target.wants
[root@localhost philip]#
```

正如您所看到的，有各种各样的进程是使用`systemctl`命令进行管理的。

# 总结

在本章中，我们处理了与在 shell 中处理进程相关的各个方面。我们从`ps`命令开始。展示了在 shell 中显示当前运行的进程的方法。接下来，我们看到了如何打印系统上运行的所有进程。然后是暴露每个进程使用的命令。然后，我们专注于过滤特定用户的输出，也可以通过用户 ID 进行过滤。之后，我们触及了对进程进行过滤，也可以通过进程 ID 进行过滤。除此之外，我们还处理了按组进行过滤。然后，我们将显示更改为树状布局。

此外，我们看到了如何获取内存和 CPU 信息；我们调用`watch`命令实时更新结果。最后，我们看到了如何使用`ps`命令结合`kill`命令终止一个进程。接下来，我们触及了`pstree`命令；这以分层格式呈现进程。我们甚至操纵了它的输出来缩小到特定的进程；此外，我们还检查了特定用户的进程。

在此之后，我们触及了`pgrep`命令，也称为进程 grep。这是另一种查找进程 ID 的方法；可以提供进程名称，也可以指定要显示的用户。在此之后，我们触及了`pkill`命令；顾名思义，它用于终止一个进程。我们在演示中看到了这一点。之后，我们使用`top`命令，使用各种技术来操作结果的输出，并探讨了如何在`top`命令中终止一个进程。

接下来，我们使用了`service`命令；我们谈到了我们通常在哪里找到它，并查看了它在较新的 Linux 发行版中的支持。使用`service`命令进行了各种演示。最后，我们使用了`systemctl`命令；这是迄今为止在使用系统的较新 Linux 发行版中管理进程的最佳方法，而不是使用 SysVinit 脚本的较旧的 Linux 发行版。

在下一章中，我们将深入探讨管理进程。有时我们希望优先考虑一个进程而不是另一个。这是下一章的重点。这不仅能让您管理系统上的进程，还能让您比其他人更有优势，从而让您更接近认证。希望能在那里见到您。

# 问题

1.  哪个命令打印在新终端中启动的进程？

A. `pkill`

B. `chmod`

C. `ps`

D. `chage`

1.  `ps`命令的哪个选项可以用于打印系统中运行的所有进程？

A. `-B`

B. `-b`

C. `-e`

D. `-x`

1.  `ps`命令的哪个选项可以用于以分层布局打印输出？

A. `-forest`

B. `--forest`

C. `--tree`

D. `-tree`

1.  `ps`命令的哪个选项用于指定用户进程？

A. `-x`

B. `-a`

C. `-u`

D. `-d`

1.  `kill`命令的哪个选项用于显示各种 SIG 术语？

A. `-`

B. `-l`

C. `-i`

D. `-d`

1.  使用`kill`命令时，哪个数字等同于`SIGKILL`？

A. `8`

B. `10`

C. `7`

D. `9`

1.  `top`命令的哪个选项可以指定用户？

A. `-u`

B. `-p`

C. `-v`

D. `-a`

1.  `top`实用程序中用哪个字母设置刷新结果的频率？

A. `-a`

B. `b`

C. `d`

D. `e`

1.  哪个选项可以用于使用`service`命令重新读取进程配置？

A. `reboot`

B. `stop`

C. `status`

D. `reload`

1.  在由`systemctl`命令管理的系统中，单位`/processes`位于哪个目录？

A. `/var/lib/systemd`

B. `/usr/lib/systemd/system`

C. `/usr/systemd/system`

D. `/usr/system/systemd`

# 进一步阅读

+   有关进程的更多信息，请参阅：[`www.tutorialspoint.com.`](https://www.tutorialspoint.com)

+   这个网站为您提供了许多有关进程的有用提示：[`www.linux.com.`](https://www.linux.com)

+   这个最后的链接为您提供了与各种命令相关的一般信息。您可以在那里发布您的问题，其他社区成员将会回答：[`www.linuxquestions.org.`](https://www.linuxquestions.org)
