# Microsoft Fabric 深度学习计划（2026 版）

> 适用人群：有云数据平台基础（Azure / AWS / GCP）、了解 Spark / SQL / Python 任一，希望系统掌握 Microsoft Fabric 端到端能力的数据/AI 工程师、架构师、售前。
>
> 预期投入：**8 周 × 8–10 小时/周** ≈ 70 小时。可压缩为 4 周强化版（双倍投入）。

---

## 0. 学习目标（结业后你能做到）

- ✅ 准确说清 **Fabric = OneLake + 6 大 workload + Copilot/Agents** 这套架构，能在白板上画出
- ✅ 在 Fabric 工作区里独立搭建 **Medallion（Bronze→Silver→Gold）** 数据管道
- ✅ 熟练使用 **Lakehouse、Warehouse、KQL DB、Notebook、Pipeline、Eventstream、Power BI 语义模型** 七大核心组件
- ✅ 用 **Mirroring + Shortcuts** 实现零 ETL 集成（含 Snowflake / Cosmos DB / Azure SQL）
- ✅ 用 **DirectLake** 模式让 Power BI 直读 OneLake，不走 Import 也不走 DirectQuery
- ✅ 用 **Copilot in Fabric / Data Agents / Fabric IQ** 做自然语言查询和 AI 应用
- ✅ 能在客户/选型场景下，**清晰对比 Fabric vs Databricks vs Snowflake**，说出每种场景该选谁
- ✅ 完成 **DP-600 / DP-700 认证**（可选但强烈推荐）

---

## 1. 学习前置（Week 0，1–2 天）

### 1.1 必备背景

| 知识 | 程度 | 不熟需要补 |
|------|------|-----------|
| SQL（窗口函数、CTE） | 中等 | SQLBolt / Mode SQL Tutorial |
| Python | 入门 | 能跑通 pandas / pyspark 基本操作 |
| Spark 概念 | 大致了解 | RDD / DataFrame / Lazy Eval |
| Delta Lake 格式 | 大致了解 | Delta Lake 官方 5 分钟介绍 |
| Power BI / DAX | 入门 | Microsoft Learn 的 PL-300 前两章 |
| Azure 基础 | 入门 | Resource Group、订阅、RBAC |

### 1.2 准备环境

1. 注册 **Microsoft Fabric 60 天免费试用**：<https://learn.microsoft.com/fabric/fundamentals/fabric-trial>
2. 申请 **Microsoft 365 Developer Program**（可白嫖 E5 + Power BI Pro）：<https://developer.microsoft.com/microsoft-365/dev-program>
3. 准备一个 **Azure 订阅**（Pay-as-you-go 或 MSDN）—— 配合 Mirroring / Eventstream 需要源端 Azure SQL/Cosmos DB
4. 安装：**VS Code + Polyglot Notebooks 扩展**、**Power BI Desktop**、**Azure Data Studio**（可选）

---

## 2. 八周学习路径

### Week 1 — Fabric 基础与 OneLake

**目标**：理解 Fabric 是什么、它的 SaaS 定位、OneLake 为什么是核心。

**学习材料**：
- 📺 Microsoft Mechanics: *Microsoft Fabric in 8 minutes* — <https://youtu.be/...>
- 📚 MS Learn 学习路径：**Get started with Microsoft Fabric**（4 小时）
  <https://learn.microsoft.com/training/paths/get-started-fabric/>
- 📖 官方文档：*OneLake overview* 与 *Architecture and components* 章节
- 📰 James Serra 博客的 Fabric 系列（业内最权威的中立视角）

**动手实验**：
1. 创建工作区（Workspace），分配 F2/F4 容量（用试用版）
2. 创建第一个 **Lakehouse**，上传一个 CSV，自动转 Delta 表
3. 用 **OneLake File Explorer**（Windows 桌面版）像访问 OneDrive 一样浏览 Lakehouse
4. 创建一个 **Shortcut** 指向 ADLS Gen2 / S3 / Google Cloud Storage 中的数据

**里程碑**：能用一张图解释 OneLake 与 ADLS Gen2 / S3 / Snowflake Storage 的差异。

---

### Week 2 — Data Engineering（Lakehouse + Notebook + Spark）

