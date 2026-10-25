# Day 14/30 — Multilevel Queue (MLQ) & Multilevel Feedback Queue (MLFQ)

So far, every scheduling algorithm we've studied has treated all processes the same way, with one ready queue and one rule. But real systems have different kinds of processes: the code editor you're typing in, a system update running quietly in the background, and the OS's own internal tasks. Today we look at scheduling that splits the ready queue into **multiple queues**, one for each kind of process. Then we look at the more flexible version, where processes can **move between queues**.

---

## First, the Types of Processes

The handwritten notes start with a classification of processes:

```
                  Process
                 /        \
          User process    System process
           /        \          │
    Foreground    Background   └─ Performs the OS's internal / system tasks
        │              │
  Interacts      Runs in the background
  directly with  (system updates,
  the user       background synchronization)
  (e.g. VS Code)
```

- **User process**: started by the user.
  - **Foreground process**: interacts directly with the user, for example a code editor like VS Code.
  - **Background process**: runs in the background, for example a system update or background synchronization.
- **System process**: performs the OS's internal and system tasks.

The notes also distinguish processes by what they need most:

- **CPU-bound process**: needs a lot of CPU time to compute.
- **I/O-bound process**: needs a lot of I/O, so it spends most of its time waiting for devices.

Keep these two categories in mind, because MLQ and MLFQ both try to treat them differently.

---

## 1. Multilevel Queue Scheduling (MLQ)

### The Idea
- The **ready queue is divided into multiple queues**, based on priority.
- A process is **permanently assigned to one queue**, based on some property of the process: its **type, memory size, process priority**, and so on.
- **Each queue has its own scheduling algorithm.**

The notes give an example:

| Queue | Process type | Algorithm |
|---|---|---|
| **SP** (System Process) queue | System processes | RR |
| **IP** (Interactive Process) queue | Foreground processes | RR |
| **BP** (Batch Process) queue | Background processes | FCFS |

### The Structure

```
   Highest priority
   ┌────────────────────────────────┐
   │ System process (SP) queue      │  ← created by the OS
   └────────────────────────────────┘
   ┌────────────────────────────────┐
   │ Interactive process (IP) queue │  ← needs user input (foreground)
   └────────────────────────────────┘
   ┌────────────────────────────────┐
   │ Batch process (BP) queue       │  ← runs silently, no user input (background)
   └────────────────────────────────┘
   Lowest priority
```

- **System process**: created by the OS. Highest priority.
- **Interactive process (foreground)**: needs user input (I/O).
- **Batch process (background)**: runs silently, with no user input required.

### Scheduling Between Queues
The notes say scheduling **among the different queues** is implemented as **fixed-priority preemptive scheduling**.

- The **foreground queue has absolute priority** over the background queue.
- If an **interactive process arrives** while a **batch process is running**, the batch process is **preempted**.

### The Problems with MLQ
- **Starvation:** a lower-level queue is only scheduled once **all processes in the higher-level queues are completed**. If higher queues keep getting new arrivals, the lower queues may starve.
- **Convoy effect is present** within the queues that use FCFS.
- **Inflexible:** a process stays in its queue forever, even if its behavior changes. A process that starts as a batch job and becomes interactive is still stuck in the batch queue.

That last point is the reason MLFQ exists.

---

## 2. Multilevel Feedback Queue Scheduling (MLFQ)

### The Idea
MLFQ keeps the multiple queues of MLQ, but lets processes **move between queues**.

- The idea is to **separate processes according to the characteristics of their burst times**.
- If a process **uses too much CPU time**, it is **moved to a lower-priority queue**.
- **I/O-bound and interactive processes stay in the higher-priority queues**, because they give up the CPU quickly and need fast responses.

### Fixing Starvation with Aging
- A process that **waits too long** in a **lower-priority queue** is **moved up** to a higher-priority queue.
- This form of **aging** prevents starvation.

