# 自托管 AI 智能体的聊天应用

将 nanobot 连接到 Telegram、Discord、Slack、微信、Email、Mattermost 以及
其他聊天平台。本页面是完整的聊天频道参考。如果你想
获取针对某个平台的聚焦配置路径，请从对应指南开始：

| 平台 | 指南 |
|---|---|
| Telegram | [使用 nanobot 构建 Telegram AI 智能体](./guides/telegram-ai-agent.md) |
| Discord | [使用 nanobot 构建 Discord AI 智能体](./guides/discord-ai-agent.md) |
| Slack | [使用 nanobot 构建 Slack AI 智能体](./guides/slack-ai-agent.md) |
| Feishu | [使用 nanobot 构建飞书 AI 智能体](./guides/feishu-ai-agent.md) |
| WhatsApp | [使用 nanobot 构建 WhatsApp AI 智能体](./guides/whatsapp-ai-agent.md) |
| WeChat | [使用 nanobot 构建微信 AI 智能体](./guides/wechat-ai-agent.md) |
| QQ | [使用 nanobot 构建 QQ AI 智能体](./guides/qq-ai-agent.md) |
| Email | [使用 nanobot 构建 Email AI 智能体](./guides/email-ai-agent.md) |
| Mattermost | [使用 nanobot 构建 Mattermost AI 智能体](./guides/mattermost-ai-agent.md) |

想要构建你自己的频道？请查看[频道插件指南](./channel-plugin-guide.md)。

在配置聊天应用之前，请确保本地 CLI 路径可用：

```bash
nanobot agent -m "Hello!"
```

如果失败，请先通过 [`quick-start.md`](./quick-start.md)、[`providers.md`](./providers.md) 和 [`troubleshooting.md`](./troubleshooting.md) 修复安装、配置、提供商或模型配置。聊天应用要求在配置频道后保持 `nanobot gateway` 持续运行。

下面的大多数示例都是要合并到 `~/.nanobot/config.json` 中的片段。当
片段包含 `allowFrom` 时，它展示的是静态允许列表。对于
支持配对的频道要使用基于配对的访问，请省略 `allowFrom`；Slack 和
Mattermost 还需要将 `dm.policy` 设为 `"allowlist"`，DM 才会签发配对
码。

> [!NOTE]
> 如果你正在从默认安装了聊天应用 SDK 的版本升级，
> 请在同一 Python 环境中安装该频道的可选依赖后再启用或
> 重启该频道：
>
> ```bash
> nanobot plugins enable <channel>
> ```
>
> 将 `<channel>` 替换为诸如 `telegram`、`slack`、`feishu`、
> `dingtalk`、`matrix`、`qq`、`napcat`、`weixin`、`wecom` 或 `msteams` 的名称。
> 若要稍后关闭某个频道，运行 `nanobot plugins disable <channel>`。
> nanobot 会保留已保存的设置，但下次重启后将停止加载该频道。

## 通用配置模式

每个聊天应用都使用相同的形式：

1. 在聊天平台上创建或准备机器人/账户。
2. 复制平台提供给你的令牌、密钥、二维码登录状态、Webhook URL 或账号 ID。
3. 将该平台的 JSON 片段合并到 `~/.nanobot/config.json` 中。
4. 对于支持 DM 的频道，优先使用配对：省略 `allowFrom`，让第一条 DM 接收配对码，然后使用 `/pairing approve <code>` 批准。
5. 对于不支持配对的频道（例如 Email），请使用 `allowFrom` 或平台特定的允许列表将访问范围限制得更窄。
6. 检查 nanobot 是否能看到已配置的频道：

```bash
nanobot channels status
```

7. 启动网关，保持该终端运行：

```bash
nanobot gateway
```

8. 发送一条测试 DM。如果机器人返回配对码，请批准它并再次发送消息。在群聊中，请遵循该频道 `groupPolicy` 的行为：许多频道默认仅在被 @ 时响应，而 Matrix 和 WhatsApp 默认开放式回复群聊。

如果 `nanobot channels status` 没有把该频道显示为启用，说明配置片段放错了位置、频道名称拼错，或你编辑的配置文件不是 nanobot 正在读取的那个。如果频道已启用但消息未到达，请运行 `nanobot gateway --verbose`，并对比平台侧的凭据、事件权限和允许列表。

> `allowFrom: ["*"]` 会绕过配对，允许任何能接触到该频道的人与机器人对话。仅在你有意为之或在私有沙箱中临时测试时使用。

