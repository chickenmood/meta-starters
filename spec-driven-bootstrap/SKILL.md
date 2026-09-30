---
name: spec-driven-bootstrap
description: '把已有 <project>_design.md 蒸馏成 3 件套 SOP: <project>_operator_prompt.md(操作 SOP,含 Story/Iteration 拆分 / AC 黑盒规范 / 反模式)、<project>_spec_prompts.md(窗口 prompt 速查四类模板)、~/.claude/skills/<slug>-spec-driven-dev/SKILL.md(项目专属 SKILL,从 PROJECT_SKILL_TEMPLATE emit)。当用户说"按设计文档蒸馏"、"跑 SOP"、"生成项目专属 SKILL"、"项目开工"、"蒸馏出可执行流程"时调用。本 SKILL 只蒸馏,不写实现代码,不做新架构决策;前提 _design.md 必须存在且 §1 §2 §6 三块齐。'
allowedPrompts: [{tool: "Write", prompt: "生成 SOP 与项目 SKILL 文件(operator_prompt + spec_prompts + emit 项目专属 SKILL)"}]
---

# spec-driven-bootstrap

## 何时调用

**触发关键词**(任一):
- 用户说"按设计文档蒸馏 / 跑 SOP / 生成 3 件套 / 项目开工"
- 用户说"我要落地这个项目了 / 帮我出可执行流程 / 把设计文档转成 Story 循环"
- 用户的 CLAUDE.md 或 STARTER 接力调用

**前置条件**(必须全满足):
- `<project_name>_design.md` 已存在
- 文件含 §1 项目原则 + §2 技术栈 + §6 AC / Story 拆分 初版
- 文件顶部 frontmatter `status: ready-for-bootstrap`(或同等明示"已审完")
- 用户已审完设计文档,所有 `<!-- TODO -->` 标记已清理

**前置条件不满足 → 提示用户先跑 design-doc-coach**:

> "检测到 `<project_name>_design.md` 不存在或仍在草稿阶段(`status: awaiting-review`)。请先跑 `design-doc-coach` 完成采访 + 草稿输出 + 审稿 TODO 清理。"

## 核心原则(3 条不妥协)

1. **只蒸馏,不创造新决策**:所有锁定项从 `_design.md` 抄,不允许此处改。若发现 design doc 有 gap,**标 TODO 回去让人补**,不要替人填
2. **3 件套之间单向引用,不复制内容**:`_operator_prompt.md` 引用 design doc 章节号,不抄全文;`_spec_prompts.md` 引用 operator prompt 章节号;`<project>-spec-driven-dev/SKILL.md` 引用前两者路径。避免多 source of truth 漂移
3. **项目专属 SKILL 必须从模板 emit**:不手写,用 `~/.claude/skills/new-project-coach/PROJECT_SKILL_TEMPLATE.md` 替换 `{{var}}` 占位符 emit。模板缺失 → 立即报错,**不要 fallback 到手写**

## 关键输入

| 输入 | 来源 | 必读? |
|---|---|---|
| `<project_name>_design.md` | 整个项目周期 | ✅ 全文(只读) |
| `PROJECT_SKILL_TEMPLATE.md` | `~/.claude/skills/new-project-coach/` | ✅ 模板全文 + 占位符表 |
| 设计文档顶部 frontmatter | `_design.md` 头部 | ✅ 取项目名 / version / 5 维度参数 |
| 用户指定输出目录 | 来自 STARTER 或用户当前消息 | ✅ 决定 2 件 SOP 落到哪里(operator/prompts 与 SKILL 路径分开)|
| 5 维度参数(若有 interview log) | design-doc-coach 阶段输出 | ✅ 兜底(若 design frontmatter 缺字段)|

## 工作流(4 阶段)

### Stage 0:状态确认(轻量校验 + fuzzy 标题兼容,不重活)

