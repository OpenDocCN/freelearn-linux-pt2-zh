# 第八章：执行文件管理

在上一章中，我们处理了 Red Hat 世界中的软件包管理。特别是，我们涵盖了`yum`，`dnf`，`rpm`和`yumex`实用程序。

在本章中，我们的重点将转向文件管理。我们将探讨在 shell 中工作的方法。我们将致力于创建，修改和删除文件。此外，我们将使用目录，演示如何创建，移动和删除目录。接下来，我们将涉及对文件和目录进行搜索。最后，我们将涵盖管道和重定向。

本章将涵盖以下主题：

+   在 CLI 中查看和移动文件和目录

+   创建，复制，移动，重命名和删除文件

+   创建和删除目录

+   查找文件和目录

+   管道和重定向

# 在 CLI 中查看和移动文件和目录

首先，您需要熟悉在 CLI 中的工作。在前几章中，我们与 shell 进行了交互。现在，我们希望在 CLI 中变得高效。当我们首次打开终端时，我们被放置在用户的主目录中，如下所示：

```
[philip@localhost ~]$
```

在上述输出中，我们被放置到用户`philip`的主目录中。可以通过发出打印工作目录（`pwd`）命令来确认这一点，如下所示：

```
[philip@localhost ~]$ pwd
/home/philip
[philip@localhost ~]$
```

在上述输出中，我们确认了我们确实在`/home/philip`目录中。但是，更有趣的是。`/home/philip`内有各种目录。我们可以通过使用列表（`ls`）命令来确认这一点，如下所示：

```
[philip@localhost ~]$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
[philip@localhost ~]$
```

上述输出中列出的目录（文件夹）是为系统中的每个用户创建的。现在，目录的显示方式并没有告诉我们很多。为了深入了解，我们可以再次发出`ls`命令；这次，我们将传递`-l`选项。`-l`选项会显示文件类型，用户权限，组权限，用户所有权，组所有权，大小和上次修改日期等信息，如下所示：

```
[philip@localhost ~]$ ls -l
total 32
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Desktop
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Documents
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Downloads
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Music
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Pictures
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Public
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Templates
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Videos
[philip@localhost ~]$
```

我们可以从上述输出中获得一些有用的信息。例如，每个目录都有权限；我们还可以看到所有权和大小。此外，在 Linux 中，我们有所谓的隐藏文件/文件夹。当我们执行列表时，默认情况下不会显示它们；要获取它们，我们必须添加`-a`选项：

```
[philip@localhost ~]$ ls -al
drwx------. 15 philip philip 4096 Aug  2 10:28 .
drwxr-xr-x.  3 root   root   4096 Jul 31 14:58 ..
-rw-r--r--.  1 philip philip   18 Mar 15 09:56 .bash_logout
-rw-r--r--.  1 philip philip  193 Mar 15 09:56 .bash_profile
-rw-r--r--.  1 philip philip  231 Mar 15 09:56 .bashrc
drwx------. 14 philip philip 4096 Jul 31 14:59 .cache
drwx------. 14 philip philip 4096 Jul 31 14:59 .config
drwxr-xr-x.  2 philip philip 4096 Jul 31 14:59 Desktop
drwxr-xr-x.  2 philip philip 4096 Jul 31 14:59 Documents
drwxr-xr-x.  2 philip philip 4096 Jul 31 14:59 Downloads
-rw-------.  1 philip philip   16 Jul 31 14:58 .esd_auth
-rw-------.  1 philip philip  620 Aug  2 10:28 .ICEauthority
drwx------.  3 philip philip 4096 Jul 31 14:59 .local
drwxr-xr-x.  4 philip philip 4096 Apr 25 02:33 .mozilla
drwxr-xr-x.  2 philip philip 4096 Jul 31 14:59 Music
drwxr-xr-x.  2 philip philip 4096 Jul 31 14:59 Pictures
drwxrw----.  3 philip philip 4096 Jul 31 14:59 .pki
[philip@localhost ~]$
```

太棒了！这样，我们可以知道文件或目录是否被隐藏；这些文件/目录的名称前面有一个句点。为了在目录之间移动，我们使用`cd`命令。更改目录或`cd`允许我们导航 Linux 文件系统。所以，让我们继续到`/home/philip/Documents`。我们使用以下命令：

```
[philip@localhost ~]$ cd /home/philip/Documents
[philip@localhost Documents]$
```

还有另一种在目录之间移动的方法。我们使用的第一种方法称为绝对路径；这意味着我们指定了到目录的完整路径。在目录之间移动的下一种方法是指定相对路径，如下所示：

```
[philip@localhost ~]$ cd Documents/
[philip@localhost Documents]$
```

相对方法要求您必须在子目录的父目录中才能工作。

一旦我们进入子目录，我们可以执行`ls`命令，如下所示：

```
[philip@localhost Documents]$ ls
[philip@localhost Documents]$
```

当前，该目录中没有内容。为了返回到父目录，我们可以使用`cd`命令，如下所示：

```
[philip@localhost Documents]$ cd /home/philip
[philip@localhost ~]$ pwd
/home/philip
[philip@localhost ~]$
```

在上述输出中，我们指定了路径。这种方法总是有效的。我们还可以以另一种方式使用`cd`命令，如下所示：

```
[philip@localhost Documents]$ cd ..
[philip@localhost ~]$
```

在上述方法中，我们使用了双点。双点表示父目录。如果我们指定了一个单点，结果将如下所示：

```
[philip@localhost Documents]$ cd .
[philip@localhost Documents]$
```

单点引用当前目录本身。无论您在哪里，都可以使用以下方法：

```
[philip@localhost Documents]$ cd ~
[philip@localhost ~]$
```

波浪号（`~`）字符将始终将我们带回用户的主目录。为了说明这一点，我们将进入`/etc`目录，如下所示：

