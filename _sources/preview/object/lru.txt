对象的空转时长
-------------------------------

除了前面介绍过的 ``type`` 、 ``encoding`` 、 ``ptr`` 和 ``refcount`` 四个属性之外，
``redisObject`` 结构包含的最后一个属性为 ``lru`` 属性，
该属性记录了对象最后一次被命令程序访问的时间：

::

    typedef struct redisObject {

        // ...

        unsigned lru:22;

        // ...

    } robj;

:ref:`OBJECT IDLETIME <OBJECT>` 命令可以打印出给定键的空转时长，
这一空转时长就是通过将当前时间减去键的值对象的 ``lru`` 时间计算得出的：

::

    redis> SET msg "hello world"
    OK

    # 等待一小段时间
    redis> OBJECT IDLETIME msg
    (integer) 20

    # 等待一阵子
    redis> OBJECT IDLETIME msg
    (integer) 180

    # 访问 msg 键的值
    redis> GET msg
    "hello world"

    # 键处于活跃状态，空转时长为 0
    redis> OBJECT IDLETIME msg
    (integer) 0

.. topic:: 注意

    :ref:`OBJECT IDLETIME <OBJECT>` 命令的实现是特殊的，
    这个命令在访问键的值对象时，
    不会修改值对象的 ``lru`` 属性。

除了可以被 :ref:`OBJECT IDLETIME <OBJECT>` 命令打印出来之外，
键的空转时长还有另外一项作用：
如果服务器打开了 ``maxmemory`` 选项，
并且服务器用于回收内存的算法为 ``volatile-lru`` 或者 ``allkeys-lru`` ，
那么当服务器占用的内存数超过了 ``maxmemory`` 选项所设置的上限值时，
空转时长较高的那部分键会优先被服务器释放，
从而回收内存。

配置文件的 ``maxmemory`` 选项和 ``maxmemory-policy`` 选项的说明介绍了关于这方面的更多信息。
