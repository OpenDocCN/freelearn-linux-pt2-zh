让我们玩寻找游戏

我们有时都会忘记放东西的地方；我总是忘记放我的钱包和保存文件的位置。我很确定你也会忘记放文件的位置，因此在本章中，您将学习两种不同的搜索和定位文件的方法。

# 第十二章：locate 命令

如果您知道文件的名称，但不确定文件的位置，您可以使用`locate`命令获取文件的路径。

`locate`命令在预先构建的文件数据库中搜索文件位置，因此在使用`locate`命令之前更新文件数据库至关重要。如果您不更新数据库，`locate`命令可能无法检索新创建文件的位置。

# 更新文件数据库

要更新文件数据库，您必须以 root 用户身份运行`updatedb`命令：

```
root@ubuntu-linux:~# updatedb
```

`updatedb`命令不会显示任何输出。

现在，假设我们忘记了文件`facts.txt`的位置，我们不记得它在哪里；在这种情况下，您可以运行`locate`命令，然后跟上文件名：

```
root@ubuntu-linux:~# locate facts.txt
/home/elliot/facts.txt
/var/facts.txt
```

哇！它显示了文件`facts.txt`的位置。

现在我将向您展示如果搜索新创建的文件而不更新文件数据库会发生什么。

在`/home`目录中创建一个名为`ghost.txt`的空文件：

```
root@ubuntu-linux:/# touch /home/ghost.txt
```

现在尝试搜索文件`ghost.txt`：

```
root@ubuntu-linux:/# locate ghost.txt 
root@ubuntu-linux:/#
```

`locate`命令找不到它！为什么？........那是因为您创建了一个新文件，文件数据库还不知道它。您必须先运行`updatedb`命令来更新文件数据库：

```
root@ubuntu-linux:/# updatedb 
root@ubuntu-linux:/# locate ghost.txt
/home/ghost.txt
```

是的！更新文件数据库后，`locate`命令现在可以获取文件`ghost.txt`的位置。

您还可以使用`locate`命令来使用通配符。例如，`locate *.log`将搜索系统中的所有日志文件。您还可以使用`-r`选项在搜索中启用`regex`。

# find 命令

`find`命令是您可以在 Linux 中用于搜索文件的更强大的命令。与`locate`命令不同，`find`命令实时运行，因此您无需更新任何文件数据库。`find`命令的一般语法如下：

```
find [starting-point(s)] [options] [expression]
```

`find`命令将在您指定的每个起点（目录）下搜索。

例如，要在您的`/home`目录下搜索所有`.txt`文件，您可以运行：

```
root@ubuntu-linux:~# find /home -name "*.txt"
/home/elliot/facts2.txt
/home/elliot/dir1/directory2/file1.txt
/home/elliot/dir1/directory2/file3.txt
/home/elliot/dir1/directory2/file2.txt
/home/elliot/soft.txt
/home/elliot/facts.txt
/home/elliot/practise.txt
/home/elliot/upper.txt
/home/elliot/mydate.txt
/home/elliot/all.txt
/home/elliot/Mars.txt
/home/elliot/output.txt
/home/elliot/planets.txt
/home/elliot/error.txt
/home/elliot/animals.txt
/home/ghost.txt
```

`-name`选项搜索文件名；您可以在`find`命令中使用许多其他选项。

`-type`选项搜索文件类型；例如，要在`/home/elliot/dir1`中搜索所有目录，您可以运行：

```
root@ubuntu-linux:~# find /home/elliot/dir1 -type d
/home/elliot/dir1
/home/elliot/dir1/cities
/home/elliot/dir1/directory2
```

请注意，它只列出了`/home/elliot/dir1`中的目录。要列出常规文件，您可以运行：

```
root@ubuntu-linux:~# find /home/elliot/dir1 -type f
/home/elliot/dir1/cities/paris
/home/elliot/dir1/cities/london
/home/elliot/dir1/cities/berlin
/home/elliot/dir1/directory2/file1.txt
/home/elliot/dir1/directory2/file3.txt
/home/elliot/dir1/directory2/file2.txt
```

要搜索常规文件和目录，您可以使用逗号：

