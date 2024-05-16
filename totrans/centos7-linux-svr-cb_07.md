# 第七章。构建网络

在本章中，我们将涵盖以下主题：

+   使用 CUPS 打印

+   运行 DHCP 服务器

+   使用 WebDAV 进行文件共享

+   安装和配置 NFS

+   使用 NFS

+   使用 Samba 安全共享资源

# 引言

本章是一系列食谱的集合，涵盖了当今工作环境的许多方面。从跨不同类型的办公计算机系统的打印和文件共享到保持您的计算机在线，本章提供了关于如何快速使用 CentOS 来实施必要的工具，这些工具将在您的网络环境中最大化效率的必要细节。

# 使用 CUPS 打印

**打印服务器**允许本地打印设备连接到网络并被多个用户和部门共享。使用这样的系统有很多优点，包括不需要为每个用户、房间或部门购买专用的打印机硬件。**通用 Unix 打印系统**（**CUPS**）是 Linux 以及包括 OS X 在内的 Unix 发行版上打印服务器的事实标准。它采用典型的客户端/服务器架构，网络中的客户端将打印作业发送到中央打印服务器，该服务器安排这些任务，然后将实际的打印任务委派给本地连接到我们的打印服务器的打印机，或者将打印作业远程发送到具有请求打印机的物理连接的计算机，或者发送到独立的网络打印机。如果您在 CUPS 系统中设置打印机，几乎所有 Linux 和 OS X 打印应用程序在您网络中的任何客户端上都将自动配置为开箱即用，无需安装额外的驱动程序。在这里，在本食谱中，我们将向您展示如何开始使用 CUPS 打印服务器系统。

## 准备

要完成这个食谱，您将需要一个具有 root 权限的工作 CentOS 7 操作系统安装，您选择的基于控制台的文本编辑器，以及连接到互联网以下载额外的软件包。在这个食谱中，我们将使用具有 IP 地址`192.168.1.8`的网络接口，以及相应的网络地址`192.168.1.0/24`，将 CUPS 打印服务器提供给我们的网络。

## 怎么做...

我们从这个食谱开始安装 CUPS 打印服务器软件，这在新鲜的 CentOS 7 最小系统上默认不可用：

1.  为此，以`root`身份登录并安装以下软件包：

    ```
    yum install cups

    ```

1.  接下来，为 CUPS 服务器创建一个 SSL 证书，我们将需要它来进行安全的身份验证到 CUPS Web 应用程序（在询问时添加一个安全密码）：

    ```
    cd /etc/pki/tls/certs
    make cups-server.key

    ```

1.  现在，让我们打开 CUPS 主配置文件以自定义服务器（首先备份）：

    ```
    cp /etc/cups/cupsd.conf /etc/cups/cupsd.conf.BAK
    vi /etc/cups/cupsd.conf

    ```

1.  首先，为了让 CUPS 在整个网络上可用，找到以下行：`Listen localhost:631`，然后将其更改为：

    ```
    Listen 631
    ```

1.  接下来，我们想要配置对基于 Web 的 CUPS 前端所有常规网页的访问。搜索`<Location />`指令（不要与其他指令如`<Location /admin>`混淆），并通过添加您的网络地址来更改整个块。更改后，整个块看起来像这样：

    ```
    <Location />
     Order allow,deny
     Allow 192.168.1.0/24
    </Location>
    ```

1.  接下来，为`/admin`和`/admin/conf Location`指令设置访问权限，仅授予本地服务器的访问权限：

    ```
    <Location /admin>
       Order allow,deny
       Allow localhost
    </Location>
    <Location /admin/conf>
       AuthType Default
       Require user @SYSTEM
       Order allow,deny
       Allow localhost
    </Location>
    ```

1.  最后，将我们的 SSL 证书信息添加到配置文件的末尾：

    ```
    ServerCertificate /etc/pki/tls/certs/cups-server.crt
    ServerKey /etc/pki/tls/certs/cups-server.key
    ```

1.  关闭并保存文件，然后重新启动 CUPS 服务器并在启动时启用它：

    ```
    systemctl restart cups.service systemctl enable cups.service

    ```

1.  现在，我们必须在 firewalld 中打开 CUPS 服务器端口，以便网络中的其他计算机可以连接到它：

    ```
    firewall-cmd --permanent --add-service=ipp firewall-cmd --reload

    ```

1.  您可以通过从`192.168.1.0/24`网络中的另一台计算机浏览以下位置来测试您的 CUPS 服务器的可访问性（在浏览器询问时允许安全异常）：

    ```
    https://<IP address of your CUPS server>:631
    ```

1.  要访问 CUPS 前端内的管理区域，您需要位于运行 CUPS 的服务器上（在 CentOS 7 最小安装上，请安装窗口管理器和浏览器），然后使用系统用户`root`和适当的密码登录。

## 它是如何工作的...

在本配方中，我们向您展示了安装和设置 CUPS 打印服务器是多么容易。

那么，我们从这次经历中学到了什么？

我们的旅程始于在我们的服务器上安装 CUPS 服务器软件包，因为默认情况下 CentOS 7 系统上不可用。之后，我们生成了一个 SSL 密钥对，我们将在稍后的过程中需要它（了解更多信息，请阅读第六章中的*生成自签名证书*配方，*提供安全性*）。它用于允许通过安全的 HTTPS 连接加密提交您的登录凭据到 CUPS 管理 Web 前端。接下来，我们使用我们选择的文本编辑器打开了 CUPS 的主配置文件`/etc/cups/cupsd.conf`。如您所见，配置格式与 Apache 配置文件格式非常相似。我们开始通过删除本地主机名来更改`Listen`地址，从而允许您网络中的所有客户端（`192.168.1.0/24`）访问我们的 CUPS 服务器端口`631`，而不是仅允许本地接口连接到打印服务器。

### 注意

默认情况下，CUPS 服务器启用了`浏览功能`，它会每 30 秒向同一子网上的所有客户端计算机广播系统中所有共享打印机的更新列表。如果还想向其他子网广播，请使用`BrowseRelay`指令。

