# Assignment 9 – Report

## Q1. VM Print

Implemented the `sys_vmprint` system call (syscall number 22) along with the `vmprint()` function in `vm.c`. The function traverses page-directory entries below `KERNBASE`, following an approach similar to `freevm()`. It uses `walkpgdir()` to examine mappings and displays each page's virtual address, physical address, and permission flags (P/W/U).

The output also includes the number of valid user pages and page-table pages, accounting for the page directory. A page is considered a valid user page only when its `PTE_U` flag is set. The stack guard page at address `0x1000` is displayed separately because it is present but has `U=0`, so it is excluded from the user-page count.

Testing with `pttest` confirmed that the process initially contains two user pages. After executing `sbrk(4*4096)`, the count increases to six, representing exactly four additional pages. **Result: PASS.**

## Q2. Lazy Allocation

Modified `sys_sbrk()` so that positive increments update the process size (`sz`) without immediately allocating physical memory. Memory deallocation during shrinking remains unchanged.

Updated `copyuvm()` to skip pages that have not yet been allocated, preventing unnecessary kernel panics. The existing `deallocuvm()` implementation already handles non-present page-table entries correctly, and the relevant logic was documented.

Added page-fault handling for `T_PGFLT` in `trap.c`. When a fault occurs at an address below the process size and the corresponding page is not mapped, the handler allocates a physical page using `kalloc()`, initializes it with `memset()`, and maps it using `mappages()` with `PTE_W | PTE_U` permissions. Each successfully handled fault increments the `pgfaults` counter, after which execution resumes.

Invalid faults, including accesses beyond the process's allocated address range, cause only the offending process to be terminated. The handler prints the process ID and faulting address for debugging. A new `getfaults()` system call provides access to the page-fault counter. The `mappages()` function was changed from `static` to externally accessible so that `trap.c` could use it.

Testing with `faulttest` involved requesting 32 pages using `sbrk(32*4096)` and accessing five previously untouched pages. Exactly five page faults were handled successfully. An additional test verified that a child process accessing an address beyond its valid range was terminated without affecting its parent.

**Result: PASS.**

## Q3. FIFO and LRU Page Replacement

Created `pagerepl.c` to simulate the FIFO and LRU page-replacement algorithms. The program displays the frame contents after each page reference and identifies whether the reference results in a hit or a page fault.

Both algorithms were tested using the following reference string with three available frames:

`7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1`

The simulation produced **15 page faults for FIFO** and **12 page faults for LRU**, consistent with the expected results.

The results for different frame counts are summarized below:

| Number of Frames | FIFO Faults | LRU Faults | FIFO Hit Ratio | LRU Hit Ratio |
|---:|---:|---:|---:|---:|
| 3 | 15 | 12 | 25% | 40% |
| 4 | 10 | 8 | 50% | 60% |
| 5 | 9 | 7 | 55% | 65% |

The results demonstrate that LRU performs better than FIFO for this reference string, producing fewer page faults at each tested frame count.
