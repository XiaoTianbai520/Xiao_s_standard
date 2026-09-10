# Ubuntu服务器规范

## 一、文件目录

`/`
`├── etc/                # 系统和服务配置`
`├── opt/                # 自己部署的软件、服务、开发工具`
`│   ├── apps/           # 自己的业务应用`
`│   ├── middleware/     # 中间件`
`│   ├── runtime/        # JDK、Node 等运行环境`
`│   └── tools/          # 手动安装的工具`
`│`
`├── data/               # 重要业务数据【重点】`
`│   ├── mysql/`
`│   ├── redis/`
`│   ├── minio/`
`│   ├── docker/`
`│   ├── uploads/`
`│   └── backup/`
`│`
`├── var/`
`│   └── log/            # 服务日志`
`│       └── apps/`
`│`
`├── var/www/            # 网站静态文件`
`│`
`├── home/`
`│   └── 你的用户名/      # 个人桌面环境`
`│       ├── Desktop/`
`│       ├── Documents/`
`│       ├── Downloads/`
`│       ├── Pictures/`
`│       ├── Videos/`
`│       └── Projects/   # 开发中的源码`
`│`
`└── tmp/                # 临时文件`

**每个服务内部**：

```
/opt/middleware/rocketmq/
├── docker-compose.yml
├── conf/
└── scripts/

/opt/middleware/redis/
├── docker-compose.yml
├── conf/
└── scripts/
```

**业务项目统一：**

```
/opt/apps/
├── huimeng/
│   ├── docker-compose.yml
│   ├── app.jar
│   ├── conf/
│   └── scripts/
│
└── other-project/
```

| 类型           | 放哪里            | 示例                           |
| -------------- | ----------------- | ------------------------------ |
| 系统配置       | `/etc`            | Nginx、SSH 配置                |
| 已部署服务     | `/opt/apps`       | Spring Boot 项目               |
| 中间件         | `/opt/middleware` | Nginx、Tomcat 等手动安装的软件 |
| 运行环境       | `/opt/runtime`    | JDK、Node.js                   |
| 工具           | `/opt/tools`      | 自己下载的 CLI 工具            |
| 数据           | `/data`           | MySQL、MinIO、上传文件         |
| 日志           | `/var/log`        | Java/Nginx 日志                |
| 网站静态文件   | `/var/www`        | Vue/React dist                 |
| 开发源码       | `~/Projects`      | IDEA 项目                      |
| 微信、浏览器等 | 系统默认安装位置  | 不手动整理                     |
| 个人文件       | `~/Documents` 等  | 文档、图片、下载文件           |

## 二、服务

### 服务器IP：192.168.1.102

### 服务端口

| 服务                | 描述                    | 地址                |
| ------------------- | ----------------------- | ------------------- |
| Cockpit             | 服务器管理              | 192.168.1.102:9090  |
| cpolar              | 内网穿透管理            | 192.168.1.102:9200  |
| RocketMQ NameServer | RocketMQ 注册发现       | 192.168.1.102:9876  |
| RocketMQ Broker     | RocketMQ 消息 Broker    | 192.168.1.102:10911 |
| RocketMQ Proxy      | RocketMQ 5.x 客户端入口 | 192.168.1.102:8081  |
| Nacos Console       | Nacos Web 管理控制台    | 192.168.1.102:18080   |
| Nacos Server        | Nacos 注册中心/配置中心 | 192.168.1.102:8848    |
| Nacos gRPC          | Nacos 客户端 gRPC 通信  | 192.168.1.102:9848    |
| MySQL |  | 192.168.1.102:3306 |
| Redis |  | 192.168.1.102:6379 |

### 服务账号密码

| 服务  | 账号  | 密码   |
| ----- | ----- | ------ |
| nacos | nacos | nacos  |
| mysql | root  | 123456 |
| redis | root  | 123456 |



## 三、容器命名规范

建议简单统一。

中间件：

```
mysql
redis
nginx
rmqnamesrv
rmqbroker
rmqproxy
minio
```

业务项目：

```
项目名-模块名
```

例如：

```
huimeng-api
huimeng-web
huimeng-worker
```

## 四、RocketMQ

#### 怎么管理

启动：

```
cd /opt/middleware/rocketmq
docker compose up -d
```

停止：

```
docker compose stop
```

重新启动：

```
docker compose restart
```

查看：

```
docker compose ps
```

实时看 Broker：

```
docker logs -f rmqbroker
```

彻底删除容器：

```
docker compose down
```

注意，即使 `down`：

```
/data/rocketmq/broker/store
```

仍然保留，因此你的消息数据不会因为容器消失而一起消失。

## 五、Nacos

### 怎么管理

以后基本只需要记这一套：

启动：

```
cd /opt/middleware/nacos
docker compose up -d
```

停止：

```
docker compose stop
```

重启：

```
docker compose restart
```

查看：

```
docker compose ps
```

日志：

```
docker logs -f nacos
```

删除容器：

```
docker compose down
```

即使：

```
docker compose down
```

你的 Derby 数据仍然在：

```
/data/nacos/derby-data
```

日志仍然在：

```
/var/log/nacos
```

## 六、Mysql

启动

```
cd /opt/middleware/mysql
docker compose up -d
```

停止

```
docker compose stop
```

重启

```
docker compose restart
```

查看

```
docker compose ps
```

日志

```
docker logs -f mysql
```

删除容器

```
docker compose down
```

最重要的一点：

```
docker compose down
```

**不会删除你的数据库数据。**

因为数据库真正的数据在：

```
/data/mysql
```

而不是容器内部。官方文档也推荐通过宿主机目录挂载 `/var/lib/mysql` 来持久化数据库。
