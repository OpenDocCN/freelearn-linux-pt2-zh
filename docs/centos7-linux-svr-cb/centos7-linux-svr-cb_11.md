# 第十一章：提供邮件服务

在本章中，我们将涵盖：

+   使用 Postfix 配置域内邮件服务

+   使用 Postfix

+   使用 Dovecot 投递邮件

+   使用 Fetchmail

# 简介

本章是一系列配方的集合，提供了实施和维护当今互联网上最古老、最多才多艺的技术之一的必要步骤。每个人都希望能够发送和接收电子邮件，本章提供了部署这种服务所需的起点，以便及时且高效地进行。

# 使用 Postfix 配置域内邮件服务

Postfix 是一个**邮件传输代理**（**MTA**），负责使用 SMTP 协议在邮件服务器之间传输电子邮件。Postfix 现在是 CentOS 7 上的默认 MTA。在这里，与其他大多数关键网络服务一样，其默认配置允许发送邮件，但不接受来自本地主机以外的任何主机的传入网络连接。如果你只需要一个本地 Linux 用户邮件系统，并且从 localhost 发送邮件到其他外部邮件服务器，这是有道理的。但如果你想为自己的私有网络和域运行自己的集中式邮件服务器，这就相当限制了。因此，本配方的目的是将 Postfix 设置为域内邮件服务，允许网络中的任何主机发送电子邮件，如果收件人是本地域内的有效电子邮件地址，则将其投递到邮件服务器上的正确邮箱。

## 准备工作

要完成本配方，你需要一个具有 root 权限的 CentOS 7 操作系统的安装，你选择的基于控制台的文本编辑器，以及连接到互联网以下载额外的软件包。你需要正确设置你的本地网络，并确保所有想要通过你的单域邮件服务器发送邮件的计算机都在同一个网络中，并且可以 ping 通这个服务器。此外，为任何邮件服务器正确设置系统时间也非常重要。在开始配置之前，请应用第二章，“配置系统”中的“使用 NTP 和 chrony 套件同步系统时钟”配方。最后，你需要为你的邮件服务器设置一个**完全限定域名**（**FQDN**）。请参考第二章，“配置系统”中的“设置你的主机名并解析网络”配方。预计你的服务器将使用静态 IP 地址，并且它将维护一个或多个系统用户帐户。还假设你将按照本章中出现的顺序逐个配方地工作。

## 如何操作...

Postfix 默认安装在所有 CentOS 7 版本上，并且应该处于运行状态。在我们的示例中，我们希望为我们的网络 192.168.1.0/24 构建一个中央邮件服务器，其本地域名为`centos7.home`。

1.  首先以 root 身份登录，并测试 Postfix 是否已经在本地工作，并且可以向您的系统用户发送本地邮件。输入以下命令向指定的 Linux 用户`<username>`发送邮件：

    ```
    echo "This is a testmail" | sendmail <username>

    ```

1.  在 CentOS 7 上，Postfix 也已经配置为无需对配置文件进行任何更改即可向外部电子邮件地址发送邮件（但仅从 localhost）。例如，您可以直接使用：

    ```
    echo "This is a testmail" | sendmail contact@example.com

    ```

    ### 注意

    如果您没有受信任的域和证书支持您的 Postfix 服务器，在大量垃圾邮件的时代，大多数外部邮件服务器将拒绝或将此类邮件直接放入垃圾邮件文件夹。

1.  要查看本地邮件是否已成功投递，请显示最新的邮件日志（按*Ctrl*+*C*退出日志）：

    ```
    tail -f /var/log/maillog

    ```

1.  接下来，检查我们的服务器是否有可用的 FQDN。这是强制性的，如果没有正确设置，请参阅第二章，*配置系统*以设置一个（在我们的示例中，这将输出名称`mailserver.centos7.home`）：

    ```
    hostname --fqdn

    ```

1.  现在在打开此文件之前创建主 Postfix 配置文件的备份副本：

    ```
    cp /etc/postfix/main.cf /etc/postfix/main.cf.BAK && vi /etc/
    postfix/main.cf

    ```

1.  首先，我们希望 Postfix 监听所有网络接口，而不仅仅是本地接口。激活或取消注释以下行（这意味着删除行首的`#`符号），该行以`inet_interfaces`开头，使其读取如下：

    ```
    inet_interfaces = all

    ```

1.  现在，在下面的一些行中，您会找到读取`inet_interfaces = localhost.`的行。通过在行首放置一个`#`符号来禁用它或注释掉它：

    ```
    # inet_interfaces = localhost

    ```

1.  接下来，我们需要设置邮件服务器的本地域名。例如，如果我们的邮件服务器的 FQDN 是`mailserver.centos7.home`，并且这个邮件服务器负责为整个私有`centos7.home`域投递邮件，那么域名将是（最好将其放在读取`#mydomain = domain.tld`的行下方）：

    ```
    mydomain = centos7.home

    ```

1.  考虑到此服务器可能成为域范围内的邮件服务器，您现在应该更新以下以`mydestination`开头的行，使其读取如下（例如，在`mydestination`部分，注释掉第一行`mydestination`并取消注释第二行）：

    ```
    mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

    ```

1.  接下来，我们需要指定相对于用户主目录的邮箱文件路径名。为此，向下滚动并找到以`home_mailbox`开头的行，并取消注释以下选项（删除行首的`#`符号）：

    ```
    home_mailbox = Maildir/

    ```

1.  保存并关闭文件。现在我们希望在防火墙中打开正确的 Postfix 服务器端口，以允许服务器接收 SMTP 连接：

    ```
    firewall-cmd --permanent --add-service=smtp && firewall-cmd --reload

    ```

1.  接下来，如下重新启动 Postfix 服务：

    ```
    systemctl restart postfix

    ```

1.  之后，登录到同一网络中的另一台计算机并安装**瑞士军刀 SMTP**（**swaks**），以远程测试我们的 Postfix 服务器连接。在 CentOS 上，输入以下内容（需要事先安装 EPEL 存储库）：

    ```
    yum install swaks

    ```

1.  现在，为了测试是否可以使用标准 SMTP 邮件端口 25 连接到我们新的 Postfix 服务器，我们的 Postfix 服务器运行在 IP 地址`192.168.1.100`，我们远程发送一封邮件给 Linux 系统用户`john`，他在我们的 Postfix 服务器上有一个系统用户账户：

    ```
    swaks --server 192.168.1.100 --to john@centos7.home

    ```

1.  Swaks 创建的输出应该给我们一个提示，表明邮件传输是否成功。例如（输出已被截断）：

    ```
    -> This is a test mailing
    <-  250 2.0.0 Ok: queued as D18EE52B38
     -> QUIT
    <-  221 2.0.0 Bye
    ```