**目标**：用 Fabric 的 Notebook 做 ETL，构建 Medallion 架构。

**学习材料**：
- MS Learn: **Ingest data with Microsoft Fabric**、**Use Apache Spark in Microsoft Fabric**
- 官方示例数据集：**NYC Taxi**、**Wide World Importers**
- 视频：**Guy in a Cube** 的 Fabric Notebook 系列

**动手实验**：
1. 从公开数据集（NYC Taxi Parquet）通过 **Pipeline → Copy Activity** 落地到 Bronze
2. 用 **PySpark Notebook** 做清洗、去重、类型转换 → Silver
3. 写一个 **聚合 + 维度建模**的 Notebook → Gold（事实表 + 维表）
4. 启用 **V-Order + Delta optimization**，对比查询性能
5. 用 **Notebook Resources** 上传自定义 lib（whl 包）

**里程碑**：交付一个完整 Medallion Lakehouse，能在 PySpark / Spark SQL 两种语法间切换。

---

### Week 3 — Data Warehouse + SQL Endpoint

**目标**：理解 Fabric DW（事务型 T-SQL）与 Lakehouse 的 SQL Endpoint（只读）的差异，选择正确组件。

**学习材料**：
- MS Learn: **Get started with data warehouses in Microsoft Fabric**
- 官方文档：*Data Warehouse* 章节，重点看 **Workload management**、**Concurrency**、**MERGE/INSERT/UPDATE 限制**
- 博客：**Fabric DW vs Synapse Dedicated SQL Pool** 迁移对比

**动手实验**：
1. 创建一个 **Warehouse**，建立星型模型（fact + dim）
2. 用 **CETAS** 把 Warehouse 表导出到 OneLake parquet
3. 用 **跨数据库查询** 在 Warehouse 里 JOIN Lakehouse 的 Delta 表（同一个 OneLake，零拷贝）
4. 测试 **Time Travel / Zero-copy clone** 功能
5. 配置 **Workload management**：把高优队列与低优队列隔离

**里程碑**：清楚说出"什么时候选 Warehouse、什么时候选 Lakehouse"。

---

### Week 4 — Real-Time Intelligence（Eventstream + KQL DB + Activator）

**目标**：搞清 Fabric 的实时数据栈，理解 KQL / Kusto 的查询模型。

**学习材料**：
- MS Learn: **Get started with Real-Time Intelligence in Microsoft Fabric**
- KQL 速成：**Kusto Detective Agency**（游戏化学习）<https://detective.kusto.io>
- 官方文档：*Eventstream*、*Eventhouse*、*KQL Database*、*Activator* 四章

**动手实验**：
1. 用 **Eventstream** 接一个真实源：Azure Event Hub / IoT Hub / Sample data
2. 在 **KQL Database** 里写查询：`summarize`、`mv-expand`、`evaluate`、**时序函数**
3. 创建 **Real-Time Dashboard**（KQL 驱动）
4. 配置 **Activator** 规则：当某指标超阈值 → Teams 通知 / 触发 Pipeline
5. 用 **Mirroring（新）**：把 Azure SQL CDC 流接进 Eventstream

**里程碑**：能解释"为什么有些场景 KQL 比 Spark SQL 快 100 倍"。

---

### Week 5 — Power BI + DirectLake（这是 Fabric 最大杀手锏）

**目标**：理解 DirectLake 的革命性意义 —— BI 直接读 OneLake Delta，**既不导入也不 DirectQuery**。

**学习材料**：
- 微软白皮书：**DirectLake mode explained**
- SQLBI（Marco Russo / Alberto Ferrari）的 DirectLake 深度文章 ★必读★
- MS Learn: **Get started with Power BI in Microsoft Fabric**

**动手实验**：
1. 在 Week 2 的 Gold 层上，**自动**生成一个**默认语义模型**
2. 创建一个 **自定义语义模型**，用 DirectLake 模式连 Gold 表
3. 写 5 个 DAX 指标：YoY、MoM、累计、占比、Top N
4. 对比 **DirectLake vs Import vs DirectQuery** 的刷新延迟与查询性能
5. 启用 **Translytical Task Flows**（FabCon 2026 新特性）—— Power BI 报表直接写回数据库

**里程碑**：能用一句话说清 DirectLake 相对 Snowflake + Tableau / BigQuery + Looker 的核心优势。

