# Day 11/30 — Swapping, Context Switching, Orphan & Zombie Processes

Day 10 showed how a process moves between states and queues. Today we look at four things that happen to processes behind the scenes: the **medium-term scheduler** that swaps processes out of memory, the **context switch** that makes multitasking possible, and two special process situations, **orphan** and **zombie** processes, that come up when parents and children interact.

---

## 1. Swapping and the Medium-Term Scheduler

### The Problem
The degree of multiprogramming is the number of processes in memory. When that number goes up, RAM space runs out. The Long-Term Scheduler controls how many processes enter memory, but sometimes memory needs freeing up even after processes are already in it, for example when a process's memory requirements grow.

### The Solution: Medium-Term Scheduler (MTS)
A time-sharing system may have a **Medium-Term Scheduler (MTS)**. Its job:

1. **Remove processes from memory** to reduce the degree of multiprogramming.
2. **Reintroduce them later**, so their execution continues **where it left off**.

This removal and reintroduction is called **swapping**.

- **Swap-out** moves a process from main memory to secondary storage (swap space on disk).
- **Swap-in** brings it back into main memory later.
- Both swap-out and swap-in are done by the **MTS**.
- The MTS works on processes in the **ready queue**, and its control is tied to the degree of multiprogramming.

### Why Swapping Is Needed
- To **improve the process mix**, so that the right combination of CPU-bound and I/O-bound processes runs at the same time.
- Because a **change in memory requirements** has **overcommitted** the available memory, so memory must be freed.

### The Swapping Diagram
Here's the handwritten example from the notes:

```
  Memory:   [ P1 | P2 | P3 | P4 ]        P1 is running
                        │    │
              swap-out  ▼    ▼
                    Swap space (secondary memory)
                        [ P3 ]
                        [ P4 ]
                        │
              swap-in   ▼   (after P1 terminates)
```

P1 keeps running in memory. P3 and P4 are swapped out to disk to free space. When P1 finishes, P3 and P4 can be swapped back in.

### Where the MTS Fits
The notes place the three schedulers in order:

| Scheduler | Moves processes between | How often it runs |
|---|---|---|
| **Long-Term (LTS)** | Job queue → memory | Least often; controls degree of multiprogramming |
| **Medium-Term (MTS)** | Memory ↔ secondary storage (swap) | Occasionally, when memory is tight |
| **Short-Term (STS)** | Ready queue → CPU | Very frequently |

---

## 2. Context Switching

Context switching is what the OS does when it stops one process and starts another on the same CPU.

- Switching the CPU to another process requires a **state save** of the current process and a **state restore** of a different process.
- The **kernel saves the context** of the old process into its **PCB**, and **loads the saved context** of the new process scheduled to run.
- It is **pure overhead**. While switching, the system does **no useful work** for any process.
- The speed of a switch **varies from machine to machine**, depending on:
  - **Memory speed**, and
  - **the number of registers** that must be copied.

This is the same save-and-restore cycle we looked at on Day 9, where the registers are saved into the PCB.

> **Why it still matters:** context switching is the price of multitasking and multiprogramming. The OS pays a small, unavoidable overhead so that many processes can share one CPU.

---

## 3. Orphan Process

An **orphan process** is a process whose **parent has terminated**, but the process is **still running**.

- The OS doesn't let it become parentless for long. Orphan processes are **adopted by the init process**.
- **Init** is the **first process of the OS**. It starts when the system boots, and it has the process ID 1 on Linux-style systems.

So an orphan isn't a problem by itself. It keeps running normally, and init takes over as its parent.

---

## 4. Zombie Process (Defunct Process)

A **zombie process** is a process whose **execution has completed**, but which **still has an entry in the process table**.

### Why It Happens
- Zombies usually occur for **child processes**.
- When a child finishes, its exit status has to be **read by its parent**. The parent does this with the **`wait()` system call**.
- The parent may call `wait()` on the child **much later**, after the child has already terminated. In between, the child sits as a zombie.

### How It Goes Away: Reaping
- Once the parent reads the child's exit status using `wait()`, the zombie is **eliminated from the process table**.
- This is called **reaping** the zombie process.

### Why the Child Stays a Zombie
The process table entry **can only be removed after the parent reads the child's exit status**. Until then, the child remains a zombie, even though it has finished running.

---

## Orphan vs Zombie

These two are easy to mix up, so here's the difference side by side.

| | Orphan Process | Zombie Process |
|---|---|---|
| **Parent status** | Parent has terminated | Parent is still alive |
| **Child status** | Still running | Has finished execution |
| **Entry in process table** | Yes, as a normal process | Yes, only as a leftover entry |
| **What resolves it** | Adopted by init | Parent calls `wait()`, and the zombie is reaped |

**Quick memory trick:** an **orphan** has lost its parent, so **init adopts it**. A **zombie** has finished but its parent hasn't collected it, so it **waits to be reaped**.

---

## Key Takeaways

- The **Medium-Term Scheduler** swaps processes between main memory and secondary storage to reduce the degree of multiprogramming when memory is tight.
- **Swapping** lets a process pause and resume later, and **swap-out and swap-in** are done by the MTS.
- **Context switching** saves the old process's state into its PCB, loads the new one's state, and does no useful work in the meantime, so its speed depends on memory speed and the number of registers.
- An **orphan** process has lost its parent and is **adopted by init**.
- A **zombie** process has finished but still has a process table entry until its parent **waits for it** and it is **reaped**.

---

**Next up — Day 12:** **Process Scheduling: Introduction, FCFS, and the Convoy Effect**. We'll look at how the CPU scheduler chooses the next process, starting with the simplest algorithm, First Come First Served, and the problem it causes.
