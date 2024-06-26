# 第十三章：BSD、Solaris、Windows、IaaS 和 PaaS 以及 DevOps

本章将涵盖以下主题：

+   确定你所在系统的类型

+   了解 BSD 的不同之处

+   了解 Solaris 和 illumos 的不同之处

+   了解 Windows 的不同之处

+   IaaS（基础设施即服务）

+   PaaS（平台即服务）

+   运维与 DevOps 之争

# 介绍

当我被要求写这本书时，最初是要求我写 12 章关于 Linux 系统和现代管理的。现在我希望我当初同意了那个提议，但我却大胆地建议增加第十三章。我真是个傻瓜。

所以在这里，这是这篇艰难阅读的最终章节（抱歉，真的很抱歉），它关于计算世界中你需要了解的其他系统，因为不幸的是，现代计算和 IT 基础设施经常是 Windows、Linux 和它们之间的混合体。

我们将简要介绍一下 BSD，因为它们可能是你在当今时代最接近“真正”Unix 的系统，它们也足够接近 Linux，以至于有些 BSD 用户在你使用“它们足够接近 Linux”这样的短语时会感到愤怒。

然后，我们将简要讨论 Solaris，并谈谈它在现代基础设施中的两种形式。

我们将不得不讨论 Windows，尽管我将尝试尽量简短地介绍这一部分。如果你和我一样讨厌“Windows”这个词以及它带来的所有内涵。（微软，你近年来对 Linux 的半嬉皮式态度并不能愚弄我们——我们中有些人喜欢固执地守旧。）

在我们探索其他操作系统之后，我们还将研究基础设施即服务（IaaS）和平台即服务（PaaS），尽管这些缩写有多愚蠢，因为它们是现代 DevOps 和平台创建的重要组成部分。在任何完整的职业生涯中，你都将不得不使用 AWS 和 Azure 等服务，所以最好尽早了解它们的工作方式。

最后，我们将谈一下 DevOps（我会在那一节保留惊喜）。

# 确定你所在系统的类型

想象一下：你被蒙上眼睛，被塞进车后备箱，然后在长途旅程的另一端被揭开面纱，看到一个闪烁的提示符。你将如何确定你所在的系统类型？

你的第一反应可能是认为你被放在了一个 Linux 盒子前，但这并不是确定的。虽然 Linux 确实主导了服务器领域，但只是因为实例有黑屏、白色文本和登录提示符，并不意味着你被放在了我们友好的企鹅操作系统前。

它可能是 Linux、BSD 系统、Solaris 系统，或者是九十年代的许多 Unix 衍生系统之一。

假设你已经获得了登录凭据，那就登录吧。

# 如何做…

这是一个简单的起点。

# uname

当你成功登录后，确定你正在运行的内核类型：

```
$ uname
Linux
```

好吧，这真是令人失望…这只是一个普通的 Linux 系统。

但如果不是呢？想象一下：

```
$ uname
FreeBSD
```

这更像是一个 FreeBSD 盒子！

或者是另一种 BSD？

```
$ uname
OpenBSD
```

一个 OpenBSD 盒子，很酷！但我们可以更进一步：

```
$ uname
SunOS
```

嗯？`SunOS`到底是什么？

简短的答案是你可以假设你已经登陆到了 Oracle Solaris 或 illumos 发行版，它们都相对罕见，但值得尊重。

# 文件系统检查

如果你还不确定，你可以快速检查 slash-root（`/`）所使用的文件系统类型：

```
$ mount | grep "/ "
/dev/mapper/VolGroup00-LogVol00 on / type xfs (rw,relatime,seclabel,attr2,inode64,noquota)
```

XFS 通常出现在 Linux 系统上，特别是 RHEL 和 CentOS：

```
$ mount | grep "/ "
/dev/ada0s1a on / (ufs, local, journaled soft-updates)
```

Unix 文件系统（UFS）通常出现在 FreeBSD 盒子上（如果你有足够的 RAM，还会有 ZFS）：

```
$ mount | grep "/ "
/dev/wd0a on / type ffs (local)
```

FFS？你在开玩笑…不，这是快速文件系统（FFS），它通常用于 OpenBSD 盒子：

FFS 和 UFS 有相同的传承，尽管代码库并非完全相同。

```
$ mount | grep "/ "
/ on rpool/ROOT/openindiana read/write/setuid/devices/dev=4410002 on Thu Jan 1 00:00:00 1970
```

虽然我们在这里没有得到实际的文件系统类型，但我们可以看到输出中列出了`openindiana`，我们知道这是一个 illumos 发行版。然后我们可以使用`zfs`来确定我们的文件系统设计是什么（并确认它是`zfs`）：

