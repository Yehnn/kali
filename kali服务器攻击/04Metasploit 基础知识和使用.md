# Metasploit 基础知识和使用

## 一、实验介绍

### 1.1 实验简介

本实验主要介绍 Metasploit 基本框架的知识，以及对 Metasploit 的核心概念，主要的攻击方式，基本使用方法进行介绍。Metasploit 渗透测试框架，提供很多的接口，选项，变量以及模块，使用者可能刚开始被这些繁杂的东西所吓倒，但是只要熟悉一些常用的工具方法，就能做到举一反三。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 实验知识点

本实验将分别介绍 Metasploit 的基本框架知识，以及它的核心概念，以及基本的使用方法。在实验后面，也会提及一些常用的工具集合，实验的主要的知识点如下：

- Metasploit 基本框架
- Metasploit 核心概念
- 基本使用方法
- 攻击工具集合
- 对目标主机进行扫描

### 1.3 实验环境

本实验在实验楼的环境中进行，通过实验楼环境，由 ssh 登录 Kali，并在 Kali 上进行相关操作。本实验提供的目标主机为 Metasploitable2。通过 Metasploit 的使用，让使用者更加熟悉 Metasploit 的环境，从而掌握渗透流程。


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479276932432.png-wm)


## 二、Metasploit 介绍

Metasploit 是一款开源的安全漏洞检测工具，可以帮助安全和IT专业人士识别安全性问题，验证漏洞的缓解措施，并管理专家驱动的安全性进行评估，提供真正的安全风险情报。这些功能包括智能开发，代码审计，Web 应用程序扫描，社会工程。团队合作，在 Metasploit 和综合报告提出了他们的发现。这款工具，附带数百个已知的漏洞，并保持频繁更新。被整个安全社区冠以"可以黑掉整个宇宙"外号的渗透框架。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479276539883.png-wm)


### 2.1 Metasploit 的核心

在 Metasploit 的设计中，尽可能地使用模块化的概念，以便提高代码的复用效率。该框架是用 Ruby 语言开发的，包括 Perl 写的脚本，C ，汇编，和 Python 各种组件。它基本上是专为 Linux 的操作系统设计的，因此它的命令结构具有与 Linux 命令外壳非常相似，但现在，它支持所有主流操作系统，如 Windows，Solaris 和 Mac 上。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479204451678.png-wm)


### 2.2 Metasploit 框架介绍

Metasploit 有三种接口模块用来执行利用各种 exp：控制台、命令行、web。实际上这三种模块都没有太大的区别，但是一般情况下控制台是三种方式中功能最全、最强大的。

#### 2.2.1 基础知识介绍

Metasploit 是由 HDmoore 创立的，它是一个开放的漏洞研究与渗透代码开发的平台。MSF 终端（MSFCONSOLE）是 Metasploit 框架一个非常流行的用户接口，其灵活强大，深受用户喜爱。在下面的实验中，我们将使用这个 MSF 终端进行实验。 Metasploit 框架，总体由由五部分组成，它们分别是：
1. 模块
2. 接口
3. 插件
4. 功能程序
5. 基础库文件

我们最常用的部分是模块部分。模块又可以分为辅助模块 Auxiliary、渗透攻击模块 Exploits、攻击载荷模块 Payloads、空指令模块 Nops、编码器模块 Encoders 和后渗透攻击模块 Post 模块组成。

- 渗透攻击模块（Exploit）：这是一段程序，运行起来的时候会利用目标的安全漏洞进行攻击。

- 辅助模块（Auxiliary）：这里包含了一系列的辅助支持模块，包括扫描模块、Fuzz 测试漏洞发掘模块、网络协议欺骗以及其他一些模块。

- 编码器模块（Encoder）：编码器模块通常用来对我们的攻击模块进行代码混淆，来逃过目标安全保护机制的检测。目标安全保护机制包括杀毒软件和防火墙等。

