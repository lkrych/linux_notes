# Disks and File Systems

## Table of Contents
* [Components of a Disk](#components-of-a-disk)
* [Partition Tables](#partition-tables)
    * [Viewing a partition table](#viewing-a-partition-table)

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