接下来，我们配置了对 CUPS Web 界面的访问。这个前端可以用来方便地浏览网络上所有可用的打印机，或者如果您使用管理员账户登录，甚至可以安装新打印机或配置它们。由于用户界面中有不同的任务，因此可以使用三个不同的指令来精细调整其访问权限。可以使用`<Location />`指令设置对所有正常网页的访问，而所有管理页面可以使用`<Location /admin>`管理，更具体地更改配置可以使用`<Location /admin/conf>`标签。在这些`Location`标签中的每一个中，我们都添加了不同的`Allow`指令，从而允许从您的完整网络（例如，`192.168.1.0/24`）访问正常的 CUPS 网页（例如，浏览所有可用的网络打印机），而访问特殊管理页面则限制为运行 CUPS 服务的`localhost`服务器。请记住，如果这对您的环境来说太严格，您可以随时调整这些`Allow`设置。此外，还有各种其他类型的`Location`可用，例如用于在其他子网中激活我们的服务的类型。请使用`man cupsd.conf`阅读 CUPS 配置手册。接下来，我们配置了 SSL 加密，从而为 Web 界面激活了安全的`https://`地址。然后，我们首次启动了 CUPS 服务器，并启用了它在服务器启动时自动启动。最后，我们添加了`ipp` firewalld 服务，从而允许 CUPS 客户端连接到服务器。

## 不仅如此...

既然我们已经成功设置并配置了 CUPS 服务器，那么是时候向其添加一些打印机并打印测试页了。在这里，我们将向您展示如何使用命令行将*两种不同*类型的打印机添加到系统中。

### 注意

使用基于图形的 Web 界面 CUPS，也可以添加或配置打印机。

首先，我们将安装一个真正的*网络*打印机，该打印机已经在我们 CUPS 服务器所在的同一网络（在我们的例子中，是`192.168.1.0/24`网络）中可用，然后是一个本地连接的打印机（例如，通过 USB 连接到我们的 CUPS 服务器或同一网络中的任何其他计算机）。

### 注意

为什么您应该将已连接的网络打印机安装到我们的 CUPS 服务器上？CUPS 不仅仅可以打印：它是一个集中式打印服务器，因此可以管理打印机及其作业的调度和队列，为不同子网中的打印机提供服务，并为任何 Linux 或 Mac 客户端提供统一的打印协议和标准，以便方便访问。

### 如何将网络打印机添加到 CUPS 服务器

要开始将网络打印机添加到我们的 CUPS 服务器，我们将使用`lpinfo -v`命令列出 CUPS 服务器已知的所有可用打印设备或驱动程序。通常情况下，CUPS 服务器会自动识别所有本地（USB、并行、串行等）和远程可用（网络协议，如`socket`、`http`、`ipp`、`lpd`等）的打印机，而不会出现任何问题。在我们的示例中，以下网络打印机已成功识别（输出已被截断）：

```
network dnssd://Photosmart%20C5100%20series%20%5BF8B652%5D._pdl-datastream._tcp.local/

```

接下来，我们将把这个打印机安装到 CUPS 服务器上，使其处于其控制之下。首先，我们需要找到正确的打印机驱动程序。正如我们在最后输出中看到的，它是一台 HP Photosmart C5100 系列打印机。因此，让我们在 CUPS 服务器上所有当前安装的驱动程序列表中搜索该驱动程序：

```
lpinfo --make-and-model HP -m | grep Photosmart

```

列表中不包含我们的型号 C5100，因此我们必须使用以下命令安装额外的 HP 驱动程序包：

```
yum install hplip

```

现在，如果我们再次发出命令，我们就能找到正确的驱动程序：

```
lpinfo --make-and-model HP -m | grep Photosmart | grep c5100

```

### 注意

对于其他打印机型号和制造商，也有其他可用的驱动程序包，例如，`gutenprint-cups` RPM 包。

这个打印机的正确驱动程序将显示如下：

```
drv:///hp/hpcups.drv/hp-photosmart_c5100_series.ppd

```

现在，我们准备好使用以下语法安装打印机：

```
lpadmin -p <printer-name> -v <device-uri> -m <model> -L <location> -E

```

在我们的示例中，我们使用以下命令安装了它：

```
lpadmin -p hp-photosmart -v "dnssd://Photosmart%20C5100%20series%20%5BF8B652%5D._pdl-datastream._tcp.local/" -m "drv:///hp/hpcups.drv/hp-photosmart_c5100_series.ppd" -L room123 -E

```

现在，打印机应该处于我们 CUPS 服务器的控制之下，并且应该立即在整个网络中共享并被任何 Linux 或 OS X 计算机看到（在 CentOS 7 最小客户端上，你还需要首先安装`cups`包，并使用 firewalld 的`ipp-client`服务启用传入的`ipp`连接，然后我们 CUPS 服务器共享的网络打印机信息才会变得可用）。

你可以稍后通过打开并更改位于`/etc/cups/printers.conf`的文件来更改此打印机的配置。要实际打印测试页，你现在应该能够使用其名称`hp-photosmart`从任何客户端访问打印机（在 CentOS 7 最小客户端上，你需要安装`cups-client`包）：

```
echo "Hello printing world" | lpr -P hp-photosmart  -H 192.168.1.8:631

```

### 如何将本地打印机共享到 CUPS 服务器

如果你想将物理连接到我们 CUPS 服务器的本地打印机共享出去，只需将打印机插入系统（例如，通过 USB），然后按照之前的步骤操作，即*如何将网络打印机添加到 CUPS 服务器*。在步骤`lpinfo -v`中，你应该看到它以`usb://`地址出现，因此你需要获取这个地址并遵循剩余的步骤。

如果您想连接并共享位于 CUPS 网络中任何其他计算机上的中央 CUPS 服务器上的打印机，请在该其他计算机上安装`cups`守护程序（遵循主节中的所有步骤），然后按照本节中的说明安装打印机驱动程序。这将确保本地 CUPS 守护程序将在网络上提供打印机，就像在我们的中央 CUPS 服务器上一样。现在它已在网络上可用，您可以轻松地将其添加到我们的主 CUPS 服务器，以享受中央打印服务器带来的所有好处。

