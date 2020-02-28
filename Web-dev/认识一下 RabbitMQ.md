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

