---
layout: post
title: Linux常用命令
date: 2016-8-29
categories: blog
tags: [linux]
description: 
---


**终端快捷键**     

```
Tab 自动补全
Ctrl+a 光标移动到开始位置
Ctrl+e 光标移动到最末尾
Shift+Ctrl+C 复制
Shift+Ctrl+V 粘贴
Ctrl+l 相当于clear，即清屏
Ctrl+p 向上显示缓存命令
Ctrl+n 向下显示缓存命令
Ctrl+d 关闭终端
```

**终端翻页**

```
Shift-pageup
Shift-pagedown
```

**man**

看手册(叫做manual或man page)。每一个命令和系统函数都有自己的man page。

```
man man
man read 查看read命令的man page
man 2 read 查看read系统函数的man page(在第二个section中，表示为read(2))
man -k read 以read为关键字查找相关的man page
```



#### 目录和文件

类Unix系统目录结构
ubuntu没有盘符这个概念，只有一个根目录/，所有文件都在它下面

```
/ 根目录
bin //系统可执行程序，如命令
boot //内核和启动程序，所有和启动相关的文件都保存在这里
grub //引导器相关文件
dev //设备文件
etc //系统软件的启动和配置文件，系统在启动过程中需要读取的文件都在这个目录。如LILO参数、用
户账户和密码。
home //用户的主目录。下面是自己定义的用户名的文件夹
lib //系统程序库文件,这个目录里存放着系统最基本的动态链接共享库，类似于Windows下的system32
目录，几乎所有的应用程序都需要用到这些共享库。
media //挂载媒体设备，如光驱、U盘等
mnt //目录是让用户临时挂载别的文件系统，如挂载Windows下的某个分区，ubuntu默认还是挂载在/media
目录。
opt //可选的应用软件包（很少使用）
proc //这个目录是系统内存的映射，我们可以直接访问这个目录来获取系统信息。也就是说，这个目录
的内容不在硬盘上而是在内存里。
sbin //管理员系统程序
selinux
srv
sys //udev用到的设备目录树，/sys反映你机器当前所接的设备
tmp //临时文件夹
usr //这是个最庞大的目录，我们要用到的很多应用程序和文件几乎都存放在这个目录下。]
bin // 应用程序
game //游戏程序
include
lib //应用程序的库文件
lib64
local //包含用户程序等
sbin //管理员应用程序
```

#### 文件属性和用户用户组           

```
whoami
查看当前登陆用户

chmod

* 文字设定法
chmod [who] [+|-|=] [mode] 文件名
操作对象who可是下述字母中的任一个或者它们的组合：
u 表示“用户（user）”，即文件或目录的所有者。
g 表示“同组（group）用户”，即与文件属主有相同组ID的所有用户。
o 表示“其他（others）用户”。
a 表示“所有（all）用户”。它是系统默认值。

操作符号可以是：
+ 添加某个权限。
- 取消某个权限。
= 赋予给定权限并取消其他所有权限（如果有的话）。

设置mode所表示的权限可用下述字母的任意组合：
r 可读。
w 可写。
x 可执行。

数字设定法

chmod [mode] 文件名
我们必须首先了解用数字表示的属性的含义：
0表示没有权限，
1表示可执行权限，
2表示可写权限，
4表示可读权限，

然后将其相加。所以数字属性的格式应为3个从0到7的八进制数，其顺序是（u）（g）
（o）。
例如，如果想让某个文件的属主有“读/写”二种权限，需要把4（可读）+2（可写）＝
6（读/写）。


chown

chown [OPTION]… [OWNER:GROUP] FILE…
chown [OPTION]… –reference=RFILE FILE…

更改某个文件或目录的属主和属组。这个命令也很常用。例如root用户把自己的一个文
件拷贝给用户A, 为了让用户A能够存取这个文件，root用户应该把这个文件的属主设为A，
否则，用户A无法存取这个文件。

OPTION的主要参数：

* -R 递归式地改变指定目录及其下的所有子目录和文件的拥有者。
* -v 显示chown命令所做的工作。
比如把一个文件改为itcast用户和nogroup用户组所有
$ sudo chown itcast:nogroup file1
注意

chown需要特权用户才能执行
* 一个文件的owner和owning group是没有关联的。一个文件属于用户A，也属于用户组
B，并不表示用户A属于用户组B。


chgrp

chgrp [OPTION]… GROUP FILE…
chgrp [OPTION]… –reference=RFILE FILE…
该命令改变（指定）指定文件所属的用户组。其中group可以是用户组ID，也可以是/
etc/group文件中用户组的组名。文件名是以空格分开的要改变属组的文件列表，支持通配
符。如果用户不是该文件的属主或超级用户，则不能改变该文件的组。
OPTION的主要参数：
-R 递归式地改变指定目录及其下的所有子目录和文件的属组。

```


