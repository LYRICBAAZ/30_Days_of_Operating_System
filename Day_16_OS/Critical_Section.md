# Day 16/30 — The Critical Section Problem & How to Address It

Day 15 ended with a warning: when threads share memory, the result can depend on the order in which they run. Today we turn that warning into a precise problem. We'll see what a **critical section** is, why an innocent-looking `count++` can go wrong, and the approaches the OS uses to protect shared data: atomic operations, locks, Peterson's solution, and semaphores, which we'll cover next.

---

## Why Synchronization Matters

**Process synchronization techniques** play a key role in maintaining the **consistency of shared data**. When several processes or threads access the same data, the OS needs rules that keep that data correct.

---

## Critical Section

> The **critical section** is the segment of code where processes or threads access **shared resources**, such as common variables and files, and **perform write operations** on them.

Since processes and threads execute concurrently, **any process can be interrupted in the middle of execution**. That interruption is what makes the critical section dangerous.

**Simple definition from the handwritten notes:** the critical section is the part of the code where shared data or resources are accessed.

---

## Race Condition

A **race condition** occurs when **two or more threads can access shared data and try to change it at the same time**.

Because the thread scheduling algorithm can swap between threads at any time, **you don't know the order** in which the threads will access the shared data. So the result depends on the scheduling algorithm. Both threads are "racing" to access and change the data.

### Why `count++` Is Not Safe

The handwritten notes break `count++` into the steps it actually performs:

```
count++;        looks like one step, but it is really:

temp  = count + 1;
count = temp;
```

Now imagine `count` is 5, and two threads each execute `count++` once:

| Step | Thread A | Thread B | `count` |
|---|---|---|---|
| 1 | reads `count` (5), computes `temp = 6` | | 5 |
| 2 | | reads `count` (5), computes `temp = 6` | 5 |
| 3 | writes `count = 6` | | 6 |
| 4 | | writes `count = 6` | 6 |

Both threads incremented, so `count` should be **7**, but it ends up as **6**. One update was **lost**. The scheduler simply interleaved the steps in a bad order. This is exactly the kind of error a race condition produces, and it can appear only once in a million runs, which makes it very hard to debug.

---

## Solutions to the Race Condition

The notes list three main approaches:

### 1. Atomic Operations
- Make the **critical code section an atomic operation**, meaning it runs as a single indivisible step.
- The notes phrase it as "executed in one CPU cycle". Once an operation is atomic, no other thread can see or interrupt it halfway through.
- Hardware instructions such as compare-and-swap are built for this purpose.

### 2. Mutual Exclusion Using Locks (Mutex)
- Only **one thread at a time** is allowed in the critical section.
- A thread takes the lock before entering, and releases it when it leaves. Other threads wait until the lock is free.

### 3. Semaphores
- A more general synchronization tool that uses an integer counter and two operations. We'll study these in detail on Day 17.

---

## Can a Simple Flag Variable Solve It?

The notes answer directly: **No.**

A flag variable seems like it should work: a thread sets `flag = true` before entering the critical section and sets it back to `false` when leaving. The trouble is that **checking the flag and setting it are two separate steps**. Two threads can both check the flag, both see `false`, and both enter at the same time. The check-then-set itself becomes a race condition.

This is why we need either atomic hardware operations or a carefully designed algorithm, such as Peterson's solution.

---

## Peterson's Solution

Peterson's solution is a **software-based synchronization algorithm** used in the OS. Its main purpose is to **solve the critical section problem**.

### The Two Shared Variables

- **`flag[2]`**: an array of two boolean values, one for each process. `flag[i] = true` means **process i wants to enter** the critical section.
- **`turn`**: decides whose turn it is if both processes are interested at the same time.
  - `turn = 0` means that if there is a conflict, **P0 gets preference**.
  - `turn = 1` means that if there is a conflict, **P1 gets preference**.

### Example from the Notes
- `flag[0] = true` → P0 wants to enter.
- `flag[1] = true` → P1 wants to enter.
- `turn = 0` → there is a conflict, and P0 has preference, so **P0 enters first**.

### The Algorithm (Standard Form)

The notes describe the idea rather than the code, so here is the standard form for process `i`, where `j` is the other process:

```
flag[i] = true;          // I want to enter
turn    = j;             // but let the other process go first if it also wants in

while (flag[j] && turn == j)
    ;                    // wait while the other one wants in and has the turn

    // ---- critical section ----

flag[i] = false;         // I am done
```

Why it works: a process only waits when **the other process wants to enter and has the turn**. Whichever process sets `turn` last gives the other one priority, so both can't enter at the same time.

### The Limitation
> **Peterson's solution works for only 2 processes or threads.**

It cannot be extended cleanly to many threads, which is why real systems prefer hardware atomic operations and locks.

---

## Mutex / Locks

Locks can be used to implement mutual exclusion and avoid race conditions, by **allowing only one thread or process to access the critical section at a time**.

### Disadvantages of Locks

The notes list four problems:

| Problem | What happens |
|---|---|
| **Contention** | When one thread holds the lock, the other threads are **busy waiting**. Worse, if the thread that holds the lock **dies**, every other thread waits **forever**. |
| **Deadlocks** | Two or more threads each hold a lock the other needs, so none of them can continue. |
| **Debugging** | Lock-related bugs are hard to reproduce and trace, because they depend on timing. |
| **Starvation of high-priority threads** | A high-priority thread may keep losing the lock to others, so it can't make progress. |

Locks solve the race condition, but they introduce new problems. Every synchronization tool is a trade-off between safety and these kinds of problems.

---

## Summary Table: Problems and Their Fixes

| Problem | Fix | Limitation |
|---|---|---|
| Race condition on shared data | Atomic operations | Only works for simple, hardware-supported operations |
| Race condition on shared data | Mutex / locks | Contention, deadlock, debugging, starvation |
| Race condition with 2 processes | Peterson's solution | Only for 2 processes or threads |
| Race condition with a flag variable | Doesn't work | Check and set are two separate steps |
| General synchronization | Semaphores (Day 17) | More general, covered next |

---

## Key Takeaways

- **Synchronization** keeps shared data consistent when processes or threads run concurrently.
- The **critical section** is the code that accesses shared resources and writes to them. Any process can be interrupted inside it.
- A **race condition** happens when threads change shared data at the same time, and the result depends on the scheduling order. `count++` is really two steps, so it can lose updates.
- The three main solutions are **atomic operations**, **mutual exclusion with locks**, and **semaphores**. A simple flag variable does not work.
- **Peterson's solution** is a software algorithm that solves the critical section problem, but only for **two** processes or threads.
- **Locks** fix the race but bring **contention, deadlocks, debugging difficulty, and starvation** with them.

---

**Next up — Day 17:** **Condition Variables and Semaphores**: how semaphores count available resources and signal between threads, and how condition variables let a thread wait for a specific condition.
