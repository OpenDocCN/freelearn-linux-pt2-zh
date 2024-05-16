您得到了一个软件包

在本章中，您将学习如何在 Linux 系统上管理软件应用程序。您将学习如何使用 Debian 软件包管理器来下载、安装、删除、搜索和更新软件包。

# 第十三章：什么是软件包？

在 Linux 中，软件包是一个压缩的存档文件，其中包含特定软件应用程序运行所需的所有必要文件。例如，像 Firefox 这样的网络浏览器以一个包的形式提供，其中包含了 Firefox 运行所需的所有文件。

# 软件包管理器的作用

软件包管理器是我们在 Linux 中用来管理软件包的程序；也就是说，下载、安装、删除、搜索和更新软件包。请记住，不同的 Linux 发行版有不同的软件包管理器。例如，`dpkg`代表 Debian 软件包管理器，是 Ubuntu 和其他基于 Debian 的 Linux 发行版的软件包管理器。另一方面，基于 RedHat 的 Linux 发行版如 Fedora 和 CentOS 使用`rpm`，代表 RedHat 软件包管理器。其他 Linux 发行版如 SUSE 使用`zypper`作为软件包管理器等等。

# 软件包从哪里来？

很少有经验丰富的 Linux 用户会像 Windows 或 macOS 用户那样去网站下载软件包。相反，每个 Linux 发行版都有其软件包来源列表，大部分软件包都来自这些来源。这些来源也被称为**存储库**。以下图示了在您的 Linux 系统上下载软件包的过程：

![](img/db4d532e-24a0-4944-ab21-aa59b9014390.png)

图 1：软件包存储在存储库中。请注意，软件包存储在多个存储库中

# 如何下载软件包

在 Ubuntu 和其他 Debian Linux 发行版上，您可以使用命令行实用程序`apt-get`来管理软件包。在幕后，`apt-get`利用软件包管理器`dpkg`。要下载软件包，您可以运行命令`apt-get download`后跟软件包名称：

```
apt-get download package_name
```

作为`root`用户，切换到`/tmp`目录：

```
root@ubuntu-linux:~# cd /tmp
```

要下载`cmatrix`软件包，您可以运行以下命令：

```
root@ubuntu-linux:/tmp# apt-get download cmatrix
Get:1 http://ca.archive.ubuntu.com/ubuntu bionic/universe amd64 cmatrix amd64
1.2a-5build3 [16.1 kB]
Fetched 16.1 kB in 1s (32.1 kB/s) 
```

`cmatrix`软件包将被下载到`/tmp`目录中：

```
root@ubuntu-linux:/tmp# ls 
cmatrix_1.2a-5build3_amd64.deb
```

请注意软件包名称中的`.deb`扩展名，这表示它是一个 Debian 软件包。在 RedHat 发行版上，软件包名称以`.rpm`扩展名结尾。您可以通过运行以下命令`dpkg -c`来列出`cmatrix`软件包中的文件：

```
root@ubuntu-linux:/tmp# dpkg -c cmatrix_1.2a-5build3_amd64.deb
drwxr-xr-x root/root     0 2018-04-03 06:17 ./
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/bin/
-rwxr-xr-x root/root 18424 2018-04-03 06:17 ./usr/bin/cmatrix
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/share/
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/share/consolefonts/
-rw-r--r-- root/root  4096 1999-05-13 08:55 ./usr/share/consolefonts/matrix.fnt
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/share/doc/
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/share/doc/cmatrix/
-rw-r--r-- root/root  2066 2000-04-03 19:29 ./usr/share/doc/cmatrix/README
-rw-r--r-- root/root   258 1999-05-13 09:12 ./usr/share/doc/cmatrix/TODO
-rw-r--r-- root/root  1128 2018-04-03 06:17 ./usr/share/doc/cmatrix/copyright
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/share/man/
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/share/man/man1/
-rw-r--r-- root/root   932 2018-04-03 06:17 ./usr/share/man/man1/cmatrix.1.gz
drwxr-xr-x root/root     0 2018-04-03 06:17 ./usr/share/menu/
-rw-r--r-- root/root   392 2018-04-03 06:17 ./usr/share/menu/cmatrix
```