```bash
test -f <project>_design.md || { echo "BLOCKED: run design-doc-coach first"; return 1; }

# Fuzzy check §1 项目原则 — 严格 §1 / 模糊 ## 原则 / 关键词 锁定决策,任一命中即可
if ! grep -qE "^## §1" <project>_design.md \
   && ! grep -qE "^## 原则" <project>_design.md \
   && ! grep -qE "锁定决策|原则与目标" <project>_design.md; then
  echo "WARN: §1 项目原则 missing(严格字面 §1 / 模糊 ## 原则 / 关键词 '锁定决策|原则与目标' 均未命中)"
  echo "      → design-doc-coach 输出的文档天然有 §1;手写文档加 '## §1 项目原则' 节"
fi

# Fuzzy check §2 技术栈 — 严格 §2 / 模糊 ## 技术栈 / 关键词
if ! grep -qE "^## §2" <project>_design.md \
   && ! grep -qE "^## 技术栈" <project>_design.md \
   && ! grep -qE "语言版本|构建工具|核心框架" <project>_design.md; then
  echo "WARN: §2 技术栈 missing"
  echo "      → 加 '## §2 技术栈'(语言 / 构建工具 / 依赖版本锁)"
fi

# Fuzzy check §6 AC / Story 拆分 — 严格 §6 / 模糊 ## AC 总账 / 关键词
if ! grep -qE "^## §6" <project>_design.md \
   && ! grep -qE "^## AC ?总账" <project>_design.md \
   && ! grep -qE "AC-NN|Story.*拆分|拆分映射|验收标准" <project>_design.md; then
  echo "WARN: §6 AC / Story 拆分 missing"
  echo "      → 这是蒸馏的强制前置:至少 1 条 AC-NN + 至少 1 个 Story/Iteration 拆分"
fi

# Status 强校验(无论 fuzzy 如何,都要 ready 声明)
grep -q "ready-for-bootstrap" <project>_design.md || {
  echo "WARN: frontmatter status 不在 ready-for-bootstrap"
  echo "      → 审稿完后切 status: awaiting-review → ready-for-bootstrap"
}
```

任一 WARN → 提示用户补完再回来。fuzzy 命中表示"用户手写的章节标题不规范",依然可继续(不强阻);完全 miss 才强提示。

### Stage 1:解析设计文档 → 元数据 JSON(in-memory,不落盘)

读 `_design.md`,提取以下键值(供 Stage 2/3/4 使用):

| 字段 | 来源章节 | 用途 |
|---|---|---|
| `project_name` | frontmatter / §0 | 所有文件标题 / 触发词 |
| `project_slug` | frontmatter / 派生(`<project_name>` → kebab-case)| 文件名 / 路径 |
| `version` | frontmatter | SOP / prompts / SKILL 版本号同步 |
| `last_updated` | frontmatter / 今天 | 各件尾 "Last updated" |
| `iteration_unit` | §4 / §3 | prompts 中的 "Story #NNN" 替换 |
| `iteration_id_pattern` | §6 | triggers / prompt 模板 |
| `iteration_split_rules` | §6 | prompts + project SKILL 反模式 |
| `tech_stack_summary` | §2 | project SKILL frontmatter + prompts |
| `build_command` | §2 | project SKILL `{{build_command}}` |
| `test_command` | §2 / §7 | project SKILL `{{test_command}}` |
| `verify_command_pattern` | §7 | project SKILL `{{verify_command_pattern}}` |
| `commit_message_format` | git workflow(默认 fallback)| project SKILL `{{commit_message_format}}` |
| `branch_strategy` | git workflow(默认 fallback)| project SKILL `{{branch_strategy}}` |
| `default_branch` | git workflow(默认 `main`)| project SKILL `{{default_branch}}` |
| `constitution_path` | §3 模块结构 / 默认 `.specify/memory/constitution.md` | project SKILL `{{constitution_path}}` |
| `design_doc_path` | frontmatter / 用户指定 | project SKILL `{{design_doc_path}}`(绝对路径)|
| `operator_prompt_path` | 本 Stage 2 输出 | project SKILL `{{operator_prompt_path}}` |
| `spec_prompts_path` | 本 Stage 3 输出 | project SKILL `{{spec_prompts_path}}` |
| `risk_register_threshold` | §11 风险登记(若 §11 缺失则默认 ≥6)| operator_prompt 风险章节 |
| `error_code_convention` | §8 错误码(若 §8 缺失则默认 `PROJ-<域><NN>`)| operator_prompt 错误章节 |
| `tooling_check_commands` | §2 派生(build_command / test_command / verify 第一 word 去重;§6 若提 SpecKit + `command -v specify`)| project SKILL `{{tooling_check_commands}}` — Pre-flight 多行 `command -v X \|\| echo MISSING_X` |
| `roadmap_path` | 用户指定 / 默认 `<project_slug>/specs/ROADMAP.md`(SpecKit 风格)| Stage 5 emit + project SKILL Step 4.5 更新 ROADMAP 引用 |

