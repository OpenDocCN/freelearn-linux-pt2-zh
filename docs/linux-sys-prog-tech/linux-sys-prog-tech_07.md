# 第七章：使用 systemd 处理您的守护进程

现在我们知道如何构建我们自己的守护进程，是时候看看我们如何使用**systemd**让 Linux 来处理它们了。在本章中，我们将学习 systemd 是什么，如何启动和停止服务，什么是单元文件，以及如何创建它们。我们还将学习守护进程如何记录到 systemd 中以及如何读取这些日志。

然后，我们将了解 systemd 可以处理的不同类型的服务和守护进程，并将上一章的守护进程放到 systemd 控制下。

在本章中，我们将涵盖以下示例：

+   了解 systemd

+   为守护进程编写一个单元文件

+   启用和禁用服务，以及启动和停止它

+   为 systemd 创建一个更现代的守护进程

+   使新的守护进程成为 systemd 服务

+   阅读日志

# 技术要求

对于这个示例，您需要一台使用 systemd 的 Linux 发行版的计算机——今天几乎每个发行版都是如此，只有一些少见的例外。

您还需要 GCC 编译器和 Make 工具。这些工具的安装说明在*第一章*中有涵盖。您还需要本章的通用 Makefile，在本章的 GitHub 存储库中可以找到，以及本章的所有代码示例。本章的 GitHub 存储库文件夹的 URL 是[`github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch7`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch7)。