在本节中，我们仅触及了设置 CUPS 服务器的基础知识，并为您介绍了基础知识。总有更多要学习的内容，您可以构建非常复杂的 CUPS 服务器系统，在企业环境中管理数百台打印机，这超出了本节的范围。

# 运行 DHCP 服务器

如果需要建立网络连接，每台计算机都需要在其系统上安装正确的**互联网协议**（**IP**）配置才能进行通信。使用**动态主机控制协议**（**DHCP**）从*中央点*自动分配 IP 客户端配置可以使管理员的工作更轻松，并且与在网络中的每台计算机系统上手动设置静态 IP 信息相比，简化添加新机器到网络的过程。在小型家庭网络中，人们通常直接在互联网路由器上安装内置的 DHCP 服务器，但这些设备通常缺乏高级功能，并且只有一套基本的配置选项。大多数情况下，这对于大型网络或企业环境来说是不够的，在这些环境中，您更有可能找到专用的 DCHP 服务器，以应对更复杂的场景和更好的控制。在本节中，我们将向您展示如何在 CentOS 7 系统上安装和配置 DHCP 服务器。

## 准备工作

要完成本节，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，您选择的基于控制台的文本编辑器以及互联网连接，以便下载额外的软件包。预计您的 DHCP 服务器将使用静态 IP 地址；如果您没有静态 IP 地址，请参阅第二章中的“构建静态网络连接”节。如果您还计划通过 DHCP 向客户端发送 DNS 信息，则应已应用第八章中的“安装和配置简单名称服务器”节。

## 如何操作...

在本例中，我们将为静态网络接口配置 DHCP 服务器，该接口为单个网络提供所有可用的 IP 地址，所有直接连接到它的计算机（它们都在同一子网中）。

1.  首先，以`root`身份登录，并输入以下命令以安装 DHCP 服务器软件包：

    ```
    yum install dhcp

    ```

1.  在我们的示例中，我们将使用名为`ifcfg-enp5s0f1`的网络接口来处理我们的 DHCP 请求。接下来，我们需要收集一些非常重要的网络信息，我们将在稍后配置 DHCP 服务器时使用（将网络接口名称更改为适合您自己的需求）：

    ```
    cat /etc/sysconfig/network-scripts/ifcfg-enp5s0f1

    ```

1.  从这次输出中，我们需要以下信息，请记下来（很可能，您的输出会有所不同）：

    ```
    BOOTPROTO="static"
    IPADDR="192.168.1.8"
    NETMASK="255.255.255.0"
    GATEWAY="192.168.1.254"

    ```

1.  我们还需要子网网络地址，这可以通过以下行计算得出：

    ```
    ipcalc -n 192.168.1.8/24

    ```

1.  这将打印以下输出（请记下来以备后用）：

    ```
    NETWORK=192.168.1.0

    ```

1.  现在，我们将打开主 DHCP 配置文件，在此之前，我们需要对原始文件进行备份：

    ```
    cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.BAK
    vi /etc/dhcp/dhcpd.conf

    ```

1.  将以下行附加到文件末尾，考虑到您在前面的步骤中从个人网络接口配置中获得的配置（`routers = GATEWAY`，`subnet = NETWORK`）：

    ```
    authoriative;
    default-lease-time 28800;
    max-lease-time 86400;
    shared-network MyNetwork {
        option domain-name           "example.com";
        option domain-name-servers      8.8.8.8, 8.8.4.4;
        option routers                  192.168.1.254;
        subnet 192.168.1.0 netmask 255.255.255.0 {
            range 192.168.1.10 192.168.1.160;
        }
    }
    ```

1.  最后，启动并启用 DHCP 服务：

    ```
    systemctl start dhcpd
    systemctl enable dhcpd

    ```

## 它是如何工作的...

在本食谱中，我们向您展示了为单个网络设置 DHCP 服务器是多么容易。有了这个，每当有新机器加入网络时，计算机都会自动获得正确的 IP 信息，这是它连接到网络所必需的，无需任何进一步的人工干预。

那么，我们从这次经历中学到了什么？

我们通过安装 DHCP 服务器软件包开始了这个配方，因为它不是随 CentOS 7 一起提供的。由于我们的 DHCP 守护进程通过网络接口与其客户端通信以分配 IP 信息，因此下一步我们必须选择将用于该服务的网络设备。在我们的示例中，我们选择了名为`enp5s0f1`的设备。默认情况下，DHCP 服务器可以管理与关联网络接口相同的子网中的所有可用 IP 地址。请记住，主 DHCP 服务器的网络接口必须配置为静态获取其自己的 IP 信息，而不是通过（另一个）DHCP 服务器！接下来，我们使用`cat`命令打印出我们`enp5s0f1`网络接口配置文件中所有有趣的行，这些行我们将需要用于配置 DHCP 服务器。之后，我们使用`ipcalc`工具计算我们 DHCP 服务器的网络接口的（子网）网络地址。然后，我们打开了主 DHCP 服务器配置，开始配置一些*全局*设置，并定义了一个新的*共享网络*。在全局设置中，我们首先将我们的 DHCP 服务器设置为`authoriative`，这意味着它是网络中唯一且主要的负责 DHCP 服务器。接下来，我们将`default-lease-time`定义为`28800`秒，即八小时，将`max-lease-time`定义为`86400`，即 24 小时。租约时间是 DHCP 服务器将 IP 地址“出租”给客户端的时间，之后客户端必须再次向 DHCP 服务器注册以请求 IP 租约的延期。如果它没有在那时请求现有租约的续订，IP 地址将从客户端释放并放回空闲 IP 地址池中，准备提供给想要连接到网络的新机器。客户端可以自行定义它想要租用 IP 地址的时间。如果客户端没有向 DHCP 服务器提供时间范围，将使用默认租约时间。

