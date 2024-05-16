# 第四章：设计硬盘布局

在前一章中，我们专注于运行级别和引导目标。我们与运行`init`和`systemd`的 Linux 系统进行了交互。我们看到了如何启动服务，以及如何在运行级别和引导目标之间切换。我们查看了各种启动和停止脚本，还查看了脚本的结构。

本章重点介绍在 CLI 中创建分区和分割物理硬盘。我们将特别关注`fdisk`实用程序和`parted`实用程序的使用。然后，我们将逐步介绍使用各种`mkfs`命令创建、删除和定义分区类型以及格式化硬盘的步骤。最后，我们将探讨挂载和卸载分区的方法。

因此，我们将在本章中涵盖以下主题：

+   使用`fdisk`实用程序

+   使用`parted`实用程序

+   格式化硬盘的步骤

+   挂载和卸载分区

# 使用 fdisk 实用程序

在 Linux 中，每当我们使用硬盘时，我们很可能会在某个时候**分区硬盘**。*分区*简单地意味着分离硬盘。这使我们能够拥有不同大小的分区，并使我们能够满足各种软件安装要求。此外，当我们分区硬盘时，每个分区都被操作系统视为完全独立的硬盘。`fdisk`（固定磁盘或格式化磁盘）是一个基于命令行的实用程序，可用于操作硬盘。使用`fdisk`，您可以查看、创建、删除和更改等操作。

首先，让我们在 Ubuntu 分发中公开硬盘：

```
philip@ubuntu:~$ ls /dev/ | grep sd
sda
sda1
sda2
sda5
philip@ubuntu:~$
```

从前面的输出中，系统中的硬盘由`/dev/sda`表示。第一个分区是`/dev/sda1`，第二个分区是`/dev/sda2`，依此类推。为了查看分区信息，我们将运行以下命令：

```
philip@ubuntu:~$ fdisk -l /dev/sda
fdisk: cannot open /dev/sda: Permission denied
philip@ubuntu:~$
```

从前面的输出中，我们得到了`Permission denied`。这是因为我们需要 root 权限来查看和更改硬盘的分区。让我们以 root 用户重试：

```
philip@ubuntu:~$ sudo su
[sudo] password for philip:
root@ubuntu:/home/philip#
root@ubuntu:/home/philip# fdisk -l /dev/sda
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf54f42a0

Device   Boot Start     End       Sectors  Size Id  Type
/dev/sda1  *   2048     39845887  39843840 19G 83   Linux
/dev/sda2      39847934 41940991  2093058  1022M  5 Extended
/dev/sda5      39847936 41940991  2093056 1022M 82 Linux swap / Solaris
root@ubuntu:/home/philip#
```

从前面的输出中，阅读的方式如下：

磁盘`/dev/sda`：20 GiB，21,474,836,480 字节，41,943,040 扇区：这是实际的物理硬盘：

| **设备** | **引导 ** | **开始 ** | ** 结束** | ** 扇区** | **大小** | ** ID** | **类型** | **注释** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `/dev/sda1` | *  |  2048 |  39,845,887  |   39,843,840  | 19 G  | 83  |  Linux | 第一分区为 19 GB |
| `/dev/sda2` |  | 39,847,934  | 41,940,991 | 2,093,058 |  1,022 M  |   5 | 扩展 | 第二分区为 1,022 MB |
| `/dev/sda5  ` |  |   39,847,936  |   41,940,991  |   2,093,056  | 1,022 M |  82  | Linux swap / Solaris | 第五分区为 1,022 MB |

现在，为了能够进行任何更改，我们将再次使用`fdisk`命令。这次我们将省略`-l`选项：

```
root@ubuntu:/home/philip# fdisk /dev/sda

Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help):
```

从前面的代码中，我们现在在`fdisk`实用程序中，并且收到了一条不错的消息。

在进行任何更改之前，请确保您了解删除分区周围的危险；如果删除存储系统文件的分区，系统可能会变得不稳定，例如`/boot/`和`/`。

要查看可用选项，我们可以按`m`键：

```
Command (m for help): m
Help:
 DOS (MBR)
 a   toggle a bootable flag
 b   edit nested BSD disklabel
 c   toggle the dos compatibility flag
Generic
 d   delete a partition
 F   list free unpartitioned space
 l   list known partition types
 n   add a new partition
 p   print the partition table
 t   change a partition type
 v   verify the partition table
 i   print information about a partition

Misc
 m   print this menu
 u   change display/entry units
 x   extra functionality (experts only)
Script
 I   load disk layout from sfdisk script file
 O   dump disk layout to sfdisk script file

Save & Exit
 w   write table to disk and exit
 q   quit without saving changes
Create a new label
 g   create a new empty GPT partition table
 G   create a new empty SGI (IRIX) partition table
 o   create a new empty DOS partition table
 s   create a new empty Sun partition table

Command (m for help):
```

