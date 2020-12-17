# Devices

## Table of Contents
* [Device Files](#device-files)
* [sysfs Device Path](#sysfs-device-path)
* [dd](#dd)

Our goal in this section is to introduce you to how you can extract information about devices attached to teh system. Later details will go into more detail about specific kinds of devices.

The **udev** system **enables user-space programs to automatically configure and use new devices**.

## Device Files

It is **easy to manipulate most devices** on a Unix system because **the kernel presents many of the device I/O interfaces** to user processes **as files**. This means that programmers can use regular file operations to interact with a device.

Device files can be found in the `/dev` directory. If you navigate to this directory and run `ls -l`, you will note that the first character of each mode varies. This first character denotes the file type. 

Four of the most common file types are `b, c, p and s`.

* **Block device (b)** - Programs **access data from a block device in fixed chunks**. Disks can be easily split up into blocks of data. Because a block devices total size is fixed and easy to index, **processes have random access to any block** in the device with help from the kernel.

* **Character device (d)** - Character devices **work with data streams**. You can only read characters from or write characters to character devices. They don't have a size. When you read or write to one, the kernel usually performs a read or write operation on the device. During character device interaction, the kernel cannot back up and reexamine the data stream after it has passed data. 

* **Pipe device (p)** - Named pipes are like character devices, with another process at the other end of the I/O stream instead of a kernel driver.

* **Socket device (s)** - Sockets are a special purpose interface that are frequently used for IPC. 

```bash
/dev » ls -l                                                                                                                              
total 0
crw-------  1 root    wheel           19,   1 Dec 15 08:57 afsc_type5
crw-------  1 root    wheel           10,   3 Dec 15 08:57 auditpipe
crw-r--r--  1 root    wheel            9,   3 Dec 15 08:57 auditsessions
crw-------  1 root    wheel           21,   0 Dec 15 08:59 autofs
crw-------  1 root    wheel           33,   0 Dec 15 08:59 autofs_control
```

The numbers before the dates in the lines above correspond to **the major and minor device numbers** that help the kernel identify the device. Similar devices usually have the same major number.

## sysfs Device Path

The traditional Unix `/dev` directory is a convenient way for user processes to reference and interface with devices supported by the kernel, but it is also simplistic. The name of the device is not very descriptive, and the kernel assigns devices in the order that they are found. This can lead to a device having a different name between a reboot.

To provide a **uniform view for attached devices**, the Linux kernel offers the **sysfs interface**. The base path for devices is `/sys/devices`. Comparing the `/sys/devices` file paths and the `/dev` is like comparing apples and oranges. The `/dev` file is there so that user processes can use the device. The `sys/devices` path is used to view information and manage the device.

Let's take a look at one of the `sys/devices` directories and you will see what I mean.

```bash
lkrych@lkrych-VirtualBox:/sys/devices/LNXSYSTM:00/LNXCPU:00$ ls
hid       path           power      thermal_cooling
modalias  physical_node  subsystem  uevent
```

The files and subdirectories here are **meant to be read primarily by programs rather than humans**. There are a few shortcuts in the `sys` directory. For examples, `sys/block` should contain all the block devices available on the system. However, these are just symbolic links, you can use `ls -l` to reveal the true sysfs paths.

It can be difficult to find sysfs locations of a device. You can use the `udevadm` command to show the path and other attributes.

```bash
lkrych@lkrych-VirtualBox:/sys$ udevadm info --query=all --name=/dev/sda
P: /devices/pci0000:00/0000:00:0d.0/ata3/host2/target2:0:0/2:0:0:0/block/sda
N: sda
S: disk/by-id/ata-VBOX_HARDDISK_VBb27b1526-ecdec596
S: disk/by-path/pci-0000:00:0d.0-ata-1
E: DEVLINKS=/dev/disk/by-id/ata-VBOX_HARDDISK_VBb27b1526-ecdec596 /dev/disk/by-path/pci-0000:00:0d.0-ata-1
E: DEVNAME=/dev/sda
E: DEVPATH=/devices/pci0000:00/0000:00:0d.0/ata3/host2/target2:0:0/2:0:0:0/block/sda
E: DEVTYPE=disk
#blah blah blah blah
```

We'll cover the udev system in more detail soon.

## dd 

The program `dd` is very useful when working with block and character devices. The **sole function of this program is to read from an input file or stream and write to an output file or stream**. It copies data in blocks of a fixed size. Here's an example of how to use dd with a character device.

```bash
lkrych@lkrych-VirtualBox:~$ ls
Desktop    Downloads         Music     Public     Videos
Documents  examples.desktop  Pictures  Templates
lkrych@lkrych-VirtualBox:~$ dd if=/dev/zero of=zeros bs=32 count=1
1+0 records in
1+0 records out
32 bytes copied, 0.000202533 s, 158 kB/s
lkrych@lkrych-VirtualBox:~$ ls
Desktop    Downloads         Music     Public     Videos
Documents  examples.desktop  Pictures  Templates  zeros
```

* if - input file
* of - output file
* bs - the block size. `dd` reads and writes this many bytes of data at a time.
* count - the total number of blocks to copy