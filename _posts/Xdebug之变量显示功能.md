---
title: Xdebug之变量显示功能
date: 2016-10-22 20:30:00
tags:
- Xdebug
categories:
- PHP
---


Xdebug替换了PHP的var_dump（）函数来显示变量。 Xdebug的版本包括不同类型的不同颜色，并且限制了数组元素/对象属性的数量，最大深度和字符串长度。 还有一些其他函数处理可变显示。

<!-- more -->

有一些设置控制Xdebug修改的var_dump（）函数的输出：
- xdebug.var_display_max_children，
- xdebug.var_display_max_data
- xdebug.var_display_max_depth

 这三个设置的效果最好用一个例子。 下面的脚本运行四次，每次使用不同的设置。 您可以使用选项卡查看差异。
```php
<?php
class test {
    public $pub = false;
    private $priv = true;
    protected $prot = 42;
}
$t = new test;
$t->pub = $t;
$data = array(
    'one' => 'a somewhat long string!',
    'two' => array(
        'two.one' => array(
            'two.one.zero' => 210,
            'two.one.one' => array(
                'two.one.one.zero' => 3.141592564,
                'two.one.one.one'  => 2.7,
            ),
        ),
    ),
    'three' => $t,
    'four' => range(0, 5),
);
var_dump( $data );
?>
```

- 默认
```
array
  'one' => string 'a somewhat long string!' (length=23)
  'two' =>
    array
      'two.one' =>
        array
          'two.one.zero' => int 210
          'two.one.one' =>
            array
              ...
  'three' =>
    object(test)[1]
      public 'pub' =>
        &object(test)[1]
      private 'priv' => boolean true
      protected 'prot' => int 42
  'four' =>
    array
      0 => int 0
      1 => int 1
      2 => int 2
      3 => int 3
      4 => int 4
      5 => int 5
```

- children=2
```
array
  'one' => string 'a somewhat long string!' (length=23)
  'two' =>
    array
      'two.one' =>
        array
          'two.one.zero' => int 210
          'two.one.one' =>
            array
              ...
  more elements...
```

- data=16
```
array
  'one' => string 'a somewhat long '... (length=23)
  'two' =>
    array
      'two.one' =>
        array
          'two.one.zero' => int 210
          'two.one.one' =>
            array
              ...
  'three' =>
    object(test)[1]
      public 'pub' =>
        &object(test)[1]
      private 'priv' => boolean true
      protected 'prot' => int 42
  'four' =>
    array
      0 => int 0
      1 => int 1
      2 => int 2
      3 => int 3
      4 => int 4
      5 => int 5
```

- depth=2
```
array
  'one' => string 'a somewhat long string!' (length=23)
  'two' =>
    array
      'two.one' =>
        array
          ...
  'three' =>
    object(test)[1]
      public 'pub' =>
        &object(test)[1]
      private 'priv' => boolean true
      protected 'prot' => int 42
  'four' =>
    array
      0 => int 0
      1 => int 1
      2 => int 2
      3 => int 3
      4 => int 4
      5 => int 5
```

- children=3, data=8, depth=1
```
array
  'one' => string 'a somewh'... (length=23)
  'two' =>
    array
      ...
  'three' =>
    object(test)[1]
      ...
  more elements...
```

## 相关设置

### **xdebug.cli_color**

类型：整数，默认值：0，在Xdebug> 2.2中引入

- 如果此设置为1，则Xdebug将在CLI模式下和输出为tty时对var_dumps和堆栈跟踪输出进行颜色。在Windows上，需要安装ANSICON工具。

- 如果设置为2，那么无论Xdebug是否连接到tty或是否安装了ANSICON，Xdebug都将始终对var_dumps和堆栈跟踪进行着色。在这种情况下，您可能会看到转义码。

