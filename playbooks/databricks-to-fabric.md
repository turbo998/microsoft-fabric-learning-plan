# Databricks 客户：迁移 / 共存方案

> **场景**：客户已经用 Databricks 一段时间，遇到 BI/Copilot/治理痛点，问能不能换 Fabric / 引入 Fabric。
>
> **本文不是劝退 Databricks**，而是提供两条务实路径：
> 1. **共存（多数情况）** —— Databricks 留作 ML 工厂，Fabric 补 BI/Agent/治理
> 2. **迁移（少数情况）** —— 全量替换 Databricks 到 Fabric

---

## 1. 先判断：客户究竟该不该动？

| 客户现状 | 建议 |
|---------|------|
| Databricks 用得"不错但 BI 体验差、Power BI 报表慢" | **共存 模式 A** |
| Databricks 主要做 ETL + Lakehouse，ML 用得少 | **可迁移**（评估后） |
| Databricks 做了重 ML（Mosaic AI、Feature Store、Vector Search） | **共存 模式 B**，不要迁移 ML |
| Databricks 在 AWS / GCP，要降本 | **共存** 或 **谨慎迁移**（涉及跨云搬数据，成本高） |
| Databricks 是工程师的精神图腾 | **共存**（强迁移会引发组织抵抗） |
| 还在 PoC、Databricks 没真正生产 | **可彻底切换**（沉没成本低） |

**判断公式**：

> **沉没投入 < 6 个月 + 工作负载 < 20 TB + 团队接受度高** → 可迁移
> **否则** → 优先共存

---

## 2. 共存模式（推荐 80% 场景）

### 模式 A：Databricks 当 ML 工厂 + Fabric 当业务交付层

```
原始数据
    ↓
Databricks（DLT 管道 + 模型训练 + Feature Store）
    ↓ 写出 Delta 表 / Delta Sharing
OneLake（通过 Shortcut 引用，零拷贝）
    ↓
Fabric Warehouse / Lakehouse / Power BI DirectLake / Data Agents
    ↓
业务用户、坐席、管理层
```

**关键技术**：
- **OneLake Shortcut to Databricks ADLS** —— 把 Databricks 的 Delta 表通过 Shortcut 引用到 OneLake，零拷贝
- **Delta Sharing** —— Databricks 主动 share，Fabric 通过 Recipient 接收
- **Unity Catalog ↔ Purview** —— 两边治理标签可以打通（FabCon 2026 后能力增强）

**职责分工**：
| 角色 | Databricks | Fabric |
|------|-----------|--------|
| 大规模 ETL（PB 级） | ✅ DLT/Workflows | — |
| ML 训练 / 模型管理 | ✅ Mosaic AI/MLflow | — |
| Feature Store | ✅ | — |
| BI 报表（DirectLake） | — | ✅ |
| 业务用户自助查询 | — | ✅ Data Agents |
| 实时分析 | — | ✅ RTI / KQL |
| 数据治理（端到端 lineage） | UC | Purview（含 UC） |
| Mirroring 老系统（SAP/Oracle） | — | ✅ |

### 模式 B：Fabric 当统一 + Databricks 仅做特定重 ML

适用：客户**主要工作负载在 Fabric**，但有 1-2 个特定大模型训练任务必须用 Databricks。

```
所有数据 → Fabric（OneLake + 全 workload）
              ↓ Shortcut（双向）
           Databricks（仅大模型训练）
              ↓ 模型回写为 ONNX / MLflow model
           Fabric Model Serving / Power BI
```

### 共存清单

- [ ] OneLake Shortcut 指向 Databricks ADLS 路径
- [ ] Databricks 服务账号给 Fabric 只读权限
- [ ] 模型用 MLflow 注册，Fabric 通过 MLflow Tracking URI 调用
- [ ] 治理：Unity Catalog 标签 → Purview Asset Tag 同步
- [ ] 监控：Databricks 系统表 + Fabric Capacity Metrics 双仪表板

---