```
$ zfs list
NAME USED AVAIL REFER MOUNTPOINT
rpool 6.82G 41.1G 31.5K /rpool
rpool/ROOT 2.70G 41.1G 23K legacy
rpool/ROOT/openindiana 2.70G 41.1G 2.46G /
rpool/ROOT/openindiana/var 203M 41.1G 202M /var
rpool/dump 2.00G 41.1G 2.00G -
rpool/export 73.5K 41.1G 23K /export
rpool/export/home 50.5K 41.1G 23K /export/home
rpool/export/home/vagrant 27.5K 41.1G 27.5K /export/home/vagrant
rpool/swap 2.13G 43.3G 12K -
```

# 它是如何工作的...

尽管提示可能看起来相同（尽管通常，这些发行版不会将 Bash 设置为默认 shell；这是 Linux 的事情），但底层系统可能与您熟悉的 GNU 用户空间大不相同。

当我们运行`uname`时，正如我们之前讨论过的，我们输出了我们登录到的系统的内核。

希望它是一个 Linux 系统，特别是如果您被迫去修复某些东西，但即使您对 Linux 有一定了解，事情也应该相对熟悉。进程运行，默认工具相同，您可以阅读`README`文件或 man 页面来了解您尚不了解的内容。

我们进行的文件系统检查并不是最科学的，但通常可以保证像`mount`和`df`这样的命令对您是可用的，这要归功于它们的血统。

# 还有更多...

在前面的部分中有一个显而易见的遗漏，那就是如何确定您是否在 Windows 系统上工作。

我发现检查我是否在 Windows 提示符下的最简单方法是测量我的灵魂从身体中流逝的速度。如果失败，我会寻找文件系统中的“音乐”文件夹，这似乎不可思议地出现在桌面和服务器安装中。

显然，如果前面两种方法对您无效，那么请考虑 Windows 通常具有图形用户界面（除非是现代服务器操作系统与时尚的系统管理员，那么可能只是一个蓝色的 PowerShell 提示符。无论哪种情况，我想您会知道）。

# 了解 BSD 的不同之处

您可能已经注意到，我在本章中故意将 OpenBSD 和 FreeBSD 分开，但它们只是不同的“BSD”发行版，对吧？

错误。

与 Linux 不同，BSD 的不同“风味”不共享内核，更像是不同的操作系统而不是不同的发行版。

OpenBSD、FreeBSD、NetBSD 和 Dragonfly BSD 都是独特而独立的项目。

NetBSD 甚至有一个 Dreamcast 端口。最后，那个控制台有了用途！

这并不是说各个发行版之间没有代码和修复的共享——只要移植比编写自己的实现更快，而且 BSD 更有可能使用更“自由”的开源许可证，比如 MIT 许可证，而不是“限制性”的开源许可证，比如 GPL（通常出于意识形态原因）。

# 不同之处

正如我们已经说过的，BSD 是独立的操作系统，其受欢迎程度的估计排名如下：

+   FreeBSD

+   OpenBSD

+   NetBSD

+   其他 BSD

在这里，我将谈到两个最受欢迎和知名的：FreeBSD 和 OpenBSD。

# FreeBSD

作为一个关注多样化的操作系统（服务器、桌面、物联网设备），FreeBSD 是 BSD 衍生版中最受欢迎的。

它以拥有大量可用软件而自豪，既有预先构建的软件包（在撰写时每季度构建一次），也有本地构建的“端口”，通常是该软件的最新版本。

许多开源软件都是专注于 Linux 的。这并不是 BSD 的错——这只是市场份额的问题。由于这种关注，FreeBSD 项目有很多志愿者将他们的时间投入到使软件包在 FreeBSD 上本地运行，有时需要对软件进行小的甚至重大的调整才能使其编译和运行。然后，这些努力大部分都会被推回上游，我还没有看到一个项目不感谢 FreeBSD 支持的。

使 FreeBSD 成为 BSD 思维人士的热门选择的另一个因素是它作为标准附带 ZFS，并提示您使用 ZFS 作为设备的根文件系统以及额外存储的文件系统。

ZFS（特别是 OpenZFS）是一个文件系统、存储管理和全方位的存储解决方案。我听说它被称为“文件系统的最后一句话”，而它的许可证使它在 Linux 平台上变得不常见（FreeBSD 没有这样的顾虑）。

拥抱，或者**通用开发和分发许可证**（** CDDL **）在开源世界中是一种相当罕见的许可证。由 Sun Microsystems 在其鼎盛时期制作，该许可证曾被称为与 GPL“不兼容”。

