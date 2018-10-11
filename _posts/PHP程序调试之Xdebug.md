---
title: PHP程序调试之Xdebug
date: 2016-10.22 18:30:00
tags:
- Xdebug
categories:
- PHP
---

Xdebug是一个开放源代码的PHP程序调试器(即一个Debug工具)，可以用来跟踪，调试和分析PHP程序的运行状况。

<!-- more -->

## 安装
```
$ tar -zxvf xdebug-2.4.1.tgz
$ cd xdebug-2.4.1
$ /usr/local/bin/phpize
$ ./configure
$ make
$ make install
Installing shared extensions:     /usr/local/Cellar/php54/5.4.45_6/lib/php/extensions/no-debug-non-zts-20100525/
```
在`/usr/local/Cellar/php54/5.4.45_6/lib/php/extensions/no-debug-non-zts-20100525/` 查看`xdebug.so`是否已经生成


注：`/usr/local/Cellar/php54/5.4.45_6/lib/php/extensions/no-debug-non-zts-20100525/` 不同的PHP版本路径不同，也不一定要放在该路径，可以在zend_extension_ts中自行指定xdebug.so所在位置。

## 配置
修改配置文件 php.ini

```
[XDebug]  
zend_extension ="/usr/local/Cellar/php54/5.4.45_6/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so"  

xdebug.remote_handler=dbgp  
;开启远程调试  
xdebug.remote_enable = On  
;远程主机  
xdebug.remote_host=localhost  
;主机端口  
xdebug.remote_port=9000  
;开启自动跟踪  
xdebug.auto_trace = On  
;开启异常跟踪  
xdebug.show_exception_trace = On  
;开启远程调试自动启动  
xdebug.remote_autostart = On  
;收集变量   
xdebug.collect_vars = On  
;收集返回值   
xdebug.collect_return = On  
;收集参数   
xdebug.collect_params = On  
;设定函数调用监测信息的输出文件的路径。
;xdebug.trace_output_dir="/home/xdebug_log"  
;显示局部变量  
xdebug.show_local_vars = On  
xdebug.profiler_enable = On  
;设定效能监测信息输出文件的路径。
;xdebug.profiler_output_dir ="/home/xdebug_log"  
xdebug.trace_enable_trigger =On  
```

注：xdebug是一个zend扩展，所以要用zend_extension来加载，不能使用extensions来加载

运行`php -m`查看xdebug模块是否已经加载

### 其他配置选项说明：


#### 日志

xdebug.trace_output_dir
> 日志追踪输出目录
>
> 类型：字符串，默认值：/tmp
>
> 确保php运行用户对该目录有写权限

xdebug.trace_output_name
> 日志文件名，xdebug提供了一系列的标识符，生成相应格式的文件名

> 类型: string, 默认值: trace.%c

> 此设置确定用于将跟踪转储到的文件的名称。 设置使用格式说明符指定格式，非常类似于sprintf（）和strftime（）。 有几个格式说明符可以用于格式化文件名。 “.xt”扩展名总是自动添加。

可能的格式说明符是：

说明|	含义|	示例|	对应文件名
---|----|-----|---------
%c|	crc32 of the current working directory	|trace.%c	|trace.1258863198.xt
%p|	pid	|trace.%p	|trace.5174.xt
%r|	random number	|trace.%r	|trace.072db0.xt
%s|	script name |cachegrind.out.%s	|cachegrind.out.\_home_httpd_html_test_xdebug_test_php
%t|	timestamp (seconds)	|trace.%t	|trace.1179434742.xt
%u|	timestamp (microseconds)	|trace.%u	|trace.1179434749_642382.xt
%H|	$\_SERVER['HTTP_HOST']	|trace.%H	|trace.kossu.xt
%R|	$\_SERVER['REQUEST_URI']	|trace.%R	|trace.\_test_xdebug_test_php_var=1_var2=2.xt
%U|	$\_SERVER['UNIQUE_ID'] 3	|trace.%U	|trace.TRX4n38AAAEAAB9gBFkAAAAB.xt
%S|	session_id (from $\_COOKIE if set)	|trace.%S |trace.c70c1ec2375af58f74b390bbdd2a679d.xt
%%|	literal %	|trace.%%	|trace.%%.xt


xdebug.trace_options
> 记录添加到文件中方式：

> 类型：整数，默认值：0

>当设置为'1'时，跟踪文件将被附加到，而不是在后续请求中被覆盖。


#### 显示数据

xdebug.collect_params

> 类型：整数，默认值：0

> 此设置默认为0，控制function的参数显示选项
>
- 0	不显示.
- 1	参数类型，值  (例如：array(9))
- 2	同上1，只是在CLI模式下略微有区别
- 3	所有变量内容
- 4	所有变量内容和变量名(例如：array(0 => 9))
- 5	PHP序列化变量内容，没有名称。 （Xdebug 2.3中的新功能）


xdebug.collect_return

> 类型：布尔值，默认值：0

> 此设置默认为0，控制Xdebug是否应将函数调用的返回值写入跟踪文件。

> 对于计算机化的跟踪文件（xdebug.trace_format = 1），这只能从Xdebug 2.3起。


xdebug.collect_vars
> 类型：布尔值，默认值：0

> 此设置告诉Xdebug收集有关在某个范围中使用哪些变量的信息。 这个分析可能很慢，因为Xdebug必须逆向工程PHP的操作码数组。 此设置不会记录不同变量具有的值，因为使用xdebug.collect_params。 仅当您希望使用xdebug_get_declared_vars（）时，才需要启用此设置。

