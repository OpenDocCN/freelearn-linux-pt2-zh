# 第二章：系统引导

在上一章中，我们涵盖了我们日常管理的常见硬件设置。我们提到了一些命令，可以用来识别 Linux 系统中的硬件。本章将从那里继续，并进一步进行，这次重点是系统引导的过程。它查看了 GRUB 和 GRUB2 配置文件，重点关注了计时器、默认引导项以及向 GRUB/GRUB2 引导菜单传递参数。它还涵盖了 `chkconfig`、`pstree`、`ps`、`systemctl` 和 `dmeg` 命令，以及各种启动/停止脚本。

本章将涵盖以下主题：

+   解释引导过程

+   理解 GRUB 和 GRUB2

+   使用 GRUB

+   使用 GRUB2

# 解释引导过程

在 Linux 中，在启动过程中，会在硬盘上查找引导扇区。一旦找到引导扇区，它会搜索引导加载程序。引导加载程序然后加载引导管理器。在 Linux 中，这通常是 GRUB 或 GRUB2。在这个阶段之后，用户会看到一个引导菜单。最后，用户有机会选择要加载的操作系统或编辑现有条目。可用的选项通常是不同版本的 Linux 内核。有时，它可能是完全不同的 Linux 发行版。然而，在混合环境中，你可能会接触到另一个操作系统，比如 Microsoft Windows。

用户选择 Linux 内核后，根据 Linux 发行版的不同，会启动一个名为 `init` 的单个进程，它代表*初始化*。`init` 通常被称为*System V init* 或 SysV，因为 System V 是第一个商业 Unix 操作系统。大多数早期的 Linux 发行版与 System V 操作系统相同。用于管理 Linux 发行版的另一个守护进程称为 `systemd`，代表 System Management Daemon。以下是我们刚刚讨论的过程的简单流程：

*引导扇区 > 引导加载程序 > 引导菜单 => 操作系统加载*

在 Linux 中，你可能会遇到术语**守护进程**。请放心，这只是指一个进程。

在我们深入之前，让我们记住 `init` 和 `systemd` 之间最大的区别之一：`init` 逐个启动脚本，而 `systemd` 同时并行启动多个脚本。话虽如此，在使用 `init` 的 CentOS 5 系统上，以下是 `pstree` 命令的输出：

![](img/00041.gif)

从前面的输出中，我们可以看到所有起源于 `init` 的进程；因此，它们被视为子进程。

注意：出于简洁起见，一些输出已在整个章节中被省略。

我们还可以利用 `ps` 命令在我们的 CentOS 5 系统上查看 `init` 使用的实际进程号：

```
[philip@localhost Desktop]$ ps -aux
 Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
 USER PID %CPU  %MEM  VSZ RSS TTY STAT START TIME COMMAND
 root  1   0.3  0.1  19364 1524 ? Ss 05:48   0:01 /sbin/init
 root  2   0.0  0.0   0    0    ? S  05:48   0:00 [kthreadd]
 root  3   0.0  0.0   0    0    ? S  05:48   0:00 [migration/0]
 root  4   0.0  0.0   0    0    ? S  05:48   0:00 [ksoftirqd/0]
 root  5   0.0  0.0   0    0    ? S  05:48   0:00 [migration/0]
 root  6   0.0  0.0   0    0    ? S  05:48   0:00 [watchdog/0]
 root  7   0.2  0.0   0    0    ? S  05:48   0:00 [events/0]
 root  8   0.0  0.0   0    0    ? S  05:48   0:00 [cgroup]
 root  9   0.0  0.0   0    0    ? S  05:48   0:00 [khelper]
 root  10  0.0  0.0   0    0    ? S  05:48   0:00 [netns]
 root  11  0.0  0.0   0    0    ? S  05:48   0:00 [async/mgr]
 root  12  0.0  0.0   0    0    ? S  05:48   0:00 [pm]
 root  13  0.0  0.0   0    0    ? S  05:48   0:00 [sync_supers]
 root  14  0.0  0.0   0    0    ? S  05:48   0:00 [bdi-default]
 root  15  0.0  0.0   0    0    ? S  05:48   0:00 [kintegrityd/]
 root  16  0.5  0.0   0    0    ? S  05:48   0:01 [kblockd/0]
```

从前面的输出中，我们可以看到第一个启动的进程是 `PID 1`，它确实是 `init` 进程。

以下是我们可以与 `ps` 命令一起使用的一些选项：

```
[philip@localhost Desktop]$ ps --help
 ********* simple selection ********* ********* selection by list *********
 -A all processes -C by command name
 -N negate selection -G by real group ID (supports names)
 -a all w/ tty except session leaders -U by real user ID (supports names)
 -d all except session leaders -g by session OR by effective group name
 -e all processes -p by process ID
 T all processes on this terminal -s processes in the sessions given
 a all w/ tty, including other users -t by tty
 g OBSOLETE -- DO NOT USE -u by effective user ID (supports names)
 r only running processes U processes for specified users
 x processes w/o controlling ttys t by tty
 *********** output format ********** *********** long options ***********
 -o,o user-defined -f full --Group --User --pid --cols --ppid
 -j,j job control s signal --group --user --sid --rows --info
 -O,O preloaded -o v virtual memory --cumulative --format --deselect
 -l,l long u user-oriented --sort --tty --forest --version
 -F extra full X registers --heading --no-heading --context
 ********* misc options *********
 -V,V show version L list format codes f ASCII art forest
 -m,m,-L,-T,H threads S children in sum -y change -l format
 -M,Z security data c true command name -c scheduling class
 -w,w wide output n numeric WCHAN,UID -H process hierarchy
 [philip@localhost Desktop]$ 
```