```
root@ubuntu-linux:~# find /home/elliot/dir1 -type d,f
/home/elliot/dir1
/home/elliot/dir1/cities
/home/elliot/dir1/cities/paris
/home/elliot/dir1/cities/london
/home/elliot/dir1/cities/berlin
/home/elliot/dir1/directory2
/home/elliot/dir1/directory2/file1.txt
/home/elliot/dir1/directory2/file3.txt
/home/elliot/dir1/directory2/file2.txt
```

现在，以 root 用户身份在`/root`中创建两个文件`large.txt`和`LARGE.TXT`：

```
root@ubuntu-linux:~# touch large.txt LARGE.TXT
```

假设您忘记了这两个文件的位置；在这种情况下，您可以使用`/`作为起点：

```
root@ubuntu-linux:~# find / -name large.txt
/root/large.txt
```

请注意，它只列出了`large.txt`的位置。如果您还想要另一个文件`LARGE.TXT`怎么办？在这种情况下，您可以使用`-iname`选项，使搜索不区分大小写：

```
root@ubuntu-linux:~# find / -iname large.txt
/root/LARGE.TXT
/root/large.txt
```

让我们将行“12345”附加到文件`large.txt`中：

```
root@ubuntu-linux:~# echo 12345 >> large.txt
```

请注意文件`large.txt`和`LARGE.txt`的大小：

```
root@ubuntu-linux:~# du -b large.txt LARGE.TXT
6 large.txt
0 LARGE.TXT
```

文件`LARGE.TXT`的大小为零字节，因为它是空的。您可以使用`-size`选项根据文件大小搜索文件。

例如，要在`/root`目录下搜索空文件，您可以运行以下命令：

```
root@ubuntu-linux:~# find /root -size 0c
/root/LARGE.TXT
```

如您所见，它列出了`LARGE.TXT`，因为它有零个字符；`0c`表示零个字符（或字节）。现在，如果您想在`/root`下搜索大小为`6`字节的文件，您可以运行：

```
root@ubuntu-linux:~# find /root -size 6c
/root/large.txt
```

如您所见，它列出了文件`large.txt`。

您甚至可以在搜索中使用大小范围；`Table 16`向您展示了使用`find`命令的大小范围的一些示例。

| **命令** | **作用** |
| --- | --- |
| `find / -size +100M` | 将搜索所有大于`100` MB 的文件。 |
| `find / -size -5c` | 将搜索所有小于`5`字节的文件。 |
| `find / -size +50M -size -100M` | 将搜索所有大于`50` MB 但小于`100` MB 的文件。 |
| `find / -size +1G` | 将搜索所有大于`1` GB 的文件。 |

表 16：使用大小范围

`-mtime`和`-atime`选项根据修改和访问时间搜索文件。`-exec`也是一个有用的命令选项，允许您对`find`结果运行另一个命令。

例如，您可以通过运行以下命令在`/root`中对所有空文件进行长列表：

```
root@ubuntu-linux:~# find /root -size 0c -exec ls -l {} +
-rw-r--r-- 1 root root 0 May 16 14:31 /root/LARGE.TXT
```

很多人在使用`-exec`选项时忘记包括`{} +`；`{} +`引用了在查找结果中找到的所有文件。

您可以在`-exec`选项中使用任何命令。例如，您可能希望删除从查找结果中获得的文件，而不是进行长列表。在这种情况下，您可以运行：

```
root@ubuntu-linux:~# find /root -size 0c -exec rm {} +
```

现在文件`LARGE.TXT`已被删除：

```
root@ubuntu-linux:~# ls -l LARGE.TXT
ls: cannot access 'LARGE.TXT': No such file or directory
```

我强烈建议您阅读`man`页面，以探索可以使用的众多其他选项。

# 知识检查

对于以下练习，打开您的终端并尝试解决以下任务：

1.  使用`locate`命令找到文件`boot.log`的路径。

1.  查找所有大小大于`50` MB 的文件。

1.  查找所有大小在`70` MB 和`100` MB 之间的文件。

1.  查找所有属于用户`smurf`的文件。

1.  查找所有属于组`developers`的文件。
