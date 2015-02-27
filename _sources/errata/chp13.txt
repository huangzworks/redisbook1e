第 13 章《客户端》勘误
==============================

168 页
----------

图 13-5 中错误地包含了两个 ``argv[1]`` ，
第二个 ``argv[1]`` 应为 ``argv[2]`` ，
以下是修正后的图片：

.. graphviz::

    digraph {

        label = "\n 图 13-5    argv 属性和 argc 属性示例";

        rankdir = LR;

        node [shape = record];

        redisClient [label = " redisClient | ... | <argv> argv | argc \n 3 | ... ", width = 2];

        argv [label = " { { <head> argv[0] | StringObject \n \"SET\" } | { argv[1] | StringObject \n \"key\" } | { argv[2] | StringObject \n \"value\" } } "];

        redisClient:argv -> argv:head;

    }

感谢 凯旋冲锋 反馈这个错误。



173 页
-----------

在 13.2.2 节，
客户端被关闭的原因中的第 4 点：

    如果用户为服务器设置了 ``timeout`` 配置选项，
    那么当客户端的空转时间超过 ``timeout`` 选项设置的值时，
    客户端将被关闭。
    不过 ``timeout`` 选项有一些例外情况：
    **如果客户端是主服务器（打开了** ``REDIS_MASTER`` **标志），
    从服务器（打开了** ``REDIS_SLAVE`` **标志），
    正在被** ``BLPOP`` **等命令阻塞（打开了** ``REDIS_BLOCKED`` **标志）**\ ，
    或者正在执行 ``SUBSCRIBE`` 、 ``PSUBSCRIBE`` 等订阅命令，
    那么即使客户端的空转时间超过了 ``timeout`` 选项的值，
    客户端也不会被服务器关闭。

其中加粗部分的各个子句应该使用顿号而不是句号来进行分割，
改正后的句子为：

    如果用户为服务器设置了 ``timeout`` 配置选项，
    那么当客户端的空转时间超过 ``timeout`` 选项设置的值时，
    客户端将被关闭。
    不过 ``timeout`` 选项有一些例外情况：
    **如果客户端是主服务器（打开了** ``REDIS_MASTER`` **标志）、
    从服务器（打开了** ``REDIS_SLAVE`` **标志）、
    正在被** ``BLPOP`` **等命令阻塞（打开了** ``REDIS_BLOCKED`` **标志）**\ ，
    或者正在执行 ``SUBSCRIBE`` 、 ``PSUBSCRIBE`` 等订阅命令，
    那么即使客户端的空转时间超过了 ``timeout`` 选项的值，
    客户端也不会被服务器关闭。

一个更好的办法是修改一下整个段落的表现形式，
让各个例外条件变得更清晰一些：

    如果用户为服务器设置了 ``timeout`` 配置选项，
    那么当客户端的空转时间超过 ``timeout`` 选项设置的值时，
    客户端将被关闭。
    不过 ``timeout`` 选项有一些例外情况 ——
    如果客户端满足以下条件中的任意一种：

    1. 客户端是一个主服务器（该客户端的 ``REDIS_MASTER`` 标志处于打开状态）；
    2. 客户端是一个从服务器（该客户端的 ``REDIS_SLAVE`` 标志处于打开状态）；
    3. 客户端正在被 ``BLPOP`` 等命令阻塞（该客户端的 ``REDIS_BLOCKED`` 标志处于打开状态）；
    4. 客户端正在执行 ``SUBSCRIBE`` 、 ``PSUBSCRIBE`` 等订阅命令；

    那么即使客户端的空转时间超过了 ``timeout`` 选项的值，
    客户端也不会被服务器关闭。

感谢 `不装嫩就会死的尹大侠 <http://weibo.com/u/2714840873>`_ 反馈这个错误。
