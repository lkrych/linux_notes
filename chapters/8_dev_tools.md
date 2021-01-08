# Development Tools

## Table of Contents
* [Compiling C Code](#compiling-c-code)

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