字典
==========================

字典，
又称符号表（symbol table）、关联数组（associative array）或者映射（map），
是一种用于保存键值对（key-value pair）的抽象数据结构。

在字典中，
一个键（key）可以和一个值（value）进行关联（或者说将键映射为值），
这些关联的键和值就被称为键值对。

字典中的每个键都是独一无二的，
程序可以在字典中根据键查找与之关联的值，
或者通过键来更新值，
又或者根据键来删除整个键值对，
等等。

字典经常作为一种数据结构内置在很多高级编程语言里面，
但 Redis 所使用的 C 语言并没有内置这种数据结构，
因此 Redis 构建了自己的字典实现。

字典在 Redis 中的应用相当广泛，
比如 Redis 的数据库就是使用字典来作为底层实现的，
对数据库的增、删、查、改操作也是构建在对字典的操作之上的。

举个例子，
当我们执行命令：

::

    redis> SET msg "hello world"
    OK

在数据库中创建一个键为 ``"msg"`` ，
值为 ``"hello world"`` 的键值对时，
这个键值对就是保存在代表数据库的字典里面的。

除了用来表示数据库之外，
字典还是哈希键的底层实现之一：
当一个哈希键包含的键值对比较多，
又或者键值对中的元素都是比较长的字符串时，
Redis 就会使用字典作为哈希键的底层实现。

举个例子，
``website`` 是一个包含 ``10086`` 个键值对的哈希键，
这个哈希键的键都是一些数据库的名字，
而键的值就是数据库的主页网址：

::

    redis> HLEN website
    (integer) 10086

    redis> HGETALL website
    1) "Redis"
    2) "Redis.io"
    3) "MariaDB"
    4) "MariaDB.org"
    5) "MongoDB"
    6) "MongoDB.org"
    # ...

``website`` 键的底层实现就是一个字典，
字典中包含了 ``10086`` 个键值对：

- 其中一个键值对的键为 ``"Redis"`` ，
  值为 ``"Redis.io"`` 。

- 另一个键值对的键为 ``"MariaDB"`` ，
  值为 ``"MariaDB.org"`` ；

- 还有一个键值对的键为 ``"MongoDB"`` ，
  值为 ``"MongoDB.org"`` ；

诸如此类。

除了用来实现数据库和哈希键之外，
Redis 的不少功能也用到了字典，
在后续的章节中会不断地看到字典在 Redis 中的各种不同应用。

本章接下来的内容将对 Redis 的字典实现进行详细的介绍，
并列出字典的操作 API 。

本章不会对字典的基本定义和基础算法进行介绍，
如果有需要的话，
可以参考以下这些资料：

- 维基百科的 Associative Array 词条（\ http://en.wikipedia.org/wiki/Associative_array\ ）和 Hash Table 词条（\ http://en.wikipedia.org/wiki/Hash_table\ ）。

- `《算法：C 语言实现（第 1 ～ 4 部分）》 <http://book.douban.com/subject/4065258/>`_\ 一书的第 14 章。

- `《算法导论（第三版）》 <http://book.douban.com/subject/3904676/>`_ 一书的第 11 章。