虽然 ZFS 确实是存储的一个很好的解决方案，但对于新手来说可能会有点困惑，因为它不遵循传统文件系统的模式（甚至是 UFS——FreeBSD 的更简单的替代方案），并且在文件系统和分区之间模糊了很多界限。它也有一些缺陷。由于其设计，ZFS 需要大量内存，而鉴于近年来内存价格的波动，这对终端用户来说可能是障碍。

FreeBSD 还注重稳定性，这意味着它包括了一些功能，比如能够轻松回滚更改（如果使用 ZFS），并且有一个非常可靠的升级路径（尽管可能有点令人困惑和复杂）。

监狱也值得一提，因为如果我不提到它们，FreeBSD 的粉丝们会感到恼火。在 Docker 之前，有监狱，可以在 FreeBSD 系统上将 OS 的部分分隔开来。它们于 2000 年引入，允许 FreeBSD 用户在主机系统上隔离其软件，甚至在彼此之间安装不同版本的 FreeBSD，以便专门为 FreeBSD 9 编写的软件可以在 FreeBSD 12 上运行。

不公平的是，监狱并没有真正起飞，尽管这在很大程度上归因于 FreeBSD 的市场份额。尽管它们可以说更优越，但它们也比 Docker 和其他容器解决方案更加笨重。我认为 Docker 之所以如此成功，很大程度上归功于像 Docker Hub 这样的地方，而 FreeBSD 则缺乏类似的东西。

总之，我可以这样总结 FreeBSD：

+   **优点**：

+   附带 ZFS

+   提供非常新的软件包

+   稳定的系统

+   FreeBSD 监狱

+   第一方软件包和第三方软件包的合理分割（附加软件包通常完全安装在`/usr/local/`下）

+   非常好的“FreeBSD 手册”

+   **缺点**：

+   安装基数比 Linux 系统小

+   不太可能有更新的驱动程序（尽管近年来情况有所改善）

+   通常更倾向于较旧的解决方案（X 优于 Wayland）

+   升级过程可能会令人困惑

+   有时会被供应商忽视补丁和安全披露（考虑到英特尔 CPU 的漏洞）

FreeBSD 是一个不错的选择，但我故意不推荐给 Windows 或 macOS 的切换者。相反，我会把他们指向像 Ubuntu 和 Fedora 这样的流行 Linux 发行版。FreeBSD 有它的位置，对于存储服务器来说，你可能会有更糟糕的选择，但它并不是很多人心目中的第一选择。

# OpenBSD

OpenBSD 是一个几乎传奇般专注于安全和稳定性的操作系统，它是因为担心保留它的“Linux 兼容层”的安全性而将其删除的软件。

如果你从未听说过或使用过 OpenBSD，我可以保证你至少使用过一个相关项目。一些属于 OpenBSD 领域的项目包括以下内容：

+   LibreSSL

+   OpenSMTPD

+   OpenNTPD

+   OpenSSH

+   httpd（这是 OpenBSD 特定的软件包，而不是重新打包的 Apache）

这意味着虽然你可能从未 SSH 连接到过 OpenBSD 系统本身，但你仍然在主要基于 OpenBSD 系统构建的软件上使用。我知道没有一个 Linux 发行版不使用 OpenSSH 作为默认的 SSH 客户端和服务器（尽管也存在替代方案）。

除了软件，对于那些软件有多好，OpenBSD 还有更多。

作为一个相对较小的项目，再次强调它是开源和捐赠资助的，OpenBSD 的安装基数并不是很大，但在嵌入式系统、防火墙和路由器方面非常受欢迎。这是因为虽然它的多处理器元素可能没有其他一些项目那么强大，但它具有诸如**数据包过滤器**（**pf**）防火墙之类的软件，并以安全优先而闻名。

OpenBSD 在其网站上的标语是“默认安装中只有两个远程漏洞，而且已经很长时间了！”，这正好说明了他们对安全的承诺。我看到 OpenBSD 积极地删除软件，因为它存在可疑的安全问题。

LibreSSL 诞生于对 OpenSSL 的挫败感，以及开发者认为更容易分叉和修复该项目（而不是争取上游的增强安全性）。很容易看出，支撑互联网安全的软件应该保持安全。

