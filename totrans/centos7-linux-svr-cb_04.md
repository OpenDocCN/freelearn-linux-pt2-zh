# 第四章。使用 YUM 管理软件包

在本章中，我们将涵盖以下主题：

+   使用 YUM 更新系统

+   使用 YUM 搜索软件包

+   使用 YUM 安装软件包

+   使用 YUM 删除软件包

+   保持 YUM 的清洁和整洁

+   了解您的优先级

+   使用第三方仓库

+   创建 YUM 仓库

+   使用 RPM 软件包管理器

# 引言

本章是一系列教程的集合，提供了对扩展服务器所需的工具的回顾。软件包管理是任何基于 Linux 的系统的核心，本章的目的强调了在基于 CentOS 的服务器上管理软件包所需的关键工具。

# 使用 YUM 更新系统

在本教程中，我们将探讨**Yellowdog Updater, Modified**（**YUM**）软件包管理器在运行系统更新方面的角色。每隔一段时间，您可能会意识到有更新，或者可能只是想了解是否存在更新。应用补丁和更新是每个服务器管理员的常规任务，一个最新的系统可以帮助提高或确保您的服务器的安全性，因为软件错误和漏洞一直在被发现，并且必须及时修复。在本教程中，您将学习如何借助 YUM 实现这一点。

## 准备工作

要完成本教程，您需要具备具有 root 权限的 CentOS 7 操作系统的正常安装，您选择的基于控制台的文本编辑器，以及连接到互联网以便于下载额外软件包的能力。

## 如何操作...

您可以根据自己的选择的时间表经常运行本教程，但应该经常进行，完全了解有时某些更新可能需要完全系统重启：

1.  以 root 身份登录并检查是否有任何适用于您已安装软件包的更新。为此，请登录并输入以下命令：

    ```
    yum check-update

    ```

1.  如果没有可用的更新，那么更新过程将结束，不需要进行进一步的工作。然而，如果有可用的更新，YUM 现在将返回来自系统已知仓库的所有软件包更新的列表。要完成更新过程，请输入以下命令：

    ```
    yum -y update

    ```

1.  通过使用`-y`标志，前面的命令现在将绕过确认交易摘要的需要，您的系统现在将立即进行更新过程。完成后，您将获得一个最终报告，该报告标识了已安装的依赖项和已更新的软件包。

1.  通常情况下，不需要进一步的工作，您可以恢复正常的操作。然而，如果安装了新的内核，或者进行了重要的安全更新，可能需要重启系统以使新的更改生效。为此，请输入以下命令：

    ```
    reboot

    ```

    ### 注意

    尽管关于更新是否需要在实践中进行全面系统重启存在很多争议，但这只有在进行内核更新后才需要考虑，内核更新是对`glibc`和在启动过程中激活的特定安全功能的更新。

## 它是如何工作的...

YUM 是 CentOS 默认的包管理器，其职责之一是自动计算哪些包可能需要更新，哪些依赖项是必需的，并以非常简单的方式管理整个系统更新过程。

那么，我们从这次经历中学到了什么？

我们通过使用`yum`命令和`check-update`选项检查系统是否有可用更新来开始这个配方。这样，YUM 现在将检查中央存储库以确认我们的系统是否有适用的更新。存储库是一个包含预制软件包和实用程序的远程目录或网站。YUM 将使用此设施自动定位和获取正确的**Red Hat Package Manager**（**RPM**）和依赖项，如果有可用更新，YUM 将相应地响应，提供有关哪些包和依赖项可用的完整摘要。因此，YUM 是一个非常有用的工具，毫无疑问，其机制确实简化了与包管理相关的流程，因为它可以与存储库对话，这使我们不必手动查找和安装新应用程序或更新。如果有可用更新，输出将向我们显示受影响的包，然后我们可以继续使用 YUM 的`update`参数更新系统。在这种情况下，前面的命令包括`-y`标志。这是为了绕过同意交易摘要的需要，并确认我们已经在运行前面的检查后同意进行这些更新。否则，你只需使用*Y*键确认请求。

## 还有更多...

你还可以使用更新参数来更新单个包而不是整个系统，只需提供包名，例如：`yum update package_name`。YUM 将确保在安装应用程序时满足所有要求，并自动安装系统上尚未存在的任何依赖项的包。然而，我确信你会很高兴听到这一点，如果新应用程序有与现有软件冲突的要求，YUM 将中止过程而不对系统进行任何更改。如果你想使用特定时间间隔自动更新系统，可以安装`yum-cron`包，该包可以高度定制，但超出了本书的范围。安装后启动，使用`man yum-cron`。

# 使用 YUM 搜索包

