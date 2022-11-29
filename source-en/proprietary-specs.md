# 专有协议和编码的 CloudEvent 规范

<!-- no verify-specs -->

免责声明:CloudEvents 不支持这些协议或规范，也不确保它们是 CloudEvents 当前版本的最新版本。
这是各自项目维护者的责任。

- [Apache RocketMQ Transport Binding](https://github.com/apache/rocketmq-externals/blob/master/rocketmq-cloudevents-binding/rocketmq-transport-binding.md)
- [Google Cloud Pub/Sub Protocol Binding](https://github.com/google/knative-gcp/blob/master/docs/spec/pubsub-protocol-binding.md)

**想要向专有传输添加绑定?**

- 创建一个遵循现有绑定规范结构的规范 (e.g. [http](bindings/http-protocol-binding.md) or [amqp](bindings/amqp-protocol-binding.md)) - 这将有助于 SDK 开发。
  - **NOTES:**
    - 规范必须是公开的，并由提议的组织进行管理。
    - 规范必须明确说明支持的 CloudEvents 的版本。
- 对该文件打开一个拉请求。
- 响应关于 pull 请求的任何评论，并可能加入一个定期安排的工作组会议。
