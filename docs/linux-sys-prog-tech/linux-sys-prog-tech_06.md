# 第六章：生成进程和使用作业控制

在本章中，我们将了解系统上如何创建进程，哪个进程是第一个进程，以及所有进程如何相互关联。然后，我们将学习 Linux 中涉及进程和进程管理的许多术语。之后，我们将学习如何分叉新进程以及**僵尸**和**孤儿**是什么。在本章结束时，我们将学习**守护进程**是什么以及如何创建它，然后学习信号是什么以及如何实现它们。

了解系统上如何创建进程对于实现良好的守护进程、处理安全性和创建高效的程序至关重要。它还将让您更好地了解整个系统。在本章中，我们将涵盖以下示例：

+   探索进程是如何创建的

+   在 Bash 中使用作业控制

+   使用信号控制和终止进程

+   用`execl()`替换进程中的程序

+   分叉进程

+   在分叉进程中执行新程序

+   使用`system()`启动新进程

+   创建僵尸进程

+   了解孤儿进程是什么

+   创建守护进程

+   实现信号处理程序

让我们开始吧！

# 技术要求

在本章中，您将需要 GCC 编译器和 Make 工具。我们在[*第一章*]（B13043_01_Final_SK_ePub.xhtml#_idTextAnchor020）中安装了这些工具，获取必要的工具并编写我们的第一个 Linux 程序。

您还需要一个名为`pstree`的新程序来完成本章。您可以使用软件包管理器安装它。如果您使用的是 Debian 或 Ubuntu，可以使用`sudo apt install psmisc`进行安装。另一方面，如果您使用的是 Fedora 或 CentOS，可以使用`sudo dnf install psmisc`进行安装。

您还需要我们在[*第三章*]（B13043_03_Final_SK_ePub.xhtml#_idTextAnchor097）中编写的通用`Makefile`。Makefile 也可以在 GitHub 上找到，本章的所有代码示例也可以在[`github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch6`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch6)找到。

