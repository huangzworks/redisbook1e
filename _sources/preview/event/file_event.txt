文件事件
---------------

Redis 基于 `Reactor 模式 <http://en.wikipedia.org/wiki/Reactor_pattern>`_\ 开发了自己的网络事件处理器：
这个处理器被称为文件事件处理器（file event handler）：

- 文件事件处理器使用 `I/O 多路复用（multiplexing） <http://en.wikipedia.org/wiki/Multiplexing>`_\ 程序来同时监听多个套接字，
  并根据套接字目前执行的任务来为套接字关联不同的事件处理器。

- 当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关闭（close）等操作时，
  与操作相对应的文件事件就会产生，
  这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

虽然文件事件处理器以单线程方式运行，
但通过使用 I/O 多路复用程序来监听多个套接字，
文件事件处理器既实现了高性能的网络通信模型，
又可以很好地与 Redis 服务器中其他同样以单线程方式运行的模块进行对接，
这保持了 Redis 内部单线程设计的简单性。


文件事件处理器的构成
^^^^^^^^^^^^^^^^^^^^^^^^^^

图 IMAGE_CONSTRUCT_OF_FILE_EVENT_HANDLER 展示了文件事件处理器的四个组成部分，
它们分别是套接字、
I/O 多路复用程序、
文件事件分派器（dispatcher）、
以及事件处理器。

.. graphviz::

    digraph {

        label = "\n 图 IMAGE_CONSTRUCT_OF_FILE_EVENT_HANDLER    文件事件处理器的四个组成部分";

        rankdir = LR;

        node [shape = box];

        subgraph cluster_sockets {

            style = dashed

            label = "套接字";

            c1 [label = "s1", shape = circle];
            c2 [label = "s2", shape = circle];
            other_client [label = "...", width = 1.1, shape = plaintext];
            c3 [label = "sN", shape = circle];

        }

        io_multiplexing [label = "I\n/\nO\n多\n路\n复\n用\n程\n序"];

        file_event_processor [label = "文\n件\n事\n件\n分\n派\n器"];

        subgraph cluster_handlers {

            style = dashed

            label = "事件处理器";

            write_handler [label = "命令请求处理器"];

            read_handler [label = "命令回复处理器"];

            connect_handler [label = "连接应答处理器"];

            other_handlers [label = "...", width = 1.6];

        }

        c1 -> io_multiplexing;
        c2 -> io_multiplexing;
        other_client -> io_multiplexing [style = invis];
        c3 -> io_multiplexing;

        io_multiplexing -> file_event_processor;

        file_event_processor -> write_handler;
        file_event_processor -> read_handler;
        file_event_processor -> connect_handler;
        file_event_processor -> other_handlers;
    }

文件事件是对套接字操作的抽象，
每当一个套接字准备好执行连接应答（accept）、写入、读取、关闭等操作时，
就会产生一个文件事件。
因为一个服务器通常会连接多个套接字，
所以多个文件事件有可能会并发地出现。

I/O 多路复用程序负责监听多个套接字，
并向文件事件分派器传送那些产生了事件的套接字。

尽管多个文件事件可能会并发地出现，
但 I/O 多路复用程序总是会将所有产生事件的套接字都入队到一个队列里面，
然后通过这个队列，
以有序（sequentially）、同步（synchronously）、每次一个套接字的方式向文件事件分派器传送套接字：
当上一个套接字产生的事件被处理完毕之后（该套接字为事件所关联的事件处理器执行完毕），
I/O 多路复用程序才会继续向文件事件分派器传送下一个套接字，
如图 IMAGE_DISPATCH_EVENT_VIA_QUEUE 。

.. graphviz::

    digraph {

        rankdir = LR;

        node [shape = record];

        label = "\n图 IMAGE_DISPATCH_EVENT_VIA_QUEUE    I/O 多路复用程序通过队列向文件事件分派器传送套接字";

        //

        subgraph cluster_io_multiplexing {

            //style = dashed
    
            label = "队列";

            queue [label = " { 套接字 sN | 套接字 sN-1 | ... | 套接字 s3 | 套接字 s2 } "];

        }

        file_event_processor [label = "文\n件\n事\n件\n分\n派\n器"];

        //

        queue -> file_event_processor [label = "传送\n 套接字 s1", style = dashed];
    }

