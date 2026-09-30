---
name: '{{project_slug}}-spec-driven-dev'
description: '{{project_name}} 工程的渐进式迭代开发流程 — 基于 {{project_name}}_design.md 设计文档,按 {{iteration_unit}} 循环跑"明确范围 → 实施 → 验证 → 合入",而不是一次性大爆炸。当用户说"开始 {{project_name}} 的 {{iteration_unit}} {{iteration_id_pattern}}"、"按 {{project_name}}_design.md 推 {{iteration_unit}}"、"跑 {{project_name}} 的 AC 验证"、"接 {{project_name}} 工程"时调用。'
---

<!--
TEMPLATE: <project>-spec-driven-dev/SKILL.md
=============================================

PURPOSE
    This file is a TEMPLATE, not an active SKILL. It is read by the
    spec-driven-bootstrap SKILL, which substitutes {{var}} placeholders to emit
    the actual per-project SKILL.md at:

        ~/.claude/skills/{{project_slug}}-spec-driven-dev/SKILL.md

    The user then copies (or symlinks) that emitted file into the project's own
    .claude/skills/ folder so it auto-loads inside the project repo.

PLACEHOLDER TABLE
    Mirror from design-doc-coach 5-dimension interview output:

    | Placeholder                   | Source (dimension / question)             | Example                          |
    |-------------------------------|-------------------------------------------|----------------------------------|
    | {{project_name}}              | identity (project name)                   | "FooFlow"                        |
    | {{project_slug}}              | derived from project_name (kebab-case)    | "fooflow"                        |
    | {{iteration_unit}}            | decomposition (dim 3) / iteration term    | "Story" / "Iteration" / "Task"   |
    | {{iteration_id_pattern}}      | decomposition (dim 3) / id pattern        | "#NNN" / "vN.M.K" / "iter-NNN"   |
    | {{iteration_split_rules}}     | decomposition (dim 3) / split thresholds  | "≤5 核心文件 / ≤3 ErrorCode"    |
    | {{iteration_unit_id}}        | decomposition (dim 3) / 具体 iteration id(每次 emit 时具体值,如 `#001` / `v1.0.0`) | "#001"                          |
    | {{tech_stack_summary}}        | tech stack (dim 1)                        | "Python 3.12 + FastAPI + uv"     |
    | {{build_command}}             | tech stack (dim 1) / initial validator    | "uv sync" / "mvn validate"       |
    | {{test_command}}              | validation (dim 4) / unit test runner     | "pytest" / "mvn -pl core test"   |
    | {{verify_command_pattern}}    | validation (dim 4) / AC verifier template | "pytest -k ac_nn" / "mvn test -Dtest=AC_NN_*" |
    | {{tooling_check_commands}}    | toolchain (auto-derived from §2 / §6)      | "command -v mvn \|\| echo MISSING_MVN\ncommand -v specify \|\| echo MISSING_SPECIFY" (multi-line bash) |
    | {{roadmap_path}}              | 工程 tracker 路径                          | `{{project_slug}}/specs/ROADMAP.md` |
    | {{readme_path}}               | 项目根 README 路径(含 Story 路线图 / 历史段) | `<repo>/README.md`              |
    | {{commit_message_format}}     | git convention (project-specific)         | "feat(scope): Story #NNN ..."    |
    | {{branch_strategy}}           | git convention                            | "main + feature/Story-NNN"       |
    | {{default_branch}}            | git convention                            | "main" / "develop"               |
    | {{constitution_path}}         | repo layout                               | "<repo>/.specify/memory/constitution.md" |
    | {{design_doc_path}}           | repo layout (absolute)                    | "<abs>/fooflow_design.md"        |
    | {{operator_prompt_path}}      | repo layout (absolute)                    | "<abs>/fooflow_operator_prompt.md" |
    | {{spec_prompts_path}}         | repo layout (absolute)                    | "<abs>/fooflow_spec_prompts.md"  |
    | {{last_updated_date}}         | generated at emit time                    | "2026-09-28"                     |
    | {{version}}                   | generated at emit time (start at 1.0.0)   | "1.0.0"                          |

