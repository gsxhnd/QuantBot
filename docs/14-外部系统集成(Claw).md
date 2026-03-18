# 外部系统集成 (Claw)

## 1. 系统定位

| 系统 | 角色 | 职责 |
|------|------|------|
| **QuantBot** | 分析引擎 | 数据分析、信号生成、风险评估、告警触发 |
| **Claw** | 信息中枢 | 新闻抓取、情绪分析、用户通知 |
| **用户** | 决策者 | 接收建议、手动交易 |

## 2. Claw集成

### 2.1 支持的Claw类型

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| **OpenClaw** | 开源版本，自部署 | 完全本地化，数据隐私要求高 |
| **ZeroClaw** | 托管服务版本 | 快速接入，无需运维 |

### 2.2 配置切换

```yaml
# config/claw.yaml
claw:
  # 类型选择：openclaw 或 zeroclaw
  type: openclaw
  
  # OpenClaw配置
  openclaw:
    endpoint: http://localhost:8081
    api_key: ${OPENCLAW_API_KEY}
  
  # ZeroClaw配置
  zeroclaw:
    endpoint: https://api.zeroclaw.com
    api_key: ${ZEROCLAW_API_KEY}
```

## 3. 通信协议

### 3.1 通信方式概览

| 方向 | 数据类型 | 协议 | 说明 |
|------|---------|------|------|
| Claw → QuantBot | 新闻数据 | HTTP Webhook | Claw主动推送 |
| QuantBot → Claw | 交易建议 | HTTP POST | QuantBot主动推送 |
| QuantBot → Claw | 告警通知 | HTTP POST | QuantBot主动推送 |

### 3.2 Claw → QuantBot（新闻推送）

**Webhook配置**：
```
POST /api/claw/news
Content-Type: application/json
X-Claw-Signature: sha256=...
```

**新闻数据格式**：

```json
{
  "news_id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Apple Reports Record Q4 Earnings",
  "source": "Reuters",
  "author": "John Smith",
  "published_at": "2024-01-15T10:30:00Z",
  "url": "https://...",
  "related_stocks": ["AAPL", "MSFT"],
  "category": "earnings",
  "importance": "high",
  "sentiment": {
    "overall": "bullish",
    "confidence": 0.85,
    "dimensions": {
      "financial_impact": {
        "score": 0.7,
        "direction": "positive"
      },
      "market_sentiment": {
        "score": 0.6,
        "direction": "positive"
      },
      "regulatory_risk": {
        "score": 0.1,
        "direction": "neutral"
      }
    },
    "impact_level": "high",
    "expected_duration": "medium"
  },
  "summary": "Apple reported record Q4 earnings...",
  "raw_content": "Full article content..."
}
```

**情绪维度说明**：

| 维度 | 说明 | 取值范围 |
|------|------|---------|
| overall | 整体情绪 | bullish/bearish/neutral |
| confidence | 置信度 | 0.0 - 1.0 |
| financial_impact | 财务影响 | score: 0-1, direction: positive/negative/neutral |
| market_sentiment | 市场情绪 | score: 0-1, direction: positive/negative/neutral |
| regulatory_risk | 监管风险 | score: 0-1, direction: positive/negative/neutral |
| impact_level | 影响程度 | high/medium/low |
| expected_duration | 预期持续时间 | short/medium/long |

### 3.3 QuantBot → Claw（信号推送）

**请求格式**：
```
POST /api/signals
Content-Type: application/json
Authorization: Bearer {API_KEY}
```

**扩展交易信号格式**：

```json
{
  "signal_id": "660e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2024-01-15T14:30:00Z",
  "stock_code": "AAPL",
  "stock_name": "Apple Inc.",
  "action": "buy",
  "confidence": 0.78,
  "reason": "GRU预测上涨1.5% + 正面新闻情绪",
  "price": {
    "current": 185.50,
    "suggested_entry": 185.00,
    "stop_loss": 180.00,
    "target": 195.00
  },
  "risk": {
    "var_95": 0.015,
    "position_suggestion": 5000,
    "position_percent": 0.05,
    "risk_level": "medium"
  },
  "cost_estimate": {
    "commission": 5.00,
    "slippage_estimate": 0.001,
    "total_cost_percent": 0.015
  },
  "execution_suggestion": {
    "timing": "开盘后30分钟",
    "method": "TWAP分批",
    "reason": "订单较大，建议分批执行"
  },
  "related_news": [
    {
      "news_id": "550e8400-...",
      "title": "Apple Reports Record Q4 Earnings",
      "sentiment": "bullish"
    }
  ],
  "valid_until": "2024-01-16T09:30:00Z"
}
```

