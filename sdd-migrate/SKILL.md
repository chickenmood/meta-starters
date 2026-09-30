---
name: sdd-migrate
description: '把项目从 SpecKit-era(绿地零到一)平稳迁移到 OpenSpec-era(棕地稳态增量开发)。迁移 SpecKit 4 件套 + per-Story 三件套 + ROADMAP + constitution 到 OpenSpec 的 openspec/specs/<capability>/spec.md + openspec/changes/<change-id>/ + openspec/AGENTS.md 结构,emit 新的项目专属 OpenSpec SKILL,归档旧 SpecKit 制品。当用户说"项目已发布 v1.0,准备切到 OpenSpec"、"迁移到 OpenSpec"、"进入增量开发阶段"、"SpecKit 收官"、"从绿地转棕地"时调用。本 SKILL 是单向迁移 — 运行后项目进入 OpenSpec-era,不再回 SpecKit;前提:项目版本 ≥ 1.0 + ≥ 5 个 ✅ 合入 Story + SpecKit 4 件套完整 + constitution 合入 main。'
allowedPrompts: [{tool: "Write", prompt: "迁移项目到 OpenSpec 结构(emit openspec/ 目录树 + emit 新项目 SKILL + 归档 SpecKit-era 制品 + 输出 migration-report.md)"}, {tool: "Bash", prompt: "在项目仓跑 git log / find / cat 等只读命令验证前置条件 + 计算 capability 命名 + 移动归档目录"}]
---

# sdd-migrate

## 何时调用

**触发关键词**(任一):
- 用户说"项目已发布 v1.0,准备切到 OpenSpec" / "进入增量开发阶段" / "SpecKit 收官" / "从绿地转棕地"
- 用户说"迁移到 OpenSpec" / "改用 OpenSpec" / "上 OpenSpec"
- 用户粘贴 lingshu(或其他 SpecKit-era 项目)的 `ROADMAP.md`,问"接下来怎么演进"

**何时 NOT 调用**:
- 项目仍在 0.x 版本 + 还没 ≥ 5 个 ✅ Story → 太早,**继续用 `spec-driven-bootstrap` + `<project>-spec-driven-dev/SKILL.md`**(绿地工具更顺手)
- 项目已是 OpenSpec-era(有 `openspec/specs/` `openspec/changes/` 目录)→ 跑新项目专属 `<project>-openspec-dev/SKILL.md`
- 用户只想给项目**新增 1 个 capability**,不是整个生命周期迁移 → 跑项目专属 OpenSpec SKILL 的 `/opsx:propose`
- 用户的提问明显是另一个项目的元操作(比如装环境 / 写 lingshu 仓管理)→ 不要误触发

## 前置条件(Stage 0 强制校验,任一缺失中止)

| # | 前置 | 校验方式 | 失败时提示 |
|---|---|---|---|
| 1 | 项目版本 ≥ 1.0 | `cat <design>.md` frontmatter `version: ≥1.0` | "版本 < 1.0,继续用 SpecKit 跑 Story,不要切" |
| 2 | ≥ 5 个 ✅ 已合 Story | 数 `ROADMAP.md` 的 `✅ 已完成` 段条目 | "Story 数 < 5,继续积累 Story 再迁" |
| 3 | SpecKit 4 件套完整 | 检查 `<project>_design.md` + `<project>_operator_prompt.md` + `<project>_spec_prompts.md` + `<slug>-spec-driven-dev/SKILL.md` 4 个文件全在 | 列出缺哪个的 |
| 4 | constitution 合入 main | `git log main -- .specify/memory/constitution.md` 至少 1 commit | "constitution 还没合入,先 git push" |
| 5 | `command -v openspec` 已装 | Pre-flight 跑 `command -v openspec` 或 `npm root -g` 看 `@fission-ai/openspec` | "OpenSpec CLI 未装,装法见 README「工具链安装指引」" |
| 6 | 当前工作目录是项目仓根 | `git rev-parse --show-toplevel` 不报错 + 含 `_design.md` | "请 cd 到项目仓根再跑本 SKILL" |

**任一失败 → 强制中止**,不静默 fallback。

## 核心原则(4 条不妥协)

1. **单向迁移**:运行后不回 SpecKit —— OpenSpec 是稳态增量开发的更合适工具(`proposal → apply → archive` 三阶段比 SpecKit 的 `spec/plan/tasks` 更适合"小步快跑 + 不丢历史")
2. **3-way merge**:从 ✅ Story 派生 `openspec/specs/<capability>/spec.md`(Source of Truth),从 🟡 派生 `openspec/changes/<change-id>/`(进行中),从 ⏸ 派生 `openspec/changes/future/`(未来启动时再细化)
3. **归档不删除**:SpecKit-era 制品全部 `mv` 到 `archived/spec-kit-era/`,git 历史可查(不 `rm`,防"想回去"无路)
4. **Capability 命名 = 派生,不是创造**:kebab-case,默认策略"一个 ✅ Story 一个 capability",用户可在迁移后 refactor(Stage 2 输出规则)