请注意，我们只下载了软件包，但尚未安装。如果您运行`cmatrix`命令，将不会发生任何事情：

```
root@ubuntu-linux:/tmp# cmatrix
bash: /usr/bin/cmatrix: No such file or directory
```

# 如何安装软件包

您可以使用`dpkg`命令的`-i`选项来安装已下载的软件包：

```
root@ubuntu-linux:/tmp# dpkg -i cmatrix_1.2a-5build3_amd64.deb 
Selecting previously unselected package cmatrix.
(Reading database ... 178209 files and directories currently installed.) Preparing to unpack cmatrix_1.2a-5build3_amd64.deb ...
Unpacking cmatrix (1.2a-5build3) ... 
Setting up cmatrix (1.2a-5build3) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ... 
root@ubuntu-linux:/tmp#
```

就是这样！现在运行`cmatrix`命令：

```
root@ubuntu-linux:/tmp# cmatrix
```

您将在终端上看到矩阵运行，就像下图中一样：

![](img/18aeaee2-1d00-49f9-817d-e4a6f47959d6.png)

图 2：cmatrix

我们已经采取了安装`cmatrix`软件包的长途旅程。我们首先下载了软件包，然后安装了它。您可以通过运行命令`apt-get install`后跟软件包名称来立即安装软件包（无需下载）：

```
apt-get install package_name
```

例如，您可以通过运行以下命令安装**GNOME Chess**游戏：

```
root@ubuntu-linux:/tmp# apt-get install gnome-chess 
Reading package lists... Done
Building dependency tree
Reading state information... Done 
Suggested packages:
 bbchess crafty fairymax fruit glaurung gnuchess phalanx sjeng stockfish toga2 
The following NEW packages will be installed:
 gnome-chess
0 upgraded, 1 newly installed, 0 to remove and 357 not upgraded. 
Need to get 0 B/1,514 kB of archives.
After this operation, 4,407 kB of additional disk space will be used. 
Selecting previously unselected package gnome-chess.
(Reading database ... 178235 files and directories currently installed.) Preparing to unpack .../gnome-chess_1%3a3.28.1-1_amd64.deb ...
Unpacking gnome-chess (1:3.28.1-1) ...
Processing triggers for mime-support (3.60ubuntu1) ...
Processing triggers for desktop-file-utils (0.23-1ubuntu3.18.04.2) ... 
Processing triggers for libglib2.0-0:amd64 (2.56.3-0ubuntu0.18.04.1) ... 
Setting up gnome-chess (1:3.28.1-1) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ... 
Processing triggers for gnome-menus (3.13.3-11ubuntu1.1) ... 
Processing triggers for hicolor-icon-theme (0.17-2) ...
```

现在您可以通过运行`gnome-chess`命令来启动游戏：

```
root@ubuntu-linux:/tmp# gnome-chess
```

![](img/8f33af2e-d60f-4ac0-a46d-e659edb64dcc.png)

图 3：GNOME Chess

# 如何删除软件包

您可以通过运行命令`apt-get remove`后跟软件包名称来轻松删除一个软件包：

```
apt-get remove package_name
```

例如，如果你厌倦了矩阵生活方式，并决定删除`cmatrix`软件包，你可以运行：

```
root@ubuntu-linux:/tmp# apt-get remove cmatrix 
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be REMOVED:
 cmatrix
0 upgraded, 0 newly installed, 1 to remove and 357 not upgraded. 
After this operation, 49.2 kB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 178525 files and directories currently installed.) 
Removing cmatrix (1.2a-5build3) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

现在，如果您运行`cmatrix`命令，您将收到一个错误：

```
root@ubuntu-linux:/tmp# cmatrix
Command 'cmatrix' not found, but can be installed with: 
apt install cmatrix
```

`apt-get remove`命令会删除（卸载）一个软件包，但不会删除软件包的配置文件。您可以使用`apt-get purge`命令来删除软件包以及其配置文件。

例如，如果您想删除`gnome-chess`软件包以及其配置文件，您可以运行：

```
root@ubuntu-linux:/tmp# apt-get purge gnome-chess 
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:    
  hoichess
