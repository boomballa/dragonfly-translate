# 访问控制列表 (ACL)
Dragonfly 内置了对 ACL 的支持。 Dragonfly 操作员可以通过 ACL 系列命令对访问数据存储的方式和人员进行细粒度控制。 由于 Dragonfly 被设计为 Redis 的直接替代品，因此您可以期待与 Redis 中相同的 ACL API 功能。

Dragofnly 中的所有连接默认为用户`default`（除非该用户被禁用）。默认情况下，用户 `default` 可以在 Dragonfly 中使用任何密码`AUTH`， 并被允许执行任何命令，并且是所有可用 ACL 组的一部分。

给定用户的权限通过特定于域的语言 (DSL) 进行控制，分为 4 类：

1. ACL组
2. 命令
3. 发布/订阅消息（尚未实现）
4. 按键（尚未实现）

授予或撤销用户权限就像调用 `ACL SETUSER` 命令一样简单。例如：

```Plain Text
ACL SETUSER John ON >mypassword +@ADMIN +SET
```
如果用户`John` 不存在，则会使用命令参数列表中指定的权限创建该用户。 否则，`SETUSER` 充当该用户的条目（及其权限）的 `update`。

用户可以是`ON` 或`OFF`。默认情况下，所有用户（`default` 除外）都是 `OFF`（除非您明确授予他们`ON`）。这 `ON`/`OFF`机制授予或撤销用户使用`authenticate`的能力> 命令。`AUTH`

## [密码​](/docs/managing-dragonfly/Access-Control-Lists.md#密码​ "直接链接到密码")
可以使用 `>` 字符后跟密码来设置密码。例如：`ACL SETUSER Mike >mypassword`，创建用户`Mike` 通过`mypassword`。目前，每个用户只能拥有一个密码。

特殊词`nopass`允许用户`AUTH`使用任何密码。

请注意，以后对现有用户使用`>`时，请更新密码。

## [​身份验证](/docs/managing-dragonfly/Access-Control-Lists.md#​身份验证 "直接链接到身份验证")
用户可以使用`AUTH <username> <password>`授权与给定用户的连接。之后，在连接内发出的所有命令 将遵守用户指定的权限。将 `default` 用户状态更改为 `OFF` 或密码，将需要所有传入连接 进行身份验证。

请注意，如果密码已更改且用户已更改`authenticated`，则在重新连接之前无需重新进行身份验证。 基本上，密码更改不充当连接驱逐机制。但是，如果使用 `ACL DELUSER` 将用户从系统中删除， 然后他们的连接就会被系统终止。此外，使用 `ACL SETUSER` 对用户权限列表进行的任何更改都将传播到已 活动且经过身份验证的连接。

另请注意，标记`--requirepass` 还会更改`default` 用户密码。因此，如果在 Dragonfly 启动期间设置了标志 `requirepass`， 那么`default`用户的密码将是该标志中指定的密码。

## [ACL 组​](/docs/managing-dragonfly/Access-Control-Lists.md#acl-组​ "直接链接到 ACL 组")
每个命令都属于一组 ACL 组。指定组的语法是：

```Plain Text
+@GROUP_NAME
-@GROUP_NAME
```
前面的符号指示操作（`+` 表示授予，`-` 表示撤销）。 `@` 表示类别（ACL 组）， `GROUP_NAME` 是组的名称。例如：

```Plain Text
ACL SETUSER John ON +@FAST
```
更新用户的权限`John`并授予他运行组中任何命令的能力`FAST`。撤销此规定， 很简单：

```Plain Text
ACL SETUSER John -@FAST
```
还有特殊关键字`+@ALL, -@ALL`，允许用户运行 `ALL GROUPS` 中找到的命令。

使用`@ALL`可以执行以下操作：

```Plain Text
ACL SETUSER John +@ALL -@ADMIN -@FAST
```
这基本上将除 `@ADMIN` 和 `@FAST` 组之外的所有组授予用户。这样，就可以很容易地表达哪些权限组 用户不应该是其中的一部分。可使用命令 `ACL CAT` 和用户特定信息访问所有类别的列表 可以通过命令`ACL GETUSER <username>`访问。

## [命令​](/docs/managing-dragonfly/Access-Control-Lists.md#命令​ "直接链接到命令")
将命令分组提供了快速授予/撤销权限的极大灵活性，但它在某种程度上受到限制，因为 这些组不是用户定义的。因此，为了更好地控制，用户可以指定明确允许执行的命令列表。 例如：

```Plain Text
ACL SETUSER John +GET +SET +@FAST
```
这允许用户`John`仅执行`SET`和`GET`命令以及与该命令关联的所有命令组`FAST`。 用户`John`发出上述以外的任何命令的任何尝试都将被系统拒绝。

请注意，语法与 ACL 组类似，但没有前缀`@`。

特殊的`+ALL`（注意没有`@`）用于表示所有当前实现的命令。

## [持久性​](/docs/managing-dragonfly/Access-Control-Lists.md#持久性​​ "直接链接到持久性")
可以捕获所有用户的状态及其权限并将其放置在文件中。与 Redis 一样， `--aclfile` 选项用于指定 Dragonfly 从中加载 ACL 状态的文件。

之后，在运行时完成的任何更改都可以随时通过命令 `ACL SAVE` 保存，该命令 将当前存储的 ACL 状态逐出到 `--aclfile` 选项中指定的文件。

请注意，`aclfile` 文件与 Redis 兼容（但不得包含任何键或 pub/sub DSL，因为这些尚不受支持，因此如果您打算迁移，只需打开文件并将其删除）。

如果您希望 `aclfile` 可写，也就是说，如果您希望 `ACL SAVE` 工作，我们建议不要放置 。 访问。您可以通过编辑来更改此行为 systemd 服务文件位于 目录下，因为通常该目录只能由 Dragonfly 作为 `aclfile``/etc``readonly``/lib/systemd/system/dragonfly.service`

为方便起见，我们建议将 acl 文件放置在`/var/lib/dragonfly/`中。

## [​日志](/docs/managing-dragonfly/Access-Control-Lists.md#​日志 "直接链接到日志")
无法验证的所有连接以及无法运行命令的所有经过验证的用户（因为 他们的权限）存储在日志中。日志的大小可以通过选项`--acllog_max_len`配置。 该标志的运行方式与 Redis 略有不同。具体来说，因为 Dragonfly 在每个核心架构中使用无共享线程， 每个执行线程都有自己的日志。因此，日志条目的总大小是标志数乘以 由可用的 Dragonfly 线程数决定。例如，如果您正在运行具有 4 个线程的 Dragonfly `--acllog_max_len=8` 那么任意时刻系统中存储的日志条目总数可以为`32`，每个核心最多可以存储`8`个条目。 当达到每线程阈值时，每个新的日志条目都会导致最旧的日志条目被驱逐。

可以通过命令打印日志信息`ACL LOG`。

