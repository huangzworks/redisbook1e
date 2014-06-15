重点回顾
------------

- Redis 使用 SDS 来保存位数组。

- SDS 使用逆序来保存位数组，
  这种保存顺序简化了 :ref:`SETBIT` 命令的实现，
  使得 :ref:`SETBIT` 命令可以在不移动现有二进制位的情况下，
  对位数组进行空间扩展。

- :ref:`BITCOUNT` 命令使用了查表算法和 variable-precision SWAR 算法来优化命令的执行效率。

- :ref:`BITOP` 命令的所有操作都使用 C 语言内置的位操作来实现。