有关更多信息，请参阅此[文章](http://drck.me/clicolor-9cr)。


### **xdebug.overload_var_dump**

类型：boolean，默认值：2，在Xdebug> 2.1中引入
- 默认情况下，当html_errors php.ini设置为1或2时，Xdebug会自动重载var_dump（）并显示变量。

- 如果您不想要，可以将此设置设置为0，但首先检查它是否不是更聪明的关闭html_errors。

- 您也可以使用2作为此设置的值。除了格式化var_dump（）输出很好，它还将添加文件名和行号到输出。还尊重xdebug.file_link_format设置。 （Xdebug 2.3中的新功能）

在Xdebug 2.4之前，此设置的默认值为1。


### **xdebug.var_display_max_children**

类型：整数，默认值：128

控制使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时显示的数组子元素和对象的属性的数量。

- 要禁用任何限制，请使用-1作为值。

此设置对通过远程调试功能发送到客户端的子项数没有任何影响。


### **xdebug.var_display_max_data**

类型：整数，默认值：512

控制使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时显示的最大字符串长度。

- 要禁用任何限制，请使用-1作为值。

此设置对通过远程调试功能发送到客户端的子项数没有任何影响。


### **xdebug.var_display_max_depth**

类型：整数，默认值：3

当使用xdebug_var_dump（），xdebug.show_local_vars或通过函数跟踪显示变量时，控制数组元素和对象属性的嵌套级别数。

- 您可以选择的最大值为1023.您还可以使用-1作为值来选择此最大数。

此设置对通过远程调试功能发送到客户端的子项数没有任何影响。


## 相关函数

### **void var_dump（[mixture var [，...]]）**
> 显示有关变量的详细信息

此函数由Xdebug重载，请参阅xdebug_var_dump（）的说明。

### **void xdebug_debug_zval（[string varname [，...]]）**
> 显示有关变量的信息

此函数显示有关一个或多个变量的结构化信息，包括其类型，值和引用计数信息。使用值递归地探索数组。这个函数的实现不同于PHP的debug_zval_dump（）函数，以解决该函数的问题，因为变量本身实际上传递给函数。 Xdebug的版本是更好的，因为它使用变量名称来查找内部符号表中的变量，并直接访问所有的属性，而不必处理实际传递一个变量到一个函数。结果是，该函数返回的信息比PHP自己的用于显示zval信息的函数准确得多。

从Xdebug 2.3开始支持除了简单变量名（例如下面的“a [2]”）之外的任何东西。


Example:
```php
<?php
    $a = array(1, 2, 3);
    $b =& $a;
    $c =& $a[2];

    xdebug_debug_zval('a');
    xdebug_debug_zval("a[2]");
?>
```
结果：(refcount:引用次数，)
```
a: (refcount=2, is_ref=1)=array (
	0 => (refcount=1, is_ref=0)=1,
	1 => (refcount=1, is_ref=0)=2,
	2 => (refcount=2, is_ref=1)=3)
a[2]: (refcount=2, is_ref=1)=3
```


### **void xdebug_debug_zval_stdout（[string varname [，...]]）**
> 将有关变量的信息返回到stdout。

此函数显示有关一个或多个变量的结构化信息，包括其类型，值和引用计数信息。 使用值递归地探索数组。 与xdebug_debug_zval（）的区别在于，该信息不通过Web服务器API层显示，而是直接显示在stdout上（因此，在Apache以单进程模式运行时，它会在控制台上运行）。

Example:
```
<?php
    $a = array(1, 2, 3);
    $b =& $a;
    $c =& $a[2];

    xdebug_debug_zval_stdout('a');

```
返回：
```
a: (refcount=2, is_ref=1)=array (
	0 => (refcount=1, is_ref=0)=1,
	1 => (refcount=1, is_ref=0)=2,
	2 => (refcount=2, is_ref=1)=3)
```

### **void xdebug_dump_superglobals（）**
> 显示有关超级全局的信息

此函数转储指定的超级全局元素的值通过php.ini中的xdebug.dump.\*选项。 对于下面的示例，php.ini中的设置是：

Example:
```
xdebug.dump.GET=*
xdebug.dump.SERVER=REMOTE_ADDR

Query string:
?var=fourty%20two&array[a]=a&array[9]=b
```

结果：
![img](http://n.sinaimg.cn/games/3ece443e/20161022/8F0658D7-C951-4B12-BC74-92E15E31C44C.png)


### **void xdebug_var_dump（[mixed var [，...]]）**
> 显示有关变量的详细信息

此函数显示有关包含其类型和值的一个或多个表达式的结构化信息。 使用值递归地探索数组。 请参阅变量显示功能的介绍，其中php.ini设置影响此功能。

Example:
```
<?php
ini_set('xdebug.var_display_max_children', 3 );
$c = new stdClass;
$c->foo = 'bar';
$c->file = fopen( '/etc/passwd', 'r' );
var_dump(
    array(
        array(TRUE, 2, 3.14, 'foo'),
        'object' => $c
    )
);
?>  
```
结果
```
array
  0 =>
    array
      0 => boolean true
      1 => int 2
      2 => float 3.14
      more elements...
  'object' =>
    object(stdClass)[1]
      public 'foo' => string 'bar' (length=3)
      public 'file' => resource(3, stream)
```
