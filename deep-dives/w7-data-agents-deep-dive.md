# Week 7 Deep Dive · Data Agents + Fabric IQ + MCP

> **目标**：构建一个能回答 *"今年华东区咖啡机退货率"* 的 Data Agent，
> 让业务用户用自然语言查数据；理解 Fabric IQ 业务本体如何让 AI 用业务概念推理。
>
> **投入**：8–10 小时 ｜ **前置**：W5 DirectLake + W6 Mirroring

---

## 0. 为什么 Data Agent 是 2026 最重要的新范式

过去 30 年的"业务用户拿数据"路径：

```
业务用户 → 提需求 → BI 分析师 → 写 SQL → 出报表 → 业务用户看
   ↑                                                  ↓
   └────────── 反复迭代、平均 2 周一轮 ──────────────────┘
```

Data Agent 范式：

```
业务用户 → 自然语言问问题 → Data Agent ──┐
                ↑                       ├→ 自动选数据源
                │                       ├→ 自动写 SQL/DAX/KQL
                │                       ├→ 自动调用工具（Power BI 报表、Excel、邮件）
                │                       └→ 返回答案 + 数据出处 + 推理链
                │                            ↓
                └────────── 5 秒内拿到答案 ────┘
```

Fabric 在这个范式上有三件套互锁：
1. **Data Agents** —— 把数据资产打包成 Agent
2. **Fabric IQ Ontology** —— 给 Agent 业务语义（订单 / 客户 / 产品的关系）
3. **MCP 协议** —— 让 Agent 能被 Copilot / Copilot Studio / 自建 LLM 应用消费

---

## 1. 概念地图

```
┌─────────────────────────────────────────────────────────────┐
│             Consumer Layer                                  │
│  M365 Copilot │ Copilot Studio │ Custom LLM App │ Power BI  │
└──────────────────────┬──────────────────────────────────────┘
                       │ MCP / OData / REST
                       ▼
┌─────────────────────────────────────────────────────────────┐
│             Data Agent                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │  Instructions │  │ Grounding Src │  │ Skills/Tools │       │
│  │ (Prompt + RAG)│  │ (Lakehouse/DW │  │ (KQL/SQL/DAX │       │
│  │               │  │  /Sem Model)  │  │  generation) │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│             Fabric IQ (Ontology)                            │
│   订单 ←─ has ─→ 客户                                        │
│   订单 ←─ contains ─→ 产品                                   │
│   产品 ←─ belongs to ─→ 品类                                  │
│   退货 ←─ references ─→ 订单                                  │
│   ...（业务概念图）                                            │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│             OneLake Data                                    │
│  Lakehouse · Warehouse · KQL DB · Semantic Models           │
└─────────────────────────────────────────────────────────────┘
```

**核心隔离**：
- 数据层只关心物理存储
- 语义层（IQ）告诉 Agent 业务概念
- Agent 层把语义和工具能力组合成"会回答问题的助手"
- 消费层用 MCP 统一接入

---

## 2. 实操 Lab 1：创建第一个 Data Agent

### 2.1 准备数据

延用 W6 Lab 5 的 `retail_gold.fact_orders` + 维度表。

### 2.2 创建 Agent

1. Workspace → New → **Data agent**（preview/GA 视区域）
2. 命名：`retail-insights-agent`
3. 添加 grounding sources：
   - ✅ Lakehouse: `retail_gold`
   - ✅ Semantic Model: `retail_directlake_model`
   - （可选）✅ KQL Database: `clickstream_kql`

### 2.3 写 Instructions

```text
你是「跨境零售数据助手」。

职责：
- 帮业务用户回答关于订单、客户、产品、退货的问题
- 优先用语义模型 retail_directlake_model（DAX 查询，最快）
- 复杂分析或时序问题用 retail_gold Lakehouse 的 SQL
- 实时点击流问题用 clickstream_kql 的 KQL

回答风格：
- 总是先给一句话结论，再给数字证据
- 数字大于 4 位用千分位（如 12,345）
- 涉及金额用 ¥ 或 $ 前缀
- 不确定时说"我不确定"，不要编造

边界：
- 不回答与零售业务无关的问题
- 不暴露客户姓名、电话、邮箱等 PII
- 涉及预测时声明"基于历史数据估算"
```

