事务的实现
---------------

一个事务从开始到结束通常会经历以下三个阶段：

1. 事务开始。

2. 命令入队。

3. 事务执行。

本节接下来的内容将对这三个阶段进行介绍，
说明一个事务从开始到结束的整个过程。


事务开始
^^^^^^^^^^

:ref:`MULTI` 命令的执行标志着事务的开始：

::

    redis> MULTI
    OK

:ref:`MULTI` 命令可以将执行该命令的客户端从非事务状态切换至事务状态，
这一切换是通过在客户端状态的 ``flags`` 属性中打开 ``REDIS_MULTI`` 标识来完成的，
:ref:`MULTI` 命令的实现可以用以下伪代码来表示：

.. code-block:: python

    def MULTI():

        # 打开事务标识
        client.flags |= REDIS_MULTI

        # 返回 OK 回复
        replyOK()

.. ||

命令入队
^^^^^^^^^^^^^

当一个客户端处于非事务状态时，
这个客户端发送的命令会立即被服务器执行：

::

    redis> SET "name" "Practical Common Lisp"
    OK

    redis> GET "name"
    "Practical Common Lisp"

    redis> SET "author" "Peter Seibel"
    OK

    redis> GET "author"
    "Peter Seibel"

与此不同的是，
当一个客户端切换到事务状态之后，
服务器会根据这个客户端发来的不同命令执行不同的操作：

- 如果客户端发送的命令为 :ref:`EXEC` 、 :ref:`DISCARD` 、 :ref:`WATCH` 、 :ref:`MULTI` 四个命令的其中一个，
  那么服务器立即执行这个命令。

- 与此相反，
  如果客户端发送的命令是 :ref:`EXEC` 、 :ref:`DISCARD` 、 :ref:`WATCH` 、 :ref:`MULTI` 四个命令以外的其他命令，
  那么服务器并不立即执行这个命令，
  而是将这个命令放入一个事务队列里面，
  然后向客户端返回 ``QUEUED`` 回复。

服务器判断命令是该入队还是该立即执行的过程可以用流程图 IMAGE_ENQUEUE_OR_EXEC 来描述。

.. graphviz:: 

    digraph enque_or_execute {

        label = "\n图 IMAGE_ENQUEUE_OR_EXEC    服务器判断命令是该入队还是该执行的过程";

        node [shape = box];

        //

        command_in [label = "服务器接到来自客户端的命令"];

        in_transaction_or_not [label = "这个客户端正处于事务状态？", shape = diamond];

        not_exec_and_discard [label = "这个命令是否\nEXEC 、 DISCARD 、\nWATCH 或 MULTI ？", shape = diamond];

        enqueu_command [label = "将命令放入事务队列"];

        return_enqueued [label = "向客户端返回 QUEUED"];

        exec_command [label = "执行这个命令"];

        return_command_result [label = "向客户端返回命令的执行结果"];

        // 

        command_in -> in_transaction_or_not;

        in_transaction_or_not -> not_exec_and_discard [label = "是"];

        not_exec_and_discard -> enqueu_command [label = "否"];

        not_exec_and_discard -> exec_command [label = "是"];

        in_transaction_or_not -> exec_command [label = "否"];

        exec_command -> return_command_result;

        enqueu_command -> return_enqueued;
    }

..
    以下是一些命令入队的例子：

    ::

        redis> SET "name" "Practical Common Lisp"
        QUEUED

        redis> GET "name"
        QUEUED

        redis> SET "author" "Peter Seibel"
        QUEUED

        redis> GET "author"
        QUEUED


事务队列
^^^^^^^^^^^^^^^

每个 Redis 客户端都有自己的事务状态，
这个事务状态保存在客户端状态的 ``mstate`` 属性里面：

::

    typedef struct redisClient {

        // ...

        // 事务状态
        multiState mstate;      /* MULTI/EXEC state */

        // ...

    } redisClient;

事务状态包含一个事务队列，
以及一个已入队命令的计数器
（也可以说是事务队列的长度）：

::

    typedef struct multiState {

        // 事务队列，FIFO 顺序
        multiCmd *commands;

        // 已入队命令计数
        int count;

    } multiState;

事务队列是一个 ``multiCmd`` 类型的数组，
数组中的每个 ``multiCmd`` 结构都保存了一个已入队命令的相关信息，
包括指向命令实现函数的指针，
命令的参数，
以及参数的数量：

