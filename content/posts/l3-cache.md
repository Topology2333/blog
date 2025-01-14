---
title: "L3 cache mapping on Sandy Bridge CPUs, Mark Seaborn"
date: "2025-01-14"
categories: ["blogs"]
tags: ["Cache", "RowHammer"]
---

## 前言

出自这篇 [博客](https://lackingrhoticity.blogspot.com/2015/04/l3-cache-mapping-on-sandy-bridge-cpus.html), by Mark Seaborn, on Monday, 27 April 2015

## 简介

Sandy Bridge 处理器的 L3 缓存（三级缓存）是多个核心共享的，通常位于每个处理器模块内。每个模块包含两个核心，多个模块构成一个处理器。L3 缓存的大小通常为 3MB、6MB 或 8MB，根据处理器型号的不同而有所不同。  
在一些测试中，Sandy Bridge 的 L3 缓存使用的是分布式环形结构（NUCA, Non-Uniform Cache Architecture），不同核心之间可以共享缓存。由于这种架构，缓存访问的延迟会根据物理地址的映射和访问的核心而有所不同。

L3 缓存映射：Sandy Bridge 的 L3 缓存被划分为多个缓存切片，每个核心对应一个缓存切片，处理器通过物理地址哈希算法决定每个地址映射到哪个缓存切片。这一机制是对内存访问的优化，减少了不同核心之间访问共享缓存的延迟。  
行锤攻击：Sandy Bridge 的缓存架构和内存控制器特性可能被用于行锤攻击（Row Hammering）。在这种攻击中，攻击者可以通过频繁访问内存中的某些行，诱使 DRAM 出现位翻转，这可能导致数据损坏或安全漏洞。

> 然而我的老电脑 Intel Core i5-7200U 不属于这个结构，而是 [Kaby Lake](https://www.intel.com/content/www/us/en/products/sku/95443/intel-core-i57200u-processor-3m-cache-up-to-3-10-ghz/specifications.html)

## 原文概述

2013 年，一些研究人员逆向工程了 Intel Sandy Bridge CPU 如何将物理地址映射到 L3 缓存（最后一级缓存）中的缓存集合[^1]。他们对缓存映射感兴趣，因为它可以用来绕过内核的 ASLR[^2]。博客作者感兴趣的原因是，the cache mapping can be used to test whether cached memory accesses can do row hammering.

[^1]: https://ieeexplore.ieee.org/document/6547110?reload=true&arnumber=6547110
[^2]: [Address_space_layout_randomization](https://en.wikipedia.org/wiki/Address_space_layout_randomization)

### Some background

在 Sandy Bridge CPU 上，L3 缓存被划分为多个切片。物理地址通过哈希函数决定它们将存储在哪个 L3 缓存切片中。

L3 缓存是分布式的，并且基于环形结构。每个核心有一个切片，但 CPU 中的所有核心都可以通过环形总线访问所有的缓存切片，环形总线将所有核心及其缓存连接在一起。

当一个核心访问内存位置时，如果该位置映射到另一个核心的缓存切片上，访问速度会稍微变慢，因为需要绕过环形总线进行一到两次跳跃才能访问该位置。环形总线上使用的协议基于 QPI[^3][^4]

每个缓存切片包含 2048 个缓存集合。在低端 CPU 上，缓存集合是 12 路关联的，因此一个缓存切片的大小为 1.5MB（2048 个集合，12 路 每个缓存行 64 字节 = 1.5MB）；在高端 CPU 上，缓存集合是 16 路关联的，因此一个缓存切片的大小为 2MB。

### Cache mapping

研究人员（Hund 等人）发现，L3 缓存使用物理地址的位如下：

- **位 0-5**：这 6 位表示在 64 字节缓存行内的字节偏移。
- **位 6-16**：这 11 位表示缓存切片内的缓存集编号。
- **位 17-31**：这些位经过哈希运算，决定使用哪个缓存切片。
- **位 32 及以后**：未使用。

选择缓存切片的哈希函数如下：

- **在 4 核 CPU 上**，有 4 个缓存切片，因此切片号是 2 位。切片号的两个位分别是 h1 和 h2，其中：

  - **h1** 是物理地址位 18、19、21、23、25、27、29、30、31 的 XOR。
  - **h2** 是物理地址位 17、19、20、21、22、23、24、26、28、29、31 的 XOR。

- **在 2 核 CPU 上**，有 2 个缓存切片，因此切片号是 1 位。切片号是物理地址位 17、18、20、22、24、25、26、27、28、30 的 XOR。这等同于 **h1** 和 **h2** 的 XOR。（位 19、21、23、29 和 31 在 XOR 计算时会相互抵消，这部分是博客作者发现的内容）

[^3]: Intel 的 [QuickPath Interconnect](https://en.wikipedia.org/wiki/Intel_QuickPath_Interconnect)，其设计目标是替代之前的“前端总线”技术，以实现快速路径互连。QPI 是一种用于在高端多插槽系统中连接多个 CPU 的协议。后于 2017 年，在 Skylake-SP Xeon 平台上，QPI 被 Intel Ultra Path Interconnect（UPI）替代。
[^4]: 非常遗憾的是，我并没有找到 Intel Core i5-7200U 的相关说明。只找到了 i7 某些型号采用了 QPI 的文档 [Performance Analysis Guide for Intel® Core™ i7 Processor and Intel® Xeon™ 5500 processors](https://www.intel.com/content/dam/develop/external/us/en/documents/performance-analysis-guide-181827.pdf)，pg5 有配图，说明了不同 LLC 通过 QPI 的联系。（还找到一篇相关[博客](https://community.intel.com/t5/Intel-Moderncode-for-Parallel/Core-to-Core-Communication-Latency-in-Skylake-Kaby-Lake/m-p/1061658)，待阅读）

### Verifying the cache mapping

步骤如下：

- 选择 N 个物理内存地址，这些地址根据我们猜测的缓存映射应该映射到同一个缓存集。
- 使用 Linux 的 /proc/PID/pagemap 接口来确定我们可以访问哪些物理地址。
- 测量访问这些 N 个地址所需的时间。具体来说，程序首先访问前 N-1 个地址，然后测量访问第 N 个地址的时间。
- 程序针对多个 N 值进行测试。

如果正确猜测了缓存映射，那么，在具有 12 路缓存的 CPU 上，我们应该会看到在 N=13 时，内存访问时间大幅上升。这是因为，在 N=13 时，我们访问的内存位置已经不再适合 12 路缓存集，导致 L3 cache miss。内存访问时间将从 L3 缓存的延迟增加到 DRAM 的延迟。

> 注意： 这也假设缓存使用 LRU or Pseudo-LRU eviction policy（Sandy Bridge 使用的策略）。然而，Ivy Bridge 的 cache eviction policy 发生了变化。

如果我们猜错了缓存映射，内存访问时间将以 N 的较高值逐渐上升。在一个 2 缓存片 CPU 上，如果我们得到的地址到片散列函数错误，我们将看到访问时间达到 DRAM 延迟 N = 13 \* 2，平均，因为 N 个物理地址将分布在 2 个片上，所以在片上的 2 个缓存集溢出并产生缓存丢失之前，平均需要 13 \* 2 个地址。

### Ivy Bridge

这种 L3 缓存映射似乎同样适用于 Ivy Bridge 系列的 CPU。作者在配有 Ivy Bridge CPU 的机器上运行了相同的测试（2-core, 4-hyperthread），最初得到了相同的图形结果。然而，这些结果在该机器上并没有稳定复现。后续的测试显示，在 N<=12 时，内存访问时间更高。

这与报告一致，说明 Ivy Bridge 的 L3 缓存使用了 DIP (Dynamic Insertion Policy)[^5] 作为其 cache eviction policy，以避免 cache thrashing。DIP 会在 LRU 和 BIP 之间动态切换：LRU 更适用于较小的工作集（可以完全装入缓存），而 BIP 更适用于较大的工作集（无法完全装入缓存）。对于 N>12，作者的测试可能会产生足够的缓存未命中，从而导致缓存切换到 BIP 模式。这意味着测试 N 值的顺序可能会影响最终结果。

[^5]: 找到了一篇有关于 DIP 的 [论文](https://www.cs.cmu.edu/afs/cs/academic/class/15740-f18/www/papers/isca07-qureshi-dip.pdf) 以及 [PPT](https://www.eecg.utoronto.ca/~moshovos/000/lib/exe/fetch.php?media=wiki:aca2017:cache_insertion_policies.pptx)，还有 [博客](https://medium.com/@arpitguptarag/adaptive-insertion-policies-for-high-performance-caching-a741c52f515c)

### Thanks

Thanks to Yossef Oren for pointing me to the paper by Hund et al, which is referenced by the paper he coauthored, "The Spy in the Sandbox -- Practical Cache Attacks in Javascript" (Yossef Oren, Vasileios P. Kemerlis, Simha Sethumadhavan, Angelos D. Keromytis).

> 附原作者致谢

## [源码](https://github.com/google/rowhammer-test/blob/9a426c30ac2cc1bad0a6714e3e75e763bfdee4ea/cache_analysis/cache_test_physaddr.cc)阅读

### frame_number_from_pagemap, init_pagemap, get_physical_addr

```cpp
// Extract the physical page number from a Linux /proc/PID/pagemap entry.
uint64_t frame_number_from_pagemap(uint64_t value) {
  return value & ((1ULL << 54) - 1); // 保留低 54 位
}

void init_pagemap() {
  g_pagemap_fd = open("/proc/self/pagemap", O_RDONLY);
  assert(g_pagemap_fd >= 0);
}

uint64_t get_physical_addr(uintptr_t virtual_addr) {
  uint64_t value;
  off_t offset = (virtual_addr / page_size) * sizeof(value); // 页表偏移
  int got = pread(g_pagemap_fd, &value, sizeof(value), offset); // 读 8 个字节
  assert(got == 8);

  // Check the "page present" flag.
  assert(value & (1ULL << 63));

  uint64_t frame_num = frame_number_from_pagemap(value);
  return (frame_num * page_size) | (virtual_addr & (page_size - 1)); // 物理页号，偏移量
}
```

### get_cache_slice, in_same_cache_set

哈希相关缓存位，计算物理地址对应的 cache slice

```cpp
int get_cache_slice(uint64_t phys_addr, int bad_bit) {
  static const int bits[] = { 17, 18, 20, 22, 24, 25, 26, 27, 28, 30 };

  int count = sizeof(bits) / sizeof(bits[0]);
  int hash = 0;
  for (int i = 0; i < count; i++) {
    hash ^= (phys_addr >> bits[i]) & 1;
  }
  if (bad_bit != -1) {
    hash ^= (phys_addr >> bad_bit) & 1;
  }
  return hash;
}
```

检查两个物理地址是否属于相同的 cache set，对比低 17 位是否相等 && 所处的 cache slice 是否一样

```cpp
bool in_same_cache_set(uint64_t phys1, uint64_t phys2, int bad_bit) {
  uint64_t mask = ((uint64_t) 1 << 17) - 1;
  return ((phys1 & mask) == (phys2 & mask) && get_cache_slice(phys1, bad_bit) == get_cache_slice(phys2, bad_bit));
}
```

### time_access, timing

```cpp
// Execute a CPU memory barrier.  This is an attempt to prevent memory
// accesses from being reordered, in case reordering affects what gets
// evicted from the cache.  It's also an attempt to ensure we're
// measuring the time for a single memory access.
//
// However, this appears to be unnecessary on Sandy Bridge CPUs, since
// we get the same shape graph without this. （这是为什么呢？）
inline void mfence() {
  asm volatile("mfence");
}

// Measure the time taken to access the given address, in nanoseconds.
int time_access(uintptr_t ptr) {
  struct timespec ts0;
  int rc = clock_gettime(CLOCK_MONOTONIC, &ts0);
  assert(rc == 0);

  g_dummy += *(volatile int *) ptr;
  mfence();

  struct timespec ts;
  rc = clock_gettime(CLOCK_MONOTONIC, &ts);
  assert(rc == 0);
  return (ts.tv_sec - ts0.tv_sec) * 1000000000 + (ts.tv_nsec - ts0.tv_nsec); // 合成秒&纳秒差
}
```

> 关于单调时钟 CLOCK_MONOTONIC：A nonsettable system-wide clock that represents monotonic time since—as described by POSIX—"some unspecified point in the past". On Linux, that point corresponds to the number of seconds that the system has been running since it was booted.

---

测量多个内存地址的访问时间。通过在给定地址集上进行多次内存访问，测量缓存是否被命中，以及时间的变化。  
取到第一个物理地址之后，筛选所有和他在同一个 cache set 的物理地址。做 10 次测量，取中位数时间。

```cpp
int timing(int addr_count, int bad_bit) {
  size_t size = 16 << 20;  // 分配 16MB 内存
  uintptr_t buf = (uintptr_t) mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS | MAP_POPULATE, -1, 0);
  assert(buf);

  uintptr_t addrs[addr_count];
  addrs[0] = buf;
  uintptr_t phys1 = get_physical_addr(addrs[0]);

  uintptr_t next_addr = buf + page_size;
  uintptr_t end_addr = buf + size;
  int found = 1;
  while (found < addr_count) {
    uintptr_t addr = next_addr;
    next_addr += page_size;
    uint64_t phys2 = get_physical_addr(addr);
    if (in_same_cache_set(phys1, phys2, bad_bit)) {
      addrs[found] = addr;
      found++;
    }
  }

  int runs = 10;
  int times[runs];
  for (int run = 0; run < runs; run++) {
    g_dummy += *(volatile int *) addrs[0];
    mfence();
    for (int i = 1; i < addr_count; i++) { // 访问一轮
      g_dummy += *(volatile int *) addrs[i];
    }
    mfence();
    times[run] = time_access(addrs[0]); // 重新访问 addrs[0]
  }
  std::sort(times, &times[runs]);
  int median_time = times[runs / 2];

  int rc = munmap((void *) buf, size);
  assert(rc == 0);

  return median_time;
}
```

## TODO

- 添加本机测试
- 下一篇相关 [博客](https://blog.stuffedcow.net/2013/01/ivb-cache-replacement/)
