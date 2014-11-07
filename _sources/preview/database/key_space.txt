数据库键空间
----------------

Redis 是一个键值对（key-value pair）数据库服务器，
服务器中的每个数据库都由一个 ``redis.h/redisDb`` 结构表示，
其中，
``redisDb`` 结构的 ``dict`` 字典保存了数据库中的所有键值对，
我们将这个字典称为键空间（key space）：

::

    typedef struct redisDb {

        // ...

        // 数据库键空间，保存着数据库中的所有键值对
        dict *dict;

        // ...

    } redisDb;

键空间和用户所见的数据库是直接对应的：

- 键空间的键也就是数据库的键，
  每个键都是一个字符串对象。

- 键空间的值也就是数据库的值，
  每个值可以是字符串对象、列表对象、哈希表对象、集合对象和有序集合对象在内的任意一种 Redis 对象。

举个例子，
如果我们在空白的数据库中执行以下命令：

::

    redis> SET message "hello world"
    OK

    redis> RPUSH alphabet "a" "b" "c"
    (integer) 3

    redis> HSET book name "Redis in Action"
    (integer) 1

    redis> HSET book author "Josiah L. Carlson"
    (integer) 1

    redis> HSET book publisher "Manning"
    (integer) 1

那么在这些命令执行之后，
数据库的键空间将会是图 IMAGE_DB_EXAMPLE 所展示的样子：

- ``alphabet`` 是一个列表键，
  键的名字是一个包含字符串 ``"alphabet"`` 的字符串对象，
  键的值则是一个包含三个元素的列表对象。

- ``book`` 是一个哈希表键，
  键的名字是一个包含字符串 ``"book"`` 的字符串对象，
  键的值则是一个包含三个键值对的哈希表对象。

- ``message`` 是一个字符串键，
  键的名字是一个包含字符串 ``"message"`` 的字符串对象，
  键的值则是一个包含字符串 ``"hello world"`` 的字符串对象。

.. graphviz::

    digraph {

        label = "\n图 IMAGE_DB_EXAMPLE    数据库键空间例子";

        rankdir = LR;

        node [shape = record];

        //

        redisDb [label = "redisDb | ... | <dict> dict | ..."];

        dict [label = "<dict> dict | <alphabet> StringObject \n \"alphabet\" | <book> StringObject \n \"book\" | <message> StringObject \n \"message\""];

        subgraph cluster_alphabet {

            a [label = " StringObject \n \"a\" "];
            b [label = " StringObject \n \"b\" "];
            c [label = " StringObject \n \"c\" "];

            a -> b -> c;

            label = "ListObject";

        }

        //alphabet [label = "<head> ListObject | { StringObject \n \"a\" | \"b\" | \"c\" }"];

        book [label = "<head> HashObject | <name> StringObject \n \"name\" | <author> StringObject \n \"author\" | <publisher> StringObject \n \"publisher\""];

        //name [label = " StringObject \n \"Redis in Action\""];
        name [label = " StringObject \n \"Redis in Action\""];

        author [label = " StringObject \n \"Josiah L. Carlson\""];

        publisher [label = " StringObject \n \"Manning\""];

        message [label = " StringObject \n \"hello world\""];

        //

        redisDb:dict -> dict:dict;

        dict:alphabet -> a;
        dict:book -> book:head;
        dict:message -> message;

        book:name -> name;
        book:publisher -> publisher;
        book:author -> author;

    }

因为数据库的键空间是一个字典，
所以所有针对数据库的操作 ——
比如添加一个键值对到数据库，
或者从数据库中删除一个键值对，
又或者在数据库中获取某个键值对，
等等，
实际上都是通过对键空间字典进行操作来实现的，
以下几个小节将分别介绍数据库的添加、删除、更新、取值等操作的实现原理。


添加新键
^^^^^^^^^^^^^

添加一个新键值对到数据库，
实际上就是将一个新键值对添加到键空间字典里面，
其中键为字符串对象，
而值则为任意一种类型的 Redis 对象。

举个例子，
如果键空间当前的状态如图 IMAGE_DB_EXAMPLE 所示，
那么在执行以下命令之后：

::

    redis> SET date "2013.12.1"
    OK

键空间将添加一个新的键值对，
这个新键值对的键是一个包含字符串 ``"date"`` 的字符串对象，
而键值对的值则是一个包含字符串 ``"2013.12.1"`` 的字符串对象，
如图 IMAGE_DB_AFTER_ADD_NEW_KEY 所示。

