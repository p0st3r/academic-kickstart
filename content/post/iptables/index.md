---
title: iptables防火墙的应用和SNAT/DNAT策略
subtitle: 
summary: 
authors:
- admin
tags: ["iptables"]
categories: ["Hack"]
date: "2017-04-12T00:00:00Z"
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

#  0x00 iptables简介

netfilter/iptables（简称为iptables）组成Linux平台下的包过滤防火墙，与大多数的Linux软件一样，这个包过滤防火墙是免费的，它可以代替昂贵的商业防火墙解决方案。

Linux防火墙体系主要工作在网络层，针对TCP/IP数据包实施过滤和限制，完成封包过滤、封包重定向和网络地址转换（NAT）等功能，属于典型的包过滤防火墙（也称网络层防火墙）。其基于内核编码实现，具有非常稳定的性能和高效率，因此被广泛的应用。

## 1. Netfilter和iptables的区别：

- Netfilter:指的是Linux内核中实现包过滤防火墙的内部结构，不以程序或文件的形式存在，属于“内核态”（KernelSpace，又称内核空间）的防火墙功能体系；
- Iptables：指的是用来管理Linux防火墙的命令程序，通常位于/sbin/iptables，属于“用户态”（UserSpace，又称用户空间）的防火墙管理体系；

所以其实iptables只是Linux防火墙的管理工具而已，位于/sbin/iptables。真正实现防火墙功能的是 netfilter，它是Linux内核中实现包过滤的内部结构。

## 2. 规则（rules)

规则（rules）其实就是网络管理员预定义的条件，规则一般的定义为“如果数据包头符合这样的条件，就这样处理这个数据包”。

规则存储在内核空间的信息 包过滤表中，这些规则分别指定了**源地址**、**目的地址**、**传输协议**（如TCP、UDP、ICMP）和**服务类型**（如HTTP、FTP和SMTP）等。当数据包与规则匹配时，iptables就根据规则所定义的方法来处理这些数据包，如**放行（accept）**、**拒绝（reject）**和**丢弃（drop）**等。

配置防火墙的主要工作就是添加、修改和删除这些规则。

## 3. 包过滤的工作层次：

主要是网络层，针对IP数据包。体现在对包内的IP地址、端口等信息的处理上。