#### 查找与检索            

**find**     

根据文件名查找                
find [OPTION] path… [expression]                       
在目录中搜索文件，path指定目录路径，系统从这里开始沿着目录树向下查找文件。它
是一个路径列表，相互用空格分离，如果不写path，那么默认为当前目录。Expression 是
find命令接受的表达式，find命令的所有操作都是针对表达式的。                    
一条最常用的find命令－－在当前目录及子目录下查找所有以file开头的文件名。   

```
$ find . -name 'file*'
$ find / -name 'vimrc'
$ find ~ -name '*.c'
```


**grep**

根据内容检索

```
grep [options] PATTERN [FILE...]
```

在指定文件中搜索特定的内容，并将含有这些内容的行输出到标准输出。若不指定文件
名，则从标准输入读取。

[options]部分包含的主要参数：       

``` 
-c：只输出匹配行的计数。        
-I：不区分大小写（只适用于单字符）。        
-h：查询多文件时不显示文件名。           

-l：查询多文件时只输出包含匹配字符的文件名。
-n：显示匹配行及行号。
-s：不显示不存在或无匹配文本的错误信息。
-v：显示不包含匹配文本的所有行。
-R: 连同子目录中所有文件一起查找。
```

比如到系统头文件目录下查找所有包含printf的文件

```
$ grep 'printf' /usr/include -R
```



#### 安装卸载软件          

**apt-get**

更新源服务器列表             
sudo vi /etc/apt/sources.list       

更新完服务器列表后需要更新下源


```
sudo apt-get update 更新源
sudo apt-get install package 安装包
sudo apt-get remove package 删除包
sudo apt-cache search package 搜索软件包
sudo apt-cache show package 获取包的相关信息，如说明、大小、版本等
sudo apt-get install package --reinstall 重新安装包
sudo apt-get -f install 修复安装
sudo apt-get remove package --purge 删除包，包括配置文件等
sudo apt-get build-dep package 安装相关的编译环境
sudo apt-get upgrade 更新已安装的包
sudo apt-get dist-upgrade 升级系统
sudo apt-cache depends package 了解使用该包依赖那些包
sudo apt-cache rdepends package 查看该包被哪些包依赖
sudo apt-get source package 下载该包的源代码
sudo apt-get clean && sudo apt-get autoclean 清理无用的包
sudo apt-get check 检查是否有损坏的依赖
```

**deb包安装*

```
安装deb软件包命令： sudo dpkg -i xxx.deb
删除软件包命令： sudo dpkg -r xxx.deb
连同配置文件一起删除命令： sudo dpkg -r --purge xxx.deb
查看软件包信息命令： sudo dpkg -info xxx.deb
查看文件拷贝详情命令： sudo dpkg -L xxx.deb
查看系统中已安装软件包信息命令： sudo dpkg -l
重新配置软件包命令： sudo dpkg-reconfigure xxx
```


**原码安装**      

```
1. 解压缩源代码包
2. cd dir
3. ./configure
检测文件是否缺失，创建Makefile,检测编译环境
4. make
编译源码，生成库和可执行程序
5. sudo make install
把库和可执行程序，安装到系统路径下
```

#### 压缩包管理      

**tar**        

tar [主选项+辅选项] 文件或者目录            
tar可以为文件和目录创建档案。利用tar命令用户可以为某一特定文件创建档案（备份
文件），也可以在档案中改变文件，或者向档案中加入新的文件。使用该命令时，主选项是
必须要有的，辅选项是辅助使用的，可以选用。    

主选项包括：

