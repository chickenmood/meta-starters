---
name: '{{project_slug}}-openspec-dev'
description: '{{project_name}} 工程的 OpenSpec 增量开发流程 — 基于 openspec/project.md + openspec/AGENTS.md,按 Change 循环跑"propose → apply → archive",而不是一次性大爆炸。当用户说"为 {{project_name}} 提 change"、"跑 {{project_name}} 的 change #NNN"、"开始 {{project_name}} 的 OpenSpec 增量开发"、"为 {{project_name}} 加个新 capability"时调用。'
---

<!--
TEMPLATE: <project>-openspec-dev/SKILL.md
=============================================

PURPOSE
    This file is a TEMPLATE, not an active SKILL. It is read by the
    sdd-migrate SKILL, which substitutes {{var}} placeholders to emit
    the actual per-project OpenSpec-era SKILL.md at:

        ~/.claude/skills/{{project_slug}}-openspec-dev/SKILL.md

    This template is the OpenSpec-era counterpart of
    PROJECT_SKILL_TEMPLATE.md (SpecKit-era). The two are intentionally
    separate so that SpecKit-era projects continue to use spec-driven-dev
    while OpenSpec-era projects use openspec-dev.

    The user then copies (or symlinks) that emitted file into the project's own
    .claude/skills/ folder so it auto-loads inside the project repo.

PLACEHOLDER TABLE
    Mirror from sdd-migrate Stage 7 emit output:

    | Placeholder                   | Source (sdd-migrate Stage)            | Example                          |
    |-------------------------------|---------------------------------------|----------------------------------|
    | {{project_name}}              | Stage 1 project_meta.name             | "LingShu"                        |
    | {{project_slug}}              | Stage 1 project_meta.slug             | "lingshu"                        |
    | {{one_liner}}                 | Stage 1 project_meta.one_liner        | "JDK 8+ Agent 引擎,9 Slot SPI"   |
    | {{openspec_dir}}              | 固定值                                | "openspec/"                      |
    | {{capabilities}}              | Stage 2 派生 capability 列表          | "[agent-bootstrap, ...]"         |
    | {{active_changes}}            | Stage 3 派生 🟡 change 列表          | "[022-spring-ai-tool, ...]"      |
    | {{tooling_check_commands}}    | 固定值(OpenSpec 工具链)             | "command -v openspec\ncommand -v git" |
    | {{verify_command_pattern}}    | 从原 spec-driven-dev 模板继承        | "mvn test" / "pytest"            |
    | {{readme_path}}               | 项目根 README 路径                   | "<repo>/README.md"               |
    | {{default_branch}}            | 项目 git 约定                         | "main"                           |
    | {{last_updated_date}}         | emit 时日期                           | "2026-09-28"                     |
    | {{version}}                   | emit 时版本(起始 1.0.0)              | "1.0.0"                          |

DESIGN CONTRACT
    sdd-migrate MUST emit a SKILL.md where every {{var}} above has
    been substituted with a non-empty concrete value. Missing placeholder =
    sdd-migrate returns to retry.

FRONTMATTER IS INTENTIONAL
    This template has frontmatter at byte 0 (so VSCode Markdown Preview renders
    it as a 2-column table, matching spec-driven-bootstrap/SKILL.md's visual
    style). The {{var}} placeholders in `name:` / `description:` prevent
    Claude Code's SKILL auto-loader from matching any real user phrase against
    this template.

    sdd-migrate Stage 7 strips the HTML comment block above (between the
    closing `---` of frontmatter and the `# {{project_name}}-openspec-dev`
    heading) during emit, so it does NOT appear in the per-project SKILL.md.
-->

# {{project_name}}-openspec-dev

## 何时调用

- 用户说"为 {{project_name}} 提 change #NNN" / "propose change for {{project_name}}"
- 用户说"跑 {{project_name}} 的 change #NNN 的 apply" / "apply openspec change for {{project_name}}"
- 用户说"为 {{project_name}} 加个新 capability" / "为 {{project_name}} 改已有 capability"
- 用户粘贴了 {{capabilities}} 列表让逐个实施 change
- 新窗口启动 + 用户提到"{{project_name}} OpenSpec 增量开发" / "接 {{project_name}} 工程"

## Pre-flight(每次会话首 turn 必跑 — 防"半路才发现缺工具"白烧 token)

**严格 AND 校验**:OpenSpec 工具链 binary **全部必须已装**,任一缺失**强制中止**(不允许"挑一个装就够")。装法见 `_meta-starters/README.md` 的「工具链安装指引」。

```bash
# 自动派生(每个 binary 一行,缺失 echo MISSING_X)
{{tooling_check_commands}}
```

