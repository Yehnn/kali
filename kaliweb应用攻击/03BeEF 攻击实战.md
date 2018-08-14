# BeEF 攻击实战

## 1. 课程说明

本课程为动手实验教程，为了能说清楚实验中的一些操作会加入理论内容，也会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

## 2. 学习方法

实验楼的 Kali 系列课程包含五个训练营，本训练营主要讲述 Web 应用攻击方法。课程包含20个实验，每个实验都提供详细的步骤和截图。适用于有一定 Linux 系统基础，想快速上手 Kali 渗透测试的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触 BeEF，并使用 BeEF 完成一次渗透实战，原理并不困难，但是原理很重要，需要依次完成下面几项任务：
v
- BeEF 的大体实现结构
- BeEF 的所属攻击类型
- BeEF 的网页源码窃取
- BeEF 的弹窗提示
- BeEF 的钓鱼登陆窃密实现

## 4. 推荐阅读

本节实验推荐阅读下述内容：

1. [BeEF github wiki:](https://github.com/beefproject/beef/wiki/Architecture)

## 5. BeEF 简介

### 5.1 BeEF 结构

BeEF 是目前最为流行的 web 框架攻击平台，专注于利用浏览器漏洞，它的全称是 The Browser Exploitation Framework。

BeEF 是一个用 ruby 语言编写的开源框架，你可以在 github 中找到它，若是愿意的话向他们还可以提交自己的代码。

BeEF 是这样工作的：

![beef-framework.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111185964.png/wm)
（此图来自于 [beef_wiki](https://github.com/beefproject/beef/wiki/Architecture)）

BeEF 会提供一个 web 界面供操作，只要访问了嵌入 hook.js 页面，亦或者加载了 hook.js 文件的浏览器，就会不断的以 GET 的方式将其自身的相关消息发送到 BeEF 的 server 端，如此便可以掌握对方的详细信息，便可寻找相关漏洞，实行攻击亦或者将至当成肉鸡想其他设备发起攻击。其中被攻击的浏览器通常称之为 Zombie。

> 就像钓鱼一般，只要鱼想吃鱼饵就会吃到鱼钩，我们就可通过鱼钩来控制鱼、捕获鱼，攻击目标就是鱼，而鱼钩便是 hook.js，这个名字也非常的形象叫做 hook。

![beef-framework1.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111222718.png/wm)
（此图来自于 [beef_wiki](https://github.com/beefproject/beef/wiki/Architecture)）

而一旦 BeEF 运行起来，其中有两个组件最为重要：

- User Interface
- Communication Server

其中 User Interface 用于提供我们所操作的 Web 界面。而 Communication Server 则是用于通过 HTTP 来接收 “中招” 用户浏览器发来的信息，这组件是 BeEF 最为重要的组件了。

其他的功能的都是通过扩展与模块添加而来，我们可以通过 BeEF 的目录便可看出：

在实验桌面中，双击 Xfce 终端，打开终端：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374588595.png/wm)

然后我们使用 `sudo virsh start Kali` 命令启动虚拟机，注意区分大小写，虚拟机的名字是大写的字母开始:

接着使用 SSH 连接到 Kali，注意用户名 root，密码 toor 是不显示的，使用命令 `ssh root@kali` 即可，因为当前实验环境中已经把 IP 地址和主机名的对应写入到了 `/etc/hosts` 文件中，避免输入不好记的 IP 地址：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374660945.png/wm)

然后我们通过 `cd /usr/share/beef-xss/` 命令进入 BeEF 的主目录中：

![beef-ls.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111252573.png/wm)

其中文件的作用：

- arerules：该目录中的功能会在 BeEF 启动前预加载，并在目标被 hook 的时候触发
- beef：一个运行文件，用于启动 BeEF 所有功能的运行脚本
- beef_cert.pem：用于 https 时候的证书
- beef_key.pem：用于 https 时候的密钥
- config.yaml：用于配置 BeEF 启动是加载的一些参数
- Gemfile：用于配置 ruby 的一些依赖包
- core：是 BeEF 的核心目录，主要功能的实现。并负责加载 extension（扩展模块）与 moudule（攻击模块）
- extensions：用于放置扩展模块
- modules：用于放置各种攻击模块，其中攻击模块主要有这样三个文件：
  + command.js：命令文件；
  + config.yaml：模块的配置文件；
  + module.rb：定义了模块中使用的类，以及一些功能上的处理。

其中最为主要的便是 core、extensions、modules 这三个目录。

### 5.2 BeEF 的类别

BeEF 的攻击手段属于我们在上文中所介绍的 XSS。

那么所谓的 XSS 是什么呢？

跨站脚本攻击(Cross Site Scripting)，为不和层叠样式表(Cascading Style Sheets，CSS)的缩写混淆，故将跨站脚本攻击缩写为 XSS。恶意攻击者往 Web 页面里插入恶意 JavaScript 代码，当用户浏览该页之时，嵌入其中的 JavaScript 代码会被执行，从而达到恶意攻击用户的目的。（来自百度百科）

恶意的代码能够执行是因为当我们访问一个页面时，浏览器回去服务器中下载该页面，然后从上到下的去解析 html，当遇到 script 编辑时会去调用 javascript 的解析器并执行该脚本，执行完毕之后继续 html 的解析，所以恶意者的代码能够被运行。

而通过这样的攻击可以做到盗取用户 cookie、账户劫持、DDOS 等等。

XSS 的攻击又分为：

- 反射型 XSS：服务器不对用户提交的数据进行存储，直接输出。也就是通过 GET 提交表单或者其他操作获取变量，直接通过 URL 输出运行。
- 存储型 XSS：提交的恶意代码存储在了服务器上，只要访问该网站的用户都受到影响，而且如果不删除恶意代码，攻击将不会停止，所以存储型 XSS 也叫持续型 XSS。
- DOM 型 XSS：修改网页中的DOM节点，再经过网页脚本解析之后产生，与反射型极为类似。

而在 BeEF 中便是通过 hook.js 使得攻击者与被攻击者之间建立连接关系，这样便可以获取被攻击者的详细信息，并且也能够通过 hook.js 将本地的一些攻击代码传至被攻击者的主机中执行，并且 BeEF 不仅能够进行一些简单的 XSS 攻击，还能够联合 metasploit 进行更深入的攻击，甚至是远程操控被攻击者的设备。

其中最为关键的 hook.js 并不是一个静态存在的 js 文件，它是由 beef 来动态建立的，有兴趣的同学可以在 `core/main/handlers/hookedbrowsers.rb` 的第 50 行左右与 `core/main/handlers/modules/beefjs.rb` 中的 16 行左右查找到其如何建立与如何定义的。

hook.js 主要就是为了伪造用户的身份来获取用户的相关信息。

## 6 BeEF 基础

### 6.1 BeEF 的启动

通过上文所述我们对 BeEF 有了一个大概的了解，至少知道它的作用了，接下来我们就来做一些简单的尝试。

我们通过这样的一个命令进入 BeEF 的主目录中：

```
cd /usr/share/beef-xss
```

![beef-home.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111279208.png/wm)

通过 `less config.yaml` 查看 BeEF 的配置信息:

![beef-less-config.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111293477.png/wm)

在配置文件中我们查看关于 http 相关的信息，我们可以知道 BeEF 运行在 3000 号端口上：

![beef-config-http.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111309283.png/wm)

通过搜查 user 关键字，我们可以看到 BeEF 所使用的数据库，以及数据库的名称，用户名，密码，以及管理员的用户名、密码：

![beef-config-db.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111323676.png/wm)

通过 `q` 退出配置文件的查看之后运行主目录中程序的启动脚本

```
./beef
```

![beef-start.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111350472.png/wm)

接着我们便会看到字符界面中有程序启动的信息浮现出来，启动 BeEF 有些许缓慢，需要耐心等待，待得出现这样一行信息便表示 BeEF 已经完全启动好了，我们便可以通过浏览器访问后台页面了。

![beef-boot.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111362704.png/wm)

我们可以通过桌面的 Firefox 访问刚刚我们所获得的 URL，进入后台页面会要求我们输入用户名密码，而用户名密码都是使用的默认值：` beef `。

![beef-login.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111378046.png/wm)

进入后台页面之后我们看到这样的几个板块：

![beef-panel.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111394566.png/wm)

然后我们新打开一个页面访问这样的一个 url：

```
http://192.168.122.101:3000/demos/butcher/index.html
```

![beef-view-demo.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111413148.png/wm)

访问这个页面之后，我们回到后台页面，我们会发现在 `Online Browsers` 中多了一个主机，这是因为在我们刚刚访问的页面中便嵌入了 `hook.js`，而我们的浏览器从服务器中下载了该页面的 html 并解析执行，在读取到 `hook.js` 的时候也不管三七二十一一股脑的都执行了，由此该浏览器便与 BeEF 后台相连接，并保持通信，所以我们的 `Online Browsers` 多了一台主机。

![beef-hooked-view.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111428958.png/wm)

我们返回刚刚的钓鱼页面，并通过 F12 查看页面的相关信息，切换至网络的选项，我们可以看到该页面中有一个 hook.js 在不断的向外 GET 与 POST 信息：

![beef-hooked-view.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111456901.png/wm)

我们还可以随便查看一条消息，知道其发送的具体内容：

![beef-demos-page-f12-detail.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111477115.png/wm)

当然我们同样还可以通过 wireshark 抓取 virbr0 网卡上的数据包查看更为详尽的内容。

只要页面不关闭，hook.js 不停止运行，就会不断的发送消息。

我们回到后台的管理页面中，点击主机的图标，我们可以看到关于该主机浏览器的详细信息便展示出来了：

![beef-browser-info-1.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111494821.png/wm)

![beef-browser-info-2.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111508978.png/wm)

在获取到相关的平台信息之后，相信大家最关心的还是如何去攻击目标，切换至 Commands 选项卡我们可以看到所有已经加载进来的攻击模块：

![beef-command.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111529759.png/wm)

随便打开一个文件夹我们便可看到每个攻击模块前面都有不同颜色的灯，不同的颜色代表着不同的意义，若是有认真查看 Getting started 页面的同学便知道各项是什么意思：

![beef-command-corlor-meaning.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111545947.png/wm)

### 6.2 BeEF 的初试

在了解了 BeEF 大概的框架结构之后我们可以来尝试一番，展开 `Browser`-->`Hooked Domain`，点击 `Get Page HTML` 模块，我们可以在用户未察觉的情况下获取目标当且访问的页面源码：

![beef-get-html.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111736174.png/wm)

在点击执行之后我们会查看到 `Module Results History` 中多了一项任务，点击该任务我们便可以看到用户正在查看页面的源码了：

![beef-get-html-info.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111756333.png/wm)

其实这样的实现并不困难，我们查看一下源码便知道，其中 Get Page HTML 的命令源码位于 `/usr/share/beef-xss/modules/browser/hooked_domain/get_page_html/command.js`。我们用 less 工具来查看一下

```
less /usr/share/beef-xss/modules/browser/hooked_domain/get_page_html/command.js
```

查看其中的相关命令：

![beef-getpagehtml-command.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111776751.png/wm)

那么调用的这两个函数又做了怎样的事情呢？我们用 less 来查看一下这个文件 `/usr/share/beef-xss/core/main/client/browser.js`，查找一下刚刚两个函数的名字（直接输入 /getPage 然后一个回车便可搜索）

![beef-get-html-code.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111790228.png/wm)

原理很简单，就是将这两个简单的命令发送给远程的浏览器，而远程的浏览器便调用 javascript 的解析器解析并执行这两个命令，这样就获取到了我们所需要的相关信息，

而能够做到这一切的前提是 hook.js 在客户端接收着我们发送的数据。

只要稍微懂一点 javascript 的同学就知道既然能够做到这一点，那么给目标主机的浏览器弹一个窗口也没有问题咯，的确是这样：

![beef-alert.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111805903.png/wm)

执行之后，我们查看之前让我们上钩的页面，我们可以看到：

![beef-alert-result.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111820810.png/wm)

通过 `less /usr/share/beef-xss/modules/browser/hooked_domain/alert_dialog/command.js` 我们可以看到，这个的实现更简单，就是让远程调用 alert 函数即可。

我们还可以让页面重定向：

![beef-redirect.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111843790.png/wm)

执行之后我们会发现页面跳转到了 localhost 的页面上了：

![beef-redirect-result.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111863072.png/wm)

你不会以为 BeEF 中的攻击模块都这么无用吗？不是的，我们还可以使用这样一个模块：

![beef-start-pretty.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111879121.png/wm)

该模块会弹出一个与 Facebook 登陆窗口一模一样的窗口，让你误以为你掉线了，需要重新登陆，此时若是你输入了用户名、密码，那么攻击者就真的获取到了你的登陆名与密码。

执行之后的效果：

![beef-pretty-facebook.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111895582.png/wm)

若是输入了用户名与密码：

![beef-pretty-login.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111909274.png/wm)

返回我们的后台页面，我们可以看到，目标用户的登录名与密码就这么轻易的被我们获取到了。

![beef-pretty-result.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111926689.png/wm)

同样我们可以查看源码 `less /usr/share/beef-xss/modules/social_engineering/pretty_theft/command.js`，原理并不是多么的高深：

通过 DOM 执行一些 HTML 的语句，来创造出登陆时的界面：

![beef-pretty-html.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111946038.png/wm)

然后等待用户将用户名与密码提交之后将结果返回回来：

![beef-pretty-getusernamepassword.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111960370.png/wm)

这就是钓鱼网站的骗局，弄出一个与官网一模一样的界面，让你误以为就是官网，从而骗取的登陆信息等等。原理并不复杂，代码也不多，这里有这么多是因为模拟了 7 个登陆界面，还有做一些判断在里面。

### 6.3 BeEF 的联合

通过前面的介绍我们知道 metasploit 是一套非常强大的工具，所以 Kali 将其收纳了进来，同时 BeEF 也是 xss 的一大神器，若是能够强强联手那就更厉害了，使得攻击面进一步的扩大，能做的事情更多。

本环节将带领大家让 BeEF 加载 metasploit 相关的模块，但是由于环境的限制，BeEF 虽然能够正常的加载 metasploit 的攻击模块，但是在的 Command 加载的时候会卡住，若是有兴趣的同学可以在本地的环境中尝试一番。metasploit 的联合甚至可以攻破加载了 hook.js 的目标用户 shell，从而使得我们能够远程登陆操控。

我们只需要做这样的一些配置即可：

1.修改 BeEF 配置，开启 metasploit

BeEF 的配置位于其主目录中的 config.yaml 文件中：

```
vim /usr/share/beef-xss/config.yaml
```

我们只需要修改这样的一些内容：

```
#原文

metasploit:
    enable: false

#修改后

metasploit:
    enable: true
```

开启在 BeEF 启动时对 metasploit 相关模块的加载

2.修改 metasploit 扩展模块的配置

需要更新位于扩展中 metasploit 模块的配置：

```
vim /usr/share/beef-xss/extensions/metasploit/config.yaml
```

做这样的一些修改：

```
#原文

{os: 'custom', path: ''}

#修改后

{os: 'custom', path: '/usr/share/metasploit-framework/'}

```

同时我们可以在配置文件中看到 user、pass 的配置，这样配置的 BeEF 与 metasploit 连接时之间的认证，建议不要修改。

修改完成之后我们保存并退出，然后重启 postgresql（`service postgresql restart`），接着启动 metasploit（`msfconsole`），启动 metasploit 的 console 页面之后我们需要使其加载与 BeEF 相连接的模块：

```
msf > load msgrpc ServerHost=127.0.0.1 Pass=abc123
```

![load-msgrpc.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111979257.png/wm)

此时我们再次启动 BeEF，我们可以看到加载信息中有 metasploit 的相关模块的加载：

![star-with-metasploit.png](https://doc.shiyanlou.com/document-uid113508labid2398timestamp1482111997482.png/wm)

这样我们便实现了 BeEF 与 metasploit 的强强联合，如此我们就可以利用 XSS 做更深入、更自动化的渗透了。但是由于我们环境的原因，在 BeEF 后台加载 metasploit 相关 Command 的时候会一直连接不上，有兴趣的同学可以在本地尝试一下。

## 7. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

- BeEF 的大体实现结构
- BeEF 的所属攻击类型
- BeEF 的网页源码窃取
- BeEF 的弹窗提示
- BeEF 的钓鱼登陆窃密实现

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

## 8. 作业

1. 本地中 BeEF 与 metasploit 的联合尝试。