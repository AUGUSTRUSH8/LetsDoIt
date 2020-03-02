### 认识一下 RabbitMQ

分布式系统中，如何在各个应用之间高效的进行通信，是系统设计中的一个关键。

使用 **消息代理（message broker）** 是一个优雅的解决方案。

**RabbitMQ** 就是一个被广泛应用的消息代理，遵循 **AMQP协议**。

接下来我们就了解一下：

- Message Broker 概念
- AMQP 协议的核心构成
- 消息转发的 4 种模式

#### Message Broker

**broker** 是经纪人的意思，促成卖方、买方的交易，例如房产经纪人。

消息模型中，有消息的生产者、消费者，就相当于卖方、买方。

所以，也需要一个消息经纪人，这就引出了 **message broker** 的概念。

message broker 从生产者接收消息，再发送给消费者，这样，生产者、消费者可以完全隔离。

**RabbitMQ** 就是一个 **message broker**。

#### AMQP协议

具体如何传递消息？要看使用的消息协议。

RabbitMQ 支持多种协议，其中最重要的是 **AMQP**（Advanced Message Queuing Protocol）。

AMQP 的概念模型很简单，包含3个部分：

- Queue
- Binding
- Exchange

当一个消息发布到 RabbitMQ 后，首先到达 Exchange，然后 Exchange 把消息分配给 Queue，消费者从 Queue 中得到消息。

AMQP 是一个可编程的协议，我们可以自由配置 exchange, binding, queue。

#### Queue队列

Queue 是先进先出数据结构。

Queue 是 RabbitMQ 存储消息的地方。

Queue 可以灵活的配置，例如：

- 设置 name
- 配置可靠模式，即使 broker 宕机也可以保障数据安全
- 消息自动删除
- 独占模式
- ……

#### Consumer 消费者

一个 queue 可以同时连接多个 consumer。

consumer 既可以自己从 queue 拉取消息，也可以由 queue 主动把消息推给 consumer。

#### Binding 绑定

Binding 是 Queue 与 Exchange 建立连接的**规则**。

每个 Queue 都会与一个默认 Exchange 连接，我们可以为这个 Queue 连接更多的 Exchange。

Exchange 通过 Binding 规则，把消息路由到相应的 Queue。

#### Exchange

Exchange 是 RabbitMQ 的消息网关。

Exchange 就像是一个接线员，收到消息后决定如何转发。

主要有 4 种转发类型：

- Direct
- Fanout
- Topic
- Header

下面具体了解一下。

#### Direct 直传

```java
Routing key == Binding key
```

- `routing key` 是消息的一个属性。
- `binding key` 是绑定 queue 与 exchange 时指定的。

消息到达指定的 queue 之后，会以**轮询**的形式分派给消费者，确保负载均衡。

#### Fanout 扇出

此方式会忽略 `routing key`，把消息分派给所有连接的 queue。

最常见的场景就是**消息广播**。

注意，此方式是 exchange 广播给 queue，不是 queue 广播给 consumer。queue 到 consumer 还是轮询的方式。

#### Topic 主题

`routing key` 与 `binding key` 进行模式匹配。

```java
Routing key == Pattern in binding key
```

RabbitMQ 使用 `*` 和 `#` 这2个通配符。

`*` - 匹配一个词。

`#` - 匹配 0 个或多个词。

这种形式有非常多的应用场景，可以用于发布-订阅模式、将相关数据分发给期望的 worker 等等。

#### Header

一种特殊类型的 exchange，基于消息头中的 key 进行路由。

使用这种方式后，就会忽略消息的 **routing key** 属性。

对一个 header exchange 创建 binding 时，可以对一个 queue 绑定多个 header，这种情况下，消息生产者需要告诉 RabbitMQ 匹配哪些 key，producer 可以指定一个标识 `x-match`，值可以是：

- **any** - 只有一个值应该匹配。
- **all** - 所有值都必须匹配。

#### 消息确认

消息到达目的地之后，broker 应该从队列中将其删除，这是为了防止消息过多导致溢出。

删除消息之前，broker 必须得到确认通知。

有 2 种通知方式：

- **自动通知**：只要 consumer 接收到消息即可，不管是否处理完成。
- **明确显示通知**：只有在 consumer 发送回来一个确认信息后才可以，这样保证 consumer 处理完成后再删除。

#### 在CentOS 7上安装RabbitMQ服务器