所有共享同一物理网络接口的子网都应在`shared-network`声明中定义，因此我们使用方括号定义了这个区域。这也称为作用域。在我们的示例中，我们只有一个网络，因此只需要一个共享网络作用域。在其中，我们首先定义了一个`domain-name`选项，该选项将被发送并可被客户端用作其基础域名。接下来，我们将**域名服务器**（**DNS**）添加到我们的配置中。向客户端发送 DNS 信息不是 DHCP 服务器的强制要求，但可能很有用。客户端为给定网络获得的信息越多，越好，因为需要的手动配置步骤就越少。

### 注意

您可以通过 DHCP 向客户端发送有关其连接到的网络的大量其他有用信息：网关、时间、WINS 等。

在我们的示例中，我们使用了官方的 Google DNS 服务器；如果您已经设置了自己的 DNS 服务器（请参阅第八章，*使用 FTP*），您也可以在这里使用这些地址。接下来，我们指定了`routers`选项，这是另一个有用的信息，也将发送给客户端。之后，我们指定了任何 DHCP 服务器最重要的部分：`subnet`范围。在这里，我们定义了分配给客户端的 IP 地址的网络范围。我们需要提供子网网络地址、其子网掩码，然后是我们要允许客户端的开始和结束 IP 地址范围。在我们的示例中，我们允许主机 IP 地址从`192.168.1.10`，`192.168.1.11`，`192.168.1.12` ...到`192.168.1.160`。如果您有多个子网，可以使用多个`subnet`范围指令（称为多宿主 DHCP 服务器）。

接下来，我们启动了 DHCP 服务器并在启动时启用它。现在，您的客户端应该能够从我们的新系统动态获取 IP 地址。

总之，我们只向您展示了一些非常基本的 DHCP 服务器配置选项，以帮助您入门，并且还有许多其他设置可用，允许您构建非常复杂的 DHCP 服务器解决方案。要更好地了解其可能性，请查看 DHCP 服务器文档提供的示例配置文件`less /usr/share/doc/dhcp-4*/dhcpd.conf.example`。

## 还有更多...

在主配方中，我们配置了基本的 DHCP 服务器，以便能够向客户端发送完整的 IP 网络信息，使他们能够加入我们的网络。要使用此服务器，您需要在客户端的网络接口上启用 DHCP 寻址。在 CentOS 客户端上，请不要忘记使用`BOOTPROTO=dhcp`并删除所有静态条目，例如`IPADDR`在适当的网络脚本`ifcfg`文件中（阅读配方，*构建静态网络连接*在第二章，*配置系统*以帮助您开始使用网络脚本文件）。然后，要进行 DHCP 请求，请使用`systemctl restart network`重新启动网络或尝试重新启动客户端系统（使用`ONBOOT=yes`选项）。使用`ip addr list`进行确认。

# 使用 WebDAV 进行文件共享

**基于 Web 的分布式创作和版本控制**（**WebDAV**）开放标准可用于网络上的文件共享。它是一种流行的协议，可以方便地访问远程数据，就像一个*在线硬盘*。许多在线存储和电子邮件提供商通过 WebDAV 账户提供在线空间。大多数图形化的 Linux 或 Windows 系统都可以在其文件管理器中开箱即用地访问 WebDAV 服务器。对于其他操作系统，也有免费的选择。另一个很大的优势是 WebDAV 运行在普通的 HTTP 或 HTTPS 端口上，因此您可以确保它几乎在任何环境中都能工作，即使是在受限的防火墙后面。

在这里，我们将向您展示如何安装和配置 WebDAV 作为 FTP 协议的替代方案，以满足您的文件共享需求。我们将使用 HTTPS 作为我们的通信协议，以实现安全连接。

## 准备就绪

要完成本教程，您需要一个具有 root 权限的工作 CentOS 7 操作系统和一个您选择的基于控制台的文本编辑器。您需要一个在您的网络中可访问的带有 SSL 加密的工作 Apache Web 服务器；请参阅第十一章，*提供邮件服务*，了解如何安装 HTTP 守护程序，特别是*使用 SSL 设置 HTTPS*的教程。此外，熟悉 Apache 配置文件格式也是有利的。

## 如何操作…

1.  创建一个用于共享数据和 WebDAV 锁文件的位置：

    ```
    mkdir -p /srv/webdav /etc/httpd/var/davlock

    ```

1.  由于 WebDAV 作为 Apache 模块在 HTTPS 上运行，我们必须为标准`httpd`用户设置适当的权限：

    ```
    chown apache:apache /srv/webdav /etc/httpd/var/davlock
    chmod 770 /srv/webdav

    ```

1.  现在，创建并打开以下 Apache WebDAV 配置文件：

    ```
    vi /etc/httpd/conf.d/webdav.conf

    ```

1.  输入以下内容：

    ```
    DavLockDB "/etc/httpd/var/davlock"
    Alias /webdav /srv/webdav
    <Location /webdav>
        DAV On
        SSLRequireSSL
        Options None
        AuthType Basic
        AuthName webdav
        AuthUserFile /etc/httpd/conf/dav_passwords
        Require valid-user
    </Location>
    ```

1.  保存并关闭文件。现在，要添加一个名为`john`的新 WebDAV 用户（在提示时为该用户输入新密码）：

    ```
    htpasswd -c /etc/httpd/conf/dav_passwords john

    ```

1.  最后，重新启动 Apache2 Web 服务器：

    ```
    systemctl restart httpd

    ```

1.  要测试我们是否可以连接到我们的 WebDAV 服务器，您可以使用任何客户端网络上的图形用户界面（大多数 Linux 文件管理器支持 WebDAV 浏览），或者我们可以使用命令行挂载驱动器。

1.  在任何客户端机器上以`root`身份登录，该机器与我们的 WebDAV 服务器位于同一网络中（在 CentOS 上，您需要从 EPEL 仓库安装`davfs2`文件系统驱动程序包，并且必须禁用文件锁的使用，因为当前版本无法与文件锁一起工作），输入我们的 DAV 用户账户名为`john`的密码，并在询问时确认自签名证书：

    ```
    yum install davfs2
    echo "use_locks 0" >> /etc/davfs2/davfs2.conf
    mkdir /mnt/webdav
    mount -t davfs https://<WebDAV Server IP>/webdav /mnt/webdav

    ```

