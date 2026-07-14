# 部署

在本地 `nanobot agent -m "Hello!"` 正常工作后再阅读本页。部署用于让长时间运行的组件保持在线：WebUI、聊天应用、心跳、Dream、cron 任务以及频道连接。

## 部署之前

在使用 Docker、systemd 或 LaunchAgent 之前，先确认以下几项：

| 检查项 | 重要原因 |
|---|---|
| `nanobot status` 显示了预期的配置和工作区 | 确认进程会读取你想运行的实例 |
| `nanobot agent -m "Hello!"` 可以正常工作 | 在增加服务层之前，验证安装、配置、提供商、模型以及工作区写入均正常 |
| 密钥保存在环境变量或受保护的配置文件中 | API 密钥、机器人 token、OAuth 状态和聊天凭据不应被全局可读 |
| `~/.nanobot/` 或你自定义的配置/工作区路径是持久化的 | 会话、记忆、频道登录状态、生成的产物以及 cron 任务都存储在这里 |
| 频道访问控制符合预期 | 在对外暴露机器人之前，使用 `allowFrom`、配对、WebSocket 的 `token`/`tokenIssueSecret`，或私有测试频道 |
| 端口已经规划好 | 网关健康检查默认仅本地 `127.0.0.1:18790`；WebUI/WebSocket 默认端口为 `8765`；`nanobot serve` 默认端口为 `8900` |
| 日志容易获取 | 排查启动问题时可使用 `docker compose logs`、`journalctl`、LaunchAgent 日志文件或 `nanobot gateway --verbose` |

修改 `config.json` 后需要重启已部署的进程。长时间运行的进程只在启动时读取一次配置。

## 选择运行时

| 运行时 | 适用场景 | 状态位置 | 有用的首个命令 |
|---|---|---|---|
| Docker Compose | 在 Linux 服务器或工作站上进行可复用的容器化运行 | 将 `~/.nanobot` 绑定挂载到 `/home/nanobot/.nanobot` | `docker compose run --rm nanobot-cli agent -m "Hello!"` |
| Docker CLI | 手动的容器测试或小型一次性主机 | 将 `~/.nanobot` 绑定挂载到 `/home/nanobot/.nanobot` | `docker run -v ~/.nanobot:/home/nanobot/.nanobot --rm nanobot status` |
| systemd 用户服务 | 自动重启的 Linux 用户级网关 | 除非显式传入路径，否则使用主机用户的 `~/.nanobot` | `systemctl --user status nanobot-gateway` |
| macOS LaunchAgent | 在登录后自动启动的 macOS 网关 | 除非 plist 显式传入路径，否则使用主机用户的 `~/.nanobot` | `launchctl list | grep ai.nanobot.gateway` |

## Docker

> [!TIP]
> `-v ~/.nanobot:/home/nanobot/.nanobot` 参数会将本机配置目录挂载进容器，使配置和工作区在容器重启后仍然保留。
> 容器以非 root 用户 `nanobot`（UID 1000）运行，并从 `/home/nanobot/.nanobot` 读取配置。请始终将主机的配置目录挂载到 `/home/nanobot/.nanobot`，而不是 `/root/.nanobot`。
> 如果遇到 **Permission denied**，先在主机上修复所有权：`sudo chown -R 1000:1000 ~/.nanobot`，或传入 `--user $(id -u):$(id -g)` 与主机 UID 保持一致。Podman 用户可以使用 `--userns=keep-id`。
>
> [!IMPORTANT]
> 官方 Docker 用法目前指的是使用本仓库中提供的 `Dockerfile` 进行构建。第三方命名空间下的 Docker Hub 镜像并非由 HKUDS/nanobot 维护或验证；除非你信任发布者，否则不要向其中挂载 API 密钥或机器人 token。