RabbitMQ是一个免费的开源企业消息代理软件。 它是用Erlang编写的，并实现了高级消息队列协议（AMQP）。 它提供所有主要编程语言的客户端库。 它支持多种消息传递协议，消息队列，传送确认，灵活的路由到队列，多种交换类型。 它还提供易于使用的HTTP-API，命令行工具和用于管理RabbitMQ的Web UI；

**更新基本系统**

```shell
yum -y update
```

**安装Erlang**

RabbitMQ是用Erlang语言编写的，这里将安装最新版本的Erlang到服务器中。 Erlang在默认的YUM存储库中不可用，因此需要安装EPEL存储库。 运行以下命令

```shell
yum -y install epel-release
yum -y update
```

使用以下命令安装Erlang。

```shell
yum -y install erlang socat
```

现在可以使用以下命令检查Erlang版本。

```shell
erl -version
```

将得到以下输出。

```shell
[root@liptan-pc ~]# erl -version
Erlang (ASYNC_THREADS,HIPE) (BEAM) emulator version 5.10.4
```

**安装RabbitMQ**

这里RabbitMQ为预编译并可以直接安装的RPM软件包。 唯一需要的依赖是Erlang环境。 我们已经安装了Erlang，可以进一步下载RabbitMQ。 通过以下命令下载Erlang RPM软件包。

```shell
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.10/rabbitmq-server-3.6.10-1.el7.noarch.rpm
```

如果你没有安装wget ，可以运行yum -y install wget 。

通过运行导入GPG密钥：

```shell
rpm –import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
```

运行RPM安装RPM包：

```shell
rpm -Uvh rabbitmq-server-3.6.10-1.el7.noarch.rpm
```

RabbitMQ现已安装在您的系统上。	

**开启RabbitMQ**

可以通过运行以下命令启动RabbitMQ服务器进程。

```shell
systemctl start rabbitmq-server
```

要在引导时自动启动RabbitMQ，运行以下命令。

```shell
systemctl enable rabbitmq-server
```

要检查RabbitMQ服务器的状态，运行：

```shell
systemctl status rabbitmq-server
```

如果启动成功，应该得到以下输出。

```shell
? rabbitmq-server.service - RabbitMQ broker
   Loaded: loaded (/usr/lib/systemd/system/rabbitmq-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2017-07-15 18:59:14 UTC; 3min 22s ago
 Main PID: 29006 (beam.smp)
   Status: "Initialized"
   CGroup: /system.slice/rabbitmq-server.service
           ??29006 /usr/lib64/erlang/erts-9.0/bin/beam.smp -W w -A 64 -P 1048576 -t 5000000 -stbt db -zdbbl 32000 -K tr...
           ??29149 /usr/lib64/erlang/erts-9.0/bin/epmd -daemon
           ??29283 erl_child_setup 1024
           ??29303 inet_gethost 4
           ??29304 inet_gethost 4

Jul 15 18:59:13 centos rabbitmq-server[29006]: Starting broker...
Jul 15 18:59:14 centos rabbitmq-server[29006]: systemd unit for activation check: "rabbitmq-server.service"
Jul 15 18:59:14 centos systemd[1]: Started RabbitMQ broker.
Jul 15 18:59:14 centos rabbitmq-server[29006]: completed with 0 plugins.
```

**修改防火墙规则**

我这里使用的是阿里云轻量级应用服务器，需要在Web端放开**15672和5672**两个端口，一个是Web运维的网络端口，一个是MQ服务器访问本身需要开放的端口。

如果是其他单机系统环境，请搜索对应指令放开端口防火墙拦截。

**访问Web控制台**

启动RabbitMQ Web管理控制台，方法是运行：

```shell
rabbitmq-plugins enable rabbitmq_management
```

通过运行以下命令，将RabbitMQ文件的所有权提供给RabbitMQ用户：

```shell
chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/
```

现在，需要为RabbitMQ Web管理控制台创建管理用户。 运行以下命令。

```
rabbitmqctl add_user admin StrongPassword
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin “.*” “.*” “.*”
```

确保将StrongPassword更改为非常强大的密码。

要访问RabbitMQ的管理面板，使用浏览器打开以下URL。

```shell
http://Your_Server_IP:15672/
```

然后你就会看到RabbitMQ的Web管理界面。

**遇到的坑及解决办法**

1.RabbitMq 本地连接报错 org.springframework.amqp.AmqpIOException: java.io.IOException

解决办法参见[链接](https://blog.csdn.net/qq_22638399/article/details/81705606)

**SpringBoot测试与RabbitMQ交互**

参见[链接](https://www.cnblogs.com/5ishare/p/10163318.html)