**如何解读输出**:
- 全部 silent → 工具链齐备,继续后续 Step
- 任一 `MISSING_X` → **强制中止**,提示用户装对应工具(指向 README 的「工具链安装指引」),**不让硬跑** —— 半路失败会浪费 token

**Placeholder 来源**:`{{tooling_check_commands}}` 由 `sdd-migrate` Stage 7 派生,固定为 OpenSpec + git 两条:
- `command -v openspec`(OpenSpec CLI)
- `command -v git`(归档/分支管理)

## 核心原则(4 条不妥协)

```
不是 一次写完所有 Change
是   一次改 {{openspec_dir}}/specs/<cap>/spec.md(SoT) → 按 Change 循环:
     /opsx:propose → /opsx:apply → /opsx:verify → /opsx:archive
```

1. **`{{openspec_dir}}/specs/<cap>/spec.md` 是 single source of truth**:任何 capability 改动必读对应 capability 的 spec,禁止凭印象改
2. **`{{openspec_dir}}/AGENTS.md` 是项目"宪法"**:跨 capability 改它必须走 RFC,不能随手改
3. **Change 边界 = 验收边界**:每个 Change 自带 tasks + AC 黑盒验证,verify 全过才 archive
4. **新窗口不跨 Change 累积 context**:每次开窗从 `{{openspec_dir}}/specs/<cap>/spec.md` 重新读对应 capability,避免 staleness

## 关键输入(每次 Change 开始前必读)

| 输入 | 路径 | 必读? |
|---|---|---|
| 项目上下文 | `{{openspec_dir}}/project.md` | ✅ 完整读 + 项目原则对齐 |
| AI 章程 | `{{openspec_dir}}/AGENTS.md` | ✅ 完整读 + 硬约束 + 反模式 |
| 现有 capability spec(若改已有) | `{{openspec_dir}}/specs/<cap>/spec.md` | ✅ 读对应 capability |
| 待 apply change 列表 | `{{openspec_dir}}/changes/` | ✅ ls 整个目录 |
| 项目 README | `{{readme_path}}` | ✅ 含已合 capability 历史 / 已 archive change 路线图 — 与 `{{openspec_dir}}/changes/archive/` 互补(archive 是实时,README 是公开宣告面) |
| 验证手段 | `{{verify_command_pattern}}` | ✅ AC 通过的判定方式 |
| Git 默认分支 | `{{default_branch}}` | ✅ archive 后合入目标 |

## 工作流(3 阶段 + 验证,Step -1 按需)

### Step -1:环境探测

```bash
cd <{{project_slug}} 主仓根目录>
test -d {{openspec_dir}} || echo "MISSING_OPENSPEC_DIR"
test -f {{openspec_dir}}/config.yaml || echo "MISSING_CONFIG"
test -f "{{readme_path}}" || echo "WARN_MISSING_README(项目根 README 不存在 — 外部协作者/读者无宣告面,建议创建)"
```

任一输出非空 → 跑 Step 0 环境初始化;否则跳过。`WARN_MISSING_README` 仅警告,不阻塞。

### Step 1:`/opsx:propose <change-id>`(起草 Change Proposal)

```bash
openspec new <change-id> --capability <cap-name>  # 或 /opsx:new
# 触发 /opsx:ff 自动草稿,或 /opsx:continue 手动填写
```

填 4 段(OpenSpec 强约束):
- **Why**(1-2 句话说明问题,50-1000 字符)
- **What Changes**(bullet,breaking 标 `BREAKING`)
- **Capabilities**(New / Modified,kebab-case)
- **Impact**(受影响代码/API/依赖,≤5 个文件)

生成产物:
```
{{openspec_dir}}/changes/<change-id>/
├── proposal.md        ← 4 段必填
├── design.md          ← 可选(技术设计)
├── tasks.md           ← 实施 checklist
└── specs/
    └── <cap-name>/
        └── spec.md    ← Delta Spec(ADDED/MODIFIED/REMOVED/RENAMED Requirements)
```

**Delta Spec 写作规范**(OpenSpec 强约束):
- Requirement 文本必含 `SHALL` 或 `MUST`(RFC 2119)
- 每个 Requirement 必含 ≥ 1 个 `#### Scenario:`
- Scenario 用 `**GIVEN**` / `**WHEN**` / `**THEN**` / `**AND**` 4 段
- Requirement ≤ 500 字符,Purpose 50-1000 字符
- 跑 `openspec validate --strict` 必须无 ERROR

### Step 2:`/opsx:apply <change-id>`(实施 + 验证)

