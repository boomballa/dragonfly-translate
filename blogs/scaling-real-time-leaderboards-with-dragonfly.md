# 使用 Dragonfly 实时扩展排行榜
探索如何使用 Dragonfly 和 PostgreSQL 构建动态实时排行榜。了解如何有效管理用户分数和数据，以实现高性能、可扩展的游戏化功能排行榜系统。

[周乔](https://www.dragonflydb.io/blog/authors/joe-zhou)  2024 年 1 月 18 日

![image](/images/3YhuteoaElov6BwUijN84B_NXkEbS7zmD8Ukr3uGWIk.png)

## 介绍
在当今的数字时代，排行榜已成为许多应用程序不可或缺的一部分，提供了一种动态方式来显示用户分数和排名。为了为任何应用程序（即游戏、教育平台）构建游戏化功能，排行榜可以作为吸引和激励用户的强大工具。在这篇博文中，我们将深入探讨构建实用且现实的排行榜系统的过程。

我们的旅程将涉及利用 [Dragonfly](https://github.com/dragonflydb/dragonfly) 的功能，Dragonfly 是 Redis 的高效替代品，以其超高吞吐量和多线程无共享架构而闻名。具体来说，我们将利用 Dragonfly 的两种数据类型：`Sorted-Set`和`Hash`。这些数据结构非常适合处理实时数据和排名系统，使其成为我们排行榜的理想选择。

此外，为了确保我们的排行榜不仅是实时的，而且是持久的，我们将在我们的系统中集成 SQL 数据库 (PostgreSQL)。这种方法使我们能够维护不同时间范围内用户评分的全面记录。因此，我们将能够展示三种不同类型的排行榜：

* 反映总体用户得分的**历史排行榜。**
* 捕获最新用户活动的**本周排行榜。**
* **前几周的排行榜**，让用户深入了解过去的趋势和表现，还可能为表现最好的人提供奖励和奖品。

通过此次落地实施，我们的目标是演示如何利用 Dragonfly 与传统 SQL 数据库结合创建健壮、可扩展且高效的排行榜系统。那么，让我们开始构建吧！

---
## 执行
### 1\. 数据库架构
在我们的排行榜系统的实现中，精心设计的 SQL 数据库模式发挥着关键作用。该架构的核心是表`users`，它对于存储基本用户信息至关重要。该表包括`id`（每个用户的唯一标识符，自动递增为`BIGSERIAL`）、 `email`（防止重复注册的唯一字段）、`password`、`username`和时间戳等字段`created_at`，并`updated_at`跟踪每个用户记录的创建和最后更新。请注意，`password`出于安全目的，该字段应存储用户密码的散列或加密版本。

```sql
CREATE TABLE IF NOT EXISTS users
(
    id         BIGSERIAL PRIMARY KEY,
    email      VARCHAR(255) UNIQUE NOT NULL,
    password   VARCHAR(255)        NOT NULL,
    username   VARCHAR(255)        NOT NULL DEFAULT '',
    created_at TIMESTAMPTZ         NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ         NOT NULL DEFAULT NOW()
);
```
接下来，我们有一个`user_score_transactions`表，它记录用户的所有交易分数。它由一个`id`作为唯一的交易标识符，`user_id`链接到用户表， `score_added`代表分数变化，`reason`用于分数变化（例如赢得比赛或完成任务），以及`created_at`交易记录的时间戳。

```sql
CREATE TABLE IF NOT EXISTS user_score_transactions
(
    id          BIGSERIAL PRIMARY KEY,
    user_id     BIGINT       NOT NULL REFERENCES users (id),
    score_added INT          NOT NULL,
    reason      VARCHAR(255) NOT NULL,
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```
最后，该`user_total_scores`表专门用于维护每个用户的累积分数。它包含`id`每条记录，`user_id`以引用用户表，`total_score`指示用户的总体分数，以及`updated_at`最后一次分数更新的时间戳。

```sql
CREATE TABLE IF NOT EXISTS user_total_scores
(
    id          BIGSERIAL PRIMARY KEY,
    user_id     BIGINT      NOT NULL REFERENCES users (id),
    total_score INT         NOT NULL DEFAULT 0,
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```
因为它强调标准化，该模式特别有效，通过将用户信息、交易分数和总分数分离到不同的表中来减少冗余。它通过使用数据类型`BIGSERIAL`和`BIGINT`容纳大量记录来确保可扩展性。此外，单独的`user_score_transactions`表提供了对每个用户的分数历史记录的宝贵见解，这有利于分析和审计跟踪。我们还将创建具体化视图，以进一步支持前几周的排行榜，稍后我们将看到。通过`user_total_scores`隔离表中的总分，系统可以快速访问和更新用户的总分，从而提高性能。因此，这种结构良好的模式构成了我们排行榜系统的支柱，支持实时更新和全面的分数历史记录。

### 2\. Dragonfly key 和数据类型
数据库架构就位后，我们现在可以专注于用于存储排行榜数据的 Dragonfly 键值对。该`Sorted-Set`数据类型非常适合存储用户分数和排名，而该`Hash`数据类型非常适合存储显示所需的用户信息。经典`String`数据类型对于缓存前几周排行榜的物化视图结果非常有用。以下是我们将使用的键和数据类型：

* `leaderboard:user_scores:all_time (Sorted-Set)`：存储历史排行榜的用户 ID 和分数。
* `leaderboard:user_scores:week_of_{monday_of_the_week} (Sorted-Set)`：存储本周排行榜的用户 ID 和分数。
* `leaderboard:users:{user_id} (Hash)`：存储特定用户在排行榜上显示的最少信息。
* `leaderboard:cache_top_100:week_of_{monday_of_the_week} (String)`：缓存数据库中相应物化视图中特定上周的前 100 位用户。

键空间的示例如下所示：

```bash
dragonfly$> KEYS leaderboard:*
1) "leaderboard:user_scores:all_time"           # Sorted-Set
2) "leaderboard:user_scores:week_of_2024_01_15" # Sorted-Set
3) "leaderboard:users:1"                        # Hash
4) "leaderboard:users:2"                        # Hash
5) "leaderboard:users:3"                        # Hash
6) ...
```
### 3\. 历史和本周排行榜
在历史排行榜和当周排行榜的实现中，我们关注的是如何更新用户的分数以及如何从这些排行榜中查询前100名用户。

为了更新分数，我们首先将交易分数记录在`user_score_transactions`表中，然后更新`user_total_scores`表。此操作应包装在数据库事务中以确保数据完整性。

```sql
BEGIN;

-- Record score transaction for user with ID 1.
INSERT INTO user_score_transactions (user_id, score_added, reason)
VALUES (1, 100, 'WINNING_A_GAME');

-- Update total score for user with ID 1.
UPDATE user_total_scores
SET total_score = total_score + 100,
    updated_at  = NOW()
WHERE user_id = 1;

COMMIT;
```
接下来，我们更新 Dragonfly 的历史排行榜和本周排行榜。请注意，操作被更好地流水线化，以减少应用程序和 Dragonfly 之间的往返次数。

```bash
dragonfly$> ZINCRBY leaderboard:user_scores:all_time 100 1
dragonfly$> ZINCRBY leaderboard:user_scores:week_of_2024_01_15 100 1
```
现在我们已经保存了数据库中的分数变化，并更新了 Dragonfly 中的值，当从排行榜（历史或本周）查询前 100 位用户时，我们可以简单地使用 `Sorted-Set` 的 `ZREVRANGE`  命令来检索前 100 位用户用户，然后使用`HGETALL`命令从 `hash`  key 中检索用户详细信息。

```Plain Text
dragonfly$> ZREVRANGE leaderboard:user_scores:all_time 0 99 WITHSCORES
 1) "1"    # user_id = 1
 2) "1000" # score for user_id = 1
 3) "2"    # user_id = 2
 4) "900"  # score for user_id = 2
 5) "3"
 6) "800"
 7) "4"
 8) "700"
 9) "5"
10) "600"
# ...

dragonfly$> HGETALL leaderboard:users:1
dragonfly$> HGETALL leaderboard:users:2
dragonfly$> HGETALL leaderboard:users:3
dragonfly$> HGETALL leaderboard:users:4
dragonfly$> HGETALL leaderboard:users:5
# ...
```
根据`leaderboard:user_scores:all_time`  key 中记录的用户数量，我们需要使用 1 个`ZREVRANGE`命令，甚至可能使用 100 个`HGETALL`命令来检索排名靠前的用户。这听起来可能有很多命令，但我们可以再次对这些命令进行管道化，以减少应用程序和 Dragonfly 之间的往返次数。事实上，可以在单次往返中检索最高用户分数及其详细信息，并且响应时间仍应在几毫秒内。另一方面，我们完全避免了为顶级用户查询数据库的需要，这是一个昂贵得多的操作。这就是为什么我们有信心说 Dragonfly 正在为排行榜检索提供实时体验。

### 4\. 前几周排行榜
对于前几周排行榜的实现，我们采用了一种有效平衡数据库查询和缓存的策略。该过程涉及两个主要步骤：创建物化视图和利用 Dragonfly 的缓存功能。

我们利用该`user_score_transactions`表生成过去一周排行榜的具体视图。物化视图本质上是查询结果的快照，存储起来是为了高效访问。`user_score_transactions`这些视图是通过聚合表中每个用户在特定一周内的分数来创建的。创建特定周的物化视图的示例 SQL 语句可能如下所示：

```sql
CREATE MATERIALIZED VIEW leaderboard_week_of_2024_01_15 AS
SELECT u.id, u.username, u.email, sum(ust.score_added) AS weekly_score
FROM user_score_transactions ust
         JOIN users u ON ust.user_id = u.id
WHERE ust.created_at BETWEEN '2024-01-15 00:00:00' AND '2024-01-21 23:59:59'
GROUP BY u.id
ORDER BY weekly_score DESC;
```
创建一周排行榜的物化视图后，我们可以将其结果缓存在 Dragonfly 中，以便于快速检索。我们利用 Dragonfly 的`String`数据类型来存储排行榜的序列化形式，可以是 JSON、XML 或任何其他格式。原因是过去的排行榜无法再更改，并且顺序保留在物化视图中，因此我们可以简单地按原样缓存结果。

```sql
SELECT * FROM leaderboard_week_of_2024_01_15 LIMIT 100;
```
```bash
dragonfly$> SET leaderboard:cache_top_100:week_of_2024_01_15 'serialized_leaderboard_data'
```
---
## 其他考虑因素
### 1\. 计算一周的开始
对于每周排行榜，必须有一个一致的方法来确定每周的开始，通常设置为星期一。此计算至关重要，因为它会影响 Dragonfly 中键的命名约定以及在数据库中创建和刷新物化视图的逻辑。有必要在应用程序代码中实现准确计算任何给定周的星期一的辅助方法。这种一致性确保数据库视图和 Dragonfly 键在它们所代表的时间段上保持同步。Go 中的此类实现可能如下所示：

```go
// MondayOfTime returns the Monday of the week of the given time.
func MondayOfTime(ts time.Time) time.Time {
    tt := ts.UTC()
    weekday := tt.Weekday()
    if weekday == time.Monday {
        return tt.Truncate(24 * time.Hour)
    }
    daysToSubtract := (weekday - time.Monday + 7) % 7
    return tt.AddDate(0, 0, -int(daysToSubtract)).Truncate(24 * time.Hour)
}

// MondayOfTimeStr returns the Monday of the week of the given time in string format.
func MondayOfTimeStr(ts time.Time) string {
    return MondayOfTime(ts).Format("2006_01_02")
}
```
### 2\.  Dragonfly Key的管理
历史排行榜数据在 Dragonfly 中用一个 `Sorted-Set` Key 来表示，是一个可以无限期保存的长期数据集。该 Key 不需要过期，因为它会随着时间的推移不断累积用户分数。

相反，Dragonfly 中的当前周`Sorted-Set` Key 应该通过过期策略进行管理。为此 Key 设置一个到期时间点，最好是在下周初，这样可以确保数据不会过时，并且仅反映本周的分数。这种做法有助于保持本周排行榜的相关性和准确性。方便的是，通过在代码中正确使用上述 `MondayOfTime` 函数，当我们进入新的一周时，在第一次分数更新时，Dragonfly 会自动创建一个新的 `Sorted-Set` Key，如果 Key 不存在则使用`ZINCRBY`创建一个 `Sorted-Set` Key。这将成为本周的新开始。

最后，Dragonfly 中在历史和当周排行榜上共享的用户详细信息的 `Hash` Key也可以无限期保留。然而，保持这些用户详细信息的 `Hash`  Key中的数据与数据库中的相应记录保持最新是至关重要的。每当数据库中用户的详细信息发生更改时，这些更改都应立即反映在Dragonfly 中的 `Hash` Key中。这种同步可确保排行榜始终显示最新且准确的用户信息。

### 3\. Key的命名约定
对于 Dragonfly 中存储的不同类型的数据采用清晰且独特的命名约定非常重要。具体来说，本周排行`Sorted-Set`和缓存的物化视图（`String`数据类型）的 Key 名称应该不同，以防止混淆。清晰的命名策略有助于避免对错误的 Dragonfly 数据类型进行意外操作。

---
## 结论
在本博客中，我们探讨了如何将 Dragonfly 与 SQL 数据库结合使用，为游戏和其他应用程序构建强大且高效的排行榜系统。我们讨论了各种数据类型和技术，可以轻松地利用这些数据类型和技术以最小的更新和检索延迟创建实时排行榜。

我们有一个录制的研讨会，“扩展实时排行榜”，您可以[在此处](https://www.dragonflydb.io/events/2023-12-06-workshop-scaling-real-time-leaderboards)观看。[这篇博文中的代码片段可以在 Dragonfly示例存储库](https://github.com/dragonflydb/dragonfly-examples)中找到。最后，我们鼓励您[亲自尝试 Dragonfly](/docs/getting-start/README.md)，亲身体验它的功能，并用它构建令人惊叹的应用程序！

![image](/images/Nq2DN8w2lWawiAMBG1vLHtL2s7Ufbj7t_5CYM__ZBZw.png)

