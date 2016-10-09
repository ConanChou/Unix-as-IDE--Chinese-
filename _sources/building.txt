构建
====

编译项目有时是个复杂而重复的过程。一个好的集成开发环境可以提供一种简单、高效甚至自动化的软件编译方法。Unix 及其衍生系统用 ``Makefile`` 
来完成这个工序。这是一种标准格式的文档，是用来把源代码和目标文件编译成可执行文件的“菜谱”。它能够考虑源文件的修改决定只编译必要的文件，以此避免重复编译的浪费。

关于 ``make`` 还有个非常有趣的特点值得注意，就是虽然它通常用于自动化编译而且它为此提供了不少捷径，但是其实凡是把一堆文件生成为另一堆文件的情况都可以利用它。一种可能的用法是，网站部署时将原图片优化成网页友好的图片；另一种可以是从代码生成静态 HTML 页面，而非运行时生成页面（译者注：利用 github 建博客就是这个原理）。正是基于这样一种更宽泛的“编译”概念，一些现代的此类工具（如 `Ruby's rake <http://rake.rubyforge.org/>`_ ）才得到广泛地应用于自动化一些普通流程，生产和安装各种代码和文件。

剖析 ``Makefile``
-----------------

``Makefile`` 的一般格式包含一系列变量、一系列目标，以及用来生产目标的源和／或对象。目标也不一定是链接了的二进制文件；它们也可以包含操作产生文件的动作，比如 ``install`` 用来把生成的编译文件部署到系统里，或者用 ``clean`` 从源代码树里清除已编译的文件。

正是这种目标产物的灵活性使得 ``make`` 可以自动化任何与生成产品软件相关的任务；不仅仅是编译器执行的典型的语法分析、预处理、编译和连接步骤，还包括运行测试（ ``make test`` ），或把文档的源文件编译成一种或多种适当的格式，或将代码自动部署到产品系统里，比如通过 ``git push`` 或类似的内容跟踪系统来上传到网站。

一个简单的软件项目的 ``Makefile`` 看起来大概像这样： ::

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

以上的 ``Makefile`` 可能并不是最优的，但是它提供了一种只用输入 ``make`` 就可以编译并安装已链接的二进制文件的方法。每个 *目标文件（ target ）* 都包含一系列之后的命令所需要的 *依赖项（ dependencies ）*\。这意味着定义的顺序是任意的，调用 ``make`` 的时候相应的命令会按合适的顺序被运行。

例子中很多是冗余或重复的，比方说如果一个目标文件是直接从一个同名 C 文件编译而来，我们并不需要包含这样的目标文件， ``make`` 会帮我们解决这些。类似的，把经常调用的命令用变量来代替是明智的，这样的话我们在替换编译器或变更标签的时候也不用一个一个地修改了。一个更加简明扼要的版本如下所示： ::
    
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


``make`` 更为广泛的使用
------------------------

然而，为了自动化，将思维扩散到只对代码的编译和连接之外是很具启发性的。举个简单的网站项目的例子，该项目涉及到要部署 PHP 代码到产品服务器。这项任务通常不会被联系到 ``make`` 的使用上，但是其机制是一样的：代码编译完成可以执行时，我们还有一些目的需要达到。

PHP 文件当然是不需要编译的了，但是网站资源文件经常需要。对网站开发人员来说很熟悉的例子就是为了部署，把图片从源矢量图导出成已缩放和优化过的光栅图片。你管理着你的源文件，到了部署的时候你要生成网页友好的版本。

我们假设这个项目里，网站用到一组4个图标，这些图标都是64＊64像素的。我们有 SVG 矢量格式的源文件，在版本控制系统中静静地等待着。现在我们需要为网站生成小一些、可以部署的位图。我们这就可以定义一个目标叫 ``icons``\，设置好依赖项，然后输入要运行的命令。 ``Makefile`` 的句法中才是 Unix 命令行工具真正开始闪闪发光的地方： ::
    
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

做完以上的这些，输入命令 ``make icons`` 就会在 Bash 循环里把每个源图标文件遍历一遍，将它们用 ImageMagick 的 ``convert`` 从 SVG 转化成 PNG，再用 ``pngcrush`` 优化，于是便生成了可以上传的图片文件。

生成多种格式的帮助文件也可以用类似的方法，比如从 Markdown 源文件生成 HTML 文件： ::
    
    docs: README.html credits.html

    README.html: README.md
        markdown README.md > README.html

    credits.html: credits.md
        markdown credits.md > credits.html

最后也可以用 ``git push web`` 部署网站，但只能是在图标文件已栅格化和文档已转化 _之后_ ： ::
    
    deploy: icons docs
        git push web

为了更加简短而高效地把一种后缀的文件转化成另一种，你可以用 ``.SUFFIXES`` 指令，通过定义一些特殊符号来做到这些。转化图片的代码可能会变成下面这样，在这个例子里， ``$<`` 指源文件， ``$*`` 指没有后缀的文件名， ``$@`` 指目标文件： ::
    
    icons: create.png read.png update.png delete.png

    .SUFFIXES: .svg .png

    .svg.png:
        convert $< $*.raw.png && \
        pngcrush $*.raw.png $@


创建 ``Makefile`` 的工具
------------------------

GNU Autotools toolchain 里有多样工具用来为大型软件项目从更高层构造 ``configure``
脚本和 ``make`` 文件，具体来说就是 `autoconf <http://en.wikipedia.org/wiki/Autoconf>`_ 和 `automake <http://en.wikipedia.org/wiki/Automake>`_\。
使用这些工具可以在很大的源上生成 ``configure`` 脚本和 ``make`` 文件，它们免除了你必须要手动编写大量 makefile，并且一些自动步骤的运行可
以保证源文件在不同的操作系统上保持一致性而且可编译。

这个过程涵盖的内容之复杂足够再写一系列文章加以阐述，这已经超出了本篇的范畴。

*在此特别感谢用户 samwyse 在评论中关于* ``.SUFFIXES`` *的建议*
