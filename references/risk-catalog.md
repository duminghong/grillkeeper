# 风险库索引与计分规则

> **本文件是索引，不是定义源。** 权威定义在 vendor 层，见下表。诊断问题、严重度指南、"What Not to Flag" 反噪音清单**均不在此重复**，直接读 vendor 原文。

## 六大代码衰减风险（R1–R6）

| 码 | 名称 | 诊断问题 | 权威定义 |
|---|---|---|---|
| R1 | Cognitive Overload | 理解这段代码需要多少脑力？ | `vendor/brooks-lint/skills/_shared/decay-risks.md` § Risk 1 |
| R2 | Change Propagation | 改一处会连带崩掉多少无关的东西？ | 同上 § Risk 2 |
| R3 | Knowledge Duplication | 同一个决策是否在多处重复表达？ | 同上 § Risk 3 |
| R4 | Accidental Complexity | 代码是否比问题本身更复杂？ | 同上 § Risk 4 |
| R5 | Dependency Disorder | 依赖方向是否一致？ | 同上 § Risk 5 |
| R6 | Domain Model Distortion | 代码是否忠实表达领域？ | 同上 § Risk 6 |

每条风险的定义包含：Symptoms 明细、Sources 书目溯源表（Fowler/McConnell/Evans/Ousterhout 等）、Severity Guide、**What Not to Flag**。最后一项是误报防线，缺失会导致审查噪音泛滥——**必须读原文，不要凭印象判断**。

## 六大测试衰减风险（T1–T6）

| 码 | 名称 | 权威定义 |
|---|---|---|
| T1 | Test Obscurity | `vendor/brooks-lint/skills/_shared/test-decay-risks.md` § Risk T1 |
| T2 | Test Brittleness | 同上 § Risk T2 |
| T3 | Test Duplication | 同上 § Risk T3 |
| T4 | Mock Abuse | 同上 § Risk T4 |
| T5 | Coverage Illusion | 同上 § Risk T5 |
| T6 | Architecture Mismatch | 同上 § Risk T6 |

## 计分 / 配置 / 历史（不抄公式，读 vendor）

评分公式、配置 schema、历史文件格式**不在此重复定义**——防摘录漂移，一律读权威原文：

| 主题 | 权威位置 |
|------|---------|
| Health Score 计分（基础分 / 扣分权重 / 下限） | `vendor/brooks-lint/skills/_shared/common.md` § Health Score Calculation |
| 项目配置 `.grillkeeper/lint/config.yaml`（schema + 校验规则） | 同上 § Project Config / Config Validation。**注**：该路径由 vendor 源就地本地化而来（原 `.brooks-lint.yaml`），见 `vendor/SOURCES.md` § 本地修改 |
| suppress 匹配 / Trend 行 / 历史文件格式 | 同上 § Post-Report Triage / History Tracking |

**防混淆纪律（融合层规则，非摘录）**：阶段二（架构对抗审查）的扣分权重与 brooks-lint 不同——按模式各从其源，报告必须明确列出实际所用权重，禁止混用。
