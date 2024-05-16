# 第十章：使用数据库

在本章中，我们将涵盖：

+   安装 MariaDB 数据库服务器

+   管理 MariaDB 数据库

+   允许对 MariaDB 服务器的远程访问

+   安装 PostgreSQL 服务器和管理数据库

+   配置对 PostgreSQL 的远程访问

+   安装 phpMyAdmin 和 phpPgAdmin

# 引言

本章是一系列配方的集合，提供了在 Linux 世界中实施和维护两个最流行的数据库管理系统所需的步骤。数据的需求无处不在，对于几乎任何服务器来说，它都是*必须提供的服务*，本章提供了在任何环境中部署这些数据库系统所需的起点。

# 安装 MariaDB 数据库服务器

支持超过 70 种排序规则，30 多种字符集，多种存储引擎，以及在虚拟化环境中的部署，MySQL 是一个关键任务的数据库服务器，被全球的生产服务器所使用。它能够托管大量的独立数据库，并能为你的整个网络提供各种角色的支持。MySQL 服务器已经成为**万维网** (**WWW**) 的代名词，被桌面软件使用，扩展本地服务，并且是全球最受欢迎的关系数据库系统之一。本配方的目的是向你展示如何下载、安装和锁定 MariaDB，这是 CentOS 7 中 MySQL 的默认实现。MariaDB 是开源的，与 MySQL 完全兼容，并增加了几个新功能；例如，非阻塞客户端 API 库，具有更好性能的新存储引擎，增强的服务器状态变量，以及复制。

## 准备工作

为了完成这个配方，你需要一个具有 root 权限的 CentOS 7 操作系统的工作安装，一个你选择的基于控制台的文本编辑器，以及一个互联网连接以下载额外的软件包。预计你的服务器将使用静态 IP 地址。

## 如何操作...

由于 MariaDB **数据库管理系统** (**DBMS**) 在 CentOS 7 上默认未安装，我们将从这个配方开始安装所需的软件包。

1.  首先，以 root 身份登录并输入以下命令来安装所需的软件包：

    ```
    yum install mariadb-server mariadb

    ```

1.  完成后，确保服务在启动时启动，然后再启动服务：

    ```
    systemctl enable mariadb.service && systemctl start mariadb.service

    ```

1.  最后，使用以下命令开始安全安装过程：

    ```
    mysql_secure_installation

    ```

1.  当你首次运行前面的命令时，系统会要求你提供一个密码，但由于此值尚未设置，请按*Enter*键表示该值（空白）无。

1.  现在，你将被问到一系列简单的问题，这些问题将帮助你在加固 MariaDB DBMS 系统的过程中。除非你已经是 MariaDB 专家并且确实需要某个特定功能，否则选择 Yes (`Y`) 回答每个问题以获得最大安全性是一个好建议。

1.  最后，测试你是否可以使用 MariaDB 命令行客户端`mysql`本地连接并登录到 MariaDB 服务。如果以下命令输出了 MariaDB 服务器已知的所有用户名及其关联的主机，则测试通过（在提示时输入你在上一步设置的管理员 root 密码）：

    ```
    echo "select User,Host from user" | mysql -u root -p mysql

    ```

## 它是如何运作的...

MariaDB 是一个快速、高效、多线程且强大的 SQL 数据库服务器。它支持多个用户，并提供多种存储引擎的访问。通过遵循几个简短的步骤，你现在知道如何安装、保护并登录到你的 MariaDB 服务器。

那么，我们从这次经历中学到了什么？

我们首先安装了 MariaDB 服务器所需的软件包（`mariadb-server`），以及用于控制和查询服务器的客户端 Shell 接口（`mariadb`）。完成这一步后，我们确保 MariaDB 守护进程（`mariadb.service`）会在启动过程中启动，然后我们才真正启动它。此时，我们有了一个可用的安装，但为了确保我们的安装是安全的，我们随后调用了安全安装脚本，引导我们通过几个简单的步骤来加强我们的基本安装。由于基本安装过程不允许我们为 root 用户设置默认密码，我们在这里作为脚本的第一步进行了设置，这样我们就可以确保没有人可以在没有所需授权的情况下访问 MariaDB 的 root 用户账户。然后我们发现，典型的 MariaDB 安装保留了一个匿名用户。这样做的目的是允许任何人在没有有效用户账户的情况下登录到我们的数据库服务器。它通常仅用于测试目的，除非你处于需要此功能的特殊情况，否则总是建议删除此功能。接下来，为了确保 root 用户无法访问我们的 MariaDB 服务器安装，我们选择禁止远程 root 访问，然后删除测试数据库并重新加载权限表。最后，我们运行了一个小测试，看看我们是否可以使用 root 用户连接到数据库，并从`user`表（这是标准`mysql`数据库的一部分）查询一些数据。

