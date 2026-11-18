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
`│   ├── rustfs/`
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

SSH 用户：`linus-stark`，密码从受控的密码管理工具获取。

root 凭据不得写入本文档或代码仓库。

### 服务端口

| 服务                | 描述                    | 地址                |
| ------------------- | ----------------------- | ------------------- |
| Cockpit             | 服务器管理              | 192.168.1.102:9090  |
| cpolar              | 内网穿透管理            | 192.168.1.102:9200  |
| Tailscale           | 私有组网与远程访问      | 100.83.74.68（SSH：22/TCP） |
| RocketMQ NameServer | RocketMQ 注册发现       | 192.168.1.102:9876  |
| RocketMQ Broker     | RocketMQ 消息 Broker    | 192.168.1.102:10911 |
| RocketMQ Proxy      | RocketMQ 5.x 客户端入口 | 192.168.1.102:8081  |
| RocketMQ Dashboard  | RocketMQ Web 管理控制台 | 192.168.1.102:8082  |
| SkyWalking UI       | 链路追踪 Web 控制台    | 192.168.1.102:18082 |
| SkyWalking OAP gRPC | Java 等 Agent 上报入口 | 192.168.1.102:11800 |
| SkyWalking OAP HTTP | 查询及 HTTP 上报入口   | 192.168.1.102:12800 |
| Nacos Console       | Nacos Web 管理控制台    | 192.168.1.102:18080   |
| Nacos Server        | Nacos 注册中心/配置中心 | 192.168.1.102:8848    |
| Nacos gRPC          | Nacos 客户端 gRPC 通信  | 192.168.1.102:9848    |
| Jenkins             | 持续集成与持续部署      | 192.168.1.102:18081   |
| XXL-JOB Admin       | 任务调度管理控制台      | 192.168.1.102:18084   |
| Nginx               | 前端静态站点入口（选课平台） | 192.168.1.102:80（2026-10-25 起已停止） |
| SmartUniversity API | 选课平台后端（容器 `smart-university`） | 192.168.1.102:18083（2026-10-25 起已停止） |
| MinIO S3 API | 对象存储 S3 接口 | 192.168.1.102:18090 |
| MinIO Console | MinIO Web 控制台 | 192.168.1.102:18091 |
| RustFS S3 API | 对象存储 S3 接口（RustFS，2026-11-17 起） | 192.168.1.102:9000 |
| RustFS Console | RustFS Web 控制台（入口 `/rustfs/console/`） | 192.168.1.102:9001 |
| MySQL |  | 192.168.1.102:3306 |
| PostgreSQL | 关系型数据库（容器 `postgres`） | 192.168.1.102:5432 |
| Redis |  | 192.168.1.102:6379 |
| Ollama | 本地大模型推理服务（systemd 原生） | 0.0.0.0:11434（局域网 `192.168.1.102:11434` 可达） |
| Elasticsearch | Ragent 关键词检索（IK 中文分词） | 127.0.0.1:19200（SSH 转发） |
| LightRAG | 知识图谱 API 与 WebUI | 127.0.0.1:9621（SSH 转发） |
| Neo4j Browser | 图数据库 Web 控制台 | 127.0.0.1:7474（SSH 转发） |
| Neo4j Bolt | 图数据库驱动接口 | 127.0.0.1:7687（SSH 转发） |
| LangFuse | Agent 追踪控制台与 OTLP 入口 | 127.0.0.1:13000（SSH 转发） |

### 服务账号密码

| 服务  | 账号  | 密码   |
| ----- | ----- | ------ |
| nacos | nacos | nacos |
| mysql | `root` | 123456 |
| redis | root | 123456 |
| XXL-JOB | `admin` | `123456` |
| minio | `admin` | `12345678` |
| rustfs | `rustfsadmin` | `rustfsadmin` |
| postgres | `root` | `123456`（库 `toonflow`） |



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
rmqdashboard
skywalking-banyandb
skywalking-oap
skywalking-ui
jenkins
xxl-job-admin
minio
rustfs
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

### 怎么管理

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

### RocketMQ Dashboard

Dashboard 已加入 `/opt/middleware/rocketmq/docker-compose.yml`，与 NameServer 共用 `rocketmq` 容器网络。使用官方镜像 `apacherocketmq/rocketmq-dashboard:2.1.0`，容器名为 `rmqdashboard`，通过 `rmqnamesrv:9876` 连接 NameServer。宿主机仅在 `192.168.1.102:8082` 监听，访问地址为 `http://192.168.1.102:8082/`。

新增服务的关键配置：

```yaml
dashboard:
  image: apacherocketmq/rocketmq-dashboard:2.1.0
  container_name: rmqdashboard
  depends_on:
    - namesrv
  environment:
    JAVA_OPTS: "-Drocketmq.namesrv.addr=rmqnamesrv:9876"
  ports:
    - "192.168.1.102:8082:8082"
  restart: unless-stopped
  networks:
    - rocketmq
```

前提：Docker、Docker Compose 和 RocketMQ NameServer 已正常运行；从局域网访问时，客户端须能连通服务器的 `8082` 端口。Dashboard 提供 Topic、Consumer 等管理操作，只向可信设备开放该端口。

仅启动 Dashboard：

```bash
cd /opt/middleware/rocketmq
docker compose up -d --no-deps dashboard
```

仅停止 Dashboard：

```bash
cd /opt/middleware/rocketmq
docker compose stop dashboard
```

仅重启 Dashboard：

```bash
cd /opt/middleware/rocketmq
docker compose restart dashboard
```

查看状态和日志：

```bash
cd /opt/middleware/rocketmq
docker compose ps dashboard
docker logs --tail 100 rmqdashboard
```

检查页面和集群连接：

```bash
curl -I http://192.168.1.102:8082/
curl -sS http://192.168.1.102:8082/cluster/list.query
```

页面应返回 `HTTP 200`；集群接口返回 `"status":0`，且能看到 Broker。Dashboard 的配置文件已备份为 `/opt/middleware/rocketmq/docker-compose.yml.pre-dashboard`。镜像下载慢时，可临时配置可从服务器访问的代理；本机的 `127.0.0.1:7890` 不能直接作为服务器容器的代理地址。

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

## 七、SSH连接方法

服务器IP：192.168.1.102

SSH 用户：`linus-stark`，认证信息从受控的密码管理工具获取，不得写入本文档。

## 八、SkyWalking

### 部署信息

部署目录为 `/opt/middleware/skywalking`，使用 `docker-compose.yml` 管理三个容器：

| 服务 | 镜像 | 作用 |
| --- | --- | --- |
| `skywalking-banyandb` | `apache/skywalking-banyandb:0.10.3` | 持久化存储 |
| `skywalking-oap` | `apache/skywalking-oap-server:10.4.0-java17` | 接收和分析数据 |
| `skywalking-ui` | `apache/skywalking-ui:10.4.0-java17` | Web 控制台 |

BanyanDB 数据存放在 `/data/skywalking/banyandb`，只在 Compose 内部网络提供服务，不向宿主机发布数据库端口。OAP 对局域网开放 `11800`（gRPC）和 `12800`（HTTP）；UI 地址为 `http://192.168.1.102:18082/`。Java Agent 的后端地址使用 `192.168.1.102:11800`，具体应用仍需单独接入 Agent 才会显示链路数据。

### Java Agent 准备

官方 Java Agent `9.7.0` 已安装在 `/opt/tools/skywalking-java-agent-9.7.0`。Agent 必须附加到被监控应用的 JVM；仅安装文件不会产生链路数据。当前 `/opt/apps` 尚无业务 Java 应用，尚未修改任何现有 Java 服务的启动配置。

普通 JAR 应用的启动参数示例：

```bash
export SW_AGENT_NAME=my-service
export SW_AGENT_COLLECTOR_BACKEND_SERVICES=192.168.1.102:11800
java -javaagent:/opt/tools/skywalking-java-agent-9.7.0/skywalking-agent.jar -jar /opt/apps/my-service/app.jar
```

上述 `my-service` 和 JAR 路径是示例，按实际应用替换。在 Docker 容器中使用时，需将 Agent 目录挂载进容器，并将 `-javaagent` 路径改为容器内路径。设置独立的 `SW_AGENT_NAME`，将 Agent 日志写到应用可写的目录或设置 `SW_LOGGING_OUTPUT=CONSOLE`。接入后访问应用接口，再到 SkyWalking UI 验证服务、JVM 指标和链路。

### 管理命令

前提：Docker 和 Docker Compose 已安装，且服务器有足够的内存和磁盘空间。

启动并等待健康检查：

```bash
cd /opt/middleware/skywalking
docker compose up -d --wait
```

停止全部 SkyWalking 组件：

```bash
cd /opt/middleware/skywalking
docker compose stop
```

重启全部 SkyWalking 组件：

```bash
cd /opt/middleware/skywalking
docker compose restart
```

查看状态与日志：

```bash
cd /opt/middleware/skywalking
docker compose ps
docker logs --tail 100 skywalking-oap
docker logs --tail 100 skywalking-banyandb
docker logs --tail 100 skywalking-ui
```

### 验证与数据保护

```bash
curl -fsS http://192.168.1.102:12800/healthcheck
curl -I http://192.168.1.102:18082/
```

OAP 应返回 `200 OK`，UI 应返回 HTTP 200。停止或删除容器不会自动删除 `/data/skywalking/banyandb` 中的数据；删除该目录会永久丢失链路和指标数据，清理前先备份。UI 未配置登录认证，仅向可信的局域网设备开放 `18082` 端口。

## 九、Jenkins

### 部署信息

Jenkins 使用 Docker Compose 部署，未安装为系统软件包，也没有 `jenkins.service`。截至 `2026-09-29`，部署情况如下：

| 项目 | 当前配置 |
| --- | --- |
| Compose 目录 | `/opt/middleware/jenkins` |
| Compose 文件 | `/opt/middleware/jenkins/docker-compose.yml` |
| 容器名 | `jenkins` |
| 镜像 | `myubuntu/jenkins:lts-jdk21-docker` |
| Jenkins 版本 | `2.568.3` |
| Java 版本 | Eclipse Temurin `21.0.12.1` LTS |
| Web 地址 | `http://192.168.1.102:18081/` |
| 端口映射 | 宿主机 `18081` 映射到容器 `8080` |
| Agent 端口 | 容器内 `50000`，当前未发布到宿主机 |
| 数据目录 | `/data/jenkins` 挂载到容器 `/var/jenkins_home` |
| 重启策略 | `unless-stopped` |
| Docker 能力 | 挂载宿主机 `/var/run/docker.sock`，镜像内安装 Docker CLI 与 Compose |
| 应用目录挂载 | 宿主机 `/opt/apps` 挂载到 Jenkins 容器同路径 |

当前启用了 Jenkins 自带用户数据库、登录认证和 CSRF 防护。`2026-09-29` 检查时，Jenkins 登录页返回 `HTTP 200`，容器状态为 `running`，重启次数为 `0`。

当前 Compose 配置：

```yaml
services:
  jenkins:
    build:
      context: .
      dockerfile: Dockerfile
    image: myubuntu/jenkins:lts-jdk21-docker
    container_name: jenkins
    restart: unless-stopped
    group_add:
      - "973"
    ports:
      - "18081:8080"
    volumes:
      - /data/jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /opt/apps:/opt/apps
```

`group_add` 中的 `973` 是检查时宿主机 Docker socket 的组 ID，不是所有服务器的固定值。重建前执行 `stat -c '%g' /var/run/docker.sock`，并使其与 `group_add` 一致。挂载 Docker socket 等价于让 Jenkins 任务拥有很高的宿主机控制权限，因此只允许受信任的管理员修改任务和流水线脚本，不要运行未经审查的 Pull Request 代码。

自定义 Jenkins 镜像的 `/opt/middleware/jenkins/Dockerfile`：

```dockerfile
FROM jenkins/jenkins:lts-jdk21

USER root

RUN apt-get update \
    && apt-get install -y --no-install-recommends docker-cli docker-compose \
    && rm -rf /var/lib/apt/lists/*

USER jenkins
```

### SmartUniversity 自动部署流水线

#### 当前配置

| 项目 | 当前值 |
| --- | --- |
| Jenkins 任务 | `SmartUniversity` |
| Git 仓库 | `https://github.com/XiaoTianbai520/SmartUniversity.git` |
| 监听分支 | `master` |
| 变更检测 | `H/2 * * * *`，约每 2 分钟由 Jenkins 分散轮询 |
| 应用目录 | `/opt/apps/smart-university` |
| 应用 Compose 文件 | `/opt/apps/smart-university/docker-compose.yml` |
| 应用容器 | `smart-university` |
| 应用镜像 | `smart-university:current` |
| Spring Profile | `prod`，由流水线和 Compose 的 `SPRING_PROFILES_ACTIVE` 明确覆盖 |
| 中间件宿主机 | `192.168.1.102`，由 `SMART_UNIVERSITY_SERVER_IP` 覆盖 `prod` 的回环地址 |
| 对外地址 | `http://192.168.1.102:18083/` |
| 容器端口 | `8080` |
| 构建保留 | 最近 `20` 次 |
| 总超时 | `30` 分钟 |

应用使用多阶段镜像构建。构建阶段基于 Maven 和 JDK 21 执行 `mvn -B -ntp -DskipTests package`，运行阶段只保留 JRE 21 和生成的 JAR，并以 UID `10001` 的非 root 用户启动。流水线固定 `APP_PROFILE=prod`，Compose 将其传入容器的 `SPRING_PROFILES_ACTIVE`；即使仓库中的 `application.yml` 默认是 `dev`，服务器部署仍会使用 `prod`。仓库的 `application-prod.yml` 假设应用直接运行在宿主机，默认服务地址为 `127.0.0.1`；当前应用实际运行在容器中，因此部署层使用 `SMART_UNIVERSITY_SERVER_IP=192.168.1.102` 覆盖该属性，使容器能够访问宿主机上的 MySQL、Redis 和 RocketMQ。`docker-compose.yml` 将宿主机 `18083` 映射到容器 `8080`，并通过容器内 TCP 连接检查 `8080` 是否可用。

#### 完整流程

```text
GitHub master 发生变化
        ↓
Jenkins SCM 每约 2 分钟轮询一次
        ↓
Checkout：清空工作区，浅克隆最新一次提交，失败最多重试 3 次
        ↓
Build image：构建 smart-university:candidate-<BUILD_NUMBER>
        ↓
保存旧版本：current → previous（若 current 存在）
        ↓
发布候选版本：candidate → current
        ↓
Docker Compose 强制重建容器并等待健康检查，最长 120 秒
        ├── 成功：删除 candidate 标签，构建成功
        └── 失败：输出最近 100 行应用日志，previous → current，重建旧版本并将构建标记为失败
```

流水线还启用了以下保护：

- `disableConcurrentBuilds()`：同一任务不会同时部署两个版本。
- `timeout(30 分钟)`：避免网络或构建异常无限占用执行器。
- `buildDiscarder(20)`：只保留最近 20 次构建记录。
- `retry(3)`、浅克隆和不拉取标签：降低 GitHub 临时网络中断的影响和下载量。
- `post always`：无论成功或失败，都尝试清理当前构建的 `candidate` 镜像标签。
- 部署失败时自动回滚，但只有已存在 `smart-university:previous` 时才有可回滚版本；首次部署失败没有旧版本可用。

`pollSCM` 不是严格的整点定时器。`H/2 * * * *` 由 Jenkins 根据任务散列到每 2 分钟周期中的某个时间点；远程提交没有变化时只记录 `No changes`，不会执行构建和部署。

#### 日常验证

先确认 Jenkins 自身和应用状态，不要一上来就重启或重新部署：

