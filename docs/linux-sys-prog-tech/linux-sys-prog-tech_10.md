# 第十章：使用不同类型的 IPC

在本章中，我们将学习通过所谓的**进程间通信**（**IPC**）的各种方式。我们将编写使用不同类型的 IPC 的各种程序，从信号和管道到 FIFO、消息队列、共享内存和套接字。

进程有时需要交换信息-例如，在同一台计算机上运行的客户端和服务器程序的情况下。也可能是一个分叉成两个进程的进程，它们需要以某种方式进行通信。

这种 IPC 可以以多种方式进行。在本章中，我们将学习一些最常见的方式。

如果您想编写不仅仅是最基本程序的程序，了解 IPC 是必不可少的。迟早，您将拥有由多个部分或多个程序组成的程序，需要共享信息。

在本章中，我们将介绍以下配方：

+   使用信号进行 IPC-为守护程序构建客户端

+   使用管道进行通信

+   FIFO-在 shell 中使用它

+   FIFO-构建发送方

+   FIFO-构建接收方

+   消息队列-创建发送方

+   消息队列-创建接收方

+   使用共享内存在子进程和父进程之间进行通信

+   在不相关的进程之间使用共享内存

+   Unix 套接字-创建服务器

+   Unix 套接字-创建客户端

让我们开始吧！

# 技术要求

对于本章，您将需要*第三章*中的 GCC 编译器，Make 工具和通用的 Makefile，*深入 Linux 中的 C 语言*。如果您尚未安装这些工具，请参阅*第一章*，*获取必要的工具并编写我们的第一个 Linux 程序*，以获取安装说明。

本章的所有代码示例和通用的 Makefile 都可以从 GitHub 上下载，网址为[`github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch10`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch10)。