在本食谱中，我们将研究使用 YUM 查找包的作用。YUM 是为了改进 RPM 软件包的安装而开发的，它用于访问提供服务器提供的全方位服务的不断增长的数学包列表。YUM 使用起来很简单，但如果你不确定包的名称，那么作为服务器管理员的工作就会变得更加困难。为了克服这一点，YUM 维护了广泛的发现工具，本食谱的目的是向您展示如何使用此功能来搜索各种存储库并找到您需要的包。

## 准备工作

要完成本食谱，您需要具有 root 权限的 CentOS 7 操作系统的工作安装、您选择的基于控制台的文本编辑器以及互联网连接。

## 如何做到这一点...

本食谱将向您展示如何通过调用 YUM 的搜索选项来找到一个或多个包。为此，您需要以 root 用户身份登录并完成以下过程：

1.  要搜索单个包，请将 `keyword` 值替换为适当的短语、字符串或参数，然后键入以下内容：

    ```
    yum search keyword

    ```

1.  等待搜索结果的摘要，当生成列表时，您可以通过简单地将 `package_name` 替换为适当的值来查询任何显示的包：

    ```
    yum info package_name

    ```

1.  如果前面的结果令人满意，并且您想查看与该包相关的依赖项列表，请键入以下内容：

    ```
    yum deplist package_name

    ```

## 它是如何工作的...

使用 YUM 搜索包的方式与在 **万维网**（**WWW**）上搜索任何内容的方式相同。您可以搜索的单词类型可以像您喜欢的那样具体或一般。它们甚至可以由完整或部分单词组成；找到您可能感兴趣的包后，您会注意到，本食谱还向您展示了如何发现有关该包的更多信息。

那么，我们从这次经历中学到了什么？

YUM 维护了广泛的搜索功能，允许您通过关键字、包名和路径名查询包。例如，如果您想找到编译 C、Objective-C 和 C++ 代码的正确包，可以使用 `yum search compiler` 查询。在命令行上使用这些搜索词时，会出现许多相关结果，每个包都附有简短描述，使我们能够通过简单的排除过程来选择最明显或最相关的值。考虑到这一点，您可以使用 `info` 参数向 YUM 查询以了解有关某些包的更多信息。此选项显示了完整的包详细信息以及包旨在提供的功能的详细描述。一般来说，您可能不需要知道更多细节。

然而，在某些情况下，您可能想知道该软件包如何与整个服务器交互（特别是如果您正在处理源安装或修复损坏的软件包），因此我们可以使用 YUM 的`deplist`参数，它可以提供相当详细的报告；如果您碰巧有任何损坏的软件包，您可以使用此输出来详细说明您可能需要或不需要安装哪些依赖项来解决潜在问题。当调试依赖关系或处理基于源的安装时，此命令特别有用。

## 还有更多...

有时，您可能不想搜索特定的软件包，而是可能更愿意以目录式格式显示您存储库的内容。同样，这很容易做到，YUM 提供了以下命令来实现这一功能。如果您想简单地列出当前系统使用的存储库中所有可用的软件包，请键入`yum list all`。然而，由于此列表可能非常详尽，您可能更愿意通过使用`yum list all | less`来分页浏览结果。类似地，如果您只想列出系统上当前安装的所有软件，请键入`yum list installed | less`。如果您想确定哪些软件包提供了特定的文件或功能，只需随时运行以下命令，将`your_filename_here`替换为与您自己的需求更相关的内容：`yum provides your_filename_here`。

# 使用 YUM 安装软件包

在本教程中，我们将探讨 YUM 在服务器上安装新软件包的作用。对于每位服务器管理员来说，安装应用程序和服务是一项重要任务。有多种方法可以实现这一点，但最有效的方法涉及使用 YUM 包管理器。YUM 能够搜索任意数量的存储库，自动解决软件包依赖关系，并指定安装一个或多个软件包。YUM 是现代且确定性的方法，用于在服务器上安装软件包，本教程的目的就是向您展示如何做到这一点。

## 准备工作

要完成本教程，您需要具备 CentOS 7 操作系统的有效安装，拥有 root 权限，选择一个基于控制台的文本编辑器，以及连接到互联网以便下载额外的软件包。如果您已经找到了一些有趣的软件包来安装，那么使用*使用 YUM 搜索软件包*教程中的说明来学习这些软件包是很好的。

## 如何做到这一点...

本教程将向您展示如何通过调用 YUM 安装选项来安装一个或多个软件包。为此，您需要以 root 用户身份登录并完成以下过程：

1.  要安装单个软件包，请将`package_name`值替换为适当的值，然后键入以下内容：

    ```
    yum install package_name

    ```

1.  你的系统现在将提供一个交易报告，需要你的批准。因此，当提示时，只需使用*Y*或*N*键并按*Return*键来接受或拒绝交易，如下所示：

    ```
    Is this ok [y/d/N]: y

    ```