1.  您还可以通过以用户`john`登录到 Postfix 服务器，然后检查并阅读您的本地邮箱收件箱，来测试最后一个命令是否成功，收件箱应该包含一个由 swaks 工具发送的测试邮件的文件（文件名在您的计算机上会有所不同），如下所示：

    ```
    ls ~/Maildir/new
    less ~/Maildir/new/14941584.Vfd02I1M246414.mailserver.centos7.home

    ```

## 它是如何工作的...

我们已经看到，Postfix 默认安装并运行在每个 CentOS 7 系统上，在其基本配置中，邮件服务器监听本地主机地址以接收邮件，因此您已经可以在服务器本地 Linux 系统用户之间发送本地邮件，而无需联系外部 MTA。它已经在运行，因为您的系统已经在使用它为多个本地服务，例如 crond 守护进程或发送关于安全漏洞的警告（例如，以非 sudo 用户身份运行`sudo`命令）。

在我们解释这个配方是如何工作之前，我们需要回顾一些关于 Postfix MTA 系统的更多基础知识。Postfix MTA 服务可以接收来自邮件客户端或其他远程 MTA 服务器的传入电子邮件，使用 SMTP 协议。如果传入的电子邮件是针对 MTA 服务器配置的最终目的地域（例如，一封发送到收件人地址`john@centos7.home`的邮件传入到配置的`centos7.home` Postfix MTA 服务器），它将把邮件投递到服务器上安装的本地邮箱（无论是在文件系统中还是在数据库系统中，如 MariaDB）。如果传入的邮件不是针对这个服务器，它将被转发（转发）到另一个 MTA。

请记住，这就是 Postfix 服务器所能做的一切，不多也不少：接收来自邮件客户端或其他 MTA 的传入 SMTP 连接，将邮件投递到服务器上的本地邮箱，并使用 SMTP 将邮件转发到其他 MTA。与普遍看法相反，Postfix 不能将其本地邮箱中的邮件传输给最终用户。这里我们需要另一种类型的 MTA，称为**投递代理**，它使用不同的邮件协议，如 IMAP 或 POP3。

在这个方法中，我们配置了我们的 Postfix 服务器，以便同一网络中的其他计算机和服务器也可以向我们的 Postfix 服务器发送邮件，这些邮件默认是被阻止的（默认情况下只有服务器本身可以发送邮件）。如果从我们网络中的另一台计算机发送的入站电子邮件，其收件人的电子邮件地址中的域名与我们的 Postfix 服务器的 FQDN 相同，那么它将被传递到由电子邮件的收件人部分定义的适当的本地邮箱；所有外部电子邮件地址都会被转发到一个外部 MTA。

那么我们从这次经历中学到了什么？

我们首先测试了是否可以向系统用户发送本地邮件。在这里，我们以 root 用户身份登录，并使用 Postfix 软件包中包含的 sendmail 程序向有效的本地系统用户发送邮件。每当你使用 sendmail 发送邮件时，你应该能够在`/var/log/maillog`文件中看到一些新行出现，该文件包含邮件的状态信息和其他重要的日志文本。如果你从`root`向用户`john`发送了一条消息，并且你的服务器的 FQDN 是`centos7.home`，那么追加到日志文件的新输出行应该包含一些内容，例如`from=<root@centos7.home>`，`to=<john@centos7.home>`，以及如果成功交付，则包含`status=sent`信息。如果没有出现这样的日志信息，请检查 Postfix 服务的运行状态。

随后，我们展示了我们服务器的 FQDN。正确设置这一点非常重要，因为这些信息将用于在连接到其他 MTA 或邮件客户端时对 Postfix 服务器进行身份验证。MTA 会检查其合作伙伴宣布的 FQDN，有些甚至会拒绝连接，如果未提供或与服务器的实际 DNS 域名不同。在我们的初步测试之后，我们开始编辑 Postfix 的主配置文件，首先对其进行了备份。如前所述，默认情况下，只有位于运行 Postfix 服务的同一服务器上的用户才能在它们之间发送邮件，因为服务器默认仅监听环回设备。因此，我们首先启用了 Postfix 以监听所有可用网络接口，使用`inet_interfaces = all`参数。这确保了我们网络中的所有客户端都可以连接到此服务器。接下来，我们使用`mydomain`参数设置了我们想要的 Postfix 域名。为了使 Postfix 在我们的网络中工作，此处定义的域名变量的值必须与我们的服务器网络的域名完全相同。之后，我们通过选择添加`$mydomain`参数的行来更改`mydestination`参数。这将定义我们的 Postfix 邮件服务器视为最终目的地的所有域。如果 Postfix 邮件服务器被配置为某个域的最终目的地，它将把消息投递到接收用户的本地邮箱，可以在`/var/spool/mail/<username>`（我们将在下一步更改此位置）而不是将邮件转发到其他 MTA（由于我们在示例中将`$mydomain`添加到了最终目的地的列表中，我们将投递所有发送到`centos7.home`域的邮件）。

在这里，您还需要记住，默认情况下，Postfix *信任*与 Postfix 服务器位于同一 IP 子网中的所有其他计算机（SMTP 客户端），以便通过我们的中央服务器向外部电子邮件地址发送邮件（转发邮件到外部 MTAs），这可能对您的网络策略来说过于宽松。由于电子邮件垃圾邮件是互联网上的一个持续问题，我们不希望允许任何用户滥用我们的邮件服务器发送垃圾邮件（开放中继邮件服务器会这样做；它从任何客户端接收任何内容并将其发送到任何邮件服务器），我们可以通过设置`mynetworks_style = host`来进一步提高安全性，该设置仅信任并允许本地主机向外部 MTAs 发送邮件。减少垃圾邮件风险的另一种方法可能是使用`mynetworks`参数，您可以在其中指定允许连接到我们的邮件服务器并通过它发送电子邮件的网络或 IP 地址；例如，`mynetworks = 127.0.0.0/8, 192.168.1.0/24`。要了解更多关于所有可用 Postfix 设置的信息，请参考 Postfix 配置参数手册，使用命令`man 5 postconf`。之后，我们更改了本地邮件应该存储的位置。默认情况下，所有传入邮件都发送到位于`/var/spool/mail/<username>`的中央邮箱空间。为了使本地用户能够在自己的主目录中接收邮件，我们为`home_mailbox`选项使用了`Maildir`参数，该参数将系统更改为将所有邮件发送到`/home/<username>/Maildir/`而不是。之后，我们在 firewalld 中打开了标准 SMTP 协议端口，使用 Postfix 用于与其他 MTAs 或发送传入邮件的邮件客户端通信的 SMPT 服务。

Postfix 已经配置为在启动时启动，但为了完成本节食谱，我们重新启动了 Postfix 服务，以便它接受新的配置设置。在这一阶段，配置 Postfix 的过程已经完成，但为了测试远程访问，我们需要登录到同一网络中的另一台计算机。在这里，我们安装了一个名为`swaks`的小型基于命令行的邮件客户端，它可以用来测试本地或远程 SMTP 服务器连接。我们通过向我们的远程 Postfix 邮件服务器发送邮件并提供收件人用户和我们的 SMTP 服务器的 IP 地址来运行测试。完成此操作后，您应该已收到测试消息，并且作为结果，您应该很高兴知道一切正常工作。但是，如果您碰巧遇到任何错误，则应参考位于`/var/log/maillog`的邮件服务器日志文件。

