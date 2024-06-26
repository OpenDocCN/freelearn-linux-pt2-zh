# 第一章：安装 CentOS

在本章中，我们将涵盖：

+   在 Windows 或 OS X 上下载 CentOS 并确认校验和

+   在 Windows 或 OS X 上创建 USB 安装介质

+   使用图形安装程序执行 CentOS 安装

+   通过 HTTP 进行网络安装

+   使用 kickstart 文件安装 CentOS

+   重新安装引导加载程序

+   在救援模式下排除系统故障

+   开始使用并自定义引导加载程序

+   更新安装并使用额外的管理和发展工具增强最小安装

# 引言

本章是一系列涵盖安装 CentOS 7 操作系统基本实践的操作的集合。本章的目的是向您展示您可以多快地让 CentOS 运行起来，同时让您能够使用一些“行业技巧”来定制您的安装。

# 在 Windows 或 OS X 上下载 CentOS 并确认校验和

在本操作中，我们将学习如何使用典型的 Windows 或 OS X 桌面计算机下载并确认一个或多个 CentOS 7 磁盘映像的校验和。CentOS 通过 HTTP、FTP 或 rsync 协议从全球各地的一系列镜像站点或通过 BitTorrent 网络以各种格式提供。对于从互联网下载非常重要的文件，例如操作系统映像，验证这些文件的校验和被认为是最佳实践，以确保任何生成的媒体在安装时都能按预期运行和执行。这也确保了文件是真实的，并且来自原始来源。

## 准备

要完成此操作，假设您正在使用具有完全管理权限的典型 Windows（Windows 7、Windows Vista 或类似）或 OS X 计算机。您需要互联网连接以下载所需的安装文件，并且需要访问具有适当软件的标准 DVD/CD 刻录机，以便从映像文件创建相关的安装磁盘。对于此操作，假设所有下载将存储在 Windows 上的个人`C:\Users\<username>\Downloads`文件夹中，或者如果使用 OS X 系统，则存储在`/Users/<username>/Downloads`文件夹中。

## 如何操作...

无论您下载哪种类型的安装文件，以下技术都可以应用于 CentOS 项目提供的所有映像文件：