完成这些步骤后，我们了解到安装和保护 MariaDB 服务器的过程非常简单。当然，总有一些事情可以做，以使安装更有用，但这个食谱的目的是向你展示，安装新数据库系统最重要的部分是使其安全。记住，运行`mysql_secure_installation`对于所有 MariaDB 服务器都是推荐的，无论你是在构建开发服务器还是用于生产环境的服务器，这都是明智的。作为服务器管理员，安全应始终是你的首要任务。

# 管理 MariaDB 数据库

在本配方中，我们将学习如何为 MariaDB 服务器创建一个新的数据库和数据库用户。MariaDB 可以与各种图形工具（例如，免费的 MySQL Workbench）结合使用，但在您只需要创建一个数据库、提供关联用户并分配正确权限的情况下，通常使用命令行执行此任务非常有用。被称为 MariaDB shell 的这个简单的交互式基于文本的命令行工具支持完整的 SQL 命令范围，并提供对数据库服务器的本地和远程访问。该 shell 为您提供了对数据库服务器的完全控制，因此它代表了您开始 MariaDB 工作的完美工具。

## 准备工作

要完成本配方，您需要一个正常运行的 CentOS 7 操作系统。预计您的服务器上已经安装并运行了 MariaDB 服务器。

## 如何做到这一点...

MariaDB 命令行工具支持在批处理模式（从文件或标准输入读取）和交互模式（输入语句并等待结果）中执行命令。在本配方中，我们将使用后者。

1.  首先，使用您喜欢的任何系统用户登录到您的 CentOS 7 服务器，然后输入以下命令，以便使用名为`root`的主要 MariaDB 管理用户通过 MariaDB shell 访问 MariaDB 服务器（使用在前一个配方中创建的密码）：

    ```
    mysql -u root -p

    ```

1.  成功登录后，您将看到 MariaDB 命令行界面。此功能由 MariaDB shell 提示符表示：

    ```
    MariaDB [(none)]>

    ```

1.  在第一步中，我们将创建一个新的数据库。为此，只需通过将以下命令中的`<database-name>`替换为适当的值来定制命令：

    ```
    CREATE DATABASE <database-name> CHARACTER SET utf8 COLLATE utf8_general_ci;

    ```

    ### 注意

    如果您是第一次接触 MariaDB shell，请记住在每行末尾加上分号（`;`），并在输入每个命令后按*Enter*键。

1.  创建了我们的数据库后，我们现在将创建一个 MariaDB 用户。每个用户将由一个用户名和一个与操作系统用户完全独立的密码组成。出于安全考虑，我们将确保数据库的访问仅限于本地主机。要继续，只需通过更改以下命令中的`<username>`、`<password>`和`<database-name>`值来定制命令以反映您的需求：

    ```
    GRANT ALL ON <database-name>.* TO '<username>'@'localhost' IDENTIFIED BY '<password>' WITH GRANT OPTION;

    ```

1.  接下来，让 MariaDB DBMS 知道您的新用户：

    ```
    FLUSH PRIVILEGES;

    ```

1.  现在，只需输入以下命令即可退出 MariaDB shell：

    ```
    EXIT;

    ```

1.  最后，您可以通过以下方式从命令行访问 MariaDB shell 来测试新`<username>`的可访问性：

    ```
    mysql -u <username> -p

    ```

1.  现在回到 MariaDB shell（`MariaDB [(none)]>`），输入以下命令：

    ```
    SHOW DATABASES;
    EXIT;

    ```

## 它是如何工作的...

在本配方过程中，您不仅被展示了如何创建数据库，还展示了如何创建数据库用户。

那么，我们从这次经历中学到了什么？

我们通过使用`mysql`命令以 root 用户身份访问 MariaDB shell 开始了这个操作步骤。这样，我们就可以使用简单的 SQL 函数`CREATE DATABASE`创建一个数据库，并为`<database-name>`字段提供一个自定义名称。我们还指定了`utf8`作为新数据库的字符集，以及`utf8_general_ci`作为排序规则。字符集是数据库中字符的编码方式，而排序规则是一组比较字符集中的字符的规则。由于历史原因，并为了保持 MariaDB 与旧版本服务器的向后兼容性，默认字符集是`latin1`和`latin1_swedish_ci`，但对于任何现代数据库，您应该始终倾向于使用`utf-8`，因为它是国际字符集（非英语字母）最标准和兼容的编码。但是，可以通过使用以下命令来修改此命令，以检查数据库名称是否已在使用中：`CREATE DATABASE IF NOT EXISTS <database-name>`。这样，您就可以使用以下命令删除或移除数据库：