### 2.4 测试

在 Agent 测试面板输入：

```
今年华东区咖啡机的退货率多少？
对比去年同期有什么变化？
```

观察：
- Agent 自动选了哪个 source
- 生成了什么 DAX / SQL
- 答案是否带数据出处链接

---

## 3. 实操 Lab 2：用 Fabric IQ 定义业务本体

### 3.1 为什么需要本体？

裸数据问题：
- 数据库里叫 `cust_id` 还是 `customer_id`？
- "退货率" 是 `returned_orders / total_orders` 还是 `returned_items / total_items`？
- "华东区" 在哪个表的哪个字段？

没本体的 Agent 只能猜，命中率 60-70%。有本体后 95%+。

### 3.2 定义 Ontology

Fabric IQ → New ontology → `retail-business-ontology`：

```yaml
concepts:
  Customer:
    description: "下过至少一笔订单的自然人或企业"
    properties:
      - id: customer_id
      - name: customer_name (sensitive)
      - region: customer_region    # 华东/华南/华北/华西
      - tier: customer_tier        # VIP/Gold/Silver/Regular

  Order:
    description: "客户的一次购买行为"
    properties:
      - id: order_id
      - placed_at: order_date
      - amount: total_amount (currency=CNY)
      - status: order_status       # placed/shipped/delivered/returned
    relations:
      - placed_by: Customer (via customer_id)
      - contains: Product (via OrderItem)

  Product:
    description: "可销售的 SKU"
    properties:
      - id: sku
      - name: product_name
      - category: category_name    # 咖啡机/水杯/...
      - country_of_origin: origin

  Return:
    description: "客户主动发起的退货请求"
    properties:
      - id: return_id
      - reason: return_reason
    relations:
      - references: Order (via order_id)

metrics:
  return_rate:
    description: "退货率 = 退货订单数 / 总订单数"
    formula: "count(Return) / count(Order)"
    grain:
      - by_region: Customer.region
      - by_category: Product.category
      - by_period: Order.placed_at (D/W/M/Q/Y)

  revenue:
    description: "总收入"
    formula: "sum(Order.amount where Order.status != 'returned')"
```

### 3.3 把 Ontology 绑给 Agent

Agent 编辑 → Linked ontology → 选 `retail-business-ontology`

再问："今年华东区咖啡机退货率" —— Agent 直接命中 `return_rate` 指标，按 `Customer.region='华东'` 和 `Product.category='咖啡机'` 过滤，无需猜测字段。

---

## 4. 实操 Lab 3：让 Agent 调用 Power BI 报表

Data Agent 不只回答问题，还能"打开报表"：

### 4.1 注册 Skill

Agent → Skills → Add → **Power BI Report Skill**

配置：
```json
{
  "name": "open_retail_dashboard",
  "description": "打开零售总览仪表板，支持按区域/品类过滤",
  "report_id": "<your-report-guid>",
  "filters_schema": {
    "region": "Customer.region",
    "category": "Product.category"
  }
}
```

### 4.2 测试

业务用户问：
> "把华东咖啡机的销售看板打开给我"

Agent 行为：
1. 识别意图："打开报表"
2. 调用 `open_retail_dashboard` skill
3. 注入 filter: `region=华东, category=咖啡机`
4. 返回带过滤参数的 Power BI 链接

---

## 5. 实操 Lab 4：MCP 协议让 Agent 被外部应用消费

### 5.1 暴露为 MCP server

Agent → Settings → API → 启用 **MCP endpoint**

得到一个 endpoint：
```
https://api.fabric.microsoft.com/v1/workspaces/<ws>/dataagents/<agent>/mcp
```

### 5.2 在 Copilot Studio / 自建 LLM 应用里接入

```python
# 用 OpenAI SDK + MCP 调用
from openai import OpenAI
import requests

client = OpenAI()

mcp_tools = requests.get(
    "https://api.fabric.microsoft.com/v1/.../mcp/tools",
    headers={"Authorization": f"Bearer {token}"}
).json()

resp = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "今年华东区咖啡机退货率"}],
    tools=mcp_tools  # Fabric Data Agent 的工具被直接挂载
)
```

