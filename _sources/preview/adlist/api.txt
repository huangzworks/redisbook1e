链表和链表节点的 API
^^^^^^^^^^^^^^^^^^^^^^^^^^

表 3-1 列出了所有用于操作链表和链表节点的 API 。

----

表 3-1    链表和链表节点 API

+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| 函数                      | 作用                                                      | 时间复杂度                                    |
+===========================+===========================================================+===============================================+
| ``listSetDupMethod``      | 将给定的函数设置为链表的节点值复制函数。                  | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listGetDupMethod``      | 返回链表当前正在使用的节点值复制函数。                    | 复制函数可以通过链表的 ``dup`` 属性直接获得， |
|                           |                                                           | :math:`O(1)`                                  |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listSetFreeMethod``     | 将给定的函数设置为链表的节点值释放函数。                  | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listGetFree``           | 返回链表当前正在使用的节点值释放函数。                    | 释放函数可以通过链表的 ``free`` 属性直接获得，|
|                           |                                                           | :math:`O(1)`                                  |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listSetMatchMethod``    | 将给定的函数设置为链表的节点值对比函数。                  | :math:`O(1)`                                  |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listGetMatchMethod``    | 返回链表当前正在使用的节点值对比函数。                    | 对比函数可以通过链表的 ``match``              |
|                           |                                                           | 属性直接获得，                                |
|                           |                                                           | :math:`O(1)`                                  |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listLength``            | 返回链表的长度（包含了多少个节点）。                      | 链表长度可以通过链表的  ``len`` 属性直接获得，|
|                           |                                                           | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listFirst``             | 返回链表的表头节点。                                      | 表头节点可以通过链表的 ``head`` 属性直接获得，|
|                           |                                                           | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listLast``              | 返回链表的表尾节点。                                      | 表尾节点可以通过链表的 ``tail`` 属性直接获得，|
|                           |                                                           | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listPrevNode``          | 返回给定节点的前置节点。                                  | 前置节点可以通过节点的 ``prev`` 属性直接获得，|
|                           |                                                           | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listNextNode``          | 返回给定节点的后置节点。                                  | 后置节点可以通过节点的 ``next`` 属性直接获得，|
|                           |                                                           | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listNodeValue``         | 返回给定节点目前正在保存的值。                            | 节点值可以通过节点的 ``value`` 属性直接获得， |
|                           |                                                           | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listCreate``            | 创建一个不包含任何节点的新链表。                          | :math:`O(1)`                                  |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listAddNodeHead``       | 将一个包含给定值的新节点添加到给定链表的表头。            | :math:`O(1)`                                  |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listAddNodeTail``       | 将一个包含给定值的新节点添加到给定链表的表尾。            | :math:`O(1)`                                  |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listInsertNode``        | 将一个包含给定值的新节点添加到给定节点的之前或者之后。    | :math:`O(1)`                                  |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listSearchKey``         | 查找并返回链表中包含给定值的节点。                        | :math:`O(N)` ， ``N`` 为链表长度。            |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listIndex``             | 返回链表在给定索引上的节点。                              | :math:`O(N)` ， ``N`` 为链表长度。            |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listDelNode``           | 从链表中删除给定节点。                                    | :math:`O(1)` 。                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listRotate``            | 将链表的表尾节点弹出，然后将被弹出的节点插入到链表的表头，| :math:`O(1)`                                  |
|                           | 成为新的表头节点。                                        |                                               |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listDup``               | 复制一个给定链表的副本。                                  | :math:`O(N)` ， ``N`` 为链表长度。            |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+
| ``listRelease``           | 释放给定链表，以及链表中的所有节点。                      | :math:`O(N)` ， ``N`` 为链表长度。            |
+---------------------------+-----------------------------------------------------------+-----------------------------------------------+

----

