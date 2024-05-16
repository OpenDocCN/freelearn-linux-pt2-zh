# 第二十一章：评估

# 第一章：配置硬件设置

1.  答案是 A - `/dev`

1.  答案是 D - cat /proc/cpuinfo

1.  答案是 C - cat /proc/meminfo

1.  答案是 C - `free -h`

1.  答案是 C - mkswap

1.  答案是 D - `swapon`

1.  答案是 B - swapon

1.  答案是 B - `/dev/null`

1.  答案是 D - lsmod

1.  答案是 D - modprobe

# 第二章：启动系统

1.  答案是 A - 引导扇区

1.  答案是 D - System V

1.  答案是 C - pstree

1.  答案是 B - init

1.  答案是 D - `systemd`

1.  答案是 C - `systemctl list-unit-files`

1.  答案是 D - dmesg

1.  答案是 C - /boot/grub/

1.  答案是 A - 标题

1.  答案是 C - `menuentry`

1.  答案是 B - E

# 第三章：更改运行级别和引导目标

1.  答案是 B - 5

1.  答案是 C - runlevel

1.  答案是 D - `who -r`

1.  答案是 B - 在更改为当前运行级别之前的先前运行级别

1.  答案是 D - 单用户

1.  答案是 B - init

1.  答案是 C - `telinit`

1.  答案是 A - `systemctl get-default`

1.  答案是 C - systemctl list-dependencies -type target

1.  答案是 B - `systemctl isolate multi-user.target`

1.  答案是 A - `systemctl status multi-user.target`

# 第四章：设计硬盘布局

1.  答案是 C - fdisk - l /dev/sda

1.  答案是 D - *n*

1.  答案是 B - a

1.  答案是 A - l

1.  答案是 B - n

1.  答案是 D - *w*

1.  答案是 B - parted

1.  答案是 C - print

1.  答案是 A - `mount /dev/sdb1`

1.  答案是 A - blkid

# 第五章：安装 Linux 发行版

1.  答案是 C - RAM

1.  答案是 D - 尝试 Ubuntu 而不安装

1.  答案是 A - 安装 Ubuntu…

1.  答案是 B - 一个活动的互联网连接

1.  答案是 A - 其他

1.  答案是 C - 主要

1.  答案是 B - 为了防止系统因意外删除/boot 中的文件而无法启动

1.  答案是 B - 我将配置分区

1.  答案是 A - `grub-install`

1.  答案是 D - Minimal Install

# 第六章：使用 Debian 软件包管理

1.  答案是 B - dpkg is -l

1.  答案是 C - `dpkg-query -s`

1.  答案是 A - `cat /var/log/dpkg.log`

1.  答案是 A - `dpkg --get-selections`

1.  答案是 D - dpkg is -i

1.  答案是 C - dpkg is -P

1.  答案是 B - `apt-get update`

1.  答案是 B - `apt-cache search`

1.  答案是 B - apt-get purge

1.  答案是 C - aptitude update

# 第七章：使用 YUM 软件包管理

1.  答案是 B - `yum list`

1.  答案是 A - yum makecache fast

1.  答案是 D - `yum provides`

1.  答案是 A - `dpkg --get-selections`

1.  答案是 B - yum clean all

1.  答案是 A - yum update

1.  答案是 B - dnf repolist all

1.  答案是 A - dnf check-update

1.  答案是 C - `rpm -qip`

1.  答案是 B - `rpm --erase`

# 第八章：执行文件管理

1.  答案是 D - /

1.  答案是 C - cd

1.  答案是 B - pwd

1.  答案是 A - ls

1.  答案是 D - -l

1.  答案是 C - -a

1.  答案是 B - rm

1.  答案是 C - -empty -delete

1.  答案是 D - updatedb

1.  答案是 D - tee

# 第九章：创建、监视、终止和重新启动进程

1.  答案是 C - ps

1.  答案是 C - -e

1.  答案是 B - --forest

1.  答案是 C - -u

1.  答案是 B - -l

1.  答案是 D - 9

1.  答案是 A - -u

1.  答案是 C - d

1.  答案是 D - reload