1.  现在，让我们看看是否可以写入新的网络存储类型：

    ```
    touch /mnt/webdav/testfile.txt

    ```

1.  如果您遇到连接问题，请检查 WebDAV 服务器上的防火墙设置，以及客户端上的`http`和`https`服务。

## 它是如何工作的…

在本教程中，我们向您展示了如何轻松设置 WebDAV 服务器以实现简单的文件共享。

那么，我们从这次经历中学到了什么？

我们的旅程始于创建两个目录：一个用于存放 WebDAV 服务器的所有共享文件，另一个用于为 WebDAV 服务器进程创建锁定文件数据库。后者是必需的，以便用户可以*阻止*对文档的访问，以避免与其他人发生冲突，如果文件当前正在被他们修改。由于 WebDAV 作为本机 Apache 模块（`mod_dav`）运行，该模块在 CentOS 7 中默认启用，因此我们只需要创建一个新的 Apache 虚拟主机配置文件，我们可以在其中设置所有 WebDAV 设置。首先，我们必须将 WebDAV 主机链接到用于跟踪用户锁定的锁定数据库的完整路径。接下来，我们为 WebDAV 共享文件夹定义了一个别名，然后使用`Location`指令对其进行了配置。如果有人在`/webdav`路径 URL 上使用特定的 HTTP 方法，这将激活。在此区域内，我们指定此 URL 将是一个 DAV 启用的共享，为其启用 SSL 加密，并指定基于用户的密码身份验证。用户帐户的密码将存储在名为`/etc/httpd/conf/dav_passwords`的用户帐户数据库中。为了在此数据库文件中创建有效帐户，我们随后在命令行上使用了 Apache2 `htpasswd`实用程序。最后，我们重新启动了服务以应用我们的更改。

为了测试，我们使用了`davfs`文件系统驱动程序，您需要在 CentOS 7 上使用 EPEL 存储库中的`davfs2`包进行安装。还有许多其他选项，例如`cadaver` WebDAV 命令行客户端（也来自 EPEL 存储库）；或者，您可以直接使用 GNOME、KDE 或 Xfce 等图形用户界面中的集成 WebDAV 支持来访问它。

# 安装和配置 NFS

**网络文件系统**（**NFS**）协议通过网络连接实现对文件系统的远程访问。它基于客户端-服务器架构，允许中央服务器与其他计算机共享文件。客户端可以将这些导出的共享挂载到自己的文件系统中，以便方便地访问，就像它们位于本地存储上一样。虽然 Samba 和 AFP 在 Windows 和 OS X 上更常见的分布式文件系统，但 NFS 现在已成为事实上的标准，是任何 Linux 服务器系统的关键组成部分。在本食谱中，我们将向您展示如何轻松设置 NFS 服务器以在网络上共享文件。

## 准备工作

要完成此操作，您需要具备 CentOS 7 操作系统的有效安装，具有 root 权限，您选择的基于控制台的文本编辑器以及连接到互联网以方便下载其他软件包。预计您的 NFS 服务器和所有客户端将能够相互 ping 通，并且通过静态 IP 地址相互连接（请参阅第二章，*配置系统*中的*建立静态网络连接*配方）。在我们的示例中，NFS 服务器以 IP`192.168.1.10`运行，两个客户端的 IP 分别为`192.168.1.11`和`192.168.1.12`，网络的域名为`example.com`。

## 如何操作...

在本节中，我们将学习如何安装和配置 NFS 服务器，并在客户端上创建和导出共享。

### 安装和配置 NFS 服务器

NFSv4 默认未安装，因此我们将首先下载并安装所需的软件包：

1.  为此，请以`root`身份登录到您要运行 NFS 守护程序的服务器上，并键入以下命令以安装所需的软件包：

    ```
    yum install nfs-utils

    ```

1.  为了使 NFSv4 正常工作，我们需要所有客户端和 NFS 服务器的*相同基*域。因此，如果我们尚未使用 DNS 设置域名（请参阅第九章，*域操作*），我们将在`/etc/hosts`文件中为我们的计算机设置一个新主机名：

    ```
    echo "192.168.1.10 myServer.example.com" >> /etc/hosts
    echo "192.168.1.11 myClient1.example.com" >> /etc/hosts
    echo "192.168.1.12 myClient2.example.com" >> /etc/hosts
    ```

1.  现在，打开`/etc/idmapd.conf`文件，并输入 NFS 服务器的基域名（不是完整的域名）；查找读取`#Domain = local.domain.edu`的行，并将其替换为以下内容：

    ```
    Domain = example.com
    ```

1.  接下来，我们需要为服务器打开一些防火墙端口，以便它具有适当的 NFS 访问权限：

    ```
    for s in {nfs,mountd,rpc-bind}; do firewall-cmd --permanent --add-service $s; done; firewall-cmd --reload

    ```

1.  最后，让我们启动 NFS 服务器服务并在重启时启用它：

    ```
    systemctl start rpcbind nfs-server systemctl enable rpcbind nfs-server systemctl status nfs-server

    ```

### 创建导出共享

既然我们的 NFS 服务器已配置并运行，现在是时候创建一些文件共享，我们可以将其导出到我们的客户端：

1.  首先，让我们为我们的共享创建一个文件夹并更改其权限：

    ```
    mkdir /srv/nfs-data

    ```

1.  创建一个具有特定 GID 的新组，并将其与导出关联，然后更改权限：

    ```
    groupadd -g 50000 nfs-share;chown root:nfs-share /srv -R;chmod 775 /srv -R

    ```

1.  打开以下文件：

    ```
    vi /etc/exports

    ```

1.  现在，输入以下文本，但输入时要非常专注：

    ```
    /srv/nfs-data *(ro) 192.168.1.11(rw) 192.168.1.12(rw) /home *.example.com(rw)

    ```

