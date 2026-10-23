# Day 10/30 — Process States and Process Queues

On Day 9 we saw how a process is created and how its PCB stores its details. But a process doesn't just sit still. As it runs, it moves between situations: waiting for the CPU, actually running, waiting for an I/O device, and finally finishing. Today we map that life cycle with **process states**, and then look at the **queues** the OS uses to keep track of processes waiting their turn.

---

## Process States

As a process executes, it **changes state**. At any moment, each process is in exactly one of five states.

| State | What it means |
|---|---|
| **New** | The OS is about to pick the program and convert it into a process. The process is being created, and the CPU has not been allocated yet. |
| **Ready** | The process is in main memory, waiting to be assigned a processor. It has everything it needs except the CPU. |
| **Running** | Instructions are being executed. The CPU has been allocated to this process. |
| **Waiting** | The process is waiting for an I/O operation or some other event to complete. It can't use the CPU until that event happens. |
| **Terminated** | The process has finished execution. Its PCB entry is removed from the process table. |

---

## The State Diagram: How a Process Moves

The notes show how a process moves between these states. Each arrow is a transition, and each one has a trigger:

```
              admitted             scheduler dispatch
   ┌─────┐  ───────────►  ┌───────┐  ───────────────►  ┌─────────┐
   │ new │                │ ready │                    │ running │
   └─────┘  ◄───────────  └───────┘  ◄───────────────  └─────────┘
              (interrupt)               (interrupt)         │  │
                  ▲                                         │  │
                  │ I/O or event completion                  │  │ exit
                  │                                         │  ▼
                  │         I/O or event wait         ┌────────────┐
                  └──────────────────────────────────  │  waiting   │  
                                                      └────────────┘
```

A cleaner way to read the same diagram, transition by transition:

| From → To | Trigger |
|---|---|
| **New → Ready** | **Admitted**: the OS accepts the process into the ready state. |
| **Ready → Running** | **Scheduler dispatch**: the CPU scheduler picks the process and gives it the CPU. |
| **Running → Ready** | **Interrupt**: the process's time slice ends or a higher-priority event occurs, so it goes back to waiting its turn. |
| **Running → Waiting** | **I/O or event wait**: the process needs an I/O operation or an event, so it can't continue. |
| **Waiting → Ready** | **I/O or event completion**: the operation finishes, so the process is ready to run again. |
| **Running → Terminated** | **Exit**: the process finishes execution. |

> **A key point:** a waiting process does **not** go straight back to running. When its I/O finishes, it goes to **ready** first, and then the scheduler has to pick it again.

---

## Process Queues

The OS doesn't keep all processes in one big pile. It organizes them into **queues**, each one for a particular stage of the life cycle.

### 1. Job Queue
- Holds the processes in the **new** state.
- These processes are present in **secondary memory** (the disk), waiting to be brought in.
- The **Job Scheduler**, also known as the **Long-Term Scheduler (LTS)**, picks processes from this pool and **loads them into memory** for execution.

### 2. Ready Queue
- Holds the processes in the **ready** state.
- These processes are present in **main memory**, ready and waiting for the CPU.
- The **CPU Scheduler**, also known as the **Short-Term Scheduler (STS)**, picks a process from the ready queue and **dispatches it to the CPU**.

### 3. Waiting Queue
- Holds the processes in the **wait** state.
- These processes are waiting for an I/O operation or an event to complete.

### The Queues at a Glance

```
Secondary memory            Main memory                        CPU
┌────────────┐   LTS    ┌──────────────┐   STS    ┌──────────┐
│ Job Queue  │ ───────► │  Ready Queue │ ───────► │ Running  │
│ (new)      │          │  (ready)     │          │          │
└────────────┘          └──────────────┘          └────┬─────┘
                                ▲                      │ I/O wait
                                │  I/O completion      ▼
                                └────────────  ┌──────────────┐
                                               │ Waiting Queue│
                                               │  (wait)      │
                                               └──────────────┘
```

---

## Two Schedulers, Two Different Jobs

The notes name two schedulers, and they're easy to mix up because both "pick processes." The difference is **how often they run** and **what they control**.

| | Long-Term Scheduler (LTS) | Short-Term Scheduler (STS) |
|---|---|---|
| **Also called** | Job Scheduler | CPU Scheduler |
| **Picks from** | Job queue (secondary memory) | Ready queue (main memory) |
| **Moves process to** | Main memory (ready state) | CPU (running state) |
| **How often it runs** | Less often | Very frequently |
| **What it controls** | The **degree of multiprogramming** | Which ready process gets the CPU next |

### Degree of Multiprogramming
The **degree of multiprogramming** is the **number of processes in memory**. The notes say the **LTS controls it**. By deciding how many processes to load into memory, the LTS decides how many processes are competing for the CPU at once.

- Too few processes in memory, and the CPU sits idle when they wait for I/O.
- Too many, and each process gets too little time and memory.

The LTS keeps the balance. This ties back to the multiprogramming idea from Day 2: keep enough work in memory that the CPU always has something to run.

### Dispatcher
The **dispatcher** is the module of the OS that **gives control of the CPU to the process selected by the STS**. The STS decides *which* process runs next, and the dispatcher carries out the handover: switching context, restoring registers from the PCB, and starting the process.

---

## Putting It Together: One Process's Journey

Here's a single process moving through everything in this topic:

1. **New**: the OS starts converting the program into a process. It sits in the **job queue** on disk.
2. **Admitted**: the LTS loads it into memory. It moves to **ready** and joins the **ready queue**.
3. **Running**: the STS picks it, and the dispatcher gives it the CPU.
4. **Waiting**: it asks for disk data, so it moves to the **waiting queue**.
5. **Ready**: the disk finishes, so it goes back to the **ready queue**.
6. **Running** again: it gets the CPU, and eventually finishes.
7. **Terminated**: it exits, and its PCB is removed from the process table.

---

## Key Takeaways

- A process is in exactly one of five states: **new, ready, running, waiting, or terminated**.
- The arrows between states have triggers: **admitted, scheduler dispatch, interrupt, I/O wait, I/O completion, and exit**.
- A process that finishes waiting goes to **ready first**, not straight to running.
- There are three queues: the **job queue** (new), the **ready queue** (ready), and the **waiting queue** (wait).
- The **Long-Term Scheduler** controls the **degree of multiprogramming** by deciding what enters memory. The **Short-Term Scheduler** picks which ready process gets the CPU next.
- The **dispatcher** It is module of OS and gives CPU control to a process selected by STS.

---

**Next up — Day 11:** **Swapping** | **Context-Switching** | **Orphan-Process** | **Zombie-Process**. 
