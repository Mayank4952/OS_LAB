# Assignment 10 Report

This assignment extends xv6-public at commit `eeb7b41` with memory-management
system calls and user programs demonstrating four paging topics. The complete
kernel source is in [`../xv6/`](../xv6/), and the question folders contain the
test sources and captured output.

## Implementation

- **Q1 — Physical-memory statistics:** [`kalloc.c`](../xv6/kalloc.c) tracks
  free, allocated, and freed pages under the allocator lock. Boot-time free-list
  initialization is excluded from the runtime counters. `memstats()` reports
  total, available, allocated, freed, and currently allocated pages.
  [`memtest.c`](../xv6/memtest.c) allocates, touches, and releases three pages.
- **Q2 — Virtual-to-physical translation:** [`vm.c`](../xv6/vm.c) implements
  `vtop()`, checking for a present user mapping and preserving the virtual-page
  offset. [`vttest.c`](../xv6/vttest.c) tests a mapped address, the next byte,
  and an unmapped address.
- **Q3 — Page information and read-only protection:** `pageinfo()` reports the
  frame and P/W/U bits, while `setreadonly()` clears `PTE_W` and invalidates the
  TLB entry. [`pagetest.c`](../xv6/pagetest.c) verifies the page flags and forks
  a child to attempt a write.
- **Q4 — Page replacement:** [`pagereplace.c`](../xv6/pagereplace.c) traces
  FIFO, LRU, and Optimal for each reference with three frames, then compares
  faults, hits, and their ratios with three, four, and five frames.

System-call declarations, dispatch, and user stubs are wired through
[`sysproc.c`](../xv6/sysproc.c), [`syscall.c`](../xv6/syscall.c),
[`syscall.h`](../xv6/syscall.h), [`defs.h`](../xv6/defs.h),
[`user.h`](../xv6/user.h), and [`usys.S`](../xv6/usys.S). The xv6
[`Makefile`](../xv6/Makefile) includes all four test programs in the filesystem
image.

## Results

The tests were run in QEMU from the built xv6 image. Question-specific console
transcripts and screenshots are stored with each question:

- [Q1 transcript](../Q1/memtest.txt) · [Q1 screenshot](../Q1/memtest.png)
- [Q2 transcript](../Q2/vttest.txt) · [Q2 screenshot](../Q2/vttest.png)
- [Q3 transcript](../Q3/pagetest.txt) · [Q3 screenshot](../Q3/pagetest.png)
- [Q4 transcript](../Q4/pagereplace.txt) · [Q4 screenshot](../Q4/pagereplace.png)

### Q1 — Physical-memory statistics

The runtime allocator pool contained **57,000 pages**. Before the test
allocation, 56,789 pages were available, with 425 runtime allocations and 214
releases (211 currently allocated). Allocating and touching three pages reduced
availability to 56,786 and raised current allocations to 214. Releasing the
pages restored availability to 56,789 and current allocations to 211; the
released-page counter increased to 217.

### Q2 — Virtual-to-physical translation

In the recorded run, virtual address `0x3000` translated to `0xDFBC000`, and
`0x3001` translated to `0xDFBC001`, preserving the page offset. Unmapped address
`0x7FFFF000` returned `-1`.

### Q3 — Page information and read-only protection

The allocated page mapped to frame 57276 (`0xDFBC000`). Its flags were P=1,
W=1, U=1 before protection; after `setreadonly()`, W became 0. The forked
child's write caused trap 14 (page fault, error 7), and the parent continued.

### Q4 — Replacement algorithm results

Reference string: `7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1` (20 references).

| Frames | Algorithm | Faults | Hits | Hit ratio | Fault ratio |
|---:|---|---:|---:|---:|---:|
| 3 | FIFO | 15 | 5 | 25.00% | 75.00% |
| 3 | LRU | 12 | 8 | 40.00% | 60.00% |
| 3 | Optimal | 9 | 11 | 55.00% | 45.00% |
| 4 | FIFO | 10 | 10 | 50.00% | 50.00% |
| 4 | LRU | 8 | 12 | 60.00% | 40.00% |
| 4 | Optimal | 8 | 12 | 60.00% | 40.00% |
| 5 | FIFO | 9 | 11 | 55.00% | 45.00% |
| 5 | LRU | 7 | 13 | 65.00% | 35.00% |
| 5 | Optimal | 7 | 13 | 65.00% | 35.00% |

Optimal has the fewest faults with three frames. With four and five frames,
Optimal and LRU tie for this reference string.

## Build and validation

The kernel, user programs, `xv6.img`, and `fs.img` built successfully with
`-Werror` enabled. The installed compiler reports legacy warnings in `mp.c`
and `sh.c`; the build exempts those two existing warning classes. All four
programs were run from fresh QEMU boots.
