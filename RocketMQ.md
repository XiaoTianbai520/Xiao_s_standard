# RocketMQ

## 一、角色

| 名称          | 作用                       | 类比           |
| ------------- | -------------------------- | -------------- |
| Producer      | 发送消息                   | 寄快递的人     |
| Consumer      | 消费消息                   | 收快递的人     |
| Broker        | 存储、转发消息             | 快递仓库       |
| Topic         | 消息分类                   | 快递分类区     |
| Tag           | Topic 里的进一步分类       | 分类区里的标签 |
| ConsumerGroup | 一组消费者                 | 一个收货团队   |
| NameServer    | 告诉Producer他的Broker在哪 | 注册中心       |