> [!IMPORTANT]
> 网关和 WebSocket 频道在 `config.json` 中默认使用 `host: "127.0.0.1"`（在 `nanobot/config/schema.py` 中设置）。Docker 的 `-p` 端口转发无法访问容器的 loopback 接口，因此如果需要让主机或局域网访问暴露的端口，必须在启动容器前在 `~/.nanobot/config.json` 中将两个绑定地址都设置为 `0.0.0.0`。要从 Docker 提供内置 WebUI，需要将 WebSocket 频道对外绑定，并使用密钥保护引导过程：
>
> ```json
> {
>   "gateway": { "host": "0.0.0.0" },
>   "channels": {
>     "websocket": {
>       "host": "0.0.0.0",
>       "port": 8765,
>       "tokenIssueSecret": "your-secret-here"
>     }
>   }
> }
> ```
>
> 当 WebSocket `host` 为 `0.0.0.0` 时，除非同时配置了 `token` 或 `tokenIssueSecret`，否则频道将拒绝启动。详情见 [`webui.md#lan-access`](./webui.md#lan-access)。
> 网关的健康检查路由本身有意保持极简且未鉴权。当容器将其绑定到 `0.0.0.0` 时，仅将端口 `18790` 发布到主机 loopback；任何远程监控的健康检查端点都应放在防火墙或反向代理之后。如果确实需要另一台主机直接探测，请将端口映射中的 `127.0.0.1` 替换为可信主机接口，并将入站流量限制在监控系统范围内。

### Docker Compose

```bash
docker compose run --rm nanobot-cli onboard   # 首次安装
vim ~/.nanobot/config.json                     # 添加 API 密钥
docker compose up -d nanobot-gateway           # 启动网关
```

```bash
docker compose run --rm nanobot-cli agent -m "Hello!"   # 运行 CLI
docker compose logs -f nanobot-gateway                   # 查看日志
docker compose down                                      # 停止
```

### Docker

```bash
# 构建镜像
docker build -t nanobot .

# 初始化配置（仅首次）
docker run -v ~/.nanobot:/home/nanobot/.nanobot --rm nanobot onboard

# 在主机上编辑配置以添加 API 密钥
vim ~/.nanobot/config.json

# 运行网关（连接已启用的频道，例如 Telegram/Discord/Mochat）。
# 参数与 docker-compose.yml 中声明的安全能力和端口映射保持一致：
#   - 启用 `tools.exec.sandbox: "bwrap"` 时需要 `--cap-drop ALL --cap-add SYS_ADMIN`
#     以及 unconfined apparmor/seccomp（bwrap 需要 CAP_SYS_ADMIN 才能使用用户命名空间）。
#     没有它们，`bwrap` 会以 `clone3: Operation not permitted` 退出。
#   - `-p 8765:8765` 暴露 WebSocket 频道 / WebUI，与网关健康检查端点 18790 并列。
docker run \
  --cap-drop ALL --cap-add SYS_ADMIN \
  --security-opt apparmor=unconfined \
  --security-opt seccomp=unconfined \
  -v ~/.nanobot:/home/nanobot/.nanobot \
  -p 127.0.0.1:18790:18790 -p 8765:8765 \
  nanobot gateway

# 或执行单条命令
docker run -v ~/.nanobot:/home/nanobot/.nanobot --rm nanobot agent -m "Hello!"
docker run -v ~/.nanobot:/home/nanobot/.nanobot --rm nanobot status
```

## Linux 服务

将网关作为 systemd 用户服务运行，以便自动启动并在失败时自动重启。

先预览生成的 unit 文件：

```bash
nanobot gateway install-service --manager systemd --dry-run
```

安装、启用并启动它：

```bash
nanobot gateway install-service --manager systemd
```

对于自定义实例，传入你运行网关时使用的相同 config/workspace 选择器：

```bash
nanobot gateway install-service \
  --manager systemd \
  --name nanobot-telegram \
  --config ~/.nanobot-telegram/config.json \
  --workspace ~/.nanobot-telegram/workspace
```

常见操作：

```bash
systemctl --user status nanobot-gateway        # 查看状态
systemctl --user restart nanobot-gateway       # 修改配置后重启
journalctl --user -u nanobot-gateway -f        # 追踪日志
nanobot gateway uninstall-service --manager systemd
```

安装器会写入 `~/.config/systemd/user/nanobot-gateway.service`，执行
`systemctl --user daemon-reload`，启用该 unit 并重启它。它使用当前的 Python 可执行文件通过 `python -m nanobot gateway --foreground` 运行，因此服务与你安装 nanobot 时的环境一致。

> **注意：** 用户服务只在你登录期间运行。若要在注销后仍保持网关运行，需启用 lingering：
>
> ```bash
> loginctl enable-linger $USER
> ```

## macOS LaunchAgent

如果你希望 `nanobot gateway` 在你登录后保持在线，且无需一直保留终端窗口，请使用 LaunchAgent。

先预览生成的 plist：

```bash
nanobot gateway install-service --manager launchd --dry-run
```

安装、加载、启用并启动它：

```bash
nanobot gateway install-service --manager launchd
```

对于自定义实例：

```bash
nanobot gateway install-service \
  --manager launchd \
  --name nanobot-telegram \
  --config ~/.nanobot-telegram/config.json \
  --workspace ~/.nanobot-telegram/workspace
```

常见操作：

```bash
launchctl list | grep ai.nanobot.gateway
launchctl kickstart -k gui/$(id -u)/ai.nanobot.gateway
nanobot gateway uninstall-service --manager launchd
```

安装器会写入 `~/Library/LaunchAgents/ai.nanobot.gateway.plist`，使用当前的 Python 可执行文件通过 `python -m nanobot gateway --foreground` 运行，并将 LaunchAgent 日志写入 `~/.nanobot/logs/`。

> **注意：** 如果启动时出现 "address already in use"，请先停止手动启动的 `nanobot gateway` 进程。
