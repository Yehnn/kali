# Web 弱密码暴力破解

## 1. 课程说明

本课程为动手实验教程，为了能说清楚实验中的一些操作会加入理论内容，也会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

## 2. 学习方法

实验楼的 Kali 系列课程包含五个训练营，本训练营主要讲述 Web 应用攻击方法。课程包含20个实验，每个实验都提供详细的步骤和截图。适用于有一定 Linux 系统基础，想快速上手 Kali 渗透测试的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触 Kali Linux 和渗透测试的概念。需要依次完成下面几项任务：

- 暴力破解简介
- 暴力破解工具
- 暴力破解实战
- 暴力破解防范

## 4. 推荐阅读

本节实验推荐阅读下述内容：

1. [Testing for Brute Force](https://www.owasp.org/index.php/Testing_for_Brute_Force_(OWASP-AT-004))
2. [Kali 中 THC-Hydra](http://tools.kali.org/password-attacks/hydra)

## 5.暴力破解

### 5.1 简介

暴力破解法（Brute Force）又被称为穷举法，是一种密码分析的方法，即将密码进行逐个推算直到找出真正的密码为止。例如一个已知是四位并且全部由数字组成的密码，其可能共有10000种组合，因此最多尝试9999次就能找到正确的密码。理论上除了具有完善保密性的密码以外，利用这种方法可以破解任何一种密码，问题只在于如何缩短试误时间。有些人运用计算机来增加效率，有些人辅以字典来缩小密码组合的范围。（来自[维基百科](https://zh.wikipedia.org/wiki/%E6%9A%B4%E5%8A%9B%E7%A0%B4%E8%A7%A3%E6%B3%95)）

暴力破解一般来说有三种方式：

- 排列组合：将数字、大写字母、小写字母、各种特殊字符排列组合，若是在不知道密码的长度情况下需要更多的逐渐增多位数，这样的运算量非常的大，这种方法需要高性能的破解算法和CPU/GPU做支持。
- 字典破解：通过社会工程学与人们常用的密码建立破译字典，然后逐个尝试。
- 排列组合+字典破解：两种方式的结合，增大破解的几率

从暴力破解的方式我们就可以看出来，他有一个简单、通俗易懂的名字，叫做：猜。一个个试，总有一天可以试出来，因为大小写字母、数字、特殊符号的个数是有限的，他们的排列组合也是有限的，只是特别多而已。

这样的方式在遇到弱口令是十分有效的，例如简单的 `123456`、`电话号码`、`abcde`。

所以只要密码设置的足够长，足够复杂便可以防范暴力破解这样的攻击方式。

### 5.2 应用场景

暴力破解在 web 应用中主要用于用户登陆环节的破解，例如网页端的登陆、QQ、邮件、FTP、VNC、Telnet 等等。

可以看到这样的方式是非常消耗时间，穷举就是一种以时间换结果的一种方式。所以主要在两个时候使用：

- 攻击初期：尝试管理员用户的登陆密码是否为弱口令，或者是默认用户名密码。若是得到管理员用户名密码后面的攻击将会简单许多。
- 攻击末期：很难找到对方的漏洞与逻辑薄弱点，就只能尝试暴力破解。

### 5.3 工具的选择

在 Kali 中有两个暴力破解的工具很受人的喜爱：

- burpsuite：拥有 web 界面，功能非常齐全，从代理到扫描、爬虫、修改发送数据包等等。总的来说：简单、易用、功能全。
- hydra：著名黑客组织 thc 的一款开源的暴力破解工具，主要对需要网络登录的系统进行快速的字典攻击，包括 FTP、POP3、IMAP、Netbios、Telnet、HTTP Auth、LDAP NNTP、VNC、ICQ、Socks5、PCNFS 等协议！

因为本实验环境的限制，无法使用 kali 的图形界面，所以我们将对 hydra 进行一些简单的实验与尝试。

### 5.4 启动环境

我们将使用 DVWA 为我们提供的环境，所以第一步依然是启动环境，熟悉这一步的同学请跳过：

在实验桌面中，双击 Xfce 终端，打开终端：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374588595.png/wm)

使用 `sudo virsh start Metasploitable2` 命令即可启动我们的靶机系统虚拟机：

![start-metasploit.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139532596.png/wm)

等待大约四分钟，待得虚拟机完全启动之后我们打开桌面上的 Firefox：

![open-firefox.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139552463.png/wm)

访问我们的靶机系统所使用的 IP 地址`192.168.122.102`：

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

### 5.5 破解尝试

切换至 Brute Force 选项卡中，我们可以看到 DVWA 为我们提供了一个登陆的环境：

![show-brute-force](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740526624.png/wm)

面对这样的登陆页面，对他的攻击方式主要有两种：

- SQL 注入
- 暴力破解

SQL 注入不在过多的讲解。暴力破解我们将使用 hydra 工具来破解。hydra 可以通过在线的方式破解，也可以通过离线的方式破解，此处我们将使用离线的方式破解。

而 hydra 的破解主要依赖于三个参数

- 登陆用户的字典：也就是说将需要破解登陆用户名都罗列出来。
- 密码的字典：将可能成功的密码都罗列出来。
- 攻击地址：我们将要攻击页面的 http-get url

开启Kali，并进行链接
```
sudo virsh start Kali
ssh root@Kali    
```
我们需要创建一个 user-list 的列表，将我们需要破解密码的账户用户名都罗列出来。

```
vim user-list.txt
```

输入这样的一些用户名：

```
admin
test
abc
good
```

因为是离线破解，我们需要一个本地的字典，我们创建这么一个文件：

```
vim passwd-list.txt
```

里面存放这样的一些内容：

```
abc123
richardwei
shiyanlou
abcdefg
12345678
pwd
password
```

如此我们已经准备好了两个必备的条件，我们在测试的环境中使用 'F12' 打开开发者工具，切换至网络选项，查看所有：

![show-f12](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740547792.png/wm)

然后我们在登陆框与密码框中随机输入：

![show-get](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740556231.png/wm)

从图中我们看到，登陆是输入的用户名与密码是通过 GET 方式提交到服务器。点击文件栏名字最长，类型是 html 的那一栏数据，我们可以看到：

![show-header](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740565764.png/wm)

是通过 GET 的方式提交的数据，我们从 URL 观察处主要提交了 3 个参数，username、password、Login，从参数的选项卡中我们也可以看到。

从请求头中我们还可以看到会提交相关的 cookie 信息，所以我们构造出这样的命令：

```
hydra -L user-list.txt -P passwd-list.txt http-get-form://192.168.122.102"/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:F=Username and/or password incorrect.:H=Cookie:security=low; PHPSESSID=19e123724a6af7960b0838d56a4432bc"

```

`-L` 参数用于指定需要尝试的用户名列表，`-P` 用于指定密码的字典，`http-get-form` 用于指定我们需要破解站点的路径与 GET 提交方式的格式。

其中 `//192.168.122.102/dvwa/vulnerabilities/brute/` 用于表示我们需要暴力破解的相关站点的 URL，紧接着 `:F=Username and/or password incorrect.` 表示当不匹配时的相关输出，而最后 `:H=Cookie:security=low; PHPSESSID=19e123724a6af7960b0838d56a4432bc` 提交 cookie 信息是因为我们当前的测试平台，是先登录在使用，在没有登陆的情况下会直接跳转至登陆页面并生成新的 cookie 信息，所以此时我们需要填写此时的 cookie 信息。

**注意**：此处的 cookie 信息是我登陆时生成的信息，所以你在使用的时候需要使用自己的相关 cookie 信息。

执行命令之后我们可以看到这样的结果：

![show-result](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740576410.png/wm)

由此我们可以看到只有正确的用户名与密码就能够匹配，尝试出来。

你会觉得此处是我们创造的两个字典，两个字典都是在我已知用户名密码的情况下创建出来的，但是若我们是在线破解呢？网上有专用的社工库，里面有大量已经被爆出来的用户名密码，所以较为简单的弱密码或者是默认未曾修改的密码是非常容易破解的。

我们可以打开 wireshark:

![find-wireshark](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740585163.png/wm)

监听 kali 所在虚拟的网络端口，其提交的数据包：

![listen-kali](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740593855.png/wm)

此时我们再次使用刚刚的命令，破解一次，我们会看到有无数的 TCP 数据包涌现出来：

![show-packet](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740603429.png/wm)

如此之多的数据包，我们不可能一一查看，我们需要一定的过滤条件来查看我们想看的数据包，所以在 FIlter 中我们输入这样的过滤条件：

```
ip.src == 192.168.122.101 && http
```

改过滤条件表示只查看从 `192.168.122.101` 也就是 Kali 主机使用的 IP 地址所发出来的数据包，并且只查看 HTTP 协议的数据包，因为每次 hydra 尝试一次，就必须与 dvwa 建立一次连接，建立一次连接需要三次握手，结束一个连接需要四次握手，其中由 Kali 主机发出来的数据包就会多达 5 个，这样就使得我们收到的数据包的数量非常庞大，所以我们需要过滤。

在输入过滤条件之后我们看到数据包显示：

![show-filter](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740611498.png/wm)

只剩下从 kali 发出来，并且是 HTTP 协议发送的数据包了，从内容上看我们就知道它尝试的方式了：

![find-try-packet](https://doc.shiyanlou.com/document-uid113508labid2427timestamp1482740633167.png/wm)

我们可以看到其实就是用户名与密码不断的尝试匹配的结果。

### 5.6 防范建议

暴力破解的原理很简单，但是不同工具使用不同的算法机制，使用不同的字典所带来的效率是不同的。

所以暴力破解是万不得已的做法，效率太低，而且破解成功的几率不高。

同时防范暴力破解方法十分简单，将密码设置的尽量长，尽量复杂，同时包含有大小写字母、数据、特殊符号。对于开发人员，所有的组件尽量不要使用默认的账户与密码。

## 6. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

- 暴力破解简介
- 暴力破解工具
- 暴力破解实战
- 暴力破解防范

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

## 7. 作业

1.利用 mutillidae 环境注册用户，完成同样的玻璃破解实验，熟练掌握 hydra 的用法。