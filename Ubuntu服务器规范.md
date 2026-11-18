# Ubuntu服务器规范

## 一、文件目录

根据服务器实际目录布局梳理（`2026-11-18` 核对）：

```text
/
├── etc/                          # 系统与原生服务配置（Nginx、SSH、systemd 等）
├── usr/local/etc/cpolar/         # cpolar 配置（cpolar.yml，属主 nobody:nogroup）
├── opt/                          # 自部署软件、服务与工具
│   ├── apps/                     # 业务应用
│   │   ├── smart-university/     # 选课平台后端（2026-10-25 已停用）
│   │   └── toonflow/             # ToonFlow 业务项目（运行中）
│   ├── middleware/               # Docker Compose 中间件
│   │   ├── elasticsearch/
│   │   ├── graphrag/             # LightRAG + Neo4j
│   │   ├── jenkins/
│   │   ├── langfuse/
│   │   ├── minio/
│   │   ├── mysql/
│   │   ├── nacos/
│   │   ├── postgres/
│   │   ├── redis/
│   │   ├── rocketmq/
│   │   ├── rustfs/
│   │   ├── skywalking/
│   │   └── xxl-job/
│   └── tools/                    # 手动安装的工具
│       └── skywalking-java-agent-9.7.0/
│
├── data/                         # 重要业务与持久化数据【重点，710+ollama ACL】
│   ├── backup/                   # 各服务与数据库备份（jenkins/mysql/postgres/xxl-job 等）
│   ├── elasticsearch/
│   ├── graphrag/                 # lightrag/ 与 neo4j/
│   ├── jenkins/
│   ├── langfuse/                 # clickhouse/、minio/、postgres/、redis/
│   ├── minio/
│   ├── mysql/
│   ├── nacos/
│   ├── ollama/                   # models/（ollama:ollama 750）
│   ├── postgres/
│   ├── redis/
│   ├── rocketmq/
│   ├── rustfs/
│   └── skywalking/
│
├── var/
│   ├── log/                      # 服务日志
│   │   ├── apps/                 # 业务日志（course-selection-web/minio/smart-university/toonflow*.log/xxl-job）
│   │   ├── cpolar/
│   │   ├── nacos/
│   │   └── nginx/
│   └── www/                      # 网站静态文件
│       ├── course-selection-web/ # 选课平台前端（已停用保留）
│       └── html/
│
├── home/
│   └── linus-stark/              # 个人桌面环境（桌面/文档/下载/图片/公共/snap）
│
└── tmp/                          # 临时文件（tmpfs，重启清空）
```

**每个中间件服务内部**：

```text
/opt/middleware/<服务名>/
├── docker-compose.yml
├── <服务名>.env / .env           # 凭据文件（权限 600 root:root）
└── conf/                         # 可选，自定义配置或 Dockerfile
```

**业务项目统一**：

```text
/opt/apps/<项目名>/
├── docker-compose.yml / compose.yaml
├── .env / secrets/
├── conf/
└── scripts/
```

| 类型           | 放哪里            | 示例                                  |
| -------------- | ----------------- | ------------------------------------- |
| 系统与原生配置 | `/etc`            | Nginx、SSH、systemd（含 Ollama unit） |
| 已部署业务服务 | `/opt/apps`       | `toonflow`、`smart-university`        |
| 中间件         | `/opt/middleware` | MySQL、PostgreSQL、Redis、MinIO、RustFS、ES 等 |
| 手动工具       | `/opt/tools`      | SkyWalking Java Agent                 |
| 持久化数据     | `/data`           | 数据库、对象存储、模型文件、`/data/backup` |
| 日志           | `/var/log`        | `/var/log/apps`、`nginx`、`cpolar`、`nacos` |
| 网站静态文件   | `/var/www`        | 前端构建产物 `dist`                   |
| 个人文件       | `/home/linus-stark` | 桌面、文档、下载等                  |

## 二、服务

### 服务器 IP 与 SSH

- 服务器 IP：`192.168.1.102`
- SSH 用户：`linus-stark`，密码/认证信息从受控的密码管理工具获取。
- `root` 凭据不得写入本文档或代码仓库。

### 服务端口