DESIGN CONTRACT
    spec-driven-bootstrap MUST emit a SKILL.md where every {{var}} above has
    been substituted with a non-empty concrete value. Missing placeholder =
    spec-driven-bootstrap returns to design-doc-coach to ask the user.

FRONTMATTER IS INTENTIONAL
    This template has frontmatter at byte 0 (so VSCode Markdown Preview renders
    it as a 2-column table, matching spec-driven-bootstrap/SKILL.md's visual
    style). The {{var}} placeholders in `name:` / `description:` prevent
    Claude Code's SKILL auto-loader from matching any real user phrase against
    this template.

    spec-driven-bootstrap Stage 4 strips the HTML comment block above (between
    the closing `---` of frontmatter and the `# {{project_name}}-spec-driven-dev`
    heading) during emit, so it does NOT appear in the per-project SKILL.md.
-->

# {{project_name}}-spec-driven-dev

## 何时调用

- 用户说"开始 {{project_name}} 的 {{iteration_unit}} {{iteration_id_pattern}}"
- 用户说"按 {{project_name}}_design.md 拆分 AC 出 {{iteration_unit}}"
- 用户说"跑 {{project_name}} 的 AC 验证" / "按 AC 实施"
- 用户粘贴了 {{iteration_unit}} 列表让逐条实施
- 新窗口启动 + 用户提到"{{project_name}} 工程化" / "接 {{project_name}} 工程"

## Pre-flight(每次会话首 turn 必跑 — 防"半路才发现缺工具"白烧 token)

**严格 AND 校验**:工具链 binary **全部必须已装**,任一缺失**强制中止**(不允许"挑一个装就够")。装法见 `_meta-starters/README.md` 的「工具链安装指引」。

```bash
# 自动派生(每个 binary 一行,缺失 echo MISSING_X)
{{tooling_check_commands}}
```

**如何解读输出**:
- 全部 silent → 工具链齐备,继续后续 Step
- 任一 `MISSING_X` → **强制中止**,提示用户装对应工具(指向 README 的「工具链安装指引」),**不让硬跑** —— 半路失败会浪费 token

**Placeholder 来源**:`{{tooling_check_commands}}` 由 `spec-driven-bootstrap` Stage 1 从设计文档派生:
- `build_command` / `test_command` / `verify_command_pattern` 的第一个 word(去重)
- 设计文档 §6 若提到 SpecKit workflow,加 `command -v specify`
- 用户 §2 自己列的额外 CLI binary

---

## 核心原则(5 条不妥协)

```
不是 一次写完所有 PR
是   一次写 {{constitution_path}} → 按 {{iteration_unit}} 循环:
     明确范围 → 实施 → AC 验证 → 合入
```

1. **{{design_doc_path}} 是 single source of truth**:任何代码改动前必读对应章节,禁止凭印象改
2. **{{constitution_path}} 是项目"宪法"**:跨 {{iteration_unit}} 改它必须走 RFC,不能随手改
3. **{{iteration_unit}} 边界 = 验收边界**:每个 {{iteration_unit}} 自带 AC 黑盒验证,verify 全过才合入
4. **{{iteration_split_rules}}**:超过就拆,绝不跨边界大爆炸
5. **新窗口不跨 {{iteration_unit}} 累积 context**:每次开窗从 {{design_doc_path}} 重新读对应章节,避免 staleness

## 关键输入(每次 {{iteration_unit}} 开始前必读)

| 输入 | 路径 | 必读? |
|---|---|---|
| 设计文档 | `{{design_doc_path}}` | ✅ 读对应章节(由 {{operator_prompt_path}} §X.Y 锚定) |
| 项目 SOP | `{{operator_prompt_path}}` | ✅ 完整读 + 同步操作约束 |
| 窗口速查 | `{{spec_prompts_path}}` | ✅ 开窗时引用模板 |
| 宪章 | `{{constitution_path}}` | ✅ 必须已合入 {{default_branch}} |
| 项目 README | `{{readme_path}}` | ✅ 含已执行 {{iteration_unit}} 路线图 / 历史段 — 与 ROADMAP 互补(ROADMAP = 实时 tracker,README = 公开宣告面) |
| ROADMAP tracker | `{{roadmap_path}}` | ✅ 实时 P0/P2 优先级 + 实施顺序 |
| 前序 {{iteration_unit}} 合入记录 | `git log {{default_branch}} --oneline -20` | ✅ 理解已沉淀接口 |
| 技术栈基线 | {{tech_stack_summary}} | ✅ 引用锁定的依赖版本 |
| 验证手段 | `{{verify_command_pattern}}` | ✅ AC 通过的判定方式 |