现在，让我们把注意力转向 `systemd`。我们将在我们的 Linux 系统上运行 `pstree` 命令：

![](img/00042.jpeg)

从前面的输出中，我们可以看到系统生成的所有其他进程。这些被称为子进程。

我们也可以在 CentOS 7 发行版上运行 `pstree` 命令，并看到类似的结果：

```
[philip@localhost ~]$ pstree
 systemd─┬─ModemManager───2*[{ModemManager}]
 ├─NetworkManager─┬─dhclient
 │ └─3*[{NetworkManager}]
 ├─VGAuthService
 ├─abrt-watch-log
 ├─abrtd
 ├─accounts-daemon───2*[{accounts-daemon}]
 ├─alsactl
 ├─anacron
 ├─at-spi-bus-laun─┬─dbus-daemon───{dbus-daemon}
 │ └─3*[{at-spi-bus-laun}]
 ├─at-spi2-registr───2*[{at-spi2-registr}]
 ├─atd
 ├─auditd─┬─audispd─┬─sedispatch
 │ │ └─{audispd}
 │ └─{auditd}
 ├─avahi-daemon───avahi-daemon
 ├─chronyd
 ├─colord───2*[{colord}]
 ├─crond
 ├─cupsd
 ├─2*[dbus-daemon───{dbus-daemon}]
 ├─dbus-launch
 ├─dconf-service───2*[{dconf-service}]
 ├─dnsmasq───dnsmasq
```

在几乎所有较新的 Linux 发行版中，`systemd` 已经取代了 `init`。

现在，让我们使用 `ps` 命令查看 Linux 系统上 `systemd` 使用的进程号：

```
root@ubuntu:/home/philip# ps -aux
 USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
 root 1 0.0 0.5 185620 4996 ? Ss Jun19 0:05 /lib/systemd/systemd --system --d
 root 2 0.0 0.0 0 0 ? S Jun19 0:00 [kthreadd]
 root 3 0.0 0.0 0 0 ? S Jun19 0:06 [ksoftirqd/0]
 root 5 0.0 0.0 0 0 ? S< Jun19 0:00 [kworker/0:0H]
 root 7 0.0 0.0 0 0 ? S Jun19 0:06 [rcu_sched]
 root 8 0.0 0.0 0 0 ? S Jun19 0:00 [rcu_bh]
 root 9 0.0 0.0 0 0 ? S Jun19 0:00 [migration/0]
 root 10 0.0 0.0 0 0 ? S Jun19 0:00 [watchdog/0]
 root 11 0.0 0.0 0 0 ? S Jun19 0:00 [kdevtmpfs]
 root 12 0.0 0.0 0 0 ? S< Jun19 0:00 [netns]
 root 13 0.0 0.0 0 0 ? S< Jun19 0:00 [perf]
 root 14 0.0 0.0 0 0 ? S Jun19 0:00 [khungtaskd]
 root 15 0.0 0.0 0 0 ? S< Jun19 0:00 [writeback]
 root 16 0.0 0.0 0 0 ? SN Jun19 0:00 [ksmd]
 root 17 0.0 0.0 0 0 ? SN Jun19 0:01 [khugepaged]
 root 18 0.0 0.0 0 0 ? S< Jun19 0:00 [crypto]
 root 19 0.0 0.0 0 0 ? S< Jun19 0:00 [kintegrityd]
 root 20 0.0 0.0 0 0 ? S< Jun19 0:00 [bioset]
 root 21 0.0 0.0 0 0 ? S< Jun19 0:00 [kblockd]
 root 22 0.0 0.0 0 0 ? S< Jun19 0:00 [ata_sff]
 root 23 0.0 0.0 0 0 ? S< Jun19 0:00 [md]
 root 24 0.0 0.0 0 0 ? S< Jun19 0:00 [devfreq_wq]

Some output is omitted for the sake of brevity.
```

从前面的输出中，我们可以清楚地看到系统确实被列为第一个启动的进程。

`systemd` 模拟 `init`。例如，我们可以使用 `service` 命令启动/停止守护进程。

现在，为了查看在 Linux 发行版上启动的进程，我们可以在我们的 CentOS 7 发行版上运行 `chkconfig` 命令：

