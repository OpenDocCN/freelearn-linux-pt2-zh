存档和压缩文件

在本章中，您将学习如何将一组文件放在一个单独的存档中。您还将学习如何使用各种压缩方法压缩存档文件。

# 第十九章：创建存档

让我们为`/home/elliot`目录中的所有 bash 脚本创建一个备份。作为`root`用户，在`/root`中创建一个名为`backup`的目录：

```
root@ubuntu-linux:~# mkdir /root/backup
```

要创建存档，我们使用磁带存档命令`tar`。创建存档的一般语法如下：

```
tar -cf archive_name files
```

`-c`选项是`--create`的简写，用于创建存档。`-f`选项是`--file`的简写，用于指定存档名称。

现在让我们在`/root/backup`中为`/home/elliot`中的所有 bash 脚本创建一个名为`scripts.tar`的存档。为此，我们首先切换到`/home/elliot`目录：

```
root@ubuntu-linux:~# cd /home/elliot 
root@ubuntu-linux:/home/elliot#
```

然后我们运行命令：

```
root@ubuntu-linux:/home/elliot# tar -cf /root/backup/scripts.tar *.sh
```

这将在`/root/backup`中创建存档文件`scripts.tar`，并且不会有命令输出：

```
root@ubuntu-linux:/home/elliot# ls -l /root/backup/scripts.tar
-rw-r--r-- 1 root root 20480 Nov 1 23:12 /root/backup/scripts.tar
```

我们还可以添加`-v`选项以查看正在存档的文件：

```
root@ubuntu-linux:/home/elliot# tar -cvf /root/backup/scripts.tar *.sh 
3x10.sh
detect.sh 
empty.sh 
filetype.sh 
fun1.sh 
game.sh 
hello20.sh 
hello2.sh 
hello3.sh 
hello.sh 
math.sh 
mydate.sh 
noweb.sh 
numbers.sh 
rename.sh 
size2.sh 
size3.sh 
size.sh
```

# 查看存档内容

您可能想要查看存档的内容。为此，您可以使用`-t`选项以及后跟存档名称的`-f`选项：

```
tar -tf archive
```

例如，要查看我们刚刚创建的`scripts.tar`存档的内容，可以运行以下命令：

```
root@ubuntu-linux:/home/elliot# tar -tf /root/backup/scripts.tar 
3x10.sh
detect.sh 
empty.sh 
filetype.sh 
fun1.sh 
game.sh 
hello20.sh 
hello2.sh 
hello3.sh 
hello.sh 
math.sh 
mydate.sh 
noweb.sh 
numbers.sh 
rename.sh 
size2.sh 
size3.sh 
size.sh
```

如您所见，它列出了`scripts.tar`存档中的所有文件。

# 提取存档文件

您可能还想从存档中提取文件。为了演示，让我们在`/root`中创建一个名为`myscripts`的目录：

```
root@ubuntu-linux:/# mkdir /root/myscripts
```

要从存档中提取文件，我们使用`-x`选项以及后跟存档名称的`-f`选项。然后，我们使用`-C`选项，后跟目标目录，如下所示：

```
tar -xf archive -C destination
```

因此，要将`scripts.tar`存档中的所有文件提取到`/root/myscripts`目录中，您可以运行以下命令：

```
root@ubuntu-linux:/# tar -xf /root/backup/scripts.tar -C /root/myscripts
```

`-x`选项是`--extract`的简写，用于从存档中提取文件。我们还使用了`-C`选项，它在执行任何操作之前基本上会切换到`/root/myscripts`目录，因此文件被提取到`/root/myscripts`而不是当前目录。

现在让我们验证文件确实提取到了`/root/myscripts`目录中：

```
root@ubuntu-linux:/# ls /root/myscripts
3x10.sh 
empty.sh 
fun1.sh 
hello20.sh 
hello3.sh 
math.sh 
noweb.sh 
rename.sh 
size3.sh 
detect.sh 
filetype.sh 
game.sh 
hello2.sh 
hello.sh 
mydate.sh 
numbers.sh 
size2.sh 
size.sh
```

果然，我们在`/root/myscripts`目录中看到了所有的 bash 脚本！

# 使用 gzip 进行压缩

