rehash
--------------

随着操作的不断执行，
哈希表保存的键值对会逐渐地增多或者减少，
为了让哈希表的负载因子（load factor）维持在一个合理的范围之内，
当哈希表保存的键值对数量太多或者太少时，
程序需要对哈希表的大小进行相应的扩展或者收缩。

扩展和收缩哈希表的工作可以通过执行 rehash （重新散列）操作来完成，
Redis 对字典的哈希表执行 rehash 的步骤如下：

1. 为字典的 ``ht[1]`` 哈希表分配空间，
   这个哈希表的空间大小取决于要执行的操作，
   以及 ``ht[0]`` 当前包含的键值对数量
   （也即是 ``ht[0].used`` 属性的值）：
   
   - 如果执行的是扩展操作，
     那么 ``ht[1]`` 的大小为第一个大于等于 ``ht[0].used * 2`` 的 :math:`2^n` （\ ``2`` 的 ``n`` 次方幂）；
   
   - 如果执行的是收缩操作，
     那么 ``ht[1]`` 的大小为第一个大于等于 ``ht[0].used`` 的 :math:`2^n` 。

2. 将保存在 ``ht[0]`` 中的所有键值对 rehash 到 ``ht[1]`` 上面：
   rehash 指的是重新计算键的哈希值和索引值，
   然后将键值对放置到 ``ht[1]`` 哈希表的指定位置上。

3. 当 ``ht[0]`` 包含的所有键值对都迁移到了 ``ht[1]`` 之后
   （\ ``ht[0]`` 变为空表\ ），
   释放 ``ht[0]`` ，
   将 ``ht[1]`` 设置为 ``ht[0]`` ，
   并在 ``ht[1]`` 新创建一个空白哈希表，
   为下一次 rehash 做准备。

举个例子，
假设程序要对图 4-8 所示字典的 ``ht[0]`` 进行扩展操作，
那么程序将执行以下步骤：

1. ``ht[0].used`` 当前的值为 ``4`` ，
   ``4 * 2 = 8`` ，
   而 ``8`` （\ :math:`2^3`\ ）恰好是第一个大于等于 ``4`` 的 ``2`` 的 ``n`` 次方，
   所以程序会将 ``ht[1]`` 哈希表的大小设置为 ``8`` 。
   图 4-9 展示了 ``ht[1]`` 在分配空间之后，
   字典的样子。

2. 将 ``ht[0]`` 包含的四个键值对都 rehash 到 ``ht[1]`` ，
   如图 4-10 所示。

3. 释放 ``ht[0]`` ，并将 ``ht[1]`` 设置为 ``ht[0]`` ，然后为 ``ht[1]`` 分配一个空白哈希表，如图 4-11 所示。

至此，
对哈希表的扩展操作执行完毕，
程序成功将哈希表的大小从原来的 ``4`` 改为了现在的 ``8`` 。

.. graphviz::

    digraph {

        label = "\n 图 4-8    执行 rehash 之前的字典";

        rankdir = LR;

        node [shape = record];

        // 字典

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n -1 "];

        // 哈希表

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 4"];

        dictht1 [label = " <head> dictht | <table> table | <size> size \n 0 | <sizemask> sizemask \n 0 | <used> used \n 0"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];

        table1 [label = "NULL", shape = plaintext];

        // 哈希表节点

        kv0 [label = " <head> dictEntry | { k0 | v0 } "];
        kv1 [label = " <head> dictEntry | { k1 | v1 } "];
        kv2 [label = " <head> dictEntry | { k2 | v2 } "];
        kv3 [label = " <head> dictEntry | { k3 | v3 } "];

        //

        node [shape = plaintext, label = "NULL"];

        null0;
        null1;
        null2;
        null3;

        //

        dict:ht -> dictht0:head [label = "ht[0]"];
        dict:ht -> dictht1:head [label = "ht[1]"];

        dictht0:table -> table0:head;
        dictht1:table -> table1;

        table0:0 -> kv2:head -> null0;
        table0:1 -> kv0:head -> null1;
        table0:2 -> kv3:head -> null2;
        table0:3 -> kv1:head -> null3;

    }

.. graphviz::

    digraph {

        label = "\n 图 4-9    为字典的 ht[1] 哈希表分配空间";

        rankdir = LR;

        node [shape = record];

        // 字典

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n -1 "];

        // 哈希表

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 4"];

        dictht1 [label = " <head> dictht | <table> table | <size> size \n 8 | <sizemask> sizemask \n 7 | <used> used \n 0"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];

        table1 [label = " <head> dictEntry*[8] | <0> 0 | <1> 1 | <2> 2 | ... | <7> 7 "];

        // 哈希表节点

        kv0 [label = " <head> dictEntry | { k0 | v0 } "];
        kv1 [label = " <head> dictEntry | { k1 | v1 } "];
        kv2 [label = " <head> dictEntry | { k2 | v2 } "];
        kv3 [label = " <head> dictEntry | { k3 | v3 } "];

        //

        node [shape = plaintext, label = "NULL"];

        //

        dict:ht -> dictht0:head [label = "ht[0]"];
        dict:ht -> dictht1:head [label = "ht[1]"];

        dictht0:table -> table0:head;
        dictht1:table -> table1:head;

        table0:0 -> kv2:head -> null0;
        table0:1 -> kv0:head -> null1;
        table0:2 -> kv3:head -> null2;
        table0:3 -> kv1:head -> null3;

        table1:0 -> null10;
        table1:1 -> null11;
        table1:2 -> null12;
        table1:7 -> null17;

    }

