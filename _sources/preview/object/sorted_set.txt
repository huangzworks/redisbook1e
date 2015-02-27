有序集合对象
------------------

有序集合的编码可以是 ``ziplist`` 或者 ``skiplist`` 。

``ziplist`` 编码的有序集合对象使用压缩列表作为底层实现，
每个集合元素使用两个紧挨在一起的压缩列表节点来保存，
第一个节点保存元素的成员（member），
而第二个元素则保存元素的分值（score）。

压缩列表内的集合元素按分值从小到大进行排序，
分值较小的元素被放置在靠近表头的方向，
而分值较大的元素则被放置在靠近表尾的方向。

举个例子，
如果我们执行以下 :ref:`ZADD` 命令，
那么服务器将创建一个有序集合对象作为 ``price`` 键的值：

::

    redis> ZADD price 8.5 apple 5.0 banana 6.0 cherry
    (integer) 3

如果 ``price`` 键的值对象使用的是 ``ziplist`` 编码，
那么这个值对象将会是图 8-14 所示的样子，
而对象所使用的压缩列表则会是 8-15 所示的样子。

.. graphviz::

    digraph {

        label = "\n 图 8-14    ziplist 编码的有序集合对象";

        rankdir = LR;

        node [shape = record];

        redisObject [label = " redisObject | type \n REDIS_ZSET | encoding \n REDIS_ENCODING_ZIPLIST | <ptr> ptr | ... "];

        ziplist [label = "压缩列表", width = 4.0];

        redisObject:ptr -> ziplist;

    }

.. graphviz::

    digraph {

        label = "\n 图 8-15    有序集合元素在压缩列表中按分值从小到大排列";

        //

        node [shape = record];

        ziplist [label = " zlbytes | zltail | zllen | <banana> \"banana\" | <banana_price> 5.0 | <cherry> \"cherry\" | <cherry_price> 6.0 | <apple> \"apple\" | <apple_price> 8.5 | zlend  "];

        node [shape = plaintext];

        banana [label = "分值最少的元素"];
        cherry [label = "分值排第二的元素"];
        apple [label = "分值最大的元素"];

        //

        edge [style = dashed]

        banana -> ziplist:banana [label = "成员"];
        banana -> ziplist:banana_price [label = "分值"];

        cherry -> ziplist:cherry;
        cherry -> ziplist:cherry_price;

        apple -> ziplist:apple;
        apple -> ziplist:apple_price;

    }

``skiplist`` 编码的有序集合对象使用 ``zset`` 结构作为底层实现，
一个 ``zset`` 结构同时包含一个字典和一个跳跃表：

::

    typedef struct zset {

        zskiplist *zsl;

        dict *dict;

    } zset;

``zset`` 结构中的 ``zsl`` 跳跃表按分值从小到大保存了所有集合元素，
每个跳跃表节点都保存了一个集合元素：
跳跃表节点的 ``object`` 属性保存了元素的成员，
而跳跃表节点的 ``score`` 属性则保存了元素的分值。
通过这个跳跃表，
程序可以对有序集合进行范围型操作，
比如 :ref:`ZRANK` 、 :ref:`ZRANGE` 等命令就是基于跳跃表 API 来实现的。

除此之外，
``zset`` 结构中的 ``dict`` 字典为有序集合创建了一个从成员到分值的映射，
字典中的每个键值对都保存了一个集合元素：
字典的键保存了元素的成员，
而字典的值则保存了元素的分值。
通过这个字典，
程序可以用 :math:`O(1)` 复杂度查找给定成员的分值，
:ref:`ZSCORE` 命令就是根据这一特性实现的，
而很多其他有序集合命令都在实现的内部用到了这一特性。

有序集合每个元素的成员都是一个字符串对象，
而每个元素的分值都是一个 ``double`` 类型的浮点数。
值得一提的是，
虽然 ``zset`` 结构同时使用跳跃表和字典来保存有序集合元素，
但这两种数据结构都会通过指针来共享相同元素的成员和分值，
所以同时使用跳跃表和字典来保存集合元素不会产生任何重复成员或者分值，
也不会因此而浪费额外的内存。

