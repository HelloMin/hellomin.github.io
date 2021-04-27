---
layout:     post
title:      K8S failed to find subsystem mount for required subsystem pids
subtitle:   
date:       2021-04-27
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Kubernetes
    - TroubleShooting

---

# 问题表现

kubectl登录container或者init新的pod出现异常，此时检查node状态发现一个master节点状态为NotReady

```shell
[root@host path]$ kubectl get nodes
NAME       STATUS      ROLES    AGE    VERSION
...
hostname   NotReady    master   264d   v1.18.6
```

此时查看node 的log，发现这个master节点反复start kubelet

```
[root@host path]$ kubectl describe nodes
...
Events:
  Type    Reason                   Age    From                                   Message
  ----    ------                   ----   ----                                   -------
  Normal  Starting                 60m    kubelet, hostname  Starting kubelet.
  Normal  NodeHasSufficientMemory  60m    kubelet, hostname  Node hostname status is now: NodeHasSufficientMemory
  Normal  NodeHasNoDiskPressure    60m    kubelet, hostname  Node hostname status is now: NodeHasNoDiskPressure
  Normal  NodeHasSufficientPID     60m    kubelet, hostname  Node hostname status is now: NodeHasSufficientPID
  Normal  Starting                 59m    kubelet, hostname  Starting kubelet.
  Normal  NodeHasSufficientMemory  59m    kubelet, hostname  Node hostname status is now: NodeHasSufficientMemory
```

登录这台host，查看kubectl的log：/var/log/messages

```
Apr 27 06:27:36 host kubelet: F0427 06:27:36.540622    8344 kubelet.go:1384] Failed to start ContainerManager failed to initialize top level QOS containers: failed to update top level Burstable QOS cgroup : failed to set supported cgroup subsystems for cgroup [kubepods burstable]: failed to find subsystem mount for required subsystem: pids
```

搜索之后怀疑是cgroup不支持pid。查看linux是否支持cgroup pid

支持的host输出如下

```
[root@eanshlt2kvm20 kubernetes]# cat /proc/cgroups
#subsys_name    hierarchy       num_cgroups     enabled
...
pids    6       139     1
...

[root@host kubernetes]# cat /boot/config-`uname -r` | grep CGROUP
...
CONFIG_CGROUP_PIDS=y
...
```

出问题的host不支持，此处不会出现这行 pids 和 CONFIG_CGROUP_PIDS=y

# 解决方式

找到K8S的配置文件，通过查看service状态可以看到路径是 /usr/lib/systemd/system/kubelet.service

```
[root@host log]# systemctl status kubelet.service
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
```

在路径下找到配置文件：

```
/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
```

在ExecStart上添加 `--feature-gates SupportPodPidsLimit=false --feature-gates SupportNodePidsLimit=false` 

再次查看kubelet启动日志，成功启动，node状态正常。

