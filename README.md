# _meta-starters/ — 元工具(跨项目可复用)SpecKit 体系

> **本目录是什么**:把"开新项目"的整条流程蒸馏成可调用的 SKILL + starter markdown,**任何新项目**都可以拿来即用,而不是每次重新发明。
>
> **与 lingshu 项目资产的关系**:本目录是"通用元工具"层,跟 `../MyDSHAgentDesign/` 下的 lingshu 项目资产(`dsh_agent_design.md` / `speckit_operator_prompt.md` / `lingshu-spec-driven-dev/SKILL.md` 等)是**两层**——前者管"如何为任意新项目**生成**这套资产",后者管"灵枢这个项目**怎么落地**"。

---

## 文件清单(本目录)

```
_meta-starters/
├── README.md                                ← 本文档(单源,无镜像)
├── design-doc-coach/SKILL.md                ← meta-SKILL #1:5 维度采访 → _design.md 草稿(v1.3)
├── spec-driven-bootstrap/SKILL.md           ← meta-SKILL #2:蒸馏 _design.md → 4 件套 SOP(含 ROADMAP,v1.3)
├── sdd-migrate/SKILL.md                     ← meta-SKILL #3:SpecKit-era → OpenSpec-era 单向迁移(v1.0)
└── new-project-coach/
    ├── STARTER.md                           ← 用户开场白模板(纯 markdown,无 frontmatter)
    ├── PROJECT_SKILL_TEMPLATE.md            ← 项目专属 SKILL 模板 — SpecKit-era(18 个 {{var}})
    └── OPENSPEC_PROJECT_SKILL_TEMPLATE.md   ← 项目专属 SKILL 模板 — OpenSpec-era(12 个 {{var}},v1.2)
```

**镜像位置**:`~/.claude/skills/` 同名目录(Claude Code auto-loader 看到 frontmatter 自动加载 3 个 meta-SKILL)。**所有 5 个文件 + 镜像** = 完整流水线。

---

## 5 件套流水线(扩展 4 件套 + Stage 6 sdd-migrate)

```
用户新开窗口 + 粘 STARTER.md 开场白
   │
   ▼
new-project-coach/STARTER.md(状态探测)
   │
   ├─ 无 _design.md → 跑 design-doc-coach
   │     │
   │     ▼ 5 维度采访(用 AskUserQuestion)
   │     ▼ 输出 _design.md 0.3 版草稿(含 <!-- TODO -->)
   │
   │  用户审稿 + 替换 TODO + 切 frontmatter `status: ready-for-bootstrap`
   │
   ├─ 有 _design.md → 跑 spec-driven-bootstrap
   │     │
   │     ▼ Stage 0:fuzzy 校验 §1 §2 §6(严格/模糊/关键词 三层)
   │     ▼ Stage 1:解析元数据(in-memory JSON)
   │     ▼ Stage 2:输出 _operator_prompt.md(实施 SOP)
   │     ▼ Stage 3:输出 _spec_prompts.md(窗口速查)
   │     ▼ Stage 4:从 PROJECT_SKILL_TEMPLATE 替换 {{var}} emit 项目专属 SKILL(SpecKit-era)
   │     ▼ Stage 5:emit ROADMAP.md(工程 tracker — Story 进度 + 实施顺序 + P0/P2 优先级)
   │
   ▼
用户把项目专属 SKILL + ROADMAP 复制到目标仓:
   cp ~/.claude/skills/<slug>-spec-driven-dev/SKILL.md <repo>/.claude/skills/
   cp ROADMAP.md <repo>/specs/ROADMAP.md
   │
   ▼ 项目开工(绿地阶段)
每次开窗口说"开始 <project> 的 Story #NNN",项目 SKILL 自动加载
   │
   ├─ 必读 4 件套 SOP + 设计文档 + ROADMAP tracker
   └─ **必读 README.md**(<repo>/README.md — 含已执行 Story 路线图段 / 历史段 / 核心特性段,与 ROADMAP 互补:ROADMAP = 实时 tracker,README = 公开宣告面)
   │
   ▼ 按 Story/Iteration 循环
   │
   ▼ 每次 PR 合入
项目 SKILL Step 4.5 自动跑:
   ├─ 🟡→✅ 行迁移 + 统计更新 + 实施顺序划掉 + ROADMAP 版本 bump
   └─ **README 「Story 路线图」段同步追加本 Story**(与 ROADMAP 同步,避免外部读者看到过期信息)
   │
   ▼ 项目 ≥ v1.0 + ≥ 5 ✅ Story(进入棕地阶段)
用户说"迁移到 OpenSpec" → sdd-migrate 自动加载
   │
   ├─ Stage 0:前置校验(版本 + Story 数 + 4 件套 + constitution + openspec CLI + cwd)
   ├─ Stage 1:解析 SpecKit 制品(design + SOP + prompts + **README** + ROADMAP + per-Story + constitution)
   ├─ Stage 2:派生 openspec/specs/<cap>/spec.md(✅ → capability,SHALL/MUST 改写 AC-NN)
   ├─ Stage 3:派生 openspec/changes/<change-id>/(🟡 → proposal,⏸ → changes/future/)
   ├─ Stage 4:emit openspec/AGENTS.md(从 constitution 蒸馏)
   ├─ Stage 5:emit openspec/project.md(从 design §0/§1/§2/§3/§4/§5/§12)
   ├─ Stage 6:emit openspec/config.yaml
   ├─ Stage 7:从 OPENSPEC_PROJECT_SKILL_TEMPLATE 替换 {{var}} emit 项目专属 SKILL(OpenSpec-era)
   ├─ Stage 8:归档 SpecKit-era 制品(→ archived/spec-kit-era/)+ **整 README 归档 + emit 新 README skeleton 指向 openspec/ 制品**
   └─ Stage 9:emit openspec/migration-report.md(含 README 处理摘要)
   │
   ▼ 后续增量开发(棕地阶段)
每次开窗口说"为 <project> 提 change #NNN",项目 SKILL 自动加载
   │
   ├─ 必读 openspec/project.md + AGENTS.md + specs/<cap>/spec.md + changes/
   └─ **必读 README.md**(含已换 capability 历史 / 已 archive change 路线图 — 与 openspec/changes/archive/ 互补:archive = 工程 tracker,README = 公开宣告面)
   │
   ▼ 按 /opsx:propose → /apply → /verify → /archive 循环
   │
   ▼ 每次 Change archive 后
项目 SKILL Step 4 反向同步:
   ├─ README 「已合 capability / 已 archive change」段追加本 Change(与 archive 同步,避免外部读者看到过期信息)
```

