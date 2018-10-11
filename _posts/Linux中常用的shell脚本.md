---
title: Linux中常用的shell脚本
date: 2016-10-30 20:30:00
tags:
- shell
categories:
- Linux
---

![Linux中常用的shell脚本](http://n.sinaimg.cn/games/3ece443e/20161030/bash.png)

<!-- more -->
1、对于进程来说，查看其运行时的环境变量
```
cat /proc/$PID/environ
```
示例
```
#首先需要我们运行一个程序
# pgrep inotifywait
8977
# cat /proc/8977/environ
HOSTNAME=localhost.localdomainSELINUX_ROLE_REQUESTED=TERM=xterm-256colorSHELL=/bin/bashHISTSIZE=1000SSH_CLIENT=192.168.1.100 63684 22SELINUX_USE_CURRENT_RANGE=SSH_TTY=/dev/pts/0USER=rootLS_COLORS=rs=0:di=38;... %sG_BROKEN_FILENAMES=1_=/usr/local/bin/inotifywaitOLDPWD=/data
#一般返回的环境变量会有很多，我们采用换行方式查看会更直观

# cat /proc/8977/environ | tr '\0' '\n'
HOSTNAME=localhost.localdomain
SELINUX_ROLE_REQUESTED=
TERM=xterm-256color
SHELL=/bin/bash
HISTSIZE=1000
SSH_CLIENT=192.168.1.100 63684 22
SELINUX_USE_CURRENT_RANGE=
SSH_TTY=/dev/pts/0
USER=root
....
```
2、获得字符串长度
```
# var=wahaha
# echo ${var}
wahaha
# echo ${#var}
6
```
3、识别当前使用的是哪种shell
```
echo $SHELL
/bin/bash
```
4、shell脚本检查是执行当前脚本的是否为root用户
```
# cat check_root.sh
#!/bin/bash
if [ $UID -ne 0 ]; then
	echo "Non root user. Please run as root."
else
	echo Root User
fi
```
5、修改bash提示字符串（username@hastname:~$）
```
#利用PS1环境变量来定制
# cat ~/.bashrc | grep PS1
PS1='${debian_chroot:+$($debian_chroot)}\u@\h:\w\$ '
# \u 用户名
# \h 主机名
# \w 当前工作目录
```
6、如何判断一个命令是否执行成功
```
当一个命令发生错误并退回时，他会返回一个非0的退出状态；
而当命令成功完成后，它会返回数字0.
退出状态可以从特殊变量 $? 中获得（在命令执行之后立刻运行 $? ,就可以打印出退出状态）
```
```
#!/bin/bash
CMD="commod"  #commod代表需要检测退出状态的命令
$CMD
if [ $? -eq 0 ];
then
  echo "$CMD executed succ"
else
  echo "$CMD excuted unsucc"
fi
```

7、shell中数组的定义与操作
```
#定义1
array_var=(1 2 3 4 5)
#定义2
array_var[0]="test"
array_var[1]="test1"
array_var[2]="test2"
array_var[3]="test3"
#打印特定索引数组元素
echo ${array_var[0]}
#以清单形式打印出数组中的所有值
echo ${array_var[*]}
test test1 test2 test3
#也可以
echo ${array_var[@]}
test test1 test2 test3
#输出数组元素个数
echo ${#array_var[*]}
4
```
8、按照指定格式打印系统当前时间
```
# date "+%Y%m%d"
20161029
# %a  星期  例，Sat
# %A  星期  例，Saturday
# %b  月    例，Nov
# %B  月    例，November
# %d  日   例，31
# %D  固定格式(mm/dd/yy)  例，10/18/10
# %y  年   例，16
# %Y  年   例，2016
# %I或%H 小时  例，08
# %M  分钟  例，30
# %S  秒   例，10
# %N  纳秒  例，694394444
```
9、shell函数参数传递
```
#!/bin/bash
func(){
  echo $1,$2  #访问参数1和参数2
  echo "$@"   #以列表方式一次性打印所有参数
  echo "$*"   #类似于$@,但是参数被作为单个实体
  return 0;   #返回值
}

# $1,  第一个参数
# $2,  第二个参数
# $n,  第n个参数
# "$@",   被扩展成"$1" "$2" "$3"等
# "$*",   被扩展成"$1c$2c$3" ，其中c是IFS的第一个字符
# "$@"要比"$*"用的多。由于"$*"将所有参数当做单个字符串，因此它很少被使用
```
10、利用子shell生成独立进程
```
# cat sub_shell.sh
#!/bin/bash
pwd;
(cd /bin; pwd);
pwd;
# ./sub_shell.sh
/root/shell
/bin
/root/shell

# 我们可以发现（）中的子shell对于当前shell没有影响，它所有的改变仅限于（）内部
```
11、运行命令直至成功
```
repeat(){
  while true
  do
    $@  && return
  done
}
# 原理：该函数通过$@接收传参。如果命令执行成功，则返回并退出循环，否则一直循环

# Tips：将该函数加入的shell的rc文件中，方便我们使用

# 示例：使用repeat（）从网上下载一个文件，直至成功
repeat wget -c http://xxxx.com/example.tar.gz
```
12、删除多余的空行
```
# cat -s file >> newfile
```
13、文件查找
```
# find /home -name "*.txt"
# find . \( -name "*.txt" -o -name "*.pdf" \)
...
```
14、排序、去重
```
# 排序  sort
# 去重  uniq
sort file.txt | uniq
```
```
# cat check_sorted.sh
#!/bin/bash
# 功能：检查文件是否已经排序过
sort -C filename;
if [ $? -eq 0 ]
then
  echo Sorted
else
  echo Unsorted
fi
```
15、交互输出自动化实现
```
#!/bin/bash
# 文件名：enter.sh
read -p "Enter number:" no ;
read -p "Enter name" name ;
echo You have entered $no $name;
```
```
#利用echo -e来生成输入序列
echo -e "1\nhello\n" | ./enter.sh
```
16、利用并行进程加速命令执行
```
#!/bin/bash
PIDARRAY=()
for file in File1.ios File2.ios
do
  md5sum $file &
  PIDARRAY+=("$!")
done
wait ${PIDARRAY[@]}
# 原理：利用bash的操作符&，它使得shell将命令置于后台并继续执行脚本。使用 $! 来获得进程的PID（在bash中 $! 保存最近一个后台进程的PID）。将这些进程PID放入PIDARRAY数组中。使用wait命令等待这些进程执行完成。
```
17、查找并删除内容重复的文件
```
#创建测试文件testfile，testfile1，testfile2，newfile ;其中testfile1和testfile2都是testfile的副本

# echo "hello" >> testfile
# cp testfile testfile1
# cp testfile testfile2
# echo hello world >> newfile

# cat remove_duplicates.sh
#!/bin/bash

ls -lS --time-style=long-iso | awk 'BEGIN {
	getline; getline;
	name1=$8; size=$5;
}{
	name2=$8
	if(size==$5){
		"md5sum "name1 | getline; csum1=$1;
		"md5sum "name2 | getline; csum2=$1;
		if(csum1==csum2){
			print name1;
			print name2;
		}
	}
	size=$5;
	name1=name2;
}' | sort -u > duplicate_files

cat duplicate_files | xargs -I {} md5sum {} | sort | uniq -w 32 | awk '{print "^"$2"$" }' | sort -u > duplicate_sample

echo Removing...
comm duplicate_files duplicate_sample -2 -3 | tee /dev/stderr | xargs rm
rm -f duplicate_files duplicate_sample
echo Removed duplicates files successfully

# 执行脚本
# ./remove_duplicates.sh
Removing...
testfile
testfile1
Removed duplicates files successfully

# 如上testfile、testfile1、testfile2这三个相同内容的文件，只保留了testfile2.

# 脚本原理说明：
# ls -lS 将当前目录下的所有文件按照文件大小进行排序，并列出文件的详细信息
# awk命令的执行步骤：awk首先会执行BEGIN{}语句块，处理完{}中所有的命令后，在执行END{}语句块
# 在awk中，外部的命令的输出可以用这样的方式来读取，"cmd" | getline ，随后我们就可以用 $0 来获取命令的输出 ，在$1,$2,$3...$n 获取命令输出中的每一列
# BEGIN{}语句块中有两个getline;是因为 ls -lS --time-style=long-iso 命令的输出的第一行是文件的数量，所以我们直接跳过，来获取下一行数据
# comm 通常只接受排过序的文件。所以在之前需要使用 sort -u 进行排序
# tee 命令在这有一个妙用：它将文件名在传递给rm命令的同时，也起到了print的效果。tee将来自stdin的行写入文件，同事将其发送到stdout
```
18、解析文本中的电子邮件地址和URL
```
# 正则表达式
#   mail：[a-zA-Z0-9.]+@[a-zA-Z0-9.]+\.[a-zA-Z]{2,4}
#   URL： (http|https)://[a-zA-Z0-9.]+\.[a-zA-Z]{2,3}

# wget www.sina.com.cn/index.html

# 解析index.html中的email
# egrep -o "[a-zA-Z0-9.]+@[a-zA-Z0-9.]+\.[a-zA-Z]{2,4}" index.html
jubao@vip.sina.com
jubao@vip.sina.com

# 解析index.html中的URL
# egrep -o "(http|https)://[a-zA-Z0-9.]+\.[a-zA-Z]{2,3}" index.html
http://www.sina.com.cn
http://www.sina.com.cn
http://auto.sina.com.cn
http://www.sina.com.cn
...
http://hq.sinajs.cn
http://rm.sina.com.cn
http://d4.sina.com.cn
http://d1.sina.com.cn
http://i1.sinaimg.cn
http://news.sina.com.cn
http://login.sina.com.cn
http://login.sina.com.cn

```

19、以纯文本形式下载网页
```
lynx:是一款基于命令行的web浏览器。我们可以利用它获取纯文本形式的网页

命令：
# lynx -dump http://www.sina.com.cn
-dump 选项表示将网页内容以ASCII编码形式打印

另，lynx会将页面中所有的超链接（<a href="link">）作为文本的页脚，单独放置在标题为 References 的文本区域。如果我们需要匹配除某个页面中所有的超链接也可以使用该方法，也省的我们用正则匹配
```
20、列出网络上所有的活动主机
```
# cat ping.sh
#!/bin/bash
for ip in 192.168.1.{1..255};
do
	ping $ip -c 2 &> /dev/null;

	if [ $? -eq 0 ]
	then
		echo $ip is alive
	fi
done

# 并行ping
# cat ping_1.sh
#!/bin/bash
for ip in 192.168.1.{1..255};
do
	(
	ping $ip -c 2 &> /dev/null;

	if [ $? -eq 0 ]
	then
		echo $ip is alive
	fi
	)&
done
wait

# 将wait放在脚本最后，它就会一直等到所有的子脚本进程全部结束
```
21、网络上利用套接字进行快速文件复制
```
# 设置侦听套接字
nc -l 1234

# 在另一终端或主机中连接到该套接字
nc 127.0.0.1 1234
message_test

# 连接到套接字后就可以在终端中输入信息回车就会发送

# 文件复制
在接收端执行以下命令：
nc -l 1234 > destination_filename
在发送端执行下列命令：
nc 127.0.0.1 1234 < source_filename
```