文件事件分派器接收 I/O 多路复用程序传来的套接字，
并根据套接字产生的事件的类型，
调用相应的事件处理器。

服务器会为执行不同任务的套接字关联不同的事件处理器，
这些处理器是一个个函数，
它们定义了某个事件发生时，
服务器应该执行的动作。


I/O 多路复用程序的实现
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Redis 的 I/O 多路复用程序的所有功能都是通过包装常见的 
``select`` 、 ``epoll`` 、 ``evport`` 和 ``kqueue``
这些 I/O 多路复用函数库来实现的，
每个 I/O 多路复用函数库在 Redis 源码中都对应一个单独的文件，
比如 ``ae_select.c`` 、 ``ae_epoll.c`` 、 ``ae_kqueue.c`` ，
诸如此类。

因为 Redis 为每个 I/O 多路复用函数库都实现了相同的 API ，
所以 I/O 多路复用程序的底层实现是可以互换的，
如图 IMAGE_MULTI_LIB 所示。

.. graphviz::

    digraph {

        label = "图 IMAGE_MULTI_LIB    Redis 的 I/O 多路复用程序有多个 I/O 多路复用库实现可选";

        node [shape = box];

        io_multiplexing [label = "I/O 多路复用程序"];

        subgraph cluster_imp {

            style = dashed

            label = "底层实现";
            labelloc = "b";

            kqueue [label = "kqueue"];
            evport [label = "evport"];
            epoll [label = "epoll"];
            select [label = "select"];
        }

        //

        edge [dir = back];

        io_multiplexing -> select;
        io_multiplexing -> epoll;
        io_multiplexing -> evport;
        io_multiplexing -> kqueue;

    }

Redis 在 I/O 多路复用程序的实现源码中用 ``#include`` 宏定义了相应的规则，
程序会在编译时自动选择系统中性能最高的 I/O 多路复用函数库来作为 Redis 的 I/O 多路复用程序的底层实现：

::

    /* Include the best multiplexing layer supported by this system.
     * The following should be ordered by performances, descending. */
    #ifdef HAVE_EVPORT
    #include "ae_evport.c"
    #else
        #ifdef HAVE_EPOLL
        #include "ae_epoll.c"
        #else
            #ifdef HAVE_KQUEUE
            #include "ae_kqueue.c"
            #else
            #include "ae_select.c"
            #endif
        #endif
    #endif


事件的类型
^^^^^^^^^^^^^

I/O 多路复用程序可以监听多个套接字的 ``ae.h/AE_READABLE`` 事件和 ``ae.h/AE_WRITABLE`` 事件，
这两类事件和套接字操作之间的对应关系如下：

- 当套接字变得可读时（客户端对套接字执行 ``write`` 操作，或者执行 ``close`` 操作），
  或者有新的可应答（acceptable）套接字出现时（客户端对服务器的监听套接字执行 ``connect`` 操作），
  套接字产生 ``AE_READABLE`` 事件。

- 当套接字变得可写时（客户端对套接字执行 ``read`` 操作），
  套接字产生 ``AE_WRITABLE`` 事件。

I/O 多路复用程序允许服务器同时监听套接字的 ``AE_READABLE`` 事件和 ``AE_WRITABLE`` 事件，
如果一个套接字同时产生了这两种事件，
那么文件事件分派器会优先处理 ``AE_READABLE`` 事件，
等到 ``AE_READABLE`` 事件处理完之后，
才处理 ``AE_WRITABLE`` 事件。

这也就是说，
如果一个套接字又可读又可写的话，
那么服务器将先读套接字，
后写套接字。


API
^^^^^^^^^^

``ae.c/aeCreateFileEvent`` 函数接受一个套接字描述符、
一个事件类型、
以及一个事件处理器作为参数，
将给定套接字的给定事件加入到 I/O 多路复用程序的监听范围之内，
并对事件和事件处理器进行关联。

``ae.c/aeDeleteFileEvent`` 函数接受一个套接字描述符和一个监听事件类型作为参数，
让 I/O 多路复用程序取消对给定套接字的给定事件的监听，
并取消事件和事件处理器之间的关联。

``ae.c/aeGetFileEvents`` 函数接受一个套接字描述符，
返回该套接字正在被监听的事件类型：

