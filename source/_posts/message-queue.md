---
title: message-queue
excerpt: "消息队列"
date: 2025-07-29 09:25:39
tag: [Queue,中间件]
categories: [MCP,中间件]
---

### 前言
有些系统面临大流量的冲击，然而系统的处理能力是有限的，也不能吧不能处理的流量丢弃, 
所以我们可以采取一种措施，将这些流量保存起来，等系统空闲的时候，待处理。所以消息队列应运而生。

在现代分布式系统架构中，消息队列作为关键的中间件组件，承担着系统解耦、流量削峰、异步通信等重要职责。
随着业务规模的不断扩大和系统复杂性的持续提升，选择合适的消息队列技术成为保障系统稳定性和可扩展性的关键决策

以RabbitMQ、RocketMQ、kafka为例：

# RabbitMQ vs RocketMQ vs Kafka 对比

| **特性**         | **RabbitMQ**      | **RocketMQ**              | **Kafka**                             |
|------------------|-------------------|---------------------------|---------------------------------------|
| **开发语言**     | Erlang            | Java                      | Scala/Java                            |
| **协议**         | AMQP, STOMP, MQTT | 自定义协议（TCP + 二进制）          | 自定义协议（TCP + 二进制）            |
| **设计定位**     | 企业级通用消息代理         | 金融级高可靠/低延迟消息队列            | 高吞吐分布式流处理平台                |
| **消息模型**     | 队列模型（Queue）       | 队列模型 + 发布/订阅              | 发布/订阅（分区日志模型）             |
| **消息顺序**     | 单队列内保证顺序          | **严格顺序消息**（同一队列分区内）       | 单分区内保证顺序                      |
| **消息存储**     | 内存/磁盘（可选）         | **持久化磁盘存储**（高性能CommitLog） | **持久化磁盘顺序写**                 |
| **吞吐量**       | 中等（万级TPS）         | 高（十万级TPS）                 | **极高（百万级TPS）**                 |
| **延迟**         | 微秒~毫秒级            | **毫秒级**                   | 毫秒~秒级（受批处理影响）             |
| **可靠性**       | 高（ACK/持久化/镜像队列）   | **极高（同步刷盘+主从同步）**         | 高（副本同步+ISR机制）               |
| **事务消息**     | 支持（性能较低）          | **原生支持（两阶段提交）**           | 支持（0.11+版本）                    |
| **消息回溯**     | 不支持               | **支持（按时间/偏移量）**           | **支持（按偏移量）**                 |
| **死信队列**     | **需要安装插件**        | 原生支持                      | 不支持（需自定义实现）               |
| **流处理能力**   | 弱（需插件）            | 有限（Connector支持）           | **强（Kafka Streams生态）**           |
| **管理界面**     | **完善（Web UI）**    | 命令行工具（Dashboard需第三方）      | 第三方工具（如Kafka Manager）        |
| **典型场景**     | 业务解耦、异步任务、轻量级消息   | 电商交易、金融支付、高可靠消息           | 日志收集、实时流处理、大数据管道      |
| **代表用户**     | 金融、传统企业           | 阿里巴巴、蚂蚁金服、京东              | LinkedIn、Netflix、Uber             |

---

## 关键差异总结

1. **吞吐量与延迟**
    - **Kafka**：吞吐量最高，适合大数据量场景，但实时延迟稍高（批处理设计）。
    - **RocketMQ**：平衡型选手，高吞吐+毫秒级延迟，适合金融交易。
    - **RabbitMQ**：吞吐量最低，但微秒级延迟适合轻量级实时任务。

2. **消息模型**
    - **RabbitMQ**：基于 `Exchange-Queue-Binding` 的路由机制，灵活性高。
    - **RocketMQ**：主题+标签（Tag）过滤，支持顺序和事务消息。
    - **Kafka**：分区日志模型，天然支持流式处理，但消费组管理较复杂。

3. **可靠性**
    - **RocketMQ**：同步刷盘+主从强同步，金融级可靠性。
    - **Kafka**：通过ISR副本保证数据不丢失（可配置不同一致性级别）。
    - **RabbitMQ**：依赖镜像队列，网络分区时可能脑裂（需配合仲裁队列）。

4. **生态扩展**
    - **Kafka**：最强流处理生态（Kafka Streams/Connect）。
    - **RocketMQ**：阿里云深度集成，支持事件驱动（EventBridge）。
    - **RabbitMQ**：丰富插件（延迟队列、Sharding等）。

---

## 选型建议

- **选 RabbitMQ 当**：  
  需要灵活路由、轻量级部署、复杂业务逻辑（如死信队列、优先级消息）。

