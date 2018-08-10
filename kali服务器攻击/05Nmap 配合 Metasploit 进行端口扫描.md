# Nmap 配合 Metasploit 进行端口扫描

## 一、实验简介

### 1.1 实验介绍

本实验主要讲解 Nmap 的基本使用和高级使用。以及将扫描的数据，导入 Metasploit，以及介绍 Metasploit 中使用的模块。

本课程为纯动手实验教程，只有真正的动手实践，才能够学到真正的 IT 知识，同时，文章的末尾，会推荐一些精彩的 Nmap 使用方法，在动手实践的同时扎实理论基础。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 主要知识点

本实验中，将带领大家继续学习 Kali Linux 中的 Nmap 基本操作和高级操作，以及 Metasploit 中的模块，本课程所涉及到的知识点如下：

- Nmap 扫描器基本使用
- Nmap 扫描器高级使用
- 扫描数据导入到 Metasploit
- Metasploit 中使用的模块
- Metasploit 分析漏洞

其中，本实验五基本知识点的思维导图为：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479546439313.png-wm)


### 1.3 实验环境

实验楼采用的实验环境包含两台虚拟机，分别是攻击机和靶机：

攻击机：Kali Linux 2.0 虚拟机，主机名是 kali，IP 地址为 192.168.122.101，默认用户名密码为 root/toor

目标靶机：Metasploitable2 虚拟机，主机名是 target，IP 地址为 192.168.122.102，默认用户密码为 msfadmin/msfadmin

实验在 kali Linux 环境下进行渗透测试。先通过 ssh 登录 Kali，再使用 Metasploit 对目标主机进行操作。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479545541098.png-wm)

## 二、 Nmap 扫描器基本使用

### 2.1 Nmap 的基本使用

Nmap（网络映射器）是一款用于网络发现和安全审计的网络安全工具，它是自由软件。软件名字 Nmap 是 Network Mapper 的简称。

Nmap 可以检测目标主机是否在线、端口开放情况、侦测运行的服务类型及版本信息、侦测操作系统与设备类型等信息。它是网络管理员必用的软件之一，用以评估网络系统安全。下面我们介绍 Namp 的基本使用方法，并在实验楼的环境中，带领大家，逐一熟悉它的用法。

Nmap 包含四项基本功能：

- 主机发现（Host Discovery）
- 端口扫描（Port Scanning）
- 版本侦测（Version Detection）
- 操作系统侦测（Operating System Detection）

在实验楼的环境中，启动 Kali 和目标靶机 Metasploitable2：

```
sudo virsh start Kali

sudo virsh start Metasploitable2
```
再通过 ssh 登录 Kali，密码默认是 `toor`：

```
ssh root@Kali
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479370040177-wm)

注意，`virsh` 启动过程，大概要等四分钟左右。

先登录进入 Kali，接着输入命令，启动服务：

```
sudo service postgresql start
```

这里启动服务，会等大概十秒钟左右：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483500198879.png-wm)

然后初始化数据库。初始化数据库后需要等待大约 10 分钟的时间。

```
msfdb init
```

接着输入命令，进入 msfconsole：

```
sudo msfconsole
```
这里估计会等两分钟左右：

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479370633248-wm)

在终端输入命令，进行数据库缓存重建：

```
db_rebuild_cache
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483500288931.png-wm)


执行 db_rebuild_cache 命令来创建缓存，创建缓存的过程大概 5 - 10 分钟，创建缓存后 search 命令的执行速度可以很快在几秒钟得到结果。

环境启动后，下面将带领大家逐一体验一下 Nmap 的功能。

#### 2.1.1 全面扫描

使用全面扫描，得到更多的信息，语法如下：

```
nmap -T4 -A <target ip>
```
在实验楼终端中，输入命令：

