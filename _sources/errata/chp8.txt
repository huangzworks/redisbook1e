第 8 章《对象》
==================

64 页
-------------

在这一页的倒数第二段：

    如果字符串对象保存的是一个字符串值， 
    并且这个字符串值的长度大于 ``32`` 字节， 
    那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串值， 
    并将对象的编码设置为 ``raw`` 。

在本书定稿之后，
Redis 将这个值从 ``32`` 改为了 ``39`` ，
所以我们也需要进行相应的更新，
修改之后的内容为：

    如果字符串对象保存的是一个字符串值， 
    并且这个字符串值的长度大于 ``39`` 字节， 
    那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串值， 
    并将对象的编码设置为 ``raw`` 。

受此影响，
65 页的图 8-2 也要更新为以下模样（图中的 ``len`` 属性有改动）：

.. graphviz::

    digraph {

        label = "\n 图 8-2    raw 编码的字符串对象";

        rankdir = LR;

        node [shape = record];

        redisObject [label = " redisObject | type \n REDIS_STRING | encoding \n REDIS_ENCODING_RAW | <ptr> ptr | ... "];

        sdshdr [label = " <head> sdshdr | free \n 0 | len \n 43 | <buf> buf"];

        buf [label = " { 'L' | 'o' | 'n' | 'g' | ... | 'k' | 'i' | 'n' | 'g' | ' ' | '.' | '.' | '.' | '\\0' } " ];

        //

        redisObject:ptr -> sdshdr:head;
        sdshdr:buf -> buf;

    }

而创建图 8-2 所示字符串的代码也要改为：

::

    redis> SET story "Long, long, long ago there lived a king ..."
    OK

    redis> STRLEN story
    (integer) 43

    redis> OBJECT ENCODING story
    "raw"

感谢 `xp <http://redisbook.com/en/latest/preview/object/string.html#comment-1481763423>`_ 反馈这个问题。


65 页
------------

在这一页的中间开头第一段：

    如果字符串对象保存的是一个字符串值， 
    并且这个字符串值的长度小于等于 ``32`` 字节， 
    那么字符串对象将使用 ``embstr`` 编码的方式来保存这个字符串值。

在本书定稿之后，
Redis 将这个值从 ``32`` 改为了 ``39`` ，
所以我们也需要进行相应的更新，
修改之后的内容为：

    如果字符串对象保存的是一个字符串值， 
    并且这个字符串值的长度小于等于 ``39`` 字节， 
    那么字符串对象将使用 ``embstr`` 编码的方式来保存这个字符串值。

感谢 `xp <http://redisbook.com/en/latest/preview/object/string.html#comment-1481763423>`_ 反馈这个问题。


75 页
------------

在 8.5 节开头的介绍内容中，
第二段代码：

::

    redis> SAD Dfruits "apple" "banana" "cherry"
    (integer)3

中的 ``SADD`` 命令印刷出错，
正确的代码应该为：

::

    redis> SADD fruits "apple" "banana" "cherry"
    (integer)3

感谢 `zhkzyth <http://www.douban.com/people/zhkzyth/>`_ 反馈这个错误。


77 页
-----------

8.6 节开头的第二段：

    ``ziplist`` 编码的压缩列表对象使用压缩列表作为底层实现，
    ……

中的“压缩列表对象”有误，应该改为“有序集合对象”才对，
以下是修正之后的内容：

    ``ziplist`` 编码的有序集合对象使用压缩列表作为底层实现，
    ……


80 页
-----------

本页的最后一句：

    因为有序集合键的值为哈希对象，
    所以用于有序集合键的所有命令都是针对哈希对象来构建的，
    表 8-1 列出了其中一部分有序集合键命令，
    以及这些命令在不同编码的哈希对象下的实现方法。

这里的“哈希对象”应该全部改为“有序集合对象”才对，
以下是修正后的正文：

    因为有序集合键的值为有序集合对象，
    所以用于有序集合键的所有命令都是针对有序集合对象来构建的，
    表 8-1 列出了其中一部分有序集合键命令，
    以及这些命令在不同编码的有序集合对象下的实现方法。

感谢 Andy 反馈这个问题。
