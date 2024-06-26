# 前言

CentOS 7 Linux 是最可靠的 Linux 操作系统之一，可用于计算机基础设施中的多种功能。对于任何系统管理员来说，它就像潘多拉魔盒，因为他可以塑造它来执行环境中的任何任务。

在任何基础设施中拥有 CentOS 7 服务器可以帮助部署许多有用的服务，以智能和自动化的方式维护、保护和管理基础设施。

# 本书涵盖的内容

第一章，“高级用户管理”，教您如何在 CentOS 7 上管理用户和组，以更好地了解其组织结构。

第二章，“安全”，展示了保护您的 CentOS 7 和一些宝贵服务免受可能禁用服务或暴露一些关键数据的攻击的最佳实践。

第三章，“用于不同目的的 Linux”，列举并介绍了一步一步的教程，说明如何设置计算机基础设施应具有的一系列非常有用的服务。

第四章，“使用 Postfix 的邮件服务器”，介绍了 Postfix 作为常见的开源邮件服务器，以便安装和配置以进行高级使用。

第五章，“监控和日志记录”，通过用户友好的监控和日志记录工具监控您的基础设施，并跟踪您的机器问题。

第六章，“虚拟化”，启动您的虚拟环境，并探索所有虚拟技术可以提供的可能性和好处。

第七章，“云计算”，通过使用 OpenStack 及其令人惊叹的组件构建自己的云环境，探索云计算。

第八章，“配置管理”，将您的基础设施提升到一个高级水平，使一切都在使用 Puppet 进行配置管理，因为它是这个领域中最著名的配置管理工具之一。

第九章，“一些额外的技巧和工具”，教您在管理 CentOS 7 服务器时可以使生活更轻松的一些小技巧和工具。

# 您需要什么

要正确地遵循本书，我们建议您拥有一个 CentOS 7 服务器，以满足大多数这些服务的以下特征：

+   CPU：4 核 3.00 GHz

+   内存：6 GB RAM

+   硬盘：150 GB

+   网络：1 Gbit/s

此外，您需要一些具有以下特征的机器来测试服务：

+   CPU：2 核 3.00 GHz

+   内存：2 GB RAM

+   硬盘：50 GB

+   网络：1Gbit/s

还需要良好的互联网连接和千兆网络交换机。

# 这本书适合谁

如果您是具有中级管理水平的 Linux 系统管理员，那么这是您掌握全新的 CentOS 发行版的机会。如果您希望拥有一个完全可持续的 Linux 服务器，具有所有新工具和调整，为用户和客户提供各种服务，那么这本书非常适合您。这是您轻松适应最新变化的通行证。

# 惯例

本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是这些样式的一些示例以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“我们可以通过使用`include`指令包含其他上下文。”

代码块设置如下：

```
<html>
    <title>
  Test page
    </title>
    <body>
  <h1>This is a test page</h1>
    </body>
</html>
```

任何命令行输入或输出都以以下形式编写：

```
testuser:x:1001:1001::/home/testuser:/bin/bash

```

**新术语**和**重要单词**以粗体显示。屏幕上看到的单词，例如菜单或对话框中的单词，会以这样的方式出现在文本中："然后我们定义要填写的字段**国家名称**，**州或省名称**，**地点名称**，**组织名称**，**组织单位名称**，**通用名称**和**电子邮件地址**"。

### 注意

警告或重要说明会出现在这样的框中。

### 提示

技巧和窍门会出现在这样。
