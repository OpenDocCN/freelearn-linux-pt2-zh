# 第九章：使用 Cron 自动化任务

在本章中，我们将涵盖：

+   创建和运行 crontab 文件

+   每隔一周运行一个命令

+   报告来自 crontab 文件的错误

# 介绍

cron 守护程序通常由操作系统自动启动，每分钟查看所有 crontab 文件。如果满足了标准，就会运行命令。在本章中，我们将展示如何使用 crontab 程序创建和维护您的 crontab 文件。

根据系统设置的方式，cron 作业是否允许（允许或不允许）取决于用户。控制这一点的文件位于`/etc`中，命名为`cron.allow`和`cron.deny`。这些将在以下部分中解释：

+   `cron.allow`：如果此文件存在，则用户必须在其中列出，才能使用 crontab

+   `cron.deny`：如果`cron.allow`不存在但`cron.deny`存在，则用户不得列在`cron.deny`文件中

如果两个文件都不存在，则只有 root 用户可以使用该命令。在大多数 Linux 系统中，只有`cron.deny`存在且为空。在运行以下命令之前，请在您的系统上检查这一点。

我们将使用一个用户帐户来尝试 crontab。`crontab`命令用于对 crontab 文件进行更改，不应直接编辑。要查看当前用户的 crontab，请运行`crontab -l`。

编辑当前用户的 crontab 文件的命令是`crontab -e`。默认情况下，这将在 vi 编辑器中打开 crontab 文件。但是，您可以更改`EDITOR`环境变量以使用任何文本编辑器。

以下是典型用户 crontab 文件的格式：

```
#
#  crontab for jklewis
# Field 1   2    3            4              5             6
#      Min Hour Day of month Month of year  Day of week
#     0-59 0-23 1-31         1-12           0-6   0=Sun /path/command
#
# Days of the week: 0=Sun 1=Mon 2=Tues 3=Wed 4=Thu 5=Fri 6=Sat

```

我总是将此模板添加到 crontab 文件的顶部，以便我可以轻松记住这些字段是什么。以下是一个示例 crontab 条目：

```
 10     0     *     *     0     /path/mycommand

```

这些值可以是整数、范围、元素或元素列表，或星号。星号表示匹配所有有效值。

+   `#`：这表示一行被注释。它必须是行上的第一件事（不要把它放在行的末尾）。

+   **字段 1**：这是分钟。它从 0 开始，意味着 00:00。

+   **字段 2**：这是小时。它从 0 开始，意味着凌晨 12:00。

+   **字段 3**：这是一个月中的日期。

+   **字段 4**：这是一年中的月份。也可以使用名称。

+   **字段 5**：这是一周中的日期。它从 0 开始，即星期日。也可以使用名称。

+   **字段 6**：这是要运行的路径/命令。

此示例条目将在每个星期日的凌晨 12:10 运行`mycommand`。

### 提示

只使用空格来分隔字段。不要使用制表符。

# 创建和运行 crontab 文件

在这里，我们将展示创建和运行用户 crontab 文件的示例。

## 准备工作

确保您的系统按照*介绍*中概述的方式设置。我们将使用两个终端会话，以便更容易地查看我们的结果。

## 如何做...

以下是创建和运行 crontab 文件的示例：

1.  在终端会话中运行`tty`并记住输出。这将在第 10 步中使用。

1.  打开或使用另一个终端，使用用户帐户。我将使用`jklewis`，就像在前几章中一样。

1.  让我们通过运行以下命令来查看 crontab 文件：

```
crontab -l

```

1.  它可能会说类似于`jklewis 没有 crontab`，这是可以接受的。

1.  现在让我们通过运行以下命令来创建一个：

```
crontab -e

```

1.  它可能会说类似于`jklewis 没有 crontab - 使用空的`，这是好的。

1.  Crontab 应该在 vi 中打开一个临时文件（除非您已更改了`EDITOR`变量，就像我在我的系统上做的那样）。在保存文件并结束会话之前，文件不会被使用。

1.  我建议将我上面创建的模板剪切并粘贴到您的 crontab 文件中。这将使您更容易记住这些字段。

1.  现在让我们添加一个条目，我们可以看到效果很快。将以下行插入文件（剪切和粘贴应该有效）：

```
TTY=/dev/pts/17
# crontab example 1
*   *   *    *    *     date > $TTY; echo "Yes it works" > $TTY

```

1.  将`TTY`行更改为您在步骤 1 中找到的内容。

1.  现在保存文件并退出会话。您应该会看到以下消息：

```
crontab: installing new crontab

```

1.  在接下来的一分钟内，您应该在另一个会话中看到以下输出：

```
Tue May  7 19:52:01 CDT 2013
Yes it works

```

1.  我们刚刚使用所有这些星号创建的条目意味着每分钟运行一次命令。再次编辑 crontab 并将行更改为以下代码：

```
*/5    *   *   *   *   date > $TTY; echo "Every 5 minutes"  > $TTY

```

1.  那种奇怪的语法是一种跳过或增加值的方法。现在尝试以下命令：

```
35     9     *     *     1-5   date > $TTY; echo "9:35 every week day"  > $TTY

```

1.  每周一至周五上午 9:35 运行。日期字段中的 1-5 是一个范围。

1.  名称也可以用于日期和月份字段，如以下命令：

```
17   13    8     May   Wed   date > $TTY; echo "May 8 at  13:17"  > $TTY

```

1.  以及以下命令：

