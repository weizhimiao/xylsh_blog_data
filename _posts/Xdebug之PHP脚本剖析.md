---
title: Xdebug之PHP脚本剖析
date: 2016-10-22 20:30:00
tags:
- Xdebug
categories:
- PHP
---


Xdebug内置分析器允许您在脚本中找到瓶颈，并使用外部工具（如KCacheGrind或WinCacheGrind）可视化这些瓶颈。

<!-- more -->

## 介绍

Xdebug的Profiler是一个强大的工具，使您能够分析您的PHP代码并确定瓶颈，或者通常查看代码的哪些部分很慢，并可以使用速度提升。
Xdebug 2中的分析器以高速缓存磨削兼容文件的形式输出分析信息。这允许您使用优秀的KCacheGrind工具（Linux，KDE）来分析概要分析数据。
- 如果你在Linux上，你可以安装KCacheGrind通过你最喜欢的包管理器。

- 如果你在Windows上，有预编译的QCacheGrind二进制文件可用。 （QCacheGrind是没有KDE绑定的KCacheGrind）。

- 如果你在Mac OSX上，还有如何构建QCacheGrind的说明。

Windows的用户可以选择使用WinCacheGrind。该功能不同于KCacheGrind，因此在此页面上记录使用KCacheGrind的部分不适用于此程序。 WinCacheGrind目前不支持Xdebug 2.3引入的cachegrind文件的文件和函数压缩。

还有一个替代的配置文件信息呈现工具xdebugtoolkit，一个称为Webgrind的基于Web的前端，以及一个名为XCallGraph的基于Java的工具。

如果您不能使用KDE（或不想使用KDE），kcachegrind软件包也会附带一个perl脚本“ct_annotate”，该脚本从profiler跟踪文件生成ASCII输出。

## 启动Profiler

通过在php.ini中将xdebug.profiler_enable设置为1来启用分析。
这将指示Xdebug开始将分析信息写入使用xdebug.profiler_output_dir指令配置的转储目录。
生成的文件的名称始终以“cachegrind.out”开头。并以PHP（或Apache）进程的PID（进程ID）或包含最初调试的脚本的目录的crc32哈希结束。
请确保您的xdebug.profiler_output_dir中有足够的空间，因为分析器生成的信息量对于复杂脚本非常大，例如对于像eZ Publish这样的复杂应用程序，最多可以有500MB的空间。

您还可以选择性地启用分析器，将xdebug.profiler_enable_trigger设置为1.
如果设置为1，那么可以使用名为XDEBUG_PROFILE的GET / POST或COOKIE变量启用分析器。
可以用于启用调试器（请参阅HTTP调试会话）的FireFox 2扩展也可以与此设置一起使用。为了使触发器正常工作，xdebug.profiler_enable需要设置为0。

## 分析Profiler

生成配置文件信息文件后，可以使用 [KCacheGrind](https://kcachegrind.github.io/) 打开它：

![img](https://xdebug.org/images/docs/kc-open.png)

一旦打开文件，您就可以在KCacheGrind的不同窗格中获得大量信息。在左侧，找到“Flat Profile”窗格，其中显示了脚本中按照此函数中的时间花费及其所有子项排序的所​​有函数。第二列“Self”显示此函数（没有其子项）的时间花费，第三列“Called”显示特定函数的调用频率，最后一列“Function”显示函数的名称。 Xdebug通过用“php ::”作为前缀来更改内部PHP函数名称，并且包含文件也以特殊方式处理。调用include（和include_one，require和require_once）后跟“::”和包含文件的文件名。在左边的截图中你可以看到“include :: / home / httpd / ez_34 / v ...”，内部PHP函数的例子是“php :: mysql_query”。前两列中的数字可以是脚本的完整运行时间的百分比（如在示例中）或绝对时间（1单位是1 / 1,000,000秒）。您可以使用右侧显示的按钮在两种模式之间切换。

![img](https://xdebug.org/images/docs/kc-profile.png)

右侧的窗格包含上窗格和下窗格。 上面的图显示了有关称为当前所选函数的函数的信息（“eztemplatedesignresource-> executecompiledtemplate”）。下面的窗格显示当前所选函数调用的函数的信息。

![img](https://xdebug.org/images/docs/kc-right-call.png)

上方窗格中的“成本”列显示从列表中的函数调用时当前所选函数的时间花费。 添加的“费用”列中的数字将始终为100％。 下方窗格中的“成本”列显示从列表中调用函数所花费的时间。 在添加此列表中的数字时，您很可能永远不会达到100％，因为所选的函数本身也需要时间来执行。

![img](https://xdebug.org/images/docs/kc-right-callers.png)

“所有呼叫者”和“所有呼叫”选项卡不仅显示从其调用该函数的直接调用，而且还显示所有直接调用的函数调用，还显示函数调用更多级别上下调用。 左侧屏幕截图中的上部窗格显示了所有调用当前所选函数的函数，直接和间接地使用堆栈上它们之间的其他函数。 “距离”列显示列出的和当前选择的函数调用之间有多少个函数调用（-1）。 如果两个函数之间有不同的距离，则显示为一个范围（例如“5-24”）。 括号中的数字是中值距离。 下面的窗格是类似的，除了它显示从当前选择的函数调用的函数的信息，再次是直接或间接。


## 相关的设置

### xdebug.profiler_append

类型：整数，默认值：0

当此设置设置为1时，当新请求映射到同一文件时（不在xdebug.profiler_output_name设置上），将不会覆盖分析器文件，而是使用新配置文件附加文件。

### xdebug.profiler_enable

类型：整数，默认值：0

启用Xdebug的概要分析器，它在概要文件输出目录中创建文件。这些文件可以由KCacheGrind读取以可视化您的数据。
无法使用ini_set（）在脚本中设置此设置。如果要选择性地启用分析器，请将xdebug.profiler_enable_trigger设置为1，而不使用此设置。

### xdebug.profiler_enable_trigger

类型：整数，默认值：0

当此设置设置为1时，可以使用XDEBUG_PROFILE GET / POST参数触发剖析器文件的生成，或者设置名为XDEBUG_PROFILE的cookie。
这将然后将分析器数据写入定义的目录。为了防止profiler为每个请求生成概要文件文件，您需要将xdebug.profiler_enable设置为0.对触发器本身的访问可以通过xdebug.profiler_enable_trigger_value配置。

### xdebug.profiler_enable_trigger_value

类型：字符串，默认值：“”，在Xdebug> 2.3中引入

此设置可用于限制谁可以使用xdebug.profiler_enable_trigger中概述的XDEBUG_PROFILE功能。当从空字符串的默认值更改时，cookie，GET或POST参数的值需要使用此设置匹配共享机密集，
以便分析器启动。

### xdebug.profiler_output_dir

类型：字符串，默认值：/ tmp

将写入分析器输出的目录，确保PHP将运行的用户具有对该目录的写入权限。无法使用ini_set（）在脚本中设置此设置。

### xdebug.profiler_output_name

类型：字符串，默认值：cachegrind.out。％p

此设置确定用于将跟踪转储到的文件的名称。设置使用格式说明符指定格式，非常类似于sprintf（）和strftime（）。有几个格式说明符可以用于格式化文件名。

有关受支持的说明符，请参见 [xdebug.trace_output_name](https://xdebug.org/docs/all_settings#trace_output_name) 文档。

## 相关函数

### string xdebug get profiler filename（）

返回配置文件信息文件名

返回用于将配置文件信息保存到的文件的名称。