.. topic:: 为什么有序集合需要同时使用跳跃表和字典来实现？

    在理论上来说，
    有序集合可以单独使用字典或者跳跃表的其中一种数据结构来实现，
    但无论单独使用字典还是跳跃表，
    在性能上对比起同时使用字典和跳跃表都会有所降低。

    举个例子，
    如果我们只使用字典来实现有序集合，
    那么虽然以 :math:`O(1)` 复杂度查找成员的分值这一特性会被保留，
    但是，
    因为字典以无序的方式来保存集合元素，
    所以每次在执行范围型操作 ——
    比如 :ref:`ZRANK` 、 :ref:`ZRANGE` 等命令时，
    程序都需要对字典保存的所有元素进行排序，
    完成这种排序需要至少 :math:`O(N \log N)` 时间复杂度，
    以及额外的 :math:`O(N)` 内存空间
    （因为要创建一个数组来保存排序后的元素）。

    另一方面，
    如果我们只使用跳跃表来实现有序集合，
    那么跳跃表执行范围型操作的所有优点都会被保留，
    但因为没有了字典，
    所以根据成员查找分值这一操作的复杂度将从 :math:`O(1)` 上升为 :math:`O(\log N)` 。

    因为以上原因，
    为了让有序集合的查找和范围型操作都尽可能快地执行，
    Redis 选择了同时使用字典和跳跃表两种数据结构来实现有序集合。

举个例子，
如果前面 ``price`` 键创建的不是 ``ziplist`` 编码的有序集合对象，
而是 ``skiplist`` 编码的有序集合对象，
那么这个有序集合对象将会是图 8-16 所示的样子，
而对象所使用的 ``zset`` 结构将会是图 8-17 所示的样子。

.. graphviz::

    digraph {

        label = "\n 图 8-16    skiplist 编码的有序集合对象";

        rankdir = LR;

        node [shape = record];

        redisObject [label = " redisObject | type \n REDIS_ZSET | encoding \n REDIS_ENCODING_SKIPLIST | <ptr> ptr | ... "];

        zset [label = " <head> zset | <dict> dict | <zsl> zsl "];

        node [shape = plaintext];

        dict [label = "..."];

        zsl [label = "..."];

        redisObject:ptr -> zset:head;
        zset:dict -> dict;
        zset:zsl -> zsl;

    }

.. graphviz::

    digraph {

        rankdir = LR;

        //

        node [shape = record];

        zset [label = " <head> zset | <dict> dict | <zsl> zsl "];

        dict [label = " <head> dict | ... | <ht0> ht[0] | ... "];

        ht0 [label = " <head> dictht | ... | <table> table | ... "];
        
        table [label = " <banana> StringObject \n \"banana\" | <apple> StringObject \n \"apple\" | <cherry> StringObject \n \"cherry\" "];

        node [shape = plaintext];

        apple_price [label = "8.5"];
        banana_price [label = "5.0"];
        cherry_price [label = "6.0"];

        //

        zset:dict -> dict:head;
        dict:ht0 -> ht0:head;
        ht0:table -> table:head;

        table:apple -> apple_price;
        table:banana -> banana_price;
        table:cherry -> cherry_price;

        //

        node [shape = record, width = "0.5"];

        //

        l [label = " <header> header | <tail> tail | level \n 5 | length \n 3 "];

        subgraph cluster_nodes {

            style = invisible;

            header [label = " <l32> L32 | ... | <l5> L5 | <l4> L4 | <l3> L3 | <l2> L2 | <l1> L1 "];

            bw_null [label = "NULL", shape = plaintext];

            level_null [label = "NULL", shape = plaintext];

            A [label = " <l4> L4 | <l3> L3 | <l2> L2 | <l1> L1 | <backward> BW | 5.0 | StringObject \n \"banana\" "];

            B [label = " <l2> L2 | <l1> L1 | <backward> BW | 6.0 | StringObject \n \"cherry\" "];

            C [label = " <l5> L5 | <l4> L4 | <l3> L3 | <l2> L2 | <l1> L1 | <backward> BW | 8.5 | StringObject \n \"apple\" "];

        }

        subgraph cluster_nulls {

            style = invisible;

            n1 [label = "NULL", shape = plaintext];
            n2 [label = "NULL", shape = plaintext];
            n3 [label = "NULL", shape = plaintext];
            n4 [label = "NULL", shape = plaintext];
            n5 [label = "NULL", shape = plaintext];

        }

        //

        l:header -> header;
        l:tail -> C;

        header:l32 -> level_null;
        header:l5 -> C:l5;
        header:l4 -> A:l4;
        header:l3 -> A:l3;
        header:l2 -> A:l2;
        header:l1 -> A:l1;

        A:l4 -> C:l4;
        A:l3 -> C:l3;
        A:l2 -> B:l2;
        A:l1 -> B:l1;

        B:l2 -> C:l2;
        B:l1 -> C:l1;

        C:l5 -> n5;
        C:l4 -> n4;
        C:l3 -> n3;
        C:l2 -> n2;
        C:l1 -> n1;

        bw_null -> A:backward -> B:backward -> C:backward [dir = back];

        zset:zsl -> l:header;

        // HACK: 放在开头的话 NULL 指针的长度会有异样
        label = "\n 图 8-17    有序集合元素同时被保存在字典和跳跃表中";

    }

