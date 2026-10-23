# Day 5/30 — System Calls: How Apps Talk to the Kernel

Day 4 showed the OS as two worlds: **user space**, where apps run without privileged access, and **kernel space**, where the kernel controls the hardware. Day 5 answers the obvious follow-up: if apps can't touch the hardware, how do they get anything done? The answer is the **system call**, the one controlled door between the two worlds.

---

## The Core Question: How Do Apps Interact with the Kernel?

Apps interact with the kernel **using system calls**.

> A **system call** is a way by which a user program requests a service from the kernel.

A system call is a mechanism that lets a user program request a service from the kernel for something it **doesn't have permission to do on its own**. User programs typically can't:

- Access I/O devices directly (disk, keyboard, network card).
- Communicate with other programs directly.
- Create or destroy processes, or change memory belonging to the system.

So the program asks, and the kernel decides whether and how to do it.

---

## Example 1: `mkdir` — Creating a Directory

When you run `mkdir projects` in a terminal, here's what happens:

1. The `mkdir` program runs in user space.
2. It **indirectly** calls the kernel and asks the **file management module** to create a new directory.
3. `mkdir` itself is just a **wrapper** around the actual system call. Its job is to package the request nicely.
4. Under the hood, it's a system call that goes through the kernel.

**Takeaway:** many familiar commands and library functions are not doing the work themselves. They're wrappers that call system calls.

---

## Example 2: Creating a Process

This example shows the full round trip between the two modes:

1. **User executes a process** — the request starts in user space.
2. **The program gets a system call** — it needs the kernel's help, so it makes the request.
3. **Exec system call to create a process** — the kernel, running in kernel space, creates the new process.
4. **Return to user space** — control goes back to the program, which carries on.

```
User space:    program ──(system call)──┐
                                        ▼
Kernel space:                  exec → create process
                                        │
User space:    program ◄──(return)──────┘
```

---

## Transitions Between User Mode and Kernel Mode

Moving from user space (US) into kernel space (KS) is done by a **software interrupt**. The CPU is essentially told "stop, switch to privileged mode, and run the kernel's handler for this request." When the kernel finishes, control returns to user mode.

Two key facts from the notes:

- **Transitions from US to KS are done by software interrupts.**
- **System calls are the only way through which a process can go from user mode into kernel mode.**


---

## Implementation Details

- **System calls are implemented in C.** That's why the C language sits so close to the OS in most systems.
- **glibc** (the GNU C Library) sits between the user app and the system call interface. It's a **C library** that provides many common functions, such as `printf`, `malloc`, and `fopen`, and those functions in turn use system calls. Programmers can write C code that uses these library functions instead of dealing with raw system calls.
- The **System Call Interface (SCI)** is the layer inside the kernel boundary that receives requests and dispatches each one to the correct kernel routine.

---

## The Layered Picture

```
┌────────────────────────┐
│        User App        │   ← user mode
├────────────────────────┤
│         glibc         │   ← C library (wrappers)
├────────────────────────┤
│   System Call Interface (SCI)  │ ← software interrupt boundary
├────────────────────────┤
│         Kernel         │   ← kernel mode
├────────────────────────┤
│        Hardware        │
└────────────────────────┘
```

A call from an app travels **down** through glibc and the SCI into the kernel, the kernel does the privileged work, and the result travels **back up** to the app.

---

## Types of System Calls

System calls are grouped into five categories based on what they do.

### 1. Process Control
- End, abort a process.
- Load and execute a program.
- Create a process, terminate a process.
- Get or set process attributes.
- Wait for time.
- Wait for an event, signal an event.
- Allocate and free memory.

### 2. File Management
- Create a file, delete a file.
- Open and close a file.
- Read, write, and reposition within a file.
- Get or set file attributes.

### 3. Device Management
- Request a device, release a device.
- Read, write, and reposition on a device.
- Get or set device attributes.
- Logically attach or detach devices.

### 4. Information Maintenance
- Get or set the time or date.
- Get or set system data.
- Get process, file, or device attributes.
- Set process, file, or device attributes.

### 5. Communication Management
- Create and delete a communication connection.
- Send and receive messages.
- Transfer status information.
- Attach or detach remote devices.

---

## Examples: Windows vs Unix System Calls

The same job has different names in different operating systems. The category stays the same; the function names change.

| Category | Windows | Unix |
|---|---|---|
| **Process Control** | `CreateProcess()`, `ExitProcess()`, `WaitForSingleObject()` | `fork()`, `exit()`, `wait()` |
| **File Management** | `CreateFile()`, `ReadFile()`, `WriteFile()`, `CloseHandle()`, `SetFileSecurity()`, `InitializeSecurityDescriptor()`, `SetSecurityDescriptorGroup()` | `open()`, `read()`, `write()`, `close()`, `chmod()`, `umask()`, `chown()` |
| **Device Management** | `SetConsoleMode()`, `ReadConsole()`, `WriteConsole()` | `ioctl()`, `read()`, `write()` |
| **Information Management** | `GetCurrentProcessID()`, `SetTimer()`, `Sleep()` | `getpid()`, `alarm()`, `sleep()` |
| **Communication** | `CreatePipe()`, `CreateFileMapping()`, `MapViewOfFile()` | `pipe()`, `shmget()`, `mmap()` |

**Spot the pattern:** Unix keeps its names short and lowercase (`read`, `write`, `open`), while Windows uses longer, descriptive, PascalCase names (`ReadFile`, `WriteFile`, `CreateFile`). The job is the same either way.

---

## Key Takeaways

- Apps can't touch hardware or other programs directly. They ask the kernel through **system calls**.
- A system call is the **only sanctioned route** from user mode into kernel mode, and it's triggered by a **software interrupt**.
- Many everyday commands (like `mkdir`) and library functions are **wrappers** around system calls.
- System calls are **implemented in C**, and **glibc** provides a friendlier C layer on top of them.
- There are **five categories**: process control, file management, device management, information maintenance, and communication management.
- Windows and Unix provide the same categories of services under different function names.

---

**Next up — Day 6:** **What happens when you turn on your computer?** — the boot process, from pressing the power button to the OS being ready for you.
