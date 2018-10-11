---
title: Xdebug之函数轨迹跟踪
date: 2016-10-22 20:30:00
tags:
- Xdebug
categories:
- PHP
---


Xdebug允许您记录所有函数调用，包括参数和返回值到不同格式的文件。

那些所谓的“函数跟踪”可以帮助当你是新的一个应用程序，或当你试图找出当你的应用程序运行时究竟是怎么回事。
函数跟踪还可以选择显示传递给函数和方法的变量的值，以及返回值。 在默认跟踪中，这两个元素不可用。

<!-- more -->

## 输出格式
有三种输出格式。 一个是作为一个人类可读的跟踪，另一个更适合计算机程序，因为它更容易解析，最后一个使用HTML格式化跟踪。 您可以使用xdebug.trace_format设置在两种不同的格式之间切换。 有一些设置控制哪些信息写入跟踪文件。 有一些设置包括变量（xdebug.collect_params）和包括返回值（xdebug.collect_return）例如。 下面的示例显示了不同设置对人类可读功能轨迹的影响。

脚本
```php
<?php
$str = "Xdebug";
function ret_ord( $c )
{
    return ord( $c );
}

foreach ( str_split( $str ) as $char )
{
    echo $char, ": ", ret_ord( $char ), "\n";
}
?>
```
结果：

以下是使用xdebug.collect_params设置的不同设置的结果。 因为这不是一个Web环境,值2没有任何意义，因为工具提示不工作在文本文件。

- default
```
TRACE START [2007-05-06 14:37:06]
    0.0003     114112   -> {main}() ../trace.php:0
    0.0004     114272     -> str_split() ../trace.php:8
    0.0153     117424     -> ret_ord() ../trace.php:10
    0.0165     117584       -> ord() ../trace.php:5
    0.0166     117584     -> ret_ord() ../trace.php:10
    0.0167     117584       -> ord() ../trace.php:5
    0.0168     117584     -> ret_ord() ../trace.php:10
    0.0168     117584       -> ord() ../trace.php:5
    0.0170     117584     -> ret_ord() ../trace.php:10
    0.0170     117584       -> ord() ../trace.php:5
    0.0172     117584     -> ret_ord() ../trace.php:10
    0.0172     117584       -> ord() ../trace.php:5
    0.0173     117584     -> ret_ord() ../trace.php:10
    0.0174     117584       -> ord() ../trace.php:5
    0.0177      41152
TRACE END   [2007-05-06 14:37:07]
```

- collect_params=1
```
TRACE START [2007-05-06 14:37:11]
    0.0003     114112   -> {main}() ../trace.php:0
    0.0004     114272     -> str_split(string(6)) ../trace.php:8
    0.0007     117424     -> ret_ord(string(1)) ../trace.php:10
    0.0007     117584       -> ord(string(1)) ../trace.php:5
    0.0009     117584     -> ret_ord(string(1)) ../trace.php:10
    0.0009     117584       -> ord(string(1)) ../trace.php:5
    0.0010     117584     -> ret_ord(string(1)) ../trace.php:10
    0.0011     117584       -> ord(string(1)) ../trace.php:5
    0.0012     117584     -> ret_ord(string(1)) ../trace.php:10
    0.0013     117584       -> ord(string(1)) ../trace.php:5
    0.0014     117584     -> ret_ord(string(1)) ../trace.php:10
    0.0014     117584       -> ord(string(1)) ../trace.php:5
    0.0016     117584     -> ret_ord(string(1)) ../trace.php:10
    0.0016     117584       -> ord(string(1)) ../trace.php:5
    0.0019      41152
TRACE END   [2007-05-06 14:37:11]
```

- collect_params=3
```
TRACE START [2007-05-06 14:37:13]
    0.0003     114112   -> {main}() ../trace.php:0
    0.0004     114272     -> str_split('Xdebug') ../trace.php:8
    0.0007     117424     -> ret_ord('X') ../trace.php:10
    0.0007     117584       -> ord('X') ../trace.php:5
    0.0009     117584     -> ret_ord('d') ../trace.php:10
    0.0009     117584       -> ord('d') ../trace.php:5
    0.0010     117584     -> ret_ord('e') ../trace.php:10
    0.0011     117584       -> ord('e') ../trace.php:5
    0.0012     117584     -> ret_ord('b') ../trace.php:10
    0.0013     117584       -> ord('b') ../trace.php:5
    0.0014     117584     -> ret_ord('u') ../trace.php:10
    0.0014     117584       -> ord('u') ../trace.php:5
    0.0016     117584     -> ret_ord('g') ../trace.php:10
    0.0016     117584       -> ord('g') ../trace.php:5
    0.0019      41152
TRACE END   [2007-05-06 14:37:13]
```