## 还有更多...

在本节食谱中，我们将更改您的电子邮件发件人地址，加密 SMTP 连接，并配置您的 BIND DNS 服务器以包含我们新邮件服务器的信息。

### 更改电子邮件的显示域名

如果一个 MTA 发送电子邮件，Postfix 默认会自动在发件人的电子邮件地址后附加主机名，除非另有明确说明，这是一个很好的功能，可以帮助你在本地网络中追踪哪台计算机发送了电子邮件（否则，如果你有多台计算机通过名为**root**的用户发送邮件，那么找到邮件的原始来源将会很困难）。通常在向远程 MTA 发送消息时，你不希望本地主机名出现在电子邮件中。

这里最好只保留域名。为了改变这一点，转到你想要从中发送邮件的 Postfix MTA，打开 Postfix 配置文件`/etc/postfix/main.cf`，并通过取消注释（删除行首的`#`符号）以下行来启用此功能以确定原始来源（之后重新启动 Postfix 服务）：

```
myorigin = $mydomain

```

### 使用 TLS-（SSL）加密进行 SMTP 通信

即使你在一个小型或私人环境中运行自己的 Postfix 服务器，也应该始终意识到，正常的 SMTP 流量将通过互联网以明文形式发送，这使得任何人都有可能嗅探通信。TLS 将允许我们在服务器和邮件客户端之间建立加密的 SMTP 连接，这意味着整个通信将被加密，第三方无法读取。为了做到这一点，如果你还没有购买官方 SSL 证书或为你的域名生成一些自签名证书，请先在这里创建一个（阅读第六章，*提供安全*中的*生成自签名证书*配方了解更多信息）。首先以 root 用户登录到你的服务器，然后转到标准证书位置：`/etc/pki/tls/certs`。接下来，创建一个包含证书及其嵌入的公钥以及私钥的 TLS/SSL 密钥对（输入你的 Postfix 的 FQDN 作为`通用名称`，例如，`mailserver.centos7.home`），为此输入`make postfix-server.pem`。之后，使用你喜欢的文本编辑器打开 Postfix 主配置文件`/etc/postfix/main.cf`，并在文件末尾添加以下行：

```
smtpd_tls_cert_file = /etc/pki/tls/certs/postfix-server.pem
smtpd_tls_key_file = $smtpd_tls_cert_file
smtpd_tls_security_level = may
smtp_tls_security_level = may
smtp_tls_loglevel = 1
smtpd_tls_loglevel = 1
```

然后保存并关闭此文件。请注意，将`smtpd_tls_security_level`设置为`may`将激活 TLS 加密（如果邮件客户端程序中可用），否则将使用未加密的连接。只有在您绝对确定所有向您的邮件服务器发送邮件的用户都支持此功能时，才应将此值设置为`encrypt`（这将强制在任何情况下都使用 SSL/TLS 加密）。如果任何发送者（外部 MTA 或邮件客户端）不支持此功能，连接将被拒绝。这意味着来自这些来源的电子邮件将不会被投递到您的本地邮箱。我们还为从我们的 Postfix 服务器到其他 MTAs 的可能的出站 SMTP 连接指定了 TLS 加密，使用`smtp_tls_security_level = may`。通过将 Postfix 的客户端和服务器模式 TLS 日志级别设置为`1`，我们可以获得更详细的输出，以便检查 TLS 连接是否正常工作。一些非常古老的邮件客户端使用古老的 465 端口进行 SSL/TLS 加密的 SMTP，而不是标准的 25 端口。

为了激活此功能，打开`/etc/postfix/master.cf`并搜索，然后取消注释（删除每行开头的`#`）以下行，使其读作：

```
smtps       inet   n       -       n       -       -       smtpd
-o syslog_name=postfix/smtps
-o smtpd_tls_wrappermode=yes
```

保存并关闭文件，然后重新启动 Postfix。接下来，我们需要在防火墙中打开 SMTPS 端口，以允许传入连接到我们的服务器。由于 CentOS 7 中没有可用的 SMTPS firewalld 规则，我们将首先使用`sed`实用程序创建我们自己的服务文件：

```
sed 's/25/465/g' /usr/lib/firewalld/services/smtp.xml | sed 's/Mail (SMTP)/Mail (SMTP) over SSL/g' > /etc/firewalld/services/smtps.xml
firewall-cmd --reload
firewall-cmd --permanent --add-service=smtps; firewall-cmd --reload

```

现在，您应该能够测试是否可以使用我们的`swaks`SMTP 命令行工具，使用`-tls`参数从远程计算机到运行在 IP 192.168.1.100 上的 Postfix 服务器建立 SMTPS 连接，例如`swaks --server 192.168.1.100 --to john@centos7.home -tls`。此命令行将测试 SMTP 服务器是否支持 TLS 加密（`STARTTLS`），并在任何原因不可用时退出并显示错误消息。正常工作的输出将如下所示（截断以仅向您显示最重要的行）：

```
 -> STARTTLS
<-  220 2.0.0 Ready to start TLS
=== TLS started with cipher TLSv1.2:ECDHE-RSA-AES128-GCM-SHA256:128
 ~> This is a test mailing
<~  250 2.0.0 Ok: queued as E36F652B38
```

然后，您还可以通过转到 Postfix 服务器上的主邮件日志文件并查找与上一步的 swaks 测试邮件相对应的以下行（您的输出将不同）来重新检查您的 TLS 设置：

```
Anonymous TLS connection established from unknown[192.168.1.22]: TLSv1.2 with cipher ECDHE-RSA-AES256-GCM-SHA384 (256/256 bits)
```

### 配置 BIND 以使用您的新邮件服务器

在我们的域内 Postfix 服务器安装和配置完成后，我们现在应该使用 DNS 服务器在我们的域中宣布这一新的邮件服务。请参考第八章，*使用 FTP*，了解如何设置和配置 BIND 服务器，特别是如果您还没有阅读过，请阅读有关**邮件交换器**（**MX**）记录的部分。然后在您的 BIND 正向和相应的反向区域文件中添加一个新的 MX 记录。在您的正向区域文件中，为我们的 Postfix 服务器添加以下行，IP 为`192.168.1.100`：

```
IN      MX      10      mailhost.centos7.home.
mailhost                   IN      A       192.168.1.100
```

在您的反向区域文件中，您可以添加以下行：

```
100                 IN  PTR         mailhost.centos7.local.
```

# 使用 Postfix

在前面的一个教程中，我们学习了如何安装和配置 Postfix 作为我们的域内电子邮件服务器。在处理电子邮件时，Linux 提供了许多不同的工具和程序，我们已经向您展示了如何通过`sendmail`程序以及`swaks`工具发送电子邮件。在本教程中，我们将向您展示如何使用 Unix 和 Linux 中最常用的邮件工具之一，名为`mailx`，它具有`sendmail`包中缺少的一些有用功能，用于发送邮件或阅读您的邮箱。

