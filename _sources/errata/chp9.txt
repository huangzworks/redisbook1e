第 9 章《数据库》勘误
===============================

91 页
----------

在 9.2 节第一段代码示例中：

::

    redis [2] SET msg"another world"
    OK

``msg`` 和 ``"another world"`` 之间应该有一个空格，
正确的代码为：

::

    redis [2] SET msg "another world"
    OK

感谢 `zionwu <http://book.douban.com/people/zionwu/>`_ 反馈这个错误。


103 页
-----------

在此页末尾的 ``PEXPIREAT`` 函数中：

.. code-block:: python

    def PEXPIREAT(key, expire_time_in_ms):

        # ...

        if key not in redisDb.dict:
            return0

        # ...

``return`` 和 ``0`` 之间应该有一个空格，
正确的代码为：

.. code-block:: python

    if key not in redisDb.dict:
        return 0

感谢 `zionwu <http://book.douban.com/people/zionwu/>`_ 反馈这个错误。
