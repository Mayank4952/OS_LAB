# Operating Systems Assignment 7: Implementation in xv6

## 1. Introduction

This assignment extends xv6 with four operating system features: thread management, semaphore-based synchronization, deadlock detection and avoidance, and priority-based CPU scheduling. Each feature is implemented and evaluated through user-level test programs.

## 2. Question 1: Thread Management

The `clone()` system call creates a thread that shares its parent's address space while maintaining a separate kernel stack and execution context. The `join()` system call waits for a thread to terminate and releases its resources without destroying shared memory.

A new `is_thread` field is added to `struct proc`, and the `wait()` and `exit()` routines are adjusted to handle threads correctly.

**Experimental result:** Three threads increment a shared counter 100,000 times each. Without synchronization, the expected value of 300,000 drops to 187,429 because concurrent updates overwrite one another.

## 3. Question 2: Semaphore-Based Synchronization

Kernel-level counting semaphores are implemented using spinlocks and xv6's `sleep()` and `wakeup()` mechanisms.

Three system calls manage synchronization:

- `sem_init()` initializes a semaphore.
- `sem_wait()` decrements its count and blocks when necessary.
- `sem_signal()` increments the count and wakes a waiting process when appropriate.

In `synctest.c`, a binary semaphore protects the shared counter. The final value reaches exactly 300,000, demonstrating correct mutual exclusion.

The `prodcons.c` program implements a bounded buffer of capacity five using `mutex = 1`, `empty = 5`, and `full = 0`. Ten items are produced and consumed without buffer overflow or underflow.

## 4. Question 3: Deadlock Detection and Avoidance

The `detect_deadlock()` function uses the `Available`, `Allocation`, and `Request` matrices to identify processes that cannot complete with the currently available resources.

Banker's algorithm calculates:

`Need = Max - Allocation`

Resource requests are granted only when they satisfy the process's remaining need and available-resource constraints, and the resulting state remains safe. Otherwise, the tentative allocation is rolled back.

Three matrix-based tests demonstrate deadlock detection, safe-state identification, and rejection of an unsafe request. The safe sequence obtained is:

`P1 -> P3 -> P4 -> P0 -> P2`

A separate pipe-based experiment models two processes waiting on resources held by each other, demonstrating circular waiting.

## 5. Question 4: Priority-Based CPU Scheduling

The default Round-Robin scheduler is modified to support preemptive priority scheduling with dynamic aging.

- Priority ranges from 1 (highest) to 20 (lowest), with a default of 10.
- `set_priority()` changes a process's priority, while `get_priority()` retrieves it.
- Timer interrupts allow the scheduler to reconsider runnable processes.
- Processes with equal priority are scheduled in Round-Robin order.
- After 30 waiting ticks, a runnable process receives a one-level priority improvement. Its waiting counter resets when scheduled.

The `priotest.c` program runs four CPU-intensive processes with initial priorities 5, 10, 15, and 20. Higher-priority processes generally finish earlier, while aging allows lower-priority processes to receive CPU time.

The lowest-priority process receives four boosts, moving from priority 20 to 16, and eventually completes.

## 6. Conclusion

The assignment demonstrates shared-memory thread creation, synchronization through semaphores, deadlock analysis using standard resource-allocation algorithms, and preemptive priority scheduling. The experiments illustrate how synchronization prevents lost updates, safe-state analysis helps avoid deadlocks, and dynamic aging improves scheduling fairness.