1.  如果你拒绝了交易，那么不需要进行进一步的工作，你将退出软件包管理例程。然而，如果你确认了交易，那么请观察安装进度，最终它会显示一个`Complete!`消息。

1.  恭喜！你现在已经成功安装了你选择的软件包。

## 它是如何工作的...

所有软件包都存储在 RPM 软件包文件格式中，而 YUM 的作用是提供对存储在互联网上各种仓库中的这些文件的访问。YUM 是 CentOS 软件包管理背后的力量，它确实使得安装过程变得非常简单，但我们从这次经历中学到了什么？

调用`install`命令后，YUM 将在各个仓库中进行搜索，以找到与所讨论的软件包相关的相关标题和元数据。例如，如果你想安装一个名为`wget`的软件包，你将首先发出这样的`install`命令：`yum install wget`。然后，YUM 将定位该软件包，并生成一个交易摘要，该摘要不仅指示所需的磁盘空间和预期的安装大小，还将指示所请求的软件包所需的任何必要依赖项。YUM 将检查几个不同的仓库（`base`，`extras`，和`updates`），并在解决了任何必要依赖项的需求后，YUM 将要求我们在继续安装过程之前确认请求。因此，正如你所见，通过使用*Y*键，我们将向 YUM 提供执行请求的权限，这将导致下载、验证和安装相关软件包。

## 还有更多...

有时你可能希望一次性安装多个软件包。要实现这一点，只需调用相同的`install`命令，但不是指定单个软件包，而是以形成一个长购物清单的方式列出你可能需要的所有软件包：

```
yum install package_name1 package_name2 package_name3

```

你可以以这种方式安装的软件包数量没有限制，但始终在每个软件包名称之间留一个空格，并保持命令在一行上。对于非常长的安装指令，可能会发生行包装。

你不需要以任何特定顺序列出软件包，请求将以与原始配方完全相同的方式处理，并且在列出交易摘要后，它将保持待定状态，直到确认或拒绝。再次使用*Y*键确认你的请求，以便完成该过程。

# 使用 YUM 删除软件包

在这个食谱中，我们将探讨使用 YUM 的目的，即从你的服务器上移除软件包。在你的服务器生命周期中，某些应用程序和服务可能不再需要。在这种情况下，通常你会想要移除这些软件包以优化你的工作环境，而这个食谱的目的就是向你展示如何做到这一点。

## 准备工作

要完成这个食谱，你需要一个安装了 CentOS 7 操作系统的工作环境，具有 root 权限，选择一个基于控制台的文本编辑器，以及互联网连接。

## 如何操作...

这个食谱将向你展示如何通过调用`yum remove`选项来移除一个或多个软件包。为此，你需要以 root 用户身份登录并完成以下过程：

1.  要移除单个软件包，将`package_name`替换为适当的值，并输入以下内容：

    ```
    yum remove package_name

    ```

1.  等待交易摘要和确认提示显示，然后按*Y*键确认，或按*N*键拒绝交易，如下所示：

    ```
    Is this ok [y/d/N]: y

    ```

1.  如果你拒绝了交易，那么不需要进一步的工作，你将退出 YUM。然而，如果你确认了交易，那么只需观察软件包移除的进度，直到它被确认并打印出`Complete!`消息。

## 它是如何工作的...

不再需要的应用程序可以通过 YUM 移除。这个过程非常直观，类似于安装新软件包，只需要你确认要移除的软件包名称。

那么，我们从这次经历中学到了什么？

调用了`remove`命令后，YUM 将在你的系统中搜索相关软件包；通过阅读软件包头和元数据，它还将确定这将影响哪些依赖项。例如，如果我们想移除名为`wget`的软件包，我们将开始发出`remove`命令，如下所示：`yum remove wget`。然后，YUM 将从你的系统中找到软件包详细信息，并获取一个交易摘要，其中可能包括任何不再需要的必要依赖项。打印出的交易将保持待定状态，直到你指示 YUM 移除相关软件包。确认后，YUM 将完成交易，这将导致移除软件包或软件包。如果摘要提到任何依赖项，你应该格外小心，因为这些可能被其他 RPM 所需。如果你担心某些依赖项应该保留在系统上，通常一个好的做法是结束当前交易，并简单地停用或禁用相关软件。与`install`命令一样，你也可以一次移除多个软件包，软件包名称之间留一个空格：

```
yum remove package_name1 package_name2 package_name3

```

# 保持 YUM 的整洁

在这个步骤中，我们将调查 YUM 在确保工作缓存保持最新方面的作用。作为其典型操作模式的一部分，YUM 将创建一个由元数据和软件包组成的缓存。这些文件非常有用，但随着时间的推移，它们会积累到一定程度，你可能会发现 YUM 行为异常或不如预期。这种情况发生的频率因系统而异，但通常意味着 YUM 缓存系统需要你立即关注。这种情况可能会非常令人沮丧，但这个步骤的目的是提供一个快速解决方案，帮助你清理缓存并将 YUM 恢复到原始工作状态。