---

### Week 6 — 零 ETL：Mirroring & Shortcuts（最能体现 Fabric 优势的功能）

**目标**：掌握 Fabric 的"反 ETL"哲学 —— 把数据**留在原地**，但让 Fabric 像本地一样查。

**学习材料**：
- 官方文档：*Mirroring* 全部章节
- 博客：*Why Mirroring matters: Snowflake to Fabric without ETL*
- FabCon 2026 主题演讲（Mirroring 扩展到 SAP / Oracle / SharePoint / Dremio）

**动手实验**：
1. **Mirror Snowflake**：把 Snowflake 表镜像到 OneLake（自动 CDC、零计算费用、几乎实时）
2. **Mirror Azure SQL DB**：开启 CDC 后镜像同步
3. **Mirror Cosmos DB**：NoSQL 也能做 Mirror
4. 镜像后立即用 **DirectLake** 在 Power BI 出报表（无需 ETL）
5. 用 **Shortcut Transformations**（新）—— Shortcut 落地时自动转 Delta + AI 转换（摘要、翻译）

**里程碑**：在白板上能画"Snowflake/Oracle 数据 0 ETL 进 Fabric → Power BI"的端到端架构。

---

### Week 7 — AI / Copilot / Data Agents（2026 年最重点）

**目标**：理解 Fabric 是怎么从"数据平台"演进为"智能层"的。

**学习材料**：
- 官方：**Copilot in Fabric** 各 workload 章节
- 官方：**Fabric Data Agents** 与 **Fabric IQ ontology**
- FabCon 2026 keynote：*Agentic era of Fabric*

**动手实验**：
1. 在 Notebook 里用 **Copilot** 生成 PySpark / SQL（自然语言 → 代码）
2. 在 Power BI 里用 **Copilot** 生成 DAX、报表页面、Q&A
3. 在 KQL Database 里用 **Copilot** 生成 KQL 查询
4. 创建一个 **Data Agent**：把 Lakehouse / Warehouse / 语义模型挂为 grounding 源
5. 用 **MCP** 协议把 Data Agent 接到 Copilot Studio / 自建 LLM 应用
6. 用 **Fabric IQ** 定义业务本体（订单、客户、产品的关系语义），让 Agent 用业务语义推理

**里程碑**：你的同事能用自然语言"今年华东区咖啡机退货率"问 Data Agent，并拿到准确答案 + 源引用。

---

### Week 8 — 治理、CI/CD、容量管理 + Capstone 实战

**目标**：让 Fabric 能进生产 —— 权限、敏感度、Git、生命周期、容量监控。

**学习材料**：
- MS Learn: **Administer Microsoft Fabric**
- 官方：*Workspace Git integration*、*Deployment Pipelines*、*Capacity metrics app*
- Microsoft Purview 与 Fabric 的集成（Sensitivity Labels、Lineage、DLP）

**动手实验**：
1. 给 Workspace 接上 **GitHub / Azure DevOps**，把 Notebook / 语义模型 / Pipeline 全部 git 化
2. 配置 **Deployment Pipeline**：Dev → Test → Prod 三环境
3. 在 Workspace 启用 **Sensitivity Labels**，给 Gold 表打 *Confidential*
4. 安装 **Capacity Metrics App**，分析谁吃了 CU、是否触发 throttling
5. 配置 **OneLake security**（行级 / 列级安全）

**Capstone 项目**（用 5–8 小时整合所有所学）：

> **跨境电商 360 度客户视图**
>
> 数据源：
> - Snowflake（订单事实表）→ Mirror
> - Azure SQL（客户主数据）→ Mirror  
> - Eventstream（实时点击流）→ KQL DB
> - 公开数据集（关税、汇率）→ Lakehouse Shortcut
>
> 产出：
> - Medallion Lakehouse + Warehouse 星型模型
> - Power BI DirectLake 仪表板（客户全景 + 实时点击）
> - Data Agent：自然语言查询客户画像
> - Activator 告警：高价值客户首次投诉自动通知 CSM

完成 Capstone 后，**录一段 5 分钟的演示视频**，作为路演资产。

---

## 3. 能体现 Fabric 独特优势的场景（选型时讲故事用）