---

## 工具链安装指引(Pre-flight 跑 `command -v X` 之前必读)

`_meta-starters/` emit 出的项目专属 SKILL 会跑 **Pre-flight**(详见 `PROJECT_SKILL_TEMPLATE.md` 自动派生),**严格 AND 校验**所有构建/测试/规范工具 —— 任一缺失**强制中止**(防半路 fail 烧 token)。这是 AND 不是 OR:项目设计文档说"用 SpecKit + Maven",`command -v specify` 和 `command -v mvn` 都要过。

### SpecKit CLI(驱动 `_operator_prompt.md` 的 `/specify` `/plan` `/tasks` 命令)

**正常网络环境**:

```bash
uv tool install --from git+https://github.com/githubnext/spec-kit specify
specify --version   # 验证(应返回版本号)
```

**fake-ip / GFW 拦截 443 模式**(代理返回 fake-IP 而非真实 IP 时):

`uv tool install` 会在依赖解析阶段卡死(网络栈问题)。用 **git clone + 本地 pip install** 绕过:

```bash
git clone https://github.com/githubnext/spec-kit /tmp/spec-kit
cd /tmp/spec-kit
uv pip install --system -e .   # 退化用 pip install -e . 也可
which specify                  # 应返回可执行路径(例如 ~/.local/bin/specify)
specify --version              # 验证
```

**装不上的退化路径**:项目 SKILL 仍能 emit,只是 `/specify` 等命令不可用 —— 用户得**手动写** spec.md / plan.md / tasks.md(模板在 `_operator_prompt.md` §3.2/§3.3/§3.4)。Pre-flight 会以 "MISSING_SPECIFY" 提示,用户确认继续。

### OpenSpec CLI(项目 ≥ 1.0 + ≥ 5 ✅ Story 切棕地后必装 — 驱动 `/opsx:propose` 等命令)

**正常网络环境**:

```bash
npm install -g @fission-ai/openspec@latest
openspec --version         # 验证
cd <project_repo>
openspec init              # 项目内初始化(创建 openspec/ 目录)
```

**fake-ip / GFW 拦截 443 模式**(与 SpecKit 相同问题):

```bash
# npm 默认走 fake-ip 会卡,用国内镜像或挂代理
npm install -g @fission-ai/openspec@latest --registry=https://registry.npmmirror.com
# 或显式 npm config
npm config set registry https://registry.npmmirror.com
npm install -g @fission-ai/openspec@latest
```

