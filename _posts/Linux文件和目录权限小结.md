---
title: Linux文件和目录权限小结
date: 2016-10-06 22:30:00
categories:
- Linux
---


## 普通权限
文件的权限：	- r  w  -  -  w  x  r  -  x
```
第一位：文件的类型
    -：文件
    d：文件夹
    l：连接
    c：字符设备文件
    b：块设备
    s：套接口文件
第二位：所有者读权限
第三位：所有者写权限
第四位：所有者执行权限
第五位：所有者组读权限
第六位：所有者组写权限
第七位：所有者组执行权限
第八位：其他组读权限
第九位：其他组写权限
第十位：其他组执行权限
r		4
w		2
x		1
```

<!-- more -->

## 特殊权限（SUID/SGID/SBIT）

先来看看两个特殊的文件与目录
```
[root@yufei ~]# ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 26968 Jan 29  2010 /usr/bin/passwd
[root@yufei ~]# ls -l /usr/bin/wall
-r-xr-sr-x. 1 root tty 10932 Apr 27  2010 /usr/bin/wall
[root@yufei ~]# ls -ld /tmp/
drwxrwxrwt. 7 root root 4096 Jan 20 11:00 /tmp/
```
第一个passwd命令在所有者的地方多了一个s，
第二个wall命令在用户组的位置多了一个s，
第三个tmp目录，多了一个t。

这是为什么呢？下面我们就来具体看看，这些特殊的权限是什么意思？如何设置？

### 特殊权限的介绍Set UID**

当s这个标志出现在文件所有者的x权限上时，如/usr/bin/passwd这个文件的权限状态：“-rwsr-xr-x.”，此时就被称为Set UID，简称为SUID。那么这个特殊权限的特殊性的作用是什么呢？

- 1、SUID权限仅对二进制程序(binary program)有效；
- 2、执行者对于该程序需要具有x的可执行权限；
- 3、本权限仅在执行该程序的过程中有效(run-time)；
- 4、执行者将具有该程序拥有者(owner)的权限。

SUID的目的就是：让本来没有相应权限的用户运行这个程序时，可以访问他没有权限访问的资源。passwd就是一个很鲜明的例子，下面我们就来了解一下这相passwd执行的过程。

我们知道，系统中的用户密码是保存在/etc/shadow中的，而这个文件的权限是———-. （这个权限和以前版本的RHEL也有差别，以前的是-r——–）。其实有没有r权限不重要，因为我们的root用户是拥有最高的权限，什么都能干了。关键是要把密码写入到/etc/shadow中。我们知道，除了root用户能修改密码外，用户自己同样也能修改密码，为什么没有写入权限，还能修改密码，就是因为这个SUID功能。

下面就是passwd这个命令的执行过程
- 1、因为/usr/bin/passwd的权限对任何的用户都是可以执行的，所以系统中每个用户都可以执行此命令。
- 2、而/usr/bin/passwd这个文件的权限是属于root的。
- 3、当某个用户执行/usr/bin/passwd命令的时候，就拥有了root的权限了。
- 4、于是某个用户就可以借助root用户的权力，来修改了/etc/shadow文件了。
- 5、最后，把密码修改成功。

注：这个SUID只能运行在二进制的程序上（系统中的一些命令），不能用在脚本上（script），因为脚本还是把很多的程序集合到一起来执行，而不是脚本自身在执行。同样，这个SUID也不能放到目录上，放上也是无效的。

### Set GID

我们前面讲过，当s这个标志出现在文件所有者的x权限上时，则就被称为Set UID。那么把这个s放到文件的所属用户组x位置上的话，就是SGID。如开头的/usr/bin/wall命令。
那么SGID的功能是什么呢？和SUID一样，只是SGID是获得该程序所属用户组的权限。

这相SGID有几点需要我们注意：
- 1、SGID对二进制程序有用；
- 2、程序执行者对于该程序来说，需具备x的权限；
- 3、SGID主要用在目录上；
理解了SUID，我想SGID也很容易理解。如果用户在此目录下具有w权限的话，若使用者在此目录下建立新文件，则新文件的群组与此目录的群组相同。

### Sticky Bit

这个就是针对others来设置的了，和上面两个一样，只是功能不同而已。

SBIT（Sticky Bit）目前只针对目录有效，对于目录的作用是：当用户在该目录下建立文件或目录时，仅有自己与 root才有权力删除。

