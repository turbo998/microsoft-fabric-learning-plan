# Fabric vs 友商：详细对比矩阵（2026）

> 涵盖：Microsoft Fabric · Databricks · Snowflake · AWS (Redshift/EMR/Glue/QuickSight) · GCP BigQuery
>
> 数据更新基准：**FabCon 2026（March）** + Snowflake Summit 2025 + Databricks Data+AI Summit 2025

---

## 1. 形态与架构

| 维度 | Microsoft Fabric | Databricks | Snowflake | AWS 数据栈 | GCP BigQuery |
|------|------------------|------------|-----------|-----------|--------------|
| **产品形态** | SaaS（仅 Azure） | PaaS/SaaS（多云） | SaaS（多云） | 5–8 个产品拼接 | SaaS（仅 GCP） |
| **底层存储** | OneLake（Delta 单一） | DBFS + Delta + Iceberg | 内置专有 + Iceberg | S3 + Redshift Managed | BigQuery Storage + Iceberg |
| **开放格式** | Delta（开源） | Delta + Iceberg | Iceberg + 专有 | Iceberg + 专有 | Iceberg + 专有 |
| **计算单位** | Capacity Units（F-SKU） | DBU（Photon/Standard） | Credits（Warehouse 大小） | 多种（RPU/DPU/...） | Slots / On-demand |
| **基础架构管理** | 0（SaaS） | 部分（cluster 配置） | 0（SaaS） | 全部（IaC） | 0（SaaS） |
| **多云** | ❌ | ✅（三大云） | ✅（三大云） | ❌ | ❌ |

---

## 2. 各 Workload 深度对比

### 2.1 Data Warehouse（结构化 DW）

| 维度 | Fabric DW | Databricks SQL | Snowflake |
|------|-----------|----------------|-----------|
| 引擎成熟度 | ★★★（Polaris，新） | ★★★★（Photon，几年） | ★★★★★（行业标杆） |
| T-SQL / ANSI SQL 兼容 | T-SQL（受限） | ANSI SQL | ANSI SQL |
| 并发性能 | 中 | 高 | 极高（multi-cluster warehouse） |
| Time Travel | ✅ | ✅（Delta） | ✅（专有，最强） |
| Zero-Copy Clone | ✅ | ✅（Delta shallow） | ✅（行业首创） |
| MERGE 性能 | 中 | 高 | 高 |
| 数据共享 | ❌（用 Shortcut） | Delta Sharing | **Marketplace + Sharing** ★★★★★ |

**结论**：纯 DW → Snowflake；Lake+DW 一体 → Fabric / Databricks。

### 2.2 Data Engineering / Lakehouse

| 维度 | Fabric Lakehouse | Databricks |
|------|------------------|-----------|
| Spark 版本 / 性能 | 3.5+，集成 | 3.5+ + Photon（更快） |
| Notebook 体验 | ★★★★（Copilot 强） | ★★★★★（最成熟） |
| Delta Live Tables 类能力 | ❌（用 Pipeline / Dataflow） | ✅ DLT（最强声明式） |
| Auto Loader | ❌ | ✅ |
| Streaming | Eventstream + KQL | Structured Streaming + DLT |
| 多语言（Scala/R/SQL/Py） | 主要 PySpark + SQL | 四种全支持 |
| Job 调度 | Pipeline + Schedule | Workflows |

**结论**：纯 Lakehouse 工程深度 → Databricks；要和 BI/AI 一体化 → Fabric。

### 2.3 ML / AI

| 维度 | Fabric | Databricks | Snowflake | GCP |
|------|--------|-----------|-----------|------|
| 实验跟踪 | MLflow（嵌入） | MLflow（原生最强） | Cortex | Vertex AI |
| Feature Store | ❌（roadmap） | ✅（最成熟） | ✅ | ✅ |
| Model Serving | Online endpoint | **Mosaic AI Serving**（最强） | Snowpark Container | Vertex AI |
| 自动 ML | Copilot 辅助 | AutoML | Snowflake AutoML | Vertex AI AutoML |
| LLM 训练 / fine-tune | Azure ML（外挂） | **Mosaic AI**（最强，独立产品） | Cortex（推理为主） | Vertex AI |
| RAG / Agent 框架 | **Data Agents + Fabric IQ + MCP** | Agent Bricks | Cortex Agents | Vertex AI Agent |
| GPU 集群 | Azure ML（外挂） | ✅（一体） | ❌（外挂） | ✅（一体） |

