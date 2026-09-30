# Grillkeeper — 打磨访谈守门员

> **grill + gatekeeper**：一场不依不饶的打磨访谈，兼全流程质量守门员。
> 自包含的全周期质量守护 Skill，运行时不依赖任何外部 skill。

把"需求打磨"和"质量审查"从串行的两个阶段拧成一个内循环：**每轮访谈都内嵌即时审查，审查发现直接挂为设计树的新节点，随 frontier 推进，不可能被漏掉**。

- 中文名：**打磨访谈守门员**
- 安装：把整个 `grillkeeper/` 目录放到 `.trae/skills/`（项目级）或 `~/.trae-cn/skills/`（用户级）下即可。若 `git clone` 后手动拷贝，请排除 `.git/`（用 `robocopy /XD .git` 或 `rsync --exclude .git`），只带 `SKILL.md`、`references/`、`vendor/`、`README.md`
- 说明：vendor 内所有上游 `SKILL.md` 已改名为 `SKILL.disabled.md`，防止宿主递归扫描把它们注册成独立 skill（撞名 + 违反"禁止外部 skill 分流"）
- 触发：`SKILL.md` 含 `disable-model-invocation: true`——自然语言**不会自动加载**本 skill，须显式调用（`/grillkeeper grill` 等快速命令）；下文各处的"触发词"是加载之后的阶段路由依据

---

## 目录结构

```
grillkeeper/
├── SKILL.md                 # 主入口：阶段路由 + 五阶段流程 + 共享规则
├── README.md
├── references/              # 融合层：索引、跨能力融合、访谈内循环（5 个文件）
└── vendor/                  # 权威层：6 个上游来源的逐字副本（179 文件 / ~1.35 MB）
    ├── SOURCES.md           # 来源、版本、许可证、能力映射表
    ├── brooks-lint/         # 代码/测试质量诊断六模式（MIT © hyhmrright）
    ├── staff-engineer-mode/ # 64 个工程专家 + 路由矩阵 + 71 模板（MIT © sirmarkz）
    ├── product-lifecycle-workbench/  # 8 种产品文档模板（许可未声明）
    ├── architecture-breaker-review/  # 架构破坏专家方法论（许可未声明）
    ├── grilling/            # 访谈机制权威：设计树 + frontier（许可未声明）
    └── domain-modeling/     # CONTEXT.md 术语表 + ADR 格式与判定
```

**双层约定**：`vendor/` 是权威源（canonical），逐字复制、零改写失真；`references/` 只做导航与跨能力融合，不重复定义诊断问题、评分公式或检查项。**冲突时以 vendor 为准**。详见 [vendor/SOURCES.md](vendor/SOURCES.md)。

---

## 能力总览

### 1. 需求打磨访谈（核心能力）

**触发词**："打磨需求"、"拷问设计"、"这个方案靠谱吗"、"帮我完善计划"

访谈机制以 `vendor/grilling/SKILL.disabled.md` 为权威：

- **设计树**：把需求/设计映射成一棵树，每个决策分叉出挂在它下面的子决策
- **frontier（前沿）分轮**：每轮问完"前置条件已全部落定"的所有问题——即不用猜尚未听到的答案就能问的问题；依赖本轮未决问题的问题属于后续轮次
- **问题格式固定**：编号 + 问题正文 + **推荐答案**（把开放式拷问变成可反驳的判断）
- **找事实是 agent 的职责**：需要环境事实时派 sub-agent 查，不阻塞，只让下游问题等
- **收束 = frontier 为空**：设计树每个分支都走过，没有任何东西被默默假设；用户确认前不动手实施

**Grillkeeper 在上游机制上叠加的增强**：

| 增强 | 说明 |
|------|------|
| **定基调根节点** | 第一轮必问"是否需要向后兼容"+"按最彻底还是最小改动"，模糊回答逼二选一；答案改变后续所有轮次口径 |
| **即时审查内循环** | 每轮收到回答后立即做对抗性扫描（共 14 项：断头路/自相矛盾/越权依赖 3 项每轮必跑，CAP 矛盾/破防场景回归/术语钉住/可测性/复用-YAGNI 等 11 项信号触发），🔴/🟡 发现**挂为设计树新节点**随 frontier 推进；15 轮未收束则强制盘点残余节点 |
| **会话状态落盘** | 基调落定后创建 `.grillkeeper/session.md`，每轮追加 frontier 快照与审查发现；中断后读文件重建状态，不凭记忆续跑 |
| **条件触发能力** | 命中信号才跑：容量/成本粗算、威胁建模、数据契约检查、原型验证、历史记忆回查 |
| **禁止项** | 内循环不跑：完整 Mermaid 依赖图重绘、Health Score 每轮重算、全量十二书风险扫描 |
| **领域建模落盘** | 术语当场钉入 `CONTEXT.md`（含 `_Avoid_` 同义词）；ADR 克制——"难逆转 + 无上下文则困惑 + 真实权衡"三条件全真才写 |

### 2. 架构对抗审查

**触发词**："审查架构"、"找自我矛盾"、"这个设计有坑吗"

权威源 `vendor/architecture-breaker-review/SKILL.disabled.md`。五维度扫描 + Mermaid 依赖图（红/黄/绿上色）：

1. **战略合理性** — 过度设计 vs 设计不足
2. **逻辑闭环** — 断头路、CAP/BASE 矛盾
3. **模块边界** — 循环依赖、跨层直调、同名异义
4. **生产生存** — 超时/重试/幂等/熔断/降级矩阵、TraceId、黄金指标
5. **资源隔离** — Full GC 路径模拟

本模式专属规则：