按 tasks.md 逐项实现:
- 每个 T-NN 一个 commit(commit message 走 `{{project_name}} 约定`)
- **必含 validate task**(跑 `openspec validate --strict`)
- **必含 AC verify task**(对应 proposal.md 的 Capabilities 影响范围)
- 验证命令模板:`{{verify_command_pattern}}`

```bash
# 跑 OpenSpec 自身验证
openspec validate <change-id> --strict

# 跑 AC 黑盒
{{verify_command_pattern}}
```

任一不过 → 修复后重跑,不进入 Step 3。

### Step 3:`/opsx:verify <change-id>`(AC 黑盒最终验证)

二次验证:
- [ ] `openspec validate <change-id> --strict` 无 ERROR
- [ ] 跑完所有 AC-NN 黑盒 + 贴验证输出到 PR body
- [ ] `{{openspec_dir}}/specs/<cap>/spec.md` 已同步更新(若 ADDED → spec.md 加对应 Requirement;若 MODIFIED → 改原 Requirement 文本;若 REMOVED → 删原 Requirement)
- [ ] PR 标题 + body 符合 `{{project_name}} 约定`
- [ ] `{{openspec_dir}}/AGENTS.md` §风险登记 更新(若有 R-XX 缓解)

### Step 4:`/opsx:archive <change-id>`(归档)

```bash
openspec archive <change-id>
```

副作用:
- `{{openspec_dir}}/changes/<change-id>/` → `{{openspec_dir}}/changes/archive/YYYY-MM-DD-<change-id>/`
- 自动写入归档元数据(日期 + 实施者 + AC 验证输出)
- 若 MODIFIED/REMOVED/RENAMED,`{{openspec_dir}}/specs/<cap>/spec.md` 已合并 delta
- 后续 git commit 到 `{{default_branch}}`

**反向同步**(archive 后必跑 — archive 是工程 tracker,README 是公开宣告面,两者必须同步):
- [ ] `{{readme_path}}`「已合 capability / 已 archive change」段追加本 Change(若 README 存在该段)
- [ ] `{{openspec_dir}}/AGENTS.md` §风险登记 更新(若有 R-XX 缓解)

**禁止**:
- ❌ 直接改 `{{openspec_dir}}/specs/<cap>/spec.md` 而不写 delta spec(等效 SpecKit-era 跳宪章)
- ❌ `/opsx:apply` 中不跑 verify 就 `/opsx:archive`(跳过 Step 2 → Step 3)
- ❌ 归档前不跑 `openspec validate --strict`(降级验收)
- ❌ 跨 Change 改 `{{openspec_dir}}/AGENTS.md`(不走 RFC)
- ❌ 把 Proposal 直接 commit 到 `{{default_branch}}`(必须 feature branch + review)

---

## 当前 Capability 列表(派生自 sdd-migrate Stage 2)

> **首次调用时**读 `{{openspec_dir}}/specs/`,把 kebab-case 目录名抄进来。后续用户加 capability 后,本段需手动追加。

| Capability | Source Story | AC 数 |
|---|---|---|
{{capabilities}}

## 当前 Active Change 列表(派生自 sdd-migrate Stage 3)

> **首次调用时**读 `{{openspec_dir}}/changes/`,把非 `archive/` 子目录的改动抄进来。后续 archive 后需手动移除。

| Change ID | Status | Source Story | Capabilities Affected |
|---|---|---|---|
{{active_changes}}

---

## 反模式(看到立刻停)

- ❌ 跳过 `/opsx:propose` 直接改 `{{openspec_dir}}/specs/<cap>/spec.md`(绕开审计)
- ❌ 改 `{{openspec_dir}}/specs/` 而不写 delta spec → 失去"为什么这么改"的可解释性
- ❌ `/opsx:apply` 中不跑 verify 就 `/opsx:archive`(降级验收)
- ❌ 跳过 `openspec validate --strict` 提交 PR(validate 必跑)
- ❌ 跨 Change 改 `{{openspec_dir}}/AGENTS.md`(不走 RFC 流程)
- ❌ 把 `{{openspec_dir}}/changes/<change-id>/` 直接 commit 到 `{{default_branch}}`(必须 feature branch)
- ❌ 改 `{{openspec_dir}}/specs/<cap>/spec.md` 但不 archive change(SoT 与 change 历史脱钩)
- ❌ Capability 名含大写 / 含 `_`(OpenSpec kebab-case 强约束)

## 输出检查清单(每个 Change archive 前自检)

