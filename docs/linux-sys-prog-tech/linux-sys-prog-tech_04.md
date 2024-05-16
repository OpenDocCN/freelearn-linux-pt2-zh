# 第四章：处理程序中的错误

在本章中，我们将学习在 Linux 中的 C 程序中的**错误处理**，具体来说是如何捕获错误并打印相关信息。我们还将学习如何将这些知识与我们之前学到的关于**stdin**、**stdout**和**stderr**的知识结合起来。

我们将继续学习系统调用的路径，并了解一个特定的变量称为**errno**。大多数系统调用在发生错误时使用这个变量保存特定的错误值。

在程序中处理错误将使它们更加稳定。错误确实会发生；只是要正确处理它们。一个良好处理的错误对最终用户来说不会看起来像是错误。例如，不要让你的程序在硬盘已满时以某种神秘的方式崩溃，最好是捕获错误并打印一个人类可读且友好的消息。这样，对最终用户来说，它只是信息而不是错误。这反过来会使你的程序看起来更友好，最重要的是更加稳定。

在本章中，我们将涵盖以下食谱：

+   为什么错误处理在系统编程中很重要

+   处理一些常见错误

+   错误处理和`errno`

+   处理更多的`errno`宏

+   使用`strerror()`和`errno`

+   使用`perror()`和`errno`

+   返回一个错误值

让我们开始吧！

# 技术要求

