解决键冲突
--------------

当有两个或以上数量的键被分配到了哈希表数组的同一个索引上面时，
我们称这些键发生了冲突（collision）。

Redis 的哈希表使用链地址法（separate chaining）来解决键冲突：
每个哈希表节点都有一个 ``next`` 指针，
多个哈希表节点可以用 ``next`` 指针构成一个单向链表，
被分配到同一个索引上的多个节点可以用这个单向链表连接起来，
这就解决了键冲突的问题。

举个例子，
假设程序要将键值对 ``k2`` 和 ``v2`` 添加到图 4-6 所示的哈希表里面，
并且计算得出 ``k2`` 的索引值为 ``2`` ，
那么键 ``k1`` 和 ``k2`` 将产生冲突，
而解决冲突的办法就是使用 ``next`` 指针将键 ``k2`` 和 ``k1`` 所在的节点连接起来，
如图 4-7 所示。

.. graphviz::

    digraph {

        label = "\n 图 4-6    一个包含两个键值对的哈希表";

        rankdir = LR;

        //

        node [shape = record];

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 2"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];
        //table1 [label = "NULL", shape = plaintext];

        kv0 [label = " <head> dictEntry | { k0 | v0 } "];
        kv1 [label = " <head> dictEntry | { k1 | v1 } "];

        //

        node [shape = plaintext, label = "NULL"];

        null0;
        null1;
        null2;
        null3;

        //

        dictht0:table -> table0:head;

        table0:0 -> kv0:head;
        kv0:head -> null0 [label = "next"];
        table0:1 -> null1;
        table0:2 -> kv1:head;
        kv1:head -> null2 [label = "next"];
        table0:3 -> null3;

    }

.. graphviz::

    digraph {

        label = "\n 图 4-7    使用链表解决 k2 和 k1 的冲突";

        rankdir = LR;

        //

        node [shape = record];

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 3"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];
        //table1 [label = "NULL", shape = plaintext];

        kv0 [label = " <head> dictEntry | { k0 | v0 } "];
        kv1 [label = " <head> dictEntry | { k1 | v1 } "];
        kv2 [label = " <head> dictEntry | { k2 | v2 } "];

        //

        node [shape = plaintext, label = "NULL"];

        null0;
        null1;
        null2;
        null3;

        //

        dictht0:table -> table0:head;

        table0:0 -> kv0:head;
        kv0:head -> null0 [label = "next"];
        table0:1 -> null1;
        table0:2 -> kv2:head;
        kv2:head -> kv1:head [label = "next"];
        kv1:head -> null2 [label = "next"];
        table0:3 -> null3;

    }

因为 ``dictEntry`` 节点组成的链表没有指向链表表尾的指针，
所以为了速度考虑，
程序总是将新节点添加到链表的表头位置（复杂度为 :math:`O(1)`\ ），
排在其他已有节点的前面。