最具有代表的就是/tmp目录，任何人都可以在/tmp内增加、修改文件（因为权限全是rwx），但仅有该文件/目录建立者与 root能够删除自己的目录或文件。

**注：这个SBIT对文件不起作用。**

### SUID/SGID/SBIT权限设置

和我们前面说的rwx差不多，也有两种方式，一种是以字符，一种是以数字。

- 4 为 SUID ＝ u+s
- 2 为 SGID ＝ g+s
- 1 为 SBIT ＝ o+t

下面我们就来看看如何设置，并看看达到的效果。

#### 先看SUID的作用及设置
```
[root@yufei ~]# cd /tmp/
[root@yufei tmp]# cp /usr/bin/passwd ./
[root@yufei tmp]# mkdir testdir
```
上面两步是在/tmp目录下创建passwd文件和testdir目录

下面看看这两个的权限
```
[root@yufei tmp]# ls -l passwd ; ls -ld testdir/
-rwxr-xr-x. 1 root root 26968 Jan 20 23:27 passwd
drwxr-xr-x. 2 root root 4096 Jan 20 19:25 testdir/
```
我们切换到yufei用户，然后修改自己的密码
```
[root@yufei tmp]# su yufei
[yufei@yufei tmp]$ ./passwd
Changing password for user yufei.
Changing password for yufei.
(current) UNIX password:
New password:           
Retype new password:
passwd: Authentication token manipulation error
```
发现上面的yufei改不了自己的密码，为什么呢？就是因为没有权限把密码写入到/etc/shadow中。想让普通用户能修改/etc/shadow的话，那就需要用到SUID了。
```
[yufei@yufei tmp]$ su root
Password:
[root@yufei tmp]# chmod u+s passwd
[root@yufei tmp]# ls -l passwd
-rwsr-xr-x. 1 root root 26968 Jan 20 23:27 passwd
```
看到有SUID权限了，下面再来修改yufei自己的密码
```
[yufei@yufei tmp]$ ./passwd
Changing password for user yufei.
Changing password for yufei.
(current) UNIX password:
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```
我们发现已经成功了。

我想这一下，你对SUID的作用已经了解了吧。

如果想把这个改回来（就是把SUID的权限去掉），我们用数字方式来设置
```
[root@yufei tmp]# chmod 0755 passwd
[root@yufei tmp]# ls -l passwd
-rwxr-xr-x. 1 root root 26968 Jan 20 23:27 passwd
```
OK这样就改过来了，这个数字的原理和我们前面讲的rwx是一样的，只是在最前面设置相应的数字而已。

**注：在普通用户修改自己的密码是，密码要设置的复杂点，否则的话，通过不了认证，普通用户和root用户的权限是不同的。**

#### 再看SGID的作用及设置

我们以前面建立的/tmp/testdir为例子
```
[root@yufei tmp]# ls -ld testdir/
[root@yufei tmp]# chmod 757 testdir/
[root@yufei tmp]# ls -ld testdir/
drwxr-xrwx. 2 root root 4096 Jan 20 19:25 testdir/
```
这时候，任何用户对此目录都有写入权限，那么我们就在这个目录里面创建文件与目录，并看看他们的权限如何
```
[root@yufei tmp]# su yufei
[yufei@yufei tmp]$ touch testdir/file1
[yufei@yufei tmp]$ mkdir testdir/dir1
[yufei@yufei tmp]$ ls -l testdir
total 0
drw-rw-r–. 1 yufei yufei 0 Jan 21 10:33 dir1
-rw-rw-r–. 1 yufei yufei 0 Jan 21 10:33 file1
```
这时候的文件与目录权限都是创建者的本身
下面我们就来看看，把这个目录加上SGID权限后，再创建文件与目录，会是什么样的效果
```
[yufei@yufei tmp]$ su root
Password:
[root@yufei tmp]# chmod g+s testdir/
[root@yufei tmp]# ls -ld testdir/
drwxr-srwx. 2 root root 4096 Jan 21 10:33 testdir/
[root@yufei tmp]# su yufei
[yufei@yufei tmp]$ touch testdir/file2
[yufei@yufei tmp]$ mkdir testdir/dir2
[yufei@yufei tmp]$ ls -l testdir/
total 0
drw-rw-r–. 1 yufei yufei 0 Jan 21 10:33 dir1
drw-rw-r–. 1 yufei root  0 Jan 21 10:36 dir2
-rw-rw-r–. 1 yufei yufei 0 Jan 21 10:33 file1
-rw-rw-r–. 1 yufei root  0 Jan 21 10:35 file2
[yufei@yufei tmp]$ ls -ld testdir/
drwxr-srwx. 2 root root 4096 Jan 21 10:36 testdir/
```
这时候我们就发现，file2和dir2的用户组变成了root了，也就是他们上层目录testdir这个目录的所属用户组。
这个应用，应用在一个项目的共同开发上，是很方便的。
```
[yufei@yufei tmp]$ su root
Password:
[root@yufei tmp]# chmod g-s testdir/
[yufei@yufei tmp]$ ls -ld testdir/
drwxr-xrwx. 2 root root 4096 Jan 21 10:36 testdir/
```
这样就还原了

