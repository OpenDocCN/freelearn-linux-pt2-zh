Bash 脚本很有趣

在 Linux 中完成特定任务时，你经常会发现自己一遍又一遍地运行相同的一组命令。这个过程会浪费很多宝贵的时间。在本章中，你将学习如何创建 bash 脚本，以便在 Linux 中更加高效。

# 第十七章：创建简单脚本

我们的第一个 bash 脚本将是一个简单的脚本，它将在屏幕上输出一行“你好，朋友！”。在艾略特的主目录中，创建一个名为`hello.sh`的文件，并插入以下两行：

```
elliot@ubuntu-linux:~$ cat hello.sh 
#!/bin/bash
echo "Hello Friend!"
```

现在我们需要将脚本设置为可执行：

```
elliot@ubuntu-linux:~$ chmod a+x hello.sh
```

最后，运行脚本：

```
elliot@ubuntu-linux:~$ ./hello.sh 
Hello Friend!
```

恭喜！你现在已经创建了你的第一个 bash 脚本！让我们花一分钟时间讨论一些事情；每个 bash 脚本必须做到以下几点：

+   `#!/bin/bash`

+   要可执行

你必须在任何 bash 脚本的第一行插入`#!/bin/bash`；字符序列`#!`被称为 shebang 或 hashbang，后面跟着 bash shell 的路径。

# PATH 变量

你可能已经注意到我使用了`./hello.sh`来运行脚本；如果省略前导的`./`，你会得到一个错误：

```
elliot@ubuntu-linux:~$ hello.sh 
hello.sh: command not found
```

shell 找不到命令`hello.sh`。当你在终端上运行一个命令时，shell 会在存储在`PATH`变量中的一组目录中寻找该命令。

你可以使用`echo`命令查看你的`PATH`变量的内容：

