---
title: "Esxi Customizer Ps"
date: 2022-10-14T22:32:10+08:00
draft: false
categories: ["server"]
tags: ["h3c", "esxi"]
---

## 起因

H3C R2900 G3 安装 EXSi 6 时，因为没有集成网卡驱动而不能继续，而后在网上看了下，解决办法是向镜像添加驱动。

## 过程

0. 准备
   1. 如果是 Windows 7 系统，需要安装 [Windows Management Framework 3.0](https://www.microsoft.com/en-us/download/details.aspx?id=34595)
   2. 为了脚本能顺利运行，需要解除执行 ps1 脚本限制

        ```Powershell
        PS C:\Users\Administrator> Set-ExecutionPolicy Unrestricted

        执行策略更改
        执行策略可帮助你防止执行不信任的脚本。更改执行策略可能会产生安全风险，如 https:/go.microsoft.com/fwlink/?LinkID=135170
        中的 about_Execution_Policies 帮助主题所述。是否要更改执行策略?
        [Y] 是(Y)  [A] 全是(A)  [N] 否(N)  [L] 全否(L)  [S] 暂停(S)  [?] 帮助 (默认值为“N”): A
        ```

1. 从 [ESXi-Customizer-PS Github release](https://github.com/VFrontDe/ESXi-Customizer-PS/releases) 下载解压得到 `ESXi-Customizer-PS.ps1`
2. 从 [官方链接1](https://customerconnect.vmware.com/downloads/details?downloadGroup=PCLI650R1&productId=614) / [官方链接2](https://customerconnect.vmware.com/downloads/get-download?downloadGroup=PCLI650R1) 下载并安装 `VMware PowerCLI 6.5 Release 1`
3. 从硬件官网下载并解压驱动到文件夹，此处假定存放目录为 `NIC-Driver`
4. 打开 Powershell 切换到脚本所在目录，开始创建镜像

    ```Powershell
    PS C:\Users\Administrator> .\ESXi-Customizer-PS.ps1 -nsc -v67 -vft -pkgDir .\NIC-Driver
    ```

    参数说明

    - `-nsc`: -NoSignatureCheck，跳过签名检查，开启此选项避免无签名驱动添加失败
    - `-v67`: 指定镜像版本，可选项：`-v50`, `-v51`, `-v55`, `-v60`, `-v65`, `-v67`, `-v70`
    - `-vft`: 连接到 V-Front 在线仓库。如不指定则连接到 VMware ESXi 软件仓库
    - `-pkgDir <dir>[,...]`: 指定驱动的目录
  
完整帮助使用 `.\ESXi-Customizer-PS.ps1 -help` 查看
  
等待创建完成后，就可以将镜像写入 U 盘进行安装了。

## 参考

1. [VMware ESXi添加第三方网卡驱动_风行無痕的博客-CSDN博客_esxi添加网卡驱动 - CSDN](https://blog.csdn.net/gmaaa123/article/details/124892945)
