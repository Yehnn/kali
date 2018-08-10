# 攻击 Ingreslock 后门漏洞

## 一、实验简介

### 1.1 实验介绍

本实验主要利用 Ingreslock  服务漏洞进行攻击。实验环境基于实验楼的 Kali 和所要渗透的目标靶机 Metasploitable2。

在上个实验中，大家学习了如何开发自己的 Metasploit 扫描器，在这一实验中，我们要介绍的内容，是攻击 Ingreslock 后门漏洞。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 实验知识点

实验通过对实验楼目标靶机 Metasploit 的渗透扫描，找到漏洞 `Ingreslock ` 服务，并通过在终端，输入相应的命令，进行渗透攻击，最终获得 Metasploitable2 的管理权限。

- Ingreslock 漏洞原理
- Nmap 渗透扫描基础
- 攻击 Metsploitable2 获取权限
- 验证是否攻击成功

### 1.3 实验环境

实验在实验楼的环境中进行，本实验由实验楼提供两台虚拟机，分别是攻击机和目标靶机。这两台虚拟机的主机名称，主机名，以及 IP 地址分别如下：

| 主机     | 主机名称        | 主机名 | IP 地址         |
| -------- | --------------- | ------ | --------------- |
| 攻击机   | Kali Linux 2.0  | Kali   | 192.168.122.101 |
| 目标靶机 | Metasploitable2 | target | 192.168.122.102 |


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480388470011.png-wm)



## 二、环境启动

### 2.1 实验环境启动

本实验的实验环境由实验楼提供，宿主机为 Ubuntu，宿主机上安装有两台虚拟机。它们分别为 Kali Linux 虚拟机和 Metasploitable2 目标靶机。本实验所有的操作，都是在 Kali Linux 下进行的。

首先我们在实验楼宿主机上启动 Kali Linux 和 Metasploitable2，输入如下命令启动攻击机 Kali 和目标靶机：

```
# 启动 Kali Linux 攻击机
sudo virsh start Kali

# 启动目标靶机 Metasploitable2
sudo virsh start Metasploitable2

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483505113969.png-wm)

使用 `ping` 命令可以知道 Kali Linux 是否已经完全启动成功，当宿主机 Ubuntn 和 Kali Linux 两台主机能够 ping 通的时候，说明 Kali 已经启动。此时输入命令使用 ssh 进行连接：

```
# 连接 Kali Linux，密码为 toor
ssh root@Kali
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483505661337.png-wm)

## 三、渗透扫描

### 3.1 漏洞原理

Ingreslock 服务安全漏洞原理：

> ingreslock 1524/TCP 端口是 Ingres 数据库管理系统（DBMS）锁定服务    

ingreslock 1524/TCP 端口经常被用作后门程序监听端口，这个端口曾经一段时间非常流行，是一个非常古老的漏洞。许多攻击脚本当通过其他服务的漏洞攻击成功后，可以通过这个端口安装一个后门 Shell。

**注意：这个漏洞太古老了，不过目前仍然有少数的机器上仍然开放着这个端口，可以通过简单的 Nmap 扫描就能够发现，这个端口最大的作用是提供后门监听端口又不容易被发现。**

### 3.2 漏洞扫描

在下面的实验中，我们依旧遵循着渗透攻击的流程。渗透测试的流程分别为：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480387936307.png-wm)


渗透扫描，我们一般使用扫描神器 Nmap 对即将要渗透的目标主机的端口进行扫描。其中，Nmap (Network Mapper(网络映射器) 是一款开放源代码的网络探测和安全审核的工具。它的设计目标是快速地扫描大型网络，你也可以用它扫描单个主机。

好了，接下来，我们首先在终端中，打开 `postgresql` 服务，在 Kali 终端中，输入命令如下（由于使用 root 登录的 Kali，sudo 可忽略）：

```
# 打开 postgresql 服务
sudo service postgresql start

```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480000480023.png-wm)


接着在实验楼的 Kali 终端中，输入命令，打开 MSF 终端：

```
# 打开终端 MSF
msfconsole
```

注意：如果是新开的环境，要记得像之前那样先使用 `msfdb init` 初始化数据库，进入 MSF 终端后使用 `db_rebuild_cache` 来重建缓存，以加快后面扫描的速度。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483506121734.png-wm)

使用扫描神器 Nmap 对渗透的目标主机，进行扫描：

