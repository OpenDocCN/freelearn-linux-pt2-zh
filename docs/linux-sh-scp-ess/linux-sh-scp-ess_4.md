# 第四章：模块化和调试

在现实世界中，当你编写代码时，你要么永远维护它，要么以后有人接管它并对其进行更改。非常重要的是，您编写一个质量良好的 shell 脚本，以便更容易进一步维护它。同样重要的是，shell 脚本没有错误，以便按预期完成工作。在生产系统上运行的脚本非常关键，因为脚本的任何错误或错误行为可能会造成轻微或重大的损害。为了解决这些关键问题，重要的是尽快解决问题。

在本章中，我们将看到如何编写模块化和可重用的代码，以便快速和无需任何麻烦地维护和更新我们的 shell 脚本应用程序。我们还将看到如何使用不同的调试技术快速轻松地解决 shell 脚本中的错误。我们将看到如何通过在脚本中提供命令行选项的支持为不同的任务提供不同的选择。了解如何在脚本中提供命令行完成甚至会增加使用脚本的便利性。

本章将详细介绍以下主题：

+   将你的脚本模块化

+   将命令行参数传递给脚本

+   调试您的脚本

+   命令完成

# 将你的脚本模块化

在编写 shell 脚本时，有一个阶段我们会觉得一个 shell 脚本文件变得太大，难以阅读和管理。为了避免这种情况发生在我们的 shell 脚本中，保持脚本模块化非常重要。

为了保持脚本的模块化和可维护性，您可以执行以下操作：

+   创建函数而不是一遍又一遍地写相同的代码

+   在一个单独的脚本中编写一组通用的函数和变量，然后源来使用它

我们已经看到如何在第三章 *有效脚本编写*中定义和使用函数。在这里，我们将看到如何将一个更大的脚本分成更小的 shell 脚本模块，然后通过源使用它们。换句话说，我们可以说在`bash`中创建库。

## 源到脚本文件

源是一个 shell 内置命令，它在当前 shell 环境中读取并执行脚本文件。如果一个脚本调用另一个脚本文件的源，那么该文件中可用的所有函数和变量将被加载以供调用脚本使用。

### 语法

使用源的语法如下：

`source <script filename> [arguments]`

或：

`. <script filename> [arguments]`

`脚本文件名`可以带有或不带有路径名。如果提供了绝对或相对路径，它将仅在该路径中查找。否则，将在`PATH`变量中指定的目录中搜索文件名。

`arguments`被视为脚本文件名的位置参数。

`source`命令的退出状态将是在脚本文件中执行的最后一个命令的退出代码。如果脚本文件不存在或没有权限，则退出状态将为`1`。

### 创建一个 shell 脚本库

库提供了一个功能集合，可以被另一个应用程序重用，而无需从头开始重写。我们可以通过将我们的函数和变量放入一个 shell 脚本文件中来创建一个 shell 库，以便重用。

以下的`shell_library.sh`脚本是一个 shell 库的例子：

```
#!/bin/bash
# Filename: shell_library.sh
# Description: Demonstrating creation of library in shell

# Declare global variables
declare is_regular_file
declare is_directory_file

# Function to check file type
function file_type()
{
  is_regular_file=0
  is_directory_file=0
  if [ -f $1 ]
  then
    is_regular_file=1
  elif [ -d $1 ]
  then
    is_directory_file=1
  fi
}

# Printing regular file detail
function print_file_details()
{
   echo "Filename - $1"
   echo "Line count - `cat $1 | wc -l`"
   echo "Size - `du -h $1 | cut -f1`"
   echo "Owner - `ls -l $1 | tr -s ' '|cut -d ' ' -f3`"
   echo "Last modified date - `ls -l $1 | tr -s ' '|cut -d ' ' -f6,7`"
}

# Printing directory details
function print_directory_details()
{
   echo "Directory Name - $1"
   echo "File Count in directory - `ls $1|wc -l`"
   echo "Owner - `ls -ld $1 | tr -s ' '|cut -d ' ' -f3`"
   echo "Last modified date - `ls -ld $1 | tr -s ' '|cut -d ' ' -f6,7`"
}
```