```
DROP DATABASE IF EXISTS <database-name>;

```

完成这些操作后，只需通过运行我们的`GRANT ALL`命令为新数据库用户添加适当的权限。在这里，我们为本地主机上的`<username>`提供了通过定义的`<password>`获得完全权限。由于选择了特定的`<database-name>`，因此这种级别的权限将限于该特定数据库，并且使用`<database-name>.*`允许我们将这些规则指定给该数据库中的所有表（使用星号符号）。为了向选定的用户提供特定权限，一般语法是：

```
GRANT [type of permission] ON <database name>.<table name> TO '<username>'@'<hostname>';
```

出于安全考虑，在本操作步骤中，我们将`<hostname>`限制为本地主机，但如果您想授予远程用户权限，则需要更改此值（稍后会看到）。在我们的示例中，我们将`[type of permission]`设置为`ALL`，但您始终可以通过提供单个或以逗号分隔的权限类型列表来决定最小化特权，如下所示：

```
GRANT SELECT, INSERT, DELETE ON <database name>.* TO '<username>'@'localhost';
```

使用前面的技术，以下是可以使用的权限的总结：

+   `ALL`：允许`<username>`值拥有所有可用的权限类型

+   `CREATE`：允许`<username>`值创建新表或数据库

+   `DROP`：允许`<username>`值删除表或数据库

+   `DELETE`：允许`<username>`值删除表行

+   `INSERT`：允许`<username>`值插入表行

+   `SELECT`：允许`<username>`值从表中读取

+   `UPDATE`：允许`<username>`值更新表行

然而，一旦授予了特权，操作步骤就会向您展示我们必须`FLUSH`系统，以便使我们的新设置对系统本身可用。需要注意的是，MariaDB shell 中的所有命令都应以分号（`;`）结尾。完成任务后，我们只需使用`EXIT;`语句退出控制台。

MariaDB 是一个出色的数据库系统，但像所有服务一样，它可能会被滥用。因此，始终保持警惕，并且通过考虑之前的建议，你可以确信你的 MariaDB 安装将保持安全和稳定。

## 还有更多...

创建受限用户是提供数据库访问的一种方式，但如果你的开发团队需要持续访问开发服务器，你可能希望考虑提供一个拥有超级用户权限的通用用户。要实现这一点，只需使用管理员用户 root 登录到 MariaDB shell，然后按照以下方式创建一个新用户：

```
GRANT ALL ON *.* TO '<username>'@'localhost' IDENTIFIED BY '<password>' WITH GRANT OPTION;

```

通过这样做，你将使`<username>`能够添加、删除和管理整个 MariaDB 服务器上的数据库（`*.*`中的星号告诉 MariaDB 将权限应用于数据库服务器上找到的所有数据库及其关联表），但由于管理功能的范围，这个新用户账户将所有活动限制在本地主机。所以简单来说，如果你想为`<username>`提供对任何数据库或任何表的访问权限，始终使用星号（`*`）代替数据库名或表名。最后，每次更新或更改用户权限时，请务必在使用`EXIT;`命令退出 MariaDB shell 之前使用`FLUSH PRIVILEGES`命令。

### 审查和撤销权限或删除用户

除非用户账户正在使用，否则保留活动状态并不是一个好主意，因此你首先在 MariaDB shell 中（使用管理员用户 root 登录）考虑的是通过输入以下内容来审查它们当前的状态：

```
SELECT HOST,USER FROM mysql.user WHERE USER='<username>';
```

完成此操作后，如果你打算`REVOKE`权限或从此处列出的用户中删除用户，你可以使用`DROP`命令来执行此操作。首先，你应该审查感兴趣的用户拥有的权限，通过运行：

```
SHOW GRANTS FOR '<username>'@'localhost';
```

你现在有两种选择，首先是撤销用户的权限，如下所示：

```
REVOKE ALL PRIVILEGES, GRANT OPTION FROM '<username>'@'localhost';
```

然后你可以选择重新分配权限，使用主配方中提供的公式，或者你可以决定通过输入以下内容来删除用户：