```
# 使用 nmap 进行扫描
nmap -sV -T4 target
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480388571702.png-wm)



| 参数  | 参数所代表含义                                               |
| ----- | ------------------------------------------------------------ |
| `-sV` | 扫描目标主机端口，并显示详细端口信息                         |
| `-T4` | 设定 nmap 扫描的时间策略，数字为0-6，越大越快。扫描的越慢则越不容易被发现，也不会给目标带来太大的网络流量 |

由上图可看到我们所扫描的结果，在这个扫描的结果中，我们将对 Nmap 扫描的结果进行分析。

### 3.3 对扫描结果进行分析

我们将要对 Ingreslock 端口进行渗透攻击。这个漏洞，将是你见过最为简单的一个漏洞。该漏洞存在于端口 1524，一般黑客们在攻击后，使用该端口创建 Shell 后门，进而继续危害受侵者的个人主机。

由上图的扫描结果可以看出，该端口的描述为，说明在实验环境中已经有一个后门 Shell 监听在这个端口上了： 

> Metasploitable root shell

入侵的黑客连接上该端口，获得 root shell 权限，只需一条简单的命令。

如图所示，在实验楼 Kali 的 MSF 终端中，我们输入如下命令，而其中 `telnet` 命令的详细语法为，：

```
# telnet 命令语法
# telnet <target ip> <port>
telnet 192.168.122.102 1524
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483506462552.png-wm)

由 gif 动图可以知道，使用命令后已经连接上目标主机 `192.168.122.102`。

## 四、验证攻击是否成功

### 4.1 验证渗透攻击成功与否

在连接的终端中，输入命令 `ifconfig` 查看所登录的主机 ip 地址，用以验证是否渗透成功：

```
# 查看所登录主机的 ip 地址
ifconfig
```


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480393981395.png-wm)

输入命令 `hostname`，用以查看所登录的主机名称：

```
# 查看登录的主机名称
hostname
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480394128487.png-wm)

输入命令 `whoami` ，查看当前用户身份:

```
# 查看当前用户身份
whoami
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480394252124.png-wm)

到了这一步，渗透测试成功。下面我们再次回顾下，渗透攻击中，测试是否渗透成功的三个步骤参数，及其代表的含义：

| 命令参数   | 代表含义                       |
| ---------- | ------------------------------ |
| `ifconfig` | 查看当前主机的 ip 地址参数     |
| `hostname` | 查看当前主机的名称             |
| `whoami`   | 查看当前主机的所登录的当前用户 |

## 五、总结

### 5.1 实验总结

本实验中尝试使用 telnet 连接 Ingreslock 后门漏洞，较为简单。在过去的黑客攻击历史里，它常被黑客用于入侵一个暴露的服务器后绑定后门。它的利用是如此的简单，以至于你只要输入一条简单的命令，就可以直接渗透攻击入对方的主机。

现在由于补丁不断地出现，以及系统的升级，只有一些比较老旧的服务器才会出现这个漏洞。这个漏洞常常在被黑客攻陷后，留下一个 Shell 后门，方便黑客再次访问被入侵的主机。

在学完这个课程后，你至少明白了这些要点：

- Ingreslock 后门漏洞是什么
- Nmap 渗透扫描基础
- Metasploit 攻击 Ingreslock 后门漏洞的流程
- 渗透成功后的三种验证方法

## 六、推荐阅读

### 6.1 推荐阅读

攻击 Ingreslock 后门漏洞在整个 Kali 渗透训练营中，属于比较简单的一个实验。该实验只要掌握 `telent` 的命令，且对方存在 Ingreslock 后门漏洞时，即可直取对方的主机的 `root` 权限。这在平时的渗透测试中，是一件令人惊讶的事情。

学完本实验后，老师推荐大家阅读以下材料，用以加深对 Kali 环境下，渗透测试的理解，希望同学们能够仔细阅读，在动手实践的同时，增强自己的理论基础：

> https://myexploit.wordpress.com/port-number-exploits/

国外 Youtube 上的攻击 1524 端口的视频，要翻墙，能科学上网的同学，可以观看一下：

> https://www.youtube.com/watch?v=3YqpwHixiIA


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480395886665.png-wm)


## 七、课后作业

### 7.1 课后作业

该实验，主要讲述攻击 Ingreslock 后门漏洞，以及攻击该漏洞的过程，在本实验中，老师带领大家，先通过渗透扫描，再通过分析其端口信息，最后通过 `telnet` 命令，成功获取到了目标主机的 `root` 权限，并验收是否渗透成功。本实验的课后作业：

- 掌握 Ingreslock 的概念内容
- 掌握攻击 Ingreslock 的流程方法
- 阅读老师所推荐文档（可选）

对于不懂的问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。