#### 最后我们来看SBIT的作用及设置
```
[root@yufei tmp]# rm -fr testdir/*
[root@yufei tmp]# ls -ld testdir/
drwxr-xrwx. 2 root root 4096 Jan 21 11:42 testdir/
```
清空/tmp/testdir/目录里面的全部内容。

我们切换成普通用户，然后再里面创建文件，至少需要两个普通用户来测试这个，如果没有的话，就自己建立。
```
[root@yufei tmp]# su yufei
[yufei@yufei tmp]$ touch testdir/yufei_file
[yufei@yufei tmp]$ ls -l testdir/
total 0
-rw-rw-r– 1 yufei yufei 0 Jan 21 11:45 yufei_file
```
这时候我们建立了一个文件，我们换成另外一个用户
```
[yufei@yufei tmp]$ suopsers
Password:
[opsers@yufei tmp]$ ls -ld testdir/
drwxr-xrwx. 2 root root 4096 Jan 21 11:45 testdir/
```
我们看到，虽然其他用户对yufei_file只有只读权限，但由于yufei_file所在的目录，对其他人是全部的权限，所以，我们换其他用户还是可以删除这个文件的，看操作
```
[opsers@yufei tmp]$ rm -f testdir/yufei_file
[opsers@yufei tmp]$ ls testdir/
```
发现我们已经删除了这个不属于我们的权限。
下面我们就给这个目录加上SBIT权限，再来看看效果
```
[opsers@yufei tmp]$ su root
Password:
[root@yufei tmp]# chmod o+t testdir
[root@yufei tmp]# ls -ld testdir/
drwxr-xrwt. 2 root root 4096 Jan 21 11:49 testdir/
```
再一次切换普通用户，创建文件
```
[root@yufei tmp]# su yufei
[yufei@yufei tmp]$ touch testdir/yufei_file
[yufei@yufei tmp]$ ls -l testdir/yufei_file
-rw-rw-r– 1 yufei yufei 0 Jan 21 11:51 testdir/yufei_file
```
这个文件的权限还是和第一次创建的时候是一样的，我们再换成其他的用户，看看能不能再次删除这个文件
```
[yufei@yufei tmp]$ su opsers
Password:
[opsers@yufei tmp]$ rm -f testdir/yufei_file
rm: cannot remove `testdir/yufei_file’: Operation not permitted
```
看到提示，说权限不够了，只能由这个文件的创建者或root用户才能删除。这个我们就不演示了。
如果要还原权限的话，
```
[opsers@yufei tmp]$ su root
Password:
[root@yufei tmp]# chmod o-t testdir
[root@yufei tmp]# ls -ld testdir/
drwxr-xrwx. 2 root root 4096 Jan 21 11:51 testdir/
```
两个需要注意的问题

OK，关于SUID/SGID/SBIT这些特殊权限的应用和作用我们已经完了。但如果你仔细一点的话，会发现，我并没有用数字方式来更改这个特殊的权限，为什么呢？且看下面的分析。

### question

#### 问题1：用数字改变目录的特殊权限，不起作用。**

我们把/tmp/下面，我们自己建立的实验文件删除
```
[root@yufei tmp]# rm -fr testdir/
[root@yufei tmp]# rm -fr passwd
```
然后再重新创建一个文件和目录，
```
[root@yufei tmp]# cp /usr/bin/passwd ./
[root@yufei tmp]# mkdir testdir
[root@yufei tmp]# ls -l passwd ;ls -ld testdir/
-rwxr-xr-x 1 root root 26968 Jan 21 12:00 passwd
drwxr-xr-x 2 root root 4096 Jan 21 12:00 testdir/
```
下面我们就来用数字方式来更改这三个特殊的权限，看看会有什么样的结果
```
[root@yufei tmp]# chmod 4755 passwd
[root@yufei tmp]# chmod 3755 testdir/
[root@yufei tmp]# ls -l passwd ;ls -ld testdir/
-rwsr-xr-x 1 root root 26968 Jan 21 12:00 passwd
drwxr-sr-x 2 root root 4096 Jan 21 12:00 testdir/
```
发现用这种方式增加这三个特殊权限没有问题，那么我们再把权限改回去看看
```
[root@yufei tmp]# chmod 0755 passwd
[root@yufei tmp]# chmod 0755 testdir/
[root@yufei tmp]# ls -l passwd ;ls -ld testdir/
-rwxr-xr-x 1 root root 26968 Jan 21 12:00 passwd
drwxr-sr-x 2 root root 4096 Jan 21 12:00 testdir/
```
我们发现，对文件，权限是改回去了，而对于目录，只改回去了SBIT的权限，对SUID和SGID改不回去。这是RHEL6上的实验结果，可能是出于安全性的考虑吗？这个我就不清楚了，也找不到相关的资料。

所以说，建议大家还是用最明了的方式，直接用+-来更改，无论方法如何，最终能得到结果就OK了。哈哈……

#### 问题2：为什么会有大写的S和T。

还是用上面的文件和目录
```
[root@yufei tmp]# ls -l passwd ;ls -ld testdir/
-rwxr-xr-x 1 root root 26968 Jan 21 12:00 passwd
drwxr-sr-x 2 root root 4096 Jan 21 12:00 testdir/
```
我们把passwd和testdir的x权限去掉
```
[root@yufei tmp]# chmod u-x passwd
[root@yufei tmp]# chmod o-x testdir/
[root@yufei tmp]# ls -l passwd ;ls -ld testdir/
-rw-r-xr-x 1 root root 26968 Jan 21 12:00 passwd
drwxr-sr– 2 root root 4096 Jan 21 12:00 testdir/
```
再给他们加上SUID和SBIT权限
```
[root@yufei tmp]# chmod u+s passwd
[root@yufei tmp]# chmod o+t testdir/
[root@yufei tmp]# ls -l passwd ;ls -ld testdir/
-rwSr-xr-x 1 root root 26968 Jan 21 12:00 passwd
drwxr-sr-T 2 root root 4096 Jan 21 12:00 testdir/
```
我们看到，这时候的小s和小t已经变成了大S和大T了，为什么呢？因为他们这个位置没有了x权限，如果没有了x权限，根据我们上面讲的内容，其实，这个特殊的权限就相当于一个空的权限，没有意义。也就是说，如果你看到特殊权限位置上变成了大写的了，那么，就说明，这里有问题，需要排除。


##	ACL权限（用来解决用户权限身份不足的问题）
问题：只要使用ACL权限递归赋予权限，就很有可能出现权限溢出的危险（应为文件和目录的wrx权限含义是不同的）

举例：
```
/www
	sc --> root
	61 -->  fgroup
	o
	770		
