# 第十二章：提供 Web 服务

在本章中，我们将介绍以下内容：

+   安装 Apache 并提供网页服务

+   启用系统用户并构建发布目录

+   实施基于名称的托管

+   使用 Perl 和 Ruby 实现 CGI

+   安装、配置和测试 PHP

+   保护 Apache

+   使用安全套接字层（SSL）设置 HTTPS

# 引言

本章是一系列食谱的集合，提供了为网页提供服务的必要步骤。从安装 Web 服务器到通过 SSL 提供动态页面，本章提供了在任何时间和任何地点实施行业标准托管解决方案所需的起点。

# 安装 Apache 并提供网页服务

在本食谱中，我们将学习如何安装和配置 Apache Web 服务器以启用静态网页服务。Apache 是世界上最受欢迎的开源 Web 服务器之一。它作为后端运行着超过一半的互联网网站，并可用于提供静态和动态网页。通常被称为`httpd`，它支持广泛的功能。本食谱的目的是向您展示如何轻松地使用 YUM 包管理器进行安装，以便您可以保持服务器最新的安全更新。Apache 2.4 在 CentOS 7 上可用。

## 准备就绪

要完成本食谱，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，您选择的基于控制台的文本编辑器，以及连接到互联网以下载其他软件包的能力。预计您的服务器将使用静态 IP 地址和主机名。

## 如何操作...

Apache 默认未安装，因此我们将从使用 YUM 包管理器安装必要的软件包开始。

1.  首先，以 root 身份登录并输入以下命令：

    ```
    yum install httpd

    ```

1.  通过输入以下内容创建主页：

    ```
    vi /var/www/html/index.html

    ```

1.  现在添加所需的 HTML。您可以使用以下代码作为起点，但预计您会希望对其进行修改以满足您的需求：

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head><title>Welcome to my new web server</title></head>
    <body><h1>Welcome to my new web server</h1>
    <p>Lorem ipsum dolor sit amet, adipiscing elit.</p></body>
    </html>
    ```

1.  您现在可以使用以下命令删除 Apache 2 测试页面：

    ```
    rm -f /etc/httpd/conf.d/welcome.conf

    ```

1.  完成这些步骤后，我们现在将考虑为基本使用配置`httpd`服务的需要。为此，打开您最喜欢的文本编辑器中的`httpd`配置文件，输入（在您已经备份文件之后）：

    ```
    cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.BAK
    vi /etc/httpd/conf/httpd.conf

    ```

1.  现在向下滚动找到行`ServerAdmin root@localhost`。设置此值的传统方法基于使用 webmaster 身份，因此只需修改电子邮件地址以反映更符合您自己需求的内容。例如，如果您的服务器的域名是`www.centos7.home`，那么您的条目将与此类似：

    ```
    ServerAdmin webmaster@centos7.home

    ```

1.  现在向下滚动几行，找到以下`ServerName`指令：`#ServerName www.example.com:80`。取消注释此行（即删除其前面的#符号），并将值`www.example.com`替换为更适合您自己需求的值。例如，如果您的服务器域名为`www.centos7.home`，则您的条目将如下所示：

    ```
    ServerName www.centos7.home:80

    ```

1.  接下来，我们将稍微扩展一下`DirectoryIndex`指令。找到`DirectoryIndex index.html`这一行，它是`<IfModule dir_module>`块的一部分，然后将其更改为：

    ```
    DirectoryIndex index.html index.htm

    ```

1.  保存并关闭文件，然后键入以下命令以测试配置文件：

    ```
    apachectl configtest

    ```

1.  接下来，让我们通过允许传入的`http`连接（默认为端口 80）到服务器来配置 Web 服务器的防火墙：

    ```
    firewall-cmd --permanent --add-service http && firewall-cmd --reload 

    ```

1.  现在继续设置`httpd`服务以在启动时启动并启动该服务：

    ```
    systemctl enable httpd && systemctl start httpd

    ```

1.  现在，您可以从与您的 Web 服务器位于同一网络中的任何计算机上测试`httpd`（两个系统应该能够相互看到并进行 ping 操作），通过将以下 URL 中的`XXX.XXX.XXX.XXX`替换为您的服务器 IP 地址，以查看我们创建的自定义 Apache 测试页面：

    ```
    http://XXX.XXX.XXX.XXX.

    ```

1.  或者，如果您没有 Web 浏览器，可以使用`curl`从网络中的任何计算机上获取我们的测试页面，以检查 Apache 是否正在运行：

    ```
    curl http://XXX.XXX.XXX

    ```

## 它是如何工作的...

Apache 是一个软件包，它使您能够发布和提供 Web 页面，并且更常被称为`httpd`、Apache2 或简称为 Apache。本食谱的目的是向您展示 CentOS 如何轻松地让您开始创建您的第一个网站。

那么我们从这次经历中学到了什么？

我们首先通过 YUM 包管理器安装了名为`httpd`的 Apache，并了解到在 CentOS 7 上，默认提供静态 HTML 的位置是`/var/www/html`。因此，我们的首要任务是创建一个合适的欢迎页面，我们将其放置在`/var/www/html/index.html`。我们使用了一个基本的 HTML 模板来帮助您起步，并期望您会希望自定义这个页面的外观和感觉。接着，我们删除了位于`/etc/httpd/conf.d/welcome.conf`的默认 Apache 2 欢迎页面。随后，下一步是在备份`httpd.conf`配置文件后，使用我们喜欢的文本编辑器打开它，以便在出现问题时可以恢复更改。首先，我们定义了服务器的电子邮件地址和服务器名称，这些信息通常出现在服务器生成的网页的错误消息中；因此，它应该反映您的域名。接下来，我们调整了`DirectoryIndex`指令，该指令定义了在请求目录时将首先发送给浏览器的文件。通常，人们请求的不是特定的网页而是目录。例如，如果您浏览到`www.example.com`，您请求的是一个目录，而`www.example.com/welcome.html`是一个特定的网页。默认情况下，Apache 发送请求目录中的`index.html`，但我们扩展了这一点，因为许多网站使用`.htm`扩展名。最后，我们以通常的方式保存并关闭了`httpd`配置文件，然后使用`apachectl configtest`命令检查 Apache 配置文件是否存在任何错误。这应该会打印出一条`Syntax OK`消息，然后我们可以启用`httpd`服务在启动时自动启动。我们必须在 firewalld 中打开标准的 HTTP 端口 80，以允许对服务器的传入 HTTP 请求，最后我们启动了`httpd`服务。请记住，如果配置文件已被更改，您也可以始终在不完全重启服务的情况下重新加载 Apache 的配置文件，方法是使用：`systemctl reload httpd`。完成这些步骤后，只需从同一网络中的另一台计算机打开浏览器，并选择一种查看我们新 Apache 启动页面的方法。您可以使用服务器的 IP 地址（例如，`http://192.168.1.100`），而那些支持主机名的人可以输入主机名（例如，`http://www.centos7.home`）。Apache 的访问和错误日志文件可以在`/var/log/httpd`中找到。要实时查看谁正在访问您的 Web 服务器，请打开`/var/log/httpd/access_log`；要查看所有错误，请输入`/var/log/httpd/error_log`。

