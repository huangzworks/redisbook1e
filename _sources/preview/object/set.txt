集合对象
-------------

集合对象的编码可以是 ``intset`` 或者 ``hashtable`` 。

``intset`` 编码的集合对象使用整数集合作为底层实现，
集合对象包含的所有元素都被保存在整数集合里面。

举个例子，
以下代码将创建一个如图 8-12 所示的 ``intset`` 编码集合对象：

::

    redis> SADD numbers 1 3 5 
    (integer) 3

.. graphviz::

    digraph {

        label = "\n 图 8-12    intset 编码的 numbers 集合对象";

        rankdir = LR;

        node [shape = record];

        redisObject [label = " redisObject | type \n REDIS_SET | encoding \n REDIS_ENCODING_INTSET | <ptr> ptr | ... "];
        intset [label = " <head> intset | encoding \n INTSET_ENC_INT16 | length \n 3 | <contents> contents "];

        contents [label = " { 1 | 3 | 5 } "];

        redisObject:ptr -> intset:head;
        intset:contents -> contents;

    }

另一方面，
``hashtable`` 编码的集合对象使用字典作为底层实现，
字典的每个键都是一个字符串对象，
每个字符串对象包含了一个集合元素，
而字典的值则全部被设置为 ``NULL`` 。

举个例子，
以下代码将创建一个如图 8-13 所示的 ``hashtable`` 编码集合对象：

::

    redis> SADD fruits "apple" "banana" "cherry"
    (integer) 3

.. graphviz::

    digraph {

        label = "\n 图 8-13    hashtable 编码的 fruits 集合对象";

        rankdir = LR;

        node [shape = record];

        redisObject [label = " redisObject | type \n REDIS_SET | encoding \n REDIS_ENCODING_HT | <ptr> ptr | ... "];

        dict [label = " <head> dict | <cherry> StringObject \n \"cherry\" | <apple> StringObject \n \"apple\" | <banana> StringObject \n \"banana\" ", width = 1.5];

        redisObject:ptr -> dict:head;

        node [shape = plaintext, label = "NULL"];

        dict:apple -> nullX;
        dict:banana -> nullY;
        dict:cherry -> nullZ;
 
    }

..
    集合对象保存的每个元素就是整数集合的一个元素。
    ``intset`` 编码使用整数集合作为集合对象的底层实现，
      集合元素会作为整数集合的元素被保存起来。



    - 其中 ``intset`` 编码使用整数集合作为集合对象的底层实现，
      集合元素会作为整数集合的元素被保存起来。

    - 而 ``hashtable`` 编码则使用字典作为集合对象的底层实现，
      字典中的键就是集合中的元素，
      字典中的值则全部被设置为 ``NULL`` 。

    举个例子，
    以下代码将创建一个如图 8-12 所示的 ``intset`` 编码集合对象：

    ::

        redis> SADD numbers 1 3 5 
        (integer) 3

    .. graphviz::

        digraph {

            label = "\n 图 8-12    intset 编码的 numbers 集合对象";

            rankdir = LR;

            node [shape = record];

            redisObject [label = " redisObject | type \n REDIS_SET | encoding \n REDIS_ENCODING_INTSET | <ptr> ptr | ... "];
            intset [label = " <head> intset | encoding \n INTSET_ENC_INT16 | length \n 3 | <contents> contents "];

            contents [label = " { 1 | 3 | 5 } "];

            redisObject:ptr -> intset:head;
            intset:contents -> contents;

        }

    而以下代码则会创建一个如图 8-13 所示的 ``hashtable`` 编码集合对象：

    ::

        redis> SADD fruits "apple" "banana" "cherry"
        (integer) 3

    .. graphviz::

        digraph {

            label = "\n 图 8-13    hashtable 编码的 fruits 集合对象";

            rankdir = LR;

            node [shape = record];

            redisObject [label = " redisObject | type \n REDIS_SET | encoding \n REDIS_ENCODING_HT | <ptr> ptr | ... "];

            dict [label = " <head> dict | <cherry> \"cherry\" | <apple> \"apple\" | <banana> \"banana\" ", width = 1.5];

            redisObject:ptr -> dict:head;

            node [shape = plaintext, label = "NULL"];

            dict:apple -> nullX;
            dict:banana -> nullY;
            dict:cherry -> nullZ;
     
        }


编码的转换
^^^^^^^^^^^^^^^^^^^

当集合对象可以同时满足以下两个条件时，
对象使用 ``intset`` 编码：

1. 集合对象保存的所有元素都是整数值；

2. 集合对象保存的元素数量不超过 ``512`` 个；

不能满足这两个条件的集合对象需要使用 ``hashtable`` 编码。

.. topic:: 注意

    第二个条件的上限值是可以修改的，
    具体请看配置文件中关于 ``set-max-intset-entries`` 选项的说明。

