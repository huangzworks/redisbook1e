链表和链表节点的实现
-----------------------------

每个链表节点使用一个 ``adlist.h/listNode`` 结构来表示：

::

    typedef struct listNode {

        // 前置节点
        struct listNode *prev;

        // 后置节点
        struct listNode *next;

        // 节点的值
        void *value;

    } listNode;

多个 ``listNode`` 可以通过 ``prev`` 和 ``next`` 指针组成双端链表，
如图 3-1 所示。

.. graphviz::

    digraph {

        label = "\n 图 3-1    由多个 listNode 组成的双端链表"

        rankdir = LR;

        node [shape = record];

        //

        more_prev [label = "...", shape = plaintext];
        x [label = "<head> listNode | value \n ..."];
        y [label = "<head> listNode | value \n ..."];
        z [label = "<head> listNode | value \n ..."];
        more_next [label = "...", shape = plaintext];

        //

        more_prev -> x [label = "next"];
        x -> more_prev [label = "prev"];
       

        x -> y [label = "next"];
        y -> x [label = "prev"];

        y -> z [label = "next"];
        z -> y [label = "prev"];

        z -> more_next [label = "next"];
        more_next -> z [label = "prev"];
    }

虽然仅仅使用多个 ``listNode`` 结构就可以组成链表，
但使用 ``adlist.h/list`` 来持有链表的话，
操作起来会更方便：

::

    typedef struct list {

        // 表头节点
        listNode *head;

        // 表尾节点
        listNode *tail;

        // 链表所包含的节点数量
        unsigned long len;

        // 节点值复制函数
        void *(*dup)(void *ptr);

        // 节点值释放函数
        void (*free)(void *ptr);

        // 节点值对比函数
        int (*match)(void *ptr, void *key);

    } list;

``list`` 结构为链表提供了表头指针 ``head`` 、表尾指针 ``tail`` ，
以及链表长度计数器 ``len`` ，
而 ``dup`` 、 ``free`` 和 ``match`` 成员则是用于实现多态链表所需的类型特定函数：

- ``dup`` 函数用于复制链表节点所保存的值；

- ``free`` 函数用于释放链表节点所保存的值；

- ``match`` 函数则用于对比链表节点所保存的值和另一个输入值是否相等。

图 3-2 是由一个 ``list`` 结构和三个 ``listNode`` 结构组成的链表：

.. graphviz::

    digraph {

        label = "\n 图 3-2    由 list 结构和 listNode 结构组成的链表"

        rankdir = LR;

        node [shape = record];

        //

        list [label = "list | <head> head | <tail> tail | <len> len \n 3 | <dup> dup | <free> free | <match> match ", width = 2.0];

        more_prev [label = "NULL", shape = plaintext];
        x [label = "<head> listNode | value \n ..."];
        y [label = "<head> listNode | value \n ..."];
        z [label = "<head> listNode | value \n ..."];
        more_next [label = "NULL", shape = plaintext];

        dup [label = "...", shape = plaintext];
        free [label = "...", shape = plaintext];
        match [label = "...", shape = plaintext];

        //

        list:head -> x;
        list:tail -> z;

        list:dup -> dup;
        list:free -> free;
        list:match -> match;

        x -> y;
        y -> x;

        y -> z;
        z -> y;

        //

        more_prev -> x [dir = back];
        z -> more_next;

    }

Redis 的链表实现的特性可以总结如下：

- 双端：
  链表节点带有 ``prev`` 和 ``next`` 指针，
  获取某个节点的前置节点和后置节点的复杂度都是 :math:`O(1)` 。

- 无环：
  表头节点的 ``prev`` 指针和表尾节点的 ``next`` 指针都指向 ``NULL`` ，
  对链表的访问以 ``NULL`` 为终点。

- 带表头指针和表尾指针：
  通过 ``list`` 结构的 ``head`` 指针和 ``tail`` 指针，
  程序获取链表的表头节点和表尾节点的复杂度为 :math:`O(1)` 。

- 带链表长度计数器：
  程序使用 ``list`` 结构的 ``len`` 属性来对 ``list`` 持有的链表节点进行计数，
  程序获取链表中节点数量的复杂度为 :math:`O(1)` 。

- 多态：
  链表节点使用 ``void*`` 指针来保存节点值，
  并且可以通过 ``list`` 结构的 ``dup`` 、 ``free`` 、 ``match`` 三个属性为节点值设置类型特定函数，
  所以链表可以用于保存各种不同类型的值。