```
elliot@ubuntu-linux:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

冒号字符分隔每个目录的路径。你不需要包括这些目录中任何命令或脚本（或任何可执行文件）的完整路径。到目前为止，你学到的所有命令都驻留在`/bin`和`/sbin`中，它们都存储在你的`PATH`变量中。因此，你可以运行`pwd`命令：

```
elliot@ubuntu-linux:~$ pwd
/home/elliot
```

没有必要包含它的完整路径：

```
elliot@ubuntu-linux:~$ /bin/pwd
/home/elliot
```

好消息是你可以很容易地将一个目录添加到你的`PATH`变量中。例如，要将`/home/elliot`添加到你的`PATH`变量中，你可以使用`export`命令如下：

```
elliot@ubuntu-linux:~$ export PATH=$PATH:/home/elliot
```

现在你不需要前导的`./`来运行`hello.sh`脚本：

```
elliot@ubuntu-linux:~$ hello.sh 
Hello Friend!
```

它将运行，因为 shell 现在也在`/home/elliot`目录中寻找可执行文件：

```
elliot@ubuntu-linux:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/elliot
```

好了！现在让我们创建几个更多的 bash 脚本。我们将创建一个名为`hello2.sh`的脚本，它打印出“你好，朋友！”然后显示你当前的工作目录：

```
elliot@ubuntu-linux:~$ cat hello2.sh 
#!/bin/bash
echo "Hello Friend!" 
pwd
```

现在让我们运行它：

```
elliot@ubuntu-linux:~$ hello2.sh
-bash: /home/elliot/hello2.sh: Permission denied
```

糟糕！我忘记了要将其设置为可执行：

```
elliot@ubuntu-linux:~$ chmod a+x hello2.sh 
elliot@ubuntu-linux:~$ ./hello2.sh
Hello Friend!
/home/elliot
```

# 读取用户输入

让我们创建我们的`hello.sh`脚本的更好版本。我们将让用户输入他/她的名字，然后我们将向用户打招呼；创建一个名为`greet.sh`的脚本，包含以下几行：

```
elliot@ubuntu-linux:~$ cat greet.sh 
#!/bin/bash
echo "Please enter your name:" 
read name
echo "Hello $name!"
```

现在让脚本可执行，然后运行它：

```
elliot@ubuntu-linux:~$ chmod a+x greet.sh 
elliot@ubuntu-linux:~$ ./greet.sh
Please enter your name:
```

当你运行脚本时，它会提示你输入你的名字；我输入了`Elliot`作为我的名字：

```
elliot@ubuntu-linux:~$ ./greet.sh 
Please enter your name:
Elliot
Hello Elliot!
```

脚本向我打招呼说“你好，艾略特！”。我们使用`read`命令获取用户输入，并注意在`echo`语句中，我们使用了美元符号`$`来打印变量`name`的值。

让我们创建另一个脚本，从用户那里读取文件名，然后输出文件的大小（以字节为单位）；我们将命名我们的脚本为`size.sh`：

```
elliot@ubuntu-linux:~$ cat size.sh
#!/bin/bash
echo "Please enter a file path:" 
read file
filesize=$(du -bs $file| cut -f1)
echo "The file size is $filesize bytes"
```

并且永远不要忘记将脚本设置为可执行：

```
elliot@ubuntu-linux:~$ chmod a+x size.sh
```

现在让我们运行脚本：

```
elliot@ubuntu-linux:~$ size.sh 
Please enter a file path
/home/elliot/size.sh
The file size is 128 bytes
```

我使用`size.sh`作为文件路径，输出为 128 字节；是真的吗？让我们来检查一下：

```
elliot@ubuntu-linux:~$ du -bs size.sh
128 size.sh
```

的确如此；请注意脚本中的以下一行：

```
filesize=$(du -bs $file| cut -f1)
```

它将`du -bs $file | cut -f1`命令的结果存储在变量`filesize`中：

```
elliot@ubuntu-linux:~$ du -bs size.sh | cut -f1 
128
```

还要注意，命令`du -bs $file cut -f1`被括号和美元符号（在左边）包围；这被称为命令替换。一般来说，命令替换的语法如下：

```
var=$(command)
```

`command`的结果将存储在变量`var`中。

# 向脚本传递参数

除了从用户那里读取输入，你还可以将参数传递给 bash 脚本。例如，让我们创建一个名为`size2.sh`的 bash 脚本，它做的事情与脚本`size.sh`相同，但是不是从用户那里读取文件，而是将其作为参数传递给脚本`size2.sh`：

```
elliot@ubuntu-linux:~$ cat size2.sh 
#!/bin/bash
filesize=$(du -bs $1| cut -f1)
echo "The file size is $filesize bytes"
```

现在让我们将脚本设置为可执行：

```
elliot@ubuntu-linux:~$ chmod a+x size2.sh
```

最后，你可以运行脚本：

```
elliot@ubuntu-linux:~$ size2.sh /home/elliot/size.sh 
The file size is 128 bytes
```

你将得到与`size.sh`相同的输出。注意我们提供了文件路径

`/home/elliot/size.sh`作为脚本`size2.sh`的参数。

我们在脚本`size2.sh`中只使用了一个参数，它由`$1`引用。你也可以传递多个参数；让我们创建另一个脚本`size3.sh`，它接受两个文件（两个参数）并输出每个文件的大小：

```
elliot@ubuntu-linux:~$ cat size3.sh #!/bin/bash
filesize1=$(du -bs $1| cut -f1) 
filesize2=$(du -bs $2| cut -f1) 
echo "$1 is $filesize1 bytes" 
echo "$2 is $filesize2 bytes"
```

现在使脚本可执行并运行它：

```
elliot@ubuntu-linux:~$ size3.sh /home/elliot/size.sh /home/elliot/size3.sh
/home/elliot/size.sh is 128 bytes
/home/elliot/size3.sh is 136 bytes
```

太棒了！如你所见，第一个参数由`$1`引用，第二个参数由`$2`引用。所以一般来说：

```
bash_script.sh argument1 argument2 argument3 ...
 $1         $2         $3
```

# 使用 if 条件

你可以通过使其在不同的情况下表现不同来为你的 bash 脚本增加智能。为此，我们使用条件`if`语句。

一般来说，`if 条件`的语法如下：

```
if [ condition is true ]; then 
    do this ...