```
[philip@localhost Desktop]$ chkconfig
 NetworkManager 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 abrt-ccpp 0:off 1:off 2:off 3:on 4:off 5:on 6:off
 abrtd 0:off 1:off 2:off 3:on 4:off 5:on 6:off
 acpid 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 atd 0:off 1:off 2:off 3:on 4:on 5:on 6:off
 auditd 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 blk-availability 0:off 1:on 2:on 3:on 4:on 5:on 6:off
 bluetooth 0:off 1:off 2:off 3:on 4:on 5:on 6:off
 cpuspeed 0:off 1:on 2:on 3:on 4:on 5:on 6:off
 crond 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 cups 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 dnsmasq 0:off 1:off 2:off 3:off 4:off 5:off 6:off
 firstboot 0:off 1:off 2:off 3:on 4:off 5:on 6:off
 haldaemon 0:off 1:off 2:off 3:on 4:on 5:on 6:off
 htcacheclean 0:off 1:off 2:off 3:off 4:off 5:off 6:off
 httpd 0:off 1:off 2:off 3:off 4:off 5:off 6:off
 ip6tables 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 iptables 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 irqbalance 0:off 1:off 2:off 3:on 4:on 5:on 6:off
 kdump 0:off 1:off 2:off 3:on 4:on 5:on 6:off
 lvm2-monitor 0:off 1:on 2:on 3:on 4:on 5:on 6:off
 mdmonitor 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 messagebus 0:off 1:off 2:on 3:on 4:on 5:on 6:off
 netconsole 0:off 1:off 2:off 3:off 4:off 5:off 6:off
 netfs 0:off 1:off 2:off 3:on 4:on 5:on 6:off
 network 0:off 1:off 2:on 3:on 4:on 5:on 6:off
```

在前面的输出中，我们只显示使用`init`的守护程序。这对于运行原生`init`的系统非常有用，比如早期的 Linux 发行版。

以下是可以与使用`init`的旧 Linux 发行版一起传递的`chkconfig`命令的最常用选项：

| `--level levels` | 指定操作应适用于的运行级别。它以 0 到 6 的数字字符串给出。例如，`--level 35`指定运行级别 3 和 5。 |
| --- | --- |
| `--add name` | 此选项通过`chkconfig`添加一个新的服务进行管理。添加新服务时，`chkconfig`确保该服务在每个运行级别中都有启动或杀死条目。如果任何运行级别缺少这样的条目，`chkconfig`将根据`init`脚本中的默认值创建适当的条目。请注意，LSB 分隔的`INIT INFO`部分中的默认条目优先于`initscript`中的默认运行级别；如果存在任何`required-start`或`required-stop`条目，则脚本的启动和停止优先级将根据这些依赖关系进行调整。 |
| `--del name` | 服务从`chkconfig`管理中删除，并且`/etc/rc[0-6].d`中与之相关的任何符号链接都将被删除。请注意，将来安装此服务的软件包可能会运行`chkconfig --add`，这将重新添加这些链接。要禁用服务，请运行`chkconfig name off`。 |
| `--override name` | 如果服务名称的配置与在`/etc/chkconfig.d/name`中没有覆盖文件的情况下指定`--add`选项完全相同，并且现在`/etc/chkconfig.d/name`存在并且与基本`initscript`不同，这将更改服务名称的配置，使其遵循覆盖而不是基本配置。 |
| `--list name` | 此选项列出`chkconfig`知道的所有服务，以及它们在每个运行级别中是停止还是启动的。如果指定了名称，则只显示有关服务名称的信息。 |

为了查看在较新的 Linux 发行版中启动的守护程序，我们将使用`systemctl`命令：

```
[philip@localhost ~]$ systemctl
 add-requires hybrid-sleep reload-or-restart
 add-wants is-active reload-or-try-restart
 cancel is-enabled rescue
 cat is-failed reset-failed
 condreload isolate restart
 condrestart is-system-running set-default
 condstop kexec set-environment
 daemon-reexec kill set-property
 daemon-reload link show
 default list-dependencies show-environment
 delete list-jobs snapshot
 disable list-sockets start
 edit list-timers status
 emergency list-unit-files stop
 enable list-units suspend
 exit mask switch-root
 force-reload poweroff try-restart
 get-default preset unmask
 halt reboot unset-environment
 help reenable
 hibernate reload
 [philip@localhost ~]$ 
```

从前面的输出中，我们可以看到可以与`systemctl`命令一起传递的各种选项；我们将使用`systemctl`的`list-unit-files`选项：

```
[philip@localhost ~]$ systemctl list-unit-files
 UNIT FILE                           STATE
 proc-sys-fs-binfmt_misc.automount   static
 dev-hugepages.mount                 static
 dev-mqueue.mount                    static
 proc-fs-nfsd.mount                  static
 proc-sys-fs-binfmt_misc.mount       static
 sys-fs-fuse-connections.mount       static
 sys-kernel-config.mount             static
 sys-kernel-debug.mount              static
 tmp.mount                           disabled
 var-lib-nfs-rpc_pipefs.mount        static
 brandbot.path                       disabled
 cups.path                           enabled
 systemd-ask-password-console.path   static
 systemd-ask-password-plymouth.path  static
 systemd-ask-password-wall.path      static
```

为了简洁起见，省略了一些输出：

```
 umount.target                    static
 virt-guest-shutdown.target       static
 chrony-dnssrv@.timer             disabled
 fstrim.timer                     disabled
 mdadm-last-resort@.timer         static
 systemd-readahead-done.timer     indirect
 systemd-tmpfiles-clean.timer     static
392 unit files listed.
```

从前面的输出中，我们可以看到列出了 392 个单元。我们可以更具体地查找只启用/运行的服务：