```
# 这个命令，扫描的信息比较多，所以得稍等一分钟左右

# 我们在境配置实验楼环境的时候，已经把靶机 ip 地址配置为 target 了，下面使用 target 和 192.168.122.102 是等价的

nmap -T4 -A target
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479452449996-wm)

其中， `-T` 是扫描的速度设置:


| 参数       | 参数所代表的含义                                    |
| ---------- | --------------------------------------------------- |
| `nmap T0 ` | 非常慢的扫描，用于IDS(入侵检测系统)逃避             |
| `nmap T1`  | 缓慢的扫描，介于0和2之间的速度，同样可以躲开某些IDS |
| `nmap T2`  | 降低扫描速度，通常不用                              |
| `nmap T3`  | 默认扫描速度                                        |
| `nmap T4`  | 可能会淹没目标，如果有防火墙很可能会触发            |
| `nmap T5`  | 极速扫描，牺牲了准确度来换取速度                    |

#### 2.1.2 主机发现

Nmap 的简单扫描之一，主机发现，语法如下：

```
nmap -T4 -sn <target ip>
```
在实验楼终端中，输入如下命令：

```
nmap -T4 -sn target
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483500624979.png-wm)


#### 2.1.3 端口扫描

端口扫描的语法如下：

```
namp -T4 <target ip>
```
在实验楼终端中，输入命令：

```
nmap -T4 target
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479452945761-wm)

#### 2.1.4 操作系统扫描

操作系统扫描，语法如下：

```
nmap -T4 -O <target ip>
```
在实验楼终端中，输入命令：
```
nmap -T4 -O target
```

| 参数 | 参数所代表的含义                                             |
| ---- | ------------------------------------------------------------ |
| `-O` | 这个选项激活对 TCP/IP 指纹特征 (fingerprinting) 的扫描，获得远程主机的标志 |




![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479453056282-wm)

上述的扫描方式能满足一般的信息搜集需求。而若想利用 Nmap 探索出特定的场景中更详细的信息，则需仔细地设计 Nmap 命令行参数，以便精确地控制 Nmap 的扫描行为。

## 三、Nmap 扫描器高级使用

下面介绍一下 Nmap 的扫描器的高级使用。

#### 3.1.1 扫描整个子网

扫描整个子网的语法为：

```
nmap <target ip/24>
```

```
nmap 192.168.122.102/24

# 或者 nmap target/24 
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479450104489-wm)

#### 3.1.2 扫描多个 ip 目标：

扫描多个 ip 地址，语法如下：

```
nmap <target ip> <target ip>
```
在实验楼的终端中，输入如下命令：

```
nmap 192.168.122.102 127.0.0.1

# 等价 nmap target Kali
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479450482085-wm)


#### 3.1.3 扫描一个范围的目标：

这里扫描一百个 IP 地址：

```
nmap 192.168.122.1-100
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479375726088-wm)

#### 3.1.4 扫描除了一个 IP 外的所有主机

扫描除了一个 IP 外的所有主机，在终端中，输入命令：

```
nmap 192.168.122.102/24 -exclude 192.168.122.88
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479450670404-wm)

#### 3.1.5 扫描特定端口

nmap 扫描特定的端口

```
nmap -p<端口>，<端口>，<端口> <target ip>
```

| 参数 | 参数所代表的含义             |
| ---- | ---------------------------- |
| `-P` | 指定端口号(或端口号范围)扫描 |

比如这里扫描 22，66， 88 端口，在终端中，输入命令：

```
nmap -p80,22,66 192.168.122.102
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479450968235-wm)

#### 3.1.6 查看本地路由

```
nmap --iflist
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479451986418-wm)

#### 3.1.7 指定网口和 ip 地址

指定网口和 ip 地址，语法命令如下：

```
nmap -e eth0 <target ip>
```
在实验楼终端中，输入命令如下：

```
nmap -e eth0 target
```
| 参数 | 参数所代表含义                         |
| ---- | -------------------------------------- |
| `-e` | 告诉 nmap 使用哪个接口发送和接受数据包 |

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479453596702-wm)

#### 3.1.8 定制探测包

Nmap提供 `-scanflags` 选项，用户可以对需要发送的 TCP 探测包的标志位进行完全的控制。

#### 3.1.9 SYN 扫描

利用基本的 SYN 扫描方式，探测窗口的开放状态：
```
nmap -sS -T4 target
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479453877376-wm)

Nmap 默认扫描只扫描 1000 个最可能开发的端口。

#### 3.1.10 FIN 扫描