**装不上的退化路径**:`sdd-migrate` 仍能跑(只生成文件结构,不调 openspec CLI),但用户后续 `/opsx:propose` `/apply` `/archive` 命令不可用 —— 用户得直接编辑 `openspec/changes/<id>/proposal.md` 等。Pre-flight 会以 "MISSING_OPENSPEC" 提示,用户确认继续。

**触发场景**:
- 项目 ≥ v1.0 + ≥ 5 个 ✅ Story 后,用户说"迁移到 OpenSpec" / "切到 OpenSpec" / "SpecKit 收官"
- sdd-migrate Stage 0 强制校验 `command -v openspec`,缺失中止

### 其他构建工具(Java/Maven、Python/uv、Go、Rust、Node 等)

由各项目 README 自己写。Pre-flight 只跑 `command -v X`,具体装法不重复 —— 本 README 不重复。

---

## 下次开新项目 — 6 步流程

实操步骤(从零起步,**预计 ~30 分钟** 到 SOP 全部就位 + 项目可开工):

1. **粘 STARTER 开场白**:`cat ~/Documents/AIFullStack/_meta-starters/new-project-coach/STARTER.md`,把"第二步 a"或"第二步 b"模板整段粘到新窗口首条消息
2. **跑 design-doc-coach**(无 `_design.md` 时):Claude 跑 5 维度 `AskUserQuestion` 采访 → 输出 `<project>_design.md` 0.3 版草稿(含 `<!-- TODO -->`)
3. **审稿 + 切状态**:
   - 把 `<!-- TODO -->` 全替换为确认内容
   - §6 AC 总账补到 ≥ 5 条
   - §11 风险登记补到 ≥ 3 条
   - 改 frontmatter:`status: awaiting-review` → `ready-for-bootstrap`
4. **跑 spec-driven-bootstrap**:新窗口粘"第二步 b"模板,Claude 跑 Stage 0-4 → 输出 `<project>_operator_prompt.md` + `<project>_spec_prompts.md` + emit 项目专属 SKILL(`~/.claude/skills/<slug>-spec-driven-dev/SKILL.md`)
5. **复制项目专属资产到目标仓**:
   ```bash
   mkdir -p <project_repo>/.claude/skills/ <project_repo>/specs/
   cp ~/.claude/skills/<slug>-spec-driven-dev/SKILL.md \
      <project_repo>/.claude/skills/<slug>-spec-driven-dev/SKILL.md
   cp ROADMAP.md <project_repo>/specs/ROADMAP.md
   ```
   同时复制 `<project>_operator_prompt.md` + `<project>_spec_prompts.md` 到目标仓工作目录(给同事看 / git diff)
6. **项目开工 + 维护 ROADMAP + README**:下次开窗口说"开始 `<project>` 的 Story #001"(或对应迭代单位),项目 SKILL 自动加载 → 按 Story/Iteration 循环;每次 PR 合入后,SKILL Step 4.5 自动更新:
   - ROADMAP(🟡→✅ 行迁移 + 统计 + 实施顺序划掉)
   - **README「Story 路线图」段追加本 Story**(与 ROADMAP 同步,README 含已执行 Story 历史 / 核心特性段,是外部读者看的"公开宣告面")
7. **项目 ≥ v1.0 + ≥ 5 ✅ Story → 切 OpenSpec**(可选,棕地过渡):新窗口说"迁移到 OpenSpec",`sdd-migrate` 自动加载 → Stage 0-9 跑完:
   - 复制新项目 SKILL:`cp ~/.claude/skills/<slug>-openspec-dev/SKILL.md <repo>/.claude/skills/`
   - 旧 `<slug>-spec-driven-dev/SKILL.md` + SpecKit 4 件套 + ROADMAP + **README 整文件** 已自动归档到 `archived/spec-kit-era/`
   - **新 `<repo>/README.md` skeleton 已 emit**,指向 `openspec/project.md` + `openspec/AGENTS.md` + `openspec/specs/` + `openspec/changes/archive/`
   - 后续增量开发走 `/opsx:propose → /apply → /verify → /archive`;每次 archive 后,SKILL Step 4 反向同步 README「已合 capability」段(详见「OpenSpec vs SpecKit」对照表)

**关键提示**:
- 步骤 1-3 耗时最长(采访 + 审稿),**不要为了快跳过 TODO 替换**
- 步骤 4 自动跑,不必手动触发
- 步骤 5 复制项目 SKILL 后,**新开窗口试一次触发词**确认 auto-loader 加载到位
- 若 `_design.md` 改动,重跑步骤 4(spec-driven-bootstrap)重蒸馏 **4 件套** + bump 版本,**不要直接改 SOP**
- **ROADMAP 特殊性**:4 件套中**唯一可变**的 —— Story 合入后由项目 SKILL Step 4.5 更新(🟡→✅ + 统计 + 实施顺序),**不回 design doc 重蒸馏**