尽管 Apache 是一个庞大的主题，我们无法涵盖其每一个细节，但在接下来的章节中，我们将继续揭示更多的功能，这些功能将帮助您构建一个理想的 Web 服务器。

# 启用系统用户和构建发布目录

在本食谱中，我们将学习 Apache 如何为您提供允许系统用户在其家目录中托管网页的选项。这种方法自 Web 托管开始以来就被 ISP 使用，并且在许多方面，由于它能够避免更复杂的虚拟托管方法，它继续蓬勃发展。在前一个食谱中，您被展示了如何安装 Apache Web 服务器，并且出于为系统用户提供托管设施的愿望，本食谱的目的是向您展示如何在 CentOS 7 中实现这一点。

## 准备就绪

要完成本食谱，您将需要一个具有 root 权限的工作 CentOS 7 操作系统安装和一个您选择的基于控制台的文本编辑器。预计您的服务器将使用支持主机名或域名的静态 IP 地址，并且 Apache Web 服务器已经安装并正在运行。此外，服务器上至少应有一个系统用户帐户。

## 如何操作...

为了提供本食谱所提供的功能，不需要额外的软件包，但我们需要对 Apache 配置文件进行一些修改。

1.  首先，以 root 身份登录，并在您最喜欢的文本编辑器中打开 Apache 用户目录配置文件，首先创建其备份副本，然后输入以下命令：

    ```
    cp /etc/httpd/conf.d/userdir.conf /etc/httpd/conf.d/userdir.conf.BAK
    vi /etc/httpd/conf.d/userdir.conf

    ```

1.  在文件中，找到读作`UserDir disabled`的指令。将其更改为以下内容：

    ```
    UserDir public_html

    ```

1.  现在滚动到`<Directory "/home/*/public_html">`部分，并用这里的块替换现有的块：

    ```
    <Directory /home/*/public_html>
        AllowOverride All
        Options Indexes FollowSymLinks
        Require all granted
    </Directory>
    ```

1.  保存并退出文件。现在以任何系统用户身份登录，以便与您的发布网页目录一起工作（`su - <username>`），然后在您的家目录中创建一个网页发布文件夹和一个新的用户主页：

    ```
    mkdir ~/public_html && vi ~/public_html/index.html

    ```

1.  现在添加所需的 HTML。您可以使用以下代码作为起点，但预计您会对其进行修改以满足自己的需求：

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head><title>Welcome to my web folder's home page</title></head>
    <body><h1>Welcome to my personal home page</h1></body>
    </html>
    ```

1.  现在通过键入以下内容来修改 Linux 系统用户的`<username>`家目录的权限：

    ```
    chmod 711 /home/<username>

    ```

1.  将`public_html`的读/写权限设置为`755`，以便 Apache 稍后可以执行它：

    ```
    chmod 755 ~/public_html -R

    ```

1.  现在再次以 root 身份登录，使用`su - root`命令来适当地配置 SELinux，以便使用 http 家目录：

    ```
    setsebool -P httpd_enable_homedirs true

    ```

1.  作为 root，更改用户网页公共目录的 SELinux 安全上下文（这需要安装`policycoreutils-python`软件包），用户名为`<user>`：

    ```
    semanage fcontext -a -t httpd_user_content_t /home/<user>/public_html
    restorecon -Rv /home/<user>/public_html

    ```

1.  要完成本食谱，只需重新加载`httpd`服务配置：

    ```
    apachectl configtest && systemctl reload httpd

    ```

1.  您现在可以通过在任何浏览器中浏览到（适当替换<username>）：`http://<SERVER IP ADDRESS>/~<username>`来测试您的设置。

## 它是如何工作的...

在本食谱中，我们了解到通过在 Apache Web 服务器上启用用户目录来托管自己的对等体是多么容易。

我们从这次经历中学到了什么？

我们首先通过在 Apache 的`userdir.conf`中进行一些小的配置更改来开始这个配方，以便设置用户目录支持。我们通过将`UserDir`指令从禁用调整为指向每个用户主目录内的 HTML 网页目录的名称来激活用户目录，该目录将包含我们所有用户的网页内容，并将其称为`public_html`（您可以更改此目录名称，但`public_html`是命名它的既定标准）。然后，我们继续修改`<Directory /home/*/public_html>`标签。此指令将其封闭的所有选项应用于开始标签`/home/*/public_html`定义的文件系统部分。在我们的示例中，为该目录启用了以下选项：当目录没有`index.html`时，使用`Indexes`显示目录的文件和文件夹内容作为 HTML。正如我们将在配方*Securing Apache*中看到的，这应该避免用于您的网络根目录，而对于提供用户目录，如果您只想让您的家庭文件夹对您的同行可访问，以便他们可以快速共享一些文件（如果您有任何安全问题，请删除此选项），这可能是一个不错的选择。`FollowSymLinks`选项允许从`public_html`目录到文件系统中任何其他目录或文件的符号链接（`man ln`）。同样，避免在您的网络根目录中使用，但对于家庭目录，如果您需要在不将它们复制到其中的情况下使文件或文件夹在`public_html`文件夹中可访问，这可能很有用（用户目录通常有磁盘配额）。接下来，我们配置了对`public_html`文件夹的访问控制。我们通过设置`Require` `all granted`来实现这一点，这告诉 Apache，在这个`public_html`文件夹中，任何人都可以通过 HTTP 协议访问内容。如果您想限制对`public_html`文件夹的访问，则可以替换`all granted`为不同的选项。例如，要基于主机名允许访问，请使用`Require host example.com`。使用`ip`参数，我们可以将`public_html`文件夹限制为仅内部可用的网络，例如`Require ip 192.168.1.0/24`。如果您的 Web 服务器具有多个网络接口，并且一个 IP 地址用于连接到公共 Internet，另一个用于您的内部专用网络，这特别有用。您可以在`Directory`块内添加多个`Require`行。请始终至少设置`Require local`，这允许本地访问。