fi
```

例如，让我们创建一个名为`empty.sh`的脚本，它将检查文件是否为空：

```
elliot@ubuntu-linux:~$ cat empty.sh 
#!/bin/bash
filesize=$(du -bs $1 | cut -f1) 
if [ $filesize -eq 0 ]; then 
echo "$1 is empty!"
fi
```

现在让我们使脚本可执行，并创建一个名为`zero.txt`的空文件：

```
elliot@ubuntu-linux:~$ chmod a+x empty.sh 
elliot@ubuntu-linux:~$ touch zero.txt
```

现在让我们在文件`zero.txt`上运行脚本：

```
elliot@ubuntu-linux:~$ ./empty.sh zero.txt 
zero.txt is empty!
```

如你所见，脚本正确地检测到`zero.txt`是一个空文件；这是因为在这种情况下测试条件为真，因为文件`zero.txt`的确是零字节大小的：

```
if [ $filesize -eq 0 ];
```

我们使用了`-eq`来测试相等。现在如果你在一个非空文件上运行脚本，将不会有输出：

```
elliot@ubuntu-linux:~$ ./empty.sh size.sh 
elliot@ubuntu-linux:~$
```

我们需要修改脚本`empty.sh`，以便在传递非空文件时显示输出；为此，我们将使用`if-else`语句：

```
if [ condition is true ]; then 
    do this ...
else
    do this instead ...
fi
```

让我们通过添加以下`else`语句来编辑`empty.sh`脚本：

```
elliot@ubuntu-linux:~$ cat empty.sh 
#!/bin/bash
filesize=$(du -bs $1 | cut -f1) 
if [ $filesize -eq 0 ]; then 
echo "$1 is empty!"
else
echo "$1 is not empty!" 
fi
```

现在让我们重新运行脚本：

```
elliot@ubuntu-linux:~$ ./empty.sh size.sh 
size.sh is not empty!
elliot@ubuntu-linux:~$ ./empty.sh zero.txt 
zero.txt is empty!
```

如你所见，现在它完美地运行了！

你也可以使用`elif`（**else-if**）语句来创建多个测试条件：

```
if [ condition is true ]; then 
    do this ...
elif [ condition is true]; then 
    do this instead ...
fi
```

让我们创建一个名为`filetype.sh`的脚本，它检测文件类型。脚本将输出文件是普通文件、软链接还是目录：

```
elliot@ubuntu-linux:~$ cat filetype.sh 
#!/bin/bash
file=$1
if [ -f $1 ]; then
echo "$1 is a regular file" 
elif [ -L $1 ]; then
echo "$1 is a soft link" 
elif [ -d $1 ]; then 
echo "$1 is a directory" 
fi
```

现在让我们使脚本可执行，并创建一个指向`/tmp`的软链接，名为`tempfiles`：

```
elliot@ubuntu-linux:~$ chmod a+x filetype.sh 
elliot@ubuntu-linux:~$ ln -s /tmp tempfiles
```

现在在任何目录上运行脚本：

```
elliot@ubuntu-linux:~$ ./filetype.sh /bin
/bin is a directory
```

它正确地检测到`/bin`是一个目录。现在在任何普通文件上运行脚本：

```
elliot@ubuntu-linux:~$ ./filetype.sh zero.txt 
zero.txt is a regular file
```

它正确地检测到`zero.txt`是一个普通文件。最后，在任何软链接上运行脚本：

```
elliot@ubuntu-linux:~$ ./filetype.sh tempfiles 
tempfiles is a soft link
```

它正确地检测到`tempfiles`是一个软链接。

以下`man`页面包含了所有的测试条件：

```
elliot@ubuntu-linux:~$ man test
```

所以永远不要死记硬背！利用并使用 man 页面。

# 在 bash 脚本中循环

循环的能力是 bash 脚本的一个非常强大的特性。例如，假设你想要在终端上打印出"Hello world"这一行 20 次；一个天真的方法是创建一个有 20 个`echo`语句的脚本。幸运的是，循环提供了一个更聪明的解决方案。

## 使用 for 循环

`for`循环有几种不同的语法。如果你熟悉 C++或 C 编程，那么你会认出以下`for`循环的语法：

```
for ((initialize ; condition ; increment)); do
// do something 
done
```

使用前面提到的 C 风格语法；以下`for`循环将打印出"Hello World"二十次：

```
for ((i = 0 ; i < 20 ; i++)); do 
    echo "Hello World"
