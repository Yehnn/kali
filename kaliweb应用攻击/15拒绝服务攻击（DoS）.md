# 拒绝服务攻击（DoS）

## 1. 课程说明

> **注意**：本实验环境与上一章节实验环境不同，请勿延续上个实验环境，若是已保留，请结束实验，重新启动实验环境即可

本课程为动手实验教程，为了能说清楚实验中的一些操作会加入理论内容，也会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

## 2. 学习方法

实验楼的 Kali 系列课程包含五个训练营，本训练营主要讲述 Web 应用攻击方法。课程包含20个实验，每个实验都提供详细的步骤和截图。适用于有一定 Linux 系统基础，想快速上手 Kali 渗透测试的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触 Kali Linux 和渗透测试的概念。需要依次完成下面几项任务：

- Dos 简介
- Dos syn 原理
- Dos 实战

## 4. 推荐阅读

本节实验推荐阅读下述内容：

1. [ab 压力测试工具的使用：](https://httpd.apache.org/docs/2.4/programs/ab.html)

## 5. 拒绝服务攻击

### 5.1 简介

DoS到底是什么？DoS 是 Denial of Service 的简称，即拒绝服务攻击，这样的攻击其目的是使计算机或网络无法提供正常的服务。

作个形象的比喻来理解 DoS。街头的餐馆是为大众提供餐饮服务，如果一群地痞流氓要 DoS 餐馆的话，手段会很多，比如霸占着餐桌不结账，堵住餐馆的大门不让路，骚扰餐馆的服务员或厨子不能干活，甚至更恶劣……相应的计算机和网络系统则是为 Internet 用户提供互联网资源的，如果有黑客要进行 DoS 攻击的话，可以想象同样有好多手段！今天最常见的 DoS 攻击有对计算机网络的带宽攻击和连通性攻击。带宽攻击指以极大的通信量冲击网络，使得所有可用网络资源都被消耗殆尽，最后导致合法的用户请求无法通过。连通性攻击指用大量的连接请求冲击计算机，使得所有可用的操作系统资源都被消耗殆尽，最终计算机无法再处理合法用户的请求。

传统上，攻击者所面临的主要问题是网络带宽，由于较小的网络规模和较慢的网络速度的限制，攻击者无法发出过多的请求。虽然类似 “the ping of death” 的攻击类型只需要较少量的包就可以摧毁一个没有打过补丁的 UNIX 系统，但大多数的 DoS 攻击还是需要相当大的带宽的，而以个人为单位的黑客们很难使用高带宽的资源。为了克服这个缺点，DoS 攻击者开发了分布式的攻击。攻击者简单利用工具集合许多的网络带宽来同时对同一个目标发动大量的攻击请求，这就是 DDoS(Distributed Denial of Service)攻击。[1] 

无论是 DoS 攻击还是 DDoS 攻击，简单的看，都只是一种破坏网络服务的黑客方式，虽然具体的实现方式千变万化，但都有一个共同点，就是其根本目的是使受害主机或网络无法及时接收并处理外界请求，或无法及时回应外界请求。（此段来自于[百度百科](http://baike.baidu.com/view/209571.htm)）

DOS 的攻击大多是使用数据包泛洪来攻击，所谓的泛洪非常好理解，就是用大量的数据包来淹没服务器，让服务器没有办法喘过气来，如此服务器便没有办法正常工作了。

而数据包的泛洪主要有：

- UDP flood
- SYN flood
- 异常 TCP flood

这类的泛洪攻击主要的方式有：

- Ping flood :发送ping包的攻击者压倒性的数量给受害者
- Ping of Death：攻击者发送修改后的 ping 数据包，如添加分包使前后的逻辑不正确，或者加长数据包使其超过 IP 报文的限制
- Teardrop attacks：向目标机器发送损坏的IP包，诸如重叠的包或过大的包载荷。
- UDP flood：利用大量UDP小包冲击服务器
- SYN flood：利用 TCP 的连接过程，来消耗系统资源。
- CC（challenge Collapsar）：通过构造有针对性的、对最为消耗服务器端资源的业务请求，让服务器“劳累过度”而停止服务等等

此外还有 ARP 欺骗等一类的攻击方式。

### 5.2 原理

Dos 攻击的方式非常的明确，我们可以通过 SYN 攻击来理解 Dos 的攻击。

在所有黑客攻击事件中，SYN 攻击是最常见又最容易被利用的一种攻击手法。在 2000 年时 YAHOO 的网站遭受的攻击，就是黑客利用的就是简单而有效的 SYN 攻击，而 SYN 攻击依靠的就是 TCP 三次握手。（想对网络有更深入了解的同学建议学习由浅入深学网络课程，其中会将三次握手的数据包详情都展示给大家。）

>**TCP**( Transmission Control Protocol 传输控制协议）是一个 Internet 协议套件的核心协议。它起源于最初的网络实现，它补充了互联网协议（IP,网络层的 IP 协议，IPv4、IPv6）。因此，整个套件通常被称为 TCP/IP。TCP 提供可靠的，有序的，和错误检查流在 IP 网络上的主机通信的运行的应用程序之间传递数据。主要的互联网应用，如万维网（www）、电子邮件(smtp,pop)、远程管理(telnet)和文件传输(ftp)都是依靠 TCP。（来自于[wikipedia_DDOS](https://en.wikipedia.org/wiki/Denial-of-service_attack#.28S.29SYN_flood)）

TCP 是面向连接的，可靠的进程通信的协议，提供全双工服务，即数据可在同一时间双向传输，也正是因为 TCP 的可靠连接，所以广泛应用在大多数的应用层协议，而 TCP 在建立一个连接需要三次交互，这个过程叫 Three-way Handshake（三次握手）。

![TCP-connect-finialpacket.png](https://doc.shiyanlou.com/document-uid113508labid2122timestamp1476354199104.png/wm)（此图来自于由浅入深学网络）

1. 由客户端使用一个随机的端口号，向服务器端特定的端口号发送 SYN 建立连接的请求，并将 TCP 的 SYN 控制信号位至为1。SYN（synchronous）是 TCP/IP 建立连接时使用的握手信号，客户端将该段的序列号设置为一个随机值。
2. 服务器端收到了客户端的请求，会向客户端发送一个确认的信息，表示已收到请求，这使得数据包里会将 ACK 设置为客户端请求序列号+1，同时服务器端还会向客户端发送一个 SYN 建立连接的请求，SYN 的序列号为另外一个随机的值
3. 客户端在收到了服务器端的确认型号以及请求信号之后，也会向服务器端发送一个确认信号，确认信号的序列号为服务器端发送来的请求序列号+1。在这一点上，客户端和服务器都已收到了连接的确认。步骤1，2建立一个方向的连接，步骤2、3建立了另一个方向的连接，并被确认。如此便建立了一个全双工通信。

这样是一次完整的 TCP 连接过程，整个服务器建立连接时 TCP 状态的变化过程：

CLOSED -> LISTEN -> SYN recv -> ESTABLISHED

要完全建立一次 TCP 连接，需要经过这样的一些步骤与状态，整个流程不能少，然而客户端发送了 SYN 的请求连接的信号，而服务器响应了我的请求，发送来了确认信息以及他的请求信号，并将该信息加入未连接的队列中，变成SYN_RCVD的状态，此时客户端不再响应服务器，剩下的流程不再进行了，那么只能完成一次半连接状态，服务器需要一段时间的等待才会放弃此次的连接。

虽然这样的一个过程不会消耗太多的资源，但是量变产生质变，成千上万这样的请求会使得服务器力不从心，资源不足，CPU 等一些珍贵的资源无法分配。

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081470294625485-wm)

### 5.3 实战

实现 Dos 的工具有很多，诸如 hping3，nping，nmap，ab等等。

同样首先我们将使用 `sudo virsh start Kali` 来启动 Kali 的实验环境：

![start-kali](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483601152509.png/wm)

通过 `sudo virsh start Metasploitable2` 命令来启动靶机系统：

![start-metasploit](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483601144011.png/wm)

启动虚拟机需要一定的时间，稍等大约 4 分钟我们便可使用 `ssh root@kali` 登陆至 Kali 的终端：

![ssh-kali](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483601135573.png/wm)

若是出现这样的情况，说明 Kali 虚拟机还未启动完全：

![error-ssh](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483601129180.png/wm)

![error-ssh2](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483601120965.png/wm)

我们可以通过这样一个命令来模拟一次 SYN flood 的攻击：

```
hping3 -S -P -U --flood -V --rand-source 192.168.122.102
```

![run-hping3](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483601113642.png/wm)

该命令便是通过 hping3 工具，设置 SYN、PUSH、URG 的标志位，向 192.168.122.102 发动 flood 攻击，并且其中 `--rand-source` 参数设置数据包中的 IP 源地址为随机地址，而不是其本身的 IP 地址。

稍等片刻我们再次访问 `192.168.122.102/mutillidae` 时，我们会发现其异常的缓慢，网页在不停的加载中，一直处于加载中：

![show-result](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483074122435.png/wm)

这样访问异常的慢，便是由 Dos 所导致，而之所以还能服务，这是因为我们所发送的数据包量还不够大，还在服务器的承受范围之内。

于此同时我们还可以尝试登陆我们的靶机 `ssh msfadmin@target`，我们都会发现异常的缓慢，半天密码提示框都没有出现：

![ssh-no-response](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483601102633.png/wm)

若是如此，我们觉得效果不明显，我们可以在 Kali 中使用 `ctrl+c` 来结束 hping3 命令。

同样我们还可以使用 apache 出品的一个压力测试工具 ab：

```
ab -n 10000000 -c 600 http://192.168.122.102/mutillidae
```

该命令便是向我们靶机模拟发送1000万个请求，同时并发量为600个，我们可以 ssh 登陆上我们的靶机：

![ssh-metasploit](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483601091371.png/wm)

通过 top 查看此时系统的负载情况:

![show-top](https://doc.shiyanlou.com/document-uid113508labid2432timestamp1483074108207.png/wm)

我们可以看到此时系统的负载非常的高，并且全是 apache2 的进程。这就是 Dos 攻击，让目的主机疲于应付我们的大量请求，而无法处理正常的请求。

### 5.3 防范

在知道 Dos 的攻击原理之后我们可以很清楚的看到，就是使用大量的数据包，来让服务器的 CPU 应接不暇，所以防范这样的攻击：

- 设置 apache 或者 nginx 等工具中单个 IP 的连接数量
- 设置 iptables 中 syn 的连接数量限制
- 设置 net.ipv4.tcp_syncookies 的开启，当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击

当然这也是针对与 SYN 洪攻击的应对措施，Dos 的攻击方式还有很多中情况，但是原理都与 SYN 类似，我们常说的 DDos 的攻击是 Distributed Denial of Service 的缩写，也就是 Dos 的进化版，随着服务器的性能提升，我们使用单个设备这样的攻击压力，服务器已经能够承受，所以发展出了 DDos。

DDos 就是分布式的 Dos，一台设备不够，我们便找更多的机器来凑，就是使用更多的设备来帮我们一起攻击，直到目标设置受不了为止。

## 6. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

- Dos 简介
- Dos 中 syn 原理
- Dos 实战
- Dos 防范

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。