# Java项目接入外部服务通用模式

起初是我发现Java项目在引入一个技术时例如redis和rocketMQ好像都是先注册一个客户端对象，再通过客户端对象进行调用方法。

而且这个规律远远不止 Redis 和 RocketMQ。

你以后会不断看到：

| 技术          | 常用客户端入口                           |
| ------------- | ---------------------------------------- |
| Redis         | `RedisTemplate`                          |
| Redisson      | `RedissonClient`                         |
| RocketMQ      | `RocketMQTemplate` / `DefaultMQProducer` |
| Kafka         | `KafkaTemplate`                          |
| Elasticsearch | `ElasticsearchClient`                    |
| MongoDB       | `MongoTemplate`                          |
| RabbitMQ      | `RabbitTemplate`                         |
| HTTP          | `RestClient` / `WebClient`               |
| 数据库        | `DataSource`                             |
| MinIO         | `MinioClient`                            |
| AWS S3        | `S3Client`                               |

于是你以后看到一个新技术，可以先条件反射地问四个问题：

```
① 它的 Java Client 是什么？

② Client 需要配置什么？
   host？
   port？
   username？
   password？
   timeout？

③ Spring Boot 有没有 Starter？
   如果有，可能连 Client 都不用自己创建。

④ 业务代码通过哪个 Bean 调用？
```

你现在其实可以抽象出一个很好用的公式：

```
引入依赖
    ↓
写配置
    ↓
创建 / 自动配置 Client
    ↓
Client 进入 IoC 容器
    ↓
业务类注入 Client
    ↓
调用 API
```