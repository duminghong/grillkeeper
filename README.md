# Grillkeeper — 打磨访谈守门员

> **grill + gatekeeper**：一场不依不饶的打磨访谈，兼全流程质量守门员。
> 自包含的全周期质量守护 Skill，运行时不依赖任何外部 skill。

把"需求打磨"和"质量审查"从串行的两个阶段拧成一个内循环：**每轮访谈都内嵌即时审查，审查发现直接挂为设计树的新节点，随 frontier 推进，不可能被漏掉**。

- 中文名：**打磨访谈守门员**
- 本文受众是**人**（安装、评估、维护）；**agent 的行为细则以 [SKILL.md](SKILL.md) 为准**——本文件不重复其流程、原则与路由表

---

## 为什么存在

- **传统做法**：先打磨需求，再审查质量。两次切换、上下文断裂，审查发现经常回不去、被漏掉。
- **本 skill**：把审查塞进打磨的每一轮，发现变成设计树的新节点，跟着 frontier 走，漏不掉。
- **自包含**：能力来自仓库内快照（vendor 层）+ 融合导航层（references 层），运行时不调用任何外部 skill，也不受宿主环境同名 skill 干扰。

---

## 安装

把整个 `grillkeeper/` 目录放到 `.trae/skills/`（项目级）或 `~/.trae-cn/skills/`（用户级）下即可。若 `git clone` 后手动拷贝，请排除 `.git/`（用 `robocopy /XD .git` 或 `rsync --exclude .git`），只带 `SKILL.md`、`references/`、`vendor/`、`README.md`。

- vendor 内所有上游 `SKILL.md` 已改名为 `SKILL.disabled.md`，防止宿主递归扫描把它们注册成独立 skill（撞名 + 违反"禁止外部 skill 分流"）。
- `SKILL.md` 含 `disable-model-invocation: true`——自然语言**不会自动加载**本 skill，须显式调用 `/grillkeeper`。

---

## 能力一览

各能力的触发词、流程、输出格式、评分权重见 [SKILL.md](SKILL.md) 的「阶段路由表」与五个阶段章节。

| 能力 | 一句话 |
|------|--------|
| 需求打磨访谈（核心） | 设计树 + frontier 分轮提问，每轮内嵌即时审查，发现挂为新节点 |
| 架构对抗审查 | 五维度扫描 + 可复现破防场景 + Mermaid 依赖图 |
| 代码/测试质量扫描 | Brooks-Lint 六子模式，R1–R6 / T1–T6 风险库 |
| 发布就绪审查 | Staff-Engineer 64 专家路由，输出 GO / NO-GO / CONDITIONAL GO |
| 文档产出 | 8 种产品文档模板 |

---

## 如何调用

显式 `/grillkeeper <子命令>`。下表仅为速查，**权威定义在 [SKILL.md](SKILL.md) §阶段路由表 / §快速命令速查**，二者若不一致以 SKILL.md 为准。

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

## 目录结构

```
grillkeeper/
├── SKILL.md                 # 主入口（agent 行为权威）：阶段路由 + 五阶段流程 + 共享规则
├── README.md                # 本文件（人读入口）：是什么 / 安装 / 导航
├── references/              # 融合层：索引、跨能力融合、访谈内循环（5 个文件）
└── vendor/                  # 权威层：6 个上游来源的快照（含路径本地化补丁 / 179 文件 / ~1.35 MB）
    ├── SOURCES.md           # 来源、版本、许可证、能力映射表
    ├── brooks-lint/         # 代码/测试质量诊断六模式（MIT © hyhmrright）
    ├── staff-engineer-mode/ # 64 个工程专家 + 路由矩阵 + 71 模板（MIT © sirmarkz）
    ├── product-lifecycle-workbench/  # 8 种产品文档模板（许可未声明）
    ├── architecture-breaker-review/  # 架构破坏专家方法论（许可未声明）
    ├── grilling/            # 访谈机制权威：设计树 + frontier（许可未声明）
    └── domain-modeling/     # CONTEXT.md 术语表 + ADR 格式与判定
```

---

## 单一真源（哪份文档说了算）

| 文档 | 受众 | 承载 | 权威性 |
|------|------|------|--------|
| [SKILL.md](SKILL.md) | Agent | 阶段路由、五阶段流程、共享规则、落盘路径 | **行为唯一真源** |
| 本 README | 人 | 是什么、安装、目录结构、许可、升级 | 不承载行为权威 |
| [vendor/SOURCES.md](vendor/SOURCES.md) | 维护者 | 来源、版本、许可证、本地修改清单、升级步骤 | **来源唯一真源** |

**无重复规则**：同一主题只在一处定义。README 不复制 SKILL 的原则、流程、路由表；行为细节一律指向 SKILL.md，避免两份文档漂移。

---

## 双层约定

`vendor/` 是权威源（canonical）——上游内容快照，除 [vendor/SOURCES.md](vendor/SOURCES.md) §本地修改（fork diffs）列出的**路径本地化补丁**外逐字；`references/` 只做导航与跨能力融合，不重复定义诊断问题、评分公式或检查项。**冲突时以 vendor 为准**（唯一例外：上述路径补丁属有意偏离）。落盘路径基准与产物布局见 [SKILL.md](SKILL.md) §落盘路径基准。

---

## 许可证与来源

- `brooks-lint/`（MIT © 2025 hyhmrright）、`staff-engineer-mode/`（MIT © sirmarkz）：LICENSE 与 THIRD_PARTY_NOTICES 已随 vendor 原样保留
- `product-lifecycle-workbench/`、`architecture-breaker-review/`、`grilling/`、`domain-modeling/`：**上游未声明许可，仅作本地参考；公开发布前需确认授权**
- 完整来源、版本、能力映射见 [vendor/SOURCES.md](vendor/SOURCES.md)

## 升级

vendor 副本是快照，不自动跟随上游。升级步骤（重铺 → **重放 fork diffs** → grep 核对 + 打包改名）见 [vendor/SOURCES.md](vendor/SOURCES.md) §上游更新。**勿凭本段记忆操作**——只重铺不重放补丁会静默丢掉全部路径本地化。