从前面的输出中，我们可以看到各种选择。我们甚至可以使用`l`来查看已知的分区类型：

![](img/00057.jpeg)

从前面的截图中，我们可以看到各种可用于使用的不同分区类型。常见类型包括`5 Extended`，`7 NTFS NTSF`，`82 Linux swap`，83（Linux），`a5 FreeBSD`，`ee GPT`和`ef EFI`等。

现在，要查看已创建的分区，我们可以使用`p`：

![](img/00058.jpeg)

我已经向该系统添加了第二个硬盘，因此让我们验证一下：

```
root@ubuntu:/home/philip# ls /dev/ | grep sd
sda
sda1
sda2
sda5
sdb
root@ubuntu:/home/philip#
```

太棒了！我们现在可以看到`/dev/sdb`。我们将使用`fdisk`处理这个新硬盘：

```
root@ubuntu:/home/philip# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x0079e169.
Command (m for help):
```

现在，让我们按下`p`，这将打印`/dev/sdb`上的当前分区：

```
Command (m for help): p
Disk /dev/sdb: 15 GiB, 16106127360 bytes, 31457280 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0079e169

Command (m for help):
```

如您所见，`/dev/sdb`上没有分区。为了创建一个分区，我们将使用`n`键：

```
Command (m for help): n
Partition type
 p   primary (0 primary, 0 extended, 4 free)
 e   extended (container for logical partitions)
Select (default p):
```

这将要求我们声明分区的类型。`fdisk`实用程序提供了主分区和扩展分区类型。还有一个逻辑分区类型。为了安装操作系统，我们将选择`p`，代表*主分区类型*。

您不会在逻辑分区类型上安装操作系统。

如您所见，我们使用`n`来创建新分区。需要注意的一个重要点是，到目前为止我们创建的分区都是 Linux 类型的分区。如果出于某种原因我们想要更改分区类型，我们可以使用`t`来更改它。让我们将`/dev/sdb2`更改为`HPFS/NTFS/exFAT`分区。我们将在`fdisk`实用程序中使用`type 7`：

```
Command (m for help): t
Partition number (1-3, default 3): 2
Partition type (type L to list all types): l
0  Empty  24  NEC DOS 81  Minix / old Lin bf  Solaris 
1  FAT12  27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
2  XENIX root  39  Plan 9  83  Linux  c4  DRDOS/sec (FAT-
3  XENIX usr   3c  PartitionMagic 84 OS/2 hidden or c6 DRDOS/sec (FAT-
4  FAT16 <32M  40  Venix 80286     85  Linux extended  c7  Syrinx 
5  Extended   41  PPC PReP Boot   86  NTFS volume set da  Non-FS data 
6  FAT16    42  SFS  87  NTFS volume set db  CP/M / CTOS / .
7  HPFS/NTFS/exFAT
```

太棒了！现在我们可以看到分区类型为`type 7`：

```
Partition type (type L to list all types): 7
Changed type of partition 'Empty' to 'HPFS/NTFS/exFAT'.
Command (m for help): p
Disk /dev/sdb: 15 GiB, 16106127360 bytes, 31457280 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x2584b986
Device       Boot     Start    End      Sectors Size  Id   Type
/dev/sdb1    2048     10487807  10485760  5G      83        Linux
/dev/sdb2    10487808 18876415  8388608   4G      7     HPFS/NTFS/exFAT
/dev/sdb3    18876416 31457279  12580864  6G      0           Empty
```

此外，我们将把`/dev/sdb3`分区更改为类型`ef`：

![](img/00059.jpeg)

现在当我们重新运行`p`命令时，我们可以看到我们新创建的分区类型设置为`ef`：

```
Device     Boot      Start     End     Sectors Size Id Type
/dev/sdb1   2048     10487807  10485760  5G    83 Linux
/dev/sdb2   10487808 18876415  8388608   4G    7 HPFS/NTFS/exFAT
/dev/sdb3   18876416 31457279  12580864  6G    ef EFI (FAT-12/16/32)
```

现在，如果我们决定安装操作系统，那么我们将不得不使其中一个分区可引导。我们将使第三个分区`/dev/sdb3`可引导：