### 5.3 在 Teams 里集成

通过 Copilot Studio → Connect Teams → 让团队成员直接 @bot 问问题。

---

## 6. 实操 Lab 5：进阶 — Translytical Task Flow

FabCon 2026 新特性：**让 Power BI 报表里的 Agent 直接写回数据**。

业务场景：客户经理在 Power BI 看到某客户投诉激增，想把 TA 升级为 VIP。
- 报表上有个 Agent 对话框
- 输入"把客户 C-12345 升级为 VIP"
- Agent 调用 Translytical Skill → 触发写回 Azure SQL（源端）→ Mirror 同步回 OneLake → 报表自动刷新

```python
# Skill 定义示意
{
  "name": "upgrade_customer_tier",
  "description": "升级客户等级",
  "parameters": {"customer_id": "string", "new_tier": "enum[VIP,Gold,Silver]"},
  "writeback_target": {
    "type": "azure_sql",
    "table": "Customers",
    "operation": "UPDATE Customers SET tier=@new_tier WHERE customer_id=@customer_id"
  },
  "rbac": "role:customer_manager"
}
```

这是 Fabric 把 **BI 从"看"变成"操作"** 的关键步骤。

---

## 7. 安全与治理

| 控制点 | 实现 |
|--------|------|
| **数据访问** | Agent 继承用户 Entra ID 身份；用户能看的它才看（OneLake Security 行/列级 + Sensitivity Label） |
| **指令边界** | Instructions 里硬约束（如 "不回答非业务问题"）+ Content Safety 兜底 |
| **PII 防泄** | Ontology 标记 `sensitive` 的属性 Agent 默认不返回字段值 |
| **审计** | 每次问答记录到 Purview Activity Log，包含查询、生成的 SQL、返回结果 |
| **成本** | Agent 调用计 CU；可配 daily quota |

---

## 8. 与友商同类能力对比

| 能力 | Fabric Data Agents | Databricks AI/BI Genie | Snowflake Cortex Agents | M365 Copilot |
|------|--------------------|-----------------------|-------------------------|--------------|
| 自然语言→SQL/DAX | ✅ | ✅ | ✅ | ✅ |
| 业务语义层 | **Fabric IQ ★★★★★** | Unity Catalog tags | tags | M365 graph |
| 工具调用 | ✅（含报表打开、写回） | 弱 | 中 | ✅ |
| MCP 协议 | ✅ | ✅ | 部分 | ✅ |
| 跨数据源（Lake+DW+KQL） | ✅ | Lake+DW | DW 为主 | 外部源 |
| 写回数据（Translytical） | **✅ 独有** | ❌ | ❌ | ❌ |
| 与 Power BI 报表深度集成 | ✅ | 弱 | 外接 | ✅ |
| 治理（Purview） | ✅ | UC | Horizon | M365 |

**结论**：业务用户面 Data Agent → Fabric **目前最完整**（业务本体 + 写回 + Power BI 深度集成是三个独有点）。

---

## 9. 自检

完成本周后你应该能：

- [ ] 解释 Data Agent / IQ Ontology / MCP 三件套各自解决什么问题
- [ ] 创建一个 Data Agent，挂多种 grounding sources
- [ ] 定义 Ontology 把业务概念明确化
- [ ] 注册 Skill 让 Agent 调用 Power BI / 触发写回
- [ ] 用 MCP 协议把 Agent 接入外部应用
- [ ] 在客户场景下区分"Copilot vs Data Agent vs 直接写报表"该用谁

---

## 10. 推荐资源

- 官方：[Fabric Data Agents docs](https://learn.microsoft.com/fabric/data-agents/)（GA 跟踪）
- 官方：[Fabric IQ overview](https://learn.microsoft.com/fabric/fabric-iq/)
- MCP 规范：<https://modelcontextprotocol.io>
- FabCon 2026 talk：*The Agentic Era of Fabric*
- 微软示例：<https://github.com/microsoft/fabric-samples>（搜 `data-agent`）
