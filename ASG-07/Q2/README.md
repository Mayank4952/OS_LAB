# Question 2: Synchronization Using Semaphores in xv6

## 1. Overview
Concurrent threads modifying shared data can cause race conditions. To prevent this, counting semaphores are implemented in xv6 using spinlocks and the `sleep()` and `wakeup()` primitives.

## 2. Semaphore Implementation

A semaphore structure is defined in `semaphore.h`:

```c
#define MAX_SEM 10

struct semaphore {
    int value;
    int active;
    struct spinlock lk;
};

struct semaphore sem_table[MAX_SEM];
```

The kernel maintains a fixed-size table of semaphores.

### System Calls

- **`sem_init(sem_id, value)`:** Validates the semaphore ID and initial value, initializes its count and lock, and marks it active.
- **`sem_wait(sem_id)`:** Decreases the semaphore count under a spinlock. If no resource is available, the calling thread sleeps until signaled.
- **`sem_signal(sem_id)`:** Increases the count and wakes one waiting thread when necessary.

The spinlock protects semaphore state, while `sleep()` and `wakeup()` coordinate waiting threads.

## 3. User Programs

### A. synctest.c
A binary semaphore initialized to 1 protects the shared counter:

```c
sem_init(0, 1);

sem_wait(0);
shared_counter++;
sem_signal(0);
```

Three threads each perform 100,000 increments. With correct synchronization, the final counter is exactly **300,000**, preventing lost updates.

### B. prodcons.c
This program implements the bounded-buffer Producer-Consumer problem using a buffer of size 5.

- `mutex = 1`: Protects shared buffer operations.
- `empty = 5`: Counts available slots.
- `full = 0`: Counts occupied slots.

The producer generates 10 items, while the consumer removes 10 items. Output messages track buffer operations and help verify that overflow and underflow do not occur.

## 4. Execution

Start xv6 using:

```bash
make clean
make qemu-nox
```

Run the synchronization test:

```text
$ synctest
```

Run the Producer-Consumer test:

```text
$ prodcons
```

## 5. Conclusion
Semaphores provide synchronization between concurrent threads and protect shared resources. The counter test demonstrates mutual exclusion, while the Producer-Consumer program illustrates coordination between processes operating on a bounded buffer.