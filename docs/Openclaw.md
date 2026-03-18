# OpenClaw

## OpenClaw是什么

[OpenClaw](https://github.com/openclaw/openclaw) 是一个**个人AI助手框架**（类似Copilot），而不是"新闻抓取+情绪分析"系统。

**OpenClaw 的核心功能**：

- 多消息渠道集成（WhatsApp、Telegram、Slack、Discord、Google Chat等）
- WebSocket-based Gateway（控制平面）
- 支持skills（插件系统）
- 语音能力、Canvas画布、浏览器控制等

## OpenClaw 提供的通信接口

是的，OpenClaw 确实提供通信接口：

| 接口类型 | 说明 |
|---------|------|
| **WebSocket Gateway** | `ws://localhost:18789` — 核心控制平面 |
| **Webhooks** | 支持外部触发的webhook |
| **CLI** | `openclaw` 命令行工具 |
| **REST API** | 通过Gateway间接提供 |

具体参考：

- [Webhooks文档](https://docs.openclaw.ai/automation/webhook)
- [Gateway架构](https://docs.openclaw.ai/concepts/architecture)

---

## 现实情况

您的项目文档中的"OpenClaw"定义（**新闻抓取 → 情绪分析 → 量化系统 → 通知**）**不是真实的OpenClaw**。

这意味着您有两个选择：

1. **自己实现** 一个类似的新闻系统（参考您文档中的设计）
2. **使用OpenClaw作为通知渠道** — 把它当作消息分发层，而不是情绪分析源

---

建议：**您的自定义设计（HTTP Webhook + HTTP POST）是正确的方向**。与其依赖真实OpenClaw的功能，不如自己实现或集成专门的新闻情绪分析API（如 Alpha Vantage、newsapi.ai等）。

明白，那就围绕这个架构设计：

**OpenClaw做新闻抓取+情绪分析，推送给QuantBot**。

## OpenClaw的能力确认

OpenClaw **可以** 做到这个，通过：

| 能力 | 用途 |
|------|------|
| **Cron Jobs** | 定时抓取新闻 |
| **Browser Control** | 网页爬取 / API调用 |
| **Skills** | 自定义新闻处理逻辑 |
| **LLM调用** | Claude分析情绪（调用你的API key） |
| **HTTP.post** | 推送新闻+情绪给QuantBot |

参考：

- [OpenClaw Cron](https://docs.openclaw.ai/automation/cron-jobs)
- [Skills](https://docs.openclaw.ai/tools/skills)
- [HTTP](https://docs.openclaw.ai/automation/webhook)

---
