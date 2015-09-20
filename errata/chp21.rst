第 22 章《排序》勘误
==========================

第 359 页
------------

执行 ``SORT students ALPHA STORE sorted_students`` 命令时步骤 7 描述有误，
命令返回的不是 ``"jack"`` 、 ``"peter"`` 、 ``"tom"`` 三个元素，
而是代表被储存元素数量的数字 ``3`` 。

请将：

    7) 遍历数组，向客户端返回 ``"jack"`` 、 ``"peter"`` 、 ``"tom"`` 三个元素。

改为：

    7) 向客户端返回 ``sorted_students`` 列表的长度 —— 3 ，表示共有三个已排序元素被储存到了指定的列表里面。

感谢 `comancheNet <http://weibo.com/comanchenet>`_ 反馈这个错误。
