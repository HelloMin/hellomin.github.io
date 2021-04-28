---
layout:     post
title:      CentOS局域网访问外网设置
subtitle:   
date:       2021-04-28
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Network
---

# 目标

局域网内有一台机器有外网IP，其他机器都只有局域网IP：168.168.0.xxx，目标是让所有局域网内的机器能通过这台唯一有外网IP的机器访问外网

# 步骤

### 不能访问外网的机器：设置网关为能访问外网的机器

查看网卡信息

```
[root@host ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    ...
2: em1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
   ...
```

修改网卡配置，配置文件地址

```
[root@host ~]# cd /etc/sysconfig/network-scripts
```

网卡配置文件为ifcfg-网卡名字

将GATEWAY字段IP改为能访问外网的机器的IP

```
[root@host network-scripts]# cat ifcfg-em1
...
IPADDR=168.168.0.1
GATEWAY=168.168.1.0
NETMASK=255.255.0.0
```

更新网卡信息

```
ifup ifcfg-em1
```



### 能访问外网的机器：打开转发功能

开启 IP 路由转发

```
[root@host ~]# echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
[root@host ~]# sysctl -p
net.ipv4.ip_forward = 1
```

添加IPTABLE RULE，开启NAT

```
[root@host ~]# iptables -t nat -F    # 清除原有的 NAT 表中的规则
[root@host ~]# iptables -F    # 清除原有的 filter 表中的规则
[root@host ~]# iptables -P FORWARD ACCEPT    # 缺省允许 IP 转发
# 利用 iptables 实现 NAT MASQUERADE 共享上网，此处 em1 需要是能够访问外部网络的网卡接口
[root@host ~]# iptables -t nat -A POSTROUTING -o em1 -j MASQUERADE
```



参考：

https://ccie.lol/knowledge-base/linux-centos-route-forwarding/