# 前言

Linux 系统编程就是为 Linux 操作系统开发系统程序。Linux 是世界上最流行的开源操作系统，可以运行在从大型服务器到小型物联网设备的各种设备上。了解如何为 Linux 编写系统程序将使您能够扩展操作系统并将其与其他程序和系统连接起来。

我们将从学习如何使我们的程序易于脚本化和易于与其他程序交互开始。当我们为 Linux 编写系统程序时，我们应该始终努力使它们小巧并且只做一件事，并且做得很好。这是 Linux 中的一个关键概念：创建可以以简单方式相互交换数据的小程序。

随着我们的进展，我们将深入研究 C 语言，了解编译器的工作原理，链接器的功能，如何编写 Makefile 等等。

然后，我们将学习关于 forking 和守护进程的所有知识。我们还将创建自己的守护进程，并将其置于 systemd 的控制之下。这将使我们能够使用内置的 Linux 工具启动、停止和重新启动守护进程。

我们还将学习如何使用不同类型的进程间通信（IPC）使我们的进程交换信息。我们还将学习如何编写多线程程序。

在本书的结尾，我们将介绍如何使用 GNU 调试器（GDB）和 Valgrind 调试我们的程序。

在本书结束时，您将能够为 Linux 编写各种系统程序，从过滤器到守护程序。

# 本书适合对象

本书适用于任何希望为 Linux 开发系统程序并深入了解 Linux 系统的人。任何面临与 Linux 系统编程特定部分相关的问题并寻找特定的解决方案的人都可以从本书中受益。

# 本书内容

[*第一章*]《获取必要的工具并编写我们的第一个 Linux 程序》向您展示了如何安装本书中需要的工具。本章还介绍了我们的第一个程序。

[*第二章*]《使您的程序易于脚本化》介绍了我们应该如何以及为什么要使我们的程序易于脚本化，并且易于其他系统上的程序使用。

[*第三章*]《深入了解 Linux 中的 C 语言》带领我们深入了解 Linux 中 C 编程的内部工作原理。我们将学习如何使用系统调用，编译器的工作原理，使用 Make 工具，指定不同的 C 标准等。

[*第四章*]《处理程序中的错误》教会我们如何优雅地处理错误。

[*第五章*]《使用文件 I/O 和文件系统操作》介绍了如何使用文件描述符和流读写文件。本章还介绍了如何创建和删除文件以及使用系统调用读取文件权限。

[*第六章*]《生成进程并使用作业控制》介绍了 forking 的工作原理，如何创建守护进程，父进程是什么，以及如何将作业发送到后台和前台。

[*第七章*]《使用 systemd 处理您的守护程序》向我们展示了如何将上一章中的守护程序置于 systemd 的控制之下。本章还教会我们如何将日志写入 systemd 的日志记录中，并且如何读取这些日志。

[*第八章*]《创建共享库》教会我们什么是共享库，它们为何重要，以及如何制作我们自己的共享库。

*第九章*，*终端 I/O 和更改终端行为*，介绍了如何以不同方式修改终端，例如如何禁用密码提示的回显。

*第十章*，*使用不同类型的 IPC*，介绍了 IPC，即如何使进程在系统上相互通信。本章涵盖了 FIFO、Unix 套接字、消息队列、管道和共享内存。

*第十一章*，*在程序中使用线程*，解释了线程是什么，如何编写多线程程序，如何避免竞争条件，以及如何优化多线程程序。

*第十二章*，*调试您的程序*，介绍了使用 GDB 和 Valgrind 进行调试。

# 充分利用本书

要充分利用本书，您需要对 Linux 有基本的了解，熟悉一些基本命令，熟悉在文件系统中移动和安装新程序。最好还具备对编程的基本了解，最好是 C 语言。

您需要一台具有 root 访问权限的 Linux 计算机，可以通过 su 或 sudo 完成所有的操作。您还需要安装 GCC 编译器、Make 工具、GDB、Valgrind 和其他一些较小的工具。具体的 Linux 发行版并不那么重要。本书中有这些程序的 Debian、Ubuntu、CentOS、Fedora 和 Red Hat 的安装说明。

**如果您使用本书的数字版本，我们建议您自己输入代码或通过 GitHub 存储库访问代码（链接在下一节中提供）。这样做将有助于避免与复制和粘贴代码相关的潜在错误。**

## 下载示例代码文件

您可以从 GitHub 上下载本书的示例代码文件，链接为[`github.com/PacktPublishing/Linux-System-Programming-Techniques`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 代码演示

本书的代码演示视频可在 https://bit.ly/39ovGd6 上观看。

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：`www.packtpub.com/sites/default/files/downloads/9781789951288_ColorImages.pdf`。

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码词、目录、文件名、文件扩展名、路径名、虚拟 URL、用户输入等。以下是一个示例：“将`libprime.so.1`文件复制到`/usr/local/lib`。”

代码块设置如下：

```
#include <stdio.h>
```

```
int main(void)
```

```
{
```

```
    printf("Hello, world!\n");
```

```
    return 0;
```

```
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```
#include <stdio.h>
```

```
int main(void)
```

```
{
```

```
    printf("Hello, world!\n");
```

```
    return 0;
```

```
}
```

任何命令行输入或输出都以如下形式书写：

```
$> mkdir cube
```

```
$> cd cube
```

在编号列表中，命令行输入以粗体显示。`$>`字符表示提示符，不是您应该写的内容。

1.  这是一个带编号的列表示例：

```
\ character. This is the same character as you use to break long lines in the Linux shell. The line under it has a > character to indicate that the line is a continuation of the previous line. The > character is *not* something you should write; the Linux shell will automatically put this character on a new line where the last line was broken up with a \ character. For example:

```

$> ./exist.sh /asdf &> /dev/null; \

> 如果[ $? -eq 3 ]; then echo "不存在"; fi

不存在

```

Key combinations are written in italics. Here is an example: "Press *Ctrl* + *C* to exit the program."`customercare@packtpub.com`.`copyright@packt.com` with a link to the material.**If you are interested in becoming an author**: If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, please visit [authors.packtpub.com](http://authors.packtpub.com).ReviewsPlease leave a review. Once you have read and used this book, why not leave a review on the site that you purchased it from? Potential readers can then see and use your unbiased opinion to make purchase decisions, we at Packt can understand what you think about our products, and our authors can see your feedback on their book. Thank you!For more information about Packt, please visit [packt.com](http://packt.com).
```
