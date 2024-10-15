# MQTT

MQTT（Message Queuing Telemetry Transport）是一种基于发布/订阅模式的轻量级消息协议，主要用于物联网（IoT）设备之间的数据传输。
它的设计初衷是提供一种轻量、高效且具有低带宽和低功耗的通信方式，适合嵌入式设备和不可靠网络环境。

## 基本概念

MQTT 的基本架构：
* 客户端（Client）：可以是消息的发布者（Publisher）或订阅者（Subscriber），它们通过 Broker 进行通信。
* Broker（代理）：负责接收和转发消息，管理订阅者和发布者的关系。
* 主题（Topic）：消息发布的地址，发布者将消息发布到特定的主题上，订阅者通过订阅该主题来接收消息。

关键特点：
* 轻量级：消息格式极其简单，开销非常小，适合资源受限的设备和带宽受限的网络。
* 发布/订阅模型：客户端通过订阅主题接收消息，消息通过主题进行广播，发送方和接收方之间没有直接联系，解耦了消息的生产和消费。
* QoS（服务质量）级别：MQTT 提供三种不同的服务质量（QoS）级别：
* QoS 0：消息最多传递一次，不保证消息到达。
* QoS 1：消息至少传递一次，可能会有重复消息。
* QoS 2：消息仅传递一次，确保消息不重复、不丢失。
* 持久化会话：支持客户端和服务器之间的持久会话，即使客户端短暂掉线，消息仍然能够在下次连接时传递。
* 心跳机制：通过 KeepAlive 设置定期心跳包来保持连接稳定。
* 低带宽、高效传输：即便是在高延迟、低带宽的网络中，MQTT 也能有效运作。


|特性|MQTT| Kafka  |
|--|---|---|
|  |   |   |
|  |   |   |
|  |   |   |
|  |   |   |
|  |   |   |
|  |   |   |


## 在 java 中编写

Some introductory information.

1. Step with a code block

   ```bash
    run this --that
   ```

2. Step with a [link](https://www.jetbrains.com)

3. Step with a list.
   - List item
   - List item
   - List item