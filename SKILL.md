---
name: grillkeeper
description: |
  打磨访谈守门员：自包含的全周期质量守护 Skill，不依赖任何外部 skill。
  以访谈打磨需求、内循环即时审查，覆盖架构对抗审查、代码审查、测试质量、
  技术债务、发布就绪及文档产出。当用户要求"打磨需求"、"拷问方案"、
  "全流程质量审查"、"综合质量门禁"或涉及多维度质量评估时使用。
# 客户端扩展字段（开放规范未定义；Claude Code / Cursor / VS Code 支持）：仅允许显式 /grillkeeper 触发
disable-model-invocation: true
compatibility: 自包含、无外部依赖；需可读 vendor/ 目录（179 个文件）。设计为仅显式 /grillkeeper 触发。
---

# 打磨访谈守门员（Grillkeeper）

**自包含 Skill：运行时不调用任何外部 skill。** 能力来自两层：

| 层 | 位置 | 角色 |
|---|---|---|
| **vendor 层** | `vendor/` | 上游内容逐字副本 = **权威源**（179 文件 / ~1.35 MB） |
| **融合层** | `references/` + 本文件 | 索引、跨能力融合、访谈内循环 |

冲突时以 vendor 为准。来源、版本、许可证见 `vendor/SOURCES.md`。vendor 文本内的 slash 引用（如 `/grilling`）一律解析为本仓库 vendor 路径，禁止解析为外部 skill 调用。

## 内置能力域

- **代码/测试质量扫描** — 6 个 brooks 子模式；6 大代码衰减风险 R1–R6 + 6 大测试风险 T1–T6
- **架构对抗审查** — 五维度扫描 + 可复现破防场景
- **工程场景路由** — 64 个专家检查清单 + 60+ 条 exact-slug 消歧规则
- **打磨访谈** — 设计树 + frontier 分轮推进（上游机制）+ 定基调根节点 + 每轮即时审查内循环
- **领域建模** — 术语钉入 `CONTEXT.md`、ADR 三条件判定与落盘
- **文档产出** — 8 种产品文档模板（vendor 全文）
- **修复与 triage** — 修复分级、accept/dismiss/defer/skip、suppress 机制

## 核心原则

1. **Iron Law**: 每个发现必须 `Symptom → Source → Consequence → Remedy` 四段式；未完成风险诊断前不得给修复建议
2. **对抗性优先**: 先找断头路、自我矛盾、越权依赖，再谈优点
3. **打磨审查一体化**: 需求打磨和架构审查不是串行两阶段，而是每轮访谈内嵌即时审查，审查发现直接挂为设计树新节点（随 frontier 推进，不可能被漏掉）
4. **权威源优先**: 诊断问题、严重度指南、评分公式、检查项一律读 vendor 原文，不凭印象发挥
5. **文档即交付**: 审查结论必须能沉淀为结构化文档（PRD/ADR/报告）

---

## 阶段路由表（Phase Router）

| 用户意图 | 主模式 | 权威来源 | 输出物 |
|---------|--------|---------|--------|
| "这个需求/设计靠谱吗" | 打磨访谈 + 即时审查内循环 | `vendor/grilling/SKILL.disabled.md`（机制）+ `references/grill-protocol.md`（叠加层） | 打磨后的需求/设计（含设计树与即时审查记录） |
| "审查架构是否合理" | 架构对抗审查 | `references/adversarial-review-checklist.md` | 架构审查报告 + 依赖图 |
| "审查这个 PR/代码" | 代码审查 | `vendor/brooks-lint/skills/brooks-review/SKILL.disabled.md` | PR 审查报告 |
| "技术债务怎么还" | 债务评估 | `vendor/brooks-lint/skills/brooks-debt/SKILL.disabled.md` | 债务优先级矩阵 |
| "测试质量怎么样" | 测试审查 | `vendor/brooks-lint/skills/brooks-test/SKILL.disabled.md` | 测试质量报告 |
| "上线前全面检查" | 四维扫描 + 就绪审查 | `vendor/brooks-lint/skills/brooks-health/SKILL.disabled.md` + `vendor/staff-engineer-mode/specialists/production-readiness-review.md` | 发布就绪报告 |
| "自动修复所有问题" | 全量扫描+修复 | `vendor/brooks-lint/skills/brooks-sweep/SKILL.disabled.md` | 修复日志 + 残余问题 |
| "写 PRD/设计文档" | 文档产出 | `vendor/product-lifecycle-workbench/skills/product-doc-writing/references/*.md` | 结构化文档 |

