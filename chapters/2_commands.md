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


