# Week 6 Deep Dive · Mirroring（零 ETL 集成深度实操）

> **目标**：在 5 个练习里，把 Snowflake / Azure SQL / Cosmos DB 数据 **零 ETL** 镜像到 OneLake，并立刻用 DirectLake 出报表。
>
> **投入**：8–10 小时 ｜ **前置**：完成 [W5 DirectLake](../fabric-learning-plan.md)

---

## 0. 为什么 Mirroring 是 Fabric 最被低估的能力

**传统做法**：Snowflake → ADF / Fivetran / Airbyte → ADLS → Spark 清洗 → Lakehouse → Power BI
- 一份数据存 3 次（Snowflake、ADLS Stage、Lakehouse）
- 维护 N 个 pipeline、CDC、调度
- 延迟 30 min 到几小时

**Mirroring 做法**：Snowflake → Mirror → OneLake →（DirectLake）→ Power BI
- 数据**逻辑上**留在源、**物理上**在 OneLake 有一份 Delta 副本
- 微软**自动维护** CDC + 增量同步
- **不收存储费** + **不收计算费**（只在查询时按 CU 计费）
- 延迟 < 1 分钟

这是 Snowflake 和 Databricks 暂时都没有的**同类最强**能力（Lakehouse Federation 只是查询联邦，不是物理副本）。

---

## 1. 概念地图

```
┌──────────────┐      Mirror（CDC 自动捕获）     ┌──────────────┐
│              │ ─────────────────────────────> │              │
│   源数据库    │                                 │   OneLake    │
│              │ <── 元数据/Schema 拉取（不写回）── │  Delta 副本  │
└──────────────┘                                 └──────────────┘
        ↑                                                ↑
   仍是 source of truth                          Power BI / Spark / SQL
   原有应用继续读写                                可立即查询
```

**关键属性**：
- ✅ **单向**：Mirror 只从源到 OneLake，不会反向写回（防误删源）
- ✅ **近实时**：典型延迟 < 1 分钟
- ✅ **零计算费**：存储和同步计算由微软承担（仅查询时计 CU）
- ✅ **自动 schema 演进**：源加列 / 删列 / 改类型自动反映
- ⚠️ **DML 限制**：镜像表在 OneLake 侧**只读**，要改数据回源写
- ⚠️ **配额限制**：每容量最多镜像若干源（看 SKU），生产前确认

---

## 2. 三种典型 Mirroring 源对比

| 源 | 启用条件 | 同步方式 | 典型延迟 |
|----|---------|---------|----------|
| **Azure SQL DB** | 开启 SI（Snapshot Isolation）+ Change Feed | CDC | < 1 min |
| **Cosmos DB**（NoSQL） | 容器开启 Continuous Backup | CDC（changefeed） | < 1 min |
| **Snowflake** | 创建只读 service account + ACCOUNTADMIN 授权 | 增量轮询（基于流） | < 5 min |
| **PostgreSQL** | 启用 `wal_level=logical` + 复制槽 | Logical replication | < 1 min |
| **MySQL** | 启用 binlog (row mode) | Binlog | < 1 min |
| **Oracle**（FabCon 2026 新） | 启用 LogMiner 或 Oracle GoldenGate | CDC | < 5 min |
| **SAP**（FabCon 2026 新） | SAP Datasphere 桥接 | CDC | < 5 min |

---

## 3. 实操 Lab 1：Mirror Azure SQL Database

### 3.1 准备源

```sql
-- 在 Azure SQL DB 上执行
ALTER DATABASE [retaildb] SET ALLOW_SNAPSHOT_ISOLATION ON;
ALTER DATABASE [retaildb] SET CHANGE_TRACKING = ON
    (CHANGE_RETENTION = 3 DAYS, AUTO_CLEANUP = ON);

-- 在每张要镜像的表上开启变更跟踪
ALTER TABLE [dbo].[Orders] ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);
ALTER TABLE [dbo].[Customers] ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);
```

### 3.2 创建 Mirror

1. Fabric Workspace → New → **Mirrored Azure SQL Database**
2. 配置连接：endpoint / database / 凭据（推荐 Service Principal）
3. 选择要镜像的表：✅ Orders、✅ Customers
4. 点击 **Mirror database** —— 5–10 分钟初始化

