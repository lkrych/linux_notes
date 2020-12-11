# A Gentle Introduction to Linux

## Table of Contents
* [OS's are like ogres are like onions](#oss-have-layers)

### OS's have layers

![](https://media.giphy.com/media/pyQV6sy5qOALu/giphy.gif)

Operating Systems use abstraction to split computing systems into components that are easy to work with. Components are arranged into **layers**.

There are three main layers in an OS.
1. **The Hardware** - Memory, the CPU, and Devices (disks, network interfaces, etc.)
2. **The kernel** - Software residing in memory that tells the CPU what to do. It manages the hardware and **acts as an interface between the hardware and any running program**.
3. **User Processes** - The running programs that the kernel manages.

Kernel processes run in kernel mode, user processes run in user mode. 

Code running in kernel mode has unrestricted access to the processor and main memory.

![](https://media.giphy.com/media/l0HlUJecJZbcdAl56/giphy.gif)
<figcaption>Code running in kernel mode</figcaption>

