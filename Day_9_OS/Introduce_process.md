# Day 9/30 — Introduction to Process: From Program to Running Task

Up to now we've used the word "process" many times, from multitasking on Day 2 to thread scheduling on Day 3. Today we define it properly. A process is what a program becomes once it runs. We'll look at how the OS creates one, what memory a process uses, and the data structure the OS keeps for each process, the **Process Control Block (PCB)**.

---

## Program vs Process

These two words are easy to mix up, so the notes separate them clearly:

| | Program | Process |
|---|---|---|
| **Definition** | Compiled code that is ready to execute. | A program under execution. |
| **Where it lives** | Stored on disk. | Resides in primary memory (RAM). |
| **State** | Passive. It just sits there. | Active. It's doing work. |

A simple way to remember it: a program is a recipe written on paper, and a process is the cooking happening in the kitchen.

---

## How the OS Creates a Process

Converting a program into a process takes a sequence of steps:

1. **Load the program and static data into memory.** The compiled code and the global and static variables are copied from disk into RAM.
2. **Allocate the runtime stack.** Space is reserved for function calls, local variables, and return addresses.
3. **Heap memory allocation.** Space is reserved for memory the program requests while it runs.
4. **I/O tasks.** The OS sets up the input and output the process needs.
5. **Hand control to `main()`.** The OS passes control to the program's entry point, and the process starts executing its own code.

From step 5 onward, the program is no longer just a file. It's a process with its own memory and its own identity.

---

## The Architecture of a Process

A process's memory is divided into sections. The notes show four, stacked in memory:

```
   ┌──────────────────────────┐
   │ Stack                    │  Local variables, function arguments & return values
   │          ↓ grows down    │
   │                          │
   │          ↑ grows up      │
   │ Heap                     │  Dynamically allocated variables
   ├──────────────────────────┤
   │ Data                     │  Global & static data
   ├──────────────────────────┤
   │ Text (code)              │  Compiled code, loaded from disk
   └──────────────────────────┘
```

| Section | What it holds |
|---|---|
| **Text** (code) | The compiled machine code of the program, loaded from disk. |
| **Data** | Global and static variables. |
| **Heap** | Memory allocated dynamically while the program runs (for example, with `malloc` in C). |
| **Stack** | Local variables, function arguments, and return values. Every function call adds a frame here. |

> **A precision note:** in most systems the stack and heap grow toward each other in the address space, which is why the diagram shows them meeting in the middle. The exact layout varies by OS and architecture, but these four sections are the standard model.

Put simply, a process has two halves:

```
Process
 ├── Memory section  →  Code, Data, Stack, Heap
 └── Control information  →  PCB, Registers
```

The **memory section** is what the process uses to run. The **control information** is what the OS uses to manage it.

---

## Attributes of a Process

### Unique Identifier
Every process has a feature that identifies it uniquely: its **Process ID (PID)**. No two running processes share the same PID at the same time.

### The Process Table
The OS tracks all processes in a table-like data structure, called the **process table**. Each row in the table is a process entry, and each entry is a **PCB**.

The notes show a simple version:

| PID | PName |
|---|---|
| 1 | P1 |
| 2 | P2 |
| 3 | P3 |

> **Notes-based definition:** each block of the process table is called a **PCB**, or **Process Control Block**.

### The PCB
The **PCB** is the data structure the OS uses for each process. It stores all the information the OS needs to manage that process, such as its process ID, program counter, process state, and priority.

---

## The PCB Structure

Here are the fields in a PCB, as listed in the notes:

| Field | What it stores |
|---|---|
| **Process ID** | A unique identifier for the process. |
| **Program Counter (PC)** | The address of the next instruction the program will execute. |
| **Process State** | The current state of the process (we'll cover states in Day 10). |
| **Priority** | How important the process is. The priority decides how much CPU time it gets. |
| **Registers** | The saved values of the CPU registers for this process. |
| **List of open files** | The files this process currently has open. |
| **List of open devices** | The I/O devices this process currently uses. |

---

## Why Registers Are Saved in the PCB

This is the most important idea in the topic, so it's worth going through slowly.

The CPU has only one set of registers, but many processes take turns using it. So what happens when a process has to pause?

1. A process is running, and its **time slice expires**. Its turn on the CPU is over.
2. The **current values of its registers** are saved into its **PCB**.
3. The process is **swapped out**, and another process takes over the CPU.
4. Later, when the first process is **scheduled to run again**, its register values are **read from the PCB** and **written back into the CPU registers**.
5. The process continues exactly where it left off.

> **The main purpose of the registers in the PCB** is to make this pause-and-resume possible. The PCB is the process's saved "save game."

This is the same idea as the **context switch** we saw on Days 2 and 3. A context switch is what happens when this save and restore cycle runs between processes.

---

## Where Is the PCB Stored?

The notes add an important point: the **PCB is stored in main memory**. It has to be in RAM so the OS can read and update it quickly while it schedules processes.

---

## Key Takeaways

- A **program** is compiled code on disk. A **process** is that program running in memory.
- The OS creates a process by loading the code and data, allocating the stack and heap, setting up I/O, and handing control to `main()`.
- A process has four memory sections: **text (code), data, heap, and stack**.
- Each process is identified by a unique **Process ID**, and the OS tracks all processes in a **process table**.
- Each entry in the process table is a **PCB**, which stores the process's ID, program counter, state, priority, registers, open files, and open devices.
- The **registers saved in the PCB** let a process pause and resume without losing its place. The PCB itself lives in **main memory**.

---

**Next up — Day 10:** **Process State and Process Queues**: the life cycle of a process (new, ready, running, waiting, terminated) and how the OS keeps track of processes waiting their turn.
