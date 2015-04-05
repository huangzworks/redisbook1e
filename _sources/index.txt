.. Redis 设计与实现 documentation master file, created by
   sphinx-quickstart on Fri Apr 18 21:53:39 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Redis 设计与实现
=======================================

.. image:: image/cover.png
   :align: left

欢迎来到《Redis 设计与实现》的支持网站！

《Redis 设计与实现》一书全面而完整地讲解了 Redis 的内部运行机制，
对 Redis 的大多数单机功能以及所有多机功能的实现原理进行了介绍，
展示了这些功能的核心数据结构以及关键的算法思想。
通过阅读本书，
读者可以快速、有效地了解 Redis 的内部构造以及运作机制，
从而学会如何更高效地使用 Redis 。

你可以通过访问本站，
或者关注本书作者的\ `微博 <http://weibo.com/huangz1990>`_\ 、\ `twitter <https://twitter.com/huangz1990>`_\ 和\ `豆瓣 <http://www.douban.com/people/i_m_huangz/>`_\ 来获知本书的最新消息。

购买本书请访问：
`京东商城 <http://item.jd.com/11486101.html>`_ 、
`互动出版网（china-pub） <http://product.china-pub.com/3770218>`_ 、
`亚马逊 <http://www.amazon.cn/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF%E4%B8%9B%E4%B9%A6-Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0-%E9%BB%84%E5%81%A5%E5%AE%8F/dp/B00L4XHH0S>`_ 、
`当当网 <http://product.dangdang.com/23501734.html>`_ ，
另外本书的 `Kindle 版本 <http://www.amazon.cn/Redis/dp/B00LZNV5B4>`_ 、 `多看阅读版本 <http://www.duokan.com/book/53962>`_ 和 `豆瓣阅读版本 <http://read.douban.com/ebook/7519526/>`_ 也已有售。

..
    另外，
    本书还提供了作者签名版可供购买，
    请访问 :doc:`signed` 页面了解更多信息。



内容与特色介绍
-----------------

本书介绍了以下内容：

- 字符串（string）、散列（hash）、列表（list）、集合（set）和有序集合（sorted set）这五种类型的键的底层实现数据结构。
- Redis 的对象处理机制以及数据库的实现原理。
- 事务实现原理。
- 订阅与发布实现原理。
- Lua 脚本功能的实现原理。
- ``SORT`` 命令的实现原理。
- ``BITOP`` 、 ``BITCOUNT`` 等二进制位处理命令的实现原理。
- 慢查询日志的实现原理。
- RDB 持久化和 AOF 持久化的实现原理。
- Redis 事件处理器的实现原理。
- Redis 服务器和客户端的实现原理。
- 复制（replication）、Sentinel 和集群（cluster）这三个多机功能的实现原理。

本书的特色是：

- 带有丰富的图示和表格，
  帮助读者更好地理解书中的知识点。
- 关注功能的高层设计思路而不是底层的实现代码，
  让读者无须花时间研读代码就可以了解到 Redis 的内部实现。
- 提供带有中文注释的 Redis 源码，
  帮助有需要的读者做进一步的学习。


查看目录并试读
-----------------

《Redis 设计与实现》全书共有 388 页，分为 4 个部分，共 24 章。

**以下目录中可点击的为试读内容。**

- :doc:`preview/preface`

- :doc:`preview/ack`

1. :doc:`preview/introduction/content`

  - :ref:`intro_version`
  - :ref:`intro_chapters`
  - :ref:`intro_how_to_read`
  - :ref:`intro_rules`
  - :ref:`intro_site`

**第一部分：数据结构与对象**

2. :doc:`preview/sds/content`
  
  - :doc:`preview/sds/implementation`
  - :doc:`preview/sds/different_between_sds_and_c_string`
  - :doc:`preview/sds/api`
  - :doc:`preview/sds/review`
  - :doc:`preview/sds/reference`

3. :doc:`preview/adlist/content`

  - :doc:`preview/adlist/implementation`
  - :doc:`preview/adlist/api`
  - :doc:`preview/adlist/review`

