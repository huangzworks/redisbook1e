.. Redis 设计与实现 documentation master file, created by
   sphinx-quickstart on Fri Apr 18 21:53:39 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Redis 设计与实现（新版）
=======================================

.. image:: image/cover.jpg
   :align: left
   :scale: 55%

欢迎访问新版《Redis 设计与实现》支持网站！

新版《Redis 设计与实现》对\ `旧版《Redis 设计与实现》 <http://origin.redisbook.com>`_\ 进行了完全重写，
并添加了包括复制、Sentinel、集群、二进制位操作和排序在内的众多新主题，
内容覆盖 Redis 2.6 至 Redis 3.0 
（具体细节请查看 :doc:`different`\ ）。


新版《Redis 设计与实现》将由机械工业出版社出版，
**预计 2014 年 6 月发售。**

你可以通过访问本站，
或者关注本书作者的\ `微博 <http://weibo.com/huangz1990>`_\ 、\ `twitter <https://twitter.com/huangz1990>`_\ 和\ `豆瓣 <http://www.douban.com/people/i_m_huangz/>`_\ 来获知本书的最新消息。

新版《Redis 设计与实现》的豆瓣主页： http://book.douban.com/subject/25900156/


购买方法
----------------

《Redis 设计与实现》目前尚未开始销售，
请通过微博和 twitter 等渠道关注书本的最新消息。


查看目录并试读
-----------------

全书约有 400 页，分为 4 个部分，共 24 章。**以下目录中可点击的为试读内容。**

- 前言

- 致谢

1. 简介

  - 版本说明
  - 章节编排
  - 推荐的阅读方法
  - 行文规则
  - 配套网站

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

8. 对象

  - 对象的类型与编码
  - 字符串对象
  - 列表对象
  - 哈希对象
  - 集合对象
  - 有序集合对象
  - 类型检查与命令多态
  - 内存回收
  - 对象共享
  - 对象的空转时长
  - 重点回顾

**第二部分：单机数据库的实现**

9. 数据库
  
  - 服务器中的数据库
  - 切换数据库
  - 数据库键空间
  - 设置键的生存时间或过期时间
  - 过期键删除策略
  - Redis 的过期键删除策略
  - AOF 、RDB 和复制功能对过期键的处理
  - 数据库通知
  - 重点回顾

10. RDB 持久化

  - RDB 文件的创建与载入
  - 自动间隔性保存
  - RDB 文件结构
  - 分析 RDB 文件
  - 重点回顾
  - 参考资料

11. AOF 持久化

  - AOF 持久化的实现
  - AOF 文件的载入与数据还原
  - AOF 重写
  - 重点回顾

12. 事件

  - 文件事件
  - 时间事件
  - 事件的调度与执行
  - 重点回顾
  - 参考资料

13. 客户端

  - 客户端属性
  - 客户端的创建与关闭
  - 重点回顾

14. 服务器

  - 命令请求的执行过程
  - serverCron 函数
  - 初始化服务器
  - 重点回顾

**第三部分：多机数据库的实现**

15. 复制

  - 旧版复制功能的实现
  - 旧版复制功能的缺陷
  - 新版复制功能的实现
  - 部分重同步的实现
  - PSYNC 命令的实现
  - 复制的实现
  - 心跳检测
  - 重点回顾

16. Sentinel

  - 启动并初始化 Sentinel
  - 获取主服务器信息
  - 获取从服务器信息
  - 向主服务器和从服务器发送信息
  - 接收来自主服务器和从服务器的频道信息
  - 检测主观下线状态
  - 检查客观下线状态
  - 选举领头 Sentinel
  - 故障转移
  - 重点回顾
  - 参考资料

17. 集群

  - 节点
  - 槽指派
  - 在集群中执行命令
  - 重新分片
  - ASK 错误
  - 复制与故障转移
  - 消息
  - 重点回顾

**第四部分：独立功能的实现**

18. 发布与订阅

  - 频道的订阅与退订
  - 模式的订阅与退订
  - 发送消息
  - 查看订阅信息
  - 重点回顾
  - 参考资料

19. 事务

  - 事务的实现
  - WATCH 命令的实现
  - 事务的 ACID 性质
  - 重点回顾
  - 参考资料

20. Lua 脚本

  - 创建并修改 Lua 环境
  - Lua 环境协作组件
  - EVAL 命令的实现
  - EVALSHA 命令的实现
  - 脚本管理命令的实现
  - 脚本复制
  - 重点回顾
  - 参考资料

21. 排序

  - SORT <key> 命令的实现
  - ALPHA 选项的实现
  - ASC 选项和 DESC 选项的实现
  - BY 选项的实现
  - 带有 ALPHA 选项的 BY 选项的实现
  - LIMIT 选项的实现
  - GET 选项的实现
  - STORE 选项的实现
  - 多个选项的执行顺序
  - 重点回顾

22. 二进制位数组

  - 位数组的表示
  - GETBIT 命令的实现
  - SETBIT 命令的实现
  - BITCOUNT 命令的实现
  - BITOP 命令的实现
  - 重点回顾
  - 参考资料

23. 慢查询日志

  - 慢查询记录的保存
  - 慢查询日志的阅览和删除
  - 添加新日志
  - 重点回顾

24. 监视器

  - 成为监视器
  - 向监视器发送命令信息
  - 重点回顾