```
DROP USER '<username>'@'localhost';
```

最后，使用`FLUSH PRIVILEGES;`以通常的方式更新所有你的权限，然后在使用`EXIT;`命令退出 shell 之前。

# 允许远程访问 MariaDB 服务器

除非你正在运行 MariaDB 数据库服务器来驱动同一服务器硬件上的本地 Web 应用程序，否则如果禁止远程访问数据库服务器，大多数工作环境将变得毫无用处。在许多 IT 环境中，你会发现高可用性、集中式的专用数据库服务器在硬件上进行了优化（例如，大量的 RAM），并托管多个数据库，允许从外部到服务器的数百个并行连接。在本配方中，我们将向你展示如何使远程连接到服务器成为可能。

## 准备就绪

要完成此配方，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装。预计 MariaDB 服务器已经安装并运行，并且您已经阅读并应用了*管理 MariaDB 数据库*配方，以了解权限以及如何测试（本地）数据库连接。

## 如何做到这一点...

在我们的示例中，我们希望从同一网络中的客户端计算机（IP 地址为`192.168.1.33`）访问 IP 地址为`192.168.1.12`的 MariaDB 数据库服务器。请根据您的需求适当更改：

1.  首先，以 root 身份登录到您的 MariaDB 数据库服务器，并为传入的 MariaDB 连接打开防火墙：

    ```
    firewall-cmd --permanent --add-service=mysql && firewall-cmd --reload

    ```

1.  之后，我们需要创建一个可以远程连接到我们的 MariaDB 服务器的用户账户（因为我们已经阻止`root`这样做，以增强安全性），使用 MariaDB 命令行界面`mysql`以用户`root`登录数据库服务器，并输入以下 MariaDB 语句（将`XXXX`替换为您选择的密码，也可以根据需要调整用户名和客户端的远程 IP 地址——在我们的例子中，客户端的 IP 地址是`192.168.1.33`）：

    ```
    GRANT SELECT ON mysql.user TO 'johndoe'@'192.168.1.33' IDENTIFIED BY 'XXXX';
    FLUSH PRIVILEGES;EXIT;

    ```

1.  现在我们可以从我们网络中 IP 地址为`192.168.1.33`的客户端计算机测试连接。这台计算机需要安装 MariaDB shell（在 CentOS 7 客户端上，安装软件包`mariadb`），并且需要能够 ping 通运行 MariaDB 服务的服務器（在我们的示例中，IP 为`192.168.1.12`）。您可以通过使用以下命令测试与服务器的连接（成功的话，这将打印出`mysql`用户表的内容）：

    ```
    echo "select user from mysql.user" | mysql -u johndoe -p mysql -h 192.168.1.12

    ```

## 工作原理...

我们的旅程始于通过使用 firewalld 预定义的 MariaDB 服务打开标准的 MariaDB 防火墙端口 3306，该服务在 CentOS 7 上默认是禁用的。之后，我们配置了允许访问数据库服务器的 IP 地址，这是在数据库级别使用 MariaDB shell 完成的。在我们的示例中，我们使用`GRANT SELECT`命令允许用户`johndoe`在客户端 IP 地址`192.168.1.33`和密码`'XXXX'`访问名为`mysql`的数据库和用户表，仅进行`SELECT`查询。请记住，在这里您也可以在`<hostname>`字段中使用通配符`%`（表示任何字符）。例如，为了定义 C 类网络中的任何可能的主机名组合，您可以使用`%`符号，如下所示`192.168.1.%`。授予对`mysql.user`数据库和表的访问权限仅用于测试目的，您应该在完成测试后使用以下命令从该访问权限中删除用户`johndoe`：`REVOKE ALL PRIVILEGES`, `GRANT OPTION FROM 'johndoe'@'192.168.1.33';`。如果您愿意，也可以删除用户`DROP USER 'johndoe'@'192.168.1.33';`，因为我们不再需要它了。

# 安装 PostgreSQL 服务器和管理数据库

在本食谱中，我们不仅将学习如何在服务器上安装 PostgreSQL DBMS，还将学习如何添加新用户并创建我们的第一个数据库。PostgreSQL 被认为是世界上最先进的开源数据库系统。它以稳定、可靠和精心设计的系统而闻名，完全能够支持高事务和关键任务应用程序。PostgreSQL 是 Ingres 数据库的后代。它由来自世界各地的大量贡献者社区驱动和维护。它可能不如 MariaDB 灵活或普及，但由于 PostgreSQL 是一个非常安全的数据库系统，在数据完整性方面表现出色，因此本食谱的目的是向您展示如何开始探索这个被遗忘的朋友。

