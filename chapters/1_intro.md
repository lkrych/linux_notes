# A Gentle Introduction to Linux

## Table of Contents
* [OS's are like ogres are like onions](#oss-have-layers)
* [What does the kernel do?](#an-introduction-to-the-kernel)

### OS's have layers

![](https://media.giphy.com/media/pyQV6sy5qOALu/giphy.gif)

Operating Systems use abstraction to split computing systems into components that are easy to work with. Components are arranged into **layers**.

There are three main layers in an OS.
1. **The hardware** - Memory, the CPU, and Devices (disks, network interfaces, etc.)
2. **The kernel** - Software residing in memory that tells the CPU what to do. It manages the hardware and **acts as an interface between the hardware and any running program**.
3. **User processes** - The running programs that the kernel manages.

Kernel processes run in kernel mode, user processes run in user mode. 

**Code running in kernel mode has unrestricted access to the processor and main memory.** With great power comes great responsibility.

![](https://media.giphy.com/media/l0HlUJecJZbcdAl56/giphy.gif)

**User mode**, in comparison, **restricts access to a subset of memory as well as safe CPU operations**. If a process crashes, the consequences are limited and the process can be cleaned up by the kernel.

### An introduction to the kernel

