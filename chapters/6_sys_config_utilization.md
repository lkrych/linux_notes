# System Configuration and Resource Utilization

## Table of Contents
* [System Configuration](#system-configuration)

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