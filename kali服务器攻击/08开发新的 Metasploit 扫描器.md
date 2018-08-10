# 开发新的 Metasploit 扫描器

## 一、实验简介

### 1.1 实验介绍

本节实验课的主要内容，先回顾 Metasploit 的模块构成及功能分析，接着将会向大家重点介绍扫描器，并开发一个属于自己的 Metasploit 扫描器。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 实验知识点

本实验将带领大家开发属于自己的 Metasploit 扫描器。整个实验将在实验楼的环境下进行，主要知识点如下：

- Metasploit 模块构成
- Metasploit 扫描器的作用
- Metasploit 扫描器的编写
- Ruby 基本知识

由于 Metasploit 扫描器模块，是由 Ruby 进行编写，如果有其他语言基础的同学，可以很快地看懂代码，另外，对于本实验的代码，老师会给出关键步骤的注释，尽可能地帮助你完成本实验的内容，开发出属于你自己的 Metasploit 扫描器。

### 1.3 实验环境

实验楼采用的实验环境包含两台虚拟机，分别是攻击机和靶机：

攻击机：Kali Linux 2.0 虚拟机，主机名是 kali，IP 地址为 192.168.122.101，默认用户名密码为 root/toor
靶机：Metasploitable2 虚拟机，主机名是 target，IP 地址为 192.168.122.102，默认用户密码为 msfadmin/msfadmin

本实验在实验楼的环境下进行。在实验楼的环境中，采用的实验环境包含两台虚拟机，分别是攻击机和靶机，攻击机和靶机的账户和密码参数分别如下：

| 主机   | 主机名            | 用户名     | 密码       |
| ------ | ----------------- | ---------- | ---------- |
| 攻击机 | `Kali Linux 2.0`  | `root`     | `toor`     |
| 靶机   | `Metasploitable2` | `msfadmin` | `msfadmin` |


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479800834785.png-wm)


## 二、环境启动

### 2.1 实验环境启动


本次实验中，我们所有的操作都是在实验楼的 Kali Linux 下进行的，宿主机为 Ubuntu 14.04。先双击桌面上的 Xfce 终端，打开终端窗口。

接着先使用 `virsh list` 命令查看当前环境中虚拟机的列表和状态，注意需要使用 sudo，另外需要加上参数 `--all` 才会显示所有关机状态的虚拟机：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374596120.png/wm)

然后我们使用 `virsh start` 命令启动虚拟机，再次查看状态虚拟机已经进入 running 状态：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374654148.png/wm)

注意由于虚拟机启动需要时间，大概要等四分钟左右我们就可以使用 SSH 访问两台虚拟机了。

首先使用 SSH 连接到 Kali，我们大部分的攻击操作都需要在 Kali 虚拟机中进行，注意用户名root，密码 toor 是不显示的，使用命令 `ssh root@kali` 即可，因为当前实验环境中已经把 IP 地址和主机名的对应写入到了 `/etc/hosts` 文件中，避免输入不好记的 IP 地址：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374660945.png/wm)

然后打开一个新的终端标签页，SSH 连接到 Metasploitable2 中，用户名 msfadmin，密码 msfadmin：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374671316.png/wm)

在 Kali 虚拟机中 `ping target` 测试两台虚拟机都可以通过内部的虚拟网络进行连接，使用 Ctrl-C 退出 ping：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374678330.png/wm)

现在两台实验环境都已经启动了，我们可以开始渗透测试了。


## 三、 Metasploit 结构模块回顾

### 3.1 Metasploit 核心

在 Metasploit 的设计中，尽可能地使用模块化的概念，以便提高代码的复用效率。该框架是用 Ruby 语言开发的，包括 Perl 写的脚本，C ，汇编，和 Python 各种组件。它基本上是专为 Linux 的操作系统设计的，因此它的命令结构具有与 Linux 命令外壳非常相似，但现在，它支持所有主流操作系统，如 Windows，Solaris 和 Mac 上。