```
c 创建新的档案文件。如果用户想备份一个目录或是一些文件，就要选择这个选项。
r 把要存档的文件追加到档案文件的未尾。
t 列出档案文件的内容，查看已经备份了哪些文件。
u 更新文件。用新增的文件取代原备份文件，如果在备份文件中找不到要更新的文件，则把它追加到备份文件的最
后。
x 从档案文件中释放文件。（常用）
```

辅选项包括：

```
f 使用档案文件或设备，这个选项通常是必选的。（常用）
k 保存已经存在的文件。
m 在还原文件时，把所有文件的修改时间设定为现在。
M 创建多卷的档案文件，以便在几个磁盘中存放。
v 详细报告tar处理的文件信息。如无此选项，tar不报告文件信息。（常用）
w 每一步都要求确认。
z 用gzip来压缩/解压缩文件，加上该选项后可以将档案文件进行压缩，但还原时也一定要使用该选项进行解压
缩。（常用）
j 用bzip2来压缩/解压缩文件，加上该选项后可以将档案文件进行压缩，但还原时也一定要使用该选项进行解压
缩。（常用）
```

要将文件备份到一个特定的设备，只需把设备名作为备份文件名。

打包：

```
tar cvf dir.tar dir
tar xvf dir.tar dir
```

打gz压缩包：

```
tar zcvf dir.tar.gz dir
tar zxvf dir.tar.gz
```

打bz2压缩包:

```
tar jcvf dir.tar.bz2 dir
tar jxvf dir.tar.bz2
```

指定目录解压缩：

```
tar zxvf dir.tar.gz -C ~/test
```

**rar**

```
打包：把dir压缩成newdir.rar
rar a -r newdir dir
解包：把newdir.rar解压缩到当前目录
unrar x newdir.rar
```


**zip**

```
打包：
zip -r dir.zip dir
解包：
unzip dir.zip
```


#### 进程管理      

**who**

查看当前在线上的用户情况。所有的选项都是可选的，不使用任何选项时，who命令将
显示以下三项内容：         

- login name：登录用户名；
- terminal line：使用终端设备；
- login time：登录到系统的时间。   

**ps**

ps [选项]

ps命令用于监控后台进程的工作情况，因为后台进程是不和屏幕键盘这些标准输入/输
出设备进行通信的，所以如果需要检测其情况，便可以使用ps命令了。选项部分如下：


```
-e 显示所有进程。
-f 全格式。
-h 不显示标题。
-l 长格式。
-w 宽输出。
a 显示终端上的所有进程，包括其他用户的进程。
r 只显示正在运行的进程。
x 显示没有控制终端的进程。-w 宽输出。
```


这个命令参数有很多，但一般的用户只需掌握一些最常用的命令参数就可以了。最常
用的三个参数是u、a、x， 我们首先以root身份登录系统，查看当前进程状况


```
xingwenpeng@ubuntu:~$ ps aux
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
root 1 0.0 0.0 3672 2008 ? Ss 08:46 0:01 /sbin/init
xingwenpeng@ubuntu:~$ ps ajx
PPID PID PGID SID TTY TPGID STAT UID TIME COMMAND
4592 6948 6948 4592 pts/3 6948 R+ 1000 0:00 ps ajx
xingwenpeng@ubuntu:~$ ps -Lf 2423
UID PID PPID LWP C NLWP STIME TTY STAT TIME CMD
1000 2423 2282 2423 0 4 08:46 ? Ssl 0:00 gnome-session --session=ubuntu
1000 2423 2282 2465 0 4 08:46 ? Ssl 0:00 gnome-session --session=ubuntu
1000 2423 2282 2466 0 4 08:46 ? Ssl 0:00 gnome-session --session=ubuntu
1000 2423 2282 2468 0 4 08:46 ? Ssl 0:00 gnome-session --session=ubuntu
```

Head标头：