::

    typedef struct multiCmd {

        // 参数
        robj **argv;

        // 参数数量
        int argc;

        // 命令指针
        struct redisCommand *cmd;

    } multiCmd;

事务队列以先进先出（FIFO）的方式保存入队的命令：
较先入队的命令会被放到数组的前面，
而较后入队的命令则会被放到数组的后面。

举个例子，
如果客户端执行以下命令：

::

    redis> MULTI
    OK

    redis> SET "name" "Practical Common Lisp"
    QUEUED

    redis> GET "name"
    QUEUED

    redis> SET "author" "Peter Seibel"
    QUEUED

    redis> GET "author"
    QUEUED

那么服务器将为客户端创建图 IMAGE_TRANSACTION_STATE 所示的事务状态：

- 最先入队的 :ref:`SET` 命令被放在了事务队列的索引 ``0`` 位置上。

- 第二入队的 :ref:`GET` 命令被放在了事务队列的索引 ``1`` 位置上。

- 第三入队的另一个 :ref:`SET` 命令被放在了事务队列的索引 ``2`` 位置上。

- 最后入队的另一个 :ref:`GET` 命令被放在了事务队列的索引 ``3`` 位置上。

.. graphviz::

    digraph {

        label = "\n 图 IMAGE_TRANSACTION_STATE    事务状态";

        rankdir = LR;

        node [shape = record];

        //redisClient [label = " <head> redisClient | ... | <mstate> mstate | ... "];

        multiState [label = " <head> multiState | <commands> commands | count \n 4 "];

        commands [label = " <head> multiCmd[4] | <0> [0] | <1> [1] | <2> [2] | <3> [3] "];

        multiCmd0 [label = " <head> multiCmd | <argv> argv | argc \n 3 | <cmd> cmd "];

        multiCmd1 [label = " <head> multiCmd | <argv> argv | argc \n 2 | <cmd> cmd "];

        multiCmd2 [label = " <head> multiCmd | <argv> argv | argc \n 3 | <cmd> cmd "];

        multiCmd3 [label = " <head> multiCmd | <argv> argv | argc \n 2 | <cmd> cmd "];

        //redisClient:mstate -> multiState:head;

        multiState:commands -> commands:head;

        commands:0 -> multiCmd0:head;
        commands:1 -> multiCmd1:head;
        commands:2 -> multiCmd2:head;
        commands:3 -> multiCmd3:head;

        argv0 [label = " robj*[3] | { StringObject \n \"SET\" | StringObject \n \"name\" | StringObject \n \"Practical Common Lisp\" } "];
        cmd0 [label = " setCommand ", shape = plaintext];

        multiCmd0:argv -> argv0;
        multiCmd0:cmd -> cmd0;

        argv1 [label = " robj*[2] | { StringObject \n \"GET\" | StringObject \n \"name\" } "];
        cmd1 [label = " getCommand ", shape = plaintext];

        multiCmd1:argv -> argv1;
        multiCmd1:cmd -> cmd1;

        argv2 [label = " robj*[3] | { StringObject \n \"SET\" | StringObject \n \"author\" | StringObject \n \"Peter Seibel\" } "];
        cmd2 [label = " setCommand ", shape = plaintext];

        multiCmd2:argv -> argv2;
        multiCmd2:cmd -> cmd2;

        argv3 [label = " robj*[2] | { StringObject \n \"GET\" | StringObject \n \"author\" } "];
        cmd3 [label = " getCommand ", shape = plaintext];

        multiCmd3:argv -> argv3;
        multiCmd3:cmd -> cmd3;

    }


执行事务
^^^^^^^^^^^^^

当一个处于事务状态的客户端向服务器发送 :ref:`EXEC` 命令时，
这个 :ref:`EXEC` 命令将立即被服务器执行：
服务器会遍历这个客户端的事务队列，
执行队列中保存的所有命令，
最后将执行命令所得的结果全部返回给客户端。

举个例子，
对于图 IMAGE_TRANSACTION_STATE 所示的事务队列来说，
服务器首先会执行命令：

::

    SET "name" "Practical Common Lisp"

接着执行命令：

::

    GET "name"

之后执行命令：

::

    SET "author" "Peter Seibel"

再之后执行命令：

::

    GET "author"