⚠️ 若 `iteration_unit` / `iteration_id_pattern` / `commit_message_format` 任何一个缺失 → 提示用户回 design-doc-coach 补,**不要替人默认**(上面只列 fallback 的字段例外)。

### Stage 2:输出 `_operator_prompt.md`

模板参考(参考 lingshu 那份结构,详见 lingshu 那条 `speckit_operator_prompt.md` 的章节布局):

```
§0   项目身份(name / version / source 设计文档路径 / status)
§1   核心结论(30s 读完 — 这不是一次写完,是按 Story 循环)
§2   双层架构(宪章 vs Story spec 的职责边界)
§3   Story/Iteration 拆分映射表
       table 列: # | slug | source chapter | AC | 主要交付物 | 风险标记
§3.1  Story 总览
§3.2  spec.md 模板
§3.3  plan.md 模板
§3.4  tasks.md 模板
§4   在新窗口的落地姿势
       §4.0  Step -1:环境探测(若适用)
       §4.1  Step 0:宪章蒸馏
       §4.2  Step N(N≥1):第 1 个 Story 完整循环
       §4.3  关键约束
§5   反模式
§6   与 Claude Code SKILL 体系的对应(引用 PROJECT_SKILL_TEMPLATE / 项目专属 SKILL)
§7   附录:每个 Story 的可验证输出(AC 列表 + 验证命令)
§8   附录 A:Project Skeleton Init(若有仓库冷启动场景)
```

**关键约束**:
- Story 拆分表的 id 模式必须沿用 `_design.md` §6 的 `iteration_id_pattern`
- AC 黑盒测试章节必须沿用 `_design.md` §6 + §7 的 NFR 数字
- 反模式表必须包含"普适 5 条"(一次写多个 / 跳过宪章 / 不读前序 PR / 省略 AC / 不走 RFC 改宪章)+ 任何项目特定的(例如 banned-dependencies)
- 行数预算 200-500 行(精简 6 节版可不写 §6 / §7)

### Stage 3:输出 `_spec_prompts.md`

模板参考(参考 lingshu 那份):

```
A. 冷启动(第 1 次开窗)
   A1. 完全从零(没有项目仓)
   A2. 仓已存在但没初始化
   A3. 已有初始化过的仓,新窗口接手
B. 续接(中间接手新窗口)
   B1. 标准续接(给下个 Claude Code session)
   B2. 接力但要改动 Story 计划
   B3. 完全 cold resume
C. 日常迭代推进
   C1. 标准推进
   C2. 带前置 PR 上下文
   C3. 显式只跑某一阶段(不跑全流程)
   C4. 跳过验证(应急)
   C5. 加快节奏(批量推进)
D. 异常处理
   D1. Story 跑挂了(验证失败)
   D2. 修改宪章(非常规)
   D3. 重新跑某个 Story(规格改了)
   D4. AC 跑不通但代码看起来对
   D5. 跨 Story 接口冲突
E. 信息查询
   E1. 当前进度
   E2. 查看 Story 详情
   E3. AC 状态总览
   E4. Risk Register 更新
   E5. 性能预算 baseline
F. 协作交接
   F1. 交接给同事
   F2. PR review 准备
   F3. PR 改 review 反馈
G. 元操作
   G1. 查看本 SKILL 自身版本
   G2. SKILL 与 operator prompt 同步检查
   G3. 回滚到上一个 Story 完成态
H. 反模式(不要这样说)
I. Prompt 模板填空速查
   "开 Story"模板
   "续接 Story"模板
   "异常处理"模板
```

**关键约束**:
- 文中的项目名(例如 "lingshu")全部替换为 `{{project_name}}`
- 实际命令(如 `mvn validate` / `pytest` / `go test`)替换为 `_design.md` §2 的 `build_command / test_command / verify_command_pattern`
- 占位符(如 `<commit-sha>`, `<NNN>`, `<slug>`)保持原样
- 行数预算 200-400 行