## 准备工作

为了完成本食谱，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，您选择的基于控制台的文本编辑器，以及互联网连接以便下载额外的软件包。预计您的服务器将使用静态 IP 地址。

## 如何操作...

PostgreSQL（也称为 Postgres）是一个对象关系数据库管理系统。它支持大部分 SQL 标准，并且可以通过服务器管理员以多种方式进行扩展。然而，为了开始，我们必须首先安装必要的软件包：

1.  首先以 root 身份登录您的服务器，然后输入：

    ```
    yum install postgresql postgresql-server

    ```

1.  安装数据库系统后，我们现在必须通过输入以下命令在启动时启用数据库服务器：

    ```
    systemctl enable postgresql

    ```

1.  完成上述步骤后，按照以下方式初始化数据库系统：

    ```
    postgresql-setup initdb

    ```

1.  现在通过启动数据库服务器来完成此过程：

    ```
    systemctl start postgresql

    ```

1.  现在为您的`postgres`管理员设置一个新的初始密码。由于默认的`postgres`用户目前使用的是对等认证，我们需要以`postgres`用户身份执行任何与 Postgres 相关的命令：

    ```
    su - postgres -c "psql --command '\password postgres'"

    ```

1.  为了消除`postgres`用户必须在系统用户基础上登录才能执行如`psql`等与 Postgres 相关的命令的要求，并允许使用数据库用户账户登录，我们需要将`localhost`的认证方法从`peer`更改为`md5`。您可以手动执行此操作，或者使用`sed`工具，如下所示，首先备份文件：

    ```
    cp /var/lib/pgsql/data/pg_hba.conf /var/lib/pgsql/data/pg_hba.conf.BAK
    sed -i 's/^\(local.*\)peer$/\1md5/g' /var/lib/pgsql/data/pg_hba.conf

    ```

1.  接下来，我们必须重启`postgresql`服务以应用我们的更改：

    ```
    systemctl restart postgresql

    ```

1.  现在，您将能够使用用户`postgres`登录到您的 Postgres 服务器，而无需先登录`postgres` Linux 系统用户：

    ```
    psql -U postgres

    ```

1.  要退出 shell（`postgres=#`），请输入以下命令（然后按*回车*键）：

    ```
    \q

    ```

1.  现在，我们将发出一个 shell 命令来创建一个新的数据库用户，通过将 `<username>` 替换为适合您自己需求的相应用户名（在提示时输入新用户的密码，重复它，然后输入管理员用户 `postgres` 的密码以应用这些设置）：

    ```
    createuser -U postgres -P <username>

    ```

1.  现在，也在 shell 中创建您的第一个数据库，并将其分配给我们新用户，通过将 `<database-name>` 和 `<username>` 的值替换为更适合您需求的值（输入 `postgres` 用户的密码）：

    ```
    createdb -U postgres <database-name> -O <username>

    ```

1.  最后，通过打印所有数据库名称来测试您是否可以使用新用户访问 Postgres 服务器：

    ```
    psql -U <username> -l

    ```

## 它是如何工作的...

PostgreSQL 是一种对象关系型数据库管理系统，它适用于所有 CentOS 服务器。虽然 Postgres 可能不如 MariaDB 常见，但其架构和丰富的功能确实使其成为许多关注数据完整性的公司的吸引解决方案。

那么我们从这次经历中学到了什么？

我们从这个配方开始，通过使用 `yum` 安装必要的服务器和客户端 `rpm` 包。完成此操作后，我们在启动时使 Postgres 系统可用，然后使用 `postgresql-setup initdb` 命令初始化数据库系统。我们通过启动数据库服务完成了这个过程。在下一阶段，我们被要求为 Postgres 管理员用户设置密码以加强系统安全性。默认情况下，`postgresql` 包创建一个名为 `postgres` 的新 Linux 系统用户（也用作访问我们的 Postgres DBMS 的管理员 Postgres 用户帐户），通过使用 `su - postgres - c`，我们能够以 `postgres` 用户身份执行 `psql` 命令，这在安装时是强制性的（这称为对等身份验证）。

