对象共享
---------------

除了用于实现引用计数内存回收机制之外，
对象的引用计数属性还带有对象共享的作用。

举个例子，
假设键 A 创建了一个包含整数值 ``100`` 的字符串对象作为值对象，
如图 8-20 所示。

.. graphviz::

    digraph {

        label = "\n 图 8-20    未被共享的字符串对象";

        rankdir = LR;

        key_a [label = "键 A", shape = box, width = 1.5];

        redisObject [label = " <head> redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_INT | <ptr> ptr | refcount \n 1 | ... ", shape = record];

        node [shape = plaintext];

        number [label = "100"]

        redisObject:ptr -> number;

        key_a -> redisObject:head;

    }

如果这时键 B 也要创建一个同样保存了整数值 ``100`` 的字符串对象作为值对象，
那么服务器有以下两种做法：

1. 为键 B 新创建一个包含整数值 ``100`` 的字符串对象；

2. 让键 A 和键 B 共享同一个字符串对象；

以上两种方法很明显是第二种方法更节约内存。

在 Redis 中，
让多个键共享同一个值对象需要执行以下两个步骤：

1. 将数据库键的值指针指向一个现有的值对象；

2. 将被共享的值对象的引用计数增一。

举个例子，
图 8-21 就展示了包含整数值 ``100`` 的字符串对象同时被键 A 和键 B 共享之后的样子，
可以看到，
除了对象的引用计数从之前的 ``1`` 变成了 ``2`` 之外，
其他属性都没有变化。

.. graphviz::

    digraph {

        label = "\n 图 8-21    被共享的字符串对象";

        rankdir = LR;

        key_a [label = "键 A", shape = box, width = 1.5];
        key_b [label = "键 B", shape = box, width = 1.5];

        redisObject [label = " <head> redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_INT | <ptr> ptr | refcount \n 2 | ... ", shape = record];

        node [shape = plaintext];

        number [label = "100"]

        redisObject:ptr -> number;

        key_a -> redisObject:head;
        key_b -> redisObject:head;

    }

共享对象机制对于节约内存非常有帮助，
数据库中保存的相同值对象越多，
对象共享机制就能节约越多的内存。

比如说，
假设数据库中保存了整数值 ``100`` 的键不只有键 A 和键 B 两个，
而是有一百个，
那么服务器只需要用一个字符串对象的内存就可以保存原本需要使用一百个字符串对象的内存才能保存的数据。

目前来说，
Redis 会在初始化服务器时，
创建一万个字符串对象，
这些对象包含了从 ``0`` 到 ``9999`` 的所有整数值，
当服务器需要用到值为 ``0`` 到 ``9999`` 的字符串对象时，
服务器就会使用这些共享对象，
而不是新创建对象。

.. topic:: 注意 

    创建共享字符串对象的数量可以通过修改 ``redis.h/REDIS_SHARED_INTEGERS`` 常量来修改。

举个例子，
如果我们创建一个值为 ``100`` 的键 ``A`` ，
并使用 :ref:`OBJECT REFCOUNT <OBJECT>` 命令查看键 ``A`` 的值对象的引用计数，
我们会发现值对象的引用计数为 ``2`` ：

::

    redis> SET A 100
    OK

    redis> OBJECT REFCOUNT A
    (integer) 2

引用这个值对象的两个程序分别是持有这个值对象的服务器程序，
以及共享这个值对象的键 ``A`` ，
如图 8-22 所示。

.. graphviz::

    digraph {

        label = "\n 图 8-22    引用数为 2 的共享对象";

        rankdir = LR;

        server [label = "服务器程序", shape = box, width = 1.5];
        key_a [label = "键 A", shape = box, width = 1.5];

        redisObject [label = " <head> redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_INT | <ptr> ptr | refcount \n 2 | ... ", shape = record];

        node [shape = plaintext];

        number [label = "100"]

        redisObject:ptr -> number;

        server -> redisObject:head;
        key_a -> redisObject:head;

    }

如果这时我们再创建一个值为 ``100`` 的键 ``B`` ，
那么键 ``B`` 也会指向包含整数值 ``100`` 的共享对象，
使得共享对象的引用计数值变为 ``3`` ：

::

    redis> SET B 100
    OK

    redis> OBJECT REFCOUNT A
    (integer) 3

    redis> OBJECT REFCOUNT B
    (integer) 3

