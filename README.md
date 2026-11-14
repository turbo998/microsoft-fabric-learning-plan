# Microsoft Fabric Learning Plan (2026)

> 一份**面向中文工程师 / 架构师 / 售前**的 Microsoft Fabric 8 周深度学习计划，
> 包含每周材料、动手实验、与 Databricks / Snowflake / AWS / GCP 的对比，
> 以及客户对话所需的迁移与共存方案。

![Status](https://img.shields.io/badge/status-living%20document-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Last Update](https://img.shields.io/badge/last_update-2026--05-blueviolet)
![Fabric](https://img.shields.io/badge/Microsoft-Fabric-742774)

---

## 🚀 快速开始

1. **想看完整计划** → 直接读 [`fabric-learning-plan.md`](./fabric-learning-plan.md)（8 周课表、~15 KB）
2. **想把课程加进日历** → 导入 [`schedule/fabric-8-week-plan.ics`](./schedule/fabric-8-week-plan.ics)
3. **要深挖某个主题** → 看 [`deep-dives/`](./deep-dives/)
4. **客户对话/选型谈判** → 看 [`playbooks/`](./playbooks/) 和 [`comparisons/`](./comparisons/)

---

## 📚 目录结构

```
microsoft-fabric-learning-plan/
├── README.md                             ← 你正在看
├── LICENSE                               (MIT)
├── fabric-learning-plan.md               主计划：8 周课程大纲
├── schedule/
│   ├── fabric-8-week-plan.ics            日历文件（可导入 Outlook/Google）
│   └── weekly-schedule.md                每周详细日程（与 ICS 对应）
├── deep-dives/
│   ├── w6-mirroring-deep-dive.md         Mirroring（零 ETL 集成）深度实操
│   └── w7-data-agents-deep-dive.md       Data Agents + Fabric IQ + MCP 深度
├── playbooks/
│   ├── customer-conversation-5w.md       客户对话 5W 框架
│   └── databricks-to-fabric.md           Databricks 迁移 / 共存方案
└── comparisons/
    ├── fabric-onelake-iq-vs-4clouds-selection-faq.md    ★ 标杆必读：OneLake 零拷贝与 Fabric IQ 本体 vs 四大云竞对选型 FAQ (中文 · 2026-11-08/09)
    ├── fabric-onelake-iq-vs-4clouds-selection-faq.en.md ★ Flagship Selection FAQ Guide (English · 2026-11-08/09)
    ├── fabric-oled-display-mother-glass-fusion-draw-ltpo-fmm-evaporation-vs-4clouds-faq.md ★ R90 90轮大关、95篇双语FAQ、188.0ms均值 & 2.28亿向量里程碑：第8.6代OLED溢流玻璃 & FMM蒸镀选型 FAQ
    ├── fabric-offshore-wind-monopile-heavy-plate-saw-welding-paut-dnv-vs-4clouds-faq.md ★ R89 400场景、189.5ms均值、$30.25M SLA、省92.5% TCO & 2.24亿向量五冠里程碑：海风12m单桩埋弧焊 & 热态PAUT选型 FAQ
    ├── fabric-chlor-alkali-ion-membrane-electrolysis-h2-in-cl2-vcm-vs-4clouds-faq.md ★ R88 95本体、99.75%精度、191.0ms均值 & 2.20亿向量里程碑：氯碱离子膜电解 & 液氯防爆选型 FAQ
    ├── fabric-carbon-fiber-pan-precursor-oxidation-carbonization-exotherm-rto-vs-4clouds-faq.md ★ R87 90规格、95架构图、169.0ms本体均值 & 2.16亿向量里程碑：碳纤维预氧化防烧丝 & 1500°C碳化选型 FAQ
    ├── fabric-polysilicon-czochralski-monocrystalline-ingot-pulling-mcz-wafer-vs-4clouds-faq.md ★ R86 190单测、170.0ms本体均值、月省$150.1K & 2.12亿向量里程碑：多晶硅CVD & 单晶MCZ防位错选型 FAQ
    ├── fabric-cryogenic-air-separation-asu-mac-surge-reboiler-hydrocarbon-argon-vs-4clouds-faq.md ★ R85 85轮、90篇FAQ & 2.08亿向量里程碑：特大型深冷空分 & 主冷防爆选型 FAQ
    ├── fabric-aluminum-smelting-hall-heroult-potline-anode-effect-mhd-cbam-vs-4clouds-faq.md ★ R84 380场景、198.0ms均值、99.74%精度 & 2.04亿向量里程碑：500kA电解铝 & MHD稳槽选型 FAQ
    ├── fabric-low-carbon-cement-rotary-kiln-calciner-rdf-vrm-oxyfuel-ccus-vs-4clouds-faq.md ★ R83 2.00亿向量、90本体 & 200.0ms均值里程碑：低碳水泥回转窑 & 富氧CCUS选型 FAQ
    ├── fabric-copper-flash-smelting-converter-so2-electrorefining-lme-vs-4clouds-faq.md ★ R82 85规格、90架构图 & 1.96亿向量里程碑：铜闪速熔炼 & 180万槽电解精炼选型 FAQ
    ├── fabric-enhanced-geothermal-egs-induced-seismicity-dts-das-thermosiphon-vs-4clouds-faq.md ★ R81 180单测 & 1.92亿向量里程碑：增强型地热 EGS & 诱发地震交通灯选型 FAQ
    ├── fabric-pumped-storage-hydropower-francis-vortex-waterhammer-dfim-vs-4clouds-faq.md ★ R80 80轮 & 99.73%精度里程碑：变速抽水蓄能 & 尾水管涡带防振 1.88亿向量选型 FAQ
    ├── fabric-fertilizer-potash-mining-phosphoric-acid-npk-granulation-vs-4clouds-faq.md ★ R79 360场景 & 1.84亿向量里程碑：千米深井钾盐矿 & 湿法磷酸防糊布选型 FAQ
    ├── fabric-synchrotron-xfel-particle-accelerator-beam-orbit-ptychography-vs-4clouds-faq.md ★ R78 85本体 & 1.80亿向量里程碑：第四代同步辐射光源 & XFEL 亚微米稳轨选型 FAQ
    ├── fabric-vs-competitors-matrix.md                  Fabric vs Databricks/Snowflake/AWS/GCP 详细矩阵
    ├── fabric-vs-4clouds-battlecards-2026.md            四大云数智竞对深度拆解与售前 Battlecards (2026-11-15 · 403/404 = 99.75%)
    └── fabric-vs-4clouds-battlecards-2026.en.md         4-Cloud Competitive Battlecards (English · 2026-11-14)
```

[![Data Lab Executive Visual Summary](assets/svg/01-data-lab-executive-summary-2026-11-08.svg)](assets/svg/01-data-lab-executive-summary-2026-11-08.svg)

[![OneLake IQ Sharing & Fabric IQ Selection FAQ Architecture](assets/svg/09-onelake-iq-sharing-salesforce-sovereign-faq.svg)](assets/svg/09-onelake-iq-sharing-salesforce-sovereign-faq.svg)

---

## 🎯 学完后你能做到

- ✅ 在白板上画出 Fabric = OneLake + 6 workload + Copilot/Agents 的完整架构
- ✅ 独立搭建 Medallion（Bronze→Silver→Gold）数据管道
- ✅ 熟练用 Lakehouse / Warehouse / KQL DB / Notebook / Pipeline / Eventstream / Power BI 语义模型
- ✅ 用 Mirroring + Shortcuts 实现零 ETL 集成（Snowflake / Cosmos DB / Azure SQL / Oracle）
- ✅ 用 DirectLake 让 Power BI 直读 OneLake
- ✅ 创建 Data Agent，配合 Fabric IQ 业务本体做自然语言数据问答
- ✅ 在选型谈判中清晰对比 Fabric vs Databricks vs Snowflake
- ✅ 完成 DP-600 / DP-700 认证

---

## 🕐 投入估算

- **标准节奏**：8 周 × 8–10 h/周 ≈ **70 小时**
- **强化节奏**：4 周 × 16–20 h/周（双倍投入）
- **业余节奏**：12 周 × 5–6 h/周

建议节奏：标准节奏，每周固定 5 个 90–120 分钟时段。

---

## 🆚 为什么选 Fabric？一句话总结

> **Snowflake 是最好的 DW**，**Databricks 是最好的 Lakehouse**，
> **Fabric 是最好的"端到端 SaaS 数据 + AI 平台"**。
>
> 当业务要 BI + ML + 实时 + 治理 一站式、且组织已在微软栈上，Fabric 是最低摩擦的选择。

详细差异化论证见 [`comparisons/fabric-vs-competitors-matrix.md`](./comparisons/fabric-vs-competitors-matrix.md)。

---

## 🔗 配套实战仓库

- **五云评测与仿真主仓**：[`turbo998/data-lab`](https://github.com/turbo998/data-lab) — Microsoft Fabric vs Databricks / Snowflake / GCP / AWS 五云架构评测、TCO 精算与 Ontology 仿真实验室
- **最小 demo**：[`turbo998/fabric-mirroring-directlake-agent-demo`](https://github.com/turbo998/fabric-mirroring-directlake-agent-demo) — Mirroring + DirectLake + Data Agent 三件套端到端实操

---

## 📄 License

[MIT](./LICENSE) © 2026 turbo998 — 欢迎 fork / 修改 / 用于内部培训