## 工作流(8 阶段)

### Stage 0:前置校验

按上面前置条件表逐项 check:

```bash
# 1. 版本
version=$(grep -E '^version:' <project>_design.md | head -1 | awk '{print $2}')
# 解析 "1.5.34" → major ≥ 1
[[ "$(echo $version | cut -d. -f1)" -ge 1 ]] || echo "FAIL_VERSION"

# 2. ✅ Story 数
done_count=$(grep -c '^| ✅\|^- \[x\]' ROADMAP.md || true)
[[ "$done_count" -ge 5 ]] || echo "FAIL_STORY_COUNT"

# 3. 4 件套
for f in <project>_design.md <project>_operator_prompt.md <project>_spec_prompts.md \
         .claude/skills/<slug>-spec-driven-dev/SKILL.md; do
  test -f "$f" || echo "MISSING_$f"
done

# 4. constitution 合入
git log main --oneline -- .specify/memory/constitution.md | head -1 || echo "FAIL_CONSTITUTION"

# 5. openspec CLI
command -v openspec || echo "MISSING_OPENSPEC"

# 6. 项目仓根
cd "$(git rev-parse --show-toplevel 2>/dev/null)" || echo "FAIL_CWD"
```

### Stage 1:解析 SpecKit 制品

读下列制品,输出 in-memory mental model(JSON 风格):

```yaml
project_meta:
  name: <project_name>
  slug: <project_slug>
  version: <X.Y.Z>
  one_liner: <design §0>
  principles: [<design §1 bullets>]
  tech_stack: <design §2 summary>
  module_structure: <design §3 tree>
  core_interfaces: <design §4>
  extension_points: <design §5>
  glossary: <design §12>

readme_excerpts:   # 读 README.md 的 Story 路线图段 + 历史段 + 核心特性段(与 ROADMAP 互补)
  story_roadmap_section: "<extracted Markdown>"  # "Story 路线图" / "已合 capability" 段
  history_changelog: "<extracted bullets>"        # 版本变更日志
  core_features_bullets: "<extracted bullets>"    # 核心特性段(供迁移后 manifest 引用)

stories_done:     # 从 ROADMAP ✅ 段 + operator_prompt 拆分映射表 + README Story 路线图段交叉验证
  - id: "#NNN"
    slug: "<slug>"
    title: "<title>"
    acs: ["AC-NN: ...", ...]
    pr: "#PR-NN"
    merged_at: "<date>"
    capability_name: "<derived kebab-case>"

stories_in_progress:   # 从 ROADMAP 🟡 段
  - id, slug, title, acs, capability_name

stories_p2:            # 从 ROADMAP ⏸ 段
  - id, slug, title, acs, capability_name

constitution:     # 读 .specify/memory/constitution.md §1-§10
  principles: §1
  tech_stack: §2
  nfr_baseline: §3
  error_codes: §4
  test_strategy: §5
  compat_matrix: §6
  lts_policy: §7
  glossary: §8
  review_rhythm: §9
  risk_register: §10

operator_prompt_split_table:   # 从 <project>_operator_prompt.md §3.1
spec_prompts_window_templates: # 从 <project>_spec_prompts.md
```

**为何要读 README**:ROADMAP 是**实时工程 tracker**(✅/🟡/⏸/📋/🎯 段),README 是**公开宣告面**(Story 路线图段 / 历史段 / 核心特性段,给协作者 / 外部读者看的「已做 / 待做」快照)。两者**必须交叉验证**:迁移后 openspec/ 的 capability 列表应同时出现在 README 路线图段和 ROADMAP 历史段,**任意一侧不一致** = Stage 2/3 输出有遗漏 → 回到 Stage 1 重新解析。

### Stage 2:派生 `openspec/specs/<capability>/spec.md`(从 ✅ 已合)

**逐个 ✅ Story → 一个 capability**(默认策略):

```bash
mkdir -p openspec/specs/
for story in $stories_done; do
  cap=$(echo "$story.slug" | tr '_' '-')   # kebab-case
  mkdir -p "openspec/specs/$cap/"
  # → emit spec.md(下面模板)
done
```

**`openspec/specs/<capability>/spec.md` 模板**:

```markdown
<!--
Capability: <capability>
Derived from: <project>-spec-driven-dev Story #NNN "<title>" (merged <date>, PR #PR-NN)
Migration: SpecKit-era Story → OpenSpec Source of Truth

DO NOT EDIT THIS FILE DIRECTLY
For new requirements on this capability, create a change proposal:
  /opsx:proposal <change-name> --capability <capability>
-->

# <capability>

## Purpose

<从 Story spec.md 的 "Intent" / "Background" 段派生,50-1000 字符,描述这个 capability 是干嘛的>

## Requirements

### Requirement: <从 AC-NN 派生>

The system SHALL <<<用 SHALL/MUST 改写 AC-NN>>>。

#### Scenario: <场景名>

- **WHEN** <<<触发条件>>>
- **THEN** <<<预期结果>>>
- **AND** <<<附加断言>>>

### Requirement: <下一个 AC-NN>

The system MUST ...

#### Scenario: <...>

- **GIVEN** <<<前置条件>>>
- **WHEN** <<<触发条件>>>
- **THEN** <<<预期结果>>>
```

