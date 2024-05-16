# 第一章：获取必要的工具并编写我们的第一个 Linux 程序

在本章中，我们将在我们的 Linux 系统上安装必要的工具，如 GCC，GNU Make，GDB 和 Valgrind。我们还将尝试它们，并看看它们是如何工作的。知道如何使用这些工具是成为快速高效的开发人员的关键。然后，我们将编写我们的第一个程序——Linux 风格。通过了解 C 程序的不同部分，您可以以最佳实践方式轻松地与系统的其余部分进行交互。之后，我们将学习如何使用内置手册页（简称 man 页）查找命令、库和系统调用——这是我们在整本书中需要大量使用的技能。知道如何在相关的内置手册页中查找信息通常比在互联网上搜索答案更快、更精确。

在本章中，我们将涵盖以下内容：

+   安装 GCC 和 GNU Make

+   安装 GDB 和 Valgrind

+   为 Linux 编写一个简单的 C 程序

+   编写一个解析命令行选项的程序

+   在内置手册页中查找信息

+   搜索手册以获取信息

让我们开始吧！

# 技术要求

本章，您需要一台已经设置好 Linux 的计算机。无论是本地机器还是远程机器都没关系。您使用的特定发行版也不太重要。我们将看看如何在基于 Debian 的发行版以及基于 Fedora 的发行版中安装必要的程序。大多数主要的 Linux 发行版要么是基于 Debian 的，要么是基于 Fedora 的。

您还将使用`vi`和`nano`，它们几乎在任何地方都可以使用。不过在本书中我们不会介绍如何使用文本编辑器。

本章的 C 文件可以从[`github.com/PacktPublishing/Linux-System-Programming-Techniques`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques)的`ch1`目录中下载。GitHub 上的文件名对应本书中的文件名。

您还可以将整个存储库克隆到您的计算机上。本章的文件位于`ch1`目录中。您可以使用以下命令克隆存储库：

```
$> git clone https://github.com/PacktPublishing/Linux-System-Programming-Techniques.git
```

如果您的计算机上没有安装 Git，您将需要根据您的发行版遵循一些安装说明。

