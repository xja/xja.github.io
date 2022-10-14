---
title: "DRIVE_LAYOUT_INFORMATION_EX 中的 PartitionCount 分区数量"
date: 2022-10-14T22:10:31+08:00
draft: false
categories: ["storage"]
tags: ["windows", "mbr", "winioctl"]
---

微软文档中，结构体 `DRIVE_LAYOUT_INFORMATION_EX` 的[定义](https://docs.microsoft.com/en-us/windows/win32/api/winioctl/ns-winioctl-drive_layout_information_ex)如下

```C
typedef struct _DRIVE_LAYOUT_INFORMATION_EX {
  DWORD                    PartitionStyle;
  DWORD                    PartitionCount;
  union {
    DRIVE_LAYOUT_INFORMATION_MBR Mbr;
    DRIVE_LAYOUT_INFORMATION_GPT Gpt;
  } DUMMYUNIONNAME;
  PARTITION_INFORMATION_EX PartitionEntry[1];
} DRIVE_LAYOUT_INFORMATION_EX, *PDRIVE_LAYOUT_INFORMATION_EX;
```

字段 `PartitionCount` 的描述为：
> The number of partitions on the drive. On hard disks with the MBR layout, this value will always be a multiple of 4. Any partitions that are actually unused will have a partition type of PARTITION_ENTRY_UNUSED (0) set in the PartitionType member of the PARTITION_INFORMATION_MBR structure of the Mbr member of the PARTITION_INFORMATION_EX structure of the PartitionEntry member of this structure.

简单翻译如下：硬盘上的分区数量。在 MBR 形式的硬盘上，这个值始终为 4 的倍数。未使用的分区的分区类型为 `PARTITION_ENTRY_UNUSED`(其值为 0)，这个值存储在当前结构体下的 `PARTITION_INFORMATION_EX` 结构体成员 `PartitionEntry` 中 `PARTITION_INFORMATION_MBR` 结构体成员  `Mbr` 的  `PartitionType` 成员中。其完整调用形式如下所示

```C
layout.PartitionEntry[index].Mbr.PartitionType
```

*`PartitionType` 的对照表见参考链接 2*

此处便引出了一个问题，`PartitionCount` 的值始终为 4 的倍数应该怎么理解？

首先说一下我的结论：

$PartitionCount=4+Extended*4+(Logical-1)*4$

MBR 限制最多只能有 4 个主分区（Primary）或者 3 个主分区加 1 个扩展分区（Extended），当需要更多分区时就必须在扩展分区下创建逻辑分区（Logical），每个逻辑分区都有一个和MBR结构类似的扩展引导记录（EBR）。

|主分区数量|扩展分区数量|逻辑分区数量|PartitionCount|
|:-:|:-:|:-:|:-:|
|1|0|0|4|
|4|0|0|4|
|1|1|0|8|
|3|1|0|8|
|1|1|1|8|
|1|1|5|24|

上表便很好地说明了 `PartitionCount` 的含义。因此，当磁盘为 MBR 格式时，通过该参数能简单地辨别磁盘上是否有扩展分区。

## 参考链接

1. [【Linux】MBR磁盘分区表只能有四个分区？_remo0x的博客-CSDN博客_mbr为什么只能建四个主分区](https://blog.csdn.net/White_Idiot/article/details/80088115)
2. [List of partition identifiers for PCs](https://www.win.tue.nl/~aeb/partitions/partition_types-1.html)
