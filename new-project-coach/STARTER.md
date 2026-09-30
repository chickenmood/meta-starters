# new-project-starter

> **用法**:开新项目时,把"开场白模板"小节里的对应 block 整段复制到一个新 Claude Code 窗口的首条消息。Claude 会按窗口首条 message 的语义自动加载对应的 meta-SKILL(`design-doc-coach` 或 `spec-driven-bootstrap`)。
>
> **本文件不是 SKILL**(无 frontmatter),不会被 Claude Code auto-loader 触发。它是用户的"开场白模板"。

---

## 第一步:状态探测

开窗之前先看一眼目标目录:

```bash
ls -la <目标目录>/
test -f <目标目录>/<project_name>_design.md && echo "DESIGN_EXISTS" || echo "NO_DESIGN_YET"
```

- `DESIGN_EXISTS` → 进 **第二步 b**(bootstrap)
- `NO_DESIGN_YET` → 进 **第二步 a**(先做采访)

**判定标准**:`<project_name>_design.md` 存在 + 含 §1 项目原则 + §2 技术栈 + §6 Story/Iteration 拆分初版,**这三块齐**就视为"可 bootstrap";否则同样回第二步 a。

---

## 第二步 a:无设计文档 → 跑 design-doc-coach

把下面这段(连同反引号)粘到新窗口:

```text
我要开新项目 `<project_name>`(1 句话定位: <xxx>)。
请跑 design-doc-coach,从 5 维度采访开始。
```

**预期**:
- `design-doc-coach` SKILL 自动加载
- Claude 用 `AskUserQuestion` 跑 5 维度采访(项目元数据 / 技术栈 / 模块结构 / 拆分原则 / 验证手段 / 风险阈值)
- 输出 `<project_name>_design.md` 0.3 版草稿(含 `<!-- TODO -->` 标记让你审)
- 你审稿 + 把 TODO 替换为确认内容
- 改完进第二步 b

**如果你已经手写过设计文档**:
- 也粘 `design-doc-coach`,告诉它"我已写完 `<path>`,请帮我跑 Step 0 校验 + 补充 TODO 标记"
- 它会读你的现有文件 + 指出哪些章节缺、未填项是哪些,**不会重写**

---

## 第二步 b:已有设计文档 → 跑 spec-driven-bootstrap

把下面这段粘到新窗口:

```text
新项目 `<project_name>` 的设计文档已就位:
- 路径: <absolute path>/<project_name>_design.md
- 版本: vX.Y
- 推荐输出目录: <absolute path>/(本目录,或新建子目录)

请跑 spec-driven-bootstrap,生成 3 件套 SOP。
```

**预期**:
- `spec-driven-bootstrap` SKILL 自动加载
- 读设计文档,生成 3 件套:
  - `<project_name>_operator_prompt.md`(SOP,Claude 实施时读)
  - `<project_name>_spec_prompts.md`(窗口速查,你开窗时粘)
  - `~/.claude/skills/<project_slug>-spec-driven-dev/SKILL.md`(项目专属自动加载 SKILL)
- 3 件套全部审稿
- 把项目专属 SKILL 复制到目标仓的 `.claude/skills/`,确认能自动加载
- 项目正式开工

---

## 后续节奏(项目开工后)

- **每次开窗口**:直接说"开始 `<project_name>` 的 Story #NNN"(或对应迭代单位),目标仓 `.claude/skills/<project_slug>-spec-driven-dev/SKILL.md` 会自动加载
- **每次改设计文档**:**不要直接改 SOP**;先改 `_design.md`,再重跑 `spec-driven-bootstrap` 蒸馏(版本号 bump 同步走 SOP + prompts + 项目 SKILL)
- **加新项目**:再开一个新窗口 + 粘本 STARTER

---

## 常见误区(粘之前看一下)

| ❌ 误区 | ✅ 正确 |
|---|---|
| "我自己写 design doc,然后让 Claude 跑 bootstrap" | 也可以,**但**写完记得跑 Step 0 校验 + TODO 清理,否则 bootstrap 会失败 |
| "我要一上来就生成所有 SOP" | 必须先有 _design.md;SKILL 会强制你做 |
| "设计文档嘛我自己写就行" | 你可以,**但**宪章蒸馏 / AC 拆分 / 风险登记这三步别省,SKILL 强制依赖 |
| "STARTER 是 SKILL 吗" | 不是,纯 markdown,不会被 Claude Code auto-loader 触发 |
| "跑完 design-doc-coach 后直接进 bootstrap" | 必须先审稿 TODO + 改完,**确认 §1 §2 §6 三块齐**再进 bootstrap |

---

## 元 SKILL 关系图

```
new-project-starter.md(本文件,用户开场白)
    │
    ├─ 检测无 _design.md
    ▼
design-doc-coach SKILL    ← 输出 _design.md 草稿(含 TODO)
    │  用户审稿
    ▼
spec-driven-bootstrap SKILL  ← 输出 3 件套(operator/prompts/SKILL)
    │
    ▼
<project_slug>-spec-driven-dev SKILL  ← 自动加载,实施时用
```

三层不跳步,每层有强制前置。

---

**Last updated**: 2026-09-21
**Version**: 1.0
(与 meta-SKILL `design-doc-coach` v1.0 / `spec-driven-bootstrap` v1.0 / `PROJECT_SKILL_TEMPLATE` v1.0 配套)
