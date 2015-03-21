整数集合 API
---------------------------

表 6-1 列出了整数集合的操作 API 。

---------------------------------------------------------------------------------------------------------------------

表 6-1    整数集合 API

+-------------------+-----------------------------------+-----------------------------------------------------------+
| 函数              | 作用                              | 时间复杂度                                                |
+===================+===================================+===========================================================+
| ``intsetNew``     | 创建一个新的整数集合。            | :math:`O(1)`                                              |
+-------------------+-----------------------------------+-----------------------------------------------------------+
| ``intsetAdd``     | 将给定元素添加到整数集合里面。    | :math:`O(N)`                                              |
+-------------------+-----------------------------------+-----------------------------------------------------------+
| ``intsetRemove``  | 从整数集合中移除给定元素。        | :math:`O(N)`                                              |
+-------------------+-----------------------------------+-----------------------------------------------------------+
| ``intsetFind``    | 检查给定值是否存在于集合。        | 因为底层数组有序，查找可以通过二分查找法来进行，          |
|                   |                                   | 所以复杂度为 :math:`O(\log N)` 。                         |
+-------------------+-----------------------------------+-----------------------------------------------------------+
| ``intsetRandom``  | 从整数集合中随机返回一个元素。    | :math:`O(1)`                                              |
+-------------------+-----------------------------------+-----------------------------------------------------------+
| ``intsetGet``     | 取出底层数组在给定索引上的元素。  | :math:`O(1)`                                              |
+-------------------+-----------------------------------+-----------------------------------------------------------+
| ``intsetLen``     | 返回整数集合包含的元素个数。      | :math:`O(1)`                                              |
+-------------------+-----------------------------------+-----------------------------------------------------------+
| ``intsetBlobLen`` | 返回整数集合占用的内存字节数。    | :math:`O(1)`                                              |
+-------------------+-----------------------------------+-----------------------------------------------------------+

---------------------------------------------------------------------------------------------------------------------

..
    创建新的整数集合
    ^^^^^^^^^^^^^^^^^^^^^^^^^

    调用 ``intsetNew`` 函数可以创建一个新的空白整数集合，
    空白整数集合的状态如图  IMAGE_INTSET_NEW 所示。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_INTSET_NEW    新常见的空白整数集合";

            rankdir = LR;

            node [shape = record];

            intset [label = " intset | encoding \n INTSET_ENC_INT16 | length \n 0 | <contents> contents "];

            null [label = "NULL", shape = plaintext];

            intset:contents -> null;

        }


    添加元素到整数集合
    ^^^^^^^^^^^^^^^^^^^^^^

    图 IMAGE_ADD_12 、图 IMAGE_ADD_5 、图 IMAGE_ADD_10 分别展示了调用 ``intsetAdd`` 三次，
    先后将 ``12`` 、 ``5`` 、 ``10`` 三个值添加到一个新建整数集合的过程。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_ADD_12    添加 12 之后的整数集合";

            rankdir = LR;

            node [shape = record];

            intset [label = " intset | encoding \n INTSET_ENC_INT16 | length \n 1 | <contents> contents "];

            contents [label = " { 12 } "];

            intset:contents -> contents;

        }

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_ADD_5    添加 5 之后的整数集合";

            rankdir = LR;

            node [shape = record];

            intset [label = " intset | encoding \n INTSET_ENC_INT16 | length \n 2 | <contents> contents "];

            contents [label = " { 5 | 12 } "];

            intset:contents -> contents;

        }

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_ADD_10    添加 10 之后的整数集合";

            rankdir = LR;

            node [shape = record];

            intset [label = " intset | encoding \n INTSET_ENC_INT16 | length \n 3 | <contents> contents "];

            contents [label = " { 5 | 10 | 12 } "];

            intset:contents -> contents;

        }

    因为 ``contents`` 数组以有序的方式保存整数值，
    所以尽管我们以 ``12`` 、 ``5`` 、 ``10`` 的顺序添加元素，
    但每次添加操作完成之后，
    数组中的元素总是有序的。

    另外，
    因为新添加的三个值都是 ``int16_t`` 类型，
    所以这三次添加操作都不会引起升级操作，
    整数集合 ``encoding`` 属性的值也仍然是 ``INTSET_ENC_INT16`` ，
    不过 ``length`` 属性的值从之前的 ``0`` 变为了 ``3`` 。

    如果我们向整数集合添加一个类型为 ``int32_t`` 的值 ``65535`` ，
    那么整数集合就会将底层数组从 ``int16_t`` 类型升级为 ``int32_t`` 类型，
    升级并添加元素 ``65535`` 之后的整数集合如图 IMAGE_UPGRADE_INT32 所示。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_UPGRADE_INT32    将底层数组升级为 int32_t 类型";

            rankdir = LR;

            node [shape = record];

            intset [label = " intset | encoding \n INTSET_ENC_INT32 | length \n 4 | <contents> contents "];

            contents [label = " { 5 | 10 | 12 | 65535 } "];

            intset:contents -> contents;

        }

    添加 ``65535`` 之后，
    整数集合 ``encoding`` 属性的值变成了 ``INTSET_ENC_INT32`` ，
    长度变为了 ``4`` 。

    接下来，
    如果我们向整数集合添加类型为 ``int64_t`` 的值 ``4294967295`` ，
    那么将再次引起升级操作，
    原来的 ``int32_t`` 类型的数组会被升级为 ``int64_t`` 类型，
    如图 IMAGE_UPGRADE_INT64 所示。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_UPGRADE_INT64    将底层数组升级为 int64_t 类型";

            rankdir = LR;

            node [shape = record];

            intset [label = " intset | encoding \n INTSET_ENC_INT64 | length \n 5 | <contents> contents "];

            contents [label = " { 5 | 10 | 12 | 65535 | 4294967295 } "];

            intset:contents -> contents;

        }


    在整数集合中查找元素
    ^^^^^^^^^^^^^^^^^^^^^^^^^^

    ``intsetSearch`` 函数查找给定值，
    并报告给定值在 ``contents`` 数组中的索引。

    比如说，
    如果在图 IMAGE_UPGRADE_INT64 所示的整数集合中查找值 ``4294967295`` ，
    那么函数将报告该值的索引为 ``4`` ；
    如果查找值 ``12`` ，
    那么函数将报告该值的索引为 ``2`` ；
    诸如此类。

    因为整数集合的元素在 ``contents`` 数组中是以有序的方式保存的，
    所以 ``intsetSearch`` 函数使用了对数复杂度的二分查找算法来查找指定元素。


    从整数集合中删除元素
    ^^^^^^^^^^^^^^^^^^^^^^^^^^

    最后，
    ``intsetRemove`` 函数用于删除整数集合中的某个元素。

    比如说，
    如果我们对图 IMAGE_UPGRADE_INT64 所示的整数集合删除元素 ``65535`` ，
    那么整数集合将变成图 IMAGE_AFTER_DELETE 所示的样子。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_AFTER_DELETE    删除 65535 之后的整数集合";

            rankdir = LR;

            node [shape = record];

            intset [label = " intset | encoding \n INTSET_ENC_INT64 | length \n 4 | <contents> contents "];

            contents [label = " { 5 | 10 | 12 | 4294967295 } "];

            intset:contents -> contents;

        }