.. graphviz::

    digraph {

        label = "\n图 IMAGE_DB_AFTER_ADD_NEW_KEY    添加 date 键之后的键空间";

        rankdir = LR;

        node [shape = record];

        //

        redisDb [label = "redisDb | ... | <dict> dict | ..."];

        dict [label = "<dict> dict | <alphabet> StringObject \n \"alphabet\" | <book> StringObject \n \"book\" | <message> StringObject \n \"message\" | <date> StringObject \n \"date\""];

        subgraph cluster_alphabet {

            a [label = " StringObject \n \"a\" "];
            b [label = " StringObject \n \"b\" "];
            c [label = " StringObject \n \"c\" "];

            a -> b -> c;

            label = "ListObject";

        }

        book [label = "<head> HashObject | <name> StringObject \n \"name\" | <author> StringObject \n \"author\" | <publisher> StringObject \n \"publisher\""];

        name [label = " StringObject \n \"Redis in Action\""];

        author [label = " StringObject \n \"Josiah L. Carlson\""];

        publisher [label = " StringObject \n \"Manning\""];

        message [label = " StringObject \n \"hello world\""];

        date [label = " StringObject \n \"2013.12.1\""];

        //

        redisDb:dict -> dict:dict;

        dict:alphabet -> a;
        dict:book -> book:head;
        dict:message -> message;

        book:name -> name;
        book:publisher -> publisher;
        book:author -> author;

        dict:date -> date;

        // 
        
        node [shape = plaintext]

        newadd [label = "新添加"]

        newadd -> dict:date [style = dashed]

    }


删除键
^^^^^^^^^

删除数据库中的一个键，
实际上就是在键空间里面删除键所对应的键值对对象。

举个例子，
如果键空间当前的状态如图 IMAGE_DB_EXAMPLE 所示，
那么在执行以下命令之后：

::

    redis> DEL book
    (integer) 1

键 ``book`` 以及它的值将从键空间中被删除，
如图 IMAGE_DB_AFTER_DEL 所示。

.. graphviz::

    digraph {

        label = "\n图 IMAGE_DB_AFTER_DEL    删除 book 键之后的键空间";

        rankdir = LR;

        node [shape = record];

        //

        redisDb [label = "redisDb | ... | <dict> dict | ..."];

        dict [label = "<dict> dict | <alphabet> StringObject \n \"alphabet\" |  <message> StringObject \n \"message\""];

        subgraph cluster_alphabet {

            a [label = " StringObject \n \"a\" "];
            b [label = " StringObject \n \"b\" "];
            c [label = " StringObject \n \"c\" "];

            a -> b -> c;

            label = "ListObject";

        }

        message [label = " StringObject \n \"hello world\""];

        //

        redisDb:dict -> dict:dict;

        dict:alphabet -> a;
        dict:message -> message;

    }


更新键
^^^^^^^^^^

对一个数据库键进行更新，
实际上就是对键空间里面键所对应的值对象进行更新，
根据值对象的类型不同，
更新的具体方法也会有所不同。

举个例子，
如果键空间当前的状态如图 IMAGE_DB_EXAMPLE 所示，
那么在执行以下命令之后：

::

    redis> SET message "blah blah"
    OK

键 ``message`` 的值对象将从之前包含 ``"hello world"`` 字符串更新为包含 ``"blah blah"`` 字符串，
如图 IMAGE_DB_UPDATE_CAUSE_SET 所示。

.. graphviz::

    digraph {

        label = "\n图 IMAGE_DB_UPDATE_CAUSE_SET    使用 SET 命令更新 message 键";

        rankdir = LR;

        node [shape = record];

        //

        redisDb [label = "redisDb | ... | <dict> dict | ..."];

        dict [label = "<dict> dict | <alphabet> StringObject \n \"alphabet\" | <book> StringObject \n \"book\" | <message> StringObject \n \"message\""];

        subgraph cluster_alphabet {

            a [label = " StringObject \n \"a\" "];
            b [label = " StringObject \n \"b\" "];
            c [label = " StringObject \n \"c\" "];

            a -> b -> c;

            label = "ListObject";

        }

        book [label = "<head> HashObject | <name> StringObject \n \"name\" | <author> StringObject \n \"author\" | <publisher> StringObject \n \"publisher\""];

        name [label = " StringObject \n \"Redis in Action\""];

        author [label = " StringObject \n \"Josiah L. Carlson\""];

        publisher [label = " StringObject \n \"Manning\""];

        message [label = " StringObject \n \"blah blah\""];

        //

        redisDb:dict -> dict:dict;

        dict:alphabet -> a;
        dict:book -> book:head;
        dict:message -> message;

        book:name -> name;
        book:publisher -> publisher;
        book:author -> author;

        //

        node [shape = plaintext]

        update [label = "更新值对象"]

        update -> message [style = dashed]

    }

再举个例子，
如果我们继续执行以下命令：

::

    redis> HSET book page 320
    (integer) 1

那么键空间中 ``book`` 键的值对象（一个哈希对象）将被更新，
新的键值对 ``page`` 和 ``320`` 会被添加到值对象里面，
如图 IMAGE_UPDATE_BY_HSET 所示。