前面的`shell_library.sh` shell 脚本包含了`is_regular_file`和`is_directory_file`全局变量，可以在调用`file_type()`函数后用于知道给定的文件是普通文件还是目录。此外，根据文件的类型，可以打印有用的详细信息。

### 加载一个 shell 脚本库

创建 shell 库是没有用的，除非它在另一个 shell 脚本中使用。我们可以直接在 shell 中使用 shell 脚本库，也可以在另一个脚本文件中使用。要加载 shell 脚本库，我们将使用 source 命令或.（句点字符），然后是 shell 脚本库。

#### 在 bash 中调用 shell 库

要在 shell 中使用`shell_library.sh`脚本文件，我们可以这样做：

```
$ source  shell_library.sh

```

或：

```
$ . shell_library.sh

```

调用它们中的任何一个将使函数和变量可用于当前 shell 中使用：

```
$ file_type /usr/bin
$ echo $is_directory_file
1
$ echo $is_regular_file
0
$ if [ $is_directory_file -eq 1 ]; then print_directory_details /usr/bin; fi
Directory Name - /usr/bin
File Count in directory - 2336
Owner - root
Last modified date - Jul 12
```

当执行`file_type /usr/bin`命令时，将调用带有`/usr/bin`参数的`file_type()`函数。结果是，全局变量`is_directory_file`或`is_regular_file`将设置为`1`（`true`），取决于`/usr/bin`路径的类型。使用 shell 的`if`条件，我们测试`is_directory_file`变量是否设置为`1`。如果设置为`1`，则调用`print_directory_details()`函数，参数为`/usr/bin`，以打印其详细信息。

#### 在另一个 shell 脚本中调用 shell 库

以下示例解释了在 shell 脚本文件中使用 shell 库的用法：

```
#!/bin/bash
# Filename: shell_library_usage.sh
# Description: Demonstrating shell library usage in shell script

# Print details of all files/directories in a directory
echo "Enter path of directory"
read dir

# Loading shell_library.sh module
. $PWD/shell_library.sh

# Check if entered pathname is a directory
# If directory, then print files/directories details inside it
file_type $dir
if [ $is_directory_file -eq 1 ]
then
   pushd $dir > /dev/null       # Save current directory and cd to $dir
   for file in `ls`
   do
     file_type $file
     if [ $is_directory_file -eq 1 ]
     then
       print_directory_details $file
       echo
     elif [ $is_regular_file -eq 1 ]
     then
       print_file_details $file
       echo
     fi
   done
fi
```

在运行`shell_library_usage.sh`脚本后，得到以下输出：

```
$ sh  shell_library_usage.sh	# Few outputs from /usr directory
Enter path of directory
/usr
Directory Name - bin
File Count in directory - 2336
Owner - root
Last modified date - Jul 12

Directory Name - games
File Count in directory - 0
Owner - root
Last modified date - Aug 16

Directory Name - include
File Count in directory - 172
Owner - root
Last modified date - Jul 12

Directory Name - lib
File Count in directory - 603
Owner - root
Last modified date - Jul 12

Directory Name - lib64
File Count in directory - 3380
Owner - root
Last modified date - Jul 12

Directory Name - libexec
File Count in directory - 170
Owner - root
Last modified date - Jul 7
```

### 注意

要加载 shell 脚本库，使用`source`或`.`，然后是`script_filename`。

`source`和`.`（句点字符）都在当前 shell 中执行脚本。`./script`与`. script`不同，因为`./script`在子 shell 中执行脚本，而`. script`在调用它的 shell 中执行。

# 将命令行参数传递给脚本

到目前为止，我们已经看到了诸如`grep`、`head`、`ls`、`cat`等命令的用法。这些命令还支持通过命令行传递参数给命令。一些命令行参数是输入文件、输出文件和选项。根据输出的需要提供参数。例如，执行`ls -l filename`以获得长列表输出，而使用`ls -R filename`用于递归显示目录的内容。

