# Operating Systems Assignment 9: xv6 Virtual Memory

This repository contains implementations for three xv6 virtual-memory problems. Each question is organized separately and includes the modified kernel source files, a test program, a patch file, and output captured during actual execution.

## Assignment Overview

| Folder | Description | Test Program | Verified Result |
|---|---|---|---|
| `Q1` | Implementing the `vmprint()` system call to display user page-table mappings | `pttest.c` | 2 user pages initially and 6 after `sbrk(4*4096)`, confirming 4 additional pages |
| `Q2` | Implementing lazy memory allocation, page-fault handling, and the `getfaults()` system call | `faulttest.c` | 5 page faults for 5 accessed pages; an invalid memory access terminates only the child process |
| `Q3t` | Simulating FIFO and LRU page-replacement algorithms | `pagerepl.c` | With 3 frames, FIFO produces 15 faults and LRU produces 12 |

The repository also includes [`brief_report.md`](brief_report.md), which summarizes the implementation details, testing procedures, and results for all three questions.

## Repository Structure

Each question has its own directory containing the following files:

```text
Qn_*/
  kernel/                 Modified xv6 kernel source files
                          (changes marked "Assignment 9")
  user/                   Test program for the question
  xv6.patch               Patch file relative to the base commit
  output/                 Captured terminal output and screenshots
```

## Building and Running

The implementation is based on the official [`mit-pdos/xv6-public`](https://github.com/mit-pdos/xv6-public) repository, using commit `eeb7b41`.

Apply each patch to a separate, clean copy of the base repository. This is important because Q1 and Q2 modify `sbrk()` behavior differently for their respective tests.

```bash
git clone https://github.com/mit-pdos/xv6-public
cd xv6-public
git checkout eeb7b41

# Apply the patch for the required question
patch -p1 < ../os-assignment-9/Q1_page_table/xv6.patch

# Build and launch xv6
make qemu-nox
```

Replace the Q1 patch path with the corresponding Q2 or Q3 path when testing another question. At the xv6 shell prompt, execute the relevant test program:

- `pttest` for Q1
- `faulttest` for Q2
- `pagerepl` for Q3

### Q3: Page-Replacement Simulator

The `pagerepl` program supports several execution modes:

```text
pagerepl
pagerepl 3 7,0,1,2,0,3,0,4,2,3,0,3,2,1,2,0,1,7,0,1
pagerepl table <refs>
```

The first command runs the default demonstration. The second simulates the supplied reference string using three frames, while the third displays a comparison table for the requested references.

Use comma-separated page references because the xv6 shell has a limit of ten arguments.

## Testing and Validation

All three implementations were compiled using the xv6 Makefile with `-Werror` enabled, and the generated kernel and disk images built successfully.

Because QEMU was unavailable in the testing environment, the xv6 boot disk and file system were executed using the v86 x86 emulator through Node.js. Commands were entered through the xv6 serial console, and the resulting UART output was captured.

