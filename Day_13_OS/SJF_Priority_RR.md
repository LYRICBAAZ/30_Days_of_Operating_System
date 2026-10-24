# Day 13/30 — CPU Scheduling: SJF, Priority & Round Robin

Day 12 showed the main weakness of FCFS: the convoy effect, where one long process at the front of the queue holds up everyone behind it. Today we look at three algorithms that each try to fix that in a different way: **Shortest Job First (SJF)**, which favors short jobs; **Priority Scheduling**, which favors important jobs; and **Round Robin (RR)**, which gives every process a fair slice of CPU time.

---

## 1. Shortest Job First (SJF)

### The Idea
> The process with the **least burst time** is dispatched to the CPU first.

Short jobs finish quickly, so they don't pile up behind long ones. That's the whole intuition.

### The Catch: Burst Time Must Be Estimated
- The OS must **estimate the burst time** of each process in the ready queue beforehand.
- An exact calculation is **practically impossible**. The estimate may be right, or it may be off, so a process estimated at 5 units might actually take 8.
- SJF is only as good as its estimates.

### SJF Non-Preemptive
- Once the CPU is assigned to a process, it keeps the CPU **until the process terminates**.
- It then picks the job with the **lowest burst time at that instant**.
- Because the CPU isn't taken away, there's **no context switch** during a process's run.

**Drawbacks:**
- It **suffers from the convoy effect** if the first process to arrive has a large burst time.
- **Starvation** can happen.

### SJF Preemptive (Shortest Remaining Time First)
- If a newly arriving process has a **shorter burst time than the remaining time** of the running process, the running process is preempted.
- **Less starvation**, and **no convoy effect**.
- Gives the **lowest average waiting time** for a given set of processes. Scheduling a short job before a long one decreases the waiting time of the short job **more** than it increases the waiting time of the long one.

### Worked Example: Non-Preemptive SJF

| Process | Arrival Time | Burst Time |
|---|---|---|
| P1 | 0 | 7 |
| P2 | 2 | 4 |
| P3 | 4 | 1 |
| P4 | 5 | 4 |

**Timeline:**
- **t = 0:** only P1 has arrived, so it runs from **0 to 7**. Non-preemptive, so it finishes.
- **t = 7:** P2 (BT 4), P3 (BT 1), and P4 (BT 4) are waiting. P3 has the shortest burst, so it runs from **7 to 8**.
- **t = 8:** P2 and P4 both have burst time 4. The tie goes to whoever arrived first, so P2 runs from **8 to 12**, then P4 from **12 to 16**.

Gantt chart:

```
| P1      | P3 | P2   | P4   |
0         7    8      12     16
```

| Process | CT | TAT = CT − AT | WT = TAT − BT |
|---|---|---|---|
| P1 | 7 | 7 − 0 = 7 | 7 − 7 = **0** |
| P2 | 12 | 12 − 2 = 10 | 10 − 4 = **6** |
| P3 | 8 | 8 − 4 = 4 | 4 − 1 = **3** |
| P4 | 16 | 16 − 5 = 11 | 11 − 4 = **7** |

Average waiting time = (0 + 6 + 3 + 7) / 4 = **4**

Compare that with FCFS on the same arrival times: the average waiting time would be higher, because P3 (the 1-unit job) would have to wait behind P2. That is SJF doing its job.

---

## 2. Priority Scheduling

### The Idea
- Every process is **assigned a priority when it is created**.
- The CPU goes to the process with the **highest priority** in the ready queue.

> **SJF is a special case of priority scheduling**, where the priority is **inversely proportional to the burst time**. A shorter job gets a higher priority.

### Priority Non-Preemptive
- Once a process gets the CPU, it keeps it until it terminates or waits.
- A new high-priority arrival has to wait for the current process to finish.

### Priority Preemptive
- If a **new process arrives with a higher priority** than the one currently running, the running process is **preempted**.

### The Problem: Starvation
Priority scheduling can cause **indefinite waiting**, also called **starvation**, for low-priority processes. If high-priority processes keep arriving, a low-priority process may **never get executed**. This is true for both the preemptive and non-preemptive versions.

### The Solution: Aging
- **Aging** means gradually increasing the priority of processes that have waited for a long time.
- Example from the notes: **increase the priority by 1 every 15 minutes**.
- Over time, even the lowest-priority process eventually rises high enough to run.

The notes summarize it this way: aging is the solution, because it **gradually increases the priority of a process that waits so long**.

---

## 3. Round Robin (RR)

