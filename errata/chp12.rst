第 12 章《事件》勘误
============================

158 页
-----------

``processTimeEvents`` 函数的定义中，
``else`` 语句应该对齐的是：

::

    if retval == AE_NOMORE:

语句，
而不是：

::

    if time_event.when <= unix_ts_now()

语句，
修正后的定义应该为：

.. code-block:: python

    def processTimeEvents():

        # 遍历服务器中的所有时间事件
        for time_event in all_time_event():

            # 检查事件是否已经到达
            if time_event.when <= unix_ts_now():

                # 事件已到达
                # 执行事件处理器，并获取返回值
                retval = time_event.timeProc()

                # 如果这是一个定时事件
                if retval == AE_NOMORE:

                    # 那么将该事件从服务器中删除
                    delete_time_event_from_server(time_event)

                # 如果这是一个周期性事件
                else:

                    # 那么按照事件处理器的返回值更新时间事件的 when 属性
                    # 让这个事件在指定的时间之后再次到达
                    update_when(time_event, retval)

感谢 `zhkzyth <http://www.douban.com/people/zhkzyth/>`_ 反馈这个错误。