## 3. 迁移模式（仅当判断公式通过）

### 3.1 评估阶段（2 周）

| 评估项 | 工具 / 方法 |
|--------|-------------|
| 数据资产清单 | Unity Catalog export + 容量统计 |
| Notebook 数量与复杂度 | Databricks REST API 拉清单 |
| Workflow / Job 数量 | Workflow API |
| 集成点（上下游） | 抽样访谈 + 网络流量分析 |
| ML 模型清单 | MLflow Registry export |
| 团队 Spark 经验 | 内部调研 |
| 月度成本基线 | Databricks 账单分析 |

**输出**：迁移可行性报告（含 TCO 对比、风险清单、迁移路线图）。

### 3.2 迁移路径

```
Phase 1 (Week 1-4): 数据层迁移
  Databricks Delta on ADLS → 通过 Shortcut 接入 OneLake（零拷贝）
  验证：Fabric 能查到与 Databricks 一致的数据

Phase 2 (Week 5-8): Pipeline 迁移
  DLT/Workflows → Fabric Pipeline / Notebook + Schedule
  逐管道迁移，双跑对账 2 周后切换

Phase 3 (Week 9-12): BI 层切换
  Databricks SQL Dashboard → Power BI DirectLake
  业务用户培训 + 报表重建

Phase 4 (Week 13-16): ML 迁移（如适用）
  MLflow → Fabric Data Science 实验
  模型重训练验证一致性

Phase 5 (Week 17-20): 治理切换 + Databricks 关停
  Unity Catalog 资产 → Purview 注册
  逐 workspace 关停 Databricks，保留 1 个月只读以防回滚
```

### 3.3 Notebook 自动迁移工具

```python
# 示意：把 Databricks Notebook (.ipynb) 转 Fabric Notebook
# Fabric 完全兼容 Jupyter ipynb 格式，绝大多数 PySpark 代码可直接跑

# 需要替换的常见 API：
DATABRICKS_TO_FABRIC = {
    "dbutils.widgets.get": "fabric.utils.parameters.get",        # 参数
    "dbutils.notebook.run": "fabric.notebookutils.run",          # 嵌套 notebook
    "dbutils.fs.ls": "notebookutils.fs.ls",                      # 文件
    "dbutils.secrets.get": "notebookutils.credentials.getSecret",# 凭据
    "spark.sql('USE CATALOG x')": "# Fabric 用 Lakehouse 隔离",
}
```

绝大多数 PySpark / Spark SQL 代码**无需修改**即可在 Fabric Notebook 跑通。

### 3.4 主要差异与陷阱

| 维度 | Databricks | Fabric | 注意 |
|------|-----------|--------|------|
| `dbutils.*` | 原生 | 用 `notebookutils.*` | 批量替换 |
| Auto Loader | ✅ | ❌（用 Pipeline + Eventstream） | 重写为 Pipeline |
| Delta Live Tables | ✅ | ❌（用 Pipeline + 编排） | 重写 |
| Photon | ✅ | ❌（Fabric 用自己的优化） | 性能可能下降 30%，需测试 |
| Unity Catalog | ✅ | OneLake security + Purview | 标签映射 |
| Cluster 配置 | 细粒度 | F-SKU 容量 | 心智转变 |
| Vector Search | ✅ | Azure AI Search 外挂 | 重新建索引 |

---

## 4. 成本对比示意（中型企业，月度）

| 项目 | Databricks（基线） | 迁移到 Fabric | 共存模式 A |
|------|-------------------|--------------|------------|
| 数据存储 | $2,000 (ADLS) | $500 (OneLake) + $2,000 (保留 ADLS via shortcut) | $2,000 |
| 计算 | $15,000 (DBU) | $8,000 (F64) | $10,000 (DBU 减一半) + $4,000 (F32) |
| BI（Power BI Pro 200 用户）| $2,000 | $0 (含 F-SKU) | $0 (含 F-SKU) |
| 治理（Purview） | 外购 $1,500 | 含 | 含 |
| **合计** | **$20,500** | **$10,500** | **$16,000** |