- collect_params=4
```
TRACE START [2007-05-06 14:37:16]
    0.0003     114112   -> {main}() ../trace.php:0
    0.0004     114272     -> str_split('Xdebug') ../trace.php:8
    0.0007     117424     -> ret_ord($c = 'X') ../trace.php:10
    0.0007     117584       -> ord('X') ../trace.php:5
    0.0009     117584     -> ret_ord($c = 'd') ../trace.php:10
    0.0009     117584       -> ord('d') ../trace.php:5
    0.0010     117584     -> ret_ord($c = 'e') ../trace.php:10
    0.0011     117584       -> ord('e') ../trace.php:5
    0.0012     117584     -> ret_ord($c = 'b') ../trace.php:10
    0.0013     117584       -> ord('b') ../trace.php:5
    0.0014     117584     -> ret_ord($c = 'u') ../trace.php:10
    0.0014     117584       -> ord('u') ../trace.php:5
    0.0016     117584     -> ret_ord($c = 'g') ../trace.php:10
    0.0016     117584       -> ord('g') ../trace.php:5
    0.0019      41152
TRACE END   [2007-05-06 14:37:16]
```

除了xdebug.collect_params设置，还有另一些影响跟踪文件输出的设置。
第一个选项卡“默认”显示与上面的默认值相同。
第二个选项卡“show_mem_delta = 1”还显示输出文件中两个不同行之间的内存使用差异。


在“collect return = 1”选项卡上，所有函数调用的返回值也是可见的。 这使用xdebug.collect返回设置打开。

名为“collect assignments = 1”的选项卡显示可变分配，可以使用xdebug.collect分配设置打开。

最后一个选项卡显示不同的输出格式，更容易解析，但更难阅读。
因此，如果有一个额外的工具来解释跟踪文件，xdebug.trace_format设置是非常有用的。

- default
```
TRACE START [2007-05-06 14:37:06]
    0.0003     114112   -> {main}() ../trace.php:0
    0.0004     114272     -> str_split() ../trace.php:8
    0.0153     117424     -> ret_ord() ../trace.php:10
    0.0165     117584       -> ord() ../trace.php:5
    0.0166     117584     -> ret_ord() ../trace.php:10
    0.0167     117584       -> ord() ../trace.php:5
    0.0168     117584     -> ret_ord() ../trace.php:10
    0.0168     117584       -> ord() ../trace.php:5
    0.0170     117584     -> ret_ord() ../trace.php:10
    0.0170     117584       -> ord() ../trace.php:5
    0.0172     117584     -> ret_ord() ../trace.php:10
    0.0172     117584       -> ord() ../trace.php:5
    0.0173     117584     -> ret_ord() ../trace.php:10
    0.0174     117584       -> ord() ../trace.php:5
    0.0177      41152
TRACE END   [2007-05-06 14:37:07]
```

- show_mem_delta=1
```
TRACE START [2007-05-06 14:37:26]
    0.0003     114112  +114112   -> {main}() ../trace.php:0
    0.0004     114272     +160     -> str_split('Xdebug') ../trace.php:8
    0.0007     117424    +3152     -> ret_ord($c = 'X') ../trace.php:10
    0.0007     117584     +160       -> ord('X') ../trace.php:5
    0.0009     117584       +0     -> ret_ord($c = 'd') ../trace.php:10
    0.0009     117584       +0       -> ord('d') ../trace.php:5
    0.0011     117584       +0     -> ret_ord($c = 'e') ../trace.php:10
    0.0011     117584       +0       -> ord('e') ../trace.php:5
    0.0013     117584       +0     -> ret_ord($c = 'b') ../trace.php:10
    0.0013     117584       +0       -> ord('b') ../trace.php:5
    0.0014     117584       +0     -> ret_ord($c = 'u') ../trace.php:10
    0.0015     117584       +0       -> ord('u') ../trace.php:5
    0.0016     117584       +0     -> ret_ord($c = 'g') ../trace.php:10
    0.0017     117584       +0       -> ord('g') ../trace.php:5
    0.0019      41152
TRACE END   [2007-05-06 14:37:26]
```

