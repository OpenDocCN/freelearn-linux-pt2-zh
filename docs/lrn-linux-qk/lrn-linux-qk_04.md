复制、移动和删除文件

如果您以前拥有过计算机，那么您就知道能够在文件之间复制和移动文件是多么重要。这就是为什么我专门写了一整章来讨论这个问题：复制、移动和删除文件。

# 第五章：复制一个文件

有时您需要复制单个文件。幸运的是，在命令行上这是一个简单的操作。我在我的主目录中有一个名为`cats.txt`的文件：

```
elliot@ubuntu-linux:~$ cat cats.txt 
I love cars!
I love cats!
I love penguins!
elliot@ubuntu-linux:~$
```

我可以使用`cp`命令复制名为`cats.txt`的文件并命名为`copycats.txt`，方法如下：

```
elliot@ubuntu-linux:~$ cp cats.txt copycats.txt 
elliot@ubuntu-linux:~$ cat copycats.txt
I love cars!
I love cats!
I love penguins!
elliot@ubuntu-linux:~$
```

如您所见，复制的文件`copycats.txt`与原始文件`cats.txt`具有相同的内容。

我也可以将文件`cats.txt`复制到另一个目录。例如，我可以通过运行`cp cats.txt /tmp`命令将文件`cats.txt`复制到`/tmp`中：

```
elliot@ubuntu-linux:~$ cp cats.txt /tmp
elliot@ubuntu-linux:~$ cd /tmp
elliot@ubuntu-linux:/tmp$ ls
cats.txt
elliot@ubuntu-linux:/tmp$
```

请注意，复制的文件与原始文件具有相同的名称。我也可以在**`/tmp`**中用不同的名称再复制一份：

```
elliot@ubuntu-linux:~$ cp cats.txt /tmp/cats2.txt
elliot@ubuntu-linux:~$ cd /tmp
elliot@ubuntu-linux:/tmp$ ls 
cats2.txt  cats.txt
elliot@ubuntu-linux:/tmp$
```

# 复制多个文件

您可能还想一次复制多个文件。为了演示，让我们首先在 Elliot 的主目录中创建三个文件`apple.txt`、`banana.txt`和`carrot.txt`：

```
elliot@ubuntu-linux:~$ touch apple.txt banana.txt carrot.txt
elliot@ubuntu-linux:~$ ls
apple.txt carrot.txt copycats.txt dir1 
banana.txt cats.txt Desktop
elliot@ubuntu-linux:~$
```

要将三个新创建的文件复制到`/tmp`，您可以运行`cp apple.txt ba- nana.txt carrot.txt /tmp`命令：

```
elliot@ubuntu-linux:~$ cp apple.txt banana.txt carrot.txt /tmp
elliot@ubuntu-linux:~$ cd /tmp
elliot@ubuntu-linux:/tmp$ ls
apple.txt banana.txt carrot.txt cats2.txt cats.txt
elliot@ubuntu-linux:/tmp$
```

小菜一碟！一般来说，`cp`命令遵循以下语法：

```
cp source_file(s) destination
```

# 复制一个目录

您可能还想复制整个目录；这也很容易实现。为了演示，在您的主目录中创建一个名为`cities`的目录，并在`cities`中创建三个文件`paris`、`tokyo`和`london`，如下所示：

```
elliot@ubuntu-linux:~$ mkdir cities
elliot@ubuntu-linux:~$ cd cities/
elliot@ubuntu-linux:~/cities$ touch paris tokyo london
elliot@ubuntu-linux:~/cities$ ls
london paris tokyo
```

现在，如果您想将`cities`目录复制到`/tmp`，您必须向`cp`命令传递递归的`-r`选项，如下所示：

```
elliot@ubuntu-linux:~/cities$ cd ..
elliot@ubuntu-linux:~$ cp -r cities /tmp
```

如果您省略了`-r`选项，将会收到错误消息：

