# Unix as IDE: Building

Posted on [February 13, 2012](http://blog.sanctum.geek.nz/unix-as-ide-
building/) by  [Tom Ryder](http://blog.sanctum.geek.nz/author/tom/)

Because compiling projects can be such a complicated and repetitive process, a
good IDE provides a means to abstract, simplify, and even automate software
builds. Unix and its descendents accomplish this process with a `Makefile`, a
prescribed recipe in a standard format for generating executable files from
source and object files, taking account of changes to only rebuild what's
necessary to prevent costly recompilation.

One interesting thing to note about `make` is that while it's generally used
for compiled software build automation and has many shortcuts to that effect,
it can actually effectively be used for any situation in which it's required
to generate one set of files from another. One possible use is to generate
web-friendly optimised graphics from source files for deployment for a
website; another use is for generating static HTML pages from code, rather
than generating pages on the fly. It's on the basis of this more flexible
understanding of software “building” that modern takes on the tool like
[Ruby’s `rake`](http://rake.rubyforge.org/) have become popular, automating
the general tasks for producing and installing code and files of all kinds.

## Anatomy of a `Makefile`

The general pattern of a `Makefile` is a list of variables and a list of
_targets_, and the sources and/or objects used to provide them. Targets may
not necessarily be linked binaries; they could also constitute actions to
perform using the generated files, such as `install` to instate built files
into the system, and `clean` to remove built files from the source tree.

It's this flexibility of targets that enables `make` to automate any sort of
task relevant to assembling a production build of software; not just the
typical parsing, preprocessing, compiling proper and linking steps performed
by the compiler, but also running tests (`make test`), compiling documentation
source files into one or more appropriate formats, or automating deployment of
code into production systems, for example, uploading to a website via a `git
push` or similar content-tracking method.

An example `Makefile` for a simple software project might look something like
the below:

    
    all: example
    
    example: main.o example.o library.o
        gcc main.o example.o library.o -o example
    
    main.o: main.c
        gcc -c main.c -o main.o
    
    example.o: example.c
        gcc -c example.c -o example.o
    
    library.o: library.c
        gcc -c library.c -o library.o
    
    clean:
        rm *.o example
    
    install: example
        cp example /usr/bin

The above isn't the most optimal `Makefile` possible for this project, but it
provides a means to build and install a linked binary simply by typing `make`.
Each _target_ definition contains a list of the _dependencies_ required for
the command that follows; this means that the definitions can appear in any
order, and the call to `make` will call the relevant commands in the
appropriate order.

Much of the above is needlessly verbose or repetitive; for example, if an
object file is built directly from a single C file of the same name, then we
don't need to include the target at all, and `make` will sort things out for
us. Similarly, it would make sense to put some of the more repeated calls into
variables so that we would not have to change them individually if our choice
of compiler or flags changed. A more concise version might look like the
following:

    
    CC = gcc
    OBJECTS = main.o example.o library.o
    BINARY = example
    
    all: example
    
    example: $(OBJECTS)
        $(CC) $(OBJECTS) -o $(BINARY)
    
    clean:
        rm -f $(BINARY) $(OBJECTS)
    
    install: example
        cp $(BINARY) /usr/bin

## More general uses of `make`

In the interests of automation, however, it's instructive to think of this a
bit more generally than just code compilation and linking. An example could be
for a simple web project involving deploying PHP to a live webserver. This is
not normally a task people associate with the use of `make`, but the
principles are the same; with the source in place and ready to go, we have
certain targets to meet for the build.

PHP files don't require compilation, of course, but web assets often do. An
example that will be familiar to web developers is the generation of scaled
and optimised raster images from vector source files, for deployment to the
web. You keep and version your original source file, and when it comes time to
deploy, you generate a web-friendly version of it.

Let's assume for this particular project that there's a set of four icons used
throughout the site, sized to 64 by 64 pixels. We have the source files to
hand in SVG vector format, safely tucked away in version control, and now need
to _generate_ the smaller bitmaps for the site, ready for deployment. We could
therefore define a target `icons`, set the dependencies, and type out the
commands to perform. This is where command line tools in Unix really begin to
shine in use with `Makefile` syntax:

    
    icons: create.png read.png update.png delete.png
    
    create.png: create.svg
        convert create.svg create.raw.png && \
        pngcrush create.raw.png create.png
    
    read.png: read.svg
        convert read.svg read.raw.png && \
        pngcrush read.raw.png read.png
    
    update.png: update.svg
        convert update.svg update.raw.png && \
        pngcrush update.raw.png update.png
    
    delete.png: delete.svg
        convert delete.svg delete.raw.png && \
        pngcrush delete.raw.png delete.png

With the above done, typing `make icons` will go through each of the source
icons files in a Bash loop, convert them from SVG to PNG using ImageMagick's
`convert`, and optimise them with `pngcrush`, to produce images ready for
upload.

A similar approach can be used for generating help files in various forms, for
example, generating HTML files from Markdown source:

    
    docs: README.html credits.html
    
    README.html: README.md
        markdown README.md > README.html
    
    credits.html: credits.md
        markdown credits.md > credits.html

And perhaps finally deploying a website with `git push web`, but only _after_
the icons are rasterized and the documents converted:

    
    deploy: icons docs
        git push web

For a more compact and abstract formula for turning a file of one suffix into
another, you can use the `.SUFFIXES` pragma to define these using special
symbols. The code for converting icons could look like this; in this case,
`$<` refers to the source file, `$*` to the filename with no extension, and
`$@` to the target.

    
    
    icons: create.png read.png update.png delete.png
    
    .SUFFIXES: .svg .png
    
    .svg.png:
        convert $< $*.raw.png && \
        pngcrush $*.raw.png $@
    

## Tools for building a `Makefile`

A variety of tools exist in the GNU Autotools toolchain for the construction
of `configure` scripts and `make` files for larger software projects at a
higher level, in particular
`[autoconf](http://en.wikipedia.org/wiki/Autoconf)` and
`[automake](http://en</code>.wikipedia.org/wiki/Automake)`. The use of these
tools allows generating `configure` scripts and `make` files covering very
large source bases, reducing the necessity of building otherwise extensive
makefiles manually, and automating steps taken to ensure the source remains
compatible and compilable on a variety of operating systems.

Covering this complex process would be a series of posts in its own right, and
is out of scope of this survey.

_Thanks to user samwyse for the `.SUFFIXES` suggestion in the comments._

  
[Unix as IDE](http://blog.sanctum.geek.nz/series/unix-as-ide/)

  * [Unix as IDE: Introduction](http://blog.sanctum.geek.nz/unix-as-ide-introduction/)
  * [Unix as IDE: Files](http://blog.sanctum.geek.nz/unix-as-ide-files/)
  * [Unix as IDE: Editing](http://blog.sanctum.geek.nz/unix-as-ide-editing/)
  * [Unix as IDE: Compiling](http://blog.sanctum.geek.nz/unix-as-ide-compiling/)
  * Unix as IDE: Building
  * [Unix as IDE: Debugging](http://blog.sanctum.geek.nz/unix-as-ide-debugging/)
  * [Unix as IDE: Revisions](http://blog.sanctum.geek.nz/unix-as-ide-revisions/)

This entry is part 5 of 7 in the series [Unix as
IDE](http://blog.sanctum.geek.nz/series/unix-as-ide/).

[<< Unix as IDE: Compiling](http://blog.sanctum.geek.nz/unix-as-ide-
compiling/)[Unix as IDE: Debugging >>](http://blog.sanctum.geek.nz/unix-as-
ide-debugging/)