查看以下链接以查看“代码实战”视频：[`bit.ly/3cxmXab`](https://bit.ly/3cxmXab)

# 了解 systemd

在这个示例中，我们将探讨 systemd 是什么，它如何处理系统以及所有系统的服务。

从历史上看，Linux 一直由几个较小的部分管理。例如，`init`是系统上的第一个进程，它启动其他进程和守护进程来启动系统。系统守护进程由 shell 脚本处理，也称为*init 脚本*。日志记录是通过守护进程自己通过文件或`syslog`来完成的。网络也是由多个脚本处理的（在一些 Linux 发行版中仍然是这样）。

然而，现在整个系统都由 systemd 处理。例如，系统上的第一个进程现在是`systemd`（我们在之前的章节中已经看到了）。守护进程由称为*单元文件*的东西处理，它在系统上创建了一种统一的控制守护进程的方式。日志记录由**journald**处理，它是 systemd 的日志记录守护进程。但请注意，**syslog**仍然被许多守护进程用于额外的日志记录。在本章的*使新的守护进程成为 systemd 服务*部分中，我们将重新编写*第六章*中的守护进程，以记录到日志中。

了解 systemd 的工作原理将使您能够在编写守护进程的单元文件时正确使用它。它还将帮助您以“新”的方式编写守护进程，以利用 systemd 的日志记录功能。您将成为一个更好的系统管理员，也将成为一个更好的 Linux 开发人员。

## 准备工作

对于这个示例，您只需要一个使用 systemd 的 Linux 发行版，大多数发行版今天都使用 systemd。

如何做…

在这个示例中，我们将看一下 systemd 涉及的一些组件。这将让我们俯瞰 systemd、journald、它的命令和**单元文件**。所有的细节将在本章的后续示例中介绍：

1.  在控制台窗口中键入`systemctl`并按*Enter*。这将显示您机器上当前所有活动的*单元*。如果您浏览列表，您会注意到一个单元可以是任何东西——硬盘、声卡、挂载的网络驱动器、各种服务、定时器等等。

1.  我们在上一步看到的所有服务都作为单元文件存储在`/lib/systemd/system`或`/etc/systemd/system`中。转到这些目录并查看文件。这些都是典型的单元文件。

1.  现在是时候来看一下日志，即 systemd 的日志。我们需要以`sudo journalctl`命令运行此命令，或者首先切换到 root 用户，然后输入`journalctl`。这将显示 systemd 和其所有服务的整个日志。按*Spacebar*键几次以在日志中向下滚动。要转到日志的末尾，在日志显示时输入大写*G*。

## 它是如何工作的...

这三个步骤让我们对 systemd 有了一个概述。在接下来的教程中，我们将更深入地介绍细节。

已安装的软件包将其单元文件放在`/lib/systemd/system`中，如果是 Debian/Ubuntu 系统，则放在`/usr/lib/systemd/system`中，如果是 CentOS/Fedora 系统。但是，在 CentOS/Fedora 上，`/lib`是指向`/usr/lib`的符号链接，因此`/lib/systemd/system`是通用的。

所谓的*local*单元文件放在`/etc/systemd/system`中。本地单元文件意味着特定于此系统的单元文件，例如，由管理员修改或手动添加的某些程序。

## 还有更多...

在 systemd 之前，Linux 有其他的初始化系统。我们已经简要提到了第一个`init`。那个初始化系统`init`通常被称为*Sys-V-style init*，来自 UNIX 版本五（V）。

在 Sys-V-style init 之后，出现了 Upstart，这是 Ubuntu 开发的`init`的完全替代品。Upstart 也被 CentOS 6 和 Red Hat Enterprise Linux 6 使用。

然而，如今，大多数主要的 Linux 发行版都使用 systemd。由于 systemd 是 Linux 的一个重要组成部分，这使得所有发行版几乎都是相似的。十五年前，从一个发行版跳到另一个发行版并不容易。如今，这变得更容易了。

## 另请参阅

系统上有多个手册页面，我们可以阅读以更深入地了解 systemd、其命令和日志：

+   `man systemd`

+   `man systemctl`

+   `man journalctl`

+   `man systemd.unit`

# 为守护进程编写单元文件

在这个教程中，我们将把我们在*第六章*中编写的守护程序，*生成进程和使用作业控制*，变成 systemd 下的一个服务。这个守护程序是 systemd 称之为*forking daemon*的，因为它就是这样。它分叉。这通常是守护程序的工作方式，它们仍然被广泛使用。在本章的*将新守护程序变成 systemd 服务*部分中，我们将稍微修改它以记录到 systemd 的日志中。但首先，让我们将我们现有的守护程序变成一个服务。

## 准备工作

在这个教程中，您将需要我们在*第六章*中编写的文件`my-daemon-v2.c`，*生成进程和使用作业控制*。如果您没有该文件，在 GitHub 的本章目录中有一份副本，网址为[`github.com/PacktPublishing/Linux-System-Programming-Techniques/blob/master/ch7/my-daemon-v2.c`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/blob/master/ch7/my-daemon-v2.c)。

除了`my-daemon-v2.c`，您还需要 GCC 编译器、Make 工具和本章*技术要求*部分中涵盖的通用 Makefile。

## 如何做...

在这里，我们将把我们的守护程序置于 systemd 的控制之下：

1.  如果您还没有编译`my-daemon-v2`，我们需要从那里开始。像我们迄今为止制作的任何其他程序一样编译它：

```
$> make my-daemon-v2
gcc -Wall -Wextra -pedantic -std=c99    my-daemon-v2.c   -o my-daemon-v2
```

1.  为了使其成为系统守护程序，我们应该将其放在其中一个专门用于此目的的目录中。一个很好的地方是`/usr/local/sbin`。`/usr/local 目录`通常是我们想要放置我们自己添加到系统中的东西的地方，也就是第三方的东西。`sbin`子目录用于系统二进制文件或超级用户二进制文件（因此在*bin*之前有一个*s*）。要将我们的守护程序移到这里，我们需要成为 root 用户：

```
$> sudo mv my-daemon-v2 /usr/local/sbin/
```

1.  现在来写守护程序的*单元文件*，这才是令人兴奋的部分。以 root 身份创建文件`/etc/systemd/system/my-daemon.service`。使用`sudo`或`su`成为 root。在文件中写入下面显示的内容并保存。单元文件分为几个部分。在这个文件中，部分是`[Unit]`、`[Service]`和`[Install]`。`[Unit]`部分包含有关单元的信息，例如我们的描述。`[Service]`部分包含有关此服务应如何工作和行为的信息。在这里，我们有`ExecStart`，其中包含守护程序的路径。我们还有`Restart=on-failure`。这告诉 systemd 如果守护程序崩溃，应重新启动它。然后我们有`Type`指令，在我们的情况下是 forking。请记住，我们的守护程序创建了一个自己的分支，父进程退出。这就是*forking*类型的含义。我们告诉 systemd 类型，以便它知道如何处理守护程序。然后我们有`PIDFile`，其中包含我们的`WantedBy`设置为`multi-user.target`。这意味着当系统进入多用户阶段时，此守护程序应该启动：

```
[Unit]
Description=A small daemon for testing
[Service]
ExecStart=/usr/local/sbin/my-daemon-v2
Restart=on-failure
Type=forking
PIDFile=/var/run/my-daemon.pid
[Install]
WantedBy=multi-user.target
```

1.  为了让系统识别我们的新单元文件，我们需要*重新加载*systemd 守护程序本身。这将读取我们的新文件。这必须以 root 身份完成：

```
$> sudo systemctl daemon-reload
```

1.  我们现在可以使用`systemctl`的`status`命令来查看 systemd 是否识别我们的新守护程序。请注意，我们在这里从单元文件中看到了描述，以及实际使用的单元文件。我们还看到守护程序当前是*禁用*和*未激活*的：

```
$> sudo systemctl status my-daemon
. my-daemon.service - A small daemon for testing
   Loaded: loaded (/etc/systemd/system/my-daemon.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
```

## 它是如何工作的...

为守护程序创建一个 systemd 服务并不比这更难。一旦我们学会了 systemd 和单元文件，就比在旧日写*init 脚本*更容易。只用了九行，我们就将守护程序置于 systemd 的控制之下。

单元文件大部分都是不言自明的。在我们的情况下，对于一个传统的分叉守护程序，我们将类型设置为*forking*并指定一个 PID 文件。然后 systemd 使用 PID 文件中的 PID 号来跟踪守护程序的状态。这样，如果 systemd 注意到 PID 从系统中消失，它就可以重新启动守护程序。

在状态消息中，我们看到服务被*禁用*和*未激活*。**禁用**意味着系统启动时不会自动启动。**未激活**意味着它还没有启动。

## 还有更多...

如果您为使用网络的守护程序编写一个单元文件，例如互联网守护程序，您可以明确告诉 systemd 等待直到网络准备就绪。为了实现这一点，我们在`[Unit]`部分下添加以下行：

```
After=network-online.target
```

```
Wants=network-online.target
```

当然，您也可以为其他依赖关系使用`After`和`Wants`。还有另一个依赖语句可以使用，称为`Requires`。

它们之间的区别在于`After`指定了单元的顺序。具有`After`的单元将在所需单元启动后等待启动。然而，`Wants`和`Requires`只指定了依赖关系，而不是顺序。使用`Wants`，即使其他所需单元未成功启动，单元仍将启动。但是使用`Requires`，如果所需单元未启动，单元将无法启动。

## 另请参阅

在`man systemd.unit`中有关于单元文件的不同部分以及我们可以在每个部分中使用的指令的大量信息。

# 启用和禁用服务 - 以及启动和停止它

在上一个教程中，我们使用一个单元文件将我们的守护程序添加为 systemd 的一个服务。在这个教程中，我们将学习如何启用、启动、停止和禁用它。启用和启动以及禁用和停止服务之间有区别。

启用服务意味着系统启动时将自动启动。启动服务意味着它将立即启动，无论它是否已启用。禁用服务意味着它将不再在系统启动时启动。停止服务会立即停止它，无论它是否已启用或禁用。

了解如何做所有这些可以让你控制系统的服务。

## 准备工作

为了使这个教程起作用，你首先需要完成前面的教程，*为守护进程编写一个单元文件*。

## 如何做...

1.  让我们首先再次检查守护进程的状态。它应该是禁用和未激活的：

```
$> systemctl status my-daemon
. my-daemon.service - A small daemon for testing
   Loaded: loaded (/etc/systemd/system/my-daemon.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
```

1.  现在我们将*启用*它，这意味着它将在启动时自动启动（当系统进入*多用户模式*时）。由于这是一个修改系统的命令，我们必须以 root 身份发出此命令。还要注意当我们启用它时发生了什么。没有什么神秘的事情发生；它只是从我们的单元文件创建一个符号链接到`/etc/systemd/system/multi-user.target.wants/my-daemon.service`。请记住，`multi-user.target`是我们在单元文件中指定的目标。因此，当系统达到多用户级别时，systemd 将启动该目录中的所有服务：

```
$> sudo systemctl enable my-daemon
Created symlink /etc/systemd/system/multi-user.target.wants/my-daemon.service → /etc/systemd/system/my-daemon.service.
```

1.  现在让我们检查一下守护进程的状态，因为我们已经启用了它。现在它应该显示*已启用*而不是*已禁用*。但是，它仍然是*未激活*（未启动）：

```
$> sudo systemctl status my-daemon
. my-daemon.service - A small daemon for testing
   Loaded: loaded (/etc/systemd/system/my-daemon.service; enabled; vendor preset: enabled)
   Active: inactive (dead)
```

1.  现在是启动守护进程的时候了：

```
$> sudo systemctl start my-daemon
```

1.  让我们再次检查状态。它应该是启用和活动的（也就是已启动）。这一次，我们将获得比以前更多关于守护进程的信息。我们将看到它的 PID、状态、内存使用情况等。我们还将在最后看到日志的片段：

```
$> sudo systemctl status my-daemon
. my-daemon.service - A small daemon for testing
   Loaded: loaded (/etc/systemd/system/my-daemon.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2020-12-06 14:50:35 CET; 9s ago
  Process: 29708 ExecStart=/usr/local/sbin/my-daemon-v2 (code=exited, status=0/SUCCESS)
 Main PID: 29709 (my-daemon-v2)
    Tasks: 1 (limit: 4915)
   Memory: 152.0K
   CGroup: /system.slice/my-daemon.service
           └─29709 /usr/local/sbin/my-daemon-v2
dec 06 14:50:35 red-dwarf systemd[1]: Starting A small daemon for testing...
dec 06 14:50:35 red-dwarf systemd[1]: my-daemon.service: Can't open PID file /run/my-daemon.pid (yet?) after start
dec 06 14:50:35 red-dwarf systemd[1]: Started A small daemon for testing.
```

1.  让我们验证一下，如果守护进程崩溃或被杀死，systemd 是否会重新启动它。首先，我们用`ps`查看进程。然后我们用`KILL`信号杀死它，所以它没有机会正常退出。然后我们再次用`ps`查看它，并注意到它有一个新的 PID，因为它是一个新的进程。旧的进程被杀死了，systemd 启动了一个新的实例：

```
$> ps ax | grep my-daemon-v2
923 pts/12   S+     0:00 grep my-daemon-v2
29709 ?        S      0:00 /usr/local/sbin/my-daemon-v2
$> sudo kill -KILL 29709
$> ps ax | grep my-daemon-v2
 1103 ?        S      0:00 /usr/local/sbin/my-daemon-v2
 1109 pts/12   S+     0:00 grep my-daemon-v2
```

1.  我们还可以查看守护进程在`/tmp`目录中写入的文件：

```
$> tail -n 5 /tmp/my-daemon-is-alive.txt 
Daemon alive at Sun Dec  6 15:24:11 2020
Daemon alive at Sun Dec  6 15:24:41 2020
Daemon alive at Sun Dec  6 15:25:11 2020
Daemon alive at Sun Dec  6 15:25:41 2020
Daemon alive at Sun Dec  6 15:26:11 2020
```

1.  最后，让我们停止守护进程。我们还将检查它的状态，并检查进程是否已经消失了`ps`：

```
$> sudo systemctl stop my-daemon
$> sudo systemctl status my-daemon
. my-daemon.service - A small daemon for testing
   Loaded: loaded (/etc/systemd/system/my-daemon.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Sun 2020-12-06 15:27:49 CET; 7s ago
  Process: 1102 ExecStart=/usr/local/sbin/my-daemon-v2 (code=exited, status=0/SUCCESS)
 Main PID: 1103 (code=killed, signal=TERM)
dec 06 15:18:41 red-dwarf systemd[1]: Starting A small daemon for testing...
dec 06 14:50:35 red-dwarf systemd[1]: my-daemon.service: Can't open PID file /run/my-daemon.pid (yet?) after start
dec 06 15:18:41 red-dwarf systemd[1]: Started A small daemon for testing.
dec 06 15:27:49 red-dwarf systemd[1]: Stopping A small daemon for testing...
dec 06 15:27:49 red-dwarf systemd[1]: my-daemon.service: Succeeded.
dec 06 15:27:49 red-dwarf systemd[1]: Stopped A small daemon for testing.
$> ps ax | grep my-daemon-v2
 2769 pts/12   S+     0:00 grep my-daemon-v2
```

1.  为了防止守护进程在系统重新启动时启动，我们还必须*禁用*该服务。请注意这里发生了什么。当我们启用服务时创建的符号链接现在被删除了：

```
$> sudo systemctl disable my-daemon
Removed /etc/systemd/system/multi-user.target.wants/my-daemon.service.
```

## 它是如何工作的...

当我们启用或禁用一个服务时，systemd 会在*target*目录中创建一个符号链接。在我们的情况下，目标是*multi-user*，也就是当系统达到多用户级别时。

在第五步，当我们启动守护进程时，我们在状态输出中看到了*Main PID*。这个 PID 与守护进程创建的`/var/run/my-daemon.pid`文件中的 PID 匹配。这就是 systemd 如何跟踪*forking*守护进程的方式。在下一个教程中，我们将看到如何在 systemd 中创建一个不需要 fork 的守护进程。

# 为 systemd 创建一个更现代的守护进程

由 systemd 处理的守护进程不需要 fork 或关闭它们的文件描述符。相反，建议使用标准输出和标准错误将守护进程的日志写入日志。日志是 systemd 的日志记录设施。

在这个教程中，我们将编写一个新的守护进程，一个不会 fork 并留下`/tmp/my-daemon-is-alive.txt`文件的守护进程（与之前一样）。这种类型的守护进程有时被称为`my-daemon-v2.c`，被称为**SysV 风格守护进程**。**SysV**是 systemd 之前的 init 系统的名称。

## 准备工作

对于这个教程，你只需要本章节*技术要求*部分列出的内容。

## 如何做...

在这个教程中，我们将编写一个**新式守护进程**：

1.  这个程序有点长，所以我把它分成了几个步骤。将代码写入文件并保存为`new-style-daemon.c`。所有代码都放在一个文件中，即使有几个步骤。我们将首先编写所有的`include`语句，信号处理程序的函数原型和`main()`函数体。请注意，我们这里不进行 fork。我们也不关闭任何文件描述符或流。相反，我们将“*守护程序活着*”文本写入标准输出。请注意，我们需要在这里*刷新*stdout。通常，流是行缓冲的，这意味着它们在每个新行上都会被刷新。但是当 stdout 被重定向到其他地方时，比如使用 systemd，它会被完全缓冲。为了能够看到打印的文本，我们需要刷新它；否则，在停止守护程序或缓冲区填满之前，我们将看不到日志中的任何内容：

```
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <signal.h>
#include <time.h>
void sigHandler(int sig);
int main(void)
{
    time_t now; /* for the current time */
    struct sigaction action; /* for sigaction */
    /* prepare for sigaction */
    action.sa_handler = sigHandler;
    sigfillset(&action.sa_mask);
    action.sa_flags = SA_RESTART;
    /* register the signal handler */
    sigaction(SIGTERM, &action, NULL);
    sigaction(SIGUSR1, &action, NULL);
    sigaction(SIGHUP, &action, NULL);
    for (;;) /* main loop */
    {
        time(&now); /* get current date & time */
        printf("Daemon alive at %s", ctime(&now));
        fflush(stdout);
        sleep(30);
    }
    return 0;
}
```

1.  现在我们将编写信号处理程序的函数。请注意，我们在这里捕获了`SIGHUP`和`SIGTERM`。`SIGHUP`经常用于重新加载任何配置文件，而无需重新启动整个守护程序。捕获`SIGTERM`是为了让守护程序在自己之后进行清理（关闭所有打开的文件描述符或流并删除任何临时文件）。我们这里没有任何配置文件或临时文件，所以我们将消息打印到标准输出：

```
void sigHandler(int sig)
{
    if (sig == SIGUSR1)
    {
        printf("Hello world!\n");
    }
    else if (sig == SIGTERM)
    {
        printf("Doing some cleanup...\n");
        printf("Bye bye...\n");
        exit(0);
    }
    else if (sig == SIGHUP)
    {
        printf("HUP is used to reload any " 
            "configuration files\n");
    }
} 
```

1.  现在是时候编译守护程序，这样我们就可以使用它了：

```
$> make new-style-daemon
gcc -Wall -Wextra -pedantic -std=c99    new-style-daemon.c   -o new-style-daemon
```

1.  我们可以交互式运行它以验证它是否正常工作：

```
$> ./new-style-daemon 
Daemon alive at Sun Dec  6 18:51:47 2020
Ctrl+C
```

## 它是如何工作的...

这个守护程序的工作方式几乎与我们编写的任何其他程序一样。无需进行任何 forking、更改工作目录、关闭文件描述符或流，或者其他任何操作。它只是一个常规程序。

请注意，我们不在信号处理程序中刷新 stdout 缓冲区。每次程序接收到信号并打印消息时，程序都会回到`for`循环中，打印另一条“*守护程序活着*”消息，然后在`for`循环中的`fflush(stdout)`处刷新。如果信号是`SIGTERM`，则在`exit(0)`时刷新所有缓冲区，因此我们这里也不需要刷新。

在下一个食谱中，我们将使这个程序成为 systemd 服务。

## 另请参阅

您可以在`man 7 daemon`中获取更多深入的信息。

# 使新守护程序成为 systemd 服务

现在我们已经在上一个食谱中制作了一个**新式守护程序**，我们将看到为这个守护程序制作一个单元文件更容易。

了解如何编写单元文件以适应新式守护程序非常重要，因为越来越多的守护程序是以这种方式编写的。在为 Linux 制作新的守护程序时，我们应该以这种新的方式制作它们。

## 准备工作

对于这个食谱，您需要完成上一个食谱。我们将在这里使用那个食谱中的守护程序。

## 如何做...

在这里，我们将使**新式守护程序**成为 systemd 服务：

1.  让我们首先将守护程序移动到`/usr/local/sbin`，就像我们对传统守护程序所做的那样。请记住，您需要以 root 身份进行操作：

```
$> sudo mv new-style-daemon /usr/local/sbin/
```

1.  现在我们将编写新的单元文件。创建`/etc/systemd/system/new-style-daemon.service`文件，并给它以下内容。请注意，我们不需要在这里指定任何 PID 文件。另外，请注意，我们已将`Type=forking`更改为`Type=simple`。Simple 是 systemd 服务的默认类型：

```
[Unit]
Description=A new-style daemon for testing
[Service]
ExecStart=/usr/local/sbin/new-style-daemon
Restart=on-failure
Type=simple
[Install]
WantedBy=multi-user.target
```

1.  重新加载 systemd 守护程序，以便识别新的单元文件：

```
$> sudo systemctl daemon-reload
```

1.  启动守护程序，并检查其状态。请注意，我们也会在这里看到一个“*守护程序活着*”消息。这是日志中的一个片段。请注意，这次我们不会*启用*服务。除非我们希望它自动启动，否则我们不需要启用服务：

```
$> sudo systemctl start new-style-daemon
$> sudo systemctl status new-style-daemon
. new-style-daemon.service - A new-style daemon for testing
   Loaded: loaded (/etc/systemd/system/new-style-daemon.service; disabled; vendor preset: enabled
   Active: active (running) since Sun 2020-12-06 19:51:25 CET; 7s ago
 Main PID: 8421 (new-style-daemo)
    Tasks: 1 (limit: 4915)
   Memory: 244.0K
   CGroup: /system.slice/new-style-daemon.service
           └─8421 /usr/local/sbin/new-style-daemon
dec 06 19:51:25 red-dwarf systemd[1]: Started A new-style daemon for testing.
dec 06 19:51:25 red-dwarf new-style-daemon[8421]: Daemon alive at Sun Dec  6 19:51:25 2020
```

1.  让守护程序运行，并在下一个食谱中查看日志。

## 它是如何工作的...

由于这个守护程序没有 forking，systemd 可以在没有 PID 文件的情况下跟踪它。对于这个守护程序，我们使用了`Type=simple`，这是 systemd 中的默认类型。

当我们在*Step 4*中启动守护进程并检查其状态时，我们看到了“*守护进程活动*”消息的第一行。我们可以在不使用`sudo`的情况下查看守护进程的状态，但是我们就看不到日志的片段（因为它可能包含敏感数据）。

由于我们在`for`循环中的每个`printf()`后刷新了标准输出缓冲区，因此每次写入新条目时，日志都会实时更新。

在下一个步骤中，我们将查看日志。

# 阅读日志

在这个步骤中，我们将学习如何阅读日志。日志是 systemd 的日志记录设施。守护进程打印到标准输出或标准错误的所有消息都会添加到日志中。但是我们在这里可以找到的不仅仅是系统守护进程的日志。还有系统的引导消息，等等。

了解如何阅读日志可以让您更轻松地找到系统和守护进程中的错误。

## 准备工作

对于这个步骤，您需要`new-style-daemon`服务正在运行。如果您的系统上没有运行它，请返回到上一个步骤，了解如何启动它。

## 如何做...

在这个步骤中，我们将探讨如何阅读日志以及我们可以在其中找到什么样的信息。我们还将学习如何跟踪特定服务的日志：

1.  我们将首先检查来自我们的服务`new-style-daemon`的日志。 `-u`选项代表*单元*：

```
$> sudo journalctl -u new-style-daemon
```

现在日志可能已经很长了，所以您可以通过按*Spacebar*向下滚动日志。要退出日志，请按*Q*。

1.  请记住，我们为`SIGUSR1`实现了一个信号处理程序？让我们尝试向我们的守护进程发送该信号，然后再次查看日志。但是这次，我们将使用`--lines 5`仅显示日志中的最后五行。通过使用`systemctl status`找到进程的 PID。注意“*Hello world*”消息（在以下代码中已突出显示）：

```
$> systemctl status new-style-daemon
. new-style-daemon.service - A new-style daemon for testing
   Loaded: loaded (/etc/systemd/system/new-style-daemon.service; disabled; vendor preset: enabled
   Active: active (running) since Sun 2020-12-06 19:51:25 CET; 31min ago
 Main PID: 8421 (new-style-daemo)
    Tasks: 1 (limit: 4915)
   Memory: 412.0K
   CGroup: /system.slice/new-style-daemon.service
           └─8421 /usr/local/sbin/new-style-daemon
$> sudo kill -USR1 8421
$> sudo journalctl -u new-style-daemon --lines 5
-- Logs begin at Mon 2020-11-30 18:05:24 CET, end at Sun 2020-12-06 20:24:46 CET. --
dec 06 20:23:31 red-dwarf new-style-daemon[8421]: Daemon alive at Sun Dec  6 20:23:31 2020
dec 06 20:24:01 red-dwarf new-style-daemon[8421]: Daemon alive at Sun Dec  6 20:24:01 2020
dec 06 20:24:31 red-dwarf new-style-daemon[8421]: Daemon alive at Sun Dec  6 20:24:31 2020
dec 06 20:24:42 red-dwarf new-style-daemon[8421]: Hello world!
dec 06 20:24:42 red-dwarf new-style-daemon[8421]: Daemon alive at Sun Dec  6 20:24:42 2020
```

1.  还可以*跟踪*服务的日志，即“实时”查看。打开第二个终端并运行以下命令。`-f`代表*跟踪*：

```
$> sudo journalctl -u new-style-daemon -f
```

1.  现在，在第一个终端中，使用`sudo kill -USR1 8421`发送另一个`USR1`信号。您会立即在第二个终端中看到“*Hello world*”消息，而不会有任何延迟。要退出跟踪模式，只需按*Ctrl* + *C*。

1.  `journalctl`命令提供了广泛的过滤功能。例如，可以使用`--since`和`--until`仅选择两个日期之间的日志条目。也可以省略其中一个来查看自特定日期以来或直到特定日期的所有消息。在这里，我们展示了两个日期之间的所有消息：

```
$> sudo journalctl -u new-style-daemon \
> --since "2020-12-06 20:32:00" \
> --until "2020-12-06 20:33:00"
-- Logs begin at Mon 2020-11-30 18:05:24 CET, end at Sun 2020-12-06 20:37:01 CET. --
dec 06 20:32:12 red-dwarf new-style-daemon[8421]: Daemon alive at Sun Dec  6 20:32:12 2020
dec 06 20:32:42 red-dwarf new-style-daemon[8421]: Daemon alive at Sun Dec  6 20:32:42 2020
```

1.  通过省略`-u`选项和单元名称，我们可以查看所有服务的所有日志条目。试一下，用*Spacebar*滚动浏览。您还可以尝试只查看最后 10 行，就像我们之前用`--line 10`一样。

现在是时候停止`new-style-daemon`服务了。在停止服务后，我们还将查看日志中的最后五行。注意来自守护进程的告别消息。这是我们为`SIGTERM`信号制作的信号处理程序。当我们在 systemd 中停止服务时，它会发送一个`SIGTERM`信号给服务：

```
$> sudo systemctl stop new-style-daemon
$> sudo journalctl -u new-style-daemon --lines 5
-- Logs begin at Mon 2020-11-30 18:05:24 CET, end at Sun 2020-12-06 20:47:02 CET. --
dec 06 20:46:44 red-dwarf systemd[1]: Stopping A new-style daemon for testing...
dec 06 20:46:44 red-dwarf new-style-daemon[8421]: Doing some cleanup...
dec 06 20:46:44 red-dwarf new-style-daemon[8421]: Bye bye...
dec 06 20:46:44 red-dwarf systemd[1]: new-style-daemon.service: Succeeded.
dec 06 20:46:44 red-dwarf systemd[1]: Stopped A new-style daemon for testing.
```

## 工作原理...

由于日志负责处理所有发送到标准输出和标准错误的消息，我们不需要自己处理日志记录。这使得编写由 systemd 处理的 Linux 守护进程变得更容易。正如我们在查看日志时看到的那样，每条消息都有一个时间戳。这使得在寻找错误时可以轻松地过滤出特定的日期或时间。

使用`-f`选项跟踪特定服务的日志在尝试新的或未知服务时很常见。

## 另请参阅

`man journalctl`的手册页面上甚至有更多关于如何过滤日志的技巧和提示。