```
elliot@ubuntu-linux:~$ cp cities /tmp
cp: -r not specified; omitting directory 'cities'
```

您可以通过列出`/tmp`中的文件来验证`cities`目录是否已复制到`/tmp`中：

```
elliot@ubuntu-linux:~$ cd /tmp
elliot@ubuntu-linux:/tmp$ ls
apple.txt banana.txt carrot.txt cats2.txt cats.txt cities
elliot@ubuntu-linux:/tmp$ ls cities
london paris tokyo
```

# 复制多个目录

您还可以像复制多个文件一样复制多个目录；唯一的区别是您必须向`cp`命令传递递归的`-r`选项。

为了演示，在 Elliot 的主目录中创建三个目录`d1`、`d2`和`d3`：

```
elliot@ubuntu-linux:~$ mkdir d1 d2 d3
```

现在，您可以通过运行`cp -r d1 d2 d3 /tmp`命令将所有三个目录复制到`/tmp`中：

```
elliot@ubuntu-linux:~$ cp -r d1 d2 d3 /tmp
elliot@ubuntu-linux:~$ cd /tmp
elliot@ubuntu-linux:/tmp$ ls
apple.txt banana.txt carrot.txt cats2.txt cats.txt cities d1 d2 d3
```

# 移动一个文件

有时，您可能希望将文件（或目录）移动到不同的位置，而不是复制并浪费磁盘空间。

为此，您可以使用`mv`命令。例如，您可以通过运行`mv copycats.txt /tmp`命令，将文件`copycats.txt`从 Elliot 的主目录移动到`/tmp`：

```
elliot@ubuntu-linux:~$ mv copycats.txt /tmp
elliot@ubuntu-linux:~$ ls
apple.txt   carrot.txt cities d2  Desktop  Downloads
banana.txt  cats.txt   d1     d3  dir1     Pictures
elliot@ubuntu-linux:~$ cd /tmp
elliot@ubuntu-linux:/tmp$ ls
apple.txt  carrot.txt cats.txt copycats.txt d2
banana.txt cats2.txt  cities   d1           d3
```

请注意，`copycats.txt`现在已经从 Elliot 的主目录中消失，因为它已经迁移到`/tmp`中。

# 移动多个文件

您也可以像复制多个文件一样移动多个文件。例如，您可以将三个文件`apple.txt`、`banana.txt`和`carrot.txt`从`/tmp`移动到`/home/elliot/d1`，方法如下：

```
elliot@ubuntu-linux:/tmp$ mv apple.txt banana.txt carrot.txt /home/elliot/d1
elliot@ubuntu-linux:/tmp$ ls
cats2.txt cats.txt cities copycats.txt d1 d2 d3
elliot@ubuntu-linux:/tmp$ cd /home/elliot/d1
elliot@ubuntu-linux:~/d1$ ls
apple.txt banana.txt carrot.txt
elliot@ubuntu-linux:~/d1$
```

如您所见，三个文件`apple.txt`、`banana.txt`和`carrot.txt`不再位于`/tmp`中，因为它们都移动到了`/home/elliot/d1`。一般来说，`mv`命令遵循以下语法：

```
mv source_file(s) destination
```

# 移动一个目录

您还可以使用`mv`命令移动目录。例如，如果要移动目录`d3`并将其放入`d2`中，则可以运行`mv d3 d2`命令：

```
elliot@ubuntu-linux:~$ mv d3 d2
elliot@ubuntu-linux:~$ cd d2
elliot@ubuntu-linux:~/d2$ ls 
d3
elliot@ubuntu-linux:~/d2$
```

请注意，移动目录不需要使用递归的`-r`选项。

# 移动多个目录

您还可以一次移动多个目录。为了演示，在 Elliot 的主目录中创建一个名为`big`的目录：

```
elliot@ubuntu-linux:~$ mkdir big
```

现在您可以将三个目录`d1`、`d2`和`cities`移动到`big`目录中，方法如下：