**结论**：大规模 ML 训练 → Databricks；业务面 AI Agent → Fabric；纯推理 → 看场景。

### 2.4 实时分析 / Streaming

| 维度 | Fabric RTI | Databricks | Snowflake | AWS |
|------|-----------|-----------|-----------|------|
| 流处理引擎 | Eventstream（轻量） + KQL DB | Structured Streaming + DLT | Snowpipe Streaming（追加） | Kinesis + MSK |
| 时序查询性能 | **KQL（亚秒级）** ★★★★★ | Spark SQL（一般） | SQL（一般） | Timestream（专用） |
| 复杂事件处理 | KQL `evaluate`、时序函数 | Spark + 自研 | 弱 | Flink on KDA |
| 告警 / 触发动作 | **Activator（原生）** | 外挂（Webhook） | 外挂 | EventBridge |
| 端到端延迟 | < 5 秒 | 几十秒 | 1 分钟级 | 5–30 秒 |

**结论**：实时分析 → **Fabric RTI 是 2026 最被低估的能力**。

### 2.5 BI / 可视化

| 维度 | Fabric (Power BI) | Databricks SQL | Snowflake + Tableau | BigQuery + Looker |
|------|-------------------|----------------|---------------------|-------------------|
| 原生 BI 引擎 | **Power BI** ★★★★★ | Dashboards（基本） | 无（外接） | Looker（强） |
| **DirectLake 模式** | ✅ **独有** | ❌ | ❌ | ❌ |
| Import 模式性能 | 极强（VertiPaq） | — | — | — |
| 自助分析 | 极强 | 中 | 强 | 强 |
| Copilot 写 DAX | ✅ | ❌ | ❌ | ❌ |
| 写回数据库 | **Translytical Flows**（新） | ❌ | ❌ | ❌ |

**结论**：BI 体验 → Fabric **降维打击**，DirectLake 是其他家暂时没有的概念。

---

## 3. 治理 / 安全 / 合规

| 维度 | Fabric | Databricks | Snowflake | AWS |
|------|--------|-----------|-----------|------|
| 元数据治理 | Purview（原生） | Unity Catalog | Horizon | Lake Formation + Glue |
| Lineage 自动捕获 | ✅ | ✅ | ✅ | 弱 |
| Sensitivity Labels | **M365 端到端贯通** ★★★★★ | ❌ | 标签（自建） | 弱 |
| 行级 / 列级安全 | ✅ | ✅ | ✅ | ✅ |
| 数据脱敏 | ✅ | ✅ | ✅ | ✅ |
| 加密客户密钥 | Customer Key | CMK | CMEK | KMS |
| 合规认证（SOC/ISO/HIPAA） | 全覆盖 | 全覆盖 | 全覆盖 | 全覆盖 |
| 审计日志 | M365 统一 | 自管 | 自管 | CloudTrail |

**结论**：金融 / 政府 / 医疗强合规 → Fabric 凭 Purview + M365 标签体系优势明显。

---

## 4. 零 ETL 集成（2026 关键能力）

| 维度 | Fabric | Databricks | Snowflake | AWS |
|------|--------|-----------|-----------|------|
| **Mirroring 源** | Snowflake / Cosmos DB / Azure SQL / PostgreSQL / MySQL / **SAP / Oracle / SharePoint / Dremio / Azure Monitor** | Lakehouse Federation（查询联邦） | Iceberg Tables（被动） | Zero-ETL（仅 Aurora→Redshift） |
| **CDC 实时同步** | ✅（绝大多数源） | ❌ | ❌ | 部分 |
| **存储费用** | **0**（指向源） | 0（联邦） | 视源 | 视源 |
| **计算费用** | 仅查时付 CU | 仅查时付 DBU | 仅查时付 Credits | 仅查时付 |
| **Shortcut（指向外部存储）** | ✅ ADLS/S3/GCS/Dataverse | 联邦表 | Iceberg External | 较弱 |

**结论**：**Fabric Mirroring 在 2026 是同类最强**，是把"老数据栈"快速变成"AI Ready"的最低摩擦路径。

---

## 5. AI / Copilot 集成深度