设置管理员密码后，为了更像 MariaDB shell 类型的登录过程，其中每个数据库用户（包括管理员 `postgres` 用户）都可以使用数据库 `psql` 客户端的用户 `-U` 参数登录，我们将这种 `peer` 身份验证更改为 `md5` 数据库密码身份验证，用于本地主机在 `pg_hba.conf` 文件中（请参阅下一个配方）。重新启动服务后，我们然后使用 Postgres 的 `createuser` 和 `createdb` 命令行工具创建一个新的 Postgres 用户并将其连接到新数据库（我们需要为 `postgres` 用户提供 `-U` 参数，因为只有他有权限）。最后，我们向您展示了如何使用新用户使用 `-l` 标志（列出所有可用数据库）测试与数据库的连接。此外，您可以使用 `-d` 参数使用以下语法连接到特定数据库：`psql -d <database-name> -U <username>`。

## 还有更多...

除了使用`createuser`或`createdb`Postgres 命令行工具（正如我们在这个示例中向你展示的那样）来创建数据库和用户之外，你还可以使用 Postgres shell 来完成相同的操作。实际上，这些命令行工具实际上只是 Postgres shell 命令的包装器，两者之间没有实质性的区别。`psql`是用于在 Postgres 服务器上输入 SQL 查询或其他命令的主要命令行客户端工具，类似于本章中另一个示例中向你展示的 MariaDB shell。在这里，我们将使用名为`template1`的模板启动`psql`，这是用于开始构建数据库的样板（或默认模板）。登录后（`psql -U postgres template1`），输入管理员密码，你应该会看到交互式 Postgres 提示符（`template1=#`）。现在，要在`psql` shell 中创建一个新用户，请输入：

```
CREATE USER <username> WITH PASSWORD '<password>';

```

要创建数据库，请输入：

```
CREATE DATABASE <database-name>;

```

将最近创建的数据库的所有权限授予新用户的选项是：

```
GRANT ALL ON DATABASE <database-name> to <username>;

```

要退出交互式 shell，请使用：`\q`，然后按*回车*键。

完成这个示例后，你可以说你不仅知道如何安装 PostgreSQL，而且这个过程还突显了这个数据库系统与 MariaDB 之间的一些简单的架构差异。

# 配置对 PostgreSQL 的远程访问

在这个示例中，我们将学习如何配置对默认情况下禁用的 Postgres 服务器的远程访问。Postgres 使用一种称为基于主机的身份验证的方法，这个示例的目的是向你介绍其概念，以便为你提供运行安全可靠的数据库服务器所需的访问权限。

## 准备工作

要完成这个示例，你需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，以及你选择的文本编辑器。预计 PostgreSQL 已经安装并正在运行。

## 如何操作...

在前面的示例中，我们已经使用`sed`修改了基于主机的身份验证配置文件`pg_hba.conf`，以管理我们的 Postgres 客户端身份验证，从对等模式更改为`md5`。在这里，我们将对其进行更改，以管理对我们的 Postgres 服务器的远程访问。

1.  首先，以 root 身份登录，并打开防火墙以允许任何传入的 PostgreSQL 连接到服务器：

    ```
    firewall-cmd --permanent --add-service=postgresql;firewall-cmd --reload

    ```

1.  现在，在你的首选文本编辑器中打开基于主机的身份验证配置文件，方法是输入：

    ```
    vi /var/lib/pgsql/data/pg_hba.conf

    ```

1.  滚动到文件末尾，并添加以下行，使这些行读作如下内容（将`XXX.XXX.XXX.XXX/XX`值替换为你想要授予访问权限的网络地址。例如，如果你的服务器 IP 地址是`192.168.1.12`，那么网络地址将是`192.168.1.0/24`）：

    ```
    host    all          all         XXX.XXX.XXX.XXX/XX    md5

    ```

1.  完成后，只需以通常的方式保存并关闭文件，然后通过输入打开主 Postgres 配置文件：

    ```
    vi /var/lib/pgsql/data/postgresql.conf

    ```

1.  将以下行添加到文件末尾：

    ```
    listen_addresses = '*'
    port = 5432

    ```

1.  完成后，以通常的方式保存文件，然后通过输入以下命令重新启动数据库服务器：

    ```
    systemctl restart postgresql

    ```

1.  在同一网络中的任何其他计算机上（由之前设置的`XXX.XXX.XXX.XXX/XX`值定义），您现在可以使用`psql` shell 测试与您的 Postgres 服务器的远程连接是否正常工作（如果您的客户端计算机是 CentOS，您需要使用`yum install postgresql`安装它），通过远程登录到服务器并打印出一些测试数据。在我们的例子中，Postgres 服务器正在运行，IP 地址为`192.168.1.12`。

    ```
    psql -h 192.168.1.12 -U <username> -d <database-name>

    ```