利用 FIN 扫描方式探测防火墙状态。FIN 扫描方式用于识别端口是否关闭，收到 RST 回复说明该端口关闭，否则说明是 open 或 filtered 状态。

在实验楼终端中，输入命令如下：

```
nmap -sF -T4 target
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479458103334-wm)

#### 3.1.11 ACK 扫描

利用 ACK 扫描判断端口是否被过滤。针对 ACK 探测包，未被过滤的端口（无论打开、关闭）会回复 RST 包。

```
nmap -sA -T4 target
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479458212222-wm)

#### 3.1.12 扫描路由器 TFTP

大多数的路由器都支持TFTP协议（简单文件传输协议），该协议常用于备份和恢复路由器的配置文件，运行在 UDP 69 端口上。使用上述命令可以探测出路由器是否开放 TFTP。

```
nmap -sU -p69 -nvv target
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479458727550-wm)

## 四、扫描数据导入 Metasploit

### 4.1 扫描的数据导入 Metasplit 

使用 nmap 对目标主机的子网进行扫描，并且以 `xml` 的格式，输入到文件名为 `shiyanlou.xml`的文件中：

```
nmap -sV -Pn -oX shiyanlou.xml target/24
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479459281528-wm)

输入命令，导入文件 `shiyanlou.xml` 到 Metasploit 中：

```
db_import shiyanlou.xml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483508737294.png-wm)


利用命令查看存储在数据库的扫描结果，在终端中，输入命令：

```
services
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479459678086-wm)

-----------

## 五、在 Metasploit 中使用模块

### 5.1 在 Metasploit 中显示模块

分别通过 `show` 命令，可以查看可用模块，由于模块数量较多，加载时间需要耐心等待：

```
msf > show
msf > show auxiliary
msf > show exploits
msf > show payloads
msf > show encoders
msf > show nops
```
### 5.2 在 Metasploit 中搜索模块并使用

`ms12-020` 这个漏洞，是微软发布的安全公告，远程桌面中的漏洞可能允许远程执行代码 (2671387)。发布的时间，是 12 年的 3 月 31 日。如果目标主机存在 `ms12-020` 这个漏洞，那么我们攻击的具体流程为：

##### 1. 使用命令，搜索模块：

```
search ms12-020
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479461240666-wm)

##### 2. 命令 `use` 使用模块：

```
use auxiliary/dos/windows/rdp/ms12_020_maxchannelids
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483500764733.png-wm)


##### 3. 命令 `show options` 查看需要填写的参数：

```
show options
```
##### 4. 再由 `set` 设置所需的参数，如这里的 `RHOST` 参数：

```
set RHOST 192.168.122.102

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483508920880.png-wm)


##### 5. 最后由 `run` 命令，执行我们的 `auxiliary` 攻击模块


## 六、分析漏洞

现在由我们之前的 Nmap 扫描，对靶机信息进行枚举，以便提高后续的渗透成功率。这些基础的工作，在展开渗透测试中是非常重要的一步。由之前的 Nmap 扫描，掌握了目标靶机的一些信息，如最基本的：

- 其运行的操作系统版本
- 服务器名称
- 服务的版本
- 端口开放情况

在分析漏洞的过程中，善于利用 Nmap 进行分析。再举个例子，例如判断防火墙后主机状态，则用 `nmap -sP IP 或者网段` 。其中

| 参数            | 参数所代表含义                                             |
| --------------- | ---------------------------------------------------------- |
| `-sP ping` 扫描 | 通过发送特定的ICMP报文，并根据返回的响应信息来判断主机状态 |

只有不断地分析，才能更好地找出目标主机的漏洞，从而提高渗透率。

## 七、作业

### 7.1 课后作业

按照实验步骤，那么你应该熟悉了 Metasploit 的使用流程，现在请你完成以下作业：

1. 使用 Nmap 的基本用法
2. 使用 Nmap 的高级用法
3. 使用 search 搜索一个漏洞，并通过 show option 查看设置选项
4. 使用 set 设置必须参数，并进行攻击

如果想了解更多的知识，推荐阅读：

> https://nmap.org/man/zh/

对于不懂的问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。