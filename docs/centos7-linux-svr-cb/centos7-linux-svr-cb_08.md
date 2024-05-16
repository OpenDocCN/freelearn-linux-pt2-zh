# 第八章。使用 FTP

在本章中，我们将介绍以下主题：

+   安装和配置 FTP 服务

+   使用虚拟 FTP 用户

+   定制 FTP 服务

+   解决用户和文件传输问题

# 引言

本章是一系列操作的集合，提供了揭示 Linux 世界中最基本服务之一的步骤，并提供了安装、配置和无犹豫地提供文件传输协议所需的起点。

# 安装和配置 FTP 服务

尽管存在几种现代且非常安全的网络文件共享技术，但古老的**文件传输协议**（**FTP**）仍然是计算机之间共享和传输文件的最广泛使用的协议之一。在 Linux 世界中，有多种不同的 FTP 服务器可用。在本操作中，您将学习如何安装和配置**非常安全的 FTP 守护程序**（**vsftpd**），这是一个著名的 FTP 服务器解决方案，支持广泛的功能，并允许您在本地网络和互联网上上传和分发大文件。在这里，我们将展示如何安装 vsftpd 守护程序，并提供一些基本设置，主要目标是提高守护程序的安全性。

### 注意

完成此操作后，建议使用 SSL/TLS 加密以进一步增强您的 FTP 服务器（请参阅第六章，*使用 SELinux*，以了解更多关于 SELinux 的信息。接下来，我们在`systemd`中启用`vsftpd`开机启动并启动服务。此时，`vsftpd`将开始运行，并且可以使用任何常规的 FTP 桌面软件进行测试。用户可以使用有效的系统用户名和密码通过连接到服务器的名称、域或 IP 地址（取决于服务器的配置）进行登录。

本食谱的目的是向你展示`vsftpd`并不是一个难以安装和配置的软件包。总有很多事情要做，但是通过这个简单的介绍，我们已经迅速启用了我们的服务器来运行标准的 FTP 服务。

## 还有更多...

安装并配置了基本的 FTP 服务后，你可能会想知道如何将用户引导到其主目录中的特定文件夹。为此，打开你选择的编辑器中的主配置文件`/etc/vsftpd/vsftpd.conf`。

滚动到文件底部，并通过将`<users_local_folder_name>`值替换为更适合你需求的值来添加以下行：

```
local_root=<users_local_folder_name>

```

例如，如果这个 FTP 服务器主要是为了访问和上传用户私人网页的内容，而这些网页托管在同一服务器上，你可能需要配置 Apache 使用用户的主目录中的一个名为/`home/<username>/public_html`的文件夹。为此，你可以在`vsftpd`配置文件的底部添加以下参考：

```
local_root=public_html

```

完成后，保存并关闭配置文件，然后重新启动`vsftpd`服务。在测试此新功能时，请确保`local_root`位置存在于你想要登录的用户的家目录中（例如，`~/public_html`）。

# 使用虚拟 FTP 用户

在本食谱中，你将学习如何实现虚拟用户，以摆脱使用本地系统用户账户的限制。在你的服务器生命周期中，可能会有时候你希望为没有本地系统账户的用户启用 FTP 认证。你可能还希望考虑实施一个解决方案，允许特定个人维护多个账户，以便访问服务器上的不同位置。这种配置意味着使用虚拟用户提供了一定程度的灵活性。由于你不是使用本地系统账户，可以说这种方法提供了改进的安全性。

## 准备就绪

要完成此步骤，你需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，以及你选择的基于控制台的文本编辑器。预计你的服务器将使用静态 IP 地址，并且`vsftpd`已经安装了 chroot 监狱并且正在运行。此步骤需要安装`policycoreutils-python`包。

## 如何做到这一点...

1.  第一步是作为 root 登录到我们的`vsftpd`服务器，并创建一个名为`virtual-users.txt`的纯文本文件，该文件维护虚拟用户的用户名和密码列表。为此，请输入以下命令：

    ```
    vi /tmp/virtual-users.txt

    ```