```
[philip@localhost ~]$ cd /etc
[philip@localhost etc]$ pwd
/etc
[philip@localhost etc]$
```

现在，我们将再次发出`cd`命令，传递波浪号（`~`）：

```
[philip@localhost etc]$ cd ~
[philip@localhost ~]$ pwd
/home/philip
[philip@localhost ~]$
```

太棒了！您现在可以看到波浪号（`~`）字符的威力。在文件系统层次结构的顶部是根目录。我们通常将根目录称为`/`；这不应与`/root`目录混淆。后者是 root 用户的主目录。从`/`开始创建所有其他目录。我们可以这样进入`/`：

```
[philip@localhost ~]$ cd /
[philip@localhost /]$ pwd
/
[philip@localhost /]$
```

在上述输出中，我们位于文件系统的根目录。我们可以以类似的方式查看此目录，如下所示：

```
[philip@localhost /]$ ls
bin   dev  home  lib64       media  opt   root  sbin  sys  usr
boot  etc  lib   lost+found  mnt    proc  run   srv   tmp  var
[philip@localhost /]$
```

您会注意到这里有一些熟悉的目录，比如`/home`和`/dev`。有趣的是，我们可以看到列出了`/root`目录。我们可以切换到该目录并执行列表，如下所示：

```
[philip@localhost /]$ cd /root
bash: cd: /root: Permission denied
[philip@localhost /]$ 
```

由于我们没有权限查看`/root`目录，所以出现了上述错误。让我们以 root 用户的身份进行身份验证并重试，如下所示：

```
[root@localhost /]# cd /root
[root@localhost ~]#
```

看吧！我们现在位于`/root`目录。这一次，当我们列出时，我们会立刻注意到这不是`/`目录：

```
[root@localhost ~]# ls
anaconda-ks.cfg
[root@localhost ~]#
```

根据上述输出，浏览目录结构相当直观。

# 创建、复制、移动、重命名和删除文件

这一部分听起来有点复杂。不用担心；它涵盖了创建和删除文件的技术。它还涵盖了复制和重命名文件的方法。

我们每天使用各种文件。我们可以在`/home/philip/Documents/NewTest`目录上执行`ls`命令，如下所示：

```
philip@localhost Documents]$ cd NewTest/
[philip@localhost NewTest]$ ll -a
total 8
drwxrwxr-x. 2 philip philip 4096 Aug  6 12:04 .
drwxr-xr-x. 3 philip philip 4096 Aug  6 13:45 ..
[philip@localhost NewTest]$
```

目前，该目录内没有任何文件。在 Linux 中，我们可以从 shell 中创建文件；我们可以使用`touch`命令来实现这一点：

```
[philip@localhost NewTest]$ touch OurFile
[philip@localhost NewTest]$ ll
total 0
-rw-rw-r--. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

该文件是使用一些默认权限创建的。特别地，`-rw-rw-r--`表示用户（`-rw`）、组（`-rw`）和其他人（`-r--`）。第一个破折号（`-`）是文件类型的引用。在这种情况下，它是一个常规文件。(`rw-`)表示用户/所有者具有读取和写入权限。第二组`rw-`表示组也具有读取和执行权限。最后，`r--`表示其他人（所有其他人）具有读取权限。另外，`philip philip`部分指的是文件的所有者和文件所属的组。我们可以使用`chmod`命令更改此文件的权限。假设我们想要给其他人（所有其他人）读取和写入权限。我们可以这样做：

```
[philip@localhost NewTest]$ chmod o+w OurFile
[philip@localhost NewTest]$ ll
total 0
-rw-rw-rw-. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

太棒了！我们现在可以看到其他权限为`rw-`。除了使用`o+w`之外，还有另一种更改权限的方法，那就是使用数字值。我将使用数字格式将其他人改回`r--`，如下所示：

```
[philip@localhost NewTest]$ chmod 664 OurFile
[philip@localhost NewTest]$ ll
total 0
```

```
-rw-rw-r--. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

我们可以将上述代码解读如下：在`664`中，`6`等于读取和写入，`6`等于读取和写入，`4`等于读取。第一个数字是用户的占位符。第二个数字是组的占位符，最后一个数字是其他的占位符。为了进一步说明这一点，我们可以去掉读取并保留写入组权限，如下所示：

```
[philip@localhost NewTest]$ chmod 624 OurFile
[philip@localhost NewTest]$ ll
total 0
-rw--w-r--. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

同样地，我们可以通过增加数值来添加权限。让我们选择其他人；我们将给其他人读取和执行权限，如下所示：

```
[philip@localhost NewTest]$ chmod 625 OurFile
[philip@localhost NewTest]$ ll
total 0
-rw--w-r-x. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

太棒了！我们甚至可以在单个命令中为用户、组或其他人提供所有权限（读取、写入和执行）。让我们从用户开始，如下所示：

```
[philip@localhost NewTest]$ ll
total 0
-rwx-w-r-x. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

现在，我们可以看到用户具有读取、写入和执行权限。我通过将读取等于`4`、写入等于`2`和执行等于`1`相加得到值`7`。我们现在将给组所有权限，如下所示：

```
[philip@localhost NewTest]$ chmod 775 OurFile
[philip@localhost NewTest]$ ll
total 0
-rwxrwxr-x. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

太棒了！我们还可以在单个命令中删除用户、组或其他人的所有权限。让我们删除其他人的权限（读取、写入和执行），如下所示：

```
[philip@localhost NewTest]$ chmod 770 OurFile
[philip@localhost NewTest]$ ll
total 0
-rwxrwx---. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

放置零（`0`）会取消特定部分（用户、组或其他）的所有权限。您可以看到权限的强大。同样，我们可以使用字母，就像之前看到的那样。`u`表示用户，`g`表示组，`o`表示其他。我们可以按如下方式从组中删除执行权限：

