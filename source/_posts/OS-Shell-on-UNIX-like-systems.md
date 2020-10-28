---
title: 【OS】Shell on UNIX-like systems
date: 2020-10-25 01:47:00
categories:
- 从硬件到OS
tags:
- Shell
- OS
- Unix
- Linux
- MacOS
catalog: true

---

# Shell overview
shell即“壳”，是用户和操作系统（系统内核）之间的接口程序，是个命令解释程序，拥有自己内建的 shell 命令集。提示符下输入的每个命令都由shell先解释然后传给操作系统内核。

Linux下有Bash、Zsh等，Windows下有CMD、PowerShell等，都是Shell。


## Linux中Shell的类型
Linux 中的 shell 有很多类型，其中最常用的几种是: Bourne shell (sh)、C shell (csh) 和 Korn shell (ksh), 各有优缺点。Bourne shell 是 UNIX 最初使用的 shell，并且在每种 UNIX 上都可以使用, 在 shell 编程方面相当优秀，但在处理与用户的交互方面做得不如其他几种shell。Linux 操作系统缺省的 shell 是Bourne Again shell，它是 Bourne shell 的扩展，简称 Bash，与 Bourne shell 完全向后兼容，并且在Bourne shell 的基础上增加、增强了很多特性。Bash放在/bin/bash中，它有许多特色，可以提供如命令补全、命令编辑和命令历史表等功能，它还包含了很多 C shell 和 Korn shell 中的优点，有灵活和强大的编程接口，同时又有很友好的用户界面。

https://blog.csdn.net/MonMama/article/details/53390610
https://blog.csdn.net/wenlifu71022/article/details/4069929

# 查找命令
http://www.ruanyifeng.com/blog/2009/10/5_ways_to_search_for_files_using_the_terminal.html

# grep命令

# 权限问题
## 管理者权限
**sudo命令** - 以系统管理者的身份执行指令，也就是说，经由 sudo 所执行的指令就好像是 root 亲自执行。
使用权限：在 /etc/sudoers 中有出现的使用者。

**su**命令为切换用户的命令。

切换为root用户执行命令：
 `sudo su root`或·`sudo su`: 切换到root用户，不改变当前变量
 `sudo su - root`或·`sudo su - `: 切换到root用户，并获得root的环境变量及执行权限
只要当前用户为管理员用户，即可使用sudo切换到root用户，此时输入的密码为当前管理员用户的密码。

若直接使用命令```su root```，则需要输入的密码为root用户的密码。输入当前管理员用户的密码会返回 su: Sorry。
可以使用管理员权限修改root用户的密码后，再执行```su root```命令，这样输入刚刚修改好的root密码即可切换到root用户如下图：

![Alt text](./su_root.png)


## 查看

```bash
ls -l 文件夹
```
权限说明一共10位: drwxrwxrwx
第一位 - 文件类型，" + "代表目录，" - "代表非目录。
2-4位 - 所有者user的权限说明
5-7位 - 组群group的权限说明
8-10位 - 其他人other的权限说明

r代表可读权限，w代表可写权限，x代表可执行权限。

> MacOS上，权限说明后可能有 + 或 @
> '+' 表示扩展属性(extended attributes)
> '@' 表示扩展安全信息(extended security information)
> 详情可通过命令 ls -le 或 ls -l@ 查看

## 修改权限
```bash
chmod o w xxx.xxx
```
授予其他人写xxx.xxx这个文件的权限

```bash
chmod go-rw xxx.xxx
```
删除xxx.xxx中组群和其他人的读和写的权限

> u 代表所有者（user）
> g 代表所有者所在的组群（group）
> o 代表其他人，但不是u和g （other）
> a 代表全部的人，也就是包括u，g和o

r w x可以换成数字表示：
r 对应 4， w 对应 2， x 对应 1
rwx 或 - 的任意组合对应的数字是唯一的，所以可用一个数字表示一组rwx权限。
如 7 = 4+2+1 表示赋予目录可读可写可执行权限

```bash
sudo chmod  -R 777 /var/test
```

# 环境变量相关
环境变量是在操作系统中一个具有特定名字的对象，它包含了一个或多个应用程序将使用到的信息。Linux是一个多用户的操作系统，每个用户登录系统时都会有一个专用的运行环境。这个默认环境就是一组环境变量的定义。每个用户都可以通过修改环境变量的方式对自己的运行环境进行配置。

## 环境变量分类
### 对所有用户生效的永久性变量（系统级）
位置：/etc/profile
添加方式：vim打开/etc/ profile文件，用export指令添加环境变量。
调用```source /etc/profile```才能立即生效，否则下次进入此用户生效。