1.  保存并关闭文件，然后使用以下命令重新导出`/etc/exports`中的所有条目：

    ```
    exportfs -ra

    ```

## 它是如何工作的...

在 CentOS 7 上，您可以安装版本 4 的 NFS，它比以前的版本有一些增强，例如更灵活的认证选项，并且与旧版本的 NFS 完全向后兼容。在这里，我们向您展示了安装和配置 NFS 服务器以及为我们的客户端创建一些共享导出是多么容易。

那么，我们从这次经历中学到了什么？

我们首先通过安装`nfs-utils`包来启动这个配置过程，因为在 CentOS 7 上默认情况下 NFS 服务器功能是不可用的。接下来，我们使用`/etc/hosts`文件配置了服务器域名，因为在我们的示例中，我们还没有配置自己的 DNS 服务器。如果你已经设置了 DNS 服务器，你应该遵循与此处类似的域名模式，因为这对 NFSv4 的正常工作至关重要，因为所有客户端和服务器都应该位于同一个基本域中。在我们的示例中，我们指定了它们都是`example.com`的子域：`myClient1.example.com`，`myClient2.example.com`和`myServer.example.com`。这是一种确保数据共享安全的方法，因为 NFS 服务器只允许来自客户端的文件访问，如果域名匹配（在我们的示例中，服务器和客户端都是`example.com`域的一部分）。接下来，我们将这个基本域放入`idmapd.conf`文件中，该文件负责将用户名和组 ID 映射到 NFSv4 ID。之后，我们在 firewalld 实例中启用了`nfs`，`mountd`和`rpc-bind`服务，这些都是客户端和服务器之间完整支持和通信所必需的。为了完成我们的基本配置，我们启动了`rpcbind`和 NFS 服务器，并设置了开机自启。

在成功设置 NFS 服务器之后，我们为其添加了一些导出项，实际上允许客户端从服务器访问一些共享文件夹。因此，我们在文件系统中创建了一个特殊的目录，用于存放我们所有的共享文件。我们将这个共享文件夹`/srv/nfs-data`与一个新组`nfs-share`关联，并赋予其读/写/执行权限。出于实际原因，我们将在组级别上控制我们的导出 Linux 文件权限。名称并不重要，但其组标识符（GID）必须设置为静态值（例如，`50000`）。这个新的 GID 在服务器和每个客户端上都必须相同，对于每个想要拥有写权限的用户来说，因为 NFS 在服务器和客户端之间通过用户（UID）或 GID 级别在网络上传递任何访问权限。整个共享的魔法发生在`/etc/exports`文件中。它包含一个表格；在其中，你指定了所有关于你的共享文件夹及其客户端访问安全性的重要信息。这个文件中的每一行都相当于系统中的一个共享文件夹，以及一个允许访问它们的以空格分隔的主机列表，以及它们的访问选项。如你所见，有不同的方法来定义目标客户端，使用 IP 地址或主机名。对于主机名，你可以使用通配符如`*`和`?`来使文件更紧凑，并允许一次多个机器，但你也可以为每个单独的主机名定义导出选项。解释所有选项超出了本书的范围；如果你需要更多帮助，请阅读导出手册，可以通过`man exports`找到。

例如，行 `/srv/nfs-data *(ro) 192.168.1.11(rw) 192.168.1.12(rw)` 定义了我们希望将 `/srv/nfs-data` 文件夹的内容导出到所有主机名（因为使用了 `*` 符号）；只读 (`ro`) 意味着每个客户端都可以读取文件夹的内容但不能写入。对于 IP 地址以 `192.168.1` 结尾，且末尾为 `11` 和 `12` 的客户端，我们允许读写 (`rw`)。第二行定义了我们正在将 `/home` 目录导出到 `*.example.com` 子域中的所有客户端，并具有读写能力。每当您对 `/etc/exports` 文件进行更改时，运行 `exportfs -r` 命令以将更改应用到 NFS 服务器。

最后，我们可以说在 CentOS 7 中设置和启动 NFSv4 非常容易。它是 Linux 系统之间共享文件或集中式家目录的完美解决方案。

# 使用 NFS

在客户端计算机可以使用 NFS 服务器共享的文件系统导出之前，它必须配置为正确访问该系统。在本步骤中，我们将向您展示如何在客户端机器上设置和使用 NFS。

## 准备工作

要完成此操作，您需要一个具有 root 权限的 CentOS 7 操作系统的工作安装，您选择的基于控制台的文本编辑器，以及互联网连接以便下载额外的软件包。我们假设您已经遵循了 *安装和配置 NFS* 的步骤，并已经设置了一个 NFS 服务器，例如在本例中。我们假设所有客户端都可以相互 ping 通，并且连接到了 NFS 服务器，并且将使用静态 IP 地址（请参阅 第二章 *系统配置* 中的 *建立静态网络连接* 步骤）。在我们的示例中，NFS 服务器运行在 IP `192.168.1.10` 上，两个客户端分别在 IP `192.168.1.11` 和 `192.168.1.12` 上。

## 操作步骤...

在我们的客户端系统上，我们也需要相同的 NFS 软件包，以及与服务器上类似的配置，以便在它们之间建立通信：

1.  首先，以 `root` 身份登录到您的客户端，并应用与 *安装和配置 NFS* 步骤中完全相同的步骤，直到步骤 3 结束。跳过步骤 4，因为不需要打开 firewalld 服务。然后，在步骤 5 中，使用以下命令，这些命令不会启动和启用 `nfs-server`，而只会启动 `rpcbind` 服务：

    ```
    systemctl start rpcbind
    systemctl enable rpcbind

    ```

1.  在此停止，不要应用原始步骤中的其他内容。为了测试与我们的 NFS 服务器的连接，请使用以下命令：

    ```
    showmount -e myServer.example.com

    ```

