简单动态字符串
===========================

Redis 没有直接使用 C 语言传统的字符串表示（以空字符结尾的字符数组，以下简称 C 字符串），
而是自己构建了一种名为简单动态字符串（simple dynamic string，SDS）的抽象类型，
并将 SDS 用作 Redis 的默认字符串表示。

在 Redis 里面，
C 字符串只会作为字符串字面量（string literal），
用在一些无须对字符串值进行修改的地方，
比如打印日志：

::

    redisLog(REDIS_WARNING,"Redis is now ready to exit, bye bye...");

当 Redis 需要的不仅仅是一个字符串字面量，
而是一个可以被修改的字符串值时，
Redis 就会使用 SDS 来表示字符串值：
比如在 Redis 的数据库里面，
包含字符串值的键值对在底层都是由 SDS 实现的。

举个例子，
如果客户端执行命令：

::

    redis> SET msg "hello world"
    OK

那么 Redis 将在数据库中创建了一个新的键值对，
其中：

- 键值对的键是一个字符串对象，
  对象的底层实现是一个保存着字符串 ``"msg"`` 的 SDS 。

- 键值对的值也是一个字符串对象，
  对象的底层实现是一个保存着字符串 ``"hello world"`` 的 SDS 。

又比如说，
如果客户端执行命令：

::

    redis> RPUSH fruits "apple" "banana" "cherry"
    (integer) 3

那么 Redis 将在数据库中创建一个新的键值对，
其中：

- 键值对的键是一个字符串对象，
  对象的底层实现是一个保存了字符串 ``"fruits"`` 的 SDS 。

- 键值对的值是一个列表对象，
  列表对象包含了三个字符串对象，
  这三个字符串对象分别由三个 SDS 实现：
  第一个 SDS 保存着字符串 ``"apple"`` ，
  第二个 SDS 保存着字符串 ``"banana"`` ，
  第三个 SDS 保存着字符串 ``"cherry"`` 。

除了用来保存数据库中的字符串值之外，
SDS 还被用作缓冲区（buffer）：
AOF 模块中的 AOF 缓冲区，
以及客户端状态中的输入缓冲区，
都是由 SDS 实现的，
在之后介绍 AOF 持久化和客户端状态的时候，
我们会看到 SDS 在这两个模块中的应用。

本章接下来将对 SDS 的实现进行介绍，
说明 SDS 和 C 字符串的不同之处，
解释为什么 Redis 要使用 SDS 而不是 C 字符串，
并在本章的最后列出 SDS 的操作 API 。