```
[philip@localhost NewTest]$ chmod g-x OurFile
[philip@localhost NewTest]$ ll
total 0
-rwxrw----. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

我们可以使用加号（`+`）符号（添加权限）或减号（`-`）符号（删除权限）。我们还可以将文件从一个位置复制到另一个位置，或者在同一个位置内部。如果文件的目的地在与源相同的位置内部，则必须给出不同的名称。

`cp`命令用于复制。我们将复制文件并将其放在`/home/philip/Documents/`中，如下所示：

```
[philip@localhost NewTest]$ cp OurFile /home/philip/Documents/NewFile
[philip@localhost NewTest]$ ll /home/philip/Documents/
-rwxrw----. 1 philip philip 0 Aug 6 14:34 NewFile
drwxrwxr-x. 2 philip philip 4096 Aug 6 13:52 NewTest
[philip@localhost NewTest]$
```

太棒了！

目录的权限前面有一个`d`。

我们还可以移动文件；`mv`命令用于移动文件。让我们移动`/home/philip/Documents/NewFile`并将其放在`/home/philip/Documents/NewTest`中：

```
[philip@localhost NewTest]$ mv /home/philip/Documents/NewFile .
[philip@localhost NewTest]$ ll
-rwxrw----. 1 philip philip 0 Aug  6 14:34 NewFile
-rwxrw----. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$  
```

我们使用的方法是为位置指定一个句点（`.`）。这表示当前工作目录；因此，我们可以使用句点（`.`）而不是键入完整的目标路径。

我们还可以使用`mv`命令重命名文件。让我们重命名`NewFile`：

```
[philip@localhost NewTest]$ mv NewFile RenameFile
[philip@localhost NewTest]$ ll
total 0
-rwxrw----. 1 philip philip 0 Aug  6 13:52 OurFile
-rwxrw----. 1 philip philip 0 Aug  6 14:34 RenameFile
[philip@localhost NewTest]$
```

哇！我们还可以重命名文件并将其放在另一个目录中，如下所示：

```
[philip@localhost NewTest]$ mv RenameFile /home/philip/Documents/
[philip@localhost NewTest]$ ll
-rwxrw----. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$
```

该文件不再位于当前目录中，而是位于`/home/philip/Documents`目录中：

```
[philip@localhost NewTest]$ ll /home/philip/Documents/
total 4
drwxrwxr-x. 2 philip philip 4096 Aug  6 14:57 NewTest
-rwxrw----. 1 philip philip    0 Aug  6 14:34 RenameFile
[philip@localhost NewTest]$
```

太棒了！我们还可以使用`rm`命令删除文件。让我们删除`/home/philip/Documents/NewTest/OurFile`，如下所示：

```
[philip@localhost NewTest]$ ll
total 0
-rwxrw----. 1 philip philip 0 Aug  6 13:52 OurFile
[philip@localhost NewTest]$ rm OurFile
```

```
[philip@localhost NewTest]$ ll
total 0
[philip@localhost NewTest]$
```

# 创建和删除目录

我们可以使用另一个常用命令来创建目录。`mkdir`命令可用于创建目录。让我们使用`ls`命令进行列表，如下所示：

```
[philip@localhost ~]$ ls
Desktop Documents Downloads Music Pictures Public Templates Videos
[philip@localhost ~]$
```

现在，让我们在`/home/philip`内创建自己的目录：

```
[philip@localhost ~]$ mkdir NewTest
[philip@localhost ~]$ ll
total 36
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Desktop
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Documents
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Downloads
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Music
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Pictures
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Public
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Templates
drwxrwxr-x. 2 philip philip 4096 Aug  6 12:04 NewTest
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Videos
[philip@localhost ~]$
```

在前面的代码中，我们的新目录列在底部。您还会注意到我们使用了`ll`命令；这只是`ls -l`命令的别名。可以通过使用`which`命令快速验证这一点，如下所示：

```
philip@localhost ~]$ which ll
alias ll='ls -l --color=auto'
 /usr/bin/ls
