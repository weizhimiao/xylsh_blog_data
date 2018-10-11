---
title: Xdebug之变量显示功能
date: 2016-10-22 20:30:00
tags:
- Xdebug
categories:
- PHP
---


当Xdebug被激活时，只要PHP决定显示通知，警告，错误等，它将显示一个堆栈跟踪。堆栈跟踪显示的信息以及它们的呈现方式可以配置为满足您的需要。

Xdebug在错误情况下显示的堆栈跟踪（如果display_errors在php.ini中设置为On）在它们显示的信息量上相当保守。 这是因为大量的信息可以减慢脚本的执行和在浏览器中呈现堆栈跟踪本身。 但是，可以使堆栈跟踪显示具有不同设置的更详细信息。

<!-- more -->

## 堆栈跟踪中的变量

默认情况下，Xdebug现在会在它生成的堆栈跟踪中显示变量信息。 在收集或显示时，可变信息可能需要相当多的资源。 然而，在许多情况下，显示变量信息是有用的，这就是为什么Xdebug具有设置xdebug.collect_params。 下面的脚本结合了该设置的不同值的输出结果，如下例所示。

示例：
```php
<?php
function foo( $a ) {
    for ($i = 1; $i < $a['foo']; $i++) {
        if ($i == 500000) xdebug_break();
    }
}

set_time_limit(1);
$c = new stdClass;
$c->bar = 100;
$a = array(
    42 => false, 'foo' => 912124,
    $c, new stdClass, fopen( '/etc/passwd', 'r' )
);
foo( $a );
?>
```

结果:
xdebug.collect_params设置的不同值提供不同的输出，您可以在下面看到：

- default

![default](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-0@2x.png)

- 1
```
ini_set('xdebug.collect_params', '1');
```

![default](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-1@2x.png)

- 2
```
ini_set('xdebug.collect_params', '2');
```

![default](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-2@2x.png)

- 3
```
ini_set('xdebug.collect_params', '3');
```

![default](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-3@2x.png)

- 4
```
ini_set('xdebug.collect_params', '4');
```

![default](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-4@2x.png)

## 附加信息
除了显示传递给每个函数的变量的值之外，Xdebug还可以通过使用xdebug.dump_globals和xdebug.dump.\* 设置来选择性地显示关于所选超级元素的信息。 设置xdebug.dump_once和xdebug.dump_undefined会稍微修改可用超级元素显示的时间和信息。 使用xdebug.show_local_vars设置，您可以指示Xdebug为用户定义的函数显示最顶层栈级中可用的所有变量。 下面的示例显示了这一点（脚本从上面的示例中使用）。

- default

![default](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-5@2x.png)

- dump_superglobals=1
```
ini_set('xdebug.collect_vars', 'on');
ini_set('xdebug.collect_params', '4');
ini_set('xdebug.dump_globals', 'on');
ini_set('xdebug.dump.SERVER', 'REQUEST_URI');
```

![default](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-6@2x.png)


- show_local_vars=1
```
ini_set('xdebug.collect_vars', 'on');
ini_set('xdebug.collect_params', '4');
ini_set('xdebug.dump_globals', 'on');
ini_set('xdebug.dump.SERVER', 'REQUEST_URI');
ini_set('xdebug.show_local_vars', 'on');
```

![default](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-8@2x.png)


## 相关设置

### **xdebug.cli_color**

类型：整数，默认值：0，在Xdebug> 2.2中引入

如果此设置为1，则Xdebug将在CLI模式下和输出为tty时对var_dumps和堆栈跟踪输出进行颜色。在Windows上，需要安装ANSICON工具。

如果设置为2，那么无论Xdebug是否连接到tty或是否安装了ANSICON，Xdebug都将始终对var_dumps和堆栈跟踪进行着色。在这种情况下，您可能会看到转义码。

