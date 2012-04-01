编辑器
======

文本编辑器对所有程序员来说都是核心工具，这也是为什么它会引发虽不当真却也狂热的争议。Unix，很显而易见的，是与两大最经久不衰的受宠编辑器最有渊源的操作系统。这两个编辑器是 Emacs 和 Vi，它们的现代版本便是 GNU Emacs 和 Vim，两个编辑哲学迥异的编辑器在效能上却旗鼓相当。

作为 Vim 的异教徒，我来聊聊 Vim 对于编程不可或缺的功能，具体来说就是从 Vim 里调用 Linux shell 工具来完善编辑器的内建功能性。这里谈到的一些原理对 Emacs 也适用，但是对那些欠强大的编辑器来说并无参考价值，比如 Nano。

这篇帖子只能是纵览全局，毕竟 Vim 的编程工具组非常多，但即使是很泛泛地谈，它也会是一篇相当长的帖子。我会把重点放在要点和一些我认为很有帮助的东西上，并且有可能的话提供一些文章的链接以便对此话题有更加全面的了解。同时也别忘了 Vim 的 ``:help``\，很多新人都没有想到这份文档是如此高质量又好用。

文件类型侦测
------------

Vim 有内建的设置用于调整其运作方式，具体来说比如它基于被加载的文件类型进行句法高亮，这么做一直非常有效。另外，文件类型的侦测还可以让你根据某种语言常规的书写风格为此语言设置特定的缩进风格。这应该是第一批你需要加进 ``.vimrc`` 里的一部分： ::
    
    if has("autocmd")
        filetype on
        filetype indent on
        filetype plugin on
    endif

句法高亮
--------

即便你只是在16位色的命令行下工作，如果你还没有这么做，那赶紧在你的 ``.vimrc`` 里加入这句话吧： ::
    
    syntax on

默认的16位色的命令行配色不怎么好看也是不得已，但是他们已经起到作用了，绝大多数语言的句法定义文件是可轻易获取的，而且它们的效果都不错。这儿有 `一大堆配色方案 <http://code.google.com/p/vimcolorschemetest/>`_\，而且调整甚至自己写都不困难。当然用 `256色的命令行 <http://vim.wikia.com/wiki/256_colors_in_vim>`_ 或 gVim 会提供更多选项。好的句法高亮文件还会在有明显句法错误的时候用醒目的红色背景标示。

行号
----

可能你在传统的 IDE 里经常使用，打开行号： ::
    
    set number

如果你现在在用高于 Vim 7.3 的版本，你可能也会想试试把绝对行号换成相对行号： ::
    
    set relativenumber

标签文件
--------

Vim 对 ``ctags`` 实用工具的输出 `支持得很好 <http://amix.dk/blog/post/19329>`_\。它能让你快速地在整个项目中搜索某个特定的标示符；或者不管在不在同一个文件，直接从某变量被使用的地方跳转到该变量被申明的位置。对还有多个文件的 C 语言项目，这可以节省大把本来会被浪费掉的时间，而且很有可能 Vim 是目前有类似功能的主流 IDE 中最好的。

在你的项目根目录下（很多流行的编程语言都可以）运行 ``:!ctags -R`` 来生成 ``tags`` 文件，此文件里是整个项目里所有的申明和标示符的位置。一旦 ``ctags`` 文件被生成，你就可以像下面这样来搜索某些标签了： ::
    
    :tag someClass

用 ``:tn`` 和 ``:tp``\，你就可以遍历搜索结果了。自带的标签功能已经可以满足你大部分的需求了，但是像标签列表窗口这样的功能，你可以是是安装很受欢迎的 `Taglist 插件 <http://vim-taglist.sourceforge.net/>`_\。Tim Pope 的 `Unimpaired 插件 <https://github.com/tpope/vim-unimpaired>`_ 也有一些有用的相关映射。

调用外部程序
------------

有两中主要方法可以在 Vim 里调用外部程序：

* ``:!<command>``\——在从 Vim 内容跑某命令时很有用，尤其是在你想把运行结果输出到 Vim buffer 的情况下。
* ``:shell``\——以 Vim 子进程的方式弹开一个命令行。适合交互式命令。

第三种方法我不想在这里深入讨论，就是用像 `Conque <http://code.google.com/p/conque/>`_ 在 Vim buffer 里模拟命令行。我自己试了下发现几乎不能用，我敢断言这是个糟糕的设计。一下摘自 ``:help design-not:``\：
    
    Vim 不是 Shell 也不是操作系统。你不能在 Vim 里跑 Shell 也不能用它来控制调试器。相反的：把 Vim 当作 Shell 或 IDE 的一部分。