| 服务                | 描述                                      | 地址                                        |
| ------------------- | ----------------------------------------- | ------------------------------------------- |
| Cockpit             | 服务器管理                                | 192.168.1.102:9090                          |
| cpolar              | 内网穿透管理                              | 192.168.1.102:9200                          |
| Tailscale           | 私有组网与远程访问                        | 100.83.74.68（SSH：22/TCP）                 |
| RocketMQ NameServer | RocketMQ 注册发现                         | 192.168.1.102:9876                          |
| RocketMQ Broker     | RocketMQ 消息 Broker                      | 192.168.1.102:10911                         |
| RocketMQ Proxy      | RocketMQ 5.x 客户端入口                   | 192.168.1.102:8081                          |
| RocketMQ Dashboard  | RocketMQ Web 管理控制台                   | 192.168.1.102:8082                          |
| SkyWalking UI       | 链路追踪 Web 控制台                       | 192.168.1.102:18082                         |
| SkyWalking OAP gRPC | Java 等 Agent 上报入口                    | 192.168.1.102:11800                         |
| SkyWalking OAP HTTP | 查询及 HTTP 上报入口                      | 192.168.1.102:12800                         |
| Nacos Console       | Nacos Web 管理控制台                      | 192.168.1.102:18080                         |
| Nacos Server        | Nacos 注册中心/配置中心                   | 192.168.1.102:8848                          |
| Nacos gRPC          | Nacos 客户端 gRPC 通信                    | 192.168.1.102:9848                          |
| Jenkins             | 持续集成与持续部署                        | 192.168.1.102:18081                         |
| XXL-JOB Admin       | 任务调度管理控制台                        | 192.168.1.102:18084                         |
| Nginx               | 前端静态站点入口（选课平台）              | 192.168.1.102:80（2026-10-25 起已停止）     |
| SmartUniversity API | 选课平台后端（容器 `smart-university`）   | 192.168.1.102:18083（2026-10-25 起已停止）  |
| MinIO S3 API        | 对象存储 S3 接口                          | 192.168.1.102:18090                         |
| MinIO Console       | MinIO Web 控制台                          | 192.168.1.102:18091                         |
| RustFS S3 API       | 对象存储 S3 接口（RustFS，2026-11-17 起） | 192.168.1.102:9000                          |
| RustFS Console      | RustFS Web 控制台（入口 `/rustfs/console/`） | 192.168.1.102:9001                       |
| MySQL               |                                           | 192.168.1.102:3306                          |
| PostgreSQL          | 关系型数据库（容器 `postgres`）           | 192.168.1.102:5432                          |
| Redis               |                                           | 192.168.1.102:6379                          |
| Ollama              | 本地大模型推理服务（systemd 原生）        | 0.0.0.0:11434（局域网 `192.168.1.102:11434` 可达） |
| Elasticsearch       | Ragent 关键词检索（IK 中文分词）          | 127.0.0.1:19200（SSH 转发）                 |
| LightRAG            | 知识图谱 API 与 WebUI                     | 127.0.0.1:9621（SSH 转发）                  |
| Neo4j Browser       | 图数据库 Web 控制台                       | 127.0.0.1:7474（SSH 转发）                  |
| Neo4j Bolt          | 图数据库驱动接口                          | 127.0.0.1:7687（SSH 转发）                  |
| LangFuse            | Agent 追踪控制台与 OTLP 入口              | 127.0.0.1:13000（SSH 转发）                 |

### 服务账号密码

| 服务     | 账号        | 密码                     |
| -------- | ----------- | ------------------------ |
| nacos    | nacos       | nacos                    |
| mysql    | `root`      | 123456                   |
| redis    | root        | 123456                   |
| XXL-JOB  | `admin`     | `123456`                 |
| minio    | `admin`     | `12345678`               |
| rustfs   | `rustfsadmin` | `rustfsadmin`          |
| postgres | `root`      | `123456`（库 `toonflow`） |

## 三、容器命名与通用运维规范

### 容器命名规范

中间件使用简洁统一名称（`mysql`、`redis`、`postgres`、`rmqnamesrv`、`rmqbroker`、`rmqproxy`、`rmqdashboard`、`skywalking-banyandb`、`skywalking-oap`、`skywalking-ui`、`jenkins`、`xxl-job-admin`、`minio`、`rustfs`、`elasticsearch`、`lightrag`、`neo4j`、`langfuse-*`）；业务项目统一使用 `项目名-模块名`（如 `smart-university`、`toonflow-server-1`、`toonflow-agentRuntime-1`、`toonflow-postgres-1`）。

### Docker Compose 通用管理命令

除下文各章节注明的特殊命令外，所有部署在 `/opt/middleware/<服务>` 或 `/opt/apps/<项目>` 的容器化服务通用以下命令：

```bash
cd /opt/middleware/<服务目录>
docker compose config --quiet    # 校验配置语法
docker compose up -d             # 启动（可加 --wait 等待健康检查）
docker compose stop              # 停止（保留容器和网络）
docker compose restart           # 重启
docker compose ps                # 查看状态
docker logs --tail 100 <容器名>  # 查看最近日志（加 -f 实时跟踪）
```

> **破坏性命令说明**：`docker compose down` 会删除容器与 Compose 网络并中断服务，但**不会删除宿主机 `/data/...` 挂载的数据目录**。禁止随意执行 `docker compose down -v`（会删除具名卷）或 `docker system prune`（可能误删其他服务镜像或构建缓存）。清理镜像前，先执行 `docker ps -a --filter ancestor=<repo:tag> --format '{{.Names}}'` 确认无容器引用后再 `docker rmi`。

### 安全与网络通用原则