[philip@localhost ~]$
```

干得好！我们可以使用`cd`命令进入我们新创建的目录：

```
[philip@localhost ~]$ cd NewTest/
[philip@localhost NewTest]$ ls
[philip@localhost NewTest]$ pwd
/home/philip/NewTest
[philip@localhost NewTest]$
```

接下来，假设我们已经创建了一个目录并且打错了字。不用担心；我们可以利用`mv`命令，它可以重命名目录。让我们尝试重命名`/home/Test`目录：

```
[philip@localhost NewTest]$ pwd
/home/philip/Documents/NewTest
[philip@localhost NewTest]$ mv /home/philip/Documents/NewTest/ /home/philip/
[philip@localhost NewTest]$ pwd
/home/philip/Documents/NewTest
[philip@localhost NewTest]$
```

我们遇到这个错误是因为我们在目录内。让我们尝试带有`-v`选项的命令：

```
[philip@localhost NewTest]$ mv -v /home/philip/Documents/NewTest/ /home/philip/
mv: cannot stat '/home/philip/Documents/NewTest/'
[philip@localhost NewTest]$
```

为了解决这个问题，我们需要跳出目录，然后重新尝试`mv`命令，如下所示：

```
[philip@localhost ~]$ mv /home/philip/Documents/NewTest/ .
[philip@localhost ~]$ ll
total 36
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Desktop
drwxr-xr-x. 2 philip philip 4096 Aug  6 15:12 Documents
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Downloads
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Music
drwxrwxr-x. 2 philip philip 4096 Aug  6 15:00 NewTest
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Pictures
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Public
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Templates
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Videos
[philip@localhost ~]$
```

太棒了！现在，`NewTest`文件不再存在于`/home/philip/Documents/`中；这可以通过执行以下命令来显示：

```
[philip@localhost ~]$ ll Documents/
total 0
-rwxrw----. 1 philip philip 0 Aug  6 14:34 RenameFile
[philip@localhost ~]$
```

我们还可以使用`mv`命令重命名目录。诀窍是在调用`mv`命令时指定目录名称，如下所示：

```
[philip@localhost ~]$ mv NewTest/ ReName
[philip@localhost ~]$ ll
total 36
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Desktop
drwxr-xr-x. 2 philip philip 4096 Aug  6 15:12 Documents
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Downloads
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Music
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Pictures
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Public
drwxrwxr-x. 2 philip philip 4096 Aug  6 15:00 ReName
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Templates
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Videos
[philip@localhost ~]$
```

重命名目录就是这么简单。我们还可以更改目录的权限。让我们从组中删除读取、写入和执行权限，如下所示：

```
[philip@localhost ~]$ chmod -R 705 ReName/
[philip@localhost ~]$ ll
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Desktop
drwxr-xr-x. 2 philip philip 4096 Aug  6 15:12 Documents
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Downloads
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Music
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Pictures
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Public
drwx---r-x. 2 philip philip 4096 Aug  6 15:00 ReName
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Templates
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Videos
[philip@localhost ~]$
```

太棒了！`-R`选项告诉`chmod`命令将权限应用于`/home/philip/ReName`目录内的所有内容。当我们完成一个目录时，我们可以删除它。`rmdir`命令用于删除目录。让我们删除`/home/philip/ReName`目录，如下所示：

```
[philip@localhost ~]$ rmdir ReName/
[philip@localhost ~]$ ll
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Desktop
drwxr-xr-x. 2 philip philip 4096 Aug  6 15:12 Documents
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Downloads
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Music
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Pictures
```

```
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Public
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Templates
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Videos
[philip@localhost ~]$
```

基于前面的代码，没有遇到错误。在你的环境中可能不是这样。通常情况下，您要么有文件，要么有其他目录位于您试图删除的目录中。让我们快速创建一个目录，并在其中放置三个文件。然后，我们将尝试删除该目录：

```
[philip@localhost ~]$ mkdir TempDir
[philip@localhost ~]$ ll
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Desktop
drwxrwxr-x. 2 philip philip 4096 Aug  6 15:46 TempDir
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Templates
drwxr-xr-x. 2 philip philip 4096 Jul 31 14:59 Videos
[philip@localhost ~]$ touch TempDir/File1
[philip@localhost ~]$ touch TempDir/File2
[philip@localhost ~]$ touch TempDir/File3
[philip@localhost ~]$ ll TempDir/
total 0
-rw-rw-r--. 1 philip philip 0 Aug  6 15:47 File1
-rw-rw-r--. 1 philip philip 0 Aug  6 15:47 File2
-rw-rw-r--. 1 philip philip 0 Aug  6 15:47 File3
[philip@localhost ~]$  
```

现在，我们将重试`rm`命令并查看差异：

```
[philip@localhost ~]$ rmdir TempDir/
rmdir: failed to remove 'TempDir/': Directory not empty
[philip@localhost ~]$
```

瞧，我们遇到了一个错误。当目录不为空时，这是很常见的。我们可以很容易地解决这个问题；这次，我们将使用带有`-r`的`rm`命令，这意味着删除其后的所有内容。我们还可以添加`-v`选项，它将显示任何潜在权限问题的详细信息：

```
[philip@localhost ~]$ ll TempDir/
total 0
-rw-rw-r--. 1 philip philip 0 Aug  6 15:53 File1
-rw-rw-r--. 1 philip philip 0 Aug  6 15:53 File2
-rw-rw-r--. 1 philip philip 0 Aug  6 15:53 File3
[philip@localhost ~]$ rm -rv TempDir/
removed 'TempDir/File3'
removed 'TempDir/File1'
removed 'TempDir/File2'
removed directory 'TempDir/'
[philip@localhost ~]$ ll TempDir/
ls: cannot access 'TempDir/': No such file or directory
[philip@localhost ~]$
```

太棒了！

您可以使用`-f`删除整个目录，而无需确认。

# 查找文件和目录

通常，我们会在图形用户界面中搜索文件和目录。我们也可以在 shell 中执行搜索。首先，我们可以使用`find`命令；让我们搜索具有`.conf`扩展名的文件。搜索功能如下：

```
[philip@localhost ~]$ find /etc -iname "*.cfg"
find: ‘/etc/grub.d': Permission denied
find: ‘/etc/cups/ssl': Permission denied
/etc/libblockdev/conf.d/00-default.cfg
find: ‘/etc/audit': Permission denied
find: ‘/etc/dhcp': Permission denied
find: ‘/etc/sssd': Permission denied
/etc/grub2.cfg
find: ‘/etc/audisp': Permission denied
find: ‘/etc/polkit-1/rules.d': Permission denied
find: ‘/etc/polkit-1/localauthority': Permission denied
find: ‘/etc/openvpn/server': Permission denied
find: ‘/etc/openvpn/client': Permission denied
 [philip@localhost ~]$
