# Question 4: Priority-Based CPU Scheduling in xv6

## 1. Overview
The default xv6 scheduler uses Round-Robin scheduling without priorities. This implementation introduces preemptive priority scheduling with dynamic aging to improve CPU allocation and prevent starvation.

## 2. Scheduling Mechanism
- **Priority range:** 1 (highest) to 20 (lowest), with a default priority of 10.
- **Preemption:** Timer interrupts trigger `yield()`, allowing the scheduler to select a higher-priority runnable process.
- **Tie-breaking:** Processes with equal priorities are scheduled using Round-Robin ordering.
- **Dynamic aging:** Each runnable process tracks its waiting time. After 30 ticks, its priority improves by one level, up to priority 1. The waiting counter resets when the process is scheduled.

## 3. System Calls

**`set_priority(pid, priority)`**
Validates the priority, locates the process, updates its priority, and resets its waiting counter. It returns the previous priority on success or `-1` on failure.

**`get_priority(pid)`**
Returns the current priority of the specified process or `-1` if the process is not found.

## 4. Testing with `priotest.c`
Four CPU-intensive child processes are created with priorities 5, 10, 15, and 20.

Higher-priority processes generally receive CPU time sooner. Dynamic aging gradually increases the priority of waiting processes, allowing lower-priority tasks to execute instead of starving indefinitely.

The kernel can print aging messages showing when a process receives a priority boost.

## 5. Execution

Compile and launch xv6:

```bash
make clean
make qemu-nox
```

Run the test program:

```text
$ priotest
```

## 6. Conclusion
Priority-based scheduling favors important processes, while dynamic aging improves fairness by gradually promoting waiting processes. Together, these mechanisms provide more flexible CPU scheduling than basic Round-Robin scheduling.