下面 8 个场景，是**最能让 Fabric 胜出**的典型，选型谈话时按这些角度切入。

### S1. Microsoft 全家桶 + Power BI 重度用户
**Fabric 优势**：原生集成 Entra ID、M365、Teams、Purview、Sensitivity Labels；Power BI Premium 容量天然就是 Fabric 容量；DirectLake 让 BI 无延迟。  
**对比**：Snowflake / Databricks 要拼接 Tableau / Power BI，治理要外挂。

### S2. 已经在 Snowflake / BigQuery / Oracle 上，想加 AI 不想搬数据
**Fabric 优势**：**Mirroring** 零 ETL 把别家数据镜像进 OneLake，立刻可以用 Copilot / Data Agents / Power BI，**不收存储和计算费**，只查的时候付 CU。  
**对比**：Databricks Lakehouse Federation 类似但更偏 Spark 计算；Snowflake 没有等价能力。

### S3. 分析师驱动的中型企业，IT 团队薄弱
**Fabric 优势**：SaaS 一键开通，无 cluster / warehouse 容量预热；Copilot 写 SQL/DAX；自助分析门槛低。  
**对比**：Databricks 工程化心智重；Snowflake 简单但缺 BI / ML 一体化。

### S4. 实时运营 + 历史分析同时要
**Fabric 优势**：Eventstream + KQL DB + Lakehouse + Power BI 在一个产品里，KQL 在亚秒级查询和时序聚合上是降维打击；Activator 闭环触发动作。  
**对比**：要在 Databricks（Spark Streaming）+ Snowflake（Snowpipe）+ Kafka + Tableau 之间拼接。

### S5. 大量结构化 + 半结构化 SaaS 数据（SAP / Salesforce / SharePoint / D365）
**Fabric 优势**：FabCon 2026 后 Mirroring 覆盖了 SAP / Oracle / SharePoint / Dremio，连接器最全；与 Microsoft 365 Copilot 共享 grounding 数据。  
**对比**：Snowflake 要靠 Fivetran；Databricks 要写 Spark 连接器或 Lakehouse Federation。

### S6. 监管严格的金融 / 政府 / 医疗（合规与可解释优先）
**Fabric 优势**：Purview 原生治理、敏感度标签端到端贯通、Data Lineage 自动捕获、行/列级安全、审计日志统一。  
**对比**：Databricks Unity Catalog 强但需要规划；Snowflake 治理强但 BI/AI 链路较长。

### S7. 业务部门要做 AI Agent / 数据问答
**Fabric 优势**：Data Agents + Fabric IQ 业务本体 + MCP 协议三件套，把语义模型变成"会回答业务问题的 Agent"，且引用源数据可追溯。  
**对比**：Snowflake Cortex 偏 LLM 包装，缺业务本体；Databricks Agent Bricks 偏向 Spark/MLflow 工程师。

### S8. 中等规模、要"一站式"省心
**Fabric 优势**：容量计费（F-SKU）一笔账搞定 6 大 workload；自动扩缩 + Burst；不用关心 Spark cluster / warehouse warm-up。  
**对比**：Snowflake 多 warehouse 各自计费；Databricks 多 cluster 容易失控。

---

## 4. 友商对比矩阵（深度版）

### 4.1 总览