查看以下链接以查看“代码实战”视频：[`bit.ly/3wdEoV6`](https://bit.ly/3wdEoV6)

## 安装 Git 以下载代码存储库

只有在您想要克隆（下载）整个代码存储库到您的计算机上时，才需要安装 Git。这里列出的步骤假定您的用户具有`sudo`权限。如果不是这种情况，您可以首先运行`su`切换到 root 用户，然后跳过`sudo`（假设您知道 root 密码）。

### 基于 Debian 的发行版

这些说明适用于大多数基于 Debian 的发行版，如 Ubuntu：

1.  首先，更新存储库缓存：

```
$> sudo apt update
```

1.  然后，使用`apt`安装 Git：

```
$> sudo apt install git
```

### 基于 Fedora 的发行版

这些说明适用于所有较新的基于 Fedora 的发行版，如 CentOS 和 Red Hat（如果您使用的是旧版本，您可能需要用`yum`替换`dnf`）：

+   使用`dnf`安装 Git 包：

```
$> sudo dnf install git
```

# 安装 GCC 和 GNU Make

在本节中，我们将安装本书中将需要的基本工具；即，编译器 GCC。它是将 C 源代码转换为可以在系统上运行的二进制程序的**编译器**。我们编写的所有 C 代码都需要编译。

我们还将安装 GNU Make，这是一个我们以后将用来自动化包含多个源文件的项目编译的工具。

## 做好准备

由于我们要在系统上安装软件，我们需要使用`sudo`权限。在本教程中，我将使用`sudo`，但如果您在没有`sudo`的系统上，您可以在输入命令之前切换到 root 用户使用`su`（然后省略`sudo`）。

## 如何做…

我们将安装一个称为元包或组的软件包，该软件包包含其他软件包的集合。这个元包包括 GCC、GNU Make、几个手册页面和其他在开发时很有用的程序和库。

### 基于 Debian 的系统

这些步骤适用于所有基于 Debian 的系统，如 Debian、**Ubuntu**和**Linux Mint**：

1.  更新存储库缓存，以在下一步中获取最新版本：

```
$> sudo apt-get update
```

1.  安装`build-essential`软件包，并在提示时回答`y`：

```
$> sudo apt-get install build-essential
```

### 基于 Fedora 的系统

这对所有基于 Fedora 的系统都适用，如 Fedora、**CentOS**和**Red Hat**：

+   安装一个名为*Development Tools*的软件组：

```
$> sudo dnf group install 'Development Tools'
```

### 验证安装（Debian 和 Fedora）

这些步骤对 Debian 和 Fedora 都适用：

1.  通过列出安装的版本来验证安装。请注意，确切的版本会因系统而异；这是正常的：

```
$> gcc --version
gcc (Debian 8.3.0-6) 8.3.0
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
$> make --version
GNU Make 4.2.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later http://gnu.org/licenses/gpl.html
This is free software: you are free to change and redistribute it. There is NO WARRANTY, to the extent permitted by law.
```

1.  现在，是时候尝试使用 GCC 编译器编译一个最小的 C 程序了。请将源代码输入编辑器并保存为`first-example.c`。该程序将在终端上打印文本“Hello, world!”：

```
#include <stdio.h>
int main(void)
{
    printf("Hello, world!\n");
    return 0;
}
```

1.  现在，使用 GCC 编译它。这个命令会产生一个名为`a.out`的文件：

```
$> gcc first-example.c
```

1.  现在，让我们尝试运行程序。在 Linux 中运行不在通常的二进制目录（`/bin`、`/sbin`、`/usr/bin`等）中的程序时，您需要在文件名前输入特殊的`./`序列。这会从当前路径执行程序：

```
$> ./a.out
Hello, world!
```

1.  现在，重新编译程序。这次，我们将使用`-o`选项（`-o`代表*output*）指定程序的名称。这次，程序文件将被命名为`first-example`：

```
$> gcc first-example.c -o first-example
```

1.  让我们重新运行程序，这次使用新名称`first-example`：

```
$> ./first-example
Hello world!
```

1.  现在，让我们尝试使用 Make 来编译它：

```
$> rm first-example
$> make first-example
cc     first-example.c   -o first-example
```

1.  最后，重新运行程序：

```
$> ./first-example
Hello, world!
```

## 工作原理…

在系统上安装软件始终需要 root 权限，可以通过常规 root 用户或`sudo`来获取。例如，Ubuntu 使用`sudo`，并禁用了常规 root 用户。另一方面，Debian 在默认安装中根本不使用`sudo`。要使用它，您必须自己设置。

Debian 和 Ubuntu 在安装软件包之前使用`apt-get update`命令。

基于 Fedora 的系统在较新版本上使用`dnf`。如果您使用的是旧版本，可能需要用`yum`替换`dnf`。

在这两种情况下，我们安装了一组包，其中包含我们在本书中将需要的实用程序、手册页面和编译器。

安装完成后，在尝试编译任何内容之前，我们列出了 GCC 版本和 Make 版本。

最后，我们编译了一个简单的 C 程序，首先直接使用 GCC，然后使用 Make。使用 GCC 的第一个示例生成了一个名为`a.out`的程序，它代表*assembler output*。这个名字有着悠久的历史，可以追溯到 1971 年 Unix 的第一版。尽管文件格式`a.out`不再使用，但这个名字今天仍然存在。

然后，我们使用`-o`选项指定了程序名称，其中`-o`代表*output*。这将生成一个我们选择的程序名称。我们给程序取名为`first-example`。

使用 Make 时，我们不需要输入源代码的文件名。我们只需写出编译器生成的二进制程序的名称。Make 程序足够聪明，可以推断出源代码具有相同的文件名，但以`.c`结尾。

当我们执行程序时，我们将其作为`./first-example`运行。`./`序列告诉 shell 我们要从当前目录运行程序。如果我们省略`./`，它将不会使用`$PATH`变量——通常是`/bin`、`/usr/bin`、`/sbin`和`/usr/sbin`。

# 安装 GDB 和 Valgrind

GDB 和 Valgrind 是两个有用的**调试**工具，我们将在本书中稍后使用。

GDB 是 GNU 调试器，这是一个我们可以用来逐步执行程序并查看其内部发生情况的工具。我们可以监视变量，查看它们在运行时如何变化，设置我们希望程序暂停的断点，甚至更改变量。错误是不可避免的，但是有了 GDB，我们可以找到这些错误。

Valgrind 也是一个我们可以用来查找错误的工具，尽管它是专门用于查找内存泄漏的。没有像 Valgrind 这样的程序，内存泄漏可能很难找到。您的程序可能在几周内按预期工作，但突然之间，事情可能开始出错。这可能是内存泄漏。

知道如何使用这些工具将使您成为更好的开发人员，使您的程序更加安全。

## 准备工作

由于我们将在这里安装软件，所以我们需要以 root 权限执行这些命令。如果我们的系统有传统的 root 用户，我们可以通过切换到 root 用户来使用`su`。如果我们在一个具有`sudo`的系统上，并且我们的常规用户具有管理权限，您可以使用`sudo`来执行这些命令。在这里，我将使用`sudo`。

## 如何做…

如果您使用的是 Debian 或 Ubuntu，您将需要使用`apt-get`工具。另一方面，如果您使用的是基于 Fedora 的发行版，您将需要使用`dnf`工具。

### 基于 Debian 的系统

这些步骤适用于 Debian、Ubuntu 和 Linux Mint：

1.  在安装软件之前更新存储库缓存：

```
$> sudo apt-get update
```

1.  使用`apt-get`安装 GDB 和 Valgrind。在提示时回答`y`：

```
$> sudo apt-get install gdb valgrind
```

### 基于 Fedora 的系统

这一步适用于所有基于 Fedora 的系统，如 CentOS 和 Red Hat。如果您使用的是较旧的系统，您可能需要用`yum`替换`dnf`：

+   使用`dnf`安装 GDB 和 Valgrind。在提示时回答`y`：

```
$> sudo dnf install gdb valgrind
```

### 验证安装

这一步对于基于 Debian 和基于 Fedora 的系统是相同的：

+   验证 GDB 和 Valgrind 的安装：

```
$> gdb --version
GNU gdb (Debian 8.2.1-2+b3) 8.2.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later http://gnu.org/licenses/gpl.html
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
$> valgrind --version
valgrind-3.14.0
```

## 它是如何工作的…

GDB 和 Valgrind 是两个调试工具，它们不包括在我们在上一个教程中安装的组包中。这就是为什么我们需要将它们作为单独的步骤安装。在基于 Debian 的发行版上安装软件的工具是`apt-get`，而在 Fedora 上是`dnf`。由于我们正在系统上安装软件，所以我们需要以 root 权限执行这些命令。这就是为什么我们需要使用`sudo`。请记住，如果您的用户或系统不使用`sudo`，您可以使用`su`来成为 root。

最后，我们通过列出安装的版本来验证了安装。确切的版本可能因系统而异。

版本不同的原因是每个 Linux 发行版都有自己的软件仓库，并且每个 Linux 发行版都维护自己的软件版本作为“最新版本”。这意味着特定 Linux 发行版中程序的最新版本不一定是该程序的最新版本。

# 为 Linux 编写一个简单的 C 程序

在这个教程中，我们将构建一个小的 C 程序，该程序将对作为参数传递给程序的值进行求和。C 程序将包含一些我们在 Linux 编程时需要了解的基本元素。这些元素包括返回值、参数和帮助文本。随着我们在这本书中的进展，这些元素将一次又一次地出现，还会有一些我们在学习过程中会了解到的新元素。

掌握这些元素是为 Linux 编写优秀软件的第一步。

## 准备工作

这个教程唯一需要的是 C 源代码`sum.c`和 GCC 编译器。您可以选择自己输入代码，也可以从 GitHub 上下载。自己输入代码可以让您学会如何编写代码。

## 如何做…

按照以下步骤在 Linux 中编写您的第一个程序：

1.  打开一个文本编辑器，输入以下代码，将文件命名为`sum.c`。该程序将对输入到程序中的所有数字进行求和。程序的参数包含在`argv`数组中。要将参数转换为整数，我们可以使用`atoi()`函数：

```
#include <stdio.h>
#include <stdlib.h>
void printhelp(char progname[]);
int main(int argc, char *argv[])
{
    int i;
    int sum = 0;
    /* Simple sanity check */
    if (argc == 1)
    {
        printhelp(argv[0]);
        return 1;
    }
    for (i=1; i<argc; i++)
    {
        sum = sum + atoi(argv[i]);
    }
    printf("Total sum: %i\n", sum);
    return 0;
}
void printhelp(char progname[])
{
    printf("%s integer ...\n", progname);
    printf("This program takes any number of " 
        "integer values and sums them up\n");
}
```

1.  现在，是时候使用 GCC 编译源代码了：

```
$> gcc sum.c -o sum
```

1.  运行程序。不要忘记在文件名前加上`./`：

```
$> ./sum
./sum integer …
This program takes any number of integer values and sums them up
```

1.  现在，让我们在做其他事情之前检查程序的**退出码**：

```
$> echo $?
1
```

1.  让我们重新运行程序，这次使用一些整数，让程序为我们求和：

```
$> ./sum 45 55 12
Total sum: 112
```

1.  再次检查程序的退出码：

```
$> echo $?
0
```

## 它是如何工作的...

让我们从探索代码的基础知识开始，以便我们理解不同部分的作用以及它们为什么重要。

### 源代码

首先，我们包含了一个`stdio.h`。这个文件是`printf()`所需要的。*stdio*代表`printf()`在屏幕上打印字符，它被归类为*stdio*函数。

我们包含的另一个头文件是`stdlib.h`，它代表`atoi()`函数，我们可以用它将**字符串**或**字符**转换为整数。

在此之后，我们有一个`printhelp()`。关于这个函数没有什么特别要说的；在 C 语言中，将`main()`和函数原型放在最开始是一个很好的实践。函数原型告诉程序的其余部分函数需要什么参数，以及它返回什么类型的值。

然后，我们声明了`main()` `int main(int argc, char *argv[])`。

两个变量`argc`和`argv`有特殊的含义。第一个`argc`是一个整数，包含了传递给程序的参数数量。它至少为 1，即使没有参数传递给程序；第一个参数是程序本身的名称。

下一个变量是`argv`，它包含了传递给程序的所有参数，`argv[0]`包含了程序的名称，也就是执行程序时的命令行。如果程序被执行为`./sum`，那么`argv[0]`包含字符串`./sum`。如果程序被执行为`/home/jack/sum`，那么`argv[0]`包含字符串`/home/jack/sum`。

正是这个参数，或者说是程序名称，我们将其传递给`printhelp()`函数，以便它打印程序的名称和帮助文本。在 Linux 和 Unix 环境中这样做是一个很好的实践。

在那之后，我们执行了一个我们构建的简单的`printhelp()`函数。紧接着，我们用代码 1 从`main()`中`return`，告诉`main()`使用`return`，将代码发送到 shell 并退出程序。这些代码有特殊的含义，我们将在本书的后面更深入地探讨。简单来说，0 表示一切正常，而非 0 表示错误代码。在 Linux 中使用返回值是必须的；这是其他程序和 shell 得知执行情况的方式。

稍微往下一点，我们有`for()` `argc`来遍历参数列表。我们从`i=1`开始。我们不能从 0 开始，因为`argv[]`数组中的索引 0 是程序名称。索引 1 是第一个参数；也就是我们可以传递给程序的整数。

在`for()`循环中，我们有`sum = sum + atoi(argv[i]);`。这里我们要重点关注的是`atoi(argv[i])`。我们通过命令行传递给程序的所有参数都被作为字符串传递。为了能够对它们进行计算，我们需要将它们转换为整数，而`atoi()`函数就是为我们做这个工作的。`atoi()`的名称代表*to integer*。

一旦结果被用`printf()`打印到屏幕上，我们用 0 从`main()`中`return`，表示一切正常。当我们从`main()`返回时，我们从整个进程返回到 shell；换句话说，是**父进程**。

### 执行和返回值

当我们在`$PATH`环境变量中未提及的目录中执行程序时，需要在文件名前加上`./`。

当程序结束时，它将返回值传递给 shell，shell 将其保存到一个名为`?`的变量中。当另一个程序结束时，该变量将被该程序的最新返回值覆盖。我们打印`echo`的值，这是一个从 shell 直接在屏幕上打印文本和变量的小型实用程序。要打印环境变量，我们需要在变量名前加上`$`符号，例如`$?`。

## 还有更多…

还有另外三个类似于`atoi()`的函数，分别是`atol()`、`atoll()`和`atof()`。以下是它们的简短描述：

+   `atoi()`将一个字符串转换为整型。

+   `atol()`将一个字符串转换为长整型。

+   `atoll()`将一个字符串转换为长长整型。

+   `atof()`将一个字符串转换为浮点数（双精度类型）。

如果你想探索其他程序的返回值，你可以执行诸如`ls`之类的程序，指定一个存在的目录，并用`echo $?`打印变量。然后，你可以尝试列出一个不存在的目录，并再次打印`$?`的值。

提示

在本章中，我们已经多次提到了`$PATH`环境变量。如果你想知道该变量包含什么，你可以用`echo $PATH`打印它。如果你想临时向`$PATH`变量添加一个新目录，比如`/home/jack/bin`，你可以执行`PATH=${PATH}:/home/jack/bin`命令。

# 编写解析命令行选项的程序

在这个示例中，我们将创建一个更高级的程序——一个解析命令行`argc`和`argv`的程序。我们也会在这里使用这些变量，但是用于选项。选项是带有连字符的字母，比如`-a`或`-v`。

这个程序与上一个程序类似，不同之处在于这个程序可以同时进行加法和乘法运算；`-s`代表“求和”，`-m`代表“乘法”。

在 Linux 中，几乎所有的程序都有不同的选项。了解如何解析程序的选项是必须的；这是用户改变程序行为的方式。

## 准备工作

你需要的只是一个文本编辑器、GCC 编译器和 Make。

## 操作步骤…

由于这个源代码会比较长，它将被分成三部分。但整个代码都放在同一个文件中。完整的程序可以从 GitHub 上下载：[`github.com/PacktPublishing/Linux-System-Programming-Techniques/blob/master/ch1/new-sum.c`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/blob/master/ch1/new-sum.c)。让我们开始吧：

1.  打开一个文本编辑器，输入以下代码，并将其命名为`new-sum.c`。这一部分与上一个示例非常相似，只是有一些额外的变量和顶部的**宏**：

```
#define _XOPEN_SOURCE 500
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
void printhelp(char progname[]);
int main(int argc, char *argv[])
{
    int i, opt, sum;
    /* Simple sanity check */
    if (argc == 1)
    {
        printhelp(argv[0]);
        return 1;
    }
```

1.  现在，继续在同一个文件中输入。这部分是用于解析命令行选项、进行计算和打印结果。我们使用`getopt()`和`switch`语句来解析选项。注意，这次我们还可以进行乘法运算：

```
    /* Parse command-line options */
    while ((opt = getopt(argc, argv, "smh")) != -1)
    {
        switch (opt)
        {
           case 's': /* sum the integers */
               sum = 0;
               for (i=2; i<argc; i++)
                   sum = sum + atoi(argv[i]);
               break;
           case 'm': /* multiply the integers */
               sum = 1;
               for (i=2; i<argc; i++)
                   sum = sum * atoi(argv[i]);
               break;
           case 'h': /* -h for help */
               printhelp(argv[0]);
               return 0;
           default: /* in case of invalid options*/
               printhelp(argv[0]);
               return 1;
        }
    }
    printf("Total: %i\n", sum);
    return 0;
}
```

1.  最后，在同一个文件中，将`printhelp()`函数添加到底部。这个函数打印一个帮助消息，有时被称为*用法*消息。当用户使用`-h`选项或发生某种形式的错误时，例如没有给出参数时，将显示此消息：

```
void printhelp(char progname[])
{
    printf("%s [-s] [-m] integer ...\n", progname);
    printf("-s sums all the integers\n"
        "-m multiplies all the integers\n"
        "This program takes any number of integer "
        "values and either add or multiply them.\n"
        "For example: %s -m 5 5 5\n", progname);
} 
```

1.  保存并关闭文件。

1.  现在，是时候编译程序了。这次，我们将尝试使用 Make：

```
$> make new-sum
cc     new-sum.c   -o new-sum
```

1.  测试程序：

```
$> ./new-sum
./new-sum [-s] [-m] integer ...
-s sums all the integers
-m multiplies all the integers
This program takes any number of integer values and either add or multiply them.
For example: ./new-sum -m 5 5 5
$> ./new-sum -s 5 5 5
Total: 15
$> ./new-sum -m 5 5 5
Total: 125
```

## 工作原理…

第一部分与上一个示例非常相似，只是我们有一些更多的变量`unistd.h`，这是`getopt()`函数所需的，我们用它来解析程序的选项。

还有一个新的看起来有点奇怪的部分；那就是第一行：

```
#define _XOPEN_SOURCE 500
```

我们将在本书的后面详细介绍这个。但现在，只需知道它是一个特性宏，我们用它来遵循`getopt()`，我们将在下一个食谱中详细介绍。

### getopt()函数

这个食谱的下一步——第二步——是令人兴奋的部分。在这里，我们使用`getopt()`函数解析选项，该函数代表*获取选项*。

使用`getopt()`的方法是在`while`循环中循环遍历参数，并使用`switch`语句捕获选项。让我们更仔细地看看`while`循环，并将其分解成较小的部分：

```
while ((opt = getopt(argc, argv, "smh")) != -1)
```

`getopt()`函数返回它解析的选项的实际字母。这意味着第一部分`opt = getopt`将选项保存到`opt`变量中，但只保存实际的字母。因此，例如，`-h`保存为`h`。

然后，我们有必须传递给`getopt()`函数的参数，即`argc`（参数计数）、`argv`（实际参数）和最后应该接受的选项（这里是`smh`，它被翻译为`-s`、`-m`和`-h`）。

最后一部分`!= -1`是为了`while`循环。当`getopt()`没有更多选项返回时，它返回-1，表示它已经完成了选项解析。这时`while`循环应该结束。

### 在 while 循环内

在循环内，我们使用`switch`语句对每个选项执行特定的操作。在每个`case`下，我们执行计算并在完成后`break`出该`case`。就像在上一个食谱中一样，我们使用`atoi()`将参数字符串转换为整数。

在`h`情况下（帮助的`-h`选项），我们打印帮助消息并返回代码 0。我们请求帮助，因此这不是一个错误。但在此之下，我们有默认情况，即如果没有其他选项匹配，则捕获的情况；也就是说，用户输入了一个不被接受的选项。这确实是一个错误，所以在这里，我们返回代码 1，表示错误。

### 帮助消息函数

帮助消息应该显示程序接受的各种选项、其参数和一个简单的使用示例。

使用`printf()`，我们可以在代码中将长行分成多个较小的行，就像我们在这里做的一样。独特的字符序列`\n`是一个换行字符。行将在放置这个字符的地方断开。

### 编译和运行程序

在这个食谱中，我们使用 Make 编译了程序。Make 实用程序又使用`cc`，它只是`gcc`的符号链接。本书后面，我们将学习如何通过在 Makefile 中编写规则来改变 Make 的行为。

然后我们尝试了这个程序。首先，我们没有使用任何选项或参数来运行它，导致程序退出并显示帮助文本（返回值为 1）。

然后我们尝试了两个选项：`-s`用于总结所有整数和`-m`用于将所有整数相乘。

# 在内置手册页中查找信息

在这个食谱中，我们将学习如何在内置手册页中查找信息。我们将学习如何查找从命令、系统调用到**标准库函数**的所有内容。一旦习惯使用它们，手册页就非常强大。与在互联网上搜索答案相比，查看手册通常更快、更准确。

## 准备工作

一些手册页（库调用和系统调用）作为 Debian 和 Ubuntu 的*build-essential*软件包的一部分安装。在基于 Fedora 的发行版（如 CentOS）中，这些通常已经作为*man pages*软件包的一部分安装在基本系统中。如果您缺少一些手册页，请确保已安装这些软件包。查看本章中的第一个食谱，了解更多关于安装软件包的信息。

如果你使用的是最小化或精简安装，可能没有安装`man`命令。如果是这种情况，你需要使用发行版包管理器安装两个软件包。软件包名称为*man-db*（几乎所有发行版都是一样的）和*manpages*（在基于 Debian 的系统中），或者*man-pages*（在基于 Fedora 的系统中）用于实际的手册页。在基于 Debian 的系统中，你还需要安装*build-essential*软件包。

## 操作方法...

让我们逐步探索手册页，如下所示：

1.  在控制台中键入`man ls`。你会看到`ls`命令的手册页。

1.  使用*箭头*键或*Enter*键逐行滚动手册页。

1.  按下*空格键*以一次滚动一个完整页面（窗口）。

1.  按下字母*b*来向上滚动整个页面。一直按*b*直到达到顶部。

1.  现在，按*/*打开搜索提示。

1.  在搜索提示中键入`human-readable`，然后按*Enter*。手册页现在会自动向前滚动到该单词的第一次出现。

1.  现在你可以按*n*键跳转到下一个单词的出现位置 - 如果有的话。

1.  按*q*退出手册。

### 调查不同的部分

有时，同名但在不同部分的手册页可能有多个。在这里，我们将调查这些部分，并学习如何指定我们感兴趣的部分：

1.  在命令提示符中键入`man printf`。你将看到`printf`命令的手册页，而不是同名的 C 函数。

1.  按*q*退出手册。

1.  现在，在控制台中键入`man 3 printf`。这是`printf()`C 函数的手册页。`3`表示手册的第三部分。查看手册页的标题，你会看到你现在所在的部分。此刻应该会显示**PRINTF(3)**。

1.  让我们列出所有的部分。退出你正在查看的手册页，然后在控制台中键入`man man`。向下滚动一点，直到找到列出所有部分的表格。在那里，你还会找到每个部分的简短描述。正如你所看到的，第三部分是用于库调用的，这就是`printf()`所在的部分。

1.  通过在控制台中键入`man 2 unlink`来查找`unlink()`系统调用的手册。

1.  退出手册页，然后在控制台中键入`man unlink`。这次，你会看到`unlink`命令的手册。

## 工作原理...

手册总是从第一部分开始，并打开它找到的第一个手册。这就是为什么当我们不指定部分号码时，你会得到`printf`和`unlink`命令，而不是 C 函数和系统调用。查看打开的手册页的标题总是一个好主意，以确保你正在阅读正确的手册页。

## 还有更多...

从上一个步骤中记住，我“只是知道”当没有更多选项需要解析时，`getopt()`返回`-1`？其实不是，这都在手册中有。通过在控制台中键入`man 3 getopt`来打开`getopt()`的手册。向下滚动到*Return value*标题。在那里，你可以阅读有关`getopt()`返回值的所有信息。几乎所有涵盖库函数和系统调用的手册页都有以下标题：名称、概要、描述、返回值、环境、属性、符合、注释、示例和参见。

概要标题列出了我们需要包含的特定函数的头文件。这真的很有用，因为我们无法记住每个函数及其对应的头文件。

提示

手册中有很多关于手册本身的有用信息 - `man man` - 所以至少浏览一下。在本书中，我们将经常使用手册页来查找有关库函数和系统调用的信息。

# 搜索手册以获取信息

如果我们不知道特定命令、函数或系统调用的确切名称，我们可以搜索系统中所有手册以找到正确的。在这个示例中，我们将学习如何使用 `apropos` 命令搜索手册页。

## 准备工作

这里适用于前一个示例的相同要求。

## 操作方法…

让我们搜索不同的单词，每一步都缩小我们的结果：

1.  输入 `apropos directory`。会出现一个手册页的长列表。在每个手册页后面，都有括号里的一个数字。这个数字是手册页所在的章节。

1.  要将搜索范围缩小到只有第三章节（库调用），输入 `apropos -s 3 directory`。

1.  让我们进一步缩小搜索范围。输入 `apropos -s 3 -a remove directory`。`-a` 选项代表 *and*。

## 工作原理…

`apropos` 命令搜索手册页描述和关键字。当我们用 `apropos -s 3 -a remove directory` 缩小搜索范围时，`-a` 选项代表 *and*，表示 *remove* 和 *directory* 必须同时存在。如果我们省略 `-a` 选项，它会搜索这两个关键字，而不管它们是否都存在。

关于 `apropos` 如何工作的更多信息，请参阅其手册页（`man apropos`）。

## 更多内容…

如果我们只想知道特定命令或函数的作用，可以使用 `whatis` 命令查找它的简短描述，如下所示：

```
$> whatis getopt 
```

```
getopt (1)           - parse command options (enhanced) 
```

```
getopt (3)           - Parse command-line options
```

```
$> whatis creat 
```

```
creat (2)            - open and possibly create a file
```

```
$> whatis opendir 
```

```
opendir (3)          - open a directory
```
