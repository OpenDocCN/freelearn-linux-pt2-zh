# 第九章：终端 I/O 和更改终端行为

在本章中，我们将学习**TTY**（**TeleTYpewriter**的缩写）和**PTY**（**Pseudo-TeletYpewriter**的缩写）是什么，以及如何获取有关它们的信息。我们还将学习如何设置它们的属性。然后，我们编写一个接收输入但不回显文本的小程序——非常适合密码提示。我们还编写一个检查当前终端大小的程序。

终端可以采用多种形式——例如，在 X 中的终端窗口（图形前端）；通过*Ctrl* + *Alt* + *F1*到*F7*访问的七个终端；旧的串行终端；拨号终端；或者远程终端，比如**Secure Shell**（**SSH**）。

**TTY**是硬件终端，比如通过*Ctrl* + *Alt* + *F1*到*F7*访问的控制台，或者串行控制台。

一个`xterm`，`rxvt`，`tmux`。也可以是远程终端，比如 SSH。

由于我们在日常生活中都使用 Linux 终端，了解如何获取有关它们的信息并控制它们可以帮助我们编写更好的软件。一个例子是在密码提示中隐藏密码。

在本章中，我们将涵盖以下内容：

+   查看终端信息

+   使用`stty`更改终端设置

+   调查 TTY 和 PTY 并向它们写入

+   检查它是否是 TTY

+   创建一个 PTY

+   禁用密码提示的回显

+   读取终端大小

# 技术要求

在本章中，我们将需要所有常用的工具，比如`screen`。如果您还没有安装，可以使用您发行版的软件包管理器进行安装——例如，对于 Debian/Ubuntu，可以使用`sudo apt-get install screen`，对于 CentOS/Fedora，可以使用`sudo dnf install screen`。

本章的所有代码示例都可以从[`github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch9`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch9)下载。

