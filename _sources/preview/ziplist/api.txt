压缩列表 API
----------------------

表 7-4 列出了所有用于操作压缩列表的 API 。

--------------------------------------------------------------------------------------------------------------------------

表 7-4    压缩列表 API

+------------------------+----------------------------------------------+-----------------------------------------------+
| 函数                   | 作用                                         | 算法复杂度                                    |
+========================+==============================================+===============================================+
| ``ziplistNew``         | 创建一个新的压缩列表。                       | :math:`O(1)`                                  |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistPush``        | 创建一个包含给定值的新节点，                 | 平均 :math:`O(N)` ，最坏 :math:`O(N^2)` 。    |
|                        | 并将这个新节点添加到压缩列表的表头或者表尾。 |                                               |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistInsert``      | 将包含给定值的新节点插入到给定节点之后。     | 平均 :math:`O(N)` ，最坏 :math:`O(N^2)` 。    |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistIndex``       | 返回压缩列表给定索引上的节点。               | :math:`O(N)`                                  |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistFind``        | 在压缩列表中查找并返回包含了给定值的节点。   | 因为节点的值可能是一个字节数组，              |
|                        |                                              | 所以检查节点值和给定值是否相同的复杂度为      |
|                        |                                              | :math:`O(N)` ，                               |
|                        |                                              | 而查找整个列表的复杂度则为 :math:`O(N^2)` 。  |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistNext``        | 返回给定节点的下一个节点。                   | :math:`O(1)`                                  |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistPrev``        | 返回给定节点的前一个节点。                   | :math:`O(1)`                                  |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistGet``         | 获取给定节点所保存的值。                     | :math:`O(1)`                                  |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistDelete``      | 从压缩列表中删除给定的节点。                 | 平均 :math:`O(N)` ，最坏 :math:`O(N^2)` 。    |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistDeleteRange`` | 删除压缩列表在给定索引上的连续多个节点。     | 平均 :math:`O(N)` ，最坏 :math:`O(N^2)` 。    |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistBlobLen``     | 返回压缩列表目前占用的内存字节数。           | :math:`O(1)`                                  |
+------------------------+----------------------------------------------+-----------------------------------------------+
| ``ziplistLen``         | 返回压缩列表目前包含的节点数量。             | 节点数量小于 ``65535`` 时 :math:`O(1)` ，     |
|                        |                                              | 大于 ``65535`` 时 :math:`O(N)` 。             |
+------------------------+----------------------------------------------+-----------------------------------------------+

--------------------------------------------------------------------------------------------------------------------------

因为 ``ziplistPush`` 、 ``ziplistInsert`` 、 ``ziplistDelete`` 和 ``ziplistDeleteRange`` 四个函数都有可能会引发连锁更新，
所以它们的最坏复杂度都是 :math:`O(N^2)` 。