Use 'apt autoremove' to remove it.
The following packages will be REMOVED:
 gnome-chess*
0 upgraded, 0 newly installed, 1 to remove and 357 not upgraded. 
After this operation, 4,407 kB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 178515 files and directories currently installed.) 
Removing gnome-chess (1:3.28.1-1) ...
Processing triggers for mime-support (3.60ubuntu1) ...
Processing triggers for desktop-file-utils (0.23-1ubuntu3.18.04.2) ... 
Processing triggers for libglib2.0-0:amd64 (2.56.3-0ubuntu0.18.04.1) ... Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for gnome-menus (3.13.3-11ubuntu1.1) ... 
Processing triggers for hicolor-icon-theme (0.17-2) ...
(Reading database ... 178225 files and directories currently installed.) 
Purging configuration files for gnome-chess (1:3.28.1-1) ...
```

您甚至可以在输出的最后一行中看到`Purging configuration files for gnome-chess (1:3.28.1-1) ...`，这意味着`gnome-chess`的配置文件也正在被删除。

# 如何搜索软件包

有时您不确定软件包名称。在这种情况下，您无法安装它，直到查找它。您可以使用`apt-cache search`命令，后跟搜索词或关键词来搜索软件包：

```
apt-cache search keyword
```

例如，假设您想安装`wireshark`软件包，但您只记得软件包名称中有`shark`这个词。在这种情况下，您可以运行以下命令：

```
root@ubuntu-linux:/tmp# apt-cache search shark
dopewars - drug-dealing game set in streets of New York City
dopewars-data - drug-dealing game set in streets of New York City - data files forensics-extra - Forensics Environment - extra console components (metapackage) kernelshark - Utilities for graphically analyzing function tracing in the kernel libcrypto++-dev - General purpose cryptographic library - C++ development libshark-dev - development files for Shark
libshark0 - Shark machine learning library
libwireshark-data - network packet dissection library -- data files 
libwireshark-dev - network packet dissection library -- development files libwireshark10 - network packet dissection library -- shared library 
libwiretap-dev - network packet capture library -- development files
libwsutil-dev - network packet dissection utilities library -- development files libwsutil8 - network packet dissection utilities library -- shared library netmate - netdude clone that shows pcap dump lines in network header style plowshare-modules - plowshare drivers for various file sharing websites
shark-doc - documentation for Shark
tcpxtract - extract files from network traffic based on file signatures 
tshark - network traffic analyzer - console version
wifite - Python script to automate wireless auditing using aircrack-ng tools wireshark - network traffic analyzer - meta-package
wireshark-common - network traffic analyzer - common files 
wireshark-dev - network traffic analyzer - development tools 
wireshark-doc - network traffic analyzer - documentation 
wireshark-gtk - network traffic analyzer - GTK+ version 
wireshark-qt - network traffic analyzer - Qt version
zeitgeist-explorer - GUI application for monitoring and debugging zeitgeist forensics-extra-gui - Forensics Environment - extra GUI components (metapackage) horst - Highly Optimized Radio Scanning Tool
libvirt-wireshark - Wireshark dissector for the libvirt protocol 
libwiretap7 - network packet capture library -- shared library 
libwscodecs1 - network packet dissection codecs library -- shared library minetest-mod-animals - Minetest mod providing animals
nsntrace - perform network trace of a single process by using network namespaces libwireshark11 - network packet dissection library -- shared library 
libwiretap8 - network packet capture library -- shared library
libwscodecs2 - network packet dissection codecs library -- shared library libwsutil9 - network packet dissection utilities library -- shared library
```

然后您会被大量输出淹没，列出所有软件包名称中包含`shark`这个词的软件包。我敢打赌您可以在输出的中间找到`wireshark`软件包。我们可以通过使用`-n`选项获得一个更短和精炼的输出：

```
root@ubuntu-linux:/tmp# apt-cache -n search shark
kernelshark - Utilities for graphically analyzing function tracing in the kernel libshark-dev - development files for Shark
libshark0 - Shark machine learning library
libwireshark-data - network packet dissection library -- data files 
libwireshark-dev - network packet dissection library -- development files
libwireshark10 - network packet dissection library -- shared library 
shark-doc - documentation for Shark
tshark - network traffic analyzer - console version 
wireshark - network traffic analyzer - meta-package 
wireshark-common - network traffic analyzer - common files 
wireshark-dev - network traffic analyzer - development tools 
wireshark-doc - network traffic analyzer - documentation 
wireshark-gtk - network traffic analyzer - GTK+ version 
wireshark-qt - network traffic analyzer - Qt version
libndpi-wireshark - extensible deep packet inspection library - wireshark dissector 
libvirt-wireshark - Wireshark dissector for the libvirt protocol
libwireshark11 - network packet dissection library -- shared library
```

这将只列出软件包名称中包含`shark`这个词的软件包。现在，您可以通过运行以下命令来安装`wireshark`：

```
root@ubuntu-linux:/tmp# apt-get install wireshark
```

# 如何显示软件包信息

要查看软件包信息，您可以使用`apt-cache show`命令，后跟软件包名称：

```
apt-cache show package_name
```

例如，要显示`cmatrix`软件包信息，您可以运行：

```
root@ubuntu-linux:~# apt-cache show cmatrix 
Package: cmatrix
Architecture: amd64 
Version: 1.2a-5build3 
Priority: optional 
Section: universe/misc 
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com> 
Original-Maintainer: Diego Fernández Durán <diego@goedi.net>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug 
Installed-Size: 48
Depends: libc6 (>= 2.4), libncurses5 (>= 6), libtinfo5 (>= 6) 
Recommends: kbd
Suggests: cmatrix-xfont
Filename: pool/universe/c/cmatrix/cmatrix_1.2a-5build3_amd64.deb 
Size: 16084
MD5sum: 8dad2a99d74b63cce6eeff0046f0ac91 
SHA1: 3da3a0ec97807e6f53de7653e4e9f47fd96521c2
SHA256: cd50212101bfd71479af41e7afc47ea822c075ddb1ceed83895f8eaa1b79ce5d Homepage: http://www.asty.org/cmatrix/
Description-en_CA: simulates the display from "The Matrix" 
Screen saver for the terminal based in the movie "The Matrix".
 * Support terminal resize.
 * Screen saver mode: any key closes it.
 * Selectable color.
 * Change text scroll rate.