最后，
服务器会将执行这四个命令所得的回复返回给客户端：

::

    redis> EXEC
    1) OK
    2) "Practical Common Lisp"
    3) OK
    4) "Peter Seibel"


:ref:`EXEC` 命令的实现原理可以用以下伪代码来描述：

.. code-block:: python

    def EXEC():

        # 创建空白的回复队列
        reply_queue = []

        # 遍历事务队列中的每个项
        # 读取命令的参数，参数的个数，以及要执行的命令
        for argv, argc, cmd in client.mstate.commands:

            # 执行命令，并取得命令的返回值
            reply = execute_command(cmd, argv, argc)

            # 将返回值追加到回复队列末尾
            reply_queue.append(reply)

        # 移除 REDIS_MULTI 标识，让客户端回到非事务状态
        client.flags &= ~REDIS_MULTI

        # 清空客户端的事务状态，包括：
        # 1）清零入队命令计数器
        # 2）释放事务队列
        client.mstate.count = 0
        release_transaction_queue(client.mstate.commands)

        # 将事务的执行结果返回给客户端
        send_reply_to_client(client, reply_queue)


..
    在事务和非事务状态下执行命令
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    无论在事务状态下，
    还是在非事务状态下，
    Redis 命令都由同一个函数执行，
    所以它们共享很多服务器的一般设置，
    比如 AOF 的配置、RDB 的配置，以及内存限制，等等。
    不过事务中的命令和普通命令在执行上还是有一点区别的，其中最重要的两点是：
    1. 非事务状态下的命令以单个命令为单位执行，前一个命令和后一个命令的客户端不一定是同一个；
       而事务状态则是以一个事务为单位，执行事务队列中的所有命令：除非当前事务执行完毕，否则服务器不会中断事务，也不会执行其他客户端的其他命令。
    2. 在非事务状态下，执行命令所得的结果会立即被返回给客户端；
       而事务则是将所有命令的结果集合到回复队列，再作为 :ref:`EXEC` 命令的结果返回给客户端。