```bash
cd /opt/middleware/jenkins
docker compose config --quiet
docker compose ps
docker inspect -f '{{.Name}} {{.State.Status}} {{if .State.Health}}{{.State.Health.Status}}{{end}} restarts={{.RestartCount}}' \
  jenkins smart-university
docker inspect -f '{{range .Config.Env}}{{println .}}{{end}}' smart-university | \
  grep -E '^(SPRING_PROFILES_ACTIVE=prod|SMART_UNIVERSITY_SERVER_IP=192\.168\.1\.102)$'
curl -fsS -o /dev/null -w 'jenkins=%{http_code}\n' \
  http://127.0.0.1:18081/login
curl -fsS -o /dev/null -w 'app=%{http_code}\n' \
  http://127.0.0.1:18083/
```

正常结果应是 Jenkins 容器为 `running`，应用容器为 `running healthy`，两个 HTTP 状态码均为 `200`。如果应用没有根路径接口，`404` 不一定表示部署失败，应改为请求项目实际的健康检查接口；容器健康状态仍应为 `healthy`。

确认自动触发器是否在工作：

```bash
grep -A 4 -B 1 'SCMTrigger' \
  /data/jenkins/jobs/SmartUniversity/config.xml
tail -n 100 /data/jenkins/jobs/SmartUniversity/scm-polling.log
```

配置中应出现 `<spec>H/2 * * * *</spec>`。轮询日志中的 `No changes` 表示轮询成功但远程提交未变化；`Changes found` 才会触发新构建。也可以在 Jenkins 页面进入 `SmartUniversity → 立即构建`，手动验证完整流程，但这会实际构建镜像并重新部署应用。

#### 分层调试

按“触发 → 拉取 → 构建 → 部署 → 运行”的顺序定位，避免用反复重启 Jenkins 掩盖真实故障。

1. 检查轮询和远程仓库。

```bash
tail -n 100 /data/jenkins/jobs/SmartUniversity/scm-polling.log
docker exec jenkins git ls-remote -h \
  https://github.com/XiaoTianbai520/SmartUniversity.git master
```

如果 `git ls-remote` 超时或出现 TLS、DNS 错误，先检查服务器到 GitHub 的网络和 DNS。流水线会自动重试拉取 3 次；持续失败时不要通过关闭 TLS 校验解决。

2. 检查 Jenkins 是否能操作 Docker。

```bash
stat -c 'socket gid=%g mode=%a' /var/run/docker.sock
docker exec jenkins id
docker exec jenkins docker version
docker exec jenkins docker compose version
```

若出现 `permission denied`，检查 socket 组 ID 是否与 Compose 的 `group_add` 一致，然后再评估是否需要重建 Jenkins 容器。不要把 socket 改成全员可写的 `666` 权限。

3. 查看 Jenkins 构建日志。

在 Jenkins 页面进入 `SmartUniversity → 构建历史 → Console Output`。重点判断失败发生在哪个阶段：

- `Checkout`：通常是 GitHub 网络、DNS、分支名或仓库权限问题。
- `Build image`：通常是 Maven 依赖下载、编译、测试配置、Dockerfile 或磁盘空间问题。
- `Deploy`：通常是端口冲突、应用启动异常、外部依赖不可用或健康检查超时。

服务器上也可查看构建目录，但其中可能包含敏感输出，不要把整个日志复制到公开位置：

```bash
ls -la /data/jenkins/jobs/SmartUniversity/builds
tail -n 200 /data/jenkins/jobs/SmartUniversity/builds/<build-number>/log
```

4. 检查镜像构建与磁盘。

```bash
docker image ls 'smart-university*'
docker system df
df -h / /data /opt
```

不要为了排障直接执行 `docker system prune`。该命令可能影响其他服务，清理前必须确认准确目标和恢复影响。

**2026-10-30 实测的安全清理清单**（当天回收 21GB，根分区从 60G 已用降到 39G）：

| 可安全清理 | 命令 | 当天收益 |
| --- | --- | --- |
| 悬空镜像（Jenkins 构建残留） | `docker image prune -f` | 16.93GB |
| 构建缓存 | `docker builder prune -f` | 3.32GB |
| 已废弃/被替代的 tag 镜像 | `docker rmi <精确 tag>`，**先** `docker ps -a --filter ancestor=<img>` 确认零引用 | 3.8GB |
| 一次性构建容器（`maven:*` 跑完就退出） | `docker rm <名字>` | 57MB |

**不要清的东西**（清理前必须逐条确认）：

- `smart-university:previous` —— 发布回滚的唯一样本，删了 `docker tag ... :current` 的回滚路径就断了。（`huimeng-api:prev-*` 已于 `2026-10-31` 随 huimeng 停用一并删除，属主动放弃回滚点）
- 已停止但仍在用的容器：`smart-university`（`Exited 143` 是主动停服，恢复靠 `docker compose start`）、`skywalking-*`（监控栈，停着但随时要起）。
- `eclipse-temurin:21-jre-jammy`、`maven:3.9.11-*` —— 是 Dockerfile 的 `FROM` 基础镜像，删了下次构建要重新拉。
- `huimeng-m3-maven` 具名卷已于 `2026-10-31` 删除；其它项目的 Maven 缓存卷同理，删前先确认没有构建任务在跑。

判断某个镜像到底有没有用，不要看 tag 名字猜，直接查引用：

```bash
docker ps -a --filter ancestor=<repo:tag> --format '{{.Names}}'
```

返回空才说明真没有容器在用。

5. 检查应用部署和启动日志。

```bash
cd /opt/apps/smart-university
docker compose config --quiet
docker compose ps
docker compose logs --tail 200 smart-university
docker inspect -f '{{json .State.Health}}' smart-university
ss -lntp | grep ':18083'
curl -v http://127.0.0.1:18083/
```

应用容器不断重启时，优先从日志中确认数据库、Redis、RocketMQ、Nacos 等外部依赖是否可达，以及环境配置是否正确。不要只增加健康检查等待时间；如果应用实际无法启动，延长时间不会解决问题。

6. 必要时在应用目录复现部署。

以下命令会重新创建应用容器，会造成短暂中断，只在已确认当前镜像和影响后执行：

```bash
cd /opt/apps/smart-university
docker compose up -d --force-recreate --wait --wait-timeout 120
docker compose ps
docker compose logs --tail 100 smart-university
```

#### 回滚与恢复

流水线部署失败时会自动尝试回滚。若构建已显示失败，先检查应用当前是否已恢复为 `healthy`，不要立即覆盖现场。

手动回滚会重新创建容器并造成短暂中断。执行前先确认 `previous` 镜像存在，并记录当前镜像 ID：

```bash
docker image inspect smart-university:current \
  --format 'current={{.Id}} created={{.Created}}'
docker image inspect smart-university:previous \
  --format 'previous={{.Id}} created={{.Created}}'
```

获得管理员确认后，执行：

```bash
docker tag smart-university:previous smart-university:current
docker compose --project-directory /opt/apps/smart-university \
  --file /opt/apps/smart-university/docker-compose.yml \
  up -d --force-recreate --wait --wait-timeout 120
curl -fsS http://127.0.0.1:18083/ > /dev/null
```

`previous` 只保存最近一次部署前的镜像，不是长期版本库。需要长期保留版本时，应使用不可变的提交 SHA 标签和受控镜像仓库，不要依赖 `current`、`previous` 两个浮动标签。

#### 日志管理

应用日志不只依赖 `docker logs`。Compose 将容器目录 `/var/log/apps/smart-university` 挂载到宿主机同路径，因此流水线强制重建容器后，历史日志仍会保留。

| 日志 | 宿主机位置 | 内容 |
| --- | --- | --- |
| 应用日志 | `/var/log/apps/smart-university/application.log` | Spring Boot 启动、业务和异常日志 |
| 应用归档 | `/var/log/apps/smart-university/application.<日期>.<序号>.log.gz` | 按天和 `20 MB` 大小滚动的历史日志 |
| HTTP 访问日志 | `/var/log/apps/smart-university/access*.log` | 客户端 IP、请求方法、URI、HTTP 状态码、响应字节和耗时 |
| Docker 控制台日志 | Docker `json-file` | 当前容器标准输出，单文件最多 `20 MB`，最多保留 `5` 个文件 |

应用归档和访问日志保留 `14` 天，应用归档总大小上限为 `2 GB`。访问日志不记录 Authorization、Cookie、请求体或响应体，避免把 Token、密码和业务数据写入日志。Docker 日志轮转只限制当前容器的控制台日志，并不代替宿主机持久化日志。

实时查看：

```bash
docker logs -f --tail 200 smart-university
tail -F /var/log/apps/smart-university/application.log
tail -F /var/log/apps/smart-university/access*.log
```

按时间或关键词排查：

```bash
docker logs --since 30m smart-university
grep -nE 'ERROR|Exception|Caused by' \
  /var/log/apps/smart-university/application.log | tail -n 100
grep ' 5[0-9][0-9] ' \
  /var/log/apps/smart-university/access*.log | tail -n 100
```

查看占用和轮转状态：

```bash
du -sh /var/log/apps/smart-university
find /var/log/apps/smart-university -maxdepth 1 -type f \
  -printf '%TY-%Tm-%Td %TH:%TM %10s %f\n' | sort
docker inspect -f '{{json .HostConfig.LogConfig}}' smart-university
```

不要直接删除正在写入的 `application.log` 或当前访问日志。需要紧急释放空间时，先确认具体文件、应用写入状态和保留要求；优先删除已压缩的过期归档，并保留故障时间段日志。

### 首次部署或重建

前提：Docker 和 Docker Compose 已安装。首次使用全新数据目录时，先创建目录并保证容器内 UID `1000` 可以写入：

```bash
sudo install -d -m 755 /opt/middleware/jenkins
sudo install -d -m 755 -o 1000 -g 1000 /data/jenkins
```

将上面的 Compose 配置保存为 `/opt/middleware/jenkins/docker-compose.yml`，然后启动并观察初始化日志：

```bash
cd /opt/middleware/jenkins
docker compose up -d
docker compose ps
docker logs --tail 100 jenkins
```

仅在全新实例首次初始化时，使用以下命令读取一次性初始管理员密码：

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

该密码属于敏感信息，不要写入本文档、代码仓库或命令输出记录。完成初始化后创建正式管理员账号，并将凭据保存到受控的密码管理工具中。已有实例使用现有管理员账号登录，不要重新执行初始化流程。

### 管理命令

启动：

```bash
cd /opt/middleware/jenkins
docker compose up -d
```

停止：

```bash
cd /opt/middleware/jenkins
docker compose stop
```

重启：

```bash
cd /opt/middleware/jenkins
docker compose restart
```

查看状态和资源占用：

```bash
cd /opt/middleware/jenkins
docker compose ps
docker stats --no-stream jenkins
```

查看最近日志或持续跟踪日志：

```bash
docker logs --tail 100 jenkins
docker logs -f jenkins
```

删除容器：

```bash
cd /opt/middleware/jenkins
docker compose down
```

`docker compose down` 会删除容器和 Compose 网络，但不会删除宿主机的 `/data/jenkins`。不要直接删除该目录，其中包含任务配置、插件、构建记录、用户配置和加密密钥。

### 验证与检查

检查登录页：

```bash
curl -I http://192.168.1.102:18081/login
```

响应头应包含 `HTTP/1.1 200 OK` 和 `X-Jenkins`。查看容器内 Jenkins 与 Java 版本：

```bash
curl -sSI http://192.168.1.102:18081/login | grep -i '^X-Jenkins:'
docker exec jenkins java -version
```

当前 `18081` 绑定在 `0.0.0.0` 和 `::`，并且服务器的 UFW 未启用，因此应仅在可信局域网中访问，不要直接暴露到公网。若只需从固定内网地址访问，优先将 Compose 端口映射改为 `192.168.1.102:18081:8080`，或配置防火墙白名单。若需公网访问，应在反向代理上配置 HTTPS、访问控制和可信来源限制。

Jenkins 系统根 URL 曾配置为 `http://192.168.1.102:18081/instance/`，而实际直接入口是 `http://192.168.1.102:18081/`。排查邮件、Webhook 或构建页面的绝对链接异常时，先在“系统管理 → 系统配置 → Jenkins URL”核对当前值；若服务器没有 `/instance/` 反向代理或 `--prefix` 配置，应使用实际根地址。

### 数据备份与升级

`2026-09-28` 已创建一次完整备份 `/data/backup/jenkins/jenkins-20260928-134119.tar.gz`，当时已通过压缩包读取和 SHA-256 校验。备份只代表该时间点，不能替代后续变更前的新备份。升级镜像或修改关键配置前，应先停止 Jenkins 并备份数据目录，避免复制过程中产生不一致数据：

```bash
set -euo pipefail
cd /opt/middleware/jenkins
backup_file="/data/backup/jenkins/jenkins-$(date +%Y%m%d-%H%M%S).tar.gz"
sudo install -d -m 700 /data/backup/jenkins
docker compose stop
trap 'docker compose start' EXIT
sudo tar -C /data -czf "$backup_file" jenkins
sudo chmod 600 "$backup_file"
sudo test -s "$backup_file"
sudo tar -tzf "$backup_file" > /dev/null
sudo sha256sum "$backup_file" | sudo tee "$backup_file.sha256" > /dev/null
trap - EXIT
docker compose start
```

上述命令会检查归档非空、验证压缩包可读取并生成 SHA-256 校验文件；任何步骤失败时，退出陷阱会尝试重新启动 Jenkins。仍需检查命令退出状态和 `docker compose ps`，不能仅凭文件存在就认定备份成功。备份文件包含凭据和密钥，只允许管理员读取，不要提交到代码仓库。服务器本机备份不能防范整机或磁盘故障，应定期将加密副本保存到其他受控存储，并执行恢复演练。

恢复会覆盖现有 Jenkins 配置和数据，属于破坏性操作。恢复前必须停止容器、再次备份当前 `/data/jenkins`，校验备份文件，并由管理员明确确认目标归档和恢复时间点。

自定义镜像的基础镜像使用浮动标签 `jenkins/jenkins:lts-jdk21`。升级前记录当前 Jenkins 和插件版本并完成备份，然后显式拉取基础镜像并重新构建自定义镜像，再检查登录、Docker CLI、任务配置、插件兼容性和构建执行情况：

```bash
cd /opt/middleware/jenkins
docker compose build --pull jenkins
docker compose up -d jenkins
docker compose ps
docker logs --tail 100 jenkins
docker exec jenkins docker version
docker exec jenkins docker compose version
```

## 十、Tailscale

### 部署信息

Tailscale 使用官方 APT 软件源安装，由 `systemd` 管理 `tailscaled.service`。截至 `2026-09-27`，部署情况如下：

| 项目 | 当前配置 |
| --- | --- |
| Tailscale 版本 | `1.102.4` |
| Ubuntu 软件源代号 | `resolute` |
| APT 源 | `/etc/apt/sources.list.d/tailscale.list` |
| 软件源密钥 | `/usr/share/keyrings/tailscale-archive-keyring.gpg` |
| systemd 服务 | `tailscaled.service`，已启用并运行 |
| Tailscale IPv4 | `100.83.74.68` |
| Tailscale IPv6 | `fd7a:115c:a1e0::e92e:4a45` |
| MagicDNS | `linus-stark-nhx0db-de.taild30eeb.ts.net` |
| 状态目录 | `/var/lib/tailscale`，权限 `700 root:root` |
| 状态文件 | `/var/lib/tailscale/tailscaled.state`，权限 `600 root:root` |

节点当前在线且健康，`tailscale netcheck` 显示 UDP 通信可用。未启用出口节点、子网路由和 Tailscale SSH；远程登录仍由服务器现有 OpenSSH 提供。