- collect_return=1
```
TRACE START [2007-05-06 14:37:35]
    0.0003     114112   -> {main}() ../trace.php:0
    0.0004     114272     -> str_split('Xdebug') ../trace.php:8
                          >=> array (0 => 'X', 1 => 'd', 2 => 'e', 3 => 'b', 4 => 'u', 5 => 'g')
    0.0007     117424     -> ret_ord($c = 'X') ../trace.php:10
    0.0007     117584       -> ord('X') ../trace.php:5
                            >=> 88
                          >=> 88
    0.0009     117584     -> ret_ord($c = 'd') ../trace.php:10
    0.0009     117584       -> ord('d') ../trace.php:5
                            >=> 100
                          >=> 100
    0.0011     117584     -> ret_ord($c = 'e') ../trace.php:10
    0.0011     117584       -> ord('e') ../trace.php:5
                            >=> 101
                          >=> 101
    0.0013     117584     -> ret_ord($c = 'b') ../trace.php:10
    0.0013     117584       -> ord('b') ../trace.php:5
                            >=> 98
                          >=> 98
    0.0015     117584     -> ret_ord($c = 'u') ../trace.php:10
    0.0016     117584       -> ord('u') ../trace.php:5
                            >=> 117
                          >=> 117
    0.0017     117584     -> ret_ord($c = 'g') ../trace.php:10
    0.0018     117584       -> ord('g') ../trace.php:5
                            >=> 103
                          >=> 103
                        >=> 1
    0.0021      41152
TRACE END   [2007-05-06 14:37:35]
```

- trace_format=1
```
Version: 2.0.0RC4-dev
TRACE START [2007-05-06 18:29:01]
1	0	0	0.010870	114112	{main}	1	../trace.php	0
2	1	0	0.032009	114272	str_split	0	../trace.php	8
2	1	1	0.032073	116632
2	2	0	0.033505	117424	ret_ord	1	../trace.php	10
3	3	0	0.033531	117584	ord	0	../trace.php	5
3	3	1	0.033551	117584
2	2	1	0.033567	117584
2	4	0	0.033718	117584	ret_ord	1	../trace.php	10
3	5	0	0.033740	117584	ord	0	../trace.php	5
3	5	1	0.033758	117584
2	4	1	0.033770	117584
2	6	0	0.033914	117584	ret_ord	1	../trace.php	10
3	7	0	0.033936	117584	ord	0	../trace.php	5
3	7	1	0.033953	117584
2	6	1	0.033965	117584
2	8	0	0.034108	117584	ret_ord	1	../trace.php	10
3	9	0	0.034130	117584	ord	0	../trace.php	5
3	9	1	0.034147	117584
2	8	1	0.034160	117584
2	10	0	0.034302	117584	ret_ord	1	../trace.php	10
3	11	0	0.034325	117584	ord	0	../trace.php	5
3	11	1	0.034342	117584
2	10	1	0.034354	117584
2	12	0	0.034497	117584	ret_ord	1	../trace.php	10
3	13	0	0.034519	117584	ord	0	../trace.php	5
3	13	1	0.034536	117584
2	12	1	0.034549	117584
1	0	1	0.034636	117584
TRACE END   [2007-05-06 18:29:01]
```


**VIM语法文件**

Xdebug附带了语法高亮显示跟踪文件的VIM语法文件：xt.vim。
为了使VIM能够识别这种新格式，您需要执行以下步骤：

1. 复制 xt.vim 文件到 ~/.vim/syntax
2. 修改, 或者创建, ~/.vim/filetype.vim 并添加下列内容
```
augroup filetypedetect
au BufNewFile,BufRead *.xt  setf xt
augroup END
```

使用这些设置，打开的跟踪文件看起来像：

![img](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-10@2x.png)


## 相关的设置

### **xdebug.auto_trace**

类型：布尔值，默认值：0

当此设置设置为on时，将在脚本运行之前启用函数调用的跟踪。这使得可以跟踪auto_prepend_file中的代码。

### **xdebug.collect_assignments**

类型：boolean，默认值：0，在Xdebug> 2.1中引入

此设置（默认为0）控制Xdebug是否应向函数轨迹添加变量分配。

### **xdebug.collect_includes**