Description-md5: 9af1f58e4b6301a6583f036c780c6ae6
```

您可以在输出中看到许多有用的信息，包括软件包描述和软件包维护者的联系信息，如果您发现错误并希望报告它，则这些信息很有用。您还将了解软件包是否依赖于其他软件包。

**软件包依赖**可能会变成一场噩梦，因此我强烈建议您尽可能使用`apt-get install`命令来安装软件包，因为它在安装软件包时会检查和解决软件包依赖关系。另一方面，`dpkg -i`命令不会检查软件包依赖关系。记住这一点！

您可以使用`apt-cache depends`命令来列出软件包依赖关系：

```
apt-cache depends package_name
```

例如，要查看为使`cmatrix`正常工作而需要安装的软件包列表，您可以运行以下命令：

```
root@ubuntu-linux:~# apt-cache depends cmatrix 
cmatrix
 Depends: libc6 
 Depends: libncurses5 
 Depends: libtinfo5 
 Recommends: kbd 
 Suggests: cmatrix-xfont
```

正如您所看到的，`cmatrix`软件包依赖于三个软件包：

+   `libc6`

+   `libncurses5`

+   `libtinfo5`

这三个软件包必须安装在系统上，以便`cmatrix`正常运行。

# 列出所有软件包

您可以使用`dpkg -l`命令来列出系统上安装的所有软件包：

```
root@ubuntu-linux:~# dpkg -l
```

您还可以使用`apt-cache pkgnames`命令来列出所有可供您安装的软件包：

```
root@ubuntu-linux:~# apt-cache pkgnames 
libdatrie-doc
libfstrcmp0-dbg 
libghc-monadplus-doc 
librime-data-sampheng 
python-pyao-dbg 
fonts-georgewilliams
python3-aptdaemon.test 
libcollada2gltfconvert-dev 
python3-doc8
r-bioc-hypergraph
.
.
.
.
.
```

您可以将输出导入到`wc -l`命令中，以获得可用软件包的总数：

```
root@ubuntu-linux:~# apt-cache pkgnames | wc -l 64142
```

哇！这是一个庞大的数字；我的系统上有超过 64,000 个可用软件包。

您可能还想知道您的系统用于获取所有这些软件包的存储库（源）。这些存储库包含在文件`/etc/ap- t/sources.list`中，以及在目录`/etc/apt/- sources.list.d/`下具有后缀`.list`的任何文件中。您可以查看`man`页面：

```
root@ubuntu-linux:~# man sources.list
```

了解如何向系统添加存储库。

您还可以使用`apt-cache policy`命令来列出系统上启用的所有存储库：

```
root@ubuntu-linux:~# apt-cache policy 
Package files:
100 /var/lib/dpkg/status 
    release a=now