**AC-NN → Requirement 改写规则**:

| 输入(AC-NN 原文) | 输出(Requirement 改写) |
|---|---|
| `AC-001: 支持 X 功能` | ❌ 拒绝 — 无具体行为,反查 Story spec.md |
| `AC-001: 上传 CSV 时,系统在 100ms 内完成 schema 校验` | ✅ "The system MUST validate uploaded CSV files against the registered schema within 100ms." |
| `AC-002: API 返回 JSON 含 field1, field2, field3` | ✅ "When the /api/foo endpoint is called, the system SHALL return JSON containing fields field1, field2, and field3." |
| `AC-003: 性能 P95 < 200ms` | ✅ "The system MUST respond to <endpoint> with P95 latency below 200ms under nominal load." |

**改写校验**:
- 必含 `SHALL` 或 `MUST`(RFC 2119)
- ≤ 500 字符
- 必含 ≥ 1 个 `#### Scenario:`(GIVEN/WHEN/THEN/AND)
- 反例:`支持 X` / `性能要好` / `要稳定` → 缺具体行为,**回到 Story spec.md 找原始约束再改**

**Capability 命名例外**:
- 同一 capability 跨多个 Story → 合并到同一 `spec.md`,旧 Story 的 Requirements 用 `<sup>Story #NNN</sup>` 注脚标识(用户后续可 refactor)
- Story 标题含 `<module>.<sub>` → 取 `<sub>` 作为 capability
- Story slug 含下划线 → 转 kebab-case

### Stage 3:派生 `openspec/changes/<change-id>/`(从 🟡 + ⏸)

**🟡 在跑 Story → openspec/changes/<change-id>/(完整 proposal)**:

```bash
for story in $stories_in_progress; do
  change_id="$(echo $story.id | tr -d '#')-$(echo $story.slug | tr '_' '-')"
  mkdir -p "openspec/changes/$change_id/specs/$story.capability_name/"
  # → emit proposal.md, design.md, tasks.md, specs/<cap>/spec.md(下面模板)
done
```

**`openspec/changes/<change-id>/proposal.md` 模板**:

```markdown
# Change: <change-id>

> **Status**: Proposed  
> **Created**: <迁移日期>  
> **Source**: SpecKit-era Story #NNN "<title>" (in-progress)

## Why

<从 Story spec.md 的 "Background" / "Problem" 段派生,1-2 句话>

## What Changes

- <implementation-action-1>
- <implementation-action-2>

<若涉及不向后兼容变更,标记 `- **BREAKING** <action>`>

## Capabilities

### New Capabilities

- `<capability_name>` — <一句话描述>

### Modified Capabilities

- *(若有)* `<existing_capability>` — 见 `specs/<existing_capability>/spec.md`

## Impact

- Affected code: `<module>` (≤5 个文件,具体列表从 Story plan.md 摘)
- Affected APIs: `<endpoint>` (若有)
- Dependencies: `<dependency>` (若有)
```

**`openspec/changes/<change-id>/specs/<capability>/spec.md`(Delta Spec)模板**:

```markdown
# Delta for <capability>

## ADDED Requirements

### Requirement: <新行为>

The system SHALL <<<新行为>>>。

#### Scenario: <...>

- **WHEN** <<<...>>>
- **THEN** <<<...>>>
```

Delta 段 4 选 1(按需):
- `## ADDED Requirements` — 新行为
- `## MODIFIED Requirements` — 改已有(原 spec 在 `openspec/specs/`,diff 用 `### MODIFIED Requirement:` 标识)
- `## REMOVED Requirements` — 删除
- `## RENAMED Requirements` — 改名 + 新名

**`openspec/changes/<change-id>/tasks.md` 模板**:

```markdown
# Tasks for <change-id>

> **Source**: 派生自 SpecKit-era Story #NNN 的 plan.md + tasks.md  
> **Status**: Proposed  

## 1. <task group>

- [ ] T-1.1 <action>(Story plan.md T-NN)
- [ ] T-1.2 <action>(Story plan.md T-NN)

## 2. <task group>

- [ ] T-2.1 <action>

## 验证

- [ ] 跑 `<verify_command_pattern>` 全过
- [ ] AC-NN 全过(贴验证输出到 PR body)
```

**⏸ P2 Story → openspec/changes/future/<change-id>/(仅 proposal)**:

```bash
for story in $stories_p2; do
  change_id="$(echo $story.id | tr -d '#')-$(echo $story.slug | tr '_' '-')"
  mkdir -p "openspec/changes/future/$change_id/"
  # → emit proposal.md(Why + What Changes + Capabilities + Impact,无 tasks.md / specs/)
done
```

**`openspec/changes/future/<change-id>/proposal.md`** 模板与上面相同,但状态标 `Future (deferred from SpecKit-era Story #NNN)`,且不生成 `tasks.md` / `specs/`(等用户 `/opsx:continue <change-id>` 时再补)。

### Stage 4:emit `openspec/AGENTS.md`(从 constitution)

**AGENTS.md 结构**(OpenSpec 期望的 AI 工作流格式):

```markdown
# <project_name> — AI 协作章程

> **来源**:SpecKit-era `<project>_design.md` §1 + `.specify/memory/constitution.md` §1-§10  
> **本目录**:`openspec/AGENTS.md` — OpenSpec 运行时 AI 自动加载

## 项目原则(<= 5 条)

1. <principle-1>
2. <principle-2>
...

## 技术栈基线

<tech_stack_summary>

## 硬约束(从 constitution 蒸馏)

| 维度 | 约束 |
|---|---|
| NFR | <constitution §3 bullets> |
| 错误码 | <constitution §4 命名 + 上限> |
| 兼容矩阵 | <constitution §6 JDK/Python/Node + SemVer> |
| LTS 政策 | <constitution §7 支持窗口> |
| 测试金字塔 | <constitution §5 单测/集成/E2E 比例> |
| Review 节奏 | <constitution §9 PR review SLA> |

## Spec 写作规范

- Requirement 文本必含 `SHALL` 或 `MUST`(RFC 2119)
- 每个 Requirement 必含 ≥ 1 个 `#### Scenario:`
- Scenario 用 `**GIVEN**` / `**WHEN**` / `**THEN**` / `**AND**` 4 段
- Requirement ≤ 500 字符,Purpose 50-1000 字符

## 变更管理流程(OpenSpec)

```
```
1. /opsx:propose <change-id>     起草 proposal + delta specs + tasks
2. /opsx:apply <change-id>       按 tasks.md 实施 + 验证
3. /opsx:verify <change-id>      跑 openspec validate + AC 黑盒
4. /opsx:archive <change-id>     归档到 changes/archive/YYYY-MM-DD-<change-id>/
```

## 风险登记(精简到 ≤ 10 项,从 constitution §10)

| R-ID | 风险 | 分值 | 缓解状态 |
|---|---|---|---|
| R-XX | <risk> | <score> | <open/mitigated> |

## 反模式(看到立刻停)

- ❌ 跳过 `/opsx:propose` 直接改 `openspec/specs/`(绕开审计)
- ❌ 改 `openspec/specs/` 而不写 delta spec(直接改 SoT,无 review 链)
- ❌ `/opsx:apply` 中不跑 verify 就 `/opsx:archive`(降级验收)
- ❌ 把 Proposal 直接 commit 到 main(必须 feature branch + review)

---

### Stage 5:emit `openspec/project.md`(从 design §0/§1/§2/§3/§4/§12)

```markdown
# <project_name>

> **一句话定位**(<design §0>):
> <<<定位文本>>>

## 项目原则

<from §1>

## 技术栈

<from §2 — 语言 / 构建工具 / 依赖 + 版本锁>

## 模块结构

<from §3 — 目录树 + 依赖方向>

## 核心接口 / 数据结构

<from §4>

## 扩展点

<from §5 — Java SPI / Python entry_points / Go interface / Rust trait / Node exports 等>

## Glossary

<from §12 — 术语表精简 ≤ 20 项>
```

### Stage 6:emit `openspec/config.yaml`

```yaml
# OpenSpec config for <project_name>
# Generated by sdd-migrate on <date> from SpecKit-era artifacts

schema: spec-driven
context: |
  <<<design §0 一句话定位,1-2 句>>>
rules:
  requirement:
    shall_must: true
    max_length: 500
    min_scenarios: 1
  scenario:
    format: gherkin
    keywords: [GIVEN, WHEN, THEN, AND]
  change:
    max_deltas: 10
    proposal:
      required: [why, what_changes, capabilities, impact]
references:
  - openspec/AGENTS.md
  - openspec/project.md
