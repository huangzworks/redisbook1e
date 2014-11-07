类型检查与命令多态
--------------------------------------------

Redis 中用于操作键的命令基本上可以分为两种类型。

其中一种命令可以对任何类型的键执行，
比如说 :ref:`DEL` 命令、 :ref:`EXPIRE` 命令、 :ref:`RENAME` 命令、 :ref:`TYPE` 命令、 :ref:`OBJECT` 命令，
等等。

举个例子，
以下代码就展示了使用 :ref:`DEL` 命令来删除三种不同类型的键：

::

    # 字符串键
    redis> SET msg "hello"
    OK

    # 列表键
    redis> RPUSH numbers 1 2 3 
    (integer) 3

    # 集合键
    redis> SADD fruits apple banana cherry
    (integer) 3

    redis> DEL msg
    (integer) 1

    redis> DEL numbers
    (integer) 1

    redis> DEL fruits
    (integer) 1

而另一种命令只能对特定类型的键执行，
比如说：

- :ref:`SET` 、 :ref:`GET` 、 :ref:`APPEND` 、 :ref:`STRLEN` 等命令只能对字符串键执行；

- :ref:`HDEL` 、 :ref:`HSET` 、 :ref:`HGET` 、 :ref:`HLEN` 等命令只能对哈希键执行；

- :ref:`RPUSH` 、 :ref:`LPOP` 、 :ref:`LINSERT` 、 :ref:`LLEN` 等命令只能对列表键执行；

- :ref:`SADD` 、 :ref:`SPOP` 、 :ref:`SINTER` 、 :ref:`SCARD` 等命令只能对集合键执行；

- :ref:`ZADD` 、 :ref:`ZCARD` 、 :ref:`ZRANK` 、 :ref:`ZSCORE` 等命令只能对有序集合键执行；

诸如此类。

举个例子，
我们可以用 :ref:`SET` 命令创建一个字符串键，
然后用 :ref:`GET` 命令和 :ref:`APPEND` 命令操作这个键，
但如果我们试图对这个字符串键执行只有列表键才能执行的 :ref:`LLEN` 命令，
那么 Redis 将向我们返回一个类型错误：

::

    redis> SET msg "hello world"
    OK

    redis> GET msg
    "hello world"

    redis> APPEND msg " again!"
    (integer) 18

    redis> GET msg
    "hello world again!"

    redis> LLEN msg
    (error) WRONGTYPE Operation against a key holding the wrong kind of value


类型检查的实现
^^^^^^^^^^^^^^^^^^

从上面发生类型错误的代码示例可以看出，
为了确保只有指定类型的键可以执行某些特定的命令，
在执行一个类型特定的命令之前，
Redis 会先检查输入键的类型是否正确，
然后再决定是否执行给定的命令。

类型特定命令所进行的类型检查是通过 ``redisObject`` 结构的 ``type`` 属性来实现的：

- 在执行一个类型特定命令之前，
  服务器会先检查输入数据库键的值对象是否为执行命令所需的类型，
  如果是的话，
  服务器就对键执行指定的命令；

- 否则，
  服务器将拒绝执行命令，
  并向客户端返回一个类型错误。

举个例子，
对于 :ref:`LLEN` 命令来说：

- 在执行 :ref:`LLEN` 命令之前，
  服务器会先检查输入数据库键的值对象是否为列表类型，
  也即是，
  检查值对象 ``redisObject`` 结构 ``type`` 属性的值是否为 ``REDIS_LIST`` ，
  如果是的话，
  服务器就对键执行 :ref:`LLEN` 命令；

- 否则的话，
  服务器就拒绝执行命令并向客户端返回一个类型错误；

图 8-18 展示了这一类型检查过程。

.. graphviz::

    digraph {

        label = "\n 图 8-18    LLEN 命令执行时的类型检查过程";

        //

        call_command [label = "客户端发送 LLEN <key> 命令", shape = box];

        check_type [label = "服务器检查 \n 键 key 的值对象\n是否列表对象", shape = diamond];

        execute_command [label = "对键 key 执行 LLEN 命令", shape = box];

        type_error [label = "返回一个类型错误", shape = box];

        //

        call_command -> check_type;

        check_type -> execute_command [label = "是"];

        check_type -> type_error [label = "否"];

    }

其他类型特定命令的类型检查过程也和这里展示的 :ref:`LLEN` 命令的类型检查过程类似。


多态命令的实现
^^^^^^^^^^^^^^^^^^^

Redis 除了会根据值对象的类型来判断键是否能够执行指定命令之外，
还会根据值对象的编码方式，
选择正确的命令实现代码来执行命令。

举个例子，
在前面介绍列表对象的编码时我们说过，
列表对象有 ``ziplist`` 和 ``linkedlist`` 两种编码可用，
其中前者使用压缩列表 API 来实现列表命令，
而后者则使用双端链表 API 来实现列表命令。

现在，
考虑这样一个情况，
如果我们对一个键执行 :ref:`LLEN` 命令，
那么服务器除了要确保执行命令的是列表键之外，
还需要根据键的值对象所使用的编码来选择正确的 :ref:`LLEN` 命令实现：

- 如果列表对象的编码为 ``ziplist`` ，
  那么说明列表对象的实现为压缩列表，
  程序将使用 ``ziplistLen`` 函数来返回列表的长度；

- 如果列表对象的编码为 ``linkedlist`` ，
  那么说明列表对象的实现为双端链表，
  程序将使用 ``listLength`` 函数来返回双端链表的长度；

借用面向对象方面的术语来说，
我们可以认为 :ref:`LLEN` 命令是多态（\ `polymorphism <http://en.wikipedia.org/wiki/Polymorphism_(computer_science)>`_\ ）的：
只要执行 :ref:`LLEN` 命令的是列表键，
那么无论值对象使用的是 ``ziplist`` 编码还是 ``linkedlist`` 编码，
命令都可以正常执行。

图 8-19 展示了 :ref:`LLEN` 命令从类型检查到根据编码选择实现函数的整个执行过程，
其他类型特定命令的执行过程也是类似的。

.. graphviz::

    digraph {

        label = "\n 图 8-19    LLEN 命令的执行过程";

        //

        node [shape = box];

        call_command [label = "客户端发送 LLEN <key> 命令"];

        check_type [label = "服务器检查 \n 键 key 的值对象\n是否列表对象", shape = diamond];

        //execute_command [label = "对键 key 执行 LLEN 命令"];

        select_encoding [label = "对象的编码是 \n ziplist 还是 linkedlist ？", shape = diamond];

        ziplist [label = "调用 ziplistLen 函数 \n 返回压缩列表的长度"];

        linkedlist [label = "调用 listLength 函数 \n 返回双端链表的长度"];

        type_error [label = "返回一个类型错误"];

        //

        call_command -> check_type;

        //check_type -> execute_command [label = "是"];

        check_type -> type_error [label = "否"];

        //execute_command -> select_encoding;

        check_type -> select_encoding [label = "是"];

        select_encoding -> ziplist [label = "ziplist \n 编码"];

        select_encoding -> linkedlist [label = "linkedlist \n 编码"];

    }

实际上，
我们可以将 :ref:`DEL` 、 :ref:`EXPIRE` 、 :ref:`TYPE` 等命令也称为多态命令，
因为无论输入的键是什么类型，
这些命令都可以正确地执行。

:ref:`DEL` 、 :ref:`EXPIRE` 等命令和 :ref:`LLEN` 等命令的区别在于，
前者是基于类型的多态 —— 一个命令可以同时用于处理多种不同类型的键，
而后者是基于编码的多态 —— 一个命令可以同时用于处理多种不同编码。
