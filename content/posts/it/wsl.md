---
title: "Wsl问题"
date: 2021-02-08T15:11:34+08:00
tags: ["WSL"]

---

#### 解决使用VPN（Cisco AnyConnect）之后，wsl无法使用网络的问题

github链接：https://github.com/microsoft/WSL/issues/5068

有用的一招

![image-20210208151430565](https://i.loli.net/2021/02/08/jzYtx6UDiNTZhX5.png)

```shell
Get-NetAdapter | Where-Object {$_.InterfaceDescription -Match "Cisco AnyConnect"} | Set-NetIPInterface -InterfaceMetric 6000
Get-NetIPInterface -InterfaceAlias "vEthernet (WSL)" | Set-NetIPInterface -InterfaceMetric 1
```

**备注：每次VPN启动之后需要设置下**

Get-NetAdapter | Where-Object {$_.InterfaceDescription -Match "Cisco AnyConnect"} | Set-NetIPInterface -InterfaceMetric 6000

像是提高VPN 的IP接口的权重值

**VPN关闭之后，`ping baidu.com`又不好使了**

1. 编辑`/etc/wsl.conf`文件

```ini
[network]
generateResolvConf = false
```

2. 编辑`/etc/resolv.conf`

```ini
nameserver 8.8.8.8
```

ping通了