### 2.3 基本使用方法

MSF 终端（Msfconsole）作为Metasploit 框架最受欢迎的用户接口，提供与用户交互式的输入，用户体验非常好。下面将带领大家，使用 MSF 终端在实验楼环境中进行实验。在终端里输入如下命令，这个时候，启动 msfconsole 会比较久，大概 `两到三分钟`，这是由于加载模块比较多的原因。 如果一切顺利，你将会看到：

```
sudo msfconsole
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479278737070-wm)


#### 2.3.1 help 命令

在 MSF 终端中，输入命令：

```
help
```
或者直接输入一个问号 `?`，也是同样的效果。其中，`Command` 是命令名称， `Description` 是命令描述：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483496824720.png-wm)



#### 2.3.2 search modules 命令

命令 search modules，用以搜索可用模块，在实验楼的 MSF 终端中输入如下命令：

```
search linux
```
由于文件比较多，搜索 linux 这个模块的文件所花费时间，大概在 `一分钟` 左右。如果一切顺利，你将会看到：

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479279805405-wm)

如果屏幕足够大，可以看到参数整齐地显示：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479280408105.png-wm)

图中参数分别有 `Nmae`,`Disclosure Date`，`Rank`，`Description` 四个，分别代表的含义是名字，泄露日期，是否正常，以及功能描述。

#### 2.3.3 use 命令

在 MSF 终端中，输入命令，使用 `jtr_linux` 密码破解模块：

```
use auxiliary/analyze/jtr_linux
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483496957806.png-wm)

#### 2.3.4 show options 命令

使用 show options 查看模块库的有效选项，在 MSF 终端中，输入命令，如果一切顺利，你将会看到：

```
# 查看参数选项

show options
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483497085162.png-wm)


或者使用 info 命令，能得到更详细的信息：

```
# 显示更详细的信息

info 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483498319607.png-wm)


#### 2.3.5 setg 和 set 命令

命令 setg 用于给某个对象赋值的同时设定作用域为全局，在模块进行切换的时候，该对象的值不会被改变。而 set 命令，只是给定某个特定的对象赋值。

如在终端中，输入命令 set 设置 JOHN_PATH ：

```
set JOHN_PATH /usr/share/metasploit-framework/data/john/wordlists/ password.lst
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483498114720.png-wm)


#### 2.3.6 exploit 命令

命令 exploit 用于启动一个渗透攻击模块，这个用法会在后面演示攻击 MySQL 服务器时用到。

#### 2.3.7 run 命令

命令 run 用于在设置一个辅助模块需要的所有选项之后，启动该辅助模块。

#### 2.3.8 back 命令

命令 back 用于取消当前选择的模块，并且退回到上一级命令窗口，在终端中输入：

```
back
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483497232541.png-wm)


## 三、攻击方式

### 3.1 Metasploit 的主要攻击方式

#### 3.1.1 Exploit 模块

Exploit 是攻击模块，基本上目前所有的操作系统在 Metasploit Framework上均有对应的攻击模块。

Exploit 操纵计算机系统中特定漏洞的恶意代码. Metasploi 提供了跨多个操作系统和应用程序的 Exploit，提供了突破一台电脑的多种途径。可以用 Nessus 搭配 Nmap 进行漏洞扫描,并使用 Metasploit 进行漏洞利用。

#### 3.1.2 Payload 模块

Payload 是在目标系统被攻陷之后执行的代码，如添加用户账号，获取shell交互权限等。

### 3.2 基本使用方法

#### 3.2.1 登录 Metasploit 控制台

首先登陆 Kali, 接着打开终端，输入命令，测试是否能够 Ping 通目标主机：

```
ping 192.168.122.102
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483498605024.png-wm)


接着在终端中，输入命令，查看 `postgresql` 状态：

```
# 查看 postgresql 状态
sudo service postgresql status
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483497393392.png-wm)


