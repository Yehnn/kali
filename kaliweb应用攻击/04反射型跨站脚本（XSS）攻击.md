# 反射型跨站脚本（XSS）攻击

## 1. 课程说明

本课程为动手实验教程，为了能说清楚实验中的一些操作会加入理论内容，也会精选最值得读的文章推荐给你，在动手实践的同时扎实理论基础。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

## 2. 学习方法

实验楼的 Kali 系列课程包含五个训练营，本训练营主要讲述 Web 应用攻击方法。课程包含20个实验，每个实验都提供详细的步骤和截图。适用于有一定 Linux 系统基础，想快速上手 Kali 渗透测试的同学。

学习方法是多实践，多提问。启动实验后按照实验步骤逐步操作，同时理解每一步的详细内容。

如果实验开始部分有推荐阅读的材料，请务必先阅读后再继续实验，理论知识是实践必要的基础。

## 3. 本节内容简介

本实验中我们初步接触 Kali Linux 和渗透测试的概念。需要依次完成下面几项任务：

- 反射型 XSS 的简介
- 反射型 XSS 的实现
- 反射型 XSS 的防范

## 4. 推荐阅读

本节实验推荐阅读下述内容：

1. [xss 简介](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS))
2. [session 管理](https://www.owasp.org/index.php/Session_Management_Cheat_Sheet)

## 5. 反射型 XSS

### 5.1 反射型 XSS 简介

在上一节试验中我们了解到什么是 XSS 攻击，并且了解到 XSS 攻击的具体分类。

那么什么是反射型 XSS 呢？

反射型 XSS （Reflected xss）又称为非持久型 xss，非持久的意思是这种方式的 javascript 注入并没有修改到服务器上，其他用户访问该页面并不会受到影响，但是通过攻击者提供的特定 URL 就会“遭殃”。

也就是通过 URL 的修改，能够执行一些特殊的命令，从而获取我们所需要的信息

### 5.2 DVMA 简介

这样简单的文字定义与讲解很抽象，我们并不能真正的理解到底是什么意思，所以我们会通过 DVMA 这样的一个平台来学习 xss。

DVMA 是 Damn Vulnerable Web App 的简称，它是一个专门用于 web 渗透方式的学习平台，上面提供了特定的漏洞供我们测试、理解相关攻击的方式与原理。

在我们的实验环境中已经为大家提供好了该平台，只需要启动 Metasploitable2 的虚拟机即可。

在实验桌面中，双击 Xfce 终端，打开终端：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2290timestamp1479374588595.png/wm)

使用 `sudo virsh start Metasploitable2` 命令即可启动我们的靶机系统虚拟机：

![start-metasploit.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139532596.png/wm)

稍等片刻，待得虚拟机完全启动之后我们打开桌面上的 Firefox：

![open-firefox.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139552463.png/wm)

访问我们的靶机系统所使用的 IP 地址`192.168.122.102`：

![view-metasploit-url.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139568548.png/wm)

正常的启动靶机系统之后，我们访问其 IP 地址可以得到这样的一个页面。

其中提供了这样一些内容：

- TWiki：一个灵活强大的 wiki 系统
- phpMyAdmin：因为 DVWA 平台使用的是 PHP/MySQL 的架构
- Mutillidae：与 DVWA 类似，通过了 10 大典型的 web 应用漏洞
- DVWA：提供典型的 web 漏洞平台

点击 DVMA 我们便可进入到 DVMA 的登陆页面，默认的登陆用户与密码是 admin 与 password，登陆之后便会进入这样的页面：

![dvwa-index.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139587112.png/wm)

从页面上我们可以看到，该平台为我们提供了所有常见的 web 漏洞攻击方式：

![dvwa-info.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139626005.png/wm)

并且我们还可以通过调整不同的安全级别，让我们不仅能方便我们对攻击方式原理的理解，同时还能让我们连接对应攻击方式的防范措施。

首先为了能够进行最简单的攻击，我们会把安全默认调制最低，首先进入安全模式的调整页面：

![dvwa-config-security.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139662479.png/wm)

然后调整安全的 level 到 low：

![dvwa-config-security-1.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139678741.png/wm)

当看到页面的下方 Level 的显示变化后，说明修改成功了：

![dvwa-config-security-proof.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139693059.png/wm)

## 6. DVWA 的实战

我们选择 XSS reflected 的选型，我们就会进入到 DVWA 为我们提供的反射型 XSS 的攻击平台：

![dvwa-reflected.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139718790.png/wm)

这样的场景就像我们在上论坛、电商、教务网等等的时候，我们在修改信息、登陆、判断时填写表单然后提交一样。