.. graphviz::

    digraph {

        label = "\n图 IMAGE_UPDATE_BY_HSET    使用 HSET 更新 book 键";

        rankdir = LR;

        node [shape = record];

        //

        redisDb [label = "redisDb | ... | <dict> dict | ..."];

        dict [label = "<dict> dict | <alphabet> StringObject \n \"alphabet\" | <book> StringObject \n \"book\" | <message> StringObject \n \"message\" "];

        subgraph cluster_alphabet {

            a [label = " StringObject \n \"a\" "];
            b [label = " StringObject \n \"b\" "];
            c [label = " StringObject \n \"c\" "];

            a -> b -> c;

            label = "ListObject";

        }

        book [label = "<head> HashObject | <name> StringObject \n \"name\" | <author> StringObject \n \"author\" | <publisher> StringObject \n \"publisher\" | <page> StringObject \n \"page\" "];

        name [label = " StringObject \n \"Redis in Action\""];

        author [label = " StringObject \n \"Josiah L. Carlson\""];

        publisher [label = " StringObject \n \"Manning\""];

        page [label = " StringObject \n 320"];

        message [label = " StringObject \n \"blah blah\""];

        //

        redisDb:dict -> dict:dict;

        dict:alphabet -> a;
        dict:book -> book:head;
        dict:message -> message;

        book:name -> name;
        book:publisher -> publisher;
        book:author -> author;
        book:page -> page;

        //

        node [shape = plaintext]

        update [label = "新添加"]

        update -> book:page [style = dashed]

    }


对键取值
^^^^^^^^^^^^

对一个数据库键进行取值，
实际上就是在键空间中取出键所对应的值对象，
根据值对象的类型不同，
具体的取值方法也会有所不同。

举个例子，
如果键空间当前的状态如图 IMAGE_DB_EXAMPLE 所示，
那么当执行以下命令时：

::

    redis> GET message
    "hello world"

:ref:`GET` 命令将首先在键空间中查找键 ``message`` ，
找到键之后接着取得该键所对应的字符串对象值，
之后再返回值对象所包含的字符串 ``"hello world"`` ，
取值过程如图 IMAGE_FETCH_VALUE_VIA_GET 所示。

.. graphviz::

    digraph {

        label = "\n图 IMAGE_FETCH_VALUE_VIA_GET    使用 GET 命令取值的过程";

        rankdir = LR;

        node [shape = record];

        //

        redisDb [label = "redisDb | ... | <dict> dict | ..."];

        dict [label = "<dict> dict | <alphabet> StringObject \n \"alphabet\" | <book> StringObject \n \"book\" | <message> StringObject \n \"message\""];

        subgraph cluster_alphabet {

            a [label = " StringObject \n \"a\" "];
            b [label = " StringObject \n \"b\" "];
            c [label = " StringObject \n \"c\" "];

            a -> b -> c;

            label = "ListObject";

        }

        book [label = "<head> HashObject | <name> StringObject \n \"name\" | <author> StringObject \n \"author\" | <publisher> StringObject \n \"publisher\""];

        name [label = " StringObject \n \"Redis in Action\""];

        author [label = " StringObject \n \"Josiah L. Carlson\""];

        publisher [label = " StringObject \n \"Manning\""];

        message [label = " StringObject \n \"hello world\""];

        get [label = "GET", shape = plaintext];

        //

        redisDb:dict -> dict:dict;

        dict:alphabet -> a;
        dict:book -> book:head;
        dict:message -> message:head [label = "2）取值", style = dashed];

        book:name -> name;
        book:publisher -> publisher;
        book:author -> author;

        get -> dict:message [label = "1）查找键", style = dashed];

    }

再举一个例子，
当执行以下命令时：

::

    redis> LRANGE alphabet 0 -1
    1) "a"
    2) "b"
    3) "c"

:ref:`LRANGE` 命令将首先在键空间中查找键 ``alphabet`` ，
找到键之后接着取得该键所对应的列表对象值，
之后再返回列表对象中包含的三个字符串对象的值，
取值过程如图 IMAGE_FETCH_VALUE_VIA_LRANGE 所示。

