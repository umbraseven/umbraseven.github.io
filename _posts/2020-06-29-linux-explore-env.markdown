---
layout: post
title:  "Linux探索：环境、工具、软件"
date:   2020-06-29 12:00:00 +0800
categories: Linux
tags: Linux 
description: 
---

*  目录
{:toc}

***

## 软件包管理

参考文章：
[rpm和yum](https://www.cnblogs.com/redirect/p/7797309.html)
[yum源的配置和使用](https://www.cnblogs.com/mchina/archive/2013/01/04/2842275.html)

***
### yum和rpm关系


RPM是redhat package manager，是redhat提供的软件包管理工具，类似于windows下的添加/删除应用程序，它关注的是软件包的安装和卸载，它的命令是rpm，安装文件是.rpm，具体用法可以rpm —help查看。


而yum呢？yellow dog updater modified。它是基于rpm机制开发的一种包管理软件，它是民间组织完成的，它的焦点是解决包依赖问题。

所以，rpm是本地的，是计算机自身的，是面向单个软件的。

而yum呢，则是为了解决软件复杂的依赖关系而生的，且核心是好的仓库，通过一个中央仓库来维护各种软件之间的依赖关系。

yum的底层毫无疑问需要调用rpm来完成软件包的安装与卸载，它的大概工作是，从中央仓库里查询包，获取依赖关系，下载包，完成本地安装。

这种一键下载安装，甚至可以一键搞定其依赖的方式实在太好用了，所以centos预装了yum。

***
### rpm

rpm：Rpm package manager。软件包管理工具。

一个软件包包含什么？压缩文件（archive files，比如tar.gz）和元数据（meta-data）。元数据描述了如何使用这个压缩文件，比如一些执行脚本、描述之类的。包可以是二进制形式，也有是源码形式。

基本模式：query，verify，install/upgrade/freshen/reinstall，uninstall，set，show……

其它的都挺常规的……

***
### yum的配置和使用


最重要的是，自带的文档非常非常好，yum —help只有基本使用，man yum就非常详细和完整了。要养成看文档的好习惯。

linux里，安装包管理是一件大事。debian-ubuntu系列用的是mkpg，比如常用到的apg-get命令。RedHat和Centos里呢，则是rpm机制，典型命令就是yum了。

yum的基本运行机制是怎么样的？记录各种软件的依赖关系，不满足就不安装。如果要安装呢？则可以直接从配置的仓库里下载。

所以yum的配置有三部分：
* /etc/yum.conf 主配置文件，在/etc主目录中，核心配置项都在这里。
* /etc/yum/  有多个文件、文件夹。
* /etc/yum.repo.d/，关于仓库部分的配置。

因为国内糟糕的外网访问速度，所以可以配置国内的镜像源。典型的有阿里云、网易、清华等源。



配置源是一件头疼的事。

首先，yum是根据配置文件来选择源的，所以要改配置，是.repo文件。这种文件有一大堆，但是其中每一个的项都不一样。其中base是必须的，里面会有[base][updates][extras][centosplus]等。

yum应该是会读所有的repo文件，每一个项目读取一个配置（重复的会忽略还是会覆盖？不太确定，但确实有插件，能配置多个源，自动选择速度最快的，应该就是加了个壳，在选择之前测试一下网速）。

一个项有什么内容？典型的：
```
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
#        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
#        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```
其中，name不重要；baseurl最关键，是主链接；mirrorlist不是必须项，如果不能访问最好注释掉（比如外网无法访问源内网链接）；gpg是一种流行的加密算法，check表示使能，key为密钥。

在其中，有$releasever，这是指定版本，有时候解析会失败，可以考虑直接手动替换为版本号，比如7。

$basearch呢？会根据你想找的自动替换，第一级就是系统架构，比如i386或者x86_64。centos7只提供64位源！压根没有32位！简直坑爹！

这些镜像文件如果想下载，是可以直接通过浏览器查看和下载的，就是走http协议嘛。也有的源是ftp协议，比如上海交大的。


***
另外还有个media文件，其中有[c7-media]，这是本地源，文件里可能是这样：

```
[c7-media]
name=CentOS-$releasever - Media
baseurl=file:///media/CentOS/
        file:///mnt/cdrom/
        file:///media/cdrecorder/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-5


```
如果要启用，需要将enabled修改为1。

***
### yum和dnf

dnf是yum的下一代版本，centos8中，已经完全替换为dnf了，对yum的操作会重定位到dnf。

dnf的命令列表实在太长了：

alias, autoremove, check, check-update, clean, diplist, distro-sync, downgrade, group, help, history, info, install, list, makecache, mark, module, provides, reinstall, remove, repoinfo, repolist, repoquery, repository-packages, search, shell, swap, updateinfo, upgrade, upgrade-minimal, upgrade-to…


也许有必要，但我现在肯定没时间去看所有的细节。我暂时的需求呢？配置几个国内的镜像源，dnf不是可以配置多源嘛。还有就是掌握简单的软件包管理操作，比如安装、检查更新、卸载之类的。dnf的文档超级全，例子很多，用的时候可以细看。

配置阿里云的镜像源很简单，直接导入阿里云的.repo文件为aliyun-centos8.repo，并不需要覆盖原生的base文档，再用dnf makecache就ok了。

***
### Mirrors：Aliyun

简直就是无所不含！

地址：mirrors.aliyun.com，如果在浏览器中访问，会直接跳转到网页版本。

有以下大项目：

镜像、域名解析dns、网络授时ntp。

镜像里面，最重要的是系统镜像，其中不但包括了各种os iso的下载，还有它们的软件安装包管理，非常方便。另外还有容器、语言等，比如docker、maven之类的，总之林子很大，什么鸟都有。

为什么要提供dns？如果不选择阿里云的dns，默认的dns系统是指向哪里？是否被污染得很厉害？

网络授时呢？它所做什么的，凭什么可以独占一个大项？好像授时是网络中的一个重要课题，国内有国家授时中心，有中科院等高校的，有阿里的。

最好的是阿里的，为什么？应该是因为阿里有大量的外售主机，所以阿里对这方面的需求很高，所以搭建了很完善的基建。

***
## centos

参考文章：
[centos8安装图解](https://www.lagou.com/lgeduarticle/45882.html)
[老司机定制centos8](https://blog.51cto.com/sery/2441438)

### 版本

我应该选什么版本？ 

首先，阿里云的os镜像，centos7都只有64版本了，我竟然还装了个32版本，真是疯了。必须得重新弄一个上来踩坑。 

如网上介绍： 

>当我们下载CentOS 7 时会发现有几个版本可以选择，如下： 
1、CentOS-7-DVD版本：DVD是标准安装盘，一般下载这个就可以了。 
2、CentOS-7-NetInstall版本：网络安装镜像。 
3、CentOS-7-Everything版本：对完整版安装盘的软件进行补充，集成所有软件。 
4、CentOS-7-LiveGnome版本：GNOME桌面版。 
5、CentOS-7-KdeLive版本：KDE桌面版。 
6、CentOS-7.0-livecd版本：光盘上运行的系统，类拟于winpe 


我们不需要桌面，也不需要很多的东西，我们只需要简简单单的dvd版本就ok了，它内部都有啥？readme.txt里说了，它内部包含了所有可以通过installer拿到的package。 

但是这些软件包呢，并不是默认全部安装的，它是我们可以自行选择的。比如上次跟着流程走，我们就只安装了开发工具包。 

但是我们必须得装点什么，不然就是个裸机了！为什么我的centos8里啥都没有，wget和yum都没有？我想就是这个问题了，什么鬼简易安装，不靠谱。 

所以我应该是要重新折腾一次的，选择centos7和8的64版本，按自己需要的方式来安装。 

*** 
### 安装流程


首选，不要用简易安装！就走自定义！自定义的流程是怎么样的呢？你首先创建好一个虚拟机环境，也就是一个裸机，分配好cpu、内存、虚拟磁盘之类的，准备好虚拟硬件环境。

然后再用cd的形式来安装你想要的linux系统，.iso光盘文件就是一个cd文件。

需要总体可以分为两步：

第一步，按照自定义流程创建不带操作系统的硬件裸机。

第二步，通过修改虚拟机，“插入”cd，读取到iso文件，开始安装。


***
### centos安装配置

* 语言，直接英文吧。
* 时间，折腾一下，自己配置一个ntp吧，比如阿里云的（以前我应该会直接跳过去，尽可能不参与）。
* 网络，可以直接打开网络，修改主机名称之类的。
* 软件安装包。这是最重要的选项。
    * server：服务器
    * server with gui：服务器也是可以带gui的
    * Workstation：工作站，和服务器之间有定义上的区别。比如说个人工作站，就是个人用的高性能pc，拿来搞设计之类的，有gui，而且要求很高。
    * minimal：最小安装，只有最基本的系统，适合高手自己搭环境。
    * custom：自定义
    * virtualization host：虚拟主机，要在这个主机上跑虚拟化服务


问题来了，我应该怎么选？server，然后自己添加想要的工具，比如development tools？工具们看起来都很心动啊。。还是全部都装上，不管三七二十一都探索一番？

选择了custom，然后选了三个工具包：”standard””development tools””system tools”。

***
### 网络连接

参考文章：

[centos8配置静态ip](https://www.cnblogs.com/qianyuliang/p/11591970.html)


***
如果要使用桥接bridge模式，那需要自己配置网卡，将ip修改为静态模式，同时配置ipaddr/gateway/dns/netmask这几个关键参数，然后nmcli c reload并重启机器（要重启，否则不会立即生效）。

如果是NAT连接，用“nmcli c reload”，就ok了。

在centos8中，传统的service network restart已经不适用了，要尽量避免使用ifconfig，而改用功能更为强大的nmcli。

***
## Arch

### arm与intel

参考文章：
[arm与intel(]https://zhuanlan.zhihu.com/p/21266987)

intel和x86架构和arm的arm架构有着本质上的区别：

* x86是复杂指令集架构，cisc，也就是设计非常复杂的电路，使得它向外提供的单条指令可以做更多的事情，比如两个数相加，可能就包括了从内存读取，处理相加过程，放到寄存器……其实这是要分多个步骤完成的，每一个步骤可以看作是一条“微指令”。所以一条对外提供的指令可以看作是多条微指令共同完成的，这是在硬件上进行支持的。
* arm则是精简指令集架构，risc，尽量只提供最基本的操作，所以它提供的指令更类似于微指令本身，只做最基本的操作，接近于原子操作。
* 以吃饭为例，对于x86，一条“吃饭”的指令就搞定了，内部怎么处理全由硬件搞定，软件层不需要管；对于arm，则需要软件自己编写，先盛饭、拿筷子、扒饭、收拾……
* 有何影响？从硬件上来说，x86毫无疑问要复杂很多，arm则可以将电路做得尽可能简单；x86性能上领先，原因可能是硬件电路处理比软件编程写逻辑效率天然更高；arm功耗上领先，虽然x86跑得更快，但它浪费得也多，为了支持复杂的硬件指令，内部应该有异常复杂的电路处理，这会大大增加功耗。
* 可以看到两者的设计理念上是非常不同的：intel可能更强调所提供服务的完整、易用性，偏向于极繁；而arm，够用就好，能跑就行，极简。
* 两者的适用场景也不同，对于x86，如果不怕发热、不担心没电、追求性能，那它很适合，比如台式机、服务器；对于arm，如果需要充电、怕发热、性能上没那么敏感，那非常好，比如手机等移动设备。


还有一些花边：

* 除了这两大架构之外，还有一个mips，学院派，授权门槛低，典型的就是中国的龙芯，主要用在嵌入式领域，比如路由器；还有一个alpha，中国的很多超算就是基于这个架构。
* x86上，intel和amd有很多恩怨纠葛，专利也是互相持有。特别值得一提的是，在cpu迭代上，从32位到64位是一次非常重要的变革（比4g到5g应该重要多了）。intel搞了个不兼容32位的ia64，凉了；amd搞了个兼容32位的x86_64，成为了标准。arm则推出了一个非常简明优秀的64位架构，加上移动互联网的兴起，安卓系统的出现，得到了大力发展。
* arm的异构也很有意思，可以同时兼容低功耗核和高性能核，平时运行时用低功耗核，可能用顺序指令跑；玩游戏了就开启高性能核，乱序跑……
* 当arm在移动端一统天下，自然就成为了标准，其他厂家要做这一块，都得搞个尴尬的转换层，自然会有兼容问题、性能问题。这就是垄断和标准的力量。
* 技术从来都不是纯粹的技术。回头看intel和arm的各种抉择，只有结合市场、产品、当时不可预知的未来等等，才能给出事后的评价……
* 谷歌的安卓可真的是帮了arm的大忙……
* Linux nb！


和我有什么关系？

我以后主要和什么架构打交道？目前来看应该是x86_64，服务器嘛。其实现在内核的开发和应用的开发，也有一种强烈的对比味道：应用的开发是趋于极繁的，有非常多、嵌套层次非常深的各种框架工具；而内核的开发则越发古朴拙重。

我对于架构相关代码还没有很强的感觉，对编译器其实感觉也不深。



将linux内核编译成arm架构和x86架构，汇编代码会有什么不一样？

arm的汇编代码果然完全不一样，精简指令集名不虚传。比如在x86中，mov指令可以在内存和寄存器之间各种使用，在arm中则不行，mov只能从寄存器到寄存器，ldr和str分别是从内存中载入到寄存器，以及从寄存器保存到内存。



在linux内核中，有哪些代码是体系架构相关的，必须放置到arch/下，这是如何划分的？或者说，为什么很多代码可以是架构通用的？这种架构之间的兼容是如何实现的？和编译器有相关吗？


***
## 帮助与文档

### man文档

刚读了systemd的man文档，稍作总结。

它有哪些项？

* NAME：基本定义。
* SYNOPSIS：命令格式。
* DESCRIPTION：简单描述，它是什么，基本功能等等。
* OPTIONS：参数选项，典型的有—name 和 —name==， 前者为boolean类型（默认一般为false），后者则可以赋值。简写方式为单字母 -h 或者 -h xxx。完整版好记，缩写版容易输入。
* CONCEPTS：详细说明。
* DIRECTORIES：文件目录等的说明。
* SIGNALS：信号交互说明。
* ENVIRONMENT：环境变量使用说明。linux的参数理念：用户设置的为最高优先级，不设置则按序读取默认值。比如—log-level，环境变量中有默认值，但用户一般可以在启动时通过options参数覆写。
* KERNEL COMMAND LINE：命令行说明，往往和options相关。
* SOCKETS AND FIFOS：套接字和管道的说明。
* SEE ALSO：拓展，比如相关命令。
* NOTES：注意项，比如参考文档。


还可能有哪些项：
* EXAMPLES：命令实例。


为什么是这些项？

当我们在命令行输入一个命令时，最终到底是发生了什么？shell执行了某个程序，因此提供了某种服务，或者完成了某个功能。但是这些服务、功能的完成一般都是通过进程完成的。

是这样的吗？比如当我们打开一个file，是通过open系统调用，最终转到正在系统中跑的vfs、文件系统、设备驱动等等完成的。

对于yum这种呢？它一直在系统中运行吗？还是说每次输入一个yum命令，都是启动一个新的yum进程，跑完任务然后退出？（应该是要退出的，因为shell还在等着嘛，至于能不能启动多个我就不清楚了，大概是不行的）

那像systemd这种需要持续运行，管理服务的呢？当我们运行systemctl时，是启动一个进程，然后和systemd主系统之间通过某种方式交互，来促使它完成我们想要的操作吗？



我们怎么向使用者描述一个命令？或者说，作为一个使用者，如何通过阅读文档来弄懂一个命令？

它的定义、功能、介绍、起源、内容、原理、格式、使用方式、参数、文件信息、交互信息、环境信息……

好像大概也就这些了。


***
## 系统工具

### pkg-conf

官方定义是：

>a system for configuring build dependency information.

>pkgconf is a program which helps to configure compiler and linker flags for development libraries.This allows build systems to detect other dependencies and use them with the system toolchain.


实例：

pkgconf —cflags foo
pkgconf systemd —print-variables：打印模块systemd中的所有变量


文档中说，它主要是被系统开发工具所使用的，比如compiler和linker之类的。暂时还不太熟悉，之后应该要打交道的。



***
## systemd


参考文章：
[systemd(1) — Linux manual page](https://man7.org/linux/man-pages/man1/systemd.1.html)
[阮一峰-systemd入门教程](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
[Linux服务管理两种方式service和systemctl](https://www.cnblogs.com/shijingjing07/p/9301590.html)

 ### service

传统service。

官方文档的定义是：
>service - run a System V init script

脚本存放目录为/etc/init.d。在centos8里，其中只有两个文件，readme和functions。readme中进行了说明——

* 传统的初始化脚本已经被原生的systemd services files所替代。新的命令是systemctl，有以下典型用法：
* systemctl：列出来现在正在运行的所有服务已经其他单元（and other units）。
* systemctl list-unit-files：列出所有unit files，不管是否在执行。
* systemctl start/stop foobar.service 启动/停止某个服务
* 传统的service依然能用，但实际上是在启动时映射成了systemd所管理的一个unit。

functions是一个shell scripts，提供了service命令的重定位功能，截取其中一小段，感受一下：
```
syetemctl_redirect(){
	local s
	local prog=${1##*/}
	local command=$2
	local options=“”

	case “$command” in
	start)
		s=$”Starting $prog (via systemctl):”
	…
	
	action “$s” /bin/systemctl $options $command “$prog.service”
}

```

***
### systemd文档


**官方文档定义**

>systemd, init - system and service manager.

>systemd is a system and service manager for Linux operating systems.

***
**unit**

一共有11种。

service/socket/target/device/mount/timer/swap/automount/path/slice/scope

以及其他：special。

每一类的具体功能参考文档。

***
**依赖关系**

依赖关系有两类：requirement dependencies和ordering dependencies。比如A依赖于B，但并不要求B先于A启动，那么两者就可以并行启动。在依赖关系中，有require和confilct，正向依赖和负向依赖。


以unit为最小粒度进行管理。

每当它加入时，可以做各种判断，直至确认它可以和其它已经存在的jobs和谐共存，才将它加入进来。

***
**文件目录**

默认目录：
/usr/local/lib/systemd/system
/usr/lib/systemd/system

但会先读取用户自己设置的，就是以下命令返回的目录：
pkg-config systemd —variable=systemdsystemunitdir


***
**信号交互**

为什么信号有专门的一栏？信号是和进程异步通信的手段。systemd也是作为一个进程在运行的，且pid为1，涵盖了之前boot时的init进程，以及启动之后的service manager。

如果我们需要和它通信呢？我们是否需要和它通信？


如果我们需要启动一个服务，执行systemctl命令，底层执行路径是什么？系统调用？还是信号？


***
**环境变量**

如$SYSTEM_LOG_LEVEL

systemd将会从环境变量中读取log level，它会被用户指定参数 —log-level=所覆盖。

***
### systemd讨论：kernel、os、init、service……


**Kernel vs OS**

内核仅仅提供基础服务，仅仅提供运行service的基础，而不管实际业务。所以内核只负责执行到init为止，至于要怎么启动，内核不管，那需要内核之外的部分编写启动脚本。

所以在之前的实现中，启动过程很长（串行）、脚本编写非常复杂（判断各种依赖关系、环境）。


为了解决这个问题，引入了systemd，由它来启动守护进程（daemon），所以它的名字为system daemon。在centos8中，systemd就是pid为1的进程。


一个完整的操作系统运行时，需要提供什么？需要运行什么？

内核不管业务，它将自己初始化完成，然后就放空了。而操作系统呢？一个完整的操作系统，应该是要包含基于内核实现的各种service的。

所以在内核的启动过程里，我们只需要关注各个子系统的初始化，从bios到boot，从进程0到进程1。从进程1开始呢？操作系统还有很多很多需要做的事情，它需要实际地挂载文件系统，加载和初始化各种设备，开启各种服务、定时器……

这个过程是面向用户的——这个用户既指作为用户的人，也指作为用户的应用程序。操作系统本身是为了给用户使用的。从这个角度来看，内核作为操作系统的一部分，为这种需求的实现提供了运行基础。

***
**Service**

什么是服务？

我们是否可以将任何开机启动的程序，都看作是一种“服务”呢？服务的本质是什么？一个进程在运行，监视某些io端口，捕获到某种输入时进行某些处理并输出。

某些服务可能没有gui，直接通过socket监听网络端口。但似乎也不妨碍某些服务有gui，可以接受用户通过点击屏幕所产生的io输入。

所以，服务并不一定得是在我们看不见的地方，似乎是和我们日常所见到的进程不一样的、特别神秘的存在。服务本质上就是提供某种服务的进程嘛。

可能区别于服务和普通进程（比如ls）之处在于：服务进程往往都是持久的、循环监听的、不会主动退出的，所以需要人为开启和关闭。

类似于服务器-客户机模型，服务进程提供某种服务，等待客户的访问。而用户（程序or人）可以发起这种访问，获得服务。


***
**Unit**

操作系统中只需要启动服务吗？

显然不够。除了运行某些服务进程之外，我们还需要提前做很多事，比如加载文件系统和设备，这显然不同于“服务”，它改变的是系统中的某些状态，而不是作为一个执行流一直存在。

比如文件系统，可能我们需要反复调用mount命令（类似于一次性服务），将系统中的各个文件系统都挂载进来，不管是磁盘文件系统还是内核虚拟文件系统如/proc和/sys。

这些所有的需要“启动”和“管理”的，都可以被视为是某种“资源”，用“Unit”来统一称呼。

***
**system config files**

在systemd配置文件中，有很多种类型（11+），比如其中就有service、target、wants、mount等。

在mount中，除了有[unit]之外，还有[mount]，其中有属性what、where、type等属性，我们可以想象出最终它会如此被执行：

&gt;mount -t type what where

我们甚至可以想到在systemd的内核实现中，它定义了unit作为所有资源的基类，它有属性如description,documentation,requires,wants,conflicts,after,before,condition,assert…它定义了一个资源的通用属性，比如依赖关系。

而每一种特定的资源都通过内嵌unit的方式继承它，又有了自己的字段，比如对于service，有[service]，其中有字段type,execstart,execstop…我们通过其中定义的命令来启动。

而target、wants等unit文件呢，更多的是作为辅助的存在，使得我们可以整体处理，可以分层。


***
**init & Service**

通过execStart和execStop我们可以重新来看“service”，它是基于“命令exec”所实现的，面向“用户”的某种“服务”，它的基本视角是用户。

为了实现和管理这种服务，可能需要多个命令，比如启动、关闭、重启、重新载入参数等等。如果说单个的命令更多的是面向过程的，那“服务”则更多地体现了面向对象的思想。


为什么init和service总是在一块？

如果将启动完成之后的操作系统视为某种环境，那这个环境是如何搭建起来的？正是在内核启动之后，从进程0到进程1之后，按照某种依赖关系，一点一点地加载系统中的各种资源，一个一个开启服务程序，最终才成为了“环境”。

为什么是由systemd（pid 1）来管理服务？因为搭建环境的过程就是加载资源、启动服务的过程，不由它管理由谁来管理呢？

***
**OS & Environment**

对于“服务”或者“资源”，不就是进程和各种文件设备吗，我自己操作不行吗？

当然可以。但是，之所以抽象为“服务”和“资源”，之所以要用systemd来整体管理，我们的目的是什么？是希望不需要基于内核来一个一个跑我们需要的程序，我们希望已经有某种面向用户的成熟的“环境”，这个环境已经可以向我们提供各种“服务”，有各种“资源”可以使用了。

也就是说，正是因为用户不希望直接和内核交互，而希望和某种更抽象的“环境”的交互，这个环境就是操作系统所提供的，而操作系统管理这个环境的核心就是systemd。

所以啊，不妨以抽象层的角度来看内核、操作系统、systemd、服务、用户程序……


***
### 守护进程

粗略了解了一下守护进程，发生之前的理解有很多偏差。

daemon定义：在后台长期运行的进程。

它的启动过程和普通的终端进程是不一样的，比如在父子关系、控制终端、根目录、会话等方面都有特别的处理。

但是这些都只是为了服务的实现：能让服务进程在后台长期运行。


***
### systemd中的服务管理


**其一是开机启动**

目录1:	/etc/systemd/system：其中为符号链接，开机启动项
目录2:	/usr/lib/systemd/system：系统中存在的各种服务本体。

systemctl enable httpd：建立符号链接，加入到目录1中，开机启动。


***
**其二为手动操作**

systemctl start/stop…

看文档操作吧。配置文件其实很简单，能看懂配置文件是理解这一套的基础。


***
### Systemd 架构图

![10a585d04d7170b6e3a5bbec674e2075.png](evernotecid://DB3D5FB5-8235-49A0-BD47-308B3511E2EB/appyinxiangcom/21050468/ENResource/p3947)

（来源网络）

***
## 远程连接

### ssh

ssh，远程登录程序，允许客户机通过远程登录服务机的shell。


**连接**

>ssh ssh://winn@192.168.0.144:22 

输入密码就可以登录了。

***
### scp和sftp

远程复制文件，底层使用ssh。

我用scp的时候身份验证老是失败，用sftp没有问题。


比如

>sftp root@192.168.0.144
mypassword
put test.file

在sftp连接后，默认是操作连接成功的服务机，如果要操作本地客户机，可以在指令前加上l，比如lls，lcd等。

用sftp向虚拟机传文件很方便。

***
### wget

非交互式网络下载，可以基于http、https、ftp等基本传输协议。

基本格式：wget url，非常简单。

但其实它可以设置大量的参数，具体可以查看man文档。这里列一些比较抓我眼球的：

* —input-file=file：通过file给出urls，一行一个，依次抓取。
* —limit-rate=amount：可以限速
* —user=user —password=password：在需要的时候可以携带登录信息

还有很多很多，因为会涉及到http参数设置，所以超级多……

***
## 网络管理

### nmcli

网络管理命令行工具，“创建、显示、编辑、删除、激活、停止”网络连接。

创建一个网络连接意味着什么？比如通过wifi的方式连接到路由器，应该是基于ip层的？需要输入的参数是ip地址、子网掩码、网关地址、dns这些。ip层往下是链路层和物理层，可以有各种形式。对于一台计算机以及其上的程序而言，“有网”就是说可以通过路由器连接到整个网络了，也就是ip层被建立起来了。


典型命令：

* general commands：综合命令，如nmcli general status/hostname/permissions/logging
* networking control commands：如nmcli networking on/off/connectivity
* connection management commands：如nmcli con show/up/down/modify/add/edit/clone/delete/load/reload…
* device management commands：如nmcli device status/show/set/connect/disconnect…


我的目标：基于同一个网卡设置两种连接：dhcp和static，可以随时切换。

几经周折，终于搞定了。因为家里有好几个wifi，桥接模式下稍微不注意ip就对不上导致ping不通，纠结了很久：

* 日常使用的较多的，应该是nat+dhcp的方式。就是在虚拟机里设置为nat，在con里设置为dhcp，这样切换wifi的时候就能自动联网了。比如网卡名字为ens33，默认连接名字也为ens33，会在/etc/sysconfig/network-scripts/文件夹下，生成ifcfg-ens33配置文件。
* 但是如果想在宿主机和虚拟机之间用ssh/sftp之类的，那就要用桥接。所以可以新建一个ens44：nmcli con clone ens33 ens44；找到ifcfg-ens44配置文件，将bootproto改为static，然后添加ipaddr/netmast/gateway/dns/prefix等字段。
* 在需要的时候（比如在家中某种特定wifi，其ip为192.168.0.xxx），切换到桥接，用nmcli con up ens44，手动从ens33切换至ens44……

nmcli功能很强大，我了解得还仅仅是我目前需要用到的极小的一部分，等待探索。

***
### dns

dns负责从域名变为ip地址。这一步必不可少，所以dns污染、dns劫持也是多得很。

我们可以指定dns地址，这个不受运营商限制。默认的方式下，都是从路由器那里动态获取。比如你买的是联通的路由器，那自然就是从联通拿。这样的话，似乎内部cache会访问得很快。

所以在家里的话，最优方式应该是更改路由器的dns指向，每一台设备都是动态获取。


但是我们当然要选择倒腾。

比如mac，可以在network，高级设置里，设置dns服务器，我选择了阿里云的223.5.5.5，貌似还有通用的114.114.114.114和8.8.8.8，但都是国外的。


如何查看本机的dns指向？

nslookup aliyun.com

最前面的server，就是dns服务器地址了。

***