```

现在，如果您遇到这些错误，这表明您需要一些高级权限。让我们再次尝试搜索，作为 root 用户：

```
[philip@localhost ~]$ su
Password:
[root@localhost philip]# find /etc -iname "*.cfg"'
/etc/libblockdev/conf.d/00-default.cfg
/etc/grub2-efi.cfg
/etc/vdpau_wrapper.cfg
/etc/grub2.cfg
[root@localhost philip]#
```

太棒了！我们甚至可以扩大我们想要执行搜索的区域。让我们搜索整个文件系统，如下所示：

```
[root@localhost philip]# find / -iname "*.cfg"
/home/philip/.config/yelp/yelp.cfg
/run/media/philip/Fedora-WS-Live-28-1-1/EFI/BOOT/grub.cfg
/run/media/philip/Fedora-WS-Live-28-1-1/isolinux/isolinux.cfg
find: ‘/run/user/1000/gvfs': Permission denied
/usr/lib64/libreoffice/share/config/soffice.cfg
/usr/lib64/libreoffice/help/en-US/schart.cfg
/usr/lib64/libreoffice/help/en-US/smath.cfg
/usr/lib64/libreoffice/help/en-US/sbasic.cfg
grub2.cfg
/boot/grub2/grub.cfg
[root@localhost philip]
```

出于简洁起见，本章中省略了一些输出。

我们也可以根据名称的一部分进行搜索。让我们搜索以`gru`开头的任何文件：

```
[root@localhost philip]# find /boot -iname "gru*"
/boot/efi/EFI/fedora/grubia32.efi
/boot/efi/EFI/fedora/grubenv
/boot/efi/EFI/fedora/grubx64.efi
/boot/grub2
/boot/grub2/grub.cfg
/boot/grub2/grubenv
[root@localhost philip]#
```

在前面的输出中，我们在`/boot`目录中进行了搜索。空文件通常只是静静地放在目录中，没有被使用。我们可以使用`find`命令来搜索空文件。传递`-type`选项以指定我们要搜索的内容：

```
[root@localhost philip]# find /home/philip/Documents  -empty
/home/philip/Documents/RenameFile
[root@localhost philip]#
```

太棒了！但是，等等；我们可以通过传递`-delete`选项来进行一些清理，以删除`find`命令从我们的搜索中返回的任何文件。我们可以这样做：

使用`-delete`选项时要小心，因为它会在某些情况下删除文件，甚至删除目录。在运行带有`-delete`选项的`find`之前，始终备份数据。

```
[root@localhost philip]# find /home/philip/Documents  -empty -delete
[root@localhost philip]# ll /home/philip/Documentsls: cannot access '/home/philip/Documents': No such file or directory
```

在前面的输出中，您会注意到`/home/philip/Documents/RenameFile`以及`/home/philip/Documents`已被删除。每当您传递`-delete`选项时都要非常小心。尽管在我们的情况下，我们正在使用实验环境，请务必记住这一点。在尝试传递`-delete`选项之前备份数据。

我们还可以根据权限搜索文件或目录。是的！我们将使用`find`命令传递`-readable`、`-writable`和`-executable`选项。它将如下所示：

```
[root@localhost philip]# find /etc/yum.repos.d/ -readable
/etc/yum.repos.d/
/etc/yum.repos.d/fedora-updates.repo
/etc/yum.repos.d/fedora.repo
/etc/yum.repos.d/fedora-cisco-openh264.repo
/etc/yum.repos.d/fedora-updates-testing.repo
 [root@localhost philip]# ll /etc/yum.repos.d/
total 16
-rw-r--r--. 1 root root  707 Apr 23 13:03 fedora-cisco-openh264.repo
-rw-r--r--. 1 root root 1331 Apr 23 13:03 fedora.repo
-rw-r--r--. 1 root root 1392 Apr 23 13:03 fedora-updates.repo
-rw-r--r--. 1 root root 1450 Apr 23 13:03 fedora-updates-testing.repo
[root@localhost philip]#
```

太棒了！您可以看到`find`命令的结果与具有`read`权限的文件的列表匹配。同样，我们可以搜索具有执行权限的文件和目录，如下所示：

```
[root@localhost philip]# ll /etc/init.d/
total 52
-rw-r--r--. 1 root root 18561 Jan  2  2018 functions
-rwxr-xr-x. 1 root root  7288 Apr 25 02:39 livesys
-rwxr-xr-x. 1 root root  1054 Apr 25 02:39 livesys-late
-rwxr-xr-x. 1 root root  4334 Jan  2  2018 netconsole
-rwxr-xr-x. 1 root root  7613 Jan  2  2018 network
-rw-r--r--. 1 root root  1161 Apr 18 17:59 README[root@localhost philip]# find /etc/init.d/ -perm -o+x
/etc/init.d/
/etc/init.d/livesys
/etc/init.d/livesys-late
/etc/init.d/netconsole
/etc/init.d/network
[root@localhost philip]#
```

在前面的输出中，只显示了其他人具有执行权限的文件。

此外，我们可以搜索具有写权限的文件和目录，如下所示：

```
[root@localhost philip]# find /etc/init.d/ -perm -o+w
[root@localhost philip]#
```

干得好！结果为空，因为没有文件或目录对其他人有写权限。同样，我们可以使用数字进行搜索。我们可以搜索执行权限，如下所示：

```
[root@localhost philip]# find /etc/init.d/ -perm -005
/etc/init.d/
/etc/init.d/livesys
/etc/init.d/livesys-late
/etc/init.d/netconsole
/etc/init.d/network
[root@localhost philip]#
```

在前面的输出中，只显示具有执行权限的目录。我们可以搜索具有写权限的文件和目录，如下所示：

```
[root@localhost philip]# find /etc/init.d/ -perm -002
[root@localhost philip]#
```

有趣的是，结果如预期般返回，因为其他人没有写权限。同样，我们可以搜索组的写权限，如下所示：

```
[root@localhost philip]# ll /etc/init.d/
total 52
-rw-r--r--. 1 root root 18561 Jan  2  2018 functions
-rwxr-xr-x. 1 root root  7288 Apr 25 02:39 livesys
-rwxr-xr-x. 1 root root  1054 Apr 25 02:39 livesys-late
-rwxr-xr-x. 1 root root  4334 Jan  2  2018 netconsole
```

```
-rwxr-xr-x. 1 root root  7613 Jan  2  2018 network
-rw-r--r--. 1 root root  1161 Apr 18 17:59 README
[root@localhost philip]# find /etc/init.d/ -perm -020
[root@localhost philip]#
```

太棒了！结果为空，因为组没有写权限。最后，我们可以搜索用户的写权限；这将产生以下结果：

```
[root@localhost philip]# find /etc/init.d/ -perm -200
/etc/init.d/
/etc/init.d/functions
/etc/init.d/livesys
/etc/init.d/README
/etc/init.d/livesys-late
/etc/init.d/netconsole
/etc/init.d/network
[root@localhost philip]#
```

干得好！语法是`-perm`，后跟用户（第一个数字）、组（第二个数字）和其他人（最后一个数字）。

搜索文件和目录的另一种流行方法是使用`locate`命令。`locate`实用程序在结果方面比`find`实用程序更快，这是因为`locate`命令使用数据库来执行查找。数据库称为`mlocate`。我们可以执行对我们创建的文件的简单搜索，如下所示：

```
[philip@localhost ~]$ locate TestFile
[philip@localhost ~]$
```

在前面的输出中，`locate` 命令不知道指定的文件。不用担心；我们只需要更新数据库，如下所示：

```
[philip@localhost ~]$ updatedb
updatedb: can not open a temporary file for `/var/lib/mlocate/mlocate.db'
[philip@localhost ~]$
```

