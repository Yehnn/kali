# Kali 实验环境介绍与使用

## 1. 课程说明

本课程为纯动手实验教程，为了能说清楚实验中的一些操作会加入理论内容，也会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

## 2. 学习方法

实验楼的 Kali 系列课程包含五个训练营，本训练营主要讲述服务器攻击方法。课程包含20个实验，每个实验都提供详细的步骤和截图。适用于有一定Linux系统基础，想快速上手 Kali 渗透测试的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触 Kali Linux 和渗透测试的概念。需要依次完成下面几项任务：

1. 理解 Kali Linux 是什么
2. 学习如何安装 Kali Linux
3. 了解 Metasploitable2 靶机
4. 学习实验楼提供的多机实验环境如何使用
5. 尝试由 Kali 向靶机进行简单的漏洞扫描

## 4. 推荐阅读

本节实验推荐阅读下述内容：

1. [Kali 使用入门](http://docs.kali.org/category/introduction)
2. [Metasploitable2 使用指南](https://community.rapid7.com/docs/DOC-1875)

## 5. Kali 简介

### 5.1 Kali Linux 是什么？

[Kali Linux](https://www.kali.org/) 是一个基于 Debian 的用来进行渗透测试和安全审计的 Linux 操作系统，最初在 2013年3月 由具备类似功能的 BackTrack 系统开发而来，由 Offensive Security 团队支持开发。为了支持安全审计和渗透测试的一系列功能，Kali Linux 进行了定制化，包括内核，网络服务和用户都进行了特殊的设置。

Kali 系统内预装了很多安全相关的工具，例如非常有名的 Nmap 端口扫描器、John the Ripper 密码破解器、Metasploit Framework 远程攻击框架等。实验楼的该训练营主要讲述如何利用 Kali Linux 的一系列工具对具备漏洞的靶机进行攻击，同时会介绍下攻击的原理。

Kali Linux 中目前最新版是 2016 年发布的 2.0-2016.2 版本，实验楼的实验环境中使用的就是这个版本的虚拟机。在其中包含了 600 多个渗透测试的工具，并继承了为安全审计定制的内核补丁。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374556255.png/wm)

*图片来自 Kali 官网*

### 5.2 渗透测试

渗透测试指的是在具备一定授权的情况下，对目标服务器或网络进行信息安全防护能力的评估的过程。包括但不限于采用各种扫描和攻击手段对目标的防护系统进行探测、攻击、破坏以及设置后门。渗透测试是信息安全评估的一部分，目的是为了保证目标系统的网络防御能力达到预期。目前有很多家公司和团队可以提纲渗透测试的专业服务，很多互联网公司都对这个领域由大量的人才需求。

### 5.3 漏洞分析

漏洞分析是通过渗透测试的相关扫描和探测工具对目标系统潜在的漏洞进行测试，评估目标系统是否已经完善了补丁或安全策略。如果目标系统仍然存在相应的漏洞，则要对漏洞的风险进行评估，依据发现和利用的难度等指标。

### 5.4 社会工程学

社会工程学（Social Engineering）应用到信息安全领域，指的是通过社会或人际之间的关系接触来获得一定的特权及可用于攻击的敏感信息。

举个例子，通过钓鱼网页的形式获得用户登录某网站的用户名和密码，甚至银行账户信息等。利用的就是在用户不知情的情况下采用伪装和欺骗的形式，很自然的获取必要信息。

## 6. Kali 安装部署

**注意： 实验楼中已经配置好了 Kali 虚拟机的实验环境，无需再次安装，下列步骤仅供本地环境搭建参考。** 

Kali Linux 已经被安装到了实验楼环境的虚拟机中，这里我们仅仅进行简单的安装步骤的介绍，供感兴趣的同学在本地环境中安装。

Kali Linux 安装有几种方法：

1. 下载官方 ISO 镜像，安装到物理机器或虚拟机中
2. 下载官方虚拟机镜像，在虚拟机管理软件中启动
3. 下载并部署 ARM 镜像到 ARM 系统中

任何方案都需要从官方网站下载镜像，下载之后务必校验 SHA1 值，避免下载到有安全漏洞的系统。

通常作为学习环境，我们都会安装到本地的虚拟机，实验楼中的环境采用的就是该方案，通过下载现成的虚拟机磁盘镜像避免了安装的复杂过程。如果希望使用官方的 ISO 镜像进行安装，可以从下面的两个链接地址获得帮助：

+ [Kali ISO 下载链接](https://www.kali.org/downloads/) 请注意区分版本，其中的 Light 版本只包含常用的工具，建议下载 Kali Linux 完全版本。
+ [Kali 安装方法](http://docs.kali.org/category/installation)


建议使用 VirtualBox 虚拟化管理软件，该软件在 Ubuntu 上可以通过下面的方式安装：

```
sudo apt-get update
sudo apt-get install virtualbox
```

安装完成后默认会自动配置好虚拟网络，下一步是在 Offensive Security 官网下载虚拟机镜像：

+ [Kali 虚拟机镜像下载链接](https://www.offensive-security.com/kali-linux-vmware-virtualbox-image-download/) 

注意需要根据你使用的虚拟化管理软件下载相应的虚拟机镜像，我们使用的是 VirtualBox，则需要下载下面的这个地址：

+ [Kali VirtualBox 镜像下载链接](https://images.offensive-security.com/virtual-images/Kali-Linux-2016.2-vbox-amd64.ova)

虚拟机镜像的版本有四个，推荐下载 64 位的 Kali Linux 64bit VBox，下载得到的是一个 OVA，直接使用 VirtualBox 的导入功能就可以加载到 VirtualBox 中，点击开始虚拟机就启动了。

启动后的 Kali 虚拟环境中默认进入桌面系统，默认的用户名是 root，密码是 toor。


## 7. 实验环境启动

### 7.1 实验环境介绍

实验楼采用的实验环境包含两台虚拟机，分别是攻击机和靶机：

- 攻击机：Kali Linux ，因环境中虚拟机太慢，这里采用 docker 容器。进入 kali linux 系统方式如下：

```bash
$ docker run -ti --network host 6f113 bash
```

进入 kali 容器过后，执行如下命令在 `/etc/hosts` 中追加内容，使我们在攻击靶机时不用每次输入 ip 地址，而是直接使用靶机的主机名。

```bash
$ sudo echo "192.168.122.102 target" >> /etc/hosts
```

- 靶机：Metasploitable2 虚拟机，主机名是 target，IP 地址为 192.168.122.102，默认用户密码为 msfadmin/msfadmin 。

### 7.2 虚拟机管理

上述的两台虚拟机已经被安装到实验楼的虚拟环境中，可以通过 Libvirt 系列命令进行管理和查看。

> Libvirt是一套免费、开源的支持Linux下主流虚拟化工具的C函数库，其旨在为包括Xen在内的各种虚拟化工具提供一套方便、可靠的编程接口，支持与C,C++,Ruby,Python等多种主流开发语言的绑定。当前主流Linux平台上默认的虚拟化管理工具virt-manager(图形化),virt-install（命令行模式）等均基于libvirt开发而成。

> Libvirt 库是一种实现 Linux 虚拟化功能的 Linux API，它支持各种虚拟机监控程序，包括 Xen 和 KVM，以及 QEMU 和用于其他操作系统的一些虚拟产品。

简单的说Libvirt是一套标准化虚拟化管理接口，可以管理上述我们提到的各种虚拟资源。Libvirt除了各种语言的SDK外，还提供一个命令行工具`virsh`。在我们的实验环境中可以通过下列方式来启动，请花些时间熟悉`virsh`命令，非常有助于我们后续开发过程中的调试。

```
# 启动libvirt-bin服务
sudo service libvirt-bin start
# 查看当前系统中的虚拟机列表，默认返回为空
sudo virsh list
# 查看当前系统中虚拟网络列表，默认返回default网络
sudo virsh net-list --all
```

`virsh`常用命令列表：

```
命令	           说明
help	        显示该命令的说明
quit	        结束 virsh，回到 Shell
connect	        连接到指定的虚拟机服务器
create	        启动一个新的虚拟机
destroy	        删除一个虚拟机
start	        开启（已定义的）非启动的虚拟机
define	        从 XML 定义一个虚拟机
undefine        取消定义的虚拟机
dumpxml	        转储虚拟机的设置值
list	        列出虚拟机
reboot	        重新启动虚拟机
save	        存储虚拟机的状态
restore	        回复虚拟机的状态
suspend	        暂停虚拟机的执行
resume	        继续执行该虚拟机
dump	        将虚拟机的内核转储到指定的文件，以便进行分析与排错
shutdown        关闭虚拟机
setmem	        修改内存的大小
setmaxmem       设置内存的最大值
setvcpus        修改虚拟处理器的数量
```


### 7.3 查看并启动实验环境

在实验桌面中，双机 Xfce 终端，打开终端，后续所有的操作命令都在这个终端中输入。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374588595.png/wm)

点击终端上的`文件按钮-》打开标签`新建一个标签，然后使用 `virsh list` 命令查看当前环境中虚拟机的列表和状态，注意需要使用 sudo，另外需要加上参数 `--all` 才会显示所有关机状态的虚拟机：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374596120.png/wm)

然后我们使用 `virsh start` 命令启动虚拟机，注意区分大小写，虚拟机的名字是大写的字母开始，再次查看状态虚拟机已经进入 running 状态：

```bash
$ sudo virsh start Metasploitable2
```

注意由于虚拟机启动需要时间，大概要等四分钟左右。执行如下命令，如果能 ping 通靶机，说明靶机已经启动起来了：

```bash
ping -c 3 target
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971533885010535-wm)

在上一个标签执行 `ping target` 测试 kali 能否 ping 通靶机，使用 Ctrl-C 退出 ping：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1533885576808.png-wm)

现在两台实验环境都已经启动了，我们可以开始渗透测试了。

## 8. Kali 包管理和软件安装

### 8.1 Kali APT 包管理

Kali 采用的包管理工具跟 Ubuntu 相同，但需要使用 Kali 自己特定的软件源。

> APT是Advance Packaging Tool（高级包装工具）的缩写，是Debian及其派生发行版的软件包管理器，APT可以自动下载，配置，安装二进制或者源代码格式的软件包，因此简化了Unix系统上管理软件的过程。APT最早被设计成dpkg的前端，用来处理deb格式的软件包。现在经过APT-RPM组织修改，APT已经可以安装在支持RPM的系统管理RPM包。这个包管理器包含以 `apt-` 开头的的多个工具，如 `apt-get` `apt-cache` `apt-cdrom` 等，在Debian系列的发行版中使用。

当你在执行安装操作时，首先`apt-get` 工具会在**本地**的一个数据库中搜索目标软件相关信息，并根据这些信息在相关的服务器上下载软件安装，这里大家可能会一个疑问：既然是在线安装软件，为啥会在本地的数据库中搜索？要解释这个问题就得提到几个名词了：

- **软件源镜像服务器** 
- **软件源** 

我们需要定期从服务器上下载一个软件包列表，使用 `sudo apt-get update` 命令来保持本地的软件包列表是最新的（有时你也需要手动执行这个操作，比如更换了软件源），而这个表里会有**软件依赖**信息的记录，对于软件依赖，我举个例子：我们安装 `w3m` 软件的时候，而这个软件需要 `libgc1c2` 这个软件包才能正常工作，这个时候 `apt-get` 在安装软件的时候会一并替我们安装了，以保证 `w3m` 能正常的工作。

由于 Kali 默认登陆的是 root，所以 sudo 可以去掉。

### 8.2 apt-get 工具

`apt-get`使用各用于处理`apt`包的公用程序集，我们可以用它来在线安装、卸载和升级软件包等，下面列出一些`apt-get`包含的常用的一些工具：

| 工具           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| `install`      | 其后加上软件包名，用于安装一个软件包                         |
| `update`       | 从软件源镜像服务器上下载/更新用于更新本地软件源的软件包列表  |
| `upgrade`      | 升级本地可更新的全部软件包，但存在依赖问题时将不会升级，通常会在更新之前执行一次`update` |
| `dist-upgrade` | 解决依赖关系并升级(存在一定危险性)                           |
| `remove`       | 移除已安装的软件包，包括与被移除软件包有依赖关系的软件包，但不包含软件包的配置文件 |
| `autoremove`   | 移除之前被其他软件包依赖，但现在不再被使用的软件包           |
| `purge`        | 与remove相同，但会完全移除软件包，包含其配置文件             |
| `clean`        | 移除下载到本地的已经安装的软件包，默认保存在/var/cache/apt/archives/ |
| `autoclean`    | 移除已安装的软件的旧版本软件包                               |


下面是一些`apt-get`常用的参数：

| 参数                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| `-y`                 | 自动回应是否安装软件包的选项，在一些自动化安装脚本中使用这个参数将十分有用 |
| `-s`                 | 模拟安装                                                     |
| `-q`                 | 静默安装方式，指定多个`q`或者`-q=#`,#表示数字，用于设定静默级别，这在你不想要在安装软件包时屏幕输出过多时很有用 |
| `-f`                 | 修复损坏的依赖关系                                           |
| `-d`                 | 只下载不安装                                                 |
| `--reinstall`        | 重新安装已经安装但可能存在问题的软件包                       |
| `--install-suggests` | 同时安装APT给出的建议安装的软件包                            |


### 8.4 安装软件包

关于安装，如前面演示的一样你只需要执行`apt-get install <软件包名>`即可，除了这一点，你还应该掌握的是如何重新安装软件包。
很多时候我们需要重新安装一个软件包，比如你的系统被破坏，或者一些错误的配置导致软件无法正常工作。

你可以使用如下方式重新安装：

```
$ sudo apt-get --reinstall install w3m
```

另一个你需要掌握的是，如何在不知道软件包完整名的时候进行安装。通常我们是使用`Tab`键补全软件包名，后面会介绍更好的方法来搜索软件包。有时候你需要同时安装多个软件包，你还可以使用正则表达式匹配软件包名进行批量安装。

### 8.5 软件升级

```
# 更新软件源
$ sudo apt-get update
# 升级没有依赖问题的软件包
$ sudo apt-get upgrade
# 升级并解决依赖关系
$ sudo apt-get dist-upgrade
```


### 8.6 卸载软件

如果你现在觉得 `w3m` 这个软件不合自己的胃口，或者是找到了更好的，你需要卸载它，那么简单！同样是一个命令加回车 `sudo apt-get remove w3m` ，系统会有一个确认的操作，之后这个软件便“滚蛋了”。

或者，你可以执行

```
# 不保留配置文件的移除
$ sudo apt-get purge w3m
# 或者 sudo apt-get --purge remove
# 移除不再需要的被依赖的软件包
$ sudo apt-get autoremove
```

### 8.7 软件搜索

当自己刚知道了一个软件，想下载使用，需要确认软件仓库里面有没有，就需要用到搜索功能了，命令如下：

```
sudo apt-cache search softname1 softname2 softname3……
```

`apt-cache` 命令则是针对本地数据进行相关操作的工具，`search` 顾名思义在本地的数据库中寻找有关 `softname1` `softname2` …… 相关软件的信息。

关于在线安装的的内容我们就介绍这么多，想了解更多关于APT的内容，你可以参考：

- [APT HowTo](http://www.debian.org/doc/manuals/apt-howto/index.zh-cn.html#contents)

## 9. Metasploitable2 靶机介绍

### 9.1 Metasploitable2 简介

Metasploitable2 是实验楼训练营中使用的靶机系统，本身是一个定制的 Ubuntu 操作系统，用在渗透测试和漏洞演示的场景中。这个系统可以说是充满了常见的安全漏洞，我们后续的实验中都将使用 Kali Linux 对该系统进行渗透测试，发现漏洞并进行攻击。

Metasploitable2 以虚拟机的形式发布，实验楼中使用到的是目前可以下载到的最新版本 2.0.0，不过这个版本在 2012 年之后就没有再更新过。

由于实验楼环境可以快速秒级创建，所以不用担心在渗透测试过程中搞坏了靶机系统，点击停止实验然后再次开始实验就能够获得一个全新的实验环境。

**注意：**本节部分内容来自 [Metasploitable2官方文档](https://community.rapid7.com/docs/DOC-1875)。

### 9.2 Metasploitable2 安装

**注意：实验楼环境中已经部署好了 Metasploitable2 虚拟机，所以下列步骤仅供感兴趣的同学在本地搭建靶机系统。** 

首先，需要在以下链接下载最新版本的 Metasploitable2 虚拟机镜像：

+ [Metasploitable2 下载链接](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/)

可以使用 wget 下载，下载得到的是一个 zip 压缩包，需要使用下面的命令进行解压：

```
unzip metasploitable-linux-2.0.0.zip
```

解压后的文件是一个虚拟磁盘，格式为 VMDK。同样，使用 VMware 或 VirtualBox 软件创建一个新的虚拟机，在选择磁盘的时候选使用现有的磁盘，不要选择创建新磁盘。创建好的虚拟机，直接点击开机就可以了。

Metasploitable2 环境中的默认的用户名和密码都是 msfadmin，并且 SSH 服务默认打开，可以直接使用 SSH 登录到系统上。

### 9.3 登录 Metasploitable2

按照 7.3 的步骤，我们使用 SSH 登录到 Metasploitable2中，用户名 msfadmin，密码 msfadmin：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374727064.png/wm)

### 9.4 查看开放的服务

为了能够更大范围的测试各种服务的漏洞，当前的系统中开放了很多种服务，使用命令 `netstat -tln` 查看当前系统监听的端口：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374737464.png/wm)

从上面的列表中可以发现，当前系统启动了非常多的网络服务，例如 21 端口是 FTP 服务的监听端口，22 是 SSH 服务监听端口，这些服务都具备一定被攻击的风险，后续的若干个实验都会尝试在外部通过相应服务的漏洞攻入 Metasploitable2 系统中。

我们再使用 `rpcinfo` 命令查看当前系统开放的服务：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374747716.png/wm)

这个列表中分别列出了端口号和对应的服务。

### 9.5 弱口令

Metasploitable2 上的系统和数据账户都有非常严重的弱口令，至少包括下面的弱口令：

1. msfadmin:msfadmin
2. user:user
3. postgres:postgres
4. sys:batman
5. klog:123456789
6. service:service

### 9.6 后门服务

系统中内置了很多的潜在后门服务，例如 ftp 服务使用的 vsftpd 软件，具备一个可以作为后门的漏洞。这个版本的软件如果接受到的 ftp 用户名后面有笑脸符号，则会自动在 6200 端口上打开一个监听的 Shell。简单的测试方式是使用 telnet 尝试连接 vsftp 服务并发个笑脸 `:)` 符号获取 Shell。

这个简单的实验留作本节的课后作业供大家自己尝试。可以将尝试方法发到讨论去中讨论。

### 9.7 Web 服务

系统中预装了几个具备各种 Web 漏洞的 Web 服务，可以通过打开实验环境中的 Firefox 浏览器，输入 `http://target` 来查看：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374760210.png/wm)

这些 Web 应用都具备很多容易利用的 Web 漏洞，我们将在另外一个训练营中学习如何利用 Web 漏洞进行 Web 渗透。

## 10. 实验环境下的扫描操作

### 10.1 登录 Kali

首先使用 SSH 连接到 Kali，我们大部分的攻击操作都需要在 Kali 虚拟机中进行，注意用户名root，密码 toor 是不显示的，使用命令 `ssh root@kali` 即可，因为当前实验环境中已经把 IP 地址和主机名的对应写入到了 `/etc/hosts` 文件中，避免输入不好记的 IP 地址：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374769353.png/wm)

### 10.2 扫描目标主机开放的服务

我们可以在 Kali 使用 Nmap 工具扫描 Metasploitable2 中开放的服务，为了节省时间，我们只扫描1-1000号端口，注意 Nmap 的参数使用：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374777134.png/wm)

在后续的实验中，我们将学习 Nmap 的详细使用，这里仅仅简单介绍下如何在实验环境的两台虚拟机之间使用工具扫描。

## 11. 总结


本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. 理解 Kali Linux 是什么
2. 学习如何安装 Kali Linux
3. 了解 Metasploitable2 靶机
4. 学习实验楼提供的多机实验环境如何使用
5. 尝试由 Kali 向靶机进行简单的漏洞扫描

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

## 12. 作业

1. 在本地部署一台 Kali VirtualBox 虚拟机，启动后对本地操作系统进行扫描
2. 在实验楼环境中启动靶机，并完成 9.6 节中通过发笑脸符号给 FTP 服务获得 Shell 权限。