.. topic:: 注意

    为了展示方便，
    图 8-17 在字典和跳跃表中重复展示了各个元素的成员和分值，
    但在实际中，
    字典和跳跃表会共享元素的成员和分值，
    所以并不会造成任何数据重复，
    也不会因此而浪费任何内存。


编码的转换
^^^^^^^^^^^^^^^^^^^

当有序集合对象可以同时满足以下两个条件时，
对象使用 ``ziplist`` 编码：

1. 有序集合保存的元素数量小于 ``128`` 个；

2. 有序集合保存的所有元素成员的长度都小于 ``64`` 字节；

不能满足以上两个条件的有序集合对象将使用 ``skiplist`` 编码。

.. topic:: 注意

    以上两个条件的上限值是可以修改的，
    具体请看配置文件中关于 ``zset-max-ziplist-entries`` 选项和 ``zset-max-ziplist-value`` 选项的说明。

对于使用 ``ziplist`` 编码的有序集合对象来说，
当使用 ``ziplist`` 编码所需的两个条件中的任意一个不能被满足时，
程序就会执行编码转换操作，
将原本储存在压缩列表里面的所有集合元素转移到 ``zset`` 结构里面，
并将对象的编码从 ``ziplist`` 改为 ``skiplist`` 。

以下代码展示了有序集合对象因为包含了过多元素而引发编码转换的情况：

::

    # 对象包含了 128 个元素
    redis> EVAL "for i=1, 128 do redis.call('ZADD', KEYS[1], i, i) end" 1 numbers
    (nil)

    redis> ZCARD numbers
    (integer) 128

    redis> OBJECT ENCODING numbers
    "ziplist"

    # 再添加一个新元素
    redis> ZADD numbers 3.14 pi
    (integer) 1

    # 对象包含的元素数量变为 129 个
    redis> ZCARD numbers
    (integer) 129

    # 编码已改变
    redis> OBJECT ENCODING numbers
    "skiplist"

以下代码则展示了有序集合对象因为元素的成员过长而引发编码转换的情况：

::

    # 向有序集合添加一个成员只有三字节长的元素
    redis> ZADD blah 1.0 www
    (integer) 1

    redis> OBJECT ENCODING blah
    "ziplist"

    # 向有序集合添加一个成员为 66 字节长的元素
    redis> ZADD blah 2.0 oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo
    (integer) 1

    # 编码已改变
    redis> OBJECT ENCODING blah
    "skiplist"


有序集合命令的实现
^^^^^^^^^^^^^^^^^^^^

因为有序集合键的值为有序集合对象，
所以用于有序集合键的所有命令都是针对有序集合对象来构建的，
表 8-11 列出了其中一部分有序集合键命令，
以及这些命令在不同编码的有序集合对象下的实现方法。

