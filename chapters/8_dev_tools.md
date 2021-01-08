# Development Tools

## Table of Contents

## The C Compiler

The source code for Linux is written in C or C++. C programs follow a traditional development process, you write the programs, you compile them, you run them.

Most C programs are too large to reasonably fit inside a single source code file. Therefore developers group components of the source code together, giving each piece its own file. 

When compiling most .c files, you don't create an executable right away. Instead, use the compiler's `-c` option to create **object files**. 

```bash
cc -c main.c # will produce main.o
```

An **object file** is a ***binary file that a processor can almost understand***. Here are the caveats: The OS doesn't know how to run an object file, and second, you'll **likely need to combine several object files and some system libraries to make a complete program**.

To build a fully functioning executable from one or more object files, you must run the **linker**, the `ld` command in Unix. Programmers rarely run this program because the compiler knows how to run it. To create an executable called `doit` from two object files, you can use the compiler.

```bash
cc -o doit main.o aux.o
```