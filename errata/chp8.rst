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
