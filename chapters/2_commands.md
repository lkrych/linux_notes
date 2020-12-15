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