```
Command (m for help): a 
Partition number (1-3, default 3): 3
The bootable flag on partition 3 is enabled now.
Command (m for help): p
Disk /dev/sdb: 15 GiB, 16106127360 bytes, 31457280 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x2584b986
Device     Boot    Start      End  Sectors Size Id Type
/dev/sdb1           2048 10487807 10485760   5G 83 Linux
/dev/sdb2       10487808 18876415  8388608   4G  7 HPFS/NTFS/exFAT
/dev/sdb3  *    18876416 31457279 12580864   6G ef EFI (FAT-12/16/32)
```

从前面的输出中，`/dev/sdb3`现在标记为可引导。

最后，要更改或保存我们的更改，我们将按`w`，保存并退出：

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

![](img/00060.jpeg)

# 使用 parted 实用程序

`parted`实用程序是针对我们有一个大于 2TB 的硬盘或硬盘的情况。此外，我们可以调整分区；`fdisk`实用程序无法调整分区。几乎所有较新的 Linux 发行版都支持`parted`实用程序。`parted`来自 GNU；它是一个基于文本的分区实用程序，可以与各种磁盘类型一起使用，例如 MBR、GPT 和 BSD 等。

在对分区进行任何更改之前，请备份数据。

首先，我们将在`/dev/sdb`上使用`parted`命令：

```
root@ubuntu:/home/philip# parted /dev/sdb
GNU Parted 3.2
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)                                                                 
```

从这里，我们进入了`parted`实用程序。与`fdisk`实用程序类似，`parted`实用程序是交互式的。现在让我们假设我们想查看`help`菜单，我们可以在 CLI 中列出`help`：

```
(parted) help 
 align-check TYPE N check partition N for TYPE(min|opt) alignment
 help [COMMAND] print general help, or help on COMMAND
 mklabel,mktable LABEL-TYPE create a new disklabel (partition table)
 mkpart PART-TYPE [FS-TYPE] START END make a partition
 name NUMBER NAME name partition NUMBER as NAME
 print [devices|free|list,all|NUMBER] display the partition table, available devices, free space, all found partitions, or a particular partition
 quit exit program 
 rescue START END rescue a lost partition near START and END
 resizepart NUMBER END resize partition NUMBER
 rm NUMBER delete partition NUMBER
 select DEVICE choose the device to edit
 disk_set FLAG STATE change the FLAG on selected device
 disk_toggle [FLAG] toggle the state of FLAG on selected device
 set NUMBER FLAG STATE change the FLAG on partition NUMBER
 toggle [NUMBER [FLAG]] toggle the state of FLAG on partition NUMBER
```

```
 unit UNIT set the default unit to UNIT
 version display the version number and copyright information of GNU Parted
(parted)
```

从前面的输出中，我们有一长串命令供我们使用。

在对分区进行任何更改之前，请备份数据。

现在，要查看`/dev/sdb`的当前分区表，我们将输入`print`：

```
(parted) print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 16.1GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
Number  Start   End     Size    Type  File system  Flags
 1     1049kB  5370MB 5369MB  primary
 2     5370MB  9665MB 4295MB  primary
 3     9665MB  16.1GB 6441MB  primary boot, esp
(parted) 
```

这将打印出`/dev/sdb`的分区表。但是，我们可以使用`print`命令和`list`选项来查看系统中所有可用的硬盘。让我们试试看：

```
(parted) print list
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 16.1GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
Number  Start   End     Size    Type    File system  Flags
 1      1049kB  5370MB  5369MB  primary
 2      5370MB  9665MB  4295MB  primary
 3      9665MB  16.1GB  6441MB  primary            boot, esp
 Model: VMware, VMware Virtual S (scsi)
Disk /dev/sda: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
Number  Start   End     Size    Type      File system     Flags
 1      1049kB  20.4GB  20.4GB  primary   ext4            boot
 2      20.4GB  21.5GB  1072MB  extended
 5      20.4GB  21.5GB  1072MB  logical   linux-swap(v1)
(parted)
```

太好了！如您所见，`/dev/sda`现在也被列出。接下来，让我们看看如何调整分区大小。为了实现这一点，我们将利用另一个强大的命令，即`resizepart`命令，这个命令本身的命名也很合适。

我们将选择第二个分区进行练习；我们将说`resizepart 2`，并将其减少到 2GB：

```
 (parted) resizepart
 Partition number? 2
 End? [5370MB]? 7518
 (parted) print
 Disk /dev/sdb: 16.1GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
Number Start End Size Type File system Flags
 1 1049kB 5370MB 5369MB primary
 2 5370MB 7518MB 2148MB primary
 3 9665MB 16.1GB 6441MB primary boot, esp
(parted)            
```