```

### Stage 7:emit 项目专属 OpenSpec SKILL

从 `new-project-coach/OPENSPEC_PROJECT_SKILL_TEMPLATE.md` 替换 `{{var}}` 占位符,emit 到 `~/.claude/skills/<slug>-openspec-dev/SKILL.md`。

**emit 变量**(12 个,与模板 PLACEHOLDER TABLE 对齐):

| 占位符 | 来源 |
|---|---|
| `{{project_name}}` | Stage 1 `project_meta.name` |
| `{{project_slug}}` | Stage 1 `project_meta.slug` |
| `{{one_liner}}` | Stage 1 `project_meta.one_liner` |
| `{{openspec_dir}}` | 固定 `openspec/` |
| `{{capabilities}}` | Stage 2 派生出的 capability 列表 |
| `{{active_changes}}` | Stage 3 派生出的 🟡 change 列表 |
| `{{tooling_check_commands}}` | `command -v openspec\ncommand -v git`(多行 bash) |
| `{{verify_command_pattern}}` | Stage 1 `project_meta.verify_command`(从原 SpecKit 项目继承) |
| `{{readme_path}}` | Stage 1 `project_meta.readme_path`(项目根 README 绝对路径) |
| `{{default_branch}}` | Stage 1 `project_meta.default_branch`(`main` / `develop`) |
| `{{last_updated_date}}` | emit 日期(`date +%Y-%m-%d`) |
| `{{version}}` | 起始 `1.0.0` |

**emit bash**(真跑命令 — 含 HTML comment 替 strip):

```bash
TEMPLATE="$HOME/.claude/skills/new-project-coach/OPENSPEC_PROJECT_SKILL_TEMPLATE.md"
DEST="$HOME/.claude/skills/${PROJECT_SLUG}-openspec-dev/SKILL.md"
TMP="${DEST}.tmp.$$"

# 前置:确认变量都已 set(避免 sed 把空值写进 emit)
: "${PROJECT_NAME:?need PROJECT_NAME from Stage 1 project_meta.name}"
: "${PROJECT_SLUG:?need PROJECT_SLUG from Stage 1 project_meta.slug}"
: "${ONE_LINER:?need ONE_LINER from Stage 1 project_meta.one_liner}"
: "${CAPS_TABLE:?need CAPS_TABLE from Stage 2 derived capabilities list}"
: "${ACTIVE_CHANGES_TABLE:?need ACTIVE_CHANGES_TABLE from Stage 3 derived active change list}"
: "${VERIFY_CMD:?need VERIFY_CMD from Stage 1 project_meta.verify_command}"
: "${README_PATH:?need README_PATH from Stage 1 project_meta.readme_path}"
: "${DEFAULT_BRANCH:=main}"
: "${LAST_UPDATED_DATE:=$(date +%Y-%m-%d)}"
: "${VERSION:=1.0.0}"

mkdir -p "$(dirname "$DEST")"

# Pass 1 + Pass 2:用 awk 一次完成 {{var}} 替换 + HTML comment 块 strip
# (awk gsub 原生支持多行替换,无需 sed 的 `\<newline>` 技巧;awk 状态机天然处理 HTML comment 块)
# 注意:`{{tooling_check_commands}}` 是固定多行 bash 字符串(OpenSpec + git 两条),
# 但为安全起见(Stage 1 未来若派生出更多 tool)走 temp file + awk getline 模式
TOOLING_CHECK_FILE="${TMP}.tooling"
cat > "$TOOLING_CHECK_FILE" << 'TOOLING_EOF'
command -v openspec || echo MISSING_OPENSPEC
command -v git || echo MISSING_GIT
TOOLING_EOF