### Stage 4:emit `<project_slug>-spec-driven-dev/SKILL.md`

读取 `~/.claude/skills/new-project-coach/PROJECT_SKILL_TEMPLATE.md`,按其 PLACEHOLDER TABLE 用 Stage 1 收集的元数据做字符串替换,emit 到:

```
~/.claude/skills/<project_slug>-spec-driven-dev/SKILL.md
```

**占位符替换清单**(22 个,与模板 PLACEHOLDER TABLE 对齐):

```
{{project_name}}              ← design frontmatter
{{project_slug}}              ← kebab-case 派生
{{iteration_unit}}            ← design §4 / §3
{{iteration_id_pattern}}      ← design §6
{{iteration_split_rules}}     ← design §6
{{iteration_unit_id}}         ← 具体 iteration id(每次 emit 时动态值,如 `#001` / `v1.0.0`)
{{tech_stack_summary}}        ← design §2
{{build_command}}             ← design §2
{{test_command}}              ← design §2 / §7
{{verify_command_pattern}}    ← design §7
{{tooling_check_commands}}    ← Stage 1 派生(build_command/test_command/verify 第一 word 去重 + §6 若提 SpecKit → 加 specify)
{{readme_path}}               ← 项目根 README 绝对路径(默认 `<repo>/README.md`)
{{commit_message_format}}     ← git workflow(默认 `feat(scope): <msg>`)
{{branch_strategy}}           ← git workflow(默认 `main + feature/<iteration_unit-id>`)
{{default_branch}}            ← git workflow(默认 `main`)
{{constitution_path}}         ← repo layout(默认 `.specify/memory/constitution.md`)
{{design_doc_path}}           ← absolute path
{{operator_prompt_path}}      ← absolute path
{{spec_prompts_path}}         ← absolute path
{{roadmap_path}}              ← 用户指定 / 默认 `<project_slug>/specs/ROADMAP.md`(SpecKit 风格)
{{last_updated_date}}         ← today
{{version}}                   ← design version
```

**任一 placeholder 替换为 `<unknown>` 或空 → 报错回去问 design-doc-coach,不允许 fallback 默认值。**

**emit bash**(真跑命令 — 含 HTML comment strip,保证 Claude Code auto-loader 能识别 emit 出来的 SKILL.md):

```bash
TEMPLATE="$HOME/.claude/skills/new-project-coach/PROJECT_SKILL_TEMPLATE.md"
DEST="$HOME/.claude/skills/${PROJECT_SLUG}-spec-driven-dev/SKILL.md"
TMP="${DEST}.tmp.$$"

# 前置:确认变量都已 set(Stage 1 metadata JSON 应已 in-memory 加载)
: "${PROJECT_NAME:?need PROJECT_NAME from Stage 1 metadata}"
: "${PROJECT_SLUG:?need PROJECT_SLUG from Stage 1 metadata}"
: "${ITERATION_UNIT:?need ITERATION_UNIT from design §3}"
: "${ITERATION_ID_PATTERN:?need ITERATION_ID_PATTERN from design §6}"
: "${ITERATION_SPLIT_RULES:?need ITERATION_SPLIT_RULES from design §6}"
: "${TECH_STACK_SUMMARY:?need TECH_STACK_SUMMARY from design §2}"
: "${BUILD_COMMAND:?need BUILD_COMMAND from design §2}"
: "${TEST_COMMAND:?need TEST_COMMAND from design §7}"
: "${VERIFY_COMMAND_PATTERN:?need VERIFY_COMMAND_PATTERN from design §7}"
: "${TOOLING_CHECK_COMMANDS:?need TOOLING_CHECK_COMMANDS derived in Stage 1}"
: "${README_PATH:?need README_PATH — default <repo>/README.md}"
: "${COMMIT_MESSAGE_FORMAT:=feat(scope): <msg>}"
: "${BRANCH_STRATEGY:=main + feature/<iteration_unit-id>}"
: "${DEFAULT_BRANCH:=main}"
: "${CONSTITUTION_PATH:=.specify/memory/constitution.md}"
: "${DESIGN_DOC_PATH:?need DESIGN_DOC_PATH — absolute path}"
: "${OPERATOR_PROMPT_PATH:?need OPERATOR_PROMPT_PATH — absolute path}"
: "${SPEC_PROMPTS_PATH:?need SPEC_PROMPTS_PATH — absolute path}"
: "${ROADMAP_PATH:?need ROADMAP_PATH — default <project_slug>/specs/ROADMAP.md}"
: "${LAST_UPDATED_DATE:=$(date +%Y-%m-%d)}"
: "${VERSION:?need VERSION from design frontmatter}"

