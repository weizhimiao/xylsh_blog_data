---
title: PHP中is_func函数汇总
date: 2016-10-18 18:30:00
categories:
- PHP
---


![PHP中is_func函数汇总](http://n.sinaimg.cn/games/3ece443e/20161018/is_func.png)

<!-- more -->

## is_float
> (PHP 4, PHP 5, PHP 7)
>
> is_float — 检测变量是否是浮点型

描述
```
bool is_float ( mixed $var )
```
如果 var 是 float 则返回 TRUE，否则返回 FALSE。

## is_soap_fault
> is_soap_fault — 检查SOAP调用是否失败

说明
```
bool is_soap_fault ( mixed $object )
```
此函数用于检查SOAP调用是否失败，但不使用异常。 要使用它，请创建一个SoapClient对象，将异常选项设置为零或FALSE。 在这种情况下，SOAP方法将返回封装故障详细信息（faultcode，faultstring，faultactor和faultdetails）的特殊SoapFault对象。

如果未设置异常，那么SOAP调用将在错误时抛出异常。 is_soap_fault（）检查给定的参数是否是SoapFault对象。

## is_bool
> (PHP 4, PHP 5, PHP 7)
>
> is_bool — 检测变量是否是布尔型

描述
```
bool is_bool ( mixed $var )
```
如果 var 是 boolean 则返回 TRUE。


## is_integer
> (PHP 4, PHP 5, PHP 7)
>
> is_integer — is_int() 的别名

描述

此函数是 is_int() 的别名函数。

## is_dir
> (PHP 4, PHP 5, PHP 7)
>
> is_dir — 判断给定文件名是否是一个目录

说明
```
bool is_dir ( string $filename )
```
判断给定文件名是否是一个目录。

## is_writeable
> (PHP 4, PHP 5, PHP 7)
>
> is_writeable — is_writable() 的别名

说明

此函数是该函数的别名：is_writable()。

## is_real
> (PHP 4, PHP 5, PHP 7)
>
> is_real — is_float() 的别名


描述

此函数是 is_float() 的别名函数。


## is_file
> (PHP 4, PHP 5, PHP 7)
>
> is_file — 判断给定文件名是否为一个正常的文件

说明
```
bool is_file ( string $filename )
```
判断给定文件名是否为一个正常的文件。


## is_subclass_of
> (PHP 4, PHP 5, PHP 7)
>
> is_subclass_of — 如果此对象是该类的子类，则返回 TRUE

说明
```
bool is_subclass_of ( object $object , string $class_name )
```
如果对象 object 所属类是类 class_name 的子类，则返回 TRUE，否则返回 FALSE。



## is_tainted
> (PECL taint >=0.1.0)
>
> is_tainted — 检查字符串是否被污染

说明
```
bool is_tainted ( string $string )
```
检查字符串是否被污染

## is_resource
> (PHP 4, PHP 5, PHP 7)
>
> is_resource — 检测变量是否为资源类型

描述
```
bool is_resource ( mixed $var )
```
如果给出的参数 var 是 resource 类型，is_resource() 返回 TRUE，否则返回 FALSE。

PHP中[资源类型列表](http://php.net/manual/zh/resource.php)

## is_readable
> (PHP 4, PHP 5, PHP 7)
>
> is_readable — 判断给定文件名是否可读

说明
```
bool is_readable ( string $filename )
```
判断给定文件名是否存在并且可读。


## is_writable
> (PHP 4, PHP 5, PHP 7)
>
> is_writable — 判断给定的文件名是否可写

说明
```
bool is_writable ( string $filename )
```
如果文件存在并且可写则返回 TRUE。filename 参数可以是一个允许进行是否可写检查的目录名。

**记住 PHP 也许只能以运行 webserver 的用户名（通常为 'nobody'）来访问文件。不计入安全模式的限制。**

## is_scalar
> PHP 4 >= 4.0.5, PHP 5, PHP 7)
>
> is_scalar — 检测变量是否是一个标量

描述
```
bool is_scalar ( mixed $var )
```
如果给出的变量参数 var 是一个标量，is_scalar() 返回 TRUE，否则返回 FALSE。

标量变量是指那些包含了 integer、float、string 或 boolean的变量，而 array、object 和 resource 则不是标量。


## is_long
> (PHP 4, PHP 5, PHP 7)
>
> is_long — is_int() 的别名

描述

此函数是 is_int() 的别名函数。


## is_double
> (PHP 4, PHP 5, PHP 7)
>
> is_double — is_float() 的别名

描述

此函数是 is_float() 的别名函数。


## is_array
> (PHP 4, PHP 5, PHP 7)
>
> is_array — 检测变量是否是数组

描述
```
bool is_array ( mixed $var )
```
如果 var 是 array，则返回 TRUE，否则返回 FALSE。

## is_object
> (PHP 4, PHP 5, PHP 7)
>
> is_object — 检测变量是否是一个对象

描述
```
bool is_object ( mixed $var )
```
如果 var 是一个 object 则返回 TRUE，否则返回 FALSE。


## is_null
> (PHP 4 >= 4.0.4, PHP 5, PHP 7)
>
> is_null — 检测变量是否为 NULL

描述
```
bool is_null ( mixed $var )
```
如果 var 是 null 则返回 TRUE，否则返回 FALSE。

查看 NULL 类型获知变量什么时候被认为是 NULL，而什么时候不是。

特殊的 NULL 值表示一个变量没有值。NULL 类型唯一可能的值就是 NULL。在下列情况下一个变量被认为是 NULL：

- 被赋值为 NULL。

- 尚未被赋值。

- 被 unset()。


## is_string
> (PHP 4, PHP 5, PHP 7)
>
> is_string — 检测变量是否是字符串

描述
```
bool is_string ( mixed $var )
```
如果 var 是 string 则返回 TRUE，否则返回 FALSE。


## is_finite
> (PHP 4 >= 4.2.0, PHP 5, PHP 7)
>
> is_finite — 判断是否为有限值

说明
```
bool is_finite ( float $val )
```
检查 val 是否是是本机平台上浮点数所允许范围中的一个合法的有限值。

## is_callable
> (PHP 4 >= 4.0.6, PHP 5, PHP 7)
>
> is_callable — 检测参数是否为合法的可调用结构

说明
```
bool is_callable ( callable $name [, bool $syntax_only = false [, string &$callable_name ]] )
```
验证变量的内容能否作为函数调用。 这可以检查包含有效函数名的变量，或者一个数组，包含了正确编码的对象以及函数名。

## is_numeric
> (PHP 4, PHP 5, PHP 7)
>
> is_numeric — 检测变量是否为数字或数字字符串

描述
```
bool is_numeric ( mixed $var )
```
如果 var 是数字和数字字符串则返回 TRUE，否则返回 FALSE。


## is_infinite
> (PHP 4 >= 4.2.0, PHP 5, PHP 7)
>
> is_infinite — 判断是否为无限值

说明
```
bool is_infinite ( float $val )
```
如果 val 为无穷大（正的或负的），例如 log(0) 的结果或者任何超出本平台的浮点数范围的值，则返回 TRUE。



## is_executable
> (PHP 4, PHP 5, PHP 7)
>
> is_executable — 判断给定文件名是否可执行

说明
```
bool is_executable ( string $filename )
```
判断给定文件名是否可执行。



## is_int
> (PHP 4, PHP 5, PHP 7)
>
> is_int — 检测变量是否是整数

描述 ¶
```
bool is_int ( mixed $var )
```
如果 var 是 integer 则返回 TRUE，否则返回 FALSE。

**Note:**
若想测试一个变量是否是数字或数字字符串（如表单输入，它们通常为字符串），必须使用 is_numeric()。



## is_nan
> (PHP 4 >= 4.2.0, PHP 5, PHP 7)
>
> is_nan — 判断是否为合法数值

说明
```
bool is_nan ( float $val )
```
如果 val 为“非数值”，例如 acos(1.01) 的结果，则返回 TRUE。


## is_link
> (PHP 4, PHP 5, PHP 7)
>
> is_link — 判断给定文件名是否为一个符号连接

说明
```
bool is_link ( string $filename )
```
判断给定文件名是否为一个符号连接。



## is_uploaded_file
> (PHP 4 >= 4.0.3, PHP 5, PHP 7)
>
> is_uploaded_file — 判断文件是否是通过 HTTP POST 上传的

说明
```
bool is_uploaded_file ( string $filename )
```
如果 filename 所给出的文件是通过 HTTP POST 上传的则返回 TRUE。这可以用来确保恶意的用户无法欺骗脚本去访问本不能访问的文件，例如 /etc/passwd。

**这种检查显得格外重要**，如果上传的文件有可能会造成对用户或本系统的其他用户显示其内容的话。

为了能使 is_uploaded_file() 函数正常工作，变量指定类似于 $\_FILES['userfile']\['tmp_name'] 的变量，而在从客户端上传的文件名 $\_FILES['userfile']\['name'] 不能正常运作。
