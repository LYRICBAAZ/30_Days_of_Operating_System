# Day 3/30 — Multi-Tasking vs Multi-Threading

Day 2 introduced multitasking as one of the stages in the evolution of the OS. Day 3 zooms into it properly — and pairs it against a concept that's easy to confuse it with: **multithreading**. To get there cleanly, today starts one level below both of them, with the building blocks: **program, process, and thread**.

---

## Program → Process → Thread

### Program
A **program** is an executable file that contains a certain set of instructions written to complete a specific job or operation on your computer.

- It's **compiled code**, ready to be executed.
- It's **stored on disk**. The moment you double-click it, it stops being a program and becomes a **process**.

### Process
A **process** is simply a **program under execution**. While a program sits passively on disk, a process is active and **resides in the computer's primary memory (RAM)**.

### Thread
A **thread** is a **single sequence stream within a process** — an independent path of execution inside that process. It's often called a **light-weight process**.

> The main use of a thread is to achieve **parallelism**.

Threads let a process split its work into independent paths of execution that can run alongside each other. A classic example: multiple tabs in a browser, or a text editor — while you're typing, spell-checking, formatting, and saving are all happening **concurrently**, handled by separate threads of the same process.

**Worth remembering:** threading only pays off when there's more than one CPU to actually run things in parallel on. If the number of CPUs is 1, there's no significant gain from multithreading — the threads still have to take turns on that single CPU.

---

## Multi-Tasking vs Multi-Threading

| | Multi-Tasking | Multi-Threading |
|---|---|---|
| **Definition** | The execution of more than one task simultaneously. | A process is divided into several sub-tasks called threads, each with its own path of execution. |
| **What's being switched** | More than one **process** being context switched. | More than one **thread** being context switched. |
| **CPU requirement** | 1 CPU. | ≥1 CPU — better to have more than 1. |
| **Isolation & memory protection** | Exists — the OS must allocate separate memory and resources to each program the CPU is executing. | Doesn't exist — resources are shared among threads of the same process. The OS allocates memory to the process as a whole; its threads share that same memory and those same resources. |

The core distinction: multitasking juggles **separate, isolated processes**, each walled off from the others. Multithreading juggles **threads within one process**, which deliberately share memory and resources because they're all part of the same job.

---

## Thread Scheduling

Threads are scheduled for execution based on their **priority**. Even while executing within the process's runtime, every thread is assigned processor time slices by the operating system — the OS doesn't just let one thread run forever.

---

## Thread Context Switching vs Process Context Switching

| Thread Context Switching | Process Context Switching |
|---|---|
| OS saves the current state of a thread & switches to another thread **of the same process**. | OS saves the current state of a process & switches to another process by restoring its state. |
| Doesn't include switching of memory address space (though program counter, registers & stack *are* included). | Includes switching of memory address space. |
| Fast switching. | Slow switching. |
| CPU's cache state is **preserved**. | CPU's cache state is **flushed**. |

This is exactly why multithreading tends to be cheaper than multitasking: switching between threads of the same process skips the expensive part — there's no memory address space to swap out, and the CPU cache stays warm. Switching between entirely different processes means reloading a whole new memory context, which flushes the cache and costs more time.

---

## Key Takeaways

- The chain is **Program → Process → Thread**: a program is dormant code on disk, a process is that code running in RAM, and a thread is one independent path of execution inside a process.
- A thread's whole purpose is **parallelism** — but that purpose is only realized with more than one CPU.
- **Multitasking** = multiple isolated processes sharing a CPU. **Multithreading** = multiple threads of one process sharing that process's memory and resources.
- Process context switches are slow and flush the cache; thread context switches are fast and preserve it — because threads skip the memory-address-space swap that processes require.

---

**Next up — Day 4:** the **components of an OS** — breaking the operating system itself apart into the pieces that make it work.