1.  现在，为了测试挂载 NFS 导出是否有效，您可以手动使用新用户 `john` 进行测试。首先需要将 `john` 添加到 `nfs-share` 组中，以便我们可以在共享上进行写操作：

    ```
    groupadd -g 50000 nfs-share;useradd john;passwd john;usermod -G nfs-share john
    mount -t nfs4 myServer.example.com:/srv/nfs-data /mnt
    su - john;touch /mnt/testfile.txt

    ```

1.  如果在共享目录中创建文件成功，您可以将导入项放入`fstab`文件中，以便在系统启动时自动挂载：

    ```
    vi /etc/fstab

    ```

1.  添加以下行：

    ```
    myServer.example.com:/srv/nfs-data  /mnt nfs defaults 0 0

    ```

1.  最后，要从`fstab`重新挂载所有内容，请键入以下内容：

    ```
    mount -a

    ```

## 它是如何工作的...

在本食谱中，我们向您展示了从现有的 NFSv4 服务器使用一些共享文件系统导出是多么容易。

那么，我们从这次经历中学到了什么？

正如您所见，要设置 NFS 客户端，您需要与 NFS 服务器本身非常相似的设置，除了启动`rpcbind`服务而不是`nfs-server`（顾名思义，仅在服务器端需要）。`rpcbind`服务是一个端口映射器，用于**远程过程调用**（**RPC**），这是 NFS 工作所需的通信标准。在配置中您应该记住的另一个非常重要的步骤是在`/etc/idmapd.conf`文件中设置域名。我们必须在服务器上使用*相同*的基本域名（`example.com`），以便在服务器和客户端之间进行 NFSv4 通信。在启动并启用`rpcbind`服务后，我们可以将 NFS 共享挂载到本地目录，要么直接使用`mount`命令（使用`-t`类型`nfs4`），要么通过`fstab`文件。请记住，每个想要对共享具有适当的读/写/执行权限的系统用户都需要在 NFS 服务器上具有*相同*的权限；在我们的示例中，我们在相同的 GID 级别上管理正确的权限。我们使用默认选项挂载共享；如果您需要不同的或高级的选项，请参阅`man fstab`。为了应用对`fstab`文件的更改，执行`mount -a`以从该文件重新挂载所有内容。

# 使用 Samba 安全地共享资源

**Samba**是一个软件包，它使您能够跨网络共享文件、打印机和其他常见资源。对于任何工作环境来说，它都是一个无价的工具。在异构网络（即不同的计算机系统，如 Windows 和 Linux）上共享文件资源的最常见方式之一是安装和配置 Samba 作为独立文件服务器，通过*用户级安全*使用系统用户的家目录提供基本的文件共享服务。独立服务器被配置为提供本地身份验证和对其维护的所有资源的访问控制。总而言之，每个管理员都知道 Samba 仍然是一个非常流行的开源发行版，本食谱的目的是向您展示如何提供一种即时文件共享方法，该方法可以在整个工作环境中无缝集成任何数量的用户在任何类型的现代计算机上。

## 准备

要完成此配方，您需要一个具有 root 权限的 CentOS 7 操作系统的工作安装，您选择的基于控制台的文本编辑器，以及连接到 Internet 以便下载其他软件包。预计您的服务器将使用静态 IP 地址。

## 如何操作...

Samba 默认不安装，因此我们将从下载和安装所需软件包开始。

1.  为此，以`root`身份登录并键入以下命令以安装所需的软件包：

    ```
    yum install samba samba-client samba-common

    ```

1.  完成此操作后，第一步是重命名原始配置文件：

    ```
    mv /etc/samba/smb.conf /etc/samba/smb.conf.BAK

    ```

1.  现在，在您喜欢的文本编辑器中创建一个新的配置文件，方法是键入以下内容：

    ```
    vi /etc/samba/smb.conf

    ```

1.  开始构建新的配置，方法是添加以下行，将显示的值替换为更好地代表您自己需求的值：

    ```
    [global]
    unix charset = UTF-8
    dos charset = CP932
    workgroup = <WORKGROUP_NAME>
    server string = <MY_SERVERS_NAME>
    netbios name = <MY_SERVERS_NAME>
    dns proxy = no
    wins support = no
    interfaces = 127.0.0.0/8 XXX.XXX.XXX.XXX/24 <NETWORK_NAME>
    bind interfaces only = no
    log file = /var/log/samba/log.%m
    max log size = 1000
    syslog only = no
    syslog = 0
    panic action = /usr/share/samba/panic-action %d
    ```

    ### 注意

    `WORKGROUP_NAME`是 Windows 工作组的名称。如果您没有此值，请使用标准 Windows 名称`WORKGROUP`。`MY_SERVERS_NAME`指的是您的服务器名称。在大多数情况下，这可能是`FILESERVER`或`SERVER1`等形式。`XXX.XXX.XXX.XXX/XX`指的是 Samba 服务运行的主要网络地址，例如`192.168.1.0/24`。`NETWORK_NAME`指的是您的以太网接口的名称。这可能是`enp0s8`。

1.  现在，我们将配置 Samba 作为独立服务器。为此，只需继续将以下行添加到主配置文件中：

    ```
    security = user
    encrypt passwords = true
    passdb backend = tdbsam
    obey pam restrictions = yes
    unix password sync = yes
    passwd program = /usr/bin/passwd %u
    passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
    pam password change = yes
    map to guest = bad user
    usershare allow guests = no
    ```

1.  对于这个配方，我们不打算将 Samba 配置为域主或主浏览器。为此，添加以下行：

    ```
    domain master = no
    local master = no
    preferred master = no
    os level = 8
    ```

1.  现在，我们将添加对主目录共享的支持，允许有效用户访问其主目录。此功能将支持适当的读/写权限，并且所有文件夹都将保持对其他用户的私密性。为此，添加以下新行：

    ```
    [homes]
         comment = Home Directories
         browseable = no
         writable = yes
         valid users = %S
         create mask =0755
         directory mask =0755
    ```

1.  保存并关闭文件。要测试我们刚刚创建的 Samba 配置文件的语法，请使用以下命令：

    ```
    testparm

    ```