Lint 程序和句法检查器
`````````````````````

调用外部程序（如 ``perl -c``\, ``gcc``\）来检测句法是在 Vim 里用 ``:!`` 很好的例子。如果你在编写 Perl 文件，就可以这样跑： ::
    
    :!perl -c %

    /home/tom/project/test.pl syntax OK

    Press Enter or type command to continue

上面的百分号 ``%`` 是一种表示当前显示内容的简略方式。如果运行的命令有回显，那回显就会显示在命令下方。如果你需要经常调用句法检查器，你也可以在 ``.vimrc`` 里把它设置成命令，甚至再设置一个组合键。在这个例子里，我们可以定义一个 ``:PerlLint`` 命令，并且可以在正常模式下用 ``\l`` 触发（译者按：1. 作者指的正常模式是相对于 Vim 里的输入模式和选择模式； 2. 作者 Vim 的 ``<leader>``
键是用的 ``\``\，这个设置因人而异）： ::
    
    command PerlLint !perl -c %
    nnoremap <leader>l :PerlLint<CR>

对不少语言而言其实都有一个更好的办法来实现以上的事情，而且将要谈的这个方法能让我们利用起 Vim 自带的 quickfix 窗口。首先我们要对特定的文件类型设置一个合适的 ``makeprg``\,在这个例子里面，包含可以被 Vim 用以输出到 quicklist 的模块并定义两种输出格式： ::
    
    :set makeprg=perl\ -c\ -MVi::QuickFix\ %
    :set errorformat+=%m\ at\ %f\ line\ %l\.
    :set errorformat+=%m\ at\ %f\ line\ %l

你有可能先得从 CPAN 或 Debian 包管理器安装 ``libvi-quickfix-perl`` 模块。安装完成，保存文档，然后输入 ``:make`` 来检查句法。如果找到错误了，你可以用 ``:copen`` 打开 quicklist 窗口检查那些错误。用 ``:cn`` 和 ``:cp`` 上下移动。

.. figure:: origin/vim-quickfix.png
   :alt: vim-quickfix

   用 Vim quickfix 检测一个 Perl 文件

这种方法同样也适用于 `gcc <http://tldp.org/HOWTO/C-editing-with-VIM-HOWTO/quickfix.html>`_ 的输出，以及几乎其他任何一种句法检测的输出，输出包括文件名、行号、错误信息。它甚至可以支持 `像 PHP 一样专注于网页的语言 <http://stackoverflow.com/questions/7193547/debugging-php-with-vim-using-quickfix>`_\，还有像 `JSLint for JavaScript <https://github.com/hallettj/jslint.vim>`_
这样的工具。另外还有一个非常棒的插件叫 `Syntastic <http://www.vim.org/scripts/script.php?script_id=2736>`_ 也有类似的功效。

从其他命令读取输出
``````````````````
你可以用 ``:r!`` 把呼叫命令的回显直接贴到当前工作的文档里。例如，把当前目录的文档列表放进当前编辑文件就可以输入： ::
    
    :r!ls

这种读取方式当然不光可以用在命令回显；你可以用 ``:r`` 轻松读进其他文件的内容，比如你的公钥或是你自定义的样版文件： ::
    
    :r ~/.ssh/id_rsa.pub
    :r ~/dev/perl/boilerplate/copyright.pl

用其他命令过滤输出
``````````````````
加以延伸，其实你可以把 Vim buffer 里的文字放进外部命令过滤，或者是用选择模式选择一个文本区块，然后用命令的输出覆盖。因为 Vim 的块状选择模式很适合用在柱形数据，所以它很适合配合 ``column``\、 ``cut``\、 ``sort``\、 ``awk`` 等类似的工具使用。

例如，你可以将整个文件按第二列逆序排列： ::
    
    :%!sort -k2 -r

你可以在所选则文字中找到符合 ``/vim/`` 样式并只显示其中的第三列： ::
    
    :'<,'>!awk '/vim/ {print $3}'

你也可以把前10行的关键词用漂亮的行列格式排好： ::
    
    :1,10!column -t

真的，所有类型的文字过滤器或命令都可以像上面的例子一样用在 Vim 里，一个简单的互操作性就可以让编辑器的能力无限延伸。这很有效地将 Vim buffer 变成了字符流，而字符流正是这些经典工具之间用以交流的语言。

自带的其他选择
``````````````
值得注意的是，像排序和查找之类常见的操作，Vim 有其自带的方法 ``:sort`` 和 ``:grep``\，这些或许在你在 Windows 下用 Vim 时遇到困难时很有帮助，但是这些自带方法并不具备适应 shell 命令的能力。

对比文件
--------