类型：布尔值，默认值：1

此设置默认为1，控制Xdebug是否应将include（），include_once（），require（）或require_once（）中使用的文件名写入跟踪文件。

### **xdebug.collect_params**

类型：整数，默认值：0

此设置默认为0，控制在函数跟踪或堆栈跟踪中记录函数调用时，Xdebug是否应收集传递给函数的参数。

该设置默认为0，因为对于非常大的脚本，它可能使用大量的内存，因此使巨量脚本无法运行。您可以最安全地打开此设置，但是您可以预期在具有大量函数调用和/或巨大的数据结构作为参数的脚本中存在一些问题。 Xdebug 2不会有增加的内存使用这个问题，因为它永远不会将此信息存储在内存中。相反，它将只被写入磁盘。这意味着您需要查看磁盘使用情况。

此设置可以有四个不同的值。对于每个值，示出了不同量的信息。下面你将看到每个值提供什么信息。另请参见功能堆栈跟踪的几个截图的介绍。

显示的值|参数信息
-------|-----
0 |无。
1 |元素的类型和数量（f.e. string（6），array（8））。
2 |元素的类型和数量，带有完整信息的工具提示1。1在CLI版本的PHP中，它不会有工具提示，也不会在输出文件中。
3 |完全变量内容（具有由xdebug.var_display_max_children，xdebug.var_display_max_data和xdebug.var_display_max_depth设置的限制）。
4 |完全变量内容和变量名。
5 |PHP序列化变量内容，没有名称。 （Xdebug 2.3中的新功能）


### **xdebug.collect_return**

类型：布尔值，默认值：0

此设置默认为0，控制Xdebug是否应将函数调用的返回值写入跟踪文件。

对于计算机化的跟踪文件（xdebug.trace_format = 1），这只能从Xdebug 2.3起。

### **xdebug.show_mem_delta**

类型：整数，默认值：0

当这个设置设置为某些！= 0 Xdebug的人类可读的生成的跟踪文件将显示在函数调用之间的内存使用的差异。如果Xdebug配置为生成计算机可读的跟踪文件，则它们将始终显示此信息。

### **xdebug.trace_enable_trigger**

类型：boolean，默认值：0，在Xdebug> 2.2中引入

当此设置设置为1时，可以使用XDEBUG_TRACE GET / POST参数触发跟踪文件的生成，或者设置名为XDEBUG_TRACE的cookie。然后将跟踪数据写入定义的目录。为了防止Xdebug为每个请求生成跟踪文件，您需要将xdebug.auto_trace设置为0.对触发器本身的访问可以通过xdebug.trace_enable_trigger_value配置。

### **xdebug.trace_enable_trigger_value**

类型：字符串，默认值：“”，在Xdebug> 2.3中引入

此设置可用于限制谁可以使用xdebug.trace_enable_trigger中概述的XDEBUG_TRACE功能。当从空字符串的默认值更改时，cookie，GET或POST参数的值需要使用此设置匹配共享机密集，以便生成跟踪文件。

### **xdebug.trace_format**

类型：整数，默认值：0

跟踪文件的格式。

值|说明
--|----
0 |显示一个人工可读的缩进跟踪文件，具有：时间索引，内存使用，内存增量（如果设置xdebug.show_mem_delta启用），级别，函数名称，函数参数（如果设置xdebug.collect_params启用），文件名和行 数。
1 |写入具有两个不同记录的计算机可读格式。 有不同的记录用于输入堆栈帧，并留下堆栈帧。 下表列出了每种记录类型中的字段。 字段是制表符分隔的。
2 |写入以（简单）HTML格式化的跟踪。

### **xdebug.trace_options**

类型：整数，默认值：0

当设置为'1'时，跟踪文件将被附加到，而不是在后续请求中被覆盖。

### **xdebug.trace_output_dir**

类型：字符串，默认值：/ tmp

将写入跟踪文件的目录，确保PHP将运行的用户具有对该目录的写入权限。

### **xdebug.trace_output_name**

类型：字符串，默认值：trace。％c

此设置确定用于将跟踪转储到的文件的名称。 设置使用格式说明符指定格式，非常类似于sprintf（）和strftime（）。 有几个格式说明符可以用于格式化文件名。 “.xt”扩展名总是自动添加。

可能的格式说明符是：

