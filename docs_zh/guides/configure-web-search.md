# 如何为 nanobot AI 智能体配置网页搜索

nanobot 内置了网页搜索和网页抓取工具。搜索默认使用 DuckDuckGo，并且可以配置
为使用基于 API 的或自托管的提供商。

## 你将构建什么

- 在 nanobot 中启用的网页工具
- 在 `config.json` 中选择的一个搜索提供商
- 可选的用于页面读取的网页抓取设置

## 何时使用

当智能体在任务中需要最新信息、公开网页调研、来源发现或页面抓取时，请配置
网页搜索。

## 安装

```bash
python -m pip install nanobot-ai
nanobot onboard --wizard
nanobot agent -m "Hello!"
```

网页工具默认启用。仅当你希望指定某个提供商、API 密钥、代理、抓取行为或 SSRF
白名单时才需要配置它们。

## 最小可用示例

使用默认搜索提供商：

```json
{
  "tools": {
    "web": {
      "enable": true,
      "search": {
        "provider": "duckduckgo"
      }
    }
  }
}
```

或使用基于 API 的提供商：

```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "brave",
        "apiKey": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

提出一个需要最新信息的问题，并在 WebUI 或日志中查看工具活动。

## 生产环境注意事项

- 将 API 密钥保存在环境变量中。
- 当每次查询需要更少或更多搜索结果时，设置 `maxResults`。
- 仅将 `tools.web.proxy` 设置为你信任的代理。
- 如果需要本地页面转换，将 `fetch.useJinaReader: false`。

## 安全注意事项

- 网页抓取和 HTTP MCP 共享同一个 SSRF 防护。
- 私有、回环、链路本地以及云元数据地址默认被拦截。
- 仅为狭窄的可信 CIDR 添加 `tools.ssrfWhitelist`。
- 未经审查，不要向公开的聊天用户开放无限制的网页和 shell 访问。

## 故障排查

- 如果搜索没有返回结果，请更换提供商或检查提供商的 API 密钥。
- 如果抓取被拦截，请检查目标 URL 和 SSRF 白名单。
- 如果代理改变了网络行为，请核对 `NO_PROXY` 和代理设置。

## 相关 nanobot 文档

- [配置：网页工具](../configuration.md#web-tools)
- [安全](../configuration.md#security)
- [WebUI](../webui.md)