- **局域网弱口令风险**：服务器当前 UFW 未启用，部分中间件使用弱口令或默认凭据（如 `123456`、`rustfsadmin`），仅限受控局域网访问。禁止将未鉴权或弱口令端口（如 `80`、`5432`、`9000/9001`、`11434`、`18081`、`18084`、`18090/18091`）直接暴露到公网或挂载 cpolar。
- **公网暴露前提**：若确需对外开放，必须配置反向代理 + HTTPS + 访问控制/来源白名单，并轮换强密码。
- **凭据文件权限**：独立 `.env` 或配置文件统一设为 `600 root:root`，不得提交到代码仓库。
- **整机硬件限制**：服务器总内存 `16 GB`（受限于多服务常驻与 swap 空间），调整数据库缓存或加载本地大模型前先 `free -m` 确认余量。

## 四、RocketMQ

- 部署目录：`/opt/middleware/rocketmq`，数据目录：`/data/rocketmq/broker/store`（`docker compose down` 会保留消息数据）。
- **RocketMQ Dashboard**：容器名 `rmqdashboard`（镜像 `apacherocketmq/rocketmq-dashboard:2.1.0`），与 NameServer 共用 `rocketmq` 网络，连接 `rmqnamesrv:9876`，宿主机仅监听 `192.168.1.102:8082`。原配置备份在 `/opt/middleware/rocketmq/docker-compose.yml.pre-dashboard`。

单独管理 Dashboard 与验证：

```bash
cd /opt/middleware/rocketmq
docker compose up -d --no-deps dashboard   # 仅启动 Dashboard
docker compose stop dashboard              # 仅停止 Dashboard
docker compose restart dashboard           # 仅重启 Dashboard
curl -I http://192.168.1.102:8082/
curl -sS http://192.168.1.102:8082/cluster/list.query
```

页面应返回 `HTTP 200`，集群接口返回 `"status":0` 且可见 Broker。

## 五、Nacos

- 部署目录：`/opt/middleware/nacos`，容器名 `nacos`。
- 数据目录：`/data/nacos/derby-data`，日志目录：`/var/log/nacos`。

## 六、MySQL

- 部署目录：`/opt/middleware/mysql`，容器名 `mysql`。
- 数据目录：`/data/mysql`（挂载到容器 `/var/lib/mysql`，`docker compose down` 不会删除数据库数据）。

## 七、SkyWalking

### 部署信息

部署目录为 `/opt/middleware/skywalking`：

| 服务 | 镜像 | 作用 |
| --- | --- | --- |
| `skywalking-banyandb` | `apache/skywalking-banyandb:0.10.3` | 持久化存储（数据目录 `/data/skywalking/banyandb`，仅内部网络通信） |
| `skywalking-oap` | `apache/skywalking-oap-server:10.4.0-java17` | 接收和分析数据（`11800` gRPC、`12800` HTTP） |
| `skywalking-ui` | `apache/skywalking-ui:10.4.0-java17` | Web 控制台（`http://192.168.1.102:18082/`，无登录认证） |

### Java Agent 接入

官方 Java Agent `9.7.0` 已安装在 `/opt/tools/skywalking-java-agent-9.7.0`。普通 JAR 启动示例（容器部署时需挂载 Agent 目录并配置 `SW_LOGGING_OUTPUT=CONSOLE` 或可写日志目录）：

```bash
export SW_AGENT_NAME=my-service
export SW_AGENT_COLLECTOR_BACKEND_SERVICES=192.168.1.102:11800
java -javaagent:/opt/tools/skywalking-java-agent-9.7.0/skywalking-agent.jar -jar /opt/apps/my-service/app.jar
```

### 验证

```bash
cd /opt/middleware/skywalking
docker compose up -d --wait
curl -fsS http://192.168.1.102:12800/healthcheck
curl -I http://192.168.1.102:18082/
```

## 八、Jenkins

### 部署信息

| 项目 | 当前配置 |
| --- | --- |
| Compose 目录 | `/opt/middleware/jenkins`（含 `docker-compose.yml` 与 `Dockerfile`） |
| 容器名 / 镜像 | `jenkins` / `myubuntu/jenkins:lts-jdk21-docker`（基于 `jenkins/jenkins:lts-jdk21`） |
| 版本 | Jenkins `2.568.3` / Eclipse Temurin `21.0.12.1` LTS |
| Web 地址 | `http://192.168.1.102:18081/`（端口映射 `18081:8080`，Agent `50000` 未暴露） |
| 数据目录 | `/data/jenkins` → `/var/jenkins_home`（备份于 `/data/backup/jenkins`） |
| 宿主机挂载 | `/var/run/docker.sock`（`group_add: ["973"]`）与 `/opt/apps` |
| 重启策略 | `unless-stopped` |

- 挂载 Docker socket 权限极高，只允许受信任的管理员修改任务，禁止构建未经审查的 PR。
- 若重建容器后出现 Docker `permission denied`，先用 `stat -c '%g' /var/run/docker.sock` 核对 gid 是否与 Compose `group_add` 一致，切勿将 socket 改为 `666`。
- 若邮件或 Webhook 链接出现多余的 `/instance/` 前缀，在“系统管理 → 系统配置 → Jenkins URL”核对为 `http://192.168.1.102:18081/`。

### 备份、升级与验证