## 准备就绪

要完成这个步骤，你需要一个具有 root 权限的 CentOS 7 操作系统的工作安装，一个你选择的基于控制台的文本编辑器，以及一个互联网连接，以便于下载额外的软件包。

## 如何操作...

在我们开始之前，重要的是要意识到，虽然我们正在解决当前的问题，但这个相同的步骤可以按需运行，以保持 YUM 处于最佳工作状态：

1.  我们将从这个步骤开始，让 YUM 清理任何缓存的软件包信息。为此，以 root 身份登录并输入以下内容：

    ```
    yum clean packages

    ```

1.  允许系统响应一段时间，完成后，输入以下命令以删除任何缓存的基于 XML 的元数据：

    ```
    yum clean metadata

    ```

1.  再次等待 YUM 响应，准备就绪后，输入以下命令以删除任何缓存的数据库文件：

    ```
    yum clean dbcache

    ```

1.  接下来，你需要清理所有文件以确认前面的指令，并确保不使用不必要的磁盘空间。为此，输入以下行：

    ```
    yum clean all

    ```

1.  最后，你需要通过输入以下内容来重建 YUM 缓存：

    ```
    yum makecache

    ```

## 它是如何工作的...

YUM 是一个非常强大的工具，以其解决软件包依赖关系和自动化软件包管理过程的能力而闻名，但正如所有事物一样，有时即使是最好的工具也会感到困惑，可能会报告错误或行为异常。解决这个问题相对简单，本步骤中概述的方法也将有助于保持你的软件包管理器在你的操作系统生命周期内保持健康运行状态。

那么，我们从这次经历中学到了什么？

在 YUM 的典型运行过程中，它会在`/var/cache/yum`位置创建一个元数据和包的缓存。这些文件至关重要，但随着它们的大小增长，这个缓存最终会减慢该工具的整体使用速度，甚至可能导致一些问题。为了解决这个问题，我们首先使用以下命令清理当前基于包的缓存，使用 YUM 的`clean packages`参数选项。然后，我们通过执行`clean metadata`命令清理元数据缓存，这将删除任何多余的基于 XML 的文件。YUM 在其正常操作中使用 SQLite 数据库，因此下一步是使用`clean dbcache`参数删除任何剩余的数据库文件。接下来，我们清理所有与启用仓库相关的文件，以回收任何未使用的磁盘空间：`yum clean all`。最后，我们通过使用`makecache`选项重建缓存，将 YUM 恢复到正常工作状态。

## 还有更多...

在典型的服务器上，YUM 是一个出色的工具，它将解决与包依赖关系和包管理相关的最复杂问题。然而，在明知混合了不兼容的仓库或使用了不完整的源的情况下，YUM 可能无法提供帮助。

### 注意

记住，在这种情况下，你应该将以下建议视为仅是临时补救措施。忽视 YUM 提供的任何警告只会导致将来出现更大的问题。

如果发生这种情况，并且错误是基于 RPM 的，作为临时修复，你可以使用以下命令跳过损坏的包：

```
yum -y update --skip-broken

```

这个命令将允许 YUM 继续工作，绕过任何有错误的包，但正如前面所述，这应该被视为仅是临时修复。你应该始终意识到，一个依赖关系损坏的系统不被认为是一个健康的系统。这种情况应该不惜一切代价避免，在这种情况下，修复此类错误应该成为你的首要任务。

# 了解你的优先级

在本食谱中，我们将探讨准备 YUM 管理额外仓库的任务，通过安装一个名为**YUM priorities**的插件。YUM 有能力从各种远程位置搜索、删除、安装、检索和更新包。这些功能使 YUM 成为一个强大的工具，但如果你决定添加额外的第三方仓库，有可能冲突会导致系统不稳定。稳定性是使用 CentOS 操作系统的众多优势之一，本食谱的目的是展示如何在同时允许添加新仓库的情况下保持这种信心。

## 准备就绪

要完成这个食谱，你需要一个具有 root 权限的 CentOS 7 操作系统的正常安装，一个基于控制台的文本编辑器，以及一个互联网连接，以便下载额外的包。

## 如何做...

本食谱将向您展示如何准备 YUM，以便通过安装和配置 YUM 优先级来管理使用一个或多个第三方仓库的过程：

1.  要开始这个食谱，请以 root 身份登录并输入以下内容：

    ```
    yum install yum-plugin-priorities

    ```

1.  确认安装，完成后输入以下内容：

    ```
    vi /etc/yum/pluginconf.d/priorities.conf

    ```

1.  您应该确保此文件指示插件已启用。它应该显示指令`enabled = 1`。通常不需要在此文件中进行任何更改，但如果您已进行任何更改，请在继续之前保存并关闭文件。