```
elliot@ubuntu-linux:~$ mv d1 d2 cities big
elliot@ubuntu-linux:~$ ls big
cities d1 d2
elliot@ubuntu-linux:~$
```

# 重命名文件

您还可以使用`mv`命令重命名文件。例如，如果要将文件`cats.txt`重命名为`dogs.txt`，可以运行`mv cats.txt dogs.txt`命令：

```
elliot@ubuntu-linux:~$ mv cats.txt dogs.txt
elliot@ubuntu-linux:~$ cat dogs.txt
I love cars!
I love cats!
I love penguins!
elliot@ubuntu-linux:~$
```

如果要将目录`big`重命名为`small`，可以运行`mv big small`命令：

```
elliot@ubuntu-linux:~$ mv big small
elliot@ubuntu-linux:~$ ls small 
cities d1 d2
elliot@ubuntu-linux:~$
```

总之，这就是`mv`命令的工作原理：

1.  如果目标目录存在，`mv`命令将移动源文件到目标目录。

1.  如果目标目录不存在，`mv`命令将重命名源文件。

请记住，您一次只能重命名一个文件（或一个目录）。

# 隐藏文件

您可以通过将文件重命名为以点开头的名称来隐藏任何文件。

让我们试试吧；您可以通过将文件重命名为`.dogs.txt`来隐藏文件`dogs.txt`，如下所示：

```
elliot@ubuntu-linux:~$ ls
apple.txt banana.txt carrot.txt dogs.txt Desktop dir1 small
elliot@ubuntu-linux:~$ mv dogs.txt .dogs.txt
elliot@ubuntu-linux:~$ ls
apple.txt banana.txt carrot.txt Desktop dir1 small
elliot@ubuntu-linux:~$
```

正如您所看到的，文件`dogs.txt`现在被隐藏了，因为它被重命名为`.dogs.txt`。您可以通过重命名它并删除文件名前面的点来取消隐藏`.dogs.txt`：

```
elliot@ubuntu-linux:~$ mv .dogs.txt dogs.txt
elliot@ubuntu-linux:~$ ls
apple.txt banana.txt carrot.txt dogs.txt Desktop dir1 small
elliot@ubuntu-linux:~$
```

是的，先生！您也可以以相同的方式隐藏和取消隐藏目录。我会留下这个让你作为练习。

# 删除文件

您可以使用`rm`命令来删除文件。例如，如果要删除文件`dogs.txt`，可以运行`rm dogs.txt`命令：

```
elliot@ubuntu-linux:~$ ls
apple.txt banana.txt carrot.txt dogs.txt Desktop dir1 small
elliot@ubuntu-linux:~$ rm dogs.txt
elliot@ubuntu-linux:~$ ls
apple.txt banana.txt carrot.txt Desktop dir1 small
```

您也可以一次删除多个文件。例如，您可以通过运行`rm apple.txt banana.txt carrot.txt`命令来删除三个文件`apple.txt`，`banana.txt`和`carrot.txt`：

```
elliot@ubuntu-linux:~$ rm apple.txt banana.txt carrot.txt
elliot@ubuntu-linux:~$ ls
Desktop dir1 small 
elliot@ubuntu-linux:~$
```

# 删除目录

您可以通过传递递归的`-r`选项来删除目录的`rm`命令。为了演示，让我们首先在 Elliot 的主目录中创建一个名为`garbage`的目录：

```
elliot@ubuntu-linux:~$ mkdir garbage
elliot@ubuntu-linux:~$ ls
Desktop dir1 garbage small
```

现在让我们尝试删除`garbage`目录：

```
elliot@ubuntu-linux:~$ rm garbage
rm: cannot remove 'garbage': Is a directory
elliot@ubuntu-linux:~$
```

糟糕！我出错了，因为我没有传递递归的`-r`选项。这次我会传递递归选项：

```
elliot@ubuntu-linux:~$ rm -r garbage
elliot@ubuntu-linux:~$ ls
Desktop dir1 small
```