| 频道 | 你需要的凭据 |
|---------|---------------|
| **Telegram** | 来自 @BotFather 的 Bot token |
| **Discord** | Bot token + Message Content intent |
| **WhatsApp** | 扫描二维码（`nanobot channels login whatsapp`） |
| **WeChat (Weixin)** | 扫描二维码（`nanobot channels login weixin`） |
| **Feishu** | 扫描二维码（`nanobot channels login feishu`）或 App ID + App Secret |
| **DingTalk** | App Key + App Secret |
| **Slack** | Bot token + App-Level token |
| **Matrix** | Homeserver URL + Access token |
| **Email** | IMAP/SMTP 凭据 |
| **QQ** | App ID + App Secret |
| **Napcat (QQ)** | Napcat 正向 WebSocket URL + 访问令牌 |
| **Wecom** | Bot ID + Bot Secret |
| **Microsoft Teams** | App ID + App Password + 公网 HTTPS 端点 |
| **Mochat** | Claw token（支持自动配置） |
| **Signal** | signal-cli 守护进程 + 电话号码 |

<details>
<summary><b>Telegram</b></summary>

**安装可选的频道依赖**

```bash
nanobot plugins enable telegram
```

**1. 创建一个机器人**
- 打开 Telegram，搜索 `@BotFather`
- 发送 `/newbot`，按提示操作
- 复制令牌

**2. 配置**

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": ["YOUR_USER_ID"]
    }
  }
}
```

> 你可以在 Telegram 设置中找到你的 **User ID**。它显示为 `@yourUserId`。复制此值时**不要包含 `@` 符号**，然后粘贴到配置文件中。
>
> `richMessages` 默认为 `false`。仅当你的 Telegram 客户端支持 Bot API 10.1 富消息且你想要更丰富的 Markdown 渲染时才将其设为 `true`；对于 Telegram Web 请保持禁用，否则可能会显示不支持的消息错误。


**3. 运行**

```bash
nanobot gateway
```

**Webhook 模式（可选）**

Telegram 默认使用长轮询。若要通过 Webhook 接收更新，请暴露一个公网 HTTPS URL，把请求转发到 nanobot 的本地监听端口，并将 `mode` 设为 `webhook`：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "mode": "webhook",
      "webhookUrl": "https://example.com/telegram",
      "webhookListenHost": "127.0.0.1",
      "webhookListenPort": 8081,
      "webhookPath": "/telegram",
      "webhookSecretToken": "CHANGE_ME_RANDOM_SECRET",
      "webhookMaxConnections": 4,
      "allowFrom": ["YOUR_USER_ID"]
    }
  }
}
```

> Webhook 模式下必须提供 `webhookSecretToken`。不要在没有反向代理或隧道前置的情况下，将本地 Webhook 监听器直接暴露到公网。TLS/Host 策略由你的代理处理；nanobot 只在 `webhookListenHost:webhookListenPort` 上监听，并校验 Telegram 的 Webhook 密钥令牌。`webhookMaxConnections` 默认为 `4`；nanobot 在将 Telegram 更新转发给智能体之前仍会按会话串行化。
>
> `webhookUrl` 是注册到 Telegram 的公网 HTTPS URL。`webhookPath` 是 nanobot 本地监听的路径。它们通常使用同一路径，但当反向代理或隧道会重写请求路径时可能不同。

</details>

<details>
<summary><b>Mochat (Claw IM)</b></summary>

默认使用 **Socket.IO WebSocket**，并以 HTTP 轮询作为回退。

**安装可选的实时依赖**

```bash
nanobot plugins enable mochat
```

即使不安装此可选依赖，Mochat 也仍可通过 HTTP 轮询工作。

**1. 让 nanobot 帮你自动配置 Mochat**

只需向 nanobot 发送以下消息（将 `xxx@xxx` 替换为你的真实邮箱）：

```
Read https://raw.githubusercontent.com/HKUDS/MoChat/refs/heads/main/skills/nanobot/skill.md and register on MoChat. My Email account is xxx@xxx Bind me as your owner and DM me on MoChat.
```

nanobot 将自动注册、配置 `~/.nanobot/config.json` 并连接到 Mochat。

**2. 重启网关**

```bash
nanobot gateway
```

就这么简单 —— 剩下的都交给 nanobot！

<br>

<details>
<summary>手动配置（进阶）</summary>

如果你更偏好手动配置，请将以下内容添加到 `~/.nanobot/config.json`：

> 请保管好 `claw_token`。它只应通过 `X-Claw-Token` 请求头发送到你的 Mochat API 端点。

```json
{
  "channels": {
    "mochat": {
      "enabled": true,
      "base_url": "https://mochat.io",
      "socket_url": "https://mochat.io",
      "socket_path": "/socket.io",
      "claw_token": "claw_xxx",
      "agent_user_id": "6982abcdef",
      "sessions": ["*"],
      "panels": ["*"],
      "reply_delay_mode": "non-mention",
      "reply_delay_ms": 120000
    }
  }
}
```



</details>

</details>

<details>
<summary><b>Discord</b></summary>