图 8-23 展示了共享值对象的三个程序。

.. graphviz::

    digraph {

        label = "\n 图 8-23    引用数为 3 的共享对象";

        rankdir = LR;

        server [label = "服务器程序", shape = box, width = 1.5];
        key_a [label = "键 A", shape = box, width = 1.5];
        key_b [label = "键 B", shape = box, width = 1.5];

        redisObject [label = " <head> redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_INT | <ptr> ptr | refcount \n 3 | ... ", shape = record];

        node [shape = plaintext];

        number [label = "100"]

        redisObject:ptr -> number;

        key_a -> redisObject:head;
        key_b -> redisObject:head;
        server -> redisObject:head;

    }


另外，
这些共享对象不单单只有字符串键可以使用，
那些在数据结构中嵌套了字符串对象的对象（\ ``linkedlist`` 编码的列表对象、 ``hashtable`` 编码的哈希对象、 ``hashtable`` 编码的集合对象、以及 ``zset`` 编码的有序集合对象）都可以使用这些共享对象。


.. topic:: 为什么 Redis 不共享包含字符串的对象？

    当服务器考虑将一个共享对象设置为键的值对象时，
    程序需要先检查给定的共享对象和键想创建的目标对象是否完全相同，
    只有在共享对象和目标对象完全相同的情况下，
    程序才会将共享对象用作键的值对象，
    而一个共享对象保存的值越复杂，
    验证共享对象和目标对象是否相同所需的复杂度就会越高，
    消耗的 CPU 时间也会越多：

    - 如果共享对象是保存整数值的字符串对象，
      那么验证操作的复杂度为 :math:`O(1)` ；

    - 如果共享对象是保存字符串值的字符串对象，
      那么验证操作的复杂度为 :math:`O(N)` ；

    - 如果共享对象是包含了多个值（或者对象的）对象，
      比如列表对象或者哈希对象，
      那么验证操作的复杂度将会是 :math:`O(N^2)` 。

    因此，
    尽管共享更复杂的对象可以节约更多的内存，
    但受到 CPU 时间的限制，
    Redis 只对包含整数值的字符串对象进行共享。

..
    共享对象的修改方法
    ^^^^^^^^^^^^^^^^^^^^^^^^^

    !!!这个例子是错误的，因为就算对键 B 执行了 INCR ，键 B 也不会创建新的对象，只会指向 101 的共享对象，需要使用 APPEND 命令改写。!!!

    当一个修改命令对一个共享对象执行时，
    程序需要先创建共享对象的一个拷贝，
    对共享对象的引用计数属性进行减一操作，
    然后才对拷贝对象执行命令。

    继续用上面的键 A 和键 B 作为例子，
    如果我们对键 B 执行 :ref:`INCR` 命令，
    那么程序将执行以下两个步骤：

    1. 创建共享对象的拷贝对象，将共享对象的引用计数值减一，把键 B 的指针指向拷贝对象，如图 IMAGE_COPY 所示。

    2. 对键 B 的对象执行 :ref:`INCR` 命令，将对象保存的整数值增一，如图 IMAGE_INCR 所示。

    .. graphviz::

        digraph {

            rankdir = LR;

            key_a [label = "键 A", shape = circle];
            key_b [label = "键 B", shape = circle];

            redisObject [label = " <head> redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_INT | <ptr> ptr | refcount \n 1 | ... ", shape = record];
            another_redisObject [label = " <head> redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_INT | <ptr> ptr | refcount \n 1 | ... ", shape = record];

            node [shape = plaintext];

            number [label = "100"];
            another_number [label = "100"];

            redisObject:ptr -> number;
            another_redisObject:ptr -> another_number;

            key_a -> redisObject:head;
            key_b -> another_redisObject:head;

        }

    .. graphviz::

        digraph {

            rankdir = LR;

            key_a [label = "键 A", shape = circle];
            key_b [label = "键 B", shape = circle];

            redisObject [label = " <head> redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_INT | <ptr> ptr | refcount \n 1 | ... ", shape = record];
            another_redisObject [label = " <head> redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_INT | <ptr> ptr | refcount \n 1 | ... ", shape = record];

            node [shape = plaintext];

            number [label = "100"];
            another_number [label = "10087"];

            redisObject:ptr -> number;
            another_redisObject:ptr -> another_number;

            key_a -> redisObject:head;
            key_b -> another_redisObject:head;

        }
