# Unix as IDE: Revisions

Posted on [February 15, 2012](http://blog.sanctum.geek.nz/unix-as-ide-
revisions/) by  [Tom Ryder](http://blog.sanctum.geek.nz/author/tom/)

Version control is now seen as an indispensable part of professional software
development, and GUI IDEs like Eclipse and Visual Studio have embraced it and
included support for industry standard version control systems in their
products. Modern version control systems trace their lineage back to Unix
concepts from programs such as `diff` and `patch` however, and there are
plenty of people who will insist that the best way to use a version control
system is still at a shell prompt.

In this last article in the [Unix as an IDE
series](http://blog.sanctum.geek.nz/series/unix-as-ide/), I'll follow the
evolution of common open-source version control systems from the basic
concepts of `diff` and `patch`, among the very first version control tools.

## `diff`, `patch`, and RCS

A central concept for version control systems has been that of the _unified
diff_, a file expressing in human and computer readable terms a set of changes
made to a file or files. The `diff` command was first released by Douglas
McIlroy in 1974 for the 5th Edition of Unix, so it's one of the oldest
commands still in regular use on modern systems.

A _unified diff_, the most common and interoperable format, can be generated
by comparing two versions of a file with the following syntax:

    
    $ diff -u example.{1,2}.c
    --- example.c.1    2012-02-15 20:15:37.000000000 +1300
    +++ example.c.2    2012-02-15 20:15:57.000000000 +1300
    @@ -1,8 +1,9 @@
     #include <stdio.h>
    +#include <stdlib.h> 
    
     int main (int argc, char* argv[])
     {
         printf("Hello, world!\n");
    -    return 0;
    +    return EXIT_SUCCESS;
     }

In this example, the second file has a header file added, and the call to
`return` changed to use the standard `EXIT_SUCCESS` rather than a literal `0`
as the return value for `main()`. Note that the output for `diff` also
includes metadata such as the filename that was changed and the last
modification time of each of the files.

A primitive form of version control for larger code bases was thus for
developers to trade `diff` output, called _patches_ in this context, so that
they could be applied to one another's code bases with the `patch` tool. We
could save the output from `diff` above as a patch like so:

    
    $ diff -u example.{1,2}.c > example.patch

We could then send this patch to a developer who still had the old version of
the file, and they could automatically apply it with:

    
    $ patch example.1.c < example.patch

A patch can include `diff` output from more than one file, including within
subdirectories, so this provides a very workable way to apply changes to a
source tree.

The operations involved in using `diff` output to track changes were
sufficiently regular that for keeping in-place history of a file, the [Source
Code Control System](http://en.wikipedia.org/wiki/Source_Code_Control_System)
and the [Revision Control
System](http://en.wikipedia.org/wiki/Revision_Control_System) that has pretty
much replaced it were developed. RCS enabled "locking" files so that they
could not be edited by anyone else while "checked out" of the system, paving
the way for other concepts in more developed version control systems.

RCS retains the advantage of being very simple to use. To place an existing
file under version control, one need only type `ci <filename>` and provide an
appropriate description for the file:

    
    $ ci example.c
    example.c,v  <--  example.c
    enter description, terminated with single '.' or end of file:
    NOTE: This is NOT the log message!
    >> example file
    >> .
    initial revision: 1.1
    done

This creates a file in the same directory, `example.c,v`, that will track the
changes. To make changes to the file, you _check it out_, make the changes,
then _check it back in_:

    
    $ co -l example.c
    example.c,v  -->  example.c
    revision 1.1 (locked)
    done
    $ vim example.c
    $ ci -u example.c
    example.c,v  <--  example.c
    new revision: 1.2; previous revision: 1.1
    enter log message, terminated with single '.' or end of file:
    >> added a line
    >> .
    done

You can then view the history of a project with `rlog`:

    
    $ rlog example.c
    
    RCS file: example.c,v
    Working file: example.c
    head: 1.2
    branch:
    locks: strict
    access list:
    symbolic names:
    keyword substitution: kv
    total revisions: 2;	selected revisions: 2
    description:
    example file
    ----------------------------
    revision 1.2
    date: 2012/02/15 07:39:16;  author: tom;  state: Exp;  lines: +1 -0
    added a line
    ----------------------------
    revision 1.1
    date: 2012/02/15 07:36:23;  author: tom;  state: Exp;
    Initial revision
    =============================================================================

And get a patch in unified `diff` format between two revisions with `rcsdiff
-u`:

    
    $ rcsdiff -u -r1.1 -r1.2 ./example.c
    ===================================================================
    RCS file: ./example.c,v
    retrieving revision 1.1
    retrieving revision 1.2
    diff -u -r1.1 -r1.2
    --- ./example.c	2012/02/15 07:36:23	1.1
    +++ ./example.c	2012/02/15 07:39:16	1.2
    @@ -4,6 +4,7 @@
     int main (int argc, char* argv[])
     {
         printf("Hello, world!\n");
    +    printf("Extra line!\n");
         return EXIT_SUCCESS;
     }

It would be misleading to imply that simple patches were now in disuse as a
method of version control; they are still very commonly used in the forms
above, and also figure prominently in both centralised and decentralised
version control systems.

## CVS and Subversion

To handle the problem of resolving changes made to a code base by multiple
developers, _centralized version systems_ were developed, with the [Concurrent
Versions System
(CVS)](http://en.wikipedia.org/wiki/Concurrent_Versions_System) developed
first and the slightly more advanced
[Subversion](http://en.wikipedia.org/wiki/Apache_Subversion) later on. The
central feature of these systems are using a _central server_ that contains
the repository, from which authoritative versions of the codebase at any
particular time or revision can be retrieved. These are termed _working
copies_ of the code.

For these systems, the basic unit of the systems remained the _changeset_, and
the most common way to represent these to the user was in the archetypal
`diff` format used in earlier systems. Both systems work by keeping records of
these changesets, rather than the actual files themselves from state to state.

Other concepts introduced by this generation of systems were of _branching_
projects so that separate instances of the same project could be worked on
concurrently, and then merged into the mainline, or _trunk_ with appropriate
testing and review. Similarly, the concept of _tagging_ was introduced to flag
certain revisions as representing the state of a codebase at the time of a
release of the software. The concept of the `merge` was also introduced;
reconciling conflicting changes made to a file manually.

## Git and Mercurial

The next generation of version control systems are _distributed_ or
_decentralized_ systems, in which working copies of the code themselves
contain a complete history of the project, and are hence not reliant on a
central server to contribute to the project. In the open source, Unix-friendly
environment, the standout systems are Git and Mercurial, with their client
programs `git` and `hg`.

For both of these systems, the concept of communicating changesets is done
with the operations `push`, `pull` and `merge`; changes from one repository
are accepted by another. This decentralized system allows for a very complex
but tightly controlled ecosystem of development; Git was originally developed
by Linus Torvalds to provide an open-source DVCS capable of managing
development for the Linux kernel.

Both Git and Mercurial differ from CVS and Subversion in that the basic unit
for their operations is not changesets, but complete files (blobs) saved using
compression. This makes finding the log history of a single file or the
differences between two revisions of a file slightly more expensive, but the
output of `git log --patch` still retains the familiar unified `diff` output
for each revision, some forty years after `diff` was first being used:

    
    commit c1e5559ddb09f8d02b989596b0f4100ad1aab422
    Author: Tom Ryder <tom@sanctum.geek.nz>
    Date:   Thu Feb 2 01:14:21 2012
    
        Changed my mind about this one.
    
    diff --git a/vim/vimrc b/vim/vimrc
    index cfbe8e0..65a3143 100644
    --- a/vim/vimrc
    +++ b/vim/vimrc
    @@ -47,10 +47,6 @@ set shiftwidth=4
     set softtabstop=4
     set tabstop=4
    
    -" Heresy
    -inoremap <C-a> <Home>
    -inoremap <C-e> <End>
    -
     " History
     set history=1000

The two systems have considerable overlap in functionality and even in command
set, and the question of which to use provokes [considerable
debate](http://stackoverflow.com/questions/35837/what-is-the-difference-
between-mercurial-and-git). The best introductions I've seen to each are [Pro
Git](http://progit.org/) by Scott Chacon, and [Hg Init](http://hginit.com/) by
Joel Spolsky.

## Conclusion

This is the last post in the [Unix as IDE
series](http://blog.sanctum.geek.nz/series/unix-as-ide/); I've tried to offer
a rapid survey of the basic tools available just within a shell on Linux for
all of the basic functionality afforded by professional IDEs. At points I've
had to be not quite as thorough as I'd like in explaining certain features,
but to those unfamiliar to development on Linux machines this will all have
hopefully given some idea of how comprehensive a development environment the
humble shell can be, and all with free, highly mature, and standard software
tools.

  
[Unix as IDE](http://blog.sanctum.geek.nz/series/unix-as-ide/)

  * [Unix as IDE: Introduction](http://blog.sanctum.geek.nz/unix-as-ide-introduction/)
  * [Unix as IDE: Files](http://blog.sanctum.geek.nz/unix-as-ide-files/)
  * [Unix as IDE: Editing](http://blog.sanctum.geek.nz/unix-as-ide-editing/)
  * [Unix as IDE: Compiling](http://blog.sanctum.geek.nz/unix-as-ide-compiling/)
  * [Unix as IDE: Building](http://blog.sanctum.geek.nz/unix-as-ide-building/)
  * [Unix as IDE: Debugging](http://blog.sanctum.geek.nz/unix-as-ide-debugging/)
  * Unix as IDE: Revisions

This entry is part 7 of 7 in the series [Unix as
IDE](http://blog.sanctum.geek.nz/series/unix-as-ide/).

[<< Unix as IDE: Debugging](http://blog.sanctum.geek.nz/unix-as-ide-
debugging/)
