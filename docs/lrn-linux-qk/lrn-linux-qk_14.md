Sudo 的威力

在本章中，您将学习如何为系统上的非 root 用户授予权限，以便他们可以运行特权命令。在现实生活中，系统管理员不应该将 root 密码给系统上的任何用户。但是，系统上的一些用户可能需要运行特权命令；现在的问题是：*非 root 用户如何可以运行特权命令而不获取对系统的 root 访问权限？*好吧，让我来告诉你！

# 第十五章：特权命令的示例

您会发现大多数需要 root 权限的命令在`/sbin`和`/usr/sbin`目录中。让我们切换到用户`smurf`：

```
elliot@ubuntu-linux:~$ su - smurf 
Password:
smurf@ubuntu-linux:~$
```

现在让我们看看`smurf`是否可以向系统添加新用户：

```
smurf@ubuntu-linux:~$ useradd bob 
useradd: Permission denied.
```

用户`smurf`收到了权限被拒绝的错误。那是因为`useradd`命令是一个特权命令。好吧！让我们尝试安装`terminator`软件包，我必须说这是一个非常酷的终端仿真器：

```
smurf@ubuntu-linux:~$ apt-get install terminator
E: Could not open lock file /var/lib/dpkg/lock-frontend - open
 (13: Permission denied)
E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), 
are you root?
```

再次！用户`smurf`遇到了错误。没有 root 权限生活就不好玩，我听到你在说。

# 使用 sudo 授予权限

用户`smurf`现在非常难过，因为他无法在系统上添加用户`bob`或安装`terminator`软件包。您可以使用`visudo`命令授予用户`smurf`运行他想要的两个特权命令的权限。

以 root 用户身份运行`visudo`命令：

```
root@ubuntu-linux:~# visudo
```

这将打开文件`/etc/sudoers`，以便您可以编辑它：

```
# This file MUST be edited with the 'visudo' command as root. 
#
# Please consider adding local content in /etc/sudoers.d/ instead of 
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file. 
#
Defaults           env_reset
Defaults           mail_badpass 
# Host alias specification 
# User alias specification 
# Cmnd alias specification
# User privilege specification 
root               ALL=(ALL:ALL) ALL
# Members of the admin group may gain root privileges
%admin             ALL=(ALL) ALL
# Allow members of group sudo to execute any command
%sudo              ALL=(ALL:ALL) ALL
# See sudoers(5) for more information on "#include" directives: 
#includedir /etc/sudoers.d
```

所有以井号字符开头的行都是注释，因此只关注这些行：

```
root   ALL=(ALL:ALL) ALL
%admin ALL=(ALL) ALL
%sudo  ALL=(ALL:ALL) ALL
```

第一行`root ALL=(ALL:ALL) ALL`是一个规则，授予用户`root`在系统上运行所有命令的权限。

现在我们可以添加一个规则，授予用户`smurf`运行`useradd`命令的权限。`/etc/sudoers`文件中规则的语法规范如下：

```
user hosts=(user:group) commands
```

现在将以下规则添加到`/etc/sudoers`文件中：

```
smurf    ALL=(ALL)       /usr/sbin/useradd
```

`ALL`关键字表示没有限制。请注意，您还必须包括命令的完整路径。现在，保存并退出文件，然后切换到用户`smurf`：

```
root@ubuntu-linux:~# su - smurf 
smurf@ubuntu-linux:~$
```

现在在`useradd`命令之前加上`sudo`，如下所示：

```
smurf@ubuntu-linux:~$ sudo useradd bob 
[sudo] password for smurf: 
smurf@ubuntu-linux:~$ 
```

它将提示用户`smurf`输入密码；输入密码，就这样！用户`bob`已添加：

```
smurf@ubuntu-linux:~$ id bob
uid=1005(bob) gid=1005(bob) groups=1005(bob) 
smurf@ubuntu-linux:~$
```

酷！所以`smurf`现在可以向系统添加用户；但是，他仍然无法在系统上安装任何软件包：

```
smurf@ubuntu-linux:~$ sudo apt-get install terminator
Sorry, user smurf is not allowed to execute '/usr/bin/apt-get install 
terminator' as root on ubuntu-linux.
```

现在让我们来修复这个问题。切换回 root 用户，并运行`visudo`命令来编辑用户`smurf`的`sudo`规则：

```
smurf ALL=(ALL) NOPASSWD: /usr/sbin/useradd, /usr/bin/apt-get install terminator
```

请注意，我还添加了`NOPASSWD`，这样`smurf`就不会被提示输入密码。我还添加了安装`terminator`软件包的命令。现在，保存并退出，然后切换回用户`smurf`，尝试安装`terminator`软件包：