## 工作流(5 步走,Step -1 按需)

### Step -1:环境探测

```bash
cd <{{project_slug}} 主仓根目录>
test -f <build config file> || echo "MISSING_BUILD_CONFIG"
test -f "{{readme_path}}" || echo "WARN_MISSING_README(项目根 README 不存在 — 外部协作者/读者无宣告面,建议创建)"
```

任一输出非空 → 跑 Step 0 环境初始化;否则跳过。`WARN_MISSING_README` 仅警告,不阻塞(允许新项目暂未生成 README)。

### Step 1:{{iteration_unit}} 边界确认

读 {{operator_prompt_path}} 的 {{iteration_unit}} 拆分映射表,确认本 {{iteration_unit}}:
- id 与 slug
- 来源章节(`{{design_doc_path}} §X.Y`)
- 对应 AC(AC-NN)
- 前置 {{iteration_unit}}(必须已合入)
- 涉及 ErrorCode(若有)

**判断标准**:`{{iteration_split_rules}}` → 超过就拆。

### Step 2:三件套生成(spec/plan/tasks)

按 {{operator_prompt_path}} 的模板,落 `{{iteration_unit_id}}<slug>/` 目录,出 spec + plan + tasks 三件套。

### Step 3:实施 + 验证

按 tasks 逐项实现:
- 每个 T-NN 一个 commit(commit message 走 `{{commit_message_format}}`)
- **必含 validate task**(跑 AC-NN 黑盒)
- AC-NN 全过才能提 PR

验证命令模板:`{{verify_command_pattern}}`

### Step 4:PR + 合入

- PR 标题:`{{commit_message_format}}` 风格
- PR body:贴 spec + plan + tasks + AC 验证输出
- 合入后同步:
  - [ ] {{operator_prompt_path}} 的拆分映射表新增本 {{iteration_unit}}
  - [ ] {{spec_prompts_path}} 的窗口速查补本 {{iteration_unit}} 模板(若有)
  - [ ] {{constitution_path}} §10 风险登记更新(若有 R-XX 缓解)

### Step 4.5:更新 ROADMAP(Story 合入后必跑 — 工程 tracker 与 SOP 双轨)

合入完成后,**立即**更新 `{{roadmap_path}}`:

1. **行迁移**:🟡 §X 待补 段本 {{iteration_unit}} 行 → ✅ 已完成 段
   - 新增列:PR 号 / 合并日期 / 验证输出(tests pass 数 / R-XX 缓解数 / ErrorCode 数)
2. **统计更新**:`📊 统计` 段:N 已合 +1,M 待补 -1,文件数 / ErrorCode / R-XX 缓解率累加
3. **实施顺序建议**:本 {{iteration_unit}} 在依赖图里**划掉**(~~strikethrough~~)
4. **🎯 实施节奏 `next =`** 更新为下一个候选 {{iteration_unit}}(若有强依赖)
5. **版本 bump**:ROADMAP 文件尾 `Last updated` + `Version` +1

**反向同步**(若有 R-XX 缓解 / 新增 ErrorCode):
- [ ] `{{constitution_path}}` §10 风险登记更新 R-XX 缓解状态
- [ ] `{{design_doc_path}}` §13 changelog 加本 {{iteration_unit}} 行(若有 §X 文档变更)
- [ ] `{{readme_path}}`「Story 路线图」段追加本 {{iteration_unit}}(若 README 存在该段)

**禁止**:
- ❌ 直接改 `{{design_doc_path}}` 而不重跑 spec-driven-bootstrap
- ❌ ROADMAP 状态停留在 🟡 而实际 PR 已合(漂移)
- ❌ 跳过 §10 R-XX 缓解记录

---