| 维度 | Fabric | Databricks | Snowflake | AWS | GCP |
|------|--------|-----------|-----------|------|------|
| Notebook 内 Copilot | ✅（深） | ✅ AI Assistant | Cortex Copilot | Q Developer | Duet AI |
| 自然语言 → SQL | ✅ 各 workload | ✅ Genie | ✅ Cortex | Q for QuickSight | BigQuery DataCanvas |
| 自然语言 → DAX | ✅ 独有 | ❌ | ❌ | ❌ | ❌ |
| 业务语义层 / Ontology | **Fabric IQ** ★★★★★ | Unity Catalog 标签 | 标签 | 弱 | Looker 语义层 |
| Data Agent 框架 | ✅（Power BI 报表内可调） | Agent Bricks | Cortex Agents | Bedrock Agents | Vertex AI Agent |
| MCP 协议支持 | ✅ | ✅ | 部分 | ✅ | 部分 |
| 业务用户自助问答 | ★★★★★ | ★★★ | ★★★ | ★★★ | ★★★★ |

**结论**：AI 工程深度 → Databricks / Vertex；业务用户面 AI 体验 → **Fabric 最完整**。

---

## 6. 总拥有成本（TCO）参考

> 仅作量级参考。真实成本因 region / 折扣 / 用量分布差异很大。

| 场景 | Fabric | Databricks | Snowflake |
|------|--------|-----------|-----------|
| 中小企业（10 用户、1 TB、轻 BI） | **F2 ≈ $260/月** | Std ≈ $400+/月 | $500+/月 |
| 中型企业（100 用户、20 TB、含 ML） | F32 ≈ $4K/月 | $8–15K/月 | $10–20K/月 |
| 大型企业（500 用户、200 TB、全 workload） | F128 ≈ $16K/月 | $30–60K/月 | $40–80K/月 |
| BI 用户许可（已包含在容量） | 0（容量内） | 不含 BI | 不含 BI |

**结论**：
- 中小 + BI 重 → Fabric 显著便宜
- 大数据 + ML 训练 → Databricks 优化空间大
- 纯 DW 查询 → Snowflake 单位查询便宜，但用户许可外购贵

---

## 7. 何时该选谁 —— 决策树

```
你的核心 workload 是什么？
├─ 只做 DW + BI
│  ├─ 微软栈重 → Fabric
│  └─ 不限 → Snowflake
│
├─ 做大规模 ML / 训练大模型
│  └─ Databricks（Mosaic AI）
│
├─ 做实时分析 + 闭环动作
│  └─ Fabric RTI（亚秒级，Activator 闭环）
│
├─ 做 AI Agent / 业务用户自然语言查询
│  └─ Fabric Data Agents + Fabric IQ
│
├─ DW + Lake + ML + BI + 实时 全都要
│  ├─ 已在 Azure / 微软栈 → Fabric
│  ├─ 已在 AWS / GCP → Databricks
│  └─ 多云策略 → Databricks 或 Snowflake + Databricks
│
└─ 已经用了 Snowflake / Oracle / SAP，想加 AI
   └─ Fabric Mirroring 不搬数据（最低风险）
```

---

## 8. 共存架构（最常见的真实部署）

实际项目里**单一选型很少见**。常见共存模式：

### 模式 A：Snowflake 当 System of Record + Fabric 当 AI/BI 层

```
SAP/Oracle/Sales → Snowflake（核心 DW）
                       ↓ Mirroring（零 ETL）
                  OneLake
                       ↓ DirectLake
                  Power BI + Data Agents
```

### 模式 B：Databricks 当 ML 工厂 + Fabric 当业务交付

```
原始数据 → Databricks（DLT 管道 + 模型训练）
              ↓ Delta Sharing / Shortcut
           OneLake
              ↓
           Fabric Warehouse / Power BI / Data Agents（业务交付）
```

### 模式 C：Fabric 当统一 + Databricks 当重 ML 外挂

```
所有数据 → Fabric（OneLake + Pipeline + Lakehouse + Power BI）
              ↓ Shortcut（双向）
           Databricks（仅做大模型训练、特定算法）
              ↓ 模型回写
           Fabric Model Serving
```

---

## 9. 一图流总结

| 你的关键词 | 首选 |
|----------|------|
| Power BI、M365、Entra、Purview | **Fabric** |
| Spark、MLflow、Photon、Notebook | **Databricks** |
| 数据共享、Marketplace、多云 DW | **Snowflake** |
| 多云、AWS 重投入 | **Databricks / Snowflake** |
| 业务用户自助 + AI Agent | **Fabric** |
| 时序、日志、亚秒查询 | **Fabric KQL DB** |
| Mirroring 零 ETL | **Fabric**（同类最强） |
| 大规模 LLM 训练 | **Databricks Mosaic AI** |
| BigQuery 已是核心 | **GCP 继续 BigQuery** |
| 完全离线 / 边缘 | **都不行**（找开源 Delta / Iceberg + Trino） |