awk -v name="$PROJECT_NAME" \
    -v slug="$PROJECT_SLUG" \
    -v one_liner="$ONE_LINER" \
    -v caps="$CAPS_TABLE" \
    -v active="$ACTIVE_CHANGES_TABLE" \
    -v tooling_file="$TOOLING_CHECK_FILE" \
    -v verify="$VERIFY_CMD" \
    -v readme="$README_PATH" \
    -v default_br="$DEFAULT_BRANCH" \
    -v date="$LAST_UPDATED_DATE" \
    -v ver="$VERSION" \
  '
  function read_tooling(   cmd, line, content) {
    cmd = "cat \"" tooling_file "\""
    while ((cmd | getline line) > 0) {
      content = (content == "" ? line : content "\n" line)
    }
    close(cmd)
    return content
  }
  BEGIN {
    in_c = 0
    tooling = read_tooling()
  }
  {
    if (in_c == 1) {
      if ($0 ~ /^-->$/) in_c = 0
      next
    }
    if ($0 ~ /^<!--$/) { in_c = 1; next }
    line = $0
    gsub(/\{\{project_name\}\}/, name, line)
    gsub(/\{\{project_slug\}\}/, slug, line)
    gsub(/\{\{one_liner\}\}/, one_liner, line)
    gsub(/\{\{openspec_dir\}\}/, "openspec", line)
    gsub(/\{\{capabilities\}\}/, caps, line)
    gsub(/\{\{active_changes\}\}/, active, line)
    gsub(/\{\{tooling_check_commands\}\}/, tooling, line)
    gsub(/\{\{verify_command_pattern\}\}/, verify, line)
    gsub(/\{\{readme_path\}\}/, readme, line)
    gsub(/\{\{default_branch\}\}/, default_br, line)
    gsub(/\{\{last_updated_date\}\}/, date, line)
    gsub(/\{\{version\}\}/, ver, line)
    if ($0 ~ /^#/) in_c = 2
    print line
  }' "$TEMPLATE" > "$TMP"
rm -f "$TOOLING_CHECK_FILE"

# 验证:emit 后文件首行必须是 `---`,且 # heading 必须出现,无残留 `{{var}}`
head -1 "$TMP" | grep -q '^---$' || { echo "FAIL: frontmatter opener not at byte 0"; rm -f "$TMP"; exit 1; }
grep -q "^# ${PROJECT_NAME}-openspec-dev" "$TMP" || { echo "FAIL: heading missing"; rm -f "$TMP"; exit 1; }
grep -q "{{[a-z_]*}}" "$TMP" && { echo "FAIL: unsubstituted {{var}} remaining"; grep -o "{{[a-z_]*}}" "$TMP" | sort -u; rm -f "$TMP"; exit 1; }

# 落地
mv "$TMP" "$DEST"
echo "✓ emitted: $DEST"
```

**关键 strip 行为**:Pass 2 用 awk 把 frontmatter closer `---`(行 4)与首个 `# heading`(约行 56)之间的 HTML comment 块全部删除,只 emit heading 之后的内容。最终 per-project SKILL.md 首行是 `---`、第 5 行起就是 `# <project>-openspec-dev heading`,**无任何 HTML comment 残留**——Claude Code auto-loader 立即识别。

**emit 后**用户 `cp` 到 `<repo>/.claude/skills/<slug>-openspec-dev/SKILL.md`(替换 `<slug>-spec-driven-dev/SKILL.md`,后者归档到 `archived/spec-kit-era/`)。

### Stage 8:归档 SpecKit-era 制品

```bash
mkdir -p archived/spec-kit-era/specs/

# 设计文档
mv <project>_design.md archived/spec-kit-era/

# 4 件套 SOP
mv <project>_operator_prompt.md archived/spec-kit-era/
mv <project>_spec_prompts.md archived/spec-kit-era/

# ROADMAP(顶部加 SpecKit-era 历史摘要)
{
  echo "# SpecKit-era ROADMAP (历史)"
  echo ""
  echo "> **状态**:已迁移到 OpenSpec,本文件归档。后续变更请用 OpenSpec workflow(`/opsx:propose → apply → archive`)。"
  echo "> **迁移日期**:<date>"
  echo "> **派生能力列表**:见 openspec/specs/<cap>/spec.md"
  echo ""
  echo "---"
  echo ""
  cat ROADMAP.md
} > archived/spec-kit-era/ROADMAP.md
mv ROADMAP.md archived/spec-kit-era/ROADMAP.md

# per-Story specs
mv specs/<NNN>-<slug>/ archived/spec-kit-era/specs/  # 对每个 ✅ Story

# constitution
mv .specify/memory/constitution.md archived/spec-kit-era/constitution.md
# 空目录清掉
rmdir .specify/memory/ 2>/dev/null || true
rmdir .specify/ 2>/dev/null || true

# 旧 project SKILL
mv .claude/skills/<slug>-spec-driven-dev/SKILL.md archived/spec-kit-era/

# README(顶部加 SpecKit-era 路线图归档说明)
mv README.md archived/spec-kit-era/README.md
# 在 repo 根创建新的棕地 README skeleton,指向 openspec/
cat > README.md << 'EOF'
# <project_name>

> **已进入 OpenSpec-era**(迁移于 <date>,sdd-migrate v1.0)。  
> 历史 SpecKit-era 制品归档在 `archived/spec-kit-era/`。

## 项目原则 / 一句话定位 / 技术栈

详见 `openspec/project.md`。

## AI 协作章程

详见 `openspec/AGENTS.md`。

## 已合 capability / 已 archive change

详见 `openspec/specs/` 与 `openspec/changes/archive/`。

## 历史(SpecKit-era)

`<id>`Story 列表见 `archived/spec-kit-era/ROADMAP.md` ✅ 已完成段。
EOF
```

**README 处理策略**(必读):
- 旧 README 含 "Story 路线图" / "已合 capability" / 历史段 → **整文件归档**到 `archived/spec-kit-era/README.md`,**不重写**(保留历史宣告)
- 在 repo 根创建**新的棕地 README skeleton**,指向 `openspec/project.md` + `openspec/AGENTS.md` + `openspec/specs/` + `openspec/changes/archive/`
- 用户后续可按需扩展新 README 内容(基于新制品,不再提 SpecKit-era 制品)

### Stage 9:emit `openspec/migration-report.md`

```markdown
# SpecKit → OpenSpec 迁移报告

> **迁移日期**:<YYYY-MM-DD>  
> **执行 SKILL**:sdd-migrate v1.0  
> **项目**:<project_name> v<version>

## 派生 Capability 列表(<N> 项)

| Capability | Source Story | AC 数 |
|---|---|---|
| <cap> | #NNN "<title>" | <AC count> |

## 新生成的 Change Proposal(<M> 项)

| Change ID | Status | Source Story |
|---|---|---|
| <change-id> | Proposed(等 /opsx:apply)| 🟡 #NNN "<title>" |
| future/<change-id> | Future | ⏸ #NNN "<title>" |

## 归档的 SpecKit-era 制品

```
```
archived/spec-kit-era/
├── <project>_design.md
├── <project>_operator_prompt.md
├── <project>_spec_prompts.md
├── ROADMAP.md(顶部加 SpecKit-era 历史摘要)
├── constitution.md
├── <slug>-spec-driven-dev/SKILL.md
├── README.md(整文件归档,保留历史宣告)
└── specs/<NNN>-<slug>/
```

## README 处理摘要

- **旧 README**:整文件 mv 到 `archived/spec-kit-era/README.md`(保留历史宣告,不重写)
- **新 README skeleton**:emit 到 `<repo>/README.md`,指向 `openspec/project.md` + `AGENTS.md` + `specs/` + `changes/archive/`
- 后续用户按需扩展新 README 内容(基于新制品,不再提 SpecKit-era 制品)

## OpenSpec 命令清单(下一步)

```bash
# 验证生成结果
openspec validate --strict

# 列出待 apply 的 changes
openspec list --changes

# 跑变更(选 1 个当前 Story 启动)
opensx:apply <change-id>

# 跑完后归档
openspec archive <change-id>
```

## 后续建议

- [ ] 跑 `openspec validate --strict` 看是否有 cap/spec 格式问题
- [ ] 把 `~/.claude/skills/<slug>-openspec-dev/SKILL.md` cp 到 `<repo>/.claude/skills/`
- [ ] 新开窗口试触发词"propose change X for <project>"验证 SKILL 自动加载
- [ ] 跑 1 个 🟡 change 的 `/opsx:apply` 验证 workflow 通畅
- [ ] 把 `archived/spec-kit-era/` 加进 `.gitignore`(可选,保留也行)
- [ ] 验证新 README skeleton:打开 README 看是否正确指向 `openspec/` 制品

## 反模式(看到立刻停)

| ❌ 反模式 | 为什么不许 |
|---|---|
| 版本 < 1.0 就跑迁移 | 绿地项目继续用 SpecKit 跑 Story,OpenSpec 是稳态增量工具,不等 |
| 直接改 `openspec/specs/<cap>/draft.md` 而不写 delta spec | 绕开审计链,失去"为什么这么改"的可解释性 |
| 跳过 Stage 0 前置校验 | 缺制品时跑迁移会污染生成结果,留下半截制品 |
| 用 `rm` 而非 `mv` 归档 SpecKit 制品 | 一旦想回去无路,git 历史虽在但制品位置找不到 |
| 一 capability 对多 Story 时不合并 | 后续 refactor 成本高,**合并到同一 spec.md,Requirements 用注脚标识 Story 来源** |
| ⏸ P2 Story 仍生成完整 tasks.md + specs/ | 浪费精力(用户不启动),**只 proposal.md 即可** |
| 改 `openspec/specs/` 而不跑 `/opsx:propose` | 等价于 SpecKit-era 跳宪章,OpenSpec 不接受这种变更 |
| 迁移后不删旧 `<slug>-spec-driven-dev/SKILL.md` | 双 SKILL 会被 auto-loader 都加载,触发词冲突 |
| Stage 1 不读 README | README 含 Story 路线图 / 历史段,**与 ROADMAP 互补**(ROADMAP 实时 tracker + README 公开宣告面),**只读 ROADMAP 会漏外部读者关心的"已做"宣告** |
| 旧 README 不归档 + 不 emit 新 README skeleton | 迁移后仓库根 README 仍说"SpecKit-era 项目",**外部读者看到过期信息** |

## 输出检查清单(自检)

- [ ] Stage 0 全过(版本 / Story 数 / 4 件套 / constitution / openspec CLI / cwd)
- [ ] `openspec/specs/<cap>/spec.md` 每个 ✅ Story 都有对应 capability + Requirement 含 SHALL/MUST + Scenario 含 GIVEN/WHEN/THEN
- [ ] `openspec/changes/<change-id>/proposal.md` 每个 🟡 Story 都有对应 change + 4 段全
- [ ] `openspec/changes/future/<change-id>/proposal.md` ⏸ Story 都有,但**只有 proposal.md**(无 tasks/specs)
- [ ] `openspec/AGENTS.md` 硬约束表 + Spec 写作规范 + 变更管理流程全在
- [ ] `openspec/project.md` §0/§1/§2/§3/§4/§5/Glossary 全在
- [ ] `openspec/config.yaml` schema/规则/引用都对
- [ ] `~/.claude/skills/<slug>-openspec-dev/SKILL.md` emit 成功(替换 12 个 {{var}},含 `{{readme_path}}`)
- [ ] `archived/spec-kit-era/` 8 类制品全在(design + operator + spec_prompts + ROADMAP + constitution + 旧 SKILL + README + specs/)
- [ ] 旧 README 已归档到 `archived/spec-kit-era/README.md`(保留历史宣告)
- [ ] 新 `<repo>/README.md` 已 emit(skeleton 指向 `openspec/` 制品)
- [ ] `openspec/migration-report.md` 输出(含能力列表 + change 列表 + README 处理摘要 + 命令清单)
- [ ] 跑 `openspec validate --strict` 无 ERROR(可 WARNING)
- [ ] 镜像两侧 md5 MATCH

## 相关文档(单 source of truth 链)

```
<project>_design.md(已归档)
   │
   ├─ 蒸馏(Stage 2)──> openspec/specs/<cap>/spec.md   ← Source of Truth(✅ 已合)
   ├─ 蒸馏(Stage 3)──> openspec/changes/<change-id>/   ← Proposed(🟡 进行中)
   ├─ 蒸馏(Stage 4)──> openspec/AGENTS.md              ← AI 工作流章程
   ├─ 蒸馏(Stage 5)──> openspec/project.md             ← 项目上下文
   ├─ emit(Stage 6)────> openspec/config.yaml           ← OpenSpec 配置
   ├─ emit(Stage 7)────> <slug>-openspec-dev/SKILL.md   ← 项目专属 SKILL(替换 spec-driven-dev)
   └─ emit(Stage 9)────> openspec/migration-report.md  ← 迁移报告
```

**修改优先级**:**改 openspec/specs/ 必走 change proposal**(Stage 2 之后)—— 这是 OpenSpec 的核心约束。

**Stage 7 emit 的项目 SKILL** 见 `~/.claude/skills/new-project-coach/OPENSPEC_PROJECT_SKILL_TEMPLATE.md`,其模板里已带 OpenSpec 触发词 + Pre-flight + 3 阶段工作流。

## 元 SKILL 关系(本 SKILL 是第三层)

```
设计文档层(SST):<project>_design.md → 已归档到 archived/spec-kit-era/
       │
       ├─ design-doc-coach(元 SKILL #1,生成设计文档)
       │
       ▼ 审稿 + status: ready-for-bootstrap
SpecKit 制品层(已归档):
       <project>_operator_prompt.md
       <project>_spec_prompts.md
       ROADMAP.md
       specs/<NNN>-<slug>/{spec,plan,tasks}.md
       .specify/memory/constitution.md
       │
       ├─ spec-driven-bootstrap(元 SKILL #2,蒸馏 SpecKit 4 件套)
       │
       ▼ 项目 ≥ v1.0 + ≥ 5 ✅ Story
       │
       sdd-migrate(本 SKILL #3,SpecKit → OpenSpec 迁移)
       │
       ▼
OpenSpec 制品层:
       openspec/specs/<cap>/spec.md(Source of Truth)
       openspec/changes/<change-id>/(Proposed)
       openspec/AGENTS.md(AI 工作流)
       openspec/project.md(项目上下文)
       openspec/config.yaml(配置)
       openspec/migration-report.md(迁移报告)
       │
       ├─ <slug>-openspec-dev/SKILL.md(项目专属 SKILL,OpenSpec-era)
       │
       ▼ 后续增量开发走 /opsx:propose → /opsx:apply → /opsx:archive
```

---

**Last updated**: 2026-09-29
**Version**: 1.3(v1.2 → v1.3: +**修 fence 配对 / 删 orphan + 加显式 opener** — (a) 删除 line 681 orphan ` ``` `(造成「`## 反模式`/`## 元 SKILL 关系`/Stage 5/6/7/9 标题 + 树图 + 版本页脚全部被吃进 code block」,版本页脚 `**Last updated**:` 显示字面 `**` 而非加粗,`---` 不渲染 HR);(b) 删除 line 370 stray ` ``` ` 改成 `---` HR(同上原因);(c) numbered list ` ``` ` 加显式 opener(line 352,原 ` ``` ` 既闭 315 markdown 又要开新 block,Python markdown lib 不支持同 line 双动作);(d) SpecKit-era 归档树加显式 opener(line 642,原 ` ``` ` 在 619 markdown close 之后被强制 OPENER,导致 `## README 处理摘要`+`## OpenSpec 命令清单`+Stage 6 bash 整段进 code block)— 修「Stage 6/7/8/9 标题 + `## 元 SKILL 关系` + 版本元数据 全部渲染为 code 文字 / `---` 不渲染 HR」latent bug;前轮 v1.1 → v1.2 Stage 7 emit bash + 占位符表 + 检查清单保留不变)

(由用户调用"迁移到 OpenSpec"等触发词自动加载;前置条件见 Stage 0)