```
USER 用户名
UID 用户ID（User ID）
PID 进程ID（Process ID）
PPID 父进程的进程ID（Parent Process id）
SID 会话ID（Session id）
%CPU 进程的cpu占用率
%MEM 进程的内存占用率
VSZ 进程所使用的虚存的大小（Virtual Size）
RSS 进程使用的驻留集大小或者是实际内存的大小，Kbytes字节。
TTY 与进程关联的终端（tty）
STAT 进程的状态：进程状态使用字符表示的（STAT的状态码）
R 运行Runnable (on run queue) 正在运行或在运行队列中等待。
S 睡眠Sleeping 休眠中, 受阻, 在等待某个条件的形成或接受到信号。
I 空闲Idle
Z 僵死Zombie（a defunct process) 进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调
用后释放。
D 不可中断Uninterruptible sleep (ususally IO) 收到信号不唤醒和不可运行, 进程必须等待直到有中
断发生。
T 停止Terminate 进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行。
P 等待交换页
W 无驻留页has no resident pages 没有足够的记忆体分页可分配。
X 死掉的进程

< 高优先级进程高优先序的进程
N 低优先级进程低优先序的进程
L 内存锁页Lock 有记忆体分页分配并缩在记忆体内
s 进程的领导者（在它之下有子进程）；
l 多进程的（使用CLONE_THREAD, 类似NPTL pthreads）
+ 位于后台的进程组
START 进程启动时间和日期
TIME 进程使用的总cpu时间
COMMAND 正在执行的命令行命令
NI 优先级(Nice)
PRI 进程优先级编号(Priority)
WCHAN 进程正在睡眠的内核函数名称；该函数的名称是从/root/system.map文件中获得的。
FLAGS 与进程相关的数字标识
```

**jobs**        

用来显示当前shell 下正在运行哪些作业（即后台作业）。

**fg**  

fg [job…]

把指定的后台作业或挂起作业移到前台运行。参数job是一个或多个进程的PID，或者
是命令名称，或者是作业号（作业号前面要带一个%号）。

通常在shell中输入命令启动进程后，如果该进程需要与用户交互，那么此后用户的键
盘输入都被该进程读取，直到该进程退出后才出现shell提示符$，这种进程为前台进程。

如果在命令行的末尾加上&字符，则shell为这个命令创建一个后台进程，它虽然也可以
输出到屏幕，但是不能读取键盘输入，不管执行命令的进程有没有退出都立刻回到shell提
示符接受下一条命令的输入。如果该进程也需要读取键盘输入，则被挂起等待直到用户用fg
命令把它变成前台进程。如果一个命令需要较长的处理时间并且不需要与用户交互，就适合
把它放在后台执行。


**bg**

bg [job…]

把被挂起的进程提到后台执行。其中，job是一个或多个进程的PID、命令名称或者作
业号，在参数前要带%号。

所以如果通过ssh连接服务器，开启一个进程，想要退出后继续执行，可以把ctr+z后，把进程提到后台执行。

**kill**

```
向指定进程发送信号
kill [ -signal | -s signal ] pid ...
查看信号编号
kill -l [ signal ]
给一个进程发信号，或终止一个进程的运行
```


```
$ cat
（按Ctrl-z挂起当前进程）
[1]+ Stopped cat
$ ps
PID TTY TIME CMD
5819 pts/1 00:00:00 bash
5893 pts/1 00:00:00 cat
5894 pts/1 00:00:00 ps
$ kill -SIGKILL 5893
$（再次按回车键）
[1]+ Killed cat
$
kill
```

kill命令如果不带参数而直接跟pid，就是发给该进程SIGTERM信号，大部分进程收到该
信号就会终止。但是被挂起的进程不能处理信号，所以必须发SIGKILL信号，由系统强制终
止进程。


**env**

```
查看当前进程环境变量
$env
* vim /.bashrc
配置当前用户环境变量
* vim /etc/profile
配置系统环境变量,配置时需要有root权限
```


#### 用户管理          

**创建用户**    

```
sudo useradd -s /bin/bash -g xingwenpeng -d /home/xingwenpeng -m xingwenpeng
sudo useradd -s /bin/sh -g group -G adm,root xwp
```

此命令新建了一个用户xwp，该用户的登录Shell是/bin/sh，他属于group用户组，同时
又属于adm和root用户组，其中group用户组是其主组。

```
-s 指定新用户登陆时shell类型
-g 指定所属组，该组必须已经存在
-G 指定附属组，该组必须已经存在
-d 用户家目录
-m 用户家目录不存在时，自动创建该目录
```

**设置用户组**

sudo groupadd xingwenpeng

**设置密码**

sudo passwd xingwenpeng

