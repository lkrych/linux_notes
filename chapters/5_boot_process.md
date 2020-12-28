# Boot Process - Kernel and User Space

## Table of Contents

## Kernel Boot

The kernel boot process is the series of steps that loads the kernel into memory.

A simplified high-level view of the process is:

1. The machine's BIOS or boot firmware loads and runs a boot loader.
2. The boot loader finds the kernel image on disk, loads it into memory, and starts it.
3. The kernel initializes the devices and its drivers.
4. The kernel mounts the root filesystem.
5. The kernel starts a program called `init` with a process ID of 1. This point is the user space start.
6. `init` sets the rest of the system process in motion.
7. At some point, `init` starts a process allowing you to log inn, usually at the end or near the end of the boot.

### Viewing kernel startup messages

There are two ways you can view the kernel's boot and runtime diagnostic messages:

1. Look at the kernel system log file. You'll often find this in `var/log/kern.log`.
2. Use the `dmesg` command.

### Kernel Initialization and Boot Options

Upon startup, the kernel initializes in this general order:

1. CPU inspection
2. Memory inspection
3. Device bus discovery
4. Device discovery
5. Auxiliary kernel subsystem setup (networking, etc.)
6. Root filesystem mount
7. User space start

When the kernel gets to devices, a question of dependencies arise. For example, the disk device drivers might depend upon bus support and SCSI subsystem support. 

When running the kernel, the boot loader passes in a set of text-based **kernel parameters** that tell the kernel how it should start. You can view the kernel parameters for your system using the following command:

```bash
lkrych@lkrych-VirtualBox:~$ cat /proc/cmdline 
BOOT_IMAGE=/boot/vmlinuz-5.4.0-42-generic root=UUID=9967eaed-1f42-4eaf-b0a6-6edd3a628bee ro quiet splash
```

The most critical parameter here is the **root parameter**. This is the **location of the root filesystem** and without it, the kernel cannot find init and therefore cannot perform the user space start.

### Boot Loaders

At the start of the boot process, before the kernel and init start, a boot loader starts the kernel. The **task of a boot loader** is simple, it **loads the kernel into memory**, and then **starts it with a set of kernel parameters**. Consider the following questions: where is the kernel, what parameters should be passed to the kernel?

The answers to these questions are that the kernel and its parameters are usually somewhere on the root filesystem. The only catch here is that the filesystem is not loaded yet.

Boot loaders use the **Basic Input/Output System (BIOS)** or **Unified Extensible Firmware Interface (UEFI)** to access disks. Nearly all disk hardware has firmware that allows the BIOS to access attached storage hardware. Most modern boot loaders can read partition tables and hae built-in support for read-only access to filesystems. Thus, they can find and read files.

The main bootloader that you will probably encounter is the Grand Unified Boot Loader (GRUB). One of GRUB's most important capabilities is filesystem navigation that allow for easier kernel image and configuration selection.

### How do Boot Loaders work?

There are two main mechanisms