1.  答案是 D - `/usr/lib/systemd/system`

# 第十章：修改进程执行

1.  答案是 B - l

1.  答案是 A - NI

1.  答案是 D - `NI`

1.  答案是 C - 20

1.  答案是 D - -20

1.  答案是 B - /lib/systemd/system

1.  答案是 A - `systemctl daemon-reload`

1.  答案是 B - `PID`

1.  答案是 A - `fg`

1.  答案是 C - bg

# 第十一章：显示管理器

1.  答案是 A - X 显示管理器

1.  答案是 B - `/etc/X11/xdm`

1.  答案是 A - `Xaccess`

1.  答案是 C - /etc/sysconfig/desktop

1.  答案是 B - groupinstall

1.  答案是 C - system-switch-displaymanager

1.  答案是 A - 会话类型

1.  答案是 D - dpkg-reconfigure

1.  答案是 C - /etc/X11/default-display-manager

1.  答案是 B - `ls -l /etc/systemd/system/display-manager.service`

# 第十二章：管理用户和组帐户

1.  答案是 D - /etc/skel/.bashrc

1.  答案是 A - `/etc.skel/.bash_logout`

1.  答案是 B - `~/.bash_history`

1.  答案是 C - -D

1.  答案是 D - -s

1.  答案是 D - user-add

1.  答案是 B - -l

1.  答案是 C - L

1.  答案是 A - -g

1.  答案是 C - groupmod

# 第十三章：自动化任务

1.  答案是 B - 乱码时间

1.  答案是 C - 在下周的上午 9:00

1.  答案是 B - 按下了*CTRL+ D*

1.  答案是 D - `-l`

1.  答案是 D - `-r`

1.  答案是 C - `atq`

1.  答案是 C - `*****`

1.  答案是 B - `-e`

1.  答案是 C - `@每周`

1.  答案是 A - `-f`

# 第十四章：维护系统时间和日志记录

1.  答案是 A - `-s`

1.  答案是 C - `set-ntp`

1.  答案是 D - `+%T`

1.  答案是 A - `设置时间`

1.  答案是 D - `/etc/localtime`

1.  答案是 B - `tzdata`

1.  答案是 B - `tzselect`

1.  答案是 D - `-u`

1.  答案是 D - `TCP`

1.  答案是 C - `记录器`

# 第十五章：互联网协议基础

1.  答案是 C - `10.0.0.1`

1.  答案是 C - `192.168.0.1`

1.  答案是 A - `127.0.0.1`

1.  答案是 A - `169.0.0.1`

1.  答案是 A - `128.0.0.1`

1.  答案是 C - `ff00::/8`

1.  答案是 B - `::0/0`

1.  答案是 C - `::1/128`

1.  答案是 D - `fe80::/10`

1.  答案是 C - `TCP 80`

# 第十六章：网络配置和故障排除

1.  答案是 D - `-a`

1.  答案是 A - `默认`

1.  答案是 C - ICMP

1.  答案是 B - `/etc/hostname`

1.  答案是 C - `tracepath`

1.  答案是 D - `dig`

1.  答案是 A - `ip -6 route add default via 2001:db8:0:f101::2`

1.  答案是 D - `-ulp`

1.  答案是 C - `nmap`

1.  答案是 B - `whois`

# 第十七章：执行管理安全任务

1.  答案是 C - `生成`

1.  答案是 B - `替代用户`

1.  答案是 A - `根用户`

1.  答案是 B - `-c`

1.  答案是 D - `%`

1.  答案是 A - `ssh-keygen`

1.  答案是 A - `ssh-add`

1.  答案是 B - `ssh-copy-id`

1.  答案是 B - `-e`

1.  答案是 C - `-r`

# 第十八章：Shell 脚本和 SQL 数据管理

1.  答案是 C - `#!`

1.  答案是 A - `SHELL`

1.  答案是 C - `完成`

1.  答案是 A - `.`

1.  答案是 D - `读取`

1.  答案是 B - `||`

1.  答案是 C - `*`

1.  答案是 C - `在哪里`

1.  答案是 B - `创建表`

1.  答案是 C - `更新`