对于使用 ``intset`` 编码的集合对象来说，
当使用 ``intset`` 编码所需的两个条件的任意一个不能被满足时，
对象的编码转换操作就会被执行：
原本保存在整数集合中的所有元素都会被转移并保存到字典里面，
并且对象的编码也会从 ``intset`` 变为 ``hashtable`` 。

举个例子，
以下代码创建了一个只包含整数元素的集合对象，
该对象的编码为 ``intset`` ：

::

    redis> SADD numbers 1 3 5 
    (integer) 3

    redis> OBJECT ENCODING numbers
    "intset"

不过，
只要我们向这个只包含整数元素的集合对象添加一个字符串元素，
集合对象的编码转移操作就会被执行：

::

    redis> SADD numbers "seven"
    (integer) 1

    redis> OBJECT ENCODING numbers
    "hashtable"

除此之外，
如果我们创建一个包含 ``512`` 个整数元素的集合对象，
那么对象的编码应该会是 ``intset`` ：

::

    redis> EVAL "for i=1, 512 do redis.call('SADD', KEYS[1], i) end" 1 integers
    (nil)

    redis> SCARD integers
    (integer) 512

    redis> OBJECT ENCODING integers
    "intset"

但是，
只要我们再向集合添加一个新的整数元素，
使得这个集合的元素数量变成 ``513`` ，
那么对象的编码转换操作就会被执行：

::

    redis> SADD integers 10086
    (integer) 1

    redis> SCARD integers
    (integer) 513

    redis> OBJECT ENCODING integers
    "hashtable"


集合命令的实现
^^^^^^^^^^^^^^^^^^^^

因为集合键的值为集合对象，
所以用于集合键的所有命令都是针对集合对象来构建的，
表 8-10 列出了其中一部分集合键命令，
以及这些命令在不同编码的集合对象下的实现方法。

-------------------------------------------------------------------------------------------------------------------------

表 8-10    集合命令的实现方法

+-----------------------+-------------------------------------------+---------------------------------------------------+
| 命令                  | ``intset`` 编码的实现方法                 | ``hashtable`` 编码的实现方法                      |
+=======================+===========================================+===================================================+
| :ref:`SADD`           | 调用 ``intsetAdd`` 函数，                 | 调用 ``dictAdd`` ，                               |
|                       | 将所有新元素添加到整数集合里面。          | 以新元素为键， ``NULL`` 为值，                    |
|                       |                                           | 将键值对添加到字典里面。                          |
+-----------------------+-------------------------------------------+---------------------------------------------------+
| :ref:`SCARD`          | 调用 ``intsetLen`` 函数，                 | 调用 ``dictSize`` 函数，                          |
|                       | 返回整数集合所包含的元素数量，            | 返回字典所包含的键值对数量，                      |
|                       | 这个数量就是集合对象所包含的元素数量。    | 这个数量就是集合对象所包含的元素数量。            |
+-----------------------+-------------------------------------------+---------------------------------------------------+
| :ref:`SISMEMBER`      | 调用 ``intsetFind`` 函数，                | 调用 ``dictFind`` 函数，                          |
|                       | 在整数集合中查找给定的元素，              | 在字典的键中查找给定的元素，                      |
|                       | 如果找到了说明元素存在于集合，            | 如果找到了说明元素存在于集合，                    |
|                       | 没找到则说明元素不存在于集合。            | 没找到则说明元素不存在于集合。                    |
+-----------------------+-------------------------------------------+---------------------------------------------------+
| :ref:`SMEMBERS`       | 遍历整个整数集合，                        | 遍历整个字典，                                    |
|                       | 使用 ``intsetGet`` 函数返回集合元素。     | 使用 ``dictGetKey`` 函数返回字典的键作为集合元素。|
+-----------------------+-------------------------------------------+---------------------------------------------------+
| :ref:`SRANDMEMBER`    | 调用 ``intsetRandom`` 函数，              | 调用 ``dictGetRandomKey`` 函数，                  |
|                       | 从整数集合中随机返回一个元素。            | 从字典中随机返回一个字典键。                      |
+-----------------------+-------------------------------------------+---------------------------------------------------+
| :ref:`SPOP`           | 调用 ``intsetRandom`` 函数，              | 调用 ``dictGetRandomKey`` 函数，                  |
|                       | 从整数集合中随机取出一个元素，            | 从字典中随机取出一个字典键，                      |
|                       | 在将这个随机元素返回给客户端之后，        | 在将这个随机字典键的值返回给客户端之后，          |
|                       | 调用 ``intsetRemove`` 函数，              | 调用 ``dictDelete`` 函数，                        |
|                       | 将随机元素从整数集合中删除掉。            | 从字典中删除随机字典键所对应的键值对。            |
+-----------------------+-------------------------------------------+---------------------------------------------------+
| :ref:`SREM`           | 调用 ``intsetRemove`` 函数，              | 调用 ``dictDelete`` 函数，                        |
|                       | 从整数集合中删除所有给定的元素。          | 从字典中删除所有键为给定元素的键值对。            |
+-----------------------+-------------------------------------------+---------------------------------------------------+