> **"架构"类请求消歧**：审查对象是**设计/方案文档**（未落地）→ 阶段二架构对抗审查；审查对象是**已落地代码**的模块结构 → 阶段三 brooks-audit。触发词模糊时按此裁决，不允许两个模式混用。

---

## 阶段一：需求打磨 + 即时审查（Grill-Review 模式）

**触发词**: "打磨需求"、"拷问设计"、"这个方案靠谱吗"、"帮我完善计划"

### 流程

**访谈机制以上游为准**（`vendor/grilling/SKILL.disabled.md`）：把需求映射成**设计树**，每轮问完当前 **frontier**（前置条件已落定的全部决策），给每个问题编号并附推荐答案，等用户回答后重算 frontier。

1. 读取 `references/grill-protocol.md`（融合层）与 `vendor/grilling/SKILL.disabled.md`（机制权威）
2. **第一轮先问定基调两问（树的根节点，必问）**：是否需要向后兼容？按最彻底还是最小改动？模糊回答必须逼二选一
3. 按 frontier 分轮提问，问题格式固定：`❓ **Q1** - **<标题>**: <正文>` + `➡️ <推荐答案>`
4. **每轮收到回答后执行即时审查**（这是本 skill 的叠加，上游没有）：
   - 用 `references/adversarial-review-checklist.md` 扫描逻辑闭环/边界/生产生存问题
   - 用 `vendor/staff-engineer-mode/skills/staff-engineer-mode/references/routing-matrix.md` 做 exact-slug 路由，命中后打开对应 specialist
   - 按定基调附加检查：兼容=是 → 兼容性检查；最小改动 → 增量检查
   - 扫描分级（A 组 3 项每轮必跑 / B 组 11 项信号触发）、节点挂载门槛与 15 轮预算，见 `references/grill-protocol.md` 叠加 3
5. **审查发现 = 挂进设计树的新节点**（以该决策为前置），随之进入 frontier；仅 🔴/🟡 级挂节点，🟢 记入残余观察
6. **状态落盘**：基调两问落定后创建 `.grillkeeper/session.md`，每轮追加 frontier 快照与审查发现；中断重启先读它重建状态，禁止凭记忆续跑（见 grill-protocol 叠加 7）
7. **找事实是 agent 的职责**：需要环境事实时派 sub-agent 查，不要问用户；不要阻塞，只让下游问题等
8. 收束 = **frontier 为空**（设计树每个分支都走过，无默默假设）；未获用户确认前不动手实施

### 即时审查循环（每轮必做）

```
本轮问完整个 frontier（编号 + 推荐答案）
   ↓
用户回答
   ↓
【叠加】对每个回答即时审查
   ↓
审查发现 → 新树节点 → 进入 frontier
   ↓
重算 frontier → 非空？→ 是 → 下一轮
                  ↓ 否
              收束（等用户确认）
```

### 输出格式
见 `references/grill-protocol.md` § 四、输出格式（含设计树、访谈基调、破防场景回归、术语表、ADR）。

---

## 阶段二：架构对抗审查模式

**触发词**: "审查架构"、"找自我矛盾"、"架构破坏专家"、"这个设计有坑吗"