从前面的输出中，您可以看到`parted`实用程序非常强大。我们已经有效地从第二个分区中拿走了 2GB（大约）。现在，如果您考虑一下，我们有 2GB 的可用空间。

硬盘空间在大型数据中心中至关重要，因此在为服务器进行配置时请记住这一点。

现在，为了演示我们如何使用 2GB 的可用空间，让我们创建另一个分区。`parted`实用程序非常强大，它可以识别从其他磁盘实用程序（如`fdisk`）创建的分区。在`parted`中，我们将使用`mkpart`命令来创建一个分区：

```
(parted)
(parted) mkpart 
Partition type?  primary/extended? 
```

到目前为止，您可以看到`fdisk`和`parted`之间存在相似之处，它们都会询问分区是主分区还是扩展分区。这在我们处理操作系统安装时非常重要。为了我们的目的，我们将创建另一个主分区：

```
Partition type?  primary/extended? primary
File system type?  [ext2]?
Start?
```

现在，在这一点上，我们将不得不指定我们即将创建的分区的起始大小。我们将使用第二个分区结束的大小：

```
File system type? [ext2]? 
Start? 7518 
End? 9665 
(parted) 
```

太棒了！现在让我们重新运行`print`命令：

```
(parted) print 
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 16.1GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
Number Start   End     Size    Type     File system  Flags
 1      1049kB  5370MB  5369MB  primary
 2      5370MB  7518MB  2148MB  primary
 4      7518MB  9665MB  2146MB  primary  ext2         lba
 3      9665MB  16.1GB  6441MB  primary               boot, esp
(parted)
```

从前面的输出中，我们现在可以看到我们新创建的大约 2GB 的分区。

现在我们可以将`boot`标志从当前的第三个分区`/dev/sdb3`移动到第四个分区`/dev/sdb4`。我们将使用`set`命令：

```
(parted) set 
Partition number? 4 
Flag to Invert?
```

从这里开始，我们必须告诉`parted`实用程序，我们要移动`boot`标志：

```
Flag to Invert? boot 
New state?  [on]/off?
```

现在，我们需要确认我们的更改，`on`是默认值，所以我们按*Enter*：

```
New state?  [on]/off? 
(parted) print 
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 16.1GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:
Number  Start   End     Size    Type     File system  Flags
 1      1049kB  5370MB  5369MB  primary
 2      5370MB  7518MB  2148MB  primary
 4      7518MB  9665MB  2146MB  primary  ext2         boot, lba
 3      9665MB  16.1GB  6441MB  primary               esp
(parted)                                             
```

太棒了！现在我们可以看到`boot`标志已经移动到第四个分区`/dev/sdb4`。

最后，要保存我们的更改，我们只需输入`quit`：

```
(parted) quit 
Information: You may need to update /etc/fstab.
root@ubuntu:/home/philip#     
```

您需要在`/etc/fstab`中添加条目，以便自动挂载分区到它们各自的挂载点。

# 格式化硬盘的步骤

创建分区后，下一步是通过文件系统使分区可访问。在 Linux 中，当我们格式化分区时，系统会擦除分区，这使系统能够在分区上存储数据。

在 Linux 系统中有许多文件系统类型可用。我们使用`mkfs`命令结合所需的文件系统类型。要查看可用的文件系统，我们可以这样做：

![](img/00061.jpeg)

从前面的屏幕截图中，在这个 Ubuntu 发行版中，主要是`ext4`类型是当前使用的文件系统。我们还可以使用带有`-f`选项的`lsblk`命令来验证这一点：

```
root@ubuntu:/home/philip# lsblk -f
```

![](img/00062.jpeg)

从前面的屏幕截图中，我们可以看到两个硬盘，`/dev/sda`和`/dev/sdb`。此外，我们看到了一个`FSTYPE`列。这标识了当前正在使用的文件系统。我们可以看到整个`/dev/sdb(1-4)`的`FSTYPE`为空。

我们也可以使用`blkid`命令查看系统正在使用的文件系统：

![](img/00063.jpeg)

从给定的输出中，`TYPE=`部分显示正在使用的文件系统。请注意，对于`/dev/sdb(1-4)`，`TYPE=`都是缺失的。这意味着我们尚未格式化`/dev/sdb`上的任何分区。

现在让我们开始格式化我们的分区。我们将在`/dev/sdb1`上使用`ext4`文件系统：

