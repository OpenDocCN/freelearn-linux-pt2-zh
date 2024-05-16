# 前言

Linux+认证证明了技术能力，并提供了对 Linux 操作系统的广泛认识。获得 Linux+认证的专业人士展示了对安装，操作，管理和故障排除服务的重要知识。

CompTIA Linux+认证指南是对该认证的概述，让您深入了解系统架构。您将了解如何安装和卸载 Linux 发行版，然后使用各种软件包管理器。一旦您掌握了所有这些，您将继续在命令行界面（CLI）操作文件和进程，并创建，监视，终止，重新启动和修改进程。随着您的进步，您将能够使用显示管理器，并学习如何创建，修改和删除用户帐户和组，以及了解如何自动执行任务。最后一组章节将帮助您配置日期并设置本地和远程系统日志记录。除此之外，您还将探索不同的互联网协议，以及发现网络配置，安全管理，Shell 脚本和 SQL 管理。

通过本书，您不仅将通过练习问题和模拟考试来掌握所有模块，而且还将为通过 LX0-103 和 LX0-104 认证考试做好充分准备。

# 本书适合人群

CompTIA Linux+认证指南适用于希望获得 CompTIA Linux+证书的人。本指南还适用于系统管理员和新手 Linux 专业人士，他们有兴趣提高他们的 Linux 和 Shell 脚本技能。不需要 Linux 的先前知识，尽管对 Shell 脚本的一些了解会有所帮助。

# 本书涵盖的内容

第一章，*配置硬件设置*，本章重点介绍查看中断，查看`/proc/interrupts`，查看 CPU 信息，查看`/proc/cpuinfo`，查看 raid 状态，查看`/proc/mdstat`，设备目录`/dev`，`/proc`虚拟目录，`lsmod`命令和用法，`modprobe`命令和用法，`lspci`命令和用法。

第二章，*系统引导*，本章重点介绍系统引导的过程，查看 GRUB 和 GRUB2 配置文件，重点关注计时器，默认引导条目，在 GRUB/GRUB2 引导菜单中传递参数，`chkconfig`命令，systemctl，各种启动/停止`scripts0`。

第三章，*更改运行级别和引导目标*，本章重点介绍运行级别和引导目标的介绍，LINUX 发行版中可用的运行级别和引导目标的类型，运行级别和引导目标之间的区别，在 CLI 中使用运行级别，还在 CLI 中使用引导目标。

第四章，*设计硬盘布局*，本章重点介绍在 CLI 上创建分区/分割物理硬盘，重点介绍`fdisk`实用程序的使用，`parted`实用程序，创建，删除，定义分区类型，使用各种`mkfs`命令格式化硬盘的步骤。

第五章，*安装 Linux 发行版*，本章重点介绍安装 Linux 发行版，特别是 CentOS 的 Red Hat 风味和 Ubuntu 的 Debian 风味，读者将通过使用 Live CD 的一种常见方法来安装 Linux 发行版。

第六章，使用 Debian 软件包管理，在 Linux 中，软件可以通过多种方式添加、删除。这里重点介绍了在 Debian 发行版中添加软件的方式，特别是使用 CLI 中的 dpkg、apt-get、aptitude 命令，以及 GUI 中的 synaptic，读者将学习如何在 Debian 发行版中添加、删除、更新、擦除软件。

第七章，使用 YUM 软件包管理，在本章中，我们重点介绍了在 Red Hat 发行版中添加软件的方式，特别是使用 CLI 中的 yum、dnf、rpm 命令，以及 GUI 中的 yumex，读者将学习如何在 Red Hat 环境中添加、删除、更新和擦除软件。

第八章，执行文件管理，在本章中，读者将了解 Linux 提供的各种命令，这些命令是常见发行版用于在 CLI 中操作文件、进程的。这些命令可以分为几类：文件系统导航、文件操作、目录操作、文件位置和文件检查、CPU 硬件标识、进程优先级、操作进程的 CPU 优先级。

第九章，创建、监视、终止和重新启动进程，在 Linux 中，进程或多或少等同于正在运行的程序。init/systemd 是内核启动时运行的第一个进程。本章重点介绍了如何创建进程，监视现有进程的硬件使用情况，终止/杀死进程或在 CLI 中重新启动进程。

第十章，修改进程执行，有时您可能希望优先处理重要程序而不是其他程序，还可以将一些程序发送到后台，允许用户继续使用 shell 或将一些程序带到前台。本章重点介绍了使用 nice 和 renice、fg、bg 命令来实现这一点的方法。

第十一章，显示管理器，本章重点介绍了 Linux 发行版中可用的各种显示管理器，如 X 显示管理器（XDM）、KDE 显示管理器（KDM）、Gnome 显示管理器（GDM）、Light 显示管理器（LightDM），用于处理 GUI 登录，它们都使用 XDMCP - X 显示管理器控制协议，启动本地计算机的 X 服务器。

第十二章，管理用户和组帐户，本章重点介绍了用户和组管理，包括用户帐户创建、修改现有用户帐户、删除用户帐户、组创建、修改用户组、删除组，以及在管理用户和组时要考虑的最佳实践。重点是使用诸如 useradd、usermod、userdel、groupadd、groupmod、groupdel、who、w、change、passwd、last、whoami 等命令，以及配置文件，如/etc/passwd、/etc/shadow、/etc/group、/etc/skel 文件。