1.  现在，将现有系统用户`john`添加到 Samba 用户管理系统（这是为了稍后测试；请根据您系统上的用户名适当更改）：

    ```
    smbpasswd -a john

    ```

1.  现在，保存文件并关闭它；回到命令行，打开防火墙中的端口：

    ```
    firewall-cmd --permanent --add-service=samba && firewall-cmd --reload

    ```

1.  配置 SELinux 以使用 Samba 主目录：

    ```
    setsebool -P samba_enable_home_dirs on

    ```

1.  现在，确保`samba`和`nmb`服务将在启动过程中启动，并立即启动它们：

    ```
    systemctl enable smb && systemctl enable nmb systemctl start smb && systemctl start nmb

    ```

## 它是如何工作的...

本配方的目的是安装 Samba 并配置其文件共享服务，从而为您的网络中的所有现代计算机系统提供完全的连接性。

那么，我们从这次经历中学到了什么？

安装了必要的软件包后，我们将原始安装的配置文件重命名，以便在以后出现问题时备份，然后我们开始从头设置 Samba，从空白的`smb.conf`配置文件开始。打开这个新文件后，我们开始设置全局配置选项；第一步是声明与基于 Unicode 的字符集兼容。您需要注意，由于您的具体情况和网络，值可能会有所不同。更多信息请参阅`man smb.conf`。

完成这一步后，我们接着确认了工作组和服务器的名称，禁用了 WINS，建立了 Samba 日志文件，并注册了网络接口。然后，我们选择了以下独立选项，包括基于用户的安全选项、密码加密和`tdbsam`数据库后端。首选的安全模式是用户级安全，采用这种方法意味着每个共享可以分配给特定用户。因此，当用户请求连接共享时，Samba 通过验证配置文件和 Samba 数据库中的授权用户提供的用户名和密码来验证此请求。接下来，我们添加了`master`信息。在混合操作系统环境中，当单个客户端试图成为主浏览器时，将产生已知的冲突。这种情况可能不会破坏整个文件共享服务，但会在 Samba 日志文件中记录潜在问题。因此，通过配置 Samba 服务器不声明自己为主浏览器，您将能够降低此类问题被报告的可能性。因此，完成这些步骤后，该方案接下来考虑了启用`homes`目录文件共享的主要任务。当然，您可以尝试显示的选项，但这一简单的指令集不仅确保了有效用户能够使用相关的读/写权限访问其主目录，而且通过将`browseable`标志设置为`no`，您还能够隐藏主目录，使其不在公共视图中显示，从而为用户提供更高程度的隐私。在我们的设置中，Samba 与您的 Linux 系统用户配合工作，但您应该记住，任何现有或新用户都不会自动添加到 Samba 中，必须使用`smbpasswd -a`手动添加。

因此，在保存您的新配置文件后，我们使用`testparm`程序测试其正确性，并使用`samba`服务在 firewalld 中打开与 Samba 相关的传入端口。下一步是确保在启动过程中使用`systemctl`使 Samba 及其相关进程可用。Samba 为了正确工作需要两个主要进程：`smbd`和`nmbd`。从`smbd`开始，该服务的角色是为使用 SMB（或 CIFS）协议的 Windows 客户端提供文件共享、打印服务、用户认证和资源锁定。同时，`nmbd`服务的角色是监听、理解和回复 NetBIOS 名称服务的请求。

### 注意

Samba 通常包括另一个名为`winbindd`的服务，但由于提供基于**Windows Internet Naming Service**（**WINS**）的服务或 Active Directory 认证需要额外的考虑，这超出了本食谱的范围，因此它被广泛忽略。

因此，我们的最终任务是启动 Samba 服务（`smb`）和相关的 NetBIOS 服务（`nmb`）。

您现在知道安装、配置和维护 Samba 是多么简单。总有更多要学习的内容，但这个简单的介绍已经说明了 Samba 的相对易用性和其语法的简单性。它提供了一个解决方案，能够支持各种不同的需求和一系列不同的计算机系统，它将满足您未来多年的文件共享需求。

## 还有更多...

您可以从网络中的任何客户端测试我们的 Samba 服务器配置，只要该客户端可以 ping 通服务器。如果是基于 Windows 的客户端，请打开**Windows 资源管理器**地址栏，并使用以下语法：`\\<Samba 服务器的 IP 地址>\<Linux 用户名>`。例如，我们使用`\\192.168.1.10\john`（成功连接后，您需要输入 Samba 用户名的密码）。在任何 Linux 客户端系统上（在 CentOS 7 上需要安装`samba-client`包），要列出 NFS 服务器的所有可用共享，请使用以下命令：

```
smbclient -L <hostname or IP address of NFS server> -U <username>

```

在我们的示例中，我们将使用以下内容：

```
smbclient -L 192.168.1.10 -U john

```

要进行测试，请使用以下语法挂载共享（这需要在 CentOS 7 上安装`cifs-utils`包）：

```
mount -t cifs  //<ip address of the Samba server>/<linux username> <local mount point> -o  "username=<linux username>"

```

在我们的示例中，我们将使用以下内容：

```
mkdir /mnt/samba-share
mount -t cifs //192.168.1.10/john  /mnt/samba-share -o "username=john"

```

您还可以将此导入放入`/etc/fstab`文件中以进行永久挂载，使用以下语法：

```
//<server>/<share> <mount point> cifs <list of options>  0  0

```

例如：

例如，向文件中添加以下行：

```
//192.168.1.10/john /mnt/samba-share cifs username=john,password=xyz  0 0

```

如果您不想在此文件中使用明文密码，请阅读有关使用`man mount.cifs`的凭据的部分，然后创建一个凭据文件，并使用`chmod 600`在您的主目录中保护它，以确保没有其他人可以读取它。

在本章中，我们向您展示了如何将 Samba 配置为独立服务器并启用家目录，以及如何从客户端连接到它以开始使用。但 Samba 的功能远不止于此！它可以提供打印服务或充当完整的域控制器。如果您想了解更多信息，请随时访问[`www.packtpub.com/`](https://www.packtpub.com/)以了解其他可用材料。