---

## 镜像规则(双写纪律)

**所有 4 个 meta-文件** 同时存在于两个位置:

| 位置 | 角色 | 用途 |
|---|---|---|
| `~/Documents/AIFullStack/_meta-starters/` | 项目工作区 | git 跟踪 / diff / 给同事看 / 文档可发现 |
| `~/.claude/skills/` | Claude Code 运行时 | auto-loader 看到 frontmatter 自动加载 2 个 meta-SKILL |

**操作纪律**(避免漂移):

1. **写入即镜像**:改一个位置 → 立刻 `cp` 到另一个。不积累延迟。
2. **不用 symlink**:文件系统迁移 / 权限问题容易断。复制最稳。
3. **校验**:`md5 -q <src> <dst>` 两边对比必须 MATCH。
4. **PARITY 校验脚本**(贴到 `~/.zshrc` 备用):
   ```bash
   for slug in design-doc-coach spec-driven-bootstrap sdd-migrate; do
     for meta in SKILL.md; do
       a=$(md5 -q ~/.claude/skills/$slug/$meta)
       b=$(md5 -q ~/Documents/AIFullStack/_meta-starters/$slug/$meta)
       [ "$a" = "$b" ] && echo "MATCH  $slug/$meta" || echo "DRIFT! $slug/$meta ($a vs $b)"
     done
   done
   for meta in STARTER.md PROJECT_SKILL_TEMPLATE.md OPENSPEC_PROJECT_SKILL_TEMPLATE.md; do
     a=$(md5 -q ~/.claude/skills/new-project-coach/$meta)
     b=$(md5 -q ~/Documents/AIFullStack/_meta-starters/new-project-coach/$meta)
     [ "$a" = "$b" ] && echo "MATCH  new-project-coach/$meta" || echo "DRIFT! new-project-coach/$meta"
   done
   ```

---

## 版本纪律

- **任何改动**都 bump `Last updated` + `Version`(加小版本号 v1.0 → v1.1)
- 改动说明附在 Version 后,例如:
  ```
  **Version**: 1.1(v1.0 → v1.1: +fuzzy check Stage 0;+allowedPrompts 免权限弹窗;流程不变)
  ```
- **镜像两侧同步 bump**(改一侧的 Version,另一侧复制时 Version 同步)

---

## 何时 NOT 把文件放这里

为保持本目录的**元工具**性质,以下文件请勿混入:

| ❌ 别放这里 | ✅ 正确位置 |
|---|---|
| 任何具体项目的 `_design.md`(如 `fooflow_design.md`)| 项目仓根(由 design-doc-coach 生成) |
| 任何具体项目的 `_operator_prompt.md` / `_spec_prompts.md` | 项目工作目录(由 spec-driven-bootstrap 生成) |
| 任何具体项目的 `<project>-spec-driven-dev/SKILL.md`(已 emit)| 项目仓 `<repo>/.claude/skills/` |
| 改 lingshu 项目设计文档副本 | 改 `../MyDSHAgentDesign/dsh_agent_design.md` 主源 |

**例外**:`PROJECT_SKILL_TEMPLATE.md` 虽然形式上看起来像项目专属 SKILL,但它带 18 个 `{{var}}` 占位符,是 spec-driven-bootstrap Stage 4 的 emit 模板,所以归本目录。

---

## 添加新 meta-SKILL / 改既有 SKILL 流程

### 改既有 meta-SKILL(小改动)

1. 编辑 `~/.claude/skills/<slug>/SKILL.md`(auto-loader 立等可见)
2. 编辑 `~/Documents/AIFullStack/_meta-starters/<slug>/SKILL.md`(同步镜像)
3. 两边 bump `Version` + 改动说明
4. 跑 PARITY 校验脚本验证 md5 MATCH
5. (可选)开新窗口试触发词,验证 SKILL 自动加载

### 加新 meta-SKILL(大改动)

1. 起草 `~/.claude/skills/<new-slug>/SKILL.md`(frontmatter + body)
2. 草稿里写清"何时调用 / 何时 NOT 调用"(防误触发)
3. **不带 frontmatter 时不是 SKILL**:确定要不要 SKILL 行为
4. smoke test:新窗口试触发词 → auto-loader 应自动加载
5. 镜像到 `~/Documents/AIFullStack/_meta-starters/<new-slug>/SKILL.md`
6. 跑 PARITY 校验
7. **本 README 的"文件清单"小节加一行** + "添加新 meta-SKILL 流程"小节必要时更新
8. bump 本 README 版本

