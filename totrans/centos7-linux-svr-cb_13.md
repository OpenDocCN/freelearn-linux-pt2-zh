# 第十三章：操作系统级虚拟化

在本章中，我们将涵盖：

+   安装和配置 Docker

+   下载镜像并运行容器

+   从 Dockerfile 创建自己的镜像并上传到 Docker Hub

+   设置和使用私有 Docker 仓库

# 引言

本章是一系列食谱的集合，提供了安装、配置和使用 Docker 的基本步骤，Docker 是一个开放平台，通过操作系统级虚拟化技术构建、运输、共享和运行分布式应用程序，这种技术在 Linux 世界中已经存在多年，并且可以提供比传统虚拟化技术更快的速度和效率优势。

# 安装和配置 Docker

传统的虚拟化技术提供*硬件虚拟化*，这意味着它们创建了一个完整的硬件环境，因此每个**虚拟机**（**VM**）都需要一个完整的操作系统来运行它。因此，它们有一些主要缺点，因为它们很重，运行时会产生大量开销。这就是开源 Docker 容器化引擎提供有吸引力的替代方案的地方。它可以帮助您在 Linux 容器中构建应用程序，从而提供应用程序虚拟化。

这意味着您可以将任何选择的 Linux 程序及其所有依赖项和自己的环境打包，然后共享它或运行多个实例，每个实例都是完全隔离和分离的进程，在任何现代 Linux 内核上运行，从而提供本机运行时性能、易于移植性和高可扩展性。在这里，在本食谱中，我们将向您展示如何在您的 CentOS 7 服务器上安装和配置 Docker。

## 准备就绪

要完成本食谱，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，您选择的基于控制台的文本编辑器，以及连接到互联网以便下载额外的`rpm`包和测试 Docker 镜像。

## 如何做到这一点...

虽然 Docker 在官方的 CentOS 7 仓库中作为一个包可用，但我们将在我们的系统上使用官方的 Docker 仓库来安装它。

1.  首先，以 root 身份登录并更新您的 YUM 包，然后使用以下命令下载并执行官方的 Docker Linux 安装脚本：

    ```
    yum update && curl -sSL https://get.docker.com/ | sh

    ```

1.  接下来，在启动 Docker 守护进程之前（第一次启动时会花费一些时间），启用 Docker 在启动时自动启动：

    ```
    systemctl enable docker && systemctl start docker

    ```

1.  最后，启动 Docker 后，您可以通过输入以下内容来验证它是否正常工作：

    ```
    docker run hello-world

    ```

## 它是如何工作的...

在 CentOS 7 上安装任何软件时，大多数情况下，使用官方 CentOS 仓库中的包而不是从第三方位置下载和安装是一个非常好的建议。在这里，我们通过使用官方 Docker 仓库安装 Docker 来做出例外。我们这样做是因为 Docker 是一个非常年轻的项目，发展迅速，变化频繁。虽然您可以使用 Docker 运行任何 Linux 应用程序，包括关键的 Web 服务器或处理机密数据的程序，但 Docker 程序中发现的或引入的错误可能会产生严重的安全后果。通过使用官方 Docker 仓库，我们确保始终能够尽快从这一快速发展的项目的开发者那里获得最新的更新和补丁。因此，将来任何时候您输入`yum update`，您的包管理器都会自动查询并检查 Docker 仓库，看看是否有新的 Docker 版本可供您使用。

那么我们从这次经历中学到了什么？