4. :doc:`preview/dict/content`

  - :doc:`preview/dict/datastruct`
  - :doc:`preview/dict/hash_algorithm`
  - :doc:`preview/dict/collision_resolution`
  - :doc:`preview/dict/rehashing`
  - :doc:`preview/dict/incremental_rehashing`
  - :doc:`preview/dict/api`
  - :doc:`preview/dict/review`

5. :doc:`preview/skiplist/content`

  - :doc:`preview/skiplist/datastruct`
  - :doc:`preview/skiplist/api`
  - :doc:`preview/skiplist/review`

6. :doc:`preview/intset/content`

  - :doc:`preview/intset/datastruct`
  - :doc:`preview/intset/upgrade`
  - :doc:`preview/intset/why_upgrade`
  - :doc:`preview/intset/downgrade`
  - :doc:`preview/intset/api`
  - :doc:`preview/intset/review`

7. :doc:`preview/ziplist/content`

  - :doc:`preview/ziplist/list`
  - :doc:`preview/ziplist/node`
  - :doc:`preview/ziplist/cascade_update`
  - :doc:`preview/ziplist/api`
  - :doc:`preview/ziplist/review`

8. :doc:`preview/object/content`

 - :doc:`preview/object/object`
 - :doc:`preview/object/string`
 - :doc:`preview/object/list`
 - :doc:`preview/object/hash`
 - :doc:`preview/object/set`
 - :doc:`preview/object/sorted_set`
 - :doc:`preview/object/type_check`
 - :doc:`preview/object/refcount`
 - :doc:`preview/object/share_object`
 - :doc:`preview/object/lru`
 - :doc:`preview/object/review`

**第二部分：单机数据库的实现**

9. 数据库
  
  - 服务器中的数据库
  - 切换数据库
  - :doc:`preview/database/key_space`
  - 设置键的生存时间或过期时间
  - 过期键删除策略
  - Redis 的过期键删除策略
  - AOF 、RDB 和复制功能对过期键的处理
  - 数据库通知
  - :doc:`preview/database/review`

10. RDB 持久化

  - RDB 文件的创建与载入
  - 自动间隔性保存
  - :doc:`preview/rdb/rdb_struct`
  - 分析 RDB 文件
  - :doc:`preview/rdb/review`

11. AOF 持久化

  - :doc:`preview/aof/aof_implement`
  - AOF 文件的载入与数据还原
  - AOF 重写
  - :doc:`preview/aof/review`

12. 事件

  - :doc:`preview/event/file_event`
  - 时间事件
  - 事件的调度与执行
  - :doc:`preview/event/review`
  - :doc:`preview/event/reference`

13. 客户端

  - :doc:`preview/client/redis_client_property`
  - 客户端的创建与关闭
  - :doc:`preview/client/review`

14. 服务器

  - :doc:`preview/server/execute_command`
  - serverCron 函数
  - 初始化服务器
  - :doc:`preview/server/review`

**第三部分：多机数据库的实现**

15. 复制

  - :doc:`preview/replication/replicate-before-2-8`
  - 旧版复制功能的缺陷
  - 新版复制功能的实现
  - 部分重同步的实现
  - PSYNC 命令的实现
  - 复制的实现
  - 心跳检测
  - :doc:`preview/replication/review`

16. Sentinel

  - :doc:`preview/sentinel/init_sentinel`
  - 获取主服务器信息
  - 获取从服务器信息
  - 向主服务器和从服务器发送信息
  - 接收来自主服务器和从服务器的频道信息
  - 检测主观下线状态
  - 检查客观下线状态
  - 选举领头 Sentinel
  - 故障转移
  - :doc:`preview/sentinel/review`
  - :doc:`preview/sentinel/reference`

17. 集群

  - :doc:`preview/cluster/node`
  - 槽指派
  - 在集群中执行命令
  - 重新分片
  - ASK 错误
  - 复制与故障转移
  - 消息
  - :doc:`preview/cluster/review`