### 流程
1. 读取 `references/adversarial-review-checklist.md`
2. 确定审查范围（模块/切片/全架构）
3. 输出 Mermaid 依赖图（先扫描，后上色 red/yellow/green）
4. 按五大维度扫描：
   - 战略合理性（过度设计 vs 设计不足）
   - 逻辑闭环（断头路、CAP/BASE 矛盾）
   - 模块边界（循环依赖、跨层直调）
   - 生产生存（超时/重试/幂等/熔断/降级）
   - 资源隔离（Full GC 路径模拟）
5. 每个发现必须给出**可复现的破防场景**（触发条件 → 级联路径 → 全崩结果）
6. **审查发现的问题立即转化为拷问问题**，进入 Grill-Review 循环
7. 用户回答后更新设计，重新扫描，直到无新发现或用户叫停

### 输出格式
严格遵循 `vendor/architecture-breaker-review/SKILL.disabled.md` § 输出模板（Mermaid 依赖图上色、F1..Fn 编号、破防场景、Health Score 权重）。报告开头带 `Style: Brooks-Lint Iron Law + 对抗性破坏扫描`；本模式权重与 brooks-lint 不同，必须明确列出实际所用权重。

---

## 阶段三：代码质量扫描模式

**触发词**: "审查代码"、"PR review"、"技术债务"、"测试质量"

### 子模式路由

| 子模式 | 触发词 | 权威 SKILL |
|--------|--------|-----------|
| PR Review | "review 这个 PR"、"代码有没有问题" | `vendor/brooks-lint/skills/brooks-review/SKILL.disabled.md` |
| Architecture Audit | "模块依赖对吗"、"代码结构合理吗" | `vendor/brooks-lint/skills/brooks-audit/SKILL.disabled.md` |
| Tech Debt | "债务怎么还"、"哪里最痛" | `vendor/brooks-lint/skills/brooks-debt/SKILL.disabled.md` |
| Test Quality | "测试靠谱吗"、"覆盖率高但 bug 多" | `vendor/brooks-lint/skills/brooks-test/SKILL.disabled.md` |
| Health Dashboard | "整体质量怎么样"、"全面检查" | `vendor/brooks-lint/skills/brooks-health/SKILL.disabled.md` |
| Full Sweep | "自动修复所有问题" | `vendor/brooks-lint/skills/brooks-sweep/SKILL.disabled.md` |

### 通用流程
1. 读 `vendor/brooks-lint/skills/_shared/common.md`（Iron Law、Auto Scope Detection、Report Template）
2. 读 `.brooks-lint.yaml`（如存在）应用配置，校验规则见 `common.md` § Config Validation
3. 读权威风险定义：
   - 代码审查 → `vendor/brooks-lint/skills/_shared/decay-risks.md`（R1–R6）
   - 测试审查 → `vendor/brooks-lint/skills/_shared/test-decay-risks.md`（T1–T6）
   - **必须读 What Not to Flag 段**，否则误报泛滥
4. 按 Auto Scope Detection 自动检测范围（staged → diff → branch → 全项目）
5. 输出 Health Score（基础 100；Critical −15 / Warning −5 / Suggestion −1；下限 0）+ 四段式 Findings
6. 追加记录到 `.brooks-lint-history.json`，报告附 Trend 行

> 风险码索引与计分速查：`references/risk-catalog.md`

---

## 阶段四：发布就绪审查模式

**触发词**: "能上线吗"、"发布前检查"、"go/no-go"、"readiness"

### 流程
1. 读 `vendor/staff-engineer-mode/skills/staff-engineer-mode/references/routing-matrix.md` 做 exact-slug 路由
2. 根据发布类型选定主专家（含消歧）：
   - 功能发布 → `specialists/production-readiness-review.md`
   - 架构变更 → `specialists/architecture-decisions.md` + `production-readiness-review.md`
   - 数据库变更 → `specialists/database-operations.md`
   - 安全相关 → `specialists/secure-sdlc-and-threat-modeling.md`；逐 sink 输入防御 → `input-validation-and-injection-defense.md`
   - 发布节奏/灰度 → `specialists/progressive-delivery.md`、`rollout-plan.md`