- 发现按 **F1..Fn 编号**，每个 Critical 附**可复现破防场景**（触发条件 → 级联路径 → 全崩结果）
- Health Score 权重 **−15 / −5 / −2**（Suggestion 为 −2，与 brooks-lint 的 −1 不同，按模式各从其源，报告中明确列出权重）
- **只读审查**：不写代码、不改文件；好的基建显式 credit，同类问题不重复挑刺
- 审查发现立即转化为拷问问题，回到打磨循环

### 3. 代码/测试质量扫描（Brooks-Lint 六模式）

基于十二本经典软件工程书的诊断体系（Brooks / McConnell / Fowler / Martin / Pragmatic Programmer / Evans / Ousterhout / Software Engineering at Google / Meszaros / Osherove / Feathers / How Google Tests Software）。

**Iron Law**：未完成风险诊断前不得给修复建议；每个发现必须 `Symptom → Source → Consequence → Remedy` 四段式，否则是噪音。

| 子模式 | 触发词 | 输出 |
|--------|--------|------|
| PR Review | "review 这个 PR" | PR 审查报告 |
| Architecture Audit | "模块依赖对吗"、"代码结构合理吗" | 模块依赖图 + 风险清单 |
| Tech Debt | "债务怎么还" | 债务优先级矩阵 |
| Test Quality | "测试靠谱吗" | 测试质量报告 |
| Health Dashboard | "整体质量怎么样" | 四维健康看板 |
| Full Sweep | "自动修复所有问题" | 修复日志 + 残余问题 |

- **风险库**：6 大代码衰减风险 R1–R6（认知过载 / 变更传播 / 知识重复 / 偶然复杂度 / 依赖失序 / 领域模型扭曲）+ 6 大测试衰减风险 T1–T6（测试晦涩 / 脆弱 / 重复 / Mock 滥用 / 覆盖率幻觉 / 架构错配），每个含症状明细、书目溯源、严重度指南、**What Not to Flag 反噪音段**
- **Health Score**：基础 100；Critical −15 / Warning −5 / Suggestion −1；下限 0
- **项目配置**：`.brooks-lint.yaml`（disable / severity / ignore / focus / custom_risks / suppress）
- **历史趋势**：`.brooks-lint-history.json`，报告附 Trend 行（如 `85 → 82 (−3) over last 3 runs`）
- **Post-Report Triage**：交互式逐条 accept / dismiss / defer / skip；dismiss/defer 写入 suppress，支持到期复活

**修复分两层，别混**：

| 动作 | 性质 | 权威源 |
|------|------|--------|
| `--fix` / "修复这些问题" | **只读**：给每个 Remedy 补 Target/Action/Rationale，按 `[quick-fix]/[guided]/[manual]` 分级，附 Fix Summary 表；不改文件、不生成 diff、不重算分 | `vendor/brooks-lint/skills/_shared/remedy-guide.md` |
| "自动修复所有问题" / sweep | **写文件**：pre-flight consent 批准 → review→test→debt→audit 四维顺序应用 Safe + Extended-Safe 修复 → 测试验证 → 迭代收敛 | `vendor/brooks-lint/skills/brooks-sweep/` |

### 4. 工程场景路由（Staff-Engineer 专家库）

**触发词**："能上线吗"、"发布前检查"、"go/no-go"

- **64 个专家检查清单**全文：`high-availability-design`、`backup-and-recovery`、`database-operations`、`event-workflows`、`slo-and-error-budgets`、`secure-sdlc-and-threat-modeling`、`llm-application-security`、`migration-and-deprecation`、`progressive-delivery`、`oncall-health`、`agent-pr-review`……
- **60+ 条 exact-slug 路由消歧规则**：例如"定时任务漏跑 → `scheduled-job-reliability`，而数据集新鲜度/回填 → `data-pipeline-reliability`"、"客户端安全 vs 逐 sink 输入校验 vs LLM 安全"三分，防止路由到语义相近但错误的专家
- **71 个结构化产出模板**：PRR 检查表、发布计划、威胁模型、事故复盘、ADR、容量规划……

`references/engineering-checklists.md` 提供 12 张常见场景的入门摘要，每张标题即真实 specialist slug，命中后直达 vendor 原文。

### 5. 文档产出

**触发词**："写 PRD"、"写设计文档"、"写技术方案"

8 种产品文档模板全文（vendor 层）：PRD、竞品分析、软件设计文档、设计系统规范、技术文档、营销文案、白皮书、内部备忘录。配套写作纪律（禁用清单、结构选择、视觉元素，见 `references/writing-style.md`）。

---

## 核心原则

1. **Iron Law** — 每个发现必须 `Symptom → Source → Consequence → Remedy` 四段式
2. **对抗性优先** — 先找断头路、自我矛盾、越权依赖，再谈优点
3. **打磨审查一体化** — 审查发现挂为设计树新节点，随 frontier 推进，不可能被漏掉
4. **权威源优先** — 诊断问题、严重度指南、评分公式、检查项一律读 vendor 原文，不凭印象发挥
5. **文档即交付** — 审查结论必须能沉淀为结构化文档

---

## 快速命令

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

## 许可证与来源

- `brooks-lint/`（MIT © 2025 hyhmrright）、`staff-engineer-mode/`（MIT © sirmarkz）：LICENSE 与 THIRD_PARTY_NOTICES 已随 vendor 原样保留
- `product-lifecycle-workbench/`、`architecture-breaker-review/`、`grilling/`、`domain-modeling/`：**上游未声明许可，仅作本地参考；公开发布前需确认授权**
- 完整来源、版本、能力映射见 [vendor/SOURCES.md](vendor/SOURCES.md)

## 升级

vendor 副本是快照，不自动跟随上游。升级时重新复制上游目录覆盖 `vendor/<origin>/`，并同步更新 `vendor/SOURCES.md` 中的版本号。
