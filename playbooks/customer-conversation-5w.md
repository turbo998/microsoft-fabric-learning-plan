# 客户对话 5W 框架（Microsoft Fabric vs 友商）

> 当客户问"为什么不选 Databricks / Snowflake / BigQuery / Redshift？"时，
> 不要陷入"我们这功能更强"的局部对比，按 **5 个 W** 把对话往业务上拉。

---

## 1. Workloads — 你要做几种活？

**问客户**：你的数据团队当前 / 未来 12 个月要承载哪些工作负载？

| 客户回答 | 推荐路径 |
|---------|----------|
| 只做 DW + BI | **Snowflake** 或 Fabric 都行，Snowflake 引擎更成熟 |
| 只做大规模 ML / 模型训练 | **Databricks**（Mosaic AI、万 GPU） |
| DW + BI + ML + 实时 + AI Agent 一起做 | **Fabric**（一个产品全覆盖） |
| 只做实时分析 / 时序 | **Fabric KQL DB** 或 ClickHouse / Druid |

**话术**：
> "如果你只做一件事，专业产品确实更深；但每多一种 workload，
> 多产品拼接的运维 / 治理 / 集成成本就指数上涨。Fabric 在 3 种以上 workload 时优势开始显现。"

---

## 2. Workforce — 你的团队画像是？

**问客户**：你团队里数据工程师、数据分析师、数据科学家、BI 开发的比例大概是？

| 团队结构 | 推荐路径 |
|---------|---------|
| 工程师驱动（写代码 > 拖拽） | **Databricks**（Notebook + Spark） |
| 分析师驱动（SQL + Excel + Power BI） | **Fabric**（Copilot 写 SQL/DAX、低代码 Pipeline） |
| 科学家驱动（PyTorch / TensorFlow） | **Databricks** 或 **Vertex AI** |
| 混合，且 BI 是核心交付物 | **Fabric**（DirectLake 让分析师无障碍） |

**话术**：
> "Databricks 的强大需要工程能力支撑。如果你团队里 70% 是分析师 + BI 开发，
> Fabric 的 SaaS 体验 + Copilot 能让他们 day-1 就上手。"

---

## 3. Wallet — 钱怎么花最舒服？

**问客户**：你 CFO 更看重 *可预测预算* 还是 *按用量付费*？

| 财务偏好 | 推荐路径 |
|---------|---------|
| 月度预算固定、不希望惊喜账单 | **Fabric Capacity (F-SKU)** —— 一份容量包多种 workload |
| 按需用、闲时不付费 | **Snowflake**（warehouse auto-suspend）/ **Databricks** Serverless |
| 已有 Power BI Premium | **Fabric** —— 容量可以平滑升级，原投资不浪费 |
| 数据量峰谷差异大 | **Snowflake / Databricks**（弹性更细） |

**话术**：
> "Fabric 容量像办公楼包年租金，预算可控；Snowflake/Databricks 像按小时酒店，
> 闲时省钱但旺季可能爆炸。你的业务节奏更像哪种？"

---

## 4. Where — 单云还是多云？

**问客户**：你的数据现在在哪？未来 3 年的云策略？

| 现状 | 推荐路径 |
|------|---------|
| 已是 Azure 大客户 | **Fabric**（共享 Entra ID、Purview、Defender、计费）|
| 已是 AWS 大客户、要数据平台 | **Databricks on AWS** 或 **Snowflake** |
| 多云、要避免 vendor lock-in | **Databricks** 或 **Snowflake**（都支持三大云） |
| 数据在 Azure、AI 在其他云 | **Fabric** + 跨云 Shortcut / Mirroring |

**话术**：
> "Fabric 是 SaaS-only 在 Azure 上的。如果你坚定多云，Databricks / Snowflake 灵活；
> 但绝大多数企业其实是 *主云 + 试点云*，主云在 Azure 时 Fabric 永远是默认答案。"

---

## 5. What's Next — 你下一步在想什么？

**问客户**：未来 12–18 个月，你最想用数据做的新事是什么？

| 未来重点 | 推荐路径 |
|---------|---------|
| AI Agent / 数据问答 / 业务用户自助 | **Fabric Data Agents + Fabric IQ + MCP** |
| 训练自有大模型 | **Databricks Mosaic AI** |
| 数据共享 / 数据交易 | **Snowflake Marketplace** |
| 实时运营决策 + 闭环动作 | **Fabric Real-Time Intelligence + Activator** |
| 把 SAP / Oracle 数据用起来 | **Fabric Mirroring**（连接器最全） |

**话术**：
> "如果你的'下一步'是 *业务用户用自然语言问数据*，
> Fabric Data Agents + IQ 业务本体是目前最完整的方案 ——
> 不需要 fine-tune 模型，开箱即用。"

---

## 6. 反对话术 — 怎么回应常见的"Fabric 不行"

| 客户质疑 | 回应方向 |
|---------|---------|
| "Fabric 不如 Databricks Spark 性能" | "对于 TB 级以下 + 中等并发，差距不显著；TB 以上 + 复杂 ML 才选 Databricks。" |
| "Fabric 不如 Snowflake 成熟" | "DW 引擎确实不如，但 Fabric DW + Mirroring Snowflake 是常见架构，让 Snowflake 当 system of record。" |
| "Fabric 锁死 Azure" | "你的 Power BI / M365 / Entra 已经锁死 Azure 了。Fabric 是把已锁的资产价值最大化。" |
| "容量计费贵" | "对 < 100 个用户、< 5 TB 数据是贵；对 1000 用户 + 50 TB 反而便宜，因为不按席位计。" |
| "新产品风险大" | "Fabric 是 Power BI Premium + Synapse 重新打包，**底层已经跑了 5+ 年**，不是从零开始。" |
| "中文文档少" | "MS Learn 中文有 70%+ 覆盖；社区博客以英文为主，但 Copilot 可中文交互。" |

---

## 7. 不该卖 Fabric 的场景（诚实分类）

承认 Fabric **不适合**的场景反而能赢得信任：

- ❌ 客户坚定多云策略且每朵云都有重 workload
- ❌ 已深度投资 Databricks Unity Catalog 与 Mosaic AI，且团队是工程师
- ❌ 业务核心是 PB 级 ML 训练 + 大量 GPU
- ❌ 客户已经签了 5 年 Snowflake 大单且数据共享是核心场景
- ❌ 完全离线 / 边缘部署需求（Fabric 是 SaaS）

**话术**：
> "Fabric 不是银弹。在你的 X 场景下我建议你 Y。但你的 Z 场景，
> Fabric 仍然是最佳补充 —— 我们可以共存而不是替代。"

诚实推荐反而能拿到下一个项目。

---

## 8. 三句话收尾

任何选型对话最后用这三句话收：

1. "**最低摩擦** —— 你已经在用的 Power BI / M365 / Entra 就是 Fabric 的底座，部署 0 摩擦。"
2. "**最快价值** —— Mirroring + DirectLake + Copilot 三件套，从有数据到出报表最快 1 天。"
3. "**最长保值** —— Fabric 是 Microsoft 数据 + AI 战略的核心载体，未来 5 年投入只增不减。"