如果遇到这个错误，这意味着您需要以 root 用户身份运行命令，如下所示：

```
[philip@localhost ~]$ su
Password:
[root@localhost philip]# updatedb
[root@localhost philip]#
Now, let's retry the locate command for the given file:
[root@localhost philip]# locate TestFile
/home/philip/Documents/TestFile1
[root@localhost philip]#
```

就是这样！我们也可以按扩展名搜索。为此，我们可以使用通配符，如下所示：

```
[root@localhost philip]# locate *.key
/etc/brlapi.key
/etc/trusted-key.key
/usr/lib64/libreoffice/help/en-US/sbasic.key
/usr/lib64/libreoffice/help/en-US/scalc.key
/usr/lib64/libreoffice/help/en-US/schart.key
/usr/lib64/libreoffice/help/en-US/sdatabase.key
/usr/lib64/libreoffice/help/en-US/sdraw.key
/usr/lib64/libreoffice/help/en-US/simpress.key
/usr/lib64/libreoffice/help/en-US/smath.key
/usr/lib64/libreoffice/help/en-US/swriter.key
/usr/share/doc/openssh/PROTOCOL.key
/usr/share/doc/python3-pycurl/tests/certs/server.key
[root@localhost philip]#
```

在前面的输出中，只显示了小写名称的结果；我们可以通过传递 `-i` 来解决这个问题，这告诉 `locate` 命令忽略大小写：

```
[root@localhost philip]# locate -i *.key
/etc/brlapi.key
/etc/trusted-key.key
/usr/lib64/libreoffice/help/en-US/sbasic.key
/usr/lib64/libreoffice/help/en-US/scalc.key
/usr/lib64/libreoffice/help/en-US/schart.key/usr/lib64/libreoffice/help/en-US/sdatabase.key
/usr/lib64/libreoffice/help/en-US/sdraw.key
/usr/lib64/libreoffice/help/en-US/simpress.key
/usr/lib64/libreoffice/help/en-US/smath.key
/usr/lib64/libreoffice/help/en-US/swriter.key
/usr/share/doc/openssh/PROTOCOL.key
/usr/share/doc/python3-pycurl/tests/certs/server.key
[root@localhost philip]#
```

在这种情况下，由于文件是小写，结果是相同的。我们还可以控制输出的显示方式；我们可以传递 `--null` 选项，如下所示：

```
[root@localhost philip]# locate --null *types
/etc/ethertypes/etc/mime.types/etc/firewalld/icmptypes/etc/selinux/targeted/contexts/customizable_types/etc/selinux/targeted/contexts/securetty_types/usr/include/bits/types/usr/lib/firewalld/icmptypes/usr/lib64/libreoffice/program/types/usr/lib64/libreoffice/share/filter/vml-shape-types/usr/lib64/perl5/bits/types/usr/lib64/python2.7/ctypes/usr/lib64/python2.7/ctypes/macholib/REAshare/icons/hicolor/512x512/mimetypes/usr/share/icons/hicolor/64x64/mimetypes/usr/share/icons/hicolor/72x72/mimetypes/usr/share/icons/hicolor/96x96/mimetypes/usr/share/icons/hicolor/scalable/mimetypes/usr/share/icons/locolor/16x16/mimetypes/usr/share/icons/locolor/32x32/mimetypes/usr/share/mime/types
[root@localhost philip]#
```

在前面的输出中，我们可以看到期望的结果。最后，我们可以查看有关数据库的信息；为此，我们可以使用 `-S` 选项：

```
[root@localhost philip]# locate -S
Database /var/lib/mlocate/mlocate.db:
12,517 directories
162,475 files
8,292,135 bytes in file names
3,883,960 bytes used to store database
[root@localhost philip]#
```

干得好！除了大小，我们还可以看到数据库的位置。

# 管道和重定向

通常，当我们查看各种命令的输出时，输出有点模糊。不用担心；我们有所谓的管道和重定向。基本上，当使用管道 (`|`) 时，我们获取一个命令的输出并将其作为另一个命令的输入。重定向 (`>`, `<`, `>>`, `2>`, 和 `2>&1`) 类似于从命令获取输出，但这次我们将其发送到一个位置，比如一个文件或另一个位置，等等。

首先，让我们使用 `ls` 命令；代码如下：

```
[root@localhost philip]# ls /etc
abrt               default    gdbinit.d        kernel                    networks           rc4.d      subuid-
adjtime            depmod.d   gdm               krb5.conf                 nfs.conf           rc5.d      sudoers
aliases            dhcp        geoclue          krb5.conf.d      nfsmount.conf      rc6.d     sudoers.d
alsa               DIR_COLORS  glvnd          ld.so.cache     nsswitch.conf      rc.d         sysconfig
alternatives       DIR_COLORS.256color          gnupg          ld.so.conf         nsswitch.conf.bak  rdma   sysctl.conf
[root@localhost philip]#
```