500 http://dl.google.com/linux/chrome/deb stable/main amd64 
    Packages release v=1.0,o=Google LLC,a=stable,n=stable,l=Google,c=main,
    b=amd64 origin dl.google.com
100 http://ca.archive.ubuntu.com/ubuntu bionic-backports/main i386 
    Packages release v=18.04,o=Ubuntu,a=bionic-backports,n=bionic,l=Ubuntu,
    c=main,b=i386 origin ca.archive.ubuntu.com
100 http://ca.archive.ubuntu.com/ubuntu bionic-backports/main amd64 
    Packages release v=18.04,o=Ubuntu,a=bionic-backports,n=bionic,l=Ubuntu,
    c=main,b=amd64 origin ca.archive.ubuntu.com
500 http://ca.archive.ubuntu.com/ubuntu bionic/multiverse i386 
    Packages release v=18.04,o=Ubuntu,a=bionic,n=bionic,
    l=Ubuntu,c=multiverse,b=i386 origin ca.archive.ubuntu.com
500 http://ca.archive.ubuntu.com/ubuntu bionic/multiverse amd64 
    Packages release v=18.04,o=Ubuntu,a=bionic,n=bionic,l=Ubuntu,
    c=multiverse,b=amd64 origin ca.archive.ubuntu.com
500 http://ca.archive.ubuntu.com/ubuntu bionic/universe i386 
    Packages release v=18.04,o=Ubuntu,a=bionic,n=bionic,l=Ubuntu,
    c=universe,b=i386 origin ca.archive.ubuntu.com
500 http://ca.archive.ubuntu.com/ubuntu bionic/universe amd64 
    Packages release v=18.04,o=Ubuntu,a=bionic,n=bionic,l=Ubuntu,
    c=universe,b=amd64 origin ca.archive.ubuntu.com
500 http://ca.archive.ubuntu.com/ubuntu bionic/restricted i386 
    Packages release v=18.04,o=Ubuntu,a=bionic,n=bionic,l=Ubuntu,
    c=restricted,b=i386 origin ca.archive.ubuntu.com
500 http://ca.archive.ubuntu.com/ubuntu bionic/restricted amd64 
    Packages release v=18.04,o=Ubuntu,a=bionic,n=bionic,l=Ubuntu,
    c=restricted,b=amd64 origin ca.archive.ubuntu.com
500 http://ca.archive.ubuntu.com/ubuntu bionic/main i386 
    Packages release v=18.04,o=Ubuntu,a=bionic,
    n=bionic,l=Ubuntu,c=main,b=i386 origin ca.archive.ubuntu.com
500 http://ca.archive.ubuntu.com/ubuntu bionic/main amd64 
    Packages release v=18.04,o=Ubuntu,a=bionic,n=bionic,
    l=Ubuntu,c=main,b=amd64 origin ca.archive.ubuntu.com