**1. 创建一个机器人**
- 前往 https://discord.com/developers/applications
- 创建一个应用 → Bot → Add Bot
- 复制 Bot 令牌

**2. 启用 intents**
- 在 Bot 设置中，启用 **MESSAGE CONTENT INTENT**
- （可选）如果你计划基于成员数据使用允许列表，请启用 **SERVER MEMBERS INTENT**

**3. 获取你的 User ID**
- Discord 设置 → 高级 → 启用 **开发者模式**
- 右键点击你的头像 → **复制用户 ID**

**4. 配置**

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": ["YOUR_USER_ID"],
      "allowChannels": [],
      "groupPolicy": "mention",
      "streaming": true
    }
  }
}
```

> `groupPolicy` 控制机器人如何在群组频道中响应：
> - `"mention"`（默认）— 仅在被 @ 时响应
> - `"open"` — 响应所有消息
> DM 在发送者位于 `allowFrom` 中时始终响应。
> - 如果你把群组策略设为 open，请将新线程创建为私有线程，然后再把机器人 @ 进去。否则线程本身以及你开线程所在的频道都会派生出一个机器人会话。
> `allowChannels` 将机器人限制在特定的 Discord 频道 ID 内。为空（默认）表示在机器人能看到的每个频道都响应。示例：`["1234567890", "0987654321"]`。此过滤在 `allowFrom` 之后应用，因此两者都必须通过。允许的父频道下的 Discord 线程也允许通过；对于 Forum 频道，允许其父 Forum 频道则允许该 Forum 内所有线程/帖子。
> `streaming` 默认为 `true`。仅在你明确希望非流式回复时才禁用它。

**5. 邀请机器人**
- OAuth2 → URL Generator
- Scopes：`bot`
- Bot Permissions：`Send Messages`、`Read Message History`
- 打开生成的邀请 URL，将机器人添加到你的服务器

**6. 运行**

```bash
nanobot gateway
```

</details>

<details>
<summary><b>Matrix (Element)</b></summary>

首先启用 Matrix 支持：

```bash
nanobot plugins enable matrix
```

> [!NOTE]
> Matrix 加密在 Windows 上默认禁用，因为 `matrix-nio[e2e]` 依赖 `python-olm`，而后者没有 Windows 预编译 wheel。如果你需要 Matrix E2EE，请使用 macOS、Linux 或 WSL2。

**1. 创建/选择一个 Matrix 账户**

- 在你的 homeserver（例如 `matrix.org`）上创建或复用一个 Matrix 账户。
- 确认你可以通过 Element 登录。

**2. 获取凭据**

- 你需要：
  - `userId`（示例：`@nanobot:matrix.org`）
  - `password`

（注：出于兼容性原因，`accessToken` 和 `deviceId` 仍受支持，但为了可靠的加密，推荐改用密码登录。如果提供了 `password`，`accessToken` 和 `deviceId` 将被忽略。）

**3. 配置**

```json
{
  "channels": {
    "matrix": {
      "enabled": true,
      "homeserver": "https://matrix.org",
      "userId": "@nanobot:matrix.org",
      "password": "mypasswordhere",
      "e2eeEnabled": true,
      "sasVerification": true,
      "allowFrom": ["@your_user:matrix.org"],
      "groupPolicy": "open",
      "groupAllowFrom": [],
      "allowRoomMentions": false,
      "maxMediaBytes": 20971520
    }
  }
}
```

> 保留一个持久的 `matrix-store` —— 如果这些跨重启发生变化，加密会话状态会丢失。

| 选项 | 描述 |
|--------|-------------|
| `allowFrom` | 允许交互的用户 ID。为空则拒绝所有；使用 `["*"]` 表示允许所有人。 |
| `groupPolicy` | `open`（默认）、`mention` 或 `allowlist`。 |
| `groupAllowFrom` | 房间允许列表（当策略为 `allowlist` 时使用）。 |
| `allowRoomMentions` | 在 mention 模式下接受 `@room` 提及。 |
| `e2eeEnabled` | E2EE 支持（默认 `true`）。设为 `false` 则仅使用明文。 |
| `sasVerification` | 自动完成来自允许用户的 SAS 设备验证请求（默认 `false`）。对 Element X 有用，因为它不会为第三方设备暴露手动信任。 |
| `maxMediaBytes` | 附件最大大小（默认 `20MB`）。设为 `0` 则屏蔽所有媒体。 |




**4. 运行**

```bash
nanobot gateway
```

</details>

<details>
<summary><b>WhatsApp</b></summary>

需要 WhatsApp 的可选依赖：

```bash
nanobot plugins enable whatsapp
```

**1. 使用二维码链接设备**

```bash
nanobot channels login whatsapp
# 用 WhatsApp 扫描二维码 → 设置 → 已链接的设备
```

**2. 配置**

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["1234567890"]
    }
  }
}
```

