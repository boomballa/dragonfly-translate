# 使用 Lua 编写脚本
Dragonfly 允许用户执行用[Lua](https://lua.org/)编写的脚本。[它用于管理和编写脚本的接口与Redis提供的接口](https://redis.io/docs/manual/programmability/eval-intro/)兼容。

Dragonfly 使用 Lua 5.4 版本。

## 脚本[flags](/docs/managing-dragonfly/Scripting-with-Lua.md#脚本flags "直接链接到脚本flags")
Dragonfly 通过特殊的脚本标志提供了额外的灵活性。默认情况下，未设置任何内容。

可以通过多种方式配置标志：

#### 1.脚本源[代码](/docs/managing-dragonfly/Scripting-with-Lua.md#1脚本源代码 "直接链接到1.脚本源代码内部")
```lua
#!lua flags=allow-undeclared-keys,disable-atomicty
-- script body below
```
#### 2\. 作为默认[flags](/docs/managing-dragonfly/Scripting-with-Lua.md#2-作为默认flags "直接链接到 2. 作为默认标志")
默认标志适用于所有脚本，并且可以作为参数提供给 Dragonfly。

```Plain Text
./dragonfly --default_lua_flags=allow-undeclared-keys,disable-atomicty
```
#### 3.通过`SCRIPT FLAGS`[命令](/docs/managing-dragonfly/Scripting-with-Lua.md#3通过script-flags命令 "直接链接到 3. 通过`SCRIPT FLAGS`")
可以通过 SHA 为脚本设置标志。

```Plain Text
SCRIPT FLAGS sha1 disable-atomicity allow-undeclared-keys
```
*** 甚至可以在加载脚本之前 ***调用此命令。这使得修补框架或侧面应用程序使用的脚本成为可能。

首先，确定框架/应用程序使用哪些 SHA。这可以通过命令来完成`SCRIPT LIST`。然后，在启动框架/应用程序之前，使用`SCRIPT FLAGS sha1 [flags ...]`所需的标志调用所有必需的脚本。

### 允许未声明的[Key](/docs/managing-dragonfly/Scripting-with-Lua.md#允许未声明的key "直接链接到允许未声明的密钥")
Dragonfly 禁止从脚本访问未声明的键并返回以下错误：

```Plain Text
script tried accessing undeclared key
```
为了允许访问任何Key，包括未声明的Key，`allow-undeclared-keys`应使用该标志。默认情况下禁用此选项，因为不可预测性、原子性和多线程不能很好地混合。如果启用，Dragonlfy 必须在脚本运行时停止所有其他操作。

### 禁用[原子性](/docs/managing-dragonfly/Scripting-with-Lua.md#禁用原子性 "直接链接到禁用原子性")
禁用脚本的原子性允许 Dragonfly 在脚本仍在运行时执行访问任何声明的Key的命令。脚本的执行可以与一系列管道命令进行比较，只是它不是由远程客户端生成的，而是由脚本本身生成的。

该`disable-atomicity`标志禁用原子性。

此行为对于不需要严格原子性的长时间运行的脚本非常有用。由于Key始终可供其他客户端进行读取和写入，因此可以避免延迟峰值。

Dragonfly 的异步模型即使在核心数量较少的情况下也能保持此标志的功能。请注意，只有在调用命令时才能中断脚本。密集计算仍然会导致延迟峰值。