Shell 脚本还支持提供命令行参数，我们可以通过 shell 脚本进一步处理。

命令行参数可以如下给出：

```
<script_file> arg1 arg2 arg3 … argN

```

这里，`script_file`是要执行的 shell 脚本文件，`arg1`、`arg2`、`arg3`、`argN`等是命令行参数。

## 在脚本中读取参数

命令行参数作为位置参数传递给 shell 脚本。因此，`arg1`在脚本中将被访问为`$1`，`arg2`为`$2`，依此类推。

以下 shell 演示了命令行参数的用法：

```
#!/bin/bash
# Filename: command_line_arg.sh
# Description: Accessing command line parameters in shell script

# Printing first, second and third command line parameters"
echo "First command line parameter = $1"
echo "Second command line parameter = $2"
echo "Third command line parameter = $3" 
```

在带有参数运行`command_line_arg.sh`脚本后，得到以下输出：

```
$  sh command_line_arg.sh Linux Shell Scripting
First command line parameter = Linux
Second command line parameter = Shell
Third command line parameter = Scripting
```

以下表格显示了有用的特殊变量，用于获取有关命令行参数的更多信息：

| 特殊变量 | 描述 |
| --- | --- |
| `$#` | 命令行参数的数量 |
| `$*` | 以单个字符串的形式包含所有命令行参数的完整集合，即`'$1 $2 … $n'` |
| `$@` | 完整的命令行参数集合，但每个参数都用单独的引号括起来，即`'$1' '$2' … '$n'` |
| `$0` | shell 脚本本身的名称 |
| `$1, $1, … $N` | 分别指代参数 1、参数 2、…、参数 N |

在脚本中使用`$#`来检查命令行参数的数量将非常有助于进一步处理参数。

以下是另一个接受命令行参数的 shell 脚本示例：

```
#!/bin/bash
# Filename: command_line_arg2.sh
# Description: Creating directories in /tmp

# Check if at least 1 argument is passed in command line
if [ $# -lt 1 ]
then
  echo "Specify minimum one argument to create directory"
  exit 1
else
  pushd /tmp > /dev/null
  echo "Directory to be created are: $@"
  mkdir $@      # Accessing all command line arguments
fi
```

在执行`command_line_arg2.sh`脚本后，得到以下输出：

```
$  sh command_line_arg2.sh a b
Directory to be created are: a b
$  sh command_line_arg2.sh
Specify minimum one argument to create directory

```

## 移动命令行参数

要将命令行参数向左移动，可以使用`shift`内置命令。语法如下：

`shift N`

这里，`N`是它可以向左移动的参数个数。

例如，假设当前的命令行参数是`arg1`，`arg2`，`arg3`，`arg4`和`arg5`。它们可以在 shell 脚本中分别作为`$1`，`$2`，`$3`，`$4`和`$5`访问；`$#`的值为`5`。当我们调用`shift 3`时，参数会被移动`3`个位置。现在，`$1`包含`arg4`，`$2`包含`arg5`。此外，`$#`的值现在是`2`。

以下 shell 脚本演示了`shift`的用法：

```
#!/bin/bash
# Filename: shift_argument.sh
# Description: Usage of shift shell builtin

echo "Length of command line arguments = $#"
echo "Arguments are:"
echo "\$1 = $1, \$2 = $2, \$3 = $3, \$4 = $4, \$5 = $5, \$6 = $6"
echo "Shifting arguments by 3"
shift 3
echo "Length of command line arguments after 3 shift = $#"
echo "Arguments after 3 shifts are"
echo "\$1 = $1, \$2 = $2, \$3 = $3, \$4 = $4, \$5 = $5, \$6 = $6"
```

使用参数`a b c d e f`运行`shift_argument.sh`脚本后获得以下输出：

```
$ sh shift_argument.sh a b c d e f
Length of command line arguments = 6
Arguments are:
$1 = a, $2 = b, $3 = c, $4 = d, $5 = e, $6 = f
Shifting arguments by 3
Length of command line arguments after 3 shift = 3
Arguments after 3 shifts are
$1 = d, $2 = e, $3 = f, $4 = , $5 = , $6 = 

```