### Advantages of MLFQ
- **Less starvation** than MLQ.
- **Flexible**: processes can change queues as their behavior changes.
- **Can be configured** to match a specific system design requirement, such as the number of queues, the time quantum of each queue, and the rules for moving processes.

---

## Sample MLFQ Design

The notes show a four-queue design:

```
   Queue 0:  TQ = 2   ◄── new processes enter here (highest priority)
        │  demote if not finished in TQ
        ▼
   Queue 1:  TQ = 4
        │
        ▼
   Queue 2:  TQ = 8
        │
        ▼
   Queue 3:  FCFS     ◄── lowest priority, runs to completion
```

### How a Job Moves Through It

Take a CPU-bound job with a burst time of **20**:

1. **Queue 0 (TQ = 2):** it runs for 2 units. It isn't finished, so it is **demoted**. Remaining: 18.
2. **Queue 1 (TQ = 4):** it runs for 4 units. Not finished, so it is **demoted**. Remaining: 14.
3. **Queue 2 (TQ = 8):** it runs for 8 units. Not finished, so it is **demoted**. Remaining: 6.
4. **Queue 3 (FCFS):** it runs its remaining 6 units to completion.

Total: 2 + 4 + 8 + 6 = **20**. The job has been pushed down to the bottom queue, which is exactly what MLFQ does with a process that uses a lot of CPU.

Now take an **I/O-bound job**. It starts in Queue 0, does a little computation, and then waits for I/O before its 2-unit quantum runs out. It never gets demoted, so it **stays in the high-priority queue** and gets its I/O served quickly. That's the behavior MLFQ is designed to reward.

> **The intuition:** short, interactive work stays near the top and feels responsive. Long, CPU-hungry work gradually sinks to the bottom, where it can use the CPU without slowing down the interactive work above it.

---

## Comparison of All the Algorithms

The notes end with a summary table comparing every algorithm from Days 12 to 14.

| | FCFS | SJF | PSJF | Priority | P-Priority | RR | MLQ | MLFQ |
|---|---|---|---|---|---|---|---|---|
| **Design** | Simple | Complex | Complex | Complex | Complex | Simple | Complex | Complex |
| **Preemption** | No | No | Yes | No | Yes | Yes | Yes | Yes |
| **Convoy effect** | Yes | Yes | No | Yes | Yes | No | Yes | Yes |
| **Overhead** | No | No | Yes | No | Yes | Yes | Yes | Yes |

**How to read the abbreviations:**
- **SJF** here is the **non-preemptive** version, because **PSJF** (preemptive SJF, also called SRTF) is listed separately.
- **Priority** is non-preemptive, and **P-Priority** is preemptive priority.

**Key patterns in the table:**
- **Only PSJF and RR avoid the convoy effect.** PSJF does it by always running the shortest job, and RR does it by giving every process a time slice.
- **FCFS is the only simple design besides RR.** RR is the only simple design that's also preemptive and convoy-free.
- **Overhead is "Yes" for every preemptive algorithm**, because preemption means context switches. That's the trade-off for responsiveness.

---

## Key Takeaways

- Processes can be **user processes** (foreground or background) or **system processes**. They can also be **CPU-bound** (needs computation) or **I/O-bound** (needs I/O).
- **MLQ** splits the ready queue into queues by process type or priority, gives each queue its own algorithm, and schedules between queues with fixed-priority preemption. Its weaknesses are starvation, the convoy effect in FCFS queues, and inflexibility.
- **MLFQ** lets processes move between queues. CPU-hungry processes sink to lower queues, interactive and I/O-bound processes stay high, and **aging** moves long-waiting processes up to prevent starvation.
- The sample design uses **TQ = 2, TQ = 4, TQ = 8, and FCFS**. A process that keeps using the CPU sinks through the queues.
- Across all eight algorithms, **only PSJF and RR avoid the convoy effect**, and every preemptive algorithm pays a context-switch overhead.

---

**Next up — Day 15:** **Introduction to Concurrency**: what it means for processes and threads to run at the same time, and why it's harder than it looks.