```
[philip@localhost ~]$ systemctl list-unit-files | grep enabled
 cups.path                                   enabled
 abrt-ccpp.service                           enabled
 abrt-oops.service                           enabled
 abrt-vmcore.service                         enabled
 abrt-xorg.service                           enabled
 abrtd.service                               enabled
 accounts-daemon.service                     enabled
 atd.service                                 enabled
 auditd.service                              enabled
 autovt@.service                             enabled
 avahi-daemon.service                        enabled
 bluetooth.service                           enabled
 chronyd.service                             enabled
 crond.service                               enabled
 cups.service                                enabled
 dbus-org.bluez.service                      enabled
 dbus-org.fedoraproject.FirewallD1.service   enabled
 dbus-org.freedesktop.Avahi.service          enabled
 dbus-org.freedesktop.ModemManager1.service  enabled
 dbus-org.freedesktop.NetworkManager.service enabled
 dbus-org.freedesktop.nm-dispatcher.service  enabled
 display-manager.service                     enabled
 dmraid-activation.service                   enabled
 firewalld.service                           enabled
```

我们还可以使用`systemctl`命令查看守护程序的状态、守护程序被执行的目录以及守护程序的**进程 ID**（**PID**）。我们将使用`status`选项：

```
[philip@localhost ~]$ systemctl status sshd.service
 ● sshd.service - OpenSSH server daemon
 Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
 Active: active (running) since Wed 2018-06-20 09:35:31 PDT; 1h 43min ago
 Docs: man:sshd(8)
 man:sshd_config(5)
 Main PID: 1072 (sshd)
 CGroup: /system.slice/sshd.service
 └─1072 /usr/sbin/sshd -D
 [philip@localhost ~]$ 
```

我们也可以使用`systemctl`命令停止、启动、重启、启用和禁用守护程序。假设我们想使用`systemctl`命令停止`ssd`服务。我们只需这样做：

```
[philip@localhost ~]$ systemctl stop sshd
```

现在，当我们在 CentOS 7 系统上按下*Enter*键时，我们将收到一个身份验证提示，因为我们正在尝试以标准用户身份停止`sshd`服务：

![](img/00043.jpeg)

`sshd`被认为是一个系统服务。此外，在`systemd`的上下文中，一个单元是一个服务，反之亦然。

现在我们将输入 root 密码：

![](img/00044.jpeg)

现在`sshd`服务已经停止：

```
[philip@localhost ~]$ systemctl stop sshd
 [philip@localhost ~]$
```

现在让我们重新检查`sshd`服务的状态，以确认它确实已经停止，使用`systemctl`命令：

```
[philip@localhost ~]$ systemctl status sshd.service
 ● sshd.service - OpenSSH server daemon
 Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
 Active: inactive (dead) since Wed 2018-06-20 11:20:16 PDT; 21min ago
 Docs: man:sshd(8)
 man:sshd_config(5)
 Main PID: 1072 (code=exited, status=0/SUCCESS)
 [philip@localhost ~]$ 
```

从前面的代码中，我们可以得出`sshd`服务已经停止。

# DMESG

现在，当系统启动时，屏幕上会快速显示与我们的系统各个方面相关的许多消息，从硬件到服务。在故障排除时，能够查看这些消息将非常有用。收集尽可能多的信息以帮助故障排除总是很有用。

我们还可以利用另一个强大的命令，即`dmesg`命令：

```
philip@ubuntu:~$ dmesg
 [ 0.000000] Initializing cgroup subsys cpuset
 [ 0.000000] Initializing cgroup subsys cpu
 [ 0.000000] Initializing cgroup subsys cpuacct
 [ 0.000000] Linux version 4.4.0-128-generic (buildd@lcy01-amd64-019) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.9) ) #154-Ubuntu SMP Fri May 25 14:15:18 UTC 2018 (Ubuntu 4.4.0-128.154-generic 4.4.131)
 [ 0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-4.4.0-128-generic root=UUID=adb5d090-3400-4411-aee2-dd871c39db38 ro find_preseed=/preseed.cfg auto noprompt priority=critical locale=en_US quiet
```

为了简洁起见，省略了一些输出：

```
[ 13.001702] audit: type=1400 audit(1529517046.911:8): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/bin/evince" pid=645 comm="apparmor_parser"
 [ 19.155619] e1000: ens33 NIC Link is Up 1000 Mbps Full Duplex, Flow Control: None
 [ 19.156584] IPv6: ADDRCONF(NETDEV_CHANGE): ens33: link becomes ready
 [ 105.095992] do_trap: 33 callbacks suppressed
 [ 105.095996] traps: pool[2056] trap int3 ip:7f778e83c9eb sp:7f776b1eb6f0 error:0
 philip@ubuntu:~$
```

从前面的输出中，我们可以看到各种信息，包括 CPU 检测、PCI 驱动程序和以太网等。

# GRUB 和 GRUB2

现在，我们将转变方向，讨论引导管理器的工作是呈现引导菜单，用户可以从中选择要加载或编辑的操作系统/ Linux 内核。首先，我们将重点放在 GRUB 上，然后转到 GRUB2。

# GRUB

GRUB 代表**Grand Unified Bootloader**。GRUB 主要用于引导 Linux 发行版。但是，GRUB 可以与其他引导加载程序一起使用。一个常见的用例是与 Microsoft 操作系统双引导，它通过将控制权移交给 Windows 引导加载程序来实现这一点。

GRUB 使用`/boot/grub/grub.conf`文件。有时您会看到`/boot/grub/menu.lst`，但是这个文件只是`/boot/grub/grub.conf`的符号链接。使用 CentOS 6.5 发行版，运行以下命令：

