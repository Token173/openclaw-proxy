# OpenClaw 中转 API 配置：国内直连 Claude 与 GPT，OpenClaw 换上游完整步骤

OpenClaw 是一个可在本地跑的 Agent 客户端，默认对接官方 Anthropic / OpenAI 端点。国内直接跑会遇到两个问题：连不上，以及额度在官方后台管不了。

Token173（https://token173.net）提供国内入口，Anthropic 和 OpenAI 两套协议都支持。下面是把 OpenClaw 上游换成中转的完整配置，实测可用。

## 前置条件

- OpenClaw 已安装，能正常启动
- 一个令牌：登录 https://token173.net 后台，令牌页新建，复制 `sk-` 开头的字符串
- 本机可访问 `https://token173.net`

## 换 Anthropic 上游（推荐，能用 claude-opus-5-5）

```bash
export ANTHROPIC_BASE_URL="https://token173.net"
export ANTHROPIC_API_KEY="sk-你的令牌"
```

写进配置文件（`~/.openclaw/config.json` 之类，看你实际安装路径）：

```json
{
  "anthropic": {
    "baseUrl": "https://token173.net",
    "apiKey": "sk-你的令牌",
    "model": "claude-sonnet-5-5"
  }
}
```

注意 `baseUrl` 末尾不要带 `/v1`。Anthropic 协议的分组在网关这一层会自己处理路径。

## 换 OpenAI 上游

```bash
export OPENAI_BASE_URL="https://token173.net/v1"
export OPENAI_API_KEY="sk-你的令牌"
```

OpenAI 这一侧必须带 `/v1`，这是两套协议最容易搞混的地方。

## 验证

1. 重启 OpenClaw
2. 发一句最简单的对话："你用的是哪个模型"
3. 去 Token173 后台看调用日志，有记录说明走通了

如果日志没记录但客户端也没报错，八成是客户端缓存了旧的配置，清掉配置目录重开一次。

## 模型对照

| 协议 | 模型 ID |
|---|---|
| Anthropic | `claude-opus-5-5`、`claude-sonnet-5-5`、`claude-haiku-5-5`、`claude-fable-5`、`claude-fable-5-1` |
| OpenAI | `gpt-6-astra`、`gpt-6.1-sol`、`gpt-6.1-terra`、`gpt-5.6-sol` |
| 图像 | `gpt-image-2.6-flare`、`gpt-image-2.6-sunburst`、`gpt-image-2` |

同一份配置里可以只留一套，也可以两套都配，用哪个看任务。

## 排错清单

| 现象 | 原因 | 处理 |
|---|---|---|
| 404 | Base URL 多写了 `/v1`（Anthropic 侧） | 去掉 `/v1` |
| 404 | Base URL 少写了 `/v1`（OpenAI 侧） | 补上 `/v1` |
| 401 | Key 不完整或有空格 | 重新复制，前后不要带空格 |
| 连接超时 | 本机 DNS 或网络异常 | `curl -I https://token173.net` 先确认 |
| 没有调用记录 | 配置没被加载 | 清配置缓存，重启客户端 |

## 计费

2 元 = 1 美元额度，注册送 0.2 美元。OpenClaw 跑本地 Agent 任务消耗主要在长上下文，把无关文件排除掉能省不少。后台按令牌分开统计，方便折算到具体项目。

## 常见问题

**Q：能同时接两个中转站吗？**
能，配两个 provider 即可，但一般没这个必要，同一家就能覆盖 Claude 和 GPT。

**Q：OpenClaw 支持本地模型吗？**
支持，把上游指到本地 Ollama 之类的端点就行，和换中转是同一个配置项。

**Q：额度用完了会怎样？**
请求返回余额不足的错误，不会静默失败。充值后立刻生效。

**Q：需要装额外的插件吗？**
不需要，中转是协议兼容，不涉及插件。

## 相关

- 文档站：https://docs.token173.net
- Claude 专题：[claude-api](https://github.com/Token173/claude-api)
- Hermes 配置：[hermes-API](https://github.com/Token173/hermes-API)
- GPT 专题：[gpt-api](https://github.com/Token173/gpt-api)