保存工作后，我们开始对主目录进行各种更改。首先，我们在用户的主目录中创建了实际的`public_html`文件夹，这将成为稍后个人网页发布的实际文件夹。接下来，我们将权限更改为`755`，这意味着我们的用户可以在文件夹中执行所有操作，但其他用户和组只能读取和执行其内容（并进入该文件夹）。这种权限是必需的，因为如果有人通过 Apache Web 服务器请求其内容，`public_html`文件夹中的所有文件都将由名为`apache`的用户和组`apache`访问。如果未为`其他用户`标志设置读取或执行权限（`man chmod`），我们将在浏览器中收到`访问被拒绝`的消息。如果我们不提前更改父`/home/<username>`目录的权限，也会出现这种情况，因为父目录权限可以影响其子文件夹的权限。CentOS Linux 中的普通用户主目录具有`700`权限，这意味着主目录的所有者可以执行任何操作，但其他所有人都完全被锁定在主文件夹及其内容之外。

如前所述，Apache 用户需要访问子文件夹`public_html`，因此我们必须将主文件夹的权限更改为`711`，以便其他人至少可以进入目录（然后也可以访问子文件夹`public_html`，因为这被设置为可读/写访问）。接下来，我们为 SELinux 设置新网页文件夹的安全上下文。在运行 SELinux 的系统上，必须将所有 Apache 网页发布文件夹设置为`httpd_user_content_t` SELinux 标签（及其内容），以便使它们对 Apache 可用。此外，我们确保设置了正确的 SELinux 布尔值以启用 Apache 主目录（默认情况下已启用）：`httpd_enable_homedirs`为`true`。阅读第十四章，*使用 SELinux*了解更多关于 SELinux 的信息。

您应该知道，管理主目录的过程应该为每个用户重复。您不必每次启用新系统用户时都重新启动 Apache，但是，在第一次完成这些步骤后，只需重新加载`httpd`服务的配置以反映对配置文件所做的初始更改即可。从这一点开始，您的本地系统用户现在可以使用基于其用户名的唯一 URL 发布网页。

# 实施基于名称的托管

通常情况下，如果您按照之前的步骤安装了 Apache，您可以托管一个可以通过服务器 IP 地址或 Apache 运行的域名访问的网站，例如`http://192.168.1.100`或`http://www.centos7.home`。这种系统对于服务器资源来说非常浪费，因为您需要为每个想要托管的域名单独安装服务器。**基于名称的**或**虚拟主机**用于在同一 Apache Web 服务器上托管多个域名。如果已经通过 DNS 服务器或本地`/etc/hosts`文件将多个不同的域名分配给您的 Apache Web 服务器的 IP 地址，则可以为每个可用的域名配置虚拟主机，以将用户引导至 Apache 服务器上包含站点信息的特定目录。任何现代的网络空间提供商都使用这种类型的虚拟主机将一个 Web 服务器的空间分割成多个站点。只要您的 Web 服务器能够处理其流量，就没有限制可以创建的站点数量。在本步骤中，我们将学习如何在 Apache Web 服务器上配置基于名称的虚拟主机。

## 准备工作

要完成本步骤，您需要一个具有 root 权限的 CentOS 7 操作系统的正常安装，以及您选择的基于控制台的文本编辑器。预计您的服务器将使用静态 IP 地址，Apache 已安装并正在运行，并且您已经在之前的步骤中启用了系统用户发布目录。如果没有事先设置一个或多个域名或子域名，虚拟主机将无法工作。

为了测试，您可以在`/etc/hosts`中设置（参见第二章中的“设置主机名和解决网络问题”步骤），或者在您的 BIND DNS 服务器中配置一些 A 或 CNAMES（参考第九章），使用不同的域名或子域名，如`www.centos7.home`，全部指向您的 Apache Web 服务器的 IP 地址。

### 注意

一个常见的误解是，Apache 可以自行为您的 Apache Web 服务器创建域名。这是不正确的。您希望使用虚拟主机将不同的域名连接到不同目录之前，需要在 DNS 服务器或`/etc/hosts`文件中设置这些域名，使其指向您的 Apache 服务器的 IP 地址。

## 如何操作...

为了本配方的目的，我们将构建一些具有以下 Apache 示例子域名的本地虚拟主机：`www.centos7.home`，`web1.centos7.home`，`web2.centos7.home`和`<username>.centos7.home`，对应于 Web 发布文件夹`/var/www/html`，`/var/www/web1`，`/var/www/web2`和`/home/<username>/public_html`，以及域的网络名称`centos7.home`。这些名称是可互换的，预计您将希望根据更适合您自己需求和情况的内容来定制此配方。

1.  首先，以 root 身份登录到您的 Apache 服务器，并创建一个新的配置文件，该文件将包含我们所有的虚拟主机定义：

    ```
    vi /etc/httpd/conf.d/vhost.conf

    ```

1.  现在，请输入以下内容，将`centos7.home`的值和用户名`<username>`定制以适应您自己的需求：

    ```
    <VirtualHost *:80>
        ServerName centos7.home
        ServerAlias www.centos7.home
        DocumentRoot /var/www/html/
    </VirtualHost>   
    <VirtualHost *:80>
        ServerName  web1.centos7.home
        DocumentRoot /var/www/web1/public_html/
    </VirtualHost>
    <VirtualHost *:80>
        ServerName  web2.centos7.home
        DocumentRoot /var/www/web2/public_html/
    </VirtualHost>
    <VirtualHost *:80>
        ServerName  <username>.centos7.home
        DocumentRoot /home/<username>/public_html/
    </VirtualHost>
    ```

1.  现在以通常的方式保存并关闭文件，然后继续为当前缺失的两个虚拟主机创建目录：

    ```
    mkdir -p /var/www/web1/public_html /var/www/web2/public_html

    ```