mkdir -p "$(dirname "$DEST")"

# Pass 1 + Pass 2:用 awk 一次完成 {{var}} 替换 + HTML comment 块 strip
# (awk gsub 原生支持多行替换,无需 sed 的 `\<newline>` 技巧;awk 状态机天然处理 HTML comment 块)
# 注意:`{{tooling_check_commands}}` 是 Stage 1 派生的多行 bash 字符串,通过 temp file + awk getline 避免 `-v` 多行问题
TOOLING_CHECK_COMMANDS_FILE="${TMP}.tooling"
printf '%s\n' "$TOOLING_CHECK_COMMANDS" > "$TOOLING_CHECK_COMMANDS_FILE"

awk -v name="$PROJECT_NAME" \
    -v slug="$PROJECT_SLUG" \
    -v unit="$ITERATION_UNIT" \
    -v id_pat="$ITERATION_ID_PATTERN" \
    -v split_rules="$ITERATION_SPLIT_RULES" \
    -v unit_id="$ITERATION_UNIT_ID" \
    -v tech="$TECH_STACK_SUMMARY" \
    -v build="$BUILD_COMMAND" \
    -v test_cmd="$TEST_COMMAND" \
    -v verify="$VERIFY_COMMAND_PATTERN" \
    -v readme="$README_PATH" \
    -v commit_fmt="$COMMIT_MESSAGE_FORMAT" \
    -v branch_strat="$BRANCH_STRATEGY" \
    -v default_br="$DEFAULT_BRANCH" \
    -v const="$CONSTITUTION_PATH" \
    -v design="$DESIGN_DOC_PATH" \
    -v op="$OPERATOR_PROMPT_PATH" \
    -v spec_p="$SPEC_PROMPTS_PATH" \
    -v roadmap="$ROADMAP_PATH" \
    -v date="$LAST_UPDATED_DATE" \
    -v ver="$VERSION" \
    -v tooling_file="$TOOLING_CHECK_COMMANDS_FILE" \
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
    gsub(/\{\{iteration_unit\}\}/, unit, line)
    gsub(/\{\{iteration_id_pattern\}\}/, id_pat, line)
    gsub(/\{\{iteration_split_rules\}\}/, split_rules, line)
    gsub(/\{\{iteration_unit_id\}\}/, unit_id, line)
    gsub(/\{\{tech_stack_summary\}\}/, tech, line)
    gsub(/\{\{build_command\}\}/, build, line)
    gsub(/\{\{test_command\}\}/, test_cmd, line)
    gsub(/\{\{verify_command_pattern\}\}/, verify, line)
    gsub(/\{\{tooling_check_commands\}\}/, tooling, line)
    gsub(/\{\{readme_path\}\}/, readme, line)
    gsub(/\{\{commit_message_format\}\}/, commit_fmt, line)
    gsub(/\{\{branch_strategy\}\}/, branch_strat, line)
    gsub(/\{\{default_branch\}\}/, default_br, line)
    gsub(/\{\{constitution_path\}\}/, const, line)
    gsub(/\{\{design_doc_path\}\}/, design, line)
    gsub(/\{\{operator_prompt_path\}\}/, op, line)
    gsub(/\{\{spec_prompts_path\}\}/, spec_p, line)
    gsub(/\{\{roadmap_path\}\}/, roadmap, line)
    gsub(/\{\{last_updated_date\}\}/, date, line)
    gsub(/\{\{version\}\}/, ver, line)
    if ($0 ~ /^#/) in_c = 2
    print line
  }' "$TEMPLATE" > "$TMP"
rm -f "$TOOLING_CHECK_COMMANDS_FILE"

