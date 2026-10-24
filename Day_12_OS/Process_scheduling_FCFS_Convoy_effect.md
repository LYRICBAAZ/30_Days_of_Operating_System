# Day 12/30 — Process Scheduling Basics, FCFS, Convoy Effect & Starvation

Day 11 covered what the OS does behind the scenes: swapping, context switching, and orphan and zombie processes. Today we reach the decision at the heart of multiprogramming: **which ready process gets the CPU next?** That decision is **process scheduling**. Its simplest algorithm is **First Come, First Served (FCFS)**, and FCFS leads to two problems you must be able to tell apart: the **convoy effect** and **starvation**.

---

## 1. Process Scheduling: The Basics

- Process scheduling is the **basis of a multiprogramming OS**.
- By **switching the CPU among processes**, the OS makes the computer more productive. The CPU is never left idle while processes are waiting.
- Many processes stay in memory at once. When a process has to wait (for I/O) or its **time quantum expires**, the OS takes the CPU away and gives it to another process. This pattern keeps repeating.

---

## 2. The CPU Scheduler

- **Whenever the CPU becomes idle**, the OS must **select one process from the ready queue** to execute.
- This selection is done by the **Short-Term Scheduler (STS)**, which we met on Day 10.

---

## 3. Two Kinds of Scheduling

### Non-Preemptive Scheduling
- Once the CPU is allocated to a process, the process **keeps the CPU** until it releases it, either by **terminating** or by **switching to the wait state**.
- The OS cannot take the CPU away in the middle of execution.

**Drawbacks:**
- **Starvation**: a process with a long burst time can hold the CPU and starve processes with shorter burst times.
- **Low CPU utilization**: the CPU can sit underused while the holding process isn't making progress.

### Preemptive Scheduling
- The CPU **can be taken away** when the **time quantum expires**, as well as when the process terminates or switches to the wait state.

**Advantages:**
- **Less starvation**, because no single process can hold the CPU indefinitely.
- **High CPU utilization**, because the CPU keeps rotating between processes.

| | Non-Preemptive | Preemptive |
|---|---|---|
| CPU taken away? | Only on termination or wait | Also when the time quantum expires |
| Starvation | Possible | Less |
| CPU utilization | Low | High |

---

## 4. Goals of CPU Scheduling

A good scheduling algorithm tries to achieve these:

1. **Maximum CPU utilization**: keep the CPU as busy as possible.
2. **Minimum turnaround time (TAT)**: finish each process quickly, from submission to completion.
3. **Minimum waiting time**: reduce the time processes spend in the ready queue.
4. **Minimum response time**: make processes start responding quickly.
5. **Maximum throughput**: complete as many processes as possible per unit time.

No single algorithm achieves all five at once, which is why there are so many scheduling algorithms. Each one makes different trade-offs.

---

## 5. Key Terms

These are the measurements you'll use in every scheduling problem.

| Term | Definition | Formula |
|---|---|---|
| **Throughput** | Number of processes completed per unit time. | — |
| **Arrival Time (AT)** | The time when a process arrives in the ready queue. | — |
| **Burst Time (BT)** | The time the process needs for its execution on the CPU. | — |
| **Completion Time (CT)** | The time when the process terminates. | — |
| **Turnaround Time (TAT)** | Time from when the process first enters the ready state until it terminates. | **TAT = CT − AT** |
| **Waiting Time (WT)** | Total time the process spends waiting for the CPU. | **WT = TAT − BT** |
| **Response Time** | Time between the process entering the ready queue and getting the CPU for the **first time**. | — |

> **Remember the two formulas:** TAT = CT − AT, and WT = TAT − BT. Almost every scheduling question is solved with these two.

---

## 6. FCFS: First Come, First Served

FCFS is the simplest scheduling algorithm:

- **Whichever process arrives first** in the ready queue gets the CPU first.
- Once a process gets the CPU, it runs to completion, because FCFS is **non-preemptive**.
- The ready queue works like a ticket counter: first in, first out.

### Worked Example

Three processes arrive at time 0:

| Process | Arrival Time | Burst Time |
|---|---|---|
| P1 | 0 | 24 |
| P2 | 0 | 3 |
| P3 | 0 | 3 |

