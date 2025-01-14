---
title: "zfs: couldn't save system state"
date: "2024-06-27"
categories: ["solve"]
---

bpool 空间不够了。

在 reddit 上找到了解决方案，用 [GitHub](https://github.com/Venomtek/zsysctl-manual-gc/) 上的脚本成功解决。以下引用自这个 [reddit](https://www.reddit.com/r/linuxquestions/comments/13jdzn3/how_to_maintain_20_minimum_free_space_on_bpool/)。

尽管 zfs 只是 storing the differences，但是每个 state 都需要大约 100MB 的新存储空间。
由于 zsys 的默认策略是保持至少 20 个状态，这将需要 20 \* 100MB = 2GB 的存储空间。这意味着我们要使用整个 boot 分区来进行快照。
还有一个解决方案是参考这篇 [博客](https://didrocks.fr/2020/06/04/zfs-focus-on-ubuntu-20.04-lts-zsys-state-collection/)，方法是手动编辑 `zsys.conf` 减少快照存储数量。
