---
title: 基于KVM架构的VPS服务器搭建ss及锐速优化教程
subtitle: 
summary: 整理了这个SS搭建及优化的教程，适用于所有KVM技术的VPS，希望对喜欢折腾的朋友能起到一定的参考作用。
authors:
- admin
tags: ["VPN","Proxy"]
categories: ["Tools"]
date: "2016-10-25T00:00:00Z"
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

# 准备

- KVM架构虚拟服务器
- xshell

# 服务器

- 任意一家运营商的KVM架构VPS服务器
- Ubuntu 14.04 64bit系统
- 运行`apt-get install vim` 安装vim

# 搭建shadowsocks环境

使用`xshell`连接服务器主机

## 安装shadowsocks服务端

```
apt-get install python-pip
pip install shadowsocks
```

## 配置shadowsocks

- 用vim新建`shadowsocks.json`文件

  ```
  vim /etc/shadowsocks.json
  ```

- 复制以下内容进去

  ```
  {
      "server":"0.0.0.0",
      "local_address":"127.0.0.1",
      "local_port":1080,
      "port_password":{
          "10000":"Password1",
          "10001":"Password2",
          "10002":"Password3",
          "10003":"Password4"
          },
          "timeout": 300,
          "method":"rc4-md5",
          "fast_open": true
  }
  ```

“10000”是指端口，“Password1”是指此端口的密码，均可以随意设置

**常用 vim 操作自己百度，如果 vim 命令不可用是因为没安装 vim，可以用 vi 替代**

保存刚才的文档，然后启动 shadowsocks 服务（每次重启服务器后都必须再 次执行下面的命令） ：

```
ssserver -c /etc/shadowsocks.json -d start
```

# 锐速优化（可选）

锐速现在最低套餐是 300 元一年，新手不建议使用，如需使用百度锐速官网

## 更换内核

### 查询当前内核

- 输入以下命令查询当前内核

  ```
  uname -r
  ```

### 安装指定内核

- 目前锐速最高支持linux-image-3.13.0-46-generic内核，运行以下命令安装此内核

  ```
  apt-get install linux-image-3.13.0-46-generic
  ```

### 卸载其他内核

- 运行命令查询本系统的其他内核

  ```
  sudo dpkg --get-selections | grep linux-image
  ```

- **实例**，查询出有其他5个内核

  ```
  linux-image-3.16.0-30-generic
  linux-image-3.16.0-60-generic
  linux-image-extra-3.16.0-30-generic
  linux-image-extra-3.16.0-60-generic
  linux-image-generic-lts-utopic
  ```

- 运行命令卸载其他内核

  ```
  sudo apt-get remove linux-image-3.16.0-30-generic linux-image-3.16.0-60-generic linux-image-extra-3.16.0-30-generic linux-image-extra-3.16.0-60-generic linux-image-generic-lts-utopic
  ```

- 删除后执行grub更新和重启

  ```
  sudo update-grub
  sudo reboot now
  ```

## 固定内核版本

- 防止内核意外升级

  ```
  sudo apt-mark hold linux-image
  ```

## 优化内核

- 使用vim打开`limits.conf`

  ```
  vim /etc/security/limits.conf
  ```

  然后添加下面语句

  ```
  *    soft    nofile    51200
  *    hard    nofile    51200
  ```

- 修改`/etc/pam.d/common-session`,加入以下内容

  ```
  session required pam_limits.so
  ```

- 修改`/etc/profile`,最下面加入以下内容

  ```
  ulimit    -SHn    51200
  ```

- 修改`/etc/sysctl.conf`,加入以下内容

  ```
  fs.file-max = 51200
  net.core.rmem_max = 67108864
  net.core.wmem_max = 67108864
  net.core.netdev_max_backlog = 250000
  net.core.somaxconn = 4096
  net.ipv4.tcp_syncookies = 1
  net.ipv4.tcp_tw_reuse = 1
  net.ipv4.tcp_tw_recycle = 0
  net.ipv4.tcp_fin_timeout = 30
  net.ipv4.tcp_keepalive_time = 1200
  net.ipv4.ip_local_port_range = 10000 65000
  net.ipv4.tcp_max_syn_backlog = 8192
  net.ipv4.tcp_max_tw_buckets = 5000
  net.ipv4.tcp_rmem = 4096 87380 67108864
  net.ipv4.tcp_wmem = 4096 65536 67108864
  net.ipv4.tcp_mtu_probing = 1
  net.ipv4.tcp_congestion_control = hybla
  ```

**保存修改后执行 sysctl -p 使配置生效。 再额外使用一次 sudo reboot now 重启以生效。**

## 安装锐速

### 安装

- 输入以下命令安装锐速：

  ```
  wget http ://my.serverspeeder.com/d/ls/serverSpeederInstaller.tar.gz
  tar -xzvf serverSpeederInstaller.tar.gz
  sudo bash serverSpeederInstaller.sh
  ```

- 安装过程中依次输入以下命令：

  ```
  你的锐速邮箱
  你的锐速密码
  eth0
  1000000
  1000000
  0
  y
  y
  ```

### 优化锐速

- 打开`/serverspeeder/etc/config`，编辑如下内容：

  ```
  rsc="1"
  gso="1"
  maxmode="1"
  advinacc="1"
  ```

### 重启

- 重启锐速完成优化

  ```
  service serverSpeeder restart
  ```

# 开机启动

## 添加路径

- 在`/etc/init.d`目录下新建`ss_start`文件并加入如下内容：

  ```
  nohup /usr/local/bin/ss-server -c /etc/shadowsocks.json > /dev/null 2>&1 &
  ```

- 在`/etc/init.d`目录下新建`rs_start`文件并加入如下内容：

  ```
  /serverspeeder/bin/serverSpeeder.sh start
  ```

## 加权限

```
chmod +x /etc/init.d/ss_start
chmod +x /etc/init.d/rs_start
```

## 加自启

```
sudo update-rc.d ss_start defaults 91
sudo update-rc.d rs_start defaults 91
```

# 安全措施

- 关闭 ping 功能：

  根据我的经验，如果不关闭 ping，会经常有黑客试探攻击服务器，所以最好 关闭 ping 服务，只需要每次启动或者重启服务器后执行一行代码：

  ```
  echo "1" >/proc/sys/net/ipv4/icmp_echo_ignore_all
  ```

  至此，搭建+优化SS全过程已完毕。