字典的实现
-----------------

Redis 的字典使用哈希表作为底层实现，
一个哈希表里面可以有多个哈希表节点，
而每个哈希表节点就保存了字典中的一个键值对。

接下来的三个小节将分别介绍 Redis 的哈希表、哈希表节点、以及字典的实现。


哈希表
^^^^^^^^^^^^

Redis 字典所使用的哈希表由 ``dict.h/dictht`` 结构定义：

::

    typedef struct dictht {
        
        // 哈希表数组
        dictEntry **table;

        // 哈希表大小
        unsigned long size;
        
        // 哈希表大小掩码，用于计算索引值
        // 总是等于 size - 1
        unsigned long sizemask;

        // 该哈希表已有节点的数量
        unsigned long used;

    } dictht;

``table`` 属性是一个数组，
数组中的每个元素都是一个指向 ``dict.h/dictEntry`` 结构的指针，
每个 ``dictEntry`` 结构保存着一个键值对。

``size`` 属性记录了哈希表的大小，
也即是 ``table`` 数组的大小，
而 ``used`` 属性则记录了哈希表目前已有节点（键值对）的数量。

``sizemask`` 属性的值总是等于 ``size - 1`` ，
这个属性和哈希值一起决定一个键应该被放到 ``table`` 数组的哪个索引上面。

图 4-1 展示了一个大小为 ``4`` 的空哈希表
（没有包含任何键值对）。

.. graphviz::

    digraph {

        label = "\n 图 4-1    一个空的哈希表";

        rankdir = LR;

        //

        node [shape = record];

        dictht [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 0"];

        table [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];

        //

        node [shape = plaintext, label = "NULL"];

        null0;
        null1;
        null2;
        null3;

        //

        dictht:table -> table:head;

        table:0 -> null0;
        table:1 -> null1;
        table:2 -> null2;
        table:3 -> null3;

    }


哈希表节点
^^^^^^^^^^^^

哈希表节点使用 ``dictEntry`` 结构表示，
每个 ``dictEntry`` 结构都保存着一个键值对：

::

    typedef struct dictEntry {
        
        // 键
        void *key;

        // 值
        union {
            void *val;
            uint64_t u64;
            int64_t s64;
        } v;

        // 指向下个哈希表节点，形成链表
        struct dictEntry *next;

    } dictEntry;

``key`` 属性保存着键值对中的键，
而 ``v`` 属性则保存着键值对中的值，
其中键值对的值可以是一个指针，
或者是一个 ``uint64_t`` 整数，
又或者是一个 ``int64_t`` 整数。

``next`` 属性是指向另一个哈希表节点的指针，
这个指针可以将多个哈希值相同的键值对连接在一次，
以此来解决键冲突（collision）的问题。

举个例子，
图 4-2 就展示了如何通过 ``next`` 指针，
将两个索引值相同的键 ``k1`` 和 ``k0`` 连接在一起。

.. graphviz::

    digraph {

        label = "\n 图 4-2    连接在一起的键 k1 和键 k0";

        rankdir = LR;

        //

        node [shape = record];

        dictht [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 2"];

        table [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];

        dictEntry0 [label = " <head> dictEntry | { k0 | v0 }"];
        dictEntry1 [label = " <head> dictEntry | { k1 | v1 }"];

        //

        node [shape = plaintext, label = "NULL"];

        null0;
        null1;
        null2;
        null3;

        //

        dictht:table -> table:head;

        table:0 -> null0;
        table:1 -> null1;
        table:2 -> dictEntry1;
        dictEntry1 -> dictEntry0 -> null2 [label = "next"];
        table:3 -> null3;

    }


字典
^^^^^^^^^^^^

Redis 中的字典由 ``dict.h/dict`` 结构表示：

::

    typedef struct dict {

        // 类型特定函数
        dictType *type;

        // 私有数据
        void *privdata;

        // 哈希表
        dictht ht[2];

        // rehash 索引
        // 当 rehash 不在进行时，值为 -1
        int rehashidx; /* rehashing not in progress if rehashidx == -1 */

    } dict;

``type`` 属性和 ``privdata`` 属性是针对不同类型的键值对，
为创建多态字典而设置的：

- ``type`` 属性是一个指向 ``dictType`` 结构的指针，
  每个 ``dictType`` 结构保存了一簇用于操作特定类型键值对的函数，
  Redis 会为用途不同的字典设置不同的类型特定函数。

- 而 ``privdata`` 属性则保存了需要传给那些类型特定函数的可选参数。

::

    typedef struct dictType {

        // 计算哈希值的函数
        unsigned int (*hashFunction)(const void *key);

        // 复制键的函数
        void *(*keyDup)(void *privdata, const void *key);

        // 复制值的函数
        void *(*valDup)(void *privdata, const void *obj);

        // 对比键的函数
        int (*keyCompare)(void *privdata, const void *key1, const void *key2);

        // 销毁键的函数
        void (*keyDestructor)(void *privdata, void *key);
        
        // 销毁值的函数
        void (*valDestructor)(void *privdata, void *obj);

    } dictType;

``ht`` 属性是一个包含两个项的数组，
数组中的每个项都是一个 ``dictht`` 哈希表，
一般情况下，
字典只使用 ``ht[0]`` 哈希表，
``ht[1]`` 哈希表只会在对 ``ht[0]`` 哈希表进行 rehash 时使用。

除了 ``ht[1]`` 之外，
另一个和 rehash 有关的属性就是 ``rehashidx`` ：
它记录了 rehash 目前的进度，
如果目前没有在进行 rehash ，
那么它的值为 ``-1`` 。

图 4-3 展示了一个普通状态下（没有进行 rehash）的字典：

.. graphviz::

    digraph {

        label = "\n 图 4-3    普通状态下的字典";

        rankdir = LR;

        //

        node [shape = record];

        dict [label = " <head> dict | type | privdata | <ht> ht | rehashidx \n -1 "];

        dictht0 [label = " <head> dictht | <table> table | <size> size \n 4 | <sizemask> sizemask \n 3 | <used> used \n 2"];

        dictht1 [label = " <head> dictht | <table> table | <size> size \n 0 | <sizemask> sizemask \n 0 | <used> used \n 0"];

        table0 [label = " <head> dictEntry*[4] | <0> 0 | <1> 1 | <2> 2 | <3> 3 "];
        table1 [label = "NULL", shape = plaintext];

        dictEntry0 [label = " <head> dictEntry | { k0 | v0 }"];
        dictEntry1 [label = " <head> dictEntry | { k1 | v1 }"];

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

        table0:0 -> null0;
        table0:1 -> dictEntry0:head -> null1;
        table0:2 -> null2;
        table0:3 -> dictEntry1:head -> null3;
    }
