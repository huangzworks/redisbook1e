渐进式 rehash
-----------------------

上一节说过，
扩展或收缩哈希表需要将 ``ht[0]`` 里面的所有键值对 rehash 到 ``ht[1]`` 里面，
但是，
这个 rehash 动作并不是一次性、集中式地完成的，
而是分多次、渐进式地完成的。

这样做的原因在于，
如果 ``ht[0]`` 里只保存着四个键值对，
那么服务器可以在瞬间就将这些键值对全部 rehash 到 ``ht[1]`` ；
但是，
如果哈希表里保存的键值对数量不是四个，
而是四百万、四千万甚至四亿个键值对，
那么要一次性将这些键值对全部 rehash 到 ``ht[1]`` 的话，
庞大的计算量可能会导致服务器在一段时间内停止服务。

因此，
为了避免 rehash 对服务器性能造成影响，
服务器不是一次性将 ``ht[0]`` 里面的所有键值对全部 rehash 到 ``ht[1]`` ，
而是分多次、渐进式地将 ``ht[0]`` 里面的键值对慢慢地 rehash 到 ``ht[1]`` 。

以下是哈希表渐进式 rehash 的详细步骤：

1. 为 ``ht[1]`` 分配空间，
   让字典同时持有 ``ht[0]`` 和 ``ht[1]`` 两个哈希表。

2. 在字典中维持一个索引计数器变量 ``rehashidx`` ，
   并将它的值设置为 ``0`` ，
   表示 rehash 工作正式开始。

3. 在 rehash 进行期间，
   每次对字典执行添加、删除、查找或者更新操作时，
   程序除了执行指定的操作以外，
   还会顺带将 ``ht[0]`` 哈希表在 ``rehashidx`` 索引上的所有键值对 rehash 到 ``ht[1]`` ，
   当 rehash 工作完成之后，
   程序将 ``rehashidx`` 属性的值增一。

4. 随着字典操作的不断执行，
   最终在某个时间点上，
   ``ht[0]`` 的所有键值对都会被 rehash 至 ``ht[1]`` ，
   这时程序将 ``rehashidx`` 属性的值设为 ``-1`` ，
   表示 rehash 操作已完成。

渐进式 rehash 的好处在于它采取分而治之的方式，
将 rehash 键值对所需的计算工作均滩到对字典的每个添加、删除、查找和更新操作上，
从而避免了集中式 rehash 而带来的庞大计算量。

图 4-12 至图 4-17 展示了一次完整的渐进式 rehash 过程，
注意观察在整个 rehash 过程中，
字典的 ``rehashidx`` 属性是如何变化的。

.. graphviz::

    digraph {

        label = "\n 图 4-12    准备开始 rehash";

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

        label = "\n 图 4-13    rehash 索引 0 上的键值对";

        rankdir = LR;

        node [shape = record];

        // 字典

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n 0 "];

        // 哈希表

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 3"];

        dictht1 [label = " <head> dictht | <table> table | <size> size \n 8 | <sizemask> sizemask \n 7 | <used> used \n 1"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];

        table1 [label = " <head> dictEntry*[8] | ... | <4> 4 | ... "];

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
        table0:1 -> kv0:head -> null1;
        table0:2 -> kv3:head -> null2;
        table0:3 -> kv1:head -> null3;

        table1:4 -> kv2:head -> null14

    }

.. graphviz::

    digraph {

        label = "\n 图 4-14    rehash 索引 1 上的键值对";

        rankdir = LR;

        node [shape = record];

        // 字典

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n 1 "];

        // 哈希表

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 2"];

        dictht1 [label = " <head> dictht | <table> table | <size> size \n 8 | <sizemask> sizemask \n 7 | <used> used \n 2"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];

        table1 [label = " <head> dictEntry*[8] | ... | <4> 4 | <5> 5 | ... "];

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
        table0:2 -> kv3:head -> null2;
        table0:3 -> kv1:head -> null3;

        table1:4 -> kv2:head -> null14
        table1:5 -> kv0:head -> null15;

    }

.. graphviz::

    digraph {

        label = "\n 图 4-15    rehash 索引 2 上的键值对";

        rankdir = LR;

        node [shape = record];

        // 字典

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n 2 "];

        // 哈希表

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 1"];

        dictht1 [label = " <head> dictht | <table> table | <size> size \n 8 | <sizemask> sizemask \n 7 | <used> used \n 3"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];

        table1 [label = " <head> dictEntry*[8] | ... | <1> 1 | ... | <4> 4 | <5> 5 | ... "];

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
        table0:3 -> kv1:head -> null3;

        table1:1 -> kv3:head -> null11;
        table1:4 -> kv2:head -> null14
        table1:5 -> kv0:head -> null15;

    }

.. graphviz::

    digraph {

        label = "\n 图 4-16    rehash 索引 3 上的键值对";

        rankdir = LR;

        node [shape = record];

        // 字典

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n 3 "];

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
        table1:4 -> kv2:head -> null14
        table1:5 -> kv0:head -> null15;
        table1:7 -> kv1:head -> null17;

    }

.. graphviz::

    digraph {

        label = "\n 图 4-17    rehash 执行完毕";

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


渐进式 rehash 执行期间的哈希表操作
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

因为在进行渐进式 rehash 的过程中，
字典会同时使用 ``ht[0]`` 和 ``ht[1]`` 两个哈希表，
所以在渐进式 rehash 进行期间，
字典的删除（delete）、查找（find）、更新（update）等操作会在两个哈希表上进行：
比如说，
要在字典里面查找一个键的话，
程序会先在 ``ht[0]`` 里面进行查找，
如果没找到的话，
就会继续到 ``ht[1]`` 里面进行查找，
诸如此类。

另外，
在渐进式 rehash 执行期间，
新添加到字典的键值对一律会被保存到 ``ht[1]`` 里面，
而 ``ht[0]`` 则不再进行任何添加操作：
这一措施保证了 ``ht[0]`` 包含的键值对数量会只减不增，
并随着 rehash 操作的执行而最终变成空表。