有关更多信息，请参阅此[文章](http://drck.me/clicolor-9cr)。

### **xdebug.collect_includes**

类型：布尔值，默认值：1

此设置默认为1，控制Xdebug是否应将include（），include_once（），require（）或require_once（）中使用的文件名写入跟踪文件。


### **xdebug.collect_params**

类型：整数，默认值：0

此设置默认为0，控制在函数跟踪或堆栈跟踪中记录函数调用时，Xdebug是否应收集传递给函数的参数。

该设置默认为0，因为对于非常大的脚本，它可能使用大量的内存，因此使巨量脚本无法运行。您可以最安全地打开此设置，但是您可以预期在具有大量函数调用和/或巨大的数据结构作为参数的脚本中存在一些问题。

Xdebug 2不会有增加的内存使用这个问题，因为它永远不会将此信息存储在内存中。相反，它将只被写入磁盘。这意味着您需要查看磁盘使用情况。

此设置可以有四个不同的值。对于每个值，示出了不同量的信息。下面你将看到每个值提供什么信息。另请参见功能堆栈跟踪的几个截图的介绍。


值|显示参数信息
--|-------
0 |无。
1 |元素的类型和数量（f.e. string（6），array（8））。
2 |元素的类型和数量，带有完整信息的工具提示1。在CLI版本的PHP它不会有工具提示，而不是在输出文件。
3 |完全变量内容（具有由xdebug.var_display_max_children，xdebug.var_display_max_data和xdebug.var_display_max_depth设置的限制）。
4 |完全变量内容和变量名称。
5 | PHP序列化变量内容，没有名称。 （Xdebug 2.3中的新功能）

### **xdebug.collect_vars**

类型：布尔值，默认值：0

此设置告诉Xdebug收集有关在某个范围中使用哪些变量的信息。 这个分析可能很慢，因为Xdebug必须逆向工程PHP的操作码数组。 此设置不会记录不同变量具有的值，因为使用xdebug.collect_params。 仅当您希望使用xdebug_get_declared_vars（）时，才需要启用此设置。

### **xdebug.dump.\***

类型：字符串，默认值：空

\*可以是COOKIE，FILES，GET，POST，REQUEST，SERVER，SESSION中的任何一个。 这七个设置控制当发生错误情况时来自超级溶剂的数据。

每个php.ini设置可以包括一个逗号分隔的变量从这个超级全局转储，或*所有的。 请确保您在此设置中不添加空格。

为了在发生错误时转储REMOTE_ADDR和REQUEST_METHOD，以及所有GET参数，请添加以下设置：
```
xdebug.dump.SERVER = REMOTE_ADDR,REQUEST_METHOD
xdebug.dump.GET = *
```

### **xdebug.dump_globals**
类型：布尔值，默认值：1

控制是否应显示由xdebug.dump.\*设置定义的超级元素的值。

### **xdebug.dump_once**
类型：布尔值，默认值：1

控制是否应在所有错误情况（设置为0）或仅在第一个（设置为1）上转储超级元素的值。

### **xdebug.dump_undefined**
类型：布尔值，默认值：0
如果要从超级元组转储未定义的值，您应该将此设置设置为1，否则将其设置为0。

### **xdebug.file_link_format**
类型：字符串，默认值：，在Xdebug> 2.1中引入

此设置确定在使用文件名的堆栈跟踪显示中进行的链接的格式。 这允许IDE设置一个链接协议，通过单击Xdebug在堆栈跟踪中显示的文件名，可以直接转到行和文件。 示例格式可能如下所示：
```
myide://%f@%l
```
可能的格式说明符是：

说明符|含义
-----|---
％f  |文件名
％l  |行号

对于各种IDE / OSses，有一些关于如何使这项工作的指示：

Linux上的Firefox

- 打开about：config
- 添加一个新的布尔设置“network.protocol-handler.expose.xdebug”并将其设置为“false”
- 将以下内容添加到shell脚本〜/ bin / ff-xdebug.sh中：
```sh
#! /bin/sh
f=`echo $1 | cut -d @ -f 1 | sed 's/xdebug:\/\///'`
l=`echo $1 | cut -d @ -f 2`
```
添加（取决于你有komodo，vim vs netbeans）：
```sh
komodo $f -l $l
gvim --remote-tab +$l $f
netbeans "$f:$l"
```

- 使用chmod + x〜/ bin / ff-xdebug.sh来使脚本可执行
- 将xdebug.file_link_format设置为xdebug：//％f @％l

Windows和netbeans

- 创建文件netbeans.bat并将其保存在您的路径（C：\ Windows）：
```
@echo off
setlocal enableextensions enabledelayedexpansion
set NETBEANS=%1
set FILE=%~2
%NETBEANS% --nosplash --console suppress --open "%FILE:~19%"
nircmd win activate process netbeans.exe
```

注意：如果没有nircmd，请删除最后一行。

- 将以下代码保存在netbeans protocol.reg中：

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\netbeans]
"URL Protocol"=""
@="URL:Netbeans Protocol"

