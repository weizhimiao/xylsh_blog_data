---
title: 如何查看表的相关信息
date: 2016-09-17 22:00:00
tags:
categories:
- MySQL
---

在文件系统中，MySQL将每个数据库（也被称为schema）保存为数据目录下的一个子目录。在创建表的时候，MySQL会在数据库子目录下创建一个和表同名的 .frm 文件对表的定义。例如，创建一个名为 mytable 的表，MySQL会在 mytable.frm 文件中保存对该表的定义。

通常，我们可以使用 show table status 命令来显示表的相关信息。（MySQL 5.0 + 的版本，也可以查看 INFOMATION_SCHEMA 中对应的表的信息）。
例如，对于MySQL数据库中的 user 表：
```sql
mysql> show table status like 'user' \G;
*************************** 1. row ***************************
           Name: user
         Engine: MyISAM
        Version: 10
     Row_format: Dynamic
           Rows: 14
 Avg_row_length: 63
    Data_length: 952
Max_data_length: 281474976710655
   Index_length: 2048
      Data_free: 64
 Auto_increment: NULL
    Create_time: 2012-12-25 14:23:08
    Update_time: 2015-08-11 11:17:42
     Check_time: NULL
      Collation: utf8_bin
       Checksum: NULL
 Create_options:
        Comment: Users and global privileges
1 row in set (0.01 sec)
```
<!-- more -->

下面简单介绍一下每行的含义：
- Name
> 表名。

- Engine
> 表的存储引擎。

- Row_format
> 行的格式，对于MyISAM表，可选的值为Dynamic、Fixed、Compressed。
  - Dynamic，表示行的长度是可变的，一般包含可变长度的字段，如varchar、blob等。
  - Fixed，行的长度是固定的，只包含固定长短的列。
  - Compressed，只在压缩表中出现，表示是被压缩的。

- Rrows
> 表中行数，对于MyISAM和其他的一些存储引擎该值是精确的，但对于InnoDB该值只是一个大概值。

- Avg_row_length
> 平均每行包含的字节数

- Data_length
> 表数据的大小(单位：字节)

- Max_data_length
> 表数据的最大容量，该值和存储引擎有关。

- Index_length
> 索引的大小(单位：字节)

- Data_free
> 对于MyISAM表示已经分配但是目前没有被使用的空间。这部分空间包括之前删除的行，以及后续可以被Insert利用的行。

- Auto_increment
> 下一个Auto_increment的值。

- Create_time
> 表的创建时间。

- Update_time
> 表数据的最后修改时间。

- Check_time
> 使用check table命令或者muisamchk 工具最后一次检查表的时间。

- Collation
> 表的默认字符集和字符列排序规则。

- Checksum
> 如果启用，保存的是整个表的实时校验和。

- Create_options
> 创建表时指定的其他选项。

- Comment
> 该列包含了一些其他的额外信息。对于MyISAM表，保存的是在表创建的时候附带的注释。对于InnoDB表，该值保存的是InnoDB表空间的剩余空间信息。如果是一个视图，该列将会包含『VIEW』字样。