1.  完成此操作后，我们现在可以使用我们喜欢的文本编辑器为缺失的子域`web1`和`web2`创建默认索引页面，如下所示：

    ```
    echo "<html><head></head><body><p>Welcome to Web1</p></body></html>" > /var/www/web1/public_html/index.html
    echo "<html><head></head><body><p>Welcome to Web2</p></body></html>" > /var/www/web2/public_html/index.html

    ```

1.  现在重新加载 Apache Web 服务器：

    ```
    apachectl configtest && systemctl reload httpd

    ```

1.  现在，为了简单的测试目的，我们将在想要访问这些虚拟主机的客户端计算机的`hosts`文件中配置我们新的 Apache Web 服务器的所有子域，但请记住，您也可以在 BIND DNS 服务器中配置这些子域。以 root 身份登录到此客户端计算机（它需要与我们的 Apache 服务器在同一网络中），并将以下行添加到`/etc/hosts`文件中，假设我们的 Apache 服务器具有 IP 地址 192.168.1.100：

    ```
    192.168.1.100 www.centos7.home
    192.168.1.100 centos7.home
    192.168.1.100 web1.centos7.home
    192.168.1.100 web2.centos7.home
    192.168.1.100 john.centos7.home

    ```

1.  现在，在这台计算机上，打开浏览器并通过在地址栏中输入以下地址来测试（将`<username>`替换为您为虚拟主机定义的用户名）：`http://www.centos7.home`，`http://web1.centos7.home`，`http://web2.centos7.home`和`http://<username>.centos7.home`。

## 它是如何工作的...

本配方的目的是向您展示实现基于名称的虚拟主机是多么容易。这种技术将提高您的工作效率，采用这种方法将为您提供无限的机会来进行基于域名的网络托管。

那么，我们从这次经历中学到了什么？

我们首先创建一个新的 Apache 配置文件来存放我们所有的虚拟主机配置。请记住，在`/etc/httpd/conf.d/`目录中以`.conf`扩展名结尾的所有文件将在 Apache 启动时自动加载。接着，我们继续添加相关的指令块，从我们的默认服务器根目录`centos7.home`和别名`www.centos7.home`开始。任何虚拟主机块中最重要的选项是`ServerName`指令，它将我们 Web 服务器的 IP 地址的现有域名映射到文件系统上的特定目录。当然，您可以包含更多的设置，但之前的解决方案提供了基本的构建块，使您能够将其作为完美的起点。接下来，我们为我们的`centos7.home`子域`web1`、`web2`和`<username>`创建了单独的条目。请记住，每个虚拟主机都支持典型的 Apache 指令，并且可以根据您的需要进行定制。请参考官方的 Apache 手册（安装 YUM 包`httpd-manual`，然后转到位置`/usr/share/httpd/manual/vhosts/`）以了解更多信息。在我们为每个想要的子域创建了虚拟主机块之后，我们继续创建了存放实际内容的目录，并在每个目录中创建了一个基本的`index.html`。在这个例子中，我们的`web1`和`web2`内容目录被添加到了`/var/www`。这并不是说你不能在其他地方创建这些新文件夹。实际上，大多数生产服务器通常将这些新目录放在主文件夹中，如我们的`/home/<username>/public_html`示例所示。但是，如果您确实打算采用这种方法，请记住修改这些新目录的权限和所有权，以及 SELinux 标签（在`/var/www`之外，您需要将 Apache 目录标记为`httpd_sys_content_t`），以便它们可以按预期使用。最后，我们重新加载了 Apache Web 服务，以便我们的新设置会立即生效。然后，我们可以在客户端的`/etc/hosts`中或在 BIND DNS 服务器上正确设置后，直接在浏览器中使用子域名浏览到我们的虚拟主机。

# 使用 Perl 和 Ruby 实现 CGI

在本章之前的食谱中，我们的 Apache 服务仅提供静态内容，这意味着网页浏览器请求的所有内容在服务器上已经处于恒定状态，例如作为不会改变的纯 HTML 文本文件。Apache 只是将 Web 服务器上特定文件的内容作为响应发送到浏览器，然后在那里进行解释和渲染。如果没有办法改变发送给客户端的内容，互联网将会非常无聊，也不会像今天这样取得巨大成功。甚至连最简单的动态内容示例，例如显示带有 Web 服务器当前本地时间的网页，都不可能实现。

因此，早在 20 世纪 90 年代初，一些聪明的人开始发明机制，使 Web 服务器和安装在服务器上的可执行程序之间的通信成为可能，以动态生成网页。这意味着发送给用户的 HTML 内容可以根据不同的上下文和条件改变。这些程序通常用脚本语言编写，如 Perl 或 Ruby，但也可以用任何其他计算机语言编写，如 Python、Java 或 PHP（见后文）。因为 Apache 是用纯 C 和 C++编写的，所以它不能执行或解释任何其他编程语言，如 Perl。因此，需要在服务器和程序之间建立一座桥梁，定义一些外部程序如何与服务器交互。这些方法之一被称为**通用网关接口**（**CGI**），这是一种非常古老的方式来提供动态内容。大多数 Apache Web 服务器使用某种形式的 CGI 应用程序，在这个食谱中，我们将向您展示如何安装和配置 CGI 以与 Perl 和 Ruby 一起使用，以生成我们的第一个动态内容。

### 注意

还存在一些特殊的 Apache Web 服务器模块，如`mod_perl`、`mod_python`、`mod_ruby`等，这些模块通常应该被优先考虑，因为它们直接将语言的解释器嵌入到 Web 服务器进程中，因此与任何接口技术（如 CGI）相比，它们要快得多。

## 准备就绪

为了完成这个食谱，你需要一个带有 root 权限的 CentOS 7 操作系统的有效安装，你选择的基于控制台的文本编辑器，以及一个互联网连接，以便下载额外的软件包。

预计你的服务器将使用静态 IP 地址，Apache 已安装并正在运行，并且你的服务器支持一个或多个域或子域。

## 如何操作...

由于 Perl 和 Ruby 这两种脚本语言在 CentOS 7 Minimal 中默认不安装，我们将从使用 YUM 安装所有必需的软件包开始这个食谱。

1.  开始时，以 root 身份登录并输入以下命令：

    ```
    yum install perl perl-CGI ruby

    ```

1.  接下来，重新启动 Apache Web 服务器：

    ```
    systemctl restart httpd

    ```

1.  接下来，我们需要为使用 CGI 脚本适当地配置 SELinux：

    ```
    setsebool -P httpd_enable_cgi 1

    ```