- **选 RocketMQ 当**：  
  需要高可靠事务消息（如支付核心）、严格顺序消息（如订单状态变更）、阿里云环境。

- **选 Kafka 当**：  
  需要超高吞吐（如日志采集）、实时流处理（如Flink集成）、大数据分析管道。


### RabbitMQ

#### 架构

Producer -> Exchange -> Queue -> Consumer

1. Producer（生产者）
- 负责创建并发送消息到 Exchange
- 可以附加消息属性和路由键（Routing Key）
2. Exchange（交换机）
- 接收来自生产者的消息，并根据路由规则将消息分发到相应的 Queue
- 支持多种类型：
   - Direct Exchange：精确匹配路由键
   - Topic Exchange：模式匹配路由键
   - Fanout Exchange：广播到所有绑定队列
   - Headers Exchange：基于消息头属性匹配
3. Queue（队列）
- 存储消息的缓冲区
- 消息在队列中等待消费者处理
- 支持持久化、排他性、自动删除等特性
4. Binding（绑定）
- Exchange 和 Queue 之间的连接规则
- 定义了消息如何从交换机路由到队列
5. Consumer（消费者）
- 从 Queue 中获取并处理消息
- 处理完成后发送确认（ACK）给 Broker

#### 部署
```yaml
version: '3.8'

services:
  rabbitmq:
    #restart: always
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - 5672:5672
      - 15672:15672
    environment:
      TZ: Asia/Shanghai
      RABBITMQ_DEFAULT_USER: root
      RABBITMQ_DEFAULT_PASS: root
    volumes:
      - ./data:/var/lib/rabbitmq
      - ./rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf
      - ./rabbitmq_delayed_message_exchange-4.1.0.ez:/plugins/rabbitmq_delayed_message_exchange-4.1.0.ez  # 延迟消息插件
      - ./enabled_plugins:/etc/rabbitmq/enabled_plugins
```
启用从插件

enabled_plugins文件 内容
[rabbitmq_management,rabbitmq_prometheus,rabbitmq_delayed_message_exchange].

#### 使用示例
```java
@Configuration
public class RabbitmqConfig {

    public static final String DELAY_EXCHANGE = "delay.exchange";

    public static final String DELAY_QUEUE = "delay.queue";

    public static final String DELAY_ROUTING_KEY = "delay.key";

    /**
     * 定义生产者的交换机
     * @return
     */
    @Bean
    public CustomExchange delayExchange() {
        Map<String, Object> args = new HashMap<>();
        //设置延时消息头
        args.put("x-delayed-type", "direct");
        return new CustomExchange(DELAY_EXCHANGE,"x-delayed-message",true,
                false, args);

    }

    /**
     * durable true 持久化 false 不持久化
     * @return
     */
    @Bean
    public Queue delayQueue() {
        return new Queue(DELAY_QUEUE, true);
    }

    /**
     * 绑定交换机与延迟队列，routingkey 表示 消息是否路由到相同key的queue
     * @param delayExchange
     * @param delayQueue
     * @return
     */
    @Bean
    public Binding delayBinding(CustomExchange delayExchange, Queue delayQueue) {
        return BindingBuilder.bind(delayQueue)
                .to(delayExchange)
                .with(DELAY_ROUTING_KEY)
                .noargs();
    }

}
```

```java
@Service
@RequiredArgsConstructor
public class ProductMessage {

    private final RabbitTemplate rabbitTemplate;

    public void sendDelayMessage(String message,Long delayTime) {

        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setTimestamp(new Date());
        messageProperties.setDelayLong(delayTime);
        messageProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);//持久化消息

        Message rabbitMsg = new Message(message.getBytes(StandardCharsets.UTF_8), messageProperties);
        
        //发送延时消息
        rabbitTemplate.convertAndSend(RabbitmqConfig.DELAY_EXCHANGE,
                RabbitmqConfig.DELAY_ROUTING_KEY, rabbitMsg);
    }
}
```

```java
@Slf4j
@Service
public class ConsumerMessage {

    @RabbitListener(queues = RabbitmqConfig.DELAY_QUEUE,ackMode = "MANUAL")
    public void handleDelayMessage(Message message, Channel channel) throws IOException {

       try {
           String msg = new String(message.getBody());

           long time = message.getMessageProperties().getTimestamp().getTime();

           log.info("接收到的消息：{}，实际延迟时间：{}",msg,System.currentTimeMillis()-time);

           //手动ack
           channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
       }catch (Exception e){
           //失败拒绝，重入队列
           channel.basicReject(message.getMessageProperties().getDeliveryTag(),false);
       }
    }
}
```