..
    示例：完整的事务执行过程
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    为了进一步熟悉事务的执行过程，
    让我们来看一个完整的事务执行例子。

    首先，
    假设客户端当前为非事务状态，
    那么当它向服务器发送 :ref:`MULTI` 命令之后，
    客户端切换为事务状态：

    ::

        redis> MULTI
        OK

    接下来，
    客户端向服务器发送 :ref:`SET` 命令：

    ::

        redis> SET book-name "Mastering C++ in 21 days"
        QUEUED

    因为客户端正处于事务状态，
    并且 :ref:`SET` 命令不是 :ref:`EXEC` 、 :ref:`DISCARD` 、 :ref:`WATCH` 或 :ref:`MULTI` 命令的其中一个，
    所以这个命令不会被直接执行，
    而是被保存到事务队列里面。

    表 TABLE_FIRST_STEP 记录了 :ref:`SET` 命令入队之后，
    事务队列的状态。

    ----

    表 TABLE_FIRST_STEP    :ref:`SET` 命令入队之后的事务队列

    =========  =============== ===========================================================   ==================
    数组索引    cmd             argv                                                            argc
    =========  =============== ===========================================================   ==================
    ``0``       ``SET``         ``["book-name", "Mastering C++ in 21 days"]``                   ``2``
    =========  =============== ===========================================================   ==================

    ----

    因为 :ref:`SET` 命令是第一个入队的命令，
    所以它会放在事务队列的索引 ``0`` 上面。

    接下来，
    客户端继续向服务器发送 :ref:`GET` 命令：

    ::

        redis> GET book-name
        QUEUED

    和前面的 :ref:`SET` 命令类似，
    这个 :ref:`GET` 命令也不会被立即执行，
    而是被放到事务队列的索引 ``1`` 上面。

    表 TABLE_SECOND_STEP 记录了 :ref:`GET` 命令入队之后，
    事务队列的状态。

    ----

    表 TABLE_SECOND_STEP    :ref:`GET` 命令入队之后的事务队列

    =========  =============== ===========================================================   ==================
    数组索引    cmd             argv                                                            argc
    =========  =============== ===========================================================   ==================
    ``0``       ``SET``         ``["book-name", "Mastering C++ in 21 days"]``                   ``2``

    ``1``       ``GET``         ``["book-name"]``                                               ``1``
    =========  =============== ===========================================================   ==================

    ----

    之后，
    客户端继续发送 :ref:`SADD` 命令和 :ref:`SMEMBERS` 命令，
    这两个命令将分别入队到事务队列的索引 ``2`` 和索引 ``3`` 里面：

    ::

        redis> SADD tag "C++" "Programming" "Mastering Series"
        QUEUED

        redis> SMEMBERS tag
        QUEUED

    表 TABLE_THREETH_STEP 记录了 :ref:`SADD` 和 :ref:`SMEMBERS` 命令入队之后，
    事务队列的状态。

    ----

    表 TABLE_THREETH_STEP    :ref:`SADD` 和 :ref:`SMEMBERS` 命令入队之后的事务队列

    .. include:: _example_of_transaction_queue.include

    ----

    最后，
    客户端向服务器发送 :ref:`EXEC` 命令，
    这个命令会立即执行，
    触发事务队列中的所有命令被执行：

    - 首先被执行的是事务队列中索引为 ``0`` 的 :ref:`SET` 命令，
      这个命令产生一个 status reply  ``OK`` 。

    - 第二被执行的是事务队列中索引为 ``1`` 的 :ref:`GET` 命令，
      这个命令产生一个 bulk reply ``"Mastering C++ in 21 days"`` 。

    - 第三个被执行的是事务队列中索引为 ``2`` 的 :ref:`SADD` 命令，
      这个命令产生一个 integer reply ``(integer) 3`` 。

    - 最后一个被执行的是事务队列中索引为 ``3`` 的 :ref:`SMEMBERS` 命令，
      这个命令产生一个 multi bulk reply ，
      回复的三个元素分别是 ``"Mastering Series"`` 、 ``"C++"`` 和 ``"Programming"`` 。

    这些事务命令的回复首先被服务器放到一个回复队列里面，
    之后再作为 :ref:`EXEC` 命令的结果发送给客户端：

    ::

        redis> EXEC
        1) OK
        2) "Mastering C++ in 21 days"
        3) (integer) 3
        4) 1) "Mastering Series"
           2) "C++"
           3) "Programming"

    至此，
    事务执行完毕。


    事务状态下的 DISCARD 、 WATCH 和 MULTI 命令
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    除了 :ref:`EXEC` 命令以外，
    服务器在客户端处于事务状态时，
    不加入到事务队列而直接执行的另外三个命令是 :ref:`DISCARD` 、 :ref:`WATCH` 和 :ref:`MULTI` 。

    :ref:`DISCARD` 命令用于中途取消一个事务，
    当这个命令执行时，
    服务器清空客户端的事务队列，
    并将客户端从事务状态转换回非事务状态，
    最后向客户端返回 ``OK`` ，
    表示事务已被取消：

    ::

        redis redis> MULTI
        OK

        redis redis> SET book-name "Mastering C++ in 21 days"
        QUEUED

        redis redis> GET book-name
        QUEUED

        redis redis> DISCARD
        OK

    :ref:`WATCH` 命令用于监视给定的键，
    它只能在客户端进入事务状态之前执行，
    客户端在事务状态下发送 :ref:`WATCH` 命令将引发一个错误，
    但这个错误不会对客户端状态以及事务队列产生任何影响，
    命令可以继续入队，
    事务也可以正常执行：

    ::

        redis redis> MULTI
        OK

        redis redis> SET book-name "Mastering C++ in 21 days"
        QUEUED

        redis redis> WATCH book-name
        (error) ERR WATCH inside MULTI is not allowed

        redis redis> GET book-name
        QUEUED

        redis redis> EXEC
        1) OK
        2) "Mastering C++ in 21 days"


    Redis 的事务是不可嵌套的，
    已经处于事务状态的客户端向服务器再次发送 :ref:`MULTI` 命令将引发一个错误，
    但这个错误不会对客户端状态以及事务队列产生任何影响，
    命令可以继续入队，
    事务也可以正常执行
    （和处理 :ref:`WATCH` 命令的方法相同）：

    ::

        redis redis> MULTI
        OK

        redis redis> SET book-name "Mastering C++ in 21 days"
        QUEUED

        redis redis> MULTI
        (error) ERR MULTI calls can not be nested

        redis redis> GET book-name
        QUEUED
        
        redis redis> EXEC
        1) OK
        2) "Mastering C++ in 21 days"