.. graphviz::

    digraph {

        label = "\n图 IMAGE_FETCH_VALUE_VIA_LRANGE    使用 LRANGE 命令取值的过程";

        rankdir = LR;

        node [shape = record];

        //

        redisDb [label = "redisDb | ... | <dict> dict | ..."];

        dict [label = "<dict> dict | <alphabet> StringObject \n \"alphabet\" | <book> StringObject \n \"book\" | <message> StringObject \n \"message\""];

        subgraph cluster_alphabet {

            a [label = " StringObject \n \"a\" "];
            b [label = " StringObject \n \"b\" "];
            c [label = " StringObject \n \"c\" "];

            a -> b -> c [style = dashed];

            label = "ListObject";

        }

        book [label = "<head> HashObject | <name> StringObject \n \"name\" | <author> StringObject \n \"author\" | <publisher> StringObject \n \"publisher\""];

        name [label = " StringObject \n \"Redis in Action\""];

        author [label = " StringObject \n \"Josiah L. Carlson\""];

        publisher [label = " StringObject \n \"Manning\""];

        message [label = " StringObject \n \"hello world\""];

        lrange [label = "LRANGE", shape = plaintext];

        //

        redisDb:dict -> dict:dict;

        dict:alphabet -> a [label = "2）取值", style = dashed];
        dict:book -> book:head;
        dict:message -> message:head;

        book:name -> name:head;
        book:publisher -> publisher:head;
        book:author -> author:head;

        lrange -> dict:alphabet [label = "1）查找键", style = dashed];

    }

..
    关于取值要处理的另外两种情况是：

    - 如果要取值的键并不存在于键空间中，
      那么服务器将向发送取值命令的客户端返回一个空回复。

    - 如果要取值的键存在于键空间中，
      但该键的类型和取值命令所能处理的键的类型不相同，
      那么服务器将向发送取值命令的客户端返回一个类型错误。

    比如说，
    如果我们试图对一些不存在的键进行取值，
    那么服务器将返回空回复：

    ::

        redis> GET not_exists_key
        (nil)

        redis> LRANGE another_not_exists_key 0 -1
        (empty list or set)

    返回空回复是因为 :ref:`GET` 命令和 :ref:`LRANGE` 命令在键空间中都没有找到命令所指示的键，
    所以它们只能向客户端返回空回复。

    另外，
    如果我们试图用 :ref:`LRANGE` 命令取出字符串键 ``message`` 的值，
    那么服务器将返回一个类型错误：

    ::

        redis> LRANGE message 0 -1
        (error) WRONGTYPE Operation against a key holding the wrong kind of value

    因为 :ref:`LRANGE` 命令虽然能找到 ``message`` 键，
    但它却发现该键的值是自己所不能处理的字符串对象，
    所以 :ref:`LRANGE` 命令向客户端返回一个类型错误。


其他键空间操作
^^^^^^^^^^^^^^^^^^

除了上面列出的添加、删除、更新、取值操作之外，
还有很多针对数据库本身的 Redis 命令，
也是通过对键空间进行处理来完成的。

比如说，
用于清空整个数据库的 :ref:`FLUSHDB` 命令，
就是通过删除键空间中的所有键值对来实现的。

又比如说，
用于随机返回数据库中某个键的 :ref:`RANDOMKEY` 命令，
就是通过在键空间中随机返回一个键来实现的。

另外，
用于返回数据库键数量的 :ref:`DBSIZE` 命令，
就是通过返回键空间中包含键值对的数量来实现的。

类似的命令还有 :ref:`EXISTS` 、 :ref:`RENAME` 、 :ref:`KEYS` ，
等等，
这些命令都是通过对键空间进行操作来实现的。


读写键空间时的维护操作
^^^^^^^^^^^^^^^^^^^^^^^^^^

当使用 Redis 命令对数据库进行读写时，
服务器不仅会对键空间执行指定的读写操作，
还会执行一些额外的维护操作，
其中包括：

- 在读取一个键之后（读操作和写操作都要对键进行读取），
  服务器会根据键是否存在，
  以此来更新服务器的键空间命中（hit）次数或键空间不命中（miss）次数，
  这两个值可以在 :ref:`INFO stats <info>` 命令的 ``keyspace_hits`` 属性和 ``keyspace_misses`` 属性中查看。

- 在读取一个键之后，
  服务器会更新键的 LRU （最后一次使用）时间，
  这个值可以用于计算键的闲置时间，
  使用命令 :ref:`OBJECT idletime \<key\> <OBJECT>` 命令可以查看键 ``key`` 的闲置时间。

- 如果服务器在读取一个键时，
  发现该键已经过期，
  那么服务器会先删除这个过期键，
  然后才执行余下的其他操作，
  本章稍后对过期键的讨论会详细说明这一点。

- 如果有客户端使用 :ref:`WATCH` 命令监视了某个键，
  那么服务器在对被监视的键进行修改之后，
  会将这个键标记为脏（dirty），
  从而让事务程序注意到这个键已经被修改过，
  《事务》一章会详细说明这一点。
  
- 服务器每次修改一个键之后，
  都会对脏（dirty）键计数器的值增一，
  这个计数器会触发服务器的持久化以及复制操作执行，
  《RDB 持久化》、《AOF 持久化》和《复制》这三章都会说到这一点。

- 如果服务器开启了数据库通知功能，
  那么在对键进行修改之后，
  服务器将按配置发送相应的数据库通知，
  本章稍后讨论数据库通知功能的实现时会详细说明这一点。