已经激活（Active exited），按 `Ctrl + c` 退出。若未激活，则输入命令激活：

```
sudo service postgresql start
```

然后初始化数据库。初始化后需要等待大约 10 分钟。

```
msfdb init
```

接着在终端中，输入命令，打开 Metasploit 的控制台：

```
sudo msfconsole
```
由于加载模块比较多，这个过程大概两到三分钟左右，请耐心等待，卡在那不动了，不是你电脑的问题，别慌。如若一切顺利，你将会见到 Metaploit 的控制台登录界面：

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479275229119-wm)

在控制台中，重建数据缓存，输入命令：

```
db_rebuild_cache
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483498265618.png-wm)



#### 3.2.2 攻击 MySQL 数据库服务器

下面演示下攻击 MySQL 数据库服务器大概流程：

##### 1. 使用 mysql_login 登录模块：

```
use auxiliary/scanner/mysql/mysql_login
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483497778084.png-wm)


##### 2. 由 show options 查看模块库有效选项：

```
show options
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479289201711-wm)

##### 3. 设置攻击的目标主机：

```
set RHOST 192.168.122.102
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483499433258.png-wm)


##### 4. 用 set 设置 USER_FILE 用户名文件（这里只是走流程）：

```
set user_file /root/Desktop/usernames.txt
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483499505721.png-wm)

##### 5. 用 set 设置 PASS_FILE 密码文件：

```
set pass_file /root/Desktop/passwords.txt
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1483499589325.png-wm)


##### 6. 使用攻击命令，执行攻击，在命令行终端中输入:
```
# 攻击命令 
exploit
```
以上就是攻击的整个流程。

![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479445693384-wm)

##### 7. 攻击原理

Metasploit 会尝试输入包含在两个文件中的所有用户名和密码组合。成功后，会有名字密码组合。原理是利用目标靶机上的 MySQL 漏洞。在选择 MySQL 登录利用模块之后，我们设置了选项并执行了漏洞利用，这让我们能够爆破 MySQL 登录。Metasploit 在这过程中，使用 `提供的用户名和密码文件`进错尝试。渗透的账户和密码，需要平时的搜集。所以平时的钓鱼网站，诱骗用户注册，也就是这回事。


### 3.3 对目标主机进行扫描

##### 3.3.1 使用扫描神器 Nmap 进行扫描

Metasploit Framework 平台集成了 Nmap 组件。通常在对目标系统发起攻击之前需要进行一些必要的信息收集，如获取网络中的活动主机、主机开放的端口等。

在命令端中，输入命令：

```
nmap 192.168.122.101
```
![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479292353288-wm)

由扫描可知，其中 PORT 端口 22/tcp 为 open 开放模式。

##### 3.3.2 Nessus

Nessus 是当前使用最广泛的漏洞扫描工具之一。Nessus 采用 client/sever 模式，服务器端负责进行安全检查，客户端用来配置管理服务器端。在服务端还采用了 plug-in  的体系，允许用户加入执行特定功能的插件，这插件可以进行更快速和更复杂的安全检查。


## 四、开发定制

### 4.1 开发定制

Meterpreter 的插件以动态链接库文件的形式存在，可以选择你喜欢的编程语言按照 Meterpreter 的接口形式编写你需要的功能，然后编译成动态链接库，拷贝至相应目录即可。


![实验楼](https://dn-simplecloud.shiyanlou.com/2120081479347377867-wm)

## 五、总结

本实验中我们初步接触了 Metasploit 的基本概念，实践中学习了以下知识点：

- Metasploit 基本框架
- Metasploit 核心概念
- 基本使用方法
- 攻击工具集合
- 对目标主机进行扫描

## 六、作业

### 6.1 实验作业

按照实验教程步骤，熟悉 Metasploit 操作流程，学会使用 search 和 use ，以及 show options 与 set，对于攻击流程，有一个清晰认识。

