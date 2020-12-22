# Disks and File Systems

## Table of Contents
* [Components of a Disk](#components-of-a-disk)
* [Partition Tables](#partition-tables)
    * [Viewing a partition table](#viewing-a-partition-table)
* [Hard Disks](#disks)
* [SSDs](#solid-state-disks)
* [Filesystems](#filesystems)
    * [What is a filesystem?](#what-is-a-filesystem?)
    * [Creating a filesystem](#creating-a-filesystem)
    * [Mounting a filesystem](#mounting-a-filesystem)
    * [Filesystem UUID](#filesystem-uuid)
    * [Buffering and Caching](#buffering-and-caching)
    * [fstab](#/etc/fstab)
    * [Filesystem capacity](#filesystem-capacity)
    * [Checking filesystems](#checking-filesystems)
* [Swap Space](#swap-space)
* [Summary](#summary)
## Components of a Disk

Disks are broken up into **partitions**. On Linux, they are denoted with a number after the whole block device, and therefore have device names like `/dev/sda1`. The **kernel presents each partition as a block device**, just as it would an entire disk. 

Partitions are defined on a small area of the disk called a **partition table**.

The next layer after the partition in a disk is the **filesystem**. The filesystem is the database of files and directories that are the interface for users interacting with the disk. If you want to access the data in a file, you need to get the appropriate location from the partition table and then search the filesystem database on that partition for the desired file data.

## Partition Tables

### Viewing a partition table

Use the `parted` command to view your partition table.

### Linux
```bash
lkrych@lkrych-VirtualBox:~$ parted -l
Warning: Unable to open /dev/sr0 read-write (Read-only file system).  /dev/sr0
has been opened read-only.
Error: /dev/sr0: unrecognised disk label
Model: VBOX CD-ROM (scsi)                                                 
Disk /dev/sr0: 60.6MB
Sector size (logical/physical): 2048B/2048B
Partition Table: unknown
Disk Flags: 
```

### Mac
```bash
~/linux_notes/chapters(main*) » diskutil list                                                                                                       127 ↵ lkrych@LKRYCH-M-W49D
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   1:                        EFI EFI                     314.6 MB   disk0s1
   2:                 Apple_APFS Container disk1         500.0 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +500.0 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD - Data     237.2 GB   disk1s1
   2:                APFS Volume Preboot                 82.0 MB    disk1s2
   3:                APFS Volume Recovery                528.9 MB   disk1s3
   4:                APFS Volume VM                      4.3 GB     disk1s4
   5:                APFS Volume Macintosh HD            11.3 GB    disk1s5
```
## Disks

Any device with moving parts introduces complexity into a software system because there are physical elements that are difficult or impossible to abstract. Even though we think of a hard disk as a block device with random access to any block, there are serious performance consequences if you aren't careful about how data is laid out onto the disk. This is mainly stuff that disk manufacturers think about, but it's good to be aware of.

## Solid State Disks

SSDs have no moving parts. Random access is not a problem because there is no head to sweep across a platter. One of the most significant factors affecting performance of SSDs is **partition alignment**.

When you read data from an SSD, you read it in chunks -- typically 4096 bytes at a time -- and the read must begin at a multiple of the same size. So if you're partition and its data do not lie on a 4096-byte boundary, you have to do two reads instead of one.

Many partitioning utilities include functionality to put newly created partitions at the proper offsets from the beginning of the disks, so you may not need to sorry about improper partition alignment. However, if you're curious about where your partition begins, you can find this information in the `sys/block`.

```bash
lkrych@lkrych-VirtualBox:/sys/block/sda$ cat sda1/start
2048
```

because this partition is not divisible by 4096, the partition would not be attaining optimal performance if it was on ann SSD. 

## Filesystems

The last link between the kernel and user space for disks is the filesystem. The filesystem is a form of database, it supplied the structure to transform a simple block device into the hierarchy of files and subdirectories you know and love.

Filesystems now perform a variety of tasks, such as system interfaces that you see in `/proc` adn `/sys`.

The **Virtual File System (VFS)** is an abstraction layer that completes the filesystem implementation. Much as the SCSI subsystem standardizes communication between different device types and kernel control commands, **VFS ensures that all filesystem implementations support a standard interface** so that user space applications access files and directories in the same manner.

### What is a filesystem?

A traditional Unix filesystem has two primary components: a **pool of data blocks** where you can **store data** and a **database system** that **manages the data pool**.

The database is centered around the **inode** data structure. An inode is a data structure that describes a particular file, including where it lives in the data pool, the files type, permissions, etc.

To view inode details, use the `ls -i` command.

```bash
~/linux_notes/chapters(main*) » ls -i                                                                                   
85918094 1_intro.md                 87321290 3_devices.md               87238300 resources
86138201 2_commands.md              87480018 4_disks_and_filesystems.md
```

The filesystem knows which blocks are in use and which are available using various data structures, the simplest one is the block bitmap.

### Creating a filesystem

Filesystem creation should only be a task that you do after adding a new disk or repartitioning an old one.

```bash
mkfs -t ext4 /dev/sdf2
```

`mkfs` is a frontend for a series of filesystem creation programs, `mkfs.fs`, where `fs` is a filesystem type.

The most common filesystem you will see is `ext4` which stands for Fourth Extended filesystem. This filesystem has [journaling](https://github.com/lkrych/aos_notes/blob/c2f184a5dec9b89f91df5fb3079fe125d486e0d1/lectures/review_file_systems.md#journaling) to enhance data integrity and hasten booting.

### Mounting a filesystem

On Unix, the **process of attaching a filesystem** is called **mounting**. When the system boots, the kernel reads some configuration data and mounts root(`/`) based onn the figuration data.

To mount a filesystem, you need to know:
1. The filesystem's device.
2. The filesystem type.
3. The mount point -  the place in the current system directory hierarchy where teh filesystem will be attached.

To learn the current filesystem status of your system, run `mount`.

```bash
~/linux_notes/chapters(main*) »mount                                                                                
/dev/disk1s5 on / (apfs, local, read-only, journaled)
devfs on /dev (devfs, local, nobrowse)
/dev/disk1s1 on /System/Volumes/Data (apfs, local, journaled, nobrowse)
/dev/disk1s4 on /private/var/vm (apfs, local, journaled, nobrowse)
map auto_home on /System/Volumes/Data/home (autofs, automounted, nobrowse)
/dev/disk1s3 on /Volumes/Recovery (apfs, local, journaled, nobrowse)
```

Each line corresponds to one currently mounted filesystem. The device is specified, then the mount point, finally the filesystem type and options are specified.

A filesystem can be unmounted using `umount`.

### Filesystem UUID

The method of mounting filesystems discussed above depends on device names. However, becauee device names can change, **filesystems are typically mounted by UUID**. This process is the preferred way to automatically mount filesystems in `etc/fstab` at boot time.

### Buffering and Caching

Linux **buffers writes to disk**. This means that the **kernel doesn't write changes immediately to filesystems** when a process requests changes. **Instead, it stores the changes in RAM** until the kernel can conveniently make the actual change. Usually this is done in batches for efficiency's sake.

In addition, the kernel has a series of mechanisms that use RAM to automatically cache blocks read from a disk. Therefore, if one or more processes repeatedly accesses a file, the kernel doesn't have to go to the disk again.

### /etc/fstab

To mount filesystems at boot time and remove some of the manual drudgery out of the `mount` command, Linux keeps a permanent list of filesystems and options in `etc/fstab`.

### Filesystem Capacity

To view the size and utilization of your currently mounted filesystem, use the `df` command.

```bash
~/linux_notes/chapters(main*) » df                                                                                   
Filesystem    512-blocks      Used Available Capacity iused      ifree %iused  Mounted on
/dev/disk1s5   976490568  22027592 479128968     5%  488346 4881964494    0%   /
devfs                671       671         0   100%    1162          0  100%   /dev
/dev/disk1s1   976490568 463337520 479128968    50% 4754896 4877697944    0%   /System/Volumes/Data
/dev/disk1s4   976490568  10487896 479128968     3%       6 4882452834    0%   /private/var/vm
map auto_home          0         0         0   100%       0          0  100%   /System/Volumes/Data/home
/dev/disk1s3   976490568   1032928 479128968     1%      54 4882452786    0%   /Volumes/Recovery
```

If the disk fills up and you need to know where all the space-hogging files are, use the `du` command.

```bash
~/linux_notes/chapters(main*) » du                                                                                   
880	./resources
968	.
```

### Checking Filesystems

For filesystems to work seamlessly, the kernel has to trust that there are nno errors in a mounted filesystem. If errors exist, data loss and system crashes might happen.

Filesystem errors typically happen because a user shuts down the computer in a rude way. Journaling helps prevent this, but you should still be cautious.

The tool to check a filesystem is `fsck`. `fsck` looks through a filesystem and checks that blocks and inodes are correctly aligned.  As a rule, you should never use `fsck` on a mounted filesystem because the kernel may alter the disk data as you run the check, causing runtime mismatches that could crash your system.

## Swap Space

Not every partition on a disk contains a filesystem. It's also **possible to augment RAM on a machine with disk space**. If you run out of real memory, **the Linux virtual memory system can automatically move pieces of memory to and from a disk storage**. This process is called **swapping**.

The disk area used to store memory pages is called **swap space**.

## Summary 

With disk-related components of a Unix system, the kernel handles raw block I/O from the devices and user-space tools can use the block I/O through device files. However, user-space typically uses the block I/O only for initializing operations. In normal use, user space uses only the filesystem support that the kernel provides on top of the block I/O.