[HKEY_CLASSES_ROOT\netbeans\DefaultIcon]
@="\"C:\\Program Files\\NetBeans 7.1.1\\bin\\netbeans.exe,1\""

[HKEY_CLASSES_ROOT\netbeans\shell]

[HKEY_CLASSES_ROOT\netbeans\shell\open]

[HKEY_CLASSES_ROOT\netbeans\shell\open\command]
@="\"C:\\Windows\\netbeans.bat\" \"C:\\Program Files\\NetBeans 7.1.1\\bin\\netbeans.exe\" \"%1\""
```

注意：确保更改Netbeans（两次）的路径以及netbeans.bat批处理文件（如果将其保存在C：\\Windows 以外的其他位置）。


Double click on the netbeans_protocol.reg file to import it into the registry.
Set the xdebug.file_link_format setting to

- 双击netbeans protocol.reg文件将其导入到注册表中。

- 将xdebug.file_link_format设置设置为`xdebug.file_link_format = "netbeans://open/?f=%f:%l"`


### **xdebug.manual_url**

类型：字符串，默认值：http：//www.php.net，在Xdebug <2.2.1中引入

这是从函数跟踪和错误消息到函数的手册页的链接的基本URL。建议将此设置设置为使用最近的镜像。

### **xdebug.show_exception_trace**

类型：整数，默认值：0

当此设置设置为1时，每当引发异常时，即使此异常实际被捕获，Xdebug也将显示堆栈跟踪。

### **xdebug.show_local_vars**

类型：整数，默认值：0

当这个设置设置为某事！= 0 Xdebug的错误情况下生成的堆栈转储也将显示最顶层范围中的所有变量。请注意，这可能会生成大量的信息，因此默认情况下关闭。

### **xdebug.show_mem_delta**

类型：整数，默认值：0

当这个设置设置为某些！= 0 Xdebug的人类可读的生成的跟踪文件将显示在函数调用之间的内存使用的差异。如果Xdebug配置为生成计算机可读的跟踪文件，则它们将始终显示此信息。

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

### **array xdebug_get_declared_vars（）**

返回声明的变量

返回一个数组，其中每个元素是在当前范围中定义的变量名称。 需要启用xdebug.collect_vars设置。

Example:
```php
<?php
    class strings {
        static function fix_strings($a, $b) {
            foreach ($b as $item) {
            }
            var_dump(xdebug_get_declared_vars());
        }
    }
    strings::fix_strings(array(1,2,3), array(4,5,6));
?>
```
Returns:
```
array
  0 => string 'a' (length=1)
  1 => string 'b' (length=1)
  2 => string 'item' (length=4)
```
在5.1之前的PHP版本中，变量名“a”不在返回的数组中，因为它在调用函数xdebug_get_declared_vars（）的作用域中不使用。

### **array xdebug_get_function_stack（）**

返回有关堆栈的信息

返回类似于此点的堆栈跟踪的数组。 示例脚本：

Example:
```php
<?php
    class strings {
        function fix_string($a)
        {
            var_dump(xdebug_get_function_stack());
        }

        function fix_strings($b) {
            foreach ($b as $item) {
                $this->fix_string($item);
            }
        }
    }

    $s = new strings();
    $ret = $s->fix_strings(array('Derick'));