我们在文本框中输入任意字符，然后点击提交：

![dvwa-reflected-test-1.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139739967.png/wm)

提交之后我们会看到这样的变化：

![dvwa-reflected-test-change.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139751458.png/wm)

这是一个很简单的功能，就是我们输入了用户名，后台的 PHP 程序将我们的输入存储在一个变量之中，然后将该变量提交至后台，后台读取该变量的值，然后输出。

从 URL 的变化我们可以看出使用了一个 name 的变量，那我们若是手动的更改 URL 上 name 变量的值：

![dvwa-reflected-change-get](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139813754.png/wm)

敲回车以后我们会发现，页面中的输出的内容发生了变化：

![dvwa-reflected-get-change](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139837396.png/wm)

在下方我们有一个查看源码的按钮，我们可以查看一下这个功能是如何实现的：

![dvwa-reflected-source](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139862395.png/wm)

我们可以看到这段代码，就是做了一个简单的判断，输入框中是否有值，没有值就设置判断是否为空的变量 $isempty 为真，然后不做任何的处理，若是有值就输出 hello 与我们输入的内容。

这也就以为者，这段 php 代码没有做任何的过滤，只要我们输入的值不为空，它都会输出，都会执行，那我们说是输入一段 javascript 的代码呢？

我们在输入框中输入这样的内容：

```
<script>alert("nihao,shiyanlou")</script>
```

![dvwa-reflected-js-alert](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139882293.png/wm)

然后我们再次提交，我们可以看到此时我们的页面会有弹窗出来：

![dvwa-reflected-alert-proof](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139895031.png/wm)

这就是页面执行了我们注入的恶意 javascript 代码，我们可以看到 URL 的变化，我们若是复制该 URL 给其他人使用，会得到与刚刚同样的结果。可以尝试一下。

这就是反射型 XSS，这样黑客们发现这样的漏洞，就会将这样恶意的 URL 四处传播，让人上当。当然攻击者不会是弹窗这样简单的代码了。

那反射型 XSS 是如何实现的呢？

我们可以再次查看源代码：

在判断变量不是空之后，他的做法是：

```
 echo '<pre>';
 echo 'Hello ' . $_GET['name'];
 echo '</pre>'; 
```

之前使用 echo 来输出 HTML 的相关标签与变量。而浏览器解析到这是 HTML 代码，就以 HTML 代码的格式展现，解析到 script 标签就以 javascript 的方式解析、运行。

所以当我们输入的内容是 `<script>alert("nihao,shiyanlou")</script>` 时，后台的 php 的代码就变成这样：

```
 echo '<pre>';
 echo 'Hello ' <script>alert("nihao,shiyanlou")</script>;
 echo '</pre>'; 
```

去除 php 代码的语法之后代码是：

```
<pre>
Hello <script>alert("nihao,shiyanlou")</script>
</pre>
```

我们可以通过 F12 查看此时的页面代码（若是查看器中没有内容，记得停止页面的加载）

这样停止页面的加载：

![stop-load-page.png](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139912352.png/wm)

然后找到此时弹窗的代码：

![dvwa-reflected-js-source-proof](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139931342.png/wm)

这就是反射型 XSS 的原理，我们还能通过这样的漏洞获取此时用户的 cookie 值，在文本框中输入 `<script>alert(document.cookie)</script>`，然后提交：

![dvwa-reflected-get-cookie](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139942932.png/wm)

通过 cookie 自动登陆，使得我们在获取到用户的 cookie 值之后便可实现中间人攻击，仿冒用户登陆，进而查看更多的私密信息，设置用此获利。