1.  然后，我们需要为 SELinux 的工作更改我们`cgi-bin`目录的正确安全上下文：

    ```
    semanage fcontext -a -t httpd_sys_script_exec_t /var/www/cgi-bin
    restorecon -Rv /var/www/cgi-bin

    ```

### 创建你的第一个 Perl CGI 脚本

1.  现在通过打开新文件`vi /var/www/cgi-bin/perl-test.cgi`并输入以下内容来创建以下 Perl CGI 脚本文件：

    ```
    #!/usr/bin/perl
    use strict;
    use warnings;
    use CGI qw(:standard);
    print header;
    my $now = localtime;
    print start_html(-title=>'Server time via Perl CGI'),
    h1('Time'),
    p("The time is $now"),
    end_html;
    ```

1.  接下来，将文件权限更改为 755，以便我们的`apache`用户可以执行它：

    ```
    chmod 755 /var/www/cgi-bin/perl-test.cgi

    ```

1.  接下来，为了测试并实际看到从前面的脚本生成的 HTML，你可以在命令行上直接执行`perl`脚本；只需输入：

    ```
    /var/www/cgi-bin/perl-test.cgi

    ```

1.  现在在网络中的一台计算机上打开浏览器，运行你的第一个 Perl CGI 脚本，它将通过使用 URL 打印本地时间：

    ```
    http://<server name or IP address>/cgi-bin/perl-test.cgi

    ```

1.  如果脚本不工作，请查看日志文件`/var/log/httpd/error_log`。

### 创建你的第一个 Ruby CGI 脚本

1.  创建新的 Ruby CGI 脚本文件`vi /var/www/cgi-bin/ruby-test.cgi`，并放入以下内容：

    ```
    #!/usr/bin/ruby
    require "cgi"
    cgi = CGI.new("html4")
    cgi.out{
     cgi.html{
     cgi.head{ cgi.title{"Server time via Ruby CGI"} } +
     cgi.body{
     cgi.h1 { "Time" } +
     cgi.p { Time.now}
     }
     }
    }

    ```

1.  现在将文件权限更改为`755`，以便我们的`apache`用户可以执行它：

    ```
    chmod 755 /var/www/cgi-bin/ruby-test.cgi

    ```

1.  要实际查看从前面脚本生成的 HTML，您可以在命令行上直接执行 Ruby 脚本；只需键入`/var/www/cgi-bin/ruby-test.cgi`。当显示行`offline mode: enter name=value pairs on standard input`时，按*Ctrl*+*D*查看实际的 HTML 输出。

1.  现在，在您的网络中的计算机上打开一个浏览器，运行您的第一个 Ruby CGI 脚本，该脚本将通过以下 URL 打印本地时间：

    ```
    http://<server name or IP address>/cgi-bin/ruby-test.cgi

    ```

1.  如果它不工作，请查看日志文件`/var/log/httpd/error.log`。

## 它是如何工作的...

在这个配方中，我们向您展示了使用 CGI 创建一些动态网站是多么容易。当访问 CGI 资源时，Apache 服务器在服务器上执行该程序，并将输出发送回浏览器。这个系统的主要优点是 CGI 不受任何编程语言的限制，只要程序可以在 Linux 命令行上执行并生成某种形式的文本输出即可。CGI 技术的主要缺点是它是一种非常老旧且过时的技术：对 CGI 资源的每个用户请求都会启动程序的新进程。例如，对 Perl CGI 脚本的每个请求都会启动并将新的解释器实例加载到内存中，这将产生大量开销，因此使得 CGI 仅适用于较小的网站或较低的并行用户请求数。如前所述，还有其他技术可以解决这个问题，例如 FastCGI 或 Apache 模块，如`mod_perl`。

那么我们从这次经历中学到了什么？

我们从这个配方开始，以 root 身份登录，并安装了`perl`解释器和`CGI.pm`模块，因为它们不包含在 Perl 标准库中（我们将在脚本中使用它），以及安装了 Ruby 编程语言的`ruby`解释器。之后，为了确保我们的 Apache Web 服务器注意到我们系统上安装的新编程语言，我们重新启动了 Apache 进程。

接下来，我们确保 SELinux 已启用以与 CGI 脚本配合工作，然后我们为标准的 Apache `cgi-bin`目录`/var/www/cgi-bin`提供了正确的 SELinux 上下文类型，以允许系统范围内的执行。要了解更多关于 SELinux 的信息，请阅读第十四章，*使用 SELinux*。然后，我们将 Perl 和 Ruby CGI 脚本放入此目录，并使它们对 Apache 用户可执行。在主 Apache 配置文件中，`/var/www/cgi-bin`目录默认被定义为标准 CGI 目录，这意味着您放入此目录的任何可执行文件，只要具有适当的访问和执行权限以及`.cgi`扩展名，都会自动定义为 CGI 脚本，并且可以从您的网络浏览器访问和执行，无论它使用哪种编程或脚本语言编写。为了测试我们的脚本，我们随后打开了一个网络浏览器，并访问了 URL `http://<服务器名称或 IP 地址>/cgi-bin/`，后面跟着`.cgi`脚本的名称。

## 还有更多...

如果您希望在其他网站目录中也能执行 CGI 脚本，您需要将以下两行（`Options`和`AddHandler`）添加到任何虚拟主机或现有的`Directive`指令中，或者按照以下方式创建一个新的（请记住，您还需要为新的 CGI 位置设置 SELinux `httpd_sys_script_exec_t`标签）：

```
<Directory "/var/www/html/cgi-new">
   Options +ExecCGI
   AddHandler cgi-script .cgi
</Directory>
```

# 安装、配置和测试 PHP

**超文本预处理器**（**PHP**）仍然是用于 Web 开发的最流行的服务器端脚本语言之一。它已经支持一些很好的功能，例如开箱即用地连接到关系数据库（如 MariaDB），这可以用来非常快速地实现现代 Web 应用程序。虽然可以看到一些大型企业倾向于放弃 PHP 而转向一些新技术，如 Node.js（服务器端 JavaScript），但它仍然是消费者市场上的主要脚本语言。世界上每家托管公司都提供某种 LAMP 堆栈（Linux、Apache、MySQL、PHP）来运行 PHP 代码。此外，许多非常流行的 Web 应用程序都是用 PHP 编写的，例如 WordPress、Joomla 和 Drupal，因此可以说 PHP 几乎是任何 Apache Web 服务器的必备功能。在本操作指南中，我们将向您展示如何在 Apache Web 服务器上开始安装和运行 PHP，使用模块`mod_php`。