## {{iteration_unit}} 拆分模板

> 拆分粒度 = 一次 PR + 一次 AC 验证 + 一次合入的可独立交付单元
>
> 具体规则:`{{iteration_split_rules}}`

(具体 {{iteration_unit}} 列表从 `{{operator_prompt_path}}` 的「拆分映射表」读取,本 SKILL 不重复展开,避免与 SOP 双 source of truth。)

## {{constitution_path}} 蒸馏配方(首次调用时)

首次跑 {{project_slug}}-spec-driven-dev(在实施任何 {{iteration_unit}} 之前),先蒸馏 `{{constitution_path}}`。

`{{constitution_path}}` 必须包含以下章节(从 `{{design_doc_path}}` 蒸馏,**不引入新决策**,每条带"来源:{{design_doc_path}} §X.Y"注释):

| 宪章节 | 来源 |
|---|---|
| §1 项目原则 | {{design_doc_path}} §1 锁定决策 |
| §2 技术栈 | {{tech_stack_summary}}(锁定依赖版本) |
| §3 NFR 基线 | {{design_doc_path}} 性能/可靠性基线章节 |
| §4 错误码约定 | {{design_doc_path}} 错误码章节(命名约定/数量上限) |
| §5 测试策略 | {{design_doc_path}} 测试金字塔章节 |
| §6 兼容性矩阵 | {{design_doc_path}} 兼容/SemVer 章节 |
| §7 LTS 政策 | {{design_doc_path}} 支持窗口章节 |
| §8 Glossary | {{design_doc_path}} 术语表(精简 ≤ 20 项) |
| §9 Review 节奏 | {{design_doc_path}} review/CI 章节 |
| §10 风险登记 | {{design_doc_path}} 风险章节,只抄达到阈值的高风险项 |

## 反模式(看到立刻停)

- ❌ 一次出多个 {{iteration_unit}} 的 spec+plan+tasks → 上下文爆炸
- ❌ 跳过 {{constitution_path}} 直接实施 → 没有"宪法"做约束
- ❌ {{iteration_unit}} 间不读前序 PR → 接口冲突
- ❌ AC-NN 黑盒验证省略 → 验收降级
- ❌ 跨 {{iteration_unit}} 改 {{constitution_path}}(不走 RFC)→ 破坏兼容性承诺
- ❌ 把 `{{design_doc_path}}` 整篇贴 prompt → context 爆炸,LLM 第 N 千行漏看
- ❌ 跳过 docs / changelog 同步 → 下一位工程师(包括 Claude Code)抓瞎
- ❌ 用 git 直推 {{default_branch}}(不走 feature branch)→ 无法回滚
- ❌ 合入 Story 后 `{{roadmap_path}}` 不更新 → 实施状态与 tracker 漂移,新窗口无法接力
- ❌ 改 `{{roadmap_path}}` 状态而不真跑验证(口头说"合了"但 tracker 改了 AC 没跑)
- ❌ 合入 Story 后 `{{readme_path}}` 「Story 路线图」段不追加 → 公开宣告面与实际进度漂移,外部读者看到过时信息

## 输出检查清单(每个 {{iteration_unit}} 合入前自检)

- [ ] spec/plan/tasks 三件套完整
- [ ] AC-NN 全过(贴验证输出到 PR body)
- [ ] README / docs / changelog 同步
- [ ] {{constitution_path}} §10 风险登记更新(若有新风险/缓解)
- [ ] PR 标题 + body 符合 `{{commit_message_format}}`
- [ ] {{operator_prompt_path}} 拆分映射表新增本 {{iteration_unit}} 条目
- [ ] 旧 {{iteration_unit}} 文件归档(若 spec 改了需重跑)
- [ ] `{{roadmap_path}}` 已更新(🟡→✅ 行迁移 + 统计 + 实施顺序划掉 + ROADMAP 版本 bump)
- [ ] `{{readme_path}}` 「Story 路线图」段已追加本 {{iteration_unit}}(若 README 存在该段)

## 相关文档(单 source of truth 链)