![](https://dn-simplecloud.shiyanlou.com/uid/212008/1479204451678.png-wm)

| 英文名称   | 模块名字       | 具体作用                                                     |
| ---------- | -------------- | ------------------------------------------------------------ |
| `Aux`      | 辅助模块       | 在渗透信息搜集环节提供了大量的辅助模块支持，包括针对各种网络服务的扫描与查点、构建虚假服务收集登录密码、口令猜测等模块 |
| `Exploits` | 渗透攻击模块   | 利用发现的安全漏洞或配置弱点对远程目标系统进行攻击，以植入和运行攻击载荷，从而获得对目标系统访问控制权的代码组件 |
| `Post`     | 后渗透攻击模块 | 主要支持在渗透攻击取得目标系统远程控制权之后，在受控系统中进行各种各样的后渗透攻击动作，比如获取敏感信息，进一步括展，实施跳板攻击 |
| `Payloads` | 攻击载荷模块   | 攻击载荷是在渗透攻击成功后促使目标系统运行的一段植入代码，通常作用是为渗透攻击者打开在目标系统上的控制会话连接 |
| `Encoders` | 编码器模块     | 攻击载荷与空指令模块组装完成一个指令序列后，在这段指令被渗透攻击模块加入邪恶数据缓冲区交由目标系统运行之前，Metasploit 框架还需要完成一道非常重要的编码工序 |
| `Nops`     | 空指令模块     | 空指令（NOP)是一些对程序运行状态不会造成任何实质影响的空操作或无关操作指令 |

### 3.2 Metasploit 的几种漏洞扫描组件

#### 3.2.1 `Nmap` 扫描器

Nmap 适用于 Winodws、Linux、Mac 等操作系统。它用于主机发现、端口发现或枚举、服务发现，检测操作系统、硬件地址、软件版本以及脆弱性的漏洞。Metasploit Framework平台集成了Nmap 组件。通常在对目标系统发起攻击之前需要进行一些必要的信息收集，如获取网络中的活动主机、主机开放的端口等。

#### 3.2.2 `NeXpose` 扫描器

NeXpose 通过扫描网络，可以查找出网络上正在运行的设备，并识别出设备的操作系统和应用程序的漏洞，并对扫描出的数据进行分析和处理，生成漏洞扫描报告。

#### 3.2.3 `Nessus` 扫描器

Nessus 是当前使用最广泛的漏洞扫描工具之一。Nessus 采用 client/sever 模式，服务器端负责进行安全检查，客户端用来配置管理服务器端。在服务端还采用了 plug-in 的体系，允许用户加入执行特定功能的插件，这插件可以进行更快速和更复杂的安全检查。


## 四、编写 Metasploit 扫描器

### 4.1 进入 `scanner` 模块下

确保已经登录实验楼的 Kali 终端：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479804623200.png-wm)

首先搜索 Metasploit 的 `scanner` 模块：

```
# 查找 scanner 模块
sudo find /usr -name scanner 
```
`find` 命令用法：

```
# find 语法命令
find <位置> -name <文件名>
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479804833731.png-wm)

第一个文件就是我们要找的文件路径，通过 `cd` 命令，进入到该文件夹下。教同学们一个快速的方法，Kali 中的粘贴复制快捷键分别是 `ctrl + shit + c` 和 `ctrl + shit + v`：

```
# 进入到 scanner 文件夹下
cd /usr/share/metasploit-framework/modules/auxiliary/scanner
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479805030179.png-wm)


### 4.2 编写 `simple_tcp.rb` 文件

在该文件夹下，建立文件 `simple_tcp.rb`：

```
# 编辑  simple_tcp.rb 模块
sudo vi simple_tcp.rb
```
并在代码中，粘贴如下代码：

```
require 'msf/core'  
class MetasploitModule < Msf::Auxiliary  
    include Msf::Exploit::Remote::Tcp  
    include Msf::Auxiliary::Scanner  
  
    def initialize  
        super(  
            'Name'        => 'Mr_Zhou Scanner',  
            'Version'     => '$Revision$',  
            'Description' => 'Shiyanlou TCP Scanner',  
            'Author'      => 'lucat',  
            'License'     => MSF_LICENSE  
        )  
        register_options(  
            [  
                Opt::RPORT(12345)  
            ], self.class)  
    end  
  
    def run_host(ip)  
        connect()  
        sock.puts('HELLO SERVER')  
        data = sock.recv(1024)  
        print_status("Received: #{data} from #{ip}")  
        disconnect()  
    end  
end 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479805528729.png-wm)

接着按 `esc`，再输入 `:wq` 保存退出

### 4.3 代码解释

这个地方的代码，是我们在 MSF（Metasploit）终端中，使用了这个模块，输入 `info` 时，显示的消息 

```
    def initialize  
        super(  
            'Name'        => 'Mr_Zhou Scanner',  
            'Version'     => '$Revision$',  
            'Description' => 'Shiyanlou TCP Scanner',  
            'Author'      => 'lucat',  
            'License'     => MSF_LICENSE  
        )  