```
root@ubuntu:/home/philip# mkfs.ext4 /dev/sdb1
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 1310720 4k blocks and 327680 inodes
Filesystem UUID: fc51dddf-c23d-4160-8e49-f8a275c9b2f0
Superblock backups stored on blocks:
 32768, 98304, 163840, 229376, 294912, 819200, 884736
Allocating group tables: done 
Writing inode tables: done 
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
root@ubuntu:/home/philip#
```

从前面的输出中，`mkfs`实用程序，特别是`mkfs.ext4`，在原始分区上创建文件系统；然后为`/dev/sdb1`分区分配一个 UUID 以唯一标识它。

在格式化分区之前，您需要具有 root 权限。

接下来，让我们在`/dev/sdb2`上使用`ext3`文件系统：

```
root@ubuntu:/home/philip# mkfs.ext3 /dev/sdb2
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 524288 4k blocks and 131328 inodes
Filesystem UUID: fd6aab0f-0f16-4922-86c1-11fcb54fc466
Superblock backups stored on blocks:
 32768, 98304, 163840, 229376, 294912
Allocating group tables: done 
Writing inode tables: done 
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
root@ubuntu:/home/philip#
```

现在我们将在`/dev/sdb3`上使用`ext2`，在`/dev/sdb4`上使用`ntfs`：

```
root@ubuntu:/home/philip# mkfs.ext2 /dev/sdb3
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 1572608 4k blocks and 393216 inodes
Filesystem UUID: b7e075df-541d-468d-ab16-e3ec2e5fb5f8
Superblock backups stored on blocks:
 32768, 98304, 163840, 229376, 294912, 819200, 884736
Allocating group tables: done 
Writing inode tables: done 
Writing superblocks and filesystem accounting information: done
root@ubuntu:/home/philip# mkfs.ntfs /dev/sdb4
Cluster size has been automatically set to 4096 bytes.
Initializing device with zeroes: 100% - Done.
Creating NTFS volume structures.
mkntfs completed successfully. Have a nice day.
root@ubuntu:/home/philip#
```

您还可以使用`mk2fs`来创建`ext2`文件系统。

太棒了！现在我们刚刚格式化了`/dev/sdb1`，`/dev/sdb2`，`dev/sdb3`和`/dev/sdb4`。如果我们现在使用`lsblk`命令和`-f`选项重新运行，我们将看到两个分区的文件系统类型(`FSTYPE`)已经填充：

```
root@ubuntu:/home/philip# lsblk -f
NAME   FSTYPE LABEL UUID                                 MOUNTPOINT
sda 
├─sda1 ext4         adb5d090-3400-4411-aee2-dd871c39db38 /
├─sda2 
└─sda5 swap         025b1992-80ba-46ed-8490-e7aa68271e7b [SWAP]
sdb 
├─sdb1 ext4         fc51dddf-c23d-4160-8e49-f8a275c9b2f0
├─sdb2 ext3         fd6aab0f-0f16-4922-86c1-11fcb54fc466
├─sdb3 ext2         b7e075df-541d-468d-ab16-e3ec2e5fb5f8
└─sdb4 ntfs         1D9E4A6D4088D79A 
sr0 
root@ubuntu:/home/philip#
```

从前面的输出中，我们可以看到`FSTYPE`反映了我们所做的更改。

我们还可以重新运行`blkid`命令，查看为`/dev/sdb1`和`/dev/sdb2`创建的 UUID：

```
root@ubuntu:/home/philip# blkid
/dev/sda1: UUID="adb5d090-3400-4411-aee2-dd871c39db38" TYPE="ext4" PARTUUID="f54f42a0-01"
/dev/sda5: UUID="025b1992-80ba-46ed-8490-e7aa68271e7b" TYPE="swap" PARTUUID="f54f42a0-05"
/dev/sdb1: UUID="fc51dddf-c23d-4160-8e49-f8a275c9b2f0" TYPE="ext4" PARTUUID="7e707ac0-01"
/dev/sdb2: UUID="fd6aab0f-0f16-4922-86c1-11fcb54fc466" SEC_TYPE="ext2" TYPE="ext3" PARTUUID="7e707ac0-02"
/dev/sdb3: UUID="2a8a5768-1a7f-4ab4-8aa1-f45d30df5631" TYPE="ext2" PARTUUID="7e707ac0-03"
/dev/sdb4: UUID="1D9E4A6D4088D79A" TYPE="ntfs" PARTUUID="7e707ac0-04"
root@ubuntu:/home/philip#
```

如您所见，系统现在可以存储有关各个分区的信息。

# 挂载和卸载分区

在格式化分区后的最后一步是挂载分区。我们使用`mount`命令来挂载分区，使用`unmount`命令来卸载分区。`mount`命令还用于查看系统中当前的挂载点。但是，在重新启动后，除非我们在`/etc/fstab`目录中创建了条目，否则所有分区都将被卸载。