### The Idea
Round Robin is the **most popular** scheduling algorithm, and it's **designed for time-sharing systems**.

- It works like **FCFS, but preemptive**.
- Each process gets the CPU for a fixed **time quantum (TQ)**. When the quantum expires, the process goes to the back of the ready queue, and the next process gets its turn.
- The scheduling criteria are **arrival time and time quantum**. It does **not** depend on burst time.

### Why It's Good
- **No process waits forever**, so starvation is very low.
- **No convoy effect**, because a long process only holds the CPU for one quantum at a time.
- **Easy to implement.**

### The Trade-Off: Choosing the Time Quantum
- If the **TQ is small**, the CPU switches between processes very often, so there are **more context switches** and **more overhead**.
- If the **TQ is large**, RR behaves more and more like FCFS, with the convoy effect coming back.

So the time quantum is a balance: small enough for fairness, large enough to keep overhead low.


### How RR Works: The Flowchart

```
            Ready Queue
                 │
          Pick a process (FCFS)
                 │
         BT < TQ ? ──Yes──► Execute till termination ──► Terminate
                 │                                          ▲
                 No                                         │
                 │                                          │
        Execute for TQ                                      │
                 │                                          │
           TQ expires                                       │
                 │                                          │
        Process execution done? ──Yes──────────────────────┘
                 │
                 No
                 └──► back to Ready Queue
```

### Worked Example: Round Robin

All processes arrive at time 0, with **TQ = 4**:

| Process | Arrival Time | Burst Time |
|---|---|---|
| P1 | 0 | 24 |
| P2 | 0 | 3 |
| P3 | 0 | 3 |

**Timeline:** the ready queue starts as P1, P2, P3.

- P1 runs **0 to 4**, and has 20 units left. It goes to the back of the queue.
- P2 runs **4 to 7**. Its burst is 3, which is less than the quantum, so it finishes.
- P3 runs **7 to 10**, and finishes.
- P1 keeps getting 4-unit slices: **10–14, 14–18, 18–22, 22–26, 26–30**. It finishes at 30.

Gantt chart:

```
| P1 | P2  | P3  | P1 | P1 | P1 | P1 | P1 |
0    4     7     10   14   18   22   26   30
```

| Process | CT | TAT = CT − AT | WT = TAT − BT |
|---|---|---|---|
| P1 | 30 | 30 − 0 = 30 | 30 − 24 = **6** |
| P2 | 7 | 7 − 0 = 7 | 7 − 3 = **4** |
| P3 | 10 | 10 − 0 = 10 | 10 − 3 = **7** |

Average waiting time = (6 + 4 + 7) / 3 ≈ **5.67**

Compare this with FCFS on the same processes in Day 12, where the average waiting time was **17** when P1 went first. Round Robin cut it to about 5.67, because the short processes finished early instead of waiting behind P1.

---

## Comparison at a Glance

| Algorithm | Preemptive? | Starvation | Convoy Effect | Depends on | Notes |
|---|---|---|---|---|---|
| **SJF (non-preemptive)** | No | Possible | Possible | Burst time (estimated) | Lowest average wait among non-preemptive, but needs estimates |
| **SJF (preemptive)** | Yes | Less | No | Burst time (estimated) | Lowest average wait for a given set of processes |
| **Priority (non-preemptive)** | No | Possible (fix with aging) | Possible | Priority | SJF is a special case |
| **Priority (preemptive)** | Yes | Possible (fix with aging) | Possible | Priority | A higher-priority arrival preempts the running process |
| **Round Robin** | Yes | Very low | No | Arrival time and time quantum | Most popular; fair; overhead depends on TQ |

---

## Key Takeaways

- **SJF** dispatches the process with the **shortest burst time**. Its burst times must be **estimated**, and the non-preemptive version can still suffer the convoy effect.
- **Preemptive SJF** avoids the convoy effect, causes less starvation, and gives the lowest average waiting time for a given set of processes.
- **Priority scheduling** assigns a priority to each process at creation. SJF is the special case where priority is inversely proportional to burst time.
- **Starvation** in priority scheduling is solved by **aging**, which raises the priority of long-waiting processes.
- **Round Robin** is preemptive FCFS with a **time quantum**. It is fair and has very low starvation, but a **small time quantum means more context switches and overhead**.

---

**Next up — Day 14:** **Multilevel Queue (MLQ) and Multilevel Feedback Queue (MLFQ)**: how the OS separates processes into different queues, and how processes can move between them.
