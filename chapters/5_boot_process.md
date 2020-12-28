# Boot Process - Kernel and User Space

## Table of Contents
* [Kernel Boot](#kernel-boot)
    * [Viewing kernel startup messages](#viewing-kernel-startup-messages)
    * [Kernel initialization](#kernel-initialization-and-boot-options)
    * [Boot Loaders](#boot-loaders)
    * [How do boot loaders work?](#how-do-boot-loaders-work)
* [User Space Boot](#user-space-boot)
    * [init](#init)
    * [systemd](#systemd)
    * [Tracking Processes in systemd](#tracking-processes-in-systemd)
* [Shutting Down](#shutting-down)


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

`systemd`, and `Upstart` have attempted to remedy the performance issue by **allowing many services to start in parallel** thereby speeding up the boot process. Their implementations are quite different though, `systemd` is goal-oriented. The sysadmin defines a target that you want to achieve, along with its dependencies, and when you want to reach the target. `systemd` satisfies these deps and resolves the target. By contrast, `Upstart` receives events and runs jobs.

### Systemd

In addition to handling the regular boot process, `systemd` aims to incorporate a number of standard Unix services such as `cron` and `inetd`. It takes inspiration from Apple's `launchd`. The basic outline of how `systemd` works is as follows:

1. `systemd` loads its configuration.
2. `systemd` determines its boot goal, which is typically named `default.target`.
3. `systemd` determines all the dependencies of the default boot goal, dependencies of these dependencies and so on.
4. `systemd` activates the dependencies and the boot goal.
5. After boot, `systemd` can react to system events (such as `uevents`) and activate additional components.

It's important to remember that when starting services, `systemd` does not follow a rigid sequence.

One of `systemd`'s **most significant features is its ability to delay a unit startup until it is absolutely needed**.

As mentioned earlier, `systemd` can mount many different types of things on the system, not just processes and services. These include filesystems, timers, and network sockets. Each of these things is specified as a `unit type`. There are service units, mount units, and target units.

You can interact with `systemd` with the `systemctl` command, which allows you to activate and deactivate services, list status, reload the configuration, and much more.

The most essential basic command deals with obtaining unit information. To view a list of active units use `systemctl list-units`.

```bash
lkrych@lkrych-VirtualBox:~$ systemctl list-units
UNIT                          LOAD   ACTIVE SUB       DESCRIPTION              
proc-sys-fs-binfmt_misc.automount loaded active waiting   Arbitrary Executable File Formats F
sys-devices-pci0000:00-0000:00:01.1-ata1-host0-target0:0:0-0:0:0:0-block-sr0.device loaded ac
sys-devices-pci0000:00-0000:00:03.0-net-enp0s3.device loaded active plugged   82540EM Gigabit
sys-devices-pci0000:00-0000:00:05.0-sound-card0.device loaded active plugged   82801AA AC'97 
sys-devices-pci0000:00-0000:00:0d.0-ata3-host2-target2:0:0-2:0:0:0-block-sda-sda1.device load
sys-devices-pci0000:00-0000:00:0d.0-ata3-host2-target2:0:0-2:0:0:0-block-sda.device loaded ac
sys-devices-platform-serial8250-tty-ttyS0.device loaded active plugged   /sys/devices/platfor
sys-devices-platform-serial8250-tty-ttyS1.device loaded active plugged   /sys/devices/platfor
sys-devices-platform-serial8250-tty-ttyS10.device loaded active plugged   /sys/devices/platfo
sys-devices-platform-serial8250-tty-ttyS11.device loaded active plugged   /sys/devices/platfo
sys-devices-platform-serial8250-tty-ttyS12.device loaded active plugged   /sys/devices/platfo
```

A particularly useful `systemctl` operation is getting the status of a unit: `systemctl status some-unit`

```bash
lkrych@lkrych-VirtualBox:~$ systemctl status NetworkManager.service
● NetworkManager.service - Network Manager
   Loaded: loaded (/lib/systemd/system/NetworkManager.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2020-12-28 10:30:45 PST; 1h 1min ago
     Docs: man:NetworkManager(8)
 Main PID: 710 (NetworkManager)
    Tasks: 4 (limit: 4664)
   CGroup: /system.slice/NetworkManager.service
           ├─710 /usr/sbin/NetworkManager --no-daemon
           └─917 /sbin/dhclient -d -q -sf /usr/lib/NetworkManager/nm-dhcp-helper -pf /run/dhclient-enp0

Dec 28 10:30:51 lkrych-VirtualBox NetworkManager[710]: <info>  [1609180251.9942] device (virbr0): state
Dec 28 10:30:51 lkrych-VirtualBox NetworkManager[710]: <info>  [1609180251.9989] device (virbr0): Activ
Dec 28 10:30:51 lkrych-VirtualBox NetworkManager[710]: <info>  [1609180251.9999] device (virbr0-nic): s
Dec 28 10:30:52 lkrych-VirtualBox NetworkManager[710]: <info>  [1609180252.0004] device (virbr0-nic): s
Dec 28 10:30:52 lkrych-VirtualBox NetworkManager[710]: <info>  [1609180252.0038] device (virbr0-nic): A
Dec 28 10:30:52 lkrych-VirtualBox NetworkManager[710]: <info>  [1609180252.0618] device (virbr0-nic): s
Dec 28 10:30:52 lkrych-VirtualBox NetworkManager[710]: <info>  [1609180252.0620] device (virbr0): bridg
Dec 28 10:30:52 lkrych-VirtualBox NetworkManager[710]: <info>  [1609180252.0620] device (virbr0-nic): r
```

Other useful commands are `systemctl reload unit-name` or `systemctl daemon-reload`, where the first command reloads the configuration for a specific unit, whereas the second command will reload all unit configurations.

Requests to activate, reactivate and restart units are known as **jobs** in `systemd`, and they are essentially unit state changes. You can check the current jobs on a system with `systemctl list-jobs`.

### Tracking Processes in systemd

`systemd` wants some information about and to control every process that it starts. The main problem that it faces is that a service can start in different ways. It may fork new instances of itself, or even daemonize and a detach itself from the original process.

To minimize the work that a package developer or administrator needs to do, `systemd` uses **control groups (cgroups)**, a **Linux kernel feature that allows for finer tracking of a process hierarchy**. In `systemd`, you need to specify the `Type` option in your unit file to indicate its startup behavior. It can be simple, or forking.

### Shutting down

`init` controls how the system shuts down and reboots. The commands to shut down the system are always teh same regardless of which `init` implementation you use. The proper way to shutdown a Linux system is to use `shutdown`.

The shutdown process does the following:

1. `init` asks every process to shut down cleanly.
2. If a process doesn't respond after a while, `init` kills it, first trying a TERM signal.
3. If the TERM signal doesn't work, init uses the KILL signal.
4. The system locks system files into place and makes other preparations for shutdown.
5. The system unmounts all filesystems other than root.
6. The system remounnts the root filesystem read-only
7. The system writes all buffered data out to the filesystem with the `sync` program.
8. The final step is to tell the kernel to reboot or stop with the reboot system call. This can be done by `init` or an auxilliary program.