我们可以通过引入另一个强大的命令——`less` 命令，一页一页地查看输出：

```
[root@localhost philip]# ls /etc | less
cron.daily
cron.deny
cron.hourly
cron.monthly
crontab
cron.weekly
crypto-policies
crypttab
csh.cshrc
csh.login
cups
cupshelpers
:
[root@localhost philip]#
```

为了退出 `less` 命令，我们可以在键盘上使用 `q`。使用 `less` 命令的好处在于我们可以前后移动，而不是 `more` 命令，它只能向前移动。我们还可以使用管道 (`|`) 字符传递另一个命令期望的值。我们可以使用 `wc` 命令来说明这一点，如下所示：

```
[root@localhost philip]# ls /etc | wc -w
263
[root@localhost philip]#
```

在前面的输出中，我们获取了 `ls` 命令的输出并将其作为 `wc` 命令的输入。`wc` 命令用于计算单词数；`-w` 选项用于显示总单词数。

接下来，我们可以以多种方式使用重定向；在 Linux 中，我们有三种类型的流，如下所示：

+   标准输入 = 输入 `<`

+   标准输出 = 输出 ``>``

+   标准错误 = 标准错误 `2>`

此外，我们可以混合和匹配流，稍后在本节中您将看到。让我们从标准输入开始；我们可以使用 `wc` 命令并从文件中调用输入，如下所示：

```
[root@localhost philip]# wc -w < /boot/grub2/grub.cfg
435
[root@localhost philip]#
```

太棒了！`/boot/grub2/grub.cfg` 的字数被传递给 `wc` 命令。继续进行标准输出，我们可以获取命令的输出并将其存储到文件中。让我们使用 `ls` 命令，如下所示：

```
[root@localhost philip]# ls /etc/init.d/ > /home/philip/Documents/ls.txt
[root@localhost philip]#
```

在前面的输出中，我们列出了 `/etc/init.d/` 并将输出保存到了 `/home/philip/Documents/ls.txt`。可以通过以下方式进行验证：

```
[root@localhost philip]# cat /home/philip/Documents/ls.txt
functions
livesys
livesys-late
netconsole
network
README
[root@localhost philip]#
```

现在，假设我们使用 `ls` 命令来查看另一个目录；这将覆盖 `/home/philip/Documents/ls.txt` 的现有内容：

```
[root@localhost philip]# ls /boot/grub2/ > /home/philip/Documents/ls.txt
[root@localhost philip]# cat /home/philip/Documents/ls.txt
device.map
fonts
grub.cfg
grubenv
i386-pc
localethemes
[root@localhost philip]#
```

正如你所看到的，事实就在眼前。解决这个问题的方法是告诉标准输出我们想要追加输出，而不是覆盖它：

```
[root@localhost philip]# ls /var/ >> /home/philip/Documents/ls.txt
[root@localhost philip]# cat /home/philip/Documents/ls.txt
device.map
fonts
grub.cfg
grubenv
i386-pc
locale
themes
account
adm
cache
crash
db
empty
ftp
games
gopher
kerberos
lib
spool
tmp
www
yp
[root@localhost philip]#
```

好了！所以，我们使用 `>>` 将数据追加到现有文件中。接下来，我们可以将命令的标准输入结果与标准输出合并。如下所示：

```
[root@localhost philip]# wc -w < /var/log/boot.log  > /home/philip/Documents/STDIN_STDOUT.txt
[root@localhost philip]# cat /home/philip/Documents/STDIN_STDOUT.txt
2021
[root@localhost philip]#
```

干得好！我们还可以将标准错误重定向到一个文件。让我们使用文件命令，如下所示：

```
[root@localhost philip]# ls -l /tmp TestFileWithError 2> /home/philip/Documents/STDERR.txt
/tmp:
drwx------. 3 root root 60 Aug 2 10:23 systemd-private-a7a23120abff44c8bca6807f1711c1c2-bolt.service-7uNmVr
drwx------. 3 root root 60 Aug 2 10:22 systemd-private-a7a23120abff44c8bca6807f1711c1c2-chronyd.service-LPD8zu
drwx------. 3 root root 60 Aug 2 10:23 systemd-private-a7a23120abff44c8bca6807f1711c1c2-colord.service-13Vcs8
drwx------. 3 root root 60 Aug 2 10:28 systemd-private-a7a23120abff44c8bca6807f1711c1c2-fwupd.service-XOnyvf
drwx------. 3 root root 60 Aug 2 10:22 systemd-private-a7a23120abff44c8bca6807f1711c1c2-rtkit-daemon.service-ZD5mO7
drwx------. 2 philip philip 40 Aug 2 10:31 tracker-extract-files.1000
drwx------. 2 root root 40 Aug 2 10:22 vmware-root
[root@localhost philip]#
```

在前面的输出中，似乎命令已经执行。事实上，`/tmp` 的列表已经显示，但是文件 `TestFileWithError` 的错误没有被显示出来。错误被发送到了 `/home/philip/Documents/STDERR.txt`。可以通过以下方式进行验证：

```
[root@localhost philip]# cat /home/philip/Documents/STDERR.txt
ls: cannot access 'TestFileWithError': No such file or directory
[root@localhost philip]#
```

干得好！我们还可以将标准输出与标准错误合并到一个文件中。这是通过告诉 shell 我们想要将标准错误与标准输出一起存储在文件 `2>&1` 中实现的。可以这样做：

