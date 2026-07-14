# 多实例

同时运行多个 nanobot 实例，每个实例拥有独立的配置和运行时数据。以 `--config` 作为主入口。当你希望初始化或更新某个实例保存的工作区时，可选地在 `onboard` 时传入 `--workspace`。

## 快速开始

如果你希望每个实例从一开始就拥有自己专属的工作区，请在初始化时同时传入 `--config` 和 `--workspace`。

**初始化实例：**

```bash
# 创建各自独立的实例配置和工作区
nanobot onboard --config ~/.nanobot-telegram/config.json --workspace ~/.nanobot-telegram/workspace
nanobot onboard --config ~/.nanobot-discord/config.json --workspace ~/.nanobot-discord/workspace
nanobot onboard --config ~/.nanobot-feishu/config.json --workspace ~/.nanobot-feishu/workspace
```

**配置每个实例：**

编辑 `~/.nanobot-telegram/config.json`、`~/.nanobot-discord/config.json` 等文件，为它们配置不同的频道设置。你在 `onboard` 时传入的工作区会作为该实例的默认工作区被保存到各自的配置中。

**运行实例：**

```bash
# 启动前检查某个实例
nanobot status --config ~/.nanobot-telegram/config.json

# 实例 A —— Telegram 机器人
nanobot gateway --config ~/.nanobot-telegram/config.json

# 实例 B —— Discord 机器人
nanobot gateway --config ~/.nanobot-discord/config.json

# 实例 C —— 使用自定义端口的飞书机器人
nanobot gateway --config ~/.nanobot-feishu/config.json --port 18792
```

## 路径解析

使用 `--config` 时，nanobot 会根据配置文件位置派生出运行时数据目录。除非你使用 `--workspace` 覆盖，否则工作区仍然来自 `agents.defaults.workspace`。

要在本地针对其中某个实例打开一个 CLI 会话：

```bash
nanobot agent -c ~/.nanobot-telegram/config.json -m "Hello from Telegram instance"
nanobot agent -c ~/.nanobot-discord/config.json -m "Hello from Discord instance"

# 为特定实例打开浏览器工作台
nanobot webui -c ~/.nanobot-telegram/config.json

# 可选的一次性工作区覆盖
nanobot agent -c ~/.nanobot-telegram/config.json -w /tmp/nanobot-telegram-test
```

> `nanobot agent` 会使用所选的工作区/配置启动一个本地 CLI 智能体。它不会挂接或代理到已经运行中的 `nanobot gateway` 进程。

| 组件 | 解析来源 | 示例 |
|-----------|---------------|---------|
| **配置** | `--config` 路径 | `~/.nanobot-A/config.json` |
| **工作区** | `--workspace` 或配置文件 | `~/.nanobot-A/workspace/` |
| **Cron 任务** | 工作区目录 | `~/.nanobot-A/workspace/cron/` |
| **媒体 / 运行时状态** | 配置目录 | `~/.nanobot-A/media/` |

## 工作原理

- `--config` 决定加载哪个配置文件
- 默认情况下，工作区来自该配置中的 `agents.defaults.workspace`
- 如果传入 `--workspace`，它会覆盖配置文件中的工作区

## 最小配置

1. 将基础配置复制到一个新的实例目录。
2. 为该实例设置不同的 `agents.defaults.workspace`。
3. 使用 `--config` 启动实例。

配置片段示例：

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.nanobot-telegram/workspace"
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_TELEGRAM_BOT_TOKEN"
    }
  },
  "gateway": {
    "host": "127.0.0.1",
    "port": 18790
  }
}
```

复制的基础配置可以继续使用相同的 `modelPresets` 和 `agents.defaults.modelPreset`。如果该实例需要不同的模型，添加另一个预设并将 `agents.defaults.modelPreset` 设为该预设名称。

启动各个实例：

```bash
nanobot status --config ~/.nanobot-telegram/config.json
nanobot gateway --config ~/.nanobot-telegram/config.json
nanobot gateway --config ~/.nanobot-discord/config.json
```

每个网关实例还会在 `gateway.host:gateway.port` 上暴露一个轻量的 HTTP 健康检查端点。默认情况下，网关绑定 `127.0.0.1`，因此除非你显式将 `gateway.host` 设置为公网或局域网可达地址，否则该端点仅本地可访问。

- `GET /health` 返回 `{"status":"ok"}`
- 其他路径返回 `404`

在需要时为一次性运行覆盖工作区：

```bash
nanobot gateway --config ~/.nanobot-telegram/config.json --workspace /tmp/nanobot-telegram-test
```

## 常见用例

- 为 Telegram、Discord、飞书以及其他平台运行独立的机器人
- 保持测试和生产实例相互隔离
- 为不同团队使用不同的模型或提供商
- 使用独立的配置和运行时数据为多个租户提供服务

## 注意事项

- 同时运行的实例必须使用不同的端口
- 如果希望记忆、会话和技能相互隔离，请为每个实例使用不同的工作区
- `--workspace` 会覆盖配置文件中定义的工作区
- Cron 任务存储在当前工作区中；运行时的媒体/状态则派生自配置目录