### 安装方法

以下命令适用于当前 Ubuntu `26.04`（`resolute`）。先添加官方签名密钥和软件源：

```bash
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/resolute.noarmor.gpg \
  -o /tmp/tailscale-archive-keyring.gpg
curl -fsSL https://pkgs.tailscale.com/stable/ubuntu/resolute.tailscale-keyring.list \
  -o /tmp/tailscale.list

tailscale_key_fingerprint="$(gpg --show-keys --with-colons \
  /tmp/tailscale-archive-keyring.gpg | \
  awk -F: '$1 == "fpr" { print $10; exit }')"
test "$tailscale_key_fingerprint" = "2596A99EAAB33821893C0A79458CA832957F5868" || {
  echo "Tailscale 软件源密钥指纹不匹配，停止安装" >&2
  exit 1
}

grep -Fxq \
  'deb [signed-by=/usr/share/keyrings/tailscale-archive-keyring.gpg] https://pkgs.tailscale.com/stable/ubuntu resolute main' \
  /tmp/tailscale.list || {
  echo "Tailscale 软件源内容不匹配，停止安装" >&2
  exit 1
}

sudo install -d -o root -g root -m 0755 /usr/share/keyrings /etc/apt/sources.list.d
sudo install -o root -g root -m 0644 \
  /tmp/tailscale-archive-keyring.gpg \
  /usr/share/keyrings/tailscale-archive-keyring.gpg
sudo install -o root -g root -m 0644 \
  /tmp/tailscale.list \
  /etc/apt/sources.list.d/tailscale.list
```

官方软件源签名密钥指纹为：

```text
2596 A99E AAB3 3821 893C 0A79 458C A832 957F 5868
```

安装并登录：

```bash
sudo apt-get update
sudo apt-get install -y tailscale
sudo tailscale up
systemctl is-enabled tailscaled
systemctl is-active tailscaled
rm -f /tmp/tailscale-archive-keyring.gpg /tmp/tailscale.list
```

`tailscale up` 会输出一次性授权地址。使用受信任的 Tailscale 账号完成授权，不要把授权地址或 Auth Key 写入本文档、脚本或代码仓库。