单独将文件放在存档中并不会节省磁盘空间。我们需要压缩存档以节省磁盘空间。在 Linux 上有许多压缩方法可供我们使用。但是，我们只将介绍三种最流行的压缩方法。

在 Linux 上最受欢迎的压缩方法可能是`gzip`，好处是它非常快速。您可以使用`tar`命令的`-z`选项将存档文件压缩为`gzip`，如下所示：

```
tar -czf compressed_archive archive_name
```

因此，要将`scripts.tar`存档压缩为名为`scripts.tar.gz`的`gzip`压缩存档，您首先需要切换到`/root/backup`目录，然后运行以下命令：

```
root@ubuntu-linux:~/backup# tar -czf scripts.tar.gz scripts.tar
```

现在，如果列出`backup`目录的内容，您将看到新创建的`gzip`压缩存档`scripts.tar.gz`：

```
root@ubuntu-linux:~/backup# ls 
scripts.tar scripts.tar.gz
```

通过使用`-z`选项进行了魔术操作，该选项使用`gzip`压缩方法压缩了存档。就是这样！请注意，这与创建存档非常相似：我们只是添加了`-z`选项-这是唯一的区别。

现在让我们在两个存档上运行`file`命令：

```
root@ubuntu-linux:~/backup# file scripts.tar 
scripts.tar: POSIX tar archive (GNU) 
root@ubuntu-linux:~/backup# file scripts.tar.gz
scripts.tar.gz: gzip compressed data, last modified: Sat Nov 2 22:13:44 2019, 
from Unix
```

如您所见，`file`命令检测到了两个存档的类型。现在让我们比较一下两个存档的大小（以字节为单位）：

```
root@ubuntu-linux:~/backup# du -b scripts.tar scripts.tar.gz 
20480 scripts.tar
1479 scripts.tar.gz
```

与未压缩存档`scripts.tar`相比，压缩存档`scripts.tar.gz`的大小要小得多，这是我们预期的。如果要将压缩存档`scripts.tar.gz`中的文件提取到`/root/myscripts`，可以运行： 

```
root@ubuntu-linux:~/backup# tar -xf scripts.tar.gz -C /root/myscripts
```

请注意，这与提取未压缩存档的内容的方式完全相同。

# 使用 bzip2 进行压缩

`bzip2`是 Linux 上另一种流行的压缩方法。平均而言，`bzip2`比`gzip`慢；然而，`bzip2`在将文件压缩到更小的大小方面做得更好。

你可以使用`tar`命令的`-j`选项来使用`bzip2`压缩压缩存档，如下所示：

```
tar -cjf compressed_archive archive_name
```

注意这里唯一的区别是我们使用`bzip2`压缩的`-j`选项，而不是`gzip`压缩的`-z`选项。

因此，要将`scripts.tar`存档压缩成名为`scripts.tar.bz2`的`bzip2`压缩存档，你首先需要切换到`/root/backup`目录，然后运行以下命令：

```
root@ubuntu-linux:~/backup# tar -cjf scripts.tar.bz2 scripts.tar
```

现在，如果你列出`backup`目录的内容，你会看到新创建的`bzip2`压缩的存档`scripts.tar.bz2`：

```
root@ubuntu-linux:~/backup# ls
scripts.tar scripts.tar.bz2 scripts.tar.gz
```

让我们在`bzip2`压缩的存档`scripts.tar.bz2`上运行`file`命令：

```
root@ubuntu-linux:~/backup# file scripts.tar.bz2 
scripts.tar.bz2: bzip2 compressed data, block size = 900k
```

它正确地检测到了用于存档`scripts.tar.bz2`的压缩方法。太棒了-现在让我们比较`gzip`压缩的存档`scripts.tar.gz`和`bzip2`压缩的存档`scripts.tar.bz2`的大小（以字节为单位）：

```
root@ubuntu-linux:~/backup# du -b scripts.tar.bz2 scripts.tar.gz 
1369 scripts.tar.bz2
1479 scripts.tar.gz
```

注意`bzip2`压缩的存档`scripts.tar.bz2`比`gzip`压缩的存档`scripts.tar.gz`要小。如果你想要将压缩存档`scripts.tar.bz2`中的文件提取到`/root/myscripts`，你可以运行：