--------------------------------------------------------------------------------------------------------------------------

表 8-11    有序集合命令的实现方法

+-------------------+-----------------------------------------------+---------------------------------------------------+
| 命令              | ``ziplist`` 编码的实现方法                    | ``zset`` 编码的实现方法                           |
+===================+===============================================+===================================================+
| :ref:`ZADD`       | 调用 ``ziplistInsert`` 函数，                 | 先调用 ``zslInsert`` 函数，                       |
|                   | 将成员和分值作为两个节点分别插入到压缩列表。  | 将新元素添加到跳跃表，                            |
|                   |                                               | 然后调用 ``dictAdd`` 函数，                       |
|                   |                                               | 将新元素关联到字典。                              |
+-------------------+-----------------------------------------------+---------------------------------------------------+
| :ref:`ZCARD`      | 调用 ``ziplistLen`` 函数，                    | 访问跳跃表数据结构的 ``length`` 属性，            |
|                   | 获得压缩列表包含节点的数量，                  | 直接返回集合元素的数量。                          |
|                   | 将这个数量除以 ``2`` 得出集合元素的数量。     |                                                   |
+-------------------+-----------------------------------------------+---------------------------------------------------+
| :ref:`ZCOUNT`     | 遍历压缩列表，                                | 遍历跳跃表，                                      |
|                   | 统计分值在给定范围内的节点的数量。            | 统计分值在给定范围内的节点的数量。                |
+-------------------+-----------------------------------------------+---------------------------------------------------+
| :ref:`ZRANGE`     | 从表头向表尾遍历压缩列表，                    | 从表头向表尾遍历跳跃表，                          |
|                   | 返回给定索引范围内的所有元素。                | 返回给定索引范围内的所有元素。                    |
+-------------------+-----------------------------------------------+---------------------------------------------------+
| :ref:`ZREVRANGE`  | 从表尾向表头遍历压缩列表，                    | 从表尾向表头遍历跳跃表，                          |
|                   | 返回给定索引范围内的所有元素。                | 返回给定索引范围内的所有元素。                    |
+-------------------+-----------------------------------------------+---------------------------------------------------+
| :ref:`ZRANK`      | 从表头向表尾遍历压缩列表，                    | 从表头向表尾遍历跳跃表，                          |
|                   | 查找给定的成员，                              | 查找给定的成员，                                  |
|                   | 沿途记录经过节点的数量，                      | 沿途记录经过节点的数量，                          |
|                   | 当找到给定成员之后，                          | 当找到给定成员之后，                              |
|                   | 途经节点的数量就是该成员所对应元素的排名。    | 途经节点的数量就是该成员所对应元素的排名。        |
+-------------------+-----------------------------------------------+---------------------------------------------------+
| :ref:`ZREVRANK`   | 从表尾向表头遍历压缩列表，                    | 从表尾向表头遍历跳跃表，                          |
|                   | 查找给定的成员，                              | 查找给定的成员，                                  |
|                   | 沿途记录经过节点的数量，                      | 沿途记录经过节点的数量，                          |
|                   | 当找到给定成员之后，                          | 当找到给定成员之后，                              |
|                   | 途经节点的数量就是该成员所对应元素的排名。    | 途经节点的数量就是该成员所对应元素的排名。        |
+-------------------+-----------------------------------------------+---------------------------------------------------+
| :ref:`ZREM`       | 遍历压缩列表，                                | 遍历跳跃表，                                      |
|                   | 删除所有包含给定成员的节点，                  | 删除所有包含了给定成员的跳跃表节点。              |
|                   | 以及被删除成员节点旁边的分值节点。            | 并在字典中解除被删除元素的成员和分值的关联。      |
+-------------------+-----------------------------------------------+---------------------------------------------------+
| :ref:`ZSCORE`     | 遍历压缩列表，                                | 直接从字典中取出给定成员的分值。                  |
|                   | 查找包含了给定成员的节点，                    |                                                   |
|                   | 然后取出成员节点旁边的分值节点保存的元素分值。|                                                   |
+-------------------+-----------------------------------------------+---------------------------------------------------+