> 显示当前作用域使用了哪些变量，显示变量名，该选项不会记录变量的值，如果需要，使用xdebug.collect_params

xdebug.collect_assignments

> 类型：boolean，默认值：0，在Xdebug> 2.1中引入

> 此设置（默认为0）控制Xdebug是否应向函数轨迹添加变量分配。

> 1 = 添加一行显示变量赋值（若为1，形如$a = 1;这类Assignment Expression会在trace文件里显示）

#### 格式

xdebug.trace_format

> 类型：整数，默认值：0

> - 0 = 人可读. 从左至右每列分别表示：时间点, 内存, 内存差 (需要设置xdebug.show_mem_delta=1), 等级, 函数名,函数参数 (需要设置，xdebug.collect_params=1，只要是非零), 当前代码行所在文件名 , 行号;
> - 1 = 机器可读[1]. 需要借助第三方app，例如：xdebug trace file parser 或者 xdebug trace viewer; 2 = html格式 即table，用browser打开，显示table

xdebug.show_mem_delta

> 类型：整数，默认值：0

> 当这个设置设置为某些！= 0 Xdebug的人类可读的生成的跟踪文件将显示在函数调用之间的内存使用的差异。 如果Xdebug配置为生成计算机可读的跟踪文件，则它们将始终显示此信息。

> - 1 = 显示每次函数调用内存消耗（内存差）

#### 行为

xdebug.auto_trace
> 类型：布尔值，默认值：0

> 当此设置设置为on时，将在脚本运行之前启用函数调用的跟踪。 这使得可以跟踪auto_prepend_file中的代码。

> 1 = 打开自动追踪. （追踪方式有2种，一种是自动追踪，所有php脚本运行时，都会产生trace文件；另一种是触发方式追踪，如下）



xdebug.trace_enable_trigger[2]
> 类型：boolean，默认值：0，在Xdebug> 2.2中引入

> 当此设置设置为1时，可以使用XDEBUG_TRACE GET / POST参数触发跟踪，或者设置名为XDEBUG_TRACE的cookie。 然后将跟踪数据写入定义的目录。 为了防止Xdebug为每个请求生成跟踪文件，您需要将xdebug.auto_trace设置为0.对触发器本身的访问可以通过xdebug.trace_enable_trigger_value配置。

> 1 = 使用 XDEBUG_TRACE GET/POST 触发追踪, 或者通过设置cookie XDEBUG_TRACE. 为了避免每次请求时，都会生成相应trace追踪文件，你需要把auto_trace设置为0

> 注：该特性只在2.2+版本才能设置
> [xdebug-general] Re: Is trace_enable_trigger defunct?

#### 限制


xdebug.var_display_max_depth
> 类型：整数，默认值：3

> 当使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时，控制数组元素和对象属性的嵌套级别数。

> 您可以选择的最大值为1023.您还可以使用-1作为值来选择此最大数。

> 此设置对通过远程调试功能发送到客户端的子项数没有任何影响。

> 数组和对象元素显示深度：主要用在数组嵌套，对象属性嵌套时，显示几级的元素内容. Default 3.


xdebug.var_display_max_data
> 变量值为字符串时显示多长. Default 512.

> 类型：整数，默认值：512
> 控制使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时显示的最大字符串长度。

> 要禁用任何限制，请使用-1作为值。

> 此设置对通过远程调试功能发送到客户端的子项数没有任何影响。


xdebug.var_display_max_children
> 数组和对象元素显示的个数. Default 128

> 类型：整数，默认值：128
> 控制使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时显示的数组子元素和对象的属性的数量。

> 要禁用任何限制，请使用-1作为值。

> 此设置对通过远程调试功能发送到客户端的子项数没有任何影响。


## 使用

一些常用到的xdebug函数

Function|	Description
--------|----------
void xdebug_enable()	|手动打开，相当于xdebug.default_enable=on
void var_dump()	|覆写php提供的var_dump，出错时，显示函数堆栈信息，（前提：php.ini里html_errors为1），使用xdebug.overload_var_dump 设置是否覆写
void xdebug_start_trace( string trace_file_path [, integer options] )	|手动控制需要追踪的代码段 trace_file_path ：文件路径（相对或绝对，若为空）.如果为空，或者不传参， 使用xdebug.trace_output_dir设置的目录options ：XDEBUG_TRACE_APPEND: 1 = 追加文件内容末尾, 0 = 覆写该文件 XDEBUG_TRACE_COMPUTERIZED:2 =同 xdebug.trace_format=1 .XDEBUG_TRACE_HTML: 4 = 输出HTML表格，浏览器打开为一table
void xdebug_stop_trace()	|停止追踪，代码追踪在该行停止
string xdebug_get_tracefile_name()	|获得输出文件名，与 xdebug.auto_trace配合使用.
void xdebug_var_dump([mixed var[,...]])	 |输出变量详细信息，相当于php里的var_dump，具体显示请看这里xdebug.show_local_vars	 默认为0，不显示；非零时，在php执行出错时，显示出错代码所在作用域所有本地变量（注：这会产生大量信息，因此默认是closed），具体显示差别如下图[3]
array xdebug_get_declared_vars()	|显示当前作用域中已声明的变量
array xdebug_get_code_coverage()	|显示某一段代码内，代码执行到哪些行[4]