## 如何操作...

我们将从在我们的服务器上安装`mailx`包开始，该服务器运行我们的域内 Postfix 服务，因为默认情况下 CentOS 7 上不提供它。

1.  首先以 root 身份登录并键入以下命令：

    ```
    yum install mailx

    ```

1.  最简单的方法是使用`mailx`的标准输入模式，如下所示：

    ```
    echo "this is the mail body." | mail -s "subject" john@centos7.home

    ```

1.  您还可以从文本文件发送邮件。这在从 shell 脚本调用`mailx`命令时很有用，使用多个收件人，或者向电子邮件附加一些文件：

    ```
    cat ~/.bashrc | mail -s "Content of roots bashrc file" john
    echo "another mail body" | mail -s "body" john,paul@example.com,chris
    echo "this is the email body" | mailx -s "another testmail but with attachment" -a "/path/to/file1" -a "/path/to/another/file" john@gmail.com

    ```

### 将 mailx 连接到远程 MTA

`mailx`相对于`sendmail`程序的一大优势是，我们可以直接连接并与远程 MTA 邮件服务器通信。为了测试这一功能，请登录到另一台基于 Linux 的计算机，该计算机应与我们的 Postfix 服务器位于同一网络中，安装`mailx`包，并通过我们的 Postfix 服务器的 IP 地址`192.168.1.100`发送邮件（我们已经在之前的教程中打开了传入 SMTP 防火墙端口）。在我们的示例中，我们将向用户`john`发送一封本地邮件：

```
echo "This is the body" | mail -S smtp=192.168.1.100 -s "This is a remote test" -v john@centos7.home

```

### 从邮箱中阅读本地邮件

不仅`mailx`程序可以将电子邮件消息发送到任何 SMTP 服务器，当在 Postfix 服务器上本地启动时，它还提供了一个方便的邮件阅读器界面，用于您的本地邮箱。如果您使用`-f`指定用户邮箱运行邮件程序，程序将开始显示所有收件箱电子邮件。但请记住，`mailx`只能在程序在同一服务器上启动时读取本地邮箱（如果您想使用它远程访问您的邮箱，则需要安装 MTA 访问代理，如 Dovecot—稍后介绍—使用 POP3 或 IMAP）。例如，作为 Linux 系统用户`john`登录到 Postfix 服务器，然后，要使用您的用户的本地邮箱打开邮件阅读器，请键入：`mailx -f ~/Maildir`。

现在你将看到当前收件箱中所有邮件消息的列表。如果你想阅读特定的邮件，你需要输入它的编号并按下*回车*键。阅读后，你可以输入*d*后跟*回车*来删除它，或者输入*r*后跟*回车*来回复它。要返回到当前邮件消息概览屏幕，输入*z*后跟*回车*。如果你有超过一屏的邮件消息，输入*z-*（z 减号）后跟*回车*来返回一页。输入*x*后跟*回车*来退出程序。要了解更多信息，请参考`mailx`手册（`man mailx`）。

## 它是如何工作的...

在这个示例中，我们展示了如何安装和使用`mailx`，这是一个用于发送和阅读互联网邮件的程序。它基于一个名为 Berkely mail 的旧 Unix 邮件程序，并提供了 POSIX `mailx`命令的功能。它应该安装在每个严肃的 CentOS 7 服务器上，因为它比`sendmail`程序有一些优势，并且理解 IMAP、POP3 和 SMTP 协议（如果你需要一个更加用户友好的邮件阅读器和发送器，你可以查看 mutt。输入`yum install mutt`来安装它。然后输入`man mutt`来阅读它的手册）。

我们从这次经历中学到了什么？

我们首先在 Postfix 服务器上使用 YUM 包管理器安装了`mailx`包。它包含了`mailx`命令行程序，可以通过`mail`或`mailx`命令运行。之后，我们使用`-s`参数运行程序，该参数指定电子邮件主题，并且还需要一个收件人电子邮件地址作为参数，可以是外部地址或本地 Linux 系统用户名或邮件。如果没有其他设置，`mailx`会默认它与邮件服务器在同一台服务器上运行，因此它会隐式地将邮件发送到本地主机 MTA，在我们的例子中是 Postfix。此外，在最简单的形式中，`mailx`启动时进入交互模式，允许你在命令行手动输入消息正文字段。这对于快速编写测试邮件很有用，但在大多数情况下，你会通过管道将内容从另一个来源输入到`mailx`。这里我们展示了如何使用`echo`命令将字符串写入`mailx`的标准输入（STDIN），但你也可以使用`cat`命令将文件内容输入到`mailx`。

一个常用的例子是通过`cron`在特定预定时间点将某种文件输出或失败命令的日志文件内容发送给管理员用户或系统报告。之后，我们发现可以通过逗号分隔电子邮件地址的方式向多个收件人发送邮件，并向您展示了如何使用`-a`选项在邮件中附带附件。在下一节中，我们向您展示了如何使用`-S`选项设置内部选项（`variable=value`）向远程 SMTP 邮件服务器发送邮件。如果您未在 DNS 服务器上指定标准邮件服务器，或者需要测试远程邮件服务器，这是一个非常有用的功能。最后，在最后一节中，我们向您展示了如何使用`mailx`在您的 Postfix 服务器上阅读本地邮箱。它具有方便的浏览功能，可以阅读、删除和回复，并为您的本地邮箱进行高级电子邮件管理。您可以通过在`mailx`交互式会话中输入命令，然后按下*回车*键来实现这一点。请记住，如果您不喜欢这种方式浏览邮件，您还可以始终使用命令行工具（如`grep`、`less`等）在用户`~/Maildir`目录中阅读或过滤邮件。例如，要搜索所有新邮件中区分大小写的`PackPub.com`关键字，请输入`grep -i packtpub ~/Maildir/new`。

# 使用 Dovecot 发送邮件

在之前的操作中，您已经了解了如何将 Postfix 配置为域范围的邮件传输代理。正如我们在本章第一个操作中学到的，Postfix 只理解 SMTP 协议，并且在将消息从另一个 MTA 或邮件用户客户端传输到其他远程邮件服务器或将邮件存储到其本地邮箱方面做得非常出色。存储或转发邮件后，Postfix 的工作就结束了。Postfix 只能理解和使用 SMTP 协议，并且不能将消息发送到除 MTA 之外的任何其他地方。邮件消息的任何可能的收件人现在都需要使用 ssh 登录到运行 Postfix 服务的服务器，并查看其本地邮箱目录，或者使用`mailx`本地查看其消息，以定期检查是否有新邮件。这是非常不方便的，没有人会使用这样的系统。相反，用户选择从自己的工作站访问和阅读邮件，而不是从我们的 Postfix 服务器所在的位置。因此，开发了另一组 MTA，有时称为**访问代理**，其主要功能是将本地邮箱消息从运行 Postfix 守护程序的服务器同步或传输到外部邮件程序，用户可以在其中阅读这些消息。这些 MTA 系统使用不同于 SMTP 的协议，即 POP3 或 IMAP。其中一个 MTA 程序是 Dovecot。大多数专业服务器管理员都认为 Postfix 和 Dovecot 是完美的合作伙伴，本操作的目的是学习如何配置 Postfix 与 Dovecot 配合工作，以便为我们的邮箱提供基本的 POP3/IMAP 和 POP3/IMAP over SSL（POP3S/IMAPS）服务，为本地网络上的用户提供行业标准的电子邮件服务。

