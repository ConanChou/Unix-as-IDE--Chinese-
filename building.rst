编译
====

编译项目可能是个复杂的重复劳动，而一个好的集成开发环境可以提供一种精简甚至自动化
编译的方法。 Unix 及其衍生系统用 ``Makefile`` 
来完成这个工序，这是一种有着标准格式，用来把源代码和目标文件编译成可执行文件的“
经方”，而且它只会去重新编译那些有变动的文件，这样的话效率也更高。

关于 ``make`` 还有个非常有趣的东西值得注意，就是它通常用在自动化编译的时候而且它
有不少快捷方式去达到一样的效果，而事实上凡是把一堆文件生成成另一堆文件的情况都可
以利用它。一种可能的用法，比如说，在做网站部署的时候将原图片优化成网页友好的图片
；另一种用法可以是从代码生成静态 HTML 页面，而不是运行时生成页面（译者按：利用 
github 建博客就是这个原理）。在此基础上对软件“编译”更加灵活理解的工具，如 `Ruby 
的 rake <http://rake.rubyforge.org/>`_ 
就被大家广泛使用，它自动化了像产生和安装代码和文件等东西的一般任务。

剖析 ``Makefile``
-----------------

``Makefile`` 的一般格式包含一列变量、一列目标产物，以及要用的源和／或目标。目标产物也
不一定是连接了的二进制文件；它们也可以是由对生成文件的操作动作构成，比如 ``install`` 用来把生成的编译文件部署到系统里，用 ``clean`` 从源代码树里
清除已编译的文件。

正是这种目标产物的灵活性才使得 ``make`` 可以自动化任何相关产品软件的组建任务：
并不仅仅是特定的语法分析、预处理、编译和连接步骤，还包括跑测试（ ``make test`` 
），或把文档的源文件编译成一种或多种适当的格式，或将代码自动部署到产品系统里，
比如通过 ``git push`` 或类似的内容跟踪系统来上传服务器的方法。

一个简单的软件项目的 ``Makefile`` 大概看起来像这样： ::

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

以上的 ``Makefile`` 可能并不是最优的，但是它提供了一种只用输入 ``make`` 就可以
编译并安装已连接的二进制文件的方法。每个 *目标文件（ target ）* 都包含一连
串下一行命令所需要的 *依赖关系（ dependencies ）*\。这意味着定义可以任意顺序，
呼叫 ``make`` 的时候相应的命令会按合适的顺序被运行。

上面代码中很多是冗余的，比方说如果一个目标文件是直接从一个同名 C 文件编译而来，
我们并不需要包含这样的目标文件， ``make`` 会帮我们解决这些。类似的，把经常呼叫的
命令用变量来代替是明智的，这样的话我们在替换编译器或变更标签的时候也不用一个一个
地修改了。一个更加简明扼要的版本如下所示： ::
    
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

然而，为了自动化，将思维扩散到只对代码的编译和连接之外是很具启发性的。举个简单
的网站项目的例子，该项目里涉及到要部署 PHP 代码到产品服务器。这项任务通常不会
被联系到 ``make`` 的使用上，但是其机理是一样的：代码已到位随时待命，而我们有些
目标文件需要生成。

PHP 文件当然是不需要编译的了，但是网站相关的文件经常需要。对网站开发人员来
说很熟悉的例子就是为了部署，把图片从源矢量图导出成已缩放和优化过的光栅图片。
你平时就保留和版本管理你的源文件，到了部署的时候就生成网页友好的版本。

我们假设这儿有个项目，网站里用到一组4个图标，这些图标都是64＊64像素的。我们现
在用 SVG 矢量格式的源文件，被很安全地存储在版本控制系统里，现在我们需要为网站
部署做准备，生成小一些的位图。我们这就可以定义一个目标产物叫 ``icons``\，设置好
依赖关系，然后输入要运行的命令。这里用的 ``Makefile`` 句法才是 Unix 命令行工
具真正开始闪闪发光的地方： ::
    
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

做完以上的这些，输入命令 ``make icons`` 就会在 Bash 循环里把每个源图标文件
遍历一遍，将它们用 ImageMagick 的 ``convert`` 从 SVG 转化成 PNG，再用 
``pngcrush`` 优化，进而生成可以上传的图片文件。

生成多种格式的帮助文件也可以用相仿的方法，比如从 Markdown 源文件生成 HTML 文件： ::
    
    docs: README.html credits.html

    README.html: README.md
        markdown README.md > README.html

    credits.html: credits.md
        markdown credits.md > credits.html

或许最后再用 ``git push web`` 部署网站，但只能是在图标文件已栅格化和文档已转化之后： ::
    
    deploy: icons docs
        git push web

为了更加凝练地把文件的一种后缀转化成另一种，你可以用 ``.SUFFIXES`` 编译指令
来定义这些，用一些特殊符号。转化图片的代码可能会变成下面这样，在这个例子里， ``$<`` 
指源文件， ``$*`` 指没有后缀的文件名， ``$@`` 指目标文件： ::
    
    icons: create.png read.png update.png delete.png

    .SUFFIXES: .svg .png

    .svg.png:
        convert $< $*.raw.png && \
        pngcrush $*.raw.png $@


创建 ``Makefile`` 的工具
------------------------

GNU Autotools toolchain 里有多样工具用来为大型软件项目从更高层构造 ``configure``
脚本和 ``make`` 文件，具体来说就是 `autoconf <http://en.wikipedia.org/wiki/Autoconf>`_ 和 `automake <http://en.wikipedia.org/wiki/Automake>`_\。
使用这些工具可以在很大的源文件基础上生成 ``configure`` 脚本和 ``make`` 文件，它们免除了你必须要手动编写大量 makefile，并且一些自动步骤的运行可
以保证源文件在不同的操作系统上保持一致性而且可编译。
