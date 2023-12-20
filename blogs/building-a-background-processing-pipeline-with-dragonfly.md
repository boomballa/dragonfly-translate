# 使用 Dragonfly 构建后台处理管道
在这篇博文中，您将学习如何使用 Redis 列表通过 Dragonfly 构建后台处理 pipline。

[阿里·肖特兰](https://www.dragonflydb.io/blog/authors/ari-shotland)  2023 年 4 月 5 日

![image](/images/-_qJT8jN4wk07cj1MdY_6kESyi6v-xxc8udKLTX4O_M.png)

## 介绍
在这篇博文中，您将学习如何使用 [Redis 列表](https://redis.io/docs/data-types/lists/) 用 [Dragonfly](https://www.dragonflydb.io/) 构建后台处理 pipline。

Dragonfly 是专为现代应用程序工作负载构建的内存数据存储。它与 Redis 和 Memcached API 完全兼容，可提供 25 倍以上的吞吐量、更高的缓存命中率、更低的尾部延迟以及轻松的垂直可扩展性。

我们将探索的特定用例是营销自动化系统，该系统采集用户事件并根据其中的一些事件发送通知 - 例如，当用户在网站上注册时发送欢迎电子邮件。

## 高级解决方案架构
该应用程序包括以下组件：

1. Dragonfly - 与 Redis 兼容的内存数据存储，用作生产者和工作应用程序的后端
2. 模拟网站用户注册过程的生产者应用程序。
3. 处理这些用户注册请求并发送欢迎电子邮件的工作应用程序。

生产者和工作人员应用程序都是用 Go 编写的，并使用 \_gocelery\_ 库。gocelery库可让您在 Go\_中\_实现 celery 工作程序并提交 celery 任务。

\_gocelery\_在幕后使用 Redis List，这样我们就不必在应用程序中处理它。相反，\_gocelery\_ 库为我们提供了一个简化的 API 来提交和处理需要异步处理的任务。

或者，您可以使用 go-worker 或任何 Go Redis 客户端来使用普通 Redis List 命令（`LPUSH` 和 `BRPOP` 的组合）来实现此目的。

### 生产者应用
我们从生产者应用程序开始，它生成假用户注册并将其添加到任务队列中。

在下面的代码块中，发生了以下事情：

1. 首先，我们创建一个Redis连接池并使用它来创建`gocelery.CeleryClient`。该`Delay`方法用于向工作应用程序提交任务。
2. 第一个参数是任务的名称，第二个参数是工作应用程序需要处理的数据。
3. 然后我们创建一个 Go 通道，用于监听`SIGINT`信号`SIGTERM`。A`goroutine`每秒开始向工作应用程序发送模拟用户注册。模拟数据是使用 go-random 数据库生成的。
4. 最后，我们等待`exit`通道接收信号，然后终止生产者循环和应用程序。

```go
package main

import (
   "fmt"
   "log"
   "os"
   "os/signal"
   "syscall"
   "time"

   rdata "github.com/Pallinder/go-randomdata"
   "github.com/gocelery/gocelery"
   "github.com/gomodule/redigo/redis"
)

const (
   redisHostEnvVar     = "REDIS_HOST"
   redisPasswordEnvVar = "REDIS_PASSWORD"
   taskName            = "users.registration.email"
)

var (
   redisHost string
)

func init() {
   redisHost = os.Getenv(redisHostEnvVar)
   if redisHost == "" {
       redisHost = "localhost:6379"
   }

}

func main() {
   redisPool := &redis.Pool{
       Dial: func() (redis.Conn, error) {
           c, err := redis.Dial("tcp", redisHost)

           if err != nil {
               return nil, err
           }
           return c, err
       },
   }

   celeryClient, err := gocelery.NewCeleryClient(
       gocelery.NewRedisBroker(redisPool),
       &gocelery.RedisCeleryBackend{Pool: redisPool},
       1,
   )

   if err != nil {
       log.Fatal(err)
   }

   exit := make(chan os.Signal, 1)
   signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)
   closed := false

   go func() {
       fmt.Println("celery producer started...")

       for !closed {
           res, err := celeryClient.Delay(taskName, rdata.FullName(rdata.RandomGender)+","+rdata.Email())
           if err != nil {
               panic(err)
           }
           fmt.Println("sent data and generated task", res.TaskID, "for worker")
           time.Sleep(1 * time.Second)
       }
   }()

   <-exit
   log.Println("exit signalled")

   closed = true
   log.Println("celery producer stopped")
}
```
### Worker 应用
现在我们转向 Worker 应用程序，它应该从队列中挑选任务并发送与它们相对应的电子邮件。

1. 首先我们创建一个Redis连接池并使用它来创建`gocelery.CeleryClient`. Celery客户端用于接收来自Dragonfly List 的任务。
2. 该`Register`方法用于注册一个函数，当收到任务时将调用该函数。第一个参数是任务的名称，第二个参数是收到任务时将调用的函数。
3. 在 a 中`goroutine`，我们启动工作实例，它将开始侦听来自 Dragonfly 的事件。
4. 最后，我们等待`exit`通道接收信号，停止 Celery 工作实例，然后正常关闭应用程序。

```go
package main

import (
   "fmt"
   "log"
   "math/rand"
   "net/smtp"
   "os"
   "os/signal"
   "strings"
   "syscall"
   "time"

   "github.com/gocelery/gocelery"
   "github.com/gomodule/redigo/redis"
   "github.com/hashicorp/go-uuid"
)

const (
   redisHostEnvVar = "REDIS_HOST"
   taskNameEnvVar  = "TASK_NAME"
   smtpServer      = "localhost:1025"
   taskName        = "users.registration.email"
)

const fromEmail = "admin@foo.com"

const emailBodyTemplate = "Hi %s!!\n\nHere is your auto-generated password %s. Visit https://foobar.com/login to login update your password.\n\nCheers,\nTeam FooBar.\n\n[processed by %s]"

const autogenPassword = "foobarbaz_foobarbaz"

const emailHeaderTemplate = "From: %s" + "\n" +
   "To: %s" + "\n" +
   "Subject: Welcome to FooBar! Here are your login instructions\n\n" +
   "%s"

var (
   redisHost string
   workerID  string
)

func init() {
   redisHost = os.Getenv(redisHostEnvVar)
   if redisHost == "" {
       redisHost = "localhost:6379"
   }

   rnd, _ := uuid.GenerateUUID()
   workerID = "worker-" + rnd
}

func main() {
   redisPool := &redis.Pool{
       Dial: func() (redis.Conn, error) {
           c, err := redis.Dial("tcp", redisHost)
           if err != nil {
               return nil, err
           }
           return c, err
       },
   }

   celeryClient, err := gocelery.NewCeleryClient(
       gocelery.NewRedisBroker(redisPool),
       &gocelery.RedisCeleryBackend{Pool: redisPool},
       1,
   )

   if err != nil {
       log.Fatal("failed to create celery client ", err)
   }

   sendEmail := func(registrationEvent string) {
       name := strings.Split(registrationEvent, ",")[0]
       userEmail := strings.Split(registrationEvent, ",")[1]

       fmt.Println("user registration info:", name, userEmail)

       sleepFor := rand.Intn(9) + 1
       time.Sleep(time.Duration(sleepFor) * time.Second)

       body := fmt.Sprintf(emailBodyTemplate, name, autogenPassword, workerID)
       msg := fmt.Sprintf(emailHeaderTemplate, fromEmail, userEmail, body)


       err := smtp.SendMail(smtpServer, nil, "test@localhost", []string{"foo@bar.com"}, []byte(msg))
       if err != nil {
           log.Fatal("failed to send email - ", err)
       }

       fmt.Println("sent email to", userEmail)

   }

   celeryClient.Register(taskName, sendEmail)

   go func() {
       celeryClient.StartWorker()
       fmt.Println("celery worker started. worker ID", workerID)
   }()

   exit := make(chan os.Signal, 1)
   signal.Notify(exit, syscall.SIGINT, syscall.SIGTERM)

   <-exit
   fmt.Println("exit signalled")

   celeryClient.StopWorker()
   fmt.Println("celery worker stopped")
}
```
## 先决条件
要使用博客中的说明运行整个应用程序，您需要在本地计算机上安装以下软件：

1. 去
2. 码头工人

## 运行应用程序
我们的 worker 应用程序将发送电子邮件。我们将使用模拟 SMTP 服务器来使事情变得简单而现实，而不是使用外部服务。

1. 在 Docker 中启动 SMTP 服务器

```Plain Text
docker run --rm -p 1080:1080 -p 1025:1025 soulteary/maildev
```
容器启动后，您应该看到以下日志：

```Plain Text
MailDev using directory /tmp/maildev-1
MailDev webapp running at http://0.0.0.0:1080
MailDev SMTP Server running at 0.0.0.0:1025
```
2. 在 Docker 中启动 Dragonfly（Linux 或 MacOS 具体说明请参阅文档）

```bash
docker run --rm -p 6379:6379 --ulimit memlock=-1 docker.Dragonfly.io/Dragonfly/dragonfly
```
3. 启动 worker 应用程序

```bash
go run worker/worker.go
```
您应该看到此日志（您的情况下的工作 ID 可能有所不同）：

```Plain Text
celery worker started. worker ID worker-0d8d8b51-d351-ba87-c012-2b404439e1f2
```
4. 启动生产者应用程序

```Plain Text
go run producer/producer.go
```
当生产者应用程序启动并继续发送模拟数据时，您应该看到这些日志（任务 ID 在您的情况下可能不同）：

```bash
celery producer started...
sent data and generated task 3687275c-748b-4eaf-b540-dbe772123fa8 for worker
sent data and generated task 7da24e60-f33b-4690-9262-6408ac85a9ef for worker
...
```
### 验证结果
当生产者应用程序发送模拟用户数据时，工作应用程序将对其进行处理并发送电子邮件。要检查电子邮件，请使用浏览器导航至 SMTP 服务器的用户界面 -`http://localhost:1080/`

![image](/images/XwbX03tiR7KISv0Z5DcmXWSfOTkhNAVfF6B7nRdyO6E.png)
<p style="text-align: center;"> worker 应用程序发送的电子邮件</p>
<center>  worker 应用程序发送的电子邮件 </center>  

现在是了解 Dragonfly 的好时机。由于它在 Docker 容器中本地运行，因此我们可以使用`redis-cli`. 连接后，检查一个名为 `celery` 的 key， 这是充当队列来保存最终由工作应用程序处理的用户注册事件的列表。

```Plain Text
127.0.0.1:6379> llen celery
(integer) 106
```
在电子邮件的末尾，您会注意到`[processed by worker-<random uuid>]`。下一节将详细介绍这一点。

### 扩展后台处理 pipline
使用列表允许我们通过简单地添加更多工作应用程序来水平扩展处理管道。要启动工作应用程序的另一个实例：

```Plain Text
go run worker/worker.go
```
几秒钟后，您将从工作程序应用程序日志中注意到两个实例都在处理 List ( `celery`) 中的数据。如果您仔细查看电子邮件内容，最后您将能够看到工作应用程序的哪个实例实际处理并发送了该电子邮件。

这仅用于学习/测试目的。

您可以继续启动其他工作实例。

## 解决方案代码演练
现在您已经看到了应用程序的运行情况，让我们看一下代码。我将重点介绍与博客文章相关的代码关键部分。

[完整的代码可以在 GitHub 上找到](https://github.com/dragonflydb/background-process-example)

## 结论
在这篇博文中，我们了解了如何使用 Dragonfly 和 Go 构建后台处理管道。该用例是一个营销自动化系统，我们向在网站上注册的用户发送电子邮件。然而，相同的方法可用于使用 Dragonfly 作为消息后端来构建任何类型的处理管道。我们还了解了如何通过简单地添加更多工作应用程序来水平扩展后台处理管道。