### 对单一用户生效的永久性变量（用户级）
设置方法：在用户主目录”~”下的隐藏文件 “.bashrc”中添加自己想要的环境变量。
同样调用```source ./.bashrc ```该文件才会生效。否则只能在下次重进此用户时才能生效。

> * ~/.bash_profile是交互式login方式进入bash shell运行，只会在用户登录的时候读取一次。
> * ~/ .bashrc是交互式non-login方式进入bash shell运行，在每次打开终端进行一次新的会话时都会读取。
### 临时有效的环境变量（只对当前shell有效）
此类环境变量只对当前的shell有效。当我们退出登录或者关闭终端再重新打开时，这个环境变量就会消失。是临时的。
设置方法：直接使用export指令添加。 如```export PATH=/usr/local/bin:$PATH```


## 常用环境变量
* PPID: 是当前进程的父进程的PID
* PWD: 当前工作目录。命令pwd的输出该变量值。
* RANDOM: 随机数变量。每次引用这个变量会得到一个0~32767的随机数。
* SECONDS: 脚本已经运行的时间（以秒为单位）
* PATH 指定命令的搜索路径。通过设置环境变量PATH可以让我们运行程序或指令更加方便。
> 每一个冒号都是一个路径，这些搜索路径都是一些可以找到可执行程序的目录列表。当我们输入一个指令时，shell会先检查命令是否是内部命令，不是的话会再检查这个命令是否是一个应用程序。然后shell会试着从搜索路径，即PATH中寻找这些应用程序。如果shell在这些路径目录里没有找到可执行文件。则会报错。若找到，shell内部命令或应用程序将被分解为系统调用并传给Linux内核。

* SHELL  指当前用户用的是哪种shell


## 设置环境变量常用的几个指令
* echo - 查看显示环境变量，变量使用时要加上符号“$”
    `echo $PATH`
* export - 设置新的环境变量
 `export PATH=/usr/local/bin:$PATH`
* 修改环境变量 - 修改环境变量没有指令，可以直接使用环境变量名进行修改。
 `MYNAME="Prin"`
* env  查看所有环境变量
* set  查看本地定义的所有shell变量
* unset 删除一个环境变量
* readonly 设置只读环境变量
 `readonly MYNAME`

### 变量的引用
```bash
$variable
${variable}
"$variable"
"${variable}"
```
### 命令替换：$(command)
命令替换的目的是获取命令的输出、为变量赋值，或对命令的输出做进一步的处理。

# 进程相关技巧
```bash
ps -A | grep name
```
https://www.runoob.com/linux/linux-comm-grep.html
grep 命令用于查找文件里符合条件的字符串。上条命令可用于查找包含特定字符串的进程

如果出现进程未响应的状况，则查找到进程的PID后，可调用`kill -9 PID`强制终止进程

# 压缩解压
## 概览
.tar
解包：tar xvf FileName.tar
打包：tar cvf FileName.tar DirName
（注：tar是打包，不是压缩！）

.gz
解压1：gunzip FileName.gz
解压2：gzip -d FileName.gz
压缩：gzip FileName
.tar.gz 和 .tgz
解压：tar zxvf FileName.tar.gz
压缩：tar zcvf FileName.tar.gz DirName

.bz2
解压1：bzip2 -d FileName.bz2
解压2：bunzip2 FileName.bz2
压缩： bzip2 -z FileName
.tar.bz2
解压：tar jxvf FileName.tar.bz2
压缩：tar jcvf FileName.tar.bz2 DirName

.bz
解压1：bzip2 -d FileName.bz
解压2：bunzip2 FileName.bz
压缩： 未知
.tar.bz
解压：tar jxvf FileName.tar.bz
压缩： 未知

.Z
解压：uncompress FileName.Z
压缩：compress FileName
.tar.Z
解压：tar Zxvf FileName.tar.Z
压缩：tar Zcvf FileName.tar.Z DirName

.zip
解压：unzip FileName.zip
压缩：zip FileName.zip DirName

.rar
解压：rar x FileName.rar
压缩：rar a FileName.rar DirName


## zip / unzip 命令

unzip指令 格式如下：

//待更


-d 目录名	将压缩文件解压到指定目录下

# 拓展延伸 Console, TTY等
https://www.zhihu.com/question/26860370

## ref
https://www.iteye.com/blog/henry-cong-1060014
https://www.jianshu.com/p/09f521c60c30
https://blog.csdn.net/Axela30W/article/details/78981749
https://blog.csdn.net/huayangshiboqi/article/details/80150842
