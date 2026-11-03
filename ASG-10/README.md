# Operating Systems Assignment 10

This assignment extends xv6-public with memory-management system calls and
demonstrates four paging concepts. The source checkout is based on commit
`eeb7b41` from `mit-pdos/xv6-public`.

## Assignment files

Each question has its own folder, following the layout used by the earlier
assignments. The folder contains the corresponding test source, console output,
and screenshot.

| Folder | Topic | xv6 test program |
|---|---|---|
| [`Q1/`](Q1/) | Physical-memory allocation statistics | `memtest` |
| [`Q2/`](Q2/) | Virtual-to-physical address translation | `vttest` |
| [`Q3/`](Q3/) | Page information and read-only protection | `pagetest` |
| [`Q4/`](Q4/) | FIFO, LRU, and Optimal page replacement | `pagereplace` |

[`Report/REPORT.md`](Report/REPORT.md) describes the implementation and
summarizes the recorded results. The complete modified xv6 source and its build
files are in [`xv6/`](xv6/).

The assignment sheet is [`OS_Assignment_10.pdf`](OS_Assignment_10.pdf).

## Build and run

From the `xv6` directory, build the boot and filesystem images:

```sh
make TOOLPREFIX= CFLAGS='-fno-pic -static -fno-builtin -fno-strict-aliasing -O2 -Wall -Werror -Wno-error=array-bounds -Wno-error=infinite-recursion -MD -ggdb -m32 -fno-omit-frame-pointer -fno-stack-protector -fno-pie -no-pie' xv6.img fs.img
```

With QEMU installed, start the xv6 shell:

```sh
make qemu-nox
```

Run the test programs at the xv6 prompt:

```text
memtest
vttest
pagetest
pagereplace
```

`pagereplace` traces FIFO, LRU, and Optimal with three frames, then prints a
comparison table for three, four, and five frames.