## 它是如何工作的...

PostgreSQL 是一个安全可靠的数据库系统，但我们访问它的方式（无论是远程还是本地）常常会引起混淆。本食谱的目的是揭开基于主机的认证的神秘面纱，并提供一个易于使用的解决方案，帮助您让系统运行起来。

那么我们从这次经历中学到了什么？

我们首先在防火墙中打开 Postgres 服务的标准端口，以便首先从任何远程计算机建立连接。然后，我们使用最喜欢的文本编辑器打开名为`pg_hba.conf`的 Postgres 基于主机的认证配置文件。请记住，我们已经在之前的食谱中将所有本地连接从`对等`更改为`md5`认证，以提供基于用户的认证。插入的主机记录行指定了连接类型、数据库名称、用户名、客户端 IP 地址范围和认证方法。虽然许多之前的命令可能已经理解，但重要的是要认识到有几种不同的认证方法：

+   **信任**: 无条件允许连接，并允许任何人无需密码即可连接到数据库服务器。

+   **拒绝**: 允许数据库服务器无条件拒绝连接，在过滤某些 IP 地址或某些主机时，这一功能仍然很有用。

+   **md5**: 意味着客户端需要提供一个 MD5 加密的密码进行认证。

+   **对等和标识**: 如果客户端登录的 Linux 用户名在系统中作为数据库用户被找到，则授予访问权限。标识用于远程连接，而对等用于本地连接。

完成这项任务后，我们保存并关闭文件，然后打开位于`/var/lib/pgsql/data/postgresql.conf`的主 PostgreSQL 配置文件。你可能知道也可能不知道，除非服务器以适当的`listen_addresses`值启动，否则远程连接是不可能的。默认设置将此设置为本地回环地址，因此有必要允许数据库服务器监听所有网络接口（用星号或`*`表示）以接收 5432 端口的 Postgres 连接。完成后，我们只需保存文件并重新启动数据库服务器。

总有更多的东西要学习，但通过完成这个配方，你不仅对基于主机的认证有了更好的理解，而且你还有能力在本地和远程访问你的 PostgreSQL 数据库服务器。

# 安装 phpMyAdmin 和 phpPgAdmin

使用 MariaDB 或 Postgres 命令行 shell 足以执行基本的数据库管理任务，例如用户权限设置或创建简单的数据库，正如我们在本章中向你展示的那样。随着你的模式和表之间的关系变得更加复杂，以及你的数据增长，你应该考虑使用一些图形数据库用户界面以获得更好的控制和工作性能。对于新手数据库管理员来说也是如此，因为这样的工具为你提供了语法高亮和验证，有些工具甚至有数据库的图形表示（例如，显示实体关系模型）。在这个配方中，我们将向你展示如何安装市场上最流行的两个图形开源数据库管理软件，即`phpMyadmin`和`phpPgAdmin`，它们是基于 Web 的浏览器应用程序，用 PHP 编写。

## 准备就绪

要完成这个配方，你需要具备以下条件：CentOS 7 操作系统的有效安装，具有 root 权限，你选择的基于控制台的文本编辑器，以及互联网连接以便下载额外的软件包。预计你的 MariaDB 或 PostgreSQL 服务器已经按照本章中的配方运行。此外，你需要一个运行中的 Apache 网络服务器，该服务器已安装 PHP，并且必须可以从你的私人网络中的所有计算机访问以部署这些应用程序（请参阅第十二章，*提供网络服务*以获取说明）。此外，你需要启用 EPEL 存储库以安装正确的软件包（请参阅第四章中的配方*使用第三方存储库*，*使用 YUM 管理软件包*）。最后，你需要在你的网络中有一台计算机，该计算机具有图形窗口管理器和现代网络浏览器，以便访问这些网络应用程序。

## 如何做到这一点...

在这个配方中，我们将首先向你展示如何安装和配置`phpMyAdmin`以进行远程访问，然后是如何为`phpPgAdmin`做同样的事情。

### 安装和配置 phpMyAdmin

要安装和配置 phpMyAdmin，请执行以下步骤：

1.  输入以下命令以安装所需的软件包：

    ```
    yum install phpMyAdmin

    ```

1.  现在创建主`phpMyadmin`配置文件的副本：

    ```
    cp /etc/httpd/conf.d/phpMyAdmin.conf /etc/httpd/conf.d/phpMyAdmin.conf.BAK

    ```