1.  现在我们需要为每个仓库建立一个优先级值。这是一个升序的数值，其中最高优先级被赋予最低的数字。为此，按照以下所示打开以下文件：

    ```
    vi /etc/yum.repos.d/CentOS-Base.repo

    ```

1.  在`[base]`部分末尾添加以下行：

    ```
    priority=1

    ```

1.  现在，在`[updates]`部分末尾添加以下行：

    ```
    priority=1

    ```

1.  最后，在`[extras]`部分末尾添加以下行：

    ```
    priority=1

    ```

1.  完成后，保存并关闭文件，然后运行软件包更新：

    ```
    yum update

    ```

## 它是如何工作的...

YUM 优先级是一个简单的插件，它使 YUM 能够决定在安装和更新新软件包时哪些仓库将具有最高优先级。使用此插件将减少软件包混淆的可能性，确保任何特定软件包始终从同一仓库安装或更新。这样，您可以添加无限数量的仓库，并使 YUM 保持对软件包管理的控制。

那么，我们从这次经历中学到了什么？

通过安装`yum-plugin-priorities`软件包并确保在其配置文件中启用它，我们简单地增强了 YUM。然后我们发现优先级是按升序设置的，其中最低值优先于所有其他值。当然，这简化了整个过程，因此我们确保默认仓库被赋予值`1`（`priority=1`）。这将确保默认仓库保持最高优先级，因此当您决定添加其他仓库时，可以为它们分配优先级值 2、3、4…和 10，或更多。另一方面，应该注意的是，我们只在三个主要部分设置了此值：`[base]`、`[updates]`和`[extras]`。简单来说，这只是因为其他部分显示为禁用。例如，您可能已经注意到`/etc/yum.repos.d/CentOS-Base.repo`中的`[centosplus]`部分包含以下行：`enabled=0`，而`[updates]`和`[extras]`部分显示此值为`enabled=1`。当然，如果您打算激活此仓库，则需要为其设置优先级值，但为了本食谱的目的，不需要进行此操作。最后，我们运行了一个简单的 YUM 软件包更新以激活我们的修订设置。

因此，正如我们所见，YUM 优先级是一个极其灵活的软件包，它使您能够确定在扩展安装选项时哪些存储库具有优先权。然而，您应该始终意识到，YUM 优先级可能并不适合您的系统，因为您赋予了它决定哪些软件包将被忽略、哪些软件包将被安装、哪些软件包将被更新以及以何种顺序和从哪个存储库获取它们的权力。对于大多数不倾向于远离典型服务器功能的用户来说，这可能不是立即关注的问题；您甚至可以安全地忽略这个警告。但如果稳定性和安全性是压倒性的关注点，并且您确实打算从外部存储库使用额外的软件包，那么您应该仔细考虑使用此插件，或者至少考虑并研究所使用的第三方存储库的完整性。

# 使用第三方存储库

在本配方中，我们将探讨充分利用 CentOS 可用软件包的愿望，通过安装 EPEL 和 Remi 存储库。CentOS 是一个以稳定性为傲的企业级操作系统，在您的服务器生命周期中，可能并非您需要的每一件软件都能在默认存储库中找到。您也可能需要当前软件的更新软件包，出于这些原因，许多服务器管理员选择安装 EPEL 和 Remi 存储库。这些不是唯一可用的存储库，但由于它们代表了最受欢迎的组合之一，因此本配方的目的是向您展示如何将 EPEL 和 Remi 存储库添加到您的系统中。

## 准备工作

要完成这个配方，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，您选择的基于控制台的文本编辑器，以及连接到互联网以便于下载额外软件包的能力。

## 如何操作...

在我们开始之前，假设您已经遵循了之前的配方，该配方向您展示了如何安装和激活 YUM 优先级。

1.  首先，以 root 身份登录并使用 YUM 安装 EPEL 发布存储库：

    ```
    yum install epel-release

    ```

1.  接下来，从您的家目录中，键入以下命令以下载`remi release` `rpm`软件包：

    ```
    curl -O http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

    ```

    ### 注意

    请注意，当您阅读本文时，此 URL 可能已更改；如果是这样，请进行一些互联网研究，以了解是否有新的 URL 可用。

1.  前面的文件现在应该位于您的家目录中。要继续，请键入以下命令：

    ```
    rpm -Uvh remi-release-7.rpm

    ```

1.  安装完成后，使用您最喜欢的文本编辑器打开 Remi 存储库文件：

    ```
    vi /etc/yum.repos.d/remi.repo

    ```

1.  将`enabled=0`更改为`enabled=1`，并在`[remi]`部分的末尾添加行`priority=10`。

1.  现在，使用您最喜欢的文本编辑器打开 EPEL 存储库文件：

    ```
    vi /etc/yum.repos.d/epel.repo

    ```