## 准备就绪

要完成此操作，您需要一个具有 root 权限的 CentOS 7 操作系统的工作安装，您选择的基于控制台的文本编辑器，以及连接到互联网以下载其他软件包。还假设您按照本章中出现的顺序逐个操作，因此预计已将 Postfix 配置为域范围的邮件传输代理。

### 注意

本操作作为设置本地网络上受信任用户的 POP3S/IMAPS 服务的指南。如果不采取额外的安全措施，则不适合在互联网上使用。

## 如何操作...

Dovecot 默认不安装，因此我们必须首先按照给出的步骤安装必要的软件包：

1.  首先，以 root 身份登录并输入以下命令：

    ```
    yum install dovecot

    ```

1.  安装后，通过输入以下命令在启动时启用 Dovecot 服务：

    ```
    systemctl enable dovecot

    ```

1.  现在，在创建备份副本后，使用您喜欢的文本编辑器打开 Dovecot 主配置文件，方法是输入：

    ```
    cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.BAK
    vi /etc/dovecot/dovecot.conf

    ```

1.  首先通过激活（删除行首的`#`符号）并修改以下行来确认我们要使用的`协议`，使其读作：

    ```
    protocols = pop3 imap imaps pop3s

    ```

1.  接下来，启用 Dovecot 监听所有网络接口而不仅仅是回环地址。搜索行`#listen = *`, `::`，然后将其修改为：

    ```
    listen = *

    ```

1.  现在以通常的方式保存并关闭文件，然后在您最喜欢的文本编辑器中打开`10-mail.conf`文件之前，先对其进行备份：

    ```
    cp /etc/dovecot/conf.d/10-mail.conf /etc/dovecot/conf.d/10-mail.conf.BAK
    vi /etc/dovecot/conf.d/10-mail.conf

    ```

1.  向下滚动并取消注释（删除`#`字符）以下行，使其读作：

    ```
    mail_location = maildir:~/Maildir

    ```

1.  再次，在创建备份副本之前，以通常的方式保存并关闭文件，然后在您最喜欢的文本编辑器中打开以下文件：

    ```
    cp /etc/dovecot/conf.d/20-pop3.conf /etc/dovecot/conf.d/20-pop3.conf.BAK
    vi /etc/dovecot/conf.d/20-pop3.conf

    ```

1.  首先取消注释以下行：

    ```
    pop3_uidl_format = %08Xu%08Xv

    ```

1.  现在向下滚动并修改以下行：

    ```
    pop3_client_workarounds = outlook-no-nuls oe-ns-eoh

    ```

1.  以通常的方式保存并关闭文件。现在我们将允许纯文本登录。为此，在打开以下文件之前先进行备份：

    ```
    cp /etc/dovecot/conf.d/10-auth.conf /etc/dovecot/conf.d/10-auth.conf.BAK
    vi /etc/dovecot/conf.d/10-auth.conf

    ```

1.  将行`#disable_plaintext_auth = yes`更改为：

    ```
    disable_plaintext_auth = no

    ```

1.  保存并关闭文件。在我们的最终配置设置中，我们将告诉 Dovecot 使用我们的自签名服务器证书。只需使用本章另一食谱中的 Postfix 证书或创建一个新的（否则跳过此步骤）：

    ```
    cd /etc/pki/tls/certs; make postfix-server.pem

    ```

1.  在备份文件后打开 Dovecot 的标准 SSL 配置文件：

    ```
    cp /etc/dovecot/conf.d/10-ssl.conf /etc/dovecot/conf.d/10-ssl.conf.BAK
    vi /etc/dovecot/conf.d/10-ssl.conf

    ```

1.  现在将以下行（`ssl = required`）更改为读作：

    ```
    ssl = yes

    ```

1.  现在将以下两行更改为指向您服务器自己的证书路径：

    ```
    ssl_cert = < /etc/pki/tls/certs/postfix-server.pem
    ssl_key = </etc/pki/tls/certs/postfix-server.pem

    ```

1.  保存并关闭此文件。接下来，在我们的防火墙中启用 IMAP、IMAPS、POP3 和 POP3S 端口，以允许在相应端口上的传入连接。对于 POP3 和 IMAP，我们需要指定自己的`firewalld`服务文件，因为它们在 CentOS 7 中默认不可用：

    ```
    sed 's/995/110/g' /usr/lib/firewalld/services/pop3s.xml | sed 's/ over SSL//g' > /etc/firewalld/services/pop3.xml
    sed 's/993/143/g' /usr/lib/firewalld/services/imaps.xml | sed 's/ over SSL//g' > /etc/firewalld/services/imap.xml
    firewall-cmd --reload
    for s in pop3 imap pop3s imaps; do firewall-cmd --permanent --add-service=$s; done;firewall-cmd --reload

    ```

1.  现在在启动 Dovecot 服务之前保存并关闭文件：

    ```
    systemctl start dovecot

    ```

1.  最后，为了测试我们的新 POP3/SMTP 网络服务，只需在同一网络中的另一台计算机上登录并运行以下命令，使用`mailx`访问远程 Postfix 服务器上的本地邮箱，该服务器由 Dovecot 提供，使用不同的访问代理协议。在我们的示例中，我们希望使用 IP `192.168.1.100`远程访问系统用户`john`在 Postfix 服务器上的本地邮箱（要登录到 john 的账户，您需要他的 Linux 用户密码）：

    ```
    mailx -f pop3://john@192.168.1.100
    mailx -f imap://john@192.168.1.100

    ```

1.  接下来，为了测试安全连接，使用以下命令并在确认证书是自签名且不受信任时输入`yes`：

    ```
    mailx -v -S nss-config-dir=/etc/pki/nssdb -f pop3s://john@192.168.1.100
    mailx -v -S nss-config-dir=/etc/pki/nssdb -f imaps://john@192.168.1.100

    ```

1.  对于所有四个命令，您应该看到用户`john`邮箱的正常`mailx`收件箱视图，其中包含所有邮件消息，就像您在 Postfix 服务器上本地运行`mailx`命令以阅读本地邮件一样。

## 工作原理...

