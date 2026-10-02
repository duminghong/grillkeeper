# 来源与许可（SOURCES）

本目录主体是**第三方内容的上游快照**，上游为本地已安装的 skill，保留原文以确保能力完整。

**例外**：为把该 skill 的全部产物约束到 `.grillkeeper/` 下，对**路径字面量**做了本地补丁（**只改路径，不含任何诊断标准、评分公式、风险定义、书目溯源**）。因此本目录**不是逐字副本**——完整清单见下方「本地修改（fork diffs）」，该清单是升级时的重放依据。

## 来源清单

| vendor 目录 | 上游来源 | 版本 | 许可证 | 文件数 |
|---|---|---|---|---|
| `brooks-lint/` | plugin `trae-remote-official:brooks-lint` | 1.3.0 | MIT © 2025 hyhmrright（`LICENSE` 已随附） | 21 |
| `staff-engineer-mode/` | plugin `trae-remote-official:staff-engineer-mode` | 2.1.0 | MIT © sirmarkz（`LICENSE` 已随附） | 143 |
| `product-lifecycle-workbench/` | plugin `trae-remote-official:product-lifecycle-workbench` | 0.2.1 | ⚠️ 未声明许可——仅作本地参考，公开发布前需确认授权 | 9 |
| `architecture-breaker-review/` | 用户级 skill（`~/.trae-cn/skills/`） | 未标注 | ⚠️ 未声明许可——仅作本地参考，公开发布前需确认授权 | 1 |
| `grilling/` | 项目级 skill（`.trae/skills/grilling`） | 未标注 | ⚠️ 未声明许可——仅作本地参考，公开发布前需确认授权 | 1 |
| `domain-modeling/` | 项目级 skill（`.trae/skills/domain-modeling`） | 未标注 | ⚠️ 未声明许可——仅作本地参考，公开发布前需确认授权 | 3 |

`brooks-lint/LICENSE`、`brooks-lint/THIRD_PARTY_NOTICES.md`、`staff-engineer-mode/LICENSE`、`staff-engineer-mode/THIRD_PARTY_NOTICES.md` 均已原样保留，符合 MIT 的声明保留要求。

## 完整性说明

- 六个来源的**能力文件（`skills/**`、`SKILL.disabled.md`、`references/`、`specialists/`、`assets/`）已 100% vendor，无遗漏**（除「本地修改」一节列出的路径补丁外逐字）。未收录的只有打包/营销类文件（`plugin.json`、`package.json`、`README.md`、`SECURITY.md`、图标、`.success`、`.codexignore`、`agents/openai.yaml`），它们不承载能力。
- `references/adversarial-review-checklist.md` 是从 `architecture-breaker-review/SKILL.disabled.md` 提炼的融合层清单，原文保留在 `vendor/`。
- `references/grill-protocol.md` **不是自研替代品**：它以 `vendor/grilling/SKILL.disabled.md` 的设计树/frontier 机制为骨架，只在其上叠加"即时审查内循环"；术语与 ADR 格式遵循 `vendor/domain-modeling/`。
- `domain-modeling` 的格式文件被 `references/grill-protocol.md` 叠加 6 直接引用（术语表 / ADR 格式权威），故一并 vendor 以满足自包含要求。

## 本地修改（fork diffs）

**目的**：`.grillkeeper/` 是本 skill 的唯一写入根，故将上游钉死在 project root / repo root 的路径改写到 `.grillkeeper/` 下。**仅改路径字符串**，未改任何诊断标准、评分公式、风险码、严重度定义与书目溯源。

| 文件 | 上游原文 | 本地改为 |
|---|---|---|
| `brooks-lint/skills/_shared/common.md` | `.brooks-lint.yaml`（project root） | `.grillkeeper/lint/config.yaml` |
| 同上 | `.brooks-lint-history.json` | `.grillkeeper/lint/history.json` |
| `brooks-lint/skills/_shared/custom-risks-guide.md` | `.brooks-lint.yaml` | `.grillkeeper/lint/config.yaml` |
| `brooks-lint/skills/brooks-sweep/sweep-guide.md` | `.brooks-lint.yaml` | `.grillkeeper/lint/config.yaml` |
| `domain-modeling/SKILL.disabled.md` | root `CONTEXT.md` / `docs/adr/` | `.grillkeeper/project/CONTEXT.md` / `.grillkeeper/project/adr/` |
| `domain-modeling/CONTEXT-FORMAT.md` | root `CONTEXT.md` / `CONTEXT-MAP.md` | `.grillkeeper/project/` 下同名文件 |
| `domain-modeling/ADR-FORMAT.md` | `docs/adr/` | `.grillkeeper/project/adr/` |

**有意保留未改**（改了反而错）：

- `domain-modeling/SKILL.disabled.md` 多上下文示例中 `src/<ctx>/CONTEXT.md`、`src/<ctx>/docs/adr/` —— 这是**per-package 局部路径**，不属于 `.grillkeeper/` 管辖范围，示例语义就是"每个包自己有一份"。
- `common.md:23` 括号内 "(this repo's localized path for `.brooks-lint.yaml`)" —— 保留上游名，便于溯源对照。

