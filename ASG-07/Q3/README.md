# Question 3: Deadlock Detection and Avoidance in xv6

## 1. Overview
This project implements deadlock detection and Banker's algorithm in xv6 user space. It also demonstrates circular waiting using processes communicating through pipes.

## 2. Algorithms Implemented

### A. Deadlock Detection
The `detect_deadlock()` function uses the `Available`, `Allocation`, and `Request` matrices.

- Initializes `Work` with available resources and sets `Finish` based on resource allocation.
- Finds processes whose outstanding requests can be satisfied by `Work`.
- Adds their allocated resources back to `Work` and marks them finished.
- If no further process can finish, processes still marked unfinished are identified as potentially deadlocked.

### B. Banker's Algorithm
The `is_safe_state()` function determines whether granting a resource request preserves a safe state.

1. Calculates `Need = Max - Allocation`.
2. Checks whether the request exceeds the process's remaining need or available resources.
3. Temporarily allocates the requested resources.
4. Checks whether a safe execution sequence exists.
5. Grants the request if the state is safe; otherwise, restores the previous state and rejects it.

## 3. Test Cases

**Test 1: Deadlock Detection**
Five processes and three resource types are tested. The algorithm identifies `{P1, P2, P3, P4}` as deadlocked.

**Test 2: Safe State**
The algorithm finds the safe sequence:

`P1 -> P3 -> P4 -> P0 -> P2`

This sequence allows every process to complete.

**Test 3: Banker's Algorithm**
Process `P4` requests `[3, 3, 0]`. The hypothetical allocation leaves `[0, 0, 2]` available and produces an unsafe state. Therefore, the request is rejected.

**Test 4: Circular Wait Using Pipes**
Two processes hold different pipe endpoints and wait for data from each other. This creates a circular dependency that is analyzed using the deadlock detection algorithm.

## 4. Execution

Build and launch xv6:

```bash
make clean
make qemu-nox
```

Execute the program:

```text
$ deadlock
```

## 5. Conclusion
Deadlock detection identifies processes that cannot proceed under the current resource allocation, while Banker's algorithm prevents unsafe allocations. The pipe experiment demonstrates how circular waiting can arise between processes.