1.  再次，如果未自动设置，请将`enabled=0`更改为`enabled=1`，并在`[epel]`部分中添加行`priority=10`。

1.  最后，按照以下方式更新 YUM：

    ```
    yum update

    ```

1.  如果有可用更新，请选择*Y*继续。完成更新过程后，您现在将能够从 Remi 和 EPEL 存储库下载和安装软件包，作为默认使用的补充。

## 它是如何工作的...

为了使用和享受第三方存储库的好处，您首先需要使用 YUM 和 RPM 包管理器安装并启用它。

那么，我们从这次经历中学到了什么？

开始执行配方后，安装 Remi 和 EPEL 存储库的任务是一个非常顺畅的过程。虽然使用 YUM 安装 EPEL 存储库非常安全，不会对系统造成影响，但 Remi 存储库的先前 URL 由存储库所有者自行维护，因此您应始终确保它们是最新的。然而，一旦获得了必要的存储库设置文件，接下来就是使用基于 RPM 的命令在系统上安装所有必要的存储库文件。完成这一步后，我们还需要打开每个已安装存储库的相关配置文件并启用它们（通过将`enabled=0`更改为`enabled=1`），并设置优先级值（`priority=10`）。前一个值仅会打开存储库，而后一个值将由 YUM 用于在我们调用`update`命令时正确识别哪些存储库最为合适。正如在前一个关于 YUM 优先级的配方中所讨论的，一个简单的经验法则是记住这句话：“数字越低，优先级越高。”这本身（取决于您的目的）可能不是一件坏事，但对于本配方的目的，它表明默认的 CentOS 存储库应该优先于所有其他存储库。当然，您可能不同意这一点，并且确实没有任何阻止您将相同的优先级规则应用于第三方供应商，但我确实在您深入之前提醒您，特别是在这是为了关键任务生产服务器的情况下。请记住，如果所有优先级值都相同，那么 YUM 将默认尝试下载最新版本。

将 Remi 和 EPEL 的优先级设置得比现有的基于 CentOS 的仓库更高，是基于考虑安全更新的需要。除非你另有决定，否则始终建议基础文件应首先来自 CentOS。这包括但不限于内核更新、SELinux 及相关软件包。第三方仓库应用于获取无法从原始来源获得的额外软件包，或访问可能不适用于 CentOS 基础版本的特定更新。这可能包括 Apache、MariaDB 或 PHP 等软件包。最后，你可能会注意到 Remi 和 EPEL 仓库共享相同的优先级值。这是有意为之，因为这些仓库通常被视为合作伙伴。然而，如果你决定开始混合使用仓库，或使用此方法作为安装此处未提及的其他仓库的途径，那么你应该始终进行研究，并逐个评估每个第三方仓库。Remi 和 EPEL 仓库非常受欢迎，所以如果你确实打算添加更多第三方资源，请围绕主题进行阅读，谨慎选择你的仓库，并保持忠诚。

## 还有更多...

