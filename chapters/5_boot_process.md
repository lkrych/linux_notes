# Boot Process - Kernel and User Space

## Table of Contents
* [Kernel Boot](#kernel-boot)
    * [Viewing kernel startup messages](#viewing-kernel-startup-messages)
    * [Kernel initialization](#kernel-initialization-and-boot-options)
    * [Boot Loaders](#boot-loaders)
    * [How do boot loaders work?](#how-do-boot-loaders-work)
* [User Space Boot](#user-space-boot)

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

There are two main schemes by which PCs boot: **MBR or UEFI**.

The **Master Boot Record (MBR)** includes a small area that the PC BIOS loads and executes after its Power-on Self Test(POST). Unfortunately, this is too little storage to house almost any boot loader, so additional space is necessary. In this case, the initial piece of code in the MBR does nothing other than load the rest of the boot loader code.

PIC manufacturers and software companies decided that BIOS is severely limited, so they created **Unified Extensible Firmware Interface (UEFI)**. Booting is radically different on UEFI systems. Rather than executable boot code residing outside of a filesystem, there is always a special filesystem called the EFI System Partition (ESP), which contains a directory named `efi`. Each boot loader has its own identifier  and a corresponding subdirectory in the `efi` directory, ex: `efi/grub`.

## User Space Boot

The point where the kernel starts its first user-space process, `init`, is significant not only because it is **where the memory and CPU are ready for normal system operation**, but there's where the rest of the system builds. User space starts roughly in this order:

1. `init`
2. Essential low-level services such as `udevd` and `syslogd`
3. Network configuration
4. Mid- and high-level services (`cron`, etc.)
5. Login prompts, GUIs, and other high-level applications.

### Init

`init` is a user-space program like any other program on Linux. You can find it in `/sbin` with many other system binaries. It's main purpose is to **start and stop the essential service processes on the system**. There are three major implementations of `init` on Linux: `system V`, `systemd`, and `Upstart`.

There are many different implementations of `init` because `system V` and other **older versions** relied onn a sequence that performed **only one startup task at a time**. Under this system it is easy to resolve dependencies, however **the performance isn't terribly good** because two parts of the boot sequence cannot run at once.

`systemd`, and `Upstart` have attempted to remedy the performance issue by **allowing many services to start in parallel** thereby speeding up the boot process.