在`/etc/fstab`中保存任何更改都需要 root 权限。在进行任何更改之前，也要备份任何配置文件。

# 挂载命令

我们可以发出`mount`命令而不带任何参数来查看当前的挂载点：

```
root@ubuntu:/home/philip# mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=478356k,nr_inodes=119589,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=99764k,mode=755)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/lib/systemd/systemd-cgroups-agent,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup(rw,relatime,user_id=0,group_id=0,default_permissions,allow_other)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=99764k,mode=700,uid=1000,gid=1000)
gvfsd-fuse on /run/user/1000/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
root@ubuntu:/home/philip#
```

出于简洁起见，部分输出被省略。

从前面的输出中，我们可以看到许多挂载点（挂载点只是将分区/驱动器与文件夹/目录关联起来）。我们可以过滤`mount`命令，只显示`/dev/`：

```
root@ubuntu:/home/philip# mount | grep /dev
udev on /dev type devtmpfs (rw,nosuid,relatime,size=478356k,nr_inodes=119589,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
mqueue on /dev/mqueue type mqueue (rw,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime)
root@ubuntu:/home/philip#
```

根据过滤器，我们可以看到`/dev/sda1`目前挂载在`/`目录上。如您所知，`/`目录是根目录。所有其他目录都属于`/`目录。

我们还可以使用带有`-h`选项的`df`命令查看更简洁的输出：

```
root@ubuntu:/home/philip# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            468M     0  468M   0% /dev
tmpfs            98M  6.2M   92M   7% /run
/dev/sda1        19G  5.1G   13G  29% /
tmpfs           488M  212K  487M   1% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           488M     0  488M   0% /sys/fs/cgroup
tmpfs            98M   44K   98M   1% /run/user/1000
root@ubuntu:/home/philip#
```

太好了！现在这是以结构化格式呈现的，更容易阅读。根据输出，只有`/dev/sda1`分区当前被挂载。

现在我们可以继续挂载`/dev/sdb1`到`/mnt`上。`/mnt`是一个空目录，我们在想要挂载分区时使用它。

一次只能挂载一个分区。

我们将运行以下`mount`命令：

```
root@ubuntu:/# mount /dev/sdb1 /mnt
root@ubuntu:/#
```

请注意，没有任何选项，`mount`命令可以正常工作。现在让我们重新运行`mount`命令，并过滤只显示`/dev`：

```
root@ubuntu:/# mount | grep /dev
udev on /dev type devtmpfs (rw,nosuid,relatime,size=478356k,nr_inodes=119589,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
mqueue on /dev/mqueue type mqueue (rw,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime)
/dev/sdb1 on /mnt type ext4 (rw,relatime,data=ordered)
root@ubuntu:/#
```

根据前面的输出，我们可以看到`/dev/sdb1`目前挂载在`/mnt`上。

我们还可以利用带有`h`选项的`df`命令来查看类似的结果：

```
root@ubuntu:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            468M     0  468M   0% /dev
tmpfs            98M  6.2M   92M   7% /run
/dev/sda1        19G  5.1G   13G  29% /
tmpfs           488M  212K  487M   1% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           488M     0  488M   0% /sys/fs/cgroup
tmpfs            98M   44K   98M   1% /run/user/1000
/dev/sdb1       4.8G   10M  4.6G   1% /mnt
root@ubuntu:/#
```

从前面的输出中，我们可以看到分区的大小以及与分区关联的挂载点。

现在让我们创建两个目录，用于`/dev/sdb2`和`/dev/sdb4`分区：

```
root@ubuntu:/# mkdir /folder1
root@ubuntu:/# mkdir /folder2
root@ubuntu:/# ls
bin  dev   folder2  initrd.img.old  lost+found  opt run srv usr      vmlinuz.old
boot etc  home lib   media    proc  sbin  sys  var
cdrom  folder1  initrd.img  lib64     mnt         root  snap  tmp  vmlinuz
root@ubuntu:/#
```

现在我们将把`/dev/sdb2`和`/dev/sdb4`挂载到`/folder1`和`/folder2`目录中：

```
root@ubuntu:/# mount /dev/sdb2 /folder1
root@ubuntu:/# mount /dev/sdb4 /folder2
root@ubuntu:/#
root@ubuntu:/# mount | grep /dev
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
/dev/sdb1 on /mnt type ext4 (rw,relatime,data=ordered)
/dev/sdb2 on /folder1 type ext3 (rw,relatime,data=ordered)
/dev/sdb4 on /folder2 type fuseblk (rw,relatime,user_id=0,group_id=0,allow_other,blksize=4096)
root@ubuntu:/#
```

