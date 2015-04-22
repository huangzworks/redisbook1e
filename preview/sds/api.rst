SDS API
==================

表 2-2 列出了 SDS 的主要操作 API 。

----

表 2-2    SDS 的主要操作 API

+-------------------+---------------------------------------+-------------------------------------------------------+
| 函数              | 作用                                  | 时间复杂度                                            |
+===================+=======================================+=======================================================+
| ``sdsnew``        | 创建一个包含给定 C 字符串的 SDS 。    | :math:`O(N)` ， ``N`` 为给定 C 字符串的长度。         |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdsempty``      | 创建一个不包含任何内容的空 SDS 。     | :math:`O(1)`                                          |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdsfree``       | 释放给定的 SDS 。                     | :math:`O(1)`                                          |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdslen``        | 返回 SDS 的已使用空间字节数。         | 这个值可以通过读取 SDS 的 ``len`` 属性来直接获得，    |
|                   |                                       | 复杂度为 :math:`O(1)` 。                              |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdsavail``      | 返回 SDS 的未使用空间字节数。         | 这个值可以通过读取 SDS 的 ``free`` 属性来直接获得，   |
|                   |                                       | 复杂度为 :math:`O(1)` 。                              |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdsdup``        | 创建一个给定 SDS 的副本（copy）。     | :math:`O(N)` ， ``N`` 为给定 SDS 的长度。             |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdsclear``      | 清空 SDS 保存的字符串内容。           | 因为惰性空间释放策略，复杂度为 :math:`O(1)` 。        |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdscat``        | 将给定 C 字符串拼接到 SDS             | :math:`O(N)` ， ``N`` 为被拼接 C 字符串的长度。       |
|                   | 字符串的末尾。                        |                                                       |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdscatsds``     | 将给定 SDS 字符串拼接到另一个 SDS     | :math:`O(N)` ， ``N`` 为被拼接 SDS 字符串的长度。     |
|                   | 字符串的末尾。                        |                                                       |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdscpy``        | 将给定的 C 字符串复制到 SDS 里面，    | :math:`O(N)` ， ``N`` 为被复制 C 字符串的长度。       |
|                   | 覆盖 SDS 原有的字符串。               |                                                       |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdsgrowzero``   | 用空字符将 SDS 扩展至给定长度。       | :math:`O(N)` ， ``N`` 为扩展新增的字节数。            |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdsrange``      | 保留 SDS 给定区间内的数据，           | :math:`O(N)` ， ``N`` 为被保留数据的字节数。          |
|                   | 不在区间内的数据会被覆盖或清除。      |                                                       |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdstrim``       | 接受一个 SDS 和一个 C 字符串作为参数，| :math:`O(M*N)` ， ``M`` 为 SDS 的长度，               |
|                   | 从 SDS 左右两端分别移除所有在 C       | ``N`` 为给定 C 字符串的长度。                         |
|                   | 字符串中出现过的字符。                |                                                       |
+-------------------+---------------------------------------+-------------------------------------------------------+
| ``sdscmp``        | 对比两个 SDS 字符串是否相同。         | :math:`O(N)` ， ``N`` 为两个 SDS 中较短的那个 SDS     |
|                   |                                       | 的长度。                                              |
+-------------------+---------------------------------------+-------------------------------------------------------+

