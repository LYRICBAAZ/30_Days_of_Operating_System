# Day 6/30 — What Happens When You Turn On Your Computer? (Firmware & the Boot Process)

Days 1 to 5 all assumed the OS was already running. Day 6 goes back to the very beginning: you press the power button, the screen is still black, and nothing is in control yet. How does a dead machine turn into a working operating system? The answer has two parts: **firmware**, the low-level software living inside hardware, and the **boot process**, the step-by-step chain that hands control from firmware to the OS.

---

## Part 1: Firmware — The Software Inside the Hardware

**Firmware** is a specific type of software that is **permanently or semi-permanently stored inside a piece of hardware**. It tells the hardware *how to work*. It is low-level software that controls the hardware, and it is usually stored in **non-volatile memory** on the device, so it survives power loss.

### Why firmware matters: the keyboard example

When you press the letter **A** on a keyboard, several things must happen, and the keyboard has to handle them itself:

1. Which key was pressed? (The keyboard must detect it.)
2. How is that key press turned into a signal? (The keyboard generates the electrical signal.)
3. What data should be sent to the computer? (The keyboard decides the message to transmit.)

These basic instructions are handled by the **firmware stored inside the keyboard**. The computer never has to know the keyboard's internal details; it just receives the data.

### More firmware examples

| Device | What its firmware does |
|---|---|
| **Router** | Manages network and device settings, and handles how packets are forwarded. |
| **SSD / HDD** | Operates the storage controller, which reads and writes the actual data on the drive. |
| **BIOS / UEFI** | Helps the motherboard start the boot process. |

**Key idea:** firmware is the bridge between a hardware component and the rest of the system. It's the hardware's own "operating instructions," written in software and burned into the chip.

---

## Part 2: The Boot Process — From Power Button to OS

Here's the sequence, in order. Think of it as a relay race where each runner hands the baton to the next.

### Step 1: PC On
You press the power button. Power flows to the components, and the CPU starts.

### Step 2: CPU Initializes and Finds the Firmware
The CPU initializes itself and looks for a firmware program called the **BIOS** (Basic Input/Output System). The BIOS lives on a **BIOS chip**, a **ROM chip on the motherboard** that allows access to and setup of the computer at the most basic level.

- In **modern PCs**, the CPU loads **UEFI** (Unified Extensible Firmware Interface) instead. UEFI is the newer replacement for BIOS.

### Step 3: BIOS/UEFI Runs POST
The firmware runs the **POST**, or **Power-On Self-Test**:

- It **tests and initializes the system hardware** (CPU, RAM, storage, and so on).
- It **loads configuration settings** (the setup you can change in the BIOS menu).
- If something is wrong, such as **missing RAM**, it throws an **error** and the boot process **stops**.

> **A note on UEFI:** UEFI can do much more than just initialize hardware. It is really a tiny operating system in its own right. For example, Intel CPUs have the **Intel Management Engine**, which provides features including remote management of business PCs.

### Step 4: Handoff to the Bootloader
Once the hardware is ready, the firmware **hands responsibility for booting** to the OS's **bootloader**. Here is how it finds it:

- **Legacy BIOS:** the BIOS looks at the **MBR (Master Boot Record)**, a special **boot sector at the very beginning of a disk**. The MBR contains a small piece of code.
- **UEFI:** the firmware looks for a small program on an **EFI System Partition** instead.

In both cases, the firmware **executes the small bootloader program it found**.

### Step 5: The Bootloader Loads the OS
The **bootloader** is a small program with one big job: **boot the rest of the operating system**. It loads the **kernel** first, and then the **user space** comes up on top of it.

Different systems use different bootloaders:

| OS | Bootloader |
|---|---|
| Windows | Windows Boot Manager (`Bootmgr.exe`) |
| Most Linux systems | GRUB |
| macOS | `boot.efi` |

---

## The Whole Chain at a Glance

```
Power button
   │
   ▼
CPU initializes → finds firmware (BIOS chip / UEFI)
   │
   ▼
POST: test & initialize hardware, load config
   │   (missing RAM? → error, boot stops)
   ▼
Firmware looks for bootloader (MBR or EFI System Partition)
   │
   ▼
Bootloader (Bootmgr / GRUB / boot.efi)
   │
   ▼
Kernel loads  →  User space starts  →  You see the desktop
```

---

## Key Takeaways

- **Firmware** is software stored inside hardware (usually in non-volatile memory) that tells that hardware how to work, like the keyboard firmware that turns a key press into a signal.
- The **BIOS** (or the newer **UEFI**) lives on a chip on the motherboard and is the first code the CPU runs.
- **POST** tests and initializes the hardware. If something essential is missing, the boot stops with an error.
- The firmware hands control to a **bootloader**, found in the **MBR** (BIOS) or the **EFI System Partition** (UEFI).
- The bootloader loads the **kernel**, and then the **user space** comes up. That's the moment the OS we studied in Days 1 to 5 takes over.

---

**Next up — Day 7:** **32-bit vs 64-bit operating systems**: what the "bit" actually means, and why it matters for memory and performance.
