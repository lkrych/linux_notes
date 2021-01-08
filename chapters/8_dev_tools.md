# Development Tools

## Table of Contents
* [Compiling C Code](#compiling-c-code)
    * [header files](#header-files)
    * [C Preprocessor](#c-preprocessor)
    * [Linking with Libraries](#linking-with-libraries)
    * [Shared Libraries](#shared-libraries)
* [Make](#make)

## Compiling C Code

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

Although you can do it manually, it is recommended that you use the `make` system for managing compiles.

### Header Files

C **header files** are additional source code files that usually **contain type and library function declarations**. The default include directory in Unix is `/usr/include`. The compiler always looks there unless you explicitly tell it not to.

If you need to include a header from a nonstandard directory, you can use the `-I option`.

```bash
cc -c -I/usr/somerandomdir/include abinary.c
```

You should **be aware of includes that use double quotes**. Double quotes means that **the header file is not in a system include directory but that the compiler should otherwise search its include path**. It often means that the include file is in the same directory as the source file.

```c
#include "customheader.h"
```

### C Preprocessor

The **C compiler does not actually do the work of looking for all of these include files**. This **task falls to the C preprocessor (cpp)**, a program that the compiler runs on your source code before parsing the actual program. The **preprocessor rewrites source code into a form that the compiler understands**.

Preprocessor commands in the source code are called **directives**. They start with the `#` character. There are thee basic types of directives:

1. **Include files** - an `#include` directive instruct the preprocessor to include an entire file.
2. **Macro definitions** - a line such as `#define AVARIABLE 5` tells the preprocessor to substitute 5 for all occurrences of AVARIABLE in the source code.
3. **Conditionals** - You can mark out certain pieces of code with `#ifdef`, `#if`, and `#endif`. The `#ifdef MACRO` checks to see whether the preprocessor macro is defined, and `#if condition` tests to see whether the condition is nonzero.

### Linking with Libraries

The C compiler doesn't know enough about the system to create a useful program all by itself. Most programmers need **libraries** to build complete programs. **Libraries come into play primarily at link time**, when the linker program creates an executable from object files.

To include libraries, you must give the compiler the `-l` flag. If the library you want to include is named `gobject`, then the command looks like this:

```bash
cc -o main main.o -lgobject
```

You must tell the linker about nonstandard library locations. The paramter for this is `-L`. Let's say `gobject` lives in `/usr/somerandom/lib`

```bash
cc -o main main.o -L/usr/somerandom/lib -lgobject
```

### Shared Libraries

A library file ending with `.a` is called a **static library**. When you link a program against a static library, **the linker copies machine code from the library files into your executable**. Therefore, the final executable **does not need the original library file to run**.

One problem with static libraries is that they are often growing in size, and this makes using **static libraries wasteful in terms of disk space and memory**. In addition, if a static library is found to be insecure, there is no way to change any executable linked against it short of recompiling it. 

**Shared libraries** counter these problems. When you run a program linked to a shared library, the **system loads the shared library's code into the process memory space only when necessary**. Many processes can share the same shared library code inn memory. The downside is that shared libraries are harder to manage and have a complicated linking procedure.

A shared library has a suffix that contains `.so` (shared object). They usually reside in the same place as static libraries. The two standard library directories in Linux are `/lib` and `/usr/lib`. The `/lib` directory should not contain static libraries.

To see what shared libraries a program uses, run `ldd program`.

```bash
root@bc0a7eb17782: ldd /bin/bash
	linux-vdso.so.1 (0x00007fff429e9000)
	libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x00007f311558e000)
	libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f3115588000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3115396000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f31156ee000)
```

In the interest of performance and flexibility, executables don't typically know the locations of their shared libraries, only the names of them. A small program named `ld.so` (the runtime dynamic linker/loader) finds and loads shared libraries for a program at runtime. The first place the dynamic linker looks for shared libraries is in an executable's pre-configured runtime library search path (`rpath`). Then it looks in a system cache, `/etc/ld.so.cache`. 

## make

The traditional Unix compile management utility is make. make is a big system, but we will give a short overview of how it works here.

The basic idea behind make is the **target**. A **target is a goal you want to achieve**. A target can be a file (a `.o` file, an executable, etc.) or a label. In addition, **some targets depend on other targets**. These requirements are called **dependencies**.

To build a target, make follows a **rule**. Make already knows several rules. Let's take a look at a sample Makefile.

```makefile
#object files
OBJS=aux.o main.o

all: doit

doit: $(OBJS)
    $(CC) -o doit $(OBJS)
```

The first thing we see above is a macro definition of the variable `OBJS`. It is set to two object filenames.

The next item is the first target `all`. The first target is always the **default target**, the target that `make` wants to build when you run `make` by itself at the command line.

The **rule for building a target** comes after a colon (`:`). For `all`, this Makefile says that you need to satisfy something called `doit`. This is the **first dependency** in the file, `all` depends on `doit`.

To build `doit`, this Makefile uses the macro `$(OBJS)` in the dependencies. The macro expands to `aux.o main.o`, which indicates that `doit` depends on these two files existing.

This Makefile assumes that you have two C source files named `aux.c` and `main.c` in the same directory. Running `make` on the makefile yields the following output.

```bash
root@bc0a7eb17782:/home/example# make
cc    -c -o aux.o aux.c
cc    -c -o main.o main.c
cc -o doit aux.o main.o
```

So how does `make` know to go from `aux.c` to `aux.o`, `aux.c` isn't in the Makefile! The answer is that `make` here is following built-in rules. It knows to look for a `.c` file when you want a `.o` file.