..
    和其他字符串类型一样，
    SDS 也有一套自己的函数库。

    因为 SDS 并不是一个通用的字符串库，
    它的目标仅仅是满足 Redis 的应用需求，
    所以 SDS 的 API 数量并不多：
    这些 API 主要用于实现一些常用的字符串操作，
    比如创建、删除、拼接、复制，
    等等。

    下面各个小节将分别对 SDS 的各个主要 API 进行介绍。


    创建新 SDS
    ---------------

    函数 ``sdsnew`` 接受一个字符串字面量作为输入，
    创建并返回一个包含该字符串的 SDS ：

    ::

        sdshdr *sdsnew(const char *init);

    新创建的 SDS 不分配任何未使用空间，
    也就是说，
    ``sdshdr.buf`` 成员的值总为 ``0`` 。

    举个例子，
    执行调用：

    ::

        sdsnew("Redis");
        
    函数将返回以下 SDS ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n5 | <buf> buf"];

            buf [label = "{ 'R' | 'e' | 'd' | 'i' | 's' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    另外，
    执行调用：

    ::

        sdsnew("hello world");

    函数将返回以下 SDS ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n11 | <buf> buf"];

            buf [label = "{ 'h' | 'e' | 'l' | 'l' | 'o' | ' ' | 'w' | 'o' | 'r' | 'l' | 'd' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    对于长度为 :math:`N` 的字符串输入，
    程序需要复制 :math:`N` 个字符到 ``sdshdr.buf`` 数组中，
    因此，
    ``sdsnew`` 函数的复杂度为 :math:`O(N)` 。


    创建空白 SDS
    -----------------

    ``sdsempty`` 函数用于创建并返回一个空白 SDS ：

    ::

        sdshdr *sdsempty(void);

    空白 SDS 的 ``free`` 成员的值为 ``0`` ，
    ``len`` 成员的值也为 ``0`` ，
    而 ``buf`` 成员里面也只有一个函数自己添加的空字符。

    执行：

    ::

        sdsempty();

    将返回以下 SDS ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n0 | <buf> buf"];

            buf [label = "{'\\0'}"];

            //

            sdshdr:buf -> buf;

        }

    ``sdsempty`` 适用于那些需要创建空白 SDS 来进行初始化工作，
    然后使用拼接或者复制来将内容写入到 SDS 的场合。

    因为 ``sdsempty`` 函数的全部工作就是创建并初始化一个空白的 ``sdshdr`` 结构，
    所以它的复杂度为 :math:`O(1)` 。


    释放 SDS
    ---------------

    ``sdsfree`` 用于释放 SDS 所保存的字符串，
    以及 ``sdshdr`` 结构本身：

    ::

        void sdsfree(sdshdr *s);

    对于以下 SDS 来说：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n5 | <buf> buf"];

            buf [label = "{ 'R' | 'e' | 'd' | 'i' | 's' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    执行 ``sdsfree`` 的时候，
    函数将先释放 ``sdshdr.buf`` 成员里保存的字符串：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            free [label = "释放", shape = plaintext];

            sdshdr [label = "sdshdr | free:\n0 | len:\n5 | <buf> buf"];

            buf [label = "{ 'R' | 'e' | 'd' | 'i' | 's' | '\\0' }"];


            //

            sdshdr:buf -> buf;

            free -> buf [style = dashed];
        }

    然后再释放 SDS 对应的 ``sdshdr`` 结构：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n5 | <buf> buf"];

            free [label = "释放", shape = plaintext];

            //

            free -> sdshdr [style = dashed];
        }

    对于长度为 :math:`N` 的字符串，
    释放 ``sdshdr.buf`` 数组需要释放 :math:`N` 个字符，
    因此，
    ``sdsfree`` 的复杂度为 :math:`O(N)` 。


    SDS 成员选择函数
    --------------------

    成员选择函数用于返回给定 SDS 的各项属性。

    ``sdslen`` 接受一个 SDS 作为输入，
    并返回该 SDS 的已使用空间字节数量：

    ::

        size_t sdslen(const sdshdr *s);


    ``sdsavail`` 接受一个 SDS 作为输入，
    并返回该 SDS 的未使用空间字节数量：

    ::

        size_t sdsavail(const sdshdr *s);

    比如说，
    对于以下 SDS ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n3 | <buf> buf"];

            buf [label = "{ 'a' | 'b' | 'c' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    ``sdslen`` 函数将返回 ``3`` ，
    而 ``sdsavail`` 函数将返回 ``0`` 。

    另一方面，
    对于以下 SDS ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n5 | len:\n5 | <buf> buf"];

            buf [label = "{ 'R' | 'e' | 'd' | 'i' | 's' | '\\0' | | | | | }"];

            //

            sdshdr:buf -> buf;

        }

    ``sdslen`` 函数将返回 ``5`` ，
    而 ``sdsavail`` 函数也将返回 ``5`` 。

    因为 ``sdslen`` 的工作就是读取 ``sdshdr`` 结构的 ``len`` 成员，
    而 ``sdsavail`` 的工作就是读取 ``sdshdr`` 结构的 ``free`` 成员，
    所以这两个函数的复杂度都是 :math:`O(1)` 。


    创建 SDS 副本
    -------------------

    ``sdsdup`` 函数接受一个 SDS 作为输入，
    并返回该 SDS 的副本（duplicate）：

    ::

        sdshdr *sdsdup(const sdshdr *s);

    举个例子，
    对于一个 SDS 值 ``source`` 来说：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "<head> sdshdr | free:\n0 | len:\n3 | <buf> buf"];

            buf [label = "{ 'a' | 'b' | 'c' | '\\0' }"];

            source [shape = plaintext];

            //

            sdshdr:buf -> buf;

            source -> sdshdr:head;

        }

    执行 ``sdshdr *copy = sdsdup(source);`` 将返回一个和 ``source`` 完全相同的 SDS 值 ``copy`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "<head> sdshdr | free:\n0 | len:\n3 | <buf> buf"];

            buf [label = "{ 'a' | 'b' | 'c' | '\\0' }"];

            copy [shape = plaintext];

            //

            sdshdr:buf -> buf;

            copy -> sdshdr:head;
        }

    注意，
    输入 SDS 和 SDS 副本并不共享 ``buf`` 数组 ——
    这两个 SDS 的 ``buf`` 都有自己的独立空间。

    创建 SDS 副本需要复制输入 SDS 的 ``buf`` 数组里面的所有内容，
    对于长度为 :math:`N` 的数组，
    程序要复制 :math:`N` 个字符，
    因此这个函数的复杂度为 :math:`O(N)` 。


    重置 SDS
    --------------------

    ``sdsclear`` 接受一个 SDS 作为输入，
    将 SDS 所保存的字符串重置为空字符串，
    并且重置不会改动 ``sdshdr.buf`` 数组的空间大小：

    ::

        void sdsclear(sdshdr *s);

    举个例子，
    如果对以下 SDS 执行 ``sdsclear`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n3 | <buf> buf"];

            buf [label = "{ 'a' | 'b' | 'c' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    那么这个 SDS 将被更新为以下状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n3 | len:\n0 | <buf> buf"];

            buf [label = "{ '\\0' | 'b' | 'c' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    ``sdsclear`` 更新了 ``sdshdr.free`` 成员和 ``sdshdr.len`` 成员的值，
    并将空字符放到了 ``sdshdr.buf`` 数组的索引位置 ``0`` 上面。

    整个重置操作都是惰性的，
    因为函数：

    1. 既不会对 ``sdshdr.buf`` 进行内存重分配；

    2. 也不会擦除 ``sdshdr.buf`` 里原有字符串遗留下来的内容 ——
       比如上面 SDS 示例中的 ``'b'`` 、 ``'c'`` 和数组末尾的 ``'\0'`` 三个字符：
       因为当有新字符写入的时候，
       这些旧的字符就会被覆盖，
       所以没有必要特意去进行擦除操作。

    和前面介绍过的空间预分配一样，
    这种惰性释放空间的策略也是 SDS 基于未使用空间而设置的优化手段：
    程序期望将来对 SDS 的操作会用到这些未被释放的空间，
    以此来减少对 SDS 进行内存释放和内存重分配的次数。

    因为 ``sdsclear`` 只是简单地对 ``sdshdr`` 结构的几个成员执行了一些常数复杂度的设置操作，
    所以 ``sdsclear`` 函数的复杂度为 :math:`O(1)` 。


    拼接
    --------------

    ``sdscat`` 接受一个 SDS 和一个字符串字面量作为输入参数，
    并将字符串字面量的值拼接到 SDS 原有的字符串值之后：

    ::

        sdshdr *sdscat(sdshdr *s, const char *t);

    如果 SDS 的未使用空间不足以容纳要拼接的字符串，
    那么函数将先对 SDS 进行扩展，
    然后再执行拼接操作。

    举个例子，
    如果我们有这样一个 SDS ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n2 | <buf> buf"];

            buf [label = "{ 'R' | 'e' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    那么在执行：

    ::

        sdscat(s, "dis");

    之后，
    这个 SDS 将变成这样：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n5 | len:\n5 | <buf> buf"];

            buf [label = "{ 'R' | 'e' | 'd' | 'i' | 's' | '\\0' | | | | | }"];

            //

            sdshdr:buf -> buf;

        }

    注意，
    由于前面提到的内存预分配策略，
    SDS 在拼接操作完成之后，
    ``buf`` 数组里仍然会留有一些未使用空间，
    等待将来使用。

    除了 ``sdscat`` 之外，
    ``sdscatsds`` 也可以进行字符串拼接操作，
    不过这两个函数接受的参数稍有不同 ——
    ``sdscat`` 接受一个 SDS 和一个字符串字面量，
    而 ``sdscatsds`` 则接受一个源 SDS 和一个目标 SDS ，
    并将目标 SDS 所保存的字符串值拼接到源 SDS 现有字符串值之后：

    ::

        sdshdr *sdscatsds(sdshdr *s, const sdshdr *t);

    比如说，
    对于以下两个 SDS 值 ``source`` 和 ``target`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            target [shape = plaintext];

            target_sdshdr [label = "<head> sdshdr | free:\n0 | len:\n3 | <buf> buf"];

            target_buf [label = "{ 'd' | 'i' | 's' | '\\0' }"];

            //

            target_sdshdr:buf -> target_buf;

            target -> target_sdshdr:head;

            //

            source [shape = plaintext];

            source_sdshdr [label = "<head> sdshdr | free:\n0 | len:\n2 | <buf> buf"];

            source_buf [label = "{ 'R' | 'e' | '\\0' }"];

            //

            source_sdshdr:buf -> source_buf;

            source -> source_sdshdr:head;

        }

    执行函数：

    ::

        sdscatsds(source, target);

    会将 ``source`` 更新为以下状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            source [shape = plaintext];

            sdshdr [label = "<head> sdshdr | free:\n5 | len:\n5 | <buf> buf"];

            buf [label = "{ 'R' | 'e' | 'd' | 'i' | 's' | '\\0' | | | | | }"];

            //

            sdshdr:buf -> buf;

            source -> sdshdr:head;

        }

    和 ``sdscat`` 一样，
    ``sdscatsds`` 也会在源字符串的未使用空间不足时，
    自动扩展字符串空间，
    并且会预分配一些未使用空间。

    另一方面，
    被复制的目标 SDS ``target`` 不会有任何变化 ——
    ``sdscatsds`` 只读取目标 SDS 所保存的字符串值，
    而不会对这个 SDS 进行任何修改。

    因为 ``sdscat`` 和 ``sdscatsds`` 都需要将长度为 :math:`N` 的字符串拼接到 SDS 已有的字符串之后，
    所以它们的复杂度都为 :math:`O(N)` 。

    另外，
    虽然 ``sdscat`` 和 ``sdscatsds`` 两个函数都由同一个底层函数实现，
    但是由于 ``sdscat`` 需要执行 :math:`O(N)` 复杂度的 ``<string.h>/strlen`` 来获取字符串字面量的长度，
    而 ``sdscatsds`` 只需执行 :math:`O(1)` 复杂度的 ``sdslen`` 就可以完成获取 SDS 长度的工作，
    所以 ``sdscatsds`` 的执行效率要比 ``sdscat`` 要高。


    复制 SDS
    --------------

    ``sdscpy`` 函数用于将字符串 ``t`` 完整地复制到给定 SDS 的 ``buf`` 数组中：

    ::

        sdshdr* sdscpy(sdshdr *sds, const char *t);

    复制从 ``buf`` 数组的开头 —— 也即是索引 ``0`` 开始进行，
    数组中已有的内容会被复制后的内容覆盖。

    当 ``buf`` 数组的长度不足以容纳 ``t`` 的时候，
    函数先对 ``sds`` 进行扩展，
    然后再进行复制操作。

    假设有一个保存了字符串 ``"NoSQL"`` 的 SDS 值 ``s1`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "<head> sdshdr | free:\n0 | len:\n5 | <buf> buf"];

            buf [label = "{ 'N' | 'o' | 'S' | 'Q' | 'L' | '\\0' }"];

            //

            sdshdr:buf -> buf;
        }

    当执行调用 ``sdscpy(s1, "Hi");`` 之后，
    ``s1`` 将被修改成这样：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "<head> sdshdr | free:\n3 | len:\n2 | <buf> buf"];

            buf [label = "{ 'H' | 'i' | '\\0' | 'Q' | 'L' | '\\0' }"];

            //

            sdshdr:buf -> buf;
        }

    因为 SDS 的 ``buf`` 数组的长度足以容纳 ``t`` ，
    所以复制操作会直接进行。

    复制完成之后， ``s1`` 发生了以下变化：

    - 保存的字符串值从 ``"NoSQL"`` 变为 ``"Hi"`` ；

    - 已使用空间字节数从 ``5`` 变为 ``2`` ；

    - 未使用空间字节数从 ``0`` 变为 ``3`` 。

    现在，
    假设有一个保存了字符串 ``"Hi"`` 、
    并且没有任何未使用空间的 SDS 值 ``s2`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "<head> sdshdr | free:\n0 | len:\n2 | <buf> buf"];

            buf [label = "{ 'H' | 'i' | '\\0' }"];

            //

            sdshdr:buf -> buf;
        }

    如果我们对 ``s2`` 执行 ``sdscpy(s2, "NoSQL");`` ，
    那么函数会先对 ``s2`` 进行扩展，
    然后再复制字符串值。

    以下是复制完成之后，
    ``s2`` 的样子：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "<head> sdshdr | free:\n5 | len:\n5 | <buf> buf"];

            buf [label = "{ 'N' | 'o' | 'S' | 'Q' | 'L' | '\\0' | | | | | }"];

            //

            sdshdr:buf -> buf;
        }

    复制完成之后， ``s2`` 发生了以下变化：

    - 保存的字符串值从 ``"Hi"`` 变为 ``"NoSQL"`` ；

    - 已使用空间字节数从 ``2`` 变为 ``5`` ；

    - 未使用空间字节数从 ``0`` 变为 ``5`` 。

    对于长度为 :math:`N` 的字符串输入，
    ``sdscpy`` 需要复制 :math:`N` 个字符，
    所以 ``sdscpy`` 的复杂度为 :math:`O(N)` 。


    扩展并用空字符填充 SDS
    ---------------------------

    ``sdsgrowzero`` 函数接受一个参数 ``len`` ，
    并将 SDS 所保存的字符串扩展至 ``len`` 所指定的大小：

    ::

        sdshdr* sdsgrowzero(sdshdr *sds, size_t len);

    扩展大小所产生的空间会使用空字符进行填充。

    比如说，
    以下是一个保存了字符串 ``"Hi"`` 的 SDS ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "<head> sdshdr | free:\n0 | len:\n2 | <buf> buf"];

            buf [label = "{ 'H' | 'i' | '\\0' }"];

            //

            sdshdr:buf -> buf;
        }

    如果对这个 SDS 执行 ``sdsgrowzero(sds, 5);`` 的话，
    它将变成这个样子：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "<head> sdshdr | free:\n5 | len:\n5 | <buf> buf"];

            buf [label = "{ 'H' | 'i' | '\\0' | '\\0' | '\\0' | '\\0' | | | | | }"];

            //

            sdshdr:buf -> buf;
        }

    更新后的 SDS 共有四个空字符，
    其中前三个（数组索引 ``2`` 、 ``3`` 、 ``4`` ）为填充空字符。

    另外，
    由于预分配空间策略的作用，
    扩展后的 SDS 会带有额外的未使用空间。

    因为 ``sdsgrowzero`` 扩展并填充 :math:`N` 个字节总需要复制 :math:`N` 个空字符，
    所以 ``sdsgrowzero`` 的复杂度为 :math:`O(N)` 。


    区间截取
    -------------

    ``sdsrange`` 用于按索引区间截取 SDS ：

    ::

        void sdsrange(sdshdr *sds, int start, int end);

    以下是该函数行为的详细描述：
       
    - 字符串从索引 ``start`` 到 ``end`` 的部分都会被保留（\ ``start`` 和 ``end`` 都包含在内），
      其他部分则会被删除；

    - 索引从 ``0`` 开始，
      最大值为 ``sdslen(sds) - 1`` ；

    - 索引也可以是负数：
      ``sdslen(sds) - 1 == -1`` ，
      ``sdslen(sds) - 2 == -2`` ，
      以此类推；

    - 超过字符串长度范围的索引会被自动忽略。

    比如说，
    对以下 SDS 执行 ``sdsrange(sds, 2, 3);`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n5 | <buf> buf"];

            buf [label = "{ 'a' | 'b' | 'c' | 'd' | 'e' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    SDS 将更新为以下状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n3 | len:\n2 | <buf> buf"];

            buf [label = "{ 'c' | 'd' | '\\0' | 'd' | 'e' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    对于长度为 :math:`N` 的截取区间，
    ``sdsrange`` 需要对 :math:`N` 个字节进行移动（\ ``memmove``\ ），
    因此，
    ``sdsrange`` 的复杂度为 :math:`O(N)` 。


    修剪
    -----------

    ``sdstrim`` 用于对 SDS 所保存的字符串进行修剪（trim）：

    ::

        sdshdr* sdstrim(sdshdr *s, const char *cset);

    函数将从 ``s`` 的头尾两端删除所有包含在 ``cset`` 字符串中的字符。

    举个例子，
    对于以下 SDS 值 ``s`` ：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n0 | len:\n7 | <buf> buf"];

            buf [label = "{ 'a' | 'b' | 'a' | 'h' | 'i' | 'b' | 'c' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    执行 ``sdstrim(s, "abc");`` 将使得这个 SDS 更新为以下状态：

    .. graphviz::

        digraph {

            rankdir = LR;

            node [shape = record];

            //

            sdshdr [label = "sdshdr | free:\n5 | len:\n2 | <buf> buf"];

            buf [label = "{ 'h' | 'i' | '\\0' | 'h' | 'i' | 'b' | 'c' | '\\0' }"];

            //

            sdshdr:buf -> buf;

        }

    ``sdstrim`` 执行之后，
    SDS 发生了以下改变：

    - 保存的字符串从 ``"abahibc"`` 变为 ``"hi"`` ；

    - 未使用空间字节数从 ``0`` 变为 ``5`` ；

    - 已使用空间字节数从 ``7`` 变为 ``2`` 。

    对于 SDS 两端的每个字符 ``c`` ，
    ``sdstrim`` 都需要在长度为 :math:`N` 的字符串 ``cset`` 中检查 ``c`` 是否包含在 ``cset`` 之内，
    因此，
    ``sdstrim`` 的复杂度为 :math:`O(N^2)` 。


    对比
    ------------------------

    ``sdscmp`` 函数对两个 SDS 所保存的字符串进行对比：

    ::

        int sdscmp(const sdshdr *s1, const sdshdr *s2);

    - 当 ``s1`` 和 ``s2`` 所保存的字符串相等时，
      ``sdscmp`` 返回 ``0`` ；

    - 当 ``s1`` 的字符串比 ``s2`` 的字符串要大时，
      返回正数；

    - 当 ``s2`` 的字符串比 ``s1`` 的字符串大时，
      返回负数。

    以下是一些 ``sdscmp`` 示例：

    - 执行代码 ``sdscmp(sdsnew("Redis"), sdsnew("Redis"));`` 将返回 ``0`` ；

    - 执行代码 ``sdscmp(sdsnew("aaa"), sdsnew("zzz"));`` 将返回负数，因为 ``"aaa"`` 小于 ``"zzz"`` ；

    - 执行代码 ``sdsnew(sdsnew("zzz"), sdsnew("aaa"));`` 将返回正数，因为 ``"zzz"`` 大于 ``"aaa"`` 。

    因为至少需要对比 :math:`N` 个字符，
    才能决定两个字符串是否相等，
    或者这哪个更大哪个更小，
    所以 ``sdscmp`` 的复杂度为 :math:`O(N)` 。


    其他 SDS 函数
    --------------------

    因为边幅所限，
    我们不能介绍 SDS 函数库中的所有 API ，
    但是，
    上面介绍的内容已经覆盖了 SDS 函数库中最重要的那部分 API 。

    因为 SDS 是 Redis 各个功能的基础，
    所以应该尽可能地去理解 SDS 的定义，
    并熟悉 SDS 的 API ，
    这对于理解之后介绍的所有功能都会有帮助。