![img](https://pic3.zhimg.com/80/v2-ac1c43e4005e585e8b039f6d0f654d76_720w.png)



## 3. iptables的表、链结构：

iptables作用：为包过滤机制的实现提供规则（或策略），通过各种不同的规则，告诉netfilter对来自某些源、前往某些目的或具有某些协议特征的数据包应该如何处理。

iptables内置了4个表，即**filter**表、**nat**表、**mangle**表和**raw表**，分别用于实现**包过滤**，**网络地址转换**、**包重构(修改)**和**数据跟踪处理**。 链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一 条或数条规则。

当一个数据包到达一个链时，iptables就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据 该条规则所定义的方法处理该数据包；否则iptables将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables就会根据该链预先定 义的默认策略来处理数据包。

### 规则链：

- 规则的作用：对数据包进行过滤或处理
- 链的作用：容纳各种防火墙规则
- 链的分类依据：处理数据包的不同时机

默认包括5种规则链

- INPUT：处理入站数据包
- OUTPUT：处理出站数据包
- FORWARD：处理转发数据包
- POSTROUTING链：在进行路由选择后处理数据包（对数据链进行源地址修改转换）
- PREROUTING链：在进行路由选择前处理数据包（做目标地址转换）

**INPUT**、**OUTPUT**链主要用在“主机型防火墙”中，即主要针对服务器本机进行保护的防火墙；而**FORWARD**、**PREROUTING=**、**POSTROUTING**链多用在“网络型防火墙”中。

### 规则表

- 表的作用：容纳各种规则链
- 表的划分依据：防火墙规则的作用相似

默认包括4个规则表：

- raw表：确定是否对该数据包进行状态跟踪；对应iptable_raw，表内包含两个链：OUTPUT、PREROUTING
- mangle表：为数据包的TOS（服务类型）、TTL（生命周期）值，或者为数据包设置Mark标记，以实现流量整形、策略路由等高级应用。其对应iptable_mangle，表内包含五个链：PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD
- nat表：修改数据包中的源、目标IP地址或端口；其对应的模块为iptable_nat，表内包括三个链：PREROUTING、POSTROUTING、OUTPUT
- filter表：确定是否放行该数据包（过滤）；其对应的内核模块为iptable_filter，表内包含三个链：INPUT、FORWARD、OUTPUT



![img](https://pic1.zhimg.com/80/v2-3716f456bd27f19d2b71c522b7e4a7d4_720w.png)

Iptables采用“表”和“链”的分层结构。注意一定要明白这些表和链的关系及作用。



## 4. 链、表的优先顺序

- 规则表之间的优先顺序

- - Raw=\=》mangle=\=》nat==》filter

- 规则链之间的顺序

- - 入站：PREROUTING==》INPUT
  - 出站：OUTPUT==》POSTROUTING
  - 转发：PREROUTING=\=》FORWARD==》POSTROUTIN

- 规则链内的匹配顺序

- - 按顺序依次检查，**匹配即停止**（LOG策略例外）
  - 若找不到相匹配的规则，则按该链的默认策略处理

# 0x01 编写防火墙规则

## 1. iptables **的基本语法、控制类型**

```text
iptables [ -t 表名] 选项 [链名] [条件] [ -j 控制类型] 
```

**注意事项：**

- 不指定表名时，-t 默认指filter表
- 不指定链名时，默认指表内的所有链
- 除非设置链的默认策略，否则必须指定匹配条件
- 选项、链名、控制类型使用大写字母，其余均为小写

## 2. 数据包的常见控制类型

>- ACCEPT：允许通过
>- DROP：直接丢弃，不给出任何回应
>- REJECT：拒绝通过，必要时会给出提示
>- LOG：在/var/log/messages文件中记录日志信息，然后传给下一条规则继续匹配（匹配即停止对LOG操作不起作用，因为LOG只是一种辅助动作，并没有真正的处理数据包）

## 3. iptables命令的管理控制选项

- -A 在指定链的末尾添加（append）一条新的规则 -D 删除（delete）指定链中的某一条规则，可以按规则序号和内容删除
- -I 在指定链中插入（insert）一条新的规则，默认在第一行添加-R 修改、替换（replace）指定链中的某一条规则，可以按规则序号和内容替换
- -L 列出（list）指定链中所有的规则进行查看
- -E 重命名用户定义的链，不改变链本身
- -F 清空（flush）
- -N 新建（new-chain）一条用户自己定义的规则链
- -X 删除指定表中用户自定义的规则链（delete-chain）
- -P 设置指定链的默认策略（policy）
- -Z 将所有表的所有链的字节和数据包计数器清零
- -n 使用数字形式（numeric）显示输出结果
- -v：以更详细的方式显示规则信息
- --line-numbers：查看规则时，显示规则的序号-X：删除自定义的规则链 **eg：将filter表中FORWARD链中的默认策略设为丢弃，OUTPUT链的默认策略设为允许**

## 4. 规则的匹配条件

- 通用匹配

- - 协议匹配: -p [协议名]

  - 地址匹配

  - - -s [源地址]
    - -d [目标地址]

  - 接口匹配

  - - -i [入站网卡]
    - -o [出站网卡]

- 隐含匹配

- - 端口匹配

  - - -sport [源端口]
    - -dport [目标端口]

  - TCP标记匹配：--tcp-flags [检查范围] [被设置的标记]

  - ICMP类型匹配：--icmp-type [ICMP类型]

- 显式匹配

- - 多端口匹配：-m multiport --sport | --dport [端口列表]
  - IP范围匹配：-m iprange --src-range [IP范围]
  - MAC地址匹配：-m mac --mac-range [MAC地址]
  - 状态匹配：-m state --state [连接状态]

### 常见通用匹配条件：

1. **协议匹配：-p [协议名]**

（eg：**tcp**、**udp**、**icmp**、**all**(针对所有IP数据包)），可用的协议类型存放于Linux系统的/etc/procotols文件中；

eg：丢弃通过icmp协议访问防火墙本机的数据包、允许转发经过防火墙的除icmp协议以外的数据包：

```text
[root@iptables ~]# iptables -I INPUT -p icmp -j DROP
[root@iptables ~]# iptables -A FORWARD -p ! icmp -j ACCEPT
```

【！】表示取反

**2. 地址匹配：-s [源地址]、 -d [目标地址]**

可以是IP地址、网段地址，但不建议使用主机名、域名地址，因为解析过程会影响效率

eg：拒绝转发源地址为192.168.10.100的数据、允许转发源地址位于192.168.1.0/24网段的数据：

```
[root@iptables ~]# iptables -A FORWARD -s 192.168.10.100 -j REJECT [root@iptables ~]# iptables -A FORWARD -s 192.168.1.0/24 -j ACCEPT 
```

当遇到小规模的网络扫描或攻击时，封IP地址是比较有效的方式。

eg：添加防火墙规则封锁来自172.16.16.0/24网段的频繁扫描、登录穷举等不良企图：

```
[root@iptables ~]# iptables -I INPUT -s 172.16.16.0/24 -j DROP [root@iptables ~]# iptables -I FORWARD -s 172.16.16.0/24 -j DROP
```

**3. 接口匹配**：**-i [入站网卡]、-o [出站网卡]**

eg：丢弃从外网接口eth0访问防火墙本机且源地址为私有地址的数据包：

```text
[root@iptables ~]# iptables -A INPUT -i eth0 -s 172.16.0.0/12 -j DROP
```

### 常用隐含匹配条件：

**1. 端口匹配**：**--sport [源端口]、--dport [目的端口]**

单个端口号或者以冒号“：”分隔的端口范围都是可以接受的，但不连续的多个端口不能采用这种方式。

eg：允许为网段192.168.1.0/24转发DNS查询数据包：

`[root@iptables ~]# iptables -A FORWARD -s 192.168.1.0/24 -p udp --dport 53 -j ACCEPT [root@iptables ~]# iptables -A FORWARD -d 192.168.1.0/24 -p udp --sport 53 -j ACCEPT`

eg：构建vsftpd服务器时，开放20、21端口，以及用于被动模式的端口范围24500~24600：

 `[root@iptables ~]# iptables -A INPUT -p tcp --dport 20:21 -j ACCEPT [root@iptables ~]# iptables -A INPUT -p tcp --dport 24500:24600 -j ACCEPT`

**2. TCP标记匹配**：**--tcp-flags 检查范围 被设置的标记**

针对协议为TCP、用来检查数据包的标记位（--tcp-flags）

“检查范围”指出需要检查数据包的哪几个标记，“被设置的标记”则明确匹配对应值为1的标记，多个标记之间以逗号进行分隔。

eg：拒绝从外网接口eth0直接访问防火墙本机的TCP请求，但允许其他主机发给防火墙的TCP等响应数据包请求：

 `[root@iptables ~]# iptables -P INPUT DROP [root@iptables ~]# iptables -I INPUT -i eth0 -p tcp --tcp-flags SYN,RST,ACK SYN -j DROP [root@iptables ~]# iptables -I INPUT -i eth0 -p tcp --tcp-flags ! --syn -j ACCEPT`

**3. ICMP类型匹配**：**--icmp-type ICMP类型**

ICMP类型使用字符串或数字代码表示：

- Echo-Request代码为8——表ICMP请求；
- Echo-Reply代码为0——ICMP回显；
- Destination-Unreachable代码为3——ICMP目标不可达；

eg：禁止从其他主机ping本机，但是允许本机ping其他主机：

`[root@iptables ~]# iptables -A INPUT -p icmp --icmp-type 8 -j DROP [root@iptables ~]# iptables -A INPUT -p icmp --icmp-type 0 -j ACCEPT [root@iptables ~]# iptables -A INPUT -p icmp --icmp-type 3 -j ACCEPT [root@iptables ~]# iptables -A INPUT -p icmp -j DROP `

### 常用的显式匹配条件

**1. 多端口匹配**：**-m multiport --sports [源端口列表]**

**-m multiport --dports [目的端口列表]**

eg：允许本机开放25、80、110、143端口，以便提供电子邮件服务：

```text
[root@iptables ~]# iptables -A INPUT -p tcp -m multiport --dport 25,80,110,143 -j ACCEPT
```

**2. IP范围匹配**：**-m iprange --src-range [IP范围]**

用来检查数据包的源地址、目标地址，其中IP范围采用“起始地址—结束地址”的形式：

eg：禁止转发源IP地址位于192.168.1.10与192.168.1.20之间的TCP数据包：

```text
[root@iptables ~]# iptables -A FORWARD -p tcp -m iprange --src-range 192.168.1.10-192.168.1.20 -j ACCEPT
```

**3. MAC地址匹配**：**-m mac --mac-source [MAC地址]**

因为MAC地址的局限性，此类匹配一般只适用于内部网络。

eg：根据MAC地址封锁主机，禁止其访问本机的任何应用：

```text
[root@iptables ~]# iptables -A INPUT -m mac --mac-source 00:01:02:03:04:cc -j DROP
```

**4. 状态匹配**：**-m state --state [连接状态]**

- 基于iptables的状态跟踪机制用来检查数据包的连接状态（State）
- 常见的连接状态包括NEW（与任何连接无关的）、ESTABLISHED（响应请求或者已建立连接的）、RELATED（与已有连接有相关性的，eg：FTP数据连接）。

eg：禁止转发与正常TCP连接无关的非“--syn”请求数据包（如伪造的一些网络攻击数据包）：

```text
[root@iptables ~]# iptables -A FORWARD -m state --state NEW -p tcp ! --syn -j DROP
```

eg：只开放本机的web服务（80端口），但对发给本机的TCP应答数据包予以放行，其他入站数据包均丢弃：

```text
[root@iptables ~]# iptables -I INPUT -p tcp -m multiport --dport 80 -j ACCEPT
[root@iptables ~]# iptables -I INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
[root@iptables ~]# iptables -P INPUT DROP
```

# 0x02 iptables防火墙规则的保存与恢复

1、保存iptables的规则，避免开机失效

```text
iptables-save > /etc/iptables 
```

2、编辑网卡，写入开机加载iptables规则

```text
vi  /etc/network/interfaces
```

3、 在网卡配置文件中写入加载 之前保存的规则文件 使其开机可以加载iptables的规则文件

```text
 iptables-restore < /etc/iptables 
```

# 0x03 SNAT和DNAT

SNAT是指在数据包从网卡发送出去的时候，把数据包中的源地址部分替换为指定的IP，这样，接收方就认为数据包的来源是被替换的那个IP的主机。

DNAT就是指数据包从网卡发送出去的时候，修改数据包中的目的IP，表现为如果你想访问A，可是因为网关做了DNAT，把所有访问A的数据包的目的IP全部修改为B，那么，你实际上访问的是B 因为，路由是按照目的地址来选择的。

因此DNAT是在PREROUTING链上来进行的，而SNAT是在数据包发送出去的时候才进行，所以是在POSTROUTING链上进行的。 通过SNAT和DNAT可以使内网和外网进行相互通讯。#

##SNAT策略概述

- SNAT策略的典型应用环境

SNAT策略的典型应用环境

​		局域网主机共享单个公网IP地址接入Internet

- SNAT策略的原理

  **源地址转换**（Source Network Address Translation）是linux防火墙的一种地址转换操作，也是iptables命令中的一种数据包控制类型，并根据指定条件修改数据包的源IP地址。

- 实验环境拓扑：

- 实验分析：

  - a：只开启路由转发，未做地址转换的情况：

    分析：

    - 从局域网PC机访问Internet的数据包经过网关转发后其源IP地址保持不变；
    - 当Internet中的主机收到这样的请求数据包后，响应数据包将无法正确返回，从而导致访问失败。

  - b：开启路由转发，并设置SNAT转换的情况：

    分析：

    - 局域网PC机访问Internet的数据包到达网关服务器时，会先进行路由选择；
    - 如果该数据包需要从外网接口eth0向外转发，则将其源IP地址192.168.10.2修改为网关的外网接口地址210.106.46.151，然后发送给目标主机。
  
  b访问方式的优点：Internet中的服务器并不知道局域网PC机的实际IP地址，中间的转换完全由网关主机完成，起到了保护内部网络的作用。

## SNAT策略的应用

前提条件：

- 局域网各主机正确设置IP地址/子网掩码
- 局域网各主机正确设置默认网关地址
- Linux网关支持IP路由转发

实现方法：

- 编写SNAT转换规则

SNAT共享固定IP地址上网：

- 实验环境描述：
  - Linux网关服务器两块网卡eth0：210.106.46.151连接Internet、eth1：192.168.10.1连接局域网，开启IP路由功能
  - 局域网PC机的默认网关设为192.168.10.1，并设置正确的DNS服务器。
  - 内网和外网分别新建客户机，分别指定对应的网关地址，在外网客户机上开启httpd服务，在内网客户机中访问httpd服务，最后查看httpd客户机的访问记录；
  - 要求：192.168.10.0/24网段的PC机能够通过共享方式正常访问internet。

- 实验步骤：

  1：打开网关的路由转发（IP转发是实现路由功能的关键所在）：

  打开路由转发的两种方式：

  - 永久打开（修改`/proc`文件系统中的`ip_forward`，当值为1时表示开启，为0表示关闭）：

  - 临时开启，临时生效

  2：正确设置SNAT策略（若要保持SNAT策略长期有效，应将相关命令写入rc.local中）：

	```text
	[root@localhost ~]# iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j SNAT --to-source 210.106.46.151
	```
	
	3：测试SNAT共享接入的结果：
	
	​	上诉操作完成后，使用局域网PC就可以正常访问Internet中的网站。
	
	​	对于被访问的网站服务器，在日志文件中将会记录以网关主机210.106.46.151访问。

**共享动态IP地址上网**：

- MASQUERADE —— 地址伪装
- 适用于外网IP地址非固定的情况
- 对于ADSL拨号连接，接口通常为ppp0、ppp1
- 将SNAT规则改为MASQUERADE即可

实例：

```text
[root@localhost ~]# iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o ppp0 -j MASQUERADE
```

如果网关使用固定的公网IP地址，建议选择SNAT策略而不是MASQUERADE策略，以减少不必要的系统开销。

##DNAT策略概述

- DNAT策略的原理：

**目标地址转换**，Destination Network Address Translation，是Linux防火墙的另一种地址转换操作，也是iptables命令中的一种数据包控制类型，其作用是根据指定条件修改数据包的目标IP地址、目标端口。

SNAT用来修改源IP地址，而DNAT用来修改目标IP地址、目标端口；SNAT只能用在NAT表的POSTROUTING链，而DNAT只能用在NAT表的PREROUTING链和OUTPUT链（或被其调用的链）中。