# Day 8/30 — Storage Devices Basics: The Memory Hierarchy

Every program and every piece of data on a computer has to live somewhere. The catch is that no single storage option is fast, big, cheap, and permanent all at once. So computers use a **hierarchy** of storage: a few very fast, very small, very expensive layers near the CPU, and many slower, bigger, cheaper layers further away. Today we map out that hierarchy and understand why it's built the way it is.

---

## The Big Question: What Memory Does a Computer Have?

The notes start with a simple question: *what are the different memory present in the computer system?* The answer is laid out as a hierarchy, split into two groups.

```
                 Primary Memory                    Secondary Memory
        ┌──────────────────────────┐      ┌──────────────────────────────┐
        │ Register                 │      │ Electronic Disk              │
        │ Cache                    │      │ Magnetic Disk                │
        │ Main Memory (RAM)        │      │ Optical Disk                 │
        └──────────────────────────┘      │ Magnetic Tapes               │
                                          └──────────────────────────────┘
```

- **Primary memory** is what the CPU can access directly and quickly. It holds the data and instructions being used right now.
- **Secondary memory** is storage media where the computer keeps data and programs for the long term. The CPU can't work on it directly; data must be brought into primary memory first.

---

## Primary Memory

### 1. Register
- The **smallest unit of storage**.
- It is **part of the CPU itself**.
- A register may hold an **instruction**, a **storage address**, or **any data**, such as a bit sequence or individual characters.
- Registers quickly accept, store, and transfer the data and instructions the CPU is using **immediately**.

Registers are the fastest storage in the system. They're also the most expensive per byte, which is why there are only a handful of them.

### 2. Cache
- An **additional memory system** that temporarily stores **frequently used instructions and data**.
- Its purpose is **quicker processing** by the CPU. If the CPU needs something again soon, it can get it from cache instead of going to RAM.
- Cache sits between the registers and main memory in both speed and size.

### 3. Main Memory (RAM)
- The main memory is **RAM** (Random Access Memory).
- It holds the programs and data that are currently running, so the CPU can reach them directly.
- It's much bigger than the cache and registers, but slower.

### How Data Moves Up Through Primary Memory
When the CPU wants a piece of data, it looks in the fastest layer first:

```
Register  →  Cache  →  Main Memory (RAM)
(fastest)                 (slowest of these three)
```

This is why the cache and registers matter so much: they keep the CPU busy without waiting on slower memory.

---

## Secondary Memory

Secondary memory is the long-term storage layer. **storage media on which the computer can store data and programs**.

The four types listed are:

| Type | Notes |
|---|---|
| **Electronic disk** | Built from electronic chips with no moving parts. Works like flash storage. |
| **Magnetic disk** | Stores data on spinning magnetic platters. This is the traditional hard disk. |
| **Optical disk** | Stores data read by light, like CDs and DVDs. |
| **Magnetic tape** | Stores data sequentially on long magnetic tape. Used mainly for backup and archives. |

> **A quick link to earlier days:** the **HDD** (hard disk drive) is a magnetic disk, and the **SSD** (solid-state drive) is an electronic disk. Both are secondary storage, and both hold the files the kernel maps and backs up on Day 4.

---

## Primary vs Secondary: A Full Comparison

The notes compare the two groups on four properties.

### 1. Cost
- **Primary storage is costly.** Registers are the most expensive, due to expensive semiconductors and labour.
- **Secondary storage is cheaper** than primary storage, so it's the practical choice for holding large amounts of data.

### 2. Access Speed
- **Primary memory has higher access speed** than secondary memory.
- Among the primary layers, **registers are fastest, then cache, then main memory**.

### 3. Storage Size
- **Secondary memory has more space.** Primary memory is limited, so it can only hold what's being used now.

### 4. Volatility
- **Primary memory is volatile.** It loses its contents when the power goes off.
- **Secondary memory is non-volatile.** Data stays there after the computer is turned off.

| Property | Primary Memory | Secondary Memory |
|---|---|---|
| Cost | Costly | Cheaper |
| Access speed | Higher | Lower |
| Storage size | Small | Large |
| Volatility | Volatile | Non-volatile |
| Examples | Registers, cache, RAM | Electronic, magnetic, optical disks, tape |

---

## Why the Hierarchy Exists

Put all four properties together and the design logic becomes clear:

> Fast memory is expensive and small. Cheap memory is slow and large. A computer needs both, so it stacks them in layers: the fastest, smallest layer is closest to the CPU, and the largest, cheapest layer is furthest away.

The CPU uses the fast layers to keep working, while the big layers keep everything else safe and available. Most of the time, the system works well because the data the CPU needs next is already in a faster layer.

---

## Key Takeaways

- Memory is split into **primary** (register, cache, RAM), which the CPU can access directly, and **secondary** (electronic, magnetic, optical disks, and tape), which holds data long-term.
- **Registers** are the smallest and fastest, and they live inside the CPU. **Cache** holds frequently used data. **RAM** is the main working memory.
- Primary memory is **faster and costlier, but smaller and volatile**. Secondary memory is **slower and cheaper, but larger and non-volatile**.
- The **memory hierarchy** exists because no single storage type is fast, big, cheap, and permanent at once.

---

**Next up — Day 9:** **Introduction to Process**: what a process is, how it differs from a program, and what the OS tracks about it.
