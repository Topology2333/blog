---
title: "some papers about rowhammer"
date: "2025-01-13"
categories: ["papers"]
tags: ["RowHammer"]
draft: true
---

阅读一些有关 [RowHammer](https://en.wikipedia.org/wiki/Row_hammer) 的文献。

## [Github Page](https://github.com/google/rowhammer-test)

运行测试需要在 x86 平台上，依赖 CLFLUSH 指令。测试通过随机选取内存地址反复访问来触发问题（需谨慎运行）

```bash
./make.sh
./rowhammer_test
```

### 主要函数

`HammerAddressesStandard` 传入两个地址范围，

```c
uint64_t HammerAddressesStandard(const std::pair<uint64_t, uint64_t>& first_range, const std::pair<uint64_t, uint64_t>second_range, uint64_t number_of_reads) {
  volatile uint64_t* first_pointer = reinterpret_cast<uint64_t*>(first_range.first);
  volatile uint64_t* second_pointer = reinterpret_cast<uint64_t*>(second_range.first);
  uint64_t sum = 0;

  while (number_of_reads-- > 0) {
    sum += first_pointer[0];
    sum += second_pointer[0];
    asm volatile(
        "clflush (%0);\n\t"
        "clflush (%1);\n\t"
        : : "r" (first_pointer), "r" (second_pointer) : "memory");
  }
  return sum;
}
```

### 其余函数

`HammerAllReachableRows` 传入 hammer 函数，hammer 次数。  
通过 SetupMapping 把

```cpp
void HammerAllReachableRows(HammerFunction *hammer, uint64_t number_of_reads)
{
    uint64_t mapping_size;
    void *mapping;
    SetupMapping(&mapping_size, &mapping);

    HammerAllReachablePages(1024 * 256, mapping, mapping_size, hammer, number_of_reads);
}
```

`SetupMapping`

```c
void SetupMapping(uint64_t *mapping_size, void **mapping)
{
    // *0.3
    *mapping_size = static_cast<uint64_t>((static_cast<double>(GetPhysicalMemorySize()) * fraction_of_physical_memory));

    *mapping = mmap(NULL, *mapping_size, PROT_READ | PROT_WRITE, MAP_POPULATE | MAP_ANONYMOUS | MAP_PRIVATE, -1, 0);
    assert(*mapping != (void *)-1);

    // Initialize the mapping so that the pages are non-empty.
    printf("[!] Initializing large memory mapping ...");
    for (uint64_t index = 0; index < *mapping_size; index += 0x1000)
    {
        uint64_t *temporary = reinterpret_cast<uint64_t *>(static_cast<uint8_t *>(*mapping) + index);
        temporary[0] = index;
    }
    printf("done\n");
}
```

```
RETURN VALUE
       On success, mmap() returns a pointer to the mapped area.  On error, the value MAP_FAILED (that is, (void *) -1) is returned, and errno is set to indicate the cause of the error.
```

`HammerAllReachablePages`

```cpp
// A comprehensive test that attempts to hammer adjacent rows for a given
// assumed row size (and assumptions of sequential physical addresses for
// various rows.
uint64_t HammerAllReachablePages(uint64_t presumed_row_size, // 1024 * 256
                                    void *memory_mapping, uint64_t memory_mapping_size, HammerFunction *hammer,
                                    uint64_t number_of_reads)
{
    // This vector will be filled with all the pages we can get access to for a
    // given row size.
    std::vector<std::vector<uint8_t *>> pages_per_row;
    uint64_t total_bitflips = 0;

    pages_per_row.resize(memory_mapping_size / presumed_row_size);
    int pagemap = open("/proc/self/pagemap", O_RDONLY);
    assert(pagemap >= 0);

    printf("[!] Identifying rows for accessible pages ... ");
    for (uint64_t offset = 0; offset < memory_mapping_size; offset += 0x1000)
    {
        uint8_t *virtual_address = static_cast<uint8_t *>(memory_mapping) + offset;
        uint64_t page_frame_number = GetPageFrameNumber(pagemap, virtual_address);
        uint64_t physical_address = page_frame_number * 0x1000;
        uint64_t presumed_row_index = physical_address / presumed_row_size;
        // printf("[!] put va %lx pa %lx into row %ld\n", (uint64_t)virtual_address,
        //     physical_address, presumed_row_index);
        if (presumed_row_index > pages_per_row.size())
        {
            pages_per_row.resize(presumed_row_index);
        }
        pages_per_row[presumed_row_index].push_back(virtual_address);
        // printf("[!] done\n");
    }
    printf("Done\n");

    // We should have some pages for most rows now.
    for (uint64_t row_index = 0; row_index + 2 < pages_per_row.size();
            ++row_index)
    {
        if ((pages_per_row[row_index].size() != 64) ||
            (pages_per_row[row_index + 2].size() != 64))
        {
            printf("[!] Can't hammer row %ld - only got %ld/%ld pages "
                    "in the rows above/below\n",
                    row_index + 1, pages_per_row[row_index].size(),
                    pages_per_row[row_index + 2].size());
            continue;
        }
        else if (pages_per_row[row_index + 1].size() == 0)
        {
            printf("[!] Can't hammer row %ld, got no pages from that row\n",
                    row_index + 1);
            continue;
        }
        printf("[!] Hammering rows %ld/%ld/%ld of %ld (got %ld/%ld/%ld pages)\n",
                row_index, row_index + 1, row_index + 2, pages_per_row.size(),
                pages_per_row[row_index].size(), pages_per_row[row_index + 1].size(),
                pages_per_row[row_index + 2].size());
        // Iterate over all pages we have for the first row.
        for (uint8_t *first_row_page : pages_per_row[row_index])
        {
            // Iterate over all pages we have for the second row.
            for (uint8_t *second_row_page : pages_per_row[row_index + 2])
            {
                // Set all the target pages to 0xFF.
                for (uint8_t *target_page : pages_per_row[row_index + 1])
                {
                    memset(target_page, 0xFF, 0x1000);
                }
                // Now hammer the two pages we care about.
                std::pair<uint64_t, uint64_t> first_page_range(
                    reinterpret_cast<uint64_t>(first_row_page),
                    reinterpret_cast<uint64_t>(first_row_page + 0x1000));
                std::pair<uint64_t, uint64_t> second_page_range(
                    reinterpret_cast<uint64_t>(second_row_page),
                    reinterpret_cast<uint64_t>(second_row_page + 0x1000));
                hammer(first_page_range, second_page_range, number_of_reads);
                // Now check the target pages.
                uint64_t number_of_bitflips_in_target = 0;
                for (const uint8_t *target_page : pages_per_row[row_index + 1])
                {
                    for (uint32_t index = 0; index < 0x1000; ++index)
                    {
                        if (target_page[index] != 0xFF)
                        {
                            ++number_of_bitflips_in_target;
                        }
                    }
                }
                if (number_of_bitflips_in_target > 0)
                {
                    printf("[!] Found %ld flips in row %ld (%lx to %lx) when hammering "
                            "%lx and %lx\n",
                            number_of_bitflips_in_target, row_index + 1,
                            ((row_index + 1) * presumed_row_size),
                            ((row_index + 2) * presumed_row_size) - 1,
                            GetPageFrameNumber(pagemap, first_row_page) * 0x1000,
                            GetPageFrameNumber(pagemap, second_row_page) * 0x1000);
                    total_bitflips += number_of_bitflips_in_target;
                }
            }
        }
    }
    return total_bitflips;
}
```