```
[root@localhost ~]# ls -l /boot/grub
 total 274
 -rw-r--r--. 1 root root 63 Jun 20 01:47    device.map
 -rw-r--r--. 1 root root 13380 Jun 20 01:47 e2fs_stage1_5
 -rw-r--r--. 1 root root 12620 Jun 20 01:47 fat_stage1_5
 -rw-r--r--. 1 root root 11748 Jun 20 01:47 ffs_stage1_5
 -rw-------. 1 root root 769 Jun 20 01:48   grub.conf
 -rw-r--r--. 1 root root 11756 Jun 20 01:47 iso9660_stage1_5
 -rw-r--r--. 1 root root 13268 Jun 20 01:47 jfs_stage1_5
 lrwxrwxrwx. 1 root root 11 Jun 20 01:47    menu.lst -> ./grub.conf
 -rw-r--r--. 1 root root 11956 Jun 20 01:47 minix_stage1_5
 -rw-r--r--. 1 root root 14412 Jun 20 01:47 reiserfs_stage1_5
 -rw-r--r--. 1 root root 1341 Nov 14 2010   splash.xpm.gz
 -rw-r--r--. 1 root root 512 Jun 20 01:47    stage1
 -rw-r--r--. 1 root root 126100 Jun 20 01:47 stage2
 -rw-r--r--. 1 root root 12024 Jun 20 01:47  ufs2_stage1_5
 -rw-r--r--. 1 root root 11364 Jun 20 01:47  vstafs_stage1_5
 -rw-r--r--. 1 root root 13964 Jun 20 01:47  xfs_stage1_5
 [root@localhost ~]#
```

从前面的输出中，我们可以看到`/boot/grub/grub.conf`，还有符号链接`/boot/grub/menu.lst`。

我们可以查看实际的`/boot/grub/grub.conf`文件：

```
[root@localhost ~]# cat /boot/grub/grub.conf
 # grub.conf generated by anaconda
 #
 # Note that you do not have to rerun grub after making changes to this file
 # NOTICE: You have a /boot partition. This means that
 # all kernel and initrd paths are relative to /boot/, eg.
 # root (hd0,0)
 # kernel /vmlinuz-version ro root=/dev/sda2
 # initrd /initrd-[generic-]version.img
 #boot=/dev/sda
 default=0
 timeout=5
 splashimage=(hd0,0)/grub/splash.xpm.gz
 hiddenmenu
 title CentOS (2.6.32-431.el6.x86_64)
 root (hd0,0)
 kernel /vmlinuz-2.6.32-431.el6.x86_64 ro root=UUID=05527d71-25b6-4931-a3bb-8fe505f3fa64 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
 initrd /initramfs-2.6.32-431.el6.x86_64.img
 [root@localhost ~]#
```

从前面的输出中，常见的选项将是以下内容。`default=0`表示它是菜单中要启动的第一个条目。`timeout=5`给出了菜单显示的秒数（在这种情况下为 5），在加载 Linux 内核或 Windows 引导加载程序从 GRUB 手中接管之前。`splashimage=(hd0,0)/grub/splash.xpm.gz`是引导菜单的背景图像。`root (hd0,0)`指的是第一个硬盘和第一个硬盘上的第一个分区。

# GRUB2

GRUB2 在菜单呈现方式上使用了更加程序化的方法。乍一看，GRUB2 可能看起来令人生畏，但请放心，它并不像看起来那么复杂。语法类似于编程语言，有很多*if...then*语句。以下是 CentOS 7 系统上`/boot/grub/grub.cfg`的样子：

```
[root@localhost ~]# cat /boot/grub2/grub.cfg
 #
 # DO NOT EDIT THIS FILE
 #
 # It is automatically generated by grub2-mkconfig using templates
 # from /etc/grub.d and settings from /etc/default/grub
 #
### BEGIN /etc/grub.d/00_header ###
 set pager=1
if [ -s $prefix/grubenv ]; then
 load_env
 fi
 if [ "${next_entry}" ] ; then
 set default="${next_entry}"
 set next_entry=
```

```
 save_env next_entry
 set boot_once=true
 else
 set default="${saved_entry}"
 fi
```

出于简洁起见，以下部分输出被省略。以下显示了`/boot/grub/grub.cfg`的最后部分：

```
### BEGIN /etc/grub.d/10_linux ###
 menuentry 'CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-693.el7.x86_64-advanced-16e2de7b-b679-4a12-888e-55081af4dad8' {
 load_video
 set gfxpayload=keep
 insmod gzio
 insmod part_msdos
 insmod xfs
 set root='hd0,msdos1'
 if [ x$feature_platform_search_hint = xy ]; then
 search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1' 40c7c63f-1c93-438a-971a-5331e265419b
 else
 search --no-floppy --fs-uuid --set=root 40c7c63f-1c93-438a-971a-5331e265419b
 fi
 linux16 /vmlinuz-3.10.0-693.el7.x86_64 root=UUID=16e2de7b-b679-4a12-888e-55081af4dad8 ro crashkernel=auto rhgb quiet LANG=en_US.UTF-8
 initrd16 /initramfs-3.10.0-693.el7.x86_64.img
 }
 ### END /etc/grub.d/10_linux ###
```