### 3.4 QuantBot → Claw（告警推送）

**请求格式**：
```
POST /api/alerts
Content-Type: application/json
Authorization: Bearer {API_KEY}
```

**告警数据格式**：

```json
{
  "alert_id": "770e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2024-01-15T14:30:00Z",
  "level": "CRITICAL",
  "type": "stop_loss_trigger",
  "stock_code": "AAPL",
  "stock_name": "Apple Inc.",
  "current_price": 179.50,
  "trigger_condition": "止损价 180.00",
  "message": "AAPL 触及止损价180.00，当前价格179.50",
  "suggestion": "建议考虑是否执行止损操作",
  "action_required": false,
  "details": {
    "position_id": "pos_001",
    "entry_price": 190.00,
    "quantity": 100,
    "unrealized_pnl": -1050.00,
    "unrealized_pnl_percent": -0.055
  }
}
```

**告警类型**：

| 类型 | 说明 | 级别 |
|------|------|------|
| stop_loss_trigger | 止损价触发 | CRITICAL |
| take_profit_trigger | 止盈价触发 | INFO |
| trailing_stop_trigger | 移动止损触发 | WARNING |
| var_breach | VaR突破 | CRITICAL |
| drawdown_warning | 回撤预警 | WARNING |
| model_drift | 模型漂移 | WARNING |

## 4. 用户通知机制

### 4.1 新闻推送规则

Claw根据以下规则向用户推送新闻：

| 新闻类型 | 推送条件 | 优先级 |
|---------|---------|--------|
| 重要公告 | 持仓相关 + 高重要性 | 高 |
| 行业动态 | 持仓行业 + 高影响 | 中 |
| 宏观经济 | 高影响 | 中 |
| 持仓相关 | 任何持仓相关新闻 | 高 |

### 4.2 信号推送规则

| 条件 | 推送方式 |
|------|---------|
| 置信度 > 0.7 | 即时推送 |
| 置信度 0.5-0.7 | 摘要推送 |
| 置信度 < 0.5 | 不推送，仅记录 |

### 4.3 告警推送规则

| 级别 | 推送方式 |
|------|---------|
| INFO | 不推送，仅记录 |
| WARNING | 普通通知（App推送） |
| ERROR | 即时通知（App推送 + 邮件） |
| CRITICAL | 紧急通知（App推送 + 邮件 + 短信） |

## 5. 数据流程图

```
┌─────────────┐                    ┌─────────────┐
│   新闻源    │                    │   QuantBot  │
│  (媒体/API) │                    │  (分析引擎)  │
└──────┬──────┘                    └──────┬──────┘
       │                                  │
       ▼                                  ▼
┌─────────────┐  新闻+情绪   ┌─────────────────────┐
│    Claw     │─────────────▶│   新闻接收/存储     │
│ (信息中枢)   │              └──────────┬──────────┘
└──────┬──────┘                         │
       │                                ▼
       │                      ┌─────────────────────┐
       │                      │   情绪因子计算      │
       │                      └──────────┬──────────┘
       │                                │
       │                                ▼
       │                      ┌─────────────────────┐
       │                      │   AI模型推理        │
       │                      └──────────┬──────────┘
       │                                │
       │                                ▼
       │                      ┌─────────────────────┐
       │                      │   信号生成/风险评估 │
       │                      └──────────┬──────────┘
       │                                │
       │◀────────信号/告警───────────────┘
       │
       ▼
┌─────────────┐
│    用户     │
│  (决策者)   │
└─────────────┘
```

## 6. API接口

### 6.1 QuantBot提供的接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/claw/news` | POST | 接收Claw推送的新闻 |
| `/api/claw/heartbeat` | POST | Claw心跳检测 |

### 6.2 Claw提供的接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/signals` | POST | 接收交易信号 |
| `/api/alerts` | POST | 接收告警通知 |
| `/api/health` | GET | 健康检查 |

## 7. 安全机制

### 7.1 认证

- API Key认证
- HMAC签名验证（可选）

### 7.2 数据安全

- HTTPS加密传输
- 敏感数据不传输（账户密码等）

### 7.3 防重放

- 请求时间戳验证
- 请求ID去重

## 8. 容错机制

| 场景 | 处理方式 |
|------|---------|
| Claw不可用 | 新闻缓存本地，信号本地存储，恢复后重发 |
| QuantBot不可用 | Claw缓存新闻，恢复后重推 |
| 网络中断 | 本地队列缓冲，自动重试 |
| 数据格式错误 | 记录日志，丢弃错误数据 |