# 第八章：创建共享库

在本章中，我们将学习库是什么，以及为什么它们是 Linux 的重要组成部分。我们还将了解静态库和动态库之间的区别。当我们知道库是什么时，我们开始编写我们自己的库——静态和动态的。我们还快速查看动态库的内部。

使用库有许多好处，例如，开发人员不需要一遍又一遍地重新发明功能，因为通常库中已经存在一个现有的功能。动态库的一个重要优势是，生成的程序大小要小得多，并且即使在程序编译完成后，库也是可升级的。

在本章中，我们将学习如何制作具有有用功能的自己的库，并将其安装到系统上。知道如何制作和安装库使您能够以标准化的方式与他人共享您的功能。

在本章中，我们将涵盖以下配方：

+   库的作用和意义

+   创建静态库

+   使用静态库

+   创建动态库

+   在系统上安装动态库

+   在程序中使用动态库

+   编译一个静态链接的程序

# 技术要求

在本章中，我们将需要**GNU 编译器集合**（**GCC**）编译器和 Make 工具。您可以在*第一章*中找到这些工具的安装说明，*获取必要的工具并编写我们的第一个 Linux 程序*。本章的所有代码示例都可以在本章的 GitHub 目录中找到，网址为[`github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch8`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch8)。

点击以下链接查看《代码实战》视频：[`bit.ly/3fygqOm`](https://bit.ly/3fygqOm)

# 库的作用和意义

在我们深入了解库的细节之前，了解它们是什么以及它们对我们的重要性是至关重要的。了解静态库和动态库之间的区别也很重要：

这些知识将使您在制作自己的库时能够做出更明智的选择。

**动态库**是动态**链接**到使用它的二进制文件的。这意味着库代码不包含在二进制文件中。库驻留在二进制文件之外。这有几个优点。首先，由于库代码不包含在其中，生成的二进制文件大小会更小。其次，库可以在不需要重新编译二进制文件的情况下进行更新。缺点是我们不能将动态库从系统中移动或删除。如果这样做，二进制文件将不再起作用。

另一方面，**静态库**包含在二进制文件中。这样做的优点是一旦编译完成，二进制文件将完全独立于库。缺点是二进制文件会更大，并且库不能在不重新编译二进制文件的情况下更新。

我们已经在*第三章**中看到了一个动态库的简短示例，在 Linux 中深入 C*。

在这个配方中，我们将看一些常见的库。我们还将通过包管理器在系统上安装一个新的库，然后在程序中使用它。

## 准备工作

对于这个配方，您将需要 GCC 编译器。您还需要通过`su`或`sudo`以 root 访问系统。

## 操作方法…

在这个配方中，我们将探索一些常见的库，看看它们在系统上的位置，然后安装一个新的库并查看库的内部。在这个配方中，我们只处理动态库。

1.  让我们首先看看您系统中已经存在的许多库。这些库将驻留在一个或多个这些目录中，具体取决于您的发行版：

```
/usr/lib
/usr/lib64
/usr/lib32
```

1.  现在，我们将使用 Linux 发行版软件包管理器在系统上安装一个新的库。我们将安装的库是用于**cURL**的，这是一个从互联网上获取文件或数据的应用程序和库，例如通过**超文本传输协议**（**HTTP**）。根据您的发行版，按照以下说明进行操作：

- **Debian/Ubuntu**:

```
   $> sudo apt install libcurl4-openssl-dev
```

- **Fedora/CentOS/Red Hat**:

```
   $> sudo dnf install libcurl-devel
```

1.  现在，让我们使用`nm`来查看库的内部。但首先，我们需要使用`whereis`找到它。不同发行版的库路径是不同的。这个示例来自 Debian 10 系统。我们要找的文件是`.so`文件。请注意，我们使用`grep`和`nm`一起使用，只列出带有`T`的行。这些是库提供的函数。如果我们去掉`grep`部分，我们还会看到这个库依赖的函数。我们还在命令中添加了`head`，因为函数列表很长。如果您想看到所有函数，请省略`head`：

```
$> whereis libcurl
libcurl: /usr/lib/x86_64-linux-gnu/libcurl.la
/usr/lib/x86_64-linux-gnu/libcurl.a /usr/lib/x86_64
linux-gnu/libcurl.so
$> nm -D /usr/lib/x86_64-linux-gnu/libcurl.so \
> | grep " T " | head -n 7
000000000002f750 T curl_easy_cleanup
000000000002f840 T curl_easy_duphandle
00000000000279b0 T curl_easy_escape
000000000002f7e0 T curl_easy_getinfo
000000000002f470 T curl_easy_init
000000000002fc60 T curl_easy_pause
000000000002f4e0 T curl_easy_perform
```

1.  现在我们对库有了更多了解，我们可以在程序中使用它。在文件中编写以下代码，并将其保存为`get-public-ip.c`。该程序将向位于`ifconfig.me`的 Web 服务器发送请求，并给出您的公共**Internet Protocol**（**IP**）地址。cURL 库的完整手册可以在[`curl.se/libcurl/c/`](https://curl.se/libcurl/c/)上找到。请注意，我们不从 cURL 打印任何内容。库将自动打印从服务器接收到的内容：

```
#include <stdio.h>
#include <curl/curl.h>
int main(void)
{
    CURL *curl;
    curl = curl_easy_init();
    if(curl) 
    {
        curl_easy_setopt(curl, CURLOPT_URL, 
            "https://ifconfig.me"); 
        curl_easy_perform(curl); 
        curl_easy_cleanup(curl);
    }
    else
    {
        fprintf(stderr, "Cannot initialize curl\n");
        return 1;
    }
    return 0;
}
```

1.  编译代码。请注意，我们还必须使用`-l`选项链接到 cURL 库：

```
$> gcc -Wall -Wextra -pedantic -std=c99 -lcurl \
> get-public-ip.c -o get-public-ip
```

1.  现在，最后，我们可以运行程序来获取我们的公共 IP 地址。我的 IP 地址在下面的输出中被掩盖了：

```
$> ./get-public-ip 
158.174.xxx.xxx
```

## 工作原理…

在这里，我们已经看到了使用库添加新功能所涉及的所有步骤。我们使用软件包管理器在系统上安装了库。我们使用`whereis`找到了它的位置，使用`nm`调查了它包含的函数，最后在程序中使用了它。

`nm`程序提供了一种快速查看库包含哪些函数的方法。我们在这个示例中使用的`-D`选项是用于动态库的。我们使用`grep`只查看库提供的函数；否则，我们还会看到这个库依赖的函数（这些行以`U`开头）。

由于这个库不是`libc`的一部分，我们需要使用`-l`选项将其链接到`gcc`。库的名称应该紧跟在`l`后面，没有任何空格。

[ifconfig.me](http://ifconfig.me) 网站是一个返回请求该站点的客户端的公共 IP 的站点和服务。

## 还有更多…

cURL 也是一个程序。许多 Linux 发行版都预装了它。cURL 库提供了一种方便的方式，在您自己的程序中使用 cURL 函数。

您可以运行`curl ifconfig.me`来获得与我们编写的程序相同的结果，假设您已经安装了 cURL。

# 创建一个静态库

在*第三章*中，*深入 Linux 中的 C 编程*，我们看到了如何创建动态库以及如何从当前工作目录链接它。在这个示例中，我们将创建一个**静态库**。

静态库在编译过程中包含在二进制文件中。优点是二进制文件更具可移植性和独立性。我们可以在编译后删除静态库，程序仍然可以正常工作。

缺点是二进制文件会稍微变大，而且在将库编译到程序中后无法更新库。

了解如何创建静态库将使在新程序中分发和重用您的函数变得更加容易。

## 准备工作

对于这个示例，我们将需要 GCC 编译器。我们还将在这个示例中使用一个名为`ar`的工具。`ar`程序几乎总是默认安装的。

## 如何做…

在这个教程中，我们将制作一个小的静态库。该库将包含两个函数：一个用于将摄氏度转换为华氏度，另一个用于将摄氏度转换为开尔文：

1.  让我们从编写库函数开始。在文件中写入以下代码，并将其保存为`convert.c`。该文件包含我们的两个函数：

```
float c_to_f(float celsius)
{
    return (celsius*9/5+32);
}
float c_to_k(float celsius)
{
    return (celsius + 273.15);
}
```

1.  我们还需要一个包含这些函数原型的头文件。创建另一个文件，并在其中写入以下代码。将其保存为`convert.h`：

```
float c_to_f(float celsius);
float c_to_k(float celsius);
```

1.  制作库的第一步是将`convert.c`编译成 GCC 的`-c`选项：

```
$> gcc -Wall -Wextra -pedantic -std=c99 -c convert.c
```

1.  我们现在应该在当前目录中有一个名为`convert.o`的文件。我们可以使用`file`命令来验证这一点，它还会告诉我们文件的类型：

```
$> file convert.o
convert.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```

1.  使其成为静态库的最后一步是使用`ar`命令将其打包。`-c`选项表示*创建*存档；`-v`选项表示*详细*输出；`-r`选项表示*替换*具有相同名称的成员。名称`libconvert.a`是我们的库将得到的结果文件名：

```
$> ar -cvr libconvert.a convert.o 
a - convert.o
```

1.  在继续之前，让我们用`nm`查看我们的静态库：

```
$> nm libconvert.a 
convert.o:
0000000000000000 T c_to_f
0000000000000037 T c_to_k
```

## 它是如何工作的…

正如我们在这里看到的，静态库只是存档中的一个对象文件。

当我们用`file`命令查看对象文件时，我们注意到它说*not stripped*，这意味着所有的**符号**仍然在文件中。*符号*是暴露函数的东西，使得程序可以访问和使用它们。在下一个教程中，我们将回到符号和*stripped*与*not stripped*的含义。

## 参见

在其手册页`man 1 ar`中有关`ar`的大量有用信息，例如，可以修改和删除已经存在的静态库。

# 使用静态库

在这个教程中，我们将在程序中使用上一个教程中创建的静态库。使用静态库比使用动态库要容易一些。我们只需将静态库（存档文件）添加到将编译为最终二进制文件的文件列表中。

知道如何使用静态库将使您能够使用其他人的库并重用自己的代码作为静态库。

## 准备工作

对于这个教程，您将需要`convert.h`文件和静态库文件`libconvert.a`。您还需要 GCC 编译器。

## 如何做…

在这里，我们将编写一个小程序，该程序使用我们在上一个教程中创建的库中的函数：

1.  在文件中写入以下代码，并将其保存为`temperature.c`。注意从当前目录包含头文件的语法。

该程序接受两个参数：一个选项（`-f`或`-k`，分别表示华氏度或开尔文）和一个摄氏度作为浮点值。然后程序将根据所选的选项将摄氏度转换为华氏度或开尔文：

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "convert.h"
void printUsage(FILE *stream, char progname[]);
int main(int argc, char *argv[])
{
    if ( argc != 3 )
    {
        printUsage(stderr, argv[0]);
        return 1;
    }
    if ( strcmp(argv[1], "-f") == 0 )
    {
        printf("%.1f C = %.1f F\n", 
            atof(argv[2]), c_to_f(atof(argv[2])));
    }
    else if ( strcmp(argv[1], "-k") == 0  )
    {
        printf("%.1f C = %.1f F\n", 
            atof(argv[2]), c_to_k(atof(argv[2])));
    }
    else
    {
        printUsage(stderr, argv[0]);
        return 1;
    }

    return 0;
}
void printUsage(FILE *stream, char progname[])
{
    fprintf(stream, "%s [-f] [-k] [temperature]\n"
        "Example: %s -f 25\n", progname, progname);
}
```

1.  让我们编译这个程序。要包含静态库，我们只需将其添加到 GCC 的文件列表中。还要确保`convert.h`头文件在您当前的工作目录中：

```
$> gcc -Wall -Wextra -pedantic -std=c99 \
> temperature.c libconvert.a -o temperature
```

1.  现在我们可以用一些不同的温度测试程序：

```
$> ./temperature -f 30
30.0 C = 86.0 F
$> ./temperature -k 15
15.0 C = 288.1 F
```

1.  最后，使用`nm`查看生成的`temperature`二进制文件：

```
c_to_f, c_to_k, printUsage, and main (the Ts). We also see which functions from dynamic libraries the program is depending on—for example, printf (preceded by a U). What we see here are called *symbols*. 
```

1.  由于该二进制文件将用作独立程序，我们不需要符号。可以使用`strip`命令从二进制文件中*strip*符号。这会使程序的大小变小一点。一旦我们从二进制文件中删除了符号，让我们再次用`nm`查看它：

```
$> strip temperature
$> nm temperature
nm: temperature: no symbols
```

1.  我们可以用`file`命令查看程序或库是否被剥离。请记住，静态库不能被剥离；否则，链接器将无法看到函数，链接将失败：

```
$> file temperature
temperature: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter/lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=95f583af98ff899c657ac33d6a014493c44c362b, stripped
$> file convert.o
convert.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```

## 它是如何工作的…

当我们想在程序中使用静态库时，我们将存档文件的文件名和程序的`c`文件提供给 GCC，从而生成一个包含静态库的二进制文件。

在最后几个步骤中，我们使用`nm`检查了二进制文件，显示了所有符号。然后我们使用`strip`命令剥离 - 移除 - 这些符号。如果我们使用`file`命令查看`ls`，`more`，`sleep`等程序，我们会注意到这些程序也被*剥离*。这意味着程序已经删除了其符号。

静态库必须保持其符号不变。如果它们被移除 - 剥离 - 链接器将找不到函数，链接过程将失败。因此，我们永远不应该剥离我们的静态库。

# 创建一个动态库

虽然静态库方便且易于创建和使用，**动态库**更常见。正如我们在本章开头看到的那样，许多开发人员选择提供库而不仅仅是程序 - 例如，cURL。

在这个配方中，我们将重新制作本章前面介绍的“创建静态库”配方中的库，使其成为一个动态库。

了解如何创建动态库使您能够将代码分发为其他开发人员易于实现的库。

## 准备工作

对于这个配方，您将需要本章前面的“创建静态库”中的两个`convert.c`和`convert.h`文件。您还需要 GCC 编译器。

## 如何做…

在这里，我们从本章前面的“创建静态库”中的`convert.c`创建一个动态库：

1.  首先，让我们删除之前创建的对象文件和旧的静态库。这样可以确保我们不会错误地使用错误的对象文件或错误的库：

```
$> rm convert.o libconvert.a
```

1.  我们需要做的第一件事是从`c`文件创建一个新的对象文件。`-c`选项创建一个对象文件，而不是最终的二进制文件。`-fPIC`选项告诉 GCC 生成所谓的`file`：

```
$> gcc -Wall -Wextra -pedantic -std=c99 -c -fPIC \
> convert.c
$> file convert.o 
convert.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
```

1.  下一步是创建一个`.so`文件，`-shared`选项做了它说的 - 它创建了一个共享对象。`-Wl`选项意味着我们想要将所有逗号分隔的选项传递给链接器。在这种情况下，传递给链接器的选项是`-soname`，参数是`libconvert.so`，它将动态库的名称设置为*libconvert.so*。最后，`-o`选项指定了输出文件的名称。然后，我们使用`nm`列出了这个共享库提供的符号。由`T`前缀的符号是这个库提供的符号：

```
$> gcc -shared -Wl,-soname,libconvert.so -o \
> libconvert.so.1 convert.o
$> nm -D libconvert.so.1
00000000000010f5 T c_to_f
000000000000112c T c_to_k
                 w __cxa_finalize
                 w __gmon_start__
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
```

## 工作原理…

创建动态库涉及两个步骤：创建一个位置无关的对象文件，并将该文件打包成一个`.so`文件。

共享库中的代码在运行时加载。由于它无法预测自己将在内存中的何处结束，因此需要是位置无关的。这样，代码将在内存中的任何位置正确工作。

`-Wl，-soname，libconvert.so` GCC 选项可能需要进一步解释。`-Wl`选项告诉 GCC 将逗号分隔的单词视为链接器的选项。由于我们不能使用空格 - 那将被视为一个新的选项 - 我们用逗号代替`-soname`和`libconvert.so`。然而，链接器将其视为`-soname libconvert.so`。

`soname`是*共享对象名称*的缩写，它是库中的内部名称。在引用库时使用这个名称。

使用`-o`选项指定的实际文件名有时被称为库的*真实名称*。使用包含库版本号的真实名称是一个标准约定，例如在这个例子中使用`1`。也可以包括一个次要版本 - 例如，`1.3`。在我们的例子中，它看起来像这样：`libconvert.so.1.3`。*真实名称*和*soname*都必须以`lib`开头，缩写为*库*。总的来说，这给我们提供了真实名称的五个部分：

+   `lib`（库的缩写）

+   `convert`（库的名称）

+   `.so`（扩展名，缩写为*共享对象*）

+   `.1`（库的主要版本）

+   `.3`（库的次要版本，可选）

## 还有更多…

与静态库相反，动态库可以被剥离并且仍然可以工作。但是请注意，剥离必须在创建`.so`文件的动态库之后进行。如果我们剥离对象（`.o`）文件，那么我们将丢失所有符号，使其无法链接。但是`.so`文件将符号保留在一个称为`.dynsym`的特殊表中，`strip`命令不会触及。可以使用`readelf`命令的`--symbols`选项在剥离的动态库上查看此表。因此，如果`nm`命令在动态库上回复*no symbols*，可以尝试使用`readelf --symbols`。

## 另请参阅

**GCC**是一个庞大的软件，有很多选项。GNU 的网站上提供了每个 GCC 版本的 PDF 手册。这些手册大约有 1000 页，可以从 https://gcc.gnu.org/onlinedocs/下载。

# 在系统上安装动态库

我们现在已经看到如何创建静态库和动态库，在*第三章**，深入 Linux 中的 C 编程*中，我们甚至看到了如何从我们的主目录中使用动态库。但现在，是时候将动态库系统范围内安装，以便计算机上的任何用户都可以使用它了。

知道如何在系统上安装动态库将使您能够为任何用户添加系统范围的库。

## 准备工作

对于这个步骤，您将需要在上一个步骤中创建的`libconvert.so.1`动态库。您还需要 root 访问系统，可以通过`sudo`或`su`来获取。

## 如何做...

安装动态库只是将库文件和头文件移动到正确的目录并运行命令的问题。但是，我们应该遵循一些约定：

1.  我们需要做的第一件事是将库文件复制到系统的正确位置。用户安装的库的常见目录是`/usr/local/lib`，我们将在这里使用。由于我们将文件复制到家目录之外的地方，我们需要以 root 用户的身份执行该命令。我们将在这里使用`install`来设置用户、组和模式，因为它是系统范围的安装，我们希望它由 root 拥有。它还应该是可执行的，因为它将在运行时被包含和执行：

```
$> sudo install -o root -g root -m 755 \
> libconvert.so.1 /usr/local/lib/libconvert.so.1
```

1.  现在，我们必须运行`ldconfig`命令，它将创建必要的链接并更新缓存。

```
$> sudo ldconfig
$> cd /usr/local/lib/
$> ls -og libconvert*
lrwxrwxrwx 1 15 dec 27 19:12 libconvert.so ->
libconvert.so.1
-rwxr-xr-x 1 15864 dec 27 18:16 libconvert.so.1
```

1.  我们还必须将头文件复制到系统目录；否则，用户将不得不手动下载并跟踪头文件，这不太理想。用户安装的头文件的一个好地方是`/usr/local/include`。单词*include*来自 C 语言的`#include`行：

```
$> sudo install -o root -g root -m 644 convert.h \
> /usr/local/include/convert.h
```

1.  由于我们已经在整个系统中安装了库和头文件，我们可以继续从当前工作目录中删除它们。这样做将确保我们在下一个步骤中使用正确的文件：

```
$> rm libconvert.so.1 convert.h
```

## 它是如何工作的...

我们使用`install`程序安装了库文件和头文件。这个程序非常适合这样的任务，因为它可以在单个命令中设置用户（`-o`选项）、组（`-g`选项）和模式（`-m`选项）。如果我们使用`cp`来复制文件，它将由创建它的用户拥有。我们总是希望系统范围内的二进制文件、库和头文件由 root 用户拥有，以确保安全。

`/usr/local`目录是用户创建的东西的一个好地方。我们将库放在`/usr/local/lib`下，将头文件放在`/usr/local/include`下。系统库和头文件通常放在`/usr/lib`和`/usr/include`中。

当我们稍后使用库时，系统将在以`.so`结尾的文件中查找它，因此我们需要一个指向库的符号链接，名称为`libconvert.so`。但我们不需要自己创建该链接；`ldconfig`已经为我们处理了。

另外，由于我们已经将头文件放在`/usr/local/include`中，我们不再需要在当前工作目录中拥有该文件。现在我们可以像包含任何其他系统头文件一样使用相同的语法。我们将在下一个示例中看到这一点。

# 在程序中使用动态库

现在我们已经创建了一个动态库并将其安装在系统上，现在是时候在程序中尝试它了。实际上，自从本书的开头以来，我们一直在使用动态库而不自知。诸如`printf()`等函数都是标准库的一部分。在本章前面的*库的作用和原因*示例中，我们使用了另一个名为 cURL 的动态库。在这个示例中，我们将使用我们在上一个示例中安装的自己的库。

了解如何使用自定义库将使您能够使用其他开发人员的代码，这将加快开发过程。通常没有必要重新发明轮子。

## 准备工作

对于这个示例，我们将需要本章前面的*使用静态库*示例中的`temperature.c`代码。该程序将使用动态库。在尝试此示例之前，您还需要完成上一个示例。

## 如何做...

在这个示例中，我们将使用`temperature.c`代码来利用我们在上一个示例中安装的库：

1.  由于我们将使用`/usr/local/include`，我们必须修改`temperature.c`中的`#include`行。`temperature.c`中的*第 4 行*当前显示为：

```
#include "convert.h"
```

将前面的代码更改为：

```
#include <convert.h>
```

然后，将其保存为`temperature-v2.c`。

1.  现在我们可以继续编译程序了。GCC 将使用系统范围的头文件和库文件。请记住，我们需要使用`-l`选项链接到库。这样做时，我们必须省略`lib`部分和`.so`结尾：

```
$> gcc -Wall -Wextra -pedantic -std=c99 \
> -lconvert temperature-v2.c -o temperature-v2
```

1.  然后，让我们尝试一些不同的温度：

```
$> ./temperature-v2 -f 34
34.0 C = 93.2 F
$> ./temperature-v2 -k 21
21.0 C = 294.1 F
```

1.  我们可以使用`ldd`验证动态链接的库。当我们在我们的程序上运行此工具时，我们会看到我们的`libconvert.so`库，`libc`和称为`vdso`（*虚拟动态共享对象*）的东西：

```
$> ldd temperature-v2
        linux-vdso.so.1 (0x00007fff4376c000)
        libconvert.so => /usr/local/lib/libconvert.so (0x00007faaeefe2000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007faaeee21000)
        /lib64/ld-linux-x86-64.so.2 (0x00007faaef029000)
```

## 它是如何工作的...

当我们从当前目录包含本地头文件时，语法是`#include "file.h"`。但对于系统范围的头文件，语法是`#include <file.h>`。

由于库现在安装在系统目录之一中，我们不需要指定路径。仅需使用`-lconvert`链接到库即可。这样做时，所有常见的系统范围目录都会搜索该库。当我们使用`-l`进行链接时，我们省略了文件名的`lib`部分和`.so`结尾——链接器会自行解决这个问题。

在最后一步中，我们使用`ldd`验证了我们正在使用`libconvert.so`的系统范围安装。在这里，我们还看到了标准 C 库`libc`和称为`vdso`的东西。标准 C 库具有我们一次又一次使用的所有常用函数，例如`printf()`。然而，`vdso`库有点更加神秘，这不是我们要在这里讨论的内容。简而言之，它将一小部分经常使用的系统调用导出到用户空间，以避免过多的上下文切换，这将影响性能。

还有更多...

在本章中，我们已经谈论了很多关于`ld`的内容。为了更深入地了解链接器，我建议您阅读其手册页，使用`man 1 ld`。

## 另请参阅

有关`ldd`的更多信息，请参阅`man 1 ldd`。

对于好奇的人，可以在`man 7 vdso`中找到有关`vdso`的详细解释。

# 编译静态链接程序

现在我们对库和链接有了如此深刻的理解，我们可以创建一个**静态链接**程序——也就是说，一个将所有依赖项编译到其中的程序。这使得程序基本上不依赖于其他库。制作静态链接程序并不常见，但有时可能是可取的——例如，如果由于某种原因需要将单个预编译的二进制文件分发到许多计算机而不必担心安装所有的库。但请注意：并不总是可能创建完全不依赖于其他程序的程序。如果一个程序使用了依赖于另一个库的库，这就不容易实现。

制作和使用静态链接程序的缺点是它们的大小变得更大。此外，不再能够更新程序的库而不重新编译整个程序。因此，请记住这只在极少数情况下使用。

但是，通过了解如何编译静态链接程序，你不仅可以增强你的知识，还可以将预编译的二进制文件分发到没有必要的库的系统上，而且可以在许多不同的发行版上实现。

## 准备工作

对于这个示例，你需要完成前两个示例——换句话说，你需要在系统上安装`libconvert.so.1`库，并且需要编译`temperature-v2.c`。像往常一样，你还需要 GCC 编译器。

## 如何做…

在这个示例中，我们将编译`temperature-v2.c`的静态链接版本。然后，我们将从系统中删除库，并注意到静态链接的程序仍然可以工作，而另一个则不能：

重要提示

在 Fedora 和 CentOS 上，默认情况下不包括`libc`的静态库。要安装它，运行`sudo dnf install glibc-static`。

1.  为了静态链接到库，我们需要所有库的静态版本。这意味着我们必须重新创建库的存档（`.a`）版本，并将其安装。这些步骤与本章前面的*创建静态库*示例中的步骤相同。首先，如果我们仍然有对象文件，我们将删除它。然后，我们创建一个新的对象文件，并从中创建一个存档：

```
$> rm convert.o
$> gcc -Wall -Wextra -pedantic -std=c99 -c convert.c
$> ar -cvr libconvert.a convert.o 
a - convert.o
```

1.  接下来，我们必须在系统上安装静态库，最好与动态库放在同一个位置。静态库不需要可执行文件，因为它是在编译时包含的，而不是在运行时包含的：

```
$> sudo install -o root -g root -m 644 \
> libconvert.a /usr/local/lib/libconvert.a
```

1.  现在，编译`temperature-v2.c`的静态链接版本。`-static`选项使二进制文件静态链接，这意味着它将在二进制文件中包含库代码：

```
$> gcc -Wall -Wextra -pedantic -std=c99 -static \
> temperature-v2.c -lconvert -o temperature-static
```

1.  在我们尝试这个程序之前，让我们用`ldd`来检查它，并用`du`来查看它的大小。请注意，在我的系统上，二进制文件现在几乎有 800 千字节（在另一个系统上，它有 1.6 兆字节）。与动态版本相比，动态版本只有大约 20 千字节：

```
$> du -sh temperature-static 
788K    temperature-static
$> du -sh temperature-v2
20K     temperature-v2
$> ldd temperature-static 
        not a dynamic executable
```

1.  现在，让我们尝试这个程序：

```
$> ./temperature-static -f 20
20.0 C = 68.0 F
```

1.  让我们从系统中删除静态和动态库：

```
$> sudo rm /usr/local/lib/libconvert.a \
> /usr/local/lib/libconvert.so \ 
> /usr/local/lib/libconvert.so.1
```

1.  现在，让我们尝试动态链接的二进制文件，由于我们已经删除了它所依赖的库，所以它不应该工作：

```
$> ./temperature-v2 -f 25
./temperature-v2: error while loading shared
libraries: libconvert.so: cannot open shared object
file: No such file or directory
```

1.  最后，让我们尝试静态链接的二进制文件，它应该和以前一样正常工作：

```
$> ./temperature-static -f 25
25.0 C = 77.0 F
```

## 工作原理…

静态链接的程序包括所有库的所有代码，这就是为什么在这个示例中我们的二进制文件变得如此庞大。要构建一个静态链接的程序，我们需要程序所有库的静态版本。这就是为什么我们需要重新创建静态库并将其放在系统目录中的原因。我们还需要标准 C 库的静态版本，如果我们使用的是 CentOS 或 Fedora 机器，我们会安装它。在 Debian/Ubuntu 上，它已经安装好了。