..
    Redis 链表实现的函数库和其他链表函数库没什么两样，
    大部分操作都是相同的，
    并且使用的算法都可以在书上找到。

    Redis 链表实现的 API 包括：

    - 创建链表、释放链表、复制链表；

    - 返回链表的各项属性；

    - 添加节点到表头、添加节点到表尾、插入节点到链表；

    - 在链表中查找包含给定值的节点；

    - 按索引返回链表中的节点；

    - 删除链表中的节点；

    本节以下内容将对链表实现的所有 API 进行介绍。

    读取宏和设置宏
    """"""""""""""""""

    链表实现定义了一些宏，
    用于读取或者修改链表成员和链表节点成员。

    以下三个宏分别用于读取链表的 ``len`` 、 ``head`` 和 ``tail`` 三个成员的值：

    ::

        // 返回给定链表所包含的节点数量
        #define listLength(l) ((l)->len)

        // 返回给定链表的表头节点
        #define listFirst(l) ((l)->head)

        // 返回给定链表的表尾节点
        #define listLast(l) ((l)->tail)

    对于带有三个节点 ``x`` 、 ``y`` 和 ``z`` 的链表 ``l`` 来说：

    .. graphviz::

        digraph {

            rankdir = LR;

            //

            subgraph cluster_pointers {

                style = invisible;

                //

                node [shape = plaintext];
                edge [style = invis];

                //

                x -> y;
                y -> z;
            }

            //

            subgraph cluster_ll {

                style = invisible;

                node [shape = record];

                //

                list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

                x_node [label = "<head> listNode | value \n ..."];
                y_node [label = "<head> listNode | value \n ..."];
                z_node [label = "<head> listNode | value \n ..."];

                //

                list:head -> x_node;
                list:tail -> z_node;

                x_node -> y_node;
                y_node -> x_node;

                y_node -> z_node;
                z_node -> y_node;
            }

            //

            x -> x_node;
            y -> y_node;
            z -> z_node;
        }

    执行调用 ``listLength(l);`` 将返回数字 ``3`` 。

    执行调用 ``listFirst(l);`` 将返回节点 ``x`` 。

    执行调用 ``listLast(l);`` 将返回节点 ``z`` 。

    以下是链表类型特定函数的读取宏和设置宏，
    它们用于读取或者设置链表的 ``dup`` 、 ``free`` 和 ``match`` 三个成员的值：

    ::

        // 将链表 l 的值复制函数设置为 m
        #define listSetDupMethod(l,m) ((l)->dup = (m))

        // 返回给定链表的值复制函数
        #define listGetDupMethod(l) ((l)->dup)

        // 将链表 l 的值释放函数设置为 m
        #define listSetFreeMethod(l,m) ((l)->free = (m))

        // 返回给定链表的值释放函数
        #define listGetFree(l) ((l)->free)

        // 将链表的对比函数设置为 m
        #define listSetMatchMethod(l,m) ((l)->match = (m))

        // 返回给定链表的值对比函数
        #define listGetMatchMethod(l) ((l)->match)

    如果程序执行了以下代码：

    ::

        list *l = listCreate();

        listSetDupMethod(l, someDupMethod);
        listSetFreeMethod(l, someFreeMethod);
        listSetMatchMethod(l, someMatchMethod);

    那么执行调用 ``listGetDupMethod(l);`` 将返回 ``someDupMethod`` 。

    执行调用 ``listGetFree(l);`` 将返回 ``someFreeMethod`` 。

    执行调用 ``listGetMatchMethod(l);`` 将返回 ``someMatchMethod`` 。

    以下是链表节点的成员读取宏，
    它们分别返回节点的 ``prev`` 、 ``next`` 和 ``value`` 三个成员的值：

    ::

        // 返回给定节点的前置节点
        #define listPrevNode(n) ((n)->prev)

        // 返回给定节点的后置节点
        #define listNextNode(n) ((n)->next)

        // 返回给定节点的值
        #define listNodeValue(n) ((n)->value)

    对于以下三个链表节点 ``x`` 、 ``y`` 和 ``z`` 来说：

    .. graphviz::

        digraph {

            rankdir = LR;

            //

            subgraph cluster_pointers {

                style = invisible;

                //

                node [shape = plaintext];
                edge [style = invis];

                //

                x -> y;
                y -> z;
            }

            //

            subgraph cluster_ll {

                style = invisible;

                node [shape = record];

                //

                x_node [label = "<head> listNode | value \n ..."];
                y_node [label = "<head> listNode | value \n value_of_y"];
                z_node [label = "<head> listNode | value \n ..."];

                //

                x_node -> y_node;
                y_node -> x_node;

                y_node -> z_node;
                z_node -> y_node;
            }

            //

            x -> x_node;
            y -> y_node;
            z -> z_node;
        }

    执行调用 ``listPrevNode(y);`` 将返回节点 ``x`` 。

    执行调用 ``listNextNode(y);`` 将返回节点 ``z`` 。

    执行调用 ``listNodeValue(y);`` 将返回节点 ``y`` 的值 ``value_of_y`` 。

    以上提到的所有宏的复杂度都是 :math:`O(1)` 。

    创建链表
    """""""""""""

    ``listCreate`` 函数用于创建并返回一个新的链表：

    ::

        list *listCreate(void);

    以下是新创建链表的状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 0 | <dup> dup | <free> free | <match> match "];

            //

            node [shape = plaintext];

            head_null [label = "NULL"];
            tail_null [label = "NULL"];
            dup_null [label = "NULL"];
            free_null [label = "NULL"];
            match_null [label = "NULL"];

            //

            list:head -> head_null;
            list:tail -> tail_null;

            list:dup -> dup_null;
            list:free -> free_null;
            list:match -> match_null;
        }

    除了 ``len`` 成员是 ``0`` 之外，
    其他成员的值都为 ``NULL`` 。

    因为调用 ``listCreate`` 需要为一个 ``list`` 结构分配空间，
    所以该函数的复杂度为 :math:`O(1)` 。

    添加节点到表头
    """"""""""""""""""""""""

    ``listAddNodeHead`` 函数将一个值为 ``value`` 的新节点添加到链表表头，
    并将链表的 ``len`` 成员的值增一：

    ::

        list *listAddNodeHead(list *list, void *value);

    对于一个刚刚使用 ``listCreate`` 创建的新链表 ``l`` 来说：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 0 | ... "];

            //

            node [shape = plaintext];

            head_null [label = "NULL"];
            tail_null [label = "NULL"];

            //

            list:head -> head_null;
            list:tail -> tail_null;
        }

    执行 ``listAddNodeHead(l, z);`` 之后，
    链表将被更新为以下状态
    （图中带虚线的箭头表示一个被更新过的指针）：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 1 | ... "];

            z [label = "<head> listNode | value \n z"];

            //

            list:head -> z [style = dashed];
            list:tail -> z [style = dashed];
        }

    注意，
    链表除了将 ``head`` 成员和 ``tail`` 成员指向了包含值 ``x`` 的节点之外，
    还将 ``len`` 属性的值更新为 ``1`` 。

    继续执行 ``listAddNodeHead(l, y);`` ，
    链表将被更新为以下状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 2 | ... "];

            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> y [style = dashed];
            list:tail -> z;

            y -> z [style = dashed];
            z -> y [style = dashed];
        }

    链表的表头节点变更为包含值 ``y`` 的节点，
    并且 ``len`` 属性的值被更新为 ``2`` 。

    以下是执行 ``listAddNodeHead(l, x);`` 之后，
    链表的状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> x [style = dashed];
            list:tail -> z;

            x -> y [style = dashed];
            y -> x [style = dashed];

            y -> z;
            z -> y;
        }

    链表的表头节点变更为包含值 ``x`` 的节点，
    并且 ``len`` 属性的值被更新为 ``3`` 。

    因为链表带有 ``head`` 节点，
    所以程序可以在 :math:`O(1)` 复杂度内找到链表的表头节点，
    并进行添加新表头节点的动作，
    因此，
    ``listAddNodeHead`` 的复杂度为 :math:`O(1)` 。

    添加节点到表尾
    """"""""""""""""""""""""""

    ``listAddNodeTail`` 函数则将一个值为 ``value`` 的新节点添加到链表表尾，
    并将链表的 ``len`` 成员的值增一：

    ::

        list *listAddNodeTail(list *list, void *value);

    对于一个刚刚使用 ``listCreate`` 创建的新链表 ``l`` 来说：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 0 | ... "];

            //

            node [shape = plaintext];

            head_null [label = "NULL"];
            tail_null [label = "NULL"];

            //

            list:head -> head_null;
            list:tail -> tail_null;
        }

    执行 ``listAddNodeTail(l, x);`` 之后，
    链表将被更新为以下状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 1 | ... "];

            x [label = "<head> listNode | value \n x"];

            //

            list:head -> x [style = dashed];
            list:tail -> x [style = dashed];
        }

    注意，
    链表除了将 ``head`` 成员和 ``tail`` 成员指向了包含值 ``x`` 的节点之外，
    还将 ``len`` 属性的值更新为 ``1`` 。

    继续执行 ``listAddNodeTail(l, y);`` ，
    链表将被更新为以下状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 2 | ... "];

            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];

            //

            list:head -> x;
            list:tail -> y [style = dashed];

            x -> y [style = dashed];
            y -> x [style = dashed];
        }

    链表的表尾节点变更为包含值 ``y`` 的节点，
    并且 ``len`` 属性的值被更新为 ``2`` 。

    以下是执行 ``listAddNodeTail(l, z);`` 之后，
    链表的状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> x;
            list:tail -> z [style = dashed];

            x -> y;
            y -> x;

            y -> z [style = dashed];
            z -> y [style = dashed];
        }

    链表的表尾节点变更为包含值 ``z`` 的节点，
    并且 ``len`` 属性的值被更新为 ``3`` 。

    因为链表带有 ``tail`` 节点，
    所以程序可以在 :math:`O(1)` 复杂度内找到链表的表尾节点，
    并进行添加新表尾节点的动作，
    因此，
    ``listAddNodeTail`` 的复杂度为 :math:`O(1)` 。

    插入新节点
    """""""""""""""

    ``listInsertNode`` 函数用于在给定的节点 ``old_node`` 之前或之后插入一个包含值 ``value`` 的新节点，
    并将链表的 ``len`` 成员的值增一：

    ::
        
        list *listInsertNode(list *list, listNode *old_node, void *value, int after);

    如果 ``after`` 参数的值为 ``0`` ，
    那么新节点插入到 ``old_node`` 之前；
    如果 ``after`` 参数的值为 ``1`` ，
    那么新节点插入到 ``old_node`` 之后。

    举个例子，
    对于以下链表 ``l`` ，
    以及节点指针 ``old`` 来说：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            old [shape = plaintext];

            old -> x;

            //

            subgraph cluster_ll {

                style = invisible;

                //

                list [label = "list | <head> head | <tail> tail | <len> len \n 1 | ... "];

                x [label = "<head> listNode | value \n x"];

                //

                list:head -> x;
                list:tail -> x;
            }

        }

    执行调用 ``listInsertNode(l, old, before, 0);`` 之后，
    一个包含值 ``before`` 的新节点将插入到 ``old`` 所指向节点的前面：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            // 

            old [shape = plaintext];

            //

            subgraph cluster_ll {

                style = invisible;

                //

                list [label = "list | <head> head | <tail> tail | <len> len \n 2 | ... "];

                before [label = "<head> listNode | value \n before"];
                x [label = "<head> listNode | value \n x"];

                //

                list:head -> before [style = dashed];
                list:tail -> x;

                x -> before [style = dashed];
                before -> x [style = dashed];
            }

            // 必须放这里，否则链表会以表尾为表头展示
            old -> x;
        }

    如果继续对 ``l`` 执行调用 ``listInsertNode(l, old, after, 1);`` ，
    那么一个包含值 ``after`` 的新节点将插入到 ``old`` 所指向节点的后面：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            // 

            old [shape = plaintext];

            //

            subgraph cluster_ll {

                style = invisible;

                //

                list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

                before [label = "<head> listNode | value \n before"];
                x [label = "<head> listNode | value \n x"];
                after [label = "<head> listNode | value \n after"];

                //

                list:head -> before;
                list:tail -> after [style = dashed];

                x -> before;
                before -> x;

                x -> after [style = dashed];
                after -> x [style = dashed];
            }

            // 必须放这里，否则链表图形会乱
            old -> x;
        }

    在插入一个新节点时，
    ``listInsertNode`` 函数的所有工作，
    就是修改新节点、原有节点和原有节点的毗邻节点之间的指针，
    因此，
    这个函数的复杂度为 :math:`O(1)` 。

    查找链表
    """""""""""""""""

    ``listSearchKey`` 函数接受一个指针 ``key`` ，
    并根据 ``key`` 提供的信息，
    在链表 ``list`` 中查找和 ``key`` 相匹配的节点：

    ::

        listNode *listSearchKey(list *list, void *key);

    如果在链表中找到了匹配节点，
    那么返回指向该节点的指针；
    如果没找到的话，
    返回 ``NULL`` 。

    匹配需要用到链表的 ``match`` 函数，
    如果 ``match`` 函数没有设置（值为 ``NULL`` ），
    那么 ``listSearchKey`` 直接使用 ``==`` 操作来对比 ``key`` 和节点的值 ——
    以下是 ``listSearchKey`` 函数的代码片段，
    它说明了两种不同的对比方法：

    ::

        // list 为链表
        // node 为当前被迭代到的链表节点

        if (list->match) {
                
            // 设置了 match 函数
            // 使用 match 函数来进行匹配

            if (list->match(node->value, key)) {
                // ...
            }
        } else {

            // 没有设置 match 函数
            // 直接用 == 操作来进行匹配

            if (key == node->value) {
                // ...
            }
        }

    假设链表 ``l`` 带有三个节点，
    它们分别保存了值 ``x`` 、 ``y`` 和 ``z`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> x;
            list:tail -> z;

            x -> y;
            y -> x;

            y -> z;
            z -> y;
        }

    那么执行调用 ``listSearchKey(l, z);`` 时，
    程序将执行以下步骤：

    1. 定位到链表的第一个节点，
       对比这个节点的值 ``x`` 和查找值 ``z`` ，
       结果是不匹配；

    2. 定位到链表的第二个节点，
       对比这个节点的值 ``y`` 和查找值 ``z`` ，
       结果仍然是不匹配；

    3. 定位到链表的第三个节点，
       对比这个节点的值 ``z`` 和查找值 ``z`` ，
       结果是匹配，
       返回指向链表第三个节点的指针。

    下图用虚线在链表中标示了 ``listSearchKey`` 函数的查找轨迹，
    虚线上的数字分别对应上面的三个步骤：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            ret [label = "返回", shape = plaintext];

            //

            subgraph cluster_nodes {

                style = invisible;

                //

                x [label = "<head> listNode | value \n x"];
                y [label = "<head> listNode | value \n y"];
                z [label = "<head> listNode | value \n z"];

                //

                x -> y [dir = back];
                y -> x [dir = back, style = dashed, label = "2"];

                y -> z [dir = back];
                z -> y [dir = back, style = dashed, label = "3"];
            }

            //

            list:head -> x [style = dashed, label = "1"];
            list:tail -> z;

            // 

            listSearchKey [label = "list\nSearch\nKey", shape = plaintext];

            listSearchKey -> list:head [style = dashed];

            //

            ret -> z [style = dashed];
        }

    因为查找是从表头开始的，
    而链表允许不同的节点包含同样的值，
    所以，
    如果链表中有多个和给定 ``key`` 相匹配的节点，
    那么 ``listSearchKey`` 返回第一个匹配节点。

    举个例子，
    假设链表 ``l`` 带有三个节点，
    其中最后两个节点保存了同样的值 ``z`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            x [label = "<head> listNode | value \n x"];
            z1 [label = "<head> listNode | value \n z"];
            z2 [label = "<head> listNode | value \n z"];

            //

            list:head -> x;
            list:tail -> z2;

            x -> z1;
            z1 -> x;

            z1 -> z2;
            z2 -> z1;
        }

    那么在执行 ``listSearchKey(l, z);`` 的时候，
    函数将返回指向第二个节点的指针，
    因为这个节点是程序第一个发现值可以和 ``z`` 相匹配的节点：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            ret [label = "返回", shape = plaintext];

            subgraph cluster_nodes {

                style = invisible;

                //

                x [label = "<head> listNode | value \n x"];
                z1 [label = "<head> listNode | value \n z"];
                z2 [label = "<head> listNode | value \n z"];

                //

                x -> z1 [dir = back];
                z1 -> x [dir = back, style = dashed, label = "2"];

                z1 -> z2;
                z2 -> z1;
            }

            //

            list:head -> x [style = dashed, label = "1"];
            list:tail -> z2;

            // 

            listSearchKey [label = "list\nSearch\nKey", shape = plaintext];

            listSearchKey -> list:head [style = dashed];

            // 
            
            ret -> z1 [style = dashed];
        }

    上图用带数字的虚线标示了 ``listSearchKey`` 的查找轨迹。

    在最坏情况下，
    ``listSearchKey`` 为了查找和给定值相匹配的节点，
    需要遍历链表的所有节点，
    因此，
    ``listSearchKey`` 的复杂度为 :math:`O(N)` 。

    根据索引返回节点
    """""""""""""""""""""""

    ``listIndex`` 函数用于返回链表在指定索引上的节点：

    ::

        listNode *listIndex(list *list, long index);

    索引参数 ``index`` 可以是正数，
    也可以是负数：

    - 正数索引用 ``0`` 表示表头节点，
      ``1`` 表示链表第二个节点，
      以此类推；

    - 负数索引用 ``-1`` 表示表尾节点，
      ``-2`` 表示链表倒数第二个节点，
      以此类推；

    - 当给定索引大于等于链表长度，
      也即是 ``index`` 大于等于 ``list.len`` 的时候，
      返回 ``NULL`` 。

    在下图所示的链表 ``l`` 中，
    带虚线的箭头分别标示了各个节点的正数和负数索引值：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            //

            list:head -> x;
            list:tail -> z;

            //

            subgraph cluster_nodes {

                style = invisible;

                //

                x [label = "<head> listNode | value \n x"];
                y [label = "<head> listNode | value \n y"];
                z [label = "<head> listNode | value \n z"];

                //

                x -> y;
                y -> x;

                y -> z;
                z -> y;
            }

            //

            subgraph cluster_pointers {

                style = invisible;

                edge [style = dashed];

                //

                zero [label = "0 \n -3", shape = plaintext];
                one [label = "1 \n -2", shape = plaintext];
                two [label = "2 \n -1" , shape = plaintext];

                //

                zero -> one -> two [style = invis];

                zero -> x;
                one -> y;
                two -> z;
            }
        }

    以下是一些对 ``l`` 执行 ``listIndex`` 函数的例子：

    - 当执行调用 ``listIndex(l, 0);`` 或者 ``listIndex(l, -3);`` 时，
      函数返回包含值 ``x`` 的节点；

    - 当执行调用 ``listIndex(l, 1);`` 或者 ``listIndex(l, -2);`` 时，
      函数返回包含值 ``y`` 的节点；

    - 当执行调用 ``listIndex(l, 2);`` 或者 ``listIndex(l, -1);`` 时，
      函数返回包含值 ``z`` 的节点。

    对于一个长度为 :math:`N` 的链表，
    ``listIndex`` 最多需要遍历 :math:`N` 个节点，
    才能找到给定索引所指定的节点，
    因此，
    ``listIndex`` 的复杂度为 :math:`O(N)` 。

    删除节点
    """"""""""""""""

    ``listDelNode`` 函数用于删除链表 ``list`` 中的节点 ``node`` ，
    并将链表的长度减一：

    ::

        void listDelNode(list *list, listNode *node);

    举个例子，
    对于以下链表 ``l`` 和节点指针 ``node`` 来说：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            n [label = "node", shape = plaintext];

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            //

            subgraph cluster_nodes {

                style = invisible;

                x [label = "<head> listNode | value \n x"];
                y [label = "<head> listNode | value \n y"];
                z [label = "<head> listNode | value \n z"];

                x -> y;
                y -> x;

                y -> z;
                z -> y;

            }

            //

            list:head -> x;
            list:tail -> z;

            // 必须放这里，否则链表图形会乱
            n -> y;
        }

    执行调用 ``listDelNode(l, node);`` 之后，
    链表将被更新为以下状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 2 | ... "];

            x [label = "<head> listNode | value \n x"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> x;
            list:tail -> z;

            x -> z [style = dashed];
            z -> x [style = dashed];
        }

    注意，
    不仅节点 ``node`` 所指向的节点从链表中被删除了，
    链表的长度也从原来的 ``3`` 变成了 ``2`` 。

    有了指向删除目标节点的指针 ``node`` ，
    所有删除工作都可以通过调整 ``node`` 的毗邻节点的指针来完成，
    并且对链表的 ``head`` 、 ``tail`` 或者 ``len`` 的修改也可以通过简单的赋值来完成，
    因此，
    ``listDelNode`` 的复杂度为 :math:`O(1)` 。

    回转链表
    """"""""""""""""""

    ``listRotate`` 函数用于对链表进行回转操作 ——
    每次调用 ``listRotate`` ，
    函数都会将链表的表尾节点弹出，
    然后将被弹出的节点插入到链表的表头处，
    成为新的表头节点：

    ::

        void listRotate(list *list);

    假设链表 ``l`` 带有三个节点，
    分别保存了值 ``x`` 、 ``y`` 和 ``z`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> x;
            list:tail -> z;

            x -> y;
            y -> x;

            y -> z;
            z -> y;
        }

    执行 ``listRotate(l);`` 之后，
    包含值 ``z`` 的节点将成为链表的新表头节点：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            z [label = "<head> listNode | value \n z"];
            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];

            //

            list:head -> z [style = dashed];
            list:tail -> y;

            z -> x [style = dashed];
            x -> z [style = dashed];

            x -> y;
            y -> x;
        }

    再次执行 ``listRotate(l);`` ，
    包含值 ``y`` 的节点将成为链表的新表头节点：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = "list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];
            x [label = "<head> listNode | value \n x"];

            //

            list:head -> y [style = dashed];
            list:tail -> x;

            y -> z [style = dashed];
            z -> y [style = dashed];

            z -> x;
            x -> z;
        }

    因为 ``listRotate`` 函数只需对链表的表头和表尾进行相应的指针操作，
    并且所有指针操作都可以在常数时间内完成，
    因此，
    ``listRotate`` 的复杂度为 :math:`O(1)` 。

    复制链表
    """"""""""""""""

    ``listDup`` 函数用于复制输入的链表 ``orig`` ，
    并返回该链表的副本：

    ::

        list *listDup(list *orig);

    复制要用到链表设置的 ``dup`` 函数：
    如果设置了 ``dup`` 函数，
    那么使用 ``dup`` 函数来复制节点所保存的值；
    如果没有设置 ``dup`` 函数（\ ``list.dup`` 为 ``NULL``\ ），
    那么直接使用 ``=`` 赋值操作来复制节点的值。

    以下代码段来自 ``listDup`` 函数，
    它说明了函数复制节点值的方法：

    ::

        // list 为输入链表
        // copy 为新创建的链表副本
        // node 为当前迭代到的 list 的节点

        if (list->dup) {

            // 如果有设置 dup 函数
            // 那么使用 dup 函数来复制节点的值
            value = list->dup(node->value);

        } else {

            // 否则，直接使用赋值操作
            value = node->value;

        }

    给定一个链表 ``source`` ，
    它的三个节点分别保存了值 ``x`` 、 ``y`` 和 ``z`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = " <list> list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> x;
            list:tail -> z;

            x -> y;
            y -> x;

            y -> z;
            z -> y;

            //

            source [shape = plaintext];
            source -> list:list;
        }

    那么执行代码 ``list *copy = listDup(source);`` ，
    将产生和 ``source`` 同样带有三个节点的链表 ``copy`` ，
    并且 ``copy`` 的三个节点也同样保存了值 ``x`` 、 ``y`` 和 ``z`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = " <list> list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> x;
            list:tail -> z;

            x -> y;
            y -> x;

            y -> z;
            z -> y;

            //

            copy [shape = plaintext];
            copy -> list:list;
        }

    对于一个长度为 :math:`N` 的链表来说，
    要创建这个链表的副本，
    ``listDup`` 必须复制 :math:`N` 个节点，
    因此，
    ``listDup`` 的复杂度为 :math:`O(N)` 。

    释放链表
    """"""""""""""""

    ``listRelease`` 函数用于删除整个链表的所有节点，
    以及链表本身：

    ::

        void listRelease(list *list);

    对于带有三个节点的链表 ``lst`` 来说，
    执行 ``listRelease(lst);`` 时，
    函数将首先删除链表的第一个节点 —— 该节点保存了值 ``x`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = " <list> list | <head> head | <tail> tail | <len> len \n 3 | ... "];

            x [label = "<head> listNode | value \n x"];
            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> x;
            list:tail -> z;

            x -> y;
            y -> x;

            y -> z;
            z -> y;

            // 

            lst [shape = plaintext];

            lst -> list:list;

            delete [label = "删除", shape = plaintext];
            delete -> x [style = dashed];
        }

    接着，
    函数将删除链表的第二个节点 —— 该节点保存了值 ``y`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = " <list> list | <head> head | <tail> tail | <len> len \n 2 | ... "];

            y [label = "<head> listNode | value \n y"];
            z [label = "<head> listNode | value \n z"];

            //

            list:head -> y;
            list:tail -> z;

            y -> z;
            z -> y;

            // 

            lst [shape = plaintext];

            lst -> list:list;

            delete [label = "删除", shape = plaintext];
            delete -> y [style = dashed];
        }

    然后删除链表的第三个节点 —— 该节点包含值 ``z`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = " <list> list | <head> head | <tail> tail | <len> len \n 1 | ... "];

            z [label = "<head> listNode | value \n z"];

            //

            list:head -> z;
            list:tail -> z;

            // 

            lst [shape = plaintext];

            lst -> list:list;

            delete [label = "删除", shape = plaintext];
            delete -> z [style = dashed];
        }

    最后，
    函数将代表链表本身的 ``list`` 结构也删除掉：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            list [label = " <list> list | <head> head | <tail> tail | <len> len \n 0 | ... "];

            null_head [label = "NULL", shape = plaintext];
            null_tail [label = "NULL", shape = plaintext];

            //

            list:head -> null_head;
            list:tail -> null_tail;

            //

            lst [shape = plaintext];

            lst -> list:list;

            delete [label = "删除", shape = plaintext];
            delete -> list [style = dashed];
        }

    至此，
    ``listRelease`` 函数执行结束。

    对于一个长度为 :math:`N` 的链表来说，
    删除操作共需要对 :math:`N` 个链表节点执行，
    因此，
    ``listRelease`` 的复杂度为 :math:`O(N)` 。
