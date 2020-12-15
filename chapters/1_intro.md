# A Gentle Overview of Linux

## Table of Contents
* [OS's are like ogres are like onions](#oss-have-layers)
* [What does the kernel do?](#an-introduction-to-the-kernel)
    * [Process Management](#process-management)
    * [Memory Management](#memory-management)
    * [Device Management](#device-management)
    * [System Calls](#system-calls)
* [ Users](#users)

### OS's have layers

![](https://media.giphy.com/media/pyQV6sy5qOALu/giphy.gif)

Operating Systems use abstraction to split computing systems into components that are easy to work with. Components are arranged into **layers**.

There are three main layers in an OS.
1. **The hardware** - Memory, the CPU, and Devices (disks, network interfaces, etc.)
2. **The kernel** - Software residing in memory that tells the CPU what to do. It manages the hardware and **acts as an interface between the hardware and any running program**.
3. **User processes** - The running programs that the kernel manages.

Kernel processes run in kernel mode, user processes run in user mode. 

**Code running in kernel mode has unrestricted access to the processor and main memory.** With great power comes great responsibility.

**User mode**, in comparison, **restricts access to a subset of memory as well as safe CPU operations**. If a process crashes, the consequences are limited and the process can be cleaned up by the kernel.

### An introduction to the kernel

Main memory is one of the most important pieces of hardware in a computer. A CPU is simply an operator on memory -- it reads instructions and data from memory and writes data back out to memory. Why are we talking about memory in a kernel section? Nearly, everything the kernel does involves interacting with main memory.

The kernel manages four general system areas: **processes, memory, device drivers and system calls and their support**.

#### Process Management

Process management involves knowing how to start, pause, resume and terminate processes. Processes are instances of running programs and they are represented in memory in a process control block.

Processes run one-at-a-time on a processor for a fraction of a second, then another process is run for a fraction of a second and so on. Each piece of time is called a **time slice**. Time slices are so small that they imperceptible to the human eye -- the system appears to be running multiple processes at the same time (**multitasking**).

The act of loading a new process into the CPU is called **context switching** and it is the responsibility of the kernel. Here are the steps of a context switch:


1. The CPU (hardware) interrupts the current process based on an internal timer. It switches into kernel mode and hands control back to the kernel.
2. The kernel records the current state of the CPU and memory - this is essential for resuming the process that was interrupted.
3. The kernel handles any intervening tasks that might have come up during the preceding time slice like I/O operations.
4. The kernel is now ready to let another process run. It uses the scheduler to grab the next process.
5. The kernel prepares memory and the CPU for the process by copying the appropriate data into both. 
6. The kernel tells the CPU how long the time slice for this process will be.
7. The kernel switches the CPU into user mode and let's it rip.

![](https://i.stack.imgur.com/6h1xc.png)

`src: https://stackoverflow.com/a/17228664/4458404`

#### Memory Management

As we said earlier, managing memory is perhaps the most important thing the kernel has to do. It is also one of the more complex things that it has to do. Here's why: 

* The kernel must set aside it's own private area in memory that user processes can't access. 
* Each user process must have it's own section of memory that is independent and inaccessible from other user processes.
* User processes, when using IPC, should be able to share memory.
* The system can use more memory than physically exists by paging out to the disk.

Yeesh, this sounds complicated. The way linux handles this is by using a memory access scheme called **virtual memory**. When using this scheme, a process doesn't directly access the memory by its physical location in the hardware. Instead, the kernel sets up each process with an illusion - **an illusion that the process owns all of the memory on the machine**.

When the process accesses some of its memory, the OS sends the address that the process is using to the **memory management unit (MMU)**, a **hardware table in the CPU**. This table **translates between the virtual memory address that the process is using to the actual physical memory location on the machine**. This table acts as a cache for recently translated virtual memory addresses. If there is a miss on this cache, the kernel consults an OS-managed mapping, an abstraction known as the **page table**. The kernel must initialize and continuously maintain the page table.

The following figure is a little bit more detail than what was given above. All you need to know is that the TLB is the specific part of the MMU that maintains the cached mapping.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/b/be/Page_table_actions.svg/2880px-Page_table_actions.svg.png)

`src: https://en.wikipedia.org/wiki/Page_table`

#### Device Management

Because device implementation is heterogenous, it is has been the responsibility of the kernel to provide device drivers to present a uniform interface to user processes in order to simplify the application developer's job. 

#### System Calls

Because user processes don't have the free rein to interact with the hardware like the kernel does, the kernel exports an **API of actions that user processes can use to interact with the system**. This API is known as **system calls**. 

* **fork()** - when a process calls `fork()`, the kernel creates a nearly identical copy of the process.
* **exec()** - when a process calls `exec(program)`, the kernel starts program, replacing the current process.

Other than the initializationn, all user processes on a Linux system start as a result of a `fork()` call.

### Users

The Linux kernel supports the concept of a Unix user. A **user** is **an entity that can run processes and own files.** 

Users exist primarily to **support permissions and boundaries**. Every user-space process as a user `owner`, and processes are said to run as this owner.

Groups are a set of users. The primary purpose of groups is to allow a user to share file access to other users in a group.