指定符|含义|示例格式|示例文件名
-----|---|-------|---------
%c	|crc32 of the current working directory	|trace.%c	|trace.1258863198.xt
%p	|pid	|trace.%p	|trace.5174.xt
%r	|random number	|trace.%r	|trace.072db0.xt
%s	|script name 这一个不可用于跟踪文件名。|cachegrind.out.%s	|cachegrind.out.\_home_httpd_html_test_xdebug_test_php
%t	|timestamp (seconds)	|trace.%t	|trace.1179434742.xt
%u	|timestamp (microseconds)	|trace.%u	|trace.1179434749_642382.xt
%H	|$\_SERVER['HTTP_HOST']	|trace.%H	|trace.kossu.xt
%R	|$\_SERVER['REQUEST_URI']	|trace.%R	|trace.\_test_xdebug_test_php_var=1_var2=2.xt
%U	|$\_SERVER['UNIQUE_ID'] 版本2.2中的新功能。 这个由Apache mod_unique_id模块设置	|trace.%U	|trace.TRX4n38AAAEAAB9gBFkAAAAB.xt
%S	|session_id (from $\_COOKIE if set)	|trace.%S	|trace.c70c1ec2375af58f74b390bbdd2a679d.xt
%%	|literal %	|trace.%%	|trace.%%.xt


### **xdebug.var_display_max_children**

类型：整数，默认值：128

控制使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时显示的数组子元素和对象的属性的数量。

要禁用任何限制，请使用-1作为值。

此设置对通过远程调试功能发送到客户端的子项数没有任何影响。

### **xdebug.var_display_max_data**

类型：整数，默认值：512

控制使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时显示的最大字符串长度。

要禁用任何限制，请使用-1作为值。

此设置对通过远程调试功能发送到客户端的子项数没有任何影响。

### **xdebug.var_display_max_depth**

类型：整数，默认值：3

当使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时，控制数组元素和对象属性的嵌套级别数。

您可以选择的最大值为1023.您还可以使用-1作为值来选择此最大数。

此设置对通过远程调试功能发送到客户端的子项数没有任何影响。

## 相关函数

### **string xdebug_get_tracefile_name（）**

返回函数跟踪文件的名称

返回用于跟踪此脚本的输出的文件的名称。这在启用xdebug.auto_trace时非常有用。

### **string xdebug_start_trace（[string trace_file [，integer options]]）**

启动新的函数跟踪

启动跟踪从此点到trace_file参数中的文件的函数调用。如果未指定文件名，则跟踪文件将放置在由xdebug.trace_output_dir设置配置的目录中。

如果文件名称作为第一个参数，则名称相对于当前工作目录。此当前工作目录可能与您预期的不同，因此，如果指定文件名，请使用绝对路径。使用PHP函数getcwd（）来找出当前工作目录是什么。

跟踪文件的名称为“{trace_file} .xt”。如果启用了xdebug.auto_trace，则文件名的格式为“{filename} .xt”，其中“{filename}”部分取决于xdebug.trace_output_name设置。 options参数是一个位域;目前有三个选项：

- XDEBUG_TRACE_APPEND（1）
> 使追踪文件在追加模式而不是覆盖模式下打开

- XDEBUG_TRACE_COMPUTERIZED（2）
> 创建具有1“xdebug.trace_format”下描述的格式的跟踪文件。

- XDEBUG_TRACE_HTML（4）
> 将跟踪文件创建为HTML表

- XDEBUG_TRACE_NAKED_FILENAME（8）
> 通常，Xdebug总是将“.xt”添加到文件名的末尾，该文件名作为第一个参数传递给此函数。 如果设置了XDEBUG_TRACE_NAKED_FILENAME标志，则不会添加“.xt”。 （Xdebug 2.3中的新功能）。

与Xdebug 1不同，Xdebug 2不会在内存中存储函数调用，但始终只写入磁盘以减轻对已用内存的压力。 设置xdebug.collect_includes，xdebug.collect_params和xdebug.collect_return影响什么信息记录到跟踪文件，设置xdebug.trace_format影响跟踪文件的格式。
从此函数返回Xdebug跟踪的完整路径和文件名。 这将是您传入的文件名（可能添加了“.xt”），或者如果没有传入文件名，则为自动生成的文件名。


### **string xdebug_start_trace（）**

停止当前函数轨迹

停止跟踪函数调用并关闭跟踪文件。

该函数返回写入跟踪的文件的文件名。
