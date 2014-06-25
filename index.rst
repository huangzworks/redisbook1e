.. Redis 设计与实现 documentation master file, created by
   sphinx-quickstart on Fri Apr 18 21:53:39 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Redis 设计与实现
=======================================

.. image:: image/cover.jpg
   :align: left
   :scale: 55%

欢迎来到《Redis 设计与实现》的支持网站！

《Redis 设计与实现》一书全面而完整地讲解了 Redis 的内部运行机制，
对 Redis 的大多数单机功能以及所有多机功能的实现原理进行了介绍，
展示了这些功能的核心数据结构以及关键的算法思想。
通过阅读本书，
读者可以快速、有效地了解 Redis 的内部构造以及运作机制，
从而学会如何更高效地使用 Redis 。

你可以在 `互动出版网（china-pub） <http://product.china-pub.com/3770218>`_ 、
`京东商城 <http://item.jd.com/11486101.html>`_ 、
`亚马逊 <http://www.amazon.cn/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8A%80%E6%9C%AF%E4%B8%9B%E4%B9%A6-Redis%E8%AE%BE%E8%AE%A1%E4%B8%8E%E5%AE%9E%E7%8E%B0-%E9%BB%84%E5%81%A5%E5%AE%8F/dp/B00L4XHH0S>`_ 或者
`当当网 <http://product.dangdang.com/23501734.html>`_ 购买本书。

另外，
本书还提供了作者签名版可供购买，
请访问 :doc:`signed` 页面了解更多信息。

你可以通过访问本站，
或者关注本书作者的\ `微博 <http://weibo.com/huangz1990>`_\ 、\ `twitter <https://twitter.com/huangz1990>`_\ 和\ `豆瓣 <http://www.douban.com/people/i_m_huangz/>`_\ 来获知本书的最新消息。


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

请访问 :doc:`toc` 页面来查看本书的详细目录，
并试读公开的章节。


相关资源
-----------------

Redis 3.0 源码注释 —— 
包含中文注释的 Redis 3.0 源码，
帮助有兴趣的读者深入了解 Redis 的实现细节。
即将公开，
敬请期待。

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

以下列出的是本书已确认的勘误信息：

.. toctree::
   :maxdepth: 1

   errata/chp4
   errata/chp6

读者可以在阅读本书之前，
根据这些勘误对书本内容进行校正，
给你带来的不便作者深感抱歉。