## 准备就绪

要完成此操作，您需要一个具有 root 权限的工作 CentOS 7 操作系统安装，以及您选择的基于控制台的文本编辑器和互联网连接。预计您的服务器将使用静态 IP 地址，Apache 已安装并正在运行，并且您的服务器支持一个或多个域或子域。

## 如何操作...

我们将从安装 PHP 超文本处理器开始，同时安装 Apache 的`mod_php`模块，这两者在 CentOS 7 最小安装中默认不安装。

1.  首先，以 root 身份登录并输入以下命令：

    ```
    yum install mod_php

    ```

1.  现在，在我们先对原始文件进行备份之后，让我们打开标准的 PHP 配置文件：

    ```
    cp /etc/php.ini /etc/php.ini.bak && vi /etc/php.ini

    ```

1.  找到行`; date.timezone =`并将其替换为您自己的时区。所有可用的 PHP 时区列表可以在`http://php.net/manual/en/timezones.php`找到。例如（请确保删除前面的`;`，因为这会禁用命令的解释；这称为注释掉），要将时区设置为欧洲柏林市，请使用：

    ```
    date.timezone = "Europe/Berlin"

    ```

1.  为了确保新模块和设置已正确加载，请重启 Apache Web 服务器：

    ```
    systemctl restart httpd

    ```

1.  为了与前一个配方中的 CGI 示例保持一致，我们将创建我们的第一个动态 PHP 脚本，该脚本将打印出当前本地服务器时间，并在脚本`vi /var/www/html/php-test.php`中运行流行的 PHP 函数`phpinfo()`，我们可以使用它来打印出重要的 PHP 信息：

    ```
    <html><head><title>Server time via Mod PHP</title></head>
    <h1>Time</h1>
    <p>The time is <?php print Date("D M d, Y G:i a");?></p><?php phpinfo(); ?></body></html>
    ```

1.  要实际查看从前面脚本生成的 HTML，您可以直接在命令行上执行 PHP 脚本；只需输入：`php /var/www/html/php-test.php`。

1.  现在，在您网络中的计算机上打开浏览器，运行您的第一个 PHP 脚本，该脚本将通过以下 URL 打印本地时间：`http://<服务器名称或 IP 地址>/php-test.php`。

## 如何操作...

在本配方中，我们向您展示了通过使用`mod_php`模块将 PHP 轻松安装并集成到任何 Apache Web 服务器中是多么容易。该模块启用了一个内部 PHP 解释器，该解释器直接在 Apache 进程中运行，比使用 CGI 更高效，并且应该是任何可用时的首选方法。

那么我们从这次经历中学到了什么？

我们从使用 YUM 安装`mod_php`模块开始本节，这将安装 PHP 作为依赖项，因为这两者都不在任何标准的 CentOS 7 最小安装中。安装`mod_php`添加了`/etc/php.ini`配置文件，我们在备份原始文件后打开了它。该文件是主要的 PHP 配置文件，应谨慎编辑，因为许多设置可能与您的 Web 服务器的安全性相关。如果您刚刚开始使用 PHP，请将文件中的所有内容保持原样，不要更改任何内容，除了`date.timezone`变量。我们将其设置为反映我们当前的时区，这对于 PHP 是必要的，因为它被许多不同的时间和日期函数使用（我们还将在我们第一个 PHP 脚本中使用一些日期函数，如下所示）。接下来，我们重新启动了 Apache Web 服务器，它也会自动重新加载 PHP 配置。之后，我们创建了我们的第一个 PHP 脚本，并将其放入主 Web 根目录`/var/www/html/php-test.php`；这会打印出当前服务器时间以及`phpinfo()` PHP 函数的结果。这为您提供了一个分类良好的表格概览，显示了当前的 PHP 安装，帮助您诊断与服务器相关的问题或查看哪些模块在 PHP 中可用。

与 CGI 相比，您可能会问自己为什么我们不需要将 PHP 脚本放入任何特殊文件夹，如`cgi-bin`。通过安装`mod_php`，一个名为`/etc/httpd/conf.d/php.conf`的 Apache 配置文件被部署到 Apache 配置文件夹中，这正是回答了这个问题，它指定了每当 PHP 脚本从 Web 目录中的任何位置获得`.php`扩展名时，它们将被执行为有效的 PHP 代码。

# 确保 Apache 安全

尽管 Apache HTTP 服务器是 CentOS 7 中包含的最成熟和最安全的服务器应用程序之一，但总有余地进行改进，并且有大量选项和技术可用于进一步强化您的 Web 服务器的安全性。虽然我们无法向用户展示每一个安全特性，因为这超出了本书的范围，但在本节中，我们将尝试教授在为生产系统保护 Apache Web 服务器时被认为是良好实践的内容。

## 准备工作

要完成本节，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，以及您选择的基于控制台的文本编辑器。预计您的服务器将使用静态 IP 地址，并且 Apache 已安装并正在运行，并且您的服务器支持一个或多个域或子域。

## 如何操作...

大多数安全选项和技术都必须在 Apache 的主配置文件中设置，因此我们将从在喜欢的文本编辑器中打开它开始本节。

### 配置 httpd.conf 以提供更好的安全性

1.  首先，以 root 身份登录并打开 Apache 的主配置文件：

    ```
    vi /etc/httpd/conf/httpd.conf

    ```

1.  现在转到你的主文档根目录。为此，搜索名为：

    ```
    <Directory "/var/www/html">

    ```

1.  在开始`<Directory "/var/www/html">`和结束`</Directory>`标签之间找到行`Options Indexes FollowSymLinks`，然后通过在前面放置一个`#`来禁用（注释掉）该行，使其读取：

    ```
    #  Options Indexes FollowSymLinks

    ```

1.  现在滚动到配置文件的末尾，在`# Supplemental configuration`行之前插入以下行。我们不希望服务器通过标头泄露任何详细信息，因此我们输入：

    ```
    ServerTokens Prod

    ```

1.  之后，重新加载 Apache 配置以应用你的更改：

    ```
    apachectl configtest && systemctl reload httpd

    ```

### 移除不需要的 httpd 模块

