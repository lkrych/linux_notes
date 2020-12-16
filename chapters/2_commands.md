# Commands and Directory Structure

## Table of Contents
* [Standard Input and Output](#standard-input-and-output)
* [Simple Unix Commands](#simple-unix-commands)
* [Directory Commands](#directory-commands)


## Standard Input and Output

Unix processes use I/O steams to read and write data. Standard input, standard output, and standard error are the most common I/O streams. The kernel gives each process a standard output. The `cat` command always writes to standard output. When you run it in the terminal, the standard output is connected to the terminal, so that's why you see the output there. I/O streams are flexible and you can easily manipulate them to read and write to places.

## Simple Unix Commands

| Command      | What it does | Example     |
| :---        |    ----   |          ---|
| `cat`      | prints the content of what follows it       |  `cat file1 file2`   |
| `ls`   | lists the contents of a directory        | `ls -l`     |
| `cp`   | copies files        | `cp file1 file2`     |
| `mv`   | renames ore moves a file        | `mv file1 file2`     |
| `touch`   | creates a file        | `touch file`     |
| `rm`   | deletes (removes) a file    | `rm file`     |
| `echo`   | prints arguments to standard output        | `echo Hello Darkness, my old friend`  |


## Directory Commands

| Command      | What it does | Example     |
| :---        |    ----   |          ---|
| `cd`   | change the shell's current working directory    | `cd dir`     |
| `mkdir`   | creates a new directory    | `mkdir dir`     |
| `rmdir`   | removes a directory (only if it is empty)    | `rmdir dir` |
| `rm -rf`   | removes a directory and everything inside of it    | `rm -rf dir` |


## Shell globbing

The shell can match simple patterns to file and directory names in a process known as **globbing**. The shell matches arguments containing globs, substitutes the filenames for those arguments, and then runs the revised command line. This substitution is called **expansion**.

```bash
grep . *.c
```

Will match any files in the current directory that have a `.c` extension.


## Intermediate Unix Commands

| Command      | What it does | Example     | Important Options |
| :---        |    ----   |          ---| ---- |
| `grep`      | prints the lines from a file or input stream that match an expression |  `grep file1 yaddayadda`   | `-i` (case-insensitive), `-v` (all non-matches) |
| `less`   | useful for reading a big file or output    | `less big_file` | `/word` to search for a word|
| `pwd` | outputs the name of the current working directory | `pwd` | n/a|
| `diff` | see the differennce between two files | `diff file1 file2` | n/a |
| `file` | gives file format | `file file1` | n/a |
| `find` | finds a file in a directory, `locate` can be faster unless the file is new | `find . somefile` | n/a |
| `locate` | like find, but uses an index that the system created so it is faster | `locate . somefile` | n/a |
| `head` and `tail` | quickly view either the top or the bottom of a file | `head file1` | `-n` (lines to print) |
| `sort` | puts the lines of a text file in alphanumeric order | `sort file1` | `-r` (reverses) |
| `passwd` | change your password | `passwd` | n/a |

## Environment and Shell Variables

The shell can store temporary variables called **shell variables**, containing text strings. It's pretty easy to assign a value to a variable, just use the equals sign.

```bash
SOMETHING=NOTHING
echo $SOMETHING
```

Shell variables are only accessible to the shell that defined them. **Environment variables** are not specific to the shell that defined them.

```bash
SOMETHING=NOTHING
export SOMETHING
```

The `PATH` is a special environment variable that contains the **command path**, a list of system directories tha the shell searches when trying to locate a command.

For example, when you run `cat`, the shell searches the directories listed in `PATH` for the `cat` program.

## Shell Keyboard Shortcuts

| Command      | Action |
| :---        |    ----   | 
| CTRL-A   | move cursor to the beginning of the line |
| CTRL-E   | move the cursor to the end of the line |
| CTRL-W   | erase the preceding word |
| CTRL-U   | erase from cursor to the beginning of the line |
| CTRL-K   | erase from cursor to end of line |

## Shell Input and Output

To send the output of a *command* to a file instead of the terminal, use the `>` redirection operator. The shell creates *file* if it doesn't already exist. If *file* exists, the shell erases the original file first. 

```bash
command > file
```

You can append the output to the file instead of overwriting it with the `>>` redirection operator.

```bash
command >> file
```

To send the standard output of a command to the standard input of another command, use the pipe operator `|`.

```bash
head file1 | grep somesearch
```

If you want to redirect the output of **standard error** you need to use some special syntax. The number 2 specifies the `stream ID` that the shell modifies.

```bash
ls /ffffff > f 2> e
```

## Listing, Manipulating, and Killing Processes

Each process on a linux system has a numeric process ID (PID). For a quick list of running processes, you can run `ps`.

```bash
$ ps                                     
  PID TTY      STAT  TIME CMD
 1683 ttys004  R     0:00.66 -zsh
```
* **PID** - The process ID
* **TTY** - The terminal device where the process is running
* **STAT** - the process status, S means, R means running.
* **TIME** - The amount of CPU time in minutes and seconds that the process has spent running instructions on the processor.
* **CMD** - The command running, be aware this can change.

### PS options

* `ps x` - show all of your running processes.
* `ps ax` - show all processes on the system, not just your own.
* `ps u` - include more detailed information on processes.
* `ps w` - show full command names, not just what fits on one line.


### Killing and Stopping Processes

To terminate a process, send it a signal with the `kill` command. 

```bash
kill pid
```

There are many types of signals, the most brutal way to kill a process is with the KILL command, or -9. Other signals give the process a chance to clean up after itself, but with KILL, the OS terminates the process and forcibly removes it.

A more gentle way to stop a process is to use the STOP signal. STOPped processes are still in memory and can be continued (CONT).

```bash
kill -STOP pid
# do something
kill -CONT pid
```

### Running a process in the background

You can use the `&` to push a process into the background of a terminal. The shell should respond by printing the PID of the new background process, and the prompt should return so that you can continue working. You can try to use `fg` to bring it back. 

```bash
./script_that_does_something_for_ten_minutes &
> PID
#do some work
# check in on script
fg
```

## File Modes and Permissions

Every Unix file has a set of **permissions** that determine where you can read, write or run the file. Running `ls -l` will display the permissions.

```bash
~/linux_notes/chapters(main*) » ls -l                      
-rw-r--r--  1 lkrych  staff  6532 Dec 13 10:10 1_intro.md
-rw-r--r--  1 lkrych  staff  7154 Dec 16 08:51 2_commands.md
```

The file's **mode** represents the file's permissions and some extra info. There are four points to the mode. Let's look at the mode for this chapter:  `-rw-r--r--`.
* The first character of the mode is the file type, a dash `-` denotes a regular file, meaning there is nothing special about the file. The rest of the file types are discussed in the next chapter.
* The rest of the file's mode contains the permissionns, which break into three sets: user, group, and other. 

The permissions above show that the file is readable and writable by the user, readable by the staff group, and everyone else, the other, have read permissions as well.

### Modifying Permissions

To change permissions, use the `chmod` command. First pick the set of permissions that you want to change, and then pick the bit to change. For example, add and remove the execution ability for `file`.

```bash
chmod u+x file
chmod u-x file
```

You will sometimes see people changing permissions with octal forms. See `chmod(1)` in the man pages for more info.

```bash
chmod 644 file
```

### Symbolic Links

A **symbolic link** is a file that points to another file or directory, creating an alias. They offer quick access to obscure directory paths. In a `ls` call, a symbolic link will have a file type of `l` in the file mode.

To create a symbolic link from target to linkname use:

```bash
ln -s target linkname
```

## Archiving and Compressing Files

The program `gzip` is one of the current standard Unix compression programs. A file ends with a `.gz` suffix is a GNU Zip archive.

```bash
gzip somefile
gunzip somefile.gz
```

Unlike the zip programs of other operating systems, **gzip does not create archives of files**. This means that it does not pack multiple files and directories into one file. To create ann archive, use `tar` instead.

```bash
tar cvf archive.tar file1 file2 ... #pack
tar xvc archive.tar #unpack
```

Archives created by tar usually have a `.tar` suffix. The `c` flag above activates create mode. The `v` flag activates verbose diagnostic output, the `f` flag denotes the file option. The `x` flag in the second command specifies the extract (unpack) mode.

## Linux Directory Hierarchy

The details of the Linux directory structure are highlighted in the [Filesystem Hierarchy Standard](https://www.pathname.com/fhs/).

<img src="./resources/fhs2.png">

Here are the most important subdirectories in root:

* **/bin** - Contains executables, including most of the basic Unix utilities like `cp` and `ls`.
* **/boot** - Contains kernel boot loader files.
* **/dev** - Contains device files.
* **/etc** - Core system configuration. Contains user password, boot, device, networking and other setup files. Many of these files are specific to the machines hardware.
* **/home** - Holds personal directories for regular users. 
* **/lib** - Holds library files that executables can use. This directory should contain only shared libraries.
* **/proc** - Provides system statistics through a browsable directory interface. It contains information about currently running processes and some kernel parameters.
* **/sys** - Provides a device and system interface, more about sys in the next chapter.
* **/sbin** - The place for system executables. Only root can execute these programs.
* **/tmp** - A storage area for small, temporary files. Any user can read and write from /tmp, but the user may not have permission to access another user's files there. If something is important, don't put it in /tmp.
* **/usr** - Contains a large directory hierarchy, including the bulk of the Linux system.
* **/var** - Where programs record runtime information. System logging, user tracking, caches, and other files that system programs create and manage are here. 


The `/usr` directory is where most user space programs and data reside. 

On linux systems the kernel is normally in `vmlinuz` or `/boot/vmlinuz`. A boot loader loads this file into memory and sets it in motion. when the system boots. Once the boot loader runs and sets the kernel in motion, the main kernel file is no longer used by the system. The kernel does however use modules on demand and these modules live in `/lib/modules`.