可选的会话数据库路径：

```json
{
  "channels": {
    "whatsapp": {
      "databasePath": "~/.nanobot/whatsapp-auth/neonize.db"
    }
  }
}
```

**从旧的 bridge 迁移**

- 移除 `bridgeUrl` 和 `bridgeToken`；WhatsApp 不再运行本地 Node.js bridge。
- 重新运行 `nanobot channels login whatsapp`；neonize 不会复用旧的 Baileys bridge 认证数据。
- 将 `allowFrom` 条目更新为不带前导 `+` 的 WhatsApp 发送者 ID。

**3. 运行**

```bash
nanobot gateway
```

**可选：静态 LID 映射**

现代 WhatsApp 可能会传递发送者的 LID 而非其电话号码。nanobot
会在两个标识都出现时于运行时学习 LID 到电话号码的映射，但你
也可以预先播种映射，这样从第一条消息起电话号码就能被解析：

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["1234567890"],
      "lidMappings": { "123456789012345": "1234567890" }
    }
  }
}
```

</details>

<details>
<summary><b>Feishu</b></summary>

使用 **WebSocket** 长连接 —— 无需公网 IP。

**快速配置：二维码登录**

```bash
nanobot plugins enable feishu
nanobot channels login feishu
# 使用 --force 可创建/登录一个新机器人
```

打开打印出的 URL，或用手机上的 Feishu/Lark 扫描二维码。如果安装了可选的 `qrcode` 包，nanobot 会在终端显示二维码；否则会打印登录 URL。nanobot 会在当前使用的配置文件的 `channels.feishu` 下写入 `appId`、`appSecret`、`domain` 和 `enabled`。使用 `--config <path>` 可更新非默认配置。

如果二维码登录对你的账户不可用，请使用下面的手动配置。

**手动配置**

**1. 创建飞书机器人**
- 访问[飞书开放平台](https://open.feishu.cn/app)
- 创建新应用 → 启用 **机器人** 能力
- **权限**：
  - `im:message`（发送消息）和 `im:message.p2p_msg:readonly`（接收消息）
  - **流式回复**（nanobot 默认）：添加 **`cardkit:card:write`**（在飞书开发者控制台中通常显示为 **创建和更新卡片**）。CardKit 实体和流式助手文本需要它。旧版应用可能还没有此权限 —— 打开**权限管理**，启用该权限，如控制台要求则**发布**一个新的应用版本。
  - 如果你**无法**添加 `cardkit:card:write`，请在 `channels.feishu` 下设置 `"streaming": false`（见下）。机器人仍然可用；回复会使用普通交互卡片而非逐 token 流式。
- **事件**：添加 `im.message.receive_v1`（接收消息）
  - 选择**长连接**模式（需要先运行 nanobot 才能建立连接）
- 从"凭证与基础信息"中获取 **App ID** 和 **App Secret**
- 发布应用

**2. 配置**

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_xxx",
      "appSecret": "xxx",
      "encryptKey": "",
      "verificationToken": "",
      "allowFrom": ["ou_YOUR_OPEN_ID"],
      "groupPolicy": "mention",
      "reactEmoji": "OnIt",
      "doneEmoji": "DONE",
      "toolHintPrefix": "🔧",
      "streaming": true,
      "domain": "feishu"
    }
  }
}
```