done
```

循环将整数变量`i`初始化为`0`，然后测试条件（`i < 20`）；如果为真，则执行 echo "Hello World"这一行，并递增变量`i`一次，然后循环再次运行，直到`i`不再小于`20`。

现在让我们创建一个名为`hello20.sh`的脚本，其中包含我们刚讨论的`for`循环：

```
elliot@ubuntu-linux:~$ cat hello20.sh 
#!/bin/bash
for ((i = 0 ; i < 20 ; i++)); do 
 echo "Hello World"
done
```

现在使脚本可执行并运行它：

```
elliot@ubuntu-linux:~$ chmod a+x hello20.sh 
elliot@ubuntu-linux:~$ hello20.sh
Hello World 
Hello World
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World 
Hello World
```

它输出了"Hello World"这一行二十次，正如我们所预期的那样。除了 C 风格的语法，你也可以在`for`循环中使用范围语法：

```
for i in {1..20}; do 
    echo "Hello World"
done
```

这也将输出"Hello World"20 次。这种范围语法在处理文件列表时特别有用。为了演示，创建以下五个文件：

```
elliot@ubuntu-linux:~$ touch one.doc two.doc three.doc four.doc five.doc
```

现在假设我们想要将所有五个文件的扩展名从`.doc`改为

`.document`。我们可以创建一个名为`rename.sh`的脚本，其中包含以下`for`循环：

```
#!/bin/bash
for i in /home/elliot/*.doc; do
    mv $i $(echo $i | cut -d. -f1).document
done
```

使脚本可执行并运行它：

```
#!/bin/bash
elliot@ubuntu-linux:~$ chmod a+x rename.sh 
elliot@ubuntu-linux:~$ ./rename.sh 
elliot@ubuntu-linux:~$ ls *.document
five.document four.document one.document three.document two.document
```

正如你所看到的，它将所有扩展名为`.doc`的文件重命名为`.document`。现在想象一下，如果你想对一百万个文件执行此操作。如果你不懂 bash 脚本，你可能要花十年的时间。我们都应该感谢 Linux 之神的 bash 脚本。

## 使用`while`循环

`while`循环是另一个流行且直观的循环。`while`循环的一般语法如下：

```
while [ condition is true ]; do
  // do something 
done
```

例如，我们可以创建一个简单的脚本`numbers.sh`，打印从一到十的数字：

```
elliot@ubuntu-linux:~$ cat numbers.sh 
#!/bin/bash
number=1
while [ $number -le 10 ]; do 
echo $number 
number=$(($number+1))
done
```

使脚本可执行并运行它：

```
elliot@ubuntu-linux:~$ chmod a+x numbers.sh 
elliot@ubuntu-linux:~$ ./numbers.sh
1
2
3
4
5
6
7
8
9
10
```

脚本很容易理解；我们首先将变量 number 初始化为`1`：

```
number=1
```

然后我们创建了一个测试条件，只要变量`number`小于或等于 10，`while`循环将继续运行：

```
while [ $number -le 10 ]; do
```

在`while`循环的主体中，我们首先打印变量`number`的值，然后将其增加 1。请注意，要评估算术表达式，它需要在双括号内，如`$((arithmetic-expression))`：

```
echo $number 
number=$(($number+1))
```

现在是时候玩一些有趣的东西了！我们将创建一个猜数字游戏。但在我们开始之前，让我向你介绍一个非常酷的命令。你可以使用`shuf`命令来生成随机排列。例如，要生成 1 到 10 之间数字的随机排列，你可以运行以下命令：

```
elliot@ubuntu-linux:~$ shuf -i 1-10 
1
6
5
2
10
8
3
9
7
4
```

请记住，我的输出很可能与你的输出不同，因为它是随机的！你有一百万分之一的机会和我有相同的输出。

现在我们可以使用`-n`选项从排列中选择一个数字。这个数字也将是随机的。因此，要生成 1 到 10 之间的随机数，你可以运行以下命令：

```
elliot@ubuntu-linux:~$ shuf -i 1-10 -n 1
6
```

输出将是 1 到 10 之间的一个随机数。`shuf`命令将在我们的游戏中发挥关键作用。我们将生成 1 到 10 之间的随机数，然后我们将看看用户（玩家）猜中随机数需要多少次尝试。

这是我们精心制作的脚本`game.sh`：

```
elliot@ubuntu-linux:~$ cat game.sh 
#!/bin/bash
random=$(shuf -i 1-10 -n 1) #generate a random number between 1 and 10\. 
echo "Welcome to the Number Guessing Game"
echo "The lucky number is between 1 and 10." 
echo "Can you guess it?"
tries=1
while [ true ]; do
echo -n "Enter a Number between 1-10: " 
read number
if [ $number -gt $random ]; then 
echo "Too high!"
elif [ $number -lt $random ]; then 
echo "Too low!"
else
echo "Correct! You got it in $tries tries" 
break #exit the loop
fi 
tries=$(($tries+1)) 
done
```

现在使脚本可执行并运行它来开始游戏：

```
elliot@ubuntu-linux:~$ chmod a+x game.sh 
elliot@ubuntu-linux:~$ game.sh
Welcome to the Number Guessing Game 
The lucky number is between 1 and 10\. 
Can you guess it?
Enter a Number between 1-10: 4 
Too low!
Enter a Number between 1-10: 7 
Too low!
Enter a Number between 1-10: 9 
Too high!
Enter a Number between 1-10: 8 
Correct! You got it in 4 tries
```

在我的第一次尝试游戏中，我猜了四次；我打赌你可以轻松地击败我！

让我们逐行查看我们的游戏脚本。我们首先生成一个 1 到 10 之间的随机数，并将其赋值给变量`random`：

```
random=$(shuf -i 1-10 -n 1) #generate a random number between 1 and 10.
```

请注意，你可以在你的 bash 脚本中添加注释，就像我在这里使用井号字符，后面跟上你的注释一样。

然后我们打印三行来向玩家解释游戏规则：

```
echo "Welcome to the Number Guessing Game" 
echo "The lucky number is between 1 and 10." 
echo "Can you guess it?"
```

接下来，我们将变量`tries`初始化为`1`，以便我们可以跟踪玩家猜了多少次：

```
tries=1
```

然后我们进入游戏循环：

```
while [ true ]; do
```

请注意，测试条件`while [ true ]`将始终为`true`，因此循环将永远运行（无限循环）。

游戏循环中我们做的第一件事是要求玩家输入 1 到 10 之间的数字：

```
echo -n "Enter a Number between 1-10: " 
read number
```

然后我们测试玩家输入的数字是大于、小于还是等于`random`数字：

```
if [ $number -gt $random ]; then 
echo "Too high!"
elif [ $number -lt $random ]; then 
echo "Too low!"
else
echo "Correct! You got it in $tries tries" 
break #exit the loop
fi
```

如果`number`大于`random`，我们告诉玩家猜测太高，以便玩家下次更容易猜对。同样，如果`number`小于`random`，我们告诉玩家猜测太低。否则，如果是正确的猜测，那么我们打印玩家用来做出正确猜测的总次数，并且我们从循环中退出。

请注意，你需要`break`语句来退出无限循环。没有`break`语句，循环将永远运行。

最后，我们每次猜错（高或低）都会将`tries`的数量增加 1：

```
tries=$(($tries+1))
```

我必须警告你，这个游戏很容易上瘾！特别是当你和朋友一起玩时，看谁能在最少的尝试次数中猜对。

## 使用`until`循环

`for`和`while`循环都会在测试条件为`true`时运行。相反，`until`循环会在测试条件为`false`时继续运行。也就是说，它会在测试条件为`true`时停止运行。

`until`循环的一般语法如下：

```
until [condition is true]; do 
  [commands]
done
```

例如，我们可以创建一个简单的脚本`3x10.sh`，打印出`3`的前十个倍数：

```
elliot@ubuntu-linux:~$ cat 3x10.sh 
#!/bin/bash
counter=1
until [ $counter -gt 10 ]; do 
echo $(($counter * 3)) 
counter=$(($counter+1))
done
```

现在让脚本可执行，然后运行它：

```
elliot@ubuntu-linux:~$ chmod a+x 3x10.sh 
elliot@ubuntu-linux:~$ 3x10.sh
3
6
9
12
15
18
21
24
27
30
```

脚本很容易理解，但你可能会在尝试理解`until`循环的测试条件时有点困惑：

```
until [ $counter -gt 10 ]; do
```

测试条件基本上是这样说的：“直到`counter`大于 10，继续运行！”

请注意，我们可以使用具有相反测试条件的`while`循环来实现相同的结果。你只需否定`until`循环的测试条件，就会得到`while`循环的等价形式：

```
while [ $counter -le 10 ]; do
```

在数学中，大于（`>`）的相反（否定）是小于或等于（`≤`）。很多人忘记了`等于`部分。不要成为那些人中的一个！

# Bash 脚本函数

当你的脚本变得越来越大时，事情可能会变得非常混乱。为了克服这个问题，你可以使用 bash 函数。函数的理念是你可以重用脚本的部分，从而产生更有组织和可读性的脚本。

bash 函数的一般语法如下：

```
function_name () {
<commands>
}
```

让我们创建一个名为`hello`的函数，打印出“Hello World”这一行。我们将`hello`函数放在一个名为`fun1.sh`的新脚本中：

```
elliot@ubuntu-linux:~$ cat fun1.sh 
#!/bin/bash

hello () {
echo "Hello World"
}

hello     # Call the function hello() 
hello     # Call the function hello() 
hello     # Call the function hello()
```

现在让脚本可执行，然后运行它：

```
elliot@ubuntu-linux:~$ chmod a+x fun1.sh 
elliot@ubuntu-linux:~$ ./fun1.sh
Hello World 
Hello World 
Hello World
```

该脚本将在终端上输出“Hello World”三次。请注意，我们调用（使用）了函数`hello`三次。

## 传递函数参数

函数也可以像脚本一样接受参数。为了演示，我们将创建一个名为`math.sh`的脚本，其中有两个函数`add`和`sub`：

```
elliot@ubuntu-linux:~$ cat math.sh 
#!/bin/bash

add () {
echo "$1 + $2 =" $(($1+$2))
}

sub () {
echo "$1 - $2 =" $(($1-$2))
}

add 7 2
sub 7 2
```

使脚本可执行，然后运行它：

```
elliot@ubuntu-linux:~$ chmod a+x math.sh 
elliot@ubuntu-linux:~$ ./math.sh
7 + 2 = 9
7 - 2 = 5
```

该脚本有两个函数`add`和`sub`。`add`函数计算并输出任意两个数字的总和。另一方面，`sub`函数计算并输出任意两个数字的差。

# 你不能浏览网页

我们将用一个相当酷的 bash 脚本`noweb.sh`来结束本章，确保没有用户在 Firefox 浏览器上浏览网页时玩得开心：

```
elliot@ubuntu-linux:~$ cat noweb.sh 
#!/bin/bash

shutdown_firefox() { 
killall firefox 2> /dev/null
}

while [ true ]; do 
shutdown_firefox
sleep 10 #wait for 10 seconds 
done
```

现在将 Firefox 作为后台进程打开：

```
elliot@ubuntu-linux:~$ firefox & 
[1] 30436
```

最后，使脚本可执行，并在后台运行脚本：

```
elliot@ubuntu-linux:~$ chmod a+x noweb.sh 
elliot@ubuntu-linux:~$ ./noweb.sh &
[1] 30759
```

一旦运行你的脚本，Firefox 就会关闭。此外，如果以`root`用户身份运行脚本，系统用户将无法享受 Firefox！

# 知识检查

对于以下练习，打开你的终端并尝试解决以下任务：

1.  创建一个 bash 脚本，显示当前月份的日历。

1.  修改你的脚本，以便显示任何年份（作为参数传递）的日历。

1.  修改你的脚本，以便显示从`2000`年到`2020`年的所有年份的日历。
