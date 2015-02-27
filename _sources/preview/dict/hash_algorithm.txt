哈希算法
------------

当要将一个新的键值对添加到字典里面时，
程序需要先根据键值对的键计算出哈希值和索引值，
然后再根据索引值，
将包含新键值对的哈希表节点放到哈希表数组的指定索引上面。

Redis 计算哈希值和索引值的方法如下：

::

    # 使用字典设置的哈希函数，计算键 key 的哈希值
    hash = dict->type->hashFunction(key);

    # 使用哈希表的 sizemask 属性和哈希值，计算出索引值
    # 根据情况不同， ht[x] 可以是 ht[0] 或者 ht[1]
    index = hash & dict->ht[x].sizemask;

.. graphviz::

    digraph {

        label = "\n 图 4-4    空字典";

        rankdir = LR;

        //

        node [shape = record];

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n -1 "];

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 0"];

        dictht1 [label = "...", shape = plaintext];
        //dictht1 [label = " <head> dictht | <table> table | <size> size \n 0 | <sizemask> sizemask \n 0 | <used> used \n 0"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];
        //table1 [label = "NULL", shape = plaintext];

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
        //dictht1:table -> table1;

        table0:0 -> null0;
        table0:1 -> null1;
        table0:2 -> null2;
        table0:3 -> null3;

    }

举个例子，
对于图 4-4 所示的字典来说，
如果我们要将一个键值对 ``k0`` 和 ``v0`` 添加到字典里面，
那么程序会先使用语句：

::

    hash = dict->type->hashFunction(k0);

计算键 ``k0`` 的哈希值。

假设计算得出的哈希值为 ``8`` ，
那么程序会继续使用语句：

::

    index = hash & dict->ht[0].sizemask = 8 & 3 = 0;

计算出键 ``k0`` 的索引值 ``0`` ，
这表示包含键值对 ``k0`` 和 ``v0`` 的节点应该被放置到哈希表数组的索引 ``0`` 位置上，
如图 4-5 所示。

.. graphviz::

    digraph {

        label = "\n 图 4-5    添加键值对 k0 和 v0 之后的字典";

        rankdir = LR;

        //

        node [shape = record];

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n -1 "];

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 1"];

        dictht1 [label = "...", shape = plaintext];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];
        //table1 [label = "NULL", shape = plaintext];

        dictEntry [label = " <head> dictEntry | { k0 | v0 } "];

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
        //dictht1:table -> table1;

        table0:0 -> dictEntry:head -> null0;
        table0:1 -> null1;
        table0:2 -> null2;
        table0:3 -> null3;

    }

当字典被用作数据库的底层实现，
或者哈希键的底层实现时，
Redis 使用 MurmurHash2 算法来计算键的哈希值。

MurmurHash 算法最初由 Austin Appleby 于 2008 年发明，
这种算法的优点在于，
即使输入的键是有规律的，
算法仍能给出一个很好的随机分布性，
并且算法的计算速度也非常快。

MurmurHash 算法目前的最新版本为 MurmurHash3 ，
而 Redis 使用的是 MurmurHash2 ，
关于 MurmurHash 算法的更多信息可以参考该算法的主页：
http://code.google.com/p/smhasher/ 。

..  这里为了简洁期间，把 djb 算法的说明省略了。
    djb 算法的一个大小写无关（case insensitive）版本：该算法的原始版本由 Daniel J. Bernstein 发明，这种算法的实现简单，效率高且分布性良好。更详细的信息请参考： http://www.cse.yorku.ca/~oz/hash.html 。