# 验证
head -1 "$TMP" | grep -q '^---$' || { echo "FAIL: frontmatter opener not at byte 0"; rm -f "$TMP"; exit 1; }
grep -q "^# ${PROJECT_NAME}-spec-driven-dev" "$TMP" || { echo "FAIL: heading missing"; rm -f "$TMP"; exit 1; }
grep -q "{{[a-z_]*}}" "$TMP" && { echo "FAIL: unsubstituted {{var}} remaining"; grep -o "{{[a-z_]*}}" "$TMP" | sort -u; rm -f "$TMP"; exit 1; }

# 落地
mv "$TMP" "$DEST"
echo "✓ emitted: $DEST"
```

**关键 strip 行为**:Pass 2 用 awk 把 frontmatter closer `---`(行 4)与首个 `# heading` 之间的 HTML comment 块全部删除,只 emit heading 之后的内容。最终 per-project SKILL.md 首行是 `---`,**无任何 HTML comment 残留**——Claude Code auto-loader 立即识别。

完成后,提醒用户把生成的 `<slug>-spec-driven-dev/SKILL.md` 复制到目标仓:
```bash
mkdir -p <project_repo>/.claude/skills/ <project_repo>/specs/
cp ~/.claude/skills/<slug>-spec-driven-dev/SKILL.md \
   <project_repo>/.claude/skills/<slug>-spec-driven-dev/SKILL.md
cp ROADMAP.md <project_repo>/specs/ROADMAP.md
```

### Stage 5:emit `<project_slug>/specs/ROADMAP.md`(工程 tracker — 区分于 SOP)