## 在脚本中处理命令行选项

提供命令行选项使 shell 脚本更具交互性。从命令行参数中，我们还可以解析选项以供 shell 脚本进一步处理。

以下 shell 脚本显示了带有选项的命令行用法：

```
#!/bin/bash
# Filename: myprint.sh
# Description: Showing how to create command line options in shell script

function display_help()
{
  echo "Usage: myprint [OPTIONS] [arg ...]"
  echo "--help  Display help"
  echo "--version       Display version of script"
  echo  "--print        Print arguments"
}

function display_version()
{
  echo "Version of shell script application is 0.1"
}

function myprint()
{
  echo "Arguments are: $*"
}

# Parsing command line arguments

if [ "$1" != "" ]
then
   case $1 in
        --help ) 
             display_help
             exit 1
            ;;
        --version )
             display_version
             exit 1
             ;;
        --print )
             shift
             myprint $@
             exit 1
            ;;
    *)
    display_help
    exit 1
   esac
fi
```

执行`myprint.sh`脚本后获得以下输出：

```
$ sh myprint.sh --help
Usage: myprint [OPTIONS] [arg ...]
--help      Display help
--version     Display version of script
--print         Print arguments
$ sh myprint.sh --version
Version of shell script application is 0.1
$ sh myprint.sh --print Linux Shell Scripting
Arguments are: Linux Shell Scripting
```

# 调试您的脚本

我们编写不同的 shell 脚本来执行不同的任务。在执行 shell 脚本时，您是否曾遇到过任何错误？答案很可能是肯定的！这是可以预料的，因为几乎不可能总是编写完美的 shell 脚本，没有错误或漏洞。

例如，以下 shell 脚本在执行时是有错误的：

```
#!/bin/bash
# Filename: buggy_script.sh
# Description: Demonstrating a buggy script

a=12 b=8
if [ a -gt $b ]
then
  echo "a is greater than b"
else
  echo "b is greater than a"
fi
```

执行`buggy_script.sh`后获得以下输出：

```
$ sh buggy_script.sh 
buggy_script.sh: line 6: [: a: integer expression expected
b is greater than a

```

从输出中，我们看到错误`[: a: integer expression expected`发生在第 6 行。仅仅通过查看错误消息，通常不可能知道错误的原因，特别是第一次看到错误时。此外，在处理冗长的 shell 脚本时，手动查看代码并纠正错误是困难的。

为了克服在解决 shell 脚本中的错误或漏洞时遇到的各种麻烦，最好调试代码。调试 shell 脚本的方法如下：

+   在脚本的预期错误区域使用`echo`打印变量或要执行的命令的内容。

+   在运行脚本时使用`-x`调试整个脚本

+   使用 set 内置命令在脚本内部使用`-x`和`+x`选项调试脚本的一部分

## 使用 echo 进行调试

`echo`命令非常有用，因为它打印提供给它的任何参数。当我们在执行脚本时遇到错误时，我们知道带有错误消息的行号。在这种情况下，我们可以使用`echo`在实际执行之前打印将要执行的内容。

在我们之前的例子`buggy_script.sh`中，我们在第 6 行得到了一个错误——即`if [ a -gt $b ]`——在执行时。我们可以使用`echo`语句打印实际将在第 6 行执行的内容。以下 shell 脚本在第 6 行添加了`echo`，以查看最终将在第 6 行执行的内容：

```
#!/bin/bash
# Filename: debugging_using_echo.sh
# Description: Debugging using echo

a=12 b=8
echo "if [ a -gt $b ]"
exit
if [ a -gt $b ]
then
  echo "a is greater than b"
else
  echo "b is greater than a"
fi
```

我们现在将按以下方式执行`debugging_using_echo.sh`脚本：

```
$ sh debugging_using_echo.sh
if [ a -gt 8 ]

```

