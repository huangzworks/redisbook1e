GETBIT 命令的实现
--------------------

:ref:`GETBIT` 命令用于返回位数组 ``bitarray`` 在 ``offset`` 偏移量上的二进制位的值：

::

    GETBIT <bitarray> <offset>

:ref:`GETBIT` 命令的执行过程如下：

1. 计算 :math:`byte = \lfloor offset \div 8 \rfloor` ，
   ``byte`` 值记录了 ``offset`` 偏移量指定的二进制位保存在位数组的哪个字节。

2. 计算 :math:`bit = (offset \bmod 8) + 1` ，
   ``bit`` 值记录了 ``offset`` 偏移量指定的二进制位是 ``byte`` 字节的第几个二进制位。

3. 根据 ``byte`` 值和 ``bit`` 值，
   在位数组 ``bitarray`` 中定位 ``offset`` 偏移量指定的二进制位，
   并返回这个位的值。

举个例子，
对于图 IMAGE_BIT_EXAMPLE 所示的位数组来说，
命令：

::

    GETBIT <bitarray> 3

将执行以下操作：

1. :math:`\lfloor 3 \div 8 \rfloor` 的值为 ``0`` 。

2. :math:`(3 \bmod 8) + 1` 的值为 ``4`` 。

3. 定位到 ``buf[0]`` 字节上面，
   然后取出该字节上的第 ``4`` 个二进制位（从左向右数）的值。

4. 向客户端返回二进制位的值 ``1`` 。

命令的执行过程如图 IMAGE_SEARCH_EXAMPLE 所示。

.. graphviz::

    digraph {

        label = "\n 图 IMAGE_SEARCH_EXAMPLE    查找并返回 offset 为 3 的二进制位的过程";

        //

        rankdir = LR;

        point_to_buf0 [label = "1) 定位到 buf[0] 字节", shape = plaintext];

        point_to_idx3 [label = "2) 返回第 4 个二进制位的值", shape = plaintext];

        buf [label = " { <buf0> buf[0] | 1 | 0 | 1 | <idx3> 1 | 0 | 0 | 1 | 0 } | { buf[1] （空字符） } ", shape = record];

        //

        edge [style = dashed];
        
        point_to_buf0 -> buf:buf0;
        point_to_idx3 -> buf:idx3;

    }

再举一个例子，
对于图 IMAGE_ANOTHER_BIT_EXAMPLE 所示的位数组来说，
命令：

::

    GETBIT <bitarray> 10

将执行以下操作：

1. :math:`\lfloor 10 \div 8 \rfloor` 的值为 ``1`` 。

2. :math:`(10 \bmod 8) + 1` 的值为 ``3`` 。

3. 定位到 ``buf[1]`` 字节上面，
   然后取出该字节上的第 ``3`` 个二进制位的值。

4. 向客户端返回二进制位的值 ``0`` 。

命令的执行过程如图 IMAGE_ANOTHER_SEARCH_EXAMPLE 所示。

.. graphviz::

    digraph {

        label = "\n 图 IMAGE_ANOTHER_SEARCH_EXAMPLE    查找并返回 offset 为 10 的二进制位的过程";

        rankdir = LR;

        //

        node [shape = record];

        buf [label = " { buf[0] | 1 | 0 | 1 | 0 | 0 | 1 | 0 | 1 } | { <buf1> buf[1] | 1 | 1 | <bit> 0 | 0 | 0 | 0 | 1 | 1 } | { buf[2] | 0 | 0 | 0 | 0 | 1 | 1 | 1 | 1 } | { buf[3] | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 } "];

        node [shape = plaintext];

        point_to_buf [label = "1） 定位到 buf[1] 字节"];
        point_to_bit [label = "2） 返回第 3 个二进制位的值"];

        //

        edge [style = dashed];
        point_to_buf -> buf:buf1;
        point_to_bit -> buf:bit;

    }

因为 :ref:`GETBIT` 命令执行的所有操作都可以在常数时间内完成，
所以该命令的算法复杂度为 :math:`O(1)` 。