```
[root@localhost philip]# ls -l  /tmp TestFileWithError > /home/philip/Documents/STDERR.txt 2>&1
[root@localhost philip]# cat /home/philip/Documents/STDERR.txt
ls: cannot access 'TestFileWithError': No such file or directory
/tmp:
total 0
drwx------. 3 root   root   60 Aug  2 10:23 systemd-private-a7a23120abff44c8bca6807f1711c1c2-bolt.service-7uNmVr
drwx------. 3 root   root   60 Aug  2 10:22 systemd-private-a7a23120abff44c8bca6807f1711c1c2-chronyd.service-LPD8zu
drwx------. 3 root   root   60 Aug  2 10:23 systemd-private-a7a23120abff44c8bca6807f1711c1c2-colord.service-13Vcs8
drwx------. 3 root   root   60 Aug  2 10:28 systemd-private-a7a23120abff44c8bca6807f1711c1c2-fwupd.service-XOnyvf
drwx------. 3 root   root   60 Aug  2 10:22 systemd-private-a7a23120abff44c8bca6807f1711c1c2-rtkit-daemon.service-ZD5mO7
drwx------. 2 philip philip 40 Aug  7 14:45 tracker-extract-files.1000
drwx------. 2 root   root   40 Aug  2 10:22 vmware-root
[root@localhost philip]#
```

在上面的输出中，我们可以看到文件开头的错误，然后是`/tmp`的列表。最后，可以显示命令的输出并同时将输出重定向到文件；这是由另一个强大的命令——`tee`命令实现的。以下显示了`tee`命令的使用：

```
[root@localhost philip]# cat  /etc/hosts.allow | tee /home/philip/Documents/The_Tee_command.txt
#
# hosts.allow     This file contains access rules which are used to
#      allow or deny connections to network services that
#      either use the tcp_wrappers library or that have been
#      started through a tcp_wrappers-enabled xinetd.
#
#       See 'man 5 hosts_options' and 'man 5 hosts_access'
#       for information on rule syntax.
#       See 'man tcpd' for information on tcp_wrappers
[root@localhost philip]#
[root@localhost philip]# cat /home/philip/Documents/The_Tee_command.txt
#
# hosts.allow     This file contains access rules which are used to
#         allow or deny connections to network services that
#         either use the tcp_wrappers library or that have been
#        started through a tcp_wrappers-enabled xinetd.
#
#     See 'man 5 hosts_options' and 'man 5 hosts_access'
#     for information on rule syntax.
#     See 'man tcpd' for information on tcp_wrappers
[root@localhost philip]#
```

在上面的输出中，您可以看到`tee`命令的强大之处。

# 总结

这一章非常详细。我必须说我在这一章的工作中玩得很开心。我们涵盖了文件系统结构。您学会了如何使用`cd`命令导航文件系统。然后，我们看了如何识别工作目录。之后，我们介绍了查看目录内容的方法。除此之外，我们还揭示了默认情况下未显示的目录中的隐藏文件和目录。

接下来，我们讨论了如何在 shell 中创建文件。此外，您还了解了文件的各种权限以及如何更改这些权限。接着，我们转向了 Linux 环境中的目录。我们探讨了创建、移动、重命名和删除目录的各种方法。下一个主题涉及搜索文件和目录的技术。首先，我们广泛使用了`find`命令。接下来，我们探讨了`locate`命令。最后，我们在 shell 环境的背景下使用了管道和重定向。您看到了如何利用命令的输出并将其作为另一个命令的输入。然后，您看到了如何重定向到文件，包括 STDOUT 和 STDERR。最后，我们看了另一个强大的命令：`tee`命令。

在下一章中，我们将在 shell 环境的背景下研究进程。特别是，我们将研究一种管理进程的技术。将涵盖一些流行的命令，如`top`，`service`和`systemctl`，用于识别和管理进程。下一章将简洁明了，我们将专注于在 shell 环境中工作时每个 Linux 工程师都应该了解的方法。您将获得的技能将在您迈向认证之路上增加更多的信心。

# 问题

1.  以下哪个目录是`root`目录？

A. `/root/`

B. `/root`

C. `/home/root`

D. `/`

1.  以下哪个命令用于切换到另一个目录？

A. `pwd`

B. `chage`

C. `cd`

D. `change dir`

1.  以下哪个命令将打印当前工作目录？

A. `print dir`

B. `pwd`

C. `display`

D. `cd`

1.  以下哪个命令用于打印目录的内容？

A. `ls`

B. `which`

C. `whereis`

D. `cat`

1.  以下哪个选项可以用于使用`ls`命令显示文件和目录的权限？

A. `-r`

B. `-b`

C. `-a`

D. `-l`

1.  以下哪个选项可以用于使用`ls`命令显示隐藏文件和目录？

A. `-l`

B. `-b`

C. `-a`

D. `-u`

1.  以下哪个命令用于删除一个目录，即使它不是空的？

A. `rmdir`

B. `rm`

C. `remove`

D. `mv`

1.  以下哪个选项用于查找并删除空文件和目录，使用`find`命令？

A. `empty -remove`

B. `-empty -clean`

C. `-empty -delete`

D. `-empty -cycle`

1.  以下哪个命令用于更新`locate`命令使用的数据库？

A. `updatelocate`

B. `updatedatabase`

C. `locateupdate`

D. `updatedb`

1.  以下哪个命令显示命令的输出并同时将结果保存到文件中？

A. `less`

B. `more`

C. `wc`

D. `tee`

# 进一步阅读

+   您可以在以下网站获取有关文件操作的各种发行版的更多信息：[`unix.stackexchange.com`](https://unix.stackexchange.com)

+   要获取来自 Linux 社区用户的许多有用的技巧和最佳实践，请参阅：[`journalxtras.com`](https://journalxtras.com)

+   有关适用于 CentOS 和 Ubuntu 的各种命令的一般信息，以及发布问题供其他社区成员回答的能力，请参考[`www.linuxquestions.org`](https://www.linuxquestions.org)。