### 3.3 验证

```sql
-- 在 Fabric 的 SQL Endpoint 查询（自动生成）
SELECT TOP 100 * FROM [retaildb].dbo.Orders;
SELECT COUNT(*) FROM [retaildb].dbo.Customers;
```

在源端插入一条记录：
```sql
INSERT INTO Orders VALUES (...);
```
等 30 秒，在 Fabric 端 `SELECT` 应该已经看到。

### 3.4 观察 Delta 文件
```python
# 在 Fabric Notebook 里
df = spark.read.format("delta").load(
    "abfss://<workspace>@onelake.dfs.fabric.microsoft.com/<MirroredDb>/Tables/Orders"
)
df.printSchema()
df.show(5)
```

---

## 4. 实操 Lab 2：Mirror Snowflake

### 4.1 准备 Snowflake 端

```sql
-- Snowflake 端：创建只读 service account
USE ROLE ACCOUNTADMIN;
CREATE ROLE fabric_mirror_role;
CREATE USER fabric_mirror_user 
    PASSWORD='<strong>' 
    DEFAULT_ROLE=fabric_mirror_role 
    DEFAULT_WAREHOUSE=COMPUTE_WH;
GRANT ROLE fabric_mirror_role TO USER fabric_mirror_user;

-- 授权读权限
GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE fabric_mirror_role;
GRANT USAGE ON DATABASE RETAIL TO ROLE fabric_mirror_role;
GRANT USAGE ON SCHEMA RETAIL.PUBLIC TO ROLE fabric_mirror_role;
GRANT SELECT ON ALL TABLES IN SCHEMA RETAIL.PUBLIC TO ROLE fabric_mirror_role;
GRANT SELECT ON FUTURE TABLES IN SCHEMA RETAIL.PUBLIC TO ROLE fabric_mirror_role;

-- 必须启用 CHANGE_TRACKING
ALTER TABLE RETAIL.PUBLIC.ORDERS SET CHANGE_TRACKING = TRUE;
ALTER TABLE RETAIL.PUBLIC.CUSTOMERS SET CHANGE_TRACKING = TRUE;
```

### 4.2 创建 Mirror

1. Fabric Workspace → New → **Mirrored Snowflake**
2. 连接配置：
   - Account: `xy12345.us-east-1.snowflakecomputing.com`
   - Warehouse: `COMPUTE_WH`
   - Database: `RETAIL`
   - User / Password
3. 选表 → Start mirroring

### 4.3 观察成本

- **Snowflake 侧**：每分钟 1 次轻量查询拉变更，单次成本 < $0.001
- **Fabric 侧**：存储和同步完全免费，只在你查询时计 CU

---

## 5. 实操 Lab 3：Mirror Cosmos DB

### 5.1 准备

```bash
# Cosmos DB 容器必须启用 Continuous Backup
az cosmosdb update --name <account> --resource-group <rg> \
    --backup-policy-type Continuous
```

### 5.2 创建 Mirror

1. Fabric Workspace → New → **Mirrored Azure Cosmos DB**
2. 选择 NoSQL 账号 / 数据库 / 容器
3. 启动镜像

### 5.3 半结构化数据查询

```python
# Cosmos JSON 文档镜像后变 Delta
df = spark.read.format("delta").load(
    "abfss://.../<MirroredCosmos>/Tables/orders_container"
)
# JSON 字段自动 flatten 为列
df.select("id", "customer.name", "items[0].sku").show()
```

---

## 6. 实操 Lab 4：用 Shortcut Transformation 做"轻 ETL"

FabCon 2026 后 Shortcut 不仅能"指向"外部数据，**还能在落地时做转换**（含 AI 转换）：

1. Workspace → New Shortcut → 选 ADLS Gen2
2. 指向 `https://...blob.core.windows.net/raw/customer-feedback/`（一堆 CSV）
3. 启用 **Transformation**：
   - 自动转 Delta
   - **AI 转换**：把 `feedback_text` 列调用 Foundry GPT 自动加 `sentiment`、`summary`、`translated_en` 三列
4. 在 Lakehouse 里立刻可查到带 AI 特征的 Delta 表