3. 打开对应 `vendor/staff-engineer-mode/specialists/<slug>.md` 执行检查清单
4. 需要结构化产出时用 `vendor/staff-engineer-mode/skills/_shared/assets/templates/<name>.md`（如 `prr-checklist.md`、`rollout-plan.md`）
5. 输出结构化就绪报告（含阻塞项/风险项/建议项）；结论必须显式分级为 **GO / NO-GO / CONDITIONAL GO**

> 常见场景摘要：`references/engineering-checklists.md`（64 个专家的导航索引）

---

## 阶段五：文档产出模式

**触发词**: "写 PRD"、"写设计文档"、"写技术方案"、"写分析报告"

### 流程
1. 读 `vendor/product-lifecycle-workbench/skills/product-doc-writing/SKILL.disabled.md` 确定体裁
2. 打开对应权威模板全文（8 种）：
   `references/prd-document.md`、`competitive-analysis.md`、`software-design-document.md`、`design-system-specification.md`、`technical-documentation.md`、`marketing-copy.md`、`whitepaper-report.md`、`internal-memo.md`
   （均在 `vendor/product-lifecycle-workbench/skills/product-doc-writing/references/`）
3. 遵循 `references/writing-style.md` 的写作纪律（禁用清单、结构选择、视觉元素）
4. 产出后主动询问：是否需要转为 HTML/PDF 格式

---

## 共享规则（所有阶段通用）

### 语言规则
- 输出语言与用户查询语言一致
- 保留英文的：Iron Law 字段名（Symptom/Source/Consequence/Remedy）、书名、原则名、模板结构词（Findings/Summary/Critical/Warning/Suggestion）

### 报告通用头尾
各阶段报告统一使用此头尾；正文模板以对应阶段指针的 vendor 原文为准。

```markdown
# Grillkeeper — <模式名称>

**Mode:** [Grill / Architecture Breaker / PR Review / Architecture Audit / Tech Debt / Test Quality / Health Dashboard / Full Sweep / Production Readiness / Document Writing]
**Scope:** [文件/目录/范围描述]
**Health Score:** XX/100（如适用）
**Trend:** XX → XX (ΔN) over last N runs（如适用）
**Config:** .brooks-lint.yaml applied (N risks disabled, M paths ignored)（如适用）

[一句话总体定性]
```

```markdown
---
**生成时间**: <ISO 8601>
**生成工具**: Grillkeeper
**历史记录**: 已追加到 .brooks-lint-history.json（如适用）
```

### 读取预算
每阶段分两档：**必读最小集**（清单/索引级）与**命中后才读**（vendor 全文）。内循环每轮只对照融合层清单，禁止每轮重读 vendor 原文。

| 阶段 | 必读最小集 | 命中后才读 |
|------|-----------|-----------|
| 一·打磨访谈 | `grill-protocol.md` + `adversarial-review-checklist.md` | `vendor/grilling/SKILL.disabled.md`（首轮一次）、routing-matrix 对应行、命中 slug 的 specialist |
| 二·对抗审查 | `adversarial-review-checklist.md` | `vendor/architecture-breaker-review/SKILL.disabled.md`（首次） |
| 三·质量扫描 | `risk-catalog.md` + 子模式路由行 | `common.md` + 对应风险库（首次进入读一次） |
| 四·发布就绪 | `engineering-checklists.md` | 命中 slug 的 specialist + template |
| 五·文档产出 | `writing-style.md` | 命中体裁的 vendor 模板 |

### 落盘路径基准
所有产物（`CONTEXT.md`、`docs/adr/`、`.grillkeeper/session.md`、`.brooks-lint-*`、打磨记录）一律写入**用户当前工作区根目录**，运行前先确认工作区位置；禁止写入 skill 自身目录。