**切换用户**


su 用户名



**root用户**

变成root用户            
sudo su             

设置root密码              
passwd         


**删除用户**

```
userdel 选项用户名
常用的选项是-r，他的作用是把用户的主目录一起删除。例如：
sudo userdel -r xingwenpeng
此命令删除用户xingwenpeng在系统文件（主要是/etc/passwd，/etc/shadow，/etc/
group等）中的记录，同时删除用户的主目录。
```


#### 网络管理 


**ifconfig**

```
1.查看网卡信息
ifconfig
2.关闭网卡
sudo ifconfig eth0 down
3.开启网卡eth0
sudo ifconfig eth0 up
4.给eth0配置临时IP
sudo ifconfig eth0 IP
```

**ping**

ping [选项] 主机名/IP地址

查看网络上的主机是否在工作。它向该主机发送ICMP ECHO_REQUEST包。有时我们想从
网络上的某台主机上下载文件，可是又不知道那台主机是否开着，就需要使用ping命令查
看。

命令中各选项的含义如下：

```
-c 数目在发送指定数目的包后停止。
-d 设定SO_DEBUG的选项。
-f 大量且快速地送网络封包给一台机器，看它的回应。
-I 秒数设定间隔几秒送一个网络封包给一台机器，预设值是一秒送一次。
-l 次数在指定次数内，以最快的方式送封包数据到指定机器（只有超级用户可以使用此选项）。
-q 不显示任何传送封包的信息，只显示最后的结果。
-r 不经由网关而直接送封包到一台机器，通常是查看本机的网络接口是否有问题。
-s 字节数指定发送的数据字节数，预设值是56，加上8字节的ICMP头，一共是64ICMP数据字节。
```


**netstat**   

netstat [选项]

显示网络连接、路由表和网络接口信息，可以让用户得知目前都有哪些网络连接正在运
作。命令中各选项的含义如下：


```
-a 显示所有socket，包括正在监听的。
-c 每隔1秒就重新显示一遍，直到用户中断它。
-i 显示所有网络接口的信息，格式同“ifconfig -e”。
-n 以网络IP地址代替名称，显示出网络连接情形。
-r 显示核心路由表，格式同“route -e”。
-t 显示TCP协议的连接情况。
-u 显示UDP协议的连接情况。
-v 显示正在进行的工作。
```

**nslookup**

nslookup name


查询一台机器的IP地址和其对应的域名。它通常需要一台域名服务器来提供域名服务。
如果用户已经设置好域名服务器，就可以用这个命令查看不同主机的IP地址对应的域名。
不带参数使用nslookup命令时，出现提示符“>”，在后面输入要查询的IP地址或域名
并回车即可。如果要退出该命令，输入exit并回车即可。

```
xingwenpeng@ubuntu:~$ nslookup
> www.itcast.cn
Server: 127.0.0.1
Address: 127.0.0.1#53
Non-authoritative answer:
Name: www.itcast.cn
Address: 115.29.149.42
>
```


**finger**

　finger [-lmsp] user [user@host …] 查询用户的信息，通常会显示系统中某个用
户的用户名、主目录、停滞时间、登录时间、登录shell等信息。如果要查询远程机上的用
户信息，需要在用户名后面接“@主机名”，采用[用户名@主机名]的格式，不过要查询的网
络主机需要运行finger守护进程。命令中各选项的含义如下：


-s 显示用户的注册名、实际姓名、终端名称、写状态、停滞时间、登录时间等信息。         
-l 除了用-s选项显示的信息外，还显示用户主目录、登录shell、邮件状态等信息，以
及用户主目录下的.plan、.project和.forward文件的内容。           
-p 除了不显示.plan文件和.project文件以外，与-l选项相同。       


```
xingwenpeng@ubuntu:~$ finger xingwenpeng
Login: xingwenpeng Name: xingwenpeng
Directory: /home/xingwenpeng Shell: /bin/bash
On since Mon Sep 8 08:55 (CST) on tty7 14 hours 48 minutes idle
On since Mon Sep 8 21:57 (CST) on pts/1 from :0
11 minutes 18 seconds idle
On since Mon Sep 8 23:12 (CST) on pts/2 from :0
6 seconds idle
No mail.
No Plan.
```


