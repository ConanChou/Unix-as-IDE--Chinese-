# Unix as IDE: Debugging

Posted on [February 14, 2012](http://blog.sanctum.geek.nz/unix-as-ide-
debugging/) by  [Tom Ryder](http://blog.sanctum.geek.nz/author/tom/)

When unexpected behaviour is noticed in a program, Linux provides a wide
variety of command-line tools for diagnosing problems. The use of `gdb`, the
GNU debugger, and related tools like the lesser-known Perl debugger, will be
familiar to those using IDEs to set breakpoints in their code and to examine
program state as it runs. Other tools of interest are available however to
observe in more detail how a program is interacting with a system and using
its resources.

## Debugging with `gdb`

You can use `gdb` in a very similar fashion to the built-in debuggers in
modern IDEs like Eclipse and Visual Studio. If you are debugging a program
that you've just compiled, it makes sense to compile it with its _debugging
symbols_ added to the binary, which you can do with a `gcc` call containing
the `-g` option. If you're having problems with some code, it helps to also
use `-Wall` to show any errors you may have otherwise missed:

    
    $ gcc -g -Wall example.c -o example

The classic way to use `gdb` is as the shell for a running program compiled in
C or C++, to allow you to inspect the program's state as it proceeds towards
its crash.

    
    $ gdb example
    ...
    Reading symbols from /home/tom/example...done.
    (gdb)

At the `(gdb)` prompt, you can type `run` to start the program, and it may
provide you with more detailed information about the causes of errors such as
segmentation faults, including the source file and line number at which the
problem occurred. If you're able to compile the code with debugging symbols as
above and inspect its running state like this, it makes figuring out the cause
of a particular bug a lot easier.

    
    (gdb) run
    Starting program: /home/tom/gdb/example 
    
    Program received signal SIGSEGV, Segmentation fault.
    0x000000000040072e in main () at example.c:43
    43	   printf("%d\n", *segfault);

After an error terminates the program within the `(gdb)` shell, you can type
`backtrace` to see what the calling function was, which can include the
specific parameters passed that may have something to do with what caused the
crash.

    
    (gdb) backtrace
    #0  0x000000000040072e in main () at example.c:43

You can set breakpoints for `gdb` using the `break` to halt the program's run
if it reaches a matching line number or function call:

    
    (gdb) break 42
    Breakpoint 1 at 0x400722: file example.c, line 42.
    (gdb) break malloc
    Breakpoint 1 at 0x4004c0
    (gdb) run
    Starting program: /home/tom/gdb/example 
    
    Breakpoint 1, 0x00007ffff7df2310 in malloc () from /lib64/ld-linux-x86-64.so.2

Thereafter it's helpful to _step_ through successive lines of code using
`step`. You can repeat this, like any `gdb` command, by pressing Enter
repeatedly to step through lines one at a time:

    
    (gdb) step
    Single stepping until exit from function _start,
    which has no line number information.
    0x00007ffff7a74db0 in __libc_start_main () from /lib/x86_64-linux-gnu/libc.so.6

You can even attach `gdb` to a process that is already running, by finding the
process ID and passing it to `gdb`:

    
    $ pgrep example
    1524
    $ gdb -p 1524

This can be useful for [redirecting streams of
output](http://stackoverflow.com/questions/593724/redirect-stderr-stdout-of-a
-process-after-its-been-started-using-command-lin) for a task that is taking
an unexpectedly long time to run.

## Debugging with `valgrind`

The much newer [valgrind](http://valgrind.org/) can be used as a debugging
tool in a similar way. There are many different checks and debugging methods
this program can run, but one of the most useful is its Memcheck tool, which
can be used to detect common memory errors like buffer overflow:

    
    $ valgrind --leak-check=yes ./example
    ==29557== Memcheck, a memory error detector
    ==29557== Copyright (C) 2002-2011, and GNU GPL'd, by Julian Seward et al.
    ==29557== Using Valgrind-3.7.0 and LibVEX; rerun with -h for copyright info
    ==29557== Command: ./example
    ==29557==
    ==29557== Invalid read of size 1
    ==29557==    at 0x40072E: main (example.c:43)
    ==29557==  Address 0x0 is not stack'd, malloc'd or (recently) free'd
    ==29557==
    ...

The `gdb` and `valgrind` tools [can be used
together](http://valgrind.org/docs/manual/manual-core-adv.html#manual-core-
adv.gdbserver) for a very thorough survey of a program's run. Zed Shaw's
[Learn C the Hard Way](http://c.learncodethehardway.org/book/) includes a
really good introduction for elementary use of `valgrind` with a deliberately
broken program.

## Tracing system and library calls with `ltrace`

The `strace` and `ltrace` tools are designed to allow watching system calls
and library calls respectively for running programs, and logging them to the
screen or, more usefully, to files. On Linux, `ltrace` is preferred as it
enables you to log both system and library calls.

You can run `ltrace` and have it run the program you want to monitor in this
way for you by simply providing it as the sole parameter. It will then give
you a listing of all the system and library calls it makes until it exits.

    
    $ ltrace ./example
    __libc_start_main(0x4006ad, 1, 0x7fff9d7e5838, 0x400770, 0x400760
    srand(4, 0x7fff9d7e5838, 0x7fff9d7e5848, 0, 0x7ff3aebde320) = 0
    malloc(24)                                                  = 0x01070010
    rand(0, 0x1070020, 0, 0x1070000, 0x7ff3aebdee60)            = 0x754e7ddd
    malloc(24)                                                  = 0x01070030
    rand(0x7ff3aebdee60, 24, 0, 0x1070020, 0x7ff3aebdeec8)      = 0x11265233
    malloc(24)                                                  = 0x01070050
    rand(0x7ff3aebdee60, 24, 0, 0x1070040, 0x7ff3aebdeec8)      = 0x18799942
    malloc(24)                                                  = 0x01070070
    rand(0x7ff3aebdee60, 24, 0, 0x1070060, 0x7ff3aebdeec8)      = 0x214a541e
    malloc(24)                                                  = 0x01070090
    rand(0x7ff3aebdee60, 24, 0, 0x1070080, 0x7ff3aebdeec8)      = 0x1b6d90f3
    malloc(24)                                                  = 0x010700b0
    rand(0x7ff3aebdee60, 24, 0, 0x10700a0, 0x7ff3aebdeec8)      = 0x2e19c419
    malloc(24)                                                  = 0x010700d0
    rand(0x7ff3aebdee60, 24, 0, 0x10700c0, 0x7ff3aebdeec8)      = 0x35bc1a99
    malloc(24)                                                  = 0x010700f0
    rand(0x7ff3aebdee60, 24, 0, 0x10700e0, 0x7ff3aebdeec8)      = 0x53b8d61b
    malloc(24)                                                  = 0x01070110
    rand(0x7ff3aebdee60, 24, 0, 0x1070100, 0x7ff3aebdeec8)      = 0x18e0f924
    malloc(24)                                                  = 0x01070130
    rand(0x7ff3aebdee60, 24, 0, 0x1070120, 0x7ff3aebdeec8)      = 0x27a51979
    --- SIGSEGV (Segmentation fault) ---
    +++ killed by SIGSEGV +++

You can also attach it to a process that's already running:

    
    $ pgrep example
    5138
    $ ltrace -p 5138

Generally, there's quite a bit more than a couple of screenfuls of text
generated by this, so it's helpful to use the `-o` option to specify an output
file to which to log the calls:

    
    $ ltrace -o example.ltrace ./example

You can then view this trace in a text editor like Vim, which includes syntax
highlighting for `ltrace` output:

[![Vim session with ltrace output](http://blog.sanctum.geek.nz/wp-
content/uploads/2012/02/ltrace-vim.png)](http://blog.sanctum.geek.nz/wp-
content/uploads/2012/02/ltrace-vim.png)

Vim session with ltrace output

I've found `ltrace` very useful for debugging problems where I suspect
improper linking may be at fault, or the absence of some needed resource in a
`chroot` environment, since among its output it shows you its search for
libraries at dynamic linking time and opening configuration files in `/etc`,
and the use of devices like `/dev/random` or `/dev/zero`.

## Tracking open files with `lsof`

If you want to view what devices, files, or streams a running process has
open, you can do that with `lsof`:

    
    $ pgrep example
    5051
    $ lsof -p 5051

For example, the first few lines of the `apache2` process running on my home
server are:

    
    # lsof -p 30779
    COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
    apache2 30779 root  cwd    DIR    8,1     4096       2 /
    apache2 30779 root  rtd    DIR    8,1     4096       2 /
    apache2 30779 root  txt    REG    8,1   485384  990111 /usr/lib/apache2/mpm-prefork/apache2
    apache2 30779 root  DEL    REG    8,1          1087891 /lib/x86_64-linux-gnu/libgcc_s.so.1
    apache2 30779 root  mem    REG    8,1    35216 1079715 /usr/lib/php5/20090626/pdo_mysql.so
    ...

Interestingly, another way to list the open files for a process is to check
the corresponding entry for the process in the dynamic `/proc` directory:

    
    # ls -l /proc/30779/fd

This can be very useful in confusing situations with file locks, or
identifying whether a process is holding open files that it needn't.

## Viewing memory allocation with `pmap`

As a final debugging tip, you can view the memory allocations for a particular
process with `pmap`:

    
    # pmap 30779
    30779:   /usr/sbin/apache2 -k start
    00007fdb3883e000     84K r-x--  /lib/x86_64-linux-gnu/libgcc_s.so.1 (deleted)
    00007fdb38853000   2048K -----  /lib/x86_64-linux-gnu/libgcc_s.so.1 (deleted)
    00007fdb38a53000      4K rw---  /lib/x86_64-linux-gnu/libgcc_s.so.1 (deleted)
    00007fdb38a54000      4K -----    [ anon ]
    00007fdb38a55000   8192K rw---    [ anon ]
    00007fdb392e5000     28K r-x--  /usr/lib/php5/20090626/pdo_mysql.so
    00007fdb392ec000   2048K -----  /usr/lib/php5/20090626/pdo_mysql.so
    00007fdb394ec000      4K r----  /usr/lib/php5/20090626/pdo_mysql.so
    00007fdb394ed000      4K rw---  /usr/lib/php5/20090626/pdo_mysql.so
    ...
     total           152520K

This will show you what libraries a running process is using, including those
in shared memory. The total given at the bottom is a little misleading as for
loaded shared libraries, the running process is not necessarily the only one
using the memory; [determining “actual” memory usage for a given
process](http://stackoverflow.com/questions/118307/a-way-to-determine-a
-processs-real-memory-usage-i-e-private-dirty-rss) is a little more in-depth
than it might seem with shared libraries added to the picture.

  
[Unix as IDE](http://blog.sanctum.geek.nz/series/unix-as-ide/)

  * [Unix as IDE: Introduction](http://blog.sanctum.geek.nz/unix-as-ide-introduction/)
  * [Unix as IDE: Files](http://blog.sanctum.geek.nz/unix-as-ide-files/)
  * [Unix as IDE: Editing](http://blog.sanctum.geek.nz/unix-as-ide-editing/)
  * [Unix as IDE: Compiling](http://blog.sanctum.geek.nz/unix-as-ide-compiling/)
  * [Unix as IDE: Building](http://blog.sanctum.geek.nz/unix-as-ide-building/)
  * Unix as IDE: Debugging
  * [Unix as IDE: Revisions](http://blog.sanctum.geek.nz/unix-as-ide-revisions/)

This entry is part 6 of 7 in the series [Unix as
IDE](http://blog.sanctum.geek.nz/series/unix-as-ide/).

[<< Unix as IDE: Building](http://blog.sanctum.geek.nz/unix-as-ide-
building/)[Unix as IDE: Revisions >>](http://blog.sanctum.geek.nz/unix-as-ide-
revisions/)