```

监听的端口设置，这里我们监听端口 `12345` ：

```
        register_options(  
            [  
                Opt::RPORT(12345)  
            ], self.class)  
```

连接主机：

```
    def run_host(ip)  
        connect()  
        sock.puts('HELLO SERVER')  
        data = sock.recv(1024)  
        print_status("Received: #{data} from #{ip}")  
        disconnect()  
    end  
```

### 4.4 创建监听程序

创建监听程序的目的，是为了当我们写的 `simple_tcp.rb` 扫描器扫描该端口时，能进行消息反馈：

在 Kali 终端中，输入命令，启动 Metasploit 终端：

```
# 启动终端操作
sudo msfconsole
```

注意：需要等待几分钟才能启动好。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479806189453.png-wm)

接着在实验楼中，打开新的 `Xfce` 终端，并创建 `shiyanlou.txt` 文件：

```
# 编辑 shiyanlou.txt 文件
vi shiyanlou.txt
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479806114320.png-wm)

在该 `shiyanlou.txt` 文件中，输入任意代码，用以当扫描器扫到该端口时，触发的的欢迎消息，这里我输入：

```
# 这里可以输入任意欢迎代码
Life is short, i use Python.
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479806414906.png-wm)

按 `esc` 并输入 `:wq` 保存退出。

在实验楼终端中输入命令，启动监听：

```
# 启动监听命令
sudo nc -l 12345 < shiyanlou.txt
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479806541659.png-wm)

回车，这时程序卡在了那里，这是正常现象，别慌。说明程序以及开始对端口 `12345` 进行了监听，命令 `nc` 是 Linux 监听端口的命令，参数 `-l` 后面代表的是监听的端口号。而 `< shiyanlou.txt` 代表的是当扫描器扫到这个端口时，反馈回去 `shiyanlou.txt` 中的内容。

### 4.5 在 MSF 启用我们的 `simple_tcp` 模块：

回到 Kali 终端中，这个时候，我们应该已经启动 `msfconsole` 完毕：

在 MSF 终端中，输入如下命令，使用 `simple_tcp` 模块：

```
# 使用刚才编写的 simple_tcp 模块
use auxiliary/scanner/simple_tcp
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479806978533.png-wm)

在终端中，输入命令 `info` ，可以查看我们编写的 `simple_tcp` 这个模块信息;

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479807067685.png-wm)

接着设置必要参数，这里我们将扫描的主机，也就是真正监听的主机的 IP 地址：

```
# 设置 RHOSTS 
set RHOSTS 192.168.122.1
```
**注意，不是 RHOST 而是 RHOSTS，多了一个 `S`，什么时候是 RHOSTS 而不是 RHOST可以 使用 `show optionis` 查看**
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479807244433.png-wm)

接着输入命令 `run`，启动进行扫描，扫描成功标志，得到扫描的目标主机的反馈：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479807343731.png-wm)


## 五、总结

通过代码的编写，以及讲解，相信你现在对 Metasploit 的编写，有了一定的了解。如果想开发更加复杂的扫描器，需要不断的进行学习，以及对 Metasploit 框架源码的演讲。

通过学习本实验，你现在应该明白了了以下知识：

- Metasploit 模块构成
- Metasploit 常用的扫描器
- Metasploit 扫描器的作用
- Metasploit 扫描器的编写

## 六、推荐阅读

对于有能力的同学，希望能够阅读英文资料，增加对 Metasploit 扫描器的理解：

> https://www.offensive-security.com/metasploit-unleashed/vulnerability-scanning/

## 七、课后作业

通过实验的讲解，希望同学们能够独立完成下面的课后作业，从而加深对 Metasploit 扫描器的理解：

- 独立 完成Metaploit 扫描器 `simple_tcp.rb` 的编写
- 阅读所推荐的 Metasploit 扫描器英文材料

对于不懂的问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。