太棒了！我们摆脱了`garbage`目录。

你也可以使用`rmdir`命令来删除只有空目录。为了演示，让我们创建一个名为`garbage2`的新目录，并在其中创建一个名为`old`的文件：

```
elliot@ubuntu-linux:~$ mkdir garbage2
elliot@ubuntu-linux:~$ cd garbage2
elliot@ubuntu-linux:~/garbage2$ touch old
```

现在让我们回到 Elliot 的主目录，并尝试使用`rmdir`命令删除`garbage2`：

```
elliot@ubuntu-linux:~/garbage2$ cd ..
elliot@ubuntu-linux:~$ rmdir garbage2
rmdir: failed to remove 'garbage2': Directory not empty
```

正如您所看到的，它不允许您删除非空目录。因此，让我们删除`garbage2`中的文件`old`，然后重新尝试删除`garbage2`：

```
elliot@ubuntu-linux:~$ rm garbage2/old
elliot@ubuntu-linux:~$ rmdir garbage2
elliot@ubuntu-linux:~$ ls
Desktop dir1 small 
elliot@ubuntu-linux:~$
```

哇！`garbage2`目录永远消失了。这里要记住的一件事是，`rm -r`命令将删除任何目录（空目录和非空目录）。另一方面，`rmdir`命令只会删除空目录。

在本章的最后一个示例中，让我们创建一个名为`garbage3`的目录，然后在其中创建两个文件`a1.txt`和`a2.txt`：

```
elliot@ubuntu-linux:~$ mkdir garbage3
elliot@ubuntu-linux:~$ cd garbage3/
elliot@ubuntu-linux:~/garbage3$ touch a1.txt a2.txt
elliot@ubuntu-linux:~/garbage3$ ls
a1.txt a2.txt
```

现在让我们回到 Elliot 的主目录，尝试删除`garbage3`：

```
elliot@ubuntu-linux:~/garbage3$ cd ..
elliot@ubuntu-linux:~$ rmdir garbage3
rmdir: failed to remove 'garbage3': Directory not empty
elliot@ubuntu-linux:~$ rm -r garbage3
elliot@ubuntu-linux:~$ ls
Desktop dir1 Downloads Pictures small
elliot@ubuntu-linux:~$
```

正如您所看到的，`rmdir`命令未能删除非空目录`garbage3`，而`rm -r`命令成功删除了它。

没有什么比一个好的知识检查练习更能让信息牢固地留在你的脑海中了。

# 知识检查

对于以下练习，打开您的终端并尝试解决以下任务：

1.  在您的主目录中创建三个文件`hacker1`，`hacker2`和`hacker3`。

1.  在您的主目录中创建三个目录`Linux`，`Windows`和`Mac`。

1.  在您在任务 2 中创建的`Linux`目录中创建一个名为`cool`的文件。

1.  在您在任务 2 中创建的`Windows`目录中创建一个名为`boring`的文件。

1.  在您在任务 2 中创建的`Mac`目录中创建一个名为`expensive`的文件。

1.  将两个文件`hacker1`和`hacker2`复制到`/tmp`目录。

1.  将两个目录`Windows`和`Mac`复制到`/tmp`目录。

1.  将文件`hacker3`移动到`/tmp`目录。

1.  将目录`Linux`移动到`/tmp`目录。

1.  从您的主目录中的`Mac`目录中删除文件`expensive`。

1.  从您的主目录中删除目录`Mac`。

1.  从您的主目录中删除目录`Windows`。

1.  从您的主目录中删除文件`hacker2`。

1.  将文件`hacker1`重命名为`hacker01`。

## 真或假

1.  `cp`命令可以复制目录，而不使用递归选项`-r`。

1.  在移动目录时，您必须使用递归选项`-r`。

1.  您可以使用`mv`命令来重命名文件或目录。

1.  您可以使用`rmdir`命令删除非空目录。

1.  您可以使用`rm -r`命令删除非空目录。