```bash
# 验证
curl -sSI http://192.168.1.102:18081/login | grep -i '^X-Jenkins:'
docker exec jenkins java -version

# 停止并完整备份（破坏性升级或改关键配置前必做）
set -euo pipefail
cd /opt/middleware/jenkins
backup_file="/data/backup/jenkins/jenkins-$(date +%Y%m%d-%H%M%S).tar.gz"
sudo install -d -m 700 /data/backup/jenkins
docker compose stop
trap 'docker compose start' EXIT
sudo tar -C /data -czf "$backup_file" jenkins
sudo chmod 600 "$backup_file" && sudo tar -tzf "$backup_file" > /dev/null
sudo sha256sum "$backup_file" | sudo tee "$backup_file.sha256" > /dev/null
trap - EXIT
docker compose start

# 升级镜像（须先完成上述备份）
docker compose build --pull jenkins
docker compose up -d jenkins
docker exec jenkins docker compose version
```

> 恢复归档会覆盖现有 `/data/jenkins` 配置与密钥，属于破坏性操作，执行前须先停止容器、备份现状并由管理员确认。

### SmartUniversity 流水线（已于 2026-10-25 停用）

- **当前状态**：后端容器 `smart-university` 已随前端站点停止，端口 `18083` 无监听。
- **保留配置**：Jenkins 任务 `SmartUniversity`（监听 `https://github.com/XiaoTianbai520/SmartUniversity.git` `master` 分支，`H/2 * * * *` 轮询），应用目录 `/opt/apps/smart-university`，应用日志 `/var/log/apps/smart-university`（滚动保留 14 天 / 上限 2 GB），运行环境变量 `SPRING_PROFILES_ACTIVE=prod`、`SMART_UNIVERSITY_SERVER_IP=192.168.1.102`，镜像 `smart-university:current`（及回滚用 `smart-university:previous`）。
- **恢复与安全清理**：
  - 恢复：`cd /opt/apps/smart-university && docker compose start`。
  - 日常释放磁盘可安全执行 `docker image prune -f` 与 `docker builder prune -f`，但注意保留 `smart-university:previous`（唯一回滚点）及基础镜像 `eclipse-temurin:21-jre-jammy`、`maven:3.9.11-*`。

## 九、Tailscale

### 部署信息

使用官方 APT 源安装并由 `systemd`（`tailscaled.service`）管理：

| 项目 | 当前配置 |
| --- | --- |
| 版本 / 源代号 | `1.102.4` / Ubuntu `26.04`（`resolute`） |
| 源与密钥 | `/etc/apt/sources.list.d/tailscale.list`、`/usr/share/keyrings/tailscale-archive-keyring.gpg` |
| Tailscale IPv4 / IPv6 | `100.83.74.68` / `fd7a:115c:a1e0::e92e:4a45` |
| MagicDNS | `linus-stark-nhx0db-de.taild30eeb.ts.net` |
| 状态目录 | `/var/lib/tailscale`（`700 root:root`），状态文件 `tailscaled.state`（`600`） |