对于本章，你将需要 GCC 编译器、Make 工具以及所有手册页（dev 和 POSIX）已安装。我们在*第一章**中介绍了如何安装 GCC 和 Make，以及在*第三章**中介绍了手册页*，深入 Linux 中的 C*。你还需要我们在*第三章**中创建的通用 Makefile。将该文件放在你为本章编写代码的同一目录中。你将在 GitHub 文件夹中找到该文件的副本，以及我们在这里编写的所有其他源代码文件，网址为[`github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch4`](https://github.com/PacktPublishing/Linux-System-Programming-Techniques/tree/master/ch4)。

点击以下链接查看代码演示视频：[`bit.ly/39rJIdQ`](https://bit.ly/39rJIdQ)

# 为什么错误处理在系统编程中很重要

这个食谱是对错误处理是什么的一个简短介绍。我们还将看到一个常见错误的例子：*权限不足*。掌握这些基本技能将使你在长远的道路上成为一个更好的程序员。

## 准备工作

对于这个食谱，你只需要 GCC 编译器，最好是通过元包或组安装，就像我们在*第一章**中介绍的那样，获取必要的工具并编写我们的第一个 Linux 程序。确保在本食谱的源代码所在的同一目录中放置*技术要求*部分提到的 Makefile。

## 如何做…

按照以下步骤来探索一个常见错误以及如何处理它：

1.  首先，我们将编写没有任何`simple-touch-v1.c`的程序。该程序将创建一个用户指定的空文件作为参数。`PATH_MAX`宏对我们来说是新的。它包含我们在 Linux 系统上可以在路径中使用的最大字符数。它在`linux/limits.h`头文件中定义：

```
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <linux/limits.h>
int main(int argc, char *argv[])
{
   char filename[PATH_MAX] = { 0 };
   if (argc != 2)
   {
      fprintf(stderr, "You must supply a filename "
          "as an argument\n");
      return 1;
   }
   strncpy(filename, argv[1], PATH_MAX-1);
   creat(filename, 00644);
   return 0;
}
```

1.  编译程序：

```
$> make simple-touch-v1
gcc -Wall -Wextra -pedantic -std=c99    simple-touch-v1.c   -o simple-touch-v1
```

1.  现在，让我们尝试运行程序并看看会发生什么。如果我们不给它任何参数，它将打印一个错误消息并返回`1`。当我们给它一个不存在的文件时，它将以权限 644 创建它（我们将在下一章中介绍权限）：

```
$> ./simple-touch-v1 
You must supply a filename as an argument
$> ./simple-touch-v1 my-test-file
$> ls -l my-test-file 
-rw-r--r-- 1 jake jake 0 okt 12 22:46 my-test-file
```

1.  让我们看看如果我们尝试在我们的家目录之外创建一个文件会发生什么；也就是说，在一个我们没有写权限的目录：

```
$> ./simple-touch-v1 /abcd1234
```

1.  这似乎已经起作用，因为它没有抱怨，但实际上并没有。让我们尝试检查文件：

```
$> ls -l /abcd1234
ls: cannot access '/abcd1234': No such file or directory
```

1.  让我们重写文件，以便在`creat()`无法创建文件时向 stderr 打印错误消息——*无法创建文件*。为了实现这一点，我们将整个对`creat()`的调用包装在一个`if`语句中。将新版本命名为`simple-touch-v2.c`。与上一个版本的更改在这里突出显示：

```
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <linux/limits.h>
int main(int argc, char *argv[])
{
   char filename[PATH_MAX] = { 0 };
   if (argc != 2)
   {
      fprintf(stderr, "You must supply a filename "
         "as an argument\n");
      return 1;
   }
   strncpy(filename, argv[1], PATH_MAX-1);
   if ( creat(filename, 00644) == -1 )
   {
fprintf(stderr, "Can't create file %s\n", 
         filename);
      return 1;
   }
   return 0;
}
```

1.  编译新版本：

```
$> make simple-touch-v2
gcc -Wall -Wextra -pedantic -std=c99    simple-touch-v2.c   -o simple-touch-v2
```

1.  最后，让我们重新运行它，既使用我们可以创建的文件，又使用我们无法创建的文件。当我们尝试创建一个我们没有权限创建的文件时，我们将收到一个错误消息，指出*无法创建文件*：

```
$> ./simple-touch-v2 hello123
$> ./simple-touch-v2 /abcd1234
Couldn't create file /abcd1234
```

## 它是如何工作的...

在这个示例中，我们使用了一个系统调用`creat()`，它在文件系统上创建一个文件。该函数有两个参数：第一个是要创建的文件，第二个是新创建的文件应该具有的文件访问模式。在这种情况下，我们设置了文件的`644`，这是用户拥有文件的读写权限，对所有者的组和其他所有人的读权限。我们将在*第五章**，使用文件 I/O 和文件系统操作*中更深入地介绍文件访问模式。

如果它无法创建我们要求创建的文件，不会发生任何“坏事”。它只是将-1 返回给调用函数（在这种情况下是`main()`）。这意味着在我们的程序的第一个版本中，似乎一切都很顺利，文件已经创建了，而实际上并没有。作为程序员，我们需要捕获返回代码并对其进行操作。我们可以在函数的手册页`man 2 creat`中找到函数的返回值。

在程序的第二个版本中，我们添加了一个`if`语句来检查-1。如果函数返回-1，则会向 stderr 打印错误消息，并返回 1 给 shell。我们现在已经通知了用户和可能依赖于该程序来创建文件的任何程序。

获取函数的返回值是检查错误的最常见和最直接的方法。我们都应该养成这个习惯。一旦我们使用了某个函数，我们就应该检查它的返回值（当然，只要合理）。

# 处理一些常见的错误

在这个示例中，我们将看一些常见的错误，我们可以处理。知道要寻找哪些错误是掌握错误处理的第一步。如果警察不知道要寻找哪些犯罪，他们就无法抓到坏人。

我们将看看由于计算机资源限制、权限错误和数学错误可能发生的错误。但重要的是要记住，大多数函数在发生错误时会返回一个特殊值（通常是-1 或一些预定义的值）。当没有错误发生时，实际数据被返回。

我们还将简要涉及处理缓冲区溢出的主题。缓冲区溢出是一个值得一本书的广泛主题，但一些简短的例子可以帮助。

## 准备工作

在这个示例中，我们将编写更短的代码示例，并使用 GCC 和 Make 进行编译。我们还将阅读*POSIX 程序员手册*中的一些 man 页面。如果您使用的是 Debian 或 Ubuntu，您必须首先安装这些手册页面，我们在*第三章**，深入 Linux 中的 C*中的*获取有关 Linux 和 Unix 特定头文件的信息*部分中已经安装了这些手册页面。

## 如何做...

查找使用特定函数时最有可能发生的错误的最简单方法是阅读函数手册页的**返回值**部分。在这里，我们将看一些例子：

1.  大多数`creat()`、`open()`和`write()`。查看`errno`以获取更具体的信息。我们将在本章后面介绍`errno`。

1.  现在，查看幂函数`pow()`的手册页面。滚动到`pow()`函数返回计算的答案，如果出现错误，它不能返回 0 或-1；这可能是某些计算的答案。相反，定义了一些特殊的数字，称为`HUGE_VAL`、`HUGE_VALF`和`HUGE_VALL`。但是，在大多数系统中，这些被定义为无穷大。然而，我们仍然可以使用这些宏来进行测试，如下例所示。将文件命名为`huge-test.c`：

```
#include <stdio.h>
#include <math.h>
int main(void)
{
   int number = 9999;
   double answer;
   if ( (answer = pow(number, number)) == HUGE_VAL )
   {
      fprintf(stderr, "A huge value\n");
      return 1;
   }
   else
   {
      printf("%lf\n", answer);
   }
   return 0;
}
```

1.  编译程序并测试它。记得链接到`math`库使用`-lm`：

```
$> gcc -Wall -Wextra -pedantic -lm huge-test.c \
> -o huge-test
$> ./huge-test 
A huge value
```

1.  其他可能发生的错误，不会给我们返回值的大多是溢出错误。这在处理`strcat()`、`strncat()`、`strdup()`等函数时尤其如此。尽可能使用这些函数。编写以下程序并将其命名为`str-unsafe.c`：

```
#include <stdio.h>
#include <string.h>
int main(int argc, char *argv[])
{
    char buf[10] = { 0 };
    strcat(buf, argv[1]);
    printf("Text: %s\n", buf);
    return 0;
}
```

1.  现在，使用 Make（以及我们放在此目录中的 Makefile）编译它。请注意，我们将从编译器这里得到一个警告，因为我们没有使用`argc`变量。这个警告来自于 GCC 的`-Wextra`选项。然而，这只是一个警告，说明我们在代码中从未使用过`argc`，所以我们可以忽略这条消息。始终阅读警告消息；有时，事情可能更严重：

```
$> make str-unsafe
gcc -Wall -Wextra -pedantic -std=c99    str-unsafe.c   -o str-unsafe
str-unsafe.c: In function 'main':
str-unsafe.c:4:14: warning: unused parameter 'argc' [-Wunused-parameter]
 int main(int argc, char *argv[])
          ~~~~^~~~
```

1.  现在，用不同的输入长度来测试。如果我们根本不提供任何输入，或者提供太多的输入（超过 9 个字符），就会发生分段错误：

```
$> ./str-unsafe 
Segmentation fault
$> ./str-unsafe hello
Text: hello
$> ./str-unsafe "hello! how are you doing?"
Text: hello! how are you doing?
Segmentation fault
```

1.  让我们重写程序。首先，我们必须确保用户输入了一个参数；其次，我们必须用`strncat()`替换`strcat()`。将新版本命名为`str-safe.c`：

```
#include <stdio.h>
#include <string.h>
int main(int argc, char *argv[])
{
   if (argc != 2)
   {
      fprintf(stderr, "Supply exactly one "
         "argument\n");
      return 1;
   }
   char buf[10] = { 0 };
   strncat(buf, argv[1], sizeof(buf)-1);
   printf("Test: %s\n", buf);
   return 0;
}
```

1.  编译它。这次，我们不会收到关于`argc`的警告，因为我们在代码中使用了它：

```
$> make str-safe
gcc -Wall -Wextra -pedantic -std=c99    str-safe.c   -o str-safe
```

1.  让我们用不同的输入长度运行它。注意长文本在第九个字符处被截断，从而防止分段错误。还要注意，我们通过要求精确一个参数来处理空输入的分段错误：

```
$> ./str-safe 
Supply exactly one argument
$> ./str-safe hello
Text: hello
$> ./str-safe "hello, how are you doing?"
Text: hello, ho
$> ./str-safe asdfasdfasdfasdfasdfasdfasdfasdf
Text: asdfasdfa
```

## 它是如何工作的...

在*步骤 2*中，我们查看了一些手册页面，以了解在处理它们时可以期望处理什么样的错误。在这里，我们了解到大多数系统调用在出现错误时返回-1，并且大多数错误都与权限或系统资源有关。

在*步骤 2*和*3*中，我们看到数学函数在出现错误时可以返回特殊的数字（因为通常的数字——0、1 和-1——可能是计算的有效答案）。

在*步骤 4*至*9*中，我们简要涉及了处理用户输入和`strcat()`、`strcpy()`和`strdup()`是不安全的，因为它们复制它们得到的任何东西，即使目标缓冲区没有足够的空间。当我们给程序一个长于 10 个字符的字符串（实际上是九个，因为`NULL`字符占用一个位置）时，程序会崩溃并显示*分段错误*。

这些*str*函数有相应的带有*n*字符的函数名称；例如，`strncat()`。这些函数只会复制作为第三个参数给定的大小。在我们的示例中，我们将大小指定为`sizeof(buf)-1`，在我们的程序中为 9。我们使用比`buf`实际大小少一个的原因是为了为末尾的空终止字符(`\0`)腾出空间。最好使用`sizeof(buf)`而不是使用一个字面数。如果我们在这里使用了字面数 9，然后将缓冲区的大小更改为 5，我们很可能会忘记更新`strncat()`的数字。

# 错误处理和 errno

Linux 和其他类 UNIX 系统中的大多数系统调用函数设置了一个名为`errno`的特殊变量。

在这个示例中，我们将学习`errno`是什么，如何从中读取值，以及何时设置它。我们还将看到`errno`的一个示例用例。了解`errno`对于系统编程至关重要，主要是因为它与系统调用一起使用。

本章中接下来的几个食谱与本食谱密切相关。 在本食谱中，我们将学习关于`errno`；在接下来的三个食谱中，我们将学习如何解释我们从`errno`得到的错误代码并打印人类可读的错误消息。

## 准备工作

您将需要本食谱中与上一个食谱相同的组件；也就是说，我们已经安装的 GCC 编译器、Make 工具和*POSIX 程序员手册*。 如果没有，请参阅*第一章**，获取必要的工具并编写我们的第一个 Linux 程序*，以及*第三章**，深入了解 Linux 中的 C*中的*获取有关 Linux 和 UNIX 特定头文件的信息*部分。

## 如何做…

在这个食谱中，我们将继续构建本章第一个食谱中的`simple-touch-v2.c`。 在这里，我们将扩展它，以便在无法创建文件时打印一些更有用的信息：

1.  将以下代码写入文件并保存为`simple-touch-v3.c`。 在这个版本中，我们将使用`errno`变量来检查错误是否是由权限错误（`EACCES`）或其他未知错误引起的。 更改的代码已在这里突出显示：

```
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <errno.h>
#include <linux/limits.h>
int main(int argc, char *argv[])
{
   char filename[PATH_MAX] = { 0 };
   if (argc != 2)
   {
      fprintf(stderr, "You must supply a filename "
         "as an argument\n");
      return 1;
   }
   strncpy(filename, argv[1], sizeof(filename)-1);
   if ( creat(filename, 00644) == -1 )
   {
      fprintf(stderr, "Can't create file %s\n", 
         filename);
      if (errno == EACCES)
      {
         fprintf(stderr, "Permission denied\n");
      }
      else
      {
         fprintf(stderr, "Unknown error\n");
      }
      return 1;
   }
   return 0;
}
```

1.  让我们编译这个版本：

```
$> make simple-touch-v3
gcc -Wall -Wextra -pedantic -std=c99    simple-touch-v3.c   -o simple-touch-v3
```

1.  最后，让我们运行新版本。 这一次，程序会给我们更多关于出错原因的信息。 如果是权限错误，它会告诉我们。 否则，它会打印“未知错误”：

```
$> ./simple-touch-v3 asdf
$> ls -l asdf
-rw-r--r-- 1 jake jake 0 okt 13 23:30 asdf
$> ./simple-touch-v3 /asdf
Can't create file /asdf
Permission denied
$> ./simple-touch-v3 /non-existent-dir/hello
Can't create file /non-existent-dir/hello
Unknown error
```

## 它是如何工作的…

我们将注意到这个版本的第一个区别是我们现在包括一个名为`errno.h`的头文件。 如果我们希望使用`errno`变量和我们在新版本中使用的许多错误`EACCES`，则需要此文件。

下一个区别是，我们现在使用`sizeof(filename)-1`而不是`PATH_MAX-1`作为`strncpy()`的大小参数。 这是我们在上一个食谱中学到的东西。

然后，我们有`if (errno == EACCES)`行，它检查`errno`变量是否为`EACCES`。 我们可以在`man errno.h`和`man 2 creat`中阅读关于这些宏的信息，比如`EACCES`。 这个特定的宏意味着*权限被拒绝*。

当我们使用`errno`时，我们应该首先检查函数或系统调用的返回值，就像我们在这里使用`creat()`周围的`if`语句一样。 `errno`变量就像任何其他变量一样，这意味着在系统调用之后它不会被清除。 如果我们在检查函数的返回值之前直接检查`errno`，`errno`可能包含来自先前错误的错误代码。

在我们的`touch`版本中，我们只处理了这个特定的错误。 接下来，我们有一个`else`语句，它捕获所有其他错误并打印一个“未知错误”消息。

在*步骤 3*中，我们通过尝试在我们系统上不存在的目录中创建文件来生成了一个“未知错误”消息。 在下一个食谱中，我们将扩展我们的程序，以便它可以考虑更多的宏。

# 处理更多的 errno 宏

在本食谱中，我们将继续处理我们的`touch`版本中的更多`errno`宏。 在上一个食谱中，我们设法引发了一个“未知错误”消息，因为我们只处理了权限被拒绝的错误。 在这里，我们将找出到底是什么导致了那个错误以及它叫什么。 然后，我们将实现另一个`if`语句来处理它。 知道如何找到正确的`errno`宏将帮助您更深入地了解计算、Linux、系统调用和错误处理。

## 准备工作

再次，我们将查看手册页面，找到我们正在寻找的信息。 这个食谱所需的唯一东西是手册页面，GCC 编译器和 Make 工具。

## 如何做…

按照以下步骤完成本食谱：

1.  首先阅读`creat()`的手册页面，使用`man 2 creat`。 向下滚动到`ENOENT`（缩写为*错误无条目*）。

1.  让我们实现一个新的`if`语句来处理`ENOENT`。将新版本命名为`simple-touch-v4.c`。完整的程序如下。以下是与上一个版本的更改。还要注意，我们已经删除了突出显示的代码中一些`if`语句的括号：

```
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <errno.h>
#include <linux/limits.h>
int main(int argc, char *argv[])
{
   char filename[PATH_MAX] = { 0 };
   if (argc != 2)
   {
      fprintf(stderr, "You must supply a filename "
         "as an argument\n");
      return 1;
   }
   strncpy(filename, argv[1], sizeof(filename)-1);
   if ( creat(filename, 00644) == -1 )
   {
fprintf(stderr, "Can't create file %s\n", 
         filename);
      if (errno == EACCES)
         fprintf(stderr, "Permission denied\n");
      else if (errno == ENOENT)
         fprintf(stderr, "Parent directories does "
            "not exist\n");
      else
         fprintf(stderr, "Unknown error\n");
      return 1;
   }
   return 0;
}
```

1.  编译新版本：

```
$> make simple-touch-v4
gcc -Wall -Wextra -pedantic -std=c99    simple-touch-v4.c   -o simple-touch-v4
```

1.  让我们运行它并生成一些错误。这次，当目录不存在时，它将打印出一个错误消息：

```
$> ./simple-touch-v4 asdf123
$> ./simple-touch-v4 /hello
Can't create file /hello
Permission denied
$> ./simple-touch-v4 /non-existent/hello
Can't create file /non-existent/hello
Parent directories do not exist
```

## 它是如何工作的...

在这个版本中，我从内部的`if`、`else if`和`else`语句中删除了括号，以节省空间。如果每个`if`、`else if`和`else`下只有一个语句，这是有效的代码。然而，这样做可能是危险的，因为很容易出错。如果我们在其中一个`if`语句中写更多的语句，那些语句将不属于`if`语句，尽管看起来正确且没有错误编译。这种情况称为*误导性缩进*。缩进会让大脑误以为是正确的。

代码中的新内容是`else if (errno == ENOENT)`行及其下面的行。这是我们处理`ENOENT`错误宏的地方。

## 还有更多...

`man 2 syscalls`中列出的几乎所有函数都设置了`errno`变量。查看一些这些函数的手册页，并滚动到不同函数设置的`errno`宏。

还要阅读`man errno.h`，其中包含有关这些宏的有用信息。

# 使用 errno 和 strerror()

与查找每个可能的 errno 宏并弄清楚哪些适用以及它们的含义相比，使用一个名为`strerror()`的函数更容易。这个函数将`errno`代码转换为可读的消息。使用`strerror()`比自己实现所有内容要快得多。这样做也更安全，因为我们犯错的风险更小。每当有一个函数可以帮助我们减轻手动工作时，我们都应该使用它。

请注意，这个函数的目的是将`errno`宏转换为可读的错误消息。如果我们想以某种特定的方式处理特定的错误，我们仍然需要使用实际的`errno`值。

## 准备工作

上一个配方的要求也适用于这个配方。这意味着我们需要 GCC 编译器、Make 工具（以及 Makefile）和手册页。

## 如何做...

在这个配方中，我们将继续开发我们自己的`touch`版本。我们将从上一个版本继续。这次，我们将重写我们为不同的宏制作的`if`语句，并使用`strerror()`代替。让我们开始吧：

1.  编写以下代码，并将其保存为`simple-touch-v5.c`。注意，现在代码已经变得更小，因为我们用`strerror()`替换了`if`语句。这个版本更加清晰。以下是与上一个版本的更改：

```
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <errno.h>
#include <linux/limits.h>
int main(int argc, char *argv[])
{
   int errornum;
   char filename[PATH_MAX] = { 0 };
   if (argc != 2)
   {
      fprintf(stderr, "You must supply a filename "
         "as an argument\n");
      return 1;
   }
   strncpy(filename, argv[1], sizeof(filename)-1);
   if ( creat(filename, 00644) == -1 )
   {
      errornum = errno;
      fprintf(stderr, "Can't create file %s\n", 
         filename);
      fprintf(stderr, "%s\n", strerror(errornum));
      return 1;
   }
   return 0;
}
```

1.  编译这个新版本：

```
$> make simple-touch-v5
gcc -Wall -Wextra -pedantic -std=c99    simple-touch-v5.c   -o simple-touch-v5
```

1.  让我们试一下。注意程序现在打印出描述出了什么问题的错误消息。我们甚至不需要检查`errno`变量是否存在可能的错误：

```
$> ./simple-touch-v5 hello123 
$> ls hello123
hello123
$> ./simple-touch-v5 /asdf123
Can't create file /asdf123
Permission denied
$> ./simple-touch-v5 /asdf123/hello
Can't create file /asdf123/hello
No such file or directory
How it works…
```

所有的`if`、`else if`和`else`语句现在都被一行代码替换了：

```
fprintf(stderr, "%s\n", strerror(error));
```

我们还将`errno`的值保存在一个名为`errornum`的新变量中。我们这样做是因为在下一个发生错误时，`errno`中的值将被新的错误代码覆盖。为了防止在`errno`被覆盖时显示错误的消息，将其保存到一个新变量中更安全。

然后，我们使用存储在`errornum`中的错误代码作为`strerror()`的参数。这个函数将错误代码转换为可读的错误消息，并将该消息作为字符串返回。这样，我们就不必为可能发生的每个错误创建`if`语句。

在*步骤 3*中，我们看到`strerror()`如何将`EACCES`宏翻译成*Permission denied*，将`ENOENT`翻译成*No such file or directory*。

## 还有更多...

在`man 3 strerror`手册页面中，您会找到一个类似的函数，可以以用户首选的语言打印错误消息。

# 使用 perror()和 errno

在上一个步骤中，我们使用`strerror()`从`errno`获取包含人类可读错误消息的字符串。还有一个类似于`strerr()`的函数叫做`perror()`。它的名字代表**打印错误**，这就是它的作用；它直接将错误消息打印到*stderr*。

在这个步骤中，我们将编写我们的 simple touch 程序的第六个版本。这次，我们将用`perror()`替换两个`fprintf()`行。

## 准备工作

这个步骤所需的程序只有 GCC 编译器和 Make 工具（以及通用的 Makefile）。

## 如何做…

按照以下步骤创建一个更短更好的`simple-touch`版本：

1.  将以下代码写入文件并保存为`simple-touch-v6.c`。这次，程序更小了。我们删除了两个`fprintf()`语句，并用`perror()`替换了它们。与上一个版本相比的变化在这里突出显示：

```
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <errno.h>
#include <linux/limits.h>
int main(int argc, char *argv[])
{
   char filename[PATH_MAX] = { 0 };
   if (argc != 2)
   {
      fprintf(stderr, "You must supply a filename "
         "as an argument\n");
      return 1;
   }
   strncpy(filename, argv[1], sizeof(filename)-1);
   if ( creat(filename, 00644) == -1 )
   {
      perror("Can't create file");
      return 1;
   }
   return 0;
}
```

1.  使用 Make 编译它：

```
$> make simple-touch-v6
gcc -Wall -Wextra -pedantic -std=c99    simple-touch-v6.c   -o simple-touch-v6
```

1.  运行并观察错误消息输出的变化：

```
$> ./simple-touch-v6 abc123
$> ./simple-touch-v6 /asdf123
Can't create file: Permission denied
$> ./simple-touch-v6 /asdf123/hello
Can't create file: No such file or directory
How it works…
```

这次，我们用一行替换了两个`fprintf()`行：

```
perror("Can't create file");
```

`perror()`函数接受一个参数，一个包含描述或函数名称的字符串。在这种情况下，我选择给它一个通用的错误消息`无法创建文件`。当`perror()`打印错误消息时，它会抓取`errno`中的最后一个错误代码（注意我们没有指定任何错误代码变量），并在文本`无法创建文件`之后应用该错误消息。因此，我们不再需要`fprintf()`行。

尽管在调用`perror()`时没有明确说明`errno`，但它仍然使用它。如果发生另一个错误，那么下一次调用`perror()`将打印该错误消息。`perror()`函数总是打印*最后*的错误。

## 还有更多…

手册页面`man 3 perror`中有一些很好的提示。例如，包含导致错误的函数名称是个好主意。这样在用户报告错误时更容易调试程序。

# 返回错误值

尽管可读的错误消息很重要，但我们不能忘记返回一个指示错误的值给 shell。我们已经知道返回 0 表示一切正常，而返回其他值（大多数情况下是 1）表示发生了某种错误。然而，如果需要，我们可以返回更具体的值，以便依赖我们程序的其他程序可以读取这些数字。例如，我们实际上可以返回`errno`变量，因为它只是一个整数。我们已经看到的所有宏，如`EACCES`和`ENOENT`，都是整数（分别为`EACCES`和`ENOENT`的 13 和 2）。

在这个步骤中，我们将学习如何将`errno`数字返回给 shell，以提供更具体的信息。

## 准备工作

与上一个步骤提到的相同的程序集适用于这个步骤。

## 如何做…

在这个步骤中，我们将制作我们的`simple-touch`程序的第七个版本。让我们开始吧：

1.  在这个版本中，我们只会改变一个单独的行。打开`simple-touch-v6.c`，将`perror()`行下面的`return`语句改为`return errno;`。将新文件保存为`simple-touch-v7.c`。最新版本如下，突出显示了更改的行：

```
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <errno.h>
#include <linux/limits.h>
int main(int argc, char *argv[])
{
   char filename[PATH_MAX] = { 0 };
   if (argc != 2)
   {
      fprintf(stderr, "You must supply a filename "
         "as an argument\n");
      return 1;
   }
   strncpy(filename, argv[1], sizeof(filename)-1);
   if ( creat(filename, 00644) == -1 )
   {
      perror("Can't create file");
      return errno;
   }
   return 0;
}
```

1.  编译新版本：

```
$> make simple-touch-v7
gcc -Wall -Wextra -pedantic -std=c99    simple-touch-v7.c   -o simple-touch-v7
```

1.  运行并检查退出代码：

```
$> ./simple-touch-v7 asdf
$> echo $
0
$> ./simple-touch-v7 /asdf
Can't create file: Permission denied
$> echo $?
13
$> ./simple-touch-v7 /asdf/hello123
Can't create file: No such file or directory
$> echo $?
2
```

## 工作原理…

在`errno.h`中定义的错误宏是常规整数。因此，例如，如果我们返回`EACCES`，我们返回数字 13。因此，在发生错误时，首先在幕后设置了`errno`。然后，`perror()`使用存储在`errno`中的值打印人类可读的错误消息。最后，程序返回到 shell，并使用存储在`errno`中的整数指示其他程序出了什么问题。不过，我们应该对此略加小心，因为有一些保留的返回值。例如，在 shell 中，返回值`2`通常表示*Missuse of shell builtins*。但是，在`errno`中，返回值`2`表示*No such file or directory* (`ENOENT`)。这不应该给您带来太多麻烦，但以防万一请记住这一点。

## 还有更多...

有一个名为`errno`的小程序，可以打印所有宏及其整数。不过，默认情况下并未安装此工具。软件包的名称是`moreutils`。

安装完成后，您可以通过运行`errno -l`命令打印所有宏的列表，其中`l`选项代表*list*。

在*Debian*和*Ubuntu*中安装软件包，作为 root 用户输入`apt install moreutils`。

在*Fedora*中安装软件包，请以 root 身份使用`dnf install moreutils`。

在*CentOS*和*Red Hat*中，您必须首先使用`dnf install epel-release`添加`epel-release`存储库，然后以 root 身份使用`dnf install moreutils`安装软件包。在撰写本文时，关于 CentOS 8 存在一些关于`moreutils`的依赖性问题，因此可能无法正常工作。