Pinned packages:
```

如果您渴望知道哪个存储库提供了特定的软件包，您可以使用`apt-cache policy`命令，后跟软件包名称：

```
apt-cache policy package_name
```

例如，要知道哪个存储库提供了`cmatrix`软件包，您可以运行：

```
root@ubuntu-linux:~# apt-cache policy cmatrix 
cmatrix:
 Installed: 1.2a-5build3 
 Candidate: 1.2a-5build3 
 Version table:
*** 1.2a-5build3 500
 500 http://ca.archive.ubuntu.com/ubuntu bionic/universe amd64 Packages
 100 /var/lib/dpkg/status
```

从输出中，您可以看到`cmatrix`软件包来自于[`ca.archive.ubuntu.com/ubuntu`](http://ca.archive.ubuntu.com/ubuntu)的 bionic/universe 存储库。

# 修补您的系统

如果某个软件包有可用的更新版本，那么您可以使用`apt-get install --only-upgrade`命令，后跟软件包名称来升级它：

```
apt-get install --only-upgrade package_name
```

例如，您可以通过运行以下命令来升级`nano`软件包：

```
root@ubuntu-linux:~# apt-get install --only-upgrade nano 
Reading package lists... Done
Building dependency tree
Reading state information... Done
nano is already the newest version (2.9.3-2).
The following package was automatically installed and is no longer required: 
 hoichess
Use 'apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 357 not upgraded.
```

您还可以通过运行以下命令来升级系统上安装的所有软件包：

1.  `apt-get update`

1.  `apt-get upgrade`

第一个命令`apt-get update`将更新可用软件包及其版本的列表，但不会进行任何安装或升级：

```
root@ubuntu-linux:~# apt-get update
Ign:1 http://dl.google.com/linux/chrome/deb stable InRelease 
Hit:2 http://ca.archive.ubuntu.com/ubuntu bionic InRelease
Hit:3 http://ppa.launchpad.net/linuxuprising/java/ubuntu bionic InRelease 
Hit:4 http://dl.google.com/linux/chrome/deb stable Release
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease 
Hit:6 http://ca.archive.ubuntu.com/ubuntu bionic-updates InRelease 
Hit:8 http://ca.archive.ubuntu.com/ubuntu bionic-backports InRelease 
Reading package lists... Done
```

第二个命令`apt-get upgrade`将升级系统上安装的所有软件包：

```
root@ubuntu-linux:~# apt-get upgrade 
Reading package lists... Done 
Building dependency tree
Reading state information... Done 
Calculating upgrade... Done
The following package was automatically installed and is no longer required: 
 hoichess
Use 'apt autoremove' to remove it.
The following packages have been kept back:
 gstreamer1.0-gl libcogl20 libgail-3-0 libgl1-mesa-dri libgstreamer-gl1.0-0 
 libreoffice-calc libreoffice-core libreoffice-draw libreoffice-gnome 
    libreoffice-gtk3 
 libwayland-egl1-mesa libxatracker2 linux-generic linux-headers-generic
 software-properties-common software-properties-gtk ubuntu-desktop 
The following packages will be upgraded:
 apt apt-utils aptdaemon aptdaemon-data aspell base-files bash bind9-host bluez 
 python2.7-minimal python3-apt python3-aptdaemon python3-aptdaemon.gtk3widgets 
 python3-problem-report python3-update-manager python3-urllib3 python3.6
342 upgraded, 0 newly installed, 0 to remove and 30 not upgraded. 
Need to get 460 MB of archives.
After this operation, 74.3 MB of additional disk space will be used. 
Do you want to continue? [Y/n]
```

请记住顺序很重要；也就是说，在运行`apt-get upgrade`命令之前，您需要运行`apt-get update`命令。

在 Linux 术语中，升级系统上安装的所有软件包的过程称为**打补丁系统**。

# 知识检查

对于以下练习，打开您的终端并尝试解决以下任务：

1.  在您的系统上安装`tmux`软件包。

1.  列出`vim`软件包的所有依赖项。

1.  在您的系统上安装`cowsay`软件包。

1.  删除`cowsay`软件包以及其所有配置文件。

1.  升级系统上的所有软件包（打补丁您的系统）。