我们通过以 root 身份登录服务器并更新 YUM 包的数据库开始这个操作步骤。然后，我们使用一个命令从[`get.docker.com/`](https://get.docker.com/)下载并执行官方 Docker 安装脚本，一步完成。该脚本的作用是将官方 Docker 仓库添加到 YUM 包管理器作为新的包源，然后自动在后台安装 Docker。之后，我们通过使用`systemd`在启动时启用 Docker 服务并启动它。最后，为了测试我们的安装，我们发出了`docker run hello-world`命令，该命令从官方 Docker 注册表下载一个特殊的镜像来测试我们的安装。如果一切顺利，您应该会看到以下成功消息（输出已截断）：

```
Hello from Docker

```

这条消息表明您的安装似乎运行正常。

# 下载镜像并运行容器

一个常见的误解是 Docker 是一个运行容器的系统。Docker 只是一个构建工具，用于将任何基于 Linux 的软件及其所有依赖项打包到一个包含运行所需一切的完整文件系统中：代码、运行时、系统工具和系统库。运行 Linux 容器的技术称为操作系统级虚拟化，它提供了在每个现代 Linux 内核中默认构建的多个隔离环境。这保证了它无论部署在什么环境中都将始终以相同的方式运行；从而使您的应用程序具有可移植性。因此，当涉及到将您的 Docker 应用程序分发到 Linux 容器时，必须引入两个主要的概念性术语：**Docker 镜像**和**容器**。如果您曾经想设置并运行自己的 WordPress 安装，在本操作步骤中，我们将向您展示如何通过从官方 Docker Hub 下载预制的 WordPress 镜像来以最快的方式实现这一点；然后我们将从中运行一个容器。

## 准备就绪

要完成此教程，您需要一个具有 root 权限的 CentOS 7 操作系统的工作安装，您选择的基于控制台的文本编辑器，以及连接到互联网以便下载额外的 Docker 镜像。预计 Docker 已经安装并正在运行。

## 如何操作...

从 Docker Hub 下载的官方 WordPress 镜像不包含自己的 MySQL 服务器。相反，它依赖于外部的 MySQL 服务器，因此我们将从安装并运行一个从 Docker Hub 下载的 MySQL Docker 容器开始这个教程。

1.  首先，以 root 身份登录并键入以下命令，将`<PASSWORD>`替换为您自己选择的强 MySQL 数据库密码（在撰写本文时，最新的 WordPress 需要 MySQL v.5.7；这在未来可能会改变，因此请查看官方 WordPress Docker Hub 页面）：

    ```
    docker run --restart=always --name wordpressdb -e MYSQL_ROOT_PASSWORD=<PASSWORD> -e MYSQL_DATABASE=wordpress -d mysql:5.7

    ```

1.  接下来，安装并运行官方 WordPress 镜像，并将其作为 Docker 容器运行，将其连接到 MySQL 容器（提供与前一步骤相同的`<PASSWORD>`字符串）：

    ```
    docker run --restart=always -e WORDPRESS_DB_PASSWORD=<password> -d --name wordpress --link wordpressdb:mysql -p 8080:80 wordpress

    ```

1.  现在，MySQL 和 WordPress 容器应该已经在运行。要检查当前正在运行的容器，请键入：

    ```
    docker ps

    ```

1.  要获取所有 Docker WordPress 容器设置，请使用：

    ```
    docker inspect wordpress

    ```

1.  要检查我们的 WordPress 容器的日志文件，请运行以下命令：

    ```
    docker logs -f wordpress

    ```

1.  在同一网络中与运行 Docker 守护进程的服务器相连的计算机上打开浏览器，输入以下命令以访问您的 WordPress 安装（将 IP 地址替换为您的 Docker 服务器的 IP 地址）：

    ```
    http://<IP ADDRESS OF DOCKER SERVER>:8080/

    ```

## 它是如何工作的...

Docker 镜像是一组构成软件应用程序及其功能依赖的所有文件，以及有关您修改或改进其内容时所做的任何更改的信息（以更改日志的形式）。它是您的应用程序的不可运行、只读版本，可以与 ISO 文件相比较。如果您想运行这样的镜像，Linux 容器将自动从它创建出来。这就是实际执行的内容。它是一个真正的可扩展系统，因为您可以从同一镜像运行多个容器。正如我们所见，Docker 不仅仅是您需要与镜像和容器一起工作的工具，它还是一个完整的平台，因为它还提供了访问各种 Linux 服务器软件的预制镜像的工具。这整个 Docker 系统的美丽之处在于，大多数时候您不必重新发明轮子，试图从头开始创建自己的 Docker 镜像。只需访问 Docker Hub（[`hub.docker.com`](https://hub.docker.com)），搜索您想要作为容器运行的软件，找到它后，只需使用`docker run`命令，提供 Docker Hub 镜像的名称，就完成了。当考虑到尝试让最新流行的程序与您需要编译的所有依赖项一起工作并尝试安装它们时，Docker 真的可以成为救星。

那么我们从这次经历中学到了什么？

我们的旅程始于使用`docker run`命令，该命令从远程 Docker Hub 仓库下载了两个镜像并将其放入本地镜像存储（称为`mysql:5.7`和`wordpress`），然后运行它们（创建容器）。要获取机器上所有下载的镜像列表，请键入`docker images`。正如我们所见，两个`run`命令行都提供了`-e`命令行参数，我们需要设置一些基本的环境变量，这些变量随后将在容器内可见。这些包括我们想要运行的 MySQL 数据库以及设置和访问它们的 MySQL 根密码。这里我们看到 Docker 的一个非常重要的特性：能够相互通信的容器！通常，您可以将应用程序从不同的 Docker 容器部件堆叠在一起，使整个系统非常易于使用。另一个重要参数是`-p`，它用于从我们的主机端口`8080`创建到内部 HTTP 端口 80 的端口映射，并打开防火墙以允许此端口上的传入流量。`--restart=always`对于使图像容器可重新启动很有用，因此容器会在主机机器重新启动时自动重新启动。之后，我们向您介绍了 Docker 的`ps`命令行参数，该参数打印出所有正在运行的 Docker 容器。此命令应打印出两个名为`wordpressdb`和`wordpress`的运行容器，以及它们的`CONTAINER_ID`。此 ID 是唯一的 MD5 哈希，我们将在大多数 Docker 命令行输入中使用它，无论何时我们需要引用特定的容器（在本食谱中，我们通过容器名称引用，这也是可能的）。之后，我们向您展示了如何使用`inspect`参数打印出容器的配置。然后，为了以开放流的形式获取 Wordpress 容器的日志文件，我们使用了日志`-f`参数。最后，由于`-p 8080:80`映射允许在端口 8080 上对我们的服务器进行传入访问，因此我们可以从同一网络中的任何计算机使用浏览器访问我们的 Wordpress 安装。这将打开 Wordpress 安装屏幕。

### 注意

请注意，如果您在任何时候从 Docker 下载任何容器时遇到任何连接问题，例如`dial tcp: lookup index.docker.io: no such host`，请在再次尝试之前重新启动 Docker 服务。

## 还有更多...

在本节中，我们将向您展示如何启动和停止容器以及如何附加到您的容器。

### 停止和启动容器

在主配方中，我们使用了 Docker 的`run`命令，它实际上是两个其他 Docker 命令的包装：`create`和`start`。正如这些命令的名称所暗示的，`create`命令从现有镜像创建（克隆）一个容器，如果它不在本地镜像缓存中，则从给定的 Docker 注册表（如预定义的 Docker hub）下载它，而`start`命令实际上启动它。要获取计算机上所有容器（运行或停止）的列表，请输入：`docker ps -a`。现在识别一个停止或启动的容器，并找出其特定的`CONTAINER_ID`。然后，我们可以通过提供正确的`CONTAINER_ID`来启动一个停止的容器或停止一个运行的容器，例如`docker start CONTAINER_ID`。示例包括：`docker start 03b53947d812`或`docker stop a2fe12e61545`（`CONTAINER_ID`哈希值将根据你的计算机而变化）。

有时你可能需要删除一个容器；例如，如果你想在从镜像创建容器时完全更改其命令行参数。要删除容器，请使用`rm`命令（但请记住，它必须在停止后才能删除）：`docker stop b7f720fbfd23; docker rm b7f720fbfd23`

### 连接并与你的容器交互

Linux 容器是完全隔离的进程，在你的服务器上运行在分离的环境中，无法像使用`ssh`登录普通服务器那样登录到它。如果你需要访问容器的 BASH shell，则可以运行`docker exec`命令，这对于调试问题或修改容器（例如，安装新软件包或更新程序或文件）特别有用。请注意，这只适用于运行中的容器，并且你需要在运行以下命令之前知道容器的 ID（输入`docker ps`以查找）：`docker exec -it CONTAINER_ID /bin/bash`，例如`docker exec -it d22ddf594f0d /bin/bash`。一旦成功连接到容器，你将看到一个略有变化的命令行提示符，其中`CONTAINER_ID`作为主机名；例如，`root@d22ddf594f0d:/var/www/html#`。如果你需要退出容器，请输入`exit`。

# 通过 Dockerfiles 创建自己的镜像并上传到 Docker Hub

除了图像和容器，Docker 还有一个非常重要的术语叫做**Dockerfile**。Dockerfile 就像是一个创建特定应用程序环境的食谱，意味着它包含了构建特定镜像文件的蓝图和确切描述。例如，如果我们想要容器化一个基于 Web 服务器的应用程序，我们会在 Dockerfile 中定义所有依赖项，比如提供系统依赖项的基础 Linux 系统，如 Ubuntu、Debian、CentOS 等（这并不意味着我们*虚拟化*了完整的操作系统，而只是使用了系统依赖项），以及所有应用程序、动态库和服务，如 PHP、Apache 和 MySQL，还包括所有特殊的配置选项或环境变量。有两种方法可以构建自己的自定义镜像。一种方法是从我们之前在 Wordpress 食谱中下载的现有基础镜像开始，然后使用 BASH 附加到容器，安装额外的软件，对配置文件进行更改，然后将容器作为新镜像提交到注册表。或者，在本食谱中，我们将教您如何从新的 Dockerfile 为 Express.js Web 应用程序服务器构建自己的 Docker 镜像，并将其上传到您自己的 Docker Hub 账户。

## 准备就绪

要完成本食谱，您需要一个具有 root 权限的 CentOS 7 操作系统的工作安装，您选择的基于控制台的文本编辑器，以及连接到互联网以便与 Docker Hub 通信。预计 Docker 已经安装并正在运行。此外，为了将您的新镜像上传到 Docker Hub，您需要在 Docker Hub 上创建一个新的用户账户。只需访问[`hub.docker.com/`](https://hub.docker.com/)并免费注册。在我们的示例中，我们将使用一个虚构的新 Docker Hub 用户 ID，称为`johndoe`。

## 如何操作...

1.  首先，以 root 身份登录，使用您的 Docker Hub 用户 ID 创建一个新的目录结构（将`johndoe`目录名称适当地替换为您自己的 ID），并打开一个空的 Dockerfile，您将在其中放入镜像构建的蓝图：

    ```
    mkdir -p ~/johndoe/centos7-expressjs
    cd $_; vi Dockerfile

    ```

1.  将以下内容放入该文件中：

    ```
    FROM centos:centos7
    RUN yum install -y epel-release;yum install -y npm;
    RUN npm install express --save
    COPY . ./src
    EXPOSE 8080
    CMD ["node", "/src/index.js"]

    ```

1.  保存并关闭文件。现在创建您的第一个 Express.js Web 应用程序，我们将在新容器上部署它。在当前目录中打开以下文件：

    ```
    vi index.js

    ```

1.  现在将以下 JavaScript 内容放入：

    ```
    var express = require('express'), app = express();
    app.get('/', function (req, res) {res.send('Hello CentOS 7 cookbook!\n');});
    app.listen(8080);

    ```

1.  现在要从这个 Dockerfile 构建一个镜像，请保持在当前目录中，并使用以下命令（不要忘记此行末尾的点，并将`johndoe`替换为您自己的 Docker Hub ID）：

    ```
    docker build -t johndoe/centos7-expressjs .

    ```

1.  成功构建镜像后，让我们将其作为容器运行：

    ```
    docker run -p 8081:8080 -d johndoe/centos7-expressjs

    ```

1.  最后，测试我们是否可以向我们新创建的容器中运行的 Express.js Web 应用程序服务器发出 HTTP 请求：

    ```
    curl -i localhost:8081

    ```

1.  如果 Docker 镜像成功运行在 Express.js 服务器上，应该会出现以下 HTTP 响应（截断至最后一行）：

    ```
    Hello CentOS 7 cookbook!

    ```

### 将您的映像上传到 Docker Hub

1.  创建一个新的 Docker Hub 账号 ID，名为`johndoe`后，我们将开始使用以下命令登录网站——保持在您放置 Dockerfile 的目录中，例如`~/johndoe/centos7-expressjs`（在提示时提供用户名、密码和注册电子邮件）：

    ```
    docker login

    ```

1.  现在，要将本教程中创建的新映像推送到 Docker Hub（再次将`johndoe`替换为您自己的用户 ID），请使用：

    ```
    docker push johndoe/centos7-expressjs

    ```

1.  上传后，您将能够在 Docker Hub 网页搜索中找到您的映像。或者，您可以使用命令行：

    ```
    docker search expressjs

    ```

## 它是如何工作的...

在这篇简短的教程中，我们向您展示了如何创建您的第一个 Dockerfile，该文件将创建一个用于运行 Express.js 应用程序的 CentOS 7 容器，这是一种现代的 LAMP 堆栈替代方案，您可以在客户端和服务器端编程 JavaScript。

那么我们从这次经历中学到了什么？

如您所见，Dockerfile 是一种优雅的方式来描述创建映像的所有指令。命令易于理解，您使用特殊的关键字来指示 Docker 如何操作以从其生成映像。`FROM`命令告诉 Docker 我们应该使用哪个基础映像。幸运的是，已经有人从 CentOS 7 系统依赖项创建了基础映像（这将从 Docker Hub 下载）。接下来，我们使用`RUN`命令，它只是在 BASH 命令行上执行命令。我们使用此命令在我们的系统上安装依赖项以运行 Express.js 应用程序（它是基于 Node.js rpm 包的，我们首先通过安装 EPEL 存储库来访问它）。`COPY`命令将文件从我们的主机复制到容器上的特定位置。我们需要这个来复制我们的`index.js`文件，该文件将在稍后的步骤中创建我们所有的 Express.js Web 服务器代码到容器上。`EXPOSE`，顾名思义，将内部容器端口暴露给外部主机系统。由于默认情况下 Express.js 监听 8080 端口，我们需要在这里这样做。虽然到目前为止显示的所有这些命令只在创建映像时执行一次，但下一个命令`CMD`将在我们每次启动容器时运行。`node /src/index.js`命令将被执行，并指示系统使用`index.js`文件启动 Express.js Web 服务器（我们已经通过从主机复制它来提供此文件）。我们不想深入讨论程序的 JavaScript 部分——它只是处理 HTTP GET 请求并返回`Hello World`字符串。在本教程的第二部分中，我们向您展示了如何将我们新创建的映像推送到 Docker Hub。为此，请使用您的 Docker 用户账户登录。然后我们可以将我们的映像推送到存储库。

由于这是一个非常简单的 Dockerfile，关于这个主题还有很多要学习的内容。要查看 Dockerfile 中所有可用命令的列表，请使用`man Dockerfile`。此外，你应该访问 Docker Hub 并浏览一些有趣项目的 Dockerfiles（在*GitHub 上托管的源代码库*部分下），以学习如何仅用几个命令就能创建一些高度复杂的镜像文件。

# 设置和使用私有 Docker Registry

在本章的前一个配方中，我们已经了解到将自己的镜像上传到官方 Docker Hub 是多么容易，但我们在那里上传的所有内容都将公开。如果你在一个企业环境中处理私有或闭源项目，或者只是想在向所有人发布之前测试一些东西，那么你很可能更倾向于拥有自己的、受保护的或企业范围内的私有 Docker Registry。在本配方中，我们将向你展示如何设置和使用你自己的 Docker Registry，该 Registry 将在你自己的私有网络中可用，并通过 TLS 加密和用户认证进行保护，这样你就可以精确控制谁可以使用它（推送和拉取镜像到和从它）。

## 准备工作

要完成这个配方，你需要一个安装了 CentOS 7 操作系统并具有 root 权限的工作环境，一个你选择的基于控制台的文本编辑器，以及一个互联网连接以便下载额外的软件包。在我们的例子中，我们将在 IP 地址为`192.168.1.100`的服务器上安装 Docker Registry。根据你的需求适当调整配方的命令。你需要为这台服务器设置一个完全限定域名（FQDN），否则注册表将无法工作。为了简化，我们将使用`/etc/hosts`方法而不是设置和配置一个 DNS 服务器（如果你想这样做，请参阅第九章/Docker registry/g' | sed 's/<description>.*<\/description>//g' > /etc/firewalld/services/docker-reg.xml
    firewall-cmd --reload
    firewall-cmd --permanent --add-service=docker-reg; firewall-cmd --reload

    ```

### 需要在每个需要访问我们注册表的客户端上执行的步骤

1.  最后，我们可以通过在同一网络中的任何计算机上以 root 身份登录来测试连接到我们自己的新 TLS 增强的私有 Docker 注册表的用户身份验证。

1.  第一步是在每个想要连接到 Docker 注册表的客户端上安装 Docker：

    ```
    yum update && curl -sSL https://get.docker.com/ | sh

    ```

1.  接下来，在每个想要连接到我们新 Docker 注册表的客户端上，首先在客户端上设置服务器的证书，然后我们才能连接到它（此步骤仅在 CentOS 7 客户端上测试过）：

    ```
    mkdir -p /etc/docker/certs.d/$DCKREG\:5000
    curl http://$DCKREG/docker-registry.crt -o /tmp/cert.crt
    cp /tmp/cert.crt /etc/docker/certs.d/$DCKREG\:5000/ca.crt
    cp /tmp/cert.crt /etc/pki/ca-trust/source/anchors/docker-registry.crt
    update-ca-trust

    ```

1.  为了测试，我们首先从官方 Docker Hub 拉取一个新的小的测试镜像。使用您的 Docker Hub 帐户登录到官方 Docker Hub（请参阅本章中的前一个配方）：

    ```
    docker login

    ```

1.  现在拉取一个名为`busybox`的小镜像：

    ```
    docker pull busybox

    ```

1.  之后，将 Docker 注册表服务器切换到我们在此配方中设置的自己的服务器（输入用户名和密码，例如，`johndoe / mysecretpassword`。电子邮件字段留空）：

    ```
    docker login $DCKREG:5000

    ```

1.  接下来，为了将 Docker 镜像从客户端推送到我们新的私有 Docker 注册表，我们需要将其标记为在我们的注册表域中：

    ```
    docker tag busybox $DCKREG:5000/busybox

    ```

1.  最后，将镜像推送到我们自己的注册表：

    ```
    docker push $DCKREG:5000/busybox

    ```

1.  恭喜！你刚刚将你的第一个镜像推送到了你的私有 Docker 仓库。现在，你可以在任何配置为与我们的仓库通信的其他客户端上拉取这个镜像`$DCKREG:5000/busybox`。要获取所有可用镜像的列表，请使用（根据需要更改账户信息）：

    ```
    curl https://johndoe:mysecretpassword@$DCKREG:5000/v2/_catalog

    ```

## 它是如何工作的...

在本食谱中，我们向你展示了如何在服务器上的 Docker 容器中设置你自己的 Docker 注册表。理解这一点非常重要：你需要为你的注册表服务器配置一个 FQDN，因为这是整个系统工作的必要条件。

那么，我们从这次经历中学到了什么？

我们首先在每台计算机上通过`/etc/hosts`方法配置 Docker 注册表的完全限定域名（FQDN）。然后，我们在 Docker 注册表服务器上创建了一个新证书，该证书将用于客户端和注册表之间通过 TLS 加密进行安全通信。接下来，我们在`httpd`服务器上安装了新生成的证书，以便稍后所有客户端都可以访问；同时，在特定的 Docker 目录中，以便 Docker 也可以访问；并且在服务器默认信任的证书位置，我们还为该服务器重建了证书缓存。之后，我们使用`docker run`命令下载、安装并在该服务器上的 Docker 容器中运行我们的新 Docker 注册表。我们提供了一组参数来配置 TLS 加密和用户认证。

在下一步中，我们连接到注册表以创建新的`htpasswd`账户。每当你的注册表需要新账户时，你都可以重复此步骤。别忘了之后重启注册表容器。接下来，对于我们希望与之通信的每个客户端，我们都需要在服务器本身上的相同位置安装服务器证书；因此，我们从之前实现的 HTTP 源下载了它，并将其复制到各个位置。为了在客户端上测试，接下来我们连接到官方 Docker Hub 下载我们想要在下一步推送到我们自己的注册表的随机镜像。我们将`busybox`镜像下载到我们自己的镜像缓存中，然后切换到连接到我们的新私有 Docker 注册表。在我们能够将镜像上传到新位置之前，我们必须给它一个适合新服务器名称的适当标签，然后我们才能够将镜像推送到我们的新 Docker 注册表。该服务器现在在整个网络中端口 5000 上可用。请记住，如果你不想在客户端上再使用自己的注册表，你可以随时切换回官方`docker`仓库，使用`docker login`。

关于 Docker，还有很多东西需要学习。在本章的食谱中，我们只是触及了 Docker 平台的表面。如果你想了解更多关于它的信息，请考虑访问[`www.Packtpub.com`](https://www.Packtpub.com)，并查看该网站上提供的许多相关书籍。