成功完成此配方后，您刚刚为网络中的所有有效服务器用户创建了一个基本的 POP3/SMTP 服务（带或不带 SSL 加密），该服务将从 Postfix 服务器向客户端的电子邮件程序传递本地邮件。每个本地系统用户都可以直接进行身份验证并连接到邮件服务器，并远程获取他们的邮件。当然，还有很多可以做的事情来增强服务，但现在您可以让所有本地系统账户持有者配置他们最喜欢的电子邮件桌面软件，使用您的服务器发送和接收电子邮件消息。

### 注意

与 IMAP 不同，POP3 将邮件从服务器下载到本地机器上，并在之后删除它们，而 IMAP 则与邮件服务器同步您的邮件，而不删除它们。

那么我们从这次经历中学到了什么？

我们首先通过安装 Dovecot 来开始这个配置过程。完成这一步后，我们启用了 Dovecot 在启动时运行，并接着对一系列配置文件进行了一些简短的修改。首先，我们需要确定在`/etc/dovecot/dovecot.cf`文件中的 Dovecot 配置文件中将使用哪种协议：IMAP、POP3、IMAPS 和 POP3S。与其他大多数基本的网络服务一样，安装后它们仅监听回环设备，因此我们启用了 Dovecot 以监听服务器上安装的所有网络接口。在`10-mail.conf`文件中，我们确认了 Dovecot 的邮箱目录位置（使用`mail_location`指令），这是 Postfix 在接收邮件时将它们放入的位置，以便 Dovecot 可以在这里找到它们并拾取。接下来，我们在`20-pop3.conf`文件中打开了 POP3 协议，添加了一个与各种电子邮件客户端（例如 Outlook 客户端）相关的修复，使用了`pop3_uidl_format`和`pop3_client_workarounds`指令。最后，我们通过在`/etc/dovecot/conf.d/10-auth.conf`文件中进行几处更改，启用了纯文本授权。请记住，在没有 SSL 加密的情况下使用纯文本授权与 POP3 或 IMAP 被认为是不安全的，但由于我们专注于局域网（为一组可信的服务器用户），我们不一定将此视为风险。之后，我们通过将`10-ssl.conf`文件中的`ssl`指令指向一些现有的自签名服务器证书，启用了 POP3 和 IMAP 的 SSL（POP3S 和 IMAPS）。在这里，我们将`ssl = required`更改为`ssl=yes`，以不强制客户端连接到 Dovecot 服务时使用 SSL 加密，因为我们确实希望给用户选择启用加密认证的选项，但如果他喜欢的话，不要将其作为强制性的，特别是对于较旧的客户端。之后，为了让我们的 Dovecot 服务可以从网络中的其他计算机访问，我们必须启用四个端口以允许 POP3、IMAP、POP3S 和 IMAPS，即 993、995、110、143，通过使用预定义的`firewalld`服务文件并为我们自己创建缺失的 IMAP 和 POP3 文件。稍后，我们启动了 Dovecot 服务，并使用`mailx`命令远程测试了我们新的 POP3/IMAP 服务器。通过提供一个`-f`文件参数，我们能够指定我们的协议和位置。对于使用 SSL 连接，我们需要提供一个额外的`nss-config-dir`选项，指向我们在 CentOS 7 中存储证书的本地网络安全性服务数据库。

记住，如果你遇到任何错误，你应该始终参考位于`/var/log/maillog`的日志文件。在真正的企业环境中不应使用纯文本授权，而应首选 POP3/IMAP 的 SSL。

## 还有更多...

在主配方中，你被展示了如何安装 Dovecot 以允许具有系统账户的受信任本地系统用户发送和接收电子邮件。这些用户将能够使用他们现有的用户名作为电子邮件地址的基础，但通过做一些改进，你可以快速启用别名，这是一种为现有用户定义替代电子邮件地址的方式。

要开始构建用户别名列表，你应该首先在你的首选文本编辑器中打开以下文件：

```
vi /etc/aliases

```

现在将你的新身份添加到文件末尾，其中`<username>`将是实际系统账户的名称：

```
#users aliases for mail
newusernamea:    <username>
newusernameb:    <username>

```

例如，如果你有一个名为`john`的用户，目前仅接受来自`john@centos7.home`的电子邮件，但你想为`john`创建一个名为`johnwayne@centos7.home`的新别名，你将这样写：

```
johnwayne:    john

```

为所有别名重复此操作，但完成后记得以通常的方式保存并关闭文件，然后运行以下命令：`newaliases`。

### 设置电子邮件软件

市场上有大量的电子邮件客户端，到目前为止，你将希望开始设置你的本地用户能够发送和接收电子邮件。这并不复杂，但为了有一个良好的起点，你将希望考虑以下原则。电子邮件地址的格式将是`system_username@domain-name.home`。

传入的 POP3 设置将类似于以下内容：

```
mailserver.centos7.home, Port 110
Username: system_username
Connection Security: None
Authentication: Password/None
```

对于 POP3S，只需将端口更改为`995`并使用`连接安全性`：`SSL/TLS`。对于 IMAP，只需将端口更改为`143`，对于 IMAPS 使用端口`993`和`连接安全性`：`SSL/TLS`。

外发的 SMTP 设置将类似于以下内容：

```
mailserver.centos7.home, Port 25
Username: system_username
Connection Security: None
Authentication: None
```

# 使用 Fetchmail

到目前为止，在本章中，我们已经向您展示了两种不同形式的 MTA。首先，我们向您介绍了 Postfix MTA，这是一种用于将电子邮件从邮件客户端路由到邮件服务器或邮件服务器之间，并使用 SMTP 协议将它们传递到邮件服务器上的本地邮箱的传输代理。然后，我们向您展示了另一种有时称为访问代理的 MTA，Dovecot 程序可以用于此目的。这将从本地 Postfix 邮箱向任何远程邮件客户端程序提供邮件，使用 POP3 或 IMAP 协议。现在，我们将向您介绍第三种 MTA，可以称为检索代理，并解释我们将使用 Fetchmail 程序的目的。如今，几乎每个人都有多个电子邮件帐户，来自一个或多个不同的邮件提供商，如果您需要登录所有这些不同的 Webmail 站点或使用邮件程序中的不同帐户，则可能难以维护。这就是 Fetchmail 发挥作用的地方。它是一个程序，在您的域内 Postfix 邮件服务器上运行，可以从您所有不同的邮件提供商那里检索所有不同的电子邮件，并将它们传递到 Postfix MTA 的本地用户邮箱中。一旦它们存储在适当的位置，用户就可以通过 Dovecot 访问代理提供的通常方式通过 POP3 或 IMAP 访问所有这些邮件。在本操作中，我们将向您展示如何安装并将 Fetchmail 集成到运行 Postfix MTA 的服务器中。

## 准备工作