| 维度 | Microsoft Fabric | Databricks | Snowflake | AWS（Redshift+EMR+QuickSight） | GCP BigQuery |
|------|------------------|------------|-----------|--------------------------------|--------------|
| **形态** | SaaS（仅 Azure） | PaaS/SaaS（多云） | SaaS（多云） | PaaS 拼装 | SaaS（仅 GCP） |
| **存储** | OneLake（Delta 单一格式） | DBFS + Delta（开放） | 内置专有 + Iceberg | S3 + Redshift Managed | BigQuery Storage |
| **计算单位** | Capacity Units (F-SKU) | DBU (Photon/Standard) | Credits (Warehouse) | RPU / DPU 等多种 | Slots / On-demand |
| **BI 集成** | Power BI 原生 + DirectLake | Databricks SQL（弱 BI） | 外接 Tableau / Power BI | QuickSight（一般） | Looker / Studio |
| **ML / AI** | Data Science + Copilot + Data Agents | **MLflow + Mosaic AI（最强）** | Cortex（追赶中） | SageMaker（外挂） | Vertex AI（强） |
| **实时流** | Eventstream + KQL（强） | Structured Streaming + DLT | Snowpipe Streaming（弱） | Kinesis + MSK 拼接 | Pub/Sub + Dataflow |
| **治理** | Purview 原生 + OneLake security | Unity Catalog（强） | 原生 + Horizon | Lake Formation + IAM | Dataplex |
| **零 ETL 集成** | **Mirroring（最强）** | Lakehouse Federation | Iceberg Tables（弱） | Zero-ETL (Aurora→Redshift) | Datastream / Storage Read API |
| **开放格式** | Delta（开源） | Delta + Iceberg | Iceberg + 专有 | Iceberg + 专有 | Iceberg + 专有 |
| **多云** | ❌（仅 Azure） | ✅（AWS/Azure/GCP） | ✅（三大云） | ❌（仅 AWS） | ❌（仅 GCP） |
| **Copilot 深度** | **★★★★★ 各 workload 嵌入** | ★★★（AI Assistant） | ★★（Cortex Copilot） | ★★（Q for QuickSight） | ★★★ |
| **入门难度** | **低** | 中 | 低 | 高 | 中 |
| **企业级运维** | 中 | 高 | 中 | 高 | 中 |

### 4.2 Fabric 相对各家的差异化优势

#### vs Databricks
| Fabric 胜在 | Databricks 胜在 |
|-------------|----------------|
| SaaS 即开即用，无 cluster 心智 | Spark / MLflow / Photon 工程深度 |
| Power BI 原生 + DirectLake 性能 | 真正大规模 ML/AI workload（万 GPU 训练） |
| Mirroring 零 ETL 集成更全面 | 多云部署灵活 |
| 与 M365 / Teams / Entra 一体化 | Unity Catalog 治理更成熟 |
| 单一容量计费简单 | 按需付费成本更可控（中等规模） |
| Copilot / Data Agents / Fabric IQ 业务面更友好 | Agent Bricks / Mosaic AI 工程深度更高 |

**结论**：业务驱动 / 微软栈 / BI 重 → Fabric；工程驱动 / 多云 / ML 重 → Databricks。

#### vs Snowflake
| Fabric 胜在 | Snowflake 胜在 |
|-------------|----------------|
| 一个产品覆盖 ETL/DW/Lake/ML/RTI/BI | DW 引擎最成熟、并发性能领先 |
| Mirroring 让 Snowflake 数据零 ETL 进来 | Data Sharing / Marketplace 生态强 |
| Power BI DirectLake 体验最佳 | 多云部署成熟 |
| 实时 KQL 引擎 | Snowpark 在 Python 开发体验上追赶迅速 |
| Copilot 嵌入更深 | Time Travel / Zero-Copy Clone 等数据特性久经验证 |

**结论**：BI/AI 一体化 → Fabric；纯 DW + 数据共享 → Snowflake。两者**经常共存**（Snowflake 当 system of record，Fabric 做 BI/AI 层）。

#### vs AWS (Redshift / EMR / Athena / QuickSight)
| Fabric 胜在 | AWS 胜在 |
|-------------|---------|
| 单一产品 vs 5–8 个产品拼接 | 生态最全、与 AWS 服务深度集成 |
| 治理统一、Purview 原生 | IAM 颗粒度最细 |
| Copilot 深度集成 | SageMaker 在 AWS 生态内 ML 领先 |
| Power BI 强于 QuickSight | 多 region / availability zone |

**结论**：客户重 AWS → Redshift Serverless + Glue；微软栈或要简化 → Fabric。

#### vs GCP BigQuery
| Fabric 胜在 | BigQuery 胜在 |
|-------------|--------------|
| BI + ML + 实时一体化 | 大表 SQL 性能与按 GB 扫描计费灵活 |
| 多源 Mirroring 零 ETL | BigQuery Omni 多云查询 |
| Power BI 体验 | Looker / Looker Studio 集成 |
| 治理与 M365 集成 | Vertex AI 深度集成 |

**结论**：客户在 GCP → BigQuery；客户跨 Azure/M365 → Fabric。

### 4.3 一个常用的客户对话框架

被客户问"为什么不选 X？"时，按 **5 个 W** 结构回应：