1.  接下来，打开主`phpMyAdmin.conf`配置文件，并在您想要授权访问 Web 应用程序的已定义子网的网络地址下添加一行`Require ip XXX.XXX.XXX.XXX/XX`，例如，在`Require ip 127.0.0.1`行下添加`Require ip 192.168.1.0/24`。您需要在文件中执行此操作两次，或者可以使用`sed`自动执行此操作，如下所示。在命令行中，根据您自己的子网网络地址相应地定义环境变量`NET=`。

    ```
    NET="192.168.1.0/24"

    ```

1.  然后输入以下行以将更改应用到配置文件：

    ```
    sed -i "s,\(Require ip 127.0.0.1\),\1\nRequire ip $NET,g" /etc/httpd/conf.d/phpMyAdmin.conf

    ```

1.  之后，重新加载您的 Apache 服务器，现在您应该能够从子网中的任何其他计算机浏览到运行 Web 应用程序的服务器的 IP 地址的`phpMyAdmin`网站，例如`192.168.1.12`（使用 MariaDB 管理员用户 root 或其他数据库用户登录）：

    ```
    http://192.168.1.12/phpMyAdmin

    ```

### 安装和配置 phpPgAdmin

以下是安装和配置 phpPgAdmin 的步骤：

1.  输入以下命令以安装所需的软件包：

    ```
    yum install phpPgAdmin

    ```

1.  在编辑`phpPgAdmin`主配置之前，首先对其进行备份：

    ```
    cp /etc/httpd/conf.d/phpPgAdmin.conf /etc/httpd/conf.d/phpPgAdmin.conf.BAK

    ```

1.  允许远程访问`phpPgAdmin`与`phpMyAdmin`非常相似。在这里，您也可以在`phpPgAdmin.conf`文件中的`Require local`行下添加一行`Require ip XXX.XXX.XXX.XXX/XX`，其中包含您定义的子网网络地址，或者使用`sed`实用程序自动执行此操作：

    ```
    NET="192.168.1.0/24"
    sed -i "s,\(Require local\),\1\nRequire ip $NET,g" /etc/httpd/conf.d/phpPgAdmin.conf

    ```

1.  重启 Apache 并浏览到`phpPgAdmin`主页：

    ```
    http://192.168.1.12/phpPgAdmin

    ```

## 它是如何工作的...

在这个相当简单的教程中，我们向您展示了如何在同一台服务器上安装两个最流行的 MariaDB 和 Postgres 的图形化管理工具，这些工具作为 Web 应用程序在您的浏览器中运行（使用 PHP 编写），并启用了对它们的远程访问。

那么我们从这次经历中学到了什么？

使用`yum`包管理器安装`phpMyAdmin`以管理 MariaDB 数据库和`phpPgAdmin`以管理 Postgres 数据库就像安装相应的`rpm`包一样简单。由于这两个工具在官方的 CentOS 7 仓库中找不到，您需要在能够访问和安装这些包之前启用第三方仓库 EPEL。默认情况下，在安装这两个 Web 应用程序时，拒绝来自服务器本身（仅本地）以外的任何连接。由于我们希望从网络中的不同计算机访问它，因此安装了 Web 浏览器后，您需要首先允许远程连接。对于这两个 Web 应用程序，可以使用 Apache 的`Require ip`指令来实现，该指令是 Apache 的`mod_authz_core`模块的一部分。在`phpMyAdmin`和`phpPgAdmin`的配置文件中，我们定义了一个完整的子网，例如`192.168.1.0/24`，以允许连接到服务器，但您也可以在这里使用单个 IP 地址，您希望允许访问该地址。`sed`命令将这些重要的`Require`行插入到配置文件中，但如前所述，如果您愿意，也可以通过使用您选择的文本编辑器编辑这些文件来手动完成。重新加载 Apache 配置后，您就可以使用本食谱中显示的两个 URL 浏览到网页。在两个网站的首页上，您可以使用任何数据库用户登录，无需为他们启用远程权限；任何具有本地权限的用户都足够了。

总之，我们可以说我们只向您展示了两种管理工具的基本配置。总有更多需要学习的内容；例如，您应该考虑使用 SSL 加密来保护 PHP 网站，或者配置您的实例以连接到不同的数据库服务器。此外，如果您更喜欢使用桌面软件来管理数据库，可以查看开源的 MySQL Workbench 社区版，该版本可以从官方 MySQL 网站下载，适用于所有主要操作系统（Windows、OS X、Linux）。
