频道的订阅与退订
---------------------------

当一个客户端执行 :ref:`SUBSCRIBE` 命令，
订阅某个或某些频道的时候，
这个客户端与被订阅频道之间就建立起了一种订阅关系。

Redis 将所有频道的订阅关系都保存在服务器状态的 ``pubsub_channels`` 字典里面，
这个字典的键是某个被订阅的频道，
而键的值则是一个链表，
链表里面记录了所有订阅这个频道的客户端：

::

    struct redisServer {

        // ...

        // 保存所有频道的订阅关系
        dict *pubsub_channels;

        // ...

    };

比如说，
图 IMAGE_PUBSUB_CHANNELS 就展示了一个 ``pubsub_channels`` 字典示例，
这个字典记录了以下信息：

- ``client-1`` 、 ``client-2`` 、 ``client-3`` 三个客户端正在订阅 ``"news.it"`` 频道。

- 客户端 ``client-4`` 正在订阅 ``"news.sport"`` 频道。

- ``client-5`` 和 ``client-6`` 两个客户端正在订阅 ``"news.business"`` 频道。

.. graphviz::

    digraph {

        label = "\n 图 IMAGE_PUBSUB_CHANNELS    一个 pubsub_channels 字典示例";

        rankdir = LR;

        //

        node [shape = record];

        pubsub_channels [label = " pubsub_channels | <news_it> \"news.it\" | <news_sport> \"news.sport\" | <news_business> \"news.business\" ", height = 3, width = 2.2];

        client_1 [label = "client-1"];
        client_2 [label = "client-2"];
        client_3 [label = "client-3"];
        client_4 [label = "client-4"];
        client5 [label = "client-5"];
        client6 [label = "client-6"];

        //

        pubsub_channels:news_it -> client_1; client_1 -> client_2; client_2 -> client_3;

        pubsub_channels:news_sport -> client_4;

        pubsub_channels:news_business -> client5 -> client6;

    }


订阅频道
^^^^^^^^^^^

每当客户端执行 :ref:`SUBSCRIBE` 命令，
订阅某个或某些频道的时候，
服务器都会将客户端与被订阅的频道在 ``pubsub_channels`` 字典中进行关联。

根据频道是否已经有其他订阅者，
关联操作分为两种情况执行：

- 如果频道已经有其他订阅者，
  那么它在 ``pubsub_channels`` 字典中必然有相应的订阅者链表，
  程序唯一要做的就是将客户端添加到订阅者链表的末尾。

- 如果频道还未有任何订阅者，
  那么它必然不存在于 ``pubsub_channels`` 字典，
  程序首先要在 ``pubsub_channels`` 字典中为频道创建一个键，
  并将这个键的值设置为空链表，
  然后再将客户端添加到链表，
  成为链表的第一个元素。

举个例子，
假设服务器 ``pubsub_channels`` 字典的当前状态如图 IMAGE_PUBSUB_CHANNELS 所示，
那么当客户端 ``client-10086`` 执行命令：

::

    SUBSCRIBE "news.sport" "news.movie"

之后，
``pubsub_channels`` 字典将更新至图 IMAGE_AFTER_SUBSCRIBE 所示的状态，
其中用虚线包围的是新添加的节点：

- 更新后的 ``pubsub_channels`` 字典新增了 ``"news.movie"`` 键，
  该键对应的链表值只包含一个 ``client-10086`` 节点，
  表示目前只有 ``client-10086`` 一个客户端在订阅 ``"news.movie"`` 频道。

- 至于原本就已经有客户端在订阅的 ``"news.sport"`` 频道，
  ``client-10086`` 的节点放在了频道对应链表的末尾，
  排在 ``client-4`` 节点的后面。

.. graphviz::

    digraph {

        label = "\n 图 IMAGE_AFTER_SUBSCRIBE    执行 SUBSCRIBE 之后的 pubsub_channels 字典";

        rankdir = LR;

        //

        node [shape = record];

        pubsub_channels [label = "pubsub_channels | <news_it> \"news.it\" | <news_sport> \"news.sport\" | <news_business> \"news.business\" | <news_movie> \"news.movie\" ", height = 4, width = 2.2];

        client_1 [label = "client-1"];
        client_2 [label = "client-2"];
        client_3 [label = "client-3"];
        client_4 [label = "client-4"];
        client5 [label = "client-5"];
        client6 [label = "client-6"];
        sport_client123 [label = "client-10086", style = dashed];
        movies_client123 [label = "client-10086", style = dashed];

        //

        pubsub_channels:news_it -> client_1; client_1 -> client_2; client_2 -> client_3;

        pubsub_channels:news_sport -> client_4 -> sport_client123;

        pubsub_channels:news_business -> client5 -> client6;

        pubsub_channels:news_movie -> movies_client123;

    }

