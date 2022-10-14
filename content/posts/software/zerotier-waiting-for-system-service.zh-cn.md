---
title: "解决 Waiting for ZeroTier system service 一直等待的问题"
date: 2022-10-14T22:21:52+08:00
draft: false
categories: ["software"]
tags: ["windows", "zerotier"]
---

## 解决办法

### 方法一

*仅供参考*
创建路径为 `C:\ProgramData\ZeroTier\One\local.conf` 内容为 `{}` 的文件并重启 `ZeroTierOneService` 服务
*需要验证此处仅创建空文件是否能解决该问题*

### 方法二

*此方法仍然需要验证，仅作为猜想供参考*
创建路径为 `C:\ProgramData\ZeroTier\One\zerotier-one.port` 内容为 `9993` 的文件并重启 `ZeroTierOneService` 服务，其中 `9993` 也可以为其它任意未被使用的端口号

## 问题详情

0. Windows 11 干净系统新装 ZeroTier 1.10.1 完成后通知区域右键一直提示 `Waiting for ZeroTier system service...`

1. 按照[此处](https://discuss.zerotier.com/t/the-dreaded-waiting-for-zerotier-system-service-on-windows-server-2019/6662)说明尝试无效，猜测配置文件问题

    ```cmd
    C:\Program Files (x86)\ZeroTier\One>zerotier-cli join 123456789abcdef0
    C:\ProgramData\ZeroTier\One\zerotier-one_x64.exe: missing port and zerotier-one.port not found in C:\ProgramData\ZeroTier\One
    ```

2. 根据[此处](https://docs.zerotier.com/zerotier/zerotier.conf/)文档打开 `C:\ProgramData\ZeroTier\One` 目录，并未找到文中提到的配置文件 `local.conf`
3. 依照方法一所述操作后，问题解决
4. 其实在尝试加入网络时的错误提示上就已经指明了 `zerotier-one.port` 未找到，但是我没仔细看也没反应过来那是一个文件。在创建 `local.conf` 时虽然没有端口相关参数，但是重启时软件能正常读取配置文件便会自动使用默认端口运行