```
smurf@ubuntu-linux:~$ sudo apt-get install terminator 
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required: 
 gsfonts-x11 java-common
Use 'sudo apt autoremove' to remove them. 
The following NEW packages will be installed:
 terminator
```

成功！请注意，`sudo`规则只授予`smurf`安装`terminator`软件包的权限。如果他尝试安装其他软件包，他将收到错误提示：

```
smurf@ubuntu-linux:~$ sudo apt-get install cmatrix
Sorry, user smurf is not allowed to execute '/usr/bin/apt-get install cmatrix' 
as root on ubuntu-linux.
```

# 用户和命令别名

您可以使用用户别名在`/etc/sudoers`文件中引用多个用户。例如，您可以创建一个名为`MANAGERS`的用户别名，其中包括`usersmurf`和`bob`，如下所示：

```
User_Alias MANAGERS = smurf,bob
```

您可以使用命令别名将多个命令分组在一起。例如，您可以创建一个名为`USER_CMDS`的命令别名，其中包括`useradd`、`userdel`和`usermod`命令：

```
Cmnd_Alias USER_CMDS = /usr/sbin/useradd, /usr/sbin/userdel, /usr/sbin/usermod
```

现在您可以同时使用别名：

```
MANAGERS ALL=(ALL) USER_CMDS
```

授予用户`smurf`和`bob`运行`useradd`、`userdel`和`usermod`命令的权限。

# 组权限

您还可以在`/etc/sudoers`文件中指定组。组名前面加上百分号字符，如下所示：

```
%group hosts=(user:group) commands
```

以下规则将授予`developers`组在系统上安装任何软件包的权限：

```
%developers ALL=(ALL) NOPASSWD: /usr/bin/apt-get install
```

以下规则将授予`developers`组在系统上运行任何命令的权限：

```
%developers ALL=(ALL) NOPASSWD: ALL
```

# 列出用户权限

您可以使用命令`sudo -lU`来显示用户可以运行的`sudo`命令列表：

```
sudo -lU username 
```

例如，您可以运行以下命令：

```
root@ubuntu-linux:~# sudo -lU smurf
Matching Defaults entries for smurf on ubuntu-linux: 
 env_reset, mail_badpass

User smurf may run the following commands on ubuntu-linux:
 (ALL) NOPASSWD: /usr/sbin/useradd, /usr/bin/apt-get install terminator
```

列出用户`smurf`可以运行的所有`sudo`命令。

如果用户不被允许运行任何`sudo`命令，则`sudo-lU`命令的输出将如下所示：

```
root@ubuntu-linux:~# sudo -lU rachel
User rachel is not allowed to run sudo on ubuntu-linux.
```

# visudo 与/etc/sudoers

您可能已经注意到，我使用`visudo`命令编辑文件`/etc/sudoers`，您可能会问自己一个非常合理的问题：为什么不直接编辑文件`/etc/sudoers`而不使用`visudo`？好吧，我将以实际的方式回答您的问题。

首先，运行`visudo`命令并添加以下行：

```
THISLINE=WRONG
```

现在尝试保存并退出：

```
root@ubuntu-linux:~# visudo
>>> /etc/sudoers: syntax error near line 14 <<< 
What now?
Options are:
 (e)dit sudoers file again
 e(x)it without saving changes to sudoers file 
 (Q)uit and save changes to sudoers file (DANGER!)
What now?
```

正如您所看到的，`visudo`命令检测到错误，并指定了错误发生的行号。

为什么这很重要？好吧，如果您保存了带有错误的文件，`/etc/sudoers`中的所有`sudo`规则都将无法工作！让我们按`Q`保存更改，然后尝试列出用户`smurf`可以运行的`sudo`命令：

```
What now? Q
root@ubuntu-linux:~# sudo -lU smurf
>>> /etc/sudoers: syntax error near line 14 <<< 
sudo: parse error in /etc/sudoers near line 14 
sudo: no valid sudoers sources found, quitting 
sudo: unable to initialize policy plugin
```

我们遇到了一个错误，所有的`sudo`规则现在都被破坏了！返回并运行`visudo`命令，删除包含错误的行。

如果您直接编辑文件`/etc/sudoers`而不使用`visudo`命令，它将不会检查语法错误，这可能会导致灾难性后果，就像您看到的那样。因此，这里的经验法则是：在编辑`/etc/sudoers`文件时始终使用`visudo`。

# 知识检查

对于以下练习，打开您的终端并尝试解决以下任务：

1.  添加一个`sudo`规则，使用户`smurf`可以运行`fdisk`命令。

1.  添加一个`sudo`规则，使`developers`组可以运行`apt-get`命令。

1.  列出用户`smurf`的所有`sudo`命令。