```
0   0    *     *      Fri   date > $TTY; echo "Run every  Friday at 12:00 am"  > $TTY

```

对于名称，使用标准的三个字母缩写，大小写不重要。使用名称可能更容易，但是您不能在名称中使用范围或步长。

# 每隔一周运行一次命令

现在我们已经了解了 cron 的基础知识，您将如何设置条目以每隔一周运行一次命令？您可能会尝试类似以下代码的东西：

```
 *  *  *  *  0/2   /path/command

```

这意味着从星期日开始，然后每隔一周运行一次，对吗？不，这是错误的，但是您经常会在网站上看到这样的解决方案。Cron 实际上没有内置的方法来做到这一点，但有一个解决方法。

## 操作步骤如下...

以下是每隔一周运行命令的步骤：

1.  在您的主目录中创建以下脚本，并将其命名为`cron-weekly1`（随意剪切和粘贴）：

```
#!/bin/sh
# cron-weekly1
# Use this script to run a cron job every other week
FN=$HOME/cron-weekly.txt
if [ -f $FN  ] ; then
 rm $FN
 exit
fi
touch $FN
echo Run the command here

```

1.  通过运行以下命令使脚本可执行：

```
chmod 755 cron-weekly1

```

1.  通过运行以下命令在您的用户帐户下运行：

```
crontab -e

```

1.  添加以下行：

```
 0  0  *  *  0    $HOME/cron-weekly1

```

1.  结束您的编辑会话。这将安装修改后的 crontab 文件。

## 工作原理...

看看这个脚本。第一次运行时，`cron-weekly.txt`文件不存在，因此它会被创建，并且会执行命令（您想每隔一周运行的命令）。下周，当再次运行此脚本时，它会看到`cron-weekly.txt`文件存在，然后删除它，然后退出脚本（不运行命令）。这样每周交替进行，有效地每隔一周运行一次命令。很酷，对吧？

在前面的`cron-weekly1`脚本中，命令在第一次运行脚本时执行。您可以通过将运行命令的行移动到`if`语句内部来更改为从下一周开始运行命令。

尽管这样做可能非常诱人，但不要在行尾加上注释`#`。Cron 无法判断它是注释还是命令的一部分。如果 cron 报告了一些您不理解的错误，请检查放错位置的注释。是的，我承认我偶尔还会这样做。

如果您做了 cron 不喜欢的事情（例如在前面部分显示的`* * * * 0/2 command`行），当您关闭会话时它通常会报告错误。然后它会给您重新编辑文件的选项。务必这样做，要么解决问题，要么至少注释该行。您可以稍后返回并再次编辑它。

您可以通过运行`crontab -r`完全删除 crontab 文件。我建议在执行此操作之前先备份文件，以防万一。您应该能够通过您选择使用的任何文本编辑器将文件保存为新名称。

## 还有更多...

Crontab 文件可以使用环境变量。以下是一些常见的环境变量：

+   **SHELL**：这告诉操作系统使用特定的 shell，覆盖了`/etc/passwd`文件中的内容。例如，`SHELL=/bin/sh`。

+   **MAILTO**：这告诉 cron 将错误邮寄给此用户。语法是`MAILTO=<user>`，即`MAILTO=jklewis`。

+   **CRON_TZ**：用于设置特定的时区变量，即`CRON_TZ=Japan`。

Cron 有一些快捷方式可供使用。这些用于替代 5 个时间和日期字段，如下所示：

| 命令的快捷方式 | 命令 | 输出 |
| --- | --- | --- |
| **@reboot** |   | 重启后运行一次 |
| **@yearly or @annually** | **0 0 1 1 *** | 每年的第一天 |
| **@monthly** | **0 0 1 * *** | 每月第一天 |
| **@weekly** | **0 0 * * 0** | 每周日凌晨 12:00 运行 |
| **@daily** | **0 0 * * *** | 每天凌晨 12:00 运行 |
| **@hourly** | **0 * * * *** | 每小时整点运行 |

因此，在前面的示例中，我们可以放置`@weekly $HOME/cron-weekly1`。

不要将 cron 用于任何时间敏感的任务。通常在运行命令之前会有短暂的延迟，只有几秒钟。如果您需要更好的粒度，可以使用脚本和 sleep 例程。

您还可以为 root 设置一个 crontab。要了解更多信息，请使用`man -a crontab`。

# 报告 crontab 文件中的错误

您可能会想知道，如果 crontab 文件中有错误，计算机会如何报告它？它通过使用 sendmail 系统向 crontab 用户发送邮件来完成此操作。

## 如何做...

以下是 cron 报告错误的示例：

1.  使用用户帐户打开终端。

1.  通过运行以下命令编辑您的 crontab 文件：

```
crontab -e

```

1.  现在让我们故意引发一个错误。滚动到底部并添加以下行：

```
* * * *    date > /baddirectory/date.txt

```

1.  保存文件。等到 cron 在下一分钟运行，然后在用户终端中按*Enter*。

1.  您应该看到一条消息，说您有邮件。运行以下命令：

```
mail

```

1.  应该有一封邮件指示错误（在本例中，文件未找到）。您可以通过按下*D*然后按*Q*来删除邮件。

1.  最后，确保重新编辑您的 crontab 文件并删除我们刚刚添加的错误行。

## 还有更多...

您还可以监视`/var/log/cron`文件，以查看系统在一天中的运行情况。当首次创建 crontab 文件并尝试使其正确时，这非常有帮助。