> `streaming` 默认为 `true`。如果你的应用没有 **`cardkit:card:write`** 权限（见上文），请使用 `false`。
> `encryptKey` 和 `verificationToken` 在长连接模式下是可选的。
> `allowFrom`：添加你的 open_id（当你向机器人发消息时，可在 nanobot 日志中找到）。使用 `["*"]` 允许所有用户。
> `groupPolicy`：`"mention"`（默认 — 仅在被 @ 时响应）、`"open"`（响应所有群消息）。私聊始终响应。
> `reactEmoji`：用于"处理中"状态的表情（默认：`OnIt`）。参见[可用表情](https://open.larkoffice.com/document/server-docs/im-v1/message-reaction/emojis-introduce)。
> `doneEmoji`：用于"已完成"状态的可选表情（例如 `DONE`、`OK`、`HEART`）。设置后，机器人会在移除 `reactEmoji` 之后添加此反应。
> `toolHintPrefix`：流式卡片中内联工具提示的前缀（默认：`🔧`）。
> `domain`：`"feishu"`（默认）用于中国（open.feishu.cn），`"lark"` 用于国际版 Lark（open.larksuite.com）。

**3. 运行**

```bash
nanobot gateway
```

> [!TIP]
> 飞书使用 WebSocket 接收消息 —— 无需 Webhook 或公网 IP！

</details>

<details>
<summary><b>QQ (QQ单聊)</b></summary>

使用 **botpy SDK** 配合 WebSocket —— 无需公网 IP。目前**仅支持私聊消息**。

**安装可选的频道依赖**

```bash
nanobot plugins enable qq
```

**1. 注册并创建机器人**
- 访问 [QQ 开放平台](https://q.qq.com) → 注册为开发者（个人或企业）
- 创建新的机器人应用
- 前往 **开发设置** → 复制 **AppID** 和 **AppSecret**

**2. 设置沙箱以进行测试**
- 在机器人管理控制台中，找到 **沙箱配置**
- 在 **在消息列表配置** 下，点击 **添加成员** 并添加你自己的 QQ 号
- 添加后，用手机 QQ 扫描机器人二维码 → 打开机器人资料 → 点击"发消息"开始聊天

**3. 配置**

> - `allowFrom`：添加你的 openid（当你向机器人发消息时，可在 nanobot 日志中找到）。使用 `["*"]` 表示公开访问。
> - `msgFormat`：可选。使用 `"plain"`（默认）以最大化兼容旧版 QQ 客户端，或使用 `"markdown"` 在新版客户端上获得更丰富的格式。
> - 生产环境：在机器人控制台提交审核并发布。请参见 [QQ 机器人文档](https://bot.q.qq.com/wiki/) 了解完整的发布流程。

```json
{
  "channels": {
    "qq": {
      "enabled": true,
      "appId": "YOUR_APP_ID",
      "secret": "YOUR_APP_SECRET",
      "allowFrom": ["YOUR_OPENID"],
      "msgFormat": "plain"
    }
  }
}
```

**4. 运行**

```bash
nanobot gateway
```

现在从 QQ 向机器人发送一条消息 —— 它应该会响应！

</details>

<details>
<summary><b>Napcat (QQ via OneBot v11 支持群聊等功能)</b></summary>

通过其**正向 WebSocket**（OneBot v11）连接到 [Napcat](https://github.com/NapNeko/NapCatQQ) 实例。当你有自己的 QQ 账号通过 Napcat 运行，且想要完整支持私聊 + 群聊时使用此方式。

**1. 设置 Napcat**

- 安装并登录 Napcat，然后启用一个**正向 WebSocket** 服务器。参见 [Napcat 官方 Docker 教程](https://github.com/NapNeko/NapCat-Docker)。
- 在 WebUI 中，依次点击"网络配置" -> "新建" -> "Websocket 服务器"以创建正向 WebSocket 服务器。默认情况下，URL 为 `ws://127.0.0.1:3001`
- 复制正向 WebSocket 服务器的令牌
- （可选）在 WebUI 中，依次点击"系统配置" -> "登陆配置" -> "快速登录QQ"以在重启后自动登录

**安装可选的频道依赖**

```bash
nanobot plugins enable napcat
```

**2. 配置**

```json
{
  "channels": {
    "napcat": {
      "enabled": true,
      "wsUrl": "ws://127.0.0.1:3001",
      "accessToken": "YOUR_WEBSOCKET_TOKEN",
      "allowFrom": ["*"],
      "groupPolicy": "mention",
      "groupPolicyOverrides": {
        "123456789": "open",
        "987654321": 0.2
      },
      "welcomeNewMembers": true
    }
  }
}
```

| 选项 | 作用 |
|--------|--------------|
| `wsUrl` | Napcat 正向 WebSocket 端点。通过 `accessToken` 的 Bearer 认证在 `Authorization` 请求头中发送。 |
| `allowFrom` | 允许与机器人对话的 QQ 号。`["*"]` = 任何人。要触发 `welcomeNewMembers`，需要 `["*"]`（或包含加入的用户）。 |
| `groupPolicy` | `"mention"`（默认）— 仅在被 @ 或有人回复机器人自己的消息时响应。`"open"` — 响应每条群消息。`[0.0, 1.0]` 范围内的浮点数 `p` —— @ 提及和回复机器人始终会响应；其他每条群消息以概率 `p` 响应（因此 `0.0` 等价于 `"mention"`，`1.0` 等价于 `"open"`）。私聊始终响应。 |
| `groupPolicyOverrides` | 可选的每群覆盖 `groupPolicy`，以群 id（字符串）作为键。每个值的形态与 `groupPolicy` 相同（`"mention"`、`"open"` 或浮点数）。未列出的群回退到 `groupPolicy`。 |
| `welcomeNewMembers` | 为 true 时，`notice.group_increase` 事件会作为合成消息推送到消息总线，使智能体能够欢迎新加入的成员。 |
| `maxImageBytes` | 入站图片下载的硬上限（字节）。默认为 20 MB。较大的图片将被丢弃并给出警告。 |

</details>

<details>
<summary><b>DingTalk (钉钉)</b></summary>

使用**流模式** —— 无需公网 IP。

**安装可选的频道依赖**

```bash
nanobot plugins enable dingtalk
```

**1. 创建钉钉机器人**
- 访问[钉钉开放平台](https://open-dev.dingtalk.com/)
- 创建新应用 -> 添加**机器人**能力
- **配置**：
  - 打开 **Stream Mode** 开关
- **权限**：添加发送消息所需的权限
- 从"凭证"中获取 **AppKey**（Client ID）和 **AppSecret**（Client Secret）
- 发布应用

**2. 配置**

```json
{
  "channels": {
    "dingtalk": {
      "enabled": true,
      "clientId": "YOUR_APP_KEY",
      "clientSecret": "YOUR_APP_SECRET",
      "allowFrom": ["YOUR_STAFF_ID"],
      "groupUserIsolation": false
    }
  }
}
```

> `allowFrom`：添加你的员工 ID。使用 `["*"]` 允许所有用户。
>
> `groupUserIsolation`：可选。默认为 `false`，一个群聊共享一个会话。设为 `true` 可为钉钉群聊中每个发送者提供独立会话，同时回复仍会返回到同一个群。

**3. 运行**

```bash
nanobot gateway
```

</details>

<details>
<summary><b>Slack</b></summary>

使用 **Socket Mode** —— 无需公网 URL。

**安装可选的频道依赖**

```bash
nanobot plugins enable slack
```

**1. 创建 Slack 应用**
- 前往 [Slack API](https://api.slack.com/apps) → **Create New App** → "From scratch"
- 选择一个名字并选择你的 workspace

**2. 配置应用**
- **Socket Mode**：切换为 ON → 生成带有 `connections:write` 作用域的 **App-Level Token** → 复制它（`xapp-...`）
- **OAuth & Permissions**：添加 bot 作用域：`chat:write`、`reactions:write`、`app_mentions:read`、`files:read`、`files:write`、`channels:history`、`groups:history`、`im:history`、`mpim:history`
- **Event Subscriptions**：切换为 ON → 订阅 bot 事件：`message.im`、`message.channels`、`app_mention` → 保存更改
- **App Home**：滚动到 **Show Tabs** → 启用 **Messages Tab** → 勾选 **"Allow users to send Slash commands and messages from the messages tab"**
- **Install App**：点击 **Install to Workspace** → 授权 → 复制 **Bot Token**（`xoxb-...`）

> 读取用户发送给 nanobot 的文件需要 `files:read`。nanobot 发送图片、视频和其他文件上传需要 `files:write`。若你之后再添加任一作用域，请将 Slack 应用重新安装到 workspace，并重启 nanobot 以使用更新后的 bot token。

**3. 配置 nanobot**

```json
{
  "channels": {
    "slack": {
      "enabled": true,
      "botToken": "xoxb-...",
      "appToken": "xapp-...",
      "allowFrom": ["YOUR_SLACK_USER_ID"],
      "groupPolicy": "mention"
    }
  }
}
```

**4. 运行**

```bash
nanobot gateway
```

直接给机器人发 DM，或在频道中 @ 它 —— 它应该会响应！

> [!TIP]
> - `groupPolicy`：`"mention"`（默认 — 仅在被 @ 时响应）、`"open"`（响应所有频道消息），或 `"allowlist"`（通过 `groupAllowFrom` 限制到特定频道）。
> - `groupAllowFrom`：当 `groupPolicy` 为 `"allowlist"` 时，机器人可以响应的频道 ID。
> - `groupRequireMention`：当为 `true` 且 `groupPolicy` 为 `"allowlist"` 时，机器人只在 `groupAllowFrom` 中的频道**且**只在被 @ 时回复（而非每条消息）。对 `"mention"`/`"open"` 无效。使用它可以将机器人限制在已批准的频道中，同时保持仅 @ 才响应的行为。
> - DM 策略默认开放。设置 `"dm": {"enabled": false}` 可禁用 DM。

</details>

<details>
<summary><b>Email</b></summary>

给 nanobot 分配一个属于它自己的电子邮箱账户。它通过 **IMAP** 轮询接收邮件，并通过 **SMTP** 回复 —— 就像一个个人邮件助理。

**1. 获取凭据（Gmail 示例）**
- 为你的机器人创建一个专用 Gmail 账户（例如 `my-nanobot@gmail.com`）
- 启用两步验证 → 创建一个[应用专用密码](https://myaccount.google.com/apppasswords)
- 将此应用专用密码同时用于 IMAP 和 SMTP

**2. 配置**

> - `consentGranted` 必须为 `true` 才允许访问邮箱。这是一个安全门 —— 设为 `false` 可完全禁用。
> - `allowFrom`：添加你的邮箱地址。使用 `["*"]` 可接受任何人的邮件。
> - `smtpUseTls` 与 `smtpUseSsl` 默认分别为 `true` / `false`，这对于 Gmail 是正确的（587 端口 + STARTTLS）。无需显式设置。
> - 如果你只想读取/分析邮件而不自动回复，请设置 `"autoReplyEnabled": false`。
> - `postAction`：对已处理邮件的可选后处理：`"delete"` 或 `"move"`（默认 `null`）。
>   仅在被接受的邮件成功送达 AI 流水线后运行。
> - `postActionMoveMailbox`：当 `postAction` 为 `"move"` 时使用的目标邮箱（例如 `"Processed"` 或 `"[Gmail]/Trash"`）。
> - `postActionIgnoreSkipped`：如为 `true`（默认），跳过的邮件在后处理中被忽略，不会被移动/删除。
> - `postActionExpunge`：当为 `true` 时，如果基于 UID 范围的 EXPUNGE 不可用或失败，频道允许使用全邮箱 `EXPUNGE` 作为回退（默认 `false`）。仅在缺乏现代 UIDPLUS 支持的非常老旧的 IMAP 服务器上启用。请注意此回退将**清除**邮箱中所有被标记为删除的消息，包括并非由智能体处理的那些。对所有现代 IMAP 服务器保持关闭即可安全。
> - `allowedAttachmentTypes`：保存匹配这些 MIME 类型的入站附件 —— `["*"]` 表示全部，例如 `["application/pdf", "image/*"]`（默认 `[]` = 禁用）。
> - `maxAttachmentSize`：每个附件的最大大小（字节），默认 `2000000`（2MB）。
> - `maxAttachmentsPerEmail`：每封邮件保存的最大附件数（默认 `5`）。

```json
{
  "channels": {
    "email": {
      "enabled": true,
      "consentGranted": true,
      "imapHost": "imap.gmail.com",
      "imapPort": 993,
      "imapUsername": "my-nanobot@gmail.com",
      "imapPassword": "your-app-password",
      "smtpHost": "smtp.gmail.com",
      "smtpPort": 587,
      "smtpUsername": "my-nanobot@gmail.com",
      "smtpPassword": "your-app-password",
      "fromAddress": "my-nanobot@gmail.com",
      "allowFrom": ["your-real-email@gmail.com"],
      "postAction": "move",
      "postActionMoveMailbox": "[Gmail]/Trash",
      "postActionIgnoreSkipped": true,
      "postActionExpunge": false,
      "allowedAttachmentTypes": ["application/pdf", "image/*"]
    }
  }
}
```


**3. 运行**

```bash
nanobot gateway
```

</details>

<details>
<summary><b>WeChat (微信 / Weixin)</b></summary>

使用 **HTTP 长轮询**并通过 ilinkai 个人微信 API 进行二维码登录。无需本地微信桌面客户端。

**1. 启用 WeChat 支持**

```bash
nanobot plugins enable weixin
```

**2. 配置**

```json
{
  "channels": {
    "weixin": {
      "enabled": true,
      "allowFrom": ["YOUR_WECHAT_USER_ID"]
    }
  }
}
```

> - `allowFrom`：添加你在 nanobot 日志中看到的、属于你微信账号的发送者 ID。使用 `["*"]` 允许所有用户。
> - `token`：可选。如果省略，请交互式登录，nanobot 会为你保存令牌。
> - `routeTag`：可选。当你的上游 Weixin 部署需要请求路由时，nanobot 会将其作为 `SKRouteTag` 请求头发送。
> - `stateDir`：可选。默认使用 nanobot 用于 Weixin 状态的运行时目录。
> - `pollTimeout`：可选的长轮询超时时间（秒）。

**3. 登录**

```bash
nanobot channels login weixin
```

使用 `--force` 可重新认证并忽略任何已保存的令牌：

```bash
nanobot channels login weixin --force
```

**4. 运行**

```bash
nanobot gateway
```

</details>

<details>
<summary><b>Wecom (企业微信)</b></summary>

> 这里我们使用 [wecom-aibot-sdk-python](https://github.com/chengyongru/wecom_aibot_sdk)（官方 [@wecom/aibot-node-sdk](https://www.npmjs.com/package/@wecom/aibot-node-sdk) 的社区 Python 版本）。
>
> 使用 **WebSocket** 长连接 —— 无需公网 IP。

**1. 启用 WeCom 支持**

```bash
nanobot plugins enable wecom
```

**2. 创建 WeCom AI 机器人**

进入企业微信管理后台 → 智能机器人 → 创建机器人 → 选择 **API 模式**并使用**长连接**。复制 Bot ID 和 Secret。

**3. 配置**

```json
{
  "channels": {
    "wecom": {
      "enabled": true,
      "botId": "your_bot_id",
      "secret": "your_bot_secret",
      "allowFrom": ["your_id"]
    }
  }
}
```

**4. 运行**

```bash
nanobot gateway
```

</details>

<details>
<summary><b>Microsoft Teams</b>（MVP — 仅 DM）</summary>

> 双向 DM 文本、面向租户的 OAuth、会话引用持久化。
> 使用公网 HTTPS Webhook —— 不使用 WebSocket；你需要一个隧道或反向代理。

**1. 启用 Microsoft Teams 支持**

```bash
nanobot plugins enable msteams
```

**2. 创建 Teams / Azure bot 应用注册**

创建或复用一个 Microsoft Teams / Azure bot 应用注册。将机器人 messaging 端点设置为以 `/api/messages` 结尾的公网 HTTPS URL。

**3. 配置**

```json
{
  "channels": {
    "msteams": {
      "enabled": true,
      "appId": "YOUR_APP_ID",
      "appPassword": "YOUR_APP_SECRET",
      "tenantId": "YOUR_TENANT_ID",
      "host": "0.0.0.0",
      "port": 3978,
      "path": "/api/messages",
      "allowFrom": ["*"],
      "replyInThread": true,
      "mentionOnlyResponse": "Hi — what can I help with?",
      "validateInboundAuth": true,
      "refTtlDays": 30,
      "pruneWebChatRefs": true,
      "pruneNonPersonalRefs": true,
      "refTouchIntervalS": 300
    }
  }
}
```

> - `replyInThread: true`：当存储了 `activity_id` 时，回复到触发的 Teams activity。
> - `mentionOnlyResponse` 控制当用户仅发送机器人提及（`<at>Nanobot</at>`）时 Nanobot 接收到的内容。设为 `""` 可忽略仅有提及的消息。
> - `validateInboundAuth: true` 启用入站 Bot Framework bearer-token 校验（签名、issuer、audience、生命周期、`serviceUrl`）。这是公网部署的安全默认值。仅在本地开发或严格受控的测试中才设置为 `false`。
> - `refTtlDays`（默认 `30`）控制存储的会话引用可保留多久后被裁剪。
> - `pruneWebChatRefs`（默认 `true`）会丢弃 service URL 为 `webchat.botframework.com` 的引用。
> - `pruneNonPersonalRefs`（默认 `true`）会丢弃 `conversation_type` 不是 `personal` 的引用。
> - `refTouchIntervalS`（默认 `300`）会限制成功发送对活跃引用刷新 `updated_at` 的频率。

**4. 运行**

```bash
nanobot gateway
```

</details>

<details>
<summary><b>Signal</b></summary>

在 HTTP 模式下使用 **signal-cli** 守护进程 —— 通过 SSE 接收消息，通过 JSON-RPC 发送。

**1. 安装 signal-cli**

安装 [signal-cli](https://github.com/AsamK/signal-cli) 并注册一个电话号码：

```bash
signal-cli -u +1234567890 register
signal-cli -u +1234567890 verify <CODE>
```

启动守护进程：

```bash
signal-cli -a +1234567890 daemon --http localhost:8080
```

**2. 配置**

```json
{
  "channels": {
    "signal": {
      "enabled": true,
      "phoneNumber": "+1234567890",
      "daemonHost": "localhost",
      "daemonPort": 8080,
      "dm": {
        "enabled": true,
        "policy": "open"
      },
      "group": {
        "enabled": true,
        "policy": "open",
        "requireMention": true
      }
    }
  }
}
```

> - `phoneNumber`：你已注册的 Signal 电话号码。
> - `daemonHost` / `daemonPort`：signal-cli 守护进程的监听地址（默认 `localhost:8080`）。
> - `dm.policy`：`"open"`（任何人都可以 DM）或 `"allowlist"`（仅列出的号码/UUID）。当为 `"allowlist"` 时，未列出的 DM 发送者会收到一个配对码。
> - `dm.allowFrom`：允许的电话号码或 UUID 列表（当策略为 `"allowlist"` 时使用）。
> - `group.policy`：`"open"`（所有群组）或 `"allowlist"`（仅列出的群组 ID）。
> - `group.requireMention`：当为 `true`（默认）时，机器人只在被 @ 时于群组中响应。
> - `group.allowFrom`：允许的群组 ID 列表（当群组策略为 `"allowlist"` 时使用）。
> - `attachmentsDir`：覆盖 signal-cli 存储入站附件的目录。默认 `~/.local/share/signal-cli/attachments`（Linux 默认路径）。当 signal-cli 使用自定义 `XDG_DATA_HOME` 运行，或运行在 macOS/Windows 上时，请设置该值。
> - `groupMessageBufferSize`：为上下文保留的最近群消息数（默认 `20`，必须 > 0）。

**3. 运行**

```bash
nanobot gateway
```

> [!TIP]
> 如果与 signal-cli 守护进程的连接中断，频道会以指数回退自动重连。
> 机器人回复中的 Markdown 会自动转换为 Signal 文本样式（粗体、斜体、代码等）。

</details>