因此，要解释`/boot/grub/grub.cfg`文件，我们要查找以`menuentry`开头的行。这些行开始了操作系统的实际菜单条目，例如 Linux 发行版或 Windows 操作系统。

条目被包含在大括号{}中。

# 与 GRUB 一起工作

现在我们将与 GRUB 进行交互。我们将添加一个自定义引导条目。这将在重新启动时呈现。我们将使用`vi`命令，它将在可视编辑器中打开`/boot/grub/grub.conf`：

在使用 GRUB 之前，始终备份您的`/boot/grub/grub.conf`。

```
[root@localhost ~]# cat /boot/grub/grub.conf
 # grub.conf generated by anaconda
 #
 # Note that you do not have to rerun grub after making changes to this file
 # NOTICE: You have a /boot partition. This means that
 # all kernel and initrd paths are relative to /boot/, eg.
 # root (hd0,0)
 # kernel /vmlinuz-version ro root=/dev/sda2
 # initrd /initrd-[generic-]version.img
 #boot=/dev/sda
 default=0
 timeout=5
 splashimage=(hd0,0)/grub/splash.xpm.gz
 hiddenmenu
 title CentOS (2.6.32-431.el6.x86_64)
 root (hd0,0)
 kernel /vmlinuz-2.6.32-431.el6.x86_64 ro root=UUID=05527d71-25b6-4931-a3bb-8fe505f3fa64 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
 initrd /initramfs-2.6.32-431.el6.x86_64.img
 [root@localhost ~]# vi /boot/grub/grub.conf
```

现在我们在`vi`中。我们将按下键盘上的*I*进入插入模式，使用下箭头键向下滚动，直到达到最后一行，然后按*Enter*进入新行：

```
# grub.conf generated by anaconda
 #
 # Note that you do not have to rerun grub after making changes to this file
 # NOTICE: You have a /boot partition. This means that
 # all kernel and initrd paths are relative to /boot/, eg.
 # root (hd0,0)
 # kernel /vmlinuz-version ro root=/dev/sda2
 # initrd /initrd-[generic-]version.img
 #boot=/dev/sda
 default=0
 timeout=5
 splashimage=(hd0,0)/grub/splash.xpm.gz
 hiddenmenu
 title CentOS (2.6.32-431.el6.x86_64)
 root (hd0,0)
 kernel /vmlinuz-2.6.32-431.el6.x86_64 ro root=UUID=05527d71-25b6-4931-a3bb-8fe505f3fa64 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
 initrd /initramfs-2.6.32-431.el6.x86_64.img
~
 ~
 ~
 -- INSERT --
```

接下来，我们将使用以下关键字启动我们的条目：`title`、`root`、`kernel`和`initrd`。我们将插入我们自己的自定义值，如下所示：

```
# grub.conf generated by anaconda
 #
 # Note that you do not have to rerun grub after making changes to this file
 # NOTICE: You have a /boot partition. This means that
 # all kernel and initrd paths are relative to /boot/, eg.
 # root (hd0,0)
 # kernel /vmlinuz-version ro root=/dev/sda2
 # initrd /initrd-[generic-]version.img
 #boot=/dev/sda
 default=0
 timeout=5
 splashimage=(hd0,0)/grub/splash.xpm.gz
 hiddenmenu
 title CentOS (2.6.32-431.el6.x86_64)
 root (hd0,0)
 kernel /vmlinuz-2.6.32-431.el6.x86_64 ro root=UUID=05527d71-25b6-4931-a3bb-8fe505f3fa64 rd_NO_LUKS rd_NO_LVM LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashkernel=auto KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM rhgb quiet
 initrd /initramfs-2.6.32-431.el6.x86_64.img
 title CompTIA Linux+ (Our.Custom.Entry)
 root (hd0,0)
 kernel /vmlinuz-2.6.32-431.el6.x86 ro
 initrd /initramfs-2.6.32-431.el6.x86_64.img
 -- INSERT --
```

现在我们将保存并退出`vi`。我们使用`:wq`来保存我们的更改并退出`vi`：

```
title CompTIA Linux+ (Our.Custom.Entry)
 root (hd0,0)
 kernel /vmlinuz-2.6.32-431.el6.x86 ro
 initrd /initramfs-2.6.32-431.el6.x86_64.img
 :wq
```

根据前面的输出，这是我们自定义条目的分解：

+   `title`定义了我们的自定义引导条目。

+   `root (hd0,0)` 告诉它搜索第一个硬盘和第一个硬盘上的第一个分区。

+   `kernel /vmlinuz-2.6.32-431.el6.x86 ro` 告诉 GRUB 查找 Linux 内核的位置。在这种情况下，它是`vmlinuz-2.6.32-431.el6.x86 ro`（`ro`表示以只读方式加载内核）。

+   `inidrd /initramfs-2.6.32-431.el6.x86_64.img`指定要使用的初始 RAM 磁盘文件（这有助于系统启动）。

最后一步是重新启动我们的 CentOS 系统，并显示 GRUB 引导菜单：

![](img/00045.gif)

从前面的输出中，我们可以看到我们的新自定义引导条目显示在 GRUB 中，这很棒。我们可以实时交互，就在 GRUB 菜单上。假设我们想要在这些条目中添加或删除选项。我们只需按下*E*键，如下所示：

![](img/00046.gif)

现在我们可以再次按*E*键编辑该项。假设我们想指定根文件系统位于`/dev/`；我们可以按照以下截图所示进行操作：

