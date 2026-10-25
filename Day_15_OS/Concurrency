# Day 15/30 — Introduction to Concurrency

Up to now, most of our scheduling discussion has been about one process at a time getting the CPU. Today we step into **concurrency**: what happens when several sequences of instructions make progress at the same time. Concurrency is the reason threads exist, and it's also the source of the trickiest bugs in operating systems. Tomorrow's topic, the critical section problem, starts exactly where today ends.

---

## What Is Concurrency?

> **Concurrency** is the execution of multiple instruction sequences at the same time.

In an operating system, it happens whenever **several process threads are running in parallel**. Your browser with ten tabs open, a text editor saving a file while you keep typing, a music player running alongside a download: all of these involve concurrent execution.

---

## Thread: A Quick Recap

The notes define a thread the same way we did on Day 3, and the definition is worth holding onto because everything in concurrency builds on it:

- A **single sequence stream** within a process.
- An **independent path of execution** inside a process.
- A **light-weight process**.
- Used to achieve **parallelism** by dividing a process's tasks into independent paths of execution.

**Example:** in a text editor, typing, spell checking, formatting, and saving can all run concurrently as separate threads of the same process.

---

## Thread Scheduling

Threads are scheduled for execution **based on their priority**. Even though threads run inside a process's runtime, the **OS assigns each thread processor time slices**. The thread scheduling algorithm decides which thread gets the CPU next.

---

## Thread Context Switching

When the OS switches from one thread to another, the notes list the key facts:

- The OS **saves the current state of a thread** and **switches to another thread of the same process**.
- It **does not** switch the **memory address space**, because the threads share it. The **program counter, registers, and stack** are still switched, because those are per-thread.
- It is **fast** compared with process switching.
- The **CPU's cache state is preserved**, because the threads work on the same data.

This is why thread switching is cheaper than process switching. We saw the same comparison table on Day 3.

---

## How Each Thread Gets Access to the CPU

The notes walk through the mechanism step by step:

1. **Each thread has its own program counter.** This tells the CPU which instruction the thread should run next.
2. **The thread scheduling algorithm decides** which thread gets the CPU, based on priority or time slices.
3. **The OS fetches the instructions** at the PC of that thread and executes them.

So even though the threads share code and data, each one is an independent path of execution with its own place in the program.

---

## I/O- or Time-Quantum-Based Context Switching

Thread switches can happen for the same reasons as process switches:

- A thread **waits for I/O**, so the OS switches to another thread.
- A thread's **time quantum expires**, so the OS switches to another thread.

To save and restore each thread's state, the OS keeps a **TCB (Thread Control Block)**. It works like the PCB from Day 9, but for threads: it stores the state a thread needs in order to resume.

---

## Will a Single-CPU System Gain from Multithreading?

The notes answer this directly:

- **Never.**
- On a single CPU, two threads have to **take turns** and **context switch** with each other. The CPU can still only do one thing at a time.
- Because of the switching overhead, **it won't give any gain**.

This is the most important point to remember. **Multithreading only gives a real speedup when there is more than one CPU** to run threads in parallel. On a single CPU, threads give you concurrency (interleaving), not parallelism.

---

## Benefits of Multithreading

The notes list four main benefits:

| Benefit | What it means |
|---|---|
| **Responsiveness** | An application can keep responding to the user while part of it is busy. For example, the UI thread stays responsive while a background thread downloads a file. |
| **Resource sharing** | Threads of the same process share memory and resources, which makes sharing efficient. |
| **Economy** | Creating and context switching threads is cheaper than doing the same with processes. Allocating memory and resources for process creation is costly, so it's better to divide tasks into threads of the same process. |
| **Scalability** | Threads let a program use **multiprocessor architectures** to a greater scale and with better efficiency. |

---

## Process vs Thread Memory: The Diagram

The notes include a diagram that shows how memory is laid out for each case.

```
        Process                      Thread (multiple threads, one process)

     ┌───────────┐                 ┌──────────────┐
     │   stack   │  ← single       │   stack 2    │  ← thread t2
     │           │    stack        │              │
     ├───────────┤                 ├──────────────┤
     │   heap    │                 │   stack 1    │  ← thread t1
     ├───────────┤                 │              │
     │   data    │                 ├──────────────┤
     ├───────────┤                 │   heap       │  ┐
     │   text    │                 ├──────────────┤  │ shared by all
     └───────────┘                 │   data       │  │ threads
                                   ├──────────────┤  │
                                   │   text       │  ┘
                                   └──────────────┘
```

- A **process** has **one stack**, plus its heap, data, and text.
- A **process with multiple threads** has **one stack per thread**, while the **heap, data, and text are shared** by all the threads.

This picture explains everything above. Each thread gets its own stack, so it has its own local variables and function calls. But all threads read and write the same heap and data. That shared memory is exactly what makes threads efficient, and also exactly what makes concurrency dangerous.

---

## Why Shared Memory Creates a Problem

Shared memory is the reason we need the next topic. If two threads read and update the same variable at the same time, the result can depend on which thread runs first, or on how their instructions interleave. The same program can produce different results on different runs. This is called a **race condition**, and it's the core of the **critical section problem** we'll study on Day 16.

---

## Key Takeaways

- **Concurrency** is the execution of multiple instruction sequences at the same time, and it happens in an OS when several threads run in parallel.
- A **thread** is a light-weight, independent path of execution inside a process. Threads are scheduled by **priority**, and each has its own **program counter**.
- **Thread context switching** saves thread state without switching the memory address space, so it's faster than process switching and keeps the cache warm.
- **A single-CPU system gains nothing from multithreading.** Real parallelism needs more than one CPU.
- **Threads share heap, data, and text**, but each has its own **stack**. This sharing is efficient, and it's also what creates race conditions.

---

**Next up — Day 16:** **The Critical Section Problem and How to Address It**: why shared data needs protection, the three requirements any solution must meet, and the standard approaches.
