第 4 章《字典》勘误
==========================

第 25 页
--------------

4.1.2 节第一个 ``dictEntry`` 结构的定义中，
联合（union）值 ``v`` 的定义应该是：

::

    typedef struct dictEntry {

        // ...

        union {
            void *val;
            uint64_t u64;
            int64_t s64;
        } v;

        // ...

    } dictEntry;

感谢 `jackli066519 <http://weibo.com/1601172061>`_ 反馈此错误。


第 26 页
-----------

4.1.3 节中的 ``dict`` 结构的定义中，
末尾的 ``rehashidx`` 属性的定义应该为：

::

    typedef struct dict {

        // ...

        int rehashidx;

    } dict;

感谢 `jackli066519 <http://weibo.com/1601172061>`_ 反馈此错误。