未启用出口节点、子网路由与 Tailscale SSH，远程登录仍走 OpenSSH。重装或跨大版本升级系统时参考 [Tailscale Linux 官方文档](https://tailscale.com/docs/install/linux) 并同步更新 APT 源代号。

### 管理与连接

> 若当前 SSH 会话经由 Tailscale 建立，重启、断开、注销或升级 `tailscaled` 会直接中断连接；操作前须确保局域网 SSH、Cockpit 或物理控制台可用。

```bash
systemctl status tailscaled --no-pager
tailscale status && tailscale netcheck
sudo systemctl restart tailscaled
sudo tailscale down && sudo tailscale up   # 临时断开/重连（保留登录状态）
sudo tailscale logout                      # 破坏性：注销节点，重连需重新授权
sudo apt-get update && sudo apt-get install --only-upgrade tailscale
```

通过 Tailscale 连接 SSH（客户端须在同 Tailnet 且 ACL 允许 `22` 端口）：

```bash
ssh linus-stark@100.83.74.68
ssh linus-stark@linus-stark-nhx0db-de.taild30eeb.ts.net
```

## 十、XXL-JOB

### 部署与配置

- **版本与地址**：官方 `3.4.2`（`/opt/middleware/xxl-job/docker-compose.yml`，容器 `xxl-job-admin`），登录地址 `http://192.168.1.102:18084/auth/login`，账号 `admin` / `123456`。
- **数据库与日志**：复用 MySQL 容器（经 `mysql_default` 网络），库名 `xxl_job`（账号 `xxl_job_app`）；应用日志位于 `/var/log/apps/xxl-job`。
- **密钥文件**：`/opt/middleware/xxl-job/conf/application.properties`（权限 `600 root:root`），保存数据库密码及随机生成的 `xxl.job.accessToken`。
- **历史备份与告警说明**：原库迁移前备份于 `/data/backup/xxl-job/xxl_job-20261004-183445.sql.gz`。因无 SMTP 凭据，任务失败告警邮件地址已清空（原地址保存在 `/data/backup/xxl-job/alarm-email-20261004-183743.sql`），恢复邮件前须先配好 SMTP 并备份当前库。

### 本机开发接入执行器

仅调试定时任务时开启（平时保持 `enabled: false`），需确保开发机能连 `192.168.1.102:18084`，且服务器能回连开发机 `19999/TCP`：

1. 从服务器读取访问令牌填入环境变量 `XXL_JOB_ACCESS_TOKEN`（勿泄露到仓库或日志）：

```powershell
ssh myServer "sudo sed -n 's/^xxl.job.accessToken=//p' /opt/middleware/xxl-job/conf/application.properties"
```

2. 项目配置要点：`xxl-job.admin.addresses` 填根地址 `http://192.168.1.102:18084`（`3.4.1+` 已移除 `/xxl-job-admin` 后缀），`executor.ip` 填服务器可达的开发机局域网或 Tailscale IP，`executor.port` 为 `19999`。测试结束后改回 `enabled: false` 并重启本地应用。

验证登录页：`curl -sS -o /dev/null -w '%{http_code}\n' http://192.168.1.102:18084/auth/login`（应返回 `200`）。

## 十一、cpolar 内网穿透与已停用静态站点

### cpolar 内网穿透管理

- **运行状态**：`cpolar 3.3.12`，`cpolar.service` 以 `nobody` 用户运行，配置文件 `/usr/local/etc/cpolar/cpolar.yml`，日志 `/var/log/cpolar/access.log`，Web 仪表盘 `http://192.168.1.102:9200`。
- **权限关键约束**：`cpolar.yml` 含 authtoken，**属主必须是 `nobody:nogroup`（文件 `640`，目录 `/usr/local/etc/cpolar` `750`）**。若错设为 `root:nogroup`，`nobody` 无法写回登录信息，仪表盘会报 `permission denied` 登录失败。
- **隧道配置注意事项**：
  - 保留隧道配置中在线分配的 `id`，否则重启会新建隧道占用免费版 4 条额度并更换域名。
  - 修改 `cpolar.yml` 前先备份；暂时释放额度可将对应隧道 `start_type` 改为 `disable` 后重启。
  - 账号下若开有 `ssh → 22` 隧道，仅作临时调试，用完及时删除。

```bash
cpolar list
systemctl status cpolar --no-pager
systemctl restart cpolar
```

### 前端静态站点：高校智慧教务选课平台（已于 2026-10-25 停用）

- **当前状态**：APT 原生安装的 Nginx `1.28.3`（`nginx.service` 已 `stop` 并 `disable`）与后端 `smart-university` 容器均已停止，端口 `80`、`18083` 无监听，cpolar 隧道 `course-selection-web`（`https://571d6917.r36.cpolar.top` → `80`）与 `SmartUniversity`（`https://573564d7.r36.cpolar.top` → `18083`）仍保留配置但回源返回 `404`。
- **保留资产**：静态文件 `/var/www/course-selection-web`，Nginx 配置 `/etc/nginx/sites-available/course-selection-web.conf`（已软链至 `sites-enabled`，`/api/` 反代 `http://127.0.0.1:18083`，SPA 回退 `try_files $uri $uri/ /index.html`），日志 `/var/log/nginx/course-selection-web.*.log`。
- **恢复方式**：

```bash
systemctl enable --now nginx
cd /opt/apps/smart-university && docker compose start
```

## 十二、MinIO

### 部署信息

| 项目 | 当前配置 |
| --- | --- |
| Compose 目录 | `/opt/middleware/minio` |
| 容器名 / 镜像 | `minio` / `cgr.dev/chainguard/minio@sha256:87896cb33f93ecfcca4e883c3fb2f86a0a1a600c003a104011ed30631c1869c3` |
| S3 API / Console | `http://192.168.1.102:18090` / `http://192.168.1.102:18091`（仅绑定局域网 IP） |
| 数据目录 | `/data/minio/data`（属主 `65532:65532`） |
| 凭据文件 | `/opt/middleware/minio/minio.env`（`600 root:root`），账号 `admin` / `12345678` |
| 健康检查 | `mc ready local`（依赖镜像内置 `/usr/bin/mc`） |

> 因上游官方 `minio/minio` 归档及 AIStor 需 license，现网采用 Chainguard 镜像。升级须拉取新 `latest` 后在 Compose 中替换 `digest`；新建数据目录须执行 `chown -R 65532:65532 /data/minio/data`，否则启动报 `file access denied`。

### 常用操作、客户端与备份

```bash
# 健康检查
curl -sS -o /dev/null -w 's3=%{http_code} console=%{http_code}\n' \
  http://192.168.1.102:18090/minio/health/live http://192.168.1.102:18091/

# 使用容器内 mc 管理（宿主机文件须通过 stdin 管道写入）
docker exec -e MC_HOST_local=http://admin:12345678@127.0.0.1:9000 minio mc ls local
printf 'hello\n' | docker exec -i -e MC_HOST_local=http://admin:12345678@127.0.0.1:9000 \
  minio mc pipe local/some-bucket/hello.txt

# 规范备份（禁止直接 cp 数据目录，尚未配置自动定时备份）
docker exec -e MC_HOST_local=http://admin:12345678@127.0.0.1:9000 \
  -e MC_HOST_backup=http://<备份目标> minio mc mirror --overwrite local backup
```

- **客户端要点**（Java AWS SDK v2 等）：Endpoint 填 `http://192.168.1.102:18090`，Region 填 `us-east-1`，**必须开启 `.forcePathStyle(true)`**（否则报 `NoSuchBucket` 或域名解析失败）。
- **改密**：修改 `/opt/middleware/minio/minio.env` 后 `docker compose up -d` 重建，并同步更新调用方。
- **常见排查**：`file access denied` 查 `65532:65532` 属主；`403 SignatureDoesNotMatch` 查密钥或系统时钟偏差。

## 十三、PostgreSQL

### 部署信息

| 项目 | 当前配置 |
| --- | --- |
| Compose 目录 | `/opt/middleware/postgres`（`docker-compose.yml`、`postgres.env` `600 root:root`） |
| 容器名 / 镜像 | `postgres` / `postgres:16.15-alpine-pgvector-0.8.7`（PostgreSQL 16.15 + pgvector 0.8.7） |
| 端口 | `0.0.0.0:5432`（IPv4 + IPv6 全网卡监听） |
| 数据目录 | `/data/postgres/data` → `/var/lib/postgresql/data`（属主 `70:70`） |
| 超级用户 / 加密 | `root`（无默认 `postgres` 角色）/ `scram-sha-256`、`data_checksums=on`、`TimeZone=Asia/Shanghai` |
| 业务库 | `toonflow`（owner `root`）、`ragent` 与 `ragent_bit`（owner `ragent`）、`ragent_lightrag`（独立同名角色） |
| 历史备份 | `/data/backup/postgres/toonflow-pg18-20261030-113545.dump` 及 `.sql`（日常自动备份未配置） |

### 镜像挂载与版本规则

1. **挂载点必须对齐镜像 `VOLUME`**（PG16 为 `/var/lib/postgresql/data`，PG18 起为 `/var/lib/postgresql`），换镜像先用 `docker image inspect <image> --format '{{json .Config.Volumes}}'` 核对，避免数据落入匿名卷。
2. **运行 UID 差异**：`alpine` 变体为 `70:70`，`debian` 变体为 `999:999`。
3. **跨大版本降级**：不能直接复用数据目录；若从 PG17+ `pg_dump` 导回 PG16，须使用纯 SQL（`-Fp`）并删除文件头 PG17+ 独有的 `SET transaction_timeout = 0;`。

### 认证、改密与常用连接

`pg_hba.conf` 仅对容器内 `local` 与回环启用 `trust`（即宿主机 `docker` 组用户免密，加人入组需谨慎），所有外部与局域网连接强制 `scram-sha-256`。

```bash
# 容器内免密进入与查看库列表
docker exec -it postgres psql -U root -d toonflow
docker exec -it postgres psql -U root -d toonflow -c '\l'

# 外部连接
PGPASSWORD='123456' psql -h 192.168.1.102 -p 5432 -U root -d toonflow

# 改密（两步须同步）
docker exec -it postgres psql -U root -d toonflow -c "alter role root with password '新密码';"
# 同步修改 /opt/middleware/postgres/postgres.env 中的 POSTGRES_PASSWORD
```

- **JDBC**：`jdbc:postgresql://192.168.1.102:5432/toonflow?stringtype=unspecified`
- **Python (SQLAlchemy)**：`postgresql+psycopg://root:123456@192.168.1.102:5432/toonflow`（密码含 `@`、`#`、`/` 时 URL 须转义）
- **备份提示**：禁止直接 `cp` 运行中的数据目录，规范使用 `pg_dump -Fc`。
- **内存调优提示**：当前 `shared_buffers` 默认 `128MB`、`max_connections=100`，本机总内存 `16 GB`，调大参数前先确认其他容器实际剩余可用内存。

### 排查

| 现象 | 优先检查 |
| --- | --- |
| `data directory has wrong ownership` | `/data/postgres/data` 属主是否为 `70:70` |
| `database files are incompatible with server` | 数据目录跨了大版本，须逻辑导出重建 |
| `FATAL: role "postgres" does not exist` | 实例超级用户是 `root`，没有 `postgres` 角色 |
| 应用连不上但 `psql` 正常 | 检查 `max_connections=100` 上限及目标库名是否正确 |
| `docker volume ls` 出现匿名卷 | 挂载路径与镜像 `VOLUME` 声明不一致 |

## 十四、Ollama（本地大模型推理）

### 部署信息

采用**官方二进制 + systemd 原生安装**（未用 Docker，也**不用官方 `install.sh`**——因机器可能因内核/驱动错配触发重装 `cuda-drivers` 覆盖现有 `nvidia-driver-595-open` 与 GNOME 桌面）：

| 项目 | 当前配置 |
| --- | --- |
| 版本与路径 | `0.40.16`；`/usr/local/bin/ollama` → `/usr/local/lib/ollama/bin/ollama` |
| 数据目录 | `/data/ollama/models`（`ollama:ollama` 750） |
| 服务单元与日志 | `/etc/systemd/system/ollama.service`（`enabled` + `active`）/ `journalctl -u ollama` |
| 监听与环境变量 | `OLLAMA_HOST=0.0.0.0:11434`（客户端 `/etc/profile.d/ollama.sh` 设为 `127.0.0.1:11434`） |
| 并发与常驻限制 | `OLLAMA_KEEP_ALIVE=30m`、`OLLAMA_MAX_LOADED_MODELS=1`、`OLLAMA_NUM_PARALLEL=1` |
| GPU 硬件 | NVIDIA GTX 1650 Ti（3.6 GiB 可用显存，100% GPU offload） |

> **权限特殊要求**：`/data` 保持 `700 root:root`，须通过 ACL 单独授予穿越权限：`setfacl -m u:ollama:x /data`（`getfacl -p /data` 应有 `user:ollama:--x`）。切勿将 `/data` 改为 `755`。

### 升级与网络代理

`registry.ollama.ai` 拉取模型可直连，但下载二进制安装包经 GitHub Release 需走本机 `mihomo` 代理 `127.0.0.1:7897`（新版仅发 `.tar.zst`）：

```bash
export HTTPS_PROXY=http://127.0.0.1:7897
curl -fL -o /tmp/ollama-linux-amd64.tar.zst https://ollama.com/download/ollama-linux-amd64.tar.zst
sudo zstd -dc /tmp/ollama-linux-amd64.tar.zst | sudo tar -xf - -C /usr/local
sudo systemctl restart ollama
```

### 已装模型与硬件限制

| 模型 | 磁盘 | 上下文 | 能力 | 显存驻留 |
| --- | --- | --- | --- | --- |
| `lfm3:1.7b` | 1.1G | 256K（默认开 `4096`） | completion、tools、thinking | 1.2G |
| `lfm3-omni:2b` | 2.0G | 128K（默认开 `4096`） | completion、**vision**、**audio**、**speech**、tools | 1.4–1.6G |
| `qwen2.5:1.5b` | 986M | 32K | completion、tools | 1.2G |

- 3.6 GiB 显存是硬上限（7B Q4 约 4.7G 会部分退化到 CPU），且整机总内存 `16 GB` 常驻多套服务，**建议只跑 `<= 3B` 模型，勿同时加载多个模型**。
- `lfm3:1.7b` 默认开启 thinking，不需要时请求传 `"think": false`；`lfm3-omni:2b` 不支持 thinking，且流式音频回答（`output_modalities:["audio"]`）尚未合入 Ollama，须用下述 STT/TTS 端点。

### 常用命令与 API 端点

```bash
systemctl status ollama && systemctl restart ollama
journalctl -u ollama -f
ollama list
ollama ps                        # 检查是否 100% GPU
ollama stop <model>              # 手动卸载显存
```

```bash
# 对话（OpenAI 兼容，原生端点为 /api/generate）
curl http://192.168.1.102:11434/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"model":"qwen2.5:1.5b","messages":[{"role":"user","content":"hi"}]}'

# STT（lfm3-omni:2b）
curl -X POST http://192.168.1.102:11434/v1/audio/transcriptions \
  -F "file=@/tmp/audio.wav" -F "model=lfm3-omni:2b"

# TTS（lfm3-omni:2b，返回 WAV 头长度为流式 0xFFFFFFFF，严格校验可用 ffmpeg -c copy 重写）
curl -X POST http://192.168.1.102:11434/v1/audio/speech \
  -H 'Content-Type: application/json' \
  -d '{"model":"lfm3-omni:2b","input":"你好","response_format":"wav"}' -o out.wav
```

### 排查

| 现象 | 优先检查 |
| --- | --- |
| `mkdir /data/ollama: permission denied` | 检查 `/data` 是否存在 `user:ollama:--x` ACL |
| `nvidia-smi` 报 `Driver/library version mismatch` | 内核态/用户态驱动版本错配（重启系统对齐前常见）；CUDA 推理通常不受影响，用 `ollama ps` 确认是否 `100% GPU` |
| `ollama ps` 显示 `100% CPU` | 查 `journalctl -u ollama \| grep -i cuda` 与 `/dev/nvidia0` |
| `ollama list` 报 `could not connect to ollama server` | 先看 `systemctl is-active ollama` 与 `ss -lntp \| grep 11434`；非登录 shell/cron 须显式设置 `OLLAMA_HOST=127.0.0.1:11434` |
| `ollama pull` 卡住 | 给 systemd 单元临时配置 `HTTPS_PROXY=http://127.0.0.1:7897` |

## 十五、RustFS

### 部署信息

与既有 MinIO 并存（均使用 S3 协议，端口与凭据独立，勿在两套上混用同桶业务数据；**禁止使用 `alpha` 版本**，现网已固定正式版 `1.0.6`）：

| 项目 | 当前配置 |
| --- | --- |
| Compose 目录 | `/opt/middleware/rustfs`（`docker-compose.yml`、`rustfs.env` `600 root:root`） |
| 容器名 / 镜像 | `rustfs` / `rustfs/rustfs:1.0.6`（`sha256:ee717072290bec9b11abfc0365f8310b00df1cd9b3639943edae47bdef05044a`） |
| 运行身份 | uid/gid `10001:10001`（非 root） |
| S3 API / Console | `http://192.168.1.102:9000` / `http://192.168.1.102:9001/rustfs/console/`（仅绑定局域网 IP） |
| 数据与日志目录 | `/data/rustfs/data` → `/data`、`/data/rustfs/logs` → `/logs`（属主 `10001:10001`，权限 `750`） |
| 账号 | `rustfsadmin` / `rustfsadmin` |

### 使用、验证与排查

- **客户端与备份**：与 MinIO 一致，AWS SDK 须开启 `.forcePathStyle(true)`，Endpoint 为 `http://192.168.1.102:9000`；数据备份须用 `mc mirror`（勿直接 `cp` 目录，尚未配自动备份）。
- **Tailscale 注意**：端口仅绑定 `192.168.1.102`，Tailscale 网段（`100.83.74.68`）默认无法访问。

```bash
# 健康检查
curl -sS -o /dev/null -w 'health=%{http_code}\n' http://192.168.1.102:9000/health
curl -sS -o /dev/null -w 'console=%{http_code}\n' http://192.168.1.102:9001/rustfs/console/index.html

# 借用 minio 容器的 mc 验证
docker exec -e MC_HOST_rustfs=http://rustfsadmin:rustfsadmin@192.168.1.102:9000 minio mc ls rustfs
```

| 现象 | 优先检查 |
| --- | --- |
| 启动退出报 `Permission denied` | `/data/rustfs/data` 与 `/data/rustfs/logs` 属主是否为 `10001:10001` |
| 控制台根路径 `/` 返回 `403` | 正常现象，必须访问 `/rustfs/console/` |
| 升级后控制台页面异常 | 强刷浏览器（`Ctrl+Shift+R`）并清理站点 `localStorage` |

## 十六、Elasticsearch（2026-11-18）

| 项目 | 部署配置 |
| --- | --- |
| 版本与容器 | Elasticsearch 8.17.0 + IK 8.17.0，容器 `elasticsearch` |
| 镜像 | `ragent/elasticsearch-ik:8.17.0`，IK 随镜像构建 |
| 端口 | `127.0.0.1:19200 → 9200`。宿主机 `9200` 由 cpolar 使用 |
| 配置目录 | `/opt/middleware/elasticsearch`，含 `docker-compose.yml`、`Dockerfile` 和 `conf/analysis-ik-8.17.0.zip` |
| 数据目录 | `/data/elasticsearch/data` |
| 资源 | JVM 堆 512 MiB，容器上限 1.25 GiB、2 核 |
| 验收 | 集群状态 `green`，`ik_max_word`、`ik_smart` 均已验证 |

十六～十八节三套服务仅在 Ubuntu 回环地址监听。Windows 使用现有的 SSH 别名转发端口，保持命令窗口打开即可访问：

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

## 十七、LightRAG + Neo4j（2026-11-18）

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

Ragent 通过转发访问 `rag.graph.lightrag.base-url=http://127.0.0.1:9621`。启用时将 `rag.graph.type` 设为 `lightrag`，并将 `rag.search.channels.graph.enabled` 设为 `true`。数据库备份包含 `ragent_lightrag` 和 Neo4j 图数据两个部分。

## 十八、LangFuse（2026-11-18）

| 项目 | 部署配置 |
| --- | --- |
| 版本 | LangFuse 4.82.0，Web 和 Worker 镜像按 digest 固定 |
| 容器 | `langfuse-web`、`langfuse-worker`、`langfuse-postgres`、`langfuse-redis`、`langfuse-clickhouse`、`langfuse-minio` |
| Web 与 OTLP | `http://127.0.0.1:13000`、`http://127.0.0.1:13000/api/public/otel/v1/traces`。宿主机 `3000` 由 ToonFlow 使用 |
| 配置目录 | `/opt/middleware/langfuse`，含 Compose、ClickHouse 配置、`.env`（权限 600） |
| 数据目录 | `/data/langfuse/{postgres,redis,minio}` 和 `/data/langfuse/clickhouse/{data,logs}` |
| 登录 | `admin@ragent.dev`，登录密码见 `.env` 的 `LF_ADMIN_PASSWORD` |
| 追踪密钥 | `.env` 的 `LF_PUBLIC_KEY` 和 `LF_SECRET_KEY`，项目名 `ragent-service` |

Redis、PostgreSQL、ClickHouse 和 MinIO 在专用 Docker 网络中运行。Redis 使用 `noeviction`，ClickHouse 容器内存上限 3 GiB。

```bash
cd /opt/middleware/langfuse
docker compose ps
curl -fsS http://127.0.0.1:13000/api/public/health
```

Ragent 启用追踪时将 `agent.trace.enabled` 设为 `true`，把 `agent.trace.endpoint` 指向上述 OTLP URL，并从本机配置传入对应的项目公钥与密钥。

以上三套服务各自在对应目录执行 `docker compose stop` 停用，执行 `docker compose up -d --pull never --wait` 恢复。持久化数据位于表中列出的 `/data` 目录。