```
{{design_doc_path}}    ← 项目真理(SKILL 不重复展开,引用即可)
   │
   ├─ 蒸馏(Stage 2)──> {{operator_prompt_path}}   ← 实施 SOP(基本不动)
   ├─ 蒸馏(Stage 3)──> {{spec_prompts_path}}     ← 窗口速查模板(基本不动)
   ├─ emit(Stage 4)────> 本 SKILL(本文件)         ← 项目专属,Claude Code 自动触发
   ├─ emit(Stage 5)────> {{roadmap_path}}         ← 工程 tracker(**Story 合入后可变**)
   └─ 必读参考 ─────> {{readme_path}}            ← 项目根 README(Story 路线图段 / 历史段 / 与 ROADMAP 互补)
```

修改优先级:**永远先改设计文档**,再蒸馏 SOP/prompts/SKILL/ROADMAP,不直接改 SKILL。

**ROADMAP 特殊性**:4 件套中**唯一可变**的 —— Story 合入后由 Step 4.5 更新(🟡→✅ + 统计 + 实施顺序),不需要回 design doc 重蒸馏。

**README 特殊性**:`{{readme_path}}` 是项目**对外宣告面**,含 Story 路线图段(历史已合 / 待补 / 节奏建议),与 ROADMAP 互补 —— ROADMAP 偏实时工程 tracker,README 偏公开宣告 + 给外部读者 / 协作者看的「已做 / 待做」快照。两者**必须同步**(Step 4.5 加 README 「Story 路线图」段同步项)。

## 元 SKILL 关系(本 SKILL 是被生成的,不是手写的)

- 本 `{{project_slug}}-spec-driven-dev/SKILL.md` 由 `spec-driven-bootstrap` SKILL 从**本模板**(`~/.claude/skills/new-project-coach/PROJECT_SKILL_TEMPLATE.md`)emit 出来
- 上游:`design-doc-coach` SKILL 产出 `{{design_doc_path}}` + 5 维度参数
- 下游:每次开窗自动加载本 SKILL,不再走上游
- **四层链路**:**设计文档(SST)→ SOP(实施指南)+ prompts(速查)+ ROADMAP(tracker)→ 本 SKILL(自动触发)**
- **ROADMAP 例外**:四层链路中 ROADMAP 是**唯一可变**的 — Story 合入后由本 SKILL 的 Step 4.5 更新(🟡→✅ + 统计 + 实施顺序),不需要回 design doc 重蒸馏

---

**Last updated**: {{last_updated_date}}
**Version**: {{version}}
(由 spec-driven-bootstrap emit;若设计文档改了,**重跑 distillation** 生成新版本,**不要直接编辑本文件**)

---

## 更新日志(本模板自身的版本)

- **v1.4 → v1.5**: +frontmatter `name:` / `description:` 用 YAML 单引号包裹(让 `{{var}}` 占位符在 VSCode Markdown Preview 中合法 parse — `{{` 不是 YAML scalar 起始,裸字段会让 frontmatter 渲染为纯文本而非 2 列表格)— 修「VSCode 报 `Failed to parse frontmatter — Unexpected scalar at node end`」latent bug
- **v1.3 → v1.4**: +frontmatter 移到 byte 0(让 VSCode Markdown Preview 渲染为 2 列表格)+ HTML comment 下移到 frontmatter 之后(改写警告为 "FRONTMATTER IS INTENTIONAL")+ spec-driven-bootstrap Stage 4 emit bash + PLACEHOLDER TABLE 补 `{{iteration_unit_id}}` 到 22 个 — 修「模板 frontmatter 渲染为纯文本」+「emit 时 HTML comment 污染 per-project SKILL.md」+「body 引用了 TABLE 未列的占位符」三条 latent bug
- **v1.2 → v1.3**: +`{{readme_path}}` 占位符 + 关键输入加 README + Pre-flight 加 README 存在性 check + 反模式 +输出检查清单 +「README 特殊性」说明 — 补"绿地项目 README 含已执行 Story 路线图 / 历史段,是必读参考(与 ROADMAP 互补)"gap
- v1.1 → v1.2: +Pre-flight 严格 AND 校验 + `{{tooling_check_commands}}` 占位符 + Step 4.5 ROADMAP 更新
- v1.0 → v1.1: 初版