太好了！现在我们可以看到我们的挂载点在`mount`命令中显示出来。同样，我们可以使用带有`-h`选项的`df`命令以可读的格式显示：

```
root@ubuntu:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            468M     0  468M   0% /dev
tmpfs            98M  6.2M   92M   7% /run
/dev/sda1        19G  5.1G   13G  29% /
tmpfs           488M  212K  487M   1% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           488M     0  488M   0% /sys/fs/cgroup
tmpfs            98M   44K   98M   1% /run/user/1000
/dev/sdb1       4.8G   10M  4.6G   1% /mnt
/dev/sdb2       2.0G  3.1M  1.9G   1% /folder1
/dev/sdb4       2.0G   11M  2.0G   1% /folder2
root@ubuntu:/#
```

正如您所看到的，挂载分区的步骤非常简单。但是，在某些发行版上，您将不得不指定文件系统类型。在网络中，挂载共享是一种常见的操作。挂载共享的一个例子如下：

```
root@ubuntu:/#mount //172.16.175.144/share /netshare -t cifs  -o user=philip,password=pass123,uid=1000,gid=1000,rw
```

# 卸载命令

在挂载了分区并进行了更改之后，清理和卸载分区总是一个好主意。我们使用`unmount`命令来卸载分区。

在运行`unmount`命令之前，始终更改/移出目录。

让我们卸载`/dev/sdb1`。格式如下：

```
root@ubuntu:/# umount /dev/sdb1
root@ubuntu:/#
root@ubuntu:/# mount | grep /dev
udev on /dev type devtmpfs (rw,nosuid,relatime,size=478356k,nr_inodes=119589,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
mqueue on /dev/mqueue type mqueue (rw,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime)
/dev/sdb2 on /folder1 type ext3 (rw,relatime,data=ordered)
/dev/sdb4 on /folder2 type fuseblk (rw,relatime,user_id=0,group_id=0,allow_other,blksize=4096)
root@ubuntu:/#
```

现在我们可以看到`/dev/sdb1`不再挂载；我们也可以使用`df`命令来确认：

```
root@ubuntu:/# df -h
Filesystem Size Used Avail Use% Mounted on
udev 468M 0 468M 0% /dev
tmpfs 98M 7.5M 91M 8% /run
/dev/sda1 19G 5.2G 13G 30% /
tmpfs 488M 212K 487M 1% /dev/shm
tmpfs 5.0M 4.0K 5.0M 1% /run/lock
tmpfs 488M 0 488M 0% /sys/fs/cgroup
tmpfs 98M 48K 98M 1% /run/user/1000
/dev/sdb2 2.0G 3.1M 1.9G 1% /folder1
/dev/sdb4 2.0G 11M 2.0G 1% /folder2
root@ubuntu:/#
```

我们还可以使用`lsblk`命令来确认相同的情况：

```
root@ubuntu:/# lsblk -f
NAME   FSTYPE LABEL UUID       MOUNTPOINT
sda 
├─sda1 ext4         adb5d090-3400-4411-aee2-dd871c39db38 /
├─sda2 
└─sda5 swap         025b1992-80ba-46ed-8490-e7aa68271e7b [SWAP]
sdb 
├─sdb1 ext4         fc51dddf-c23d-4160-8e49-f8a275c9b2f0
├─sdb2 ext3         fd6aab0f-0f16-4922-86c1-11fcb54fc466 /folder1
├─sdb3 ext2         2a8a5768-1a7f-4ab4-8aa1-f45d30df5631
└─sdb4 ntfs         1D9E4A6D4088D79A                     /folder2
sr0 
root@ubuntu:/#
```

现在让我们也卸载`/dev/sdb2`：

```
root@ubuntu:/# umount /folder1
```

![](img/00064.jpeg)

从前面的截图中，您会注意到，我使用了目录`/folder1`而不是分区`/dev/sdb2`；这完全取决于您；它们都被接受。此外，我们可以从`lsblk`命令中看到，`/dev/sdb2`没有列出挂载点。

现在，假设您希望在系统重新启动期间保持挂载点。那么，请放心，我们可以通过在`/etc/fstab`中创建条目来实现这一点。

首先，让我们在`/etc/fstab`中为`/dev/sdb4`创建一个条目。我们将使用`/dev/sdb4`的 UUID 来帮助我们。让我们运行`blkid`并保存`/dev/sdb4`的 UUID：