#### 常用服务器构建         

**ftp服务器** 

```
1.安装vsftpd服务器
sudo apt-get install vsftpd

2.配置vsftpd.conf文件
sudo vi /etc/vsftpd.conf
添加下面设置
anonymous_enable=YES
anon_root=/home/xinwenpeng/ftp
no_anon_password=YES
write_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES

3.重启服务器     
service vsftpd restart


```


**lftp客户端**          

lftp也是一种ftp客户程序。它是以文本方式操作的，但是比起图形界面更为方
便。lftp几乎具有bash的所有方便功能，Tab 补全，bookmark, queue, 后台下载等可以
得到支持。用法与ftp类似，主要的指令如下：

```
put 上传文件
mput 上传多个文件
get 下载文件
mget 下载多个文件
mirror 下载整个目录及其子目录
mirror –R 上传整个目录及其子目录
!command 调用本地shell执行命令command
```

**nfs** 

```
1.安装nfs服务器
sudo apt-get install nfs-kernel-server
2.设置/etc/exports配置文件
sudo vi /etc/exports
添加这行配置
/home/用户名/nfs *(rw,sync,no_root_squash)
3.在用户目录下创建nfs目录
mkdir /home/用户名/nfs
4.重启服务器，重新加载配置文件
sudo /etc/init.d/nfs-kernel-server restart
5.在/home/用户名/nfs目录下创建测试文件hello
cd /home/用户名/nfs
touch hello
6.测试服务器，把服务器共享目录nfs挂在到/mnt节点
sudo mount -t nfs -o nolock -o tcp IP:/home/用户名/nfs /mnt
7.进入/mnt目录可以看到hello文件，表示构建成功
8.卸载网络共享目录
sudo umount /mnt
```


**ssh**

```
1.安装ssh服务器
sudo apt-get install openssh-server
2.远程登陆
ssh 用户名@IP
```


### 其他  

**alias**

alias [-p] name=value …             
将value字符串起个别名叫name，以后在命令行输入name，shell自动将其解释为
value，如果不带参数执行本命令，或以参数-p执行，则显示当前定义的别名列表。

```
$ alias
alias ls='ls --color=auto'
alias rm='rm -i'
```

**echo**                

echo [-n] 字符串        

在显示器上显示一段文字，一般起到一个提示的作用。其中选项n表示输出文字后不换
行；字符串可以加引号，也可以不加引号。用echo命令输出加引号的字符串时，将字符串原
样输出；用echo命令输出不加引号的字符串时，将字符串中的各个单词作为字符串输出，各
字符串之间用一个空格分割。                        
查看上一个程序退出数值，正常情况程序退出值是0    


echo $?


**date**

查看当前时间

**shutdown**            

shutdown -t 秒数[-rkhncfF] 时间[警告讯息]


```
-t 秒数: 设定在切换至不同的runlevel之前, 警告和删除二讯号之间的延迟时间(秒).
-k : 仅送出警告讯息文字, 但不是真的要shutdown.
-r : shutdown 之後重新开机.
-h : shutdown 之後关机.
-n : 不经过init , 由shutdown 指令本身来做关机动作.(不建议你用)
-f : 重新开机时, 跳过fsck 指令, 不检查档案系统.
-F : 重新开机时, 强迫做fsck 检查.
-c : 将已经正在shutdown 的动作取消.
```

例子

```
shutdown -r now 立刻重新开机
shutdown -h now 立刻关机
shutdown -k now 'Hey! Go away! now....' 发出警告讯息, 但没有真的关机
shutdown -t3 -r now 立刻重新开机, 但在警告和删除processes 之间, 延迟3秒钟.
shutdown -h 10:42 'Hey! Go away!' 10:42 分关机
shutdown -r 10 'Hey! Go away!' 10 分钟後关机
shutdown -c 将刚才下的shutdown 指令取消,必须切换至其它tty, 登入之後, 才能下此一指令.
shutdown now 切换至单人操作模式(不加任何选项时)
```

注意事项:
时间参数务必要加: 不是用now, 便是用hh:mm 或mm
now 其实就是0 的意思.


**一些系统命令** 

```
查看内核版本信息
uname -a

查看发行版信息
lsb_release -a

查看空闲内存
free -m
```