⚠️ 真实成本因 region / commit / 用量分布差异巨大，仅供量级参考。

---

## 5. 迁移风险与缓解

| 风险 | 缓解 |
|------|------|
| Spark 代码不兼容 | 试点 5-10 个代表性 Notebook，量化失败率 |
| 性能下降（无 Photon） | 用 V-Order + Adaptive Query Execution 调优；接受 10-20% 差距 |
| 团队抵抗 | 共存先行；3 个月看效果再决定全迁；明确 Databricks 老兵的新角色 |
| 历史模型不可复现 | MLflow 模型先在 Fabric 重新部署验证，再下线源端 |
| 上下游集成断开 | 双跑期间逐个对账 |
| 数据治理空窗期 | Purview 提前注册所有资产；UC 标签同步脚本 |

---

## 6. 销售话术模板

### 6.1 客户："为什么我要从 Databricks 换 Fabric？"

> "**多数情况下我不建议你换。** Databricks 是优秀的 Lakehouse 工程平台。
>
> 你应该考虑 Fabric 的情况是：
> 1. 你的 BI 体验跟不上业务速度（Power BI DirectLake 是质变）
> 2. 你的 IT/分析师/业务用户比例失衡，工程门槛太高
> 3. 你需要把数据资产快速变成 AI Agent / 自然语言查询
> 4. 你已经买了 Power BI Premium / M365 E5，但 Databricks 是单独账单
>
> 即使以上都满足，我的第一推荐也是**共存** —— 让 Databricks 继续做它最擅长的 ML，
> 让 Fabric 来做 BI/Agent/治理。这样**零迁移风险**，60 天内业务就能感受到改善。"

### 6.2 客户："共存会不会数据重复存？"

> "不会。OneLake Shortcut 是引用而非拷贝，Databricks 的 Delta 表
> 在 OneLake 里看起来就像本地表，但物理上**还是同一份文件**。
>
> 你也不会被双重计费 —— ADLS 那份存储费照付，OneLake Shortcut 是免费的。"

### 6.3 客户："那 Unity Catalog 怎么办？"

> "FabCon 2026 后 Fabric 已经能 **import** Unity Catalog 的标签到 Purview。
> 短期内两边继续各管各的，长期看 Purview 治理面更广（覆盖 M365、Power Platform、SaaS），
> 是更合适的企业级治理母舰。"

### 6.4 客户："如果以后想全切 Fabric，可逆吗？"

> "可逆。OneLake 用的是开放 Delta 格式，你随时可以指回 Databricks 读。
> 这是 Fabric 故意设计的 —— 用**开放性**降低你的决策风险。"

---

## 7. 客户案例参考

> ⚠️ 以下为典型场景模式，非真实命名

**案例 A：欧洲零售**
- 起点：Databricks on Azure + Tableau，BI 慢、用户少
- 决策：共存模式 A，Power BI DirectLake 替换 Tableau
- 6 个月：BI 用户从 50 涨到 800，Databricks ETL 不变
- 结果：业务满意度提升、Databricks 用量持平

**案例 B：金融科技**
- 起点：Databricks + 自研 BI，治理一团乱
- 决策：迁移 BI + 治理到 Fabric，ML 留在 Databricks
- 9 个月：Purview 治理全量上线，BI 性能 5×
- 结果：合规审计通过率从 60% 到 95%

**案例 C：制造业**
- 起点：Databricks + Snowflake + Power BI 三栈
- 决策：Mirroring Snowflake 进 OneLake，Databricks 改用 OneLake Shortcut
- 12 个月：三栈合一，ETL 工程量减半
- 结果：月成本降 35%

---

## 8. 一句话总结

> **Databricks 是马拉松冠军，Fabric 是十项全能。**
>
> 不要让客户在两个冠军之间二选一 —— 让两个冠军做各自最擅长的事，
> 用 OneLake 把数据资产打通，让客户得到 1 + 1 > 2 的体验。