?>
```
Returns:
```
array
  0 =>
    array
      'function' => string '{main}' (length=6)
      'file' => string '/var/www/xdebug_get_function_stack.php' (length=63)
      'line' => int 0
      'params' =>
        array
          empty
  1 =>
    array
      'function' => string 'fix_strings' (length=11)
      'class' => string 'strings' (length=7)
      'file' => string '/var/www/xdebug_get_function_stack.php' (length=63)
      'line' => int 18
      'params' =>
        array
          'b' => string 'array (0 => 'Derick')' (length=21)
  2 =>
    array
      'function' => string 'fix_string' (length=10)
      'class' => string 'strings' (length=7)
      'file' => string '/var/www/xdebug_get_function_stack.php' (length=63)
      'line' => int 12
      'params' =>
        array
          'a' => string ''Derick'' (length=8)
```

### **array xdebug_get_monitored_functions（）**

返回有关受监视函数的信息

在2.4版本中引入

返回一个结构，其中包含有关在脚本中执行受监视函数的位置的信息。 以下示例显示如何使用此和返回的信息：

Example:
```php
<?php
/* Start the function monitor for strrev and array_push: */
xdebug_start_function_monitor( [ 'strrev', 'array_push' ] );

/* Run some code: */
echo strrev("yes!"), "\n";

echo strrev("yes!"), "\n";

var_dump(xdebug_get_monitored_functions());
xdebug_stop_function_monitor();
?>  
```
Returns:
```
/tmp/monitor-example.php:10:
array(2) {
  [0] =>
  array(3) {
    'function' =>
    string(6) "strrev"
    'filename' =>
    string(24) "/tmp/monitor-example.php"
    'lineno' =>
    int(6)
  }
  [1] =>
  array(3) {
    'function' =>
    string(6) "strrev"
    'filename' =>
    string(24) "/tmp/monitor-example.php"
    'lineno' =>
    int(8)
  }
}
```

### **integer xdebug_get_stack_depth（）**

返回当前堆栈深度级别

返回堆栈深度级别。 脚本的主体是级别0，每个包含和/或函数调用将向堆栈深度级别添加一个。

### **none xdebug_print_function_stack( [ string message [, int options ] ] )**

显示当前函数栈

以类似于Xdebug在错误情况下显示的方式显示当前函数堆栈。

“message”参数允许您使用自己的头部替换消息。 （在Xdebug 2.1中引入）。

Example:
```php
<?php
function foo( $far, $out )
{
    xdebug_print_function_stack( 'Your own message' );
}
foo( 42, 3141592654 );
?>
```
Returns:

![img](http://n.sinaimg.cn/games/3ece443e/20161022/QQ20161022-9@2x.png)

位掩码“options”允许您配置一些额外的选项。 目前支持以下选项：

- XDEBUG_STACK_NO_DESC
> 如果设置了此选项，则打印的堆栈跟踪将不具有标题。 如果要从自己的错误处理程序打印堆栈跟踪，否则打印位置是从中调用xdebug_print_function_stack（）时，这是有用的。 （在Xdebug 2.3中引入）。

### **void xdebug_start_function_monitor（array $ list_of_functions_to_monitor）**

开始功能监控

在2.4版本中引入

此函数开始监视列表中给出的函数作为此函数的参数。 函数监视允许您找到代码中提供的作为参数的函数在哪里调用。 这可以用于跟踪使用旧的或不鼓励的函数的位置。

Example:
```php
<?php
xdebug_start_function_monitor( [ 'strrev', 'array_push' ] );
?>
```

您还可以向定义要监视的函数的数组添加类方法和静态方法。 例如，要捕获对DramModel :: canSee的静态调用和对Whisky-> drink的动态调用，您将启动监视器：

Example:
```php
<?php
xdebug_start_function_monitor( [ 'DramModel::canSee', 'Whisky->drink'] );
?>
```
定义的函数区分大小写，并且不会捕获对静态方法的动态调用。


### **void xdebug_stop_function_monitor（）**

停止监视功能

在2.4版本中引入

此功能停止功能监视器。 为了获得受监视函数的列表，您需要使用xdebug_get_monitored_functions（）函数。
