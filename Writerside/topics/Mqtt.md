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

| 特性     | MQTT                        | Kafka                      |
|--------|-----------------------------|----------------------------|
| 协议类型   | 轻量级消息传输协议，主要用于物联网和嵌入式设备     | 分布式流处理平台，适合大规模、高吞吐量的实时数据处理 |
| 消息传递模型 | 发布/订阅，支持 QoS 等级             | 发布/订阅，基于分区的消费者模型           |
| 适用场景   | 物联网、嵌入式系统、低带宽环境、实时性要求不高的场景  | 日志聚合、实时流处理、事件驱动架构、大规模数据处理  |
| 可扩展性   | 适合小规模系统，可根据 Broker 扩展性增加扩展性 | 高度可扩展，能够通过增加节点实现水平扩展       |

1. MQTT 主要用于轻量级、低带宽的物联网场景，适合资源受限的设备和不稳定的网络环境。它以快速、简洁的方式传输小数据量，支持 QoS
   机制，确保消息传递的可靠性。MQTT 的主要优势在于其高效性、轻量性和适应性。
2. Kafka 则是为高吞吐量、分布式数据处理场景设计的，擅长处理大量实时数据流，支持消息的持久化和消费者的灵活消费。Kafka
   广泛应用于日志收集、数据管道、事件流处理等大规模应用场景。

## 在 java 中编写

1. 引入依赖

   ```xml
    <dependency>
        <groupId>org.springframework.integration</groupId>
        <artifactId>spring-integration-mqtt</artifactId>
        <version>5.5.15</version> <!-- 请使用最新的稳定版本 -->
    </dependency>
   ```

2. 配置连接信息，从yml或者Nacos中获得配置信息
   ```Java
   @Data
   @Configuration
   @ConfigurationProperties(prefix = "spring.mqtt.fs")
   public class MqttPropertiesForFS {
      private String broker;
      private String username;
      private String password;
   }
   ```
   * broker: tcp://ip:port
   * username: admin
   * password: password

3. 创建一个客户端管理类，用于初始化连接和管理客户端，并回调处理连接丢失、消息到达、消息发送

   ```Java
   import org.eclipse.paho.client.mqttv3.*;
   import org.springframework.stereotype.Component;
   import javax.annotation.PostConstruct;
   
   @Component
   @Getter
   @Slf4j
   @RequiredArgsConstructor
   public class MqttClientManager implements MqttCallback {
   
       private MqttClient client;
       private final MqttPropertiesForFS mqttPropertiesForFS;
   
       private boolean isConnected = false;
       private int retryCount = 0;
   
       @PostConstruct
       public void initializeMqttClientForFS() {
           connectAndSubscribe();
       }
   
       private synchronized void connectAndSubscribe() {
           try {
               if (client != null && client.isConnected()) {
                   client.disconnect();
               }
               client = new MqttClient(mqttPropertiesForFS.getBroker(), MqttClient.generateClientId(), new MemoryPersistence());
               client.setCallback(this);
               MqttConnectOptions options = new MqttConnectOptions();
               options.setUserName(mqttPropertiesForFS.getUsername());
               options.setPassword(mqttPropertiesForFS.getPassword().toCharArray());
               options.setConnectionTimeout(ConstantAC.SIXTY_SECONDS);
               options.setKeepAliveInterval(ConstantAC.SIXTY_SECONDS);
               client.connect(options);
               // 订阅主题
               client.subscribe(ConstantAC.REPLY_TOPIC);
               client.subscribe(ConstantAC.REPLY_TOPIC_SPLIT);
               log.info("订阅佛山设备信息完毕");
               isConnected = true;
               retryCount = 0; // 重置重试计数
           } catch (MqttException e) {
               log.error("发生mqtt错误，错误信息{}", e.getMessage());
               // 可以在这里实现重试逻辑，例如延迟后重试
               isConnected = false;
           }
       }
   
       @Override
       public void connectionLost(Throwable cause) {
           log.error("Connection to FS MQTT broker lost! Retrying to reconnect! {}", cause.getMessage());
           isConnected = false;
       }
   
       @Override
       public void messageArrived(String topic, MqttMessage message) {
           // 需要处理任何处理消息时候的异常 否则会导致客户端断开连接
           try{
              // 处理接收到的消息
           log.info("收到消息 - 主题: {}, 消息: {}", topic, new String(message.getPayload()));
           } catch (Exception e) {
               log.error("处理消息时发生错误: {}", e.getMessage());
           }
       }
   
       @Override
       public void deliveryComplete(IMqttDeliveryToken token) {
           // 消息传递完成的回调
           log.info("消息传递完成: {}", token.getMessageId());
       }
   
       @Scheduled(fixedDelay = 5000)
       public void reconnectIfNeeded() {
           if (!isConnected) {
               log.info("尝试重新连接到 MQTT broker...");
               connectAndSubscribe();
               if (!isConnected) {
                   retryCount++;
                   long delay = Math.min(60000, (long) Math.pow(2, retryCount) * 1000); // 指数退避，最多60秒
                   log.info("重试连接将在 {} 毫秒后进行", delay);
                   try {
                       Thread.sleep(delay);
                   } catch (InterruptedException e) {
                       log.error("重连延迟被中断: {}", e.getMessage());
                   }
               }
           }
       }
   }

   ```
   * 单例Mqtt客户端，减少资源开销
   * Mqtt客户端天然线程安全，不会引发线程安全问题
   * 统一配置，简化配置管理
   * 指数回避策略重试连接
   * 通过 synchronized 确保 connectAndSubscribe 方法的线程安全。

4. 发送和接收消息
```Java
MqttClient client = mqttClientManager.getClient();
try {
      client.publish(ConstantAC.TOPIC, new MqttMessage(jsonString.getBytes()));
  } catch (MqttException e) {
      log.error("发送失败", e);
  }
```