**为什么需要**(用户场景):
- Story 拆分不可能一次到位(到 §6 实施时才发现空白,如 lingshu Story #009e 实测发现)
- Story 优先级分 P0/P2 需动态调整(等 §6 全做完再讨论 P2)
- 已合 Story 与待补 Story 状态分离(快照 + tracker 双轨)
- 月度 review 需依赖 ROADMAP 而非 design doc(后者改必 RFC)

**输入**:
- `_design.md` §6 AC 总账(全量 Story 候选 → 🟡 待补)
- `_design.md` §11 风险登记(P0 / P2 分类)
- `_design.md` §5 / §6 OQ 标记(OQ-Future 候选)
- **已有** `<roadmap_path>`(若存在 → 保留 ✅ 已完成段,不重新生成)

**输出**:`<roadmap_path>`(默认 `<project_slug>/specs/ROADMAP.md`,SpecKit 风格)

**模板**(参考 lingshu,去项目特定):

```
✅ 已完成(从已有 ROADMAP 抄 — 不重生成)
   | Story # | slug | design § | 范围 | 文件数 | ErrorCode | PR | 日期 |
🟡 §X 关键实现 待补(P0 — 从 design §6 + §11 派生)
   | 序 | Story # | slug | design § | 范围 | 文件数 | ErrorCode | 依赖 |
⏸ §Y 生产增强 滞后(P2 — 从 design §11 派生)
   | N | 标题 | 潜在 Story # | 备注 |
📋 OQ-Future 项(从 design §5 / §6 OQ 派生)
   | OQ | 锚定 § | 标题 | 状态 | 重启 trigger |
🎯 实施顺序建议(依赖图 + 已合划掉)
🎯 选 Story 的入口(specify 命令速查)
📊 统计(N 已合 / M 待补 / X 文件 / Y ErrorCode / Z R-XX 缓解率)
```

**3-way merge 行为**(Stage 5 二次跑):
- 已有 ROADMAP → **保留** ✅ 已完成段(防回滚丢失)
- 重生成 🟡 / ⏸ / 📋 / 🎯 / 📊 段(反映 design 最新版)
- 若 design.md 加了新 Story,Stage 5 自动追加到 🟡 待补

**关键约束**:
- ROADMAP 是 **tracker**(可修改),与 constitution(法律,改必 RFC)严格区分
- 反过来,改 `_design.md` 后**必须重跑 Stage 5** —— 否则 ROADMAP 与设计漂移
- 每月 1 号月度 review:检查 P0 → P2 升级 / 滞后顺序调整

---

## 4 件套引用关系图(单 source of truth)

```
                <project>_design.md          ← 真理(SST)
                       │
        ┌──────────────┼──────────────┬─────────────────┐
        ▼              ▼              ▼                        ▼
   Stage 2         Stage 3         Stage 4                  Stage 5
_operator_    _spec_prompts    ~/.claude/skills/         specs/ROADMAP.md
  prompt.md      .md        <slug>-spec-driven-dev/       (工程 tracker)
   实施 SOP       窗口速查         SKILL.md
  (基本不动)    (基本不动)      (基本不动)                **(Story 合入后可变)**
```

**SST 边界**:
- `_design.md` = 真理(改必 RFC)
- `*_operator_prompt.md` / `*_spec_prompts.md` / 项目 SKILL = 蒸馏产物(改必重蒸馏)
- `ROADMAP.md` = tracker(每次 Story 合入可变 —— **4 件套中唯一可变**,不需回 design doc 重蒸馏)

**单向不回头**(ROADMAP 除外)。改任何 SOP / SKILL / prompts,**先回到 `_design.md`**,再重跑本 SKILL 重蒸馏。

## 反模式(看到立刻停)

| ❌ 反模式 | 为什么不许 |
|---|---|
| 替用户在 Stage 1 填占位符(design doc 没写的值)| 蒸馏不允许创造新决策;缺值就回去让 design-doc-coach 追问 |
| 把 `_design.md` 全文抄到 `_operator_prompt.md` | 引用章节号即可,避免多 SST 漂移 |
| `_spec_prompts.md` 复制了 `_operator_prompt.md` 内容 | prompts 只放窗口速查模板,内容重复会导致更新双倍工作量 |
| 项目专属 SKILL 不从模板 emit,自己手写 | PLACEHOLDER TABLE 是单一来源,手写会与未来模板更新漂移 |
| 一次蒸馏后不再维护版本号 | `_design.md` 改 → 3 件套全 bump,SKILL frontmatter `description` 也同步 |
| 跳过宪章蒸馏提示 | `_operator_prompt.md` 必须引用 `constitution.md`,否则第 1 个 Story 实施时会缺"宪法"约束 |
| 用通用"默认技术栈"(Java + Spring + Maven)填项目 | 锁定的技术栈必须从 design 抄,不允许 fallback |

## 输出检查清单(自检)

- [ ] `_operator_prompt.md` 含 Story 拆分映射表 + AC 黑盒规范 + 反模式表
- [ ] `_spec_prompts.md` 含 A-I 9 大类窗口模板
- [ ] `<slug>-spec-driven-dev/SKILL.md` 已 emit 到 `~/.claude/skills/`
- [ ] SKILL frontmatter 触发词唯一(不与现有 meta-SKILL 撞 — 检查 `lingshu-spec-driven-dev` / `design-doc-coach` / `spec-driven-bootstrap` 名字)
- [ ] 3 件套之间所有 `{{var}}` 已替换为非空值
- [ ] 元数据 JSON 中所有 source chapter 都对应 `_design.md` 实际存在的章节
- [ ] 文件尾 `Last updated` + `Version` 已填

## 下一步

1. 用户审 4 件套(operator_prompt / spec_prompts / SKILL / ROADMAP — 可分头审,不一定同 session)
2. 用户把 `<slug>-spec-driven-dev/SKILL.md` 复制到目标仓 `.claude/skills/`,把 ROADMAP 复制到 `<project_repo>/specs/`
3. 下次开窗口说"开始 `<project_name>` 的 Story #001"(或对应迭代单位),目标仓 SKILL 自动加载 → 按 Story 循环实施 + 每次合入自动更新 ROADMAP(项目 SKILL Step 4.5)

---

**Last updated**: 2026-09-21
**Version**: 1.4(v1.3 → v1.4: +Stage 4 真 bash 命令(awk 跳过 frontmatter 与首个 heading 之间的 HTML comment 块)+ Stage 4 placeholder list 补 2 个(`{{readme_path}}` `{{iteration_unit_id}}`)+ PROJECT_SKILL_TEMPLATE 重组 frontmatter 到 byte 0 + 补 `{{iteration_unit_id}}` 到 22 项 — 修「模板 frontmatter 渲染为纯文本」+「emit 时 HTML comment 污染 per-project SKILL.md」+「body 引用 TABLE 未列占位符」三条 latent bug;v1.3 的 Stage 5 ROADMAP emit + 3-way merge + 4 件套引用关系不变)
