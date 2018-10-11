---
title: Xdebug之代码覆盖率分析
date: 2016-10-22 20:30:00
tags:
- Xdebug
categories:
- PHP
---

代码覆盖率告诉您在请求期间已执行了哪些行的脚本（或一组脚本）。 有了这些信息，你可以找出你的单元测试有多好。

<!-- more -->

## 相关设置

### xdebug.coverage_enable

类型：boolean，默认值：1，在Xdebug> = 2.2中引入

如果此设置设置为0，则Xdebug不会设置内部结构以允许代码覆盖。 这加快了Xdebug相当有点，但当然，代码覆盖率分析将不工作。

## 相关函数

### boolean xdebug_code_coverage_started（）

返回代码覆盖是否处于活动状态。

返回代码覆盖是否已开始。


Example:

```php
<?php
    var_dump(xdebug_code_coverage_started());

    xdebug_start_code_coverage();

    var_dump(xdebug_code_coverage_started());
?>  
```

Returns:

```
bool(false)
bool(true)
```

### array xdebug_get_code_coverage（）

返回代码覆盖率信息

返回一个结构，其中包含有关在脚本中执行哪些行（包括include文件）的信息。 以下示例显示了一个特定文件的代码覆盖率：


Example:
```php
<?php
    xdebug_start_code_coverage();

    function a($a) {
        echo $a * 2.5;
    }

    function b($count) {
        for ($i = 0; $i < $count; $i++) {
            a($i + 0.17);
        }
    }

    b(6);
    b(10);

    var_dump(xdebug_get_code_coverage());
?>  
```

Returns:
```
array
  '/home/httpd/html/test/xdebug/docs/xdebug_get_code_coverage.php' =>
    array
      5 => int 1
      6 => int 1
      7 => int 1
      9 => int 1
      10 => int 1
      11 => int 1
      12 => int 1
      13 => int 1
      15 => int 1
      16 => int 1
      18 => int 1
```

### void xdebug_start_code_coverage（[int options]）

开始代码覆盖

此函数开始收集代码覆盖的信息。 收集的信息包括一个二维数组，其主要索引为执行的文件名，辅助键为行号。 元素中的值表示行是否已执行或它是否具有不可达行。


每行的返回值为：

- 1：这行被执行

- -1：此行未执行

- -2：这行没有可执行代码就可以了

值-1仅在启用XDEBUG_CC_UNUSED时返回，并且仅当启用了XDEBUG_CC_UNUSED和XDEBUG_CC_DEAD_CODE时才返回值-2。

此函数有两个选项，用作位字段：

- XDEBUG_CC_UNUSED
> 启用代码扫描，以确定哪行有可执行代码。 如果没有这个选项，返回的数组将只包含实际执行的行。

- XDEBUG_CC_DEAD_CODE
> 启用分支分析以确定是否可以执行代码。
> 启用这些选项会使代码覆盖率显着降低。
> 您可以使用以下示例中所示的选项。

Example:
```php
<?php
xdebug_start_code_coverage( XDEBUG_CC_UNUSED | XDEBUG_CC_DEAD_CODE );
?>
```

### void xdebug_stop_code_coverage（[int cleanup = true]）
停止代码覆盖

此功能停止收集信息，内存中的信息将被销毁。 如果传递“false”作为参数，那么代码覆盖率信息不会被销毁，因此您可以再次使用xdebug_start_code_coverage（）函数恢复信息收集。