要完成此操作，您需要具备 CentOS 7 操作系统的有效安装，拥有 root 权限，选择一个基于控制台的文本编辑器，以及连接到互联网以便下载额外的软件包。假设您是按照本章节中出现的顺序逐个进行操作，因此预计 Postfix 已配置为域内邮件传输代理（MTA），Dovecot 已安装以提供 POP3/IMAP 邮件访问服务。为了在本操作中测试 Fetchmail，我们还需要注册一些外部电子邮件地址：您需要外部电子邮件服务器地址的名称和您电子邮件提供商的端口，以及您的用户登录凭据。通常，您可以在电子邮件提供商网站的常见问题（FAQ）部分找到这些信息。此外，对于某些电子邮件地址，您需要在电子邮件设置中首先启用 POP3 或 IMAP，然后才能使用 Fetchmail。

## 操作步骤...

由于 Fetchmail 并非默认安装，因此我们必须首先安装必要的软件包。请按照以下步骤操作：

1.  首先，登录运行 Postfix 服务器的邮件服务器并输入：

    ```
    yum install fetchmail

    ```

1.  安装完成后，我们将登录到我们希望为其实现 Fetchmail 下载外部邮件的系统用户账户，在我们的例子中是系统用户`john`：`su - john`。现在让我们使用外部电子邮件地址配置 Fetchmail。如果您的邮件提供商名为`mailhost.com`，它在`pop.mailhost.com`上运行 POP3 服务器，在`imap.mailhost.com`上运行 IMAP，用户名为`<user-name>`，这里（请替换您自己的值）是一个示例命令行，用于测试与该提供商的连接并从中获取邮件：

    ```
    fetchmail pop.mailhost.com -p pop3 -u <user-name> -k -v

    ```

1.  如果您想使用同一提供商的 IMAP：

    ```
    fetchmail imap.mailhost.com -p IMAP -u <user-name> -v

    ```

1.  如果 Fetchmail 命令成功，所有新消息将从服务器下载到您的用户账户的本地邮箱中。

## 它是如何工作的...

在本教程中，我们向您展示了如何安装和测试 Fetchmail，它为我们的 Postfix 服务器上拥有本地邮箱的任何用户账户提供了自动邮件检索功能。因此，对于使用 POP3 或 IMAP 连接到邮件服务器的客户端，通过这种方式获取的邮件看起来就像正常的入站电子邮件。Fetchmail 常用于将所有不同的邮件账户合并到一个账户中，但如果您的邮件提供商没有良好的病毒或垃圾邮件过滤器，您也可以使用它。您从主机邮件服务器下载邮件，然后使用 SpamAssassin 或 ClamAV 等工具处理邮件，再将邮件发送给客户。

我们从这次经历中学到了什么？

我们首先通过安装 YUM 包来开始 Fetchmail 的配置。由于我们希望为名为`john`的系统用户邮箱设置 Fetchmail，接下来我们以该用户身份登录。之后，我们通过运行一个简单的命令行来测试 Fetchmail 程序，以从单个邮件提供商处获取邮件。如前所述，为了成功登录到外部邮件提供商，您需要知道服务器的准确登录信息（服务器地址、端口、用户名和密码，以及协议类型）才能使用 Fetchmail。

