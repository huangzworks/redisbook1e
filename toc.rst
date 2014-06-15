目录和试读
=================

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
  - 重点回顾
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
  - 重点回顾
  - :doc:`preview/event/review`

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