1.  现在以以下方式添加你的用户名和相应的密码：

    ```
    virtual-username1
    password1
    virtual-username2
    password2
    virtual-username3
    password3

    ```

    ### 注意

    根据需要为每个用户重复此过程，但出于明显的原因，请保持良好的密码策略，并且不要重复使用同一个虚拟用户名。

1.  完成操作后，只需按照常规方式保存并关闭文件。然后，通过输入以下命令来构建数据库文件：

    ```
    db_load -T -t hash -f /tmp/virtual-users.txt /etc/vsftpd/virtual-users.db

    ```

1.  完成此操作后，我们现在将创建将使用此数据库验证虚拟用户的 PAM 文件。为此，请输入以下命令：

    ```
    vi /etc/pam.d/vsftpd-virtual

    ```

1.  现在添加以下行：

    ```
    auth required pam_userdb.so db=/etc/vsftpd/virtual-users
    account required pam_userdb.so db=/etc/vsftpd/virtual-users

    ```

1.  完成后，按照常规方式保存并关闭文件。按照以下方式在你的首选文本编辑器中打开主`vsftpd`配置文件：

    ```
    vi /etc/vsftpd/vsftpd.conf

    ```

1.  现在，在打开的文件中，找到`pam_service_name=vsftpd`行，并通过在行首添加`#`符号来禁用它，使其读作如下：

    ```
    #pam_service_name=vsftpd

    ```

1.  向下滚动到文件底部，并通过自定义`local_root`的值以满足你自己的特定需求来添加以下行——这将是所有虚拟用户将*居住*的基本目录（例如，我们将使用`/srv/virtualusers/$USER`，如下所示）：

    ```
    virtual_use_local_privs=YES
    guest_enable=YES
    pam_service_name=vsftpd-virtual
    user_sub_token=$USER
    local_root=/srv/virtualusers/$USER
    hide_ids=YES

    ```

1.  现在在你之前定义的`/tmp/virtual-users.txt`文件中为每个虚拟用户创建一个子文件夹，并在你使用`local_root`指令指定的目录中。记得将此文件夹的所有权委派给 FTP 用户。为了保持我们的`/srv/virtualusers`示例，我们将使用以下命令以自动方式执行此操作（再次，如果需要，请自定义`/srv/virtualusers`目录）：

    ```
    for u in `sed -n 1~2p /tmp/virtual-users.txt`;
    do
    mkdir -p /srv/virtualusers/$u
    chown ftp: /srv/virtualusers/$u
    done

    ```

1.  现在我们需要通知 SELinux 允许对我们的自定义`local_root`目录进行读/写访问，该目录位于典型的`/home`目录之外：

    ```
    setsebool -P allow_ftpd_full_access on
    semanage fcontext -a -t public_content_rw_t "/srv/virtualusers(/.*)?"
    restorecon -R -v /srv/virtualusers

    ```

1.  接下来，按照以下方式重新启动 FTP 服务：

    ```
    systemctl restart vsftpd

    ```

1.  出于安全原因，现在删除纯文本文件，并使用以下命令保护生成的数据库文件：

    ```
    rm /tmp/virtual-users.txt
    chmod 600 /etc/vsftpd/virtual-users.db

    ```

## 它是如何工作的...

遵循前面的步骤后，你现在将能够邀请无限数量的虚拟用户访问你的 FTP 服务。此功能的配置非常简单；你的整体安全性得到了提升，并且所有访问都限制在你选择的定义的`local_root`目录中。请注意，使用虚拟用户将禁用系统用户从第一个步骤登录到 FTP 服务器。

那么我们从这次经历中学到了什么？

我们首先创建了一个新的临时文本文件，该文件将包含我们所有用户名及其对应的明文密码。然后，我们逐个添加所有必需的用户名和密码，每行一个。完成对每个虚拟用户的这一步骤后，我们保存并关闭了文件，然后运行了 CentOS 7 默认安装的`db_load`命令。该命令用于从我们的文本文件生成一个 BerkeleyDB 数据库，稍后将用于 FTP 用户认证。完成这一步骤后，我们的下一个任务是在`/etc/pam.d/vsftpd-virtual`创建一个 Pluggable Authentication Modules（PAM）文件。该文件读取前面的数据库文件，使用典型的 PAM 配置文件语法（更多信息，请参阅`man pam.d`）为我们的`vsftpd`服务提供认证。然后，我们打开、修改并添加新的配置指令到主`vsftpd`配置文件`/etc/vsftpd/vsftpd.conf`，以便让`vsftpd`通过 PAM 意识到我们的虚拟用户认证。