第十三章，自动化任务，本章重点介绍了在 Linux 环境中自动化常见的管理任务以及在设置给定任务的自动化时要考虑的常用方法。重点是使用诸如 crontab、at、atq、atrm、anacron 等命令，以及配置文件，如/etc/at.deny、/etc/at.allow、/etc/cron.{daily,hourly,monthly,weekly}、/etc/cron.allow、/etc/anacrontab。

第十四章，维护系统时间和日志，本章重点介绍配置日期和时间以及设置时区。此外，设置在 Linux 发行版中使用`rsyslog`，`logrotate`进行本地日志记录以及配置日志记录发送到远程`syslog`服务器进行管理的步骤。涵盖的命令包括`tzselect`，`tzconfig`，`date`，`journalctl`，目录包括`/etc/timezone`，`/etc/localtime`，`/usr/share/zoneinfo`，`/etc/logrotate.conf`，`/etc/logrotate.d/`，`/etc/systemd/journald.conf`，`/var/log/`，`/var/log/journal/`，`/etc/rsyslog.conf`。

第十五章，互联网协议基础，本章重点介绍互联网等网络如何工作的基本原理，通过解释两台计算机如何相互通信，我们深入研究互联网协议（IP）寻址，特别是 IPv4，IPv4 的各种类别，如 A 类，B 类，C 类，CIDR 表示法，然后我们看子网划分。接下来我们看一下 IPv6，IPv6 地址的格式，众所周知的 IPv6 地址，缩短 IPv6 地址的方法。

最后，我们看一下一些著名协议（如 UDP，TCP 和 ICMP）及其端口号之间的区别。

第十六章，网络配置和故障排除，本章重点介绍 Linux 环境中的基本网络配置，包括配置 IPv4 地址，子网掩码，默认网关。接下来我们看一下配置 IPv6 地址，默认网关，然后我们专注于配置客户端 DNS，最后我们专注于网络故障排除。命令如`ifup`，`ifdown`，`ifconfig`，`ip`，`ip link`，`ip route`，`route`，`ping`，`ping6`，`netstat`，`traceroute`，`traceroute6`，`tracepath`，`tracepath6`，`dig`，`host`，`hostname`。

第十七章，执行安全管理任务，本章重点介绍 Linux 环境中执行安全管理任务，重点放在设置主机安全，授予用户特殊权限与 sudoers，日期加密。涵盖的命令有`sudo`，`ssh-keygen`，`ssh-agent`，`ssh-add`，`gpg`，配置文件包括`/etc/sudoers`，`/etc/hosts.allow`，`/etc/hosts.deny`，`~/.ssh/id_rsa`，`~/.ssh/id_rsa.pub`，`/etc/ssh/ssh_host_rsa_key`，`~/.ssh/authorized_keys`，`/etc/ssh_known_hosts`。

第十八章，Shell 脚本和 SQL 数据管理，本章重点介绍 Linux 环境中的 Shell 脚本和 SQL 数据管理。首先，我们看一下编写脚本时的基本格式，识别脚本的解释器，配置脚本为可执行，使用`for`，`while`循环，`if`语句。然后我们将注意力集中在 SQL 数据管理上，我们涵盖基本的 SQL 命令，如`insert`，`update`，`select`，`delete`，`from`，`where`，`group by`，`order by`，`join`。

第十九章，模拟考试-1，这个模拟考试将包括现实世界的考试问题和答案。您将获得来自真实场景的示例，详细的指导和关键主题的权威覆盖。最近测试的现实考试问题，为您带来为 CompTIA LX0-103/LX0-104 考试做准备的最佳方法。

第二十章，模拟考试-2，这个模拟考试将包括现实世界的考试问题和答案。您将获得来自真实场景的示例，详细的指导和关键主题的权威覆盖。最近测试的现实考试问题，为您带来为 CompTIA LX0-103/LX0-104 考试做准备的最佳方法。

# 为了充分利用本书

假设一些读者可能对 Linux 操作系统有限或没有知识。还假设一些读者是 Linux 用户，但可能需要对与 Linux 环境交互有所恢复。

巩固每一章记忆的关键是获取各种 Linux 发行版的副本；即 CentOS、Fedora 和 Ubuntu。然后在虚拟环境中安装各种操作系统，如 VMware 或 VirtualBox。接下来，通过在各种 Linux 发行版中练习，跟随每一章（各章节相互独立，因此您可以选择任意一章来学习/练习），以更好地掌握每一章。在练习了各种章节之后，您将在 Linux 环境中变得更加高效；这将使您在混合环境中更加适应，其中既有 Windows 操作系统，也有 Linux 操作系统。

您可以在本书的第五章中的*安装 Linux 发行版*教程中跟随操作开始安装。

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789344493_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789344493_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“要实时查看 shell 中的运行级别，我们可以使用`runlevel`命令。”

代码块设置如下：

```
while <condition>
do
          <command1>
          <command2 
```

任何命令行输入或输出都以以下方式编写：

```
$[philip@localhost Desktop]$ who -r
run-level 5 2018-06-20 08:20 last=S
[philip@localhost Desktop]$ 
```

**粗体**：表示一个新术语，一个重要的词，或者你在屏幕上看到的词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“从管理面板中选择系统信息。”

警告或重要说明会以这种方式出现。

提示和技巧会以这种方式出现。