- 如果套接字没有任何事件被监听，
  那么函数返回 ``AE_NONE`` 。

- 如果套接字的读事件正在被监听，
  那么函数返回 ``AE_READABLE`` 。

- 如果套接字的写事件正在被监听，
  那么函数返回 ``AE_WRITABLE`` 。

- 如果套接字的读事件和写事件正在被监听，
  那么函数返回 ``AE_READABLE | AE_WRITABLE`` 。

``ae.c/aeWait`` 函数接受一个套接字描述符、一个事件类型和一个毫秒数为参数，
在给定的时间内阻塞并等待套接字的给定类型事件产生，
当事件成功产生，
或者等待超时之后，
函数返回。

``ae.c/aeApiPoll`` 函数接受一个 ``sys/time.h/struct timeval`` 结构为参数，
并在指定的时间內，
阻塞并等待所有被 ``aeCreateFileEvent`` 函数设置为监听状态的套接字产生文件事件，
当有至少一个事件产生，
或者等待超时后，
函数返回。

``ae.c/aeProcessEvents`` 函数是文件事件分派器，
它先调用 ``aeApiPoll`` 函数来等待事件产生，
然后遍历所有已产生的事件，
并调用相应的事件处理器来处理这些事件。

``ae.c/aeGetApiName`` 函数返回 I/O 多路复用程序底层所使用的 I/O 多路复用函数库的名称：
返回 ``"epoll"`` 表示底层为 ``epoll`` 函数库，
返回 ``"select"`` 表示底层为 ``select`` 函数库，
诸如此类。


文件事件的处理器
^^^^^^^^^^^^^^^^^^^

Redis 为文件事件编写了多个处理器，
这些事件处理器分别用于实现不同的网络通讯需求，
比如说：

- 为了对连接服务器的各个客户端进行应答，
  服务器要为监听套接字关联连接应答处理器。

- 为了接收客户端传来的命令请求，
  服务器要为客户端套接字关联命令请求处理器。

- 为了向客户端返回命令的执行结果，
  服务器要为客户端套接字关联命令回复处理器。

- 当主服务器和从服务器进行复制操作时，
  主从服务器都需要关联特别为复制功能编写的复制处理器。

- 等等。

在这些事件处理器里面，
服务器最常用的要数与客户端进行通信的连接应答处理器、
命令请求处理器和命令回复处理器。


连接应答处理器
""""""""""""""""""""

``networking.c/acceptTcpHandler`` 函数是 Redis 的连接应答处理器，
这个处理器用于对连接服务器监听套接字的客户端进行应答，
具体实现为 ``sys/socket.h/accept`` 函数的包装。

当 Redis 服务器进行初始化的时候，
程序会将这个连接应答处理器和服务器监听套接字的 ``AE_READABLE`` 事件关联起来，
当有客户端用 ``sys/socket.h/connect`` 函数连接服务器监听套接字的时候，
套接字就会产生 ``AE_READABLE`` 事件，
引发连接应答处理器执行，
并执行相应的套接字应答操作，
如图 IMAGE_SERVER_ACCEPT_CONNECT 所示。

.. graphviz::
    
    digraph {

        label = "\n图 IMAGE_SERVER_ACCEPT_CONNECT    服务器对客户端的连接请求进行应答";

        rankdir = LR;

        client [label = "客户端", shape = circle];

        server [label = "服务器\n\n\n服务器监听套接字产生\nAE_READABLE 事件\n执行连接应答处理器", shape = box, height = 2];

        client -> server [label = "连接监听套接字"];

    }


命令请求处理器
"""""""""""""""""""

``networking.c/readQueryFromClient`` 函数是 Redis 的命令请求处理器，
这个处理器负责从套接字中读入客户端发送的命令请求内容，
具体实现为 ``unistd.h/read`` 函数的包装。

当一个客户端通过连接应答处理器成功连接到服务器之后，
服务器会将客户端套接字的 ``AE_READABLE`` 事件和命令请求处理器关联起来，
当客户端向服务器发送命令请求的时候，
套接字就会产生 ``AE_READABLE`` 事件，
引发命令请求处理器执行，
并执行相应的套接字读入操作，
如图 IMAGE_SERVER_RECIVE_COMMAND_REQUEST 所示。