![](img/00047.gif)

现在，我们可以按*Enter*键保存我们的更改，按*Esc*键返回到上一个屏幕；我们将看到新添加的选项：

![](img/00048.gif)

从前面的输出中，我们可以看到在 GRUB 引导菜单中实时工作以及如何在 GRUB 中添加自定义引导项是多么容易。

在 GRUB 中，第一个硬盘和第一个分区被标识为`(hd0, 0)`，而在 Linux shell 中，第一个硬盘和第一个分区被标识为`(sda1)`。

# 使用 GRUB2

我们以与 GRUB 略有不同的方式在 GRUB2 中添加自定义引导项。在 GRUB2 中，我们不是编辑实际的`/boot/grub/grub.cfg`，而是使用`/etc/default/grub`和`/etc/grub.d`。让我们列出`/etc/grub.d`以查看所有可用的文件：

```
philip@ubuntu:~$ ls -l /etc/grub.d/
 total 76
 -rwxr-xr-x 1 root root 9791 Apr 15 2016 00_header
 -rwxr-xr-x 1 root root 6258 Mar 15 2016 05_debian_theme
 -rwxr-xr-x 1 root root 12261 Apr 15 2016 10_linux
 -rwxr-xr-x 1 root root 11082 Apr 15 2016 20_linux_xen
 -rwxr-xr-x 1 root root 1992 Jan 28 2016 20_memtest86+
 -rwxr-xr-x 1 root root 11692 Apr 15 2016 30_os-prober
 -rwxr-xr-x 1 root root 1418 Apr 15 2016 30_uefi-firmware
 -rwxr-xr-x 1 root root 214 Apr 15 2016 40_custom
 -rwxr-xr-x 1 root root 216 Apr 15 2016 41_custom
 -rw-r--r-- 1 root root 483 Apr 15 2016 README
 philip@ubuntu:~$
```

在使用 GRUB2 之前，始终备份您的`/boot/grub/grub.cfg`。

从前面的输出中，我们可以看到许多文件。它们的名称以数字开头，并且数字是按顺序读取的。假设我们想在 GRUB2 中添加一个自定义的引导项。我们将创建一个自定义项并命名为`/etc/grub/40_custom`。我们将在`vi`中看到以下代码：

```
#!/bin/sh
 exec tail -n +3 $0
 # This file provides an easy way to add custom menu entries. Simply type the
 # menu entries you want to add after this comment. Be careful not to change
 # the 'exec tail' line above.
 echo "Test Entry"
 cat << EOF
 menuentry "CompTIA_LINUX+" {
 set root ='hd0,0'
}
 EOF
```

从前面的输出中，我们可以看到语法与编程有些相似。在 GRUB2 中，它是一种完整的编程语言。下一步是保存我们的更改，然后运行`grub-mkconfig`（名称暗示我们在谈论旧版 GRUB，但实际上是指 GRUB2）。这取决于 Linux 发行版。在 CentOS 7 中，您将看到以`grub2`开头的命令：

```
root@ubuntu:/home/philip# grub-mkconfig
 Generating grub configuration file ...
 #
 # DO NOT EDIT THIS FILE
 #
 # It is automatically generated by grub-mkconfig using templates
 # from /etc/grub.d and settings from /etc/default/grub
 #
### BEGIN /etc/grub.d/00_header ###
 if [ -s $prefix/grubenv ]; then
 set have_grubenv=true
 load_env
 fi
```

出于简洁起见，以下部分输出被省略：

```
### BEGIN /etc/grub.d/40_custom ###
 # This file provides an easy way to add custom menu entries. Simply type the
 # menu entries you want to add after this comment. Be careful not to change
 # the 'exec tail' line above.
 echo "Test Entry"
 cat << EOF
 menuentry "CompTIA_LINUX+" {
 set root ='hd0,0'
}
 EOF
```

当我们运行此命令时，`grub-mkconfig`命令会找到自定义项。然后生成一个新的引导菜单。在系统下一次重启时，我们将看到新的引导菜单。我们还可以更改`/etc/default/grub`中的选项，包括默认操作系统、计时器等。以下是`/etc/default/grub`的内容：

```
root@ubuntu:/home/philip# cat /etc/default/grub
 # If you change this file, run 'update-grub' afterwards to update
 # /boot/grub/grub.cfg.
 # For full documentation of the options in this file, see:
 # info -f grub -n 'Simple configuration'
GRUB_DEFAULT=0
 GRUB_HIDDEN_TIMEOUT=0
 GRUB_HIDDEN_TIMEOUT_QUIET=true
 GRUB_TIMEOUT=10
 GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
 GRUB_CMDLINE_LINUX_DEFAULT="quiet"
 GRUB_CMDLINE_LINUX="find_preseed=/preseed.cfg auto noprompt priority=critical locale=en_US"
```

根据前面的输出，计时器值设置为`10`。还要注意默认值为`0`。在配置文件中继续向下，我们看到以下代码：