CentOS 7 有许多其他有趣的仓库，例如专注于硬件相关软件包的 ELRepo，如文件系统驱动、图形驱动、网络驱动、声音驱动以及摄像头或视频驱动。访问[`elrepo.org`](http://elrepo.org)学习如何安装和访问它。

# 创建 YUM 仓库

如果你在本地网络中维护多个 CentOS 服务器，并希望节省互联网带宽或加快重复下载相同远程仓库软件包的速度，或者处于一个非常受限的网络环境中，你的客户端无法访问任何远程 CentOS 仓库，你可能需要考虑运行自己的 YUM 仓库。拥有自己的仓库也是一个很好的解决方案，如果你想向本地用户推出一些自定义或非官方的 RPM 软件包（例如内部配置文件或程序），或者你只是想创建一个官方 CentOS 7 仓库的镜像站点。在本方法中，我们将向你展示如何设置你自己的第一个 YUM CentOS 7 仓库，以及如何为你的本地网络提供服务。

## 准备就绪

要完成此配方，您需要一个具有 root 权限的 CentOS 7 操作系统的有效安装，您选择的基于控制台的文本编辑器，以及连接到互联网以方便下载额外软件包。为了使此配方工作，您还需要将 CentOS 7 Everything DVD iso 文件图像放置在服务器的根目录中，如果您还没有下载它，请参考第一章中的第一个配方，*安装 CentOS*（但下载最新的`CentOS-7-x86_64-Everything-XXXX.iso`文件而不是最小 iso 文件）。此外，我们需要一个正在运行的 Apache Web 服务器来共享我们的 YUM 仓库到我们的本地网络；请阅读第十二章中的第一个配方，*提供 Web 服务*，以了解如何设置它。

## 如何操作...

要创建我们自己的 YUM 仓库，我们需要`createrepo`程序，该程序在 CentOS 7 上默认未安装。让我们从安装它开始我们的旅程。在本例中，我们将使用 IP 地址`192.168.1.7`作为我们的 YUM 仓库服务器：

1.  以 root 身份登录到您的服务器并安装以下软件包：

    ```
    yum install createrepo

    ```

1.  接下来，对于您想要共享的每个仓库，在 Apache Web 根目录下的`/var/www/html/repository/`下创建一个子文件夹，当 Apache 运行时，该文件夹将公开可用；例如，要共享完整的 CentOS 7 `Everything`仓库包，您可以使用：

    ```
    mkdir -p /var/www/html/repository/centos/7.1

    ```

1.  现在，将您选择的 RPM 包文件放入此处创建的仓库文件夹中。在我们的示例中，我们将把`Everything` iso 文件中的所有 RPM 包放入我们新的本地仓库位置，之后我们将把 iso 文件的内容挂载到文件系统上：

    ```
    mount ~/CentOS-7-x86_64-Everything-1503-01.iso /mnt/
    cp -r /mnt/Packages/* /var/www/html/repository/centos/7.1/

    ```

1.  之后，我们需要为复制到 Apache Web 根目录中的所有新文件更新 SELinux 安全上下文：

    ```
    restorecon -v -R /var/www/html

    ```

1.  现在，对于我们想要设置的每个仓库，运行以下命令：

    ```
    createrepo --database /var/www/html/repository/centos/7.1

    ```

1.  恭喜，您现在已经成功创建了您的第一个 YUM 仓库，该仓库可以通过正在运行的 Apache Web 服务器从同一网络中的任何计算机访问。为了测试它，以 root 身份登录到任何其他可以 ping 通我们的仓库服务器的 CentOS 7 系统，并将我们的新仓库添加到其 YUM 仓库配置目录中：

    ```
    vi /etc/yum.repos.d/myCentosMirror.repo

    ```

1.  将以下内容添加到这个空文件中（根据您的需要适当更改`baseurl`）：

    ```
    [myCentosMirror]
    name=my CentOS 7.1 mirror
    baseurl=http://192.168.1.7/repository/centos/7.1
    gpgcheck=1
    gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7

    ```

1.  保存并关闭文件，然后测试您的新仓库是否可用（它应该出现在列表中）在您的客户端上：

    ```
    yum repolist | grep myCentosMirror

    ```

1.  现在，为了测试我们的新 YUM 仓库，我们可以尝试以下命令：

    ```
    yum --disablerepo="*" --enablerepo="myCentosMirror" list available

    ```

## 它是如何工作的...

在本节中，我们已经向你展示了安装和设置本地 YUM 仓库是多么容易。然而，我们只向你展示了如何创建所有 CentOS 7 Everything iso RPM 包的镜像站点，但你可以重复此过程来创建你想要与你的网络共享的任何类型的包的 YUM 仓库。

那么，我们从这次经历中学到了什么？

设置自己的 YUM 仓库只需安装`createrepo`包，并将你想要共享的所有 RPM 包复制到 Apache 文档根目录下的一个子文件夹中（在我们的例子中，我们需要挂载 CentOS 7 Everything iso 文件到文件系统，以便访问我们想要共享的包含的 RPM 包文件）。由于 Apache 的文档根目录受 SELinux 控制，之后我们需要将该目录中新 RPM 文件的安全上下文设置为`httpd_sys_content_t`类型标签；否则，通过 Web 服务器将无法访问。最后，我们需要在我们的新仓库文件夹上运行`createrepo`命令，这将创建我们新仓库所需的元数据，以便任何想要连接到仓库的 YUM 客户端稍后可以对其进行查询。

随后，为了测试我们的新仓库，我们在另一台想要使用这项新服务的 CentOS 7 系统上创建了一个新的仓库定义文件，该系统必须与我们的 YUM 仓库服务器处于同一网络中。在这个自定义的`.repo`配置文件中，我们设置了正确的仓库 URL 路径，启用了`gpg`检查，并采用了标准的 CentOS 7 `gpgkey`，以便我们的 YUM 客户端能够验证官方仓库包的有效性。最后，我们使用带有`--disablerepo="*"`和`--enablerepo="myCentosMirror"`参数的`yum`命令，这将确保只使用我们的新自定义仓库作为源。你可以将这两个参数与任何其他`yum`命令（如`install`、`search`、`info`、`list`等）结合使用。这只是为了测试；如果你想要将你的新仓库与现有的仓库结合使用，请使用 YUM 优先级（如本章另一节中所述）。

## 还有更多...

现在，在我们向我们的网络宣布新的集中式 YUM 仓库之前，我们应该首先更新自 CentOS Everything iso 发布以来已更改的所有 RPM 包。为此，请访问[`www.centos.org`](http://www.centos.org)并选择一个`rsync://`镜像链接，该链接地理位置上靠近您当前的位置。例如，如果您位于德国，一个选项可能是[rsync://ftp.hosteurope.de/centos/](http://rsync://ftp.hosteurope.de/centos/)（有关导航 CentOS 网站的更详细说明，请阅读第一章-version number (1.4.6)-release(1)-CPU architecture (x86_64)

```

接下来，我们使用 RPM 包管理器安装了下载的`pv`包，该管理器可以通过命令行上的`rpm`命令执行。我们使用了带有`-Uvh`命令参数的`rpm`命令，以及下载的包 rpm 文件的完整名称。

### 注意

如果使用 rpm 命令安装或升级 rpm 软件包，您应该始终使用`-Uvh`，但有一个例外；即内核包。`-U`会在更新时删除旧包，如果您安装新内核，这不是您想要的。请改用`-i`（用于安装），因为这将保留旧内核文件，以便在遇到问题时可以回退到较早的版本。

`-U`是安装或升级包的参数。如果系统上未安装该包，它将被安装；否则，如果 RPM 包版本比已安装的版本新，`rpm`将尝试升级它。`-v`参数打印更详细的输出，而`-h`显示一个漂亮的进度条。如果您在系统上未启用 EPEL 仓库的情况下安装`pv`包，将收到以下警告消息：

```
pv-1.6.0-1.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 3fc56f51: NOKEY

```

RPM 在安装前会自动检查包的签名有效性，以确保包的内容自签名后未被篡改。同时，它还会检查 RPM 包的可信度，因为应该由官方第三方权威供应商使用加密密钥进行签名。您可以忽略此消息，因为 EPEL 仓库的包来自安全源。要永久信任 EPEL 源，您可以使用以下命令在系统上安装其`gpg`公钥，从而消除所有未来的签名警告消息：

```
rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7

```

成功安装包后，我们现在拥有一个名为`pv`的漂亮命令行工具，它可以显示通过 Unix 管道的数据进度，如果您通过管道传输大量数据，这可能会很有用，因为您通常永远不会知道当前的进度状态。之后，我们查询了存储有关 CentOS 7 系统上所有已安装包信息的 RPM 数据库，使用带有`-q`标志的`rpm`命令。在处理 RPM 数据库时，我们必须使用真实的包名（`pv`）而不是安装包时使用的文件名（`pv-1.4.6-1.x86_64.rpm`）。在卸载已安装的包时也是如此；请指定包名，而不是版本号或完整文件名。

要获取有关已安装包`pv`的详细信息，我们使用了`-qi`（`i`表示信息），并使用`-ql`参数；我们显示了包中所有文件的完整文件名和路径。`-qd`显示了包中包含文档的所有文件。要了解更多的查询选项，请键入`man rpm`并查看`PACKAGE QUERY OPTIONS`部分。

总之，我们可以说，在系统管理员的生活中，有时需要安装一个不是通过官方仓库分发的软件（例如，非开源、前沿程序或测试版、有许可证不允许将其放入仓库的软件，如 Java，或来自独立开发者的软件），这时需要下载单独的 RPM 包并手动安装它们。在幕后，YUM 也依赖并使用 RPM 包管理器，因此你也可以使用 YUM 程序来安装 rpm 文件（`yum install <filename.rpm>`）。然而，当涉及到查询你下载的 rpm 文件或系统上已安装的包时，有时使用较旧的`rpm`命令而不必安装额外的基于 YUM 的软件（如`yum-utils`）会更好。

RPM 的最大弱点是它不支持仓库，并且缺少依赖管理系统。如果你仅使用 RPM 在 CentOS 系统上安装所有软件，你很容易遇到包依赖问题，因为你无法安装特定的包，因为它依赖于其他一些包。通常，当你尝试安装依赖的包时，你需要它们依赖的其他包，如此类推。这可能是一项非常繁琐的工作，应该始终通过使用 YUM 来避免。

## 还有更多...

`rpm`命令不仅可以用于查询 rpm 数据库中关于已安装包的信息，还可以用于查询你下载的 rpm 文件。例如，使用`-qlp`参数来显示本地`rpm`包文件中的所有文件：

```
rpm -qlp ~/pv-1.4.6-1.el7.x86_64.rpm

```

要从`rpm`文件中获取关于包的详细信息，使用`-qip`参数，如下所示：

```
rpm -qip ~/pv-1.4.6-1.el7.x86_64.rpm

```

如果你想安装一个本地下载的 RPM 包，并且该包有依赖关系，你可以使用`yum localinstall`命令。一旦提供了包的文件名，该命令将安装本地包，并尝试从远程源解决所有依赖关系，例如：

```
wget http://location/to/a/rpm/package_name.rpm
yum localinstall package_name.rpm

```