即使是稳定性最高、成熟度最高、经过充分测试的程序也可能包含漏洞，正如最近关于 OpenSSL 中的 Heartbleed 漏洞或 Bash 中的 Shellshock 漏洞的新闻所显示的那样，Apache Web 服务器也不例外。因此，通常有益的是移除所有不需要的软件以限制功能，从而减少系统中出现安全问题的可能性。对于 Apache Web 服务器，我们可以移除所有不需要的模块以提高安全性（这也可以提高性能和内存消耗）。让我们通过审查所有当前安装的 Apache 模块来开始这个过程。

1.  要显示所有当前安装和加载的 Apache 模块，请以 root 用户身份输入：

    ```
    httpd -M

    ```

1.  前面命令输出的所有模块都通过`/etc/httpd/conf.modules.d`文件夹中的特殊配置文件加载到 Apache Web 服务器中，它们根据其主要目标分组到以下文件中：

    ```
    00-base.conf, 00-dav.conf, 00-lua.conf, 00-mpm.conf, 00-proxy.conf, 00-ssl.conf, 00-systemd.conf, 01-cgi.conf, 10-php.conf

    ```

1.  因此，与其逐一检查所有模块，`conf.modules.d`文件夹中的这种文件结构可以使我们的生活变得更加轻松，因为我们可以在整个模块组中启用/禁用。例如，如果你知道自己不需要任何 Apache DAV 模块，因为你不会提供任何 WebDAV 服务器，你可以通过将`00-dav.conf`配置文件的扩展名重命名来禁用所有与 DAV 相关的模块，因为只有以`.conf`结尾的文件才会被 Apache 自动读取和加载。为此，请输入：

    ```
    mv /etc/httpd/conf.modules.d/00-dav.conf /etc/httpd/conf.modules.d/00-dav.conf.BAK

    ```

1.  之后，重新加载 Apache 配置以将你的更改应用于模块目录：

    ```
    apachectl configtest && systemctl reload httpd

    ```

1.  如果你需要更精细的控制，你也可以在所有这些配置文件中启用/禁用单个模块。例如，在你的首选文本编辑器中打开`00-base.conf`，并通过在要禁用的行的开头添加`#`来禁用单个行。例如：

    ```
    # LoadModule userdir_module modules/mod_userdir.so

    ```

1.  如果你决定稍后使用一些禁用的模块文件，只需将`.BAK`文件重命名为原始文件名，或者在重新加载`httpd`之前，在特定的模块配置文件中删除`#`。

### 保护你的 Apache 文件

提高 Apache Web 服务器安全性的另一种简单方法是保护服务器端脚本和配置。在我们的场景中，有一个用户（root）单独负责并维护整个 Apache Web 服务器、网站（例如，将新的 HTML 页面上传到服务器）、服务器端脚本和配置。因此，我们将给予他/她完整的文件权限（读/写/执行）。`apache`用户仍然需要适当的读取和执行权限来服务和访问所有与 Apache 相关的文件，从而最小化您的 Apache Web 服务器向其他系统用户暴露潜在安全风险或通过 HTTP 攻击被破坏的风险。这可以通过两个步骤完成：

1.  首先，我们将更改或重置完整的 Apache 配置目录和标准 Web 根目录的所有权，所有者为`root`，组为`apache`：

    ```
    chown -R root:apache /var/www/html /etc/httpd/conf*

    ```

1.  之后，我们将更改文件权限，以便除了我们专门的`apache`用户（以及`root`）之外，任何人都无法读取这些文件：

    ```
    chmod 750 -R /var/www/html /etc/httpd/conf*

    ```

## 它是如何工作的...

我们从这个食谱开始，打开主 Apache 配置文件`httpd.conf`，以更改我们主 Apache 根 Web 内容目录`/var/www/html`的设置。在这里，我们禁用了完整的`Options`指令，包括`Indexes`和`FollowSymLinks`参数。正如我们所学，如果您从 Apache 服务器请求目录而不是文件，`index.html`或该目录中的`index.htm`文件将自动发送。现在，`Indexes`选项配置 Apache Web 服务器，以便如果在请求的目录中找不到这样的文件，Apache 将自动生成该目录内容的列表，就像您在命令行中输入`ls`（用于列出目录）一样，并将其作为 HTML 页面显示给用户。我们通常不希望这个功能，因为它可能会向未经授权的用户暴露秘密或私人数据，许多系统管理员会告诉您，索引通常被认为是一种安全威胁。`FollowSymLinks`指令也不应在生产系统中使用，因为如果您使用不当，它很容易暴露文件系统的一部分，例如完整的根目录。最后，我们添加了另一个措施来提高服务器的基本安全性，这是通过禁用服务器版本横幅信息来实现的。当 Apache Web 服务器生成网页或错误页面时，有价值的信息（例如 Apache 服务器版本和激活的模块）会自动发送到浏览器，潜在的攻击者可以从中获取有关您系统的宝贵信息。我们通过简单地将`ServerTokens`设置为`Prod`来阻止这种情况发生。之后，我们向您展示了如何禁用 Apache 模块以减少系统中错误和利用的一般风险。最后，我们展示了如何调整您的 Apache 文件权限，这也是一种很好的通用保护措施。

在加固 Apache Web 服务器时，有许多其他因素需要考虑，但大多数这些技术，如限制 HTTP 请求方法，`TraceEnable`，设置带有`HttpOnly`和安全标志的 cookie，禁用 HTTP 1.0 协议或 SSL v2，或使用有用的安全相关 HTTP 或自定义标头（如`X-XSS-Protection`）修改 HTTP 标头，都是更高级的概念，并且可能会过度限制通用目的的 Apache Web 服务器。

# 使用安全套接字层（SSL）设置 HTTPS

在本操作中，我们将学习如何通过使用 OpenSSL 创建自签名 SSL 证书来为 Apache Web 服务器添加安全连接。如果网站在服务器上运行时传输敏感数据，如信用卡或登录信息，则通常需要 Web 服务器。在前一个操作中，您已经了解了如何安装 Apache Web 服务器，随着对安全连接的需求不断增长，本操作的目的是向您展示如何通过教您如何扩展 Apache Web 服务器的功能来增强当前服务器配置。

## 准备工作