以调整软件以减少漏洞出现的机会而闻名，这有时会对 OpenBSD 产生反作用，因为他们可能会因为害怕立即解决问题（而不是等到“官方”日期才公开漏洞）而被忽视。有关此问题的有趣案例研究，请参见“KRACK Attacks”漏洞，以及 OpenBSD 的响应（[`www.krackattacks.com/#openbsd`](https://www.krackattacks.com/#openbsd)）。

因为 OpenBSD 是与 FreeBSD 不同的操作系统，它不包括类似 jails 的功能，也没有 ZFS（而是支持 FFS）。在默认安装中，你几乎不能进行任何调整，有人认为“你不应该这样做”。

它可以用作桌面、服务器（通常是防火墙，据我所见），或嵌入式操作系统。

你可能可以这样总结 OpenBSD：

+   **优点**：

+   极其重视安全。

+   非常重视代码质量

+   合理的升级路径（尽管耗时）

+   频繁的发布周期和快速的修补

+   Pf，说实话很棒

+   **缺点**：

+   缺少软件包（尽管其端口系统有很多）

+   稳定和安全可能意味着缺乏功能（这取决于个人）

+   非常小的安装基数

+   FFS 老化且显示出年龄

+   Theo de Raadt（OpenBSD 的终身慈善独裁者）以直言不讳而闻名（你可以理解为你愿意的）。

我在一台笔记本电脑上使用 OpenBSD，主要是为了体验，但现在我们使用的是 6.4 版本，最初安装的是 6.0 版本（也有一些很棒的发布艺术作品）。笔记本电脑运行良好，尽管大部分时间都在闲置。我也尝试过将 OpenBSD 用作服务器，但出于我的罪恶感，我很快发现当我无法获得我认为非常标准的软件包时，这让我很烦恼。

# 了解 Solaris 和 illumos 的区别

我有点害怕这一部分，因为很难谈论像 Oracle 这样的公司而不至少有点恼火。

我们将简要介绍 Oracle 及其 Sun 收购的历史，以及 OpenSolaris 运动和由此产生的系统。

在我们开始之前，我应该指出 illumos 和 Solaris 有非常 Unix 的背景，它们可以说是当今“最纯粹”的 Unix 衍生产品。你可能从未使用过它们中的任何一个，但很有可能你使用过一个由这两个操作系统支持的网站或在线门户。

# 区别

首先，稍微了解一下历史。

Solaris（曾经的 SunOS）是在九十年代初由 Sun Microsystems 发布的，最初设计用于 Sun SPARC 系列处理器，尽管它很快就扩展到支持 x86 处理器。

有一段时间，Sun（以及它的紫色巨兽在数据中心随处可见）备受推崇，在数据中心是一个相当常见的景象。该公司不断发展，开发了诸如 Solaris Zones（就像我们之前讨论的 FreeBSD jails）、ZFS 和 Java 等东西。

并不是一个家庭用户系统，Solaris 在企业环境中与 SPARC 处理器一起很受欢迎，尽管随着 Linux 的发展和市场份额的增加，这个和其他替代操作系统很快就失去了市场。

2005 年，Sun 决定在 CDDL 许可下开源 Solaris，同时创建了“OpenSolaris”项目。这是为了围绕 Solaris 建立一个社区，可能增加使用率和市场份额。

然而，当甲骨文在 2009 年收购 Sun（2010 年完成）时，他们几乎立即停止了向源代码分发公共更新的做法，并有效地将 Solaris 11（因为它是）恢复为一个闭源产品。

他们无法把精灵重新装进瓶子里，而且代码已经被释放了一次，这意味着虽然 OpenSolaris 实际上已经死了，但派生版本仍在继续。

令人困惑的是，许多这些派生版本都属于“illumos”范畴，这可能是 OpenSolaris 项目的主要分支，还有一些相关项目，如 SmartOS（来自 Joyent，现在是三星公司）稍有偏离。

illumos（小写的“i”，出于某种原因）包括一个内核、系统库、系统软件和设备驱动程序。

一般来说，这意味着当人们现在提到“Solaris”时，他们要么是怀念旧时的 Sun，以及可能十年前做过的安装，要么是用它来指代甲骨文仍在发布、支持和据称正在积极开发的 Solaris 11。在撰写本文时，最新版本是 2018 年 8 月的 11.4。

在随意的谈话中，我将 SmartOS、OpenIndiana 和其他系统称为 Solaris，尽管这在技术上是不正确的，可能会导致我有一天收到甲骨文的愤怒信件。

# 甲骨文 Solaris

正如我们已经说过的，甲骨文仍然积极发布和支持 Solaris，尽管您这些天可能会遇到的很多安装都可能是旧的安装。Solaris 10，首次发布于 2005 年，从技术上讲仍在支持中。

Solaris 10 在 2021 年 1 月正式停止支持，尽管我愿意打赌，支持的持续时间取决于你的财力有多深。

具有 SPARC 处理器支持和倾向于数据库安装，Solaris 可能是您作为工程师在一生中会遇到的东西，如果您决定熟悉它，您很有可能会加入一个知道如何支持它的人群，这意味着您可能会发现自己很受欢迎。

请不要决定在未来的工作中学习甲骨文 Solaris 的所有细节，我不会对你发现自己成为一个死亡领域的专家负责。

ZFS 也是 Solaris 的一个强大特性，虽然 OpenZFS 项目尝试了跨兼容性，但由于代码基础的分歧和甲骨文似乎不愿意保持功能的一致性，这似乎在最近几年已经不再重要了（尽管不要相信我的话，我只是一个谣言的消费者）。

# illumos

OpenSolaris 项目的精神延续，illumos 构成了一些不同发行版的基础，这些发行版试图保持基于 CDDL 的 Solaris 的传统。

OpenIndiana 可能是这些发行版中最用户友好的，仍在不断增强。它可以在虚拟机中下载和尝试（我鼓励这样做，只是为了试试水）。

然而，当你运行它时，不要惊讶地发现有关 Solaris 和 SunOS 的引用：

```
$ uname -a
SunOS openindiana 5.11 illumos-42a3762d01 i86pc i386 i86pc
```

由于软件包规模较小，你也不太可能找到当今流行的软件，但它将具有你熟悉的编程语言，并且它无疑是稳定的。

甲骨文的 Solaris 和 illumos 在过去曾有一些优秀的头脑为其工作，这意味着它们也具有稳定的内核和合理的开发方法（如果你相信一些曾在其上工作的更有声望的工程师）。

遗憾的是，在 Solaris、BSD 和 Linux 世界中存在一定程度的冲突，每个人都对特定事物的“正确”做法有着自己的看法，尽管所有这些操作系统都可以追溯到一个共同的核心（Unix）。

就个人而言，我喜欢安装操作系统和进行调试——我就是这样奇怪。

# 了解 Windows 的不同之处

你将不得不使用 Windows——这是一个现实（至少目前是这样）。

如果你不是被迫在工作场所使用 Windows 作为桌面操作系统的选择，那么很可能你至少会有一个 Windows 服务器需要照看或管理。

Windows 仍然常见的安装原因有：

+   活动目录

+   使用 Exchange 进行电子邮件

+   文件服务器

它也被广泛应用于服务器领域，像 IIS 这样的软件在网络中占据了相当大的一部分（尽管远远小于开源软件的份额）。

从一开始，正如我们之前讨论的，Windows 和 Linux 在一些关键方面有所不同。

# 不同之处

Windows 是有许可的。这可能是开源软件和专有软件之间最明显的区别。如果你想在生产中使用 Windows，你必须确保你有正确的许可证，否则就会面临违规罚款。

有趣的是，这可能是 Linux 获得采用的最大原因。当面对一个可以用来做与 Windows 相同事情的免费软件时，大多数管理员至少会先尝试免费版本。

第二个最明显的区别，尽管这种情况正在慢慢改变，是 Windows 默认安装图形用户界面，而 Linux 更喜欢简单的文本提示符。

CLI 与 GUI 的争论已经持续了多年，我现在不打算继续，但我会说在一个几乎不会被登录的操作系统上耗费额外的资源来支持图形功能，似乎非常愚蠢和浪费。

虽然现在完全有可能安装一个不带图形界面的精简版 Windows（例如 Windows Server 2016 Nano），但在数据中心中仍然不常见，很多功能仍然主要由图形界面驱动（尤其是第三方软件）。

与 Linux 不同，Windows 采用远程桌面协议（RDP）作为其首选的连接方法，将服务器的桌面图形表示传送到本地计算机。

有趣的是，RDP 也开始被远程 PowerShell 连接和甚至 SSH 所取代（微软似乎开始欣赏 SSH 作为一种非常好的轻量级解决方案，尽管采用率仍然未知）。

显然，Windows 也没有采用 Linux 内核，而是使用 NT 内核与硬件和低级工作进行交互。

趣闻时间！

几年前，我和一个朋友在下班后在城市里散步时聊天。我当时年轻而天真，所以我随口提到 Linux 用户更多地使用 Windows 可能是个好主意，因为我们这边的采用只会推动对方的改进。

这导致了一阵喧闹的笑声和嘲笑，我的朋友在我们到达餐馆后向其他同事传达了这个疯狂的建议。大家都对我开了个玩笑，这件事就这样结束了，我们 Linux 人的普遍共识是 Windows 永远都是垃圾，没有什么能拯救它。

快进几年，同样的朋友现在积极倡导 PowerShell，在他的工作机器上安装了 Windows，并谈论使用 Windows 而不是 Linux 进行选择性任务的优点。

我提出这个问题是因为我怀疑他迟早会读到这本书，我只是想提醒他，在很久以前，他只是因为是 Windows 而对 Windows 不屑一顾。

时代确实在变化，尽管微软仍然因其在隐私等方面的立场而受到很多（可以说是合理的）批评，但他们似乎正在努力将他们的操作系统实现为云世界的解决方案。

阅读本文的一些人可能已经记得“拥抱、扩展和扑灭”（EEE），这是微软内部用来谈论其操纵开放和广泛采用的标准的计划的短语，通过说他们的产品可以做到其他产品能做的一切，甚至更多，然后推动开放和免费的替代品退出市场，扩展它们以专有的附加功能。 （想想 AD，基本上就是带有更多花哨功能的 LDAP。）

有一些人认为微软最近的“微软热爱 Linux”的立场只是一个策略，我们即将看到“EEE”方法的复兴。

# IaaS（基础设施即服务）

简而言之，IaaS 可以被简单地总结为“云服务器”。

IaaS 是云提供商用来表示你可以将所有那些又旧又吵又昂贵的本地服务器转移到“云”中的术语。

实际上，云只是“各种数据中心中的一堆服务器”的营销术语，这让许多工程师感到恼火，他们不喜欢那些只会混淆和困惑的羊毛术语。

像将基础设施转移到云中这样的做法的好处应该是显而易见的，我们之前已经讨论过“基础设施即代码”（IaC）的概念。

过去，部署开发环境意味着从分销商购买新服务器，架设和布线，确保你有一个体面的方法将你选择的操作系统安装在上面。

现在，从交换机到防火墙和服务器的基础设施可以只需点击几下鼠标，或者更好的是，键盘上输入几个命令就可以部署。

IaaS 意味着你可以在几秒钟内部署成千上万的服务器，并且可以同样快速地将它们拆除。这种可扩展性和部署便利性意味着以前需要为每月运行一次的作业而拥有整个数据中心的公司现在可以通过简单地为一个“十五分钟的作业”启动这些服务器，然后再次将它们删除来节省电力和冷却成本。

然而，使用云解决方案的最大好处可能是你不必担心底层硬件。其他人（提供商）负责确保你正在构建的服务器正常工作，通常你不必担心诸如硬件警报之类的事情。

# IaaS 提供商和功能

目前 IaaS 服务的最大提供商是亚马逊，他们提供的 AWS。一个在线零售商也是最大的云解决方案供应商可能看起来有点奇怪，但当你考虑到他们必须为自己的平台设计和构建的基础设施时，他们会想到将其作为服务出售。

AWS 得到了 IaC 工具（如 Terraform 和 Packer）的很好支持，成为一流的公民，它还拥有自己的工具，比如 CloudFormation（类似于 Terraform）。

有趣的是，AWS 出于某种奇怪的原因也混淆了一些名称，导致出现了像[`www.expeditedssl.com/aws-in-plain-english`](https://www.expeditedssl.com/aws-in-plain-english)这样的网站，该网站提供了亚马逊的一些名称以及相应的英文对照。

例如，EC2 基本上是亚马逊对服务器的称呼。

AWS 于 2006 年推出，这意味着他们在 2010 年推出的 Azure 之前就已经领先一步。

微软的 Azure 可能是云解决方案的第二大供应商，他们有一个额外的优势，即如果你的企业签署了使用 Office365 进行电子邮件和文档的协议，微软很有可能会推动你使用 Azure 来进行基础设施。

显然还有其他提供商，不是所有的 IaaS 都必须是远程解决方案。你可以在你的数据中心部署一个 OpenStack 安装，然后与其 API 交互，创建一个 IaaS 平台，用于编程地构建虚拟基础设施。显然，这里的警告是你仍然需要维护基础框、操作系统和 IaaS 软件。

谷歌有一个 IaaS 提供，甲骨文、Rackspace 和 IBM 也有。在较小的一边，你有 DigitalOcean、Scaleway 和 OVH。你可以选择使用哪一个，通常取决于提供的功能。

如果你有特定的要求（比如数据主权），你可能会发现你绝对必须使用国内提供的服务，这意味着你可能会发现一些 IaaS 提供商的竞争对手被立即排除在外，但通常会有一些东西来满足你的需求。

IaaS 为管理员提供了灵活性，这意味着你不再有低估特定软件的风险，因为你可以确定是否需要更多资源，然后简单地销毁这个盒子，然后创建一个更大型的新盒子。

IaaS 意味着你的防火墙和负载均衡器规则不再存储在一个不起眼的盒子的转储配置文件中。相反，你可以配置文本文件，然后在构建你的基础设施的同时读取和应用它们。

IaaS 甚至意味着你可以定期测试你的基础设施，按计划销毁和重建整个集群，只是为了确保操作系统更新或代码更改没有破坏任何东西。

# PaaS（平台即服务）

在 IaaS 的另一侧，或者与之并行的是 PaaS 的概念。

平台即服务几乎是基础设施虚拟化的逻辑延续，进一步抽象化，并提出了一个问题：“为什么我们要求用户为 PostgreSQL 启动服务器，而我们可以直接启动一个 PostgreSQL 实例呢？”

是的，总会有一个服务器存在，这是肯定的。这些服务不仅仅是在一个充满雾气的房间里飘忽不定地运行（尽管这是一个很酷的心理形象），但这种理念的关键部分是你不在乎。

实际上，你并不关心你的数据库运行在什么操作系统上，只要它确实在运行，而且不会立即崩溃。然而，作为过去的管理员，你将被要求做到这一点——保持对补丁和重新启动的掌握，以确保数据库本身继续运行。

因此，PaaS 作为一个概念听起来很美好。

为什么你要为托管网站、运行数据库或部署 Redis 而启动多个操作系统实例，当你可以使用现成的提供端点连接，并且不用担心操作系统呢？

在新世界中，你可以将你的网站部署到一个共享的网络段，连接到你指定的数据库，并与一个 Redis 端点进行交互，你根本不知道它运行在什么版本的 Linux 之上（如果它真的是 Linux 的话！）。

从理论上讲，这也意味着开发人员在编写软件时会更容易，因为他们不必关心特定的操作系统怪癖或可能影响他们的代码的微小差异。只要开发人员针对 PaaS 提供的共同平台，他们就不需要知道底层运行的操作系统是什么。

# PaaS 提供商和功能

与 IaaS 一样，AWS 和 Azure 都充斥着 PaaS 的提供。

最明显的产品是数据库，AWS 提供 PaaS 关系型数据库、NoSQL 数据库（DynamoDB）和缓存系统，比如 Amazon ElastiCache。

Azure 的数据库产品包括 MSSQL 部署（显然）和最近还包括了 PostgreSQL。

亚马逊提供域名服务，名为 Route 53（哈哈，伙计们，非常聪明，现在别再用愚蠢的命名了）。他们还提供 CDN 解决方案、VPN 设置和 Active Directory 服务。

Azure 还提供 CDN 解决方案，以及 Web 应用（用于托管 Web 应用和网站）、Azure Blob 存储和非关系型数据库产品，比如 Cosmos DB。甚至还有一个专门命名的“Redis 缓存”产品，让你不必自己创建。

这个列表还在继续，这意味着在部署中使用的潜力中，对于绿地项目来说迷失并不罕见。我敢说，对于任何 21 世纪的系统管理员来说，一个很好的经验法则应该是“如果你不必管理它，就不要尝试”。

如果你可以使用 PaaS 选项，那就使用吧，因为从长远来看，它会为你节省大量的麻烦。虽然一开始的学习曲线可能很陡峭，但下次出现重大操作系统漏洞并需要开始一轮关键的补丁时，你会庆幸自己选择了 PaaS 产品。

Azure 和 AWS 是最明显的两个，但与 IaaS 一样，还存在其他提供商。GCP（Google Compute Platform）是最明显的第三个竞争者，但更小的提供商也在市场上迈出了试探性的第一步。

DigitalOcean 已经开始提供像托管的 Kubernetes（这在某种程度上是 PaaS，因为你可以将自己的容器部署到托管平台中）、块存储和负载均衡器等产品。Scaleway 已经开始公测对象存储（在撰写本文时）。

我认为 PaaS 最大的问题在于，为了对最终用户无缝使用，背后需要投入大量工作。

你像消耗无物一样使用数据库产品，创建你的模式在一个透明的结构之上，但在这个链条的某个地方，有人必须设计和维护这些 PaaS 产品所依赖的系统……我希望他们得到了很多钱来做这件事。

显然，有负面影响——当你选择 PaaS 时，经常会出现所见即所得的情况，你使用的现成产品并不总是百分之百适合你的需求。如果它适合百分之九十，你需要判断是否值得那最后的百分之十，或者你宁愿选择一个全功能但可定制的 IaaS 部署。

# Ops 与 DevOps 之间的战争

DevOps 作为一个词被误解、滥用和扭曲，到处都有招聘人员这样做。

去 DevOps 的 subreddit（https://old.reddit.com/r/devops）上，你会看到 DevOps 被称为一个“运动”，这正是它最初的意图。

DevOps 是开发和运维的缩写，它应该是一种将软件开发原则与传统 IT 运维相结合的方法论。

然而，我们生活在现实世界中，虽然你可能会觉得在电话那头对招聘人员大喊“DevOps 不是一个职位！”很有趣（不要这样做），但这并不会减弱变革的力量。

我被称为 DevOps 工程师，我也认识很多其他被称为 DevOps 工程师的人。我也申请过专门招聘“一名 DevOps 人员加入并帮助我们发展”的工作。这取决于使用情况，就像语言世界中的很多东西一样，如果招聘人员说“我想要一个 DevOps”，那么招聘人员就会为之做广告。

这个行业本身正在不断壮大，因为将传统的开发方法与基础设施和应用程序管理相结合的明显优势得到了证实，IaaS 和 PaaS 解决方案的崛起只加速了这一趋势。

简而言之，我认为在 DevOps 环境中工作归结为以下几点：

+   欣赏基础设施即代码

+   重视代码和实践的可重用性

+   评估和采用新技术或实践

许多人对“DevOps”的定义有不同的看法，尽管有些人大声喊叫说这是一个黑白分明的定义，但也许最糟糕的是那些固执的系统管理员和运维工程师，他们根本不相信进步。

我遇到过这样的人 - 那些不相信编码的人，他们似乎喜欢独特的服务器，总是自称为管理员，尽管因此而被忽视而无法得到工作。

了解和欣赏传统系统管理的价值是肯定的，但这些技能与现代方法紧密相连。

# 更像是一场小规模战斗。

过去，每个系统管理员都被期望至少了解一点 SQL，因为他们通常被认为也是基础设施的数据库管理员。现在，人们假定系统管理员至少也精通一些“DevOps”工具。

“DevOps 工具”也有一些模糊的定义，但 Hashicorp 公司的任何东西可能都适用。

事情开始变得棘手的地方在于责任的下放。

有一些公司有以下情况：

+   一个平台团队

+   一个运维团队

+   一个 DevOps 团队

+   一个拥有 DevOps 工程师的开发团队

根据上述列表，你告诉我，谁负责确保 OS 是最新的。

这是平台团队，对吧？这个平台是他们的，所以他们有责任确保环境中的所有 OS 都是最新版本。

但是 DevOps 团队呢？他们编写了配置盒子的代码，我们知道他们的 Ansible 在一开始就运行了“yum update” - 他们应该再次运行相同的 Ansible 吗？

运维呢？过去，他们的责任可能是确保 OS 更改不会影响软件，所以他们应该是更新 OS 的人吧？

开发人员是想要最新 OS 软件包功能的人，他们要求进行更新 - 因此责任是否应该由他们的“DevOps”团队成员承担？

看到关于责任的争论，甚至是关于个人被雇佣的工作标题的争论，这是令人困惑但并不罕见的。

部分原因是公司没有定义严格的结构，但也是那些自称某种方式然后将自己与他们认为与自己的工作无关的事情隔离的人的错。

如果你发现自己在这样的公司里，有不愿意承担责任的敌对派系，可能最好是与你的上级，或上级的上级，甚至是 CTO，讨论一下结构。

我的一个朋友曾经说过（我在这里是引用），“工作标题可能会改变，但归根结底我们还是运维工程师。”他的意思是，虽然新技术和趋势来来去去（我在看你，Node.js！），但当一切都变得一团糟时，我们仍然需要以我们传统的能力来解决问题。

# 总结 - BSD，Solaris，Windows，IaaS 和 PaaS，DevOps

我希望这一章节能够涉及我们作为系统管理员存在的各种补充元素。这些概念中的许多概念都可以单独填满一章，但是我已经被一些人告知这本书太长了，而我甚至还没有完成写作！

希望你对世界上存在的其他操作系统有一些了解，如果你之前不知道的话，甚至你可能会有倾向离开这一章并安装 SmartOS 或 OpenBSD，我会极力鼓励。不要把自己局限在一个领域，谁知道呢？在未来的某个时刻，我们所知道的这个怪物 Linux 可能会消亡，然后其他东西可能会从灰烬中崛起来取而代之。你应该做好准备。

就像 Linux 正在消亡一样，传统的系统管理肯定正在改变，如果我在本章中的讽刺语气暗示了什么，那就是你应该准备跟上变化。IaaS 已经很普遍了，尽管在企业世界，它的市场份额正日益流失给 PaaS 解决方案。学会如何在不触及底层操作系统的情况下部署网站，你将会很受欢迎（至少目前是这样）。

最后，还有 DevOps，我特意试图保持简短，因为我已经认为会有人和我争论定义。我的朋友是对的，我们的工作标题似乎每五年就会改变一次，但最终它总是回到同一个问题上——你知道在哪里找到日志文件，你能弄清楚为什么服务崩溃了吗？

如果你从这一部分中得到了什么，那就是你永远不可能知道关于系统管理世界的所有事情，但你知道的越多，你就能赚得越多...或者你可能会对自己的工作感到更自豪——就像那样一些空洞的东西。