1.  让我们首先在网络浏览器中访问[`www.centos.org`](http://www.centos.org)，导航到**立即获取 CentOS**按钮链接。然后点击文本中的**当前镜像列表**链接。

1.  镜像站点被分类，因此从链接列表中选择一个地理位置接近您当前位置的镜像。例如，如果您在伦敦（英国），您可以选择来自**欧盟**和**英国**的镜像。现在通过选择 HTTP 或 FTP 链接来选择一个镜像站点。

1.  做出选择后，现在将看到所有可用 CentOS 版本的目录列表。要继续，只需单击读取`7`的相应文件夹。接下来，您将看到一个额外的目录列表，如`atomic`、`centosplus`、`cloud`等。我们选择`isos`目录继续。

1.  CentOS 7 目前仅支持 64 位架构，因此浏览到唯一可用的标记为`x86_64`的目录，这是一个包含 64 位版本的容器。

1.  现在将为您提供一系列可供下载的文件。首先下载有效的校验和结果副本，标识为`md5sum.txt`。

1.  如果您是 CentOS 新手或打算遵循本书中找到的配方，那么最小安装是理想的。这包含尽可能少的软件包以拥有一个功能系统，因此选择以下内容（`XXXX`是此版本的月份标记）：

    ```
    CentOS-7-x86_64-Minimal-XXXX.iso

    ```

1.  仅在基于 Windows 的系统上（在 Mac 上，此工具已在系统中可用），在浏览器中访问[`mirror.centos.org/centos/dostools/`](http://mirror.centos.org/centos/dostools/)并下载程序`md5sum.exe`。

1.  现在在 Windows 上，打开命令提示符（通常位于**开始** | **所有程序** | **附件** | **命令提示符**）并输入以下命令到打开的窗口中（在所有行的末尾按下*回车*键）：

    ```
    cd downloads
    dir

    ```

1.  在 OS X 上，打开程序**Finder** | **Applications** | **Utilities** | **Terminal**，然后输入以下命令（在所有行的末尾按下*回车*键）：

    ```
    cd ~/Downloads
    ls

    ```

1.  现在您应该看到下载文件夹中的所有文件（包括所有下载的 CentOS 安装图像文件、md5sum.txt 文件以及在 Windows 上的 md5sum.exe 程序）。

1.  根据显示的文件名，修改以下命令以检查您下载的 ISO 图像文件的校验和。在 Windows 上，输入以下命令（根据需要更改`XXXX`月份标记）：

    ```
    md5sum.exe CentOS-7-x86_64-Minimal-XXXX.iso

    ```

1.  在 OS X 上，请使用以下命令：

    ```
    md5 CentOS-7-x86_64-Minimal-XXXX.iso

    ```

1.  按下*回车*键继续，然后等待命令提示符响应。响应被称为 MD5 总和，结果可能看起来像以下内容：

    ```
    d07ab3e615c66a8b2e9a50f4852e6a77  CentOS-7-x86_64-Minimal-1503-01.iso

    ```

1.  现在查看总和，并与`md5sum.txt`文件中与您的特定图像文件相关的列表进行比较（在文本编辑器中打开）。如果两个数字匹配，那么您可以确信您确实下载了一个有效的 CentOS 图像文件。如果不是，您的下载文件可能已损坏，因此请重新启动此过程，再次下载图像文件。

1.  完成后，只需使用您首选的桌面软件将您的图像文件刻录到空白 CD-ROM 或 DVD-ROM 上，或者从中创建 USB 安装介质，我们将在本章的下一个配方中向您展示。

## 它是如何工作的…

那么我们从这次经历中学到了什么？

下载 CentOS 安装镜像只是构建完美服务器的第一步。尽管这个过程非常简单，但许多人忘记了确认校验和的必要性。在本书中，我们将使用最小安装镜像，但您应该知道还有其他安装选项可供您选择，例如 NetInstall、DVD、Everything 和各种 LiveCDs。

# 在 Windows 或 OS X 上创建 USB 安装介质

在本操作中，我们将学习如何在 Windows 或 OS X 上创建 USB 安装介质。如今，越来越多的服务器系统、台式机和笔记本电脑在发货时不带任何光驱。使用 USB 设备安装新操作系统，例如 CentOS Linux，对于它们来说变得至关重要，因为没有其他安装选项可用，因为没有其他方式可以启动安装介质。此外，使用 USB 介质安装 CentOS 可以比使用 CD/DVD 方法快得多。

## 准备工作

在我们开始之前，假设您已经按照前面的操作步骤，下载了最小化的 CentOS 镜像并确认了相关镜像文件的校验和。同时，假设所有下载内容（包括下载的 ISO 文件）都存储在 Windows 上的`C:\Users\<username>\Downloads`文件夹中，或者在使用 OS X 系统时，存储在`/Users/<username>/Downloads`文件夹中。接下来，您需要一个能够被操作系统识别的空闲 USB 设备，该设备有足够的总空间，并且是空的或者其上的数据可以丢弃。准备作为 CentOS 7 最小版本安装介质的 USB 设备所需的总空间大约为 700 兆字节。如果您使用的是 Windows 计算机，您需要一个有效的互联网连接来下载额外的软件。在 OS X 上，您需要一个管理员用户账户。

## 如何操作...

开始此操作前，启动您的 Windows 或 OS X 操作系统，然后连接一个足够容量的空闲 USB 设备，并等待它被 Windows 下的**文件管理器**或 OS X 下的**Finder**识别。

1.  在基于 Windows 的系统上，我们需要下载一个额外的软件叫做`dd`。访问您最喜欢的浏览器中的[`www.chrysocome.net/dd`](http://www.chrysocome.net/dd)。现在下载那里最新的`dd-XX.zip`文件，其中`XX`是最新的稳定版本号。例如，`dd-0.5.zip`。

1.  在 Windows 上，使用**文件管理器**导航到您的`Downloads`文件夹。在这里，您将找到`dd-05.zip`文件。右键点击它并选择**全部提取**，然后提取`dd.exe`文件，不要创建任何子目录。

1.  在 Windows 上，打开命令提示符（通常位于**开始** | **所有程序** | **附件** | **命令提示符**）并输入以下命令：

    ```
    cd downloads
    dd.exe --list

    ```

1.  在 OS X 上，打开**Finder** | **应用程序** | **实用工具** | **终端**程序，然后输入以下命令：

    ```
    cd ~/Downloads
    diskutil list

    ```

1.  在 Windows 系统中，要找到你想要用作安装介质的正确 USB 设备的名称，请查看命令输出中的`removable media`部分。在此之下，你应该会找到一条以`Mounting on`开头的行，后面跟着一个驱动器字母，例如，`\.\e:`。这个用晦涩方式书写的驱动器字母是下一步中最重要的部分，所以请记下来。

1.  在 OS X 系统中，设备路径可以在前一个命令的输出中找到，格式为`/dev/disk<number>`，其中`number`是磁盘的唯一标识符。磁盘编号从零（`0`）开始。磁盘`0`很可能是 OS X 恢复磁盘，而磁盘`1`很可能是你的主 OS X 安装。要识别你的 USB 设备，尝试比较`NAME`、`TYPE`和`SIZE`列与你的 USB 闪存盘的规格。如果你已经确定了设备名称，请记下来，例如，`/dev/disk3`。

1.  在 Windows 系统中，输入以下命令，假设你选作安装介质的 USB 设备在 Windows 中的设备名称为`\\.\e:`（根据需要更改此设置，并小心输入——这可能会造成巨大的数据丢失）。同时，在下一个命令中将`XXXX`替换为正确的`iso`文件版本号：

    ```
    dd.exe if=CentOS-7-x86_64-Minimal-XXXX.iso of=\\.\e: bs=1M

    ```

1.  在 OS X 系统中，你需要两个命令，这两个命令会要求输入管理员密码（将`XXXX`和`disk3`替换为正确的版本号和正确的 USB 设备路径）：

    ```
    sudo diskutil unmountDisk /dev/disk3
    sudo dd if=./CentOS-7-x86_64-Minimal-XXXX.iso of=/dev/disk3 bs=1m

    ```

1.  在`dd`程序完成后，会有一些输出统计信息，显示复制过程耗时以及传输了多少数据。在 OS X 系统中，忽略任何关于磁盘不可读的警告信息。

1.  恭喜！你现在已经创建了你的第一个 CentOS 7 USB 安装介质。现在你可以在 Windows 或 OS X 系统中安全地移除 USB 驱动器，并物理拔出设备，将其用作在目标机器上安装 CentOS 7 的启动设备。

## 它是如何工作的...

那么，我们从这次经历中学到了什么？

本教程的目的是向你介绍使用`dd`命令行程序在 USB 设备上创建 CentOS 安装 ISO 文件的精确副本的概念。`dd`程序是一个基于 Unix 的工具，可用于将位从源复制到目标文件。这意味着源被逐位读取并写入目标，不考虑内容或文件分配；它只是涉及读取和写入纯原始数据。它期望两个基于文件名的参数：输入文件（`if`）和输出文件（`of`）。我们将使用 CentOS 镜像文件作为我们的输入文件名，将其精确地`1:1`克隆到 USB 设备上，该设备可通过其设备文件作为我们的输出文件参数访问。`bs`参数定义了块大小，即一次要复制的数据量。请小心，这是一个绝对的专业工具，在复制数据时会覆盖目标上的任何现有数据，而无需进一步确认或任何安全检查。因此，至少要双重检查目标 USB 设备的驱动器字母，切勿混淆它们！例如，如果你在`D:`上安装了第二个硬盘，在`E:`上安装了 USB 设备（在 OS X 上，分别为`/dev/disk2`和`/dev/disk3`），并且你混淆了驱动器字母`E:`和`D:`（或`/dev/disk3`和`/dev/disk2`），你的第二个硬盘将被擦除，几乎没有恢复任何丢失数据的机会。所以请小心处理！如果你对正确的输出文件设备有疑问，切勿启动`dd`程序！

总之，可以公平地说，对于创建 CentOS 7 的 USB 安装媒体，存在比`dd`命令更为便捷的解决方案，例如 Fedora Live USB Creator。但本教程的目的不仅在于制作一个即用的 CentOS USB 安装器，还在于让你熟悉`dd`命令。这是一个常见的 Linux 命令，每位 CentOS 系统管理员都应该知道如何使用。它可以用于广泛的日常任务，例如安全擦除硬盘、网络速度基准测试或创建随机二进制文件。

# 使用图形安装程序执行 CentOS 安装

在本教程中，我们将学习如何使用 CentOS 7 中引入的新图形安装程序界面执行典型的 CentOS 安装。在很多方面，这被认为是安装系统的推荐方法，因为它不仅允许你创建所需的硬盘分区，还允许你在很多方面自定义安装（例如，键盘布局、软件包选择、安装类型等）。然后，你的安装将构成一个服务器的基础，在这个服务器上，你可以构建、开发和运行任何类型的服务，你将来可能希望提供这些服务。

## 准备就绪

在我们开始之前，假设您已经按照之前的步骤操作，其中您被展示了如何下载 CentOS 镜像，确认相关镜像文件的校验和，并创建相关的安装光盘或 USB 介质。您的系统必须是 64 位（x86_64）架构，至少需要 406MB RAM 来加载图形安装程序（如果安装图形窗口管理器如 Gnome，则建议 1GB 或更多），并且至少有 10GB 的可用硬盘空间。

## 如何操作...

要开始这个步骤，插入您的安装介质（CD/DVD 或 USB 设备），重新启动计算机，并在启动时按下选择启动设备的正确键。然后从列表中选择插入的设备（对于许多计算机，这可以通过*F11*或*F12*访问，但在您的系统上可能不同。请参考您的主板手册）。

1.  在欢迎屏幕上，**测试此媒体并安装 CentOS 7**选项已被预选，我们将使用此选项。准备好后，按*回车*键继续。

1.  加载一些初始文件后，安装程序开始测试安装介质。单个测试应在 30 秒到五分钟之间，并会报告您的安装介质上是否有任何错误。当这个过程完成后，系统最终将加载图形安装程序。

1.  CentOS 安装程序现在将显示图形安装欢迎屏幕。从这一点开始，您可以使用键盘和鼠标（后者强烈推荐），但如果您打算使用数字键盘，请记得在键盘上启用数字锁定。

1.  在左侧，您会看到主要的语言类别，在右侧，安装程序的子语言。您还可以使用左下角的文本框搜索语言。对语言设置的所有更改将立即生效，因此，当您准备好后，选择**继续**按钮以继续。

1.  现在我们到达了主要的安装菜单，称为**安装摘要**。

1.  这里显示的大多数选项已经有一些预设值，无需更改即可使用，而那些没有默认值且需要您注意的选项则标有红色感叹号，例如**系统**类别下的**安装目的地**。因此，让我们使用鼠标点击它。

1.  点击**安装位置**按钮后，你将看到一个图形列表，列出了当前连接到你计算机的所有硬盘设备，你可以用它们来安装操作系统。你可以通过点击正确的硬盘图标来选择你的目标硬盘。然后它会在上面打上勾。如果你不确定正确的硬盘，尝试通过比较菜单中显示的品牌和总大小来识别它。在安装可以进行之前，你必须选择一个硬盘。请小心并明智地选择你的目标硬盘，因为安装过程中它将擦除任何现有数据。准备就绪后，点击**完成**按钮。

1.  如果你的选定硬盘已经包含数据，那么在点击**完成**时，你可能会看到一个可以被描述为警告/错误消息的信息。该消息可能显示：**你没有足够的可用空间来安装 CentOS**。别担心！这是预期中的，消息只是要求你重新初始化你的硬盘，因为 CentOS 只能安装在空白的硬盘上。在大多数情况下，特别是如果你在硬盘上有多个分区，只需点击**回收空间**，这将显示一个新窗口，其中列出了该驱动器上的所有分区。在这里，只需点击**删除所有**，然后再次点击**回收空间**，以丢弃该硬盘上的任何数据，这将完成硬盘初始化的任务，并允许你继续下一步。完成后，点击**完成**按钮。

1.  回到**安装摘要**屏幕，**安装位置**项上的感叹号现在应该消失了。

1.  在**系统**类别下，我们可以选择性地点击**网络与主机名**。在接下来的页面中，左侧你可以选择你希望连接到互联网的主要网络适配器，并通过点击它来选中。对于选中的设备，点击右侧的开关以启用并自动连接，使用开关的**开启**位置。最后，在关闭此子菜单之前，将其文本字段中的主机名更改为适当的名称。点击**完成**。

1.  现在回到**安装摘要**屏幕，所有重要设置都已经完成或获得了预设值，所有感叹号都消失了。如果你对这些设置满意，点击**开始安装**按钮，或者根据需要更改设置。

1.  在下一个屏幕上，你需要为 root 用户创建并确认一个 root 密码，同时新系统在后台进行安装。选择一个不少于六个字符的安全密码。

1.  在此屏幕上，你还可以创建一个标准用户账户，这是非常推荐的。如果你创建了一个新用户，请不要勾选**使该用户成为管理员**。准备就绪后，点击**完成**（如果你输入了一个弱密码，你需要通过点击两次来确认）。

1.  CentOS 现在将在后台对硬盘进行分区和格式化，并解决任何依赖关系，安装程序将开始向硬盘写入数据。这可能需要一些时间，但进度条将指示安装状态。完成后，安装程序将通知您整个过程已完成，并且安装成功。准备好后，点击**重启**按钮。现在从驱动器中取出安装介质。

1.  恭喜！您已成功在计算机上安装了 CentOS 7。

## 它是如何工作的…

在本教程中，您已经了解了如何安装 CentOS 7 操作系统。在介绍了图形安装过程的典型方法之后，您现在可以对服务器进行额外的配置更改和软件包安装，以满足您打算让服务器承担的角色。这个图形安装程序旨在非常直观和灵活，使得安装过程非常简单，因为它将引导用户完成一些强制性任务，这些任务必须在开始安装主系统之前完成。

# 通过 HTTP 进行网络安装

在本教程中，我们将学习如何通过 HTTP（使用 URL 方法）启动网络安装过程，以便安装 CentOS 7。这是一个使用一个小型映像文件来启动计算机，并允许用户通过网络连接选择并安装他想要的软件包和服务的过程，而不安装其他任何东西，从而提供了极大的灵活性。

## 准备就绪

在我们开始之前，假设您已经知道如何下载和校验 CentOS 7 安装映像，以及如何从中创建相关的安装介质。对于本教程，我们需要下载并创建网络安装映像的安装介质（下载最新的 CentOS-7-x86_64-NetInstall-XXXX.iso 文件），而不是本章中另一个教程中显示的最小 ISO。此外，假设您至少已经通过一次图形安装过程，确切知道如何从安装介质启动并使用安装程序。

## 如何操作…

要开始本教程，请插入您准备好的网络安装介质，从它启动计算机，并等待欢迎屏幕出现：

1.  在欢迎屏幕上，**测试此介质并安装 CentOS 7**选项已被预选，我们将使用此选项。准备好后，按*回车*键继续。

1.  测试完成后，图形安装程序将加载并显示典型的图形安装摘要屏幕。

    ### 注意

    在这里，安装程序应该像在正常图形安装教程中一样进行配置，除了对**网络和主机名**以及**安装源**菜单项进行以下强制性更改（由红色感叹号指示）。

1.  在我们能够通过网络安装 CentOS 之前，我们必须确保我们有可用的网络连接。因此，您应该首先点击 **网络和主机名** 菜单项，并将您的网络适配器激活到连接状态。有关更多详细信息，请参阅正常安装食谱。

1.  接下来，点击 **安装源** 进入设置。由于我们将通过 HTTP（也称为 URL 方法）进行安装，因此您应该在 **您想使用哪种安装源？** 部分中保留默认的 **网络** 选项。

1.  现在在标准的 `http://` 文本框中输入以下 URL，我们将使用它来下载所有必需的安装包：[`mirror.centos.org/centos/7/os/x86_64/`](http://%20http://mirror.centos.org/centos/7/os/x86_64/)。

1.  或者，您也可以使用个人仓库，这需要您提前创建（请参阅 第四章，*使用 YUM 管理软件包*）

1.  准备就绪后，点击 **完成** 开始初始化过程。

1.  成功后，安装程序将开始检索适当的 `install.img` 文件。这可能需要几分钟才能完成，但一旦解决，进度条将指示所有下载活动。当此过程成功完成后，**安装源** 处的感叹号将消失，但会出现另一个感叹号，告诉用户缺少 **软件选择**。点击它并选择适合您需求的选项。对于本食谱的目的，只需在 **基本环境** 下选择 **最小安装**，然后点击 **完成**。

1.  如果 **您想使用哪种安装源？** 保持灰色且无法更改，则您的网络适配器存在连接问题。如果出现这种情况，请返回配置 **网络和主机名** 并更改网络设置，直到达到连接状态。

1.  CentOS 7 现在将以常规方式安装操作系统，并在安装过程完成后向您表示祝贺。由于所有软件包都必须从互联网上获取，因此它可能比从物理安装介质安装要慢。

## 工作原理...

本食谱的目的是向您介绍 CentOS 网络安装过程的概念，以展示这种方法可以多么简单。通过完成本食谱，您不仅通过将初始下载限制为仅安装过程所需的文件来节省时间，而且还能够利用完整的图形安装方法，而无需完整的 DVD 套件。

# 使用 kickstart 文件安装 CentOS 7

虽然使用图形安装程序手动安装 CentOS 7 在一台服务器上是可以的，但在多台系统上这样做可能会很繁琐。Kickstart 文件可以自动化服务器系统的安装过程，这里我们将展示如何实现这一点。它们是简单的基于文本的配置文件，提供了关于目标系统应如何设置和安装的详细和精确的指令（例如，安装哪种键盘布局或额外的软件包）。

## 准备就绪

为了成功完成此方法，您需要访问一个已安装的 CentOS 7 系统以获取我们要使用的 kickstart 配置文件并用于自动化安装。在这个预安装的 CentOS 服务器上，您还需要一个有效的互联网连接来下载额外的软件。

接下来，我们需要下载并创建 DVD 或 Everything 映像的安装介质（下载最新的`CentOS-7-x86_64-DVD-XXXX.iso`或`CentOS-7-x86_64-Everything-XXXX.iso`文件），而不是本章中另一方法中显示的最小 iso 文件。然后，您需要另一个在 Linux 系统上可读写的 USB 设备（格式化为 FAT16、FAT32、EXT2、EXT3、EXT4 或 XFS 文件系统）。

## 如何操作...

为了使此方法生效，我们首先需要物理访问另一个已完成 CentOS 7 安装的 kickstart 文件，我们将使用它作为新 CentOS 7 安装的模板。

1.  在现有的 CentOS 7 系统上以 root 身份登录，并通过输入以下命令并按*回车*键执行来确保 kickstart 配置文件存在（这将显示文件的详细信息）：

    ```
    ls -l /root/anaconda-ks.cfg

    ```

1.  接下来，物理上插入一个 USB 设备，然后输入以下命令，这将为您提供当前连接到计算机的所有硬盘设备的列表：

    ```
    fdisk -l

    ```

1.  尝试通过比较其大小、分区以及识别的文件系统与您的 USB 设备的规格来确定设备名称。设备名称将是`/dev/sdX`的形式，其中`X`是一个字母字符，如`b`、`c`、`d`、`e`等。如果您无法使用`fdisk`命令找到正确的设备名称，请尝试以下技巧：运行`fdisk -l`两次 - 首先在未插入 USB 设备的情况下，然后在插入 USB 设备的情况下，比较第二次输出如何变化 - 它比第一次输出多一个设备名称：您感兴趣的设备名称！

1.  如果您在列表中找到了正确的设备名称，请创建一个目录以将其挂载到当前文件系统：

    ```
    mkdir /mnt/kickstart-usb

    ```

1.  接下来，实际上将 USB 设备挂载到此文件夹，假设您选择的 USB 分区位于`/dev/sdc1`（根据需要更改此设置）：

    ```
    mount /dev/sdc1 /mnt/kickstart-usb

    ```

1.  现在，我们将在 USB 设备上创建 kickstart 文件的工作副本以进行定制：

    ```
    cp /root/anaconda-ks.cfg /mnt/kickstart-usb

    ```

1.  然后，使用您喜欢的文本编辑器打开 USB 设备上的复制 kickstart 文件（这里我们将使用编辑器 nano，如果您还没有安装它，请输入`yum install nano`）：

    ```
    nano /mnt/kickstart-usb/anaconda-ks.cfg

    ```

1.  现在，我们将修改文件以在新目标系统上安装 CentOS。在 nano 中，使用上下箭头键转到以（`<your_hostname>`将是您在安装过程中给出的主机名，例如`minimal.home`）开头的行：

    ```
    network  --hostname=<your_hostname>

    ```

1.  现在，编辑`<your_hostname>`字符串，为其赋予一个新的唯一主机名。例如，在任何现有名称的末尾添加一个`-2`，如下所示：

    ```
    network  --hostname=minimal-2.home

    ```

1.  接下来，使用上下箭头键将光标向下移动，直到它停在写着`%packages.`的行上。在其下方附加以下行（您可以进一步自定义此设置，并提供您希望自动安装的其他软件包）：

    ```
    mariadb-server
    httpd
    rsync
    net-tools

    ```

1.  现在保存并关闭文件，在 nano 编辑器中，使用*Ctrl*+*o*键组合（这意味着，按住键盘上的*Ctrl*键，然后按*o*键，不要松开*Ctrl*键）来写入更改。然后按*Return*确认文件名，并按*Ctrl*+*x*退出编辑器。

1.  接下来，安装以下 CentOS 软件包：

    ```
    yum install system-config-kickstart

    ```

1.  现在，我们使用`ksvalidator`程序验证 kickstart 文件的语法，该程序包含在我们刚刚安装的软件包中：

    ```
    ksvalidator /mnt/kickstart-usb/anaconda-ks.cfg

    ```

1.  如果`config`文件没有错误，现在使用以下命令卸载 USB 闪存驱动器：

    ```
    cd
    umount /mnt/kickstart-usb

    ```

1.  当您再次获得新的命令提示符时，从系统中物理拔出包含 kickstart 文件的 USB 设备，以便在目标机器上使用。

1.  现在，您需要物理访问您想要安装 CentOS 的目标机器，使用刚刚创建的 kickstart 文件。断开任何其他不需要在安装过程中使用的外部文件存储设备。

1.  开启计算机并插入准备好的 CentOS 安装介质（必须是 CentOS DVD 或 Everything 安装盘镜像，准备在 CD/DVD 光盘或 USB 设备安装器上）。同时，将包含在先前步骤中创建的 kickstart 文件的 USB 闪存驱动器连接到计算机（如果使用 USB 驱动器安装 CentOS，则总共需要两个空闲的 USB 端口来完成此操作）。

1.  接下来，启动服务器并在初始启动屏幕期间按下与引导您刚刚连接的 CentOS 安装介质相关的正确键。

1.  在 CentOS 安装程序开始加载后，将显示常见的标准 CentOS 7 安装欢迎屏幕，并且**测试此媒体并安装 CentOS 7**选项将由光标预先选中。

1.  接下来，在键盘上按一次*Esc*键，切换到`boot: prompt`。

1.  现在我们准备好开始 kickstart 安装了。为此，您需要知道 USB 设备上 kickstart 文件所在的确切分区名称。输入以下命令，假设您的分区位于`/dev/sdc1`（根据需要更改此设置），然后按*Return*键开始 kickstart 安装过程：

    ```
    linux ks=hd:sdc1:/anaconda-ks.cfg

    ```

    ### 注意

    如果你无法找到 USB 闪存盘的正确设备和分区名称，你需要在救援模式下启动目标系统（参考*在救援模式下排除系统故障*的解决方案），通过比较其大小、分区以及识别的文件系统与你的闪存盘规格来确定正确的设备名称和分区号。

1.  现在，新系统将自动使用提供的 kickstart 文件中的指令进行安装。你可以查看安装输出消息，因为它会向用户显示详细的安装进度。

1.  如果系统安装完成，重启系统并登录到你的新机器，以验证新系统是否按照我们使用 kickstart 文件描述的方式进行了设置。

## 它是如何工作的...

在这个解决方案中，你已经看到，每个运行 CentOS 7 安装的服务器都在其根目录中保留了 kickstart 文件，该文件包含了系统安装过程中设置的详细信息。kickstart 文件可以用来自动化多个具有相同配置的系统的安装。这可以节省大量时间，因为安装过程中不需要用户交互。此外，如果目标机器的 RAM 不符合图形化安装的最低要求，但需要文本模式安装器不提供的其他功能（如自定义分区），我们也可以使用这种方法。kickstart 配置文件是简单的纯文本文件，可以手动从头创建。由于使用 kickstart 语法构建系统有相当多的不同命令，我们使用了一个现有的文件作为模板，并对其进行了定制以满足我们的需求，而不是完全从头开始。我们没有使用最小化安装镜像来驱动我们的 kickstart 安装，因为我们安装了一些额外的软件包，这些软件包不包含在最小化 ISO 文件中，例如 Apache Web 服务器。

# 开始和自定义引导加载程序

当你打开计算机时，引导加载程序是第一个启动的程序，负责加载并将控制权转移到下层的操作系统。如今，几乎任何现代 Linux 发行版都使用**GRand Unified Bootloader 版本 2**（**GRUB2**）来启动系统。它在配置上非常灵活，并支持许多不同的操作系统。在这个解决方案中，我们将展示如何通过禁用菜单显示的等待时间来定制 GRUB2 引导加载程序，从而提高系统启动时间。

## 准备工作

要完成这个解决方案，你需要访问一个已经安装了 CentOS 7 操作系统（最小化安装或其他类型的 CentOS 7 安装都可以）的系统，并具有 root 权限。此外，你还需要具备使用文本编辑器（如 nano）修改配置文件的基本经验。

## 如何操作...

我们通过使用我们选择的文本编辑器打开主 GRUB2 配置文件并对其进行修改，开始这个食谱。

1.  首先以 root 身份登录到你的系统，并为 GRUB2 配置文件创建一个备份副本，以备需要时回滚。按下*返回*键完成：

    ```
    cp /etc/default/grub /etc/default/grub.BAK

    ```

1.  使用以下命令打开我们要编辑的主 GRUB2 配置文件，并按下*返回*键（这里我们将使用编辑器 nano，如果你还没有安装它，请输入`yum install nano`）：

    ```
    nano /etc/default/grub

    ```

1.  在光标所在的第一行按下*返回*键插入新行，然后插入以下行：

    ```
    GRUB_HIDDEN_TIMEOUT=0

    ```

1.  在以下行的开头添加一个`#`符号，如下所示：

    ```
    GRUB_TIMEOUT=0

    ```

1.  现在在 nano 中使用*Ctrl*+*o*保存文件（并按*返回*确认保存的文件名）。使用*Ctrl*+*x*退出编辑器，然后运行以下命令：

    ```
    dmesg | grep -Fq "EFI v"

    ```

1.  如果前面的命令没有产生任何输出，运行以下命令：

    ```
    grub2-mkconfig -o /boot/grub2/grub.cfg

    ```

1.  否则，如果有输出，运行：

    ```
    grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg

    ```

1.  如果`grub2-mkconfig`成功，它将打印`Done.`。现在使用以下命令重新启动你的系统：

    ```
    reboot

    ```

1.  在重启过程中，你会发现 GRUB2 引导菜单不再出现，系统将更快地启动。

## 它是如何工作的...

完成这个食谱后，我们现在知道如何自定义 GRUB2 引导加载程序。在这个非常简单的食谱中，我们只向你展示了引导加载程序的一些基本修改，但它可以做更多！它支持广泛的文件系统，并且可以启动几乎任何兼容的操作系统。如果你计划在同一台机器上运行多个操作系统，这也是特别有用的。要了解更多关于 GRUB2 配置文件语法的信息，请输入`info grub2` | `less`命令，并转到`6.1 简单配置处理`部分（阅读第二章，*系统配置*中的食谱*使用 less 浏览文本文件*，了解如何浏览此文档）。

# 在救援模式下排除系统故障

我们都会犯错，这对新手 Linux 系统管理员尤其如此。Linux 的学习曲线可能很陡峭，迟早会在你的职业生涯中遇到 CentOS 安装由于各种原因无法启动的情况，包括硬件问题或人为错误，如配置错误。如果这种情况发生在你身上，那么你可以使用 CentOS 救援模式来启动一个无法启动的系统，并尝试撤消你的错误或找出问题的根源。在本食谱中，我们将向你展示三个常见的情况，何时使用此选项：

+   访问文件系统以恢复重要数据或撤消对配置文件的更改，如果 CentOS 无法启动

+   更改 root 密码，如果你忘记了它

+   重新安装引导加载程序，这可能在同一硬盘上安装另一个操作系统时损坏 CentOS

## 准备就绪

要完成此操作，您需要一个标准的 CentOS 7 操作系统安装媒体（CD/DVD 或 USB 设备）。为了从系统中恢复数据，您需要将某种外部存储设备连接到系统，例如外部硬盘或一个正常工作的网络连接到另一台计算机，以便将所有珍贵数据复制到不同位置。

## 如何操作...

要开始此操作，您应该从 CentOS 安装 CD/DVD 或 USB 设备启动服务器，并等待直到第一个欢迎屏幕出现，光标等待在**测试此媒体并安装 CentOS 7**菜单选项上。

### 进入救援模式

1.  从主菜单中，使用向下箭头键选择**故障排除**，然后按*回车*键继续。

1.  在**故障排除**屏幕上，使用向下箭头键高亮显示**救援 CentOS 系统**。准备就绪后，按*回车*键继续。

1.  经过一些加载时间后，我们进入救援屏幕，其中包括各种确认子屏幕。要开始此部分，使用左右箭头键选择**继续**，然后按*回车*键继续。

1.  在第一个子屏幕上，选择**确定**并按*回车*键继续。

1.  同样，在接下来的子屏幕中，选择**确定**并按*回车*键继续。

1.  在下一个屏幕上，选择**启动**shell，并使用*Tab*键，高亮显示**确定**，然后按*回车*键继续。

1.  完成上述步骤后，将启动一个 shell 会话。您将在显示屏底部注意到这一点。shell 会话的当前状态将显示如下：

    ```
    bash-4.2#_

    ```

1.  在提示符下，输入以下指令以更改根文件系统，然后在按*回车*键完成您的请求之前：

    ```
    chroot /mnt/sysimage

    ```

1.  恭喜！您已进入救援模式。在任何时候退出，只需输入以下命令，然后按*回车*键以完成您的请求（现在不要这样做，因为这将重启系统）：

    ```
    reboot

    ```

1.  在基本救援模式达到后，我们根据问题的类型有以下选项。

### 访问文件系统

如果您现在处于救援模式并需要从文件系统备份重要文件，您需要一个数据传输的目的地位置。为了将我们想要从服务器恢复的数据传输到另一台计算机，请将外部 USB 设备物理连接到它。您也可以使用网络存储进行恢复。例如，您可以导入 NFS 服务器共享并将数据复制到它。请参阅第七章，*构建网络*中的*使用 NFS*操作。

1.  在救援模式命令行中，输入以下命令，该命令将显示系统上所有当前连接的分区，然后按*回车*键完成您的请求：

    ```
    fdisk -l

    ```

1.  现在，您需要找出连接设备的正确设备名称和分区号；比较各个设备的总大小或文件系统输出与您的 USB 设备的规格可以帮助您完成此过程。您还可以尝试以下技巧：运行`fdisk -l`命令两次，第一次插入 USB 设备，然后再次拔出 USB 设备，并比较两个命令的输出。应该会有一个设备名称不同，这就是您正在寻找的！

1.  如果在列表中找到了正确的设备名称，请创建一个目录以将 USB 设备挂载到文件系统：

    ```
    mkdir /mnt/hdd-recovery

    ```

1.  接下来，将磁盘分区挂载到此文件夹。这里我们假设感兴趣的 USB 设备的设备名称为`sdd1`（如果您的系统不同，请更改）：

    ```
    mount /dev/sdd1 /mnt/hdd-recovery

    ```

1.  救援系统自动将原始系统的硬盘的根分区挂载在特定文件夹下（位于`/mnt/sysimage`下），如果需要访问它，例如更改导致启动问题的配置文件或进行完整或部分备份。例如，如果需要备份 Apache Web 服务器配置文件，请使用：

    ```
    cp -r /mnt/sysimage/etc/http /mnt/hdd-recovery

    ```

1.  如果需要访问当前挂载的根分区以外的分区上的数据，请使用`fdisk -l`来识别感兴趣的分区。然后创建一个目录，将分区挂载到该目录，并切换到该目录以访问数据，就像挂载 USB 设备时所做的那样。

1.  要完成备份文件，请输入：

    ```
    reboot

    ```

### 访问文件系统

1.  如果在救援模式下更改 root 密码，只需使用以下命令并提供新密码：

    ```
    passwd

    ```

1.  要完成更改密码，请输入：

    ```
    reboot

    ```

### 重新安装 CentOS 引导加载程序

1.  现在，我们将使用`fdisk`命令来查找所有当前分区的名称。为此，请输入以下指令，然后按*回车*键以完成请求：

    ```
    fdisk –l

    ```

1.  现在运行以下命令：

    ```
    dmesg | grep -Fq "EFI v"

    ```

1.  如果前面的命令没有产生任何输出，请在`fdisk`列表的引导列中查找`*`符号以找到正确的起始分区，并假设您的引导磁盘位于`/dev/sda1`（根据需要更改此设置），请输入以下内容：

    ```
    grub2-install /dev/sda

    ```

1.  否则，如果有输出，请运行以下命令：

    ```
    yum reinstall grub2-efi shim

    ```

1.  如果没有错误报告，控制台应响应如下：

    ```
    # this device map was generated by anaconda
    (hd0) /dev/sda

    ```

1.  最后一步的控制台输出确认 GRUB 已成功恢复。

1.  要重新启动计算机，请输入：

    ```
    reboot

    ```

## 它是如何工作的...

救援模式环境提供的工具可以解决各种广泛的问题。这些问题通常涉及启动问题，但也可能来自其他类型，例如忘记 root 密码。救援模式可以救急，掌握它是一项非常重要的技能。因此，我们认为应该将这样的操作指南放在手边。

### 提示

请记住，在处理引导加载程序命令时要小心，因为不当使用可能会导致操作系统无法启动。

# 更新安装并增强最小化安装，添加额外的管理和开发工具

在本食谱中，我们将学习如何通过添加额外的工具来增强最小化安装，这些工具将为你提供各种管理和开发选项，反过来，在你的服务器生命周期中证明是至关重要的，并且对于本书中的一些食谱来说是必不可少的。最小化安装可能是最有效的服务器安装方式，但即便如此，最小化安装确实需要一些额外的功能，以使其成为一个更具吸引力的模型。

## 准备就绪

要完成这个食谱，你需要一个具有 root 权限的 CentOS 7 操作系统的最小化安装，并且需要连接到互联网，以便下载额外的包。

## 如何做到这一点...

我们将从这个食谱开始更新系统。

1.  要更新系统，请以 root 身份登录并输入：

    ```
    yum -y update

    ```

1.  CentOS 现在将搜索相关的更新，如果可用，它们将被安装。完成后，根据更新了什么（即内核和新安全功能等），你可以决定重新启动计算机。要这样做，请输入：

    ```
    reboot

    ```

1.  你的服务器现在将重新启动并返回到登录屏幕。我们现在将完成这个食谱，并通过一系列包组来增强我们当前的安装，这些包组在未来将证明非常有用。要这样做，请以 root 身份登录并输入：

    ```
    yum -y groupinstall "Base" "Development Libraries" "Development Tools"
    yum -y install policycoreutils-python

    ```

## 它是如何工作的...

本食谱的目的是增强 CentOS 7 操作系统的最小化安装，通过这样做，你不仅介绍了自己给**Yellowdog Updater Modified**（**YUM**）包管理器（我们将在本书后面再次回到这一点），而且你现在拥有了一个能够运行大量应用程序的系统，开箱即用。

那么我们从这次经历中学到了什么？

我们首先通过更新系统来开始这个配置过程，以确保系统是最新的。在这个阶段，通常重启系统是个好主意。虽然我们不期望经常这样做，但在安装操作系统后首次更新时是必要的，因为很可能有重大的更新可用。这样做的原因通常是基于希望利用新的内核或更新的安全补丁。在下一阶段，配置过程展示了如何添加一系列可能在将来证明非常有用的软件包组。为了节省时间，我们将安装三个主要软件包组的指令打包在一起：`Base`、`Development Libraries` 和 `Development Tools`。仅此一项操作就安装了超过 200 个单独的软件包，从而使您的服务器能够编译代码并运行大量即装即用的应用程序，这些应用程序可能在服务器的整个生命周期内都需要。要查看某个组中所有软件包的列表，例如 `Base` 组，可以运行 `yum groupinfo Base` 命令。我们还安装了 `policycoreutils-python` 软件包，它提供了管理 Linux 安全增强访问控制的工具和程序，我们将在本书的各个章节中频繁使用这些工具。