```
```
[root@localhost ~]# mkdir /www
[root@localhost ~]# chmod 770 /www/
[root@localhost ~]# groupadd fgroup
[root@localhost ~]# gpasswd -a sc fgroup
```
正在将用户“sc”加入到“fgroup”组中
```
[root@localhost ~]# gpasswd -a aa fgroup
```
正在将用户“aa”加入到“fgroup”组中
```
[root@localhost ~]# chown root:fgroup  /www
[root@localhost ~]# ll -d  /www/
drwxrwx--- 2 root fgroup 4096 04-25 14:56 /www/
```

###	getfacl  文件名		查询文件的acl权限

###	setfacl  选项  文件名		设定acl权限
```
	-m			设定权限
	-b			删除权限
setfacl  -m  u:用户名:权限   文件名
setfacl  -m  g:组名：权限   文件名
setfacl  -m u:aa:rwx  /test		给test目录赋予aa是读写执行的acl权限

setfacl -m u:cc:rx -R soft/		赋予递归acl权限，只能赋予目录
		-R  递归
setfacl  -b  /test		删除acl权限
```
###	setfacl  -m d:u:aa:rwx -R /test	acl默认权限。		
> 注意：默认权限只能赋予目录
>
> 注意：如果给目录赋予acl权限，两条命令都要输入

- -R 递归
- -m  u:用户名：-R 权限		只对已经存在的文件生效
- -m  d:u:用户名：-R 权限		只对未来要新建的文件生效