查看以下链接以查看实际操作视频：[`bit.ly/2O8j7Lu`](https://bit.ly/2O8j7Lu)

# 查看终端信息

在这个配方中，我们将学习更多关于 TTY 和 PTY 是什么，以及如何读取它们的属性和信息。这将有助于我们在本章中继续了解 TTY。在这里，我们将学习如何找出我们正在使用的 TTY 或 PTY，它在文件系统中的位置，以及如何读取它的属性。

## 准备工作

这个配方没有特殊要求。我们只会使用已经安装的标准程序。

## 如何做…

在这个配方中，我们将探讨如何找到自己的 TTY，它具有什么属性，它的对应文件在哪里，以及它是什么类型的 TTY：

1.  首先在终端中输入`tty`。这将告诉您在系统上使用的 TTY。在单个系统上可以有许多 TTY 和 PTY。它们每个都由系统上的一个文件表示：

```
$> tty
/dev/pts/24
```

1.  现在，让我们检查一下那个文件。正如我们在这里看到的，这是一种特殊的文件类型，称为*字符特殊*：

```
$> ls -l /dev/pts/24
crw--w---- 1 jake tty 136, 24 jan  3 23:19 /dev/pts/24
$> file /dev/pts/24 
/dev/pts/24: character special (136/24)
```

1.  现在，让我们使用一个名为`stty`的程序来检查终端的属性。`-a`选项告诉`stty`显示所有属性。我们得到的信息，例如终端的大小（行数和列数）；它的速度（只在串行终端、拨号等上重要）；用于`-parenb`的*Ctrl*键组合。所有没有减号的值，比如`cs8`，都是启用的：

```
$> stty -a
speed 38400 baud; rows 14; columns 88; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = M-^?; eol2 = M-^?;
swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V;
discard = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 hupcl -cstopb cread -clocal -crtscts
-ignbrk brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc
ixany imaxbel iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl
echoke -flusho -extproc
```

1.  还可以查看另一个终端的属性，假设您拥有它，这意味着已登录用户必须是您。如果我们尝试查看另一个用户的终端，将会收到*权限被拒绝*的错误：

```
$> stty -F /dev/pts/33 
speed 38400 baud; line = 0;
lnext = <undef>; discard = <undef>; min = 1; time = 0; -brkint -icrnl ixoff -imaxbel iutf8
-icanon -echo
$> stty -F /dev/tty2
stty: /dev/tty2: Permission denied
```

## 工作原理…

单个 Linux 系统可以有数百或数千个已登录用户。每个用户都通过 TTY 或 PTY 连接。在过去，这通常是硬件终端(TTY)通过串行线连接到计算机。如今，硬件终端相当罕见；相反，我们通过**SSH**登录或使用终端程序。

在我们的例子中，当前用户登录在`/dev/pts/24`上；那是*pts*，而不是*pty*。PTY 有两个部分，一个主部分和一个从属部分。**PTS**代表*伪终端从属*，我们连接的就是这部分。主部分打开/创建伪终端，但我们使用的是从属部分。我们将在本章稍后深入探讨这个概念。

在*步骤 3*中我们使用的设置（`-parenb`和`cs8`）意味着`parenb`被禁用，因为它有一个减号，而`cs8`被启用。`parenb`选项将生成一个奇偶校验位，并期望在输入中返回一个。奇偶校验位在拨号连接和串行通信中被广泛使用。`cs8`选项将字符大小设置为 8 位。

`stty`程序可以用来查看和设置终端的属性。在下一个食谱中，我们将返回到`stty`来更改一些值。

只要我们是终端设备的所有者，我们就可以读写它，就像我们在食谱的最后一步中看到的那样。

## 另请参阅

在`man 1 tty`和`man 1 stty`中有很多有用的信息。

# 使用 stty 更改终端设置

在这个食谱中，我们将学习如何更改终端的设置（或属性）。在上一个食谱中，我们用`stty -a`列出了我们当前的设置。在这个食谱中，我们将改变其中一些设置，使用相同的`stty`程序。

了解如何更改终端设置将使您能够根据自己的喜好进行调整。

## 准备好

这个食谱没有特殊要求。

## 如何做…

在这里，我们将更改当前终端的一些设置：

1.  让我们首先关闭`whoami`，并得到一个答案。请注意，当您输入时，您看不到`whoami`命令：

```
$> stty -echo
$> *whoami* jake 
$> 
```

1.  要再次打开回显，我们再次输入相同的命令，但不带减号。请注意，当您输入时，您看不到`stty`命令：

```
$> *stty echo*
$> whoami
jake
```

1.  我们还可以更改特殊的键序列——例如，通常情况下，EOF 字符是*Ctrl* + *D*。如果需要，我们可以将其重新绑定为一个单点（`.`）：

```
$> stty eof .
```

1.  现在输入一个单点（`.`），您当前的终端将退出或注销。当您启动一个新终端或重新登录时，设置将恢复正常。

1.  为了保存设置以便以后重用，我们首先进行必要的更改——例如，将 EOF 设置为一个点。然后，我们使用`stty --save`。该选项将打印一长串十六进制数字——这些数字就是设置。因此，为了保存它们，我们可以将`stty --save`的输出重定向到一个文件中：

```
$> stty eof .
$> stty --save
5500:5:bf:8a3b:3:1c:7f:15:2e:0:1:0:11:13:1a:0:12:f:17:16:0:0:0:0:0:0:0:0:0:0:0:0:0:0:0:0
$> stty --save > my-tty-settings
```

1.  现在，按下一个点来注销。

1.  重新登录（或重新打开终端窗口）。尝试输入一个点，什么也不会发生。为了重新加载我们的设置，我们使用上一步的`my-tty-settings`文件。`$()`序列*展开*括号内的命令，然后用作`stty`的参数：

```
$> stty $(cat my-tty-settings)
```

1.  现在，我们可以再次尝试按下一个点来注销。

## 它是如何工作的…

终端通常是一个“愚蠢”的设备，因此需要大量的设置才能使其正常工作。这也是旧硬件电传打字机的遗留物之一。`stty`程序用于在终端设备上设置属性。

带有减号的选项被否定，即被禁用。没有减号的选项是启用的。在我们的例子中，我们首先关闭了回显，这是密码提示的常见做法，等等。

没有真正的方法可以保存 TTY 的设置，除了我们在这里看到的通过将其保存到文件并稍后重新读取它。

# 调查 TTY 和 PTY 并向它们写入

在这个食谱中，我们将学习如何列出当前登录的用户，他们使用的 TTY 以及他们正在运行的程序。我们还将学习如何向这些用户和终端写入。正如我们将在这个食谱中看到的，我们可以像写入文件一样向**终端设备**写入，假设我们有正确的权限。

知道如何写入其他终端会加深对终端工作原理和终端的理解。它还使您能够编写一些有趣的软件，并且最重要的是，它将使您成为一个更好的系统管理员。它还教会您有关终端安全的知识。

## 如何做…

我们将首先调查已登录用户；然后，我们将学习如何向他们发送消息：

1.  为了使事情变得更有趣，打开三到四个终端窗口。如果您没有使用**X-Window System**，请在多个 TTY 上登录。或者，如果您正在使用远程服务器，请多次登录。

1.  现在，在其中一个终端中键入`who`命令。您将获得所有已登录用户的列表，他们正在使用的 TTY/PTY，以及他们登录的日期和时间。在我的例子中，我通过 SSH 登录了多次。如果您正在使用具有多个`xterm`应用程序的本地计算机，则将看到`(:0)`而不是**Internet Protocol**（**IP**）地址：

```
$> who
root     tty1         Jan  5 16:03
jake     pts/0        Jan  5 16:04 (192.168.0.34)
jake     pts/1        Jan  5 16:04 (192.168.0.34)
jake     pts/2        Jan  5 16:04 (192.168.0.34)
```

1.  还有一个类似的命令`w`，甚至显示每个终端上的用户当前正在使用的程序：

```
$> w
 16:09:33 up 7 min,  4 users,  load average: 0.00, 0.16, 0.13
USER  TTY    FROM          LOGIN@  IDLE  JCPU   PCPU WHAT
root  tty1   -             16:03   6:05  0.07s  0.07s -bash
jake  pts/0  192.168.0.34  16:04   5:25  0.01s  0.01s -bash
jake  pts/1  192.168.0.34  16:04   0.00s 0.04s  0.01s w
jake  pts/2  192.168.0.34  16:04   5:02  0.02s  0.02s -bash
```

1.  让我们找出我们正在使用哪个终端：

```
$> tty
/dev/pts/1
```

1.  现在我们知道我们正在使用哪个终端，让我们向另一个用户和终端发送消息。在本书的开头，我提到一切都只是一个文件或一个进程。即使对于终端也是如此。这意味着我们可以使用常规重定向向终端发送数据：

```
$> echo "Hello" > /dev/pts/2
```

文本*Hello*现在将出现在 PTS2 终端上。

1.  仅当发送消息的用户与另一个终端上已登录的用户相同时，使用`echo`向终端发送消息才有效。例如，如果我尝试向 root 已登录的 TTY1 发送消息，它不起作用——有一个很好的原因：

```
$> echo "Hello" > /dev/tty1
-bash: /dev/tty1: Permission denied
```

1.  然而，存在一个允许用户向彼此终端写入的程序，假设他们已经允许。该程序称为`write`。要允许或禁止消息，我们使用`mesg`程序。如果您可以在终端上以 root（或其他用户）登录，请这样做，然后允许消息（字母`y`代表*yes*）：

```
#> tty
/dev/tty1
#> whoami
root
#> mesg y
```

1.  现在，从另一个用户，我们可以向该用户和终端写入：

```
$> write root /dev/tty1
Hello! How are you doing?
*Ctrl*+*D*
```

该消息现在将出现在 TTY1 上，其中 root 已登录。

1.  还有另一个命令允许用户在*所有*终端上写入。但是，root 是唯一可以向关闭消息的用户发送消息的用户。当以 root 身份登录时，请发出以下命令，向所有已登录用户写入有关即将重新启动的消息：

```
#> wall "The machine will be rebooted later tonight"
```

这将在所有用户的终端上显示一个消息，如下所示：

```
Broadcast message from root (tty1) (Tue Jan  5 16:59:33)
The machine will be rebooted later tonight
```

## 工作原理…

由于所有终端都由文件表示在文件系统上，因此向它们发送消息很容易。然而，常规权限也适用，以防止用户向其他用户写入或窥视其终端。

使用`write`程序，用户可以快速地向彼此写入消息，而无需任何第三方软件。

## 还有更多…

`wall`程序用于警告用户即将重新启动或关闭计算机。例如，如果 root 发出`shutdown -h +5`命令以安排在 5 分钟内关闭计算机，所有用户都将收到警告。使用`wall`程序会自动发送该警告。

## 另请参阅

有关本配方中涵盖的命令的更多信息，请参阅以下手册页面：

+   `man 1 write`

+   `man 1 wall`

+   `man 1 mesg`

# 检查它是否是 TTY

在这个配方中，我们将开始查看一些 C 函数来检查 TTY。在这里，我们指的是 TTY 的广义，即 TTY 和 PTY。

我们将在这里编写的程序将检查 stdout 是否是终端。如果不是，它将打印错误消息。

知道如何检查 stdin、stdout 或 stderr 是否是终端设备将使您能够为需要终端才能工作的程序编写错误检查。

## 准备工作

对于这个配方，我们需要 GCC 编译器，Make 工具和通用 Makefile。通用 Makefile 可以从本章的 GitHub 文件夹下载，网址为 https://github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch9。

## 如何做…

在这里，我们将编写一个小程序，如果 stdout 不是终端，则打印错误消息：

1.  在文件中编写以下小程序并将其保存为`ttyinfo.c`。我们在这里使用了两个新函数。第一个是`isatty()`，它检查一个`ttyname()`，它打印连接到 stdout（或实际上是路径）的终端的名称：

```
#include <stdio.h>
#include <unistd.h>
#include <errno.h>
int main(void)
{
    if ( (isatty(STDOUT_FILENO) == 1) )
    {
        printf("It's a TTY with the name %s\n",
            ttyname(STDOUT_FILENO));
    }
    else
    {
        perror("isatty");
    }
    printf("Hello world\n");
    return 0;
}
```

1.  编译程序：

```
$> make ttyinfo
gcc -Wall -Wextra -pedantic -std=c99    ttyinfo.c   -o ttyinfo
```

1.  让我们尝试一下这个程序。首先，我们不使用任何重定向来运行它。程序将打印终端的名称和文本*Hello world*：

```
$> ./ttyinfo 
It's a TTY with the name /dev/pts/10
Hello world
```

1.  但是，如果我们将文件描述符 1 重定向到文件，它就不再是终端（因为那个文件描述符指向文件而不是终端）。这将打印一个错误消息，但*Hello world*消息仍然被重定向到文件：

```
$> ./ttyinfo > my-file
isatty: Inappropriate ioctl for device
$> cat my-file 
Hello world
```

1.  为了证明这一点，我们可以将文件描述符 1“重定向”到`/dev/stdout`。然后一切将像往常一样工作，因为文件描述符 1 再次成为 stdout：

```
$> ./ttyinfo > /dev/stdout
It's a TTY with the name /dev/pts/10
Hello world
```

1.  另一个证明这一点的步骤是重定向到我们自己的终端设备。这将类似于我们在上一个配方中看到的，当我们使用`echo`将文本打印到终端时：

```
$> tty
/dev/pts/10
$> ./ttyinfo > /dev/pts/10 
It's a TTY with the name /dev/pts/10
Hello world
```

1.  为了进行实验，让我们打开第二个终端。使用`tty`命令找到新终端的 TTY 名称（在我的情况下是`/dev/pts/26`）。然后，从第一个终端再次运行`ttyinfo`程序，但将文件描述符 1（stdout）重定向到第二个终端：

```
$> ./ttyinfo > /dev/pts/26
```

在*当前*终端上不会显示任何输出。但是，在*第二*终端上，我们可以看到程序的输出，以及第二个终端的名称：

```
It's a TTY with the name /dev/pts/26
Hello world
```

## 工作原理...

我们使用`STDOUT_FILENO`宏，它与`isatty()`和`ttyname()`一起使用，只是整数 1-也就是文件描述符 1。

请记住，当我们用`>`符号重定向 stdout 时，我们重定向文件描述符 1。

通常，文件描述符 1 是 stdout，它连接到您的终端。如果我们使用`>`字符将文件描述符 1 重定向到文件，它将指向该文件。由于常规文件不是终端，我们会从程序（从`isatty()`函数的`errno`变量）得到一个错误消息。

当我们将文件描述符 1 重新重定向回`/dev/stdout`时，它再次成为 stdout，不会打印错误消息。

在最后一步中，当我们将程序的输出重定向到另一个终端时，所有文本都被重定向到该终端。不仅如此-程序打印的 TTY 名称确实是第二个终端的。原因是连接到文件描述符 1 的终端设备确实是那个终端（在我的情况下是`/dev/pts/26`）。

## 另请参阅

有关我们在配方中使用的函数的更多信息，我建议您阅读`man 3 isatty`和`man 3 ttyname`。

# 创建一个 PTY

在这个配方中，我们将创建一个`screen`并开始输入，字符将被打印到主设备和从设备上。从设备是`screen`程序连接的地方，在这种情况下是我们的终端。主设备通常是静默的并在后台运行，但为了演示目的，我们也会在主设备上打印字符。

了解如何创建 PTY 使您能够编写自己的终端应用程序，如`xterm`，Gnome 终端，`tmux`等。

## 准备工作

对于这个配方，您将需要 GCC 编译器，Make 工具和`screen`程序。有关`screen`的安装说明，请参阅本章的*技术要求*部分。

## 如何做...

在这里，我们将编写一个创建 PTY 的小程序。然后我们将使用`screen`连接到这个 PTY 的从端口-PTS。然后我们可以输入字符，它们会被打印回 PTS 上：

1.  我们将首先为这个配方编写程序。这里有很多新概念，所以代码被分成了几个步骤。将所有代码写在一个名为`my-pty.c`的单个文件中。我们将首先定义`_XOPEN_SOURCE`（用于`posix_openpt()`），并包括我们需要的所有头文件：

```
#define _XOPEN_SOURCE 600
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <string.h>
#include <unistd.h> 
```

1.  接下来，我们将开始`main()`函数并定义一些我们需要的变量：

```
int main(void)
{
   char rxbuf[1];
   char txbuf[3];
   int master; /* for the pts master fd */
   int c; /* to catch read's return value */ 
```

1.  现在，是时候使用`posix_openpt()`创建 PTY 设备了。这将返回一个文件描述符，我们将保存在`master`中。然后，我们运行`grantpt()`，它将把设备的所有者设置为当前用户，组设置为*tty*，并将设备的模式更改为`620`。在使用之前，我们还必须使用`unlockpt()`进行解锁。为了知道我们应该连接到哪里，我们还使用`ptsname()`打印从属设备的路径：

```
   master = posix_openpt(O_RDWR);
   grantpt(master);
   unlockpt(master);
   printf("Slave: %s\n", ptsname(master));
```

1.  接下来，我们创建程序的主循环。在循环中，我们从 PTS 中读取一个字符，然后再次将其写回 PTS。在这里，我们还将字符打印到主设备上，以便我们知道它是主/从设备对。由于终端设备相当原始，我们必须手动检查**回车**字符（*Enter*键），并且打印**换行**和回车以换行：

```
  while(1) /* main loop */
   {
      /* read from the master file descriptor */
      c = read(master, rxbuf, 1);
      if (c == 1)
      {
         /* convert carriage return to '\n\r' */
         if (rxbuf[0] == '\r')
         {
            printf("\n\r"); /* on master */
            sprintf(txbuf, "\n\r"); /* on slave */
         }
         else
         { 
            printf("%c", rxbuf[0]); 
            sprintf(txbuf, "%c", rxbuf[0]);
         }
         fflush(stdout);
         write(master, txbuf, strlen(txbuf));
      }
```

1.  如果没有收到任何字符，则连接到从属设备的设备已断开。如果是这种情况，我们将返回，因此退出程序：

```
      else /* if c is not 1, it has disconnected */
      {
         printf("Disconnected\n\r");
         return 0;
      } 
   }
   return 0;
}
```

1.  现在，是时候编译程序以便我们可以运行它了：

```
$> make my-pty
gcc -Wall -Wextra -pedantic -std=c99    my-pty.c   -o my-pty
```

1.  现在，在当前终端中运行程序并记下从主设备获得的从属路径：

```
$> ./my-pty
Slave: /dev/pts/31
```

1.  在继续连接之前，让我们检查一下设备。在这里，我们将看到我的用户拥有它，它确实是一个*字符特殊*设备，对终端来说很常见：

```
$> ls -l /dev/pts/31
crw--w---- 1 jake tty 136, 31 jan  3 20:32 /dev/pts/31
$> file /dev/pts/31
/dev/pts/31: character special (136/31)
```

1.  现在，打开一个新的终端并连接到您从主设备获得的从属路径。在我的情况下，它是`/dev/pts/31`。要连接到它，我们将使用`screen`：

```
$> screen /dev/pts/31
```

1.  现在，我们可以随意输入，所有字符都将被打印回给我们。它们也将出现在主设备上。要断开并退出`screen`，首先按下*Ctrl* + *A*，然后输入一个单独的*K*，如 kill。然后会出现一个问题（*真的要杀死这个窗口吗[y/n]*）；在这里输入*Y*。现在您将在启动`my-pty`的终端中看到*已断开*，程序将退出。

## 它是如何工作的...

我们使用`posix_openpt()`函数打开一个新的 PTY。我们使用`O_RDWR`设置为读和写。通过打开一个新的 PTY，在`/dev/pts/`中创建了一个新的字符设备。这就是我们后来使用`screen`连接的字符设备。

由于`posix_openpt()`返回一个文件描述符，我们可以使用所有常规的文件描述符系统调用来读取和写入数据，比如`read`和`write`。

终端设备，比如我们在这里创建的设备，相当原始。如果我们按下*Enter*，光标将返回到行的开头。首先不会创建新行。这实际上是*Enter*键以前的工作方式。为了解决这个问题，我们在程序中检查读取的字符是否是回车（*Enter*键发送的内容），如果是，我们将首先打印一个换行字符，然后是一个回车。

如果我们只打印换行符，我们只会得到一个新行，就在当前光标下面。这种行为是从旧式电传打字机设备留下的。在打印当前字符（或换行和回车）后，我们使用`fflush()`。原因是在主端打印的字符（`my-pty`程序运行的地方）后面没有新行。Stdout 是行缓冲的，这意味着它只在换行时刷新。但是由于我们希望在输入每个字符时都能看到它，我们必须在每个字符上刷新它，使用`fflush()`。

## 另请参阅

手册页面中有很多有用的信息。我特别建议您阅读以下手册页面：`man 3 posix_openpt`，`man 3 grantpt`，`man 3 unlockpt`，`man 4 pts`和`man 4 tty`。

# 禁用密码提示的回显

为了防止用户的密码被肩窥，最好隐藏他们输入的内容。隐藏密码不被显示的方法是禁用**回显**。在这个示例中，我们将编写一个简单的密码程序，其中禁用了回显。

在编写需要某种秘密输入的程序（如密码或密钥）时，了解如何禁用回显是关键。

## 准备工作

对于这个示例，你需要 GCC 编译器、Make 工具和通用的 Makefile。

## 如何做...

在这个示例中，我们将构建一个带有密码提示的小程序

1.  由于本示例中的代码将会相当长，有些部分有点晦涩，我已经将代码分成了几个步骤。但请注意，所有的代码都应该放在一个文件中。将文件命名为`passprompt.c`。让我们从`include`行、`main()`函数和我们需要的变量开始。名为`term`的`termios`类型的结构是一个特殊的结构，它保存了终端的属性：

```
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <termios.h>
int main(void)
{
    char mypass[] = "super-secret";
    char buffer[80];
    struct termios term;
```

1.  接下来，我们将首先禁用回显，但首先需要使用`tcgetattr()`获取终端的所有当前设置。一旦我们获得了所有设置，我们就修改它们以禁用回显。我们这样做的方式是使用`ECHO`。`~`符号否定一个值。稍后在*它是如何工作...*部分会详细介绍：

```
    /* get the current settings */
    tcgetattr(STDIN_FILENO, &term);
    /* disable echoing */
    term.c_lflag = term.c_lflag & ~ECHO;
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &term);
```

1.  然后，我们编写密码提示的代码；这里没有什么新鲜的，我们已经知道了：

```
    printf("Enter password: ");
    scanf("%s", buffer);
    if ( (strcmp(mypass, buffer) == 0) )
    {
        printf("\nCorrect password, welcome!\n");
    }
    else
    {
        printf("\nIncorrect password, go away!\n");
    }    
```

1.  然后，在退出程序之前，我们必须再次打开回显；否则，即使程序退出后，回显也将保持关闭。这样做的方法是`ECHO`。这将撤销我们之前所做的事情：

```
    /* re-enable echoing */
    term.c_lflag = term.c_lflag | ECHO;
    tcsetattr(STDIN_FILENO, TCSAFLUSH, &term);
    return 0;
}
```

1.  现在，让我们编译程序：

```
$> make passprompt
gcc -Wall -Wextra -pedantic -std=c99    passprompt.c   -o passprompt
```

1.  现在，我们可以尝试这个程序，我们会注意到我们看不到自己输入的内容：

```
$> ./passprompt 
Enter password: *test*+*Enter*
Incorrect password, go away!
$> ./passprompt 
Enter password: *super-secret*+*Enter*
Correct password, welcome!
```

## 它是如何工作的...

使用`tcsetattr()`对终端进行更改的方法是使用`tcgetattr()`获取当前属性，然后修改它们，最后将这些更改后的属性应用到终端上。

`tcgetattr()`和`tcsetattr()`的第一个参数都是我们要更改的文件描述符。在我们的情况下，是 stdin。

`tcgetattr()`的第二个参数是属性将被保存的结构。

`tcsetattr()`的第二个参数确定更改何时生效。在这里，我们使用`TCSAFLUSH`，这意味着更改发生在所有输出被写入后，所有接收但未读取的输入将被丢弃。

`tcsetattr()`的第三个参数是包含属性的结构。

为了保存和设置属性，我们需要一个名为`termios`的结构（与我们使用的头文件同名）。该结构包含五个成员，其中四个是模式。这些是输入模式（`c_iflag`）、输出模式（`c_oflag`）、控制模式（`c_cflag`）和本地模式（`c_lflag`）。我们在这里改变的是本地模式。

首先，我们在`c_lflag`成员中有当前的属性，它是一个无符号整数，由一堆位组成。这些位就是属性。

然后，要关闭一个设置，例如，在我们的情况下关闭回显，我们对`ECHO`宏进行否定（"反转"它），然后使用按位与（`&`符号）将其添加回`c_lflag`。

`ECHO`宏是`010`（八进制 10），或者十进制 8，二进制中是`00001000`（8 位）。取反后是`11110111`。然后对这些位与原始设置的其他位进行按位与操作。

按位与操作的结果然后应用到终端上，使用`tcsetattr()`关闭回显。

在结束程序之前，我们通过对新值进行按位或操作来逆转这个过程，然后使用`tcsetattr()`应用该值，再次打开回显。

## 还有更多...

我们可以通过这种方式设置很多属性，例如，可以禁用中断和退出信号的刷新等。`man 3 tcsetattr()`手册页中列出了每种模式使用的宏的完整列表。

# 读取终端大小

在这个示例中，我们将继续深入研究我们的终端。在这里，我们编写一个有趣的小程序，实时报告终端的大小。当你调整终端窗口的大小时（假设你正在使用 X 控制台应用程序），你会立即看到新的大小被报告。

为了使这个工作，我们将使用一个特殊的`ioctl()`函数。

了解如何使用这两个工具、转义序列和`ioctl()`将使您能够在终端上做一些有趣的事情。

## 准备工作

为了充分利用这个配方，最好使用`xterm`，`rxvt`，*Konsole*，*Gnome Terminal*等。

您还需要 GCC 编译器，Make 工具和通用 Makefile。

## 如何做…

在这里，我们将编写一个程序，首先使用特殊的转义序列清除屏幕，然后获取终端的大小并打印到屏幕上：

1.  在文件中写入以下代码并将其保存为`terminal-size.c`。程序使用一个无限循环，因此要退出程序，我们必须使用*Ctrl* + *C*。在循环的每次迭代中，我们首先通过打印特殊的*转义序列*来清除屏幕。然后，我们使用`ioctl()`获取终端大小并在屏幕上打印大小：

```
#include <stdio.h>
#include <unistd.h>
#include <termios.h>
#include <sys/ioctl.h>
int main(void)
{
   struct winsize termsize;
   while(1)
   {
      printf("\033[1;1H\033[2J");
      ioctl(STDOUT_FILENO, TIOCGWINSZ, &termsize);
      printf("Height: %d rows\n", 
         termsize.ws_row);
      printf("Width: %d columns\n", 
         termsize.ws_col);
      sleep(0.1);
   }
   return 0;
} 
```

1.  编译程序：

```
$> make terminal-size
gcc -Wall -Wextra -pedantic -std=c99    terminal-size.c   -o terminal-size
```

1.  现在，在终端窗口中运行程序。当程序正在运行时，调整窗口大小。您会注意到大小会立即更新。使用*Ctrl* + *C*退出程序：

```
$> ./terminal-size
Height: 20 rows
Width: 97 columns
*Ctrl*+*C*
```

## 它是如何工作的…

首先，我们定义一个名为`termsize`的结构，类型为`winsize`。我们将在这个结构中保存终端大小。该结构有两个成员（实际上有四个，但只使用了两个）。成员是`ws_row`表示行数和`wc_col`表示列数。

然后，为了清除屏幕，我们使用`printf()`打印一个特殊的转义序列，`\033[1;1H\033[2J`。`\033`序列是转义码。在转义码之后，我们有一个`[`字符，然后我们有实际的代码告诉终端要做什么。第一个`1;1H`将光标移动到位置 1,1（第一行和第一列）。然后，我们再次使用`\033`转义码，以便我们可以使用另一个代码。首先，我们有`[`字符，就像以前一样。然后，我们有`[2J`代码，这意味着擦除整个显示。

一旦我们清除了屏幕并移动了光标，我们使用`ioctl()`来获取终端大小。第一个参数是文件描述符；在这里，我们使用 stdout。第二个参数是要发送的命令；在这里，它是`TIOCGWINSZ`以获取终端大小。这些宏/命令可以在`man 2 ioctl_tty`手册页中找到。第三个参数是`winsize`结构。

一旦我们在`winsize`结构中有了尺寸，我们就使用`printf()`打印值。

为了避免耗尽系统资源，我们在下一次迭代之前睡眠 0.1 秒。

## 还有更多…

在`man 4 console_codes`手册页中，有许多其他代码可以使用。您可以做任何事情，从使用颜色到粗体字体，到移动光标，到响铃终端等等。

例如，要以闪烁的品红色打印*Hello*，然后重置为默认值，您可以使用以下命令：

```
printf("\033[35;5mHello!\033[0m\n");
```

但请注意，并非所有终端都能闪烁。

## 另请参阅

有关`ioctl()`的更多信息，请参阅`man 2 ioctl`和`man 2 ioctl_tty`手册页。后者包含有关`winsize`结构和宏/命令的信息。
