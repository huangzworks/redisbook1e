重点回顾
-----------------------

- 服务器状态在 ``pubsub_channels`` 字典保存了所有频道的订阅关系：
  :ref:`SUBSCRIBE` 命令负责将客户端和被订阅的频道关联到这个字典里面，
  而 :ref:`UNSUBSCRIBE` 命令则负责解除客户端和被退订频道之间的关联。

- 服务器状态在 ``pubsub_patterns`` 链表保存了所有模式的订阅关系：
  :ref:`PSUBSCRIBE` 命令负责将客户端和被订阅的模式记录到这个链表中，
  而 :ref:`UNSUBSCRIBE` 命令则负责移除客户端和被退订模式在链表中的记录。

- :ref:`PUBLISH` 命令通过访问 ``pubsub_channels`` 字典来向频道的所有订阅者发送消息，
  通过访问 ``pubsub_patterns`` 链表来向所有匹配频道的模式的订阅者发送消息。

- :ref:`PUBSUB` 命令的三个子命令都是通过读取 ``pubsub_channels`` 字典和 ``pubsub_patterns`` 链表中的信息来实现的。