```
root@ubuntu:/# blkid
/dev/sdb4: UUID="1D9E4A6D4088D79A" TYPE="ntfs" PARTUUID="7e707ac0-04"
root@ubuntu:/#
```

现在让我们编辑`/etc/fstab`文件：

```
# /etc/fstab: static file system information. #
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=adb5d090-3400-4411-aee2-dd871c39db38 / ext4 errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=025b1992-80ba-46ed-8490-e7aa68271e7b none swap sw 0   0
/dev/fd0  /media/floppy0  auto   rw,user,noauto,exec,utf8 0  0
UUID=1D9E4A6D4088D79A   /folder2   ntfs    0       0
```

现在最后一个条目引用了`/dev/sdb4`。格式以分区开头，由`UUID`表示，然后是`挂载点`，`文件系统`，`转储`和`通过`。

当系统重新启动时，`/dev/sdb4`将被挂载到`/folder2`上。这样可以避免重复输入。

# 总结

在本章中，我们看了如何格式化硬盘以及各种可用的分区实用程序。我们使用`fdisk`实用程序创建了分区，并打开了`boot`标志。然后我们看了`parted`实用程序，以及如何创建分区；此外，我们还看到了如何调整分区的大小。这在数据中心环境中非常有用。然后我们格式化了我们的分区，这使我们能够开始存储数据。我们研究了使用各种`mkfs`命令。然后我们专注于如何挂载我们的分区。在我们的挂载点上保存数据后，我们卸载了我们的分区/挂载点。最后，我们看到了如何通过在`/etc/fstab`文件中创建条目来避免重复输入；这在启动时为我们挂载了我们的分区。

接下来，在下一章中，我们将介绍安装各种 Linux 发行版。我们将特别关注红帽发行版，即 CentOS。另一方面，我们将介绍 Debian 发行版，特别是 Ubuntu 以及安装 Linux 发行版的最佳技术，这些技术在不同的发行版之间略有不同。此外，我们将介绍双引导环境，让我们面对现实吧，迟早你会在 Linux 职业生涯中接触到 Windows 操作系统。不过，你不用担心，因为我们会逐步详细介绍安装过程的每一步。完成下一章后，你肯定会在跨所有平台上安装 Linux 发行版的方法上变得更加熟练。安装 Linux 发行版所获得的技能将对您作为 Linux 工程师大有裨益。

# 问题

1.  哪个字母用于列出硬盘的分区，而不进入`fdisk`实用程序？

A. `fdisk –a /dev/sda`

B. `fdisk –c /dev/sda`

C. `fdisk –l /dev/sda`

D. `fdisk –r /dev/sda`

1.  哪个字母用于在`fdisk`实用程序内创建分区？

A. *b*

B. *c*

C. *r*

D. *n*

1.  哪个字母用于在`fdisk`实用程序内切换引导标志？

A. *b*

B. *a*

C. *d*

D. *c*

1.  哪个字母用于在`fdisk`实用程序内打印已知的分区类型？

A. *l*

B. *r*

C. *n*

D. *b*

1.  哪个字母用于在`fdisk`实用程序内创建分区？

A. *p*

B. *n*

C. *c*

D. *d*

1.  哪个字母用于在`fdisk`实用程序内写入更改？

A. *q*

B. *c*

C. *d*

D. *w*

1.  哪个命令用于启动`parted`实用程序？

A. `part -ad`

B. `parted`

C. `part -ed`

D. `part`

1.  哪个选项用于在`parted`实用程序内显示分区表？

A. `display`

B. `parted`

C. `print`

D. `console`

1.  哪个选项用于从 CLI 中挂载分区？

A. `mount /dev/sdb1`

B. `mnt /dev/sdb1`

C. `mt /dev/sdb1`

D. `mont /dev/sdb1`

1.  哪个命令在 CLI 上显示已知分区的 UUID？

A. `blkid`

B. `df -h`

C. `du -h`

D. `mount`

# 进一步阅读

+   您可以通过查看以下内容获取有关 CentOS 发行版的更多信息，例如安装、配置最佳实践等：[`www.centos.org`](https://www.centos.org)。

+   以下网站为您提供了许多有用的技巧和 Linux 社区用户的最佳实践，特别是适用于 Debian 发行版（如 Ubuntu）的：[`askubuntu.com`](https://askubuntu.com)。

+   最后，这个链接为您提供了与在 CentOS 和 Ubuntu 上运行的各种命令相关的一般信息。您可以在那里发布您的问题，其他社区成员将会回答：[`www.linuxquestions.org`](https://www.linuxquestions.org)。