**这不影响独立可运行性**：`CONTEXT.md` 与 `docs/adr/` 的缺失只是降级（行为仍正确）；`.grillkeeper/lint/config.yaml` 是本仓库的固定约定，无论经由 grillkeeper 还是直接加载 vendor skill，读的是同一路径，**不再存在"standalone 读不到配置"的静默失效**。

## 使用约定

**vendor 层是权威源（canonical），`references/` 层是索引与融合层。**

发生冲突时以 vendor 为准，`references/` 只做导航与跨能力融合，不重复定义诊断问题、评分公式或检查项。

| 需要什么 | 读哪里（权威） |
|---|---|
| 六大代码衰减风险完整定义（症状/书目溯源/严重度指南/**What Not to Flag**） | `vendor/brooks-lint/skills/_shared/decay-risks.md` |
| 六大测试衰减风险完整定义 | `vendor/brooks-lint/skills/_shared/test-decay-risks.md` |
| Iron Law、配置、计分、历史、triage、报告模板 | `vendor/brooks-lint/skills/_shared/common.md` |
| 十二本书的覆盖矩阵与例外 | `vendor/brooks-lint/skills/_shared/source-coverage.md` |
| 修复分级与不可自动修复项 | `vendor/brooks-lint/skills/_shared/remedy-guide.md` |
| 自定义风险码 | `vendor/brooks-lint/skills/_shared/custom-risks-guide.md` |
| 六个 brooks 子模式（audit/debt/health/review/sweep/test） | `vendor/brooks-lint/skills/brooks-*/SKILL.disabled.md` |
| **exact-slug 路由消歧规则**（60+ 条） | `vendor/staff-engineer-mode/skills/staff-engineer-mode/references/routing-matrix.md` |
| 工程生命周期主流程与上下文引导 | `vendor/staff-engineer-mode/skills/staff-engineer-mode/SKILL.disabled.md`、`references/bootstrap-context.md` |
| **64 个专家检查清单全文** | `vendor/staff-engineer-mode/specialists/<slug>.md` |
| 专家产出的结构化模板（71 个） | `vendor/staff-engineer-mode/skills/_shared/assets/templates/` |
| 8 种产品文档模板全文 | `vendor/product-lifecycle-workbench/skills/product-doc-writing/references/*.md` |
| 产品文档写作总纲 | `vendor/product-lifecycle-workbench/skills/product-doc-writing/SKILL.disabled.md` |
| 架构破坏专家原始方法论 | `vendor/architecture-breaker-review/SKILL.disabled.md` |
| **访谈机制（设计树 / frontier / 问题格式 / 收束条件）** | `vendor/grilling/SKILL.disabled.md` |
| **术语表 `CONTEXT.md` 格式**（含 `_Avoid_` 同义词、多上下文 `CONTEXT-MAP.md`） | `vendor/domain-modeling/CONTEXT-FORMAT.md` |
| **ADR 格式 + "何时才该写 ADR"三条件判定** | `vendor/domain-modeling/ADR-FORMAT.md` |
| 领域建模纪律（挑战术语 / 磨利模糊语言 / 与代码交叉验证） | `vendor/domain-modeling/SKILL.disabled.md` |

## 上游更新

vendor 副本是**快照**，不会自动跟随上游。升级步骤：

1. 重新复制上游目录覆盖 `vendor/<origin>/`，并同步更新本文件的版本号
2. **重放「本地修改（fork diffs）」表**——重铺上游会覆盖全部路径补丁
3. 核对补丁重放完整（两个互补断言，缺一不可）：
   - **旧形态将消失**：`grep -rn "brooks-lint\.yaml\|brooks-lint-history\|docs/adr" vendor/` 应只剩溯源标注与 per-package 示例
   - **新形态必存在**：`grep -rl "\.grillkeeper/" vendor/` 必须命中 `brooks-lint/skills/_shared/common.md`、`brooks-lint/skills/brooks-sweep/sweep-guide.md`、`domain-modeling/SKILL.disabled.md`、`domain-modeling/CONTEXT-FORMAT.md`、`domain-modeling/ADR-FORMAT.md`
   - **注**：比对模式**不含** `CONTEXT\.md`——新路径 `.grillkeeper/project/CONTEXT.md` 本身即含该子串，纳入会令期望永不可满足（假断言）

> **打包约定（必做）**：重铺上游后，把 vendor 内的所有 `SKILL.md` 改名为 `SKILL.disabled.md`。否则宿主若用递归扫描（`**/SKILL.md`）发现 skill，会把 vendor 里 11 个上游 skill 注册成独立 skill——与"禁止外部 skill 分流"约束冲突，且与用户已装的同名 skill（grilling / domain-modeling / brooks-* / staff-engineer-mode 等）撞名。

升级后**必须核对融合层是否漂移**：`references/` 中引用的章节锚点、严重度判定标准、模式切换规则逐一对照新 vendor 原文（评分公式与配置 schema 已是纯指针，无需核对）。