### 配置加载
- 文件名 `.brooks-lint.yaml`，schema 与校验规则见 `references/risk-catalog.md` § 项目配置（权威为 `vendor/brooks-lint/skills/_shared/common.md` § Project Config）
- 配置生效时在报告 `Scope` 行下追加 `Config: .brooks-lint.yaml applied (N risks disabled, M paths ignored)`

### 历史追踪
- 追加到 `.brooks-lint-history.json`：`{ date, mode, score, findings: { critical, warning, suggestion }, scope }`
- 报告含 Trend 行：`85 → 82 (−3) over last 3 runs`

### 修复模式（两层，别混）
- **`--fix` / "修复这些问题"** → 只读增强：读 `vendor/brooks-lint/skills/_shared/remedy-guide.md`，给每个 Remedy 补 Target/Action/Rationale 并按 `[quick-fix]/[guided]/[manual]` 分级，**不改文件、不生成 diff、不重算分**
- **"自动修复所有问题" / sweep** → 写文件：读 `vendor/brooks-lint/skills/brooks-sweep/`（SKILL.disabled.md + sweep-guide.md），先展示 pre-flight consent 等用户批准，再按 review → test → debt → audit 顺序应用 Safe + Extended-Safe 修复并跑测试验证

### 三问收敛（Post-Report Triage）
交互式会话专属（CI/headless 跳过）。报告输出后，按严重度从低到高逐条询问：
> accept / dismiss / defer / skip

- **Dismiss**: 给一行理由 → 写入 `.brooks-lint.yaml` 的 `suppress:` 段 → 后续降级为 info
- **Defer**: 同上并加 `expires: YYYY-MM-DD`（默认 90 天），到期恢复原严重度

---

## 快速命令速查

| 命令 | 等效于 |
|------|--------|
| `/grillkeeper grill` | 需求打磨模式 |
| `/grillkeeper break` | 架构对抗审查 |
| `/grillkeeper review` | PR/代码审查 |
| `/grillkeeper debt` | 技术债务评估 |
| `/grillkeeper test` | 测试质量审查 |
| `/grillkeeper health` | 四维健康看板 |
| `/grillkeeper sweep` | 全量扫描+自动修复 |
| `/grillkeeper ready` | 发布就绪审查 |
| `/grillkeeper doc prd` | 写 PRD |
| `/grillkeeper doc hld` | 写架构设计文档 |

---

## 文件索引

### 融合层（`references/`）

| 文件 | 用途 |
|------|------|
| `grill-protocol.md` | 访谈融合层（上游 frontier 机制 + 定基调根节点 + 即时审查叠加） |
| `adversarial-review-checklist.md` | 架构对抗审查清单（五维度） |
| `engineering-checklists.md` | 64 个专家的导航索引（摘要） |
| `risk-catalog.md` | R1–R6 / T1–T6 索引 + 计分与配置速查 |
| `writing-style.md` | 写作纪律 + 文档模板骨架摘要 |

### 权威层（`vendor/`）

| 目录 | 内容 | 文件数 |
|------|------|--------|
| `vendor/SOURCES.md` | 来源、版本、许可证、映射表 | 1 |
| `vendor/brooks-lint/` | 6 子模式 + 风险库 + 书目矩阵 + 修复 + 自定义风险 | 21 |
| `vendor/staff-engineer-mode/` | 64 专家 + 路由矩阵 + 71 模板 | 143 |
| `vendor/product-lifecycle-workbench/` | 8 文档模板全文 | 9 |
| `vendor/architecture-breaker-review/` | 原始方法论 | 1 |
| `vendor/grilling/` | **访谈机制权威**：设计树 / frontier / 问题格式 / 收束 | 1 |
| `vendor/domain-modeling/` | `CONTEXT.md` 术语表格式 + ADR 格式与判定 | 3 |