最重要的设置是`local_root`指令，它定义了所有虚拟用户目录的基本位置。别忘了在路径末尾加上`$USER`字符串。然后，您被提示为文本文件中定义的每个虚拟用户创建相关的虚拟主机文件夹。

由于虚拟用户不是真正的系统用户，我们必须将 FTP 系统用户分配给我们的新 FTP 用户，以完全拥有这些文件。我们使用 bash `for`循环来自动化对临时`/tmp/virtual-users.txt`文件中定义的所有用户的处理过程。接下来，我们设置了正确的 SELinux 布尔值，以允许虚拟用户访问系统，并为我们的`/srv/virtualusers`目录设置了正确的上下文。应用所有这些更改只需使用`systemctl`命令重新启动`vsftpd`服务。

之后，我们删除了包含我们明文密码的临时用户文本文件。我们通过删除除 root 之外的所有访问权限来保护 BerkeleyDB 数据库文件。如果您定期更新、添加或删除 FTP 用户，最好不要删除这个临时明文`/tmp/virtual-users.txt`文件，而是将其放在安全的地方，例如`/root`目录。然后，您应该使用`chmod 600`来保护它。然后，每当您对这个文件进行更改时，都可以重新运行`db_load`命令以保持用户信息的最新状态。如果您需要在以后添加新用户，您还必须为他们创建新的虚拟用户文件夹（请重新运行第 9 步的命令）。之后运行`restorecon -R -v /srv/virtualusers`命令。

现在，您可以通过使用本菜谱中创建的新账户登录 FTP 服务器来测试您的新虚拟用户账户。

# 自定义 FTP 服务

在本教程中，您将学习如何自定义您的`vsftpd`安装。`vsftpd`有许多配置参数，这里我们将展示如何创建一个自定义欢迎横幅，更改服务器的默认超时时间，限制用户连接，以及禁止用户访问服务。

## 准备就绪

要完成本教程，您需要一个具有 root 权限的 CentOS 7 操作系统的工作安装和一个您选择的基于控制台的文本编辑器。预计您的服务器将使用静态 IP 地址，并且`vsftpd`已经安装了 chroot 监狱并且正在运行。

## 如何做到这一点...

1.  首先，以 root 身份登录并打开主要的`vsftpd`配置文件：

    ```
    vi /etc/vsftpd/vsftpd.conf

    ```

1.  首先提供一个替代的欢迎信息，取消注释以下行，并根据需要修改信息。例如，您可以使用这个：

    ```
    ftpd_banner=Welcome to my new FTP server

    ```

1.  要更改默认 FTP 超时时间，取消注释这些行并根据需要替换数值：

    ```
    idle_session_timeout=600
    data_connection_timeout=120

    ```

1.  现在，我们将限制连接：数据传输速率每秒字节数，客户端数量，以及每个 IP 地址的最大并行连接数。在文件末尾添加以下行：

    ```
    local_max_rate=1000000
    max_clients=50 
    max_per_ip=2

    ```

1.  接下来，保存并关闭文件。要禁止特定用户，您可以使用以下命令，同时将用户名替换为适合您需求的适当系统用户值：

    ```
    echo "username" >> /etc/vsftpd/user_list

    ```

1.  现在要应用更改，请重启 FTP 服务：

    ```
    systemctl restart vsftpd

    ```

## 它是如何工作的...

在本教程中，我们已经展示了一些最重要的`vsftpd`设置。涵盖所有配置参数超出了本教程的范围。要了解更多信息，请阅读整个主要的`vsftpd`配置文件`/etc/vsftpd/vsftpd.conf`，因为它包含了许多有用的注释；或者，您可以阅读`man vsftpd.conf`手册。

