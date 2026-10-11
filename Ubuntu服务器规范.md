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
| MySQL |  | 192.168.1.102:3306 |
| Redis |  | 192.168.1.102:6379 |

### 服务账号密码

| 服务  | 账号  | 密码   |
| ----- | ----- | ------ |
| nacos | nacos | nacos |
| mysql | `root` | 123456 |
| redis | root | 123456 |
| XXL-JOB | `admin` | `123456` |



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