.. graphviz::

    digraph {

        label = "\n 图 4-10    ht[0] 的所有键值对都已经被迁移到 ht[1]";

        rankdir = LR;

        node [shape = record];

        // 字典

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n -1 "];

        // 哈希表

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 0"];

        dictht1 [label = " <head> dictht | <table> table | <size> size \n 8 | <sizemask> sizemask \n 7 | <used> used \n 4"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];

        table1 [label = " <head> dictEntry*[8] | ... | <1> 1 | ... | <4> 4 | <5> 5 | ... | <7> 7 "];

        // 哈希表节点

        kv0 [label = " <head> dictEntry | { k0 | v0 } "];
        kv1 [label = " <head> dictEntry | { k1 | v1 } "];
        kv2 [label = " <head> dictEntry | { k2 | v2 } "];
        kv3 [label = " <head> dictEntry | { k3 | v3 } "];

        //

        node [shape = plaintext, label = "NULL"];

        //

        dict:ht -> dictht0:head [label = "ht[0]"];
        dict:ht -> dictht1:head [label = "ht[1]"];

        dictht0:table -> table0:head;
        dictht1:table -> table1:head;

        table0:0 -> null0;
        table0:1 -> null1;
        table0:2 -> null2;
        table0:3 -> null3;

        table1:1 -> kv3:head -> null11;
        table1:4 -> kv2:head -> null14;
        table1:5 -> kv0:head -> null15;
        table1:7 -> kv1:head -> null17;

    }


.. graphviz::

    digraph {

        label = "\n 图 4-11    完成 rehash 之后的字典";

        rankdir = LR;

        node [shape = record];

        // 字典

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n -1 "];

        // 哈希表

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 8 | <sizemask> sizemask \n 7 | <used> used \n 4"];

        dictht1 [label = " <head> dictht | <table> table | <size> size \n 0 | <sizemask> sizemask \n 0 | <used> used \n 0"];

        table0 [label = " <head> dictEntry*[8] | ... | <1> 1 | ... | <4> 4 | <5> 5 | ... | <7> 7 "];

        table1 [label = "NULL", shape = plaintext];

        // 哈希表节点

        kv0 [label = " <head> dictEntry | { k0 | v0 } "];
        kv1 [label = " <head> dictEntry | { k1 | v1 } "];
        kv2 [label = " <head> dictEntry | { k2 | v2 } "];
        kv3 [label = " <head> dictEntry | { k3 | v3 } "];

        //

        node [shape = plaintext, label = "NULL"];

        //

        dict:ht -> dictht0:head [label = "ht[0]"];
        dict:ht -> dictht1:head [label = "ht[1]"];

        dictht0:table -> table0:head;
        dictht1:table -> table1;

        table0:1 -> kv3:head -> null11;
        table0:4 -> kv2:head -> null14;
        table0:5 -> kv0:head -> null15;
        table0:7 -> kv1:head -> null17;

    }


哈希表的扩展与收缩
^^^^^^^^^^^^^^^^^^^^

当以下条件中的任意一个被满足时，
程序会自动开始对哈希表执行扩展操作：

1. 服务器目前没有在执行 :ref:`BGSAVE` 命令或者 :ref:`BGREWRITEAOF` 命令，
   并且哈希表的负载因子大于等于 ``1`` ；

2. 服务器目前正在执行 :ref:`BGSAVE` 命令或者 :ref:`BGREWRITEAOF` 命令，
   并且哈希表的负载因子大于等于 ``5`` ；

其中哈希表的负载因子可以通过公式：

::

    # 负载因子 = 哈希表已保存节点数量 / 哈希表大小
    load_factor = ht[0].used / ht[0].size

计算得出。

比如说，
对于一个大小为 ``4`` ，
包含 ``4`` 个键值对的哈希表来说，
这个哈希表的负载因子为：

::

    load_factor = 4 / 4 = 1

又比如说，
对于一个大小为 ``512`` ，
包含 ``256`` 个键值对的哈希表来说，
这个哈希表的负载因子为：

::

    load_factor = 256 / 512 = 0.5

根据 :ref:`BGSAVE` 命令或 :ref:`BGREWRITEAOF` 命令是否正在执行，
服务器执行扩展操作所需的负载因子并不相同，
这是因为在执行 :ref:`BGSAVE` 命令或 :ref:`BGREWRITEAOF` 命令的过程中，
Redis 需要创建当前服务器进程的子进程，
而大多数操作系统都采用写时复制（\ `copy-on-write <http://en.wikipedia.org/wiki/Copy-on-write>`_\ ）技术来优化子进程的使用效率，
所以在子进程存在期间，
服务器会提高执行扩展操作所需的负载因子，
从而尽可能地避免在子进程存在期间进行哈希表扩展操作，
这可以避免不必要的内存写入操作，
最大限度地节约内存。

另一方面，
当哈希表的负载因子小于 ``0.1`` 时，
程序自动开始对哈希表执行收缩操作。

.. 实际上只会对数据库字典和哈希键字典进行收缩，
.. 像是 server.pubsub_channels 这种字典就自会扩展不会收缩