- [ ] `{{openspec_dir}}/changes/<change-id>/proposal.md` 4 段全填(Why / What / Capabilities / Impact)
- [ ] `{{openspec_dir}}/changes/<change-id>/specs/<cap>/spec.md` delta spec 4 选 1 段 + Requirement 含 SHALL/MUST + Scenario 含 GIVEN/WHEN/THEN
- [ ] `openspec validate <change-id> --strict` 无 ERROR
- [ ] `{{verify_command_pattern}}` 全过(贴验证输出到 PR body)
- [ ] `{{openspec_dir}}/specs/<cap>/spec.md` 已同步(若 ADDED/MODIFIED/REMOVED/RENAMED)
- [ ] PR 标题 + body 符合 `{{project_name}} commit message 约定`
- [ ] `{{openspec_dir}}/AGENTS.md` §风险登记 更新(若有 R-XX 缓解)
- [ ] 已 archive 到 `{{openspec_dir}}/changes/archive/YYYY-MM-DD-<change-id>/`
- [ ] `{{readme_path}}`「已合 capability / 已 archive change」段已追加本 Change(若 README 存在该段)

## 相关文档(单 source of truth 链)

```
{{openspec_dir}}/project.md         ← 项目上下文(SKILL 不重复展开,引用即可)
{{openspec_dir}}/AGENTS.md          ← AI 工作流章程(改必 RFC)
{{openspec_dir}}/specs/<cap>/spec.md ← Source of Truth(改必走 Change Proposal)
{{openspec_dir}}/changes/<change-id>/  ← 进行中的 Change Proposal
{{openspec_dir}}/changes/archive/   ← 已归档 Change
```

修改优先级:**改 capability 必走 change proposal**(Step 1-4 起来),不直接改 SoT。

**OpenSpec 与 SpecKit 的关键差异**:
| 维度 | SpecKit-era | OpenSpec-era |
|---|---|---|
| 工具 | `/specify /plan /tasks` | `/opsx:propose /apply /verify /archive` |
| 制品目录 | `specs/<NNN>-<slug>/` | `{{openspec_dir}}/specs/<cap>/` + `{{openspec_dir}}/changes/<change-id>/` |
| 真理源 | `<project>_design.md` | `{{openspec_dir}}/specs/<cap>/spec.md`(SoT 是历史沉淀,不是项目真理) |
| 状态记录 | `ROADMAP.md`(手维护) | `{{openspec_dir}}/changes/archive/`(自动) |
| AC 黑盒 | AC-NN(可 grep) | Scenario Spec 内 GIVEN/WHEN/THEN(自描述) |

## 元 SKILL 关系(本 SKILL 是被生成的,不是手写的)

- 本 `{{project_slug}}-openspec-dev/SKILL.md` 由 `sdd-migrate` SKILL 从**本模板**(`~/.claude/skills/new-project-coach/OPENSPEC_PROJECT_SKILL_TEMPLATE.md`)emit 出来
- 上游:`spec-driven-bootstrap` SKILL 产出 SpecKit 4 件套 → `sdd-migrate` 迁移到 OpenSpec 制品
- 下游:每次开窗自动加载本 SKILL,不再走上游
- **链路**:`{{openspec_dir}}/project.md + AGENTS.md → specs/<cap>/spec.md → changes/<change-id>/ → archive/`

---

**Last updated**: {{last_updated_date}}
**Version**: {{version}}
(由 sdd-migrate emit;若 project.md / AGENTS.md 改了,**重跑 sdd-migrate** 生成新版本,**不要直接编辑本文件**)

---

## 更新日志(本模板自身的版本)

- **v1.2 → v1.3**: +frontmatter `name:` / `description:` 用 YAML 单引号包裹(让 `{{var}}` 占位符在 VSCode Markdown Preview 中合法 parse — `{{` 不是 YAML scalar 起始,裸字段会让 frontmatter 渲染为纯文本而非 2 列表格)— 修「VSCode 报 `Failed to parse frontmatter — Unexpected scalar at node end`」latent bug
- **v1.1 → v1.2**: +frontmatter 移到 byte 0(让 VSCode Markdown Preview 渲染为 2 列表格)+ HTML comment 下移到 frontmatter 之后(改写警告为 "FRONTMATTER IS INTENTIONAL")+ sdd-migrate Stage 7 emit bash + 占位符表对齐到 12 个 + 输出检查清单 `11 个 → 12 个` — 修「模板 frontmatter 渲染为纯文本」+「emit 时 HTML comment 污染 per-project SKILL.md」两条 latent bug
- **v1.0 → v1.1**: +`{{readme_path}}` 占位符 + 关键输入加 README + Step -1 加 README 存在性 WARN check + Step 4 反向同步 + 输出检查清单 — 补"棕地阶段 README 仍含已合 capability / 已 archive change 路线图,与 `{{openspec_dir}}/changes/archive/` 互补"gap
- v1.0: 初版