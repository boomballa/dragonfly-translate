# Dragonfly与Redis的区别

## 字符串长度、索引

字符串大小限制为 256MB。
索引（比如在 GETRANGE 和 SETRANGE 命令中）应该是 [-2147483647, 2147483648] 范围内的有符号 32 位整数。

### 字符串处理

SORT 不考虑任何地区、地域设置。

## 过期闲置
有效期被限制为8年。 F。对于像 PEXPIRE 或 PSETEX 这样具有毫秒精度的命令，大于 2^28ms 的过期时间会悄悄舍入到最接近的秒，损失精度小于 0.001%。

## Lua
我们使用 2022 年发布的 lua 5.4.4。这意味着我们也支持 [lua integers](https://github.com/redis/redis/issues/5261)。