**第四部分：独立功能的实现**

18. 发布与订阅

  - :doc:`preview/pubsub/channel`
  - 模式的订阅与退订
  - 发送消息
  - 查看订阅信息
  - :doc:`preview/pubsub/review`
  - :doc:`preview/pubsub/reference`

19. 事务

  - :doc:`preview/transaction/transaction_implement`
  - WATCH 命令的实现
  - 事务的 ACID 性质
  - :doc:`preview/transaction/review`
  - :doc:`preview/transaction/reference`

20. Lua 脚本

  - :doc:`preview/script/init_lua_env`
  - Lua 环境协作组件
  - EVAL 命令的实现
  - EVALSHA 命令的实现
  - 脚本管理命令的实现
  - 脚本复制
  - :doc:`preview/script/review`
  - :doc:`preview/script/reference`

21. 排序

  - :doc:`preview/sort/sort_key`
  - ALPHA 选项的实现
  - ASC 选项和 DESC 选项的实现
  - BY 选项的实现
  - 带有 ALPHA 选项的 BY 选项的实现
  - LIMIT 选项的实现
  - GET 选项的实现
  - STORE 选项的实现
  - 多个选项的执行顺序
  - :doc:`preview/sort/review`

22. 二进制位数组

  - 位数组的表示
  - :doc:`preview/bit/getbit`
  - SETBIT 命令的实现
  - BITCOUNT 命令的实现
  - BITOP 命令的实现
  - :doc:`preview/bit/review`
  - :doc:`preview/bit/reference`

23. :doc:`preview/slowlog/content`

  - :ref:`slowlog_save`
  - :ref:`slowlog_view_and_delete`
  - :ref:`slowlog_add`
  - :ref:`slowlog_review`

24. :doc:`preview/monitor/content`

  - :doc:`preview/monitor/become_monitor`
  - :doc:`preview/monitor/propagate_command`
  - :doc:`preview/monitor/review`


注释源码
-----------------

为了帮助有需要的读者进一步了解 Redis 的实现细节，
本书附带了一份包含详细中文注释的 Redis 3.0 版本源码可供参考：
`https://github.com/huangz1990/redis-3.0-annotated <https://github.com/huangz1990/redis-3.0-annotated>`_ 。


相关资源
-----------------

`《如何阅读 Redis 源码》 <http://blog.huangz.me/diary/2014/how-to-read-redis-source-code.html>`_ ——
文章给出了一个推荐的 Redis 源码阅读顺序以供参考，
读者可以在阅读完本书之后，
根据文章描述的顺序来尝试阅读源码，
从而进一步提高对 Redis 的了解。

`《Redis 设计与实现》图片集 <http://1e-gallery.redisbook.com>`_ ——
展示了本书包含的绝大多数图片以及图片的源码，
方便读者在写博客、记笔记或者做演讲稿时引用本书的图片，
或者通过阅读图片的源码来学习 dot 语言和 Graphviz 图片生成工具。

`《Redis 多机特性工作原理简介》 <http://www.chinahadoop.cn/course/31>`_ ——
这个课程对 Redis 的复制、Sentinel 和集群三个特性的工作原理进行了基本的介绍。
因为课程的内容都提取自本书的《复制》、《Sentinel》和《集群》三个章节，
所以可以把这个课程看作是这三个章节的简介版本。

`旧版《Redis 设计与实现》 <http://origin.redisbook.com>`_ ——
本书的上一版，
介绍了 Redis 2.6 的内部运作机制和单机功能。
要了解本书和旧版之间的区别，
请阅读 :doc:`different` 页面。


勘误
-----------------

:doc:`errata/index` 页面列出了本书已确认的勘误信息，
请读者在阅读本书之前，
根据这些信息对书本进行校正，
由此带来的不便作者深感抱歉。

如果读者发现了勘误页面目前尚未记录的新错误，
可以在本页面的 disqus 论坛进行反馈，
又或者通过 `huangz.me <http://huangz.me>`_ 页面展示的任意一种联系方式来联系作者。