我们可以看到字符`a`正在与`8`进行比较，而我们期望的是变量`a`的值。这意味着我们错误地忘记了在`a`中使用`$`来提取变量`a`的值。

## 使用-x 调试整个脚本

使用`echo`进行调试很容易，如果脚本很小，或者我们知道问题出在哪里。使用`echo`的另一个缺点是，每次我们进行更改，都必须打开一个 shell 脚本，并相应地修改`echo`命令。调试后，我们必须记住删除为调试目的添加的额外`echo`行。

为了克服这些问题，bash 提供了`-x`选项，可以在执行 shell 脚本时使用。使用`-x`选项运行脚本会以调试模式运行脚本。这会打印所有要执行的命令以及脚本的输出。

以以下 shell 脚本为例：

```
#!/bin/bash
# Filename : debug_entire_script.sh
# Description: Debugging entire shell script using -x

# Creating diretcories in /tmp
dir1=/tmp/$1
dir2=/tmp/$2
mkdir $dir1 $dir2
ls -ld $dir1
ls -ld $dir2
rmdir $dir1
rmdir $dir2
```

现在，我们将按以下方式运行前述脚本：

```
$ sh debug_entire_script.sh pkg1
mkdir: cannot create directory '/tmp/': File exists
drwxrwxr-x. 2 skumari skumari 40 Jul 14 01:47 /tmp/pkg1
drwxrwxrwt. 23 root root 640 Jul 14 01:47 /tmp/
rmdir: failed to remove '/tmp/': Permission denied

```

它会给出`/tmp/`目录已经存在的错误。通过查看错误，我们无法知道为什么它要创建`/tmp`目录。为了跟踪整个代码，我们可以使用带有`-x`选项运行`debug_entire_script.sh`脚本：

```
$ sh -x debug_entire_script.sh pkg1
+ dir1=/tmp/pkg1
+ dir2=/tmp/
+ mkdir /tmp/pkg1 /tmp/
mkdir: cannot create directory '/tmp/': File exists
+ ls -ld /tmp/pkg1
drwxrwxr-x. 2 skumari skumari 40 Jul 14 01:47 /tmp/pkg1
+ ls -ld /tmp/
drwxrwxrwt. 23 root root 640 Jul 14 01:47 /tmp/
+ rmdir /tmp/pkg1
+ rmdir /tmp/
rmdir: failed to remove '/tmp/': Permission denied

```

我们可以看到`dir2`是`/tmp/`。这意味着没有输入来创建第二个目录。

使用`-v`选项以及`-x`使得调试更加详细，因为`-v`会显示输入行：

```
$ sh -xv debug_entire_script.sh pkg1
#!/bin/bash
# Filename : debug_entire_script.sh
# Description: Debugging entire shell script using -x

# Creating diretcories in /tmp
dir1=/tmp/$1
+ dir1=/tmp/pkg1
dir2=/tmp/$2
+ dir2=/tmp/
mkdir $dir1 $dir2
+ mkdir /tmp/pkg1 /tmp/
mkdir: cannot create directory '/tmp/': File exists
ls -ld $dir1
+ ls -ld /tmp/pkg1
drwxrwxr-x. 2 skumari skumari 40 Jul 14 01:47 /tmp/pkg1
ls -ld $dir2
+ ls -ld /tmp/
drwxrwxrwt. 23 root root 640 Jul 14 01:47 /tmp/
rmdir $dir1
+ rmdir /tmp/pkg1
rmdir $dir2
+ rmdir /tmp/
rmdir: failed to remove '/tmp/': Permission denied
```

通过详细输出，很明显`dir1`和`dir2`变量期望从命令行参数中提供两个参数。因此，必须从命令行提供两个参数：

```
$  sh  debug_entire_script.sh pkg1 pkg2
drwxrwxr-x. 2 skumari skumari 40 Jul 14 01:50 /tmp/pkg1
drwxrwxr-x. 2 skumari skumari 40 Jul 14 01:50 /tmp/pkg2

```

现在，脚本可以正常运行而不会出现任何错误。

### 注意