> 这是把 [contact-center demo](https://github.com/turbo998/contact-center-on-gpt-realtime) 的"音频 → AI 特征"模式应用到批量数据上的方式。

---

## 7. 实操 Lab 5：组合 — Mirror + DirectLake + Power BI

把上面三个 Mirror 组合成端到端业务：

```
Azure SQL Orders ─┐
Snowflake CUSTOMERS ┤── Fabric Lakehouse JOIN ── Gold 维度模型
Cosmos DB Items ──┘                                  ↓
                                          Power BI 语义模型
                                          （DirectLake 模式）
                                                     ↓
                                              业务用户实时仪表板
```

### 7.1 Notebook 做 JOIN

```python
orders = spark.read.format("delta").load("abfss://.../MirrorSql/Tables/Orders")
customers = spark.read.format("delta").load("abfss://.../MirrorSnowflake/Tables/CUSTOMERS")
items = spark.read.format("delta").load("abfss://.../MirrorCosmos/Tables/orders_container")

gold = (
    orders.alias("o")
    .join(customers.alias("c"), "customer_id")
    .join(items.alias("i"), orders["order_id"] == items["order_id"])
    .select(...)
)
gold.write.format("delta").mode("overwrite").saveAsTable("retail_gold.fact_orders")
```

### 7.2 自动建语义模型

- Lakehouse → New semantic model → DirectLake mode → 选 `fact_orders`、`dim_customer`、`dim_product`
- 关系自动识别
- Power BI Service 报表，**0 数据导入、0 刷新调度**

### 7.3 验证零数据延迟

- 在 Azure SQL 改一条 Orders 状态
- 等 30 秒
- Power BI 报表点击刷新 → 立即反映

---

## 8. 生产化注意事项

| 项 | 建议 |
|----|------|
| **凭据管理** | 用 Service Principal + Key Vault；定期轮换 |
| **网络** | 私有终结点（Private Endpoint）覆盖源和 OneLake |
| **配额监控** | Capacity Metrics App 查 Mirror 占用的存储与同步带宽 |
| **失败告警** | Activator 监控 Mirror 状态，失败 5 min 内发 Teams |
| **Schema 演进** | 源加列**会**自动传播；删列**不会**自动删（防破坏下游报表）|
| **大表初始化** | TB 级首次镜像可能数小时；尽量晚间触发 |
| **成本归因** | 镜像查询的 CU 计在**目标 Workspace**，不在源端 Fabric |

---

## 9. 与友商同类能力对比

| 能力 | Fabric Mirroring | Databricks Lakehouse Federation | Snowflake Iceberg Tables |
|------|------------------|--------------------------------|--------------------------|
| 数据是否落地 | ✅ Delta 物理副本 | ❌ 查询联邦 | ✅ Iceberg（需自己写） |
| 自动 CDC | ✅ | ❌ | ❌ |
| 计算费用 | 0（同步），按查询 CU | 按 DBU | 按 Credits |
| 存储费用 | **0** | 0 | 按 Iceberg 存储 |
| 源支持广度 | **★★★★★**（SAP/Oracle/SharePoint等） | ★★★（SQL DW 为主） | ★★ |
| 延迟 | < 1 min | 实时查询 | 自己控 |

**结论**：要把多个异构源**物理上**汇聚到一个 Lake 给 BI/AI 用，Fabric Mirroring 是 2026 唯一开箱即用的选择。

---

## 10. 自检

完成本周后你应该能：

- [ ] 解释 Mirroring 与 ETL、CDC pipeline、查询联邦的区别
- [ ] 配置 Azure SQL / Snowflake / Cosmos DB 三种源的 Mirror
- [ ] 在 Notebook 里直接读 Mirror 出来的 Delta
- [ ] 用 Mirror + DirectLake 跑通端到端零 ETL Demo
- [ ] 在客户场景下评估"Mirror vs 传统 ETL"的 TCO 差异

---

## 11. 推荐资源

- 官方文档：[What is Mirroring in Fabric?](https://learn.microsoft.com/fabric/database/mirrored-database/overview)
- FabCon 2026 keynote：*Mirroring goes everywhere*
- James Serra 博客：*Why Mirroring will change data integration*
- 微软示例：<https://github.com/microsoft/fabric-samples>（搜 `mirroring`）