```
root@ubuntu-linux:~/backup# tar -xf scripts.tar.bz2 -C /root/myscripts
```

注意它与提取`gzip`压缩的存档的内容的方式完全相同。

# 使用 xz 压缩

`xz`压缩方法是 Linux 上另一种流行的压缩方法。平均而言，`xz`压缩在减小（压缩）文件大小方面做得比所有三种压缩方法中的其他方法都要好。

你可以使用`tar`命令的`-J`选项来使用`xz`压缩压缩存档，如下所示：

```
tar -cJf compressed_name archive_name
```

注意这里我们使用大写字母`J`与`xz`压缩。因此，要将`scripts.tar`存档压缩成名为`scripts.tar.xz`的`xz`压缩存档，你首先需要切换到`/root/backup`目录，然后运行以下命令：

```
root@ubuntu-linux:~/backup# tar -cJf scripts.tar.xz scripts.tar
```

现在，如果你列出`backup`目录的内容，你会看到新创建的`xz`压缩的存档`scripts.tar.xz`：

```
root@ubuntu-linux:~/backup# ls
scripts.tar scripts.tar.bz2 scripts.tar.gz scripts.tar.xz
```

让我们在`scripts.tar.xz`上运行`file`命令：

```
root@ubuntu-linux:~/backup# file scripts.tar.xz 
scripts.tar.xz: XZ compressed data
```

它正确地检测到了用于存档`scripts.tar.xz`的压缩方法。

# 性能测量

你可以使用`time`命令来测量命令（或程序）执行所需的时间。`time`命令的一般语法如下：

```
time command_or_program
```

例如，要测量`date`命令执行所需的时间，你可以运行以下命令：

```
root@ubuntu-linux:~# time date 
Sun Nov 3 16:36:33 CST 2019

real 0m0.004s 
user 0m0.003s 
sys 0m0.000s
```

在我的系统上运行`date`命令只用了四毫秒；这相当快！

`gzip`压缩方法是所有三种压缩方法中最快的；好吧，让我们看看我是在撒谎还是在说实话！切换到`/root/backup`目录：

```
root@ubuntu-linux:~# cd /root/backup 
root@ubuntu-linux:~/backup#
```

现在让我们看看为`/boot`中的所有文件创建一个`gzip`压缩的存档文件需要多长时间：

```
root@ubuntu-linux:~/backup# time tar -czf boot.tar.gz /boot 
real 0m4.717s
user 0m4.361s 
sys 0m0.339s
```

在我的系统上，运行`gzip`花了 4.717 秒！现在让我们测量创建相同目录`/boot`的`bzip2`压缩存档所需的时间：

```
root@ubuntu-linux:~/backup# time tar -cjf boot.tar.bz2 /boot 
real 0m19.306s
user 0m18.809s 
sys   0m0.359s
```

`bzip2`花了巨大的 19.306 秒！你可以看到`gzip`压缩比`bzip2`快得多。现在让我们看看创建相同目录`/boot`的`xz`压缩存档所需的时间：

```
root@ubuntu-linux:~/backup# time tar -cJf boot.tar.xz /boot 
real 0m53.745s
user 0m52.679s 
sys   0m0.873s
```

`xz`几乎花了整整一分钟！我们可以得出结论，`gzip`绝对是我们讨论的所有三种压缩方法中最快的。

最后，让我们检查三个压缩存档的大小（以字节为单位）：

```
root@ubuntu-linux:~/backup# du -b boot.* 
97934386 boot.tar.bz2
98036178 boot.tar.gz
94452156 boot.tar.xz
```

正如你所看到的，`xz`在压缩文件方面做得最好。`bzip2`排名第二，`gzip`排名最后。

# 知识检查

对于以下练习，打开你的终端并尝试解决以下任务：

1.  在`/root`中为`/var`中的所有文件创建一个名为`var.tar.gz`的`gzip`存档。

1.  在`/root`中为`/tmp`中的所有文件创建一个名为`tmp.tar.bz2`的`bzip2`存档。

1.  在`/root`目录中为`/etc`目录中的所有文件创建一个名为`etc.tar.xz`的`xz`归档文件。