---

## git 操作(可选,本目录尚未 git init)

如果要把 `\_meta-starters/` 独立 git 跟踪:

```bash
cd ~/Documents/AIFullStack/_meta-starters
git init
git add .
git commit -m "feat(meta): v1.0 — design-doc-coach + spec-driven-bootstrap + STARTER + PROJECT_SKILL_TEMPLATE"
```

**注意**:`~/.claude/skills/`(运行时目录)**不要 git init** —— 那是 Claude Code 管的目录,可能含项目专属 SKILL、用户偏好文件等,**不属于本元工具体系**。

---

## 相关阅读

- **lingshu 项目**(本元体系的灵感来源 + 落地验证):
  - `../MyDSHAgentDesign/CLAUDE.md` — 项目级 Claude Code 引导
  - `../MyDSHAgentDesign/dsh_agent_design.md` — 设计文档(v1.5.34 / 7250 行 / 项目真理)
  - `../MyDSHAgentDesign/speckit_operator_prompt.md` — SpecKit 操作 SOP(v1.18)
  - `../MyDSHAgentDesign/lingshu_spec_prompts.md` — 新窗口 prompt 速查(v1.0.16)
  - `../../.claude/skills/lingshu-spec-driven-dev/SKILL.md` — lingshu 项目专属 SKILL(v1.0.21)

- **设计思想跨层级映射**:
  | 层级 | lingshu 项目实例 | 元工具体系实例 |
  |---|---|---|
  | 设计文档 | `dsh_agent_design.md` v1.5.34 | `*_design.md` v0.3(由 design-doc-coach 生成) |
  | 操作 SOP | `speckit_operator_prompt.md` v1.18 | `*_operator_prompt.md`(由 spec-driven-bootstrap 生成) |
  | Prompt 速查 | `lingshu_spec_prompts.md` v1.0.16 | `*_spec_prompts.md`(由 spec-driven-bootstrap 生成) |
  | 项目 SKILL | `lingshu-spec-driven-dev/SKILL.md` v1.0.21 | `<slug>-spec-driven-dev/SKILL.md`(由 PROJECT_SKILL_TEMPLATE emit) |
  | 元工具 | — | 本 README + 上述 4 件套 |

---

**Last updated**: 2026-09-29
**Version**: 1.10(v1.9 → v1.10: +**sdd-migrate fence 配对修正** — v1.2 → v1.3:(a) 删 line 681 orphan ` ``` `;(b) 删 line 370 stray ` ``` ` 改 `---` HR;(c) numbered list 加显式 opener line 352;(d) SpecKit-era 归档树加显式 opener line 642 — 修「Stage 6/7/8/9 标题 + `## 元 SKILL 关系` + 版本元数据 全部渲染成 code 块 / `---` 不渲染 HR / `**Last updated**` 显示字面 `**` 而非加粗」latent bug)

(配套:`design-doc-coach` v1.3 / `spec-driven-bootstrap` v1.4 / `sdd-migrate` v1.3 / `STARTER.md` v1.0 / `PROJECT_SKILL_TEMPLATE.md` v1.5 / `OPENSPEC_PROJECT_SKILL_TEMPLATE.md` v1.3)

---

## OpenSpec vs SpecKit 关键对照(切棕地前必读)

| 维度 | SpecKit-era(绿地) | OpenSpec-era(棕地增量)|
|---|---|---|
| 工具 | `/specify /plan /tasks` | `/opsx:propose /apply /verify /archive` |
| 真理源 | `<project>_design.md`(集中) | `openspec/specs/<cap>/spec.md`(分散,每 capability 一份 SoT)|
| 制品目录 | `specs/<NNN>-<slug>/`(per-Story 三件套) | `openspec/specs/<cap>/`(SoT)+ `openspec/changes/<change-id>/`(proposal)|
| 状态记录 | `ROADMAP.md`(手维护 ✅/🟡/⏸/📋)+ `README.md`「Story 路线图」段 | `openspec/changes/archive/YYYY-MM-DD-<id>/`(自动归档)+ `README.md`「已合 capability」段 |
| AC 表达 | AC-NN(可 grep)| Scenario Spec 内 GIVEN/WHEN/THEN/AND(自描述)|
| 迭代单位 | Story(#NNN-slug)| Change(`<NNN>-<slug>` kebab-case capability)|
| 切换触发 | — | 项目 ≥ v1.0 + ≥ 5 ✅ Story + `command -v openspec` |