要完成此操作，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，您选择的基于控制台的文本编辑器，以及互联网连接以便于下载额外的包。预计 Apache Web 服务器已安装并正在运行。在这里，我们将为 Apache 创建一个新的 SSL 证书。如果您想了解更多信息，请参考第六章，*提供安全性*，以获取有关生成自签名证书的建议。由于正确的域名对于 SSL 的工作至关重要，我们将继续将 Apache Web 服务器的配置域名命名为`centos7.home`以使此操作生效（根据您的需要进行更改）。

## 如何操作...

Apache 默认不支持 SSL 加密，因此我们将首先使用 yum 包管理器安装必要的包`mod_ssl`。

1.  首先，以 root 身份登录并输入以下命令：

    ```
    yum install mod_ssl

    ```

1.  在安装 mod_ssl 包的过程中，会自动生成一个自签名证书以及 Apache Web 服务器的密钥对；这些证书缺少您 Web 服务器域名的正确通用名称。在我们能够使用下一步中的`Makefile`重新生成我们自己的所需 SSL 文件之前，我们需要删除这些文件：

    ```
    rm /etc/pki/tls/private/localhost.key /etc/pki/tls/certs/localhost.crt

    ```

1.  我们现在需要为我们的 Apache Web 服务器创建我们打算使用的自签名证书和服务器密钥。为此，请输入以下命令：

    ```
    cd /etc/pki/tls/certs

    ```

1.  要创建自签名 Apache SSL 密钥对，包括证书及其嵌入的公钥以及私钥，请输入：

    ```
    make testcert

    ```

1.  在创建证书的过程中，首先您将被要求输入一个新的密码，然后验证它。之后，您需要第三次输入它。通常，输入一个安全的密码。然后，您将被问一系列问题。填写所有必需的详细信息，特别注意通用名称值。此值应反映您的 Web 服务器的域名或 SSL 证书所针对的 IP 地址。例如，您可以输入：

    ```
    www.centos7.home

    ```

1.  当您创建证书的过程完成后，我们将通过以下方式打开主要的 Apache SSL 配置（在备份之后）：

    ```
    cp /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.conf.BAK
    vi /etc/httpd/conf.d/ssl.conf

    ```

1.  向下滚动到以`<VirtualHost _default_:443>`开头的部分，并找到该块内的行`# DocumentRoot "/var/www/html"`。然后通过删除`#`字符来激活它，使其读作：

    ```
    DocumentRoot "/var/www/html"

    ```

1.  在下面，找到读作`#ServerName www.example.com:443`的行。激活此行并修改显示的值以匹配创建证书时使用的通用名称值，如下所示：

    ```
    ServerName www.centos7.home:443

    ```

1.  保存并关闭文件，接下来我们需要在我们的 firewalld 中启用 HTTPS 端口，以允许通过端口`443`进行传入的 HTTP SSL 连接：

    ```
    firewall-cmd --permanent --add-service=https && firewall-cmd --reload

    ```

1.  现在重新启动 Apache `httpd`服务以应用您的更改。请注意，如果提示，您必须输入创建 SSL 测试证书时添加的 SSL 密码：

    ```
    systemctl restart httpd

    ```

1.  做得好！现在您可以通过替换我们为服务器定义的所有可用 HTTP URL，使用 HTTPS 而不是 HTTP 来访问您的服务器。例如，转到`https://www.centos7.home`而不是`http://www.centos7.home`。

    ### 注意

    当您访问此网站时，您会收到一条警告消息，指出签名证书颁发机构是未知的。使用自签名证书时，这种异常是可以预料的，并且可以确认。

## 它是如何工作的...

我们通过使用 YUM 包管理器安装`mod_ssl`开始了这个过程，这是默认的 Apache 模块，用于启用 SSL。接下来，我们前往 CentOS 7 中所有系统证书的标准位置，即`/etc/pki/tls/certs`。在这里，我们可以找到一个`Makefile`，这是一个方便生成自签名 SSL 测试证书的辅助脚本，它为你隐藏了 OpenSSL 程序的复杂命令行参数。请记住，`Makefile`目前缺少一个`clean`选项，因此每次运行它时，我们都需要手动删除以前运行生成的任何旧版本文件，否则它将不会开始做任何事情。删除旧的 Apache SSL 文件后，我们使用`make`命令和`testcert`参数，这将为 Apache Web 服务器创建自签名证书，并将它们放在标准位置，这些位置已经在`ssl.conf`文件中配置好了（`SSLCertificateFile`和`SSLCertificateKeyFile`指令），因此我们不需要在这里做任何更改。在过程中，在完成一系列问题之前，你会被要求提供一个密码。完成问题，但要特别注意通用名称。正如在主配方中提到的，这个值应该反映你的服务器域名或 IP 地址。在下一阶段，你需要在你的首选文本编辑器中打开 Apache 的 SSL 配置文件，该文件位于`/etc/httpd/conf.d/ssl.conf`。在其中，我们启用了`DocumentRoot`指令，将其置于 SSL 控制之下，并激活了`ServerName`指令，其预期域值必须与我们定义的通用名称值相同。然后，我们保存并关闭了配置文件，并在防火墙中启用了 HTTPS 端口，从而允许通过标准 HTTPS `443`端口进行传入连接。完成这些步骤后，你现在可以享受使用自签名服务器证书的安全连接的好处。只需在任何 URL 地址前输入`https://`而不是`http://`即可。但是，如果你打算在面向公众的生产服务器上使用 SSL 证书，那么最好的选择是从受信任的证书颁发机构购买 SSL 证书。

## 还有更多...

我们了解到，由于我们的 SSL 证书受密码保护，因此每当需要重启 Apache Web 服务器时，都需要输入密码。这对于服务器重启来说是不切实际的，因为 Apache 会在没有密码的情况下拒绝启动。为了消除密码提示，我们将把密码放在一个特殊文件中，并确保只有 root 用户可以访问它。

1.  创建包含你密码的文件的备份：

    ```
    cp /usr/libexec/httpd-ssl-pass-dialog /usr/libexec/httpd-ssl-pass-dialog.BAK

    ```

1.  现在用以下内容覆盖这个密码文件，将命令行中的`XXXX`替换为你的当前 SSL 密码：

    ```
    echo -e '#!/bin/bash\necho "XXXX"' >  /usr/libexec/httpd-ssl-pass-dialog

    ```

1.  最后，更改权限，使得只有 root 用户可以读取和执行它们：

    ```
    chmod 500 /usr/libexec/httpd-ssl-pass-dialog

    ```