不再需要从命令行传递`-xv`选项给 bash，我们可以在脚本文件的`shebang`行中添加它，即`#!/bin/bash -xv`。

## 使用设置选项调试脚本的部分

调试 shell 脚本时，并不总是需要一直调试整个脚本。有时，调试部分脚本更有用且节省时间。我们可以使用`set`内置命令在 shell 脚本中实现部分调试：

```
set -x  (Start debugging from here)
set +x  (End debugging here)
```

我们可以在 shell 脚本的多个位置使用`set +x`和`set -x`，具体取决于需要。当执行脚本时，它们之间的命令将与输出一起打印出来。

考虑以下 shell 脚本作为示例：

```
#!/bin/bash
# Filename: eval.sh
# Description: Evaluating arithmetic expression

a=23
b=6
expr $a + $b
expr $a - $b
expr $a * $b
```

执行此脚本会得到以下输出：

```
$ sh eval.sh
29
17
expr: syntax error
```

我们得到了一个语法错误，最有可能是第三个表达式，即`expr $a * $b`。

为了调试，在`expr $a * $b`之前使用`set -x`，之后使用`set +x`。

另一个带有部分调试的脚本`partial_debugging.sh`如下：

```
#!/bin/bash
# Filename: partial_debugging.sh
# Description: Debugging part of script of eval.sh

a=23
b=6
expr $a + $b

expr $a - $b

set -x
expr $a * $b
set +x
```

执行`partial_debugging.sh`脚本后得到以下输出：

```
$  sh partial_debugging.sh
29
17
+ expr 23 eval.sh partial_debugging.sh 6
expr: syntax error
+ set +x
```

从前面的输出中，我们可以看到`expr $a * $b`被执行为`expr 23 eval.sh partial_debugging.sh 6`。这意味着，bash 在执行乘法时，扩展了`*`作为当前目录中的任何内容的行为。因此，我们需要转义字符`*`的行为，以防止其被扩展，即`expr $a \* $b`。

脚本`eval_modified.sh`是`eval.sh`脚本的修改版本：

```
#!/bin/bash
# Filename: eval_modified.sh
# Description: Evaluating arithmetic expression

a=23
b=6
expr $a + $b
expr $a - $b
expr $a \* $b
```

现在，运行`eval_modified.sh`的输出将如下所示：

```
$  sh eval_modified.sh 
29
17
138
```

脚本现在可以完美运行而不会出现任何错误。

