重点回顾
------------

- Redis 服务器在启动时，
  会对内嵌的 Lua 环境执行一系列修改操作，
  从而确保内嵌的 Lua 环境可以满足 Redis 在功能性、安全性等方面的需要。

- Redis 服务器专门使用一个伪客户端来执行 Lua 脚本中包含的 Redis 命令。

- Redis 使用脚本字典来保存所有被 :ref:`EVAL` 命令执行过，
  或者被 :ref:`SCRIPT_LOAD` 命令载入过的 Lua 脚本，
  这些脚本可以用于实现 :ref:`SCRIPT_EXISTS` 命令，
  以及实现脚本复制功能。

- :ref:`EVAL` 命令为客户端输入的脚本在 Lua 环境中定义一个函数，
  并通过调用这个函数来执行脚本。

- :ref:`EVALSHA` 命令通过直接调用 Lua 环境中已定义的函数来执行脚本。

- :ref:`SCRIPT_FLUSH` 命令会清空服务器 ``lua_scripts`` 字典中保存的脚本，
  并重置 Lua 环境。

- :ref:`SCRIPT_EXISTS` 命令接受一个或多个 SHA1 校验和为参数，
  并通过检查 ``lua_scripts`` 字典来确认校验和对应的脚本是否存在。

- :ref:`SCRIPT_LOAD` 命令接受一个 Lua 脚本为参数，
  为该脚本在 Lua 环境中创建函数，
  并将脚本保存到 ``lua_scripts`` 字典中。

- 服务器在执行脚本之前，
  会为 Lua 环境设置一个超时处理钩子，
  当脚本出现超时运行情况时，
  客户端可以通过向服务器发送 :ref:`SCRIPT_KILL` 命令来让钩子停止正在执行的脚本，
  或者发送 :ref:`SHUTDOWN nosave <SHUTDOWN>` 命令来让钩子关闭整个服务器。

- 主服务器复制 :ref:`EVAL` 、 :ref:`SCRIPT_FLUSH` 、 :ref:`SCRIPT_LOAD` 三个命令的方法和复制普通 Redis 命令一样 ——
  只要将相同的命令传播给从服务器就可以了。

- 主服务器在复制 :ref:`EVALSHA` 命令时，
  必须确保所有从服务器都已经载入了 :ref:`EVALSHA` 命令指定的 SHA1 校验和所对应的 Lua 脚本，
  如果不能确保这一点的话，
  主服务器会将 :ref:`EVALSHA` 命令转换成等效的 :ref:`EVAL` 命令，
  并通过传播 :ref:`EVAL` 命令来获得相同的脚本执行效果。