..
    创建新压缩列表
    ^^^^^^^^^^^^^^^^^^

    调用 ``ziplistNew`` 函数可以创建一个新的压缩列表，
    这个列表如图 IMAGE_NEW 所示。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_NEW    新创建的压缩列表";

            node [shape = record];

            ziplist [label = " zlbytes \n 0xb | zltail \n 0xA | zllen \n 0x0 | zlend \n 0xFF "];

        }

    ``zlbytes`` 属性的值为 ``0xb`` （十进制 ``11``\ ），
    表示这个新的压缩列表总长 ``11`` 字节。

    ``zltail`` 属性的值为 ``0xA`` （十进制 ``10``\ ），
    表示达到表尾节点所需的偏移量为 ``10`` 字节。

    ``zllen`` 属性的值为 ``0x0`` （十进制 ``0``\ ），
    表示这个压缩列表目前还没有包含任何节点。

    ``zlend`` 属性的值为 ``0xFF`` （十进制 ``255``\ ），
    这个值标识了压缩列表的末端。


    将新节点推入到表头或者表尾
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistPush`` 函数用于将一个包含给定值的新节点推入到给定压缩列表的表头或者表尾，
    使得这个新节点成为压缩列表的新表头节点或者新表尾节点。

    当压缩列表为空时，
    ``ziplistPush`` 将新节点推入到表头或者表尾的效果都是一样的。

    比如说，
    如果我们对图 IMAGE_NEW 所示的新压缩列表推入一个新节点 ``new_entry`` ，
    那么无论这个列表被推入到表头还是表尾，
    我们都会得到图 IMAGE_PUSH_WHEN_EMPTY 所示的压缩列表。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_PUSH_WHEN_EMPTY    向新的压缩列表推入一个新节点";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | <new_entry> new_entry | zlend "];

            p [label = "推入", shape = plaintext];

            p -> ziplist:new_entry;

        }

    另一方面，
    对于一个非空压缩列表来说，
    ``ziplistPush`` 将节点推入的方向则会产生不同的效果。

    比如说，
    对于图 IMAGE_NON_EMPTY_LIST 所示的压缩列表来说：

    - 如果我们将新节点 ``new_entry`` 推入到压缩列表的表头，
      那么我们将得到图 IMAGE_PUSH_ON_HEAD 所示的压缩列表。

    - 如果我们将新节点 ``new_entry`` 推入到压缩列表的表尾，
      那么我们将得到图 IMAGE_PUSH_ON_TAIL 所示的压缩列表。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_NON_EMPTY_LIST    一个包含三个节点的压缩列表";

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | entry_x | entry_y | entry_z | zlend "];

        }


    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_PUSH_ON_HEAD    将新节点推入到表头";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | <new_entry> new_entry | entry_x | entry_y | entry_z | zlend "];

            add [label = "推入到表头", shape = plaintext];

            add -> ziplist:new_entry;

        }

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_PUSH_ON_TAIL    将新节点推入到表尾";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | entry_x | entry_y | entry_z | <new_entry> new_entry | zlend "];

            add [label = "推入到表尾", shape = plaintext];

            add -> ziplist:new_entry;

        }


    将新节点插入到给定节点之后
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistInsert`` 函数用于将一个包含给定值的新节点插入到给定节点之后。

    举个例子，
    对于图 IMAGE_BEFORE_INSERT 所示的压缩列表来说，
    如果我们将节点 ``entry_x`` 作为参数调用 ``ziplistInsert`` 函数，
    那么新节点 ``new_entry`` 将被添加到 ``entry_x`` 节点之后，
    如图 IMAGE_AFTER_INSERT 所示。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_BEFORE_INSERT    一个包含三个节点的压缩列表";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | <entry_x> entry_x | entry_y | entry_z | zlend "];

            p [label = "插入到这个节点之后", shape = plaintext];

            p -> ziplist:entry_x;
        }

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_AFTER_INSERT    将 new_entry 插入到 entry_x 之后";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | <entry_x> entry_x | <new_entry> new_entry | entry_y | entry_z | zlend "];

            p [label = "新节点", shape = plaintext];

            p -> ziplist:new_entry;

        }


    .. topic:: 没有 ``ziplistInsertBefore`` 函数

        压缩列表并没有可以将新节点插入到给定节点之前的函数，
        因为对于压缩列表来说，
        ``ziplistInsert`` 函数唯一不能做的事情就是将新节点插入到压缩列表的表头节点之前，
        但这一操作却可以用 ``ziplistPush`` 函数来完成，
        所以即使没有和 ``ziplistInsert`` 函数相应的 ``ziplistInsertBefore`` 函数，
        也不会对压缩列表的功能产生影响，
        最多就是有一些不方便而已。

        举个例子，
        对于图 IMAGE_INSERT_BEFORE 所示的压缩列表来说，
        调用 ``ziplistInsert`` 函数是没办法将新节点插入到表头节点 ``entry_x`` 之前的，
        但使用 ``ziplistPush`` 函数却可以。
        
        .. graphviz::

            digraph {

                label = "\n 图 IMAGE_INSERT_BEFORE    将新节点插入到表头节点之前";

                rankdir = BT;

                node [shape = record];

                ziplist [label = " zlbytes | zltail | zllen | <entry_x> entry_x | entry_y | entry_z | zlend "];

                p [label = "使用 ziplistInsert 没办法将新节点插入到这个节点之前\n但是使用 ziplistPush 却可以", shape = plaintext];

                p -> ziplist:entry_x;

            }


    返回给定索引上的节点
    ^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistIndex`` 函数用于返回压缩列表给定索引上的节点：

    - 如果索引值为正数，
      那么索引就以表头为 ``0`` 开始计算。

    - 如果索引值为负数，
      那么索引就以表尾为 ``-1`` 开始计算。

    图 IMAGE_INDEX 展示了一个包含五个节点的压缩列表，
    以及使用 ``ziplistIndex`` 函数返回各个节点所属的索引值：

    - 如果我们向 ``ziplistIndex`` 输入索引 ``1`` ，
      那么函数将返回节点 ``entry_b`` ；

    - 如果我们向 ``ziplistIndex`` 函数输入索引 ``-2`` ，
      那么函数将返回节点 ``entry_d`` ；

    诸如此类。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_INDEX    压缩列表各个节点的索引值";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | <zllen> zllen | <entry1> entry_a | <entry2> entry_b | <entry3> entry_c | <entry4> entry_d | <entry5> entry_e | zlend "];

            node [shape = plaintext];

            edge [style = invis];

            index [label = "正数索引：\n负数索引："];

            index -> ziplist:zllen;

            entry1 [label = "0 \n -5"];
            entry2 [label = "1 \n -4"];
            entry3 [label = "2 \n -3"];
            entry4 [label = "3 \n -2"];
            entry5 [label = "4 \n -1"];

            entry1 -> ziplist:entry1;
            entry2 -> ziplist:entry2;
            entry3 -> ziplist:entry3;
            entry4 -> ziplist:entry4;
            entry5 -> ziplist:entry5;

        }


    在列表中查找节点
    ^^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistFind`` 函数在压缩列表中查找并返回包含给定值的节点。

    举个例子，
    对于图 IMAGE_FIND 所示的压缩列表来说：

    - 查找字节数组 ``"hello"`` 将返回节点 ``entry_x`` ；

    - 查找字节数组 ``"world"`` 将返回节点 ``entry_y`` ；

    - 查找整数 ``10086`` 将返回节点 ``entry_z`` 。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_FIND    压缩列表";

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | { entry_x | { ... | content \n \"hello\" }} | { entry_y | { ... | content \n \"world\" }} | { entry_z | { ... | content \n 10086 }} | zlend "];

        }



    返回给定节点的前一个节点和后一个节点
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistPrev`` 函数和 ``ziplistNext`` 函数分别用于返回给定节点的前一个节点和后一个节点。

    以图 IMAGE_PREV_NEXT 所示的压缩列表为例：

    - 如果我们对节点 ``entry_b`` 调用 ``ziplistPrev`` 函数，
      那么函数将返回节点 ``entry_a`` ；

    - 如果我们对节点 ``entry_b`` 调用 ``ziplistNext`` 函数，
      那么函数将返回节点 ``entry_c`` 。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_PREV_NEXT    对 entry_b 调用 ziplistPrev 函数和 ziplistNext 函数";

            //rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | <entry_a> entry_a | <entry_b> entry_b | <entry_c> entry_c | zlend "];

            node [shape = plaintext];

            ziplist:entry_b -> ziplist:entry_a [label = "ziplistPrev    "];
            ziplist:entry_b -> ziplist:entry_c [label = "                                    ziplistNext"];

        }

    另外，
    通过多次调用 ``ziplistNext`` 函数或者 ``ziplistPrev`` 函数，
    我们可以从表头向表尾、或者从表尾向表头对压缩列表进行遍历：

    - 图 IMAGE_ITER_FROM_HEAD 展示了使用 ``ziplistNext`` 函数从表头节点 ``entry_a`` 遍历到表尾节点 ``entry_e`` 的整个过程。

    - 图 IMAGE_ITER_FROM_TAIL 展示了使用 ``ziplistPrev`` 函数从表尾节点 ``entry_e`` 遍历到表头节点 ``entry_a`` 的整个过程。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_ITER_FROM_HEAD    从表头向表尾遍历";

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | <entry_a> entry_a | <entry_b> entry_b | <entry_c> entry_c | <entry_d> entry_d | <entry_e> entry_e | zlend "];

            // HACK：如果不用空格的话两个 ziplistPrev label 就会重叠在一起

            ziplist:entry_a -> ziplist:entry_b //[label = "ziplistNext                        "];
            ziplist:entry_b -> ziplist:entry_c //[label = "   ziplistNext"] ;
            ziplist:entry_c -> ziplist:entry_d //[label = "                           ziplistNext"];
            ziplist:entry_d -> ziplist:entry_e [label = "                                                          ziplistNext"];

        }

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_ITER_FROM_TAIL    从表尾向表头遍历";

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | <entry_a> entry_a | <entry_b> entry_b | <entry_c> entry_c | <entry_d> entry_d | <entry_e> entry_e | zlend "];

            edge [dir = back];

            ziplist:entry_a -> ziplist:entry_b [label = "ziplistPrev                        "];//[label = "ziplistNext                        "];
            ziplist:entry_b -> ziplist:entry_c //[label = "   ziplistNext"] ;
            ziplist:entry_c -> ziplist:entry_d //[label = "                           ziplistNext"];
            ziplist:entry_d -> ziplist:entry_e //[label = "                                                               ziplistNext"];

        }



    从压缩列表中删除指定节点
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistDelete`` 用于从压缩列表中删除指定的一个节点。

    举个例子，
    对于图 IMAGE_BEFORE_DELETE 所示的压缩列表来说，
    对节点 ``entry_x`` 执行 ``ziplistDelete`` 函数会导致节点 ``entry_x`` 被删除，
    删除操作执行之后的压缩列表如图 IMAGE_AFTER_DELETE 所示。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_BEFORE_DELETE    删除 entry_x 之前的压缩列表";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | <entry_x> entry_x | entry_y | entry_z | zlend "];

            p [label = "调用 ziplistDelete 删除这个节点", shape = plaintext];

            p -> ziplist:entry_x;

        }

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_AFTER_DELETE    删除 entry_x 之后的压缩列表";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | zllen | entry_y | entry_z | zlend "];

        }


    删除给定索引上的连续多个节点
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistDeleteRange`` 函数用于删除指定索引 ``i`` 之后的连续 ``n`` 个节点。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_BEFORE_RANGE_DELETE    压缩列表各个节点的索引值";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | <zllen> zllen | <entry1> entry_a | <entry2> entry_b | <entry3> entry_c | <entry4> entry_d | <entry5> entry_e | zlend "];

            node [shape = plaintext];

            edge [style = invis];

            index [label = "正数索引：\n负数索引："];

            index -> ziplist:zllen;

            entry1 [label = "0 \n -5"];
            entry2 [label = "1 \n -4"];
            entry3 [label = "2 \n -3"];
            entry4 [label = "3 \n -2"];
            entry5 [label = "4 \n -1"];

            entry1 -> ziplist:entry1;
            entry2 -> ziplist:entry2;
            entry3 -> ziplist:entry3;
            entry4 -> ziplist:entry4;
            entry5 -> ziplist:entry5;

        }


    举个例子，
    对于图 IMAGE_BEFORE_RANGE_DELETE 所示的压缩列表来说：

    - 如果我们以 ``i`` 为 ``2`` ，
      ``n`` 为 ``3`` 调用 ``ziplistDeleteRange`` 函数，
      那么程序将删除压缩列表从索引 ``2`` 开始算起的三个节点（\ ``entry_c`` 、 ``entry_d`` 、 ``entry_e`` ），
      删除操作执行之后的压缩列表如图 IMAGE_DELETE_RANGE_1 所示。

    - 如果我们以 ``i`` 为 ``0`` ，
      ``n`` 为 ``2`` 调用 ``ziplistDeleteRange`` 函数，
      那么程序将删除压缩列表从索引 ``0`` 开始算起的两个节点（\ ``entry_a`` 和 ``entry_b``\ ），
      删除操作执行之后的压缩列表如图 IMAGE_DELETE_RANGE_2 所示。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_DELETE_RANGE_1    ziplistDeleteRange 示例一";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | <zllen> zllen | <entry1> entry_a | <entry2> entry_b | zlend "];

            node [shape = plaintext];

            edge [style = invis];

            index [label = "正数索引：\n负数索引："];

            index -> ziplist:zllen;

            entry1 [label = "0 \n -2"];
            entry2 [label = "1 \n -1"];

            entry1 -> ziplist:entry1;
            entry2 -> ziplist:entry2;

        }

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_DELETE_RANGE_2    ziplistDeleteRange 示例二";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " zlbytes | zltail | <zllen> zllen | <entry3> entry_c | <entry4> entry_d | <entry5> entry_e | zlend "];

            node [shape = plaintext];

            edge [style = invis];

            index [label = "正数索引：\n负数索引："];

            index -> ziplist:zllen;

            entry3 [label = "0 \n -3"];
            entry4 [label = "1 \n -2"];
            entry5 [label = "2 \n -1"];

            entry3 -> ziplist:entry3;
            entry4 -> ziplist:entry4;
            entry5 -> ziplist:entry5;

        }

    和 ``ziplistIndex`` 一样，
    ``ziplistDeleteRange`` 函数的索引也可以是负数值。

    比如说：
    如果我们以 ``i`` 为 ``-3`` ，
    ``n`` 为 ``3`` 调用 ``ziplistDeleteRange`` 函数，
    那么删除操作执行之后的压缩列表如图 IMAGE_DELETE_RANGE_1 所示的一样。

    获取给定节点的值
    ^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistGet`` 函数用于取出给定节点所保存的值。

    举个例子，
    对图 IMAGE_GET 所示的节点调用 ``ziplistGet`` 可以取出字节数组 ``"hello"`` ，
    而对图 IMAGE_ANOTHER_GET 所示的节点调用 ``ziplistGet`` 则可以取出整数 ``10086`` 。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_GET    保存字节数组的节点";

            node [shape = record];

            entry [label = " previous_entry_length \n ... | encoding \n ... | content \n \"hello\" "];

        }

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_ANOTHER_GET    保存整数的节点";

            node [shape = record];

            entry [label = " previous_entry_length \n ... | encoding \n ... | content \n 10086 "];

        }

    调用 ``ziplistGet`` 函数需要同时传入一个字节数组指针和一个整数指针，
    这两个指针都要被初始化为 ``NULL`` ：

    - 如果节点保存的是字节数组，
      那么 ``ziplistGet`` 将字节数组指针指向这个数组；

    - 如果节点保存的是整数，
      那么 ``ziplistGet`` 将整数指针指向这个整数；

    在调用完 ``ziplistGet`` 函数之后，
    程序通过检查哪个指针非空来确定节点保存了什么值：

    - 如果字节数组指针非空，
      那么节点保存的就是字节数组；

    - 如果整数指针非空，
      那么节点保存的就是整数；

    基于以上原理，
    ``ziplistGet`` 函数总是可以取出节点保存的值，
    不论这个值是字节数组还是整数。

    返回压缩列表的字节长度以及节点数量
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    ``ziplistBlobLen`` 函数返回给定压缩列表占用的内存字节数量，
    该函数读取并返回压缩列表 ``zlbytes`` 属性的值；
    而 ``ziplistLen`` 函数则返回给定压缩列表包含的节点数量，
    该函数读取并返回压缩列表 ``zllen`` 属性的值；
    如图 IMAGE_READER 所示。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_READER    ziplistBlobLen 函数和 ziplistLen 函数";

            rankdir = BT;

            node [shape = record];

            ziplist [label = " <zlbytes> zlbytes | zltail | <zllen> zllen | ... | zlend "];

            node [shape = plaintext];

            ziplistBlobLen -> ziplist:zlbytes [label = "读取并返回\n内存字节数"];

            ziplistLen -> ziplist:zllen [label = "读取并返回\n节点数量"];

        }

    比如说，
    对于图 IMAGE_LIST_EXMPALE 所示的压缩列表来说，
    ``ziplistLen`` 将返回 ``5`` （十六进制 ``0x5``\ ），
    表示压缩列表包含了五个节点；
    而 ``ziplistBlobLen`` 函数将返回 ``210`` （十六进制 ``0xd2``\ ），
    表示压缩列表总长度为 ``210`` 字节。

    .. graphviz::

        digraph {

            label = "\n 图 IMAGE_LIST_EXMPALE    一个包含五个节点的压缩列表";

            node [shape = record];

            ziplist [label = " zlbytes \n 0xd2 | zltail \n 0xb3 | zllen \n 0x5 | entry1 | entry2 | entry3 | entry4 | entry5 | zlend \n 0xFF "];

        }