.. graphviz::
    
    digraph {

        label = "\n图 IMAGE_SERVER_RECIVE_COMMAND_REQUEST    服务器接收客户端发来的命令请求";

        rankdir = LR;

        client [label = "客户端", shape = circle];

        server [label = "服务器\n\n\n客户端套接字产生\nAE_READABLE 事件\n执行命令请求处理器", shape = box, height = 2];

        client -> server [label = "发送命令请求"];

    }

在客户端连接服务器的整个过程中，
服务器都会一直为客户端套接字的 ``AE_READABLE`` 事件关联命令请求处理器。


命令回复处理器
"""""""""""""""""""""""""

``networking.c/sendReplyToClient`` 函数是 Redis 的命令回复处理器，
这个处理器负责将服务器执行命令后得到的命令回复通过套接字返回给客户端，
具体实现为 ``unistd.h/write`` 函数的包装。

当服务器有命令回复需要传送给客户端的时候，
服务器会将客户端套接字的 ``AE_WRITABLE`` 事件和命令回复处理器关联起来，
当客户端准备好接收服务器传回的命令回复时，
就会产生 ``AE_WRITABLE`` 事件，
引发命令回复处理器执行，
并执行相应的套接字写入操作，
如图 IMAGE_SERVER_SEND_REPLY 所示。

.. graphviz::
    
    digraph {

        label = "\n图 IMAGE_SERVER_SEND_REPLY    服务器向客户端发送命令回复";

        rankdir = LR;

        client [label = "客户端", shape = circle];

        server [label = "服务器\n\n\n客户端套接字产生\nAE_WRITABLE 事件\n执行命令回复处理器", shape = box, height = 2];

        client -> server [dir = back, label = "发送命令回复"];

    }

当命令回复发送完毕之后，
服务器就会解除命令回复处理器与客户端套接字的 ``AE_WRITABLE`` 事件之间的关联。


一次完整的客户端与服务器连接事件示例
""""""""""""""""""""""""""""""""""""""""""""

让我们来追踪一次 Redis 客户端与服务器进行连接并发送命令的整个过程，
看看在过程中会产生什么事件，
而这些事件又是如何被处理的。

假设一个 Redis 服务器正在运作，
那么这个服务器的监听套接字的 ``AE_READABLE`` 事件应该正处于监听状态之下，
而该事件所对应的处理器为连接应答处理器。

如果这时有一个 Redis 客户端向服务器发起连接，
那么监听套接字将产生 ``AE_READABLE`` 事件，
触发连接应答处理器执行：
处理器会对客户端的连接请求进行应答，
然后创建客户端套接字，
以及客户端状态，
并将客户端套接字的 ``AE_READABLE`` 事件与命令请求处理器进行关联，
使得客户端可以向主服务器发送命令请求。

之后，
假设客户端向主服务器发送一个命令请求，
那么客户端套接字将产生 ``AE_READABLE`` 事件，
引发命令请求处理器执行，
处理器读取客户端的命令内容，
然后传给相关程序去执行。

执行命令将产生相应的命令回复，
为了将这些命令回复传送回客户端，
服务器会将客户端套接字的 ``AE_WRITABLE`` 事件与命令回复处理器进行关联：
当客户端尝试读取命令回复的时候，
客户端套接字将产生 ``AE_WRITABLE`` 事件，
触发命令回复处理器执行，
当命令回复处理器将命令回复全部写入到套接字之后，
服务器就会解除客户端套接字的 ``AE_WRITABLE`` 事件与命令回复处理器之间的关联。

图 IMAGE_COMMAND_PROGRESS 总结了上面描述的整个通讯过程，
以及通讯时用到的事件处理器。

.. graphviz::

    digraph {

        label = "\n图 IMAGE_COMMAND_PROGRESS    客户端和服务器的通讯过程";

        splines = ortho;

        rankdir = LR;

        node [shape = box, height = 3.0];

        client [label = "客\n户\n端"];

        server [label = "服\n务\n器"];

        client -> server [label = "客户端向服务器发送连接请求\n服务器执行连接应答处理器"];

        client -> server [label = "\n\n客户端向服务器发送命令请求\n服务器执行命令请求处理器"];

        server -> client [label = "\n\n服务器向客户端发送命令回复\n服务器执行命令回复处理器"];

    }
