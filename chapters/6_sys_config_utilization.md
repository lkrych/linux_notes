# System Configuration and Resource Utilization

## Table of Contents
* [System Configuration](#system-configuration)
    * [System Logging](#system-logging)
    * [User Management Files](#user-management-files)
    * [Time](#time)
    * [Cron](#cron)
    * [Understanding User IDs](#understanding-user-ids)
    * [Authentication](#authentication)
* [Resource Utilization](#resource-utilization)
    * [Tracking Processes](#tracking-processes)
    * [Finding open files - lsof](#finding-open-files-with-lsof)
    * [Tracing Program Executionn](#tracing-program-execution)
    * [Threads](#threads)
    * [Measuring CPU time](#measuring-cpu-time)
    * [Process Priorities](#process-priorities)
    * [Load Averages](#load-averages)
    * [Memory](#memory)
    * [Page Faults](#page-faults)
    * [vmstat](#vmstat)
    * [iostat](#iostat)


## System Configuration

Most system configuration files on a Linux system are found in `/etc`. Historically, each program had onne of more configuration files there and because there are so many packages on a Unix system, `/etc` would accumulate too many files.

The most common practice now is to place system configuration files into subdirectories under `/etc`. The basic rules of what kind of configuration files are found in `/etc` are that they should be customizable configurations for a single machine, such as user information, and network details. 

### System Logging

Most system programs write their diagnostic output to the `syslog` service. When something goes wrong and you don't know where to start, check the system log files.

Most Linux distributions run a new version of `syslogd` called `rsyslogd`. This daemon does much more than write log messages to files, but today we will just talk about log messages. Here is an example of a log message:

```bash
lkrych@lkrych-VirtualBox:/var/log$ cat auth.log
Dec 28 11:05:08 lkrych-VirtualBox pkexec: pam_unix(polkit-1:session): session opened for user root by (uid=1000)
```

The base `rsyslogd` configuration file is `/etc/rsyslog.conf` A traditional rule has a **selector** and an **action**. A selector show how to catch logs, an action shows where to send them. There is also an extended syntax. This syntax usually begins with a `$`. One of the most common extensions allow syou to load additional configuration files. 

```bash
lkrych@lkrych-VirtualBox:/etc/rsyslog.d$ cat 50-default.conf 
#  Default rules for rsyslog.
#
#			For more information see rsyslog.conf(5) and /etc/rsyslog.conf

#
# First some standard log files.  Log by facility.
#
auth,authpriv.*			/var/log/auth.log
*.*;auth,authpriv.none		-/var/log/syslog
#cron.*				/var/log/cron.log
#daemon.*			-/var/log/daemon.log
kern.*				-/var/log/kern.log
#lpr.*				-/var/log/lpr.log
mail.*				-/var/log/mail.log
#user.*				-/var/log/user.log
```

The **selector** is a pattern that matches the **facility** and **priority** or log messages. The facility is a general category of message. Above you can see a facility for the kernel (`kern.*`), and for cron output (`#cron.*`). The priority follows the dot after the facility. The order of priorities from lowest to highest priority is:

1. debug
2. info
3. notice
4. warning
5. err
6. crit
7. alert
8. emerg

When you put a specific priority in a selector, `rsyslogd` **sends messages with that priority and all higher priorities to the destination** on that line.

### User Management Files

Unix systems allow for multiple independent users. At the kernel level, users are simply numbers (user IDs), **usernames only exist in user space**. Any program that works with usernames will need to be able to map a username to a user ID. 

The plaintext file `/etc/passwd` maps usernames to userIds.

```bash
lkrych@lkrych-VirtualBox:/etc/rsyslog.d$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
lkrych:x:1000:1000:lkrych,,,:/home/lkrych:/bin/bash
```

It has the following fields separated by colon (`:`).
1. The **username**.
2. The user's encrypted **password**. On most Linux systems, the password is not actually stored here, but in the shadow file. Normal users cannot read the Shadow file which is encrypted. An `x` in the field suggests that the password is in the shadow file.
3. The **user ID**.
4. The **group ID**.
5. The user's real name
6. The **user's home directory**
7. The **user's shell**

The `/etc/group` file defines the group IDs. 

```bash
lkrych@lkrych-VirtualBox:/$ cat /etc/group
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:syslog,lkrych
tty:x:5:
disk:x:6:
```

It has the following fields separated by colon (`:`).
1. The **group name**.
2. The group password. This is hardly ever used.
3. The **group ID**.
4. An optional list of users that belong to that group.

### Time

Unix machines depend on accurate timekeeping. The kernel maintains the **system clock**, which is the clock that is consulted when you run commands like `date`. Unfortunately, the kernel is terrible at keeping time. Machines that are on for a long time tend to develop **time/clock drift**, a difference between the actual time and the kernel time. The best solution to this problem is to use a network time daemon.

### Cron

The Unix `cron` service runs programs repeatedly on a fixed schedule. It is a commonly used admin tool. You can run any program with `cron` at whatever time suits you. The program running through `cron` is called a **cron job**.  To install a cron job, you'll create ann entry line in your **crontab file**. This can be done using the `crontab` command.

A cron entry looks like the following:

```bash 
15 09 * * * /home/lkrych/bin/hello
```

This cron entry schedules the hello binary to be run daily at 09:15 AM.

The fields are as follows:
1. Minute (0 through 59)
2. Hour (0 through 23)
3. Day of Month (1 through 31)
4. Month (1 through 12)
5. Day of the week (0 through 7)

Each user has is or her own crontab file. These can be found in `var/spool/cron/crontabs`. Users cannot write to this directory, they nee to use the `crontab` command instead.

* `crontab file` : install file as your current crontab
* `crontab -e`: edit your crontab
* `crontab -l` : list cron jobs
* `crontab -r` : remove the crontab

The **system's crontab** exists in `etc/crontab`. Don't use `crontab` to edit this file.

```bash
lkrych@lkrych-VirtualBox:/$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```
### Understanding User IDs

The `setuid` programs such as `sudo` and `su` allow you to change users. There are two ways to change a user ID in Linux systems, and the kernel handles both of them. The first is with the `setuid` executables and the second is through the `setuid()` family of system calls.

The kernel has three basic rules about what a process can or can't do:

1. A process running as root (userid 0) can use `setuid()` to become any other user.
2. A process not running as root has severe restrictions on how it may use `setuid()`.
3. Any process can execute a `setuid` program as long as it has adequate file permissions.

Every process has more than one user ID. There is the **effective user ID (euid)**, which **defines the access rights for a process**. The second user ID is the **real user ID (ruid)**, which **indicates who initiated a process**. 

On normal Linux systems, most processes have the same effective user ID and real user ID. You can use a custom `ps` call to view this information.

```bash
lkrych@lkrych-VirtualBox:/$ ps -eo pid,euser,ruser,comm
    PID EUSER    RUSER    COMMAND
    1 root     root     systemd
    2 root     root     kthreadd
    3 root     root     rcu_gp
    ...
    1363 lkrych   lkrych   systemd
    1364 lkrych   lkrych   (sd-pam)
    1377 lkrych   lkrych   gnome-keyring-d
```

Because the Linux kernel handles all user switches, systems developers and administrators need to be extremely careful with two things:
1. the programs that have `setuid` permissions. 
2. what those programs do.

Exploiting weaknesses in programs running as root is a primary method of systems intrusion.

### Authentication

The kernel doesn't know anything about authentication. In some systems, **Pluggable Authentication Modules (PAM)** are used to handle this duty.

## Resource Utilization

There are **three basic kinds of hardware resources**: **CPU, memory and I/O**. Processes vie for these resources, and the **kernel's job is to allocate resources fairly**.

Many of the tools that will be introduced are often thought of as performance-monitoring tools. They are also useful for understanding how the kernel works. 

### Tracking Processes

The `ps` command is useful because it displays the current processes running on the system. Unfortunately, **it does little to tell you how the processes are changing over time**. 

The `top` program displays the current system status as well as many of the fields in a `ps` listing. It also updates the display every second. Most importantly, it **shows the most active processes (those currently taking up the most CPU time)**. You can send commands to `top`.

```bash
lkrych@lkrych-VirtualBox:/$ top
top - 12:17:41 up  1:41,  1 user,  load average: 0.12, 0.06, 0.01
Tasks: 208 total,   1 running, 159 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.2 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  4030264 total,  2153888 free,   816652 used,  1059724 buff/cache
KiB Swap:   483800 total,   483800 free,        0 used.  2949728 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND    
 1595 lkrych    20   0 4064456 271520 102460 S   3.0  6.7   0:27.64 gnome-she+ 
 1773 lkrych    20   0 1026372  24724  19348 S   0.3  0.6   0:00.21 gsd-media+ 
 1964 lkrych    20   0  804840  38776  28348 S   0.3  1.0   0:03.27 gnome-ter+ 
 2634 lkrych    20   0   48884   3784   3144 R   0.3  0.1   0:00.23 top        
    1 root      20   0  160032   9196   6640 S   0.0  0.2   0:01.75 systemd    
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kthreadd

```

* **spacebar** - updates the display immediately.
* **M** - sorts by current resident memory usage.
* **T** - sorts by total (cumulative) CPU usage.
* **P** - sorts by current CPU usage (the default).
* **u** - displays only one user's processes.
* **f** - selects different statistics to display.

Two other utilities for linux offer more information, `atop` and `htop`.

### Finding Open Files with lsof

The `lsof` command **lists open files and the processes using them**. Because Unix places a lot of emphasis on files, `lsof` is among the most useful tools.

Let's take a look at the output of `lsof`.

```bash
h@lkrych-VirtualBox:/$ lsof | tail
lsof      2667               lkrych  txt       REG                8,1   163224        938 /usr/bin/lsof
lsof      2667               lkrych  mem       REG                8,1 10281936       3670 /usr/lib/locale/locale-archive
lsof      2667               lkrych  mem       REG                8,1   144976     138671 /lib/x86_64-linux-gnu/libpthread-2.27.so
lsof      2667               lkrych  mem       REG                8,1    14560     138561 /lib/x86_64-linux-gnu/libdl-2.27.so
lsof      2667               lkrych  mem       REG                8,1   464824     138660 /lib/x86_64-linux-gnu/libpcre.so.3.13.3
lsof      2667               lkrych  mem       REG                8,1  2030544     138538 /lib/x86_64-linux-gnu/libc-2.27.so
lsof      2667               lkrych  mem       REG                8,1   154832     138683 /lib/x86_64-linux-gnu/libselinux.so.1
lsof      2667               lkrych  mem       REG                8,1   170960     138510 /lib/x86_64-linux-gnu/ld-2.27.so
lsof      2667               lkrych    4r     FIFO               0,13      0t0      37644 pipe
lsof      2667               lkrych    7w     FIFO               0,13      0t0      37645 pipe
```

Here are what each column refers to:

1. **Command** - the command name for the process that holds the file descriptor
2. **PID** - the process ID
3. **User** - the user running the process
4. **FD** - This field can contain two types of elements. It can show the purpose of the file, it can also list the file descriptor of the open file.
5. **Type** - The file type (regular, directory, socket, etc.)
6. **Device** - The major and minor number of the device that holds the file.
7. **Size** - The file's size.
8. **Node** - The file's inode number.
9. **Name** - the filename.

There are two basic approaches to running `lsof`:
1. list everything and pipe the output to a command and then search for what you are looking for. This can take a while because `lsof` outputs A LOT of information.
2. Narrow down the list that `lsof`provides with command-line options.

You can use command line options to provide a filename as an argument and `lsof` will list only entries that match the arguments.
```bash
lsof some_file
```

Another option to is list the open files for a particular process ID
```bash
lsof -p PID
```

### Tracing Program Execution

The tools we've seen so far, `top` and `lsof`, examine active processes. However, if you have no idea why a program crashes immediately after starting up, even `lsof` won't help you because the process won't be active!

The `strace` (system call trace) and `ltrace` (library call trace) commands can **help you discover what a program attempts to do**. These tools produce EXTRAORDINARILY large amounts of output, but once you know what to look for, they can become useful.

A **system call** is a **privileged operation that a user space process asks the kernel to perform**. The `strace` tool prints all the system calls that a process makes. The `ltrace` command tracks shared library calls. It doesn't track anything at the kernel level. 

### Threads

A **thread** is similar to a process, it has a unique identifier, and the kernel schedules and runs threads just like processes. However, unlike separate processes, which do not share system resources, **all threads inside a single process share their system resources**. 

Many programs have only one thread. A process with multiple threads is known as multithreaded. **All processes start with a single thread**, this thread is known as the **main thread**.  The main thread starts new threads in order for the process to become multithreaded, this is similar to the way that a process can call `fork()` to start a new process.

The primary advantage of a multithreaded process is that when the process has a lot to do, **threads can run simultaneously on multiple processors**, **potentially speeding up computation**. Although you can also achieve this with multiple processes, **threads start faster than processes**, and it is often easier and **more efficient for threads to communicate** because they can use shared memory.

To view threads, you can use the `m` option with `ps`

```bash
lkrych@lkrych-VirtualBox:/$ ps m
  PID TTY      STAT   TIME COMMAND
 1381 tty1     -      0:00 /usr/lib/gdm3/gdm-x-session --run-script env GNOME_SHELL_SESSION_MODE=ubuntu gnome-session 
    - -        Ssl+   0:00 -
    - -        Ssl+   0:00 -
    - -        Ssl+   0:00 -
 1383 tty1     -      0:12 /usr/lib/xorg/Xorg vt1 -displayfd 3 -auth /run/user/1000/gdm/Xauthority -background none -n
    - -        Sl+    0:11 -
    - -        Sl+    0:00 -
    - -        Sl+    0:00 -
    - -        Sl+    0:00 -
    - -        Sl+    0:00 -
    - -        Sl+    0:00 -
    - -        Sl+    0:00 -
    - -        Sl+    0:00 -
    - -        Sl+    0:00 -
    - -        Sl+    0:01 -
```

Each line shows processes along with threads. Each line with a number in the PID column represents a process, as in the normal `ps` output. The lines with dashes in the PID column represent the threads associated with the process. 

If you woul dlike to view the thread IDs with `ps`, you can use a custom output format.

```bash
lkrych@lkrych-VirtualBox:/$ ps m -o pid,tid,command
  PID   TID COMMAND
 1381     - /usr/lib/gdm3/gdm-x-session --run-script env GNOME_SHELL_SESSION_MODE=ubuntu gnome-session --session=ubunt
    -  1381 -
    -  1382 -
    -  1396 -
 1383     - /usr/lib/xorg/Xorg vt1 -displayfd 3 -auth /run/user/1000/gdm/Xauthority -background none -noreset -keeptty
    -  1383 -
    -  1384 -
    -  1385 -
    -  1386 -
    -  1387 -
    -  1388 -
    -  1389 -
    -  1390 -

```

### Measuring CPU time

To monitor one or more specific processes over time, use the -p option with `top`.

```bash
lkrych@lkrych-VirtualBox:/$ top -p pid1 [-p pid2 ...]
```

To find out how much CPU time a command uses during its lifetime, use `time`. Beware that most shells have a built-in `time` command that doesn't provide extensive statistics, so you'll probably need to run `/usr/bin/time`

```bash
lkrych@lkrych-VirtualBox:/$ /usr/bin/time ls
bin   cdrom  etc   initrd.img	   lib	  lost+found  mnt  proc  run   snap  swapfile  tmp  var      vmlinuz.old
boot  dev    home  initrd.img.old  lib64  media       opt  root  sbin  srv   sys       usr  vmlinuz
0.00user 0.00system 0:00.00elapsed 100%CPU (0avgtext+0avgdata 2572maxresident)k
0inputs+0outputs (0major+108minor)pagefaults 0swaps
```

The key fields of this output are:

1. **User time** -  The number of seconds that the CPU has spent running the program's own code. On modern processors, some commands run so quickly, that the time rounds down to zero.

2. **System time** - How much time the kernel spends going the process's work (for example, reading files and directories).

3. **Elapsed time** - The total time it took to run the process from start to finish, including the time that the CPU spent doing other tasks.

### Process Priorities

You can change the way the kernel schedules a process in order to give the process more or less CPU time then other processes. The **kernel runs each process according to its scheduling priority**, which is a **number between -20 and 20, with -20 being the most priority**.

You can use `ps -l` to see the current priority of a process, but it's a little easier to see the priorities in action with the `top` command.

```bash
lkrych@lkrych-VirtualBox:/$ top
top - 12:17:41 up  1:41,  1 user,  load average: 0.12, 0.06, 0.01
Tasks: 208 total,   1 running, 159 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.2 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  4030264 total,  2153888 free,   816652 used,  1059724 buff/cache
KiB Swap:   483800 total,   483800 free,        0 used.  2949728 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND    
 1595 lkrych    20   0 4064456 271520 102460 S   3.0  6.7   0:27.64 gnome-she+ 
 1773 lkrych    20   0 1026372  24724  19348 S   0.3  0.6   0:00.21 gsd-media+ 
 1964 lkrych    20   0  804840  38776  28348 S   0.3  1.0   0:03.27 gnome-ter+ 
 2634 lkrych    20   0   48884   3784   3144 R   0.3  0.1   0:00.23 top        
    1 root      20   0  160032   9196   6640 S   0.0  0.2   0:01.75 systemd    
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.00 kthreadd

```

The PR column lists the kernel's current schedule priority for the process. The higher the number, the less likely the kernel is to schedule the process if other processes need CPU time. The schedule **priority alone does not determine the kernel's decision to give CPU time**. 

Next to the priority column is the **nice value (NI)**, which gives a hint to the kernel's scheduler. The **kernel adds the nice value to the current priority to determine the next time slot for the process**. By defaulou can t, this value is 0. To change the nice value, you can use the `renice` command.

```bash
renice 20 1595 # sets the value of NI for process 1595 to 20
```

If you're a superuser, you can set the nice value to a negative number. This feature is used when there are many tenants in a system (seems useful for cloud computing).

### Load Averages

**Load average** is the **average number of processes currently ready to run**. When thinking about load average, keep in mind that most processes on a system are waiting for input. This means that most processes are not ready to run and should contribute nothing to the load average.**Only processes that are actually doing something affect the load average**.

The `uptime` command tells you three load averages.

```bash
lkrych@lkrych-VirtualBox:/$ uptime
 16:23:32 up  3:04,  1 user,  load average: 0.20, 0.07, 0.02
```
The three numbers at the end are the l**oad averages for the past 1 minute, 5 minutes and 15 minutes**. 

**A high load average does not necessarily mean that your system is having trouble**. A system with enough memory and I/O resources can easily handle many running processes. If your system has a high load average, but it is still responding well, don't panic! 

However, if you sense the system is running slow and the load average is high, you might be running into memory performance problems. When the **system is low on memory**, the kernel can start to **thrash**, or **rapidly swap memory for processes to and from disk**. 

### Memory

One of the simplest ways to check your system's memory status is run the `free` command.

```bash
lkrych@lkrych-VirtualBox:/$ free
              total        used        free      shared  buff/cache   available
Mem:        4030264      884444     2069852       16216     1075968     2873440
Swap:        483800           0      483800

```

Performance problems can arise from memory shortages. The CPU has a **memory management unit (MMU)** that **translates the virtual memory addresses used by processes into real ones**. The kernel assists the MMU by breaking the memory used by processes into smaller chunks called **pages**. 

The kernel maintains a data structure, called a **page table**, that contains a **mapping of a processes' virtual page address to real page address** in memory. As a process accesses memory, the MMU translates the virtual address used by the process into real addresses based on the kernel's page table. 

A user process does not actually need all of its pages to be immediately available in order to run. The kernel generally **loads and allocates pages as a process needs them**, this is called **demand paging**. 

### Page Faults

If a memory page is not ready when a process wants to use it, the process triggers a **page fault**. In the event of a page fault, the kernel takes control of the CPU from the process in order to get the page ready.

There are two kinds of page faults, minor and major. A **minor page faults** occurs when a **desired page exists in main memory but the MMU doesn't know where it is**.  This can happen when the process requests more memory or when the MMU doesn't have enough space to store all the page locations for a process. In this case, the kernel tells the MMu about the page adn permits the process to continue to run.

A **major page fault** occurs when the **desired memory page isn't in main memory at all**, which means the **kernel must load it from disk**. A lot of major page faults will slow a system down because the kernel must do a substantial amount of work to provide the pages, robbing normal processes of their chance to run.

Some major page faults are unavoidable, such as those that occur when you load the code from disk when running a process for the first time. The biggest problems happen when you start running out of memory and the kernel starts to swap pages of working memory out to the disk in order to make room for new pages.

You can view page faults with `ps`, `top` and `time`.

```bash
lkrych@lkrych-VirtualBox:/$ /usr/bin/time cal > /dev/null
0.00user 0.00system 0:00.00elapsed 12%CPU (0avgtext+0avgdata 2476maxresident)k
64inputs+0outputs (1major+99minor)pagefaults 0swaps
```

```bash
lkrych@lkrych-VirtualBox:/$ ps -o pid,min_flt,maj_flt
  PID  MINFL  MAJFL
 1973   7226      1
 2888    160      0
```

Viewing page faults by process can help you zero in on certain problematic components

### vmstat

`vmstat` is a low-overhead tool that is useful for giving a **high-level view** of how often the kernel is **swapping pages in and out, busy the CPU is and I/O utilization**.

```bash
lkrych@lkrych-VirtualBox:/$ vmstat 5
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 2068436 125132 951172    0    0    18     2   28   48  0  0 100  0  0
 0  0      0 2068444 125132 951192    0    0     0     0  861 1878  1  1 98  0  0
```

The output falls into categories: 
* **procs** for processes 
* **memory** for memory usage 
* **swap** for pages pulled in and out of swap 
* **io** for disk usage 
* **system** for the number of times the kernel switches into kernel code
* **cpu** for the time used by different parts of the system

The parameter fed to the executable determines the interval (in seconds) for checking statistics. The `b` and `r` columns indicates whether a process is blocked or running.

### iostat

Like `vmstat`, `iostat` will give detailed statistics about io usage.

```bash
lkrych@lkrych-VirtualBox:/$ iostat
Linux 5.4.0-58-generic (lkrych-VirtualBox) 	12/31/2020 	_x86_64_	(4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.17    0.01    0.08    0.06    0.00   99.69

Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
loop0             0.05         0.08         0.00        988          0
loop1             0.00         0.00         0.00         44          0
loop2             0.00         0.03         0.00        332          0
loop3             0.83         0.85         0.00      10862          0
loop4             0.01         0.08         0.00       1077          0
loop5             0.01         0.09         0.00       1163          0
loop6             0.00         0.01         0.00        116          0
loop7             0.00         0.01         0.00        121          0
scd0              0.00         0.01         0.00        184          0
sda               3.35        75.00         9.77     953221     124200
```

tps is transfers per second, the rest are read and write data for the specific devices.

If you need to dig even deeper to see I/O resources used by individual processes, the `iotop` tool can help. It is similar to `top`, in that it continuously displays processes usign the most I/O, with a general summary on top.

I/O in Linux has the concept of **priorities**, it **affects how quickly the kernel schedules I/O reads and writes** for a process. In a priority such as `be/4`, the `be` part is a **scheduling class**, the number is the priority level. As with the CPU, lower numbers are higher priority.

You'll see three scheduling classes from `iotop`.

* **be - Best-effort**. The kernel does its best to fairly schedule I/O for this class. Most processes run under this class.
* **rt - Real-time**. The kernel schedules any real-time I/O before any other class of I/O no matter what.
* **idle** - The kernel performs I/O for this class only whenn no other I/O work needs to be done.j