1. **Workloads**：你需要几种？1–2 种（纯 DW / 纯 ML）选专业产品；3+ 种 Fabric 更划算
2. **Workforce**：团队是工程师重 → Databricks；分析师 + BI 重 → Fabric
3. **Wallet**：要可预测预算 → Fabric 容量；要按需弹性 → Snowflake / Databricks
4. **Where**：单云 / Azure 优先 → Fabric；多云 → Databricks / Snowflake
5. **What's next**：要 Agentic AI / 业务面 AI → Fabric；要重训练 / 大模型 → Databricks

---

## 5. 推荐资源汇总

### 5.1 官方
- **MS Learn 学习路径**：<https://learn.microsoft.com/training/browse/?products=fabric>
- **官方文档**：<https://learn.microsoft.com/fabric>
- **Fabric 社区**：<https://community.fabric.microsoft.com>
- **Fabric Roadmap**：<https://aka.ms/fabricroadmap>
- **What's new in Fabric**（月度）：<https://learn.microsoft.com/fabric/whats-new>

### 5.2 视频频道
- **Microsoft Mechanics**（产品介绍）
- **Guy in a Cube**（Power BI + Fabric，最实用）
- **Advancing Analytics**（深度技术，含对比 Databricks）
- **SQLBI**（DirectLake / DAX 顶尖）
- **Adam Saxton / Patrick LeBlanc**（Power BI 王炸组合）

### 5.3 博客
- **James Serra**（前微软 / Databricks，业内最权威中立视角）
- **Chris Webb's BI Blog**（DAX / 语义模型必读）
- **Microsoft Fabric Blog**：<https://blog.fabric.microsoft.com>
- **Synapx / Bowdark / Kanerika**（FabCon 总结）

### 5.4 书籍
- *Microsoft Fabric: A Beginner's Guide*（Packt, 2024）
- *Pro Microsoft Fabric*（Apress, 2024）
- *Definitive Guide to DAX*（SQLBI，DirectLake 部分 2024 更新）

### 5.5 认证
- **DP-600** — Implementing Analytics Solutions Using Microsoft Fabric（核心）
- **DP-700** — Implementing Data Engineering Solutions Using Microsoft Fabric（工程深度）
- **PL-300** — Power BI Data Analyst（前置或并行）

### 5.6 实战代码与示例
- **microsoft/fabric-samples**（GitHub 官方仓库）：<https://github.com/microsoft/fabric-samples>
- **Fabric Toolbox**：社区工具集合
- **fabric-cicd**（微软官方 CI/CD 工具）：<https://github.com/microsoft/fabric-cicd>

### 5.7 中文资源
- 微软中国官方 Tech Community
- 极客时间 / 51CTO 的 Fabric 系列（注意时效性，2024 前内容很多过时）
- 知乎 *Microsoft Fabric* 话题（PMC / James Serra 中文搬运多）

---

## 6. 8 周后下一步

完成本计划后建议：

1. **考一个认证**（DP-600 或 DP-700）—— 简历加分 + 强迫系统复习
2. **做一个公开项目**（GitHub）—— 比如本仓库的客服 demo 升级版，加 Fabric 数据底座
3. **参加 FabCon**（年度大会）—— 2027 春季是下一届
4. **进阶专题**：
   - 性能调优（V-Order、Spark adaptive execution、DW 统计信息）
   - 大规模迁移（Synapse → Fabric、Databricks → Fabric）
   - 安全合规深度（Purview 集成、敏感度标签全链路、Customer Key）
   - **Fabric IQ 业务本体设计**（这是 2026–2027 最值得投入的差异化能力）

---

## 7. 学习进度自检表

| 周 | 主题 | 完成日期 | 自我评分（1–5） | 备注 |
|----|------|----------|----------------|------|
| W1 | OneLake + 基础 | | | |
| W2 | Data Engineering | | | |
| W3 | Data Warehouse | | | |
| W4 | Real-Time Intelligence | | | |
| W5 | Power BI + DirectLake | | | |
| W6 | Mirroring + Shortcuts | | | |
| W7 | Copilot + Agents + IQ | | | |
| W8 | 治理 + Capstone | | | |
| — | DP-600 考试 | | | |
| — | DP-700 考试 | | | |
