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