查看以下链接以查看代码演示视频：[`bit.ly/3cxY0eQ`](https://bit.ly/3cxY0eQ)

# 探索进程是如何创建的

在深入了解如何创建进程和守护进程之前，我们需要对进程有一个基本的理解。获得这种理解的最佳方法是查看系统上已经运行的进程，这就是我们将在本示例中要做的事情。

系统上的每个进程都是通过从另一个进程*生成*（forked）而开始其生命周期的。在 Unix 和 Linux 系统上使用的第一个进程历史上一直是`init`进程，现代 Linux 发行版已将其替换为**systemd**。它们都具有相同的目的；启动系统的其余部分。

典型的**进程树**可能如下所示，其中用户通过终端登录（即，如果我们跳过 X Window 登录的复杂性）：

```
|- systemd (1)
```

```
  \- login (6384)
```

```
    \- bash (6669)
```

```
      \- more testfile.txt (7184)
```

进程 ID 是括号中的数字。`systemd`（或一些旧系统上的`init`）有一个`init`，即使使用的是`systemd`。在这种情况下，`init`只是指向`systemd`的链接。仍然有一些使用`init`的 Linux 系统。

当涉及编写系统程序时，深入了解进程**生成**是至关重要的。例如，当我们想要创建一个守护进程时，我们经常会生成一个新进程。还有许多其他用例，我们必须生成进程或从现有进程执行新程序。

## 准备就绪

对于这个示例，您将需要`pstree`。`pstree`的安装说明列在本章的*技术要求*部分中。

## 如何做…

在这个示例中，我们将查看我们的系统和它运行的进程。我们将使用`pstree`来获得这些进程的可视化表示。让我们开始吧：

1.  首先，我们需要一种方法来获取我们当前的进程 ID。`$$`环境变量包含当前 shell 的**PID**。请注意，PID 在每个系统上以及从一次到另一次都会有所不同：

```
$> echo $$
18817
```

1.  现在，让我们用`pstree`来查看我们当前的进程，以及它的父进程和子进程。父进程是启动该进程的进程，而子进程是其下的任何进程：

```
$> pstree -A -p -s $$
systemd(1)---tmux (4050)---bash(18817)---pstree(18845)
```

1.  `pstree`命令的输出在您的计算机上很可能会有所不同。您可能有`xterm`、`konsole`、`mate-terminal`或类似的东西，而不是`tmux`。`-A`选项表示使用 ASCII 字符打印行，`-p`选项表示打印 PID 号，`-s`选项表示我们要显示所选进程的父进程（在我们的情况下是`$$`）。在我的例子中，`tmux`是`systemd`的子进程，`bash`是`tmux`的子进程，`pstree`是`bash`的子进程。

1.  一个进程也可以有多个子进程。例如，我们可以在 Bash 中启动多个进程。在这里，我们将启动三个`sleep`进程。每个`sleep`进程将休眠 120 秒。然后我们将打印另一个`pstree`。在这个例子中，`pstree`和三个`sleep`进程都是`bash`的子进程：

```
$> sleep 120 &
[1] 21902
$> sleep 120 &
[2] 21907
$> sleep 120 &
[3] 21913
$> pstree -A -p -s $$
systemd(1)---tmux (4050)---bash(18817)-+-pstree(21919)
                                       |-sleep(21902)
                                       |-sleep(21907)
                                       `-sleep(21913)
```

1.  在本章的开头，我们提供了一个显示名为`login`的进程的示例进程树。该进程最初是作为管理系统 TTY 的进程`getty`启动的。`getty`/`login`的概念，切换到 TTY3，使用*Ctrl*+*Alt*+*F3*进行激活。然后，返回到 X（通常在*Ctrl*+*Alt*+*F7*或*Ctrl*+*Alt*+*F1*）。在这里，我们将使用`grep`和`ps`来查找 TTY3 并记录其 PID。`ps`程序用于查找和列出系统上的进程。然后，我们将使用用户在 TTY3 上登录（*Ctrl*+*Alt*+*F3*）。之后，我们需要再次返回到我们的 X Window 会话（和我们的终端），并使用`grep`来找到我们从 TTY3 中记录的 PID。该进程中的程序现在已被替换为`login`。换句话说，一个进程可以替换其程序：

```
Ctrl+Alt+F3
login: 
Ctrl+Alt+F7
$> ps ax | grep tty3
9124 tty3     Ss+    0:00 /sbin/agetty -o -p -- \u --
noclear tty3 linux
Ctrl+Alt+F3
login: jake
Password: 
$> 
Ctrl+Alt+F7
$> ps ax | grep 9124
9124 tty3     Ss     0:00 /bin/login -p –
```

## 工作原理…

在这个教程中，我们学习了关于 Linux 系统上进程的几个重要概念。我们将需要这些知识继续前进。首先，我们了解到所有进程都是从现有进程中生成的。第一个进程是`init`。在较新的 Linux 发行版中，这是指向`systemd`的符号链接。然后，`systemd`在系统上生成几个进程，比如`getty`，来处理终端。当用户开始在 TTY 上登录时，`getty`会被`login`替换，这个程序处理登录。当用户最终登录时，`login`进程为用户生成一个 shell，比如 Bash。然后，每当用户执行一个程序时，Bash 会生成一个自身的副本，并用用户执行的程序替换它。

为了澄清一下进程/程序术语：`getty`/`login`示例。

在这个教程中使用 TTY3 的原因是，我们可以通过`getty`/`login`获得一个*真正的*登录过程，而在通过 X Window 会话或 SSH 登录时我们无法获得。

进程 ID 表示为 PID。父进程 ID 表示为`1`）。

我们还了解到一个进程可以有多个子进程，就像`sleep`进程的示例一样。我们在`sleep`进程的末尾使用`&`符号启动了`sleep`进程。这个&符号告诉 shell 我们要在后台启动该进程。

## 还有更多…

TTY 的首字母缩写来自于过去的实际*电传打字机*连接到机器上。电传打字机是一种看起来像打字机的终端。您在打字机上输入命令，然后在纸上读取响应。对于任何对电传打字机感兴趣的人，哥伦比亚大学在[`www.columbia.edu/cu/computinghistory/teletype.html`](http://www.columbia.edu/cu/computinghistory/teletype.html)上有一些令人兴奋的图片和信息。

# 在 Bash 中使用作业控制

作业控制不仅能让你更好地理解前台和后台进程，还能让你在终端上工作时更加高效。能够将一个进程放到后台可以让你的终端做其他任务。

## 准备工作

这个教程不需要特别的要求，除了 Bash shell。Bash 通常是默认的 shell，所以你很可能已经安装了它。

## 操作方法…

在这个教程中，我们将启动和停止几个进程，将它们发送到后台，并将它们带回前台。这将让我们了解后台和前台进程。让我们开始吧：

1.  之前，我们已经看到如何使用&在后台启动一个进程。我们将在这里重复这个步骤，但我们还将列出当前正在运行的作业，并将其中一个带到前台。我们将在这里启动的第一个后台进程是`sleep`，而另一个是手册页面：

```
$> sleep 300 &
[1] 30200
$> man ls &
[2] 30210
```

1.  现在我们在`jobs`中有两个进程：

```
$> jobs
[1]-  Running                 sleep 300 &
[2]+  Stopped                 man ls
```

1.  `sleep`进程处于运行状态，这意味着程序中的秒数正在减少。`man ls`命令已经停止了。`man`命令正在等待你对它做一些事情，因为它需要一个终端。所以，现在它什么也不做。我们可以使用`fg`命令（`fg`命令是`jobs`列表中的作业 ID）将它带到前台：

```
$> fg 2
```

1.  按*Q*退出手册页面。`man ls`将出现在屏幕上。

1.  现在，使用`fg 1`将`sleep`进程带到前台。它只显示`sleep 300`，没有更多的信息。但现在，程序在前台运行。这意味着我们现在可以按下*Ctrl*+*Z*来停止程序：

```
sleep 300
Ctrl+Z
[1]+  Stopped                 sleep 300
```

1.  程序已经停止，这意味着它不再倒计时。我们现在可以再次用`fg 1`将其带回前台并让它完成。

1.  现在上一个进程已经完成，让我们开始一个新的`sleep`进程。这次，我们可以在前台启动它（省略了&）。然后，我们可以按下*Ctrl*+*Z*来停止程序。列出作业并注意程序处于停止状态：

```
$> sleep 300
Ctrl+Z
[1]+  Stopped                 sleep 300
$> jobs
[1]+  Stopped                 sleep 300
```

1.  现在，我们可以使用`bg`命令在后台继续运行程序（`bg`代表*background*）：

```
$> bg 1
[1]+ sleep 300 &
$> jobs
[1]+  Running                 sleep 300 &
```

1.  我们还可以使用一个叫做`pgrep`的命令来找到程序的 PID。`pgrep`的名称代表*Process Grep*。`-f`选项允许我们指定完整的命令，包括它的选项，以便我们得到正确的 PID：

```
$> pgrep -f "sleep 300"
4822
```

1.  现在我们知道了 PID，我们可以使用`kill`来终止程序：

```
$> kill 4822
$> Enter
[1]+  Terminated              sleep 300
```

1.  我们也可以使用`pkill`来终止一个程序。在这里，我们将启动另一个进程，并使用`pkill`来终止它。这个命令和`pgrep`使用相同的选项：

```
$> sleep 300 &
[1] 6526
$> pkill -f "sleep 300"
[1]+  Terminated              sleep 300
```

## 工作原理…

在这个教程中，我们学习了后台进程、前台进程、停止和运行的作业、终止进程等基本概念。这些是 Linux 作业控制中使用的一些基本概念。

当我们用`kill`杀死进程时，`kill`向后台进程发送了一个信号。`kill`的默认信号是`TERM`信号。`TERM`信号是 15 号信号。一个无法处理的信号——总是终止程序的信号是 9 号信号，或者`KILL`信号。我们将在下一个教程中更深入地介绍信号处理。

# 使用信号来控制和终止进程

现在我们对进程有了一些了解，是时候转向信号并学习如何使用信号来终止和控制进程了。在这个教程中，我们还将编写我们的第一个 C 程序，其中将包含一个信号处理程序。

## 准备工作

对于这个教程，你只需要本章节*技术要求*部分列出的内容。

## 操作方法…

在这个教程中，我们将探讨如何使用信号来控制和终止进程。让我们开始吧：

1.  让我们首先列出我们可以使用`kill`命令发送给进程的信号。从这个命令得到的列表相当长，所以这里没有包含。最有趣和使用的信号是前 31 个：

```
$> kill -L
```

1.  让我们看看这些信号是如何工作的。我们可以向一个进程发送`STOP`信号（编号 19），这与我们在`sleep`中按下*Ctrl*+*Z*看到的效果相同。但是这里，我们直接向一个后台进程发送`STOP`信号：

```
$> sleep 120 &
[1] 16392
$> kill -19 16392
 [1]+  Stopped                 sleep 120
$> jobs
[1]+  Stopped                 sleep 120
```

1.  现在，我们可以通过发送`CONT`信号（**continue**的缩写）来继续进程。如果愿意，我们也可以输入信号的名称，而不是它的编号：

```
$> kill -CONT 16392
$> jobs
[1]+  Running                 sleep 120 &
```

1.  现在，我们可以通过发送`KILL`信号（编号 9）来终止进程：

```
$> kill -9 16392
$> Enter
[1]+  Killed                  sleep 120
```

1.  现在，让我们创建一个根据不同信号执行操作并忽略（或阻塞）*Ctrl*+*C*（中断信号）的小程序。`USR1`和`USR2`信号非常适合这个目的。将以下代码写入一个文件并保存为`signals.c`。这里将这段代码分成了多个步骤，但所有代码都放在这个文件中。要在程序中注册信号处理程序，我们可以使用`sigaction()`系统调用。由于`sigaction()`及其相关函数不包含在严格的 C99 中，我们需要定义`_POSIX_C_SOURCE`。我们还需要包含必要的头文件，编写处理程序函数原型，并开始`main()`函数：

```
#define _POSIX_C_SOURCE 200809L
#include <stdio.h>
#include <sys/types.h>
#include <signal.h>
#include <unistd.h>
void sigHandler(int sig);
int main(void)
{
```

1.  现在，让我们创建一些我们需要的变量和结构。我们将创建的`sigaction`结构`action`是为了`sigaction()`系统调用。在代码中稍后一点，我们设置它的成员。首先，我们必须将`sa_handler`设置为我们的函数，当接收到信号时将执行该函数。其次，我们使用`sigfillset()`将`sa_mask`设置为所有信号。这将在执行我们的信号处理程序时忽略所有信号，防止它被中断。第三，我们将`sa_flags`设置为`SA_RESTART`，这意味着任何中断的系统调用将被重新启动：

```
    pid_t pid; /* to store our pid in */
    pid = getpid(); /* get the pid */
    struct sigaction action; /* for sigaction */
    sigset_t set; /* signals we want to ignore */
    printf("Program running with PID %d\n", pid);
    /* prepare sigaction() */
    action.sa_handler = sigHandler;
    sigfillset(&action.sa_mask);
    action.sa_flags = SA_RESTART;
```

1.  现在，是时候使用`sigaction()`注册信号处理程序了。`sigaction()`的第一个参数是我们想要捕获的信号，第二个参数是新操作的结构，第三个参数给出了旧操作。如果我们对旧操作不感兴趣，我们将其设置为`NULL`。操作必须是`sigaction`结构：

```
    /* register two signal handlers, one for USR1
       and one for USR2 */
    sigaction(SIGUSR1, &action, NULL);
    sigaction(SIGUSR2, &action, NULL);
```

1.  记住我们希望程序忽略*Ctrl*+*C*（中断信号）吗？这可以通过在应该忽略信号的代码之前调用`sigprocmask()`来实现。但首先，我们必须创建一个包含所有应该忽略/阻塞的信号的*信号集*。首先，我们将使用`sigemptyset()`清空集合，然后使用`sigaddset()`添加所需的信号。`sigaddset()`函数可以多次调用以添加更多的信号。`sigprocmask()`的第一个参数是行为，这里是`SIG_BLOCK`。第二个参数是信号集，而第三个参数可以用于检索旧集。但是，在这里，我们将其设置为`NULL`。之后，我们开始无限的`for`循环。循环结束后，我们再次解除信号集的阻塞。在这种情况下，这是不必要的，因为我们将退出程序，但在其他情况下，建议在我们已经过了应该忽略它们的代码部分后解除信号的阻塞：

```
    /* create a "signal set" for sigprocmask() */
    sigemptyset(&set);
    sigaddset(&set, SIGINT);
    /* block SIGINT and run an infinite loop */
    sigprocmask(SIG_BLOCK, &set, NULL);
    /* infinite loop to keep the program running */
    for (;;)
    {
        sleep(10);
    }
    sigprocmask(SIG_UNBLOCK, &set, NULL);
    return 0;
}
```

1.  最后，让我们编写将在`SIGUSR1`和`SIGUSR2`上执行的函数。该函数将打印接收到的信号：

```
void sigHandler(int sig)
{
    if (sig == SIGUSR1)
    {
        printf("Received USR1 signal\n");
    }
    else if (sig == SIGUSR2)
    {
        printf("Received USR2 signal\n");
    }
}
```

1.  让我们编译程序：

```
$> make signals
gcc -Wall -Wextra -pedantic -std=c99    signals.c   -o
 signals
```

1.  运行程序，可以在单独的终端或者在同一个终端的后台运行。请注意，我们在这里使用`kill`命令的信号名称；这比跟踪数字要容易一些：

```
$> ./signals &
[1] 25831
$> Program running with PID 25831
$> kill -USR1 25831
Received USR1 signal
$> kill -USR1 25831
Received USR1 signal
$> kill -USR2 25831
$> kill -USR2 25831
Received USR2 signal
$> Ctrl+C
^C
$> kill -USR1 25831
Received USR1 signal
$> kill -TERM 25831
$> ENTER 
[1]+  Terminated              ./signals
```

## 工作原理…

首先，我们探索了许多`TERM`、`KILL`、`QUIT`、`STOP`、`HUP`、`INT`、`STOP`和`CONT`，就像我们在这里看到的那样。

然后，我们使用`STOP`和`CONT`信号来实现与上一个示例相同的效果；也就是说，停止和继续运行后台进程。在上一个示例中，我们使用`bg`来继续在后台运行进程，而要停止进程，我们按下*Ctrl*+*Z*。这一次，我们不需要将程序打开在前台来停止它；我们只需用`kill`发送`STOP`信号。

之后，我们继续编写了一个 C 程序，捕获了两个信号`USR1`和`USR2`，并阻止了`SIGINT`信号（*Ctrl*+*C*）。根据我们发送给程序的信号，将打印不同的文本。我们通过实现信号处理程序来实现这一点。一个`sigaction()`函数。

在调用`sigaction()`系统调用之前，我们必须使用有关处理程序函数的信息填充`sigaction`结构，该结构在处理程序执行期间忽略的信号，以及它应该具有的行为。

信号集，无论是 sigaction 的`sa_mask`还是`sigprocmask()`，都是使用`sigset_t`类型创建的，并通过以下函数调用进行操作（在这里，我们假设使用了名为`s`的`sigset_t`变量：

+   `sigemptyset(&s);`清除`s`中的所有信号

+   `sigaddset(&s, SIGUSR1);`将`SIGUSR1`信号添加到`s`

+   `sigdelset(&s, SIGUSR1);`从`s`中删除`SIGUSR`信号

+   `sigfillset(&s);`设置`s`中的所有信号

+   `sigismember(&s, SIGUSR1);`找出`SIGUSR1`是否是`s`的成员（在我们的示例代码中未使用）

要在进程启动时打印进程的 PID，我们必须使用`getpid()`系统调用来获取 PID。我们将 PID 存储在`pid_t`类型的变量中，就像我们之前看到的那样。

## 另请参阅

在`kill`、`pkill`、`sigprocmask()`和`sigaction()`系统调用的手册页中有很多有用的信息。我建议您使用以下命令阅读它们：

+   `man 1 kill`

+   `man 1 pkill`

+   `man 2 sigprocmask`

+   `man 2 sigaction`

还有一个更简单的系统调用，称为`signal()`，也用于信号处理。如今，这个系统调用基本上被认为是不推荐使用的。但如果您感兴趣，可以在`man 2 signal`中阅读相关信息。

# 使用 execl()在进程中替换程序

在本章的开头，我们看到当用户登录时，`getty`被`login`替换。在这个示例中，我们将编写一个小程序，正好可以做到这一点——用新程序替换其程序。这个系统调用被称为`execl()`。

了解如何使用`execl()`使您能够编写在现有进程内执行新程序的程序。它还使您能够在生成的进程中启动新程序。当我们启动一个新进程时，我们可能希望用新程序替换该副本。因此，理解`execl()`是至关重要的。

## 准备就绪

您需要阅读本章的前三个示例，才能充分理解这个示例。本示例的其他要求在本章的*技术要求*部分中提到；例如，您将需要`pstree`工具。

您还需要两个终端或两个终端窗口。在其中一个终端中，我们将运行程序，而在另一个终端中，我们将查看`pstree`以查看进程。

## 如何做…

在这个示例中，我们将编写一个小程序，用它替换进程中正在运行的程序。让我们开始吧：

1.  在文件中编写以下代码并将其保存为`execdemo.c`：

```
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
int main(void)
{
   printf("My PID is %d\n", getpid());
   printf("Hit enter to continue ");
   getchar(); /* wait for enter key */
   printf("Executing /usr/bin/less...\n");
   /* execute less using execl and error check it */
   if ( execl("/usr/bin/less", "less", 
      "/etc/passwd", (char*)NULL) == -1 )
   {
      perror("Can't execute program");
      return 1;
   }
   return 0;
}
```

1.  使用 Make 编译程序：

```
$> make execdemo
gcc -Wall -Wextra -pedantic -std=c99    execdemo.c   -o execdemo
```

1.  现在，在*当前*终端中运行程序：

```
$> ./execdemo
My PID is 920
Hit enter to continue
```

1.  现在，启动一个*新*终端，并使用`execdemo`的 PID 执行`pstree`：

```
$> pstree -A -p -s 920
systemd(1)---tmux(4050)---bash(18817)---execdemo(920)
```

1.  现在，回到运行`execdemo`的第一个终端，并按*Enter*。这将使用`less`打印密码文件。

1.  最后，回到第二个终端——您运行`pstree`的终端。重新运行相同的`pstree`命令。请注意，即使 PID 仍然相同，`execdemo`已被替换为`less`：

```
$> pstree -A -p -s 920
systemd(1)---tmux(4050)---bash(18817)---less(920)
```

## 它是如何工作的…

`execl()`函数执行一个新程序，并在同一个进程中替换旧程序。为了让程序暂停执行，以便我们有时间在`pstree`中查看它，我们使用了`getchar()`。

`execl()`函数有四个必需的参数。第一个是我们想要执行的程序的路径。第二个参数是程序的名称，就像从`argv[0]`中打印出来的那样。最后，第三个和之后的参数是我们想要传递给即将执行的程序的参数。为了*终止*我们想要传递给程序的参数列表，我们必须以`NULL`的指针结束，并将其转换为`char`类型。

另一种看待一个进程的方式是把它看作一个执行环境。在这个环境中运行的程序可以被替换。这就是为什么我们谈论进程，为什么我们称它们为*Process IDs*，而不是 Program IDs。

## 另请参阅

还有其他几个`exec()`函数可以使用，每个函数都有自己独特的特性和特点。这些通常被称为"`exec()` family"。你可以使用`man 3 execl`命令来了解它们的所有信息。

# fork 一个进程

之前，我们一直在说当一个程序创建一个新的进程时使用*spawned*。正确的术语是**fork**一个进程。发生的情况是一个进程创建了自己的一个副本——它*forks*。

在之前的教程中，我们学习了如何使用`execl()`在一个进程中执行一个新程序。在这个教程中，我们将学习如何使用`fork()`来 fork 一个进程。被 fork 的进程——子进程——是调用进程——父进程——的一个副本。

知道如何 fork 一个进程使我们能够在系统中以编程方式创建新的进程。如果不能 fork，我们只能限制在一个进程中。例如，如果我们想要从一个现有的程序中启动一个新程序并保留原始程序，我们必须 fork。

## 准备工作

就像在之前的教程中一样，你需要`pstree`工具。*技术要求*部分介绍了如何安装它。你还需要 GCC 编译器和 Make 工具。你还需要两个终端；一个终端用来执行程序，另一个用来用`pstree`查看进程树。

## 如何做...

在这个教程中，我们将使用`fork()`来 fork 一个进程。我们还将查看一个进程树，以便我们可以看到发生了什么。让我们开始吧：

1.  在一个程序中写下以下代码并保存为`forkdemo.c`。这段代码中突出显示了`fork()`系统调用。在我们`fork()`之前，我们打印出进程的 PID：

```
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(void)
{
   pid_t pid;
   printf("My PID is %d\n", getpid());
   /* fork, save the PID, and check for errors */
   if ( (pid = fork()) == -1 )
   { 
      perror("Can't fork");
      return 1;
   }
   if (pid == 0)
   {
      /* if pid is 0 we are in the child process */
      printf("Hello from the child process!\n");
      sleep(120);
   }

   else if(pid > 0)
   {
      /* if pid is greater than 0 we are in 
       * the parent */
      printf("Hello from the parent process! "
         "My child has PID %d\n", pid);
      sleep(120);
   }
   else
   {
      fprintf(stderr, "Something went wrong "
         "forking\n");
      return 1;
   }
   return 0;
}
```

1.  现在，编译程序：

```
$> make forkdemo
gcc -Wall -Wextra -pedantic -std=c99    forkdemo.c   
-o forkdemo
```

1.  在你*当前*的终端中运行程序并注意 PID：

```
$> ./forkdemo 
My PID is 21764
Hello from the parent process! My child has PID 21765
Hello from the child process!
```

1.  现在，在一个新的终端中，用`forkdemo`的 PID 运行`pstree`。在这里，我们可以看到`forkdemo`已经 fork 了，而我们在 fork 之前从程序中得到的 PID 是父进程。fork 的进程是正在运行的`forkdemo`：

```
$> pstree -A -p -s 21764
systemd(1)---tmux(4050)---bash(18817)---
forkdemo(21764)---forkdemo(21765)
```

## 它是如何工作的...

当一个进程 fork 时，它创建了自己的一个副本。这个副本成为调用`fork()`的进程的子进程——`fork()`返回子进程的 PID。在子进程中，返回`0`。这就是为什么父进程可以打印出子进程的 PID。

两个进程包含相同的程序代码，两个进程都在运行，但只有`if`语句中的特定部分会被执行，这取决于进程是父进程还是子进程。

## 还有更多...

一般来说，父进程和子进程是相同的，除了 PID。然而，还有一些其他的差异；例如，子进程中的 CPU 计数器会被重置。还有其他一些微小的差异，你可以在`man 2 fork`中了解到。然而，整个程序代码是相同的。

# 在一个 forked 进程中执行一个新程序

在上一个示例中，我们学习了如何使用`fork()`系统调用分叉进程。在之前的示例中，我们学习了如何用`execl()`替换进程中的程序。在这个示例中，我们将结合这两个，`fork()`和`execl()`，在一个分叉的进程中执行一个新程序。这就是每次在 Bash 中运行程序时发生的事情。Bash 分叉自身并执行我们输入的程序。

了解如何使用`fork()`和`execl()`使您能够编写启动新程序的程序。例如，您可以使用这些知识编写自己的 shell。

## 准备工作

对于这个示例，您需要`pstree`工具、GCC 编译器和 Make 工具。您可以在本章的*技术要求*部分找到这些程序的安装说明。

## 操作步骤…

在这个示例中，我们将编写一个程序，`fork()`并在子进程中执行一个新程序。让我们开始吧：

1.  在文件中写入以下程序代码，并将其保存为`my-fork.c`。当我们在子进程中执行一个新程序时，我们应该等待子进程完成。这就是我们使用`waitpid()`的方式。`waitpid()`调用还有另一个重要的功能，即从子进程获取返回状态：

```
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <string.h>
#include <sys/wait.h>
int main(void)
{
   pid_t pid;
   int status;
   /* Get and print my own pid, then fork
      and check for errors */
   printf("My PID is %d\n", getpid());
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
   if (pid == 0)
   {
      /* If pid is 0 we are in the child process,
         from here we execute 'man ls' */
      if ( execl("/usr/bin/man", "man", "ls",
         (char*)NULL) == -1 )
      {
         perror("Can't exec");
         return 1;
      }
   }
   else if(pid > 0)
   {
      /* In the parent we must wait for the child
         to exit with waitpid(). Afterward, the
         child exit status is written to 'status' */
      waitpid(pid, &status, 0);
      printf("Child executed with PID %d\n", pid);
      printf("Its return status was %d\n", status);
      printf("Its return status was %d\n", status);
   }
   else
   {
      fprintf(stderr, "Something went wrong "
         "forking\n");
      return 1;
   }
   return 0;
}
```

1.  使用 Make 编译程序：

```
$> make my-fork
gcc -Wall -Wextra -pedantic -std=c99    my-fork.c   -o
my-fork
```

1.  在当前的终端中，找到当前 shell 的 PID 并做个记录：

```
$> echo $$
18817
```

1.  现在，使用`./my-fork`执行我们编译的程序。这将显示`ls`的手册页。

1.  打开一个新的终端，查看另一个终端中 shell 的进程树。注意，`my-fork`已经分叉并用`man`替换了其内容，`man`又分叉并用`pager`替换了其内容（以显示内容）：

```
$> pstree -A -p -s 18817
systemd(1)---tmux(4050)---bash(18817)---my-fork(5849)-
--man(5850)---pager(5861)
```

1.  通过按下*Q*退出第一个终端中的手册页。这将产生以下文本。比较`pstree`中父进程和子进程的 PID。注意子进程是`5850`，这是`man`命令。它最初是`my-fork`的副本，但后来用`man`替换了其程序：

```
My PID is 5849
Child executed with PID 5850
Its return status was 0
```

## 它是如何工作的…

`fork()`系统调用负责在 Linux 和 Unix 系统上分叉进程。然后，`execl()`（或其他`exec()`函数之一）负责执行并用新程序替换自己的程序。这基本上是系统上任何程序启动的方式。

请注意，我们需要告诉父进程使用`waitpid()`等待子进程。如果我们需要运行一个不需要终端的程序，我们可以不使用`waitpid()`。但是，我们应该始终等待子进程。如果不等待，子进程将最终成为**孤儿**。这是我们将在本章后面详细讨论的内容，在*学习孤儿是什么*这个示例中。

但在这种特殊情况下，我们执行需要终端的`man`命令，我们需要等待子进程才能让一切正常工作。`waitpid()`调用还使我们能够获取子进程的*返回状态*。我们还防止子进程变成孤儿。

当我们运行程序并用`pstree`查看进程树时，我们发现`my-fork`进程已经分叉并用`man`替换了其程序。我们可以看到这一点，因为`man`命令的 PID 与`my-fork`的子进程的 PID 相同。我们还注意到`man`命令反过来又分叉并用`pager`替换了其子进程。`pager`命令负责在屏幕上显示实际文本，通常是`less`。

# 使用 system()启动一个新进程

我们刚刚讨论的使用`fork()`、`waitpid()`和`execl()`在分叉的进程中启动新程序的内容是理解 Linux 和进程更深层次的关键。这种理解是成为优秀系统开发人员的关键。但是，有一个捷径。我们可以使用`system()`来代替手动处理分叉、等待和执行。`system()`函数为我们完成所有这些步骤。

## 准备工作

对于这个示例，你只需要本章节*技术要求*部分中列出的内容。

## 如何做…

在这个示例中，我们将使用`system()`函数重写前一个程序`my-fork`。你会注意到这个程序与前一个程序相比要短得多。让我们开始吧：

1.  将以下代码写入文件并保存为`sysdemo.c`。注意这个程序有多小（和简单）。`system()`函数为我们完成了所有复杂的工作：

```
#include <stdio.h>
#include <stdlib.h>
int main(void)
{
   if ( (system("man ls")) == -1 )
   {
      fprintf(stderr, "Error forking or reading "
         "status\n");
      return 1;
   }
   return 0;
}
```

1.  编译程序：

```
$> make sysdemo
gcc -Wall -Wextra -pedantic -std=c99    sysdemo.c   -o
sysdemo
```

1.  使用`$$`变量记录 shell 的 PID：

```
$> echo $$
957
```

1.  现在在当前终端中运行程序。这将显示`ls`命令的手册页。让它继续运行：

```
$> ./sysdemo
```

1.  在新终端中启动并对*步骤 3*中的 PID 执行`pstree`。请注意，这里有一个额外的名为`sh`的进程。这是因为`system()`函数从`sh`（基本的 Bourne Shell）执行`man`命令：

```
$> pstree -A -p -s 957
systemd(1)---tmux(4050)---bash(957)---sysdemo(28274)--
-sh(28275)---man(28276)---pager(28287)
```

## 它是如何工作的…

这个程序要小得多，编写起来也更容易。然而，正如我们在`pstree`中看到的那样，与上一个示例相比，有一个额外的进程：`sh`（shell）。`system()`函数通过从`sh`执行`man`命令来工作。手册页（`man 3 system`）清楚地说明了这一点。它通过以下`execl()`调用执行我们指定的命令：

```
execl("/bin/sh", "sh", "-c", command, (char *) 0);
```

结果是一样的。它执行`fork()`，然后是`execl()`调用，并且使用`waitpid()`等待子进程。这也是一个使用低级系统调用的高级函数的很好的例子。

# 创建一个僵尸进程

要完全理解 Linux 中的进程，我们还需要看看什么是僵尸进程。为了完全理解这一点，我们需要自己创建一个。

**僵尸**进程是指子进程在父进程之前退出，而父进程没有等待子进程的状态。"僵尸进程"这个名字来源于这个事实，即进程是*不死的*。进程已经退出，但在系统进程表中仍然有一个条目。

了解什么是僵尸进程以及它是如何创建的将有助于你避免编写在系统上创建僵尸进程的糟糕程序。

## 准备工作

对于这个示例，你只需要本章节*技术要求*部分中列出的内容。

## 如何做…

在这个示例中，我们将编写一个小程序，在系统上创建一个僵尸进程。我们还将使用`ps`命令查看僵尸进程。为了证明我们可以通过等待子进程来避免僵尸进程，我们还将使用`waitpid()`编写第二个版本。让我们开始吧：

1.  将以下代码写入文件并命名为`create-zombie.c`。这个程序与我们在`forkdemo.c`文件中看到的程序相同，只是子进程在父进程退出之前使用`exit(0)`退出。父进程在子进程退出后睡眠 2 分钟，而不等待子进程使用`waitpid()`，从而创建一个僵尸进程。这里突出显示了`exit()`的调用：

```
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
int main(void)
{
   pid_t pid;
   printf("My PID is %d\n", getpid());
   /* fork, save the PID, and check for errors */
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
   if (pid == 0)
   {
      /* if pid is 0 we are in the child process */
      printf("Hello and goodbye from the child!\n");
      exit(0);
      /* if pid is greater than 0 we are in 
       * the parent */
      printf("Hello from the parent process! "
         "My child had PID %d\n", pid);
      sleep(120);
   }
   else 
   {
      fprintf(stderr, "Something went wrong "
         "forking\n");
      return 1;
   }
   return 0;
}
```

1.  编译程序：

```
$> make create-zombie
gcc -Wall -Wextra -pedantic -std=c99    create-
zombie.c   -o create-zombie
```

1.  在当前终端中运行程序。程序（父进程）将保持活动状态 2 分钟。与此同时，子进程是僵尸的，因为父进程没有等待它或它的状态：

```
$> ./create-zombie
My PID is 2429
Hello from the parent process! My child had PID 2430
Hello and goodbye from the child!
```

1.  当程序正在运行时，打开另一个终端并使用`ps`检查子进程的 PID。你可以从`create-zombie`之前的输出中得到子进程的 PID。在这里，我们可以看到进程是僵尸的，因为它的状态是`Z+`，并且在进程名后面有`<defunct>`这个词：

```
$> ps a | grep 2430
  2430 pts/18   Z+     0:00 [create-zombie] <defunct>
  2824 pts/34   S+     0:00 grep 2430
```

1.  2 分钟后——当父进程执行完毕时——使用相同的 PID 重新运行`ps`命令。僵尸进程现在将不复存在：

```
$> ps a | grep 2430
  3364 pts/34   S+     0:00 grep 2430
```

1.  现在，重写程序，使其如下所示。将新版本命名为`no-zombie.c`。这里突出显示了添加的代码：

```
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
int main(void)
{
   pid_t pid;
   int status;
   printf("My PID is %d\n", getpid());
   /* fork, save the PID, and check for errors */
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
   if (pid == 0)
   {
      /* if pid is 0 we are in the child process */
      printf("Hello and goodbye from the child!\n");
      exit(0);
   }
   else if(pid > 0)
   {
      /* if pid is greater than 0 we are in 
       * the parent */
      printf("Hello from the parent process! "
         "My child had PID %d\n", pid);
      waitpid(pid, &status, 0); /* wait for child */
      sleep(120);
   }
   else
   {
      fprintf(stderr, "Something went wrong "
         "forking\n");
      return 1;
   }
   return 0;
}
```

1.  编译这个新版本：

```
$> make no-zombie
gcc -Wall -Wextra -pedantic -std=c99    no-zombie.c  
-o no-zombie
```

1.  在当前终端中运行程序。就像以前一样，它将创建一个子进程，该子进程将立即退出。父进程将继续运行 2 分钟，给我们足够的时间来搜索子进程的 PID：

```
$> ./no-zombie
My PID is 22101
Hello from the parent process! My child had PID 22102
Hello and goodbye from the child!
```

1.  当`no-zombie`程序正在运行时，在新的终端中使用`ps`和`grep`搜索子进程的 PID。正如你所看到的，没有与子进程的 PID 匹配的进程。因此，由于父进程等待其状态，子进程已正确退出：

```
$> ps a | grep 22102
22221 pts/34   S+     0:00 grep 22102
```

## 工作原理…

我们始终希望避免在系统上创建僵尸进程，而最好的方法是等待子进程完成。

在*步骤 1 到 5*中，我们编写了一个创建僵尸进程的程序。由于父进程没有使用`waitpid()`系统调用等待子进程，因此创建了僵尸进程。子进程确实退出了，但它仍然留在系统进程表中。当我们使用`ps`和`grep`搜索进程时，我们看到子进程的状态为`Z+`，表示僵尸。该进程不存在，因为它已经使用`exit()`系统调用退出。但是，根据系统进程表，它仍然存在；因此，它是不死不活的—一个僵尸。

在*步骤 6 到 9*中，我们使用`waitpid()`系统调用重写了程序以等待子进程。子进程仍然在父进程之前存在，但这次父进程获得了子进程的状态。

僵尸进程不会占用任何系统资源，因为进程已经终止。它只驻留在系统进程表中。但是，系统上的每个进程—包括僵尸进程—都占用一个 PID 号。由于系统可用的 PID 号是有限的，如果死进程占用 PID 号，就有耗尽 PID 号的风险。

## 还有更多…

在 Linux 的`waitpid()`手册页中有关于子进程及其状态变化的许多细节。实际上，在 Linux 中有三个可用的`wait()`函数。你可以使用`man 2 wait`命令阅读有关它们的所有内容。

# 了解孤儿的含义

了解 Linux 系统中孤儿的含义就像了解僵尸一样重要。这将使你更深入地了解整个系统以及进程如何被`systemd`继承。

一个`systemd`，它是系统上的第一个进程—PID 为`1`。

在本食谱中，我们将编写一个小程序，该程序分叉，从而创建一个子进程。然后父进程将退出，将子进程留下来作为孤儿。

## 准备就绪

本章的*技术要求*部分列出了本食谱所需的一切。

## 如何做…

在本食谱中，我们将编写一个创建孤儿进程的简短程序，该进程将由`systemd`继承。让我们开始吧：

1.  在文件中编写以下代码并将其保存为`orphan.c`。该程序将创建一个在后台运行 5 分钟的子进程。当我们按下*Enter*时，父进程将退出。这给了我们时间在父进程退出之前和之后使用`pstree`调查子进程：

```
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
int main(void)
{
   pid_t pid;
   printf("Parent PID is %d\n", getpid());
   /* fork, save the PID, and check for errors */
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
   if (pid == 0)
   {
      /* if pid is 0 we are in the child process */
      printf("I am the child and will run for "
         "5 minutes\n");
      sleep(300);
      exit(0);
   }
   else if(pid > 0)
   {
      /* if pid is greater than 0 we are in 
       * the parent */
      printf("My child has PID %d\n" 
         "I, the parent, will exit when you "
         "press enter\n", pid);
      getchar();
      return 0;
   }
   else
   {
      fprintf(stderr, "Something went wrong "
         "forking\n");
      return 1;
   }
   return 0;
}
```

1.  编译此程序：

```
$> make orphan
gcc -Wall -Wextra -pedantic -std=c99    orphan.c   -o
 orphan
```

1.  在当前终端中运行程序并让程序继续运行。暂时不要按*Enter*：

```
$> ./orphan
My PID is 13893
My child has PID 13894
I, the parent, will exit when you press enter
I am the child and will run for 2 minutes
```

1.  现在，在一个新的终端中，使用子进程的 PID 运行`pstree`。在这里，我们将看到它看起来就像在之前的食谱中一样。进程已经被分叉，从而创建了一个具有相同内容的子进程：

```
$> pstree -A -p -s 13894
systemd(1)---tmux(4050)---bash(18817)---orphan(13893)-
--orphan(13894)
```

1.  现在，是时候结束父进程了。回到`orphan`仍在运行的终端并按下*Enter*。这将结束父进程。

1.  现在，在第二个终端中再次运行`pstree`。这与刚刚运行的命令相同。正如你所看到的，子进程现在已被`systemd`继承，因为其父进程已经死亡。5 分钟后，子进程将退出：

```
$> pstree -A -p -s 13894
systemd(1)---orphan(13894)
```

1.  我们可以使用其他更标准化的工具来查看`ps`。运行以下`ps`命令以查看有关子进程的更详细信息。在这里，我们将看到更多信息。对我们来说最重要的是 PPID、PID 和**会话 ID**（**SID**）。我们还将在这里看到**用户 ID**（**UID**），它指定了谁拥有该进程：

```
$> ps jp 13894
PPID PID PGID  SID   TTY  TPGID STAT UID TIME COMMAND
1  13894 13893 18817 pts/18 18817 S 1000 0:00 ./orphan
```

## 工作原理…

每个进程都需要一个父进程。这就是为什么`systemd`会继承系统上任何成为孤儿的进程的原因。

`if (pid == 0)`中的代码继续运行了 5 分钟。这给了我们足够的时间来检查子进程是否已被`systemd`继承。

在最后一步，我们使用`ps`查看了有关子进程的更多详细信息。在这里，我们看到了 PPID、PID、PGID 和 SID。这里提到了一些重要的新名称。我们已经知道 PPID 和 PID，但 PGID 和 SID 还没有被介绍过。

**PGID**代表**进程组 ID**，是系统对进程进行分组的一种方式。子进程的 PGID 是父进程的 PID。换句话说，这个 PGID 是为了将父进程和子进程分组在一起而创建的。系统将 PGID 设置为创建该组的父进程的 PID。我们不需要自己创建这些组；这是系统为我们做的事情。

`18817`，这是 Bash shell 的 PID。这里也适用相同的规则；SID 号将与启动会话的进程的 PID 相同。这个会话包括我的用户 shell 和我从中启动的所有程序。这样，系统就可以在我注销系统时终止属于该会话的所有进程。

## 另请参阅

使用`ps`可以获得很多信息。我建议你至少浏览一下`man 1 ps`的手册。

# 创建守护进程

在系统编程中常见的任务是创建各种守护进程。**守护进程**是在系统上运行并执行一些任务的后台进程。SSH 守护进程就是一个很好的例子。另一个很好的例子是 NTP 守护进程，它负责同步计算机时钟，有时甚至分发时间给其他计算机。

了解如何创建守护进程将使您能够创建服务器软件；例如，Web 服务器、聊天服务器等。

在本教程中，我们将创建一个简单的守护进程来演示一些重要的概念。

## 准备工作

你只需要本章节*技术要求*部分列出的组件。

## 操作方法

在本教程中，我们将编写一个在我们的系统中后台运行的小型守护进程。守护进程唯一的“工作”是将当前日期和时间写入文件。这证明了守护进程是活着的。让我们开始吧：

1.  与我们以前的示例相比，守护进程的代码相当长。因此，代码已分成几个步骤。这里还有一些我们还没有涉及的新东西。将代码写入一个文件并将其保存为`my-daemon.c`。请记住，所有步骤中的所有代码都放入这个文件中。我们将从我们需要的所有`include`文件、我们需要的变量和我们的`fork()`开始，就像我们以前看到的那样。这个`fork()`将是两个中的第一个：

```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
int main(void)
{
   pid_t pid;
   FILE *fp;
   time_t now; /* for the current time */
   const char pidfile[] = "/var/run/my-daemon.pid";
   const char daemonfile[] = 
      "/tmp/my-daemon-is-alive.txt";
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
```

1.  现在我们已经 forked，我们希望父进程退出。一旦父进程退出，我们将处于子进程中。在子进程中，我们将使用`setsid()`创建一个新的会话。创建一个新的会话将释放进程的控制终端：

```
   else if ( (pid != 0) )
   {
      exit(0);
   }
   /* the parent process has exited, so this is the
    * child. create a new session to lose the 
    * controlling terminal */
   setsid();
```

1.  现在，我们想再次`fork()`。这第二次 fork 将创建一个新的进程，就像以前一样，但由于它是一个已经存在的会话中的新进程，它不会成为会话领导者，从而阻止它获取一个新的控制终端。新的子进程被称为孙子。再一次，我们退出父进程（子进程）。然而，在退出子进程之前，我们将孙子的 PID 写入**PID 文件**。这个 PID 文件用于跟踪守护进程：

```
   /* fork again, creating a grandchild, 
    * the actual daemon */
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
   /* the child process which will exit */
   else if ( pid > 0 )
   {
      /* open pid-file for writing and error 
       * check it */
      if ( (fp = fopen(pidfile, "w")) == NULL )
      {
         perror("Can't open file for writing");
         return 1;
      }
      /* write pid to file */
      fprintf(fp, "%d\n", pid); 
      fclose(fp); /* close the file pointer */
      exit(0);
   }
```

1.  现在，将默认模式（*umask*）设置为守护进程的合理值。我们还必须将当前工作目录更改为`/`，以便守护进程不会阻止文件系统卸载或目录被删除。然后，我们必须打开守护进程文件，这是我们将写入消息的地方。消息将包含当前日期和时间，并告诉我们一切是否正常。通常，这将是一个日志文件：

```
   umask(022); /* set the umask to something ok */
   chdir("/"); /* change working directory to / */
   /* open the "daemonfile" for writing */
   if ( (fp = fopen(daemonfile, "w")) == NULL )
   {
      perror("Can't open daemonfile");
      return 1;
   }
```

1.  由于守护进程只会在后台运行，我们不需要 stdin、stdout 和 stderr，所以让我们将它们全部关闭。但是，将它们关闭是不安全的。如果代码中的某些部分稍后打开文件描述符，它将获得文件描述符 0，通常是 stdin。文件描述符是按顺序分配的。如果没有打开的文件描述符，第一次调用`open()`将获得描述符`0`；第二次调用将获得描述符`1`。另一个问题可能是，某些部分可能尝试写入 stdout，但 stdout 已经不存在，这会导致程序崩溃。因此，我们必须重新打开它们全部，但是重新打开到`/dev/null`（黑洞）：

```
   /* from here, we don't need stdin, stdout or, 
    * stderr anymore, so let's close them all, 
    * then re-open them to /dev/null */
   close(STDIN_FILENO);
   close(STDOUT_FILENO);
   close(STDERR_FILENO);
   open("/dev/null", O_RDONLY); /* 0 = stdin */
   open("/dev/null", O_WRONLY); /* 1 = stdout */
   open("/dev/null", O_RDWR); /* 2 = stderr */
```

1.  最后，我们可以开始守护进程的工作。这只是一个`for`循环，向守护进程文件写入一条消息，说明守护进程仍然存活。请注意，我们必须在每次`fprintf()`后使用`fflush()`刷新文件指针。通常，在 Linux 中，事情是*行缓冲*的，这意味着在写入之前只缓冲一行。但由于这是一个文件而不是 stdout，它实际上是完全缓冲的，这意味着它会缓冲所有数据，直到缓冲区满或文件流关闭。如果没有`fflush()`，我们在填满缓冲区之前将看不到文件中的任何文本。通过在每次`fprintf()`后使用`fflush()`，我们可以在文件中实时看到文本：

```
   /* here we start the daemons "work" */
   for (;;)
   {
      /* get the current time and write it to the
         "daemonfile" that we opened above */
      time(&now);
      fprintf(fp, "Daemon alive at %s", 
         ctime(&now));
      fflush(fp); /* flush the stream */
      sleep(30);
   }
   return 0;
}
```

1.  现在，是时候编译整个守护进程了：

```
$> make my-daemon
gcc -Wall -Wextra -pedantic -std=c99    my-daemon.c  
-o my-daemon
```

1.  现在，我们可以启动守护进程。由于我们将 PID 文件写入`/var/run`，我们需要以 root 身份执行守护进程。我们不会从守护进程中得到任何输出；它将悄悄地与终端分离：

```
$> sudo ./my-daemon
```

1.  现在守护进程正在运行，让我们检查已写入`/var/run/my-daemon.pid`的 PID 号码：

```
$> cat /var/run/my-daemon.pid 
5508
```

1.  让我们使用`ps`和`pstree`来调查守护进程。如果一切都按照预期进行，它的父进程应该是`systemd`，并且它应该在自己的会话中（SID 应该与进程 ID 相同）：

```
$> ps jp 5508
PPID PID PGID SID TTY TPGID STAT UID TIME COMMAND
1   5508 5508 5508?   -1    Ss    0  0:00 ./my-daemon
$> pstree -A -p -s 5508
systemd(1)---my-daemon(5508)
```

1.  让我们还看看`/tmp/my-daemon-is-alive.txt`文件。这个文件应该包含一些指定日期和时间的行，相隔 30 秒：

```
$> cat /tmp/my-daemon-is-alive.txt 
Daemon alive at Sun Nov 22 23:25:45 2020
Daemon alive at Sun Nov 22 23:26:15 2020
Daemon alive at Sun Nov 22 23:26:45 2020
Daemon alive at Sun Nov 22 23:27:15 2020
Daemon alive at Sun Nov 22 23:27:45 2020
Daemon alive at Sun Nov 22 23:28:15 2020
Daemon alive at Sun Nov 22 23:28:45 2020
```

1.  最后，让我们杀死守护进程，以防止它继续写入文件：

```
$> sudo kill 5508
```

## 工作原理…

我们刚刚编写的守护进程是一个基本的传统守护进程，但它演示了我们需要充分理解的所有概念。其中一个新的重要概念是如何使用`setsid()`启动一个新会话。如果我们不创建一个新会话，守护进程仍将是用户登录会话的一部分，并在用户注销时终止。但由于我们为守护进程创建了一个新会话，并且它被`systemd`继承，它现在独立存在，不受启动它的用户和进程的影响。

第二次分叉的原因是，会话领导者——也就是我们在`setsid()`调用后的第一个子进程——如果打开终端设备，可以获取一个新的控制终端。当我们进行第二次分叉时，新的子进程只是第一个子进程创建的会话的成员，而不是领导者，因此它不再能获取**控制终端**。避免控制终端的原因是，如果该终端退出，守护进程也会退出。在创建守护进程时进行两次分叉通常被称为**双重分叉**技术。

需要以 root 身份启动守护进程的原因是它需要写入`/var/run/`。如果我们改变目录，或者完全跳过它，守护进程将作为普通用户正常运行。然而，大多数守护进程确实以 root 身份运行。然而，也有一些以普通用户身份运行的守护进程；例如，处理与用户相关的事务的守护进程，比如`tmux`（一个终端复用器）。

我们还将工作目录更改为`/`。这样守护进程就不会锁定目录。顶级根目录不会被删除或卸载，这使其成为守护进程的安全工作目录。

## 还有更多...

我们在这里编写的是传统的 Linux/Unix 守护进程。这些类型的守护进程今天仍在使用，例如，用于像这样的小型和快速守护进程。然而，自从`systemd`出现以来，我们不再需要像刚才那样“使守护进程成为守护进程”。例如，建议保留 stdout 和 stderr 打开，并将所有日志消息发送到那里。然后这些消息将显示在*journal*中。我们将在*第七章**，使用 systemd 处理您的守护进程*中更深入地介绍 systemd 和 journal。

我们在这里编写的守护进程类型在 systemd 语言中被称为*forking*，我们以后会更多地了解它。

就像`system()`在执行新程序时为我们简化了事情一样，还有一个名为`daemon()`的函数可以为我们创建守护进程。这个函数将为我们做所有繁重的工作，比如分叉、关闭和重新打开文件描述符、更改工作目录等。然而，请注意，这个函数不使用我们在本篇中用于守护进程的双重分叉技术。这一事实在`man 3 daemon`手册页的 BUGS 部分中明确说明。

# 实现信号处理程序

在上一篇中，我们编写了一个简单但功能齐全的守护进程。然而，它也存在一些问题；例如，当守护进程被终止时，PID 文件没有被删除。同样，当守护进程被终止时，打开的文件流（`/tmp/my-daemon-is-alive.txt`）也没有被关闭。一个合适的守护进程在退出时应该进行清理。

为了能够在退出时进行清理，我们需要实现一个信号处理程序。然后信号处理程序应该在守护进程终止之前处理所有的清理工作。在本章中，我们已经看到了信号处理程序的例子，所以这个概念并不新鲜。

然而，并不只有守护进程使用信号处理程序。这是一种常见的控制进程的方式，特别是那些没有控制终端的进程。

## 准备工作

在阅读本篇之前，您应该先阅读上一篇，以便了解守护进程的功能。除此之外，您还需要本章*技术要求*部分列出的程序。

## 操作方法

在本篇中，我们将为上一篇中编写的守护进程添加信号处理程序。由于代码会有点长，我将其分成几个步骤。不过，请记住，所有的代码都在同一个文件中。让我们开始吧：

1.  将以下代码写入文件并命名为`my-daemon-v2.c`。我们将从`#include`文件和变量开始，就像之前一样。但是请注意，这一次我们已经将一些变量移到了全局空间。我们这样做是为了让信号处理程序可以访问它们。没有办法向信号处理程序传递额外的参数，所以这是访问它们的最佳方式。在这里，我们还必须为`sigaction()`定义`_POSIX_C_SOURCE`。我们还必须在这里创建我们的信号处理程序的原型，称为`sigHandler()`。另外，请注意新的`sigaction`结构：

```
#include <sys/types.h>
#include <sys/stat.h>
#include <time.h>
#include <fcntl.h>
#include <signal.h>
void sigHandler(int sig);
/* moved these variables to the global scope
   since they need to be access/deleted/closed
   from the signal handler */
FILE *fp;
const char pidfile[] = "/var/run/my-daemon.pid";
int main(void)
{
   pid_t pid;
   time_t now; /* for the current time */
   struct sigaction action; /* for sigaction */
   const char daemonfile[] = 
      "/tmp/my-daemon-is-alive.txt";
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
   else if ( (pid != 0) )
   {
      exit(0);
   }
```

1.  就像之前一样，我们必须在第一次分叉后创建一个新会话。之后，我们必须进行第二次分叉，以确保它不再是一个会话领导者：

```
   /* the parent process has exited, which makes 
    * the rest of the code the child process */
   setsid(); /* create a new session to lose the 
                controlling terminal */

   /* fork again, creating a grandchild, the 
    * actual daemon */
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
   /* the child process which will exit */
   else if ( pid > 0 )
   {
      /* open pid-file for writing and check it */
      if ( (fp = fopen(pidfile, "w")) == NULL )
      {
         perror("Can't open file for writing");
         return 1;
      }
      /* write pid to file */
      fprintf(fp, "%d\n", pid); 
      fclose(fp); /* close the file pointer */
      exit(0);
   }
```

1.  与之前一样，我们必须更改 umask、当前工作目录，并使用`fopen()`打开守护进程文件。接下来，我们必须关闭并重新打开 stdin、stdout 和 stderr：

```
   umask(022); /* set the umask to something ok */
   chdir("/"); /* change working directory to / */
   /* open the "daemonfile" for writing */
   if ( (fp = fopen(daemonfile, "w")) == NULL )
   {
      perror("Can't open daemonfile");
      return 1;
   }
   /* from here, we don't need stdin, stdout or, 
    * stderr anymore, so let's close them all, 
    * then re-open them to /dev/null */
   close(STDIN_FILENO);
   close(STDOUT_FILENO);
   close(STDERR_FILENO);
   open("/dev/null", O_RDONLY); /* 0 = stdin */
   open("/dev/null", O_WRONLY); /* 1 = stdout */
   open("/dev/null", O_RDWR); /* 2 = stderr */
```

1.  现在，终于是时候准备并注册信号处理程序了。这正是我们在本章前面讨论过的内容，只是在这里，我们为所有常见的退出信号注册处理程序，比如终止、中断、退出和中止。一旦我们处理了信号处理程序，我们将开始守护进程的工作；也就是，将消息写入守护进程文件的`for`循环：

```
/* prepare for sigaction */
   action.sa_handler = sigHandler;
   sigfillset(&action.sa_mask);
   action.sa_flags = SA_RESTART;
   /* register the signals we want to handle */
   sigaction(SIGTERM, &action, NULL);
   sigaction(SIGINT, &action, NULL);
   sigaction(SIGQUIT, &action, NULL);
   sigaction(SIGABRT, &action, NULL);
   /* here we start the daemons "work" */
   for (;;)
   {
      /* get the current time and write it to the
         "daemonfile" that we opened above */
      time(&now);
      fprintf(fp, "Daemon alive at %s", 
         ctime(&now));
      fflush(fp); /* flush the stream */
      sleep(30);
   }
   return 0;
}
```

1.  最后，我们必须实现信号处理程序的函数。在这里，我们通过在退出之前删除 PID 文件来清理守护进程。我们还关闭了打开的文件流到守护进程文件：

```
void sigHandler(int sig)
{
    int status = 0;
    if ( sig == SIGTERM || sig == SIGINT 
        || sig == SIGQUIT 
        || sig == SIGABRT )
    {
        /* remove the pid-file */
        if ( (unlink(pidfile)) == -1 )
            status = 1;
        if ( (fclose(fp)) == EOF )
            status = 1;
        exit(status); /* exit with the status set*/
    }
    else /* some other signal */
    {
        exit(1);
    }
}
```

1.  编译守护进程的新版本：

```
$> make my-daemon-v2
gcc -Wall -Wextra -pedantic -std=c99    my-daemon-v2.c
-o my-daemon-v2
```

1.  以 root 身份启动守护进程，就像我们之前做的那样：

```
$> sudo ./my-daemon-v2 
```

1.  查看 PID 文件中的 PID 并做好记录：

```
$> cat /var/run/my-daemon.pid 
22845
```

1.  使用`ps`命令查看它是否按预期运行：

```
$> ps jp 22845
  PPID   PID  PGID   SID TTY TPGID STAT UID TIME
COMMAND
    1 22845 22845 22845 ?      -1 Ss     0 0:00 ./my
daemon-v2
```

1.  用默认信号`TERM`杀死守护进程：

```
$> sudo kill 22845
```

1.  如果一切按计划进行，PID 文件将被删除。尝试使用`cat`命令访问 PID 文件：

```
$> cat /var/run/my-daemon.pid 
cat: /var/run/my-daemon.pid: No such file or directory
```

## 工作原理…

在这个示例中，我们实现了一个信号处理程序，负责所有清理工作。它会删除 PID 文件并关闭打开的文件流。为了处理最常见的“退出”信号，我们使用四个不同的信号注册了处理程序：*终止*、*中断*、*退出*和*中止*。当守护进程接收到其中一个信号时，它会触发`sigHandler()`函数。该函数然后会删除 PID 文件并关闭文件流。最后，该函数通过调用`exit()`退出整个守护进程。

然而，由于我们无法将文件名或文件流作为参数传递给信号处理程序，我们将这些变量放在全局范围内。这样一来，`main()`和`sigHandler()`都可以访问它们。

## 更多内容…

记得我们之前必须刷新流才能在`/tmp/my-daemon-is-alive.txt`中显示时间和日期吗？由于现在守护进程退出时关闭文件流，我们不再需要`fflush()`。数据在关闭时被写入文件。然而，这样一来，我们就无法在守护进程运行时“实时”看到时间和日期。这就是为什么我们在代码中仍然保留了`fflush()`。
