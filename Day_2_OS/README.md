# Day 2/30 — The Many Faces of an Operating System: Types of OS

Day 1 answered *what* an OS is and *why* it exists. Day 2 is about *how* that idea evolved — operating systems didn't arrive fully formed; they grew in stages, each stage solving a limitation of the one before it. Today's notes trace that evolution through seven types of OS, each tied to a real system that pioneered it.

---

## What Is an OS Actually Trying to Achieve?

Before looking at the types, it helps to pin down the goals every OS is working toward, regardless of its era or design:

- **Maximum CPU utilization** — keep the processor doing useful work as much as possible, not sitting idle.
- **Less process starvation** — make sure no process waits forever while others hog the system.
- **Higher priority job execution** — important jobs should get a chance to run ahead of less important ones.

Every type of OS below is really just a different strategy for chasing these three goals — and each new type exists because the previous one fell short on at least one of them.

## The Seven Types, at a Glance

| # | Type | Pioneering System |
|---|------|--------------------|
| i | Single Process OS | MS-DOS, 1981 |
| ii | Batch-Processing OS | ATLAS, Manchester Univ. (late 1950s–early 1960s) |
| iii | Multiprogramming OS | THE, by Dijkstra (early 1960s) |
| iv | Multitasking OS | CTSS, MIT (early 1960s) |
| v | Multi-processing OS | Windows NT |
| vi | Distributed OS | LOCUS |
| vii | Real-Time OS | ATCS |

```
            Types of OS
                 │
   ┌─────┬───────┼───────┬─────┬──────┬─────┐
   │     │       │       │     │      │     │
Single Batch   Multi-  Multi- Multi- Dist-  Real
Process Proc.  prog.   task.  proc.  ributed Time
```

## 1. Single Process OS — One at a Time, the Old-School Way

The oldest approach: **only one process executes at a time from the ready queue.**

> The CPU handles one process, completes it (or waits for it), and *then* moves to the next process.

There's no juggling here — it's strictly sequential. Simple to build, but obviously inefficient: if that one process is waiting on I/O, the CPU just sits there doing nothing.

## 2. Batch-Processing OS — Group Similar Jobs, Run Them Together

This is where things get more organized. The workflow:

1. The user prepares their job using punch cards.
2. They submit the job to the computer operator.
3. The operator collects jobs from different users and **sorts them into batches** with similar needs.
4. The operator submits the batches to the processor **one by one**.
5. All the jobs within one batch are executed together.

```
User 1 → Job 1 ┐
User 2 → Job 2 ┼→ Operator → [Batch 2: Job7,Job5,Job2] → [Batch 1: Job4,Job1] → CPU + OS
User n → Job n ┘
```

Inside a batch, jobs still run strictly one after another (J1 → J2 → execute), but batching at least cuts down on manual, job-by-job handling by the operator.

**The catches:**
- Priorities can't be set — a high-priority job that arrives mid-batch just has to wait its turn.
- This can lead to **starvation**, since a batch might take a long time to fully complete.
- The CPU can still go idle whenever a job is doing I/O — batching doesn't fix that.

## 3. Multiprogramming OS — Don't Let the CPU Sit Idle

This is the first real fix for the "CPU idle during I/O" problem. **Multiprogramming increases CPU utilization by keeping multiple jobs (code and data) in memory at once**, so the CPU always has *something* to execute if the current job gets stuck waiting on I/O.

Key traits:
- Still a **single CPU**.
- **Context switching** between processes.
- The switch happens specifically **when the current process goes into a wait state** (e.g., it's blocked on I/O).
- Net effect: **CPU idle time is reduced**.

This is the first appearance of an idea that becomes central to all modern OS design: **context switching**, a.k.a. giving the illusion that multiple things are happening at once by rapidly swapping what the CPU is working on.

## 4. Multitasking OS — Multiprogramming, Taken Further

Multitasking is described as a **logical extension of multiprogramming** — same core idea, pushed further:

- Still a **single CPU**.
- Able to run **more than one task simultaneously** (in the sense of interleaving, not literal parallelism on one core).
- Uses both **context switching and time sharing**.
- **Increases responsiveness** — the system feels snappier because no single task hogs the CPU for too long.
- **CPU idle time is reduced even further** compared to plain multiprogramming.

The distinction worth holding onto: multiprogramming switches jobs mainly to avoid idling during I/O waits, while multitasking adds **time sharing** on top — actively slicing CPU time between tasks to keep everything moving and the system responsive to the user.

## 5. Multi-processing OS — More Than One CPU in the Picture

So far every type above still assumed a single CPU. Multi-processing OS breaks that assumption: **more than one CPU in a single computer.**

Benefits:
- **Increases reliability** — if one CPU fails, the other(s) can keep working.
- **Better throughput** — more work done per unit time.
- **Lesser process starvation** — if one CPU is busy with a process, another process can be executed on a different CPU at the same time.

This is the jump from *faking* parallelism (via fast switching on one CPU) to genuine parallel execution across multiple CPUs.

## 6. Distributed OS — Not Even on the Same Machine Anymore

Distributed OS pushes the idea of "multiple processors" past a single box entirely:

> A Distributed OS is an OS that manages multiple network-connected computers and coordinates their resources and processes so that they can work together as an integrated system.

A distributed OS manages many *bunches* of resources — ≥1 CPUs, ≥1 memory units, ≥1 GPUs — spread across **loosely connected, autonomous, interconnected computer nodes**. It's a collection of independent, networked, communicating, and *physically separate* computational nodes, coordinated to feel like one system.

```
       System 1
          │
   S5 ── O.S. ── S2
        /   \
      S4     S3
```

The OS here sits in the middle, coordinating multiple physically separate systems (S1 through S5 in the notes) as if they were one.

## 7. Real-Time OS (RTOS) — When Being Late Is Not an Option

The last type is built around a completely different priority: **correctness within strict time limits.**

> Real-time, error-free computations within tight-time boundaries.

This isn't about raw speed or throughput — it's about **guaranteed** response within a deadline, every single time. Classic examples: **air traffic control systems, robots**, and similar systems where a late or incorrect response could be catastrophic rather than just inconvenient.

## Key Takeaways

- Every OS type is chasing the same three goals: **maximum CPU utilization, minimal starvation, and respecting priority**.
- The evolution has a clear logic: *Single Process* (no overlap at all) → *Batch* (group similar work) → *Multiprogramming* (don't idle during I/O) → *Multitasking* (add time sharing for responsiveness) → *Multi-processing* (add real parallel hardware) → *Distributed* (spread across separate machines) → *Real-Time* (optimize for guaranteed deadlines instead of throughput).
- **Context switching** first shows up in multiprogramming and becomes a recurring theme from here on.
- Distributed and Real-Time OS solve fundamentally different problems than the rest — one is about *scale across machines*, the other about *timing guarantees*, not raw performance.

---

**Next up — Day 3:** going deeper into **multitasking and multithreading** — how a single process can itself be split into multiple threads, and how that's different from the task-level multitasking covered today.