```
# Uncomment to enable BadRAM filtering, modify to suit your needs
# This works with Linux (no patch required) and with any kernel that obtains
# the memory map information from GRUB (GNU Mach, kernel of FreeBSD ...)
#GRUB_BADRAM="0x01234567,0xfefefefe,0x89abcdef,0xefefefef"
# Uncomment to disable graphical terminal (grub-pc only)
#GRUB_TERMINAL=console
# The resolution used on graphical terminal
# note that you can use only modes which your graphic card supports via VBE
# you can see them in real GRUB with the command `vbeinfo'
#GRUB_GFXMODE=640x480
# Uncomment if you don't want GRUB to pass "root=UUID=xxx" parameter to Linux
#GRUB_DISABLE_LINUX_UUID=true
# Uncomment to disable generation of recovery mode menu entries
#GRUB_DISABLE_RECOVERY="true"
# Uncomment to get a beep at grub start
#GRUB_INIT_TUNE="480 440 1"
```

现在，让我们重新启动 Ubuntu 系统并查看 GRUB2 引导菜单：

![](img/00049.jpeg)

从前面的截图中，我们现在可以在 GRUB2 中看到我们的自定义菜单选项。我们甚至可以通过按 E 键滚动浏览条目并编辑它们。

在 GRUB2 中，第一个硬盘从`0`开始，第一个分区从`1`开始，与旧版 GRUB 不同。

# 总结

在本章中，我们看了一下引导过程。然后讨论了`init`和`systemd`。我们使用了`pstree`命令并看到了加载的第一个进程。此外，我们使用了`ps`命令来识别进程号。然后，我们查看了通常会在屏幕上滚动的引导消息，使用`dmesg`命令。显示的消息为我们提供了关于引导时加载的内容的提示。此外，我们可以使用显示的消息来帮助我们进行故障排除。接下来，我们讨论了 GRUB 和 GRUB2，查看了 GRUB 的结构，特别是`/boot/grub/grub/conf`。我们看了如何在 GRUB 中添加自定义菜单项。我们看了如何在引导菜单中实时与 GRUB 交互。之后，我们看了 GRUB2，重点是`/boot/grub/grub.cfg`的结构。此外，我们还看了在 GRUB2 配置中起作用的其他位置：`/etc/default/grub/`和`/etc/grub.d/`目录。然后，我们使用`/etc/grub.d/40_custom`文件在`/etc/grub.d/`中添加了自定义菜单项。之后，我们使用`grub-mkconfig`（Ubuntu 发行版）更新了 GRUB2。最后，我们实时与 GRUB2 引导菜单进行了交互。

在下一章中，我们将专注于运行级别和引导目标。这些是我们作为 Linux 工程师需要充分理解的关键主题。我们将使用各种方法在命令行上管理系统。我们将涵盖`runlevel`、`init`和`systemctl`等命令。在下一章中，有很多有用的信息可以获得。了解运行级别的工作原理至关重要。此外，还有引导目标的概念。在大多数较新的发行版中，您将接触到引导目标。这将帮助您在命令行环境中管理 Linux 系统。在下一章中，您的技能将继续增长。这将进一步使您更接近成功，获得认证。

# 问题

1.  引导加载程序位于硬盘的哪个位置？

A. 引导扇区

B. 次要分区

C. 逻辑分区

D. 以上都不是

1.  哪个是第一个商业 Unix 操作系统？

A. systemd

B. upstart

C. System X

D. System V

1.  哪个命令显示从父进程开始的进程，然后是子进程？

A. `dnf`

B. `systemctl`

C. ` pstree`

D. `ps`

1.  在 CentOS 5 系统上启动的第一个进程是什么？

A. `systemd`

B. `init`

C. `kickstart`

D. `upstart`

1.  在 Linux 内核的较新版本中，`init`被什么取代了？

A. `telinit`

B. `systemctl`

C. `systemb`

D. `systemd`

1.  哪个命令列出了在 CentOS 7 发行版上运行的进程？

A. `systemd list-unit-files`

B. `systemX list-unit-files`

C. `systemctl list-unit-files`

D. `service status unit-files`

1.  哪个命令列出了系统引导时加载的硬件驱动程序？

A. `cat /var/log/messages`

B. `tail –f /var/log/startup`

C. `head /var/messages`

D. `dmesg`

1.  在 CentOS 5 发行版中，GRUB 配置文件位于哪个目录？

A. `/boot/`

B. `/grub/boot/`

C. `/boot/grub/`

D. `/grub/grub-config/`

1.  在 GRUB 中添加自定义菜单项时，是什么启动了自定义菜单项？

A. `title`

B. `menu entry`

C. `操作系统`

D. `default =0`

1.  在 GRUB2 中添加自定义菜单项时，是什么启动了自定义菜单项？

A. `title`

B. `root = /vmlinuz/`

C. `menuentry`

D. `menu entry`

1.  哪个字母键用于在 GRUB 引导菜单中实时编辑条目？

A. *C*

B. *E*

C. *B*

D. *A*

# 进一步阅读

+   您可以在[`www.centos.org.`](https://www.centos.org)获取有关 CentOS 发行版的更多信息，如安装、配置最佳实践等。

+   以下网站为您提供了许多有用的提示和 Linux 社区用户的最佳实践，特别是适用于 Debian 发行版，如 Ubuntu：[`askubuntu.com.`](https://askubuntu.com)

+   以下链接为您提供了一般信息，涉及适用于 CentOS 和 Ubuntu 的各种命令。您可以在那里发布您的问题，其他社区成员将会回答：[`www.linuxquestions.org.`](https://www.linuxquestions.org)
