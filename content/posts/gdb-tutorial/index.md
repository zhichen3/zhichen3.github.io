+++
title = "GDB Tutorial"
author = ["zhi"]
date = 2026-01-17T00:00:00-05:00
tags = ["gdb"]
categories = ["tutorial"]
type = "posts"
draft = false
weight = 1002
+++

[GDB](https://www.sourceware.org/gdb/), the GNU Project debugger, is a useful debugger for many
languages (C, C++, Fortran, etc). Here I keep a set of useful
commands for my records.


## Using GDB {#using-gdb}

To use GDB, compile the program with debugging information.
For standard C++ program, add `-g` flag when compiling, i.e.
something like

```bash
g++ -g -o myprogram myprogram.cpp
```

For `AMReX` related application codes like `Castro`, compile with
`DEBUG=TRUE`, i.e.

```bash
make DEBUG=TRUE
```

Now launch GDB with compiled program as the following.

```bash
gdb myprogram
```

Now inside gdb, set breakpoints to tell GDB where to pause.
You can either set by line number of function name

```nil
(gdb) break main         # Break at the beginning of the main function
(gdb) break 42           # Break at line 42
(gdb) break file.cpp:55  # Break at line 55 of a specific file
```

Now run the program via `run` command. If the program needs command-line
arguments, pass them after `run`, i.e. `inputs` in this case.

```nil
(gdb) run inputs
```

The program should stop at your breakpoints, use commands like `print` to
debug.


## Commands {#commands}

Here are a set of useful commands that I keep track of:

-   **run**: run the entire program from the beginning. Do \`run arg1 arg2 ...\`

-   **break**: set a break point at a specific line of the program.
    We can also set it at the function name

-   **list**: print out the lines of the code around where I am at if no
    input is given.
    You can also print out the code lines if you pass in the line number.

-   **print**: prints out the variable

-   **up**: Go up the call stack (the function that called the current function

-   **down**: Go down the call stack.

-   **display**: display variable for every command we run. Give the variable name.

-   **undisplay**: stop displaying the variable,
    the input must be the ID number of the corresponding variable.

-   **backtrace**: Print out the entire call stack.

-   **step**: unlike next, it will go inside the function call.

-   **continue**: continue running until the next break point.

-   **finish**: run until the end of the function.

-   **watch**: watch a variable, the it stops whenever the value of the
    watched value is changed.

-   **info**: verstaile command, if the argument is breakpoint, then it
    lists out the breakpoint and watch that we set before.

-   **delete**: deletes the breakpoint or watch point. Again the argument
    must be the ID given in the info. Without argument, it deletes
    all the breakpoints.

-   **whatis**: tells you the type of the variable.

-   **target record-full**: tell gdb to record everything onward.
    This allows us to run it reverse.

-   **reverse-next**: reverse next. need target record-full before.

-   **reverse-step**: reverse step

-   **reverse-continue**: reverse continue

-   **set var**: changes the value of the variable. Needs the variable
    name after var.