请记住，虽然一些电子邮件提供商允许用户决定是否使用 SSL 进行安全连接，但像[gmail.com](http://gmail.com)这样的主机仅允许安全连接。这意味着，如果主要电子邮件提供商不支持不带 SSL 连接的 POP3/IMAP 访问，本教程中所示的示例命令可能会失败。请继续阅读下一节，了解如何使用带有 SSL POP3/IMAP 加密的 Fetchmail。

如果您的邮件提供商同时提供 SSL 加密，您应该始终优先选择 SSL 加密。此外，一些提供商如[gmail.com](http://gmail.com)默认只允许用户通过 webmail 使用其服务，并禁用 POP3/IMAP 服务功能；您需要在提供商网站上的账户设置中启用它们（稍后说明）。

我们使用`-p`参数指定使用 fetchmail 命令的邮件协议。使用`-u`参数，我们指定了登录邮件服务器时使用的用户标识，这完全取决于我们的电子邮件提供商。对于 POP3，我们应用了`-k`标志，以确保电子邮件仅从服务器获取，但永远不会被删除（这是使用 POP3 协议时的默认行为）。最后，我们使用`-v`来使输出更加详细，并为我们简单的测试提供更多信息。如果您的电子邮件提供商支持 SSL，您还需要在 Fetchmail 命令中添加一个`-ssl`标志以及邮件服务器的根证书（请参阅下一节了解更多信息）。如果您运行前面的命令，Fetchmail 将立即开始询问服务器上的收件箱中的任何邮件，并将任何邮件下载到用户的本地邮箱。

## 还有更多...

在本节中，我们将向您展示如何配置 Fetchmail，以便使用 POP3S、IMAPS 以及 POP3 和 IMAP 协议从一些实际生活中的邮件提供商下载所有电子邮件到本地邮箱的 Postfix 服务器上，使用配置文件。最后，我们将向您展示如何自动化 Fetchmail 过程。

### 配置 Fetchmail 与 gmail.com 和 outlook.com 电子邮件账户

在这里，我们将配置 Fetchmail 将从以下不同的外部邮件账户下载：流行的[gmail.com](http://gmail.com)和[outlook.com](http://outlook.com)电子邮件提供商以及假设的`my-email-server.com`。

正如我们在主要配方中学到的，Fetchmail 默认通过命令行处理配置选项，这不应是您首选的使用 Fetchmail 自动从不同邮件账户下载邮件的方式。通常情况下，Fetchmail 应该作为服务在后台以守护模式运行，或者使用`cron`作业在启动时运行，并按照特定的时间间隔轮询定义在特殊配置文件中的一系列邮件服务器。通过这种方式，您可以方便地配置多个邮件服务器和一长串其他选项。

### 注意

在撰写本书时，为了使[gmail.com](http://gmail.com)与 Fetchmail 协同工作，您需要使用您的用户账户登录[gmail.com](http://gmail.com)网站，并首先在**转发和 POP/IMAP**中启用 IMAP。此外，在**我的账户**中的**登录与安全**下启用**允许安全性较低的应用程序**。对于[outlook.com](http://outlook.com)，登录到您的邮件账户网页，然后点击**选项**，再次点击**选项**，接着点击**使用 POP 连接设备和应用**，然后点击**启用 POP**。

无论是[outlook.com](http://outlook.com)还是[gmail.com](http://gmail.com)都使用安全的 POP3S 和 IMAPS 协议，因此您需要首先在您的 Fetchmail 服务器上下载并安装他们用于签署其 SSL 证书的根证书，以便能够使用他们的服务。在这里，我们可以安装 Mozilla CA 证书包，该证书包由 Mozilla 基金会编译，并包括所有主要网站和服务使用的最常用的根服务器证书，例如我们的邮件提供商使用的证书。对于[gmail.com](http://gmail.com)，我们需要 Equifax Secure Certificate Authority 根证书，而对于[outlook.com](http://outlook.com)，我们需要 Globalsign 的根服务器证书。Fetchmail 需要这些根证书来验证从电子邮件服务器下载的任何其他 SSL 证书的有效性。以 root 身份登录到您的 Postfix 服务器并安装以下软件包：

```
yum install ca-certificates

```

之后，以 Linux 系统用户身份登录，例如，`john`，我们将为其创建一个新的 Fetchmail 配置文件，并且他已经在我们的服务器上拥有位于其主目录下的`~/Maildir`的本地 Postfix 邮箱目录。在配置 Fetchmail 配置文件中的任何账户之前，您应该始终首先使用 Fetchmail 命令行测试到特定账户的连接和身份验证是否正常，如前一个配方所示。为了测试我们不同邮件提供商的账户，我们需要三个不同的命令行调用。为了测试您的提供商是否使用 SSL 加密，您需要`–ssl`标志；对于不允许非 SSL 连接的邮件提供商，典型的输出可能是：

```
Fetchmail: SSL connection failed.
Fetchmail: socket error while fetching from <userid>@<mailserver>
Fetchmail: Query status=2 (SOCKET)
```

如果您的 google 和 outlook 用户名是`johndoe`，在两个邮件提供商处进行测试，对于使用 IMAPS 协议尝试与 google 进行测试（在提示时输入您的电子邮件用户的密码）：

```
fetchmail imap.gmail.com -p IMAP --ssl -u johndoe@gmail.com -k -v
```

如果登录成功，输出应该类似于（已截断）：

```
Fetchmail: IMAP< A0002 OK johndoe@gmail.com authenticated (Success)
9 messages (2 seen) for johndoe at imap.gmail.com.
Fetchmail: IMAP> A0005 FETCH 1:9 RFC822.SIZE
```

对于使用 POP3S 测试[outlook.com](http://outlook.com)，请使用：

```
fetchmail pop-mail.outlook.com -p POP3 --ssl -u johndoe@outlook.com -k -v
```

成功后，输出应该类似于（已截断）：

```
Fetchmail: POP3> USER johndoe@outlook.com
Fetchmail: POP3< +OK password required
Fetchmail: POP3< +OK mailbox has 1 messages
```

对于我们在`my-email-server.com`上的第三个假设电子邮件账户，我们将使用不带 SSL 的 POP3 或 IMAP 来测试它，使用我们的账户：

```
fetchmail pop3.my-email-server.com -p POP3 -u johndoe -k -v
fetchmail imap.my-email-server.com -p IMAP -u johndoe  -v
```

您还应该检查从外部提供商获取的所有邮件是否已正确下载。使用`mailx`命令查看系统用户的本地邮箱（`mailx -f ~/Maildir`）。在我们成功验证 Fetchmail 能够连接到服务器并获取一些邮件之后，我们现在可以在系统用户的家目录中创建一个本地 Fetchmail 配置文件，以便自动化此过程并为多个邮件地址进行配置。首先使用`vi ~/.fetchmailrc`打开一个新文件。请记住，可以在命令行上放置的所有命令也可以在配置文件中使用，只是名称略有不同（并且更多）。现在输入以下内容（将`john`替换为您的实际 Linux 系统用户，`johndoe`替换为您的电子邮件用户账户名，`secretpass`替换为该账户的实际邮件密码）：

```
set postmaster "john"
set logfile fetchmail.log
poll imap.gmail.com with proto IMAP
user 'johndoe@gmail.com' there with password 'secretpass' is john here
ssl
fetchall
poll pop-mail.outlook.com with proto POP3
user 'johndoe@outlook.com' there with password 'secretpass' is john here
ssl
fetchall
poll pop3.my-email-server.com with proto POP3
user 'johndoe@my-email-server.com' there with password 'secretpass' is john here
fetchall
```

保存并关闭此文件。在此文件中，我们使用了以下重要命令：

+   `postmaster`：定义将接收 Fetchmail 遇到问题时的所有警告或错误邮件的本地 Linux 用户。

+   `logfile`：定义一个日志文件名，这对于我们监督和调试 Fetchmail 输出非常有帮助，当它在后台长时间连续运行时。

+   `poll`部分：指定从特定邮件提供商下载邮件。对于每个邮件账户，您将定义一个这样的轮询部分。如您所见，语法与我们测试单个连接时在命令行上使用的语法非常相似。使用`proto`定义`mail`协议，`user`是邮件账户的登录用户，`password`是账户的登录密码，使用`is <username> here`参数指定此邮件账户绑定到哪个本地系统用户账户。对于 SSL 连接，您需要`ssl`标志，我们指定了`fetchall`参数以确保我们也下载电子邮件提供商标记为`read`的所有电子邮件消息，否则 Fetchmail 不会下载已读邮件。

接下来更改`.fetchmailrc`文件的权限，因为它包含密码，因此不应被我们自己的用户以外的任何人读取：

```
chmod 600 ~/.fetchmailrc

```

最后，我们使用配置文件中的设置执行 Fetchmail。为了测试，我们将使用一个非常详细的参数：`fetchmail -vvvv`。现在，来自您所有不同电子邮件提供商的所有新邮件都应该被获取，因此之后您应该检查输出，看看每个服务器是否都已准备好并且可以像我们之前在命令行测试中进行的单个测试一样被轮询。所有新邮件都应该被下载到本地邮箱，因此为了阅读本地邮件，您可以像往常一样使用`mailx`命令，例如：`mail -f ~/Maildir`。

### 自动化 Fetchmail

正如刚才所说，我们现在可以随时手动启动轮询过程，只需在命令行中输入`fetchmail`即可。这将轮询并从我们在新配置文件中指定的邮件服务器获取所有新邮件，然后在处理每个条目一次后退出程序。现在仍然缺少的是一个机制，以特定间隔持续查询我们的邮件服务器，每当可以获取新邮件时更新我们的邮箱。您可以使用两种方法。要么将`fetchmail`命令作为 cron 作业运行，要么作为替代方案，您可以以守护进程模式启动 Fetchmail（在您的`.fetchmailrc`配置文件中使用参数`set daemon`激活它。）并将其置于后台。这样，Fetchmail 将始终运行，并在给定的时间点唤醒，开始轮询，直到处理完所有内容，然后回到睡眠状态，直到下一个间隔到达。

由于两种方法基本相同，这里我们将向您展示如何将 Fetchmail 作为 cron 作业运行，这设置起来要容易得多，因为我们不必创建一些自定义的 systemd 服务文件（目前在 CentOS 7 中没有现成的`fetchmail systemd`服务）。对于每个系统用户（例如，`john`），他们都有一个`fetchmail`配置文件，要每 10 分钟启动一次电子邮件服务器轮询过程，请输入以下命令一次以注册 cron 作业：

```
crontab -l | { cat; echo "*/10 * * * * /usr/bin/fetchmail &> /dev/null
"; } | crontab -

```

### 注意

不要将 Fetchmail 轮询周期设置得短于每 5 分钟一次，否则一些邮件提供商可能会阻止或禁止您，因为它只是使他们的系统过载。