安装以 [Tailscale Linux 官方文档](https://tailscale.com/docs/install/linux) 和 [官方稳定版软件源](https://pkgs.tailscale.com/stable/) 为准。升级 Ubuntu 发行版后，应同步检查并更新软件源代号，不能继续照搬 `resolute`。

### 下载代理

正常情况下优先直连官方软件源。只有直连失败或速度明显过慢，并且服务器本机代理已在 `127.0.0.1:7890` 监听时，才临时使用代理。

下面两条带 `-x` 的 `curl` 命令只替换“安装方法”中的两条直接下载命令：

```bash
ss -lnt | grep '127.0.0.1:7890'

curl -x http://127.0.0.1:7890 -fsSL \
  https://pkgs.tailscale.com/stable/ubuntu/resolute.noarmor.gpg \
  -o /tmp/tailscale-archive-keyring.gpg
curl -x http://127.0.0.1:7890 -fsSL \
  https://pkgs.tailscale.com/stable/ubuntu/resolute.tailscale-keyring.list \
  -o /tmp/tailscale.list
```

下载完成后，必须返回“安装方法”继续执行 `tailscale_key_fingerprint` 指纹校验、`grep -Fxq` 软件源内容校验和两个 `sudo install` 命令。上述步骤全部成功后，才能继续执行 APT 安装。

使用以下带代理参数的命令替换“安装方法”中的普通 `apt-get` 命令：

```bash
sudo apt-get \
  -o Acquire::http::Proxy="http://127.0.0.1:7890" \
  -o Acquire::https::Proxy="http://127.0.0.1:7890" \
  update
sudo apt-get \
  -o Acquire::http::Proxy="http://127.0.0.1:7890" \
  -o Acquire::https::Proxy="http://127.0.0.1:7890" \
  install -y tailscale
```

代理参数只对当前命令生效，不要把代理永久写入 Tailscale 服务配置。检查时 `127.0.0.1:7890` 未监听，但官方软件源直连正常，因此本次安装没有使用代理。

### 管理与验证

查看服务和连接状态：

```bash
systemctl status tailscaled --no-pager
tailscale status
tailscale ip -4
tailscale ip -6
tailscale netcheck
```

如果当前 SSH 会话通过 Tailscale 地址建立，重启、断开、注销或升级 Tailscale 都可能立即切断会话。执行以下变更前，必须先确认局域网 SSH、Cockpit、物理控制台或其他带外管理通道可用。

重启服务：

```bash
sudo systemctl restart tailscaled
systemctl is-active tailscaled
```

临时断开和重新连接 Tailnet：

```bash
sudo tailscale down
sudo tailscale up
```

`tailscale down` 会停止 Tailscale 网络连接，但保留节点登录信息。以下命令会注销节点并使当前登录失效，再次连接时需要重新授权：

```bash
sudo tailscale logout
```

查看并安装稳定版更新。升级过程可能重启 `tailscaled`，远程操作前同样要准备备用连接：

```bash
sudo apt-get update
sudo apt-get install --only-upgrade tailscale
tailscale version
```

### 通过 Tailscale 连接 SSH

客户端设备必须加入同一个 Tailnet，并被 Tailnet ACL 或 Grants 允许访问该节点的 `22` 端口。通过 Tailscale IPv4 连接：

```bash
ssh linus-stark@100.83.74.68
```

启用 MagicDNS 的客户端也可以使用域名：

```bash
ssh linus-stark@linus-stark-nhx0db-de.taild30eeb.ts.net
```

当前没有启用 Tailscale SSH，因此认证方式仍是 Ubuntu OpenSSH 的密钥或密码。Tailscale 只提供私有网络路径，不会自动关闭局域网地址或其他公网入口；SSH 安全策略和防火墙仍需单独维护。

## 十一、XXL-JOB

### 部署信息与登录

截至 `2026-10-04`，XXL-JOB Admin 使用官方 `3.4.2` 镜像，通过 `/opt/middleware/xxl-job/docker-compose.yml` 管理，容器名为 `xxl-job-admin`。镜像固定为 `sha256:fa6ad9343f41be414446685f00c28456444577c8209948760c8089b493c39e35`。登录地址为 `http://192.168.1.102:18084/auth/login`，端口只绑定服务器局域网地址。账号为 `admin`，密码为 `123456`；已通过登录接口验证成功。

数据库实际名称是 `xxl_job`，不是 `xxl-job`。Admin 经 Docker 的 `mysql_default` 网络连接现有 MySQL 容器，使用仅有 `xxl_job` 库读写权限的 `xxl_job_app` 账号。数据库密码和执行器访问令牌保存在服务器的 `/opt/middleware/xxl-job/conf/application.properties`，该文件权限为 `600 root:root`，不得复制到仓库。应用日志保存在 `/var/log/apps/xxl-job`。

现有数据库原有 8 张表和 2 个任务，其中 1 个任务处于启用状态。部署前已备份到 `/data/backup/xxl-job/xxl_job-20261004-183445.sql.gz` 并通过压缩包校验。按[官方升级说明](https://github.com/xuxueli/xxl-job/releases)完成 `xxl_job_user` 的 `token` 列和密码字段长度迁移；原 `admin` 账号使用的 MD5 摘要已改为 SHA-256 摘要，登录密码保持 `123456`。更改摘要前的用户表另备份于 `/data/backup/xxl-job/xxl_job_user-pre-sha256-20261004-184732.sql`。这些备份只允许管理员读取。

`123456` 是易猜测密码，当前仅适合受控局域网使用。不要把管理端口映射到公网；后续应在密码管理工具中保存新密码并完成轮换，同时从本文档移除明文密码。

现有任务配置过失败邮件，但没有可用的 SMTP 凭据。为避免历史失败日志持续重试发信，已暂时清空 1 个任务的邮件地址，并将 2888 条历史失败告警标记为无需重试；原地址保存在 `/data/backup/xxl-job/alarm-email-20261004-183743.sql`。任务启用状态未改变。恢复邮件前先配置并验证 SMTP，再由管理员备份当前数据库并执行该受限 SQL 文件。历史告警不会自动重发。

检查时没有已注册的执行器，管理页面可登录不代表任务已执行成功。执行器应配置管理地址 `http://192.168.1.102:18084`，并通过安全方式取得与 Admin 一致的 `xxl.job.accessToken`。

### 本机开发接入执行器

只在需要调试定时任务时启用本机执行器。前提是开发机能连接 `192.168.1.102:18084`，且服务器能回连开发机的 `19999/TCP`。普通功能开发保持 `enabled: false`；启用后，数据库中已有的启用任务可能立即派发到本机。

1. 在开发机确认可连接调度中心。Windows PowerShell 示例：

```powershell
Test-NetConnection 192.168.1.102 -Port 18084
```

确认 `TcpTestSucceeded` 为 `True`。用 `ipconfig` 找开发机局域网 IPv4；若双方通过 Tailscale 互通，也可用 `tailscale ip -4` 获取开发机的 Tailscale IPv4。选择服务器能访问的地址，不使用 `127.0.0.1`。

2. 从服务器受限配置中读取实际访问令牌，放入开发工具的 `XXL_JOB_ACCESS_TOKEN` 环境变量或受控的密钥管理工具。以下命令只在自己的终端执行；输出不要贴到聊天、日志或仓库：

```powershell
ssh myServer "sudo sed -n 's/^xxl.job.accessToken=//p' /opt/middleware/xxl-job/conf/application.properties"
```

服务器使用随机生成的令牌，开发配置中的 `default_token` 不能直接沿用。

3. 需要测试调度时，按项目现有的 `xxl-job` 配置结构填写；将 `<开发机可达IP>` 换成第 1 步确认的地址：

```yaml
xxl-job:
  enabled: true
  access-token: ${XXL_JOB_ACCESS_TOKEN}
  admin:
    addresses: http://192.168.1.102:18084
  executor:
    application-name: one-coupon-merchant-admin
    ip: <开发机可达IP>
    log-retention-days: 30
    port: 19999
```

`admin.addresses` 使用根地址，不加旧的 `/xxl-job-admin` 后缀；[XXL-JOB 3.4.1 发布说明](https://github.com/xuxueli/xxl-job/releases/tag/3.4.1)明确要求移除该后缀。`application-name` 必须与调度中心内的执行器 AppName 一致。上述 `xxl-job.access-token` 是当前项目的配置格式；若改用官方 `3.4.2` 示例配置，应核对项目配置类是否将其传给官方的 `xxl.job.admin.accessToken`，不要只凭 YAML 名称判断已生效。[官方执行器配置示例](https://github.com/xuxueli/xxl-job/blob/3.4.2/xxl-job-executor-samples/xxl-job-executor-sample-springboot/src/main/resources/application.properties)

4. 启动本机应用，确认 `19999` 端口已监听，并从服务器检查回连。下面的 IP 变量须替换为真实地址；HTTP 返回认证错误仍表明网络已连通，连接超时或拒绝则应检查监听地址、Windows 防火墙或 Tailscale 访问规则。

```bash
executor_ip='<开发机可达IP>'
curl --connect-timeout 3 -sS -o /dev/null -w '%{http_code}\n' \
  "http://${executor_ip}:19999/beat"
```

最后在 XXL-JOB 管理页面检查 `one-coupon-merchant-admin` 是否注册，再手动触发一项允许在开发机执行的任务并查看调度日志。测试结束后把 `enabled` 改回 `false`，重启本机应用以停止执行器注册。

### 管理命令

前提：Docker、Docker Compose、MySQL 容器和 `mysql_default` 网络均正常。启动：

```bash
cd /opt/middleware/xxl-job
docker compose config --quiet
docker compose up -d
```

停止调度中心；此操作不会停止执行器进程：

```bash
cd /opt/middleware/xxl-job
docker compose stop xxl-job-admin
```

重启调度中心，会短暂中断调度：

```bash
cd /opt/middleware/xxl-job
docker compose restart xxl-job-admin
```

查看状态、日志及登录页：

```bash
cd /opt/middleware/xxl-job
docker compose ps
docker compose logs --tail 100 xxl-job-admin
curl -sS -o /dev/null -w '%{http_code}\n' \
  http://192.168.1.102:18084/auth/login
```

登录页应返回 `200`。`docker compose down` 会删除 Admin 容器并停止调度，但不会删除宿主机上的配置、日志、备份及 MySQL 中的 `xxl_job` 数据。执行前先用 `docker compose config` 核实挂载；不要使用 `down -v` 或删除上述数据目录。

## 十二、前端静态站点（高校智慧教务选课平台）

> **当前状态（2026-10-25）：已停止。**
> 选课平台前端 Nginx 与后端 `smart-university` 容器均已关闭，端口 `80`、`18083` 无监听，cpolar 对应隧道回源返回 `404`。
> - 前端：`systemctl stop nginx` + `systemctl disable nginx`（重启不会自启），静态文件与站点配置**全部保留**在 `/var/www/course-selection-web`、`/etc/nginx/sites-available/course-selection-web.conf`。
> - 后端：`cd /opt/apps/smart-university && docker compose stop`，容器为 `Exited (143)`，`restart: unless-stopped` 策略下不会自启，镜像 `smart-university:current` 与数据卷均保留。
>
> 恢复方式：
>
> ```bash
> systemctl enable --now nginx
> cd /opt/apps/smart-university && docker compose start
> ```
>
> 本节以下内容是停止前的完整部署记录，恢复后仍然适用。

### 部署信息

截至 `2026-10-21`，前端站点使用 APT 安装的 Nginx 直接托管静态文件，未使用 Docker。

| 项目 | 当前配置 |
| --- | --- |
| 对外地址 | `http://192.168.1.102/` |
| 静态文件目录 | `/var/www/course-selection-web` |
| 站点配置 | `/etc/nginx/sites-available/course-selection-web.conf`，软链接到 `sites-enabled` |
| Nginx 版本 | `1.28.3`（Ubuntu 26.04 官方源），systemd 管理，`2026-10-25` 起已 `stop` 并 `disable` |
| 监听端口 | `80`（IPv4 + IPv6），作为 `default_server`；原 `default` 站点已移除 |
| 后端接口 | `/api/` 反向代理到 `http://127.0.0.1:18083`（`smart-university` 容器） |
| 公网地址 | `https://571d6917.r36.cpolar.top`（cpolar 免费版随机二级域名） |
| 公网隧道 | cpolar 隧道 `course-selection-web` → `http 80`，定义在 `/usr/local/etc/cpolar/cpolar.yml` |
| 访问日志 | `/var/log/nginx/course-selection-web.access.log` |
| 错误日志 | `/var/log/nginx/course-selection-web.error.log` |
| 源码位置 | 开发机 `D:\course-selection-web - new`（Vue 3 + TypeScript + Element Plus + Vite） |

站点上线前服务器上没有安装 Nginx（`/etc/nginx` 不存在、无 `nginx.service`、无同名容器），端口 `80` 空闲，因此新增原生安装而非容器化部署。日志使用 `/var/log/nginx`，可被现有的 `/etc/logrotate.d/nginx` 自动轮转。

### 路由与缓存策略

- `vue-router` 使用 history 模式，`location /` 通过 `try_files $uri $uri/ /index.html` 回退，避免直接访问或刷新子路由时出现 404。
- `/assets/` 为带内容哈希的构建产物，返回 `Cache-Control: public, max-age=2592000, immutable`，并关闭该路径的访问日志。
- `index.html` 返回 `no-cache, no-store, must-revalidate`，保证发版后入口 HTML 立即生效。
- 对 `js` / `css` / `json` / `svg` 等启用 gzip，`gzip_min_length 1k`。
- `/api/` 反向代理保持原始 URI，不做路径重写；前端 `baseURL` 默认 `/api/v1`，与后端契约前缀一致。
- 已开启 `server_tokens off`，拒绝访问以点开头的隐藏文件。

### 发版流程

1. 本地构建：

```bash
cd "D:\course-selection-web - new"
npm run build
```

2. 覆盖上传 `dist/` 内容到 `/var/www/course-selection-web/`。在 Linux 机器上可直接用 `rsync`：

```bash
rsync -av --delete dist/ root@192.168.1.102:/var/www/course-selection-web/
```

3. 核对权限：文件 `644 root:root`、目录 `755`，`www-data` 需可读。
4. 静态文件替换后无需 reload；只有修改站点配置时才执行 `nginx -t && systemctl reload nginx`。

注意：Vite 构建会先清空 `dist` 目录，上传时务必使用覆盖模式并清理旧哈希文件，不要把多版本 `assets` 混在同一目录。

### 公网访问（cpolar 内网穿透）

> **当前状态（2026-10-25）：隧道配置保留，但回源已失效。**
> 前端与后端停止后，`https://571d6917.r36.cpolar.top`（80）与 `https://573564d7.r36.cpolar.top`（18083）均返回 `404`。
> 这两条隧道仍占着免费版 4 条额度中的 2 条；若确定不再需要公网访问，把 `course-selection-web` 与 `SmartUniversity` 两个块的 `start_type` 改为 `disable` 后 `systemctl restart cpolar` 即可释放，配置和绑定的域名不会丢。

服务器上已有 cpolar `3.3.12`，由 `cpolar.service` 以 `nobody` 用户运行，启动参数为 `start-all -daemon=on -config=/usr/local/etc/cpolar/cpolar.yml -log=/var/log/cpolar/access.log`。本次在配置中新增一条指向本机 `80` 的 HTTP 隧道：

```yaml
tunnels:
  course-selection-web:
    id: 5f22f78e-a99f-4f46-a5ad-9dfdba0fbf86
    proto: http
    addr: "80"
    bind_tls: both
    start_type: enable
```

- 公网地址为 `https://571d6917.r36.cpolar.top`，`bind_tls: both` 使 HTTP 与 HTTPS 同时可用。
- 前端页面与 `/api/` 接口共用同一域名，接口仍由 nginx 转发到 `127.0.0.1:18083`，因此浏览器侧不存在跨域问题。
- 隧道对象已把在线分配到的 `id` 固化进配置：重启服务后域名保持不变。若删掉 `id`，cpolar 每次启动可能新建一个隧道对象，既更换域名又占用免费版 4 条隧道配额。
- 查看当前隧道与分配域名：浏览器打开 `http://192.168.1.102:9200`，或读取启动回执：

```bash
grep -a 'RespStartTunnel' /var/log/cpolar/access.log | tail -2 | grep -aoE 'TunnelName\\":\\"[^\\]+.*?PublicUrl'
grep -a 'PublicUrl' /var/log/cpolar/access.log | tail -3
```

管理命令：

```bash
cpolar list                 # 列出配置中的隧道名
systemctl restart cpolar    # 修改 cpolar.yml 后必须重启才会生效
systemctl status cpolar --no-pager
```

注意事项：

- 免费版带宽与并发有限、二级域名随机；需要固定域名须在 cpolar 控制台配置保留域名（付费）。
- 站点连同 `/api/` 已对公网开放，任何拿到地址的人都能打开登录页并调用接口。如需限制，可在 nginx 上加 basic auth，或给隧道加 `-httpauth=user:pass`。
- 同一账号下还长期开着 `ssh → 22` 的 TCP 隧道，等于把服务器 SSH 暴露在公网，应仅作临时调试用，用完及时删除。
- `cpolar.yml` 中含 authtoken，权限需为 `640` 且**属主必须是运行用户 `nobody:nogroup`**。切勿设成 `640 root:nogroup`——`nobody` 属组为 `nogroup`，那样只能读不能写，而网页仪表盘登录时要把账号信息写回该文件，会直接报 `open /usr/local/etc/cpolar/cpolar.yml: permission denied`，登录失败（2026-10-25 实际踩过）。正确设置：

  ```bash
  chown nobody:nogroup /usr/local/etc/cpolar/cpolar.yml
  chmod 640 /usr/local/etc/cpolar/cpolar.yml
  chown nobody:nogroup /usr/local/etc/cpolar
  chmod 750 /usr/local/etc/cpolar     # 目录也要可写，登录可能走临时文件+改名
  ```

  `other` 位必须为 `0`（不能退回 `666`），目录 `750` 已能挡住其他用户读取。历史备份同样要 `chmod 600`。
- 修改配置前先备份，例如 `cp -a cpolar.yml cpolar.yml.bak.$(date +%Y%m%d-%H%M%S)`；配置写错会导致全部隧道（含 ssh）一起起不来。

### 验证

```bash
curl -sS -o /dev/null -w 'root=%{http_code}\n' http://127.0.0.1/
curl -sS -o /dev/null -w 'spa=%{http_code}\n' http://127.0.0.1/admin/dashboard
curl -sS -o /dev/null -w 'api=%{http_code}\n' http://127.0.0.1/api/v1
nginx -t
systemctl is-active nginx
```

根路径与任意子路由均应返回 `200`；`/api/v1` 返回的是后端自身的 JSON 响应（业务错误也说明代理链路正常）。

### 排查

| 现象 | 优先检查 |
| --- | --- |
| 页面白屏 | `index.html` 中引用的 `/assets/*.js` 是否 404，通常是上传不完整 |
| 刷新子路由 404 | `location /` 的 `try_files` 回退是否被改动 |
| 接口 404 / 502 | 后端容器状态：`cd /opt/apps/smart-university && docker compose ps` |
| 静态资源未更新 | 浏览器缓存了 `index.html`，或上传时未覆盖旧文件 |
| cpolar 仪表盘（`:9200`）报 `permission denied` 登录失败 | `cpolar.yml` 属主不是 `nobody:nogroup`，见「公网访问」注意事项里的权限说明；`tail -300 /var/log/cpolar/access.log \| grep -i 'permission denied'` 可见同一条报错 |
| 其他异常 | `tail -n 100 /var/log/nginx/course-selection-web.error.log` |

端口 `80` 目前监听在 `0.0.0.0` 和 `::`，服务器 UFW 未启用，仅应在可信局域网内访问；如需公网访问，应先配置 HTTPS、访问控制和来源限制。

## 十三、MinIO

### 部署信息

截至 `2026-10-25`，对象存储使用 Docker Compose 部署，镜像为 Chainguard 构建的 MinIO（原因见下节）。

| 项目 | 当前配置 |
| --- | --- |
| Compose 目录 | `/opt/middleware/minio` |
| Compose 文件 | `/opt/middleware/minio/docker-compose.yml` |
| 容器名 | `minio` |
| 镜像 | `cgr.dev/chainguard/minio@sha256:87896cb33f93ecfcca4e883c3fb2f86a0a1a600c003a104011ed30631c1869c3`（内部版本 `RELEASE.2026-10-23T20-06-51Z`） |
| S3 API 地址 | `http://192.168.1.102:18090` |
| Web 控制台 | `http://192.168.1.102:18091` |
| 对外监听 | 仅 `192.168.1.102`，不对全网卡发布 |
| 数据目录 | `/data/minio/data`，属主 `65532:65532` |
| 凭据文件 | `/opt/middleware/minio/minio.env`，权限 `600 root:root` |
| 账号 | `admin` / `12345678` |
| 重启策略 | `restart: unless-stopped`（随 Docker 开机自启） |
| 健康检查 | `mc ready local`，每 `30s`，启动宽限 `30s` |
| 日志 | Docker `json-file` 轮转，单文件 `20 MB`，保留 `5` 个 |

首次启动实测：容器 `9s` 进入 `healthy`；写入对象、重启容器后桶与对象均保留；健康检查依赖镜像内自带的 `/usr/bin/mc`。

### 为什么没有用官方镜像（重要背景）

| 时间 | 变化 |
| --- | --- |
| 2026-04-25 | `minio/minio` 源码仓库归档只读 |
| 2026-09-11 | Docker Hub 删除 `minio/minio`，拉取报 `pull access denied` |
| 2026-09-24 | `quay.io/minio/minio` 开始拒绝匿名拉取（`401 UNAUTHORIZED`） |
| 现在 | 官方后继 AIStor 镜像可拉，但**无 license 时拒绝全部 S3 操作** |

两种方案都实测过：

- `quay.io/minio/aistor/minio`：容器能启动、控制台可打开，但 `mc mb` / `mc ls` 均返回 `Access denied. No license is installed`。免费 license 需在 `https://www.min.io/pricing` 注册领取，再执行 `mc license update` 或配置 `MINIO_LICENSE`。若后续要走这条路，注意免费层限制为单节点、每节点最多 2 个驱动器。
- `cgr.dev/chainguard/minio`（当前采用）：Chainguard 从源码持续重建并回移 CVE 修复，匿名可拉，镜像自带 `mc`，控制台完整可用。

Chainguard 镜像的两个约束：

1. **只发布 `latest` tag**，tag 会随重建漂移，因此 Compose 中按 `digest` 钉死。升级时先 `docker pull cgr.dev/chainguard/minio:latest`，取新 digest 替换后 `docker compose up -d`。
2. **以 uid `65532` 非 root 运行**。数据目录属主必须是 `65532:65532`，否则启动报 `Unable to initialize backend: file access denied`。新机器或换数据目录后先执行 `chown -R 65532:65532 /data/minio/data`。

### 管理命令

```bash
cd /opt/middleware/minio
docker compose up -d
docker compose stop
docker compose restart
docker compose ps
docker compose down
docker logs --tail 100 minio
docker logs -f minio
```

状态与健康：

```bash
docker inspect -f 'status={{.State.Status}} health={{if .State.Health}}{{.State.Health.Status}}{{end}} restarts={{.RestartCount}}' minio
curl -sS -o /dev/null -w 's3=%{http_code} console=%{http_code}\n' \
  http://192.168.1.102:18090/minio/health/live http://192.168.1.102:18091/
```

服务端操作凭据（客户端 mc 直接指向宿主机端口即可）：

```bash
docker exec -e MC_HOST_local=http://admin:12345678@127.0.0.1:9000 minio mc admin info local
docker exec -e MC_HOST_local=http://admin:12345678@127.0.0.1:9000 minio mc ls local
```

注意 `mc cp` 的源路径是在**容器内**解析的，宿主机文件要用管道方式上传：

```bash
printf 'hello\n' | docker exec -i -e MC_HOST_local=http://admin:12345678@127.0.0.1:9000 \
  minio mc pipe local/some-bucket/hello.txt
```

`docker compose down` 不会删除 `/data/minio/data`；不要使用 `down -v`，也不要直接删该目录。

### 客户端连接

- Endpoint：`http://192.168.1.102:18090`
- **必须开启 path-style**（`http://host:port/bucket/key`），因为服务以 IP + 端口方式提供，不是虚拟主机风格的域名
- Region 可填任意值，通常 `us-east-1`
- Access Key / Secret Key：`admin` / `12345678`

mc 客户端：

```bash
mc alias set local http://192.168.1.102:18090 admin 12345678
mc mb local/demo
mc ls local
```

Java / Spring（AWS SDK v2 要点）：

```java
S3Client s3 = S3Client.builder()
    .endpointOverride(URI.create("http://192.168.1.102:18090"))
    .region(Region.US_EAST_1)
    .credentialsProvider(StaticCredentialsProvider.create(
        AwsBasicCredentials.create("admin", "12345678")))
    .forcePathStyle(true)   // 关键，否则报 NoSuchBucket / 域名解析失败
    .build();
```

### 备份

数据全部在 `/data/minio/data`，但直接复制目录不是一致备份。规范做法是用 `mc mirror` 复制到另一位置（例如 `/data/backup/minio`）：

```bash
docker exec -e MC_HOST_local=http://admin:12345678@127.0.0.1:9000 \
  -e MC_HOST_backup=http://<备份目标> minio mc mirror --overwrite local backup
```

目前**尚未配置自动备份**，需要时再单独加定时任务，并做一次恢复演练。

### 排查

| 现象 | 优先检查 |
| --- | --- |
| 启动即退出，日志 `file access denied` | `chown -R 65532:65532 /data/minio/data` |
| `mc mb` 返回 `Access denied. No license is installed` | 用的是 AIStor 镜像，需换 Chainguard 镜像或安装 license |
| 客户端报 `NoSuchBucket` / 域名解析失败 | 未开启 path-style |
| 客户端 `403 SignatureDoesNotMatch` | 密钥错误，或系统时间偏差过大 |
| 局域网访问不通 | 端口只绑了 `192.168.1.102`，`ss -lntp \| grep 1809` 核对 |
| 容器反复重启 | `docker logs --tail 200 minio`，多为数据目录权限或磁盘满 |

### 安全

- 密码 `12345678` 是 8 位纯数字弱口令，与当前服务器其他中间件同级别，仅适合受控局域网。修改方式：编辑 `/opt/middleware/minio/minio.env` 后 `docker compose up -d` 重建容器，并同步更新调用方配置。
- 服务本身无 TLS，内网明文传输。如需公网访问，必须在前面加反向代理做 HTTPS，并配合访问控制，不要直接把 `18090/18091` 暴露出去。
- 控制台 `18091` 与 API `18090` 目前都只监听局域网地址，符合现有中间件的最小暴露原则。

## 十四、临时验证环境（在服务器上跑后端测试）

用途：开发机（Windows）无法验证依赖「安全目录句柄」的功能时，把 Spring 后端临时跑在服务器上，前端仍在开发机，通过局域网访问。

截至 `2026-10-26` 实测的要点：

1. **目录**：临时工作目录放 `/tmp/huimeng-<批次>`。`/tmp` 是 tmpfs，重启即清空，不会污染 `/opt` 与 `/data`；但要注意它占用内存（见第 5 条）。
2. **源码传输**：服务器**到 GitHub 不通**（`git ls-remote https://github.com/...` 无输出直至超时），不要指望在服务器上 clone。做法是从开发机打包后上传。`2026-10-27` 复测：**paramiko SFTP `put` 可用且快**（108 MB 的 JAR 约 9 秒，约 12 MiB/s），比 base64 分片可靠得多；此前记录的「root 会话 SFTP 被拒」实为开发机 Git Bash 的 MSYS 路径转换——`/opt/...` 被改写成 `C:/.../opt/...`，SFTP 自然报 `FileNotFound`。用 Python paramiko 传字符串路径（不经 shell），或在命令前加 `MSYS_NO_PATHCONV=1`。`cat >` 走 stdin 写大文件仍可能被断连，不要用于几十 MB 以上的内容。
3. **Maven**：`maven:3.9.11-eclipse-temurin-21` 镜像已在本机，镜像内 `mvn` 可用。项目自带的 `./mvnw` 会因 `distributionSha256Sum` 校验失败退出（下载的 3.9.7 与期望摘要不符），**直接用 `mvn`**。中央仓 `repo.maven.apache.org` 可达（HTTP 200）；`maven.aliyun.com` 返回 404，不要配。
4. **依赖缓存**：`/tmp/huimeng-m30x/backend/.m2` 已有约 190 MB 热缓存，新批次直接 `cp -a` 复用，可省掉 GraalJS 等大依赖的重复下载。
5. **内存与磁盘**：整机 14 GB，常驻已占约 11 GB，可用约 4 GB；`/tmp` 是 tmpfs，占用同时计入内存。跑 Maven + Spring 测试必须限堆：`MAVEN_OPTS=-Xmx1024m` 配合 `-DargLine=-Xmx1536m`，并在启动前 `free -m` 确认余量，否则容易把机器压到 OOM。
6. **网络绑定**：容器用 `--network host` 时可直接绑定宿主机局域网地址（例如 `192.168.1.102:18097`），开发机的前端（Vite）用 `HUIMENG_API_TARGET` 指过来即可；服务地址不要绑 `0.0.0.0`。
7. **端口**：这类临时服务只用高位端口（当前项目用 `18097`），避开中间件已占用的 `80`、`3306`、`6379`、`8081`、`8848`、`18080`–`18091`。
8. **收尾**：测试结束执行 `docker rm -f <容器名>` 释放端口，并删除 `/tmp/huimeng-<批次>`；不要留下长期运行的容器。

> 上述命令均以 root 或具备 docker 权限的账号执行；**任何凭据都不写入本文档**，密码从受控的密码管理工具获取。

## 十五、绘梦（huimeng）部署现状

> **当前状态（`2026-10-31`）：服务已停用，三个容器 `huimeng-api` / `huimeng-web` / `huimeng-boundary` 已删除**，项目网络与全部 `huimeng-api` 镜像已清理。代码、配置、密钥、工作区数据、数据库均保留，可按 15.5 重建。下方内容为停用前的形态记录，供恢复时对照。

截至 `2026-10-27` 的实际布局与升级步骤；只记录形态与顺序，凭据一律在受控文件里，不写入本文档。

### 15.1 目录与服务

| 项 | 位置 / 值 |
| --- | --- |
| 应用目录 | `/opt/apps/huimeng`（`secrets/` 700；**`target/`、`validation/` 已于 2026-10-31 删除**） |
| 工作区 | `/data/huimeng/workspaces`（保留） |
| 静态站点 | `/var/www/huimeng`（目录 755、文件 644） |
| 备份 | `/data/backup/huimeng`（库 dump、web 包、`.last-*` 标记） |
| 容器 | **已删除**（原为 `huimeng-api`、`huimeng-web`、`huimeng-boundary`） |
| 端口 | 原 `huimeng-web` 映射 `192.168.1.102:18443→443`，现已释放 |
| 数据库 | 生产库 `huimeng`（当前 V10）；开发库 `huimeng_dev` 供开发机直连（容器删除不影响库） |
| 运行账号 | `huimeng_app`@`172.29.46.%`，按表 DML；迁移账号单独，仅迁移期使用 |
| 镜像回滚点 | **已删除**（原 `huimeng-api:m2` 与 7 个 tag 一并清理，重建需重新 `docker build`） |

### 15.2 升级顺序（已在 2026-10-27 实跑）

1. 备份：`mysqldump --single-transaction --routines --triggers --databases huimeng | gzip` 到 `/data/backup/huimeng`，并 `gzip -t` 校验；web 目录先 `tar -czf` 备份再替换。
2. 上传：开发机用 paramiko `put` 把 JAR、`target/native/*`、`Dockerfile`、`compose.yml`、`nginx.conf`、`finalize_database.py`、前端 `dist.tar.gz` 传到 `/opt/apps/huimeng`（注意 `MSYS_NO_PATHCONV=1`）。
3. 建镜像：`docker build -f Dockerfile -t huimeng-api:m3 .`（`compose.yml` 用 `${HM_IMAGE:-huimeng-api:m3}`，回滚时用 `HM_IMAGE=huimeng-api:m2`）。
4. 迁移：`docker compose -f compose.yml --profile migration run --rm migration`，观察 Flyway 到目标版本。
5. 授权：`python3 finalize_database.py`——表清单从 `information_schema` 现算，迁移历史只读、审计不可 UPDATE。
6. 静态站点：解压新 `dist` 到 `/var/www/huimeng`，重设 755/644。
7. 启动：`docker compose -f compose.yml up -d --force-recreate api`；容器 `read_only`，内存上限 1536 MB（GraalJS 需要额外常驻内存）。

### 15.3 踩过的坑（会重复出现）

1. **新表没授权会被报成 SQL 语法错**：MySQL 对「无权限」返回 `BadSqlGrammarException` 的外观，日志只显示 INSERT 语句，容易误判为迁移写错。升级后必须跑 `finalize_database.py`。
2. **`plugins.content` 必须是 MEDIUMBLOB**：内置节点产物 263 KiB–1.7 MiB，BLOB 只有 64 KiB；MySQL 上会静默跳过全部内置节点。V10 已修，回滚到 V9 之前会复现。
3. **环境变量会覆盖端口**：开发机的 `SERVER__PORT` 会被 Spring 松弛绑定成 `server.port`；容器里若同名前缀变量存在也要显式覆盖，启动脚本里直接写 `--server.port`。
4. **证书校验**：`curl` 在 Windows 走 Schannel 会报 `revocation status is unknown`，这不是服务端问题；用 `curl --cacert secrets/tls/ca.crt`（Linux）或 Python 严格 SSL 上下文验证。
5. **healthcheck**：`huimeng-api` 没有宿主机端口，排障用 `docker logs`；只有 `huimeng-web` 能从局域网访问。

### 15.4 模型执行器与对象存储（2026-10-27 接入）

| 项 | 实际形态 |
| --- | --- |
| ComfyUI | 工作站 `192.168.1.100:8188`（Windows，RTX 3060 12G，portable 版，0.37.0） |
| MinIO | 服务器容器 `minio`：S3 `192.168.1.102:18090`，控制台 `18091`，数据在 `/data/minio/data` |
| 生产桶 | `huimeng-media`；账号 `huimeng-media` 仅在该桶有 ListBucket / Get / Put / Delete |
| 配置位置 | `/opt/apps/huimeng/secrets/api.env` 的 `HUIMENG_COMFY_*` 与 `HUIMENG_STORAGE_*` |

1. **ComfyUI 必须显式对外监听**：portable 版 `run_nvidia_gpu.bat` 默认不带 `--listen`，只绑 `127.0.0.1`。此时从服务器或从工作站自己的局域网地址访问都是「积极拒绝」，**不是防火墙问题**。在 `main.py` 参数末尾加 `--listen 0.0.0.0` 并重启；成功标志是 `netstat` 出现 `0.0.0.0:8188 LISTENING`，且日志打印 `To see the GUI go to: http://0.0.0.0:8188`。
2. **该工作站 Windows 防火墙三个配置文件都是关闭状态**，加 `--listen` 后即可直连，不需要加入站规则；换机器时要重新确认这一点。
3. **Docker internal 网络没有默认路由**：`huimeng_entry`、`huimeng_executor`、`huimeng_database` 全是 `internal: true`，只挂这些网络的容器**出不去**（连局域网 IP 都不行）。API 需要访问外部模型供应商、插件市场与 ComfyUI，必须额外挂一条非 internal 的 `egress` 网络；这条网络只挂 API，且容器不发布端口，入口仍由 nginx 收口。
4. **对象存储缺配置时不会启动失败**：`huimeng.storage.enabled` 缺省为 false，应用照常启动，但媒体生成在准入之后被拒——`503 EXECUTOR_UNAVAILABLE`「媒体存储暂不可用」。看到这个错误先查 `HUIMENG_STORAGE_*` 是否齐全，而不是查 ComfyUI。
5. **生成的前置条件**：项目必须是 `ACTIVE`。写操作对归档项目返回 `403 CAPABILITY_FORBIDDEN`（`requireMember(write)` 里的状态判定），和成员权限无关；新增成员不会改变项目状态。
6. **验收最小路径**：登录 → `PUT /api/v1/admin/models/approval` 启用模型 → 建/选 ACTIVE 项目并加成员 → `POST /api/v1/projects/{id}/creative/ai/media/generate`（需 `Idempotency-Key`）→ 轮询 `runs/{runId}` → `media/{mediaId}/content` 下载并比对 sha256。

### 15.5 发布与回滚的硬性顺序（2026-10-27 实测）

1. **先传产物，再清站点**：`/var/www/huimeng` 的发布步骤是「备份 → 清空 → 解压」。若压缩包还没上传成功就先清空，站点会直接空白（当天发生过一次，靠 `web-pre-*.tar.gz` 恢复）。顺序必须是：`上传成功并核对大小` → `备份` → `清空` → `解压`，解压失败立即回滚备份。
2. **上传目录 `upload/` 需要先 `mkdir -p`**：它会被日常清理删掉，SFTP `put` 不会自动建目录。
3. **迁移由 migration 容器执行，API 只校验**：发布新 JAR 后若忘了 `docker compose --profile migration run --rm migration`，API 启动即失败：`FlywayValidateException: Detected resolved migration not applied to database: 11`。正确顺序是「建镜像 → 跑 migration → 重启 API」。
4. **跨 Flyway 版本不能只换 JAR 回滚**：回滚目标若不含当前库已有的迁移版本，API 会在校验阶段退出。要么回滚到**同版本**的上一镜像（已实测：`892cbd2b4872` ↔ `prev-m3 c9d00b4aa1ae`），要么先恢复配套数据库（每次发布前 `mysqldump` 已在 `/data/backup/huimeng`）。
5. **`project_media.created_by` 不再有外键**（V11）：该列同时存内部用户 id 与外部客户端 id，原来的 `users` 外键会让 MCP/A2A 的生成结果在入库时必然失败。
6. **失败要能从日志看出来**：媒体生成失败会打 `WARN 图片生成失败/视频生成失败`，带 run、remoteAttempted、confirmed 与异常类型；排障先看这条。

### 15.6 停用与清理（2026-10-31）

按需求停用并删除全部容器，**只删容器与构建产物，不动数据**。

删除命令（在 `/opt/apps/huimeng` 下执行，不带 `-v`）：

```bash
docker compose down          # 删 3 个容器 + 项目网络，保留 bind 与具名卷
```

随后清理的遗留项：

| 对象 | 处理 | 说明 |
| --- | --- | --- |
| 网络 `huimeng_executor` / `entry` / `ingress` / `egress` | 已删除 | `down` 自动处理 |
| 网络 `huimeng_database` | 已删除 | 中间件 `mysql` 曾挂在此网络供 API 访问，需先 `docker network disconnect -f huimeng_database mysql` 再 `docker network rm`；**该网络没有 compose 标签**，留着会让以后 `up` 报 `network has incorrect label` |
| 镜像 `huimeng-api` 全部 7 个 tag（`m2/m3/m4h/m4q` + `prev-m3/prev-m4h/prev-m4q`） | 已删除 | 约 2.2GB；删除后无回滚镜像，重建需 `docker build` |
| `/opt/apps/huimeng/target`、`/opt/apps/huimeng/validation` | 已删除 | 约 820MB（4 个 104MB 的 jar + m3 测试产物） |
| 具名卷 `huimeng-m3-maven` | 已删除 | 121MB Maven 缓存，下次构建需重下依赖 |

**明确保留**：`/opt/apps/huimeng/{compose.yml,Dockerfile,nginx.conf,secrets/,executor-boundary/,*.py,frontend-*.tar.gz}`、`/data/huimeng`（含 9 项工作区）、`/var/www/huimeng`、`/data/backup/huimeng`、MySQL 中的 `huimeng` / `huimeng_dev` 库。

恢复步骤：

```bash
# 1. 重建镜像（JAR 已删，需重新构建或从开发机重新上传 target/）
cd /opt/apps/huimeng && docker build -f Dockerfile -t huimeng-api:m3 .
# 2. 起服务（数据库与 secrets 都在，无需迁移时可跳过 --profile migration）
docker compose up -d
# 3. 验证
docker compose ps && curl -k https://192.168.1.102:18443/
```

注意两点：一是 `mysql` 已被移出 `huimeng_database` 网络，`up` 时 `huimeng_database` 会由 compose 重新创建，**但 mysql 不会自动重新接入**；若 API 需要按容器名连库，要手动 `docker network connect huimeng_database mysql`。二是镜像与 `target/` 都已清空，恢复前必须先有 JAR 产物。

## 十六、PostgreSQL

### 部署信息

截至 `2026-10-30`，新增 PostgreSQL 实例，使用 Docker Compose 部署，与 MySQL 并存（两者互不替代，业务库 `toonflow` 独立）。当日先按 `18-alpine` 部署，随后按需求切换为 **PostgreSQL 16**（切换过程见下节）。

| 项目 | 当前配置 |
| --- | --- |
| 镜像 | `postgres:16.15-alpine-pgvector-0.8.7`（PostgreSQL 16.15 + pgvector 0.8.7） |
| 基础镜像 digest（升级前记录） | `sha256:0cefaabd882e42bcb2a2f3dc56d3ce4770560ba62fd7f197f1a2e008fd5731ab` |
| 容器名 | `postgres`（`restart: unless-stopped`） |
| 端口 | `0.0.0.0:5432`（IPv4 + IPv6 全网卡监听） |
| 配置目录 | `/opt/middleware/postgres`（`docker-compose.yml`、`postgres.env`） |
| 数据目录 | `/data/postgres/data` → 容器 `/var/lib/postgresql/data`（`bind`，无匿名卷） |
| 实际 PGDATA | `/var/lib/postgresql/data`，即宿主机 `/data/postgres/data` 根目录 |
| 超级用户 | `root`（`POSTGRES_USER`，**没有默认的 `postgres` 角色**） |
| 业务库 | `toonflow`（owner `root`）、`ragent` 与 `ragent_bit`（owner `ragent`）、`ragent_lightrag`（独立同名角色） |
| 密码加密 | `scram-sha-256` |
| 凭据文件 | `/opt/middleware/postgres/postgres.env`（`600 root:root`） |
| 日志 | Docker json-file，单文件 20MB × 5 |
| 旧版本归档 | 原 `/data/postgres/data.pg18-20261030-113626`（64M）已于 2026-10-30 清理；迁移前 schema 仅保留在 `/data/backup/postgres/` 的两份导出中 |

容器名、目录布局遵循「中间件 → `/opt/middleware`、数据 → `/data`」的既定规范。

### 版本演进与镜像差异（16 与 18 不一样，换版本必看）

| 差异点 | PG16（当前） | PG18（已弃用） |
| --- | --- | --- |
| 默认 `PGDATA` | `/var/lib/postgresql/data` | `/var/lib/postgresql/18/docker`（按大版本建子目录） |
| 声明的 VOLUME | `/var/lib/postgresql/data` | `/var/lib/postgresql`（整个目录） |
| 对应挂载点 | `/data/postgres/data:/var/lib/postgresql/data` | `/data/postgres/data:/var/lib/postgresql` |
| 运行身份 | uid/gid `70`（alpine） | uid/gid `70`（alpine） |

两条通用规则：

1. **挂载点必须与镜像声明的 VOLUME 路径一致**，否则会在容器里生成一个匿名卷（`docker volume ls` 里一串 64 位 hash），数据落在匿名卷而非 `/data`，备份和迁移都会踩空。核对命令：`docker image inspect <image> --format '{{json .Config.Volumes}}'`。
2. **alpine 变体运行身份是 uid `70`，debian 变体是 `999`**。换 tag 前先 `docker run --rm --entrypoint sh <image> -c 'id postgres'`，数据目录属主不对会直接 `data directory has wrong ownership`。

### 从 PG18 切换到 PG16 的过程（2026-10-30 实测）

**不能直接复用数据目录**——PG16 读到 PG18 的 `PG_VERSION` 会拒绝启动。正确路径是逻辑导出 + 新建实例 + 灌回：

1. `pg_dump -Fc` 留一份归档备份（`/data/backup/postgres/toonflow-pg18-*.dump`）。
2. 再从 PG18 导出**纯 SQL**（`-Fp --no-owner --no-privileges`）。这一步是关键：PG18 的 custom 归档版本比 PG16 的 `pg_restore` 新，直接 restore 会报 archive version 不支持；纯 SQL 没有这个限制。
3. **删掉 `SET transaction_timeout = 0;`**。该 GUC 是 PG17 才引入的，PG17+ 的 `pg_dump` 会自动写进文件头，灌进 PG16 会报 `unrecognized configuration parameter`。用 `sed -i '/^SET transaction_timeout/d'` 处理后其余 `SET` 均可正常执行。
4. `docker compose down` → 把旧数据目录整体 `mv` 成 `data.pg18-<时间戳>`（**留档，不删**）→ 新建空目录并 `chown 70:70`、`chmod 700`。
5. 改 Compose 的 `image` 与 `PGDATA`/挂载点 → `docker compose up -d`，等 healthy（实测 8 秒）。
6. 用新容器的 `psql` 灌入前面处理好的 SQL（`-v ON_ERROR_STOP=1`），逐项核对表数、`flyway_schema_history`、索引与约束。

切换前务必确认库内有无真实数据：本次 7 张表全部 0 行、`flyway_schema_history` 只有一条 baseline，才按「导出→重建→灌回」处理；**若有真实数据，必须先备份并确认业务已停写**。

### 认证与安全

`pg_hba.conf` 实际生效规则（按匹配顺序）：

| # | 类型 | 地址 | 库/用户 | 认证方式 |
| --- | --- | --- | --- | --- |
| 1 | local | 套接字 | all/all | `trust` |
| 2-3 | host | `127.0.0.1/32`、`::1/128` | all/all | `trust` |
| 4-6 | local/host | 回环 | replication/all | `trust` |
| 7 | host | **`all`（0.0.0.0/0 + ::/0）** | all/all | **`scram-sha-256`** |

前 6 条是 initdb 的默认回环规则，`trust` **只对容器内本地套接字和回环地址生效**；所有远程连接（包括局域网、其他容器）命中第 7 条，必须提供密码。实测：错密码返回 `password authentication failed`，不存在的用户同样被拒（不会泄露用户是否存在）。

注意两点：

- 因为选了全网卡监听且服务器 UFW 未启用，`5432` 对整个局域网开放，**密码强度是唯一防线**。要收窄就把端口映射改成 `192.168.1.102:5432:5432` 后 `docker compose up -d`，数据不受影响。
- `local ... trust` 意味着容器内的本地连接免密，非 root 无法 `docker exec` 就进不去，但**在 `docker` 组里的用户等价于数据库超级用户**，加人入组要谨慎。

> **密码 `123456` 是弱口令**（2026-10-30 按需改成此值，原为 `c13383413150@163.com`）。`123456` 在任何暴力破解字典的第一页，而 PG 默认没有失败次数限制、也没有 fail2ban。当前仅适合受控局域网；对外暴露前必须先换掉或收窄监听地址。

改密方式（两步都要做，否则 `env` 与库内实际密码不一致，将来重建容器会让人误判）：

```bash
# 1. 改库内密码
docker exec -it postgres psql -U root -d toonflow -c "alter role root with password '新密码';"
# 2. 同步 env 文件（仅影响下次 initdb，但必须保持一致）
#    编辑 /opt/middleware/postgres/postgres.env 的 POSTGRES_PASSWORD
```

`ALTER ROLE` 立即生效，**不需要重启容器**；`pg_authid.rolpassword` 会以 `SCRAM-SHA-256$` 前缀存储，可在库内核对是否生效。

### 关键参数（默认值，未调优）

| 参数 | 值 | 说明 |
| --- | --- | --- |
| `shared_buffers` | 128MB | 默认偏保守；本机 14G 内存可考虑 1-2GB |
| `max_connections` | 100 | 连接池上限要低于此值 |
| `work_mem` | 4MB | 复杂排序/哈希连接多时上调 |
| `max_wal_size` | 1GB | |
| `wal_level` | replica | 支持流复制与 PITR 归档 |
| `data_checksums` | **on** | 建库时通过 `POSTGRES_INITDB_ARGS=--data-checksums` 开启，可尽早发现静默损坏；**只在 initdb 时生效，事后无法打开** |
| `TimeZone` | Asia/Shanghai | `timestamptz` 显示按此 |

内存只有 4.7G 可用时不要盲目调大 `shared_buffers`，需先看其他容器占用。

### 常用命令

```bash
cd /opt/middleware/postgres
docker compose ps
docker compose logs --tail 100 postgres
docker compose restart
docker exec -it postgres psql -U root -d toonflow          # 容器内免密（local trust）
docker exec -it postgres psql -U root -d toonflow -c '\l'  # 库列表
```

从外部带密码连接（不要把密码写进命令历史，用 `PGPASSWORD` 或 `~/.pgpass`）：

```bash
PGPASSWORD='123456' psql -h 192.168.1.102 -p 5432 -U root -d toonflow
```

JDBC（注意 `currentSchema` 与 SSL 参数）：

```
jdbc:postgresql://192.168.1.102:5432/toonflow?stringtype=unspecified
```

Python（SQLAlchemy）：

```
postgresql+psycopg://root:123456@192.168.1.102:5432/toonflow
```

> 密码若含 `@`、`#`、`/` 等字符，在 URL 里必须转义（`@` → `%40`），否则会被当成主机分隔符或注释起点。当前密码为纯数字，无需转义；以后换密码时注意这一点。

### 验收记录（2026-10-30 实测）

- 容器 **9 秒**进入 `healthy`，`RestartCount=0`；日志确认 `listening on IPv4 address "0.0.0.0", port 5432` 与 IPv6。
- 建表（`bigserial` 主键 + `jsonb` + `timestamptz`）→ 插入中文与 JSON → 查询回读一致；主键冲突正确报 `duplicate key`；`BEGIN/ROLLBACK` 生效。
- 跨网络连接：从外部容器经 `192.168.1.102:5432` 登录成功，返回 `TCP_OK 172.31.0.2/32:5432 as root db=toonflow`；本机 `Test-NetConnection` 到 5432 为 `True`。
- 错密码被拒（`password authentication failed`）；不存在的库报 `database "nosuchdb" does not exist`。
- `docker compose restart` 后 **8 秒**恢复 healthy，验收表数据仍在（持久化成立）。
- 验收表已删除，`public` schema 表数为 0。
- **改密验证（当日 10:19）**：`ALTER ROLE root WITH PASSWORD '123456'` 成功，`pg_authid.rolpassword` 为 `SCRAM-SHA-256$` 前缀；新密码经 `192.168.1.102:5432` 连接成功（`NEW_PASSWORD_OK as root db=toonflow`），旧密码被拒；容器 `health=healthy`、`RestartCount=0`，未重启。

**切换到 PG16 后的复验（同日 11:36）**：

- `postgres --version` = **PostgreSQL 16.15**，`data_directory=/var/lib/postgresql/data`，`data_checksums=on`、`TimeZone=Asia/Shanghai`、`password_encryption=scram-sha-256`。
- 容器 **8 秒** healthy；`docker inspect` 显示挂载为 `bind /data/postgres/data -> /var/lib/postgresql/data`，**无匿名卷**。
- schema 恢复完整：7 张业务表、9 个索引（7 个主键 + `workspace_project_directory_key` 唯一索引 + `flyway_schema_history_s_idx`）、`flyway_schema_history` 1 条 baseline 记录，与切换前一致。
- DML 实测：`workspace_project` 用序列默认值插入返回 `id=1` → 更新中文名称（`PG16 验收`）→ 删除，行数归零；`sys_setting` 缺 `setting_key` 时正确报 `not-null constraint` 违反。
- `jsonb` + 中文 + `timestamptz` 往返正常（`now()` 显示 `+08`）。
- 密码 `123456` 经 TCP 仍可登录（`TCP_OK as root db=toonflow`）。
- `docker compose restart` 后 8 秒恢复 healthy，7 张表与 flyway 记录均存活。

### 备份

已有的两份切换前备份（均在 `/data/backup/postgres/`）：

| 文件 | 说明 |
| --- | --- |
| `toonflow-pg18-20261030-113545.dump` | PG18 custom 归档（`pg_restore -l` 校验通过，TOC 31 条） |
| `toonflow-pg18-20261030-113626.sql` | 纯 SQL 版本（已去除 `SET transaction_timeout`，可直接灌入 PG16） |

**日常自动备份仍未配置**。规范做法是 `pg_dump -Fc` 到 `/data/backup/postgres` 并保留 N 天，或用 `pg_basebackup` 做物理备份 + WAL 归档实现 PITR。注意：**直接 `cp` 数据目录不是一致备份**，运行中拷贝会得到损坏的备份。需要的话可加一个 cron/systemd timer 任务。

原 PG18 的整份数据目录 `/data/postgres/data.pg18-20261030-113626`（64M）已在确认上述两份导出的 TOC 可正常读取后删除，当前 `/data/postgres/` 下只剩活跃的 `data`。

### 排查

| 现象 | 优先检查 |
| --- | --- |
| 容器起不来，日志 `data directory has wrong ownership` | `/data/postgres/data` 属主是否为 `70:70`（alpine） |
| 容器起不来，日志 `database files are incompatible with server` | 数据目录是别的 PG 大版本留下的，不能跨大版本复用，需按上文「版本切换」流程重建 |
| `FATAL: password authentication failed` | 密码是否含未转义字符；`pg_hba.conf` 第 7 条是否为 `scram-sha-256` |
| `FATAL: role "postgres" does not exist` | 本实例超级用户是 `root`，没有 `postgres` 角色 |
| 客户端报 `no pg_hba.conf entry` | 连接来源不是回环，且第 7 条被改动过 |
| 应用连不上但 `psql` 能连 | 检查连接池上限、`max_connections=100`、以及是否连了 `toonflow` 之外的库 |
| 迁移类工具报 `current transaction is aborted` | 事务内语句失败后未回滚，检查 `--single-transaction` 用法 |
| 从高版本 dump 恢复到低版本报 `unrecognized configuration parameter "transaction_timeout"` | 删掉 SQL 里的 `SET transaction_timeout`（PG17+ 独有），或改用 `-Fp` 纯 SQL 导出 |
| `pg_restore` 报 `unsupported version (1.16) in file header` | 归档由更高版本 `pg_dump` 生成，低版本 `pg_restore` 读不了；改用纯 SQL 导出 |
| `docker volume ls` 出现无主匿名卷 | 挂载点与镜像声明的 VOLUME 路径不一致，见上文版本差异表 |

## 十七、Ollama（本地大模型推理）

### 部署信息

`2026-11-17` 部署，采用**官方二进制 + systemd 原生安装**（未走 Docker，原因见下节）。

| 项目 | 当前配置 |
| --- | --- |
| 版本 | `0.40.16`（`/usr/local/bin/ollama --version`） |
| 二进制 | `/usr/local/bin/ollama`（44MB，软链目标 `/usr/local/lib/ollama/bin/ollama`） |
| 运行库 | `/usr/local/lib/ollama`（2.4G，含 CUDA/ROCm 运行库与 `llama-server`） |
| 数据目录 | `/data/ollama/models`（`ollama:ollama` 750，已用 941M） |
| 运行用户 | `ollama`（系统用户，`/usr/share/ollama`，已加入 `render`、`video` 组） |
| 服务单元 | `/etc/systemd/system/ollama.service`（`enabled` + `active`，`Restart=always`） |
| 监听 | `0.0.0.0:11434`（全部网卡；`2026-11-17` 由 `192.168.1.102:11434` 改绑，用户确认单人使用） |
| 已装模型 | 见下方「已装模型清单」 |
| 加速 | NVIDIA GTX 1650 Ti（TU117M，3.6GiB 可用），`100% GPU` 全层 offload |
| 日志 | `journalctl -u ollama` |

关键环境变量（在 service 单元内，改完需 `systemctl daemon-reload && systemctl restart ollama`）：

| 变量 | 值 | 作用 |
| --- | --- | --- |
| `OLLAMA_HOST` | `0.0.0.0:11434` | 绑定全部网卡（服务端）。**客户端**环境变量另设为 `127.0.0.1:11434` |
| `OLLAMA_MODELS` | `/data/ollama/models` | 模型目录落在 `/data` |
| `OLLAMA_KEEP_ALIVE` | `30m` | 模型常驻 30 分钟，避免频繁重载 |
| `OLLAMA_MAX_LOADED_MODELS` | `1` | 内存受限机型只留一个模型 |
| `OLLAMA_NUM_PARALLEL` | `1` | 单请求并发，避免爆内存 |

### 为什么不用官方 install.sh

官方 `install.sh` 里有一段逻辑：只要 `nvidia-smi` 跑不通（本机正是这种情况），就会判定「NVIDIA 驱动缺失」，然后 `apt install cuda-drivers` 覆盖现有驱动。本机跑着 `nvidia-driver-595-open` 与 GNOME 会话，被覆盖风险太高。

因此改为手工复刻官方步骤：下载 `ollama-linux-amd64.tar.zst` → `zstd -dc | tar -xf - -C /usr/local`（包内即 `bin/ollama` + `lib/ollama/`）→ 建用户 → 自写 systemd 单元。**效果与官方脚本一致，但不触碰显卡驱动。**

将来升级也按这个流程：下载新版 zst 解压覆盖即可，模型不受影响。

### 网络前提：必须走本机 mihomo 代理

服务器直连 `github.com` 超时，而 `ollama.com/download/*` 会 307 跳到 GitHub Release，因此**下载必须经本机代理**：

```bash
export HTTPS_PROXY=http://127.0.0.1:7897
curl -fL -o /tmp/ollama-linux-amd64.tar.zst https://ollama.com/download/ollama-linux-amd64.tar.zst
```

本机 `mihomo` 监听 `127.0.0.1:7897`，实测可达 10.5MB/s。注意：新版只发 `.tar.zst`，`.tgz` 已 404。

模型拉取（`registry.ollama.ai`）**直连可用**，不需要代理，所以服务单元里**没有**注入 `HTTP_PROXY`；如果哪天拉模型卡住，再给单元加代理环境变量。

### GPU 现状（重要）

截至 `2026-11-17`，机器已连续运行 72 天未重启，出现**驱动核态/用户态版本错配**：

- 已加载内核模块：`595.84`（`/proc/driver/nvidia/version`）
- 用户态库：`595.91.07`（软链接 `libnvidia-ml.so`）
- 结果：`nvidia-smi` 报 `Failed to initialize NVML: Driver/library version mismatch`

**但 CUDA 推理仍然可用**——Ollama 走的是 CUDA 驱动接口（`/dev/nvidia0`、`libcuda`），不依赖 NVML。实测 `ollama ps` 显示 `100% GPU`，日志 `offloaded 29/29 layers to CUDA0`，生成速度 100+ tok/s。所以「nvidia-smi 报错 ≠ GPU 不可用」。

本次已补装新内核的模块包，为重启后彻底修复做准备：

```bash
apt-get install -y --no-install-recommends linux-modules-nvidia-595-open-7.0.0-43-generic
```

结果：驱动被一并升级到 **595.99.02**，`7.0.0-38` 与 `7.0.0-43` 两个内核都有匹配模块；`7.0.0-31`（当前运行内核）目录下已无模块文件，但内存中仍加载着旧模块，**不影响当前运行**。

⚠️ **重启前必读**：`GRUB_DEFAULT=0` 指向 `7.0.0-43-generic`。重启后 `nvidia-smi` 会随驱动对齐而恢复正常。但重启会中断全部容器与服务（mysql/nacos/jenkins/rocketmq/postgres/minio 等）并结束 `linus-stark` 的 GNOME 会话，务必挑时间。重启后建议复核：`nvidia-smi`、`systemctl status ollama`、`ollama ps`。

### 性能实测

| 场景 | 结果 |
| --- | --- |
| 服务启动后首次加载模型 | ≈35s（含 CUDA 上下文初始化，一次性开销） |
| 同进程内换模型/重新加载 | 3.3s |
| 生成速度（1.5B，Q4_K_M，全 GPU） | **106.7 tok/s** |
| 预填充速度 | 1350 tok/s |
| 跨局域网端到端（16 tok） | 0.28s |
| 磁盘顺序读 | 409MB/s（Phison SATA SSD，非 IO 瓶颈） |
| `lfm3:1.7b` 首次冷加载 + 37 tok | 46.8s（其中生成仅 0.40s） |
| `lfm3-omni:2b` 文本请求（模型已驻留） | 端到端 4.3s，生成 107 tok/s |
| `lfm3-omni:2b` 识图（256×256，含视觉编码） | 端到端 10.2s，生成 110.6 tok/s |
| `lfm3-omni:2b` TTS 合成一句话 | 268KB WAV / 24kHz mono，约数秒 |
| `lfm3-omni:2b` STT 识别上一步语音 | 文本与输入逐字一致 |

### 已装模型清单

截至 `2026-11-17`，共 3 个（合计 3.9G）：

| 模型 | 磁盘 | 上下文 | 能力 | 显存驻留 | 实测速度 |
| --- | --- | --- | --- | --- | --- |
| `lfm3:1.7b` | 1.1G | 256K | completion、tools、thinking | 1.2G | 91.7 tok/s |
| `lfm3-omni:2b` | 2.0G | 128K | completion、**vision**、**audio**、**speech**、tools | 1.4–1.6G | 107–110 tok/s |
| `qwen2.5:1.5b` | 986M | 32K | completion、tools | 1.2G | 106.7 tok/s |

`lfm3` 系列是 Liquid AI 的 1.7B/2B 小模型，注意三点：

1. **`lfm3:1.7b` 默认开启 thinking**，会把思维链一起输出。不需要时在请求里传 `"think": false`（实测关掉后延迟明显更低）。
2. **`lfm3-omni:2b` 是非推理模型，不支持 thinking**。它的总参数是 2.17B（backbone 1.7B + CLIP 视觉投影 475M），所以 `ollama show` 里 `parameters` 显示 1.7B 属正常。
3. 两个模型的上下文在 Ollama 里实际只开到 **4096**（`llama-server -c 4096`，按 3.6G 显存推算）。要用长上下文得显式传 `options.num_ctx`，但显存吃紧别开太大，KV cache 会挤爆。

### 多模态与语音端点（lfm3-omni 专有）

Ollama `0.40.13+` 提供 OpenAI 兼容的语音接口，实测均可用：

```bash
# 语音转文字（STT）：multipart 上传音频文件
curl -X POST http://192.168.1.102:11434/v1/audio/transcriptions \
  -F "file=@/tmp/audio.wav" -F "model=lfm3-omni:2b"

# 文字转语音（TTS）：返回音频二进制
curl -X POST http://192.168.1.102:11434/v1/audio/speech \
  -H 'Content-Type: application/json' \
  -d '{"model":"lfm3-omni:2b","input":"你好，这是一次语音合成测试。","response_format":"wav"}' \
  -o out.wav
```

识图走标准 `images` 字段（base64 数组），`/api/generate` 与 `/api/chat` 都支持。

**已知细节**：TTS 返回的 WAV 头里 RIFF 长度字段是 `0xFFFFFFFF`（流式写法），`file` 仍能识别为 24kHz/mono/PCM16，浏览器与 ffmpeg 都能正常播放，但个别严格校验长度的工具可能需要先 `ffmpeg -i in.wav -c copy out.wav` 重写头。

**原生流式语音回答**（`/api/chat` + `output_modalities:["audio"]`）目前只是社区提案（ollama#19396），**尚未合入**，不要照那个写代码，先用上面两个端点。

### 常用命令

```bash
systemctl status ollama            # 状态
systemctl restart ollama           # 重启
journalctl -u ollama -f            # 实时日志
ollama list                        # 已装模型
ollama pull qwen2.5:7b             # 拉模型（7B 需 ~4.7G，注意内存）
ollama ps                          # 当前加载位置（是否 GPU）
ollama stop qwen2.5:1.5b           # 手动卸载显存
```

> **历史坑（已于 `2026-11-17` 消除）**：早期服务只绑 `192.168.1.102`，而 CLI 默认连 `127.0.0.1`，于是 `ollama list` 报：
>
> ```
> Error: could not connect to ollama server, run 'ollama serve' to start it
> ```
>
> **这条报错容易误判成「服务没启动」，其实服务好好的。** 遇到先 `systemctl is-active ollama` + `ss -lntp | grep 11434` 看它绑在哪，再判断是不是连错地址。
>
> 现配置：service 单元 `OLLAMA_HOST=0.0.0.0:11434`（监听全部网卡），CLI 侧 `/etc/profile.d/ollama.sh`（全用户登录 shell）+ `/root/.bashrc` 写 `export OLLAMA_HOST=127.0.0.1:11434`。服务端宽、客户端走回环，换网卡 IP 也不会失联。
>
> 验证：`curl` 打 `127.0.0.1` / `192.168.1.102` / `0.0.0.0` 三个入口的 `/api/version` 均返回 `{"version":"0.40.16"}`；从 Windows 侧 `Invoke-RestMethod http://192.168.1.102:11434/api/tags` 也能列出模型。改绑后 CLI 即使不显式 export 也正常（默认 `127.0.0.1` 已通）。
>
> 单次指定地址：`OLLAMA_HOST=127.0.0.1:11434 ollama list`；以服务用户执行：`su -s /bin/bash ollama -c '...'`。改 unit 前旧文件已备份为 `/root/ollama.service.bak.20261117-210507`。

### 客户端调用

```bash
# 原生 API
curl http://192.168.1.102:11434/api/generate -d '{
  "model": "qwen2.5:1.5b",
  "prompt": "你好",
  "stream": false
}'

# OpenAI 兼容端点（LangChain / openai-sdk / Dify 直接可用）
curl http://192.168.1.102:11434/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"model":"qwen2.5:1.5b","messages":[{"role":"user","content":"hi"}]}'
```

Python（openai SDK）：

```python
from openai import OpenAI
c = OpenAI(base_url="http://192.168.1.102:11434/v1", api_key="ollama")
print(c.chat.completions.create(model="qwen2.5:1.5b",
      messages=[{"role":"user","content":"你好"}]).choices[0].message.content)
```

### /data 权限的特殊处理

`/data` 是 `drwx------ root root`，`ollama` 用户进不去，服务启动会报：

```
Error: mkdir /data/ollama: permission denied: ensure path elements are traversable
```

**不要**把 `/data` 改成 755。正确做法是给单个用户开穿越权限：

```bash
setfacl -m u:ollama:x /data     # 只加 x（可穿越），不改变 700 对其他用户的效果
getfacl -p /data                # 应出现 user:ollama:--x
```

若重建服务器或换用户，记得这一步要重做。

### 排查

| 现象 | 优先检查 |
| --- | --- |
| 服务起不来，日志 `mkdir /data/ollama: permission denied` | `/data` 上是否还有 `user:ollama:--x` 的 ACL（见上节） |
| 服务起不来，日志 `bind: address already in use` | `ss -lntp \| grep 11434`，是否有旧进程或别人占了端口 |
| `nvidia-smi` 报 `Driver/library version mismatch` | 驱动升级后未重启。推理通常仍可用，检查 `ollama ps` 是否 `100% GPU` |
| `ollama ps` 显示 `100% CPU` | GPU 未被识别：查 `journalctl -u ollama \| grep -i cuda`、`/dev/nvidia0` 是否存在 |
| `ollama pull` 卡住不动 | `registry.ollama.ai` 直连是否可用；不行就给服务单元加 `HTTPS_PROXY=http://127.0.0.1:7897` |
| 下载二进制超时 | 必须走 `127.0.0.1:7897` 代理，`.tgz` 已废弃只剩 `.tar.zst` |
| `ollama list` 报 `could not connect to ollama server, run 'ollama serve' to start it` | **先别怀疑服务挂了**。`systemctl is-active ollama` + `ss -lntp \| grep 11434` 看实际绑定地址。`2026-11-17` 起服务已绑 `0.0.0.0:11434`，CLI 默认的 `127.0.0.1` 天然可达；若仍报错，多半是别的用户/cron 缺 `OLLAMA_HOST`，或服务真挂了 |
| CLI 报连不上但其实没有 `OLLAMA_HOST` 变量 | 环境变量只在登录 shell（profile.d）和 root 的 `.bashrc` 里生效；`su` 到别的用户或 cron/脚本里要自己显式导出 |
| 模型回答被截断（`done_reason: length`） | 请求里加大 `num_predict`，或调 `OLLAMA_CONTEXT_LENGTH` |
| 内存告急 / swap 打满 | 只跑 ≤3B 模型；`OLLAMA_MAX_LOADED_MODELS=1` 已限制，7B 以上需先扩内存 |

### 注意

- 服务**无鉴权**：`2026-11-17` 起监听 `0.0.0.0:11434`，同网段任何机器（含访客 Wi-Fi 下的设备）都能调用推理、也能 `pull`/`delete` 模型。用户已确认单人使用、接受此风险。**前提是路由器不把 11434 端口映射到公网**；要对外提供必须先加反向代理 + 认证，**不要直接挂 cpolar**。
- 想收回权限时：把 unit 里 `OLLAMA_HOST` 改回 `192.168.1.102:11434`（或 `127.0.0.1:11434` 仅本机），`daemon-reload` + `restart` 即可，客户端变量无需改动。
- 3.6GiB 显存是硬上限：1.5B（1.2G 驻留）很宽裕，7B Q4 约 4.7G 装不下全部层，会部分 offload 到 CPU，速度掉到个位数 tok/s。想跑 7B 建议选更小量化（`qwen2.5:7b-instruct-q3_K_S`）或直接用 CPU。
- 该机 15GB 内存、swap 已满、多个 Java 服务常驻，**不建议同时加载多个模型**，也不建议常驻 7B 以上。

## 十八、RustFS

截至 `2026-11-17`，新增 RustFS 对象存储，与既有 MinIO 并存（两者都是 S3 协议，端口和凭据独立，业务按需选择其一）。使用 Docker Compose 部署。当天先按参考命令的 `1.0.0-alpha.72` 部署，因控制台无法登录，同日升级为 `1.0.6`（原因见下文）。

### 部署信息

| 项目 | 当前配置 |
| --- | --- |
| Compose 目录 | `/opt/middleware/rustfs` |
| Compose 文件 | `/opt/middleware/rustfs/docker-compose.yml` |
| 容器名 | `rustfs` |
| 镜像 | `rustfs/rustfs:1.0.6`（digest `sha256:ee717072290bec9b11abfc0365f8310b00df1cd9b3639943edae47bdef05044a`，构建于 `2026-11-15`，约 488 MB） |
| 运行身份 | uid/gid `10001:10001`（镜像内用户 `rustfs`，非 root） |
| S3 API 地址 | `http://192.168.1.102:9000` |
| Web 控制台 | `http://192.168.1.102:9001/rustfs/console/` |
| 对外监听 | 仅 `192.168.1.102`，不对全网卡发布 |
| 数据目录 | `/data/rustfs/data` → 容器 `/data`，属主 `10001:10001`，权限 `750` |
| 日志目录 | `/data/rustfs/logs` → 容器 `/logs`，属主 `10001:10001`，权限 `750` |
| 凭据文件 | `/opt/middleware/rustfs/rustfs.env`，权限 `600 root:root` |
| 账号 | `rustfsadmin` / `rustfsadmin`（镜像默认值） |
| 重启策略 | `restart: unless-stopped`（随 Docker 开机自启） |
| 健康检查 | `wget -qO- http://127.0.0.1:9000/health`，每 `30s`，启动宽限 `30s` |
| ulimit | `nofile 65536`（软/硬） |
| 日志 | Docker `json-file` 轮转，单文件 `20 MB`，保留 `5` 个 |

首次启动实测：`alpha.72` 容器 `5s` 进入 `healthy`；`1.0.6` 为 `6s`，重启次数 `0`；`docker compose down` + `up` 后桶与对象均保留。

### 与参考 `docker run` 命令的差异（为什么改成 Compose）

| 参考写法 | 当前采用 | 原因 |
| --- | --- | --- |
| `-v rustfs-data:/data` 具名卷 | `- /data/rustfs/data:/data` | 规范要求业务数据落在 `/data`，具名卷藏在 `/var/lib/docker/volumes` 下，备份和迁移容易漏 |
| `-p 9000:9000` / `-p 9001:9001` | `192.168.1.102:9000:9000` / `...:9001:9001` | 与 MinIO 一致的最小暴露原则，只监听局域网地址 |
| `-e RUSTFS_ACCESS_KEY=...` 等写在命令行 | `env_file: rustfs.env` | 凭据不落进 `docker inspect` 之外的命令历史，文件权限 `600` 便于审计 |
| 末尾再传 `--access-key` / `--secret-key` | 只保留 `RUSTFS_*` 环境变量 | 同名参数与环境变量等价，重复写两份容易改一处漏一处（命令行参数会覆盖环境变量） |
| 未挂载 `/logs` | `- /data/rustfs/logs:/logs` | 镜像声明了 `/data` 和 `/logs` 两个 `VOLUME`，不显式挂载会生成匿名卷 |
| 未指定 `--user`、未处理目录属主 | 目录 `chown 10001:10001` | 镜像以非 root 运行，属主不对会启动失败 |
| 镜像 `1.0.0-alpha.72` | `1.0.6` | `alpha.72` 的 STS AssumeRole 未实现，控制台"密钥登录"必失败（见下节） |

### 为什么从 alpha.72 升到 1.0.6（2026-11-17 实测）

按参考命令部署 `1.0.0-alpha.72` 后，控制台账号密码正确也登不进去。定位过程与结论：

| 证据 | 说明 |
| --- | --- |
| 控制台登录请求**已到达服务器** | 服务端日志在登录时刻记录 `load_user_identity failed: no such user, name: rustfsadmin` 与 `UnauthorizedAccess: Your account is not signed up2`，说明不是网络/端口问题 |
| `aws sts assume-role` 打 `9000` | 返回 `<?xml ...><Code>NotImplemented</Code><Message>Unknown operation</Message>`，即 **`alpha.72` 没有实现 STS AssumeRole** |
| 控制台前端代码 | `/rustfs/console/_nuxt/*.js` 中 "密钥登录" 走的就是 STS `AssumeRole`（`arn:aws:iam::*:role/Admin`） |
| 换 `1.0.6` 复测 | 同一命令返回 `CREDENTIALS ...` 临时凭据，控制台登录成功，桶页面正常 |

结论：`alpha.72`（2025-12 构建）的控制台与后端 STS 能力不匹配，属于版本缺陷，与凭据、端口绑定、目录权限都无关。`1.0.6` 是 2026-11-15 的正式版，STS 已实现。

升级时的注意点：

- **数据格式不兼容**，不能直接换 tag 复用 `alpha` 的 `/data`。本次数据为空，把旧目录整体改名为 `/data/rustfs/data.alpha72-20261117-213742` 留档，再新建空目录初始化。
- 若已有真实对象，必须先用 S3 客户端把对象 `mc mirror` 出去，再用新版重建并灌回。
- 控制台前端从 Nuxt 换成了 Next.js，**旧浏览器缓存会失效**：升级后首次访问要强制刷新（`Ctrl+Shift+R`），必要时清掉该站点的 localStorage（旧版会存 `rustfs-server-host`）。

`docker-compose.yml`：

```yaml
services:
  rustfs:
    image: rustfs/rustfs:1.0.6
    container_name: rustfs
    restart: unless-stopped
    env_file:
      - rustfs.env
    ports:
      - "192.168.1.102:9000:9000"
      - "192.168.1.102:9001:9001"
    volumes:
      - /data/rustfs/data:/data
      - /data/rustfs/logs:/logs
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://127.0.0.1:9000/health >/dev/null 2>&1 || exit 1"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 30s
    logging:
      driver: json-file
      options:
        max-size: "20m"
        max-file: "5"
```

`rustfs.env`：

```ini
RUSTFS_ACCESS_KEY=rustfsadmin
RUSTFS_SECRET_KEY=rustfsadmin
RUSTFS_ADDRESS=:9000
RUSTFS_CONSOLE_ENABLE=true
RUSTFS_CONSOLE_ADDRESS=:9001
RUSTFS_VOLUMES=/data
RUSTFS_OBS_LOG_DIRECTORY=/logs
```

所有 `RUSTFS_*` 环境变量与命令行参数一一对应（`rustfs --help` 中每个参数都标了 `[env: RUSTFS_XXX]`）。容器入口脚本 `/entrypoint.sh` 会按 `RUSTFS_VOLUMES` 建目录、按 `RUSTFS_OBS_LOG_DIRECTORY` 建日志目录，并在检测到默认凭据时打印告警。

### 首次部署或重建

```bash
# 1. 目录（数据/日志目录属主必须是 10001）
install -d -m 755 /opt/middleware/rustfs
install -d -m 755 /data/rustfs
install -d -m 750 -o 10001 -g 10001 /data/rustfs/data
install -d -m 750 -o 10001 -g 10001 /data/rustfs/logs

# 2. 配置文件（env 必须 600）
install -m 644 docker-compose.yml /opt/middleware/rustfs/docker-compose.yml
install -m 600 rustfs.env        /opt/middleware/rustfs/rustfs.env

# 3. 启动
cd /opt/middleware/rustfs
docker compose config --quiet
docker compose up -d
docker compose ps
docker logs --tail 40 rustfs
```

正常日志形如：

```text
Initializing data directories: /data
Initializing log directory: /logs
!!!WARNING: Using default RUSTFS_ACCESS_KEY or RUSTFS_SECRET_KEY. Override them in production!
Starting: /usr/bin/rustfs  /data
RustFS API: http://172.26.0.2:9000  http://127.0.0.1:9000
Console WebUI available at: http://172.26.0.2:9001/rustfs/console/index.html
```

`WARNING` 那行是因为当前用着镜像默认凭据，不是故障。

### 管理命令

```bash
cd /opt/middleware/rustfs
docker compose up -d
docker compose stop
docker compose restart
docker compose ps
docker compose down
docker logs --tail 100 rustfs
docker logs -f rustfs
```

状态与健康：

```bash
docker inspect -f 'status={{.State.Status}} health={{if .State.Health}}{{.State.Health.Status}}{{end}} restarts={{.RestartCount}}' rustfs
curl -sS -o /dev/null -w 'health=%{http_code}\n' http://192.168.1.102:9000/health
curl -sS -o /dev/null -w 'console=%{http_code}\n' http://192.168.1.102:9001/rustfs/console/index.html
```

`docker compose down` 不会删除 `/data/rustfs/data`；不要使用 `down -v`，也不要直接删该目录。

### 验证与验收记录（2026-11-17 实测）

| 检查项 | 结果 |
| --- | --- |
| 容器状态 | `running` / `healthy`，重启次数 `0`，启动耗时约 `5s` |
| `GET /health` | `200` |
| `GET /health/ready` | `403`（该端点未开放在无鉴权访问下，属正常现象，不影响使用） |
| 控制台 `GET /rustfs/console/index.html` | `200` |
| 控制台根路径 `GET /` | `403` —— **要用 `/rustfs/console/` 访问**，不是 `http://ip:9001/` |
| S3 建桶 / 上传 / 读取 / 列举 / 删桶 | 全部成功（`mc mb`、`mc pipe`、`mc cat`、`mc ls`、`mc rb`） |
| `mc admin info` | 可用，返回 `Version: ...@1.0.6`、`Drives: 1/1 OK`（`alpha.72` 下不可用，见下） |
| STS `AssumeRole`（控制台登录链路） | 返回临时凭据 `CREDENTIALS ...`，正常 |
| 控制台登录（`rustfsadmin` / `rustfsadmin`） | 浏览器实测登录成功，进入"存储桶"页面，`0 B / 0 对象 / 0 桶` |
| 跨机访问 | 从局域网另一台 Windows 电脑经 `192.168.1.102:9001` 登录成功，绑 `192.168.1.102` 不影响局域网访问 |
| `docker compose down` + `up` 后数据 | 桶与对象均保留（`alpha.72` 阶段验证） |
| 数据落盘位置 | `/data/rustfs/data/.rustfs.sys`（元数据）+ 各桶目录 |

用服务器上已有的 `minio` 容器自带 `mc` 验证（不写别名、不污染配置）：

```bash
docker exec -e MC_HOST_rustfs=http://rustfsadmin:rustfsadmin@192.168.1.102:9000 minio mc ls rustfs
printf 'hello\n' | docker exec -i -e MC_HOST_rustfs=http://rustfsadmin:rustfsadmin@192.168.1.102:9000 \
  minio mc pipe rustfs/demo/hello.txt
```

`1.0.6` 下 `mc admin info` / `mc admin user list` 均可用（后者为空表示只有 root 身份）。`1.0.0-alpha.72` 下这两个命令会报 `Unable to get service info. The specified bucket does not exist.` —— 那是 alpha 版本管理 API 未对齐，与 S3 读写无关。

### 客户端连接

- Endpoint：`http://192.168.1.102:9000`
- **必须开启 path-style**（`http://host:port/bucket/key`），服务以 IP + 端口提供
- Region 可填任意值，通常 `us-east-1`
- Access Key / Secret Key：`rustfsadmin` / `rustfsadmin`

Java / Spring（AWS SDK v2）：

```java
S3Client s3 = S3Client.builder()
    .endpointOverride(URI.create("http://192.168.1.102:9000"))
    .region(Region.US_EAST_1)
    .credentialsProvider(StaticCredentialsProvider.create(
        AwsBasicCredentials.create("rustfsadmin", "rustfsadmin")))
    .forcePathStyle(true)   // 关键
    .build();
```

控制台：浏览器打开 `http://192.168.1.102:9001/rustfs/console/`，用同一组凭据登录。

### 与 MinIO 的分工

| | MinIO（`cgr.dev/chainguard/minio`） | RustFS（`rustfs/rustfs:1.0.6`） |
| --- | --- | --- |
| 端口 | `18090` / `18091` | `9000` / `9001` |
| 版本属性 | Chainguard 重建，含 CVE 回移，稳定 | 上游正式版 `1.0.6`（2026-11-15），控制台与 STS 均已可用 |
| 凭据 | `admin` / `12345678` | `rustfsadmin` / `rustfsadmin` |
| 适用 | 现有业务（绘梦等）依赖的稳定对象存储 | 新项目试用 / 需要 RustFS 特性时 |

两套都保留，互不备份；**不要**把同一个桶名和同一份业务数据只放在其中一套上跑生产。RustFS 侧不要再用 `alpha` 版本。

### 备份

数据全部在 `/data/rustfs/data`，直接 `cp` 不是一致性备份。正确做法是镜像到另一位置：

```bash
docker exec -e MC_HOST_rs=http://rustfsadmin:rustfsadmin@192.168.1.102:9000 \
  -e MC_HOST_bak=http://<备份目标> minio mc mirror --overwrite rs bak
```

目前**尚未配置自动备份**，需要时单独加定时任务并做一次恢复演练。

### 排查

| 现象 | 优先检查 |
| --- | --- |
| 启动即退出，日志 `Permission denied` | `/data/rustfs/data`、`/data/rustfs/logs` 属主是否 `10001:10001` |
| 控制台打不开（`/` 返回 403） | 用 `http://192.168.1.102:9001/rustfs/console/`，根路径本来就 403 |
| 控制台登录报 `UnauthorizedAccess` / `Your account is not signed up2` | 先看服务端日志时间和请求是否到达；若是 `1.0.0-alpha.72`，属 STS 未实现，**升级镜像**，不要改密码或端口 |
| `aws sts assume-role` 返回 `<Code>NotImplemented</Code>` | 同一条：该版本没有 STS，控制台登录链路必然失败 |
| `mc admin info` 报 bucket does not exist | `alpha.72` 管理 API 未实现；`1.0.6` 已正常。不是 S3 故障 |
| 客户端 `NoSuchBucket` / 域名解析失败 | 未开启 path-style |
| 客户端 `403 SignatureDoesNotMatch` | 凭据错误或系统时间偏差过大 |
| 局域网访问不通 | 端口只绑了 `192.168.1.102`，`ss -lntp \| grep -E ':(9000\|9001)'` 核对 |
| 换版本后启动异常 | `alpha` 版本间数据格式可能不兼容，先在 `/data/rustfs` 副本上验证再切 |
| 容器反复重启 | `docker logs --tail 200 rustfs`，多为目录权限或磁盘满 |

### 安全

- 凭据 `rustfsadmin` / `rustfsadmin` 是**镜像默认值**，与库内其他中间件的弱口令同级别，仅适合受控局域网。改法：编辑 `/opt/middleware/rustfs/rustfs.env` 的 `RUSTFS_ACCESS_KEY` / `RUSTFS_SECRET_KEY`，`docker compose up -d` 重建容器（**已写入的对象不受影响，但调用方配置要同步改**）。
- 服务本身无 TLS，内网明文传输。需要公网访问时必须在前面加反向代理做 HTTPS，不要直接把 `9000/9001` 暴露到公网或挂 cpolar。
- 两个端口都只监听 `192.168.1.102`，因此 Tailscale 网段（`100.83.74.68`）**访问不到**；需要远程访问时把 Compose 里的绑定地址改成 `0.0.0.0` 或 `100.83.74.68`，再自行评估暴露面。

## 十九、Elasticsearch（2026-11-18）

| 项目 | 部署配置 |
| --- | --- |
| 版本与容器 | Elasticsearch 8.17.0 + IK 8.17.0，容器 `elasticsearch` |
| 镜像 | `ragent/elasticsearch-ik:8.17.0`，IK 随镜像构建 |
| 端口 | `127.0.0.1:19200 → 9200`。宿主机 `9200` 由 cpolar 使用 |
| 配置目录 | `/opt/middleware/elasticsearch`，含 `docker-compose.yml`、`Dockerfile` 和 `conf/analysis-ik-8.17.0.zip` |
| 数据目录 | `/data/elasticsearch/data` |
| 资源 | JVM 堆 512 MiB，容器上限 1.25 GiB、2 核 |
| 验收 | 集群状态 `green`，`ik_max_word`、`ik_smart` 均已验证 |

三套服务仅在 Ubuntu 回环地址监听。Windows 使用现有的 SSH 别名转发端口，保持命令窗口打开即可访问：

```powershell
ssh -N -o ExitOnForwardFailure=yes -L 127.0.0.1:19200:127.0.0.1:19200 -L 127.0.0.1:9621:127.0.0.1:9621 -L 127.0.0.1:7474:127.0.0.1:7474 -L 127.0.0.1:7687:127.0.0.1:7687 -L 127.0.0.1:13000:127.0.0.1:13000 myServer
```

在 Ubuntu 上执行日常检查：

```bash
cd /opt/middleware/elasticsearch
docker compose ps
curl -fsS http://127.0.0.1:19200/_cluster/health
curl -fsS http://127.0.0.1:19200/_analyze -H 'Content-Type: application/json' --data '{"analyzer":"ik_smart","text":"南京市长江大桥"}'
```

Ragent 在 Windows 通过转发接入时，关键词地址使用 `rag.keyword.es.uris=http://127.0.0.1:19200`。启用时将 `rag.keyword.type` 设为 `es`，并将 `rag.search.channels.keyword.enabled` 设为 `true`。

## 二十、LightRAG + Neo4j（2026-11-18）

| 项目 | 部署配置 |
| --- | --- |
| 版本与容器 | LightRAG 1.5.7（镜像 digest 固定），Neo4j Community 5.26.33 + APOC 5.26.33 + GDS 2.13.11 |
| 配置目录 | `/opt/middleware/graphrag`，含 Compose、Neo4j Dockerfile 和 `conf/graph-data-science.jar` |
| 管理入口 | LightRAG `http://127.0.0.1:9621/webui`，Neo4j `http://127.0.0.1:7474/browser/`，Bolt `bolt://127.0.0.1:7687` |
| 数据目录 | `/data/graphrag/lightrag`、`/data/graphrag/neo4j/data`、`/data/graphrag/neo4j/logs` |
| PostgreSQL | 复用现有 `postgres:5432` 服务，独立库和角色均为 `ragent_lightrag`，pgvector 0.8.7 |
| 模型 | 百炼 `qwen-plus-latest` 和 `text-embedding-v4`（1536 维） |
| 凭据 | `/opt/middleware/graphrag/.env`（权限 600），Neo4j 用户为 `neo4j`，密码变量为 `NEO4J_PASSWORD` |

```bash
cd /opt/middleware/graphrag
docker compose ps
curl -fsS http://127.0.0.1:9621/health
```

Ragent 通过转发访问 `rag.graph.lightrag.base-url=http://127.0.0.1:9621`。启用时将 `rag.graph.type` 设为 `lightrag`，并将 `rag.search.channels.graph.enabled` 设为 `true`。数据库备份包含 `ragent_lightrag` 和 Neo4j 图数据两个部分。LightRAG 已连通 PostgreSQL、Neo4j 和百炼，实测模型返回 1536 维向量。测试文档成功抽取出 4 个节点和 3 条关系，验证结束后已移除测试文档。

## 二十一、LangFuse（2026-11-18）

| 项目 | 部署配置 |
| --- | --- |
| 版本 | LangFuse 4.82.0，Web 和 Worker 镜像按 digest 固定 |
| 容器 | `langfuse-web`、`langfuse-worker`、`langfuse-postgres`、`langfuse-redis`、`langfuse-clickhouse`、`langfuse-minio` |
| Web 与 OTLP | `http://127.0.0.1:13000`、`http://127.0.0.1:13000/api/public/otel/v1/traces`。宿主机 `3000` 由 ToonFlow 使用 |
| 配置目录 | `/opt/middleware/langfuse`，含 Compose、ClickHouse 配置、`.env`（权限 600） |
| 数据目录 | `/data/langfuse/{postgres,redis,minio}` 和 `/data/langfuse/clickhouse/{data,logs}` |
| 登录 | `admin@ragent.dev`，登录密码见 `.env` 的 `LF_ADMIN_PASSWORD` |
| 追踪密钥 | `.env` 的 `LF_PUBLIC_KEY` 和 `LF_SECRET_KEY`，项目名 `ragent-service` |

Redis、PostgreSQL、ClickHouse 和 MinIO 在专用 Docker 网络中运行。Redis 使用 `noeviction`，ClickHouse 容器内存上限 3 GiB。已通过 Web、Worker 健康检查，并验证测试 OTLP 事件写入 ClickHouse。

```bash
cd /opt/middleware/langfuse
docker compose ps
curl -fsS http://127.0.0.1:13000/api/public/health
```

Ragent 启用追踪时将 `agent.trace.enabled` 设为 `true`，把 `agent.trace.endpoint` 指向上述 OTLP URL，并从本机配置传入对应的项目公钥与密钥。

三套服务各自在对应目录执行 `docker compose stop` 停用，执行 `docker compose up -d --pull never --wait` 恢复。持久化数据位于表中列出的 `/data` 目录。