:ref:`SUBSCRIBE` 命令的实现可以用以下伪代码来描述：

.. code-block:: python

    def subscribe(*all_input_channels):

        # 遍历输入的所有频道
        for channel in all_input_channels:

            # 如果 channel 不存在于 pubsub_channels 字典（没有任何订阅者）
            # 那么在字典中添加 channel 键，并设置它的值为空链表
            if channel not in server.pubsub_channels:
                server.pubsub_channels[channel] = []

            # 将订阅者添加到频道所对应的链表的末尾
            server.pubsub_channels[channel].append(client)

.. ** fixing editor highlight


退订频道
^^^^^^^^^^^^^^^

:ref:`UNSUBSCRIBE` 命令的行为和 :ref:`SUBSCRIBE` 命令的行为正好相反 ——
当一个客户端退订某个或某些频道的时候，
服务器将从 ``pubsub_channels`` 中解除客户端与被退订频道之间的关联：

- 程序会根据被退订频道的名字，
  在 ``pubsub_channels`` 字典中找到频道对应的订阅者链表，
  然后从订阅者链表中删除退订客户端的信息。

- 如果删除退订客户端之后，
  频道的订阅者链表变成了空链表，
  那么说明这个频道已经没有任何订阅者了，
  程序将从 ``pubsub_channels`` 字典中删除频道对应的键。

举个例子，
假设 ``pubsub_channels`` 的当前状态如图 IMAGE_BEFORE_UNSUBSCRIBE 所示，
那么当客户端 ``client-10086`` 执行命令：

::

    UNSUBSCRIBE "news.sport" "news.movie"

之后，
图中用虚线包围的两个节点将被删除，
如图 IMAGE_AFTER_UNSUBSCRIBE 所示：

- 在 ``pubsub_channels`` 字典更新之后，
  ``client-10086`` 的信息已经从 ``"news.sport"`` 频道和 ``"news.movie"`` 频道的订阅者链表中被删除了。

- 另外，
  因为删除 ``client-10086`` 之后，
  频道 ``"news.movie"`` 已经没有任何订阅者，
  因此键 ``"news.movie"`` 也从字典中被删除了。

.. graphviz::

    digraph {

        label = "\n 图 IMAGE_BEFORE_UNSUBSCRIBE    执行 UNSUBSCRIBE 之前的 pubsub_channels 字典";

        rankdir = LR;

        //

        node [shape = record];

        pubsub_channels [label = " pubsub_channels | <news_it> \"news.it\" | <news_sport> \"news.sport\" | <news_business> \"news.business\" | <news_movie> \"news.movie\" ", height = 4, width = 2.2];

        client_1 [label = "client-1"];
        client_2 [label = "client-2"];
        client_3 [label = "client-3"];
        client_4 [label = "client-4"];
        client5 [label = "client-5"];
        client6 [label = "client-6"];
        sport_client123 [label = "client-10086", style = dashed];
        movies_client123 [label = "client-10086", style = dashed];

        //

        pubsub_channels:news_it -> client_1; client_1 -> client_2; client_2 -> client_3;

        pubsub_channels:news_sport -> client_4 -> sport_client123;

        pubsub_channels:news_business -> client5 -> client6;

        pubsub_channels:news_movie -> movies_client123;

    }

.. graphviz::

    digraph {

        label = "\n 图 IMAGE_AFTER_UNSUBSCRIBE    执行 UNSUBSCRIBE 之后的 pubsub_channels 字典";

        rankdir = LR;

        //

        node [shape = record];

        pubsub_channels [label = " pubsub_channels | <news_it> \"news.it\" | <news_sport> \"news.sport\" | <news_business> \"news.business\" ", height = 3, width = 2.2];

        client_1 [label = "client-1"];
        client_2 [label = "client-2"];
        client_3 [label = "client-3"];
        client_4 [label = "client-4"];
        client5 [label = "client-5"];
        client6 [label = "client-6"];

        //

        pubsub_channels:news_it -> client_1; client_1 -> client_2; client_2 -> client_3;

        pubsub_channels:news_sport -> client_4;

        pubsub_channels:news_business -> client5 -> client6;

    }


:ref:`UNSUBSCRIBE` 命令的实现可以用以下伪代码来描述：

.. code-block:: python

    def unsubscribe(*all_input_channels):

        # 遍历要退订的所有频道
        for channel in all_input_channels:

            # 在订阅者链表中删除退订的客户端
            server.pubsub_channels[channel].remove(client)

            # 如果频道已经没有任何订阅者了（订阅者链表为空）
            # 那么将频道从字典中删除
            if len(server.pubsub_channels[channel]) == 0:
                server.pubsub_channels.remove(channel)

.. ** editor highlight fixing