**Case 1: Order P1 → P2 → P3**

| Process | CT | TAT = CT − AT | WT = TAT − BT |
|---|---|---|---|
| P1 | 24 | 24 − 0 = 24 | 24 − 24 = **0** |
| P2 | 27 | 27 − 0 = 27 | 27 − 3 = **24** |
| P3 | 30 | 30 − 0 = 30 | 30 − 3 = **27** |

Average waiting time = (0 + 24 + 27) / 3 = **17**

**Case 2: Order P2 → P3 → P1** (the same processes, with the short ones first)

| Process | CT | TAT = CT − AT | WT = TAT − BT |
|---|---|---|---|
| P2 | 3 | 3 − 0 = 3 | 3 − 3 = **0** |
| P3 | 6 | 6 − 0 = 6 | 6 − 3 = **3** |
| P1 | 30 | 30 − 0 = 30 | 30 − 24 = **6** |

Average waiting time = (0 + 3 + 6) / 3 = **3**

The same processes with the same burst times give an average wait of **17** in one order and **3** in the other. The arrival order alone changes the result dramatically. That's the key weakness of FCFS.

---

## 7. The Convoy Effect

The notes define it clearly:

> **Convoy Effect** is a situation where many processes, which need a resource for a short time, are blocked by one process holding that resource for a long time.

If one process with a **long burst time** arrives first, it **holds the CPU**, and everything behind it waits. The short processes line up behind the long one, like cars stuck behind a slow truck on a single-lane road. That line is the "convoy."

In the worked example, P1 (burst 24) arrives first, and P2 and P3 (burst 3 each) wait for it to finish. Their waiting times balloon to 24 and 27, even though each needs only 3 units of CPU.

**Consequence:** this causes **poor resource management**. The CPU is busy, but it serves the long process while many short processes sit in the queue. Average waiting time and turnaround time both suffer.

---

## 8. Starvation

The handwritten notes give a short, clear definition:

> **Starvation** means a process is ignored for a long time. It doesn't get the CPU for a long period, while other processes keep getting it.

### How Starvation Is Reduced: Aging
In priority scheduling, starvation can be reduced with **aging**, a technique where the **priority of a waiting process is gradually increased** the longer it waits. Eventually even a low-priority process rises high enough to get the CPU. (We'll see this in detail on Day 13.)

---

## 9. Convoy Effect vs Starvation

These two problems are easy to confuse, so here is the side-by-side comparison from the notes:

| | Convoy Effect | Starvation |
|---|---|---|
| **What happens** | Short processes wait behind one long process. | One process waits indefinitely while others get the CPU. |
| **Where it's common** | FCFS | Priority scheduling |
| **Effect on the system** | System performance and throughput are affected. | A particular process doesn't get fair execution, and its execution isn't guaranteed. |
| **Does everything finish?** | Yes, all processes eventually execute. | Not guaranteed. A starved process may never execute. |
| **Fix** | Algorithms that favor short jobs or time-slice the CPU (such as SJF or RR) | Aging, which raises the priority of long-waiting processes |

**Quick memory trick:** in the **convoy effect**, the processes are stuck in a line, but the line eventually moves. In **starvation**, one process is left behind and the line may never call its name.

---

## Key Takeaways

- **Process scheduling** is the foundation of multiprogramming. The **STS** selects the next ready process whenever the CPU becomes idle.
- **Non-preemptive** scheduling keeps the CPU with a process until it terminates or waits. **Preemptive** scheduling can also take the CPU away when the time quantum expires.
- The five main goals are **max CPU utilization, min TAT, min WT, min response time, and max throughput**.
- **TAT = CT − AT** and **WT = TAT − BT**.
- **FCFS** serves processes in arrival order. It is simple, but its average waiting time depends heavily on arrival order.
- The **convoy effect** is many short processes stuck behind one long process. **Starvation** is one process being ignored for a long time. They are different problems with different fixes.

---

**Next up — Day 13:** **CPU Scheduling: SJF, Priority & Round Robin**. Each of these algorithms tries to fix the problems we saw today.
