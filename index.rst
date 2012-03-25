.. Unix 即集成开发环境 documentation master file, created by
   sphinx-quickstart on Tue Feb 28 02:37:06 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Unix 即集成开发环境
=========================

动机
----

前阵子，我在 `Hacker News <http://news.ycombinator.com/>`_ 上看到一个Tom Ryder的系列文章， `谈关于Unix和IDE（集成开发环境）的 <http://blog.sanctum.geek.nz/series/unix-as-ide/>`_\。其实类似的文章在HN上经常看到，只是由于我是开源和Vim狂热者才会每次都点开看看——每次都能学到新的东西。

那一阵子也正好好几次跟同学和朋友说到有关的话题。他们都是 IDE 控，这不是什么不好的事情，我自己在一些时候也会选择 IDE。但是他们普遍对使用 Unix 和类似 Vim 的编辑工具没有正确的认识，以至于我的观点得不到应有程度的共鸣或共识。而我恰好看到了这个系列文章，觉得这是一个非常不错的入门系列。而且我在很多观点上都认同作者，所以本打算写一篇博文谈谈来着，现在索性来翻译这个系列文章好了。（之所以要翻译而不是直接转载是因为我发现对于 Unix 以及相关开发工具的使用的知识缺乏在中国存在度很高，语言可以是原因之一，所以这个系列不是为我的朋友和同学而翻译，更大程度上是为了普及知识。）

如果您对此话题很感兴趣，不妨也读一读 `The unix programming environment <http://www.iu.hio.no/~mark/unix/unix_toc.html>`_\。如果你现在还在大学里，也不妨上一些有关的课程，一定会对你的职业生涯有很大的帮助。

**最后声明** ，我并不想鼓吹使用某种开发工具或使用某种工作流程，和此系列作者一样，只普及知识。如果这个系列的文章颠覆了你的世界观，你因此变成了 Unix 粉，我们概不负责。

目录
----

.. toctree::
   :maxdepth: 2
   
   前言 <introduction>
   文件 <files>
   编辑器 <editing>
   编译 <compiling>
   生成 <building>
   错误排除 <debug>
   版本控制 <revisions>
   写在后面的话 <conclude>

项目说明
--------

* 使用的是 `sphinx <http://sphinx.pocoo.org/>`_ 文档生成器。
* 项目主页 `Unix as IDE (Chinese) <https://github.com/ConanChou/Unix-as-IDE--Chinese->`_\。

贡献者
------

本翻译项目的贡献者： `ConanChou <https://github.com/ConanChou>`_\,  `KarenMeu <https://github.com/karenmeu>`_\。

.. Indices and tables
   ==================
    
   * :ref:`genindex`
   * :ref:`modindex`
   * :ref:`search`