除了我们在调试中学到的内容，您还可以使用`bashdb`调试器来更好地调试 shell 脚本。`bashdb`的源代码和文档可以在[`bashdb.sourceforge.net/`](http://bashdb.sourceforge.net/)找到。

# 命令完成

在命令行上工作时，每个人都必须执行一些常见任务，比如输入命令、选项、输入/输出文件路径和其他参数。有时，由于命令名称中的拼写错误，我们会写错命令名称。此外，输入一个很长的文件路径将很难记住。例如，如果我们想要递归查看路径为`/dir1/dir2/dir3/dir4/dir5/dir6`的目录的内容，我们将不得不运行以下命令：

```
$ ls -R /dir1/dir2/dir3/dir4/dir5/dir6

```

我们可以看到这个目录的路径非常长，很容易在输入完整路径时出错。由于这些问题，使用命令行将花费比预期更长的时间。

为了解决所有这些问题，shell 支持一个非常好的功能，称为命令完成。除了其他 shell 外，bash 也非常好地支持命令完成。

大多数 Linux 发行版，例如 Fedora、Ubuntu、Debian 和 CentOS，都预先安装了核心命令的 bash 完成。如果没有可用，可以使用相应的发行版软件包管理器下载，软件包名称为`bash-completion`。

shell 中的命令完成允许您自动完成部分输入的命令的其余字符，提供与给定命令相关的可能选项。它还建议并自动完成部分输入的文件路径。

要在 bash 中启用自动完成功能，使用*Tab*键。在输入命令时，如果单个命令匹配，单个`TAB`将自动完成命令，双[TAB]将列出所有以部分输入的命令开头的可能命令。

例如：

```
$ gr[TAB]      # Nothing happens
$ gre[TAB]      # Autocompletes to grep
$ grep[TAB][TAB]  # Lists commands installed in system and starts with grep
grep            grep-changelog  grepdiff 

```

现在，假设我们想要查看`/usr/share/man/`目录的内容，我们将不得不输入`ls /usr/share/man/`。使用 bash 完成，输入以下命令：

```
$ ls /u[TAB]/sh[TAB]/man

```

Bash 完成将自动完成缺少的部分路径，命令将变为：

```
$ ls /usr/share/man

```

## 使用 complete 管理 bash 完成

`complete`是一个内置的 shell，可用于查看系统中可用命令的 bash 完成规范。它还用于修改、删除和创建 bash 完成。

### 查看现有的 bash 完成

要了解现有的 bash 完成，请使用`complete`命令，带有或不带`-p`选项：

```
$ complete -p

```

以下是前述命令的一些输出：

```
complete cat  # No completion output
complete -F _longopt grep  # Completion as files from current directory
complete -d pushd  # Completion as directories from current directory
complete -c which  # Completion as list of all available commands

```

要在这些命令上看到 bash 完成，输入以下命令：

这将列出所有文件/目录，包括隐藏的文件/目录：

```
$ grep [TAB][TAB]

```

这将列出所有文件/目录，包括隐藏的文件/目录：

$ 猫[TAB][TAB]

这尝试列出系统中所有可用的命令。按下*y*将显示命令，按下*n*将不显示任何内容。

```
$ complete -c which [TAB][TAB]
 Display all 3205 possibilities? (y or n)

```

### 修改默认的 bash 完成行为

我们还可以使用 complete shell 内置命令修改给定命令的现有 bash 完成行为。

以下命令用于更改`which`命令的行为，不显示任何选项：

```
$ complete which
$ which [TAB][TAB]  # No auto completion option will be shown

```

以下命令用于更改`ls`命令的标签行为，仅显示目录列表作为 bash 完成：

```
$ ls ~/[TAB][TAB]    # Displays directories and file as  auto-completion
file1.sh file2.txt dir1/ dir2/ dir3/
$ complete -d ls
$ ls ~/[TAB][TAB]    # Displays only directory name as  auto-completion
dir1/ dir2/ dir3/

```

### 删除 bash 完成规范

我们可以使用 shell 内置的`complete`命令和`-r`选项删除命令的 bash 完成规范。

语法如下：

```
complete -r command_name

```

将以下内容视为示例：

```
$ complete | grep which  # Viewing bash completion specification for which
complete -c which
$ complete -r which     # Removed bash completion specification for which
$ complete | grep which  # No output

```

如果没有给出`command_name`作为`complete -r`的参数，所有完成规范都将被删除：

```
$ complete -r
$ complete

```

## 为自己的应用程序编写 bash 完成

bash-completion 包不为任何外部工具提供自动完成功能。假设我们想创建一个具有多个选项和参数的工具。要为其选项添加 bash 完成功能，我们必须创建自己的 bash 完成文件并将其源化。

例如，软件包管理器如`dnf`和`apt-get`都有自己的 bash 完成文件，以支持其选项的自动完成：

```
$ dnf up[TAB][TAB]
update      updateinfo  update-to   upgrade     upgrade-to 
$ apt-get up[TAB][TAB]
update upgrade

```

将以下 shell 脚本视为示例：

```
#!/bin/bash
# Filename: bash_completion_example.sh
# Description: Example demonstrating bash completion feature for command options

function help()
{
  echo "Usage: print [OPTIONS] [arg ...]"
  echo "-h|--help    Display help"
  echo "-v|--version Display version of script"
  echo "-p|--print     Print arguments"
}

function version()
{
  echo "Version of shell script application is 0.1"
}

function print()
{
  echo "Arguments are: $*"
}

# Parsing command line arguments

while [ "$1" != "" ]
do
   case $1 in
        -h | --help ) 
             help
             exit 1
            ;;
        -v | --version )
             version
             exit 1
             ;;
        -p | --print )
             shift
             print $@
             exit 1
            ;;
    *)
    help
    exit 1
   esac
done
```

要了解`bash_completion_example.sh`中支持的选项，我们将运行`--help`选项：

```
$ chmod +x bash_completion_example.sh	# Adding execute permission to script
$ ./bash_completion_example.sh --help
Usage: print [OPTIONS] [arg ...]
-h|--help    Display help
-v|--version Display version of script
-p|--print     Print arguments

```

所以，支持的选项是`-h`，`--help`，`-v`，`--version`，`-p`和`--print`。

要编写 bash 完成，需要以下 bash 内部变量的信息：

| Bash 变量 | 描述 |
| --- | --- |
| `COMP_WORDS` | 在命令行上键入的单词数组 |
| `COMP_CWORD` | 包含当前光标位置的单词的索引。 |
| `COMPREPLY` | 一个数组，它保存在按下[TAB][TAB]后显示的完成结果 |

`compgen`是一个内置的 shell 命令，根据选项显示可能的完成。它用于在 shell 函数中生成可能的完成。

### bash 完成的示例

我们的 shell 脚本`bash_completion_example`的 bash 完成文件将如下所示：

```
# Filename: bash_completion_example
# Description: Bash completion for bash_completion_example.sh

_bash_completion_example()
{
    # Declaring local variables
    local cur prev opts
    # An array variable storing the possible completions
    COMPREPLY=()
    # Save current word typed on command line in  cur variable
    cur="${COMP_WORDS[COMP_CWORD]}"
    # Saving previous word typed on command line in prev variable
    prev="${COMP_WORDS[COMP_CWORD-1]}"
    # Save all options provided by application in variable opts
    opts="-h -v -p --help --verbose --print"

    # Checking "${cur} == -*" means that perform completion only if current
    # word starts with a dash (-), which suggest that user is trying to complete an option.
    # Variable COMPREPLY contains the match of the current word "${cur}" against the list
    if [[ ${cur} == -* ]] ; then
        COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
        return 0
    fi
}

# Register _bash_completion_example to provide completion
# on running script bash_completion_example.sh
complete -F _bash_completion_example ./bash_completion_example.sh
```

根据惯例，bash 完成函数名称应以下划线(_)开头，后跟应用程序的名称，即`_bash_completion_example`。此外，我们将 bash 变量`COMPREPLY`重置为清除任何先前遗留的数据。然后，我们声明并设置`cur`变量为命令行的当前单词，`prev`变量为命令行中的前一个单词。另一个变量`opts`被声明并初始化为应用程序识别的所有选项；在我们的情况下，它们是`-h -v -p --help --verbose –print`。条件`if [[ ${cur} == -* ]]`检查当前单词是否等于`-*`，因为我们的选项以`-`开头，后跟任何其他字符。如果为`true`，则使用`compgen` shell 内置和`-W`选项显示所有匹配的选项。

### 运行创建的 bash 完成。

为了运行创建的 bash 完成，最简单的方法是将其源到`source bash_completion_example shell script`，然后运行脚本或命令：

```
$ source ./bash_completion_example
Now,  execute shell script:
$ ./bash_completion_example.sh -[TAB][TAB]
-h         --help     -p         --print    -v         --verbose
$ ./bash_completion_example.sh --[TAB][TAB]
--help     --print    --verbose
$  ./bash_completion_example.sh –-p[TAB]

```

在这里，`--p[TAB]`会自动完成为`-–print`。

# 总结

阅读完本章后，你现在应该能够编写一个易于维护和修改的 shell 脚本。现在，你知道如何在自己的脚本中使用现有的 shell 脚本库，使用`source`命令。你还熟悉了使用不同的调试技术来修复 shell 脚本中的错误和 bug。你还应该知道如何通过接受命令行参数并为其提供 bash 完成功能来编写脚本。

在下一章中，我们将看到如何查看、更改、创建和删除环境变量，以满足运行我们的应用程序的要求。
