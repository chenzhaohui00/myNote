## 官方简介

MQTT（Message Queuing Telemetry Transport）是一种基于**发布-订阅**的轻量级消息传递协议，专为资源受限设备和低带宽、高延迟或不可靠的网络而设计。它广泛用于物联网 （IoT） 应用，在传感器、执行器和其他设备之间提供高效的通信。



## 特性

- **轻量：** 数据包小、开销小，适合物联网设备。
- **可靠：** 支持不同的 QoS 级别、session感知和持久连接，即是在高延迟或不稳定的情况下也能确保可靠的消息传递。
- **连续的有状态会话：**MQTT 允许客户端维护与代理的有状态会话，使系统即使在断开连接后也能记住订阅和未传递的消息。客户端还可以在连接期间指定保持活动状态的间隔，该间隔会提示代理定期检查连接状态。如果连接丢失，代理将存储未传递的消息（取决于 QoS 级别），并在客户端重新连接时尝试传递这些消息。此功能可确保可靠的通信，并降低由于间歇性连接而导致数据丢失的风险。
- **安全通信：**支持传输层安全 （TLS） 和安全套接字层 （SSL） 加密。
- **大规模物联网设备支持**
- **语言支持：**支持使用各种编程语言开发的设备和应用程序



## 工作原理

**MQTT Client（MQTT 客户端）**

以往Http概念中的client和service在这边都是client。

**MQTT Broker （MQTT 代理）**

就是消息中转站，处理客户端连接、断开连接、订阅和取消订阅请求以及路由消息。

**发布-订阅模式**

跟EventBus一样的模式，就是客户端之间通过代理，进行订阅/发布。中间发布的数据叫topic

**Topic 主题**

类似与URL，用`/`区分层次结构：

```
chat/room/1

sensor/10/temperature

sensor/+/temperature
```

topic通配符：

`+` ：表示单级通配符，例如 `a/+` 匹配 `a/x` 或 `a/y` 。

`#` ：表示多级通配符，例如 `a/#` 匹配 `a/x` 。 `a/b/c/d`。

**Quality of Service (QoS)**

QoS 0：消息最多传递一次。如果客户端当前不可用，它将丢失此消息。

QoS 1：消息至少传递一次。

QoS 2：消息仅传递一次。



## 工作流

1. 客户端和代理建立连接
2. 客户端分别进行特定topic的订阅和发布
3. 代理接受到发布的topic转发给订阅的客户端，根据指定的Qos确保可靠的消息传递，并根据session类型管理断开的client的消息存储。



## Quick Tutorial

参考：https://www.emqx.com/en/blog/the-easiest-guide-to-getting-started-with-mqtt