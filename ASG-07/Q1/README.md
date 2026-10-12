# Question 1: Implementing Threads in xv6

## 1. Overview
xv6 normally uses `fork()` to create processes with separate address spaces. To support threads sharing memory, two system calls, `clone()` and `join()`, are implemented.

## 2. clone() System Call
```c
int clone(void (*fcn)(void *), void *arg, void *stack);
```
- Shares the parent's address space using `np->pgdir = curproc->pgdir` and `np->sz = curproc->sz`.
- Allocates a separate kernel stack using `kalloc()`.
- Copies the parent's trap frame and sets `eip` to the function address and `esp` to the supplied user stack.
- Places the argument and a dummy return address on the stack.
- Marks the process as a thread and duplicates file descriptors and the current working directory.
- Returns the new thread's PID.

## 3. join() System Call
`join()` waits for a child thread to terminate. It searches for zombie threads belonging to the caller, frees their kernel stacks, resets their process-table entries, and returns the terminated thread's PID.

Unlike normal process cleanup, it must not free the shared page directory because other threads may still use it.

## 4. Kernel Modifications
- **`proc.h`:** Adds an `is_thread` field to distinguish threads from processes.
- **`proc.c`:** Implements `clone()` and `join()`, modifies `exit()` to handle thread termination, and updates `wait()` to avoid freeing shared memory when cleaning up threads.

## 5. Testing and Race Condition
A test program creates three threads, each incrementing a shared global variable 100,000 times.

```c
volatile int counter = 0;

void worker(void *arg) {
    for (int i = 0; i < 100000; i++)
        counter++;
    exit();
}
```

The expected result is 300,000. However, `counter++` involves multiple operations: reading, incrementing, and writing. Concurrent execution can cause lost updates, producing a smaller value. This demonstrates a race condition. The `volatile` keyword does not make the operation atomic.

## 6. Execution
Compile and start xv6:

```bash
make clean
make qemu-nox
```

Run the test in the xv6 shell:

```text
$ threadtest
```

## 7. Conclusion
The implementation enables multiple threads to share an address space while maintaining separate execution contexts. The test demonstrates why synchronization mechanisms, such as locks, are necessary to prevent race conditions.