![reflected-cross-site-scripting.jpg](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139956321.png/wm)
（此图来自于 [hackingloops](https://www.hackingloops.com/how-to-test-reflected-cross-site-scripting-vulnerability/)）

>**中间人攻击**：中间人攻击（Man-in-the-MiddleAttack，简称“ MITM 攻击”）中间人攻击很早就成为了黑客常用的一种古老的攻击手段，并且一直到今天还具有极大的扩展空间。
>在网络安全方面，MITM 攻击的使用是很广泛的，曾经猖獗一时的 SMB 会话劫持、DNS 欺骗等技术都是典型的 MITM 攻击手段。在黑客技术越来越多的运用于以获取经济利益为目标的情况下时，MITM 攻击成为对网银、网游、网上交易等最有威胁并且最具破坏性的一种攻击方式。
>简而言之，所谓的 MITM 攻击就是通过拦截正常的网络通信数据，并进行数据篡改和嗅探，而通信的双方却毫不知情。（来自于百度百科）

## 7. 反射型 XSS 的防范

从上文我们可以看到，这里 xss 漏洞的最根本原因就是客户端可以执行任意的 javascript 的代码，所以若是需要防范这样的漏洞就是防止这样的情况出现。

我们修改我们的安全级别到中等：

![change-script-level](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482139986221.png/wm)

然后我们再次尝试我们上述弹窗的方法：

![dvwa-mid-test](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140000574.png/wm)

我们会发现不起作用了，得到的是这样的结果：

![dvwa-mid-test-proof](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140016543.png/wm)

似乎我们输入的 `<script>` 标签被替换了，使得浏览器无法将其便认为 javascript 的代码。

我们可以查看一下源代码：

![dvwa-mid-code](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140029886.png/wm)

我们发现相对于之前的代码，有这样的变化：

```
echo 'Hello ' . str_replace('<script>', '', $_GET['name']);
```

代码中，在输出 name 变量之前使用 str_replace() 这个函数做了处理。

而我们通过 php 的使用手册中可以看到该函数的作用：

![code-meaning](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140043192.png/wm)

也就是说上述的代码的意思是搜索变量中的所有 `<script>` 字符串，并将其替换成空，由此便可以防范任意的 javascript 代码的执行了。所以我们得到的结果只是输出中间那一串字符。

但是这样的防范是不够的，因为我们知道 HTML 是一种弱类型的语言，稍微的一点语法错误也可以正常执行，并且没有区分大小写，所以我们只需要将刚刚代码中的 script 改成大写的 `<SCRIPT>` 又可以正常的执行了：

![dvwa-mid-daxie](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140062928.png/wm)

这是因为刚刚那个方法中的 str_replace() 是区分大小写的，所以没能过滤掉，而 HTML 的解析却可以正常的解析：

![dvwa-mid-daxie-proof](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140074717.png/wm)

难道这样的 XSS 漏洞就没有办法解决了吗？肯定是有的，我们将安全的级别调整至最高一级：

![change-script-level-high](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140088360.png/wm)

此时，我们再次尝试会发现，无论是大小写，浏览器都不会将其解析成 javascript 代码，而是直接将其输出：

![dvwa-high-show](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140104382.png/wm)

我们再来查看此时的源代码发生了怎样的变化：

![dvwa-high-source-code](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140120328.png/wm)

我们可以看到源码中把 str_replace() 切换成了 htmlspecialchars()。

而 htmlspecialchars() 的作用会把传入的变量中的所有特殊字符都进行转义，例如常用的特殊字符：

| 字符             | 转移后   |
| ---------------- | -------- |
| & (ampersand)    | `&amp;`  |
| ” (double quote) | `&quot;` |
| ‘ (single quote) | `&#039;` |
| < (less than)    | `&lt;  ` |
| > (greater than) | `&gt;  ` |
| /                | `&#47;`  |

而插入 javascript 代码离不开 `<script>` 标签，而标签的插入离不开 <（小于符号）、>（大于符号）。

通过这个函数的转义之后浏览器得到的代码是这样的：

```
&lt;SCRIPT&gt; alert(&quot;nihao,shiyanlou&quot;)&lt;&#47;SCRIPT&gt;
```

这样的代码，浏览其并不会将其解析成 javascript 的代码，同时浏览器会将这样的转义编码的对应字符输出，展示出的就是正常的内容，所以这样便可防止 `<script>` 标签的随意注入，又可以正常的显示内容。

这就是反射型 XSS:

- 作用：能够随意注入执行攻击者的 javascript 代码，从而获取 cookie，由此可仿冒用户登陆，查看敏感信息，实现中间人攻击；
- 原因：在使用 GET 提交变量时没有过滤变量的内容；
- 特点：通过 URL 来传播，因为 GET 变量会在 URL 中体现；

## 8. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎到[实验楼问答](https://www.shiyanlou.com/questions)与我们交流：

- 反射型 XSS 的简介
- 反射型 XSS 的实现
- 反射型 XSS 的防范

请务必保证自己能够动手完成整个实验，只看文字很简单，真正操作的时候会遇到各种各样的问题，解决问题的过程才是收获的过程。

## 9. 作业

1. 若是没有学习过 PHP 的同学，尝试中等级别的防御，能否获取到 cookie。
2. 若是学习过 PHP 的同学，可在本地尝试协议一个简单的表单提交，看是否会有这样的漏洞出现。
3. 使用 Mutillidae 实现反射型 XSS 攻击。

在访问靶机 IP 地址时，选择 Mutillidae 选项进入页面。XSS 的实验环境在：

![mutillidae-show](https://doc.shiyanlou.com/document-uid113508labid2407timestamp1482140201787.png/wm)

在该环境中实现反射型 xss 的攻击。