查看以下链接以查看代码演示视频：[`bit.ly/3u3y1C0`](https://bit.ly/3u3y1C0)

# 使用信号进行 IPC-为守护程序构建客户端

在本书中，我们已经多次使用了信号。但是，当我们这样做时，我们总是使用`kill`命令来发送`my-daemon-v2`，来自*第六章*，*生成进程和使用作业控制*。

这是使用信号进行**IPC**的典型示例。守护程序有一个小的“客户端程序”来控制它，以便可以停止它，重新启动它，重新加载其配置文件等。

知道如何使用信号进行 IPC 是编写可以相互通信的程序的坚实起点。

## 准备工作

对于这个配方，你需要 GCC 编译器，Make 工具和通用的 Makefile。您还需要*第六章*中的`my-daemon-v2.c`文件，*生成进程和使用作业控制*。在本章的 GitHub 目录中有该文件的副本，网址为[`github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch10`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch10)。

## 如何做…

在这个配方中，我们将向*第六章*中的守护程序添加一个小的客户端程序，*生成进程和使用作业控制*。这个程序将向守护程序发送信号，就像`kill`命令一样。但是，这个程序只会向守护程序发送信号，不会发送给其他进程：

1.  在文件中编写以下代码并将其保存为`my-daemon-ctl.c`。这个程序有点长，所以它分成了几个步骤。不过所有的代码都放在同一个文件中。我们将从包含行、使用函数的原型和我们需要的所有变量开始：

```
#define _XOPEN_SOURCE 500
#include <stdio.h>
#include <sys/types.h>
#include <signal.h>
#include <getopt.h>
#include <string.h>
#include <linux/limits.h>
void printUsage(char progname[], FILE *fp);
int main(int argc, char *argv[])
{
   FILE *fp;
   FILE *procfp;
   int pid, opt;
   int killit = 0;
   char procpath[PATH_MAX] = { 0 };
   char cmdline[PATH_MAX] = { 0 };
   const char pidfile[] = "/var/run/my-daemon.pid";
   const char daemonPath[] = 
      "/usr/local/sbin/my-daemon-v2";
```

1.  然后，我们希望能够解析命令行选项。我们只需要两个选项；即，`-h`用于帮助，`-k`用于杀死守护进程。默认情况下是显示守护进程的状态：

```
   /* Parse command-line options */
   while ((opt = getopt(argc, argv, "kh")) != -1)
   {
      switch (opt)
      {
         case 'k': /* kill the daemon */
            killit = 1;
            break;
         case 'h': /* help */
            printUsage(argv[0], stdout);
            return 0;
         default: /* in case of invalid options */
            printUsage(argv[0], stderr);
            return 1;
      }
   }
```

1.  现在，让我们打开`/proc`中的`cmdline`文件。然后，我们必须打开该文件并从中读取完整的命令行路径：

```
   if ( (fp = fopen(pidfile, "r")) == NULL )
   {
      perror("Can't open PID-file (daemon isn't "
         "running?)");
   }
   /* read the pid (and check if we could read an 
    * integer) */
   if ( (fscanf(fp, "%d", &pid)) != 1 )
   {
      fprintf(stderr, "Can't read PID from %s\n", 
         pidfile);
      return 1;
   }
   /* build the /proc path */
   sprintf(procpath, "/proc/%d/cmdline", pid);
   /* open the /proc path */
   if ( (procfp = fopen(procpath, "r")) == NULL )
   {
      perror("Can't open /proc path"
         " (no /proc or wrong PID?)");
      return 1;
   }
   /* read the cmd line path from proc */
   fscanf(procfp, "%s", cmdline); 
```

1.  既然我们既有 PID 又有完整的命令行，我们可以再次检查 PID 是否属于`/usr/local/sbin/my-daemon-v2`而不是其他进程：

```
   /* check that the PID matches the cmdline */
   if ( (strncmp(cmdline, daemonPath, PATH_MAX)) 
      != 0 )
   {
      fprintf(stderr, "PID %d doesn't belong "
         "to %s\n", pid, daemonPath);
      return 1;
   }
```

1.  如果我们给程序加上`-k`选项，我们必须将`killit`变量设置为 1。因此，在这一点上，我们必须杀死进程。否则，我们只是打印一条消息，说明守护进程正在运行：

```
   if ( killit == 1 )
   {
      if ( (kill(pid, SIGTERM)) == 0 )
      {
         printf("Successfully terminated " 
            "my-daemon-v2\n");
      }
      else
      {
         perror("Couldn't terminate my-daemon-v2");
         return 1;
      }        
   }
   else
   {
      printf("The daemon is running with PID %d\n", 
         pid);
   }
   return 0;
}
```

1.  最后，我们为`printUsage()`函数创建函数：

```
void printUsage(char progname[], FILE *fp)
{
   fprintf(fp, "Usage: %s [-k] [-h]\n", progname);
   fprintf(fp, "If no options are given, a status "
      "message is displayed.\n"
      "-k will terminate the daemon.\n"
      "-h will display this usage help.\n");       
}
```

1.  现在，我们可以编译程序了：

```
$> make my-daemon-ctl
gcc -Wall -Wextra -pedantic -std=c99    my-daemon ctl.c   -o my-daemon-ctl
```

1.  在继续之前，请确保你已经禁用并停止了*第七章**，使用 systemd 管理守护进程*中的`systemd`服务：

```
$> sudo systemctl disable my-daemon
$> sudo systemctl stop my-daemon
```

1.  现在编译守护进程（`my-daemon-v2.c`），如果你还没有这样做的话：

```
$> make my-daemon-v2
gcc -Wall -Wextra -pedantic -std=c99    my-daemon-v2.c   -o my-daemon-v2
```

1.  然后，手动启动守护进程（这次没有`systemd`服务）：

```
$> sudo ./my-daemon-v2
```

1.  现在，我们可以尝试使用我们的新程序来控制守护进程。请注意，我们不能像普通用户一样杀死守护进程：

```
$> ./my-daemon-ctl 
The daemon is running with PID 17802 and cmdline ./my-daemon-v2
$> ./my-daemon-ctl -k
Couldn't terminate daemon: Operation not permitted
$> sudo ./my-daemon-ctl -k
Successfully terminated daemon
```

1.  如果守护进程被杀死后我们重新运行程序，它会告诉我们没有 PID 文件，因此守护进程没有运行：

```
$> ./my-daemon-ctl 
Can't open PID-file (daemon isn't running?): No such file or directory
```

## 工作原理…

由于守护进程创建了 PID 文件，我们可以使用该文件获取正在运行的守护进程的 PID。当守护进程终止时，它会删除 PID 文件，因此如果没有 PID 文件，我们可以假设守护进程没有运行。

如果 PID 文件存在，首先我们从文件中读取 PID。然后，我们使用 PID 来组装该 PID 的`/proc`文件系统中的`cmdline`文件的路径。Linux 系统上的每个进程都在`/proc`文件系统中有一个目录。在每个进程的目录中，有一个名为`cmdline`的文件。该文件包含进程的完整命令行。例如，如果守护进程是从当前目录启动的，它包含`./my-daemon-v2`，而如果它是从`/usr/local/sbin/my-daemon-v2`启动的，它包含完整路径。

例如，如果守护进程的 PID 是`12345`，那么`cmdline`的完整路径是`/proc/12345/cmdline`。这就是我们用`sprintf()`组装的内容。

然后，我们读取`cmdline`的内容。稍后，我们使用该文件的内容来验证 PID 是否与名称为`my-daemon-v2`的进程匹配。这是一项安全措施，以免误杀错误的进程。如果使用`KILL`信号杀死守护进程，它就没有机会删除 PID 文件。如果将来另一个进程获得相同的 PID，我们就有可能误杀该进程。PID 号最终会被重用。

当我们有了守护进程的 PID 并验证它确实属于正确的进程时，我们将根据`-k`选项指定的内容获取其状态或将其杀死。

这就是许多用于控制复杂守护进程的控制程序的工作方式。

## 另请参阅

有关`kill()`系统调用的更多信息，请参阅`man 2 kill`手册页。

# 使用管道进行通信

在这个示例中，我们将创建一个程序，进行分叉，然后使用**管道**在两个进程之间进行通信。有时，当我们**分叉**一个进程时，**父进程**和**子进程**需要一种通信方式。管道通常是实现这一目的的简单方法。

当你编写更复杂的程序时，了解如何在父进程和子进程之间进行通信和交换数据是很重要的。

## 准备工作

对于这个示例，我们只需要 GCC 编译器、Make 工具和通用 Makefile。

## 如何做…

让我们编写一个简单的分叉程序：

1.  将以下代码写入一个文件中，并将其命名为`pipe-example.c`。我们将逐步介绍代码。请记住，所有代码都在同一个文件中。

我们将从包含行和`main()`函数开始。然后，我们将创建一个大小为 2 的整数数组。管道将在以后使用该数组。数组中的第一个整数（0）是管道读端的文件描述符。第二个整数（1）是管道的写端：

```
#define _POSIX_C_SOURCE  200809L
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#define MAX 128
int main(void)
{
   int pipefd[2] = { 0 };
   pid_t pid;
   char line[MAX];
```

1.  现在，我们将使用`pipe()`系统调用创建管道。我们将把整数数组作为参数传递给它。之后，我们将使用`fork()`系统调用进行分叉：

```
   if ( (pipe(pipefd)) == -1 )
   {
      perror("Can't create pipe");
      return 1;
   }   
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
```

1.  如果我们在父进程中，我们关闭读端（因为我们只想从父进程中写入）。然后，我们使用`dprintf()`向管道的文件描述符（写端）写入消息：

```
   if (pid > 0)
   {
      /* inside the parent */
      close(pipefd[0]); /* close the read end */
      dprintf(pipefd[1], "Hello from parent");
   }
```

1.  在子进程中，我们做相反的操作；也就是说，我们关闭管道的写端。然后，我们使用`read()`系统调用从管道中读取数据。最后，我们使用`printf()`打印消息：

```
   else
   {
      /* inside the child */
      close(pipefd[1]); /* close the write end */
      read(pipefd[0], line, MAX-1);
      printf("%s\n", line); /* print message from
                             * the parent */
   }
   return 0;
}
```

1.  现在编译程序，以便我们可以运行它：

```
$> make pipe-example
gcc -Wall -Wextra -pedantic -std=c99    pipe-example.c   -o pipe-example
```

1.  让我们运行程序。父进程使用管道向子进程发送消息`Hello from parent`。然后，子进程在屏幕上打印该消息：

```
$> ./pipe-example 
Hello from parent
```

## 工作原理…

`pipe()`系统调用将两个文件描述符返回给整数数组。第一个，`pipefd[0]`，是管道的读端，而另一个，`pipefd[1]`，是管道的写端。在父进程中，我们向管道的*写端*写入消息。然后，在子进程中，我们从管道的*读端*读取数据。但在进行任何读写操作之前，我们关闭在各自进程中没有使用的管道端。

管道是一种比较常见的 IPC 技术。但是它们有一个缺点，即它们只能在相关进程之间使用；也就是说，具有共同父进程（或父进程和子进程）的进程。

还有另一种形式的管道可以克服这个限制：所谓的*命名管道*。命名管道的另一个名称是 FIFO。这是我们将在下一个示例中介绍的内容。

## 另请参阅

有关`pipe()`系统调用的更多信息可以在`man 2 pipe`手册页中找到。

# FIFO - 在 shell 中使用它

在上一个示例中，我提到`pipe()`系统调用有一个缺点——它只能在相关进程之间使用。但是，我们可以使用另一种类型的管道，称为**命名管道**。另一个名称是**先进先出**（**FIFO**）。命名管道可以在任何进程之间使用，无论是否相关。

命名管道，或者 FIFO，实际上是一种特殊类型的文件。`mkfifo()`函数在文件系统上创建该文件，就像创建任何其他文件一样。然后，我们可以使用该文件在进程之间读取和写入数据。

还有一个名为`mkfifo`的命令，我们可以直接从 shell 中使用它来创建命名管道。我们可以使用它在不相关的命令之间传输数据。

在这个命名管道的介绍中，我们将介绍`mkfifo`命令。在接下来的两个示例中，我们将编写一个使用`mkfifo()`函数的 C 程序，然后再编写另一个程序来读取管道的数据。

了解如何使用命名管道将为您作为用户、系统管理员和开发人员提供更多的灵活性。您不再只能在相关进程之间使用管道。您可以自由地在系统上的任何进程或命令之间传输数据，甚至可以在不同的用户之间传输数据。

## 准备工作

在这个示例中，我们不会编写任何程序，因此没有特殊要求。

## 操作步骤…

在这个示例中，我们将探讨`mkfifo`命令，并学习如何使用它在不相关的进程之间传输数据：

1.  我们将首先创建一个命名管道——一个 FIFO 文件。我们将在`/tmp`目录中创建它，这是临时文件的常见位置。但是，您可以在任何您喜欢的地方创建它：

```
$> mkfifo /tmp/my-fifo
```

1.  让我们通过使用`file`和`ls`命令来确认这确实是一个 FIFO。请注意我的 FIFO 的当前权限模式。它可以被所有人读取。但是在您的`umask`取决于您的系统，这可能会有所不同。但是，如果我们要传输敏感数据，我们应该对此保持警惕。在这种情况下，我们可以使用`chmod`命令进行更改：

```
$> file /tmp/my-fifo 
/tmp/my-fifo: fifo (named pipe)
$> ls -l /tmp/my-fifo 
prw-r--r-- 1 jake jake 0 jan 10 20:03 /tmp/my-fifo
```

1.  现在，我们可以尝试向管道发送数据。由于管道是一个文件，我们将在这里使用重定向而不是管道符号。换句话说，我们将数据重定向到管道。在这里，我们将`uptime`命令的输出重定向到管道。一旦我们将数据重定向到管道，进程将挂起，这是正常的，因为没有人在另一端接收数据。它实际上并不挂起；它*阻塞*：

```
$> uptime -p > /tmp/my-fifo
```

1.  打开一个新的终端并输入以下命令以从管道接收数据。请注意，第一个终端中的进程现在将结束：

```
$> cat < /tmp/my-fifo 
up 5 weeks, 6 days, 2 hours, 11 minutes
```

1.  我们也可以做相反的事情；也就是说，我们可以首先打开接收端，然后向管道发送数据。这将**阻塞**接收进程，直到获得一些数据。运行以下命令设置接收端，并让其运行：

```
$> cat < /tmp/my-fifo
```

1.  现在，我们使用相同的`uptime`命令向管道发送数据。请注意，一旦数据被接收，第一个进程将结束：

```
$> uptime -p > /tmp/my-fifo
```

1.  还可以从多个进程向 FIFO 发送数据。打开三个新的终端。在每个终端中，输入以下命令，但将第二个终端替换为 2，第三个终端替换为 3：

```
$> echo "Hello from terminal 1" > /tmp/my-fifo
```

1.  现在，打开另一个终端并输入以下命令。这将接收所有消息：

```
$> cat < /tmp/my-fifo
Hello from terminal 3
Hello from terminal 1
Hello from terminal 2
```

## 它是如何工作的…

FIFO 只是文件系统上的一个文件，尽管是一个特殊的文件。一旦我们将数据重定向到 FIFO，该进程将**阻塞**（或“挂起”），直到另一端接收到数据。

同样，如果我们首先启动接收进程，该进程将阻塞，直到获取管道的数据。这种行为的原因是 FIFO 不是我们可以保存数据的常规文件。我们只能用它重定向数据；也就是说，它只是一个*管道*。因此，如果我们向其发送数据，但另一端没有任何东西，进程将在那里等待，直到有人在另一端接收它。数据在管道中无处可去，直到有人连接到接收端。

还有更多...

如果系统上有多个用户，您可以尝试使用 FIFO 向它们发送消息。这样做为我们提供了一种在用户之间复制和粘贴数据的简单方法。请注意，FIFO 的权限模式必须允许其他用户读取它（如果需要，还可以写入它）。可以在创建 FIFO 时直接设置所需的权限模式，使用`-m`选项。例如，`mkfifo /tmp/shared-fifo -m 666`将允许任何用户读取和写入 FIFO。

## 另请参阅

在`man 1 mkfifo`手册页中有关于`mkfifo`命令的更多信息。有关 FIFO 的更深入解释，请参阅`man 7 fifo`手册页。

# FIFO - 构建发送方

现在我们知道了 FIFO 是什么，我们将继续编写一个可以创建和使用 FIFO 的程序。在这个示例中，我们将编写一个创建 FIFO 然后向其发送消息的程序。在下一个示例中，我们将编写一个接收该消息的程序。

了解如何在程序中使用 FIFO 将使您能够编写可以直接使用 FIFO 进行通信的程序，而无需通过 shell 重定向数据。

## 准备工作

我们需要常规工具；即 GCC 编译器、Make 工具和通用 Makefile。

## 如何做…

在这个示例中，我们将编写一个创建 FIFO 并向其发送消息的程序：

1.  在文件中写入以下代码并将其保存为`fifo-sender.c`。这段代码有点长，所以我们将在这里逐步介绍它。请记住，所有代码都放在同一个文件中。让我们从`#include`行、信号处理程序的原型和一些全局变量开始：

```
#define _XOPEN_SOURCE 700
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
#include <signal.h>
#include <stdlib.h>
#include <errno.h>
void cleanUp(int signum);
int fd; /* the FIFO file descriptor */
const char fifoname[] = "/tmp/my-2nd-fifo";
```

1.  现在，我们可以开始编写`main()`函数。首先，我们将为`sigaction()`函数创建结构体。然后，我们将检查用户是否提供了消息作为参数：

```
int main(int argc, char *argv[])
{
   struct sigaction action; /* for sigaction */
   if ( argc != 2 )
   {
      fprintf(stderr, "Usage: %s 'the message'\n",
         argv[0]);
      return 1;
   }
```

1.  现在，我们必须为我们想要捕获的所有信号注册信号处理程序。我们这样做是为了在程序退出时删除 FIFO。请注意，我们还注册了`SIGPIPE`信号——关于这一点，我们将在*它是如何工作的…*部分详细说明：

```
   /* prepare for sigaction and register signals
    * (for cleanup when we exit) */
   action.sa_handler = cleanUp;
   sigfillset(&action.sa_mask);
   action.sa_flags = SA_RESTART;
   sigaction(SIGTERM, &action, NULL);
   sigaction(SIGINT, &action, NULL);
   sigaction(SIGQUIT, &action, NULL);
   sigaction(SIGABRT, &action, NULL);
   sigaction(SIGPIPE, &action, NULL);
```

1.  现在，让我们使用模式`644`创建 FIFO。由于模式`644`是八进制的，我们需要在 C 代码中写为`0644`；否则，它将被解释为 644 十进制（在 C 中以 0 开头的任何数字都是八进制数）。之后，我们必须使用`open()`系统调用打开 FIFO——与我们用于打开常规文件的系统调用相同：

```
   if ( (mkfifo(fifoname, 0644)) != 0 )
   {
      perror("Can't create FIFO");
      return 1;
   }
   if ( (fd = open(fifoname, O_WRONLY)) == -1)
   {
      perror("Can't open FIFO");
      return 1;
   }
```

1.  现在，我们必须创建一个无限循环。在这个循环内，我们将每秒打印一次用户提供的消息。循环结束后，我们将关闭文件描述符并删除 FIFO 文件。不过在正常情况下，我们不应该达到这一步：

```
   while(1)
   {
      dprintf(fd, "%s\n", argv[1]);
      sleep(1);
   }
   /* just in case, but we shouldn't reach this */
   close(fd);
   unlink(fifoname);
   return 0;
}
```

1.  最后，我们必须创建`cleanUp()`函数，这是我们注册为信号处理程序的函数。我们使用这个函数在程序退出之前进行清理。然后，我们必须关闭文件描述符并删除 FIFO 文件：

```
void cleanUp(int signum)
{
   if (signum == SIGPIPE)
      printf("The receiver stopped receiving\n");
   else
      printf("Aborting...\n");
   if ( (close(fd)) == -1 )
      perror("Can't close file descriptor");
   if ( (unlink(fifoname)) == -1)
   {
      perror("Can't remove FIFO");
      exit(1);
   }
   exit(0);
}
```

1.  让我们编译程序：

```
$> make fifo-sender
gcc -Wall -Wextra -pedantic -std=c99    fifo-sender.c   -o fifo-sender
```

1.  让我们运行程序：

```
$> ./fifo-sender 'Hello everyone, how are you?'
```

1.  现在，启动另一个终端，以便使用`cat`接收消息。我们在程序中使用的文件名是`/tmp/my-2nd-fifo`。消息将每秒重复一次。几秒钟后，按下*Ctrl* + *C*退出`cat`：

```
$> cat < /tmp/my-2nd-fifo 
Hello everyone, how are you?
Hello everyone, how are you?
Hello everyone, how are you?
*Ctrl**+**P* 
```

1.  现在，返回到第一个终端。您会注意到它显示*接收器停止接收*。

1.  在第一个终端中再次启动`fifo-sender`程序。

1.  再次转到第二个终端，并重新启动`cat`程序以接收消息。让`cat`程序继续运行：

```
$> cat < /tmp/my-2nd-fifo
```

1.  当第二个终端上的 cat 程序正在运行时，返回到第一个终端，并通过按下*Ctrl* + *C*中止`fifo-sender`程序。请注意，这次它显示*Aborting*：

```
Ctrl+C
^CAborting...
```

第二个终端中的`cat`程序现在已退出。

## 它是如何工作的…

在这个程序中，我们注册了一个之前没有见过的额外信号：`SIGPIPE`信号。当另一端终止时，在我们的情况下是`cat`程序，我们的程序将收到一个`SIGPIPE`信号。如果我们没有捕获该信号，我们的程序将以信号 141 退出，并且不会发生清理。从这个退出代码，我们可以推断出这是由于`SIGPIPE`信号引起的，因为 141-128 = 13；信号 13 是`SIGPIPE`。有关保留返回值的解释，请参见*第二章*中的*图 2.2*，*使您的程序易于脚本化*。

在`cleanUp()`函数中，我们使用该信号号（`SIGPIPE`，它是 13 的宏）在接收器停止接收数据时打印特殊消息。

如果我们改为通过按下*Ctrl* + *C*中止`fifo-sender`程序，我们会得到另一条消息；即*Aborted*。

`mkfifo()`函数为我们创建了一个指定模式的 FIFO 文件。在这里，我们将模式指定为一个八进制数。在 C 中，任何以 0 开头的数字都是八进制数。

由于我们使用`open()`系统调用打开 FIFO，我们得到了一个`dprintf()`来将用户的消息打印到管道中。程序的第一个参数—`argv[1]`—是用户的消息。

只要 FIFO 在程序中保持打开状态，`cat`也将继续监听。这就是为什么我们可以在循环中每秒重复一次消息。

## 另请参阅

有关`mkfifo()`函数的深入解释，请参阅`man 3 mkfifo`。

有关可能信号的列表，请参阅`kill -L`。

要了解有关`dprintf()`的更多信息，请参阅`man 3 dprintf`手册页。

# FIFO – 构建接收器

在上一个示例中，我们编写了一个创建 FIFO 并向其写入消息的程序。我们还使用`cat`进行了测试以接收消息。在这个示例中，我们将编写一个 C 程序，从 FIFO 中读取。

从 FIFO 中读取与从常规文件或标准输入读取没有任何不同。

## 准备工作

在开始本教程之前，最好先完成上一个教程。我们将使用上一个教程中的程序将数据写入我们将在本教程中接收的 FIFO 中。

您还需要常规工具；即 GCC 编译器、Make 工具和通用 Makefile。

## 操作步骤如下...

在本教程中，我们将为前一个教程中编写的发送程序编写一个接收程序。让我们开始：

1.  将以下代码写入文件并保存为`fifo-receiver.c`。我们将使用文件流打开 FIFO，然后在循环中逐个字符读取，直到我们得到**文件结束**（**EOF**）：

```
#include <stdio.h>
int main(void)
{
    FILE *fp;
    signed char c;
    const char fifoname[] = "/tmp/my-2nd-fifo";
    if ( (fp = fopen(fifoname, "r")) == NULL )
    {
        perror("Can't open FIFO");
        return 1;
    }
    while ( (c = getc(fp)) != EOF )
        putchar(c);
    fclose(fp);
    return 0;
}
```

1.  编译程序：

```
$> make fifo-receiver
gcc -Wall -Wextra -pedantic -std=c99    fifo-receiver.c   -o fifo-receiver
```

1.  从上一个教程中启动`fifo-sender`并让其运行：

```
$> ./fifo-sender 'Hello from the sender'
```

1.  打开第二个终端并运行我们刚刚编译的`fifo-receiver`。在几秒钟后按*Ctrl* + *C*中止它：

```
fifo-sender will also abort, just like when we used the cat command to receive the data.
```

## 工作原理...

由于 FIFO 是文件系统上的一个文件，我们可以使用 C 中的常规函数（如文件流、`getc()`、`putchar()`等）从中接收数据。

这个程序类似于*第五章*中的`stream-read.c`程序，*使用文件 I/O 和文件系统操作*，只是这里我们逐个字符读取而不是逐行读取。

## 另请参阅

有关`getc()`和`putchar()`的更多信息，请参阅`man 3 getc`和`man 3 putchar`手册页。

# 消息队列 - 创建发送程序

另一种流行的 IPC 技术是**消息队列**。这基本上就是名字所暗示的。一个进程将消息留在队列中，另一个进程读取它们。

Linux 上有两种类型的消息队列：`mq_`函数，如`mq_open()`，`mq_send()`等。

了解如何使用消息队列使您能够从各种 IPC 技术中进行选择。

## 准备工作

对于本教程，我们只需要 GCC 编译器和 Make 工具。

## 操作步骤如下...

在本教程中，我们将创建发送程序。这个程序将创建一个新的消息队列并向其中添加一些消息。在下一个教程中，我们将接收这些消息：

1.  将以下代码写入文件并保存为`msg-sender.c`。由于代码中有一些新内容，我已将其分解为几个步骤。所有代码都放在一个文件中，名为`msg-sender.c`。

让我们从所需的头文件开始。我们还为最大消息大小定义了一个宏。然后，我们将创建一个名为`msgattr`的`mq_attr`类型的结构。然后设置它的成员；也就是说，我们将`mq_maxmsg`设置为 10，`mq_msgsize`设置为`MAX_MSG_SIZE`。第一个`mq_maxmsg`指定队列中的消息总数。第二个`mq_msgsize`指定消息的最大大小：

```
#include <stdio.h>
#include <mqueue.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <string.h>
#define MAX_MSG_SIZE 2048
int main(int argc, char *argv[])
{
   int md; /* msg queue descriptor */
   /* attributes for the message queue */
   struct mq_attr msgattr;
   msgattr.mq_maxmsg = 10;
   msgattr.mq_msgsize = MAX_MSG_SIZE;
```

1.  我们将把程序的第一个参数作为消息。因此，在这里，我们将检查用户是否输入了参数：

```
   if ( argc != 2)
   {
      fprintf(stderr, "Usage: %s 'my message'\n",
         argv[0]);
      return 1;
   }
```

1.  现在，是时候用`mq_open()`打开并创建消息队列了。第一个参数是队列的名称；在这里，它是`/my_queue`。第二个参数是标志，我们的情况下是`O_CREATE`和`O_RDWR`。这些是我们之前见过的相同标志，例如`open()`。第三个参数是权限模式；再次，这与文件相同。第四个和最后一个参数是我们之前创建的结构。`mq_open()`函数然后将消息队列描述符返回给`md`变量。

最后，我们使用`mq_send()`将消息发送到队列。这里，首先，我们给它`md`描述符。然后，我们有要发送的消息，在本例中是程序的第一个参数。然后，作为第三个参数，我们必须指定消息的大小。最后，我们必须为消息设置一个优先级；在这种情况下，我们将选择 1。它可以是任何正数（无符号整数）。

在退出程序之前，我们将做的最后一件事是使用`mq_close()`关闭消息队列描述符：

```
   md = mq_open("/my_queue", O_CREAT|O_RDWR, 0644, 
      &msgattr); 
   if ( md == -1 )
   {
      perror("Creating message queue");
      return 1;
   }
   if ( (mq_send(md, argv[1], strlen(argv[1]), 1))
      == -1 )
   {
      perror("Message queue send");
      return 1;
   }
   mq_close(md);
   return 0;
}
```

1.  编译程序。请注意，我们必须链接`rt`库，该库代表**实时扩展库**：

```
$> gcc -Wall -Wextra -pedantic -std=c99 -lrt \
> msg-sender.c -o msg-sender
```

1.  现在，运行程序并向队列发送三到四条消息：

```
$> ./msg-sender "The first message to the queue"
$> ./msg-sender "The second message"
$> ./msg-sender "And another message"
```

## 工作原理…

在这个食谱中，我们使用了 POSIX 消息队列函数来创建一个新队列，然后向其发送消息。当我们创建队列时，我们指定该队列可以包含最多 10 条消息，使用`msgattr`的`mq_maxmsg`成员。

我们还使用`mq_msgsize`成员将每条消息的最大长度设置为 2,048 个字符。

当我们调用`mq_open()`时，我们将队列命名为`/my_queue`。消息队列必须以斜杠开头。

队列创建后，我们使用`mq_send()`向其发送消息。

在这个食谱的最后，我们向队列发送了三条消息。这些消息现在已排队，等待接收。在下一个食谱中，我们将学习如何编写一个接收这些消息并在屏幕上打印它们的程序。

## 另请参阅

在 Linux 的`man 7 mq_overview`手册页中有关于 POSIX 消息队列功能的很好的概述。

# 消息队列 - 创建接收器

在上一个食谱中，我们构建了一个程序，创建了一个名为`/my_queue`的消息队列，然后向其发送了三条消息。在这个食谱中，我们将创建一个接收来自该队列的消息的程序。

## 准备工作

在开始这个食谱之前，您需要完成上一个食谱。否则，我们将收不到任何消息。

您还需要 GCC 编译器和 Make 工具来完成这个食谱。

## 操作步骤…

在这个食谱中，我们将接收上一个食谱中发送的消息：

1.  在文件中写入以下代码，并将其保存为`msg-receiver.c`。这段代码比发送程序的代码要长一些，因此它被分成了几个步骤，每个步骤都解释了一部分代码。不过，请记住，所有代码都放在同一个文件中。我们将从头文件、变量、结构和名为`buffer`的字符指针开始。稍后我们将使用它来分配内存：

```
#include <stdio.h>
#include <mqueue.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <stdlib.h>
#include <string.h>
int main(void)
{
   int md; /* msg queue descriptor */
   char *buffer;
   struct mq_attr msgattr;
```

1.  下一步是使用`mq_open()`打开消息队列。这次，我们只需要提供两个参数；队列的名称和标志。在这种情况下，我们只想从队列中读取：

```
   md = mq_open("/my_queue", O_RDONLY);
   if (md == -1 )
   {
      perror("Open message queue");
      return 1;
   }
```

1.  现在，我们还想使用`mq_getattr()`获取消息队列的属性。一旦我们有了队列的属性，我们就可以使用其`mq_msgsize`成员使用`calloc()`为该大小的消息分配内存。在本书中，我们之前没有看到`calloc()`。第一个参数是我们要为其分配内存的元素数，而第二个参数是每个元素的大小。然后，`calloc()`函数返回指向该内存的指针（在我们的情况下，就是`buffer`）：

```
   if ( (mq_getattr(md, &msgattr)) == -1 )
   {
      perror("Get message attribute");
      return 1;
   }
   buffer = calloc(msgattr.mq_msgsize, 
      sizeof(char));
   if (buffer == NULL)
   {
      fprintf(stderr, "Couldn't allocate memory");
      return 1;
   }
```

1.  接下来，我们将使用`mq_attr`结构的另一个成员`mq_curmsgs`，它包含队列中当前的消息数。首先，我们将打印消息数。然后，我们将使用`for`循环遍历所有消息。在循环内部，首先使用`mq_receive`接收消息。然后，我们使用`printf()`打印消息。最后，在迭代下一条消息之前，我们使用`memset()`将整个内存重置为 NULL 字符。

`mq_receive`的第一个参数是描述符，第二个参数是消息所在的缓冲区，第三个参数是消息的大小，第四个参数是消息的优先级，在这种情况下是 NULL，表示我们首先接收所有最高优先级的消息：

```
   printf("%ld messages in queue\n", 
      msgattr.mq_curmsgs);
   for (int i = 0; i<msgattr.mq_curmsgs; i++)
   {
      if ( (mq_receive(md, buffer, 
      msgattr.mq_msgsize, NULL)) == -1 )
      {
         perror("Message receive");
         return 1;
      }
      printf("%s\n", buffer);
      memset(buffer, '\0', msgattr.mq_msgsize);
   }
```

1.  最后，我们有一些清理工作要做。首先，我们必须使用`free()`释放缓冲区指向的内存。然后，我们必须关闭`md`队列描述符，然后使用`mq_unlink()`从系统中删除队列：

```
   free(buffer);
   mq_close(md);
   mq_unlink("/my_queue");
   return 0;
}
```

1.  现在，是时候编译程序了：

```
$> gcc -Wall -Wextra -pedantic -std=c99 -lrt \
> msg-reveiver.c -o msg-reveiver
```

1.  最后，让我们使用我们的新程序接收消息：

```
$> ./msg-reveiver 
3 messages in queue
The first message to the queue
The second message
And another message
```

1.  如果我们现在尝试重新运行程序，它将简单地指出没有这样的文件或目录存在。这是因为我们使用`mq_unlink()`删除了消息队列：

```
$> ./msg-reveiver 
Open message queue: No such file or directory
```

## 工作原理…

在上一个示例中，我们向`/my_queue`发送了三条消息。使用本示例中创建的程序，我们接收了这些消息。

要打开队列，我们使用了创建队列时使用的相同函数；也就是`mq_open()`。但这一次——因为我们正在打开一个已经存在的队列——我们只需要提供两个参数；即队列的名称和标志。

对`mq_`函数的每次调用都进行错误检查。如果发生错误，我们将使用`perror()`打印错误消息，并返回到 shell 并返回 1。

在从队列中读取实际消息之前，我们使用`mq_getattr()`获取队列的属性。通过这个函数调用，我们填充了`mq_attr`结构。对于读取消息来说，最重要的两个成员是`mq_msgsize`，它是队列中每条消息的最大大小，以及`mq_curmsgs`，它是当前队列中的消息数。

我们使用`mq_msgsize`中的最大消息大小来使用`calloc()`为消息缓冲区分配内存。`calloc()`函数返回“零化”的内存，而它的对应函数`malloc()`则不会。

要分配内存，我们需要创建一个指向我们想要的类型的指针。这就是我们在程序开始时使用`char *buffer`所做的。`calloc()`函数接受两个参数：要分配的元素数量和每个元素的大小。在这里，我们希望元素的数量与`mq_msgsize`值包含的相同。而每个元素都是`char`，所以每个元素的大小应该是`sizeof(char)`。然后函数返回一个指向内存的指针，在我们的情况下保存在`char`指针的`buffer`中。

然后，当我们接收队列消息时，我们在循环的每次迭代中将它们保存在这个缓冲区中。

循环遍历所有消息。我们从`mq_curmsgs`成员中得到消息的数量。

最后，一旦我们读完了所有的消息，我们关闭并删除了队列。

## 另请参阅

关于`mq_attr`结构的更多信息，我建议你阅读`man 3 mq_open`手册页面。

我们在这个和上一个示例中涵盖的每个函数都有自己的手册页面；例如，`man 3 mq_send`，`man 3 mq_recevie`，`man 3 mq_getattr`等等。

如果你对`calloc()`和`malloc()`函数不熟悉，我建议你阅读`man 3 calloc`。这个手册页面涵盖了`malloc()`，`calloc()`，`free()`和一些其他相关函数。

`memset()`函数也有自己的手册页面；即`man 3 memset`。

# 使用共享内存在子进程和父进程之间通信

在这个示例中，我们将学习如何在两个相关的进程——父进程和子进程之间使用**共享内存**。共享内存以各种形式存在，并且可以以不同的方式使用。在本书中，我们将专注于 POSIX 共享内存函数。

Linux 中的共享内存可以在相关进程之间使用，正如我们将在本示例中探讨的那样，还可以在无关的进程之间使用`/dev/shm`目录。我们将在下一个示例中看到这一点。

在这个示例中，我们将使用*匿名*共享内存——即不由文件支持的内存。

共享内存就像它听起来的那样——一块在进程之间共享的内存。

了解如何使用共享内存将使您能够编写更高级的程序。

## 准备工作

对于这个示例，您只需要 GCC 编译器和 Make 工具。

## 如何做…

在这个示例中，我们将编写一个使用共享内存的程序。首先，在分叉之前，进程将向共享内存写入一条消息。然后，在分叉之后，子进程将替换共享内存中的消息。最后，父进程将再次替换共享内存的内容。让我们开始吧：

1.  将以下代码写入一个文件中，并将其命名为`shm-parent-child.c`。像往常一样，我将把代码分成几个较小的步骤。尽管所有的代码都放在同一个文件中。首先，我们将写入所有的头文件。这里有相当多的头文件。我们还将为我们的内存大小定义一个宏。然后，我们将我们的三条消息写成字符数组常量：

```
#include <stdio.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/wait.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#define DATASIZE 128
int main(void)
{
   char *addr;
   int status;
   pid_t pid;
   const char startmsg[] = "Hello, we are running";
   const char childmsg[] = "Hello from child";
   const char parentmsg[] = "New msg from parent";
```

1.  现在来到令人兴奋的部分——映射共享内存空间。我们需要向内存映射函数`mmap()`提供总共六个参数。

第一个参数是内存地址，我们将其设置为 NULL——这意味着内核会为我们处理它。

第二个参数是内存区域的大小。

第三个参数是内存应该具有的保护。在这里，我们将其设置为可写和可读。

第四个参数是我们的标志，我们将其设置为共享和匿名——这意味着它可以在进程之间共享，并且不会由文件支持。

第五个参数是文件描述符。但在我们的情况下，我们使用的是匿名的，这意味着这块内存不会由文件支持。因此，出于兼容性原因，我们将其设置为-1。

最后一个参数是偏移量，我们将其设置为 0：

```
   addr = mmap(NULL, DATASIZE, 
      PROT_WRITE | PROT_READ, 
      MAP_SHARED | MAP_ANONYMOUS, -1, 0);
   if (addr == MAP_FAILED)
   {
      perror("Memory mapping failed");
      return 1;
   }
```

1.  现在内存已经准备好了，我们将使用`memcpy()`将我们的第一条消息复制到其中。`memcpy()`的第一个参数是指向内存的指针，在我们的例子中是`addr`字符指针。第二个参数是我们要从中复制的数据或消息，在我们的例子中是`startmsg`。最后一个参数是我们要复制的数据的大小，在这种情况下是`startmsg`中字符串的长度+1。`strlen()`函数不包括终止的空字符；这就是为什么我们需要加 1。

然后，我们打印进程的 PID 和共享内存中的消息。之后，我们进行分叉：

```
   memcpy(addr, startmsg, strlen(startmsg) + 1);
   printf("Parent PID is %d\n", getpid());
   printf("Original message: %s\n", addr);
   if ( (pid = fork()) == -1 )
   {
      perror("Can't fork");
      return 1;
   }
```

1.  如果我们在子进程中，我们将子进程的消息复制到共享内存中。如果我们在父进程中，我们将等待子进程。然后，我们可以将父进程的消息复制到内存中，并打印两条消息。最后，我们将通过取消映射共享内存来清理。尽管这并不是严格要求的：

```
   if (pid == 0)
   {
      /* child */
      memcpy(addr, childmsg, strlen(childmsg) + 1);
   }
   else if(pid > 0)
   {
      /* parent */
      waitpid(pid, &status, 0);
      printf("Child executed with PID %d\n", pid);
      printf("Message from child: %s\n", addr);
      memcpy(addr, parentmsg, 
         strlen(parentmsg) + 1);
      printf("Parent message: %s\n", addr);
   }
   munmap(addr, DATASIZE);
   return 0;
}
```

1.  编译程序，以便我们可以试一试。请注意，我们在这里使用了另一个 C 标准——`MAP_ANONYMOUS`宏，但**GNU11**有。**GNU11**是**C11**标准，带有一些额外的 GNU 扩展。还要注意，我们链接了*实时扩展*库：

```
$> gcc -Wall -Wextra -std=gnu11 -lrt \
> shm-parent-child.c -o shm-parent-child
```

1.  现在，我们可以测试程序了：

```
$> ./shm-parent-child 
Parent PID is 9683
Original message: Hello, we are running
Child executed with PID 9684
Message from child: Hello from child
Parent message: New msg from parent
```

## 工作原理…

共享内存是不相关进程、相关进程和线程之间的常见 IPC 技术。在这个示例中，我们看到了如何在父进程和子进程之间使用共享内存。

使用`mmap()`映射内存区域。这个函数返回映射内存的地址。如果发生错误，它将返回`MAP_FAILED`宏。一旦我们映射了内存，我们就检查指针变量是否为`MAP_FAILED`，并在出现错误时中止它。

一旦我们映射了内存并获得了指向它的指针，我们就使用`memcpy()`将数据复制到其中。

最后，我们使用`munmap()`取消映射内存。这并不是严格必要的，因为当最后一个进程退出时，它将被取消映射。但是，不这样做是一个不好的习惯。您应该始终在使用后进行清理，并释放任何分配的内存。

## 另请参阅

有关`mmap()`和`munmap()`的更详细解释，请参见`man 2 mmap`手册页。有关`memcpy()`的详细解释，请参见`man 3 memcpy`手册页。

有关各种 C 标准及 GNU 扩展的更详细解释，请参见[`gcc.gnu.org/onlinedocs/gcc/Standards.html`](https://gcc.gnu.org/onlinedocs/gcc/Standards.html)。

# 在不相关进程之间使用共享内存

在之前的示例中，我们在子进程和父进程之间使用了共享内存。在这个示例中，我们将学习如何使用文件描述符将映射内存共享给两个不相关的进程。以这种方式使用共享内存会自动在`/dev/shm`目录中创建内存的底层文件，其中**shm**代表**共享内存**。

了解如何在不相关的进程之间使用共享内存扩大了您使用这种 IPC 技术的范围。

## 准备工作

对于这个示例，您只需要 GCC 编译器和 Make 工具。

## 操作步骤…

首先，我们将编写一个程序，打开并创建一个共享内存的文件描述符，并映射内存。然后，我们将编写另一个程序来读取内存区域。与之前的示例不同，这次我们将在这里写入和检索一个由三个浮点数组成的**数组**，而不仅仅是一个消息。

### 创建写入程序

首先让我们创建写入程序：

1.  第一步是创建一个程序，用于创建共享内存并向其写入一些数据。将以下代码写入文件并保存为`write-memory.c`。和往常一样，代码将被分成几个步骤，但所有代码都放在一个文件中。

就像在之前的示例中一样，我们将有一堆头文件。然后，我们将创建所有需要的变量。在这里，我们需要一个文件描述符变量。请注意，即使我在这里称其为文件描述符，它实际上是一个内存区域的描述符。`memid`包含内存映射描述符的名称。然后，我们必须使用`shm_open()`来打开和创建“文件描述符”：

```
#include <stdio.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#define DATASIZE 128
int main(void)
{
   int fd;
   float *addr;
   const char memid[] = "/my_memory";
   const float numbers[3] = { 3.14, 2.718, 1.202};
   /* create shared memory file descriptor */
   if ( (fd = shm_open(memid, 
      O_RDWR | O_CREAT, 0600)) == -1)
   {
      perror("Can't open memory fd");
      return 1;
   }
```

1.  文件支持的内存最初大小为 0 字节。要将其扩展到我们的 128 字节，我们必须使用`ftruncate()`进行截断。

```
   /* truncate memory to DATASIZE */
   if ( (ftruncate(fd, DATASIZE)) == -1 )
   {
      perror("Can't truncate memory");
      return 1;
   }
```

1.  现在，我们必须映射内存，就像我们在之前的示例中所做的那样。但是这次，我们将给它`fd`文件描述符，而不是-1。我们还省略了`MAP_ANONYMOUS`部分，从而使这个内存由文件支持。然后，我们必须使用`memcpy()`将我们的浮点数数组复制到内存中。为了让读取程序有机会读取内存，我们必须暂停程序，并使用`getchar()`等待*Enter*键。然后，只需要清理工作，取消映射内存，并使用`shm_unlink()`删除文件描述符和底层文件：

```
   /* map memory using our file descriptor */
   addr = mmap(NULL, DATASIZE, PROT_WRITE, 
      MAP_SHARED, fd, 0);
   if (addr == MAP_FAILED)
   {
      perror("Memory mapping failed");
      return 1;
   }
   /* copy data to memory */
   memcpy(addr, numbers, sizeof(numbers));
   /* wait for enter */
   printf("Hit enter when finished ");
   getchar();
   /* clean up */
   munmap(addr, DATASIZE);
   shm_unlink(memid);
   return 0;
}
```

1.  现在，让我们编译这个程序：

```
$> gcc -Wall -Wextra -std=gnu11 -lrt write-memory.c \
> -o write-memory
```

### 创建读取程序

现在，让我们创建读取程序：

1.  现在，我们将编写一个程序，用于读取内存区域并打印数组中的数字。编写以下程序并将其保存为`read-memory.c`。这个程序类似于`write-memory.c`，但不是向内存写入，而是从内存读取：

```
#include <stdio.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#define DATASIZE 128
int main(void)
{
   int fd;
   float *addr;
   const char memid[] = "/my_memory";
   float numbers[3];
   /* open memory file descriptor */
   fd = shm_open(memid, O_RDONLY, 0600);
   if (fd == -1)
   {
      perror("Can't open file descriptor");
      return 1;
   }
   /* map shared memory */
   addr = mmap(NULL, DATASIZE, PROT_READ, 
      MAP_SHARED, fd, 0);
   if (addr == MAP_FAILED)
   {
      perror("Memory mapping failed");
      return 1;
   }
   /* read the memory and print the numbers */
   memcpy(numbers, addr, sizeof(numbers));
   for (int i = 0; i<3; i++)
   {
      printf("Number %d: %.3f\n", i, numbers[i]);
   }
   return 0;
}
```

1.  现在，编译这个程序：

```
$> gcc -Wall -Wextra -std=gnu11 -lrt read-memory.c \
> -o read-memory
```

### 测试一切

按照以下步骤进行：

1.  现在，是时候尝试一切了。打开终端并运行我们编译的`write-memory`程序。让程序保持运行：

```
$> ./write-memory 
Hit enter when finished
```

1.  打开另一个终端，查看`/dev/shm`中的文件：

```
$> ls -l /dev/shm/my_memory 
-rw------- 1 jake jake 128 jan 18 19:19 /dev/shm/my_memory
```

1.  现在运行我们刚刚编译的`read-memory`程序。这将从共享内存中检索三个数字并将它们打印在屏幕上：

```
$> ./read-memory 
Number 0: 3.140
Number 1: 2.718
Number 2: 1.202
```

1.  返回运行`write-memory`程序的终端，然后按*Enter*。这样做将清理并删除文件。完成后，让我们看看文件是否仍然在`/dev/shm`中：

```
./write-memory 
Hit enter when finished Enter
$> ls -l /dev/shm/my_memory
ls: cannot access '/dev/shm/my_memory': No such file or directory
```

## 工作原理…

使用非匿名共享内存与我们在之前的示例中所做的类似。唯一的例外是，我们首先使用`shm_open()`打开一个特殊的文件描述符。正如您可能已经注意到的，标志与常规的`open()`调用相似；即，`O_RDWR`用于读取和写入，`O_CREATE`用于在文件不存在时创建文件。以这种方式使用`shm_open()`会在`/dev/shm`目录中创建一个文件，文件名由第一个参数指定。甚至权限模式设置方式与常规文件相同——在我们的情况下，`0600`用于用户读写，其他人没有权限。

我们从`shm_open()`获得的文件描述符然后传递给`mmap()`调用。我们还在`mmap()`调用中省略了`MAP_ANONYMOUS`宏，就像我们在前面的示例中看到的那样。跳过`MAP_ANONYMOUS`意味着内存将不再是匿名的，这意味着它将由文件支持。我们使用`ls -l`检查了这个文件，并看到它确实有我们给它的名称和正确的权限。

我们编写的下一个程序使用`shm_open()`打开了相同的共享内存文件描述符。在`mmap()`之后，我们循环遍历了内存区域中的浮点数。

最后，一旦我们在`write-memory`程序中按下*Enter*，`/dev/shm`中的文件将使用`shm_unlink()`被删除。

## 另请参阅

在`man 3 shm_open`手册页中有关于`shm_open()`和`shm_unlink()`的更多信息。

# Unix 套接字-创建服务器

**Unix 套接字**类似于**TCP/IP**套接字，但它们只是本地的，并且由文件系统上的套接字文件表示。但是与 Unix 套接字一起使用的整体函数与 TCP/IP 套接字的几乎相同。Unix 套接字的完整名称是*Unix 域套接字*。

Unix 套接字是程序在本地机器上进行通信的常见方式。

了解如何使用 Unix 套接字将使编写需要在它们之间通信的程序变得更容易。

## 准备工作

在这个示例中，您只需要 GCC 编译器、Make 工具和通用 Makefile。

## 如何做…

在这个示例中，我们将编写一个充当服务器的程序。它将从客户端接收消息，并在每次接收到消息时回复“*消息已收到*”。当服务器或客户端退出时，它还会自行清理。让我们开始吧：

1.  将以下代码写入文件并保存为`unix-server.c`。这段代码比我们以前的大多数示例都要长，因此它被分成了几个步骤。不过所有的代码都在同一个文件中。

这里有相当多的头文件。我们还将为我们将接受的最大消息长度定义一个宏。然后，我们将为`cleanUp()`函数编写原型，该函数将用于清理文件。这个函数也将被用作信号处理程序。然后，我们将声明一些全局变量（以便它们可以从`cleanUp()`中访问）：

```
#define _XOPEN_SOURCE 700
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/un.h>
#include <string.h>
#include <unistd.h>
#include <signal.h>
#include <stdlib.h>
#include <errno.h>
#define MAXLEN 128
void cleanUp(int signum);
const char sockname[] = "/tmp/my_1st_socket";
int connfd;
int datafd;
```

1.  现在，是时候开始编写`main()`函数并声明一些变量了。到目前为止，这大部分对您来说应该是熟悉的。我们还将在这里为所有信号注册信号处理程序。新的是`sockaddr_un`结构。这将包含套接字类型和文件路径：

```
int main(void)
{
   int ret;
   struct sockaddr_un addr;
   char buffer[MAXLEN];
   struct sigaction action;
   /* prepare for sigaction */
   action.sa_handler = cleanUp;
   sigfillset(&action.sa_mask);
   action.sa_flags = SA_RESTART;
   /* register the signals we want to handle */
   sigaction(SIGTERM, &action, NULL);
   sigaction(SIGINT, &action, NULL);
   sigaction(SIGQUIT, &action, NULL);
   sigaction(SIGABRT, &action, NULL);
   sigaction(SIGPIPE, &action, NULL);
```

1.  现在我们已经准备好了所有的信号处理程序、变量和结构，我们可以使用`socket()`函数创建一个套接字文件描述符。一旦处理好了这个问题，我们将设置连接的类型（*family*类型）和套接字文件的路径。然后，我们将调用`bind()`，这将为我们绑定套接字，以便我们可以使用它：

```
   /* create socket file descriptor */
   connfd = socket(AF_UNIX, SOCK_SEQPACKET, 0);
   if ( connfd == -1 )
   {
      perror("Create socket failed");
      return 1;
   }
   /* set address family and socket path */
   addr.sun_family = AF_UNIX;
   strcpy(addr.sun_path, sockname);
   /* bind the socket (we must cast our sockaddr_un
    * to sockaddr) */
   if ( (bind(connfd, (const struct sockaddr*)&addr, 
      sizeof(struct sockaddr_un))) == -1 )
   {
      perror("Binding socket failed");
      return 1;
   }
```

1.  现在，我们将通过调用`listen()`准备好连接的套接字文件描述符。第一个参数是套接字文件描述符，而第二个参数是我们想要的后备大小。一旦我们做到了这一点，我们将使用`accept()`接受一个连接。这将给我们一个新的套接字（`datafd`），我们将在发送和接收数据时使用它。一旦连接被接受，我们可以在本地终端上打印*客户端已连接*：

```
   /* prepare for accepting connections */
   if ( (listen(connfd, 20)) == -1 )
   {
      perror("Listen error");
      return 1;
   }
   /* accept connection and create new file desc */
   datafd = accept(connfd, NULL, NULL);
   if (datafd == -1 )
   {
      perror("Accept error");
      return 1;
   }
   printf("Client connected\n");
```

1.  现在，我们将开始程序的主循环。在外部循环中，我们只会在接收到消息时写一个确认消息。在内部循环中，我们将从新的套接字文件描述符中读取数据，将其保存在`buffer`中，然后在我们的终端上打印出来。如果`read()`返回-1，那么出现了问题，我们必须跳出内部循环读取下一行。如果`read()`返回 0，那么客户端已断开连接，我们必须运行`cleanUp()`并退出：

```
   while(1) /* main loop */
   {
      while(1) /* receive message, line by line */
      {
         ret = read(datafd, buffer, MAXLEN);
         if ( ret == -1 )
         {
            perror("Error reading line");
            cleanUp(1);
         }
         else if ( ret == 0 )
         {
            printf("Client disconnected\n");
            cleanUp(1);
         }
         else
         {
            printf("Message: %s\n", buffer);
            break;
         }
      }
   /* write a confirmation message */
   write(datafd, "Message received\n", 18);
   }
   return 0;
}
```

1.  最后，我们必须创建`cleanUp()`函数的主体：

```
void cleanUp(int signum)
{
   printf("Quitting and cleaning up\n");
   close(connfd);
   close(datafd);
   unlink(sockname);
   exit(0);
}
```

1.  现在编译程序。这次，我们将从 GCC 得到一个关于`cleanUp（）`函数中未使用的变量`signum`的警告。这是因为我们从未在`cleanUp（）`内部使用过`signum`变量，所以我们可以安全地忽略这个警告：

```
$> make unix-server
gcc -Wall -Wextra -pedantic -std=c99    unix-server.c   -o unix-server
unix-server.c: In function 'cleanUp':
unix-server.c:94:18: warning: unused parameter 'signum' [-Wunused-parameter]
 void cleanUp(int signum)
              ~~~~^~~~~~
```

1.  运行程序。由于我们没有客户端，它暂时不会说或做任何事情。但是它确实创建了套接字文件。将程序保持不变：

```
$> ./unix-server
```

1.  打开一个新的终端并查看套接字文件。在这里，我们可以看到它是一个套接字文件：

```
$> ls -l /tmp/my_1st_socket 
srwxr-xr-x 1 jake jake 0 jan 19 18:35 /tmp/my_1st_socket
$> file /tmp/my_1st_socket 
/tmp/my_1st_socket: socket
```

1.  现在，回到运行服务器程序的终端，并使用*Ctrl* + *C*中止它。然后，看看文件是否还在那里（不应该在那里）：

```
./unix-server
Ctrl+C
Quitting and cleaning up
$> file /tmp/my_1st_socket 
/tmp/my_1st_socket: cannot open `/tmp/my_1st_socket' (No such file or directory)
```

## 它是如何工作的…

`sockaddr_un`结构是 Unix 域套接字的特殊结构。还有一个称为`sockaddr_in`的结构，用于 TCP/IP 套接字。`_un`结尾代表 Unix 套接字，而`_in`代表互联网家族套接字。

我们用来创建套接字文件描述符的`socket（）`函数需要三个参数：地址族（`AF_UNIX`），类型（`SOCK_SEQPACKET`，提供双向通信），和协议。我们将协议指定为 0，因为在套接字中没有可以选择的协议。

还有一个称为`sockaddr`的一般结构。当我们将我们的`sockaddr_un`结构作为`bind（）`的参数传递时，我们需要将其强制转换为一般类型`sockaddr`，因为这是函数期望的——更确切地说，是`sockaddr`指针。我们为`bind（）`提供的最后一个参数是结构的大小；也就是`sockaddr_un`。

一旦我们创建了套接字并用`bind（）`绑定了它，我们就用`listen（）`准备好接受传入的连接。

最后，我们使用`accept（）`接受传入的连接。这给了我们一个新的套接字文件描述符，然后我们用它来发送和接收消息。

## 另请参阅

在这个示例中，我们使用的函数的手册页中有一些更深入的信息。我建议你把它们都看一遍：

+   `man 2 socket`

+   `man 2 bind`

+   `man 2 listen`

+   `man 2 accept`

# Unix 套接字 - 创建客户端

在上一个示例中，我们创建了一个 Unix 域套接字服务器。在这个示例中，我们将为该套接字创建一个客户端，然后在客户端和服务器之间进行通信。

在这个示例中，我们将看到如何使用套接字在服务器和客户端之间进行通信。了解如何在套接字上进行通信对于使用套接字是至关重要的。

## 准备工作

在做这个示例之前，你应该已经完成了上一个示例；否则，你就没有服务器可以交谈了。

对于这个示例，你还需要 GCC 编译器、Make 工具和通用的 Makefile。

## 如何做…

在这个示例中，我们将为上一个示例中编写的服务器编写一个客户端。一旦它们连接，客户端就可以向服务器发送消息，服务器将以*收到消息*作出回应。让我们开始吧：

1.  在文件中写入以下代码并将其保存为`unix-client.c`。由于这段代码也有点长，它被分成了几个步骤。但所有的代码都在`unix-client.c`文件中。这个程序的前半部分与服务器的前半部分类似，只是我们有两个缓冲区而不是一个，而且没有信号处理：

```
#define _XOPEN_SOURCE 700
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/un.h>
#include <string.h>
#include <unistd.h>
#include <signal.h>
#include <stdlib.h>
#include <errno.h>
#define MAXLEN 128
int main(void)
{
   const char sockname[] = "/tmp/my_1st_socket";
   int fd;
   struct sockaddr_un addr;
   char sendbuffer[MAXLEN];
   char recvbuffer[MAXLEN];
   /* create socket file descriptor */
   fd = socket(AF_UNIX, SOCK_SEQPACKET, 0);
   if ( fd == -1 )
   {
      perror("Create socket failed");
      return 1;
   }
   /* set address family and socket path */
   addr.sun_family = AF_UNIX;
   strcpy(addr.sun_path, sockname);
```

1.  现在，我们将使用`connect（）`来初始化与服务器的连接，而不是使用`bind（）`，`listen（）`和`accept（）`。`connect（）`函数接受与`bind（）`相同的参数：

```
   /* connect to the server */
   if ( (connect(fd, (const struct sockaddr*) &addr, 
      sizeof(struct sockaddr_un))) == -1 )
   {
      perror("Can't connect");
      fprintf(stderr, "The server is down?\n");
      return 1;
   }
```

1.  现在我们已经连接到服务器，我们可以使用`write（）`来通过套接字文件描述符发送消息。在这里，我们将使用`fgets（）`将用户的消息读入缓冲区，将**换行符**转换为**空字符**，然后将缓冲区写入文件描述符：

```
   while(1) /* main loop */
   {
      /* send message to server */
      printf("Message to send: ");
      fgets(sendbuffer, sizeof(sendbuffer), stdin);
      sendbuffer[strcspn(sendbuffer, "\n")] = '\0';
      if ( (write(fd, sendbuffer, 
         strlen(sendbuffer) + 1)) == -1 )
      {
         perror("Couldn't write");
         break;
      }
      /* read response from server */
      if ( (read(fd, recvbuffer, MAXLEN)) == -1 )
      {
         perror("Can't read");
         return 1;
      }
      printf("Server said: %s\n", recvbuffer);
   }
   return 0;
}
```

1.  编译程序：

```
$> make unix-client
gcc -Wall -Wextra -pedantic -std=c99    unix-client.c   -o unix-client
```

1.  现在让我们尝试运行程序。由于服务器尚未启动，它不会工作：

```
$> ./unix-client 
Can't connect: No such file or directory
The server is down?
```

1.  在一个单独的终端中启动服务器并让它保持运行：

```
$> ./unix-server
```

1.  返回到具有客户端的终端并重新运行它：

```
$> ./unix-client 
Message to send:
```

现在你应该在服务器上看到一条消息，上面写着*客户端已连接*。

1.  在客户端程序中写一些消息。当您按下*Enter*键时，您应该会在服务器上看到它们出现。发送几条消息后，按下*Ctrl* + *C*：

```
$> ./unix-client 
Message to send: Hello, how are you?
Server said: Message received
Message to send: Testing 123           
Server said: Message received
Message to send: Ctrl+C
```

1.  切换到带有服务器的终端。您应该会看到类似于这样的内容：

```
Client connected
Message: Hello, how are you?
Message: Testing 123
Client disconnected
Quitting and cleaning up
```

## 工作原理…

在上一个示例中，我们编写了一个套接字服务器。在这个示例中，我们编写了一个客户端，使用`connect()`系统调用连接到该服务器。这个系统调用接受与`bind()`相同的参数。一旦连接建立，服务器和客户端都可以使用`write()`和`read()`从套接字文件描述符中写入和读取（双向通信）。

因此，实质上，一旦连接建立，它与使用文件描述符读写文件并没有太大不同。

## 另请参阅

有关`connect()`系统调用的更多信息，请参阅`man 2 connect`手册页。