那么我们从这次经历中学到了什么？

我们首先打开主要的`vsftpd`配置文件，然后使用`ftpd_banner`指令激活并自定义欢迎横幅。在下次成功登录时，您的用户应该看到您的新消息。接下来，当处理大量用户时，您可能想要考虑更改默认超时值并限制连接，以提高您的 FTP 服务的效率。

首先，我们更改了服务器的超时数值。`idle_session_timeout`为`600`秒将使在 10 分钟内不活跃（未执行 FTP 命令）的用户注销，而`data_connection_timeout`为`120`秒将在客户端数据传输停滞（未进展）20 分钟后终止连接。然后我们更改了连接限制。`local_max_rate`为`1000000`字节每秒将限制单个用户的数据传输速率大约为每秒一兆字节。`max_clients`值为`50`将告诉 FTP 服务器只允许 50 个并行用户访问系统，而`max_per_ip`为`2`只允许每个 IP 地址两个连接。

然后我们保存并关闭了文件。最后，我们展示了如何禁止用户使用我们的 FTP 服务。如果您想禁止特定用户使用 FTP 服务，则应将该用户的名称添加到`/etc/vsftpd/user_list`文件中。如果您需要随时重新启用该用户，只需通过从`/etc/vsftpd/user_list`中删除相关用户来反转之前的操作。

# 解决用户和文件传输问题

分析日志文件是解决 Linux 上各种问题或改进服务最重要的技术。在本教程中，您将学习如何配置和启用 vsftpd 的广泛日志记录功能，以帮助系统管理员在出现问题时或仅监视此服务的使用情况。

## 准备就绪

要完成本教程，您需要具备具有 root 权限的 CentOS 7 操作系统的有效安装，以及您选择的基于控制台的文本编辑器。预计您的服务器将使用静态 IP 地址，并且`vsftpd`已经安装了 chroot 监狱并正在运行。

## 如何做到这一点...

1.  要执行此操作，请以 root 身份登录，并键入以下命令以使用您喜欢的文本编辑器打开主配置文件：

    ```
    vi /etc/vsftpd/vsftpd.conf

    ```

1.  现在，将以下行添加到配置文件的末尾，以启用详细的日志记录功能：

    ```
    dual_log_enable=YES
    log_ftp_protocol=YES

    ```

1.  最后，重新启动`vsftpd`守护程序以应用更改：

    ```
    systemctl restart vsftpd

    ```

## 它是如何工作的...

在本教程中，我们展示了如何启用两个独立的日志记录机制：首先，`xferlog`日志文件将记录有关用户上传和下载的详细信息，然后是`vsftpd`日志文件，其中包含客户端和服务器之间的每个 FTP 协议事务，输出`vsftpd`可能的最详细的日志信息。

那么我们从这次经历中学到了什么？

在本教程中，我们打开了主要的`vsftpd`配置文件，并在文件末尾添加了两条指令。首先，`dual_log_enable`确保`xferlog`和`vsftpd`日志文件都将用于记录日志。之后，我们通过启用`log_ftp_protocol`来增加`vsftpd`日志文件的详细程度。

重新启动服务后，`/var/log/xferlog`和`/var/log/vsftdp.log`这两个日志文件将被创建并填充有用的 FTP 活动信息。现在，在我们打开文件之前，让我们创建一些 FTP 用户活动。使用`ftp`命令行工具在服务器上以任何 FTP 用户身份登录，并在`ftp>`提示符下发出以下 FTP 命令，将客户端上的随机文件上传到服务器：

```
put ~/.bash_profile bash_profile_test

```

现在，回到服务器上，检查`/var/log/xferlog`文件以查看有关上传文件的详细信息，并打开`/var/log/vsftpd.log`以查看其他用户活动（例如登录时间或其他用户发出的 FTP 命令）。

请注意，日志文件仅记录用户和 FTP 活动，并不用于调试`vsftpd`服务的问题，例如配置文件错误。要调试服务的一般问题，请使用`systemctl status vsftpd -l`或`journalctl -xn`。
